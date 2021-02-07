---
title: Blazor WebAssemblyProcedure consigliate per le prestazioni ASP.NET Core
author: pranavkm
description: Suggerimenti per migliorare le prestazioni in ASP.NET Core Blazor WebAssembly app ed evitare problemi comuni relativi alle prestazioni.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc, devx-track-js
ms.date: 10/09/2020
no-loc:
- appsettings.json
- ASP.NET Core Identity
- cookie
- Cookie
- Blazor
- Blazor Server
- Blazor WebAssembly
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: blazor/webassembly-performance-best-practices
ms.openlocfilehash: 28fe6a114f767246f49ac275d02c28f4572ce4e4
ms.sourcegitcommit: 19a004ff2be73876a9ef0f1ac44d0331849ad159
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/07/2021
ms.locfileid: "99804513"
---
# <a name="aspnet-core-blazor-webassembly-performance-best-practices"></a>Blazor WebAssemblyProcedure consigliate per le prestazioni ASP.NET Core

Di [](https://github.com/pranavkm) [Manuel e Steve Sanderson](https://github.com/SteveSandersonMS)

Blazor WebAssembly viene accuratamente progettato e ottimizzato per garantire prestazioni elevate in scenari di interfaccia utente dell'applicazione più realistici. Tuttavia, la produzione dei migliori risultati dipende dagli sviluppatori che usano i modelli e le funzionalità corrette. Prendere in considerazione gli aspetti seguenti:

* **Velocità effettiva di runtime**: il codice .NET viene eseguito in un interprete nel runtime di webassembly, quindi la velocità effettiva della CPU è limitata. In scenari complessi, l'app trae vantaggio dall' [ottimizzazione della velocità di rendering](#optimize-rendering-speed).
* **Tempo di avvio**: l'app trasferisce un Runtime .NET al browser, quindi è importante usare le funzionalità che [riducono al minimo le dimensioni del download dell'applicazione](#minimize-app-download-size).

## <a name="optimize-rendering-speed"></a>Ottimizzare la velocità di rendering

Le sezioni seguenti forniscono consigli per ridurre il carico di lavoro di rendering e migliorare la velocità di risposta dell'interfaccia utente. Seguendo questo Consiglio, è possibile eseguire facilmente un *miglioramento di dieci volte o superiore* nella velocità di rendering dell'interfaccia utente.

### <a name="avoid-unnecessary-rendering-of-component-subtrees"></a>Evitare il rendering superfluo di sottoalberi dei componenti

In fase di esecuzione, i componenti esistono come una gerarchia. Un componente radice contiene componenti figlio. A sua volta, i figli della radice hanno i propri componenti figlio e così via. Quando si verifica un evento, ad esempio un utente che seleziona un pulsante, questo è il modo Blazor in cui decide i componenti di cui eseguire il rendering:

1. L'evento stesso viene inviato a qualsiasi componente di cui è stato eseguito il rendering del gestore dell'evento. Dopo l'esecuzione del gestore dell'evento, viene eseguito il rendering del componente.
1. Ogni volta che viene eseguito il rendering di un componente, fornisce una nuova copia dei valori di parametro a ognuno dei relativi componenti figlio.
1. Quando si riceve un nuovo set di valori di parametro, ogni componente sceglie se eseguire nuovamente il rendering. Per impostazione predefinita, i componenti eseguono il rendering se è possibile che i valori dei parametri siano stati modificati, ad esempio se sono oggetti modificabili.

Gli ultimi due passaggi di questa sequenza continuano in modo ricorsivo la gerarchia dei componenti. In molti casi, viene eseguito il rendering dell'intero sottoalbero. Questo significa che gli eventi destinati a componenti di alto livello possono causare costosi processi di rirendering perché è necessario eseguire il rendering di tutti gli elementi al di sotto di tale punto.

Se si desidera interrompere il processo e impedire il rendering della ricorsione in un sottoalbero particolare, è possibile eseguire una delle operazioni seguenti:

* Verificare che tutti i parametri di un determinato componente siano tipi non modificabili primitivi (ad esempio,,,, `string` `int` `bool` `DateTime` e altri). La logica predefinita per il rilevamento delle modifiche ignora automaticamente il rendering se nessuno di questi valori di parametro è stato modificato. Se si esegue il rendering di un componente figlio con `<Customer CustomerId="@item.CustomerId" />` , dove `CustomerId` è un `int` valore, non viene nuovamente eseguito il rendering tranne quando viene `item.CustomerId` modificato.
* Se è necessario accettare valori di parametro non primitivi, ad esempio tipi di modelli personalizzati, callback di eventi o <xref:Microsoft.AspNetCore.Components.RenderFragment> valori, è possibile eseguire l'override <xref:Microsoft.AspNetCore.Components.ComponentBase.ShouldRender%2A> di per controllare la decisione relativa all'eventuale rendering, descritto nella sezione [utilizzo `ShouldRender` di](#use-of-shouldrender) .

Ignorando il rendering di interi sottoalberi, potrebbe essere possibile rimuovere la maggior parte del costo di rendering quando si verifica un evento.

È possibile scomporre i componenti figlio in modo specifico, in modo che sia possibile ignorare il rendering della parte dell'interfaccia utente. Si tratta di un modo valido per ridurre il costo di rendering di un componente padre.

#### <a name="use-of-shouldrender"></a>Utilizzo di `ShouldRender`

Se la creazione di un componente solo interfaccia utente che non cambia mai dopo il rendering iniziale (indipendentemente dai valori dei parametri), configurare <xref:Microsoft.AspNetCore.Components.ComponentBase.ShouldRender%2A> per restituire `false` :

```razor
@code {
    protected override bool ShouldRender() => false;
}
```

Se il componente richiede solo il rirendering quando i valori dei parametri cambiano in modo particolare, è possibile usare campi privati per tenere traccia delle informazioni necessarie per rilevare le modifiche. Nell'esempio seguente, `shouldRender` si basa sul controllo di qualsiasi tipo di modifica o mutazione che dovrebbe richiedere un rirendering. `prevOutboundFlightId` e `prevInboundFlightId` tenere traccia delle informazioni per il prossimo aggiornamento potenziale:

```razor
@code {
    [Parameter]
    public FlightInfo OutboundFlight { get; set; }
    
    [Parameter]
    public FlightInfo InboundFlight { get; set; }

    private int prevOutboundFlightId;
    private int prevInboundFlightId;
    private bool shouldRender;

    protected override void OnParametersSet()
    {
        shouldRender = OutboundFlight.FlightId != prevOutboundFlightId
            || InboundFlight.FlightId != prevInboundFlightId;

        prevOutboundFlightId = OutboundFlight.FlightId;
        prevInboundFlightId = InboundFlight.FlightId;
    }

    protected override bool ShouldRender() => shouldRender;

    // Note that 
}
```

Nel codice precedente, un gestore eventi può anche impostare `shouldRender` su in `true` modo che il componente venga nuovamente eseguito dopo l'evento.

Per la maggior parte dei componenti, questo livello di controllo manuale non è necessario. È consigliabile ignorare il rendering di sottoalberi se tali sottoalberi sono particolarmente costosi per il rendering e causano un ritardo dell'interfaccia utente.

Per altre informazioni, vedere <xref:blazor/components/lifecycle>.

::: moniker range=">= aspnetcore-5.0"

### <a name="virtualization"></a>Virtualizzazione

Quando si esegue il rendering di grandi quantità di interfaccia utente all'interno di un ciclo, ad esempio un elenco o una griglia con migliaia di voci, la quantità totale di operazioni di rendering può causare un ritardo nel rendering dell'interfaccia utente e quindi un'esperienza utente insufficiente. Dato che l'utente può visualizzare solo un numero ridotto di elementi in una sola volta senza scorrere, sembra superfluo dedicare molto tempo al rendering degli elementi che attualmente non sono visibili.

Per risolvere questo problema, Blazor fornisce il `Virtualize` componente che crea l'aspetto e i comportamenti di scorrimento di un elenco di grandi dimensioni arbitrario, ma esegue solo il rendering degli elementi dell'elenco che si trovano all'interno del viewport di scorrimento corrente. Questo significa, ad esempio, che l'app può avere un elenco con 100.000 voci, ma paga solo il costo di rendering di 20 elementi visibili in un momento qualsiasi. L'uso del `Virtualize` componente può aumentare le prestazioni dell'interfaccia utente in base agli ordini di grandezza.

Per altre informazioni, vedere <xref:blazor/components/virtualization>.

::: moniker-end

### <a name="create-lightweight-optimized-components"></a>Creazione di componenti semplici e ottimizzati

La maggior parte dei Blazor componenti non richiede attività di ottimizzazione aggressive. Questo è dovuto al fatto che la maggior parte dei componenti non si ripete spesso nell'interfaccia utente e non viene eseguito il rendering a frequenza elevata. Ad esempio, `@page` componenti e componenti che rappresentano parti dell'interfaccia utente di alto livello come finestre di dialogo o moduli, molto probabilmente vengono visualizzati solo uno alla volta e ne viene eseguito il rendering solo in risposta a un movimento dell'utente. Questi componenti non creano un carico di lavoro di rendering elevato, quindi è possibile usare liberamente qualsiasi combinazione di funzionalità del Framework che si vuole senza preoccuparsi delle prestazioni di rendering.

Tuttavia, esistono anche scenari comuni in cui si compilano componenti che devono essere ripetuti su larga scala. Ad esempio:

* I moduli annidati di grandi dimensioni possono avere centinaia di input singoli, etichette e altri elementi.
* Le griglie possono contenere migliaia di celle.
* I grafici a dispersione possono contenere milioni di punti dati.

Se si modellano le singole unità come istanze di componenti separate, ne saranno presenti così tante che le prestazioni di rendering diventano critiche. In questa sezione vengono fornite indicazioni su come rendere tali componenti leggeri, in modo che l'interfaccia utente rimanga veloce e reattiva.

#### <a name="avoid-thousands-of-component-instances"></a>Evitare migliaia di istanze di componenti

Ogni componente è un'isola separata che può essere sottoposta a rendering indipendentemente dagli elementi padre e figlio. Scegliendo come suddividere l'interfaccia utente in una gerarchia di componenti, si sta assumendo il controllo della granularità del rendering dell'interfaccia utente. Questo può essere positivo o negativo per le prestazioni.

* Suddividendo l'interfaccia utente in più componenti, è possibile avere parti più piccole del rendering dell'interfaccia utente quando si verificano gli eventi. Ad esempio, quando un utente fa clic su un pulsante in una riga di tabella, è possibile che venga eseguito solo il rirendering di una singola riga anziché l'intera pagina o tabella.
* Tuttavia, ogni componente aggiuntivo comporta un sovraccarico di memoria e CPU aggiuntivo per gestire lo stato indipendente e il ciclo di vita di rendering.

Quando si ottimizzano le prestazioni di Blazor WebAssembly in .NET 5, è stato misurato un sovraccarico di rendering di circa 0,06 ms per ogni istanza del componente. Si basa su un componente semplice che accetta tre parametri in esecuzione su un portatile tipico. Internamente, l'overhead è in gran parte dovuto al recupero dello stato per componente dai dizionari e al passaggio e alla ricezione di parametri. Per moltiplicazione, è possibile notare che l'aggiunta di 2.000 istanze di componenti aggiuntivi aggiungerà 0,12 secondi al tempo di rendering e l'interfaccia utente inizierà a rallentare gli utenti.

È possibile rendere i componenti più leggeri, in modo da poterli avere più, ma spesso la tecnica più potente non deve avere così tanti componenti. Le sezioni seguenti descrivono due approcci.

##### <a name="inline-child-components-into-their-parents"></a>Componenti figlio inline nei relativi elementi padre

Si consideri il componente seguente che esegue il rendering di una sequenza di componenti figlio:

```razor
<div class="chat">
    @foreach (var message in messages)
    {
        <ChatMessageDisplay Message="@message" />
    }
</div>
```

Per il codice di esempio precedente, il `<ChatMessageDisplay>` componente viene definito in un file `ChatMessageDisplay.razor` contenente:

```razor
<div class="chat-message">
    <span class="author">@Message.Author</span>
    <span class="text">@Message.Text</span>
</div>

@code {
    [Parameter]
    public ChatMessage Message { get; set; }
}
```

L'esempio precedente funziona correttamente e viene eseguito correttamente a condizione che migliaia di messaggi non vengano visualizzati contemporaneamente. Per mostrare migliaia di messaggi contemporaneamente, provare a *non* scomporre il `ChatMessageDisplay` componente separato. Inline invece il rendering direttamente nell'elemento padre:

```razor
<div class="chat">
    @foreach (var message in messages)
    {
        <div class="chat-message">
            <span class="author">@message.Author</span>
            <span class="text">@message.Text</span>
        </div>
    }
</div>
```

In questo modo si evita l'overhead per componente per il rendering di un numero così elevato di componenti figlio, a scapito del fatto che non è possibile eseguire nuovamente il rendering di ognuno di essi in modo indipendente.

##### <a name="define-reusable-renderfragments-in-code"></a>Definire riutilizzabile `RenderFragments` nel codice

È possibile scomporre i componenti figlio esclusivamente come un modo per riutilizzare la logica di rendering. In tal caso, è comunque possibile riutilizzare la logica di rendering senza dichiarare i componenti effettivi. Nel blocco di un componente è `@code` possibile definire un <xref:Microsoft.AspNetCore.Components.RenderFragment> che genera l'interfaccia utente e può essere chiamato da qualsiasi posizione:

```razor
<h1>Hello, world!</h1>

@RenderWelcomeInfo

@code {
    private RenderFragment RenderWelcomeInfo = __builder =>
    {
        <div>
            <p>Welcome to your new app!</p>

            <SurveyPrompt Title="How is Blazor working for you?" />
        </div>
    };
}
```

Come demonstated nell'esempio precedente, i componenti possono emettere markup dal codice all'interno del `@code` blocco e al di fuori di esso. Questo approccio definisce un <xref:Microsoft.AspNetCore.Components.RenderFragment> delegato di cui è possibile eseguire il rendering all'interno dell'output di rendering normale del componente, facoltativamente in più posizioni. È necessario che il delegato accetti un parametro denominato `__builder` di tipo <xref:Microsoft.AspNetCore.Components.Rendering.RenderTreeBuilder> in modo che il Razor compilatore possa produrre istruzioni per il rendering.

Se si desidera rendere questo oggetto riutilizzabile in più componenti, è consigliabile dichiararlo come `public static` membro:

```razor
public static RenderFragment SayHello = __builder =>
{
    <h1>Hello!</h1>
};
```

Questo ora può essere richiamato da un componente non correlato. Questa tecnica è utile per la creazione di librerie di frammenti di markup riutilizzabili che eseguono il rendering senza alcun sovraccarico per componente.

<xref:Microsoft.AspNetCore.Components.RenderFragment> i delegati possono anche accettare parametri. Per creare l'equivalente del `ChatMessageDisplay` componente dall'esempio precedente:

```razor
<div class="chat">
    @foreach (var message in messages)
    {
        @ChatMessageDisplay(message)
    }
</div>

@code {
    private RenderFragment<ChatMessage> ChatMessageDisplay = message => __builder =>
    {
        <div class="chat-message">
            <span class="author">@message.Author</span>
            <span class="text">@message.Text</span>
        </div>
    };
}
```

Questo approccio offre il vantaggio di riutilizzare la logica di rendering senza overhead per componente. Tuttavia, non è in grado di aggiornare il sottoalbero dell'interfaccia utente in modo indipendente, né di ignorare il rendering del sottoalbero dell'interfaccia utente quando viene eseguito il rendering dell'elemento padre, dal momento che non esiste alcun limite per i componenti.

Per un campo, un metodo o una proprietà non statica a cui non è possibile fare riferimento da un inizializzatore di campo, ad esempio `TitleTemplate` nell'esempio seguente, usare una proprietà anziché un campo per <xref:Microsoft.AspNetCore.Components.RenderFragment> :

```csharp
protected RenderFragment DisplayTitle => __builder =>
{
    <div>
        @TitleTemplate
    </div>   
};
```

#### <a name="dont-receive-too-many-parameters"></a>Non ricevere troppi parametri

Se un componente si ripete molto spesso, ad esempio centinaia o migliaia di volte, tenere presente che l'overhead associato al passaggio e alla ricezione di ogni parametro viene compilato.

È raro che troppi parametri limitino notevolmente le prestazioni, ma può essere un fattore. Per un `<TableCell>` componente che esegue il rendering di 1.000 volte all'interno di una griglia, ogni parametro aggiuntivo passato a esso potrebbe aggiungere circa 15 ms al costo totale di rendering. Se ogni cella accettava 10 parametri, il passaggio del parametro richiede circa 150 ms per ogni componente e, di conseguenza, 150.000 ms (150 secondi) e, di conseguenza, un'interfaccia utente ritardata.

Per ridurre questo carico, è possibile raggruppare più parametri tramite classi personalizzate. È ad esempio possibile che un `<TableCell>` componente accetti:

```razor
@typeparam TItem

...

@code {
    [Parameter]
    public TItem Data { get; set; }
    
    [Parameter]
    public GridOptions Options { get; set; }
}
```

Nell'esempio precedente, `Data` è diverso per ogni cella, ma `Options` è comune in tutti gli elementi. Naturalmente, potrebbe trattarsi di un miglioramento per non avere un `<TableCell>` componente e inline la relativa logica nel componente padre.

#### <a name="ensure-cascading-parameters-are-fixed"></a>Verificare che i parametri a catena siano corretti

Il `<CascadingValue>` componente ha un parametro facoltativo denominato `IsFixed` .

* Se il `IsFixed` valore è `false` (impostazione predefinita), ogni destinatario del valore a cascata imposta una sottoscrizione per la ricezione delle notifiche di modifica. In questo caso, ognuno `[CascadingParameter]` è **sostanzialmente più costoso** di un normale `[Parameter]` a causa del rilevamento della sottoscrizione.
* Se il `IsFixed` valore è `true` (ad esempio, `<CascadingValue Value="@someValue" IsFixed="true">` ), destinatari riceverà il valore iniziale ma *non* configurarà alcuna sottoscrizione per la ricezione degli aggiornamenti. In questo caso, ogni `[CascadingParameter]` è leggero e **non è più costoso** di un normale `[Parameter]` .

Quindi, laddove possibile, è consigliabile usare `IsFixed="true"` i valori a cascata. Questa operazione può essere eseguita ogni volta che il valore fornito non cambia nel tempo. Nel modello comune in cui un componente passa `this` come valore a catena, è necessario usare `IsFixed="true"` :

```razor
<CascadingValue Value="this" IsFixed="true">
    <SomeOtherComponents>
</CascadingValue>
```

Questo fa una grande differenza se è presente un numero elevato di altri componenti che ricevono il valore a cascata. Per altre informazioni, vedere <xref:blazor/components/cascading-values-and-parameters>.

#### <a name="avoid-attribute-splatting-with-captureunmatchedvalues"></a>Evitare l'attributo splatting con `CaptureUnmatchedValues`

I componenti possono scegliere di ricevere i valori di parametro "senza corrispondenza" utilizzando il <xref:Microsoft.AspNetCore.Components.ParameterAttribute.CaptureUnmatchedValues> flag:

```razor
<div @attributes="OtherAttributes">...</div>

@code {
    [Parameter(CaptureUnmatchedValues = true)]
    public IDictionary<string, object> OtherAttributes { get; set; }
}
```

Questo approccio consente di passare gli attributi aggiuntivi arbitrari all'elemento. Tuttavia, è anche piuttosto costoso perché il renderer deve:

* Corrisponde a tutti i parametri forniti rispetto al set di parametri noti per compilare un dizionario.
* Tenere traccia della sovrascrittura di più copie dello stesso attributo.

<xref:Microsoft.AspNetCore.Components.ParameterAttribute.CaptureUnmatchedValues>È possibile utilizzare su componenti non critici per le prestazioni, ad esempio quelli che non vengono ripetuti spesso. Tuttavia, per i componenti che eseguono il rendering su larga scala, ad esempio ogni elemento in un elenco di grandi dimensioni o celle di una griglia, provare a evitare l'attributo splatting.

Per altre informazioni, vedere <xref:blazor/components/index#attribute-splatting-and-arbitrary-parameters>.

#### <a name="implement-setparametersasync-manually"></a>Implementare `SetParametersAsync` manualmente

Uno degli aspetti principali del sovraccarico di rendering per componente consiste nel scrivere i valori dei parametri in ingresso nelle `[Parameter]` Proprietà. Per eseguire questa operazione, il renderer deve utilizzare la reflection. Anche se si tratta di un po' ottimizzato, l'assenza del supporto JIT nel runtime di webassembly impone limiti.

In alcuni casi estremi, può essere utile evitare la reflection e implementare manualmente la logica di impostazione dei parametri. Questo può essere applicabile nei casi seguenti:

* Si dispone di un componente che esegue il rendering molto spesso (ad esempio, sono presenti centinaia o migliaia di copie dell'interfaccia utente).
* Accetta molti parametri.
* Si noterà che l'overhead della ricezione dei parametri ha un impatto osservabile sulla velocità di risposta dell'interfaccia utente.

In questi casi, è possibile eseguire l'override del metodo virtuale del componente <xref:Microsoft.AspNetCore.Components.ComponentBase.SetParametersAsync%2A> e implementare la logica specifica del componente. Nell'esempio seguente vengono deliberatamente evitate le ricerche nel dizionario:

```razor
@code {
    [Parameter]
    public int MessageId { get; set; }

    [Parameter]
    public string Text { get; set; }

    [Parameter]
    public EventCallback<string> TextChanged { get; set; }

    [Parameter]
    public Theme CurrentTheme { get; set; }

    public override Task SetParametersAsync(ParameterView parameters)
    {
        foreach (var parameter in parameters)
        {
            switch (parameter.Name)
            {
                case nameof(MessageId):
                    MessageId = (int)parameter.Value;
                    break;
                case nameof(Text):
                    Text = (string)parameter.Value;
                    break;
                case nameof(TextChanged):
                    TextChanged = (EventCallback<string>)parameter.Value;
                    break;
                case nameof(CurrentTheme):
                    CurrentTheme = (Theme)parameter.Value;
                    break;
                default:
                    throw new ArgumentException($"Unknown parameter: {parameter.Name}");
            }
        }

        return base.SetParametersAsync(ParameterView.Empty);
    }
}
```

Nel codice precedente, la restituzione della classe base <xref:Microsoft.AspNetCore.Components.ComponentBase.SetParametersAsync%2A> esegue i normali metodi del ciclo di vita senza assegnare nuovamente parametri.

Come si può vedere nel codice precedente, l'override <xref:Microsoft.AspNetCore.Components.ComponentBase.SetParametersAsync%2A> e la fornitura di logica personalizzata sono complicate e laboriose, pertanto non è consigliabile usare questo approccio in generale. In casi estremi, può migliorare le prestazioni di rendering del 20-25%, ma è opportuno considerare questo approccio solo negli scenari elencati in precedenza.

### <a name="dont-trigger-events-too-rapidly"></a>Non attivare eventi troppo rapidamente

Alcuni eventi del browser vengono attivati molto spesso, ad esempio `onmousemove` e `onscroll` , che possono essere attivati decine o centinaia di volte al secondo. Nella maggior parte dei casi, non è necessario eseguire gli aggiornamenti dell'interfaccia utente in modo frequente. Se si tenta di eseguire questa operazione, si potrebbe danneggiare la velocità di risposta dell'interfaccia utente o utilizzare un tempo di CPU eccessivo.

Anziché usare eventi nativi `@onmousemove` o `@onscroll` , può essere preferibile usare l'interoperabilità js per registrare un callback che viene attivato meno di frequente. Ad esempio, il componente seguente ( `MyComponent.razor` ) Visualizza la posizione del mouse, ma solo gli aggiornamenti al massimo ogni 500 ms:

```razor
@inject IJSRuntime JS
@implements IDisposable

<h1>@message</h1>

<div @ref="myMouseMoveElement" style="border:1px dashed red;height:200px;">
    Move mouse here
</div>

@code {
    ElementReference myMouseMoveElement;
    DotNetObjectReference<MyComponent> selfReference;
    private string message = "Move the mouse in the box";

    [JSInvokable]
    public void HandleMouseMove(int x, int y)
    {
        message = $"Mouse move at {x}, {y}";
        StateHasChanged();
    }

    protected override async Task OnAfterRenderAsync(bool firstRender)
    {
        if (firstRender)
        {
            selfReference = DotNetObjectReference.Create(this);
            var minInterval = 500; // Only notify every 500 ms
            await JS.InvokeVoidAsync("onThrottledMouseMove", 
                myMouseMoveElement, selfReference, minInterval);
        }
    }

    public void Dispose() => selfReference?.Dispose();
}
```

Il codice JavaScript corrispondente, che può essere inserito nella `index.html` pagina o caricato come modulo ES6, registra il listener di eventi DOM effettivo. In questo esempio, il listener di eventi usa la [ `throttle` funzione di Lodash](https://lodash.com/docs/4.17.15#throttle) per limitare la frequenza delle chiamate:

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/lodash.js/4.17.20/lodash.min.js"></script>
<script>
  function onThrottledMouseMove(elem, component, interval) {
    elem.addEventListener('mousemove', _.throttle(e => {
      component.invokeMethodAsync('HandleMouseMove', e.offsetX, e.offsetY);
    }, interval));
  }
</script>
```

Questa tecnica può essere ancora più importante per Blazor Server , perché ogni chiamata di evento comporta il recapito di un messaggio tramite la rete. È utile per Blazor WebAssembly perché può ridurre notevolmente la quantità di lavoro di rendering.

## <a name="optimize-javascript-interop-speed"></a>Ottimizza la velocità di interoperabilità JavaScript

Le chiamate tra .NET e JavaScript comportano un sovraccarico aggiuntivo perché:

* Per impostazione predefinita, le chiamate sono asincrone.
* Per impostazione predefinita, i parametri e i valori restituiti sono serializzati in JSON. Questo consente di fornire un meccanismo di conversione di facile comprensione tra i tipi .NET e JavaScript.

Inoltre Blazor Server , queste chiamate vengono passate attraverso la rete.

### <a name="avoid-excessively-fine-grained-calls"></a>Evitare chiamate con granularità eccessiva

Poiché ogni chiamata implica un sovraccarico, può essere utile ridurre il numero di chiamate. Si consideri il codice seguente, in cui viene archiviata una raccolta di elementi nell'archivio del browser `localStorage` :

```csharp
private async Task StoreAllInLocalStorage(IEnumerable<TodoItem> items)
{
    foreach (var item in items)
    {
        await JS.InvokeVoidAsync("localStorage.setItem", item.Id, 
            JsonSerializer.Serialize(item));
    }
}
```

L'esempio precedente esegue una chiamata di interoperabilità JS separata per ogni elemento. Al contrario, l'approccio seguente riduce l'interoperabilità JS a una singola chiamata:

```csharp
private async Task StoreAllInLocalStorage(IEnumerable<TodoItem> items)
{
    await JS.InvokeVoidAsync("storeAllInLocalStorage", items);
}
```

La funzione JavaScript corrispondente definita nel modo seguente:

```javascript
function storeAllInLocalStorage(items) {
  items.forEach(item => {
    localStorage.setItem(item.id, JSON.stringify(item));
  });
}
```

Per Blazor WebAssembly , questo è in genere importante solo se si sta effettuando un numero elevato di chiamate di interoperabilità js.

### <a name="consider-making-synchronous-calls"></a>Prendere in considerazione l'esecuzione di chiamate sincrone

Per impostazione predefinita, le chiamate di interoperabilità JavaScript sono asincrone, indipendentemente dal fatto che il codice chiamato sia sincrono o asincrono. Ciò consente di garantire che i componenti siano compatibili con Blazor WebAssembly e Blazor Server . In Blazor Server , tutte le chiamate di interoperabilità JavaScript devono essere asincrone perché vengono inviate tramite una connessione di rete.

Se si è certi che l'app venga eseguita solo in Blazor WebAssembly , è possibile scegliere di eseguire chiamate di interoperabilità JavaScript sincrone. Questa operazione ha un sovraccarico leggermente inferiore rispetto all'esecuzione di chiamate asincrone e può comportare un minor numero di cicli di rendering perché non esiste alcuno stato intermedio durante l'attesa dei risultati.

Per eseguire una chiamata sincrona da .NET a JavaScript, eseguire il cast <xref:Microsoft.JSInterop.IJSRuntime> a <xref:Microsoft.JSInterop.IJSInProcessRuntime> :

```razor
@inject IJSRuntime JS

...

@code {
    protected override void HandleSomeEvent()
    {
        var jsInProcess = (IJSInProcessRuntime)JS;
        var value = jsInProcess.Invoke<string>("javascriptFunctionIdentifier");
    }
}
```

::: moniker range=">= aspnetcore-5.0"

Quando si utilizza `IJSObjectReference` , è possibile effettuare una chiamata sincrona eseguendo il cast a `IJSInProcessObjectReference` .

::: moniker-end

Per eseguire una chiamata sincrona da JavaScript a .NET, usare `DotNet.invokeMethod` anziché `DotNet.invokeMethodAsync` .

Le chiamate sincrone funzionano se:

* L'app è in esecuzione in Blazor WebAssembly , non Blazor Server .
* La funzione chiamata restituisce un valore in modo sincrono (non è un `async` metodo e non restituisce .NET <xref:System.Threading.Tasks.Task> o JavaScript `Promise` ).

Per altre informazioni, vedere <xref:blazor/call-javascript-from-dotnet>.

::: moniker range=">= aspnetcore-5.0"
 
### <a name="consider-making-unmarshalled-calls"></a>Prendere in considerazione l'esecuzione di chiamate non Marshall

Quando si esegue in Blazor WebAssembly , è possibile effettuare chiamate non Marshall da .NET a JavaScript. Si tratta di chiamate sincrone che non eseguono la serializzazione JSON di argomenti o valori restituiti. Tutti gli aspetti della gestione della memoria e delle traduzioni tra le rappresentazioni .NET e JavaScript vengono lasciati allo sviluppatore.

> [!WARNING]
> Quando `IJSUnmarshalledRuntime` si usa ha il minor sovraccarico degli approcci di interoperabilità js, le API JavaScript necessarie per interagire con queste API non sono attualmente documentate e sono soggette a modifiche di rilievo nelle versioni future.

```javascript
function jsInteropCall() {
    return BINDING.js_to_mono_obj("Hello world");
}
```

```razor
@inject IJSRuntime JS

@code {
    protected override void OnInitialized()
    {
        var unmarshalledJs = (IJSUnmarshalledRuntime)JS;
        var value = unmarshalledJs.InvokeUnmarshalled<string>("jsInteropCall");
    }
}
```

::: moniker-end

## <a name="minimize-app-download-size"></a>Ridurre le dimensioni del download dell'app

::: moniker range=">= aspnetcore-5.0"

### <a name="intermediate-language-il-trimming"></a>Troncamento del linguaggio intermedio (IL)

Il [trimming degli assembly inutilizzati da un' Blazor WebAssembly app](xref:blazor/host-and-deploy/configure-trimmer) consente di ridurre le dimensioni dell'app rimuovendo il codice inutilizzato nei file binari dell'app. Per impostazione predefinita, il trimmer viene eseguito durante la pubblicazione di un'applicazione. Per trarre vantaggio dal taglio, pubblicare l'app per la distribuzione usando il [`dotnet publish`](/dotnet/core/tools/dotnet-publish) comando con l'opzione [-c |--Configuration](/dotnet/core/tools/dotnet-publish#options) impostata su `Release` :

::: moniker-end

::: moniker range="< aspnetcore-5.0"

### <a name="intermediate-language-il-linking"></a>Collegamento del linguaggio intermedio (IL)

Il [collegamento di un' Blazor WebAssembly app](xref:blazor/host-and-deploy/configure-linker) consente di ridurre le dimensioni dell'app, ritagliando il codice inutilizzato nei file binari dell'app. Per impostazione predefinita, il linker linguaggio intermedio (IL) viene abilitato solo quando si compila nella `Release` configurazione. Per trarre vantaggio da questo problema, pubblicare l'app per la distribuzione usando il [`dotnet publish`](/dotnet/core/tools/dotnet-publish) comando con l'opzione [-c |--Configuration](/dotnet/core/tools/dotnet-publish#options) impostata su `Release` :

::: moniker-end

```dotnetcli
dotnet publish -c Release
```

### <a name="use-systemtextjson"></a>USA System.Text.Js

Blazorl'implementazione dell'interoperabilità JS di è basata su <xref:System.Text.Json> , ovvero una libreria di serializzazione JSON a prestazioni elevate con allocazione di memoria insufficiente. L'uso <xref:System.Text.Json> di non comporta dimensioni aggiuntive del payload dell'app rispetto all'aggiunta di una o più librerie JSON alternative.

Per informazioni aggiuntive sulla migrazione, vedere [come eseguire la migrazione da `Newtonsoft.Json` a `System.Text.Json` ](/dotnet/standard/serialization/system-text-json-migrate-from-newtonsoft-how-to).

### <a name="lazy-load-assemblies"></a>Assembly di caricamento lazy

Caricare gli assembly in fase di esecuzione quando gli assembly sono necessari per una route. Per altre informazioni, vedere <xref:blazor/webassembly-lazy-load-assemblies>.

### <a name="compression"></a>Compressione

Quando Blazor WebAssembly viene pubblicata un'app, l'output viene compresso in modo statico durante la pubblicazione per ridurre le dimensioni dell'app e rimuovere l'overhead per la compressione in fase di esecuzione. Blazor si basa sul server per eseguire il contenuto negotation e gestire file compressi in modo statico.

Dopo che un'app è stata distribuita, verificare che l'app servi i file compressi. Controllare la scheda rete nel Strumenti di sviluppo del browser e verificare che i file siano serviti con `Content-Encoding: br` o `Content-Encoding: gz` . Se l'host non serve file compressi, seguire le istruzioni riportate in <xref:blazor/host-and-deploy/webassembly#compression> .

### <a name="disable-unused-features"></a>Disabilitare le funzionalità inutilizzate

Blazor WebAssemblyil runtime di include le seguenti funzionalità .NET che possono essere disabilitate se l'app non le richiede per una dimensione di payload inferiore:

* Un file di dati è incluso per rendere corrette le informazioni sul fuso orario. Se l'app non richiede questa funzionalità, provare a disabilitarla impostando la `BlazorEnableTimeZoneSupport` Proprietà MSBuild nel file di progetto dell'app su `false` :

  ```xml
  <PropertyGroup>
    <BlazorEnableTimeZoneSupport>false</BlazorEnableTimeZoneSupport>
  </PropertyGroup>
  ```

::: moniker range=">= aspnetcore-5.0"

* Per impostazione predefinita, include Blazor WebAssembly le risorse di globalizzazione necessarie per visualizzare i valori, ad esempio le date e la valuta, nelle impostazioni cultura dell'utente. Se l'app non richiede la localizzazione, è possibile [configurare l'app in modo che supporti la lingua inglese](xref:blazor/globalization-localization), che è basata sulle `en-US` impostazioni cultura:

  ```xml
  <PropertyGroup>
    <InvariantGlobalization>true</InvariantGlobalization>
  </PropertyGroup>
  ```

::: moniker-end

::: moniker range="< aspnetcore-5.0"

* Le informazioni sulle regole di confronto sono incluse per fare in modo che le API <xref:System.StringComparison.InvariantCultureIgnoreCase?displayProperty=nameWithType> funzionino correttamente. Se si è certi che l'app non richiede i dati delle regole di confronto, provare a disabilitarla impostando la `BlazorWebAssemblyPreserveCollationData` Proprietà MSBuild nel file di progetto dell'app su `false` :

  ```xml
  <PropertyGroup>
    <BlazorWebAssemblyPreserveCollationData>false</BlazorWebAssemblyPreserveCollationData>
  </PropertyGroup>
  ```

::: moniker-end
