---
title: Parte 2, Razor pagine con EF core in ASP.NET Core-CRUD
author: rick-anderson
description: Parte 2 delle Razor pagine e della serie di esercitazioni Entity Framework.
ms.author: riande
ms.date: 07/22/2019
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
uid: data/ef-rp/crud
ms.openlocfilehash: c424f2a46fb62af6283ffa3a02c4134bbe35ffc9
ms.sourcegitcommit: 3982ff9dabb5b12aeb0a61cde2686b5253364f5d
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/04/2021
ms.locfileid: "102118928"
---
# <a name="part-2-razor-pages-with-ef-core-in-aspnet-core---crud"></a>Parte 2, Razor pagine con EF core in ASP.NET Core-CRUD

[Tom Dykstra](https://github.com/tdykstra), [Jon P Smith](https://twitter.com/thereformedprog) e [Rick Anderson](https://twitter.com/RickAndMSFT)

[!INCLUDE [about the series](~/includes/RP-EF/intro.md)]

::: moniker range=">= aspnetcore-5.0"

In questa esercitazione viene esaminato e personalizzato il codice CRUD (Create, Read, Update, Delete) con scaffolding.

## <a name="no-repository"></a>Nessun repository

Alcuni sviluppatori usano un livello di servizio o un modello di repository per creare un livello di astrazione tra l'interfaccia utente ( Razor pagine) e il livello di accesso ai dati. Questa esercitazione non segue questo approccio. Per ridurre la complessità e mantenere questa esercitazione incentrata su EF Core, il codice EF Core viene aggiunto direttamente alle classi dei modelli di pagina. 

## <a name="update-the-details-page"></a>Aggiornare la pagina Details

Il codice con scaffolding per le pagine Students non include i dati di iscrizione. In questa sezione vengono aggiunte le iscrizioni alla pagina Details.

### <a name="read-enrollments"></a>Leggere le iscrizioni

Per visualizzare i dati di iscrizione di uno studente nella pagina, è necessario leggerli. Il codice con scaffolding in *Pages/Students/Details.cshtml.cs* legge solo i dati degli studenti, senza i dati di iscrizione:

[!code-csharp[Main](intro/samples/cu30snapshots/2-crud/Pages/Students/Details1.cshtml.cs?name=snippet_OnGetAsync&highlight=8)]

Sostituire il metodo `OnGetAsync` con il codice seguente per leggere i dati di iscrizione per lo studente selezionato. Le modifiche sono evidenziate.

[!code-csharp[Main](intro/samples/cu30/Pages/Students/Details.cshtml.cs?name=snippet_OnGetAsync&highlight=8-12)]

I metodi [Include](/dotnet/api/microsoft.entityframeworkcore.entityframeworkqueryableextensions.include) e [ThenInclude](/dotnet/api/microsoft.entityframeworkcore.entityframeworkqueryableextensions.theninclude#Microsoft_EntityFrameworkCore_EntityFrameworkQueryableExtensions_ThenInclude__3_Microsoft_EntityFrameworkCore_Query_IIncludableQueryable___0_System_Collections_Generic_IEnumerable___1___System_Linq_Expressions_Expression_System_Func___1___2___) fanno in modo che il contesto carichi la proprietà di navigazione `Student.Enrollments` e la proprietà di navigazione `Enrollment.Course` all'interno di ogni iscrizione. Questi metodi vengono esaminati in dettaglio nell'esercitazione [leggere i dati correlati](xref:data/ef-rp/read-related-data) .

Il metodo [AsNoTracking](/dotnet/api/microsoft.entityframeworkcore.entityframeworkqueryableextensions.asnotracking#Microsoft_EntityFrameworkCore_EntityFrameworkQueryableExtensions_AsNoTracking__1_System_Linq_IQueryable___0__) migliora le prestazioni negli scenari in cui le entità restituite non vengono aggiornate nel contesto corrente. `AsNoTracking` è descritto più avanti in questa esercitazione.

### <a name="display-enrollments"></a>Visualizzare le iscrizioni

Sostituire il codice in *Pages/Students/Details.cshtml* con il codice seguente per visualizzare un elenco di iscrizioni. Le modifiche sono evidenziate.

[!code-cshtml[Main](intro/samples/cu30/Pages/Students/Details.cshtml?highlight=32-53)]

Il codice precedente esegue il ciclo nelle entità nella proprietà di navigazione `Enrollments`. Per ogni registrazione, il codice visualizza il titolo del corso e il voto. Il titolo del corso viene recuperato dall'entità Course memorizzata nella proprietà di navigazione `Course` dell'entità Enrollments.

Eseguire l'app, selezionare la scheda **Students** e fare clic sul collegamento **Details** relativo a uno studente. Viene visualizzato l'elenco dei corsi e dei voti dello studente selezionato.

### <a name="ways-to-read-one-entity"></a>Modalità di lettura di un'entità

Il codice generato usa [FirstOrDefaultAsync](/dotnet/api/microsoft.entityframeworkcore.entityframeworkqueryableextensions.firstordefaultasync#Microsoft_EntityFrameworkCore_EntityFrameworkQueryableExtensions_FirstOrDefaultAsync__1_System_Linq_IQueryable___0__System_Threading_CancellationToken_) per leggere un'entità. Questo metodo restituisce Null se non viene trovato alcun elemento. In caso contrario, viene restituita la prima riga trovata che soddisfa i criteri di filtro della query. `FirstOrDefaultAsync` è in genere una scelta migliore rispetto alle alternative seguenti:

* [SingleOrDefaultAsync](/dotnet/api/microsoft.entityframeworkcore.entityframeworkqueryableextensions.singleordefaultasync#Microsoft_EntityFrameworkCore_EntityFrameworkQueryableExtensions_SingleOrDefaultAsync__1_System_Linq_IQueryable___0__System_Linq_Expressions_Expression_System_Func___0_System_Boolean___System_Threading_CancellationToken_) - Genera un'eccezione se è presente più di un'entità che soddisfa il filtro di query. Per determinare se la query può restituire più di una riga, `SingleOrDefaultAsync` tenta di recuperare più righe. Questa operazione aggiuntiva non è necessario se la query può restituire solo un'entità, ad esempio quando esegue la ricerca in base a una chiave univoca.
* [FindAsync](/dotnet/api/microsoft.entityframeworkcore.dbcontext.findasync#Microsoft_EntityFrameworkCore_DbContext_FindAsync_System_Type_System_Object___) - Trova un'entità con la chiave primaria. Se il contesto rileva un'entità con la chiave primaria, l'entità viene restituita senza una richiesta al database. Questo metodo è ottimizzato per la ricerca di una singola entità, ma non è possibile chiamare `Include` con `FindAsync`.  Se sono necessari dati correlati, `FirstOrDefaultAsync` è quindi la scelta migliore.

### <a name="route-data-vs-query-string"></a>Dati di route o stringa di query

L'URL della pagina Details è `https://localhost:<port>/Students/Details?id=1`. Il valore della chiave primaria dell'entità si trova nella stringa di query. Alcuni sviluppatori preferiscono passare il valore della chiave nei dati della route: `https://localhost:<port>/Students/Details/1`. Per altre informazioni, vedere [Aggiornare il codice generato](xref:tutorials/razor-pages/da1#update-the-generated-code).

## <a name="update-the-create-page"></a>Aggiornare la pagina Create

Il codice `OnPostAsync` con scaffolding per la pagina Create è vulnerabile all'[overposting](#overposting). Sostituire il metodo `OnPostAsync` in *Pages/Students/Create.cshtml.cs* con il codice seguente.

[!code-csharp[Main](intro/samples/cu30/Pages/Students/Create.cshtml.cs?name=snippet_OnPostAsync)]

<a name="TryUpdateModelAsync"></a>

### <a name="tryupdatemodelasync"></a>TryUpdateModelAsync

Il codice precedente crea un oggetto Student e quindi usa i campi del modulo pubblicati per aggiornare le proprietà dell'oggetto Student. Il metodo [TryUpdateModelAsync](/dotnet/api/microsoft.aspnetcore.mvc.controllerbase.tryupdatemodelasync#Microsoft_AspNetCore_Mvc_ControllerBase_TryUpdateModelAsync_System_Object_System_Type_System_String_):

* Usa i valori del modulo pubblicati dalla proprietà [PageContext](/dotnet/api/microsoft.aspnetcore.mvc.razorpages.pagemodel.pagecontext#Microsoft_AspNetCore_Mvc_RazorPages_PageModel_PageContext) in [PageModel](/dotnet/api/microsoft.aspnetcore.mvc.razorpages.pagemodel).
* Aggiorna solo le proprietà elencate (`s => s.FirstMidName, s => s.LastName, s => s.EnrollmentDate`).
* Cerca i campi del modulo con il prefisso "student". Ad esempio: `Student.FirstMidName`. Non viene fatta distinzione tra maiuscole e minuscole.
* Usa il sistema di [associazione di modelli](xref:mvc/models/model-binding) per convertire i valori dei moduli da stringa ai tipi nel modello `Student`. Ad esempio, `EnrollmentDate` viene convertito in `DateTime` .

Eseguire l'app e creare un'entità Student per testare la pagina Create.

## <a name="overposting"></a>Overposting

L'uso di `TryUpdateModel` per l'aggiornamento dei campi con i valori inviati è una procedura di sicurezza consigliata poiché impedisce l'overposting. Ad esempio, si supponga che l'entità Student includa una proprietà `Secret` che la pagina Web non deve aggiornare o aggiungere:

[!code-csharp[Main](intro/samples/cu30snapshots/2-crud/Models/StudentZsecret.cs?name=snippet_Intro&highlight=7)]

Anche se l'app non ha un `Secret` campo nella pagina Crea o aggiorna Razor , un hacker può impostare il `Secret` valore tramite l'overposting. Un hacker potrebbe usare uno strumento come Fiddler oppure scrivere codice JavaScript per inviare un valore di modulo `Secret`. Il codice originale non limita i campi usati dallo strumento di associazione di modelli durante la creazione di un'istanza di Student.

Qualsiasi valore specificato dall'hacker per il campo di modulo `Secret` viene aggiornato nel database. La figura seguente mostra lo strumento Fiddler che aggiunge il `Secret` campo con il valore "overpost" ai valori del modulo inviato.

![Fiddler aggiunge il campo Secret](../ef-mvc/crud/_static/fiddler.png)

Il valore "OverPost" è stato aggiunto alla proprietà `Secret` della riga inserita. Ciò accade anche se il progettista dell'app non ha mai previsto che la proprietà `Secret` venisse impostata con la pagina Create.

### <a name="view-model"></a>Visualizzare il modello

I modelli di visualizzazione rappresentano un altro metodo per impedire l'overposting.

Il modello di applicazione è spesso chiamato modello di dominio. Il modello di dominio contiene in genere tutte le proprietà richieste dall'entità corrispondente nel database. Il modello di visualizzazione contiene solo le proprietà necessarie per la pagina dell'interfaccia utente, ad esempio la pagina Crea.

Oltre al modello di visualizzazione, alcune app usano un modello di associazione o un modello di input per passare i dati tra la Razor classe del modello di pagina delle pagine e il browser. 

Si consideri il modello di visualizzazione `StudentVM` seguente:

[!code-csharp[Main](intro/samples/cu50/ViewModels/StudentVM.cs?name=snippet)]

Il codice seguente usa il modello di visualizzazione `StudentVM` per creare un nuovo studente:

[!code-csharp[Main](intro/samples/cu50/Pages/Students/CreateVM.cshtml.cs?name=snippet)]

Il metodo [SetValues](/dotnet/api/microsoft.entityframeworkcore.changetracking.propertyvalues.setvalues#Microsoft_EntityFrameworkCore_ChangeTracking_PropertyValues_SetValues_System_Object_) imposta i valori di questo oggetto leggendo i valori di un altro oggetto [PropertyValues](/dotnet/api/microsoft.entityframeworkcore.changetracking.propertyvalues). `SetValues` usa la corrispondenza dei nomi di proprietà. Tipo di modello di visualizzazione:

* Non è necessario che sia correlato al tipo di modello.
* Deve avere proprietà corrispondenti.

`StudentVM`L'utilizzo di richiede l'utilizzo della pagina Create `StudentVM` anziché `Student` :

[!code-cshtml[Main](intro/samples/cu50/Pages/Students/CreateVM.cshtml)]

## <a name="update-the-edit-page"></a>Aggiornare la pagina Edit (Modifica)

In *Pages/Students/Edit.cshtml.cs* sostituire i metodi `OnGetAsync` e `OnPostAsync` con il codice seguente.

[!code-csharp[Main](intro/samples/cu30/Pages/Students/Edit.cshtml.cs?name=snippet_OnGetPost)]

Le modifiche al codice sono simili alla pagina Create con alcune eccezioni:

* `FirstOrDefaultAsync` è stato sostituito con [FindAsync](/dotnet/api/microsoft.entityframeworkcore.dbset-1.findasync). Quando non è necessario includere dati correlati, `FindAsync` è più efficiente.
* `OnPostAsync` ha un parametro `id`.
* Lo studente corrente viene recuperato dal database senza creare uno studente vuoto.

Eseguire l'app e testarla creando e modificando uno studente.

## <a name="entity-states"></a>Stati di entità

Il contesto del database tiene traccia della sincronizzazione delle entità in memoria con le righe corrispondenti nel database. Queste informazioni di traccia determinano le operazioni eseguite quando viene chiamato [SaveChangesAsync](/dotnet/api/microsoft.entityframeworkcore.dbcontext.savechangesasync#Microsoft_EntityFrameworkCore_DbContext_SaveChangesAsync_System_Threading_CancellationToken_). Ad esempio, quando una nuova entità viene passata al metodo [AddAsync](/dotnet/api/microsoft.entityframeworkcore.dbcontext.addasync), lo stato dell'entità viene impostato su [Added](/dotnet/api/microsoft.entityframeworkcore.entitystate#Microsoft_EntityFrameworkCore_EntityState_Added). Quando `SaveChangesAsync` viene chiamato, il contesto del database emette un `INSERT` comando SQL.

Un'entità può essere in uno degli [stati seguenti](/dotnet/api/microsoft.entityframeworkcore.entitystate):

* `Added`: L'entità non esiste ancora nel database. Il `SaveChanges` metodo rilascia un' `INSERT` istruzione.

* `Unchanged`: non è necessario salvare alcuna modifica con questa entità. Un'entità ha questo stato quando viene letta dal database.

* `Modified`: sono stati modificati alcuni o tutti i valori di proprietà dell'entità. Il `SaveChanges` metodo rilascia un' `UPDATE` istruzione.

* `Deleted`: l'entità è stata contrassegnata per l'eliminazione. Il `SaveChanges` Metodo emette un' `DELETE` istruzione.

* `Detached`: L'entità non viene rilevata dal contesto del database.

In un'applicazione desktop le modifiche dello stato vengono in genere impostate automaticamente. Viene letta un'entità, vengono apportate le modifiche e lo stato dell'entità viene modificato automaticamente in `Modified`. La chiamata di `SaveChanges` genera un' `UPDATE` istruzione SQL che aggiorna solo le proprietà modificate.

In un'app Web il `DbContext` che legge un'entità e visualizza i dati viene eliminato dopo il rendering di una pagina. Quando viene chiamato il metodo `OnPostAsync` di una pagina, viene effettuata una nuova richiesta Web con una nuova istanza di `DbContext`. La rilettura dell'entità nel nuovo contesto simula l'elaborazione desktop.

## <a name="update-the-delete-page"></a>Aggiornare la pagina Delete (Elimina)

In questa sezione viene implementato un messaggio di errore personalizzato quando la chiamata a ha `SaveChanges` esito negativo.

Sostituire il codice in *Pages/Students/Delete.cshtml.cs* con il codice seguente. Le modifiche sono evidenziate:

[!code-csharp[Main](intro/samples/cu50/Pages/Students/Delete.cshtml.cs?name=snippet_All&highlight=12-14,22,30-33,45-99)]

Il codice precedente aggiunge il parametro facoltativo `saveChangesError` alla firma del metodo `OnGetAsync`. `saveChangesError` indica se il metodo è stato chiamato dopo un errore di eliminazione dell'oggetto Student. L'operazione di eliminazione potrebbe non riuscire a causa di problemi di rete temporanei. Gli errori di rete temporanei sono più probabili quando il database è nel cloud. Il `saveChangesError` parametro è `false` quando la pagina Delete `OnGetAsync` viene chiamata dall'interfaccia utente. Quando `OnGetAsync` viene chiamato da `OnPostAsync` perché l'operazione di eliminazione non è riuscita, il `saveChangesError` parametro è `true` .

Il metodo `OnPostAsync` recupera l'entità selezionata, quindi chiama il metodo [Remove](/dotnet/api/microsoft.entityframeworkcore.dbcontext.remove#Microsoft_EntityFrameworkCore_DbContext_Remove_System_Object_) per impostare lo stato dell'entità su `Deleted`. Quando `SaveChanges` viene chiamato il metodo, `DELETE` viene generato un comando SQL. Se `Remove` ha esito negativo:

* Viene rilevata l'eccezione del database.
* Il metodo `OnGetAsync` delle pagine Delete viene chiamato con `saveChangesError=true`.

Aggiungere un messaggio di errore a *pages/students/Delete. cshtml*:

[!code-cshtml[Main](intro/samples/cu30/Pages/Students/Delete.cshtml?highlight=10)]

Eseguire l'app ed eliminare uno studente per testare la pagina Delete.

## <a name="next-steps"></a>Passaggi successivi

> [!div class="step-by-step"]
> [Esercitazione precedente](xref:data/ef-rp/intro) 
>  [Esercitazione successiva](xref:data/ef-rp/sort-filter-page)

::: moniker-end

::: moniker range=">= aspnetcore-3.0 < aspnetcore-5.0"

In questa esercitazione viene esaminato e personalizzato il codice CRUD (Create, Read, Update, Delete) con scaffolding.

## <a name="no-repository"></a>Nessun repository

Alcuni sviluppatori usano un livello di servizio o un modello di repository per creare un livello di astrazione tra l'interfaccia utente ( Razor pagine) e il livello di accesso ai dati. Questa esercitazione non segue questo approccio. Per ridurre la complessità e mantenere questa esercitazione incentrata su EF Core, il codice EF Core viene aggiunto direttamente alle classi dei modelli di pagina. 

## <a name="update-the-details-page"></a>Aggiornare la pagina Details

Il codice con scaffolding per le pagine Students non include i dati di iscrizione. In questa sezione le registrazioni vengono aggiunte alla pagina dei dettagli.

### <a name="read-enrollments"></a>Leggere le iscrizioni

Per visualizzare i dati di registrazione di uno studente nella pagina, è necessario leggere i dati di registrazione. Il codice con scaffolding in *Pages/Students/Details.cshtml.cs* legge solo i dati degli studenti, senza i dati di iscrizione:

[!code-csharp[Main](intro/samples/cu30snapshots/2-crud/Pages/Students/Details1.cshtml.cs?name=snippet_OnGetAsync&highlight=8)]

Sostituire il metodo `OnGetAsync` con il codice seguente per leggere i dati di iscrizione per lo studente selezionato. Le modifiche sono evidenziate.

[!code-csharp[Main](intro/samples/cu30/Pages/Students/Details.cshtml.cs?name=snippet_OnGetAsync&highlight=8-12)]

I metodi [Include](/dotnet/api/microsoft.entityframeworkcore.entityframeworkqueryableextensions.include) e [ThenInclude](/dotnet/api/microsoft.entityframeworkcore.entityframeworkqueryableextensions.theninclude#Microsoft_EntityFrameworkCore_EntityFrameworkQueryableExtensions_ThenInclude__3_Microsoft_EntityFrameworkCore_Query_IIncludableQueryable___0_System_Collections_Generic_IEnumerable___1___System_Linq_Expressions_Expression_System_Func___1___2___) fanno in modo che il contesto carichi la proprietà di navigazione `Student.Enrollments` e la proprietà di navigazione `Enrollment.Course` all'interno di ogni iscrizione. Questi metodi vengono esaminati in dettaglio nell'esercitazione [Lettura dei dati correlati](xref:data/ef-rp/read-related-data).

Il metodo [AsNoTracking](/dotnet/api/microsoft.entityframeworkcore.entityframeworkqueryableextensions.asnotracking#Microsoft_EntityFrameworkCore_EntityFrameworkQueryableExtensions_AsNoTracking__1_System_Linq_IQueryable___0__) migliora le prestazioni negli scenari in cui le entità restituite non vengono aggiornate nel contesto corrente. `AsNoTracking` è descritto più avanti in questa esercitazione.

### <a name="display-enrollments"></a>Visualizzare le iscrizioni

Sostituire il codice in *Pages/Students/Details.cshtml* con il codice seguente per visualizzare un elenco di iscrizioni. Le modifiche sono evidenziate.

[!code-cshtml[Main](intro/samples/cu30/Pages/Students/Details.cshtml?highlight=32-53)]

Il codice precedente esegue il ciclo nelle entità nella proprietà di navigazione `Enrollments`. Per ogni registrazione, il codice visualizza il titolo del corso e il voto. Il titolo del corso viene recuperato dall'entità Course memorizzata nella proprietà di navigazione `Course` dell'entità Enrollments.

Eseguire l'app, selezionare la scheda **Students** e fare clic sul collegamento **Details** relativo a uno studente. Viene visualizzato l'elenco dei corsi e dei voti dello studente selezionato.

### <a name="ways-to-read-one-entity"></a>Modalità di lettura di un'entità

Il codice generato usa [FirstOrDefaultAsync](/dotnet/api/microsoft.entityframeworkcore.entityframeworkqueryableextensions.firstordefaultasync#Microsoft_EntityFrameworkCore_EntityFrameworkQueryableExtensions_FirstOrDefaultAsync__1_System_Linq_IQueryable___0__System_Threading_CancellationToken_) per leggere un'entità. Questo metodo restituisce Null se non viene trovato alcun elemento. In caso contrario, viene restituita la prima riga trovata che soddisfa i criteri di filtro della query. `FirstOrDefaultAsync` è in genere una scelta migliore rispetto alle alternative seguenti:

* [SingleOrDefaultAsync](/dotnet/api/microsoft.entityframeworkcore.entityframeworkqueryableextensions.singleordefaultasync#Microsoft_EntityFrameworkCore_EntityFrameworkQueryableExtensions_SingleOrDefaultAsync__1_System_Linq_IQueryable___0__System_Linq_Expressions_Expression_System_Func___0_System_Boolean___System_Threading_CancellationToken_) - Genera un'eccezione se è presente più di un'entità che soddisfa il filtro di query. Per determinare se la query può restituire più di una riga, `SingleOrDefaultAsync` tenta di recuperare più righe. Questa operazione aggiuntiva non è necessario se la query può restituire solo un'entità, ad esempio quando esegue la ricerca in base a una chiave univoca.
* [FindAsync](/dotnet/api/microsoft.entityframeworkcore.dbcontext.findasync#Microsoft_EntityFrameworkCore_DbContext_FindAsync_System_Type_System_Object___) - Trova un'entità con la chiave primaria. Se il contesto rileva un'entità con la chiave primaria, l'entità viene restituita senza una richiesta al database. Questo metodo è ottimizzato per la ricerca di una singola entità, ma non è possibile chiamare `Include` con `FindAsync`.  Se sono necessari dati correlati, `FirstOrDefaultAsync` è quindi la scelta migliore.

### <a name="route-data-vs-query-string"></a>Dati di route o stringa di query

L'URL della pagina Details è `https://localhost:<port>/Students/Details?id=1`. Il valore della chiave primaria dell'entità si trova nella stringa di query. Alcuni sviluppatori preferiscono passare il valore della chiave nei dati della route: `https://localhost:<port>/Students/Details/1`. Per altre informazioni, vedere [Aggiornare il codice generato](xref:tutorials/razor-pages/da1#update-the-generated-code).

## <a name="update-the-create-page"></a>Aggiornare la pagina Create

Il codice `OnPostAsync` con scaffolding per la pagina Create è vulnerabile all'[overposting](#overposting). Sostituire il metodo `OnPostAsync` in *Pages/Students/Create.cshtml.cs* con il codice seguente.

[!code-csharp[Main](intro/samples/cu30/Pages/Students/Create.cshtml.cs?name=snippet_OnPostAsync)]

<a name="TryUpdateModelAsync"></a>

### <a name="tryupdatemodelasync"></a>TryUpdateModelAsync

Il codice precedente crea un oggetto Student e quindi usa i campi del modulo pubblicati per aggiornare le proprietà dell'oggetto Student. Il metodo [TryUpdateModelAsync](/dotnet/api/microsoft.aspnetcore.mvc.controllerbase.tryupdatemodelasync#Microsoft_AspNetCore_Mvc_ControllerBase_TryUpdateModelAsync_System_Object_System_Type_System_String_):

* Usa i valori del modulo pubblicati dalla proprietà [PageContext](/dotnet/api/microsoft.aspnetcore.mvc.razorpages.pagemodel.pagecontext#Microsoft_AspNetCore_Mvc_RazorPages_PageModel_PageContext) in [PageModel](/dotnet/api/microsoft.aspnetcore.mvc.razorpages.pagemodel).
* Aggiorna solo le proprietà elencate (`s => s.FirstMidName, s => s.LastName, s => s.EnrollmentDate`).
* Cerca i campi del modulo con il prefisso "student". Ad esempio: `Student.FirstMidName`. Non viene fatta distinzione tra maiuscole e minuscole.
* Usa il sistema di [associazione di modelli](xref:mvc/models/model-binding) per convertire i valori dei moduli da stringa ai tipi nel modello `Student`. Ad esempio, `EnrollmentDate` deve essere convertito in DateTime.

Eseguire l'app e creare un'entità Student per testare la pagina Create.

## <a name="overposting"></a>Overposting

L'uso di `TryUpdateModel` per l'aggiornamento dei campi con i valori inviati è una procedura di sicurezza consigliata poiché impedisce l'overposting. Ad esempio, si supponga che l'entità Student includa una proprietà `Secret` che la pagina Web non deve aggiornare o aggiungere:

[!code-csharp[Main](intro/samples/cu30snapshots/2-crud/Models/StudentZsecret.cs?name=snippet_Intro&highlight=7)]

Anche se l'app non ha un `Secret` campo nella pagina Crea o aggiorna Razor , un hacker può impostare il `Secret` valore tramite l'overposting. Un hacker potrebbe usare uno strumento come Fiddler oppure scrivere codice JavaScript per inviare un valore di modulo `Secret`. Il codice originale non limita i campi usati dallo strumento di associazione di modelli durante la creazione di un'istanza di Student.

Qualsiasi valore specificato dall'hacker per il campo di modulo `Secret` viene aggiornato nel database. L'immagine seguente illustra lo strumento Fiddler che aggiunge il campo `Secret` (con il valore "OverPost") ai valori di modulo inviati.

![Fiddler aggiunge il campo Secret](../ef-mvc/crud/_static/fiddler.png)

Il valore "OverPost" è stato aggiunto alla proprietà `Secret` della riga inserita. Ciò accade anche se il progettista dell'app non ha mai previsto che la proprietà `Secret` venisse impostata con la pagina Create.

### <a name="view-model"></a>Visualizzare il modello

I modelli di visualizzazione rappresentano un altro metodo per impedire l'overposting.

Il modello di applicazione è spesso chiamato modello di dominio. Il modello di dominio contiene in genere tutte le proprietà richieste dall'entità corrispondente nel database. Il modello di visualizzazione contiene solo le proprietà necessarie per l'interfaccia utente per cui viene usato, ad esempio la pagina Create.

Oltre al modello di visualizzazione, alcune app usano un modello di associazione o un modello di input per passare i dati tra la Razor classe del modello di pagina delle pagine e il browser. 

Si consideri il modello di visualizzazione `Student` seguente:

[!code-csharp[Main](intro/samples/cu30snapshots/2-crud/Models/StudentVM.cs)]

Il codice seguente usa il modello di visualizzazione `StudentVM` per creare un nuovo studente:

[!code-csharp[Main](intro/samples/cu30snapshots/2-crud/Pages/Students/CreateVM.cshtml.cs?name=snippet_OnPostAsync)]

Il metodo [SetValues](/dotnet/api/microsoft.entityframeworkcore.changetracking.propertyvalues.setvalues#Microsoft_EntityFrameworkCore_ChangeTracking_PropertyValues_SetValues_System_Object_) imposta i valori di questo oggetto leggendo i valori di un altro oggetto [PropertyValues](/dotnet/api/microsoft.entityframeworkcore.changetracking.propertyvalues). `SetValues` usa la corrispondenza dei nomi di proprietà. Poiché il tipo di modello di visualizzazione non deve essere correlato al tipo di modello, è sufficiente che abbia proprietà corrispondenti.

Se si usa `StudentVM` è necessario che [Create.cshtml](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/data/ef-rp/intro/samples/cu30snapshots/2-crud/Pages/Students/CreateVM.cshtml) venga aggiornato per l'uso di `StudentVM` anziché `Student`.

## <a name="update-the-edit-page"></a>Aggiornare la pagina Edit (Modifica)

In *Pages/Students/Edit.cshtml.cs* sostituire i metodi `OnGetAsync` e `OnPostAsync` con il codice seguente.

[!code-csharp[Main](intro/samples/cu30/Pages/Students/Edit.cshtml.cs?name=snippet_OnGetPost)]

Le modifiche al codice sono simili alla pagina Create con alcune eccezioni:

* `FirstOrDefaultAsync` è stato sostituito con [FindAsync](/dotnet/api/microsoft.entityframeworkcore.dbset-1.findasync). Quando i dati correlati inclusi non sono necessari, `FindAsync` è più efficiente.
* `OnPostAsync` ha un parametro `id`.
* Lo studente corrente viene recuperato dal database senza creare uno studente vuoto.

Eseguire l'app e testarla creando e modificando uno studente.

## <a name="entity-states"></a>Stati di entità

Il contesto del database tiene traccia della sincronizzazione delle entità in memoria con le righe corrispondenti nel database. Queste informazioni di traccia determinano le operazioni eseguite quando viene chiamato [SaveChangesAsync](/dotnet/api/microsoft.entityframeworkcore.dbcontext.savechangesasync#Microsoft_EntityFrameworkCore_DbContext_SaveChangesAsync_System_Threading_CancellationToken_). Ad esempio, quando una nuova entità viene passata al metodo [AddAsync](/dotnet/api/microsoft.entityframeworkcore.dbcontext.addasync), lo stato dell'entità viene impostato su [Added](/dotnet/api/microsoft.entityframeworkcore.entitystate#Microsoft_EntityFrameworkCore_EntityState_Added). Quando viene chiamato `SaveChangesAsync`, il contesto del database genera un comando SQL INSERT.

Un'entità può essere in uno degli [stati seguenti](/dotnet/api/microsoft.entityframeworkcore.entitystate):

* `Added`: L'entità non esiste ancora nel database. Il metodo `SaveChanges` genera un'istruzione INSERT.

* `Unchanged`: non è necessario salvare alcuna modifica con questa entità. Un'entità ha questo stato quando viene letta dal database.

* `Modified`: sono stati modificati alcuni o tutti i valori di proprietà dell'entità. Il metodo `SaveChanges` genera un'istruzione UPDATE.

* `Deleted`: l'entità è stata contrassegnata per l'eliminazione. Il metodo `SaveChanges` genera un'istruzione DELETE.

* `Detached`: L'entità non viene rilevata dal contesto del database.

In un'applicazione desktop le modifiche dello stato vengono in genere impostate automaticamente. Viene letta un'entità, vengono apportate le modifiche e lo stato dell'entità viene modificato automaticamente in `Modified`. La chiamata di `SaveChanges` genera un'istruzione SQL UPDATE che aggiorna solo le proprietà modificate.

In un'app Web il `DbContext` che legge un'entità e visualizza i dati viene eliminato dopo il rendering di una pagina. Quando viene chiamato il metodo `OnPostAsync` di una pagina, viene effettuata una nuova richiesta Web con una nuova istanza di `DbContext`. La rilettura dell'entità nel nuovo contesto simula l'elaborazione desktop.

## <a name="update-the-delete-page"></a>Aggiornare la pagina Delete (Elimina)

In questa sezione viene implementato un messaggio di errore personalizzato quando la chiamata a `SaveChanges` ha esito negativo.

Sostituire il codice in *Pages/Students/Delete.cshtml.cs* con il codice seguente. Le modifiche vengono evidenziate (a eccezione della pulizia delle istruzioni `using`).

[!code-csharp[Main](intro/samples/cu30/Pages/Students/Delete.cshtml.cs?name=snippet_All&highlight=20,22,30,38-41,53-71)]

Il codice precedente aggiunge il parametro facoltativo `saveChangesError` alla firma del metodo `OnGetAsync`. `saveChangesError` indica se il metodo è stato chiamato dopo un errore di eliminazione dell'oggetto Student. L'operazione di eliminazione potrebbe non riuscire a causa di problemi di rete temporanei. Gli errori di rete temporanei sono più probabili quando il database è nel cloud. Il parametro`saveChangesError` è false quando si chiama `OnGetAsync` della pagina Delete dall'interfaccia utente. Quando `OnGetAsync` viene chiamato da `OnPostAsync` (perché l'operazione di eliminazione ha avuto esito negativo), il parametro `saveChangesError` ha valore true.

Il metodo `OnPostAsync` recupera l'entità selezionata, quindi chiama il metodo [Remove](/dotnet/api/microsoft.entityframeworkcore.dbcontext.remove#Microsoft_EntityFrameworkCore_DbContext_Remove_System_Object_) per impostare lo stato dell'entità su `Deleted`. Quando viene chiamato `SaveChanges`, viene generato un comando SQL DELETE. Se `Remove` ha esito negativo:

* Viene rilevata l'eccezione del database.
* Il metodo della pagina Delete `OnGetAsync` viene chiamato con `saveChangesError=true` .

Aggiungere un messaggio di errore alla Razor pagina Delete (*pages/students/Delete. cshtml*):

[!code-cshtml[Main](intro/samples/cu30/Pages/Students/Delete.cshtml?highlight=10)]

Eseguire l'app ed eliminare uno studente per testare la pagina Delete.

## <a name="next-steps"></a>Passaggi successivi

> [!div class="step-by-step"]
> [Esercitazione precedente](xref:data/ef-rp/intro) 
>  [Esercitazione successiva](xref:data/ef-rp/sort-filter-page)

::: moniker-end

::: moniker range="< aspnetcore-3.0"

In questa esercitazione viene esaminato e personalizzato il codice CRUD (Create, Read, Update, Delete) con scaffolding.

Per ridurre la complessità e mantenere queste esercitazioni incentrate su EF Core, viene usato il codice EF Core nei modelli di pagina. Alcuni sviluppatori usano un livello di servizio o un modello di repository in per creare un livello di astrazione tra l'interfaccia utente ( Razor pagine) e il livello di accesso ai dati.

In questa esercitazione vengono esaminate le pagine create, Edit, DELETE e Details Razor nella cartella *students* .

Il codice con scaffolding usa il modello seguente per le pagine Create, Edit e Delete:

* Ottenere e visualizzare i dati richiesti con il metodo HTTP GET `OnGetAsync`.
* Salvare le modifiche apportate ai dati con il metodo HTTP POST `OnPostAsync`.

Le pagine Index e Details ottengono e visualizzano i dati richiesti con il metodo HTTP GET `OnGetAsync`

## <a name="singleordefaultasync-vs-firstordefaultasync"></a>`SingleOrDefaultAsync` e `FirstOrDefaultAsync`

Il codice generato usa [FirstOrDefaultAsync](/dotnet/api/microsoft.entityframeworkcore.entityframeworkqueryableextensions.firstordefaultasync#Microsoft_EntityFrameworkCore_EntityFrameworkQueryableExtensions_FirstOrDefaultAsync__1_System_Linq_IQueryable___0__System_Threading_CancellationToken_), che in genere è preferibile rispetto a [SingleOrDefaultAsync](/dotnet/api/microsoft.entityframeworkcore.entityframeworkqueryableextensions.singleordefaultasync#Microsoft_EntityFrameworkCore_EntityFrameworkQueryableExtensions_SingleOrDefaultAsync__1_System_Linq_IQueryable___0__System_Linq_Expressions_Expression_System_Func___0_System_Boolean___System_Threading_CancellationToken_).

 `FirstOrDefaultAsync` è più efficace di `SingleOrDefaultAsync` nel recupero di una sola entità:

* A meno che il codice non necessiti di verificare che non sia presente più di un'entità restituita dalla query.
* `SingleOrDefaultAsync` recupera una maggior quantità di dati ed esegue operazioni non necessarie.
* `SingleOrDefaultAsync` genera un'eccezione se è presente più di un'entità che soddisfa la parte del filtro.
* `FirstOrDefaultAsync` non genera alcun elemento se è presente più di un'entità che soddisfa la parte del filtro.

<a name="FindAsync"></a>

### <a name="findasync"></a>FindAsync

Nella maggior parte del codice con scaffolding è possibile usare [FindAsync](/dotnet/api/microsoft.entityframeworkcore.dbcontext.findasync#Microsoft_EntityFrameworkCore_DbContext_FindAsync_System_Type_System_Object___) al posto di `FirstOrDefaultAsync`.

`FindAsync`:

* Individua un'entità con la chiave primaria. Se il contesto rileva un'entità con la chiave primaria, l'entità viene restituita senza una richiesta al database.
* È semplice e conciso.
* È ottimizzato per la ricerca di una singola entità.
* Può offrire migliori prestazioni, ma raramente negli scenari Web tipici.
* Usa in modo implicito [FirstAsync](/dotnet/api/microsoft.entityframeworkcore.entityframeworkqueryableextensions.firstasync#Microsoft_EntityFrameworkCore_EntityFrameworkQueryableExtensions_FirstAsync__1_System_Linq_IQueryable___0__System_Linq_Expressions_Expression_System_Func___0_System_Boolean___System_Threading_CancellationToken_) anziché [SingleAsync](/dotnet/api/microsoft.entityframeworkcore.entityframeworkqueryableextensions.singleasync#Microsoft_EntityFrameworkCore_EntityFrameworkQueryableExtensions_SingleAsync__1_System_Linq_IQueryable___0__System_Linq_Expressions_Expression_System_Func___0_System_Boolean___System_Threading_CancellationToken_).

Se si vuole usare `Include` per includere altre entità, l'uso di `FindAsync` non è più appropriato. Ciò significa che potrebbe essere necessario abbandonare `FindAsync` e passare a una query durante la creazione dell'app.

## <a name="customize-the-details-page"></a>Personalizzare la pagina Details

Passare alla pagina `Pages/Students`. I collegamenti **Edit**, **Details** e **Delete** vengono generati dall'[helper del tag di ancoraggio](xref:mvc/views/tag-helpers/builtin-th/anchor-tag-helper) nel file *Pages/Students/Index.cshtml*.

[!code-cshtml[](intro/samples/cu21/Pages/Students/Index1.cshtml?name=snippet)]

Eseguire l'app e selezionare un collegamento **Details**. L'URL è nel formato `http://localhost:5000/Students/Details?id=2`. L'ID studente viene passato tramite una stringa di query (`?id=2`).

Aggiornare le pagine modifica, dettagli ed Elimina Razor per usare il `"{id:int}"` modello di route. Modificare la direttiva page per ognuna di queste pagine da `@page` a `@page "{id:int}"`.

Una richiesta alla pagina con il modello di route "{id: int}" che **non** include il valore di route intero restituisce un errore HTTP 404 (Non trovato). Ad esempio, `http://localhost:5000/Students/Details` restituisce un errore 404. Per rendere l'ID facoltativo, aggiungere `?` al vincolo di route:

 ```cshtml
@page "{id:int?}"
```

Eseguire l'app, fare clic su un collegamento Details e verificare che l'URL passi l'ID come dati della route (`http://localhost:5000/Students/Details/2`).

Non modificare globalmente `@page` in `@page "{id:int}"` per non interrompere i collegamenti alle pagine Home e Create.

<!-- See https://github.com/aspnet/Scaffolding/issues/590 -->

### <a name="add-related-data"></a>Aggiungere dati correlati

Il codice con scaffolding della pagina Students Index non include la proprietà `Enrollments`. In questa sezione i contenuti della raccolta `Enrollments` sono visualizzati nella pagina Details.

Il metodo `OnGetAsync` del file *Pages/Students/Details.cshtml.cs* usa il metodo `FirstOrDefaultAsync` per recuperare una singola entità `Student`. Aggiungere il codice evidenziato seguente:

[!code-csharp[](intro/samples/cu21/Pages/Students/Details.cshtml.cs?name=snippet_Details&highlight=8-12)]

I metodi [Include](/dotnet/api/microsoft.entityframeworkcore.entityframeworkqueryableextensions.include) e [ThenInclude](/dotnet/api/microsoft.entityframeworkcore.entityframeworkqueryableextensions.theninclude#Microsoft_EntityFrameworkCore_EntityFrameworkQueryableExtensions_ThenInclude__3_Microsoft_EntityFrameworkCore_Query_IIncludableQueryable___0_System_Collections_Generic_IEnumerable___1___System_Linq_Expressions_Expression_System_Func___1___2___) fanno in modo che il contesto carichi la proprietà di navigazione `Student.Enrollments` e la proprietà di navigazione `Enrollment.Course` all'interno di ogni iscrizione. Questi metodi vengono esaminati in dettaglio nell'esercitazione sulla lettura dei dati correlati.

Il metodo [AsNoTracking](/dotnet/api/microsoft.entityframeworkcore.entityframeworkqueryableextensions.asnotracking#Microsoft_EntityFrameworkCore_EntityFrameworkQueryableExtensions_AsNoTracking__1_System_Linq_IQueryable___0__) migliora le prestazioni negli scenari in cui le entità restituite non vengono aggiornate nel contesto corrente. `AsNoTracking` è descritto più avanti in questa esercitazione.

### <a name="display-related-enrollments-on-the-details-page"></a>Visualizzare le registrazioni correlate nella pagina Details

Aprire *Pages/Students/Details.cshtml*. Per visualizzare un elenco delle registrazioni, aggiungere il codice evidenziato seguente:

[!code-cshtml[](intro/samples/cu21/Pages/Students/Details.cshtml?highlight=32-53)]

Se dopo aver incollato il codice il rientro è errato, premere CTRL-K-D per correggerlo.

Il codice precedente esegue il ciclo nelle entità nella proprietà di navigazione `Enrollments`. Per ogni registrazione, il codice visualizza il titolo del corso e il voto. Il titolo del corso viene recuperato dall'entità Course memorizzata nella proprietà di navigazione `Course` dell'entità Enrollments.

Eseguire l'app, selezionare la scheda **Students** e fare clic sul collegamento **Details** relativo a uno studente. Viene visualizzato l'elenco dei corsi e dei voti dello studente selezionato.

## <a name="update-the-create-page"></a>Aggiornare la pagina Create

Aggiornare il metodo `OnPostAsync` in *Pages/Students/Create.cshtml.cs* con il codice seguente:

[!code-csharp[](intro/samples/cu21/Pages/Students/Create.cshtml.cs?name=snippet_OnPostAsync)]

<a name="TryUpdateModelAsync"></a>

### <a name="tryupdatemodelasync"></a>TryUpdateModelAsync

Esaminare il codice [TryUpdateModelAsync](/dotnet/api/microsoft.aspnetcore.mvc.controllerbase.tryupdatemodelasync#Microsoft_AspNetCore_Mvc_ControllerBase_TryUpdateModelAsync_System_Object_System_Type_System_String_):

[!code-csharp[](intro/samples/cu21/Pages/Students/Create.cshtml.cs?name=snippet_TryUpdateModelAsync)]

Nel codice precedente `TryUpdateModelAsync<Student>` tenta di aggiornare l'oggetto `emptyStudent` usando i valori di modulo inviati dalla proprietà [PageContext](/dotnet/api/microsoft.aspnetcore.mvc.razorpages.pagemodel.pagecontext#Microsoft_AspNetCore_Mvc_RazorPages_PageModel_PageContext) in [PageModel](/dotnet/api/microsoft.aspnetcore.mvc.razorpages.pagemodel). `TryUpdateModelAsync` aggiorna solo le proprietà elencate (`s => s.FirstMidName, s => s.LastName, s => s.EnrollmentDate`).

Nell'esempio precedente:

* Il secondo argomento (`"student", // Prefix`) è il prefisso usato per cercare i valori. Non viene fatta distinzione tra maiuscole e minuscole.
* I valori di modulo inviati sono convertiti nei tipi nel modello `Student` usando l'[associazione di modelli](xref:mvc/models/model-binding).

<a id="overpost"></a>

### <a name="overposting"></a>Overposting

L'uso di `TryUpdateModel` per l'aggiornamento dei campi con i valori inviati è una procedura di sicurezza consigliata poiché impedisce l'overposting. Ad esempio, si supponga che l'entità Student includa una proprietà `Secret` che la pagina Web non deve aggiornare o aggiungere:

[!code-csharp[](intro/samples/cu21/Models/StudentZsecret.cs?name=snippet_Intro&highlight=7)]

Anche se l'app non ha un `Secret` campo nella pagina di creazione/aggiornamento Razor , un hacker può impostare il `Secret` valore tramite l'overposting. Un hacker potrebbe usare uno strumento come Fiddler oppure scrivere codice JavaScript per inviare un valore di modulo `Secret`. Il codice originale non limita i campi usati dallo strumento di associazione di modelli durante la creazione di un'istanza di Student.

Qualsiasi valore specificato dall'hacker per il campo di modulo `Secret` viene aggiornato nel database. L'immagine seguente illustra lo strumento Fiddler che aggiunge il campo `Secret` (con il valore "OverPost") ai valori di modulo inviati.

![Fiddler aggiunge il campo Secret](../ef-mvc/crud/_static/fiddler.png)

Il valore "OverPost" è stato aggiunto alla proprietà `Secret` della riga inserita. La progettazione app non prevedeva in alcun modo che la proprietà `Secret` venisse impostata con la pagina Create.

<a name="vm"></a>

### <a name="view-model"></a>Visualizzare il modello

Un modello di visualizzazione contiene in genere un subset delle proprietà incluse nel modello usato dall'applicazione. Il modello di applicazione è spesso chiamato modello di dominio. Il modello di dominio contiene in genere tutte le proprietà richieste dall'entità corrispondente nel database. Il modello di visualizzazione contiene solo le proprietà necessarie per il livello di interfaccia utente, ad esempio la pagina Create. Oltre al modello di visualizzazione, alcune app usano un modello di associazione o un modello di input per passare i dati tra la Razor classe del modello di pagina delle pagine e il browser. Si consideri il modello di visualizzazione `Student` seguente:

[!code-csharp[](intro/samples/cu21/Models/StudentVM.cs)]

I modelli di visualizzazione rappresentano un altro metodo per impedire l'overposting. Il modello di visualizzazione contiene solo le proprietà da visualizzare o aggiornare.

Il codice seguente usa il modello di visualizzazione `StudentVM` per creare un nuovo studente:

[!code-csharp[](intro/samples/cu21/Pages/Students/CreateVM.cshtml.cs?name=snippet_OnPostAsync)]

Il metodo [SetValues](/dotnet/api/microsoft.entityframeworkcore.changetracking.propertyvalues.setvalues#Microsoft_EntityFrameworkCore_ChangeTracking_PropertyValues_SetValues_System_Object_) imposta i valori di questo oggetto leggendo i valori di un altro oggetto [PropertyValues](/dotnet/api/microsoft.entityframeworkcore.changetracking.propertyvalues). `SetValues` usa la corrispondenza dei nomi di proprietà. Poiché il tipo di modello di visualizzazione non deve essere correlato al tipo di modello, è sufficiente che abbia proprietà corrispondenti.

Se si usa `StudentVM` è necessario che [CreateVM.cshtml](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/data/ef-rp/intro/samples/cu21/Pages/Students/CreateVM.cshtml) venga aggiornato per l'uso di `StudentVM` anziché `Student`.

Nelle Razor pagine la `PageModel` classe derivata è il modello di visualizzazione.

## <a name="update-the-edit-page"></a>Aggiornare la pagina Edit (Modifica)

Aggiornare il modello di pagina per la pagina Edit. Le modifiche principali sono evidenziate:

[!code-csharp[](intro/samples/cu21/Pages/Students/Edit.cshtml.cs?name=snippet_OnPostAsync&highlight=20,36)]

Le modifiche al codice sono simili alla pagina Create con alcune eccezioni:

* `OnPostAsync` include un parametro `id` facoltativo.
* Lo studente corrente viene recuperato dal database senza creare uno studente vuoto.
* `FirstOrDefaultAsync` è stato sostituito con [FindAsync](/dotnet/api/microsoft.entityframeworkcore.dbset-1.findasync). `FindAsync` è una scelta ottimale quando si seleziona un'entità dalla chiave primaria. Per altre informazioni, vedere [FindAsync](#FindAsync).

### <a name="test-the-edit-and-create-pages"></a>Testare le pagine Edit e Create

Creare e modificare alcune entità studente.

## <a name="entity-states"></a>Stati di entità

Il contesto del database tiene traccia della sincronizzazione delle entità in memoria con le righe corrispondenti nel database. Le informazioni di sincronizzazione del contesto del database determinano le operazioni eseguite quando viene chiamato [SaveChangesAsync](/dotnet/api/microsoft.entityframeworkcore.dbcontext.savechangesasync#Microsoft_EntityFrameworkCore_DbContext_SaveChangesAsync_System_Threading_CancellationToken_). Ad esempio, quando una nuova entità viene passata al metodo [AddAsync](/dotnet/api/microsoft.entityframeworkcore.dbcontext.addasync), lo stato dell'entità viene impostato su [Added](/dotnet/api/microsoft.entityframeworkcore.entitystate#Microsoft_EntityFrameworkCore_EntityState_Added). Quando viene chiamato `SaveChangesAsync`, il contesto del database genera un comando SQL INSERT.

Un'entità può essere in uno degli [stati seguenti](/dotnet/api/microsoft.entityframeworkcore.entitystate):

* `Added`: l'entità non esiste ancora nel database. Il metodo `SaveChanges` genera un'istruzione INSERT.

* `Unchanged`: non è necessario salvare alcuna modifica con questa entità. Un'entità ha questo stato quando viene letta dal database.

* `Modified`: sono stati modificati alcuni o tutti i valori di proprietà dell'entità. Il metodo `SaveChanges` genera un'istruzione UPDATE.

* `Deleted`: l'entità è stata contrassegnata per l'eliminazione. Il metodo `SaveChanges` genera un'istruzione DELETE.

* `Detached`: l'entità non viene registrata dal contesto del database.

In un'applicazione desktop le modifiche dello stato vengono in genere impostate automaticamente. Viene letta un'entità, vengono apportate le modifiche e lo stato dell'entità viene modificato automaticamente in `Modified`. La chiamata di `SaveChanges` genera un'istruzione SQL UPDATE che aggiorna solo le proprietà modificate.

In un'app Web il `DbContext` che legge un'entità e visualizza i dati viene eliminato dopo il rendering di una pagina. Quando viene chiamato il metodo `OnPostAsync` di una pagina, viene effettuata una nuova richiesta Web con una nuova istanza di `DbContext`. La rilettura dell'entità nel nuovo contesto simula l'elaborazione desktop.

## <a name="update-the-delete-page"></a>Aggiornare la pagina Delete (Elimina)

In questa sezione viene aggiunto un codice per implementare un messaggio di errore personalizzato quando la chiamata a `SaveChanges` ha esito negativo. Aggiungere una stringa per i possibili messaggi di errore:

[!code-csharp[](intro/samples/cu21/Pages/Students/Delete.cshtml.cs?name=snippet1&highlight=12)]

Sostituire il metodo `OnGetAsync` con il codice seguente:

[!code-csharp[](intro/samples/cu21/Pages/Students/Delete.cshtml.cs?name=snippet_OnGetAsync&highlight=1,9,17-20)]

Il codice precedente contiene il parametro facoltativo `saveChangesError`. `saveChangesError` indica se il metodo è stato chiamato dopo un errore di eliminazione dell'oggetto Student. L'operazione di eliminazione potrebbe non riuscire a causa di problemi di rete temporanei. Gli errori di rete temporanei sono più probabili nel cloud. `saveChangesError` ha valore false quando `OnGetAsync` della pagina Delete viene chiamato dall'interfaccia utente. Quando `OnGetAsync` viene chiamato da `OnPostAsync` (perché l'operazione di eliminazione ha avuto esito negativo), il parametro `saveChangesError` ha valore true.

### <a name="the-delete-pages-onpostasync-method"></a>Metodo OnPostAsync delle pagine Delete

Sostituire `OnPostAsync` con il codice seguente:

[!code-csharp[](intro/samples/cu21/Pages/Students/Delete.cshtml.cs?name=snippet_OnPostAsync)]

Il codice precedente recupera l'entità selezionata, quindi chiama il metodo [Remove](/dotnet/api/microsoft.entityframeworkcore.dbcontext.remove#Microsoft_EntityFrameworkCore_DbContext_Remove_System_Object_) per impostare lo stato dell'entità su `Deleted`. Quando viene chiamato `SaveChanges`, viene generato un comando SQL DELETE. Se `Remove` ha esito negativo:

* Viene rilevata l'eccezione di database.
* Il metodo `OnGetAsync` delle pagine Delete viene chiamato con `saveChangesError=true`.

### <a name="update-the-delete-razor-page"></a>Aggiornare la Razor pagina Elimina

Aggiungere il messaggio di errore evidenziato seguente alla Razor pagina Elimina.
<!--
[!code-cshtml[](intro/samples/cu21/Pages/Students/Delete.cshtml?name=snippet&highlight=11)]
-->
[!code-cshtml[](intro/samples/cu21/Pages/Students/Delete.cshtml?range=1-13&highlight=10)]

Eseguire il test di Delete.

## <a name="common-errors"></a>Errori comuni

Students/Index o altri collegamenti non funzionano:

Verificare che la Razor pagina contenga la `@page` direttiva corretta. La pagina students/index, ad esempio, Razor **non** deve contenere un modello di route:

```cshtml
@page "{id:int}"
```

Ogni Razor pagina deve includere la `@page` direttiva.



## <a name="additional-resources"></a>Risorse aggiuntive

* [Versione YouTube dell'esercitazione](https://www.youtube.com/watch?v=K4X1MT2jt6o)

> [!div class="step-by-step"]
> [Precedente](xref:data/ef-rp/intro) 
>  [Avanti](xref:data/ef-rp/sort-filter-page)

::: moniker-end
