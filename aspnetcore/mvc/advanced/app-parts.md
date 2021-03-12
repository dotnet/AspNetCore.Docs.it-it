---
title: Condividi controller, visualizzazioni, Razor pagine e altro ancora con le parti dell'applicazione in ASP.NET Core
author: rick-anderson
description: Condividi controller, Visualizza, Razor pagine e altro con le parti dell'applicazione in ASP.NET Core
ms.author: riande
ms.date: 11/11/2019
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
uid: mvc/extensibility/app-parts
ms.openlocfilehash: 23bc1db6a184e7babe87e2d311a8ac4a59e78dd0
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102588360"
---
# <a name="share-controllers-views-razor-pages-and-more-with-application-parts"></a>Condividi controller, visualizzazioni, Razor pagine e altro ancora con le parti dell'applicazione

::: moniker range=">= aspnetcore-3.0"

Autore: [Rick Anderson](https://twitter.com/RickAndMSFT)

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/mvc/advanced/app-parts) ([procedura per il download](xref:index#how-to-download-a-sample))

Una *parte dell'applicazione* è un'astrazione sulle risorse di un'app. Le parti dell'applicazione consentono ASP.NET Core di individuare controller, componenti di visualizzazione, helper tag, Razor pagine, origini di compilazione Razor e altro ancora. <xref:Microsoft.AspNetCore.Mvc.ApplicationParts.AssemblyPart> è una parte dell'applicazione. `AssemblyPart` Incapsula un riferimento all'assembly ed espone i tipi e i riferimenti di compilazione.

I [provider di funzionalità](#fp) funzionano con le parti dell'applicazione per popolare le funzionalità di un'app ASP.NET Core. Il caso d'uso principale per le parti dell'applicazione consiste nel configurare un'app per individuare (o evitare il caricamento) ASP.NET Core funzionalità da un assembly. Ad esempio, potrebbe essere necessario condividere le funzionalità comuni tra più app. Con le parti dell'applicazione è possibile condividere un assembly (DLL) contenente controller, visualizzazioni, Razor pagine, origini di compilazione Razor, helper tag e altro ancora con più app. La condivisione di un assembly è preferibile alla duplicazione del codice in più progetti.

ASP.NET Core le app caricano le funzionalità da <xref:System.Web.WebPages.ApplicationPart> . La <xref:Microsoft.AspNetCore.Mvc.ApplicationParts.AssemblyPart> classe rappresenta una parte dell'applicazione supportata da un assembly.

## <a name="load-aspnet-core-features"></a>Carica ASP.NET Core funzionalità

Usare le <xref:Microsoft.AspNetCore.Mvc.ApplicationParts> <xref:Microsoft.AspNetCore.Mvc.ApplicationParts.AssemblyPart> classi e per individuare e caricare ASP.NET Core funzionalità (controller, componenti di visualizzazione e così via). Consente di tenere <xref:Microsoft.AspNetCore.Mvc.ApplicationParts.ApplicationPartManager> traccia delle parti dell'applicazione e dei provider di funzionalità disponibili. `ApplicationPartManager` è configurato in `Startup.ConfigureServices` :

[!code-csharp[](./app-parts/3.0sample1/WebAppParts/Startup.cs?name=snippet)]

Il codice seguente offre un approccio alternativo alla configurazione di `ApplicationPartManager` utilizzando `AssemblyPart` :

[!code-csharp[](./app-parts/3.0sample1/WebAppParts/Startup2.cs?name=snippet)]

I due esempi di codice precedenti caricano `SharedController` da un assembly. L'oggetto `SharedController` non è presente nel progetto dell'app. Vedere il download dell'esempio di [soluzione WebAppParts](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/mvc/advanced/app-parts/3.0sample1/WebAppParts) .

### <a name="include-views"></a>Includi visualizzazioni

Utilizzare una [ Razor libreria di classi](xref:razor-pages/ui-class) per includere viste nell'assembly.

### <a name="prevent-loading-resources"></a>Impedisci il caricamento delle risorse

Le parti dell'applicazione possono essere utilizzate per *evitare* il caricamento di risorse in un assembly o in un percorso specifico. Aggiungere o rimuovere membri della  <xref:Microsoft.AspNetCore.Mvc.ApplicationParts> raccolta per nascondere o rendere disponibili risorse. L'ordine delle voci nella raccolta `ApplicationParts` non è importante. Configurare `ApplicationPartManager` prima di usarlo per configurare i servizi nel contenitore. Configurare, ad esempio, `ApplicationPartManager` prima di richiamare `AddControllersAsServices` . Chiamare `Remove` sulla `ApplicationParts` raccolta per rimuovere una risorsa.

`ApplicationPartManager`Include le parti per:

* Assembly dell'app e assembly dipendenti.
* `Microsoft.AspNetCore.Mvc.ApplicationParts.CompiledRazorAssemblyPart`
* `Microsoft.AspNetCore.Mvc.Razor.RuntimeCompilation`
* `Microsoft.AspNetCore.Mvc.TagHelpers`.
* `Microsoft.AspNetCore.Mvc.Razor`.

<a name="fp"></a>

## <a name="feature-providers"></a>Provider di funzionalità

I provider di funzionalità dell'applicazione esaminano le parti dell'applicazione e forniscono funzionalità per tali parti. Sono disponibili provider di funzionalità predefiniti per le funzionalità di ASP.NET Core seguenti:

* <xref:Microsoft.AspNetCore.Mvc.Controllers.ControllerFeatureProvider>
* <xref:Microsoft.AspNetCore.Mvc.Razor.TagHelpers.TagHelperFeatureProvider>
* <xref:Microsoft.AspNetCore.Mvc.Razor.Compilation.MetadataReferenceFeatureProvider>
* <xref:Microsoft.AspNetCore.Mvc.Razor.Compilation.ViewsFeatureProvider>
* `internal class`[ Razor CompiledItemFeatureProvider](https://github.com/dotnet/AspNetCore/blob/main/src/Mvc/Mvc.Razor/src/ApplicationParts/RazorCompiledItemFeatureProvider.cs#L14)

I provider di funzionalità ereditano da <xref:Microsoft.AspNetCore.Mvc.ApplicationParts.IApplicationFeatureProvider`1>, dove `T` è il tipo della funzionalità. I provider di funzionalità possono essere implementati per uno qualsiasi dei tipi di funzionalità elencati in precedenza. L'ordine dei provider di funzionalità in `ApplicationPartManager.FeatureProviders` può influisca sul comportamento in fase di esecuzione. I provider aggiunti successivamente possono rispondere alle azioni eseguite dai provider aggiunti precedentemente.

### <a name="display-available-features"></a>Visualizzare funzionalità disponibili

Le funzionalità disponibili per un'app possono essere enumerate richiedendo un `ApplicationPartManager` inserimento delle [dipendenze](../../fundamentals/dependency-injection.md):

[!code-csharp[](./app-parts/sample2/AppPartsSample/Controllers/FeaturesController.cs?highlight=16,25-27)]

L' [esempio di download](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/mvc/advanced/app-parts/sample2) usa il codice precedente per visualizzare le funzionalità dell'app:

```text
Controllers:
    - FeaturesController
    - HomeController
    - HelloController
    - GenericController`1
    - GenericController`1
Tag Helpers:
    - PrerenderTagHelper
    - AnchorTagHelper
    - CacheTagHelper
    - DistributedCacheTagHelper
    - EnvironmentTagHelper
    - Additional Tag Helpers omitted for brevity.
View Components:
    - SampleViewComponent
```

## <a name="discovery-in-application-parts"></a>Individuazione nelle parti dell'applicazione

Gli errori HTTP 404 non sono rari durante lo sviluppo con le parti dell'applicazione. Questi errori sono in genere causati da un requisito essenziale per il modo in cui vengono individuate le parti delle applicazioni. Se l'app restituisce un errore HTTP 404, verificare che siano stati soddisfatti i requisiti seguenti:

* L' `applicationName` impostazione deve essere impostata sull'assembly radice utilizzato per l'individuazione. L'assembly radice utilizzato per l'individuazione è in genere l'assembly del punto di ingresso.
* L'assembly radice deve avere un riferimento alle parti utilizzate per l'individuazione. Il riferimento può essere diretto o transitivo.
* L'assembly radice deve fare riferimento a Web SDK. Il Framework dispone di una logica che timbra gli attributi nell'assembly radice utilizzati per l'individuazione.

::: moniker-end

::: moniker range="< aspnetcore-3.0"

Autore: [Rick Anderson](https://twitter.com/RickAndMSFT)

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/mvc/advanced/app-parts) ([procedura per il download](xref:index#how-to-download-a-sample))

Una *parte dell'applicazione* è un'astrazione sulle risorse di un'app. Le parti dell'applicazione consentono ASP.NET Core di individuare controller, componenti di visualizzazione, helper tag, Razor pagine, origini di compilazione Razor e altro ancora. [AssemblyPart](/dotnet/api/microsoft.aspnetcore.mvc.applicationparts.assemblypart#Microsoft_AspNetCore_Mvc_ApplicationParts_AssemblyPart) è una parte dell'applicazione. `AssemblyPart` Incapsula un riferimento all'assembly ed espone i tipi e i riferimenti di compilazione.

I *provider di funzionalità* funzionano con le parti dell'applicazione per popolare le funzionalità di un'app ASP.NET Core. Il caso d'uso principale per le parti dell'applicazione consiste nel configurare un'app per individuare (o evitare il caricamento) ASP.NET Core funzionalità da un assembly. Ad esempio, potrebbe essere necessario condividere le funzionalità comuni tra più app. Con le parti dell'applicazione è possibile condividere un assembly (DLL) contenente controller, visualizzazioni, Razor pagine, origini di compilazione Razor, helper tag e altro ancora con più app. La condivisione di un assembly è preferibile alla duplicazione del codice in più progetti.

ASP.NET Core le app caricano le funzionalità da <xref:System.Web.WebPages.ApplicationPart> . La <xref:Microsoft.AspNetCore.Mvc.ApplicationParts.AssemblyPart> classe rappresenta una parte dell'applicazione supportata da un assembly.

## <a name="load-aspnet-core-features"></a>Carica ASP.NET Core funzionalità

Usare le `ApplicationPart` `AssemblyPart` classi e per individuare e caricare ASP.NET Core funzionalità (controller, componenti di visualizzazione e così via). Consente di tenere <xref:Microsoft.AspNetCore.Mvc.ApplicationParts.ApplicationPartManager> traccia delle parti dell'applicazione e dei provider di funzionalità disponibili. `ApplicationPartManager` è configurato in `Startup.ConfigureServices` :

[!code-csharp[](./app-parts/sample1/WebAppParts/Startup.cs?name=snippet)]

Il codice seguente offre un approccio alternativo alla configurazione di `ApplicationPartManager` utilizzando `AssemblyPart` :

[!code-csharp[](./app-parts/sample1/WebAppParts/Startup2.cs?name=snippet)]

I due esempi di codice precedenti caricano `SharedController` da un assembly. L'oggetto `SharedController` non è presente nel progetto dell'applicazione. Vedere il download dell'esempio di [soluzione WebAppParts](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/mvc/advanced/app-parts/sample1/WebAppParts) .

### <a name="include-views"></a>Includi visualizzazioni

Utilizzare una [ Razor libreria di classi](xref:razor-pages/ui-class) per includere viste nell'assembly.

### <a name="prevent-loading-resources"></a>Impedisci il caricamento delle risorse

Le parti dell'applicazione possono essere utilizzate per *evitare* il caricamento di risorse in un assembly o in un percorso specifico. Aggiungere o rimuovere membri della  <xref:Microsoft.AspNetCore.Mvc.ApplicationParts> raccolta per nascondere o rendere disponibili risorse. L'ordine delle voci nella raccolta `ApplicationParts` non è importante. Configurare `ApplicationPartManager` prima di usarlo per configurare i servizi nel contenitore. Configurare, ad esempio, `ApplicationPartManager` prima di richiamare `AddControllersAsServices` . Chiamare `Remove` sulla `ApplicationParts` raccolta per rimuovere una risorsa.

Il codice seguente usa <xref:Microsoft.AspNetCore.Mvc.ApplicationParts> per rimuovere `MyDependentLibrary` dall'app: [!code-csharp[](./app-parts/sample1/WebAppParts/StartupRm.cs?name=snippet)]

`ApplicationPartManager`Include le parti per:

* Assembly dell'app e assembly dipendenti.
* `Microsoft.AspNetCore.Mvc.TagHelpers`.
* `Microsoft.AspNetCore.Mvc.Razor`.

## <a name="application-feature-providers"></a>Provider di funzionalità dell'applicazione

I provider di funzionalità dell'applicazione esaminano le parti dell'applicazione e forniscono funzionalità per tali parti. Sono disponibili provider di funzionalità predefiniti per le funzionalità di ASP.NET Core seguenti:

* [Controllers](/dotnet/api/microsoft.aspnetcore.mvc.controllers.controllerfeatureprovider)
* [Helper per i tag](/dotnet/api/microsoft.aspnetcore.mvc.razor.taghelpers.taghelperfeatureprovider)
* [Visualizza componenti](/dotnet/api/microsoft.aspnetcore.mvc.viewcomponents.viewcomponentfeatureprovider)

I provider di funzionalità ereditano da <xref:Microsoft.AspNetCore.Mvc.ApplicationParts.IApplicationFeatureProvider`1>, dove `T` è il tipo della funzionalità. I provider di funzionalità possono essere implementati per uno qualsiasi dei tipi di funzionalità elencati in precedenza. L'ordine dei provider di funzionalità in `ApplicationPartManager.FeatureProviders` può influisca sul comportamento in fase di esecuzione. I provider aggiunti successivamente possono rispondere alle azioni eseguite dai provider aggiunti precedentemente.

### <a name="display-available-features"></a>Visualizzare funzionalità disponibili

Le funzionalità disponibili per un'app possono essere enumerate richiedendo un `ApplicationPartManager` inserimento delle [dipendenze](../../fundamentals/dependency-injection.md):

[!code-csharp[](./app-parts/sample2/AppPartsSample/Controllers/FeaturesController.cs?highlight=16,25-27)]

L' [esempio di download](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/mvc/advanced/app-parts/sample2) usa il codice precedente per visualizzare le funzionalità dell'app:

```text
Controllers:
    - FeaturesController
    - HomeController
    - HelloController
    - GenericController`1
    - GenericController`1
Tag Helpers:
    - PrerenderTagHelper
    - AnchorTagHelper
    - CacheTagHelper
    - DistributedCacheTagHelper
    - EnvironmentTagHelper
    - Additional Tag Helpers omitted for brevity.
View Components:
    - SampleViewComponent
```

## <a name="discovery-in-application-parts"></a>Individuazione nelle parti dell'applicazione

Gli errori HTTP 404 non sono rari durante lo sviluppo con le parti dell'applicazione. Questi errori sono in genere causati da un requisito essenziale per il modo in cui vengono individuate le parti delle applicazioni. Se l'app restituisce un errore HTTP 404, verificare che siano stati soddisfatti i requisiti seguenti:

* L' `applicationName` impostazione deve essere impostata sull'assembly radice utilizzato per l'individuazione. L'assembly radice utilizzato per l'individuazione è in genere l'assembly del punto di ingresso.
* L'assembly radice deve avere un riferimento alle parti utilizzate per l'individuazione. Il riferimento può essere diretto o transitivo.
* L'assembly radice deve fare riferimento a Web SDK.
  * Il framework ASP.NET Core dispone di una logica di compilazione personalizzata che timbra gli attributi nell'assembly radice utilizzati per l'individuazione.

::: moniker-end
