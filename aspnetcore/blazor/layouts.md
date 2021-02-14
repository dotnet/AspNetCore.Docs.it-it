---
title: BlazorLayout ASP.NET Core
author: guardrex
description: Informazioni su come creare componenti di layout riutilizzabili per le Blazor app.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 06/23/2020
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
ms.openlocfilehash: d1f3e2028ca120b5901aca0b24802d872ae52597
ms.sourcegitcommit: 1166b0ff3828418559510c661e8240e5c5717bb7
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/12/2021
ms.locfileid: "100279735"
---
# <a name="aspnet-core-blazor-layouts"></a><span data-ttu-id="dd115-103">BlazorLayout ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="dd115-103">ASP.NET Core Blazor layouts</span></span>

<span data-ttu-id="dd115-104">Alcuni elementi dell'app, ad esempio i menu, i messaggi di copyright e i logo aziendali, fanno in genere parte del layout generale dell'app e vengono usati da ogni componente nell'app.</span><span class="sxs-lookup"><span data-stu-id="dd115-104">Some app elements, such as menus, copyright messages, and company logos, are usually part of app's overall layout and used by every component in the app.</span></span> <span data-ttu-id="dd115-105">La copia del codice di questi elementi in tutti i componenti di un'app non è un approccio efficace.</span><span class="sxs-lookup"><span data-stu-id="dd115-105">Copying the code of these elements into all of the components of an app isn't an efficient approach.</span></span> <span data-ttu-id="dd115-106">Ogni volta che uno degli elementi richiede un aggiornamento, è necessario aggiornare ogni componente.</span><span class="sxs-lookup"><span data-stu-id="dd115-106">Every time one of the elements requires an update, every component must be updated.</span></span> <span data-ttu-id="dd115-107">Tale duplicazione è difficile da gestire e può causare contenuti incoerenti nel tempo.</span><span class="sxs-lookup"><span data-stu-id="dd115-107">Such duplication is difficult to maintain and can lead to inconsistent content over time.</span></span> <span data-ttu-id="dd115-108">I *layout* risolvono questo problema.</span><span class="sxs-lookup"><span data-stu-id="dd115-108">*Layouts* solve this problem.</span></span>

<span data-ttu-id="dd115-109">Tecnicamente, un layout è semplicemente un altro componente.</span><span class="sxs-lookup"><span data-stu-id="dd115-109">Technically, a layout is just another component.</span></span> <span data-ttu-id="dd115-110">Un layout è definito in un Razor modello o nel codice C# e può usare [Data Binding](xref:blazor/components/data-binding), l' [inserimento delle dipendenze](xref:blazor/fundamentals/dependency-injection)e altri scenari di componenti.</span><span class="sxs-lookup"><span data-stu-id="dd115-110">A layout is defined in a Razor template or in C# code and can use [data binding](xref:blazor/components/data-binding), [dependency injection](xref:blazor/fundamentals/dependency-injection), and other component scenarios.</span></span> <span data-ttu-id="dd115-111">I layout si applicano solo ai componenti instradabili con Razor [`@page`](xref:mvc/views/razor#page) direttive.</span><span class="sxs-lookup"><span data-stu-id="dd115-111">Layouts only apply to routable Razor components that have [`@page`](xref:mvc/views/razor#page) directives.</span></span>

<span data-ttu-id="dd115-112">Per convertire un componente in un layout:</span><span class="sxs-lookup"><span data-stu-id="dd115-112">To convert a component into a layout:</span></span>

* <span data-ttu-id="dd115-113">Ereditare il componente da <xref:Microsoft.AspNetCore.Components.LayoutComponentBase> .</span><span class="sxs-lookup"><span data-stu-id="dd115-113">Inherit the component from <xref:Microsoft.AspNetCore.Components.LayoutComponentBase>.</span></span> <span data-ttu-id="dd115-114"><xref:Microsoft.AspNetCore.Components.LayoutComponentBase>Definisce una <xref:Microsoft.AspNetCore.Components.LayoutComponentBase.Body> proprietà per il contenuto di cui è stato eseguito il rendering all'interno del layout.</span><span class="sxs-lookup"><span data-stu-id="dd115-114">The <xref:Microsoft.AspNetCore.Components.LayoutComponentBase> defines a <xref:Microsoft.AspNetCore.Components.LayoutComponentBase.Body> property for the rendered content inside the layout.</span></span>
* <span data-ttu-id="dd115-115">Utilizzare la Razor sintassi `@Body` per specificare il percorso nel markup di layout in cui viene eseguito il rendering del contenuto.</span><span class="sxs-lookup"><span data-stu-id="dd115-115">Use the Razor syntax `@Body` to specify the location in the layout markup where the content is rendered.</span></span>

<span data-ttu-id="dd115-116">Nell'esempio di codice riportato di seguito viene illustrato il Razor modello di un componente di layout `MainLayout.razor` .</span><span class="sxs-lookup"><span data-stu-id="dd115-116">The following code sample shows the Razor template of a layout component, `MainLayout.razor`.</span></span> <span data-ttu-id="dd115-117">Il layout eredita <xref:Microsoft.AspNetCore.Components.LayoutComponentBase> e imposta l'oggetto `@Body` tra la barra di spostamento e il piè di pagina:</span><span class="sxs-lookup"><span data-stu-id="dd115-117">The layout inherits <xref:Microsoft.AspNetCore.Components.LayoutComponentBase> and sets the `@Body` between the navigation bar and the footer:</span></span>

[!code-razor[](layouts/sample_snapshot/3.x/MainLayout.razor)]

## <a name="mainlayout-component"></a><span data-ttu-id="dd115-118">Componente `MainLayout`</span><span class="sxs-lookup"><span data-stu-id="dd115-118">`MainLayout` component</span></span>

<span data-ttu-id="dd115-119">In un'app basata su uno dei Blazor modelli di progetto, il `MainLayout` componente ( `MainLayout.razor` ) si trova nella cartella dell'app `Shared` :</span><span class="sxs-lookup"><span data-stu-id="dd115-119">In an app based on one of the Blazor project templates, the `MainLayout` component (`MainLayout.razor`) is in the app's `Shared` folder:</span></span>

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](./common/samples/5.x/BlazorWebAssemblySample/Shared/MainLayout.razor)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](./common/samples/3.x/BlazorWebAssemblySample/Shared/MainLayout.razor)]

::: moniker-end

## <a name="default-layout"></a><span data-ttu-id="dd115-120">Layout predefinito</span><span class="sxs-lookup"><span data-stu-id="dd115-120">Default layout</span></span>

<span data-ttu-id="dd115-121">Specificare il layout dell'app predefinito nel <xref:Microsoft.AspNetCore.Components.Routing.Router> componente nel file dell'app `App.razor` .</span><span class="sxs-lookup"><span data-stu-id="dd115-121">Specify the default app layout in the <xref:Microsoft.AspNetCore.Components.Routing.Router> component in the app's `App.razor` file.</span></span> <span data-ttu-id="dd115-122">Il <xref:Microsoft.AspNetCore.Components.Routing.Router> componente seguente, fornito dai Blazor modelli predefiniti, imposta il layout predefinito sul `MainLayout` componente:</span><span class="sxs-lookup"><span data-stu-id="dd115-122">The following <xref:Microsoft.AspNetCore.Components.Routing.Router> component, which is provided by the default Blazor templates, sets the default layout to the `MainLayout` component:</span></span>

[!code-razor[](layouts/sample_snapshot/3.x/App1.razor?highlight=3)]

[!INCLUDE[](~/blazor/includes/prefer-exact-matches.md)]

<span data-ttu-id="dd115-123">Per fornire un layout predefinito per il <xref:Microsoft.AspNetCore.Components.Routing.Router.NotFound> contenuto, specificare un oggetto <xref:Microsoft.AspNetCore.Components.LayoutView> per il <xref:Microsoft.AspNetCore.Components.Routing.Router.NotFound> contenuto:</span><span class="sxs-lookup"><span data-stu-id="dd115-123">To supply a default layout for <xref:Microsoft.AspNetCore.Components.Routing.Router.NotFound> content, specify a <xref:Microsoft.AspNetCore.Components.LayoutView> for <xref:Microsoft.AspNetCore.Components.Routing.Router.NotFound> content:</span></span>

[!code-razor[](layouts/sample_snapshot/3.x/App2.razor?highlight=6-9)]

[!INCLUDE[](~/blazor/includes/prefer-exact-matches.md)]

<span data-ttu-id="dd115-124">Per ulteriori informazioni sul <xref:Microsoft.AspNetCore.Components.Routing.Router> componente, vedere <xref:blazor/fundamentals/routing> .</span><span class="sxs-lookup"><span data-stu-id="dd115-124">For more information on the <xref:Microsoft.AspNetCore.Components.Routing.Router> component, see <xref:blazor/fundamentals/routing>.</span></span>

<span data-ttu-id="dd115-125">La specifica del layout come layout predefinito nel router è una procedura utile perché può essere sottoposta a override in base al singolo componente o alla cartella.</span><span class="sxs-lookup"><span data-stu-id="dd115-125">Specifying the layout as a default layout in the router is a useful practice because it can be overridden on a per-component or per-folder basis.</span></span> <span data-ttu-id="dd115-126">Preferisci usare il router per impostare il layout predefinito dell'app perché è la tecnica più generale.</span><span class="sxs-lookup"><span data-stu-id="dd115-126">Prefer using the router to set the app's default layout because it's the most general technique.</span></span>

## <a name="specify-a-layout-in-a-component"></a><span data-ttu-id="dd115-127">Specificare un layout in un componente</span><span class="sxs-lookup"><span data-stu-id="dd115-127">Specify a layout in a component</span></span>

<span data-ttu-id="dd115-128">Utilizzare la [`@layout`](xref:mvc/views/razor#layout) Razor direttiva per applicare un layout a un componente instradabile Razor che dispone anche di una [`@page`](xref:mvc/views/razor#page) direttiva.</span><span class="sxs-lookup"><span data-stu-id="dd115-128">Use the [`@layout`](xref:mvc/views/razor#layout) Razor directive to apply a layout to a routable Razor component that also has an [`@page`](xref:mvc/views/razor#page) directive.</span></span> <span data-ttu-id="dd115-129">Il compilatore esegue la conversione `@layout` in un oggetto <xref:Microsoft.AspNetCore.Components.LayoutAttribute> , che viene applicato alla classe Component.</span><span class="sxs-lookup"><span data-stu-id="dd115-129">The compiler converts `@layout` into a <xref:Microsoft.AspNetCore.Components.LayoutAttribute>, which is applied to the component class.</span></span>

<span data-ttu-id="dd115-130">Il contenuto del componente seguente `MasterList` viene inserito nella `MasterLayout` posizione di `@Body` :</span><span class="sxs-lookup"><span data-stu-id="dd115-130">The content of the following `MasterList` component is inserted into the `MasterLayout` at the position of `@Body`:</span></span>

[!code-razor[](layouts/sample_snapshot/3.x/MasterList.razor?highlight=1)]

<span data-ttu-id="dd115-131">La specifica del layout direttamente in un componente sostituisce un set di *layout predefinito* nel router o una `@layout` direttiva importata da `_Imports.razor` .</span><span class="sxs-lookup"><span data-stu-id="dd115-131">Specifying the layout directly in a component overrides a *default layout* set in the router or an `@layout` directive imported from `_Imports.razor`.</span></span>

## <a name="centralized-layout-selection"></a><span data-ttu-id="dd115-132">Selezione layout centralizzata</span><span class="sxs-lookup"><span data-stu-id="dd115-132">Centralized layout selection</span></span>

<span data-ttu-id="dd115-133">Ogni cartella di un'app può contenere facoltativamente un file modello denominato `_Imports.razor` .</span><span class="sxs-lookup"><span data-stu-id="dd115-133">Every folder of an app can optionally contain a template file named `_Imports.razor`.</span></span> <span data-ttu-id="dd115-134">Il compilatore include le direttive specificate nel file Imports in tutti i Razor modelli nella stessa cartella e in modo ricorsivo in tutte le relative sottocartelle.</span><span class="sxs-lookup"><span data-stu-id="dd115-134">The compiler includes the directives specified in the imports file in all of the Razor templates in the same folder and recursively in all of its subfolders.</span></span> <span data-ttu-id="dd115-135">Pertanto, un `_Imports.razor` file contenente `@layout MyCoolLayout` garantisce che tutti i componenti di una cartella usino `MyCoolLayout` .</span><span class="sxs-lookup"><span data-stu-id="dd115-135">Therefore, an `_Imports.razor` file containing `@layout MyCoolLayout` ensures that all of the components in a folder use `MyCoolLayout`.</span></span> <span data-ttu-id="dd115-136">Non è necessario aggiungere ripetutamente `@layout MyCoolLayout` a tutti i file nella `.razor` cartella e nelle sottocartelle.</span><span class="sxs-lookup"><span data-stu-id="dd115-136">There's no need to repeatedly add `@layout MyCoolLayout` to all of the `.razor` files within the folder and subfolders.</span></span> <span data-ttu-id="dd115-137">`@using` le direttive vengono applicate anche ai componenti nello stesso modo.</span><span class="sxs-lookup"><span data-stu-id="dd115-137">`@using` directives are also applied to components in the same way.</span></span>

<span data-ttu-id="dd115-138">Le `_Imports.razor` importazioni di file seguenti:</span><span class="sxs-lookup"><span data-stu-id="dd115-138">The following `_Imports.razor` file imports:</span></span>

* <span data-ttu-id="dd115-139">`MyCoolLayout`.</span><span class="sxs-lookup"><span data-stu-id="dd115-139">`MyCoolLayout`.</span></span>
* <span data-ttu-id="dd115-140">Tutti i Razor componenti nella stessa cartella e in tutte le sottocartelle.</span><span class="sxs-lookup"><span data-stu-id="dd115-140">All Razor components in the same folder and any subfolders.</span></span>
* <span data-ttu-id="dd115-141">Spazio dei nomi `BlazorApp1.Data`.</span><span class="sxs-lookup"><span data-stu-id="dd115-141">The `BlazorApp1.Data` namespace.</span></span>
 
[!code-razor[](layouts/sample_snapshot/3.x/_Imports.razor)]

<span data-ttu-id="dd115-142">Il `_Imports.razor` file è simile al [file _ViewImports. cshtml per le Razor visualizzazioni e le pagine,](xref:mvc/views/layout#importing-shared-directives) ma viene applicato in modo specifico ai Razor file del componente.</span><span class="sxs-lookup"><span data-stu-id="dd115-142">The `_Imports.razor` file is similar to the [_ViewImports.cshtml file for Razor views and pages](xref:mvc/views/layout#importing-shared-directives) but applied specifically to Razor component files.</span></span>

<span data-ttu-id="dd115-143">La specifica di un layout in `_Imports.razor` sostituisce un layout specificato come *layout predefinito* del router.</span><span class="sxs-lookup"><span data-stu-id="dd115-143">Specifying a layout in `_Imports.razor` overrides a layout specified as the router's *default layout*.</span></span>

> [!WARNING]
> <span data-ttu-id="dd115-144">**Non** aggiungere una Razor `@layout` direttiva al `_Imports.razor` file radice, il che comporta un ciclo infinito di layout nell'app.</span><span class="sxs-lookup"><span data-stu-id="dd115-144">Do **not** add a Razor `@layout` directive to the root `_Imports.razor` file, which results in an infinite loop of layouts in the app.</span></span> <span data-ttu-id="dd115-145">Per controllare il layout predefinito dell'app, specificare il layout nel `Router` componente.</span><span class="sxs-lookup"><span data-stu-id="dd115-145">To control the default app layout, specify the layout in the `Router` component.</span></span> <span data-ttu-id="dd115-146">Per ulteriori informazioni, vedere la sezione [layout predefinito](#default-layout) .</span><span class="sxs-lookup"><span data-stu-id="dd115-146">For more information, see the [Default layout](#default-layout) section.</span></span>

> [!NOTE]
> <span data-ttu-id="dd115-147">La [`@layout`](xref:mvc/views/razor#layout) Razor direttiva applica solo un layout ai componenti instradabili Razor con le [`@page`](xref:mvc/views/razor#page) direttive.</span><span class="sxs-lookup"><span data-stu-id="dd115-147">The [`@layout`](xref:mvc/views/razor#layout) Razor directive only applies a layout to routable Razor components with [`@page`](xref:mvc/views/razor#page) directives.</span></span>

## <a name="nested-layouts"></a><span data-ttu-id="dd115-148">Layout annidati</span><span class="sxs-lookup"><span data-stu-id="dd115-148">Nested layouts</span></span>

<span data-ttu-id="dd115-149">Le app possono essere costituite da layout annidati.</span><span class="sxs-lookup"><span data-stu-id="dd115-149">Apps can consist of nested layouts.</span></span> <span data-ttu-id="dd115-150">Un componente può fare riferimento a un layout che a sua volta fa riferimento a un altro layout.</span><span class="sxs-lookup"><span data-stu-id="dd115-150">A component can reference a layout which in turn references another layout.</span></span> <span data-ttu-id="dd115-151">Ad esempio, i layout di annidamento vengono usati per creare una struttura di menu a più livelli.</span><span class="sxs-lookup"><span data-stu-id="dd115-151">For example, nesting layouts are used to create a multi-level menu structure.</span></span>

<span data-ttu-id="dd115-152">Nell'esempio seguente viene illustrato come utilizzare layout annidati.</span><span class="sxs-lookup"><span data-stu-id="dd115-152">The following example shows how to use nested layouts.</span></span> <span data-ttu-id="dd115-153">Il `EpisodesComponent.razor` file è il componente da visualizzare.</span><span class="sxs-lookup"><span data-stu-id="dd115-153">The `EpisodesComponent.razor` file is the component to display.</span></span> <span data-ttu-id="dd115-154">Il componente fa riferimento a `MasterListLayout` :</span><span class="sxs-lookup"><span data-stu-id="dd115-154">The component references the `MasterListLayout`:</span></span>

[!code-razor[](layouts/sample_snapshot/3.x/EpisodesComponent.razor?highlight=1)]

<span data-ttu-id="dd115-155">Il `MasterListLayout.razor` file fornisce il `MasterListLayout` .</span><span class="sxs-lookup"><span data-stu-id="dd115-155">The `MasterListLayout.razor` file provides the `MasterListLayout`.</span></span> <span data-ttu-id="dd115-156">Il layout fa riferimento a un altro layout, `MasterLayout` , in cui viene eseguito il rendering.</span><span class="sxs-lookup"><span data-stu-id="dd115-156">The layout references another layout, `MasterLayout`, where it's rendered.</span></span> <span data-ttu-id="dd115-157">`EpisodesComponent` viene sottoposto a rendering quando `@Body` viene visualizzato:</span><span class="sxs-lookup"><span data-stu-id="dd115-157">`EpisodesComponent` is rendered where `@Body` appears:</span></span>

[!code-razor[](layouts/sample_snapshot/3.x/MasterListLayout.razor?highlight=1,9)]

<span data-ttu-id="dd115-158">Infine, `MasterLayout` in `MasterLayout.razor` contiene gli elementi di layout di primo livello, ad esempio l'intestazione, il menu principale e il piè di pagina.</span><span class="sxs-lookup"><span data-stu-id="dd115-158">Finally, `MasterLayout` in `MasterLayout.razor` contains the top-level layout elements, such as the header, main menu, and footer.</span></span> <span data-ttu-id="dd115-159">`MasterListLayout` con `EpisodesComponent` viene eseguito il rendering di quando viene `@Body` visualizzato:</span><span class="sxs-lookup"><span data-stu-id="dd115-159">`MasterListLayout` with the `EpisodesComponent` is rendered where `@Body` appears:</span></span>

[!code-razor[](layouts/sample_snapshot/3.x/MasterLayout.razor?highlight=6)]

## <a name="share-a-razor-pages-layout-with-integrated-components"></a><span data-ttu-id="dd115-160">Condividere un Razor layout di pagine con componenti integrati</span><span class="sxs-lookup"><span data-stu-id="dd115-160">Share a Razor Pages layout with integrated components</span></span>

<span data-ttu-id="dd115-161">Quando i componenti instradabili sono integrati in un' Razor app pagine, il layout condiviso dell'app può essere usato con i componenti.</span><span class="sxs-lookup"><span data-stu-id="dd115-161">When routable components are integrated into a Razor Pages app, the app's shared layout can be used with the components.</span></span> <span data-ttu-id="dd115-162">Per altre informazioni, vedere <xref:blazor/components/prerendering-and-integration>.</span><span class="sxs-lookup"><span data-stu-id="dd115-162">For more information, see <xref:blazor/components/prerendering-and-integration>.</span></span>

## <a name="additional-resources"></a><span data-ttu-id="dd115-163">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="dd115-163">Additional resources</span></span>

* <xref:mvc/views/layout>
