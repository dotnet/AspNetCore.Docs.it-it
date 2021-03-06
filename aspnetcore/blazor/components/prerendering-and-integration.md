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
ms.openlocfilehash: a86b50abff9c5ec52aab2bdb7eb6d563a5197d1a
ms.sourcegitcommit: 1436bd4d70937d6ec3140da56d96caab33c4320b
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/06/2021
ms.locfileid: "102395175"
---
# <a name="prerender-and-integrate-aspnet-core-razor-components"></a><span data-ttu-id="f2248-103">Pre-rendering e integrazione dei Razor componenti ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="f2248-103">Prerender and integrate ASP.NET Core Razor components</span></span>

::: zone pivot="webassembly"

::: moniker range=">= aspnetcore-5.0"

<span data-ttu-id="f2248-104">Razor i componenti possono essere integrati in Razor pagine e app MVC in una Blazor WebAssembly soluzione ospitata.</span><span class="sxs-lookup"><span data-stu-id="f2248-104">Razor components can be integrated into Razor Pages and MVC apps in a hosted Blazor WebAssembly solution.</span></span> <span data-ttu-id="f2248-105">Quando viene eseguito il rendering della pagina o della vista, è possibile eseguire il rendering dei componenti contemporaneamente.</span><span class="sxs-lookup"><span data-stu-id="f2248-105">When the page or view is rendered, components can be prerendered at the same time.</span></span>

## <a name="configuration"></a><span data-ttu-id="f2248-106">Configurazione</span><span class="sxs-lookup"><span data-stu-id="f2248-106">Configuration</span></span>

<span data-ttu-id="f2248-107">Per configurare il prerendering per un' Blazor WebAssembly app:</span><span class="sxs-lookup"><span data-stu-id="f2248-107">To set up prerendering for a Blazor WebAssembly app:</span></span>

1. <span data-ttu-id="f2248-108">Ospitare l' Blazor WebAssembly app in un'app ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="f2248-108">Host the Blazor WebAssembly app in an ASP.NET Core app.</span></span> <span data-ttu-id="f2248-109">Blazor WebAssemblyÈ possibile aggiungere un'app autonoma a una soluzione ASP.NET Core oppure è possibile usare un'app ospitata Blazor WebAssembly creata dal [ Blazor WebAssembly modello di progetto](xref:blazor/project-structure).</span><span class="sxs-lookup"><span data-stu-id="f2248-109">A standalone Blazor WebAssembly app can be added to an ASP.NET Core solution, or you can use a hosted Blazor WebAssembly app created from the [Blazor WebAssembly project template](xref:blazor/project-structure).</span></span>

1. <span data-ttu-id="f2248-110">Rimuovere il file statico predefinito `wwwroot/index.html` dal Blazor WebAssembly progetto client.</span><span class="sxs-lookup"><span data-stu-id="f2248-110">Remove the default static `wwwroot/index.html` file from the Blazor WebAssembly client project.</span></span>

1. <span data-ttu-id="f2248-111">Eliminare la riga seguente in `Program.Main` nel progetto client:</span><span class="sxs-lookup"><span data-stu-id="f2248-111">Delete the following line in `Program.Main` in the client project:</span></span>

   ```csharp
   builder.RootComponents.Add<App>("#app");
   ```

1. <span data-ttu-id="f2248-112">Aggiungere un `Pages/_Host.cshtml` file al progetto server.</span><span class="sxs-lookup"><span data-stu-id="f2248-112">Add a `Pages/_Host.cshtml` file to the server project.</span></span> <span data-ttu-id="f2248-113">È possibile ottenere un `_Host.cshtml` file da un'app creata dal Blazor Server modello con il `dotnet new blazorserver -o BlazorServer` comando in una shell dei comandi.</span><span class="sxs-lookup"><span data-stu-id="f2248-113">You can obtain a `_Host.cshtml` file from an app created from the Blazor Server template with the `dotnet new blazorserver -o BlazorServer` command in a command shell.</span></span> <span data-ttu-id="f2248-114">Dopo aver posizionato il `Pages/_Host.cshtml` file nell'app server della soluzione ospitata Blazor WebAssembly , apportare le modifiche seguenti al file:</span><span class="sxs-lookup"><span data-stu-id="f2248-114">After placing the `Pages/_Host.cshtml` file into the server app of the hosted Blazor WebAssembly solution, make the following changes to the file:</span></span>

   * <span data-ttu-id="f2248-115">Impostare lo spazio dei nomi sulla cartella dell'app Server `Pages` (ad esempio, `@namespace BlazorHosted.Server.Pages` ).</span><span class="sxs-lookup"><span data-stu-id="f2248-115">Set the namespace to the server app's `Pages` folder (for example, `@namespace BlazorHosted.Server.Pages`).</span></span>
   * <span data-ttu-id="f2248-116">Impostare una [`@using`](xref:mvc/views/razor#using) direttiva per il progetto client (ad esempio, `@using BlazorHosted.Client` ).</span><span class="sxs-lookup"><span data-stu-id="f2248-116">Set an [`@using`](xref:mvc/views/razor#using) directive for the client project (for example, `@using BlazorHosted.Client`).</span></span>
   * <span data-ttu-id="f2248-117">Aggiornare i collegamenti del foglio di stile in modo che puntino al foglio di stile dell'app webassembly.</span><span class="sxs-lookup"><span data-stu-id="f2248-117">Update the stylesheet links to point to the WebAssembly app's stylesheet.</span></span> <span data-ttu-id="f2248-118">Nell'esempio seguente lo spazio dei nomi dell'app client è `BlazorHosted.Client` :</span><span class="sxs-lookup"><span data-stu-id="f2248-118">In the following example, the client app's namespace is `BlazorHosted.Client`:</span></span>

     ```cshtml
     <link href="css/app.css" rel="stylesheet" />
     <link href="BlazorHosted.Client.styles.css" rel="stylesheet" />
     ```

   * <span data-ttu-id="f2248-119">Aggiornare l'oggetto `render-mode` dell' [Helper Tag componente](xref:mvc/views/tag-helpers/builtin-th/component-tag-helper) per eseguire il prerendering del `App` componente radice:</span><span class="sxs-lookup"><span data-stu-id="f2248-119">Update the `render-mode` of the [Component Tag Helper](xref:mvc/views/tag-helpers/builtin-th/component-tag-helper) to prerender the root `App` component:</span></span>

     ```cshtml
     <component type="typeof(App)" render-mode="WebAssemblyPrerendered" />
     ```

   * <span data-ttu-id="f2248-120">Aggiornare l' Blazor origine dello script per usare lo script sul lato client Blazor WebAssembly :</span><span class="sxs-lookup"><span data-stu-id="f2248-120">Update the Blazor script source to use the client-side Blazor WebAssembly script:</span></span>

     ```cshtml
     <script src="_framework/blazor.webassembly.js"></script>
     ```

1. <span data-ttu-id="f2248-121">In `Startup.Configure` ( `Startup.cs` ) del progetto server:</span><span class="sxs-lookup"><span data-stu-id="f2248-121">In `Startup.Configure` (`Startup.cs`) of the server project:</span></span>

   * <span data-ttu-id="f2248-122">Chiamare il `UseDeveloperExceptionPage` Generatore di app nell'ambiente di sviluppo.</span><span class="sxs-lookup"><span data-stu-id="f2248-122">Call `UseDeveloperExceptionPage` on the app builder in the Development environment.</span></span>
   * <span data-ttu-id="f2248-123">Chiamare `UseBlazorFrameworkFiles` il generatore di app.</span><span class="sxs-lookup"><span data-stu-id="f2248-123">Call `UseBlazorFrameworkFiles` on the app builder.</span></span>
   * <span data-ttu-id="f2248-124">Modificare il fallback dal `index.html` file ( `endpoints.MapFallbackToFile("index.html");` ) nella `_Host.cshtml` pagina: `endpoints.MapFallbackToPage("/_Host");` .</span><span class="sxs-lookup"><span data-stu-id="f2248-124">Change the fallback from the `index.html` file (`endpoints.MapFallbackToFile("index.html");`) to the `_Host.cshtml` page: `endpoints.MapFallbackToPage("/_Host");`.</span></span>

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

## <a name="render-components-in-a-page-or-view-with-the-component-tag-helper"></a><span data-ttu-id="f2248-125">Eseguire il rendering dei componenti in una pagina o in una vista con l'helper Tag Component</span><span class="sxs-lookup"><span data-stu-id="f2248-125">Render components in a page or view with the Component Tag Helper</span></span>

<span data-ttu-id="f2248-126">L'helper Tag Component supporta due modalità di rendering per il rendering di un componente da un' Blazor WebAssembly app in una pagina o in una visualizzazione:</span><span class="sxs-lookup"><span data-stu-id="f2248-126">The Component Tag Helper supports two render modes for rendering a component from a Blazor WebAssembly app in a page or view:</span></span>

* <span data-ttu-id="f2248-127">`WebAssembly`: Esegue il rendering di un marcatore per un' Blazor WebAssembly app da usare per includere un componente interattivo quando viene caricato nel browser.</span><span class="sxs-lookup"><span data-stu-id="f2248-127">`WebAssembly`: Renders a marker for a Blazor WebAssembly app for use to include an interactive component when loaded in the browser.</span></span> <span data-ttu-id="f2248-128">Il componente non è preeseguito.</span><span class="sxs-lookup"><span data-stu-id="f2248-128">The component isn't prerendered.</span></span> <span data-ttu-id="f2248-129">Questa opzione rende più semplice il rendering di Blazor WebAssembly componenti diversi in pagine diverse.</span><span class="sxs-lookup"><span data-stu-id="f2248-129">This option makes it easier to render different Blazor WebAssembly components on different pages.</span></span>
* <span data-ttu-id="f2248-130">`WebAssemblyPrerendered`: Esegue il rendering del componente in HTML statico e include un marcatore per un'app da usare in un Blazor WebAssembly secondo momento per rendere il componente interattivo quando viene caricato nel browser.</span><span class="sxs-lookup"><span data-stu-id="f2248-130">`WebAssemblyPrerendered`: Prerenders the component into static HTML and includes a marker for a Blazor WebAssembly app for later use to make the component interactive when loaded in the browser.</span></span>

<span data-ttu-id="f2248-131">Nell'esempio di Razor pagine seguente `Counter` viene eseguito il rendering del componente in una pagina.</span><span class="sxs-lookup"><span data-stu-id="f2248-131">In the following Razor Pages example, the `Counter` component is rendered in a page.</span></span> <span data-ttu-id="f2248-132">Per rendere interattivo il componente, lo Blazor WebAssembly script viene incluso nella [sezione di rendering](xref:mvc/views/layout#sections)della pagina.</span><span class="sxs-lookup"><span data-stu-id="f2248-132">To make the component interactive, the Blazor WebAssembly script is included in the page's [render section](xref:mvc/views/layout#sections).</span></span> <span data-ttu-id="f2248-133">Per evitare di usare lo spazio dei nomi completo per il `Counter` componente con l'helper Tag Component ( `{APP ASSEMBLY}.Pages.Counter` ), aggiungere una [`@using`](xref:mvc/views/razor#using) direttiva per lo `Pages` spazio dei nomi dell'app client.</span><span class="sxs-lookup"><span data-stu-id="f2248-133">To avoid using the full namespace for the `Counter` component with the Component Tag Helper (`{APP ASSEMBLY}.Pages.Counter`), add an [`@using`](xref:mvc/views/razor#using) directive for the client app's `Pages` namespace.</span></span> <span data-ttu-id="f2248-134">Nell'esempio seguente lo spazio dei nomi dell'app client è `BlazorHosted.Client` :</span><span class="sxs-lookup"><span data-stu-id="f2248-134">In the following example, the client app's namespace is `BlazorHosted.Client`:</span></span>

```cshtml
...
@using BlazorHosted.Client.Pages

<h1>@ViewData["Title"]</h1>

<component type="typeof(Counter)" render-mode="WebAssemblyPrerendered" />

@section Scripts {
    <script src="_framework/blazor.webassembly.js"></script>
}
```

<span data-ttu-id="f2248-135"><xref:Microsoft.AspNetCore.Mvc.Rendering.RenderMode> Configura se il componente:</span><span class="sxs-lookup"><span data-stu-id="f2248-135"><xref:Microsoft.AspNetCore.Mvc.Rendering.RenderMode> configures whether the component:</span></span>

* <span data-ttu-id="f2248-136">Viene preeseguito nella pagina.</span><span class="sxs-lookup"><span data-stu-id="f2248-136">Is prerendered into the page.</span></span>
* <span data-ttu-id="f2248-137">Viene sottoposto a rendering come HTML statico nella pagina o se include le informazioni necessarie per il bootstrap di un' Blazor app dall'agente utente.</span><span class="sxs-lookup"><span data-stu-id="f2248-137">Is rendered as static HTML on the page or if it includes the necessary information to bootstrap a Blazor app from the user agent.</span></span>

<span data-ttu-id="f2248-138">Per ulteriori informazioni sull'helper Tag Component, incluso il passaggio di parametri e <xref:Microsoft.AspNetCore.Mvc.Rendering.RenderMode> configurazione, vedere <xref:mvc/views/tag-helpers/builtin-th/component-tag-helper> .</span><span class="sxs-lookup"><span data-stu-id="f2248-138">For more information on the Component Tag Helper, including passing parameters and <xref:Microsoft.AspNetCore.Mvc.Rendering.RenderMode> configuration, see <xref:mvc/views/tag-helpers/builtin-th/component-tag-helper>.</span></span>

<span data-ttu-id="f2248-139">Nell'esempio precedente è necessario che il layout dell'app Server ( `_Layout.cshtml` ) includa una [sezione di rendering](xref:mvc/views/layout#sections) ( <xref:Microsoft.AspNetCore.Mvc.Razor.RazorPage.RenderSection%2A> ) per lo script all'interno del tag di chiusura `</body>` :</span><span class="sxs-lookup"><span data-stu-id="f2248-139">The preceding example requires that the server app's layout (`_Layout.cshtml`) include a [render section](xref:mvc/views/layout#sections) (<xref:Microsoft.AspNetCore.Mvc.Razor.RazorPage.RenderSection%2A>) for the script inside the closing `</body>` tag:</span></span>

```cshtml
    ...

    @RenderSection("Scripts", required: false)
</body>
```

<span data-ttu-id="f2248-140">Il `_Layout.cshtml` file si trova nella `Pages/Shared` cartella in un' Razor app di pagine o `Views/Shared` in una cartella in un'app MVC.</span><span class="sxs-lookup"><span data-stu-id="f2248-140">The `_Layout.cshtml` file is located in the `Pages/Shared` folder in a Razor Pages app or `Views/Shared` folder in an MVC app.</span></span>

<span data-ttu-id="f2248-141">Se l'app deve anche applicare uno stile ai componenti con gli stili nell' Blazor WebAssembly app, includere gli stili dell'app nel `_Layout.cshtml` file.</span><span class="sxs-lookup"><span data-stu-id="f2248-141">If the app should also style components with the styles in the Blazor WebAssembly app, include the app's styles in the `_Layout.cshtml` file.</span></span> <span data-ttu-id="f2248-142">Nell'esempio seguente lo spazio dei nomi dell'app client è `BlazorHosted.Client` :</span><span class="sxs-lookup"><span data-stu-id="f2248-142">In the following example, the client app's namespace is `BlazorHosted.Client`:</span></span>

```cshtml
<head>
    ...

    <link href="css/app.css" rel="stylesheet" />
    <link href="BlazorHosted.Client.styles.css" rel="stylesheet" />
</head>
```

## <a name="render-components-in-a-page-or-view-with-a-css-selector"></a><span data-ttu-id="f2248-143">Eseguire il rendering dei componenti in una pagina o in una vista con un selettore CSS</span><span class="sxs-lookup"><span data-stu-id="f2248-143">Render components in a page or view with a CSS selector</span></span>

<span data-ttu-id="f2248-144">Aggiungere componenti radice al progetto *client* in `Program.Main` ( `Program.cs` ).</span><span class="sxs-lookup"><span data-stu-id="f2248-144">Add root components to the *Client* project in `Program.Main` (`Program.cs`).</span></span> <span data-ttu-id="f2248-145">Nell'esempio seguente il `Counter` componente viene dichiarato come componente radice con un selettore CSS che seleziona l'elemento con l'oggetto `id` che corrisponde a `my-counter` .</span><span class="sxs-lookup"><span data-stu-id="f2248-145">In the following example, the `Counter` component is declared as a root component with a CSS selector that selects the element with the `id` that matches `my-counter`.</span></span> <span data-ttu-id="f2248-146">Nell'esempio seguente lo spazio dei nomi dell'app client è `BlazorHosted.Client` :</span><span class="sxs-lookup"><span data-stu-id="f2248-146">In the following example, the client app's namespace is `BlazorHosted.Client`:</span></span>

```csharp
using BlazorHosted.Client.Pages;

...

builder.RootComponents.Add<Counter>("#my-counter");
```

<span data-ttu-id="f2248-147">Nell'esempio di Razor pagine seguente `Counter` viene eseguito il rendering del componente in una pagina.</span><span class="sxs-lookup"><span data-stu-id="f2248-147">In the following Razor Pages example, the `Counter` component is rendered in a page.</span></span> <span data-ttu-id="f2248-148">Per rendere interattivo il componente, lo Blazor WebAssembly script viene incluso nella [sezione di rendering](xref:mvc/views/layout#sections)della pagina:</span><span class="sxs-lookup"><span data-stu-id="f2248-148">To make the component interactive, the Blazor WebAssembly script is included in the page's [render section](xref:mvc/views/layout#sections):</span></span>

```cshtml
...

<h1>@ViewData["Title"]</h1>

<div id="my-counter">Loading...</div>

@section Scripts {
    <script src="_framework/blazor.webassembly.js"></script>
}
```

<span data-ttu-id="f2248-149">Nell'esempio precedente è necessario che il layout dell'app Server ( `_Layout.cshtml` ) includa una [sezione di rendering](xref:mvc/views/layout#sections) ( <xref:Microsoft.AspNetCore.Mvc.Razor.RazorPage.RenderSection%2A> ) per lo script all'interno del tag di chiusura `</body>` :</span><span class="sxs-lookup"><span data-stu-id="f2248-149">The preceding example requires that the server app's layout (`_Layout.cshtml`) include a [render section](xref:mvc/views/layout#sections) (<xref:Microsoft.AspNetCore.Mvc.Razor.RazorPage.RenderSection%2A>) for the script inside the closing `</body>` tag:</span></span>

```cshtml
    ...

    @RenderSection("Scripts", required: false)
</body>
```

<span data-ttu-id="f2248-150">Il `_Layout.cshtml` file si trova nella `Pages/Shared` cartella in un' Razor app di pagine o `Views/Shared` in una cartella in un'app MVC.</span><span class="sxs-lookup"><span data-stu-id="f2248-150">The `_Layout.cshtml` file is located in the `Pages/Shared` folder in a Razor Pages app or `Views/Shared` folder in an MVC app.</span></span>

<span data-ttu-id="f2248-151">Se l'app deve anche applicare uno stile ai componenti con gli stili nell' Blazor WebAssembly app, includere gli stili dell'app nel `_Layout.cshtml` file.</span><span class="sxs-lookup"><span data-stu-id="f2248-151">If the app should also style components with the styles in the Blazor WebAssembly app, include the app's styles in the `_Layout.cshtml` file.</span></span> <span data-ttu-id="f2248-152">Nell'esempio seguente lo spazio dei nomi dell'app client è `BlazorHosted.Client` :</span><span class="sxs-lookup"><span data-stu-id="f2248-152">In the following example, the client app's namespace is `BlazorHosted.Client`:</span></span>

```cshtml
<head>
    ...

    <link href="css/app.css" rel="stylesheet" />
    <link href="BlazorHosted.Client.styles.css" rel="stylesheet" />
</head>
```

::: moniker-end

::: moniker range="< aspnetcore-5.0"

<span data-ttu-id="f2248-153">L'integrazione di Razor componenti in Razor pagine e app MVC in una Blazor WebAssembly soluzione ospitata è supportata in ASP.NET Core in .NET 5 o versioni successive.</span><span class="sxs-lookup"><span data-stu-id="f2248-153">Integrating Razor components into Razor Pages and MVC apps in a hosted Blazor WebAssembly solution is supported in ASP.NET Core in .NET 5 or later.</span></span> <span data-ttu-id="f2248-154">Selezionare una versione di .NET 5 o successiva di questo articolo.</span><span class="sxs-lookup"><span data-stu-id="f2248-154">Select a .NET 5 or later version of this article.</span></span>

::: moniker-end

::: zone-end

::: zone pivot="server"

<span data-ttu-id="f2248-155">Razor i componenti possono essere integrati in Razor pagine e app MVC in un' Blazor Server app.</span><span class="sxs-lookup"><span data-stu-id="f2248-155">Razor components can be integrated into Razor Pages and MVC apps in a Blazor Server app.</span></span> <span data-ttu-id="f2248-156">Quando viene eseguito il rendering della pagina o della vista, è possibile eseguire il rendering dei componenti contemporaneamente.</span><span class="sxs-lookup"><span data-stu-id="f2248-156">When the page or view is rendered, components can be prerendered at the same time.</span></span>

<span data-ttu-id="f2248-157">Dopo aver [configurato l'app](#configuration), usare le istruzioni riportate nelle sezioni seguenti a seconda dei requisiti dell'app:</span><span class="sxs-lookup"><span data-stu-id="f2248-157">After [configuring the app](#configuration), use the guidance in the following sections depending on the app's requirements:</span></span>

* <span data-ttu-id="f2248-158">Componenti instradabili: per i componenti che sono direttamente instradabili dalle richieste degli utenti.</span><span class="sxs-lookup"><span data-stu-id="f2248-158">Routable components: For components that are directly routable from user requests.</span></span> <span data-ttu-id="f2248-159">Seguire queste linee guida quando i visitatori devono essere in grado di effettuare una richiesta HTTP nel browser per un componente con una [`@page`](xref:mvc/views/razor#page) direttiva.</span><span class="sxs-lookup"><span data-stu-id="f2248-159">Follow this guidance when visitors should be able to make an HTTP request in their browser for a component with an [`@page`](xref:mvc/views/razor#page) directive.</span></span>
  * [<span data-ttu-id="f2248-160">Usare componenti instradabili in un' Razor app pagine</span><span class="sxs-lookup"><span data-stu-id="f2248-160">Use routable components in a Razor Pages app</span></span>](#use-routable-components-in-a-razor-pages-app)
  * [<span data-ttu-id="f2248-161">Usare componenti instradabili in un'app MVC</span><span class="sxs-lookup"><span data-stu-id="f2248-161">Use routable components in an MVC app</span></span>](#use-routable-components-in-an-mvc-app)
* <span data-ttu-id="f2248-162">[Eseguire il rendering dei componenti da una pagina o vista](#render-components-from-a-page-or-view): per i componenti che non sono direttamente instradabili dalle richieste degli utenti.</span><span class="sxs-lookup"><span data-stu-id="f2248-162">[Render components from a page or view](#render-components-from-a-page-or-view): For components that aren't directly routable from user requests.</span></span> <span data-ttu-id="f2248-163">Seguire queste linee guida quando l'app incorpora i componenti in pagine e visualizzazioni esistenti con l' [Helper tag dei componenti](xref:mvc/views/tag-helpers/builtin-th/component-tag-helper).</span><span class="sxs-lookup"><span data-stu-id="f2248-163">Follow this guidance when the app embeds components into existing pages and views with the [Component Tag Helper](xref:mvc/views/tag-helpers/builtin-th/component-tag-helper).</span></span>

## <a name="configuration"></a><span data-ttu-id="f2248-164">Configurazione</span><span class="sxs-lookup"><span data-stu-id="f2248-164">Configuration</span></span>

<span data-ttu-id="f2248-165">Un' Razor app MVC o pagine esistente può integrare i Razor componenti in pagine e visualizzazioni:</span><span class="sxs-lookup"><span data-stu-id="f2248-165">An existing Razor Pages or MVC app can integrate Razor components into pages and views:</span></span>

1. <span data-ttu-id="f2248-166">Nel file di layout dell'app ( `_Layout.cshtml` ):</span><span class="sxs-lookup"><span data-stu-id="f2248-166">In the app's layout file (`_Layout.cshtml`):</span></span>

   * <span data-ttu-id="f2248-167">Aggiungere il `<base>` tag seguente all' `<head>` elemento:</span><span class="sxs-lookup"><span data-stu-id="f2248-167">Add the following `<base>` tag to the `<head>` element:</span></span>

     ```html
     <base href="~/" />
     ```

     <span data-ttu-id="f2248-168">Il `href` valore (il *percorso di base dell'app*) nell'esempio precedente presuppone che l'app si trovi nel percorso URL radice ( `/` ).</span><span class="sxs-lookup"><span data-stu-id="f2248-168">The `href` value (the *app base path*) in the preceding example assumes that the app resides at the root URL path (`/`).</span></span> <span data-ttu-id="f2248-169">Se l'app è un'applicazione secondaria, seguire le istruzioni nella sezione *percorso di base* dell'applicazione dell' <xref:blazor/host-and-deploy/index#app-base-path> articolo.</span><span class="sxs-lookup"><span data-stu-id="f2248-169">If the app is a sub-application, follow the guidance in the *App base path* section of the <xref:blazor/host-and-deploy/index#app-base-path> article.</span></span>

     <span data-ttu-id="f2248-170">Il `_Layout.cshtml` file si trova nella `Pages/Shared` cartella in un' Razor app di pagine o `Views/Shared` in una cartella in un'app MVC.</span><span class="sxs-lookup"><span data-stu-id="f2248-170">The `_Layout.cshtml` file is located in the `Pages/Shared` folder in a Razor Pages app or `Views/Shared` folder in an MVC app.</span></span>

   * <span data-ttu-id="f2248-171">Aggiungere un `<script>` tag per lo `blazor.server.js` script immediatamente prima della `Scripts` sezione Render:</span><span class="sxs-lookup"><span data-stu-id="f2248-171">Add a `<script>` tag for the `blazor.server.js` script immediately before the `Scripts` render section:</span></span>

     ```html
         ...
         <script src="_framework/blazor.server.js"></script>

         @await RenderSectionAsync("Scripts", required: false)
     </body>
     ```

     <span data-ttu-id="f2248-172">Il Framework aggiunge lo `blazor.server.js` script all'app.</span><span class="sxs-lookup"><span data-stu-id="f2248-172">The framework adds the `blazor.server.js` script to the app.</span></span> <span data-ttu-id="f2248-173">Non è necessario aggiungere manualmente lo script all'app.</span><span class="sxs-lookup"><span data-stu-id="f2248-173">There's no need to manually add the script to the app.</span></span>

1. <span data-ttu-id="f2248-174">Aggiungere un `_Imports.razor` file alla cartella radice del progetto con il contenuto seguente (modificare l'ultimo spazio dei nomi, `MyAppNamespace` , nello spazio dei nomi dell'app):</span><span class="sxs-lookup"><span data-stu-id="f2248-174">Add an `_Imports.razor` file to the root folder of the project with the following content (change the last namespace, `MyAppNamespace`, to the namespace of the app):</span></span>

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

1. <span data-ttu-id="f2248-175">In `Startup.ConfigureServices` registrare il Blazor Server servizio:</span><span class="sxs-lookup"><span data-stu-id="f2248-175">In `Startup.ConfigureServices`, register the Blazor Server service:</span></span>

   ```csharp
   services.AddServerSideBlazor();
   ```

1. <span data-ttu-id="f2248-176">In `Startup.Configure` aggiungere l' Blazor endpoint dell'hub a `app.UseEndpoints` :</span><span class="sxs-lookup"><span data-stu-id="f2248-176">In `Startup.Configure`, add the Blazor Hub endpoint to `app.UseEndpoints`:</span></span>

   ```csharp
   endpoints.MapBlazorHub();
   ```

1. <span data-ttu-id="f2248-177">Integrare i componenti in qualsiasi pagina o visualizzazione.</span><span class="sxs-lookup"><span data-stu-id="f2248-177">Integrate components into any page or view.</span></span> <span data-ttu-id="f2248-178">Per ulteriori informazioni, vedere la sezione [rendering dei componenti da una pagina o vista](#render-components-from-a-page-or-view) .</span><span class="sxs-lookup"><span data-stu-id="f2248-178">For more information, see the [Render components from a page or view](#render-components-from-a-page-or-view) section.</span></span>

## <a name="use-routable-components-in-a-razor-pages-app"></a><span data-ttu-id="f2248-179">Usare componenti instradabili in un' Razor app pagine</span><span class="sxs-lookup"><span data-stu-id="f2248-179">Use routable components in a Razor Pages app</span></span>

<span data-ttu-id="f2248-180">*Questa sezione riguarda l'aggiunta di componenti che sono direttamente instradabili dalle richieste degli utenti.*</span><span class="sxs-lookup"><span data-stu-id="f2248-180">*This section pertains to adding components that are directly routable from user requests.*</span></span>

<span data-ttu-id="f2248-181">Per supportare componenti instradabili Razor nelle Razor app di pagine:</span><span class="sxs-lookup"><span data-stu-id="f2248-181">To support routable Razor components in Razor Pages apps:</span></span>

1. <span data-ttu-id="f2248-182">Seguire le istruzioni riportate nella sezione di [configurazione](#configuration) .</span><span class="sxs-lookup"><span data-stu-id="f2248-182">Follow the guidance in the [Configuration](#configuration) section.</span></span>

1. <span data-ttu-id="f2248-183">Aggiungere un `App.razor` file alla radice del progetto con il contenuto seguente:</span><span class="sxs-lookup"><span data-stu-id="f2248-183">Add an `App.razor` file to the project root with the following content:</span></span>

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

1. <span data-ttu-id="f2248-184">Aggiungere un `_Host.cshtml` file alla `Pages` cartella con il contenuto seguente:</span><span class="sxs-lookup"><span data-stu-id="f2248-184">Add a `_Host.cshtml` file to the `Pages` folder with the following content:</span></span>

   ```cshtml
   @page "/blazor"
   @{
       Layout = "_Layout";
   }

   <app>
       <component type="typeof(App)" render-mode="ServerPrerendered" />
   </app>
   ```

   <span data-ttu-id="f2248-185">I componenti utilizzano il `_Layout.cshtml` file condiviso per il layout.</span><span class="sxs-lookup"><span data-stu-id="f2248-185">Components use the shared `_Layout.cshtml` file for their layout.</span></span>

   <span data-ttu-id="f2248-186"><xref:Microsoft.AspNetCore.Mvc.Rendering.RenderMode> Configura se il `App` componente:</span><span class="sxs-lookup"><span data-stu-id="f2248-186"><xref:Microsoft.AspNetCore.Mvc.Rendering.RenderMode> configures whether the `App` component:</span></span>

   * <span data-ttu-id="f2248-187">Viene preeseguito nella pagina.</span><span class="sxs-lookup"><span data-stu-id="f2248-187">Is prerendered into the page.</span></span>
   * <span data-ttu-id="f2248-188">Viene sottoposto a rendering come HTML statico nella pagina o se include le informazioni necessarie per il bootstrap di un' Blazor app dall'agente utente.</span><span class="sxs-lookup"><span data-stu-id="f2248-188">Is rendered as static HTML on the page or if it includes the necessary information to bootstrap a Blazor app from the user agent.</span></span>

   <span data-ttu-id="f2248-189">Per ulteriori informazioni sull'helper Tag Component, incluso il passaggio di parametri e <xref:Microsoft.AspNetCore.Mvc.Rendering.RenderMode> configurazione, vedere <xref:mvc/views/tag-helpers/builtin-th/component-tag-helper> .</span><span class="sxs-lookup"><span data-stu-id="f2248-189">For more information on the Component Tag Helper, including passing parameters and <xref:Microsoft.AspNetCore.Mvc.Rendering.RenderMode> configuration, see <xref:mvc/views/tag-helpers/builtin-th/component-tag-helper>.</span></span>

1. <span data-ttu-id="f2248-190">Aggiungere una route con priorità bassa per la `_Host.cshtml` pagina alla configurazione dell'endpoint in `Startup.Configure` :</span><span class="sxs-lookup"><span data-stu-id="f2248-190">Add a low-priority route for the `_Host.cshtml` page to endpoint configuration in `Startup.Configure`:</span></span>

   ```csharp
   app.UseEndpoints(endpoints =>
   {
       ...

       endpoints.MapFallbackToPage("/_Host");
   });
   ```

1. <span data-ttu-id="f2248-191">Aggiungere componenti instradabili all'app.</span><span class="sxs-lookup"><span data-stu-id="f2248-191">Add routable components to the app.</span></span> <span data-ttu-id="f2248-192">Ad esempio:</span><span class="sxs-lookup"><span data-stu-id="f2248-192">For example:</span></span>

   ```razor
   @page "/counter"

   <h1>Counter</h1>

   ...
   ```

<span data-ttu-id="f2248-193">Per ulteriori informazioni sugli spazi dei nomi, vedere la sezione relativa agli [spazi dei nomi dei componenti](#component-namespaces) .</span><span class="sxs-lookup"><span data-stu-id="f2248-193">For more information on namespaces, see the [Component namespaces](#component-namespaces) section.</span></span>

## <a name="use-routable-components-in-an-mvc-app"></a><span data-ttu-id="f2248-194">Usare componenti instradabili in un'app MVC</span><span class="sxs-lookup"><span data-stu-id="f2248-194">Use routable components in an MVC app</span></span>

<span data-ttu-id="f2248-195">*Questa sezione riguarda l'aggiunta di componenti che sono direttamente instradabili dalle richieste degli utenti.*</span><span class="sxs-lookup"><span data-stu-id="f2248-195">*This section pertains to adding components that are directly routable from user requests.*</span></span>

<span data-ttu-id="f2248-196">Per supportare componenti instradabili Razor nelle app MVC:</span><span class="sxs-lookup"><span data-stu-id="f2248-196">To support routable Razor components in MVC apps:</span></span>

1. <span data-ttu-id="f2248-197">Seguire le istruzioni riportate nella sezione di [configurazione](#configuration) .</span><span class="sxs-lookup"><span data-stu-id="f2248-197">Follow the guidance in the [Configuration](#configuration) section.</span></span>

1. <span data-ttu-id="f2248-198">Aggiungere un `App.razor` file alla radice del progetto con il contenuto seguente:</span><span class="sxs-lookup"><span data-stu-id="f2248-198">Add an `App.razor` file to the root of the project with the following content:</span></span>

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

1. <span data-ttu-id="f2248-199">Aggiungere un `_Host.cshtml` file alla `Views/Home` cartella con il contenuto seguente:</span><span class="sxs-lookup"><span data-stu-id="f2248-199">Add a `_Host.cshtml` file to the `Views/Home` folder with the following content:</span></span>

   ```cshtml
   @{
       Layout = "_Layout";
   }

   <app>
       <component type="typeof(App)" render-mode="ServerPrerendered" />
   </app>
   ```

   <span data-ttu-id="f2248-200">I componenti utilizzano il `_Layout.cshtml` file condiviso per il layout.</span><span class="sxs-lookup"><span data-stu-id="f2248-200">Components use the shared `_Layout.cshtml` file for their layout.</span></span>

   <span data-ttu-id="f2248-201"><xref:Microsoft.AspNetCore.Mvc.Rendering.RenderMode> Configura se il `App` componente:</span><span class="sxs-lookup"><span data-stu-id="f2248-201"><xref:Microsoft.AspNetCore.Mvc.Rendering.RenderMode> configures whether the `App` component:</span></span>

   * <span data-ttu-id="f2248-202">Viene preeseguito nella pagina.</span><span class="sxs-lookup"><span data-stu-id="f2248-202">Is prerendered into the page.</span></span>
   * <span data-ttu-id="f2248-203">Viene sottoposto a rendering come HTML statico nella pagina o se include le informazioni necessarie per il bootstrap di un' Blazor app dall'agente utente.</span><span class="sxs-lookup"><span data-stu-id="f2248-203">Is rendered as static HTML on the page or if it includes the necessary information to bootstrap a Blazor app from the user agent.</span></span>

   <span data-ttu-id="f2248-204">Per ulteriori informazioni sull'helper Tag Component, incluso il passaggio di parametri e <xref:Microsoft.AspNetCore.Mvc.Rendering.RenderMode> configurazione, vedere <xref:mvc/views/tag-helpers/builtin-th/component-tag-helper> .</span><span class="sxs-lookup"><span data-stu-id="f2248-204">For more information on the Component Tag Helper, including passing parameters and <xref:Microsoft.AspNetCore.Mvc.Rendering.RenderMode> configuration, see <xref:mvc/views/tag-helpers/builtin-th/component-tag-helper>.</span></span>

1. <span data-ttu-id="f2248-205">Aggiungere un'azione al controller Home:</span><span class="sxs-lookup"><span data-stu-id="f2248-205">Add an action to the Home controller:</span></span>

   ```csharp
   public IActionResult Blazor()
   {
      return View("_Host");
   }
   ```

1. <span data-ttu-id="f2248-206">Aggiungere una route con priorità bassa per l'azione del controller che restituisce la `_Host.cshtml` visualizzazione alla configurazione dell'endpoint in `Startup.Configure` :</span><span class="sxs-lookup"><span data-stu-id="f2248-206">Add a low-priority route for the controller action that returns the `_Host.cshtml` view to the endpoint configuration in `Startup.Configure`:</span></span>

   ```csharp
   app.UseEndpoints(endpoints =>
   {
       ...

       endpoints.MapFallbackToController("Blazor", "Home");
   });
   ```

1. <span data-ttu-id="f2248-207">Creare una `Pages` cartella e aggiungere componenti instradabili all'app.</span><span class="sxs-lookup"><span data-stu-id="f2248-207">Create a `Pages` folder and add routable components to the app.</span></span> <span data-ttu-id="f2248-208">Ad esempio:</span><span class="sxs-lookup"><span data-stu-id="f2248-208">For example:</span></span>

   ```razor
   @page "/counter"

   <h1>Counter</h1>

   ...
   ```

<span data-ttu-id="f2248-209">Per ulteriori informazioni sugli spazi dei nomi, vedere la sezione relativa agli [spazi dei nomi dei componenti](#component-namespaces) .</span><span class="sxs-lookup"><span data-stu-id="f2248-209">For more information on namespaces, see the [Component namespaces](#component-namespaces) section.</span></span>

## <a name="render-components-from-a-page-or-view"></a><span data-ttu-id="f2248-210">Eseguire il rendering dei componenti da una pagina o da una vista</span><span class="sxs-lookup"><span data-stu-id="f2248-210">Render components from a page or view</span></span>

<span data-ttu-id="f2248-211">*Questa sezione riguarda l'aggiunta di componenti a pagine o viste, in cui i componenti non sono direttamente instradabili dalle richieste degli utenti.*</span><span class="sxs-lookup"><span data-stu-id="f2248-211">*This section pertains to adding components to pages or views, where the components aren't directly routable from user requests.*</span></span>

<span data-ttu-id="f2248-212">Per eseguire il rendering di un componente da una pagina o da una vista, usare l' [Helper Tag Component](xref:mvc/views/tag-helpers/builtin-th/component-tag-helper).</span><span class="sxs-lookup"><span data-stu-id="f2248-212">To render a component from a page or view, use the [Component Tag Helper](xref:mvc/views/tag-helpers/builtin-th/component-tag-helper).</span></span>

### <a name="render-stateful-interactive-components"></a><span data-ttu-id="f2248-213">Eseguire il rendering di componenti interattivi con stato</span><span class="sxs-lookup"><span data-stu-id="f2248-213">Render stateful interactive components</span></span>

<span data-ttu-id="f2248-214">I componenti interattivi con stato possono essere aggiunti a una Razor pagina o a una vista.</span><span class="sxs-lookup"><span data-stu-id="f2248-214">Stateful interactive components can be added to a Razor page or view.</span></span>

<span data-ttu-id="f2248-215">Quando viene eseguito il rendering della pagina o della visualizzazione:</span><span class="sxs-lookup"><span data-stu-id="f2248-215">When the page or view renders:</span></span>

* <span data-ttu-id="f2248-216">Il componente viene preeseguito con la pagina o la visualizzazione.</span><span class="sxs-lookup"><span data-stu-id="f2248-216">The component is prerendered with the page or view.</span></span>
* <span data-ttu-id="f2248-217">Lo stato iniziale del componente utilizzato per il prerendering viene perso.</span><span class="sxs-lookup"><span data-stu-id="f2248-217">The initial component state used for prerendering is lost.</span></span>
* <span data-ttu-id="f2248-218">Quando viene stabilita la connessione, viene creato il nuovo stato del componente SignalR .</span><span class="sxs-lookup"><span data-stu-id="f2248-218">New component state is created when the SignalR connection is established.</span></span>

<span data-ttu-id="f2248-219">Nella pagina seguente viene Razor eseguito il rendering di un `Counter` componente:</span><span class="sxs-lookup"><span data-stu-id="f2248-219">The following Razor page renders a `Counter` component:</span></span>

```cshtml
<h1>My Razor Page</h1>

<component type="typeof(Counter)" render-mode="ServerPrerendered" 
    param-InitialValue="InitialValue" />

@functions {
    [BindProperty(SupportsGet=true)]
    public int InitialValue { get; set; }
}
```

<span data-ttu-id="f2248-220">Per altre informazioni, vedere <xref:mvc/views/tag-helpers/builtin-th/component-tag-helper>.</span><span class="sxs-lookup"><span data-stu-id="f2248-220">For more information, see <xref:mvc/views/tag-helpers/builtin-th/component-tag-helper>.</span></span>

### <a name="render-noninteractive-components"></a><span data-ttu-id="f2248-221">Eseguire il rendering di componenti non interattivi</span><span class="sxs-lookup"><span data-stu-id="f2248-221">Render noninteractive components</span></span>

<span data-ttu-id="f2248-222">Nella pagina seguente Razor il componente viene sottoposto a `Counter` rendering statico con un valore iniziale specificato utilizzando un form.</span><span class="sxs-lookup"><span data-stu-id="f2248-222">In the following Razor page, the `Counter` component is statically rendered with an initial value that's specified using a form.</span></span> <span data-ttu-id="f2248-223">Poiché il componente viene sottoposto a rendering statico, il componente non è interattivo:</span><span class="sxs-lookup"><span data-stu-id="f2248-223">Since the component is statically rendered, the component isn't interactive:</span></span>

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

<span data-ttu-id="f2248-224">Per altre informazioni, vedere <xref:mvc/views/tag-helpers/builtin-th/component-tag-helper>.</span><span class="sxs-lookup"><span data-stu-id="f2248-224">For more information, see <xref:mvc/views/tag-helpers/builtin-th/component-tag-helper>.</span></span>

## <a name="component-namespaces"></a><span data-ttu-id="f2248-225">Spazi dei nomi dei componenti</span><span class="sxs-lookup"><span data-stu-id="f2248-225">Component namespaces</span></span>

<span data-ttu-id="f2248-226">Quando si usa una cartella personalizzata per conservare i componenti dell'app, aggiungere lo spazio dei nomi che rappresenta la cartella alla pagina/visualizzazione o al `_ViewImports.cshtml` file.</span><span class="sxs-lookup"><span data-stu-id="f2248-226">When using a custom folder to hold the app's components, add the namespace representing the folder to either the page/view or to the `_ViewImports.cshtml` file.</span></span> <span data-ttu-id="f2248-227">Nell'esempio seguente:</span><span class="sxs-lookup"><span data-stu-id="f2248-227">In the following example:</span></span>

* <span data-ttu-id="f2248-228">Passare `MyAppNamespace` allo spazio dei nomi dell'app.</span><span class="sxs-lookup"><span data-stu-id="f2248-228">Change `MyAppNamespace` to the app's namespace.</span></span>
* <span data-ttu-id="f2248-229">Se non viene usata una cartella denominata `Components` per conservare i componenti, passare `Components` alla cartella in cui si trovano i componenti.</span><span class="sxs-lookup"><span data-stu-id="f2248-229">If a folder named `Components` isn't used to hold the components, change `Components` to the folder where the components reside.</span></span>

```cshtml
@using MyAppNamespace.Components
```

<span data-ttu-id="f2248-230">Il `_ViewImports.cshtml` file si trova nella `Pages` cartella dell' Razor app pagine o nella `Views` cartella di un'app MVC.</span><span class="sxs-lookup"><span data-stu-id="f2248-230">The `_ViewImports.cshtml` file is located in the `Pages` folder of a Razor Pages app or the `Views` folder of an MVC app.</span></span>

<span data-ttu-id="f2248-231">Per altre informazioni, vedere <xref:blazor/components/index#namespaces>.</span><span class="sxs-lookup"><span data-stu-id="f2248-231">For more information, see <xref:blazor/components/index#namespaces>.</span></span>

::: zone-end
