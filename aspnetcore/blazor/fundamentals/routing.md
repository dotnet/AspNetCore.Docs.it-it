---
title: BlazorRouting ASP.NET Core
author: guardrex
description: Informazioni su come gestire il routing delle richieste nelle app e su come usare il componente NavLink nelle Blazor app per la navigazione.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 12/09/2020
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
uid: blazor/fundamentals/routing
ms.openlocfilehash: 55e2cbc01af7352facad7121c05c754e9d438ae3
ms.sourcegitcommit: 1166b0ff3828418559510c661e8240e5c5717bb7
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/12/2021
ms.locfileid: "100279882"
---
# <a name="aspnet-core-blazor-routing"></a><span data-ttu-id="06f96-103">BlazorRouting ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="06f96-103">ASP.NET Core Blazor routing</span></span>

<span data-ttu-id="06f96-104">Questo articolo illustra come gestire il routing delle richieste e come usare il <xref:Microsoft.AspNetCore.Components.Routing.NavLink> componente per creare collegamenti di navigazione nelle Blazor app.</span><span class="sxs-lookup"><span data-stu-id="06f96-104">In this article, learn how to manage request routing and how to use the <xref:Microsoft.AspNetCore.Components.Routing.NavLink> component to create a navigation links in Blazor apps.</span></span>

## <a name="route-templates"></a><span data-ttu-id="06f96-105">Modelli di route</span><span class="sxs-lookup"><span data-stu-id="06f96-105">Route templates</span></span>

<span data-ttu-id="06f96-106">Il <xref:Microsoft.AspNetCore.Components.Routing.Router> componente consente il routing ai Razor componenti in un' Blazor app.</span><span class="sxs-lookup"><span data-stu-id="06f96-106">The <xref:Microsoft.AspNetCore.Components.Routing.Router> component enables routing to Razor components in a Blazor app.</span></span> <span data-ttu-id="06f96-107">Il <xref:Microsoft.AspNetCore.Components.Routing.Router> componente viene usato nel `App` componente delle Blazor app.</span><span class="sxs-lookup"><span data-stu-id="06f96-107">The <xref:Microsoft.AspNetCore.Components.Routing.Router> component is used in the `App` component of Blazor apps.</span></span>

<span data-ttu-id="06f96-108">`App.razor`:</span><span class="sxs-lookup"><span data-stu-id="06f96-108">`App.razor`:</span></span>

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/routing/App1.razor)]

::: moniker-end

[!INCLUDE[](~/blazor/includes/prefer-exact-matches.md)]

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/routing/App1.razor)]

::: moniker-end

<span data-ttu-id="06f96-109">Quando Razor viene compilato un componente ( `.razor` ) con una [ `@page` direttiva](xref:mvc/views/razor#page) , alla classe del componente generato viene fornito un oggetto che <xref:Microsoft.AspNetCore.Components.RouteAttribute> specifica il modello di route del componente.</span><span class="sxs-lookup"><span data-stu-id="06f96-109">When a Razor component (`.razor`) with an [`@page` directive](xref:mvc/views/razor#page) is compiled, the generated component class is provided a <xref:Microsoft.AspNetCore.Components.RouteAttribute> specifying the component's route template.</span></span>

<span data-ttu-id="06f96-110">All'avvio dell'app, l'assembly specificato come router `AppAssembly` viene analizzato per raccogliere informazioni sulla route per i componenti dell'app che hanno un <xref:Microsoft.AspNetCore.Components.RouteAttribute> .</span><span class="sxs-lookup"><span data-stu-id="06f96-110">When the app starts, the assembly specified as the Router's `AppAssembly` is scanned to gather route information for the app's components that have a <xref:Microsoft.AspNetCore.Components.RouteAttribute>.</span></span>

<span data-ttu-id="06f96-111">In fase di esecuzione, il <xref:Microsoft.AspNetCore.Components.RouteView> componente:</span><span class="sxs-lookup"><span data-stu-id="06f96-111">At runtime, the <xref:Microsoft.AspNetCore.Components.RouteView> component:</span></span>

* <span data-ttu-id="06f96-112">Riceve <xref:Microsoft.AspNetCore.Components.RouteData> da <xref:Microsoft.AspNetCore.Components.Routing.Router> insieme ai parametri di route.</span><span class="sxs-lookup"><span data-stu-id="06f96-112">Receives the <xref:Microsoft.AspNetCore.Components.RouteData> from the <xref:Microsoft.AspNetCore.Components.Routing.Router> along with any route parameters.</span></span>
* <span data-ttu-id="06f96-113">Esegue il rendering del componente specificato con il [layout](xref:blazor/layouts), inclusi eventuali altri layout annidati.</span><span class="sxs-lookup"><span data-stu-id="06f96-113">Renders the specified component with its [layout](xref:blazor/layouts), including any further nested layouts.</span></span>

<span data-ttu-id="06f96-114">Facoltativamente, specificare un <xref:Microsoft.AspNetCore.Components.RouteView.DefaultLayout> parametro con una classe layout per i componenti che non specificano un layout con la [ `@layout` direttiva](xref:blazor/layouts#specify-a-layout-in-a-component).</span><span class="sxs-lookup"><span data-stu-id="06f96-114">Optionally specify a <xref:Microsoft.AspNetCore.Components.RouteView.DefaultLayout> parameter with a layout class for components that don't specify a layout with the [`@layout` directive](xref:blazor/layouts#specify-a-layout-in-a-component).</span></span> <span data-ttu-id="06f96-115">I modelli di Blazor progetto del Framework specificano il `MainLayout` componente ( `Shared/MainLayout.razor` ) come layout predefinito dell'app.</span><span class="sxs-lookup"><span data-stu-id="06f96-115">The framework's Blazor project templates specify the `MainLayout` component (`Shared/MainLayout.razor`) as the app's default layout.</span></span> <span data-ttu-id="06f96-116">Per ulteriori informazioni sui layout, vedere <xref:blazor/layouts> .</span><span class="sxs-lookup"><span data-stu-id="06f96-116">For more information on layouts, see <xref:blazor/layouts>.</span></span>

<span data-ttu-id="06f96-117">I componenti supportano più modelli di route usando più [ `@page` direttive](xref:mvc/views/razor#page).</span><span class="sxs-lookup"><span data-stu-id="06f96-117">Components support multiple route templates using multiple [`@page` directives](xref:mvc/views/razor#page).</span></span> <span data-ttu-id="06f96-118">Il componente di esempio seguente carica le richieste per `/BlazorRoute` e `/DifferentBlazorRoute` .</span><span class="sxs-lookup"><span data-stu-id="06f96-118">The following example component loads on requests for `/BlazorRoute` and `/DifferentBlazorRoute`.</span></span>

<span data-ttu-id="06f96-119">`Pages/BlazorRoute.razor`:</span><span class="sxs-lookup"><span data-stu-id="06f96-119">`Pages/BlazorRoute.razor`:</span></span>

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/routing/BlazorRoute.razor?highlight=1-2)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Pages/routing/BlazorRoute.razor?highlight=1-2)]

::: moniker-end

> [!IMPORTANT]
> <span data-ttu-id="06f96-120">Per la corretta risoluzione degli URL, l'app deve includere un `<base>` tag nel `wwwroot/index.html` file ( Blazor WebAssembly ) o nel `Pages/_Host.cshtml` file ( Blazor Server ) con il percorso di base dell'applicazione specificato nell' `href` attributo.</span><span class="sxs-lookup"><span data-stu-id="06f96-120">For URLs to resolve correctly, the app must include a `<base>` tag in its `wwwroot/index.html` file (Blazor WebAssembly) or `Pages/_Host.cshtml` file (Blazor Server) with the app base path specified in the `href` attribute.</span></span> <span data-ttu-id="06f96-121">Per altre informazioni, vedere <xref:blazor/host-and-deploy/index#app-base-path>.</span><span class="sxs-lookup"><span data-stu-id="06f96-121">For more information, see <xref:blazor/host-and-deploy/index#app-base-path>.</span></span>

## <a name="provide-custom-content-when-content-isnt-found"></a><span data-ttu-id="06f96-122">Fornire contenuto personalizzato quando il contenuto non è stato trovato</span><span class="sxs-lookup"><span data-stu-id="06f96-122">Provide custom content when content isn't found</span></span>

<span data-ttu-id="06f96-123">Il <xref:Microsoft.AspNetCore.Components.Routing.Router> componente consente all'app di specificare contenuto personalizzato se non viene trovato contenuto per la route richiesta.</span><span class="sxs-lookup"><span data-stu-id="06f96-123">The <xref:Microsoft.AspNetCore.Components.Routing.Router> component allows the app to specify custom content if content isn't found for the requested route.</span></span>

<span data-ttu-id="06f96-124">Nel `App` componente impostare il contenuto personalizzato nel modello del <xref:Microsoft.AspNetCore.Components.Routing.Router> componente <xref:Microsoft.AspNetCore.Components.Routing.Router.NotFound> .</span><span class="sxs-lookup"><span data-stu-id="06f96-124">In the `App` component, set custom content in the <xref:Microsoft.AspNetCore.Components.Routing.Router> component's <xref:Microsoft.AspNetCore.Components.Routing.Router.NotFound> template.</span></span>

<span data-ttu-id="06f96-125">`App.razor`:</span><span class="sxs-lookup"><span data-stu-id="06f96-125">`App.razor`:</span></span>

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/routing/App2.razor?highlight=5-8)]

::: moniker-end

[!INCLUDE[](~/blazor/includes/prefer-exact-matches.md)]

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/routing/App2.razor?highlight=5-8)]

::: moniker-end

<span data-ttu-id="06f96-126">Gli elementi arbitrari sono supportati come contenuto dei `<NotFound>` tag, ad esempio altri componenti interattivi.</span><span class="sxs-lookup"><span data-stu-id="06f96-126">Arbitrary items are supported as content of the `<NotFound>` tags, such as other interactive components.</span></span> <span data-ttu-id="06f96-127">Per applicare un layout predefinito al <xref:Microsoft.AspNetCore.Components.Routing.Router.NotFound> contenuto, vedere <xref:blazor/layouts#default-layout> .</span><span class="sxs-lookup"><span data-stu-id="06f96-127">To apply a default layout to <xref:Microsoft.AspNetCore.Components.Routing.Router.NotFound> content, see <xref:blazor/layouts#default-layout>.</span></span>

## <a name="route-to-components-from-multiple-assemblies"></a><span data-ttu-id="06f96-128">Routing a componenti da più assembly</span><span class="sxs-lookup"><span data-stu-id="06f96-128">Route to components from multiple assemblies</span></span>

<span data-ttu-id="06f96-129">Usare il <xref:Microsoft.AspNetCore.Components.Routing.Router.AdditionalAssemblies> parametro per specificare gli assembly aggiuntivi da <xref:Microsoft.AspNetCore.Components.Routing.Router> considerare per il componente durante la ricerca di componenti instradabili.</span><span class="sxs-lookup"><span data-stu-id="06f96-129">Use the <xref:Microsoft.AspNetCore.Components.Routing.Router.AdditionalAssemblies> parameter to specify additional assemblies for the <xref:Microsoft.AspNetCore.Components.Routing.Router> component to consider when searching for routable components.</span></span> <span data-ttu-id="06f96-130">Verranno analizzati altri assembly oltre all'assembly specificato in `AppAssembly` .</span><span class="sxs-lookup"><span data-stu-id="06f96-130">Additional assemblies are scanned in addition to the assembly specified to `AppAssembly`.</span></span> <span data-ttu-id="06f96-131">Nell'esempio seguente `Component1` è un componente instradabile definito in una [libreria di classi di componenti](xref:blazor/components/class-libraries)a cui si fa riferimento.</span><span class="sxs-lookup"><span data-stu-id="06f96-131">In the following example, `Component1` is a routable component defined in a referenced [component class library](xref:blazor/components/class-libraries).</span></span> <span data-ttu-id="06f96-132">Nell' <xref:Microsoft.AspNetCore.Components.Routing.Router.AdditionalAssemblies> esempio seguente viene restituito il supporto del routing per `Component1` .</span><span class="sxs-lookup"><span data-stu-id="06f96-132">The following <xref:Microsoft.AspNetCore.Components.Routing.Router.AdditionalAssemblies> example results in routing support for `Component1`.</span></span>

<span data-ttu-id="06f96-133">`App.razor`:</span><span class="sxs-lookup"><span data-stu-id="06f96-133">`App.razor`:</span></span>

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/routing/App3.razor?name=snippet)]

::: moniker-end

[!INCLUDE[](~/blazor/includes/prefer-exact-matches.md)]

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/routing/App3.razor?name=snippet)]

::: moniker-end

## <a name="route-parameters"></a><span data-ttu-id="06f96-134">Parametri di route</span><span class="sxs-lookup"><span data-stu-id="06f96-134">Route parameters</span></span>

<span data-ttu-id="06f96-135">Il router usa parametri di route per popolare i [parametri del componente](xref:blazor/components/index#component-parameters) corrispondenti con lo stesso nome.</span><span class="sxs-lookup"><span data-stu-id="06f96-135">The router uses route parameters to populate the corresponding [component parameters](xref:blazor/components/index#component-parameters) with the same name.</span></span> <span data-ttu-id="06f96-136">Per i nomi dei parametri di route non viene fatta distinzione tra maiuscole</span><span class="sxs-lookup"><span data-stu-id="06f96-136">Route parameter names are case insensitive.</span></span> <span data-ttu-id="06f96-137">Nell'esempio seguente il `text` parametro assegna il valore del segmento di route alla proprietà del componente `Text` .</span><span class="sxs-lookup"><span data-stu-id="06f96-137">In the following example, the `text` parameter assigns the value of the route segment to the component's `Text` property.</span></span> <span data-ttu-id="06f96-138">Quando viene effettuata una richiesta per `/RouteParameter/amazing` , `<h1>` viene eseguito il rendering del contenuto del tag come `Blazor is amazing!` .</span><span class="sxs-lookup"><span data-stu-id="06f96-138">When a request is made for `/RouteParameter/amazing`, the `<h1>` tag content is rendered as `Blazor is amazing!`.</span></span>

<span data-ttu-id="06f96-139">`Pages/RouteParameter.razor`:</span><span class="sxs-lookup"><span data-stu-id="06f96-139">`Pages/RouteParameter.razor`:</span></span>

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/routing/RouteParameter1.razor?highlight=1)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Pages/routing/RouteParameter1.razor?highlight=1)]

::: moniker-end

::: moniker range=">= aspnetcore-5.0"

<span data-ttu-id="06f96-140">Sono supportati i parametri facoltativi.</span><span class="sxs-lookup"><span data-stu-id="06f96-140">Optional parameters are supported.</span></span> <span data-ttu-id="06f96-141">Nell'esempio seguente, il `text` parametro facoltativo assegna il valore del segmento di route alla proprietà del componente `Text` .</span><span class="sxs-lookup"><span data-stu-id="06f96-141">In the following example, the `text` optional parameter assigns the value of the route segment to the component's `Text` property.</span></span> <span data-ttu-id="06f96-142">Se il segmento non è presente, il valore di `Text` è impostato su `fantastic` .</span><span class="sxs-lookup"><span data-stu-id="06f96-142">If the segment isn't present, the value of `Text` is set to `fantastic`.</span></span>

<span data-ttu-id="06f96-143">`Pages/RouteParameter.razor`:</span><span class="sxs-lookup"><span data-stu-id="06f96-143">`Pages/RouteParameter.razor`:</span></span>

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/routing/RouteParameter2.razor?highlight=1)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

<span data-ttu-id="06f96-144">I parametri facoltativi non sono supportati.</span><span class="sxs-lookup"><span data-stu-id="06f96-144">Optional parameters aren't supported.</span></span> <span data-ttu-id="06f96-145">Nell'esempio seguente vengono applicate due [ `@page` direttive](xref:mvc/views/razor#page) .</span><span class="sxs-lookup"><span data-stu-id="06f96-145">In the following example, two [`@page` directives](xref:mvc/views/razor#page) are applied.</span></span> <span data-ttu-id="06f96-146">La prima direttiva consente la navigazione al componente senza un parametro.</span><span class="sxs-lookup"><span data-stu-id="06f96-146">The first directive permits navigation to the component without a parameter.</span></span> <span data-ttu-id="06f96-147">La seconda direttiva assegna il `{text}` valore del parametro di route alla proprietà del componente `Text` .</span><span class="sxs-lookup"><span data-stu-id="06f96-147">The second directive assigns the `{text}` route parameter value to the component's `Text` property.</span></span>

<span data-ttu-id="06f96-148">`Pages/RouteParameter.razor`:</span><span class="sxs-lookup"><span data-stu-id="06f96-148">`Pages/RouteParameter.razor`:</span></span>

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Pages/routing/RouteParameter2.razor?highlight=2)]

::: moniker-end

<span data-ttu-id="06f96-149">Usare [`OnParametersSet`](xref:blazor/components/lifecycle#after-parameters-are-set) anziché [`OnInitialized`](xref:blazor/components/lifecycle#component-initialization-methods) per consentire l'esplorazione delle app allo stesso componente con un valore di parametro facoltativo diverso.</span><span class="sxs-lookup"><span data-stu-id="06f96-149">Use [`OnParametersSet`](xref:blazor/components/lifecycle#after-parameters-are-set) instead of [`OnInitialized`](xref:blazor/components/lifecycle#component-initialization-methods) to permit app navigation to the same component with a different optional parameter value.</span></span> <span data-ttu-id="06f96-150">In base all'esempio precedente, usare `OnParametersSet` quando l'utente deve essere in grado di spostarsi da `/RouteParameter` a `/RouteParameter/amazing` o da `/RouteParameter/amazing` a `/RouteParameter` :</span><span class="sxs-lookup"><span data-stu-id="06f96-150">Based on the preceding example, use `OnParametersSet` when the user should be able to navigate from `/RouteParameter` to `/RouteParameter/amazing` or from `/RouteParameter/amazing` to `/RouteParameter`:</span></span>

```csharp
protected override void OnParametersSet()
{
    Text = Text ?? "fantastic";
}
```

## <a name="route-constraints"></a><span data-ttu-id="06f96-151">Vincoli di route</span><span class="sxs-lookup"><span data-stu-id="06f96-151">Route constraints</span></span>

<span data-ttu-id="06f96-152">Un vincolo di route impone la corrispondenza tra i tipi in un segmento di route e un componente.</span><span class="sxs-lookup"><span data-stu-id="06f96-152">A route constraint enforces type matching on a route segment to a component.</span></span>

<span data-ttu-id="06f96-153">Nell'esempio seguente, la route per il `User` componente corrisponde solo a se:</span><span class="sxs-lookup"><span data-stu-id="06f96-153">In the following example, the route to the `User` component only matches if:</span></span>

* <span data-ttu-id="06f96-154">Un `Id` segmento di route è presente nell'URL della richiesta.</span><span class="sxs-lookup"><span data-stu-id="06f96-154">An `Id` route segment is present in the request URL.</span></span>
* <span data-ttu-id="06f96-155">Il `Id` segmento è un tipo Integer ( `int` ).</span><span class="sxs-lookup"><span data-stu-id="06f96-155">The `Id` segment is an integer (`int`) type.</span></span>

<span data-ttu-id="06f96-156">`Pages/User.razor`:</span><span class="sxs-lookup"><span data-stu-id="06f96-156">`Pages/User.razor`:</span></span>

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/routing/User.razor?highlight=1)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Pages/routing/User.razor?highlight=1)]

::: moniker-end

<span data-ttu-id="06f96-157">Sono disponibili i vincoli di route indicati nella tabella seguente.</span><span class="sxs-lookup"><span data-stu-id="06f96-157">The route constraints shown in the following table are available.</span></span> <span data-ttu-id="06f96-158">Per ulteriori informazioni sui vincoli di route che corrispondono alle impostazioni cultura invarianti, vedere l'avviso sotto la tabella.</span><span class="sxs-lookup"><span data-stu-id="06f96-158">For the route constraints that match the invariant culture, see the warning below the table for more information.</span></span>

| <span data-ttu-id="06f96-159">Vincolo</span><span class="sxs-lookup"><span data-stu-id="06f96-159">Constraint</span></span> | <span data-ttu-id="06f96-160">Esempio</span><span class="sxs-lookup"><span data-stu-id="06f96-160">Example</span></span>           | <span data-ttu-id="06f96-161">Esempi di corrispondenza</span><span class="sxs-lookup"><span data-stu-id="06f96-161">Example Matches</span></span>                                                                  | <span data-ttu-id="06f96-162">Invariante</span><span class="sxs-lookup"><span data-stu-id="06f96-162">Invariant</span></span><br><span data-ttu-id="06f96-163">culture</span><span class="sxs-lookup"><span data-stu-id="06f96-163">culture</span></span><br><span data-ttu-id="06f96-164">corrispondenti</span><span class="sxs-lookup"><span data-stu-id="06f96-164">matching</span></span> |
| ---------- | ----------------- | -------------------------------------------------------------------------------- | :------------------------------: |
| `bool`     | `{active:bool}`   | <span data-ttu-id="06f96-165">`true`, `FALSE`</span><span class="sxs-lookup"><span data-stu-id="06f96-165">`true`, `FALSE`</span></span>                                                                  | <span data-ttu-id="06f96-166">No</span><span class="sxs-lookup"><span data-stu-id="06f96-166">No</span></span>                               |
| `datetime` | `{dob:datetime}`  | <span data-ttu-id="06f96-167">`2016-12-31`, `2016-12-31 7:32pm`</span><span class="sxs-lookup"><span data-stu-id="06f96-167">`2016-12-31`, `2016-12-31 7:32pm`</span></span>                                                | <span data-ttu-id="06f96-168">Sì</span><span class="sxs-lookup"><span data-stu-id="06f96-168">Yes</span></span>                              |
| `decimal`  | `{price:decimal}` | <span data-ttu-id="06f96-169">`49.99`, `-1,000.01`</span><span class="sxs-lookup"><span data-stu-id="06f96-169">`49.99`, `-1,000.01`</span></span>                                                             | <span data-ttu-id="06f96-170">Sì</span><span class="sxs-lookup"><span data-stu-id="06f96-170">Yes</span></span>                              |
| `double`   | `{weight:double}` | <span data-ttu-id="06f96-171">`1.234`, `-1,001.01e8`</span><span class="sxs-lookup"><span data-stu-id="06f96-171">`1.234`, `-1,001.01e8`</span></span>                                                           | <span data-ttu-id="06f96-172">Sì</span><span class="sxs-lookup"><span data-stu-id="06f96-172">Yes</span></span>                              |
| `float`    | `{weight:float}`  | <span data-ttu-id="06f96-173">`1.234`, `-1,001.01e8`</span><span class="sxs-lookup"><span data-stu-id="06f96-173">`1.234`, `-1,001.01e8`</span></span>                                                           | <span data-ttu-id="06f96-174">Sì</span><span class="sxs-lookup"><span data-stu-id="06f96-174">Yes</span></span>                              |
| `guid`     | `{id:guid}`       | <span data-ttu-id="06f96-175">`CD2C1638-1638-72D5-1638-DEADBEEF1638`, `{CD2C1638-1638-72D5-1638-DEADBEEF1638}`</span><span class="sxs-lookup"><span data-stu-id="06f96-175">`CD2C1638-1638-72D5-1638-DEADBEEF1638`, `{CD2C1638-1638-72D5-1638-DEADBEEF1638}`</span></span> | <span data-ttu-id="06f96-176">No</span><span class="sxs-lookup"><span data-stu-id="06f96-176">No</span></span>                               |
| `int`      | `{id:int}`        | <span data-ttu-id="06f96-177">`123456789`, `-123456789`</span><span class="sxs-lookup"><span data-stu-id="06f96-177">`123456789`, `-123456789`</span></span>                                                        | <span data-ttu-id="06f96-178">Sì</span><span class="sxs-lookup"><span data-stu-id="06f96-178">Yes</span></span>                              |
| `long`     | `{ticks:long}`    | <span data-ttu-id="06f96-179">`123456789`, `-123456789`</span><span class="sxs-lookup"><span data-stu-id="06f96-179">`123456789`, `-123456789`</span></span>                                                        | <span data-ttu-id="06f96-180">Sì</span><span class="sxs-lookup"><span data-stu-id="06f96-180">Yes</span></span>                              |

> [!WARNING]
> <span data-ttu-id="06f96-181">I vincoli di route che verificano l'URL e vengono convertiti in un tipo CLR, ad esempio `int` o <xref:System.DateTime>, usano sempre le impostazioni cultura inglese non dipendenti da paese/area geografica,</span><span class="sxs-lookup"><span data-stu-id="06f96-181">Route constraints that verify the URL and are converted to a CLR type (such as `int` or <xref:System.DateTime>) always use the invariant culture.</span></span> <span data-ttu-id="06f96-182">presupponendo che l'URL sia non localizzabile.</span><span class="sxs-lookup"><span data-stu-id="06f96-182">These constraints assume that the URL is non-localizable.</span></span>

## <a name="routing-with-urls-that-contain-dots"></a><span data-ttu-id="06f96-183">Routing con URL contenenti punti</span><span class="sxs-lookup"><span data-stu-id="06f96-183">Routing with URLs that contain dots</span></span>

<span data-ttu-id="06f96-184">Per Blazor WebAssembly le app e ospitate Blazor Server , il modello di route predefinito sul lato server presuppone che se l'ultimo segmento di un URL della richiesta contiene un punto ( `.` ) richiesto da un file.</span><span class="sxs-lookup"><span data-stu-id="06f96-184">For hosted Blazor WebAssembly and Blazor Server apps, the server-side default route template assumes that if the last segment of a request URL contains a dot (`.`) that a file is requested.</span></span> <span data-ttu-id="06f96-185">Ad esempio, l'URL `https://localhost.com:5001/example/some.thing` viene interpretato dal router come una richiesta di un file denominato `some.thing` .</span><span class="sxs-lookup"><span data-stu-id="06f96-185">For example, the URL `https://localhost.com:5001/example/some.thing` is interpreted by the router as a request for a file named `some.thing`.</span></span> <span data-ttu-id="06f96-186">Senza alcuna configurazione aggiuntiva, un'app restituisce una risposta *404 non trovata* se `some.thing` è stata progettata per il routing a un componente con una [ `@page` direttiva](xref:mvc/views/razor#page) e `some.thing` è un valore di parametro di route.</span><span class="sxs-lookup"><span data-stu-id="06f96-186">Without additional configuration, an app returns a *404 - Not Found* response if `some.thing` was meant to route to a component with an [`@page` directive](xref:mvc/views/razor#page) and `some.thing` is a route parameter value.</span></span> <span data-ttu-id="06f96-187">Per usare una route con uno o più parametri che contengono un punto, l'app deve configurare la route con un modello personalizzato.</span><span class="sxs-lookup"><span data-stu-id="06f96-187">To use a route with one or more parameters that contain a dot, the app must configure the route with a custom template.</span></span>

<span data-ttu-id="06f96-188">Si consideri il `Example` componente seguente che può ricevere un parametro di route dall'ultimo segmento dell'URL.</span><span class="sxs-lookup"><span data-stu-id="06f96-188">Consider the following `Example` component that can receive a route parameter from the last segment of the URL.</span></span>

<span data-ttu-id="06f96-189">`Pages/Example.razor`:</span><span class="sxs-lookup"><span data-stu-id="06f96-189">`Pages/Example.razor`:</span></span>

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/routing/Example.razor?highlight=1)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Pages/routing/Example.razor?highlight=2)]

::: moniker-end

<span data-ttu-id="06f96-190">Per consentire all' *`Server`* app di una soluzione ospitata Blazor WebAssembly di instradare la richiesta con un punto nel `param` parametro di route, aggiungere un modello di route per il file di fallback con il parametro facoltativo in `Startup.Configure` .</span><span class="sxs-lookup"><span data-stu-id="06f96-190">To permit the *`Server`* app of a hosted Blazor WebAssembly solution to route the request with a dot in the `param` route parameter, add a fallback file route template with the optional parameter in `Startup.Configure`.</span></span>

<span data-ttu-id="06f96-191">`Startup.cs`:</span><span class="sxs-lookup"><span data-stu-id="06f96-191">`Startup.cs`:</span></span>

```csharp
endpoints.MapFallbackToFile("/example/{param?}", "index.html");
```

<span data-ttu-id="06f96-192">Per configurare un' Blazor Server app per instradare la richiesta con un punto nel `param` parametro di route, aggiungere un modello di route della pagina di fallback con il parametro facoltativo in `Startup.Configure` .</span><span class="sxs-lookup"><span data-stu-id="06f96-192">To configure a Blazor Server app to route the request with a dot in the `param` route parameter, add a fallback page route template with the optional parameter in `Startup.Configure`.</span></span>

<span data-ttu-id="06f96-193">`Startup.cs`:</span><span class="sxs-lookup"><span data-stu-id="06f96-193">`Startup.cs`:</span></span>

```csharp
endpoints.MapFallbackToPage("/example/{param?}", "/_Host");
```

<span data-ttu-id="06f96-194">Per altre informazioni, vedere <xref:fundamentals/routing>.</span><span class="sxs-lookup"><span data-stu-id="06f96-194">For more information, see <xref:fundamentals/routing>.</span></span>

## <a name="catch-all-route-parameters"></a><span data-ttu-id="06f96-195">Parametri di route catch-all</span><span class="sxs-lookup"><span data-stu-id="06f96-195">Catch-all route parameters</span></span>

::: moniker range=">= aspnetcore-5.0"

<span data-ttu-id="06f96-196">I parametri di route catch-all, che acquisiscono percorsi tra più limiti di cartella, sono supportati nei componenti di.</span><span class="sxs-lookup"><span data-stu-id="06f96-196">Catch-all route parameters, which capture paths across multiple folder boundaries, are supported in components.</span></span>

<span data-ttu-id="06f96-197">I parametri di route catch-all sono:</span><span class="sxs-lookup"><span data-stu-id="06f96-197">Catch-all route parameters are:</span></span>

* <span data-ttu-id="06f96-198">Denominato in modo che corrisponda al nome del segmento di route.</span><span class="sxs-lookup"><span data-stu-id="06f96-198">Named to match the route segment name.</span></span> <span data-ttu-id="06f96-199">La denominazione non distingue tra maiuscole e minuscole.</span><span class="sxs-lookup"><span data-stu-id="06f96-199">Naming isn't case sensitive.</span></span>
* <span data-ttu-id="06f96-200">Tipo `string`.</span><span class="sxs-lookup"><span data-stu-id="06f96-200">A `string` type.</span></span> <span data-ttu-id="06f96-201">Il Framework non fornisce il cast automatico.</span><span class="sxs-lookup"><span data-stu-id="06f96-201">The framework doesn't provide automatic casting.</span></span>
* <span data-ttu-id="06f96-202">Alla fine dell'URL.</span><span class="sxs-lookup"><span data-stu-id="06f96-202">At the end of the URL.</span></span>

<span data-ttu-id="06f96-203">`Pages/CatchAll.razor`:</span><span class="sxs-lookup"><span data-stu-id="06f96-203">`Pages/CatchAll.razor`:</span></span>

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/routing/CatchAll.razor)]

<span data-ttu-id="06f96-204">Per l'URL `/catch-all/this/is/a/test` con un modello di route di `/catch-all/{*pageRoute}` , il valore di `PageRoute` è impostato su `this/is/a/test` .</span><span class="sxs-lookup"><span data-stu-id="06f96-204">For the URL `/catch-all/this/is/a/test` with a route template of `/catch-all/{*pageRoute}`, the value of `PageRoute` is set to `this/is/a/test`.</span></span>

<span data-ttu-id="06f96-205">Le barre e i segmenti del percorso acquisito vengono decodificati.</span><span class="sxs-lookup"><span data-stu-id="06f96-205">Slashes and segments of the captured path are decoded.</span></span> <span data-ttu-id="06f96-206">Per un modello di route di `/catch-all/{*pageRoute}` , l'URL `/catch-all/this/is/a%2Ftest%2A` restituisce `this/is/a/test*` .</span><span class="sxs-lookup"><span data-stu-id="06f96-206">For a route template of `/catch-all/{*pageRoute}`, the URL `/catch-all/this/is/a%2Ftest%2A` yields `this/is/a/test*`.</span></span>

::: moniker-end

::: moniker range="< aspnetcore-5.0"

<span data-ttu-id="06f96-207">I parametri di route catch-all sono supportati in ASP.NET Core 5,0 o versioni successive.</span><span class="sxs-lookup"><span data-stu-id="06f96-207">Catch-all route parameters are supported in ASP.NET Core 5.0 or later.</span></span> <span data-ttu-id="06f96-208">Per ulteriori informazioni, selezionare la versione 5,0 di questo articolo.</span><span class="sxs-lookup"><span data-stu-id="06f96-208">For more information, select the 5.0 version of this article.</span></span>

::: moniker-end

## <a name="uri-and-navigation-state-helpers"></a><span data-ttu-id="06f96-209">Helper per lo stato di navigazione e URI</span><span class="sxs-lookup"><span data-stu-id="06f96-209">URI and navigation state helpers</span></span>

<span data-ttu-id="06f96-210">Usare <xref:Microsoft.AspNetCore.Components.NavigationManager> per gestire gli URI e la navigazione nel codice C#.</span><span class="sxs-lookup"><span data-stu-id="06f96-210">Use <xref:Microsoft.AspNetCore.Components.NavigationManager> to manage URIs and navigation in C# code.</span></span> <span data-ttu-id="06f96-211"><xref:Microsoft.AspNetCore.Components.NavigationManager> fornisce l'evento e i metodi illustrati nella tabella seguente.</span><span class="sxs-lookup"><span data-stu-id="06f96-211"><xref:Microsoft.AspNetCore.Components.NavigationManager> provides the event and methods shown in the following table.</span></span>

| <span data-ttu-id="06f96-212">Membro</span><span class="sxs-lookup"><span data-stu-id="06f96-212">Member</span></span> | <span data-ttu-id="06f96-213">Descrizione</span><span class="sxs-lookup"><span data-stu-id="06f96-213">Description</span></span> |
| ------ | ----------- |
| <xref:Microsoft.AspNetCore.Components.NavigationManager.Uri> | <span data-ttu-id="06f96-214">Ottiene l'URI assoluto corrente.</span><span class="sxs-lookup"><span data-stu-id="06f96-214">Gets the current absolute URI.</span></span> |
| <xref:Microsoft.AspNetCore.Components.NavigationManager.BaseUri> | <span data-ttu-id="06f96-215">Ottiene l'URI di base (con una barra finale) che può essere anteposto ai percorsi URI relativi per produrre un URI assoluto.</span><span class="sxs-lookup"><span data-stu-id="06f96-215">Gets the base URI (with a trailing slash) that can be prepended to relative URI paths to produce an absolute URI.</span></span> <span data-ttu-id="06f96-216">In genere, <xref:Microsoft.AspNetCore.Components.NavigationManager.BaseUri> corrisponde all' `href` attributo sull'elemento del documento `<base>` in `wwwroot/index.html` ( Blazor WebAssembly ) o `Pages/_Host.cshtml` ( Blazor Server ).</span><span class="sxs-lookup"><span data-stu-id="06f96-216">Typically, <xref:Microsoft.AspNetCore.Components.NavigationManager.BaseUri> corresponds to the `href` attribute on the document's `<base>` element in `wwwroot/index.html` (Blazor WebAssembly) or `Pages/_Host.cshtml` (Blazor Server).</span></span> |
| <xref:Microsoft.AspNetCore.Components.NavigationManager.NavigateTo%2A> | <span data-ttu-id="06f96-217">Passa all'URI specificato.</span><span class="sxs-lookup"><span data-stu-id="06f96-217">Navigates to the specified URI.</span></span> <span data-ttu-id="06f96-218">Se `forceLoad` è `true` :</span><span class="sxs-lookup"><span data-stu-id="06f96-218">If `forceLoad` is `true`:</span></span><ul><li><span data-ttu-id="06f96-219">Il routing lato client viene ignorato.</span><span class="sxs-lookup"><span data-stu-id="06f96-219">Client-side routing is bypassed.</span></span></li><li><span data-ttu-id="06f96-220">Il browser è forzato a caricare la nuova pagina dal server, indipendentemente dal fatto che l'URI venga normalmente gestito dal router lato client.</span><span class="sxs-lookup"><span data-stu-id="06f96-220">The browser is forced to load the new page from the server, whether or not the URI is normally handled by the client-side router.</span></span></li></ul> |
| <xref:Microsoft.AspNetCore.Components.NavigationManager.LocationChanged> | <span data-ttu-id="06f96-221">Un evento che viene attivato quando il percorso di navigazione viene modificato.</span><span class="sxs-lookup"><span data-stu-id="06f96-221">An event that fires when the navigation location has changed.</span></span> |
| <xref:Microsoft.AspNetCore.Components.NavigationManager.ToAbsoluteUri%2A> | <span data-ttu-id="06f96-222">Converte un URI relativo in un URI assoluto.</span><span class="sxs-lookup"><span data-stu-id="06f96-222">Converts a relative URI into an absolute URI.</span></span> |
| <span style="word-break:normal;word-wrap:normal"><xref:Microsoft.AspNetCore.Components.NavigationManager.ToBaseRelativePath%2A></span> | <span data-ttu-id="06f96-223">Dato un URI di base (ad esempio, un URI restituito in precedenza da <xref:Microsoft.AspNetCore.Components.NavigationManager.BaseUri> ), converte un URI assoluto in un URI relativo al prefisso URI di base.</span><span class="sxs-lookup"><span data-stu-id="06f96-223">Given a base URI (for example, a URI previously returned by <xref:Microsoft.AspNetCore.Components.NavigationManager.BaseUri>), converts an absolute URI into a URI relative to the base URI prefix.</span></span> |

<span data-ttu-id="06f96-224">Per l' <xref:Microsoft.AspNetCore.Components.NavigationManager.LocationChanged> evento, <xref:Microsoft.AspNetCore.Components.Routing.LocationChangedEventArgs> fornisce le informazioni seguenti sugli eventi di navigazione:</span><span class="sxs-lookup"><span data-stu-id="06f96-224">For the <xref:Microsoft.AspNetCore.Components.NavigationManager.LocationChanged> event, <xref:Microsoft.AspNetCore.Components.Routing.LocationChangedEventArgs> provides the following information about navigation events:</span></span>

* <span data-ttu-id="06f96-225"><xref:Microsoft.AspNetCore.Components.Routing.LocationChangedEventArgs.Location>: URL della nuova posizione.</span><span class="sxs-lookup"><span data-stu-id="06f96-225"><xref:Microsoft.AspNetCore.Components.Routing.LocationChangedEventArgs.Location>: The URL of the new location.</span></span>
* <span data-ttu-id="06f96-226"><xref:Microsoft.AspNetCore.Components.Routing.LocationChangedEventArgs.IsNavigationIntercepted>: Se `true` , Blazor intercetta la navigazione dal browser.</span><span class="sxs-lookup"><span data-stu-id="06f96-226"><xref:Microsoft.AspNetCore.Components.Routing.LocationChangedEventArgs.IsNavigationIntercepted>: If `true`, Blazor intercepted the navigation from the browser.</span></span> <span data-ttu-id="06f96-227">Se `false` , <xref:Microsoft.AspNetCore.Components.NavigationManager.NavigateTo%2A?displayProperty=nameWithType> ha causato la navigazione.</span><span class="sxs-lookup"><span data-stu-id="06f96-227">If `false`, <xref:Microsoft.AspNetCore.Components.NavigationManager.NavigateTo%2A?displayProperty=nameWithType> caused the navigation to occur.</span></span>

<span data-ttu-id="06f96-228">Il componente seguente:</span><span class="sxs-lookup"><span data-stu-id="06f96-228">The following component:</span></span>

* <span data-ttu-id="06f96-229">Passa al componente dell'app `Counter` ( `Pages/Counter.razor` ) quando si seleziona il pulsante usando <xref:Microsoft.AspNetCore.Components.NavigationManager.NavigateTo%2A> .</span><span class="sxs-lookup"><span data-stu-id="06f96-229">Navigates to the app's `Counter` component (`Pages/Counter.razor`) when the button is selected using <xref:Microsoft.AspNetCore.Components.NavigationManager.NavigateTo%2A>.</span></span>
* <span data-ttu-id="06f96-230">Gestisce l'evento di modifica della posizione sottoscrivendo <xref:Microsoft.AspNetCore.Components.NavigationManager.LocationChanged?displayProperty=nameWithType> .</span><span class="sxs-lookup"><span data-stu-id="06f96-230">Handles the location changed event by subscribing to <xref:Microsoft.AspNetCore.Components.NavigationManager.LocationChanged?displayProperty=nameWithType>.</span></span>
  * <span data-ttu-id="06f96-231">Il `HandleLocationChanged` metodo è unhooked quando `Dispose` viene chiamato dal Framework.</span><span class="sxs-lookup"><span data-stu-id="06f96-231">The `HandleLocationChanged` method is unhooked when `Dispose` is called by the framework.</span></span> <span data-ttu-id="06f96-232">L'unhook del metodo consente Garbage Collection del componente.</span><span class="sxs-lookup"><span data-stu-id="06f96-232">Unhooking the method permits garbage collection of the component.</span></span>
  * <span data-ttu-id="06f96-233">Quando si seleziona il pulsante, l'implementazione del logger registra le informazioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="06f96-233">The logger implementation logs the following information when the button is selected:</span></span>

    > `BlazorSample.Pages.Navigate: Information: URL of new location: https://localhost:5001/counter`

<span data-ttu-id="06f96-234">`Pages/Navigate.razor`:</span><span class="sxs-lookup"><span data-stu-id="06f96-234">`Pages/Navigate.razor`:</span></span>

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/routing/Navigate.razor)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Pages/routing/Navigate.razor)]

::: moniker-end

<span data-ttu-id="06f96-235">Per ulteriori informazioni sull'eliminazione dei componenti, vedere <xref:blazor/components/lifecycle#component-disposal-with-idisposable> .</span><span class="sxs-lookup"><span data-stu-id="06f96-235">For more information on component disposal, see <xref:blazor/components/lifecycle#component-disposal-with-idisposable>.</span></span>

## <a name="query-string-and-parse-parameters"></a><span data-ttu-id="06f96-236">Stringa di query e parametri di analisi</span><span class="sxs-lookup"><span data-stu-id="06f96-236">Query string and parse parameters</span></span>

<span data-ttu-id="06f96-237">La stringa di query di una richiesta viene ottenuta dalla <xref:Microsoft.AspNetCore.Components.NavigationManager.Uri?displayProperty=nameWithType> proprietà:</span><span class="sxs-lookup"><span data-stu-id="06f96-237">The query string of a request is obtained from the <xref:Microsoft.AspNetCore.Components.NavigationManager.Uri?displayProperty=nameWithType> property:</span></span>

```razor
@inject NavigationManager NavigationManager

...

var query = new Uri(NavigationManager.Uri).Query;
```

<span data-ttu-id="06f96-238">Per analizzare i parametri della stringa di query:</span><span class="sxs-lookup"><span data-stu-id="06f96-238">To parse a query string's parameters:</span></span>

* <span data-ttu-id="06f96-239">Un'app può usare l' <xref:Microsoft.AspNetCore.WebUtilities> API.</span><span class="sxs-lookup"><span data-stu-id="06f96-239">An app can use the <xref:Microsoft.AspNetCore.WebUtilities> API.</span></span> <span data-ttu-id="06f96-240">Se l'API non è disponibile per l'app, aggiungere un riferimento al pacchetto nel file di progetto dell'app per [Microsoft. AspNetCore. webutilities](https://www.nuget.org/packages/Microsoft.AspNetCore.WebUtilities).</span><span class="sxs-lookup"><span data-stu-id="06f96-240">If the API isn't available to the app, add a package reference in the app's project file for [Microsoft.AspNetCore.WebUtilities](https://www.nuget.org/packages/Microsoft.AspNetCore.WebUtilities).</span></span>
* <span data-ttu-id="06f96-241">Ottenere il valore dopo aver analizzato la stringa di query con <xref:Microsoft.AspNetCore.WebUtilities.QueryHelpers.ParseQuery%2A?displayProperty=nameWithType> .</span><span class="sxs-lookup"><span data-stu-id="06f96-241">Obtain the value after parsing the query string with <xref:Microsoft.AspNetCore.WebUtilities.QueryHelpers.ParseQuery%2A?displayProperty=nameWithType>.</span></span>

<span data-ttu-id="06f96-242">Nell' `ParseQueryString` esempio di componente seguente viene analizzata una chiave del parametro della stringa di query denominata `ship` .</span><span class="sxs-lookup"><span data-stu-id="06f96-242">The following `ParseQueryString` component example parses a query string parameter key named `ship`.</span></span> <span data-ttu-id="06f96-243">Ad esempio, la coppia chiave-valore della stringa di query dell'URL `?ship=Tardis` acquisisce il valore `Tardis` in `queryValue` .</span><span class="sxs-lookup"><span data-stu-id="06f96-243">For example, the URL query string key-value pair `?ship=Tardis` captures the value `Tardis` in `queryValue`.</span></span> <span data-ttu-id="06f96-244">Per l'esempio seguente, passare all'app con l'URL `https://localhost:5001/parse-query-string?ship=Tardis` .</span><span class="sxs-lookup"><span data-stu-id="06f96-244">For the following example, navigate to the app with the URL `https://localhost:5001/parse-query-string?ship=Tardis`.</span></span>

<span data-ttu-id="06f96-245">`Pages/ParseQueryString.razor`:</span><span class="sxs-lookup"><span data-stu-id="06f96-245">`Pages/ParseQueryString.razor`:</span></span>

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/routing/ParseQueryString.razor)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Pages/routing/ParseQueryString.razor)]

::: moniker-end

## <a name="navlink-and-navmenu-components"></a><span data-ttu-id="06f96-246">`NavLink``NavMenu`componenti e</span><span class="sxs-lookup"><span data-stu-id="06f96-246">`NavLink` and `NavMenu` components</span></span>

<span data-ttu-id="06f96-247">Usare un <xref:Microsoft.AspNetCore.Components.Routing.NavLink> componente al posto di elementi collegamento ipertestuale HTML ( `<a>` ) durante la creazione di collegamenti di navigazione.</span><span class="sxs-lookup"><span data-stu-id="06f96-247">Use a <xref:Microsoft.AspNetCore.Components.Routing.NavLink> component in place of HTML hyperlink elements (`<a>`) when creating navigation links.</span></span> <span data-ttu-id="06f96-248">Un <xref:Microsoft.AspNetCore.Components.Routing.NavLink> componente si comporta come un `<a>` elemento, con la differenza che viene attivata o disattivata una `active` classe CSS a seconda che `href` corrisponda all'URL corrente.</span><span class="sxs-lookup"><span data-stu-id="06f96-248">A <xref:Microsoft.AspNetCore.Components.Routing.NavLink> component behaves like an `<a>` element, except it toggles an `active` CSS class based on whether its `href` matches the current URL.</span></span> <span data-ttu-id="06f96-249">La `active` classe consente a un utente di comprendere quale pagina è la pagina attiva tra i collegamenti di navigazione visualizzati.</span><span class="sxs-lookup"><span data-stu-id="06f96-249">The `active` class helps a user understand which page is the active page among the navigation links displayed.</span></span> <span data-ttu-id="06f96-250">Facoltativamente, assegnare un nome di classe CSS a <xref:Microsoft.AspNetCore.Components.Routing.NavLink.ActiveClass?displayProperty=nameWithType> per applicare una classe CSS personalizzata al collegamento di cui è stato eseguito il rendering quando la route corrente corrisponde a `href` .</span><span class="sxs-lookup"><span data-stu-id="06f96-250">Optionally, assign a CSS class name to <xref:Microsoft.AspNetCore.Components.Routing.NavLink.ActiveClass?displayProperty=nameWithType> to apply a custom CSS class to the rendered link when the current route matches the `href`.</span></span>

<span data-ttu-id="06f96-251">Il `NavMenu` componente seguente crea una [`Bootstrap`](https://getbootstrap.com/docs/) barra di navigazione che illustra come utilizzare i <xref:Microsoft.AspNetCore.Components.Routing.NavLink> componenti di:</span><span class="sxs-lookup"><span data-stu-id="06f96-251">The following `NavMenu` component creates a [`Bootstrap`](https://getbootstrap.com/docs/) navigation bar that demonstrates how to use <xref:Microsoft.AspNetCore.Components.Routing.NavLink> components:</span></span>

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Shared/routing/NavMenu.razor?name=snippet&highlight=4,9)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Shared/routing/NavMenu.razor?name=snippet&highlight=4,9)]

::: moniker-end

> [!NOTE]
> <span data-ttu-id="06f96-252">Il `NavMenu` componente ( `NavMenu.razor` ) viene fornito nella `Shared` cartella di un'app generata dai modelli di Blazor progetto.</span><span class="sxs-lookup"><span data-stu-id="06f96-252">The `NavMenu` component (`NavMenu.razor`) is provided in the `Shared` folder of an app generated from the Blazor project templates.</span></span>

<span data-ttu-id="06f96-253">Sono disponibili due <xref:Microsoft.AspNetCore.Components.Routing.NavLinkMatch> opzioni che è possibile assegnare all' `Match` attributo dell' `<NavLink>` elemento:</span><span class="sxs-lookup"><span data-stu-id="06f96-253">There are two <xref:Microsoft.AspNetCore.Components.Routing.NavLinkMatch> options that you can assign to the `Match` attribute of the `<NavLink>` element:</span></span>

* <span data-ttu-id="06f96-254"><xref:Microsoft.AspNetCore.Components.Routing.NavLinkMatch.All?displayProperty=nameWithType>: <xref:Microsoft.AspNetCore.Components.Routing.NavLink> È attivo quando corrisponde all'intero URL corrente.</span><span class="sxs-lookup"><span data-stu-id="06f96-254"><xref:Microsoft.AspNetCore.Components.Routing.NavLinkMatch.All?displayProperty=nameWithType>: The <xref:Microsoft.AspNetCore.Components.Routing.NavLink> is active when it matches the entire current URL.</span></span>
* <span data-ttu-id="06f96-255"><xref:Microsoft.AspNetCore.Components.Routing.NavLinkMatch.Prefix?displayProperty=nameWithType> (*impostazione predefinita*): l'oggetto <xref:Microsoft.AspNetCore.Components.Routing.NavLink> è attivo quando corrisponde a qualsiasi prefisso dell'URL corrente.</span><span class="sxs-lookup"><span data-stu-id="06f96-255"><xref:Microsoft.AspNetCore.Components.Routing.NavLinkMatch.Prefix?displayProperty=nameWithType> (*default*): The <xref:Microsoft.AspNetCore.Components.Routing.NavLink> is active when it matches any prefix of the current URL.</span></span>

<span data-ttu-id="06f96-256">Nell'esempio precedente la Home <xref:Microsoft.AspNetCore.Components.Routing.NavLink> `href=""` corrisponde all'URL Home e riceve solo la `active` classe CSS nell'URL del percorso di base predefinito dell'app (ad esempio, `https://localhost:5001/` ).</span><span class="sxs-lookup"><span data-stu-id="06f96-256">In the preceding example, the Home <xref:Microsoft.AspNetCore.Components.Routing.NavLink> `href=""` matches the home URL and only receives the `active` CSS class at the app's default base path URL (for example, `https://localhost:5001/`).</span></span> <span data-ttu-id="06f96-257">Il secondo <xref:Microsoft.AspNetCore.Components.Routing.NavLink> riceve la `active` classe quando l'utente visita un URL con un `component` prefisso, ad esempio `https://localhost:5001/component` e `https://localhost:5001/component/another-segment` .</span><span class="sxs-lookup"><span data-stu-id="06f96-257">The second <xref:Microsoft.AspNetCore.Components.Routing.NavLink> receives the `active` class when the user visits any URL with a `component` prefix (for example, `https://localhost:5001/component` and `https://localhost:5001/component/another-segment`).</span></span>

<span data-ttu-id="06f96-258"><xref:Microsoft.AspNetCore.Components.Routing.NavLink>Gli attributi dei componenti aggiuntivi vengono passati al tag di ancoraggio sottoposto a rendering.</span><span class="sxs-lookup"><span data-stu-id="06f96-258">Additional <xref:Microsoft.AspNetCore.Components.Routing.NavLink> component attributes are passed through to the rendered anchor tag.</span></span> <span data-ttu-id="06f96-259">Nell'esempio seguente il <xref:Microsoft.AspNetCore.Components.Routing.NavLink> componente include l' `target` attributo:</span><span class="sxs-lookup"><span data-stu-id="06f96-259">In the following example, the <xref:Microsoft.AspNetCore.Components.Routing.NavLink> component includes the `target` attribute:</span></span>

```razor
<NavLink href="example-page" target="_blank">Example page</NavLink>
```

<span data-ttu-id="06f96-260">Viene eseguito il rendering del markup HTML seguente:</span><span class="sxs-lookup"><span data-stu-id="06f96-260">The following HTML markup is rendered:</span></span>

```html
<a href="example-page" target="_blank">Example page</a>
```

> [!WARNING]
> <span data-ttu-id="06f96-261">A causa del modo in cui Blazor viene eseguito il rendering del contenuto figlio, `NavLink` i componenti di rendering all'interno di un `for` ciclo richiedono una variabile di indice locale se la variabile del ciclo di incremento viene utilizzata nel `NavLink` contenuto del componente (figlio):</span><span class="sxs-lookup"><span data-stu-id="06f96-261">Due to the way that Blazor renders child content, rendering `NavLink` components inside a `for` loop requires a local index variable if the incrementing loop variable is used in the `NavLink` (child) component's content:</span></span>
>
> ```razor
> @for (int c = 0; c < 10; c++)
> {
>     var current = c;
>     <li ...>
>         <NavLink ... href="@c">
>             <span ...></span> @current
>         </NavLink>
>     </li>
> }
> ```
>
> <span data-ttu-id="06f96-262">L'uso di una variabile di indice in questo scenario è un requisito per **qualsiasi** componente figlio che usa una variabile di ciclo nel [contenuto figlio](xref:blazor/components/index#child-content), non solo nel `NavLink` componente.</span><span class="sxs-lookup"><span data-stu-id="06f96-262">Using an index variable in this scenario is a requirement for **any** child component that uses a loop variable in its [child content](xref:blazor/components/index#child-content), not just the `NavLink` component.</span></span>
>
> <span data-ttu-id="06f96-263">In alternativa, usare un `foreach` ciclo con <xref:System.Linq.Enumerable.Range%2A?displayProperty=nameWithType> :</span><span class="sxs-lookup"><span data-stu-id="06f96-263">Alternatively, use a `foreach` loop with <xref:System.Linq.Enumerable.Range%2A?displayProperty=nameWithType>:</span></span>
>
> ```razor
> @foreach(var c in Enumerable.Range(0,10))
> {
>     <li ...>
>         <NavLink ... href="@c">
>             <span ...></span> @c
>         </NavLink>
>     </li>
> }
> ```

## <a name="aspnet-core-endpoint-routing-integration"></a><span data-ttu-id="06f96-264">Integrazione di routing endpoint ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="06f96-264">ASP.NET Core endpoint routing integration</span></span>

<span data-ttu-id="06f96-265">*Questa sezione si applica solo alle Blazor Server app.*</span><span class="sxs-lookup"><span data-stu-id="06f96-265">*This section only applies to Blazor Server apps.*</span></span>

<span data-ttu-id="06f96-266">Blazor Server è integrato in [ASP.net core routing dell'endpoint](xref:fundamentals/routing).</span><span class="sxs-lookup"><span data-stu-id="06f96-266">Blazor Server is integrated into [ASP.NET Core Endpoint Routing](xref:fundamentals/routing).</span></span> <span data-ttu-id="06f96-267">Un'app ASP.NET Core è configurata in modo da accettare le connessioni in ingresso per i componenti interattivi con <xref:Microsoft.AspNetCore.Builder.ComponentEndpointRouteBuilderExtensions.MapBlazorHub%2A> in `Startup.Configure` .</span><span class="sxs-lookup"><span data-stu-id="06f96-267">An ASP.NET Core app is configured to accept incoming connections for interactive components with <xref:Microsoft.AspNetCore.Builder.ComponentEndpointRouteBuilderExtensions.MapBlazorHub%2A> in `Startup.Configure`.</span></span>

<span data-ttu-id="06f96-268">`Startup.cs`:</span><span class="sxs-lookup"><span data-stu-id="06f96-268">`Startup.cs`:</span></span>

::: moniker range=">= aspnetcore-5.0"

[!code-csharp[](~/blazor/common/samples/5.x/BlazorSample_Server/routing/Startup.cs?name=snippet&highlight=5)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-csharp[](~/blazor/common/samples/3.x/BlazorSample_Server/routing/Startup.cs?name=snippet&highlight=5)]

::: moniker-end

<span data-ttu-id="06f96-269">La configurazione tipica consiste nel routing di tutte le richieste a una Razor pagina, che funge da host per la parte lato server dell' Blazor Server app.</span><span class="sxs-lookup"><span data-stu-id="06f96-269">The typical configuration is to route all requests to a Razor page, which acts as the host for the server-side part of the Blazor Server app.</span></span> <span data-ttu-id="06f96-270">Per convenzione, la pagina *host* viene in genere denominata `_Host.cshtml` nella `Pages` cartella dell'app.</span><span class="sxs-lookup"><span data-stu-id="06f96-270">By convention, the *host* page is usually named `_Host.cshtml` in the `Pages` folder of the app.</span></span>

<span data-ttu-id="06f96-271">La route specificata nel file host viene chiamata route di *fallback* perché funziona con una priorità bassa nella corrispondenza della route.</span><span class="sxs-lookup"><span data-stu-id="06f96-271">The route specified in the host file is called a *fallback route* because it operates with a low priority in route matching.</span></span> <span data-ttu-id="06f96-272">La route di fallback viene utilizzata quando altre route non corrispondono.</span><span class="sxs-lookup"><span data-stu-id="06f96-272">The fallback route is used when other routes don't match.</span></span> <span data-ttu-id="06f96-273">Ciò consente all'app di usare altri controller e pagine senza interferire con il routing dei componenti nell' Blazor Server app.</span><span class="sxs-lookup"><span data-stu-id="06f96-273">This allows the app to use other controllers and pages without interfering with component routing in the Blazor Server app.</span></span>

<span data-ttu-id="06f96-274">Per informazioni sulla configurazione <xref:Microsoft.AspNetCore.Builder.RazorPagesEndpointRouteBuilderExtensions.MapFallbackToPage%2A> di per l'hosting di server URL non radice, vedere <xref:blazor/host-and-deploy/index#app-base-path> .</span><span class="sxs-lookup"><span data-stu-id="06f96-274">For information on configuring <xref:Microsoft.AspNetCore.Builder.RazorPagesEndpointRouteBuilderExtensions.MapFallbackToPage%2A> for non-root URL server hosting, see <xref:blazor/host-and-deploy/index#app-base-path>.</span></span>
