---
title: Parte 3, pagine con impalcature Razor
author: rick-anderson
description: Parte 3 della serie di esercitazioni sulle Razor pagine.
ms.author: riande
ms.date: 09/25/2020
ms.custom: contperf-fy21q2
no-loc:
- Index
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
uid: tutorials/razor-pages/page
ms.openlocfilehash: 049e7764766a4d5d535f7d7959a3554b040607c5
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102588503"
---
# <a name="part-3-scaffolded-razor-pages-in-aspnet-core"></a><span data-ttu-id="be3f8-103">Parte 3, Razor pagine con impalcatura in ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="be3f8-103">Part 3, scaffolded Razor Pages in ASP.NET Core</span></span>

<span data-ttu-id="be3f8-104">Autore: [Rick Anderson](https://twitter.com/RickAndMSFT)</span><span class="sxs-lookup"><span data-stu-id="be3f8-104">By [Rick Anderson](https://twitter.com/RickAndMSFT)</span></span>

<span data-ttu-id="be3f8-105">Questa esercitazione esamina le Razor pagine create dall'impalcatura nell' [esercitazione precedente](xref:tutorials/razor-pages/model).</span><span class="sxs-lookup"><span data-stu-id="be3f8-105">This tutorial examines the Razor Pages created by scaffolding in the [previous tutorial](xref:tutorials/razor-pages/model).</span></span>

::: moniker range=">= aspnetcore-5.0"

<span data-ttu-id="be3f8-106">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie50) ([procedura per il download](xref:index#how-to-download-a-sample)).</span><span class="sxs-lookup"><span data-stu-id="be3f8-106">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie50) ([how to download](xref:index#how-to-download-a-sample)).</span></span>

::: moniker-end

::: moniker range="< aspnetcore-5.0 >= aspnetcore-3.0"

<span data-ttu-id="be3f8-107">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie30) ([procedura per il download](xref:index#how-to-download-a-sample)).</span><span class="sxs-lookup"><span data-stu-id="be3f8-107">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie30) ([how to download](xref:index#how-to-download-a-sample)).</span></span>

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

## <a name="the-create-delete-details-and-edit-pages"></a><span data-ttu-id="be3f8-108">Pagine di creazione, eliminazione, dettagli e modifica</span><span class="sxs-lookup"><span data-stu-id="be3f8-108">The Create, Delete, Details, and Edit pages</span></span>

<span data-ttu-id="be3f8-109">Esaminare il modello di pagina *pages/Movies/ Index . cshtml.cs* :</span><span class="sxs-lookup"><span data-stu-id="be3f8-109">Examine the *Pages/Movies/Index.cshtml.cs* Page Model:</span></span>

[!code-csharp[](razor-pages-start/snapshot_sample3/RazorPagesMovie30/Pages/Movies/Index.cshtml.cs)]

<span data-ttu-id="be3f8-110">Razor Le pagine sono derivate da `PageModel` .</span><span class="sxs-lookup"><span data-stu-id="be3f8-110">Razor Pages are derived from `PageModel`.</span></span> <span data-ttu-id="be3f8-111">Per convenzione, la `PageModel` classe derivata da è denominata `<PageName>Model` .</span><span class="sxs-lookup"><span data-stu-id="be3f8-111">By convention, the `PageModel`-derived class is named `<PageName>Model`.</span></span> <span data-ttu-id="be3f8-112">Il costruttore usa l' [inserimento delle dipendenze](xref:fundamentals/dependency-injection) per aggiungere la `RazorPagesMovieContext` alla pagina:</span><span class="sxs-lookup"><span data-stu-id="be3f8-112">The constructor uses [dependency injection](xref:fundamentals/dependency-injection) to add the `RazorPagesMovieContext` to the page:</span></span>

[!code-csharp[](razor-pages-start/snapshot_sample3/RazorPagesMovie30/Pages/Movies/Index.cshtml.cs?name=snippet1&highlight=5)]

<span data-ttu-id="be3f8-113">Vedere [Codice asincrono](xref:data/ef-rp/intro#asynchronous-code) per altre informazioni sulla programmazione asincrona con Entity Framework.</span><span class="sxs-lookup"><span data-stu-id="be3f8-113">See [Asynchronous code](xref:data/ef-rp/intro#asynchronous-code) for more information on asynchronous programming with Entity Framework.</span></span>

<span data-ttu-id="be3f8-114">Quando viene effettuata una richiesta per la pagina, il `OnGetAsync` metodo restituisce un elenco di filmati alla Razor pagina.</span><span class="sxs-lookup"><span data-stu-id="be3f8-114">When a request is made for the page, the `OnGetAsync` method returns a list of movies to the Razor Page.</span></span> <span data-ttu-id="be3f8-115">In una Razor pagina `OnGetAsync` o `OnGet` viene chiamato per inizializzare lo stato della pagina.</span><span class="sxs-lookup"><span data-stu-id="be3f8-115">On a Razor Page, `OnGetAsync` or `OnGet` is called to initialize the state of the page.</span></span> <span data-ttu-id="be3f8-116">In questo caso, `OnGetAsync` ottiene un elenco di filmati e li visualizza.</span><span class="sxs-lookup"><span data-stu-id="be3f8-116">In this case, `OnGetAsync` gets a list of movies and displays them.</span></span>

<span data-ttu-id="be3f8-117">Quando `OnGet` restituisce `void` o `OnGetAsync` restituisce `Task` , non viene utilizzata alcuna istruzione return.</span><span class="sxs-lookup"><span data-stu-id="be3f8-117">When `OnGet` returns `void` or `OnGetAsync` returns `Task`, no return statement is used.</span></span> <span data-ttu-id="be3f8-118">Ad esempio la pagina privacy:</span><span class="sxs-lookup"><span data-stu-id="be3f8-118">For example the Privacy Page:</span></span>

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie30/Pages/Privacy.cshtml.cs?name=snippet)]

<span data-ttu-id="be3f8-119">Quando il tipo restituito è `IActionResult` o `Task<IActionResult>`, è necessario specificare un'istruzione return.</span><span class="sxs-lookup"><span data-stu-id="be3f8-119">When the return type is `IActionResult` or `Task<IActionResult>`, a return statement must be provided.</span></span> <span data-ttu-id="be3f8-120">Ad esempio, il metodo *Pages/Movies/Create.cshtml.cs* `OnPostAsync`:</span><span class="sxs-lookup"><span data-stu-id="be3f8-120">For example, the *Pages/Movies/Create.cshtml.cs* `OnPostAsync` method:</span></span>

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie30/Pages/Movies/Create.cshtml.cs?name=snippet)]

<a name="index"></a> <span data-ttu-id="be3f8-121">Esaminare la pagina *pages/Movies/ Index . cshtml* Razor :</span><span class="sxs-lookup"><span data-stu-id="be3f8-121">Examine the *Pages/Movies/Index.cshtml* Razor Page:</span></span>

[!code-cshtml[](razor-pages-start/snapshot_sample3/RazorPagesMovie30/Pages/Movies/Index.cshtml)]

<span data-ttu-id="be3f8-122">Razor consente di eseguire la transizione da HTML a C# o a un Razor markup specifico.</span><span class="sxs-lookup"><span data-stu-id="be3f8-122">Razor can transition from HTML into C# or into Razor-specific markup.</span></span> <span data-ttu-id="be3f8-123">Quando un `@` simbolo è seguito da una [ Razor parola chiave riservata](xref:mvc/views/razor#razor-reserved-keywords), esegue la transizione in un Razor markup specifico, in caso contrario passa a C#.</span><span class="sxs-lookup"><span data-stu-id="be3f8-123">When an `@` symbol is followed by a [Razor reserved keyword](xref:mvc/views/razor#razor-reserved-keywords), it transitions into Razor-specific markup, otherwise it transitions into C#.</span></span>

### <a name="the-page-directive"></a><span data-ttu-id="be3f8-124">La direttiva @page</span><span class="sxs-lookup"><span data-stu-id="be3f8-124">The @page directive</span></span>

<span data-ttu-id="be3f8-125">La `@page` Razor direttiva rende il file un'azione MVC, il che significa che può gestire le richieste.</span><span class="sxs-lookup"><span data-stu-id="be3f8-125">The `@page` Razor directive makes the file an MVC action, which means that it can handle requests.</span></span> <span data-ttu-id="be3f8-126">`@page` deve essere la prima Razor direttiva in una pagina.</span><span class="sxs-lookup"><span data-stu-id="be3f8-126">`@page` must be the first Razor directive on a page.</span></span> <span data-ttu-id="be3f8-127">`@page` e `@model` sono esempi di transizione al Razor markup specifico.</span><span class="sxs-lookup"><span data-stu-id="be3f8-127">`@page` and `@model` are examples of transitioning into Razor-specific markup.</span></span> <span data-ttu-id="be3f8-128">Per ulteriori informazioni, vedere la [ Razor sintassi](xref:mvc/views/razor#razor-syntax) .</span><span class="sxs-lookup"><span data-stu-id="be3f8-128">See [Razor syntax](xref:mvc/views/razor#razor-syntax) for more information.</span></span>

<a name="md"></a>

### <a name="the-model-directive"></a><span data-ttu-id="be3f8-129">La direttiva @model</span><span class="sxs-lookup"><span data-stu-id="be3f8-129">The @model directive</span></span>

[!code-cshtml[](razor-pages-start/snapshot_sample3/RazorPagesMovie30/Pages/Movies/Index.cshtml?range=1-2&highlight=2)]

<span data-ttu-id="be3f8-130">La `@model` direttiva specifica il tipo del modello passato alla Razor pagina.</span><span class="sxs-lookup"><span data-stu-id="be3f8-130">The `@model` directive specifies the type of the model passed to the Razor Page.</span></span> <span data-ttu-id="be3f8-131">Nell'esempio precedente la `@model` riga rende `PageModel` disponibile la classe derivata da per la Razor pagina.</span><span class="sxs-lookup"><span data-stu-id="be3f8-131">In the preceding example, the `@model` line makes the `PageModel`-derived class available to the Razor Page.</span></span> <span data-ttu-id="be3f8-132">Il modello viene usato negli  [helper HTML](/aspnet/mvc/overview/older-versions-1/views/creating-custom-html-helpers-cs#understanding-html-helpers)`@Html.DisplayNameFor` e `@Html.DisplayFor` nella pagina.</span><span class="sxs-lookup"><span data-stu-id="be3f8-132">The model is used in the `@Html.DisplayNameFor` and `@Html.DisplayFor` [HTML Helpers](/aspnet/mvc/overview/older-versions-1/views/creating-custom-html-helpers-cs#understanding-html-helpers) on the page.</span></span>


<span data-ttu-id="be3f8-133">Esaminare l'espressione lambda usata nell'helper HTML seguente:</span><span class="sxs-lookup"><span data-stu-id="be3f8-133">Examine the lambda expression used in the following HTML Helper:</span></span>

```cshtml
@Html.DisplayNameFor(model => model.Movie[0].Title)
```

<span data-ttu-id="be3f8-134">L'helper HTML <xref:System.Web.Mvc.Html.DisplayNameExtensions.DisplayNameFor%2A?displayProperty=nameWithType> controlla la proprietà `Title` a cui fa riferimento nell'espressione lambda per determinare il nome visualizzato.</span><span class="sxs-lookup"><span data-stu-id="be3f8-134">The <xref:System.Web.Mvc.Html.DisplayNameExtensions.DisplayNameFor%2A?displayProperty=nameWithType> HTML Helper inspects the `Title` property referenced in the lambda expression to determine the display name.</span></span> <span data-ttu-id="be3f8-135">L'espressione lambda viene controllata anziché valutata.</span><span class="sxs-lookup"><span data-stu-id="be3f8-135">The lambda expression is inspected rather than evaluated.</span></span> <span data-ttu-id="be3f8-136">Ciò significa che non esiste alcuna violazione di accesso quando `model` , `model.Movie` o `model.Movie[0]` è `null` o vuoto.</span><span class="sxs-lookup"><span data-stu-id="be3f8-136">That means there is no access violation when `model`, `model.Movie`, or `model.Movie[0]` is `null` or empty.</span></span> <span data-ttu-id="be3f8-137">Quando viene valutata l'espressione lambda, ad esempio con `@Html.DisplayFor(modelItem => item.Title)` , vengono valutati i valori delle proprietà del modello.</span><span class="sxs-lookup"><span data-stu-id="be3f8-137">When the lambda expression is evaluated, for example, with `@Html.DisplayFor(modelItem => item.Title)`, the model's property values are evaluated.</span></span>

### <a name="the-layout-page"></a><span data-ttu-id="be3f8-138">Pagina di layout</span><span class="sxs-lookup"><span data-stu-id="be3f8-138">The layout page</span></span>

<span data-ttu-id="be3f8-139">Selezionare il menu collegamenti **Razor PagesMovie**, **Home** e **privacy**.</span><span class="sxs-lookup"><span data-stu-id="be3f8-139">Select the menu links **RazorPagesMovie**, **Home**, and **Privacy**.</span></span> <span data-ttu-id="be3f8-140">Ogni pagina mostra lo stesso layout di menu.</span><span class="sxs-lookup"><span data-stu-id="be3f8-140">Each page shows the same menu layout.</span></span> <span data-ttu-id="be3f8-141">Il layout di menu viene implementato nel file *Pages/Shared/_Layout.cshtml*.</span><span class="sxs-lookup"><span data-stu-id="be3f8-141">The menu layout is implemented in the *Pages/Shared/_Layout.cshtml* file.</span></span>

<span data-ttu-id="be3f8-142">Aprire ed esaminare il file *pages/Shared/_Layout. cshtml* .</span><span class="sxs-lookup"><span data-stu-id="be3f8-142">Open and examine the *Pages/Shared/_Layout.cshtml* file.</span></span>

<span data-ttu-id="be3f8-143">I modelli [Layout](xref:mvc/views/layout) consentono di:</span><span class="sxs-lookup"><span data-stu-id="be3f8-143">[Layout](xref:mvc/views/layout) templates allow the HTML container layout to be:</span></span>

* <span data-ttu-id="be3f8-144">Specificare il layout del contenitore HTML in un'unica posizione.</span><span class="sxs-lookup"><span data-stu-id="be3f8-144">Specified in one place.</span></span>
* <span data-ttu-id="be3f8-145">Applicare il layout in più pagine del sito.</span><span class="sxs-lookup"><span data-stu-id="be3f8-145">Applied in multiple pages in the site.</span></span>

<span data-ttu-id="be3f8-146">Trovare la riga `@RenderBody()`.</span><span class="sxs-lookup"><span data-stu-id="be3f8-146">Find the `@RenderBody()` line.</span></span> <span data-ttu-id="be3f8-147">`RenderBody` è un segnaposto dove tutte le visualizzazioni specifiche della pagina vengono presentate, *incapsulate* nella pagina di layout.</span><span class="sxs-lookup"><span data-stu-id="be3f8-147">`RenderBody` is a placeholder where all the page-specific views show up, *wrapped* in the layout page.</span></span> <span data-ttu-id="be3f8-148">Ad esempio, selezionare il collegamento **Privacy** per eseguire il rendering della visualizzazione *Pages/Privacy.cshtml* all'interno del metodo `RenderBody`.</span><span class="sxs-lookup"><span data-stu-id="be3f8-148">For example, select the **Privacy** link and the *Pages/Privacy.cshtml* view is rendered inside the `RenderBody` method.</span></span>

<a name="vd"></a>

### <a name="viewdata-and-layout"></a><span data-ttu-id="be3f8-149">ViewData e layout</span><span class="sxs-lookup"><span data-stu-id="be3f8-149">ViewData and layout</span></span>

<span data-ttu-id="be3f8-150">Si consideri il markup seguente dal file *pages/Movies/ Index . cshtml* :</span><span class="sxs-lookup"><span data-stu-id="be3f8-150">Consider the following markup from the *Pages/Movies/Index.cshtml* file:</span></span>

[!code-cshtml[](razor-pages-start/snapshot_sample3/RazorPagesMovie30/Pages/Movies/Index.cshtml?range=1-6&highlight=4-999)]

<span data-ttu-id="be3f8-151">Il markup evidenziato sopra è un esempio di Razor transizione in C#.</span><span class="sxs-lookup"><span data-stu-id="be3f8-151">The preceding highlighted markup is an example of Razor transitioning into C#.</span></span> <span data-ttu-id="be3f8-152">Tra i caratteri `{` e `}` è racchiuso un blocco di codice C#.</span><span class="sxs-lookup"><span data-stu-id="be3f8-152">The `{` and `}` characters enclose a block of C# code.</span></span>

<span data-ttu-id="be3f8-153">La `PageModel` classe base contiene una `ViewData` Proprietà Dictionary che può essere utilizzata per passare dati a una visualizzazione.</span><span class="sxs-lookup"><span data-stu-id="be3f8-153">The `PageModel` base class contains a `ViewData` dictionary property that can be used to pass data to a View.</span></span> <span data-ttu-id="be3f8-154">Gli oggetti vengono aggiunti al `ViewData` dizionario usando un modello di ***valore di chiave*** .</span><span class="sxs-lookup"><span data-stu-id="be3f8-154">Objects are added to the `ViewData` dictionary using a ***key value*** pattern.</span></span> <span data-ttu-id="be3f8-155">Nell'esempio precedente la proprietà `Title` viene aggiunta al dizionario `ViewData`.</span><span class="sxs-lookup"><span data-stu-id="be3f8-155">In the preceding sample, the `Title` property is added to the `ViewData` dictionary.</span></span>

<span data-ttu-id="be3f8-156">La proprietà `Title` viene usata nel file *Pages/Shared/_Layout.cshtml*.</span><span class="sxs-lookup"><span data-stu-id="be3f8-156">The `Title` property is used in the *Pages/Shared/_Layout.cshtml* file.</span></span> <span data-ttu-id="be3f8-157">Il markup seguente illustra le prime righe del file *_Layout.cshtml*.</span><span class="sxs-lookup"><span data-stu-id="be3f8-157">The following markup shows the first few lines of the *_Layout.cshtml* file.</span></span>

<!-- We need a snapshot copy of layout because we are changing in the next step. -->

[!code-cshtml[](razor-pages-start/snapshot_sample/RazorPagesMovie/Pages/NU/_Layout.cshtml?highlight=6)]

<span data-ttu-id="be3f8-158">La riga `@*Markup removed for brevity.*@` è un Razor commento.</span><span class="sxs-lookup"><span data-stu-id="be3f8-158">The line `@*Markup removed for brevity.*@` is a Razor comment.</span></span> <span data-ttu-id="be3f8-159">Diversamente dai commenti HTML `<!-- -->` , Razor i commenti non vengono inviati al client.</span><span class="sxs-lookup"><span data-stu-id="be3f8-159">Unlike HTML comments `<!-- -->`, Razor comments are not sent to the client.</span></span> <span data-ttu-id="be3f8-160">Per ulteriori informazioni, vedere la pagina relativa all'introduzione [al codice HTML](https://developer.mozilla.org/docs/Learn/HTML/Introduction_to_HTML/Getting_started#HTML_comments) .</span><span class="sxs-lookup"><span data-stu-id="be3f8-160">See [MDN web docs: Getting started with HTML](https://developer.mozilla.org/docs/Learn/HTML/Introduction_to_HTML/Getting_started#HTML_comments) for more information.</span></span>

### <a name="update-the-layout"></a><span data-ttu-id="be3f8-161">Aggiornare il layout</span><span class="sxs-lookup"><span data-stu-id="be3f8-161">Update the layout</span></span>

1. <span data-ttu-id="be3f8-162">Modificare l' `<title>` elemento nel file *pages/Shared/_Layout. cshtml* per visualizzare il **film** anziché **Razor PagesMovie**.</span><span class="sxs-lookup"><span data-stu-id="be3f8-162">Change the `<title>` element in the *Pages/Shared/_Layout.cshtml* file to display **Movie** rather than **RazorPagesMovie**.</span></span>

   [!code-cshtml[](razor-pages-start/sample/RazorPagesMovie30/Pages/Shared/_Layout.cshtml?range=1-6&highlight=6)]

1. <span data-ttu-id="be3f8-163">Trovare l'elemento di ancoraggio seguente nel file *Pages/Shared/_Layout.cshtml*.</span><span class="sxs-lookup"><span data-stu-id="be3f8-163">Find the following anchor element in the *Pages/Shared/_Layout.cshtml* file.</span></span>

   ```cshtml
   <a class="navbar-brand" asp-area="" asp-page="/Index">RazorPagesMovie</a>
   ```

1. <span data-ttu-id="be3f8-164">Sostituire l'elemento precedente con il markup seguente:</span><span class="sxs-lookup"><span data-stu-id="be3f8-164">Replace the preceding element with the following markup:</span></span>

   ```cshtml
   <a class="navbar-brand" asp-page="/Movies/Index">RpMovie</a>
   ```

   <span data-ttu-id="be3f8-165">L'elemento di ancoraggio precedente è un [helper tag](xref:mvc/views/tag-helpers/intro).</span><span class="sxs-lookup"><span data-stu-id="be3f8-165">The preceding anchor element is a [Tag Helper](xref:mvc/views/tag-helpers/intro).</span></span> <span data-ttu-id="be3f8-166">In questo caso, si tratta dell'[helper tag di ancoraggio](xref:mvc/views/tag-helpers/builtin-th/anchor-tag-helper).</span><span class="sxs-lookup"><span data-stu-id="be3f8-166">In this case, it's the [Anchor Tag Helper](xref:mvc/views/tag-helpers/builtin-th/anchor-tag-helper).</span></span> <span data-ttu-id="be3f8-167">L' `asp-page="/Movies/Index"` attributo e il valore dell'helper Tag creano un collegamento alla `/Movies/Index` Razor pagina.</span><span class="sxs-lookup"><span data-stu-id="be3f8-167">The `asp-page="/Movies/Index"` Tag Helper attribute and value creates a link to the `/Movies/Index` Razor Page.</span></span> <span data-ttu-id="be3f8-168">Poiché il valore dell'attributo `asp-area` è vuoto, l'area non viene usata nel collegamento.</span><span class="sxs-lookup"><span data-stu-id="be3f8-168">The `asp-area` attribute value is empty, so the area isn't used in the link.</span></span> <span data-ttu-id="be3f8-169">Per altre informazioni, vedere [Aree](xref:mvc/controllers/areas).</span><span class="sxs-lookup"><span data-stu-id="be3f8-169">See [Areas](xref:mvc/controllers/areas) for more information.</span></span>

1. <span data-ttu-id="be3f8-170">Salvare le modifiche ed eseguire il test dell'app selezionando il collegamento **RpMovie** .</span><span class="sxs-lookup"><span data-stu-id="be3f8-170">Save the changes and test the app by selecting the **RpMovie** link.</span></span> <span data-ttu-id="be3f8-171">In caso di problemi, vedere il file [_Layout.cshtml](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie30/Pages/Shared/_Layout.cshtml) in GitHub.</span><span class="sxs-lookup"><span data-stu-id="be3f8-171">See the [_Layout.cshtml](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie30/Pages/Shared/_Layout.cshtml) file in GitHub if you have any problems.</span></span>

1. <span data-ttu-id="be3f8-172">Testare i collegamenti **Home**, **RpMovie**, **create**, **Edit** e **Delete** .</span><span class="sxs-lookup"><span data-stu-id="be3f8-172">Test the **Home**, **RpMovie**, **Create**, **Edit**, and **Delete** links.</span></span> <span data-ttu-id="be3f8-173">Ogni pagina imposta il titolo, che è possibile visualizzare nella scheda del browser. Quando si aggiunge un segnalibro a una pagina, il titolo viene usato per il segnalibro.</span><span class="sxs-lookup"><span data-stu-id="be3f8-173">Each page sets the title, which you can see in the browser tab. When you bookmark a page, the title is used for the bookmark.</span></span>

> [!NOTE]
> <span data-ttu-id="be3f8-174">Potrebbe non essere possibile immettere virgole decimali nel campo `Price`.</span><span class="sxs-lookup"><span data-stu-id="be3f8-174">You may not be able to enter decimal commas in the `Price` field.</span></span> <span data-ttu-id="be3f8-175">Per supportare la [convalida jQuery](https://jqueryvalidation.org/) per le impostazioni locali diverse dall'inglese che usano una virgola (",") per un separatore decimale e formati non US-English data, è necessario eseguire le operazioni per globalizzare l'app.</span><span class="sxs-lookup"><span data-stu-id="be3f8-175">To support [jQuery validation](https://jqueryvalidation.org/) for non-English locales that use a comma (",") for a decimal point, and non US-English date formats, you must take steps to globalize the app.</span></span> <span data-ttu-id="be3f8-176">Vedere questo [problema 4076 su GitHub](https://github.com/dotnet/AspNetCore.Docs/issues/4076#issuecomment-326590420) per istruzioni sull'aggiunta della virgola decimale.</span><span class="sxs-lookup"><span data-stu-id="be3f8-176">See this [GitHub issue 4076](https://github.com/dotnet/AspNetCore.Docs/issues/4076#issuecomment-326590420) for instructions on adding decimal comma.</span></span>

<span data-ttu-id="be3f8-177">La proprietà `Layout` viene impostata nel file *Pages/_ViewStart.cshtml*:</span><span class="sxs-lookup"><span data-stu-id="be3f8-177">The `Layout` property is set in the *Pages/_ViewStart.cshtml* file:</span></span>

[!code-cshtml[](razor-pages-start/sample/RazorPagesMovie30/Pages/_ViewStart.cshtml)]

<span data-ttu-id="be3f8-178">Il markup precedente imposta il file di layout su *pages/Shared/_Layout. cshtml* per tutti Razor i file nella cartella *pages* .</span><span class="sxs-lookup"><span data-stu-id="be3f8-178">The preceding markup sets the layout file to *Pages/Shared/_Layout.cshtml* for all Razor files under the *Pages* folder.</span></span> <span data-ttu-id="be3f8-179">Vedere [Layout](xref:razor-pages/index#layout) per altre informazioni.</span><span class="sxs-lookup"><span data-stu-id="be3f8-179">See [Layout](xref:razor-pages/index#layout) for more information.</span></span>

### <a name="the-create-page-model"></a><span data-ttu-id="be3f8-180">Modello di pagina di creazione</span><span class="sxs-lookup"><span data-stu-id="be3f8-180">The Create page model</span></span>

<span data-ttu-id="be3f8-181">Esaminare il modello di pagina *Pages/Movies/Create.cshtml.cs*:</span><span class="sxs-lookup"><span data-stu-id="be3f8-181">Examine the *Pages/Movies/Create.cshtml.cs* page model:</span></span>

[!code-csharp[](razor-pages-start/snapshot_sample3/RazorPagesMovie30/Pages/Movies/Create.cshtml.cs?name=snippetALL)]

<span data-ttu-id="be3f8-182">Il metodo `OnGet` inizializza lo stato necessario per la pagina.</span><span class="sxs-lookup"><span data-stu-id="be3f8-182">The `OnGet` method initializes any state needed for the page.</span></span> <span data-ttu-id="be3f8-183">La pagina di creazione non possiede uno stato da inizializzare. Viene restituito `Page`.</span><span class="sxs-lookup"><span data-stu-id="be3f8-183">The Create page doesn't have any state to initialize, so `Page` is returned.</span></span> <span data-ttu-id="be3f8-184">Più avanti nell'esercitazione viene visualizzato un esempio di inizializzazione dello stato con `OnGet`.</span><span class="sxs-lookup"><span data-stu-id="be3f8-184">Later in the tutorial, an example of `OnGet` initializing state is shown.</span></span> <span data-ttu-id="be3f8-185">Il metodo `Page` crea un oggetto `PageResult` che esegue il rendering della pagina *Create.cshtml*.</span><span class="sxs-lookup"><span data-stu-id="be3f8-185">The `Page` method creates a `PageResult` object that renders the *Create.cshtml* page.</span></span>

<span data-ttu-id="be3f8-186">La `Movie` Proprietà usa l'attributo [[BindProperty]](xref:Microsoft.AspNetCore.Mvc.BindPropertyAttribute) per acconsentire esplicitamente all' [associazione di modelli](xref:mvc/models/model-binding).</span><span class="sxs-lookup"><span data-stu-id="be3f8-186">The `Movie` property uses the [[BindProperty]](xref:Microsoft.AspNetCore.Mvc.BindPropertyAttribute) attribute to opt-in to [model binding](xref:mvc/models/model-binding).</span></span> <span data-ttu-id="be3f8-187">Dopo che il modulo di creazione registra i valori, il runtime di ASP.NET Core associa i valori registrati al modello `Movie`.</span><span class="sxs-lookup"><span data-stu-id="be3f8-187">When the Create form posts the form values, the ASP.NET Core runtime binds the posted values to the `Movie` model.</span></span>

<span data-ttu-id="be3f8-188">Il metodo `OnPostAsync` viene eseguito quando la pagina inserisce i dati del modulo:</span><span class="sxs-lookup"><span data-stu-id="be3f8-188">The `OnPostAsync` method is run when the page posts form data:</span></span>

[!code-csharp[](razor-pages-start/snapshot_sample3/RazorPagesMovie30/Pages/Movies/Create.cshtml.cs?name=snippetPost)]

<span data-ttu-id="be3f8-189">Se il modello contiene errori, il modulo viene nuovamente visualizzato insieme ai dati del modulo inviati.</span><span class="sxs-lookup"><span data-stu-id="be3f8-189">If there are any model errors, the form is redisplayed, along with any form data posted.</span></span> <span data-ttu-id="be3f8-190">La maggior parte degli errori del modello sono riscontrabili sul lato client prima che il modulo sia inviato.</span><span class="sxs-lookup"><span data-stu-id="be3f8-190">Most model errors can be caught on the client-side before the form is posted.</span></span> <span data-ttu-id="be3f8-191">La registrazione di un valore per il campo data non convertibile in una data è un esempio di errore del modello.</span><span class="sxs-lookup"><span data-stu-id="be3f8-191">An example of a model error is posting a value for the date field that cannot be converted to a date.</span></span> <span data-ttu-id="be3f8-192">Durante l'esercitazione saranno poi trattate nel dettaglio la convalida lato client e la convalida del modello.</span><span class="sxs-lookup"><span data-stu-id="be3f8-192">Client-side validation and model validation are discussed later in the tutorial.</span></span>

<span data-ttu-id="be3f8-193">Se non sono presenti errori del modello:</span><span class="sxs-lookup"><span data-stu-id="be3f8-193">If there are no model errors:</span></span>

* <span data-ttu-id="be3f8-194">I dati vengono salvati.</span><span class="sxs-lookup"><span data-stu-id="be3f8-194">The data is saved.</span></span>
* <span data-ttu-id="be3f8-195">Il browser viene reindirizzato alla Index pagina.</span><span class="sxs-lookup"><span data-stu-id="be3f8-195">The browser is redirected to the Index page.</span></span>

### <a name="the-create-razor-page"></a><span data-ttu-id="be3f8-196">Pagina Crea Razor</span><span class="sxs-lookup"><span data-stu-id="be3f8-196">The Create Razor Page</span></span>

<span data-ttu-id="be3f8-197">Esaminare il file di paging *pages/Movies/create. cshtml* Razor :</span><span class="sxs-lookup"><span data-stu-id="be3f8-197">Examine the *Pages/Movies/Create.cshtml* Razor Page file:</span></span>

[!code-cshtml[](razor-pages-start/snapshot_sample3/RazorPagesMovie30/Pages/Movies/Create.cshtml)]

# <a name="visual-studio"></a>[<span data-ttu-id="be3f8-198">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="be3f8-198">Visual Studio</span></span>](#tab/visual-studio)

<span data-ttu-id="be3f8-199">Visual Studio visualizza i tag seguenti con un tipo di carattere in grassetto distintivo specifico per gli helper tag:</span><span class="sxs-lookup"><span data-stu-id="be3f8-199">Visual Studio displays the following tags in a distinctive bold font used for Tag Helpers:</span></span>

* `<form method="post">`
* `<div asp-validation-summary="ModelOnly" class="text-danger"></div>`
* `<label asp-for="Movie.Title" class="control-label"></label>`
* `<input asp-for="Movie.Title" class="form-control" />`
* `<span asp-validation-for="Movie.Title" class="text-danger"></span>`

![Visualizzazione di VS17 della pagina Create.cshtml](page/_static/th3.png)

# <a name="visual-studio-code--visual-studio-for-mac"></a>[<span data-ttu-id="be3f8-201">Visual Studio Code / Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="be3f8-201">Visual Studio Code / Visual Studio for Mac</span></span>](#tab/visual-studio-code+visual-studio-mac)

<span data-ttu-id="be3f8-202">Nel markup precedente sono visualizzati gli helper tag seguenti:</span><span class="sxs-lookup"><span data-stu-id="be3f8-202">The following Tag Helpers are shown in the preceding markup:</span></span>

* `<form method="post">`
* `<div asp-validation-summary="ModelOnly" class="text-danger"></div>`
* `<label asp-for="Movie.Title" class="control-label"></label>`
* `<input asp-for="Movie.Title" class="form-control" />`
* `<span asp-validation-for="Movie.Title" class="text-danger"></span>`

---

<span data-ttu-id="be3f8-203">L'elemento `<form method="post">` è un [helper tag del modulo](xref:mvc/views/working-with-forms#the-form-tag-helper).</span><span class="sxs-lookup"><span data-stu-id="be3f8-203">The `<form method="post">` element is a [Form Tag Helper](xref:mvc/views/working-with-forms#the-form-tag-helper).</span></span> <span data-ttu-id="be3f8-204">L'helper tag del modulo include automaticamente un [token antifalsificazione](xref:security/anti-request-forgery).</span><span class="sxs-lookup"><span data-stu-id="be3f8-204">The Form Tag Helper automatically includes an [antiforgery token](xref:security/anti-request-forgery).</span></span>

<span data-ttu-id="be3f8-205">Il motore di ponteggi crea Razor markup per ogni campo nel modello, ad eccezione dell'ID, simile al seguente:</span><span class="sxs-lookup"><span data-stu-id="be3f8-205">The scaffolding engine creates Razor markup for each field in the model, except the ID, similar to the following:</span></span>

[!code-cshtml[](~/tutorials/razor-pages/razor-pages-start/snapshot_sample3/RazorPagesMovie30/Pages/Movies/Create.cshtml?range=15-20)]

<span data-ttu-id="be3f8-206">Gli [Helper tag di convalida](xref:mvc/views/working-with-forms#the-validation-tag-helpers) ( `<div asp-validation-summary` e `<span asp-validation-for` ) visualizzano gli errori di convalida.</span><span class="sxs-lookup"><span data-stu-id="be3f8-206">The [Validation Tag Helpers](xref:mvc/views/working-with-forms#the-validation-tag-helpers) (`<div asp-validation-summary` and `<span asp-validation-for`) display validation errors.</span></span> <span data-ttu-id="be3f8-207">Il tema della convalida viene trattato in modo più dettagliato successivamente in questa serie.</span><span class="sxs-lookup"><span data-stu-id="be3f8-207">Validation is covered in more detail later in this series.</span></span>

<span data-ttu-id="be3f8-208">L' [Helper tag di etichetta](xref:mvc/views/working-with-forms#the-label-tag-helper) ( `<label asp-for="Movie.Title" class="control-label"></label>` ) genera la didascalia dell'etichetta e l' `[for]` attributo per la `Title` Proprietà.</span><span class="sxs-lookup"><span data-stu-id="be3f8-208">The [Label Tag Helper](xref:mvc/views/working-with-forms#the-label-tag-helper) (`<label asp-for="Movie.Title" class="control-label"></label>`) generates the label caption and `[for]` attribute for the `Title` property.</span></span>

<span data-ttu-id="be3f8-209">L' [Helper tag di input](xref:mvc/views/working-with-forms) ( `<input asp-for="Movie.Title" class="form-control">` ) USA gli attributi [DataAnnotations](/aspnet/mvc/overview/older-versions/mvc-music-store/mvc-music-store-part-6) e produce gli attributi HTML necessari per la convalida jQuery sul lato client.</span><span class="sxs-lookup"><span data-stu-id="be3f8-209">The [Input Tag Helper](xref:mvc/views/working-with-forms) (`<input asp-for="Movie.Title" class="form-control">`) uses the [DataAnnotations](/aspnet/mvc/overview/older-versions/mvc-music-store/mvc-music-store-part-6) attributes and produces HTML attributes needed for jQuery Validation on the client-side.</span></span>

<span data-ttu-id="be3f8-210">Per altre informazioni sugli helper tag, ad esempio `<form method="post">`, vedere [Helper tag in ASP.NET Core](xref:mvc/views/tag-helpers/intro).</span><span class="sxs-lookup"><span data-stu-id="be3f8-210">For more information on Tag Helpers such as `<form method="post">`, see [Tag Helpers in ASP.NET Core](xref:mvc/views/tag-helpers/intro).</span></span>

## <a name="additional-resources"></a><span data-ttu-id="be3f8-211">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="be3f8-211">Additional resources</span></span>

> [!div class="step-by-step"]
> <span data-ttu-id="be3f8-212">[Precedente: aggiungere un modello](xref:tutorials/razor-pages/model) 
>  Passaggio [successivo: utilizzo di un database](xref:tutorials/razor-pages/sql)</span><span class="sxs-lookup"><span data-stu-id="be3f8-212">[Previous: Add a model](xref:tutorials/razor-pages/model)
[Next: Work with a database](xref:tutorials/razor-pages/sql)</span></span>

::: moniker-end

::: moniker range="< aspnetcore-3.0"

## <a name="the-create-delete-details-and-edit-pages"></a><span data-ttu-id="be3f8-213">Pagine di creazione, eliminazione, dettagli e modifica</span><span class="sxs-lookup"><span data-stu-id="be3f8-213">The Create, Delete, Details, and Edit pages</span></span>

<span data-ttu-id="be3f8-214">Esaminare il modello di pagina *pages/Movies/ Index . cshtml.cs* :</span><span class="sxs-lookup"><span data-stu-id="be3f8-214">Examine the *Pages/Movies/Index.cshtml.cs* Page Model:</span></span>

[!code-csharp[](razor-pages-start/snapshot_sample/RazorPagesMovie/Pages/Movies/Index.cshtml.cs)]

<span data-ttu-id="be3f8-215">Razor Le pagine sono derivate da `PageModel` .</span><span class="sxs-lookup"><span data-stu-id="be3f8-215">Razor Pages are derived from `PageModel`.</span></span> <span data-ttu-id="be3f8-216">Per convenzione, la `PageModel` classe derivata da è denominata `<PageName>Model` .</span><span class="sxs-lookup"><span data-stu-id="be3f8-216">By convention, the `PageModel`-derived class is named `<PageName>Model`.</span></span> <span data-ttu-id="be3f8-217">Il costruttore usa l'[inserimento delle dipendenze](xref:fundamentals/dependency-injection) per aggiungere `RazorPagesMovieContext` alla pagina.</span><span class="sxs-lookup"><span data-stu-id="be3f8-217">The constructor uses [dependency injection](xref:fundamentals/dependency-injection) to add the `RazorPagesMovieContext` to the page.</span></span> <span data-ttu-id="be3f8-218">Le pagine con impalcature seguono questo modello.</span><span class="sxs-lookup"><span data-stu-id="be3f8-218">The scaffolded pages follow this pattern.</span></span> <span data-ttu-id="be3f8-219">Vedere [Codice asincrono](xref:data/ef-rp/intro#asynchronous-code) per altre informazioni sulla programmazione asincrona con Entity Framework.</span><span class="sxs-lookup"><span data-stu-id="be3f8-219">See [Asynchronous code](xref:data/ef-rp/intro#asynchronous-code) for more information on asynchronous programming with Entity Framework.</span></span>

<span data-ttu-id="be3f8-220">Quando viene effettuata una richiesta per la pagina, il `OnGetAsync` metodo restituisce un elenco di filmati alla Razor pagina.</span><span class="sxs-lookup"><span data-stu-id="be3f8-220">When a request is made for the page, the `OnGetAsync` method returns a list of movies to the Razor Page.</span></span> <span data-ttu-id="be3f8-221">`OnGetAsync` o `OnGet` viene chiamato su una Razor pagina per inizializzare lo stato della pagina.</span><span class="sxs-lookup"><span data-stu-id="be3f8-221">`OnGetAsync` or `OnGet` is called on a Razor Page to initialize the state for the page.</span></span> <span data-ttu-id="be3f8-222">In questo caso, `OnGetAsync` ottiene un elenco di filmati e li visualizza.</span><span class="sxs-lookup"><span data-stu-id="be3f8-222">In this case, `OnGetAsync` gets a list of movies and displays them.</span></span>

<span data-ttu-id="be3f8-223">Quando `OnGet` restituisce `void` o `OnGetAsync` restituisce `Task` , non viene utilizzato alcun metodo restituito.</span><span class="sxs-lookup"><span data-stu-id="be3f8-223">When `OnGet` returns `void` or `OnGetAsync` returns `Task`, no return method is used.</span></span> <span data-ttu-id="be3f8-224">Quando il tipo restituito è `IActionResult` o `Task<IActionResult>`, è necessario specificare un'istruzione return.</span><span class="sxs-lookup"><span data-stu-id="be3f8-224">When the return type is `IActionResult` or `Task<IActionResult>`, a return statement must be provided.</span></span> <span data-ttu-id="be3f8-225">Ad esempio, il metodo *Pages/Movies/Create.cshtml.cs* `OnPostAsync`:</span><span class="sxs-lookup"><span data-stu-id="be3f8-225">For example, the *Pages/Movies/Create.cshtml.cs* `OnPostAsync` method:</span></span>

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie22/Pages/Movies/Create.cshtml.cs?name=snippet)]

<a name="index"></a> <span data-ttu-id="be3f8-226">Esaminare la pagina *pages/Movies/ Index . cshtml* Razor :</span><span class="sxs-lookup"><span data-stu-id="be3f8-226">Examine the *Pages/Movies/Index.cshtml* Razor Page:</span></span>

[!code-cshtml[](razor-pages-start/snapshot_sample/RazorPagesMovie/Pages/Movies/Index.cshtml)]

<span data-ttu-id="be3f8-227">Razor consente di eseguire la transizione da HTML a C# o a un Razor markup specifico.</span><span class="sxs-lookup"><span data-stu-id="be3f8-227">Razor can transition from HTML into C# or into Razor-specific markup.</span></span> <span data-ttu-id="be3f8-228">Quando un `@` simbolo è seguito da una [ Razor parola chiave riservata](xref:mvc/views/razor#razor-reserved-keywords), esegue la transizione in un Razor markup specifico, in caso contrario passa a C#.</span><span class="sxs-lookup"><span data-stu-id="be3f8-228">When an `@` symbol is followed by a [Razor reserved keyword](xref:mvc/views/razor#razor-reserved-keywords), it transitions into Razor-specific markup, otherwise it transitions into C#.</span></span>

<span data-ttu-id="be3f8-229">La `@page` Razor direttiva trasforma il file in un'azione MVC, il che significa che può gestire le richieste.</span><span class="sxs-lookup"><span data-stu-id="be3f8-229">The `@page` Razor directive makes the file into an MVC action, which means that it can handle requests.</span></span> <span data-ttu-id="be3f8-230">`@page` deve essere la prima Razor direttiva in una pagina.</span><span class="sxs-lookup"><span data-stu-id="be3f8-230">`@page` must be the first Razor directive on a page.</span></span> <span data-ttu-id="be3f8-231">`@page` è un esempio di transizione di Razor markup specifico a.</span><span class="sxs-lookup"><span data-stu-id="be3f8-231">`@page` is an example of transitioning into Razor-specific markup.</span></span> <span data-ttu-id="be3f8-232">Per ulteriori informazioni, vedere la [ Razor sintassi](xref:mvc/views/razor#razor-syntax) .</span><span class="sxs-lookup"><span data-stu-id="be3f8-232">See [Razor syntax](xref:mvc/views/razor#razor-syntax) for more information.</span></span>

<a name="md"></a>

### <a name="the-model-directive"></a><span data-ttu-id="be3f8-233">La direttiva @model</span><span class="sxs-lookup"><span data-stu-id="be3f8-233">The @model directive</span></span>

[!code-cshtml[](razor-pages-start/snapshot_sample/RazorPagesMovie/Pages/Movies/Index.cshtml?range=1-2&highlight=2)]

<span data-ttu-id="be3f8-234">La `@model` direttiva specifica il tipo del modello passato alla Razor pagina.</span><span class="sxs-lookup"><span data-stu-id="be3f8-234">The `@model` directive specifies the type of the model passed to the Razor Page.</span></span> <span data-ttu-id="be3f8-235">Nell'esempio precedente la `@model` riga rende `PageModel` disponibile la classe derivata da per la Razor pagina.</span><span class="sxs-lookup"><span data-stu-id="be3f8-235">In the preceding example, the `@model` line makes the `PageModel`-derived class available to the Razor Page.</span></span> <span data-ttu-id="be3f8-236">Il modello viene usato negli  [helper HTML](/aspnet/mvc/overview/older-versions-1/views/creating-custom-html-helpers-cs#understanding-html-helpers)`@Html.DisplayNameFor` e `@Html.DisplayFor` nella pagina.</span><span class="sxs-lookup"><span data-stu-id="be3f8-236">The model is used in the `@Html.DisplayNameFor` and `@Html.DisplayFor` [HTML Helpers](/aspnet/mvc/overview/older-versions-1/views/creating-custom-html-helpers-cs#understanding-html-helpers) on the page.</span></span>

### <a name="html-helpers"></a><span data-ttu-id="be3f8-237">Helper HTML</span><span class="sxs-lookup"><span data-stu-id="be3f8-237">HTML Helpers</span></span>

<span data-ttu-id="be3f8-238">Esaminare l'espressione lambda usata nell'helper HTML seguente:</span><span class="sxs-lookup"><span data-stu-id="be3f8-238">Examine the lambda expression used in the following HTML Helper:</span></span>

```cshtml
@Html.DisplayNameFor(model => model.Movie[0].Title)
```

<span data-ttu-id="be3f8-239">L'helper HTML <xref:System.Web.Mvc.Html.DisplayNameExtensions.DisplayNameFor%2A?displayProperty=nameWithType> controlla la proprietà `Title` a cui fa riferimento nell'espressione lambda per determinare il nome visualizzato.</span><span class="sxs-lookup"><span data-stu-id="be3f8-239">The <xref:System.Web.Mvc.Html.DisplayNameExtensions.DisplayNameFor%2A?displayProperty=nameWithType> HTML Helper inspects the `Title` property referenced in the lambda expression to determine the display name.</span></span> <span data-ttu-id="be3f8-240">L'espressione lambda viene controllata anziché valutata.</span><span class="sxs-lookup"><span data-stu-id="be3f8-240">The lambda expression is inspected rather than evaluated.</span></span> <span data-ttu-id="be3f8-241">Non sussiste pertanto violazione di accesso quando `model`, `model.Movie`, o `model.Movie[0]` sono `null` o vuoti.</span><span class="sxs-lookup"><span data-stu-id="be3f8-241">That means there is no access violation when `model`, `model.Movie`, or `model.Movie[0]` are `null` or empty.</span></span> <span data-ttu-id="be3f8-242">Quando viene valutata l'espressione lambda, ad esempio con `@Html.DisplayFor(modelItem => item.Title)` , vengono valutati i valori delle proprietà del modello.</span><span class="sxs-lookup"><span data-stu-id="be3f8-242">When the lambda expression is evaluated, for example, with `@Html.DisplayFor(modelItem => item.Title)`, the model's property values are evaluated.</span></span>
### <a name="the-layout-page"></a><span data-ttu-id="be3f8-243">Pagina di layout</span><span class="sxs-lookup"><span data-stu-id="be3f8-243">The layout page</span></span>

<span data-ttu-id="be3f8-244">Selezionare il menu collegamenti **Razor PagesMovie**, **Home** e **privacy**.</span><span class="sxs-lookup"><span data-stu-id="be3f8-244">Select the menu links **RazorPagesMovie**, **Home**, and **Privacy**.</span></span> <span data-ttu-id="be3f8-245">Ogni pagina mostra lo stesso layout di menu.</span><span class="sxs-lookup"><span data-stu-id="be3f8-245">Each page shows the same menu layout.</span></span> <span data-ttu-id="be3f8-246">Il layout di menu viene implementato nel file *Pages/Shared/_Layout.cshtml*.</span><span class="sxs-lookup"><span data-stu-id="be3f8-246">The menu layout is implemented in the *Pages/Shared/_Layout.cshtml* file.</span></span>

<span data-ttu-id="be3f8-247">Aprire ed esaminare il file *pages/Shared/_Layout. cshtml* .</span><span class="sxs-lookup"><span data-stu-id="be3f8-247">Open and examine the *Pages/Shared/_Layout.cshtml* file.</span></span>

<span data-ttu-id="be3f8-248">I modelli di [layout](xref:mvc/views/layout) consentono di specificare il layout del contenitore HTML di un sito in un'unica posizione e quindi di applicarlo in più pagine del sito.</span><span class="sxs-lookup"><span data-stu-id="be3f8-248">[Layout](xref:mvc/views/layout) templates allow you to specify the HTML container layout of a site in one place and then apply it across multiple pages in the site.</span></span> <span data-ttu-id="be3f8-249">Trovare la riga `@RenderBody()`.</span><span class="sxs-lookup"><span data-stu-id="be3f8-249">Find the `@RenderBody()` line.</span></span> <span data-ttu-id="be3f8-250">`RenderBody` è un segnaposto dove tutte le visualizzazioni specifiche della pagina vengono presentate, *incapsulate* nella pagina di layout.</span><span class="sxs-lookup"><span data-stu-id="be3f8-250">`RenderBody` is a placeholder where all the page-specific views you create show up, *wrapped* in the layout page.</span></span> <span data-ttu-id="be3f8-251">Ad esempio, se si seleziona il collegamento **Privacy**, il rendering della visualizzazione **Pages/Privacy.cshtml** viene eseguito all'interno del metodo `RenderBody`.</span><span class="sxs-lookup"><span data-stu-id="be3f8-251">For example, if you select the **Privacy** link, the **Pages/Privacy.cshtml** view is rendered inside the `RenderBody` method.</span></span>

<a name="vd"></a>

### <a name="viewdata-and-layout"></a><span data-ttu-id="be3f8-252">ViewData e layout</span><span class="sxs-lookup"><span data-stu-id="be3f8-252">ViewData and layout</span></span>

<span data-ttu-id="be3f8-253">Si consideri il codice seguente dal file *pages/Movies/ Index . cshtml* :</span><span class="sxs-lookup"><span data-stu-id="be3f8-253">Consider the following code from the *Pages/Movies/Index.cshtml* file:</span></span>

[!code-cshtml[](razor-pages-start/snapshot_sample/RazorPagesMovie/Pages/Movies/Index.cshtml?range=1-6&highlight=4-999)]

<span data-ttu-id="be3f8-254">Il codice evidenziato precedente è un esempio di Razor transizione in C#.</span><span class="sxs-lookup"><span data-stu-id="be3f8-254">The preceding highlighted code is an example of Razor transitioning into C#.</span></span> <span data-ttu-id="be3f8-255">Tra i caratteri `{` e `}` è racchiuso un blocco di codice C#.</span><span class="sxs-lookup"><span data-stu-id="be3f8-255">The `{` and `}` characters enclose a block of C# code.</span></span>

<span data-ttu-id="be3f8-256">La classe di base `PageModel` ha una proprietà del dizionario `ViewData` che può essere usata per aggiungere i dati da passare a una visualizzazione.</span><span class="sxs-lookup"><span data-stu-id="be3f8-256">The `PageModel` base class has a `ViewData` dictionary property that can be used to add data that you want to pass to a View.</span></span> <span data-ttu-id="be3f8-257">Gli oggetti vengono aggiunti al dizionario `ViewData` usando uno schema chiave/valore.</span><span class="sxs-lookup"><span data-stu-id="be3f8-257">You add objects into the `ViewData` dictionary using a key/value pattern.</span></span> <span data-ttu-id="be3f8-258">Nell'esempio precedente la proprietà `Title` viene aggiunta al dizionario `ViewData`.</span><span class="sxs-lookup"><span data-stu-id="be3f8-258">In the preceding sample, the `Title` property is added to the `ViewData` dictionary.</span></span>

<span data-ttu-id="be3f8-259">La proprietà `Title` viene usata nel file *Pages/Shared/_Layout.cshtml*.</span><span class="sxs-lookup"><span data-stu-id="be3f8-259">The `Title` property is used in the *Pages/Shared/_Layout.cshtml* file.</span></span> <span data-ttu-id="be3f8-260">Il markup seguente illustra le prime righe del file *_Layout.cshtml*.</span><span class="sxs-lookup"><span data-stu-id="be3f8-260">The following markup shows the first few lines of the *_Layout.cshtml* file.</span></span>

<!-- We need a snapshot copy of layout because we are changing in the next step. -->

[!code-cshtml[](razor-pages-start/snapshot_sample/RazorPagesMovie/Pages/NU/_Layout.cshtml?highlight=6-99)]

<span data-ttu-id="be3f8-261">La riga `@*Markup removed for brevity.*@` è un Razor commento.</span><span class="sxs-lookup"><span data-stu-id="be3f8-261">The line `@*Markup removed for brevity.*@` is a Razor comment.</span></span> <span data-ttu-id="be3f8-262">Diversamente dai commenti HTML `<!-- -->` , Razor i commenti non vengono inviati al client.</span><span class="sxs-lookup"><span data-stu-id="be3f8-262">Unlike HTML comments `<!-- -->`, Razor comments are not sent to the client.</span></span> <span data-ttu-id="be3f8-263">Per ulteriori informazioni, vedere la pagina relativa all'introduzione [al codice HTML](https://developer.mozilla.org/docs/Learn/HTML/Introduction_to_HTML/Getting_started#HTML_comments) .</span><span class="sxs-lookup"><span data-stu-id="be3f8-263">See [MDN web docs: Getting started with HTML](https://developer.mozilla.org/docs/Learn/HTML/Introduction_to_HTML/Getting_started#HTML_comments) for more information.</span></span>

### <a name="update-the-layout"></a><span data-ttu-id="be3f8-264">Aggiornare il layout</span><span class="sxs-lookup"><span data-stu-id="be3f8-264">Update the layout</span></span>

<span data-ttu-id="be3f8-265">Modificare l' `<title>` elemento nel file *pages/Shared/_Layout. cshtml* per visualizzare il **film** anziché **Razor PagesMovie**.</span><span class="sxs-lookup"><span data-stu-id="be3f8-265">Change the `<title>` element in the *Pages/Shared/_Layout.cshtml* file to display **Movie** rather than **RazorPagesMovie**.</span></span>

[!code-cshtml[](razor-pages-start/sample/RazorPagesMovie22/Pages/Shared/_Layout.cshtml?range=1-6&highlight=6)]

<span data-ttu-id="be3f8-266">Trovare l'elemento di ancoraggio seguente nel file *Pages/Shared/_Layout.cshtml*.</span><span class="sxs-lookup"><span data-stu-id="be3f8-266">Find the following anchor element in the *Pages/Shared/_Layout.cshtml* file.</span></span>

```cshtml
<a class="navbar-brand" asp-area="" asp-page="/Index">RazorPagesMovie</a>
```

<span data-ttu-id="be3f8-267">Sostituire l'elemento precedente con il markup seguente.</span><span class="sxs-lookup"><span data-stu-id="be3f8-267">Replace the preceding element with the following markup.</span></span>

```cshtml
<a class="navbar-brand" asp-page="/Movies/Index">RpMovie</a>
```

<span data-ttu-id="be3f8-268">L'elemento di ancoraggio precedente è un [helper tag](xref:mvc/views/tag-helpers/intro).</span><span class="sxs-lookup"><span data-stu-id="be3f8-268">The preceding anchor element is a [Tag Helper](xref:mvc/views/tag-helpers/intro).</span></span> <span data-ttu-id="be3f8-269">In questo caso, si tratta dell'[helper tag di ancoraggio](xref:mvc/views/tag-helpers/builtin-th/anchor-tag-helper).</span><span class="sxs-lookup"><span data-stu-id="be3f8-269">In this case, it's the [Anchor Tag Helper](xref:mvc/views/tag-helpers/builtin-th/anchor-tag-helper).</span></span> <span data-ttu-id="be3f8-270">L' `asp-page="/Movies/Index"` attributo e il valore dell'helper Tag creano un collegamento alla `/Movies/Index` Razor pagina.</span><span class="sxs-lookup"><span data-stu-id="be3f8-270">The `asp-page="/Movies/Index"` Tag Helper attribute and value creates a link to the `/Movies/Index` Razor Page.</span></span> <span data-ttu-id="be3f8-271">Poiché il valore dell'attributo `asp-area` è vuoto, l'area non viene usata nel collegamento.</span><span class="sxs-lookup"><span data-stu-id="be3f8-271">The `asp-area` attribute value is empty, so the area isn't used in the link.</span></span> <span data-ttu-id="be3f8-272">Per altre informazioni, vedere [Aree](xref:mvc/controllers/areas).</span><span class="sxs-lookup"><span data-stu-id="be3f8-272">See [Areas](xref:mvc/controllers/areas) for more information.</span></span>

<span data-ttu-id="be3f8-273">Salvare le modifiche e testare l'app selezionando il collegamento **RpMovie**.</span><span class="sxs-lookup"><span data-stu-id="be3f8-273">Save your changes, and test the app by clicking on the **RpMovie** link.</span></span> <span data-ttu-id="be3f8-274">In caso di problemi, vedere il file [_Layout.cshtml](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie22/Pages/Shared/_Layout.cshtml) in GitHub.</span><span class="sxs-lookup"><span data-stu-id="be3f8-274">See the [_Layout.cshtml](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie22/Pages/Shared/_Layout.cshtml) file in GitHub if you have any problems.</span></span>

<span data-ttu-id="be3f8-275">Testare gli altri collegamenti (**Home**, **RpMovie**, **Create**, **Edit** e **Delete**).</span><span class="sxs-lookup"><span data-stu-id="be3f8-275">Test the other links (**Home**, **RpMovie**, **Create**, **Edit**, and **Delete**).</span></span> <span data-ttu-id="be3f8-276">Ogni pagina imposta il titolo, che è possibile visualizzare nella scheda del browser. Quando si aggiunge un segnalibro a una pagina, il titolo viene usato per il segnalibro.</span><span class="sxs-lookup"><span data-stu-id="be3f8-276">Each page sets the title, which you can see in the browser tab. When you bookmark a page, the title is used for the bookmark.</span></span>

> [!NOTE]
> <span data-ttu-id="be3f8-277">Potrebbe non essere possibile immettere virgole decimali nel campo `Price`.</span><span class="sxs-lookup"><span data-stu-id="be3f8-277">You may not be able to enter decimal commas in the `Price` field.</span></span> <span data-ttu-id="be3f8-278">Per supportare la [convalida jQuery](https://jqueryvalidation.org/) per le impostazioni locali diverse dall'inglese che usano una virgola (",") per un separatore decimale e formati non US-English data, è necessario eseguire le operazioni per globalizzare l'app.</span><span class="sxs-lookup"><span data-stu-id="be3f8-278">To support [jQuery validation](https://jqueryvalidation.org/) for non-English locales that use a comma (",") for a decimal point, and non US-English date formats, you must take steps to globalize the app.</span></span> <span data-ttu-id="be3f8-279">[Problema 4076 su GitHub](https://github.com/dotnet/AspNetCore.Docs/issues/4076#issuecomment-326590420) per istruzioni sull'aggiunta della virgola decimale.</span><span class="sxs-lookup"><span data-stu-id="be3f8-279">This [GitHub issue 4076](https://github.com/dotnet/AspNetCore.Docs/issues/4076#issuecomment-326590420) for instructions on adding decimal comma.</span></span>

<span data-ttu-id="be3f8-280">La proprietà `Layout` viene impostata nel file *Pages/_ViewStart.cshtml*:</span><span class="sxs-lookup"><span data-stu-id="be3f8-280">The `Layout` property is set in the *Pages/_ViewStart.cshtml* file:</span></span>

[!code-cshtml[](razor-pages-start/sample/RazorPagesMovie22/Pages/_ViewStart.cshtml)]

<span data-ttu-id="be3f8-281">Il markup precedente imposta il file di layout su *pages/Shared/_Layout. cshtml* per tutti Razor i file nella cartella *pages* .</span><span class="sxs-lookup"><span data-stu-id="be3f8-281">The preceding markup sets the layout file to *Pages/Shared/_Layout.cshtml* for all Razor files under the *Pages* folder.</span></span> <span data-ttu-id="be3f8-282">Vedere [Layout](xref:razor-pages/index#layout) per altre informazioni.</span><span class="sxs-lookup"><span data-stu-id="be3f8-282">See [Layout](xref:razor-pages/index#layout) for more information.</span></span>

### <a name="the-create-page-model"></a><span data-ttu-id="be3f8-283">Modello di pagina di creazione</span><span class="sxs-lookup"><span data-stu-id="be3f8-283">The Create page model</span></span>

<span data-ttu-id="be3f8-284">Esaminare il modello di pagina *Pages/Movies/Create.cshtml.cs*:</span><span class="sxs-lookup"><span data-stu-id="be3f8-284">Examine the *Pages/Movies/Create.cshtml.cs* page model:</span></span>

[!code-csharp[](razor-pages-start/snapshot_sample/RazorPagesMovie/Pages/Movies/Create.cshtml.cs?name=snippetALL)]

<span data-ttu-id="be3f8-285">Il metodo `OnGet` inizializza lo stato necessario per la pagina.</span><span class="sxs-lookup"><span data-stu-id="be3f8-285">The `OnGet` method initializes any state needed for the page.</span></span> <span data-ttu-id="be3f8-286">La pagina di creazione non possiede uno stato da inizializzare. Viene restituito `Page`.</span><span class="sxs-lookup"><span data-stu-id="be3f8-286">The Create page doesn't have any state to initialize, so `Page` is returned.</span></span> <span data-ttu-id="be3f8-287">Più avanti nell'esercitazione viene illustrato lo stato di inizializzazione del metodo `OnGet`.</span><span class="sxs-lookup"><span data-stu-id="be3f8-287">Later in the tutorial you see `OnGet` method initialize state.</span></span> <span data-ttu-id="be3f8-288">Il metodo `Page` crea un oggetto `PageResult` che esegue il rendering della pagina *Create.cshtml*.</span><span class="sxs-lookup"><span data-stu-id="be3f8-288">The `Page` method creates a `PageResult` object that renders the *Create.cshtml* page.</span></span>

<span data-ttu-id="be3f8-289">La `Movie` Proprietà usa l'attributo [[BindProperty]] <xref:Microsoft.AspNetCore.Mvc.BindPropertyAttribute> per acconsentire esplicitamente all' [associazione di modelli](xref:mvc/models/model-binding).</span><span class="sxs-lookup"><span data-stu-id="be3f8-289">The `Movie` property uses the [[BindProperty]]<xref:Microsoft.AspNetCore.Mvc.BindPropertyAttribute> attribute to opt-in to [model binding](xref:mvc/models/model-binding).</span></span> <span data-ttu-id="be3f8-290">Dopo che il modulo di creazione registra i valori, il runtime di ASP.NET Core associa i valori registrati al modello `Movie`.</span><span class="sxs-lookup"><span data-stu-id="be3f8-290">When the Create form posts the form values, the ASP.NET Core runtime binds the posted values to the `Movie` model.</span></span>

<span data-ttu-id="be3f8-291">Il metodo `OnPostAsync` viene eseguito quando la pagina inserisce i dati del modulo:</span><span class="sxs-lookup"><span data-stu-id="be3f8-291">The `OnPostAsync` method is run when the page posts form data:</span></span>

[!code-csharp[](razor-pages-start/snapshot_sample/RazorPagesMovie/Pages/Movies/Create.cshtml.cs?name=snippetPost)]

<span data-ttu-id="be3f8-292">Se il modello contiene errori, il modulo viene nuovamente visualizzato insieme ai dati del modulo inviati.</span><span class="sxs-lookup"><span data-stu-id="be3f8-292">If there are any model errors, the form is redisplayed, along with any form data posted.</span></span> <span data-ttu-id="be3f8-293">La maggior parte degli errori del modello sono riscontrabili sul lato client prima che il modulo sia inviato.</span><span class="sxs-lookup"><span data-stu-id="be3f8-293">Most model errors can be caught on the client-side before the form is posted.</span></span> <span data-ttu-id="be3f8-294">La registrazione di un valore per il campo data non convertibile in una data è un esempio di errore del modello.</span><span class="sxs-lookup"><span data-stu-id="be3f8-294">An example of a model error is posting a value for the date field that cannot be converted to a date.</span></span> <span data-ttu-id="be3f8-295">Durante l'esercitazione saranno poi trattate nel dettaglio la convalida lato client e la convalida del modello.</span><span class="sxs-lookup"><span data-stu-id="be3f8-295">Client-side validation and model validation are discussed later in the tutorial.</span></span>

<span data-ttu-id="be3f8-296">Se non sono presenti errori del modello, i dati vengono salvati e il browser viene reindirizzato alla Index pagina.</span><span class="sxs-lookup"><span data-stu-id="be3f8-296">If there are no model errors, the data is saved, and the browser is redirected to the Index page.</span></span>

### <a name="the-create-razor-page"></a><span data-ttu-id="be3f8-297">Pagina Crea Razor</span><span class="sxs-lookup"><span data-stu-id="be3f8-297">The Create Razor Page</span></span>

<span data-ttu-id="be3f8-298">Esaminare il file di paging *pages/Movies/create. cshtml* Razor :</span><span class="sxs-lookup"><span data-stu-id="be3f8-298">Examine the *Pages/Movies/Create.cshtml* Razor Page file:</span></span>

[!code-cshtml[](razor-pages-start/snapshot_sample/RazorPagesMovie/Pages/Movies/Create.cshtml)]

# <a name="visual-studio"></a>[<span data-ttu-id="be3f8-299">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="be3f8-299">Visual Studio</span></span>](#tab/visual-studio)

<span data-ttu-id="be3f8-300">Per visualizzare il tag `<form method="post">`, Visual Studio usa un tipo di carattere in grassetto distintivo specifico per gli helper tag:</span><span class="sxs-lookup"><span data-stu-id="be3f8-300">Visual Studio displays the `<form method="post">` tag in a distinctive bold font used for Tag Helpers:</span></span>

![Visualizzazione di VS17 della pagina Create.cshtml](page/_static/th.png)

# <a name="visual-studio-code"></a>[<span data-ttu-id="be3f8-302">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="be3f8-302">Visual Studio Code</span></span>](#tab/visual-studio-code)

<span data-ttu-id="be3f8-303">Per altre informazioni sugli helper tag, ad esempio `<form method="post">`, vedere [Helper tag in ASP.NET Core](xref:mvc/views/tag-helpers/intro).</span><span class="sxs-lookup"><span data-stu-id="be3f8-303">For more information on Tag Helpers such as `<form method="post">`, see [Tag Helpers in ASP.NET Core](xref:mvc/views/tag-helpers/intro).</span></span>

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="be3f8-304">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="be3f8-304">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

<span data-ttu-id="be3f8-305">Per visualizzare il tag `<form method="post">`, Visual Studio per Mac usa un tipo di carattere in grassetto distintivo specifico per gli helper tag.</span><span class="sxs-lookup"><span data-stu-id="be3f8-305">Visual Studio for Mac displays the `<form method="post">` tag in a distinctive bold font used for Tag Helpers.</span></span>

---

<span data-ttu-id="be3f8-306">L'elemento `<form method="post">` è un [helper tag del modulo](xref:mvc/views/working-with-forms#the-form-tag-helper).</span><span class="sxs-lookup"><span data-stu-id="be3f8-306">The `<form method="post">` element is a [Form Tag Helper](xref:mvc/views/working-with-forms#the-form-tag-helper).</span></span> <span data-ttu-id="be3f8-307">L'helper tag del modulo include automaticamente un [token antifalsificazione](xref:security/anti-request-forgery).</span><span class="sxs-lookup"><span data-stu-id="be3f8-307">The Form Tag Helper automatically includes an [antiforgery token](xref:security/anti-request-forgery).</span></span>

<span data-ttu-id="be3f8-308">Il motore di ponteggi crea Razor markup per ogni campo nel modello, ad eccezione dell'ID, simile al seguente:</span><span class="sxs-lookup"><span data-stu-id="be3f8-308">The scaffolding engine creates Razor markup for each field in the model, except the ID, similar to the following:</span></span>

[!code-cshtml[](~/tutorials/razor-pages/razor-pages-start/snapshot_sample/RazorPagesMovie/Pages/Movies/Create.cshtml?range=15-20)]

<span data-ttu-id="be3f8-309">Gli [Helper tag di convalida](xref:mvc/views/working-with-forms#the-validation-tag-helpers) ( `<div asp-validation-summary` e `<span asp-validation-for` ) visualizzano gli errori di convalida.</span><span class="sxs-lookup"><span data-stu-id="be3f8-309">The [Validation Tag Helpers](xref:mvc/views/working-with-forms#the-validation-tag-helpers) (`<div asp-validation-summary` and `<span asp-validation-for`) display validation errors.</span></span> <span data-ttu-id="be3f8-310">Il tema della convalida viene trattato in modo più dettagliato successivamente in questa serie.</span><span class="sxs-lookup"><span data-stu-id="be3f8-310">Validation is covered in more detail later in this series.</span></span>

<span data-ttu-id="be3f8-311">L' [Helper tag di etichetta](xref:mvc/views/working-with-forms#the-label-tag-helper) ( `<label asp-for="Movie.Title" class="control-label"></label>` ) genera la didascalia dell'etichetta e l' `[for]` attributo per la `Title` Proprietà.</span><span class="sxs-lookup"><span data-stu-id="be3f8-311">The [Label Tag Helper](xref:mvc/views/working-with-forms#the-label-tag-helper) (`<label asp-for="Movie.Title" class="control-label"></label>`) generates the label caption and `[for]` attribute for the `Title` property.</span></span>

<span data-ttu-id="be3f8-312">L' [Helper tag di input](xref:mvc/views/working-with-forms) ( `<input asp-for="Movie.Title" class="form-control">` ) USA gli attributi [DataAnnotations](/aspnet/mvc/overview/older-versions/mvc-music-store/mvc-music-store-part-6) e produce gli attributi HTML necessari per la convalida jQuery sul lato client.</span><span class="sxs-lookup"><span data-stu-id="be3f8-312">The [Input Tag Helper](xref:mvc/views/working-with-forms) (`<input asp-for="Movie.Title" class="form-control">`) uses the [DataAnnotations](/aspnet/mvc/overview/older-versions/mvc-music-store/mvc-music-store-part-6) attributes and produces HTML attributes needed for jQuery Validation on the client-side.</span></span>

## <a name="additional-resources"></a><span data-ttu-id="be3f8-313">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="be3f8-313">Additional resources</span></span>

* [<span data-ttu-id="be3f8-314">Versione YouTube dell'esercitazione</span><span class="sxs-lookup"><span data-stu-id="be3f8-314">YouTube version of this tutorial</span></span>](https://youtu.be/zxgKjPYnOMM)

> [!div class="step-by-step"]
> <span data-ttu-id="be3f8-315">[Precedente: aggiungere un modello](xref:tutorials/razor-pages/model) 
>  Passaggio [successivo: utilizzo di un database](xref:tutorials/razor-pages/sql)</span><span class="sxs-lookup"><span data-stu-id="be3f8-315">[Previous: Add a model](xref:tutorials/razor-pages/model)
[Next: Work with a database](xref:tutorials/razor-pages/sql)</span></span>

::: moniker-end
