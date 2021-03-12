---
title: "Esercitazione: implementare l'ereditarietà-ASP.NET MVC con EF Core"
description: Questa esercitazione illustra come implementare l'ereditarietà nel modello di dati usando Entity Framework Core in un'applicazione ASP.NET Core.
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
uid: data/ef-mvc/inheritance
ms.openlocfilehash: 299e00b223d287c4e2ece3d1e250581e2a7565e5
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102586371"
---
# <a name="tutorial-implement-inheritance---aspnet-mvc-with-ef-core"></a><span data-ttu-id="cbfc5-103">Esercitazione: implementare l'ereditarietà-ASP.NET MVC con EF Core</span><span class="sxs-lookup"><span data-stu-id="cbfc5-103">Tutorial: Implement inheritance - ASP.NET MVC with EF Core</span></span>

<span data-ttu-id="cbfc5-104">Nell'esercitazione precedente sono state presentate le eccezioni di concorrenza.</span><span class="sxs-lookup"><span data-stu-id="cbfc5-104">In the previous tutorial, you handled concurrency exceptions.</span></span> <span data-ttu-id="cbfc5-105">In questa esercitazione viene illustrato come implementare l'ereditarietà nel modello di dati.</span><span class="sxs-lookup"><span data-stu-id="cbfc5-105">This tutorial will show you how to implement inheritance in the data model.</span></span>

<span data-ttu-id="cbfc5-106">Nella programmazione orientata a oggetti è possibile usare l'ereditarietà per facilitare il riutilizzo del codice.</span><span class="sxs-lookup"><span data-stu-id="cbfc5-106">In object-oriented programming, you can use inheritance to facilitate code reuse.</span></span> <span data-ttu-id="cbfc5-107">In questa esercitazione verranno modificate le classi `Instructor` e `Student` in modo che derivino da una classe di base `Person` contenente proprietà quali `LastName` comuni a docenti e studenti.</span><span class="sxs-lookup"><span data-stu-id="cbfc5-107">In this tutorial, you'll change the `Instructor` and `Student` classes so that they derive from a `Person` base class which contains properties such as `LastName` that are common to both instructors and students.</span></span> <span data-ttu-id="cbfc5-108">Non verranno aggiunte o modificate pagine Web, ma si modificherà parte del codice e le modifiche verranno automaticamente riflesse nel database.</span><span class="sxs-lookup"><span data-stu-id="cbfc5-108">You won't add or change any web pages, but you'll change some of the code and those changes will be automatically reflected in the database.</span></span>

<span data-ttu-id="cbfc5-109">In questa esercitazione:</span><span class="sxs-lookup"><span data-stu-id="cbfc5-109">In this tutorial, you:</span></span>

> [!div class="checklist"]
> * <span data-ttu-id="cbfc5-110">Eseguire il mapping dell'ereditarietà al database</span><span class="sxs-lookup"><span data-stu-id="cbfc5-110">Map inheritance to database</span></span>
> * <span data-ttu-id="cbfc5-111">Creare la classe Person</span><span class="sxs-lookup"><span data-stu-id="cbfc5-111">Create the Person class</span></span>
> * <span data-ttu-id="cbfc5-112">Aggiornare Student e Instructor</span><span class="sxs-lookup"><span data-stu-id="cbfc5-112">Update Instructor and Student</span></span>
> * <span data-ttu-id="cbfc5-113">Aggiungere Person al modello</span><span class="sxs-lookup"><span data-stu-id="cbfc5-113">Add Person to the model</span></span>
> * <span data-ttu-id="cbfc5-114">Creare e aggiornare le migrazioni</span><span class="sxs-lookup"><span data-stu-id="cbfc5-114">Create and update migrations</span></span>
> * <span data-ttu-id="cbfc5-115">Testare l'implementazione</span><span class="sxs-lookup"><span data-stu-id="cbfc5-115">Test the implementation</span></span>

## <a name="prerequisites"></a><span data-ttu-id="cbfc5-116">Prerequisiti</span><span class="sxs-lookup"><span data-stu-id="cbfc5-116">Prerequisites</span></span>

* [<span data-ttu-id="cbfc5-117">Gestire la concorrenza</span><span class="sxs-lookup"><span data-stu-id="cbfc5-117">Handle Concurrency</span></span>](concurrency.md)

## <a name="map-inheritance-to-database"></a><span data-ttu-id="cbfc5-118">Eseguire il mapping dell'ereditarietà al database</span><span class="sxs-lookup"><span data-stu-id="cbfc5-118">Map inheritance to database</span></span>

<span data-ttu-id="cbfc5-119">Le classi `Instructor` e `Student` nel modello di dati School presentano molte proprietà identiche:</span><span class="sxs-lookup"><span data-stu-id="cbfc5-119">The `Instructor` and `Student` classes in the School data model have several properties that are identical:</span></span>

![Classi Student e Instructor](inheritance/_static/no-inheritance.png)

<span data-ttu-id="cbfc5-121">Si supponga di voler eliminare il codice ridondante per le proprietà condivise dalle entità `Instructor` e `Student`.</span><span class="sxs-lookup"><span data-stu-id="cbfc5-121">Suppose you want to eliminate the redundant code for the properties that are shared by the `Instructor` and `Student` entities.</span></span> <span data-ttu-id="cbfc5-122">Oppure che si desideri scrivere un servizio in grado di formattare i nomi senza sapere se il nome appartiene a un docente o a uno studente.</span><span class="sxs-lookup"><span data-stu-id="cbfc5-122">Or you want to write a service that can format names without caring whether the name came from an instructor or a student.</span></span> <span data-ttu-id="cbfc5-123">È possibile creare una classe di base `Person` che contiene solo le proprietà condivise e quindi fare in modo che le classi `Instructor` e `Student` ereditino da questa classe di base, come illustrato nella figura seguente:</span><span class="sxs-lookup"><span data-stu-id="cbfc5-123">You could create a `Person` base class that contains only those shared properties, then make the `Instructor` and `Student` classes inherit from that base class, as shown in the following illustration:</span></span>

![Classi Student e Instructor derivanti dalla classe Person](inheritance/_static/inheritance.png)

<span data-ttu-id="cbfc5-125">Questa struttura di ereditarietà può essere rappresentata nel database in diversi modi.</span><span class="sxs-lookup"><span data-stu-id="cbfc5-125">There are several ways this inheritance structure could be represented in the database.</span></span> <span data-ttu-id="cbfc5-126">È possibile usare una tabella Person che includa le informazioni relative a studenti e docenti in un'unica tabella.</span><span class="sxs-lookup"><span data-stu-id="cbfc5-126">You could have a Person table that includes information about both students and instructors in a single table.</span></span> <span data-ttu-id="cbfc5-127">Alcune colonne possono riguardare solo i docenti (HireDate), altre solo gli studenti (EnrollmentDate) e altre ancora entrambi (LastName, FirstName).</span><span class="sxs-lookup"><span data-stu-id="cbfc5-127">Some of the columns could apply only to instructors (HireDate), some only to students (EnrollmentDate), some to both (LastName, FirstName).</span></span> <span data-ttu-id="cbfc5-128">Per indicare il tipo rappresentato da ogni riga viene in genere usata una colonna discriminante.</span><span class="sxs-lookup"><span data-stu-id="cbfc5-128">Typically, you'd have a discriminator column to indicate which type each row represents.</span></span> <span data-ttu-id="cbfc5-129">La colonna discriminante può ad esempio indicare "Instructor" per i docenti e "Student" per gli studenti.</span><span class="sxs-lookup"><span data-stu-id="cbfc5-129">For example, the discriminator column might have "Instructor" for instructors and "Student" for students.</span></span>

![Esempio di tabella per gerarchia](inheritance/_static/tph.png)

<span data-ttu-id="cbfc5-131">Questo criterio di generazione di una struttura di ereditarietà delle entità da una singola tabella di database è denominato ereditarietà tabella per gerarchia.</span><span class="sxs-lookup"><span data-stu-id="cbfc5-131">This pattern of generating an entity inheritance structure from a single database table is called table-per-hierarchy (TPH) inheritance.</span></span>

<span data-ttu-id="cbfc5-132">Un'alternativa consiste nel rendere il database più simile alla struttura di ereditarietà.</span><span class="sxs-lookup"><span data-stu-id="cbfc5-132">An alternative is to make the database look more like the inheritance structure.</span></span> <span data-ttu-id="cbfc5-133">È possibile, ad esempio, includere nella tabella Person solo i campi del nome e usare tabelle Instructor e Student separate con i campi della data.</span><span class="sxs-lookup"><span data-stu-id="cbfc5-133">For example, you could have only the name fields in the Person table and have separate Instructor and Student tables with the date fields.</span></span>

> [!WARNING]
> <span data-ttu-id="cbfc5-134">La tabella per tipo (TPT) non è supportata da EF Core 3. x, ma è stata implementata in [EF Core 5,0](/ef/core/what-is-new/ef-core-5.0/plan).</span><span class="sxs-lookup"><span data-stu-id="cbfc5-134">Table Per Type (TPT) is not supported by EF Core 3.x, however it is has been implemented in [EF Core 5.0](/ef/core/what-is-new/ef-core-5.0/plan).</span></span>

![Ereditarietà tabella per tipo](inheritance/_static/tpt.png)

<span data-ttu-id="cbfc5-136">Questo criterio di creazione di una tabella di database per ogni classe di entità è denominato ereditarietà tabella per tipo.</span><span class="sxs-lookup"><span data-stu-id="cbfc5-136">This pattern of making a database table for each entity class is called table per type (TPT) inheritance.</span></span>

<span data-ttu-id="cbfc5-137">Un'altra opzione consiste nell'eseguire il mapping di tutti i tipi non astratti a singole tabelle.</span><span class="sxs-lookup"><span data-stu-id="cbfc5-137">Yet another option is to map all non-abstract types to individual tables.</span></span> <span data-ttu-id="cbfc5-138">Tutte le proprietà di una classe, incluse le proprietà ereditate, eseguono il mapping alle colonne della tabella corrispondente.</span><span class="sxs-lookup"><span data-stu-id="cbfc5-138">All properties of a class, including inherited properties, map to columns of the corresponding table.</span></span> <span data-ttu-id="cbfc5-139">Questo criterio è denominato ereditarietà della classe tabella per tipo concreto.</span><span class="sxs-lookup"><span data-stu-id="cbfc5-139">This pattern is called Table-per-Concrete Class (TPC) inheritance.</span></span> <span data-ttu-id="cbfc5-140">Se per le classi Person, Student e Instructor è stata implementata l'ereditarietà tabella per tipo concreto come illustrato in precedenza, l'aspetto delle tabelle Student e Instructor dopo l'implementazione dell'ereditarietà non sarà diverso da quello precedente.</span><span class="sxs-lookup"><span data-stu-id="cbfc5-140">If you implemented TPC inheritance for the Person, Student, and Instructor classes as shown earlier, the Student and Instructor tables would look no different after implementing inheritance than they did before.</span></span>

<span data-ttu-id="cbfc5-141">I criteri tabella per tipo concreto e tabella per gerarchia offrono in genere prestazioni migliori rispetto ai criteri tabella per tipo perché questi ultimi possono generare query join complesse.</span><span class="sxs-lookup"><span data-stu-id="cbfc5-141">TPC and TPH inheritance patterns generally deliver better performance than TPT inheritance patterns, because TPT patterns can result in complex join queries.</span></span>

<span data-ttu-id="cbfc5-142">Questa esercitazione illustra come implementare l'ereditarietà tabella per gerarchia.</span><span class="sxs-lookup"><span data-stu-id="cbfc5-142">This tutorial demonstrates how to implement TPH inheritance.</span></span> <span data-ttu-id="cbfc5-143">Il criterio di ereditarietà tabella per gerarchia è l'unico supportato da Entity Framework Core.</span><span class="sxs-lookup"><span data-stu-id="cbfc5-143">TPH is the only inheritance pattern that the Entity Framework Core supports.</span></span>  <span data-ttu-id="cbfc5-144">Verranno eseguite le operazioni di creazione di una classe `Person`, modifica delle classi `Instructor` e `Student` in modo che derivino da `Person`, aggiunta della nuova classe a `DbContext` e creazione di una migrazione.</span><span class="sxs-lookup"><span data-stu-id="cbfc5-144">What you'll do is create a `Person` class, change the `Instructor` and `Student` classes to derive from `Person`, add the new class to the `DbContext`, and create a migration.</span></span>

> [!TIP]
> <span data-ttu-id="cbfc5-145">È consigliabile salvare una copia del progetto prima di apportare le modifiche seguenti.</span><span class="sxs-lookup"><span data-stu-id="cbfc5-145">Consider saving a copy of the project before making the following changes.</span></span>  <span data-ttu-id="cbfc5-146">In caso di problemi e se fosse necessario ricominciare da capo, sarà più facile iniziare dal progetto salvato invece di annullare i passaggi eseguiti per questa esercitazione o tornare all'inizio dell'intera serie.</span><span class="sxs-lookup"><span data-stu-id="cbfc5-146">Then if you run into problems and need to start over, it will be easier to start from the saved project instead of reversing steps done for this tutorial or going back to the beginning of the whole series.</span></span>

## <a name="create-the-person-class"></a><span data-ttu-id="cbfc5-147">Creare la classe Person</span><span class="sxs-lookup"><span data-stu-id="cbfc5-147">Create the Person class</span></span>

<span data-ttu-id="cbfc5-148">Nella cartella Models creare Person.cs e sostituire il codice del modello con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="cbfc5-148">In the Models folder, create Person.cs and replace the template code with the following code:</span></span>

[!code-csharp[](intro/samples/cu/Models/Person.cs)]

## <a name="update-instructor-and-student"></a><span data-ttu-id="cbfc5-149">Aggiornare Student e Instructor</span><span class="sxs-lookup"><span data-stu-id="cbfc5-149">Update Instructor and Student</span></span>

<span data-ttu-id="cbfc5-150">In *Instructor.cs* derivare la classe Instructor dalla classe Person e rimuovere i campi chiave e nome.</span><span class="sxs-lookup"><span data-stu-id="cbfc5-150">In *Instructor.cs*, derive the Instructor class from the Person class and remove the key and name fields.</span></span> <span data-ttu-id="cbfc5-151">Il codice sarà simile all'esempio seguente:</span><span class="sxs-lookup"><span data-stu-id="cbfc5-151">The code will look like the following example:</span></span>

[!code-csharp[](intro/samples/cu/Models/Instructor.cs?name=snippet_AfterInheritance&highlight=8)]

<span data-ttu-id="cbfc5-152">Apportare le stesse modifiche in *Student.cs*.</span><span class="sxs-lookup"><span data-stu-id="cbfc5-152">Make the same changes in *Student.cs*.</span></span>

[!code-csharp[](intro/samples/cu/Models/Student.cs?name=snippet_AfterInheritance&highlight=8)]

## <a name="add-person-to-the-model"></a><span data-ttu-id="cbfc5-153">Aggiungere Person al modello</span><span class="sxs-lookup"><span data-stu-id="cbfc5-153">Add Person to the model</span></span>

<span data-ttu-id="cbfc5-154">Aggiungere il tipo di entità Person a *SchoolContext.cs*.</span><span class="sxs-lookup"><span data-stu-id="cbfc5-154">Add the Person entity type to *SchoolContext.cs*.</span></span> <span data-ttu-id="cbfc5-155">Le nuove righe sono evidenziate.</span><span class="sxs-lookup"><span data-stu-id="cbfc5-155">The new lines are highlighted.</span></span>

[!code-csharp[](intro/samples/cu/Data/SchoolContext.cs?name=snippet_AfterInheritance&highlight=19,30)]

<span data-ttu-id="cbfc5-156">L'ereditarietà tabella per gerarchia in Entity Framework è stata configurata.</span><span class="sxs-lookup"><span data-stu-id="cbfc5-156">This is all that the Entity Framework needs in order to configure table-per-hierarchy inheritance.</span></span> <span data-ttu-id="cbfc5-157">Come si vedrà, quando il database verrà aggiornato conterrà una tabella Person invece delle tabelle Student e Instructor.</span><span class="sxs-lookup"><span data-stu-id="cbfc5-157">As you'll see, when the database is updated, it will have a Person table in place of the Student and Instructor tables.</span></span>

## <a name="create-and-update-migrations"></a><span data-ttu-id="cbfc5-158">Creare e aggiornare le migrazioni</span><span class="sxs-lookup"><span data-stu-id="cbfc5-158">Create and update migrations</span></span>

<span data-ttu-id="cbfc5-159">Salvare le modifiche e compilare il progetto.</span><span class="sxs-lookup"><span data-stu-id="cbfc5-159">Save your changes and build the project.</span></span> <span data-ttu-id="cbfc5-160">Quindi aprire la finestra di comando nella cartella del progetto e immettere il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="cbfc5-160">Then open the command window in the project folder and enter the following command:</span></span>

```dotnetcli
dotnet ef migrations add Inheritance
```

<span data-ttu-id="cbfc5-161">Non eseguire ancora il comando `database update`.</span><span class="sxs-lookup"><span data-stu-id="cbfc5-161">Don't run the `database update` command yet.</span></span> <span data-ttu-id="cbfc5-162">Questo comando determinerà la perdita di dati poiché eliminerà la tabella Instructor e rinominerà la tabella Student in Person.</span><span class="sxs-lookup"><span data-stu-id="cbfc5-162">That command will result in lost data because it will drop the Instructor table and rename the Student table to Person.</span></span> <span data-ttu-id="cbfc5-163">Per mantenere i dati esistenti è necessario specificare codice personalizzato.</span><span class="sxs-lookup"><span data-stu-id="cbfc5-163">You need to provide custom code to preserve existing data.</span></span>

<span data-ttu-id="cbfc5-164">Aprire *Migrations/ \<timestamp> _Inheritance. cs* e sostituire il `Up` metodo con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="cbfc5-164">Open *Migrations/\<timestamp>_Inheritance.cs* and replace the `Up` method with the following code:</span></span>

[!code-csharp[](intro/samples/cu/Migrations/20170216215525_Inheritance.cs?name=snippet_Up)]

<span data-ttu-id="cbfc5-165">Questo codice esegue le attività di aggiornamento del database seguenti:</span><span class="sxs-lookup"><span data-stu-id="cbfc5-165">This code takes care of the following database update tasks:</span></span>

* <span data-ttu-id="cbfc5-166">Rimuove i vincoli di chiave esterna e gli indici che puntano alla tabella Student.</span><span class="sxs-lookup"><span data-stu-id="cbfc5-166">Removes foreign key constraints and indexes that point to the Student table.</span></span>

* <span data-ttu-id="cbfc5-167">Rinomina la tabella Instructor in Person e apporta le modifiche necessarie perché sia in grado di archiviare i dati Student:</span><span class="sxs-lookup"><span data-stu-id="cbfc5-167">Renames the Instructor table as Person and makes changes needed for it to store Student data:</span></span>

* <span data-ttu-id="cbfc5-168">Aggiunge un elemento EnrollmentDate che ammette i valori Null per gli studenti.</span><span class="sxs-lookup"><span data-stu-id="cbfc5-168">Adds nullable EnrollmentDate for students.</span></span>

* <span data-ttu-id="cbfc5-169">Aggiunge una colonna discriminante che indica se una riga è per uno studente o un docente.</span><span class="sxs-lookup"><span data-stu-id="cbfc5-169">Adds Discriminator column to indicate whether a row is for a student or an instructor.</span></span>

* <span data-ttu-id="cbfc5-170">Modifica HireDate in modo che ammetta i valori Null poiché le righe degli studenti non includono date di assunzione.</span><span class="sxs-lookup"><span data-stu-id="cbfc5-170">Makes HireDate nullable since student rows won't have hire dates.</span></span>

* <span data-ttu-id="cbfc5-171">Aggiunge un campo temporaneo che sarà usato per aggiornare le chiavi esterne che puntano agli studenti.</span><span class="sxs-lookup"><span data-stu-id="cbfc5-171">Adds a temporary field that will be used to update foreign keys that point to students.</span></span> <span data-ttu-id="cbfc5-172">Quando si copiano studenti nella tabella Person, questi otterranno nuovi valori di chiave primaria.</span><span class="sxs-lookup"><span data-stu-id="cbfc5-172">When you copy students into the Person table they will get new primary key values.</span></span>

* <span data-ttu-id="cbfc5-173">Copia i dati dalla tabella Student alla tabella Person.</span><span class="sxs-lookup"><span data-stu-id="cbfc5-173">Copies data from the Student table into the Person table.</span></span> <span data-ttu-id="cbfc5-174">Ciò comporta l'assegnazione di nuovi valori di chiave primaria agli studenti.</span><span class="sxs-lookup"><span data-stu-id="cbfc5-174">This causes students to get assigned new primary key values.</span></span>

* <span data-ttu-id="cbfc5-175">Corregge i valori di chiave esterna che puntano agli studenti.</span><span class="sxs-lookup"><span data-stu-id="cbfc5-175">Fixes foreign key values that point to students.</span></span>

* <span data-ttu-id="cbfc5-176">Ricrea i vincoli di chiave esterna e gli indici che ora puntano alla tabella Person.</span><span class="sxs-lookup"><span data-stu-id="cbfc5-176">Re-creates foreign key constraints and indexes, now pointing them to the Person table.</span></span>

<span data-ttu-id="cbfc5-177">Se come tipo di chiave primaria fosse stato usato GUID anziché Integer, non sarebbe necessario modificare i valori di chiave primaria degli studenti e molti di questi passaggi avrebbero potuto essere omessi.</span><span class="sxs-lookup"><span data-stu-id="cbfc5-177">(If you had used GUID instead of integer as the primary key type, the student primary key values wouldn't have to change, and several of these steps could have been omitted.)</span></span>

<span data-ttu-id="cbfc5-178">Eseguire il comando `database update`:</span><span class="sxs-lookup"><span data-stu-id="cbfc5-178">Run the `database update` command:</span></span>

```dotnetcli
dotnet ef database update
```

<span data-ttu-id="cbfc5-179">In un sistema di produzione verrebbero apportate modifiche corrispondenti al metodo `Down` nel caso fosse necessario usarlo per tornare alla versione precedente del database.</span><span class="sxs-lookup"><span data-stu-id="cbfc5-179">(In a production system you would make corresponding changes to the `Down` method in case you ever had to use that to go back to the previous database version.</span></span> <span data-ttu-id="cbfc5-180">Per questa esercitazione, il metodo `Down` non verrà usato.</span><span class="sxs-lookup"><span data-stu-id="cbfc5-180">For this tutorial you won't be using the `Down` method.)</span></span>

> [!NOTE]
> <span data-ttu-id="cbfc5-181">Quando si apportano modifiche allo schema in un database con dati esistenti è possibile che si riscontrino altri errori.</span><span class="sxs-lookup"><span data-stu-id="cbfc5-181">It's possible to get other errors when making schema changes in a database that has existing data.</span></span> <span data-ttu-id="cbfc5-182">Se si verificano errori di migrazione che non si riesce a risolvere, è possibile modificare il nome del database nella stringa di connessione o eliminare il database.</span><span class="sxs-lookup"><span data-stu-id="cbfc5-182">If you get migration errors that you can't resolve, you can either change the database name in the connection string or delete the database.</span></span> <span data-ttu-id="cbfc5-183">Un nuovo database non contiene dati di cui eseguire la migrazione e ci sono maggiori probabilità che il comando update-database venga completato senza errori.</span><span class="sxs-lookup"><span data-stu-id="cbfc5-183">With a new database, there's no data to migrate, and the update-database command is more likely to complete without errors.</span></span> <span data-ttu-id="cbfc5-184">Per eliminare il database, usare SSOX o eseguire il comando dell’interfaccia della riga di comando `database drop`.</span><span class="sxs-lookup"><span data-stu-id="cbfc5-184">To delete the database, use SSOX or run the `database drop` CLI command.</span></span>

## <a name="test-the-implementation"></a><span data-ttu-id="cbfc5-185">Testare l'implementazione</span><span class="sxs-lookup"><span data-stu-id="cbfc5-185">Test the implementation</span></span>

<span data-ttu-id="cbfc5-186">Eseguire l'app e provare diverse pagine.</span><span class="sxs-lookup"><span data-stu-id="cbfc5-186">Run the app and try various pages.</span></span> <span data-ttu-id="cbfc5-187">Tutto funziona come in precedenza.</span><span class="sxs-lookup"><span data-stu-id="cbfc5-187">Everything works the same as it did before.</span></span>

<span data-ttu-id="cbfc5-188">In **Esplora oggetti di SQL Server** espandere **Connessioni dati/SchoolContext** e quindi **Tabelle**. Si osserverà che le tabelle Student e Instructor sono state sostituite da una tabella Person.</span><span class="sxs-lookup"><span data-stu-id="cbfc5-188">In **SQL Server Object Explorer**, expand **Data Connections/SchoolContext** and then **Tables**, and you see that the Student and Instructor tables have been replaced by a Person table.</span></span> <span data-ttu-id="cbfc5-189">Dopo aver aperto la tabella Person in Progettazione tabelle si noterà che contiene tutte le colonne che in precedenza erano presenti nelle tabelle Student e Instructor.</span><span class="sxs-lookup"><span data-stu-id="cbfc5-189">Open the Person table designer and you see that it has all of the columns that used to be in the Student and Instructor tables.</span></span>

![Tabella Person in SSOX](inheritance/_static/ssox-person-table.png)

<span data-ttu-id="cbfc5-191">Fare clic con il pulsante destro del mouse sulla tabella Person e quindi su **Mostra dati tabella** per vedere la colonna discriminante.</span><span class="sxs-lookup"><span data-stu-id="cbfc5-191">Right-click the Person table, and then click **Show Table Data** to see the discriminator column.</span></span>

![Tabella Person in SSOX, dati della tabella](inheritance/_static/ssox-person-data.png)

## <a name="get-the-code"></a><span data-ttu-id="cbfc5-193">Ottenere il codice</span><span class="sxs-lookup"><span data-stu-id="cbfc5-193">Get the code</span></span>

[<span data-ttu-id="cbfc5-194">Scaricare o visualizzare l'applicazione completata.</span><span class="sxs-lookup"><span data-stu-id="cbfc5-194">Download or view the completed application.</span></span>](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/data/ef-mvc/intro/samples/cu-final)

## <a name="additional-resources"></a><span data-ttu-id="cbfc5-195">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="cbfc5-195">Additional resources</span></span>

<span data-ttu-id="cbfc5-196">Per altre informazioni sull'ereditarietà in Entity Framework Core, vedere [Ereditarietà](/ef/core/modeling/inheritance).</span><span class="sxs-lookup"><span data-stu-id="cbfc5-196">For more information about inheritance in Entity Framework Core, see [Inheritance](/ef/core/modeling/inheritance).</span></span>

## <a name="next-steps"></a><span data-ttu-id="cbfc5-197">Passaggi successivi</span><span class="sxs-lookup"><span data-stu-id="cbfc5-197">Next steps</span></span>

<span data-ttu-id="cbfc5-198">In questa esercitazione:</span><span class="sxs-lookup"><span data-stu-id="cbfc5-198">In this tutorial, you:</span></span>

> [!div class="checklist"]
> * <span data-ttu-id="cbfc5-199">Eseguire il mapping dell'ereditarietà al database</span><span class="sxs-lookup"><span data-stu-id="cbfc5-199">Mapped inheritance to database</span></span>
> * <span data-ttu-id="cbfc5-200">Creare la classe Person</span><span class="sxs-lookup"><span data-stu-id="cbfc5-200">Created the Person class</span></span>
> * <span data-ttu-id="cbfc5-201">Aggiornare Student e Instructor</span><span class="sxs-lookup"><span data-stu-id="cbfc5-201">Updated Instructor and Student</span></span>
> * <span data-ttu-id="cbfc5-202">Aggiungere Person al modello</span><span class="sxs-lookup"><span data-stu-id="cbfc5-202">Added Person to the model</span></span>
> * <span data-ttu-id="cbfc5-203">Creare e aggiornare le migrazioni</span><span class="sxs-lookup"><span data-stu-id="cbfc5-203">Created and update migrations</span></span>
> * <span data-ttu-id="cbfc5-204">Testare l'implementazione</span><span class="sxs-lookup"><span data-stu-id="cbfc5-204">Tested the implementation</span></span>

<span data-ttu-id="cbfc5-205">Passare all'esercitazione successiva per apprendere come gestire diversi scenari di Entity Framework relativamente avanzati.</span><span class="sxs-lookup"><span data-stu-id="cbfc5-205">Advance to the next tutorial to learn how to handle a variety of relatively advanced Entity Framework scenarios.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="cbfc5-206">Avanti: argomenti avanzati</span><span class="sxs-lookup"><span data-stu-id="cbfc5-206">Next: Advanced topics</span></span>](advanced.md)