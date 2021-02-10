---
title: Gestione degli errori nelle Blazor app ASP.NET Core
author: guardrex
description: Scopri in che modo ASP.NET Core Blazor come Blazor gestisce le eccezioni non gestite e come sviluppare app che rilevano e gestiscono gli errori.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 04/23/2020
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
uid: blazor/fundamentals/handle-errors
ms.openlocfilehash: cb3c64ab7340a67a6730d98af8a91c4e9837acf1
ms.sourcegitcommit: 04ad9cd26fcaa8bd11e261d3661f375f5f343cdc
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/10/2021
ms.locfileid: "100106947"
---
# <a name="handle-errors-in-aspnet-core-blazor-apps"></a>Gestione degli errori nelle Blazor app ASP.NET Core

Di [Steve Sanderson](https://github.com/SteveSandersonMS)

Questo articolo descrive come Blazor gestire le eccezioni non gestite e come sviluppare app che rilevano e gestiscono gli errori.

## <a name="detailed-errors-during-development"></a>Errori dettagliati durante lo sviluppo

Quando un' Blazor app non funziona correttamente durante lo sviluppo, la ricezione di informazioni dettagliate sugli errori dall'app è utile per la risoluzione dei problemi e per risolvere il problema. Quando si verifica un errore, le Blazor app visualizzano una barra dorata nella parte inferiore della schermata:

* Durante lo sviluppo, la barra dorata indirizza l'utente alla console del browser, in cui è possibile visualizzare l'eccezione.
* In produzione, la barra dorata informa l'utente che si è verificato un errore e consiglia di aggiornare il browser.

L'interfaccia utente per questa esperienza di gestione degli errori fa parte dei Blazor modelli di progetto.

In un' Blazor WebAssembly app, personalizzare l'esperienza nel `wwwroot/index.html` file:

```html
<div id="blazor-error-ui">
    An unhandled error has occurred.
    <a href="" class="reload">Reload</a>
    <a class="dismiss">🗙</a>
</div>
```

In un' Blazor Server app, personalizzare l'esperienza nel `Pages/_Host.cshtml` file:

```cshtml
<div id="blazor-error-ui">
    <environment include="Staging,Production">
        An error has occurred. This application may no longer respond until reloaded.
    </environment>
    <environment include="Development">
        An unhandled exception has occurred. See browser dev tools for details.
    </environment>
    <a href="" class="reload">Reload</a>
    <a class="dismiss">🗙</a>
</div>
```

L' `blazor-error-ui` elemento è nascosto dagli stili inclusi nei Blazor modelli ( `wwwroot/css/app.css` o `wwwroot/css/site.css` ) e quindi mostrati quando si verifica un errore:

```css
#blazor-error-ui {
    background: lightyellow;
    bottom: 0;
    box-shadow: 0 -1px 2px rgba(0, 0, 0, 0.2);
    display: none;
    left: 0;
    padding: 0.6rem 1.25rem 0.7rem 1.25rem;
    position: fixed;
    width: 100%;
    z-index: 1000;
}

#blazor-error-ui .dismiss {
    cursor: pointer;
    position: absolute;
    right: 0.75rem;
    top: 0.5rem;
}
```

## <a name="blazor-server-detailed-circuit-errors"></a>Blazor Server errori dettagliati del circuito

Gli errori sul lato client non includono il stack e non forniscono dettagli sulla ragione dell'errore, ma i log del server contengono tali informazioni. Ai fini dello sviluppo, le informazioni sugli errori dei circuiti sensibili possono essere rese disponibili per il client abilitando errori dettagliati.

Per abilitare Blazor Server errori dettagliati, usare gli approcci seguenti:

* <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions.DetailedErrors?displayProperty=nameWithType>.
* `DetailedErrors`Chiave di configurazione impostata su `true` , che può essere impostata nel file delle impostazioni di sviluppo dell'app ( `appsettings.Development.json` ). La chiave può essere impostata anche usando la `ASPNETCORE_DETAILEDERRORS` variabile di ambiente con un valore `true` .
* la [ SignalR registrazione lato server](xref:signalr/diagnostics#server-side-logging) ( `Microsoft.AspNetCore.SignalR` ) può essere impostata su [debug](xref:Microsoft.Extensions.Logging.LogLevel) o [traccia](xref:Microsoft.Extensions.Logging.LogLevel) per la SignalR registrazione dettagliata.

`appsettings.Development.json`:

```json
{
  "DetailedErrors": true,
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "Microsoft.Hosting.Lifetime": "Information",
      "Microsoft.AspNetCore.SignalR": "Debug"
    }
  }
}
```

> [!WARNING]
> L'esposizione delle informazioni sugli errori ai client su Internet costituisce un rischio per la sicurezza che deve essere sempre evitata.

## <a name="how-a-blazor-server-app-reacts-to-unhandled-exceptions"></a>Blazor ServerReazione di un'app alle eccezioni non gestite

Blazor Server è un Framework con stato. Mentre gli utenti interagiscono con un'app, mantengono una connessione al server noto come *circuito*. Il circuito include istanze di componenti attive, oltre a molti altri aspetti dello stato, ad esempio:

* Output più recente sottoposto a rendering dei componenti.
* Set corrente di delegati di gestione degli eventi che possono essere attivati da eventi lato client.

Se un utente apre l'app in più schede del browser, avrà più circuiti indipendenti.

Blazor Considera le eccezioni non gestite come irreversibili per il circuito in cui si verificano. Se un circuito viene terminato a causa di un'eccezione non gestita, l'utente può continuare a interagire con l'app ricaricando la pagina per creare un nuovo circuito. I circuiti al di fuori di quello terminato, ovvero circuiti per altri utenti o altre schede del browser, non sono interessati. Questo scenario è simile a un'applicazione desktop che si arresta in modo anomalo. L'app arrestata in modo anomalo deve essere riavviata, ma non sono interessate altre app.

Un circuito viene terminato quando si verifica un'eccezione non gestita per i motivi seguenti:

* Un'eccezione non gestita spesso lascia il circuito in uno stato non definito.
* L'operazione normale dell'app non può essere garantita dopo un'eccezione non gestita.
* Le vulnerabilità di sicurezza possono essere visualizzate nell'app se il circuito continua.

## <a name="manage-unhandled-exceptions-in-developer-code"></a>Gestire le eccezioni non gestite nel codice dello sviluppatore

Per continuare l'applicazione dopo un errore, l'app deve avere la logica di gestione degli errori. Le sezioni successive di questo articolo descrivono le potenziali fonti di eccezioni non gestite.

In produzione, non eseguire il rendering dei messaggi di eccezione del Framework o delle analisi dello stack nell'interfaccia utente. Il rendering dei messaggi di eccezione o delle analisi dello stack potrebbe:

* Divulgare informazioni riservate agli utenti finali.
* Aiutare un utente malintenzionato a individuare i punti deboli in un'app che può compromettere la sicurezza dell'app, del server o della rete.

## <a name="log-errors-with-a-persistent-provider"></a>Registrare gli errori con un provider persistente

Se si verifica un'eccezione non gestita, l'eccezione viene registrata <xref:Microsoft.Extensions.Logging.ILogger> nelle istanze configurate nel contenitore dei servizi. Per impostazione predefinita, Blazor le app registrano nell'output della console con il provider di registrazione della console. Prendere in considerazione la registrazione a una posizione più permanente con un provider che gestisce le dimensioni del log e la rotazione del log. Per altre informazioni, vedere <xref:fundamentals/logging/index>.

Durante lo sviluppo, Blazor in genere invia i dettagli completi delle eccezioni alla console del browser per facilitare il debug. In produzione, gli errori dettagliati nella console del browser sono disabilitati per impostazione predefinita, il che significa che gli errori non vengono inviati ai client, ma i dettagli completi dell'eccezione sono ancora registrati sul lato server. Per altre informazioni, vedere <xref:fundamentals/error-handling>.

È necessario decidere quali eventi imprevisti registrare e il livello di gravità degli eventi imprevisti registrati. Gli utenti ostili potrebbero essere in grado di attivare intenzionalmente gli errori. Ad esempio, non registrare un evento imprevisto da un errore in cui viene fornito un oggetto sconosciuto `ProductId` nell'URL di un componente che Visualizza i dettagli del prodotto. Non tutti gli errori devono essere considerati come eventi imprevisti con gravità elevata per la registrazione.

Per altre informazioni, vedere <xref:blazor/fundamentals/logging>.

## <a name="places-where-errors-may-occur"></a>Posizioni in cui possono verificarsi errori

Il Framework e il codice dell'app possono attivare eccezioni non gestite in uno dei percorsi seguenti:

* [Creazione di istanze di componenti](#component-instantiation)
* [Metodi del ciclo di vita](#lifecycle-methods)
* [Logica di rendering](#rendering-logic)
* [Gestori eventi](#event-handlers)
* [Eliminazione componenti](#component-disposal)
* [Interoperabilità JavaScript](#javascript-interop)
* [Blazor Server esecuzione del rendering](#blazor-server-prerendering)

Le eccezioni non gestite precedenti sono descritte nelle sezioni seguenti di questo articolo.

### <a name="component-instantiation"></a>Creazione di istanze di componenti

Quando Blazor Crea un'istanza di un componente:

* Il costruttore del componente viene richiamato.
* Vengono richiamati i costruttori di tutti i servizi non singleton forniti al costruttore del componente tramite la [`@inject`](xref:mvc/views/razor#inject) direttiva o l' [ `[Inject]` attributo](xref:blazor/fundamentals/dependency-injection#request-a-service-in-a-component) .

Un Blazor Server circuito ha esito negativo quando un costruttore eseguito o un setter per qualsiasi `[Inject]` proprietà genera un'eccezione non gestita. L'eccezione è irreversibile perché il Framework non è in grado di creare un'istanza del componente. Se la logica del costruttore può generare eccezioni, l'app deve intercettare le eccezioni usando un' [`try-catch`](/dotnet/csharp/language-reference/keywords/try-catch) istruzione con gestione e registrazione degli errori.

### <a name="lifecycle-methods"></a>Metodi del ciclo di vita

Durante la durata di un componente, Blazor richiama i metodi del [ciclo](xref:blazor/components/lifecycle)di vita seguenti:

* <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitialized%2A> / <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A>
* <xref:Microsoft.AspNetCore.Components.ComponentBase.OnParametersSet%2A> / <xref:Microsoft.AspNetCore.Components.ComponentBase.OnParametersSetAsync%2A>
* <xref:Microsoft.AspNetCore.Components.ComponentBase.ShouldRender%2A>
* <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRender%2A> / <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A>

Se un metodo del ciclo di vita genera un'eccezione, in modo sincrono o asincrono, l'eccezione è fatale per un Blazor Server circuito. Per i componenti che gestiscono gli errori nei metodi del ciclo di vita, aggiungere la logica di gestione degli errori.

Nell'esempio seguente viene <xref:Microsoft.AspNetCore.Components.ComponentBase.OnParametersSetAsync%2A> chiamato un metodo per ottenere un prodotto:

* Un'eccezione generata nel `ProductRepository.GetProductByIdAsync` metodo viene gestita da un' [`try-catch`](/dotnet/csharp/language-reference/keywords/try-catch) istruzione.
* Quando il `catch` blocco viene eseguito:
  * `loadFailed` è impostato su `true` , che viene utilizzato per visualizzare un messaggio di errore all'utente.
  * L'errore viene registrato.

[!code-razor[](handle-errors/samples_snapshot/3.x/product-details.razor?highlight=11,27-39)]

### <a name="rendering-logic"></a>Logica di rendering

Il markup dichiarativo in un `.razor` file componente viene compilato in un metodo C# denominato <xref:Microsoft.AspNetCore.Components.ComponentBase.BuildRenderTree%2A> . Quando viene eseguito il rendering di un componente, <xref:Microsoft.AspNetCore.Components.ComponentBase.BuildRenderTree%2A> viene eseguita e compilata una struttura di dati che descrive gli elementi, il testo e i componenti figlio del componente di cui è stato eseguito il rendering.

La logica di rendering può generare un'eccezione. Un esempio di questo scenario si verifica quando `@someObject.PropertyName` viene valutato ma `@someObject` è `null` . Un'eccezione non gestita generata dalla logica di rendering è irreversibile per un Blazor Server circuito.

Per evitare un'eccezione di riferimento null nella logica di rendering, verificare la presenza di un `null` oggetto prima di accedere ai relativi membri. Nell'esempio seguente, `person.Address` non è possibile accedere alle proprietà se `person.Address` è `null` :

[!code-razor[](handle-errors/samples_snapshot/3.x/person-example.razor?highlight=1)]

Il codice precedente presuppone che `person` non sia `null` . Spesso, la struttura del codice garantisce la presenza di un oggetto nel momento in cui viene eseguito il rendering del componente. In questi casi, non è necessario verificare la presenza di `null` nella logica di rendering. Nell'esempio precedente, `person` potrebbe essere garantito che esista perché `person` viene creato quando viene creata un'istanza del componente.

### <a name="event-handlers"></a>Gestori eventi

Il codice lato client attiva le chiamate del codice C# quando i gestori eventi vengono creati usando:

* `@onclick`
* `@onchange`
* Altri `@on...` attributi
* `@bind`

Il codice del gestore eventi potrebbe generare un'eccezione non gestita in questi scenari.

Se un gestore eventi genera un'eccezione non gestita, ad esempio una query di database ha esito negativo, l'eccezione è fatale per un Blazor Server circuito. Se l'app chiama il codice che potrebbe non riuscire per motivi esterni, intercettare le eccezioni usando un' [`try-catch`](/dotnet/csharp/language-reference/keywords/try-catch) istruzione con gestione e registrazione degli errori.

Se il codice utente non intercetta e gestisce l'eccezione, il Framework registra l'eccezione e termina il circuito.

### <a name="component-disposal"></a>Eliminazione componenti

Un componente può essere rimosso dall'interfaccia utente, ad esempio perché l'utente ha esplorato un'altra pagina. Quando un componente che implementa <xref:System.IDisposable?displayProperty=fullName> viene rimosso dall'interfaccia utente, il Framework chiama il metodo del componente <xref:System.IDisposable.Dispose%2A> .

Se il metodo del componente `Dispose` genera un'eccezione non gestita, l'eccezione è fatale per un Blazor Server circuito. Se la logica di eliminazione può generare eccezioni, l'app deve intercettare le eccezioni usando un' [`try-catch`](/dotnet/csharp/language-reference/keywords/try-catch) istruzione con gestione e registrazione degli errori.

Per ulteriori informazioni sull'eliminazione dei componenti, vedere <xref:blazor/components/lifecycle#component-disposal-with-idisposable> .

### <a name="javascript-interop"></a>Interoperabilità JavaScript

<xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A?displayProperty=nameWithType> consente al codice .NET di effettuare chiamate asincrone al runtime JavaScript nel browser dell'utente.

Le condizioni seguenti si applicano alla gestione degli errori con <xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A> :

* Se una chiamata a ha <xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A> esito negativo in modo sincrono, si verifica un'eccezione .NET. Una chiamata a <xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A> potrebbe non riuscire, ad esempio perché gli argomenti forniti non possono essere serializzati. Il codice dello sviluppatore deve intercettare l'eccezione. Se il codice dell'app in un gestore eventi o in un metodo del ciclo di vita dei componenti non gestisce un'eccezione, l'eccezione risultante è fatale per un Blazor Server circuito.
* Se una chiamata a ha <xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A> esito negativo in modo asincrono, .NET ha <xref:System.Threading.Tasks.Task> esito negativo. Una chiamata a <xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A> potrebbe non riuscire, ad esempio perché il codice sul lato JavaScript genera un'eccezione o restituisce un oggetto `Promise` completato come `rejected` . Il codice dello sviluppatore deve intercettare l'eccezione. Se si utilizza l' [`await`](/dotnet/csharp/language-reference/keywords/await) operatore, è consigliabile eseguire il wrapping della chiamata al metodo in un' [`try-catch`](/dotnet/csharp/language-reference/keywords/try-catch) istruzione con gestione e registrazione degli errori. In caso contrario, il codice in errore genera un'eccezione non gestita che è irreversibile per un Blazor Server circuito.
* Per impostazione predefinita, le chiamate a <xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A> devono essere completate entro un determinato periodo oppure si verifica il timeout della chiamata. Il periodo di timeout predefinito è di un minuto. Il timeout protegge il codice da una perdita di connettività di rete o codice JavaScript che non restituisce mai un messaggio di completamento. Se si verifica il timeout della chiamata, l'oggetto risultante ha <xref:System.Threading.Tasks> esito negativo con un oggetto <xref:System.OperationCanceledException> . Intercettare ed elaborare l'eccezione con la registrazione.

Analogamente, il codice JavaScript può avviare chiamate a metodi .NET indicati dall' [ `[JSInvokable]` attributo](xref:blazor/call-dotnet-from-javascript). Se questi metodi .NET generano un'eccezione non gestita:

* L'eccezione non viene trattata come irreversibile per un Blazor Server circuito.
* Il lato JavaScript `Promise` viene rifiutato.

È possibile scegliere di usare il codice di gestione degli errori sul lato .NET o sul lato JavaScript della chiamata al metodo.

Per altre informazioni, vedere gli articoli seguenti:

* <xref:blazor/call-javascript-from-dotnet>
* <xref:blazor/call-dotnet-from-javascript>

### <a name="blazor-server-prerendering"></a>Blazor Server tempistiche

Blazor è possibile eseguire il prerendering dei componenti usando l' [Helper Tag Component](xref:mvc/views/tag-helpers/builtin-th/component-tag-helper) , in modo che il markup HTML sottoposto a rendering venga restituito come parte della richiesta HTTP iniziale dell'utente. Funziona per:

* Creazione di un nuovo circuito per tutti i componenti di cui è stato eseguito il rendering che fanno parte della stessa pagina.
* Generazione del codice HTML iniziale.
* Trattare il circuito come `disconnected` fino a quando il browser dell'utente non stabilisce una SignalR connessione allo stesso server. Quando viene stabilita la connessione, interattività sul circuito viene ripresa e il markup HTML dei componenti viene aggiornato.

Se un componente genera un'eccezione non gestita durante il prerendering, ad esempio durante un metodo del ciclo di vita o nella logica di rendering:

* L'eccezione è fatale per il circuito.
* L'eccezione viene generata dallo stack di chiamate dall' <xref:Microsoft.AspNetCore.Mvc.TagHelpers.ComponentTagHelper> Helper tag. Pertanto, l'intera richiesta HTTP ha esito negativo a meno che l'eccezione non venga intercettata in modo esplicito dal codice dello sviluppatore.

In circostanze normali, quando si verifica un errore di prerendering, continuare a compilare ed eseguire il rendering del componente non ha senso perché non è possibile eseguire il rendering di un componente funzionante.

Per tollerare gli errori che possono verificarsi durante il prerendering, la logica di gestione degli errori deve essere inserita all'interno di un componente che può generare eccezioni. Usare [`try-catch`](/dotnet/csharp/language-reference/keywords/try-catch) le istruzioni con la gestione e la registrazione degli errori. Anziché eseguire il wrapping dell' <xref:Microsoft.AspNetCore.Mvc.TagHelpers.ComponentTagHelper> Helper tag in un' [`try-catch`](/dotnet/csharp/language-reference/keywords/try-catch) istruzione, inserire la logica di gestione degli errori nel componente di cui viene eseguito il rendering dall' <xref:Microsoft.AspNetCore.Mvc.TagHelpers.ComponentTagHelper> Helper tag.

## <a name="advanced-scenarios"></a>Scenari avanzati

### <a name="recursive-rendering"></a>Rendering ricorsivo

I componenti possono essere annidati in modo ricorsivo. Questa operazione è utile per la rappresentazione di strutture di dati ricorsive. Un componente, ad esempio, `TreeNode` può eseguire il rendering di più `TreeNode` componenti per ognuno dei figli del nodo.

Quando si esegue il rendering in modo ricorsivo, evitare i modelli di codifica che generano una ricorsione

* Non esegue il rendering in modo ricorsivo di una struttura di dati che contiene un ciclo. Ad esempio, non eseguire il rendering di un nodo dell'albero i cui elementi figlio includono se stesso.
* Non creare una catena di layout che contiene un ciclo. Non creare, ad esempio, un layout il cui layout è a sua volta.
* Non consentire a un utente finale di violare le invarianti di ricorsione (regole) tramite immissione di dati dannosi o chiamate di interoperabilità JavaScript.

Cicli infiniti durante il rendering:

* Causa la continuazione del processo di rendering.
* Equivale alla creazione di un ciclo senza terminazione.

In questi scenari, un circuito interessato ha Blazor Server esito negativo e il thread in genere tenta di:

* Utilizzare la quantità di tempo della CPU consentita dal sistema operativo, per un periodo illimitato.
* Utilizzare una quantità illimitata di memoria del server. L'utilizzo di memoria illimitata equivale allo scenario in cui un ciclo senza terminazione aggiunge voci a una raccolta in ogni iterazione.

Per evitare modelli di ricorsione infinita, verificare che il codice di rendering ricorsivo contenga condizioni di arresto appropriate.

### <a name="custom-render-tree-logic"></a>Logica dell'albero di rendering personalizzata

La maggior parte dei Blazor componenti viene implementata come `.razor` file e viene compilata per produrre la logica che opera su un <xref:Microsoft.AspNetCore.Components.Rendering.RenderTreeBuilder> per eseguire il rendering dell'output. Uno sviluppatore può implementare manualmente la <xref:Microsoft.AspNetCore.Components.Rendering.RenderTreeBuilder> logica usando il codice C# procedurale. Per altre informazioni, vedere <xref:blazor/advanced-scenarios#manual-rendertreebuilder-logic>.

> [!WARNING]
> L'uso della logica del generatore di albero di rendering manuale è considerato uno scenario avanzato e non sicuro, non consigliato per lo sviluppo di componenti generali.

Se il <xref:Microsoft.AspNetCore.Components.Rendering.RenderTreeBuilder> codice viene scritto, lo sviluppatore deve garantire la correttezza del codice. Ad esempio, lo sviluppatore deve garantire quanto segue:

* Le chiamate a <xref:Microsoft.AspNetCore.Components.Rendering.RenderTreeBuilder.OpenElement%2A> e <xref:Microsoft.AspNetCore.Components.Rendering.RenderTreeBuilder.CloseElement%2A> sono bilanciate correttamente.
* Gli attributi vengono aggiunti solo nei punti corretti.

La logica del generatore di albero di rendering manuale non corretta può causare un comportamento arbitrario non definito, tra cui arresti anomali, blocchi del server e vulnerabilità della sicurezza.

Prendere in considerazione la logica del generatore di albero di rendering manuale sullo stesso livello di complessità e con lo stesso livello di *rischio* di scrivere manualmente codice assembly o istruzioni MSIL.
