---
title: Creazione e utilizzo di Razor componenti ASP.NET Core
author: guardrex
description: Informazioni su come creare e usare Razor i componenti, tra cui la modalità di associazione ai dati, la gestione degli eventi e la gestione dei cicli di vita dei componenti.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 11/25/2020
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
uid: blazor/components/index
ms.openlocfilehash: 823c24620b369874fdbc3e314b5b08952df83c8b
ms.sourcegitcommit: 1166b0ff3828418559510c661e8240e5c5717bb7
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/12/2021
ms.locfileid: "100280111"
---
# <a name="create-and-use-aspnet-core-razor-components"></a>Creazione e utilizzo di Razor componenti ASP.NET Core

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/blazor/common/samples/) ([procedura per il download](xref:index#how-to-download-a-sample))

Blazor le app vengono compilate usando i *componenti*. Un componente è un blocco di interfaccia utente (UI) autonomo, ad esempio una pagina, una finestra di dialogo o un form. Un componente include il markup HTML e la logica di elaborazione necessaria per inserire i dati o rispondere agli eventi dell'interfaccia utente. I componenti sono flessibili e leggeri. Possono essere annidati, riutilizzati e condivisi tra i progetti.

## <a name="component-classes"></a>Classi di componenti

I componenti sono implementati in [Razor](xref:mvc/views/razor) file componente ( `.razor` ) utilizzando una combinazione di markup C# e HTML. Un componente in Blazor viene definito formalmente come *Razor componente*.

### <a name="razor-syntax"></a>Sintassi Razor

Razor i componenti nelle Blazor app utilizzano ampiamente la Razor sintassi. Se non si ha familiarità con il Razor linguaggio di markup, è consigliabile leggere il [ Razor riferimento alla sintassi per ASP.NET Core prima di](xref:mvc/views/razor) procedere.

Quando si accede al contenuto sulla Razor sintassi, prestare particolare attenzione alle sezioni seguenti:

* [Direttive](xref:mvc/views/razor#directives): `@` -parole chiave riservate con prefisso che in genere modificano il modo in cui il markup del componente viene analizzato o funzione.
* [Attributi di direttiva](xref:mvc/views/razor#directive-attributes): `@` parole chiave riservate con prefisso che in genere modificano il modo in cui gli elementi componente vengono analizzati o funzionano.

### <a name="names"></a>Nomi

Il nome di un componente deve iniziare con un carattere maiuscolo. Ad esempio, `MyCoolComponent.razor` è valido e `myCoolComponent.razor` non è valido.

### <a name="routing"></a>Routing

Il routing in Blazor viene effettuato fornendo un modello di route a ogni componente accessibile nell'app. Quando Razor viene compilato un file con una [`@page`][9] direttiva, alla classe generata viene assegnato un oggetto che <xref:Microsoft.AspNetCore.Mvc.RouteAttribute> specifica il modello di route. In fase di esecuzione, il router cerca le classi di componenti con un oggetto <xref:Microsoft.AspNetCore.Mvc.RouteAttribute> ed esegue il rendering di qualsiasi componente con un modello di route corrispondente all'URL richiesto. Per altre informazioni, vedere <xref:blazor/fundamentals/routing>.

```razor
@page "/ParentComponent"

...
```

### <a name="markup"></a>markup

L'interfaccia utente per un componente viene definita utilizzando HTML. La logica di rendering dinamica (ad esempio, cicli, condizionali, espressioni) viene aggiunta usando una sintassi C# incorporata denominata *Razor* . Quando viene compilata un'app, il markup HTML e la logica di rendering C# vengono convertiti in una classe Component. Il nome della classe generata corrisponde al nome del file.

I membri della classe Component sono definiti in un [`@code`][1] blocco. Nel [`@code`][1] blocco, lo stato del componente (proprietà, campi) viene specificato con i metodi per la gestione degli eventi o per la definizione di altre logiche dei componenti. È consentito più di un [`@code`][1] blocco.

I membri dei componenti possono essere usati come parte della logica di rendering del componente usando espressioni C# che iniziano con `@` . Ad esempio, viene eseguito il rendering di un campo C# anteponendo `@` il nome del campo. Nell'esempio seguente viene valutato ed eseguito il rendering:

* `headingFontStyle` al valore della proprietà CSS per `font-style` .
* `headingText` al contenuto dell' `<h1>` elemento.

```razor
<h1 style="font-style:@headingFontStyle">@headingText</h1>

@code {
    private string headingFontStyle = "italic";
    private string headingText = "Put on your new Blazor!";
}
```

Una volta eseguito il rendering iniziale del componente, il componente rigenera l'albero di rendering in risposta agli eventi. Blazor Confronta quindi il nuovo albero di rendering con quello precedente e applica le modifiche apportate al Document Object Model (DOM) del browser. Ulteriori dettagli sono disponibili in <xref:blazor/components/rendering> .

I componenti sono classi C# ordinarie e possono essere inseriti in qualsiasi punto all'interno di un progetto. I componenti che producono pagine Web in genere risiedono nella `Pages` cartella. I componenti non di pagina vengono spesso inseriti nella `Shared` cartella o in una cartella personalizzata aggiunta al progetto.

### <a name="namespaces"></a>Spazi dei nomi

Lo spazio dei nomi di un componente viene in genere derivato dallo spazio dei nomi radice dell'app e dal percorso (cartella) del componente all'interno dell'app. Se lo spazio dei nomi radice dell'app è `BlazorSample` e il componente si trova `Counter` nella `Pages` cartella:

* Lo `Counter` spazio dei nomi del componente è `BlazorSample.Pages` .
* Il nome completo del tipo del componente è `BlazorSample.Pages.Counter` .

Per le cartelle personalizzate che contengono componenti, aggiungere una [`@using`][2] direttiva al componente padre o al file dell'app `_Imports.razor` . L'esempio seguente rende disponibili i componenti nella `Components` cartella:

```razor
@using BlazorSample.Components
```

È anche possibile fare riferimento ai componenti usando i relativi nomi completi, che non richiedono la [`@using`][2] direttiva:

```razor
<BlazorSample.Components.MyComponent />
```

Lo spazio dei nomi di un componente creato con Razor è basato su (in ordine di priorità):

* [`@namespace`][8] designazione nel Razor markup file ( `.razor` ) `@namespace BlazorSample.MyNamespace` .
* Il progetto `RootNamespace` nel file di progetto ( `<RootNamespace>BlazorSample</RootNamespace>` ).
* Il nome del progetto, tratto dal nome file del file di progetto ( `.csproj` ) e il percorso dalla radice del progetto al componente. Il Framework, ad esempio, risolve `{PROJECT ROOT}/Pages/Index.razor` ( `BlazorSample.csproj` ) nello spazio dei nomi `BlazorSample.Pages` . I componenti seguono le regole di associazione dei nomi in C#. Per il `Index` componente in questo esempio, i componenti nell'ambito sono tutti componenti:
  * Nella stessa cartella `Pages` .
  * Componenti nella radice del progetto che non specificano in modo esplicito uno spazio dei nomi diverso.

> [!NOTE]
> La `global::` qualificazione non è supportata.
>
> L'importazione di componenti con [`using`](/dotnet/csharp/language-reference/keywords/using-statement) istruzioni con alias (ad esempio, `@using Foo = Bar` ) non è supportata.
>
> I nomi parzialmente qualificati non sono supportati. Ad esempio, `@using BlazorSample` l'aggiunta e il riferimento al `NavMenu` componente ( `NavMenu.razor` ) con `<Shared.NavMenu></Shared.NavMenu>` non è supportato.

### <a name="partial-class-support"></a>Supporto di classi parziali

Razor i componenti vengono generati come classi parziali. Razor i componenti vengono creati utilizzando uno degli approcci seguenti:

* Il codice C# è definito in un [`@code`][1] blocco con markup HTML e Razor codice in un unico file. Blazor i modelli definiscono i Razor componenti usando questo approccio.
* Il codice C# viene inserito in un file code-behind definito come classe parziale.

Nell'esempio seguente viene illustrato il `Counter` componente predefinito con un [`@code`][1] blocco in un'app generata da un Blazor modello. Il markup HTML, Razor il codice e il codice C# si trovano nello stesso file:

`Pages/Counter.razor`:

```razor
@page "/counter"

<h1>Counter</h1>

<p>Current count: @currentCount</p>

<button class="btn btn-primary" @onclick="IncrementCount">Click me</button>

@code {
    private int currentCount = 0;

    void IncrementCount()
    {
        currentCount++;
    }
}
```

Il `Counter` componente può essere creato anche usando un file code-behind con una classe parziale:

`Pages/Counter.razor`:

```razor
@page "/counter"

<h1>Counter</h1>

<p>Current count: @currentCount</p>

<button class="btn btn-primary" @onclick="IncrementCount">Click me</button>
```

`Counter.razor.cs`:

```csharp
namespace BlazorSample.Pages
{
    public partial class Counter
    {
        private int currentCount = 0;

        void IncrementCount()
        {
            currentCount++;
        }
    }
}
```

Aggiungere gli spazi dei nomi necessari al file di classe parziale in base alle esigenze. Gli spazi dei nomi tipici usati dai Razor componenti includono:

```csharp
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Components;
using Microsoft.AspNetCore.Components.Authorization;
using Microsoft.AspNetCore.Components.Forms;
using Microsoft.AspNetCore.Components.Routing;
using Microsoft.AspNetCore.Components.Web;
```

> [!IMPORTANT]
> [`@using`][2] le direttive nel `_Imports.razor` file vengono applicate solo ai Razor file ( `.razor` ) e non ai file C# ( `.cs` ).

### <a name="specify-a-base-class"></a>Specificare una classe di base

La [`@inherits`][6] direttiva può essere usata per specificare una classe di base per un componente. Nell'esempio seguente viene illustrato come un componente può ereditare una classe base, `BlazorRocksBase` , per fornire le proprietà e i metodi del componente. La classe base deve derivare da <xref:Microsoft.AspNetCore.Components.ComponentBase> .

`Pages/BlazorRocks.razor`:

```razor
@page "/BlazorRocks"
@inherits BlazorRocksBase

<h1>@BlazorRocksText</h1>
```

`BlazorRocksBase.cs`:

```csharp
using Microsoft.AspNetCore.Components;

namespace BlazorSample
{
    public class BlazorRocksBase : ComponentBase
    {
        public string BlazorRocksText { get; set; } = 
            "Blazor rocks the browser!";
    }
}
```

### <a name="use-components"></a>Usare i componenti

I componenti possono includere altri componenti dichiarando questi ultimi usando la sintassi degli elementi HTML. Il markup per l'uso di un componente è simile a un tag HTML, in cui il nome del tag è il tipo di componente.

Il markup seguente in `Pages/Index.razor` esegue il rendering di un' `HeadingComponent` istanza di:

```razor
<HeadingComponent />
```

`Components/HeadingComponent.razor`:

[!code-razor[](index/samples_snapshot/HeadingComponent.razor)]

Se un componente contiene un elemento HTML con una prima lettera maiuscola che non corrisponde a un nome di componente, viene emesso un avviso che indica che l'elemento ha un nome imprevisto. L'aggiunta di una [`@using`][2] direttiva per lo spazio dei nomi del componente rende disponibile il componente, che risolve l'avviso.

## <a name="parameters"></a>Parametri

### <a name="route-parameters"></a>Parametri di route

I componenti possono ricevere parametri di route dal modello di route fornito nella [`@page`][9] direttiva. Il router usa parametri di route per popolare i parametri del componente corrispondente.

::: moniker range=">= aspnetcore-5.0"

Sono supportati i parametri facoltativi. Nell'esempio seguente, il `text` parametro facoltativo assegna il valore del segmento di route alla proprietà del componente `Text` . Se il segmento non è presente, il valore di `Text` è impostato su `fantastic` .

`Pages/RouteParameter.razor`:

[!code-razor[](index/samples_snapshot/RouteParameter-5x.razor?highlight=1,6-7)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

`Pages/RouteParameter.razor`:

[!code-razor[](index/samples_snapshot/RouteParameter-3x.razor?highlight=2,7-8)]

I parametri facoltativi non sono supportati, quindi [`@page`][9] vengono applicate due direttive nell'esempio precedente. Il primo consente la navigazione al componente senza un parametro. La seconda [`@page`][9] direttiva riceve il `{text}` parametro di route e assegna il valore alla `Text` Proprietà.

::: moniker-end

Per informazioni sui parametri di route catch-all ( `{*pageRoute}` ), che acquisiscono percorsi tra più limiti di cartella, vedere <xref:blazor/fundamentals/routing#catch-all-route-parameters> .

### <a name="component-parameters"></a>Parametri del componente

I componenti possono avere *parametri del componente*, che vengono definiti usando proprietà semplici o complesse pubbliche nella classe Component con l' [ `[Parameter]` attributo](xref:Microsoft.AspNetCore.Components.ParameterAttribute). Usare gli attributi per specificare gli argomenti per un componente nel markup.

`Components/ChildComponent.razor`:

[!code-razor[](../common/samples/5.x/BlazorWebAssemblySample/Components/ChildComponent.razor?highlight=2,11-12)]

Ai parametri del componente è possibile assegnare un valore predefinito:

```csharp
[Parameter]
public string Title { get; set; } = "Panel Title from Child";
```

Nell'esempio seguente dall'app di esempio, `ParentComponent` imposta il valore della `Title` proprietà dell'oggetto `ChildComponent` .

`Pages/ParentComponent.razor`:

[!code-razor[](index/samples_snapshot/ParentComponent.razor?highlight=5-6)]

Per convenzione, un valore di attributo costituito da codice C# viene assegnato a un parametro usando [ Razor il `@` simbolo riservato](xref:mvc/views/razor#razor-syntax):

* Campo o proprietà padre: `Title="@{FIELD OR PROPERTY}` , dove il segnaposto `{FIELD OR PROPERTY}` è un campo o una proprietà C# del componente padre.
* Risultato di un metodo: `Title="@{METHOD}"` , dove il segnaposto `{METHOD}` è un metodo C# del componente padre.
* [Espressione implicita o esplicita](xref:mvc/views/razor#implicit-razor-expressions): `Title="@({EXPRESSION})"` , dove il segnaposto `{EXPRESSION}` è un'espressione C#.
  
Per ulteriori informazioni, vedere [ Razor riferimento alla sintassi per ASP.NET Core](xref:mvc/views/razor).

> [!WARNING]
> Non creare componenti che scrivono nei propri *parametri del componente*, usare invece un campo privato. Per ulteriori informazioni, vedere la sezione [parametri sovrascritti](#overwritten-parameters) .

## <a name="child-content"></a>Contenuto figlio

I componenti possono impostare il contenuto di un altro componente. Il componente di assegnazione fornisce il contenuto tra i tag che specificano il componente di destinazione.

Nell'esempio seguente, `ChildComponent` ha una `ChildContent` proprietà che rappresenta un oggetto <xref:Microsoft.AspNetCore.Components.RenderFragment> , che rappresenta un segmento di interfaccia utente di cui eseguire il rendering. Il valore di `ChildContent` viene posizionato nel markup del componente in cui deve essere eseguito il rendering del contenuto. Il valore di `ChildContent` viene ricevuto dal componente padre e ne viene eseguito il rendering all'interno del pannello bootstrap `panel-body` .

`Components/ChildComponent.razor`:

[!code-razor[](../common/samples/5.x/BlazorWebAssemblySample/Components/ChildComponent.razor?highlight=3,14-15)]

> [!NOTE]
> La proprietà che riceve il <xref:Microsoft.AspNetCore.Components.RenderFragment> contenuto deve essere denominata `ChildContent` per convenzione.

`ParentComponent`Nell'app di esempio può fornire contenuto per il rendering `ChildComponent` di inserendo il contenuto all'interno dei `<ChildComponent>` tag.

`Pages/ParentComponent.razor`:

[!code-razor[](index/samples_snapshot/ParentComponent.razor?highlight=7-8)]

A causa del modo in cui Blazor viene eseguito il rendering del contenuto figlio, i componenti di rendering all'interno di un `for` ciclo richiedono una variabile di indice locale se la variabile del ciclo di incremento viene utilizzata nel contenuto del componente figlio:
>
> ```razor
> @for (int c = 0; c < 10; c++)
> {
>     var current = c;
>     <ChildComponent Title="@c">
>         Child Content: Count: @current
>     </ChildComponent>
> }
> ```
>
> In alternativa, usare un `foreach` ciclo con <xref:System.Linq.Enumerable.Range%2A?displayProperty=nameWithType> :
>
> ```razor
> @foreach(var c in Enumerable.Range(0,10))
> {
>     <ChildComponent Title="@c">
>         Child Content: Count: @c
>     </ChildComponent>
> }
> ```

Per informazioni sul modo in <xref:Microsoft.AspNetCore.Components.RenderFragment> cui è possibile usare un modello per Razor l'interfaccia utente del componente, vedere gli articoli seguenti:

* <xref:blazor/components/templated-components>
* <xref:blazor/webassembly-performance-best-practices#define-reusable-renderfragments-in-code>

## <a name="attribute-splatting-and-arbitrary-parameters"></a>Attributo splatting e parametri arbitrari

I componenti possono acquisire ed eseguire il rendering di attributi aggiuntivi oltre ai parametri dichiarati del componente. È possibile acquisire attributi aggiuntivi in un dizionario e quindi *Splatted* su un elemento quando il componente viene sottoposto a rendering usando la [`@attributes`][3] Razor direttiva. Questo scenario è utile quando si definisce un componente che produce un elemento di markup che supporta un'ampia gamma di personalizzazioni. Ad esempio, può essere noioso definire gli attributi separatamente per un `<input>` che supporta molti parametri.

Nell'esempio seguente, il primo `<input>` elemento ( `id="useIndividualParams"` ) USA parametri dei singoli componenti, mentre il secondo `<input>` elemento ( `id="useAttributesDict"` ) usa l'attributo splatting:

```razor
<input id="useIndividualParams"
       maxlength="@maxlength"
       placeholder="@placeholder"
       required="@required"
       size="@size" />

<input id="useAttributesDict"
       @attributes="InputAttributes" />

@code {
    public string maxlength = "10";
    public string placeholder = "Input placeholder text";
    public string required = "required";
    public string size = "50";

    public Dictionary<string, object> InputAttributes { get; set; } =
        new Dictionary<string, object>()
        {
            { "maxlength", "10" },
            { "placeholder", "Input placeholder text" },
            { "required", "required" },
            { "size", "50" }
        };
}
```

Il tipo del parametro deve implementare `IEnumerable<KeyValuePair<string, object>>` o `IReadOnlyDictionary<string, object>` con chiavi di stringa.

Gli elementi sottoposti a rendering `<input>` usando entrambi gli approcci sono identici:

```html
<input id="useIndividualParams"
       maxlength="10"
       placeholder="Input placeholder text"
       required="required"
       size="50">

<input id="useAttributesDict"
       maxlength="10"
       placeholder="Input placeholder text"
       required="required"
       size="50">
```

Per accettare attributi arbitrari, definire un parametro component usando l' [ `[Parameter]` attributo](xref:Microsoft.AspNetCore.Components.ParameterAttribute) con la <xref:Microsoft.AspNetCore.Components.ParameterAttribute.CaptureUnmatchedValues> proprietà impostata su `true` :

```razor
@code {
    [Parameter(CaptureUnmatchedValues = true)]
    public Dictionary<string, object> InputAttributes { get; set; }
}
```

La <xref:Microsoft.AspNetCore.Components.ParameterAttribute.CaptureUnmatchedValues> proprietà su [`[Parameter]`](xref:Microsoft.AspNetCore.Components.ParameterAttribute) consente al parametro di trovare la corrispondenza con tutti gli attributi che non corrispondono ad altri parametri. Un componente può definire solo un singolo parametro con <xref:Microsoft.AspNetCore.Components.ParameterAttribute.CaptureUnmatchedValues> . Il tipo di proprietà utilizzato con <xref:Microsoft.AspNetCore.Components.ParameterAttribute.CaptureUnmatchedValues> deve essere assegnabile da `Dictionary<string, object>` con chiavi di stringa. `IEnumerable<KeyValuePair<string, object>>` o `IReadOnlyDictionary<string, object>` sono anche opzioni in questo scenario.

La posizione di [`@attributes`][3] relativa alla posizione degli attributi degli elementi è importante. Quando [`@attributes`][3] Splatted sull'elemento, gli attributi vengono elaborati da destra a sinistra (da ultimo a primo). Si consideri l'esempio seguente di un componente che utilizza un `Child` componente:

`ParentComponent.razor`:

```razor
<ChildComponent extra="10" />
```

`ChildComponent.razor`:

```razor
<div @attributes="AdditionalAttributes" extra="5" />

[Parameter(CaptureUnmatchedValues = true)]
public IDictionary<string, object> AdditionalAttributes { get; set; }
```

L' `Child` attributo del componente `extra` viene impostato a destra di [`@attributes`][3] . Il `Parent` rendering del componente `<div>` contiene `extra="5"` quando viene passato attraverso l'attributo aggiuntivo perché gli attributi vengono elaborati da destra a sinistra (da ultimo a primo):

```html
<div extra="5" />
```

Nell'esempio seguente, l'ordine di `extra` e [`@attributes`][3] viene invertito nell'oggetto del `Child` componente `<div>` :

`ParentComponent.razor`:

```razor
<ChildComponent extra="10" />
```

`ChildComponent.razor`:

```razor
<div extra="5" @attributes="AdditionalAttributes" />

[Parameter(CaptureUnmatchedValues = true)]
public IDictionary<string, object> AdditionalAttributes { get; set; }
```

Il rendering `<div>` nel `Parent` componente contiene `extra="10"` quando viene passato tramite l'attributo aggiuntivo:

```html
<div extra="10" />
```

## <a name="capture-references-to-components"></a>Acquisisci riferimenti ai componenti

I riferimenti ai componenti forniscono un modo per fare riferimento a un'istanza del componente in modo da poter emettere comandi per tale istanza, ad esempio `Show` o `Reset` . Per acquisire un riferimento a un componente:

* Aggiungere un [`@ref`][4] attributo al componente figlio.
* Definire un campo con lo stesso tipo del componente figlio.

```razor
<CustomLoginDialog @ref="loginDialog" ... />

@code {
    private CustomLoginDialog loginDialog;

    private void OnSomething()
    {
        loginDialog.Show();
    }
}
```

Quando viene eseguito il rendering del componente, il `loginDialog` campo viene popolato con l' `CustomLoginDialog` istanza del componente figlio. È quindi possibile richiamare i metodi .NET nell'istanza del componente.

> [!IMPORTANT]
> La `loginDialog` variabile viene popolata solo dopo il rendering del componente e l'output include `MyLoginDialog` l'elemento. Fino a quando non viene eseguito il rendering del componente, non vi sono riferimenti.
>
> Per modificare i riferimenti ai componenti dopo che il componente ha terminato il rendering, usare i [ `OnAfterRenderAsync` `OnAfterRender` metodi o](xref:blazor/components/lifecycle#after-component-render).
>
> Per usare una variabile di riferimento con un gestore eventi, usare un'espressione lambda o assegnare il delegato del gestore eventi nei [ `OnAfterRenderAsync` `OnAfterRender` metodi o](xref:blazor/components/lifecycle#after-component-render). In questo modo si garantisce che la variabile di riferimento venga assegnata prima dell'assegnazione del gestore eventi.
>
> ```razor
> <button type="button" 
>     @onclick="@(() => loginDialog.DoSomething())">Do Something</button>
>
> <MyLoginDialog @ref="loginDialog" ... />
>
> @code {
>     private MyLoginDialog loginDialog;
> }
> ```

Per fare riferimento ai componenti in un ciclo, vedere [Capture References to multiple analogous Child-Components (DotNet/aspnetcore #13358)](https://github.com/dotnet/aspnetcore/issues/13358).

Mentre l'acquisizione di riferimenti ai componenti usa una sintassi simile per l' [acquisizione di riferimenti a elementi](xref:blazor/call-javascript-from-dotnet#capture-references-to-elements), non è una funzionalità di interoperabilità di JavaScript. I riferimenti ai componenti non vengono passati al codice JavaScript. I riferimenti ai componenti vengono usati solo nel codice .NET.

> [!NOTE]
> **Non** usare i riferimenti ai componenti per mutare lo stato dei componenti figlio. Usare invece i normali parametri dichiarativi per passare i dati ai componenti figlio. L'utilizzo di normali parametri dichiarativi restituisce automaticamente i componenti figlio che eseguono il rendering alle ore corrette.

## <a name="synchronization-context"></a>Contesto di sincronizzazione

Blazor Usa un contesto di sincronizzazione ( <xref:System.Threading.SynchronizationContext> ) per applicare un singolo thread logico di esecuzione. I metodi del [ciclo](xref:blazor/components/lifecycle) di vita di un componente e tutti i callback di evento generati da Blazor vengono eseguiti nel contesto di sincronizzazione.

Blazor Serveril contesto di sincronizzazione tenta di emulare un ambiente a thread singolo in modo che corrisponda strettamente al modello webassembly nel browser, ovvero a thread singolo. In un determinato momento, il lavoro viene eseguito su un solo thread, con l'impressione di un singolo thread logico. Non vengono eseguite contemporaneamente due operazioni.

### <a name="avoid-thread-blocking-calls"></a>Evitare chiamate di blocco del thread

In genere, non chiamare i metodi seguenti. I metodi seguenti bloccano il thread e quindi impediscono all'app di riprendere il lavoro fino al completamento dell'oggetto sottostante <xref:System.Threading.Tasks.Task> :

* <xref:System.Threading.Tasks.Task%601.Result%2A>
* <xref:System.Threading.Tasks.Task.Wait%2A>
* <xref:System.Threading.Tasks.Task.WaitAny%2A>
* <xref:System.Threading.Tasks.Task.WaitAll%2A>
* <xref:System.Threading.Thread.Sleep%2A>
* <xref:System.Runtime.CompilerServices.TaskAwaiter.GetResult%2A>

### <a name="invoke-component-methods-externally-to-update-state"></a>Richiama i metodi del componente esternamente per aggiornare lo stato

Nel caso in cui un componente deve essere aggiornato in base a un evento esterno, ad esempio un timer o altre notifiche, usare il `InvokeAsync` metodo, che invia il Blazor contesto di sincronizzazione di. Si consideri ad esempio un *servizio Notifier* che può inviare notifiche a qualsiasi componente in ascolto dello stato aggiornato:

```csharp
public class NotifierService
{
    // Can be called from anywhere
    public async Task Update(string key, int value)
    {
        if (Notify != null)
        {
            await Notify.Invoke(key, value);
        }
    }

    public event Func<string, int, Task> Notify;
}
```

Registrare `NotifierService` :

* In Blazor WebAssembly registrare il servizio come singleton in `Program.Main` :

  ```csharp
  builder.Services.AddSingleton<NotifierService>();
  ```

* In Blazor Server registrare il servizio come ambito in `Startup.ConfigureServices` :

  ```csharp
  services.AddScoped<NotifierService>();
  ```

Usare `NotifierService` per aggiornare un componente:

```razor
@page "/"
@inject NotifierService Notifier
@implements IDisposable

<p>Last update: @lastNotification.key = @lastNotification.value</p>

@code {
    private (string key, int value) lastNotification;

    protected override void OnInitialized()
    {
        Notifier.Notify += OnNotify;
    }

    public async Task OnNotify(string key, int value)
    {
        await InvokeAsync(() =>
        {
            lastNotification = (key, value);
            StateHasChanged();
        });
    }

    public void Dispose()
    {
        Notifier.Notify -= OnNotify;
    }
}
```

Nell'esempio precedente:

* `NotifierService` richiama il metodo del componente `OnNotify` all'esterno del Blazor contesto di sincronizzazione di. `InvokeAsync` viene usato per passare al contesto corretto e accodare un rendering. Per altre informazioni, vedere <xref:blazor/components/rendering>.
* Il componente implementa <xref:System.IDisposable> e viene annullata la `OnNotify` sottoscrizione del delegato nel `Dispose` metodo, che viene chiamato dal framework quando il componente viene eliminato. Per altre informazioni, vedere <xref:blazor/components/lifecycle#component-disposal-with-idisposable>.

## <a name="use-key-to-control-the-preservation-of-elements-and-components"></a>Usare \@ la chiave per controllare la conservazione di elementi e componenti

Quando si esegue il rendering di un elenco di elementi o componenti e gli elementi o i componenti cambiano successivamente, l' Blazor algoritmo diffing deve decidere quali elementi o componenti precedenti possono essere conservati e come eseguire il mapping degli oggetti modello. In genere, questo processo è automatico e può essere ignorato, ma in alcuni casi potrebbe essere necessario controllare il processo.

Si consideri l'esempio seguente:

```csharp
@foreach (var person in People)
{
    <DetailsEditor Details="@person.Details" />
}

@code {
    [Parameter]
    public IEnumerable<Person> People { get; set; }
}
```

Il contenuto della `People` raccolta può cambiare con le voci inserite, eliminate o riordinate. Quando viene eseguito il rendering del componente, è `<DetailsEditor>` possibile che il componente venga modificato in modo da ricevere `Details` valori di parametro diversi. Questo può causare un rirendering più complesso del previsto. In alcuni casi, il rirendering può comportare differenze di comportamento visibili, ad esempio lo stato attivo degli elementi persi.

È possibile controllare il processo di mapping con l' [`@key`][5] attributo della direttiva. [`@key`][5] fa in modo che l'algoritmo di diffing garantisca la conservazione di elementi o componenti in base al valore della chiave:

```csharp
@foreach (var person in People)
{
    <DetailsEditor @key="person" Details="@person.Details" />
}

@code {
    [Parameter]
    public IEnumerable<Person> People { get; set; }
}
```

Quando la `People` raccolta viene modificata, l'algoritmo diffing mantiene l'associazione tra `<DetailsEditor>` istanze e `person` istanze:

* Se un oggetto `Person` viene eliminato dall' `People` elenco, solo l' `<DetailsEditor>` istanza corrispondente viene rimossa dall'interfaccia utente. Altre istanze rimangono invariate.
* Se un oggetto viene inserito in una determinata `Person` posizione nell'elenco, viene inserita una nuova istanza in corrispondenza della `<DetailsEditor>` posizione corrispondente. Altre istanze rimangono invariate.
* Se `Person` le voci vengono riordinate, le `<DetailsEditor>` istanze corrispondenti vengono mantenute e nuovamente ordinate nell'interfaccia utente.

In alcuni scenari, l'utilizzo di [`@key`][5] riduce al minimo la complessità del rendering ed evita potenziali problemi con le parti con stato del DOM che cambiano, ad esempio la posizione dello stato attivo.

> [!IMPORTANT]
> Le chiavi sono locali per ogni elemento contenitore o componente. Le chiavi non vengono confrontate globalmente nell'intero documento.

### <a name="when-to-use-key"></a>Quando usare la \@ chiave

In genere, è opportuno usare [`@key`][5] ogni volta che viene eseguito il rendering di un elenco (ad esempio in un blocco [foreach](/dotnet/csharp/language-reference/keywords/foreach-in) ) e un valore appropriato per definire [`@key`][5] .

È anche possibile usare [`@key`][5] per impedire a Blazor di mantenere un sottoalbero di un elemento o di un componente quando un oggetto viene modificato:

```razor
<div @key="currentPerson">
    ... content that depends on currentPerson ...
</div>
```

Se `@currentPerson` viene modificata, la [`@key`][5] direttiva attribute impone Blazor di rimuovere l'intero oggetto `<div>` e i relativi discendenti e di ricompilare il sottoalbero all'interno dell'interfaccia utente con nuovi elementi e componenti. Questo può essere utile se è necessario garantire che lo stato dell'interfaccia utente non venga mantenuto quando viene `@currentPerson` modificato.

### <a name="when-not-to-use-key"></a>Quando non usare la \@ chiave

Si verifica un costo in termini di prestazioni quando si verificano differenze con [`@key`][5] . Il costo delle prestazioni non è elevato, ma è sufficiente specificare solo [`@key`][5] se il controllo delle regole di conservazione degli elementi o dei componenti avvantaggia l'app.

Anche se [`@key`][5] non viene usato, Blazor conserva le istanze di elementi e componenti figlio il più possibile. L'unico vantaggio dell'utilizzo di [`@key`][5] è il controllo sulla *modalità* di mapping delle istanze del modello alle istanze dei componenti mantenute, anziché sull'algoritmo di diffing che seleziona il mapping.

### <a name="what-values-to-use-for-key"></a>Valori da usare per la \@ chiave

In genere, è opportuno fornire uno dei seguenti tipi di valore per [`@key`][5] :

* Istanze di oggetti modello (ad esempio, un' `Person` istanza come nell'esempio precedente). In questo modo si garantisce la conservazione in base all'uguaglianza del riferimento all'oggetto.
* Identificatori univoci, ad esempio valori di chiave primaria di tipo `int` , `string` o `Guid` .

Verificare che i valori usati per non siano in [`@key`][5] conflitto. Se vengono rilevati valori di conflitto nello stesso elemento padre, Blazor genera un'eccezione perché non è in grado di eseguire il mapping deterministico di elementi o componenti precedenti a nuovi elementi o componenti. Utilizzare solo valori distinti, ad esempio istanze di oggetti o valori di chiave primaria.

## <a name="overwritten-parameters"></a>Parametri sovrascritti

Il Blazor Framework impone in genere un'assegnazione di parametro sicura da padre a figlio:

* I parametri non vengono sovrascritti in modo imprevisto.
* Gli effetti collaterali sono ridotti al minimo. Ad esempio, i rendering aggiuntivi vengono evitati perché possono creare cicli di rendering infiniti.

Un componente figlio riceve nuovi valori di parametro che potrebbero sovrascrivere i valori esistenti quando il componente padre esegue nuovamente il rendering. Accidentale sovrascrivendo i valori dei parametri in un componente figlio spesso si verifica quando si sviluppa il componente con uno o più parametri associati a dati e lo sviluppatore scrive direttamente in un parametro nell'elemento figlio:

* Il componente figlio viene sottoposto a rendering con uno o più valori di parametro dal componente padre.
* Il figlio scrive direttamente nel valore di un parametro.
* Il componente padre esegue nuovamente il rendering e sovrascrive il valore del parametro del figlio.

La possibilità di sovrascrivere i valori del parametro si estende anche nei setter di proprietà del componente figlio.

**Le linee guida generali non consentono di creare componenti che scrivono direttamente nei propri parametri.**

Si consideri il `Expander` componente difettoso seguente che:

* Esegue il rendering del contenuto figlio.
* Consente di visualizzare o disabilitare il contenuto figlio con un parametro component ( `Expanded` ).
* Il componente scrive direttamente nel `Expanded` parametro, che illustra il problema con i parametri sovrascritti e deve essere evitato.

```razor
<div @onclick="Toggle" class="card bg-light mb-3" style="width:30rem">
    <div class="card-body">
        <h2 class="card-title">Toggle (<code>Expanded</code> = @Expanded)</h2>

        @if (Expanded)
        {
            <p class="card-text">@ChildContent</p>
        }
    </div>
</div>

@code {
    [Parameter]
    public bool Expanded { get; set; }

    [Parameter]
    public RenderFragment ChildContent { get; set; }

    private void Toggle()
    {
        Expanded = !Expanded;
    }
}
```

Il `Expander` componente viene aggiunto a un componente padre che può chiamare <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> :

```razor
@page "/expander"

<Expander Expanded="true">
    Expander 1 content
</Expander>

<Expander Expanded="true" />

<button @onclick="StateHasChanged">
    Call StateHasChanged
</button>
```

Inizialmente, i `Expander` componenti si comportano in modo indipendente quando le proprietà vengono disposte `Expanded` . I componenti figlio gestiscono gli Stati come previsto. Quando <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> viene chiamato nell'elemento padre, il `Expanded` parametro del primo componente figlio viene reimpostato sul valore iniziale ( `true` ). Il valore del secondo `Expander` componente `Expanded` non viene reimpostato perché nel secondo componente non viene eseguito il rendering del contenuto figlio.

Per mantenere lo stato nello scenario precedente, usare un *campo privato* nel `Expander` componente per mantenerne lo stato attivato/disattivato.

Il componente modificato seguente `Expander` :

* Accetta il `Expanded` valore del parametro component dall'elemento padre.
* Assegna il valore del parametro component a un *campo privato* ( `expanded` ) nell' [evento OnInitialized](xref:blazor/components/lifecycle#component-initialization-methods).
* Usa il campo privato per mantenere lo stato di attivazione/disattivo interno, che dimostra come evitare la scrittura diretta in un parametro.

```razor
<div @onclick="Toggle" class="card bg-light mb-3" style="width:30rem">
    <div class="card-body">
        <h2 class="card-title">Toggle (<code>expanded</code> = @expanded)</h2>

        @if (expanded)
        {
            <p class="card-text">@ChildContent</p>
        }
    </div>
</div>

@code {
    private bool expanded;

    [Parameter]
    public bool Expanded { get; set; }

    [Parameter]
    public RenderFragment ChildContent { get; set; }

    protected override void OnInitialized()
    {
        expanded = Expanded;
    }

    private void Toggle()
    {
        expanded = !expanded;
    }
}
```

Per ulteriori informazioni, vedere [ Blazor errore di associazione bidirezionale (DotNet/aspnetcore #24599)](https://github.com/dotnet/aspnetcore/issues/24599). 

## <a name="apply-an-attribute"></a>Applicare un attributo

Gli attributi possono essere applicati ai Razor componenti con la [`@attribute`][7] direttiva. Nell'esempio seguente viene applicato l' [ `[Authorize]` attributo](xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute) alla classe Component:

```razor
@page "/"
@attribute [Authorize]
```

## <a name="conditional-html-element-attributes"></a>Attributi dell'elemento HTML condizionale

Gli attributi degli elementi HTML vengono sottoposti a rendering in modo condizionale in base al valore .NET. Se il valore è `false` o `null` , l'attributo non viene sottoposto a rendering. Se il valore è `true` , l'attributo viene sottoposto a rendering ridotto a icona.

Nell'esempio seguente, `IsCompleted` determina se `checked` viene eseguito il rendering nel markup dell'elemento:

```razor
<input type="checkbox" checked="@IsCompleted" />

@code {
    [Parameter]
    public bool IsCompleted { get; set; }
}
```

Se `IsCompleted` è `true` , viene eseguito il rendering della casella di controllo come segue:

```html
<input type="checkbox" checked />
```

Se `IsCompleted` è `false` , viene eseguito il rendering della casella di controllo come segue:

```html
<input type="checkbox" />
```

Per ulteriori informazioni, vedere [ Razor riferimento alla sintassi per ASP.NET Core](xref:mvc/views/razor).

> [!WARNING]
> Alcuni attributi HTML, ad esempio [`aria-pressed`](https://developer.mozilla.org/docs/Web/Accessibility/ARIA/Roles/button_role#Toggle_buttons) , non funzionano correttamente quando il tipo .NET è un `bool` . In questi casi, usare un `string` tipo anziché un `bool` .

## <a name="raw-html"></a>HTML non elaborato

Le stringhe vengono in genere sottoposte a rendering usando i nodi di testo DOM, il che significa che qualsiasi markup che può contenere viene ignorato e considerato come testo letterale. Per eseguire il rendering del codice HTML non elaborato, eseguire il wrapping del contenuto HTML in un `MarkupString` valore. Il valore viene analizzato in formato HTML o SVG e inserito nel DOM.

> [!WARNING]
> Il rendering di codice HTML non elaborato costruito da un'origine non attendibile costituisce un rischio per la **sicurezza** e deve essere evitato.

Nell'esempio seguente viene illustrato l'utilizzo del `MarkupString` tipo per aggiungere un blocco di contenuto HTML statico all'output sottoposto a rendering di un componente:

```html
@((MarkupString)myMarkup)

@code {
    private string myMarkup = 
        "<p class='markup'>This is a <em>markup string</em>.</p>";
}
```

## <a name="razor-templates"></a>Razor modelli

I frammenti di rendering possono essere definiti usando la Razor sintassi del modello. Razor i modelli sono un modo per definire un frammento di interfaccia utente e presupporre il formato seguente:

```razor
@<{HTML tag}>...</{HTML tag}>
```

Nell'esempio seguente viene illustrato come specificare <xref:Microsoft.AspNetCore.Components.RenderFragment> <xref:Microsoft.AspNetCore.Components.RenderFragment%601> i valori e ed eseguire il rendering dei modelli direttamente in un componente. I frammenti di rendering possono anche essere passati come argomenti ai [componenti basati su modelli](xref:blazor/components/templated-components).

```razor
@timeTemplate

@petTemplate(new Pet { Name = "Rex" })

@code {
    private RenderFragment timeTemplate = @<p>The time is @DateTime.Now.</p>;
    private RenderFragment<Pet> petTemplate = (pet) => @<p>Pet: @pet.Name</p>;

    private class Pet
    {
        public string Name { get; set; }
    }
}
```

Output del codice precedente sottoposto a rendering:

```html
<p>The time is 10/04/2018 01:26:52.</p>

<p>Pet: Rex</p>
```

## <a name="static-assets"></a>Asset statici

Blazorsegue la convenzione di ASP.NET Core app che collocano asset statici nella [ `web root (wwwroot)` cartella](xref:fundamentals/index#web-root)del progetto.

Usare un percorso relativo di base ( `/` ) per fare riferimento alla radice Web per un asset statico. Nell'esempio seguente, `logo.png` si trova fisicamente nella `{PROJECT ROOT}/wwwroot/images` cartella:

```razor
<img alt="Company logo" src="/images/logo.png" />
```

Razor i componenti **non** supportano la notazione tilde-barra ( `~/` ).

Per informazioni sull'impostazione del percorso di base di un'app, vedere <xref:blazor/host-and-deploy/index#app-base-path> .

## <a name="tag-helpers-arent-supported-in-components"></a>Gli helper tag non sono supportati nei componenti

[`Tag Helpers`](xref:mvc/views/tag-helpers/intro) non sono supportati in Razor componenti ( `.razor` file). Per fornire funzionalità di tipo Helper tag in Blazor , creare un componente con la stessa funzionalità dell'helper tag e utilizzare il componente.

## <a name="scalable-vector-graphics-svg-images"></a>Immagini SVG (Scalable Vector Graphics)

Poiché Blazor esegue il rendering di HTML, le immagini supportate dal browser, incluse le immagini SVG (Scalable Vector Graphics) `.svg` , sono supportate tramite il `<img>` Tag:

```html
<img alt="Example image" src="some-image.svg" />
```

Analogamente, le immagini SVG sono supportate nelle regole CSS di un file di foglio di stile ( `.css` ):

```css
.my-element {
    background-image: url("some-image.svg");
}
```

Tuttavia, il markup SVG inline non è supportato in tutti gli scenari. Se si inserisce un `<svg>` tag direttamente in un file di componente ( `.razor` ), il rendering delle immagini di base è supportato, ma molti scenari avanzati non sono ancora supportati. Ad esempio, i `<use>` tag non sono attualmente rispettati e [`@bind`][10] non possono essere usati con alcuni tag svg. Per ulteriori informazioni, vedere [supporto per SVG in Blazor (#18271 DotNet/aspnetcore)](https://github.com/dotnet/aspnetcore/issues/18271).

## <a name="whitespace-rendering-behavior"></a>Comportamento di rendering degli spazi vuoti

::: moniker range=">= aspnetcore-5.0"

[`@preservewhitespace`](xref:mvc/views/razor#preservewhitespace)Se la direttiva non viene utilizzata con un valore di `true` , per impostazione predefinita viene rimosso uno spazio vuoto aggiuntivo se:

* Iniziali o finali all'interno di un elemento.
* Iniziali o finali in un `RenderFragment` parametro. Ad esempio, il contenuto figlio viene passato a un altro componente.
* Precede o segue un blocco di codice C#, ad esempio `@if` o `@foreach` .

La rimozione degli spazi vuoti può influire sull'output sottoposto a rendering quando si usa una regola CSS, ad esempio `white-space: pre` . Per disabilitare questa ottimizzazione delle prestazioni e mantenere lo spazio vuoto, effettuare una delle operazioni seguenti:

* Aggiungere la `@preservewhitespace true` direttiva all'inizio del `.razor` file per applicare la preferenza a un componente specifico.
* Aggiungere la `@preservewhitespace true` direttiva all'interno `_Imports.razor` di un file per applicare la preferenza a un'intera sottodirectory o all'intero progetto.

Nella maggior parte dei casi, non è necessaria alcuna azione, perché le app in genere continuano a funzionare normalmente (ma più velocemente). Se la rimozione dello spazio vuoto causa problemi per un particolare componente, utilizzare `@preservewhitespace true` in tale componente per disabilitare questa ottimizzazione.

::: moniker-end

::: moniker range="< aspnetcore-5.0"

Lo spazio vuoto viene mantenuto nel codice sorgente di un componente. Viene eseguito il rendering del testo con solo spazi vuoti nell'Document Object Model del browser (DOM) anche quando non è presente alcun effetto visivo.

Si consideri il Razor codice componente seguente:

```razor
<ul>
    @foreach (var item in Items)
    {
        <li>
            @item.Text
        </li>
    }
</ul>
```

Nell'esempio precedente viene eseguito il rendering degli spazi vuoti non necessari seguenti:

* All'esterno del `@foreach` blocco di codice.
* Intorno all' `<li>` elemento.
* Intorno all' `@item.Text` output.

Un elenco contenente 100 elementi restituisce 402 aree di spazio vuoto e nessuno degli spazi vuoti influiscono visivamente sull'output sottoposto a rendering.

Quando si esegue il rendering HTML statico per i componenti, gli spazi vuoti all'interno di un tag non vengono conservati Ad esempio, visualizzare l'origine del componente seguente nell'output di cui è stato eseguito il rendering:

```razor
<img     alt="My image"   src="img.png"     />
```

Lo spazio vuoto non è mantenuto dal Razor markup precedente:

```razor
<img alt="My image" src="img.png" />
```

::: moniker-end

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:blazor/security/server/threat-mitigation>: Include informazioni aggiuntive sulla creazione di Blazor Server app che devono essere confrontate con l'esaurimento delle risorse.

<!--Reference links in article-->
[1]: <xref:mvc/views/razor#code> "riferimento alla sintassi::: NO-LOC (Razor)::: per ASP.NET Core"
[2]: <xref:mvc/views/razor#using> "riferimento alla sintassi::: NO-LOC (Razor)::: per ASP.NET Core"
[3]: <xref:mvc/views/razor#attributes> "riferimento alla sintassi::: NO-LOC (Razor)::: per ASP.NET Core"
[4]: <xref:mvc/views/razor#ref> "riferimento alla sintassi::: NO-LOC (Razor)::: per ASP.NET Core"
[5]: <xref:mvc/views/razor#key> "riferimento alla sintassi::: NO-LOC (Razor)::: per ASP.NET Core"
[6]: <xref:mvc/views/razor#inherits> "riferimento alla sintassi::: NO-LOC (Razor)::: per ASP.NET Core"
[7]: <xref:mvc/views/razor#attribute> "riferimento alla sintassi::: NO-LOC (Razor)::: per ASP.NET Core"
[8]: <xref:mvc/views/razor#namespace> "riferimento alla sintassi::: NO-LOC (Razor)::: per ASP.NET Core"
[9]: <xref:mvc/views/razor#page> "riferimento alla sintassi::: NO-LOC (Razor)::: per ASP.NET Core"
[10]: <xref:mvc/views/razor#bind> "riferimento alla sintassi::: NO-LOC (Razor)::: per ASP.NET Core"
