---
title: Parte 6, aggiungere la ricerca
author: rick-anderson
description: Parte 6 della serie di esercitazioni sulle Razor pagine.
ms.author: riande
ms.date: 12/05/2019
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
uid: tutorials/razor-pages/search
ms.openlocfilehash: 0ffe4ed251eed551b2799c1a66424ba31137dc18
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102589517"
---
# <a name="part-6-add-search-to-aspnet-core-razor-pages"></a><span data-ttu-id="842e4-103">Parte 6, aggiungere la ricerca a Razor pagine ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="842e4-103">Part 6, add search to ASP.NET Core Razor Pages</span></span>

<span data-ttu-id="842e4-104">Autore: [Rick Anderson](https://twitter.com/RickAndMSFT)</span><span class="sxs-lookup"><span data-stu-id="842e4-104">By [Rick Anderson](https://twitter.com/RickAndMSFT)</span></span>

::: moniker range=">= aspnetcore-5.0"

<span data-ttu-id="842e4-105">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie50) ([procedura per il download](xref:index#how-to-download-a-sample)).</span><span class="sxs-lookup"><span data-stu-id="842e4-105">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie50) ([how to download](xref:index#how-to-download-a-sample)).</span></span>

::: moniker-end

::: moniker range="< aspnetcore-5.0 >= aspnetcore-3.0"

<span data-ttu-id="842e4-106">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie30) ([procedura per il download](xref:index#how-to-download-a-sample)).</span><span class="sxs-lookup"><span data-stu-id="842e4-106">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie30) ([how to download](xref:index#how-to-download-a-sample)).</span></span>

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

<span data-ttu-id="842e4-107">Nelle sezioni seguenti viene aggiunta la funzionalità di ricerca di film in base al *genere* oppure al *nome*.</span><span class="sxs-lookup"><span data-stu-id="842e4-107">In the following sections, searching movies by *genre* or *name* is added.</span></span>

<span data-ttu-id="842e4-108">Aggiungere le proprietà e l'istruzione using evidenziate seguenti a *pages/Movies/ Index . cshtml.cs*:</span><span class="sxs-lookup"><span data-stu-id="842e4-108">Add the following highlighted using statement and properties to *Pages/Movies/Index.cshtml.cs*:</span></span>

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie30/Pages/Movies/Index.cshtml.cs?name=snippet_newProps&highlight=3,23,24,25,26,27)]

<span data-ttu-id="842e4-109">Nel codice precedente:</span><span class="sxs-lookup"><span data-stu-id="842e4-109">In the previous code:</span></span>

* <span data-ttu-id="842e4-110">`SearchString`: Contiene il testo immesso dagli utenti nella casella di testo di ricerca.</span><span class="sxs-lookup"><span data-stu-id="842e4-110">`SearchString`: Contains the text users enter in the search text box.</span></span> <span data-ttu-id="842e4-111">`SearchString` dispone dell' [`[BindProperty]`](xref:Microsoft.AspNetCore.Mvc.BindPropertyAttribute) attributo.</span><span class="sxs-lookup"><span data-stu-id="842e4-111">`SearchString` has the [`[BindProperty]`](xref:Microsoft.AspNetCore.Mvc.BindPropertyAttribute) attribute.</span></span> <span data-ttu-id="842e4-112">`[BindProperty]` associa i valori modulo e le stringhe di query al nome della proprietà.</span><span class="sxs-lookup"><span data-stu-id="842e4-112">`[BindProperty]` binds form values and query strings with the same name as the property.</span></span> <span data-ttu-id="842e4-113">`[BindProperty(SupportsGet = true)]` è necessario per l'associazione alle richieste HTTP GET.</span><span class="sxs-lookup"><span data-stu-id="842e4-113">`[BindProperty(SupportsGet = true)]` is required for binding on HTTP GET requests.</span></span>
* <span data-ttu-id="842e4-114">`Genres`: Contiene l'elenco dei generi.</span><span class="sxs-lookup"><span data-stu-id="842e4-114">`Genres`: Contains the list of genres.</span></span> <span data-ttu-id="842e4-115">`Genres` consente all'utente di selezionare un genere dall'elenco.</span><span class="sxs-lookup"><span data-stu-id="842e4-115">`Genres` allows the user to select a genre from the list.</span></span> <span data-ttu-id="842e4-116">`SelectList` richiede `using Microsoft.AspNetCore.Mvc.Rendering;`</span><span class="sxs-lookup"><span data-stu-id="842e4-116">`SelectList` requires `using Microsoft.AspNetCore.Mvc.Rendering;`</span></span>
* <span data-ttu-id="842e4-117">`MovieGenre`: Contiene il genere specifico selezionato dall'utente.</span><span class="sxs-lookup"><span data-stu-id="842e4-117">`MovieGenre`: Contains the specific genre the user selects.</span></span> <span data-ttu-id="842e4-118">Ad esempio, "Western".</span><span class="sxs-lookup"><span data-stu-id="842e4-118">For example, "Western".</span></span>
* <span data-ttu-id="842e4-119">`Genres` e `MovieGenre` sono utilizzati più avanti in questa esercitazione.</span><span class="sxs-lookup"><span data-stu-id="842e4-119">`Genres` and `MovieGenre` are used later in this tutorial.</span></span>

[!INCLUDE[](~/includes/bind-get.md)]

<span data-ttu-id="842e4-120">Aggiornare il Index metodo della pagina `OnGetAsync` con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="842e4-120">Update the Index page's `OnGetAsync` method with the following code:</span></span>

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie30/Pages/Movies/Index.cshtml.cs?name=snippet_1stSearch)]

<span data-ttu-id="842e4-121">La prima riga del metodo `OnGetAsync` crea una query [LINQ](/dotnet/csharp/programming-guide/concepts/linq/) per selezionare i film:</span><span class="sxs-lookup"><span data-stu-id="842e4-121">The first line of the `OnGetAsync` method creates a [LINQ](/dotnet/csharp/programming-guide/concepts/linq/) query to select the movies:</span></span>

```csharp
// using System.Linq;
var movies = from m in _context.Movie
             select m;
```

<span data-ttu-id="842e4-122">A questo punto, la query è ***definita** solo con _ _ *_not_** eseguito sul database.</span><span class="sxs-lookup"><span data-stu-id="842e4-122">The query is only ***defined** _ at this point, it has _ *_not_** been run against the database.</span></span>

<span data-ttu-id="842e4-123">Se la proprietà `SearchString` non è null o vuota, la query dei film viene modificata per filtrare gli elementi in base alla stringa di ricerca:</span><span class="sxs-lookup"><span data-stu-id="842e4-123">If the `SearchString` property is not null or empty, the movies query is modified to filter on the search string:</span></span>

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie30/Pages/Movies/Index.cshtml.cs?name=snippet_SearchNull)]

<span data-ttu-id="842e4-124">Il codice `s => s.Title.Contains()` è un'[espressione lambda](/dotnet/csharp/programming-guide/statements-expressions-operators/lambda-expressions).</span><span class="sxs-lookup"><span data-stu-id="842e4-124">The `s => s.Title.Contains()` code is a [Lambda Expression](/dotnet/csharp/programming-guide/statements-expressions-operators/lambda-expressions).</span></span> <span data-ttu-id="842e4-125">Le espressioni lambda vengono usate nelle query [LINQ](/dotnet/csharp/programming-guide/concepts/linq/) basate sul metodo come argomenti per i metodi degli operatori di query standard, ad esempio il metodo [where](/dotnet/csharp/programming-guide/concepts/linq/query-syntax-and-method-syntax-in-linq) o `Contains` .</span><span class="sxs-lookup"><span data-stu-id="842e4-125">Lambdas are used in method-based [LINQ](/dotnet/csharp/programming-guide/concepts/linq/) queries as arguments to standard query operator methods such as the [Where](/dotnet/csharp/programming-guide/concepts/linq/query-syntax-and-method-syntax-in-linq) method or `Contains`.</span></span> <span data-ttu-id="842e4-126">Le query LINQ non vengono eseguite quando vengono definite o quando vengono modificate chiamando un metodo, ad esempio `Where` , `Contains` o `OrderBy` .</span><span class="sxs-lookup"><span data-stu-id="842e4-126">LINQ queries are not executed when they're defined or when they're modified by calling a method, such as `Where`, `Contains`, or `OrderBy`.</span></span> <span data-ttu-id="842e4-127">L'esecuzione della query viene invece posticipata.</span><span class="sxs-lookup"><span data-stu-id="842e4-127">Rather, query execution is deferred.</span></span> <span data-ttu-id="842e4-128">La valutazione di un'espressione viene posticipata fino a quando non viene eseguita l'iterazione del valore realizzato o `ToListAsync` viene chiamato il metodo.</span><span class="sxs-lookup"><span data-stu-id="842e4-128">The evaluation of an expression is delayed until its realized value is iterated over or the `ToListAsync` method is called.</span></span> <span data-ttu-id="842e4-129">Per altre informazioni, vedere [Esecuzione di query](/dotnet/framework/data/adonet/ef/language-reference/query-execution).</span><span class="sxs-lookup"><span data-stu-id="842e4-129">See [Query Execution](/dotnet/framework/data/adonet/ef/language-reference/query-execution) for more information.</span></span>

> [!NOTE]
> <span data-ttu-id="842e4-130">il metodo [Contains](/dotnet/api/system.data.objects.dataclasses.entitycollection-1.contains) viene eseguito nel database, non nel codice C#.</span><span class="sxs-lookup"><span data-stu-id="842e4-130">The [Contains](/dotnet/api/system.data.objects.dataclasses.entitycollection-1.contains) method is run on the database, not in the C# code.</span></span> <span data-ttu-id="842e4-131">La distinzione tra maiuscole/minuscole nella query dipende dal database e dalle regole di confronto.</span><span class="sxs-lookup"><span data-stu-id="842e4-131">The case sensitivity on the query depends on the database and the collation.</span></span> <span data-ttu-id="842e4-132">In SQL Server `Contains` esegue il mapping a [SQL LIKE](/sql/t-sql/language-elements/like-transact-sql) che fa distinzione tra maiuscole e minuscole.</span><span class="sxs-lookup"><span data-stu-id="842e4-132">On SQL Server, `Contains` maps to [SQL LIKE](/sql/t-sql/language-elements/like-transact-sql), which is case insensitive.</span></span> <span data-ttu-id="842e4-133">In SQLite, con le regole di confronto predefinite si fa distinzione tra maiuscole e minuscole.</span><span class="sxs-lookup"><span data-stu-id="842e4-133">In SQLite, with the default collation, it's case sensitive.</span></span>

<span data-ttu-id="842e4-134">Passare alla pagina Movies e aggiungere una stringa di query, ad esempio `?searchString=Ghost` all'URL.</span><span class="sxs-lookup"><span data-stu-id="842e4-134">Navigate to the Movies page and append a query string such as `?searchString=Ghost` to the URL.</span></span> <span data-ttu-id="842e4-135">Ad esempio: `https://localhost:5001/Movies?searchString=Ghost`.</span><span class="sxs-lookup"><span data-stu-id="842e4-135">For example, `https://localhost:5001/Movies?searchString=Ghost`.</span></span> <span data-ttu-id="842e4-136">Vengono visualizzati i film filtrati.</span><span class="sxs-lookup"><span data-stu-id="842e4-136">The filtered movies are displayed.</span></span>

![::: NO-LOC (index)::: View](search/_static/ghost.png)

<span data-ttu-id="842e4-138">Se il modello di route seguente viene aggiunto alla Index pagina, la stringa di ricerca può essere passata come segmento URL.</span><span class="sxs-lookup"><span data-stu-id="842e4-138">If the following route template is added to the Index page, the search string can be passed as a URL segment.</span></span> <span data-ttu-id="842e4-139">Ad esempio: `https://localhost:5001/Movies/Ghost`.</span><span class="sxs-lookup"><span data-stu-id="842e4-139">For example, `https://localhost:5001/Movies/Ghost`.</span></span>

```cshtml
@page "{searchString?}"
```

<span data-ttu-id="842e4-140">Il vincolo di route precedente consente la ricerca del titolo come dati della route (un segmento URL), anziché come valore della stringa di query.</span><span class="sxs-lookup"><span data-stu-id="842e4-140">The preceding route constraint allows searching the title as route data (a URL segment) instead of as a query string value.</span></span>  <span data-ttu-id="842e4-141">Il carattere `?` in `"{searchString?}"` indica che questo parametro di route è facoltativo.</span><span class="sxs-lookup"><span data-stu-id="842e4-141">The `?` in `"{searchString?}"` means this is an optional route parameter.</span></span>

![::: NO-LOC (index)::: View con la parola Ghost aggiunta all'URL e un elenco di film restituito di due film, Ghostbusters e Ghostbusters 2](search/_static/g2.png)

<span data-ttu-id="842e4-143">Il runtime di ASP.NET Core usa l'[associazione di modelli](xref:mvc/models/model-binding) per impostare il valore della proprietà `SearchString` dalla stringa di query (`?searchString=Ghost`) o dai dati della route (`https://localhost:5001/Movies/Ghost`).</span><span class="sxs-lookup"><span data-stu-id="842e4-143">The ASP.NET Core runtime uses [model binding](xref:mvc/models/model-binding) to set the value of the `SearchString` property from the query string (`?searchString=Ghost`) or route data (`https://localhost:5001/Movies/Ghost`).</span></span> <span data-ttu-id="842e4-144">L'associazione di modelli ***non*** distingue tra maiuscole e minuscole</span><span class="sxs-lookup"><span data-stu-id="842e4-144">Model binding is ***not*** case sensitive.</span></span>

<span data-ttu-id="842e4-145">Tuttavia, non è previsto che gli utenti modifichino l'URL per cercare un film.</span><span class="sxs-lookup"><span data-stu-id="842e4-145">However, users cannot be expected to modify the URL to search for a movie.</span></span> <span data-ttu-id="842e4-146">In questo passaggio viene aggiunta l'interfaccia utente per filtrare i film.</span><span class="sxs-lookup"><span data-stu-id="842e4-146">In this step, UI is added to filter movies.</span></span> <span data-ttu-id="842e4-147">Rimuovere il vincolo di route `"{searchString?}"`, se è stato aggiunto.</span><span class="sxs-lookup"><span data-stu-id="842e4-147">If you added the route constraint `"{searchString?}"`, remove it.</span></span>

<span data-ttu-id="842e4-148">Aprire il file *pages/Movies/ Index . cshtml* e aggiungere il markup evidenziato nel codice seguente:</span><span class="sxs-lookup"><span data-stu-id="842e4-148">Open the *Pages/Movies/Index.cshtml* file, and add the markup highlighted in the following code:</span></span>

[!code-cshtml[](razor-pages-start/sample/RazorPagesMovie30/SnapShots/Index2.cshtml?highlight=14-19&range=1-22)]

<span data-ttu-id="842e4-149">Il tag HTML `<form>` usa gli [helper tag](xref:mvc/views/tag-helpers/intro) seguenti:</span><span class="sxs-lookup"><span data-stu-id="842e4-149">The HTML `<form>` tag uses the following [Tag Helpers](xref:mvc/views/tag-helpers/intro):</span></span>

* <span data-ttu-id="842e4-150">[Helper tag di form](xref:mvc/views/working-with-forms#the-form-tag-helper).</span><span class="sxs-lookup"><span data-stu-id="842e4-150">[Form Tag Helper](xref:mvc/views/working-with-forms#the-form-tag-helper).</span></span> <span data-ttu-id="842e4-151">Quando il modulo viene inviato, la stringa di filtro viene inviata a *pagine/filmati/ Index* pagina tramite la stringa di query.</span><span class="sxs-lookup"><span data-stu-id="842e4-151">When the form is submitted, the filter string is sent to the *Pages/Movies/Index* page via query string.</span></span>
* [<span data-ttu-id="842e4-152">Helper tag di input</span><span class="sxs-lookup"><span data-stu-id="842e4-152">Input Tag Helper</span></span>](xref:mvc/views/working-with-forms#the-input-tag-helper)

<span data-ttu-id="842e4-153">Salvare le modifiche e testare il filtro.</span><span class="sxs-lookup"><span data-stu-id="842e4-153">Save the changes and test the filter.</span></span>

![::: NO-LOC (index)::: View con la parola Ghost digitata nella casella di testo del filtro del titolo](search/_static/filter.png)

## <a name="search-by-genre"></a><span data-ttu-id="842e4-155">Ricerca in base al genere</span><span class="sxs-lookup"><span data-stu-id="842e4-155">Search by genre</span></span>

<span data-ttu-id="842e4-156">Aggiornare il Index metodo della pagina `OnGetAsync` con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="842e4-156">Update the Index page's `OnGetAsync` method with the following code:</span></span>

   [!code-csharp[](razor-pages-start/sample/RazorPagesMovie30/Pages/Movies/Index.cshtml.cs?name=snippet_SearchGenre)]

<span data-ttu-id="842e4-157">Il codice seguente è una query LINQ che recupera tutti i generi dal database.</span><span class="sxs-lookup"><span data-stu-id="842e4-157">The following code is a LINQ query that retrieves all the genres from the database.</span></span>

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie30/Pages/Movies/Index.cshtml.cs?name=snippet_LINQ)]

<span data-ttu-id="842e4-158">L'elenco `SelectList` di generi viene creato selezionando generi distinti.</span><span class="sxs-lookup"><span data-stu-id="842e4-158">The `SelectList` of genres is created by projecting the distinct genres.</span></span>

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie30/Pages/Movies/Index.cshtml.cs?name=snippet_SelectList)]

### <a name="add-search-by-genre-to-the-razor-page"></a><span data-ttu-id="842e4-159">Aggiungi la ricerca per genere alla Razor pagina</span><span class="sxs-lookup"><span data-stu-id="842e4-159">Add search by genre to the Razor Page</span></span>

1. <span data-ttu-id="842e4-160">Aggiornare il *Index cshtml* [ `<form>` elemento] ( https://developer.mozilla.org/docs/Web/HTML/Element/form) come evidenziato nel markup seguente:</span><span class="sxs-lookup"><span data-stu-id="842e4-160">Update the *Index.cshtml* [`<form>` element] (https://developer.mozilla.org/docs/Web/HTML/Element/form) as highlighted in the following markup:</span></span>

   [!code-cshtml[](razor-pages-start/sample/RazorPagesMovie30/SnapShots/IndexFormGenreNoRating.cshtml?highlight=16-18&range=1-26)]

1. <span data-ttu-id="842e4-161">Eseguire il test dell'app effettuando una ricerca per genere, titolo del film e usando entrambi i filtri.</span><span class="sxs-lookup"><span data-stu-id="842e4-161">Test the app by searching by genre, by movie title, and by both.</span></span>

## <a name="additional-resources"></a><span data-ttu-id="842e4-162">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="842e4-162">Additional resources</span></span>

> [!div class="step-by-step"]
> <span data-ttu-id="842e4-163">[Precedente: aggiornare le pagine](xref:tutorials/razor-pages/da1) 
>  Passaggio [successivo: aggiungere un nuovo campo](xref:tutorials/razor-pages/new-field)</span><span class="sxs-lookup"><span data-stu-id="842e4-163">[Previous: Update the pages](xref:tutorials/razor-pages/da1)
[Next: Add a new field](xref:tutorials/razor-pages/new-field)</span></span>

::: moniker-end

::: moniker range="< aspnetcore-3.0"

<span data-ttu-id="842e4-164">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/razor-pages/razor-pages-start) ([procedura per il download](xref:index#how-to-download-a-sample)).</span><span class="sxs-lookup"><span data-stu-id="842e4-164">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/razor-pages/razor-pages-start) ([how to download](xref:index#how-to-download-a-sample)).</span></span>

<span data-ttu-id="842e4-165">Nelle sezioni seguenti viene aggiunta la funzionalità di ricerca di film in base al *genere* oppure al *nome*.</span><span class="sxs-lookup"><span data-stu-id="842e4-165">In the following sections, searching movies by *genre* or *name* is added.</span></span>

<span data-ttu-id="842e4-166">Aggiungere le proprietà evidenziate seguenti a *pages/Movies/ Index . cshtml.cs*:</span><span class="sxs-lookup"><span data-stu-id="842e4-166">Add the following highlighted properties to *Pages/Movies/Index.cshtml.cs*:</span></span>

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie22/Pages/Movies/Index.cshtml.cs?name=snippet_newProps&highlight=11-999)]

* <span data-ttu-id="842e4-167">`SearchString`: Contiene il testo immesso dagli utenti nella casella di testo di ricerca.</span><span class="sxs-lookup"><span data-stu-id="842e4-167">`SearchString`: Contains the text users enter in the search text box.</span></span> <span data-ttu-id="842e4-168">`SearchString` dispone dell' [`[BindProperty]`](xref:Microsoft.AspNetCore.Mvc.BindPropertyAttribute) attributo.</span><span class="sxs-lookup"><span data-stu-id="842e4-168">`SearchString` has the [`[BindProperty]`](xref:Microsoft.AspNetCore.Mvc.BindPropertyAttribute) attribute.</span></span> <span data-ttu-id="842e4-169">`[BindProperty]` associa i valori modulo e le stringhe di query al nome della proprietà.</span><span class="sxs-lookup"><span data-stu-id="842e4-169">`[BindProperty]` binds form values and query strings with the same name as the property.</span></span> <span data-ttu-id="842e4-170">`[BindProperty(SupportsGet = true)]` è necessario per l'associazione alle richieste HTTP GET.</span><span class="sxs-lookup"><span data-stu-id="842e4-170">`[BindProperty(SupportsGet = true)]` is required for binding on HTTP GET requests.</span></span>
* <span data-ttu-id="842e4-171">`Genres`: Contiene l'elenco dei generi.</span><span class="sxs-lookup"><span data-stu-id="842e4-171">`Genres`: Contains the list of genres.</span></span> <span data-ttu-id="842e4-172">`Genres` consente all'utente di selezionare un genere dall'elenco.</span><span class="sxs-lookup"><span data-stu-id="842e4-172">`Genres` allows the user to select a genre from the list.</span></span> <span data-ttu-id="842e4-173">`SelectList` richiede `using Microsoft.AspNetCore.Mvc.Rendering;`</span><span class="sxs-lookup"><span data-stu-id="842e4-173">`SelectList` requires `using Microsoft.AspNetCore.Mvc.Rendering;`</span></span>
* <span data-ttu-id="842e4-174">`MovieGenre`: Contiene il genere specifico selezionato dall'utente.</span><span class="sxs-lookup"><span data-stu-id="842e4-174">`MovieGenre`: Contains the specific genre the user selects.</span></span> <span data-ttu-id="842e4-175">Ad esempio, "Western".</span><span class="sxs-lookup"><span data-stu-id="842e4-175">For example, "Western".</span></span>
* <span data-ttu-id="842e4-176">`Genres` e `MovieGenre` sono utilizzati più avanti in questa esercitazione.</span><span class="sxs-lookup"><span data-stu-id="842e4-176">`Genres` and `MovieGenre` are used later in this tutorial.</span></span>

[!INCLUDE[](~/includes/bind-get.md)]

<span data-ttu-id="842e4-177">Aggiornare il Index metodo della pagina `OnGetAsync` con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="842e4-177">Update the Index page's `OnGetAsync` method with the following code:</span></span>

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie22/Pages/Movies/Index.cshtml.cs?name=snippet_1stSearch)]

<span data-ttu-id="842e4-178">La prima riga del metodo `OnGetAsync` crea una query [LINQ](/dotnet/csharp/programming-guide/concepts/linq/) per selezionare i film:</span><span class="sxs-lookup"><span data-stu-id="842e4-178">The first line of the `OnGetAsync` method creates a [LINQ](/dotnet/csharp/programming-guide/concepts/linq/) query to select the movies:</span></span>

```csharp
// using System.Linq;
var movies = from m in _context.Movie
             select m;
```

<span data-ttu-id="842e4-179">La query viene *solo* definita in questo punto, ma **non** viene eseguita nel database.</span><span class="sxs-lookup"><span data-stu-id="842e4-179">The query is *only* defined at this point, it has **not** been run against the database.</span></span>

<span data-ttu-id="842e4-180">Se la proprietà `SearchString` non è null o vuota, la query dei film viene modificata per filtrare gli elementi in base alla stringa di ricerca:</span><span class="sxs-lookup"><span data-stu-id="842e4-180">If the `SearchString` property is not null or empty, the movies query is modified to filter on the search string:</span></span>

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie22/Pages/Movies/Index.cshtml.cs?name=snippet_SearchNull)]

<span data-ttu-id="842e4-181">Il codice `s => s.Title.Contains()` è un'[espressione lambda](/dotnet/csharp/programming-guide/statements-expressions-operators/lambda-expressions).</span><span class="sxs-lookup"><span data-stu-id="842e4-181">The `s => s.Title.Contains()` code is a [Lambda Expression](/dotnet/csharp/programming-guide/statements-expressions-operators/lambda-expressions).</span></span> <span data-ttu-id="842e4-182">Le espressioni lambda vengono usate nelle query [LINQ](/dotnet/csharp/programming-guide/concepts/linq/) basate sul metodo come argomenti per i metodi degli operatori di query standard, ad esempio il metodo [where](/dotnet/csharp/programming-guide/concepts/linq/query-syntax-and-method-syntax-in-linq) o `Contains` .</span><span class="sxs-lookup"><span data-stu-id="842e4-182">Lambdas are used in method-based [LINQ](/dotnet/csharp/programming-guide/concepts/linq/) queries as arguments to standard query operator methods such as the [Where](/dotnet/csharp/programming-guide/concepts/linq/query-syntax-and-method-syntax-in-linq) method or `Contains`.</span></span> <span data-ttu-id="842e4-183">Le query LINQ non vengono eseguite quando vengono definite o quando vengono modificate chiamando un metodo, ad esempio `Where` `Contains`  o `OrderBy` .</span><span class="sxs-lookup"><span data-stu-id="842e4-183">LINQ queries are not executed when they're defined or when they're modified by calling a method, such as `Where`, `Contains`  or `OrderBy`.</span></span> <span data-ttu-id="842e4-184">L'esecuzione della query viene invece posticipata.</span><span class="sxs-lookup"><span data-stu-id="842e4-184">Rather, query execution is deferred.</span></span> <span data-ttu-id="842e4-185">La valutazione di un'espressione viene posticipata fino a quando non viene eseguita l'iterazione del valore realizzato o `ToListAsync` viene chiamato il metodo.</span><span class="sxs-lookup"><span data-stu-id="842e4-185">The evaluation of an expression is delayed until its realized value is iterated over or the `ToListAsync` method is called.</span></span> <span data-ttu-id="842e4-186">Per altre informazioni, vedere [Esecuzione di query](/dotnet/framework/data/adonet/ef/language-reference/query-execution).</span><span class="sxs-lookup"><span data-stu-id="842e4-186">See [Query Execution](/dotnet/framework/data/adonet/ef/language-reference/query-execution) for more information.</span></span>

<span data-ttu-id="842e4-187">**Nota:** il metodo [Contains](/dotnet/api/system.data.objects.dataclasses.entitycollection-1.contains) viene eseguito sul database, non nel codice C#.</span><span class="sxs-lookup"><span data-stu-id="842e4-187">**Note:** The [Contains](/dotnet/api/system.data.objects.dataclasses.entitycollection-1.contains) method is run on the database, not in the C# code.</span></span> <span data-ttu-id="842e4-188">La distinzione tra maiuscole/minuscole nella query dipende dal database e dalle regole di confronto.</span><span class="sxs-lookup"><span data-stu-id="842e4-188">The case sensitivity on the query depends on the database and the collation.</span></span> <span data-ttu-id="842e4-189">In SQL Server `Contains` esegue il mapping a [SQL LIKE](/sql/t-sql/language-elements/like-transact-sql) che fa distinzione tra maiuscole e minuscole.</span><span class="sxs-lookup"><span data-stu-id="842e4-189">On SQL Server, `Contains` maps to [SQL LIKE](/sql/t-sql/language-elements/like-transact-sql), which is case insensitive.</span></span> <span data-ttu-id="842e4-190">In SQLite, con le regole di confronto predefinite si fa distinzione tra maiuscole e minuscole.</span><span class="sxs-lookup"><span data-stu-id="842e4-190">In SQLite, with the default collation, it's case sensitive.</span></span>

<span data-ttu-id="842e4-191">Passare alla pagina Movies e aggiungere una stringa di query, ad esempio `?searchString=Ghost` all'URL.</span><span class="sxs-lookup"><span data-stu-id="842e4-191">Navigate to the Movies page and append a query string such as `?searchString=Ghost` to the URL.</span></span> <span data-ttu-id="842e4-192">Ad esempio: `https://localhost:5001/Movies?searchString=Ghost`.</span><span class="sxs-lookup"><span data-stu-id="842e4-192">For example, `https://localhost:5001/Movies?searchString=Ghost`.</span></span> <span data-ttu-id="842e4-193">Vengono visualizzati i film filtrati.</span><span class="sxs-lookup"><span data-stu-id="842e4-193">The filtered movies are displayed.</span></span>

![::: NO-LOC (index)::: View](search/_static/ghost.png)

<span data-ttu-id="842e4-195">Se il modello di route seguente viene aggiunto alla Index pagina, la stringa di ricerca può essere passata come segmento URL.</span><span class="sxs-lookup"><span data-stu-id="842e4-195">If the following route template is added to the Index page, the search string can be passed as a URL segment.</span></span> <span data-ttu-id="842e4-196">Ad esempio: `https://localhost:5001/Movies/Ghost`.</span><span class="sxs-lookup"><span data-stu-id="842e4-196">For example, `https://localhost:5001/Movies/Ghost`.</span></span>

```cshtml
@page "{searchString?}"
```

<span data-ttu-id="842e4-197">Il vincolo di route precedente consente la ricerca del titolo come dati della route (un segmento URL), anziché come valore della stringa di query.</span><span class="sxs-lookup"><span data-stu-id="842e4-197">The preceding route constraint allows searching the title as route data (a URL segment) instead of as a query string value.</span></span>  <span data-ttu-id="842e4-198">Il carattere `?` in `"{searchString?}"` indica che questo parametro di route è facoltativo.</span><span class="sxs-lookup"><span data-stu-id="842e4-198">The `?` in `"{searchString?}"` means this is an optional route parameter.</span></span>

![::: NO-LOC (index)::: View con la parola Ghost aggiunta all'URL e un elenco di film restituito di due film, Ghostbusters e Ghostbusters 2](search/_static/g2.png)

<span data-ttu-id="842e4-200">Il runtime di ASP.NET Core usa l'[associazione di modelli](xref:mvc/models/model-binding) per impostare il valore della proprietà `SearchString` dalla stringa di query (`?searchString=Ghost`) o dai dati della route (`https://localhost:5001/Movies/Ghost`).</span><span class="sxs-lookup"><span data-stu-id="842e4-200">The ASP.NET Core runtime uses [model binding](xref:mvc/models/model-binding) to set the value of the `SearchString` property from the query string (`?searchString=Ghost`) or route data (`https://localhost:5001/Movies/Ghost`).</span></span> <span data-ttu-id="842e4-201">L'associazione di modelli ***non*** distingue tra maiuscole e minuscole</span><span class="sxs-lookup"><span data-stu-id="842e4-201">Model binding is ***not*** case sensitive.</span></span>

<span data-ttu-id="842e4-202">Tuttavia, non è previsto che gli utenti modifichino l'URL per cercare un film.</span><span class="sxs-lookup"><span data-stu-id="842e4-202">However, users can't be expected to modify the URL to search for a movie.</span></span> <span data-ttu-id="842e4-203">In questo passaggio viene aggiunta l'interfaccia utente per filtrare i film.</span><span class="sxs-lookup"><span data-stu-id="842e4-203">In this step, UI is added to filter movies.</span></span> <span data-ttu-id="842e4-204">Rimuovere il vincolo di route `"{searchString?}"`, se è stato aggiunto.</span><span class="sxs-lookup"><span data-stu-id="842e4-204">If you added the route constraint `"{searchString?}"`, remove it.</span></span>

<span data-ttu-id="842e4-205">Aprire il file *pages/Movies/ Index . cshtml* e aggiungere il `<form>` markup evidenziato nel codice seguente:</span><span class="sxs-lookup"><span data-stu-id="842e4-205">Open the *Pages/Movies/Index.cshtml* file, and add the `<form>` markup highlighted in the following code:</span></span>

[!code-cshtml[](razor-pages-start/sample/RazorPagesMovie22/Pages/Movies/Index2.cshtml?highlight=14-19&range=1-22)]

<span data-ttu-id="842e4-206">Il tag HTML `<form>` usa gli [helper tag](xref:mvc/views/tag-helpers/intro) seguenti:</span><span class="sxs-lookup"><span data-stu-id="842e4-206">The HTML `<form>` tag uses the following [Tag Helpers](xref:mvc/views/tag-helpers/intro):</span></span>

* <span data-ttu-id="842e4-207">[Helper tag di form](xref:mvc/views/working-with-forms#the-form-tag-helper).</span><span class="sxs-lookup"><span data-stu-id="842e4-207">[Form Tag Helper](xref:mvc/views/working-with-forms#the-form-tag-helper).</span></span> <span data-ttu-id="842e4-208">Quando il modulo viene inviato, la stringa di filtro viene inviata a *pagine/filmati/ Index* pagina tramite la stringa di query.</span><span class="sxs-lookup"><span data-stu-id="842e4-208">When the form is submitted, the filter string is sent to the *Pages/Movies/Index* page via query string.</span></span>
* [<span data-ttu-id="842e4-209">Helper tag di input</span><span class="sxs-lookup"><span data-stu-id="842e4-209">Input Tag Helper</span></span>](xref:mvc/views/working-with-forms#the-input-tag-helper)

<span data-ttu-id="842e4-210">Salvare le modifiche e testare il filtro.</span><span class="sxs-lookup"><span data-stu-id="842e4-210">Save the changes and test the filter.</span></span>

![::: NO-LOC (index)::: View con la parola Ghost digitata nella casella di testo del filtro del titolo](search/_static/filter.png)

## <a name="search-by-genre"></a><span data-ttu-id="842e4-212">Ricerca in base al genere</span><span class="sxs-lookup"><span data-stu-id="842e4-212">Search by genre</span></span>

<span data-ttu-id="842e4-213">Aggiornare il metodo `OnGetAsync` con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="842e4-213">Update the `OnGetAsync` method with the following code:</span></span>

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie22/Pages/Movies/Index.cshtml.cs?name=snippet_SearchGenre)]

<span data-ttu-id="842e4-214">Il codice seguente è una query LINQ che recupera tutti i generi dal database.</span><span class="sxs-lookup"><span data-stu-id="842e4-214">The following code is a LINQ query that retrieves all the genres from the database.</span></span>

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie22/Pages/Movies/Index.cshtml.cs?name=snippet_LINQ)]

<span data-ttu-id="842e4-215">L'elenco `SelectList` di generi viene creato selezionando generi distinti.</span><span class="sxs-lookup"><span data-stu-id="842e4-215">The `SelectList` of genres is created by projecting the distinct genres.</span></span>

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie22/Pages/Movies/Index.cshtml.cs?name=snippet_SelectList)]

### <a name="add-search-by-genre-to-the-razor-page"></a><span data-ttu-id="842e4-216">Aggiungi la ricerca per genere alla Razor pagina</span><span class="sxs-lookup"><span data-stu-id="842e4-216">Add search by genre to the Razor Page</span></span>

<span data-ttu-id="842e4-217">Aggiornare il *Index cshtml* [ `<form>` elemento] ( https://developer.mozilla.org/docs/Web/HTML/Element/form) come evidenziato nel markup seguente:</span><span class="sxs-lookup"><span data-stu-id="842e4-217">Update the *Index.cshtml* [`<form>` element] (https://developer.mozilla.org/docs/Web/HTML/Element/form) as highlighted in the following markup:</span></span>

[!code-cshtml[](razor-pages-start/sample/RazorPagesMovie22/Pages/Movies/IndexFormGenreNoRating.cshtml?highlight=16-18&range=1-26)]

<span data-ttu-id="842e4-218">Eseguire il test dell'app effettuando una ricerca per genere, titolo del film e usando entrambi i filtri.</span><span class="sxs-lookup"><span data-stu-id="842e4-218">Test the app by searching by genre, by movie title, and by both.</span></span>
<span data-ttu-id="842e4-219">Il codice precedente usa l'helper [tag SELECT](xref:mvc/views/working-with-forms#the-select-tag-helper) e l'helper tag option.</span><span class="sxs-lookup"><span data-stu-id="842e4-219">The preceding code uses the [Select Tag Helper](xref:mvc/views/working-with-forms#the-select-tag-helper) and Option Tag Helper.</span></span>

## <a name="additional-resources"></a><span data-ttu-id="842e4-220">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="842e4-220">Additional resources</span></span>

* [<span data-ttu-id="842e4-221">Versione YouTube dell'esercitazione</span><span class="sxs-lookup"><span data-stu-id="842e4-221">YouTube version of this tutorial</span></span>](https://youtu.be/4B6pHtdyo08)

> [!div class="step-by-step"]
> <span data-ttu-id="842e4-222">[Precedente: aggiornare le pagine](xref:tutorials/razor-pages/da1) 
>  Passaggio [successivo: aggiungere un nuovo campo](xref:tutorials/razor-pages/new-field)</span><span class="sxs-lookup"><span data-stu-id="842e4-222">[Previous: Update the pages](xref:tutorials/razor-pages/da1)
[Next: Add a new field](xref:tutorials/razor-pages/new-field)</span></span>

::: moniker-end
