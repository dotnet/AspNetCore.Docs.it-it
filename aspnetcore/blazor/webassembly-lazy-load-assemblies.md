---
title: Assembly di caricamento lazy in ASP.NET Core Blazor WebAssembly
author: guardrex
description: Scopri come Lazy caricare gli assembly nelle Blazor WebAssembly app ASP.NET Core.
monikerRange: '>= aspnetcore-5.0'
ms.author: riande
ms.custom: mvc
ms.date: 09/09/2020
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
uid: blazor/webassembly-lazy-load-assemblies
ms.openlocfilehash: e8589a1e288c39b487673fafc04c59fa07916335
ms.sourcegitcommit: 1166b0ff3828418559510c661e8240e5c5717bb7
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/12/2021
ms.locfileid: "100280664"
---
# <a name="lazy-load-assemblies-in-aspnet-core-blazor-webassembly"></a><span data-ttu-id="c46b5-103">Assembly di caricamento lazy in ASP.NET Core Blazor WebAssembly</span><span class="sxs-lookup"><span data-stu-id="c46b5-103">Lazy load assemblies in ASP.NET Core Blazor WebAssembly</span></span>

<span data-ttu-id="c46b5-104">Blazor WebAssembly le prestazioni di avvio dell'app possono essere migliorate posticipando il caricamento di alcuni assembly di applicazioni fino a quando non sono necessarie, operazione denominata *caricamento lazy*.</span><span class="sxs-lookup"><span data-stu-id="c46b5-104">Blazor WebAssembly app startup performance can be improved by deferring the loading of some application assemblies until they are required, which is called *lazy loading*.</span></span> <span data-ttu-id="c46b5-105">Ad esempio, gli assembly che vengono utilizzati solo per il rendering di un singolo componente possono essere impostati per il caricamento solo se l'utente passa a tale componente.</span><span class="sxs-lookup"><span data-stu-id="c46b5-105">For example, assemblies that are only used to render a single component can be set up to load only if the user navigates to that component.</span></span> <span data-ttu-id="c46b5-106">Dopo il caricamento, gli assembly vengono memorizzati nella cache sul lato client e sono disponibili per tutte le navigazioni future.</span><span class="sxs-lookup"><span data-stu-id="c46b5-106">After loading, the assemblies are cached client-side and are available for all future navigations.</span></span>

<span data-ttu-id="c46b5-107">Blazorla funzionalità di caricamento lazy consente di contrassegnare gli assembly dell'app per il caricamento lazy, che carica gli assembly in fase di esecuzione quando l'utente passa a una route specifica.</span><span class="sxs-lookup"><span data-stu-id="c46b5-107">Blazor's lazy loading feature allows you to mark app assemblies for lazy loading, which loads the assemblies during runtime when the user navigates to a particular route.</span></span> <span data-ttu-id="c46b5-108">La funzionalità è costituita dalle modifiche apportate al file di progetto e dalle modifiche apportate al router dell'applicazione.</span><span class="sxs-lookup"><span data-stu-id="c46b5-108">The feature consists of changes to the project file and changes to the application's router.</span></span>

> [!NOTE]
> <span data-ttu-id="c46b5-109">Il caricamento lazy dell'assembly non avvantaggia Blazor Server le app perché gli assembly non vengono scaricati nel client in un' Blazor Server app.</span><span class="sxs-lookup"><span data-stu-id="c46b5-109">Assembly lazy loading doesn't benefit Blazor Server apps because assemblies aren't downloaded to the client in a Blazor Server app.</span></span>

## <a name="project-file"></a><span data-ttu-id="c46b5-110">File di progetto</span><span class="sxs-lookup"><span data-stu-id="c46b5-110">Project file</span></span>

<span data-ttu-id="c46b5-111">Contrassegnare gli assembly per il caricamento lazy nel file di progetto dell'app ( `.csproj` ) usando l' `BlazorWebAssemblyLazyLoad` elemento.</span><span class="sxs-lookup"><span data-stu-id="c46b5-111">Mark assemblies for lazy loading in the app's project file (`.csproj`) using the `BlazorWebAssemblyLazyLoad` item.</span></span> <span data-ttu-id="c46b5-112">Usare il nome dell'assembly con l' `.dll` estensione.</span><span class="sxs-lookup"><span data-stu-id="c46b5-112">Use the assembly name with the `.dll` extension.</span></span> <span data-ttu-id="c46b5-113">Il Blazor Framework impedisce il caricamento degli assembly specificati da questo gruppo di elementi all'avvio dell'app.</span><span class="sxs-lookup"><span data-stu-id="c46b5-113">The Blazor framework prevents the assemblies specified by this item group from loading at app launch.</span></span> <span data-ttu-id="c46b5-114">Nell'esempio seguente viene contrassegnato un assembly personalizzato di grandi dimensioni ( `GrantImaharaRobotControls.dll` ) per il caricamento lazy.</span><span class="sxs-lookup"><span data-stu-id="c46b5-114">The following example marks a large custom assembly (`GrantImaharaRobotControls.dll`) for lazy loading.</span></span> <span data-ttu-id="c46b5-115">Se un assembly contrassegnato per il caricamento lazy presenta dipendenze, è necessario contrassegnarlo anche per il caricamento lazy nel file di progetto.</span><span class="sxs-lookup"><span data-stu-id="c46b5-115">If an assembly that's marked for lazy loading has dependencies, they must also be marked for lazy loading in the project file.</span></span>

```xml
<ItemGroup>
  <BlazorWebAssemblyLazyLoad Include="GrantImaharaRobotControls.dll" />
</ItemGroup>
```

## <a name="router-component"></a><span data-ttu-id="c46b5-116">Componente `Router`</span><span class="sxs-lookup"><span data-stu-id="c46b5-116">`Router` component</span></span>

<span data-ttu-id="c46b5-117">Blazoril `Router` componente indica gli assembly in cui vengono Blazor cercati i componenti instradabili.</span><span class="sxs-lookup"><span data-stu-id="c46b5-117">Blazor's `Router` component designates which assemblies Blazor searches for routable components.</span></span> <span data-ttu-id="c46b5-118">Il `Router` componente è inoltre responsabile del rendering del componente per la route in cui l'utente si sposta.</span><span class="sxs-lookup"><span data-stu-id="c46b5-118">The `Router` component is also responsible for rendering the component for the route where the user navigates.</span></span> <span data-ttu-id="c46b5-119">Il `Router` componente supporta una `OnNavigateAsync` funzionalità che può essere utilizzata in combinazione con il caricamento lazy.</span><span class="sxs-lookup"><span data-stu-id="c46b5-119">The `Router` component supports an `OnNavigateAsync` feature that can be used in conjunction with lazy loading.</span></span>

<span data-ttu-id="c46b5-120">Nel componente dell'app `Router` ( `App.razor` ):</span><span class="sxs-lookup"><span data-stu-id="c46b5-120">In the app's `Router` component (`App.razor`):</span></span>

* <span data-ttu-id="c46b5-121">Aggiungere un `OnNavigateAsync` callback.</span><span class="sxs-lookup"><span data-stu-id="c46b5-121">Add an `OnNavigateAsync` callback.</span></span> <span data-ttu-id="c46b5-122">Il `OnNavigateAsync` gestore viene richiamato quando l'utente:</span><span class="sxs-lookup"><span data-stu-id="c46b5-122">The `OnNavigateAsync` handler is invoked when the user:</span></span>
  * <span data-ttu-id="c46b5-123">Visita una route per la prima volta passando direttamente dal browser.</span><span class="sxs-lookup"><span data-stu-id="c46b5-123">Visits a route for the first time by navigating to it directly from their browser.</span></span>
  * <span data-ttu-id="c46b5-124">Passa a una nuova route usando un collegamento o una <xref:Microsoft.AspNetCore.Components.NavigationManager.NavigateTo%2A?displayProperty=nameWithType> chiamata.</span><span class="sxs-lookup"><span data-stu-id="c46b5-124">Navigates to a new route using a link or a <xref:Microsoft.AspNetCore.Components.NavigationManager.NavigateTo%2A?displayProperty=nameWithType> invocation.</span></span>
* <span data-ttu-id="c46b5-125">Se gli assembly con caricamento lazy contengono componenti instradabili, aggiungere un [elenco](xref:System.Collections.Generic.List%601) \<<xref:System.Reflection.Assembly>> (ad esempio, denominato `lazyLoadedAssemblies` ) al componente.</span><span class="sxs-lookup"><span data-stu-id="c46b5-125">If lazy-loaded assemblies contain routable components, add a [List](xref:System.Collections.Generic.List%601)\<<xref:System.Reflection.Assembly>> (for example, named `lazyLoadedAssemblies`) to the component.</span></span> <span data-ttu-id="c46b5-126">Gli assembly vengono passati di nuovo alla <xref:Microsoft.AspNetCore.Components.Routing.Router.AdditionalAssemblies> raccolta, nel caso in cui gli assembly contengano componenti instradabili.</span><span class="sxs-lookup"><span data-stu-id="c46b5-126">The assemblies are passed back to the <xref:Microsoft.AspNetCore.Components.Routing.Router.AdditionalAssemblies> collection in case the assemblies contain routable components.</span></span> <span data-ttu-id="c46b5-127">Il Framework cerca gli assembly per le route e aggiorna la raccolta di route se vengono trovate nuove route.</span><span class="sxs-lookup"><span data-stu-id="c46b5-127">The framework searches the assemblies for routes and updates the route collection if any new routes are found.</span></span>

```razor
@using System.Reflection

<Router AppAssembly="@typeof(Program).Assembly" 
    AdditionalAssemblies="@lazyLoadedAssemblies" OnNavigateAsync="@OnNavigateAsync">
    ...
</Router>

@code {
    private List<Assembly> lazyLoadedAssemblies = new List<Assembly>();

    private async Task OnNavigateAsync(NavigationContext args)
    {
    }
}
```

[!INCLUDE[](~/blazor/includes/prefer-exact-matches.md)]

<span data-ttu-id="c46b5-128">Se il `OnNavigateAsync` callback genera un'eccezione non gestita, viene richiamata l' [ Blazor interfaccia utente dell'errore](xref:blazor/fundamentals/handle-errors#detailed-errors-during-development) .</span><span class="sxs-lookup"><span data-stu-id="c46b5-128">If the `OnNavigateAsync` callback throws an unhandled exception, the [Blazor error UI](xref:blazor/fundamentals/handle-errors#detailed-errors-during-development) is invoked.</span></span>

### <a name="assembly-load-logic-in-onnavigateasync"></a><span data-ttu-id="c46b5-129">Logica di caricamento dell'assembly in `OnNavigateAsync`</span><span class="sxs-lookup"><span data-stu-id="c46b5-129">Assembly load logic in `OnNavigateAsync`</span></span>

<span data-ttu-id="c46b5-130">`OnNavigateAsync` dispone di un `NavigationContext` parametro che fornisce informazioni sull'evento di spostamento asincrono corrente, inclusi il percorso di destinazione ( `Path` ) e il token di annullamento ( `CancellationToken` ):</span><span class="sxs-lookup"><span data-stu-id="c46b5-130">`OnNavigateAsync` has a `NavigationContext` parameter that provides information about the current asynchronous navigation event, including the target path (`Path`) and the cancellation token (`CancellationToken`):</span></span>

* <span data-ttu-id="c46b5-131">La `Path` proprietà è il percorso di destinazione dell'utente relativo al percorso di base dell'applicazione, ad esempio `/robot` .</span><span class="sxs-lookup"><span data-stu-id="c46b5-131">The `Path` property is the user's destination path relative to the app's base path, such as `/robot`.</span></span>
* <span data-ttu-id="c46b5-132">L'oggetto `CancellationToken` può essere utilizzato per osservare l'annullamento dell'attività asincrona.</span><span class="sxs-lookup"><span data-stu-id="c46b5-132">The `CancellationToken` can be used to observe the cancellation of the asynchronous task.</span></span> <span data-ttu-id="c46b5-133">`OnNavigateAsync` Annulla automaticamente l'attività di spostamento attualmente in esecuzione quando l'utente passa a una pagina diversa.</span><span class="sxs-lookup"><span data-stu-id="c46b5-133">`OnNavigateAsync` automatically cancels the currently running navigation task when the user navigates to a different page.</span></span>

<span data-ttu-id="c46b5-134">All'interno di `OnNavigateAsync` implementare la logica per determinare gli assembly da caricare.</span><span class="sxs-lookup"><span data-stu-id="c46b5-134">Inside `OnNavigateAsync`, implement logic to determine the assemblies to load.</span></span> <span data-ttu-id="c46b5-135">Le opzioni includono:</span><span class="sxs-lookup"><span data-stu-id="c46b5-135">Options include:</span></span>

* <span data-ttu-id="c46b5-136">Controlli condizionali all'interno del `OnNavigateAsync` metodo.</span><span class="sxs-lookup"><span data-stu-id="c46b5-136">Conditional checks inside the `OnNavigateAsync` method.</span></span>
* <span data-ttu-id="c46b5-137">Tabella di ricerca che esegue il mapping delle route ai nomi degli assembly, inseriti nel componente o implementati all'interno del [`@code`](xref:mvc/views/razor#code) blocco.</span><span class="sxs-lookup"><span data-stu-id="c46b5-137">A lookup table that maps routes to assembly names, either injected into the component or implemented within the [`@code`](xref:mvc/views/razor#code) block.</span></span>

<span data-ttu-id="c46b5-138">`LazyAssemblyLoader` è un servizio singleton fornito dal Framework per il caricamento di assembly.</span><span class="sxs-lookup"><span data-stu-id="c46b5-138">`LazyAssemblyLoader` is a framework-provided singleton service for loading assemblies.</span></span> <span data-ttu-id="c46b5-139">Inserire `LazyAssemblyLoader` nel `Router` componente:</span><span class="sxs-lookup"><span data-stu-id="c46b5-139">Inject `LazyAssemblyLoader` into the `Router` component:</span></span>

```razor
...
@using Microsoft.AspNetCore.Components.WebAssembly.Services
@inject LazyAssemblyLoader assemblyLoader

...
```

<span data-ttu-id="c46b5-140">`LazyAssemblyLoader`Fornisce il `LoadAssembliesAsync` metodo che:</span><span class="sxs-lookup"><span data-stu-id="c46b5-140">The `LazyAssemblyLoader` provides the `LoadAssembliesAsync` method that:</span></span>

* <span data-ttu-id="c46b5-141">Usa l'interoperabilità JS per recuperare assembly tramite una chiamata di rete.</span><span class="sxs-lookup"><span data-stu-id="c46b5-141">Uses JS interop to fetch assemblies via a network call.</span></span>
* <span data-ttu-id="c46b5-142">Carica gli assembly nel runtime in esecuzione su webassembly nel browser.</span><span class="sxs-lookup"><span data-stu-id="c46b5-142">Loads assemblies into the runtime executing on WebAssembly in the browser.</span></span>

<span data-ttu-id="c46b5-143">L'implementazione del caricamento lazy del Framework supporta il caricamento lazy con il prerendering in una soluzione ospitata Blazor .</span><span class="sxs-lookup"><span data-stu-id="c46b5-143">The framework's lazy loading implementation supports lazy loading with prerendering in a hosted Blazor solution.</span></span> <span data-ttu-id="c46b5-144">Durante il prerendering, si presuppone che tutti gli assembly, inclusi quelli contrassegnati per il caricamento lazy, siano caricati.</span><span class="sxs-lookup"><span data-stu-id="c46b5-144">During prerendering, all assemblies, including those marked for lazy loading, are assumed to be loaded.</span></span> <span data-ttu-id="c46b5-145">Eseguire la registrazione manuale `LazyAssemblyLoader` nel metodo del progetto *Server* `Startup.ConfigureServices` ( `Startup.cs` ):</span><span class="sxs-lookup"><span data-stu-id="c46b5-145">Manually register `LazyAssemblyLoader` in the *Server* project's `Startup.ConfigureServices` method (`Startup.cs`):</span></span>

```csharp
services.AddScoped<LazyAssemblyLoader>();
```

### <a name="user-interaction-with-navigating-content"></a><span data-ttu-id="c46b5-146">Interazione dell'utente con il `<Navigating>` contenuto</span><span class="sxs-lookup"><span data-stu-id="c46b5-146">User interaction with `<Navigating>` content</span></span>

<span data-ttu-id="c46b5-147">Durante il caricamento degli assembly, che possono richiedere alcuni secondi, il `Router` componente può indicare all'utente che è in corso una transizione di pagina:</span><span class="sxs-lookup"><span data-stu-id="c46b5-147">While loading assemblies, which can take several seconds, the `Router` component can indicate to the user that a page transition is occurring:</span></span>

* <span data-ttu-id="c46b5-148">Aggiungere una [`@using`](xref:mvc/views/razor#using) direttiva per lo <xref:Microsoft.AspNetCore.Components.Routing?displayProperty=fullName> spazio dei nomi.</span><span class="sxs-lookup"><span data-stu-id="c46b5-148">Add an [`@using`](xref:mvc/views/razor#using) directive for the <xref:Microsoft.AspNetCore.Components.Routing?displayProperty=fullName> namespace.</span></span>
* <span data-ttu-id="c46b5-149">Aggiungere un `<Navigating>` tag al componente con markup da visualizzare durante gli eventi di transizione di pagina.</span><span class="sxs-lookup"><span data-stu-id="c46b5-149">Add a `<Navigating>` tag to the component with markup to display during page transition events.</span></span>

```razor
...
@using Microsoft.AspNetCore.Components.Routing
...

<Router ...>
    <Navigating>
        <div style="...">
            <p>Loading the requested page&hellip;</p>
        </div>
    </Navigating>
</Router>

...
```

[!INCLUDE[](~/blazor/includes/prefer-exact-matches.md)]

### <a name="handle-cancellations-in-onnavigateasync"></a><span data-ttu-id="c46b5-150">Gestisci annullamenti in `OnNavigateAsync`</span><span class="sxs-lookup"><span data-stu-id="c46b5-150">Handle cancellations in `OnNavigateAsync`</span></span>

<span data-ttu-id="c46b5-151">L' `NavigationContext` oggetto passato al `OnNavigateAsync` callback contiene un oggetto `CancellationToken` impostato quando si verifica un nuovo evento di navigazione.</span><span class="sxs-lookup"><span data-stu-id="c46b5-151">The `NavigationContext` object passed to the `OnNavigateAsync` callback contains a `CancellationToken` that's set when a new navigation event occurs.</span></span> <span data-ttu-id="c46b5-152">Il `OnNavigateAsync` callback deve generare quando questo token di annullamento è impostato in modo da evitare di continuare a eseguire il `OnNavigateAsync` callback in una navigazione obsoleta.</span><span class="sxs-lookup"><span data-stu-id="c46b5-152">The `OnNavigateAsync` callback must throw when this cancellation token is set to avoid continuing to run the `OnNavigateAsync` callback on a outdated navigation.</span></span>

<span data-ttu-id="c46b5-153">Se un utente passa a instradare un oggetto e quindi immediatamente alla Route B, l'app non deve continuare a eseguire il `OnNavigateAsync` callback per il routing a:</span><span class="sxs-lookup"><span data-stu-id="c46b5-153">If a user navigates to Route A and then immediately to Route B, the app shouldn't continue running the `OnNavigateAsync` callback for Route A:</span></span>

```razor
@inject HttpClient Http
@inject ProductCatalog Products

<Router AppAssembly="@typeof(Program).Assembly" 
    OnNavigateAsync="@OnNavigateAsync">
    ...
</Router>

@code {
    private async Task OnNavigateAsync(NavigationContext context)
    {
        if (context.Path == "/about") 
        {
            var stats = new Stats = { Page = "/about" };
            await Http.PostAsJsonAsync("api/visited", stats, context.CancellationToken);
        }
        else if (context.Path == "/store")
        {
            var productIds = [345, 789, 135, 689];

            foreach (var productId in productIds) 
            {
                context.CancellationToken.ThrowIfCancellationRequested();
                Products.Prefetch(productId);
            }
        }
    }
}
```

[!INCLUDE[](~/blazor/includes/prefer-exact-matches.md)]

> [!NOTE]
> <span data-ttu-id="c46b5-154">Se il token di annullamento in `NavigationContext` viene annullato può generare un comportamento imprevisto, ad esempio il rendering di un componente da una navigazione precedente.</span><span class="sxs-lookup"><span data-stu-id="c46b5-154">Not throwing if the cancellation token in `NavigationContext` is canceled can result in unintended behavior, such as rendering a component from a previous navigation.</span></span>

### <a name="onnavigateasync-events-and-renamed-assembly-files"></a><span data-ttu-id="c46b5-155">`OnNavigateAsync` eventi e file di assembly rinominati</span><span class="sxs-lookup"><span data-stu-id="c46b5-155">`OnNavigateAsync` events and renamed assembly files</span></span>

<span data-ttu-id="c46b5-156">Il caricatore di risorse si basa sui nomi degli assembly definiti nel `blazor.boot.json` file.</span><span class="sxs-lookup"><span data-stu-id="c46b5-156">The resource loader relies on the assembly names that are defined in the `blazor.boot.json` file.</span></span> <span data-ttu-id="c46b5-157">Se gli [assembly vengono rinominati](xref:blazor/host-and-deploy/webassembly#change-the-filename-extension-of-dll-files), i nomi degli assembly utilizzati nei `OnNavigateAsync` metodi e i nomi degli assembly nel `blazor.boot.json` file non sono sincronizzati.</span><span class="sxs-lookup"><span data-stu-id="c46b5-157">If [assemblies are renamed](xref:blazor/host-and-deploy/webassembly#change-the-filename-extension-of-dll-files), the assembly names used in `OnNavigateAsync` methods and the assembly names in the `blazor.boot.json` file are out of sync.</span></span>

<span data-ttu-id="c46b5-158">Per risolvere il problema:</span><span class="sxs-lookup"><span data-stu-id="c46b5-158">To rectify this:</span></span>

* <span data-ttu-id="c46b5-159">Verificare se l'app è in esecuzione nell'ambiente di produzione quando si determinano i nomi degli assembly da usare.</span><span class="sxs-lookup"><span data-stu-id="c46b5-159">Check to see if the app is running in the Production environment when determining which assembly names to use.</span></span>
* <span data-ttu-id="c46b5-160">Archiviare i nomi di assembly rinominati in un file separato e leggere da tale file per determinare il nome dell'assembly da usare `LazyLoadAssemblyService` nei `OnNavigateAsync` metodi e.</span><span class="sxs-lookup"><span data-stu-id="c46b5-160">Store the renamed assembly names in a separate file and read from that file to determine what assembly name to use in the `LazyLoadAssemblyService` and `OnNavigateAsync` methods.</span></span>

### <a name="complete-example"></a><span data-ttu-id="c46b5-161">Esempio completo</span><span class="sxs-lookup"><span data-stu-id="c46b5-161">Complete example</span></span>

<span data-ttu-id="c46b5-162">Il componente completo seguente `Router` illustra il caricamento dell' `GrantImaharaRobotControls.dll` assembly quando l'utente passa a `/robot` .</span><span class="sxs-lookup"><span data-stu-id="c46b5-162">The following complete `Router` component demonstrates loading the `GrantImaharaRobotControls.dll` assembly when the user navigates to `/robot`.</span></span> <span data-ttu-id="c46b5-163">Durante le transizioni di pagina, all'utente viene visualizzato un messaggio con stile.</span><span class="sxs-lookup"><span data-stu-id="c46b5-163">During page transitions, a styled message is displayed to the user.</span></span>

```razor
@using System.Reflection
@using Microsoft.AspNetCore.Components.Routing
@using Microsoft.AspNetCore.Components.WebAssembly.Services
@inject LazyAssemblyLoader assemblyLoader

<Router AppAssembly="@typeof(Program).Assembly" 
    AdditionalAssemblies="@lazyLoadedAssemblies" OnNavigateAsync="@OnNavigateAsync">
    <Navigating>
        <div style="padding:20px;background-color:blue;color:white">
            <p>Loading the requested page&hellip;</p>
        </div>
    </Navigating>
    <Found Context="routeData">
        <RouteView RouteData="@routeData" DefaultLayout="@typeof(MainLayout)" />
    </Found>
    <NotFound>
        <LayoutView Layout="@typeof(MainLayout)">
            <p>Sorry, there's nothing at this address.</p>
        </LayoutView>
    </NotFound>
</Router>

@code {
    private List<Assembly> lazyLoadedAssemblies = new List<Assembly>();

    private async Task OnNavigateAsync(NavigationContext args)
    {
        try
        {
            if (args.Path.EndsWith("/robot"))
            {
                var assemblies = await assemblyLoader.LoadAssembliesAsync(
                    new List<string>() { "GrantImaharaRobotControls.dll" });
                lazyLoadedAssemblies.AddRange(assemblies);
            }
        }
        catch (Exception ex)
        {
            ...
        }
    }
}
```

[!INCLUDE[](~/blazor/includes/prefer-exact-matches.md)]

## <a name="troubleshoot"></a><span data-ttu-id="c46b5-164">Risolvere problemi</span><span class="sxs-lookup"><span data-stu-id="c46b5-164">Troubleshoot</span></span>

* <span data-ttu-id="c46b5-165">Se si verifica un rendering imprevisto (ad esempio, viene eseguito il rendering di un componente di una navigazione precedente), verificare che il codice venga generato se è impostato il token di annullamento.</span><span class="sxs-lookup"><span data-stu-id="c46b5-165">If unexpected rendering occurs (for example, a component from a previous navigation is rendered), confirm that the code throws if the cancellation token is set.</span></span>
* <span data-ttu-id="c46b5-166">Se gli assembly vengono ancora caricati all'avvio dell'applicazione, verificare che l'assembly sia contrassegnato come Lazy Loaded nel file di progetto.</span><span class="sxs-lookup"><span data-stu-id="c46b5-166">If assemblies are still loaded at application start, check that the assembly is marked as lazy loaded in the project file.</span></span>

## <a name="additional-resources"></a><span data-ttu-id="c46b5-167">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="c46b5-167">Additional resources</span></span>

* <xref:blazor/webassembly-performance-best-practices>
