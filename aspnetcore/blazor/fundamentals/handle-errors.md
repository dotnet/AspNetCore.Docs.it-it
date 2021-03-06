---
title: Gestione degli errori nelle Blazor app ASP.NET Core
author: guardrex
description: Scopri in che modo ASP.NET Core Blazor come Blazor gestisce le eccezioni non gestite e come sviluppare app che rilevano e gestiscono gli errori.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 02/25/2021
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
zone_pivot_groups: blazor-hosting-models
ms.openlocfilehash: d4c8054afc3818d58bc2a047a0aa74613ae6047b
ms.sourcegitcommit: 1436bd4d70937d6ec3140da56d96caab33c4320b
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/06/2021
ms.locfileid: "102395097"
---
# <a name="handle-errors-in-aspnet-core-blazor-apps"></a><span data-ttu-id="a92d0-103">Gestione degli errori nelle Blazor app ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="a92d0-103">Handle errors in ASP.NET Core Blazor apps</span></span>

<span data-ttu-id="a92d0-104">Questo articolo descrive come Blazor gestire le eccezioni non gestite e come sviluppare app che rilevano e gestiscono gli errori.</span><span class="sxs-lookup"><span data-stu-id="a92d0-104">This article describes how Blazor manages unhandled exceptions and how to develop apps that detect and handle errors.</span></span>

::: zone pivot="webassembly"

## <a name="detailed-errors-during-development"></a><span data-ttu-id="a92d0-105">Errori dettagliati durante lo sviluppo</span><span class="sxs-lookup"><span data-stu-id="a92d0-105">Detailed errors during development</span></span>

<span data-ttu-id="a92d0-106">Quando un' Blazor app non funziona correttamente durante lo sviluppo, la ricezione di informazioni dettagliate sugli errori dall'app è utile per la risoluzione dei problemi e per risolvere il problema.</span><span class="sxs-lookup"><span data-stu-id="a92d0-106">When a Blazor app isn't functioning properly during development, receiving detailed error information from the app assists in troubleshooting and fixing the issue.</span></span> <span data-ttu-id="a92d0-107">Quando si verifica un errore, le Blazor app visualizzano una barra gialla chiara nella parte inferiore della schermata:</span><span class="sxs-lookup"><span data-stu-id="a92d0-107">When an error occurs, Blazor apps display a light yellow bar at the bottom of the screen:</span></span>

* <span data-ttu-id="a92d0-108">Durante lo sviluppo, la barra indirizza l'utente alla console del browser, in cui è possibile visualizzare l'eccezione.</span><span class="sxs-lookup"><span data-stu-id="a92d0-108">During development, the bar directs you to the browser console, where you can see the exception.</span></span>
* <span data-ttu-id="a92d0-109">In produzione, la barra informa l'utente che si è verificato un errore e consiglia di aggiornare il browser.</span><span class="sxs-lookup"><span data-stu-id="a92d0-109">In production, the bar notifies the user that an error has occurred and recommends refreshing the browser.</span></span>

<span data-ttu-id="a92d0-110">L'interfaccia utente per questa esperienza di gestione degli errori fa parte dei [ Blazor modelli di progetto](xref:blazor/project-structure).</span><span class="sxs-lookup"><span data-stu-id="a92d0-110">The UI for this error handling experience is part of the [Blazor project templates](xref:blazor/project-structure).</span></span>

<span data-ttu-id="a92d0-111">In un' Blazor WebAssembly app, personalizzare l'esperienza nel `wwwroot/index.html` file:</span><span class="sxs-lookup"><span data-stu-id="a92d0-111">In a Blazor WebAssembly app, customize the experience in the `wwwroot/index.html` file:</span></span>

```html
<div id="blazor-error-ui">
    An unhandled error has occurred.
    <a href="" class="reload">Reload</a>
    <a class="dismiss">🗙</a>
</div>
```

<span data-ttu-id="a92d0-112">L' `blazor-error-ui` elemento è generalmente nascosto a causa della presenza dello `display: none` stile della `blazor-error-ui` classe CSS nel foglio di stile dell'app ( `wwwroot/css/app.css` ).</span><span class="sxs-lookup"><span data-stu-id="a92d0-112">The `blazor-error-ui` element is normally hidden due the presence of the `display: none` style of the `blazor-error-ui` CSS class in the app's stylesheet (`wwwroot/css/app.css`).</span></span> <span data-ttu-id="a92d0-113">Quando si verifica un errore, il Framework viene applicato `display: block` all'elemento.</span><span class="sxs-lookup"><span data-stu-id="a92d0-113">When an error occurs, the framework applies `display: block` to the element.</span></span>

## <a name="manage-unhandled-exceptions-in-developer-code"></a><span data-ttu-id="a92d0-114">Gestire le eccezioni non gestite nel codice dello sviluppatore</span><span class="sxs-lookup"><span data-stu-id="a92d0-114">Manage unhandled exceptions in developer code</span></span>

<span data-ttu-id="a92d0-115">Per continuare l'applicazione dopo un errore, l'app deve avere la logica di gestione degli errori.</span><span class="sxs-lookup"><span data-stu-id="a92d0-115">For an app to continue after an error, the app must have error handling logic.</span></span> <span data-ttu-id="a92d0-116">Le sezioni successive di questo articolo descrivono le potenziali fonti di eccezioni non gestite.</span><span class="sxs-lookup"><span data-stu-id="a92d0-116">Later sections of this article describe potential sources of unhandled exceptions.</span></span>

<span data-ttu-id="a92d0-117">In produzione, non eseguire il rendering dei messaggi di eccezione del Framework o delle analisi dello stack nell'interfaccia utente.</span><span class="sxs-lookup"><span data-stu-id="a92d0-117">In production, don't render framework exception messages or stack traces in the UI.</span></span> <span data-ttu-id="a92d0-118">Il rendering dei messaggi di eccezione o delle analisi dello stack potrebbe:</span><span class="sxs-lookup"><span data-stu-id="a92d0-118">Rendering exception messages or stack traces could:</span></span>

* <span data-ttu-id="a92d0-119">Divulgare informazioni riservate agli utenti finali.</span><span class="sxs-lookup"><span data-stu-id="a92d0-119">Disclose sensitive information to end users.</span></span>
* <span data-ttu-id="a92d0-120">Aiutare un utente malintenzionato a individuare i punti deboli in un'app che può compromettere la sicurezza dell'app, del server o della rete.</span><span class="sxs-lookup"><span data-stu-id="a92d0-120">Help a malicious user discover weaknesses in an app that can compromise the security of the app, server, or network.</span></span>

## <a name="global-exception-handling"></a><span data-ttu-id="a92d0-121">Gestione delle eccezioni globali</span><span class="sxs-lookup"><span data-stu-id="a92d0-121">Global exception handling</span></span>

[!INCLUDE[](~/blazor/includes/handle-errors/global-exception-handling.md)]

## <a name="log-errors-with-a-persistent-provider"></a><span data-ttu-id="a92d0-122">Registrare gli errori con un provider persistente</span><span class="sxs-lookup"><span data-stu-id="a92d0-122">Log errors with a persistent provider</span></span>

<span data-ttu-id="a92d0-123">Se si verifica un'eccezione non gestita, l'eccezione viene registrata <xref:Microsoft.Extensions.Logging.ILogger> nelle istanze configurate nel contenitore dei servizi.</span><span class="sxs-lookup"><span data-stu-id="a92d0-123">If an unhandled exception occurs, the exception is logged to <xref:Microsoft.Extensions.Logging.ILogger> instances configured in the service container.</span></span> <span data-ttu-id="a92d0-124">Per impostazione predefinita, Blazor le app registrano nell'output della console con il provider di registrazione della console.</span><span class="sxs-lookup"><span data-stu-id="a92d0-124">By default, Blazor apps log to console output with the Console Logging Provider.</span></span> <span data-ttu-id="a92d0-125">Prendere in considerazione la registrazione a una posizione più permanente sul server inviando informazioni sugli errori a un'API Web back-end che usa un provider di registrazione con la gestione delle dimensioni del log e la rotazione del log.</span><span class="sxs-lookup"><span data-stu-id="a92d0-125">Consider logging to a more permanent location on the server by sending error information to a backend web API that uses a logging provider with log size management and log rotation.</span></span> <span data-ttu-id="a92d0-126">In alternativa, l'app per le API Web back-end può usare un servizio di gestione delle prestazioni delle applicazioni (APM), ad esempio [applicazione Azure &dagger; Insights (monitoraggio di Azure)](/azure/azure-monitor/app/app-insights-overview), per registrare le informazioni sugli errori ricevute dai client.</span><span class="sxs-lookup"><span data-stu-id="a92d0-126">Alternatively, the backend web API app can use an Application Performance Management (APM) service, such as [Azure Application Insights (Azure Monitor)&dagger;](/azure/azure-monitor/app/app-insights-overview), to record error information that it receives from clients.</span></span>

<span data-ttu-id="a92d0-127">È necessario decidere quali eventi imprevisti registrare e il livello di gravità degli eventi imprevisti registrati.</span><span class="sxs-lookup"><span data-stu-id="a92d0-127">You must decide which incidents to log and the level of severity of logged incidents.</span></span> <span data-ttu-id="a92d0-128">Gli utenti ostili potrebbero essere in grado di attivare intenzionalmente gli errori.</span><span class="sxs-lookup"><span data-stu-id="a92d0-128">Hostile users might be able to trigger errors deliberately.</span></span> <span data-ttu-id="a92d0-129">Ad esempio, non registrare un evento imprevisto da un errore in cui viene fornito un oggetto sconosciuto `ProductId` nell'URL di un componente che Visualizza i dettagli del prodotto.</span><span class="sxs-lookup"><span data-stu-id="a92d0-129">For example, don't log an incident from an error where an unknown `ProductId` is supplied in the URL of a component that displays product details.</span></span> <span data-ttu-id="a92d0-130">Non tutti gli errori devono essere considerati come eventi imprevisti per la registrazione.</span><span class="sxs-lookup"><span data-stu-id="a92d0-130">Not all errors should be treated as incidents for logging.</span></span>

<span data-ttu-id="a92d0-131">Per altre informazioni, vedere gli articoli seguenti:</span><span class="sxs-lookup"><span data-stu-id="a92d0-131">For more information, see the following articles:</span></span>

* <xref:blazor/fundamentals/logging>
* <xref:fundamentals/error-handling>&Dagger;
* <xref:web-api/index>

<span data-ttu-id="a92d0-132">&dagger;Le funzionalità di [Application Insights](/azure/azure-monitor/app/app-insights-overview) native per supportare le Blazor WebAssembly app e il Blazor supporto del Framework nativo per [Google Analytics](https://analytics.google.com/analytics/web/) potrebbero diventare disponibili nelle versioni future di queste tecnologie.</span><span class="sxs-lookup"><span data-stu-id="a92d0-132">&dagger;Native [Application Insights](/azure/azure-monitor/app/app-insights-overview) features to support Blazor WebAssembly apps and native Blazor framework support for [Google Analytics](https://analytics.google.com/analytics/web/) might become available in future releases of these technologies.</span></span> <span data-ttu-id="a92d0-133">Per altre informazioni, vedere [supportare App Insights nel Blazor lato client di WASM (Microsoft/ApplicationInsights-DotNet #2143)](https://github.com/microsoft/ApplicationInsights-dotnet/issues/2143) e [analisi e diagnostica Web (include i collegamenti alle implementazioni della community) (DotNet/aspnetcore #5461)](https://github.com/dotnet/aspnetcore/issues/5461).</span><span class="sxs-lookup"><span data-stu-id="a92d0-133">For more information, see [Support App Insights in Blazor WASM Client Side (microsoft/ApplicationInsights-dotnet #2143)](https://github.com/microsoft/ApplicationInsights-dotnet/issues/2143) and [Web analytics and diagnostics (includes links to community implementations) (dotnet/aspnetcore #5461)](https://github.com/dotnet/aspnetcore/issues/5461).</span></span> <span data-ttu-id="a92d0-134">Nel frattempo, un'app lato client Blazor WebAssembly può usare l' [SDK Application Insights JavaScript](/azure/azure-monitor/app/javascript) con l' [interoperabilità JS](xref:blazor/call-javascript-from-dotnet) per registrare gli errori direttamente Application Insights da un'app sul lato client.</span><span class="sxs-lookup"><span data-stu-id="a92d0-134">In the meantime, a client-side Blazor WebAssembly app can use the [Application Insights JavaScript SDK](/azure/azure-monitor/app/javascript) with [JS interop](xref:blazor/call-javascript-from-dotnet) to log errors directly to Application Insights from a client-side app.</span></span>

<span data-ttu-id="a92d0-135">&Dagger;Si applica alle app ASP.NET Core lato server che sono app back-end dell'API Web per le Blazor app.</span><span class="sxs-lookup"><span data-stu-id="a92d0-135">&Dagger;Applies to server-side ASP.NET Core apps that are web API backend apps for Blazor apps.</span></span> <span data-ttu-id="a92d0-136">Le app lato client trap e inviano informazioni sugli errori a un'API Web, che registra le informazioni sull'errore in un provider di registrazione permanente.</span><span class="sxs-lookup"><span data-stu-id="a92d0-136">Client-side apps trap and send error information to a web API, which logs the error information to a persistent logging provider.</span></span>

## <a name="places-where-errors-may-occur"></a><span data-ttu-id="a92d0-137">Posizioni in cui possono verificarsi errori</span><span class="sxs-lookup"><span data-stu-id="a92d0-137">Places where errors may occur</span></span>

<span data-ttu-id="a92d0-138">Il Framework e il codice dell'app possono attivare eccezioni non gestite in uno dei percorsi seguenti, descritti più avanti nelle sezioni seguenti di questo articolo:</span><span class="sxs-lookup"><span data-stu-id="a92d0-138">Framework and app code may trigger unhandled exceptions in any of the following locations, which are described further in the following sections of this article:</span></span>

* [<span data-ttu-id="a92d0-139">Creazione di istanze di componenti</span><span class="sxs-lookup"><span data-stu-id="a92d0-139">Component instantiation</span></span>](#component-instantiation-webassembly)
* [<span data-ttu-id="a92d0-140">Metodi del ciclo di vita</span><span class="sxs-lookup"><span data-stu-id="a92d0-140">Lifecycle methods</span></span>](#lifecycle-methods-webassembly)
* [<span data-ttu-id="a92d0-141">Logica di rendering</span><span class="sxs-lookup"><span data-stu-id="a92d0-141">Rendering logic</span></span>](#rendering-logic-webassembly)
* [<span data-ttu-id="a92d0-142">Gestori eventi</span><span class="sxs-lookup"><span data-stu-id="a92d0-142">Event handlers</span></span>](#event-handlers-webassembly)
* [<span data-ttu-id="a92d0-143">Eliminazione componenti</span><span class="sxs-lookup"><span data-stu-id="a92d0-143">Component disposal</span></span>](#component-disposal-webassembly)
* [<span data-ttu-id="a92d0-144">Interoperabilità JavaScript</span><span class="sxs-lookup"><span data-stu-id="a92d0-144">JavaScript interop</span></span>](#javascript-interop-webassembly)

<h3 id="component-instantiation-webassembly"><span data-ttu-id="a92d0-145">Creazione di istanze di componenti</span><span class="sxs-lookup"><span data-stu-id="a92d0-145">Component instantiation</span></span></h3>

<span data-ttu-id="a92d0-146">Quando Blazor Crea un'istanza di un componente:</span><span class="sxs-lookup"><span data-stu-id="a92d0-146">When Blazor creates an instance of a component:</span></span>

* <span data-ttu-id="a92d0-147">Il costruttore del componente viene richiamato.</span><span class="sxs-lookup"><span data-stu-id="a92d0-147">The component's constructor is invoked.</span></span>
* <span data-ttu-id="a92d0-148">Vengono richiamati i costruttori di tutti i servizi non singleton forniti al costruttore del componente tramite la [`@inject`](xref:mvc/views/razor#inject) direttiva o l' [ `[Inject]` attributo](xref:blazor/fundamentals/dependency-injection#request-a-service-in-a-component) .</span><span class="sxs-lookup"><span data-stu-id="a92d0-148">The constructors of any non-singleton DI services supplied to the component's constructor via the [`@inject`](xref:mvc/views/razor#inject) directive or the [`[Inject]` attribute](xref:blazor/fundamentals/dependency-injection#request-a-service-in-a-component) are invoked.</span></span>

<span data-ttu-id="a92d0-149">Un errore in un costruttore eseguito o un setter per qualsiasi `[Inject]` proprietà comporta un'eccezione non gestita e impedisce al Framework di creare un'istanza del componente.</span><span class="sxs-lookup"><span data-stu-id="a92d0-149">An error in an executed constructor or a setter for any `[Inject]` property results in an unhandled exception and stops the framework from instantiating the component.</span></span> <span data-ttu-id="a92d0-150">Se la logica del costruttore può generare eccezioni, l'app deve intercettare le eccezioni usando un' [`try-catch`](/dotnet/csharp/language-reference/keywords/try-catch) istruzione con gestione e registrazione degli errori.</span><span class="sxs-lookup"><span data-stu-id="a92d0-150">If constructor logic may throw exceptions, the app should trap the exceptions using a [`try-catch`](/dotnet/csharp/language-reference/keywords/try-catch) statement with error handling and logging.</span></span>

<h3 id="lifecycle-methods-webassembly"><span data-ttu-id="a92d0-151">Metodi del ciclo di vita</span><span class="sxs-lookup"><span data-stu-id="a92d0-151">Lifecycle methods</span></span></h3>

<span data-ttu-id="a92d0-152">Durante la durata di un componente, Blazor richiama i [metodi del ciclo](xref:blazor/components/lifecycle#lifecycle-methods)di vita.</span><span class="sxs-lookup"><span data-stu-id="a92d0-152">During the lifetime of a component, Blazor invokes [lifecycle methods](xref:blazor/components/lifecycle#lifecycle-methods).</span></span> <span data-ttu-id="a92d0-153">Per i componenti che gestiscono gli errori nei metodi del ciclo di vita, aggiungere la logica di gestione degli errori.</span><span class="sxs-lookup"><span data-stu-id="a92d0-153">For components to deal with errors in lifecycle methods, add error handling logic.</span></span>

<span data-ttu-id="a92d0-154">Nell'esempio seguente viene <xref:Microsoft.AspNetCore.Components.ComponentBase.OnParametersSetAsync%2A> chiamato un metodo per ottenere un prodotto:</span><span class="sxs-lookup"><span data-stu-id="a92d0-154">In the following example where <xref:Microsoft.AspNetCore.Components.ComponentBase.OnParametersSetAsync%2A> calls a method to obtain a product:</span></span>

* <span data-ttu-id="a92d0-155">Un'eccezione generata nel `ProductRepository.GetProductByIdAsync` metodo viene gestita da un' [`try-catch`](/dotnet/csharp/language-reference/keywords/try-catch) istruzione.</span><span class="sxs-lookup"><span data-stu-id="a92d0-155">An exception thrown in the `ProductRepository.GetProductByIdAsync` method is handled by a [`try-catch`](/dotnet/csharp/language-reference/keywords/try-catch) statement.</span></span>
* <span data-ttu-id="a92d0-156">Quando il `catch` blocco viene eseguito:</span><span class="sxs-lookup"><span data-stu-id="a92d0-156">When the `catch` block is executed:</span></span>
  * <span data-ttu-id="a92d0-157">`loadFailed` è impostato su `true` , che viene utilizzato per visualizzare un messaggio di errore all'utente.</span><span class="sxs-lookup"><span data-stu-id="a92d0-157">`loadFailed` is set to `true`, which is used to display an error message to the user.</span></span>
  * <span data-ttu-id="a92d0-158">L'errore viene registrato.</span><span class="sxs-lookup"><span data-stu-id="a92d0-158">The error is logged.</span></span>

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/handle-errors/ProductDetails.razor?name=snippet&highlight=11,27-39)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Pages/handle-errors/ProductDetails.razor?name=snippet&highlight=11,27-39)]

::: moniker-end

<h3 id="rendering-logic-webassembly"><span data-ttu-id="a92d0-159">Logica di rendering</span><span class="sxs-lookup"><span data-stu-id="a92d0-159">Rendering logic</span></span></h3>

<span data-ttu-id="a92d0-160">Il markup dichiarativo in un Razor file componente ( `.razor` ) viene compilato in un metodo C# denominato <xref:Microsoft.AspNetCore.Components.ComponentBase.BuildRenderTree%2A> .</span><span class="sxs-lookup"><span data-stu-id="a92d0-160">The declarative markup in a Razor component file (`.razor`) is compiled into a C# method called <xref:Microsoft.AspNetCore.Components.ComponentBase.BuildRenderTree%2A>.</span></span> <span data-ttu-id="a92d0-161">Quando viene eseguito il rendering di un componente, <xref:Microsoft.AspNetCore.Components.ComponentBase.BuildRenderTree%2A> viene eseguita e compilata una struttura di dati che descrive gli elementi, il testo e i componenti figlio del componente di cui è stato eseguito il rendering.</span><span class="sxs-lookup"><span data-stu-id="a92d0-161">When a component renders, <xref:Microsoft.AspNetCore.Components.ComponentBase.BuildRenderTree%2A> executes and builds up a data structure describing the elements, text, and child components of the rendered component.</span></span>

<span data-ttu-id="a92d0-162">La logica di rendering può generare un'eccezione.</span><span class="sxs-lookup"><span data-stu-id="a92d0-162">Rendering logic can throw an exception.</span></span> <span data-ttu-id="a92d0-163">Un esempio di questo scenario si verifica quando `@someObject.PropertyName` viene valutato ma `@someObject` è `null` .</span><span class="sxs-lookup"><span data-stu-id="a92d0-163">An example of this scenario occurs when `@someObject.PropertyName` is evaluated but `@someObject` is `null`.</span></span>

<span data-ttu-id="a92d0-164">Per impedire una <xref:System.NullReferenceException> nella logica di rendering, verificare la presenza di un `null` oggetto prima di accedere ai relativi membri.</span><span class="sxs-lookup"><span data-stu-id="a92d0-164">To prevent a <xref:System.NullReferenceException> in rendering logic, check for a `null` object before accessing its members.</span></span> <span data-ttu-id="a92d0-165">Nell'esempio seguente, `person.Address` non è possibile accedere alle proprietà se `person.Address` è `null` :</span><span class="sxs-lookup"><span data-stu-id="a92d0-165">In the following example, `person.Address` properties aren't accessed if `person.Address` is `null`:</span></span>

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/handle-errors/PersonExample.razor?name=snippet&highlight=1)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Pages/handle-errors/PersonExample.razor?name=snippet&highlight=1)]

::: moniker-end

<span data-ttu-id="a92d0-166">Il codice precedente presuppone che `person` non sia `null` .</span><span class="sxs-lookup"><span data-stu-id="a92d0-166">The preceding code assumes that `person` isn't `null`.</span></span> <span data-ttu-id="a92d0-167">Spesso, la struttura del codice garantisce la presenza di un oggetto nel momento in cui viene eseguito il rendering del componente.</span><span class="sxs-lookup"><span data-stu-id="a92d0-167">Often, the structure of the code guarantees that an object exists at the time the component is rendered.</span></span> <span data-ttu-id="a92d0-168">In questi casi, non è necessario verificare la presenza di `null` nella logica di rendering.</span><span class="sxs-lookup"><span data-stu-id="a92d0-168">In those cases, it isn't necessary to check for `null` in rendering logic.</span></span> <span data-ttu-id="a92d0-169">Nell'esempio precedente, `person` potrebbe essere garantito che esista perché `person` viene creato quando viene creata un'istanza del componente, come illustrato nell'esempio seguente:</span><span class="sxs-lookup"><span data-stu-id="a92d0-169">In the prior example, `person` might be guaranteed to exist because `person` is created when the component is instantiated, as the following example shows:</span></span>

```razor
@code {
    private Person person = new Person();

    ...
}
```

<h3 id="event-handlers-webassembly"><span data-ttu-id="a92d0-170">Gestori eventi</span><span class="sxs-lookup"><span data-stu-id="a92d0-170">Event handlers</span></span></h3>

<span data-ttu-id="a92d0-171">Il codice lato client attiva le chiamate del codice C# quando i gestori eventi vengono creati usando:</span><span class="sxs-lookup"><span data-stu-id="a92d0-171">Client-side code triggers invocations of C# code when event handlers are created using:</span></span>

* `@onclick`
* `@onchange`
* <span data-ttu-id="a92d0-172">Altri `@on...` attributi</span><span class="sxs-lookup"><span data-stu-id="a92d0-172">Other `@on...` attributes</span></span>
* `@bind`

<span data-ttu-id="a92d0-173">Il codice del gestore eventi potrebbe generare un'eccezione non gestita in questi scenari.</span><span class="sxs-lookup"><span data-stu-id="a92d0-173">Event handler code might throw an unhandled exception in these scenarios.</span></span>

<span data-ttu-id="a92d0-174">Se l'app chiama il codice che potrebbe non riuscire per motivi esterni, intercettare le eccezioni usando un' [`try-catch`](/dotnet/csharp/language-reference/keywords/try-catch) istruzione con gestione e registrazione degli errori.</span><span class="sxs-lookup"><span data-stu-id="a92d0-174">If the app calls code that could fail for external reasons, trap exceptions using a [`try-catch`](/dotnet/csharp/language-reference/keywords/try-catch) statement with error handling and logging.</span></span>

<span data-ttu-id="a92d0-175">Se il codice utente non intercetta e gestisce l'eccezione, il Framework registra l'eccezione.</span><span class="sxs-lookup"><span data-stu-id="a92d0-175">If user code doesn't trap and handle the exception, the framework logs the exception.</span></span>

<h3 id="component-disposal-webassembly"><span data-ttu-id="a92d0-176">Eliminazione componenti</span><span class="sxs-lookup"><span data-stu-id="a92d0-176">Component disposal</span></span></h3>

<span data-ttu-id="a92d0-177">Un componente può essere rimosso dall'interfaccia utente, ad esempio perché l'utente ha esplorato un'altra pagina.</span><span class="sxs-lookup"><span data-stu-id="a92d0-177">A component may be removed from the UI, for example, because the user has navigated to another page.</span></span> <span data-ttu-id="a92d0-178">Quando un componente che implementa <xref:System.IDisposable?displayProperty=fullName> viene rimosso dall'interfaccia utente, il Framework chiama il metodo del componente <xref:System.IDisposable.Dispose%2A> .</span><span class="sxs-lookup"><span data-stu-id="a92d0-178">When a component that implements <xref:System.IDisposable?displayProperty=fullName> is removed from the UI, the framework calls the component's <xref:System.IDisposable.Dispose%2A> method.</span></span>

<span data-ttu-id="a92d0-179">Se la logica di eliminazione può generare eccezioni, l'app deve intercettare le eccezioni usando un' [`try-catch`](/dotnet/csharp/language-reference/keywords/try-catch) istruzione con gestione e registrazione degli errori.</span><span class="sxs-lookup"><span data-stu-id="a92d0-179">If disposal logic may throw exceptions, the app should trap the exceptions using a [`try-catch`](/dotnet/csharp/language-reference/keywords/try-catch) statement with error handling and logging.</span></span>

<span data-ttu-id="a92d0-180">Per ulteriori informazioni sull'eliminazione dei componenti, vedere <xref:blazor/components/lifecycle#component-disposal-with-idisposable> .</span><span class="sxs-lookup"><span data-stu-id="a92d0-180">For more information on component disposal, see <xref:blazor/components/lifecycle#component-disposal-with-idisposable>.</span></span>

<h3 id="javascript-interop-webassembly"><span data-ttu-id="a92d0-181">Interoperabilità JavaScript</span><span class="sxs-lookup"><span data-stu-id="a92d0-181">JavaScript interop</span></span></h3>

<span data-ttu-id="a92d0-182"><xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A?displayProperty=nameWithType> consente al codice .NET di effettuare chiamate asincrone al runtime JavaScript nel browser dell'utente.</span><span class="sxs-lookup"><span data-stu-id="a92d0-182"><xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A?displayProperty=nameWithType> allows .NET code to make asynchronous calls to the JavaScript runtime in the user's browser.</span></span>

<span data-ttu-id="a92d0-183">Le condizioni seguenti si applicano alla gestione degli errori con <xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A> :</span><span class="sxs-lookup"><span data-stu-id="a92d0-183">The following conditions apply to error handling with <xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A>:</span></span>

* <span data-ttu-id="a92d0-184">Se una chiamata a ha <xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A> esito negativo in modo sincrono, si verifica un'eccezione .NET.</span><span class="sxs-lookup"><span data-stu-id="a92d0-184">If a call to <xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A> fails synchronously, a .NET exception occurs.</span></span> <span data-ttu-id="a92d0-185">Una chiamata a <xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A> potrebbe non riuscire, ad esempio perché gli argomenti forniti non possono essere serializzati.</span><span class="sxs-lookup"><span data-stu-id="a92d0-185">A call to <xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A> may fail, for example, because the supplied arguments can't be serialized.</span></span> <span data-ttu-id="a92d0-186">Il codice dello sviluppatore deve intercettare l'eccezione.</span><span class="sxs-lookup"><span data-stu-id="a92d0-186">Developer code must catch the exception.</span></span>
* <span data-ttu-id="a92d0-187">Se una chiamata a ha <xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A> esito negativo in modo asincrono, .NET ha <xref:System.Threading.Tasks.Task> esito negativo.</span><span class="sxs-lookup"><span data-stu-id="a92d0-187">If a call to <xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A> fails asynchronously, the .NET <xref:System.Threading.Tasks.Task> fails.</span></span> <span data-ttu-id="a92d0-188">Una chiamata a <xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A> potrebbe non riuscire, ad esempio perché il codice sul lato JavaScript genera un'eccezione o restituisce un oggetto `Promise` completato come `rejected` .</span><span class="sxs-lookup"><span data-stu-id="a92d0-188">A call to <xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A> may fail, for example, because the JavaScript-side code throws an exception or returns a `Promise` that completed as `rejected`.</span></span> <span data-ttu-id="a92d0-189">Il codice dello sviluppatore deve intercettare l'eccezione.</span><span class="sxs-lookup"><span data-stu-id="a92d0-189">Developer code must catch the exception.</span></span> <span data-ttu-id="a92d0-190">Se si utilizza l' [`await`](/dotnet/csharp/language-reference/keywords/await) operatore, è consigliabile eseguire il wrapping della chiamata al metodo in un' [`try-catch`](/dotnet/csharp/language-reference/keywords/try-catch) istruzione con gestione e registrazione degli errori.</span><span class="sxs-lookup"><span data-stu-id="a92d0-190">If using the [`await`](/dotnet/csharp/language-reference/keywords/await) operator, consider wrapping the method call in a [`try-catch`](/dotnet/csharp/language-reference/keywords/try-catch) statement with error handling and logging.</span></span>
* <span data-ttu-id="a92d0-191">Per impostazione predefinita, le chiamate a <xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A> devono essere completate entro un determinato periodo oppure si verifica il timeout della chiamata. Il periodo di timeout predefinito è di un minuto.</span><span class="sxs-lookup"><span data-stu-id="a92d0-191">By default, calls to <xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A> must complete within a certain period or else the call times out. The default timeout period is one minute.</span></span> <span data-ttu-id="a92d0-192">Il timeout protegge il codice da una perdita di connettività di rete o codice JavaScript che non restituisce mai un messaggio di completamento.</span><span class="sxs-lookup"><span data-stu-id="a92d0-192">The timeout protects the code against a loss in network connectivity or JavaScript code that never sends back a completion message.</span></span> <span data-ttu-id="a92d0-193">Se si verifica il timeout della chiamata, l'oggetto risultante ha <xref:System.Threading.Tasks> esito negativo con un oggetto <xref:System.OperationCanceledException> .</span><span class="sxs-lookup"><span data-stu-id="a92d0-193">If the call times out, the resulting <xref:System.Threading.Tasks> fails with an <xref:System.OperationCanceledException>.</span></span> <span data-ttu-id="a92d0-194">Intercettare ed elaborare l'eccezione con la registrazione.</span><span class="sxs-lookup"><span data-stu-id="a92d0-194">Trap and process the exception with logging.</span></span>

<span data-ttu-id="a92d0-195">Analogamente, il codice JavaScript può avviare chiamate a metodi .NET indicati dall' [ `[JSInvokable]` attributo](xref:blazor/call-dotnet-from-javascript).</span><span class="sxs-lookup"><span data-stu-id="a92d0-195">Similarly, JavaScript code may initiate calls to .NET methods indicated by the [`[JSInvokable]` attribute](xref:blazor/call-dotnet-from-javascript).</span></span> <span data-ttu-id="a92d0-196">Se questi metodi .NET generano un'eccezione non gestita, il lato JavaScript `Promise` viene rifiutato.</span><span class="sxs-lookup"><span data-stu-id="a92d0-196">If these .NET methods throw an unhandled exception, the JavaScript-side `Promise` is rejected.</span></span>

<span data-ttu-id="a92d0-197">È possibile scegliere di usare il codice di gestione degli errori sul lato .NET o sul lato JavaScript della chiamata al metodo.</span><span class="sxs-lookup"><span data-stu-id="a92d0-197">You have the option of using error handling code on either the .NET side or the JavaScript side of the method call.</span></span>

<span data-ttu-id="a92d0-198">Per altre informazioni, vedere gli articoli seguenti:</span><span class="sxs-lookup"><span data-stu-id="a92d0-198">For more information, see the following articles:</span></span>

* <xref:blazor/call-javascript-from-dotnet>
* <xref:blazor/call-dotnet-from-javascript>

## <a name="advanced-scenarios"></a><span data-ttu-id="a92d0-199">Scenari avanzati</span><span class="sxs-lookup"><span data-stu-id="a92d0-199">Advanced scenarios</span></span>

### <a name="recursive-rendering"></a><span data-ttu-id="a92d0-200">Rendering ricorsivo</span><span class="sxs-lookup"><span data-stu-id="a92d0-200">Recursive rendering</span></span>

<span data-ttu-id="a92d0-201">I componenti possono essere annidati in modo ricorsivo.</span><span class="sxs-lookup"><span data-stu-id="a92d0-201">Components can be nested recursively.</span></span> <span data-ttu-id="a92d0-202">Questa operazione è utile per la rappresentazione di strutture di dati ricorsive.</span><span class="sxs-lookup"><span data-stu-id="a92d0-202">This is useful for representing recursive data structures.</span></span> <span data-ttu-id="a92d0-203">Un componente, ad esempio, `TreeNode` può eseguire il rendering di più `TreeNode` componenti per ognuno dei figli del nodo.</span><span class="sxs-lookup"><span data-stu-id="a92d0-203">For example, a `TreeNode` component can render more `TreeNode` components for each of the node's children.</span></span>

<span data-ttu-id="a92d0-204">Quando si esegue il rendering in modo ricorsivo, evitare i modelli di codifica che generano una ricorsione</span><span class="sxs-lookup"><span data-stu-id="a92d0-204">When rendering recursively, avoid coding patterns that result in infinite recursion:</span></span>

* <span data-ttu-id="a92d0-205">Non esegue il rendering in modo ricorsivo di una struttura di dati che contiene un ciclo.</span><span class="sxs-lookup"><span data-stu-id="a92d0-205">Don't recursively render a data structure that contains a cycle.</span></span> <span data-ttu-id="a92d0-206">Ad esempio, non eseguire il rendering di un nodo dell'albero i cui elementi figlio includono se stesso.</span><span class="sxs-lookup"><span data-stu-id="a92d0-206">For example, don't render a tree node whose children includes itself.</span></span>
* <span data-ttu-id="a92d0-207">Non creare una catena di layout che contiene un ciclo.</span><span class="sxs-lookup"><span data-stu-id="a92d0-207">Don't create a chain of layouts that contain a cycle.</span></span> <span data-ttu-id="a92d0-208">Non creare, ad esempio, un layout il cui layout è a sua volta.</span><span class="sxs-lookup"><span data-stu-id="a92d0-208">For example, don't create a layout whose layout is itself.</span></span>
* <span data-ttu-id="a92d0-209">Non consentire a un utente finale di violare le invarianti di ricorsione (regole) tramite immissione di dati dannosi o chiamate di interoperabilità JavaScript.</span><span class="sxs-lookup"><span data-stu-id="a92d0-209">Don't allow an end user to violate recursion invariants (rules) through malicious data entry or JavaScript interop calls.</span></span>

<span data-ttu-id="a92d0-210">Cicli infiniti durante il rendering:</span><span class="sxs-lookup"><span data-stu-id="a92d0-210">Infinite loops during rendering:</span></span>

* <span data-ttu-id="a92d0-211">Causa la continuazione del processo di rendering.</span><span class="sxs-lookup"><span data-stu-id="a92d0-211">Causes the rendering process to continue forever.</span></span>
* <span data-ttu-id="a92d0-212">Equivale alla creazione di un ciclo senza terminazione.</span><span class="sxs-lookup"><span data-stu-id="a92d0-212">Is equivalent to creating an unterminated loop.</span></span>

<span data-ttu-id="a92d0-213">In questi scenari, il thread tenta in genere di eseguire le operazioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="a92d0-213">In these scenarios, the thread usually attempts to:</span></span>

* <span data-ttu-id="a92d0-214">Utilizzare la quantità di tempo della CPU consentita dal sistema operativo, per un periodo illimitato.</span><span class="sxs-lookup"><span data-stu-id="a92d0-214">Consume as much CPU time as permitted by the operating system, indefinitely.</span></span>
* <span data-ttu-id="a92d0-215">Utilizzare una quantità illimitata di memoria del client.</span><span class="sxs-lookup"><span data-stu-id="a92d0-215">Consume an unlimited amount of client memory.</span></span> <span data-ttu-id="a92d0-216">L'utilizzo di memoria illimitata equivale allo scenario in cui un ciclo senza terminazione aggiunge voci a una raccolta in ogni iterazione.</span><span class="sxs-lookup"><span data-stu-id="a92d0-216">Consuming unlimited memory is equivalent to the scenario where an unterminated loop adds entries to a collection on every iteration.</span></span>

<span data-ttu-id="a92d0-217">Per evitare modelli di ricorsione infinita, verificare che il codice di rendering ricorsivo contenga condizioni di arresto appropriate.</span><span class="sxs-lookup"><span data-stu-id="a92d0-217">To avoid infinite recursion patterns, ensure that recursive rendering code contains suitable stopping conditions.</span></span>

### <a name="custom-render-tree-logic"></a><span data-ttu-id="a92d0-218">Logica dell'albero di rendering personalizzata</span><span class="sxs-lookup"><span data-stu-id="a92d0-218">Custom render tree logic</span></span>

<span data-ttu-id="a92d0-219">La maggior parte dei Blazor componenti viene implementata come Razor file componente ( `.razor` ) e viene compilata dal Framework per produrre la logica che opera su un <xref:Microsoft.AspNetCore.Components.Rendering.RenderTreeBuilder> per eseguire il rendering dell'output.</span><span class="sxs-lookup"><span data-stu-id="a92d0-219">Most Blazor components are implemented as Razor component files (`.razor`) and are compiled by the framework to produce logic that operates on a <xref:Microsoft.AspNetCore.Components.Rendering.RenderTreeBuilder> to render their output.</span></span> <span data-ttu-id="a92d0-220">Tuttavia, uno sviluppatore può implementare manualmente la <xref:Microsoft.AspNetCore.Components.Rendering.RenderTreeBuilder> logica usando il codice C# procedurale.</span><span class="sxs-lookup"><span data-stu-id="a92d0-220">However, a developer may manually implement <xref:Microsoft.AspNetCore.Components.Rendering.RenderTreeBuilder> logic using procedural C# code.</span></span> <span data-ttu-id="a92d0-221">Per altre informazioni, vedere <xref:blazor/advanced-scenarios#manual-rendertreebuilder-logic>.</span><span class="sxs-lookup"><span data-stu-id="a92d0-221">For more information, see <xref:blazor/advanced-scenarios#manual-rendertreebuilder-logic>.</span></span>

> [!WARNING]
> <span data-ttu-id="a92d0-222">L'uso della logica del generatore di albero di rendering manuale è considerato uno scenario avanzato e non sicuro, non consigliato per lo sviluppo di componenti generali.</span><span class="sxs-lookup"><span data-stu-id="a92d0-222">Use of manual render tree builder logic is considered an advanced and unsafe scenario, not recommended for general component development.</span></span>

<span data-ttu-id="a92d0-223">Se il <xref:Microsoft.AspNetCore.Components.Rendering.RenderTreeBuilder> codice viene scritto, lo sviluppatore deve garantire la correttezza del codice.</span><span class="sxs-lookup"><span data-stu-id="a92d0-223">If <xref:Microsoft.AspNetCore.Components.Rendering.RenderTreeBuilder> code is written, the developer must guarantee the correctness of the code.</span></span> <span data-ttu-id="a92d0-224">Ad esempio, lo sviluppatore deve garantire quanto segue:</span><span class="sxs-lookup"><span data-stu-id="a92d0-224">For example, the developer must ensure that:</span></span>

* <span data-ttu-id="a92d0-225">Le chiamate a <xref:Microsoft.AspNetCore.Components.Rendering.RenderTreeBuilder.OpenElement%2A> e <xref:Microsoft.AspNetCore.Components.Rendering.RenderTreeBuilder.CloseElement%2A> sono bilanciate correttamente.</span><span class="sxs-lookup"><span data-stu-id="a92d0-225">Calls to <xref:Microsoft.AspNetCore.Components.Rendering.RenderTreeBuilder.OpenElement%2A> and <xref:Microsoft.AspNetCore.Components.Rendering.RenderTreeBuilder.CloseElement%2A> are correctly balanced.</span></span>
* <span data-ttu-id="a92d0-226">Gli attributi vengono aggiunti solo nei punti corretti.</span><span class="sxs-lookup"><span data-stu-id="a92d0-226">Attributes are only added in the correct places.</span></span>

<span data-ttu-id="a92d0-227">La logica del generatore di albero di rendering manuale non corretta può causare un comportamento arbitrario non definito, tra cui arresti anomali, blocchi di app e vulnerabilità della sicurezza.</span><span class="sxs-lookup"><span data-stu-id="a92d0-227">Incorrect manual render tree builder logic can cause arbitrary undefined behavior, including crashes, app hangs, and security vulnerabilities.</span></span>

<span data-ttu-id="a92d0-228">Prendere in considerazione la logica del generatore di albero di rendering manuale sullo stesso livello di complessità e con lo stesso livello di *rischio* di scrivere manualmente codice assembly o istruzioni [MSIL (Microsoft Intermediate Language)](/dotnet/standard/managed-code) .</span><span class="sxs-lookup"><span data-stu-id="a92d0-228">Consider manual render tree builder logic on the same level of complexity and with the same level of *danger* as writing assembly code or [Microsoft Intermediate Language (MSIL)](/dotnet/standard/managed-code) instructions by hand.</span></span>

## <a name="additional-resources"></a><span data-ttu-id="a92d0-229">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="a92d0-229">Additional resources</span></span>

* <xref:blazor/fundamentals/logging>
* <xref:fundamentals/error-handling>&dagger;
* <xref:web-api/index>

<span data-ttu-id="a92d0-230">&dagger;Si applica a back-end ASP.NET Core app per le API Web utilizzate dalle app sul lato client Blazor WebAssembly per la registrazione.</span><span class="sxs-lookup"><span data-stu-id="a92d0-230">&dagger;Applies to backend ASP.NET Core web API apps that client-side Blazor WebAssembly apps use for logging.</span></span>

::: zone-end

::: zone pivot="server"

## <a name="detailed-errors-during-development"></a><span data-ttu-id="a92d0-231">Errori dettagliati durante lo sviluppo</span><span class="sxs-lookup"><span data-stu-id="a92d0-231">Detailed errors during development</span></span>

<span data-ttu-id="a92d0-232">Quando un' Blazor app non funziona correttamente durante lo sviluppo, la ricezione di informazioni dettagliate sugli errori dall'app è utile per la risoluzione dei problemi e per risolvere il problema.</span><span class="sxs-lookup"><span data-stu-id="a92d0-232">When a Blazor app isn't functioning properly during development, receiving detailed error information from the app assists in troubleshooting and fixing the issue.</span></span> <span data-ttu-id="a92d0-233">Quando si verifica un errore, le Blazor app visualizzano una barra gialla chiara nella parte inferiore della schermata:</span><span class="sxs-lookup"><span data-stu-id="a92d0-233">When an error occurs, Blazor apps display a light yellow bar at the bottom of the screen:</span></span>

* <span data-ttu-id="a92d0-234">Durante lo sviluppo, la barra indirizza l'utente alla console del browser, in cui è possibile visualizzare l'eccezione.</span><span class="sxs-lookup"><span data-stu-id="a92d0-234">During development, the bar directs you to the browser console, where you can see the exception.</span></span>
* <span data-ttu-id="a92d0-235">In produzione, la barra informa l'utente che si è verificato un errore e consiglia di aggiornare il browser.</span><span class="sxs-lookup"><span data-stu-id="a92d0-235">In production, the bar notifies the user that an error has occurred and recommends refreshing the browser.</span></span>

<span data-ttu-id="a92d0-236">L'interfaccia utente per questa esperienza di gestione degli errori fa parte dei [ Blazor modelli di progetto](xref:blazor/project-structure).</span><span class="sxs-lookup"><span data-stu-id="a92d0-236">The UI for this error handling experience is part of the [Blazor project templates](xref:blazor/project-structure).</span></span>

<span data-ttu-id="a92d0-237">In un' Blazor Server app, personalizzare l'esperienza nel `Pages/_Host.cshtml` file:</span><span class="sxs-lookup"><span data-stu-id="a92d0-237">In a Blazor Server app, customize the experience in the `Pages/_Host.cshtml` file:</span></span>

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

<span data-ttu-id="a92d0-238">L' `blazor-error-ui` elemento è generalmente nascosto a causa della presenza dello `display: none` stile della `blazor-error-ui` classe CSS nel foglio di stile del sito ( `wwwroot/css/site.css` ).</span><span class="sxs-lookup"><span data-stu-id="a92d0-238">The `blazor-error-ui` element is normally hidden due the presence of the `display: none` style of the `blazor-error-ui` CSS class in the site's stylesheet (`wwwroot/css/site.css`).</span></span> <span data-ttu-id="a92d0-239">Quando si verifica un errore, il Framework viene applicato `display: block` all'elemento.</span><span class="sxs-lookup"><span data-stu-id="a92d0-239">When an error occurs, the framework applies `display: block` to the element.</span></span>

## <a name="blazor-server-detailed-circuit-errors"></a><span data-ttu-id="a92d0-240">Blazor Server errori dettagliati del circuito</span><span class="sxs-lookup"><span data-stu-id="a92d0-240">Blazor Server detailed circuit errors</span></span>

<span data-ttu-id="a92d0-241">Gli errori sul lato client non includono lo stack di chiamate e non forniscono dettagli sulla ragione dell'errore, ma i log del server contengono tali informazioni.</span><span class="sxs-lookup"><span data-stu-id="a92d0-241">Client-side errors don't include the call stack and don't provide detail on the cause of the error, but server logs do contain such information.</span></span> <span data-ttu-id="a92d0-242">Ai fini dello sviluppo, le informazioni sugli errori dei circuiti sensibili possono essere rese disponibili per il client abilitando errori dettagliati.</span><span class="sxs-lookup"><span data-stu-id="a92d0-242">For development purposes, sensitive circuit error information can be made available to the client by enabling detailed errors.</span></span>

<span data-ttu-id="a92d0-243">Impostare <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions.DetailedErrors?displayProperty=nameWithType> su `true`.</span><span class="sxs-lookup"><span data-stu-id="a92d0-243">Set <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions.DetailedErrors?displayProperty=nameWithType> to `true`.</span></span> <span data-ttu-id="a92d0-244">Per altre informazioni e un esempio, vedere <xref:blazor/fundamentals/signalr#circuit-handler-options>.</span><span class="sxs-lookup"><span data-stu-id="a92d0-244">For more information and an example, see <xref:blazor/fundamentals/signalr#circuit-handler-options>.</span></span>

<span data-ttu-id="a92d0-245">Un'alternativa all'impostazione <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions.DetailedErrors?displayProperty=nameWithType> consiste nell'impostare la `DetailedErrors` chiave di configurazione su `true` nel file di impostazioni dell'ambiente di sviluppo dell'app ( `appsettings.Development.json` ).</span><span class="sxs-lookup"><span data-stu-id="a92d0-245">An alternative to setting <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions.DetailedErrors?displayProperty=nameWithType> is to set the `DetailedErrors` configuration key to `true` in the app's Development environment settings file (`appsettings.Development.json`).</span></span>  <span data-ttu-id="a92d0-246">Inoltre, impostare la [ SignalR registrazione lato server](xref:signalr/diagnostics#server-side-logging) ( `Microsoft.AspNetCore.SignalR` ) per [eseguire il debug](xref:Microsoft.Extensions.Logging.LogLevel) o la [traccia](xref:Microsoft.Extensions.Logging.LogLevel) per la SignalR registrazione dettagliata.</span><span class="sxs-lookup"><span data-stu-id="a92d0-246">Additionally, set [SignalR server-side logging](xref:signalr/diagnostics#server-side-logging) (`Microsoft.AspNetCore.SignalR`) to [Debug](xref:Microsoft.Extensions.Logging.LogLevel) or [Trace](xref:Microsoft.Extensions.Logging.LogLevel) for detailed SignalR logging.</span></span>

<span data-ttu-id="a92d0-247">`appsettings.Development.json`:</span><span class="sxs-lookup"><span data-stu-id="a92d0-247">`appsettings.Development.json`:</span></span>

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

<span data-ttu-id="a92d0-248">La <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions.DetailedErrors> chiave di configurazione può essere impostata anche per `true` usare la `ASPNETCORE_DETAILEDERRORS` variabile di ambiente con un valore `true` in server dell'ambiente di sviluppo/gestione temporanea o nel sistema locale.</span><span class="sxs-lookup"><span data-stu-id="a92d0-248">The <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions.DetailedErrors> configuration key can also be set to `true` using the `ASPNETCORE_DETAILEDERRORS` environment variable with a value of `true` on Development/Staging environment servers or on your local system.</span></span>

> [!WARNING]
> <span data-ttu-id="a92d0-249">Evitare sempre di esporre le informazioni sugli errori ai client su Internet, il che rappresenta un rischio per la sicurezza.</span><span class="sxs-lookup"><span data-stu-id="a92d0-249">Always avoid exposing error information to clients on the Internet, which is a security risk.</span></span>

## <a name="how-a-blazor-server-app-reacts-to-unhandled-exceptions"></a><span data-ttu-id="a92d0-250">Blazor ServerReazione di un'app alle eccezioni non gestite</span><span class="sxs-lookup"><span data-stu-id="a92d0-250">How a Blazor Server app reacts to unhandled exceptions</span></span>

<span data-ttu-id="a92d0-251">Blazor Server è un Framework con stato.</span><span class="sxs-lookup"><span data-stu-id="a92d0-251">Blazor Server is a stateful framework.</span></span> <span data-ttu-id="a92d0-252">Mentre gli utenti interagiscono con un'app, mantengono una connessione al server noto come *circuito*.</span><span class="sxs-lookup"><span data-stu-id="a92d0-252">While users interact with an app, they maintain a connection to the server known as a *circuit*.</span></span> <span data-ttu-id="a92d0-253">Il circuito include istanze di componenti attive, oltre a molti altri aspetti dello stato, ad esempio:</span><span class="sxs-lookup"><span data-stu-id="a92d0-253">The circuit holds active component instances, plus many other aspects of state, such as:</span></span>

* <span data-ttu-id="a92d0-254">Output più recente sottoposto a rendering dei componenti.</span><span class="sxs-lookup"><span data-stu-id="a92d0-254">The most recent rendered output of components.</span></span>
* <span data-ttu-id="a92d0-255">Set corrente di delegati di gestione degli eventi che possono essere attivati da eventi lato client.</span><span class="sxs-lookup"><span data-stu-id="a92d0-255">The current set of event-handling delegates that could be triggered by client-side events.</span></span>

<span data-ttu-id="a92d0-256">Se un utente apre l'app in più schede del browser, l'utente crea più circuiti indipendenti.</span><span class="sxs-lookup"><span data-stu-id="a92d0-256">If a user opens the app in multiple browser tabs, the user creates multiple independent circuits.</span></span>

<span data-ttu-id="a92d0-257">Blazor Considera le eccezioni non gestite come irreversibili per il circuito in cui si verificano.</span><span class="sxs-lookup"><span data-stu-id="a92d0-257">Blazor treats most unhandled exceptions as fatal to the circuit where they occur.</span></span> <span data-ttu-id="a92d0-258">Se un circuito viene terminato a causa di un'eccezione non gestita, l'utente può continuare a interagire con l'app ricaricando la pagina per creare un nuovo circuito.</span><span class="sxs-lookup"><span data-stu-id="a92d0-258">If a circuit is terminated due to an unhandled exception, the user can only continue to interact with the app by reloading the page to create a new circuit.</span></span> <span data-ttu-id="a92d0-259">I circuiti al di fuori di quello terminato, ovvero circuiti per altri utenti o altre schede del browser, non sono interessati.</span><span class="sxs-lookup"><span data-stu-id="a92d0-259">Circuits outside of the one that's terminated, which are circuits for other users or other browser tabs, aren't affected.</span></span> <span data-ttu-id="a92d0-260">Questo scenario è simile a un'applicazione desktop che si arresta in modo anomalo.</span><span class="sxs-lookup"><span data-stu-id="a92d0-260">This scenario is similar to a desktop app that crashes.</span></span> <span data-ttu-id="a92d0-261">L'app arrestata in modo anomalo deve essere riavviata, ma non sono interessate altre app.</span><span class="sxs-lookup"><span data-stu-id="a92d0-261">The crashed app must be restarted, but other apps aren't affected.</span></span>

<span data-ttu-id="a92d0-262">Il Framework termina un circuito quando si verifica un'eccezione non gestita per i motivi seguenti:</span><span class="sxs-lookup"><span data-stu-id="a92d0-262">The framework terminates a circuit when an unhandled exception occurs for the following reasons:</span></span>

* <span data-ttu-id="a92d0-263">Un'eccezione non gestita spesso lascia il circuito in uno stato non definito.</span><span class="sxs-lookup"><span data-stu-id="a92d0-263">An unhandled exception often leaves the circuit in an undefined state.</span></span>
* <span data-ttu-id="a92d0-264">L'operazione normale dell'app non può essere garantita dopo un'eccezione non gestita.</span><span class="sxs-lookup"><span data-stu-id="a92d0-264">The app's normal operation can't be guaranteed after an unhandled exception.</span></span>
* <span data-ttu-id="a92d0-265">Le vulnerabilità di sicurezza possono essere visualizzate nell'app se il circuito continua in uno stato non definito.</span><span class="sxs-lookup"><span data-stu-id="a92d0-265">Security vulnerabilities may appear in the app if the circuit continues in an undefined state.</span></span>

## <a name="manage-unhandled-exceptions-in-developer-code"></a><span data-ttu-id="a92d0-266">Gestire le eccezioni non gestite nel codice dello sviluppatore</span><span class="sxs-lookup"><span data-stu-id="a92d0-266">Manage unhandled exceptions in developer code</span></span>

<span data-ttu-id="a92d0-267">Per continuare l'applicazione dopo un errore, l'app deve avere la logica di gestione degli errori.</span><span class="sxs-lookup"><span data-stu-id="a92d0-267">For an app to continue after an error, the app must have error handling logic.</span></span> <span data-ttu-id="a92d0-268">Le sezioni successive di questo articolo descrivono le potenziali fonti di eccezioni non gestite.</span><span class="sxs-lookup"><span data-stu-id="a92d0-268">Later sections of this article describe potential sources of unhandled exceptions.</span></span>

<span data-ttu-id="a92d0-269">In produzione, non eseguire il rendering dei messaggi di eccezione del Framework o delle analisi dello stack nell'interfaccia utente.</span><span class="sxs-lookup"><span data-stu-id="a92d0-269">In production, don't render framework exception messages or stack traces in the UI.</span></span> <span data-ttu-id="a92d0-270">Il rendering dei messaggi di eccezione o delle analisi dello stack potrebbe:</span><span class="sxs-lookup"><span data-stu-id="a92d0-270">Rendering exception messages or stack traces could:</span></span>

* <span data-ttu-id="a92d0-271">Divulgare informazioni riservate agli utenti finali.</span><span class="sxs-lookup"><span data-stu-id="a92d0-271">Disclose sensitive information to end users.</span></span>
* <span data-ttu-id="a92d0-272">Aiutare un utente malintenzionato a individuare i punti deboli in un'app che può compromettere la sicurezza dell'app, del server o della rete.</span><span class="sxs-lookup"><span data-stu-id="a92d0-272">Help a malicious user discover weaknesses in an app that can compromise the security of the app, server, or network.</span></span>

## <a name="global-exception-handling"></a><span data-ttu-id="a92d0-273">Gestione delle eccezioni globali</span><span class="sxs-lookup"><span data-stu-id="a92d0-273">Global exception handling</span></span>

[!INCLUDE[](~/blazor/includes/handle-errors/global-exception-handling.md)]

<span data-ttu-id="a92d0-274">Poiché gli approcci in questa sezione gestiscono gli errori con un' [`try-catch`](/dotnet/csharp/language-reference/keywords/try-catch) istruzione, la SignalR connessione tra il client e il server non viene interrotta quando si verifica un errore e il circuito rimane attivo.</span><span class="sxs-lookup"><span data-stu-id="a92d0-274">Because the approaches in this section handle errors with a [`try-catch`](/dotnet/csharp/language-reference/keywords/try-catch) statement, the SignalR connection between the client and server isn't broken when an error occurs and the circuit remains alive.</span></span> <span data-ttu-id="a92d0-275">Eventuali eccezioni non gestite sono fatali per un circuito.</span><span class="sxs-lookup"><span data-stu-id="a92d0-275">Any unhandled exception is fatal to a circuit.</span></span> <span data-ttu-id="a92d0-276">Per ulteriori informazioni, vedere la sezione precedente sul [modo in Blazor Server cui un'applicazione reagisce alle eccezioni non gestite](#how-a-blazor-server-app-reacts-to-unhandled-exceptions).</span><span class="sxs-lookup"><span data-stu-id="a92d0-276">For more information, see the preceding section on [how a Blazor Server app reacts to unhandled exceptions](#how-a-blazor-server-app-reacts-to-unhandled-exceptions).</span></span>

## <a name="log-errors-with-a-persistent-provider"></a><span data-ttu-id="a92d0-277">Registrare gli errori con un provider persistente</span><span class="sxs-lookup"><span data-stu-id="a92d0-277">Log errors with a persistent provider</span></span>

<span data-ttu-id="a92d0-278">Se si verifica un'eccezione non gestita, l'eccezione viene registrata <xref:Microsoft.Extensions.Logging.ILogger> nelle istanze configurate nel contenitore dei servizi.</span><span class="sxs-lookup"><span data-stu-id="a92d0-278">If an unhandled exception occurs, the exception is logged to <xref:Microsoft.Extensions.Logging.ILogger> instances configured in the service container.</span></span> <span data-ttu-id="a92d0-279">Per impostazione predefinita, Blazor le app registrano nell'output della console con il provider di registrazione della console.</span><span class="sxs-lookup"><span data-stu-id="a92d0-279">By default, Blazor apps log to console output with the Console Logging Provider.</span></span> <span data-ttu-id="a92d0-280">Prendere in considerazione la registrazione a una posizione più permanente sul server con un provider che gestisce le dimensioni del log e la rotazione del log.</span><span class="sxs-lookup"><span data-stu-id="a92d0-280">Consider logging to a more permanent location on the server with a provider that manages log size and log rotation.</span></span> <span data-ttu-id="a92d0-281">In alternativa, l'app può usare un servizio di gestione delle prestazioni delle applicazioni (APM), ad esempio [applicazione Azure Insights (monitoraggio di Azure)](/azure/azure-monitor/app/app-insights-overview).</span><span class="sxs-lookup"><span data-stu-id="a92d0-281">Alternatively, the app can use an Application Performance Management (APM) service, such as [Azure Application Insights (Azure Monitor)](/azure/azure-monitor/app/app-insights-overview).</span></span>

<span data-ttu-id="a92d0-282">Durante lo sviluppo, un' Blazor Server applicazione in genere invia i dettagli completi delle eccezioni alla console del browser per facilitare il debug.</span><span class="sxs-lookup"><span data-stu-id="a92d0-282">During development, a Blazor Server app usually sends the full details of exceptions to the browser's console to aid in debugging.</span></span> <span data-ttu-id="a92d0-283">In produzione, gli errori dettagliati non vengono inviati ai client, ma i dettagli completi di un'eccezione vengono registrati sul server.</span><span class="sxs-lookup"><span data-stu-id="a92d0-283">In production, detailed errors aren't sent to clients, but an exception's full details are logged on the server.</span></span>

<span data-ttu-id="a92d0-284">È necessario decidere quali eventi imprevisti registrare e il livello di gravità degli eventi imprevisti registrati.</span><span class="sxs-lookup"><span data-stu-id="a92d0-284">You must decide which incidents to log and the level of severity of logged incidents.</span></span> <span data-ttu-id="a92d0-285">Gli utenti ostili potrebbero essere in grado di attivare intenzionalmente gli errori.</span><span class="sxs-lookup"><span data-stu-id="a92d0-285">Hostile users might be able to trigger errors deliberately.</span></span> <span data-ttu-id="a92d0-286">Ad esempio, non registrare un evento imprevisto da un errore in cui viene fornito un oggetto sconosciuto `ProductId` nell'URL di un componente che Visualizza i dettagli del prodotto.</span><span class="sxs-lookup"><span data-stu-id="a92d0-286">For example, don't log an incident from an error where an unknown `ProductId` is supplied in the URL of a component that displays product details.</span></span> <span data-ttu-id="a92d0-287">Non tutti gli errori devono essere considerati come eventi imprevisti per la registrazione.</span><span class="sxs-lookup"><span data-stu-id="a92d0-287">Not all errors should be treated as incidents for logging.</span></span>

<span data-ttu-id="a92d0-288">Per altre informazioni, vedere gli articoli seguenti:</span><span class="sxs-lookup"><span data-stu-id="a92d0-288">For more information, see the following articles:</span></span>

* <xref:blazor/fundamentals/logging>
* <xref:fundamentals/error-handling>&dagger;

<span data-ttu-id="a92d0-289">&dagger;Si applica alle app ASP.NET Core lato server che sono app back-end dell'API Web per le Blazor app.</span><span class="sxs-lookup"><span data-stu-id="a92d0-289">&dagger;Applies to server-side ASP.NET Core apps that are web API backend apps for Blazor apps.</span></span>

## <a name="places-where-errors-may-occur"></a><span data-ttu-id="a92d0-290">Posizioni in cui possono verificarsi errori</span><span class="sxs-lookup"><span data-stu-id="a92d0-290">Places where errors may occur</span></span>

<span data-ttu-id="a92d0-291">Il Framework e il codice dell'app possono attivare eccezioni non gestite in uno dei percorsi seguenti, descritti più avanti nelle sezioni seguenti di questo articolo:</span><span class="sxs-lookup"><span data-stu-id="a92d0-291">Framework and app code may trigger unhandled exceptions in any of the following locations, which are described further in the following sections of this article:</span></span>

* [<span data-ttu-id="a92d0-292">Creazione di istanze di componenti</span><span class="sxs-lookup"><span data-stu-id="a92d0-292">Component instantiation</span></span>](#component-instantiation-server)
* [<span data-ttu-id="a92d0-293">Metodi del ciclo di vita</span><span class="sxs-lookup"><span data-stu-id="a92d0-293">Lifecycle methods</span></span>](#lifecycle-methods-server)
* [<span data-ttu-id="a92d0-294">Logica di rendering</span><span class="sxs-lookup"><span data-stu-id="a92d0-294">Rendering logic</span></span>](#rendering-logic-server)
* [<span data-ttu-id="a92d0-295">Gestori eventi</span><span class="sxs-lookup"><span data-stu-id="a92d0-295">Event handlers</span></span>](#event-handlers-server)
* [<span data-ttu-id="a92d0-296">Eliminazione componenti</span><span class="sxs-lookup"><span data-stu-id="a92d0-296">Component disposal</span></span>](#component-disposal-server)
* [<span data-ttu-id="a92d0-297">Interoperabilità JavaScript</span><span class="sxs-lookup"><span data-stu-id="a92d0-297">JavaScript interop</span></span>](#javascript-interop-server)
* [<span data-ttu-id="a92d0-298">Blazor Server esecuzione del rendering</span><span class="sxs-lookup"><span data-stu-id="a92d0-298">Blazor Server rerendering</span></span>](#blazor-server-prerendering-server)

<h3 id="component-instantiation-server"><span data-ttu-id="a92d0-299">Creazione di istanze di componenti</span><span class="sxs-lookup"><span data-stu-id="a92d0-299">Component instantiation</span></span></h3>

<span data-ttu-id="a92d0-300">Quando Blazor Crea un'istanza di un componente:</span><span class="sxs-lookup"><span data-stu-id="a92d0-300">When Blazor creates an instance of a component:</span></span>

* <span data-ttu-id="a92d0-301">Il costruttore del componente viene richiamato.</span><span class="sxs-lookup"><span data-stu-id="a92d0-301">The component's constructor is invoked.</span></span>
* <span data-ttu-id="a92d0-302">Vengono richiamati i costruttori di tutti i servizi non singleton forniti al costruttore del componente tramite la [`@inject`](xref:mvc/views/razor#inject) direttiva o l' [ `[Inject]` attributo](xref:blazor/fundamentals/dependency-injection#request-a-service-in-a-component) .</span><span class="sxs-lookup"><span data-stu-id="a92d0-302">The constructors of any non-singleton DI services supplied to the component's constructor via the [`@inject`](xref:mvc/views/razor#inject) directive or the [`[Inject]` attribute](xref:blazor/fundamentals/dependency-injection#request-a-service-in-a-component) are invoked.</span></span>

<span data-ttu-id="a92d0-303">Un Blazor Server circuito ha esito negativo quando un costruttore eseguito o un setter per qualsiasi `[Inject]` proprietà genera un'eccezione non gestita.</span><span class="sxs-lookup"><span data-stu-id="a92d0-303">A Blazor Server circuit fails when any executed constructor or a setter for any `[Inject]` property throws an unhandled exception.</span></span> <span data-ttu-id="a92d0-304">L'eccezione è irreversibile perché il Framework non è in grado di creare un'istanza del componente.</span><span class="sxs-lookup"><span data-stu-id="a92d0-304">The exception is fatal because the framework can't instantiate the component.</span></span> <span data-ttu-id="a92d0-305">Se la logica del costruttore può generare eccezioni, l'app deve intercettare le eccezioni usando un' [`try-catch`](/dotnet/csharp/language-reference/keywords/try-catch) istruzione con gestione e registrazione degli errori.</span><span class="sxs-lookup"><span data-stu-id="a92d0-305">If constructor logic may throw exceptions, the app should trap the exceptions using a [`try-catch`](/dotnet/csharp/language-reference/keywords/try-catch) statement with error handling and logging.</span></span>

<h3 id="lifecycle-methods-server"><span data-ttu-id="a92d0-306">Metodi del ciclo di vita</span><span class="sxs-lookup"><span data-stu-id="a92d0-306">Lifecycle methods</span></span></h3>

<span data-ttu-id="a92d0-307">Durante la durata di un componente, Blazor richiama i [metodi del ciclo](xref:blazor/components/lifecycle#lifecycle-methods)di vita.</span><span class="sxs-lookup"><span data-stu-id="a92d0-307">During the lifetime of a component, Blazor invokes [lifecycle methods](xref:blazor/components/lifecycle#lifecycle-methods).</span></span> <span data-ttu-id="a92d0-308">Se un metodo del ciclo di vita genera un'eccezione, in modo sincrono o asincrono, l'eccezione è fatale per un Blazor Server circuito.</span><span class="sxs-lookup"><span data-stu-id="a92d0-308">If any lifecycle method throws an exception, synchronously or asynchronously, the exception is fatal to a Blazor Server circuit.</span></span> <span data-ttu-id="a92d0-309">Per i componenti che gestiscono gli errori nei metodi del ciclo di vita, aggiungere la logica di gestione degli errori.</span><span class="sxs-lookup"><span data-stu-id="a92d0-309">For components to deal with errors in lifecycle methods, add error handling logic.</span></span>

<span data-ttu-id="a92d0-310">Nell'esempio seguente viene <xref:Microsoft.AspNetCore.Components.ComponentBase.OnParametersSetAsync%2A> chiamato un metodo per ottenere un prodotto:</span><span class="sxs-lookup"><span data-stu-id="a92d0-310">In the following example where <xref:Microsoft.AspNetCore.Components.ComponentBase.OnParametersSetAsync%2A> calls a method to obtain a product:</span></span>

* <span data-ttu-id="a92d0-311">Un'eccezione generata nel `ProductRepository.GetProductByIdAsync` metodo viene gestita da un' [`try-catch`](/dotnet/csharp/language-reference/keywords/try-catch) istruzione.</span><span class="sxs-lookup"><span data-stu-id="a92d0-311">An exception thrown in the `ProductRepository.GetProductByIdAsync` method is handled by a [`try-catch`](/dotnet/csharp/language-reference/keywords/try-catch) statement.</span></span>
* <span data-ttu-id="a92d0-312">Quando il `catch` blocco viene eseguito:</span><span class="sxs-lookup"><span data-stu-id="a92d0-312">When the `catch` block is executed:</span></span>
  * <span data-ttu-id="a92d0-313">`loadFailed` è impostato su `true` , che viene utilizzato per visualizzare un messaggio di errore all'utente.</span><span class="sxs-lookup"><span data-stu-id="a92d0-313">`loadFailed` is set to `true`, which is used to display an error message to the user.</span></span>
  * <span data-ttu-id="a92d0-314">L'errore viene registrato.</span><span class="sxs-lookup"><span data-stu-id="a92d0-314">The error is logged.</span></span>

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_Server/Pages/handle-errors/ProductDetails.razor?name=snippet&highlight=11,27-39)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_Server/Pages/handle-errors/ProductDetails.razor?name=snippet&highlight=11,27-39)]

::: moniker-end

<h3 id="rendering-logic-server"><span data-ttu-id="a92d0-315">Logica di rendering</span><span class="sxs-lookup"><span data-stu-id="a92d0-315">Rendering logic</span></span></h3>

<span data-ttu-id="a92d0-316">Il markup dichiarativo in un Razor file componente ( `.razor` ) viene compilato in un metodo C# denominato <xref:Microsoft.AspNetCore.Components.ComponentBase.BuildRenderTree%2A> .</span><span class="sxs-lookup"><span data-stu-id="a92d0-316">The declarative markup in a Razor component file (`.razor`) is compiled into a C# method called <xref:Microsoft.AspNetCore.Components.ComponentBase.BuildRenderTree%2A>.</span></span> <span data-ttu-id="a92d0-317">Quando viene eseguito il rendering di un componente, <xref:Microsoft.AspNetCore.Components.ComponentBase.BuildRenderTree%2A> viene eseguita e compilata una struttura di dati che descrive gli elementi, il testo e i componenti figlio del componente di cui è stato eseguito il rendering.</span><span class="sxs-lookup"><span data-stu-id="a92d0-317">When a component renders, <xref:Microsoft.AspNetCore.Components.ComponentBase.BuildRenderTree%2A> executes and builds up a data structure describing the elements, text, and child components of the rendered component.</span></span>

<span data-ttu-id="a92d0-318">La logica di rendering può generare un'eccezione.</span><span class="sxs-lookup"><span data-stu-id="a92d0-318">Rendering logic can throw an exception.</span></span> <span data-ttu-id="a92d0-319">Un esempio di questo scenario si verifica quando `@someObject.PropertyName` viene valutato ma `@someObject` è `null` .</span><span class="sxs-lookup"><span data-stu-id="a92d0-319">An example of this scenario occurs when `@someObject.PropertyName` is evaluated but `@someObject` is `null`.</span></span> <span data-ttu-id="a92d0-320">Un'eccezione non gestita generata dalla logica di rendering è irreversibile per un Blazor Server circuito.</span><span class="sxs-lookup"><span data-stu-id="a92d0-320">An unhandled exception thrown by rendering logic is fatal to a Blazor Server circuit.</span></span>

<span data-ttu-id="a92d0-321">Per impedire una <xref:System.NullReferenceException> nella logica di rendering, verificare la presenza di un `null` oggetto prima di accedere ai relativi membri.</span><span class="sxs-lookup"><span data-stu-id="a92d0-321">To prevent a <xref:System.NullReferenceException> in rendering logic, check for a `null` object before accessing its members.</span></span> <span data-ttu-id="a92d0-322">Nell'esempio seguente, `person.Address` non è possibile accedere alle proprietà se `person.Address` è `null` :</span><span class="sxs-lookup"><span data-stu-id="a92d0-322">In the following example, `person.Address` properties aren't accessed if `person.Address` is `null`:</span></span>

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_Server/Pages/handle-errors/PersonExample.razor?name=snippet&highlight=1)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_Server/Pages/handle-errors/PersonExample.razor?name=snippet&highlight=1)]

::: moniker-end

<span data-ttu-id="a92d0-323">Il codice precedente presuppone che `person` non sia `null` .</span><span class="sxs-lookup"><span data-stu-id="a92d0-323">The preceding code assumes that `person` isn't `null`.</span></span> <span data-ttu-id="a92d0-324">Spesso, la struttura del codice garantisce la presenza di un oggetto nel momento in cui viene eseguito il rendering del componente.</span><span class="sxs-lookup"><span data-stu-id="a92d0-324">Often, the structure of the code guarantees that an object exists at the time the component is rendered.</span></span> <span data-ttu-id="a92d0-325">In questi casi, non è necessario verificare la presenza di `null` nella logica di rendering.</span><span class="sxs-lookup"><span data-stu-id="a92d0-325">In those cases, it isn't necessary to check for `null` in rendering logic.</span></span> <span data-ttu-id="a92d0-326">Nell'esempio precedente, `person` potrebbe essere garantito che esista perché `person` viene creato quando viene creata un'istanza del componente, come illustrato nell'esempio seguente:</span><span class="sxs-lookup"><span data-stu-id="a92d0-326">In the prior example, `person` might be guaranteed to exist because `person` is created when the component is instantiated, as the following example shows:</span></span>

```razor
@code {
    private Person person = new Person();

    ...
}
```

<h3 id="event-handlers-server"><span data-ttu-id="a92d0-327">Gestori eventi</span><span class="sxs-lookup"><span data-stu-id="a92d0-327">Event handlers</span></span></h3>

<span data-ttu-id="a92d0-328">Il codice lato client attiva le chiamate del codice C# quando i gestori eventi vengono creati usando:</span><span class="sxs-lookup"><span data-stu-id="a92d0-328">Client-side code triggers invocations of C# code when event handlers are created using:</span></span>

* `@onclick`
* `@onchange`
* <span data-ttu-id="a92d0-329">Altri `@on...` attributi</span><span class="sxs-lookup"><span data-stu-id="a92d0-329">Other `@on...` attributes</span></span>
* `@bind`

<span data-ttu-id="a92d0-330">Il codice del gestore eventi potrebbe generare un'eccezione non gestita in questi scenari.</span><span class="sxs-lookup"><span data-stu-id="a92d0-330">Event handler code might throw an unhandled exception in these scenarios.</span></span>

<span data-ttu-id="a92d0-331">Se un gestore eventi genera un'eccezione non gestita, ad esempio una query di database ha esito negativo, l'eccezione è fatale per un Blazor Server circuito.</span><span class="sxs-lookup"><span data-stu-id="a92d0-331">If an event handler throws an unhandled exception (for example, a database query fails), the exception is fatal to a Blazor Server circuit.</span></span> <span data-ttu-id="a92d0-332">Se l'app chiama il codice che potrebbe non riuscire per motivi esterni, intercettare le eccezioni usando un' [`try-catch`](/dotnet/csharp/language-reference/keywords/try-catch) istruzione con gestione e registrazione degli errori.</span><span class="sxs-lookup"><span data-stu-id="a92d0-332">If the app calls code that could fail for external reasons, trap exceptions using a [`try-catch`](/dotnet/csharp/language-reference/keywords/try-catch) statement with error handling and logging.</span></span>

<span data-ttu-id="a92d0-333">Se il codice utente non intercetta e gestisce l'eccezione, il Framework registra l'eccezione e termina il circuito.</span><span class="sxs-lookup"><span data-stu-id="a92d0-333">If user code doesn't trap and handle the exception, the framework logs the exception and terminates the circuit.</span></span>

<h3 id="component-disposal-server"><span data-ttu-id="a92d0-334">Eliminazione componenti</span><span class="sxs-lookup"><span data-stu-id="a92d0-334">Component disposal</span></span></h3>

<span data-ttu-id="a92d0-335">Un componente può essere rimosso dall'interfaccia utente, ad esempio perché l'utente ha esplorato un'altra pagina.</span><span class="sxs-lookup"><span data-stu-id="a92d0-335">A component may be removed from the UI, for example, because the user has navigated to another page.</span></span> <span data-ttu-id="a92d0-336">Quando un componente che implementa <xref:System.IDisposable?displayProperty=fullName> viene rimosso dall'interfaccia utente, il Framework chiama il metodo del componente <xref:System.IDisposable.Dispose%2A> .</span><span class="sxs-lookup"><span data-stu-id="a92d0-336">When a component that implements <xref:System.IDisposable?displayProperty=fullName> is removed from the UI, the framework calls the component's <xref:System.IDisposable.Dispose%2A> method.</span></span>

<span data-ttu-id="a92d0-337">Se il metodo del componente `Dispose` genera un'eccezione non gestita, l'eccezione è fatale per un Blazor Server circuito.</span><span class="sxs-lookup"><span data-stu-id="a92d0-337">If the component's `Dispose` method throws an unhandled exception, the exception is fatal to a Blazor Server circuit.</span></span> <span data-ttu-id="a92d0-338">Se la logica di eliminazione può generare eccezioni, l'app deve intercettare le eccezioni usando un' [`try-catch`](/dotnet/csharp/language-reference/keywords/try-catch) istruzione con gestione e registrazione degli errori.</span><span class="sxs-lookup"><span data-stu-id="a92d0-338">If disposal logic may throw exceptions, the app should trap the exceptions using a [`try-catch`](/dotnet/csharp/language-reference/keywords/try-catch) statement with error handling and logging.</span></span>

<span data-ttu-id="a92d0-339">Per ulteriori informazioni sull'eliminazione dei componenti, vedere <xref:blazor/components/lifecycle#component-disposal-with-idisposable> .</span><span class="sxs-lookup"><span data-stu-id="a92d0-339">For more information on component disposal, see <xref:blazor/components/lifecycle#component-disposal-with-idisposable>.</span></span>

<h3 id="javascript-interop-server"><span data-ttu-id="a92d0-340">Interoperabilità JavaScript</span><span class="sxs-lookup"><span data-stu-id="a92d0-340">JavaScript interop</span></span></h3>

<span data-ttu-id="a92d0-341"><xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A?displayProperty=nameWithType> consente al codice .NET di effettuare chiamate asincrone al runtime JavaScript nel browser dell'utente.</span><span class="sxs-lookup"><span data-stu-id="a92d0-341"><xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A?displayProperty=nameWithType> allows .NET code to make asynchronous calls to the JavaScript runtime in the user's browser.</span></span>

<span data-ttu-id="a92d0-342">Le condizioni seguenti si applicano alla gestione degli errori con <xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A> :</span><span class="sxs-lookup"><span data-stu-id="a92d0-342">The following conditions apply to error handling with <xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A>:</span></span>

* <span data-ttu-id="a92d0-343">Se una chiamata a ha <xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A> esito negativo in modo sincrono, si verifica un'eccezione .NET.</span><span class="sxs-lookup"><span data-stu-id="a92d0-343">If a call to <xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A> fails synchronously, a .NET exception occurs.</span></span> <span data-ttu-id="a92d0-344">Una chiamata a <xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A> potrebbe non riuscire, ad esempio perché gli argomenti forniti non possono essere serializzati.</span><span class="sxs-lookup"><span data-stu-id="a92d0-344">A call to <xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A> may fail, for example, because the supplied arguments can't be serialized.</span></span> <span data-ttu-id="a92d0-345">Il codice dello sviluppatore deve intercettare l'eccezione.</span><span class="sxs-lookup"><span data-stu-id="a92d0-345">Developer code must catch the exception.</span></span> <span data-ttu-id="a92d0-346">Se il codice dell'app in un gestore eventi o in un metodo del ciclo di vita dei componenti non gestisce un'eccezione, l'eccezione risultante è fatale per un Blazor Server circuito.</span><span class="sxs-lookup"><span data-stu-id="a92d0-346">If app code in an event handler or component lifecycle method doesn't handle an exception, the resulting exception is fatal to a Blazor Server circuit.</span></span>
* <span data-ttu-id="a92d0-347">Se una chiamata a ha <xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A> esito negativo in modo asincrono, .NET ha <xref:System.Threading.Tasks.Task> esito negativo.</span><span class="sxs-lookup"><span data-stu-id="a92d0-347">If a call to <xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A> fails asynchronously, the .NET <xref:System.Threading.Tasks.Task> fails.</span></span> <span data-ttu-id="a92d0-348">Una chiamata a <xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A> potrebbe non riuscire, ad esempio perché il codice sul lato JavaScript genera un'eccezione o restituisce un oggetto `Promise` completato come `rejected` .</span><span class="sxs-lookup"><span data-stu-id="a92d0-348">A call to <xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A> may fail, for example, because the JavaScript-side code throws an exception or returns a `Promise` that completed as `rejected`.</span></span> <span data-ttu-id="a92d0-349">Il codice dello sviluppatore deve intercettare l'eccezione.</span><span class="sxs-lookup"><span data-stu-id="a92d0-349">Developer code must catch the exception.</span></span> <span data-ttu-id="a92d0-350">Se si utilizza l' [`await`](/dotnet/csharp/language-reference/keywords/await) operatore, è consigliabile eseguire il wrapping della chiamata al metodo in un' [`try-catch`](/dotnet/csharp/language-reference/keywords/try-catch) istruzione con gestione e registrazione degli errori.</span><span class="sxs-lookup"><span data-stu-id="a92d0-350">If using the [`await`](/dotnet/csharp/language-reference/keywords/await) operator, consider wrapping the method call in a [`try-catch`](/dotnet/csharp/language-reference/keywords/try-catch) statement with error handling and logging.</span></span> <span data-ttu-id="a92d0-351">In caso contrario, il codice in errore genera un'eccezione non gestita che è irreversibile per un Blazor Server circuito.</span><span class="sxs-lookup"><span data-stu-id="a92d0-351">Otherwise, the failing code results in an unhandled exception that's fatal to a Blazor Server circuit.</span></span>
* <span data-ttu-id="a92d0-352">Per impostazione predefinita, le chiamate a <xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A> devono essere completate entro un determinato periodo oppure si verifica il timeout della chiamata. Il periodo di timeout predefinito è di un minuto.</span><span class="sxs-lookup"><span data-stu-id="a92d0-352">By default, calls to <xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A> must complete within a certain period or else the call times out. The default timeout period is one minute.</span></span> <span data-ttu-id="a92d0-353">Il timeout protegge il codice da una perdita di connettività di rete o codice JavaScript che non restituisce mai un messaggio di completamento.</span><span class="sxs-lookup"><span data-stu-id="a92d0-353">The timeout protects the code against a loss in network connectivity or JavaScript code that never sends back a completion message.</span></span> <span data-ttu-id="a92d0-354">Se si verifica il timeout della chiamata, l'oggetto risultante ha <xref:System.Threading.Tasks> esito negativo con un oggetto <xref:System.OperationCanceledException> .</span><span class="sxs-lookup"><span data-stu-id="a92d0-354">If the call times out, the resulting <xref:System.Threading.Tasks> fails with an <xref:System.OperationCanceledException>.</span></span> <span data-ttu-id="a92d0-355">Intercettare ed elaborare l'eccezione con la registrazione.</span><span class="sxs-lookup"><span data-stu-id="a92d0-355">Trap and process the exception with logging.</span></span>

<span data-ttu-id="a92d0-356">Analogamente, il codice JavaScript può avviare chiamate a metodi .NET indicati dall' [ `[JSInvokable]` attributo](xref:blazor/call-dotnet-from-javascript).</span><span class="sxs-lookup"><span data-stu-id="a92d0-356">Similarly, JavaScript code may initiate calls to .NET methods indicated by the [`[JSInvokable]` attribute](xref:blazor/call-dotnet-from-javascript).</span></span> <span data-ttu-id="a92d0-357">Se questi metodi .NET generano un'eccezione non gestita:</span><span class="sxs-lookup"><span data-stu-id="a92d0-357">If these .NET methods throw an unhandled exception:</span></span>

* <span data-ttu-id="a92d0-358">L'eccezione non viene trattata come irreversibile per un Blazor Server circuito.</span><span class="sxs-lookup"><span data-stu-id="a92d0-358">The exception isn't treated as fatal to a Blazor Server circuit.</span></span>
* <span data-ttu-id="a92d0-359">Il lato JavaScript `Promise` viene rifiutato.</span><span class="sxs-lookup"><span data-stu-id="a92d0-359">The JavaScript-side `Promise` is rejected.</span></span>

<span data-ttu-id="a92d0-360">È possibile scegliere di usare il codice di gestione degli errori sul lato .NET o sul lato JavaScript della chiamata al metodo.</span><span class="sxs-lookup"><span data-stu-id="a92d0-360">You have the option of using error handling code on either the .NET side or the JavaScript side of the method call.</span></span>

<span data-ttu-id="a92d0-361">Per altre informazioni, vedere gli articoli seguenti:</span><span class="sxs-lookup"><span data-stu-id="a92d0-361">For more information, see the following articles:</span></span>

* <xref:blazor/call-javascript-from-dotnet>
* <xref:blazor/call-dotnet-from-javascript>

<h3 id="blazor-server-prerendering-server"><span data-ttu-id="a92d0-362">Blazor Server tempistiche</span><span class="sxs-lookup"><span data-stu-id="a92d0-362">Blazor Server prerendering</span></span></h3>

<span data-ttu-id="a92d0-363">Blazor è possibile eseguire il prerendering dei componenti usando l' [Helper Tag Component](xref:mvc/views/tag-helpers/builtin-th/component-tag-helper) , in modo che il markup HTML sottoposto a rendering venga restituito come parte della richiesta HTTP iniziale dell'utente.</span><span class="sxs-lookup"><span data-stu-id="a92d0-363">Blazor components can be prerendered using the [Component Tag Helper](xref:mvc/views/tag-helpers/builtin-th/component-tag-helper) so that their rendered HTML markup is returned as part of the user's initial HTTP request.</span></span> <span data-ttu-id="a92d0-364">Funziona per:</span><span class="sxs-lookup"><span data-stu-id="a92d0-364">This works by:</span></span>

* <span data-ttu-id="a92d0-365">Creazione di un nuovo circuito per tutti i componenti di cui è stato eseguito il rendering che fanno parte della stessa pagina.</span><span class="sxs-lookup"><span data-stu-id="a92d0-365">Creating a new circuit for all of the prerendered components that are part of the same page.</span></span>
* <span data-ttu-id="a92d0-366">Generazione del codice HTML iniziale.</span><span class="sxs-lookup"><span data-stu-id="a92d0-366">Generating the initial HTML.</span></span>
* <span data-ttu-id="a92d0-367">Trattare il circuito come `disconnected` fino a quando il browser dell'utente non stabilisce una SignalR connessione allo stesso server.</span><span class="sxs-lookup"><span data-stu-id="a92d0-367">Treating the circuit as `disconnected` until the user's browser establishes a SignalR connection back to the same server.</span></span> <span data-ttu-id="a92d0-368">Quando viene stabilita la connessione, interattività sul circuito viene ripresa e il markup HTML dei componenti viene aggiornato.</span><span class="sxs-lookup"><span data-stu-id="a92d0-368">When the connection is established, interactivity on the circuit is resumed and the components' HTML markup is updated.</span></span>

<span data-ttu-id="a92d0-369">Se un componente genera un'eccezione non gestita durante il prerendering, ad esempio durante un metodo del ciclo di vita o nella logica di rendering:</span><span class="sxs-lookup"><span data-stu-id="a92d0-369">If any component throws an unhandled exception during prerendering, for example, during a lifecycle method or in rendering logic:</span></span>

* <span data-ttu-id="a92d0-370">L'eccezione è fatale per il circuito.</span><span class="sxs-lookup"><span data-stu-id="a92d0-370">The exception is fatal to the circuit.</span></span>
* <span data-ttu-id="a92d0-371">L'eccezione viene generata dallo stack di chiamate dall' <xref:Microsoft.AspNetCore.Mvc.TagHelpers.ComponentTagHelper> Helper tag.</span><span class="sxs-lookup"><span data-stu-id="a92d0-371">The exception is thrown up the call stack from the <xref:Microsoft.AspNetCore.Mvc.TagHelpers.ComponentTagHelper> Tag Helper.</span></span> <span data-ttu-id="a92d0-372">Pertanto, l'intera richiesta HTTP ha esito negativo a meno che l'eccezione non venga intercettata in modo esplicito dal codice dello sviluppatore.</span><span class="sxs-lookup"><span data-stu-id="a92d0-372">Therefore, the entire HTTP request fails unless the exception is explicitly caught by developer code.</span></span>

<span data-ttu-id="a92d0-373">In circostanze normali, quando si verifica un errore di prerendering, continuare a compilare ed eseguire il rendering del componente non ha senso perché non è possibile eseguire il rendering di un componente funzionante.</span><span class="sxs-lookup"><span data-stu-id="a92d0-373">Under normal circumstances when prerendering fails, continuing to build and render the component doesn't make sense because a working component can't be rendered.</span></span>

<span data-ttu-id="a92d0-374">Per tollerare gli errori che possono verificarsi durante il prerendering, la logica di gestione degli errori deve essere inserita all'interno di un componente che può generare eccezioni.</span><span class="sxs-lookup"><span data-stu-id="a92d0-374">To tolerate errors that may occur during prerendering, error handling logic must be placed inside a component that may throw exceptions.</span></span> <span data-ttu-id="a92d0-375">Usare [`try-catch`](/dotnet/csharp/language-reference/keywords/try-catch) le istruzioni con la gestione e la registrazione degli errori.</span><span class="sxs-lookup"><span data-stu-id="a92d0-375">Use [`try-catch`](/dotnet/csharp/language-reference/keywords/try-catch) statements with error handling and logging.</span></span> <span data-ttu-id="a92d0-376">Anziché eseguire il wrapping dell' <xref:Microsoft.AspNetCore.Mvc.TagHelpers.ComponentTagHelper> Helper tag in un' [`try-catch`](/dotnet/csharp/language-reference/keywords/try-catch) istruzione, inserire la logica di gestione degli errori nel componente di cui viene eseguito il rendering dall' <xref:Microsoft.AspNetCore.Mvc.TagHelpers.ComponentTagHelper> Helper tag.</span><span class="sxs-lookup"><span data-stu-id="a92d0-376">Instead of wrapping the <xref:Microsoft.AspNetCore.Mvc.TagHelpers.ComponentTagHelper> Tag Helper in a [`try-catch`](/dotnet/csharp/language-reference/keywords/try-catch) statement, place error handling logic in the component rendered by the <xref:Microsoft.AspNetCore.Mvc.TagHelpers.ComponentTagHelper> Tag Helper.</span></span>

## <a name="advanced-scenarios"></a><span data-ttu-id="a92d0-377">Scenari avanzati</span><span class="sxs-lookup"><span data-stu-id="a92d0-377">Advanced scenarios</span></span>

### <a name="recursive-rendering"></a><span data-ttu-id="a92d0-378">Rendering ricorsivo</span><span class="sxs-lookup"><span data-stu-id="a92d0-378">Recursive rendering</span></span>

<span data-ttu-id="a92d0-379">I componenti possono essere annidati in modo ricorsivo.</span><span class="sxs-lookup"><span data-stu-id="a92d0-379">Components can be nested recursively.</span></span> <span data-ttu-id="a92d0-380">Questa operazione è utile per la rappresentazione di strutture di dati ricorsive.</span><span class="sxs-lookup"><span data-stu-id="a92d0-380">This is useful for representing recursive data structures.</span></span> <span data-ttu-id="a92d0-381">Un componente, ad esempio, `TreeNode` può eseguire il rendering di più `TreeNode` componenti per ognuno dei figli del nodo.</span><span class="sxs-lookup"><span data-stu-id="a92d0-381">For example, a `TreeNode` component can render more `TreeNode` components for each of the node's children.</span></span>

<span data-ttu-id="a92d0-382">Quando si esegue il rendering in modo ricorsivo, evitare i modelli di codifica che generano una ricorsione</span><span class="sxs-lookup"><span data-stu-id="a92d0-382">When rendering recursively, avoid coding patterns that result in infinite recursion:</span></span>

* <span data-ttu-id="a92d0-383">Non esegue il rendering in modo ricorsivo di una struttura di dati che contiene un ciclo.</span><span class="sxs-lookup"><span data-stu-id="a92d0-383">Don't recursively render a data structure that contains a cycle.</span></span> <span data-ttu-id="a92d0-384">Ad esempio, non eseguire il rendering di un nodo dell'albero i cui elementi figlio includono se stesso.</span><span class="sxs-lookup"><span data-stu-id="a92d0-384">For example, don't render a tree node whose children includes itself.</span></span>
* <span data-ttu-id="a92d0-385">Non creare una catena di layout che contiene un ciclo.</span><span class="sxs-lookup"><span data-stu-id="a92d0-385">Don't create a chain of layouts that contain a cycle.</span></span> <span data-ttu-id="a92d0-386">Non creare, ad esempio, un layout il cui layout è a sua volta.</span><span class="sxs-lookup"><span data-stu-id="a92d0-386">For example, don't create a layout whose layout is itself.</span></span>
* <span data-ttu-id="a92d0-387">Non consentire a un utente finale di violare le invarianti di ricorsione (regole) tramite immissione di dati dannosi o chiamate di interoperabilità JavaScript.</span><span class="sxs-lookup"><span data-stu-id="a92d0-387">Don't allow an end user to violate recursion invariants (rules) through malicious data entry or JavaScript interop calls.</span></span>

<span data-ttu-id="a92d0-388">Cicli infiniti durante il rendering:</span><span class="sxs-lookup"><span data-stu-id="a92d0-388">Infinite loops during rendering:</span></span>

* <span data-ttu-id="a92d0-389">Causa la continuazione del processo di rendering.</span><span class="sxs-lookup"><span data-stu-id="a92d0-389">Causes the rendering process to continue forever.</span></span>
* <span data-ttu-id="a92d0-390">Equivale alla creazione di un ciclo senza terminazione.</span><span class="sxs-lookup"><span data-stu-id="a92d0-390">Is equivalent to creating an unterminated loop.</span></span>

<span data-ttu-id="a92d0-391">In questi scenari, un circuito interessato ha Blazor Server esito negativo e il thread in genere tenta di:</span><span class="sxs-lookup"><span data-stu-id="a92d0-391">In these scenarios, an affected Blazor Server circuit fails, and the thread usually attempts to:</span></span>

* <span data-ttu-id="a92d0-392">Utilizzare la quantità di tempo della CPU consentita dal sistema operativo, per un periodo illimitato.</span><span class="sxs-lookup"><span data-stu-id="a92d0-392">Consume as much CPU time as permitted by the operating system, indefinitely.</span></span>
* <span data-ttu-id="a92d0-393">Utilizzare una quantità illimitata di memoria del server.</span><span class="sxs-lookup"><span data-stu-id="a92d0-393">Consume an unlimited amount of server memory.</span></span> <span data-ttu-id="a92d0-394">L'utilizzo di memoria illimitata equivale allo scenario in cui un ciclo senza terminazione aggiunge voci a una raccolta in ogni iterazione.</span><span class="sxs-lookup"><span data-stu-id="a92d0-394">Consuming unlimited memory is equivalent to the scenario where an unterminated loop adds entries to a collection on every iteration.</span></span>

<span data-ttu-id="a92d0-395">Per evitare modelli di ricorsione infinita, verificare che il codice di rendering ricorsivo contenga condizioni di arresto appropriate.</span><span class="sxs-lookup"><span data-stu-id="a92d0-395">To avoid infinite recursion patterns, ensure that recursive rendering code contains suitable stopping conditions.</span></span>

### <a name="custom-render-tree-logic"></a><span data-ttu-id="a92d0-396">Logica dell'albero di rendering personalizzata</span><span class="sxs-lookup"><span data-stu-id="a92d0-396">Custom render tree logic</span></span>

<span data-ttu-id="a92d0-397">La maggior parte dei Blazor componenti viene implementata come Razor file componente ( `.razor` ) e viene compilata dal Framework per produrre la logica che opera su un <xref:Microsoft.AspNetCore.Components.Rendering.RenderTreeBuilder> per eseguire il rendering dell'output.</span><span class="sxs-lookup"><span data-stu-id="a92d0-397">Most Blazor components are implemented as Razor component files (`.razor`) and are compiled by the framework to produce logic that operates on a <xref:Microsoft.AspNetCore.Components.Rendering.RenderTreeBuilder> to render their output.</span></span> <span data-ttu-id="a92d0-398">Tuttavia, uno sviluppatore può implementare manualmente la <xref:Microsoft.AspNetCore.Components.Rendering.RenderTreeBuilder> logica usando il codice C# procedurale.</span><span class="sxs-lookup"><span data-stu-id="a92d0-398">However, a developer may manually implement <xref:Microsoft.AspNetCore.Components.Rendering.RenderTreeBuilder> logic using procedural C# code.</span></span> <span data-ttu-id="a92d0-399">Per altre informazioni, vedere <xref:blazor/advanced-scenarios#manual-rendertreebuilder-logic>.</span><span class="sxs-lookup"><span data-stu-id="a92d0-399">For more information, see <xref:blazor/advanced-scenarios#manual-rendertreebuilder-logic>.</span></span>

> [!WARNING]
> <span data-ttu-id="a92d0-400">L'uso della logica del generatore di albero di rendering manuale è considerato uno scenario avanzato e non sicuro, non consigliato per lo sviluppo di componenti generali.</span><span class="sxs-lookup"><span data-stu-id="a92d0-400">Use of manual render tree builder logic is considered an advanced and unsafe scenario, not recommended for general component development.</span></span>

<span data-ttu-id="a92d0-401">Se il <xref:Microsoft.AspNetCore.Components.Rendering.RenderTreeBuilder> codice viene scritto, lo sviluppatore deve garantire la correttezza del codice.</span><span class="sxs-lookup"><span data-stu-id="a92d0-401">If <xref:Microsoft.AspNetCore.Components.Rendering.RenderTreeBuilder> code is written, the developer must guarantee the correctness of the code.</span></span> <span data-ttu-id="a92d0-402">Ad esempio, lo sviluppatore deve garantire quanto segue:</span><span class="sxs-lookup"><span data-stu-id="a92d0-402">For example, the developer must ensure that:</span></span>

* <span data-ttu-id="a92d0-403">Le chiamate a <xref:Microsoft.AspNetCore.Components.Rendering.RenderTreeBuilder.OpenElement%2A> e <xref:Microsoft.AspNetCore.Components.Rendering.RenderTreeBuilder.CloseElement%2A> sono bilanciate correttamente.</span><span class="sxs-lookup"><span data-stu-id="a92d0-403">Calls to <xref:Microsoft.AspNetCore.Components.Rendering.RenderTreeBuilder.OpenElement%2A> and <xref:Microsoft.AspNetCore.Components.Rendering.RenderTreeBuilder.CloseElement%2A> are correctly balanced.</span></span>
* <span data-ttu-id="a92d0-404">Gli attributi vengono aggiunti solo nei punti corretti.</span><span class="sxs-lookup"><span data-stu-id="a92d0-404">Attributes are only added in the correct places.</span></span>

<span data-ttu-id="a92d0-405">La logica del generatore di albero di rendering manuale non corretta può causare un comportamento arbitrario non definito, tra cui arresti anomali, blocchi del server e vulnerabilità della sicurezza.</span><span class="sxs-lookup"><span data-stu-id="a92d0-405">Incorrect manual render tree builder logic can cause arbitrary undefined behavior, including crashes, server hangs, and security vulnerabilities.</span></span>

<span data-ttu-id="a92d0-406">Prendere in considerazione la logica del generatore di albero di rendering manuale sullo stesso livello di complessità e con lo stesso livello di *rischio* di scrivere manualmente codice assembly o istruzioni [MSIL (Microsoft Intermediate Language)](/dotnet/standard/managed-code) .</span><span class="sxs-lookup"><span data-stu-id="a92d0-406">Consider manual render tree builder logic on the same level of complexity and with the same level of *danger* as writing assembly code or [Microsoft Intermediate Language (MSIL)](/dotnet/standard/managed-code) instructions by hand.</span></span>

## <a name="additional-resources"></a><span data-ttu-id="a92d0-407">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="a92d0-407">Additional resources</span></span>

* <xref:blazor/fundamentals/logging>
* <xref:fundamentals/error-handling>&dagger;

<span data-ttu-id="a92d0-408">&dagger;Si applica alle app ASP.NET Core lato server che sono app back-end dell'API Web per le Blazor app.</span><span class="sxs-lookup"><span data-stu-id="a92d0-408">&dagger;Applies to server-side ASP.NET Core apps that are web API backend apps for Blazor apps.</span></span>

::: zone-end
