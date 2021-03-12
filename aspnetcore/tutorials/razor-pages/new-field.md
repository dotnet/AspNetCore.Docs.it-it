---
title: Parte 7, aggiungere un nuovo campo
author: rick-anderson
description: Parte 7 della serie di esercitazioni sulle Razor pagine.
ms.author: riande
ms.custom: mvc, contperf-fy21q2
ms.date: 09/28/2020
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
uid: tutorials/razor-pages/new-field
ms.openlocfilehash: cd637507e19735f020b4c28e6f22de0e7e772040
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102588321"
---
# <a name="part-7-add-a-new-field-to-a-razor-page-in-aspnet-core"></a><span data-ttu-id="90f05-103">Parte 7, aggiungere un nuovo campo a una Razor pagina in ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="90f05-103">Part 7, add a new field to a Razor Page in ASP.NET Core</span></span>

<span data-ttu-id="90f05-104">Autore: [Rick Anderson](https://twitter.com/RickAndMSFT)</span><span class="sxs-lookup"><span data-stu-id="90f05-104">By [Rick Anderson](https://twitter.com/RickAndMSFT)</span></span>

::: moniker range=">= aspnetcore-5.0"

<span data-ttu-id="90f05-105">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie50) ([procedura per il download](xref:index#how-to-download-a-sample)).</span><span class="sxs-lookup"><span data-stu-id="90f05-105">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie50) ([how to download](xref:index#how-to-download-a-sample)).</span></span>

<span data-ttu-id="90f05-106">In questa sezione vengono usate le Migrazioni Code First di [Entity Framework](/ef/core/get-started/aspnetcore/new-db) per:</span><span class="sxs-lookup"><span data-stu-id="90f05-106">In this section [Entity Framework](/ef/core/get-started/aspnetcore/new-db) Code First Migrations is used to:</span></span>

* <span data-ttu-id="90f05-107">Aggiungere un nuovo campo al modello.</span><span class="sxs-lookup"><span data-stu-id="90f05-107">Add a new field to the model.</span></span>
* <span data-ttu-id="90f05-108">Eseguire la migrazione nel database della modifica al nuovo schema del campo.</span><span class="sxs-lookup"><span data-stu-id="90f05-108">Migrate the new field schema change to the database.</span></span>

<span data-ttu-id="90f05-109">Quando si usa Code First di Entity Framework per creare automaticamente un database, Code First:</span><span class="sxs-lookup"><span data-stu-id="90f05-109">When using EF Code First to automatically create a database, Code First:</span></span>

* <span data-ttu-id="90f05-110">Aggiunge una [`__EFMigrationsHistory`](/ef/core/managing-schemas/migrations/history-table) tabella al database per rilevare se lo schema del database è sincronizzato con le classi del modello da cui è stato generato.</span><span class="sxs-lookup"><span data-stu-id="90f05-110">Adds an [`__EFMigrationsHistory`](/ef/core/managing-schemas/migrations/history-table) table to the database to track whether the schema of the database is in sync with the model classes it was generated from.</span></span>
* <span data-ttu-id="90f05-111">Se le classi del modello non sono sincronizzate con il database, EF genera un'eccezione.</span><span class="sxs-lookup"><span data-stu-id="90f05-111">If the model classes aren't in sync with the database, EF throws an exception.</span></span>

<span data-ttu-id="90f05-112">Con la verifica automatica che lo schema e il modello sono sincronizzati, è più semplice individuare problemi di codice di database incoerenti.</span><span class="sxs-lookup"><span data-stu-id="90f05-112">Automatic verification that the schema and model are in sync makes it easier to find inconsistent database code issues.</span></span>

## <a name="adding-a-rating-property-to-the-movie-model"></a><span data-ttu-id="90f05-113">Aggiunta di una proprietà Rating al modello Movie</span><span class="sxs-lookup"><span data-stu-id="90f05-113">Adding a Rating Property to the Movie Model</span></span>

1. <span data-ttu-id="90f05-114">Aprire il file *Models/Movie.cs* e aggiungere una proprietà `Rating`:</span><span class="sxs-lookup"><span data-stu-id="90f05-114">Open the *Models/Movie.cs* file and add a `Rating` property:</span></span>

   [!code-csharp[](razor-pages-start/sample/RazorPagesMovie50/Models/MovieDateRating.cs?highlight=13&name=snippet)]

1. <span data-ttu-id="90f05-115">Compilare l'app.</span><span class="sxs-lookup"><span data-stu-id="90f05-115">Build the app.</span></span>

1. <span data-ttu-id="90f05-116">Modificare *pages/Movies/ Index . cshtml* e aggiungere un `Rating` campo:</span><span class="sxs-lookup"><span data-stu-id="90f05-116">Edit *Pages/Movies/Index.cshtml*, and add a `Rating` field:</span></span>

   <a name="addrat"></a>

   [!code-cshtml[](razor-pages-start/sample/RazorPagesMovie50/SnapShots/IndexRating.cshtml?highlight=40-42,62-64)]

1. <span data-ttu-id="90f05-117">Aggiornare le pagine seguenti:</span><span class="sxs-lookup"><span data-stu-id="90f05-117">Update the following pages:</span></span>
   1. <span data-ttu-id="90f05-118">Aggiungere il campo `Rating` alle pagine Delete (Elimina) e Details (Dettagli).</span><span class="sxs-lookup"><span data-stu-id="90f05-118">Add the `Rating` field to the Delete and Details pages.</span></span>
   1. <span data-ttu-id="90f05-119">Aggiornare [Create.cshtml](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie50/Pages/Movies/Create.cshtml) con un campo `Rating`.</span><span class="sxs-lookup"><span data-stu-id="90f05-119">Update [Create.cshtml](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie50/Pages/Movies/Create.cshtml) with a `Rating` field.</span></span>
   1. <span data-ttu-id="90f05-120">Aggiungere il campo `Rating` alla pagina Edit (Modifica).</span><span class="sxs-lookup"><span data-stu-id="90f05-120">Add the `Rating` field to the Edit Page.</span></span>

<span data-ttu-id="90f05-121">L'app non funzionerà finché il database non verrà aggiornato in modo da includere il nuovo campo.</span><span class="sxs-lookup"><span data-stu-id="90f05-121">The app won't work until the database is updated to include the new field.</span></span> <span data-ttu-id="90f05-122">L'esecuzione dell'app senza un aggiornamento del database genera un'eccezione `SqlException` :</span><span class="sxs-lookup"><span data-stu-id="90f05-122">Running the app without an update to the database throws a `SqlException`:</span></span>

`SqlException: Invalid column name 'Rating'.`

<span data-ttu-id="90f05-123">L' `SqlException` eccezione è causata dal fatto che la classe del modello di film aggiornata è diversa dallo schema della tabella dei film del database.</span><span class="sxs-lookup"><span data-stu-id="90f05-123">The `SqlException` exception is caused by the updated Movie model class being different than the schema of the Movie table of the database.</span></span> <span data-ttu-id="90f05-124">Non è presente alcuna `Rating` colonna nella tabella di database.</span><span class="sxs-lookup"><span data-stu-id="90f05-124">There's no `Rating` column in the database table.</span></span>

<span data-ttu-id="90f05-125">Per correggere questo errore, esistono alcuni approcci:</span><span class="sxs-lookup"><span data-stu-id="90f05-125">There are a few approaches to resolving the error:</span></span>

1. <span data-ttu-id="90f05-126">Fare in modo che Entity Framework elimini e crei di nuovo automaticamente il database usando il nuovo schema di classi del modello.</span><span class="sxs-lookup"><span data-stu-id="90f05-126">Have the Entity Framework automatically drop and re-create the database using the new model class schema.</span></span> <span data-ttu-id="90f05-127">Questo approccio è utile nelle prime fasi del ciclo di sviluppo e consente di sviluppare rapidamente lo schema del modello e del database insieme.</span><span class="sxs-lookup"><span data-stu-id="90f05-127">This approach is convenient early in the development cycle, it allows you to quickly evolve the model and database schema together.</span></span> <span data-ttu-id="90f05-128">Lo svantaggio è che si perdono i dati esistenti nel database.</span><span class="sxs-lookup"><span data-stu-id="90f05-128">The downside is that you lose existing data in the database.</span></span> <span data-ttu-id="90f05-129">Non usare questo approccio in un database di produzione.</span><span class="sxs-lookup"><span data-stu-id="90f05-129">Don't use this approach on a production database!</span></span> <span data-ttu-id="90f05-130">L'eliminazione del database in caso di modifiche dello schema e l'utilizzo di un inizializzatore per eseguire automaticamente il seeding del database con dati di test è spesso un modo produttivo per sviluppare un'app.</span><span class="sxs-lookup"><span data-stu-id="90f05-130">Dropping the database on schema changes and using an initializer to automatically seed the database with test data is often a productive way to develop an app.</span></span>

2. <span data-ttu-id="90f05-131">Modificare esplicitamente lo schema del database esistente in modo che corrisponda alle classi del modello.</span><span class="sxs-lookup"><span data-stu-id="90f05-131">Explicitly modify the schema of the existing database so that it matches the model classes.</span></span> <span data-ttu-id="90f05-132">Il vantaggio di questo approccio consiste nel preservare i dati.</span><span class="sxs-lookup"><span data-stu-id="90f05-132">The advantage of this approach is to keep the data.</span></span> <span data-ttu-id="90f05-133">Apportare questa modifica manualmente o creando uno script di modifica del database.</span><span class="sxs-lookup"><span data-stu-id="90f05-133">Make this change either manually or by creating a database change script.</span></span>

3. <span data-ttu-id="90f05-134">Usare Migrazioni Code First per aggiornare lo schema del database.</span><span class="sxs-lookup"><span data-stu-id="90f05-134">Use Code First Migrations to update the database schema.</span></span>

<span data-ttu-id="90f05-135">Per questa esercitazione usare Migrazioni Code First.</span><span class="sxs-lookup"><span data-stu-id="90f05-135">For this tutorial, use Code First Migrations.</span></span>

<span data-ttu-id="90f05-136">Aggiornare la classe `SeedData` in modo che fornisca un valore per la nuova colonna.</span><span class="sxs-lookup"><span data-stu-id="90f05-136">Update the `SeedData` class so that it provides a value for the new column.</span></span> <span data-ttu-id="90f05-137">Di seguito viene illustrata una modifica di esempio, ma questa modifica viene apportata per ogni `new Movie` blocco.</span><span class="sxs-lookup"><span data-stu-id="90f05-137">A sample change is shown below, but make this change for each `new Movie` block.</span></span>

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie50/Models/SeedDataRating.cs?name=snippet1&highlight=8)]

<span data-ttu-id="90f05-138">Vedere il [file SeedData.cs completato](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie50/Models/SeedDataRating.cs).</span><span class="sxs-lookup"><span data-stu-id="90f05-138">See the [completed SeedData.cs file](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie50/Models/SeedDataRating.cs).</span></span>

<span data-ttu-id="90f05-139">Compilare la soluzione.</span><span class="sxs-lookup"><span data-stu-id="90f05-139">Build the solution.</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="90f05-140">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="90f05-140">Visual Studio</span></span>](#tab/visual-studio)

<a name="pmc"></a>

### <a name="add-a-migration-for-the-rating-field"></a><span data-ttu-id="90f05-141">Aggiungere una migrazione per il campo Rating</span><span class="sxs-lookup"><span data-stu-id="90f05-141">Add a migration for the rating field</span></span>

1. <span data-ttu-id="90f05-142">Dal menu **Strumenti** selezionare **Gestione pacchetti NuGet > Console di Gestione pacchetti**.</span><span class="sxs-lookup"><span data-stu-id="90f05-142">From the **Tools** menu, select **NuGet Package Manager > Package Manager Console**.</span></span>
2. <span data-ttu-id="90f05-143">Nella Console di Gestione pacchetti immettere i comandi seguenti:</span><span class="sxs-lookup"><span data-stu-id="90f05-143">In the PMC, enter the following commands:</span></span>

   ```powershell
   Add-Migration Rating
   Update-Database
   ```

<span data-ttu-id="90f05-144">Il comando `Add-Migration` indica al framework di:</span><span class="sxs-lookup"><span data-stu-id="90f05-144">The `Add-Migration` command tells the framework to:</span></span>

* <span data-ttu-id="90f05-145">Confrontare il `Movie` modello con lo `Movie` schema del database.</span><span class="sxs-lookup"><span data-stu-id="90f05-145">Compare the `Movie` model with the `Movie` database schema.</span></span>
* <span data-ttu-id="90f05-146">Creare il codice per eseguire la migrazione dello schema del database al nuovo modello.</span><span class="sxs-lookup"><span data-stu-id="90f05-146">Create code to migrate the database schema to the new model.</span></span>

<span data-ttu-id="90f05-147">Il nome "Rating" è arbitrario e viene usato per denominare il file di migrazione.</span><span class="sxs-lookup"><span data-stu-id="90f05-147">The name "Rating" is arbitrary and is used to name the migration file.</span></span> <span data-ttu-id="90f05-148">È consigliabile usare un nome significativo per il file di migrazione.</span><span class="sxs-lookup"><span data-stu-id="90f05-148">It's helpful to use a meaningful name for the migration file.</span></span>

<span data-ttu-id="90f05-149">Il `Update-Database` comando indica al Framework di applicare le modifiche dello schema al database e di mantenere i dati esistenti.</span><span class="sxs-lookup"><span data-stu-id="90f05-149">The `Update-Database` command tells the framework to apply the schema changes to the database and to preserve existing data.</span></span>

<a name="ssox"></a>

<span data-ttu-id="90f05-150">Se si eliminano tutti i record nel database, l'inizializzatore eseguirà il seeding del database e includerà il `Rating` campo.</span><span class="sxs-lookup"><span data-stu-id="90f05-150">If you delete all the records in the database, the initializer will seed the database and include the `Rating` field.</span></span> <span data-ttu-id="90f05-151">È possibile eseguire questa operazione con i collegamenti di eliminazione nel browser o da [Sql Server Object Explorer](xref:tutorials/razor-pages/sql#ssox) (SSOX).</span><span class="sxs-lookup"><span data-stu-id="90f05-151">You can do this with the delete links in the browser or from [Sql Server Object Explorer](xref:tutorials/razor-pages/sql#ssox) (SSOX).</span></span>

<span data-ttu-id="90f05-152">Un'altra opzione è quella di eliminare il database e usare le migrazioni per ricreare il database.</span><span class="sxs-lookup"><span data-stu-id="90f05-152">Another option is to delete the database and use migrations to re-create the database.</span></span> <span data-ttu-id="90f05-153">Per eliminare il database da SSOX:</span><span class="sxs-lookup"><span data-stu-id="90f05-153">To delete the database in SSOX:</span></span>

1. <span data-ttu-id="90f05-154">Selezionare il database in SSOX.</span><span class="sxs-lookup"><span data-stu-id="90f05-154">Select the database in SSOX.</span></span>
1. <span data-ttu-id="90f05-155">Fare clic con il pulsante destro del mouse sul database e scegliere **Elimina**.</span><span class="sxs-lookup"><span data-stu-id="90f05-155">Right-click on the database, and select **Delete**.</span></span>
1. <span data-ttu-id="90f05-156">Selezionare **Chiudi connessioni esistenti**.</span><span class="sxs-lookup"><span data-stu-id="90f05-156">Check **Close existing connections**.</span></span>
1. <span data-ttu-id="90f05-157">Selezionare **OK**.</span><span class="sxs-lookup"><span data-stu-id="90f05-157">Select **OK**.</span></span>
1. <span data-ttu-id="90f05-158">In [PMC](xref:tutorials/razor-pages/new-field#pmc)aggiornare il database:</span><span class="sxs-lookup"><span data-stu-id="90f05-158">In the [PMC](xref:tutorials/razor-pages/new-field#pmc), update the database:</span></span>

   ```powershell
   Update-Database
   ```

# <a name="visual-studio-code--visual-studio-for-mac"></a>[<span data-ttu-id="90f05-159">Visual Studio Code / Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="90f05-159">Visual Studio Code / Visual Studio for Mac</span></span>](#tab/visual-studio-code+visual-studio-mac)

### <a name="drop-and-re-create-the-database"></a><span data-ttu-id="90f05-160">Eliminare e ricreare il database</span><span class="sxs-lookup"><span data-stu-id="90f05-160">Drop and re-create the database</span></span>

> [!NOTE]
> <span data-ttu-id="90f05-161">Per questa esercitazione, è possibile usare la funzionalità *migrazioni* di Entity Framework Core, ove possibile.</span><span class="sxs-lookup"><span data-stu-id="90f05-161">For this tutorial, you use the Entity Framework Core *migrations* feature where possible.</span></span> <span data-ttu-id="90f05-162">Le migrazioni aggiornano lo schema del database in base alle modifiche nel modello di dati.</span><span class="sxs-lookup"><span data-stu-id="90f05-162">Migrations updates the database schema to match changes in the data model.</span></span> <span data-ttu-id="90f05-163">Tuttavia, le migrazioni possono eseguire solo i tipi di modifiche supportate dal provider EF Core e le funzionalità del provider SQLite sono limitate.</span><span class="sxs-lookup"><span data-stu-id="90f05-163">However, migrations can only do the kinds of changes that the EF Core provider supports, and the SQLite provider's capabilities are limited.</span></span> <span data-ttu-id="90f05-164">Ad esempio l'aggiunta di una colonna è supportata, ma la rimozione o la modifica di una colonna non è supportata.</span><span class="sxs-lookup"><span data-stu-id="90f05-164">For example, adding a column is supported, but removing or changing a column is not supported.</span></span> <span data-ttu-id="90f05-165">Se si crea una migrazione per rimuovere o modificare una colonna, il comando `ef migrations add` ha esito positivo, ma il comando `ef database update` ha esito negativo.</span><span class="sxs-lookup"><span data-stu-id="90f05-165">If a migration is created to remove or change a column, the `ef migrations add` command succeeds but the `ef database update` command fails.</span></span> <span data-ttu-id="90f05-166">A causa di queste limitazioni, in questa esercitazione non vengono usate le migrazioni per le modifiche dello schema di SQLite.</span><span class="sxs-lookup"><span data-stu-id="90f05-166">Due to these limitations, this tutorial doesn't use migrations for SQLite schema changes.</span></span> <span data-ttu-id="90f05-167">In caso di modifiche dello schema, il database viene invece eliminato e ricreato.</span><span class="sxs-lookup"><span data-stu-id="90f05-167">Instead, when the schema changes, you drop and re-create the database.</span></span>
>
><span data-ttu-id="90f05-168">La soluzione per ovviare alle limitazioni di SQLite consiste nello scrivere manualmente il codice delle migrazioni per eseguire una ricompilazione della tabella in caso di modifiche nella tabella.</span><span class="sxs-lookup"><span data-stu-id="90f05-168">The workaround for the SQLite limitations is to manually write migrations code to perform a table rebuild when something in the table changes.</span></span> <span data-ttu-id="90f05-169">La ricompilazione della tabella comporta:</span><span class="sxs-lookup"><span data-stu-id="90f05-169">A table rebuild involves:</span></span>
>
>* <span data-ttu-id="90f05-170">Creazione di una nuova tabella.</span><span class="sxs-lookup"><span data-stu-id="90f05-170">Creating a new table.</span></span>
>* <span data-ttu-id="90f05-171">Copia dei dati dalla vecchia tabella alla nuova tabella.</span><span class="sxs-lookup"><span data-stu-id="90f05-171">Copying data from the old table to the new table.</span></span>
>* <span data-ttu-id="90f05-172">Eliminazione della tabella precedente.</span><span class="sxs-lookup"><span data-stu-id="90f05-172">Dropping the old table.</span></span>
>* <span data-ttu-id="90f05-173">Ridenominazione della nuova tabella.</span><span class="sxs-lookup"><span data-stu-id="90f05-173">Renaming the new table.</span></span>
>
><span data-ttu-id="90f05-174">Per altre informazioni, vedere le risorse seguenti:</span><span class="sxs-lookup"><span data-stu-id="90f05-174">For more information, see the following resources:</span></span>
>
> * [<span data-ttu-id="90f05-175">Limitazioni del provider di database SQLite per EF Core</span><span class="sxs-lookup"><span data-stu-id="90f05-175">SQLite EF Core Database Provider Limitations</span></span>](/ef/core/providers/sqlite/limitations)
> * [<span data-ttu-id="90f05-176">Personalizzare il codice di migrazione</span><span class="sxs-lookup"><span data-stu-id="90f05-176">Customize migration code</span></span>](/ef/core/managing-schemas/migrations/#customize-migration-code)
> * [<span data-ttu-id="90f05-177">Seeding dei dati</span><span class="sxs-lookup"><span data-stu-id="90f05-177">Data seeding</span></span>](/ef/core/modeling/data-seeding)
> * [<span data-ttu-id="90f05-178">Istruzione ALTER TABLE di SQLite</span><span class="sxs-lookup"><span data-stu-id="90f05-178">SQLite ALTER TABLE statement</span></span>](https://sqlite.org/lang_altertable.html)

1. <span data-ttu-id="90f05-179">Eliminare la cartella della migrazione.</span><span class="sxs-lookup"><span data-stu-id="90f05-179">Delete the migration folder.</span></span>  

1. <span data-ttu-id="90f05-180">Usare i comandi seguenti per ricreare il database.</span><span class="sxs-lookup"><span data-stu-id="90f05-180">Use the following commands to recreate the database.</span></span>

   ```dotnetcli
   dotnet ef database drop
   dotnet ef migrations add InitialCreate
   dotnet ef database update
   ```

---

<span data-ttu-id="90f05-181">Eseguire l'app e verificare che sia possibile creare/modificare/visualizzare i film con un campo `Rating`.</span><span class="sxs-lookup"><span data-stu-id="90f05-181">Run the app and verify you can create/edit/display movies with a `Rating` field.</span></span> <span data-ttu-id="90f05-182">Se il database non è inizializzato, impostare un punto di interruzione nel metodo `SeedData.Initialize`.</span><span class="sxs-lookup"><span data-stu-id="90f05-182">If the database isn't seeded, set a break point in the `SeedData.Initialize` method.</span></span>

## <a name="additional-resources"></a><span data-ttu-id="90f05-183">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="90f05-183">Additional resources</span></span>

> [!div class="step-by-step"]
> <span data-ttu-id="90f05-184">[Precedente: Aggiungi ricerca](xref:tutorials/razor-pages/search) 
>  Passaggio [successivo: aggiungere la convalida](xref:tutorials/razor-pages/validation)</span><span class="sxs-lookup"><span data-stu-id="90f05-184">[Previous: Add Search](xref:tutorials/razor-pages/search)
[Next: Add Validation](xref:tutorials/razor-pages/validation)</span></span>

::: moniker-end

::: moniker range="< aspnetcore-5.0 >= aspnetcore-3.0"

<span data-ttu-id="90f05-185">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie30) ([procedura per il download](xref:index#how-to-download-a-sample)).</span><span class="sxs-lookup"><span data-stu-id="90f05-185">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie30) ([how to download](xref:index#how-to-download-a-sample)).</span></span>

<span data-ttu-id="90f05-186">In questa sezione vengono usate le Migrazioni Code First di [Entity Framework](/ef/core/get-started/aspnetcore/new-db) per:</span><span class="sxs-lookup"><span data-stu-id="90f05-186">In this section [Entity Framework](/ef/core/get-started/aspnetcore/new-db) Code First Migrations is used to:</span></span>

* <span data-ttu-id="90f05-187">Aggiungere un nuovo campo al modello.</span><span class="sxs-lookup"><span data-stu-id="90f05-187">Add a new field to the model.</span></span>
* <span data-ttu-id="90f05-188">Eseguire la migrazione nel database della modifica al nuovo schema del campo.</span><span class="sxs-lookup"><span data-stu-id="90f05-188">Migrate the new field schema change to the database.</span></span>

<span data-ttu-id="90f05-189">Quando si usa Code First di Entity Framework per creare automaticamente un database, Code First:</span><span class="sxs-lookup"><span data-stu-id="90f05-189">When using EF Code First to automatically create a database, Code First:</span></span>

* <span data-ttu-id="90f05-190">Aggiunge una [`__EFMigrationsHistory`](/ef/core/managing-schemas/migrations/history-table) tabella al database per rilevare se lo schema del database è sincronizzato con le classi del modello da cui è stato generato.</span><span class="sxs-lookup"><span data-stu-id="90f05-190">Adds an [`__EFMigrationsHistory`](/ef/core/managing-schemas/migrations/history-table) table to the database to track whether the schema of the database is in sync with the model classes it was generated from.</span></span>
* <span data-ttu-id="90f05-191">Se le classi del modello non sono sincronizzate con il database, EF genera un'eccezione.</span><span class="sxs-lookup"><span data-stu-id="90f05-191">If the model classes aren't in sync with the database, EF throws an exception.</span></span>

<span data-ttu-id="90f05-192">Con la verifica automatica che lo schema e il modello sono sincronizzati, è più semplice individuare problemi di codice di database incoerenti.</span><span class="sxs-lookup"><span data-stu-id="90f05-192">Automatic verification that the schema and model are in sync makes it easier to find inconsistent database code issues.</span></span>

## <a name="adding-a-rating-property-to-the-movie-model"></a><span data-ttu-id="90f05-193">Aggiunta di una proprietà Rating al modello Movie</span><span class="sxs-lookup"><span data-stu-id="90f05-193">Adding a Rating Property to the Movie Model</span></span>

1. <span data-ttu-id="90f05-194">Aprire il file *Models/Movie.cs* e aggiungere una proprietà `Rating`:</span><span class="sxs-lookup"><span data-stu-id="90f05-194">Open the *Models/Movie.cs* file and add a `Rating` property:</span></span>

   [!code-csharp[](razor-pages-start/sample/RazorPagesMovie30/Models/MovieDateRating.cs?highlight=13&name=snippet)]

1. <span data-ttu-id="90f05-195">Compilare l'app.</span><span class="sxs-lookup"><span data-stu-id="90f05-195">Build the app.</span></span>

1. <span data-ttu-id="90f05-196">Modificare *pages/Movies/ Index . cshtml* e aggiungere un `Rating` campo:</span><span class="sxs-lookup"><span data-stu-id="90f05-196">Edit *Pages/Movies/Index.cshtml*, and add a `Rating` field:</span></span>

   <a name="addrat"></a>

   [!code-cshtml[](razor-pages-start/sample/RazorPagesMovie30/SnapShots/IndexRating.cshtml?highlight=40-42,62-64)]

1. <span data-ttu-id="90f05-197">Aggiornare le pagine seguenti:</span><span class="sxs-lookup"><span data-stu-id="90f05-197">Update the following pages:</span></span>
   1. <span data-ttu-id="90f05-198">Aggiungere il campo `Rating` alle pagine Delete (Elimina) e Details (Dettagli).</span><span class="sxs-lookup"><span data-stu-id="90f05-198">Add the `Rating` field to the Delete and Details pages.</span></span>
   1. <span data-ttu-id="90f05-199">Aggiornare [Create.cshtml](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie30/Pages/Movies/Create.cshtml) con un campo `Rating`.</span><span class="sxs-lookup"><span data-stu-id="90f05-199">Update [Create.cshtml](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie30/Pages/Movies/Create.cshtml) with a `Rating` field.</span></span>
   1. <span data-ttu-id="90f05-200">Aggiungere il campo `Rating` alla pagina Edit (Modifica).</span><span class="sxs-lookup"><span data-stu-id="90f05-200">Add the `Rating` field to the Edit Page.</span></span>

<span data-ttu-id="90f05-201">L'app non funzionerà finché il database non verrà aggiornato in modo da includere il nuovo campo.</span><span class="sxs-lookup"><span data-stu-id="90f05-201">The app won't work until the database is updated to include the new field.</span></span> <span data-ttu-id="90f05-202">L'esecuzione dell'app senza un aggiornamento del database genera un'eccezione `SqlException` :</span><span class="sxs-lookup"><span data-stu-id="90f05-202">Running the app without an update to the database throws a `SqlException`:</span></span>

`SqlException: Invalid column name 'Rating'.`

<span data-ttu-id="90f05-203">L' `SqlException` eccezione è causata dal fatto che la classe del modello di film aggiornata è diversa dallo schema della tabella dei film del database.</span><span class="sxs-lookup"><span data-stu-id="90f05-203">The `SqlException` exception is caused by the updated Movie model class being different than the schema of the Movie table of the database.</span></span> <span data-ttu-id="90f05-204">Non è presente alcuna `Rating` colonna nella tabella di database.</span><span class="sxs-lookup"><span data-stu-id="90f05-204">There's no `Rating` column in the database table.</span></span>

<span data-ttu-id="90f05-205">Per correggere questo errore, esistono alcuni approcci:</span><span class="sxs-lookup"><span data-stu-id="90f05-205">There are a few approaches to resolving the error:</span></span>

1. <span data-ttu-id="90f05-206">Fare in modo che Entity Framework elimini e crei di nuovo automaticamente il database usando il nuovo schema di classi del modello.</span><span class="sxs-lookup"><span data-stu-id="90f05-206">Have the Entity Framework automatically drop and re-create the database using the new model class schema.</span></span> <span data-ttu-id="90f05-207">Questo approccio è utile nelle prime fasi del ciclo di sviluppo e consente di sviluppare rapidamente lo schema del modello e del database insieme.</span><span class="sxs-lookup"><span data-stu-id="90f05-207">This approach is convenient early in the development cycle, it allows you to quickly evolve the model and database schema together.</span></span> <span data-ttu-id="90f05-208">Lo svantaggio è che si perdono i dati esistenti nel database.</span><span class="sxs-lookup"><span data-stu-id="90f05-208">The downside is that you lose existing data in the database.</span></span> <span data-ttu-id="90f05-209">Non usare questo approccio in un database di produzione.</span><span class="sxs-lookup"><span data-stu-id="90f05-209">Don't use this approach on a production database!</span></span> <span data-ttu-id="90f05-210">L'eliminazione del database in caso di modifiche dello schema e l'utilizzo di un inizializzatore per eseguire automaticamente il seeding del database con dati di test è spesso un modo produttivo per sviluppare un'app.</span><span class="sxs-lookup"><span data-stu-id="90f05-210">Dropping the database on schema changes and using an initializer to automatically seed the database with test data is often a productive way to develop an app.</span></span>

2. <span data-ttu-id="90f05-211">Modificare esplicitamente lo schema del database esistente in modo che corrisponda alle classi del modello.</span><span class="sxs-lookup"><span data-stu-id="90f05-211">Explicitly modify the schema of the existing database so that it matches the model classes.</span></span> <span data-ttu-id="90f05-212">Il vantaggio di questo approccio consiste nel preservare i dati.</span><span class="sxs-lookup"><span data-stu-id="90f05-212">The advantage of this approach is to keep the data.</span></span> <span data-ttu-id="90f05-213">Apportare questa modifica manualmente o creando uno script di modifica del database.</span><span class="sxs-lookup"><span data-stu-id="90f05-213">Make this change either manually or by creating a database change script.</span></span>

3. <span data-ttu-id="90f05-214">Usare Migrazioni Code First per aggiornare lo schema del database.</span><span class="sxs-lookup"><span data-stu-id="90f05-214">Use Code First Migrations to update the database schema.</span></span>

<span data-ttu-id="90f05-215">Per questa esercitazione usare Migrazioni Code First.</span><span class="sxs-lookup"><span data-stu-id="90f05-215">For this tutorial, use Code First Migrations.</span></span>

<span data-ttu-id="90f05-216">Aggiornare la classe `SeedData` in modo che fornisca un valore per la nuova colonna.</span><span class="sxs-lookup"><span data-stu-id="90f05-216">Update the `SeedData` class so that it provides a value for the new column.</span></span> <span data-ttu-id="90f05-217">Di seguito viene illustrata una modifica di esempio, ma questa modifica viene apportata per ogni `new Movie` blocco.</span><span class="sxs-lookup"><span data-stu-id="90f05-217">A sample change is shown below, but make this change for each `new Movie` block.</span></span>

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie30/Models/SeedDataRating.cs?name=snippet1&highlight=8)]

<span data-ttu-id="90f05-218">Vedere il [file SeedData.cs completato](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie50/Models/SeedDataRating.cs).</span><span class="sxs-lookup"><span data-stu-id="90f05-218">See the [completed SeedData.cs file](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie50/Models/SeedDataRating.cs).</span></span>

<span data-ttu-id="90f05-219">Compilare la soluzione.</span><span class="sxs-lookup"><span data-stu-id="90f05-219">Build the solution.</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="90f05-220">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="90f05-220">Visual Studio</span></span>](#tab/visual-studio)

<a name="pmc"></a>

### <a name="add-a-migration-for-the-rating-field"></a><span data-ttu-id="90f05-221">Aggiungere una migrazione per il campo Rating</span><span class="sxs-lookup"><span data-stu-id="90f05-221">Add a migration for the rating field</span></span>

1. <span data-ttu-id="90f05-222">Dal menu **Strumenti** selezionare **Gestione pacchetti NuGet > Console di Gestione pacchetti**.</span><span class="sxs-lookup"><span data-stu-id="90f05-222">From the **Tools** menu, select **NuGet Package Manager > Package Manager Console**.</span></span>
2. <span data-ttu-id="90f05-223">Nella Console di Gestione pacchetti immettere i comandi seguenti:</span><span class="sxs-lookup"><span data-stu-id="90f05-223">In the PMC, enter the following commands:</span></span>

   ```powershell
   Add-Migration Rating
   Update-Database
   ```

<span data-ttu-id="90f05-224">Il comando `Add-Migration` indica al framework di:</span><span class="sxs-lookup"><span data-stu-id="90f05-224">The `Add-Migration` command tells the framework to:</span></span>

* <span data-ttu-id="90f05-225">Confrontare il `Movie` modello con lo `Movie` schema del database.</span><span class="sxs-lookup"><span data-stu-id="90f05-225">Compare the `Movie` model with the `Movie` database schema.</span></span>
* <span data-ttu-id="90f05-226">Creare il codice per eseguire la migrazione dello schema del database al nuovo modello.</span><span class="sxs-lookup"><span data-stu-id="90f05-226">Create code to migrate the database schema to the new model.</span></span>

<span data-ttu-id="90f05-227">Il nome "Rating" è arbitrario e viene usato per denominare il file di migrazione.</span><span class="sxs-lookup"><span data-stu-id="90f05-227">The name "Rating" is arbitrary and is used to name the migration file.</span></span> <span data-ttu-id="90f05-228">È consigliabile usare un nome significativo per il file di migrazione.</span><span class="sxs-lookup"><span data-stu-id="90f05-228">It's helpful to use a meaningful name for the migration file.</span></span>

<span data-ttu-id="90f05-229">Il `Update-Database` comando indica al Framework di applicare le modifiche dello schema al database e di mantenere i dati esistenti.</span><span class="sxs-lookup"><span data-stu-id="90f05-229">The `Update-Database` command tells the framework to apply the schema changes to the database and to preserve existing data.</span></span>

<a name="ssox"></a>

<span data-ttu-id="90f05-230">Se si eliminano tutti i record nel database, l'inizializzatore eseguirà il seeding del database e includerà il `Rating` campo.</span><span class="sxs-lookup"><span data-stu-id="90f05-230">If you delete all the records in the database, the initializer will seed the database and include the `Rating` field.</span></span> <span data-ttu-id="90f05-231">È possibile eseguire questa operazione con i collegamenti di eliminazione nel browser o da [Sql Server Object Explorer](xref:tutorials/razor-pages/sql#ssox) (SSOX).</span><span class="sxs-lookup"><span data-stu-id="90f05-231">You can do this with the delete links in the browser or from [Sql Server Object Explorer](xref:tutorials/razor-pages/sql#ssox) (SSOX).</span></span>

<span data-ttu-id="90f05-232">Un'altra opzione è quella di eliminare il database e usare le migrazioni per ricreare il database.</span><span class="sxs-lookup"><span data-stu-id="90f05-232">Another option is to delete the database and use migrations to re-create the database.</span></span> <span data-ttu-id="90f05-233">Per eliminare il database da SSOX:</span><span class="sxs-lookup"><span data-stu-id="90f05-233">To delete the database in SSOX:</span></span>

* <span data-ttu-id="90f05-234">Selezionare il database in SSOX.</span><span class="sxs-lookup"><span data-stu-id="90f05-234">Select the database in SSOX.</span></span>
* <span data-ttu-id="90f05-235">Fare clic con il pulsante destro del mouse sul database e scegliere **Elimina**.</span><span class="sxs-lookup"><span data-stu-id="90f05-235">Right-click on the database, and select **Delete**.</span></span>
* <span data-ttu-id="90f05-236">Selezionare **Chiudi connessioni esistenti**.</span><span class="sxs-lookup"><span data-stu-id="90f05-236">Check **Close existing connections**.</span></span>
* <span data-ttu-id="90f05-237">Selezionare **OK**.</span><span class="sxs-lookup"><span data-stu-id="90f05-237">Select **OK**.</span></span>
* <span data-ttu-id="90f05-238">In [PMC](xref:tutorials/razor-pages/new-field#pmc)aggiornare il database:</span><span class="sxs-lookup"><span data-stu-id="90f05-238">In the [PMC](xref:tutorials/razor-pages/new-field#pmc), update the database:</span></span>

  ```powershell
  Update-Database
  ```

# <a name="visual-studio-code--visual-studio-for-mac"></a>[<span data-ttu-id="90f05-239">Visual Studio Code / Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="90f05-239">Visual Studio Code / Visual Studio for Mac</span></span>](#tab/visual-studio-code+visual-studio-mac)

### <a name="drop-and-re-create-the-database"></a><span data-ttu-id="90f05-240">Eliminare e ricreare il database</span><span class="sxs-lookup"><span data-stu-id="90f05-240">Drop and re-create the database</span></span>

> [!NOTE]
> <span data-ttu-id="90f05-241">Per questa esercitazione, usare la funzionalità *migrazioni* di Entity Framework Core laddove possibile.</span><span class="sxs-lookup"><span data-stu-id="90f05-241">For this tutorial you, use the Entity Framework Core *migrations* feature where possible.</span></span> <span data-ttu-id="90f05-242">Le migrazioni aggiornano lo schema del database in base alle modifiche nel modello di dati.</span><span class="sxs-lookup"><span data-stu-id="90f05-242">Migrations updates the database schema to match changes in the data model.</span></span> <span data-ttu-id="90f05-243">Tuttavia, le migrazioni possono eseguire solo i tipi di modifiche supportate dal provider EF Core e le funzionalità del provider SQLite sono limitate.</span><span class="sxs-lookup"><span data-stu-id="90f05-243">However, migrations can only do the kinds of changes that the EF Core provider supports, and the SQLite provider's capabilities are limited.</span></span> <span data-ttu-id="90f05-244">Ad esempio l'aggiunta di una colonna è supportata, ma la rimozione o la modifica di una colonna non è supportata.</span><span class="sxs-lookup"><span data-stu-id="90f05-244">For example, adding a column is supported, but removing or changing a column is not supported.</span></span> <span data-ttu-id="90f05-245">Se si crea una migrazione per rimuovere o modificare una colonna, il comando `ef migrations add` ha esito positivo, ma il comando `ef database update` ha esito negativo.</span><span class="sxs-lookup"><span data-stu-id="90f05-245">If a migration is created to remove or change a column, the `ef migrations add` command succeeds but the `ef database update` command fails.</span></span> <span data-ttu-id="90f05-246">A causa di queste limitazioni, in questa esercitazione non vengono usate le migrazioni per le modifiche dello schema di SQLite.</span><span class="sxs-lookup"><span data-stu-id="90f05-246">Due to these limitations, this tutorial doesn't use migrations for SQLite schema changes.</span></span> <span data-ttu-id="90f05-247">In caso di modifiche dello schema, il database viene invece eliminato e ricreato.</span><span class="sxs-lookup"><span data-stu-id="90f05-247">Instead, when the schema changes, you drop and re-create the database.</span></span>
>
><span data-ttu-id="90f05-248">La soluzione per ovviare alle limitazioni di SQLite consiste nello scrivere manualmente il codice delle migrazioni per eseguire una ricompilazione della tabella in caso di modifiche nella tabella.</span><span class="sxs-lookup"><span data-stu-id="90f05-248">The workaround for the SQLite limitations is to manually write migrations code to perform a table rebuild when something in the table changes.</span></span> <span data-ttu-id="90f05-249">La ricompilazione della tabella comporta:</span><span class="sxs-lookup"><span data-stu-id="90f05-249">A table rebuild involves:</span></span>
>
>* <span data-ttu-id="90f05-250">Creazione di una nuova tabella.</span><span class="sxs-lookup"><span data-stu-id="90f05-250">Creating a new table.</span></span>
>* <span data-ttu-id="90f05-251">Copia dei dati dalla vecchia tabella alla nuova tabella.</span><span class="sxs-lookup"><span data-stu-id="90f05-251">Copying data from the old table to the new table.</span></span>
>* <span data-ttu-id="90f05-252">Eliminazione della tabella precedente.</span><span class="sxs-lookup"><span data-stu-id="90f05-252">Dropping the old table.</span></span>
>* <span data-ttu-id="90f05-253">Ridenominazione della nuova tabella.</span><span class="sxs-lookup"><span data-stu-id="90f05-253">Renaming the new table.</span></span>
>
><span data-ttu-id="90f05-254">Per altre informazioni, vedere le risorse seguenti:</span><span class="sxs-lookup"><span data-stu-id="90f05-254">For more information, see the following resources:</span></span>
>
> * [<span data-ttu-id="90f05-255">Limitazioni del provider di database SQLite per EF Core</span><span class="sxs-lookup"><span data-stu-id="90f05-255">SQLite EF Core Database Provider Limitations</span></span>](/ef/core/providers/sqlite/limitations)
> * [<span data-ttu-id="90f05-256">Personalizzare il codice di migrazione</span><span class="sxs-lookup"><span data-stu-id="90f05-256">Customize migration code</span></span>](/ef/core/managing-schemas/migrations/#customize-migration-code)
> * [<span data-ttu-id="90f05-257">Seeding dei dati</span><span class="sxs-lookup"><span data-stu-id="90f05-257">Data seeding</span></span>](/ef/core/modeling/data-seeding)
> * [<span data-ttu-id="90f05-258">Istruzione ALTER TABLE di SQLite</span><span class="sxs-lookup"><span data-stu-id="90f05-258">SQLite ALTER TABLE statement</span></span>](https://sqlite.org/lang_altertable.html)

1. <span data-ttu-id="90f05-259">Eliminare la cartella della migrazione.</span><span class="sxs-lookup"><span data-stu-id="90f05-259">Delete the migration folder.</span></span>  

1. <span data-ttu-id="90f05-260">Usare i comandi seguenti per ricreare il database.</span><span class="sxs-lookup"><span data-stu-id="90f05-260">Use the following commands to recreate the database.</span></span>

   ```dotnetcli
   dotnet ef database drop
   dotnet ef migrations add InitialCreate
   dotnet ef database update
   ```

---

<span data-ttu-id="90f05-261">Eseguire l'app e verificare che sia possibile creare/modificare/visualizzare i film con un campo `Rating`.</span><span class="sxs-lookup"><span data-stu-id="90f05-261">Run the app and verify you can create/edit/display movies with a `Rating` field.</span></span> <span data-ttu-id="90f05-262">Se il database non è inizializzato, impostare un punto di interruzione nel metodo `SeedData.Initialize`.</span><span class="sxs-lookup"><span data-stu-id="90f05-262">If the database isn't seeded, set a break point in the `SeedData.Initialize` method.</span></span>

## <a name="additional-resources"></a><span data-ttu-id="90f05-263">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="90f05-263">Additional resources</span></span>

> [!div class="step-by-step"]
> <span data-ttu-id="90f05-264">[Precedente: Aggiungi ricerca](xref:tutorials/razor-pages/search) 
>  Passaggio [successivo: aggiungere la convalida](xref:tutorials/razor-pages/validation)</span><span class="sxs-lookup"><span data-stu-id="90f05-264">[Previous: Add Search](xref:tutorials/razor-pages/search)
[Next: Add Validation](xref:tutorials/razor-pages/validation)</span></span>

::: moniker-end

::: moniker range="< aspnetcore-3.0"

<span data-ttu-id="90f05-265">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/razor-pages/razor-pages-start) ([procedura per il download](xref:index#how-to-download-a-sample)).</span><span class="sxs-lookup"><span data-stu-id="90f05-265">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/razor-pages/razor-pages-start) ([how to download](xref:index#how-to-download-a-sample)).</span></span>

<span data-ttu-id="90f05-266">In questa sezione vengono usate le Migrazioni Code First di [Entity Framework](/ef/core/get-started/aspnetcore/new-db) per:</span><span class="sxs-lookup"><span data-stu-id="90f05-266">In this section [Entity Framework](/ef/core/get-started/aspnetcore/new-db) Code First Migrations is used to:</span></span>

* <span data-ttu-id="90f05-267">Aggiungere un nuovo campo al modello.</span><span class="sxs-lookup"><span data-stu-id="90f05-267">Add a new field to the model.</span></span>
* <span data-ttu-id="90f05-268">Eseguire la migrazione nel database della modifica al nuovo schema del campo.</span><span class="sxs-lookup"><span data-stu-id="90f05-268">Migrate the new field schema change to the database.</span></span>

<span data-ttu-id="90f05-269">Quando si usa Code First di Entity Framework per creare automaticamente un database, Code First:</span><span class="sxs-lookup"><span data-stu-id="90f05-269">When using EF Code First to automatically create a database, Code First:</span></span>

* <span data-ttu-id="90f05-270">Aggiunge una [`__EFMigrationsHistory`](/ef/core/managing-schemas/migrations/history-table) tabella al database per rilevare se lo schema del database è sincronizzato con le classi del modello da cui è stato generato.</span><span class="sxs-lookup"><span data-stu-id="90f05-270">Adds an [`__EFMigrationsHistory`](/ef/core/managing-schemas/migrations/history-table) table to the database to track whether the schema of the database is in sync with the model classes it was generated from.</span></span>
* <span data-ttu-id="90f05-271">Se le classi del modello non sono sincronizzate con il database, EF genera un'eccezione.</span><span class="sxs-lookup"><span data-stu-id="90f05-271">If the model classes aren't in sync with the database, EF throws an exception.</span></span>

<span data-ttu-id="90f05-272">Con la verifica automatica che lo schema e il modello sono sincronizzati, è più semplice individuare problemi di codice di database incoerenti.</span><span class="sxs-lookup"><span data-stu-id="90f05-272">Automatic verification that the schema and model are in sync makes it easier to find inconsistent database code issues.</span></span>

## <a name="adding-a-rating-property-to-the-movie-model"></a><span data-ttu-id="90f05-273">Aggiunta di una proprietà Rating al modello Movie</span><span class="sxs-lookup"><span data-stu-id="90f05-273">Adding a Rating Property to the Movie Model</span></span>

<span data-ttu-id="90f05-274">Aprire il file *Models/Movie.cs* e aggiungere una proprietà `Rating`:</span><span class="sxs-lookup"><span data-stu-id="90f05-274">Open the *Models/Movie.cs* file and add a `Rating` property:</span></span>

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie22/Models/MovieDateRating.cs?highlight=13&name=snippet)]

<span data-ttu-id="90f05-275">Compilare l'app.</span><span class="sxs-lookup"><span data-stu-id="90f05-275">Build the app.</span></span>

<span data-ttu-id="90f05-276">Modificare *pages/Movies/ Index . cshtml* e aggiungere un `Rating` campo:</span><span class="sxs-lookup"><span data-stu-id="90f05-276">Edit *Pages/Movies/Index.cshtml*, and add a `Rating` field:</span></span>

[!code-cshtml[](razor-pages-start/sample/RazorPagesMovie22/Pages/Movies/IndexRating.cshtml?highlight=40-42,61-63)]

<span data-ttu-id="90f05-277">Aggiornare le pagine seguenti:</span><span class="sxs-lookup"><span data-stu-id="90f05-277">Update the following pages:</span></span>

* <span data-ttu-id="90f05-278">Aggiungere il campo `Rating` alle pagine Delete (Elimina) e Details (Dettagli).</span><span class="sxs-lookup"><span data-stu-id="90f05-278">Add the `Rating` field to the Delete and Details pages.</span></span>
* <span data-ttu-id="90f05-279">Aggiornare [Create.cshtml](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie22/Pages/Movies/Create.cshtml) con un campo `Rating`.</span><span class="sxs-lookup"><span data-stu-id="90f05-279">Update [Create.cshtml](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie22/Pages/Movies/Create.cshtml) with a `Rating` field.</span></span>
* <span data-ttu-id="90f05-280">Aggiungere il campo `Rating` alla pagina Edit (Modifica).</span><span class="sxs-lookup"><span data-stu-id="90f05-280">Add the `Rating` field to the Edit Page.</span></span>

<span data-ttu-id="90f05-281">L'app non funzionerà finché il database non verrà aggiornato in modo da includere il nuovo campo.</span><span class="sxs-lookup"><span data-stu-id="90f05-281">The app won't work until the database is updated to include the new field.</span></span> <span data-ttu-id="90f05-282">Se l'app viene eseguita adesso, l'app genera un'eccezione `SqlException` :</span><span class="sxs-lookup"><span data-stu-id="90f05-282">If the app is run now, the app throws a `SqlException`:</span></span>

`SqlException: Invalid column name 'Rating'.`

<span data-ttu-id="90f05-283">Questo errore viene visualizzato perché la classe del modello Movie aggiornata è diversa rispetto allo schema della tabella Movie nel database.</span><span class="sxs-lookup"><span data-stu-id="90f05-283">This error is caused by the updated Movie model class being different than the schema of the Movie table of the database.</span></span> <span data-ttu-id="90f05-284">Non è presente alcuna `Rating` colonna nella tabella di database.</span><span class="sxs-lookup"><span data-stu-id="90f05-284">There's no `Rating` column in the database table.</span></span>

<span data-ttu-id="90f05-285">Per correggere questo errore, esistono alcuni approcci:</span><span class="sxs-lookup"><span data-stu-id="90f05-285">There are a few approaches to resolving the error:</span></span>

1. <span data-ttu-id="90f05-286">Fare in modo che Entity Framework elimini e crei di nuovo automaticamente il database usando il nuovo schema di classi del modello.</span><span class="sxs-lookup"><span data-stu-id="90f05-286">Have the Entity Framework automatically drop and re-create the database using the new model class schema.</span></span> <span data-ttu-id="90f05-287">Questo approccio è utile nelle prime fasi del ciclo di sviluppo e consente di sviluppare rapidamente lo schema del modello e del database insieme.</span><span class="sxs-lookup"><span data-stu-id="90f05-287">This approach is convenient early in the development cycle, it allows you to quickly evolve the model and database schema together.</span></span> <span data-ttu-id="90f05-288">Lo svantaggio è che si perdono i dati esistenti nel database.</span><span class="sxs-lookup"><span data-stu-id="90f05-288">The downside is that you lose existing data in the database.</span></span> <span data-ttu-id="90f05-289">Non usare questo approccio in un database di produzione.</span><span class="sxs-lookup"><span data-stu-id="90f05-289">Don't use this approach on a production database!</span></span> <span data-ttu-id="90f05-290">L'eliminazione del database in caso di modifiche dello schema e l'utilizzo di un inizializzatore per eseguire automaticamente il seeding del database con dati di test è spesso un modo produttivo per sviluppare un'app.</span><span class="sxs-lookup"><span data-stu-id="90f05-290">Dropping the database on schema changes and using an initializer to automatically seed the database with test data is often a productive way to develop an app.</span></span>

2. <span data-ttu-id="90f05-291">Modificare esplicitamente lo schema del database esistente in modo che corrisponda alle classi del modello.</span><span class="sxs-lookup"><span data-stu-id="90f05-291">Explicitly modify the schema of the existing database so that it matches the model classes.</span></span> <span data-ttu-id="90f05-292">Il vantaggio di questo approccio consiste nel preservare i dati.</span><span class="sxs-lookup"><span data-stu-id="90f05-292">The advantage of this approach is to keep the data.</span></span> <span data-ttu-id="90f05-293">Apportare questa modifica manualmente o creando uno script di modifica del database.</span><span class="sxs-lookup"><span data-stu-id="90f05-293">Make this change either manually or by creating a database change script.</span></span>

3. <span data-ttu-id="90f05-294">Usare Migrazioni Code First per aggiornare lo schema del database.</span><span class="sxs-lookup"><span data-stu-id="90f05-294">Use Code First Migrations to update the database schema.</span></span>

<span data-ttu-id="90f05-295">Per questa esercitazione usare Migrazioni Code First.</span><span class="sxs-lookup"><span data-stu-id="90f05-295">For this tutorial, use Code First Migrations.</span></span>

<span data-ttu-id="90f05-296">Aggiornare la classe `SeedData` in modo che fornisca un valore per la nuova colonna.</span><span class="sxs-lookup"><span data-stu-id="90f05-296">Update the `SeedData` class so that it provides a value for the new column.</span></span> <span data-ttu-id="90f05-297">Di seguito viene illustrata una modifica di esempio, ma questa modifica viene apportata per ogni `new Movie` blocco.</span><span class="sxs-lookup"><span data-stu-id="90f05-297">A sample change is shown below, but make this change for each `new Movie` block.</span></span>

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie22/Models/SeedDataRating.cs?name=snippet1&highlight=8)]

<span data-ttu-id="90f05-298">Vedere il [file SeedData.cs completato](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie22/Models/SeedDataRating.cs).</span><span class="sxs-lookup"><span data-stu-id="90f05-298">See the [completed SeedData.cs file](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie22/Models/SeedDataRating.cs).</span></span>

<span data-ttu-id="90f05-299">Compilare la soluzione.</span><span class="sxs-lookup"><span data-stu-id="90f05-299">Build the solution.</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="90f05-300">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="90f05-300">Visual Studio</span></span>](#tab/visual-studio)

<a name="pmc"></a>

### <a name="add-a-migration-for-the-rating-field"></a><span data-ttu-id="90f05-301">Aggiungere una migrazione per il campo Rating</span><span class="sxs-lookup"><span data-stu-id="90f05-301">Add a migration for the rating field</span></span>

<span data-ttu-id="90f05-302">Dal menu **Strumenti** selezionare **Gestione pacchetti NuGet > Console di Gestione pacchetti**.</span><span class="sxs-lookup"><span data-stu-id="90f05-302">From the **Tools** menu, select **NuGet Package Manager > Package Manager Console**.</span></span>
<span data-ttu-id="90f05-303">Nella Console di Gestione pacchetti immettere i comandi seguenti:</span><span class="sxs-lookup"><span data-stu-id="90f05-303">In the PMC, enter the following commands:</span></span>

```powershell
Add-Migration Rating
Update-Database
```

<span data-ttu-id="90f05-304">Il comando `Add-Migration` indica al framework di:</span><span class="sxs-lookup"><span data-stu-id="90f05-304">The `Add-Migration` command tells the framework to:</span></span>

* <span data-ttu-id="90f05-305">Confrontare il `Movie` modello con lo `Movie` schema del database.</span><span class="sxs-lookup"><span data-stu-id="90f05-305">Compare the `Movie` model with the `Movie` database schema.</span></span>
* <span data-ttu-id="90f05-306">Creare il codice per eseguire la migrazione dello schema del database al nuovo modello.</span><span class="sxs-lookup"><span data-stu-id="90f05-306">Create code to migrate the database schema to the new model.</span></span>

<span data-ttu-id="90f05-307">Il nome "Rating" è arbitrario e viene usato per denominare il file di migrazione.</span><span class="sxs-lookup"><span data-stu-id="90f05-307">The name "Rating" is arbitrary and is used to name the migration file.</span></span> <span data-ttu-id="90f05-308">È consigliabile usare un nome significativo per il file di migrazione.</span><span class="sxs-lookup"><span data-stu-id="90f05-308">It's helpful to use a meaningful name for the migration file.</span></span>

<span data-ttu-id="90f05-309">Il comando `Update-Database` indica al framework di applicare le modifiche dello schema al database.</span><span class="sxs-lookup"><span data-stu-id="90f05-309">The `Update-Database` command tells the framework to apply the schema changes to the database.</span></span>

<a name="ssox"></a>

<span data-ttu-id="90f05-310">Se si eliminano tutti i record nel database, l'inizializzatore eseguirà il seeding del database e includerà il `Rating` campo.</span><span class="sxs-lookup"><span data-stu-id="90f05-310">If you delete all the records in the database, the initializer will seed the database and include the `Rating` field.</span></span> <span data-ttu-id="90f05-311">È possibile eseguire questa operazione con i collegamenti di eliminazione nel browser o da [Sql Server Object Explorer](xref:tutorials/razor-pages/sql#ssox) (SSOX).</span><span class="sxs-lookup"><span data-stu-id="90f05-311">You can do this with the delete links in the browser or from [Sql Server Object Explorer](xref:tutorials/razor-pages/sql#ssox) (SSOX).</span></span>

<span data-ttu-id="90f05-312">Un'altra opzione è quella di eliminare il database e usare le migrazioni per ricreare il database.</span><span class="sxs-lookup"><span data-stu-id="90f05-312">Another option is to delete the database and use migrations to re-create the database.</span></span> <span data-ttu-id="90f05-313">Per eliminare il database da SSOX:</span><span class="sxs-lookup"><span data-stu-id="90f05-313">To delete the database in SSOX:</span></span>

* <span data-ttu-id="90f05-314">Selezionare il database in SSOX.</span><span class="sxs-lookup"><span data-stu-id="90f05-314">Select the database in SSOX.</span></span>
* <span data-ttu-id="90f05-315">Fare clic con il pulsante destro del mouse sul database e scegliere **Elimina**.</span><span class="sxs-lookup"><span data-stu-id="90f05-315">Right-click on the database, and select **Delete**.</span></span>
* <span data-ttu-id="90f05-316">Selezionare **Chiudi connessioni esistenti**.</span><span class="sxs-lookup"><span data-stu-id="90f05-316">Check **Close existing connections**.</span></span>
* <span data-ttu-id="90f05-317">Selezionare **OK**.</span><span class="sxs-lookup"><span data-stu-id="90f05-317">Select **OK**.</span></span>
* <span data-ttu-id="90f05-318">In [PMC](xref:tutorials/razor-pages/new-field#pmc)aggiornare il database:</span><span class="sxs-lookup"><span data-stu-id="90f05-318">In the [PMC](xref:tutorials/razor-pages/new-field#pmc), update the database:</span></span>

  ```powershell
  Update-Database
  ```

# <a name="visual-studio-code--visual-studio-for-mac"></a>[<span data-ttu-id="90f05-319">Visual Studio Code / Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="90f05-319">Visual Studio Code / Visual Studio for Mac</span></span>](#tab/visual-studio-code+visual-studio-mac)

### <a name="drop-and-re-create-the-database"></a><span data-ttu-id="90f05-320">Eliminare e ricreare il database</span><span class="sxs-lookup"><span data-stu-id="90f05-320">Drop and re-create the database</span></span>

> [!NOTE]
> <span data-ttu-id="90f05-321">Per questa esercitazione, usare la funzionalità *migrazioni* di Entity Framework Core laddove possibile.</span><span class="sxs-lookup"><span data-stu-id="90f05-321">For this tutorial you, use the Entity Framework Core *migrations* feature where possible.</span></span> <span data-ttu-id="90f05-322">Le migrazioni aggiornano lo schema del database in base alle modifiche nel modello di dati.</span><span class="sxs-lookup"><span data-stu-id="90f05-322">Migrations updates the database schema to match changes in the data model.</span></span> <span data-ttu-id="90f05-323">Tuttavia, le migrazioni possono eseguire solo i tipi di modifiche supportate dal provider EF Core e le funzionalità del provider SQLite sono limitate.</span><span class="sxs-lookup"><span data-stu-id="90f05-323">However, migrations can only do the kinds of changes that the EF Core provider supports, and the SQLite provider's capabilities are limited.</span></span> <span data-ttu-id="90f05-324">Ad esempio l'aggiunta di una colonna è supportata, ma la rimozione o la modifica di una colonna non è supportata.</span><span class="sxs-lookup"><span data-stu-id="90f05-324">For example, adding a column is supported, but removing or changing a column is not supported.</span></span> <span data-ttu-id="90f05-325">Se si crea una migrazione per rimuovere o modificare una colonna, il comando `ef migrations add` ha esito positivo, ma il comando `ef database update` ha esito negativo.</span><span class="sxs-lookup"><span data-stu-id="90f05-325">If a migration is created to remove or change a column, the `ef migrations add` command succeeds but the `ef database update` command fails.</span></span> <span data-ttu-id="90f05-326">A causa di queste limitazioni, in questa esercitazione non vengono usate le migrazioni per le modifiche dello schema di SQLite.</span><span class="sxs-lookup"><span data-stu-id="90f05-326">Due to these limitations, this tutorial doesn't use migrations for SQLite schema changes.</span></span> <span data-ttu-id="90f05-327">In caso di modifiche dello schema, il database viene invece eliminato e ricreato.</span><span class="sxs-lookup"><span data-stu-id="90f05-327">Instead, when the schema changes, you drop and re-create the database.</span></span>
>
><span data-ttu-id="90f05-328">La soluzione per ovviare alle limitazioni di SQLite consiste nello scrivere manualmente il codice delle migrazioni per eseguire una ricompilazione della tabella in caso di modifiche nella tabella.</span><span class="sxs-lookup"><span data-stu-id="90f05-328">The workaround for the SQLite limitations is to manually write migrations code to perform a table rebuild when something in the table changes.</span></span> <span data-ttu-id="90f05-329">La ricompilazione della tabella comporta:</span><span class="sxs-lookup"><span data-stu-id="90f05-329">A table rebuild involves:</span></span>
>
>* <span data-ttu-id="90f05-330">Creazione di una nuova tabella.</span><span class="sxs-lookup"><span data-stu-id="90f05-330">Creating a new table.</span></span>
>* <span data-ttu-id="90f05-331">Copia dei dati dalla vecchia tabella alla nuova tabella.</span><span class="sxs-lookup"><span data-stu-id="90f05-331">Copying data from the old table to the new table.</span></span>
>* <span data-ttu-id="90f05-332">Eliminazione della tabella precedente.</span><span class="sxs-lookup"><span data-stu-id="90f05-332">Dropping the old table.</span></span>
>* <span data-ttu-id="90f05-333">Ridenominazione della nuova tabella.</span><span class="sxs-lookup"><span data-stu-id="90f05-333">Renaming the new table.</span></span>
>
><span data-ttu-id="90f05-334">Per altre informazioni, vedere le risorse seguenti:</span><span class="sxs-lookup"><span data-stu-id="90f05-334">For more information, see the following resources:</span></span>
>
> * [<span data-ttu-id="90f05-335">Limitazioni del provider di database SQLite per EF Core</span><span class="sxs-lookup"><span data-stu-id="90f05-335">SQLite EF Core Database Provider Limitations</span></span>](/ef/core/providers/sqlite/limitations)
> * [<span data-ttu-id="90f05-336">Personalizzare il codice di migrazione</span><span class="sxs-lookup"><span data-stu-id="90f05-336">Customize migration code</span></span>](/ef/core/managing-schemas/migrations/#customize-migration-code)
> * [<span data-ttu-id="90f05-337">Seeding dei dati</span><span class="sxs-lookup"><span data-stu-id="90f05-337">Data seeding</span></span>](/ef/core/modeling/data-seeding)
> * [<span data-ttu-id="90f05-338">Istruzione ALTER TABLE di SQLite</span><span class="sxs-lookup"><span data-stu-id="90f05-338">SQLite ALTER TABLE statement</span></span>](https://sqlite.org/lang_altertable.html)

<span data-ttu-id="90f05-339">Eliminare il database e usare le migrazioni per ricreare il database.</span><span class="sxs-lookup"><span data-stu-id="90f05-339">Delete the database and use migrations to re-create the database.</span></span> <span data-ttu-id="90f05-340">Per eliminare il database, eliminare il file di database (*MvcMovie.db*).</span><span class="sxs-lookup"><span data-stu-id="90f05-340">To delete the database, delete the database file (*MvcMovie.db*).</span></span> <span data-ttu-id="90f05-341">Eseguire quindi il comando `ef database update`:</span><span class="sxs-lookup"><span data-stu-id="90f05-341">Then run the `ef database update` command:</span></span>

```dotnetcli
dotnet ef database update
```

---

<span data-ttu-id="90f05-342">Eseguire l'app e verificare che sia possibile creare/modificare/visualizzare i film con un campo `Rating`.</span><span class="sxs-lookup"><span data-stu-id="90f05-342">Run the app and verify you can create/edit/display movies with a `Rating` field.</span></span> <span data-ttu-id="90f05-343">Se il database non è inizializzato, impostare un punto di interruzione nel metodo `SeedData.Initialize`.</span><span class="sxs-lookup"><span data-stu-id="90f05-343">If the database isn't seeded, set a break point in the `SeedData.Initialize` method.</span></span>

## <a name="additional-resources"></a><span data-ttu-id="90f05-344">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="90f05-344">Additional resources</span></span>

* [<span data-ttu-id="90f05-345">Versione YouTube dell'esercitazione</span><span class="sxs-lookup"><span data-stu-id="90f05-345">YouTube version of this tutorial</span></span>](https://youtu.be/3i7uMxiGGR8)

> [!div class="step-by-step"]
> <span data-ttu-id="90f05-346">[Precedente: Aggiungi ricerca](xref:tutorials/razor-pages/search) 
>  Passaggio [successivo: aggiungere la convalida](xref:tutorials/razor-pages/validation)</span><span class="sxs-lookup"><span data-stu-id="90f05-346">[Previous: Add Search](xref:tutorials/razor-pages/search)
[Next: Add Validation](xref:tutorials/razor-pages/validation)</span></span>

::: moniker-end