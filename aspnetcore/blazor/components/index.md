---
title: Creazione e utilizzo di Razor componenti ASP.NET Core
author: guardrex
description: Informazioni su come creare e usare Razor i componenti, tra cui la modalità di associazione ai dati, la gestione degli eventi e la gestione dei cicli di vita dei componenti.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 11/25/2020
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
uid: blazor/components/index
ms.openlocfilehash: 111512916cb7f0a4fc1f17648e2f9c69e366dff3
ms.sourcegitcommit: 04ad9cd26fcaa8bd11e261d3661f375f5f343cdc
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/10/2021
ms.locfileid: "100107051"
---
# <a name="create-and-use-aspnet-core-razor-components"></a><span data-ttu-id="6d7a2-103">Creazione e utilizzo di Razor componenti ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="6d7a2-103">Create and use ASP.NET Core Razor components</span></span>

<span data-ttu-id="6d7a2-104">Di [Luke Latham](https://github.com/guardrex), [Daniel Roth](https://github.com/danroth27), [Scott Addie](https://github.com/scottaddie)e [Tobias Bartsch](https://www.aveo-solutions.com/)</span><span class="sxs-lookup"><span data-stu-id="6d7a2-104">By [Luke Latham](https://github.com/guardrex), [Daniel Roth](https://github.com/danroth27), [Scott Addie](https://github.com/scottaddie), and [Tobias Bartsch](https://www.aveo-solutions.com/)</span></span>

<span data-ttu-id="6d7a2-105">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/blazor/common/samples/) ([procedura per il download](xref:index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="6d7a2-105">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/blazor/common/samples/) ([how to download](xref:index#how-to-download-a-sample))</span></span>

<span data-ttu-id="6d7a2-106">Blazor le app vengono compilate usando i *componenti*.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-106">Blazor apps are built using *components*.</span></span> <span data-ttu-id="6d7a2-107">Un componente è un blocco di interfaccia utente (UI) autonomo, ad esempio una pagina, una finestra di dialogo o un form.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-107">A component is a self-contained chunk of user interface (UI), such as a page, dialog, or form.</span></span> <span data-ttu-id="6d7a2-108">Un componente include il markup HTML e la logica di elaborazione necessaria per inserire i dati o rispondere agli eventi dell'interfaccia utente.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-108">A component includes HTML markup and the processing logic required to inject data or respond to UI events.</span></span> <span data-ttu-id="6d7a2-109">I componenti sono flessibili e leggeri.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-109">Components are flexible and lightweight.</span></span> <span data-ttu-id="6d7a2-110">Possono essere annidati, riutilizzati e condivisi tra i progetti.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-110">They can be nested, reused, and shared among projects.</span></span>

## <a name="component-classes"></a><span data-ttu-id="6d7a2-111">Classi di componenti</span><span class="sxs-lookup"><span data-stu-id="6d7a2-111">Component classes</span></span>

<span data-ttu-id="6d7a2-112">I componenti sono implementati in [Razor](xref:mvc/views/razor) file componente ( `.razor` ) utilizzando una combinazione di markup C# e HTML.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-112">Components are implemented in [Razor](xref:mvc/views/razor) component files (`.razor`) using a combination of C# and HTML markup.</span></span> <span data-ttu-id="6d7a2-113">Un componente in Blazor viene definito formalmente come *Razor componente*.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-113">A component in Blazor is formally referred to as a *Razor component*.</span></span>

### <a name="razor-syntax"></a><span data-ttu-id="6d7a2-114">Sintassi Razor</span><span class="sxs-lookup"><span data-stu-id="6d7a2-114">Razor syntax</span></span>

<span data-ttu-id="6d7a2-115">Razor i componenti nelle Blazor app utilizzano ampiamente la Razor sintassi.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-115">Razor components in Blazor apps extensively use Razor syntax.</span></span> <span data-ttu-id="6d7a2-116">Se non si ha familiarità con il Razor linguaggio di markup, è consigliabile leggere il [ Razor riferimento alla sintassi per ASP.NET Core prima di](xref:mvc/views/razor) procedere.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-116">If you aren't familiar with the Razor markup language, we recommend reading [Razor syntax reference for ASP.NET Core](xref:mvc/views/razor) before proceeding.</span></span>

<span data-ttu-id="6d7a2-117">Quando si accede al contenuto sulla Razor sintassi, prestare particolare attenzione alle sezioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="6d7a2-117">When accessing the content on Razor syntax, pay special attention to the following sections:</span></span>

* <span data-ttu-id="6d7a2-118">[Direttive](xref:mvc/views/razor#directives): `@` -parole chiave riservate con prefisso che in genere modificano il modo in cui il markup del componente viene analizzato o funzione.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-118">[Directives](xref:mvc/views/razor#directives): `@`-prefixed reserved keywords that typically change the way component markup is parsed or function.</span></span>
* <span data-ttu-id="6d7a2-119">[Attributi di direttiva](xref:mvc/views/razor#directive-attributes): `@` parole chiave riservate con prefisso che in genere modificano il modo in cui gli elementi componente vengono analizzati o funzionano.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-119">[Directive attributes](xref:mvc/views/razor#directive-attributes): `@`-prefixed reserved keywords that typically change the way component elements are parsed or function.</span></span>

### <a name="names"></a><span data-ttu-id="6d7a2-120">Nomi</span><span class="sxs-lookup"><span data-stu-id="6d7a2-120">Names</span></span>

<span data-ttu-id="6d7a2-121">Il nome di un componente deve iniziare con un carattere maiuscolo.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-121">A component's name must start with an uppercase character.</span></span> <span data-ttu-id="6d7a2-122">Ad esempio, `MyCoolComponent.razor` è valido e `myCoolComponent.razor` non è valido.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-122">For example, `MyCoolComponent.razor` is valid, and `myCoolComponent.razor` is invalid.</span></span>

### <a name="routing"></a><span data-ttu-id="6d7a2-123">Routing</span><span class="sxs-lookup"><span data-stu-id="6d7a2-123">Routing</span></span>

<span data-ttu-id="6d7a2-124">Il routing in Blazor viene effettuato fornendo un modello di route a ogni componente accessibile nell'app.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-124">Routing in Blazor is achieved by providing a route template to each accessible component in the app.</span></span> <span data-ttu-id="6d7a2-125">Quando Razor viene compilato un file con una [`@page`][9] direttiva, alla classe generata viene assegnato un oggetto che <xref:Microsoft.AspNetCore.Mvc.RouteAttribute> specifica il modello di route.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-125">When a Razor file with an [`@page`][9] directive is compiled, the generated class is given a <xref:Microsoft.AspNetCore.Mvc.RouteAttribute> specifying the route template.</span></span> <span data-ttu-id="6d7a2-126">In fase di esecuzione, il router cerca le classi di componenti con un oggetto <xref:Microsoft.AspNetCore.Mvc.RouteAttribute> ed esegue il rendering di qualsiasi componente con un modello di route corrispondente all'URL richiesto.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-126">At runtime, the router looks for component classes with a <xref:Microsoft.AspNetCore.Mvc.RouteAttribute> and renders whichever component has a route template that matches the requested URL.</span></span> <span data-ttu-id="6d7a2-127">Per altre informazioni, vedere <xref:blazor/fundamentals/routing>.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-127">For more information, see <xref:blazor/fundamentals/routing>.</span></span>

```razor
@page "/ParentComponent"

...
```

### <a name="markup"></a><span data-ttu-id="6d7a2-128">markup</span><span class="sxs-lookup"><span data-stu-id="6d7a2-128">Markup</span></span>

<span data-ttu-id="6d7a2-129">L'interfaccia utente per un componente viene definita utilizzando HTML.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-129">The UI for a component is defined using HTML.</span></span> <span data-ttu-id="6d7a2-130">La logica di rendering dinamica (ad esempio, cicli, condizionali, espressioni) viene aggiunta usando una sintassi C# incorporata denominata *Razor* .</span><span class="sxs-lookup"><span data-stu-id="6d7a2-130">Dynamic rendering logic (for example, loops, conditionals, expressions) is added using an embedded C# syntax called *Razor*.</span></span> <span data-ttu-id="6d7a2-131">Quando viene compilata un'app, il markup HTML e la logica di rendering C# vengono convertiti in una classe Component.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-131">When an app is compiled, the HTML markup and C# rendering logic are converted into a component class.</span></span> <span data-ttu-id="6d7a2-132">Il nome della classe generata corrisponde al nome del file.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-132">The name of the generated class matches the name of the file.</span></span>

<span data-ttu-id="6d7a2-133">I membri della classe Component sono definiti in un [`@code`][1] blocco.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-133">Members of the component class are defined in an [`@code`][1] block.</span></span> <span data-ttu-id="6d7a2-134">Nel [`@code`][1] blocco, lo stato del componente (proprietà, campi) viene specificato con i metodi per la gestione degli eventi o per la definizione di altre logiche dei componenti.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-134">In the [`@code`][1] block, component state (properties, fields) is specified with methods for event handling or for defining other component logic.</span></span> <span data-ttu-id="6d7a2-135">È consentito più di un [`@code`][1] blocco.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-135">More than one [`@code`][1] block is permissible.</span></span>

<span data-ttu-id="6d7a2-136">I membri dei componenti possono essere usati come parte della logica di rendering del componente usando espressioni C# che iniziano con `@` .</span><span class="sxs-lookup"><span data-stu-id="6d7a2-136">Component members can be used as part of the component's rendering logic using C# expressions that start with `@`.</span></span> <span data-ttu-id="6d7a2-137">Ad esempio, viene eseguito il rendering di un campo C# anteponendo `@` il nome del campo.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-137">For example, a C# field is rendered by prefixing `@` to the field name.</span></span> <span data-ttu-id="6d7a2-138">Nell'esempio seguente viene valutato ed eseguito il rendering:</span><span class="sxs-lookup"><span data-stu-id="6d7a2-138">The following example evaluates and renders:</span></span>

* <span data-ttu-id="6d7a2-139">`headingFontStyle` al valore della proprietà CSS per `font-style` .</span><span class="sxs-lookup"><span data-stu-id="6d7a2-139">`headingFontStyle` to the CSS property value for `font-style`.</span></span>
* <span data-ttu-id="6d7a2-140">`headingText` al contenuto dell' `<h1>` elemento.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-140">`headingText` to the content of the `<h1>` element.</span></span>

```razor
<h1 style="font-style:@headingFontStyle">@headingText</h1>

@code {
    private string headingFontStyle = "italic";
    private string headingText = "Put on your new Blazor!";
}
```

<span data-ttu-id="6d7a2-141">Una volta eseguito il rendering iniziale del componente, il componente rigenera l'albero di rendering in risposta agli eventi.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-141">After the component is initially rendered, the component regenerates its render tree in response to events.</span></span> <span data-ttu-id="6d7a2-142">Blazor Confronta quindi il nuovo albero di rendering con quello precedente e applica le modifiche apportate al Document Object Model (DOM) del browser.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-142">Blazor then compares the new render tree against the previous one and applies any modifications to the browser's Document Object Model (DOM).</span></span> <span data-ttu-id="6d7a2-143">Ulteriori dettagli sono disponibili in <xref:blazor/components/rendering> .</span><span class="sxs-lookup"><span data-stu-id="6d7a2-143">Additional detail is provided in <xref:blazor/components/rendering>.</span></span>

<span data-ttu-id="6d7a2-144">I componenti sono classi C# ordinarie e possono essere inseriti in qualsiasi punto all'interno di un progetto.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-144">Components are ordinary C# classes and can be placed anywhere within a project.</span></span> <span data-ttu-id="6d7a2-145">I componenti che producono pagine Web in genere risiedono nella `Pages` cartella.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-145">Components that produce webpages usually reside in the `Pages` folder.</span></span> <span data-ttu-id="6d7a2-146">I componenti non di pagina vengono spesso inseriti nella `Shared` cartella o in una cartella personalizzata aggiunta al progetto.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-146">Non-page components are frequently placed in the `Shared` folder or a custom folder added to the project.</span></span>

### <a name="namespaces"></a><span data-ttu-id="6d7a2-147">Spazi dei nomi</span><span class="sxs-lookup"><span data-stu-id="6d7a2-147">Namespaces</span></span>

<span data-ttu-id="6d7a2-148">Lo spazio dei nomi di un componente viene in genere derivato dallo spazio dei nomi radice dell'app e dal percorso (cartella) del componente all'interno dell'app.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-148">Typically, a component's namespace is derived from the app's root namespace and the component's location (folder) within the app.</span></span> <span data-ttu-id="6d7a2-149">Se lo spazio dei nomi radice dell'app è `BlazorSample` e il componente si trova `Counter` nella `Pages` cartella:</span><span class="sxs-lookup"><span data-stu-id="6d7a2-149">If the app's root namespace is `BlazorSample` and the `Counter` component resides in the `Pages` folder:</span></span>

* <span data-ttu-id="6d7a2-150">Lo `Counter` spazio dei nomi del componente è `BlazorSample.Pages` .</span><span class="sxs-lookup"><span data-stu-id="6d7a2-150">The `Counter` component's namespace is `BlazorSample.Pages`.</span></span>
* <span data-ttu-id="6d7a2-151">Il nome completo del tipo del componente è `BlazorSample.Pages.Counter` .</span><span class="sxs-lookup"><span data-stu-id="6d7a2-151">The fully qualified type name of the component is `BlazorSample.Pages.Counter`.</span></span>

<span data-ttu-id="6d7a2-152">Per le cartelle personalizzate che contengono componenti, aggiungere una [`@using`][2] direttiva al componente padre o al file dell'app `_Imports.razor` .</span><span class="sxs-lookup"><span data-stu-id="6d7a2-152">For custom folders that hold components, add a [`@using`][2] directive to the parent component or to the app's `_Imports.razor` file.</span></span> <span data-ttu-id="6d7a2-153">L'esempio seguente rende disponibili i componenti nella `Components` cartella:</span><span class="sxs-lookup"><span data-stu-id="6d7a2-153">The following example makes components in the `Components` folder available:</span></span>

```razor
@using BlazorSample.Components
```

<span data-ttu-id="6d7a2-154">È anche possibile fare riferimento ai componenti usando i relativi nomi completi, che non richiedono la [`@using`][2] direttiva:</span><span class="sxs-lookup"><span data-stu-id="6d7a2-154">Components can also be referenced using their fully qualified names, which doesn't require the [`@using`][2] directive:</span></span>

```razor
<BlazorSample.Components.MyComponent />
```

<span data-ttu-id="6d7a2-155">Lo spazio dei nomi di un componente creato con Razor è basato su (in ordine di priorità):</span><span class="sxs-lookup"><span data-stu-id="6d7a2-155">The namespace of a component authored with Razor is based on (in priority order):</span></span>

* <span data-ttu-id="6d7a2-156">[`@namespace`][8] designazione nel Razor markup file ( `.razor` ) `@namespace BlazorSample.MyNamespace` .</span><span class="sxs-lookup"><span data-stu-id="6d7a2-156">[`@namespace`][8] designation in Razor file (`.razor`) markup (`@namespace BlazorSample.MyNamespace`).</span></span>
* <span data-ttu-id="6d7a2-157">Il progetto `RootNamespace` nel file di progetto ( `<RootNamespace>BlazorSample</RootNamespace>` ).</span><span class="sxs-lookup"><span data-stu-id="6d7a2-157">The project's `RootNamespace` in the project file (`<RootNamespace>BlazorSample</RootNamespace>`).</span></span>
* <span data-ttu-id="6d7a2-158">Il nome del progetto, tratto dal nome file del file di progetto ( `.csproj` ) e il percorso dalla radice del progetto al componente.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-158">The project name, taken from the project file's file name (`.csproj`), and the path from the project root to the component.</span></span> <span data-ttu-id="6d7a2-159">Il Framework, ad esempio, risolve `{PROJECT ROOT}/Pages/Index.razor` ( `BlazorSample.csproj` ) nello spazio dei nomi `BlazorSample.Pages` .</span><span class="sxs-lookup"><span data-stu-id="6d7a2-159">For example, the framework resolves `{PROJECT ROOT}/Pages/Index.razor` (`BlazorSample.csproj`) to the namespace `BlazorSample.Pages`.</span></span> <span data-ttu-id="6d7a2-160">I componenti seguono le regole di associazione dei nomi in C#.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-160">Components follow C# name binding rules.</span></span> <span data-ttu-id="6d7a2-161">Per il `Index` componente in questo esempio, i componenti nell'ambito sono tutti componenti:</span><span class="sxs-lookup"><span data-stu-id="6d7a2-161">For the `Index` component in this example, the components in scope are all of the components:</span></span>
  * <span data-ttu-id="6d7a2-162">Nella stessa cartella `Pages` .</span><span class="sxs-lookup"><span data-stu-id="6d7a2-162">In the same folder, `Pages`.</span></span>
  * <span data-ttu-id="6d7a2-163">Componenti nella radice del progetto che non specificano in modo esplicito uno spazio dei nomi diverso.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-163">The components in the project's root that don't explicitly specify a different namespace.</span></span>

> [!NOTE]
> <span data-ttu-id="6d7a2-164">La `global::` qualificazione non è supportata.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-164">The `global::` qualification isn't supported.</span></span>
>
> <span data-ttu-id="6d7a2-165">L'importazione di componenti con [`using`](/dotnet/csharp/language-reference/keywords/using-statement) istruzioni con alias (ad esempio, `@using Foo = Bar` ) non è supportata.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-165">Importing components with aliased [`using`](/dotnet/csharp/language-reference/keywords/using-statement) statements (for example, `@using Foo = Bar`) isn't supported.</span></span>
>
> <span data-ttu-id="6d7a2-166">I nomi parzialmente qualificati non sono supportati.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-166">Partially qualified names aren't supported.</span></span> <span data-ttu-id="6d7a2-167">Ad esempio, `@using BlazorSample` l'aggiunta e il riferimento al `NavMenu` componente ( `NavMenu.razor` ) con `<Shared.NavMenu></Shared.NavMenu>` non è supportato.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-167">For example, adding `@using BlazorSample` and referencing the `NavMenu` component (`NavMenu.razor`) with `<Shared.NavMenu></Shared.NavMenu>` isn't supported.</span></span>

### <a name="partial-class-support"></a><span data-ttu-id="6d7a2-168">Supporto di classi parziali</span><span class="sxs-lookup"><span data-stu-id="6d7a2-168">Partial class support</span></span>

<span data-ttu-id="6d7a2-169">Razor i componenti vengono generati come classi parziali.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-169">Razor components are generated as partial classes.</span></span> <span data-ttu-id="6d7a2-170">Razor i componenti vengono creati utilizzando uno degli approcci seguenti:</span><span class="sxs-lookup"><span data-stu-id="6d7a2-170">Razor components are authored using either of the following approaches:</span></span>

* <span data-ttu-id="6d7a2-171">Il codice C# è definito in un [`@code`][1] blocco con markup HTML e Razor codice in un unico file.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-171">C# code is defined in an [`@code`][1] block with HTML markup and Razor code in a single file.</span></span> <span data-ttu-id="6d7a2-172">Blazor i modelli definiscono i Razor componenti usando questo approccio.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-172">Blazor templates define their Razor components using this approach.</span></span>
* <span data-ttu-id="6d7a2-173">Il codice C# viene inserito in un file code-behind definito come classe parziale.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-173">C# code is placed in a code-behind file defined as a partial class.</span></span>

<span data-ttu-id="6d7a2-174">Nell'esempio seguente viene illustrato il `Counter` componente predefinito con un [`@code`][1] blocco in un'app generata da un Blazor modello.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-174">The following example shows the default `Counter` component with an [`@code`][1] block in an app generated from a Blazor template.</span></span> <span data-ttu-id="6d7a2-175">Il markup HTML, Razor il codice e il codice C# si trovano nello stesso file:</span><span class="sxs-lookup"><span data-stu-id="6d7a2-175">HTML markup, Razor code, and C# code are in the same file:</span></span>

<span data-ttu-id="6d7a2-176">`Pages/Counter.razor`:</span><span class="sxs-lookup"><span data-stu-id="6d7a2-176">`Pages/Counter.razor`:</span></span>

```razor
@page "/counter"

<h1>Counter</h1>

<p>Current count: @currentCount</p>

<button class="btn btn-primary" @onclick="IncrementCount">Click me</button>

@code {
    private int currentCount = 0;

    void IncrementCount()
    {
        currentCount++;
    }
}
```

<span data-ttu-id="6d7a2-177">Il `Counter` componente può essere creato anche usando un file code-behind con una classe parziale:</span><span class="sxs-lookup"><span data-stu-id="6d7a2-177">The `Counter` component can also be created using a code-behind file with a partial class:</span></span>

<span data-ttu-id="6d7a2-178">`Pages/Counter.razor`:</span><span class="sxs-lookup"><span data-stu-id="6d7a2-178">`Pages/Counter.razor`:</span></span>

```razor
@page "/counter"

<h1>Counter</h1>

<p>Current count: @currentCount</p>

<button class="btn btn-primary" @onclick="IncrementCount">Click me</button>
```

<span data-ttu-id="6d7a2-179">`Counter.razor.cs`:</span><span class="sxs-lookup"><span data-stu-id="6d7a2-179">`Counter.razor.cs`:</span></span>

```csharp
namespace BlazorSample.Pages
{
    public partial class Counter
    {
        private int currentCount = 0;

        void IncrementCount()
        {
            currentCount++;
        }
    }
}
```

<span data-ttu-id="6d7a2-180">Aggiungere gli spazi dei nomi necessari al file di classe parziale in base alle esigenze.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-180">Add any required namespaces to the partial class file as needed.</span></span> <span data-ttu-id="6d7a2-181">Gli spazi dei nomi tipici usati dai Razor componenti includono:</span><span class="sxs-lookup"><span data-stu-id="6d7a2-181">Typical namespaces used by Razor components include:</span></span>

```csharp
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Components;
using Microsoft.AspNetCore.Components.Authorization;
using Microsoft.AspNetCore.Components.Forms;
using Microsoft.AspNetCore.Components.Routing;
using Microsoft.AspNetCore.Components.Web;
```

> [!IMPORTANT]
> <span data-ttu-id="6d7a2-182">[`@using`][2] le direttive nel `_Imports.razor` file vengono applicate solo ai Razor file ( `.razor` ) e non ai file C# ( `.cs` ).</span><span class="sxs-lookup"><span data-stu-id="6d7a2-182">[`@using`][2] directives in the `_Imports.razor` file are only applied to Razor files (`.razor`), not C# files (`.cs`).</span></span>

### <a name="specify-a-base-class"></a><span data-ttu-id="6d7a2-183">Specificare una classe di base</span><span class="sxs-lookup"><span data-stu-id="6d7a2-183">Specify a base class</span></span>

<span data-ttu-id="6d7a2-184">La [`@inherits`][6] direttiva può essere usata per specificare una classe di base per un componente.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-184">The [`@inherits`][6] directive can be used to specify a base class for a component.</span></span> <span data-ttu-id="6d7a2-185">Nell'esempio seguente viene illustrato come un componente può ereditare una classe base, `BlazorRocksBase` , per fornire le proprietà e i metodi del componente.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-185">The following example shows how a component can inherit a base class, `BlazorRocksBase`, to provide the component's properties and methods.</span></span> <span data-ttu-id="6d7a2-186">La classe base deve derivare da <xref:Microsoft.AspNetCore.Components.ComponentBase> .</span><span class="sxs-lookup"><span data-stu-id="6d7a2-186">The base class should derive from <xref:Microsoft.AspNetCore.Components.ComponentBase>.</span></span>

<span data-ttu-id="6d7a2-187">`Pages/BlazorRocks.razor`:</span><span class="sxs-lookup"><span data-stu-id="6d7a2-187">`Pages/BlazorRocks.razor`:</span></span>

```razor
@page "/BlazorRocks"
@inherits BlazorRocksBase

<h1>@BlazorRocksText</h1>
```

<span data-ttu-id="6d7a2-188">`BlazorRocksBase.cs`:</span><span class="sxs-lookup"><span data-stu-id="6d7a2-188">`BlazorRocksBase.cs`:</span></span>

```csharp
using Microsoft.AspNetCore.Components;

namespace BlazorSample
{
    public class BlazorRocksBase : ComponentBase
    {
        public string BlazorRocksText { get; set; } = 
            "Blazor rocks the browser!";
    }
}
```

### <a name="use-components"></a><span data-ttu-id="6d7a2-189">Usare i componenti</span><span class="sxs-lookup"><span data-stu-id="6d7a2-189">Use components</span></span>

<span data-ttu-id="6d7a2-190">I componenti possono includere altri componenti dichiarando questi ultimi usando la sintassi degli elementi HTML.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-190">Components can include other components by declaring them using HTML element syntax.</span></span> <span data-ttu-id="6d7a2-191">Il markup per l'uso di un componente è simile a un tag HTML, in cui il nome del tag è il tipo di componente.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-191">The markup for using a component looks like an HTML tag where the name of the tag is the component type.</span></span>

<span data-ttu-id="6d7a2-192">Il markup seguente in `Pages/Index.razor` esegue il rendering di un' `HeadingComponent` istanza di:</span><span class="sxs-lookup"><span data-stu-id="6d7a2-192">The following markup in `Pages/Index.razor` renders a `HeadingComponent` instance:</span></span>

```razor
<HeadingComponent />
```

<span data-ttu-id="6d7a2-193">`Components/HeadingComponent.razor`:</span><span class="sxs-lookup"><span data-stu-id="6d7a2-193">`Components/HeadingComponent.razor`:</span></span>

[!code-razor[](index/samples_snapshot/HeadingComponent.razor)]

<span data-ttu-id="6d7a2-194">Se un componente contiene un elemento HTML con una prima lettera maiuscola che non corrisponde a un nome di componente, viene emesso un avviso che indica che l'elemento ha un nome imprevisto.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-194">If a component contains an HTML element with an uppercase first letter that doesn't match a component name, a warning is emitted indicating that the element has an unexpected name.</span></span> <span data-ttu-id="6d7a2-195">L'aggiunta di una [`@using`][2] direttiva per lo spazio dei nomi del componente rende disponibile il componente, che risolve l'avviso.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-195">Adding an [`@using`][2] directive for the component's namespace makes the component available, which resolves the warning.</span></span>

## <a name="parameters"></a><span data-ttu-id="6d7a2-196">Parametri</span><span class="sxs-lookup"><span data-stu-id="6d7a2-196">Parameters</span></span>

### <a name="route-parameters"></a><span data-ttu-id="6d7a2-197">Parametri di route</span><span class="sxs-lookup"><span data-stu-id="6d7a2-197">Route parameters</span></span>

<span data-ttu-id="6d7a2-198">I componenti possono ricevere parametri di route dal modello di route fornito nella [`@page`][9] direttiva.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-198">Components can receive route parameters from the route template provided in the [`@page`][9] directive.</span></span> <span data-ttu-id="6d7a2-199">Il router usa parametri di route per popolare i parametri del componente corrispondente.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-199">The router uses route parameters to populate the corresponding component parameters.</span></span>

::: moniker range=">= aspnetcore-5.0"

<span data-ttu-id="6d7a2-200">Sono supportati i parametri facoltativi.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-200">Optional parameters are supported.</span></span> <span data-ttu-id="6d7a2-201">Nell'esempio seguente, il `text` parametro facoltativo assegna il valore del segmento di route alla proprietà del componente `Text` .</span><span class="sxs-lookup"><span data-stu-id="6d7a2-201">In the following example, the `text` optional parameter assigns the value of the route segment to the component's `Text` property.</span></span> <span data-ttu-id="6d7a2-202">Se il segmento non è presente, il valore di `Text` è impostato su `fantastic` .</span><span class="sxs-lookup"><span data-stu-id="6d7a2-202">If the segment isn't present, the value of `Text` is set to `fantastic`.</span></span>

<span data-ttu-id="6d7a2-203">`Pages/RouteParameter.razor`:</span><span class="sxs-lookup"><span data-stu-id="6d7a2-203">`Pages/RouteParameter.razor`:</span></span>

[!code-razor[](index/samples_snapshot/RouteParameter-5x.razor?highlight=1,6-7)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

<span data-ttu-id="6d7a2-204">`Pages/RouteParameter.razor`:</span><span class="sxs-lookup"><span data-stu-id="6d7a2-204">`Pages/RouteParameter.razor`:</span></span>

[!code-razor[](index/samples_snapshot/RouteParameter-3x.razor?highlight=2,7-8)]

<span data-ttu-id="6d7a2-205">I parametri facoltativi non sono supportati, quindi [`@page`][9] vengono applicate due direttive nell'esempio precedente.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-205">Optional parameters aren't supported, so two [`@page`][9] directives are applied in the preceding example.</span></span> <span data-ttu-id="6d7a2-206">Il primo consente la navigazione al componente senza un parametro.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-206">The first permits navigation to the component without a parameter.</span></span> <span data-ttu-id="6d7a2-207">La seconda [`@page`][9] direttiva riceve il `{text}` parametro di route e assegna il valore alla `Text` Proprietà.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-207">The second [`@page`][9] directive receives the `{text}` route parameter and assigns the value to the `Text` property.</span></span>

::: moniker-end

<span data-ttu-id="6d7a2-208">Per informazioni sui parametri di route catch-all ( `{*pageRoute}` ), che acquisiscono percorsi tra più limiti di cartella, vedere <xref:blazor/fundamentals/routing#catch-all-route-parameters> .</span><span class="sxs-lookup"><span data-stu-id="6d7a2-208">For information on catch-all route parameters (`{*pageRoute}`), which capture paths across multiple folder boundaries, see <xref:blazor/fundamentals/routing#catch-all-route-parameters>.</span></span>

### <a name="component-parameters"></a><span data-ttu-id="6d7a2-209">Parametri del componente</span><span class="sxs-lookup"><span data-stu-id="6d7a2-209">Component parameters</span></span>

<span data-ttu-id="6d7a2-210">I componenti possono avere *parametri del componente*, che vengono definiti usando proprietà semplici o complesse pubbliche nella classe Component con l' [ `[Parameter]` attributo](xref:Microsoft.AspNetCore.Components.ParameterAttribute).</span><span class="sxs-lookup"><span data-stu-id="6d7a2-210">Components can have *component parameters*, which are defined using public simple or complex properties on the component class with the [`[Parameter]` attribute](xref:Microsoft.AspNetCore.Components.ParameterAttribute).</span></span> <span data-ttu-id="6d7a2-211">Usare gli attributi per specificare gli argomenti per un componente nel markup.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-211">Use attributes to specify arguments for a component in markup.</span></span>

<span data-ttu-id="6d7a2-212">`Components/ChildComponent.razor`:</span><span class="sxs-lookup"><span data-stu-id="6d7a2-212">`Components/ChildComponent.razor`:</span></span>

[!code-razor[](../common/samples/5.x/BlazorWebAssemblySample/Components/ChildComponent.razor?highlight=2,11-12)]

<span data-ttu-id="6d7a2-213">Ai parametri del componente è possibile assegnare un valore predefinito:</span><span class="sxs-lookup"><span data-stu-id="6d7a2-213">Component parameters can be assigned a default value:</span></span>

```csharp
[Parameter]
public string Title { get; set; } = "Panel Title from Child";
```

<span data-ttu-id="6d7a2-214">Nell'esempio seguente dall'app di esempio, `ParentComponent` imposta il valore della `Title` proprietà dell'oggetto `ChildComponent` .</span><span class="sxs-lookup"><span data-stu-id="6d7a2-214">In the following example from the sample app, the `ParentComponent` sets the value of the `Title` property of the `ChildComponent`.</span></span>

<span data-ttu-id="6d7a2-215">`Pages/ParentComponent.razor`:</span><span class="sxs-lookup"><span data-stu-id="6d7a2-215">`Pages/ParentComponent.razor`:</span></span>

[!code-razor[](index/samples_snapshot/ParentComponent.razor?highlight=5-6)]

<span data-ttu-id="6d7a2-216">Per convenzione, un valore di attributo costituito da codice C# viene assegnato a un parametro usando [ Razor il `@` simbolo riservato](xref:mvc/views/razor#razor-syntax):</span><span class="sxs-lookup"><span data-stu-id="6d7a2-216">By convention, an attribute value that consists of C# code is assigned to a parameter using [Razor's reserved `@` symbol](xref:mvc/views/razor#razor-syntax):</span></span>

* <span data-ttu-id="6d7a2-217">Campo o proprietà padre: `Title="@{FIELD OR PROPERTY}` , dove il segnaposto `{FIELD OR PROPERTY}` è un campo o una proprietà C# del componente padre.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-217">Parent field or property: `Title="@{FIELD OR PROPERTY}`, where the placeholder `{FIELD OR PROPERTY}` is a C# field or property of the parent component.</span></span>
* <span data-ttu-id="6d7a2-218">Risultato di un metodo: `Title="@{METHOD}"` , dove il segnaposto `{METHOD}` è un metodo C# del componente padre.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-218">Result of a method: `Title="@{METHOD}"`, where the placeholder `{METHOD}` is a C# method of the parent component.</span></span>
* <span data-ttu-id="6d7a2-219">[Espressione implicita o esplicita](xref:mvc/views/razor#implicit-razor-expressions): `Title="@({EXPRESSION})"` , dove il segnaposto `{EXPRESSION}` è un'espressione C#.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-219">[Implicit or explicit expression](xref:mvc/views/razor#implicit-razor-expressions): `Title="@({EXPRESSION})"`, where the placeholder `{EXPRESSION}` is a C# expression.</span></span>
  
<span data-ttu-id="6d7a2-220">Per ulteriori informazioni, vedere [ Razor riferimento alla sintassi per ASP.NET Core](xref:mvc/views/razor).</span><span class="sxs-lookup"><span data-stu-id="6d7a2-220">For more information, see [Razor syntax reference for ASP.NET Core](xref:mvc/views/razor).</span></span>

> [!WARNING]
> <span data-ttu-id="6d7a2-221">Non creare componenti che scrivono nei propri *parametri del componente*, usare invece un campo privato.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-221">Don't create components that write to their own *component parameters*, use a private field instead.</span></span> <span data-ttu-id="6d7a2-222">Per ulteriori informazioni, vedere la sezione [parametri sovrascritti](#overwritten-parameters) .</span><span class="sxs-lookup"><span data-stu-id="6d7a2-222">For more information, see the [Overwritten parameters](#overwritten-parameters) section.</span></span>

## <a name="child-content"></a><span data-ttu-id="6d7a2-223">Contenuto figlio</span><span class="sxs-lookup"><span data-stu-id="6d7a2-223">Child content</span></span>

<span data-ttu-id="6d7a2-224">I componenti possono impostare il contenuto di un altro componente.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-224">Components can set the content of another component.</span></span> <span data-ttu-id="6d7a2-225">Il componente di assegnazione fornisce il contenuto tra i tag che specificano il componente di destinazione.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-225">The assigning component provides the content between the tags that specify the receiving component.</span></span>

<span data-ttu-id="6d7a2-226">Nell'esempio seguente, `ChildComponent` ha una `ChildContent` proprietà che rappresenta un oggetto <xref:Microsoft.AspNetCore.Components.RenderFragment> , che rappresenta un segmento di interfaccia utente di cui eseguire il rendering.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-226">In the following example, the `ChildComponent` has a `ChildContent` property that represents a <xref:Microsoft.AspNetCore.Components.RenderFragment>, which represents a segment of UI to render.</span></span> <span data-ttu-id="6d7a2-227">Il valore di `ChildContent` viene posizionato nel markup del componente in cui deve essere eseguito il rendering del contenuto.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-227">The value of `ChildContent` is positioned in the component's markup where the content should be rendered.</span></span> <span data-ttu-id="6d7a2-228">Il valore di `ChildContent` viene ricevuto dal componente padre e ne viene eseguito il rendering all'interno del pannello bootstrap `panel-body` .</span><span class="sxs-lookup"><span data-stu-id="6d7a2-228">The value of `ChildContent` is received from the parent component and rendered inside the Bootstrap panel's `panel-body`.</span></span>

<span data-ttu-id="6d7a2-229">`Components/ChildComponent.razor`:</span><span class="sxs-lookup"><span data-stu-id="6d7a2-229">`Components/ChildComponent.razor`:</span></span>

[!code-razor[](../common/samples/5.x/BlazorWebAssemblySample/Components/ChildComponent.razor?highlight=3,14-15)]

> [!NOTE]
> <span data-ttu-id="6d7a2-230">La proprietà che riceve il <xref:Microsoft.AspNetCore.Components.RenderFragment> contenuto deve essere denominata `ChildContent` per convenzione.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-230">The property receiving the <xref:Microsoft.AspNetCore.Components.RenderFragment> content must be named `ChildContent` by convention.</span></span>

<span data-ttu-id="6d7a2-231">`ParentComponent`Nell'app di esempio può fornire contenuto per il rendering `ChildComponent` di inserendo il contenuto all'interno dei `<ChildComponent>` tag.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-231">The `ParentComponent` in the sample app can provide content for rendering the `ChildComponent` by placing the content inside the `<ChildComponent>` tags.</span></span>

<span data-ttu-id="6d7a2-232">`Pages/ParentComponent.razor`:</span><span class="sxs-lookup"><span data-stu-id="6d7a2-232">`Pages/ParentComponent.razor`:</span></span>

[!code-razor[](index/samples_snapshot/ParentComponent.razor?highlight=7-8)]

<span data-ttu-id="6d7a2-233">A causa del modo in cui Blazor viene eseguito il rendering del contenuto figlio, i componenti di rendering all'interno di un `for` ciclo richiedono una variabile di indice locale se la variabile del ciclo di incremento viene utilizzata nel contenuto del componente figlio:</span><span class="sxs-lookup"><span data-stu-id="6d7a2-233">Due to the way that Blazor renders child content, rendering components inside a `for` loop requires a local index variable if the incrementing loop variable is used in the child component's content:</span></span>
>
> ```razor
> @for (int c = 0; c < 10; c++)
> {
>     var current = c;
>     <ChildComponent Title="@c">
>         Child Content: Count: @current
>     </ChildComponent>
> }
> ```
>
> <span data-ttu-id="6d7a2-234">In alternativa, usare un `foreach` ciclo con <xref:System.Linq.Enumerable.Range%2A?displayProperty=nameWithType> :</span><span class="sxs-lookup"><span data-stu-id="6d7a2-234">Alternatively, use a `foreach` loop with <xref:System.Linq.Enumerable.Range%2A?displayProperty=nameWithType>:</span></span>
>
> ```razor
> @foreach(var c in Enumerable.Range(0,10))
> {
>     <ChildComponent Title="@c">
>         Child Content: Count: @c
>     </ChildComponent>
> }
> ```

<span data-ttu-id="6d7a2-235">Per informazioni sul modo in <xref:Microsoft.AspNetCore.Components.RenderFragment> cui è possibile usare un modello per Razor l'interfaccia utente del componente, vedere gli articoli seguenti:</span><span class="sxs-lookup"><span data-stu-id="6d7a2-235">For information on how a <xref:Microsoft.AspNetCore.Components.RenderFragment> can be used as a template for Razor component UI, see the following articles:</span></span>

* <xref:blazor/components/templated-components>
* <xref:blazor/webassembly-performance-best-practices#define-reusable-renderfragments-in-code>

## <a name="attribute-splatting-and-arbitrary-parameters"></a><span data-ttu-id="6d7a2-236">Attributo splatting e parametri arbitrari</span><span class="sxs-lookup"><span data-stu-id="6d7a2-236">Attribute splatting and arbitrary parameters</span></span>

<span data-ttu-id="6d7a2-237">I componenti possono acquisire ed eseguire il rendering di attributi aggiuntivi oltre ai parametri dichiarati del componente.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-237">Components can capture and render additional attributes in addition to the component's declared parameters.</span></span> <span data-ttu-id="6d7a2-238">È possibile acquisire attributi aggiuntivi in un dizionario e quindi *Splatted* su un elemento quando il componente viene sottoposto a rendering usando la [`@attributes`][3] Razor direttiva.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-238">Additional attributes can be captured in a dictionary and then *splatted* onto an element when the component is rendered using the [`@attributes`][3] Razor directive.</span></span> <span data-ttu-id="6d7a2-239">Questo scenario è utile quando si definisce un componente che produce un elemento di markup che supporta un'ampia gamma di personalizzazioni.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-239">This scenario is useful when defining a component that produces a markup element that supports a variety of customizations.</span></span> <span data-ttu-id="6d7a2-240">Ad esempio, può essere noioso definire gli attributi separatamente per un `<input>` che supporta molti parametri.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-240">For example, it can be tedious to define attributes separately for an `<input>` that supports many parameters.</span></span>

<span data-ttu-id="6d7a2-241">Nell'esempio seguente, il primo `<input>` elemento ( `id="useIndividualParams"` ) USA parametri dei singoli componenti, mentre il secondo `<input>` elemento ( `id="useAttributesDict"` ) usa l'attributo splatting:</span><span class="sxs-lookup"><span data-stu-id="6d7a2-241">In the following example, the first `<input>` element (`id="useIndividualParams"`) uses individual component parameters, while the second `<input>` element (`id="useAttributesDict"`) uses attribute splatting:</span></span>

```razor
<input id="useIndividualParams"
       maxlength="@maxlength"
       placeholder="@placeholder"
       required="@required"
       size="@size" />

<input id="useAttributesDict"
       @attributes="InputAttributes" />

@code {
    public string maxlength = "10";
    public string placeholder = "Input placeholder text";
    public string required = "required";
    public string size = "50";

    public Dictionary<string, object> InputAttributes { get; set; } =
        new Dictionary<string, object>()
        {
            { "maxlength", "10" },
            { "placeholder", "Input placeholder text" },
            { "required", "required" },
            { "size", "50" }
        };
}
```

<span data-ttu-id="6d7a2-242">Il tipo del parametro deve implementare `IEnumerable<KeyValuePair<string, object>>` o `IReadOnlyDictionary<string, object>` con chiavi di stringa.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-242">The type of the parameter must implement `IEnumerable<KeyValuePair<string, object>>` or `IReadOnlyDictionary<string, object>` with string keys.</span></span>

<span data-ttu-id="6d7a2-243">Gli elementi sottoposti a rendering `<input>` usando entrambi gli approcci sono identici:</span><span class="sxs-lookup"><span data-stu-id="6d7a2-243">The rendered `<input>` elements using both approaches is identical:</span></span>

```html
<input id="useIndividualParams"
       maxlength="10"
       placeholder="Input placeholder text"
       required="required"
       size="50">

<input id="useAttributesDict"
       maxlength="10"
       placeholder="Input placeholder text"
       required="required"
       size="50">
```

<span data-ttu-id="6d7a2-244">Per accettare attributi arbitrari, definire un parametro component usando l' [ `[Parameter]` attributo](xref:Microsoft.AspNetCore.Components.ParameterAttribute) con la <xref:Microsoft.AspNetCore.Components.ParameterAttribute.CaptureUnmatchedValues> proprietà impostata su `true` :</span><span class="sxs-lookup"><span data-stu-id="6d7a2-244">To accept arbitrary attributes, define a component parameter using the [`[Parameter]` attribute](xref:Microsoft.AspNetCore.Components.ParameterAttribute) with the <xref:Microsoft.AspNetCore.Components.ParameterAttribute.CaptureUnmatchedValues> property set to `true`:</span></span>

```razor
@code {
    [Parameter(CaptureUnmatchedValues = true)]
    public Dictionary<string, object> InputAttributes { get; set; }
}
```

<span data-ttu-id="6d7a2-245">La <xref:Microsoft.AspNetCore.Components.ParameterAttribute.CaptureUnmatchedValues> proprietà su [`[Parameter]`](xref:Microsoft.AspNetCore.Components.ParameterAttribute) consente al parametro di trovare la corrispondenza con tutti gli attributi che non corrispondono ad altri parametri.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-245">The <xref:Microsoft.AspNetCore.Components.ParameterAttribute.CaptureUnmatchedValues> property on [`[Parameter]`](xref:Microsoft.AspNetCore.Components.ParameterAttribute) allows the parameter to match all attributes that don't match any other parameter.</span></span> <span data-ttu-id="6d7a2-246">Un componente può definire solo un singolo parametro con <xref:Microsoft.AspNetCore.Components.ParameterAttribute.CaptureUnmatchedValues> .</span><span class="sxs-lookup"><span data-stu-id="6d7a2-246">A component can only define a single parameter with <xref:Microsoft.AspNetCore.Components.ParameterAttribute.CaptureUnmatchedValues>.</span></span> <span data-ttu-id="6d7a2-247">Il tipo di proprietà utilizzato con <xref:Microsoft.AspNetCore.Components.ParameterAttribute.CaptureUnmatchedValues> deve essere assegnabile da `Dictionary<string, object>` con chiavi di stringa.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-247">The property type used with <xref:Microsoft.AspNetCore.Components.ParameterAttribute.CaptureUnmatchedValues> must be assignable from `Dictionary<string, object>` with string keys.</span></span> <span data-ttu-id="6d7a2-248">`IEnumerable<KeyValuePair<string, object>>` o `IReadOnlyDictionary<string, object>` sono anche opzioni in questo scenario.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-248">`IEnumerable<KeyValuePair<string, object>>` or `IReadOnlyDictionary<string, object>` are also options in this scenario.</span></span>

<span data-ttu-id="6d7a2-249">La posizione di [`@attributes`][3] relativa alla posizione degli attributi degli elementi è importante.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-249">The position of [`@attributes`][3] relative to the position of element attributes is important.</span></span> <span data-ttu-id="6d7a2-250">Quando [`@attributes`][3] Splatted sull'elemento, gli attributi vengono elaborati da destra a sinistra (da ultimo a primo).</span><span class="sxs-lookup"><span data-stu-id="6d7a2-250">When [`@attributes`][3] are splatted on the element, the attributes are processed from right to left (last to first).</span></span> <span data-ttu-id="6d7a2-251">Si consideri l'esempio seguente di un componente che utilizza un `Child` componente:</span><span class="sxs-lookup"><span data-stu-id="6d7a2-251">Consider the following example of a component that consumes a `Child` component:</span></span>

<span data-ttu-id="6d7a2-252">`ParentComponent.razor`:</span><span class="sxs-lookup"><span data-stu-id="6d7a2-252">`ParentComponent.razor`:</span></span>

```razor
<ChildComponent extra="10" />
```

<span data-ttu-id="6d7a2-253">`ChildComponent.razor`:</span><span class="sxs-lookup"><span data-stu-id="6d7a2-253">`ChildComponent.razor`:</span></span>

```razor
<div @attributes="AdditionalAttributes" extra="5" />

[Parameter(CaptureUnmatchedValues = true)]
public IDictionary<string, object> AdditionalAttributes { get; set; }
```

<span data-ttu-id="6d7a2-254">L' `Child` attributo del componente `extra` viene impostato a destra di [`@attributes`][3] .</span><span class="sxs-lookup"><span data-stu-id="6d7a2-254">The `Child` component's `extra` attribute is set to the right of [`@attributes`][3].</span></span> <span data-ttu-id="6d7a2-255">Il `Parent` rendering del componente `<div>` contiene `extra="5"` quando viene passato attraverso l'attributo aggiuntivo perché gli attributi vengono elaborati da destra a sinistra (da ultimo a primo):</span><span class="sxs-lookup"><span data-stu-id="6d7a2-255">The `Parent` component's rendered `<div>` contains `extra="5"` when passed through the additional attribute because the attributes are processed right to left (last to first):</span></span>

```html
<div extra="5" />
```

<span data-ttu-id="6d7a2-256">Nell'esempio seguente, l'ordine di `extra` e [`@attributes`][3] viene invertito nell'oggetto del `Child` componente `<div>` :</span><span class="sxs-lookup"><span data-stu-id="6d7a2-256">In the following example, the order of `extra` and [`@attributes`][3] is reversed in the `Child` component's `<div>`:</span></span>

<span data-ttu-id="6d7a2-257">`ParentComponent.razor`:</span><span class="sxs-lookup"><span data-stu-id="6d7a2-257">`ParentComponent.razor`:</span></span>

```razor
<ChildComponent extra="10" />
```

<span data-ttu-id="6d7a2-258">`ChildComponent.razor`:</span><span class="sxs-lookup"><span data-stu-id="6d7a2-258">`ChildComponent.razor`:</span></span>

```razor
<div extra="5" @attributes="AdditionalAttributes" />

[Parameter(CaptureUnmatchedValues = true)]
public IDictionary<string, object> AdditionalAttributes { get; set; }
```

<span data-ttu-id="6d7a2-259">Il rendering `<div>` nel `Parent` componente contiene `extra="10"` quando viene passato tramite l'attributo aggiuntivo:</span><span class="sxs-lookup"><span data-stu-id="6d7a2-259">The rendered `<div>` in the `Parent` component contains `extra="10"` when passed through the additional attribute:</span></span>

```html
<div extra="10" />
```

## <a name="capture-references-to-components"></a><span data-ttu-id="6d7a2-260">Acquisisci riferimenti ai componenti</span><span class="sxs-lookup"><span data-stu-id="6d7a2-260">Capture references to components</span></span>

<span data-ttu-id="6d7a2-261">I riferimenti ai componenti forniscono un modo per fare riferimento a un'istanza del componente in modo da poter emettere comandi per tale istanza, ad esempio `Show` o `Reset` .</span><span class="sxs-lookup"><span data-stu-id="6d7a2-261">Component references provide a way to reference a component instance so that you can issue commands to that instance, such as `Show` or `Reset`.</span></span> <span data-ttu-id="6d7a2-262">Per acquisire un riferimento a un componente:</span><span class="sxs-lookup"><span data-stu-id="6d7a2-262">To capture a component reference:</span></span>

* <span data-ttu-id="6d7a2-263">Aggiungere un [`@ref`][4] attributo al componente figlio.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-263">Add an [`@ref`][4] attribute to the child component.</span></span>
* <span data-ttu-id="6d7a2-264">Definire un campo con lo stesso tipo del componente figlio.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-264">Define a field with the same type as the child component.</span></span>

```razor
<CustomLoginDialog @ref="loginDialog" ... />

@code {
    private CustomLoginDialog loginDialog;

    private void OnSomething()
    {
        loginDialog.Show();
    }
}
```

<span data-ttu-id="6d7a2-265">Quando viene eseguito il rendering del componente, il `loginDialog` campo viene popolato con l' `CustomLoginDialog` istanza del componente figlio.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-265">When the component is rendered, the `loginDialog` field is populated with the `CustomLoginDialog` child component instance.</span></span> <span data-ttu-id="6d7a2-266">È quindi possibile richiamare i metodi .NET nell'istanza del componente.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-266">You can then invoke .NET methods on the component instance.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="6d7a2-267">La `loginDialog` variabile viene popolata solo dopo il rendering del componente e l'output include `MyLoginDialog` l'elemento.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-267">The `loginDialog` variable is only populated after the component is rendered and its output includes the `MyLoginDialog` element.</span></span> <span data-ttu-id="6d7a2-268">Fino a quando non viene eseguito il rendering del componente, non vi sono riferimenti.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-268">Until the component is rendered, there's nothing to reference.</span></span>
>
> <span data-ttu-id="6d7a2-269">Per modificare i riferimenti ai componenti dopo che il componente ha terminato il rendering, usare i [ `OnAfterRenderAsync` `OnAfterRender` metodi o](xref:blazor/components/lifecycle#after-component-render).</span><span class="sxs-lookup"><span data-stu-id="6d7a2-269">To manipulate components references after the component has finished rendering, use the [`OnAfterRenderAsync` or `OnAfterRender` methods](xref:blazor/components/lifecycle#after-component-render).</span></span>
>
> <span data-ttu-id="6d7a2-270">Per usare una variabile di riferimento con un gestore eventi, usare un'espressione lambda o assegnare il delegato del gestore eventi nei [ `OnAfterRenderAsync` `OnAfterRender` metodi o](xref:blazor/components/lifecycle#after-component-render).</span><span class="sxs-lookup"><span data-stu-id="6d7a2-270">To use a reference variable with an event handler, use a lambda expression or assign the event handler delegate in the [`OnAfterRenderAsync` or `OnAfterRender` methods](xref:blazor/components/lifecycle#after-component-render).</span></span> <span data-ttu-id="6d7a2-271">In questo modo si garantisce che la variabile di riferimento venga assegnata prima dell'assegnazione del gestore eventi.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-271">This ensures that the reference variable is assigned before the event handler is assigned.</span></span>
>
> ```razor
> <button type="button" 
>     @onclick="@(() => loginDialog.DoSomething())">Do Something</button>
>
> <MyLoginDialog @ref="loginDialog" ... />
>
> @code {
>     private MyLoginDialog loginDialog;
> }
> ```

<span data-ttu-id="6d7a2-272">Per fare riferimento ai componenti in un ciclo, vedere [Capture References to multiple analogous Child-Components (DotNet/aspnetcore #13358)](https://github.com/dotnet/aspnetcore/issues/13358).</span><span class="sxs-lookup"><span data-stu-id="6d7a2-272">To reference components in a loop, see [Capture references to multiple similar child-components (dotnet/aspnetcore #13358)](https://github.com/dotnet/aspnetcore/issues/13358).</span></span>

<span data-ttu-id="6d7a2-273">Mentre l'acquisizione di riferimenti ai componenti usa una sintassi simile per l' [acquisizione di riferimenti a elementi](xref:blazor/call-javascript-from-dotnet#capture-references-to-elements), non è una funzionalità di interoperabilità di JavaScript.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-273">While capturing component references use a similar syntax to [capturing element references](xref:blazor/call-javascript-from-dotnet#capture-references-to-elements), it isn't a JavaScript interop feature.</span></span> <span data-ttu-id="6d7a2-274">I riferimenti ai componenti non vengono passati al codice JavaScript.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-274">Component references aren't passed to JavaScript code.</span></span> <span data-ttu-id="6d7a2-275">I riferimenti ai componenti vengono usati solo nel codice .NET.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-275">Component references are only used in .NET code.</span></span>

> [!NOTE]
> <span data-ttu-id="6d7a2-276">**Non** usare i riferimenti ai componenti per mutare lo stato dei componenti figlio.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-276">Do **not** use component references to mutate the state of child components.</span></span> <span data-ttu-id="6d7a2-277">Usare invece i normali parametri dichiarativi per passare i dati ai componenti figlio.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-277">Instead, use normal declarative parameters to pass data to child components.</span></span> <span data-ttu-id="6d7a2-278">L'utilizzo di normali parametri dichiarativi restituisce automaticamente i componenti figlio che eseguono il rendering alle ore corrette.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-278">Use of normal declarative parameters result in child components that rerender at the correct times automatically.</span></span>

## <a name="synchronization-context"></a><span data-ttu-id="6d7a2-279">Contesto di sincronizzazione</span><span class="sxs-lookup"><span data-stu-id="6d7a2-279">Synchronization context</span></span>

<span data-ttu-id="6d7a2-280">Blazor Usa un contesto di sincronizzazione ( <xref:System.Threading.SynchronizationContext> ) per applicare un singolo thread logico di esecuzione.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-280">Blazor uses a synchronization context (<xref:System.Threading.SynchronizationContext>) to enforce a single logical thread of execution.</span></span> <span data-ttu-id="6d7a2-281">I metodi del [ciclo](xref:blazor/components/lifecycle) di vita di un componente e tutti i callback di evento generati da Blazor vengono eseguiti nel contesto di sincronizzazione.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-281">A component's [lifecycle methods](xref:blazor/components/lifecycle) and any event callbacks that are raised by Blazor are executed on the synchronization context.</span></span>

<span data-ttu-id="6d7a2-282">Blazor Serveril contesto di sincronizzazione tenta di emulare un ambiente a thread singolo in modo che corrisponda strettamente al modello webassembly nel browser, ovvero a thread singolo.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-282">Blazor Server's synchronization context attempts to emulate a single-threaded environment so that it closely matches the WebAssembly model in the browser, which is single threaded.</span></span> <span data-ttu-id="6d7a2-283">In un determinato momento, il lavoro viene eseguito su un solo thread, con l'impressione di un singolo thread logico.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-283">At any given point in time, work is performed on exactly one thread, giving the impression of a single logical thread.</span></span> <span data-ttu-id="6d7a2-284">Non vengono eseguite contemporaneamente due operazioni.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-284">No two operations execute concurrently.</span></span>

### <a name="avoid-thread-blocking-calls"></a><span data-ttu-id="6d7a2-285">Evitare chiamate di blocco del thread</span><span class="sxs-lookup"><span data-stu-id="6d7a2-285">Avoid thread-blocking calls</span></span>

<span data-ttu-id="6d7a2-286">In genere, non chiamare i metodi seguenti.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-286">Generally, don't call the following methods.</span></span> <span data-ttu-id="6d7a2-287">I metodi seguenti bloccano il thread e quindi impediscono all'app di riprendere il lavoro fino al completamento dell'oggetto sottostante <xref:System.Threading.Tasks.Task> :</span><span class="sxs-lookup"><span data-stu-id="6d7a2-287">The following methods block the thread and thus block the app from resuming work until the underlying <xref:System.Threading.Tasks.Task> is complete:</span></span>

* <xref:System.Threading.Tasks.Task%601.Result%2A>
* <xref:System.Threading.Tasks.Task.Wait%2A>
* <xref:System.Threading.Tasks.Task.WaitAny%2A>
* <xref:System.Threading.Tasks.Task.WaitAll%2A>
* <xref:System.Threading.Thread.Sleep%2A>
* <xref:System.Runtime.CompilerServices.TaskAwaiter.GetResult%2A>

### <a name="invoke-component-methods-externally-to-update-state"></a><span data-ttu-id="6d7a2-288">Richiama i metodi del componente esternamente per aggiornare lo stato</span><span class="sxs-lookup"><span data-stu-id="6d7a2-288">Invoke component methods externally to update state</span></span>

<span data-ttu-id="6d7a2-289">Nel caso in cui un componente deve essere aggiornato in base a un evento esterno, ad esempio un timer o altre notifiche, usare il `InvokeAsync` metodo, che invia il Blazor contesto di sincronizzazione di.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-289">In the event a component must be updated based on an external event, such as a timer or other notifications, use the `InvokeAsync` method, which dispatches to Blazor's synchronization context.</span></span> <span data-ttu-id="6d7a2-290">Si consideri ad esempio un *servizio Notifier* che può inviare notifiche a qualsiasi componente in ascolto dello stato aggiornato:</span><span class="sxs-lookup"><span data-stu-id="6d7a2-290">For example, consider a *notifier service* that can notify any listening component of the updated state:</span></span>

```csharp
public class NotifierService
{
    // Can be called from anywhere
    public async Task Update(string key, int value)
    {
        if (Notify != null)
        {
            await Notify.Invoke(key, value);
        }
    }

    public event Func<string, int, Task> Notify;
}
```

<span data-ttu-id="6d7a2-291">Registrare `NotifierService` :</span><span class="sxs-lookup"><span data-stu-id="6d7a2-291">Register the `NotifierService`:</span></span>

* <span data-ttu-id="6d7a2-292">In Blazor WebAssembly registrare il servizio come singleton in `Program.Main` :</span><span class="sxs-lookup"><span data-stu-id="6d7a2-292">In Blazor WebAssembly, register the service as singleton in `Program.Main`:</span></span>

  ```csharp
  builder.Services.AddSingleton<NotifierService>();
  ```

* <span data-ttu-id="6d7a2-293">In Blazor Server registrare il servizio come ambito in `Startup.ConfigureServices` :</span><span class="sxs-lookup"><span data-stu-id="6d7a2-293">In Blazor Server, register the service as scoped in `Startup.ConfigureServices`:</span></span>

  ```csharp
  services.AddScoped<NotifierService>();
  ```

<span data-ttu-id="6d7a2-294">Usare `NotifierService` per aggiornare un componente:</span><span class="sxs-lookup"><span data-stu-id="6d7a2-294">Use the `NotifierService` to update a component:</span></span>

```razor
@page "/"
@inject NotifierService Notifier
@implements IDisposable

<p>Last update: @lastNotification.key = @lastNotification.value</p>

@code {
    private (string key, int value) lastNotification;

    protected override void OnInitialized()
    {
        Notifier.Notify += OnNotify;
    }

    public async Task OnNotify(string key, int value)
    {
        await InvokeAsync(() =>
        {
            lastNotification = (key, value);
            StateHasChanged();
        });
    }

    public void Dispose()
    {
        Notifier.Notify -= OnNotify;
    }
}
```

<span data-ttu-id="6d7a2-295">Nell'esempio precedente:</span><span class="sxs-lookup"><span data-stu-id="6d7a2-295">In the preceding example:</span></span>

* <span data-ttu-id="6d7a2-296">`NotifierService` richiama il metodo del componente `OnNotify` all'esterno del Blazor contesto di sincronizzazione di.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-296">`NotifierService` invokes the component's `OnNotify` method outside of Blazor's synchronization context.</span></span> <span data-ttu-id="6d7a2-297">`InvokeAsync` viene usato per passare al contesto corretto e accodare un rendering.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-297">`InvokeAsync` is used to switch to the correct context and queue a render.</span></span> <span data-ttu-id="6d7a2-298">Per altre informazioni, vedere <xref:blazor/components/rendering>.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-298">For more information, see <xref:blazor/components/rendering>.</span></span>
* <span data-ttu-id="6d7a2-299">Il componente implementa <xref:System.IDisposable> e viene annullata la `OnNotify` sottoscrizione del delegato nel `Dispose` metodo, che viene chiamato dal framework quando il componente viene eliminato.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-299">The component implements <xref:System.IDisposable>, and the `OnNotify` delegate is unsubscribed in the `Dispose` method, which is called by the framework when the component is disposed.</span></span> <span data-ttu-id="6d7a2-300">Per altre informazioni, vedere <xref:blazor/components/lifecycle#component-disposal-with-idisposable>.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-300">For more information, see <xref:blazor/components/lifecycle#component-disposal-with-idisposable>.</span></span>

## <a name="use-key-to-control-the-preservation-of-elements-and-components"></a><span data-ttu-id="6d7a2-301">Usare \@ la chiave per controllare la conservazione di elementi e componenti</span><span class="sxs-lookup"><span data-stu-id="6d7a2-301">Use \@key to control the preservation of elements and components</span></span>

<span data-ttu-id="6d7a2-302">Quando si esegue il rendering di un elenco di elementi o componenti e gli elementi o i componenti cambiano successivamente, l' Blazor algoritmo diffing deve decidere quali elementi o componenti precedenti possono essere conservati e come eseguire il mapping degli oggetti modello.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-302">When rendering a list of elements or components and the elements or components subsequently change, Blazor's diffing algorithm must decide which of the previous elements or components can be retained and how model objects should map to them.</span></span> <span data-ttu-id="6d7a2-303">In genere, questo processo è automatico e può essere ignorato, ma in alcuni casi potrebbe essere necessario controllare il processo.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-303">Normally, this process is automatic and can be ignored, but there are cases where you may want to control the process.</span></span>

<span data-ttu-id="6d7a2-304">Si consideri l'esempio seguente:</span><span class="sxs-lookup"><span data-stu-id="6d7a2-304">Consider the following example:</span></span>

```csharp
@foreach (var person in People)
{
    <DetailsEditor Details="@person.Details" />
}

@code {
    [Parameter]
    public IEnumerable<Person> People { get; set; }
}
```

<span data-ttu-id="6d7a2-305">Il contenuto della `People` raccolta può cambiare con le voci inserite, eliminate o riordinate.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-305">The contents of the `People` collection may change with inserted, deleted, or re-ordered entries.</span></span> <span data-ttu-id="6d7a2-306">Quando viene eseguito il rendering del componente, è `<DetailsEditor>` possibile che il componente venga modificato in modo da ricevere `Details` valori di parametro diversi.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-306">When the component rerenders, the `<DetailsEditor>` component may change to receive different `Details` parameter values.</span></span> <span data-ttu-id="6d7a2-307">Questo può causare un rirendering più complesso del previsto.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-307">This may cause more complex rerendering than expected.</span></span> <span data-ttu-id="6d7a2-308">In alcuni casi, il rirendering può comportare differenze di comportamento visibili, ad esempio lo stato attivo degli elementi persi.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-308">In some cases, rerendering can lead to visible behavior differences, such as lost element focus.</span></span>

<span data-ttu-id="6d7a2-309">È possibile controllare il processo di mapping con l' [`@key`][5] attributo della direttiva.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-309">The mapping process can be controlled with the [`@key`][5] directive attribute.</span></span> <span data-ttu-id="6d7a2-310">[`@key`][5] fa in modo che l'algoritmo di diffing garantisca la conservazione di elementi o componenti in base al valore della chiave:</span><span class="sxs-lookup"><span data-stu-id="6d7a2-310">[`@key`][5] causes the diffing algorithm to guarantee preservation of elements or components based on the key's value:</span></span>

```csharp
@foreach (var person in People)
{
    <DetailsEditor @key="person" Details="@person.Details" />
}

@code {
    [Parameter]
    public IEnumerable<Person> People { get; set; }
}
```

<span data-ttu-id="6d7a2-311">Quando la `People` raccolta viene modificata, l'algoritmo diffing mantiene l'associazione tra `<DetailsEditor>` istanze e `person` istanze:</span><span class="sxs-lookup"><span data-stu-id="6d7a2-311">When the `People` collection changes, the diffing algorithm retains the association between `<DetailsEditor>` instances and `person` instances:</span></span>

* <span data-ttu-id="6d7a2-312">Se un oggetto `Person` viene eliminato dall' `People` elenco, solo l' `<DetailsEditor>` istanza corrispondente viene rimossa dall'interfaccia utente.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-312">If a `Person` is deleted from the `People` list, only the corresponding `<DetailsEditor>` instance is removed from the UI.</span></span> <span data-ttu-id="6d7a2-313">Altre istanze rimangono invariate.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-313">Other instances are left unchanged.</span></span>
* <span data-ttu-id="6d7a2-314">Se un oggetto viene inserito in una determinata `Person` posizione nell'elenco, viene inserita una nuova istanza in corrispondenza della `<DetailsEditor>` posizione corrispondente.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-314">If a `Person` is inserted at some position in the list, one new `<DetailsEditor>` instance is inserted at that corresponding position.</span></span> <span data-ttu-id="6d7a2-315">Altre istanze rimangono invariate.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-315">Other instances are left unchanged.</span></span>
* <span data-ttu-id="6d7a2-316">Se `Person` le voci vengono riordinate, le `<DetailsEditor>` istanze corrispondenti vengono mantenute e nuovamente ordinate nell'interfaccia utente.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-316">If `Person` entries are re-ordered, the corresponding `<DetailsEditor>` instances are preserved and re-ordered in the UI.</span></span>

<span data-ttu-id="6d7a2-317">In alcuni scenari, l'utilizzo di [`@key`][5] riduce al minimo la complessità del rendering ed evita potenziali problemi con le parti con stato del DOM che cambiano, ad esempio la posizione dello stato attivo.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-317">In some scenarios, use of [`@key`][5] minimizes the complexity of rerendering and avoids potential issues with stateful parts of the DOM changing, such as focus position.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="6d7a2-318">Le chiavi sono locali per ogni elemento contenitore o componente.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-318">Keys are local to each container element or component.</span></span> <span data-ttu-id="6d7a2-319">Le chiavi non vengono confrontate globalmente nell'intero documento.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-319">Keys aren't compared globally across the document.</span></span>

### <a name="when-to-use-key"></a><span data-ttu-id="6d7a2-320">Quando usare la \@ chiave</span><span class="sxs-lookup"><span data-stu-id="6d7a2-320">When to use \@key</span></span>

<span data-ttu-id="6d7a2-321">In genere, è opportuno usare [`@key`][5] ogni volta che viene eseguito il rendering di un elenco (ad esempio in un blocco [foreach](/dotnet/csharp/language-reference/keywords/foreach-in) ) e un valore appropriato per definire [`@key`][5] .</span><span class="sxs-lookup"><span data-stu-id="6d7a2-321">Typically, it makes sense to use [`@key`][5] whenever a list is rendered (for example, in a [foreach](/dotnet/csharp/language-reference/keywords/foreach-in) block) and a suitable value exists to define the [`@key`][5].</span></span>

<span data-ttu-id="6d7a2-322">È anche possibile usare [`@key`][5] per impedire a Blazor di mantenere un sottoalbero di un elemento o di un componente quando un oggetto viene modificato:</span><span class="sxs-lookup"><span data-stu-id="6d7a2-322">You can also use [`@key`][5] to prevent Blazor from preserving an element or component subtree when an object changes:</span></span>

```razor
<div @key="currentPerson">
    ... content that depends on currentPerson ...
</div>
```

<span data-ttu-id="6d7a2-323">Se `@currentPerson` viene modificata, la [`@key`][5] direttiva attribute impone Blazor di rimuovere l'intero oggetto `<div>` e i relativi discendenti e di ricompilare il sottoalbero all'interno dell'interfaccia utente con nuovi elementi e componenti.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-323">If `@currentPerson` changes, the [`@key`][5] attribute directive forces Blazor to discard the entire `<div>` and its descendants and rebuild the subtree within the UI with new elements and components.</span></span> <span data-ttu-id="6d7a2-324">Questo può essere utile se è necessario garantire che lo stato dell'interfaccia utente non venga mantenuto quando viene `@currentPerson` modificato.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-324">This can be useful if you need to guarantee that no UI state is preserved when `@currentPerson` changes.</span></span>

### <a name="when-not-to-use-key"></a><span data-ttu-id="6d7a2-325">Quando non usare la \@ chiave</span><span class="sxs-lookup"><span data-stu-id="6d7a2-325">When not to use \@key</span></span>

<span data-ttu-id="6d7a2-326">Si verifica un costo in termini di prestazioni quando si verificano differenze con [`@key`][5] .</span><span class="sxs-lookup"><span data-stu-id="6d7a2-326">There's a performance cost when diffing with [`@key`][5].</span></span> <span data-ttu-id="6d7a2-327">Il costo delle prestazioni non è elevato, ma è sufficiente specificare solo [`@key`][5] se il controllo delle regole di conservazione degli elementi o dei componenti avvantaggia l'app.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-327">The performance cost isn't large, but only specify [`@key`][5] if controlling the element or component preservation rules benefit the app.</span></span>

<span data-ttu-id="6d7a2-328">Anche se [`@key`][5] non viene usato, Blazor conserva le istanze di elementi e componenti figlio il più possibile.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-328">Even if [`@key`][5] isn't used, Blazor preserves child element and component instances as much as possible.</span></span> <span data-ttu-id="6d7a2-329">L'unico vantaggio dell'utilizzo di [`@key`][5] è il controllo sulla *modalità* di mapping delle istanze del modello alle istanze dei componenti mantenute, anziché sull'algoritmo di diffing che seleziona il mapping.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-329">The only advantage to using [`@key`][5] is control over *how* model instances are mapped to the preserved component instances, instead of the diffing algorithm selecting the mapping.</span></span>

### <a name="what-values-to-use-for-key"></a><span data-ttu-id="6d7a2-330">Valori da usare per la \@ chiave</span><span class="sxs-lookup"><span data-stu-id="6d7a2-330">What values to use for \@key</span></span>

<span data-ttu-id="6d7a2-331">In genere, è opportuno fornire uno dei seguenti tipi di valore per [`@key`][5] :</span><span class="sxs-lookup"><span data-stu-id="6d7a2-331">Generally, it makes sense to supply one of the following kinds of value for [`@key`][5]:</span></span>

* <span data-ttu-id="6d7a2-332">Istanze di oggetti modello (ad esempio, un' `Person` istanza come nell'esempio precedente).</span><span class="sxs-lookup"><span data-stu-id="6d7a2-332">Model object instances (for example, a `Person` instance as in the earlier example).</span></span> <span data-ttu-id="6d7a2-333">In questo modo si garantisce la conservazione in base all'uguaglianza del riferimento all'oggetto.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-333">This ensures preservation based on object reference equality.</span></span>
* <span data-ttu-id="6d7a2-334">Identificatori univoci, ad esempio valori di chiave primaria di tipo `int` , `string` o `Guid` .</span><span class="sxs-lookup"><span data-stu-id="6d7a2-334">Unique identifiers (for example, primary key values of type `int`, `string`, or `Guid`).</span></span>

<span data-ttu-id="6d7a2-335">Verificare che i valori usati per non siano in [`@key`][5] conflitto.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-335">Ensure that values used for [`@key`][5] don't clash.</span></span> <span data-ttu-id="6d7a2-336">Se vengono rilevati valori di conflitto nello stesso elemento padre, Blazor genera un'eccezione perché non è in grado di eseguire il mapping deterministico di elementi o componenti precedenti a nuovi elementi o componenti.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-336">If clashing values are detected within the same parent element, Blazor throws an exception because it can't deterministically map old elements or components to new elements or components.</span></span> <span data-ttu-id="6d7a2-337">Utilizzare solo valori distinti, ad esempio istanze di oggetti o valori di chiave primaria.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-337">Only use distinct values, such as object instances or primary key values.</span></span>

## <a name="overwritten-parameters"></a><span data-ttu-id="6d7a2-338">Parametri sovrascritti</span><span class="sxs-lookup"><span data-stu-id="6d7a2-338">Overwritten parameters</span></span>

<span data-ttu-id="6d7a2-339">Il Blazor Framework impone in genere un'assegnazione di parametro sicura da padre a figlio:</span><span class="sxs-lookup"><span data-stu-id="6d7a2-339">The Blazor framework generally imposes safe parent-to-child parameter assignment:</span></span>

* <span data-ttu-id="6d7a2-340">I parametri non vengono sovrascritti in modo imprevisto.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-340">Parameters aren't overwritten unexpectedly.</span></span>
* <span data-ttu-id="6d7a2-341">Gli effetti collaterali sono ridotti al minimo.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-341">Side-effects are minimized.</span></span> <span data-ttu-id="6d7a2-342">Ad esempio, i rendering aggiuntivi vengono evitati perché possono creare cicli di rendering infiniti.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-342">For example, additional renders are avoided because they may create infinite rendering loops.</span></span>

<span data-ttu-id="6d7a2-343">Un componente figlio riceve nuovi valori di parametro che potrebbero sovrascrivere i valori esistenti quando il componente padre esegue nuovamente il rendering.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-343">A child component receives new parameter values that possibly overwrite existing values when the parent component rerenders.</span></span> <span data-ttu-id="6d7a2-344">Accidentale sovrascrivendo i valori dei parametri in un componente figlio spesso si verifica quando si sviluppa il componente con uno o più parametri associati a dati e lo sviluppatore scrive direttamente in un parametro nell'elemento figlio:</span><span class="sxs-lookup"><span data-stu-id="6d7a2-344">Accidentially overwriting parameter values in a child component often occurs when developing the component with one or more data-bound parameters and the developer writes directly to a parameter in the child:</span></span>

* <span data-ttu-id="6d7a2-345">Il componente figlio viene sottoposto a rendering con uno o più valori di parametro dal componente padre.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-345">The child component is rendered with one or more parameter values from the parent component.</span></span>
* <span data-ttu-id="6d7a2-346">Il figlio scrive direttamente nel valore di un parametro.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-346">The child writes directly to the value of a parameter.</span></span>
* <span data-ttu-id="6d7a2-347">Il componente padre esegue nuovamente il rendering e sovrascrive il valore del parametro del figlio.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-347">The parent component rerenders and overwrites the value of the child's parameter.</span></span>

<span data-ttu-id="6d7a2-348">La possibilità di sovrascrivere i valori del parametro si estende anche nei setter di proprietà del componente figlio.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-348">The potential for overwriting paramater values extends into the child component's property setters, too.</span></span>

<span data-ttu-id="6d7a2-349">**Le linee guida generali non consentono di creare componenti che scrivono direttamente nei propri parametri.**</span><span class="sxs-lookup"><span data-stu-id="6d7a2-349">**Our general guidance is not to create components that directly write to their own parameters.**</span></span>

<span data-ttu-id="6d7a2-350">Si consideri il `Expander` componente difettoso seguente che:</span><span class="sxs-lookup"><span data-stu-id="6d7a2-350">Consider the following faulty `Expander` component that:</span></span>

* <span data-ttu-id="6d7a2-351">Esegue il rendering del contenuto figlio.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-351">Renders child content.</span></span>
* <span data-ttu-id="6d7a2-352">Consente di visualizzare o disabilitare il contenuto figlio con un parametro component ( `Expanded` ).</span><span class="sxs-lookup"><span data-stu-id="6d7a2-352">Toggles showing child content with a component parameter (`Expanded`).</span></span>
* <span data-ttu-id="6d7a2-353">Il componente scrive direttamente nel `Expanded` parametro, che illustra il problema con i parametri sovrascritti e deve essere evitato.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-353">The component writes directly to the `Expanded` parameter, which demonstrates the problem with overwritten parameters and should be avoided.</span></span>

```razor
<div @onclick="Toggle" class="card bg-light mb-3" style="width:30rem">
    <div class="card-body">
        <h2 class="card-title">Toggle (<code>Expanded</code> = @Expanded)</h2>

        @if (Expanded)
        {
            <p class="card-text">@ChildContent</p>
        }
    </div>
</div>

@code {
    [Parameter]
    public bool Expanded { get; set; }

    [Parameter]
    public RenderFragment ChildContent { get; set; }

    private void Toggle()
    {
        Expanded = !Expanded;
    }
}
```

<span data-ttu-id="6d7a2-354">Il `Expander` componente viene aggiunto a un componente padre che può chiamare <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> :</span><span class="sxs-lookup"><span data-stu-id="6d7a2-354">The `Expander` component is added to a parent component that may call <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A>:</span></span>

```razor
@page "/expander"

<Expander Expanded="true">
    Expander 1 content
</Expander>

<Expander Expanded="true" />

<button @onclick="StateHasChanged">
    Call StateHasChanged
</button>
```

<span data-ttu-id="6d7a2-355">Inizialmente, i `Expander` componenti si comportano in modo indipendente quando le proprietà vengono disposte `Expanded` .</span><span class="sxs-lookup"><span data-stu-id="6d7a2-355">Initially, the `Expander` components behave independently when their `Expanded` properties are toggled.</span></span> <span data-ttu-id="6d7a2-356">I componenti figlio gestiscono gli Stati come previsto.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-356">The child components maintain their states as expected.</span></span> <span data-ttu-id="6d7a2-357">Quando <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> viene chiamato nell'elemento padre, il `Expanded` parametro del primo componente figlio viene reimpostato sul valore iniziale ( `true` ).</span><span class="sxs-lookup"><span data-stu-id="6d7a2-357">When <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> is called in the parent, the `Expanded` parameter of the first child component is reset back to its initial value (`true`).</span></span> <span data-ttu-id="6d7a2-358">Il valore del secondo `Expander` componente `Expanded` non viene reimpostato perché nel secondo componente non viene eseguito il rendering del contenuto figlio.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-358">The second `Expander` component's `Expanded` value isn't reset because no child content is rendered in the second component.</span></span>

<span data-ttu-id="6d7a2-359">Per mantenere lo stato nello scenario precedente, usare un *campo privato* nel `Expander` componente per mantenerne lo stato attivato/disattivato.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-359">To maintain state in the preceding scenario, use a *private field* in the `Expander` component to maintain its toggled state.</span></span>

<span data-ttu-id="6d7a2-360">Il componente modificato seguente `Expander` :</span><span class="sxs-lookup"><span data-stu-id="6d7a2-360">The following revised `Expander` component:</span></span>

* <span data-ttu-id="6d7a2-361">Accetta il `Expanded` valore del parametro component dall'elemento padre.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-361">Accepts the `Expanded` component parameter value from the parent.</span></span>
* <span data-ttu-id="6d7a2-362">Assegna il valore del parametro component a un *campo privato* ( `expanded` ) nell' [evento OnInitialized](xref:blazor/components/lifecycle#component-initialization-methods).</span><span class="sxs-lookup"><span data-stu-id="6d7a2-362">Assigns the component parameter value to a *private field* (`expanded`) in the [OnInitialized event](xref:blazor/components/lifecycle#component-initialization-methods).</span></span>
* <span data-ttu-id="6d7a2-363">Usa il campo privato per mantenere lo stato di attivazione/disattivo interno, che dimostra come evitare la scrittura diretta in un parametro.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-363">Uses the private field to maintain its internal toggle state, which demonstrates how to avoid writing directly to a parameter.</span></span>

```razor
<div @onclick="Toggle" class="card bg-light mb-3" style="width:30rem">
    <div class="card-body">
        <h2 class="card-title">Toggle (<code>expanded</code> = @expanded)</h2>

        @if (expanded)
        {
            <p class="card-text">@ChildContent</p>
        }
    </div>
</div>

@code {
    private bool expanded;

    [Parameter]
    public bool Expanded { get; set; }

    [Parameter]
    public RenderFragment ChildContent { get; set; }

    protected override void OnInitialized()
    {
        expanded = Expanded;
    }

    private void Toggle()
    {
        expanded = !expanded;
    }
}
```

<span data-ttu-id="6d7a2-364">Per ulteriori informazioni, vedere [ Blazor errore di associazione bidirezionale (DotNet/aspnetcore #24599)](https://github.com/dotnet/aspnetcore/issues/24599).</span><span class="sxs-lookup"><span data-stu-id="6d7a2-364">For additional information, see [Blazor Two Way Binding Error (dotnet/aspnetcore #24599)](https://github.com/dotnet/aspnetcore/issues/24599).</span></span> 

## <a name="apply-an-attribute"></a><span data-ttu-id="6d7a2-365">Applicare un attributo</span><span class="sxs-lookup"><span data-stu-id="6d7a2-365">Apply an attribute</span></span>

<span data-ttu-id="6d7a2-366">Gli attributi possono essere applicati ai Razor componenti con la [`@attribute`][7] direttiva.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-366">Attributes can be applied to Razor components with the [`@attribute`][7] directive.</span></span> <span data-ttu-id="6d7a2-367">Nell'esempio seguente viene applicato l' [ `[Authorize]` attributo](xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute) alla classe Component:</span><span class="sxs-lookup"><span data-stu-id="6d7a2-367">The following example applies the [`[Authorize]` attribute](xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute) to the component class:</span></span>

```razor
@page "/"
@attribute [Authorize]
```

## <a name="conditional-html-element-attributes"></a><span data-ttu-id="6d7a2-368">Attributi dell'elemento HTML condizionale</span><span class="sxs-lookup"><span data-stu-id="6d7a2-368">Conditional HTML element attributes</span></span>

<span data-ttu-id="6d7a2-369">Gli attributi degli elementi HTML vengono sottoposti a rendering in modo condizionale in base al valore .NET.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-369">HTML element attributes are conditionally rendered based on the .NET value.</span></span> <span data-ttu-id="6d7a2-370">Se il valore è `false` o `null` , l'attributo non viene sottoposto a rendering.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-370">If the value is `false` or `null`, the attribute isn't rendered.</span></span> <span data-ttu-id="6d7a2-371">Se il valore è `true` , l'attributo viene sottoposto a rendering ridotto a icona.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-371">If the value is `true`, the attribute is rendered minimized.</span></span>

<span data-ttu-id="6d7a2-372">Nell'esempio seguente, `IsCompleted` determina se `checked` viene eseguito il rendering nel markup dell'elemento:</span><span class="sxs-lookup"><span data-stu-id="6d7a2-372">In the following example, `IsCompleted` determines if `checked` is rendered in the element's markup:</span></span>

```razor
<input type="checkbox" checked="@IsCompleted" />

@code {
    [Parameter]
    public bool IsCompleted { get; set; }
}
```

<span data-ttu-id="6d7a2-373">Se `IsCompleted` è `true` , viene eseguito il rendering della casella di controllo come segue:</span><span class="sxs-lookup"><span data-stu-id="6d7a2-373">If `IsCompleted` is `true`, the check box is rendered as:</span></span>

```html
<input type="checkbox" checked />
```

<span data-ttu-id="6d7a2-374">Se `IsCompleted` è `false` , viene eseguito il rendering della casella di controllo come segue:</span><span class="sxs-lookup"><span data-stu-id="6d7a2-374">If `IsCompleted` is `false`, the check box is rendered as:</span></span>

```html
<input type="checkbox" />
```

<span data-ttu-id="6d7a2-375">Per ulteriori informazioni, vedere [ Razor riferimento alla sintassi per ASP.NET Core](xref:mvc/views/razor).</span><span class="sxs-lookup"><span data-stu-id="6d7a2-375">For more information, see [Razor syntax reference for ASP.NET Core](xref:mvc/views/razor).</span></span>

> [!WARNING]
> <span data-ttu-id="6d7a2-376">Alcuni attributi HTML, ad esempio [`aria-pressed`](https://developer.mozilla.org/docs/Web/Accessibility/ARIA/Roles/button_role#Toggle_buttons) , non funzionano correttamente quando il tipo .NET è un `bool` .</span><span class="sxs-lookup"><span data-stu-id="6d7a2-376">Some HTML attributes, such as [`aria-pressed`](https://developer.mozilla.org/docs/Web/Accessibility/ARIA/Roles/button_role#Toggle_buttons), don't function properly when the .NET type is a `bool`.</span></span> <span data-ttu-id="6d7a2-377">In questi casi, usare un `string` tipo anziché un `bool` .</span><span class="sxs-lookup"><span data-stu-id="6d7a2-377">In those cases, use a `string` type instead of a `bool`.</span></span>

## <a name="raw-html"></a><span data-ttu-id="6d7a2-378">HTML non elaborato</span><span class="sxs-lookup"><span data-stu-id="6d7a2-378">Raw HTML</span></span>

<span data-ttu-id="6d7a2-379">Le stringhe vengono in genere sottoposte a rendering usando i nodi di testo DOM, il che significa che qualsiasi markup che può contenere viene ignorato e considerato come testo letterale.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-379">Strings are normally rendered using DOM text nodes, which means that any markup they may contain is ignored and treated as literal text.</span></span> <span data-ttu-id="6d7a2-380">Per eseguire il rendering del codice HTML non elaborato, eseguire il wrapping del contenuto HTML in un `MarkupString` valore.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-380">To render raw HTML, wrap the HTML content in a `MarkupString` value.</span></span> <span data-ttu-id="6d7a2-381">Il valore viene analizzato in formato HTML o SVG e inserito nel DOM.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-381">The value is parsed as HTML or SVG and inserted into the DOM.</span></span>

> [!WARNING]
> <span data-ttu-id="6d7a2-382">Il rendering di codice HTML non elaborato costruito da un'origine non attendibile costituisce un rischio per la **sicurezza** e deve essere evitato.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-382">Rendering raw HTML constructed from any untrusted source is a **security risk** and should be avoided!</span></span>

<span data-ttu-id="6d7a2-383">Nell'esempio seguente viene illustrato l'utilizzo del `MarkupString` tipo per aggiungere un blocco di contenuto HTML statico all'output sottoposto a rendering di un componente:</span><span class="sxs-lookup"><span data-stu-id="6d7a2-383">The following example shows using the `MarkupString` type to add a block of static HTML content to the rendered output of a component:</span></span>

```html
@((MarkupString)myMarkup)

@code {
    private string myMarkup = 
        "<p class='markup'>This is a <em>markup string</em>.</p>";
}
```

## <a name="razor-templates"></a><span data-ttu-id="6d7a2-384">Razor modelli</span><span class="sxs-lookup"><span data-stu-id="6d7a2-384">Razor templates</span></span>

<span data-ttu-id="6d7a2-385">I frammenti di rendering possono essere definiti usando la Razor sintassi del modello.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-385">Render fragments can be defined using Razor template syntax.</span></span> <span data-ttu-id="6d7a2-386">Razor i modelli sono un modo per definire un frammento di interfaccia utente e presupporre il formato seguente:</span><span class="sxs-lookup"><span data-stu-id="6d7a2-386">Razor templates are a way to define a UI snippet and assume the following format:</span></span>

```razor
@<{HTML tag}>...</{HTML tag}>
```

<span data-ttu-id="6d7a2-387">Nell'esempio seguente viene illustrato come specificare <xref:Microsoft.AspNetCore.Components.RenderFragment> <xref:Microsoft.AspNetCore.Components.RenderFragment%601> i valori e ed eseguire il rendering dei modelli direttamente in un componente.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-387">The following example illustrates how to specify <xref:Microsoft.AspNetCore.Components.RenderFragment> and <xref:Microsoft.AspNetCore.Components.RenderFragment%601> values and render templates directly in a component.</span></span> <span data-ttu-id="6d7a2-388">I frammenti di rendering possono anche essere passati come argomenti ai [componenti basati su modelli](xref:blazor/components/templated-components).</span><span class="sxs-lookup"><span data-stu-id="6d7a2-388">Render fragments can also be passed as arguments to [templated components](xref:blazor/components/templated-components).</span></span>

```razor
@timeTemplate

@petTemplate(new Pet { Name = "Rex" })

@code {
    private RenderFragment timeTemplate = @<p>The time is @DateTime.Now.</p>;
    private RenderFragment<Pet> petTemplate = (pet) => @<p>Pet: @pet.Name</p>;

    private class Pet
    {
        public string Name { get; set; }
    }
}
```

<span data-ttu-id="6d7a2-389">Output del codice precedente sottoposto a rendering:</span><span class="sxs-lookup"><span data-stu-id="6d7a2-389">Rendered output of the preceding code:</span></span>

```html
<p>The time is 10/04/2018 01:26:52.</p>

<p>Pet: Rex</p>
```

## <a name="static-assets"></a><span data-ttu-id="6d7a2-390">Asset statici</span><span class="sxs-lookup"><span data-stu-id="6d7a2-390">Static assets</span></span>

<span data-ttu-id="6d7a2-391">Blazorsegue la convenzione di ASP.NET Core app che collocano asset statici nella [ `web root (wwwroot)` cartella](xref:fundamentals/index#web-root)del progetto.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-391">Blazor follows the convention of ASP.NET Core apps placing static assets under the project's [`web root (wwwroot)` folder](xref:fundamentals/index#web-root).</span></span>

<span data-ttu-id="6d7a2-392">Usare un percorso relativo di base ( `/` ) per fare riferimento alla radice Web per un asset statico.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-392">Use a base-relative path (`/`) to refer to the web root for a static asset.</span></span> <span data-ttu-id="6d7a2-393">Nell'esempio seguente, `logo.png` si trova fisicamente nella `{PROJECT ROOT}/wwwroot/images` cartella:</span><span class="sxs-lookup"><span data-stu-id="6d7a2-393">In the following example, `logo.png` is physically located in the `{PROJECT ROOT}/wwwroot/images` folder:</span></span>

```razor
<img alt="Company logo" src="/images/logo.png" />
```

<span data-ttu-id="6d7a2-394">Razor i componenti **non** supportano la notazione tilde-barra ( `~/` ).</span><span class="sxs-lookup"><span data-stu-id="6d7a2-394">Razor components do **not** support tilde-slash notation (`~/`).</span></span>

<span data-ttu-id="6d7a2-395">Per informazioni sull'impostazione del percorso di base di un'app, vedere <xref:blazor/host-and-deploy/index#app-base-path> .</span><span class="sxs-lookup"><span data-stu-id="6d7a2-395">For information on setting an app's base path, see <xref:blazor/host-and-deploy/index#app-base-path>.</span></span>

## <a name="tag-helpers-arent-supported-in-components"></a><span data-ttu-id="6d7a2-396">Gli helper tag non sono supportati nei componenti</span><span class="sxs-lookup"><span data-stu-id="6d7a2-396">Tag Helpers aren't supported in components</span></span>

<span data-ttu-id="6d7a2-397">[`Tag Helpers`](xref:mvc/views/tag-helpers/intro) non sono supportati in Razor componenti ( `.razor` file).</span><span class="sxs-lookup"><span data-stu-id="6d7a2-397">[`Tag Helpers`](xref:mvc/views/tag-helpers/intro) aren't supported in Razor components (`.razor` files).</span></span> <span data-ttu-id="6d7a2-398">Per fornire funzionalità di tipo Helper tag in Blazor , creare un componente con la stessa funzionalità dell'helper tag e utilizzare il componente.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-398">To provide Tag Helper-like functionality in Blazor, create a component with the same functionality as the Tag Helper and use the component instead.</span></span>

## <a name="scalable-vector-graphics-svg-images"></a><span data-ttu-id="6d7a2-399">Immagini SVG (Scalable Vector Graphics)</span><span class="sxs-lookup"><span data-stu-id="6d7a2-399">Scalable Vector Graphics (SVG) images</span></span>

<span data-ttu-id="6d7a2-400">Poiché Blazor esegue il rendering di HTML, le immagini supportate dal browser, incluse le immagini SVG (Scalable Vector Graphics) `.svg` , sono supportate tramite il `<img>` Tag:</span><span class="sxs-lookup"><span data-stu-id="6d7a2-400">Since Blazor renders HTML, browser-supported images, including Scalable Vector Graphics (SVG) images (`.svg`), are supported via the `<img>` tag:</span></span>

```html
<img alt="Example image" src="some-image.svg" />
```

<span data-ttu-id="6d7a2-401">Analogamente, le immagini SVG sono supportate nelle regole CSS di un file di foglio di stile ( `.css` ):</span><span class="sxs-lookup"><span data-stu-id="6d7a2-401">Similarly, SVG images are supported in the CSS rules of a stylesheet file (`.css`):</span></span>

```css
.my-element {
    background-image: url("some-image.svg");
}
```

<span data-ttu-id="6d7a2-402">Tuttavia, il markup SVG inline non è supportato in tutti gli scenari.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-402">However, inline SVG markup isn't supported in all scenarios.</span></span> <span data-ttu-id="6d7a2-403">Se si inserisce un `<svg>` tag direttamente in un file di componente ( `.razor` ), il rendering delle immagini di base è supportato, ma molti scenari avanzati non sono ancora supportati.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-403">If you place an `<svg>` tag directly into a component file (`.razor`), basic image rendering is supported but many advanced scenarios aren't yet supported.</span></span> <span data-ttu-id="6d7a2-404">Ad esempio, i `<use>` tag non sono attualmente rispettati e [`@bind`][10] non possono essere usati con alcuni tag svg.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-404">For example, `<use>` tags aren't currently respected, and [`@bind`][10] can't be used with some SVG tags.</span></span> <span data-ttu-id="6d7a2-405">Per ulteriori informazioni, vedere [supporto per SVG in Blazor (#18271 DotNet/aspnetcore)](https://github.com/dotnet/aspnetcore/issues/18271).</span><span class="sxs-lookup"><span data-stu-id="6d7a2-405">For more information, see [SVG support in Blazor (dotnet/aspnetcore #18271)](https://github.com/dotnet/aspnetcore/issues/18271).</span></span>

## <a name="whitespace-rendering-behavior"></a><span data-ttu-id="6d7a2-406">Comportamento di rendering degli spazi vuoti</span><span class="sxs-lookup"><span data-stu-id="6d7a2-406">Whitespace rendering behavior</span></span>

::: moniker range=">= aspnetcore-5.0"

<span data-ttu-id="6d7a2-407">[`@preservewhitespace`](xref:mvc/views/razor#preservewhitespace)Se la direttiva non viene utilizzata con un valore di `true` , per impostazione predefinita viene rimosso uno spazio vuoto aggiuntivo se:</span><span class="sxs-lookup"><span data-stu-id="6d7a2-407">Unless the [`@preservewhitespace`](xref:mvc/views/razor#preservewhitespace) directive is used with a value of `true`, extra whitespace is removed by default if:</span></span>

* <span data-ttu-id="6d7a2-408">Iniziali o finali all'interno di un elemento.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-408">Leading or trailing within an element.</span></span>
* <span data-ttu-id="6d7a2-409">Iniziali o finali in un `RenderFragment` parametro.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-409">Leading or trailing within a `RenderFragment` parameter.</span></span> <span data-ttu-id="6d7a2-410">Ad esempio, il contenuto figlio viene passato a un altro componente.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-410">For example, child content passed to another component.</span></span>
* <span data-ttu-id="6d7a2-411">Precede o segue un blocco di codice C#, ad esempio `@if` o `@foreach` .</span><span class="sxs-lookup"><span data-stu-id="6d7a2-411">It precedes or follows a C# code block, such as `@if` or `@foreach`.</span></span>

<span data-ttu-id="6d7a2-412">La rimozione degli spazi vuoti può influire sull'output sottoposto a rendering quando si usa una regola CSS, ad esempio `white-space: pre` .</span><span class="sxs-lookup"><span data-stu-id="6d7a2-412">Whitespace removal might affect the rendered output when using a CSS rule, such as `white-space: pre`.</span></span> <span data-ttu-id="6d7a2-413">Per disabilitare questa ottimizzazione delle prestazioni e mantenere lo spazio vuoto, effettuare una delle operazioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="6d7a2-413">To disable this performance optimization and preserve the whitespace, take one of the following actions:</span></span>

* <span data-ttu-id="6d7a2-414">Aggiungere la `@preservewhitespace true` direttiva all'inizio del `.razor` file per applicare la preferenza a un componente specifico.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-414">Add the `@preservewhitespace true` directive at the top of the `.razor` file to apply the preference to a specific component.</span></span>
* <span data-ttu-id="6d7a2-415">Aggiungere la `@preservewhitespace true` direttiva all'interno `_Imports.razor` di un file per applicare la preferenza a un'intera sottodirectory o all'intero progetto.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-415">Add the `@preservewhitespace true` directive inside an `_Imports.razor` file to apply the preference to an entire subdirectory or the entire project.</span></span>

<span data-ttu-id="6d7a2-416">Nella maggior parte dei casi, non è necessaria alcuna azione, perché le app in genere continuano a funzionare normalmente (ma più velocemente).</span><span class="sxs-lookup"><span data-stu-id="6d7a2-416">In most cases, no action is required, as apps typically continue to behave normally (but faster).</span></span> <span data-ttu-id="6d7a2-417">Se la rimozione dello spazio vuoto causa problemi per un particolare componente, utilizzare `@preservewhitespace true` in tale componente per disabilitare questa ottimizzazione.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-417">If stripping whitespace causes any problem for a particular component, use `@preservewhitespace true` in that component to disable this optimization.</span></span>

::: moniker-end

::: moniker range="< aspnetcore-5.0"

<span data-ttu-id="6d7a2-418">Lo spazio vuoto viene mantenuto nel codice sorgente di un componente.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-418">Whitespace is retained in a component's source code.</span></span> <span data-ttu-id="6d7a2-419">Viene eseguito il rendering del testo con solo spazi vuoti nell'Document Object Model del browser (DOM) anche quando non è presente alcun effetto visivo.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-419">Whitespace-only text renders in the browser's Document Object Model (DOM) even when there's no visual effect.</span></span>

<span data-ttu-id="6d7a2-420">Si consideri il Razor codice componente seguente:</span><span class="sxs-lookup"><span data-stu-id="6d7a2-420">Consider the following Razor component code:</span></span>

```razor
<ul>
    @foreach (var item in Items)
    {
        <li>
            @item.Text
        </li>
    }
</ul>
```

<span data-ttu-id="6d7a2-421">Nell'esempio precedente viene eseguito il rendering degli spazi vuoti non necessari seguenti:</span><span class="sxs-lookup"><span data-stu-id="6d7a2-421">The preceding example renders the following unnecessary whitespace:</span></span>

* <span data-ttu-id="6d7a2-422">All'esterno del `@foreach` blocco di codice.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-422">Outside of the `@foreach` code block.</span></span>
* <span data-ttu-id="6d7a2-423">Intorno all' `<li>` elemento.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-423">Around the `<li>` element.</span></span>
* <span data-ttu-id="6d7a2-424">Intorno all' `@item.Text` output.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-424">Around the `@item.Text` output.</span></span>

<span data-ttu-id="6d7a2-425">Un elenco contenente 100 elementi restituisce 402 aree di spazio vuoto e nessuno degli spazi vuoti influiscono visivamente sull'output sottoposto a rendering.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-425">A list containing 100 items results in 402 areas of whitespace, and none of the extra whitespace visually affects the rendered output.</span></span>

<span data-ttu-id="6d7a2-426">Quando si esegue il rendering HTML statico per i componenti, gli spazi vuoti all'interno di un tag non vengono conservati</span><span class="sxs-lookup"><span data-stu-id="6d7a2-426">When rendering static HTML for components, whitespace inside a tag isn't preserved.</span></span> <span data-ttu-id="6d7a2-427">Ad esempio, visualizzare l'origine del componente seguente nell'output di cui è stato eseguito il rendering:</span><span class="sxs-lookup"><span data-stu-id="6d7a2-427">For example, view the source of the following component in rendered output:</span></span>

```razor
<img     alt="My image"   src="img.png"     />
```

<span data-ttu-id="6d7a2-428">Lo spazio vuoto non è mantenuto dal Razor markup precedente:</span><span class="sxs-lookup"><span data-stu-id="6d7a2-428">Whitespace isn't preserved from the preceding Razor markup:</span></span>

```razor
<img alt="My image" src="img.png" />
```

::: moniker-end

## <a name="additional-resources"></a><span data-ttu-id="6d7a2-429">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="6d7a2-429">Additional resources</span></span>

* <span data-ttu-id="6d7a2-430"><xref:blazor/security/server/threat-mitigation>: Include informazioni aggiuntive sulla creazione di Blazor Server app che devono essere confrontate con l'esaurimento delle risorse.</span><span class="sxs-lookup"><span data-stu-id="6d7a2-430"><xref:blazor/security/server/threat-mitigation>: Includes guidance on building Blazor Server apps that must contend with resource exhaustion.</span></span>

<!--Reference links in article-->
[1]: <xref:mvc/views/razor#code> "riferimento alla sintassi::: NO-LOC (Razor)::: per ASP.NET Core"
[2]: <xref:mvc/views/razor#using> "riferimento alla sintassi::: NO-LOC (Razor)::: per ASP.NET Core"
[3]: <xref:mvc/views/razor#attributes> "riferimento alla sintassi::: NO-LOC (Razor)::: per ASP.NET Core"
[4]: <xref:mvc/views/razor#ref> "riferimento alla sintassi::: NO-LOC (Razor)::: per ASP.NET Core"
[5]: <xref:mvc/views/razor#key> "riferimento alla sintassi::: NO-LOC (Razor)::: per ASP.NET Core"
[6]: <xref:mvc/views/razor#inherits> "riferimento alla sintassi::: NO-LOC (Razor)::: per ASP.NET Core"
[7]: <xref:mvc/views/razor#attribute> "riferimento alla sintassi::: NO-LOC (Razor)::: per ASP.NET Core"
[8]: <xref:mvc/views/razor#namespace> "riferimento alla sintassi::: NO-LOC (Razor)::: per ASP.NET Core"
[9]: <xref:mvc/views/razor#page> "riferimento alla sintassi::: NO-LOC (Razor)::: per ASP.NET Core"
[10]: <xref:mvc/views/razor#bind> "riferimento alla sintassi::: NO-LOC (Razor)::: per ASP.NET Core"
