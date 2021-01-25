---
title: BlazorStruttura del progetto ASP.NET Core
author: guardrex
description: Informazioni sulla Blazor struttura del progetto di app ASP.NET Core.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 01/19/2021
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
uid: blazor/project-structure
ms.openlocfilehash: 958fa23a1befac3696d850d5409d4021dd109c22
ms.sourcegitcommit: 610936e4d3507f7f3d467ed7859ab9354ec158ba
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 01/25/2021
ms.locfileid: "98751542"
---
# <a name="aspnet-core-no-locblazor-project-structure"></a>BlazorStruttura del progetto ASP.NET Core

Di [Daniel Roth](https://github.com/danroth27) e [Luke Latham](https://github.com/guardrex)

Questo articolo descrive i file e le cartelle che costituiscono un' Blazor app generata dai Blazor modelli di progetto.

## Blazor WebAssembly

Il Blazor WebAssembly modello ( `blazorwasm` ) crea i file e la struttura di directory iniziali per un' Blazor WebAssembly app. L'app viene popolata con il codice dimostrativo per un `FetchData` componente che carica i dati da un asset statico, `weather.json` e l'interazione dell'utente con un `Counter` componente.

* `Pages` cartella: contiene i componenti e le pagine instradabili ( `.razor` ) che costituiscono l' Blazor app. La route per ogni pagina viene specificata utilizzando la [`@page`](xref:mvc/views/razor#page) direttiva. Il modello include i componenti seguenti:
  * `Counter` Component ( `Counter.razor` ): implementa la pagina del contatore.
  * `FetchData` Component ( `FetchData.razor` ): implementa la pagina Recupera dati.
  * `Index` Component ( `Index.razor` ): implementa la Home page.
  
* `Properties/launchSettings.json`: Include la [configurazione dell'ambiente di sviluppo](xref:fundamentals/environments#development-and-launchsettingsjson).

::: moniker range=">= aspnetcore-5.0"

* `Shared` cartella: contiene i componenti e i fogli di stile condivisi seguenti:
  * `MainLayout` Component ( `MainLayout.razor` ): [componente del layout](xref:blazor/layouts)dell'app.
  * `MainLayout.razor.css`: Foglio di stile per il layout principale dell'app.
  * `NavMenu` Component ( `NavMenu.razor` ): implementa la navigazione nell'intestazione laterale. Include il [ `NavLink` componente](xref:blazor/fundamentals/routing#navlink-and-navmenu-components) ( <xref:Microsoft.AspNetCore.Components.Routing.NavLink> ) che esegue il rendering dei collegamenti di navigazione ad altri Razor componenti. Il <xref:Microsoft.AspNetCore.Components.Routing.NavLink> componente indica automaticamente uno stato selezionato quando viene caricato il componente, che consente all'utente di comprendere il componente attualmente visualizzato.
  * `NavMenu.razor.css`: Foglio di stile per il menu di navigazione dell'app.
  * `SurveyPrompt` Component ( `SurveyPrompt.razor` ): Blazor componente Survey.

::: moniker-end

::: moniker range="< aspnetcore-5.0"

* `Shared` cartella: contiene i componenti condivisi seguenti:
  * `MainLayout` Component ( `MainLayout.razor` ): [componente del layout](xref:blazor/layouts)dell'app.
  * `NavMenu` Component ( `NavMenu.razor` ): implementa la navigazione nell'intestazione laterale. Include il [ `NavLink` componente](xref:blazor/fundamentals/routing#navlink-and-navmenu-components) ( <xref:Microsoft.AspNetCore.Components.Routing.NavLink> ) che esegue il rendering dei collegamenti di navigazione ad altri Razor componenti. Il <xref:Microsoft.AspNetCore.Components.Routing.NavLink> componente indica automaticamente uno stato selezionato quando viene caricato il componente, che consente all'utente di comprendere il componente attualmente visualizzato.
  * `SurveyPrompt` Component ( `SurveyPrompt.razor` ): Blazor componente Survey.
  
::: moniker-end

::: moniker range=">= aspnetcore-5.0"

* `wwwroot`: La cartella [radice Web](xref:fundamentals/index#web-root) per l'app contenente le risorse statiche pubbliche dell'app, inclusi i `appsettings.json` file di impostazioni dell'app Environmental e le impostazioni di [configurazione](xref:blazor/fundamentals/configuration). La pagina Web `index.html` è la pagina radice dell'app implementata come pagina HTML:
  * Quando viene inizialmente richiesta una pagina dell'app, il rendering della pagina viene eseguito e restituito nella risposta.
  * La pagina specifica la posizione in cui `App` viene eseguito il rendering del componente radice. Il rendering del componente viene eseguito in corrispondenza della posizione dell' `div` elemento DOM con un valore `id` di `app` ( `<div id="app">Loading...</div>` ).

::: moniker-end

::: moniker range="< aspnetcore-5.0"

* `wwwroot`: La cartella [radice Web](xref:fundamentals/index#web-root) per l'app contenente le risorse statiche pubbliche dell'app, inclusi i `appsettings.json` file di impostazioni dell'app Environmental e le impostazioni di [configurazione](xref:blazor/fundamentals/configuration). La pagina Web `index.html` è la pagina radice dell'app implementata come pagina HTML:
  * Quando viene inizialmente richiesta una pagina dell'app, il rendering della pagina viene eseguito e restituito nella risposta.
  * La pagina specifica la posizione in cui `App` viene eseguito il rendering del componente radice. Il rendering del componente viene eseguito in corrispondenza della posizione dell' `app` elemento DOM ( `<app>Loading...</app>` ).

::: moniker-end

> [!NOTE]
> I file JavaScript (JS) aggiunti al `wwwroot/index.html` file devono essere visualizzati prima del `</body>` tag di chiusura. L'ordine con cui il codice JS personalizzato viene caricato dai file JS è importante in alcuni scenari. Ad esempio, assicurarsi che i file JS con metodi di interoperabilità siano inclusi prima dei Blazor file di framework JS.

* `_Imports.razor`: Include Razor direttive comuni da includere nei componenti dell'app ( `.razor` ), ad esempio le [`@using`](xref:mvc/views/razor#using) direttive per gli spazi dei nomi.

* `App.razor`: Componente radice dell'app che imposta il routing sul lato client usando il <xref:Microsoft.AspNetCore.Components.Routing.Router> componente. Il <xref:Microsoft.AspNetCore.Components.Routing.Router> componente intercetta la navigazione del browser ed esegue il rendering della pagina che corrisponde all'indirizzo richiesto.

::: moniker range=">= aspnetcore-5.0"

* `Program.cs`: Il punto di ingresso dell'app che configura l'host webassembly:
  
  * Il `App` componente è il componente radice dell'app. Il `App` componente viene specificato come `div` elemento DOM con un valore `id` di `app` ( `<div id="app">Loading...</div>` in `wwwroot/index.html` ) per la raccolta di componenti radice ( `builder.RootComponents.Add<App>("#app")` ).
  * I [Servizi](xref:blazor/fundamentals/dependency-injection) vengono aggiunti e configurati, ad esempio `builder.Services.AddSingleton<IMyDependency, MyDependency>()` .

::: moniker-end

::: moniker range="< aspnetcore-5.0"

* `Program.cs`: Il punto di ingresso dell'app che configura l'host webassembly:

  * Il `App` componente è il componente radice dell'app. Il `App` componente viene specificato come `app` elemento DOM ( `<app>Loading...</app>` in `wwwroot/index.html` ) per la raccolta di componenti radice ( `builder.RootComponents.Add<App>("app")` ).
  * I [Servizi](xref:blazor/fundamentals/dependency-injection) vengono aggiunti e configurati, ad esempio `builder.Services.AddSingleton<IMyDependency, MyDependency>()` .

::: moniker-end

## Blazor Server

Il Blazor Server modello ( `blazorserver` ) crea i file e la struttura di directory iniziali per un' Blazor Server app. L'app viene popolata con il codice dimostrativo per un `FetchData` componente che carica i dati da un servizio registrato, `WeatherForecastService` , e l'interazione dell'utente con un `Counter` componente.

* `Data` cartella: contiene la `WeatherForecast` classe e l'implementazione dell'oggetto `WeatherForecastService` che fornisce dati meteorologici di esempio al `FetchData` componente dell'app.

* `Pages` cartella: contiene i componenti e le pagine instradabili ( `.razor` ) che costituiscono l' Blazor app e la Razor pagina radice di un' Blazor Server app. La route per ogni pagina viene specificata utilizzando la [`@page`](xref:mvc/views/razor#page) direttiva. Il modello include quanto segue:
  * `_Host.cshtml`: Pagina radice dell'app implementata come Razor pagina:
    * Quando viene inizialmente richiesta una pagina dell'app, il rendering della pagina viene eseguito e restituito nella risposta.
    * Nella pagina host viene specificata la posizione in cui `App` viene eseguito il rendering del componente radice ( `App.razor` ).
  * `Counter` Component ( `Counter.razor` ): implementa la pagina del contatore.
  * `Error` Component ( `Error.razor` ): viene eseguito il rendering quando si verifica un'eccezione non gestita nell'app.
  * `FetchData` Component ( `FetchData.razor` ): implementa la pagina Recupera dati.
  * `Index` Component ( `Index.razor` ): implementa la Home page.

> [!NOTE]
> I file JavaScript (JS) aggiunti al `Pages/_Host.cshtml` file devono essere visualizzati prima del `</body>` tag di chiusura. L'ordine con cui il codice JS personalizzato viene caricato dai file JS è importante in alcuni scenari. Ad esempio, assicurarsi che i file JS con metodi di interoperabilità siano inclusi prima dei Blazor file di framework JS.

* `Properties/launchSettings.json`: Include la [configurazione dell'ambiente di sviluppo](xref:fundamentals/environments#development-and-launchsettingsjson).

::: moniker range=">= aspnetcore-5.0"

* `Shared` cartella: contiene i componenti e i fogli di stile condivisi seguenti:
  * `MainLayout` Component ( `MainLayout.razor` ): [componente del layout](xref:blazor/layouts)dell'app.
  * `MainLayout.razor.css`: Foglio di stile per il layout principale dell'app.
  * `NavMenu` Component ( `NavMenu.razor` ): implementa la navigazione nell'intestazione laterale. Include il [ `NavLink` componente](xref:blazor/fundamentals/routing#navlink-and-navmenu-components) ( <xref:Microsoft.AspNetCore.Components.Routing.NavLink> ) che esegue il rendering dei collegamenti di navigazione ad altri Razor componenti. Il <xref:Microsoft.AspNetCore.Components.Routing.NavLink> componente indica automaticamente uno stato selezionato quando viene caricato il componente, che consente all'utente di comprendere il componente attualmente visualizzato.
  * `NavMenu.razor.css`: Foglio di stile per il menu di navigazione dell'app.
  * `SurveyPrompt` Component ( `SurveyPrompt.razor` ): Blazor componente Survey.

::: moniker-end

::: moniker range="< aspnetcore-5.0"

* `Shared` cartella: contiene i componenti condivisi seguenti:
  * `MainLayout` Component ( `MainLayout.razor` ): [componente del layout](xref:blazor/layouts)dell'app.
  * `NavMenu` Component ( `NavMenu.razor` ): implementa la navigazione nell'intestazione laterale. Include il [ `NavLink` componente](xref:blazor/fundamentals/routing#navlink-and-navmenu-components) ( <xref:Microsoft.AspNetCore.Components.Routing.NavLink> ) che esegue il rendering dei collegamenti di navigazione ad altri Razor componenti. Il <xref:Microsoft.AspNetCore.Components.Routing.NavLink> componente indica automaticamente uno stato selezionato quando viene caricato il componente, che consente all'utente di comprendere il componente attualmente visualizzato.
  * `SurveyPrompt` Component ( `SurveyPrompt.razor` ): Blazor componente Survey.
  
::: moniker-end

* `wwwroot`: La cartella [radice Web](xref:fundamentals/index#web-root) per l'app contenente le risorse statiche pubbliche dell'app.

* `_Imports.razor`: Include Razor direttive comuni da includere nei componenti dell'app ( `.razor` ), ad esempio le [`@using`](xref:mvc/views/razor#using) direttive per gli spazi dei nomi.

* `App.razor`: Componente radice dell'app che imposta il routing sul lato client usando il <xref:Microsoft.AspNetCore.Components.Routing.Router> componente. Il <xref:Microsoft.AspNetCore.Components.Routing.Router> componente intercetta la navigazione del browser ed esegue il rendering della pagina che corrisponde all'indirizzo richiesto.

* `appsettings.json` e file di impostazioni dell'app ambientale: fornire [le impostazioni di configurazione](xref:blazor/fundamentals/configuration) per l'app.

* `Program.cs`: Il punto di ingresso dell'app che configura l' [host](xref:fundamentals/host/generic-host)ASP.NET Core.

* `Startup.cs`: Contiene la logica di avvio dell'app. La `Startup` classe definisce due metodi:

  * `ConfigureServices`: Configura i servizi [di inserimento delle dipendenze](xref:fundamentals/dependency-injection) dell'app. I servizi vengono aggiunti chiamando <xref:Microsoft.Extensions.DependencyInjection.ComponentServiceCollectionExtensions.AddServerSideBlazor%2A> e l'oggetto `WeatherForecastService` viene aggiunto al contenitore del servizio per l'utilizzo da parte del componente di esempio `FetchData` .
  * `Configure`: Configura la pipeline di gestione delle richieste dell'app:
    * <xref:Microsoft.AspNetCore.Builder.ComponentEndpointRouteBuilderExtensions.MapBlazorHub%2A> viene chiamato per configurare un endpoint per la connessione in tempo reale con il browser. La connessione viene creata con [SignalR](xref:signalr/introduction) , che è un Framework per l'aggiunta di funzionalità Web in tempo reale alle app.
    * [`MapFallbackToPage("/_Host")`](xref:Microsoft.AspNetCore.Builder.RazorPagesEndpointRouteBuilderExtensions.MapFallbackToPage*) viene chiamato per impostare la pagina radice dell'app ( `Pages/_Host.cshtml` ) e abilitare la navigazione.
