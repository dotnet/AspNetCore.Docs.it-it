---
title: 'Esercitazione: informazioni sugli scenari avanzati-MVC ASP.NET con EF Core'
description: Questa esercitazione presenta argomenti utili dopo aver appreso le nozioni di base sullo sviluppo di applicazioni Web ASP.NET Core che usano Entity Framework Core.
author: rick-anderson
ms.author: riande
ms.custom: mvc
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
uid: data/ef-mvc/advanced
ms.openlocfilehash: 135881bfb4137bc81bd0f7250f1ce191cab13dcd
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102586839"
---
# <a name="tutorial-learn-about-advanced-scenarios---aspnet-mvc-with-ef-core"></a>Esercitazione: informazioni sugli scenari avanzati-MVC ASP.NET con EF Core

Nell'esercitazione precedente è stata implementata l'ereditarietà tabella per gerarchia. Questa esercitazione presenta diversi argomenti che è utile tenere presente dopo aver appreso le nozioni di base sullo sviluppo di applicazioni Web ASP.NET Core che usano Entity Framework Core.

In questa esercitazione:

> [!div class="checklist"]
> * Eseguire query SQL non elaborate
> * Chiamare una query per restituire le entità
> * Chiamare una query per restituire altri tipi
> * Chiamare una query di aggiornamento
> * Esaminare le query SQL
> * Creare un livello di astrazione
> * Scoprire di più sul rilevamento automatico delle modifiche
> * Scoprire di più sul codice sorgente e i piani di sviluppo di EF Core
> * Imparare a usare LINQ dinamico per semplificare il codice

## <a name="prerequisites"></a>Prerequisiti

* [Implementare l'ereditarietà](inheritance.md)

## <a name="perform-raw-sql-queries"></a>Eseguire query SQL non elaborate

Uno dei vantaggi dell'utilizzo di Entity Framework è la mancanza di un collegamento troppo stretto del codice a un particolare metodo di archiviazione dei dati. Le query e i comandi SQL vengono generati automaticamente e non è necessario scriverli. Esistono tuttavia alcuni scenari particolari in cui è necessario eseguire query SQL specifiche create manualmente. Per questi scenari, l'API Code First di Entity Framework include metodi che consentono di passare i comandi SQL direttamente al database. In EF Core 1.0 sono possibili le opzioni seguenti:

* Usare il metodo `DbSet.FromSql` per le query che restituiscono tipi di entità. Gli oggetti restituiti devono essere del tipo previsto dall'oggetto `DbSet` e vengono registrati automaticamente dal contesto del database a meno che non [venga disattivata la registrazione](crud.md#no-tracking-queries).

* Usare `Database.ExecuteSqlCommand` per i comandi non query.

Se è necessario eseguire una query che restituisca i tipi che non sono entità, è possibile usare ADO.NET con la connessione di database fornita da EF. I dati restituiti non vengono registrati dal contesto di database, anche se il metodo viene usato per recuperare i tipi di entità.

Come avviene quando si eseguono comandi SQL in un'applicazione Web, è necessario adottare delle precauzioni per proteggere il sito dagli attacchi SQL injection. A questo scopo è possibile usare query parametrizzate per assicurarsi che le stringhe inviate da una pagina Web non possano essere interpretate come comandi SQL. In questa esercitazione verranno usate query parametrizzate quando l'input dell'utente viene integrato in una query.

## <a name="call-a-query-to-return-entities"></a>Chiamare una query per restituire le entità

La classe `DbSet<TEntity>` offre un metodo che è possibile usare per eseguire una query che restituisce un'entità di tipo `TEntity`. Per osservare questo funzionamento viene modificato il codice nel metodo `Details` del controller Department.

In *DepartmentsController.cs* nel metodo `Details` sostituire il codice che recupera un reparto con una chiamata al metodo `FromSql`, come illustrato nel codice evidenziato seguente:

[!code-csharp[](intro/samples/cu/Controllers/DepartmentsController.cs?name=snippet_RawSQL&highlight=8,9,10)]

Per verificare il corretto funzionamento del nuovo codice, selezionare la scheda **Departments** e quindi **Dettagli** per uno dei reparti.

![Dettagli del reparto](advanced/_static/department-details.png)

## <a name="call-a-query-to-return-other-types"></a>Chiamare una query per restituire altri tipi

In precedenza è stata creata una griglia delle statistiche degli studenti per la pagina About che visualizza il numero di studenti per ogni data di registrazione. I dati sono stati ottenuti dal set di entità Students (`_context.Students`) ed è stato usato LINQ per proiettare i risultati in un elenco degli oggetti del modello di visualizzazione `EnrollmentDateGroup`. Si supponga di voler scrivere un codice SQL anziché usare LINQ. A tale scopo è necessario eseguire una query SQL che restituisca elementi diversi dagli oggetti entità. In EF Core 1.0 è possibile scrivere codice ADO.NET e ottenere la connessione di database da EF.

In *HomeController.cs* sostituire il metodo `About` con il codice seguente:

[!code-csharp[](intro/samples/cu/Controllers/HomeController.cs?name=snippet_UseRawSQL&highlight=3-32)]

Aggiungere un'istruzione using:

[!code-csharp[](intro/samples/cu/Controllers/HomeController.cs?name=snippet_Usings2)]

Eseguire l'app e passare alla pagina About. Vengono visualizzati gli stessi dati visualizzati in precedenza.

![Pagina About (Informazioni)](advanced/_static/about.png)

## <a name="call-an-update-query"></a>Chiamare una query di aggiornamento

Si supponga che gli amministratori di Contoso University vogliano eseguire modifiche globali nel database, ad esempio modificare il numero di crediti di ogni corso. Se il numero di corsi dell'università è elevato, potrebbe non essere utile recuperarli tutti come entità e modificarli singolarmente. In questa sezione viene implementata una pagina Web che consente all'utente di specificare un fattore in base al quale modificare il numero di crediti di tutti i corsi e viene apportata la modifica eseguendo un'istruzione SQL UPDATE. La pagina Web apparirà come segue:

![Pagina di aggiornamento dei crediti dei corsi](advanced/_static/update-credits.png)

In *CoursesController.cs* aggiungere i metodi UpdateCourseCredits per HttpGet e HttpPost:

[!code-csharp[](intro/samples/cu/Controllers/CoursesController.cs?name=snippet_UpdateGet)]

[!code-csharp[](intro/samples/cu/Controllers/CoursesController.cs?name=snippet_UpdatePost)]

Quando il controller elabora una richiesta HttpGet, non viene restituito alcun elemento in `ViewData["RowsAffected"]` e la visualizzazione mostra una casella di testo vuota e un pulsante di invio, come illustrato nella figura precedente.

Quando viene fatto clic sul pulsante **Aggiorna**, viene chiamato il metodo HttpPost e il moltiplicatore ha il valore immesso nella casella di testo. Il codice esegue quindi l'SQL che aggiorna i corsi e restituisce il numero di righe interessate nella visualizzazione in `ViewData`. Quando riceve un valore `RowsAffected`, la visualizzazione mostra il numero di righe aggiornate.

In **Esplora soluzioni** fare clic con il pulsante destro del mouse sulla cartella *Views/Courses* e quindi fare clic su **Aggiungi > Nuovo elemento**.

Nella finestra di dialogo **Aggiungi nuovo elemento** fare clic su **ASP.NET Core** in **installato** nel riquadro sinistro, fare clic su **Razor Visualizza** e denominare la nuova vista *UpdateCourseCredits. cshtml*.

In *Views/Courses/UpdateCourseCredits.cshtml* sostituire il codice del modello con il codice seguente:

[!code-cshtml[](intro/samples/cu/Views/Courses/UpdateCourseCredits.cshtml)]

Eseguire il metodo `UpdateCourseCredits` selezionando la scheda **Courses**, quindi aggiungendo "/UpdateCourseCredits" alla fine dell'URL nella barra degli indirizzi del browser (ad esempio: `http://localhost:5813/Courses/UpdateCourseCredits`). Immettere un numero nella casella di testo:

![Pagina di aggiornamento dei crediti dei corsi](advanced/_static/update-credits.png)

Fare clic su **Update**. Viene visualizzato il numero di righe interessate:

![Righe interessate della pagina Update Course Credits](advanced/_static/update-credits-rows-affected.png)

Fare clic su **Torna all'elenco** per visualizzare l'elenco dei corsi con il numero di crediti modificato.

Tenere presente che il codice di produzione assicurerà che gli aggiornamenti restituiscano sempre dati validi. Il codice semplificato illustrato potrebbe moltiplicare il numero di crediti e restituire numeri maggiori di 5. (La `Credits` proprietà ha un `[Range(0, 5)]` attributo). La query di aggiornamento funzionerebbe ma i dati non validi potrebbero provocare risultati imprevisti in altre parti del sistema che presuppongono che il numero di crediti sia pari o inferiore a 5.

Per altre informazioni sulle query SQL non elaborate, vedere [Query SQL non elaborate](/ef/core/querying/raw-sql).

## <a name="examine-sql-queries"></a>Esaminare le query SQL

A volte può essere utile visualizzare le query SQL inviate al database. La funzionalità di registrazione incorporata di ASP.NET Core viene usata automaticamente da EF Core per creare i log contenenti il codice SQL di query e aggiornamenti. In questa sezione vengono presentati alcuni esempi di registrazione SQL.

Aprire *StudentsController.cs* e nel metodo `Details` impostare un punto di interruzione nell'istruzione `if (student == null)`.

Eseguire l'app in modalità di debug e passare alla pagina Details di uno studente.

Passare alla finestra **Output** con l'output del debug per visualizzare la query:

```
Microsoft.EntityFrameworkCore.Database.Command:Information: Executed DbCommand (56ms) [Parameters=[@__id_0='?'], CommandType='Text', CommandTimeout='30']
SELECT TOP(2) [s].[ID], [s].[Discriminator], [s].[FirstName], [s].[LastName], [s].[EnrollmentDate]
FROM [Person] AS [s]
WHERE ([s].[Discriminator] = N'Student') AND ([s].[ID] = @__id_0)
ORDER BY [s].[ID]
Microsoft.EntityFrameworkCore.Database.Command:Information: Executed DbCommand (122ms) [Parameters=[@__id_0='?'], CommandType='Text', CommandTimeout='30']
SELECT [s.Enrollments].[EnrollmentID], [s.Enrollments].[CourseID], [s.Enrollments].[Grade], [s.Enrollments].[StudentID], [e.Course].[CourseID], [e.Course].[Credits], [e.Course].[DepartmentID], [e.Course].[Title]
FROM [Enrollment] AS [s.Enrollments]
INNER JOIN [Course] AS [e.Course] ON [s.Enrollments].[CourseID] = [e.Course].[CourseID]
INNER JOIN (
    SELECT TOP(1) [s0].[ID]
    FROM [Person] AS [s0]
    WHERE ([s0].[Discriminator] = N'Student') AND ([s0].[ID] = @__id_0)
    ORDER BY [s0].[ID]
) AS [t] ON [s.Enrollments].[StudentID] = [t].[ID]
ORDER BY [t].[ID]
```

Si noti che il codice SQL seleziona un massimo di 2 righe (`TOP(2)`) dalla tabella Person. Il metodo `SingleOrDefaultAsync` non viene risolto in 1 riga nel server. Ecco perché:

* Se la query restituisse più righe, il metodo restituirebbe un valore Null.
* Per determinare se la query restituirà più righe, EF deve controllare se restituisce almeno 2 righe.

Si noti che non è necessario usare la modalità di debug e arrestarsi in un punto di interruzione per visualizzare l'output di registrazione nella finestra **Output**. Si tratta soltanto di un metodo pratico per arrestare la registrazione nel punto in cui si desidera visualizzare l'output. Se non si esegue questa operazione, la registrazione continua ed è necessario scorrere indietro per individuare le parti desiderate.

## <a name="create-an-abstraction-layer"></a>Creare un livello di astrazione

Molti sviluppatori scrivono codice per implementare i modelli di repository e unità di lavoro come wrapper per il codice usato con Entity Framework. Questi modelli sono progettati per la creazione di un livello di astrazione tra il livello di accesso ai dati e il livello della logica di business di un'applicazione. L'implementazione di questi modelli può essere utile per isolare l'applicazione dalle modifiche nell'archivio dati e può semplificare il testing unità automatizzato o lo sviluppo basato su test (TDD). Tuttavia, la scrittura di codice aggiuntivo per l'implementazione di questi modelli non è sempre la scelta migliore per le applicazioni che usano EF, per diverse ragioni:

* La classe del contesto EF isola il codice dal codice specifico dell'archivio dati.

* La classe del contesto di EF può svolgere la funzione di classe di unità di lavoro per gli aggiornamenti di database eseguiti usando EF.

* EF include funzionalità che consentono di implementare lo sviluppo basato su test senza scrivere codice di repository.

Per informazioni su come implementare i modelli di repository e unità di lavoro, vedere [la versione Entity Framework 5 di questa serie di esercitazioni](/aspnet/mvc/overview/older-versions/getting-started-with-ef-5-using-mvc-4/implementing-the-repository-and-unit-of-work-patterns-in-an-asp-net-mvc-application).

Entity Framework Core implementa un provider di database in memoria che è possibile usare per il test. Per altre informazioni, vedere [Test con InMemory](/ef/core/miscellaneous/testing/in-memory).

## <a name="automatic-change-detection"></a>Rilevamento automatico delle modifiche

Entity Framework determina come è stata modificata un'entità (e di conseguenza gli aggiornamenti da inviare al database) confrontando i valori correnti di un'entità con i valori originali. I valori originali vengono memorizzati quando viene eseguita una query sull'entità o quando viene collegata. I metodi che causano il rilevamento automatico delle modifiche includono:

* DbContext.SaveChanges

* DbContext.Entry

* ChangeTracker.Entries

Se viene registrato un numero elevato di entità e uno solo di questi metodi viene chiamato più volte in un ciclo, è possibile ottenere miglioramenti significativi delle prestazioni disattivando temporaneamente il rilevamento automatico delle modifiche usando la proprietà `ChangeTracker.AutoDetectChangesEnabled`. Ad esempio:

```csharp
_context.ChangeTracker.AutoDetectChangesEnabled = false;
```

## <a name="ef-core-source-code-and-development-plans"></a>Codice sorgente e piani di sviluppo di EF Core

Il Entity Framework Core origine si trova in [https://github.com/dotnet/efcore](https://github.com/dotnet/efcore) . Il repository di EF Core contiene le compilazioni notturne, la gestione dei problemi, le specifiche delle funzionalità, le note delle riunioni di progettazione e [la roadmap per lo sviluppo futuro](https://github.com/dotnet/efcore/wiki/Roadmap). È possibile archiviare o individuare i bug e contribuire.

Sebbene il codice sorgente sia disponibile, Entity Framework Core è completamente supportato come prodotto Microsoft. Il team di Microsoft Entity Framework controlla i contributi accettati ed esegue il test di tutte le modifiche al codice per garantire la qualità di ogni rilascio.

## <a name="reverse-engineer-from-existing-database"></a>Eseguire il reverse engineering dal database esistente

Per eseguire il reverse engineering di un modello di dati contenente classi di entità da un database esistente, usare il comando [scaffold-dbcontext](/ef/core/miscellaneous/cli/powershell#scaffold-dbcontext). Vedere l'[esercitazione introduttiva](/ef/core/get-started/aspnetcore/existing-db).

<a id="dynamic-linq"></a>

## <a name="use-dynamic-linq-to-simplify-code"></a>Usare LINQ dinamico per semplificare il codice

La [terza esercitazione di questa serie](sort-filter-page.md) descrive come scrivere codice LINQ impostando come hardcoded i nomi di colonna in un'istruzione `switch`. Se la selezione viene effettuata tra due colonne, il funzionamento è corretto, ma in presenza di numerose colonne il codice potrebbe diventare troppo lungo. Per risolvere il problema, è possibile usare il metodo `EF.Property` per specificare il nome della proprietà come stringa. Per provare questo approccio, sostituire il metodo `Index` in `StudentsController` con il codice seguente.

[!code-csharp[](intro/samples/cu/Controllers/StudentsController.cs?name=snippet_DynamicLinq)]

## <a name="acknowledgments"></a>Ringraziamenti

Tom Dykstra e Rick Anderson (twitter @RickAndMSFT) hanno scritto questa esercitazione. Rowan Miller, Diego Vega e altri membri del team Entity Framework hanno offerto supporto per le revisioni del codice e per il debug dei problemi sorti durante la scrittura del codice per le esercitazioni. John Parente e Paul Goldman hanno collaborato all'aggiornamento dell'esercitazione per ASP.NET Core 2.2.

<a id="common-errors"></a>

## <a name="troubleshoot-common-errors"></a>Risolvere gli errori comuni

### <a name="contosouniversitydll-used-by-another-process"></a>ContosoUniversity.dll usata da un altro processo

Messaggio di errore:

> Impossibile aprire '...bin\Debug\netcoreapp1.0\ContosoUniversity.dll' per la scrittura -- 'Il processo non può accedere al file '...\bin\Debug\netcoreapp1.0\ContosoUniversity.dll' perché è in uso da un altro processo.

Soluzione:

Arrestare il sito in IIS Express. Passare alla barra delle applicazioni di Windows, trovare IIS Express e fare clic con il pulsante destro del mouse sull'icona, selezionare il sito Contoso University e quindi fare clic su **Arresta sito**.

### <a name="migration-scaffolded-with-no-code-in-up-and-down-methods"></a>Migrazione con scaffolding senza codice nei metodi Up e Down

Possibile causa:

I comandi CLI di EF non chiudono e salvano automaticamente i file di codice. Se sono presenti modifiche non salvate quando si esegue il comando `migrations add`, EF non trova le modifiche.

Soluzione:

Eseguire il comando `migrations remove`, salvare le modifiche al codice e rieseguire il comando `migrations add`.

### <a name="errors-while-running-database-update"></a>Errori durante l'esecuzione dell'aggiornamento del database

Quando si apportano modifiche allo schema in un database con dati esistenti è possibile che si riscontrino altri errori. Se si verificano errori di migrazione che non si riesce a risolvere, è possibile modificare il nome del database nella stringa di connessione o eliminare il database. Un nuovo database non contiene dati di cui eseguire la migrazione e ci sono maggiori probabilità che il comando update-database venga completato senza errori.

L'approccio più semplice consiste nel rinominare il database in *appsettings.json* . Alla successiva esecuzione di `database update`, verrà creato un nuovo database.

Per eliminare un database in SSOX, fare clic con il pulsante destro del mouse sul database, fare clic su **Elimina** e quindi nella finestra di dialogo **Elimina database** selezionare **Chiudi connessioni esistenti** e fare clic su **OK**.

Per eliminare un database tramite l'interfaccia CLI, eseguire il comando CLI `database drop`:

```dotnetcli
dotnet ef database drop
```

### <a name="error-locating-sql-server-instance"></a>Errore di individuazione dell'istanza di SQL Server

Messaggio di errore:

> Si è verificato un errore di rete o specifico dell'istanza mentre veniva stabilita la connessione a SQL Server. Il server non è stato trovato o non è accessibile. Verificare che il nome dell'istanza sia corretto e che il server sia configurato in modo da consentire connessioni remote. (provider: interfacce di rete SQL, errore: 26 - Errore nell'individuazione del server/dell'istanza specificata)

Soluzione:

Controllare la stringa di connessione. Se il file di database è stato eliminato manualmente, modificare il nome del database nella stringa di costruzione per riniziare con un nuovo database.

## <a name="get-the-code"></a>Ottenere il codice

[Scaricare o visualizzare l'applicazione completata.](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/data/ef-mvc/intro/samples/cu-final)

## <a name="additional-resources"></a>Risorse aggiuntive

Per altre informazioni su EF Core, vedere la [documentazione di Entity Framework Core](/ef/core). È disponibile anche il libro [Entity Framework Core in Action](https://www.manning.com/books/entity-framework-core-in-action).

Per informazioni su come distribuire un'app Web, vedere <xref:host-and-deploy/index>.

Per informazioni su altri argomenti correlati ad ASP.NET Core MVC, ad esempio sull'autenticazione e l'autorizzazione, vedere <xref:index>.

## <a name="next-steps"></a>Passaggi successivi

In questa esercitazione:

> [!div class="checklist"]
> * Eseguire query SQL non elaborate
> * Chiamare una query per restituire le entità
> * Chiamare una query per restituire altri tipi
> * Chiamare una query di aggiornamento
> * Esaminare le query SQL
> * Creare un livello di astrazione
> * Scoprire di più sul rilevamento automatico delle modifiche
> * Scoprire di più sul codice sorgente e i piani di sviluppo di EF Core
> * Imparare a usare LINQ dinamico per semplificare il codice

Questa esercitazione completa la serie di esercitazioni sull'uso di Entity Framework Core in un'applicazione ASP.NET Core MVC. In questa serie di esercitazioni è stato usato un nuovo database; in alternativa, è possibile decompilare un modello di un database esistente.

> [!div class="nextstepaction"]
> [Esercitazione: EF Core con MVC, database esistente](/ef/core/get-started/aspnetcore/existing-db?toc=/aspnet/core/toc.json&bc=/aspnet/core/breadcrumb/toc.json)
