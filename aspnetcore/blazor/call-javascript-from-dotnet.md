---
title: Chiamare funzioni JavaScript da metodi .NET in ASP.NET Core Blazor
author: guardrex
description: Informazioni su come richiamare funzioni JavaScript da metodi .NET nelle Blazor app.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc, devx-track-js
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
uid: blazor/call-javascript-from-dotnet
ms.openlocfilehash: 53b702cddca778e06e617df3798bffb21677d36b
ms.sourcegitcommit: 610936e4d3507f7f3d467ed7859ab9354ec158ba
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 01/25/2021
ms.locfileid: "98751645"
---
# <a name="call-javascript-functions-from-net-methods-in-aspnet-core-no-locblazor"></a>Chiamare funzioni JavaScript da metodi .NET in ASP.NET Core Blazor

Di [Javier Calvarro Nelson](https://github.com/javiercn), [Daniel Roth](https://github.com/danroth27), [](https://github.com/pranavkm)Manuel e [Luke Latham](https://github.com/guardrex)

Un' Blazor app può richiamare funzioni JavaScript da metodi .NET e metodi .NET da funzioni JavaScript. Questi scenari sono detti *interoperabilità JavaScript (interoperabilità* *JS*).

Questo articolo illustra come richiamare funzioni JavaScript da .NET. Per informazioni su come chiamare i metodi .NET da JavaScript, vedere <xref:blazor/call-dotnet-from-javascript> .

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/blazor/common/samples/) ([procedura per il download](xref:index#how-to-download-a-sample))

> [!NOTE]
> Aggiungere i file JS ( `<script>` tag) prima del `</body>` tag di chiusura nel `wwwroot/index.html` file ( Blazor WebAssembly ) o nel `Pages/_Host.cshtml` file ( Blazor Server ). Assicurarsi che i file JS con i metodi di interoperabilità JS siano inclusi prima dei Blazor file di framework JS.

Per chiamare JavaScript da .NET, usare l' <xref:Microsoft.JSInterop.IJSRuntime> astrazione. Per rilasciare chiamate di interoperabilità JS, inserire l' <xref:Microsoft.JSInterop.IJSRuntime> astrazione nel componente. <xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A> accetta un identificatore per la funzione JavaScript che si vuole richiamare insieme a un numero qualsiasi di argomenti serializzabili in JSON. L'identificatore della funzione è relativo all'ambito globale ( `window` ). Se si desidera chiamare `window.someScope.someFunction` , l'identificatore è `someScope.someFunction` . Non è necessario registrare la funzione prima che venga chiamata. Il tipo restituito `T` deve anche essere serializzabile in JSON. `T` deve corrispondere al tipo .NET che meglio esegue il mapping al tipo JSON restituito.

Le funzioni JavaScript che restituiscono una [promessa](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Promise) vengono chiamate con <xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A> . `InvokeAsync` rimuove il wrapping della promessa e restituisce il valore atteso dalla promessa.

Per Blazor Server le app con il prerendering abilitato, non è possibile chiamare JavaScript durante il prerendering iniziale. È necessario rinviare le chiamate di interoperabilità JavaScript finché non viene stabilita la connessione con il browser. Per altre informazioni, vedere la sezione [rilevare quando un' Blazor Server app è prerendering](#detect-when-a-blazor-server-app-is-prerendering) .

L'esempio seguente è basato su [`TextDecoder`](https://developer.mozilla.org/docs/Web/API/TextDecoder) , un decodificatore basato su JavaScript. Nell'esempio viene illustrato come richiamare una funzione JavaScript da un metodo C# che Scarica un requisito dal codice dello sviluppatore a un'API JavaScript esistente. La funzione JavaScript accetta una matrice di byte da un metodo C#, decodifica la matrice e restituisce il testo al componente per la visualizzazione.

All'interno dell' `<head>` elemento di `wwwroot/index.html` ( Blazor WebAssembly ) o `Pages/_Host.cshtml` ( Blazor Server ), fornire una funzione JavaScript che usi `TextDecoder` per decodificare una matrice passata e restituire il valore decodificato:

[!code-html[](call-javascript-from-dotnet/samples_snapshot/index-script-convertarray.html)]

Il codice JavaScript, ad esempio il codice illustrato nell'esempio precedente, può anche essere caricato da un file JavaScript ( `.js` ) con un riferimento al file script:

```html
<script src="exampleJsInterop.js"></script>
```

Il componente seguente:

* Richiama la `convertArray` funzione JavaScript usando `JS` quando viene selezionato un pulsante del componente ( **`Convert Array`** ).
* Dopo la chiamata della funzione JavaScript, la matrice passata viene convertita in una stringa. La stringa viene restituita al componente per la visualizzazione.

[!code-razor[](call-javascript-from-dotnet/samples_snapshot/call-js-example.razor?highlight=2,34-35)]

## <a name="ijsruntime"></a>IJSRuntime

Per usare l' <xref:Microsoft.JSInterop.IJSRuntime> astrazione, adottare uno degli approcci seguenti:

* Inserire l' <xref:Microsoft.JSInterop.IJSRuntime> astrazione nel Razor componente ( `.razor` ):

  [!code-razor[](call-javascript-from-dotnet/samples_snapshot/inject-abstraction.razor?highlight=1)]

  All'interno dell' `<head>` elemento di `wwwroot/index.html` ( Blazor WebAssembly ) o `Pages/_Host.cshtml` ( Blazor Server ) specificare una `handleTickerChanged` funzione JavaScript. La funzione viene chiamata con <xref:Microsoft.JSInterop.JSRuntimeExtensions.InvokeVoidAsync%2A?displayProperty=nameWithType> e non restituisce un valore:

  [!code-html[](call-javascript-from-dotnet/samples_snapshot/index-script-handleTickerChanged1.html)]

* Inserire l' <xref:Microsoft.JSInterop.IJSRuntime> astrazione in una classe ( `.cs` ):

  [!code-csharp[](call-javascript-from-dotnet/samples_snapshot/inject-abstraction-class.cs?highlight=5)]

  All'interno dell' `<head>` elemento di `wwwroot/index.html` ( Blazor WebAssembly ) o `Pages/_Host.cshtml` ( Blazor Server ) specificare una `handleTickerChanged` funzione JavaScript. La funzione viene chiamata con `JS.InvokeAsync` e restituisce un valore:

  [!code-html[](call-javascript-from-dotnet/samples_snapshot/index-script-handleTickerChanged2.html)]

* Per la generazione di contenuto dinamico con [BuildRenderTree](xref:blazor/advanced-scenarios#manual-rendertreebuilder-logic), usare l' `[Inject]` attributo:

  ```razor
  [Inject]
  IJSRuntime JS { get; set; }
  ```

Nell'app di esempio lato client che accompagna questo argomento sono disponibili due funzioni JavaScript per l'app che interagiscono con il DOM per ricevere l'input dell'utente e visualizzare un messaggio di benvenuto:

* `showPrompt`: Genera un messaggio di richiesta per accettare l'input dell'utente (il nome dell'utente) e restituisce il nome al chiamante.
* `displayWelcome`: Assegna un messaggio di benvenuto dal chiamante a un oggetto DOM con un valore `id` di `welcome` .

`wwwroot/exampleJsInterop.js`:

[!code-javascript[](./common/samples/5.x/BlazorWebAssemblySample/wwwroot/exampleJsInterop.js?highlight=2-7)]

Inserire il `<script>` tag che fa riferimento al file JavaScript nel file `wwwroot/index.html` ( Blazor WebAssembly ) o nel `Pages/_Host.cshtml` file ( Blazor Server ).

`wwwroot/index.html` (Blazor WebAssembly):

[!code-html[](./common/samples/5.x/BlazorWebAssemblySample/wwwroot/index.html?highlight=22)]

`Pages/_Host.cshtml` (Blazor Server):

[!code-cshtml[](./common/samples/5.x/BlazorServerSample/Pages/_Host.cshtml?highlight=34)]

Non inserire un `<script>` tag in un file di componente perché il `<script>` tag non può essere aggiornato dinamicamente.

I metodi .NET interoperano con le funzioni JavaScript nel `exampleJsInterop.js` file chiamando <xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A?displayProperty=nameWithType> .

L' <xref:Microsoft.JSInterop.IJSRuntime> astrazione è asincrona per consentire gli Blazor Server scenari. Se l'app è un' Blazor WebAssembly app e si vuole richiamare una funzione JavaScript in modo sincrono, <xref:Microsoft.JSInterop.IJSInProcessRuntime> abbandono e chiamano <xref:Microsoft.JSInterop.IJSInProcessRuntime.Invoke%2A> invece. È consigliabile che la maggior parte delle librerie di interoperabilità JS usi le API asincrone per assicurarsi che le librerie siano disponibili in tutti gli scenari.

::: moniker range=">= aspnetcore-5.0"

> [!NOTE]
> Per abilitare l'isolamento JavaScript nei [moduli JavaScript](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Modules)standard, vedere la sezione [ Blazor riferimenti a oggetti e isolamento JavaScript](#blazor-javascript-isolation-and-object-references) .

::: moniker-end

L'app di esempio include un componente per dimostrare l'interoperabilità JS. Il componente:

* Riceve l'input dell'utente tramite un prompt di JavaScript.
* Restituisce il testo al componente per l'elaborazione.
* Chiama una seconda funzione JavaScript che interagisce con il DOM per visualizzare un messaggio di benvenuto.

`Pages/JsInterop.razor`:

```razor
@page "/JSInterop"
@using {APP ASSEMBLY}.JsInteropClasses
@inject IJSRuntime JS

<h1>JavaScript Interop</h1>

<h2>Invoke JavaScript functions from .NET methods</h2>

<button type="button" class="btn btn-primary" @onclick="TriggerJsPrompt">
    Trigger JavaScript Prompt
</button>

<h3 id="welcome" style="color:green;font-style:italic"></h3>

@code {
    public async Task TriggerJsPrompt()
    {
        var name = await JS.InvokeAsync<string>(
                "exampleJsFunctions.showPrompt",
                "What's your name?");

        await JS.InvokeVoidAsync(
                "exampleJsFunctions.displayWelcome",
                $"Hello {name}! Welcome to Blazor!");
    }
}
```

Il segnaposto `{APP ASSEMBLY}` è il nome dell'assembly dell'app (ad esempio, `BlazorSample` ).

1. Quando `TriggerJsPrompt` viene eseguito selezionando il pulsante del componente **`Trigger JavaScript Prompt`** , `showPrompt` viene chiamata la funzione JavaScript fornita nel `wwwroot/exampleJsInterop.js` file.
1. La `showPrompt` funzione accetta l'input dell'utente (il nome dell'utente), che è codificato in HTML e restituito al componente. Il componente archivia il nome dell'utente in una variabile locale, `name` .
1. La stringa archiviata in `name` viene incorporata in un messaggio di benvenuto, che viene passato a una funzione JavaScript, `displayWelcome` , che esegue il rendering del messaggio di benvenuto in un tag di intestazione.

## <a name="call-a-void-javascript-function"></a>Chiamare una funzione JavaScript void

Utilizzare <xref:Microsoft.JSInterop.JSRuntimeExtensions.InvokeVoidAsync%2A?displayProperty=nameWithType> per gli elementi seguenti:

* Funzioni JavaScript che restituiscono [void (0)/void 0](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Operators/void) o [undefined](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/undefined).
* Se .NET non è necessario per leggere il risultato di una chiamata a JavaScript.

## <a name="detect-when-a-no-locblazor-server-app-is-prerendering"></a>Rileva quando viene eseguito il Blazor Server prerendering di un'app
 
[!INCLUDE[](~/blazor/includes/prerendering.md)]

## <a name="capture-references-to-elements"></a>Acquisisci riferimenti a elementi

Alcuni scenari di interoperabilità JS richiedono riferimenti agli elementi HTML. Ad esempio, una libreria dell'interfaccia utente può richiedere un riferimento a un elemento per l'inizializzazione o potrebbe essere necessario chiamare API simili a quelle di un elemento, ad esempio `focus` o `play` .

Acquisire i riferimenti agli elementi HTML in un componente usando l'approccio seguente:

* Aggiungere un `@ref` attributo all'elemento HTML.
* Definire un campo di tipo <xref:Microsoft.AspNetCore.Components.ElementReference> il cui nome corrisponde al valore dell' `@ref` attributo.

Nell'esempio seguente viene illustrata l'acquisizione di un riferimento all' `username` `<input>` elemento:

```razor
<input @ref="username" ... />

@code {
    ElementReference username;
}
```

> [!WARNING]
> Usare solo un riferimento a un elemento per mutare il contenuto di un elemento vuoto che non interagisce con Blazor . Questo scenario è utile quando un'API di terze parti fornisce contenuto all'elemento. Poiché Blazor non interagisce con l'elemento, non è possibile che si verifichi un conflitto tra Blazor la rappresentazione dell'elemento e il Dom.
>
> Nell'esempio seguente è *pericoloso* modificare il contenuto dell'elenco non ordinato ( `ul` ) perché Blazor interagisce con il Dom per popolare gli elementi elenco di questo elemento ( `<li>` ):
>
> ```razor
> <ul ref="MyList">
>     @foreach (var item in Todos)
>     {
>         <li>@item.Text</li>
>     }
> </ul>
> ```
>
> Se l'interoperabilità JS modifica il contenuto dell'elemento `MyList` e Blazor tenta di applicare differenze all'elemento, le differenze non corrispondono al Dom.

Una <xref:Microsoft.AspNetCore.Components.ElementReference> viene passata al codice JavaScript tramite l'interoperabilità js. Il codice JavaScript riceve un' `HTMLElement` istanza, che può essere usata con le API DOM normali. Il codice seguente, ad esempio, definisce un metodo di estensione .NET che consente l'invio di un clic del mouse a un elemento:

`exampleJsInterop.js`:

```javascript
window.interopFunctions = {
  clickElement : function (element) {
    element.click();
  }
}
```

::: moniker range=">= aspnetcore-5.0"

> [!NOTE]
> Usare [`FocusAsync`](xref:blazor/components/event-handling#focus-an-element) il codice C# per concentrare un elemento, che è incorporato nel Blazor Framework e funziona con i riferimenti agli elementi.

::: moniker-end

Per chiamare una funzione JavaScript che non restituisce un valore, usare <xref:Microsoft.JSInterop.JSRuntimeExtensions.InvokeVoidAsync%2A?displayProperty=nameWithType> . Il codice seguente attiva un evento sul lato client chiamando `Click` la funzione JavaScript precedente con l'acquisizione <xref:Microsoft.AspNetCore.Components.ElementReference> :

[!code-razor[](call-javascript-from-dotnet/samples_snapshot/component1.razor?highlight=14-15)]

Per usare un metodo di estensione, creare un metodo di estensione statico che riceva l' <xref:Microsoft.JSInterop.IJSRuntime> istanza:

```csharp
public static async Task TriggerClickEvent(this ElementReference elementRef, 
    IJSRuntime js)
{
    await js.InvokeVoidAsync("interopFunctions.clickElement", elementRef);
}
```

Il `clickElement` metodo viene chiamato direttamente nell'oggetto. Nell'esempio seguente si presuppone che il `TriggerClickEvent` metodo sia disponibile dallo `JsInteropClasses` spazio dei nomi:

[!code-razor[](call-javascript-from-dotnet/samples_snapshot/component2.razor?highlight=15)]

> [!IMPORTANT]
> La `exampleButton` variabile viene popolata solo dopo il rendering del componente. Se un oggetto non popolato <xref:Microsoft.AspNetCore.Components.ElementReference> viene passato al codice JavaScript, il codice JavaScript riceve un valore di `null` . Per modificare i riferimenti agli elementi dopo che il componente ha terminato il rendering, usare i metodi del ciclo di vita dei [ `OnAfterRenderAsync` `OnAfterRender` componenti o](xref:blazor/components/lifecycle#after-component-render).

Quando si utilizzano tipi generici e si restituisce un valore, utilizzare <xref:System.Threading.Tasks.ValueTask%601> :

```csharp
public static ValueTask<T> GenericMethod<T>(this ElementReference elementRef, 
    IJSRuntime js)
{
    return js.InvokeAsync<T>("exampleJsFunctions.doSomethingGeneric", elementRef);
}
```

`GenericMethod` viene chiamato direttamente sull'oggetto con un tipo. Nell'esempio seguente si presuppone che l'oggetto `GenericMethod` sia disponibile dallo `JsInteropClasses` spazio dei nomi:

[!code-razor[](call-javascript-from-dotnet/samples_snapshot/component3.razor?highlight=17)]

## <a name="reference-elements-across-components"></a>Elementi di riferimento tra i componenti

<xref:Microsoft.AspNetCore.Components.ElementReference>Non è possibile passare un oggetto tra i componenti perché:

* L'istanza è garantita solo dopo il rendering del componente, ovvero durante o dopo l'esecuzione del metodo di un componente <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRender%2A> / <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A> .
* Un oggetto <xref:Microsoft.AspNetCore.Components.ElementReference> è un oggetto [`struct`](/csharp/language-reference/builtin-types/struct) che non può essere passato come [parametro component](xref:blazor/components/index#component-parameters).

Affinché un componente padre renda disponibile un riferimento a un elemento per altri componenti, il componente padre può:

* Consenti ai componenti figlio di registrare i callback.
* Richiamare i callback registrati durante l' <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRender%2A> evento con il riferimento all'elemento passato. Indirettamente, questo approccio consente ai componenti figlio di interagire con il riferimento all'elemento padre.

Nell' Blazor WebAssembly esempio seguente viene illustrato l'approccio.

Nella `<head>` di `wwwroot/index.html` :

```html
<style>
    .red { color: red }
</style>
```

Nella `<body>` di `wwwroot/index.html` :

```html
<script>
    function setElementClass(element, className) {
        /** @type {HTMLElement} **/
        var myElement = element;
        myElement.classList.add(className);
    }
</script>
```

`Pages/Index.razor` (componente padre):

```razor
@page "/"

<h1 @ref="title">Hello, world!</h1>

Welcome to your new app.

<SurveyPrompt Parent="this" Title="How is Blazor working for you?" />
```

`Pages/Index.razor.cs`:

```csharp
using System;
using System.Collections.Generic;
using Microsoft.AspNetCore.Components;

namespace {APP ASSEMBLY}.Pages
{
    public partial class Index : 
        ComponentBase, IObservable<ElementReference>, IDisposable
    {
        private bool disposing;
        private IList<IObserver<ElementReference>> subscriptions = 
            new List<IObserver<ElementReference>>();
        private ElementReference title;

        protected override void OnAfterRender(bool firstRender)
        {
            base.OnAfterRender(firstRender);

            foreach (var subscription in subscriptions)
            {
                try
                {
                    subscription.OnNext(title);
                }
                catch (Exception)
                {
                    throw;
                }
            }
        }

        public void Dispose()
        {
            disposing = true;

            foreach (var subscription in subscriptions)
            {
                try
                {
                    subscription.OnCompleted();
                }
                catch (Exception)
                {
                }
            }

            subscriptions.Clear();
        }

        public IDisposable Subscribe(IObserver<ElementReference> observer)
        {
            if (disposing)
            {
                throw new InvalidOperationException("Parent being disposed");
            }

            subscriptions.Add(observer);

            return new Subscription(observer, this);
        }

        private class Subscription : IDisposable
        {
            public Subscription(IObserver<ElementReference> observer, Index self)
            {
                Observer = observer;
                Self = self;
            }

            public IObserver<ElementReference> Observer { get; }
            public Index Self { get; }

            public void Dispose()
            {
                Self.subscriptions.Remove(Observer);
            }
        }
    }
}
```

Il segnaposto `{APP ASSEMBLY}` è il nome dell'assembly dell'app (ad esempio, `BlazorSample` ).

`Shared/SurveyPrompt.razor` (componente figlio):

```razor
@inject IJSRuntime JS

<div class="alert alert-secondary mt-4" role="alert">
    <span class="oi oi-pencil mr-2" aria-hidden="true"></span>
    <strong>@Title</strong>

    <span class="text-nowrap">
        Please take our
        <a target="_blank" class="font-weight-bold" 
            href="https://go.microsoft.com/fwlink/?linkid=2109206">brief survey</a>
    </span>
    and tell us what you think.
</div>

@code {
    [Parameter]
    public string Title { get; set; }
}
```

`Shared/SurveyPrompt.razor.cs`:

```csharp
using System;
using Microsoft.AspNetCore.Components;

namespace {APP ASSEMBLY}.Shared
{
    public partial class SurveyPrompt : 
        ComponentBase, IObserver<ElementReference>, IDisposable
    {
        private IDisposable subscription = null;

        [Parameter]
        public IObservable<ElementReference> Parent { get; set; }

        protected override void OnParametersSet()
        {
            base.OnParametersSet();

            if (subscription != null)
            {
                subscription.Dispose();
            }

            subscription = Parent.Subscribe(this);
        }

        public void OnCompleted()
        {
            subscription = null;
        }

        public void OnError(Exception error)
        {
            subscription = null;
        }

        public void OnNext(ElementReference value)
        {
            JS.InvokeAsync<object>(
                "setElementClass", new object[] { value, "red" });
        }

        public void Dispose()
        {
            subscription?.Dispose();
        }
    }
}
```

Il segnaposto `{APP ASSEMBLY}` è il nome dell'assembly dell'app (ad esempio, `BlazorSample` ).

## <a name="harden-js-interop-calls"></a>Chiamate di interoperabilità JS di Harden

L'interoperabilità JS potrebbe non riuscire a causa di errori di rete e deve essere considerata inaffidabile. Per impostazione predefinita, un' Blazor Server app timeout delle chiamate di interoperabilità js nel server dopo un minuto. Se un'app può tollerare un timeout più aggressivo, impostare il timeout usando uno degli approcci seguenti:

* A livello globale in `Startup.ConfigureServices` specificare il timeout:

  ```csharp
  services.AddServerSideBlazor(
      options => options.JSInteropDefaultCallTimeout = TimeSpan.FromSeconds({SECONDS}));
  ```

* Per chiamata nel codice componente, una singola chiamata può specificare il timeout:

  ```csharp
  var result = await JS.InvokeAsync<string>("MyJSOperation", 
      TimeSpan.FromSeconds({SECONDS}), new[] { "Arg1" });
  ```

Per ulteriori informazioni sull'esaurimento delle risorse, vedere <xref:blazor/security/server/threat-mitigation> .

[!INCLUDE[](~/blazor/includes/share-interop-code.md)]

## <a name="avoid-circular-object-references"></a>Evitare riferimenti a oggetti circolari

Gli oggetti che contengono riferimenti circolari non possono essere serializzati sul client per uno dei seguenti:

* Chiamate al metodo .NET.
* Chiamate al metodo JavaScript da C# quando il tipo restituito contiene riferimenti circolari.

Per altre informazioni, vedere [riferimenti circolari non supportati. prendere due (DotNet/aspnetcore #20525)](https://github.com/dotnet/aspnetcore/issues/20525).

::: moniker range=">= aspnetcore-5.0"

## <a name="no-locblazor-javascript-isolation-and-object-references"></a>Blazor Isolamento JavaScript e riferimenti a oggetti

Blazor Abilita l'isolamento JavaScript nei [moduli JavaScript](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Modules)standard. L'isolamento JavaScript offre i vantaggi seguenti:

* Il codice JavaScript importato non inquina più lo spazio dei nomi globale.
* I consumer di una libreria e i componenti non sono necessari per importare il codice JavaScript correlato.

Il modulo JavaScript seguente, ad esempio, esporta una funzione JavaScript per visualizzare un prompt del browser:

```javascript
export function showPrompt(message) {
  return prompt(message, 'Type anything here');
}
```

Aggiungere il modulo JavaScript precedente a una libreria .NET come asset Web statico ( `wwwroot/exampleJsInterop.js` ) e quindi importare il modulo nel codice .NET usando il <xref:Microsoft.JSInterop.IJSRuntime> servizio. Il servizio viene inserito come `js` (non visualizzato) per l'esempio seguente:

```csharp
var module = await js.InvokeAsync<IJSObjectReference>(
    "import", "./_content/MyComponents/exampleJsInterop.js");
```

L' `import` identificatore nell'esempio precedente è un identificatore speciale usato in modo specifico per l'importazione di un modulo JavaScript. Specificare il modulo usando il percorso di asset Web statico stabile: `./_content/{LIBRARY NAME}/{PATH UNDER WWWROOT}` . Il segmento di percorso per la directory corrente ( `./` ) è necessario per creare il percorso Asset statico corretto per il file JavaScript. Il segnaposto `{LIBRARY NAME}` è il nome della libreria. Il segnaposto `{PATH UNDER WWWROOT}` è il percorso dello script in `wwwroot` .

<xref:Microsoft.JSInterop.IJSRuntime> importa il modulo come `IJSObjectReference` , che rappresenta un riferimento a un oggetto JavaScript dal codice .NET. Usare `IJSObjectReference` per richiamare le funzioni JavaScript esportate dal modulo:

```csharp
public async ValueTask<string> Prompt(string message)
{
    return await module.InvokeAsync<string>("showPrompt", message);
}
```

`IJSInProcessObjectReference` rappresenta un riferimento a un oggetto JavaScript le cui funzioni possono essere richiamate in modo sincrono.

## <a name="use-of-javascript-libraries-that-render-ui-dom-elements"></a>Uso di librerie JavaScript che eseguono il rendering dell'interfaccia utente (elementi DOM)

In alcuni casi può essere utile usare librerie JavaScript che producono elementi dell'interfaccia utente visibili all'interno del DOM del browser. A prima vista, questo potrebbe sembrare difficile perché il Blazor sistema di differenziazione si basa sul controllo della struttura ad albero di elementi DOM e si verificano errori se il codice esterno muta l'albero DOM e invalida il meccanismo per l'applicazione di diff. Questa limitazione non è Blazor specifica. Lo stesso problema si verifica con qualsiasi framework dell'interfaccia utente basato su diff.

Fortunatamente, è facile incorporare in modo affidabile l'interfaccia utente generata dall'esterno all'interno di un' Blazor interfaccia utente del componente. La tecnica consigliata consiste nel fare in modo che il codice del componente ( `.razor` file) produca un elemento vuoto. Per quanto Blazor riguarda il sistema diffing, l'elemento è sempre vuoto, quindi il renderer non esegue la ricorsione nell'elemento e lascia il proprio contenuto. In questo modo è possibile popolare l'elemento con contenuto arbitrario gestito esternamente.

Nell'esempio seguente viene illustrato il concetto. All'interno dell' `if` istruzione quando `firstRender` è `true` , eseguire un'operazione con `myElement` . Ad esempio, chiamare una libreria JavaScript esterna per popolarla. Blazor lascia solo il contenuto dell'elemento finché questo componente non viene rimosso. Quando il componente viene rimosso, viene rimosso anche l'intero sottoalbero DOM del componente.

```razor
<h1>Hello! This is a Blazor component rendered at @DateTime.Now</h1>

<div @ref="myElement"></div>

@code {
    HtmlElement myElement;
    
    protected override async Task OnAfterRenderAsync(bool firstRender)
    {
        if (firstRender)
        {
            ...
        }
    }
}
```

Per un esempio più dettagliato, si consideri il componente seguente che esegue il rendering di una mappa interattiva usando le [API MapBox Open Source](https://www.mapbox.com/):

```razor
@inject IJSRuntime JS
@implements IAsyncDisposable

<div @ref="mapElement" style='width: 400px; height: 300px;'></div>

<button @onclick="() => ShowAsync(51.454514, -2.587910)">Show Bristol, UK</button>
<button @onclick="() => ShowAsync(35.6762, 139.6503)">Show Tokyo, Japan</button>

@code
{
    ElementReference mapElement;
    IJSObjectReference mapModule;
    IJSObjectReference mapInstance;

    protected override async Task OnAfterRenderAsync(bool firstRender)
    {
        if (firstRender)
        {
            mapModule = await JS.InvokeAsync<IJSObjectReference>(
                "import", "./mapComponent.js");
            mapInstance = await mapModule.InvokeAsync<IJSObjectReference>(
                "addMapToElement", mapElement);
        }
    }

    Task ShowAsync(double latitude, double longitude)
        => mapModule.InvokeVoidAsync("setMapCenter", mapInstance, latitude, 
            longitude).AsTask();

    private async ValueTask IAsyncDisposable.DisposeAsync()
    {
        await mapInstance.DisposeAsync();
        await mapModule.DisposeAsync();
    }
}
```

Il modulo JavaScript corrispondente, che deve essere inserito in `wwwroot/mapComponent.js` , è il seguente:

```javascript
import 'https://api.mapbox.com/mapbox-gl-js/v1.12.0/mapbox-gl.js';

// TO MAKE THE MAP APPEAR YOU MUST ADD YOUR ACCESS TOKEN FROM 
// https://account.mapbox.com
mapboxgl.accessToken = '{ACCESS TOKEN}';

export function addMapToElement(element) {
  return new mapboxgl.Map({
    container: element,
    style: 'mapbox://styles/mapbox/streets-v11',
    center: [-74.5, 40],
    zoom: 9
  });
}

export function setMapCenter(map, latitude, longitude) {
  map.setCenter([longitude, latitude]);
}
```

Nell'esempio precedente, sostituire la stringa `{ACCESS TOKEN}` con un token di accesso valido che è possibile ottenere da https://account.mapbox.com .

Per produrre lo stile corretto, aggiungere il seguente tag del foglio di stile alla pagina HTML dell'host ( `index.html` o `_Host.cshtml` ):

```html
<link rel="stylesheet" href="https://api.mapbox.com/mapbox-gl-js/v1.12.0/mapbox-gl.css" />
```

Nell'esempio precedente viene prodotta un'interfaccia utente mappa interattiva, in cui l'utente:

* È possibile trascinare per scorrere o ingrandire.
* Fare clic sui pulsanti per passare a percorsi predefiniti.

![MapBox Street Map di Tokyo, Giappone con i pulsanti per selezionare Bristol, Regno Unito e Tokyo, Giappone](https://user-images.githubusercontent.com/1101362/94939821-92ef6700-04ca-11eb-858e-fff6df0053ae.png)

I punti chiave da comprendere sono:

 * Il `<div>` con `@ref="mapElement"` viene lasciato vuoto per quanto Blazor concerne. È pertanto sicuro per `mapbox-gl.js` compilarlo e modificarne il contenuto nel tempo. Questa tecnica può essere usata con qualsiasi libreria JavaScript che esegua il rendering dell'interfaccia utente. È anche possibile incorporare componenti da un Framework di JavaScript SPA di terze parti all'interno dei Blazor componenti, purché non tenti di raggiungere e modificare altre parti della pagina. *Non* è sicuro che il codice JavaScript esterno modifichi elementi che Blazor non sono considerati vuoti.
 * Quando si usa questo approccio, tenere presente le regole su come Blazor mantiene o Elimina gli elementi DOM. Nell'esempio precedente, il componente gestisce in modo sicuro gli eventi click del pulsante e aggiorna l'istanza della mappa esistente perché gli elementi DOM vengono conservati laddove possibile per impostazione predefinita. Se si esegue il rendering di un elenco di elementi della mappa dall'interno di un `@foreach` ciclo, si desidera utilizzare `@key` per garantire la conservazione delle istanze dei componenti. In caso contrario, le modifiche nei dati dell'elenco potrebbero causare la conservazione dello stato delle istanze precedenti in modo non auspicabile da parte delle istanze dei componenti. Per ulteriori informazioni, vedere [utilizzo @key di per mantenere elementi e componenti](xref:blazor/components/index#use-key-to-control-the-preservation-of-elements-and-components).

Inoltre, l'esempio precedente Mostra come è possibile incapsulare la logica JavaScript e le dipendenze all'interno di un modulo ES6 e caricarlo dinamicamente usando l' `import` identificatore. Per ulteriori informazioni, vedere [isolamento JavaScript e riferimenti a oggetti](#blazor-javascript-isolation-and-object-references).

::: moniker-end

## <a name="size-limits-on-js-interop-calls"></a>Limiti delle dimensioni per le chiamate di interoperabilità JS

In Blazor WebAssembly il Framework non impone un limite per le dimensioni degli input e degli output di interoperabilità js.

In Blazor Server le chiamate di interoperabilità JS hanno dimensioni limitate per la dimensione massima dei messaggi in ingresso SignalR consentita per i metodi dell'hub, che viene applicata da <xref:Microsoft.AspNetCore.SignalR.HubOptions.MaximumReceiveMessageSize?displayProperty=nameWithType> (valore predefinito: 32 KB). I messaggi da JS a .NET SignalR maggiori di <xref:Microsoft.AspNetCore.SignalR.HubOptions.MaximumReceiveMessageSize> generano un errore. Il Framework non impone un limite per le dimensioni di un SignalR messaggio dall'hub a un client. Per altre informazioni, vedere <xref:blazor/call-dotnet-from-javascript#size-limits-on-js-interop-calls>.
  
## <a name="js-modules"></a>Moduli JS

Per l'isolamento JS, l'interoperabilità JS funziona con il supporto predefinito del browser per i [moduli ECMAScript (ESM)](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Modules) ([specifica ECMAScript](https://tc39.es/ecma262/#sec-modules)).

## <a name="unmarshalled-js-interop"></a>Interoperabilità JS non Marshall

Blazor WebAssembly i componenti possono compromettere le prestazioni quando gli oggetti .NET vengono serializzati per l'interoperabilità JS e si verifica una delle condizioni seguenti:

* Un volume elevato di oggetti .NET viene serializzato in modo rapido. Esempio: le chiamate di interoperabilità JS vengono effettuate sulla base dello spostamento di un dispositivo di input, ad esempio la rotazione di una rotellina del mouse.
* Per l'interoperabilità JS è necessario serializzare oggetti .NET di grandi dimensioni o molti oggetti .NET. Esempio: le chiamate di interoperabilità JS richiedono la serializzazione di dozzine di file.

<xref:Microsoft.JSInterop.IJSUnmarshalledObjectReference> rappresenta un riferimento a un oggetto JavaScript le cui funzioni possono essere richiamate senza l'overhead della serializzazione dei dati .NET.

Nell'esempio seguente:

* Uno [struct](/dotnet/csharp/language-reference/builtin-types/struct) che contiene una stringa e un numero intero viene passato non serializzato a JavaScript.
* Le funzioni JavaScript elaborano i dati e restituiscono un valore booleano o una stringa al chiamante.
* Una stringa JavaScript non è direttamente convertibile in un `string` oggetto .NET. La `unmarshalledFunctionReturnString` funzione chiama `BINDING.js_string_to_mono_string` per gestire la conversione di una stringa JavaScript.

> [!NOTE]
> Gli esempi seguenti non sono casi d'uso tipici per questo scenario perché lo [struct](/dotnet/csharp/language-reference/builtin-types/struct) passato a JavaScript non comporta prestazioni di componenti insoddisfacenti. Nell'esempio viene utilizzato un piccolo oggetto solo per illustrare i concetti per il passaggio di dati .NET non serializzati.

Contenuto di un `<script>` blocco in `wwwroot/index.html` o di un file JavaScript esterno a cui fa riferimento `wwwroot/index.html` :

```javascript
window.returnJSObjectReference = () => {
    return {
        unmarshalledFunctionReturnBoolean: function (fields) {
            const name = Blazor.platform.readStringField(fields, 0);
            const year = Blazor.platform.readInt32Field(fields, 8);

            return name === "Brigadier Alistair Gordon Lethbridge-Stewart" &&
                year === 1968;
        },
        unmarshalledFunctionReturnString: function (fields) {
            const name = Blazor.platform.readStringField(fields, 0);
            const year = Blazor.platform.readInt32Field(fields, 8);

            return BINDING.js_string_to_mono_string(`Hello, ${name} (${year})!`);
        }
    };
}
```

> [!WARNING]
> Il `js_string_to_mono_string` nome della funzione, il comportamento e l'esistenza sono soggetti a modifiche in una versione futura di .NET. Ad esempio:
>
> * È probabile che la funzione venga rinominata.
> * La funzione stessa potrebbe essere rimossa a favore della conversione automatica delle stringhe da parte del Framework.

`Pages/UnmarshalledJSInterop.razor` (URL: `/unmarshalled-js-interop` ):

```razor
@page "/unmarshalled-js-interop"
@using System.Runtime.InteropServices
@using Microsoft.JSInterop
@inject IJSRuntime JS

<h1>Unmarshalled JS interop</h1>

@if (callResultForBoolean)
{
    <p>JS interop was successful!</p>
}

@if (!string.IsNullOrEmpty(callResultForString))
{
    <p>@callResultForString</p>
}

<p>
    <button @onclick="CallJSUnmarshalledForBoolean">
        Call Unmarshalled JS & Return Boolean
    </button>
    <button @onclick="CallJSUnmarshalledForString">
        Call Unmarshalled JS & Return String
    </button>
</p>

<p>
    <a href="https://www.doctorwho.tv">Doctor Who</a>
    is a registered trademark of the <a href="https://www.bbc.com/">BBC</a>.
</p>

@code {
    private bool callResultForBoolean;
    private string callResultForString;

    private void CallJSUnmarshalledForBoolean()
    {
        var unmarshalledRuntime = (IJSUnmarshalledRuntime)JS;

        var jsUnmarshalledReference = unmarshalledRuntime
            .InvokeUnmarshalled<IJSUnmarshalledObjectReference>(
                "returnJSObjectReference");

        callResultForBoolean = 
            jsUnmarshalledReference.InvokeUnmarshalled<InteropStruct, bool>(
                "unmarshalledFunctionReturnBoolean", GetStruct());
    }

    private void CallJSUnmarshalledForString()
    {
        var unmarshalledRuntime = (IJSUnmarshalledRuntime)JS;

        var jsUnmarshalledReference = unmarshalledRuntime
            .InvokeUnmarshalled<IJSUnmarshalledObjectReference>(
                "returnJSObjectReference");

        callResultForString = 
            jsUnmarshalledReference.InvokeUnmarshalled<InteropStruct, string>(
                "unmarshalledFunctionReturnString", GetStruct());
    }

    private InteropStruct GetStruct()
    {
        return new InteropStruct
        {
            Name = "Brigadier Alistair Gordon Lethbridge-Stewart",
            Year = 1968,
        };
    }

    [StructLayout(LayoutKind.Explicit)]
    public struct InteropStruct
    {
        [FieldOffset(0)]
        public string Name;

        [FieldOffset(8)]
        public int Year;
    }
}
```

Se un' `IJSUnmarshalledObjectReference` istanza non viene eliminata nel codice C#, può essere eliminata in JavaScript. La `dispose` funzione seguente elimina il riferimento all'oggetto quando viene chiamato da JavaScript:

```javascript
window.exampleJSObjectReferenceNotDisposedInCSharp = () => {
    return {
        dispose: function () {
            DotNet.disposeJSObjectReference(this);
        },

        ...
    };
}
```

I tipi di matrice possono essere convertiti da oggetti JavaScript in oggetti .NET utilizzando `js_typed_array_to_array` , ma la matrice JavaScript deve essere una matrice tipizzata. Le matrici da JavaScript possono essere lette nel codice C# come una matrice di oggetti .NET ( `object[]` ).

Altri tipi di dati, ad esempio matrici di stringhe, possono essere convertiti, ma è necessario creare un nuovo oggetto matrice mono ( `mono_obj_array_new` ) e impostarne il valore ( `mono_obj_array_set` ).

> [!WARNING]
> Le funzioni JavaScript fornite dal Blazor Framework, ad esempio `js_typed_array_to_array` , `mono_obj_array_new` e `mono_obj_array_set` , sono soggette a modifiche dei nomi, modifiche comportamentali o rimozione nelle versioni future di .NET.

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:blazor/call-dotnet-from-javascript>
* [Esempio di InteropComponent. Razor (repository GitHub DotNet/AspNetCore, Branch versione 3,1)](https://github.com/dotnet/AspNetCore/blob/release/3.1/src/Components/test/testassets/BasicTestApp/InteropComponent.razor)
