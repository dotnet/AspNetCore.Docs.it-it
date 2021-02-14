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
# <a name="lazy-load-assemblies-in-aspnet-core-blazor-webassembly"></a>Assembly di caricamento lazy in ASP.NET Core Blazor WebAssembly

Blazor WebAssembly le prestazioni di avvio dell'app possono essere migliorate posticipando il caricamento di alcuni assembly di applicazioni fino a quando non sono necessarie, operazione denominata *caricamento lazy*. Ad esempio, gli assembly che vengono utilizzati solo per il rendering di un singolo componente possono essere impostati per il caricamento solo se l'utente passa a tale componente. Dopo il caricamento, gli assembly vengono memorizzati nella cache sul lato client e sono disponibili per tutte le navigazioni future.

Blazorla funzionalità di caricamento lazy consente di contrassegnare gli assembly dell'app per il caricamento lazy, che carica gli assembly in fase di esecuzione quando l'utente passa a una route specifica. La funzionalità è costituita dalle modifiche apportate al file di progetto e dalle modifiche apportate al router dell'applicazione.

> [!NOTE]
> Il caricamento lazy dell'assembly non avvantaggia Blazor Server le app perché gli assembly non vengono scaricati nel client in un' Blazor Server app.

## <a name="project-file"></a>File di progetto

Contrassegnare gli assembly per il caricamento lazy nel file di progetto dell'app ( `.csproj` ) usando l' `BlazorWebAssemblyLazyLoad` elemento. Usare il nome dell'assembly con l' `.dll` estensione. Il Blazor Framework impedisce il caricamento degli assembly specificati da questo gruppo di elementi all'avvio dell'app. Nell'esempio seguente viene contrassegnato un assembly personalizzato di grandi dimensioni ( `GrantImaharaRobotControls.dll` ) per il caricamento lazy. Se un assembly contrassegnato per il caricamento lazy presenta dipendenze, è necessario contrassegnarlo anche per il caricamento lazy nel file di progetto.

```xml
<ItemGroup>
  <BlazorWebAssemblyLazyLoad Include="GrantImaharaRobotControls.dll" />
</ItemGroup>
```

## <a name="router-component"></a>Componente `Router`

Blazoril `Router` componente indica gli assembly in cui vengono Blazor cercati i componenti instradabili. Il `Router` componente è inoltre responsabile del rendering del componente per la route in cui l'utente si sposta. Il `Router` componente supporta una `OnNavigateAsync` funzionalità che può essere utilizzata in combinazione con il caricamento lazy.

Nel componente dell'app `Router` ( `App.razor` ):

* Aggiungere un `OnNavigateAsync` callback. Il `OnNavigateAsync` gestore viene richiamato quando l'utente:
  * Visita una route per la prima volta passando direttamente dal browser.
  * Passa a una nuova route usando un collegamento o una <xref:Microsoft.AspNetCore.Components.NavigationManager.NavigateTo%2A?displayProperty=nameWithType> chiamata.
* Se gli assembly con caricamento lazy contengono componenti instradabili, aggiungere un [elenco](xref:System.Collections.Generic.List%601) \<<xref:System.Reflection.Assembly>> (ad esempio, denominato `lazyLoadedAssemblies` ) al componente. Gli assembly vengono passati di nuovo alla <xref:Microsoft.AspNetCore.Components.Routing.Router.AdditionalAssemblies> raccolta, nel caso in cui gli assembly contengano componenti instradabili. Il Framework cerca gli assembly per le route e aggiorna la raccolta di route se vengono trovate nuove route.

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

Se il `OnNavigateAsync` callback genera un'eccezione non gestita, viene richiamata l' [ Blazor interfaccia utente dell'errore](xref:blazor/fundamentals/handle-errors#detailed-errors-during-development) .

### <a name="assembly-load-logic-in-onnavigateasync"></a>Logica di caricamento dell'assembly in `OnNavigateAsync`

`OnNavigateAsync` dispone di un `NavigationContext` parametro che fornisce informazioni sull'evento di spostamento asincrono corrente, inclusi il percorso di destinazione ( `Path` ) e il token di annullamento ( `CancellationToken` ):

* La `Path` proprietà è il percorso di destinazione dell'utente relativo al percorso di base dell'applicazione, ad esempio `/robot` .
* L'oggetto `CancellationToken` può essere utilizzato per osservare l'annullamento dell'attività asincrona. `OnNavigateAsync` Annulla automaticamente l'attività di spostamento attualmente in esecuzione quando l'utente passa a una pagina diversa.

All'interno di `OnNavigateAsync` implementare la logica per determinare gli assembly da caricare. Le opzioni includono:

* Controlli condizionali all'interno del `OnNavigateAsync` metodo.
* Tabella di ricerca che esegue il mapping delle route ai nomi degli assembly, inseriti nel componente o implementati all'interno del [`@code`](xref:mvc/views/razor#code) blocco.

`LazyAssemblyLoader` è un servizio singleton fornito dal Framework per il caricamento di assembly. Inserire `LazyAssemblyLoader` nel `Router` componente:

```razor
...
@using Microsoft.AspNetCore.Components.WebAssembly.Services
@inject LazyAssemblyLoader assemblyLoader

...
```

`LazyAssemblyLoader`Fornisce il `LoadAssembliesAsync` metodo che:

* Usa l'interoperabilità JS per recuperare assembly tramite una chiamata di rete.
* Carica gli assembly nel runtime in esecuzione su webassembly nel browser.

L'implementazione del caricamento lazy del Framework supporta il caricamento lazy con il prerendering in una soluzione ospitata Blazor . Durante il prerendering, si presuppone che tutti gli assembly, inclusi quelli contrassegnati per il caricamento lazy, siano caricati. Eseguire la registrazione manuale `LazyAssemblyLoader` nel metodo del progetto *Server* `Startup.ConfigureServices` ( `Startup.cs` ):

```csharp
services.AddScoped<LazyAssemblyLoader>();
```

### <a name="user-interaction-with-navigating-content"></a>Interazione dell'utente con il `<Navigating>` contenuto

Durante il caricamento degli assembly, che possono richiedere alcuni secondi, il `Router` componente può indicare all'utente che è in corso una transizione di pagina:

* Aggiungere una [`@using`](xref:mvc/views/razor#using) direttiva per lo <xref:Microsoft.AspNetCore.Components.Routing?displayProperty=fullName> spazio dei nomi.
* Aggiungere un `<Navigating>` tag al componente con markup da visualizzare durante gli eventi di transizione di pagina.

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

### <a name="handle-cancellations-in-onnavigateasync"></a>Gestisci annullamenti in `OnNavigateAsync`

L' `NavigationContext` oggetto passato al `OnNavigateAsync` callback contiene un oggetto `CancellationToken` impostato quando si verifica un nuovo evento di navigazione. Il `OnNavigateAsync` callback deve generare quando questo token di annullamento è impostato in modo da evitare di continuare a eseguire il `OnNavigateAsync` callback in una navigazione obsoleta.

Se un utente passa a instradare un oggetto e quindi immediatamente alla Route B, l'app non deve continuare a eseguire il `OnNavigateAsync` callback per il routing a:

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
> Se il token di annullamento in `NavigationContext` viene annullato può generare un comportamento imprevisto, ad esempio il rendering di un componente da una navigazione precedente.

### <a name="onnavigateasync-events-and-renamed-assembly-files"></a>`OnNavigateAsync` eventi e file di assembly rinominati

Il caricatore di risorse si basa sui nomi degli assembly definiti nel `blazor.boot.json` file. Se gli [assembly vengono rinominati](xref:blazor/host-and-deploy/webassembly#change-the-filename-extension-of-dll-files), i nomi degli assembly utilizzati nei `OnNavigateAsync` metodi e i nomi degli assembly nel `blazor.boot.json` file non sono sincronizzati.

Per risolvere il problema:

* Verificare se l'app è in esecuzione nell'ambiente di produzione quando si determinano i nomi degli assembly da usare.
* Archiviare i nomi di assembly rinominati in un file separato e leggere da tale file per determinare il nome dell'assembly da usare `LazyLoadAssemblyService` nei `OnNavigateAsync` metodi e.

### <a name="complete-example"></a>Esempio completo

Il componente completo seguente `Router` illustra il caricamento dell' `GrantImaharaRobotControls.dll` assembly quando l'utente passa a `/robot` . Durante le transizioni di pagina, all'utente viene visualizzato un messaggio con stile.

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

## <a name="troubleshoot"></a>Risolvere problemi

* Se si verifica un rendering imprevisto (ad esempio, viene eseguito il rendering di un componente di una navigazione precedente), verificare che il codice venga generato se è impostato il token di annullamento.
* Se gli assembly vengono ancora caricati all'avvio dell'applicazione, verificare che l'assembly sia contrassegnato come Lazy Loaded nel file di progetto.

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:blazor/webassembly-performance-best-practices>
