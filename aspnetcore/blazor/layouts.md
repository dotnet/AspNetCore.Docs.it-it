---
title: BlazorLayout ASP.NET Core
author: guardrex
description: Informazioni su come creare componenti di layout riutilizzabili per le Blazor app.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 03/02/2021
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
uid: blazor/layouts
ms.openlocfilehash: 9f3400b766a043fdf3872838bffe392eddba4049
ms.sourcegitcommit: a1db01b4d3bd8c57d7a9c94ce122a6db68002d66
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/04/2021
ms.locfileid: "102394947"
---
# <a name="aspnet-core-blazor-layouts"></a><span data-ttu-id="e5f27-103">BlazorLayout ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="e5f27-103">ASP.NET Core Blazor layouts</span></span>

<span data-ttu-id="e5f27-104">Alcuni elementi dell'app, ad esempio i menu, i messaggi di copyright e i logo aziendali, fanno in genere parte della presentazione complessiva dell'app.</span><span class="sxs-lookup"><span data-stu-id="e5f27-104">Some app elements, such as menus, copyright messages, and company logos, are usually part of app's overall presentation.</span></span> <span data-ttu-id="e5f27-105">L'inserimento di una copia del markup per questi elementi in tutti i componenti di un'app non è efficiente.</span><span class="sxs-lookup"><span data-stu-id="e5f27-105">Placing a copy of the markup for these elements into all of the components of an app isn't efficient.</span></span> <span data-ttu-id="e5f27-106">Ogni volta che uno di questi elementi viene aggiornato, è necessario aggiornare ogni componente che usa l'elemento.</span><span class="sxs-lookup"><span data-stu-id="e5f27-106">Every time that one of these elements is updated, every component that uses the element must be updated.</span></span> <span data-ttu-id="e5f27-107">Questo approccio è costoso da gestire e può causare contenuti incoerenti in caso di mancato aggiornamento.</span><span class="sxs-lookup"><span data-stu-id="e5f27-107">This approach is costly to maintain and can lead to inconsistent content if an update is missed.</span></span> <span data-ttu-id="e5f27-108">I *layout* risolvono questi problemi.</span><span class="sxs-lookup"><span data-stu-id="e5f27-108">*Layouts* solve these problems.</span></span>

<span data-ttu-id="e5f27-109">Un Blazor layout è un Razor componente che condivide il markup con i componenti che vi fanno riferimento.</span><span class="sxs-lookup"><span data-stu-id="e5f27-109">A Blazor layout is a Razor component that shares markup with components that reference it.</span></span> <span data-ttu-id="e5f27-110">I layout possono usare [Data Binding](xref:blazor/components/data-binding), l' [inserimento delle dipendenze](xref:blazor/fundamentals/dependency-injection)e altre funzionalità dei componenti.</span><span class="sxs-lookup"><span data-stu-id="e5f27-110">Layouts can use [data binding](xref:blazor/components/data-binding), [dependency injection](xref:blazor/fundamentals/dependency-injection), and other features of components.</span></span>

## <a name="layout-components"></a><span data-ttu-id="e5f27-111">Componenti layout</span><span class="sxs-lookup"><span data-stu-id="e5f27-111">Layout components</span></span>

### <a name="create-a-layout-component"></a><span data-ttu-id="e5f27-112">Creazione di un componente di layout</span><span class="sxs-lookup"><span data-stu-id="e5f27-112">Create a layout component</span></span>

<span data-ttu-id="e5f27-113">Per creare un componente di layout:</span><span class="sxs-lookup"><span data-stu-id="e5f27-113">To create a layout component:</span></span>

* <span data-ttu-id="e5f27-114">Creare un Razor componente definito da un Razor modello o da un codice C#.</span><span class="sxs-lookup"><span data-stu-id="e5f27-114">Create a Razor component defined by a Razor template or C# code.</span></span> <span data-ttu-id="e5f27-115">I componenti di layout basati su un Razor modello usano l' `.razor` estensione del file proprio come i Razor componenti normali.</span><span class="sxs-lookup"><span data-stu-id="e5f27-115">Layout components based on a Razor template use the `.razor` file extension just like ordinary Razor components.</span></span> <span data-ttu-id="e5f27-116">Poiché i componenti di layout sono condivisi tra i componenti di un'app, vengono in genere inseriti nella `Shared` cartella dell'app.</span><span class="sxs-lookup"><span data-stu-id="e5f27-116">Because layout components are shared across an app's components, they're usually placed in the app's `Shared` folder.</span></span> <span data-ttu-id="e5f27-117">Tuttavia, i layout possono essere posizionati in qualsiasi posizione accessibile ai componenti che lo usano.</span><span class="sxs-lookup"><span data-stu-id="e5f27-117">However, layouts can be placed in any location accessible to the components that use it.</span></span> <span data-ttu-id="e5f27-118">Un layout, ad esempio, può essere inserito nella stessa cartella dei componenti che lo usano.</span><span class="sxs-lookup"><span data-stu-id="e5f27-118">For example, a layout can be placed in the same folder as the components that use it.</span></span>
* <span data-ttu-id="e5f27-119">Ereditare il componente da <xref:Microsoft.AspNetCore.Components.LayoutComponentBase> .</span><span class="sxs-lookup"><span data-stu-id="e5f27-119">Inherit the component from <xref:Microsoft.AspNetCore.Components.LayoutComponentBase>.</span></span> <span data-ttu-id="e5f27-120"><xref:Microsoft.AspNetCore.Components.LayoutComponentBase>Definisce una <xref:Microsoft.AspNetCore.Components.LayoutComponentBase.Body> Proprietà ( <xref:Microsoft.AspNetCore.Components.RenderFragment> tipo) per il contenuto di cui è stato eseguito il rendering all'interno del layout.</span><span class="sxs-lookup"><span data-stu-id="e5f27-120">The <xref:Microsoft.AspNetCore.Components.LayoutComponentBase> defines a <xref:Microsoft.AspNetCore.Components.LayoutComponentBase.Body> property (<xref:Microsoft.AspNetCore.Components.RenderFragment> type) for the rendered content inside the layout.</span></span>
* <span data-ttu-id="e5f27-121">Utilizzare la Razor sintassi `@Body` per specificare il percorso nel markup di layout in cui viene eseguito il rendering del contenuto.</span><span class="sxs-lookup"><span data-stu-id="e5f27-121">Use the Razor syntax `@Body` to specify the location in the layout markup where the content is rendered.</span></span>

<span data-ttu-id="e5f27-122">Il `DoctorWhoLayout` componente seguente mostra il Razor modello di un componente di layout.</span><span class="sxs-lookup"><span data-stu-id="e5f27-122">The following `DoctorWhoLayout` component shows the Razor template of a layout component.</span></span> <span data-ttu-id="e5f27-123">Il layout eredita <xref:Microsoft.AspNetCore.Components.LayoutComponentBase> e imposta l'oggetto `@Body` tra la barra di spostamento ( `<nav>...</nav>` ) e il piè di pagina ( `<footer>...</footer>` ).</span><span class="sxs-lookup"><span data-stu-id="e5f27-123">The layout inherits <xref:Microsoft.AspNetCore.Components.LayoutComponentBase> and sets the `@Body` between the navigation bar (`<nav>...</nav>`) and the footer (`<footer>...</footer>`).</span></span>

<span data-ttu-id="e5f27-124">`Shared/DoctorWhoLayout.razor`:</span><span class="sxs-lookup"><span data-stu-id="e5f27-124">`Shared/DoctorWhoLayout.razor`:</span></span>

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Shared/layouts/DoctorWhoLayout.razor?highlight=1,13)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Shared/layouts/DoctorWhoLayout.razor?highlight=1,13)]

::: moniker-end

### <a name="mainlayout-component"></a><span data-ttu-id="e5f27-125">Componente `MainLayout`</span><span class="sxs-lookup"><span data-stu-id="e5f27-125">`MainLayout` component</span></span>

<span data-ttu-id="e5f27-126">In un'app creata da un [ Blazor modello di progetto](xref:blazor/project-structure), il `MainLayout` componente è il [layout predefinito](#apply-a-default-layout-to-an-app)dell'app.</span><span class="sxs-lookup"><span data-stu-id="e5f27-126">In an app created from a [Blazor project template](xref:blazor/project-structure), the `MainLayout` component is the app's [default layout](#apply-a-default-layout-to-an-app).</span></span>

<span data-ttu-id="e5f27-127">`Shared/MainLayout.razor`:</span><span class="sxs-lookup"><span data-stu-id="e5f27-127">`Shared/MainLayout.razor`:</span></span>

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Shared/layouts/MainLayout.razor)]

<span data-ttu-id="e5f27-128">la [ Blazor funzionalità di isolamento CSS applica gli](xref:blazor/components/css-isolation) stili CSS isolati al `MainLayout` componente.</span><span class="sxs-lookup"><span data-stu-id="e5f27-128">[Blazor's CSS isolation feature](xref:blazor/components/css-isolation) applies isolated CSS styles to the `MainLayout` component.</span></span> <span data-ttu-id="e5f27-129">Per convenzione, gli stili vengono forniti dal foglio di stile associato con lo stesso nome `Shared/MainLayout.razor.css` .</span><span class="sxs-lookup"><span data-stu-id="e5f27-129">By convention, the styles are provided by the accompanying stylesheet of the same name, `Shared/MainLayout.razor.css`.</span></span> <span data-ttu-id="e5f27-130">L'implementazione del ASP.NET Core Framework del foglio di stile è disponibile per l'ispezione nell' [origine riferimento ASP.NET Core (repository GitHub DotNet/aspnetcore)](https://github.com/dotnet/aspnetcore/blob/main/src/ProjectTemplates/Web.ProjectTemplates/content/ComponentsWebAssembly-CSharp/Client/Shared/MainLayout.razor.css).</span><span class="sxs-lookup"><span data-stu-id="e5f27-130">The ASP.NET Core framework implementation of the stylesheet is available for inspection in the [ASP.NET Core reference source (dotnet/aspnetcore GitHub repository)](https://github.com/dotnet/aspnetcore/blob/main/src/ProjectTemplates/Web.ProjectTemplates/content/ComponentsWebAssembly-CSharp/Client/Shared/MainLayout.razor.css).</span></span>

[!INCLUDE[](~/blazor/includes/aspnetcore-repo-ref-source-links.md)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Shared/layouts/MainLayout.razor)]

::: moniker-end

## <a name="apply-a-layout"></a><span data-ttu-id="e5f27-131">Applicare un layout</span><span class="sxs-lookup"><span data-stu-id="e5f27-131">Apply a layout</span></span>

### <a name="apply-a-layout-to-a-component"></a><span data-ttu-id="e5f27-132">Applicare un layout a un componente</span><span class="sxs-lookup"><span data-stu-id="e5f27-132">Apply a layout to a component</span></span>

<span data-ttu-id="e5f27-133">Utilizzare la [`@layout`](xref:mvc/views/razor#layout) Razor direttiva per applicare un layout a un componente instradabile con Razor una [`@page`](xref:mvc/views/razor#page) direttiva.</span><span class="sxs-lookup"><span data-stu-id="e5f27-133">Use the [`@layout`](xref:mvc/views/razor#layout) Razor directive to apply a layout to a routable Razor component that has an [`@page`](xref:mvc/views/razor#page) directive.</span></span> <span data-ttu-id="e5f27-134">Il compilatore converte `@layout` in un oggetto <xref:Microsoft.AspNetCore.Components.LayoutAttribute> e applica l'attributo alla classe Component.</span><span class="sxs-lookup"><span data-stu-id="e5f27-134">The compiler converts `@layout` into a <xref:Microsoft.AspNetCore.Components.LayoutAttribute> and applies the attribute to the component class.</span></span>

<span data-ttu-id="e5f27-135">Il contenuto del componente seguente `Episodes` viene inserito in `DoctorWhoLayout` in corrispondenza della posizione di `@Body` .</span><span class="sxs-lookup"><span data-stu-id="e5f27-135">The content of the following `Episodes` component is inserted into the `DoctorWhoLayout` at the position of `@Body`.</span></span>

<span data-ttu-id="e5f27-136">`Pages/Episodes.razor`:</span><span class="sxs-lookup"><span data-stu-id="e5f27-136">`Pages/Episodes.razor`:</span></span>

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/layouts/Episodes.razor?highlight=2)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Pages/layouts/Episodes.razor?highlight=2)]

::: moniker-end

<span data-ttu-id="e5f27-137">Il markup HTML sottoposto a rendering seguente viene generato `DoctorWhoLayout` dal `Episodes` componente e precedente.</span><span class="sxs-lookup"><span data-stu-id="e5f27-137">The following rendered HTML markup is produced by the preceding `DoctorWhoLayout` and `Episodes` component.</span></span> <span data-ttu-id="e5f27-138">Il markup estraneo non viene visualizzato per concentrarsi sul contenuto fornito dai due componenti interessati:</span><span class="sxs-lookup"><span data-stu-id="e5f27-138">Extraneous markup doesn't appear in order to focus on the content provided by the two components involved:</span></span>

* <span data-ttu-id="e5f27-139">L'intestazione del **&trade; database dell'episodio Doctor** () `<h1>...</h1>` nell'intestazione ( `<header>...</header>` ), nella barra di spostamento ( `<nav>...</nav>` ) e nell'elemento informazioni sul marchio () del piè di pagina `<div>...</div>` ( `<footer>...</footer>` ) proviene dal `DoctorWhoLayout` componente.</span><span class="sxs-lookup"><span data-stu-id="e5f27-139">The **Doctor Who&trade; Episode Database** heading (`<h1>...</h1>`) in the header (`<header>...</header>`), navigation bar (`<nav>...</nav>`), and trademark information element (`<div>...</div>`) in the footer (`<footer>...</footer>`) come from the `DoctorWhoLayout` component.</span></span>
* <span data-ttu-id="e5f27-140">L'intestazione degli **episodi** ( `<h2>...</h2>` ) e l'elenco di puntatori ( `<ul>...</ul>` ) provengono dal `Episodes` componente.</span><span class="sxs-lookup"><span data-stu-id="e5f27-140">The **Episodes** heading (`<h2>...</h2>`) and episode list (`<ul>...</ul>`) come from the `Episodes` component.</span></span>

```html
<body>
    <div id="app">
        <header>
            <h1>Doctor Who&trade; Episode Database</h1>
        </header>

        <nav>
            <a href="masterlist">Master Episode List</a>
            <a href="search">Search</a>
            <a href="new">Add Episode</a>
        </nav>

        <h2>Episodes</h2>

        <ul>
            <li>...</li>
            <li>...</li>
            <li>...</li>
        </ul>

        <footer>
            Doctor Who is a registered trademark of the BBC. 
            https://www.doctorwho.tv/
        </footer>
    </div>
</body>
```

<span data-ttu-id="e5f27-141">La specifica del layout direttamente in un componente sostituisce un *layout predefinito*:</span><span class="sxs-lookup"><span data-stu-id="e5f27-141">Specifying the layout directly in a component overrides a *default layout*:</span></span>

* <span data-ttu-id="e5f27-142">Impostata da una `@layout` direttiva importata da un `_Imports` componente ( `_Imports.razor` ), come descritto nella sezione [applicare un layout a una cartella di componenti](#apply-a-layout-to-a-folder-of-components) .</span><span class="sxs-lookup"><span data-stu-id="e5f27-142">Set by an `@layout` directive imported from an `_Imports` component (`_Imports.razor`), as described in the following [Apply a layout to a folder of components](#apply-a-layout-to-a-folder-of-components) section.</span></span>
* <span data-ttu-id="e5f27-143">Impostare come layout predefinito dell'app, come descritto nella sezione [applicare un layout predefinito a un'app](#apply-a-default-layout-to-an-app) più avanti in questo articolo.</span><span class="sxs-lookup"><span data-stu-id="e5f27-143">Set as the app's default layout, as described in the [Apply a default layout to an app](#apply-a-default-layout-to-an-app) section later in this article.</span></span>

### <a name="apply-a-layout-to-a-folder-of-components"></a><span data-ttu-id="e5f27-144">Applicare un layout a una cartella di componenti</span><span class="sxs-lookup"><span data-stu-id="e5f27-144">Apply a layout to a folder of components</span></span>

<span data-ttu-id="e5f27-145">Ogni cartella di un'app può contenere facoltativamente un file modello denominato `_Imports.razor` .</span><span class="sxs-lookup"><span data-stu-id="e5f27-145">Every folder of an app can optionally contain a template file named `_Imports.razor`.</span></span> <span data-ttu-id="e5f27-146">Il compilatore include le direttive specificate nel file Imports in tutti i Razor modelli nella stessa cartella e in modo ricorsivo in tutte le relative sottocartelle.</span><span class="sxs-lookup"><span data-stu-id="e5f27-146">The compiler includes the directives specified in the imports file in all of the Razor templates in the same folder and recursively in all of its subfolders.</span></span> <span data-ttu-id="e5f27-147">Pertanto, un `_Imports.razor` file contenente `@layout DoctorWhoLayout` garantisce che tutti i componenti di una cartella usino il `DoctorWhoLayout` componente.</span><span class="sxs-lookup"><span data-stu-id="e5f27-147">Therefore, an `_Imports.razor` file containing `@layout DoctorWhoLayout` ensures that all of the components in a folder use the `DoctorWhoLayout` component.</span></span> <span data-ttu-id="e5f27-148">Non è necessario aggiungere ripetutamente `@layout DoctorWhoLayout` a tutti i Razor componenti ( `.razor` ) nella cartella e nelle sottocartelle.</span><span class="sxs-lookup"><span data-stu-id="e5f27-148">There's no need to repeatedly add `@layout DoctorWhoLayout` to all of the Razor components (`.razor`) within the folder and subfolders.</span></span>

<span data-ttu-id="e5f27-149">`_Imports.razor`:</span><span class="sxs-lookup"><span data-stu-id="e5f27-149">`_Imports.razor`:</span></span>

```razor
@layout DoctorWhoLayout
...
```

<span data-ttu-id="e5f27-150">Il `_Imports.razor` file è simile al [file _ViewImports. cshtml per le Razor visualizzazioni e le pagine,](xref:mvc/views/layout#importing-shared-directives) ma viene applicato in modo specifico ai Razor file del componente.</span><span class="sxs-lookup"><span data-stu-id="e5f27-150">The `_Imports.razor` file is similar to the [_ViewImports.cshtml file for Razor views and pages](xref:mvc/views/layout#importing-shared-directives) but applied specifically to Razor component files.</span></span>

<span data-ttu-id="e5f27-151">La specifica di un layout in `_Imports.razor` sostituisce un layout specificato come [layout predefinito dell'app](#apply-a-default-layout-to-an-app)del router, descritto nella sezione seguente.</span><span class="sxs-lookup"><span data-stu-id="e5f27-151">Specifying a layout in `_Imports.razor` overrides a layout specified as the router's [default app layout](#apply-a-default-layout-to-an-app), which is described in the following section.</span></span>

> [!WARNING]
> <span data-ttu-id="e5f27-152">**Non** aggiungere una Razor `@layout` direttiva al `_Imports.razor` file radice, il che comporta un ciclo infinito di layout.</span><span class="sxs-lookup"><span data-stu-id="e5f27-152">Do **not** add a Razor `@layout` directive to the root `_Imports.razor` file, which results in an infinite loop of layouts.</span></span> <span data-ttu-id="e5f27-153">Per controllare il layout predefinito dell'app, specificare il layout nel `Router` componente.</span><span class="sxs-lookup"><span data-stu-id="e5f27-153">To control the default app layout, specify the layout in the `Router` component.</span></span> <span data-ttu-id="e5f27-154">Per altre informazioni, vedere la sezione [applicare un layout predefinito a un'app](#apply-a-default-layout-to-an-app) .</span><span class="sxs-lookup"><span data-stu-id="e5f27-154">For more information, see the following [Apply a default layout to an app](#apply-a-default-layout-to-an-app) section.</span></span>

> [!NOTE]
> <span data-ttu-id="e5f27-155">La [`@layout`](xref:mvc/views/razor#layout) Razor direttiva applica solo un layout ai componenti instradabili Razor con una [`@page`](xref:mvc/views/razor#page) direttiva.</span><span class="sxs-lookup"><span data-stu-id="e5f27-155">The [`@layout`](xref:mvc/views/razor#layout) Razor directive only applies a layout to routable Razor components with an [`@page`](xref:mvc/views/razor#page) directive.</span></span>

### <a name="apply-a-default-layout-to-an-app"></a><span data-ttu-id="e5f27-156">Applicare un layout predefinito a un'app</span><span class="sxs-lookup"><span data-stu-id="e5f27-156">Apply a default layout to an app</span></span>

<span data-ttu-id="e5f27-157">Specificare il layout dell'app predefinito in nel `App` componente del componente <xref:Microsoft.AspNetCore.Components.Routing.Router> .</span><span class="sxs-lookup"><span data-stu-id="e5f27-157">Specify the default app layout in the in the `App` component's <xref:Microsoft.AspNetCore.Components.Routing.Router> component.</span></span> <span data-ttu-id="e5f27-158">L'esempio seguente di un'app basata su un [ Blazor modello di progetto](xref:blazor/project-structure) imposta il layout predefinito sul `MainLayout` componente.</span><span class="sxs-lookup"><span data-stu-id="e5f27-158">The following example from an app based on a [Blazor project template](xref:blazor/project-structure) sets the default layout to the `MainLayout` component.</span></span>

<span data-ttu-id="e5f27-159">`App.razor`:</span><span class="sxs-lookup"><span data-stu-id="e5f27-159">`App.razor`:</span></span>

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/layouts/App1.razor?highlight=3)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/layouts/App1.razor?highlight=3)]

::: moniker-end

[!INCLUDE[](~/blazor/includes/prefer-exact-matches.md)]

<span data-ttu-id="e5f27-160">Per ulteriori informazioni sul <xref:Microsoft.AspNetCore.Components.Routing.Router> componente, vedere <xref:blazor/fundamentals/routing> .</span><span class="sxs-lookup"><span data-stu-id="e5f27-160">For more information on the <xref:Microsoft.AspNetCore.Components.Routing.Router> component, see <xref:blazor/fundamentals/routing>.</span></span>

<span data-ttu-id="e5f27-161">La specifica del layout come layout predefinito nel `Router` componente è una procedura utile perché è possibile eseguire l'override del layout per ogni singolo componente o per ogni cartella, come descritto nelle sezioni precedenti di questo articolo.</span><span class="sxs-lookup"><span data-stu-id="e5f27-161">Specifying the layout as a default layout in the `Router` component is a useful practice because you can override the layout on a per-component or per-folder basis, as described in the preceding sections of this article.</span></span> <span data-ttu-id="e5f27-162">È consigliabile usare il `Router` componente per impostare il layout predefinito dell'app perché è l'approccio più generale e flessibile per l'uso dei layout.</span><span class="sxs-lookup"><span data-stu-id="e5f27-162">We recommend using the `Router` component to set the app's default layout because it's the most general and flexible approach for using layouts.</span></span>

### <a name="apply-a-layout-to-arbitrary-content-layoutview-component"></a><span data-ttu-id="e5f27-163">Applicare un layout al contenuto arbitrario ( `LayoutView` componente)</span><span class="sxs-lookup"><span data-stu-id="e5f27-163">Apply a layout to arbitrary content (`LayoutView` component)</span></span>

<span data-ttu-id="e5f27-164">Per impostare un layout per il contenuto di un modello arbitrario Razor , specificare il layout con un <xref:Microsoft.AspNetCore.Components.LayoutView> componente.</span><span class="sxs-lookup"><span data-stu-id="e5f27-164">To set a layout for arbitrary Razor template content, specify the layout with a <xref:Microsoft.AspNetCore.Components.LayoutView> component.</span></span> <span data-ttu-id="e5f27-165">È possibile usare un oggetto <xref:Microsoft.AspNetCore.Components.LayoutView> in qualsiasi Razor componente.</span><span class="sxs-lookup"><span data-stu-id="e5f27-165">You can use a <xref:Microsoft.AspNetCore.Components.LayoutView> in any Razor component.</span></span> <span data-ttu-id="e5f27-166">Nell'esempio seguente viene impostato un componente di layout denominato `ErrorLayout` per il `MainLayout` modello del componente <xref:Microsoft.AspNetCore.Components.Routing.Router.NotFound> ( `<NotFound>...</NotFound>` ).</span><span class="sxs-lookup"><span data-stu-id="e5f27-166">The following example sets a layout component named `ErrorLayout` for the `MainLayout` component's <xref:Microsoft.AspNetCore.Components.Routing.Router.NotFound> template (`<NotFound>...</NotFound>`).</span></span>

<span data-ttu-id="e5f27-167">`App.razor`:</span><span class="sxs-lookup"><span data-stu-id="e5f27-167">`App.razor`:</span></span>

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/layouts/App2.razor?name=snippet&highlight=6,9)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/layouts/App2.razor?name=snippet&highlight=6,9)]

::: moniker-end

[!INCLUDE[](~/blazor/includes/prefer-exact-matches.md)]

## <a name="nested-layouts"></a><span data-ttu-id="e5f27-168">Layout annidati</span><span class="sxs-lookup"><span data-stu-id="e5f27-168">Nested layouts</span></span>

<span data-ttu-id="e5f27-169">Un componente può fare riferimento a un layout che a sua volta fa riferimento a un altro layout.</span><span class="sxs-lookup"><span data-stu-id="e5f27-169">A component can reference a layout that in turn references another layout.</span></span> <span data-ttu-id="e5f27-170">I layout annidati, ad esempio, vengono utilizzati per creare una struttura di menu a più livelli.</span><span class="sxs-lookup"><span data-stu-id="e5f27-170">For example, nested layouts are used to create a multi-level menu structures.</span></span>

<span data-ttu-id="e5f27-171">Nell'esempio seguente viene illustrato come utilizzare layout annidati.</span><span class="sxs-lookup"><span data-stu-id="e5f27-171">The following example shows how to use nested layouts.</span></span> <span data-ttu-id="e5f27-172">Il componente `Episodes` illustrato nella sezione [applicare un layout a un componente](#apply-a-layout-to-a-component) è il componente da visualizzare.</span><span class="sxs-lookup"><span data-stu-id="e5f27-172">The `Episodes` component shown in the [Apply a layout to a component](#apply-a-layout-to-a-component) section is the component to display.</span></span> <span data-ttu-id="e5f27-173">Il componente fa riferimento al `DoctorWhoLayout` componente.</span><span class="sxs-lookup"><span data-stu-id="e5f27-173">The component references the `DoctorWhoLayout` component.</span></span>

<span data-ttu-id="e5f27-174">Il `DoctorWhoLayout` componente seguente è una versione modificata dell'esempio illustrato in precedenza in questo articolo.</span><span class="sxs-lookup"><span data-stu-id="e5f27-174">The following `DoctorWhoLayout` component is a modified version of the example shown earlier in this article.</span></span> <span data-ttu-id="e5f27-175">Gli elementi dell'intestazione e del piè di pagina vengono rimossi e il layout fa riferimento a un altro layout `ProductionsLayout` .</span><span class="sxs-lookup"><span data-stu-id="e5f27-175">The header and footer elements are removed, and the layout references another layout, `ProductionsLayout`.</span></span> <span data-ttu-id="e5f27-176">Il `Episodes` rendering del componente viene `@Body` visualizzato in `DoctorWhoLayout` .</span><span class="sxs-lookup"><span data-stu-id="e5f27-176">The `Episodes` component is rendered where `@Body` appears in the `DoctorWhoLayout`.</span></span>

<span data-ttu-id="e5f27-177">`Shared/DoctorWhoLayout.razor`:</span><span class="sxs-lookup"><span data-stu-id="e5f27-177">`Shared/DoctorWhoLayout.razor`:</span></span>

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Shared/layouts/DoctorWhoLayout2.razor?highlight=2,12)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Shared/layouts/DoctorWhoLayout2.razor?highlight=2,12)]

::: moniker-end

<span data-ttu-id="e5f27-178">Il `ProductionsLayout` componente contiene gli elementi di layout di primo livello, in cui ora si trovano gli elementi Header ( `<header>...</header>` ) e footer ( `<footer>...</footer>` ).</span><span class="sxs-lookup"><span data-stu-id="e5f27-178">The `ProductionsLayout` component contains the top-level layout elements, where the header (`<header>...</header>`) and footer (`<footer>...</footer>`) elements now reside.</span></span> <span data-ttu-id="e5f27-179">`DoctorWhoLayout`Viene eseguito il rendering dell'oggetto con il `Episodes` componente `@Body` , dove viene visualizzato.</span><span class="sxs-lookup"><span data-stu-id="e5f27-179">The `DoctorWhoLayout` with the `Episodes` component is rendered where `@Body` appears.</span></span>

<span data-ttu-id="e5f27-180">`Shared/ProductionsLayout.razor`:</span><span class="sxs-lookup"><span data-stu-id="e5f27-180">`Shared/ProductionsLayout.razor`:</span></span>

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Shared/layouts/ProductionsLayout.razor?highlight=13)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Shared/layouts/ProductionsLayout.razor?highlight=13)]

::: moniker-end

<span data-ttu-id="e5f27-181">Il markup HTML sottoposto a rendering seguente viene generato dal layout annidato precedente.</span><span class="sxs-lookup"><span data-stu-id="e5f27-181">The following rendered HTML markup is produced by the preceding nested layout.</span></span> <span data-ttu-id="e5f27-182">Il markup estraneo non viene visualizzato per concentrarsi sul contenuto annidato fornito dai tre componenti interessati:</span><span class="sxs-lookup"><span data-stu-id="e5f27-182">Extraneous markup doesn't appear in order to focus on the nested content provided by the three components involved:</span></span>

* <span data-ttu-id="e5f27-183">Gli elementi Header ( `<header>...</header>` ), barra di spostamento produzione () e piè di pagina `<nav>...</nav>` ( `<footer>...</footer>` ) e il relativo contenuto provengono dal `ProductionsLayout` componente.</span><span class="sxs-lookup"><span data-stu-id="e5f27-183">The header (`<header>...</header>`), production navigation bar (`<nav>...</nav>`), and footer (`<footer>...</footer>`) elements and their content come from the `ProductionsLayout` component.</span></span>
* <span data-ttu-id="e5f27-184">Il componente è il **medico che &trade; punta al database** heading ( `<h1>...</h1>` ), la barra di spostamento degli episodi ( `<nav>...</nav>` ) e l'elemento informazioni sul marchio ( `<div>...</div>` ) `DoctorWhoLayout` .</span><span class="sxs-lookup"><span data-stu-id="e5f27-184">The **Doctor Who&trade; Episode Database** heading (`<h1>...</h1>`), episode navigation bar (`<nav>...</nav>`), and trademark information element (`<div>...</div>`) come from the `DoctorWhoLayout` component.</span></span>
* <span data-ttu-id="e5f27-185">L'intestazione degli **episodi** ( `<h2>...</h2>` ) e l'elenco di puntatori ( `<ul>...</ul>` ) provengono dal `Episodes` componente.</span><span class="sxs-lookup"><span data-stu-id="e5f27-185">The **Episodes** heading (`<h2>...</h2>`) and episode list (`<ul>...</ul>`) come from the `Episodes` component.</span></span>

```html
<body>
    <div id="app">
        <header>
            <h1>Productions</h1>
        </header>

        <nav>
            <a href="master-production-list">Master Production List</a>
            <a href="production-search">Search</a>
            <a href="new-production">Add Production</a>
        </nav>

        <h1>Doctor Who&trade; Episode Database</h1>

        <nav>
            <a href="episode-masterlist">Master Episode List</a>
            <a href="episode-search">Search</a>
            <a href="new-episode">Add Episode</a>
        </nav>

        <h2>Episodes</h2>

        <ul>
            <li>...</li>
            <li>...</li>
            <li>...</li>
        </ul>

        <div>
            Doctor Who is a registered trademark of the BBC. 
            https://www.doctorwho.tv/
        </div>

        <footer>
            Footer of Productions Layout
        </footer>
    </div>
</body>
```

## <a name="share-a-razor-pages-layout-with-integrated-components"></a><span data-ttu-id="e5f27-186">Condividere un Razor layout di pagine con componenti integrati</span><span class="sxs-lookup"><span data-stu-id="e5f27-186">Share a Razor Pages layout with integrated components</span></span>

<span data-ttu-id="e5f27-187">Quando i componenti instradabili sono integrati in un' Razor app pagine, il layout condiviso dell'app può essere usato con i componenti.</span><span class="sxs-lookup"><span data-stu-id="e5f27-187">When routable components are integrated into a Razor Pages app, the app's shared layout can be used with the components.</span></span> <span data-ttu-id="e5f27-188">Per altre informazioni, vedere <xref:blazor/components/prerendering-and-integration>.</span><span class="sxs-lookup"><span data-stu-id="e5f27-188">For more information, see <xref:blazor/components/prerendering-and-integration>.</span></span>

## <a name="additional-resources"></a><span data-ttu-id="e5f27-189">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="e5f27-189">Additional resources</span></span>

* <xref:mvc/views/layout>
