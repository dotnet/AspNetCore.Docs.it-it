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
# <a name="part-6-add-search-to-aspnet-core-razor-pages"></a>Parte 6, aggiungere la ricerca a Razor pagine ASP.NET Core

Autore: [Rick Anderson](https://twitter.com/RickAndMSFT)

::: moniker range=">= aspnetcore-5.0"

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie50) ([procedura per il download](xref:index#how-to-download-a-sample)).

::: moniker-end

::: moniker range="< aspnetcore-5.0 >= aspnetcore-3.0"

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie30) ([procedura per il download](xref:index#how-to-download-a-sample)).

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

Nelle sezioni seguenti viene aggiunta la funzionalità di ricerca di film in base al *genere* oppure al *nome*.

Aggiungere le proprietà e l'istruzione using evidenziate seguenti a *pages/Movies/ Index . cshtml.cs*:

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie30/Pages/Movies/Index.cshtml.cs?name=snippet_newProps&highlight=3,23,24,25,26,27)]

Nel codice precedente:

* `SearchString`: Contiene il testo immesso dagli utenti nella casella di testo di ricerca. `SearchString` dispone dell' [`[BindProperty]`](xref:Microsoft.AspNetCore.Mvc.BindPropertyAttribute) attributo. `[BindProperty]` associa i valori modulo e le stringhe di query al nome della proprietà. `[BindProperty(SupportsGet = true)]` è necessario per l'associazione alle richieste HTTP GET.
* `Genres`: Contiene l'elenco dei generi. `Genres` consente all'utente di selezionare un genere dall'elenco. `SelectList` richiede `using Microsoft.AspNetCore.Mvc.Rendering;`
* `MovieGenre`: Contiene il genere specifico selezionato dall'utente. Ad esempio, "Western".
* `Genres` e `MovieGenre` sono utilizzati più avanti in questa esercitazione.

[!INCLUDE[](~/includes/bind-get.md)]

Aggiornare il Index metodo della pagina `OnGetAsync` con il codice seguente:

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie30/Pages/Movies/Index.cshtml.cs?name=snippet_1stSearch)]

La prima riga del metodo `OnGetAsync` crea una query [LINQ](/dotnet/csharp/programming-guide/concepts/linq/) per selezionare i film:

```csharp
// using System.Linq;
var movies = from m in _context.Movie
             select m;
```

A questo punto, la query è ***definita** solo con _ _ *_not_** eseguito sul database.

Se la proprietà `SearchString` non è null o vuota, la query dei film viene modificata per filtrare gli elementi in base alla stringa di ricerca:

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie30/Pages/Movies/Index.cshtml.cs?name=snippet_SearchNull)]

Il codice `s => s.Title.Contains()` è un'[espressione lambda](/dotnet/csharp/programming-guide/statements-expressions-operators/lambda-expressions). Le espressioni lambda vengono usate nelle query [LINQ](/dotnet/csharp/programming-guide/concepts/linq/) basate sul metodo come argomenti per i metodi degli operatori di query standard, ad esempio il metodo [where](/dotnet/csharp/programming-guide/concepts/linq/query-syntax-and-method-syntax-in-linq) o `Contains` . Le query LINQ non vengono eseguite quando vengono definite o quando vengono modificate chiamando un metodo, ad esempio `Where` , `Contains` o `OrderBy` . L'esecuzione della query viene invece posticipata. La valutazione di un'espressione viene posticipata fino a quando non viene eseguita l'iterazione del valore realizzato o `ToListAsync` viene chiamato il metodo. Per altre informazioni, vedere [Esecuzione di query](/dotnet/framework/data/adonet/ef/language-reference/query-execution).

> [!NOTE]
> il metodo [Contains](/dotnet/api/system.data.objects.dataclasses.entitycollection-1.contains) viene eseguito nel database, non nel codice C#. La distinzione tra maiuscole/minuscole nella query dipende dal database e dalle regole di confronto. In SQL Server `Contains` esegue il mapping a [SQL LIKE](/sql/t-sql/language-elements/like-transact-sql) che fa distinzione tra maiuscole e minuscole. In SQLite, con le regole di confronto predefinite si fa distinzione tra maiuscole e minuscole.

Passare alla pagina Movies e aggiungere una stringa di query, ad esempio `?searchString=Ghost` all'URL. Ad esempio: `https://localhost:5001/Movies?searchString=Ghost`. Vengono visualizzati i film filtrati.

![::: NO-LOC (index)::: View](search/_static/ghost.png)

Se il modello di route seguente viene aggiunto alla Index pagina, la stringa di ricerca può essere passata come segmento URL. Ad esempio: `https://localhost:5001/Movies/Ghost`.

```cshtml
@page "{searchString?}"
```

Il vincolo di route precedente consente la ricerca del titolo come dati della route (un segmento URL), anziché come valore della stringa di query.  Il carattere `?` in `"{searchString?}"` indica che questo parametro di route è facoltativo.

![::: NO-LOC (index)::: View con la parola Ghost aggiunta all'URL e un elenco di film restituito di due film, Ghostbusters e Ghostbusters 2](search/_static/g2.png)

Il runtime di ASP.NET Core usa l'[associazione di modelli](xref:mvc/models/model-binding) per impostare il valore della proprietà `SearchString` dalla stringa di query (`?searchString=Ghost`) o dai dati della route (`https://localhost:5001/Movies/Ghost`). L'associazione di modelli ***non*** distingue tra maiuscole e minuscole

Tuttavia, non è previsto che gli utenti modifichino l'URL per cercare un film. In questo passaggio viene aggiunta l'interfaccia utente per filtrare i film. Rimuovere il vincolo di route `"{searchString?}"`, se è stato aggiunto.

Aprire il file *pages/Movies/ Index . cshtml* e aggiungere il markup evidenziato nel codice seguente:

[!code-cshtml[](razor-pages-start/sample/RazorPagesMovie30/SnapShots/Index2.cshtml?highlight=14-19&range=1-22)]

Il tag HTML `<form>` usa gli [helper tag](xref:mvc/views/tag-helpers/intro) seguenti:

* [Helper tag di form](xref:mvc/views/working-with-forms#the-form-tag-helper). Quando il modulo viene inviato, la stringa di filtro viene inviata a *pagine/filmati/ Index* pagina tramite la stringa di query.
* [Helper tag di input](xref:mvc/views/working-with-forms#the-input-tag-helper)

Salvare le modifiche e testare il filtro.

![::: NO-LOC (index)::: View con la parola Ghost digitata nella casella di testo del filtro del titolo](search/_static/filter.png)

## <a name="search-by-genre"></a>Ricerca in base al genere

Aggiornare il Index metodo della pagina `OnGetAsync` con il codice seguente:

   [!code-csharp[](razor-pages-start/sample/RazorPagesMovie30/Pages/Movies/Index.cshtml.cs?name=snippet_SearchGenre)]

Il codice seguente è una query LINQ che recupera tutti i generi dal database.

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie30/Pages/Movies/Index.cshtml.cs?name=snippet_LINQ)]

L'elenco `SelectList` di generi viene creato selezionando generi distinti.

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie30/Pages/Movies/Index.cshtml.cs?name=snippet_SelectList)]

### <a name="add-search-by-genre-to-the-razor-page"></a>Aggiungi la ricerca per genere alla Razor pagina

1. Aggiornare il *Index cshtml* [ `<form>` elemento] ( https://developer.mozilla.org/docs/Web/HTML/Element/form) come evidenziato nel markup seguente:

   [!code-cshtml[](razor-pages-start/sample/RazorPagesMovie30/SnapShots/IndexFormGenreNoRating.cshtml?highlight=16-18&range=1-26)]

1. Eseguire il test dell'app effettuando una ricerca per genere, titolo del film e usando entrambi i filtri.

## <a name="additional-resources"></a>Risorse aggiuntive

> [!div class="step-by-step"]
> [Precedente: aggiornare le pagine](xref:tutorials/razor-pages/da1) 
>  Passaggio [successivo: aggiungere un nuovo campo](xref:tutorials/razor-pages/new-field)

::: moniker-end

::: moniker range="< aspnetcore-3.0"

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/razor-pages/razor-pages-start) ([procedura per il download](xref:index#how-to-download-a-sample)).

Nelle sezioni seguenti viene aggiunta la funzionalità di ricerca di film in base al *genere* oppure al *nome*.

Aggiungere le proprietà evidenziate seguenti a *pages/Movies/ Index . cshtml.cs*:

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie22/Pages/Movies/Index.cshtml.cs?name=snippet_newProps&highlight=11-999)]

* `SearchString`: Contiene il testo immesso dagli utenti nella casella di testo di ricerca. `SearchString` dispone dell' [`[BindProperty]`](xref:Microsoft.AspNetCore.Mvc.BindPropertyAttribute) attributo. `[BindProperty]` associa i valori modulo e le stringhe di query al nome della proprietà. `[BindProperty(SupportsGet = true)]` è necessario per l'associazione alle richieste HTTP GET.
* `Genres`: Contiene l'elenco dei generi. `Genres` consente all'utente di selezionare un genere dall'elenco. `SelectList` richiede `using Microsoft.AspNetCore.Mvc.Rendering;`
* `MovieGenre`: Contiene il genere specifico selezionato dall'utente. Ad esempio, "Western".
* `Genres` e `MovieGenre` sono utilizzati più avanti in questa esercitazione.

[!INCLUDE[](~/includes/bind-get.md)]

Aggiornare il Index metodo della pagina `OnGetAsync` con il codice seguente:

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie22/Pages/Movies/Index.cshtml.cs?name=snippet_1stSearch)]

La prima riga del metodo `OnGetAsync` crea una query [LINQ](/dotnet/csharp/programming-guide/concepts/linq/) per selezionare i film:

```csharp
// using System.Linq;
var movies = from m in _context.Movie
             select m;
```

La query viene *solo* definita in questo punto, ma **non** viene eseguita nel database.

Se la proprietà `SearchString` non è null o vuota, la query dei film viene modificata per filtrare gli elementi in base alla stringa di ricerca:

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie22/Pages/Movies/Index.cshtml.cs?name=snippet_SearchNull)]

Il codice `s => s.Title.Contains()` è un'[espressione lambda](/dotnet/csharp/programming-guide/statements-expressions-operators/lambda-expressions). Le espressioni lambda vengono usate nelle query [LINQ](/dotnet/csharp/programming-guide/concepts/linq/) basate sul metodo come argomenti per i metodi degli operatori di query standard, ad esempio il metodo [where](/dotnet/csharp/programming-guide/concepts/linq/query-syntax-and-method-syntax-in-linq) o `Contains` . Le query LINQ non vengono eseguite quando vengono definite o quando vengono modificate chiamando un metodo, ad esempio `Where` `Contains`  o `OrderBy` . L'esecuzione della query viene invece posticipata. La valutazione di un'espressione viene posticipata fino a quando non viene eseguita l'iterazione del valore realizzato o `ToListAsync` viene chiamato il metodo. Per altre informazioni, vedere [Esecuzione di query](/dotnet/framework/data/adonet/ef/language-reference/query-execution).

**Nota:** il metodo [Contains](/dotnet/api/system.data.objects.dataclasses.entitycollection-1.contains) viene eseguito sul database, non nel codice C#. La distinzione tra maiuscole/minuscole nella query dipende dal database e dalle regole di confronto. In SQL Server `Contains` esegue il mapping a [SQL LIKE](/sql/t-sql/language-elements/like-transact-sql) che fa distinzione tra maiuscole e minuscole. In SQLite, con le regole di confronto predefinite si fa distinzione tra maiuscole e minuscole.

Passare alla pagina Movies e aggiungere una stringa di query, ad esempio `?searchString=Ghost` all'URL. Ad esempio: `https://localhost:5001/Movies?searchString=Ghost`. Vengono visualizzati i film filtrati.

![::: NO-LOC (index)::: View](search/_static/ghost.png)

Se il modello di route seguente viene aggiunto alla Index pagina, la stringa di ricerca può essere passata come segmento URL. Ad esempio: `https://localhost:5001/Movies/Ghost`.

```cshtml
@page "{searchString?}"
```

Il vincolo di route precedente consente la ricerca del titolo come dati della route (un segmento URL), anziché come valore della stringa di query.  Il carattere `?` in `"{searchString?}"` indica che questo parametro di route è facoltativo.

![::: NO-LOC (index)::: View con la parola Ghost aggiunta all'URL e un elenco di film restituito di due film, Ghostbusters e Ghostbusters 2](search/_static/g2.png)

Il runtime di ASP.NET Core usa l'[associazione di modelli](xref:mvc/models/model-binding) per impostare il valore della proprietà `SearchString` dalla stringa di query (`?searchString=Ghost`) o dai dati della route (`https://localhost:5001/Movies/Ghost`). L'associazione di modelli ***non*** distingue tra maiuscole e minuscole

Tuttavia, non è previsto che gli utenti modifichino l'URL per cercare un film. In questo passaggio viene aggiunta l'interfaccia utente per filtrare i film. Rimuovere il vincolo di route `"{searchString?}"`, se è stato aggiunto.

Aprire il file *pages/Movies/ Index . cshtml* e aggiungere il `<form>` markup evidenziato nel codice seguente:

[!code-cshtml[](razor-pages-start/sample/RazorPagesMovie22/Pages/Movies/Index2.cshtml?highlight=14-19&range=1-22)]

Il tag HTML `<form>` usa gli [helper tag](xref:mvc/views/tag-helpers/intro) seguenti:

* [Helper tag di form](xref:mvc/views/working-with-forms#the-form-tag-helper). Quando il modulo viene inviato, la stringa di filtro viene inviata a *pagine/filmati/ Index* pagina tramite la stringa di query.
* [Helper tag di input](xref:mvc/views/working-with-forms#the-input-tag-helper)

Salvare le modifiche e testare il filtro.

![::: NO-LOC (index)::: View con la parola Ghost digitata nella casella di testo del filtro del titolo](search/_static/filter.png)

## <a name="search-by-genre"></a>Ricerca in base al genere

Aggiornare il metodo `OnGetAsync` con il codice seguente:

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie22/Pages/Movies/Index.cshtml.cs?name=snippet_SearchGenre)]

Il codice seguente è una query LINQ che recupera tutti i generi dal database.

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie22/Pages/Movies/Index.cshtml.cs?name=snippet_LINQ)]

L'elenco `SelectList` di generi viene creato selezionando generi distinti.

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie22/Pages/Movies/Index.cshtml.cs?name=snippet_SelectList)]

### <a name="add-search-by-genre-to-the-razor-page"></a>Aggiungi la ricerca per genere alla Razor pagina

Aggiornare il *Index cshtml* [ `<form>` elemento] ( https://developer.mozilla.org/docs/Web/HTML/Element/form) come evidenziato nel markup seguente:

[!code-cshtml[](razor-pages-start/sample/RazorPagesMovie22/Pages/Movies/IndexFormGenreNoRating.cshtml?highlight=16-18&range=1-26)]

Eseguire il test dell'app effettuando una ricerca per genere, titolo del film e usando entrambi i filtri.
Il codice precedente usa l'helper [tag SELECT](xref:mvc/views/working-with-forms#the-select-tag-helper) e l'helper tag option.

## <a name="additional-resources"></a>Risorse aggiuntive

* [Versione YouTube dell'esercitazione](https://youtu.be/4B6pHtdyo08)

> [!div class="step-by-step"]
> [Precedente: aggiornare le pagine](xref:tutorials/razor-pages/da1) 
>  Passaggio [successivo: aggiungere un nuovo campo](xref:tutorials/razor-pages/new-field)

::: moniker-end
