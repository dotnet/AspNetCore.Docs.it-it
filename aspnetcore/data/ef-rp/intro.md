---
title: Razor Pagine con Entity Framework Core in ASP.NET Core-esercitazione 1 di 8
author: rick-anderson
description: Mostra come creare un'app per le Razor pagine usando Entity Framework Core
ms.author: riande
ms.custom: mvc, seodec18
ms.date: 9/26/2020
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
uid: data/ef-rp/intro
ms.openlocfilehash: e797228d498ec9b269e718c099a0532bbb4504d3
ms.sourcegitcommit: e311cfb77f26a0a23681019bd334929d1aaeda20
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/03/2021
ms.locfileid: "99530242"
---
# <a name="razor-pages-with-entity-framework-core-in-aspnet-core---tutorial-1-of-8"></a><span data-ttu-id="2679e-103">Razor Pagine con Entity Framework Core in ASP.NET Core-esercitazione 1 di 8</span><span class="sxs-lookup"><span data-stu-id="2679e-103">Razor Pages with Entity Framework Core in ASP.NET Core - Tutorial 1 of 8</span></span>

<span data-ttu-id="2679e-104">Di [Tom Dykstra](https://github.com/tdykstra) e [Rick Anderson](https://twitter.com/RickAndMSFT)</span><span class="sxs-lookup"><span data-stu-id="2679e-104">By [Tom Dykstra](https://github.com/tdykstra) and [Rick Anderson](https://twitter.com/RickAndMSFT)</span></span>

::: moniker range=">= aspnetcore-5.0"

<span data-ttu-id="2679e-105">Questo è il primo di una serie di esercitazioni che illustrano come usare Entity Framework (EF) Core in un'app [ASP.NET Core Razor pages](xref:razor-pages/index) .</span><span class="sxs-lookup"><span data-stu-id="2679e-105">This is the first in a series of tutorials that show how to use Entity Framework (EF) Core in an [ASP.NET Core Razor Pages](xref:razor-pages/index) app.</span></span> <span data-ttu-id="2679e-106">Con queste esercitazioni viene creato un sito Web per una fittizia Contoso University.</span><span class="sxs-lookup"><span data-stu-id="2679e-106">The tutorials build a web site for a fictional Contoso University.</span></span> <span data-ttu-id="2679e-107">Il sito include funzionalità, come ad esempio l'ammissione di studenti, la creazione di corsi e le assegnazioni degli insegnati.</span><span class="sxs-lookup"><span data-stu-id="2679e-107">The site includes functionality such as student admission, course creation, and instructor assignments.</span></span> <span data-ttu-id="2679e-108">Nell'esercitazione viene usato l'approccio Code First.</span><span class="sxs-lookup"><span data-stu-id="2679e-108">The tutorial uses the code first approach.</span></span> <span data-ttu-id="2679e-109">Per informazioni su come seguire questa esercitazione usando il primo approccio al database, vedere [questo problema di GitHub](https://github.com/dotnet/AspNetCore.Docs/issues/16897).</span><span class="sxs-lookup"><span data-stu-id="2679e-109">For information on following this tutorial using the database first approach, see [this Github issue](https://github.com/dotnet/AspNetCore.Docs/issues/16897).</span></span>

[<span data-ttu-id="2679e-110">Scaricare o visualizzare l'app completata.</span><span class="sxs-lookup"><span data-stu-id="2679e-110">Download or view the completed app.</span></span>](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/data/ef-rp/intro/samples) <span data-ttu-id="2679e-111">[Istruzioni per il download](xref:index#how-to-download-a-sample).</span><span class="sxs-lookup"><span data-stu-id="2679e-111">[Download instructions](xref:index#how-to-download-a-sample).</span></span>

## <a name="prerequisites"></a><span data-ttu-id="2679e-112">Prerequisiti</span><span class="sxs-lookup"><span data-stu-id="2679e-112">Prerequisites</span></span>

* <span data-ttu-id="2679e-113">Se non si ha familiarità con Razor le pagine, vedere la serie di esercitazioni [introduttive sulle Razor pagine](xref:tutorials/razor-pages/razor-pages-start) prima di iniziare.</span><span class="sxs-lookup"><span data-stu-id="2679e-113">If you're new to Razor Pages, go through the [Get started with Razor Pages](xref:tutorials/razor-pages/razor-pages-start) tutorial series before starting this one.</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="2679e-114">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="2679e-114">Visual Studio</span></span>](#tab/visual-studio)

[!INCLUDE[VS prereqs](~/includes/net-core-prereqs-vs-5.0.md)]

# <a name="visual-studio-code"></a>[<span data-ttu-id="2679e-115">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="2679e-115">Visual Studio Code</span></span>](#tab/visual-studio-code)

[!INCLUDE[VS Code prereqs](~/includes/net-core-prereqs-vsc-5.0.md)]

---

## <a name="database-engines"></a><span data-ttu-id="2679e-116">Motori di database</span><span class="sxs-lookup"><span data-stu-id="2679e-116">Database engines</span></span>

<span data-ttu-id="2679e-117">Le istruzioni per Visual Studio usano [SQL Server Local DB](/sql/database-engine/configure-windows/sql-server-2016-express-localdb), una versione di SQL Server Express eseguita solo in Windows.</span><span class="sxs-lookup"><span data-stu-id="2679e-117">The Visual Studio instructions use [SQL Server LocalDB](/sql/database-engine/configure-windows/sql-server-2016-express-localdb), a version of SQL Server Express that runs only on Windows.</span></span>

<span data-ttu-id="2679e-118">Le istruzioni per Visual Studio Code usano [SQLite](https://www.sqlite.org/), un motore di database multipiattaforma.</span><span class="sxs-lookup"><span data-stu-id="2679e-118">The Visual Studio Code instructions use [SQLite](https://www.sqlite.org/), a cross-platform database engine.</span></span>

<span data-ttu-id="2679e-119">Se si sceglie di usare SQLite, scaricare e installare uno strumento di terze parti per la gestione e la visualizzazione di un database SQLite, ad esempio [DB Browser per SQLite](https://sqlitebrowser.org/).</span><span class="sxs-lookup"><span data-stu-id="2679e-119">If you choose to use SQLite, download and install a third-party tool for managing and viewing a SQLite database, such as [DB Browser for SQLite](https://sqlitebrowser.org/).</span></span>

## <a name="troubleshooting"></a><span data-ttu-id="2679e-120">Risoluzione dei problemi</span><span class="sxs-lookup"><span data-stu-id="2679e-120">Troubleshooting</span></span>

<span data-ttu-id="2679e-121">Se si verifica un problema che non è possibile risolvere, confrontare il codice con il [progetto completato](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/data/ef-rp/intro/samples).</span><span class="sxs-lookup"><span data-stu-id="2679e-121">If you run into a problem you can't resolve, compare your code to the [completed project](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/data/ef-rp/intro/samples).</span></span> <span data-ttu-id="2679e-122">Un buon metodo per ottenere assistenza è quello di pubblicare una domanda in StackOverflow.com con il [tag ASP.NET Core](https://stackoverflow.com/questions/tagged/asp.net-core) o il [tag EF Core](https://stackoverflow.com/questions/tagged/entity-framework-core).</span><span class="sxs-lookup"><span data-stu-id="2679e-122">A good way to get help is by posting a question to StackOverflow.com, using the [ASP.NET Core tag](https://stackoverflow.com/questions/tagged/asp.net-core) or the [EF Core tag](https://stackoverflow.com/questions/tagged/entity-framework-core).</span></span>

## <a name="the-sample-app"></a><span data-ttu-id="2679e-123">App di esempio</span><span class="sxs-lookup"><span data-stu-id="2679e-123">The sample app</span></span>

<span data-ttu-id="2679e-124">L'applicazione compilata in queste esercitazioni è il sito Web di base di un'università.</span><span class="sxs-lookup"><span data-stu-id="2679e-124">The app built in these tutorials is a basic university web site.</span></span> <span data-ttu-id="2679e-125">Gli utenti possono visualizzare e aggiornare le informazioni che riguardano studenti, corsi e insegnanti.</span><span class="sxs-lookup"><span data-stu-id="2679e-125">Users can view and update student, course, and instructor information.</span></span> <span data-ttu-id="2679e-126">Di seguito sono disponibili alcune schermate dell'esercitazione.</span><span class="sxs-lookup"><span data-stu-id="2679e-126">Here are a few of the screens created in the tutorial.</span></span>

![Pagina Student Index (Indice degli studenti)](intro/_static/students-index30.png)

![Pagina Students Edit (Modifica studenti)](intro/_static/student-edit30.png)

<span data-ttu-id="2679e-129">Lo stile dell'interfaccia utente del sito è basato sui modelli di progetto predefiniti.</span><span class="sxs-lookup"><span data-stu-id="2679e-129">The UI style of this site is based on the built-in project templates.</span></span> <span data-ttu-id="2679e-130">L'obiettivo dell'esercitazione è quello di usare EF Core con ASP.NET Core, non come personalizzare l'interfaccia utente.</span><span class="sxs-lookup"><span data-stu-id="2679e-130">The tutorial's focus is on how to use EF Core with ASP.NET Core, not how to customize the UI.</span></span>

<!-- 
Follow the link at the top of the page to get the source code for the completed project. The *cu50* folder has the code for the ASP.NET Core 5.0 version of the tutorial. Files that reflect the state of the code for tutorials 1-7 can be found in the *cu50snapshots* folder.

# [Visual Studio](#tab/visual-studio)

To run the app after downloading the completed project:

* Build the project.
* In Package Manager Console (PMC) run the following command:

  ```powershell
  Update-Database
  ```

* Run the project to seed the database.

# [Visual Studio Code](#tab/visual-studio-code)

To run the app after downloading the completed project:

* In *Program.cs*, remove the comments from `// webBuilder.UseStartup<StartupSQLite>();`  so `StartupSQLite` is used.
* Copy the contents of *appSettingsSQLite.json* into *appSettings.json*.
* Delete the *Migrations* folder, and rename *MigrationsSQL* to *Migrations*.
* Do a global search for `#if SQLiteVersion` and remove `#if SQLiteVersion` and the associated `#endif` statement.
* Build the project.
* At a command prompt in the project folder, run the following commands:

  ```dotnetcli
  dotnet tool install --global dotnet-ef -v 5.0.0-*
  dotnet ef database update
  ```

* In your SQLite tool, run the following SQL statement:

  ```sql
  UPDATE Department SET RowVersion = randomblob(8)
  ```

* Run the project to seed the database.

---

-->

## <a name="create-the-web-app-project"></a><span data-ttu-id="2679e-131">Creare il progetto di app Web</span><span class="sxs-lookup"><span data-stu-id="2679e-131">Create the web app project</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="2679e-132">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="2679e-132">Visual Studio</span></span>](#tab/visual-studio)

1. <span data-ttu-id="2679e-133">Avviare Visual Studio e selezionare **Crea un nuovo progetto**.</span><span class="sxs-lookup"><span data-stu-id="2679e-133">Start Visual Studio and select **Create a new project**.</span></span>
1. <span data-ttu-id="2679e-134">Nella finestra di dialogo **Crea un nuovo progetto** selezionare **ASP.NET Core applicazione Web** > **Avanti**.</span><span class="sxs-lookup"><span data-stu-id="2679e-134">In the **Create a new project** dialog, select **ASP.NET Core Web Application** > **Next**.</span></span>
1. <span data-ttu-id="2679e-135">Nella finestra di dialogo **Configura nuovo progetto** immettere `ContosoUniversity` per **nome progetto**.</span><span class="sxs-lookup"><span data-stu-id="2679e-135">In the **Configure your new project** dialog, enter `ContosoUniversity` for **Project name**.</span></span> <span data-ttu-id="2679e-136">È importante usare questo nome esatto, incluse le maiuscole, in modo che ogni `namespace` corrisponda al momento della copia del codice.</span><span class="sxs-lookup"><span data-stu-id="2679e-136">It's important to use this exact name including capitalization, so each `namespace` matches when code is copied.</span></span>
1. <span data-ttu-id="2679e-137">Selezionare **Crea**.</span><span class="sxs-lookup"><span data-stu-id="2679e-137">Select **Create**.</span></span>
1. <span data-ttu-id="2679e-138">Nella finestra di dialogo **Crea una nuova applicazione web ASP.NET Core** selezionare:</span><span class="sxs-lookup"><span data-stu-id="2679e-138">In the **Create a new ASP.NET Core web application** dialog, select:</span></span>
    1. <span data-ttu-id="2679e-139">**.NET Core** e **ASP.NET Core 5,0** negli elenchi a discesa.</span><span class="sxs-lookup"><span data-stu-id="2679e-139">**.NET Core** and **ASP.NET Core 5.0** in the dropdowns.</span></span>
    1. <span data-ttu-id="2679e-140">**App Web ASP.NET Core**.</span><span class="sxs-lookup"><span data-stu-id="2679e-140">**ASP.NET Core Web App**.</span></span>
    1. <span data-ttu-id="2679e-141"> 
       Crea ![ Finestra di dialogo nuovo progetto ASP.NET Core](~/data/ef-rp/intro/_static/new-aspnet5.png)</span><span class="sxs-lookup"><span data-stu-id="2679e-141">**Create**
![New ASP.NET Core Project dialog](~/data/ef-rp/intro/_static/new-aspnet5.png)</span></span>
    
# <a name="visual-studio-code"></a>[<span data-ttu-id="2679e-142">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="2679e-142">Visual Studio Code</span></span>](#tab/visual-studio-code)

* <span data-ttu-id="2679e-143">In un terminale passare alla cartella in cui deve essere creata la cartella del progetto.</span><span class="sxs-lookup"><span data-stu-id="2679e-143">In a terminal, navigate to the folder in which the project folder should be created.</span></span>
* <span data-ttu-id="2679e-144">Eseguire i comandi seguenti per creare un Razor progetto di pagine e `cd` nella nuova cartella di progetto:</span><span class="sxs-lookup"><span data-stu-id="2679e-144">Run the following commands to create a Razor Pages project and `cd` into the new project folder:</span></span>

  ```dotnetcli
  dotnet new webapp -o ContosoUniversity
  cd ContosoUniversity  
  ```

---

## <a name="set-up-the-site-style"></a><span data-ttu-id="2679e-145">Impostare lo stile del sito</span><span class="sxs-lookup"><span data-stu-id="2679e-145">Set up the site style</span></span>

<span data-ttu-id="2679e-146">Copiare e incollare il codice seguente nel file *pages/Shared/_Layout. cshtml* :</span><span class="sxs-lookup"><span data-stu-id="2679e-146">Copy and paste the following code into the *Pages/Shared/_Layout.cshtml* file:</span></span>

[!code-cshtml[Main](intro/samples/cu50/Pages/Shared/_Layout.cshtml?highlight=6,14,21-35,49)]

<span data-ttu-id="2679e-147">Il file di layout imposta l'intestazione, il piè di pagina e il menu del sito.</span><span class="sxs-lookup"><span data-stu-id="2679e-147">The layout file sets the site header, footer, and menu.</span></span> <span data-ttu-id="2679e-148">Il codice precedente apporta le modifiche seguenti:</span><span class="sxs-lookup"><span data-stu-id="2679e-148">The preceding code makes the following changes:</span></span>

* <span data-ttu-id="2679e-149">Ogni occorrenza di "ContosoUniversity" in "Contoso University".</span><span class="sxs-lookup"><span data-stu-id="2679e-149">Each occurrence of "ContosoUniversity" to "Contoso University".</span></span> <span data-ttu-id="2679e-150">Le occorrenze sono tre.</span><span class="sxs-lookup"><span data-stu-id="2679e-150">There are three occurrences.</span></span>
* <span data-ttu-id="2679e-151">Le voci del menu **Home** e **privacy** verranno eliminate.</span><span class="sxs-lookup"><span data-stu-id="2679e-151">The **Home** and **Privacy** menu entries are deleted.</span></span>
* <span data-ttu-id="2679e-152">Vengono aggiunte voci per **informazioni su**, **studenti**, **corsi**, **docenti** e **reparti**.</span><span class="sxs-lookup"><span data-stu-id="2679e-152">Entries are added for **About**, **Students**, **Courses**, **Instructors**, and **Departments**.</span></span>

<span data-ttu-id="2679e-153">In *pages/index. cshtml* sostituire il contenuto del file con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="2679e-153">In *Pages/Index.cshtml*, replace the contents of the file with the following code:</span></span>

[!code-cshtml[Main](intro/samples/cu50/Pages/Index.cshtml)]

<span data-ttu-id="2679e-154">Il codice precedente sostituisce il testo relativo ASP.NET Core con il testo relativo a questa app.</span><span class="sxs-lookup"><span data-stu-id="2679e-154">The preceding code replaces the text about ASP.NET Core with text about this app.</span></span>

<span data-ttu-id="2679e-155">Eseguire l'app per verificare che venga visualizzata la home page.</span><span class="sxs-lookup"><span data-stu-id="2679e-155">Run the app to verify that the home page appears.</span></span>

## <a name="the-data-model"></a><span data-ttu-id="2679e-156">Modello di dati</span><span class="sxs-lookup"><span data-stu-id="2679e-156">The data model</span></span>

<span data-ttu-id="2679e-157">Nelle sezioni seguenti viene descritto come creare un modello di dati:</span><span class="sxs-lookup"><span data-stu-id="2679e-157">The following sections create a data model:</span></span>

![Diagramma modello di dati Course/Enrollment/Student (Corso/Iscrizione/Studente)](intro/_static/data-model-diagram.png)

<span data-ttu-id="2679e-159">Uno studente può iscriversi a un numero qualsiasi di corsi e un corso può avere un numero qualsiasi di studenti iscritti.</span><span class="sxs-lookup"><span data-stu-id="2679e-159">A student can enroll in any number of courses, and a course can have any number of students enrolled in it.</span></span>

## <a name="the-student-entity"></a><span data-ttu-id="2679e-160">Entità Student (Studente)</span><span class="sxs-lookup"><span data-stu-id="2679e-160">The Student entity</span></span>

![Diagramma entità Student (Studente)](intro/_static/student-entity.png)

* <span data-ttu-id="2679e-162">Creare una cartella *Models* nella cartella del progetto.</span><span class="sxs-lookup"><span data-stu-id="2679e-162">Create a *Models* folder in the project folder.</span></span> 

* <span data-ttu-id="2679e-163">Creare *Models/Student.cs* con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="2679e-163">Create *Models/Student.cs* with the following code:</span></span>

  [!code-csharp[Main](intro/samples/cu30snapshots/1-intro/Models/Student.cs)]

<span data-ttu-id="2679e-164">La proprietà `ID` diventa la colonna chiave primaria della tabella del database che corrisponde a questa classe.</span><span class="sxs-lookup"><span data-stu-id="2679e-164">The `ID` property becomes the primary key column of the database table that corresponds to this class.</span></span> <span data-ttu-id="2679e-165">Per impostazione predefinita, Entity Framework Core interpreta una proprietà denominata `ID` o `classnameID` come chiave primaria.</span><span class="sxs-lookup"><span data-stu-id="2679e-165">By default, EF Core interprets a property that's named `ID` or `classnameID` as the primary key.</span></span> <span data-ttu-id="2679e-166">Il nome alternativo riconosciuto automaticamente per la chiave primaria della classe `Student` è quindi `StudentID`.</span><span class="sxs-lookup"><span data-stu-id="2679e-166">So the alternative automatically recognized name for the `Student` class primary key is `StudentID`.</span></span> <span data-ttu-id="2679e-167">Per altre informazioni, vedere [chiavi EF Core](/ef/core/modeling/keys?tabs=data-annotations).</span><span class="sxs-lookup"><span data-stu-id="2679e-167">For more information, see [EF Core - Keys](/ef/core/modeling/keys?tabs=data-annotations).</span></span>

<span data-ttu-id="2679e-168">La proprietà `Enrollments` rappresenta una [proprietà di navigazione](/ef/core/modeling/relationships).</span><span class="sxs-lookup"><span data-stu-id="2679e-168">The `Enrollments` property is a [navigation property](/ef/core/modeling/relationships).</span></span> <span data-ttu-id="2679e-169">Le proprietà di navigazione contengono altre entità correlate a questa entità.</span><span class="sxs-lookup"><span data-stu-id="2679e-169">Navigation properties hold other entities that are related to this entity.</span></span> <span data-ttu-id="2679e-170">In questo caso la proprietà `Enrollments` di un'entità `Student` contiene tutte le entità `Enrollment` correlate a tale studente.</span><span class="sxs-lookup"><span data-stu-id="2679e-170">In this case, the `Enrollments` property of a `Student` entity holds all of the `Enrollment` entities that are related to that Student.</span></span> <span data-ttu-id="2679e-171">Ad esempio, se una riga Student nel database presenta due righe Enrollment correlate, la proprietà di navigazione `Enrollments` contiene questi due entità Enrollment.</span><span class="sxs-lookup"><span data-stu-id="2679e-171">For example, if a Student row in the database has two related Enrollment rows, the `Enrollments` navigation property contains those two Enrollment entities.</span></span> 

<span data-ttu-id="2679e-172">Nel database, una riga Enrollment è correlata a una riga Student se la relativa colonna StudentID contiene il valore ID dello studente.</span><span class="sxs-lookup"><span data-stu-id="2679e-172">In the database, an Enrollment row is related to a Student row if its StudentID column contains the student's ID value.</span></span> <span data-ttu-id="2679e-173">Si supponga, ad esempio, che una riga Student abbia ID=1.</span><span class="sxs-lookup"><span data-stu-id="2679e-173">For example, suppose a Student row has ID=1.</span></span> <span data-ttu-id="2679e-174">Le righe Enrollment correlate avranno StudentID = 1.</span><span class="sxs-lookup"><span data-stu-id="2679e-174">Related Enrollment rows will have StudentID = 1.</span></span> <span data-ttu-id="2679e-175">StudentID è una *chiave esterna* nella tabella Enrollment.</span><span class="sxs-lookup"><span data-stu-id="2679e-175">StudentID is a *foreign key* in the Enrollment table.</span></span> 

<span data-ttu-id="2679e-176">La proprietà `Enrollments` è definita come `ICollection<Enrollment>` perché potrebbero essere presenti più entità Enrollment correlate.</span><span class="sxs-lookup"><span data-stu-id="2679e-176">The `Enrollments` property is defined as `ICollection<Enrollment>` because there may be multiple related Enrollment entities.</span></span> <span data-ttu-id="2679e-177">È possibile usare altri tipi di raccolta, ad esempio `List<Enrollment>` o `HashSet<Enrollment>`.</span><span class="sxs-lookup"><span data-stu-id="2679e-177">You can use other collection types, such as `List<Enrollment>` or `HashSet<Enrollment>`.</span></span> <span data-ttu-id="2679e-178">Se si specifica `ICollection<Enrollment>`, per impostazione predefinita Entity Framework Core crea una raccolta `HashSet<Enrollment>`.</span><span class="sxs-lookup"><span data-stu-id="2679e-178">When `ICollection<Enrollment>` is used, EF Core creates a `HashSet<Enrollment>` collection by default.</span></span>

## <a name="the-enrollment-entity"></a><span data-ttu-id="2679e-179">Entità Enrollment (Iscrizione)</span><span class="sxs-lookup"><span data-stu-id="2679e-179">The Enrollment entity</span></span>

![Diagramma entità Enrollment (Iscrizione)](intro/_static/enrollment-entity.png)

<span data-ttu-id="2679e-181">Creare *Models/Enrollment.cs* con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="2679e-181">Create *Models/Enrollment.cs* with the following code:</span></span>

[!code-csharp[Main](intro/samples/cu30snapshots/1-intro/Models/Enrollment.cs)]

<span data-ttu-id="2679e-182">La proprietà `EnrollmentID` è la chiave primaria. Questa entità usa il modello `classnameID` anziché `ID` da solo.</span><span class="sxs-lookup"><span data-stu-id="2679e-182">The `EnrollmentID` property is the primary key; this entity uses the `classnameID` pattern instead of `ID` by itself.</span></span> <span data-ttu-id="2679e-183">Per un modello di dati di produzione, scegliere un modello e usarlo in modo coerente.</span><span class="sxs-lookup"><span data-stu-id="2679e-183">For a production data model, choose one pattern and use it consistently.</span></span> <span data-ttu-id="2679e-184">Questa esercitazione usa entrambi solo per illustrare che entrambi funzionano.</span><span class="sxs-lookup"><span data-stu-id="2679e-184">This tutorial uses both just to illustrate that both work.</span></span> <span data-ttu-id="2679e-185">L'uso di `ID` senza `classname` rende più semplice l'implementazione di alcuni tipi di modifiche al modello di dati.</span><span class="sxs-lookup"><span data-stu-id="2679e-185">Using `ID` without `classname` makes it easier to implement some kinds of data model changes.</span></span>

<span data-ttu-id="2679e-186">La proprietà `Grade` è un oggetto`enum`.</span><span class="sxs-lookup"><span data-stu-id="2679e-186">The `Grade` property is an `enum`.</span></span> <span data-ttu-id="2679e-187">Il punto interrogativo dopo la dichiarazione del tipo `Grade` indica che la proprietà `Grade`[ammette i valori Null](/dotnet/csharp/programming-guide/nullable-types/).</span><span class="sxs-lookup"><span data-stu-id="2679e-187">The question mark after the `Grade` type declaration indicates that the `Grade` property is [nullable](/dotnet/csharp/programming-guide/nullable-types/).</span></span> <span data-ttu-id="2679e-188">Un voto con valore Null è diverso da un voto con valore zero. Null significa che un voto è sconosciuto o non è stato ancora assegnato.</span><span class="sxs-lookup"><span data-stu-id="2679e-188">A grade that's null is different from a zero grade&mdash;null means a grade isn't known or hasn't been assigned yet.</span></span>

<span data-ttu-id="2679e-189">La proprietà `StudentID` è una chiave esterna e la proprietà di navigazione corrispondente è `Student`.</span><span class="sxs-lookup"><span data-stu-id="2679e-189">The `StudentID` property is a foreign key, and the corresponding navigation property is `Student`.</span></span> <span data-ttu-id="2679e-190">Un'entità `Enrollment` è associata a un'entità `Student`, pertanto la proprietà contiene un'unica entità `Student`.</span><span class="sxs-lookup"><span data-stu-id="2679e-190">An `Enrollment` entity is associated with one `Student` entity, so the property contains a single `Student` entity.</span></span>

<span data-ttu-id="2679e-191">La proprietà `CourseID` è una chiave esterna e la proprietà di navigazione corrispondente è `Course`.</span><span class="sxs-lookup"><span data-stu-id="2679e-191">The `CourseID` property is a foreign key, and the corresponding navigation property is `Course`.</span></span> <span data-ttu-id="2679e-192">Un'entità `Enrollment` è associata a un'entità `Course`.</span><span class="sxs-lookup"><span data-stu-id="2679e-192">An `Enrollment` entity is associated with one `Course` entity.</span></span>

<span data-ttu-id="2679e-193">Entity Framework Core interpreta una proprietà come chiave esterna se è denominata `<navigation property name><primary key property name>`.</span><span class="sxs-lookup"><span data-stu-id="2679e-193">EF Core interprets a property as a foreign key if it's named `<navigation property name><primary key property name>`.</span></span> <span data-ttu-id="2679e-194">Ad esempio, `StudentID` è la chiave esterna per la proprietà di navigazione `Student`, dato che la chiave primaria `Student` dell'entità è `ID`.</span><span class="sxs-lookup"><span data-stu-id="2679e-194">For example,`StudentID` is the foreign key for the `Student` navigation property, since the `Student` entity's primary key is `ID`.</span></span> <span data-ttu-id="2679e-195">Le proprietà di chiave esterna possono anche essere denominate `<primary key property name>`.</span><span class="sxs-lookup"><span data-stu-id="2679e-195">Foreign key properties can also be named `<primary key property name>`.</span></span> <span data-ttu-id="2679e-196">Ad esempio, `CourseID` poiché la chiave primaria `Course` dell'entità è `CourseID`.</span><span class="sxs-lookup"><span data-stu-id="2679e-196">For example, `CourseID` since the `Course` entity's primary key is `CourseID`.</span></span>

## <a name="the-course-entity"></a><span data-ttu-id="2679e-197">Entità Course (Corso)</span><span class="sxs-lookup"><span data-stu-id="2679e-197">The Course entity</span></span>

![Diagramma entità Course (Corso)](intro/_static/course-entity.png)

<span data-ttu-id="2679e-199">Creare *Models/Course.cs* con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="2679e-199">Create *Models/Course.cs* with the following code:</span></span>

[!code-csharp[Main](intro/samples/cu30snapshots/1-intro/Models/Course.cs)]

<span data-ttu-id="2679e-200">La proprietà `Enrollments` rappresenta una proprietà di navigazione.</span><span class="sxs-lookup"><span data-stu-id="2679e-200">The `Enrollments` property is a navigation property.</span></span> <span data-ttu-id="2679e-201">È possibile correlare un'entità `Course` a un numero qualsiasi di entità `Enrollment`.</span><span class="sxs-lookup"><span data-stu-id="2679e-201">A `Course` entity can be related to any number of `Enrollment` entities.</span></span>

<span data-ttu-id="2679e-202">L'attributo `DatabaseGenerated` consente all'app di specificare la chiave primaria anziché chiedere al database di generarla.</span><span class="sxs-lookup"><span data-stu-id="2679e-202">The `DatabaseGenerated` attribute allows the app to specify the primary key rather than having the database generate it.</span></span>

<span data-ttu-id="2679e-203">Compilare il progetto per verificare che non siano presenti errori del compilatore.</span><span class="sxs-lookup"><span data-stu-id="2679e-203">Build the project to validate that there are no compiler errors.</span></span>

## <a name="scaffold-student-pages"></a><span data-ttu-id="2679e-204">Scaffolding delle pagine Student</span><span class="sxs-lookup"><span data-stu-id="2679e-204">Scaffold Student pages</span></span>

<span data-ttu-id="2679e-205">In questa sezione si userà lo strumento di scaffolding di ASP.NET Core per generare:</span><span class="sxs-lookup"><span data-stu-id="2679e-205">In this section, you use the ASP.NET Core scaffolding tool to generate:</span></span>

* <span data-ttu-id="2679e-206">Classe EF Core `DbContext` .</span><span class="sxs-lookup"><span data-stu-id="2679e-206">An EF Core `DbContext` class.</span></span> <span data-ttu-id="2679e-207">Il contesto è la classe principale che coordina le funzionalità di Entity Framework per un determinato modello di dati.</span><span class="sxs-lookup"><span data-stu-id="2679e-207">The context is the main class that coordinates Entity Framework functionality for a given data model.</span></span> <span data-ttu-id="2679e-208">Deriva dalla classe <xref:Microsoft.EntityFrameworkCore.DbContext?displayProperty=fullName>.</span><span class="sxs-lookup"><span data-stu-id="2679e-208">It derives from the <xref:Microsoft.EntityFrameworkCore.DbContext?displayProperty=fullName> class.</span></span>
* <span data-ttu-id="2679e-209">Razor pagine che gestiscono le operazioni di creazione, lettura, aggiornamento ed eliminazione (CRUD) per l' `Student` entità.</span><span class="sxs-lookup"><span data-stu-id="2679e-209">Razor pages that handle Create, Read, Update, and Delete (CRUD) operations for the `Student` entity.</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="2679e-210">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="2679e-210">Visual Studio</span></span>](#tab/visual-studio)

* <span data-ttu-id="2679e-211">Creare una cartella *Pages/Students*.</span><span class="sxs-lookup"><span data-stu-id="2679e-211">Create a *Pages/Students* folder.</span></span>
* <span data-ttu-id="2679e-212">In **Esplora soluzioni** fare clic con il pulsante destro del mouse sulla cartella *Pages/Students* e quindi scegliere **Aggiungi** > **Nuovo elemento di scaffolding**.</span><span class="sxs-lookup"><span data-stu-id="2679e-212">In **Solution Explorer**, right-click the *Pages/Students* folder and select **Add** > **New Scaffolded Item**.</span></span>
* <span data-ttu-id="2679e-213">Nella finestra di dialogo **Aggiungi nuovo elemento di impalcatura** :</span><span class="sxs-lookup"><span data-stu-id="2679e-213">In the **Add New Scaffold Item** dialog:</span></span>
  * <span data-ttu-id="2679e-214">Nella scheda a sinistra selezionare **installato > Razor pagine comuni >**</span><span class="sxs-lookup"><span data-stu-id="2679e-214">In the left tab, select **Installed > Common > Razor Pages**</span></span>
  * <span data-ttu-id="2679e-215">Selezionare **Razor pages using Entity Framework (CRUD)** > **Add**.</span><span class="sxs-lookup"><span data-stu-id="2679e-215">Select **Razor Pages using Entity Framework (CRUD)** > **ADD**.</span></span>
* <span data-ttu-id="2679e-216">Nella finestra di dialogo **Aggiungi Razor pagine con Entity Framework (CRUD)** :</span><span class="sxs-lookup"><span data-stu-id="2679e-216">In the **Add Razor Pages using Entity Framework (CRUD)** dialog:</span></span>
  * <span data-ttu-id="2679e-217">Nell'elenco a discesa **Classe modello** selezionare **Student (ContosoUniversity.Models)**.</span><span class="sxs-lookup"><span data-stu-id="2679e-217">In the **Model class** drop-down, select **Student (ContosoUniversity.Models)**.</span></span>
  * <span data-ttu-id="2679e-218">Nella riga **Classe contesto di dati** selezionare il segno più **+**.</span><span class="sxs-lookup"><span data-stu-id="2679e-218">In the **Data context class** row, select the **+** (plus) sign.</span></span>
    * <span data-ttu-id="2679e-219">Modificare il nome del contesto dati in modo `SchoolContext` che termini anziché `ContosoUniversityContext` .</span><span class="sxs-lookup"><span data-stu-id="2679e-219">Change the data context name to end in `SchoolContext` rather than `ContosoUniversityContext`.</span></span> <span data-ttu-id="2679e-220">Nome del contesto aggiornato: `ContosoUniversity.Data.SchoolContext`</span><span class="sxs-lookup"><span data-stu-id="2679e-220">The updated context name: `ContosoUniversity.Data.SchoolContext`</span></span>
    * <span data-ttu-id="2679e-221">Selezionare **Aggiungi** per completare l'aggiunta della classe del contesto dati.</span><span class="sxs-lookup"><span data-stu-id="2679e-221">Select **Add** to finish adding the data context class.</span></span>
   * <span data-ttu-id="2679e-222">Selezionare **Aggiungi** per completare la finestra di dialogo **Aggiungi Razor pagine** .</span><span class="sxs-lookup"><span data-stu-id="2679e-222">Select **Add** to finish the **Add Razor Pages** dialog.</span></span>

<span data-ttu-id="2679e-223">Vengono installati automaticamente i pacchetti seguenti:</span><span class="sxs-lookup"><span data-stu-id="2679e-223">The following packages are automatically installed:</span></span>

* `Microsoft.EntityFrameworkCore.SqlServer`
* `Microsoft.EntityFrameworkCore.Tools`
* `Microsoft.VisualStudio.Web.CodeGeneration.Design`

# <a name="visual-studio-code"></a>[<span data-ttu-id="2679e-224">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="2679e-224">Visual Studio Code</span></span>](#tab/visual-studio-code)

* <span data-ttu-id="2679e-225">Eseguire i comandi seguenti dell'interfaccia della riga di comando di .NET Core per installare i pacchetti NuGet necessari:</span><span class="sxs-lookup"><span data-stu-id="2679e-225">Run the following .NET Core CLI commands to install required NuGet packages:</span></span>

  ```dotnetcli
  dotnet add package Microsoft.EntityFrameworkCore.SQLite -v 5.0.0-*
  dotnet add package Microsoft.EntityFrameworkCore.SqlServer -v 5.0.0-*
  dotnet add package Microsoft.EntityFrameworkCore.Design -v 5.0.0-*
  dotnet add package Microsoft.EntityFrameworkCore.Tools -v 5.0.0-*
  dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design -v 5.0.0-*
  dotnet add package Microsoft.AspNetCore.Diagnostics.EntityFrameworkCore -v 5.0.0-*  
  ```

   <span data-ttu-id="2679e-226">Per lo scaffolding è necessario il pacchetto Microsoft.VisualStudio.Web.CodeGeneration.Design.</span><span class="sxs-lookup"><span data-stu-id="2679e-226">The Microsoft.VisualStudio.Web.CodeGeneration.Design package is required for scaffolding.</span></span> <span data-ttu-id="2679e-227">Sebbene l'app non usi SQL Server, lo strumento di scaffolding richiede il pacchetto di SQL Server.</span><span class="sxs-lookup"><span data-stu-id="2679e-227">Although the app won't use SQL Server, the scaffolding tool needs the SQL Server package.</span></span>

* <span data-ttu-id="2679e-228">Creare una cartella *Pages/Students*.</span><span class="sxs-lookup"><span data-stu-id="2679e-228">Create a *Pages/Students* folder.</span></span>

* <span data-ttu-id="2679e-229">Eseguire il comando seguente per installare lo [strumento di scaffolding aspnet-codegenerator](xref:fundamentals/tools/dotnet-aspnet-codegenerator).</span><span class="sxs-lookup"><span data-stu-id="2679e-229">Run the following command to install the [aspnet-codegenerator scaffolding tool](xref:fundamentals/tools/dotnet-aspnet-codegenerator).</span></span>

  ```dotnetcli
  dotnet tool uninstall --global dotnet-aspnet-codegenerator
  dotnet tool install --global dotnet-aspnet-codegenerator --version 5.0.0-*  
  ```

* <span data-ttu-id="2679e-230">Eseguire il comando seguente per eseguire lo scaffolding delle pagine Student.</span><span class="sxs-lookup"><span data-stu-id="2679e-230">Run the following command to scaffold Student pages.</span></span>

  <span data-ttu-id="2679e-231">**In Windows**</span><span class="sxs-lookup"><span data-stu-id="2679e-231">**On Windows**</span></span>

  ```dotnetcli
  dotnet aspnet-codegenerator razorpage -m Student -dc ContosoUniversity.Data.SchoolContext -udl -outDir Pages\Students --referenceScriptLibraries -sqlite  
  ```

  <span data-ttu-id="2679e-232">**In macOS o Linux**</span><span class="sxs-lookup"><span data-stu-id="2679e-232">**On macOS or Linux**</span></span>

  ```dotnetcli
  dotnet aspnet-codegenerator razorpage -m Student -dc ContosoUniversity.Data.SchoolContext -udl -outDir Pages/Students --referenceScriptLibraries -sqlite  
  ```

---

<span data-ttu-id="2679e-233">Se il passaggio precedente ha esito negativo, compilare il progetto e riprovare l'operazione di impalcatura.</span><span class="sxs-lookup"><span data-stu-id="2679e-233">If the preceding step fails, build the project and retry the scaffold step.</span></span>

<span data-ttu-id="2679e-234">Il processo di scaffolding:</span><span class="sxs-lookup"><span data-stu-id="2679e-234">The scaffolding process:</span></span>

* <span data-ttu-id="2679e-235">Crea Razor pagine nella cartella *pages/students* :</span><span class="sxs-lookup"><span data-stu-id="2679e-235">Creates Razor pages in the *Pages/Students* folder:</span></span>
  * <span data-ttu-id="2679e-236">*Create.cshtml* e *Create.cshtml.cs*</span><span class="sxs-lookup"><span data-stu-id="2679e-236">*Create.cshtml* and *Create.cshtml.cs*</span></span>
  * <span data-ttu-id="2679e-237">*Delete.cshtml* e *Delete.cshtml.cs*</span><span class="sxs-lookup"><span data-stu-id="2679e-237">*Delete.cshtml* and *Delete.cshtml.cs*</span></span>
  * <span data-ttu-id="2679e-238">*Details.cshtml* e *Details.cshtml.cs*</span><span class="sxs-lookup"><span data-stu-id="2679e-238">*Details.cshtml* and *Details.cshtml.cs*</span></span>
  * <span data-ttu-id="2679e-239">*Edit.cshtml* e *Edit.cshtml.cs*</span><span class="sxs-lookup"><span data-stu-id="2679e-239">*Edit.cshtml* and *Edit.cshtml.cs*</span></span>
  * <span data-ttu-id="2679e-240">*Index.cshtml* e *Index.cshtml.cs*</span><span class="sxs-lookup"><span data-stu-id="2679e-240">*Index.cshtml* and *Index.cshtml.cs*</span></span>
* <span data-ttu-id="2679e-241">Crea *Data/SchoolContext.cs*.</span><span class="sxs-lookup"><span data-stu-id="2679e-241">Creates *Data/SchoolContext.cs*.</span></span>
* <span data-ttu-id="2679e-242">Aggiunge il contesto all'inserimento delle dipendenze in *Startup.cs*.</span><span class="sxs-lookup"><span data-stu-id="2679e-242">Adds the context to dependency injection in *Startup.cs*.</span></span>
* <span data-ttu-id="2679e-243">Aggiunge una stringa di connessione al database a *appsettings.json* .</span><span class="sxs-lookup"><span data-stu-id="2679e-243">Adds a database connection string to *appsettings.json*.</span></span>

## <a name="database-connection-string"></a><span data-ttu-id="2679e-244">Stringa di connessione al database</span><span class="sxs-lookup"><span data-stu-id="2679e-244">Database connection string</span></span>

<span data-ttu-id="2679e-245">Lo strumento di impalcatura genera una stringa di connessione nel *appsettings.json* file.</span><span class="sxs-lookup"><span data-stu-id="2679e-245">The scaffolding tool generates a connection string in the *appsettings.json* file.</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="2679e-246">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="2679e-246">Visual Studio</span></span>](#tab/visual-studio)

<span data-ttu-id="2679e-247">La stringa di connessione specifica [SQL Server database locale](/sql/database-engine/configure-windows/sql-server-2016-express-localdb):</span><span class="sxs-lookup"><span data-stu-id="2679e-247">The connection string specifies [SQL Server LocalDB](/sql/database-engine/configure-windows/sql-server-2016-express-localdb):</span></span>

[!code-json[Main](intro/samples/cu50/appsettings.json?highlight=11)]

<span data-ttu-id="2679e-248">Local DB è una versione leggera del motore di database di SQL Server Express appositamente pensato per lo sviluppo di app e non per la produzione.</span><span class="sxs-lookup"><span data-stu-id="2679e-248">LocalDB is a lightweight version of the SQL Server Express Database Engine and is intended for app development, not production use.</span></span> <span data-ttu-id="2679e-249">Per impostazione predefinita, Local DB crea file con estensione *mdf* nella directory `C:/Users/<user>`.</span><span class="sxs-lookup"><span data-stu-id="2679e-249">By default, LocalDB creates *.mdf* files in the `C:/Users/<user>` directory.</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="2679e-250">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="2679e-250">Visual Studio Code</span></span>](#tab/visual-studio-code)

<span data-ttu-id="2679e-251">Abbreviare la stringa di connessione SQLite a *cu. DB*:</span><span class="sxs-lookup"><span data-stu-id="2679e-251">Shorten the SQLite connection string to *CU.db*:</span></span>

[!code-json[Main](intro/samples/cu50/appsettingsSQLite.json?highlight=11)]

---

## <a name="update-the-database-context-class"></a><span data-ttu-id="2679e-252">Aggiornare la classe del contesto di database</span><span class="sxs-lookup"><span data-stu-id="2679e-252">Update the database context class</span></span>

<span data-ttu-id="2679e-253">La classe principale che coordina le funzionalità di EF Core per un determinato modello di dati è la classe del contesto di database.</span><span class="sxs-lookup"><span data-stu-id="2679e-253">The main class that coordinates EF Core functionality for a given data model is the database context class.</span></span> <span data-ttu-id="2679e-254">Il contesto è derivato da [Microsoft.EntityFrameworkCore.DbContext](/dotnet/api/microsoft.entityframeworkcore.dbcontext).</span><span class="sxs-lookup"><span data-stu-id="2679e-254">The context is derived from [Microsoft.EntityFrameworkCore.DbContext](/dotnet/api/microsoft.entityframeworkcore.dbcontext).</span></span> <span data-ttu-id="2679e-255">Il contesto specifica le entità incluse nel modello di dati.</span><span class="sxs-lookup"><span data-stu-id="2679e-255">The context specifies which entities are included in the data model.</span></span> <span data-ttu-id="2679e-256">In questo progetto la classe è denominata `SchoolContext`.</span><span class="sxs-lookup"><span data-stu-id="2679e-256">In this project, the class is named `SchoolContext`.</span></span>

<span data-ttu-id="2679e-257">Aggiornare *Data/SchoolContext.cs* con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="2679e-257">Update *Data/SchoolContext.cs* with the following code:</span></span>

[!code-csharp[Main](intro/samples/cu30snapshots/1-intro/Data/SchoolContext.cs?highlight=13-22)]

<span data-ttu-id="2679e-258">Il codice precedente passa da singolare `DbSet<Student> Student` a plurale `DbSet<Student> Students` .</span><span class="sxs-lookup"><span data-stu-id="2679e-258">The preceding code changes from the singular `DbSet<Student> Student` to the  plural `DbSet<Student> Students`.</span></span> <span data-ttu-id="2679e-259">Per fare in Razor modo che il codice delle pagine corrisponda al nuovo `DBSet` nome, apportare una modifica globale da: `_context.Student.`</span><span class="sxs-lookup"><span data-stu-id="2679e-259">To make the Razor Pages code match the new `DBSet` name, make a global change from: `_context.Student.`</span></span>
<span data-ttu-id="2679e-260">A: `_context.Students.`</span><span class="sxs-lookup"><span data-stu-id="2679e-260">to: `_context.Students.`</span></span>

<span data-ttu-id="2679e-261">Sono presenti 8 occorrenze.</span><span class="sxs-lookup"><span data-stu-id="2679e-261">There are 8 occurrences.</span></span>

<span data-ttu-id="2679e-262">Poiché un set di entità contiene più entità, molti sviluppatori preferiscono che i `DBSet` nomi delle proprietà siano plurali.</span><span class="sxs-lookup"><span data-stu-id="2679e-262">Because an entity set contains multiple entities, many developers prefer the `DBSet` property names should be plural.</span></span>

<span data-ttu-id="2679e-263">Il codice evidenziato:</span><span class="sxs-lookup"><span data-stu-id="2679e-263">The highlighted code:</span></span>

* <span data-ttu-id="2679e-264">Crea una [proprietà \<TEntity> DbSet](/dotnet/api/microsoft.entityframeworkcore.dbset-1) per ogni set di entità.</span><span class="sxs-lookup"><span data-stu-id="2679e-264">Creates a [DbSet\<TEntity>](/dotnet/api/microsoft.entityframeworkcore.dbset-1) property for each entity set.</span></span> <span data-ttu-id="2679e-265">Nella terminologia di Entity Framework Core:</span><span class="sxs-lookup"><span data-stu-id="2679e-265">In EF Core terminology:</span></span>
  * <span data-ttu-id="2679e-266">Un set di entità corrisponde in genere alla tabella di un database.</span><span class="sxs-lookup"><span data-stu-id="2679e-266">An entity set typically corresponds to a database table.</span></span>
  * <span data-ttu-id="2679e-267">Un'entità corrisponde a una riga nella tabella.</span><span class="sxs-lookup"><span data-stu-id="2679e-267">An entity corresponds to a row in the table.</span></span>
* <span data-ttu-id="2679e-268">Chiama <xref:Microsoft.EntityFrameworkCore.DbContext.OnModelCreating%2A>.</span><span class="sxs-lookup"><span data-stu-id="2679e-268">Calls <xref:Microsoft.EntityFrameworkCore.DbContext.OnModelCreating%2A>.</span></span> <span data-ttu-id="2679e-269">`OnModelCreating`:</span><span class="sxs-lookup"><span data-stu-id="2679e-269">`OnModelCreating`:</span></span>
  * <span data-ttu-id="2679e-270">Viene chiamato quando `SchoolContext` è stato inizializzato, ma prima che il modello sia stato bloccato e utilizzato per inizializzare il contesto.</span><span class="sxs-lookup"><span data-stu-id="2679e-270">Is called when `SchoolContext` has been initialized, but before the model has been locked down and used to initialize the context.</span></span>
  * <span data-ttu-id="2679e-271">È obbligatorio perché, più avanti nell'esercitazione `Student` , l'entità avrà riferimenti alle altre entità.</span><span class="sxs-lookup"><span data-stu-id="2679e-271">Is required because later in the tutorial The `Student` entity will have references to the other entities.</span></span>
  <!-- Review, OnModelCreating needs review -->

<span data-ttu-id="2679e-272">Compilare il progetto per verificare che non siano presenti errori di compilazione.</span><span class="sxs-lookup"><span data-stu-id="2679e-272">Build the project to verify there are no compiler errors.</span></span>

## <a name="startupcs"></a><span data-ttu-id="2679e-273">Startup.cs</span><span class="sxs-lookup"><span data-stu-id="2679e-273">Startup.cs</span></span>

<span data-ttu-id="2679e-274">ASP.NET Core viene compilato tramite [dependency injection](xref:fundamentals/dependency-injection).</span><span class="sxs-lookup"><span data-stu-id="2679e-274">ASP.NET Core is built with [dependency injection](xref:fundamentals/dependency-injection).</span></span> <span data-ttu-id="2679e-275">I servizi, ad esempio, `SchoolContext` vengono registrati con l'inserimento delle dipendenze durante l'avvio dell'app.</span><span class="sxs-lookup"><span data-stu-id="2679e-275">Services such as the `SchoolContext` are registered with dependency injection during app startup.</span></span> <span data-ttu-id="2679e-276">I componenti che richiedono questi servizi, ad esempio Razor le pagine, vengono forniti tramite i parametri del costruttore.</span><span class="sxs-lookup"><span data-stu-id="2679e-276">Components that require these services, such as Razor Pages, are provided these services via constructor parameters.</span></span> <span data-ttu-id="2679e-277">Più avanti nell'esercitazione viene illustrato il codice del costruttore che ottiene un'istanza del contesto di database.</span><span class="sxs-lookup"><span data-stu-id="2679e-277">The constructor code that gets a database context instance is shown later in the tutorial.</span></span>

<span data-ttu-id="2679e-278">Lo strumento di scaffolding ha registrato automaticamente la classe di contesto nel contenitore di inserimento delle dipendenze.</span><span class="sxs-lookup"><span data-stu-id="2679e-278">The scaffolding tool automatically registered the context class with the dependency injection container.</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="2679e-279">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="2679e-279">Visual Studio</span></span>](#tab/visual-studio)

<span data-ttu-id="2679e-280">Le seguenti righe evidenziate sono state aggiunte dall'impalcatura:</span><span class="sxs-lookup"><span data-stu-id="2679e-280">The following highlighted lines were added by the scaffolder:</span></span>

[!code-csharp[Main](intro/samples/cu30/Startup.cs?name=snippet_ConfigureServices&highlight=5-6)]

# <a name="visual-studio-code"></a>[<span data-ttu-id="2679e-281">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="2679e-281">Visual Studio Code</span></span>](#tab/visual-studio-code)

<span data-ttu-id="2679e-282">Verificare il codice aggiunto dall'impalcatura chiama `UseSqlite` .</span><span class="sxs-lookup"><span data-stu-id="2679e-282">Verify the code added by the scaffolder calls `UseSqlite`.</span></span>

[!code-csharp[Main](intro/samples/cu30/StartupSQLite.cs?name=snippet_ConfigureServices&highlight=5-6)]

<span data-ttu-id="2679e-283">Per informazioni sull'uso di un database di produzione [, vedere usare SQLite per lo sviluppo, SQL Server per la produzione](xref:tutorials/razor-pages/model#use-sqlite-for-development-sql-server-for-production) .</span><span class="sxs-lookup"><span data-stu-id="2679e-283">See [Use SQLite for development, SQL Server for production](xref:tutorials/razor-pages/model#use-sqlite-for-development-sql-server-for-production) for information on using a production database.</span></span>

---

<span data-ttu-id="2679e-284">Il nome della stringa di connessione viene passato al contesto chiamando un metodo in un oggetto [DbContextOptions](/dotnet/api/microsoft.entityframeworkcore.dbcontextoptions).</span><span class="sxs-lookup"><span data-stu-id="2679e-284">The name of the connection string is passed in to the context by calling a method on a [DbContextOptions](/dotnet/api/microsoft.entityframeworkcore.dbcontextoptions) object.</span></span> <span data-ttu-id="2679e-285">Per lo sviluppo locale, il [sistema di configurazione ASP.NET Core](xref:fundamentals/configuration/index) legge la stringa di connessione dal *appsettings.json* file.</span><span class="sxs-lookup"><span data-stu-id="2679e-285">For local development, the [ASP.NET Core configuration system](xref:fundamentals/configuration/index) reads the connection string from the *appsettings.json* file.</span></span>

### <a name="add-the-database-exception-filter"></a><span data-ttu-id="2679e-286">Aggiungere il filtro eccezioni del database</span><span class="sxs-lookup"><span data-stu-id="2679e-286">Add the database exception filter</span></span>

<span data-ttu-id="2679e-287">Aggiungere <xref:Microsoft.Extensions.DependencyInjection.DatabaseDeveloperPageExceptionFilterServiceExtensions.AddDatabaseDeveloperPageExceptionFilter%2A> a `ConfigureServices` come illustrato nel codice seguente:</span><span class="sxs-lookup"><span data-stu-id="2679e-287">Add <xref:Microsoft.Extensions.DependencyInjection.DatabaseDeveloperPageExceptionFilterServiceExtensions.AddDatabaseDeveloperPageExceptionFilter%2A> to `ConfigureServices` as shown in the following code:</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="2679e-288">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="2679e-288">Visual Studio</span></span>](#tab/visual-studio)

[!code-csharp[Main](intro/samples/cu50/Startup.cs?name=snippet_ConfigureServices&highlight=8)]

<span data-ttu-id="2679e-289">Aggiungere il pacchetto NuGet [Microsoft. AspNetCore. Diagnostics. EntityFrameworkCore](https://www.nuget.org/packages/Microsoft.AspNetCore.Diagnostics.EntityFrameworkCore) .</span><span class="sxs-lookup"><span data-stu-id="2679e-289">Add the [Microsoft.AspNetCore.Diagnostics.EntityFrameworkCore](https://www.nuget.org/packages/Microsoft.AspNetCore.Diagnostics.EntityFrameworkCore) NuGet package.</span></span>

<span data-ttu-id="2679e-290">In PMC immettere quanto segue per aggiungere il pacchetto NuGet:</span><span class="sxs-lookup"><span data-stu-id="2679e-290">In the PMC, enter the following to add the NuGet package:</span></span>

```powershell
Install-Package Microsoft.AspNetCore.Diagnostics.EntityFrameworkCore
```

# <a name="visual-studio-code"></a>[<span data-ttu-id="2679e-291">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="2679e-291">Visual Studio Code</span></span>](#tab/visual-studio-code)

[!code-csharp[Main](intro/samples/cu50/StartupSQLite.cs?name=snippet_ConfigureServices&highlight=8)]

---

<span data-ttu-id="2679e-292">Il `Microsoft.AspNetCore.Diagnostics.EntityFrameworkCore` pacchetto NuGet fornisce ASP.NET Core middleware per Entity Framework Core pagine di errore.</span><span class="sxs-lookup"><span data-stu-id="2679e-292">The `Microsoft.AspNetCore.Diagnostics.EntityFrameworkCore` NuGet package provides ASP.NET Core middleware for Entity Framework Core error pages.</span></span> <span data-ttu-id="2679e-293">Questo middleware consente di rilevare e diagnosticare gli errori con Entity Framework Core migrazioni.</span><span class="sxs-lookup"><span data-stu-id="2679e-293">This middleware helps to detect and diagnose errors with Entity Framework Core migrations.</span></span>

<span data-ttu-id="2679e-294">`AddDatabaseDeveloperPageExceptionFilter`Fornisce informazioni utili sull'errore nell' [ambiente di sviluppo](xref:fundamentals/environments).</span><span class="sxs-lookup"><span data-stu-id="2679e-294">The `AddDatabaseDeveloperPageExceptionFilter` provides helpful error information in the [development environment](xref:fundamentals/environments).</span></span>

## <a name="create-the-database"></a><span data-ttu-id="2679e-295">Creare il database</span><span class="sxs-lookup"><span data-stu-id="2679e-295">Create the database</span></span>

<span data-ttu-id="2679e-296">Aggiornare *Program.cs* per creare il database se non esiste:</span><span class="sxs-lookup"><span data-stu-id="2679e-296">Update *Program.cs* to create the database if it doesn't exist:</span></span>

[!code-csharp[Main](intro/samples/cu30snapshots/1-intro/Program.cs?highlight=1-2,14-18,21-38)]

<span data-ttu-id="2679e-297">Il metodo [EnsureCreated](/dotnet/api/microsoft.entityframeworkcore.infrastructure.databasefacade.ensurecreated#Microsoft_EntityFrameworkCore_Infrastructure_DatabaseFacade_EnsureCreated) non esegue alcuna azione se esiste un database per il contesto.</span><span class="sxs-lookup"><span data-stu-id="2679e-297">The [EnsureCreated](/dotnet/api/microsoft.entityframeworkcore.infrastructure.databasefacade.ensurecreated#Microsoft_EntityFrameworkCore_Infrastructure_DatabaseFacade_EnsureCreated) method takes no action if a database for the context exists.</span></span> <span data-ttu-id="2679e-298">Se non esiste alcun database, vengono creati il database e lo schema.</span><span class="sxs-lookup"><span data-stu-id="2679e-298">If no database exists, it creates the database and schema.</span></span> <span data-ttu-id="2679e-299">`EnsureCreated` abilita il flusso di lavoro seguente per la gestione delle modifiche al modello di dati:</span><span class="sxs-lookup"><span data-stu-id="2679e-299">`EnsureCreated` enables the following workflow for handling data model changes:</span></span>

* <span data-ttu-id="2679e-300">Eliminare il database.</span><span class="sxs-lookup"><span data-stu-id="2679e-300">Delete the database.</span></span> <span data-ttu-id="2679e-301">Tutti i dati esistenti andranno perduti.</span><span class="sxs-lookup"><span data-stu-id="2679e-301">Any existing data is lost.</span></span>
* <span data-ttu-id="2679e-302">Modificare il modello di dati.</span><span class="sxs-lookup"><span data-stu-id="2679e-302">Change the data model.</span></span> <span data-ttu-id="2679e-303">Ad esempio, aggiungere un campo `EmailAddress`.</span><span class="sxs-lookup"><span data-stu-id="2679e-303">For example, add an `EmailAddress` field.</span></span>
* <span data-ttu-id="2679e-304">Eseguire l'app.</span><span class="sxs-lookup"><span data-stu-id="2679e-304">Run the app.</span></span>
* <span data-ttu-id="2679e-305">`EnsureCreated` crea un database con il nuovo schema.</span><span class="sxs-lookup"><span data-stu-id="2679e-305">`EnsureCreated` creates a database with the new schema.</span></span>

<span data-ttu-id="2679e-306">Questo flusso di lavoro funziona correttamente nelle fasi iniziali dello sviluppo quando lo schema è in rapida evoluzione, purché non sia necessario conservare i dati.</span><span class="sxs-lookup"><span data-stu-id="2679e-306">This workflow works well early in development when the schema is rapidly evolving, as long as you don't need to preserve data.</span></span> <span data-ttu-id="2679e-307">La situazione è diversa quando è necessario mantenere i dati immessi nel database.</span><span class="sxs-lookup"><span data-stu-id="2679e-307">The situation is different when data that has been entered into the database needs to be preserved.</span></span> <span data-ttu-id="2679e-308">In tal caso, usare le migrazioni.</span><span class="sxs-lookup"><span data-stu-id="2679e-308">When that is the case, use migrations.</span></span>

<span data-ttu-id="2679e-309">Più avanti nella serie di esercitazioni si vedrà come eliminare il database creato da `EnsureCreated` e usare invece le migrazioni.</span><span class="sxs-lookup"><span data-stu-id="2679e-309">Later in the tutorial series, you delete the database that was created by `EnsureCreated` and use migrations instead.</span></span> <span data-ttu-id="2679e-310">Non è possibile aggiornare un database creato da `EnsureCreated` usando le migrazioni.</span><span class="sxs-lookup"><span data-stu-id="2679e-310">A database that is created by `EnsureCreated` can't be updated by using migrations.</span></span>

### <a name="test-the-app"></a><span data-ttu-id="2679e-311">Testare l'app</span><span class="sxs-lookup"><span data-stu-id="2679e-311">Test the app</span></span>

* <span data-ttu-id="2679e-312">Eseguire l'app.</span><span class="sxs-lookup"><span data-stu-id="2679e-312">Run the app.</span></span>
* <span data-ttu-id="2679e-313">Selezionare il collegamento **Students** (Studenti) e quindi **Crea nuovo**.</span><span class="sxs-lookup"><span data-stu-id="2679e-313">Select the **Students** link and then **Create New**.</span></span>
* <span data-ttu-id="2679e-314">Eseguire il test dei collegamenti Edit (Modifica), Details (Dettagli) e Delete (Elimina).</span><span class="sxs-lookup"><span data-stu-id="2679e-314">Test the Edit, Details, and Delete links.</span></span>

## <a name="seed-the-database"></a><span data-ttu-id="2679e-315">Specificare il valore di inizializzazione del database</span><span class="sxs-lookup"><span data-stu-id="2679e-315">Seed the database</span></span>

<span data-ttu-id="2679e-316">Il metodo `EnsureCreated` crea un database vuoto.</span><span class="sxs-lookup"><span data-stu-id="2679e-316">The `EnsureCreated` method creates an empty database.</span></span> <span data-ttu-id="2679e-317">In questa sezione viene aggiunto il codice che popola il database con dati di test.</span><span class="sxs-lookup"><span data-stu-id="2679e-317">This section adds code that populates the database with test data.</span></span>

<span data-ttu-id="2679e-318">Creare *Data/DbInitializer.cs* con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="2679e-318">Create *Data/DbInitializer.cs* with the following code:</span></span>
<!-- next update, keep this file in the project and surround with #if -->
  [!code-csharp[Main](intro/samples/cu30snapshots/1-intro/Data/DbInitializer.cs)]

  <span data-ttu-id="2679e-319">Il codice controlla se esistono studenti nel database.</span><span class="sxs-lookup"><span data-stu-id="2679e-319">The code checks if there are any students in the database.</span></span> <span data-ttu-id="2679e-320">Se non sono presenti studenti, aggiunge i dati di test al database.</span><span class="sxs-lookup"><span data-stu-id="2679e-320">If there are no students, it adds test data to the database.</span></span> <span data-ttu-id="2679e-321">I dati di test vengono creati in matrici anziché in raccolte `List<T>` per ottimizzare le prestazioni.</span><span class="sxs-lookup"><span data-stu-id="2679e-321">It creates the test data in arrays rather than `List<T>` collections to optimize performance.</span></span>

<span data-ttu-id="2679e-322">In *Program.cs* sostituire la chiamata di `EnsureCreated` con una chiamata di `DbInitializer.Initialize`:</span><span class="sxs-lookup"><span data-stu-id="2679e-322">In *Program.cs*, replace the `EnsureCreated` call with a `DbInitializer.Initialize` call:</span></span>

  ```csharp
  // context.Database.EnsureCreated();
  DbInitializer.Initialize(context);
  ```

# <a name="visual-studio"></a>[<span data-ttu-id="2679e-323">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="2679e-323">Visual Studio</span></span>](#tab/visual-studio)

<span data-ttu-id="2679e-324">Arrestare l'app se è in esecuzione ed eseguire il comando seguente nella **console di Gestione pacchetti**:</span><span class="sxs-lookup"><span data-stu-id="2679e-324">Stop the app if it's running, and run the following command in the **Package Manager Console** (PMC):</span></span>

```powershell
Drop-Database -Confirm
```

<span data-ttu-id="2679e-325">Rispondere con `Y` per eliminare il database.</span><span class="sxs-lookup"><span data-stu-id="2679e-325">Respond with `Y` to delete the database.</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="2679e-326">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="2679e-326">Visual Studio Code</span></span>](#tab/visual-studio-code)

* <span data-ttu-id="2679e-327">Arrestare l'app se è in esecuzione ed eliminare il file *CU.db*.</span><span class="sxs-lookup"><span data-stu-id="2679e-327">Stop the app if it's running, and delete the *CU.db* file.</span></span>

---

* <span data-ttu-id="2679e-328">Riavviare l'app.</span><span class="sxs-lookup"><span data-stu-id="2679e-328">Restart the app.</span></span>
* <span data-ttu-id="2679e-329">Selezionare la pagina Students per visualizzare i dati di seeding.</span><span class="sxs-lookup"><span data-stu-id="2679e-329">Select the Students page to see the seeded data.</span></span>

## <a name="view-the-database"></a><span data-ttu-id="2679e-330">Visualizzare il database</span><span class="sxs-lookup"><span data-stu-id="2679e-330">View the database</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="2679e-331">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="2679e-331">Visual Studio</span></span>](#tab/visual-studio)

* <span data-ttu-id="2679e-332">Aprire **Esplora oggetti di SQL Server** dal menu **Visualizza** in Visual Studio.</span><span class="sxs-lookup"><span data-stu-id="2679e-332">Open **SQL Server Object Explorer** (SSOX) from the **View** menu in Visual Studio.</span></span>
* <span data-ttu-id="2679e-333">In Esplora oggetti di SQL Server selezionare **(localdb)\MSSQLLocalDB > Database > SchoolContext-{GUID}**.</span><span class="sxs-lookup"><span data-stu-id="2679e-333">In SSOX, select **(localdb)\MSSQLLocalDB > Databases > SchoolContext-{GUID}**.</span></span> <span data-ttu-id="2679e-334">Il nome del database viene generato dal nome del contesto specificato in precedenza con l'aggiunta di un trattino e un GUID.</span><span class="sxs-lookup"><span data-stu-id="2679e-334">The database name is generated from the context name you provided earlier plus a dash and a GUID.</span></span>
* <span data-ttu-id="2679e-335">Espandere il nodo **Tabelle**.</span><span class="sxs-lookup"><span data-stu-id="2679e-335">Expand the **Tables** node.</span></span>
* <span data-ttu-id="2679e-336">Fare clic con il pulsante destro del mouse sulla tabella **Student** (Studente) e fare clic su **Visualizza dati** per visualizzare le colonne create e le righe inserite nella tabella.</span><span class="sxs-lookup"><span data-stu-id="2679e-336">Right-click the **Student** table and click **View Data** to see the columns created and the rows inserted into the table.</span></span>
* <span data-ttu-id="2679e-337">Fare clic con il pulsante destro del mouse sulla tabella **Student** e scegliere **Visualizza codice** per vedere il mapping tra il modello `Student` e lo schema della tabella `Student`.</span><span class="sxs-lookup"><span data-stu-id="2679e-337">Right-click the **Student** table and click **View Code** to see how the `Student` model maps to the `Student` table schema.</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="2679e-338">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="2679e-338">Visual Studio Code</span></span>](#tab/visual-studio-code)

<span data-ttu-id="2679e-339">Usare lo strumento SQLite per visualizzare lo schema del database e i dati di seeding.</span><span class="sxs-lookup"><span data-stu-id="2679e-339">Use your SQLite tool to view the database schema and seeded data.</span></span> <span data-ttu-id="2679e-340">Il file di database è denominato *CU.db* e si trova nella cartella del progetto.</span><span class="sxs-lookup"><span data-stu-id="2679e-340">The database file is named *CU.db* and is located in the project folder.</span></span>

---

## <a name="asynchronous-code"></a><span data-ttu-id="2679e-341">Codice asincrono</span><span class="sxs-lookup"><span data-stu-id="2679e-341">Asynchronous code</span></span>

<span data-ttu-id="2679e-342">La programmazione asincrona è la modalità predefinita per ASP.NET Core ed Entity Framework Core.</span><span class="sxs-lookup"><span data-stu-id="2679e-342">Asynchronous programming is the default mode for ASP.NET Core and EF Core.</span></span>

<span data-ttu-id="2679e-343">Per un server Web è disponibile un numero limitato di thread e in situazioni di carico elevato tutti i thread disponibili potrebbero essere in uso.</span><span class="sxs-lookup"><span data-stu-id="2679e-343">A web server has a limited number of threads available, and in high load situations all of the available threads might be in use.</span></span> <span data-ttu-id="2679e-344">In queste circostanze il server non può elaborare nuove richieste finché i thread non saranno liberi.</span><span class="sxs-lookup"><span data-stu-id="2679e-344">When that happens, the server can't process new requests until the threads are freed up.</span></span> <span data-ttu-id="2679e-345">Con il codice sincrono, è possibile che molti thread siano collegati mentre non funzionano perché sono in attesa del completamento dell'I/O.</span><span class="sxs-lookup"><span data-stu-id="2679e-345">With synchronous code, many threads may be tied up while they aren't doing work because they're waiting for I/O to complete.</span></span> <span data-ttu-id="2679e-346">Con il codice asincrono, se un processo è in attesa del completamento dell'operazione I/O, il thread viene liberato e il server lo può usare per l'elaborazione di altre richieste.</span><span class="sxs-lookup"><span data-stu-id="2679e-346">With asynchronous code, when a process is waiting for I/O to complete, its thread is freed up for the server to use for processing other requests.</span></span> <span data-ttu-id="2679e-347">Di conseguenza, il codice asincrono consente un uso più efficiente delle risorse del server e il server può gestire una maggiore quantità di traffico senza ritardi.</span><span class="sxs-lookup"><span data-stu-id="2679e-347">As a result, asynchronous code enables server resources to be used more efficiently, and the server can handle more traffic without delays.</span></span>

<span data-ttu-id="2679e-348">Il codice asincrono comporta un minimo sovraccarico in fase di esecuzione.</span><span class="sxs-lookup"><span data-stu-id="2679e-348">Asynchronous code does introduce a small amount of overhead at run time.</span></span> <span data-ttu-id="2679e-349">In caso di traffico ridotto, il calo delle prestazioni è irrilevante, mentre nelle situazioni di traffico elevato, è essenziale ottimizzare le prestazioni potenziali.</span><span class="sxs-lookup"><span data-stu-id="2679e-349">For low traffic situations, the performance hit is negligible, while for high traffic situations, the potential performance improvement is substantial.</span></span>

<span data-ttu-id="2679e-350">Nel codice seguente la parola chiave [async](/dotnet/csharp/language-reference/keywords/async), il valore restituito `Task`, la parola chiave `await` e il metodo `ToListAsync` consentono di eseguire il codice in modo asincrono.</span><span class="sxs-lookup"><span data-stu-id="2679e-350">In the following code, the [async](/dotnet/csharp/language-reference/keywords/async) keyword, `Task` return value, `await` keyword, and `ToListAsync` method make the code execute asynchronously.</span></span>

```csharp
public async Task OnGetAsync()
{
    Students = await _context.Students.ToListAsync();
}
```

* <span data-ttu-id="2679e-351">La parola chiave `async` indica al compilatore di eseguire le operazioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="2679e-351">The `async` keyword tells the compiler to:</span></span>
  * <span data-ttu-id="2679e-352">Generare callback per parti del corpo del metodo.</span><span class="sxs-lookup"><span data-stu-id="2679e-352">Generate callbacks for parts of the method body.</span></span>
  * <span data-ttu-id="2679e-353">Creare l'oggetto [Task](/dotnet/csharp/programming-guide/concepts/async/async-return-types#BKMK_TaskReturnType) restituito.</span><span class="sxs-lookup"><span data-stu-id="2679e-353">Create the [Task](/dotnet/csharp/programming-guide/concepts/async/async-return-types#BKMK_TaskReturnType) object that's returned.</span></span>
* <span data-ttu-id="2679e-354">Il tipo restituito `Task` rappresenta il lavoro in corso.</span><span class="sxs-lookup"><span data-stu-id="2679e-354">The `Task` return type represents ongoing work.</span></span>
* <span data-ttu-id="2679e-355">La parola chiave `await` indica al compilatore di suddividere il metodo in due parti.</span><span class="sxs-lookup"><span data-stu-id="2679e-355">The `await` keyword causes the compiler to split the method into two parts.</span></span> <span data-ttu-id="2679e-356">La prima parte termina con l'operazione avviata in modo asincrono.</span><span class="sxs-lookup"><span data-stu-id="2679e-356">The first part ends with the operation that's started asynchronously.</span></span> <span data-ttu-id="2679e-357">La seconda parte viene inserita in un metodo di callback che viene chiamato al termine dell'operazione.</span><span class="sxs-lookup"><span data-stu-id="2679e-357">The second part is put into a callback method that's called when the operation completes.</span></span>
* <span data-ttu-id="2679e-358">`ToListAsync` è la versione asincrona del metodo di estensione `ToList`.</span><span class="sxs-lookup"><span data-stu-id="2679e-358">`ToListAsync` is the asynchronous version of the `ToList` extension method.</span></span>

<span data-ttu-id="2679e-359">Alcuni aspetti da considerare quando si scrive codice asincrono usato da Entity Framework Core:</span><span class="sxs-lookup"><span data-stu-id="2679e-359">Some things to be aware of when writing asynchronous code that uses EF Core:</span></span>

* <span data-ttu-id="2679e-360">Solo le istruzioni che generano query o comandi da inviare al database vengono eseguite in modo asincrono,</span><span class="sxs-lookup"><span data-stu-id="2679e-360">Only statements that cause queries or commands to be sent to the database are executed asynchronously.</span></span> <span data-ttu-id="2679e-361">Sono incluse `ToListAsync`, `SingleOrDefaultAsync`, `FirstOrDefaultAsync` e `SaveChangesAsync`.</span><span class="sxs-lookup"><span data-stu-id="2679e-361">That includes `ToListAsync`, `SingleOrDefaultAsync`, `FirstOrDefaultAsync`, and `SaveChangesAsync`.</span></span> <span data-ttu-id="2679e-362">Non sono incluse le istruzioni che modificano solo un oggetto `IQueryable`, ad esempio `var students = context.Students.Where(s => s.LastName == "Davolio")`.</span><span class="sxs-lookup"><span data-stu-id="2679e-362">It doesn't include statements that just change an `IQueryable`, such as `var students = context.Students.Where(s => s.LastName == "Davolio")`.</span></span>
* <span data-ttu-id="2679e-363">Un contesto di Entity Framework Core non è thread-safe. Non tentare di eseguire più operazioni parallelamente.</span><span class="sxs-lookup"><span data-stu-id="2679e-363">An EF Core context isn't thread safe: don't try to do multiple operations in parallel.</span></span>
* <span data-ttu-id="2679e-364">Per sfruttare i vantaggi del codice asincrono in termini di prestazioni, verificare che i pacchetti della libreria impiegati, ad esempio per il paging, usino la modalità asincrona per chiamare i metodi di EF Core che inviano query al database.</span><span class="sxs-lookup"><span data-stu-id="2679e-364">To take advantage of the performance benefits of async code, verify that library packages (such as for paging) use async if they call EF Core methods that send queries to the database.</span></span>

<span data-ttu-id="2679e-365">Per altre informazioni sulla programmazione asincrona in .NET, vedere [Panoramica della programmazione asincrona](/dotnet/standard/async) e [Programmazione asincrona con async e await](/dotnet/csharp/programming-guide/concepts/async/).</span><span class="sxs-lookup"><span data-stu-id="2679e-365">For more information about asynchronous programming in .NET, see [Async Overview](/dotnet/standard/async) and [Asynchronous programming with async and await](/dotnet/csharp/programming-guide/concepts/async/).</span></span>

<!-- Review: See https://github.com/dotnet/AspNetCore.Docs/issues/14528 -->
## <a name="performance-considerations"></a><span data-ttu-id="2679e-366">Considerazioni sulle prestazioni</span><span class="sxs-lookup"><span data-stu-id="2679e-366">Performance considerations</span></span>

<span data-ttu-id="2679e-367">In generale, una pagina Web non deve caricare un numero arbitrario di righe.</span><span class="sxs-lookup"><span data-stu-id="2679e-367">In general, a web page shouldn't be loading an arbitrary number of rows.</span></span> <span data-ttu-id="2679e-368">Una query deve usare il paging o un approccio di limitazione.</span><span class="sxs-lookup"><span data-stu-id="2679e-368">A query should use paging or a limiting approach.</span></span> <span data-ttu-id="2679e-369">Ad esempio, la query precedente può utilizzare `Take` per limitare le righe restituite:</span><span class="sxs-lookup"><span data-stu-id="2679e-369">For example, the preceding query could use `Take` to limit the rows returned:</span></span>

[!code-csharp[Main](intro/samples/cu50snapshots/Index.cshtml.cs?name=snippet)]

<span data-ttu-id="2679e-370">L'enumerazione di una tabella di grandi dimensioni in una vista può restituire una risposta HTTP 200 parzialmente costruita se un'eccezione del database si verifica in modo parziale attraverso l'enumerazione.</span><span class="sxs-lookup"><span data-stu-id="2679e-370">Enumerating a large table in a view could return a partially constructed HTTP 200 response if a database exception occurs part way through the enumeration.</span></span>

<span data-ttu-id="2679e-371"><xref:Microsoft.AspNetCore.Mvc.MvcOptions.MaxModelBindingCollectionSize> il valore predefinito è 1024.</span><span class="sxs-lookup"><span data-stu-id="2679e-371"><xref:Microsoft.AspNetCore.Mvc.MvcOptions.MaxModelBindingCollectionSize> defaults to 1024.</span></span> <span data-ttu-id="2679e-372">Il codice seguente imposta `MaxModelBindingCollectionSize` :</span><span class="sxs-lookup"><span data-stu-id="2679e-372">The following code sets `MaxModelBindingCollectionSize`:</span></span>

[!code-csharp[Main](intro/samples/cu50/StartupMaxMBsize.cs?name=snippet_ConfigureServices)]

<span data-ttu-id="2679e-373">Il paging verrà trattato più avanti nell'esercitazione.</span><span class="sxs-lookup"><span data-stu-id="2679e-373">Paging is covered later in the tutorial.</span></span>

## <a name="next-steps"></a><span data-ttu-id="2679e-374">Passaggi successivi</span><span class="sxs-lookup"><span data-stu-id="2679e-374">Next steps</span></span>

> [!div class="step-by-step"]
> [<span data-ttu-id="2679e-375">Esercitazione successiva</span><span class="sxs-lookup"><span data-stu-id="2679e-375">Next tutorial</span></span>](xref:data/ef-rp/crud)

::: moniker-end

::: moniker range=">= aspnetcore-3.0 < aspnetcore-5.0"

<span data-ttu-id="2679e-376">Questo è il primo di una serie di esercitazioni che illustrano come usare Entity Framework (EF) Core in un'app [ASP.NET Core Razor pages](xref:razor-pages/index) .</span><span class="sxs-lookup"><span data-stu-id="2679e-376">This is the first in a series of tutorials that show how to use Entity Framework (EF) Core in an [ASP.NET Core Razor Pages](xref:razor-pages/index) app.</span></span> <span data-ttu-id="2679e-377">Con queste esercitazioni viene creato un sito Web per una fittizia Contoso University.</span><span class="sxs-lookup"><span data-stu-id="2679e-377">The tutorials build a web site for a fictional Contoso University.</span></span> <span data-ttu-id="2679e-378">Il sito include funzionalità, come ad esempio l'ammissione di studenti, la creazione di corsi e le assegnazioni degli insegnati.</span><span class="sxs-lookup"><span data-stu-id="2679e-378">The site includes functionality such as student admission, course creation, and instructor assignments.</span></span> <span data-ttu-id="2679e-379">Nell'esercitazione viene usato l'approccio Code First.</span><span class="sxs-lookup"><span data-stu-id="2679e-379">The tutorial uses the code first approach.</span></span> <span data-ttu-id="2679e-380">Per informazioni su come seguire questa esercitazione usando il primo approccio al database, vedere [questo problema di GitHub](https://github.com/dotnet/AspNetCore.Docs/issues/16897).</span><span class="sxs-lookup"><span data-stu-id="2679e-380">For information on following this tutorial using the database first approach, see [this Github issue](https://github.com/dotnet/AspNetCore.Docs/issues/16897).</span></span>

[<span data-ttu-id="2679e-381">Scaricare o visualizzare l'app completata.</span><span class="sxs-lookup"><span data-stu-id="2679e-381">Download or view the completed app.</span></span>](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/data/ef-rp/intro/samples) <span data-ttu-id="2679e-382">[Istruzioni per il download](xref:index#how-to-download-a-sample).</span><span class="sxs-lookup"><span data-stu-id="2679e-382">[Download instructions](xref:index#how-to-download-a-sample).</span></span>

## <a name="prerequisites"></a><span data-ttu-id="2679e-383">Prerequisiti</span><span class="sxs-lookup"><span data-stu-id="2679e-383">Prerequisites</span></span>

* <span data-ttu-id="2679e-384">Se non si ha familiarità con Razor le pagine, vedere la serie di esercitazioni [introduttive sulle Razor pagine](xref:tutorials/razor-pages/razor-pages-start) prima di iniziare.</span><span class="sxs-lookup"><span data-stu-id="2679e-384">If you're new to Razor Pages, go through the [Get started with Razor Pages](xref:tutorials/razor-pages/razor-pages-start) tutorial series before starting this one.</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="2679e-385">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="2679e-385">Visual Studio</span></span>](#tab/visual-studio)

[!INCLUDE[VS prereqs](~/includes/net-core-prereqs-vs-3.0.md)]

# <a name="visual-studio-code"></a>[<span data-ttu-id="2679e-386">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="2679e-386">Visual Studio Code</span></span>](#tab/visual-studio-code)

[!INCLUDE[VS Code prereqs](~/includes/net-core-prereqs-vsc-3.0.md)]

---

## <a name="database-engines"></a><span data-ttu-id="2679e-387">Motori di database</span><span class="sxs-lookup"><span data-stu-id="2679e-387">Database engines</span></span>

<span data-ttu-id="2679e-388">Le istruzioni per Visual Studio usano [SQL Server Local DB](/sql/database-engine/configure-windows/sql-server-2016-express-localdb), una versione di SQL Server Express eseguita solo in Windows.</span><span class="sxs-lookup"><span data-stu-id="2679e-388">The Visual Studio instructions use [SQL Server LocalDB](/sql/database-engine/configure-windows/sql-server-2016-express-localdb), a version of SQL Server Express that runs only on Windows.</span></span>

<span data-ttu-id="2679e-389">Le istruzioni per Visual Studio Code usano [SQLite](https://www.sqlite.org/), un motore di database multipiattaforma.</span><span class="sxs-lookup"><span data-stu-id="2679e-389">The Visual Studio Code instructions use [SQLite](https://www.sqlite.org/), a cross-platform database engine.</span></span>

<span data-ttu-id="2679e-390">Se si sceglie di usare SQLite, scaricare e installare uno strumento di terze parti per la gestione e la visualizzazione di un database SQLite, ad esempio [DB Browser per SQLite](https://sqlitebrowser.org/).</span><span class="sxs-lookup"><span data-stu-id="2679e-390">If you choose to use SQLite, download and install a third-party tool for managing and viewing a SQLite database, such as [DB Browser for SQLite](https://sqlitebrowser.org/).</span></span>

## <a name="troubleshooting"></a><span data-ttu-id="2679e-391">Risoluzione dei problemi</span><span class="sxs-lookup"><span data-stu-id="2679e-391">Troubleshooting</span></span>

<span data-ttu-id="2679e-392">Se si verifica un problema che non è possibile risolvere, confrontare il codice con il [progetto completato](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/data/ef-rp/intro/samples).</span><span class="sxs-lookup"><span data-stu-id="2679e-392">If you run into a problem you can't resolve, compare your code to the [completed project](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/data/ef-rp/intro/samples).</span></span> <span data-ttu-id="2679e-393">Un buon metodo per ottenere assistenza è quello di pubblicare una domanda in StackOverflow.com con il [tag ASP.NET Core](https://stackoverflow.com/questions/tagged/asp.net-core) o il [tag EF Core](https://stackoverflow.com/questions/tagged/entity-framework-core).</span><span class="sxs-lookup"><span data-stu-id="2679e-393">A good way to get help is by posting a question to StackOverflow.com, using the [ASP.NET Core tag](https://stackoverflow.com/questions/tagged/asp.net-core) or the [EF Core tag](https://stackoverflow.com/questions/tagged/entity-framework-core).</span></span>

## <a name="the-sample-app"></a><span data-ttu-id="2679e-394">App di esempio</span><span class="sxs-lookup"><span data-stu-id="2679e-394">The sample app</span></span>

<span data-ttu-id="2679e-395">L'applicazione compilata in queste esercitazioni è il sito Web di base di un'università.</span><span class="sxs-lookup"><span data-stu-id="2679e-395">The app built in these tutorials is a basic university web site.</span></span> <span data-ttu-id="2679e-396">Gli utenti possono visualizzare e aggiornare le informazioni che riguardano studenti, corsi e insegnanti.</span><span class="sxs-lookup"><span data-stu-id="2679e-396">Users can view and update student, course, and instructor information.</span></span> <span data-ttu-id="2679e-397">Di seguito sono disponibili alcune schermate dell'esercitazione.</span><span class="sxs-lookup"><span data-stu-id="2679e-397">Here are a few of the screens created in the tutorial.</span></span>

![Pagina Student Index (Indice degli studenti)](intro/_static/students-index30.png)

![Pagina Students Edit (Modifica studenti)](intro/_static/student-edit30.png)

<span data-ttu-id="2679e-400">Lo stile dell'interfaccia utente del sito è basato sui modelli di progetto predefiniti.</span><span class="sxs-lookup"><span data-stu-id="2679e-400">The UI style of this site is based on the built-in project templates.</span></span> <span data-ttu-id="2679e-401">L'obiettivo dell'esercitazione è quello di usare EF Core e non di personalizzare l'interfaccia utente.</span><span class="sxs-lookup"><span data-stu-id="2679e-401">The tutorial's focus is on how to use EF Core, not how to customize the UI.</span></span>

<span data-ttu-id="2679e-402">Seguire il collegamento nella parte superiore della pagina per ottenere il codice sorgente per il progetto completato.</span><span class="sxs-lookup"><span data-stu-id="2679e-402">Follow the link at the top of the page to get the source code for the completed project.</span></span> <span data-ttu-id="2679e-403">La cartella *cu30* contiene il codice per la versione di ASP.NET Core 3.0 dell'esercitazione.</span><span class="sxs-lookup"><span data-stu-id="2679e-403">The *cu30* folder has the code for the ASP.NET Core 3.0 version of the tutorial.</span></span> <span data-ttu-id="2679e-404">I file che riflettono lo stato del codice per le esercitazioni 1-7 si trovano nella cartella *cu30snapshots*.</span><span class="sxs-lookup"><span data-stu-id="2679e-404">Files that reflect the state of the code for tutorials 1-7 can be found in the *cu30snapshots* folder.</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="2679e-405">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="2679e-405">Visual Studio</span></span>](#tab/visual-studio)

<span data-ttu-id="2679e-406">Per eseguire l'app dopo il download del progetto completato:</span><span class="sxs-lookup"><span data-stu-id="2679e-406">To run the app after downloading the completed project:</span></span>

* <span data-ttu-id="2679e-407">Compilare il progetto.</span><span class="sxs-lookup"><span data-stu-id="2679e-407">Build the project.</span></span>
* <span data-ttu-id="2679e-408">Nella console di Gestione pacchetti eseguire il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="2679e-408">In Package Manager Console (PMC) run the following command:</span></span>

  ```powershell
  Update-Database
  ```

* <span data-ttu-id="2679e-409">Eseguire il progetto per il seeding del database.</span><span class="sxs-lookup"><span data-stu-id="2679e-409">Run the project to seed the database.</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="2679e-410">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="2679e-410">Visual Studio Code</span></span>](#tab/visual-studio-code)

<span data-ttu-id="2679e-411">Per eseguire l'app dopo il download del progetto completato:</span><span class="sxs-lookup"><span data-stu-id="2679e-411">To run the app after downloading the completed project:</span></span>

* <span data-ttu-id="2679e-412">Eliminare *ContosoUniversity.csproj* e rinominare *ContosoUniversitySQLite.csproj* in *ContosoUniversity.csproj*.</span><span class="sxs-lookup"><span data-stu-id="2679e-412">Delete *ContosoUniversity.csproj*, and rename *ContosoUniversitySQLite.csproj* to *ContosoUniversity.csproj*.</span></span>
* <span data-ttu-id="2679e-413">In *Program.cs*, impostare come commento l' `#define Startup` `StartupSQLite` utilizzo di.</span><span class="sxs-lookup"><span data-stu-id="2679e-413">In *Program.cs*, comment out `#define Startup` so `StartupSQLite` is used.</span></span>
* <span data-ttu-id="2679e-414">Eliminare *appSettings.json* e rinominare *appSettingsSQLite.json* in *appSettings.json*.</span><span class="sxs-lookup"><span data-stu-id="2679e-414">Delete *appSettings.json*, and rename *appSettingsSQLite.json* to *appSettings.json*.</span></span>
* <span data-ttu-id="2679e-415">Eliminare la cartella *Migrations* e rinominare *MigrationsSQL* in *Migrations*.</span><span class="sxs-lookup"><span data-stu-id="2679e-415">Delete the *Migrations* folder, and rename *MigrationsSQL* to *Migrations*.</span></span>
* <span data-ttu-id="2679e-416">Eseguire una ricerca globale `#if SQLiteVersion` e rimuovere `#if SQLiteVersion` e l'istruzione associata `#endif` .</span><span class="sxs-lookup"><span data-stu-id="2679e-416">Do a global search for `#if SQLiteVersion` and remove `#if SQLiteVersion` and the associated `#endif` statement.</span></span>
* <span data-ttu-id="2679e-417">Compilare il progetto.</span><span class="sxs-lookup"><span data-stu-id="2679e-417">Build the project.</span></span>
* <span data-ttu-id="2679e-418">Al prompt dei comandi nella cartella del progetto eseguire i comandi seguenti:</span><span class="sxs-lookup"><span data-stu-id="2679e-418">At a command prompt in the project folder, run the following commands:</span></span>

  ```dotnetcli
  dotnet tool uninstall --global dotnet-ef
  dotnet tool install --global dotnet-ef --version 5.0.0-*
  dotnet ef database update
  ```

* <span data-ttu-id="2679e-419">Nello strumento SQLite eseguire l'istruzione SQL seguente:</span><span class="sxs-lookup"><span data-stu-id="2679e-419">In your SQLite tool, run the following SQL statement:</span></span>

  ```sql
  UPDATE Department SET RowVersion = randomblob(8)
  ```
  
* <span data-ttu-id="2679e-420">Eseguire il progetto per il seeding del database.</span><span class="sxs-lookup"><span data-stu-id="2679e-420">Run the project to seed the database.</span></span>

---

## <a name="create-the-web-app-project"></a><span data-ttu-id="2679e-421">Creare il progetto di app Web</span><span class="sxs-lookup"><span data-stu-id="2679e-421">Create the web app project</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="2679e-422">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="2679e-422">Visual Studio</span></span>](#tab/visual-studio)

* <span data-ttu-id="2679e-423">Dal menu **File** di Visual Studio selezionare **Nuovo** > **Progetto**.</span><span class="sxs-lookup"><span data-stu-id="2679e-423">From the Visual Studio **File** menu, select **New** > **Project**.</span></span>
* <span data-ttu-id="2679e-424">Selezionare **Applicazione Web ASP.NET Core**.</span><span class="sxs-lookup"><span data-stu-id="2679e-424">Select **ASP.NET Core Web Application**.</span></span>
* <span data-ttu-id="2679e-425">Denominare il progetto *ContosoUniversity*.</span><span class="sxs-lookup"><span data-stu-id="2679e-425">Name the project *ContosoUniversity*.</span></span> <span data-ttu-id="2679e-426">È importante usare questo nome esatto, incluse le maiuscole, in modo che gli spazi dei nomi corrispondano quando il codice viene copiato e incollato.</span><span class="sxs-lookup"><span data-stu-id="2679e-426">It's important to use this exact name including capitalization, so the namespaces match when code is copied and pasted.</span></span>
* <span data-ttu-id="2679e-427">Selezionare **.NET Core** e **ASP.NET Core 3.0** nell'elenco a discesa, quindi selezionare **Applicazione Web**.</span><span class="sxs-lookup"><span data-stu-id="2679e-427">Select **.NET Core** and **ASP.NET Core 3.0** in the dropdowns, and then select **Web Application**.</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="2679e-428">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="2679e-428">Visual Studio Code</span></span>](#tab/visual-studio-code)

* <span data-ttu-id="2679e-429">In un terminale passare alla cartella in cui deve essere creata la cartella del progetto.</span><span class="sxs-lookup"><span data-stu-id="2679e-429">In a terminal, navigate to the folder in which the project folder should be created.</span></span>

* <span data-ttu-id="2679e-430">Eseguire i comandi seguenti per creare un Razor progetto di pagine e `cd` nella nuova cartella di progetto:</span><span class="sxs-lookup"><span data-stu-id="2679e-430">Run the following commands to create a Razor Pages project and `cd` into the new project folder:</span></span>

  ```dotnetcli
  dotnet new webapp -o ContosoUniversity
  cd ContosoUniversity
  ```

---

## <a name="set-up-the-site-style"></a><span data-ttu-id="2679e-431">Impostare lo stile del sito</span><span class="sxs-lookup"><span data-stu-id="2679e-431">Set up the site style</span></span>

<span data-ttu-id="2679e-432">Per impostare l'intestazione, il piè di pagina e il menu del sito, aggiornare *Pages/Shared/_Layout.cshtml*:</span><span class="sxs-lookup"><span data-stu-id="2679e-432">Set up the site header, footer, and menu by updating *Pages/Shared/_Layout.cshtml*:</span></span>

* <span data-ttu-id="2679e-433">Modificare tutte le occorrenze di "ContosoUniversity" in "Contoso University".</span><span class="sxs-lookup"><span data-stu-id="2679e-433">Change each occurrence of "ContosoUniversity" to "Contoso University".</span></span> <span data-ttu-id="2679e-434">Le occorrenze sono tre.</span><span class="sxs-lookup"><span data-stu-id="2679e-434">There are three occurrences.</span></span>

* <span data-ttu-id="2679e-435">Eliminare le voci di menu **Home** e **Privacy**, quindi aggiungere le voci per **About** (Informazioni su), **Students** (Studenti), **Courses** (Corsi), **Instructors** (Insegnanti) e **Departments** (Dipartimenti).</span><span class="sxs-lookup"><span data-stu-id="2679e-435">Delete the **Home** and **Privacy** menu entries, and add entries for **About**, **Students**, **Courses**, **Instructors**, and **Departments**.</span></span>

<span data-ttu-id="2679e-436">Le modifiche sono evidenziate.</span><span class="sxs-lookup"><span data-stu-id="2679e-436">The changes are highlighted.</span></span>

[!code-cshtml[Main](intro/samples/cu30/Pages/Shared/_Layout.cshtml?highlight=6,14,21-35,49)]

<span data-ttu-id="2679e-437">In *Pages/Index.cshtml* sostituire il contenuto del file con il codice seguente. In questo modo il testo su ASP.NET sarà sostituito dal testo relativo a questa app:</span><span class="sxs-lookup"><span data-stu-id="2679e-437">In *Pages/Index.cshtml*, replace the contents of the file with the following code to replace the text about ASP.NET Core with text about this app:</span></span>

[!code-cshtml[Main](intro/samples/cu30/Pages/Index.cshtml)]

<span data-ttu-id="2679e-438">Eseguire l'app per verificare che venga visualizzata la home page.</span><span class="sxs-lookup"><span data-stu-id="2679e-438">Run the app to verify that the home page appears.</span></span>

## <a name="the-data-model"></a><span data-ttu-id="2679e-439">Modello di dati</span><span class="sxs-lookup"><span data-stu-id="2679e-439">The data model</span></span>

<span data-ttu-id="2679e-440">Nelle sezioni seguenti viene descritto come creare un modello di dati:</span><span class="sxs-lookup"><span data-stu-id="2679e-440">The following sections create a data model:</span></span>

![Diagramma modello di dati Course/Enrollment/Student (Corso/Iscrizione/Studente)](intro/_static/data-model-diagram.png)

<span data-ttu-id="2679e-442">Uno studente può iscriversi a un numero qualsiasi di corsi e un corso può avere un numero qualsiasi di studenti iscritti.</span><span class="sxs-lookup"><span data-stu-id="2679e-442">A student can enroll in any number of courses, and a course can have any number of students enrolled in it.</span></span>

## <a name="the-student-entity"></a><span data-ttu-id="2679e-443">Entità Student (Studente)</span><span class="sxs-lookup"><span data-stu-id="2679e-443">The Student entity</span></span>

![Diagramma entità Student (Studente)](intro/_static/student-entity.png)

* <span data-ttu-id="2679e-445">Creare una cartella *Models* nella cartella del progetto.</span><span class="sxs-lookup"><span data-stu-id="2679e-445">Create a *Models* folder in the project folder.</span></span>
* <span data-ttu-id="2679e-446">Creare *Models/Student.cs* con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="2679e-446">Create *Models/Student.cs* with the following code:</span></span>

  [!code-csharp[Main](intro/samples/cu30snapshots/1-intro/Models/Student.cs)]

<span data-ttu-id="2679e-447">La proprietà `ID` diventa la colonna chiave primaria della tabella del database che corrisponde a questa classe.</span><span class="sxs-lookup"><span data-stu-id="2679e-447">The `ID` property becomes the primary key column of the database table that corresponds to this class.</span></span> <span data-ttu-id="2679e-448">Per impostazione predefinita, Entity Framework Core interpreta una proprietà denominata `ID` o `classnameID` come chiave primaria.</span><span class="sxs-lookup"><span data-stu-id="2679e-448">By default, EF Core interprets a property that's named `ID` or `classnameID` as the primary key.</span></span> <span data-ttu-id="2679e-449">Il nome alternativo riconosciuto automaticamente per la chiave primaria della classe `Student` è quindi `StudentID`.</span><span class="sxs-lookup"><span data-stu-id="2679e-449">So the alternative automatically recognized name for the `Student` class primary key is `StudentID`.</span></span> <span data-ttu-id="2679e-450">Per altre informazioni, vedere [chiavi EF Core](/ef/core/modeling/keys?tabs=data-annotations).</span><span class="sxs-lookup"><span data-stu-id="2679e-450">For more information, see [EF Core - Keys](/ef/core/modeling/keys?tabs=data-annotations).</span></span>

<span data-ttu-id="2679e-451">La proprietà `Enrollments` rappresenta una [proprietà di navigazione](/ef/core/modeling/relationships).</span><span class="sxs-lookup"><span data-stu-id="2679e-451">The `Enrollments` property is a [navigation property](/ef/core/modeling/relationships).</span></span> <span data-ttu-id="2679e-452">Le proprietà di navigazione contengono altre entità correlate a questa entità.</span><span class="sxs-lookup"><span data-stu-id="2679e-452">Navigation properties hold other entities that are related to this entity.</span></span> <span data-ttu-id="2679e-453">In questo caso la proprietà `Enrollments` di un'entità `Student` contiene tutte le entità `Enrollment` correlate a tale studente.</span><span class="sxs-lookup"><span data-stu-id="2679e-453">In this case, the `Enrollments` property of a `Student` entity holds all of the `Enrollment` entities that are related to that Student.</span></span> <span data-ttu-id="2679e-454">Ad esempio, se una riga Student nel database presenta due righe Enrollment correlate, la proprietà di navigazione `Enrollments` contiene questi due entità Enrollment.</span><span class="sxs-lookup"><span data-stu-id="2679e-454">For example, if a Student row in the database has two related Enrollment rows, the `Enrollments` navigation property contains those two Enrollment entities.</span></span> 

<span data-ttu-id="2679e-455">Nel database, una riga Enrollment è correlata a una riga Student se la relativa colonna StudentID contiene il valore ID dello studente.</span><span class="sxs-lookup"><span data-stu-id="2679e-455">In the database, an Enrollment row is related to a Student row if its StudentID column contains the student's ID value.</span></span> <span data-ttu-id="2679e-456">Si supponga, ad esempio, che una riga Student abbia ID=1.</span><span class="sxs-lookup"><span data-stu-id="2679e-456">For example, suppose a Student row has ID=1.</span></span> <span data-ttu-id="2679e-457">Le righe Enrollment correlate avranno StudentID = 1.</span><span class="sxs-lookup"><span data-stu-id="2679e-457">Related Enrollment rows will have StudentID = 1.</span></span> <span data-ttu-id="2679e-458">StudentID è una *chiave esterna* nella tabella Enrollment.</span><span class="sxs-lookup"><span data-stu-id="2679e-458">StudentID is a *foreign key* in the Enrollment table.</span></span> 

<span data-ttu-id="2679e-459">La proprietà `Enrollments` è definita come `ICollection<Enrollment>` perché potrebbero essere presenti più entità Enrollment correlate.</span><span class="sxs-lookup"><span data-stu-id="2679e-459">The `Enrollments` property is defined as `ICollection<Enrollment>` because there may be multiple related Enrollment entities.</span></span> <span data-ttu-id="2679e-460">È possibile usare altri tipi di raccolta, ad esempio `List<Enrollment>` o `HashSet<Enrollment>`.</span><span class="sxs-lookup"><span data-stu-id="2679e-460">You can use other collection types, such as `List<Enrollment>` or `HashSet<Enrollment>`.</span></span> <span data-ttu-id="2679e-461">Se si specifica `ICollection<Enrollment>`, per impostazione predefinita Entity Framework Core crea una raccolta `HashSet<Enrollment>`.</span><span class="sxs-lookup"><span data-stu-id="2679e-461">When `ICollection<Enrollment>` is used, EF Core creates a `HashSet<Enrollment>` collection by default.</span></span>

## <a name="the-enrollment-entity"></a><span data-ttu-id="2679e-462">Entità Enrollment (Iscrizione)</span><span class="sxs-lookup"><span data-stu-id="2679e-462">The Enrollment entity</span></span>

![Diagramma entità Enrollment (Iscrizione)](intro/_static/enrollment-entity.png)

<span data-ttu-id="2679e-464">Creare *Models/Enrollment.cs* con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="2679e-464">Create *Models/Enrollment.cs* with the following code:</span></span>

[!code-csharp[Main](intro/samples/cu30snapshots/1-intro/Models/Enrollment.cs)]

<span data-ttu-id="2679e-465">La proprietà `EnrollmentID` è la chiave primaria. Questa entità usa il modello `classnameID` anziché `ID` da solo.</span><span class="sxs-lookup"><span data-stu-id="2679e-465">The `EnrollmentID` property is the primary key; this entity uses the `classnameID` pattern instead of `ID` by itself.</span></span> <span data-ttu-id="2679e-466">Per un modello di dati di produzione, scegliere un modello e usarlo in modo coerente.</span><span class="sxs-lookup"><span data-stu-id="2679e-466">For a production data model, choose one pattern and use it consistently.</span></span> <span data-ttu-id="2679e-467">Questa esercitazione usa entrambi solo per illustrare che entrambi funzionano.</span><span class="sxs-lookup"><span data-stu-id="2679e-467">This tutorial uses both just to illustrate that both work.</span></span> <span data-ttu-id="2679e-468">L'uso di `ID` senza `classname` rende più semplice l'implementazione di alcuni tipi di modifiche al modello di dati.</span><span class="sxs-lookup"><span data-stu-id="2679e-468">Using `ID` without `classname` makes it easier to implement some kinds of data model changes.</span></span>

<span data-ttu-id="2679e-469">La proprietà `Grade` è un oggetto`enum`.</span><span class="sxs-lookup"><span data-stu-id="2679e-469">The `Grade` property is an `enum`.</span></span> <span data-ttu-id="2679e-470">Il punto interrogativo dopo la dichiarazione del tipo `Grade` indica che la proprietà `Grade`[ammette i valori Null](/dotnet/csharp/programming-guide/nullable-types/).</span><span class="sxs-lookup"><span data-stu-id="2679e-470">The question mark after the `Grade` type declaration indicates that the `Grade` property is [nullable](/dotnet/csharp/programming-guide/nullable-types/).</span></span> <span data-ttu-id="2679e-471">Un voto con valore Null è diverso da un voto con valore zero. Null significa che un voto è sconosciuto o non è stato ancora assegnato.</span><span class="sxs-lookup"><span data-stu-id="2679e-471">A grade that's null is different from a zero grade&mdash;null means a grade isn't known or hasn't been assigned yet.</span></span>

<span data-ttu-id="2679e-472">La proprietà `StudentID` è una chiave esterna e la proprietà di navigazione corrispondente è `Student`.</span><span class="sxs-lookup"><span data-stu-id="2679e-472">The `StudentID` property is a foreign key, and the corresponding navigation property is `Student`.</span></span> <span data-ttu-id="2679e-473">Un'entità `Enrollment` è associata a un'entità `Student`, pertanto la proprietà contiene un'unica entità `Student`.</span><span class="sxs-lookup"><span data-stu-id="2679e-473">An `Enrollment` entity is associated with one `Student` entity, so the property contains a single `Student` entity.</span></span>

<span data-ttu-id="2679e-474">La proprietà `CourseID` è una chiave esterna e la proprietà di navigazione corrispondente è `Course`.</span><span class="sxs-lookup"><span data-stu-id="2679e-474">The `CourseID` property is a foreign key, and the corresponding navigation property is `Course`.</span></span> <span data-ttu-id="2679e-475">Un'entità `Enrollment` è associata a un'entità `Course`.</span><span class="sxs-lookup"><span data-stu-id="2679e-475">An `Enrollment` entity is associated with one `Course` entity.</span></span>

<span data-ttu-id="2679e-476">Entity Framework Core interpreta una proprietà come chiave esterna se è denominata `<navigation property name><primary key property name>`.</span><span class="sxs-lookup"><span data-stu-id="2679e-476">EF Core interprets a property as a foreign key if it's named `<navigation property name><primary key property name>`.</span></span> <span data-ttu-id="2679e-477">Ad esempio, `StudentID` è la chiave esterna per la proprietà di navigazione `Student`, dato che la chiave primaria `Student` dell'entità è `ID`.</span><span class="sxs-lookup"><span data-stu-id="2679e-477">For example,`StudentID` is the foreign key for the `Student` navigation property, since the `Student` entity's primary key is `ID`.</span></span> <span data-ttu-id="2679e-478">Le proprietà di chiave esterna possono anche essere denominate `<primary key property name>`.</span><span class="sxs-lookup"><span data-stu-id="2679e-478">Foreign key properties can also be named `<primary key property name>`.</span></span> <span data-ttu-id="2679e-479">Ad esempio, `CourseID` poiché la chiave primaria `Course` dell'entità è `CourseID`.</span><span class="sxs-lookup"><span data-stu-id="2679e-479">For example, `CourseID` since the `Course` entity's primary key is `CourseID`.</span></span>

## <a name="the-course-entity"></a><span data-ttu-id="2679e-480">Entità Course (Corso)</span><span class="sxs-lookup"><span data-stu-id="2679e-480">The Course entity</span></span>

![Diagramma entità Course (Corso)](intro/_static/course-entity.png)

<span data-ttu-id="2679e-482">Creare *Models/Course.cs* con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="2679e-482">Create *Models/Course.cs* with the following code:</span></span>

[!code-csharp[Main](intro/samples/cu30snapshots/1-intro/Models/Course.cs)]

<span data-ttu-id="2679e-483">La proprietà `Enrollments` rappresenta una proprietà di navigazione.</span><span class="sxs-lookup"><span data-stu-id="2679e-483">The `Enrollments` property is a navigation property.</span></span> <span data-ttu-id="2679e-484">È possibile correlare un'entità `Course` a un numero qualsiasi di entità `Enrollment`.</span><span class="sxs-lookup"><span data-stu-id="2679e-484">A `Course` entity can be related to any number of `Enrollment` entities.</span></span>

<span data-ttu-id="2679e-485">L'attributo `DatabaseGenerated` consente all'app di specificare la chiave primaria anziché chiedere al database di generarla.</span><span class="sxs-lookup"><span data-stu-id="2679e-485">The `DatabaseGenerated` attribute allows the app to specify the primary key rather than having the database generate it.</span></span>

<span data-ttu-id="2679e-486">Compilare il progetto per verificare che non siano presenti errori del compilatore.</span><span class="sxs-lookup"><span data-stu-id="2679e-486">Build the project to validate that there are no compiler errors.</span></span>

## <a name="scaffold-student-pages"></a><span data-ttu-id="2679e-487">Scaffolding delle pagine Student</span><span class="sxs-lookup"><span data-stu-id="2679e-487">Scaffold Student pages</span></span>

<span data-ttu-id="2679e-488">In questa sezione si userà lo strumento di scaffolding di ASP.NET Core per generare:</span><span class="sxs-lookup"><span data-stu-id="2679e-488">In this section, you use the ASP.NET Core scaffolding tool to generate:</span></span>

* <span data-ttu-id="2679e-489">Una classe *contesto* di EF Core.</span><span class="sxs-lookup"><span data-stu-id="2679e-489">An EF Core *context* class.</span></span> <span data-ttu-id="2679e-490">Il contesto è la classe principale che coordina le funzionalità di Entity Framework per un determinato modello di dati.</span><span class="sxs-lookup"><span data-stu-id="2679e-490">The context is the main class that coordinates Entity Framework functionality for a given data model.</span></span> <span data-ttu-id="2679e-491">Deriva dalla classe `Microsoft.EntityFrameworkCore.DbContext`.</span><span class="sxs-lookup"><span data-stu-id="2679e-491">It derives from the `Microsoft.EntityFrameworkCore.DbContext` class.</span></span>
* <span data-ttu-id="2679e-492">Razor pagine che gestiscono le operazioni di creazione, lettura, aggiornamento ed eliminazione (CRUD) per l' `Student` entità.</span><span class="sxs-lookup"><span data-stu-id="2679e-492">Razor pages that handle Create, Read, Update, and Delete (CRUD) operations for the `Student` entity.</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="2679e-493">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="2679e-493">Visual Studio</span></span>](#tab/visual-studio)

* <span data-ttu-id="2679e-494">Creare una cartella *Students* nella cartella *Pages*.</span><span class="sxs-lookup"><span data-stu-id="2679e-494">Create a *Students* folder in the *Pages* folder.</span></span>
* <span data-ttu-id="2679e-495">In **Esplora soluzioni** fare clic con il pulsante destro del mouse sulla cartella *Pages/Students* e quindi scegliere **Aggiungi** > **Nuovo elemento di scaffolding**.</span><span class="sxs-lookup"><span data-stu-id="2679e-495">In **Solution Explorer**, right-click the *Pages/Students* folder and select **Add** > **New Scaffolded Item**.</span></span>
* <span data-ttu-id="2679e-496">Nella finestra di dialogo **Aggiungi impalcatura** selezionare **Razor pages using Entity Framework (CRUD)** > **Add**.</span><span class="sxs-lookup"><span data-stu-id="2679e-496">In the **Add Scaffold** dialog, select **Razor Pages using Entity Framework (CRUD)** > **ADD**.</span></span>
* <span data-ttu-id="2679e-497">Nella finestra di dialogo **Aggiungi Razor pagine con Entity Framework (CRUD)** :</span><span class="sxs-lookup"><span data-stu-id="2679e-497">In the **Add Razor Pages using Entity Framework (CRUD)** dialog:</span></span>
  * <span data-ttu-id="2679e-498">Nell'elenco a discesa **Classe modello** selezionare **Student (ContosoUniversity.Models)**.</span><span class="sxs-lookup"><span data-stu-id="2679e-498">In the **Model class** drop-down, select **Student (ContosoUniversity.Models)**.</span></span>
  * <span data-ttu-id="2679e-499">Nella riga **Classe contesto di dati** selezionare il segno più **+**.</span><span class="sxs-lookup"><span data-stu-id="2679e-499">In the **Data context class** row, select the **+** (plus) sign.</span></span>
  * <span data-ttu-id="2679e-500">Modificare il nome del contesto di dati da *ContosoUniversity.Models.ContosoUniversityContext* a *ContosoUniversity.Data.SchoolContext*.</span><span class="sxs-lookup"><span data-stu-id="2679e-500">Change the data context name from *ContosoUniversity.Models.ContosoUniversityContext* to *ContosoUniversity.Data.SchoolContext*.</span></span>
  * <span data-ttu-id="2679e-501">Selezionare **Aggiungi**.</span><span class="sxs-lookup"><span data-stu-id="2679e-501">Select **Add**.</span></span>

<span data-ttu-id="2679e-502">Vengono installati automaticamente i pacchetti seguenti:</span><span class="sxs-lookup"><span data-stu-id="2679e-502">The following packages are automatically installed:</span></span>

* `Microsoft.VisualStudio.Web.CodeGeneration.Design`
* `Microsoft.EntityFrameworkCore.SqlServer`
* `Microsoft.Extensions.Logging.Debug`
* `Microsoft.EntityFrameworkCore.Tools`

# <a name="visual-studio-code"></a>[<span data-ttu-id="2679e-503">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="2679e-503">Visual Studio Code</span></span>](#tab/visual-studio-code)

* <span data-ttu-id="2679e-504">Eseguire i comandi seguenti dell'interfaccia della riga di comando di .NET Core per installare i pacchetti NuGet necessari:</span><span class="sxs-lookup"><span data-stu-id="2679e-504">Run the following .NET Core CLI commands to install required NuGet packages:</span></span>
<!-- TO DO  After testing, Replace with
[!INCLUDE[](~/includes/includes/add-EF-NuGet-SQLite-CLI.md)]
remove dotnet tool install --global  below
 -->
  ```dotnetcli
  dotnet add package Microsoft.EntityFrameworkCore.SQLite
  dotnet add package Microsoft.EntityFrameworkCore.SqlServer
  dotnet add package Microsoft.EntityFrameworkCore.Design
  dotnet add package Microsoft.EntityFrameworkCore.Tools
  dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
  dotnet add package Microsoft.Extensions.Logging.Debug
  ```

  <span data-ttu-id="2679e-505">Per lo scaffolding è necessario il pacchetto Microsoft.VisualStudio.Web.CodeGeneration.Design.</span><span class="sxs-lookup"><span data-stu-id="2679e-505">The Microsoft.VisualStudio.Web.CodeGeneration.Design package is required for scaffolding.</span></span> <span data-ttu-id="2679e-506">Sebbene l'app non usi SQL Server, lo strumento di scaffolding richiede il pacchetto di SQL Server.</span><span class="sxs-lookup"><span data-stu-id="2679e-506">Although the app won't use SQL Server, the scaffolding tool needs the SQL Server package.</span></span>

* <span data-ttu-id="2679e-507">Creare una cartella *Pages/Students*.</span><span class="sxs-lookup"><span data-stu-id="2679e-507">Create a *Pages/Students* folder.</span></span>

* <span data-ttu-id="2679e-508">Eseguire il comando seguente per installare lo [strumento di scaffolding aspnet-codegenerator](xref:fundamentals/tools/dotnet-aspnet-codegenerator).</span><span class="sxs-lookup"><span data-stu-id="2679e-508">Run the following command to install the [aspnet-codegenerator scaffolding tool](xref:fundamentals/tools/dotnet-aspnet-codegenerator).</span></span>

  ```dotnetcli
  dotnet tool install --global dotnet-aspnet-codegenerator
  ```

* <span data-ttu-id="2679e-509">Eseguire il comando seguente per eseguire lo scaffolding delle pagine Student.</span><span class="sxs-lookup"><span data-stu-id="2679e-509">Run the following command to scaffold Student pages.</span></span>

  <span data-ttu-id="2679e-510">**In Windows**</span><span class="sxs-lookup"><span data-stu-id="2679e-510">**On Windows**</span></span>

  ```dotnetcli
  dotnet aspnet-codegenerator razorpage -m Student -dc ContosoUniversity.Data.SchoolContext -udl -outDir Pages\Students --referenceScriptLibraries
  ```

  <span data-ttu-id="2679e-511">**In macOS o Linux**</span><span class="sxs-lookup"><span data-stu-id="2679e-511">**On macOS or Linux**</span></span>

  ```dotnetcli
  dotnet aspnet-codegenerator razorpage -m Student -dc ContosoUniversity.Data.SchoolContext -udl -outDir Pages/Students --referenceScriptLibraries
  ```

---

<span data-ttu-id="2679e-512">Se si riscontra un problema con il passaggio precedente, compilare il progetto e riprovare a eseguire il passaggio di scaffolding.</span><span class="sxs-lookup"><span data-stu-id="2679e-512">If you have a problem with the preceding step, build the project and retry the scaffold step.</span></span>

<span data-ttu-id="2679e-513">Il processo di scaffolding:</span><span class="sxs-lookup"><span data-stu-id="2679e-513">The scaffolding process:</span></span>

* <span data-ttu-id="2679e-514">Crea Razor pagine nella cartella *pages/students* :</span><span class="sxs-lookup"><span data-stu-id="2679e-514">Creates Razor pages in the *Pages/Students* folder:</span></span>
  * <span data-ttu-id="2679e-515">*Create.cshtml* e *Create.cshtml.cs*</span><span class="sxs-lookup"><span data-stu-id="2679e-515">*Create.cshtml* and *Create.cshtml.cs*</span></span>
  * <span data-ttu-id="2679e-516">*Delete.cshtml* e *Delete.cshtml.cs*</span><span class="sxs-lookup"><span data-stu-id="2679e-516">*Delete.cshtml* and *Delete.cshtml.cs*</span></span>
  * <span data-ttu-id="2679e-517">*Details.cshtml* e *Details.cshtml.cs*</span><span class="sxs-lookup"><span data-stu-id="2679e-517">*Details.cshtml* and *Details.cshtml.cs*</span></span>
  * <span data-ttu-id="2679e-518">*Edit.cshtml* e *Edit.cshtml.cs*</span><span class="sxs-lookup"><span data-stu-id="2679e-518">*Edit.cshtml* and *Edit.cshtml.cs*</span></span>
  * <span data-ttu-id="2679e-519">*Index.cshtml* e *Index.cshtml.cs*</span><span class="sxs-lookup"><span data-stu-id="2679e-519">*Index.cshtml* and *Index.cshtml.cs*</span></span>
* <span data-ttu-id="2679e-520">Crea *Data/SchoolContext.cs*.</span><span class="sxs-lookup"><span data-stu-id="2679e-520">Creates *Data/SchoolContext.cs*.</span></span>
* <span data-ttu-id="2679e-521">Aggiunge il contesto all'inserimento delle dipendenze in *Startup.cs*.</span><span class="sxs-lookup"><span data-stu-id="2679e-521">Adds the context to dependency injection in *Startup.cs*.</span></span>
* <span data-ttu-id="2679e-522">Aggiunge una stringa di connessione al database a *appsettings.json* .</span><span class="sxs-lookup"><span data-stu-id="2679e-522">Adds a database connection string to *appsettings.json*.</span></span>

## <a name="database-connection-string"></a><span data-ttu-id="2679e-523">Stringa di connessione al database</span><span class="sxs-lookup"><span data-stu-id="2679e-523">Database connection string</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="2679e-524">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="2679e-524">Visual Studio</span></span>](#tab/visual-studio)

<span data-ttu-id="2679e-525">Il *appsettings.json* file specifica la stringa di connessione [SQL Server database locale](/sql/database-engine/configure-windows/sql-server-2016-express-localdb).</span><span class="sxs-lookup"><span data-stu-id="2679e-525">The *appsettings.json* file specifies the connection string [SQL Server LocalDB](/sql/database-engine/configure-windows/sql-server-2016-express-localdb).</span></span>

[!code-json[Main](intro/samples/cu30/appsettings.json?highlight=11)]

<span data-ttu-id="2679e-526">Local DB è una versione leggera del motore di database di SQL Server Express appositamente pensato per lo sviluppo di app e non per la produzione.</span><span class="sxs-lookup"><span data-stu-id="2679e-526">LocalDB is a lightweight version of the SQL Server Express Database Engine and is intended for app development, not production use.</span></span> <span data-ttu-id="2679e-527">Per impostazione predefinita, Local DB crea file con estensione *mdf* nella directory `C:/Users/<user>`.</span><span class="sxs-lookup"><span data-stu-id="2679e-527">By default, LocalDB creates *.mdf* files in the `C:/Users/<user>` directory.</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="2679e-528">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="2679e-528">Visual Studio Code</span></span>](#tab/visual-studio-code)

<span data-ttu-id="2679e-529">Modificare la stringa di connessione in modo che punti a un file di database SQLite denominato *CU.db*:</span><span class="sxs-lookup"><span data-stu-id="2679e-529">Change the connection string to point to a SQLite database file named *CU.db*:</span></span>

[!code-json[Main](intro/samples/cu30/appsettingsSQLite.json?highlight=11)]

---

## <a name="update-the-database-context-class"></a><span data-ttu-id="2679e-530">Aggiornare la classe del contesto di database</span><span class="sxs-lookup"><span data-stu-id="2679e-530">Update the database context class</span></span>

<span data-ttu-id="2679e-531">La classe principale che coordina le funzionalità di EF Core per un determinato modello di dati è la classe del contesto di database.</span><span class="sxs-lookup"><span data-stu-id="2679e-531">The main class that coordinates EF Core functionality for a given data model is the database context class.</span></span> <span data-ttu-id="2679e-532">Il contesto è derivato da [Microsoft.EntityFrameworkCore.DbContext](/dotnet/api/microsoft.entityframeworkcore.dbcontext).</span><span class="sxs-lookup"><span data-stu-id="2679e-532">The context is derived from [Microsoft.EntityFrameworkCore.DbContext](/dotnet/api/microsoft.entityframeworkcore.dbcontext).</span></span> <span data-ttu-id="2679e-533">Il contesto specifica le entità incluse nel modello di dati.</span><span class="sxs-lookup"><span data-stu-id="2679e-533">The context specifies which entities are included in the data model.</span></span> <span data-ttu-id="2679e-534">In questo progetto la classe è denominata `SchoolContext`.</span><span class="sxs-lookup"><span data-stu-id="2679e-534">In this project, the class is named `SchoolContext`.</span></span>

<span data-ttu-id="2679e-535">Aggiornare *Data/SchoolContext.cs* con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="2679e-535">Update *Data/SchoolContext.cs* with the following code:</span></span>

[!code-csharp[Main](intro/samples/cu30snapshots/1-intro/Data/SchoolContext.cs?highlight=13-22)]

<span data-ttu-id="2679e-536">Il codice evidenziato crea una proprietà [DbSet \<TEntity> ](/dotnet/api/microsoft.entityframeworkcore.dbset-1) per ogni set di entità.</span><span class="sxs-lookup"><span data-stu-id="2679e-536">The highlighted code creates a [DbSet\<TEntity>](/dotnet/api/microsoft.entityframeworkcore.dbset-1) property for each entity set.</span></span> <span data-ttu-id="2679e-537">Nella terminologia di Entity Framework Core:</span><span class="sxs-lookup"><span data-stu-id="2679e-537">In EF Core terminology:</span></span>

* <span data-ttu-id="2679e-538">Un set di entità corrisponde in genere alla tabella di un database.</span><span class="sxs-lookup"><span data-stu-id="2679e-538">An entity set typically corresponds to a database table.</span></span>
* <span data-ttu-id="2679e-539">Un'entità corrisponde a una riga nella tabella.</span><span class="sxs-lookup"><span data-stu-id="2679e-539">An entity corresponds to a row in the table.</span></span>

<span data-ttu-id="2679e-540">Dato che un set di entità contiene più entità, le proprietà DBSet devono essere nomi plurali.</span><span class="sxs-lookup"><span data-stu-id="2679e-540">Since an entity set contains multiple entities, the DBSet properties should be plural names.</span></span> <span data-ttu-id="2679e-541">Dato che lo strumento di scaffolding ha creato un DBSet `Student`, questo passaggio lo modifica nel plurale `Students`.</span><span class="sxs-lookup"><span data-stu-id="2679e-541">Since the scaffolding tool created a`Student` DBSet, this step changes it to plural `Students`.</span></span> 

<span data-ttu-id="2679e-542">Per fare in modo che il Razor codice delle pagine corrisponda al nuovo nome del DBSet, apportare una modifica globale nell'intero progetto di `_context.Student` a `_context.Students` .</span><span class="sxs-lookup"><span data-stu-id="2679e-542">To make the Razor Pages code match the new DBSet name, make a global change across the whole project of `_context.Student` to `_context.Students`.</span></span>  <span data-ttu-id="2679e-543">Sono presenti 8 occorrenze.</span><span class="sxs-lookup"><span data-stu-id="2679e-543">There are 8 occurrences.</span></span>

<span data-ttu-id="2679e-544">Compilare il progetto per verificare che non siano presenti errori di compilazione.</span><span class="sxs-lookup"><span data-stu-id="2679e-544">Build the project to verify there are no compiler errors.</span></span>

## <a name="startupcs"></a><span data-ttu-id="2679e-545">Startup.cs</span><span class="sxs-lookup"><span data-stu-id="2679e-545">Startup.cs</span></span>

<span data-ttu-id="2679e-546">ASP.NET Core viene compilato tramite [dependency injection](xref:fundamentals/dependency-injection).</span><span class="sxs-lookup"><span data-stu-id="2679e-546">ASP.NET Core is built with [dependency injection](xref:fundamentals/dependency-injection).</span></span> <span data-ttu-id="2679e-547">I servizi, ad esempio il contesto di database di EF Core, vengono registrati con l'inserimento delle dipendenze durante l'avvio dell'applicazione.</span><span class="sxs-lookup"><span data-stu-id="2679e-547">Services (such as the EF Core database context) are registered with dependency injection during application startup.</span></span> <span data-ttu-id="2679e-548">I componenti che richiedono questi servizi, ad esempio Razor le pagine, vengono forniti tramite i parametri del costruttore.</span><span class="sxs-lookup"><span data-stu-id="2679e-548">Components that require these services (such as Razor Pages) are provided these services via constructor parameters.</span></span> <span data-ttu-id="2679e-549">Più avanti nell'esercitazione viene illustrato il codice del costruttore che ottiene un'istanza del contesto di database.</span><span class="sxs-lookup"><span data-stu-id="2679e-549">The constructor code that gets a database context instance is shown later in the tutorial.</span></span>

<span data-ttu-id="2679e-550">Lo strumento di scaffolding ha registrato automaticamente la classe di contesto nel contenitore di inserimento delle dipendenze.</span><span class="sxs-lookup"><span data-stu-id="2679e-550">The scaffolding tool automatically registered the context class with the dependency injection container.</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="2679e-551">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="2679e-551">Visual Studio</span></span>](#tab/visual-studio)

* <span data-ttu-id="2679e-552">In `ConfigureServices` le righe evidenziate sono state aggiunte dallo scaffolder:</span><span class="sxs-lookup"><span data-stu-id="2679e-552">In `ConfigureServices`, the highlighted lines were added by the scaffolder:</span></span>

  [!code-csharp[Main](intro/samples/cu30/Startup.cs?name=snippet_ConfigureServices&highlight=5-6)]

# <a name="visual-studio-code"></a>[<span data-ttu-id="2679e-553">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="2679e-553">Visual Studio Code</span></span>](#tab/visual-studio-code)

* <span data-ttu-id="2679e-554">In `ConfigureServices` assicurarsi che il codice aggiunto dallo scaffolder chiami `UseSqlite`.</span><span class="sxs-lookup"><span data-stu-id="2679e-554">In `ConfigureServices`, make sure the code added by the scaffolder calls `UseSqlite`.</span></span>

  [!code-csharp[Main](intro/samples/cu30/StartupSQLite.cs?name=snippet_ConfigureServices&highlight=5-6)]

---

<span data-ttu-id="2679e-555">Il nome della stringa di connessione viene passato al contesto chiamando un metodo in un oggetto [DbContextOptions](/dotnet/api/microsoft.entityframeworkcore.dbcontextoptions).</span><span class="sxs-lookup"><span data-stu-id="2679e-555">The name of the connection string is passed in to the context by calling a method on a [DbContextOptions](/dotnet/api/microsoft.entityframeworkcore.dbcontextoptions) object.</span></span> <span data-ttu-id="2679e-556">Per lo sviluppo locale, il [sistema di configurazione ASP.NET Core](xref:fundamentals/configuration/index) legge la stringa di connessione dal *appsettings.json* file.</span><span class="sxs-lookup"><span data-stu-id="2679e-556">For local development, the [ASP.NET Core configuration system](xref:fundamentals/configuration/index) reads the connection string from the *appsettings.json* file.</span></span>

## <a name="create-the-database"></a><span data-ttu-id="2679e-557">Creare il database</span><span class="sxs-lookup"><span data-stu-id="2679e-557">Create the database</span></span>

<span data-ttu-id="2679e-558">Aggiornare *Program.cs* per creare il database se non esiste:</span><span class="sxs-lookup"><span data-stu-id="2679e-558">Update *Program.cs* to create the database if it doesn't exist:</span></span>

[!code-csharp[Main](intro/samples/cu30snapshots/1-intro/Program.cs?highlight=1-2,14-18,21-38)]

<span data-ttu-id="2679e-559">Il metodo [EnsureCreated](/dotnet/api/microsoft.entityframeworkcore.infrastructure.databasefacade.ensurecreated#Microsoft_EntityFrameworkCore_Infrastructure_DatabaseFacade_EnsureCreated) non esegue alcuna azione se esiste un database per il contesto.</span><span class="sxs-lookup"><span data-stu-id="2679e-559">The [EnsureCreated](/dotnet/api/microsoft.entityframeworkcore.infrastructure.databasefacade.ensurecreated#Microsoft_EntityFrameworkCore_Infrastructure_DatabaseFacade_EnsureCreated) method takes no action if a database for the context exists.</span></span> <span data-ttu-id="2679e-560">Se non esiste alcun database, vengono creati il database e lo schema.</span><span class="sxs-lookup"><span data-stu-id="2679e-560">If no database exists, it creates the database and schema.</span></span> <span data-ttu-id="2679e-561">`EnsureCreated` abilita il flusso di lavoro seguente per la gestione delle modifiche al modello di dati:</span><span class="sxs-lookup"><span data-stu-id="2679e-561">`EnsureCreated` enables the following workflow for handling data model changes:</span></span>

* <span data-ttu-id="2679e-562">Eliminare il database.</span><span class="sxs-lookup"><span data-stu-id="2679e-562">Delete the database.</span></span> <span data-ttu-id="2679e-563">Tutti i dati esistenti andranno perduti.</span><span class="sxs-lookup"><span data-stu-id="2679e-563">Any existing data is lost.</span></span>
* <span data-ttu-id="2679e-564">Modificare il modello di dati.</span><span class="sxs-lookup"><span data-stu-id="2679e-564">Change the data model.</span></span> <span data-ttu-id="2679e-565">Ad esempio, aggiungere un campo `EmailAddress`.</span><span class="sxs-lookup"><span data-stu-id="2679e-565">For example, add an `EmailAddress` field.</span></span>
* <span data-ttu-id="2679e-566">Eseguire l'app.</span><span class="sxs-lookup"><span data-stu-id="2679e-566">Run the app.</span></span>
* <span data-ttu-id="2679e-567">`EnsureCreated` crea un database con il nuovo schema.</span><span class="sxs-lookup"><span data-stu-id="2679e-567">`EnsureCreated` creates a database with the new schema.</span></span>

<span data-ttu-id="2679e-568">Questo flusso di lavoro funziona correttamente nelle fasi iniziali dello sviluppo quando lo schema è in rapida evoluzione, purché non sia necessario conservare i dati.</span><span class="sxs-lookup"><span data-stu-id="2679e-568">This workflow works well early in development when the schema is rapidly evolving, as long as you don't need to preserve data.</span></span> <span data-ttu-id="2679e-569">La situazione è diversa quando è necessario mantenere i dati immessi nel database.</span><span class="sxs-lookup"><span data-stu-id="2679e-569">The situation is different when data that has been entered into the database needs to be preserved.</span></span> <span data-ttu-id="2679e-570">In tal caso, usare le migrazioni.</span><span class="sxs-lookup"><span data-stu-id="2679e-570">When that is the case, use migrations.</span></span>

<span data-ttu-id="2679e-571">Più avanti nella serie di esercitazioni si vedrà come eliminare il database creato da `EnsureCreated` e usare invece le migrazioni.</span><span class="sxs-lookup"><span data-stu-id="2679e-571">Later in the tutorial series, you delete the database that was created by `EnsureCreated` and use migrations instead.</span></span> <span data-ttu-id="2679e-572">Non è possibile aggiornare un database creato da `EnsureCreated` usando le migrazioni.</span><span class="sxs-lookup"><span data-stu-id="2679e-572">A database that is created by `EnsureCreated` can't be updated by using migrations.</span></span>

### <a name="test-the-app"></a><span data-ttu-id="2679e-573">Testare l'app</span><span class="sxs-lookup"><span data-stu-id="2679e-573">Test the app</span></span>

* <span data-ttu-id="2679e-574">Eseguire l'app.</span><span class="sxs-lookup"><span data-stu-id="2679e-574">Run the app.</span></span>
* <span data-ttu-id="2679e-575">Selezionare il collegamento **Students** (Studenti) e quindi **Crea nuovo**.</span><span class="sxs-lookup"><span data-stu-id="2679e-575">Select the **Students** link and then **Create New**.</span></span>
* <span data-ttu-id="2679e-576">Eseguire il test dei collegamenti Edit (Modifica), Details (Dettagli) e Delete (Elimina).</span><span class="sxs-lookup"><span data-stu-id="2679e-576">Test the Edit, Details, and Delete links.</span></span>

## <a name="seed-the-database"></a><span data-ttu-id="2679e-577">Specificare il valore di inizializzazione del database</span><span class="sxs-lookup"><span data-stu-id="2679e-577">Seed the database</span></span>

<span data-ttu-id="2679e-578">Il metodo `EnsureCreated` crea un database vuoto.</span><span class="sxs-lookup"><span data-stu-id="2679e-578">The `EnsureCreated` method creates an empty database.</span></span> <span data-ttu-id="2679e-579">In questa sezione viene aggiunto il codice che popola il database con dati di test.</span><span class="sxs-lookup"><span data-stu-id="2679e-579">This section adds code that populates the database with test data.</span></span>

<span data-ttu-id="2679e-580">Creare *Data/DbInitializer.cs* con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="2679e-580">Create *Data/DbInitializer.cs* with the following code:</span></span>
<!-- next update, keep this file in the project and surround with #if -->
  [!code-csharp[Main](intro/samples/cu30snapshots/1-intro/Data/DbInitializer.cs)]

  <span data-ttu-id="2679e-581">Il codice controlla se esistono studenti nel database.</span><span class="sxs-lookup"><span data-stu-id="2679e-581">The code checks if there are any students in the database.</span></span> <span data-ttu-id="2679e-582">Se non sono presenti studenti, aggiunge i dati di test al database.</span><span class="sxs-lookup"><span data-stu-id="2679e-582">If there are no students, it adds test data to the database.</span></span> <span data-ttu-id="2679e-583">I dati di test vengono creati in matrici anziché in raccolte `List<T>` per ottimizzare le prestazioni.</span><span class="sxs-lookup"><span data-stu-id="2679e-583">It creates the test data in arrays rather than `List<T>` collections to optimize performance.</span></span>

* <span data-ttu-id="2679e-584">In *Program.cs* sostituire la chiamata di `EnsureCreated` con una chiamata di `DbInitializer.Initialize`:</span><span class="sxs-lookup"><span data-stu-id="2679e-584">In *Program.cs*, replace the `EnsureCreated` call with a `DbInitializer.Initialize` call:</span></span>

  ```csharp
  // context.Database.EnsureCreated();
  DbInitializer.Initialize(context);
  ```

# <a name="visual-studio"></a>[<span data-ttu-id="2679e-585">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="2679e-585">Visual Studio</span></span>](#tab/visual-studio)

<span data-ttu-id="2679e-586">Arrestare l'app se è in esecuzione ed eseguire il comando seguente nella **console di Gestione pacchetti**:</span><span class="sxs-lookup"><span data-stu-id="2679e-586">Stop the app if it's running, and run the following command in the **Package Manager Console** (PMC):</span></span>

```powershell
Drop-Database
```

# <a name="visual-studio-code"></a>[<span data-ttu-id="2679e-587">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="2679e-587">Visual Studio Code</span></span>](#tab/visual-studio-code)

* <span data-ttu-id="2679e-588">Arrestare l'app se è in esecuzione ed eliminare il file *CU.db*.</span><span class="sxs-lookup"><span data-stu-id="2679e-588">Stop the app if it's running, and delete the *CU.db* file.</span></span>

---

* <span data-ttu-id="2679e-589">Riavviare l'app.</span><span class="sxs-lookup"><span data-stu-id="2679e-589">Restart the app.</span></span>

* <span data-ttu-id="2679e-590">Selezionare la pagina Students per visualizzare i dati di seeding.</span><span class="sxs-lookup"><span data-stu-id="2679e-590">Select the Students page to see the seeded data.</span></span>

## <a name="view-the-database"></a><span data-ttu-id="2679e-591">Visualizzare il database</span><span class="sxs-lookup"><span data-stu-id="2679e-591">View the database</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="2679e-592">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="2679e-592">Visual Studio</span></span>](#tab/visual-studio)

* <span data-ttu-id="2679e-593">Aprire **Esplora oggetti di SQL Server** dal menu **Visualizza** in Visual Studio.</span><span class="sxs-lookup"><span data-stu-id="2679e-593">Open **SQL Server Object Explorer** (SSOX) from the **View** menu in Visual Studio.</span></span>
* <span data-ttu-id="2679e-594">In Esplora oggetti di SQL Server selezionare **(localdb)\MSSQLLocalDB > Database > SchoolContext-{GUID}**.</span><span class="sxs-lookup"><span data-stu-id="2679e-594">In SSOX, select **(localdb)\MSSQLLocalDB > Databases > SchoolContext-{GUID}**.</span></span> <span data-ttu-id="2679e-595">Il nome del database viene generato dal nome del contesto specificato in precedenza con l'aggiunta di un trattino e un GUID.</span><span class="sxs-lookup"><span data-stu-id="2679e-595">The database name is generated from the context name you provided earlier plus a dash and a GUID.</span></span>
* <span data-ttu-id="2679e-596">Espandere il nodo **Tabelle**.</span><span class="sxs-lookup"><span data-stu-id="2679e-596">Expand the **Tables** node.</span></span>
* <span data-ttu-id="2679e-597">Fare clic con il pulsante destro del mouse sulla tabella **Student** (Studente) e fare clic su **Visualizza dati** per visualizzare le colonne create e le righe inserite nella tabella.</span><span class="sxs-lookup"><span data-stu-id="2679e-597">Right-click the **Student** table and click **View Data** to see the columns created and the rows inserted into the table.</span></span>
* <span data-ttu-id="2679e-598">Fare clic con il pulsante destro del mouse sulla tabella **Student** e scegliere **Visualizza codice** per vedere il mapping tra il modello `Student` e lo schema della tabella `Student`.</span><span class="sxs-lookup"><span data-stu-id="2679e-598">Right-click the **Student** table and click **View Code** to see how the `Student` model maps to the `Student` table schema.</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="2679e-599">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="2679e-599">Visual Studio Code</span></span>](#tab/visual-studio-code)

<span data-ttu-id="2679e-600">Usare lo strumento SQLite per visualizzare lo schema del database e i dati di seeding.</span><span class="sxs-lookup"><span data-stu-id="2679e-600">Use your SQLite tool to view the database schema and seeded data.</span></span> <span data-ttu-id="2679e-601">Il file di database è denominato *CU.db* e si trova nella cartella del progetto.</span><span class="sxs-lookup"><span data-stu-id="2679e-601">The database file is named *CU.db* and is located in the project folder.</span></span>

---

## <a name="asynchronous-code"></a><span data-ttu-id="2679e-602">Codice asincrono</span><span class="sxs-lookup"><span data-stu-id="2679e-602">Asynchronous code</span></span>

<span data-ttu-id="2679e-603">La programmazione asincrona è la modalità predefinita per ASP.NET Core ed Entity Framework Core.</span><span class="sxs-lookup"><span data-stu-id="2679e-603">Asynchronous programming is the default mode for ASP.NET Core and EF Core.</span></span>

<span data-ttu-id="2679e-604">Per un server Web è disponibile un numero limitato di thread e in situazioni di carico elevato tutti i thread disponibili potrebbero essere in uso.</span><span class="sxs-lookup"><span data-stu-id="2679e-604">A web server has a limited number of threads available, and in high load situations all of the available threads might be in use.</span></span> <span data-ttu-id="2679e-605">In queste circostanze il server non può elaborare nuove richieste finché i thread non saranno liberi.</span><span class="sxs-lookup"><span data-stu-id="2679e-605">When that happens, the server can't process new requests until the threads are freed up.</span></span> <span data-ttu-id="2679e-606">Con il codice sincrono, può succedere che molti thread siano vincolati nonostante in quel momento non stiano eseguendo alcuna operazione. Rimangono tuttavia in attesa che l'operazione I/O sia completata.</span><span class="sxs-lookup"><span data-stu-id="2679e-606">With synchronous code, many threads may be tied up while they aren't actually doing any work because they're waiting for I/O to complete.</span></span> <span data-ttu-id="2679e-607">Con il codice asincrono, se un processo è in attesa del completamento dell'operazione I/O, il thread viene liberato e il server lo può usare per l'elaborazione di altre richieste.</span><span class="sxs-lookup"><span data-stu-id="2679e-607">With asynchronous code, when a process is waiting for I/O to complete, its thread is freed up for the server to use for processing other requests.</span></span> <span data-ttu-id="2679e-608">Di conseguenza, il codice asincrono consente un uso più efficiente delle risorse del server e il server può gestire una maggiore quantità di traffico senza ritardi.</span><span class="sxs-lookup"><span data-stu-id="2679e-608">As a result, asynchronous code enables server resources to be used more efficiently, and the server can handle more traffic without delays.</span></span>

<span data-ttu-id="2679e-609">Il codice asincrono comporta un minimo sovraccarico in fase di esecuzione.</span><span class="sxs-lookup"><span data-stu-id="2679e-609">Asynchronous code does introduce a small amount of overhead at run time.</span></span> <span data-ttu-id="2679e-610">In caso di traffico ridotto, il calo delle prestazioni è irrilevante, mentre nelle situazioni di traffico elevato, è essenziale ottimizzare le prestazioni potenziali.</span><span class="sxs-lookup"><span data-stu-id="2679e-610">For low traffic situations, the performance hit is negligible, while for high traffic situations, the potential performance improvement is substantial.</span></span>

<span data-ttu-id="2679e-611">Nel codice seguente la parola chiave [async](/dotnet/csharp/language-reference/keywords/async), il valore restituito `Task<T>`, la parola chiave `await` e il metodo `ToListAsync` consentono di eseguire il codice in modo asincrono.</span><span class="sxs-lookup"><span data-stu-id="2679e-611">In the following code, the [async](/dotnet/csharp/language-reference/keywords/async) keyword, `Task<T>` return value, `await` keyword, and `ToListAsync` method make the code execute asynchronously.</span></span>

```csharp
public async Task OnGetAsync()
{
    Students = await _context.Students.ToListAsync();
}
```

* <span data-ttu-id="2679e-612">La parola chiave `async` indica al compilatore di eseguire le operazioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="2679e-612">The `async` keyword tells the compiler to:</span></span>
  * <span data-ttu-id="2679e-613">Generare callback per parti del corpo del metodo.</span><span class="sxs-lookup"><span data-stu-id="2679e-613">Generate callbacks for parts of the method body.</span></span>
  * <span data-ttu-id="2679e-614">Creare l'oggetto [Task](/dotnet/csharp/programming-guide/concepts/async/async-return-types#BKMK_TaskReturnType) restituito.</span><span class="sxs-lookup"><span data-stu-id="2679e-614">Create the [Task](/dotnet/csharp/programming-guide/concepts/async/async-return-types#BKMK_TaskReturnType) object that's returned.</span></span>
* <span data-ttu-id="2679e-615">Il tipo restituito `Task<T>` rappresenta il lavoro in corso.</span><span class="sxs-lookup"><span data-stu-id="2679e-615">The `Task<T>` return type represents ongoing work.</span></span>
* <span data-ttu-id="2679e-616">La parola chiave `await` indica al compilatore di suddividere il metodo in due parti.</span><span class="sxs-lookup"><span data-stu-id="2679e-616">The `await` keyword causes the compiler to split the method into two parts.</span></span> <span data-ttu-id="2679e-617">La prima parte termina con l'operazione avviata in modo asincrono.</span><span class="sxs-lookup"><span data-stu-id="2679e-617">The first part ends with the operation that's started asynchronously.</span></span> <span data-ttu-id="2679e-618">La seconda parte viene inserita in un metodo di callback che viene chiamato al termine dell'operazione.</span><span class="sxs-lookup"><span data-stu-id="2679e-618">The second part is put into a callback method that's called when the operation completes.</span></span>
* <span data-ttu-id="2679e-619">`ToListAsync` è la versione asincrona del metodo di estensione `ToList`.</span><span class="sxs-lookup"><span data-stu-id="2679e-619">`ToListAsync` is the asynchronous version of the `ToList` extension method.</span></span>

<span data-ttu-id="2679e-620">Alcuni aspetti da considerare quando si scrive codice asincrono usato da Entity Framework Core:</span><span class="sxs-lookup"><span data-stu-id="2679e-620">Some things to be aware of when writing asynchronous code that uses EF Core:</span></span>

* <span data-ttu-id="2679e-621">Solo le istruzioni che generano query o comandi da inviare al database vengono eseguite in modo asincrono,</span><span class="sxs-lookup"><span data-stu-id="2679e-621">Only statements that cause queries or commands to be sent to the database are executed asynchronously.</span></span> <span data-ttu-id="2679e-622">Sono incluse `ToListAsync`, `SingleOrDefaultAsync`, `FirstOrDefaultAsync` e `SaveChangesAsync`.</span><span class="sxs-lookup"><span data-stu-id="2679e-622">That includes `ToListAsync`, `SingleOrDefaultAsync`, `FirstOrDefaultAsync`, and `SaveChangesAsync`.</span></span> <span data-ttu-id="2679e-623">Non sono incluse le istruzioni che modificano solo un oggetto `IQueryable`, ad esempio `var students = context.Students.Where(s => s.LastName == "Davolio")`.</span><span class="sxs-lookup"><span data-stu-id="2679e-623">It doesn't include statements that just change an `IQueryable`, such as `var students = context.Students.Where(s => s.LastName == "Davolio")`.</span></span>
* <span data-ttu-id="2679e-624">Un contesto di Entity Framework Core non è thread-safe. Non tentare di eseguire più operazioni parallelamente.</span><span class="sxs-lookup"><span data-stu-id="2679e-624">An EF Core context isn't thread safe: don't try to do multiple operations in parallel.</span></span>
* <span data-ttu-id="2679e-625">Per sfruttare i vantaggi del codice asincrono in termini di prestazioni, verificare che i pacchetti della libreria impiegati, ad esempio per il paging, usino la modalità asincrona per chiamare i metodi di EF Core che inviano query al database.</span><span class="sxs-lookup"><span data-stu-id="2679e-625">To take advantage of the performance benefits of async code, verify that library packages (such as for paging) use async if they call EF Core methods that send queries to the database.</span></span>

<span data-ttu-id="2679e-626">Per altre informazioni sulla programmazione asincrona in .NET, vedere [Panoramica della programmazione asincrona](/dotnet/standard/async) e [Programmazione asincrona con async e await](/dotnet/csharp/programming-guide/concepts/async/).</span><span class="sxs-lookup"><span data-stu-id="2679e-626">For more information about asynchronous programming in .NET, see [Async Overview](/dotnet/standard/async) and [Asynchronous programming with async and await](/dotnet/csharp/programming-guide/concepts/async/).</span></span>

## <a name="next-steps"></a><span data-ttu-id="2679e-627">Passaggi successivi</span><span class="sxs-lookup"><span data-stu-id="2679e-627">Next steps</span></span>

> [!div class="step-by-step"]
> [<span data-ttu-id="2679e-628">Esercitazione successiva</span><span class="sxs-lookup"><span data-stu-id="2679e-628">Next tutorial</span></span>](xref:data/ef-rp/crud)

::: moniker-end

::: moniker range="< aspnetcore-3.0"

<span data-ttu-id="2679e-629">L'app Web di esempio di Contoso University illustra come creare un' Razor app ASP.NET Core Pages con Entity Framework (EF) Core.</span><span class="sxs-lookup"><span data-stu-id="2679e-629">The Contoso University sample web app demonstrates how to create an ASP.NET Core Razor Pages app using Entity Framework (EF) Core.</span></span>

<span data-ttu-id="2679e-630">L'app di esempio è un sito Web per una fittizia Contoso University.</span><span class="sxs-lookup"><span data-stu-id="2679e-630">The sample app is a web site for a fictional Contoso University.</span></span> <span data-ttu-id="2679e-631">Include funzionalità, come ad esempio l'ammissione di studenti, la creazione di corsi e le assegnazioni di insegnati.</span><span class="sxs-lookup"><span data-stu-id="2679e-631">It includes functionality such as student admission, course creation, and instructor assignments.</span></span> <span data-ttu-id="2679e-632">Questa pagina è il prima di una serie di esercitazioni che illustrano come compilare l'app di esempio di Contoso University.</span><span class="sxs-lookup"><span data-stu-id="2679e-632">This page is the first in a series of tutorials that explain how to build the Contoso University sample app.</span></span>

[<span data-ttu-id="2679e-633">Scaricare o visualizzare l'app completata.</span><span class="sxs-lookup"><span data-stu-id="2679e-633">Download or view the completed app.</span></span>](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/data/ef-rp/intro/samples) <span data-ttu-id="2679e-634">[Istruzioni per il download](xref:index#how-to-download-a-sample).</span><span class="sxs-lookup"><span data-stu-id="2679e-634">[Download instructions](xref:index#how-to-download-a-sample).</span></span>

## <a name="prerequisites"></a><span data-ttu-id="2679e-635">Prerequisiti</span><span class="sxs-lookup"><span data-stu-id="2679e-635">Prerequisites</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="2679e-636">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="2679e-636">Visual Studio</span></span>](#tab/visual-studio)

[!INCLUDE [](~/includes/net-core-prereqs-windows.md)]

# <a name="visual-studio-code"></a>[<span data-ttu-id="2679e-637">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="2679e-637">Visual Studio Code</span></span>](#tab/visual-studio-code)

[!INCLUDE [](~/includes/2.1-SDK.md)]

---

<span data-ttu-id="2679e-638">Familiarità con le [ Razor pagine](xref:razor-pages/index).</span><span class="sxs-lookup"><span data-stu-id="2679e-638">Familiarity with [Razor Pages](xref:razor-pages/index).</span></span> <span data-ttu-id="2679e-639">Prima di iniziare questa serie, i nuovi programmatori dovranno completare l' [Introduzione alle Razor pagine](xref:tutorials/razor-pages/razor-pages-start) .</span><span class="sxs-lookup"><span data-stu-id="2679e-639">New programmers should complete [Get started with Razor Pages](xref:tutorials/razor-pages/razor-pages-start) before starting this series.</span></span>

## <a name="troubleshooting"></a><span data-ttu-id="2679e-640">Risoluzione dei problemi</span><span class="sxs-lookup"><span data-stu-id="2679e-640">Troubleshooting</span></span>

<span data-ttu-id="2679e-641">Se si verifica un problema che non si sa come risolvere, è generalmente possibile trovare la soluzione confrontando il codice con il [progetto completato](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/data/ef-rp/intro/samples).</span><span class="sxs-lookup"><span data-stu-id="2679e-641">If you run into a problem you can't resolve, you can generally find the solution by comparing your code to the [completed project](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/data/ef-rp/intro/samples).</span></span> <span data-ttu-id="2679e-642">Un buon metodo per ottenere assistenza è quello di pubblicare una domanda in [StackOverflow.com](https://stackoverflow.com/questions/tagged/asp.net-core) per [ASP.NET Core](https://stackoverflow.com/questions/tagged/asp.net-core) o [EF Core](https://stackoverflow.com/questions/tagged/entity-framework-core).</span><span class="sxs-lookup"><span data-stu-id="2679e-642">A good way to get help is by posting a question to [StackOverflow.com](https://stackoverflow.com/questions/tagged/asp.net-core) for [ASP.NET Core](https://stackoverflow.com/questions/tagged/asp.net-core) or [EF Core](https://stackoverflow.com/questions/tagged/entity-framework-core).</span></span>

## <a name="the-contoso-university-web-app"></a><span data-ttu-id="2679e-643">App Web di Contoso University</span><span class="sxs-lookup"><span data-stu-id="2679e-643">The Contoso University web app</span></span>

<span data-ttu-id="2679e-644">L'applicazione compilata in queste esercitazioni è il sito Web di base di un'università.</span><span class="sxs-lookup"><span data-stu-id="2679e-644">The app built in these tutorials is a basic university web site.</span></span>

<span data-ttu-id="2679e-645">Gli utenti possono visualizzare e aggiornare le informazioni che riguardano studenti, corsi e insegnanti.</span><span class="sxs-lookup"><span data-stu-id="2679e-645">Users can view and update student, course, and instructor information.</span></span> <span data-ttu-id="2679e-646">Di seguito sono disponibili alcune schermate dell'esercitazione.</span><span class="sxs-lookup"><span data-stu-id="2679e-646">Here are a few of the screens created in the tutorial.</span></span>

![Pagina Student Index (Indice degli studenti)](intro/_static/students-index.png)

![Pagina Students Edit (Modifica studenti)](intro/_static/student-edit.png)

<span data-ttu-id="2679e-649">Lo stile dell'interfaccia utente del sito è simile a quanto è stato generato tramite i modelli predefiniti.</span><span class="sxs-lookup"><span data-stu-id="2679e-649">The UI style of this site is close to what's generated by the built-in templates.</span></span> <span data-ttu-id="2679e-650">Lo stato attivo dell'esercitazione è EF Core con le Razor pagine, non con l'interfaccia utente.</span><span class="sxs-lookup"><span data-stu-id="2679e-650">The tutorial focus is on EF Core with Razor Pages, not the UI.</span></span>

## <a name="create-the-contosouniversity-razor-pages-web-app"></a><span data-ttu-id="2679e-651">Creare l' Razor app Web ContosoUniversity Pages</span><span class="sxs-lookup"><span data-stu-id="2679e-651">Create the ContosoUniversity Razor Pages web app</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="2679e-652">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="2679e-652">Visual Studio</span></span>](#tab/visual-studio)

* <span data-ttu-id="2679e-653">Dal menu **File** di Visual Studio selezionare **Nuovo** > **Progetto**.</span><span class="sxs-lookup"><span data-stu-id="2679e-653">From the Visual Studio **File** menu, select **New** > **Project**.</span></span>
* <span data-ttu-id="2679e-654">Creare una nuova applicazione Web ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="2679e-654">Create a new ASP.NET Core Web Application.</span></span> <span data-ttu-id="2679e-655">Denominare il progetto **ContosoUniversity**.</span><span class="sxs-lookup"><span data-stu-id="2679e-655">Name the project **ContosoUniversity**.</span></span> <span data-ttu-id="2679e-656">È importante denominare il progetto *ContosoUniversity* in modo che gli spazi dei nomi corrispondano quando il codice viene copiato/incollato.</span><span class="sxs-lookup"><span data-stu-id="2679e-656">It's important to name the project *ContosoUniversity* so the namespaces match when code is copy/pasted.</span></span>
* <span data-ttu-id="2679e-657">Selezionare **ASP.NET Core 2.1** nell'elenco a discesa, quindi selezionare **Applicazione Web**.</span><span class="sxs-lookup"><span data-stu-id="2679e-657">Select **ASP.NET Core 2.1** in the dropdown, and then select **Web Application**.</span></span>

<span data-ttu-id="2679e-658">Per le immagini dei passaggi precedenti, vedere [creare un' Razor app Web](xref:tutorials/razor-pages/razor-pages-start#create-a-razor-pages-web-app).</span><span class="sxs-lookup"><span data-stu-id="2679e-658">For images of the preceding steps, see [Create a Razor web app](xref:tutorials/razor-pages/razor-pages-start#create-a-razor-pages-web-app).</span></span>
<span data-ttu-id="2679e-659">Eseguire l'app.</span><span class="sxs-lookup"><span data-stu-id="2679e-659">Run the app.</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="2679e-660">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="2679e-660">Visual Studio Code</span></span>](#tab/visual-studio-code)

```dotnetcli
dotnet new webapp -o ContosoUniversity
cd ContosoUniversity
dotnet run
```

---

## <a name="set-up-the-site-style"></a><span data-ttu-id="2679e-661">Impostare lo stile del sito</span><span class="sxs-lookup"><span data-stu-id="2679e-661">Set up the site style</span></span>

<span data-ttu-id="2679e-662">Con alcune modifiche è possibile impostare il menu del sito, il layout e la home page.</span><span class="sxs-lookup"><span data-stu-id="2679e-662">A few changes set up the site menu, layout, and home page.</span></span> <span data-ttu-id="2679e-663">Aggiornare *Pages/Shared/_Layout.cshtml* con le modifiche seguenti:</span><span class="sxs-lookup"><span data-stu-id="2679e-663">Update *Pages/Shared/_Layout.cshtml* with the following changes:</span></span>

* <span data-ttu-id="2679e-664">Modificare tutte le occorrenze di "ContosoUniversity" in "Contoso University".</span><span class="sxs-lookup"><span data-stu-id="2679e-664">Change each occurrence of "ContosoUniversity" to "Contoso University".</span></span> <span data-ttu-id="2679e-665">Le occorrenze sono tre.</span><span class="sxs-lookup"><span data-stu-id="2679e-665">There are three occurrences.</span></span>

* <span data-ttu-id="2679e-666">Aggiungere le voci di menu per **Students** (Studenti), **Courses** (Corsi), **Instructors** (Insegnanti) e **Departments** (Dipartimenti) ed eliminare la voce di menu **Contact** (Contatto).</span><span class="sxs-lookup"><span data-stu-id="2679e-666">Add menu entries for **Students**, **Courses**, **Instructors**, and **Departments**, and delete the **Contact** menu entry.</span></span>

<span data-ttu-id="2679e-667">Le modifiche sono evidenziate.</span><span class="sxs-lookup"><span data-stu-id="2679e-667">The changes are highlighted.</span></span> <span data-ttu-id="2679e-668">*Non* viene visualizzato l'intero markup.</span><span class="sxs-lookup"><span data-stu-id="2679e-668">(All the markup is *not* displayed.)</span></span>

[!code-cshtml[](intro/samples/cu21/Pages/Shared/_Layout.cshtml?highlight=6,29,35-38,50&name=snippet)]

<span data-ttu-id="2679e-669">In *Pages/Index.cshtml* sostituire il contenuto del file con il codice seguente. In questo modo il testo su ASP.NET e MVC sarà sostituito dal testo relativo a questa app:</span><span class="sxs-lookup"><span data-stu-id="2679e-669">In *Pages/Index.cshtml*, replace the contents of the file with the following code to replace the text about ASP.NET and MVC with text about this app:</span></span>

[!code-cshtml[](intro/samples/cu21/Pages/Index.cshtml)]

## <a name="create-the-data-model"></a><span data-ttu-id="2679e-670">Creare il modello di dati</span><span class="sxs-lookup"><span data-stu-id="2679e-670">Create the data model</span></span>

<span data-ttu-id="2679e-671">Creare le classi delle entità per l'app di Contoso University.</span><span class="sxs-lookup"><span data-stu-id="2679e-671">Create entity classes for the Contoso University app.</span></span> <span data-ttu-id="2679e-672">Iniziare con le tre entità seguenti:</span><span class="sxs-lookup"><span data-stu-id="2679e-672">Start with the following three entities:</span></span>

![Diagramma modello di dati Course/Enrollment/Student (Corso/Iscrizione/Studente)](intro/_static/data-model-diagram.png)

<span data-ttu-id="2679e-674">Esiste una relazione uno-a-molti tra le entità `Student` e `Enrollment`.</span><span class="sxs-lookup"><span data-stu-id="2679e-674">There's a one-to-many relationship between `Student` and `Enrollment` entities.</span></span> <span data-ttu-id="2679e-675">Esiste una relazione uno-a-molti tra le entità `Course` e `Enrollment`.</span><span class="sxs-lookup"><span data-stu-id="2679e-675">There's a one-to-many relationship between `Course` and `Enrollment` entities.</span></span> <span data-ttu-id="2679e-676">Uno studente può iscriversi a un numero qualsiasi di corsi.</span><span class="sxs-lookup"><span data-stu-id="2679e-676">A student can enroll in any number of courses.</span></span> <span data-ttu-id="2679e-677">Un corso può avere un numero qualsiasi di studenti iscritti.</span><span class="sxs-lookup"><span data-stu-id="2679e-677">A course can have any number of students enrolled in it.</span></span>

<span data-ttu-id="2679e-678">Nelle sezioni seguenti viene creata una classe per ognuna di queste entità.</span><span class="sxs-lookup"><span data-stu-id="2679e-678">In the following sections, a class for each one of these entities is created.</span></span>

### <a name="the-student-entity"></a><span data-ttu-id="2679e-679">Entità Student (Studente)</span><span class="sxs-lookup"><span data-stu-id="2679e-679">The Student entity</span></span>

![Diagramma entità Student (Studente)](intro/_static/student-entity.png)

<span data-ttu-id="2679e-681">Creare una cartella *Models* (Modelli).</span><span class="sxs-lookup"><span data-stu-id="2679e-681">Create a *Models* folder.</span></span> <span data-ttu-id="2679e-682">Nella cartella *Models* (Modelli) creare un file di classe denominato *Student.cs* con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="2679e-682">In the *Models* folder, create a class file named *Student.cs* with the following code:</span></span>

[!code-csharp[](intro/samples/cu21/Models/Student.cs?name=snippet_Intro)]

<span data-ttu-id="2679e-683">La proprietà `ID` diventa la colonna di chiave primaria della tabella di database (DB) che corrisponde a questa classe.</span><span class="sxs-lookup"><span data-stu-id="2679e-683">The `ID` property becomes the primary key column of the database (DB) table that corresponds to this class.</span></span> <span data-ttu-id="2679e-684">Per impostazione predefinita, Entity Framework Core interpreta una proprietà denominata `ID` o `classnameID` come chiave primaria.</span><span class="sxs-lookup"><span data-stu-id="2679e-684">By default, EF Core interprets a property that's named `ID` or `classnameID` as the primary key.</span></span> <span data-ttu-id="2679e-685">In `classnameID``classname` è il nome della classe.</span><span class="sxs-lookup"><span data-stu-id="2679e-685">In `classnameID`, `classname` is the name of the class.</span></span> <span data-ttu-id="2679e-686">La chiave primaria alternativa riconosciuta automaticamente è `StudentID` nell'esempio precedente.</span><span class="sxs-lookup"><span data-stu-id="2679e-686">The alternative automatically recognized primary key is `StudentID` in the preceding example.</span></span>

<span data-ttu-id="2679e-687">La proprietà `Enrollments` rappresenta una [proprietà di navigazione](/ef/core/modeling/relationships).</span><span class="sxs-lookup"><span data-stu-id="2679e-687">The `Enrollments` property is a [navigation property](/ef/core/modeling/relationships).</span></span> <span data-ttu-id="2679e-688">Le proprietà di navigazione si collegano ad altre entità correlate a questa entità.</span><span class="sxs-lookup"><span data-stu-id="2679e-688">Navigation properties link to other entities that are related to this entity.</span></span> <span data-ttu-id="2679e-689">In questo caso la proprietà `Enrollments` di `Student entity` contiene tutte le entità `Enrollment` correlate a `Student`.</span><span class="sxs-lookup"><span data-stu-id="2679e-689">In this case, the `Enrollments` property of a `Student entity` holds all of the `Enrollment` entities that are related to that `Student`.</span></span> <span data-ttu-id="2679e-690">Ad esempio, se una riga Student (Studente) nella database presenta due righe Enrollment (Iscrizione) correlate, la proprietà di navigazione `Enrollments` contiene questi due entità `Enrollment`.</span><span class="sxs-lookup"><span data-stu-id="2679e-690">For example, if a Student row in the DB has two related Enrollment rows, the `Enrollments` navigation property contains those two `Enrollment` entities.</span></span> <span data-ttu-id="2679e-691">Una riga `Enrollment` correlata è una riga che contiene il valore della chiave primaria dello studente nella colonna `StudentID`.</span><span class="sxs-lookup"><span data-stu-id="2679e-691">A related `Enrollment` row is a row that contains that student's primary key value in the `StudentID` column.</span></span> <span data-ttu-id="2679e-692">Si supponga ad esempio che per lo studente con ID = 1 siano presenti due righe nella tabella `Enrollment`.</span><span class="sxs-lookup"><span data-stu-id="2679e-692">For example, suppose the student with ID=1 has two rows in the `Enrollment` table.</span></span> <span data-ttu-id="2679e-693">La tabella `Enrollment` contiene due righe con `StudentID` = 1.</span><span class="sxs-lookup"><span data-stu-id="2679e-693">The `Enrollment` table has two rows with `StudentID` = 1.</span></span> <span data-ttu-id="2679e-694">`StudentID` è una chiave esterna nella tabella `Enrollment` che specifica lo studente nella tabella `Student`.</span><span class="sxs-lookup"><span data-stu-id="2679e-694">`StudentID` is a foreign key in the `Enrollment` table that specifies the student in the `Student` table.</span></span>

<span data-ttu-id="2679e-695">Se una proprietà di navigazione può contenere più entità, deve essere un tipo elenco, ad esempio `ICollection<T>`.</span><span class="sxs-lookup"><span data-stu-id="2679e-695">If a navigation property can hold multiple entities, the navigation property must be a list type, such as `ICollection<T>`.</span></span> <span data-ttu-id="2679e-696">È possibile specificare `ICollection<T>` o un tipo, ad esempio `List<T>` o `HashSet<T>`.</span><span class="sxs-lookup"><span data-stu-id="2679e-696">`ICollection<T>` can be specified, or a type such as `List<T>` or `HashSet<T>`.</span></span> <span data-ttu-id="2679e-697">Se si specifica `ICollection<T>`, per impostazione predefinita Entity Framework Core crea una raccolta `HashSet<T>`.</span><span class="sxs-lookup"><span data-stu-id="2679e-697">When `ICollection<T>` is used, EF Core creates a `HashSet<T>` collection by default.</span></span> <span data-ttu-id="2679e-698">Le proprietà di navigazione che contengono più entità provengono da relazioni molti-a-molti e uno-a-molti.</span><span class="sxs-lookup"><span data-stu-id="2679e-698">Navigation properties that hold multiple entities come from many-to-many and one-to-many relationships.</span></span>

### <a name="the-enrollment-entity"></a><span data-ttu-id="2679e-699">Entità Enrollment (Iscrizione)</span><span class="sxs-lookup"><span data-stu-id="2679e-699">The Enrollment entity</span></span>

![Diagramma entità Enrollment (Iscrizione)](intro/_static/enrollment-entity.png)

<span data-ttu-id="2679e-701">Nella cartella *Models* (Modelli) creare un file *Enrollment.cs* con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="2679e-701">In the *Models* folder, create *Enrollment.cs* with the following code:</span></span>

[!code-csharp[](intro/samples/cu21/Models/Enrollment.cs?name=snippet_Intro)]

<span data-ttu-id="2679e-702">La proprietà `EnrollmentID` è la chiave primaria.</span><span class="sxs-lookup"><span data-stu-id="2679e-702">The `EnrollmentID` property is the primary key.</span></span> <span data-ttu-id="2679e-703">Questa entità usa il criterio `classnameID` anziché `ID` come l'entità `Student`.</span><span class="sxs-lookup"><span data-stu-id="2679e-703">This entity uses the `classnameID` pattern instead of `ID` like the `Student` entity.</span></span> <span data-ttu-id="2679e-704">In genere gli sviluppatori scelgono un criterio che usano poi in tutto il modello di dati.</span><span class="sxs-lookup"><span data-stu-id="2679e-704">Typically developers choose one pattern and use it throughout the data model.</span></span> <span data-ttu-id="2679e-705">In un'esercitazione successiva viene illustrato l'uso di ID senza classname. In questo modo si semplifica l'implementazione dell'ereditarietà nel modello di dati.</span><span class="sxs-lookup"><span data-stu-id="2679e-705">In a later tutorial, using ID without classname is shown to make it easier to implement inheritance in the data model.</span></span>

<span data-ttu-id="2679e-706">La proprietà `Grade` è un oggetto`enum`.</span><span class="sxs-lookup"><span data-stu-id="2679e-706">The `Grade` property is an `enum`.</span></span> <span data-ttu-id="2679e-707">Il punto interrogativo dopo la dichiarazione del tipo `Grade` indica che la proprietà `Grade` ammette i valori Null.</span><span class="sxs-lookup"><span data-stu-id="2679e-707">The question mark after the `Grade` type declaration indicates that the `Grade` property is nullable.</span></span> <span data-ttu-id="2679e-708">Un voto il cui valore è Null è diverso da un voto con valore zero. Null significa che un voto è sconosciuto o non è stato ancora assegnato.</span><span class="sxs-lookup"><span data-stu-id="2679e-708">A grade that's null is different from a zero grade -- null means a grade isn't known or hasn't been assigned yet.</span></span>

<span data-ttu-id="2679e-709">La proprietà `StudentID` è una chiave esterna e la proprietà di navigazione corrispondente è `Student`.</span><span class="sxs-lookup"><span data-stu-id="2679e-709">The `StudentID` property is a foreign key, and the corresponding navigation property is `Student`.</span></span> <span data-ttu-id="2679e-710">Un'entità `Enrollment` è associata a un'entità `Student`, pertanto la proprietà contiene un'unica entità `Student`.</span><span class="sxs-lookup"><span data-stu-id="2679e-710">An `Enrollment` entity is associated with one `Student` entity, so the property contains a single `Student` entity.</span></span> <span data-ttu-id="2679e-711">L'entità `Student` si differenzia dalla proprietà di navigazione `Student.Enrollments` che contiene più entità `Enrollment`.</span><span class="sxs-lookup"><span data-stu-id="2679e-711">The `Student` entity differs from the `Student.Enrollments` navigation property, which contains multiple `Enrollment` entities.</span></span>

<span data-ttu-id="2679e-712">La proprietà `CourseID` è una chiave esterna e la proprietà di navigazione corrispondente è `Course`.</span><span class="sxs-lookup"><span data-stu-id="2679e-712">The `CourseID` property is a foreign key, and the corresponding navigation property is `Course`.</span></span> <span data-ttu-id="2679e-713">Un'entità `Enrollment` è associata a un'entità `Course`.</span><span class="sxs-lookup"><span data-stu-id="2679e-713">An `Enrollment` entity is associated with one `Course` entity.</span></span>

<span data-ttu-id="2679e-714">Entity Framework Core interpreta una proprietà come chiave esterna se è denominata `<navigation property name><primary key property name>`.</span><span class="sxs-lookup"><span data-stu-id="2679e-714">EF Core interprets a property as a foreign key if it's named `<navigation property name><primary key property name>`.</span></span> <span data-ttu-id="2679e-715">Ad esempio, `StudentID` per la proprietà di navigazione `Student`, poiché la chiave primaria `Student` dell'entità è `ID`.</span><span class="sxs-lookup"><span data-stu-id="2679e-715">For example,`StudentID` for the `Student` navigation property, since the `Student` entity's primary key is `ID`.</span></span> <span data-ttu-id="2679e-716">Le proprietà di chiave esterna possono anche essere denominate `<primary key property name>`.</span><span class="sxs-lookup"><span data-stu-id="2679e-716">Foreign key properties can also be named `<primary key property name>`.</span></span> <span data-ttu-id="2679e-717">Ad esempio, `CourseID` poiché la chiave primaria `Course` dell'entità è `CourseID`.</span><span class="sxs-lookup"><span data-stu-id="2679e-717">For example, `CourseID` since the `Course` entity's primary key is `CourseID`.</span></span>

### <a name="the-course-entity"></a><span data-ttu-id="2679e-718">Entità Course (Corso)</span><span class="sxs-lookup"><span data-stu-id="2679e-718">The Course entity</span></span>

![Diagramma entità Course (Corso)](intro/_static/course-entity.png)

<span data-ttu-id="2679e-720">Nella cartella *Models* (Modelli) creare un file *Course.cs* con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="2679e-720">In the *Models* folder, create *Course.cs* with the following code:</span></span>

[!code-csharp[](intro/samples/cu21/Models/Course.cs?name=snippet_Intro)]

<span data-ttu-id="2679e-721">La proprietà `Enrollments` rappresenta una proprietà di navigazione.</span><span class="sxs-lookup"><span data-stu-id="2679e-721">The `Enrollments` property is a navigation property.</span></span> <span data-ttu-id="2679e-722">È possibile correlare un'entità `Course` a un numero qualsiasi di entità `Enrollment`.</span><span class="sxs-lookup"><span data-stu-id="2679e-722">A `Course` entity can be related to any number of `Enrollment` entities.</span></span>

<span data-ttu-id="2679e-723">L'attributo `DatabaseGenerated` consente all'app di specificare la chiave primaria anziché chiedere al database di generarla.</span><span class="sxs-lookup"><span data-stu-id="2679e-723">The `DatabaseGenerated` attribute allows the app to specify the primary key rather than having the DB generate it.</span></span>

## <a name="scaffold-the-student-model"></a><span data-ttu-id="2679e-724">Eseguire lo scaffolding del modello Student (Studente)</span><span class="sxs-lookup"><span data-stu-id="2679e-724">Scaffold the student model</span></span>

<span data-ttu-id="2679e-725">In questa sezione viene eseguito lo scaffolding del modello Student (Studente).</span><span class="sxs-lookup"><span data-stu-id="2679e-725">In this section, the student model is scaffolded.</span></span> <span data-ttu-id="2679e-726">Lo strumento di scaffolding crea quindi le pagine per le operazioni CRUD (creazione, lettura, aggiornamento ed eliminazione) per il modello Student (Studente).</span><span class="sxs-lookup"><span data-stu-id="2679e-726">That is, the scaffolding tool produces pages for Create, Read, Update, and Delete (CRUD) operations for the student model.</span></span>

* <span data-ttu-id="2679e-727">Compilare il progetto.</span><span class="sxs-lookup"><span data-stu-id="2679e-727">Build the project.</span></span>
* <span data-ttu-id="2679e-728">Creare la cartella *Pages/Students*.</span><span class="sxs-lookup"><span data-stu-id="2679e-728">Create the *Pages/Students* folder.</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="2679e-729">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="2679e-729">Visual Studio</span></span>](#tab/visual-studio)

* <span data-ttu-id="2679e-730">In **Esplora soluzioni** fare clic con il pulsante destro del mouse sulla cartella *Pages/Students* > **Aggiungi** > **Nuovo elemento di scaffolding**.</span><span class="sxs-lookup"><span data-stu-id="2679e-730">In **Solution Explorer**, right click on the *Pages/Students* folder > **Add** > **New Scaffolded Item**.</span></span>
* <span data-ttu-id="2679e-731">Nella finestra di dialogo **Aggiungi impalcatura** selezionare **Razor pages using Entity Framework (CRUD)** > **Add**.</span><span class="sxs-lookup"><span data-stu-id="2679e-731">In the **Add Scaffold** dialog, select **Razor Pages using Entity Framework (CRUD)** > **ADD**.</span></span>

<span data-ttu-id="2679e-732">Completare la finestra di dialogo **Aggiungi Razor pagine con Entity Framework (CRUD)** :</span><span class="sxs-lookup"><span data-stu-id="2679e-732">Complete the **Add Razor Pages using Entity Framework (CRUD)** dialog:</span></span>

* <span data-ttu-id="2679e-733">Nell'elenco a discesa **Classe modello** selezionare **Student (ContosoUniversity.Models)**.</span><span class="sxs-lookup"><span data-stu-id="2679e-733">In the **Model class** drop-down, select **Student (ContosoUniversity.Models)**.</span></span>
* <span data-ttu-id="2679e-734">Nella riga **Classe contesto di dati** selezionare il segno **+** (più) e modificare il nome generato in **ContosoUniversity.Models.SchoolContext**.</span><span class="sxs-lookup"><span data-stu-id="2679e-734">In the **Data context class** row, select the **+** (plus) sign and change the generated name to **ContosoUniversity.Models.SchoolContext**.</span></span>
* <span data-ttu-id="2679e-735">Nell'elenco a discesa **Classe contesto di dati** selezionare **ContosoUniversity.Models.SchoolContext**.</span><span class="sxs-lookup"><span data-stu-id="2679e-735">In the **Data context class** drop-down,  select **ContosoUniversity.Models.SchoolContext**</span></span>
* <span data-ttu-id="2679e-736">Selezionare **Aggiungi**.</span><span class="sxs-lookup"><span data-stu-id="2679e-736">Select **Add**.</span></span>

![Finestra di dialogo CRUD](intro/_static/s1.png)

<span data-ttu-id="2679e-738">Vedere [Eseguire lo scaffolding del modello di filmato](xref:tutorials/razor-pages/model#scaffold-the-movie-model) se si riscontrano problemi con il passaggio precedente.</span><span class="sxs-lookup"><span data-stu-id="2679e-738">See [Scaffold the movie model](xref:tutorials/razor-pages/model#scaffold-the-movie-model) if you have a problem with the preceding step.</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="2679e-739">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="2679e-739">Visual Studio Code</span></span>](#tab/visual-studio-code)

<span data-ttu-id="2679e-740">Eseguire i comandi seguenti per eseguire lo scaffolding del modello Student (Studente).</span><span class="sxs-lookup"><span data-stu-id="2679e-740">Run the following commands to scaffold the student model.</span></span>

```dotnetcli
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design --version 2.1.0
dotnet tool install --global dotnet-aspnet-codegenerator
dotnet aspnet-codegenerator razorpage -m Student -dc ContosoUniversity.Models.SchoolContext -udl -outDir Pages/Students --referenceScriptLibraries
```

---

<span data-ttu-id="2679e-741">Il processo di scaffolding ha creato e modificato i file seguenti:</span><span class="sxs-lookup"><span data-stu-id="2679e-741">The scaffold process created and changed the following files:</span></span>

### <a name="files-created"></a><span data-ttu-id="2679e-742">File creati</span><span class="sxs-lookup"><span data-stu-id="2679e-742">Files created</span></span>

* <span data-ttu-id="2679e-743">Pagine Create (Crea), Delete (Elimina), Details (Dettagli), Edit (Modifica) e Index (Indice) di *Pages/Students*.</span><span class="sxs-lookup"><span data-stu-id="2679e-743">*Pages/Students* Create, Delete, Details, Edit, Index.</span></span>
* <span data-ttu-id="2679e-744">*Data/SchoolContext.cs*</span><span class="sxs-lookup"><span data-stu-id="2679e-744">*Data/SchoolContext.cs*</span></span>

### <a name="file-updates"></a><span data-ttu-id="2679e-745">Aggiornamenti dei file</span><span class="sxs-lookup"><span data-stu-id="2679e-745">File updates</span></span>

* <span data-ttu-id="2679e-746">*Startup.cs*: le modifiche a questo file sono descritte in dettaglio nella sezione successiva.</span><span class="sxs-lookup"><span data-stu-id="2679e-746">*Startup.cs* : Changes to this file are detailed in the next section.</span></span>
* <span data-ttu-id="2679e-747">*appsettings.json* : Viene aggiunta la stringa di connessione utilizzata per connettersi a un database locale.</span><span class="sxs-lookup"><span data-stu-id="2679e-747">*appsettings.json* : The connection string used to connect to a local database is added.</span></span>

## <a name="examine-the-context-registered-with-dependency-injection"></a><span data-ttu-id="2679e-748">Esaminare il contesto registrato con l'inserimento di dipendenze</span><span class="sxs-lookup"><span data-stu-id="2679e-748">Examine the context registered with dependency injection</span></span>

<span data-ttu-id="2679e-749">ASP.NET Core viene compilato tramite [dependency injection](xref:fundamentals/dependency-injection).</span><span class="sxs-lookup"><span data-stu-id="2679e-749">ASP.NET Core is built with [dependency injection](xref:fundamentals/dependency-injection).</span></span> <span data-ttu-id="2679e-750">I servizi, ad esempio il contesto di database di Entity Framework Core, vengono registrati tramite dependency injection durante l'avvio dell'applicazione.</span><span class="sxs-lookup"><span data-stu-id="2679e-750">Services (such as the EF Core DB context) are registered with dependency injection during application startup.</span></span> <span data-ttu-id="2679e-751">I componenti che richiedono questi servizi, ad esempio Razor le pagine, vengono forniti tramite i parametri del costruttore.</span><span class="sxs-lookup"><span data-stu-id="2679e-751">Components that require these services (such as Razor Pages) are provided these services via constructor parameters.</span></span> <span data-ttu-id="2679e-752">Più avanti nell'esercitazione viene illustrato il codice del costruttore che ottiene un'istanza del contesto di database.</span><span class="sxs-lookup"><span data-stu-id="2679e-752">The constructor code that gets a db context instance is shown later in the tutorial.</span></span>

<span data-ttu-id="2679e-753">Lo strumento di scaffolding ha creato automaticamente un contesto del database e lo ha registrato con il contenitore di inserimento delle dipendenze.</span><span class="sxs-lookup"><span data-stu-id="2679e-753">The scaffolding tool automatically created a DB Context and registered it with the dependency injection container.</span></span>

<span data-ttu-id="2679e-754">Esaminare il metodo `ConfigureServices` in *Startup.cs*.</span><span class="sxs-lookup"><span data-stu-id="2679e-754">Examine the `ConfigureServices` method in *Startup.cs*.</span></span> <span data-ttu-id="2679e-755">La riga evidenziata è stata aggiunta dallo scaffolder:</span><span class="sxs-lookup"><span data-stu-id="2679e-755">The highlighted line was added by the scaffolder:</span></span>

[!code-csharp[](intro/samples/cu21/Startup.cs?name=snippet_SchoolContext&highlight=13-14)]

<span data-ttu-id="2679e-756">Il nome della stringa di connessione viene passato al contesto chiamando un metodo in un oggetto [DbContextOptions](/dotnet/api/microsoft.entityframeworkcore.dbcontextoptions).</span><span class="sxs-lookup"><span data-stu-id="2679e-756">The name of the connection string is passed in to the context by calling a method on a [DbContextOptions](/dotnet/api/microsoft.entityframeworkcore.dbcontextoptions) object.</span></span> <span data-ttu-id="2679e-757">Per lo sviluppo locale, il [sistema di configurazione ASP.NET Core](xref:fundamentals/configuration/index) legge la stringa di connessione dal *appsettings.json* file.</span><span class="sxs-lookup"><span data-stu-id="2679e-757">For local development, the [ASP.NET Core configuration system](xref:fundamentals/configuration/index) reads the connection string from the *appsettings.json* file.</span></span>

## <a name="update-main"></a><span data-ttu-id="2679e-758">Aggiornare il metodo Main</span><span class="sxs-lookup"><span data-stu-id="2679e-758">Update main</span></span>

<span data-ttu-id="2679e-759">In *Program.cs* modificare il metodo `Main` per eseguire le operazioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="2679e-759">In *Program.cs*, modify the `Main` method to do the following:</span></span>

* <span data-ttu-id="2679e-760">Ottenere un'istanza del contesto di database dal contenitore di inserimento delle dipendenze.</span><span class="sxs-lookup"><span data-stu-id="2679e-760">Get a DB context instance from the dependency injection container.</span></span>
* <span data-ttu-id="2679e-761">Chiamare [EnsureCreated](/dotnet/api/microsoft.entityframeworkcore.infrastructure.databasefacade.ensurecreated#Microsoft_EntityFrameworkCore_Infrastructure_DatabaseFacade_EnsureCreated).</span><span class="sxs-lookup"><span data-stu-id="2679e-761">Call the  [EnsureCreated](/dotnet/api/microsoft.entityframeworkcore.infrastructure.databasefacade.ensurecreated#Microsoft_EntityFrameworkCore_Infrastructure_DatabaseFacade_EnsureCreated).</span></span>
* <span data-ttu-id="2679e-762">Eliminare il contesto dopo che il metodo `EnsureCreated` è stato completato.</span><span class="sxs-lookup"><span data-stu-id="2679e-762">Dispose the context when the `EnsureCreated` method completes.</span></span>

<span data-ttu-id="2679e-763">Il codice seguente illustra il file *Program.cs* aggiornato.</span><span class="sxs-lookup"><span data-stu-id="2679e-763">The following code shows the updated *Program.cs* file.</span></span>

[!code-csharp[](intro/samples/cu21/Program.cs?name=snippet)]

<span data-ttu-id="2679e-764">`EnsureCreated` assicura che esista il database per il contesto.</span><span class="sxs-lookup"><span data-stu-id="2679e-764">`EnsureCreated` ensures that the database for the context exists.</span></span> <span data-ttu-id="2679e-765">Se esiste, non viene eseguita alcuna azione.</span><span class="sxs-lookup"><span data-stu-id="2679e-765">If it exists, no action is taken.</span></span> <span data-ttu-id="2679e-766">Se non esiste, vengono creati il database e tutti i relativi schemi.</span><span class="sxs-lookup"><span data-stu-id="2679e-766">If it does not exist, then the database and all its schema are created.</span></span> <span data-ttu-id="2679e-767">`EnsureCreated` non usa le migrazioni per creare il database.</span><span class="sxs-lookup"><span data-stu-id="2679e-767">`EnsureCreated` does not use migrations to create the database.</span></span> <span data-ttu-id="2679e-768">Un database creato con `EnsureCreated` non potrà essere aggiornato successivamente usando le migrazioni.</span><span class="sxs-lookup"><span data-stu-id="2679e-768">A database that is created with `EnsureCreated` cannot be later updated using migrations.</span></span>

<span data-ttu-id="2679e-769">`EnsureCreated` viene chiamato all'avvio dell'app e consente il flusso di lavoro seguente:</span><span class="sxs-lookup"><span data-stu-id="2679e-769">`EnsureCreated` is called on app start, which allows the following work flow:</span></span>

* <span data-ttu-id="2679e-770">Eliminare il database.</span><span class="sxs-lookup"><span data-stu-id="2679e-770">Delete the DB.</span></span>
* <span data-ttu-id="2679e-771">Modificare lo schema di database, ad esempio, aggiungere un campo `EmailAddress`.</span><span class="sxs-lookup"><span data-stu-id="2679e-771">Change the DB schema (for example, add an `EmailAddress` field).</span></span>
* <span data-ttu-id="2679e-772">Eseguire l'app.</span><span class="sxs-lookup"><span data-stu-id="2679e-772">Run the app.</span></span>
* <span data-ttu-id="2679e-773">`EnsureCreated` crea un database con la colonna `EmailAddress`.</span><span class="sxs-lookup"><span data-stu-id="2679e-773">`EnsureCreated` creates a DB with the`EmailAddress` column.</span></span>

<span data-ttu-id="2679e-774">`EnsureCreated` è utile nelle prime fasi di sviluppo quando lo schema è in rapida evoluzione.</span><span class="sxs-lookup"><span data-stu-id="2679e-774">`EnsureCreated` is convenient early in development when the schema is rapidly evolving.</span></span> <span data-ttu-id="2679e-775">Più avanti nell'esercitazione il database verrà eliminato e si useranno le migrazioni.</span><span class="sxs-lookup"><span data-stu-id="2679e-775">Later in the tutorial the DB is deleted and migrations are used.</span></span>

### <a name="test-the-app"></a><span data-ttu-id="2679e-776">Testare l'app</span><span class="sxs-lookup"><span data-stu-id="2679e-776">Test the app</span></span>

<span data-ttu-id="2679e-777">Eseguire l'app e accettare i cookie criteri.</span><span class="sxs-lookup"><span data-stu-id="2679e-777">Run the app and accept the cookie policy.</span></span> <span data-ttu-id="2679e-778">Questa app non memorizza informazioni personali.</span><span class="sxs-lookup"><span data-stu-id="2679e-778">This app doesn't keep personal information.</span></span> <span data-ttu-id="2679e-779">Per informazioni sui criteri, vedere la pagina relativa cookie al [supporto per regolamento generale sulla protezione dei dati UE (GDPR)](xref:security/gdpr).</span><span class="sxs-lookup"><span data-stu-id="2679e-779">You can read about the cookie policy at [EU General Data Protection Regulation (GDPR) support](xref:security/gdpr).</span></span>

* <span data-ttu-id="2679e-780">Selezionare il collegamento **Students** (Studenti) e quindi **Crea nuovo**.</span><span class="sxs-lookup"><span data-stu-id="2679e-780">Select the **Students** link and then **Create New**.</span></span>
* <span data-ttu-id="2679e-781">Eseguire il test dei collegamenti Edit (Modifica), Details (Dettagli) e Delete (Elimina).</span><span class="sxs-lookup"><span data-stu-id="2679e-781">Test the Edit, Details, and Delete links.</span></span>

## <a name="examine-the-schoolcontext-db-context"></a><span data-ttu-id="2679e-782">Esaminare il contesto di database SchoolContext</span><span class="sxs-lookup"><span data-stu-id="2679e-782">Examine the SchoolContext DB context</span></span>

<span data-ttu-id="2679e-783">La classe principale che coordina le funzionalità di Entity Framework Core per un determinato modello di dati è la classe del contesto di database.</span><span class="sxs-lookup"><span data-stu-id="2679e-783">The main class that coordinates EF Core functionality for a given data model is the DB context class.</span></span> <span data-ttu-id="2679e-784">Il contesto dei dati è derivato da [Microsoft.EntityFrameworkCore.DbContext](/dotnet/api/microsoft.entityframeworkcore.dbcontext).</span><span class="sxs-lookup"><span data-stu-id="2679e-784">The data context is derived from [Microsoft.EntityFrameworkCore.DbContext](/dotnet/api/microsoft.entityframeworkcore.dbcontext).</span></span> <span data-ttu-id="2679e-785">Il contesto dei dati specifica le entità incluse nel modello di dati.</span><span class="sxs-lookup"><span data-stu-id="2679e-785">The data context specifies which entities are included in the data model.</span></span> <span data-ttu-id="2679e-786">In questo progetto la classe è denominata `SchoolContext`.</span><span class="sxs-lookup"><span data-stu-id="2679e-786">In this project, the class is named `SchoolContext`.</span></span>

<span data-ttu-id="2679e-787">Aggiornare *SchoolContext.cs* con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="2679e-787">Update *SchoolContext.cs* with the following code:</span></span>

[!code-csharp[](intro/samples/cu21/Data/SchoolContext.cs?name=snippet_Intro&highlight=12-14)]

<span data-ttu-id="2679e-788">Il codice evidenziato crea una proprietà [DbSet \<TEntity> ](/dotnet/api/microsoft.entityframeworkcore.dbset-1) per ogni set di entità.</span><span class="sxs-lookup"><span data-stu-id="2679e-788">The highlighted code creates a [DbSet\<TEntity>](/dotnet/api/microsoft.entityframeworkcore.dbset-1) property for each entity set.</span></span> <span data-ttu-id="2679e-789">Nella terminologia di Entity Framework Core:</span><span class="sxs-lookup"><span data-stu-id="2679e-789">In EF Core terminology:</span></span>

* <span data-ttu-id="2679e-790">Nella terminologia di Entity Framework, un set di entità corrisponde in genere alla tabella di un database.</span><span class="sxs-lookup"><span data-stu-id="2679e-790">An entity set typically corresponds to a DB table.</span></span>
* <span data-ttu-id="2679e-791">Un'entità corrisponde a una riga nella tabella.</span><span class="sxs-lookup"><span data-stu-id="2679e-791">An entity corresponds to a row in the table.</span></span>

<span data-ttu-id="2679e-792">`DbSet<Enrollment>` e `DbSet<Course>` potrebbero essere omessi.</span><span class="sxs-lookup"><span data-stu-id="2679e-792">`DbSet<Enrollment>` and `DbSet<Course>` could be omitted.</span></span> <span data-ttu-id="2679e-793">Core Entity Framework le include in modo implicito perché l'entità `Student` fa riferimento all'entità `Enrollment` e l'entità `Enrollment` fa riferimento all'entità `Course`.</span><span class="sxs-lookup"><span data-stu-id="2679e-793">EF Core includes them implicitly because the `Student` entity references the `Enrollment` entity, and the `Enrollment` entity references the `Course` entity.</span></span> <span data-ttu-id="2679e-794">Per questa esercitazione, mantenere `DbSet<Enrollment>` e `DbSet<Course>` in `SchoolContext`.</span><span class="sxs-lookup"><span data-stu-id="2679e-794">For this tutorial, keep `DbSet<Enrollment>` and `DbSet<Course>` in the `SchoolContext`.</span></span>

### <a name="sql-server-express-localdb"></a><span data-ttu-id="2679e-795">SQL Server Express LocalDB</span><span class="sxs-lookup"><span data-stu-id="2679e-795">SQL Server Express LocalDB</span></span>

<span data-ttu-id="2679e-796">La stringa di connessione specifica [SQL Server Local DB](/sql/database-engine/configure-windows/sql-server-2016-express-localdb).</span><span class="sxs-lookup"><span data-stu-id="2679e-796">The connection string specifies [SQL Server LocalDB](/sql/database-engine/configure-windows/sql-server-2016-express-localdb).</span></span> <span data-ttu-id="2679e-797">Local DB è una versione leggera del motore di database di SQL Server Express appositamente pensato per lo sviluppo di app e non per la produzione.</span><span class="sxs-lookup"><span data-stu-id="2679e-797">LocalDB is a lightweight version of the SQL Server Express Database Engine and is intended for app development, not production use.</span></span> <span data-ttu-id="2679e-798">Local DB viene avviato su richiesta ed eseguito in modalità utente; non richiede quindi una configurazione complessa.</span><span class="sxs-lookup"><span data-stu-id="2679e-798">LocalDB starts on demand and runs in user mode, so there's no complex configuration.</span></span> <span data-ttu-id="2679e-799">Per impostazione predefinita, Local DB crea file di database con estensione *mdf* nella directory `C:/Users/<user>`.</span><span class="sxs-lookup"><span data-stu-id="2679e-799">By default, LocalDB creates *.mdf* DB files in the `C:/Users/<user>` directory.</span></span>

## <a name="add-code-to-initialize-the-db-with-test-data"></a><span data-ttu-id="2679e-800">Aggiungere il codice per inizializzare il database con dati di test</span><span class="sxs-lookup"><span data-stu-id="2679e-800">Add code to initialize the DB with test data</span></span>

<span data-ttu-id="2679e-801">Entity Framework Core crea un database vuoto.</span><span class="sxs-lookup"><span data-stu-id="2679e-801">EF Core creates an empty DB.</span></span> <span data-ttu-id="2679e-802">In questa sezione viene scritto un metodo `Initialize` per popolare il database con i dati di test.</span><span class="sxs-lookup"><span data-stu-id="2679e-802">In this section, an `Initialize` method is written to populate it with test data.</span></span>

<span data-ttu-id="2679e-803">Nella cartella *Data* (Dati) creare un nuovo file di classe denominato *DbInitializer.cs* e aggiungere il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="2679e-803">In the *Data* folder, create a new class file named *DbInitializer.cs* and add the following code:</span></span>

[!code-csharp[](intro/samples/cu21/Data/DbInitializer.cs?name=snippet_Intro)]

<span data-ttu-id="2679e-804">Nota: il codice precedente USA `Models` per lo spazio dei nomi ( `namespace ContosoUniversity.Models` ) anziché `Data` .</span><span class="sxs-lookup"><span data-stu-id="2679e-804">Note: The preceding code uses `Models` for the namespace (`namespace ContosoUniversity.Models`) rather than `Data`.</span></span> <span data-ttu-id="2679e-805">`Models` è coerente con il codice generato dallo scaffolder.</span><span class="sxs-lookup"><span data-stu-id="2679e-805">`Models` is consistent with the scaffolder-generated code.</span></span> <span data-ttu-id="2679e-806">Per altre informazioni, vedere [questo problema relativo allo scaffolding in GitHub](https://github.com/aspnet/Scaffolding/issues/822).</span><span class="sxs-lookup"><span data-stu-id="2679e-806">For more information, see [this GitHub scaffolding issue](https://github.com/aspnet/Scaffolding/issues/822).</span></span>

<span data-ttu-id="2679e-807">Il codice controlla se esistono studenti nel database.</span><span class="sxs-lookup"><span data-stu-id="2679e-807">The code checks if there are any students in the DB.</span></span> <span data-ttu-id="2679e-808">Se il database non contiene studenti, il database viene inizializzato con i dati di test.</span><span class="sxs-lookup"><span data-stu-id="2679e-808">If there are no students in the DB, the DB is initialized with test data.</span></span> <span data-ttu-id="2679e-809">I dati di test vengono caricati in matrici anziché in raccolte `List<T>` per ottimizzare le prestazioni.</span><span class="sxs-lookup"><span data-stu-id="2679e-809">It loads test data into arrays rather than `List<T>` collections to optimize performance.</span></span>

<span data-ttu-id="2679e-810">Il metodo `EnsureCreated` crea automaticamente il database per il contesto di database.</span><span class="sxs-lookup"><span data-stu-id="2679e-810">The `EnsureCreated` method automatically creates the DB for the DB context.</span></span> <span data-ttu-id="2679e-811">Se il database esiste, `EnsureCreated` restituisce senza modificare il database.</span><span class="sxs-lookup"><span data-stu-id="2679e-811">If the DB exists, `EnsureCreated` returns without modifying the DB.</span></span>

<span data-ttu-id="2679e-812">In *Program.cs* modificare il metodo `Main` per chiamare `Initialize`:</span><span class="sxs-lookup"><span data-stu-id="2679e-812">In *Program.cs*, modify the `Main` method to call `Initialize`:</span></span>

[!code-csharp[](intro/samples/cu21/Program.cs?name=snippet2&highlight=14-15)]

# <a name="visual-studio"></a>[<span data-ttu-id="2679e-813">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="2679e-813">Visual Studio</span></span>](#tab/visual-studio)

<span data-ttu-id="2679e-814">Arrestare l'app se è in esecuzione ed eseguire il comando seguente nella **console di Gestione pacchetti**:</span><span class="sxs-lookup"><span data-stu-id="2679e-814">Stop the app if it's running, and run the following command in the **Package Manager Console** (PMC):</span></span>

```powershell
Drop-Database
```

# <a name="visual-studio-code"></a>[<span data-ttu-id="2679e-815">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="2679e-815">Visual Studio Code</span></span>](#tab/visual-studio-code)

* <span data-ttu-id="2679e-816">Arrestare l'app se è in esecuzione ed eliminare il file *CU.db*.</span><span class="sxs-lookup"><span data-stu-id="2679e-816">Stop the app if it's running, and delete the *CU.db* file.</span></span>

---

## <a name="view-the-db"></a><span data-ttu-id="2679e-817">Visualizzare il database</span><span class="sxs-lookup"><span data-stu-id="2679e-817">View the DB</span></span>

<span data-ttu-id="2679e-818">Il nome del database viene generato dal nome del contesto specificato in precedenza con l'aggiunta di un trattino e un GUID.</span><span class="sxs-lookup"><span data-stu-id="2679e-818">The database name is generated from the context name you provided earlier plus a dash and a GUID.</span></span> <span data-ttu-id="2679e-819">Di conseguenza, il nome del database sarà "SchoolContext-{GUID}".</span><span class="sxs-lookup"><span data-stu-id="2679e-819">Thus, the database name will be "SchoolContext-{GUID}".</span></span> <span data-ttu-id="2679e-820">Il GUID sarà diverso per ogni utente.</span><span class="sxs-lookup"><span data-stu-id="2679e-820">The GUID will be different for each user.</span></span>
<span data-ttu-id="2679e-821">Aprire **Esplora oggetti di SQL Server** dal menu **Visualizza** in Visual Studio.</span><span class="sxs-lookup"><span data-stu-id="2679e-821">Open **SQL Server Object Explorer** (SSOX) from the **View** menu in Visual Studio.</span></span>
<span data-ttu-id="2679e-822">In Esplora oggetti di SQL Server fare clic su **(localdb)\MSSQLLocalDB > Databases > SchoolContext-{GUID}**.</span><span class="sxs-lookup"><span data-stu-id="2679e-822">In SSOX, click **(localdb)\MSSQLLocalDB > Databases > SchoolContext-{GUID}**.</span></span>

<span data-ttu-id="2679e-823">Espandere il nodo **Tabelle**.</span><span class="sxs-lookup"><span data-stu-id="2679e-823">Expand the **Tables** node.</span></span>

<span data-ttu-id="2679e-824">Fare clic con il pulsante destro del mouse sulla tabella **Student** (Studente) e fare clic su **Visualizza dati** per visualizzare le colonne create e le righe inserite nella tabella.</span><span class="sxs-lookup"><span data-stu-id="2679e-824">Right-click the **Student** table and click **View Data** to see the columns created and the rows inserted into the table.</span></span>

## <a name="asynchronous-code"></a><span data-ttu-id="2679e-825">Codice asincrono</span><span class="sxs-lookup"><span data-stu-id="2679e-825">Asynchronous code</span></span>

<span data-ttu-id="2679e-826">La programmazione asincrona è la modalità predefinita per ASP.NET Core ed Entity Framework Core.</span><span class="sxs-lookup"><span data-stu-id="2679e-826">Asynchronous programming is the default mode for ASP.NET Core and EF Core.</span></span>

<span data-ttu-id="2679e-827">Per un server Web è disponibile un numero limitato di thread e in situazioni di carico elevato tutti i thread disponibili potrebbero essere in uso.</span><span class="sxs-lookup"><span data-stu-id="2679e-827">A web server has a limited number of threads available, and in high load situations all of the available threads might be in use.</span></span> <span data-ttu-id="2679e-828">In queste circostanze il server non può elaborare nuove richieste finché i thread non saranno liberi.</span><span class="sxs-lookup"><span data-stu-id="2679e-828">When that happens, the server can't process new requests until the threads are freed up.</span></span> <span data-ttu-id="2679e-829">Con il codice sincrono, può succedere che molti thread siano vincolati nonostante in quel momento non stiano eseguendo alcuna operazione. Rimangono tuttavia in attesa che l'operazione I/O sia completata.</span><span class="sxs-lookup"><span data-stu-id="2679e-829">With synchronous code, many threads may be tied up while they aren't actually doing any work because they're waiting for I/O to complete.</span></span> <span data-ttu-id="2679e-830">Con il codice asincrono, se un processo è in attesa del completamento dell'operazione I/O, il thread viene liberato e il server lo può usare per l'elaborazione di altre richieste.</span><span class="sxs-lookup"><span data-stu-id="2679e-830">With asynchronous code, when a process is waiting for I/O to complete, its thread is freed up for the server to use for processing other requests.</span></span> <span data-ttu-id="2679e-831">Di conseguenza, il codice asincrono consente un uso più efficiente delle risorse e il server è abilitato per gestire una maggiore quantità di traffico senza ritardi.</span><span class="sxs-lookup"><span data-stu-id="2679e-831">As a result, asynchronous code enables server resources to be used more efficiently, and the server is enabled to handle more traffic without delays.</span></span>

<span data-ttu-id="2679e-832">Il codice asincrono comporta un minimo sovraccarico in fase di esecuzione.</span><span class="sxs-lookup"><span data-stu-id="2679e-832">Asynchronous code does introduce a small amount of overhead at run time.</span></span> <span data-ttu-id="2679e-833">In caso di traffico ridotto, il calo delle prestazioni è irrilevante, mentre nelle situazioni di traffico elevato, è essenziale ottimizzare le prestazioni potenziali.</span><span class="sxs-lookup"><span data-stu-id="2679e-833">For low traffic situations, the performance hit is negligible, while for high traffic situations, the potential performance improvement is substantial.</span></span>

<span data-ttu-id="2679e-834">Nel codice seguente la parola chiave [async](/dotnet/csharp/language-reference/keywords/async), il valore restituito `Task<T>`, la parola chiave `await` e il metodo `ToListAsync` consentono di eseguire il codice in modo asincrono.</span><span class="sxs-lookup"><span data-stu-id="2679e-834">In the following code, the [async](/dotnet/csharp/language-reference/keywords/async) keyword, `Task<T>` return value, `await` keyword, and `ToListAsync` method make the code execute asynchronously.</span></span>

[!code-csharp[](intro/samples/cu21/Pages/Students/Index.cshtml.cs?name=snippet_ScaffoldedIndex)]

* <span data-ttu-id="2679e-835">La parola chiave `async` indica al compilatore di eseguire le operazioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="2679e-835">The `async` keyword tells the compiler to:</span></span>
  * <span data-ttu-id="2679e-836">Generare callback per parti del corpo del metodo.</span><span class="sxs-lookup"><span data-stu-id="2679e-836">Generate callbacks for parts of the method body.</span></span>
  * <span data-ttu-id="2679e-837">Creare automaticamente l' oggetto [Task](/dotnet/api/system.threading.tasks.task) restituito.</span><span class="sxs-lookup"><span data-stu-id="2679e-837">Automatically create the [Task](/dotnet/api/system.threading.tasks.task) object that's returned.</span></span> <span data-ttu-id="2679e-838">Per altre informazioni, vedere [Tipo restituito Task](/dotnet/csharp/programming-guide/concepts/async/async-return-types#BKMK_TaskReturnType).</span><span class="sxs-lookup"><span data-stu-id="2679e-838">For more information, see [Task Return Type](/dotnet/csharp/programming-guide/concepts/async/async-return-types#BKMK_TaskReturnType).</span></span>

* <span data-ttu-id="2679e-839">Il tipo restituito implicito `Task` rappresenta il lavoro in corso.</span><span class="sxs-lookup"><span data-stu-id="2679e-839">The implicit return type `Task` represents ongoing work.</span></span>
* <span data-ttu-id="2679e-840">La parola chiave `await` indica al compilatore di suddividere il metodo in due parti.</span><span class="sxs-lookup"><span data-stu-id="2679e-840">The `await` keyword causes the compiler to split the method into two parts.</span></span> <span data-ttu-id="2679e-841">La prima parte termina con l'operazione avviata in modo asincrono.</span><span class="sxs-lookup"><span data-stu-id="2679e-841">The first part ends with the operation that's started asynchronously.</span></span> <span data-ttu-id="2679e-842">La seconda parte viene inserita in un metodo di callback che viene chiamato al termine dell'operazione.</span><span class="sxs-lookup"><span data-stu-id="2679e-842">The second part is put into a callback method that's called when the operation completes.</span></span>
* <span data-ttu-id="2679e-843">`ToListAsync` è la versione asincrona del metodo di estensione `ToList`.</span><span class="sxs-lookup"><span data-stu-id="2679e-843">`ToListAsync` is the asynchronous version of the `ToList` extension method.</span></span>

<span data-ttu-id="2679e-844">Alcuni aspetti da considerare quando si scrive codice asincrono usato da Entity Framework Core:</span><span class="sxs-lookup"><span data-stu-id="2679e-844">Some things to be aware of when writing asynchronous code that uses EF Core:</span></span>

* <span data-ttu-id="2679e-845">Solo le istruzioni che generano query o comandi da inviare al database vengono eseguite in modo asincrono.</span><span class="sxs-lookup"><span data-stu-id="2679e-845">Only statements that cause queries or commands to be sent to the DB are executed asynchronously.</span></span> <span data-ttu-id="2679e-846">Sono incluse `ToListAsync`, `SingleOrDefaultAsync`, `FirstOrDefaultAsync` e `SaveChangesAsync`.</span><span class="sxs-lookup"><span data-stu-id="2679e-846">That includes, `ToListAsync`, `SingleOrDefaultAsync`, `FirstOrDefaultAsync`, and `SaveChangesAsync`.</span></span> <span data-ttu-id="2679e-847">Non sono incluse le istruzioni che modificano solo un oggetto `IQueryable`, ad esempio `var students = context.Students.Where(s => s.LastName == "Davolio")`.</span><span class="sxs-lookup"><span data-stu-id="2679e-847">It doesn't include statements that just change an `IQueryable`, such as `var students = context.Students.Where(s => s.LastName == "Davolio")`.</span></span>
* <span data-ttu-id="2679e-848">Un contesto di Entity Framework Core non è thread-safe. Non tentare di eseguire più operazioni parallelamente.</span><span class="sxs-lookup"><span data-stu-id="2679e-848">An EF Core context isn't thread safe: don't try to do multiple operations in parallel.</span></span>
* <span data-ttu-id="2679e-849">Per sfruttare i vantaggi del codice asincrono in termini di prestazioni, verificare che i pacchetti della libreria impiegati, ad esempio per il paging, usino la modalità asincrona per chiamare i metodi di Entity Framework Core che generano query da inviare al database.</span><span class="sxs-lookup"><span data-stu-id="2679e-849">To take advantage of the performance benefits of async code, verify that library packages (such as for paging) use async if they call EF Core methods that send queries to the DB.</span></span>

<span data-ttu-id="2679e-850">Per altre informazioni sulla programmazione asincrona in .NET, vedere [Panoramica della programmazione asincrona](/dotnet/standard/async) e [Programmazione asincrona con async e await](/dotnet/csharp/programming-guide/concepts/async/).</span><span class="sxs-lookup"><span data-stu-id="2679e-850">For more information about asynchronous programming in .NET, see [Async Overview](/dotnet/standard/async) and [Asynchronous programming with async and await](/dotnet/csharp/programming-guide/concepts/async/).</span></span>

<span data-ttu-id="2679e-851">Nella prossima esercitazione, vengono esaminate le operazioni CRUD per creare, leggere, aggiornare, eliminare ed elencare.</span><span class="sxs-lookup"><span data-stu-id="2679e-851">In the next tutorial, basic CRUD (create, read, update, delete) operations are examined.</span></span>



## <a name="additional-resources"></a><span data-ttu-id="2679e-852">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="2679e-852">Additional resources</span></span>

* [<span data-ttu-id="2679e-853">Versione YouTube dell'esercitazione</span><span class="sxs-lookup"><span data-stu-id="2679e-853">YouTube version of this tutorial</span></span>](https://www.youtube.com/watch?v=P7iTtQnkrNs)

> [!div class="step-by-step"]
> [<span data-ttu-id="2679e-854">Avanti</span><span class="sxs-lookup"><span data-stu-id="2679e-854">Next</span></span>](xref:data/ef-rp/crud)

::: moniker-end
