---
title: 'Esercitazione: aggiungere ordinamento, filtro e paging-ASP.NET MVC con EF Core'
description: In questa esercitazione si aggiungeranno le funzionalità di ordinamento, filtro e suddivisione in pagine alla pagina Student Index (Indice degli studenti). Verrà anche creata una pagina che esegue il raggruppamento semplice.
author: rick-anderson
ms.author: riande
ms.date: 03/27/2019
ms.topic: tutorial
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
uid: data/ef-mvc/sort-filter-page
ms.openlocfilehash: ca4bce2274233f49e1c382dbd8d5c45615006b9c
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102587645"
---
# <a name="tutorial-add-sorting-filtering-and-paging---aspnet-mvc-with-ef-core"></a>Esercitazione: aggiungere ordinamento, filtro e paging-ASP.NET MVC con EF Core

Nell'esercitazione precedente è stato implementato un set di pagine Web per operazioni CRUD di base per le entità Student. In questa esercitazione si aggiungeranno le funzionalità di ordinamento, filtro e suddivisione in pagine alla pagina Student Index (Indice degli studenti). Verrà anche creata una pagina che esegue il raggruppamento semplice.

La figura seguente illustra l'aspetto della pagina al termine dell'operazione. Le intestazioni di colonna sono collegamenti su cui l'utente può fare clic per eseguire l'ordinamento in base alla colonna. Facendo clic più volte su un'intestazione di colonna è possibile passare dall'ordinamento crescente a quello decrescente e viceversa.

![Pagina Student Index (Indice degli studenti)](sort-filter-page/_static/paging.png)

In questa esercitazione:

> [!div class="checklist"]
> * Aggiungere collegamenti per l'ordinamento delle colonne
> * Aggiungere una casella di ricerca
> * Aggiungere la suddivisione in pagine per Student Index
> * Aggiungere la suddivisione in pagine al metodo Index
> * Aggiungere collegamenti per la suddivisione in pagine
> * Creare una pagina About

## <a name="prerequisites"></a>Prerequisiti

* [Implementare la funzionalità CRUD](crud.md)

## <a name="add-column-sort-links"></a>Aggiungere collegamenti per l'ordinamento delle colonne

Per aggiungere l'ordinamento alla pagina Student Index (Indice degli studenti), sarà necessario modificare il metodo `Index` del controller Students e aggiungere codice alla visualizzazione Student Index (Indice degli studenti).

### <a name="add-sorting-functionality-to-the-index-method"></a>Aggiungere la funzionalità di ordinamento al metodo Index

In *StudentsController.cs* sostituire il `Index` metodo con il codice seguente:

[!code-csharp[](intro/samples/cu/Controllers/StudentsController.cs?name=snippet_SortOnly)]

Questo codice riceve un parametro `sortOrder` dalla stringa di query nell'URL. Il valore della stringa di query viene inviato da ASP.NET Core MVC come parametro al metodo di azione. Il parametro sarà una stringa "Name" o "Date", seguito facoltativamente da un carattere di sottolineatura e dalla stringa "desc" per specificare l'ordine decrescente. Per impostazione predefinita, l'ordinamento è crescente.

La prima volta che viene richiesta la pagina di indice, non è presente alcuna stringa di query. Gli studenti vengono visualizzati in ordine crescente in base al cognome, che è il valore predefinito determinato dal caso di fallthrough nell'istruzione `switch`. Quando l'utente fa clic sul collegamento ipertestuale di un'intestazione di colonna, nella stringa di query viene specificato il valore `sortOrder` appropriato.

I due elementi `ViewData` (NameSortParm e DateSortParm) vengono usati dalla visualizzazione per configurare i collegamenti ipertestuali dell'intestazione di colonna con i valori della stringa di query appropriata.

[!code-csharp[](intro/samples/cu/Controllers/StudentsController.cs?name=snippet_SortOnly&highlight=3-4)]

Si tratta di istruzioni ternarie. La prima specifica che, se il parametro `sortOrder` è Null o vuoto, NameSortParm deve essere impostato su "name_desc"; in caso contrario, deve essere impostato su una stringa vuota. Queste due istruzioni consentono alla visualizzazione di impostare i collegamenti ipertestuali dell'intestazione di colonna come indicato di seguito:

|  Ordinamento corrente  | Collegamento ipertestuale cognome | Collegamento ipertestuale data |
|:--------------------:|:-------------------:|:--------------:|
| Cognome in ordine crescente  | descending          | ascending      |
| Cognome in ordine decrescente | ascending           | ascending      |
| Data in ordine crescente       | ascending           | descending     |
| Data in ordine decrescente      | ascending           | ascending      |

Il metodo usa LINQ to Entities per specificare la colonna in base alla quale eseguire l'ordinamento. Il codice crea una variabile `IQueryable` prima dell'istruzione switch, la modifica nell'istruzione switch e chiama il metodo `ToListAsync` dopo l'istruzione `switch`. Quando si creano e modificano variabili `IQueryable`, nessuna query viene inviata al database. La query non viene eseguita finché l'oggetto `IQueryable` non viene convertito in una raccolta chiamando un metodo, ad esempio `ToListAsync`. Questo codice genera pertanto una singola query che non viene eseguita fino all'istruzione `return View`.

Questo codice può essere reso dettagliato con un numero elevato di colonne. Nell'[ultima esercitazione di questa serie](advanced.md#dynamic-linq) viene illustrato come scrivere codice che consente di passare il nome della colonna `OrderBy` in una variabile di stringa.

### <a name="add-column-heading-hyperlinks-to-the-student-index-view"></a>Aggiungere collegamenti ipertestuali delle intestazioni di colonna alla visualizzazione Student Index (Indice degli studenti)

Sostituire il codice in *Views/Students/Index.cshtml* con il codice seguente per aggiungere collegamenti ipertestuali delle intestazioni di colonna. Le righe modificate sono evidenziate.

[!code-cshtml[](intro/samples/cu/Views/Students/Index2.cshtml?highlight=16,22)]

Questo codice usa le informazioni contenute nelle proprietà `ViewData` per impostare i collegamenti ipertestuali con i valori della stringa di query appropriati.

Eseguire l'app, selezionare la scheda **Students** (Studenti) e fare clic sulle intestazioni di colonna **Last Name** (Cognome) e **Enrollment Date** (Data di iscrizione) per verificare che l'ordinamento funzioni correttamente.

![Pagina Student Index (Indice degli studenti) in ordine di nome](sort-filter-page/_static/name-order.png)

## <a name="add-a-search-box"></a>Aggiungere una casella di ricerca

Per aggiungere filtri alla pagina Student Index (Indice degli studenti), è necessario aggiungere alla visualizzazione una casella di testo e un pulsante di invio e apportare le modifiche corrispondenti nel metodo `Index`. La casella di testo consente di immettere una stringa per eseguire la ricerca nei campi di nome e cognome.

### <a name="add-filtering-functionality-to-the-index-method"></a>Aggiungere la funzionalità di filtro al metodo Index

In *StudentsController.cs* sostituire il metodo `Index` con il codice seguente (le modifiche sono evidenziate).

[!code-csharp[](intro/samples/cu/Controllers/StudentsController.cs?name=snippet_SortFilter&highlight=1,5,9-13)]

È stato aggiunto un parametro `searchString` al metodo `Index`. Il valore della stringa di ricerca viene ricevuto da una casella di testo che verrà aggiunta alla visualizzazione Index (Indice). È stata anche aggiunta all'istruzione LINQ una clausola where che seleziona solo gli studenti il cui nome o cognome contiene la stringa di ricerca. L'istruzione che aggiunge la clausola where viene eseguita solo se è presente un valore per la ricerca.

> [!NOTE]
> In questo esempio si chiama il metodo `Where` su un oggetto `IQueryable` e il filtro verrà elaborato nel server. In alcuni scenari potrebbe essere chiamato il metodo `Where` come metodo di estensione per una raccolta in memoria. Si supponga, ad esempio, di modificare il riferimento a in `_context.Students` modo che invece di un EF `DbSet` faccia riferimento a un metodo di repository che restituisce una `IEnumerable` raccolta. Il risultato sarebbe normalmente lo stesso, ma in alcuni casi può essere diverso.
>
>Ad esempio, l'implementazione di .NET Framework del metodo `Contains` esegue un confronto con la distinzione tra maiuscole e minuscole per impostazione predefinita, ma in SQL Server questo è determinato dall'impostazione delle regole di confronto dell'istanza di SQL Server. Questa impostazione usa come valore predefinito la non applicazione della distinzione tra maiuscole e minuscole. È possibile chiamare il metodo `ToUpper` per fare in modo che il test non applichi in modo esplicito la distinzione tra maiuscole e minuscole: *Where(s => s.LastName.ToUpper().Contains(searchString.ToUpper())*. Questo fa sì che i risultati rimangano invariati se si modifica il codice in un secondo momento per usare un repository che restituisce una raccolta `IEnumerable` invece di un oggetto `IQueryable`. Quando si chiama il `Contains` metodo su una `IEnumerable` raccolta, si ottiene l'implementazione di .NET Framework; quando viene chiamato su un `IQueryable` oggetto, si ottiene l'implementazione del provider di database. Tuttavia, si verifica una riduzione delle prestazioni per questa soluzione. Il codice `ToUpper` dovrà inserire una funzione nella clausola WHERE dell'istruzione TSQL SELECT. In questo modo si evita che l'ottimizzazione usi un indice. Dato che SQL viene installato per lo più con l'impostazione senza distinzione tra maiuscole e minuscole, è consigliabile evitare il codice `ToUpper` fino a quando non si esegue la migrazione a un archivio con distinzione tra maiuscole e minuscole.

### <a name="add-a-search-box-to-the-student-index-view"></a>Aggiungere una casella di ricerca alla visualizzazione Student Index (Indice degli studenti)

In *Views/Student/Index.cshtml*, aggiungere il codice evidenziato immediatamente prima del tag tabella di apertura per creare una barra del titolo, una casella di testo e un pulsante **Search** (Ricerca).

[!code-cshtml[](intro/samples/cu/Views/Students/Index3.cshtml?range=9-23&highlight=5-13)]

Questo codice usa l' [helper tag](xref:mvc/views/tag-helpers/intro)`<form>` per aggiungere la casella di testo e il pulsante di ricerca. Per impostazione predefinita, l'helper tag `<form>` invia i dati del modulo con una richiesta POST, il che significa che i parametri vengono passati nel corpo del messaggio HTTP e non nell'URL come stringhe di query. Quando si specifica HTTP GET, i dati del modulo vengono passati nell'URL come stringhe di query, il che consente agli utenti di inserire l'URL tra i segnalibri. Le linee guida W3C consigliano di usare l'istruzione GET quando l'azione non risulta in un aggiornamento.

Eseguire l'app, selezionare la scheda **Students** (Studenti), immettere una stringa di ricerca e fare clic su Search (Ricerca) per verificare che il filtro funzioni.

![Pagina Student Index (Indice degli studenti) con filtro](sort-filter-page/_static/filtering.png)

Si noti che l'URL contiene la stringa di ricerca.

```html
http://localhost:5813/Students?SearchString=an
```

Se questa pagina viene inserita tra i segnalibri, quando si usa il segnalibro viene visualizzato l'elenco filtrato. L'aggiunta di `method="get"` al tag `form` è l'elemento che ha determinato la generazione della stringa di query.

In questa fase, se si fa clic sul collegamento di ordinamento di un'intestazione di colonna si perderà il valore del filtro immesso nella casella **Search** (Ricerca). Nella sezione successiva verrà spiegato come correggere il problema.

## <a name="add-paging-to-students-index"></a>Aggiungere la suddivisione in pagine per Student Index

Per aggiungere la suddivisione in pagine alla pagina Student Index (Indice degli studenti), creare una classe `PaginatedList` che usa le istruzioni `Skip` e `Take` per filtrare i dati sul server invece di recuperare sempre tutte le righe della tabella. È quindi possibile apportare altre modifiche nel metodo `Index` e aggiungere i pulsanti di suddivisione in pagine alla visualizzazione `Index`. Nella figura seguente vengono illustrati i pulsanti di suddivisione in pagine.

![Pagina Student Index (Indice degli studenti) con collegamenti di suddivisione in pagine](sort-filter-page/_static/paging.png)

Nella cartella del progetto, creare `PaginatedList.cs` e quindi sostituire il codice del modello con il codice seguente.

[!code-csharp[](intro/samples/cu/PaginatedList.cs)]

Il metodo `CreateAsync` in questo codice accetta le dimensioni di pagina e il numero delle pagine e applica le istruzioni `Skip` e `Take` appropriate a `IQueryable`. Quando `ToListAsync` viene chiamato su `IQueryable`, restituisce un elenco contenente solo la pagina richiesta. Le proprietà `HasPreviousPage` e `HasNextPage` possono essere usate per abilitare o disabilitare i pulsanti di suddivisione in pagine **Previous** (Indietro) e **Next** (Avanti).

Viene usato un metodo `CreateAsync` invece di un costruttore per creare l'oggetto `PaginatedList<T>` poiché i costruttori non possono eseguire codice asincrono.

## <a name="add-paging-to-index-method"></a>Aggiungere la suddivisione in pagine al metodo Index

In *StudentsController.cs* sostituire il metodo `Index` con il codice seguente.

[!code-csharp[](intro/samples/cu/Controllers/StudentsController.cs?name=snippet_SortFilterPage&highlight=1-5,7,11-18,45-46)]

Questo codice aggiunge un parametro di numeri di pagina, un parametro di ordinamento corrente e un parametro di filtro corrente alla firma del metodo.

```csharp
public async Task<IActionResult> Index(
    string sortOrder,
    string currentFilter,
    string searchString,
    int? pageNumber)
```

La prima volta che viene visualizzata la pagina o se l'utente non ha selezionato un collegamento di suddivisione in pagine o di ordinamento, tutti i parametri saranno Null.  Se si seleziona un collegamento di suddivisione in pagine, la variabile di pagina conterrà il numero della pagina da visualizzare.

L'elemento `ViewData` denominato CurrentSort offre la visualizzazione con l'ordinamento corrente, in quanto esso deve essere incluso nei collegamenti di suddivisione in pagine per mantenere l'ordinamento durante la suddivisione in pagine.

L'elemento `ViewData` denominato CurrentFilter offre la visualizzazione con la stringa di filtro corrente. Questo valore deve essere incluso nei collegamenti di suddivisione in pagine per mantenere le impostazioni di filtro nella suddivisione in pagine e deve essere ripristinato nella casella di testo quando la pagina viene nuovamente visualizzata.

Se la stringa di ricerca viene modificata nella suddivisione in pagine, la pagina deve essere reimpostata su 1, poiché il nuovo filtro può comportare la visualizzazione di dati diversi. La stringa di ricerca viene modificata quando si immette un valore nella casella di testo e si preme il pulsante Invia. In tal caso, il parametro `searchString` non è Null.

```csharp
if (searchString != null)
{
    pageNumber = 1;
}
else
{
    searchString = currentFilter;
}
```

Alla fine del metodo `Index`, il metodo `PaginatedList.CreateAsync` converte la query degli studenti in una pagina singola di studenti in un tipo di raccolta che supporta la suddivisione in pagine. La pagina singola di studenti viene quindi passata alla visualizzazione.

```csharp
return View(await PaginatedList<Student>.CreateAsync(students.AsNoTracking(), pageNumber ?? 1, pageSize));
```

Il metodo `PaginatedList.CreateAsync` accetta un numero di pagina. I due punti interrogativi rappresentano l'operatore null-coalescing. L'operatore null-coalescing definisce un valore predefinito per un tipo nullable. L'espressione `(pageNumber ?? 1)` significa restituzione del valore di `pageNumber` se ha un valore oppure restituzione di 1 se `pageNumber` è Null.

## <a name="add-paging-links"></a>Aggiungere collegamenti per la suddivisione in pagine

In *Views/Students/Index.cshtml* sostituire il codice esistente con il codice seguente. Le modifiche sono evidenziate.

[!code-cshtml[](intro/samples/cu/Views/Students/Index.cshtml?highlight=1,27,30,33,61-79)]

L'istruzione `@model` nella parte superiore della pagina specifica che la vista ottiene ora un oggetto `PaginatedList<T>` anziché un oggetto `List<T>`.

I collegamenti delle intestazioni di colonna usano la stringa di query per passare la stringa di ricerca corrente al controller in modo che l'utente possa procedere all'ordinamento all'interno dei risultati di filtro:

```html
<a asp-action="Index" asp-route-sortOrder="@ViewData["DateSortParm"]" asp-route-currentFilter ="@ViewData["CurrentFilter"]">Enrollment Date</a>
```

I pulsanti di suddivisione in pagine vengono visualizzati dagli helper tag:

```html
<a asp-action="Index"
   asp-route-sortOrder="@ViewData["CurrentSort"]"
   asp-route-pageNumber="@(Model.PageIndex - 1)"
   asp-route-currentFilter="@ViewData["CurrentFilter"]"
   class="btn btn-default @prevDisabled">
   Previous
</a>
```

Eseguire l'app e passare alla pagina Students (Studenti).

![Pagina Student Index (Indice degli studenti) con collegamenti di suddivisione in pagine](sort-filter-page/_static/paging.png)

Fare clic sui collegamenti di suddivisione in pagine in diversi tipi di ordinamento per verificare che la suddivisione in pagine funzioni. Immettere quindi una stringa di ricerca e provare nuovamente la suddivisione in pagine per verificare che funzioni correttamente anche con l'ordinamento e il filtro.

## <a name="create-an-about-page"></a>Creare una pagina About

Per la pagina **About** (Informazioni) del sito Web Contoso University verrà visualizzato il numero di studenti iscritti per ogni data di iscrizione. Questa operazione richiede calcoli di raggruppamento e semplici sui gruppi. Per completare questa procedura, è necessario eseguire le operazioni seguenti:

* Creare una classe modello di visualizzazione per i dati che è necessario passare alla visualizzazione.
* Creare il metodo About nel controller Home.
* Creare la visualizzazione About.

### <a name="create-the-view-model"></a>Creare il modello di visualizzazione

Creare una cartella *SchoolViewModels* nella cartella *Models*.

Nella nuova cartella aggiungere un file di classe *EnrollmentDateGroup.cs* e sostituire il codice del modello con il codice seguente:

[!code-csharp[](intro/samples/cu/Models/SchoolViewModels/EnrollmentDateGroup.cs)]

### <a name="modify-the-home-controller"></a>Modificare il controller Home

In *HomeController.cs* aggiungere le seguenti istruzioni all'inizio del file:

[!code-csharp[](intro/samples/cu/Controllers/HomeController.cs?name=snippet_Usings1)]

Aggiungere una variabile di classe per il contesto del database immediatamente dopo la parentesi graffa di apertura per la classe e ottenere un'istanza del contesto da ASP.NET Core DI:

[!code-csharp[](intro/samples/cu/Controllers/HomeController.cs?name=snippet_AddContext&highlight=3,5,7)]

Aggiungere un metodo `About` con il codice seguente:

[!code-csharp[](intro/samples/cu/Controllers/HomeController.cs?name=snippet_UseDbSet)]

L'istruzione LINQ raggruppa le entità di studenti per data di registrazione, calcola il numero di entità in ogni gruppo e archivia i risultati in una raccolta di oggetti di modello della visualizzazione `EnrollmentDateGroup`.

### <a name="create-the-about-view"></a>Creare la visualizzazione About

Aggiungere un file *Views/Home/About.cshtml* con il codice seguente:

[!code-cshtml[](intro/samples/cu/Views/Home/About.cshtml)]

Eseguire l'app e passare alla pagina About. Il numero di studenti per ogni data di registrazione viene visualizzato in una tabella.

## <a name="get-the-code"></a>Ottenere il codice

[Scaricare o visualizzare l'applicazione completata.](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/data/ef-mvc/intro/samples/cu-final)

## <a name="next-steps"></a>Passaggi successivi

In questa esercitazione:

> [!div class="checklist"]
> * Aggiungere collegamenti per l'ordinamento delle colonne
> * Aggiungere una casella di ricerca
> * Aggiungere la suddivisione in pagine per Student Index
> * Aggiungere la suddivisione in pagine al metodo Index
> * Aggiungere collegamenti per la suddivisione in pagine
> * Creare una pagina About

Passare all'esercitazione successiva per apprendere come gestire le modifiche al modello di dati tramite le migrazioni.

> [!div class="nextstepaction"]
> [Passaggio successivo: gestire le modifiche al modello di dati](migrations.md)
