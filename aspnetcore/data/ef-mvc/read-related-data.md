---
title: 'Esercitazione: leggere dati correlati-MVC ASP.NET con EF Core'
description: In questa esercitazione verranno letti e visualizzati dati correlati, ovvero dati che Entity Framework carica all'interno delle proprietà di navigazione.
author: rick-anderson
ms.author: riande
ms.date: 09/28/2019
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
uid: data/ef-mvc/read-related-data
ms.openlocfilehash: 5f0e8efdf57e0a8ba5483680224d33ea5d60f06a
ms.sourcegitcommit: e311cfb77f26a0a23681019bd334929d1aaeda20
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/03/2021
ms.locfileid: "99530151"
---
# <a name="tutorial-read-related-data---aspnet-mvc-with-ef-core"></a><span data-ttu-id="2c59d-103">Esercitazione: leggere dati correlati-MVC ASP.NET con EF Core</span><span class="sxs-lookup"><span data-stu-id="2c59d-103">Tutorial: Read related data - ASP.NET MVC with EF Core</span></span>

<span data-ttu-id="2c59d-104">Nell'esercitazione precedente è stato completato il modello di dati School.</span><span class="sxs-lookup"><span data-stu-id="2c59d-104">In the previous tutorial, you completed the School data model.</span></span> <span data-ttu-id="2c59d-105">In questa esercitazione verranno letti e visualizzati dati correlati, ovvero dati che Entity Framework carica all'interno delle proprietà di navigazione.</span><span class="sxs-lookup"><span data-stu-id="2c59d-105">In this tutorial, you'll read and display related data -- that is, data that the Entity Framework loads into navigation properties.</span></span>

<span data-ttu-id="2c59d-106">Le figure seguenti illustrano le pagine che verranno usate.</span><span class="sxs-lookup"><span data-stu-id="2c59d-106">The following illustrations show the pages that you'll work with.</span></span>

![Pagina di indice dei corsi](read-related-data/_static/courses-index.png)

![Pagina di indice degli insegnanti](read-related-data/_static/instructors-index.png)

<span data-ttu-id="2c59d-109">In questa esercitazione:</span><span class="sxs-lookup"><span data-stu-id="2c59d-109">In this tutorial, you:</span></span>

> [!div class="checklist"]
> * <span data-ttu-id="2c59d-110">Scoprire come caricare i dati correlati</span><span class="sxs-lookup"><span data-stu-id="2c59d-110">Learn how to load related data</span></span>
> * <span data-ttu-id="2c59d-111">Creare una pagina Courses</span><span class="sxs-lookup"><span data-stu-id="2c59d-111">Create a Courses page</span></span>
> * <span data-ttu-id="2c59d-112">Creare una pagina Instructors</span><span class="sxs-lookup"><span data-stu-id="2c59d-112">Create an Instructors page</span></span>
> * <span data-ttu-id="2c59d-113">Ottenere informazioni sul caricamento esplicito</span><span class="sxs-lookup"><span data-stu-id="2c59d-113">Learn about explicit loading</span></span>

## <a name="prerequisites"></a><span data-ttu-id="2c59d-114">Prerequisiti</span><span class="sxs-lookup"><span data-stu-id="2c59d-114">Prerequisites</span></span>

* [<span data-ttu-id="2c59d-115">Creare un modello di dati complesso</span><span class="sxs-lookup"><span data-stu-id="2c59d-115">Create a complex data model</span></span>](complex-data-model.md)

## <a name="learn-how-to-load-related-data"></a><span data-ttu-id="2c59d-116">Scoprire come caricare i dati correlati</span><span class="sxs-lookup"><span data-stu-id="2c59d-116">Learn how to load related data</span></span>

<span data-ttu-id="2c59d-117">Il software ORM (Object-Relational Mapping), ad esempio Entity Framework, può caricare dati correlati nelle proprietà di navigazione di un'entità in diversi modi:</span><span class="sxs-lookup"><span data-stu-id="2c59d-117">There are several ways that Object-Relational Mapping (ORM) software such as Entity Framework can load related data into the navigation properties of an entity:</span></span>

* <span data-ttu-id="2c59d-118">Caricamento eager.</span><span class="sxs-lookup"><span data-stu-id="2c59d-118">Eager loading.</span></span> <span data-ttu-id="2c59d-119">Quando l'entità viene letta, i dati correlati corrispondenti vengono recuperati insieme ad essa.</span><span class="sxs-lookup"><span data-stu-id="2c59d-119">When the entity is read, related data is retrieved along with it.</span></span> <span data-ttu-id="2c59d-120">Ciò in genere ha come risultato una query join singola che recupera tutti i dati necessari.</span><span class="sxs-lookup"><span data-stu-id="2c59d-120">This typically results in a single join query that retrieves all of the data that's needed.</span></span> <span data-ttu-id="2c59d-121">Per specificare il caricamento eager in Entity Framework Core si usano i metodi `Include` e `ThenInclude`.</span><span class="sxs-lookup"><span data-stu-id="2c59d-121">You specify eager loading in Entity Framework Core by using the `Include` and `ThenInclude` methods.</span></span>

  ![Esempio di caricamento eager](read-related-data/_static/eager-loading.png)

  <span data-ttu-id="2c59d-123">È possibile recuperare alcuni dati tramite query separate. In questo caso EF "corregge" le proprietà di navigazione.</span><span class="sxs-lookup"><span data-stu-id="2c59d-123">You can retrieve some of the data in separate queries, and EF "fixes up" the navigation properties.</span></span>  <span data-ttu-id="2c59d-124">In altre parole, EF aggiunge automaticamente le entità recuperate separatamente nelle proprietà di navigazione corrispondenti delle entità recuperate in precedenza.</span><span class="sxs-lookup"><span data-stu-id="2c59d-124">That is, EF automatically adds the separately retrieved entities where they belong in navigation properties of previously retrieved entities.</span></span> <span data-ttu-id="2c59d-125">Per la query che recupera i dati correlati, è possibile usare il metodo `Load` anziché un metodo che restituisce un elenco o un oggetto, ad esempio `ToList` o `Single`.</span><span class="sxs-lookup"><span data-stu-id="2c59d-125">For the query that retrieves related data, you can use the `Load` method instead of a method that returns a list or object, such as `ToList` or `Single`.</span></span>

  ![Esempio di query separate](read-related-data/_static/separate-queries.png)

* <span data-ttu-id="2c59d-127">Caricamento esplicito.</span><span class="sxs-lookup"><span data-stu-id="2c59d-127">Explicit loading.</span></span> <span data-ttu-id="2c59d-128">Quando un'entità viene letta per la prima volta, i dati correlati non vengono recuperati.</span><span class="sxs-lookup"><span data-stu-id="2c59d-128">When the entity is first read, related data isn't retrieved.</span></span> <span data-ttu-id="2c59d-129">Il codice del caricamento consente di recuperare i dati correlati se sono necessari.</span><span class="sxs-lookup"><span data-stu-id="2c59d-129">You write code that retrieves the related data if it's needed.</span></span> <span data-ttu-id="2c59d-130">Come nel caso del caricamento eager con query separate, il caricamento esplicito ha come risultato l'invio di più query al database.</span><span class="sxs-lookup"><span data-stu-id="2c59d-130">As in the case of eager loading with separate queries, explicit loading results in multiple queries sent to the database.</span></span> <span data-ttu-id="2c59d-131">La differenza è che con il caricamento esplicito il codice specifica le proprietà di navigazione da caricare.</span><span class="sxs-lookup"><span data-stu-id="2c59d-131">The difference is that with explicit loading, the code specifies the navigation properties to be loaded.</span></span> <span data-ttu-id="2c59d-132">In Entity Framework Core 1.1, per eseguire il caricamento esplicito è possibile usare il metodo `Load`.</span><span class="sxs-lookup"><span data-stu-id="2c59d-132">In Entity Framework Core 1.1 you can use the `Load` method to do explicit loading.</span></span> <span data-ttu-id="2c59d-133">Ad esempio:</span><span class="sxs-lookup"><span data-stu-id="2c59d-133">For example:</span></span>

  ![Esempio di caricamento esplicito](read-related-data/_static/explicit-loading.png)

* <span data-ttu-id="2c59d-135">Caricamento lazy.</span><span class="sxs-lookup"><span data-stu-id="2c59d-135">Lazy loading.</span></span> <span data-ttu-id="2c59d-136">Quando un'entità viene letta per la prima volta, i dati correlati non vengono recuperati.</span><span class="sxs-lookup"><span data-stu-id="2c59d-136">When the entity is first read, related data isn't retrieved.</span></span> <span data-ttu-id="2c59d-137">La prima volta che si tenta di accedere a una proprietà di navigazione, tuttavia, i dati necessari per quest'ultima vengono recuperati automaticamente.</span><span class="sxs-lookup"><span data-stu-id="2c59d-137">However, the first time you attempt to access a navigation property, the data required for that navigation property is automatically retrieved.</span></span> <span data-ttu-id="2c59d-138">Una query viene inviata al database ogni volta che si tenta di ottenere dati da una proprietà di navigazione per la prima volta.</span><span class="sxs-lookup"><span data-stu-id="2c59d-138">A query is sent to the database each time you try to get data from a navigation property for the first time.</span></span> <span data-ttu-id="2c59d-139">Entity Framework Core 1.0 non supporta il caricamento lazy.</span><span class="sxs-lookup"><span data-stu-id="2c59d-139">Entity Framework Core 1.0 doesn't support lazy loading.</span></span>

### <a name="performance-considerations"></a><span data-ttu-id="2c59d-140">Considerazioni sulle prestazioni</span><span class="sxs-lookup"><span data-stu-id="2c59d-140">Performance considerations</span></span>

<span data-ttu-id="2c59d-141">Se si sa di aver bisogno di dati correlati per tutte le entità recuperate, il caricamento eager spesso garantisce le prestazioni migliori, perché l'invio di un'unica query al database è in genere più efficiente dell'invio di query separate per ogni entità recuperata.</span><span class="sxs-lookup"><span data-stu-id="2c59d-141">If you know you need related data for every entity retrieved, eager loading often offers the best performance, because a single query sent to the database is typically more efficient than separate queries for each entity retrieved.</span></span> <span data-ttu-id="2c59d-142">Si supponga ad esempio che ogni dipartimento abbia dieci corsi correlati.</span><span class="sxs-lookup"><span data-stu-id="2c59d-142">For example, suppose that each department has ten related courses.</span></span> <span data-ttu-id="2c59d-143">Il caricamento eager di tutti i dati correlati comporta un'unica query (join) e un unico round trip al database.</span><span class="sxs-lookup"><span data-stu-id="2c59d-143">Eager loading of all related data would result in just a single (join) query and a single round trip to the database.</span></span> <span data-ttu-id="2c59d-144">Una query separata per i corsi di ogni dipartimento comporta undici round trip al database.</span><span class="sxs-lookup"><span data-stu-id="2c59d-144">A separate query for courses for each department would result in eleven round trips to the database.</span></span> <span data-ttu-id="2c59d-145">I round trip aggiuntivi al database influiscono in modo particolarmente negativo sulle prestazioni quando la latenza è elevata.</span><span class="sxs-lookup"><span data-stu-id="2c59d-145">The extra round trips to the database are especially detrimental to performance when latency is high.</span></span>

<span data-ttu-id="2c59d-146">D'altra parte, in alcuni scenari query separate sono più efficienti.</span><span class="sxs-lookup"><span data-stu-id="2c59d-146">On the other hand, in some scenarios separate queries is more efficient.</span></span> <span data-ttu-id="2c59d-147">Il caricamento eager di tutti i dati correlati in una sola query può causare la generazione di un join molto complesso, che SQL Server non è in grado di elaborare in modo efficiente.</span><span class="sxs-lookup"><span data-stu-id="2c59d-147">Eager loading of all related data in one query might cause a very complex join to be generated, which SQL Server can't process efficiently.</span></span> <span data-ttu-id="2c59d-148">Oppure se è necessario accedere alle proprietà di navigazione di un'entità solo per un subset di un set di entità in corso di elaborazione, query separate potrebbero offrire prestazioni migliori perché il caricamento immediato di tutti i dati recupererebbe più dati di quelli necessari.</span><span class="sxs-lookup"><span data-stu-id="2c59d-148">Or if you need to access an entity's navigation properties only for a subset of a set of the entities you're processing, separate queries might perform better because eager loading of everything up front would retrieve more data than you need.</span></span> <span data-ttu-id="2c59d-149">Se le prestazioni rappresentano un aspetto essenziale, per avere la certezza di scegliere il metodo più efficiente è consigliabile testare le prestazioni di entrambi i tipi di caricamento.</span><span class="sxs-lookup"><span data-stu-id="2c59d-149">If performance is critical, it's best to test performance both ways in order to make the best choice.</span></span>

## <a name="create-a-courses-page"></a><span data-ttu-id="2c59d-150">Creare una pagina Courses</span><span class="sxs-lookup"><span data-stu-id="2c59d-150">Create a Courses page</span></span>

<span data-ttu-id="2c59d-151">L'entità Course include una proprietà di navigazione contenente l'entità Department del corso assegnato al dipartimento.</span><span class="sxs-lookup"><span data-stu-id="2c59d-151">The Course entity includes a navigation property that contains the Department entity of the department that the course is assigned to.</span></span> <span data-ttu-id="2c59d-152">Per visualizzare il nome del dipartimento assegnato in un elenco di corsi, è necessario ottenere la proprietà Name dell'entità Department nella proprietà di navigazione `Course.Department`.</span><span class="sxs-lookup"><span data-stu-id="2c59d-152">To display the name of the assigned department in a list of courses, you need to get the Name property from the Department entity that's in the `Course.Department` navigation property.</span></span>

<span data-ttu-id="2c59d-153">Creare un controller denominato CoursesController per il tipo di entità Course con le stesse opzioni del **controller MVC con visualizzazioni, tramite lo scaffolder Entity Framework** usato in precedenza per il controller Students, come illustrato nella figura seguente:</span><span class="sxs-lookup"><span data-stu-id="2c59d-153">Create a controller named CoursesController for the Course entity type, using the same options for the **MVC Controller with views, using Entity Framework** scaffolder that you did earlier for the Students controller, as shown in the following illustration:</span></span>

![Aggiungere il controller per i corsi](read-related-data/_static/add-courses-controller.png)

<span data-ttu-id="2c59d-155">Aprire *CoursesController.cs* ed esaminare il metodo `Index`.</span><span class="sxs-lookup"><span data-stu-id="2c59d-155">Open *CoursesController.cs* and examine the `Index` method.</span></span> <span data-ttu-id="2c59d-156">Lo scaffolding automatico ha specificato il caricamento eager per la proprietà di navigazione `Department` tramite il metodo `Include`.</span><span class="sxs-lookup"><span data-stu-id="2c59d-156">The automatic scaffolding has specified eager loading for the `Department` navigation property by using the `Include` method.</span></span>

<span data-ttu-id="2c59d-157">Sostituire il metodo `Index` con il codice seguente, che usa un nome più appropriato per l'`IQueryable` che restituisce entità Course (`courses` anziché `schoolContext`):</span><span class="sxs-lookup"><span data-stu-id="2c59d-157">Replace the `Index` method with the following code that uses a more appropriate name for the `IQueryable` that returns Course entities (`courses` instead of `schoolContext`):</span></span>

[!code-csharp[](intro/samples/cu/Controllers/CoursesController.cs?name=snippet_RevisedIndexMethod)]

<span data-ttu-id="2c59d-158">Aprire *Views/Courses/Index.cshtml* e sostituire il codice del modello con il codice seguente.</span><span class="sxs-lookup"><span data-stu-id="2c59d-158">Open *Views/Courses/Index.cshtml* and replace the template code with the following code.</span></span> <span data-ttu-id="2c59d-159">Le modifiche sono evidenziate:</span><span class="sxs-lookup"><span data-stu-id="2c59d-159">The changes are highlighted:</span></span>

[!code-cshtml[](intro/samples/cu/Views/Courses/Index.cshtml?highlight=4,7,15-17,34-36,44)]

<span data-ttu-id="2c59d-160">Al codice con scaffolding sono state apportate le modifiche seguenti:</span><span class="sxs-lookup"><span data-stu-id="2c59d-160">You've made the following changes to the scaffolded code:</span></span>

* <span data-ttu-id="2c59d-161">Il titolo è stato modificato da Index (Indice) a Courses (Corsi).</span><span class="sxs-lookup"><span data-stu-id="2c59d-161">Changed the heading from Index to Courses.</span></span>

* <span data-ttu-id="2c59d-162">È stata aggiunta la colonna **Number** (Numero) con il valore della proprietà `CourseID`.</span><span class="sxs-lookup"><span data-stu-id="2c59d-162">Added a **Number** column that shows the `CourseID` property value.</span></span> <span data-ttu-id="2c59d-163">Per impostazione predefinita, non viene eseguito lo scaffolding delle chiavi primarie, perché in genere non sono significative per gli utenti finali.</span><span class="sxs-lookup"><span data-stu-id="2c59d-163">By default, primary keys aren't scaffolded because normally they're meaningless to end users.</span></span> <span data-ttu-id="2c59d-164">In questo caso, tuttavia, la chiave primaria è significativa ed è necessario visualizzarla.</span><span class="sxs-lookup"><span data-stu-id="2c59d-164">However, in this case the primary key is meaningful and you want to show it.</span></span>

* <span data-ttu-id="2c59d-165">Modificare la colonna **Department** (Dipartimento) per visualizzare il nome del dipartimento.</span><span class="sxs-lookup"><span data-stu-id="2c59d-165">Changed the **Department** column to display the department name.</span></span> <span data-ttu-id="2c59d-166">Il codice visualizza la proprietà `Name` dell'entità Department che viene caricata nella proprietà di navigazione `Department`:</span><span class="sxs-lookup"><span data-stu-id="2c59d-166">The code displays the `Name` property of the Department entity that's loaded into the `Department` navigation property:</span></span>

  ```html
  @Html.DisplayFor(modelItem => item.Department.Name)
  ```

<span data-ttu-id="2c59d-167">Eseguire l'app e selezionare la scheda **Courses** (Corsi) per visualizzare l'elenco con i nomi dei dipartimenti.</span><span class="sxs-lookup"><span data-stu-id="2c59d-167">Run the app and select the **Courses** tab to see the list with department names.</span></span>

![Pagina di indice dei corsi](read-related-data/_static/courses-index.png)

## <a name="create-an-instructors-page"></a><span data-ttu-id="2c59d-169">Creare una pagina Instructors</span><span class="sxs-lookup"><span data-stu-id="2c59d-169">Create an Instructors page</span></span>

<span data-ttu-id="2c59d-170">In questa sezione verranno creati un controller e una visualizzazione per l'entità Instructor allo scopo di visualizzare la pagina Instructors:</span><span class="sxs-lookup"><span data-stu-id="2c59d-170">In this section, you'll create a controller and view for the Instructor entity in order to display the Instructors page:</span></span>

![Pagina di indice degli insegnanti](read-related-data/_static/instructors-index.png)

<span data-ttu-id="2c59d-172">Questa pagina legge e visualizza dati correlati nei modi seguenti:</span><span class="sxs-lookup"><span data-stu-id="2c59d-172">This page reads and displays related data in the following ways:</span></span>

* <span data-ttu-id="2c59d-173">L'elenco di insegnanti visualizza dati correlati provenienti dall'entità OfficeAssignment.</span><span class="sxs-lookup"><span data-stu-id="2c59d-173">The list of instructors displays related data from the OfficeAssignment entity.</span></span> <span data-ttu-id="2c59d-174">Tra le entità Instructor e OfficeAssignment esiste una relazione uno-a-zero-o-uno.</span><span class="sxs-lookup"><span data-stu-id="2c59d-174">The Instructor and OfficeAssignment entities are in a one-to-zero-or-one relationship.</span></span> <span data-ttu-id="2c59d-175">Per le entità OfficeAssignment verrà usato il caricamento eager.</span><span class="sxs-lookup"><span data-stu-id="2c59d-175">You'll use eager loading for the OfficeAssignment entities.</span></span> <span data-ttu-id="2c59d-176">Come spiegato in precedenza, il caricamento eager è in genere più efficiente quando sono necessari i dati correlati per tutte le righe recuperate della tabella primaria.</span><span class="sxs-lookup"><span data-stu-id="2c59d-176">As explained earlier, eager loading is typically more efficient when you need the related data for all retrieved rows of the primary table.</span></span> <span data-ttu-id="2c59d-177">In questo caso, si vogliono visualizzare le assegnazioni di ufficio per tutti gli insegnanti visualizzati.</span><span class="sxs-lookup"><span data-stu-id="2c59d-177">In this case, you want to display office assignments for all displayed instructors.</span></span>

* <span data-ttu-id="2c59d-178">Quando l'utente seleziona un insegnante, vengono visualizzate le entità Course correlate.</span><span class="sxs-lookup"><span data-stu-id="2c59d-178">When the user selects an instructor, related Course entities are displayed.</span></span> <span data-ttu-id="2c59d-179">Tra le entità Instructor e Course esiste una relazione molti-a-molti.</span><span class="sxs-lookup"><span data-stu-id="2c59d-179">The Instructor and Course entities are in a many-to-many relationship.</span></span> <span data-ttu-id="2c59d-180">Si userà il caricamento eager per le entità Course e le entità Department correlate.</span><span class="sxs-lookup"><span data-stu-id="2c59d-180">You'll use eager loading for the Course entities and their related Department entities.</span></span> <span data-ttu-id="2c59d-181">In questo caso, query separate potrebbero essere più efficienti, perché i corsi sono necessari solo per l'insegnante selezionato.</span><span class="sxs-lookup"><span data-stu-id="2c59d-181">In this case, separate queries might be more efficient because you need courses only for the selected instructor.</span></span> <span data-ttu-id="2c59d-182">Questo esempio, tuttavia, illustra come usare il caricamento eager per le proprietà di navigazione con entità esse stesse all'interno di proprietà di navigazione.</span><span class="sxs-lookup"><span data-stu-id="2c59d-182">However, this example shows how to use eager loading for navigation properties within entities that are themselves in navigation properties.</span></span>

* <span data-ttu-id="2c59d-183">Quando l'utente seleziona un corso, vengono visualizzati i dati correlati dal set di entità Enrollments.</span><span class="sxs-lookup"><span data-stu-id="2c59d-183">When the user selects a course, related data from the Enrollments entity set is displayed.</span></span> <span data-ttu-id="2c59d-184">Tra le entità Course ed Enrollment esiste una relazione uno-a-molti.</span><span class="sxs-lookup"><span data-stu-id="2c59d-184">The Course and Enrollment entities are in a one-to-many relationship.</span></span> <span data-ttu-id="2c59d-185">Per le entità Enrollment e le entità Student correlate si useranno query separate.</span><span class="sxs-lookup"><span data-stu-id="2c59d-185">You'll use separate queries for Enrollment entities and their related Student entities.</span></span>

### <a name="create-a-view-model-for-the-instructor-index-view"></a><span data-ttu-id="2c59d-186">Creare un modello per la visualizzazione dell'indice degli insegnanti</span><span class="sxs-lookup"><span data-stu-id="2c59d-186">Create a view model for the Instructor Index view</span></span>

<span data-ttu-id="2c59d-187">La pagina Instructors (Insegnanti) mostra i dati di tre tabelle diverse.</span><span class="sxs-lookup"><span data-stu-id="2c59d-187">The Instructors page shows data from three different tables.</span></span> <span data-ttu-id="2c59d-188">Verrà quindi creato un modello di visualizzazione che includa tre proprietà, ognuna contenente i dati di una delle tabelle.</span><span class="sxs-lookup"><span data-stu-id="2c59d-188">Therefore, you'll create a view model that includes three properties, each holding the data for one of the tables.</span></span>

<span data-ttu-id="2c59d-189">Nella cartella *SchoolViewModels* creare *InstructorIndexData.cs* e sostituire il codice esistente con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="2c59d-189">In the *SchoolViewModels* folder, create *InstructorIndexData.cs* and replace the existing code with the following code:</span></span>

[!code-csharp[](intro/samples/cu/Models/SchoolViewModels/InstructorIndexData.cs)]

### <a name="create-the-instructor-controller-and-views"></a><span data-ttu-id="2c59d-190">Creare il controller e le visualizzazioni per gli insegnanti</span><span class="sxs-lookup"><span data-stu-id="2c59d-190">Create the Instructor controller and views</span></span>

<span data-ttu-id="2c59d-191">Creare un controller per gli insegnanti con azioni di lettura/scrittura EF come illustrato nella figura seguente:</span><span class="sxs-lookup"><span data-stu-id="2c59d-191">Create an Instructors controller with EF read/write actions as shown in the following illustration:</span></span>

![Aggiungere il controller per gli insegnanti](read-related-data/_static/add-instructors-controller.png)

<span data-ttu-id="2c59d-193">Aprire *InstructorsController.cs* e aggiungere un'istruzione using tramite lo spazio dei nomi ViewModel:</span><span class="sxs-lookup"><span data-stu-id="2c59d-193">Open *InstructorsController.cs* and add a using statement for the ViewModels namespace:</span></span>

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?name=snippet_Using)]

<span data-ttu-id="2c59d-194">Sostituire il metodo Index con il codice seguente per eseguire il caricamento eager di dati correlati e inserirlo nel modello di visualizzazione.</span><span class="sxs-lookup"><span data-stu-id="2c59d-194">Replace the Index method with the following code to do eager loading of related data and put it in the view model.</span></span>

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?name=snippet_EagerLoading)]

<span data-ttu-id="2c59d-195">Il metodo accetta dati di route facoltativi (`id`) e un parametro di stringa di query (`courseID`) che forniscono i valori relativi all'ID dell'insegnante e del corso selezionati.</span><span class="sxs-lookup"><span data-stu-id="2c59d-195">The method accepts optional route data (`id`) and a query string parameter (`courseID`) that provide the ID values of the selected instructor and selected course.</span></span> <span data-ttu-id="2c59d-196">I parametri sono forniti dai collegamenti ipertestuali **Select** (Seleziona) nella pagina.</span><span class="sxs-lookup"><span data-stu-id="2c59d-196">The parameters are provided by the **Select** hyperlinks on the page.</span></span>

<span data-ttu-id="2c59d-197">Il codice inizia creando un'istanza del modello di visualizzazione e inserendola nell'elenco degli insegnanti.</span><span class="sxs-lookup"><span data-stu-id="2c59d-197">The code begins by creating an instance of the view model and putting in it the list of instructors.</span></span> <span data-ttu-id="2c59d-198">Il codice specifica il caricamento eager delle proprietà di navigazione `Instructor.OfficeAssignment` e `Instructor.CourseAssignments`.</span><span class="sxs-lookup"><span data-stu-id="2c59d-198">The code specifies eager loading for the `Instructor.OfficeAssignment` and the `Instructor.CourseAssignments` navigation properties.</span></span> <span data-ttu-id="2c59d-199">All'interno della proprietà `CourseAssignments` viene caricata la proprietà `Course` e all'interno di questa vengono caricate le proprietà `Enrollments` e `Department`, e all'interno di ogni entità `Enrollment` viene caricata la proprietà `Student`.</span><span class="sxs-lookup"><span data-stu-id="2c59d-199">Within the `CourseAssignments` property, the `Course` property is loaded, and within that, the `Enrollments` and `Department` properties are loaded, and within each `Enrollment` entity the `Student` property is loaded.</span></span>

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?name=snippet_ThenInclude)]

<span data-ttu-id="2c59d-200">Dato che la visualizzazione richiede sempre l'entità OfficeAssignment, è più efficiente recuperare quest'ultima nella stessa query.</span><span class="sxs-lookup"><span data-stu-id="2c59d-200">Since the view always requires the OfficeAssignment entity, it's more efficient to fetch that in the same query.</span></span> <span data-ttu-id="2c59d-201">Le entità Course sono necessarie quando viene selezionato un insegnante nella pagina Web. Un'unica query, quindi, è più efficiente di più query solo se nella maggior parte dei casi la pagina viene visualizzata con un corso selezionato.</span><span class="sxs-lookup"><span data-stu-id="2c59d-201">Course entities are required when an instructor is selected in the web page, so a single query is better than multiple queries only if the page is displayed more often with a course selected than without.</span></span>

<span data-ttu-id="2c59d-202">Il codice ripete `CourseAssignments` e `Course` perché da `Course` sono necessarie due proprietà.</span><span class="sxs-lookup"><span data-stu-id="2c59d-202">The code repeats `CourseAssignments` and `Course` because you need two properties from `Course`.</span></span> <span data-ttu-id="2c59d-203">La prima stringa delle chiamate `ThenInclude` ottiene `CourseAssignment.Course`, `Course.Enrollments` e `Enrollment.Student`.</span><span class="sxs-lookup"><span data-stu-id="2c59d-203">The first string of `ThenInclude` calls gets `CourseAssignment.Course`, `Course.Enrollments`, and `Enrollment.Student`.</span></span>

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?name=snippet_ThenInclude&highlight=3-6)]

<span data-ttu-id="2c59d-204">A questo punto nel codice, un'altra chiamata `ThenInclude` riguarda le proprietà di navigazione di `Student`, che non sono necessarie.</span><span class="sxs-lookup"><span data-stu-id="2c59d-204">At that point in the code, another `ThenInclude` would be for navigation properties of `Student`, which you don't need.</span></span> <span data-ttu-id="2c59d-205">Ma chiamando `Include` il codice ricomincia con le proprietà di `Instructor`. È quindi necessario ripetere la sequenza, questa volta specificando `Course.Department` anziché `Course.Enrollments`.</span><span class="sxs-lookup"><span data-stu-id="2c59d-205">But calling `Include` starts over with `Instructor` properties, so you have to go through the chain again, this time specifying `Course.Department` instead of `Course.Enrollments`.</span></span>

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?name=snippet_ThenInclude&highlight=7-9)]

<span data-ttu-id="2c59d-206">Il codice seguente viene eseguito quando è stato selezionato un insegnante.</span><span class="sxs-lookup"><span data-stu-id="2c59d-206">The following code executes when an instructor was selected.</span></span> <span data-ttu-id="2c59d-207">L'insegnante selezionato viene recuperato dall'elenco di insegnanti nel modello di visualizzazione.</span><span class="sxs-lookup"><span data-stu-id="2c59d-207">The selected instructor is retrieved from the list of instructors in the view model.</span></span> <span data-ttu-id="2c59d-208">La proprietà `Courses` del modello di visualizzazione viene quindi caricata con le entità Course dalla proprietà di navigazione `CourseAssignments` di tale insegnante.</span><span class="sxs-lookup"><span data-stu-id="2c59d-208">The view model's `Courses` property is then loaded with the Course entities from that instructor's `CourseAssignments` navigation property.</span></span>

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?range=56-62)]

<span data-ttu-id="2c59d-209">Il metodo `Where` restituisce una raccolta, ma in questo caso i criteri passati a tale metodo hanno come risultato la restituzione di una sola entità Instructor.</span><span class="sxs-lookup"><span data-stu-id="2c59d-209">The `Where` method returns a collection, but in this case the criteria passed to that method result in only a single Instructor entity being returned.</span></span> <span data-ttu-id="2c59d-210">Il metodo `Single` converte la raccolta in un'entità Instructor singola, che consente l'accesso alla proprietà `CourseAssignments` di tale entità.</span><span class="sxs-lookup"><span data-stu-id="2c59d-210">The `Single` method converts the collection into a single Instructor entity, which gives you access to that entity's `CourseAssignments` property.</span></span> <span data-ttu-id="2c59d-211">La proprietà `CourseAssignments` contiene entità `CourseAssignment`, di cui si vogliono solo le entità `Course` correlate.</span><span class="sxs-lookup"><span data-stu-id="2c59d-211">The `CourseAssignments` property contains `CourseAssignment` entities, from which you want only the related `Course` entities.</span></span>

<span data-ttu-id="2c59d-212">Usare il metodo `Single` per una raccolta quando si sa che la raccolta ha un solo elemento.</span><span class="sxs-lookup"><span data-stu-id="2c59d-212">You use the `Single` method on a collection when you know the collection will have only one item.</span></span> <span data-ttu-id="2c59d-213">Il metodo Single genera un'eccezione se la raccolta passata è vuota o se contiene più elementi.</span><span class="sxs-lookup"><span data-stu-id="2c59d-213">The Single method throws an exception if the collection passed to it's empty or if there's more than one item.</span></span> <span data-ttu-id="2c59d-214">In alternativa, è possibile usare `SingleOrDefault`, che restituisce un valore predefinito (Null in questo caso) se la raccolta è vuota.</span><span class="sxs-lookup"><span data-stu-id="2c59d-214">An alternative is `SingleOrDefault`, which returns a default value (null in this case) if the collection is empty.</span></span> <span data-ttu-id="2c59d-215">In questo caso, tuttavia, ciò avrebbe comunque come risultato un'eccezione (a causa del tentativo di cercare una proprietà `Courses` in un riferimento Null) e il messaggio di eccezione indicherebbe meno chiaramente la causa del problema.</span><span class="sxs-lookup"><span data-stu-id="2c59d-215">However, in this case that would still result in an exception (from trying to find a `Courses` property on a null reference), and the exception message would less clearly indicate the cause of the problem.</span></span> <span data-ttu-id="2c59d-216">Quando si chiama il metodo `Single`, è anche possibile passare la condizione Where anziché chiamare il metodo `Where` separatamente:</span><span class="sxs-lookup"><span data-stu-id="2c59d-216">When you call the `Single` method, you can also pass in the Where condition instead of calling the `Where` method separately:</span></span>

```csharp
.Single(i => i.ID == id.Value)
```

<span data-ttu-id="2c59d-217">Anziché:</span><span class="sxs-lookup"><span data-stu-id="2c59d-217">Instead of:</span></span>

```csharp
.Where(i => i.ID == id.Value).Single()
```

<span data-ttu-id="2c59d-218">Se è stato selezionato un corso, questo viene quindi recuperato dall'elenco dei corsi nel modello di visualizzazione.</span><span class="sxs-lookup"><span data-stu-id="2c59d-218">Next, if a course was selected, the selected course is retrieved from the list of courses in the view model.</span></span> <span data-ttu-id="2c59d-219">La proprietà `Enrollments` del modello di visualizzazione viene quindi caricata con le entità Enrollment dalla proprietà di navigazione `Enrollments` di tale corso.</span><span class="sxs-lookup"><span data-stu-id="2c59d-219">Then the view model's `Enrollments` property is loaded with the Enrollment entities from that course's `Enrollments` navigation property.</span></span>

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?range=64-69)]

### <a name="modify-the-instructor-index-view"></a><span data-ttu-id="2c59d-220">Modificare la visualizzazione dell'indice degli insegnanti</span><span class="sxs-lookup"><span data-stu-id="2c59d-220">Modify the Instructor Index view</span></span>

<span data-ttu-id="2c59d-221">In *Views/Instructors/Index.cshtml* sostituire il codice del modello con il codice seguente.</span><span class="sxs-lookup"><span data-stu-id="2c59d-221">In *Views/Instructors/Index.cshtml*, replace the template code with the following code.</span></span> <span data-ttu-id="2c59d-222">Le modifiche sono evidenziate.</span><span class="sxs-lookup"><span data-stu-id="2c59d-222">The changes are highlighted.</span></span>

::: moniker range=">= aspnetcore-2.2"

[!code-cshtml[](intro/samples/5cu-snap/Views/Instructors/Index.cshtml?range=1-62&highlight=1,3-7,15-19,24,26-31,41-52,54)]

::: moniker-end

::: moniker range="<= aspnetcore-2.1"

[!code-cshtml[](intro/samples/cu/Views/Instructors/Index1.cshtml?range=1-62&highlight=1,3-7,15-19,24,26-31,41-52,54)]

::: moniker-end

<span data-ttu-id="2c59d-223">Al codice esistente sono state apportate le modifiche seguenti:</span><span class="sxs-lookup"><span data-stu-id="2c59d-223">You've made the following changes to the existing code:</span></span>

* <span data-ttu-id="2c59d-224">La classe del modello è stata modificata in `InstructorIndexData`.</span><span class="sxs-lookup"><span data-stu-id="2c59d-224">Changed the model class to `InstructorIndexData`.</span></span>

* <span data-ttu-id="2c59d-225">Il titolo pagina è stato modificato da **Index** (Indice) a **Instructors** (Insegnanti).</span><span class="sxs-lookup"><span data-stu-id="2c59d-225">Changed the page title from **Index** to **Instructors**.</span></span>

* <span data-ttu-id="2c59d-226">È stata aggiunta la colonna **Office** (Ufficio) che visualizza `item.OfficeAssignment.Location` solo se `item.OfficeAssignment` non è Null.</span><span class="sxs-lookup"><span data-stu-id="2c59d-226">Added an **Office** column that displays `item.OfficeAssignment.Location` only if `item.OfficeAssignment` isn't null.</span></span> <span data-ttu-id="2c59d-227">Poiché questa è una relazione uno-a-zero-o-uno, potrebbe non esserci un'entità OfficeAssignment correlata.</span><span class="sxs-lookup"><span data-stu-id="2c59d-227">(Because this is a one-to-zero-or-one relationship, there might not be a related OfficeAssignment entity.)</span></span>

  ```cshtml
  @if (item.OfficeAssignment != null)
  {
      @item.OfficeAssignment.Location
  }
  ```

* <span data-ttu-id="2c59d-228">È stata aggiunta la colonna **Courses** (Corsi) che visualizza i corsi tenuti da ogni insegnante.</span><span class="sxs-lookup"><span data-stu-id="2c59d-228">Added a **Courses** column that displays courses taught by each instructor.</span></span> <span data-ttu-id="2c59d-229">Per ulteriori informazioni, vedere la sezione relativa alla [transizione di riga esplicita](xref:mvc/views/razor#explicit-line-transition) dell' Razor articolo della sintassi.</span><span class="sxs-lookup"><span data-stu-id="2c59d-229">For more information, see the [Explicit line transition](xref:mvc/views/razor#explicit-line-transition) section of the Razor syntax article.</span></span>

* <span data-ttu-id="2c59d-230">Aggiunto codice che aggiunge in modo condizionale una classe CSS bootstrap all' `tr` elemento dell'insegnante selezionato.</span><span class="sxs-lookup"><span data-stu-id="2c59d-230">Added code that conditionally adds a Bootstrap CSS class to the `tr` element of the selected instructor.</span></span> <span data-ttu-id="2c59d-231">Questa classe imposta un colore di sfondo per la riga selezionata.</span><span class="sxs-lookup"><span data-stu-id="2c59d-231">This class sets a background color for the selected row.</span></span>

* <span data-ttu-id="2c59d-232">È stato aggiunto un nuovo collegamento ipertestuale con etichetta **Select** (Seleziona) immediatamente prima degli altri collegamenti in ogni riga.Ciò comporta l'invio dell'ID dell'insegnante selezionato al metodo `Index`.</span><span class="sxs-lookup"><span data-stu-id="2c59d-232">Added a new hyperlink labeled **Select** immediately before the other links in each row, which causes the selected instructor's ID to be sent to the `Index` method.</span></span>

  ```cshtml
  <a asp-action="Index" asp-route-id="@item.ID">Select</a> |
  ```

<span data-ttu-id="2c59d-233">Eseguire l'app e selezionare la scheda **Instructors (insegnanti** ). La pagina Visualizza la proprietà location delle entità OfficeAssignment correlate e una cella della tabella vuota quando non esiste alcuna entità OfficeAssignment correlata.</span><span class="sxs-lookup"><span data-stu-id="2c59d-233">Run the app and select the **Instructors** tab. The page displays the Location property of related OfficeAssignment entities and an empty table cell when there's no related OfficeAssignment entity.</span></span>

![Pagina di indice degli insegnanti con nessuna selezione](read-related-data/_static/instructors-index-no-selection.png)

<span data-ttu-id="2c59d-235">Nel file *Views/Instructors/Index.cshtml*, dopo l'elemento di chiusura della tabella (alla fine del file), aggiungere il codice seguente.</span><span class="sxs-lookup"><span data-stu-id="2c59d-235">In the *Views/Instructors/Index.cshtml* file, after the closing table element (at the end of the file), add the following code.</span></span> <span data-ttu-id="2c59d-236">Quando è selezionato un insegnante, Questo codice visualizza un elenco dei corsi correlati all'insegnante stesso.</span><span class="sxs-lookup"><span data-stu-id="2c59d-236">This code displays a list of courses related to an instructor when an instructor is selected.</span></span>

[!code-cshtml[](intro/samples/cu/Views/Instructors/Index1.cshtml?range=63-98)]

<span data-ttu-id="2c59d-237">Questo codice legge la proprietà `Courses` del modello di visualizzazione per visualizzare l'elenco dei corsi.</span><span class="sxs-lookup"><span data-stu-id="2c59d-237">This code reads the `Courses` property of the view model to display a list of courses.</span></span> <span data-ttu-id="2c59d-238">Fornisce anche il collegamento ipertestuale **Select** (Seleziona) che invia l'ID del corso selezionato al metodo di azione `Index`.</span><span class="sxs-lookup"><span data-stu-id="2c59d-238">It also provides a **Select** hyperlink that sends the ID of the selected course to the `Index` action method.</span></span>

<span data-ttu-id="2c59d-239">Aggiornare la pagina e selezionare un insegnante.</span><span class="sxs-lookup"><span data-stu-id="2c59d-239">Refresh the page and select an instructor.</span></span> <span data-ttu-id="2c59d-240">È ora possibile vedere una griglia con i corsi assegnati all'insegnante selezionato. Per ogni corso è possibile vedere il nome del dipartimento assegnato.</span><span class="sxs-lookup"><span data-stu-id="2c59d-240">Now you see a grid that displays courses assigned to the selected instructor, and for each course you see the name of the assigned department.</span></span>

![Pagina di indice degli insegnanti con un insegnante selezionato](read-related-data/_static/instructors-index-instructor-selected.png)

<span data-ttu-id="2c59d-242">Dopo il blocco di codice appena aggiunto, aggiungere il codice seguente.</span><span class="sxs-lookup"><span data-stu-id="2c59d-242">After the code block you just added, add the following code.</span></span> <span data-ttu-id="2c59d-243">Quando è selezionato un corso, questo codice visualizza l'elenco degli studenti iscritti al corso selezionato.</span><span class="sxs-lookup"><span data-stu-id="2c59d-243">This displays a list of the students who are enrolled in a course when that course is selected.</span></span>

[!code-cshtml[](intro/samples/cu/Views/Instructors/Index1.cshtml?range=103-125)]

<span data-ttu-id="2c59d-244">Il codice legge la proprietà Enrollments del modello di visualizzazione per visualizzare l'elenco degli studenti iscritti al corso.</span><span class="sxs-lookup"><span data-stu-id="2c59d-244">This code reads the Enrollments property of the view model in order to display a list of students enrolled in the course.</span></span>

<span data-ttu-id="2c59d-245">Aggiornare di nuovo la pagina e selezionare un insegnante.</span><span class="sxs-lookup"><span data-stu-id="2c59d-245">Refresh the page again and select an instructor.</span></span> <span data-ttu-id="2c59d-246">Selezionare quindi un corso per visualizzare l'elenco degli studenti iscritti e i voti corrispondenti.</span><span class="sxs-lookup"><span data-stu-id="2c59d-246">Then select a course to see the list of enrolled students and their grades.</span></span>

![Pagina di indice degli insegnanti con un corso selezionato](read-related-data/_static/instructors-index.png)

## <a name="about-explicit-loading"></a><span data-ttu-id="2c59d-248">Informazioni sul caricamento esplicito</span><span class="sxs-lookup"><span data-stu-id="2c59d-248">About explicit loading</span></span>

<span data-ttu-id="2c59d-249">Quando è stato recuperato l'elenco degli insegnanti in *InstructorsController.cs*, per la proprietà di navigazione `CourseAssignments` è stato specificato il caricamento eager.</span><span class="sxs-lookup"><span data-stu-id="2c59d-249">When you retrieved the list of instructors in *InstructorsController.cs*, you specified eager loading for the `CourseAssignments` navigation property.</span></span>

<span data-ttu-id="2c59d-250">Si supponga che gli utenti vogliano visualizzare solo raramente le iscrizioni per un corso e un insegnante selezionati.</span><span class="sxs-lookup"><span data-stu-id="2c59d-250">Suppose you expected users to only rarely want to see enrollments in a selected instructor and course.</span></span> <span data-ttu-id="2c59d-251">In tal caso, è consigliabile caricare i dati delle iscrizioni solo se richiesti.</span><span class="sxs-lookup"><span data-stu-id="2c59d-251">In that case, you might want to load the enrollment data only if it's requested.</span></span> <span data-ttu-id="2c59d-252">Per vedere un esempio di come eseguire il caricamento esplicito, sostituire il metodo `Index` con il codice seguente, che rimuove il caricamento eager per Enrollments e carica questa proprietà in modo esplicito.</span><span class="sxs-lookup"><span data-stu-id="2c59d-252">To see an example of how to do explicit loading, replace the `Index` method with the following code, which removes eager loading for Enrollments and loads that property explicitly.</span></span> <span data-ttu-id="2c59d-253">Le modifiche al codice sono evidenziate.</span><span class="sxs-lookup"><span data-stu-id="2c59d-253">The code changes are highlighted.</span></span>

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?name=snippet_ExplicitLoading&highlight=23-29)]

<span data-ttu-id="2c59d-254">Il nuovo codice rilascia le chiamate al metodo *ThenInclude* per i dati delle iscrizioni dal codice che recupera entità Instructor.</span><span class="sxs-lookup"><span data-stu-id="2c59d-254">The new code drops the *ThenInclude* method calls for enrollment data from the code that retrieves instructor entities.</span></span> <span data-ttu-id="2c59d-255">Elimina anche `AsNoTracking`.</span><span class="sxs-lookup"><span data-stu-id="2c59d-255">It also drops `AsNoTracking`.</span></span>  <span data-ttu-id="2c59d-256">Se sono selezionati un insegnante e un corso, il codice evidenziato recupera le entità Enrollment per il corso selezionato e le entità Student per ogni Enrollment.</span><span class="sxs-lookup"><span data-stu-id="2c59d-256">If an instructor and course are selected, the highlighted code retrieves Enrollment entities for the selected course, and Student entities for each Enrollment.</span></span>

<span data-ttu-id="2c59d-257">Eseguire l'app e passare alla pagina di indice degli insegnanti. Non si noterà alcuna differenza in ciò che viene visualizzato nella pagina, anche se è stata modificata la modalità di recupero dei dati.</span><span class="sxs-lookup"><span data-stu-id="2c59d-257">Run the app, go to the Instructors Index page now and you'll see no difference in what's displayed on the page, although you've changed how the data is retrieved.</span></span>

## <a name="get-the-code"></a><span data-ttu-id="2c59d-258">Ottenere il codice</span><span class="sxs-lookup"><span data-stu-id="2c59d-258">Get the code</span></span>

[<span data-ttu-id="2c59d-259">Scaricare o visualizzare l'applicazione completata.</span><span class="sxs-lookup"><span data-stu-id="2c59d-259">Download or view the completed application.</span></span>](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/data/ef-mvc/intro/samples/cu-final)

## <a name="next-steps"></a><span data-ttu-id="2c59d-260">Passaggi successivi</span><span class="sxs-lookup"><span data-stu-id="2c59d-260">Next steps</span></span>

<span data-ttu-id="2c59d-261">In questa esercitazione:</span><span class="sxs-lookup"><span data-stu-id="2c59d-261">In this tutorial, you:</span></span>

> [!div class="checklist"]
> * <span data-ttu-id="2c59d-262">Come caricare i dati correlati</span><span class="sxs-lookup"><span data-stu-id="2c59d-262">Learned how to load related data</span></span>
> * <span data-ttu-id="2c59d-263">Creazione di una pagina Courses</span><span class="sxs-lookup"><span data-stu-id="2c59d-263">Created a Courses page</span></span>
> * <span data-ttu-id="2c59d-264">Creazione di una pagina Instructors</span><span class="sxs-lookup"><span data-stu-id="2c59d-264">Created an Instructors page</span></span>
> * <span data-ttu-id="2c59d-265">Raccolta di informazioni sul caricamento esplicito</span><span class="sxs-lookup"><span data-stu-id="2c59d-265">Learned about explicit loading</span></span>

<span data-ttu-id="2c59d-266">Passare all'esercitazione successiva per informazioni su come aggiornare i dati correlati.</span><span class="sxs-lookup"><span data-stu-id="2c59d-266">Advance to the next tutorial to learn how to update related data.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="2c59d-267">Aggiornare dati correlati</span><span class="sxs-lookup"><span data-stu-id="2c59d-267">Update related data</span></span>](update-related-data.md)
