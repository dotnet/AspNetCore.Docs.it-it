---
title: Pre-rendering e integrazione dei Razor componenti ASP.NET Core
author: guardrex
description: Informazioni sugli Razor scenari di integrazione dei componenti per Blazor le app, incluso il prerendering dei Razor componenti nel server.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 10/29/2020
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
uid: blazor/components/prerendering-and-integration
zone_pivot_groups: blazor-hosting-models
ms.openlocfilehash: a0c5cc0bdc78f2ea70b8c128616ad09328ccf87d
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102587385"
---
# <a name="prerender-and-integrate-aspnet-core-razor-components"></a>Pre-rendering e integrazione dei Razor componenti ASP.NET Core

::: zone pivot="webassembly"

::: moniker range=">= aspnetcore-5.0"

Razor i componenti possono essere integrati in Razor pagine e app MVC in una Blazor WebAssembly soluzione ospitata. Quando viene eseguito il rendering della pagina o della vista, è possibile eseguire il rendering dei componenti contemporaneamente.

## <a name="configuration"></a>Configurazione

Per configurare il prerendering per un' Blazor WebAssembly app:

1. Ospitare l' Blazor WebAssembly app in un'app ASP.NET Core. Blazor WebAssemblyÈ possibile aggiungere un'app autonoma a una soluzione ASP.NET Core oppure è possibile usare un'app ospitata Blazor WebAssembly creata dal [ Blazor WebAssembly modello di progetto](xref:blazor/project-structure).

1. Rimuovere il file statico predefinito `wwwroot/index.html` dal Blazor WebAssembly progetto client.

1. Eliminare la riga seguente in `Program.Main` nel progetto client:

   ```csharp
   builder.RootComponents.Add<App>("#app");
   ```

1. Aggiungere un `Pages/_Host.cshtml` file al progetto server. È possibile ottenere un `_Host.cshtml` file da un'app creata dal Blazor Server modello con il `dotnet new blazorserver -o BlazorServer` comando in una shell dei comandi. Dopo aver posizionato il `Pages/_Host.cshtml` file nell'app server della soluzione ospitata Blazor WebAssembly , apportare le modifiche seguenti al file:

   * Impostare lo spazio dei nomi sulla cartella dell'app Server `Pages` (ad esempio, `@namespace BlazorHosted.Server.Pages` ).
   * Impostare una [`@using`](xref:mvc/views/razor#using) direttiva per il progetto client (ad esempio, `@using BlazorHosted.Client` ).
   * Aggiornare i collegamenti del foglio di stile in modo che puntino al foglio di stile dell'app webassembly. Nell'esempio seguente lo spazio dei nomi dell'app client è `BlazorHosted.Client` :

     ```cshtml
     <link href="css/app.css" rel="stylesheet" />
     <link href="BlazorHosted.Client.styles.css" rel="stylesheet" />
     ```

   * Aggiornare l'oggetto `render-mode` dell' [Helper Tag componente](xref:mvc/views/tag-helpers/builtin-th/component-tag-helper) per eseguire il prerendering del `App` componente radice:

     ```cshtml
     <component type="typeof(App)" render-mode="WebAssemblyPrerendered" />
     ```

   * Aggiornare l' Blazor origine dello script per usare lo script sul lato client Blazor WebAssembly :

     ```cshtml
     <script src="_framework/blazor.webassembly.js"></script>
     ```

1. In `Startup.Configure` ( `Startup.cs` ) del progetto server:

   * Chiamare il `UseDeveloperExceptionPage` Generatore di app nell'ambiente di sviluppo.
   * Chiamare `UseBlazorFrameworkFiles` il generatore di app.
   * Modificare il fallback dal `index.html` file ( `endpoints.MapFallbackToFile("index.html");` ) nella `_Host.cshtml` pagina: `endpoints.MapFallbackToPage("/_Host");` .

   ```csharp
   public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
   {
       if (env.IsDevelopment())
       {
           app.UseDeveloperExceptionPage();
           app.UseWebAssemblyDebugging();
       }
       else
       {
           app.UseExceptionHandler("/Error");
           app.UseHsts();
       }

       app.UseHttpsRedirection();
       app.UseBlazorFrameworkFiles();
       app.UseStaticFiles();

       app.UseRouting();

       app.UseEndpoints(endpoints =>
       {
           endpoints.MapRazorPages();
           endpoints.MapControllers();
           endpoints.MapFallbackToPage("/_Host");
       });
   }
   ```

## <a name="render-components-in-a-page-or-view-with-the-component-tag-helper"></a>Eseguire il rendering dei componenti in una pagina o in una vista con l'helper Tag Component

L'helper Tag Component supporta due modalità di rendering per il rendering di un componente da un' Blazor WebAssembly app in una pagina o in una visualizzazione:

* `WebAssembly`: Esegue il rendering di un marcatore per un' Blazor WebAssembly app da usare per includere un componente interattivo quando viene caricato nel browser. Il componente non è preeseguito. Questa opzione rende più semplice il rendering di Blazor WebAssembly componenti diversi in pagine diverse.
* `WebAssemblyPrerendered`: Esegue il rendering del componente in HTML statico e include un marcatore per un'app da usare in un Blazor WebAssembly secondo momento per rendere il componente interattivo quando viene caricato nel browser.

Nell'esempio di Razor pagine seguente `Counter` viene eseguito il rendering del componente in una pagina. Per rendere interattivo il componente, lo Blazor WebAssembly script viene incluso nella [sezione di rendering](xref:mvc/views/layout#sections)della pagina. Per evitare di usare lo spazio dei nomi completo per il `Counter` componente con l'helper Tag Component ( `{APP ASSEMBLY}.Pages.Counter` ), aggiungere una [`@using`](xref:mvc/views/razor#using) direttiva per lo `Pages` spazio dei nomi dell'app client. Nell'esempio seguente lo spazio dei nomi dell'app client è `BlazorHosted.Client` :

```cshtml
...
@using BlazorHosted.Client.Pages

<h1>@ViewData["Title"]</h1>

<component type="typeof(Counter)" render-mode="WebAssemblyPrerendered" />

@section Scripts {
    <script src="_framework/blazor.webassembly.js"></script>
}
```

<xref:Microsoft.AspNetCore.Mvc.Rendering.RenderMode> Configura se il componente:

* Viene preeseguito nella pagina.
* Viene sottoposto a rendering come HTML statico nella pagina o se include le informazioni necessarie per il bootstrap di un' Blazor app dall'agente utente.

Per ulteriori informazioni sull'helper Tag Component, incluso il passaggio di parametri e <xref:Microsoft.AspNetCore.Mvc.Rendering.RenderMode> configurazione, vedere <xref:mvc/views/tag-helpers/builtin-th/component-tag-helper> .

Nell'esempio precedente è necessario che il layout dell'app Server ( `_Layout.cshtml` ) includa una [sezione di rendering](xref:mvc/views/layout#sections) ( <xref:Microsoft.AspNetCore.Mvc.Razor.RazorPage.RenderSection%2A> ) per lo script all'interno del tag di chiusura `</body>` :

```cshtml
    ...

    @RenderSection("Scripts", required: false)
</body>
```

Il `_Layout.cshtml` file si trova nella `Pages/Shared` cartella in un' Razor app di pagine o `Views/Shared` in una cartella in un'app MVC.

Se l'app deve anche applicare uno stile ai componenti con gli stili nell' Blazor WebAssembly app, includere gli stili dell'app nel `_Layout.cshtml` file. Nell'esempio seguente lo spazio dei nomi dell'app client è `BlazorHosted.Client` :

```cshtml
<head>
    ...

    <link href="css/app.css" rel="stylesheet" />
    <link href="BlazorHosted.Client.styles.css" rel="stylesheet" />
</head>
```

## <a name="render-components-in-a-page-or-view-with-a-css-selector"></a>Eseguire il rendering dei componenti in una pagina o in una vista con un selettore CSS

Aggiungere componenti radice al progetto *client* in `Program.Main` ( `Program.cs` ). Nell'esempio seguente il `Counter` componente viene dichiarato come componente radice con un selettore CSS che seleziona l'elemento con l'oggetto `id` che corrisponde a `my-counter` . Nell'esempio seguente lo spazio dei nomi dell'app client è `BlazorHosted.Client` :

```csharp
using BlazorHosted.Client.Pages;

...

builder.RootComponents.Add<Counter>("#my-counter");
```

Nell'esempio di Razor pagine seguente `Counter` viene eseguito il rendering del componente in una pagina. Per rendere interattivo il componente, lo Blazor WebAssembly script viene incluso nella [sezione di rendering](xref:mvc/views/layout#sections)della pagina:

```cshtml
...

<h1>@ViewData["Title"]</h1>

<div id="my-counter">Loading...</div>

@section Scripts {
    <script src="_framework/blazor.webassembly.js"></script>
}
```

Nell'esempio precedente è necessario che il layout dell'app Server ( `_Layout.cshtml` ) includa una [sezione di rendering](xref:mvc/views/layout#sections) ( <xref:Microsoft.AspNetCore.Mvc.Razor.RazorPage.RenderSection%2A> ) per lo script all'interno del tag di chiusura `</body>` :

```cshtml
    ...

    @RenderSection("Scripts", required: false)
</body>
```

Il `_Layout.cshtml` file si trova nella `Pages/Shared` cartella in un' Razor app di pagine o `Views/Shared` in una cartella in un'app MVC.

Se l'app deve anche applicare uno stile ai componenti con gli stili nell' Blazor WebAssembly app, includere gli stili dell'app nel `_Layout.cshtml` file. Nell'esempio seguente lo spazio dei nomi dell'app client è `BlazorHosted.Client` :

```cshtml
<head>
    ...

    <link href="css/app.css" rel="stylesheet" />
    <link href="BlazorHosted.Client.styles.css" rel="stylesheet" />
</head>
```

## <a name="additional-resources"></a>Risorse aggiuntive

* [Supporto del prerendering con autenticazione](xref:blazor/security/webassembly/additional-scenarios#support-prerendering-with-authentication)

::: moniker-end

::: moniker range="< aspnetcore-5.0"

L'integrazione di Razor componenti in Razor pagine e app MVC in una Blazor WebAssembly soluzione ospitata è supportata in ASP.NET Core in .NET 5 o versioni successive. Selezionare una versione di .NET 5 o successiva di questo articolo.

::: moniker-end

::: zone-end

::: zone pivot="server"

Razor i componenti possono essere integrati in Razor pagine e app MVC in un' Blazor Server app. Quando viene eseguito il rendering della pagina o della vista, è possibile eseguire il rendering dei componenti contemporaneamente.

Dopo aver [configurato l'app](#configuration), usare le istruzioni riportate nelle sezioni seguenti a seconda dei requisiti dell'app:

* Componenti instradabili: per i componenti che sono direttamente instradabili dalle richieste degli utenti. Seguire queste linee guida quando i visitatori devono essere in grado di effettuare una richiesta HTTP nel browser per un componente con una [`@page`](xref:mvc/views/razor#page) direttiva.
  * [Usare componenti instradabili in un' Razor app pagine](#use-routable-components-in-a-razor-pages-app)
  * [Usare componenti instradabili in un'app MVC](#use-routable-components-in-an-mvc-app)
* [Eseguire il rendering dei componenti da una pagina o vista](#render-components-from-a-page-or-view): per i componenti che non sono direttamente instradabili dalle richieste degli utenti. Seguire queste linee guida quando l'app incorpora i componenti in pagine e visualizzazioni esistenti con l' [Helper tag dei componenti](xref:mvc/views/tag-helpers/builtin-th/component-tag-helper).

## <a name="configuration"></a>Configurazione

Un' Razor app MVC o pagine esistente può integrare i Razor componenti in pagine e visualizzazioni:

1. Nel file di layout dell'app ( `_Layout.cshtml` ):

   * Aggiungere il `<base>` tag seguente all' `<head>` elemento:

     ```html
     <base href="~/" />
     ```

     Il `href` valore (il *percorso di base dell'app*) nell'esempio precedente presuppone che l'app si trovi nel percorso URL radice ( `/` ). Se l'app è un'applicazione secondaria, seguire le istruzioni nella sezione *percorso di base* dell'applicazione dell' <xref:blazor/host-and-deploy/index#app-base-path> articolo.

     Il `_Layout.cshtml` file si trova nella `Pages/Shared` cartella in un' Razor app di pagine o `Views/Shared` in una cartella in un'app MVC.

   * Aggiungere un `<script>` tag per lo `blazor.server.js` script immediatamente prima della `Scripts` sezione Render:

     ```html
         ...
         <script src="_framework/blazor.server.js"></script>

         @await RenderSectionAsync("Scripts", required: false)
     </body>
     ```

     Il Framework aggiunge lo `blazor.server.js` script all'app. Non è necessario aggiungere manualmente lo script all'app.

1. Aggiungere un `_Imports.razor` file alla cartella radice del progetto con il contenuto seguente (modificare l'ultimo spazio dei nomi, `MyAppNamespace` , nello spazio dei nomi dell'app):

   ```razor
   @using System.Net.Http
   @using Microsoft.AspNetCore.Authorization
   @using Microsoft.AspNetCore.Components.Authorization
   @using Microsoft.AspNetCore.Components.Forms
   @using Microsoft.AspNetCore.Components.Routing
   @using Microsoft.AspNetCore.Components.Web
   @using Microsoft.JSInterop
   @using MyAppNamespace
   ```

1. In `Startup.ConfigureServices` registrare il Blazor Server servizio:

   ```csharp
   services.AddServerSideBlazor();
   ```

1. In `Startup.Configure` aggiungere l' Blazor endpoint dell'hub a `app.UseEndpoints` :

   ```csharp
   endpoints.MapBlazorHub();
   ```

1. Integrare i componenti in qualsiasi pagina o visualizzazione. Per ulteriori informazioni, vedere la sezione [rendering dei componenti da una pagina o vista](#render-components-from-a-page-or-view) .

## <a name="use-routable-components-in-a-razor-pages-app"></a>Usare componenti instradabili in un' Razor app pagine

*Questa sezione riguarda l'aggiunta di componenti che sono direttamente instradabili dalle richieste degli utenti.*

Per supportare componenti instradabili Razor nelle Razor app di pagine:

1. Seguire le istruzioni riportate nella sezione di [configurazione](#configuration) .

1. Aggiungere un `App.razor` file alla radice del progetto con il contenuto seguente:

   ```razor
   @using Microsoft.AspNetCore.Components.Routing

   <Router AppAssembly="@typeof(Program).Assembly">
       <Found Context="routeData">
           <RouteView RouteData="routeData" />
       </Found>
       <NotFound>
           <h1>Page not found</h1>
           <p>Sorry, but there's nothing here!</p>
       </NotFound>
   </Router>
   ```

   [!INCLUDE[](~/blazor/includes/prefer-exact-matches.md)]

1. Aggiungere un `_Host.cshtml` file alla `Pages` cartella con il contenuto seguente:

   ```cshtml
   @page "/blazor"
   @{
       Layout = "_Layout";
   }

   <app>
       <component type="typeof(App)" render-mode="ServerPrerendered" />
   </app>
   ```

   I componenti utilizzano il `_Layout.cshtml` file condiviso per il layout.

   <xref:Microsoft.AspNetCore.Mvc.Rendering.RenderMode> Configura se il `App` componente:

   * Viene preeseguito nella pagina.
   * Viene sottoposto a rendering come HTML statico nella pagina o se include le informazioni necessarie per il bootstrap di un' Blazor app dall'agente utente.

   Per ulteriori informazioni sull'helper Tag Component, incluso il passaggio di parametri e <xref:Microsoft.AspNetCore.Mvc.Rendering.RenderMode> configurazione, vedere <xref:mvc/views/tag-helpers/builtin-th/component-tag-helper> .

1. Aggiungere una route con priorità bassa per la `_Host.cshtml` pagina alla configurazione dell'endpoint in `Startup.Configure` :

   ```csharp
   app.UseEndpoints(endpoints =>
   {
       ...

       endpoints.MapFallbackToPage("/_Host");
   });
   ```

1. Aggiungere componenti instradabili all'app. Ad esempio:

   ```razor
   @page "/counter"

   <h1>Counter</h1>

   ...
   ```

Per ulteriori informazioni sugli spazi dei nomi, vedere la sezione relativa agli [spazi dei nomi dei componenti](#component-namespaces) .

## <a name="use-routable-components-in-an-mvc-app"></a>Usare componenti instradabili in un'app MVC

*Questa sezione riguarda l'aggiunta di componenti che sono direttamente instradabili dalle richieste degli utenti.*

Per supportare componenti instradabili Razor nelle app MVC:

1. Seguire le istruzioni riportate nella sezione di [configurazione](#configuration) .

1. Aggiungere un `App.razor` file alla radice del progetto con il contenuto seguente:

   ```razor
   @using Microsoft.AspNetCore.Components.Routing

   <Router AppAssembly="@typeof(Program).Assembly">
       <Found Context="routeData">
           <RouteView RouteData="routeData" />
       </Found>
       <NotFound>
           <h1>Page not found</h1>
           <p>Sorry, but there's nothing here!</p>
       </NotFound>
   </Router>
   ```

   [!INCLUDE[](~/blazor/includes/prefer-exact-matches.md)]

1. Aggiungere un `_Host.cshtml` file alla `Views/Home` cartella con il contenuto seguente:

   ```cshtml
   @{
       Layout = "_Layout";
   }

   <app>
       <component type="typeof(App)" render-mode="ServerPrerendered" />
   </app>
   ```

   I componenti utilizzano il `_Layout.cshtml` file condiviso per il layout.

   <xref:Microsoft.AspNetCore.Mvc.Rendering.RenderMode> Configura se il `App` componente:

   * Viene preeseguito nella pagina.
   * Viene sottoposto a rendering come HTML statico nella pagina o se include le informazioni necessarie per il bootstrap di un' Blazor app dall'agente utente.

   Per ulteriori informazioni sull'helper Tag Component, incluso il passaggio di parametri e <xref:Microsoft.AspNetCore.Mvc.Rendering.RenderMode> configurazione, vedere <xref:mvc/views/tag-helpers/builtin-th/component-tag-helper> .

1. Aggiungere un'azione al controller Home:

   ```csharp
   public IActionResult Blazor()
   {
      return View("_Host");
   }
   ```

1. Aggiungere una route con priorità bassa per l'azione del controller che restituisce la `_Host.cshtml` visualizzazione alla configurazione dell'endpoint in `Startup.Configure` :

   ```csharp
   app.UseEndpoints(endpoints =>
   {
       ...

       endpoints.MapFallbackToController("Blazor", "Home");
   });
   ```

1. Creare una `Pages` cartella e aggiungere componenti instradabili all'app. Ad esempio:

   ```razor
   @page "/counter"

   <h1>Counter</h1>

   ...
   ```

Per ulteriori informazioni sugli spazi dei nomi, vedere la sezione relativa agli [spazi dei nomi dei componenti](#component-namespaces) .

## <a name="render-components-from-a-page-or-view"></a>Eseguire il rendering dei componenti da una pagina o da una vista

*Questa sezione riguarda l'aggiunta di componenti a pagine o viste, in cui i componenti non sono direttamente instradabili dalle richieste degli utenti.*

Per eseguire il rendering di un componente da una pagina o da una vista, usare l' [Helper Tag Component](xref:mvc/views/tag-helpers/builtin-th/component-tag-helper).

### <a name="render-stateful-interactive-components"></a>Eseguire il rendering di componenti interattivi con stato

I componenti interattivi con stato possono essere aggiunti a una Razor pagina o a una vista.

Quando viene eseguito il rendering della pagina o della visualizzazione:

* Il componente viene preeseguito con la pagina o la visualizzazione.
* Lo stato iniziale del componente utilizzato per il prerendering viene perso.
* Quando viene stabilita la connessione, viene creato il nuovo stato del componente SignalR .

Nella pagina seguente viene Razor eseguito il rendering di un `Counter` componente:

```cshtml
<h1>My Razor Page</h1>

<component type="typeof(Counter)" render-mode="ServerPrerendered" 
    param-InitialValue="InitialValue" />

@functions {
    [BindProperty(SupportsGet=true)]
    public int InitialValue { get; set; }
}
```

Per altre informazioni, vedere <xref:mvc/views/tag-helpers/builtin-th/component-tag-helper>.

### <a name="render-noninteractive-components"></a>Eseguire il rendering di componenti non interattivi

Nella pagina seguente Razor il componente viene sottoposto a `Counter` rendering statico con un valore iniziale specificato utilizzando un form. Poiché il componente viene sottoposto a rendering statico, il componente non è interattivo:

```cshtml
<h1>My Razor Page</h1>

<form>
    <input type="number" asp-for="InitialValue" />
    <button type="submit">Set initial value</button>
</form>

<component type="typeof(Counter)" render-mode="Static" 
    param-InitialValue="InitialValue" />

@functions {
    [BindProperty(SupportsGet=true)]
    public int InitialValue { get; set; }
}
```

Per altre informazioni, vedere <xref:mvc/views/tag-helpers/builtin-th/component-tag-helper>.

## <a name="component-namespaces"></a>Spazi dei nomi dei componenti

Quando si usa una cartella personalizzata per conservare i componenti dell'app, aggiungere lo spazio dei nomi che rappresenta la cartella alla pagina/visualizzazione o al `_ViewImports.cshtml` file. Nell'esempio seguente:

* Passare `MyAppNamespace` allo spazio dei nomi dell'app.
* Se non viene usata una cartella denominata `Components` per conservare i componenti, passare `Components` alla cartella in cui si trovano i componenti.

```cshtml
@using MyAppNamespace.Components
```

Il `_ViewImports.cshtml` file si trova nella `Pages` cartella dell' Razor app pagine o nella `Views` cartella di un'app MVC.

Per altre informazioni, vedere <xref:blazor/components/index#namespaces>.

::: zone-end
