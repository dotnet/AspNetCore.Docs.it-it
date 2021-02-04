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
ms.openlocfilehash: ca42b611a61fc394655e396f914e8e050c578e6a
ms.sourcegitcommit: e311cfb77f26a0a23681019bd334929d1aaeda20
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/03/2021
ms.locfileid: "99530086"
---
# <a name="call-javascript-functions-from-net-methods-in-aspnet-core-blazor"></a><span data-ttu-id="827c5-103">Chiamare funzioni JavaScript da metodi .NET in ASP.NET Core Blazor</span><span class="sxs-lookup"><span data-stu-id="827c5-103">Call JavaScript functions from .NET methods in ASP.NET Core Blazor</span></span>

<span data-ttu-id="827c5-104">Di [Javier Calvarro Nelson](https://github.com/javiercn), [Daniel Roth](https://github.com/danroth27), [](https://github.com/pranavkm)Manuel e [Luke Latham](https://github.com/guardrex)</span><span class="sxs-lookup"><span data-stu-id="827c5-104">By [Javier Calvarro Nelson](https://github.com/javiercn), [Daniel Roth](https://github.com/danroth27), [Pranav Krishnamoorthy](https://github.com/pranavkm), and [Luke Latham](https://github.com/guardrex)</span></span>

<span data-ttu-id="827c5-105">Un' Blazor app può richiamare funzioni JavaScript da metodi .NET e metodi .NET da funzioni JavaScript.</span><span class="sxs-lookup"><span data-stu-id="827c5-105">A Blazor app can invoke JavaScript functions from .NET methods and .NET methods from JavaScript functions.</span></span> <span data-ttu-id="827c5-106">Questi scenari sono detti *interoperabilità JavaScript (interoperabilità* *JS*).</span><span class="sxs-lookup"><span data-stu-id="827c5-106">These scenarios are called *JavaScript interoperability* (*JS interop*).</span></span>

<span data-ttu-id="827c5-107">Questo articolo illustra come richiamare funzioni JavaScript da .NET.</span><span class="sxs-lookup"><span data-stu-id="827c5-107">This article covers invoking JavaScript functions from .NET.</span></span> <span data-ttu-id="827c5-108">Per informazioni su come chiamare i metodi .NET da JavaScript, vedere <xref:blazor/call-dotnet-from-javascript> .</span><span class="sxs-lookup"><span data-stu-id="827c5-108">For information on how to call .NET methods from JavaScript, see <xref:blazor/call-dotnet-from-javascript>.</span></span>

<span data-ttu-id="827c5-109">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/blazor/common/samples/) ([procedura per il download](xref:index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="827c5-109">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/blazor/common/samples/) ([how to download](xref:index#how-to-download-a-sample))</span></span>

> [!NOTE]
> <span data-ttu-id="827c5-110">Aggiungere i file JS ( `<script>` tag) prima del `</body>` tag di chiusura nel `wwwroot/index.html` file ( Blazor WebAssembly ) o nel `Pages/_Host.cshtml` file ( Blazor Server ).</span><span class="sxs-lookup"><span data-stu-id="827c5-110">Add JS files (`<script>` tags) before the closing `</body>` tag in the `wwwroot/index.html` file (Blazor WebAssembly) or `Pages/_Host.cshtml` file (Blazor Server).</span></span> <span data-ttu-id="827c5-111">Assicurarsi che i file JS con i metodi di interoperabilità JS siano inclusi prima dei Blazor file di framework JS.</span><span class="sxs-lookup"><span data-stu-id="827c5-111">Ensure that JS files with JS interop methods are included before Blazor framework JS files.</span></span>

<span data-ttu-id="827c5-112">Per chiamare JavaScript da .NET, usare l' <xref:Microsoft.JSInterop.IJSRuntime> astrazione.</span><span class="sxs-lookup"><span data-stu-id="827c5-112">To call into JavaScript from .NET, use the <xref:Microsoft.JSInterop.IJSRuntime> abstraction.</span></span> <span data-ttu-id="827c5-113">Per rilasciare chiamate di interoperabilità JS, inserire l' <xref:Microsoft.JSInterop.IJSRuntime> astrazione nel componente.</span><span class="sxs-lookup"><span data-stu-id="827c5-113">To issue JS interop calls, inject the <xref:Microsoft.JSInterop.IJSRuntime> abstraction in your component.</span></span> <span data-ttu-id="827c5-114"><xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A> accetta un identificatore per la funzione JavaScript che si vuole richiamare insieme a un numero qualsiasi di argomenti serializzabili in JSON.</span><span class="sxs-lookup"><span data-stu-id="827c5-114"><xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A> takes an identifier for the JavaScript function that you wish to invoke along with any number of JSON-serializable arguments.</span></span> <span data-ttu-id="827c5-115">L'identificatore della funzione è relativo all'ambito globale ( `window` ).</span><span class="sxs-lookup"><span data-stu-id="827c5-115">The function identifier is relative to the global scope (`window`).</span></span> <span data-ttu-id="827c5-116">Se si desidera chiamare `window.someScope.someFunction` , l'identificatore è `someScope.someFunction` .</span><span class="sxs-lookup"><span data-stu-id="827c5-116">If you wish to call `window.someScope.someFunction`, the identifier is `someScope.someFunction`.</span></span> <span data-ttu-id="827c5-117">Non è necessario registrare la funzione prima che venga chiamata.</span><span class="sxs-lookup"><span data-stu-id="827c5-117">There's no need to register the function before it's called.</span></span> <span data-ttu-id="827c5-118">Il tipo restituito `T` deve anche essere serializzabile in JSON.</span><span class="sxs-lookup"><span data-stu-id="827c5-118">The return type `T` must also be JSON serializable.</span></span> <span data-ttu-id="827c5-119">`T` deve corrispondere al tipo .NET che meglio esegue il mapping al tipo JSON restituito.</span><span class="sxs-lookup"><span data-stu-id="827c5-119">`T` should match the .NET type that best maps to the JSON type returned.</span></span>

<span data-ttu-id="827c5-120">Le funzioni JavaScript che restituiscono una [promessa](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Promise) vengono chiamate con <xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A> .</span><span class="sxs-lookup"><span data-stu-id="827c5-120">JavaScript functions that return a [Promise](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Promise) are called with <xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A>.</span></span> <span data-ttu-id="827c5-121">`InvokeAsync` rimuove il wrapping della promessa e restituisce il valore atteso dalla promessa.</span><span class="sxs-lookup"><span data-stu-id="827c5-121">`InvokeAsync` unwraps the Promise and returns the value awaited by the Promise.</span></span>

<span data-ttu-id="827c5-122">Per Blazor Server le app con il prerendering abilitato, non è possibile chiamare JavaScript durante il prerendering iniziale.</span><span class="sxs-lookup"><span data-stu-id="827c5-122">For Blazor Server apps with prerendering enabled, calling into JavaScript isn't possible during the initial prerendering.</span></span> <span data-ttu-id="827c5-123">È necessario rinviare le chiamate di interoperabilità JavaScript finché non viene stabilita la connessione con il browser.</span><span class="sxs-lookup"><span data-stu-id="827c5-123">JavaScript interop calls must be deferred until after the connection with the browser is established.</span></span> <span data-ttu-id="827c5-124">Per altre informazioni, vedere la sezione [rilevare quando un' Blazor Server app è prerendering](#detect-when-a-blazor-server-app-is-prerendering) .</span><span class="sxs-lookup"><span data-stu-id="827c5-124">For more information, see the [Detect when a Blazor Server app is prerendering](#detect-when-a-blazor-server-app-is-prerendering) section.</span></span>

<span data-ttu-id="827c5-125">L'esempio seguente è basato su [`TextDecoder`](https://developer.mozilla.org/docs/Web/API/TextDecoder) , un decodificatore basato su JavaScript.</span><span class="sxs-lookup"><span data-stu-id="827c5-125">The following example is based on [`TextDecoder`](https://developer.mozilla.org/docs/Web/API/TextDecoder), a JavaScript-based decoder.</span></span> <span data-ttu-id="827c5-126">Nell'esempio viene illustrato come richiamare una funzione JavaScript da un metodo C# che Scarica un requisito dal codice dello sviluppatore a un'API JavaScript esistente.</span><span class="sxs-lookup"><span data-stu-id="827c5-126">The example demonstrates how to invoke a JavaScript function from a C# method that offloads a requirement from developer code to an existing JavaScript API.</span></span> <span data-ttu-id="827c5-127">La funzione JavaScript accetta una matrice di byte da un metodo C#, decodifica la matrice e restituisce il testo al componente per la visualizzazione.</span><span class="sxs-lookup"><span data-stu-id="827c5-127">The JavaScript function accepts a byte array from a C# method, decodes the array, and returns the text to the component for display.</span></span>

<span data-ttu-id="827c5-128">All'interno dell' `<head>` elemento di `wwwroot/index.html` ( Blazor WebAssembly ) o `Pages/_Host.cshtml` ( Blazor Server ), fornire una funzione JavaScript che usi `TextDecoder` per decodificare una matrice passata e restituire il valore decodificato:</span><span class="sxs-lookup"><span data-stu-id="827c5-128">Inside the `<head>` element of `wwwroot/index.html` (Blazor WebAssembly) or `Pages/_Host.cshtml` (Blazor Server), provide a JavaScript function that uses `TextDecoder` to decode a passed array and return the decoded value:</span></span>

[!code-html[](call-javascript-from-dotnet/samples_snapshot/index-script-convertarray.html)]

<span data-ttu-id="827c5-129">Il codice JavaScript, ad esempio il codice illustrato nell'esempio precedente, può anche essere caricato da un file JavaScript ( `.js` ) con un riferimento al file script:</span><span class="sxs-lookup"><span data-stu-id="827c5-129">JavaScript code, such as the code shown in the preceding example, can also be loaded from a JavaScript file (`.js`) with a reference to the script file:</span></span>

```html
<script src="exampleJsInterop.js"></script>
```

<span data-ttu-id="827c5-130">Il componente seguente:</span><span class="sxs-lookup"><span data-stu-id="827c5-130">The following component:</span></span>

* <span data-ttu-id="827c5-131">Richiama la `convertArray` funzione JavaScript usando `JS` quando viene selezionato un pulsante del componente ( **`Convert Array`** ).</span><span class="sxs-lookup"><span data-stu-id="827c5-131">Invokes the `convertArray` JavaScript function using `JS` when a component button (**`Convert Array`**) is selected.</span></span>
* <span data-ttu-id="827c5-132">Dopo la chiamata della funzione JavaScript, la matrice passata viene convertita in una stringa.</span><span class="sxs-lookup"><span data-stu-id="827c5-132">After the JavaScript function is called, the passed array is converted into a string.</span></span> <span data-ttu-id="827c5-133">La stringa viene restituita al componente per la visualizzazione.</span><span class="sxs-lookup"><span data-stu-id="827c5-133">The string is returned to the component for display.</span></span>

[!code-razor[](call-javascript-from-dotnet/samples_snapshot/call-js-example.razor?highlight=2,34-35)]

## <a name="ijsruntime"></a><span data-ttu-id="827c5-134">IJSRuntime</span><span class="sxs-lookup"><span data-stu-id="827c5-134">IJSRuntime</span></span>

<span data-ttu-id="827c5-135">Per usare l' <xref:Microsoft.JSInterop.IJSRuntime> astrazione, adottare uno degli approcci seguenti:</span><span class="sxs-lookup"><span data-stu-id="827c5-135">To use the <xref:Microsoft.JSInterop.IJSRuntime> abstraction, adopt any of the following approaches:</span></span>

* <span data-ttu-id="827c5-136">Inserire l' <xref:Microsoft.JSInterop.IJSRuntime> astrazione nel Razor componente ( `.razor` ):</span><span class="sxs-lookup"><span data-stu-id="827c5-136">Inject the <xref:Microsoft.JSInterop.IJSRuntime> abstraction into the Razor component (`.razor`):</span></span>

  [!code-razor[](call-javascript-from-dotnet/samples_snapshot/inject-abstraction.razor?highlight=1)]

  <span data-ttu-id="827c5-137">All'interno dell' `<head>` elemento di `wwwroot/index.html` ( Blazor WebAssembly ) o `Pages/_Host.cshtml` ( Blazor Server ) specificare una `handleTickerChanged` funzione JavaScript.</span><span class="sxs-lookup"><span data-stu-id="827c5-137">Inside the `<head>` element of `wwwroot/index.html` (Blazor WebAssembly) or `Pages/_Host.cshtml` (Blazor Server), provide a `handleTickerChanged` JavaScript function.</span></span> <span data-ttu-id="827c5-138">La funzione viene chiamata con <xref:Microsoft.JSInterop.JSRuntimeExtensions.InvokeVoidAsync%2A?displayProperty=nameWithType> e non restituisce un valore:</span><span class="sxs-lookup"><span data-stu-id="827c5-138">The function is called with <xref:Microsoft.JSInterop.JSRuntimeExtensions.InvokeVoidAsync%2A?displayProperty=nameWithType> and doesn't return a value:</span></span>

  [!code-html[](call-javascript-from-dotnet/samples_snapshot/index-script-handleTickerChanged1.html)]

* <span data-ttu-id="827c5-139">Inserire l' <xref:Microsoft.JSInterop.IJSRuntime> astrazione in una classe ( `.cs` ):</span><span class="sxs-lookup"><span data-stu-id="827c5-139">Inject the <xref:Microsoft.JSInterop.IJSRuntime> abstraction into a class (`.cs`):</span></span>

  [!code-csharp[](call-javascript-from-dotnet/samples_snapshot/inject-abstraction-class.cs?highlight=5)]

  <span data-ttu-id="827c5-140">All'interno dell' `<head>` elemento di `wwwroot/index.html` ( Blazor WebAssembly ) o `Pages/_Host.cshtml` ( Blazor Server ) specificare una `handleTickerChanged` funzione JavaScript.</span><span class="sxs-lookup"><span data-stu-id="827c5-140">Inside the `<head>` element of `wwwroot/index.html` (Blazor WebAssembly) or `Pages/_Host.cshtml` (Blazor Server), provide a `handleTickerChanged` JavaScript function.</span></span> <span data-ttu-id="827c5-141">La funzione viene chiamata con `JS.InvokeAsync` e restituisce un valore:</span><span class="sxs-lookup"><span data-stu-id="827c5-141">The function is called with `JS.InvokeAsync` and returns a value:</span></span>

  [!code-html[](call-javascript-from-dotnet/samples_snapshot/index-script-handleTickerChanged2.html)]

* <span data-ttu-id="827c5-142">Per la generazione di contenuto dinamico con [BuildRenderTree](xref:blazor/advanced-scenarios#manual-rendertreebuilder-logic), usare l' `[Inject]` attributo:</span><span class="sxs-lookup"><span data-stu-id="827c5-142">For dynamic content generation with [BuildRenderTree](xref:blazor/advanced-scenarios#manual-rendertreebuilder-logic), use the `[Inject]` attribute:</span></span>

  ```razor
  [Inject]
  IJSRuntime JS { get; set; }
  ```

<span data-ttu-id="827c5-143">Nell'app di esempio lato client che accompagna questo argomento sono disponibili due funzioni JavaScript per l'app che interagiscono con il DOM per ricevere l'input dell'utente e visualizzare un messaggio di benvenuto:</span><span class="sxs-lookup"><span data-stu-id="827c5-143">In the client-side sample app that accompanies this topic, two JavaScript functions are available to the app that interact with the DOM to receive user input and display a welcome message:</span></span>

* <span data-ttu-id="827c5-144">`showPrompt`: Genera un messaggio di richiesta per accettare l'input dell'utente (il nome dell'utente) e restituisce il nome al chiamante.</span><span class="sxs-lookup"><span data-stu-id="827c5-144">`showPrompt`: Produces a prompt to accept user input (the user's name) and returns the name to the caller.</span></span>
* <span data-ttu-id="827c5-145">`displayWelcome`: Assegna un messaggio di benvenuto dal chiamante a un oggetto DOM con un valore `id` di `welcome` .</span><span class="sxs-lookup"><span data-stu-id="827c5-145">`displayWelcome`: Assigns a welcome message from the caller to a DOM object with an `id` of `welcome`.</span></span>

<span data-ttu-id="827c5-146">`wwwroot/exampleJsInterop.js`:</span><span class="sxs-lookup"><span data-stu-id="827c5-146">`wwwroot/exampleJsInterop.js`:</span></span>

[!code-javascript[](./common/samples/5.x/BlazorWebAssemblySample/wwwroot/exampleJsInterop.js?highlight=2-7)]

<span data-ttu-id="827c5-147">Inserire il `<script>` tag che fa riferimento al file JavaScript nel file `wwwroot/index.html` ( Blazor WebAssembly ) o nel `Pages/_Host.cshtml` file ( Blazor Server ).</span><span class="sxs-lookup"><span data-stu-id="827c5-147">Place the `<script>` tag that references the JavaScript file in the `wwwroot/index.html` file (Blazor WebAssembly) or `Pages/_Host.cshtml` file (Blazor Server).</span></span>

<span data-ttu-id="827c5-148">`wwwroot/index.html` (Blazor WebAssembly):</span><span class="sxs-lookup"><span data-stu-id="827c5-148">`wwwroot/index.html` (Blazor WebAssembly):</span></span>

[!code-html[](./common/samples/5.x/BlazorWebAssemblySample/wwwroot/index.html?highlight=22)]

<span data-ttu-id="827c5-149">`Pages/_Host.cshtml` (Blazor Server):</span><span class="sxs-lookup"><span data-stu-id="827c5-149">`Pages/_Host.cshtml` (Blazor Server):</span></span>

[!code-cshtml[](./common/samples/5.x/BlazorServerSample/Pages/_Host.cshtml?highlight=34)]

<span data-ttu-id="827c5-150">Non inserire un `<script>` tag in un file di componente perché il `<script>` tag non può essere aggiornato dinamicamente.</span><span class="sxs-lookup"><span data-stu-id="827c5-150">Don't place a `<script>` tag in a component file because the `<script>` tag can't be updated dynamically.</span></span>

<span data-ttu-id="827c5-151">I metodi .NET interoperano con le funzioni JavaScript nel `exampleJsInterop.js` file chiamando <xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A?displayProperty=nameWithType> .</span><span class="sxs-lookup"><span data-stu-id="827c5-151">.NET methods interop with the JavaScript functions in the `exampleJsInterop.js` file by calling <xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A?displayProperty=nameWithType>.</span></span>

<span data-ttu-id="827c5-152">L' <xref:Microsoft.JSInterop.IJSRuntime> astrazione è asincrona per consentire gli Blazor Server scenari.</span><span class="sxs-lookup"><span data-stu-id="827c5-152">The <xref:Microsoft.JSInterop.IJSRuntime> abstraction is asynchronous to allow for Blazor Server scenarios.</span></span> <span data-ttu-id="827c5-153">Se l'app è un' Blazor WebAssembly app e si vuole richiamare una funzione JavaScript in modo sincrono, <xref:Microsoft.JSInterop.IJSInProcessRuntime> abbandono e chiamano <xref:Microsoft.JSInterop.IJSInProcessRuntime.Invoke%2A> invece.</span><span class="sxs-lookup"><span data-stu-id="827c5-153">If the app is a Blazor WebAssembly app and you want to invoke a JavaScript function synchronously, downcast to <xref:Microsoft.JSInterop.IJSInProcessRuntime> and call <xref:Microsoft.JSInterop.IJSInProcessRuntime.Invoke%2A> instead.</span></span> <span data-ttu-id="827c5-154">È consigliabile che la maggior parte delle librerie di interoperabilità JS usi le API asincrone per assicurarsi che le librerie siano disponibili in tutti gli scenari.</span><span class="sxs-lookup"><span data-stu-id="827c5-154">We recommend that most JS interop libraries use the async APIs to ensure that the libraries are available in all scenarios.</span></span>

::: moniker range=">= aspnetcore-5.0"

> [!NOTE]
> <span data-ttu-id="827c5-155">Per abilitare l'isolamento JavaScript nei [moduli JavaScript](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Modules)standard, vedere la sezione [ Blazor riferimenti a oggetti e isolamento JavaScript](#blazor-javascript-isolation-and-object-references) .</span><span class="sxs-lookup"><span data-stu-id="827c5-155">To enable JavaScript isolation in standard [JavaScript modules](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Modules), see the [Blazor JavaScript isolation and object references](#blazor-javascript-isolation-and-object-references) section.</span></span>

::: moniker-end

<span data-ttu-id="827c5-156">L'app di esempio include un componente per dimostrare l'interoperabilità JS.</span><span class="sxs-lookup"><span data-stu-id="827c5-156">The sample app includes a component to demonstrate JS interop.</span></span> <span data-ttu-id="827c5-157">Il componente:</span><span class="sxs-lookup"><span data-stu-id="827c5-157">The component:</span></span>

* <span data-ttu-id="827c5-158">Riceve l'input dell'utente tramite un prompt di JavaScript.</span><span class="sxs-lookup"><span data-stu-id="827c5-158">Receives user input via a JavaScript prompt.</span></span>
* <span data-ttu-id="827c5-159">Restituisce il testo al componente per l'elaborazione.</span><span class="sxs-lookup"><span data-stu-id="827c5-159">Returns the text to the component for processing.</span></span>
* <span data-ttu-id="827c5-160">Chiama una seconda funzione JavaScript che interagisce con il DOM per visualizzare un messaggio di benvenuto.</span><span class="sxs-lookup"><span data-stu-id="827c5-160">Calls a second JavaScript function that interacts with the DOM to display a welcome message.</span></span>

<span data-ttu-id="827c5-161">`Pages/JsInterop.razor`:</span><span class="sxs-lookup"><span data-stu-id="827c5-161">`Pages/JsInterop.razor`:</span></span>

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

<span data-ttu-id="827c5-162">Il segnaposto `{APP ASSEMBLY}` è il nome dell'assembly dell'app (ad esempio, `BlazorSample` ).</span><span class="sxs-lookup"><span data-stu-id="827c5-162">The placeholder `{APP ASSEMBLY}` is the app's app assembly name (for example, `BlazorSample`).</span></span>

1. <span data-ttu-id="827c5-163">Quando `TriggerJsPrompt` viene eseguito selezionando il pulsante del componente **`Trigger JavaScript Prompt`** , `showPrompt` viene chiamata la funzione JavaScript fornita nel `wwwroot/exampleJsInterop.js` file.</span><span class="sxs-lookup"><span data-stu-id="827c5-163">When `TriggerJsPrompt` is executed by selecting the component's **`Trigger JavaScript Prompt`** button, the JavaScript `showPrompt` function provided in the `wwwroot/exampleJsInterop.js` file is called.</span></span>
1. <span data-ttu-id="827c5-164">La `showPrompt` funzione accetta l'input dell'utente (il nome dell'utente), che è codificato in HTML e restituito al componente.</span><span class="sxs-lookup"><span data-stu-id="827c5-164">The `showPrompt` function accepts user input (the user's name), which is HTML-encoded and returned to the component.</span></span> <span data-ttu-id="827c5-165">Il componente archivia il nome dell'utente in una variabile locale, `name` .</span><span class="sxs-lookup"><span data-stu-id="827c5-165">The component stores the user's name in a local variable, `name`.</span></span>
1. <span data-ttu-id="827c5-166">La stringa archiviata in `name` viene incorporata in un messaggio di benvenuto, che viene passato a una funzione JavaScript, `displayWelcome` , che esegue il rendering del messaggio di benvenuto in un tag di intestazione.</span><span class="sxs-lookup"><span data-stu-id="827c5-166">The string stored in `name` is incorporated into a welcome message, which is passed to a JavaScript function, `displayWelcome`, which renders the welcome message into a heading tag.</span></span>

## <a name="call-a-void-javascript-function"></a><span data-ttu-id="827c5-167">Chiamare una funzione JavaScript void</span><span class="sxs-lookup"><span data-stu-id="827c5-167">Call a void JavaScript function</span></span>

<span data-ttu-id="827c5-168">Utilizzare <xref:Microsoft.JSInterop.JSRuntimeExtensions.InvokeVoidAsync%2A?displayProperty=nameWithType> per gli elementi seguenti:</span><span class="sxs-lookup"><span data-stu-id="827c5-168">Use <xref:Microsoft.JSInterop.JSRuntimeExtensions.InvokeVoidAsync%2A?displayProperty=nameWithType> for the following:</span></span>

* <span data-ttu-id="827c5-169">Funzioni JavaScript che restituiscono [void (0)/void 0](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Operators/void) o [undefined](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/undefined).</span><span class="sxs-lookup"><span data-stu-id="827c5-169">JavaScript functions that return [void(0)/void 0](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Operators/void) or [undefined](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/undefined).</span></span>
* <span data-ttu-id="827c5-170">Se .NET non è necessario per leggere il risultato di una chiamata a JavaScript.</span><span class="sxs-lookup"><span data-stu-id="827c5-170">If .NET isn't required to read the result of a JavaScript call.</span></span>

## <a name="detect-when-a-blazor-server-app-is-prerendering"></a><span data-ttu-id="827c5-171">Rileva quando viene eseguito il Blazor Server prerendering di un'app</span><span class="sxs-lookup"><span data-stu-id="827c5-171">Detect when a Blazor Server app is prerendering</span></span>
 
[!INCLUDE[](~/blazor/includes/prerendering.md)]

## <a name="capture-references-to-elements"></a><span data-ttu-id="827c5-172">Acquisisci riferimenti a elementi</span><span class="sxs-lookup"><span data-stu-id="827c5-172">Capture references to elements</span></span>

<span data-ttu-id="827c5-173">Alcuni scenari di interoperabilità JS richiedono riferimenti agli elementi HTML.</span><span class="sxs-lookup"><span data-stu-id="827c5-173">Some JS interop scenarios require references to HTML elements.</span></span> <span data-ttu-id="827c5-174">Ad esempio, una libreria dell'interfaccia utente può richiedere un riferimento a un elemento per l'inizializzazione o potrebbe essere necessario chiamare API simili a quelle di un elemento, ad esempio `focus` o `play` .</span><span class="sxs-lookup"><span data-stu-id="827c5-174">For example, a UI library may require an element reference for initialization, or you might need to call command-like APIs on an element, such as `focus` or `play`.</span></span>

<span data-ttu-id="827c5-175">Acquisire i riferimenti agli elementi HTML in un componente usando l'approccio seguente:</span><span class="sxs-lookup"><span data-stu-id="827c5-175">Capture references to HTML elements in a component using the following approach:</span></span>

* <span data-ttu-id="827c5-176">Aggiungere un `@ref` attributo all'elemento HTML.</span><span class="sxs-lookup"><span data-stu-id="827c5-176">Add an `@ref` attribute to the HTML element.</span></span>
* <span data-ttu-id="827c5-177">Definire un campo di tipo <xref:Microsoft.AspNetCore.Components.ElementReference> il cui nome corrisponde al valore dell' `@ref` attributo.</span><span class="sxs-lookup"><span data-stu-id="827c5-177">Define a field of type <xref:Microsoft.AspNetCore.Components.ElementReference> whose name matches the value of the `@ref` attribute.</span></span>

<span data-ttu-id="827c5-178">Nell'esempio seguente viene illustrata l'acquisizione di un riferimento all' `username` `<input>` elemento:</span><span class="sxs-lookup"><span data-stu-id="827c5-178">The following example shows capturing a reference to the `username` `<input>` element:</span></span>

```razor
<input @ref="username" ... />

@code {
    ElementReference username;
}
```

> [!WARNING]
> <span data-ttu-id="827c5-179">Usare solo un riferimento a un elemento per mutare il contenuto di un elemento vuoto che non interagisce con Blazor .</span><span class="sxs-lookup"><span data-stu-id="827c5-179">Only use an element reference to mutate the contents of an empty element that doesn't interact with Blazor.</span></span> <span data-ttu-id="827c5-180">Questo scenario è utile quando un'API di terze parti fornisce contenuto all'elemento.</span><span class="sxs-lookup"><span data-stu-id="827c5-180">This scenario is useful when a third-party API supplies content to the element.</span></span> <span data-ttu-id="827c5-181">Poiché Blazor non interagisce con l'elemento, non è possibile che si verifichi un conflitto tra Blazor la rappresentazione dell'elemento e il Dom.</span><span class="sxs-lookup"><span data-stu-id="827c5-181">Because Blazor doesn't interact with the element, there's no possibility of a conflict between Blazor's representation of the element and the DOM.</span></span>
>
> <span data-ttu-id="827c5-182">Nell'esempio seguente è *pericoloso* modificare il contenuto dell'elenco non ordinato ( `ul` ) perché Blazor interagisce con il Dom per popolare gli elementi elenco di questo elemento ( `<li>` ):</span><span class="sxs-lookup"><span data-stu-id="827c5-182">In the following example, it's *dangerous* to mutate the contents of the unordered list (`ul`) because Blazor interacts with the DOM to populate this element's list items (`<li>`):</span></span>
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
> <span data-ttu-id="827c5-183">Se l'interoperabilità JS modifica il contenuto dell'elemento `MyList` e Blazor tenta di applicare differenze all'elemento, le differenze non corrispondono al Dom.</span><span class="sxs-lookup"><span data-stu-id="827c5-183">If JS interop mutates the contents of element `MyList` and Blazor attempts to apply diffs to the element, the diffs won't match the DOM.</span></span>

<span data-ttu-id="827c5-184">Una <xref:Microsoft.AspNetCore.Components.ElementReference> viene passata al codice JavaScript tramite l'interoperabilità js.</span><span class="sxs-lookup"><span data-stu-id="827c5-184">An <xref:Microsoft.AspNetCore.Components.ElementReference> is passed through to JavaScript code via JS interop.</span></span> <span data-ttu-id="827c5-185">Il codice JavaScript riceve un' `HTMLElement` istanza, che può essere usata con le API DOM normali.</span><span class="sxs-lookup"><span data-stu-id="827c5-185">The JavaScript code receives an `HTMLElement` instance, which it can use with normal DOM APIs.</span></span> <span data-ttu-id="827c5-186">Il codice seguente, ad esempio, definisce un metodo di estensione .NET che consente l'invio di un clic del mouse a un elemento:</span><span class="sxs-lookup"><span data-stu-id="827c5-186">For example, the following code defines a .NET extension method that enables sending a mouse click to an element:</span></span>

<span data-ttu-id="827c5-187">`exampleJsInterop.js`:</span><span class="sxs-lookup"><span data-stu-id="827c5-187">`exampleJsInterop.js`:</span></span>

```javascript
window.interopFunctions = {
  clickElement : function (element) {
    element.click();
  }
}
```

::: moniker range=">= aspnetcore-5.0"

> [!NOTE]
> <span data-ttu-id="827c5-188">Usare [`FocusAsync`](xref:blazor/components/event-handling#focus-an-element) il codice C# per concentrare un elemento, che è incorporato nel Blazor Framework e funziona con i riferimenti agli elementi.</span><span class="sxs-lookup"><span data-stu-id="827c5-188">Use [`FocusAsync`](xref:blazor/components/event-handling#focus-an-element) in C# code to focus an element, which is built-into the Blazor framework and works with element references.</span></span>

::: moniker-end

<span data-ttu-id="827c5-189">Per chiamare una funzione JavaScript che non restituisce un valore, usare <xref:Microsoft.JSInterop.JSRuntimeExtensions.InvokeVoidAsync%2A?displayProperty=nameWithType> .</span><span class="sxs-lookup"><span data-stu-id="827c5-189">To call a JavaScript function that doesn't return a value, use <xref:Microsoft.JSInterop.JSRuntimeExtensions.InvokeVoidAsync%2A?displayProperty=nameWithType>.</span></span> <span data-ttu-id="827c5-190">Il codice seguente attiva un evento sul lato client chiamando `Click` la funzione JavaScript precedente con l'acquisizione <xref:Microsoft.AspNetCore.Components.ElementReference> :</span><span class="sxs-lookup"><span data-stu-id="827c5-190">The following code triggers a client-side `Click` event by calling the preceding JavaScript function with the captured <xref:Microsoft.AspNetCore.Components.ElementReference>:</span></span>

[!code-razor[](call-javascript-from-dotnet/samples_snapshot/component1.razor?highlight=14-15)]

<span data-ttu-id="827c5-191">Per usare un metodo di estensione, creare un metodo di estensione statico che riceva l' <xref:Microsoft.JSInterop.IJSRuntime> istanza:</span><span class="sxs-lookup"><span data-stu-id="827c5-191">To use an extension method, create a static extension method that receives the <xref:Microsoft.JSInterop.IJSRuntime> instance:</span></span>

```csharp
public static async Task TriggerClickEvent(this ElementReference elementRef, 
    IJSRuntime js)
{
    await js.InvokeVoidAsync("interopFunctions.clickElement", elementRef);
}
```

<span data-ttu-id="827c5-192">Il `clickElement` metodo viene chiamato direttamente nell'oggetto.</span><span class="sxs-lookup"><span data-stu-id="827c5-192">The `clickElement` method is called directly on the object.</span></span> <span data-ttu-id="827c5-193">Nell'esempio seguente si presuppone che il `TriggerClickEvent` metodo sia disponibile dallo `JsInteropClasses` spazio dei nomi:</span><span class="sxs-lookup"><span data-stu-id="827c5-193">The following example assumes that the `TriggerClickEvent` method is available from the `JsInteropClasses` namespace:</span></span>

[!code-razor[](call-javascript-from-dotnet/samples_snapshot/component2.razor?highlight=15)]

> [!IMPORTANT]
> <span data-ttu-id="827c5-194">La `exampleButton` variabile viene popolata solo dopo il rendering del componente.</span><span class="sxs-lookup"><span data-stu-id="827c5-194">The `exampleButton` variable is only populated after the component is rendered.</span></span> <span data-ttu-id="827c5-195">Se un oggetto non popolato <xref:Microsoft.AspNetCore.Components.ElementReference> viene passato al codice JavaScript, il codice JavaScript riceve un valore di `null` .</span><span class="sxs-lookup"><span data-stu-id="827c5-195">If an unpopulated <xref:Microsoft.AspNetCore.Components.ElementReference> is passed to JavaScript code, the JavaScript code receives a value of `null`.</span></span> <span data-ttu-id="827c5-196">Per modificare i riferimenti agli elementi dopo che il componente ha terminato il rendering, usare i metodi del ciclo di vita dei [ `OnAfterRenderAsync` `OnAfterRender` componenti o](xref:blazor/components/lifecycle#after-component-render).</span><span class="sxs-lookup"><span data-stu-id="827c5-196">To manipulate element references after the component has finished rendering use the [`OnAfterRenderAsync` or `OnAfterRender` component lifecycle methods](xref:blazor/components/lifecycle#after-component-render).</span></span>

<span data-ttu-id="827c5-197">Quando si utilizzano tipi generici e si restituisce un valore, utilizzare <xref:System.Threading.Tasks.ValueTask%601> :</span><span class="sxs-lookup"><span data-stu-id="827c5-197">When working with generic types and returning a value, use <xref:System.Threading.Tasks.ValueTask%601>:</span></span>

```csharp
public static ValueTask<T> GenericMethod<T>(this ElementReference elementRef, 
    IJSRuntime js)
{
    return js.InvokeAsync<T>("exampleJsFunctions.doSomethingGeneric", elementRef);
}
```

<span data-ttu-id="827c5-198">`GenericMethod` viene chiamato direttamente sull'oggetto con un tipo.</span><span class="sxs-lookup"><span data-stu-id="827c5-198">`GenericMethod` is called directly on the object with a type.</span></span> <span data-ttu-id="827c5-199">Nell'esempio seguente si presuppone che l'oggetto `GenericMethod` sia disponibile dallo `JsInteropClasses` spazio dei nomi:</span><span class="sxs-lookup"><span data-stu-id="827c5-199">The following example assumes that the `GenericMethod` is available from the `JsInteropClasses` namespace:</span></span>

[!code-razor[](call-javascript-from-dotnet/samples_snapshot/component3.razor?highlight=17)]

## <a name="reference-elements-across-components"></a><span data-ttu-id="827c5-200">Elementi di riferimento tra i componenti</span><span class="sxs-lookup"><span data-stu-id="827c5-200">Reference elements across components</span></span>

<span data-ttu-id="827c5-201"><xref:Microsoft.AspNetCore.Components.ElementReference>Non è possibile passare un oggetto tra i componenti perché:</span><span class="sxs-lookup"><span data-stu-id="827c5-201">An <xref:Microsoft.AspNetCore.Components.ElementReference> can't be passed between components because:</span></span>

* <span data-ttu-id="827c5-202">L'istanza è garantita solo dopo il rendering del componente, ovvero durante o dopo l'esecuzione del metodo di un componente <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRender%2A> / <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A> .</span><span class="sxs-lookup"><span data-stu-id="827c5-202">The instance is only guaranteed to exist after the component is rendered, which is during or after a component's <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRender%2A>/<xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A> method executes.</span></span>
* <span data-ttu-id="827c5-203">Un oggetto <xref:Microsoft.AspNetCore.Components.ElementReference> è un oggetto [`struct`](/csharp/language-reference/builtin-types/struct) che non può essere passato come [parametro component](xref:blazor/components/index#component-parameters).</span><span class="sxs-lookup"><span data-stu-id="827c5-203">An <xref:Microsoft.AspNetCore.Components.ElementReference> is a [`struct`](/csharp/language-reference/builtin-types/struct), which can't be passed as a [component parameter](xref:blazor/components/index#component-parameters).</span></span>

<span data-ttu-id="827c5-204">Affinché un componente padre renda disponibile un riferimento a un elemento per altri componenti, il componente padre può:</span><span class="sxs-lookup"><span data-stu-id="827c5-204">For a parent component to make an element reference available to other components, the parent component can:</span></span>

* <span data-ttu-id="827c5-205">Consenti ai componenti figlio di registrare i callback.</span><span class="sxs-lookup"><span data-stu-id="827c5-205">Allow child components to register callbacks.</span></span>
* <span data-ttu-id="827c5-206">Richiamare i callback registrati durante l' <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRender%2A> evento con il riferimento all'elemento passato.</span><span class="sxs-lookup"><span data-stu-id="827c5-206">Invoke the registered callbacks during the <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRender%2A> event with the passed element reference.</span></span> <span data-ttu-id="827c5-207">Indirettamente, questo approccio consente ai componenti figlio di interagire con il riferimento all'elemento padre.</span><span class="sxs-lookup"><span data-stu-id="827c5-207">Indirectly, this approach allows child components to interact with the parent's element reference.</span></span>

<span data-ttu-id="827c5-208">Nell' Blazor WebAssembly esempio seguente viene illustrato l'approccio.</span><span class="sxs-lookup"><span data-stu-id="827c5-208">The following Blazor WebAssembly example illustrates the approach.</span></span>

<span data-ttu-id="827c5-209">Nella `<head>` di `wwwroot/index.html` :</span><span class="sxs-lookup"><span data-stu-id="827c5-209">In the `<head>` of `wwwroot/index.html`:</span></span>

```html
<style>
    .red { color: red }
</style>
```

<span data-ttu-id="827c5-210">Nella `<body>` di `wwwroot/index.html` :</span><span class="sxs-lookup"><span data-stu-id="827c5-210">In the `<body>` of `wwwroot/index.html`:</span></span>

```html
<script>
    function setElementClass(element, className) {
        /** @type {HTMLElement} **/
        var myElement = element;
        myElement.classList.add(className);
    }
</script>
```

<span data-ttu-id="827c5-211">`Pages/Index.razor` (componente padre):</span><span class="sxs-lookup"><span data-stu-id="827c5-211">`Pages/Index.razor` (parent component):</span></span>

```razor
@page "/"

<h1 @ref="title">Hello, world!</h1>

Welcome to your new app.

<SurveyPrompt Parent="this" Title="How is Blazor working for you?" />
```

<span data-ttu-id="827c5-212">`Pages/Index.razor.cs`:</span><span class="sxs-lookup"><span data-stu-id="827c5-212">`Pages/Index.razor.cs`:</span></span>

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

<span data-ttu-id="827c5-213">Il segnaposto `{APP ASSEMBLY}` è il nome dell'assembly dell'app (ad esempio, `BlazorSample` ).</span><span class="sxs-lookup"><span data-stu-id="827c5-213">The placeholder `{APP ASSEMBLY}` is the app's app assembly name (for example, `BlazorSample`).</span></span>

<span data-ttu-id="827c5-214">`Shared/SurveyPrompt.razor` (componente figlio):</span><span class="sxs-lookup"><span data-stu-id="827c5-214">`Shared/SurveyPrompt.razor` (child component):</span></span>

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

<span data-ttu-id="827c5-215">`Shared/SurveyPrompt.razor.cs`:</span><span class="sxs-lookup"><span data-stu-id="827c5-215">`Shared/SurveyPrompt.razor.cs`:</span></span>

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

<span data-ttu-id="827c5-216">Il segnaposto `{APP ASSEMBLY}` è il nome dell'assembly dell'app (ad esempio, `BlazorSample` ).</span><span class="sxs-lookup"><span data-stu-id="827c5-216">The placeholder `{APP ASSEMBLY}` is the app's app assembly name (for example, `BlazorSample`).</span></span>

## <a name="harden-js-interop-calls"></a><span data-ttu-id="827c5-217">Chiamate di interoperabilità JS di Harden</span><span class="sxs-lookup"><span data-stu-id="827c5-217">Harden JS interop calls</span></span>

<span data-ttu-id="827c5-218">L'interoperabilità JS potrebbe non riuscire a causa di errori di rete e deve essere considerata inaffidabile.</span><span class="sxs-lookup"><span data-stu-id="827c5-218">JS interop may fail due to networking errors and should be treated as unreliable.</span></span> <span data-ttu-id="827c5-219">Per impostazione predefinita, un' Blazor Server app timeout delle chiamate di interoperabilità js nel server dopo un minuto.</span><span class="sxs-lookup"><span data-stu-id="827c5-219">By default, a Blazor Server app times out JS interop calls on the server after one minute.</span></span> <span data-ttu-id="827c5-220">Se un'app può tollerare un timeout più aggressivo, impostare il timeout usando uno degli approcci seguenti:</span><span class="sxs-lookup"><span data-stu-id="827c5-220">If an app can tolerate a more aggressive timeout, set the timeout using one of the following approaches:</span></span>

* <span data-ttu-id="827c5-221">A livello globale in `Startup.ConfigureServices` specificare il timeout:</span><span class="sxs-lookup"><span data-stu-id="827c5-221">Globally in `Startup.ConfigureServices`, specify the timeout:</span></span>

  ```csharp
  services.AddServerSideBlazor(
      options => options.JSInteropDefaultCallTimeout = TimeSpan.FromSeconds({SECONDS}));
  ```

* <span data-ttu-id="827c5-222">Per chiamata nel codice componente, una singola chiamata può specificare il timeout:</span><span class="sxs-lookup"><span data-stu-id="827c5-222">Per-invocation in component code, a single call can specify the timeout:</span></span>

  ```csharp
  var result = await JS.InvokeAsync<string>("MyJSOperation", 
      TimeSpan.FromSeconds({SECONDS}), new[] { "Arg1" });
  ```

<span data-ttu-id="827c5-223">Per ulteriori informazioni sull'esaurimento delle risorse, vedere <xref:blazor/security/server/threat-mitigation> .</span><span class="sxs-lookup"><span data-stu-id="827c5-223">For more information on resource exhaustion, see <xref:blazor/security/server/threat-mitigation>.</span></span>

[!INCLUDE[](~/blazor/includes/share-interop-code.md)]

## <a name="avoid-circular-object-references"></a><span data-ttu-id="827c5-224">Evitare riferimenti a oggetti circolari</span><span class="sxs-lookup"><span data-stu-id="827c5-224">Avoid circular object references</span></span>

<span data-ttu-id="827c5-225">Gli oggetti che contengono riferimenti circolari non possono essere serializzati sul client per uno dei seguenti:</span><span class="sxs-lookup"><span data-stu-id="827c5-225">Objects that contain circular references can't be serialized on the client for either:</span></span>

* <span data-ttu-id="827c5-226">Chiamate al metodo .NET.</span><span class="sxs-lookup"><span data-stu-id="827c5-226">.NET method calls.</span></span>
* <span data-ttu-id="827c5-227">Chiamate al metodo JavaScript da C# quando il tipo restituito contiene riferimenti circolari.</span><span class="sxs-lookup"><span data-stu-id="827c5-227">JavaScript method calls from C# when the return type has circular references.</span></span>

<span data-ttu-id="827c5-228">Per altre informazioni, vedere [riferimenti circolari non supportati. prendere due (DotNet/aspnetcore #20525)](https://github.com/dotnet/aspnetcore/issues/20525).</span><span class="sxs-lookup"><span data-stu-id="827c5-228">For more information, see [Circular references are not supported, take two (dotnet/aspnetcore #20525)](https://github.com/dotnet/aspnetcore/issues/20525).</span></span>

::: moniker range=">= aspnetcore-5.0"

## <a name="blazor-javascript-isolation-and-object-references"></a><span data-ttu-id="827c5-229">Blazor Isolamento JavaScript e riferimenti a oggetti</span><span class="sxs-lookup"><span data-stu-id="827c5-229">Blazor JavaScript isolation and object references</span></span>

<span data-ttu-id="827c5-230">Blazor Abilita l'isolamento JavaScript nei [moduli JavaScript](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Modules)standard.</span><span class="sxs-lookup"><span data-stu-id="827c5-230">Blazor enables JavaScript isolation in standard [JavaScript modules](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Modules).</span></span> <span data-ttu-id="827c5-231">L'isolamento JavaScript offre i vantaggi seguenti:</span><span class="sxs-lookup"><span data-stu-id="827c5-231">JavaScript isolation provides the following benefits:</span></span>

* <span data-ttu-id="827c5-232">Il codice JavaScript importato non inquina più lo spazio dei nomi globale.</span><span class="sxs-lookup"><span data-stu-id="827c5-232">Imported JavaScript no longer pollutes the global namespace.</span></span>
* <span data-ttu-id="827c5-233">I consumer di una libreria e i componenti non sono necessari per importare il codice JavaScript correlato.</span><span class="sxs-lookup"><span data-stu-id="827c5-233">Consumers of a library and components aren't required to import the related JavaScript.</span></span>

<span data-ttu-id="827c5-234">Il modulo JavaScript seguente, ad esempio, esporta una funzione JavaScript per visualizzare un prompt del browser:</span><span class="sxs-lookup"><span data-stu-id="827c5-234">For example, the following JavaScript module exports a JavaScript function for showing a browser prompt:</span></span>

```javascript
export function showPrompt(message) {
  return prompt(message, 'Type anything here');
}
```

<span data-ttu-id="827c5-235">Aggiungere il modulo JavaScript precedente a una libreria .NET come asset Web statico ( `wwwroot/exampleJsInterop.js` ) e quindi importare il modulo nel codice .NET chiamando <xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A> sul <xref:Microsoft.JSInterop.IJSRuntime> servizio.</span><span class="sxs-lookup"><span data-stu-id="827c5-235">Add the preceding JavaScript module to a .NET library as a static web asset (`wwwroot/exampleJsInterop.js`) and then import the module into the .NET code by calling <xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A> on the <xref:Microsoft.JSInterop.IJSRuntime> service.</span></span> <span data-ttu-id="827c5-236">Il servizio viene inserito come `js` (non visualizzato) per l'esempio seguente:</span><span class="sxs-lookup"><span data-stu-id="827c5-236">The service is injected as `js` (not shown) for the following example:</span></span>

```csharp
var module = await js.InvokeAsync<IJSObjectReference>(
    "import", "./_content/MyComponents/exampleJsInterop.js");
```

<span data-ttu-id="827c5-237">L' `import` identificatore nell'esempio precedente è un identificatore speciale usato in modo specifico per l'importazione di un modulo JavaScript.</span><span class="sxs-lookup"><span data-stu-id="827c5-237">The `import` identifier in the preceding example is a special identifier used specifically for importing a JavaScript module.</span></span> <span data-ttu-id="827c5-238">Specificare il modulo usando il percorso di asset Web statico stabile: `./_content/{LIBRARY NAME}/{PATH UNDER WWWROOT}` .</span><span class="sxs-lookup"><span data-stu-id="827c5-238">Specify the module using its stable static web asset path: `./_content/{LIBRARY NAME}/{PATH UNDER WWWROOT}`.</span></span> <span data-ttu-id="827c5-239">Il segmento di percorso per la directory corrente ( `./` ) è necessario per creare il percorso Asset statico corretto per il file JavaScript.</span><span class="sxs-lookup"><span data-stu-id="827c5-239">The path segment for the current directory (`./`) is required in order to create the correct static asset path to the JavaScript file.</span></span> <span data-ttu-id="827c5-240">L'importazione dinamica di un modulo richiede una richiesta di rete, pertanto può essere eseguita solo in modo asincrono chiamando <xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A> .</span><span class="sxs-lookup"><span data-stu-id="827c5-240">Dynamically importing a module requires a network request, so it can only be achieved asynchronously by calling <xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A>.</span></span> <span data-ttu-id="827c5-241">Il `{LIBRARY NAME}` segnaposto è il nome della libreria.</span><span class="sxs-lookup"><span data-stu-id="827c5-241">The `{LIBRARY NAME}` placeholder is the library name.</span></span> <span data-ttu-id="827c5-242">Il `{PATH UNDER WWWROOT}` segnaposto è il percorso dello script in `wwwroot` .</span><span class="sxs-lookup"><span data-stu-id="827c5-242">The `{PATH UNDER WWWROOT}` placeholder is the path to the script under `wwwroot`.</span></span>

<span data-ttu-id="827c5-243"><xref:Microsoft.JSInterop.IJSRuntime> importa il modulo come `IJSObjectReference` , che rappresenta un riferimento a un oggetto JavaScript dal codice .NET.</span><span class="sxs-lookup"><span data-stu-id="827c5-243"><xref:Microsoft.JSInterop.IJSRuntime> imports the module as a `IJSObjectReference`, which represents a reference to a JavaScript object from .NET code.</span></span> <span data-ttu-id="827c5-244">Usare `IJSObjectReference` per richiamare le funzioni JavaScript esportate dal modulo:</span><span class="sxs-lookup"><span data-stu-id="827c5-244">Use the `IJSObjectReference` to invoke exported JavaScript functions from the module:</span></span>

```csharp
public async ValueTask<string> Prompt(string message)
{
    return await module.InvokeAsync<string>("showPrompt", message);
}
```

<span data-ttu-id="827c5-245">`IJSInProcessObjectReference` rappresenta un riferimento a un oggetto JavaScript le cui funzioni possono essere richiamate in modo sincrono.</span><span class="sxs-lookup"><span data-stu-id="827c5-245">`IJSInProcessObjectReference` represents a reference to a JavaScript object whose functions can be invoked synchronously.</span></span>

## <a name="use-of-javascript-libraries-that-render-ui-dom-elements"></a><span data-ttu-id="827c5-246">Uso di librerie JavaScript che eseguono il rendering dell'interfaccia utente (elementi DOM)</span><span class="sxs-lookup"><span data-stu-id="827c5-246">Use of JavaScript libraries that render UI (DOM elements)</span></span>

<span data-ttu-id="827c5-247">In alcuni casi può essere utile usare librerie JavaScript che producono elementi dell'interfaccia utente visibili all'interno del DOM del browser.</span><span class="sxs-lookup"><span data-stu-id="827c5-247">Sometimes you may wish to use JavaScript libraries that produce visible user interface elements within the browser DOM.</span></span> <span data-ttu-id="827c5-248">A prima vista, questo potrebbe sembrare difficile perché il Blazor sistema di differenziazione si basa sul controllo della struttura ad albero di elementi DOM e si verificano errori se il codice esterno muta l'albero DOM e invalida il meccanismo per l'applicazione di diff.</span><span class="sxs-lookup"><span data-stu-id="827c5-248">At first glance, this might seem difficult because Blazor's diffing system relies on having control over the tree of DOM elements and runs into errors if some external code mutates the DOM tree and invalidates its mechanism for applying diffs.</span></span> <span data-ttu-id="827c5-249">Questa limitazione non è Blazor specifica.</span><span class="sxs-lookup"><span data-stu-id="827c5-249">This isn't a Blazor-specific limitation.</span></span> <span data-ttu-id="827c5-250">Lo stesso problema si verifica con qualsiasi framework dell'interfaccia utente basato su diff.</span><span class="sxs-lookup"><span data-stu-id="827c5-250">The same challenge occurs with any diff-based UI framework.</span></span>

<span data-ttu-id="827c5-251">Fortunatamente, è facile incorporare in modo affidabile l'interfaccia utente generata dall'esterno all'interno di un' Blazor interfaccia utente del componente.</span><span class="sxs-lookup"><span data-stu-id="827c5-251">Fortunately, it's straightforward to embed externally-generated UI within a Blazor component UI reliably.</span></span> <span data-ttu-id="827c5-252">La tecnica consigliata consiste nel fare in modo che il codice del componente ( `.razor` file) produca un elemento vuoto.</span><span class="sxs-lookup"><span data-stu-id="827c5-252">The recommended technique is to have the component's code (`.razor` file) produce an empty element.</span></span> <span data-ttu-id="827c5-253">Per quanto Blazor riguarda il sistema diffing, l'elemento è sempre vuoto, quindi il renderer non esegue la ricorsione nell'elemento e lascia il proprio contenuto.</span><span class="sxs-lookup"><span data-stu-id="827c5-253">As far as Blazor's diffing system is concerned, the element is always empty, so the renderer does not recurse into the element and instead leaves its contents alone.</span></span> <span data-ttu-id="827c5-254">In questo modo è possibile popolare l'elemento con contenuto arbitrario gestito esternamente.</span><span class="sxs-lookup"><span data-stu-id="827c5-254">This makes it safe to populate the element with arbitrary externally-managed content.</span></span>

<span data-ttu-id="827c5-255">Nell'esempio seguente viene illustrato il concetto.</span><span class="sxs-lookup"><span data-stu-id="827c5-255">The following example demonstrates the concept.</span></span> <span data-ttu-id="827c5-256">All'interno dell' `if` istruzione quando `firstRender` è `true` , eseguire un'operazione con `myElement` .</span><span class="sxs-lookup"><span data-stu-id="827c5-256">Within the `if` statement when `firstRender` is `true`, do something with `myElement`.</span></span> <span data-ttu-id="827c5-257">Ad esempio, chiamare una libreria JavaScript esterna per popolarla.</span><span class="sxs-lookup"><span data-stu-id="827c5-257">For example, call an external JavaScript library to populate it.</span></span> <span data-ttu-id="827c5-258">Blazor lascia solo il contenuto dell'elemento finché questo componente non viene rimosso.</span><span class="sxs-lookup"><span data-stu-id="827c5-258">Blazor leaves the element's contents alone until this component itself is removed.</span></span> <span data-ttu-id="827c5-259">Quando il componente viene rimosso, viene rimosso anche l'intero sottoalbero DOM del componente.</span><span class="sxs-lookup"><span data-stu-id="827c5-259">When the component is removed, the component's entire DOM subtree is also removed.</span></span>

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

<span data-ttu-id="827c5-260">Per un esempio più dettagliato, si consideri il componente seguente che esegue il rendering di una mappa interattiva usando le [API MapBox Open Source](https://www.mapbox.com/):</span><span class="sxs-lookup"><span data-stu-id="827c5-260">As a more detailed example, consider the following component that renders an interactive map using the [open-source Mapbox APIs](https://www.mapbox.com/):</span></span>

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

<span data-ttu-id="827c5-261">Il modulo JavaScript corrispondente, che deve essere inserito in `wwwroot/mapComponent.js` , è il seguente:</span><span class="sxs-lookup"><span data-stu-id="827c5-261">The corresponding JavaScript module, which should be placed at `wwwroot/mapComponent.js`, is as follows:</span></span>

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

<span data-ttu-id="827c5-262">Nell'esempio precedente, sostituire la stringa `{ACCESS TOKEN}` con un token di accesso valido che è possibile ottenere da https://account.mapbox.com .</span><span class="sxs-lookup"><span data-stu-id="827c5-262">In the preceding example, replace the string `{ACCESS TOKEN}` with a valid access token that you can get from https://account.mapbox.com.</span></span>

<span data-ttu-id="827c5-263">Per produrre lo stile corretto, aggiungere il seguente tag del foglio di stile alla pagina HTML dell'host ( `index.html` o `_Host.cshtml` ):</span><span class="sxs-lookup"><span data-stu-id="827c5-263">To produce correct styling, add the following stylesheet tag to the host HTML page (`index.html` or `_Host.cshtml`):</span></span>

```html
<link rel="stylesheet" href="https://api.mapbox.com/mapbox-gl-js/v1.12.0/mapbox-gl.css" />
```

<span data-ttu-id="827c5-264">Nell'esempio precedente viene prodotta un'interfaccia utente mappa interattiva, in cui l'utente:</span><span class="sxs-lookup"><span data-stu-id="827c5-264">The preceding example produces an interactive map UI, in which the user:</span></span>

* <span data-ttu-id="827c5-265">È possibile trascinare per scorrere o ingrandire.</span><span class="sxs-lookup"><span data-stu-id="827c5-265">Can drag to scroll or zoom.</span></span>
* <span data-ttu-id="827c5-266">Fare clic sui pulsanti per passare a percorsi predefiniti.</span><span class="sxs-lookup"><span data-stu-id="827c5-266">Click buttons to jump to predefined locations.</span></span>

![MapBox Street Map di Tokyo, Giappone con i pulsanti per selezionare Bristol, Regno Unito e Tokyo, Giappone](https://user-images.githubusercontent.com/1101362/94939821-92ef6700-04ca-11eb-858e-fff6df0053ae.png)

<span data-ttu-id="827c5-268">I punti chiave da comprendere sono:</span><span class="sxs-lookup"><span data-stu-id="827c5-268">The key points to understand are:</span></span>

 * <span data-ttu-id="827c5-269">Il `<div>` con `@ref="mapElement"` viene lasciato vuoto per quanto Blazor concerne.</span><span class="sxs-lookup"><span data-stu-id="827c5-269">The `<div>` with `@ref="mapElement"` is left empty as far as Blazor is concerned.</span></span> <span data-ttu-id="827c5-270">È pertanto sicuro per `mapbox-gl.js` compilarlo e modificarne il contenuto nel tempo.</span><span class="sxs-lookup"><span data-stu-id="827c5-270">It's therefore safe for `mapbox-gl.js` to populate it and modify its contents over time.</span></span> <span data-ttu-id="827c5-271">Questa tecnica può essere usata con qualsiasi libreria JavaScript che esegua il rendering dell'interfaccia utente.</span><span class="sxs-lookup"><span data-stu-id="827c5-271">You can use this technique with any JavaScript library that renders UI.</span></span> <span data-ttu-id="827c5-272">È anche possibile incorporare componenti da un Framework di JavaScript SPA di terze parti all'interno dei Blazor componenti, purché non tenti di raggiungere e modificare altre parti della pagina.</span><span class="sxs-lookup"><span data-stu-id="827c5-272">You could even embed components from a third-party JavaScript SPA framework inside Blazor components, as long as they don't try to reach out and modify other parts of the page.</span></span> <span data-ttu-id="827c5-273">*Non* è sicuro che il codice JavaScript esterno modifichi elementi che Blazor non sono considerati vuoti.</span><span class="sxs-lookup"><span data-stu-id="827c5-273">It is *not* safe for external JavaScript code to modify elements that Blazor does not regard as empty.</span></span>
 * <span data-ttu-id="827c5-274">Quando si usa questo approccio, tenere presente le regole su come Blazor mantiene o Elimina gli elementi DOM.</span><span class="sxs-lookup"><span data-stu-id="827c5-274">When using this approach, bear in mind the rules about how Blazor retains or destroys DOM elements.</span></span> <span data-ttu-id="827c5-275">Nell'esempio precedente, il componente gestisce in modo sicuro gli eventi click del pulsante e aggiorna l'istanza della mappa esistente perché gli elementi DOM vengono conservati laddove possibile per impostazione predefinita.</span><span class="sxs-lookup"><span data-stu-id="827c5-275">In the preceding example, the component safely handles button click events and updates the existing map instance because DOM elements are retained where possible by default.</span></span> <span data-ttu-id="827c5-276">Se si esegue il rendering di un elenco di elementi della mappa dall'interno di un `@foreach` ciclo, si desidera utilizzare `@key` per garantire la conservazione delle istanze dei componenti.</span><span class="sxs-lookup"><span data-stu-id="827c5-276">If you were rendering a list of map elements from inside a `@foreach` loop, you want to use `@key` to ensure the preservation of component instances.</span></span> <span data-ttu-id="827c5-277">In caso contrario, le modifiche nei dati dell'elenco potrebbero causare la conservazione dello stato delle istanze precedenti in modo non auspicabile da parte delle istanze dei componenti.</span><span class="sxs-lookup"><span data-stu-id="827c5-277">Otherwise, changes in the list data could cause component instances to retain the state of previous instances in an undesirable manner.</span></span> <span data-ttu-id="827c5-278">Per ulteriori informazioni, vedere [utilizzo @key di per mantenere elementi e componenti](xref:blazor/components/index#use-key-to-control-the-preservation-of-elements-and-components).</span><span class="sxs-lookup"><span data-stu-id="827c5-278">For more information, see [using @key to preserve elements and components](xref:blazor/components/index#use-key-to-control-the-preservation-of-elements-and-components).</span></span>

<span data-ttu-id="827c5-279">Inoltre, l'esempio precedente Mostra come è possibile incapsulare la logica JavaScript e le dipendenze all'interno di un modulo ES6 e caricarlo dinamicamente usando l' `import` identificatore.</span><span class="sxs-lookup"><span data-stu-id="827c5-279">Additionally, the preceding example shows how it's possible to encapsulate JavaScript logic and dependencies within an ES6 module and load it dynamically using the `import` identifier.</span></span> <span data-ttu-id="827c5-280">Per ulteriori informazioni, vedere [isolamento JavaScript e riferimenti a oggetti](#blazor-javascript-isolation-and-object-references).</span><span class="sxs-lookup"><span data-stu-id="827c5-280">For more information, see [JavaScript isolation and object references](#blazor-javascript-isolation-and-object-references).</span></span>

::: moniker-end

## <a name="size-limits-on-js-interop-calls"></a><span data-ttu-id="827c5-281">Limiti delle dimensioni per le chiamate di interoperabilità JS</span><span class="sxs-lookup"><span data-stu-id="827c5-281">Size limits on JS interop calls</span></span>

<span data-ttu-id="827c5-282">In Blazor WebAssembly il Framework non impone un limite per le dimensioni degli input e degli output di interoperabilità js.</span><span class="sxs-lookup"><span data-stu-id="827c5-282">In Blazor WebAssembly, the framework doesn't impose a limit on the size of JS interop inputs and outputs.</span></span>

<span data-ttu-id="827c5-283">In Blazor Server le chiamate di interoperabilità JS hanno dimensioni limitate per la dimensione massima dei messaggi in ingresso SignalR consentita per i metodi dell'hub, che viene applicata da <xref:Microsoft.AspNetCore.SignalR.HubOptions.MaximumReceiveMessageSize?displayProperty=nameWithType> (valore predefinito: 32 KB).</span><span class="sxs-lookup"><span data-stu-id="827c5-283">In Blazor Server, JS interop calls are limited in size by the maximum incoming SignalR message size permitted for hub methods, which is enforced by <xref:Microsoft.AspNetCore.SignalR.HubOptions.MaximumReceiveMessageSize?displayProperty=nameWithType> (default: 32 KB).</span></span> <span data-ttu-id="827c5-284">I messaggi da JS a .NET SignalR maggiori di <xref:Microsoft.AspNetCore.SignalR.HubOptions.MaximumReceiveMessageSize> generano un errore.</span><span class="sxs-lookup"><span data-stu-id="827c5-284">JS to .NET SignalR messages larger than <xref:Microsoft.AspNetCore.SignalR.HubOptions.MaximumReceiveMessageSize> throw an error.</span></span> <span data-ttu-id="827c5-285">Il Framework non impone un limite per le dimensioni di un SignalR messaggio dall'hub a un client.</span><span class="sxs-lookup"><span data-stu-id="827c5-285">The framework doesn't impose a limit on the size of a SignalR message from the hub to a client.</span></span> <span data-ttu-id="827c5-286">Per altre informazioni, vedere <xref:blazor/call-dotnet-from-javascript#size-limits-on-js-interop-calls>.</span><span class="sxs-lookup"><span data-stu-id="827c5-286">For more information, see <xref:blazor/call-dotnet-from-javascript#size-limits-on-js-interop-calls>.</span></span>
  
## <a name="js-modules"></a><span data-ttu-id="827c5-287">Moduli JS</span><span class="sxs-lookup"><span data-stu-id="827c5-287">JS modules</span></span>

<span data-ttu-id="827c5-288">Per l'isolamento JS, l'interoperabilità JS funziona con il supporto predefinito del browser per i [moduli ECMAScript (ESM)](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Modules) ([specifica ECMAScript](https://tc39.es/ecma262/#sec-modules)).</span><span class="sxs-lookup"><span data-stu-id="827c5-288">For JS isolation, JS interop works with the browser's default support for [EcmaScript modules (ESM)](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Modules) ([ECMAScript specification](https://tc39.es/ecma262/#sec-modules)).</span></span>

## <a name="unmarshalled-js-interop"></a><span data-ttu-id="827c5-289">Interoperabilità JS non Marshall</span><span class="sxs-lookup"><span data-stu-id="827c5-289">Unmarshalled JS interop</span></span>

<span data-ttu-id="827c5-290">Blazor WebAssembly i componenti possono compromettere le prestazioni quando gli oggetti .NET vengono serializzati per l'interoperabilità JS e si verifica una delle condizioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="827c5-290">Blazor WebAssembly components may experience poor performance when .NET objects are serialized for JS interop and either of the following are true:</span></span>

* <span data-ttu-id="827c5-291">Un volume elevato di oggetti .NET viene serializzato in modo rapido.</span><span class="sxs-lookup"><span data-stu-id="827c5-291">A high volume of .NET objects are rapidly serialized.</span></span> <span data-ttu-id="827c5-292">Esempio: le chiamate di interoperabilità JS vengono effettuate sulla base dello spostamento di un dispositivo di input, ad esempio la rotazione di una rotellina del mouse.</span><span class="sxs-lookup"><span data-stu-id="827c5-292">Example: JS interop calls are made on the basis of moving an input device, such as spinning a mouse wheel.</span></span>
* <span data-ttu-id="827c5-293">Per l'interoperabilità JS è necessario serializzare oggetti .NET di grandi dimensioni o molti oggetti .NET.</span><span class="sxs-lookup"><span data-stu-id="827c5-293">Large .NET objects or many .NET objects must be serialized for JS interop.</span></span> <span data-ttu-id="827c5-294">Esempio: le chiamate di interoperabilità JS richiedono la serializzazione di dozzine di file.</span><span class="sxs-lookup"><span data-stu-id="827c5-294">Example: JS interop calls require serializing dozens of files.</span></span>

<span data-ttu-id="827c5-295"><xref:Microsoft.JSInterop.IJSUnmarshalledObjectReference> rappresenta un riferimento a un oggetto JavaScript le cui funzioni possono essere richiamate senza l'overhead della serializzazione dei dati .NET.</span><span class="sxs-lookup"><span data-stu-id="827c5-295"><xref:Microsoft.JSInterop.IJSUnmarshalledObjectReference> represents a reference to an JavaScript object whose functions can be invoked without the overhead of serializing .NET data.</span></span>

<span data-ttu-id="827c5-296">Nell'esempio seguente:</span><span class="sxs-lookup"><span data-stu-id="827c5-296">In the following example:</span></span>

* <span data-ttu-id="827c5-297">Uno [struct](/dotnet/csharp/language-reference/builtin-types/struct) che contiene una stringa e un numero intero viene passato non serializzato a JavaScript.</span><span class="sxs-lookup"><span data-stu-id="827c5-297">A [struct](/dotnet/csharp/language-reference/builtin-types/struct) containing a string and an integer is passed unserialized to JavaScript.</span></span>
* <span data-ttu-id="827c5-298">Le funzioni JavaScript elaborano i dati e restituiscono un valore booleano o una stringa al chiamante.</span><span class="sxs-lookup"><span data-stu-id="827c5-298">JavaScript functions process the data and return either a boolean or string to the caller.</span></span>
* <span data-ttu-id="827c5-299">Una stringa JavaScript non è direttamente convertibile in un `string` oggetto .NET.</span><span class="sxs-lookup"><span data-stu-id="827c5-299">A JavaScript string isn't directly convertible into a .NET `string` object.</span></span> <span data-ttu-id="827c5-300">La `unmarshalledFunctionReturnString` funzione chiama `BINDING.js_string_to_mono_string` per gestire la conversione di una stringa JavaScript.</span><span class="sxs-lookup"><span data-stu-id="827c5-300">The `unmarshalledFunctionReturnString` function calls `BINDING.js_string_to_mono_string` to manage the conversion of a Javascript string.</span></span>

> [!NOTE]
> <span data-ttu-id="827c5-301">Gli esempi seguenti non sono casi d'uso tipici per questo scenario perché lo [struct](/dotnet/csharp/language-reference/builtin-types/struct) passato a JavaScript non comporta prestazioni di componenti insoddisfacenti.</span><span class="sxs-lookup"><span data-stu-id="827c5-301">The following examples aren't typical use cases for this scenario because the [struct](/dotnet/csharp/language-reference/builtin-types/struct) passed to JavaScript doesn't result in poor component performance.</span></span> <span data-ttu-id="827c5-302">Nell'esempio viene utilizzato un piccolo oggetto solo per illustrare i concetti per il passaggio di dati .NET non serializzati.</span><span class="sxs-lookup"><span data-stu-id="827c5-302">The example uses a small object merely to demonstrate the concepts for passing unserialized .NET data.</span></span>

<span data-ttu-id="827c5-303">Contenuto di un `<script>` blocco in `wwwroot/index.html` o di un file JavaScript esterno a cui fa riferimento `wwwroot/index.html` :</span><span class="sxs-lookup"><span data-stu-id="827c5-303">Content of a `<script>` block in `wwwroot/index.html` or an external Javascript file referenced by `wwwroot/index.html`:</span></span>

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
> <span data-ttu-id="827c5-304">Il `js_string_to_mono_string` nome della funzione, il comportamento e l'esistenza sono soggetti a modifiche in una versione futura di .NET.</span><span class="sxs-lookup"><span data-stu-id="827c5-304">The `js_string_to_mono_string` function name, behavior, and existence is subject to change in a future release of .NET.</span></span> <span data-ttu-id="827c5-305">Ad esempio:</span><span class="sxs-lookup"><span data-stu-id="827c5-305">For example:</span></span>
>
> * <span data-ttu-id="827c5-306">È probabile che la funzione venga rinominata.</span><span class="sxs-lookup"><span data-stu-id="827c5-306">The function is likely to be renamed.</span></span>
> * <span data-ttu-id="827c5-307">La funzione stessa potrebbe essere rimossa a favore della conversione automatica delle stringhe da parte del Framework.</span><span class="sxs-lookup"><span data-stu-id="827c5-307">The function itself might be removed in favor of automatic conversion of strings by the framework.</span></span>

<span data-ttu-id="827c5-308">`Pages/UnmarshalledJSInterop.razor` (URL: `/unmarshalled-js-interop` ):</span><span class="sxs-lookup"><span data-stu-id="827c5-308">`Pages/UnmarshalledJSInterop.razor` (URL: `/unmarshalled-js-interop`):</span></span>

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

<span data-ttu-id="827c5-309">Se un' `IJSUnmarshalledObjectReference` istanza non viene eliminata nel codice C#, può essere eliminata in JavaScript.</span><span class="sxs-lookup"><span data-stu-id="827c5-309">If an `IJSUnmarshalledObjectReference` instance isn't disposed in C# code, it can be disposed in JavaScript.</span></span> <span data-ttu-id="827c5-310">La `dispose` funzione seguente elimina il riferimento all'oggetto quando viene chiamato da JavaScript:</span><span class="sxs-lookup"><span data-stu-id="827c5-310">The following `dispose` function disposes the object reference when called from JavaScript:</span></span>

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

<span data-ttu-id="827c5-311">I tipi di matrice possono essere convertiti da oggetti JavaScript in oggetti .NET utilizzando `js_typed_array_to_array` , ma la matrice JavaScript deve essere una matrice tipizzata.</span><span class="sxs-lookup"><span data-stu-id="827c5-311">Array types can be converted from JavaScript objects into .NET objects using `js_typed_array_to_array`, but the JavaScript array must be a typed array.</span></span> <span data-ttu-id="827c5-312">Le matrici da JavaScript possono essere lette nel codice C# come una matrice di oggetti .NET ( `object[]` ).</span><span class="sxs-lookup"><span data-stu-id="827c5-312">Arrays from JavaScript can be read in C# code as a .NET object array (`object[]`).</span></span>

<span data-ttu-id="827c5-313">Altri tipi di dati, ad esempio matrici di stringhe, possono essere convertiti, ma è necessario creare un nuovo oggetto matrice mono ( `mono_obj_array_new` ) e impostarne il valore ( `mono_obj_array_set` ).</span><span class="sxs-lookup"><span data-stu-id="827c5-313">Other data types, such as string arrays, can be converted but require creating a new Mono array object (`mono_obj_array_new`) and setting its value (`mono_obj_array_set`).</span></span>

> [!WARNING]
> <span data-ttu-id="827c5-314">Le funzioni JavaScript fornite dal Blazor Framework, ad esempio `js_typed_array_to_array` , `mono_obj_array_new` e `mono_obj_array_set` , sono soggette a modifiche dei nomi, modifiche comportamentali o rimozione nelle versioni future di .NET.</span><span class="sxs-lookup"><span data-stu-id="827c5-314">JavaScript functions provided by the Blazor framework, such as `js_typed_array_to_array`, `mono_obj_array_new`, and `mono_obj_array_set`, are subject to name changes, behavioral changes, or removal in future releases of .NET.</span></span>

## <a name="additional-resources"></a><span data-ttu-id="827c5-315">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="827c5-315">Additional resources</span></span>

* <xref:blazor/call-dotnet-from-javascript>
* [<span data-ttu-id="827c5-316">Esempio di InteropComponent. Razor (repository GitHub DotNet/AspNetCore, Branch versione 3,1)</span><span class="sxs-lookup"><span data-stu-id="827c5-316">InteropComponent.razor example (dotnet/AspNetCore GitHub repository, 3.1 release branch)</span></span>](https://github.com/dotnet/AspNetCore/blob/release/3.1/src/Components/test/testassets/BasicTestApp/InteropComponent.razor)
