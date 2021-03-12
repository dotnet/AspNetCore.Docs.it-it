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
# <a name="tutorial-learn-about-advanced-scenarios---aspnet-mvc-with-ef-core"></a><span data-ttu-id="a6aba-103">Esercitazione: informazioni sugli scenari avanzati-MVC ASP.NET con EF Core</span><span class="sxs-lookup"><span data-stu-id="a6aba-103">Tutorial: Learn about advanced scenarios - ASP.NET MVC with EF Core</span></span>

<span data-ttu-id="a6aba-104">Nell'esercitazione precedente è stata implementata l'ereditarietà tabella per gerarchia.</span><span class="sxs-lookup"><span data-stu-id="a6aba-104">In the previous tutorial, you implemented table-per-hierarchy inheritance.</span></span> <span data-ttu-id="a6aba-105">Questa esercitazione presenta diversi argomenti che è utile tenere presente dopo aver appreso le nozioni di base sullo sviluppo di applicazioni Web ASP.NET Core che usano Entity Framework Core.</span><span class="sxs-lookup"><span data-stu-id="a6aba-105">This tutorial introduces several topics that are useful to be aware of when you go beyond the basics of developing ASP.NET Core web applications that use Entity Framework Core.</span></span>

<span data-ttu-id="a6aba-106">In questa esercitazione:</span><span class="sxs-lookup"><span data-stu-id="a6aba-106">In this tutorial, you:</span></span>

> [!div class="checklist"]
> * <span data-ttu-id="a6aba-107">Eseguire query SQL non elaborate</span><span class="sxs-lookup"><span data-stu-id="a6aba-107">Perform raw SQL queries</span></span>
> * <span data-ttu-id="a6aba-108">Chiamare una query per restituire le entità</span><span class="sxs-lookup"><span data-stu-id="a6aba-108">Call a query to return entities</span></span>
> * <span data-ttu-id="a6aba-109">Chiamare una query per restituire altri tipi</span><span class="sxs-lookup"><span data-stu-id="a6aba-109">Call a query to return other types</span></span>
> * <span data-ttu-id="a6aba-110">Chiamare una query di aggiornamento</span><span class="sxs-lookup"><span data-stu-id="a6aba-110">Call an update query</span></span>
> * <span data-ttu-id="a6aba-111">Esaminare le query SQL</span><span class="sxs-lookup"><span data-stu-id="a6aba-111">Examine SQL queries</span></span>
> * <span data-ttu-id="a6aba-112">Creare un livello di astrazione</span><span class="sxs-lookup"><span data-stu-id="a6aba-112">Create an abstraction layer</span></span>
> * <span data-ttu-id="a6aba-113">Scoprire di più sul rilevamento automatico delle modifiche</span><span class="sxs-lookup"><span data-stu-id="a6aba-113">Learn about Automatic change detection</span></span>
> * <span data-ttu-id="a6aba-114">Scoprire di più sul codice sorgente e i piani di sviluppo di EF Core</span><span class="sxs-lookup"><span data-stu-id="a6aba-114">Learn about EF Core source code and development plans</span></span>
> * <span data-ttu-id="a6aba-115">Imparare a usare LINQ dinamico per semplificare il codice</span><span class="sxs-lookup"><span data-stu-id="a6aba-115">Learn how to use dynamic LINQ to simplify code</span></span>

## <a name="prerequisites"></a><span data-ttu-id="a6aba-116">Prerequisiti</span><span class="sxs-lookup"><span data-stu-id="a6aba-116">Prerequisites</span></span>

* [<span data-ttu-id="a6aba-117">Implementare l'ereditarietà</span><span class="sxs-lookup"><span data-stu-id="a6aba-117">Implement Inheritance</span></span>](inheritance.md)

## <a name="perform-raw-sql-queries"></a><span data-ttu-id="a6aba-118">Eseguire query SQL non elaborate</span><span class="sxs-lookup"><span data-stu-id="a6aba-118">Perform raw SQL queries</span></span>

<span data-ttu-id="a6aba-119">Uno dei vantaggi dell'utilizzo di Entity Framework è la mancanza di un collegamento troppo stretto del codice a un particolare metodo di archiviazione dei dati.</span><span class="sxs-lookup"><span data-stu-id="a6aba-119">One of the advantages of using the Entity Framework is that it avoids tying your code too closely to a particular method of storing data.</span></span> <span data-ttu-id="a6aba-120">Le query e i comandi SQL vengono generati automaticamente e non è necessario scriverli.</span><span class="sxs-lookup"><span data-stu-id="a6aba-120">It does this by generating SQL queries and commands for you, which also frees you from having to write them yourself.</span></span> <span data-ttu-id="a6aba-121">Esistono tuttavia alcuni scenari particolari in cui è necessario eseguire query SQL specifiche create manualmente.</span><span class="sxs-lookup"><span data-stu-id="a6aba-121">But there are exceptional scenarios when you need to run specific SQL queries that you have manually created.</span></span> <span data-ttu-id="a6aba-122">Per questi scenari, l'API Code First di Entity Framework include metodi che consentono di passare i comandi SQL direttamente al database.</span><span class="sxs-lookup"><span data-stu-id="a6aba-122">For these scenarios, the Entity Framework Code First API includes methods that enable you to pass SQL commands directly to the database.</span></span> <span data-ttu-id="a6aba-123">In EF Core 1.0 sono possibili le opzioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="a6aba-123">You have the following options in EF Core 1.0:</span></span>

* <span data-ttu-id="a6aba-124">Usare il metodo `DbSet.FromSql` per le query che restituiscono tipi di entità.</span><span class="sxs-lookup"><span data-stu-id="a6aba-124">Use the `DbSet.FromSql` method for queries that return entity types.</span></span> <span data-ttu-id="a6aba-125">Gli oggetti restituiti devono essere del tipo previsto dall'oggetto `DbSet` e vengono registrati automaticamente dal contesto del database a meno che non [venga disattivata la registrazione](crud.md#no-tracking-queries).</span><span class="sxs-lookup"><span data-stu-id="a6aba-125">The returned objects must be of the type expected by the `DbSet` object, and they're automatically tracked by the database context unless you [turn tracking off](crud.md#no-tracking-queries).</span></span>

* <span data-ttu-id="a6aba-126">Usare `Database.ExecuteSqlCommand` per i comandi non query.</span><span class="sxs-lookup"><span data-stu-id="a6aba-126">Use the `Database.ExecuteSqlCommand` for non-query commands.</span></span>

<span data-ttu-id="a6aba-127">Se è necessario eseguire una query che restituisca i tipi che non sono entità, è possibile usare ADO.NET con la connessione di database fornita da EF.</span><span class="sxs-lookup"><span data-stu-id="a6aba-127">If you need to run a query that returns types that aren't entities, you can use ADO.NET with the database connection provided by EF.</span></span> <span data-ttu-id="a6aba-128">I dati restituiti non vengono registrati dal contesto di database, anche se il metodo viene usato per recuperare i tipi di entità.</span><span class="sxs-lookup"><span data-stu-id="a6aba-128">The returned data isn't tracked by the database context, even if you use this method to retrieve entity types.</span></span>

<span data-ttu-id="a6aba-129">Come avviene quando si eseguono comandi SQL in un'applicazione Web, è necessario adottare delle precauzioni per proteggere il sito dagli attacchi SQL injection.</span><span class="sxs-lookup"><span data-stu-id="a6aba-129">As is always true when you execute SQL commands in a web application, you must take precautions to protect your site against SQL injection attacks.</span></span> <span data-ttu-id="a6aba-130">A questo scopo è possibile usare query parametrizzate per assicurarsi che le stringhe inviate da una pagina Web non possano essere interpretate come comandi SQL.</span><span class="sxs-lookup"><span data-stu-id="a6aba-130">One way to do that is to use parameterized queries to make sure that strings submitted by a web page can't be interpreted as SQL commands.</span></span> <span data-ttu-id="a6aba-131">In questa esercitazione verranno usate query parametrizzate quando l'input dell'utente viene integrato in una query.</span><span class="sxs-lookup"><span data-stu-id="a6aba-131">In this tutorial you'll use parameterized queries when integrating user input into a query.</span></span>

## <a name="call-a-query-to-return-entities"></a><span data-ttu-id="a6aba-132">Chiamare una query per restituire le entità</span><span class="sxs-lookup"><span data-stu-id="a6aba-132">Call a query to return entities</span></span>

<span data-ttu-id="a6aba-133">La classe `DbSet<TEntity>` offre un metodo che è possibile usare per eseguire una query che restituisce un'entità di tipo `TEntity`.</span><span class="sxs-lookup"><span data-stu-id="a6aba-133">The `DbSet<TEntity>` class provides a method that you can use to execute a query that returns an entity of type `TEntity`.</span></span> <span data-ttu-id="a6aba-134">Per osservare questo funzionamento viene modificato il codice nel metodo `Details` del controller Department.</span><span class="sxs-lookup"><span data-stu-id="a6aba-134">To see how this works you'll change the code in the `Details` method of the Department controller.</span></span>

<span data-ttu-id="a6aba-135">In *DepartmentsController.cs* nel metodo `Details` sostituire il codice che recupera un reparto con una chiamata al metodo `FromSql`, come illustrato nel codice evidenziato seguente:</span><span class="sxs-lookup"><span data-stu-id="a6aba-135">In *DepartmentsController.cs*, in the `Details` method, replace the code that retrieves a department with a `FromSql` method call, as shown in the following highlighted code:</span></span>

[!code-csharp[](intro/samples/cu/Controllers/DepartmentsController.cs?name=snippet_RawSQL&highlight=8,9,10)]

<span data-ttu-id="a6aba-136">Per verificare il corretto funzionamento del nuovo codice, selezionare la scheda **Departments** e quindi **Dettagli** per uno dei reparti.</span><span class="sxs-lookup"><span data-stu-id="a6aba-136">To verify that the new code works correctly, select the **Departments** tab and then **Details** for one of the departments.</span></span>

![Dettagli del reparto](advanced/_static/department-details.png)

## <a name="call-a-query-to-return-other-types"></a><span data-ttu-id="a6aba-138">Chiamare una query per restituire altri tipi</span><span class="sxs-lookup"><span data-stu-id="a6aba-138">Call a query to return other types</span></span>

<span data-ttu-id="a6aba-139">In precedenza è stata creata una griglia delle statistiche degli studenti per la pagina About che visualizza il numero di studenti per ogni data di registrazione.</span><span class="sxs-lookup"><span data-stu-id="a6aba-139">Earlier you created a student statistics grid for the About page that showed the number of students for each enrollment date.</span></span> <span data-ttu-id="a6aba-140">I dati sono stati ottenuti dal set di entità Students (`_context.Students`) ed è stato usato LINQ per proiettare i risultati in un elenco degli oggetti del modello di visualizzazione `EnrollmentDateGroup`.</span><span class="sxs-lookup"><span data-stu-id="a6aba-140">You got the data from the Students entity set (`_context.Students`) and used LINQ to project the results into a list of `EnrollmentDateGroup` view model objects.</span></span> <span data-ttu-id="a6aba-141">Si supponga di voler scrivere un codice SQL anziché usare LINQ.</span><span class="sxs-lookup"><span data-stu-id="a6aba-141">Suppose you want to write the SQL itself rather than using LINQ.</span></span> <span data-ttu-id="a6aba-142">A tale scopo è necessario eseguire una query SQL che restituisca elementi diversi dagli oggetti entità.</span><span class="sxs-lookup"><span data-stu-id="a6aba-142">To do that you need to run a SQL query that returns something other than entity objects.</span></span> <span data-ttu-id="a6aba-143">In EF Core 1.0 è possibile scrivere codice ADO.NET e ottenere la connessione di database da EF.</span><span class="sxs-lookup"><span data-stu-id="a6aba-143">In EF Core 1.0, one way to do that is write ADO.NET code and get the database connection from EF.</span></span>

<span data-ttu-id="a6aba-144">In *HomeController.cs* sostituire il metodo `About` con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="a6aba-144">In *HomeController.cs*, replace the `About` method with the following code:</span></span>

[!code-csharp[](intro/samples/cu/Controllers/HomeController.cs?name=snippet_UseRawSQL&highlight=3-32)]

<span data-ttu-id="a6aba-145">Aggiungere un'istruzione using:</span><span class="sxs-lookup"><span data-stu-id="a6aba-145">Add a using statement:</span></span>

[!code-csharp[](intro/samples/cu/Controllers/HomeController.cs?name=snippet_Usings2)]

<span data-ttu-id="a6aba-146">Eseguire l'app e passare alla pagina About.</span><span class="sxs-lookup"><span data-stu-id="a6aba-146">Run the app and go to the About page.</span></span> <span data-ttu-id="a6aba-147">Vengono visualizzati gli stessi dati visualizzati in precedenza.</span><span class="sxs-lookup"><span data-stu-id="a6aba-147">It displays the same data it did before.</span></span>

![Pagina About (Informazioni)](advanced/_static/about.png)

## <a name="call-an-update-query"></a><span data-ttu-id="a6aba-149">Chiamare una query di aggiornamento</span><span class="sxs-lookup"><span data-stu-id="a6aba-149">Call an update query</span></span>

<span data-ttu-id="a6aba-150">Si supponga che gli amministratori di Contoso University vogliano eseguire modifiche globali nel database, ad esempio modificare il numero di crediti di ogni corso.</span><span class="sxs-lookup"><span data-stu-id="a6aba-150">Suppose Contoso University administrators want to perform global changes in the database, such as changing the number of credits for every course.</span></span> <span data-ttu-id="a6aba-151">Se il numero di corsi dell'università è elevato, potrebbe non essere utile recuperarli tutti come entità e modificarli singolarmente.</span><span class="sxs-lookup"><span data-stu-id="a6aba-151">If the university has a large number of courses, it would be inefficient to retrieve them all as entities and change them individually.</span></span> <span data-ttu-id="a6aba-152">In questa sezione viene implementata una pagina Web che consente all'utente di specificare un fattore in base al quale modificare il numero di crediti di tutti i corsi e viene apportata la modifica eseguendo un'istruzione SQL UPDATE.</span><span class="sxs-lookup"><span data-stu-id="a6aba-152">In this section you'll implement a web page that enables the user to specify a factor by which to change the number of credits for all courses, and you'll make the change by executing a SQL UPDATE statement.</span></span> <span data-ttu-id="a6aba-153">La pagina Web apparirà come segue:</span><span class="sxs-lookup"><span data-stu-id="a6aba-153">The web page will look like the following illustration:</span></span>

![Pagina di aggiornamento dei crediti dei corsi](advanced/_static/update-credits.png)

<span data-ttu-id="a6aba-155">In *CoursesController.cs* aggiungere i metodi UpdateCourseCredits per HttpGet e HttpPost:</span><span class="sxs-lookup"><span data-stu-id="a6aba-155">In *CoursesController.cs*, add UpdateCourseCredits methods for HttpGet and HttpPost:</span></span>

[!code-csharp[](intro/samples/cu/Controllers/CoursesController.cs?name=snippet_UpdateGet)]

[!code-csharp[](intro/samples/cu/Controllers/CoursesController.cs?name=snippet_UpdatePost)]

<span data-ttu-id="a6aba-156">Quando il controller elabora una richiesta HttpGet, non viene restituito alcun elemento in `ViewData["RowsAffected"]` e la visualizzazione mostra una casella di testo vuota e un pulsante di invio, come illustrato nella figura precedente.</span><span class="sxs-lookup"><span data-stu-id="a6aba-156">When the controller processes an HttpGet request, nothing is returned in `ViewData["RowsAffected"]`, and the view displays an empty text box and a submit button, as shown in the preceding illustration.</span></span>

<span data-ttu-id="a6aba-157">Quando viene fatto clic sul pulsante **Aggiorna**, viene chiamato il metodo HttpPost e il moltiplicatore ha il valore immesso nella casella di testo.</span><span class="sxs-lookup"><span data-stu-id="a6aba-157">When the **Update** button is clicked, the HttpPost method is called, and multiplier has the value entered in the text box.</span></span> <span data-ttu-id="a6aba-158">Il codice esegue quindi l'SQL che aggiorna i corsi e restituisce il numero di righe interessate nella visualizzazione in `ViewData`.</span><span class="sxs-lookup"><span data-stu-id="a6aba-158">The code then executes the SQL that updates courses and returns the number of affected rows to the view in `ViewData`.</span></span> <span data-ttu-id="a6aba-159">Quando riceve un valore `RowsAffected`, la visualizzazione mostra il numero di righe aggiornate.</span><span class="sxs-lookup"><span data-stu-id="a6aba-159">When the view gets a `RowsAffected` value, it displays the number of rows updated.</span></span>

<span data-ttu-id="a6aba-160">In **Esplora soluzioni** fare clic con il pulsante destro del mouse sulla cartella *Views/Courses* e quindi fare clic su **Aggiungi > Nuovo elemento**.</span><span class="sxs-lookup"><span data-stu-id="a6aba-160">In **Solution Explorer**, right-click the *Views/Courses* folder, and then click **Add > New Item**.</span></span>

<span data-ttu-id="a6aba-161">Nella finestra di dialogo **Aggiungi nuovo elemento** fare clic su **ASP.NET Core** in **installato** nel riquadro sinistro, fare clic su **Razor Visualizza** e denominare la nuova vista *UpdateCourseCredits. cshtml*.</span><span class="sxs-lookup"><span data-stu-id="a6aba-161">In the **Add New Item** dialog, click **ASP.NET Core** under **Installed** in the left pane, click **Razor View**, and name the new view *UpdateCourseCredits.cshtml*.</span></span>

<span data-ttu-id="a6aba-162">In *Views/Courses/UpdateCourseCredits.cshtml* sostituire il codice del modello con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="a6aba-162">In *Views/Courses/UpdateCourseCredits.cshtml*, replace the template code with the following code:</span></span>

[!code-cshtml[](intro/samples/cu/Views/Courses/UpdateCourseCredits.cshtml)]

<span data-ttu-id="a6aba-163">Eseguire il metodo `UpdateCourseCredits` selezionando la scheda **Courses**, quindi aggiungendo "/UpdateCourseCredits" alla fine dell'URL nella barra degli indirizzi del browser (ad esempio: `http://localhost:5813/Courses/UpdateCourseCredits`).</span><span class="sxs-lookup"><span data-stu-id="a6aba-163">Run the `UpdateCourseCredits` method by selecting the **Courses** tab, then adding "/UpdateCourseCredits" to the end of the URL in the browser's address bar (for example: `http://localhost:5813/Courses/UpdateCourseCredits`).</span></span> <span data-ttu-id="a6aba-164">Immettere un numero nella casella di testo:</span><span class="sxs-lookup"><span data-stu-id="a6aba-164">Enter a number in the text box:</span></span>

![Pagina di aggiornamento dei crediti dei corsi](advanced/_static/update-credits.png)

<span data-ttu-id="a6aba-166">Fare clic su **Update**.</span><span class="sxs-lookup"><span data-stu-id="a6aba-166">Click **Update**.</span></span> <span data-ttu-id="a6aba-167">Viene visualizzato il numero di righe interessate:</span><span class="sxs-lookup"><span data-stu-id="a6aba-167">You see the number of rows affected:</span></span>

![Righe interessate della pagina Update Course Credits](advanced/_static/update-credits-rows-affected.png)

<span data-ttu-id="a6aba-169">Fare clic su **Torna all'elenco** per visualizzare l'elenco dei corsi con il numero di crediti modificato.</span><span class="sxs-lookup"><span data-stu-id="a6aba-169">Click **Back to List** to see the list of courses with the revised number of credits.</span></span>

<span data-ttu-id="a6aba-170">Tenere presente che il codice di produzione assicurerà che gli aggiornamenti restituiscano sempre dati validi.</span><span class="sxs-lookup"><span data-stu-id="a6aba-170">Note that production code would ensure that updates always result in valid data.</span></span> <span data-ttu-id="a6aba-171">Il codice semplificato illustrato potrebbe moltiplicare il numero di crediti e restituire numeri maggiori di 5.</span><span class="sxs-lookup"><span data-stu-id="a6aba-171">The simplified code shown here could multiply the number of credits enough to result in numbers greater than 5.</span></span> <span data-ttu-id="a6aba-172">(La `Credits` proprietà ha un `[Range(0, 5)]` attributo). La query di aggiornamento funzionerebbe ma i dati non validi potrebbero provocare risultati imprevisti in altre parti del sistema che presuppongono che il numero di crediti sia pari o inferiore a 5.</span><span class="sxs-lookup"><span data-stu-id="a6aba-172">(The `Credits` property has a `[Range(0, 5)]` attribute.) The update query would work but the invalid data could cause unexpected results in other parts of the system that assume the number of credits is 5 or less.</span></span>

<span data-ttu-id="a6aba-173">Per altre informazioni sulle query SQL non elaborate, vedere [Query SQL non elaborate](/ef/core/querying/raw-sql).</span><span class="sxs-lookup"><span data-stu-id="a6aba-173">For more information about raw SQL queries, see [Raw SQL Queries](/ef/core/querying/raw-sql).</span></span>

## <a name="examine-sql-queries"></a><span data-ttu-id="a6aba-174">Esaminare le query SQL</span><span class="sxs-lookup"><span data-stu-id="a6aba-174">Examine SQL queries</span></span>

<span data-ttu-id="a6aba-175">A volte può essere utile visualizzare le query SQL inviate al database.</span><span class="sxs-lookup"><span data-stu-id="a6aba-175">Sometimes it's helpful to be able to see the actual SQL queries that are sent to the database.</span></span> <span data-ttu-id="a6aba-176">La funzionalità di registrazione incorporata di ASP.NET Core viene usata automaticamente da EF Core per creare i log contenenti il codice SQL di query e aggiornamenti.</span><span class="sxs-lookup"><span data-stu-id="a6aba-176">Built-in logging functionality for ASP.NET Core is automatically used by EF Core to write logs that contain the SQL for queries and updates.</span></span> <span data-ttu-id="a6aba-177">In questa sezione vengono presentati alcuni esempi di registrazione SQL.</span><span class="sxs-lookup"><span data-stu-id="a6aba-177">In this section you'll see some examples of SQL logging.</span></span>

<span data-ttu-id="a6aba-178">Aprire *StudentsController.cs* e nel metodo `Details` impostare un punto di interruzione nell'istruzione `if (student == null)`.</span><span class="sxs-lookup"><span data-stu-id="a6aba-178">Open *StudentsController.cs* and in the `Details` method set a breakpoint on the `if (student == null)` statement.</span></span>

<span data-ttu-id="a6aba-179">Eseguire l'app in modalità di debug e passare alla pagina Details di uno studente.</span><span class="sxs-lookup"><span data-stu-id="a6aba-179">Run the app in debug mode, and go to the Details page for a student.</span></span>

<span data-ttu-id="a6aba-180">Passare alla finestra **Output** con l'output del debug per visualizzare la query:</span><span class="sxs-lookup"><span data-stu-id="a6aba-180">Go to the **Output** window showing debug output, and you see the query:</span></span>

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

<span data-ttu-id="a6aba-181">Si noti che il codice SQL seleziona un massimo di 2 righe (`TOP(2)`) dalla tabella Person.</span><span class="sxs-lookup"><span data-stu-id="a6aba-181">You'll notice something here that might surprise you: the SQL selects up to 2 rows (`TOP(2)`) from the Person table.</span></span> <span data-ttu-id="a6aba-182">Il metodo `SingleOrDefaultAsync` non viene risolto in 1 riga nel server.</span><span class="sxs-lookup"><span data-stu-id="a6aba-182">The `SingleOrDefaultAsync` method doesn't resolve to 1 row on the server.</span></span> <span data-ttu-id="a6aba-183">Ecco perché:</span><span class="sxs-lookup"><span data-stu-id="a6aba-183">Here's why:</span></span>

* <span data-ttu-id="a6aba-184">Se la query restituisse più righe, il metodo restituirebbe un valore Null.</span><span class="sxs-lookup"><span data-stu-id="a6aba-184">If the query would return multiple rows, the method returns null.</span></span>
* <span data-ttu-id="a6aba-185">Per determinare se la query restituirà più righe, EF deve controllare se restituisce almeno 2 righe.</span><span class="sxs-lookup"><span data-stu-id="a6aba-185">To determine whether the query would return multiple rows, EF has to check if it returns at least 2.</span></span>

<span data-ttu-id="a6aba-186">Si noti che non è necessario usare la modalità di debug e arrestarsi in un punto di interruzione per visualizzare l'output di registrazione nella finestra **Output**.</span><span class="sxs-lookup"><span data-stu-id="a6aba-186">Note that you don't have to use debug mode and stop at a breakpoint to get logging output in the **Output** window.</span></span> <span data-ttu-id="a6aba-187">Si tratta soltanto di un metodo pratico per arrestare la registrazione nel punto in cui si desidera visualizzare l'output.</span><span class="sxs-lookup"><span data-stu-id="a6aba-187">It's just a convenient way to stop the logging at the point you want to look at the output.</span></span> <span data-ttu-id="a6aba-188">Se non si esegue questa operazione, la registrazione continua ed è necessario scorrere indietro per individuare le parti desiderate.</span><span class="sxs-lookup"><span data-stu-id="a6aba-188">If you don't do that, logging continues and you have to scroll back to find the parts you're interested in.</span></span>

## <a name="create-an-abstraction-layer"></a><span data-ttu-id="a6aba-189">Creare un livello di astrazione</span><span class="sxs-lookup"><span data-stu-id="a6aba-189">Create an abstraction layer</span></span>

<span data-ttu-id="a6aba-190">Molti sviluppatori scrivono codice per implementare i modelli di repository e unità di lavoro come wrapper per il codice usato con Entity Framework.</span><span class="sxs-lookup"><span data-stu-id="a6aba-190">Many developers write code to implement the repository and unit of work patterns as a wrapper around code that works with the Entity Framework.</span></span> <span data-ttu-id="a6aba-191">Questi modelli sono progettati per la creazione di un livello di astrazione tra il livello di accesso ai dati e il livello della logica di business di un'applicazione.</span><span class="sxs-lookup"><span data-stu-id="a6aba-191">These patterns are intended to create an abstraction layer between the data access layer and the business logic layer of an application.</span></span> <span data-ttu-id="a6aba-192">L'implementazione di questi modelli può essere utile per isolare l'applicazione dalle modifiche nell'archivio dati e può semplificare il testing unità automatizzato o lo sviluppo basato su test (TDD).</span><span class="sxs-lookup"><span data-stu-id="a6aba-192">Implementing these patterns can help insulate your application from changes in the data store and can facilitate automated unit testing or test-driven development (TDD).</span></span> <span data-ttu-id="a6aba-193">Tuttavia, la scrittura di codice aggiuntivo per l'implementazione di questi modelli non è sempre la scelta migliore per le applicazioni che usano EF, per diverse ragioni:</span><span class="sxs-lookup"><span data-stu-id="a6aba-193">However, writing additional code to implement these patterns isn't always the best choice for applications that use EF, for several reasons:</span></span>

* <span data-ttu-id="a6aba-194">La classe del contesto EF isola il codice dal codice specifico dell'archivio dati.</span><span class="sxs-lookup"><span data-stu-id="a6aba-194">The EF context class itself insulates your code from data-store-specific code.</span></span>

* <span data-ttu-id="a6aba-195">La classe del contesto di EF può svolgere la funzione di classe di unità di lavoro per gli aggiornamenti di database eseguiti usando EF.</span><span class="sxs-lookup"><span data-stu-id="a6aba-195">The EF context class can act as a unit-of-work class for database updates that you do using EF.</span></span>

* <span data-ttu-id="a6aba-196">EF include funzionalità che consentono di implementare lo sviluppo basato su test senza scrivere codice di repository.</span><span class="sxs-lookup"><span data-stu-id="a6aba-196">EF includes features for implementing TDD without writing repository code.</span></span>

<span data-ttu-id="a6aba-197">Per informazioni su come implementare i modelli di repository e unità di lavoro, vedere [la versione Entity Framework 5 di questa serie di esercitazioni](/aspnet/mvc/overview/older-versions/getting-started-with-ef-5-using-mvc-4/implementing-the-repository-and-unit-of-work-patterns-in-an-asp-net-mvc-application).</span><span class="sxs-lookup"><span data-stu-id="a6aba-197">For information about how to implement the repository and unit of work patterns, see [the Entity Framework 5 version of this tutorial series](/aspnet/mvc/overview/older-versions/getting-started-with-ef-5-using-mvc-4/implementing-the-repository-and-unit-of-work-patterns-in-an-asp-net-mvc-application).</span></span>

<span data-ttu-id="a6aba-198">Entity Framework Core implementa un provider di database in memoria che è possibile usare per il test.</span><span class="sxs-lookup"><span data-stu-id="a6aba-198">Entity Framework Core implements an in-memory database provider that can be used for testing.</span></span> <span data-ttu-id="a6aba-199">Per altre informazioni, vedere [Test con InMemory](/ef/core/miscellaneous/testing/in-memory).</span><span class="sxs-lookup"><span data-stu-id="a6aba-199">For more information, see [Test with InMemory](/ef/core/miscellaneous/testing/in-memory).</span></span>

## <a name="automatic-change-detection"></a><span data-ttu-id="a6aba-200">Rilevamento automatico delle modifiche</span><span class="sxs-lookup"><span data-stu-id="a6aba-200">Automatic change detection</span></span>

<span data-ttu-id="a6aba-201">Entity Framework determina come è stata modificata un'entità (e di conseguenza gli aggiornamenti da inviare al database) confrontando i valori correnti di un'entità con i valori originali.</span><span class="sxs-lookup"><span data-stu-id="a6aba-201">The Entity Framework determines how an entity has changed (and therefore which updates need to be sent to the database) by comparing the current values of an entity with the original values.</span></span> <span data-ttu-id="a6aba-202">I valori originali vengono memorizzati quando viene eseguita una query sull'entità o quando viene collegata.</span><span class="sxs-lookup"><span data-stu-id="a6aba-202">The original values are stored when the entity is queried or attached.</span></span> <span data-ttu-id="a6aba-203">I metodi che causano il rilevamento automatico delle modifiche includono:</span><span class="sxs-lookup"><span data-stu-id="a6aba-203">Some of the methods that cause automatic change detection are the following:</span></span>

* <span data-ttu-id="a6aba-204">DbContext.SaveChanges</span><span class="sxs-lookup"><span data-stu-id="a6aba-204">DbContext.SaveChanges</span></span>

* <span data-ttu-id="a6aba-205">DbContext.Entry</span><span class="sxs-lookup"><span data-stu-id="a6aba-205">DbContext.Entry</span></span>

* <span data-ttu-id="a6aba-206">ChangeTracker.Entries</span><span class="sxs-lookup"><span data-stu-id="a6aba-206">ChangeTracker.Entries</span></span>

<span data-ttu-id="a6aba-207">Se viene registrato un numero elevato di entità e uno solo di questi metodi viene chiamato più volte in un ciclo, è possibile ottenere miglioramenti significativi delle prestazioni disattivando temporaneamente il rilevamento automatico delle modifiche usando la proprietà `ChangeTracker.AutoDetectChangesEnabled`.</span><span class="sxs-lookup"><span data-stu-id="a6aba-207">If you're tracking a large number of entities and you call one of these methods many times in a loop, you might get significant performance improvements by temporarily turning off automatic change detection using the `ChangeTracker.AutoDetectChangesEnabled` property.</span></span> <span data-ttu-id="a6aba-208">Ad esempio:</span><span class="sxs-lookup"><span data-stu-id="a6aba-208">For example:</span></span>

```csharp
_context.ChangeTracker.AutoDetectChangesEnabled = false;
```

## <a name="ef-core-source-code-and-development-plans"></a><span data-ttu-id="a6aba-209">Codice sorgente e piani di sviluppo di EF Core</span><span class="sxs-lookup"><span data-stu-id="a6aba-209">EF Core source code and development plans</span></span>

<span data-ttu-id="a6aba-210">Il Entity Framework Core origine si trova in [https://github.com/dotnet/efcore](https://github.com/dotnet/efcore) .</span><span class="sxs-lookup"><span data-stu-id="a6aba-210">The Entity Framework Core source is at [https://github.com/dotnet/efcore](https://github.com/dotnet/efcore).</span></span> <span data-ttu-id="a6aba-211">Il repository di EF Core contiene le compilazioni notturne, la gestione dei problemi, le specifiche delle funzionalità, le note delle riunioni di progettazione e [la roadmap per lo sviluppo futuro](https://github.com/dotnet/efcore/wiki/Roadmap).</span><span class="sxs-lookup"><span data-stu-id="a6aba-211">The EF Core repository contains nightly builds, issue tracking, feature specs, design meeting notes, and [the roadmap for future development](https://github.com/dotnet/efcore/wiki/Roadmap).</span></span> <span data-ttu-id="a6aba-212">È possibile archiviare o individuare i bug e contribuire.</span><span class="sxs-lookup"><span data-stu-id="a6aba-212">You can file or find bugs, and contribute.</span></span>

<span data-ttu-id="a6aba-213">Sebbene il codice sorgente sia disponibile, Entity Framework Core è completamente supportato come prodotto Microsoft.</span><span class="sxs-lookup"><span data-stu-id="a6aba-213">Although the source code is open, Entity Framework Core is fully supported as a Microsoft product.</span></span> <span data-ttu-id="a6aba-214">Il team di Microsoft Entity Framework controlla i contributi accettati ed esegue il test di tutte le modifiche al codice per garantire la qualità di ogni rilascio.</span><span class="sxs-lookup"><span data-stu-id="a6aba-214">The Microsoft Entity Framework team keeps control over which contributions are accepted and tests all code changes to ensure the quality of each release.</span></span>

## <a name="reverse-engineer-from-existing-database"></a><span data-ttu-id="a6aba-215">Eseguire il reverse engineering dal database esistente</span><span class="sxs-lookup"><span data-stu-id="a6aba-215">Reverse engineer from existing database</span></span>

<span data-ttu-id="a6aba-216">Per eseguire il reverse engineering di un modello di dati contenente classi di entità da un database esistente, usare il comando [scaffold-dbcontext](/ef/core/miscellaneous/cli/powershell#scaffold-dbcontext).</span><span class="sxs-lookup"><span data-stu-id="a6aba-216">To reverse engineer a data model including entity classes from an existing database, use the [scaffold-dbcontext](/ef/core/miscellaneous/cli/powershell#scaffold-dbcontext) command.</span></span> <span data-ttu-id="a6aba-217">Vedere l'[esercitazione introduttiva](/ef/core/get-started/aspnetcore/existing-db).</span><span class="sxs-lookup"><span data-stu-id="a6aba-217">See the [getting-started tutorial](/ef/core/get-started/aspnetcore/existing-db).</span></span>

<a id="dynamic-linq"></a>

## <a name="use-dynamic-linq-to-simplify-code"></a><span data-ttu-id="a6aba-218">Usare LINQ dinamico per semplificare il codice</span><span class="sxs-lookup"><span data-stu-id="a6aba-218">Use dynamic LINQ to simplify code</span></span>

<span data-ttu-id="a6aba-219">La [terza esercitazione di questa serie](sort-filter-page.md) descrive come scrivere codice LINQ impostando come hardcoded i nomi di colonna in un'istruzione `switch`.</span><span class="sxs-lookup"><span data-stu-id="a6aba-219">The [third tutorial in this series](sort-filter-page.md) shows how to write LINQ code by hard-coding column names in a `switch` statement.</span></span> <span data-ttu-id="a6aba-220">Se la selezione viene effettuata tra due colonne, il funzionamento è corretto, ma in presenza di numerose colonne il codice potrebbe diventare troppo lungo.</span><span class="sxs-lookup"><span data-stu-id="a6aba-220">With two columns to choose from, this works fine, but if you have many columns the code could get verbose.</span></span> <span data-ttu-id="a6aba-221">Per risolvere il problema, è possibile usare il metodo `EF.Property` per specificare il nome della proprietà come stringa.</span><span class="sxs-lookup"><span data-stu-id="a6aba-221">To solve that problem, you can use the `EF.Property` method to specify the name of the property as a string.</span></span> <span data-ttu-id="a6aba-222">Per provare questo approccio, sostituire il metodo `Index` in `StudentsController` con il codice seguente.</span><span class="sxs-lookup"><span data-stu-id="a6aba-222">To try out this approach, replace the `Index` method in the `StudentsController` with the following code.</span></span>

[!code-csharp[](intro/samples/cu/Controllers/StudentsController.cs?name=snippet_DynamicLinq)]

## <a name="acknowledgments"></a><span data-ttu-id="a6aba-223">Ringraziamenti</span><span class="sxs-lookup"><span data-stu-id="a6aba-223">Acknowledgments</span></span>

<span data-ttu-id="a6aba-224">Tom Dykstra e Rick Anderson (twitter @RickAndMSFT) hanno scritto questa esercitazione.</span><span class="sxs-lookup"><span data-stu-id="a6aba-224">Tom Dykstra and Rick Anderson (twitter @RickAndMSFT) wrote this tutorial.</span></span> <span data-ttu-id="a6aba-225">Rowan Miller, Diego Vega e altri membri del team Entity Framework hanno offerto supporto per le revisioni del codice e per il debug dei problemi sorti durante la scrittura del codice per le esercitazioni.</span><span class="sxs-lookup"><span data-stu-id="a6aba-225">Rowan Miller, Diego Vega, and other members of the Entity Framework team assisted with code reviews and helped debug issues that arose while we were writing code for the tutorials.</span></span> <span data-ttu-id="a6aba-226">John Parente e Paul Goldman hanno collaborato all'aggiornamento dell'esercitazione per ASP.NET Core 2.2.</span><span class="sxs-lookup"><span data-stu-id="a6aba-226">John Parente and Paul Goldman worked on updating the tutorial for ASP.NET Core 2.2.</span></span>

<a id="common-errors"></a>

## <a name="troubleshoot-common-errors"></a><span data-ttu-id="a6aba-227">Risolvere gli errori comuni</span><span class="sxs-lookup"><span data-stu-id="a6aba-227">Troubleshoot common errors</span></span>

### <a name="contosouniversitydll-used-by-another-process"></a><span data-ttu-id="a6aba-228">ContosoUniversity.dll usata da un altro processo</span><span class="sxs-lookup"><span data-stu-id="a6aba-228">ContosoUniversity.dll used by another process</span></span>

<span data-ttu-id="a6aba-229">Messaggio di errore:</span><span class="sxs-lookup"><span data-stu-id="a6aba-229">Error message:</span></span>

> <span data-ttu-id="a6aba-230">Impossibile aprire '...bin\Debug\netcoreapp1.0\ContosoUniversity.dll' per la scrittura -- 'Il processo non può accedere al file '...\bin\Debug\netcoreapp1.0\ContosoUniversity.dll' perché è in uso da un altro processo.</span><span class="sxs-lookup"><span data-stu-id="a6aba-230">Cannot open '...bin\Debug\netcoreapp1.0\ContosoUniversity.dll' for writing -- 'The process cannot access the file '...\bin\Debug\netcoreapp1.0\ContosoUniversity.dll' because it is being used by another process.</span></span>

<span data-ttu-id="a6aba-231">Soluzione:</span><span class="sxs-lookup"><span data-stu-id="a6aba-231">Solution:</span></span>

<span data-ttu-id="a6aba-232">Arrestare il sito in IIS Express.</span><span class="sxs-lookup"><span data-stu-id="a6aba-232">Stop the site in IIS Express.</span></span> <span data-ttu-id="a6aba-233">Passare alla barra delle applicazioni di Windows, trovare IIS Express e fare clic con il pulsante destro del mouse sull'icona, selezionare il sito Contoso University e quindi fare clic su **Arresta sito**.</span><span class="sxs-lookup"><span data-stu-id="a6aba-233">Go to the Windows System Tray, find IIS Express and right-click its icon, select the Contoso University site, and then click **Stop Site**.</span></span>

### <a name="migration-scaffolded-with-no-code-in-up-and-down-methods"></a><span data-ttu-id="a6aba-234">Migrazione con scaffolding senza codice nei metodi Up e Down</span><span class="sxs-lookup"><span data-stu-id="a6aba-234">Migration scaffolded with no code in Up and Down methods</span></span>

<span data-ttu-id="a6aba-235">Possibile causa:</span><span class="sxs-lookup"><span data-stu-id="a6aba-235">Possible cause:</span></span>

<span data-ttu-id="a6aba-236">I comandi CLI di EF non chiudono e salvano automaticamente i file di codice.</span><span class="sxs-lookup"><span data-stu-id="a6aba-236">The EF CLI commands don't automatically close and save code files.</span></span> <span data-ttu-id="a6aba-237">Se sono presenti modifiche non salvate quando si esegue il comando `migrations add`, EF non trova le modifiche.</span><span class="sxs-lookup"><span data-stu-id="a6aba-237">If you have unsaved changes when you run the `migrations add` command, EF won't find your changes.</span></span>

<span data-ttu-id="a6aba-238">Soluzione:</span><span class="sxs-lookup"><span data-stu-id="a6aba-238">Solution:</span></span>

<span data-ttu-id="a6aba-239">Eseguire il comando `migrations remove`, salvare le modifiche al codice e rieseguire il comando `migrations add`.</span><span class="sxs-lookup"><span data-stu-id="a6aba-239">Run the `migrations remove` command, save your code changes and rerun the `migrations add` command.</span></span>

### <a name="errors-while-running-database-update"></a><span data-ttu-id="a6aba-240">Errori durante l'esecuzione dell'aggiornamento del database</span><span class="sxs-lookup"><span data-stu-id="a6aba-240">Errors while running database update</span></span>

<span data-ttu-id="a6aba-241">Quando si apportano modifiche allo schema in un database con dati esistenti è possibile che si riscontrino altri errori.</span><span class="sxs-lookup"><span data-stu-id="a6aba-241">It's possible to get other errors when making schema changes in a database that has existing data.</span></span> <span data-ttu-id="a6aba-242">Se si verificano errori di migrazione che non si riesce a risolvere, è possibile modificare il nome del database nella stringa di connessione o eliminare il database.</span><span class="sxs-lookup"><span data-stu-id="a6aba-242">If you get migration errors you can't resolve, you can either change the database name in the connection string or delete the database.</span></span> <span data-ttu-id="a6aba-243">Un nuovo database non contiene dati di cui eseguire la migrazione e ci sono maggiori probabilità che il comando update-database venga completato senza errori.</span><span class="sxs-lookup"><span data-stu-id="a6aba-243">With a new database, there's no data to migrate, and the update-database command is much more likely to complete without errors.</span></span>

<span data-ttu-id="a6aba-244">L'approccio più semplice consiste nel rinominare il database in *appsettings.json* .</span><span class="sxs-lookup"><span data-stu-id="a6aba-244">The simplest approach is to rename the database in *appsettings.json*.</span></span> <span data-ttu-id="a6aba-245">Alla successiva esecuzione di `database update`, verrà creato un nuovo database.</span><span class="sxs-lookup"><span data-stu-id="a6aba-245">The next time you run `database update`, a new database will be created.</span></span>

<span data-ttu-id="a6aba-246">Per eliminare un database in SSOX, fare clic con il pulsante destro del mouse sul database, fare clic su **Elimina** e quindi nella finestra di dialogo **Elimina database** selezionare **Chiudi connessioni esistenti** e fare clic su **OK**.</span><span class="sxs-lookup"><span data-stu-id="a6aba-246">To delete a database in SSOX, right-click the database, click **Delete**, and then in the **Delete Database** dialog box select **Close existing connections** and click **OK**.</span></span>

<span data-ttu-id="a6aba-247">Per eliminare un database tramite l'interfaccia CLI, eseguire il comando CLI `database drop`:</span><span class="sxs-lookup"><span data-stu-id="a6aba-247">To delete a database by using the CLI, run the `database drop` CLI command:</span></span>

```dotnetcli
dotnet ef database drop
```

### <a name="error-locating-sql-server-instance"></a><span data-ttu-id="a6aba-248">Errore di individuazione dell'istanza di SQL Server</span><span class="sxs-lookup"><span data-stu-id="a6aba-248">Error locating SQL Server instance</span></span>

<span data-ttu-id="a6aba-249">Messaggio di errore:</span><span class="sxs-lookup"><span data-stu-id="a6aba-249">Error Message:</span></span>

> <span data-ttu-id="a6aba-250">Si è verificato un errore di rete o specifico dell'istanza mentre veniva stabilita la connessione a SQL Server.</span><span class="sxs-lookup"><span data-stu-id="a6aba-250">A network-related or instance-specific error occurred while establishing a connection to SQL Server.</span></span> <span data-ttu-id="a6aba-251">Il server non è stato trovato o non è accessibile.</span><span class="sxs-lookup"><span data-stu-id="a6aba-251">The server was not found or was not accessible.</span></span> <span data-ttu-id="a6aba-252">Verificare che il nome dell'istanza sia corretto e che il server sia configurato in modo da consentire connessioni remote.</span><span class="sxs-lookup"><span data-stu-id="a6aba-252">Verify that the instance name is correct and that SQL Server is configured to allow remote connections.</span></span> <span data-ttu-id="a6aba-253">(provider: interfacce di rete SQL, errore: 26 - Errore nell'individuazione del server/dell'istanza specificata)</span><span class="sxs-lookup"><span data-stu-id="a6aba-253">(provider: SQL Network Interfaces, error: 26 - Error Locating Server/Instance Specified)</span></span>

<span data-ttu-id="a6aba-254">Soluzione:</span><span class="sxs-lookup"><span data-stu-id="a6aba-254">Solution:</span></span>

<span data-ttu-id="a6aba-255">Controllare la stringa di connessione.</span><span class="sxs-lookup"><span data-stu-id="a6aba-255">Check the connection string.</span></span> <span data-ttu-id="a6aba-256">Se il file di database è stato eliminato manualmente, modificare il nome del database nella stringa di costruzione per riniziare con un nuovo database.</span><span class="sxs-lookup"><span data-stu-id="a6aba-256">If you have manually deleted the database file, change the name of the database in the construction string to start over with a new database.</span></span>

## <a name="get-the-code"></a><span data-ttu-id="a6aba-257">Ottenere il codice</span><span class="sxs-lookup"><span data-stu-id="a6aba-257">Get the code</span></span>

[<span data-ttu-id="a6aba-258">Scaricare o visualizzare l'applicazione completata.</span><span class="sxs-lookup"><span data-stu-id="a6aba-258">Download or view the completed application.</span></span>](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/data/ef-mvc/intro/samples/cu-final)

## <a name="additional-resources"></a><span data-ttu-id="a6aba-259">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="a6aba-259">Additional resources</span></span>

<span data-ttu-id="a6aba-260">Per altre informazioni su EF Core, vedere la [documentazione di Entity Framework Core](/ef/core).</span><span class="sxs-lookup"><span data-stu-id="a6aba-260">For more information about EF Core, see the [Entity Framework Core documentation](/ef/core).</span></span> <span data-ttu-id="a6aba-261">È disponibile anche il libro [Entity Framework Core in Action](https://www.manning.com/books/entity-framework-core-in-action).</span><span class="sxs-lookup"><span data-stu-id="a6aba-261">A book is also available: [Entity Framework Core in Action](https://www.manning.com/books/entity-framework-core-in-action).</span></span>

<span data-ttu-id="a6aba-262">Per informazioni su come distribuire un'app Web, vedere <xref:host-and-deploy/index>.</span><span class="sxs-lookup"><span data-stu-id="a6aba-262">For information on how to deploy a web app, see <xref:host-and-deploy/index>.</span></span>

<span data-ttu-id="a6aba-263">Per informazioni su altri argomenti correlati ad ASP.NET Core MVC, ad esempio sull'autenticazione e l'autorizzazione, vedere <xref:index>.</span><span class="sxs-lookup"><span data-stu-id="a6aba-263">For information about other topics related to ASP.NET Core MVC, such as authentication and authorization, see <xref:index>.</span></span>

## <a name="next-steps"></a><span data-ttu-id="a6aba-264">Passaggi successivi</span><span class="sxs-lookup"><span data-stu-id="a6aba-264">Next steps</span></span>

<span data-ttu-id="a6aba-265">In questa esercitazione:</span><span class="sxs-lookup"><span data-stu-id="a6aba-265">In this tutorial, you:</span></span>

> [!div class="checklist"]
> * <span data-ttu-id="a6aba-266">Eseguire query SQL non elaborate</span><span class="sxs-lookup"><span data-stu-id="a6aba-266">Performed raw SQL queries</span></span>
> * <span data-ttu-id="a6aba-267">Chiamare una query per restituire le entità</span><span class="sxs-lookup"><span data-stu-id="a6aba-267">Called a query to return entities</span></span>
> * <span data-ttu-id="a6aba-268">Chiamare una query per restituire altri tipi</span><span class="sxs-lookup"><span data-stu-id="a6aba-268">Called a query to return other types</span></span>
> * <span data-ttu-id="a6aba-269">Chiamare una query di aggiornamento</span><span class="sxs-lookup"><span data-stu-id="a6aba-269">Called an update query</span></span>
> * <span data-ttu-id="a6aba-270">Esaminare le query SQL</span><span class="sxs-lookup"><span data-stu-id="a6aba-270">Examined SQL queries</span></span>
> * <span data-ttu-id="a6aba-271">Creare un livello di astrazione</span><span class="sxs-lookup"><span data-stu-id="a6aba-271">Created an abstraction layer</span></span>
> * <span data-ttu-id="a6aba-272">Scoprire di più sul rilevamento automatico delle modifiche</span><span class="sxs-lookup"><span data-stu-id="a6aba-272">Learned about Automatic change detection</span></span>
> * <span data-ttu-id="a6aba-273">Scoprire di più sul codice sorgente e i piani di sviluppo di EF Core</span><span class="sxs-lookup"><span data-stu-id="a6aba-273">Learned about EF Core source code and development plans</span></span>
> * <span data-ttu-id="a6aba-274">Imparare a usare LINQ dinamico per semplificare il codice</span><span class="sxs-lookup"><span data-stu-id="a6aba-274">Learned how to use dynamic LINQ to simplify code</span></span>

<span data-ttu-id="a6aba-275">Questa esercitazione completa la serie di esercitazioni sull'uso di Entity Framework Core in un'applicazione ASP.NET Core MVC.</span><span class="sxs-lookup"><span data-stu-id="a6aba-275">This completes this series of tutorials on using the Entity Framework Core in an ASP.NET Core MVC application.</span></span> <span data-ttu-id="a6aba-276">In questa serie di esercitazioni è stato usato un nuovo database; in alternativa, è possibile decompilare un modello di un database esistente.</span><span class="sxs-lookup"><span data-stu-id="a6aba-276">This series worked with a new database; an alternative is to  reverse engineer a model from an existing database.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="a6aba-277">Esercitazione: EF Core con MVC, database esistente</span><span class="sxs-lookup"><span data-stu-id="a6aba-277">Tutorial: EF Core with MVC, existing database</span></span>](/ef/core/get-started/aspnetcore/existing-db?toc=/aspnet/core/toc.json&bc=/aspnet/core/breadcrumb/toc.json)
