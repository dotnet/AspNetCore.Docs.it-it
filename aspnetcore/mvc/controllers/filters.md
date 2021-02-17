---
title: Filtri in ASP.NET Core
author: Rick-Anderson
description: Informazioni sul funzionamento dei filtri e su come usarli in ASP.NET Core.
ms.author: riande
ms.custom: mvc
ms.date: 02/04/2020
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
uid: mvc/controllers/filters
ms.openlocfilehash: 416fc292d82cf841b2134e23a8e494e3e8d945ca
ms.sourcegitcommit: f77a7467651bab61b24261da9dc5c1dd75fc1fa9
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/17/2021
ms.locfileid: "100563983"
---
# <a name="filters-in-aspnet-core"></a><span data-ttu-id="42f92-103">Filtri in ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="42f92-103">Filters in ASP.NET Core</span></span>

::: moniker range=">= aspnetcore-3.0"

<span data-ttu-id="42f92-104">Di [Kirk Larkin](https://github.com/serpent5), [Rick Anderson](https://twitter.com/RickAndMSFT), [Tom Dykstra](https://github.com/tdykstra/) e [Steve Smith](https://ardalis.com/)</span><span class="sxs-lookup"><span data-stu-id="42f92-104">By [Kirk Larkin](https://github.com/serpent5), [Rick Anderson](https://twitter.com/RickAndMSFT), [Tom Dykstra](https://github.com/tdykstra/), and [Steve Smith](https://ardalis.com/)</span></span>

<span data-ttu-id="42f92-105">I *filtri* in ASP.NET Core consentono l'esecuzione del codice prima o dopo fasi specifiche della pipeline di elaborazione della richiesta.</span><span class="sxs-lookup"><span data-stu-id="42f92-105">*Filters* in ASP.NET Core allow code to be run before or after specific stages in the request processing pipeline.</span></span>

<span data-ttu-id="42f92-106">I filtri predefiniti gestiscono attività, ad esempio:</span><span class="sxs-lookup"><span data-stu-id="42f92-106">Built-in filters handle tasks such as:</span></span>

* <span data-ttu-id="42f92-107">Autorizzazione (impedire l'accesso alle risorse per cui un utente non è autorizzato).</span><span class="sxs-lookup"><span data-stu-id="42f92-107">Authorization (preventing access to resources a user isn't authorized for).</span></span>
* <span data-ttu-id="42f92-108">Memorizzazione nella cache delle risposte (blocco della pipeline delle richieste per restituire una risposta memorizzata nella cache).</span><span class="sxs-lookup"><span data-stu-id="42f92-108">Response caching (short-circuiting the request pipeline to return a cached response).</span></span>

<span data-ttu-id="42f92-109">I filtri personalizzati possono essere creati per gestire problemi relativi a più settori.</span><span class="sxs-lookup"><span data-stu-id="42f92-109">Custom filters can be created to handle cross-cutting concerns.</span></span> <span data-ttu-id="42f92-110">Esempi di problematiche trasversali includono la gestione degli errori, la memorizzazione nella cache, la configurazione, l'autorizzazione e la registrazione.</span><span class="sxs-lookup"><span data-stu-id="42f92-110">Examples of cross-cutting concerns include error handling, caching, configuration, authorization, and logging.</span></span>  <span data-ttu-id="42f92-111">I filtri evitano la duplicazione del codice.</span><span class="sxs-lookup"><span data-stu-id="42f92-111">Filters avoid duplicating code.</span></span> <span data-ttu-id="42f92-112">Ad esempio, un filtro eccezioni per la gestione degli errori potrebbe consolidare la gestione degli errori.</span><span class="sxs-lookup"><span data-stu-id="42f92-112">For example, an error handling exception filter could consolidate error handling.</span></span>

<span data-ttu-id="42f92-113">Questo documento si applica a Razor pagine, controller API e controller con visualizzazioni.</span><span class="sxs-lookup"><span data-stu-id="42f92-113">This document applies to Razor Pages, API controllers, and controllers with views.</span></span> <span data-ttu-id="42f92-114">I filtri non funzionano direttamente con i [ Razor componenti](xref:blazor/components/index).</span><span class="sxs-lookup"><span data-stu-id="42f92-114">Filters don't work directly with [Razor components](xref:blazor/components/index).</span></span> <span data-ttu-id="42f92-115">Un filtro può influire indirettamente su un componente solo quando:</span><span class="sxs-lookup"><span data-stu-id="42f92-115">A filter can only indirectly affect a component when:</span></span>

* <span data-ttu-id="42f92-116">Il componente è incorporato in una pagina o in una vista.</span><span class="sxs-lookup"><span data-stu-id="42f92-116">The component is embedded in a page or view.</span></span>
* <span data-ttu-id="42f92-117">La pagina o il controller/Visualizzazione usa il filtro.</span><span class="sxs-lookup"><span data-stu-id="42f92-117">The page or controller/view uses the filter.</span></span>

<span data-ttu-id="42f92-118">[Visualizzare o scaricare un esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/controllers/filters/3.1sample) ([procedura per il download](xref:index#how-to-download-a-sample)).</span><span class="sxs-lookup"><span data-stu-id="42f92-118">[View or download sample](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/controllers/filters/3.1sample) ([how to download](xref:index#how-to-download-a-sample)).</span></span>

## <a name="how-filters-work"></a><span data-ttu-id="42f92-119">Funzionamento dei filtri</span><span class="sxs-lookup"><span data-stu-id="42f92-119">How filters work</span></span>

<span data-ttu-id="42f92-120">I filtri vengono eseguiti all'interno della *pipeline di chiamata di azioni ASP.NET Core*, talvolta definita *pipeline di filtro*.</span><span class="sxs-lookup"><span data-stu-id="42f92-120">Filters run within the *ASP.NET Core action invocation pipeline*, sometimes referred to as the *filter pipeline*.</span></span> <span data-ttu-id="42f92-121">La pipeline di filtro viene eseguita dopo che ASP.NET Core ha selezionato l'azione da eseguire.</span><span class="sxs-lookup"><span data-stu-id="42f92-121">The filter pipeline runs after ASP.NET Core selects the action to execute.</span></span>

![La richiesta viene elaborata tramite altri middleware, middleware di routing, selezione dell'azione e pipeline di chiamata dell'azione.](filters/_static/filter-pipeline-1.png)

### <a name="filter-types"></a><span data-ttu-id="42f92-124">Tipi di filtro</span><span class="sxs-lookup"><span data-stu-id="42f92-124">Filter types</span></span>

<span data-ttu-id="42f92-125">Ogni tipo di filtro viene eseguito in una fase diversa della pipeline di filtro:</span><span class="sxs-lookup"><span data-stu-id="42f92-125">Each filter type is executed at a different stage in the filter pipeline:</span></span>

* <span data-ttu-id="42f92-126">I [filtri di autorizzazione](#authorization-filters) vengono eseguiti per primi e consentono di determinare se l'utente è autorizzato per la richiesta.</span><span class="sxs-lookup"><span data-stu-id="42f92-126">[Authorization filters](#authorization-filters) run first and are used to determine whether the user is authorized for the request.</span></span> <span data-ttu-id="42f92-127">I filtri di autorizzazione consentono di cortocircuitare la pipeline se la richiesta non è autorizzata.</span><span class="sxs-lookup"><span data-stu-id="42f92-127">Authorization filters short-circuit the pipeline if the request is not authorized.</span></span>

* <span data-ttu-id="42f92-128">[Filtri risorse](#resource-filters):</span><span class="sxs-lookup"><span data-stu-id="42f92-128">[Resource filters](#resource-filters):</span></span>

  * <span data-ttu-id="42f92-129">Vengono eseguiti dopo l'autorizzazione.</span><span class="sxs-lookup"><span data-stu-id="42f92-129">Run after authorization.</span></span>  
  * <span data-ttu-id="42f92-130"><xref:Microsoft.AspNetCore.Mvc.Filters.IResourceFilter.OnResourceExecuting*> esegue il codice prima del resto della pipeline filtro.</span><span class="sxs-lookup"><span data-stu-id="42f92-130"><xref:Microsoft.AspNetCore.Mvc.Filters.IResourceFilter.OnResourceExecuting*> runs code before the rest of the filter pipeline.</span></span> <span data-ttu-id="42f92-131">Ad esempio, `OnResourceExecuting` esegue il codice prima dell'associazione di modelli.</span><span class="sxs-lookup"><span data-stu-id="42f92-131">For example, `OnResourceExecuting` runs code before model binding.</span></span>
  * <span data-ttu-id="42f92-132"><xref:Microsoft.AspNetCore.Mvc.Filters.IResourceFilter.OnResourceExecuted*> esegue il codice una volta completato il resto della pipeline.</span><span class="sxs-lookup"><span data-stu-id="42f92-132"><xref:Microsoft.AspNetCore.Mvc.Filters.IResourceFilter.OnResourceExecuted*> runs code after the rest of the pipeline has completed.</span></span>

* <span data-ttu-id="42f92-133">[Filtri azione](#action-filters):</span><span class="sxs-lookup"><span data-stu-id="42f92-133">[Action filters](#action-filters):</span></span>

  * <span data-ttu-id="42f92-134">Eseguire il codice immediatamente prima e dopo la chiamata di un metodo di azione.</span><span class="sxs-lookup"><span data-stu-id="42f92-134">Run code immediately before and after an action method is called.</span></span>
  * <span data-ttu-id="42f92-135">Consente di modificare gli argomenti passati in un'azione.</span><span class="sxs-lookup"><span data-stu-id="42f92-135">Can change the arguments passed into an action.</span></span>
  * <span data-ttu-id="42f92-136">Può modificare il risultato restituito dall'azione.</span><span class="sxs-lookup"><span data-stu-id="42f92-136">Can change the result returned from the action.</span></span>
  * <span data-ttu-id="42f92-137">**Non** sono supportate nelle Razor pagine.</span><span class="sxs-lookup"><span data-stu-id="42f92-137">Are **not** supported in Razor Pages.</span></span>

* <span data-ttu-id="42f92-138">I [filtri eccezioni](#exception-filters) applicano i criteri globali alle eccezioni non gestite che si verificano prima della scrittura del corpo della risposta.</span><span class="sxs-lookup"><span data-stu-id="42f92-138">[Exception filters](#exception-filters) apply global policies to unhandled exceptions that occur before the response body has been written to.</span></span>

* <span data-ttu-id="42f92-139">I [filtri risultati](#result-filters) eseguono il codice immediatamente prima e dopo l'esecuzione dei risultati dell'azione.</span><span class="sxs-lookup"><span data-stu-id="42f92-139">[Result filters](#result-filters) run code immediately before and after the execution of action results.</span></span> <span data-ttu-id="42f92-140">Vengono eseguiti solo quando il metodo di azione è stato eseguito correttamente.</span><span class="sxs-lookup"><span data-stu-id="42f92-140">They run only when the action method has executed successfully.</span></span> <span data-ttu-id="42f92-141">Sono utili per la logica che deve racchiudere l'esecuzione del formattatore o di una vista.</span><span class="sxs-lookup"><span data-stu-id="42f92-141">They are useful for logic that must surround view or formatter execution.</span></span>

<span data-ttu-id="42f92-142">Il diagramma seguente illustra come interagiscono i tipi di filtro nella pipeline di filtro.</span><span class="sxs-lookup"><span data-stu-id="42f92-142">The following diagram shows how filter types interact in the filter pipeline.</span></span>

![La richiesta passa attraverso i filtri autorizzazione, i filtri risorse, l'associazione di modelli, i filtri azione, l'esecuzione dell'azione e la conversione del risultato dell'azione, i filtri eccezioni, i filtri risultato e l'esecuzione del risultato.](filters/_static/filter-pipeline-2.png)

## <a name="implementation"></a><span data-ttu-id="42f92-145">Implementazione</span><span class="sxs-lookup"><span data-stu-id="42f92-145">Implementation</span></span>

<span data-ttu-id="42f92-146">I filtri supportano sia implementazioni sincrone che asincrone tramite definizioni di interfaccia diverse.</span><span class="sxs-lookup"><span data-stu-id="42f92-146">Filters support both synchronous and asynchronous implementations through different interface definitions.</span></span>

<span data-ttu-id="42f92-147">I filtri sincroni eseguono codice prima e dopo la fase della pipeline.</span><span class="sxs-lookup"><span data-stu-id="42f92-147">Synchronous filters run code before and after their pipeline stage.</span></span> <span data-ttu-id="42f92-148">La chiamata di <xref:Microsoft.AspNetCore.Mvc.Controller.OnActionExecuting*>, ad esempio, avviene prima della chiamata del metodo di azione.</span><span class="sxs-lookup"><span data-stu-id="42f92-148">For example, <xref:Microsoft.AspNetCore.Mvc.Controller.OnActionExecuting*> is called before the action method is called.</span></span> <span data-ttu-id="42f92-149">La chiamata di <xref:Microsoft.AspNetCore.Mvc.Controller.OnActionExecuted*> avviene dopo l'esecuzione del metodo di azione.</span><span class="sxs-lookup"><span data-stu-id="42f92-149"><xref:Microsoft.AspNetCore.Mvc.Controller.OnActionExecuted*> is called after the action method returns.</span></span>

[!code-csharp[](./filters/3.1sample/FiltersSample/Filters/MySampleActionFilter.cs?name=snippet_ActionFilter)]

<span data-ttu-id="42f92-150">Nel codice precedente, il [debug](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/mvc/controllers/filters/3.1sample/FiltersSample/Helper/MyDebug.cs) è una funzione di utilità nell' [esempio di download](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/mvc/controllers/filters/3.1sample/FiltersSample/Helper/MyDebug.cs).</span><span class="sxs-lookup"><span data-stu-id="42f92-150">In the preceding code, [MyDebug](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/mvc/controllers/filters/3.1sample/FiltersSample/Helper/MyDebug.cs) is a utility function in the [sample download](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/mvc/controllers/filters/3.1sample/FiltersSample/Helper/MyDebug.cs).</span></span>

<span data-ttu-id="42f92-151">I filtri asincroni definiscono un `On-Stage-ExecutionAsync` metodo.</span><span class="sxs-lookup"><span data-stu-id="42f92-151">Asynchronous filters define an `On-Stage-ExecutionAsync` method.</span></span> <span data-ttu-id="42f92-152">Ad esempio, <xref:Microsoft.AspNetCore.Mvc.Controller.OnActionExecutionAsync*>:</span><span class="sxs-lookup"><span data-stu-id="42f92-152">For example, <xref:Microsoft.AspNetCore.Mvc.Controller.OnActionExecutionAsync*>:</span></span>

[!code-csharp[](./filters/3.1sample/FiltersSample/Filters/SampleAsyncActionFilter.cs?name=snippet)]

<span data-ttu-id="42f92-153">Nel codice precedente, `SampleAsyncActionFilter` ha un <xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutionDelegate> ( `next` ) che esegue il metodo di azione.</span><span class="sxs-lookup"><span data-stu-id="42f92-153">In the preceding code, the `SampleAsyncActionFilter` has an <xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutionDelegate> (`next`) that executes the action method.</span></span>

### <a name="multiple-filter-stages"></a><span data-ttu-id="42f92-154">Fasi di filtro multiple</span><span class="sxs-lookup"><span data-stu-id="42f92-154">Multiple filter stages</span></span>

<span data-ttu-id="42f92-155">È possibile implementare interfacce per più fasi di filtro in una singola classe.</span><span class="sxs-lookup"><span data-stu-id="42f92-155">Interfaces for multiple filter stages can be implemented in a single class.</span></span> <span data-ttu-id="42f92-156">Ad esempio, la <xref:Microsoft.AspNetCore.Mvc.Filters.ActionFilterAttribute> classe implementa:</span><span class="sxs-lookup"><span data-stu-id="42f92-156">For example, the <xref:Microsoft.AspNetCore.Mvc.Filters.ActionFilterAttribute> class implements:</span></span>

* <span data-ttu-id="42f92-157">Sincrona: <xref:Microsoft.AspNetCore.Mvc.Filters.IActionFilter> e  <xref:Microsoft.AspNetCore.Mvc.Filters.IResultFilter></span><span class="sxs-lookup"><span data-stu-id="42f92-157">Synchronous: <xref:Microsoft.AspNetCore.Mvc.Filters.IActionFilter> and  <xref:Microsoft.AspNetCore.Mvc.Filters.IResultFilter></span></span>
* <span data-ttu-id="42f92-158">Asincrono: <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncActionFilter> e <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncResultFilter></span><span class="sxs-lookup"><span data-stu-id="42f92-158">Asynchronous: <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncActionFilter> and <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncResultFilter></span></span>
* <xref:Microsoft.AspNetCore.Mvc.Filters.IOrderedFilter>

<span data-ttu-id="42f92-159">Implementare la versione sincrona **oppure** la versione asincrona di un'interfaccia di filtro, **non** entrambe.</span><span class="sxs-lookup"><span data-stu-id="42f92-159">Implement **either** the synchronous or the async version of a filter interface, **not** both.</span></span> <span data-ttu-id="42f92-160">Il runtime controlla per prima cosa se il filtro implementa l'interfaccia asincrona e, in tal caso, la chiama.</span><span class="sxs-lookup"><span data-stu-id="42f92-160">The runtime checks first to see if the filter implements the async interface, and if so, it calls that.</span></span> <span data-ttu-id="42f92-161">In caso contrario, chiama i metodi dell'interfaccia sincrona.</span><span class="sxs-lookup"><span data-stu-id="42f92-161">If not, it calls the synchronous interface's method(s).</span></span> <span data-ttu-id="42f92-162">Se in una classe sono implementate entrambe le interfacce, sincrona e asincrona, viene chiamato solo il metodo asincrono.</span><span class="sxs-lookup"><span data-stu-id="42f92-162">If both asynchronous and synchronous interfaces are implemented in one class, only the async method is called.</span></span> <span data-ttu-id="42f92-163">Quando si usano classi astratte come <xref:Microsoft.AspNetCore.Mvc.Filters.ActionFilterAttribute> , eseguire l'override solo dei metodi sincroni o dei metodi asincroni per ogni tipo di filtro.</span><span class="sxs-lookup"><span data-stu-id="42f92-163">When using abstract classes like <xref:Microsoft.AspNetCore.Mvc.Filters.ActionFilterAttribute>, override only the synchronous methods or the asynchronous methods for each filter type.</span></span>

### <a name="built-in-filter-attributes"></a><span data-ttu-id="42f92-164">Attributi filtro predefiniti</span><span class="sxs-lookup"><span data-stu-id="42f92-164">Built-in filter attributes</span></span>

<span data-ttu-id="42f92-165">ASP.NET Core include filtri predefiniti basati su attributi che è possibile impostare come sottoclasse e personalizzare.</span><span class="sxs-lookup"><span data-stu-id="42f92-165">ASP.NET Core includes built-in attribute-based filters that can be subclassed and customized.</span></span> <span data-ttu-id="42f92-166">Il filtro dei risultati seguente, ad esempio, aggiunge un'intestazione alla risposta:</span><span class="sxs-lookup"><span data-stu-id="42f92-166">For example, the following result filter adds a header to the response:</span></span>

<a name="add-header-attribute"></a>

[!code-csharp[](./filters/3.1sample/FiltersSample/Filters/AddHeaderAttribute.cs?name=snippet)]

<span data-ttu-id="42f92-167">Gli attributi consentono ai filtri di accettare argomenti, come illustrato nell'esempio precedente.</span><span class="sxs-lookup"><span data-stu-id="42f92-167">Attributes allow filters to accept arguments, as shown in the preceding example.</span></span> <span data-ttu-id="42f92-168">Applicare `AddHeaderAttribute` a un controller o a un metodo di azione e specificare il nome e il valore dell'intestazione HTTP:</span><span class="sxs-lookup"><span data-stu-id="42f92-168">Apply the `AddHeaderAttribute` to a controller or action method and specify the name and value of the HTTP header:</span></span>

[!code-csharp[](./filters/3.1sample/FiltersSample/Controllers/SampleController.cs?name=snippet_AddHeader&highlight=1)]

<span data-ttu-id="42f92-169">Usare uno strumento come gli [strumenti di sviluppo del browser](https://developer.mozilla.org/docs/Learn/Common_questions/What_are_browser_developer_tools) per esaminare le intestazioni.</span><span class="sxs-lookup"><span data-stu-id="42f92-169">Use a tool such as the [browser developer tools](https://developer.mozilla.org/docs/Learn/Common_questions/What_are_browser_developer_tools) to examine the headers.</span></span> <span data-ttu-id="42f92-170">In **intestazioni risposta** `author: Rick Anderson` viene visualizzato.</span><span class="sxs-lookup"><span data-stu-id="42f92-170">Under **Response Headers**, `author: Rick Anderson` is displayed.</span></span>

<span data-ttu-id="42f92-171">Il codice seguente implementa un oggetto `ActionFilterAttribute` che:</span><span class="sxs-lookup"><span data-stu-id="42f92-171">The following code implements an `ActionFilterAttribute` that:</span></span>

* <span data-ttu-id="42f92-172">Legge il titolo e il nome dal sistema di configurazione.</span><span class="sxs-lookup"><span data-stu-id="42f92-172">Reads the title and name from the configuration system.</span></span> <span data-ttu-id="42f92-173">A differenza dell'esempio precedente, il codice seguente non richiede l'aggiunta di parametri di filtro al codice.</span><span class="sxs-lookup"><span data-stu-id="42f92-173">Unlike the previous sample, the following code doesn't require filter parameters to be added to the code.</span></span>
* <span data-ttu-id="42f92-174">Aggiunge il titolo e il nome all'intestazione della risposta.</span><span class="sxs-lookup"><span data-stu-id="42f92-174">Adds the title and name to the response header.</span></span>

[!code-csharp[](./filters/3.1sample/FiltersSample/Filters/MyActionFilterAttribute.cs?name=snippet)]

<span data-ttu-id="42f92-175">Le opzioni di configurazione vengono fornite dal [sistema di configurazione](xref:fundamentals/configuration/index) usando il [modello di opzioni](xref:fundamentals/configuration/options).</span><span class="sxs-lookup"><span data-stu-id="42f92-175">The configuration options are provided from the [configuration system](xref:fundamentals/configuration/index) using the [options pattern](xref:fundamentals/configuration/options).</span></span> <span data-ttu-id="42f92-176">Ad esempio, dal *appsettings.json* file:</span><span class="sxs-lookup"><span data-stu-id="42f92-176">For example, from the *appsettings.json* file:</span></span>

[!code-json[](filters/3.1sample/FiltersSample/appsettings.json)]

<span data-ttu-id="42f92-177">Nel `StartUp.ConfigureServices` eseguire le operazioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="42f92-177">In the `StartUp.ConfigureServices`:</span></span>

* <span data-ttu-id="42f92-178">La `PositionOptions` classe viene aggiunta al contenitore del servizio con l' `"Position"` area di configurazione.</span><span class="sxs-lookup"><span data-stu-id="42f92-178">The `PositionOptions` class is added to the service container with the `"Position"` configuration area.</span></span>
* <span data-ttu-id="42f92-179">`MyActionFilterAttribute`Viene aggiunto al contenitore del servizio.</span><span class="sxs-lookup"><span data-stu-id="42f92-179">The `MyActionFilterAttribute` is added to the service container.</span></span>

[!code-csharp[](./filters/3.1sample/FiltersSample/StartupAF.cs?name=snippet)]

<span data-ttu-id="42f92-180">Il codice seguente illustra la `PositionOptions` classe:</span><span class="sxs-lookup"><span data-stu-id="42f92-180">The following code shows the `PositionOptions` class:</span></span>

[!code-csharp[](filters/3.1sample/FiltersSample/Helper/PositionOptions.cs?name=snippet)]

<span data-ttu-id="42f92-181">Il codice seguente applica al `MyActionFilterAttribute` `Index2` Metodo:</span><span class="sxs-lookup"><span data-stu-id="42f92-181">The following code applies the `MyActionFilterAttribute` to the `Index2` method:</span></span>

[!code-csharp[](./filters/3.1sample/FiltersSample/Controllers/SampleController.cs?name=snippet2&highlight=9)]

<span data-ttu-id="42f92-182">In **intestazioni di risposta**, `author: Rick Anderson` e `Editor: Joe Smith` viene visualizzato quando `Sample/Index2` viene chiamato l'endpoint.</span><span class="sxs-lookup"><span data-stu-id="42f92-182">Under **Response Headers**, `author: Rick Anderson`, and `Editor: Joe Smith` is displayed when the `Sample/Index2` endpoint is called.</span></span>

<span data-ttu-id="42f92-183">Il codice seguente applica `MyActionFilterAttribute` e alla `AddHeaderAttribute` Razor pagina:</span><span class="sxs-lookup"><span data-stu-id="42f92-183">The following code applies the `MyActionFilterAttribute` and the `AddHeaderAttribute` to the Razor Page:</span></span>

[!code-csharp[](filters/3.1sample/FiltersSample/Pages/Movies/Index.cshtml.cs?name=snippet)]

<span data-ttu-id="42f92-184">Non è possibile applicare i filtri ai Razor metodi del gestore di pagina.</span><span class="sxs-lookup"><span data-stu-id="42f92-184">Filters cannot be applied to Razor Page handler methods.</span></span> <span data-ttu-id="42f92-185">Possono essere applicati al Razor modello di pagina o a livello globale.</span><span class="sxs-lookup"><span data-stu-id="42f92-185">They can be applied either to the Razor Page model or globally.</span></span>

<span data-ttu-id="42f92-186">Molte delle interfacce del filtro presentano attributi corrispondenti che possono essere usati come classi di base per implementazioni personalizzate.</span><span class="sxs-lookup"><span data-stu-id="42f92-186">Several of the filter interfaces have corresponding attributes that can be used as base classes for custom implementations.</span></span>

<span data-ttu-id="42f92-187">Attributi dei filtri:</span><span class="sxs-lookup"><span data-stu-id="42f92-187">Filter attributes:</span></span>

* <xref:Microsoft.AspNetCore.Mvc.Filters.ActionFilterAttribute>
* <xref:Microsoft.AspNetCore.Mvc.Filters.ExceptionFilterAttribute>
* <xref:Microsoft.AspNetCore.Mvc.Filters.ResultFilterAttribute>
* <xref:Microsoft.AspNetCore.Mvc.FormatFilterAttribute>
* <xref:Microsoft.AspNetCore.Mvc.ServiceFilterAttribute>
* <xref:Microsoft.AspNetCore.Mvc.TypeFilterAttribute>

## <a name="filter-scopes-and-order-of-execution"></a><span data-ttu-id="42f92-188">Ambiti dei filtri e ordine di esecuzione</span><span class="sxs-lookup"><span data-stu-id="42f92-188">Filter scopes and order of execution</span></span>

<span data-ttu-id="42f92-189">È possibile aggiungere un filtro alla pipeline in uno dei tre *ambiti*:</span><span class="sxs-lookup"><span data-stu-id="42f92-189">A filter can be added to the pipeline at one of three *scopes*:</span></span>

* <span data-ttu-id="42f92-190">Uso di un attributo in un'azione del controller.</span><span class="sxs-lookup"><span data-stu-id="42f92-190">Using an attribute on a controller action.</span></span> <span data-ttu-id="42f92-191">Gli attributi di filtro non possono essere applicati ai Razor metodi del gestore di pagine.</span><span class="sxs-lookup"><span data-stu-id="42f92-191">Filter attributes cannot be applied to Razor Pages handler methods.</span></span>
* <span data-ttu-id="42f92-192">Uso di un attributo in un controller o in una Razor pagina.</span><span class="sxs-lookup"><span data-stu-id="42f92-192">Using an attribute on a controller or Razor Page.</span></span>
* <span data-ttu-id="42f92-193">A livello globale per tutti i controller, le azioni e le Razor pagine, come illustrato nel codice seguente:</span><span class="sxs-lookup"><span data-stu-id="42f92-193">Globally for all controllers, actions, and Razor Pages as shown in the following code:</span></span>

[!code-csharp[](./filters/3.1sample/FiltersSample/StartupOrder.cs?name=snippet)]

### <a name="default-order-of-execution"></a><span data-ttu-id="42f92-194">Ordine di esecuzione predefinito</span><span class="sxs-lookup"><span data-stu-id="42f92-194">Default order of execution</span></span>

<span data-ttu-id="42f92-195">Quando sono presenti più filtri per una particolare fase della pipeline, l'ambito determina l'ordine di esecuzione predefinito dei filtri.</span><span class="sxs-lookup"><span data-stu-id="42f92-195">When there are multiple filters for a particular stage of the pipeline, scope determines the default order of filter execution.</span></span>  <span data-ttu-id="42f92-196">I filtri globali racchiudono i filtri di classe, che a loro volta racchiudono i filtri dei metodi.</span><span class="sxs-lookup"><span data-stu-id="42f92-196">Global filters surround class filters, which in turn surround method filters.</span></span>

<span data-ttu-id="42f92-197">Come risultato dell'annidamento dei filtri, il codice *after* dei filtri viene eseguito in ordine inverso rispetto al codice *before*.</span><span class="sxs-lookup"><span data-stu-id="42f92-197">As a result of filter nesting, the *after* code of filters runs in the reverse order of the *before* code.</span></span> <span data-ttu-id="42f92-198">Sequenza di filtro:</span><span class="sxs-lookup"><span data-stu-id="42f92-198">The filter sequence:</span></span>

* <span data-ttu-id="42f92-199">Codice *before* dei filtri globali.</span><span class="sxs-lookup"><span data-stu-id="42f92-199">The *before* code of global filters.</span></span>
  * <span data-ttu-id="42f92-200">Codice *prima* dei filtri di pagina e controller Razor .</span><span class="sxs-lookup"><span data-stu-id="42f92-200">The *before* code of controller and Razor Page filters.</span></span>
    * <span data-ttu-id="42f92-201">Codice *before* dei filtri del metodo di azione.</span><span class="sxs-lookup"><span data-stu-id="42f92-201">The *before* code of action method filters.</span></span>
    * <span data-ttu-id="42f92-202">Codice *after* dei filtri del metodo di azione.</span><span class="sxs-lookup"><span data-stu-id="42f92-202">The *after* code of action method filters.</span></span>
  * <span data-ttu-id="42f92-203">Codice *dopo* i filtri del controller e della Razor pagina.</span><span class="sxs-lookup"><span data-stu-id="42f92-203">The *after* code of controller and Razor Page filters.</span></span>
* <span data-ttu-id="42f92-204">Codice *after* dei filtri globali.</span><span class="sxs-lookup"><span data-stu-id="42f92-204">The *after* code of global filters.</span></span>
  
<span data-ttu-id="42f92-205">L'esempio seguente illustra l'ordine in cui i metodi dei filtri vengono chiamati per i filtri di azione sincroni.</span><span class="sxs-lookup"><span data-stu-id="42f92-205">The following example that illustrates the order in which filter methods are called for synchronous action filters.</span></span>

| <span data-ttu-id="42f92-206">Sequenza</span><span class="sxs-lookup"><span data-stu-id="42f92-206">Sequence</span></span> | <span data-ttu-id="42f92-207">Ambito del filtro</span><span class="sxs-lookup"><span data-stu-id="42f92-207">Filter scope</span></span> | <span data-ttu-id="42f92-208">Metodo del filtro</span><span class="sxs-lookup"><span data-stu-id="42f92-208">Filter method</span></span> |
|:--------:|:------------:|:-------------:|
| <span data-ttu-id="42f92-209">1</span><span class="sxs-lookup"><span data-stu-id="42f92-209">1</span></span> | <span data-ttu-id="42f92-210">Globale</span><span class="sxs-lookup"><span data-stu-id="42f92-210">Global</span></span> | `OnActionExecuting` |
| <span data-ttu-id="42f92-211">2</span><span class="sxs-lookup"><span data-stu-id="42f92-211">2</span></span> | <span data-ttu-id="42f92-212">Controller o Razor pagina</span><span class="sxs-lookup"><span data-stu-id="42f92-212">Controller or Razor Page</span></span>| `OnActionExecuting` |
| <span data-ttu-id="42f92-213">3</span><span class="sxs-lookup"><span data-stu-id="42f92-213">3</span></span> | <span data-ttu-id="42f92-214">Metodo</span><span class="sxs-lookup"><span data-stu-id="42f92-214">Method</span></span> | `OnActionExecuting` |
| <span data-ttu-id="42f92-215">4</span><span class="sxs-lookup"><span data-stu-id="42f92-215">4</span></span> | <span data-ttu-id="42f92-216">Metodo</span><span class="sxs-lookup"><span data-stu-id="42f92-216">Method</span></span> | `OnActionExecuted` |
| <span data-ttu-id="42f92-217">5</span><span class="sxs-lookup"><span data-stu-id="42f92-217">5</span></span> | <span data-ttu-id="42f92-218">Controller o Razor pagina</span><span class="sxs-lookup"><span data-stu-id="42f92-218">Controller or Razor Page</span></span> | `OnActionExecuted` |
| <span data-ttu-id="42f92-219">6</span><span class="sxs-lookup"><span data-stu-id="42f92-219">6</span></span> | <span data-ttu-id="42f92-220">Globale</span><span class="sxs-lookup"><span data-stu-id="42f92-220">Global</span></span> | `OnActionExecuted` |

### <a name="controller-level-filters"></a><span data-ttu-id="42f92-221">Filtri a livello di controller</span><span class="sxs-lookup"><span data-stu-id="42f92-221">Controller level filters</span></span>

<span data-ttu-id="42f92-222">Ogni controller che eredita dalla classe di base <xref:Microsoft.AspNetCore.Mvc.Controller> include i metodi [Controller.OnActionExecuting](xref:Microsoft.AspNetCore.Mvc.Controller.OnActionExecuting*), [Controller.OnActionExecutionAsync](xref:Microsoft.AspNetCore.Mvc.Controller.OnActionExecutionAsync*) e [Controller.OnActionExecuted](xref:Microsoft.AspNetCore.Mvc.Controller.OnActionExecuted*)
`OnActionExecuted`.</span><span class="sxs-lookup"><span data-stu-id="42f92-222">Every controller that inherits from the <xref:Microsoft.AspNetCore.Mvc.Controller> base class includes [Controller.OnActionExecuting](xref:Microsoft.AspNetCore.Mvc.Controller.OnActionExecuting*),  [Controller.OnActionExecutionAsync](xref:Microsoft.AspNetCore.Mvc.Controller.OnActionExecutionAsync*), and [Controller.OnActionExecuted](xref:Microsoft.AspNetCore.Mvc.Controller.OnActionExecuted*)
`OnActionExecuted` methods.</span></span> <span data-ttu-id="42f92-223">Questi metodi:</span><span class="sxs-lookup"><span data-stu-id="42f92-223">These methods:</span></span>

* <span data-ttu-id="42f92-224">Eseguono il wrapping dei filtri che vengono eseguiti per una determinata azione.</span><span class="sxs-lookup"><span data-stu-id="42f92-224">Wrap the filters that run for a given action.</span></span>
* <span data-ttu-id="42f92-225">La chiamata di `OnActionExecuting` avviene prima di qualsiasi filtro dell'azione.</span><span class="sxs-lookup"><span data-stu-id="42f92-225">`OnActionExecuting` is called before any of the action's filters.</span></span>
* <span data-ttu-id="42f92-226">La chiamata di `OnActionExecuted` avviene dopo tutti i filtri dell'azione.</span><span class="sxs-lookup"><span data-stu-id="42f92-226">`OnActionExecuted` is called after all of the action filters.</span></span>
* <span data-ttu-id="42f92-227">La chiamata di `OnActionExecutionAsync` avviene prima di qualsiasi filtro dell'azione.</span><span class="sxs-lookup"><span data-stu-id="42f92-227">`OnActionExecutionAsync` is called before any of the action's filters.</span></span> <span data-ttu-id="42f92-228">Il codice del filtro dopo `next` viene eseguito dopo il metodo di azione.</span><span class="sxs-lookup"><span data-stu-id="42f92-228">Code in the filter after `next` runs after the action method.</span></span>

<span data-ttu-id="42f92-229">Ad esempio, l'applicazione di `MySampleActionFilter` nell'esempio scaricato avviene a livello globale all'avvio.</span><span class="sxs-lookup"><span data-stu-id="42f92-229">For example, in the download sample, `MySampleActionFilter` is applied globally in startup.</span></span>

<span data-ttu-id="42f92-230">`TestController`:</span><span class="sxs-lookup"><span data-stu-id="42f92-230">The `TestController`:</span></span>

* <span data-ttu-id="42f92-231">Applica `SampleActionFilterAttribute` ( `[SampleActionFilter]` ) all' `FilterTest2` azione.</span><span class="sxs-lookup"><span data-stu-id="42f92-231">Applies the `SampleActionFilterAttribute` (`[SampleActionFilter]`) to the `FilterTest2` action.</span></span>
* <span data-ttu-id="42f92-232">Esegue l'override di `OnActionExecuting` e `OnActionExecuted`.</span><span class="sxs-lookup"><span data-stu-id="42f92-232">Overrides `OnActionExecuting` and `OnActionExecuted`.</span></span>

[!code-csharp[](./filters/3.1sample/FiltersSample/Controllers/TestController.cs?name=snippet)]

[!INCLUDE[](~/includes/MyDisplayRouteInfo.md)]

<!-- test via  webBuilder.UseStartup<Startup>(); -->

<span data-ttu-id="42f92-233">Passando a `https://localhost:5001/Test/FilterTest2`, viene eseguito il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="42f92-233">Navigating to `https://localhost:5001/Test/FilterTest2` runs the following code:</span></span>

* `TestController.OnActionExecuting`
  * `MySampleActionFilter.OnActionExecuting`
    * `SampleActionFilterAttribute.OnActionExecuting`
      * `TestController.FilterTest2`
    * `SampleActionFilterAttribute.OnActionExecuted`
  * `MySampleActionFilter.OnActionExecuted`
* `TestController.OnActionExecuted`

<span data-ttu-id="42f92-234">Filtri a livello di controller impostare la proprietà [Order](https://github.com/dotnet/AspNetCore/blob/master/src/Mvc/Mvc.Core/src/Filters/ControllerActionFilter.cs#L15-L17) su `int.MinValue` .</span><span class="sxs-lookup"><span data-stu-id="42f92-234">Controller level filters set the [Order](https://github.com/dotnet/AspNetCore/blob/master/src/Mvc/Mvc.Core/src/Filters/ControllerActionFilter.cs#L15-L17) property to `int.MinValue`.</span></span> <span data-ttu-id="42f92-235">I filtri a livello di controller **non** possono essere impostati per l'esecuzione dopo i filtri applicati ai metodi.</span><span class="sxs-lookup"><span data-stu-id="42f92-235">Controller level filters can **not** be set to run after filters applied to methods.</span></span> <span data-ttu-id="42f92-236">L'ordine è illustrato nella sezione successiva.</span><span class="sxs-lookup"><span data-stu-id="42f92-236">Order is explained in the next section.</span></span>

<span data-ttu-id="42f92-237">Per le Razor pagine, vedere [implementare Razor filtri di pagina eseguendo l'override dei metodi di filtro](xref:razor-pages/filter#implement-razor-page-filters-by-overriding-filter-methods).</span><span class="sxs-lookup"><span data-stu-id="42f92-237">For Razor Pages, see [Implement Razor Page filters by overriding filter methods](xref:razor-pages/filter#implement-razor-page-filters-by-overriding-filter-methods).</span></span>

### <a name="overriding-the-default-order"></a><span data-ttu-id="42f92-238">Override dell'ordine predefinito</span><span class="sxs-lookup"><span data-stu-id="42f92-238">Overriding the default order</span></span>

<span data-ttu-id="42f92-239">È possibile eseguire l'override della sequenza di esecuzione predefinita implementando <xref:Microsoft.AspNetCore.Mvc.Filters.IOrderedFilter>.</span><span class="sxs-lookup"><span data-stu-id="42f92-239">The default sequence of execution can be overridden by implementing <xref:Microsoft.AspNetCore.Mvc.Filters.IOrderedFilter>.</span></span> <span data-ttu-id="42f92-240">`IOrderedFilter` espone la proprietà <xref:Microsoft.AspNetCore.Mvc.Filters.IOrderedFilter.Order>, che ha la precedenza sull'ambito per determinare l'ordine di esecuzione.</span><span class="sxs-lookup"><span data-stu-id="42f92-240">`IOrderedFilter` exposes the <xref:Microsoft.AspNetCore.Mvc.Filters.IOrderedFilter.Order> property that takes precedence over scope to determine the order of execution.</span></span> <span data-ttu-id="42f92-241">Un filtro con un valore di `Order` inferiore:</span><span class="sxs-lookup"><span data-stu-id="42f92-241">A filter with a lower `Order` value:</span></span>

* <span data-ttu-id="42f92-242">Esegue il codice *before* prima di quello di un filtro con un valore di `Order` più alto.</span><span class="sxs-lookup"><span data-stu-id="42f92-242">Runs the *before* code before that of a filter with a higher value of `Order`.</span></span>
* <span data-ttu-id="42f92-243">Esegue il codice *after* dopo quello di un filtro con un valore di `Order` più alto.</span><span class="sxs-lookup"><span data-stu-id="42f92-243">Runs the *after* code after that of a filter with a higher `Order` value.</span></span>

<span data-ttu-id="42f92-244">La `Order` proprietà viene impostata con un parametro del costruttore:</span><span class="sxs-lookup"><span data-stu-id="42f92-244">The `Order` property is set with a constructor parameter:</span></span>

[!code-csharp[](./filters/3.1sample/FiltersSample/Controllers/Test3Controller.cs?name=snippet)]

<span data-ttu-id="42f92-245">Considerare i due filtri azione nel controller seguente:</span><span class="sxs-lookup"><span data-stu-id="42f92-245">Consider the two action filters in the following controller:</span></span>

[!code-csharp[](./filters/3.1sample/FiltersSample/Controllers/Test2Controller.cs?name=snippet)]

<span data-ttu-id="42f92-246">In è stato aggiunto un filtro globale `StartUp.ConfigureServices` :</span><span class="sxs-lookup"><span data-stu-id="42f92-246">A global filter is added in `StartUp.ConfigureServices`:</span></span>

[!code-csharp[](./filters/3.1sample/FiltersSample/StartupOrder.cs?name=snippet)]

<span data-ttu-id="42f92-247">I 3 filtri vengono eseguiti nell'ordine seguente:</span><span class="sxs-lookup"><span data-stu-id="42f92-247">The 3 filters run in the following order:</span></span>

* `Test2Controller.OnActionExecuting`
  * `MySampleActionFilter.OnActionExecuting`
    * `MyAction2FilterAttribute.OnActionExecuting`
      * `Test2Controller.FilterTest2`
    * `MyAction2FilterAttribute.OnResultExecuting`
  * `MySampleActionFilter.OnActionExecuted`
* `Test2Controller.OnActionExecuted`

<span data-ttu-id="42f92-248">La proprietà `Order` ha la precedenza sull'ambito nel determinare l'ordine di esecuzione dei filtri.</span><span class="sxs-lookup"><span data-stu-id="42f92-248">The `Order` property overrides scope when determining the order in which filters run.</span></span> <span data-ttu-id="42f92-249">I filtri vengono ordinati prima in base all'ordine, poi viene usato l'ambito per interrompere i collegamenti.</span><span class="sxs-lookup"><span data-stu-id="42f92-249">Filters are sorted first by order, then scope is used to break ties.</span></span> <span data-ttu-id="42f92-250">Tutti i filtri predefiniti implementano `IOrderedFilter` e impostano il valore predefinito di `Order` su 0.</span><span class="sxs-lookup"><span data-stu-id="42f92-250">All of the built-in filters implement `IOrderedFilter` and set the default `Order` value to 0.</span></span> <span data-ttu-id="42f92-251">Come indicato in precedenza, i filtri a livello di controller impostano la proprietà [Order](https://github.com/dotnet/AspNetCore/blob/master/src/Mvc/Mvc.Core/src/Filters/ControllerActionFilter.cs#L15-L17) su `int.MinValue` per i filtri incorporati, l'ambito determina l'ordine a meno che non `Order` sia impostato su un valore diverso da zero.</span><span class="sxs-lookup"><span data-stu-id="42f92-251">As mentioned previously, controller level filters set the [Order](https://github.com/dotnet/AspNetCore/blob/master/src/Mvc/Mvc.Core/src/Filters/ControllerActionFilter.cs#L15-L17) property to `int.MinValue` For built-in filters, scope determines order unless `Order` is set to a non-zero value.</span></span>

<span data-ttu-id="42f92-252">Nel codice precedente, `MySampleActionFilter` ha ambito globale, quindi viene eseguito prima di `MyAction2FilterAttribute` , che ha ambito del controller.</span><span class="sxs-lookup"><span data-stu-id="42f92-252">In the preceding code, `MySampleActionFilter` has global scope so it runs before `MyAction2FilterAttribute`, which has controller scope.</span></span> <span data-ttu-id="42f92-253">Per `MyAction2FilterAttribute` eseguire prima l'esecuzione, impostare l'ordine su `int.MinValue` :</span><span class="sxs-lookup"><span data-stu-id="42f92-253">To make `MyAction2FilterAttribute` run first, set the order to `int.MinValue`:</span></span>

[!code-csharp[](./filters/3.1sample/FiltersSample/Controllers/Test2Controller.cs?name=snippet2)]

<span data-ttu-id="42f92-254">Per eseguire prima l'esecuzione del filtro globale `MySampleActionFilter` , impostare `Order` su `int.MinValue` :</span><span class="sxs-lookup"><span data-stu-id="42f92-254">To make the global filter `MySampleActionFilter` run first, set `Order` to `int.MinValue`:</span></span>

[!code-csharp[](./filters/3.1sample/FiltersSample/StartupOrder2.cs?name=snippet&highlight=6)]

## <a name="cancellation-and-short-circuiting"></a><span data-ttu-id="42f92-255">Annullamento e corto circuito</span><span class="sxs-lookup"><span data-stu-id="42f92-255">Cancellation and short-circuiting</span></span>

<span data-ttu-id="42f92-256">È possibile causare il corto circuito della pipeline di filtro impostando la proprietà <xref:Microsoft.AspNetCore.Mvc.Filters.ResourceExecutingContext.Result> sul parametro <xref:Microsoft.AspNetCore.Mvc.Filters.ResourceExecutingContext> fornito al metodo di filtro.</span><span class="sxs-lookup"><span data-stu-id="42f92-256">The filter pipeline can be short-circuited by setting the <xref:Microsoft.AspNetCore.Mvc.Filters.ResourceExecutingContext.Result> property on the <xref:Microsoft.AspNetCore.Mvc.Filters.ResourceExecutingContext> parameter provided to the filter method.</span></span> <span data-ttu-id="42f92-257">Ad esempio, il filtro di risorse seguente impedisce l'esecuzione del resto della pipeline:</span><span class="sxs-lookup"><span data-stu-id="42f92-257">For instance, the following Resource filter prevents the rest of the pipeline from executing:</span></span>

<a name="short-circuiting-resource-filter"></a>

[!code-csharp[](./filters/3.1sample/FiltersSample/Filters/ShortCircuitingResourceFilterAttribute.cs?name=snippet)]

<span data-ttu-id="42f92-258">Nel codice seguente sia il filtro `ShortCircuitingResourceFilter` che il filtro `AddHeader` hanno come destinazione il metodo di azione `SomeResource`.</span><span class="sxs-lookup"><span data-stu-id="42f92-258">In the following code, both the `ShortCircuitingResourceFilter` and the `AddHeader` filter target the `SomeResource` action method.</span></span> <span data-ttu-id="42f92-259">`ShortCircuitingResourceFilter`:</span><span class="sxs-lookup"><span data-stu-id="42f92-259">The `ShortCircuitingResourceFilter`:</span></span>

* <span data-ttu-id="42f92-260">Viene eseguito per primo perché è un filtro risorsa e `AddHeader` è un filtro azione.</span><span class="sxs-lookup"><span data-stu-id="42f92-260">Runs first, because it's a Resource Filter and `AddHeader` is an Action Filter.</span></span>
* <span data-ttu-id="42f92-261">Blocca il resto della pipeline.</span><span class="sxs-lookup"><span data-stu-id="42f92-261">Short-circuits the rest of the pipeline.</span></span>

<span data-ttu-id="42f92-262">Pertanto il filtro `AddHeader` non viene mai eseguito per l'azione `SomeResource`.</span><span class="sxs-lookup"><span data-stu-id="42f92-262">Therefore the `AddHeader` filter never runs for the `SomeResource` action.</span></span> <span data-ttu-id="42f92-263">Questo comportamento sarebbe lo stesso se entrambi i filtri venissero applicati a livello di metodo di azione, a condizione che il filtro `ShortCircuitingResourceFilter` venga eseguito prima.</span><span class="sxs-lookup"><span data-stu-id="42f92-263">This behavior would be the same if both filters were applied at the action method level, provided the `ShortCircuitingResourceFilter` ran first.</span></span> <span data-ttu-id="42f92-264">`ShortCircuitingResourceFilter` viene eseguito per primo per il tipo di filtro o per l'uso esplicito della proprietà `Order`.</span><span class="sxs-lookup"><span data-stu-id="42f92-264">The `ShortCircuitingResourceFilter` runs first because of its filter type, or by explicit use of `Order` property.</span></span>

[!code-csharp[](./filters/3.1sample/FiltersSample/Controllers/SampleController.cs?name=snippet3&highlight=1,15)]

## <a name="dependency-injection"></a><span data-ttu-id="42f92-265">Inserimento di dipendenze</span><span class="sxs-lookup"><span data-stu-id="42f92-265">Dependency injection</span></span>

<span data-ttu-id="42f92-266">È possibile aggiungere filtri per tipo o per istanza.</span><span class="sxs-lookup"><span data-stu-id="42f92-266">Filters can be added by type or by instance.</span></span> <span data-ttu-id="42f92-267">Se viene aggiunta un'istanza, tale istanza viene usata per ogni richiesta.</span><span class="sxs-lookup"><span data-stu-id="42f92-267">If an instance is added, that instance is used for every request.</span></span> <span data-ttu-id="42f92-268">Se viene aggiunto un tipo, il filtro è attivato dal tipo.</span><span class="sxs-lookup"><span data-stu-id="42f92-268">If a type is added, it's type-activated.</span></span> <span data-ttu-id="42f92-269">Un filtro attivato dal tipo comporta:</span><span class="sxs-lookup"><span data-stu-id="42f92-269">A type-activated filter means:</span></span>

* <span data-ttu-id="42f92-270">La creazione di un'istanza per ogni richiesta.</span><span class="sxs-lookup"><span data-stu-id="42f92-270">An instance is created for each request.</span></span>
* <span data-ttu-id="42f92-271">Il popolamento di tutte le dipendenze del costruttore tramite [inserimento delle dipendenze](xref:fundamentals/dependency-injection).</span><span class="sxs-lookup"><span data-stu-id="42f92-271">Any constructor dependencies are populated by [dependency injection](xref:fundamentals/dependency-injection) (DI).</span></span>

<span data-ttu-id="42f92-272">I filtri implementati come attributi e aggiunti direttamente alle classi controller o ai metodi di azione non possono avere dipendenze costruttore specificate dall'[inserimento di dipendenze](xref:fundamentals/dependency-injection).</span><span class="sxs-lookup"><span data-stu-id="42f92-272">Filters that are implemented as attributes and added directly to controller classes or action methods cannot have constructor dependencies provided by [dependency injection](xref:fundamentals/dependency-injection) (DI).</span></span> <span data-ttu-id="42f92-273">Le dipendenze del costruttore non possono essere fornite tramite inserimento delle dipendenze in quanto:</span><span class="sxs-lookup"><span data-stu-id="42f92-273">Constructor dependencies cannot be provided by DI because:</span></span>

* <span data-ttu-id="42f92-274">I parametri del costruttore devono essere forniti agli attributi nella posizione in cui vengono applicati.</span><span class="sxs-lookup"><span data-stu-id="42f92-274">Attributes must have their constructor parameters supplied where they're applied.</span></span> 
* <span data-ttu-id="42f92-275">Questa è una limitazione del funzionamento degli attributi.</span><span class="sxs-lookup"><span data-stu-id="42f92-275">This is a limitation of how attributes work.</span></span>

<span data-ttu-id="42f92-276">I filtri seguenti supportano le dipendenze del costruttore fornite dall'inserimento delle dipendenze:</span><span class="sxs-lookup"><span data-stu-id="42f92-276">The following filters support constructor dependencies provided from DI:</span></span>

* <xref:Microsoft.AspNetCore.Mvc.ServiceFilterAttribute>
* <xref:Microsoft.AspNetCore.Mvc.TypeFilterAttribute>
* <span data-ttu-id="42f92-277"><xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory> implementato nell'attributo.</span><span class="sxs-lookup"><span data-stu-id="42f92-277"><xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory> implemented on the attribute.</span></span>

<span data-ttu-id="42f92-278">I filtri precedenti possono essere applicati a un controller o a un metodo di azione:</span><span class="sxs-lookup"><span data-stu-id="42f92-278">The preceding filters can be applied to a controller or action method:</span></span>

<span data-ttu-id="42f92-279">I logger sono disponibili tramite l'inserimento delle dipendenze.</span><span class="sxs-lookup"><span data-stu-id="42f92-279">Loggers are available from DI.</span></span> <span data-ttu-id="42f92-280">Evitare tuttavia di creare e usare filtri esclusivamente per scopi di registrazione.</span><span class="sxs-lookup"><span data-stu-id="42f92-280">However, avoid creating and using filters purely for logging purposes.</span></span> <span data-ttu-id="42f92-281">La funzionalità di [registrazione del framework predefinita](xref:fundamentals/logging/index) in genere fornisce il necessario per la registrazione.</span><span class="sxs-lookup"><span data-stu-id="42f92-281">The [built-in framework logging](xref:fundamentals/logging/index) typically provides what's needed for logging.</span></span> <span data-ttu-id="42f92-282">La registrazione aggiunta ai filtri:</span><span class="sxs-lookup"><span data-stu-id="42f92-282">Logging added to filters:</span></span>

* <span data-ttu-id="42f92-283">Deve concentrarsi su problemi del dominio di business o comportamenti specifici del filtro.</span><span class="sxs-lookup"><span data-stu-id="42f92-283">Should focus on business domain concerns or behavior specific to the filter.</span></span>
* <span data-ttu-id="42f92-284">**Non** deve registrare azioni o altri eventi del framework.</span><span class="sxs-lookup"><span data-stu-id="42f92-284">Should **not** log actions or other framework events.</span></span> <span data-ttu-id="42f92-285">Le azioni di log dei filtri incorporati e gli eventi del Framework.</span><span class="sxs-lookup"><span data-stu-id="42f92-285">The built-in filters log actions and framework events.</span></span>

### <a name="servicefilterattribute"></a><span data-ttu-id="42f92-286">ServiceFilterAttribute</span><span class="sxs-lookup"><span data-stu-id="42f92-286">ServiceFilterAttribute</span></span>

<span data-ttu-id="42f92-287">I tipi di implementazione del filtro di servizi sono registrati in `ConfigureServices`.</span><span class="sxs-lookup"><span data-stu-id="42f92-287">Service filter implementation types are registered in `ConfigureServices`.</span></span> <span data-ttu-id="42f92-288"><xref:Microsoft.AspNetCore.Mvc.ServiceFilterAttribute> recupera un'istanza del filtro dall'inserimento di dipendenze.</span><span class="sxs-lookup"><span data-stu-id="42f92-288">A <xref:Microsoft.AspNetCore.Mvc.ServiceFilterAttribute> retrieves an instance of the filter from DI.</span></span>

<span data-ttu-id="42f92-289">Il codice seguente illustra `AddHeaderResultServiceFilter`:</span><span class="sxs-lookup"><span data-stu-id="42f92-289">The following code shows the `AddHeaderResultServiceFilter`:</span></span>

[!code-csharp[](./filters/3.1sample/FiltersSample/Filters/LoggingAddHeaderFilter.cs?name=snippet_ResultFilter)]

<span data-ttu-id="42f92-290">Nel codice seguente l'oggetto `AddHeaderResultServiceFilter` viene aggiunto al contenitore di inserimento delle dipendenze:</span><span class="sxs-lookup"><span data-stu-id="42f92-290">In the following code, `AddHeaderResultServiceFilter` is added to the DI container:</span></span>

[!code-csharp[](./filters/3.1sample/FiltersSample/Startup.cs?name=snippet&highlight=4)]

<span data-ttu-id="42f92-291">Nel codice seguente l'attributo `ServiceFilter` recupera un'istanza del filtro `AddHeaderResultServiceFilter` dall'inserimento delle dipendenze:</span><span class="sxs-lookup"><span data-stu-id="42f92-291">In the following code, the `ServiceFilter` attribute retrieves an instance of the `AddHeaderResultServiceFilter` filter from DI:</span></span>

[!code-csharp[](./filters/3.1sample/FiltersSample/Controllers/HomeController.cs?name=snippet_ServiceFilter&highlight=1)]

<span data-ttu-id="42f92-292">Quando si usa l'impostazione `ServiceFilterAttribute`, [ServiceFilterAttribute.IsReusable](xref:Microsoft.AspNetCore.Mvc.ServiceFilterAttribute.IsReusable):</span><span class="sxs-lookup"><span data-stu-id="42f92-292">When using `ServiceFilterAttribute`, setting [ServiceFilterAttribute.IsReusable](xref:Microsoft.AspNetCore.Mvc.ServiceFilterAttribute.IsReusable):</span></span>

* <span data-ttu-id="42f92-293">Indica che l'istanza del filtro *può* essere riutilizzata all'esterno dell'ambito della richiesta in cui è stata creata.</span><span class="sxs-lookup"><span data-stu-id="42f92-293">Provides a hint that the filter instance *may* be reused outside of the request scope it was created within.</span></span> <span data-ttu-id="42f92-294">Il runtime di ASP.NET Core non garantisce:</span><span class="sxs-lookup"><span data-stu-id="42f92-294">The ASP.NET Core runtime doesn't guarantee:</span></span>

  * <span data-ttu-id="42f92-295">Che venga creata una singola istanza del filtro.</span><span class="sxs-lookup"><span data-stu-id="42f92-295">That a single instance of the filter will be created.</span></span>
  * <span data-ttu-id="42f92-296">Che il filtro non verrà richiesto di nuovo dal contenitore di inserimento delle dipendenze in un momento successivo.</span><span class="sxs-lookup"><span data-stu-id="42f92-296">The filter will not be re-requested from the DI container at some later point.</span></span>

* <span data-ttu-id="42f92-297">Non si deve usare con un filtro che dipende da servizi con una durata diversa da singleton.</span><span class="sxs-lookup"><span data-stu-id="42f92-297">Should not be used with a filter that depends on services with a lifetime other than singleton.</span></span>

 <span data-ttu-id="42f92-298"><xref:Microsoft.AspNetCore.Mvc.ServiceFilterAttribute> implementa <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory>.</span><span class="sxs-lookup"><span data-stu-id="42f92-298"><xref:Microsoft.AspNetCore.Mvc.ServiceFilterAttribute> implements <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory>.</span></span> <span data-ttu-id="42f92-299">`IFilterFactory` espone il metodo <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory.CreateInstance*> per creare un'istanza <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterMetadata>.</span><span class="sxs-lookup"><span data-stu-id="42f92-299">`IFilterFactory` exposes the <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory.CreateInstance*> method for creating an <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterMetadata> instance.</span></span> <span data-ttu-id="42f92-300">`CreateInstance` carica il tipo specificato dall'inserimento delle dipendenze.</span><span class="sxs-lookup"><span data-stu-id="42f92-300">`CreateInstance` loads the specified type from DI.</span></span>

### <a name="typefilterattribute"></a><span data-ttu-id="42f92-301">TypeFilterAttribute</span><span class="sxs-lookup"><span data-stu-id="42f92-301">TypeFilterAttribute</span></span>

<span data-ttu-id="42f92-302"><xref:Microsoft.AspNetCore.Mvc.TypeFilterAttribute> è simile a <xref:Microsoft.AspNetCore.Mvc.ServiceFilterAttribute>, ma il relativo tipo non viene risolto direttamente dal contenitore dell'inserimento di dipendenze.</span><span class="sxs-lookup"><span data-stu-id="42f92-302"><xref:Microsoft.AspNetCore.Mvc.TypeFilterAttribute> is similar to <xref:Microsoft.AspNetCore.Mvc.ServiceFilterAttribute>, but its type isn't resolved directly from the DI container.</span></span> <span data-ttu-id="42f92-303">Viene creata un'istanza del tipo tramite <xref:Microsoft.Extensions.DependencyInjection.ObjectFactory?displayProperty=fullName>.</span><span class="sxs-lookup"><span data-stu-id="42f92-303">It instantiates the type by using <xref:Microsoft.Extensions.DependencyInjection.ObjectFactory?displayProperty=fullName>.</span></span>

<span data-ttu-id="42f92-304">Poiché i tipi `TypeFilterAttribute` non vengono risolti direttamente dal contenitore di inserimento delle dipendenze:</span><span class="sxs-lookup"><span data-stu-id="42f92-304">Because `TypeFilterAttribute` types aren't resolved directly from the DI container:</span></span>

* <span data-ttu-id="42f92-305">Non è necessario che i tipi a cui viene fatto riferimento tramite `TypeFilterAttribute` siano registrati nel contenitore di inserimento delle dipendenze.</span><span class="sxs-lookup"><span data-stu-id="42f92-305">Types that are referenced using the `TypeFilterAttribute` don't need to be registered with the DI container.</span></span>  <span data-ttu-id="42f92-306">Le loro dipendenze vengono soddisfatte dal contenitore di inserimento delle dipendenze.</span><span class="sxs-lookup"><span data-stu-id="42f92-306">They do have their dependencies fulfilled by the DI container.</span></span>
* <span data-ttu-id="42f92-307">In via facoltativa, `TypeFilterAttribute` può anche accettare gli argomenti del costruttore per il tipo.</span><span class="sxs-lookup"><span data-stu-id="42f92-307">`TypeFilterAttribute` can optionally accept constructor arguments for the type.</span></span>

<span data-ttu-id="42f92-308">Quando si usa l'impostazione `TypeFilterAttribute`, [TypeFilterAttribute.IsReusable](xref:Microsoft.AspNetCore.Mvc.TypeFilterAttribute.IsReusable):</span><span class="sxs-lookup"><span data-stu-id="42f92-308">When using `TypeFilterAttribute`, setting [TypeFilterAttribute.IsReusable](xref:Microsoft.AspNetCore.Mvc.TypeFilterAttribute.IsReusable):</span></span>
* <span data-ttu-id="42f92-309">Indica che l'istanza del filtro *può* essere riutilizzata all'esterno dell'ambito della richiesta in cui è stata creata.</span><span class="sxs-lookup"><span data-stu-id="42f92-309">Provides hint that the filter instance *may* be reused outside of the request scope it was created within.</span></span> <span data-ttu-id="42f92-310">Il runtime di ASP.NET Core non fornisce alcuna garanzia che venga creata una singola istanza del filtro.</span><span class="sxs-lookup"><span data-stu-id="42f92-310">The ASP.NET Core runtime provides no guarantees that a single instance of the filter will be created.</span></span>

* <span data-ttu-id="42f92-311">Non si deve usare con un filtro che dipende da servizi con una durata diversa da singleton.</span><span class="sxs-lookup"><span data-stu-id="42f92-311">Should not be used with a filter that depends on services with a lifetime other than singleton.</span></span>

<span data-ttu-id="42f92-312">L'esempio seguente illustra come passare argomenti a un tipo usando `TypeFilterAttribute`:</span><span class="sxs-lookup"><span data-stu-id="42f92-312">The following example shows how to pass arguments to a type using `TypeFilterAttribute`:</span></span>

[!code-csharp[](filters/3.1sample/FiltersSample/Controllers/HomeController.cs?name=snippet_TypeFilter&highlight=1,2)]

<!-- 
https://localhost:5001/home/hi?name=joe
VS debug window shows 
FiltersSample.Filters.LogConstantFilter:Information: Method 'Hi' called
-->

## <a name="authorization-filters"></a><span data-ttu-id="42f92-313">Filtri autorizzazione</span><span class="sxs-lookup"><span data-stu-id="42f92-313">Authorization filters</span></span>

<span data-ttu-id="42f92-314">I filtri di autorizzazione:</span><span class="sxs-lookup"><span data-stu-id="42f92-314">Authorization filters:</span></span>

* <span data-ttu-id="42f92-315">Sono i primi filtri a essere eseguiti nella pipeline di filtro.</span><span class="sxs-lookup"><span data-stu-id="42f92-315">Are the first filters run in the filter pipeline.</span></span>
* <span data-ttu-id="42f92-316">Controllano l'accesso ai metodi di azione.</span><span class="sxs-lookup"><span data-stu-id="42f92-316">Control access to action methods.</span></span>
* <span data-ttu-id="42f92-317">Dispongono di un metodo precedente, ma non di un metodo successivo.</span><span class="sxs-lookup"><span data-stu-id="42f92-317">Have a before method, but no after method.</span></span>

<span data-ttu-id="42f92-318">I filtri di autorizzazione personalizzati richiedono un framework di autorizzazione personalizzato.</span><span class="sxs-lookup"><span data-stu-id="42f92-318">Custom authorization filters require a custom authorization framework.</span></span> <span data-ttu-id="42f92-319">È preferibile configurare criteri di autorizzazione o scrivere criteri di autorizzazione personalizzati piuttosto che scrivere un filtro personalizzato.</span><span class="sxs-lookup"><span data-stu-id="42f92-319">Prefer configuring the authorization policies or writing a custom authorization policy over writing a custom filter.</span></span> <span data-ttu-id="42f92-320">Il filtro di autorizzazione predefinito:</span><span class="sxs-lookup"><span data-stu-id="42f92-320">The built-in authorization filter:</span></span>

* <span data-ttu-id="42f92-321">Chiama il sistema di autorizzazione.</span><span class="sxs-lookup"><span data-stu-id="42f92-321">Calls the authorization system.</span></span>
* <span data-ttu-id="42f92-322">Non autorizza le richieste.</span><span class="sxs-lookup"><span data-stu-id="42f92-322">Does not authorize requests.</span></span>

<span data-ttu-id="42f92-323">**Non** generare eccezioni nei filtri di autorizzazione:</span><span class="sxs-lookup"><span data-stu-id="42f92-323">Do **not** throw exceptions within authorization filters:</span></span>

* <span data-ttu-id="42f92-324">L'eccezione non verrà gestita.</span><span class="sxs-lookup"><span data-stu-id="42f92-324">The exception will not be handled.</span></span>
* <span data-ttu-id="42f92-325">I filtri di eccezione non gestiranno l'eccezione.</span><span class="sxs-lookup"><span data-stu-id="42f92-325">Exception filters will not handle the exception.</span></span>

<span data-ttu-id="42f92-326">È consigliabile emettere una richiesta di verifica in caso di eccezione in un filtro di autorizzazione.</span><span class="sxs-lookup"><span data-stu-id="42f92-326">Consider issuing a challenge when an exception occurs in an authorization filter.</span></span>

<span data-ttu-id="42f92-327">Altre informazioni sull'[autorizzazione](xref:security/authorization/introduction).</span><span class="sxs-lookup"><span data-stu-id="42f92-327">Learn more about [Authorization](xref:security/authorization/introduction).</span></span>

## <a name="resource-filters"></a><span data-ttu-id="42f92-328">Filtri risorse</span><span class="sxs-lookup"><span data-stu-id="42f92-328">Resource filters</span></span>

<span data-ttu-id="42f92-329">I filtri di risorse:</span><span class="sxs-lookup"><span data-stu-id="42f92-329">Resource filters:</span></span>

* <span data-ttu-id="42f92-330">Implementano l'interfaccia <xref:Microsoft.AspNetCore.Mvc.Filters.IResourceFilter> o <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncResourceFilter>.</span><span class="sxs-lookup"><span data-stu-id="42f92-330">Implement either the <xref:Microsoft.AspNetCore.Mvc.Filters.IResourceFilter> or <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncResourceFilter> interface.</span></span>
* <span data-ttu-id="42f92-331">L'esecuzione racchiude la maggior parte della pipeline di filtro.</span><span class="sxs-lookup"><span data-stu-id="42f92-331">Execution wraps most of the filter pipeline.</span></span>
* <span data-ttu-id="42f92-332">Solo i [filtri di autorizzazione](#authorization-filters) vengono eseguiti prima dei filtri di risorse.</span><span class="sxs-lookup"><span data-stu-id="42f92-332">Only [Authorization filters](#authorization-filters) run before resource filters.</span></span>

<span data-ttu-id="42f92-333">I filtri di risorse sono utili per causare il corto circuito della maggior parte della pipeline.</span><span class="sxs-lookup"><span data-stu-id="42f92-333">Resource filters are useful to short-circuit most of the pipeline.</span></span> <span data-ttu-id="42f92-334">Un filtro di memorizzazione nella cache può, ad esempio, evitare il resto della pipeline in caso di riscontro nella cache.</span><span class="sxs-lookup"><span data-stu-id="42f92-334">For example, a caching filter can avoid the rest of the pipeline on a cache hit.</span></span>

<span data-ttu-id="42f92-335">Esempi di filtri di risorse:</span><span class="sxs-lookup"><span data-stu-id="42f92-335">Resource filter examples:</span></span>

* <span data-ttu-id="42f92-336">Il [filtro di risorse di corto circuito](#short-circuiting-resource-filter) illustrato in precedenza.</span><span class="sxs-lookup"><span data-stu-id="42f92-336">[The short-circuiting resource filter](#short-circuiting-resource-filter) shown previously.</span></span>
* <span data-ttu-id="42f92-337">[DisableFormValueModelBindingAttribute](https://github.com/aspnet/Entropy/blob/master/samples/Mvc.FileUpload/Filters/DisableFormValueModelBindingAttribute.cs):</span><span class="sxs-lookup"><span data-stu-id="42f92-337">[DisableFormValueModelBindingAttribute](https://github.com/aspnet/Entropy/blob/master/samples/Mvc.FileUpload/Filters/DisableFormValueModelBindingAttribute.cs):</span></span>

  * <span data-ttu-id="42f92-338">Impedisce all'associazione di modelli di accedere ai dati del modulo.</span><span class="sxs-lookup"><span data-stu-id="42f92-338">Prevents model binding from accessing the form data.</span></span>
  * <span data-ttu-id="42f92-339">Viene usato quando si caricano file di grandi dimensioni per impedire la lettura dei dati del modulo in memoria.</span><span class="sxs-lookup"><span data-stu-id="42f92-339">Used for large file uploads to prevent the form data from being read into memory.</span></span>

## <a name="action-filters"></a><span data-ttu-id="42f92-340">Filtri azione</span><span class="sxs-lookup"><span data-stu-id="42f92-340">Action filters</span></span>

<span data-ttu-id="42f92-341">I filtri azione **non** si applicano alle Razor pagine.</span><span class="sxs-lookup"><span data-stu-id="42f92-341">Action filters do **not** apply to Razor Pages.</span></span> <span data-ttu-id="42f92-342">Razor Pages supporta <xref:Microsoft.AspNetCore.Mvc.Filters.IPageFilter> e <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncPageFilter> .</span><span class="sxs-lookup"><span data-stu-id="42f92-342">Razor Pages supports <xref:Microsoft.AspNetCore.Mvc.Filters.IPageFilter> and <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncPageFilter> .</span></span> <span data-ttu-id="42f92-343">Per ulteriori informazioni, vedere [metodi di filtro per le Razor pagine](xref:razor-pages/filter).</span><span class="sxs-lookup"><span data-stu-id="42f92-343">For more information, see [Filter methods for Razor Pages](xref:razor-pages/filter).</span></span>

<span data-ttu-id="42f92-344">I filtri di azione:</span><span class="sxs-lookup"><span data-stu-id="42f92-344">Action filters:</span></span>

* <span data-ttu-id="42f92-345">Implementano l'interfaccia <xref:Microsoft.AspNetCore.Mvc.Filters.IActionFilter> o <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncActionFilter>.</span><span class="sxs-lookup"><span data-stu-id="42f92-345">Implement either the <xref:Microsoft.AspNetCore.Mvc.Filters.IActionFilter> or <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncActionFilter> interface.</span></span>
* <span data-ttu-id="42f92-346">La loro esecuzione circonda l'esecuzione dei metodi di azione.</span><span class="sxs-lookup"><span data-stu-id="42f92-346">Their execution surrounds the execution of action methods.</span></span>

<span data-ttu-id="42f92-347">Il codice seguente mostra un filtro di azione di esempio:</span><span class="sxs-lookup"><span data-stu-id="42f92-347">The following code shows a sample action filter:</span></span>

[!code-csharp[](./filters/3.1sample/FiltersSample/Filters/MySampleActionFilter.cs?name=snippet_ActionFilter)]

<span data-ttu-id="42f92-348">La classe <xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutingContext> specifica le proprietà seguenti:</span><span class="sxs-lookup"><span data-stu-id="42f92-348">The <xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutingContext> provides the following properties:</span></span>

* <span data-ttu-id="42f92-349"><xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutingContext.ActionArguments> : consente di leggere gli input in un metodo di azione.</span><span class="sxs-lookup"><span data-stu-id="42f92-349"><xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutingContext.ActionArguments> - enables reading the inputs to an action method.</span></span>
* <span data-ttu-id="42f92-350"><xref:Microsoft.AspNetCore.Mvc.Controller>: consente di modificare l'istanza del controller.</span><span class="sxs-lookup"><span data-stu-id="42f92-350"><xref:Microsoft.AspNetCore.Mvc.Controller> - enables manipulating the controller instance.</span></span>
* <span data-ttu-id="42f92-351"><xref:System.Web.Mvc.ActionExecutingContext.Result>: l'impostazione di `Result` causa il corto circuito del metodo di azione e dei filtri di azione successivi.</span><span class="sxs-lookup"><span data-stu-id="42f92-351"><xref:System.Web.Mvc.ActionExecutingContext.Result> - setting `Result` short-circuits execution of the action method and subsequent action filters.</span></span>

<span data-ttu-id="42f92-352">La generazione di un'eccezione in un metodo di azione:</span><span class="sxs-lookup"><span data-stu-id="42f92-352">Throwing an exception in an action method:</span></span>

* <span data-ttu-id="42f92-353">Impedisce l'esecuzione dei filtri successivi.</span><span class="sxs-lookup"><span data-stu-id="42f92-353">Prevents running of subsequent filters.</span></span>
* <span data-ttu-id="42f92-354">A differenza dell'impostazione di `Result`, è considerata un errore anziché un risultato positivo.</span><span class="sxs-lookup"><span data-stu-id="42f92-354">Unlike setting `Result`, is treated as a failure instead of a successful result.</span></span>

<span data-ttu-id="42f92-355">La classe <xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutedContext> specifica `Controller` e `Result` oltre alle proprietà seguenti:</span><span class="sxs-lookup"><span data-stu-id="42f92-355">The <xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutedContext> provides `Controller` and `Result` plus the following properties:</span></span>

* <span data-ttu-id="42f92-356"><xref:System.Web.Mvc.ActionExecutedContext.Canceled>: true se un altro file ha causato il corto circuito dell'azione.</span><span class="sxs-lookup"><span data-stu-id="42f92-356"><xref:System.Web.Mvc.ActionExecutedContext.Canceled> - True if the action execution was short-circuited by another filter.</span></span>
* <span data-ttu-id="42f92-357"><xref:System.Web.Mvc.ActionExecutedContext.Exception>: non Null se l'azione o un filtro di azione eseguito in precedenza ha generato un'eccezione.</span><span class="sxs-lookup"><span data-stu-id="42f92-357"><xref:System.Web.Mvc.ActionExecutedContext.Exception> - Non-null if the action or a previously run action filter threw an exception.</span></span> <span data-ttu-id="42f92-358">Se si imposta questa proprietà su Null:</span><span class="sxs-lookup"><span data-stu-id="42f92-358">Setting this property to null:</span></span>

  * <span data-ttu-id="42f92-359">L'eccezione viene gestita in modo efficace.</span><span class="sxs-lookup"><span data-stu-id="42f92-359">Effectively handles the exception.</span></span>
  * <span data-ttu-id="42f92-360">L'oggetto `Result` viene eseguito come se fosse stato restituito dal metodo di azione.</span><span class="sxs-lookup"><span data-stu-id="42f92-360">`Result` is executed as if it was returned from the action method.</span></span>

<span data-ttu-id="42f92-361">Per un oggetto `IAsyncActionFilter`, una chiamata a <xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutionDelegate>:</span><span class="sxs-lookup"><span data-stu-id="42f92-361">For an `IAsyncActionFilter`, a call to the <xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutionDelegate>:</span></span>

* <span data-ttu-id="42f92-362">Esegue qualsiasi filtro azione successivo e il metodo di azione.</span><span class="sxs-lookup"><span data-stu-id="42f92-362">Executes any subsequent action filters and the action method.</span></span>
* <span data-ttu-id="42f92-363">Restituisce `ActionExecutedContext`.</span><span class="sxs-lookup"><span data-stu-id="42f92-363">Returns `ActionExecutedContext`.</span></span>

<span data-ttu-id="42f92-364">Per causare il corto circuito, assegnare <xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutingContext.Result?displayProperty=fullName> a un'istanza del risultato e non chiamare `next` (`ActionExecutionDelegate`).</span><span class="sxs-lookup"><span data-stu-id="42f92-364">To short-circuit, assign <xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutingContext.Result?displayProperty=fullName> to a result instance and don't call `next` (the `ActionExecutionDelegate`).</span></span>

<span data-ttu-id="42f92-365">Il framework fornisce un oggetto <xref:Microsoft.AspNetCore.Mvc.Filters.ActionFilterAttribute> astratto che è possibile impostare come sottoclasse.</span><span class="sxs-lookup"><span data-stu-id="42f92-365">The framework provides an abstract <xref:Microsoft.AspNetCore.Mvc.Filters.ActionFilterAttribute> that can be subclassed.</span></span>

<span data-ttu-id="42f92-366">Il filtro di azione `OnActionExecuting` può essere usato per:</span><span class="sxs-lookup"><span data-stu-id="42f92-366">The `OnActionExecuting` action filter can be used to:</span></span>

* <span data-ttu-id="42f92-367">Convalidare lo stato del modello.</span><span class="sxs-lookup"><span data-stu-id="42f92-367">Validate model state.</span></span>
* <span data-ttu-id="42f92-368">Restituire un errore se lo stato non è valido.</span><span class="sxs-lookup"><span data-stu-id="42f92-368">Return an error if the state is invalid.</span></span>

[!code-csharp[](./filters/3.1sample/FiltersSample/Filters/ValidateModelAttribute.cs?name=snippet)]

> [!NOTE]
> <span data-ttu-id="42f92-369">I controller annotati con l' `[ApiController]` attributo convalidano automaticamente lo stato del modello e restituiscono una risposta 400.</span><span class="sxs-lookup"><span data-stu-id="42f92-369">Controllers annotated with the `[ApiController]` attribute automatically validate model state and return a 400 response.</span></span> <span data-ttu-id="42f92-370">Per altre informazioni, vedere [Risposte HTTP 400 automatiche](xref:web-api/index#automatic-http-400-responses).</span><span class="sxs-lookup"><span data-stu-id="42f92-370">For more information, see [Automatic HTTP 400 responses](xref:web-api/index#automatic-http-400-responses).</span></span>

<span data-ttu-id="42f92-371">Il metodo `OnActionExecuted` viene eseguito dopo il metodo di azione:</span><span class="sxs-lookup"><span data-stu-id="42f92-371">The `OnActionExecuted` method runs after the action method:</span></span>

* <span data-ttu-id="42f92-372">È possibile visualizzare e modificare i risultati dell'azione tramite la proprietà <xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutedContext.Result>.</span><span class="sxs-lookup"><span data-stu-id="42f92-372">And can see and manipulate the results of the action through the <xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutedContext.Result> property.</span></span>
* <span data-ttu-id="42f92-373">L'impostazione di <xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutedContext.Canceled> è true se un altro filtro ha causato il corto circuito dell'esecuzione dell'azione.</span><span class="sxs-lookup"><span data-stu-id="42f92-373"><xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutedContext.Canceled> is set to true if the action execution was short-circuited by another filter.</span></span>
* <span data-ttu-id="42f92-374">L'impostazione di <xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutedContext.Exception> è un valore non Null se l'azione o un filtro di azione successivo ha generato un'eccezione.</span><span class="sxs-lookup"><span data-stu-id="42f92-374"><xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutedContext.Exception> is set to a non-null value if the action or a subsequent action filter threw an exception.</span></span> <span data-ttu-id="42f92-375">Impostazione di `Exception` su null:</span><span class="sxs-lookup"><span data-stu-id="42f92-375">Setting `Exception` to null:</span></span>

  * <span data-ttu-id="42f92-376">Gestisce un'eccezione in modo efficace.</span><span class="sxs-lookup"><span data-stu-id="42f92-376">Effectively handles an exception.</span></span>
  * <span data-ttu-id="42f92-377">L'oggetto `ActionExecutedContext.Result` viene eseguito come se fosse stato restituito normalmente dal metodo di azione.</span><span class="sxs-lookup"><span data-stu-id="42f92-377">`ActionExecutedContext.Result` is executed as if it were returned normally from the action method.</span></span>

[!code-csharp[](./filters/3.1sample/FiltersSample/Filters/ValidateModelAttribute.cs?name=snippet2&higlight=12-99)]

## <a name="exception-filters"></a><span data-ttu-id="42f92-378">Filtri eccezioni</span><span class="sxs-lookup"><span data-stu-id="42f92-378">Exception filters</span></span>

<span data-ttu-id="42f92-379">Filtri eccezioni:</span><span class="sxs-lookup"><span data-stu-id="42f92-379">Exception filters:</span></span>

* <span data-ttu-id="42f92-380">Implementano <xref:Microsoft.AspNetCore.Mvc.Filters.IExceptionFilter> o <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncExceptionFilter>.</span><span class="sxs-lookup"><span data-stu-id="42f92-380">Implement <xref:Microsoft.AspNetCore.Mvc.Filters.IExceptionFilter> or <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncExceptionFilter>.</span></span>
* <span data-ttu-id="42f92-381">Possono essere usati per implementare criteri di gestione degli errori comuni.</span><span class="sxs-lookup"><span data-stu-id="42f92-381">Can be used to implement common error handling policies.</span></span>

<span data-ttu-id="42f92-382">Il filtro di eccezione di esempio seguente usa una visualizzazione degli errori personalizzata per mostrare i dettagli sulle eccezioni che si verificano quando l'app è in fase di sviluppo:</span><span class="sxs-lookup"><span data-stu-id="42f92-382">The following sample exception filter uses a custom error view to display details about exceptions that occur when the app is in development:</span></span>

[!code-csharp[](./filters/3.1sample/FiltersSample/Filters/CustomExceptionFilter.cs?name=snippet_ExceptionFilter&highlight=16-19)]

<span data-ttu-id="42f92-383">Il codice seguente verifica il filtro eccezioni:</span><span class="sxs-lookup"><span data-stu-id="42f92-383">The following code tests the exception filter:</span></span>

[!code-csharp[](filters/3.1sample/FiltersSample/Controllers/FailingController.cs?name=snippet)]

<span data-ttu-id="42f92-384">Filtri eccezioni:</span><span class="sxs-lookup"><span data-stu-id="42f92-384">Exception filters:</span></span>

* <span data-ttu-id="42f92-385">Non dispone di eventi precedenti o successivi.</span><span class="sxs-lookup"><span data-stu-id="42f92-385">Don't have before and after events.</span></span>
* <span data-ttu-id="42f92-386">Implementano <xref:Microsoft.AspNetCore.Mvc.Filters.IExceptionFilter.OnException*> o <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncExceptionFilter.OnExceptionAsync*>.</span><span class="sxs-lookup"><span data-stu-id="42f92-386">Implement <xref:Microsoft.AspNetCore.Mvc.Filters.IExceptionFilter.OnException*> or <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncExceptionFilter.OnExceptionAsync*>.</span></span>
* <span data-ttu-id="42f92-387">Gestire le eccezioni non gestite che si verificano nella Razor creazione della pagina o del controller, dell' [associazione di modelli](xref:mvc/models/model-binding), dei filtri azione o dei metodi di azione.</span><span class="sxs-lookup"><span data-stu-id="42f92-387">Handle unhandled exceptions that occur in Razor Page or controller creation, [model binding](xref:mvc/models/model-binding), action filters, or action methods.</span></span>
* <span data-ttu-id="42f92-388">**Non** rilevano le eccezioni che si verificano nei filtri di risorse, nei filtri dei risultati o nell'esecuzione dei risultati MVC.</span><span class="sxs-lookup"><span data-stu-id="42f92-388">Do **not** catch exceptions that occur in resource filters, result filters, or MVC result execution.</span></span>

<span data-ttu-id="42f92-389">Per gestire un'eccezione, impostare la proprietà <xref:System.Web.Mvc.ExceptionContext.ExceptionHandled> su `true` o scrivere una risposta.</span><span class="sxs-lookup"><span data-stu-id="42f92-389">To handle an exception, set the <xref:System.Web.Mvc.ExceptionContext.ExceptionHandled> property to `true` or write a response.</span></span> <span data-ttu-id="42f92-390">In questo modo si arresta la propagazione dell'eccezione.</span><span class="sxs-lookup"><span data-stu-id="42f92-390">This stops propagation of the exception.</span></span> <span data-ttu-id="42f92-391">Un filtro di eccezione non può modificare un'eccezione in una "operazione riuscita".</span><span class="sxs-lookup"><span data-stu-id="42f92-391">An exception filter can't turn an exception into a "success".</span></span> <span data-ttu-id="42f92-392">Questa operazione può essere eseguita solo tramite un filtro di azione.</span><span class="sxs-lookup"><span data-stu-id="42f92-392">Only an action filter can do that.</span></span>

<span data-ttu-id="42f92-393">Filtri eccezioni:</span><span class="sxs-lookup"><span data-stu-id="42f92-393">Exception filters:</span></span>

* <span data-ttu-id="42f92-394">Sono ideali per intercettare le eccezioni che si verificano nelle azioni.</span><span class="sxs-lookup"><span data-stu-id="42f92-394">Are good for trapping exceptions that occur within actions.</span></span>
* <span data-ttu-id="42f92-395">Non sono flessibili quanto il middleware di gestione degli errori.</span><span class="sxs-lookup"><span data-stu-id="42f92-395">Are not as flexible as error handling middleware.</span></span>

<span data-ttu-id="42f92-396">Scegliere il middleware per la gestione delle eccezioni.</span><span class="sxs-lookup"><span data-stu-id="42f92-396">Prefer middleware for exception handling.</span></span> <span data-ttu-id="42f92-397">Usare i filtri di eccezione solo quando la gestione degli errori *varia* in base al metodo di azione chiamato.</span><span class="sxs-lookup"><span data-stu-id="42f92-397">Use exception filters only where error handling *differs* based on which action method is called.</span></span> <span data-ttu-id="42f92-398">Un'app può ad esempio avere metodi di azione sia per gli endpoint API che per le visualizzazioni/HTML.</span><span class="sxs-lookup"><span data-stu-id="42f92-398">For example, an app might have action methods for both API endpoints and for views/HTML.</span></span> <span data-ttu-id="42f92-399">Gli endpoint dell'API possono restituire informazioni sull'errore come JSON, mentre le azioni basate sulla visualizzazione possono restituire una pagina di errore in formato HTML.</span><span class="sxs-lookup"><span data-stu-id="42f92-399">The API endpoints could return error information as JSON, while the view-based actions could return an error page as HTML.</span></span>

## <a name="result-filters"></a><span data-ttu-id="42f92-400">Filtri risultato</span><span class="sxs-lookup"><span data-stu-id="42f92-400">Result filters</span></span>

<span data-ttu-id="42f92-401">I filtri dei risultati:</span><span class="sxs-lookup"><span data-stu-id="42f92-401">Result filters:</span></span>

* <span data-ttu-id="42f92-402">Implementare un'interfaccia:</span><span class="sxs-lookup"><span data-stu-id="42f92-402">Implement an interface:</span></span>
  * <span data-ttu-id="42f92-403"><xref:Microsoft.AspNetCore.Mvc.Filters.IResultFilter> o <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncResultFilter></span><span class="sxs-lookup"><span data-stu-id="42f92-403"><xref:Microsoft.AspNetCore.Mvc.Filters.IResultFilter> or <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncResultFilter></span></span>
  * <span data-ttu-id="42f92-404"><xref:Microsoft.AspNetCore.Mvc.Filters.IAlwaysRunResultFilter> o <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncAlwaysRunResultFilter></span><span class="sxs-lookup"><span data-stu-id="42f92-404"><xref:Microsoft.AspNetCore.Mvc.Filters.IAlwaysRunResultFilter> or <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncAlwaysRunResultFilter></span></span>
* <span data-ttu-id="42f92-405">La loro esecuzione circonda l'esecuzione dei risultati dell'azione.</span><span class="sxs-lookup"><span data-stu-id="42f92-405">Their execution surrounds the execution of action results.</span></span>

### <a name="iresultfilter-and-iasyncresultfilter"></a><span data-ttu-id="42f92-406">IResultFilter e IAsyncResultFilter</span><span class="sxs-lookup"><span data-stu-id="42f92-406">IResultFilter and IAsyncResultFilter</span></span>

<span data-ttu-id="42f92-407">Il codice seguente illustra un filtro dei risultati che aggiunge un'intestazione HTTP:</span><span class="sxs-lookup"><span data-stu-id="42f92-407">The following code shows a result filter that adds an HTTP header:</span></span>

[!code-csharp[](./filters/3.1sample/FiltersSample/Filters/LoggingAddHeaderFilter.cs?name=snippet_ResultFilter)]

<span data-ttu-id="42f92-408">Il tipo di risultato eseguito dipende dall'azione.</span><span class="sxs-lookup"><span data-stu-id="42f92-408">The kind of result being executed depends on the action.</span></span> <span data-ttu-id="42f92-409">Un'azione che restituisce una visualizzazione include tutte le operazioni di elaborazione Razor come parte dell' <xref:Microsoft.AspNetCore.Mvc.ViewResult> oggetto in esecuzione.</span><span class="sxs-lookup"><span data-stu-id="42f92-409">An action returning a view includes all razor processing as part of the <xref:Microsoft.AspNetCore.Mvc.ViewResult> being executed.</span></span> <span data-ttu-id="42f92-410">Un metodo API può eseguire la serializzazione in quanto parte dell'esecuzione del risultato.</span><span class="sxs-lookup"><span data-stu-id="42f92-410">An API method might perform some serialization as part of the execution of the result.</span></span> <span data-ttu-id="42f92-411">Altre informazioni sui [risultati dell'azione](xref:mvc/controllers/actions).</span><span class="sxs-lookup"><span data-stu-id="42f92-411">Learn more about [action results](xref:mvc/controllers/actions).</span></span>

<span data-ttu-id="42f92-412">I filtri dei risultati vengono eseguiti solo quando un filtro azione o azione produce un risultato di azione.</span><span class="sxs-lookup"><span data-stu-id="42f92-412">Result filters are only executed when an action or action filter produces an action result.</span></span> <span data-ttu-id="42f92-413">I filtri dei risultati non vengono eseguiti nei casi seguenti:</span><span class="sxs-lookup"><span data-stu-id="42f92-413">Result filters are not executed when:</span></span>

* <span data-ttu-id="42f92-414">Un filtro di autorizzazione o un filtro risorse cortocircui la pipeline.</span><span class="sxs-lookup"><span data-stu-id="42f92-414">An authorization filter or resource filter short-circuits the pipeline.</span></span>
* <span data-ttu-id="42f92-415">Un filtro di eccezione non gestisca un'eccezione producendo un risultato dell'azione.</span><span class="sxs-lookup"><span data-stu-id="42f92-415">An exception filter handles an exception by producing an action result.</span></span>

<span data-ttu-id="42f92-416">Il metodo <xref:Microsoft.AspNetCore.Mvc.Filters.IResultFilter.OnResultExecuting*?displayProperty=fullName> può causare il corto circuito dell'esecuzione del risultato dell'azione e dei filtri dei risultati successivi se si imposta <xref:Microsoft.AspNetCore.Mvc.Filters.ResultExecutingContext.Cancel?displayProperty=fullName> su `true`.</span><span class="sxs-lookup"><span data-stu-id="42f92-416">The <xref:Microsoft.AspNetCore.Mvc.Filters.IResultFilter.OnResultExecuting*?displayProperty=fullName> method can short-circuit execution of the action result and subsequent result filters by setting <xref:Microsoft.AspNetCore.Mvc.Filters.ResultExecutingContext.Cancel?displayProperty=fullName> to `true`.</span></span> <span data-ttu-id="42f92-417">In caso di corto circuito, scrivere nell'oggetto risposta per evitare di generare una risposta vuota.</span><span class="sxs-lookup"><span data-stu-id="42f92-417">Write to the response object when short-circuiting to avoid generating an empty response.</span></span> <span data-ttu-id="42f92-418">Generazione di un'eccezione in `IResultFilter.OnResultExecuting` :</span><span class="sxs-lookup"><span data-stu-id="42f92-418">Throwing an exception in `IResultFilter.OnResultExecuting`:</span></span>

* <span data-ttu-id="42f92-419">Impedisce l'esecuzione del risultato dell'azione e dei filtri successivi.</span><span class="sxs-lookup"><span data-stu-id="42f92-419">Prevents execution of the action result and subsequent filters.</span></span>
* <span data-ttu-id="42f92-420">Viene considerato un errore anziché un risultato positivo.</span><span class="sxs-lookup"><span data-stu-id="42f92-420">Is treated as a failure instead of a successful result.</span></span>

<span data-ttu-id="42f92-421">Quando <xref:Microsoft.AspNetCore.Mvc.Filters.IResultFilter.OnResultExecuted*?displayProperty=fullName> viene eseguito il metodo, è probabile che la risposta sia già stata inviata al client.</span><span class="sxs-lookup"><span data-stu-id="42f92-421">When the <xref:Microsoft.AspNetCore.Mvc.Filters.IResultFilter.OnResultExecuted*?displayProperty=fullName> method runs, the response has probably already been sent to the client.</span></span> <span data-ttu-id="42f92-422">Se la risposta è già stata inviata al client, non è possibile modificarla.</span><span class="sxs-lookup"><span data-stu-id="42f92-422">If the response has already been sent to the client, it cannot be changed.</span></span>

<span data-ttu-id="42f92-423">L'impostazione di `ResultExecutedContext.Canceled` è `true` se un altro filtro ha causato il corto circuito dell'esecuzione del risultato dell'azione.</span><span class="sxs-lookup"><span data-stu-id="42f92-423">`ResultExecutedContext.Canceled` is set to `true` if the action result execution was short-circuited by another filter.</span></span>

<span data-ttu-id="42f92-424">L'impostazione di `ResultExecutedContext.Exception` è un valore non Null se il risultato dell'azione o un filtro dei risultati successivo ha generato un'eccezione.</span><span class="sxs-lookup"><span data-stu-id="42f92-424">`ResultExecutedContext.Exception` is set to a non-null value if the action result or a subsequent result filter threw an exception.</span></span> <span data-ttu-id="42f92-425">`Exception`L'impostazione di su null consente di gestire in modo efficace un'eccezione e impedisce che l'eccezione venga generata nuovamente in un secondo momento nella pipeline.</span><span class="sxs-lookup"><span data-stu-id="42f92-425">Setting `Exception` to null effectively handles an exception and prevents the exception from being thrown again later in the pipeline.</span></span> <span data-ttu-id="42f92-426">Non c'è alcun modo affidabile per scrivere i dati in una risposta quando si gestisce un'eccezione in un filtro dei risultati.</span><span class="sxs-lookup"><span data-stu-id="42f92-426">There is no reliable way to write data to a response when handling an exception in a result filter.</span></span> <span data-ttu-id="42f92-427">Se le intestazioni sono state scaricate nel client quando il risultato di un'azione genera un'eccezione, non c'è alcun meccanismo affidabile per inviare un codice di errore.</span><span class="sxs-lookup"><span data-stu-id="42f92-427">If the headers have been flushed to the client when an action result throws an exception, there's no reliable mechanism to send a failure code.</span></span>

<span data-ttu-id="42f92-428">Per un oggetto <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncResultFilter>, una chiamata a `await next` in <xref:Microsoft.AspNetCore.Mvc.Filters.ResultExecutionDelegate> esegue tutti i filtri dei risultati successivi e il risultato dell'azione.</span><span class="sxs-lookup"><span data-stu-id="42f92-428">For an <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncResultFilter>, a call to `await next` on the <xref:Microsoft.AspNetCore.Mvc.Filters.ResultExecutionDelegate> executes any subsequent result filters and the action result.</span></span> <span data-ttu-id="42f92-429">Per causare il corto circuito, impostare [ResultExecutingContext.Cancel](xref:Microsoft.AspNetCore.Mvc.Filters.ResultExecutingContext.Cancel) su `true` e non chiamare `ResultExecutionDelegate`:</span><span class="sxs-lookup"><span data-stu-id="42f92-429">To short-circuit, set [ResultExecutingContext.Cancel](xref:Microsoft.AspNetCore.Mvc.Filters.ResultExecutingContext.Cancel) to `true` and don't call the `ResultExecutionDelegate`:</span></span>

[!code-csharp[](./filters/3.1sample/FiltersSample/Filters/MyAsyncResponseFilter.cs?name=snippet)]

<span data-ttu-id="42f92-430">Il framework fornisce un oggetto `ResultFilterAttribute` astratto che è possibile impostare come sottoclasse.</span><span class="sxs-lookup"><span data-stu-id="42f92-430">The framework provides an abstract `ResultFilterAttribute` that can be subclassed.</span></span> <span data-ttu-id="42f92-431">La classe [AddHeaderAttribute](#add-header-attribute) illustrata in precedenza è un esempio di un attributo di filtro dei risultati.</span><span class="sxs-lookup"><span data-stu-id="42f92-431">The [AddHeaderAttribute](#add-header-attribute) class shown previously is an example of a result filter attribute.</span></span>

### <a name="ialwaysrunresultfilter-and-iasyncalwaysrunresultfilter"></a><span data-ttu-id="42f92-432">IAlwaysRunResultFilter e IAsyncAlwaysRunResultFilter</span><span class="sxs-lookup"><span data-stu-id="42f92-432">IAlwaysRunResultFilter and IAsyncAlwaysRunResultFilter</span></span>

<span data-ttu-id="42f92-433">Le interfacce <xref:Microsoft.AspNetCore.Mvc.Filters.IAlwaysRunResultFilter> e <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncAlwaysRunResultFilter> dichiarano un'implementazione di <xref:Microsoft.AspNetCore.Mvc.Filters.IResultFilter> eseguita per tutti i risultati dell'azione.</span><span class="sxs-lookup"><span data-stu-id="42f92-433">The <xref:Microsoft.AspNetCore.Mvc.Filters.IAlwaysRunResultFilter> and <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncAlwaysRunResultFilter> interfaces declare an <xref:Microsoft.AspNetCore.Mvc.Filters.IResultFilter> implementation that runs for all action results.</span></span> <span data-ttu-id="42f92-434">Sono inclusi i risultati dell'azione prodotti da:</span><span class="sxs-lookup"><span data-stu-id="42f92-434">This includes action results produced by:</span></span>

* <span data-ttu-id="42f92-435">Filtri di autorizzazione e filtri delle risorse che interessano il cortocircuito.</span><span class="sxs-lookup"><span data-stu-id="42f92-435">Authorization filters and resource filters that short-circuit.</span></span>
* <span data-ttu-id="42f92-436">Filtri eccezioni.</span><span class="sxs-lookup"><span data-stu-id="42f92-436">Exception filters.</span></span>

<span data-ttu-id="42f92-437">Ad esempio, il filtro seguente viene eseguito sempre e imposta un risultato dell'azione (<xref:Microsoft.AspNetCore.Mvc.ObjectResult>) con un codice di stato *422 Entità non elaborabile* quando la negoziazione del contenuto ha esito negativo:</span><span class="sxs-lookup"><span data-stu-id="42f92-437">For example, the following filter always runs and sets an action result (<xref:Microsoft.AspNetCore.Mvc.ObjectResult>) with a *422 Unprocessable Entity* status code when content negotiation fails:</span></span>

[!code-csharp[](./filters/3.1sample/FiltersSample/Filters/UnprocessableResultFilter.cs?name=snippet)]

## <a name="ifilterfactory"></a><span data-ttu-id="42f92-438">IFilterFactory</span><span class="sxs-lookup"><span data-stu-id="42f92-438">IFilterFactory</span></span>

<span data-ttu-id="42f92-439"><xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory> implementa <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterMetadata>.</span><span class="sxs-lookup"><span data-stu-id="42f92-439"><xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory> implements <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterMetadata>.</span></span> <span data-ttu-id="42f92-440">Pertanto, un'istanza di `IFilterFactory` può essere usata come un'istanza di `IFilterMetadata` in un punto qualsiasi della pipeline filtro.</span><span class="sxs-lookup"><span data-stu-id="42f92-440">Therefore, an `IFilterFactory` instance can be used as an `IFilterMetadata` instance anywhere in the filter pipeline.</span></span> <span data-ttu-id="42f92-441">Quando il runtime si prepara per richiamare il filtro, cerca di eseguirne il cast a un oggetto `IFilterFactory`.</span><span class="sxs-lookup"><span data-stu-id="42f92-441">When the runtime prepares to invoke the filter, it attempts to cast it to an `IFilterFactory`.</span></span> <span data-ttu-id="42f92-442">Se l'esecuzione del cast ha esito positivo, viene chiamato il metodo <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory.CreateInstance*> per creare l'istanza di `IFilterMetadata` richiamata.</span><span class="sxs-lookup"><span data-stu-id="42f92-442">If that cast succeeds, the <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory.CreateInstance*> method is called to create the `IFilterMetadata` instance that is invoked.</span></span> <span data-ttu-id="42f92-443">In questo modo viene specificata una struttura flessibile, poiché non è necessario impostare in modo esplicito la pipeline filtro all'avvio dell'app.</span><span class="sxs-lookup"><span data-stu-id="42f92-443">This provides a flexible design, since the precise filter pipeline doesn't need to be set explicitly when the app starts.</span></span>

<span data-ttu-id="42f92-444">`IFilterFactory.IsReusable`:</span><span class="sxs-lookup"><span data-stu-id="42f92-444">`IFilterFactory.IsReusable`:</span></span>

* <span data-ttu-id="42f92-445">È un suggerimento della factory che l'istanza di filtro creata dalla factory può essere riutilizzata all'esterno dell'ambito della richiesta in cui è stata creata.</span><span class="sxs-lookup"><span data-stu-id="42f92-445">Is a hint by the factory that the filter instance created by the factory may be reused outside of the request scope it was created within.</span></span>
* <span data-ttu-id="42f92-446">***Non*** deve essere usato con un filtro che dipende da servizi con una durata diversa da singleton.</span><span class="sxs-lookup"><span data-stu-id="42f92-446">Should ***not*** be used with a filter that depends on services with a lifetime other than singleton.</span></span>

<span data-ttu-id="42f92-447">Il runtime di ASP.NET Core non garantisce:</span><span class="sxs-lookup"><span data-stu-id="42f92-447">The ASP.NET Core runtime doesn't guarantee:</span></span>

* <span data-ttu-id="42f92-448">Che venga creata una singola istanza del filtro.</span><span class="sxs-lookup"><span data-stu-id="42f92-448">That a single instance of the filter will be created.</span></span>
* <span data-ttu-id="42f92-449">Che il filtro non verrà richiesto di nuovo dal contenitore di inserimento delle dipendenze in un momento successivo.</span><span class="sxs-lookup"><span data-stu-id="42f92-449">The filter will not be re-requested from the DI container at some later point.</span></span>

> [!WARNING] 
> <span data-ttu-id="42f92-450">Configurare <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory.IsReusable?displayProperty=nameWithType> per restituire solo `true` se l'origine dei filtri non è ambigua, i filtri sono senza stato e i filtri sono sicuri per l'uso in più richieste HTTP.</span><span class="sxs-lookup"><span data-stu-id="42f92-450">Only configure <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory.IsReusable?displayProperty=nameWithType> to return `true` if the source of the filters is unambiguous, the filters are stateless, and the filters are safe to use across multiple HTTP requests.</span></span> <span data-ttu-id="42f92-451">Ad esempio, non restituire i filtri da DI DI cui è stata eseguita la registrazione come ambito o temporaneo se `IFilterFactory.IsReusable` restituisce `true` .</span><span class="sxs-lookup"><span data-stu-id="42f92-451">For instance, don't return filters from DI that are registered as scoped or transient if `IFilterFactory.IsReusable` returns `true`.</span></span>

<span data-ttu-id="42f92-452">Un altro approccio alla creazione di filtri consiste nell'implementare `IFilterFactory` usando implementazioni dell'attributo personalizzate:</span><span class="sxs-lookup"><span data-stu-id="42f92-452">`IFilterFactory` can be implemented using custom attribute implementations as another approach to creating filters:</span></span>

[!code-csharp[](./filters/3.1sample/FiltersSample/Filters/AddHeaderWithFactoryAttribute.cs?name=snippet_IFilterFactory&highlight=1,4,5,6,7)]

<span data-ttu-id="42f92-453">Il filtro viene applicato nel codice seguente:</span><span class="sxs-lookup"><span data-stu-id="42f92-453">The filter is applied in the following code:</span></span>

[!code-csharp[](./filters/3.1sample/FiltersSample/Controllers/SampleController.cs?name=snippet3&highlight=21)]

<span data-ttu-id="42f92-454">Testare il codice precedente eseguendo l' [esempio di download](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/controllers/filters/3.1sample):</span><span class="sxs-lookup"><span data-stu-id="42f92-454">Test the preceding code by running the [download sample](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/controllers/filters/3.1sample):</span></span>

* <span data-ttu-id="42f92-455">Richiamare gli strumenti di sviluppo F12.</span><span class="sxs-lookup"><span data-stu-id="42f92-455">Invoke the F12 developer tools.</span></span>
* <span data-ttu-id="42f92-456">Accedere a `https://localhost:5001/Sample/HeaderWithFactory`.</span><span class="sxs-lookup"><span data-stu-id="42f92-456">Navigate to `https://localhost:5001/Sample/HeaderWithFactory`.</span></span>

<span data-ttu-id="42f92-457">Gli strumenti di sviluppo F12 visualizzano le intestazioni di risposta seguenti aggiunte dal codice di esempio:</span><span class="sxs-lookup"><span data-stu-id="42f92-457">The F12 developer tools display the following response headers added by the sample code:</span></span>

* <span data-ttu-id="42f92-458">**Autore:**`Rick Anderson`</span><span class="sxs-lookup"><span data-stu-id="42f92-458">**author:** `Rick Anderson`</span></span>
* <span data-ttu-id="42f92-459">**globaladdheader:** `Result filter added to MvcOptions.Filters`</span><span class="sxs-lookup"><span data-stu-id="42f92-459">**globaladdheader:** `Result filter added to MvcOptions.Filters`</span></span>
* <span data-ttu-id="42f92-460">**interno:**`My header`</span><span class="sxs-lookup"><span data-stu-id="42f92-460">**internal:** `My header`</span></span>

<span data-ttu-id="42f92-461">Il codice precedente crea l'intestazione di risposta **internal:** `My header`.</span><span class="sxs-lookup"><span data-stu-id="42f92-461">The preceding code creates the **internal:** `My header` response header.</span></span>

### <a name="ifilterfactory-implemented-on-an-attribute"></a><span data-ttu-id="42f92-462">Implementazione di IFilterFactory in un attributo</span><span class="sxs-lookup"><span data-stu-id="42f92-462">IFilterFactory implemented on an attribute</span></span>

<!-- Review 
This section needs to be rewritten.
What's a non-named attribute?
-->

<span data-ttu-id="42f92-463">I filtri che implementano `IFilterFactory` sono utili per i filtri che:</span><span class="sxs-lookup"><span data-stu-id="42f92-463">Filters that implement `IFilterFactory` are useful for filters that:</span></span>

* <span data-ttu-id="42f92-464">Non richiedono il passaggio di parametri.</span><span class="sxs-lookup"><span data-stu-id="42f92-464">Don't require passing parameters.</span></span>
* <span data-ttu-id="42f92-465">Hanno dipendenze del costruttore che devono essere soddisfatte dall'inserimento delle dipendenze.</span><span class="sxs-lookup"><span data-stu-id="42f92-465">Have constructor dependencies that need to be filled by DI.</span></span>

<span data-ttu-id="42f92-466"><xref:Microsoft.AspNetCore.Mvc.TypeFilterAttribute> implementa <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory>.</span><span class="sxs-lookup"><span data-stu-id="42f92-466"><xref:Microsoft.AspNetCore.Mvc.TypeFilterAttribute> implements <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory>.</span></span> <span data-ttu-id="42f92-467">`IFilterFactory` espone il metodo <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory.CreateInstance*> per creare un'istanza <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterMetadata>.</span><span class="sxs-lookup"><span data-stu-id="42f92-467">`IFilterFactory` exposes the <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory.CreateInstance*> method for creating an <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterMetadata> instance.</span></span> <span data-ttu-id="42f92-468">`CreateInstance` carica il tipo specificato dal contenitore di inserimento delle dipendenze.</span><span class="sxs-lookup"><span data-stu-id="42f92-468">`CreateInstance` loads the specified type from the services container (DI).</span></span>

[!code-csharp[](./filters/3.1sample/FiltersSample/Filters/SampleActionFilterAttribute.cs?name=snippet_TypeFilterAttribute&highlight=1,3,7)]

<span data-ttu-id="42f92-469">Il codice seguente illustra tre approcci all'applicazione di `[SampleActionFilter]`:</span><span class="sxs-lookup"><span data-stu-id="42f92-469">The following code shows three approaches to applying the `[SampleActionFilter]`:</span></span>

[!code-csharp[](./filters/3.1sample/FiltersSample/Controllers/HomeController.cs?name=snippet&highlight=1)]

<span data-ttu-id="42f92-470">Nel codice precedente la decorazione del metodo con `[SampleActionFilter]` è l'approccio da preferire per applicare `SampleActionFilter`.</span><span class="sxs-lookup"><span data-stu-id="42f92-470">In the preceding code, decorating the method with `[SampleActionFilter]` is the preferred approach to applying the `SampleActionFilter`.</span></span>

## <a name="using-middleware-in-the-filter-pipeline"></a><span data-ttu-id="42f92-471">Uso di middleware nella pipeline filtro</span><span class="sxs-lookup"><span data-stu-id="42f92-471">Using middleware in the filter pipeline</span></span>

<span data-ttu-id="42f92-472">I filtri risorse funzionano come [middleware](xref:fundamentals/middleware/index) in quanto racchiudono l'esecuzione di tutto ciò che viene dopo nella pipeline.</span><span class="sxs-lookup"><span data-stu-id="42f92-472">Resource filters work like [middleware](xref:fundamentals/middleware/index) in that they surround the execution of everything that comes later in the pipeline.</span></span> <span data-ttu-id="42f92-473">Tuttavia, i filtri differiscono dal middleware in quanto fanno parte del runtime, il che significa che hanno accesso a contesto e costrutti.</span><span class="sxs-lookup"><span data-stu-id="42f92-473">But filters differ from middleware in that they're part of the runtime, which means that they have access to context and constructs.</span></span>

<span data-ttu-id="42f92-474">Per usare il middleware come filtro, creare un tipo con un metodo `Configure` che specifica il middleware da inserire nella pipeline di filtro.</span><span class="sxs-lookup"><span data-stu-id="42f92-474">To use middleware as a filter, create a type with a `Configure` method that specifies the middleware to inject into the filter pipeline.</span></span> <span data-ttu-id="42f92-475">L'esempio seguente usa il middleware di localizzazione per stabilire le impostazioni cultura correnti per una richiesta:</span><span class="sxs-lookup"><span data-stu-id="42f92-475">The following example uses the localization middleware to establish the current culture for a request:</span></span>

[!code-csharp[](./filters/3.1sample/FiltersSample/Filters/LocalizationPipeline.cs?name=snippet_MiddlewareFilter&highlight=3,22)]

<span data-ttu-id="42f92-476">Usare <xref:Microsoft.AspNetCore.Mvc.MiddlewareFilterAttribute> per eseguire il middleware:</span><span class="sxs-lookup"><span data-stu-id="42f92-476">Use the <xref:Microsoft.AspNetCore.Mvc.MiddlewareFilterAttribute> to run the middleware:</span></span>

[!code-csharp[](./filters/3.1sample/FiltersSample/Controllers/HomeController.cs?name=snippet_MiddlewareFilter&highlight=2)]

<span data-ttu-id="42f92-477">I filtri middleware vengono eseguiti nella stessa fase della pipeline filtro come filtri risorse, prima dell'associazione di modelli e dopo il resto della pipeline.</span><span class="sxs-lookup"><span data-stu-id="42f92-477">Middleware filters run at the same stage of the filter pipeline as Resource filters, before model binding and after the rest of the pipeline.</span></span>

## <a name="next-actions"></a><span data-ttu-id="42f92-478">Azioni successive</span><span class="sxs-lookup"><span data-stu-id="42f92-478">Next actions</span></span>

* <span data-ttu-id="42f92-479">Vedere [metodi di filtro per le Razor pagine](xref:razor-pages/filter).</span><span class="sxs-lookup"><span data-stu-id="42f92-479">See [Filter methods for Razor Pages](xref:razor-pages/filter).</span></span>
* <span data-ttu-id="42f92-480">Per sperimentare i filtri, [scaricare, testare e modificare l'esempio di GitHub](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/controllers/filters/3.1sample).</span><span class="sxs-lookup"><span data-stu-id="42f92-480">To experiment with filters, [download, test, and modify the GitHub sample](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/controllers/filters/3.1sample).</span></span>

::: moniker-end

::: moniker range="< aspnetcore-3.0"

<span data-ttu-id="42f92-481">Di [Kirk Larkin](https://github.com/serpent5), [Rick Anderson](https://twitter.com/RickAndMSFT), [Tom Dykstra](https://github.com/tdykstra/) e [Steve Smith](https://ardalis.com/)</span><span class="sxs-lookup"><span data-stu-id="42f92-481">By [Kirk Larkin](https://github.com/serpent5), [Rick Anderson](https://twitter.com/RickAndMSFT), [Tom Dykstra](https://github.com/tdykstra/), and [Steve Smith](https://ardalis.com/)</span></span>

<span data-ttu-id="42f92-482">I *filtri* in ASP.NET Core consentono l'esecuzione del codice prima o dopo fasi specifiche della pipeline di elaborazione della richiesta.</span><span class="sxs-lookup"><span data-stu-id="42f92-482">*Filters* in ASP.NET Core allow code to be run before or after specific stages in the request processing pipeline.</span></span>

<span data-ttu-id="42f92-483">I filtri predefiniti gestiscono attività, ad esempio:</span><span class="sxs-lookup"><span data-stu-id="42f92-483">Built-in filters handle tasks such as:</span></span>

* <span data-ttu-id="42f92-484">Autorizzazione (impedire l'accesso alle risorse per cui un utente non è autorizzato).</span><span class="sxs-lookup"><span data-stu-id="42f92-484">Authorization (preventing access to resources a user isn't authorized for).</span></span>
* <span data-ttu-id="42f92-485">Memorizzazione nella cache delle risposte (blocco della pipeline delle richieste per restituire una risposta memorizzata nella cache).</span><span class="sxs-lookup"><span data-stu-id="42f92-485">Response caching (short-circuiting the request pipeline to return a cached response).</span></span>

<span data-ttu-id="42f92-486">I filtri personalizzati possono essere creati per gestire problemi relativi a più settori.</span><span class="sxs-lookup"><span data-stu-id="42f92-486">Custom filters can be created to handle cross-cutting concerns.</span></span> <span data-ttu-id="42f92-487">Esempi di problematiche trasversali includono la gestione degli errori, la memorizzazione nella cache, la configurazione, l'autorizzazione e la registrazione.</span><span class="sxs-lookup"><span data-stu-id="42f92-487">Examples of cross-cutting concerns include error handling, caching, configuration, authorization, and logging.</span></span>  <span data-ttu-id="42f92-488">I filtri evitano la duplicazione del codice.</span><span class="sxs-lookup"><span data-stu-id="42f92-488">Filters avoid duplicating code.</span></span> <span data-ttu-id="42f92-489">Ad esempio, un filtro eccezioni per la gestione degli errori potrebbe consolidare la gestione degli errori.</span><span class="sxs-lookup"><span data-stu-id="42f92-489">For example, an error handling exception filter could consolidate error handling.</span></span>

<span data-ttu-id="42f92-490">Questo documento si applica a Razor pagine, controller API e controller con visualizzazioni.</span><span class="sxs-lookup"><span data-stu-id="42f92-490">This document applies to Razor Pages, API controllers, and controllers with views.</span></span>

<span data-ttu-id="42f92-491">[Visualizzare o scaricare un esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/controllers/filters/sample) ([procedura per il download](xref:index#how-to-download-a-sample)).</span><span class="sxs-lookup"><span data-stu-id="42f92-491">[View or download sample](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/controllers/filters/sample) ([how to download](xref:index#how-to-download-a-sample)).</span></span>

## <a name="how-filters-work"></a><span data-ttu-id="42f92-492">Funzionamento dei filtri</span><span class="sxs-lookup"><span data-stu-id="42f92-492">How filters work</span></span>

<span data-ttu-id="42f92-493">I filtri vengono eseguiti all'interno della *pipeline di chiamata di azioni ASP.NET Core*, talvolta definita *pipeline di filtro*.</span><span class="sxs-lookup"><span data-stu-id="42f92-493">Filters run within the *ASP.NET Core action invocation pipeline*, sometimes referred to as the *filter pipeline*.</span></span>  <span data-ttu-id="42f92-494">La pipeline di filtro viene eseguita dopo che ASP.NET Core ha selezionato l'azione da eseguire.</span><span class="sxs-lookup"><span data-stu-id="42f92-494">The filter pipeline runs after ASP.NET Core selects the action to execute.</span></span>

![La richiesta viene elaborata tramite altro middleware, il middleware di routing, la selezione dell'azione e la pipeline di chiamata di azioni ASP.NET Core.](filters/_static/filter-pipeline-1.png)

### <a name="filter-types"></a><span data-ttu-id="42f92-497">Tipi di filtro</span><span class="sxs-lookup"><span data-stu-id="42f92-497">Filter types</span></span>

<span data-ttu-id="42f92-498">Ogni tipo di filtro viene eseguito in una fase diversa della pipeline di filtro:</span><span class="sxs-lookup"><span data-stu-id="42f92-498">Each filter type is executed at a different stage in the filter pipeline:</span></span>

* <span data-ttu-id="42f92-499">I [filtri di autorizzazione](#authorization-filters) vengono eseguiti per primi e consentono di determinare se l'utente è autorizzato per la richiesta.</span><span class="sxs-lookup"><span data-stu-id="42f92-499">[Authorization filters](#authorization-filters) run first and are used to determine whether the user is authorized for the request.</span></span> <span data-ttu-id="42f92-500">I filtri di autorizzazione causano il corto circuito della pipeline se la richiesta non è autorizzata.</span><span class="sxs-lookup"><span data-stu-id="42f92-500">Authorization filters short-circuit the pipeline if the request is unauthorized.</span></span>

* <span data-ttu-id="42f92-501">[Filtri risorse](#resource-filters):</span><span class="sxs-lookup"><span data-stu-id="42f92-501">[Resource filters](#resource-filters):</span></span>

  * <span data-ttu-id="42f92-502">Vengono eseguiti dopo l'autorizzazione.</span><span class="sxs-lookup"><span data-stu-id="42f92-502">Run after authorization.</span></span>  
  * <span data-ttu-id="42f92-503"><xref:Microsoft.AspNetCore.Mvc.Filters.IResourceFilter.OnResourceExecuting*> può eseguire codice prima del resto della pipeline di filtro.</span><span class="sxs-lookup"><span data-stu-id="42f92-503"><xref:Microsoft.AspNetCore.Mvc.Filters.IResourceFilter.OnResourceExecuting*> can run code before the rest of the filter pipeline.</span></span> <span data-ttu-id="42f92-504">Ad esempio, `OnResourceExecuting` può eseguire codice prima dell'associazione di modelli.</span><span class="sxs-lookup"><span data-stu-id="42f92-504">For example, `OnResourceExecuting` can run code before model binding.</span></span>
  * <span data-ttu-id="42f92-505"><xref:Microsoft.AspNetCore.Mvc.Filters.IResourceFilter.OnResourceExecuted*> può eseguire codice dopo che il resto della pipeline è stato completato.</span><span class="sxs-lookup"><span data-stu-id="42f92-505"><xref:Microsoft.AspNetCore.Mvc.Filters.IResourceFilter.OnResourceExecuted*> can run code after the rest of the pipeline has completed.</span></span>

* <span data-ttu-id="42f92-506">I [filtri azione](#action-filters) possono eseguire codice immediatamente prima e dopo la chiamata di un metodo di azione singolo.</span><span class="sxs-lookup"><span data-stu-id="42f92-506">[Action filters](#action-filters) can run code immediately before and after an individual action method is called.</span></span> <span data-ttu-id="42f92-507">Possono essere usati per modificare gli argomenti passati a un'azione e il risultato restituito dall'azione.</span><span class="sxs-lookup"><span data-stu-id="42f92-507">They can be used to manipulate the arguments passed into an action and the result returned from the action.</span></span> <span data-ttu-id="42f92-508">I filtri azione **non** sono supportati nelle Razor pagine.</span><span class="sxs-lookup"><span data-stu-id="42f92-508">Action filters are **not** supported in Razor Pages.</span></span>

* <span data-ttu-id="42f92-509">I [filtri eccezioni](#exception-filters) vengono usati per applicare i criteri globali a eccezioni non gestite che si verificano prima che qualsiasi elemento sia stato scritto nel corpo della risposta.</span><span class="sxs-lookup"><span data-stu-id="42f92-509">[Exception filters](#exception-filters) are used to apply global policies to unhandled exceptions that occur before anything has been written to the response body.</span></span>

* <span data-ttu-id="42f92-510">I [filtri risultato](#result-filters) possono eseguire codice immediatamente prima e dopo l'esecuzione di risultati di azioni singole.</span><span class="sxs-lookup"><span data-stu-id="42f92-510">[Result filters](#result-filters) can run code immediately before and after the execution of individual action results.</span></span> <span data-ttu-id="42f92-511">Vengono eseguiti solo quando il metodo di azione è stato eseguito correttamente.</span><span class="sxs-lookup"><span data-stu-id="42f92-511">They run only when the action method has executed successfully.</span></span> <span data-ttu-id="42f92-512">Sono utili per la logica che deve racchiudere l'esecuzione del formattatore o di una vista.</span><span class="sxs-lookup"><span data-stu-id="42f92-512">They are useful for logic that must surround view or formatter execution.</span></span>

<span data-ttu-id="42f92-513">Il diagramma seguente illustra come interagiscono i tipi di filtro nella pipeline di filtro.</span><span class="sxs-lookup"><span data-stu-id="42f92-513">The following diagram shows how filter types interact in the filter pipeline.</span></span>

![La richiesta passa attraverso i filtri autorizzazione, i filtri risorse, l'associazione di modelli, i filtri azione, l'esecuzione dell'azione e la conversione del risultato dell'azione, i filtri eccezioni, i filtri risultato e l'esecuzione del risultato.](filters/_static/filter-pipeline-2.png)

## <a name="implementation"></a><span data-ttu-id="42f92-516">Implementazione</span><span class="sxs-lookup"><span data-stu-id="42f92-516">Implementation</span></span>

<span data-ttu-id="42f92-517">I filtri supportano sia implementazioni sincrone che asincrone tramite definizioni di interfaccia diverse.</span><span class="sxs-lookup"><span data-stu-id="42f92-517">Filters support both synchronous and asynchronous implementations through different interface definitions.</span></span>

<span data-ttu-id="42f92-518">I filtri sincroni possono eseguire codice prima (`On-Stage-Executing`) e dopo (`On-Stage-Executed`) la relativa fase della pipeline.</span><span class="sxs-lookup"><span data-stu-id="42f92-518">Synchronous filters can run code before (`On-Stage-Executing`) and after (`On-Stage-Executed`) their pipeline stage.</span></span> <span data-ttu-id="42f92-519">La chiamata di `OnActionExecuting`, ad esempio, avviene prima della chiamata del metodo di azione.</span><span class="sxs-lookup"><span data-stu-id="42f92-519">For example, `OnActionExecuting` is called before the action method is called.</span></span> <span data-ttu-id="42f92-520">La chiamata di `OnActionExecuted` avviene dopo l'esecuzione del metodo di azione.</span><span class="sxs-lookup"><span data-stu-id="42f92-520">`OnActionExecuted` is called after the action method returns.</span></span>

[!code-csharp[](./filters/sample/FiltersSample/Filters/MySampleActionFilter.cs?name=snippet_ActionFilter)]

<span data-ttu-id="42f92-521">I filtri asincroni definiscono un metodo `On-Stage-ExecutionAsync`:</span><span class="sxs-lookup"><span data-stu-id="42f92-521">Asynchronous filters define an `On-Stage-ExecutionAsync` method:</span></span>

[!code-csharp[](./filters/sample/FiltersSample/Filters/SampleAsyncActionFilter.cs?name=snippet)]

<span data-ttu-id="42f92-522">Nel codice precedente `SampleAsyncActionFilter` ha un oggetto <xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutionDelegate> (`next`) che esegue il metodo di azione.</span><span class="sxs-lookup"><span data-stu-id="42f92-522">In the preceding code, the `SampleAsyncActionFilter` has an <xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutionDelegate> (`next`)  that executes the action method.</span></span>  <span data-ttu-id="42f92-523">Ognuno dei metodi `On-Stage-ExecutionAsync` accetta un oggetto `FilterType-ExecutionDelegate` che esegue la fase della pipeline del filtro.</span><span class="sxs-lookup"><span data-stu-id="42f92-523">Each of the `On-Stage-ExecutionAsync` methods take a `FilterType-ExecutionDelegate` that executes the filter's pipeline stage.</span></span>

### <a name="multiple-filter-stages"></a><span data-ttu-id="42f92-524">Fasi di filtro multiple</span><span class="sxs-lookup"><span data-stu-id="42f92-524">Multiple filter stages</span></span>

<span data-ttu-id="42f92-525">È possibile implementare interfacce per più fasi di filtro in una singola classe.</span><span class="sxs-lookup"><span data-stu-id="42f92-525">Interfaces for multiple filter stages can be implemented in a single class.</span></span> <span data-ttu-id="42f92-526">Ad esempio, la classe <xref:Microsoft.AspNetCore.Mvc.Filters.ActionFilterAttribute> implementa `IActionFilter`, `IResultFilter` e i relativi equivalenti asincroni.</span><span class="sxs-lookup"><span data-stu-id="42f92-526">For example, the <xref:Microsoft.AspNetCore.Mvc.Filters.ActionFilterAttribute> class implements `IActionFilter`, `IResultFilter`, and their async equivalents.</span></span>

<span data-ttu-id="42f92-527">Implementare la versione sincrona **oppure** la versione asincrona di un'interfaccia di filtro, **non** entrambe.</span><span class="sxs-lookup"><span data-stu-id="42f92-527">Implement **either** the synchronous or the async version of a filter interface, **not** both.</span></span> <span data-ttu-id="42f92-528">Il runtime controlla per prima cosa se il filtro implementa l'interfaccia asincrona e, in tal caso, la chiama.</span><span class="sxs-lookup"><span data-stu-id="42f92-528">The runtime checks first to see if the filter implements the async interface, and if so, it calls that.</span></span> <span data-ttu-id="42f92-529">In caso contrario, chiama i metodi dell'interfaccia sincrona.</span><span class="sxs-lookup"><span data-stu-id="42f92-529">If not, it calls the synchronous interface's method(s).</span></span> <span data-ttu-id="42f92-530">Se in una classe sono implementate entrambe le interfacce, sincrona e asincrona, viene chiamato solo il metodo asincrono.</span><span class="sxs-lookup"><span data-stu-id="42f92-530">If both asynchronous and synchronous interfaces are implemented in one class, only the async method is called.</span></span> <span data-ttu-id="42f92-531">Quando si usano classi astratte come <xref:Microsoft.AspNetCore.Mvc.Filters.ActionFilterAttribute> eseguire l'override solo dei metodi sincroni o del metodo asincrono per ogni tipo di filtro.</span><span class="sxs-lookup"><span data-stu-id="42f92-531">When using abstract classes like <xref:Microsoft.AspNetCore.Mvc.Filters.ActionFilterAttribute> override only the synchronous methods or the async method for each filter type.</span></span>

### <a name="built-in-filter-attributes"></a><span data-ttu-id="42f92-532">Attributi filtro predefiniti</span><span class="sxs-lookup"><span data-stu-id="42f92-532">Built-in filter attributes</span></span>

<span data-ttu-id="42f92-533">ASP.NET Core include filtri predefiniti basati su attributi che è possibile impostare come sottoclasse e personalizzare.</span><span class="sxs-lookup"><span data-stu-id="42f92-533">ASP.NET Core includes built-in attribute-based filters that can be subclassed and customized.</span></span> <span data-ttu-id="42f92-534">Il filtro dei risultati seguente, ad esempio, aggiunge un'intestazione alla risposta:</span><span class="sxs-lookup"><span data-stu-id="42f92-534">For example, the following result filter adds a header to the response:</span></span>

<a name="add-header-attribute"></a>

[!code-csharp[](./filters/sample/FiltersSample/Filters/AddHeaderAttribute.cs?name=snippet)]

<span data-ttu-id="42f92-535">Gli attributi consentono ai filtri di accettare argomenti, come illustrato nell'esempio precedente.</span><span class="sxs-lookup"><span data-stu-id="42f92-535">Attributes allow filters to accept arguments, as shown in the preceding example.</span></span> <span data-ttu-id="42f92-536">Applicare `AddHeaderAttribute` a un controller o a un metodo di azione e specificare il nome e il valore dell'intestazione HTTP:</span><span class="sxs-lookup"><span data-stu-id="42f92-536">Apply the `AddHeaderAttribute` to a controller or action method and specify the name and value of the HTTP header:</span></span>

[!code-csharp[](./filters/sample/FiltersSample/Controllers/SampleController.cs?name=snippet_AddHeader&highlight=1)]

<!-- `https://localhost:5001/Sample` -->

<span data-ttu-id="42f92-537">Molte delle interfacce del filtro presentano attributi corrispondenti che possono essere usati come classi di base per implementazioni personalizzate.</span><span class="sxs-lookup"><span data-stu-id="42f92-537">Several of the filter interfaces have corresponding attributes that can be used as base classes for custom implementations.</span></span>

<span data-ttu-id="42f92-538">Attributi dei filtri:</span><span class="sxs-lookup"><span data-stu-id="42f92-538">Filter attributes:</span></span>

* <xref:Microsoft.AspNetCore.Mvc.Filters.ActionFilterAttribute>
* <xref:Microsoft.AspNetCore.Mvc.Filters.ExceptionFilterAttribute>
* <xref:Microsoft.AspNetCore.Mvc.Filters.ResultFilterAttribute>
* <xref:Microsoft.AspNetCore.Mvc.FormatFilterAttribute>
* <xref:Microsoft.AspNetCore.Mvc.ServiceFilterAttribute>
* <xref:Microsoft.AspNetCore.Mvc.TypeFilterAttribute>

## <a name="filter-scopes-and-order-of-execution"></a><span data-ttu-id="42f92-539">Ambiti dei filtri e ordine di esecuzione</span><span class="sxs-lookup"><span data-stu-id="42f92-539">Filter scopes and order of execution</span></span>

<span data-ttu-id="42f92-540">È possibile aggiungere un filtro alla pipeline in uno dei tre *ambiti*:</span><span class="sxs-lookup"><span data-stu-id="42f92-540">A filter can be added to the pipeline at one of three *scopes*:</span></span>

* <span data-ttu-id="42f92-541">Usando un attributo di un'azione.</span><span class="sxs-lookup"><span data-stu-id="42f92-541">Using an attribute on an action.</span></span>
* <span data-ttu-id="42f92-542">Usando un attributo di un controller.</span><span class="sxs-lookup"><span data-stu-id="42f92-542">Using an attribute on a controller.</span></span>
* <span data-ttu-id="42f92-543">A livello globale per tutti i controller e le azioni come illustrato nel codice seguente:</span><span class="sxs-lookup"><span data-stu-id="42f92-543">Globally for all controllers and actions as shown in the following code:</span></span>

[!code-csharp[](./filters/sample/FiltersSample/StartupGF.cs?name=snippet_ConfigureServices)]

<span data-ttu-id="42f92-544">Il codice precedente aggiunge tre filtri a livello globale tramite la raccolta [MvcOptions.Filters](xref:Microsoft.AspNetCore.Mvc.MvcOptions.Filters).</span><span class="sxs-lookup"><span data-stu-id="42f92-544">The preceding code adds three filters globally using the [MvcOptions.Filters](xref:Microsoft.AspNetCore.Mvc.MvcOptions.Filters) collection.</span></span>

### <a name="default-order-of-execution"></a><span data-ttu-id="42f92-545">Ordine di esecuzione predefinito</span><span class="sxs-lookup"><span data-stu-id="42f92-545">Default order of execution</span></span>

<span data-ttu-id="42f92-546">Quando sono presenti più filtri *dello stesso tipo*, scope determina l'ordine predefinito di esecuzione del filtro.</span><span class="sxs-lookup"><span data-stu-id="42f92-546">When there are multiple filters *of the same type*, scope determines the default order of filter execution.</span></span>  <span data-ttu-id="42f92-547">Filtri globali circondano i filtri di classe.</span><span class="sxs-lookup"><span data-stu-id="42f92-547">Global filters surround class filters.</span></span> <span data-ttu-id="42f92-548">Filtri di classe filtri di metodo Racchiudi.</span><span class="sxs-lookup"><span data-stu-id="42f92-548">Class filters surround method filters.</span></span>

<span data-ttu-id="42f92-549">Come risultato dell'annidamento dei filtri, il codice *after* dei filtri viene eseguito in ordine inverso rispetto al codice *before*.</span><span class="sxs-lookup"><span data-stu-id="42f92-549">As a result of filter nesting, the *after* code of filters runs in the reverse order of the *before* code.</span></span> <span data-ttu-id="42f92-550">Sequenza di filtro:</span><span class="sxs-lookup"><span data-stu-id="42f92-550">The filter sequence:</span></span>

* <span data-ttu-id="42f92-551">Codice *before* dei filtri globali.</span><span class="sxs-lookup"><span data-stu-id="42f92-551">The *before* code of global filters.</span></span>
  * <span data-ttu-id="42f92-552">Codice *before* dei filtri del controller.</span><span class="sxs-lookup"><span data-stu-id="42f92-552">The *before* code of controller filters.</span></span>
    * <span data-ttu-id="42f92-553">Codice *before* dei filtri del metodo di azione.</span><span class="sxs-lookup"><span data-stu-id="42f92-553">The *before* code of action method filters.</span></span>
    * <span data-ttu-id="42f92-554">Codice *after* dei filtri del metodo di azione.</span><span class="sxs-lookup"><span data-stu-id="42f92-554">The *after* code of action method filters.</span></span>
  * <span data-ttu-id="42f92-555">Codice *after* dei filtri del controller.</span><span class="sxs-lookup"><span data-stu-id="42f92-555">The *after* code of controller filters.</span></span>
* <span data-ttu-id="42f92-556">Codice *after* dei filtri globali.</span><span class="sxs-lookup"><span data-stu-id="42f92-556">The *after* code of global filters.</span></span>
  
<span data-ttu-id="42f92-557">L'esempio seguente illustra l'ordine in cui i metodi dei filtri vengono chiamati per i filtri di azione sincroni.</span><span class="sxs-lookup"><span data-stu-id="42f92-557">The following example that illustrates the order in which filter methods are called for synchronous action filters.</span></span>

| <span data-ttu-id="42f92-558">Sequenza</span><span class="sxs-lookup"><span data-stu-id="42f92-558">Sequence</span></span> | <span data-ttu-id="42f92-559">Ambito del filtro</span><span class="sxs-lookup"><span data-stu-id="42f92-559">Filter scope</span></span> | <span data-ttu-id="42f92-560">Metodo del filtro</span><span class="sxs-lookup"><span data-stu-id="42f92-560">Filter method</span></span> |
|:--------:|:------------:|:-------------:|
| <span data-ttu-id="42f92-561">1</span><span class="sxs-lookup"><span data-stu-id="42f92-561">1</span></span> | <span data-ttu-id="42f92-562">Globale</span><span class="sxs-lookup"><span data-stu-id="42f92-562">Global</span></span> | `OnActionExecuting` |
| <span data-ttu-id="42f92-563">2</span><span class="sxs-lookup"><span data-stu-id="42f92-563">2</span></span> | <span data-ttu-id="42f92-564">Controller</span><span class="sxs-lookup"><span data-stu-id="42f92-564">Controller</span></span> | `OnActionExecuting` |
| <span data-ttu-id="42f92-565">3</span><span class="sxs-lookup"><span data-stu-id="42f92-565">3</span></span> | <span data-ttu-id="42f92-566">Metodo</span><span class="sxs-lookup"><span data-stu-id="42f92-566">Method</span></span> | `OnActionExecuting` |
| <span data-ttu-id="42f92-567">4</span><span class="sxs-lookup"><span data-stu-id="42f92-567">4</span></span> | <span data-ttu-id="42f92-568">Metodo</span><span class="sxs-lookup"><span data-stu-id="42f92-568">Method</span></span> | `OnActionExecuted` |
| <span data-ttu-id="42f92-569">5</span><span class="sxs-lookup"><span data-stu-id="42f92-569">5</span></span> | <span data-ttu-id="42f92-570">Controller</span><span class="sxs-lookup"><span data-stu-id="42f92-570">Controller</span></span> | `OnActionExecuted` |
| <span data-ttu-id="42f92-571">6</span><span class="sxs-lookup"><span data-stu-id="42f92-571">6</span></span> | <span data-ttu-id="42f92-572">Globale</span><span class="sxs-lookup"><span data-stu-id="42f92-572">Global</span></span> | `OnActionExecuted` |

<span data-ttu-id="42f92-573">Questa sequenza mostra che:</span><span class="sxs-lookup"><span data-stu-id="42f92-573">This sequence shows:</span></span>

* <span data-ttu-id="42f92-574">Il filtro del metodo è annidato all'interno del filtro del controller.</span><span class="sxs-lookup"><span data-stu-id="42f92-574">The method filter is nested within the controller filter.</span></span>
* <span data-ttu-id="42f92-575">Il filtro del controller è annidato all'interno del filtro globale.</span><span class="sxs-lookup"><span data-stu-id="42f92-575">The controller filter is nested within the global filter.</span></span>

### <a name="controller-and-razor-page-level-filters"></a><span data-ttu-id="42f92-576">Filtri a Razor livello di pagina e controller</span><span class="sxs-lookup"><span data-stu-id="42f92-576">Controller and Razor Page level filters</span></span>

<span data-ttu-id="42f92-577">Ogni controller che eredita dalla classe di base <xref:Microsoft.AspNetCore.Mvc.Controller> include i metodi [Controller.OnActionExecuting](xref:Microsoft.AspNetCore.Mvc.Controller.OnActionExecuting*), [Controller.OnActionExecutionAsync](xref:Microsoft.AspNetCore.Mvc.Controller.OnActionExecutionAsync*) e [Controller.OnActionExecuted](xref:Microsoft.AspNetCore.Mvc.Controller.OnActionExecuted*)
`OnActionExecuted`.</span><span class="sxs-lookup"><span data-stu-id="42f92-577">Every controller that inherits from the <xref:Microsoft.AspNetCore.Mvc.Controller> base class includes [Controller.OnActionExecuting](xref:Microsoft.AspNetCore.Mvc.Controller.OnActionExecuting*),  [Controller.OnActionExecutionAsync](xref:Microsoft.AspNetCore.Mvc.Controller.OnActionExecutionAsync*), and [Controller.OnActionExecuted](xref:Microsoft.AspNetCore.Mvc.Controller.OnActionExecuted*)
`OnActionExecuted` methods.</span></span> <span data-ttu-id="42f92-578">Questi metodi:</span><span class="sxs-lookup"><span data-stu-id="42f92-578">These methods:</span></span>

* <span data-ttu-id="42f92-579">Eseguono il wrapping dei filtri che vengono eseguiti per una determinata azione.</span><span class="sxs-lookup"><span data-stu-id="42f92-579">Wrap the filters that run for a given action.</span></span>
* <span data-ttu-id="42f92-580">La chiamata di `OnActionExecuting` avviene prima di qualsiasi filtro dell'azione.</span><span class="sxs-lookup"><span data-stu-id="42f92-580">`OnActionExecuting` is called before any of the action's filters.</span></span>
* <span data-ttu-id="42f92-581">La chiamata di `OnActionExecuted` avviene dopo tutti i filtri dell'azione.</span><span class="sxs-lookup"><span data-stu-id="42f92-581">`OnActionExecuted` is called after all of the action filters.</span></span>
* <span data-ttu-id="42f92-582">La chiamata di `OnActionExecutionAsync` avviene prima di qualsiasi filtro dell'azione.</span><span class="sxs-lookup"><span data-stu-id="42f92-582">`OnActionExecutionAsync` is called before any of the action's filters.</span></span> <span data-ttu-id="42f92-583">Il codice del filtro dopo `next` viene eseguito dopo il metodo di azione.</span><span class="sxs-lookup"><span data-stu-id="42f92-583">Code in the filter after `next` runs after the action method.</span></span>

<span data-ttu-id="42f92-584">Ad esempio, l'applicazione di `MySampleActionFilter` nell'esempio scaricato avviene a livello globale all'avvio.</span><span class="sxs-lookup"><span data-stu-id="42f92-584">For example, in the download sample, `MySampleActionFilter` is applied globally in startup.</span></span>

<span data-ttu-id="42f92-585">`TestController`:</span><span class="sxs-lookup"><span data-stu-id="42f92-585">The `TestController`:</span></span>

* <span data-ttu-id="42f92-586">Applica `SampleActionFilterAttribute` ( `[SampleActionFilter]` ) all' `FilterTest2` azione.</span><span class="sxs-lookup"><span data-stu-id="42f92-586">Applies the `SampleActionFilterAttribute` (`[SampleActionFilter]`) to the `FilterTest2` action.</span></span>
* <span data-ttu-id="42f92-587">Esegue l'override di `OnActionExecuting` e `OnActionExecuted`.</span><span class="sxs-lookup"><span data-stu-id="42f92-587">Overrides `OnActionExecuting` and `OnActionExecuted`.</span></span>

[!code-csharp[](./filters/sample/FiltersSample/Controllers/TestController.cs?name=snippet)]

<span data-ttu-id="42f92-588">Passando a `https://localhost:5001/Test/FilterTest2`, viene eseguito il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="42f92-588">Navigating to `https://localhost:5001/Test/FilterTest2` runs the following code:</span></span>

* `TestController.OnActionExecuting`
  * `MySampleActionFilter.OnActionExecuting`
    * `SampleActionFilterAttribute.OnActionExecuting`
      * `TestController.FilterTest2`
    * `SampleActionFilterAttribute.OnActionExecuted`
  * `MySampleActionFilter.OnActionExecuted`
* `TestController.OnActionExecuted`

<span data-ttu-id="42f92-589">Per le Razor pagine, vedere [implementare Razor filtri di pagina eseguendo l'override dei metodi di filtro](xref:razor-pages/filter#implement-razor-page-filters-by-overriding-filter-methods).</span><span class="sxs-lookup"><span data-stu-id="42f92-589">For Razor Pages, see [Implement Razor Page filters by overriding filter methods](xref:razor-pages/filter#implement-razor-page-filters-by-overriding-filter-methods).</span></span>

### <a name="overriding-the-default-order"></a><span data-ttu-id="42f92-590">Override dell'ordine predefinito</span><span class="sxs-lookup"><span data-stu-id="42f92-590">Overriding the default order</span></span>

<span data-ttu-id="42f92-591">È possibile eseguire l'override della sequenza di esecuzione predefinita implementando <xref:Microsoft.AspNetCore.Mvc.Filters.IOrderedFilter>.</span><span class="sxs-lookup"><span data-stu-id="42f92-591">The default sequence of execution can be overridden by implementing <xref:Microsoft.AspNetCore.Mvc.Filters.IOrderedFilter>.</span></span> <span data-ttu-id="42f92-592">`IOrderedFilter` espone la proprietà <xref:Microsoft.AspNetCore.Mvc.Filters.IOrderedFilter.Order>, che ha la precedenza sull'ambito per determinare l'ordine di esecuzione.</span><span class="sxs-lookup"><span data-stu-id="42f92-592">`IOrderedFilter` exposes the <xref:Microsoft.AspNetCore.Mvc.Filters.IOrderedFilter.Order> property that takes precedence over scope to determine the order of execution.</span></span> <span data-ttu-id="42f92-593">Un filtro con un valore di `Order` inferiore:</span><span class="sxs-lookup"><span data-stu-id="42f92-593">A filter with a lower `Order` value:</span></span>

* <span data-ttu-id="42f92-594">Esegue il codice *before* prima di quello di un filtro con un valore di `Order` più alto.</span><span class="sxs-lookup"><span data-stu-id="42f92-594">Runs the *before* code before that of a filter with a higher value of `Order`.</span></span>
* <span data-ttu-id="42f92-595">Esegue il codice *after* dopo quello di un filtro con un valore di `Order` più alto.</span><span class="sxs-lookup"><span data-stu-id="42f92-595">Runs the *after* code after that of a filter with a higher `Order` value.</span></span>

<span data-ttu-id="42f92-596">La proprietà `Order` può essere impostata con un parametro del costruttore:</span><span class="sxs-lookup"><span data-stu-id="42f92-596">The `Order` property can be set with a constructor parameter:</span></span>

```csharp
[MyFilter(Name = "Controller Level Attribute", Order=1)]
```

<span data-ttu-id="42f92-597">Prendere in considerazione gli stessi tre filtri di azione illustrati nell'esempio precedente.</span><span class="sxs-lookup"><span data-stu-id="42f92-597">Consider the same 3 action filters shown in the preceding example.</span></span> <span data-ttu-id="42f92-598">Se la proprietà `Order` del controller e dei filtri globali è impostata, rispettivamente, su 1 e su 2, l'ordine di esecuzione viene invertito.</span><span class="sxs-lookup"><span data-stu-id="42f92-598">If the `Order` property of the controller and global filters is set to 1 and 2 respectively, the order of execution is reversed.</span></span>

| <span data-ttu-id="42f92-599">Sequenza</span><span class="sxs-lookup"><span data-stu-id="42f92-599">Sequence</span></span> | <span data-ttu-id="42f92-600">Ambito del filtro</span><span class="sxs-lookup"><span data-stu-id="42f92-600">Filter scope</span></span> | <span data-ttu-id="42f92-601">Proprietà`Order`</span><span class="sxs-lookup"><span data-stu-id="42f92-601">`Order` property</span></span> | <span data-ttu-id="42f92-602">Metodo del filtro</span><span class="sxs-lookup"><span data-stu-id="42f92-602">Filter method</span></span> |
|:--------:|:------------:|:-----------------:|:-------------:|
| <span data-ttu-id="42f92-603">1</span><span class="sxs-lookup"><span data-stu-id="42f92-603">1</span></span> | <span data-ttu-id="42f92-604">Metodo</span><span class="sxs-lookup"><span data-stu-id="42f92-604">Method</span></span> | <span data-ttu-id="42f92-605">0</span><span class="sxs-lookup"><span data-stu-id="42f92-605">0</span></span> | `OnActionExecuting` |
| <span data-ttu-id="42f92-606">2</span><span class="sxs-lookup"><span data-stu-id="42f92-606">2</span></span> | <span data-ttu-id="42f92-607">Controller</span><span class="sxs-lookup"><span data-stu-id="42f92-607">Controller</span></span> | <span data-ttu-id="42f92-608">1</span><span class="sxs-lookup"><span data-stu-id="42f92-608">1</span></span>  | `OnActionExecuting` |
| <span data-ttu-id="42f92-609">3</span><span class="sxs-lookup"><span data-stu-id="42f92-609">3</span></span> | <span data-ttu-id="42f92-610">Globale</span><span class="sxs-lookup"><span data-stu-id="42f92-610">Global</span></span> | <span data-ttu-id="42f92-611">2</span><span class="sxs-lookup"><span data-stu-id="42f92-611">2</span></span>  | `OnActionExecuting` |
| <span data-ttu-id="42f92-612">4</span><span class="sxs-lookup"><span data-stu-id="42f92-612">4</span></span> | <span data-ttu-id="42f92-613">Globale</span><span class="sxs-lookup"><span data-stu-id="42f92-613">Global</span></span> | <span data-ttu-id="42f92-614">2</span><span class="sxs-lookup"><span data-stu-id="42f92-614">2</span></span>  | `OnActionExecuted` |
| <span data-ttu-id="42f92-615">5</span><span class="sxs-lookup"><span data-stu-id="42f92-615">5</span></span> | <span data-ttu-id="42f92-616">Controller</span><span class="sxs-lookup"><span data-stu-id="42f92-616">Controller</span></span> | <span data-ttu-id="42f92-617">1</span><span class="sxs-lookup"><span data-stu-id="42f92-617">1</span></span>  | `OnActionExecuted` |
| <span data-ttu-id="42f92-618">6</span><span class="sxs-lookup"><span data-stu-id="42f92-618">6</span></span> | <span data-ttu-id="42f92-619">Metodo</span><span class="sxs-lookup"><span data-stu-id="42f92-619">Method</span></span> | <span data-ttu-id="42f92-620">0</span><span class="sxs-lookup"><span data-stu-id="42f92-620">0</span></span>  | `OnActionExecuted` |

<span data-ttu-id="42f92-621">La proprietà `Order` ha la precedenza sull'ambito nel determinare l'ordine di esecuzione dei filtri.</span><span class="sxs-lookup"><span data-stu-id="42f92-621">The `Order` property overrides scope when determining the order in which filters run.</span></span> <span data-ttu-id="42f92-622">I filtri vengono ordinati prima in base all'ordine, poi viene usato l'ambito per interrompere i collegamenti.</span><span class="sxs-lookup"><span data-stu-id="42f92-622">Filters are sorted first by order, then scope is used to break ties.</span></span> <span data-ttu-id="42f92-623">Tutti i filtri predefiniti implementano `IOrderedFilter` e impostano il valore predefinito di `Order` su 0.</span><span class="sxs-lookup"><span data-stu-id="42f92-623">All of the built-in filters implement `IOrderedFilter` and set the default `Order` value to 0.</span></span> <span data-ttu-id="42f92-624">Per i filtri predefiniti, l'ambito determina l'ordine, a meno che non si imposti `Order` su un valore diverso da zero.</span><span class="sxs-lookup"><span data-stu-id="42f92-624">For built-in filters, scope determines order unless `Order` is set to a non-zero value.</span></span>

## <a name="cancellation-and-short-circuiting"></a><span data-ttu-id="42f92-625">Annullamento e corto circuito</span><span class="sxs-lookup"><span data-stu-id="42f92-625">Cancellation and short-circuiting</span></span>

<span data-ttu-id="42f92-626">È possibile causare il corto circuito della pipeline di filtro impostando la proprietà <xref:Microsoft.AspNetCore.Mvc.Filters.ResourceExecutingContext.Result> sul parametro <xref:Microsoft.AspNetCore.Mvc.Filters.ResourceExecutingContext> fornito al metodo di filtro.</span><span class="sxs-lookup"><span data-stu-id="42f92-626">The filter pipeline can be short-circuited by setting the <xref:Microsoft.AspNetCore.Mvc.Filters.ResourceExecutingContext.Result> property on the <xref:Microsoft.AspNetCore.Mvc.Filters.ResourceExecutingContext> parameter provided to the filter method.</span></span> <span data-ttu-id="42f92-627">Ad esempio, il filtro di risorse seguente impedisce l'esecuzione del resto della pipeline:</span><span class="sxs-lookup"><span data-stu-id="42f92-627">For instance, the following Resource filter prevents the rest of the pipeline from executing:</span></span>

<a name="short-circuiting-resource-filter"></a>

[!code-csharp[](./filters/sample/FiltersSample/Filters/ShortCircuitingResourceFilterAttribute.cs?name=snippet)]

<span data-ttu-id="42f92-628">Nel codice seguente sia il filtro `ShortCircuitingResourceFilter` che il filtro `AddHeader` hanno come destinazione il metodo di azione `SomeResource`.</span><span class="sxs-lookup"><span data-stu-id="42f92-628">In the following code, both the `ShortCircuitingResourceFilter` and the `AddHeader` filter target the `SomeResource` action method.</span></span> <span data-ttu-id="42f92-629">`ShortCircuitingResourceFilter`:</span><span class="sxs-lookup"><span data-stu-id="42f92-629">The `ShortCircuitingResourceFilter`:</span></span>

* <span data-ttu-id="42f92-630">Viene eseguito per primo perché è un filtro risorsa e `AddHeader` è un filtro azione.</span><span class="sxs-lookup"><span data-stu-id="42f92-630">Runs first, because it's a Resource Filter and `AddHeader` is an Action Filter.</span></span>
* <span data-ttu-id="42f92-631">Blocca il resto della pipeline.</span><span class="sxs-lookup"><span data-stu-id="42f92-631">Short-circuits the rest of the pipeline.</span></span>

<span data-ttu-id="42f92-632">Pertanto il filtro `AddHeader` non viene mai eseguito per l'azione `SomeResource`.</span><span class="sxs-lookup"><span data-stu-id="42f92-632">Therefore the `AddHeader` filter never runs for the `SomeResource` action.</span></span> <span data-ttu-id="42f92-633">Questo comportamento sarebbe lo stesso se entrambi i filtri venissero applicati a livello di metodo di azione, a condizione che il filtro `ShortCircuitingResourceFilter` venga eseguito prima.</span><span class="sxs-lookup"><span data-stu-id="42f92-633">This behavior would be the same if both filters were applied at the action method level, provided the `ShortCircuitingResourceFilter` ran first.</span></span> <span data-ttu-id="42f92-634">`ShortCircuitingResourceFilter` viene eseguito per primo per il tipo di filtro o per l'uso esplicito della proprietà `Order`.</span><span class="sxs-lookup"><span data-stu-id="42f92-634">The `ShortCircuitingResourceFilter` runs first because of its filter type, or by explicit use of `Order` property.</span></span>

[!code-csharp[](./filters/sample/FiltersSample/Controllers/SampleController.cs?name=snippet_AddHeader&highlight=1,9)]

## <a name="dependency-injection"></a><span data-ttu-id="42f92-635">Inserimento di dipendenze</span><span class="sxs-lookup"><span data-stu-id="42f92-635">Dependency injection</span></span>

<span data-ttu-id="42f92-636">È possibile aggiungere filtri per tipo o per istanza.</span><span class="sxs-lookup"><span data-stu-id="42f92-636">Filters can be added by type or by instance.</span></span> <span data-ttu-id="42f92-637">Se viene aggiunta un'istanza, tale istanza viene usata per ogni richiesta.</span><span class="sxs-lookup"><span data-stu-id="42f92-637">If an instance is added, that instance is used for every request.</span></span> <span data-ttu-id="42f92-638">Se viene aggiunto un tipo, il filtro è attivato dal tipo.</span><span class="sxs-lookup"><span data-stu-id="42f92-638">If a type is added, it's type-activated.</span></span> <span data-ttu-id="42f92-639">Un filtro attivato dal tipo comporta:</span><span class="sxs-lookup"><span data-stu-id="42f92-639">A type-activated filter means:</span></span>

* <span data-ttu-id="42f92-640">La creazione di un'istanza per ogni richiesta.</span><span class="sxs-lookup"><span data-stu-id="42f92-640">An instance is created for each request.</span></span>
* <span data-ttu-id="42f92-641">Il popolamento di tutte le dipendenze del costruttore tramite [inserimento delle dipendenze](xref:fundamentals/dependency-injection).</span><span class="sxs-lookup"><span data-stu-id="42f92-641">Any constructor dependencies are populated by [dependency injection](xref:fundamentals/dependency-injection) (DI).</span></span>

<span data-ttu-id="42f92-642">I filtri implementati come attributi e aggiunti direttamente alle classi controller o ai metodi di azione non possono avere dipendenze costruttore specificate dall'[inserimento di dipendenze](xref:fundamentals/dependency-injection).</span><span class="sxs-lookup"><span data-stu-id="42f92-642">Filters that are implemented as attributes and added directly to controller classes or action methods cannot have constructor dependencies provided by [dependency injection](xref:fundamentals/dependency-injection) (DI).</span></span> <span data-ttu-id="42f92-643">Le dipendenze del costruttore non possono essere fornite tramite inserimento delle dipendenze in quanto:</span><span class="sxs-lookup"><span data-stu-id="42f92-643">Constructor dependencies cannot be provided by DI because:</span></span>

* <span data-ttu-id="42f92-644">I parametri del costruttore devono essere forniti agli attributi nella posizione in cui vengono applicati.</span><span class="sxs-lookup"><span data-stu-id="42f92-644">Attributes must have their constructor parameters supplied where they're applied.</span></span> 
* <span data-ttu-id="42f92-645">Questa è una limitazione del funzionamento degli attributi.</span><span class="sxs-lookup"><span data-stu-id="42f92-645">This is a limitation of how attributes work.</span></span>

<span data-ttu-id="42f92-646">I filtri seguenti supportano le dipendenze del costruttore fornite dall'inserimento delle dipendenze:</span><span class="sxs-lookup"><span data-stu-id="42f92-646">The following filters support constructor dependencies provided from DI:</span></span>

* <xref:Microsoft.AspNetCore.Mvc.ServiceFilterAttribute>
* <xref:Microsoft.AspNetCore.Mvc.TypeFilterAttribute>
* <span data-ttu-id="42f92-647"><xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory> implementato nell'attributo.</span><span class="sxs-lookup"><span data-stu-id="42f92-647"><xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory> implemented on the attribute.</span></span>

<span data-ttu-id="42f92-648">I filtri precedenti possono essere applicati a un controller o a un metodo di azione:</span><span class="sxs-lookup"><span data-stu-id="42f92-648">The preceding filters can be applied to a controller or action method:</span></span>

<span data-ttu-id="42f92-649">I logger sono disponibili tramite l'inserimento delle dipendenze.</span><span class="sxs-lookup"><span data-stu-id="42f92-649">Loggers are available from DI.</span></span> <span data-ttu-id="42f92-650">Evitare tuttavia di creare e usare filtri esclusivamente per scopi di registrazione.</span><span class="sxs-lookup"><span data-stu-id="42f92-650">However, avoid creating and using filters purely for logging purposes.</span></span> <span data-ttu-id="42f92-651">La funzionalità di [registrazione del framework predefinita](xref:fundamentals/logging/index) in genere fornisce il necessario per la registrazione.</span><span class="sxs-lookup"><span data-stu-id="42f92-651">The [built-in framework logging](xref:fundamentals/logging/index) typically provides what's needed for logging.</span></span> <span data-ttu-id="42f92-652">La registrazione aggiunta ai filtri:</span><span class="sxs-lookup"><span data-stu-id="42f92-652">Logging added to filters:</span></span>

* <span data-ttu-id="42f92-653">Deve concentrarsi su problemi del dominio di business o comportamenti specifici del filtro.</span><span class="sxs-lookup"><span data-stu-id="42f92-653">Should focus on business domain concerns or behavior specific to the filter.</span></span>
* <span data-ttu-id="42f92-654">**Non** deve registrare azioni o altri eventi del framework.</span><span class="sxs-lookup"><span data-stu-id="42f92-654">Should **not** log actions or other framework events.</span></span> <span data-ttu-id="42f92-655">I filtri predefiniti registrano azioni ed eventi del framework.</span><span class="sxs-lookup"><span data-stu-id="42f92-655">The built in filters log actions and framework events.</span></span>

### <a name="servicefilterattribute"></a><span data-ttu-id="42f92-656">ServiceFilterAttribute</span><span class="sxs-lookup"><span data-stu-id="42f92-656">ServiceFilterAttribute</span></span>

<span data-ttu-id="42f92-657">I tipi di implementazione del filtro di servizi sono registrati in `ConfigureServices`.</span><span class="sxs-lookup"><span data-stu-id="42f92-657">Service filter implementation types are registered in `ConfigureServices`.</span></span> <span data-ttu-id="42f92-658"><xref:Microsoft.AspNetCore.Mvc.ServiceFilterAttribute> recupera un'istanza del filtro dall'inserimento di dipendenze.</span><span class="sxs-lookup"><span data-stu-id="42f92-658">A <xref:Microsoft.AspNetCore.Mvc.ServiceFilterAttribute> retrieves an instance of the filter from DI.</span></span>

<span data-ttu-id="42f92-659">Il codice seguente illustra `AddHeaderResultServiceFilter`:</span><span class="sxs-lookup"><span data-stu-id="42f92-659">The following code shows the `AddHeaderResultServiceFilter`:</span></span>

[!code-csharp[](./filters/sample/FiltersSample/Filters/LoggingAddHeaderFilter.cs?name=snippet_ResultFilter)]

<span data-ttu-id="42f92-660">Nel codice seguente l'oggetto `AddHeaderResultServiceFilter` viene aggiunto al contenitore di inserimento delle dipendenze:</span><span class="sxs-lookup"><span data-stu-id="42f92-660">In the following code, `AddHeaderResultServiceFilter` is added to the DI container:</span></span>

[!code-csharp[](./filters/sample/FiltersSample/Startup.cs?name=snippet_ConfigureServices&highlight=4)]

<span data-ttu-id="42f92-661">Nel codice seguente l'attributo `ServiceFilter` recupera un'istanza del filtro `AddHeaderResultServiceFilter` dall'inserimento delle dipendenze:</span><span class="sxs-lookup"><span data-stu-id="42f92-661">In the following code, the `ServiceFilter` attribute retrieves an instance of the `AddHeaderResultServiceFilter` filter from DI:</span></span>

[!code-csharp[](./filters/sample/FiltersSample/Controllers/HomeController.cs?name=snippet_ServiceFilter&highlight=1)]

<span data-ttu-id="42f92-662">Quando si usa l'impostazione `ServiceFilterAttribute`, [ServiceFilterAttribute.IsReusable](xref:Microsoft.AspNetCore.Mvc.ServiceFilterAttribute.IsReusable):</span><span class="sxs-lookup"><span data-stu-id="42f92-662">When using `ServiceFilterAttribute`, setting [ServiceFilterAttribute.IsReusable](xref:Microsoft.AspNetCore.Mvc.ServiceFilterAttribute.IsReusable):</span></span>

* <span data-ttu-id="42f92-663">Indica che l'istanza del filtro *può* essere riutilizzata all'esterno dell'ambito della richiesta in cui è stata creata.</span><span class="sxs-lookup"><span data-stu-id="42f92-663">Provides a hint that the filter instance *may* be reused outside of the request scope it was created within.</span></span> <span data-ttu-id="42f92-664">Il runtime di ASP.NET Core non garantisce:</span><span class="sxs-lookup"><span data-stu-id="42f92-664">The ASP.NET Core runtime doesn't guarantee:</span></span>

  * <span data-ttu-id="42f92-665">Che venga creata una singola istanza del filtro.</span><span class="sxs-lookup"><span data-stu-id="42f92-665">That a single instance of the filter will be created.</span></span>
  * <span data-ttu-id="42f92-666">Che il filtro non verrà richiesto di nuovo dal contenitore di inserimento delle dipendenze in un momento successivo.</span><span class="sxs-lookup"><span data-stu-id="42f92-666">The filter will not be re-requested from the DI container at some later point.</span></span>

* <span data-ttu-id="42f92-667">Non si deve usare con un filtro che dipende da servizi con una durata diversa da singleton.</span><span class="sxs-lookup"><span data-stu-id="42f92-667">Should not be used with a filter that depends on services with a lifetime other than singleton.</span></span>

 <span data-ttu-id="42f92-668"><xref:Microsoft.AspNetCore.Mvc.ServiceFilterAttribute> implementa <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory>.</span><span class="sxs-lookup"><span data-stu-id="42f92-668"><xref:Microsoft.AspNetCore.Mvc.ServiceFilterAttribute> implements <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory>.</span></span> <span data-ttu-id="42f92-669">`IFilterFactory` espone il metodo <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory.CreateInstance*> per creare un'istanza <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterMetadata>.</span><span class="sxs-lookup"><span data-stu-id="42f92-669">`IFilterFactory` exposes the <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory.CreateInstance*> method for creating an <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterMetadata> instance.</span></span> <span data-ttu-id="42f92-670">`CreateInstance` carica il tipo specificato dall'inserimento delle dipendenze.</span><span class="sxs-lookup"><span data-stu-id="42f92-670">`CreateInstance` loads the specified type from DI.</span></span>

### <a name="typefilterattribute"></a><span data-ttu-id="42f92-671">TypeFilterAttribute</span><span class="sxs-lookup"><span data-stu-id="42f92-671">TypeFilterAttribute</span></span>

<span data-ttu-id="42f92-672"><xref:Microsoft.AspNetCore.Mvc.TypeFilterAttribute> è simile a <xref:Microsoft.AspNetCore.Mvc.ServiceFilterAttribute>, ma il relativo tipo non viene risolto direttamente dal contenitore dell'inserimento di dipendenze.</span><span class="sxs-lookup"><span data-stu-id="42f92-672"><xref:Microsoft.AspNetCore.Mvc.TypeFilterAttribute> is similar to <xref:Microsoft.AspNetCore.Mvc.ServiceFilterAttribute>, but its type isn't resolved directly from the DI container.</span></span> <span data-ttu-id="42f92-673">Viene creata un'istanza del tipo tramite <xref:Microsoft.Extensions.DependencyInjection.ObjectFactory?displayProperty=fullName>.</span><span class="sxs-lookup"><span data-stu-id="42f92-673">It instantiates the type by using <xref:Microsoft.Extensions.DependencyInjection.ObjectFactory?displayProperty=fullName>.</span></span>

<span data-ttu-id="42f92-674">Poiché i tipi `TypeFilterAttribute` non vengono risolti direttamente dal contenitore di inserimento delle dipendenze:</span><span class="sxs-lookup"><span data-stu-id="42f92-674">Because `TypeFilterAttribute` types aren't resolved directly from the DI container:</span></span>

* <span data-ttu-id="42f92-675">Non è necessario che i tipi a cui viene fatto riferimento tramite `TypeFilterAttribute` siano registrati nel contenitore di inserimento delle dipendenze.</span><span class="sxs-lookup"><span data-stu-id="42f92-675">Types that are referenced using the `TypeFilterAttribute` don't need to be registered with the DI container.</span></span>  <span data-ttu-id="42f92-676">Le loro dipendenze vengono soddisfatte dal contenitore di inserimento delle dipendenze.</span><span class="sxs-lookup"><span data-stu-id="42f92-676">They do have their dependencies fulfilled by the DI container.</span></span>
* <span data-ttu-id="42f92-677">In via facoltativa, `TypeFilterAttribute` può anche accettare gli argomenti del costruttore per il tipo.</span><span class="sxs-lookup"><span data-stu-id="42f92-677">`TypeFilterAttribute` can optionally accept constructor arguments for the type.</span></span>

<span data-ttu-id="42f92-678">Quando si usa l'impostazione `TypeFilterAttribute`, [TypeFilterAttribute.IsReusable](xref:Microsoft.AspNetCore.Mvc.TypeFilterAttribute.IsReusable):</span><span class="sxs-lookup"><span data-stu-id="42f92-678">When using `TypeFilterAttribute`, setting [TypeFilterAttribute.IsReusable](xref:Microsoft.AspNetCore.Mvc.TypeFilterAttribute.IsReusable):</span></span>
* <span data-ttu-id="42f92-679">Indica che l'istanza del filtro *può* essere riutilizzata all'esterno dell'ambito della richiesta in cui è stata creata.</span><span class="sxs-lookup"><span data-stu-id="42f92-679">Provides hint that the filter instance *may* be reused outside of the request scope it was created within.</span></span> <span data-ttu-id="42f92-680">Il runtime di ASP.NET Core non fornisce alcuna garanzia che venga creata una singola istanza del filtro.</span><span class="sxs-lookup"><span data-stu-id="42f92-680">The ASP.NET Core runtime provides no guarantees that a single instance of the filter will be created.</span></span>

* <span data-ttu-id="42f92-681">Non si deve usare con un filtro che dipende da servizi con una durata diversa da singleton.</span><span class="sxs-lookup"><span data-stu-id="42f92-681">Should not be used with a filter that depends on services with a lifetime other than singleton.</span></span>

<span data-ttu-id="42f92-682">L'esempio seguente illustra come passare argomenti a un tipo usando `TypeFilterAttribute`:</span><span class="sxs-lookup"><span data-stu-id="42f92-682">The following example shows how to pass arguments to a type using `TypeFilterAttribute`:</span></span>

[!code-csharp[](../../mvc/controllers/filters/sample/FiltersSample/Controllers/HomeController.cs?name=snippet_TypeFilter&highlight=1,2)]
[!code-csharp[](../../mvc/controllers/filters/sample/FiltersSample/Filters/LogConstantFilter.cs?name=snippet_TypeFilter_Implementation&highlight=6)]

<!-- 
https://localhost:5001/home/hi?name=joe
VS debug window shows 
FiltersSample.Filters.LogConstantFilter:Information: Method 'Hi' called
-->

## <a name="authorization-filters"></a><span data-ttu-id="42f92-683">Filtri autorizzazione</span><span class="sxs-lookup"><span data-stu-id="42f92-683">Authorization filters</span></span>

<span data-ttu-id="42f92-684">I filtri di autorizzazione:</span><span class="sxs-lookup"><span data-stu-id="42f92-684">Authorization filters:</span></span>

* <span data-ttu-id="42f92-685">Sono i primi filtri a essere eseguiti nella pipeline di filtro.</span><span class="sxs-lookup"><span data-stu-id="42f92-685">Are the first filters run in the filter pipeline.</span></span>
* <span data-ttu-id="42f92-686">Controllano l'accesso ai metodi di azione.</span><span class="sxs-lookup"><span data-stu-id="42f92-686">Control access to action methods.</span></span>
* <span data-ttu-id="42f92-687">Dispongono di un metodo precedente, ma non di un metodo successivo.</span><span class="sxs-lookup"><span data-stu-id="42f92-687">Have a before method, but no after method.</span></span>

<span data-ttu-id="42f92-688">I filtri di autorizzazione personalizzati richiedono un framework di autorizzazione personalizzato.</span><span class="sxs-lookup"><span data-stu-id="42f92-688">Custom authorization filters require a custom authorization framework.</span></span> <span data-ttu-id="42f92-689">È preferibile configurare criteri di autorizzazione o scrivere criteri di autorizzazione personalizzati piuttosto che scrivere un filtro personalizzato.</span><span class="sxs-lookup"><span data-stu-id="42f92-689">Prefer configuring the authorization policies or writing a custom authorization policy over writing a custom filter.</span></span> <span data-ttu-id="42f92-690">Il filtro di autorizzazione predefinito:</span><span class="sxs-lookup"><span data-stu-id="42f92-690">The built-in authorization filter:</span></span>

* <span data-ttu-id="42f92-691">Chiama il sistema di autorizzazione.</span><span class="sxs-lookup"><span data-stu-id="42f92-691">Calls the authorization system.</span></span>
* <span data-ttu-id="42f92-692">Non autorizza le richieste.</span><span class="sxs-lookup"><span data-stu-id="42f92-692">Does not authorize requests.</span></span>

<span data-ttu-id="42f92-693">**Non** generare eccezioni nei filtri di autorizzazione:</span><span class="sxs-lookup"><span data-stu-id="42f92-693">Do **not** throw exceptions within authorization filters:</span></span>

* <span data-ttu-id="42f92-694">L'eccezione non verrà gestita.</span><span class="sxs-lookup"><span data-stu-id="42f92-694">The exception will not be handled.</span></span>
* <span data-ttu-id="42f92-695">I filtri di eccezione non gestiranno l'eccezione.</span><span class="sxs-lookup"><span data-stu-id="42f92-695">Exception filters will not handle the exception.</span></span>

<span data-ttu-id="42f92-696">È consigliabile emettere una richiesta di verifica in caso di eccezione in un filtro di autorizzazione.</span><span class="sxs-lookup"><span data-stu-id="42f92-696">Consider issuing a challenge when an exception occurs in an authorization filter.</span></span>

<span data-ttu-id="42f92-697">Altre informazioni sull'[autorizzazione](xref:security/authorization/introduction).</span><span class="sxs-lookup"><span data-stu-id="42f92-697">Learn more about [Authorization](xref:security/authorization/introduction).</span></span>

## <a name="resource-filters"></a><span data-ttu-id="42f92-698">Filtri risorse</span><span class="sxs-lookup"><span data-stu-id="42f92-698">Resource filters</span></span>

<span data-ttu-id="42f92-699">I filtri di risorse:</span><span class="sxs-lookup"><span data-stu-id="42f92-699">Resource filters:</span></span>

* <span data-ttu-id="42f92-700">Implementano l'interfaccia <xref:Microsoft.AspNetCore.Mvc.Filters.IResourceFilter> o <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncResourceFilter>.</span><span class="sxs-lookup"><span data-stu-id="42f92-700">Implement either the <xref:Microsoft.AspNetCore.Mvc.Filters.IResourceFilter> or <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncResourceFilter> interface.</span></span>
* <span data-ttu-id="42f92-701">L'esecuzione racchiude la maggior parte della pipeline di filtro.</span><span class="sxs-lookup"><span data-stu-id="42f92-701">Execution wraps most of the filter pipeline.</span></span>
* <span data-ttu-id="42f92-702">Solo i [filtri di autorizzazione](#authorization-filters) vengono eseguiti prima dei filtri di risorse.</span><span class="sxs-lookup"><span data-stu-id="42f92-702">Only [Authorization filters](#authorization-filters) run before resource filters.</span></span>

<span data-ttu-id="42f92-703">I filtri di risorse sono utili per causare il corto circuito della maggior parte della pipeline.</span><span class="sxs-lookup"><span data-stu-id="42f92-703">Resource filters are useful to short-circuit most of the pipeline.</span></span> <span data-ttu-id="42f92-704">Un filtro di memorizzazione nella cache può, ad esempio, evitare il resto della pipeline in caso di riscontro nella cache.</span><span class="sxs-lookup"><span data-stu-id="42f92-704">For example, a caching filter can avoid the rest of the pipeline on a cache hit.</span></span>

<span data-ttu-id="42f92-705">Esempi di filtri di risorse:</span><span class="sxs-lookup"><span data-stu-id="42f92-705">Resource filter examples:</span></span>

* <span data-ttu-id="42f92-706">Il [filtro di risorse di corto circuito](#short-circuiting-resource-filter) illustrato in precedenza.</span><span class="sxs-lookup"><span data-stu-id="42f92-706">[The short-circuiting resource filter](#short-circuiting-resource-filter) shown previously.</span></span>
* <span data-ttu-id="42f92-707">[DisableFormValueModelBindingAttribute](https://github.com/aspnet/Entropy/blob/rel/2.0.0-preview2/samples/Mvc.FileUpload/Filters/DisableFormValueModelBindingAttribute.cs):</span><span class="sxs-lookup"><span data-stu-id="42f92-707">[DisableFormValueModelBindingAttribute](https://github.com/aspnet/Entropy/blob/rel/2.0.0-preview2/samples/Mvc.FileUpload/Filters/DisableFormValueModelBindingAttribute.cs):</span></span>

  * <span data-ttu-id="42f92-708">Impedisce all'associazione di modelli di accedere ai dati del modulo.</span><span class="sxs-lookup"><span data-stu-id="42f92-708">Prevents model binding from accessing the form data.</span></span>
  * <span data-ttu-id="42f92-709">Viene usato quando si caricano file di grandi dimensioni per impedire la lettura dei dati del modulo in memoria.</span><span class="sxs-lookup"><span data-stu-id="42f92-709">Used for large file uploads to prevent the form data from being read into memory.</span></span>

## <a name="action-filters"></a><span data-ttu-id="42f92-710">Filtri azione</span><span class="sxs-lookup"><span data-stu-id="42f92-710">Action filters</span></span>

> [!IMPORTANT]
> <span data-ttu-id="42f92-711">I filtri azione **non** si applicano alle Razor pagine.</span><span class="sxs-lookup"><span data-stu-id="42f92-711">Action filters do **not** apply to Razor Pages.</span></span> <span data-ttu-id="42f92-712">Razor Pages supporta <xref:Microsoft.AspNetCore.Mvc.Filters.IPageFilter> e <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncPageFilter> .</span><span class="sxs-lookup"><span data-stu-id="42f92-712">Razor Pages supports <xref:Microsoft.AspNetCore.Mvc.Filters.IPageFilter> and <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncPageFilter> .</span></span> <span data-ttu-id="42f92-713">Per ulteriori informazioni, vedere [metodi di filtro per le Razor pagine](xref:razor-pages/filter).</span><span class="sxs-lookup"><span data-stu-id="42f92-713">For more information, see [Filter methods for Razor Pages](xref:razor-pages/filter).</span></span>

<span data-ttu-id="42f92-714">I filtri di azione:</span><span class="sxs-lookup"><span data-stu-id="42f92-714">Action filters:</span></span>

* <span data-ttu-id="42f92-715">Implementano l'interfaccia <xref:Microsoft.AspNetCore.Mvc.Filters.IActionFilter> o <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncActionFilter>.</span><span class="sxs-lookup"><span data-stu-id="42f92-715">Implement either the <xref:Microsoft.AspNetCore.Mvc.Filters.IActionFilter> or <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncActionFilter> interface.</span></span>
* <span data-ttu-id="42f92-716">La loro esecuzione circonda l'esecuzione dei metodi di azione.</span><span class="sxs-lookup"><span data-stu-id="42f92-716">Their execution surrounds the execution of action methods.</span></span>

<span data-ttu-id="42f92-717">Il codice seguente mostra un filtro di azione di esempio:</span><span class="sxs-lookup"><span data-stu-id="42f92-717">The following code shows a sample action filter:</span></span>

[!code-csharp[](./filters/sample/FiltersSample/Filters/MySampleActionFilter.cs?name=snippet_ActionFilter)]

<span data-ttu-id="42f92-718">La classe <xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutingContext> specifica le proprietà seguenti:</span><span class="sxs-lookup"><span data-stu-id="42f92-718">The <xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutingContext> provides the following properties:</span></span>

* <span data-ttu-id="42f92-719"><xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutingContext.ActionArguments>: consente di leggere gli input per un metodo di azione.</span><span class="sxs-lookup"><span data-stu-id="42f92-719"><xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutingContext.ActionArguments> - enables the inputs to an action method be read.</span></span>
* <span data-ttu-id="42f92-720"><xref:Microsoft.AspNetCore.Mvc.Controller>: consente di modificare l'istanza del controller.</span><span class="sxs-lookup"><span data-stu-id="42f92-720"><xref:Microsoft.AspNetCore.Mvc.Controller> - enables manipulating the controller instance.</span></span>
* <span data-ttu-id="42f92-721"><xref:System.Web.Mvc.ActionExecutingContext.Result>: l'impostazione di `Result` causa il corto circuito del metodo di azione e dei filtri di azione successivi.</span><span class="sxs-lookup"><span data-stu-id="42f92-721"><xref:System.Web.Mvc.ActionExecutingContext.Result> - setting `Result` short-circuits execution of the action method and subsequent action filters.</span></span>

<span data-ttu-id="42f92-722">La generazione di un'eccezione in un metodo di azione:</span><span class="sxs-lookup"><span data-stu-id="42f92-722">Throwing an exception in an action method:</span></span>

* <span data-ttu-id="42f92-723">Impedisce l'esecuzione dei filtri successivi.</span><span class="sxs-lookup"><span data-stu-id="42f92-723">Prevents running of subsequent filters.</span></span>
* <span data-ttu-id="42f92-724">A differenza dell'impostazione di `Result`, è considerata un errore anziché un risultato positivo.</span><span class="sxs-lookup"><span data-stu-id="42f92-724">Unlike setting `Result`, is treated as a failure instead of a successful result.</span></span>

<span data-ttu-id="42f92-725">La classe <xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutedContext> specifica `Controller` e `Result` oltre alle proprietà seguenti:</span><span class="sxs-lookup"><span data-stu-id="42f92-725">The <xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutedContext> provides `Controller` and `Result` plus the following properties:</span></span>

* <span data-ttu-id="42f92-726"><xref:System.Web.Mvc.ActionExecutedContext.Canceled>: true se un altro file ha causato il corto circuito dell'azione.</span><span class="sxs-lookup"><span data-stu-id="42f92-726"><xref:System.Web.Mvc.ActionExecutedContext.Canceled> - True if the action execution was short-circuited by another filter.</span></span>
* <span data-ttu-id="42f92-727"><xref:System.Web.Mvc.ActionExecutedContext.Exception>: non Null se l'azione o un filtro di azione eseguito in precedenza ha generato un'eccezione.</span><span class="sxs-lookup"><span data-stu-id="42f92-727"><xref:System.Web.Mvc.ActionExecutedContext.Exception> - Non-null if the action or a previously run action filter threw an exception.</span></span> <span data-ttu-id="42f92-728">Se si imposta questa proprietà su Null:</span><span class="sxs-lookup"><span data-stu-id="42f92-728">Setting this property to null:</span></span>

  * <span data-ttu-id="42f92-729">L'eccezione viene gestita in modo efficace.</span><span class="sxs-lookup"><span data-stu-id="42f92-729">Effectively handles the exception.</span></span>
  * <span data-ttu-id="42f92-730">L'oggetto `Result` viene eseguito come se fosse stato restituito dal metodo di azione.</span><span class="sxs-lookup"><span data-stu-id="42f92-730">`Result` is executed as if it was returned from the action method.</span></span>

<span data-ttu-id="42f92-731">Per un oggetto `IAsyncActionFilter`, una chiamata a <xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutionDelegate>:</span><span class="sxs-lookup"><span data-stu-id="42f92-731">For an `IAsyncActionFilter`, a call to the <xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutionDelegate>:</span></span>

* <span data-ttu-id="42f92-732">Esegue qualsiasi filtro azione successivo e il metodo di azione.</span><span class="sxs-lookup"><span data-stu-id="42f92-732">Executes any subsequent action filters and the action method.</span></span>
* <span data-ttu-id="42f92-733">Restituisce `ActionExecutedContext`.</span><span class="sxs-lookup"><span data-stu-id="42f92-733">Returns `ActionExecutedContext`.</span></span>

<span data-ttu-id="42f92-734">Per causare il corto circuito, assegnare <xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutingContext.Result?displayProperty=fullName> a un'istanza del risultato e non chiamare `next` (`ActionExecutionDelegate`).</span><span class="sxs-lookup"><span data-stu-id="42f92-734">To short-circuit, assign <xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutingContext.Result?displayProperty=fullName> to a result instance and don't call `next` (the `ActionExecutionDelegate`).</span></span>

<span data-ttu-id="42f92-735">Il framework fornisce un oggetto <xref:Microsoft.AspNetCore.Mvc.Filters.ActionFilterAttribute> astratto che è possibile impostare come sottoclasse.</span><span class="sxs-lookup"><span data-stu-id="42f92-735">The framework provides an abstract <xref:Microsoft.AspNetCore.Mvc.Filters.ActionFilterAttribute> that can be subclassed.</span></span>

<span data-ttu-id="42f92-736">Il filtro di azione `OnActionExecuting` può essere usato per:</span><span class="sxs-lookup"><span data-stu-id="42f92-736">The `OnActionExecuting` action filter can be used to:</span></span>

* <span data-ttu-id="42f92-737">Convalidare lo stato del modello.</span><span class="sxs-lookup"><span data-stu-id="42f92-737">Validate model state.</span></span>
* <span data-ttu-id="42f92-738">Restituire un errore se lo stato non è valido.</span><span class="sxs-lookup"><span data-stu-id="42f92-738">Return an error if the state is invalid.</span></span>

[!code-csharp[](./filters/sample/FiltersSample/Filters/ValidateModelAttribute.cs?name=snippet)]

<span data-ttu-id="42f92-739">Il metodo `OnActionExecuted` viene eseguito dopo il metodo di azione:</span><span class="sxs-lookup"><span data-stu-id="42f92-739">The `OnActionExecuted` method runs after the action method:</span></span>

* <span data-ttu-id="42f92-740">È possibile visualizzare e modificare i risultati dell'azione tramite la proprietà <xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutedContext.Result>.</span><span class="sxs-lookup"><span data-stu-id="42f92-740">And can see and manipulate the results of the action through the <xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutedContext.Result> property.</span></span>
* <span data-ttu-id="42f92-741">L'impostazione di <xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutedContext.Canceled> è true se un altro filtro ha causato il corto circuito dell'esecuzione dell'azione.</span><span class="sxs-lookup"><span data-stu-id="42f92-741"><xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutedContext.Canceled> is set to true if the action execution was short-circuited by another filter.</span></span>
* <span data-ttu-id="42f92-742">L'impostazione di <xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutedContext.Exception> è un valore non Null se l'azione o un filtro di azione successivo ha generato un'eccezione.</span><span class="sxs-lookup"><span data-stu-id="42f92-742"><xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutedContext.Exception> is set to a non-null value if the action or a subsequent action filter threw an exception.</span></span> <span data-ttu-id="42f92-743">Impostazione di `Exception` su null:</span><span class="sxs-lookup"><span data-stu-id="42f92-743">Setting `Exception` to null:</span></span>

  * <span data-ttu-id="42f92-744">Gestisce un'eccezione in modo efficace.</span><span class="sxs-lookup"><span data-stu-id="42f92-744">Effectively handles an exception.</span></span>
  * <span data-ttu-id="42f92-745">L'oggetto `ActionExecutedContext.Result` viene eseguito come se fosse stato restituito normalmente dal metodo di azione.</span><span class="sxs-lookup"><span data-stu-id="42f92-745">`ActionExecutedContext.Result` is executed as if it were returned normally from the action method.</span></span>

[!code-csharp[](./filters/sample/FiltersSample/Filters/ValidateModelAttribute.cs?name=snippet2&higlight=12-99)]

## <a name="exception-filters"></a><span data-ttu-id="42f92-746">Filtri eccezioni</span><span class="sxs-lookup"><span data-stu-id="42f92-746">Exception filters</span></span>

<span data-ttu-id="42f92-747">Filtri eccezioni:</span><span class="sxs-lookup"><span data-stu-id="42f92-747">Exception filters:</span></span>

* <span data-ttu-id="42f92-748">Implementano <xref:Microsoft.AspNetCore.Mvc.Filters.IExceptionFilter> o <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncExceptionFilter>.</span><span class="sxs-lookup"><span data-stu-id="42f92-748">Implement <xref:Microsoft.AspNetCore.Mvc.Filters.IExceptionFilter> or <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncExceptionFilter>.</span></span> 
* <span data-ttu-id="42f92-749">Possono essere usati per implementare criteri di gestione degli errori comuni.</span><span class="sxs-lookup"><span data-stu-id="42f92-749">Can be used to implement common error handling policies.</span></span>

<span data-ttu-id="42f92-750">Il filtro di eccezione di esempio seguente usa una visualizzazione degli errori personalizzata per mostrare i dettagli sulle eccezioni che si verificano quando l'app è in fase di sviluppo:</span><span class="sxs-lookup"><span data-stu-id="42f92-750">The following sample exception filter uses a custom error view to display details about exceptions that occur when the app is in development:</span></span>

[!code-csharp[](./filters/sample/FiltersSample/Filters/CustomExceptionFilter.cs?name=snippet_ExceptionFilter&highlight=16-19)]

<span data-ttu-id="42f92-751">Filtri eccezioni:</span><span class="sxs-lookup"><span data-stu-id="42f92-751">Exception filters:</span></span>

* <span data-ttu-id="42f92-752">Non dispone di eventi precedenti o successivi.</span><span class="sxs-lookup"><span data-stu-id="42f92-752">Don't have before and after events.</span></span>
* <span data-ttu-id="42f92-753">Implementano <xref:Microsoft.AspNetCore.Mvc.Filters.IExceptionFilter.OnException*> o <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncExceptionFilter.OnExceptionAsync*>.</span><span class="sxs-lookup"><span data-stu-id="42f92-753">Implement <xref:Microsoft.AspNetCore.Mvc.Filters.IExceptionFilter.OnException*> or <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncExceptionFilter.OnExceptionAsync*>.</span></span>
* <span data-ttu-id="42f92-754">Gestire le eccezioni non gestite che si verificano nella Razor creazione della pagina o del controller, dell' [associazione di modelli](xref:mvc/models/model-binding), dei filtri azione o dei metodi di azione.</span><span class="sxs-lookup"><span data-stu-id="42f92-754">Handle unhandled exceptions that occur in Razor Page or controller creation, [model binding](xref:mvc/models/model-binding), action filters, or action methods.</span></span>
* <span data-ttu-id="42f92-755">**Non** rilevano le eccezioni che si verificano nei filtri di risorse, nei filtri dei risultati o nell'esecuzione dei risultati MVC.</span><span class="sxs-lookup"><span data-stu-id="42f92-755">Do **not** catch exceptions that occur in resource filters, result filters, or MVC result execution.</span></span>

<span data-ttu-id="42f92-756">Per gestire un'eccezione, impostare la proprietà <xref:System.Web.Mvc.ExceptionContext.ExceptionHandled> su `true` o scrivere una risposta.</span><span class="sxs-lookup"><span data-stu-id="42f92-756">To handle an exception, set the <xref:System.Web.Mvc.ExceptionContext.ExceptionHandled> property to `true` or write a response.</span></span> <span data-ttu-id="42f92-757">In questo modo si arresta la propagazione dell'eccezione.</span><span class="sxs-lookup"><span data-stu-id="42f92-757">This stops propagation of the exception.</span></span> <span data-ttu-id="42f92-758">Un filtro di eccezione non può modificare un'eccezione in una "operazione riuscita".</span><span class="sxs-lookup"><span data-stu-id="42f92-758">An exception filter can't turn an exception into a "success".</span></span> <span data-ttu-id="42f92-759">Questa operazione può essere eseguita solo tramite un filtro di azione.</span><span class="sxs-lookup"><span data-stu-id="42f92-759">Only an action filter can do that.</span></span>

<span data-ttu-id="42f92-760">Filtri eccezioni:</span><span class="sxs-lookup"><span data-stu-id="42f92-760">Exception filters:</span></span>

* <span data-ttu-id="42f92-761">Sono ideali per intercettare le eccezioni che si verificano nelle azioni.</span><span class="sxs-lookup"><span data-stu-id="42f92-761">Are good for trapping exceptions that occur within actions.</span></span>
* <span data-ttu-id="42f92-762">Non sono flessibili quanto il middleware di gestione degli errori.</span><span class="sxs-lookup"><span data-stu-id="42f92-762">Are not as flexible as error handling middleware.</span></span>

<span data-ttu-id="42f92-763">Scegliere il middleware per la gestione delle eccezioni.</span><span class="sxs-lookup"><span data-stu-id="42f92-763">Prefer middleware for exception handling.</span></span> <span data-ttu-id="42f92-764">Usare i filtri di eccezione solo quando la gestione degli errori *varia* in base al metodo di azione chiamato.</span><span class="sxs-lookup"><span data-stu-id="42f92-764">Use exception filters only where error handling *differs* based on which action method is called.</span></span> <span data-ttu-id="42f92-765">Un'app può ad esempio avere metodi di azione sia per gli endpoint API che per le visualizzazioni/HTML.</span><span class="sxs-lookup"><span data-stu-id="42f92-765">For example, an app might have action methods for both API endpoints and for views/HTML.</span></span> <span data-ttu-id="42f92-766">Gli endpoint dell'API possono restituire informazioni sull'errore come JSON, mentre le azioni basate sulla visualizzazione possono restituire una pagina di errore in formato HTML.</span><span class="sxs-lookup"><span data-stu-id="42f92-766">The API endpoints could return error information as JSON, while the view-based actions could return an error page as HTML.</span></span>

## <a name="result-filters"></a><span data-ttu-id="42f92-767">Filtri risultato</span><span class="sxs-lookup"><span data-stu-id="42f92-767">Result filters</span></span>

<span data-ttu-id="42f92-768">I filtri dei risultati:</span><span class="sxs-lookup"><span data-stu-id="42f92-768">Result filters:</span></span>

* <span data-ttu-id="42f92-769">Implementare un'interfaccia:</span><span class="sxs-lookup"><span data-stu-id="42f92-769">Implement an interface:</span></span>
  * <span data-ttu-id="42f92-770"><xref:Microsoft.AspNetCore.Mvc.Filters.IResultFilter> o <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncResultFilter></span><span class="sxs-lookup"><span data-stu-id="42f92-770"><xref:Microsoft.AspNetCore.Mvc.Filters.IResultFilter> or <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncResultFilter></span></span>
  * <span data-ttu-id="42f92-771"><xref:Microsoft.AspNetCore.Mvc.Filters.IAlwaysRunResultFilter> o <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncAlwaysRunResultFilter></span><span class="sxs-lookup"><span data-stu-id="42f92-771"><xref:Microsoft.AspNetCore.Mvc.Filters.IAlwaysRunResultFilter> or <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncAlwaysRunResultFilter></span></span>
* <span data-ttu-id="42f92-772">La loro esecuzione circonda l'esecuzione dei risultati dell'azione.</span><span class="sxs-lookup"><span data-stu-id="42f92-772">Their execution surrounds the execution of action results.</span></span>

### <a name="iresultfilter-and-iasyncresultfilter"></a><span data-ttu-id="42f92-773">IResultFilter e IAsyncResultFilter</span><span class="sxs-lookup"><span data-stu-id="42f92-773">IResultFilter and IAsyncResultFilter</span></span>

<span data-ttu-id="42f92-774">Il codice seguente illustra un filtro dei risultati che aggiunge un'intestazione HTTP:</span><span class="sxs-lookup"><span data-stu-id="42f92-774">The following code shows a result filter that adds an HTTP header:</span></span>

[!code-csharp[](./filters/sample/FiltersSample/Filters/LoggingAddHeaderFilter.cs?name=snippet_ResultFilter)]

<span data-ttu-id="42f92-775">Il tipo di risultato eseguito dipende dall'azione.</span><span class="sxs-lookup"><span data-stu-id="42f92-775">The kind of result being executed depends on the action.</span></span> <span data-ttu-id="42f92-776">Un'azione che restituisce una visualizzazione include tutte le elaborazioni Razor come parte dell'elemento <xref:Microsoft.AspNetCore.Mvc.ViewResult> eseguito.</span><span class="sxs-lookup"><span data-stu-id="42f92-776">An action returning a view would include all razor processing as part of the <xref:Microsoft.AspNetCore.Mvc.ViewResult> being executed.</span></span> <span data-ttu-id="42f92-777">Un metodo API può eseguire la serializzazione in quanto parte dell'esecuzione del risultato.</span><span class="sxs-lookup"><span data-stu-id="42f92-777">An API method might perform some serialization as part of the execution of the result.</span></span> <span data-ttu-id="42f92-778">Altre informazioni sui [risultati dell'azione](xref:mvc/controllers/actions).</span><span class="sxs-lookup"><span data-stu-id="42f92-778">Learn more about [action results](xref:mvc/controllers/actions).</span></span>

<span data-ttu-id="42f92-779">I filtri dei risultati vengono eseguiti solo quando un filtro azione o azione produce un risultato di azione.</span><span class="sxs-lookup"><span data-stu-id="42f92-779">Result filters are only executed when an action or action filter produces an action result.</span></span> <span data-ttu-id="42f92-780">I filtri dei risultati non vengono eseguiti nei casi seguenti:</span><span class="sxs-lookup"><span data-stu-id="42f92-780">Result filters are not executed when:</span></span>

* <span data-ttu-id="42f92-781">Un filtro di autorizzazione o un filtro risorse cortocircui la pipeline.</span><span class="sxs-lookup"><span data-stu-id="42f92-781">An authorization filter or resource filter short-circuits the pipeline.</span></span>
* <span data-ttu-id="42f92-782">Un filtro di eccezione non gestisca un'eccezione producendo un risultato dell'azione.</span><span class="sxs-lookup"><span data-stu-id="42f92-782">An exception filter handles an exception by producing an action result.</span></span>

<span data-ttu-id="42f92-783">Il metodo <xref:Microsoft.AspNetCore.Mvc.Filters.IResultFilter.OnResultExecuting*?displayProperty=fullName> può causare il corto circuito dell'esecuzione del risultato dell'azione e dei filtri dei risultati successivi se si imposta <xref:Microsoft.AspNetCore.Mvc.Filters.ResultExecutingContext.Cancel?displayProperty=fullName> su `true`.</span><span class="sxs-lookup"><span data-stu-id="42f92-783">The <xref:Microsoft.AspNetCore.Mvc.Filters.IResultFilter.OnResultExecuting*?displayProperty=fullName> method can short-circuit execution of the action result and subsequent result filters by setting <xref:Microsoft.AspNetCore.Mvc.Filters.ResultExecutingContext.Cancel?displayProperty=fullName> to `true`.</span></span> <span data-ttu-id="42f92-784">In caso di corto circuito, scrivere nell'oggetto risposta per evitare di generare una risposta vuota.</span><span class="sxs-lookup"><span data-stu-id="42f92-784">Write to the response object when short-circuiting to avoid generating an empty response.</span></span> <span data-ttu-id="42f92-785">La generazione di un'eccezione in `IResultFilter.OnResultExecuting`:</span><span class="sxs-lookup"><span data-stu-id="42f92-785">Throwing an exception in `IResultFilter.OnResultExecuting` will:</span></span>

* <span data-ttu-id="42f92-786">Impedisce l'esecuzione del risultato dell'azione e dei filtri successivi.</span><span class="sxs-lookup"><span data-stu-id="42f92-786">Prevent execution of the action result and subsequent filters.</span></span>
* <span data-ttu-id="42f92-787">È considerata un errore anziché un risultato positivo.</span><span class="sxs-lookup"><span data-stu-id="42f92-787">Be treated as a failure instead of a successful result.</span></span>

<span data-ttu-id="42f92-788">Quando <xref:Microsoft.AspNetCore.Mvc.Filters.IResultFilter.OnResultExecuted*?displayProperty=fullName> viene eseguito il metodo, è probabile che la risposta sia già stata inviata al client.</span><span class="sxs-lookup"><span data-stu-id="42f92-788">When the <xref:Microsoft.AspNetCore.Mvc.Filters.IResultFilter.OnResultExecuted*?displayProperty=fullName> method runs, the response has likely already been sent to the client.</span></span> <span data-ttu-id="42f92-789">Se la risposta è già stata inviata al client, non è possibile modificarla ulteriormente.</span><span class="sxs-lookup"><span data-stu-id="42f92-789">If the response has already been sent to the client, it cannot be changed further.</span></span>

<span data-ttu-id="42f92-790">L'impostazione di `ResultExecutedContext.Canceled` è `true` se un altro filtro ha causato il corto circuito dell'esecuzione del risultato dell'azione.</span><span class="sxs-lookup"><span data-stu-id="42f92-790">`ResultExecutedContext.Canceled` is set to `true` if the action result execution was short-circuited by another filter.</span></span>

<span data-ttu-id="42f92-791">L'impostazione di `ResultExecutedContext.Exception` è un valore non Null se il risultato dell'azione o un filtro dei risultati successivo ha generato un'eccezione.</span><span class="sxs-lookup"><span data-stu-id="42f92-791">`ResultExecutedContext.Exception` is set to a non-null value if the action result or a subsequent result filter threw an exception.</span></span> <span data-ttu-id="42f92-792">Se si imposta `Exception` su Null l'eccezione viene gestita in modo efficace e si evita che venga generata nuovamente da ASP.NET Core in un punto successivo della pipeline.</span><span class="sxs-lookup"><span data-stu-id="42f92-792">Setting `Exception` to null effectively handles an exception and prevents the exception from being rethrown by ASP.NET Core later in the pipeline.</span></span> <span data-ttu-id="42f92-793">Non c'è alcun modo affidabile per scrivere i dati in una risposta quando si gestisce un'eccezione in un filtro dei risultati.</span><span class="sxs-lookup"><span data-stu-id="42f92-793">There is no reliable way to write data to a response when handling an exception in a result filter.</span></span> <span data-ttu-id="42f92-794">Se le intestazioni sono state scaricate nel client quando il risultato di un'azione genera un'eccezione, non c'è alcun meccanismo affidabile per inviare un codice di errore.</span><span class="sxs-lookup"><span data-stu-id="42f92-794">If the headers have been flushed to the client when an action result throws an exception, there's no reliable mechanism to send a failure code.</span></span>

<span data-ttu-id="42f92-795">Per un oggetto <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncResultFilter>, una chiamata a `await next` in <xref:Microsoft.AspNetCore.Mvc.Filters.ResultExecutionDelegate> esegue tutti i filtri dei risultati successivi e il risultato dell'azione.</span><span class="sxs-lookup"><span data-stu-id="42f92-795">For an <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncResultFilter>, a call to `await next` on the <xref:Microsoft.AspNetCore.Mvc.Filters.ResultExecutionDelegate> executes any subsequent result filters and the action result.</span></span> <span data-ttu-id="42f92-796">Per causare il corto circuito, impostare [ResultExecutingContext.Cancel](xref:Microsoft.AspNetCore.Mvc.Filters.ResultExecutingContext.Cancel) su `true` e non chiamare `ResultExecutionDelegate`:</span><span class="sxs-lookup"><span data-stu-id="42f92-796">To short-circuit, set [ResultExecutingContext.Cancel](xref:Microsoft.AspNetCore.Mvc.Filters.ResultExecutingContext.Cancel) to `true` and don't call the `ResultExecutionDelegate`:</span></span>

[!code-csharp[](./filters/sample/FiltersSample/Filters/MyAsyncResponseFilter.cs?name=snippet)]

<span data-ttu-id="42f92-797">Il framework fornisce un oggetto `ResultFilterAttribute` astratto che è possibile impostare come sottoclasse.</span><span class="sxs-lookup"><span data-stu-id="42f92-797">The framework provides an abstract `ResultFilterAttribute` that can be subclassed.</span></span> <span data-ttu-id="42f92-798">La classe [AddHeaderAttribute](#add-header-attribute) illustrata in precedenza è un esempio di un attributo di filtro dei risultati.</span><span class="sxs-lookup"><span data-stu-id="42f92-798">The [AddHeaderAttribute](#add-header-attribute) class shown previously is an example of a result filter attribute.</span></span>

### <a name="ialwaysrunresultfilter-and-iasyncalwaysrunresultfilter"></a><span data-ttu-id="42f92-799">IAlwaysRunResultFilter e IAsyncAlwaysRunResultFilter</span><span class="sxs-lookup"><span data-stu-id="42f92-799">IAlwaysRunResultFilter and IAsyncAlwaysRunResultFilter</span></span>

<span data-ttu-id="42f92-800">Le interfacce <xref:Microsoft.AspNetCore.Mvc.Filters.IAlwaysRunResultFilter> e <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncAlwaysRunResultFilter> dichiarano un'implementazione di <xref:Microsoft.AspNetCore.Mvc.Filters.IResultFilter> eseguita per tutti i risultati dell'azione.</span><span class="sxs-lookup"><span data-stu-id="42f92-800">The <xref:Microsoft.AspNetCore.Mvc.Filters.IAlwaysRunResultFilter> and <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncAlwaysRunResultFilter> interfaces declare an <xref:Microsoft.AspNetCore.Mvc.Filters.IResultFilter> implementation that runs for all action results.</span></span> <span data-ttu-id="42f92-801">Sono inclusi i risultati dell'azione prodotti da:</span><span class="sxs-lookup"><span data-stu-id="42f92-801">This includes action results produced by:</span></span>

* <span data-ttu-id="42f92-802">Filtri di autorizzazione e filtri delle risorse che interessano il cortocircuito.</span><span class="sxs-lookup"><span data-stu-id="42f92-802">Authorization filters and resource filters that short-circuit.</span></span>
* <span data-ttu-id="42f92-803">Filtri eccezioni.</span><span class="sxs-lookup"><span data-stu-id="42f92-803">Exception filters.</span></span>

<span data-ttu-id="42f92-804">Ad esempio, il filtro seguente viene eseguito sempre e imposta un risultato dell'azione (<xref:Microsoft.AspNetCore.Mvc.ObjectResult>) con un codice di stato *422 Entità non elaborabile* quando la negoziazione del contenuto ha esito negativo:</span><span class="sxs-lookup"><span data-stu-id="42f92-804">For example, the following filter always runs and sets an action result (<xref:Microsoft.AspNetCore.Mvc.ObjectResult>) with a *422 Unprocessable Entity* status code when content negotiation fails:</span></span>

[!code-csharp[](./filters/sample/FiltersSample/Filters/UnprocessableResultFilter.cs?name=snippet)]

## <a name="ifilterfactory"></a><span data-ttu-id="42f92-805">IFilterFactory</span><span class="sxs-lookup"><span data-stu-id="42f92-805">IFilterFactory</span></span>

<span data-ttu-id="42f92-806"><xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory> implementa <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterMetadata>.</span><span class="sxs-lookup"><span data-stu-id="42f92-806"><xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory> implements <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterMetadata>.</span></span> <span data-ttu-id="42f92-807">Pertanto, un'istanza di `IFilterFactory` può essere usata come un'istanza di `IFilterMetadata` in un punto qualsiasi della pipeline filtro.</span><span class="sxs-lookup"><span data-stu-id="42f92-807">Therefore, an `IFilterFactory` instance can be used as an `IFilterMetadata` instance anywhere in the filter pipeline.</span></span> <span data-ttu-id="42f92-808">Quando il runtime si prepara per richiamare il filtro, cerca di eseguirne il cast a un oggetto `IFilterFactory`.</span><span class="sxs-lookup"><span data-stu-id="42f92-808">When the runtime prepares to invoke the filter, it attempts to cast it to an `IFilterFactory`.</span></span> <span data-ttu-id="42f92-809">Se l'esecuzione del cast ha esito positivo, viene chiamato il metodo <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory.CreateInstance*> per creare l'istanza di `IFilterMetadata` richiamata.</span><span class="sxs-lookup"><span data-stu-id="42f92-809">If that cast succeeds, the <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory.CreateInstance*> method is called to create the `IFilterMetadata` instance that is invoked.</span></span> <span data-ttu-id="42f92-810">In questo modo viene specificata una struttura flessibile, poiché non è necessario impostare in modo esplicito la pipeline filtro all'avvio dell'app.</span><span class="sxs-lookup"><span data-stu-id="42f92-810">This provides a flexible design, since the precise filter pipeline doesn't need to be set explicitly when the app starts.</span></span>

<span data-ttu-id="42f92-811">Un altro approccio alla creazione di filtri consiste nell'implementare `IFilterFactory` usando implementazioni dell'attributo personalizzate:</span><span class="sxs-lookup"><span data-stu-id="42f92-811">`IFilterFactory` can be implemented using custom attribute implementations as another approach to creating filters:</span></span>

[!code-csharp[](./filters/sample/FiltersSample/Filters/AddHeaderWithFactoryAttribute.cs?name=snippet_IFilterFactory&highlight=1,4,5,6,7)]

<span data-ttu-id="42f92-812">Il codice precedente può essere testato eseguendo l'[esempio scaricato](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/controllers/filters/sample):</span><span class="sxs-lookup"><span data-stu-id="42f92-812">The preceding code can be tested by running the [download sample](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/controllers/filters/sample):</span></span>

* <span data-ttu-id="42f92-813">Richiamare gli strumenti di sviluppo F12.</span><span class="sxs-lookup"><span data-stu-id="42f92-813">Invoke the F12 developer tools.</span></span>
* <span data-ttu-id="42f92-814">Accedere a `https://localhost:5001/Sample/HeaderWithFactory`.</span><span class="sxs-lookup"><span data-stu-id="42f92-814">Navigate to `https://localhost:5001/Sample/HeaderWithFactory`.</span></span>

<span data-ttu-id="42f92-815">Gli strumenti di sviluppo F12 visualizzano le intestazioni di risposta seguenti aggiunte dal codice di esempio:</span><span class="sxs-lookup"><span data-stu-id="42f92-815">The F12 developer tools display the following response headers added by the sample code:</span></span>

* <span data-ttu-id="42f92-816">**Autore:**`Joe Smith`</span><span class="sxs-lookup"><span data-stu-id="42f92-816">**author:** `Joe Smith`</span></span>
* <span data-ttu-id="42f92-817">**globaladdheader:** `Result filter added to MvcOptions.Filters`</span><span class="sxs-lookup"><span data-stu-id="42f92-817">**globaladdheader:** `Result filter added to MvcOptions.Filters`</span></span>
* <span data-ttu-id="42f92-818">**interno:**`My header`</span><span class="sxs-lookup"><span data-stu-id="42f92-818">**internal:** `My header`</span></span>

<span data-ttu-id="42f92-819">Il codice precedente crea l'intestazione di risposta **internal:** `My header`.</span><span class="sxs-lookup"><span data-stu-id="42f92-819">The preceding code creates the **internal:** `My header` response header.</span></span>

### <a name="ifilterfactory-implemented-on-an-attribute"></a><span data-ttu-id="42f92-820">Implementazione di IFilterFactory in un attributo</span><span class="sxs-lookup"><span data-stu-id="42f92-820">IFilterFactory implemented on an attribute</span></span>

<!-- Review 
This section needs to be rewritten.
What's a non-named attribute?
-->

<span data-ttu-id="42f92-821">I filtri che implementano `IFilterFactory` sono utili per i filtri che:</span><span class="sxs-lookup"><span data-stu-id="42f92-821">Filters that implement `IFilterFactory` are useful for filters that:</span></span>

* <span data-ttu-id="42f92-822">Non richiedono il passaggio di parametri.</span><span class="sxs-lookup"><span data-stu-id="42f92-822">Don't require passing parameters.</span></span>
* <span data-ttu-id="42f92-823">Hanno dipendenze del costruttore che devono essere soddisfatte dall'inserimento delle dipendenze.</span><span class="sxs-lookup"><span data-stu-id="42f92-823">Have constructor dependencies that need to be filled by DI.</span></span>

<span data-ttu-id="42f92-824"><xref:Microsoft.AspNetCore.Mvc.TypeFilterAttribute> implementa <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory>.</span><span class="sxs-lookup"><span data-stu-id="42f92-824"><xref:Microsoft.AspNetCore.Mvc.TypeFilterAttribute> implements <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory>.</span></span> <span data-ttu-id="42f92-825">`IFilterFactory` espone il metodo <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory.CreateInstance*> per creare un'istanza <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterMetadata>.</span><span class="sxs-lookup"><span data-stu-id="42f92-825">`IFilterFactory` exposes the <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory.CreateInstance*> method for creating an <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterMetadata> instance.</span></span> <span data-ttu-id="42f92-826">`CreateInstance` carica il tipo specificato dal contenitore di inserimento delle dipendenze.</span><span class="sxs-lookup"><span data-stu-id="42f92-826">`CreateInstance` loads the specified type from the services container (DI).</span></span>

[!code-csharp[](./filters/sample/FiltersSample/Filters/SampleActionFilterAttribute.cs?name=snippet_TypeFilterAttribute&highlight=1,3,7)]

<span data-ttu-id="42f92-827">Il codice seguente illustra tre approcci all'applicazione di `[SampleActionFilter]`:</span><span class="sxs-lookup"><span data-stu-id="42f92-827">The following code shows three approaches to applying the `[SampleActionFilter]`:</span></span>

[!code-csharp[](./filters/sample/FiltersSample/Controllers/HomeController.cs?name=snippet&highlight=1)]

<span data-ttu-id="42f92-828">Nel codice precedente la decorazione del metodo con `[SampleActionFilter]` è l'approccio da preferire per applicare `SampleActionFilter`.</span><span class="sxs-lookup"><span data-stu-id="42f92-828">In the preceding code, decorating the method with `[SampleActionFilter]` is the preferred approach to applying the `SampleActionFilter`.</span></span>

## <a name="using-middleware-in-the-filter-pipeline"></a><span data-ttu-id="42f92-829">Uso di middleware nella pipeline filtro</span><span class="sxs-lookup"><span data-stu-id="42f92-829">Using middleware in the filter pipeline</span></span>

<span data-ttu-id="42f92-830">I filtri risorse funzionano come [middleware](xref:fundamentals/middleware/index) in quanto racchiudono l'esecuzione di tutto ciò che viene dopo nella pipeline.</span><span class="sxs-lookup"><span data-stu-id="42f92-830">Resource filters work like [middleware](xref:fundamentals/middleware/index) in that they surround the execution of everything that comes later in the pipeline.</span></span> <span data-ttu-id="42f92-831">I filtri sono diversi dal middleware in quanto fanno parte del runtime di ASP.NET Core, quindi hanno accesso al contesto e ai costrutti di ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="42f92-831">But filters differ from middleware in that they're part of the ASP.NET Core runtime, which means that they have access to ASP.NET Core context and constructs.</span></span>

<span data-ttu-id="42f92-832">Per usare il middleware come filtro, creare un tipo con un metodo `Configure` che specifica il middleware da inserire nella pipeline di filtro.</span><span class="sxs-lookup"><span data-stu-id="42f92-832">To use middleware as a filter, create a type with a `Configure` method that specifies the middleware to inject into the filter pipeline.</span></span> <span data-ttu-id="42f92-833">L'esempio seguente usa il middleware di localizzazione per stabilire le impostazioni cultura correnti per una richiesta:</span><span class="sxs-lookup"><span data-stu-id="42f92-833">The following example uses the localization middleware to establish the current culture for a request:</span></span>

[!code-csharp[](./filters/sample/FiltersSample/Filters/LocalizationPipeline.cs?name=snippet_MiddlewareFilter&highlight=3,22)]

<span data-ttu-id="42f92-834">Usare <xref:Microsoft.AspNetCore.Mvc.MiddlewareFilterAttribute> per eseguire il middleware:</span><span class="sxs-lookup"><span data-stu-id="42f92-834">Use the <xref:Microsoft.AspNetCore.Mvc.MiddlewareFilterAttribute> to run the middleware:</span></span>

[!code-csharp[](./filters/sample/FiltersSample/Controllers/HomeController.cs?name=snippet_MiddlewareFilter&highlight=2)]

<span data-ttu-id="42f92-835">I filtri middleware vengono eseguiti nella stessa fase della pipeline filtro come filtri risorse, prima dell'associazione di modelli e dopo il resto della pipeline.</span><span class="sxs-lookup"><span data-stu-id="42f92-835">Middleware filters run at the same stage of the filter pipeline as Resource filters, before model binding and after the rest of the pipeline.</span></span>

## <a name="next-actions"></a><span data-ttu-id="42f92-836">Azioni successive</span><span class="sxs-lookup"><span data-stu-id="42f92-836">Next actions</span></span>

* <span data-ttu-id="42f92-837">Vedere [metodi di filtro per le Razor pagine](xref:razor-pages/filter).</span><span class="sxs-lookup"><span data-stu-id="42f92-837">See [Filter methods for Razor Pages](xref:razor-pages/filter).</span></span>
* <span data-ttu-id="42f92-838">Per sperimentare i filtri, [scaricare, testare e modificare l'esempio di GitHub](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/controllers/filters/sample).</span><span class="sxs-lookup"><span data-stu-id="42f92-838">To experiment with filters, [download, test, and modify the GitHub sample](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/controllers/filters/sample).</span></span>

::: moniker-end
