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
ms.openlocfilehash: 852c26f89b229d89797d9ce6fce2983319defe9c
ms.sourcegitcommit: 422e8444b9f5cedc373be5efe8032822db54fcaf
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/18/2021
ms.locfileid: "101101146"
---
# <a name="part-7-add-a-new-field-to-a-razor-page-in-aspnet-core"></a><span data-ttu-id="9d396-103">Parte 7, aggiungere un nuovo campo a una Razor pagina in ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="9d396-103">Part 7, add a new field to a Razor Page in ASP.NET Core</span></span>

<span data-ttu-id="9d396-104">Autore: [Rick Anderson](https://twitter.com/RickAndMSFT)</span><span class="sxs-lookup"><span data-stu-id="9d396-104">By [Rick Anderson](https://twitter.com/RickAndMSFT)</span></span>

::: moniker range=">= aspnetcore-5.0"

<span data-ttu-id="9d396-105">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie50) ([procedura per il download](xref:index#how-to-download-a-sample)).</span><span class="sxs-lookup"><span data-stu-id="9d396-105">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie50) ([how to download](xref:index#how-to-download-a-sample)).</span></span>

<span data-ttu-id="9d396-106">In questa sezione vengono usate le Migrazioni Code First di [Entity Framework](/ef/core/get-started/aspnetcore/new-db) per:</span><span class="sxs-lookup"><span data-stu-id="9d396-106">In this section [Entity Framework](/ef/core/get-started/aspnetcore/new-db) Code First Migrations is used to:</span></span>

* <span data-ttu-id="9d396-107">Aggiungere un nuovo campo al modello.</span><span class="sxs-lookup"><span data-stu-id="9d396-107">Add a new field to the model.</span></span>
* <span data-ttu-id="9d396-108">Eseguire la migrazione nel database della modifica al nuovo schema del campo.</span><span class="sxs-lookup"><span data-stu-id="9d396-108">Migrate the new field schema change to the database.</span></span>

<span data-ttu-id="9d396-109">Quando si usa Code First di Entity Framework per creare automaticamente un database, Code First:</span><span class="sxs-lookup"><span data-stu-id="9d396-109">When using EF Code First to automatically create a database, Code First:</span></span>

* <span data-ttu-id="9d396-110">Aggiunge una [`__EFMigrationsHistory`](/ef/core/managing-schemas/migrations/history-table) tabella al database per rilevare se lo schema del database è sincronizzato con le classi del modello da cui è stato generato.</span><span class="sxs-lookup"><span data-stu-id="9d396-110">Adds an [`__EFMigrationsHistory`](/ef/core/managing-schemas/migrations/history-table) table to the database to track whether the schema of the database is in sync with the model classes it was generated from.</span></span>
* <span data-ttu-id="9d396-111">Se le classi del modello non sono sincronizzate con il database, EF genera un'eccezione.</span><span class="sxs-lookup"><span data-stu-id="9d396-111">If the model classes aren't in sync with the database, EF throws an exception.</span></span>

<span data-ttu-id="9d396-112">Con la verifica automatica che lo schema e il modello sono sincronizzati, è più semplice individuare problemi di codice di database incoerenti.</span><span class="sxs-lookup"><span data-stu-id="9d396-112">Automatic verification that the schema and model are in sync makes it easier to find inconsistent database code issues.</span></span>

## <a name="adding-a-rating-property-to-the-movie-model"></a><span data-ttu-id="9d396-113">Aggiunta di una proprietà Rating al modello Movie</span><span class="sxs-lookup"><span data-stu-id="9d396-113">Adding a Rating Property to the Movie Model</span></span>

1. <span data-ttu-id="9d396-114">Aprire il file *Models/Movie.cs* e aggiungere una proprietà `Rating`:</span><span class="sxs-lookup"><span data-stu-id="9d396-114">Open the *Models/Movie.cs* file and add a `Rating` property:</span></span>

   [!code-csharp[](razor-pages-start/sample/RazorPagesMovie50/Models/MovieDateRating.cs?highlight=13&name=snippet)]

1. <span data-ttu-id="9d396-115">Compilare l'app.</span><span class="sxs-lookup"><span data-stu-id="9d396-115">Build the app.</span></span>

1. <span data-ttu-id="9d396-116">Modificare *pages/Movies/ Index . cshtml* e aggiungere un `Rating` campo:</span><span class="sxs-lookup"><span data-stu-id="9d396-116">Edit *Pages/Movies/Index.cshtml*, and add a `Rating` field:</span></span>

   <a name="addrat"></a>

   [!code-cshtml[](razor-pages-start/sample/RazorPagesMovie50/SnapShots/IndexRating.cshtml?highlight=40-42,62-64)]

1. <span data-ttu-id="9d396-117">Aggiornare le pagine seguenti:</span><span class="sxs-lookup"><span data-stu-id="9d396-117">Update the following pages:</span></span>
   1. <span data-ttu-id="9d396-118">Aggiungere il campo `Rating` alle pagine Delete (Elimina) e Details (Dettagli).</span><span class="sxs-lookup"><span data-stu-id="9d396-118">Add the `Rating` field to the Delete and Details pages.</span></span>
   1. <span data-ttu-id="9d396-119">Aggiornare [Create.cshtml](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie50/Pages/Movies/Create.cshtml) con un campo `Rating`.</span><span class="sxs-lookup"><span data-stu-id="9d396-119">Update [Create.cshtml](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie50/Pages/Movies/Create.cshtml) with a `Rating` field.</span></span>
   1. <span data-ttu-id="9d396-120">Aggiungere il campo `Rating` alla pagina Edit (Modifica).</span><span class="sxs-lookup"><span data-stu-id="9d396-120">Add the `Rating` field to the Edit Page.</span></span>

<span data-ttu-id="9d396-121">L'app non funzionerà finché il database non verrà aggiornato in modo da includere il nuovo campo.</span><span class="sxs-lookup"><span data-stu-id="9d396-121">The app won't work until the database is updated to include the new field.</span></span> <span data-ttu-id="9d396-122">L'esecuzione dell'app senza un aggiornamento del database genera un'eccezione `SqlException` :</span><span class="sxs-lookup"><span data-stu-id="9d396-122">Running the app without an update to the database throws a `SqlException`:</span></span>

`SqlException: Invalid column name 'Rating'.`

<span data-ttu-id="9d396-123">L' `SqlException` eccezione è causata dal fatto che la classe del modello di film aggiornata è diversa dallo schema della tabella dei film del database.</span><span class="sxs-lookup"><span data-stu-id="9d396-123">The `SqlException` exception is caused by the updated Movie model class being different than the schema of the Movie table of the database.</span></span> <span data-ttu-id="9d396-124">Non è presente alcuna `Rating` colonna nella tabella di database.</span><span class="sxs-lookup"><span data-stu-id="9d396-124">There's no `Rating` column in the database table.</span></span>

<span data-ttu-id="9d396-125">Per correggere questo errore, esistono alcuni approcci:</span><span class="sxs-lookup"><span data-stu-id="9d396-125">There are a few approaches to resolving the error:</span></span>

1. <span data-ttu-id="9d396-126">Fare in modo che Entity Framework elimini e crei di nuovo automaticamente il database usando il nuovo schema di classi del modello.</span><span class="sxs-lookup"><span data-stu-id="9d396-126">Have the Entity Framework automatically drop and re-create the database using the new model class schema.</span></span> <span data-ttu-id="9d396-127">Questo approccio è utile nelle prime fasi del ciclo di sviluppo e consente di sviluppare rapidamente lo schema del modello e del database insieme.</span><span class="sxs-lookup"><span data-stu-id="9d396-127">This approach is convenient early in the development cycle, it allows you to quickly evolve the model and database schema together.</span></span> <span data-ttu-id="9d396-128">Lo svantaggio è che si perdono i dati esistenti nel database.</span><span class="sxs-lookup"><span data-stu-id="9d396-128">The downside is that you lose existing data in the database.</span></span> <span data-ttu-id="9d396-129">Non usare questo approccio in un database di produzione.</span><span class="sxs-lookup"><span data-stu-id="9d396-129">Don't use this approach on a production database!</span></span> <span data-ttu-id="9d396-130">L'eliminazione del database in caso di modifiche dello schema e l'utilizzo di un inizializzatore per eseguire automaticamente il seeding del database con dati di test è spesso un modo produttivo per sviluppare un'app.</span><span class="sxs-lookup"><span data-stu-id="9d396-130">Dropping the database on schema changes and using an initializer to automatically seed the database with test data is often a productive way to develop an app.</span></span>

2. <span data-ttu-id="9d396-131">Modificare esplicitamente lo schema del database esistente in modo che corrisponda alle classi del modello.</span><span class="sxs-lookup"><span data-stu-id="9d396-131">Explicitly modify the schema of the existing database so that it matches the model classes.</span></span> <span data-ttu-id="9d396-132">Il vantaggio di questo approccio consiste nel preservare i dati.</span><span class="sxs-lookup"><span data-stu-id="9d396-132">The advantage of this approach is to keep the data.</span></span> <span data-ttu-id="9d396-133">Apportare questa modifica manualmente o creando uno script di modifica del database.</span><span class="sxs-lookup"><span data-stu-id="9d396-133">Make this change either manually or by creating a database change script.</span></span>

3. <span data-ttu-id="9d396-134">Usare Migrazioni Code First per aggiornare lo schema del database.</span><span class="sxs-lookup"><span data-stu-id="9d396-134">Use Code First Migrations to update the database schema.</span></span>

<span data-ttu-id="9d396-135">Per questa esercitazione usare Migrazioni Code First.</span><span class="sxs-lookup"><span data-stu-id="9d396-135">For this tutorial, use Code First Migrations.</span></span>

<span data-ttu-id="9d396-136">Aggiornare la classe `SeedData` in modo che fornisca un valore per la nuova colonna.</span><span class="sxs-lookup"><span data-stu-id="9d396-136">Update the `SeedData` class so that it provides a value for the new column.</span></span> <span data-ttu-id="9d396-137">Di seguito viene illustrata una modifica di esempio, ma questa modifica viene apportata per ogni `new Movie` blocco.</span><span class="sxs-lookup"><span data-stu-id="9d396-137">A sample change is shown below, but make this change for each `new Movie` block.</span></span>

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie50/Models/SeedDataRating.cs?name=snippet1&highlight=8)]

<span data-ttu-id="9d396-138">Vedere il [file SeedData.cs completato](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie50/Models/SeedDataRating.cs).</span><span class="sxs-lookup"><span data-stu-id="9d396-138">See the [completed SeedData.cs file](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie50/Models/SeedDataRating.cs).</span></span>

<span data-ttu-id="9d396-139">Compilare la soluzione.</span><span class="sxs-lookup"><span data-stu-id="9d396-139">Build the solution.</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="9d396-140">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="9d396-140">Visual Studio</span></span>](#tab/visual-studio)

<a name="pmc"></a>

### <a name="add-a-migration-for-the-rating-field"></a><span data-ttu-id="9d396-141">Aggiungere una migrazione per il campo Rating</span><span class="sxs-lookup"><span data-stu-id="9d396-141">Add a migration for the rating field</span></span>

1. <span data-ttu-id="9d396-142">Dal menu **Strumenti** selezionare **Gestione pacchetti NuGet > Console di Gestione pacchetti**.</span><span class="sxs-lookup"><span data-stu-id="9d396-142">From the **Tools** menu, select **NuGet Package Manager > Package Manager Console**.</span></span>
2. <span data-ttu-id="9d396-143">Nella Console di Gestione pacchetti immettere i comandi seguenti:</span><span class="sxs-lookup"><span data-stu-id="9d396-143">In the PMC, enter the following commands:</span></span>

   ```powershell
   Add-Migration Rating
   Update-Database
   ```

<span data-ttu-id="9d396-144">Il comando `Add-Migration` indica al framework di:</span><span class="sxs-lookup"><span data-stu-id="9d396-144">The `Add-Migration` command tells the framework to:</span></span>

* <span data-ttu-id="9d396-145">Confrontare il `Movie` modello con lo `Movie` schema del database.</span><span class="sxs-lookup"><span data-stu-id="9d396-145">Compare the `Movie` model with the `Movie` database schema.</span></span>
* <span data-ttu-id="9d396-146">Creare il codice per eseguire la migrazione dello schema del database al nuovo modello.</span><span class="sxs-lookup"><span data-stu-id="9d396-146">Create code to migrate the database schema to the new model.</span></span>

<span data-ttu-id="9d396-147">Il nome "Rating" è arbitrario e viene usato per denominare il file di migrazione.</span><span class="sxs-lookup"><span data-stu-id="9d396-147">The name "Rating" is arbitrary and is used to name the migration file.</span></span> <span data-ttu-id="9d396-148">È consigliabile usare un nome significativo per il file di migrazione.</span><span class="sxs-lookup"><span data-stu-id="9d396-148">It's helpful to use a meaningful name for the migration file.</span></span>

<span data-ttu-id="9d396-149">Il `Update-Database` comando indica al Framework di applicare le modifiche dello schema al database e di mantenere i dati esistenti.</span><span class="sxs-lookup"><span data-stu-id="9d396-149">The `Update-Database` command tells the framework to apply the schema changes to the database and to preserve existing data.</span></span>

<a name="ssox"></a>

<span data-ttu-id="9d396-150">Se si eliminano tutti i record nel database, l'inizializzatore eseguirà il seeding del database e includerà il `Rating` campo.</span><span class="sxs-lookup"><span data-stu-id="9d396-150">If you delete all the records in the database, the initializer will seed the database and include the `Rating` field.</span></span> <span data-ttu-id="9d396-151">È possibile eseguire questa operazione con i collegamenti di eliminazione nel browser o da [Sql Server Object Explorer](xref:tutorials/razor-pages/sql#ssox) (SSOX).</span><span class="sxs-lookup"><span data-stu-id="9d396-151">You can do this with the delete links in the browser or from [Sql Server Object Explorer](xref:tutorials/razor-pages/sql#ssox) (SSOX).</span></span>

<span data-ttu-id="9d396-152">Un'altra opzione è quella di eliminare il database e usare le migrazioni per ricreare il database.</span><span class="sxs-lookup"><span data-stu-id="9d396-152">Another option is to delete the database and use migrations to re-create the database.</span></span> <span data-ttu-id="9d396-153">Per eliminare il database da SSOX:</span><span class="sxs-lookup"><span data-stu-id="9d396-153">To delete the database in SSOX:</span></span>

1. <span data-ttu-id="9d396-154">Selezionare il database in SSOX.</span><span class="sxs-lookup"><span data-stu-id="9d396-154">Select the database in SSOX.</span></span>
1. <span data-ttu-id="9d396-155">Fare clic con il pulsante destro del mouse sul database e scegliere **Elimina**.</span><span class="sxs-lookup"><span data-stu-id="9d396-155">Right-click on the database, and select **Delete**.</span></span>
1. <span data-ttu-id="9d396-156">Selezionare **Chiudi connessioni esistenti**.</span><span class="sxs-lookup"><span data-stu-id="9d396-156">Check **Close existing connections**.</span></span>
1. <span data-ttu-id="9d396-157">Selezionare **OK**.</span><span class="sxs-lookup"><span data-stu-id="9d396-157">Select **OK**.</span></span>
1. <span data-ttu-id="9d396-158">In [PMC](xref:tutorials/razor-pages/new-field#pmc)aggiornare il database:</span><span class="sxs-lookup"><span data-stu-id="9d396-158">In the [PMC](xref:tutorials/razor-pages/new-field#pmc), update the database:</span></span>

   ```powershell
   Update-Database
   ```

# <a name="visual-studio-code--visual-studio-for-mac"></a>[<span data-ttu-id="9d396-159">Visual Studio Code / Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="9d396-159">Visual Studio Code / Visual Studio for Mac</span></span>](#tab/visual-studio-code+visual-studio-mac)

### <a name="drop-and-re-create-the-database"></a><span data-ttu-id="9d396-160">Eliminare e ricreare il database</span><span class="sxs-lookup"><span data-stu-id="9d396-160">Drop and re-create the database</span></span>

> [!NOTE]
> <span data-ttu-id="9d396-161">Per questa esercitazione, è possibile usare la funzionalità *migrazioni* di Entity Framework Core, ove possibile.</span><span class="sxs-lookup"><span data-stu-id="9d396-161">For this tutorial, you use the Entity Framework Core *migrations* feature where possible.</span></span> <span data-ttu-id="9d396-162">Le migrazioni aggiornano lo schema del database in base alle modifiche nel modello di dati.</span><span class="sxs-lookup"><span data-stu-id="9d396-162">Migrations updates the database schema to match changes in the data model.</span></span> <span data-ttu-id="9d396-163">Tuttavia, le migrazioni possono eseguire solo i tipi di modifiche supportate dal provider EF Core e le funzionalità del provider SQLite sono limitate.</span><span class="sxs-lookup"><span data-stu-id="9d396-163">However, migrations can only do the kinds of changes that the EF Core provider supports, and the SQLite provider's capabilities are limited.</span></span> <span data-ttu-id="9d396-164">Ad esempio l'aggiunta di una colonna è supportata, ma la rimozione o la modifica di una colonna non è supportata.</span><span class="sxs-lookup"><span data-stu-id="9d396-164">For example, adding a column is supported, but removing or changing a column is not supported.</span></span> <span data-ttu-id="9d396-165">Se si crea una migrazione per rimuovere o modificare una colonna, il comando `ef migrations add` ha esito positivo, ma il comando `ef database update` ha esito negativo.</span><span class="sxs-lookup"><span data-stu-id="9d396-165">If a migration is created to remove or change a column, the `ef migrations add` command succeeds but the `ef database update` command fails.</span></span> <span data-ttu-id="9d396-166">A causa di queste limitazioni, in questa esercitazione non vengono usate le migrazioni per le modifiche dello schema di SQLite.</span><span class="sxs-lookup"><span data-stu-id="9d396-166">Due to these limitations, this tutorial doesn't use migrations for SQLite schema changes.</span></span> <span data-ttu-id="9d396-167">In caso di modifiche dello schema, il database viene invece eliminato e ricreato.</span><span class="sxs-lookup"><span data-stu-id="9d396-167">Instead, when the schema changes, you drop and re-create the database.</span></span>
>
><span data-ttu-id="9d396-168">La soluzione per ovviare alle limitazioni di SQLite consiste nello scrivere manualmente il codice delle migrazioni per eseguire una ricompilazione della tabella in caso di modifiche nella tabella.</span><span class="sxs-lookup"><span data-stu-id="9d396-168">The workaround for the SQLite limitations is to manually write migrations code to perform a table rebuild when something in the table changes.</span></span> <span data-ttu-id="9d396-169">La ricompilazione della tabella comporta:</span><span class="sxs-lookup"><span data-stu-id="9d396-169">A table rebuild involves:</span></span>
>
>* <span data-ttu-id="9d396-170">Creazione di una nuova tabella.</span><span class="sxs-lookup"><span data-stu-id="9d396-170">Creating a new table.</span></span>
>* <span data-ttu-id="9d396-171">Copia dei dati dalla vecchia tabella alla nuova tabella.</span><span class="sxs-lookup"><span data-stu-id="9d396-171">Copying data from the old table to the new table.</span></span>
>* <span data-ttu-id="9d396-172">Eliminazione della tabella precedente.</span><span class="sxs-lookup"><span data-stu-id="9d396-172">Dropping the old table.</span></span>
>* <span data-ttu-id="9d396-173">Ridenominazione della nuova tabella.</span><span class="sxs-lookup"><span data-stu-id="9d396-173">Renaming the new table.</span></span>
>
><span data-ttu-id="9d396-174">Per altre informazioni, vedere le risorse seguenti:</span><span class="sxs-lookup"><span data-stu-id="9d396-174">For more information, see the following resources:</span></span>
>
> * [<span data-ttu-id="9d396-175">Limitazioni del provider di database SQLite per EF Core</span><span class="sxs-lookup"><span data-stu-id="9d396-175">SQLite EF Core Database Provider Limitations</span></span>](/ef/core/providers/sqlite/limitations)
> * [<span data-ttu-id="9d396-176">Personalizzare il codice di migrazione</span><span class="sxs-lookup"><span data-stu-id="9d396-176">Customize migration code</span></span>](/ef/core/managing-schemas/migrations/#customize-migration-code)
> * [<span data-ttu-id="9d396-177">Seeding dei dati</span><span class="sxs-lookup"><span data-stu-id="9d396-177">Data seeding</span></span>](/ef/core/modeling/data-seeding)
> * [<span data-ttu-id="9d396-178">Istruzione ALTER TABLE di SQLite</span><span class="sxs-lookup"><span data-stu-id="9d396-178">SQLite ALTER TABLE statement</span></span>](https://sqlite.org/lang_altertable.html)

1. <span data-ttu-id="9d396-179">Eliminare la cartella della migrazione.</span><span class="sxs-lookup"><span data-stu-id="9d396-179">Delete the migration folder.</span></span>  

1. <span data-ttu-id="9d396-180">Usare i comandi seguenti per ricreare il database.</span><span class="sxs-lookup"><span data-stu-id="9d396-180">Use the following commands to recreate the database.</span></span>

   ```dotnetcli
   dotnet ef database drop
   dotnet ef migrations add InitialCreate
   dotnet ef database update
   ```

---

<span data-ttu-id="9d396-181">Eseguire l'app e verificare che sia possibile creare/modificare/visualizzare i film con un campo `Rating`.</span><span class="sxs-lookup"><span data-stu-id="9d396-181">Run the app and verify you can create/edit/display movies with a `Rating` field.</span></span> <span data-ttu-id="9d396-182">Se il database non è inizializzato, impostare un punto di interruzione nel metodo `SeedData.Initialize`.</span><span class="sxs-lookup"><span data-stu-id="9d396-182">If the database isn't seeded, set a break point in the `SeedData.Initialize` method.</span></span>

## <a name="additional-resources"></a><span data-ttu-id="9d396-183">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="9d396-183">Additional resources</span></span>

> [!div class="step-by-step"]
> <span data-ttu-id="9d396-184">[Precedente: Aggiungi ricerca](xref:tutorials/razor-pages/search) 
>  Passaggio [successivo: aggiungere la convalida](xref:tutorials/razor-pages/validation)</span><span class="sxs-lookup"><span data-stu-id="9d396-184">[Previous: Add Search](xref:tutorials/razor-pages/search)
[Next: Add Validation](xref:tutorials/razor-pages/validation)</span></span>

::: moniker-end

::: moniker range="< aspnetcore-5.0 >= aspnetcore-3.0"

<span data-ttu-id="9d396-185">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie30) ([procedura per il download](xref:index#how-to-download-a-sample)).</span><span class="sxs-lookup"><span data-stu-id="9d396-185">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie30) ([how to download](xref:index#how-to-download-a-sample)).</span></span>

<span data-ttu-id="9d396-186">In questa sezione vengono usate le Migrazioni Code First di [Entity Framework](/ef/core/get-started/aspnetcore/new-db) per:</span><span class="sxs-lookup"><span data-stu-id="9d396-186">In this section [Entity Framework](/ef/core/get-started/aspnetcore/new-db) Code First Migrations is used to:</span></span>

* <span data-ttu-id="9d396-187">Aggiungere un nuovo campo al modello.</span><span class="sxs-lookup"><span data-stu-id="9d396-187">Add a new field to the model.</span></span>
* <span data-ttu-id="9d396-188">Eseguire la migrazione nel database della modifica al nuovo schema del campo.</span><span class="sxs-lookup"><span data-stu-id="9d396-188">Migrate the new field schema change to the database.</span></span>

<span data-ttu-id="9d396-189">Quando si usa Code First di Entity Framework per creare automaticamente un database, Code First:</span><span class="sxs-lookup"><span data-stu-id="9d396-189">When using EF Code First to automatically create a database, Code First:</span></span>

* <span data-ttu-id="9d396-190">Aggiunge una [`__EFMigrationsHistory`](/ef/core/managing-schemas/migrations/history-table) tabella al database per rilevare se lo schema del database è sincronizzato con le classi del modello da cui è stato generato.</span><span class="sxs-lookup"><span data-stu-id="9d396-190">Adds an [`__EFMigrationsHistory`](/ef/core/managing-schemas/migrations/history-table) table to the database to track whether the schema of the database is in sync with the model classes it was generated from.</span></span>
* <span data-ttu-id="9d396-191">Se le classi del modello non sono sincronizzate con il database, EF genera un'eccezione.</span><span class="sxs-lookup"><span data-stu-id="9d396-191">If the model classes aren't in sync with the database, EF throws an exception.</span></span>

<span data-ttu-id="9d396-192">Con la verifica automatica che lo schema e il modello sono sincronizzati, è più semplice individuare problemi di codice di database incoerenti.</span><span class="sxs-lookup"><span data-stu-id="9d396-192">Automatic verification that the schema and model are in sync makes it easier to find inconsistent database code issues.</span></span>

## <a name="adding-a-rating-property-to-the-movie-model"></a><span data-ttu-id="9d396-193">Aggiunta di una proprietà Rating al modello Movie</span><span class="sxs-lookup"><span data-stu-id="9d396-193">Adding a Rating Property to the Movie Model</span></span>

1. <span data-ttu-id="9d396-194">Aprire il file *Models/Movie.cs* e aggiungere una proprietà `Rating`:</span><span class="sxs-lookup"><span data-stu-id="9d396-194">Open the *Models/Movie.cs* file and add a `Rating` property:</span></span>

   [!code-csharp[](razor-pages-start/sample/RazorPagesMovie30/Models/MovieDateRating.cs?highlight=13&name=snippet)]

1. <span data-ttu-id="9d396-195">Compilare l'app.</span><span class="sxs-lookup"><span data-stu-id="9d396-195">Build the app.</span></span>

1. <span data-ttu-id="9d396-196">Modificare *pages/Movies/ Index . cshtml* e aggiungere un `Rating` campo:</span><span class="sxs-lookup"><span data-stu-id="9d396-196">Edit *Pages/Movies/Index.cshtml*, and add a `Rating` field:</span></span>

   <a name="addrat"></a>

   [!code-cshtml[](razor-pages-start/sample/RazorPagesMovie30/SnapShots/IndexRating.cshtml?highlight=40-42,62-64)]

1. <span data-ttu-id="9d396-197">Aggiornare le pagine seguenti:</span><span class="sxs-lookup"><span data-stu-id="9d396-197">Update the following pages:</span></span>
   1. <span data-ttu-id="9d396-198">Aggiungere il campo `Rating` alle pagine Delete (Elimina) e Details (Dettagli).</span><span class="sxs-lookup"><span data-stu-id="9d396-198">Add the `Rating` field to the Delete and Details pages.</span></span>
   1. <span data-ttu-id="9d396-199">Aggiornare [Create.cshtml](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie30/Pages/Movies/Create.cshtml) con un campo `Rating`.</span><span class="sxs-lookup"><span data-stu-id="9d396-199">Update [Create.cshtml](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie30/Pages/Movies/Create.cshtml) with a `Rating` field.</span></span>
   1. <span data-ttu-id="9d396-200">Aggiungere il campo `Rating` alla pagina Edit (Modifica).</span><span class="sxs-lookup"><span data-stu-id="9d396-200">Add the `Rating` field to the Edit Page.</span></span>

<span data-ttu-id="9d396-201">L'app non funzionerà finché il database non verrà aggiornato in modo da includere il nuovo campo.</span><span class="sxs-lookup"><span data-stu-id="9d396-201">The app won't work until the database is updated to include the new field.</span></span> <span data-ttu-id="9d396-202">L'esecuzione dell'app senza un aggiornamento del database genera un'eccezione `SqlException` :</span><span class="sxs-lookup"><span data-stu-id="9d396-202">Running the app without an update to the database throws a `SqlException`:</span></span>

`SqlException: Invalid column name 'Rating'.`

<span data-ttu-id="9d396-203">L' `SqlException` eccezione è causata dal fatto che la classe del modello di film aggiornata è diversa dallo schema della tabella dei film del database.</span><span class="sxs-lookup"><span data-stu-id="9d396-203">The `SqlException` exception is caused by the updated Movie model class being different than the schema of the Movie table of the database.</span></span> <span data-ttu-id="9d396-204">Non è presente alcuna `Rating` colonna nella tabella di database.</span><span class="sxs-lookup"><span data-stu-id="9d396-204">There's no `Rating` column in the database table.</span></span>

<span data-ttu-id="9d396-205">Per correggere questo errore, esistono alcuni approcci:</span><span class="sxs-lookup"><span data-stu-id="9d396-205">There are a few approaches to resolving the error:</span></span>

1. <span data-ttu-id="9d396-206">Fare in modo che Entity Framework elimini e crei di nuovo automaticamente il database usando il nuovo schema di classi del modello.</span><span class="sxs-lookup"><span data-stu-id="9d396-206">Have the Entity Framework automatically drop and re-create the database using the new model class schema.</span></span> <span data-ttu-id="9d396-207">Questo approccio è utile nelle prime fasi del ciclo di sviluppo e consente di sviluppare rapidamente lo schema del modello e del database insieme.</span><span class="sxs-lookup"><span data-stu-id="9d396-207">This approach is convenient early in the development cycle, it allows you to quickly evolve the model and database schema together.</span></span> <span data-ttu-id="9d396-208">Lo svantaggio è che si perdono i dati esistenti nel database.</span><span class="sxs-lookup"><span data-stu-id="9d396-208">The downside is that you lose existing data in the database.</span></span> <span data-ttu-id="9d396-209">Non usare questo approccio in un database di produzione.</span><span class="sxs-lookup"><span data-stu-id="9d396-209">Don't use this approach on a production database!</span></span> <span data-ttu-id="9d396-210">L'eliminazione del database in caso di modifiche dello schema e l'utilizzo di un inizializzatore per eseguire automaticamente il seeding del database con dati di test è spesso un modo produttivo per sviluppare un'app.</span><span class="sxs-lookup"><span data-stu-id="9d396-210">Dropping the database on schema changes and using an initializer to automatically seed the database with test data is often a productive way to develop an app.</span></span>

2. <span data-ttu-id="9d396-211">Modificare esplicitamente lo schema del database esistente in modo che corrisponda alle classi del modello.</span><span class="sxs-lookup"><span data-stu-id="9d396-211">Explicitly modify the schema of the existing database so that it matches the model classes.</span></span> <span data-ttu-id="9d396-212">Il vantaggio di questo approccio consiste nel preservare i dati.</span><span class="sxs-lookup"><span data-stu-id="9d396-212">The advantage of this approach is to keep the data.</span></span> <span data-ttu-id="9d396-213">Apportare questa modifica manualmente o creando uno script di modifica del database.</span><span class="sxs-lookup"><span data-stu-id="9d396-213">Make this change either manually or by creating a database change script.</span></span>

3. <span data-ttu-id="9d396-214">Usare Migrazioni Code First per aggiornare lo schema del database.</span><span class="sxs-lookup"><span data-stu-id="9d396-214">Use Code First Migrations to update the database schema.</span></span>

<span data-ttu-id="9d396-215">Per questa esercitazione usare Migrazioni Code First.</span><span class="sxs-lookup"><span data-stu-id="9d396-215">For this tutorial, use Code First Migrations.</span></span>

<span data-ttu-id="9d396-216">Aggiornare la classe `SeedData` in modo che fornisca un valore per la nuova colonna.</span><span class="sxs-lookup"><span data-stu-id="9d396-216">Update the `SeedData` class so that it provides a value for the new column.</span></span> <span data-ttu-id="9d396-217">Di seguito viene illustrata una modifica di esempio, ma questa modifica viene apportata per ogni `new Movie` blocco.</span><span class="sxs-lookup"><span data-stu-id="9d396-217">A sample change is shown below, but make this change for each `new Movie` block.</span></span>

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie30/Models/SeedDataRating.cs?name=snippet1&highlight=8)]

<span data-ttu-id="9d396-218">Vedere il [file SeedData.cs completato](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie50/Models/SeedDataRating.cs).</span><span class="sxs-lookup"><span data-stu-id="9d396-218">See the [completed SeedData.cs file](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie50/Models/SeedDataRating.cs).</span></span>

<span data-ttu-id="9d396-219">Compilare la soluzione.</span><span class="sxs-lookup"><span data-stu-id="9d396-219">Build the solution.</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="9d396-220">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="9d396-220">Visual Studio</span></span>](#tab/visual-studio)

<a name="pmc"></a>

### <a name="add-a-migration-for-the-rating-field"></a><span data-ttu-id="9d396-221">Aggiungere una migrazione per il campo Rating</span><span class="sxs-lookup"><span data-stu-id="9d396-221">Add a migration for the rating field</span></span>

1. <span data-ttu-id="9d396-222">Dal menu **Strumenti** selezionare **Gestione pacchetti NuGet > Console di Gestione pacchetti**.</span><span class="sxs-lookup"><span data-stu-id="9d396-222">From the **Tools** menu, select **NuGet Package Manager > Package Manager Console**.</span></span>
2. <span data-ttu-id="9d396-223">Nella Console di Gestione pacchetti immettere i comandi seguenti:</span><span class="sxs-lookup"><span data-stu-id="9d396-223">In the PMC, enter the following commands:</span></span>

   ```powershell
   Add-Migration Rating
   Update-Database
   ```

<span data-ttu-id="9d396-224">Il comando `Add-Migration` indica al framework di:</span><span class="sxs-lookup"><span data-stu-id="9d396-224">The `Add-Migration` command tells the framework to:</span></span>

* <span data-ttu-id="9d396-225">Confrontare il `Movie` modello con lo `Movie` schema del database.</span><span class="sxs-lookup"><span data-stu-id="9d396-225">Compare the `Movie` model with the `Movie` database schema.</span></span>
* <span data-ttu-id="9d396-226">Creare il codice per eseguire la migrazione dello schema del database al nuovo modello.</span><span class="sxs-lookup"><span data-stu-id="9d396-226">Create code to migrate the database schema to the new model.</span></span>

<span data-ttu-id="9d396-227">Il nome "Rating" è arbitrario e viene usato per denominare il file di migrazione.</span><span class="sxs-lookup"><span data-stu-id="9d396-227">The name "Rating" is arbitrary and is used to name the migration file.</span></span> <span data-ttu-id="9d396-228">È consigliabile usare un nome significativo per il file di migrazione.</span><span class="sxs-lookup"><span data-stu-id="9d396-228">It's helpful to use a meaningful name for the migration file.</span></span>

<span data-ttu-id="9d396-229">Il `Update-Database` comando indica al Framework di applicare le modifiche dello schema al database e di mantenere i dati esistenti.</span><span class="sxs-lookup"><span data-stu-id="9d396-229">The `Update-Database` command tells the framework to apply the schema changes to the database and to preserve existing data.</span></span>

<a name="ssox"></a>

<span data-ttu-id="9d396-230">Se si eliminano tutti i record nel database, l'inizializzatore eseguirà il seeding del database e includerà il `Rating` campo.</span><span class="sxs-lookup"><span data-stu-id="9d396-230">If you delete all the records in the database, the initializer will seed the database and include the `Rating` field.</span></span> <span data-ttu-id="9d396-231">È possibile eseguire questa operazione con i collegamenti di eliminazione nel browser o da [Sql Server Object Explorer](xref:tutorials/razor-pages/sql#ssox) (SSOX).</span><span class="sxs-lookup"><span data-stu-id="9d396-231">You can do this with the delete links in the browser or from [Sql Server Object Explorer](xref:tutorials/razor-pages/sql#ssox) (SSOX).</span></span>

<span data-ttu-id="9d396-232">Un'altra opzione è quella di eliminare il database e usare le migrazioni per ricreare il database.</span><span class="sxs-lookup"><span data-stu-id="9d396-232">Another option is to delete the database and use migrations to re-create the database.</span></span> <span data-ttu-id="9d396-233">Per eliminare il database da SSOX:</span><span class="sxs-lookup"><span data-stu-id="9d396-233">To delete the database in SSOX:</span></span>

* <span data-ttu-id="9d396-234">Selezionare il database in SSOX.</span><span class="sxs-lookup"><span data-stu-id="9d396-234">Select the database in SSOX.</span></span>
* <span data-ttu-id="9d396-235">Fare clic con il pulsante destro del mouse sul database e scegliere **Elimina**.</span><span class="sxs-lookup"><span data-stu-id="9d396-235">Right-click on the database, and select **Delete**.</span></span>
* <span data-ttu-id="9d396-236">Selezionare **Chiudi connessioni esistenti**.</span><span class="sxs-lookup"><span data-stu-id="9d396-236">Check **Close existing connections**.</span></span>
* <span data-ttu-id="9d396-237">Selezionare **OK**.</span><span class="sxs-lookup"><span data-stu-id="9d396-237">Select **OK**.</span></span>
* <span data-ttu-id="9d396-238">In [PMC](xref:tutorials/razor-pages/new-field#pmc)aggiornare il database:</span><span class="sxs-lookup"><span data-stu-id="9d396-238">In the [PMC](xref:tutorials/razor-pages/new-field#pmc), update the database:</span></span>

  ```powershell
  Update-Database
  ```

# <a name="visual-studio-code--visual-studio-for-mac"></a>[<span data-ttu-id="9d396-239">Visual Studio Code / Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="9d396-239">Visual Studio Code / Visual Studio for Mac</span></span>](#tab/visual-studio-code+visual-studio-mac)

### <a name="drop-and-re-create-the-database"></a><span data-ttu-id="9d396-240">Eliminare e ricreare il database</span><span class="sxs-lookup"><span data-stu-id="9d396-240">Drop and re-create the database</span></span>

> [!NOTE]
> <span data-ttu-id="9d396-241">Per questa esercitazione, usare la funzionalità *migrazioni* di Entity Framework Core laddove possibile.</span><span class="sxs-lookup"><span data-stu-id="9d396-241">For this tutorial you, use the Entity Framework Core *migrations* feature where possible.</span></span> <span data-ttu-id="9d396-242">Le migrazioni aggiornano lo schema del database in base alle modifiche nel modello di dati.</span><span class="sxs-lookup"><span data-stu-id="9d396-242">Migrations updates the database schema to match changes in the data model.</span></span> <span data-ttu-id="9d396-243">Tuttavia, le migrazioni possono eseguire solo i tipi di modifiche supportate dal provider EF Core e le funzionalità del provider SQLite sono limitate.</span><span class="sxs-lookup"><span data-stu-id="9d396-243">However, migrations can only do the kinds of changes that the EF Core provider supports, and the SQLite provider's capabilities are limited.</span></span> <span data-ttu-id="9d396-244">Ad esempio l'aggiunta di una colonna è supportata, ma la rimozione o la modifica di una colonna non è supportata.</span><span class="sxs-lookup"><span data-stu-id="9d396-244">For example, adding a column is supported, but removing or changing a column is not supported.</span></span> <span data-ttu-id="9d396-245">Se si crea una migrazione per rimuovere o modificare una colonna, il comando `ef migrations add` ha esito positivo, ma il comando `ef database update` ha esito negativo.</span><span class="sxs-lookup"><span data-stu-id="9d396-245">If a migration is created to remove or change a column, the `ef migrations add` command succeeds but the `ef database update` command fails.</span></span> <span data-ttu-id="9d396-246">A causa di queste limitazioni, in questa esercitazione non vengono usate le migrazioni per le modifiche dello schema di SQLite.</span><span class="sxs-lookup"><span data-stu-id="9d396-246">Due to these limitations, this tutorial doesn't use migrations for SQLite schema changes.</span></span> <span data-ttu-id="9d396-247">In caso di modifiche dello schema, il database viene invece eliminato e ricreato.</span><span class="sxs-lookup"><span data-stu-id="9d396-247">Instead, when the schema changes, you drop and re-create the database.</span></span>
>
><span data-ttu-id="9d396-248">La soluzione per ovviare alle limitazioni di SQLite consiste nello scrivere manualmente il codice delle migrazioni per eseguire una ricompilazione della tabella in caso di modifiche nella tabella.</span><span class="sxs-lookup"><span data-stu-id="9d396-248">The workaround for the SQLite limitations is to manually write migrations code to perform a table rebuild when something in the table changes.</span></span> <span data-ttu-id="9d396-249">La ricompilazione della tabella comporta:</span><span class="sxs-lookup"><span data-stu-id="9d396-249">A table rebuild involves:</span></span>
>
>* <span data-ttu-id="9d396-250">Creazione di una nuova tabella.</span><span class="sxs-lookup"><span data-stu-id="9d396-250">Creating a new table.</span></span>
>* <span data-ttu-id="9d396-251">Copia dei dati dalla vecchia tabella alla nuova tabella.</span><span class="sxs-lookup"><span data-stu-id="9d396-251">Copying data from the old table to the new table.</span></span>
>* <span data-ttu-id="9d396-252">Eliminazione della tabella precedente.</span><span class="sxs-lookup"><span data-stu-id="9d396-252">Dropping the old table.</span></span>
>* <span data-ttu-id="9d396-253">Ridenominazione della nuova tabella.</span><span class="sxs-lookup"><span data-stu-id="9d396-253">Renaming the new table.</span></span>
>
><span data-ttu-id="9d396-254">Per altre informazioni, vedere le risorse seguenti:</span><span class="sxs-lookup"><span data-stu-id="9d396-254">For more information, see the following resources:</span></span>
>
> * [<span data-ttu-id="9d396-255">Limitazioni del provider di database SQLite per EF Core</span><span class="sxs-lookup"><span data-stu-id="9d396-255">SQLite EF Core Database Provider Limitations</span></span>](/ef/core/providers/sqlite/limitations)
> * [<span data-ttu-id="9d396-256">Personalizzare il codice di migrazione</span><span class="sxs-lookup"><span data-stu-id="9d396-256">Customize migration code</span></span>](/ef/core/managing-schemas/migrations/#customize-migration-code)
> * [<span data-ttu-id="9d396-257">Seeding dei dati</span><span class="sxs-lookup"><span data-stu-id="9d396-257">Data seeding</span></span>](/ef/core/modeling/data-seeding)
> * [<span data-ttu-id="9d396-258">Istruzione ALTER TABLE di SQLite</span><span class="sxs-lookup"><span data-stu-id="9d396-258">SQLite ALTER TABLE statement</span></span>](https://sqlite.org/lang_altertable.html)

1. <span data-ttu-id="9d396-259">Eliminare la cartella della migrazione.</span><span class="sxs-lookup"><span data-stu-id="9d396-259">Delete the migration folder.</span></span>  

1. <span data-ttu-id="9d396-260">Usare i comandi seguenti per ricreare il database.</span><span class="sxs-lookup"><span data-stu-id="9d396-260">Use the following commands to recreate the database.</span></span>

   ```dotnetcli
   dotnet ef database drop
   dotnet ef migrations add InitialCreate
   dotnet ef database update
   ```

---

<span data-ttu-id="9d396-261">Eseguire l'app e verificare che sia possibile creare/modificare/visualizzare i film con un campo `Rating`.</span><span class="sxs-lookup"><span data-stu-id="9d396-261">Run the app and verify you can create/edit/display movies with a `Rating` field.</span></span> <span data-ttu-id="9d396-262">Se il database non è inizializzato, impostare un punto di interruzione nel metodo `SeedData.Initialize`.</span><span class="sxs-lookup"><span data-stu-id="9d396-262">If the database isn't seeded, set a break point in the `SeedData.Initialize` method.</span></span>

## <a name="additional-resources"></a><span data-ttu-id="9d396-263">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="9d396-263">Additional resources</span></span>

> [!div class="step-by-step"]
> <span data-ttu-id="9d396-264">[Precedente: Aggiungi ricerca](xref:tutorials/razor-pages/search) 
>  Passaggio [successivo: aggiungere la convalida](xref:tutorials/razor-pages/validation)</span><span class="sxs-lookup"><span data-stu-id="9d396-264">[Previous: Add Search](xref:tutorials/razor-pages/search)
[Next: Add Validation](xref:tutorials/razor-pages/validation)</span></span>

::: moniker-end

::: moniker range="< aspnetcore-3.0"

<span data-ttu-id="9d396-265">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/tutorials/razor-pages/razor-pages-start) ([procedura per il download](xref:index#how-to-download-a-sample)).</span><span class="sxs-lookup"><span data-stu-id="9d396-265">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/tutorials/razor-pages/razor-pages-start) ([how to download](xref:index#how-to-download-a-sample)).</span></span>

<span data-ttu-id="9d396-266">In questa sezione vengono usate le Migrazioni Code First di [Entity Framework](/ef/core/get-started/aspnetcore/new-db) per:</span><span class="sxs-lookup"><span data-stu-id="9d396-266">In this section [Entity Framework](/ef/core/get-started/aspnetcore/new-db) Code First Migrations is used to:</span></span>

* <span data-ttu-id="9d396-267">Aggiungere un nuovo campo al modello.</span><span class="sxs-lookup"><span data-stu-id="9d396-267">Add a new field to the model.</span></span>
* <span data-ttu-id="9d396-268">Eseguire la migrazione nel database della modifica al nuovo schema del campo.</span><span class="sxs-lookup"><span data-stu-id="9d396-268">Migrate the new field schema change to the database.</span></span>

<span data-ttu-id="9d396-269">Quando si usa Code First di Entity Framework per creare automaticamente un database, Code First:</span><span class="sxs-lookup"><span data-stu-id="9d396-269">When using EF Code First to automatically create a database, Code First:</span></span>

* <span data-ttu-id="9d396-270">Aggiunge una [`__EFMigrationsHistory`](/ef/core/managing-schemas/migrations/history-table) tabella al database per rilevare se lo schema del database è sincronizzato con le classi del modello da cui è stato generato.</span><span class="sxs-lookup"><span data-stu-id="9d396-270">Adds an [`__EFMigrationsHistory`](/ef/core/managing-schemas/migrations/history-table) table to the database to track whether the schema of the database is in sync with the model classes it was generated from.</span></span>
* <span data-ttu-id="9d396-271">Se le classi del modello non sono sincronizzate con il database, EF genera un'eccezione.</span><span class="sxs-lookup"><span data-stu-id="9d396-271">If the model classes aren't in sync with the database, EF throws an exception.</span></span>

<span data-ttu-id="9d396-272">Con la verifica automatica che lo schema e il modello sono sincronizzati, è più semplice individuare problemi di codice di database incoerenti.</span><span class="sxs-lookup"><span data-stu-id="9d396-272">Automatic verification that the schema and model are in sync makes it easier to find inconsistent database code issues.</span></span>

## <a name="adding-a-rating-property-to-the-movie-model"></a><span data-ttu-id="9d396-273">Aggiunta di una proprietà Rating al modello Movie</span><span class="sxs-lookup"><span data-stu-id="9d396-273">Adding a Rating Property to the Movie Model</span></span>

<span data-ttu-id="9d396-274">Aprire il file *Models/Movie.cs* e aggiungere una proprietà `Rating`:</span><span class="sxs-lookup"><span data-stu-id="9d396-274">Open the *Models/Movie.cs* file and add a `Rating` property:</span></span>

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie22/Models/MovieDateRating.cs?highlight=13&name=snippet)]

<span data-ttu-id="9d396-275">Compilare l'app.</span><span class="sxs-lookup"><span data-stu-id="9d396-275">Build the app.</span></span>

<span data-ttu-id="9d396-276">Modificare *pages/Movies/ Index . cshtml* e aggiungere un `Rating` campo:</span><span class="sxs-lookup"><span data-stu-id="9d396-276">Edit *Pages/Movies/Index.cshtml*, and add a `Rating` field:</span></span>

[!code-cshtml[](razor-pages-start/sample/RazorPagesMovie22/Pages/Movies/IndexRating.cshtml?highlight=40-42,61-63)]

<span data-ttu-id="9d396-277">Aggiornare le pagine seguenti:</span><span class="sxs-lookup"><span data-stu-id="9d396-277">Update the following pages:</span></span>

* <span data-ttu-id="9d396-278">Aggiungere il campo `Rating` alle pagine Delete (Elimina) e Details (Dettagli).</span><span class="sxs-lookup"><span data-stu-id="9d396-278">Add the `Rating` field to the Delete and Details pages.</span></span>
* <span data-ttu-id="9d396-279">Aggiornare [Create.cshtml](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie22/Pages/Movies/Create.cshtml) con un campo `Rating`.</span><span class="sxs-lookup"><span data-stu-id="9d396-279">Update [Create.cshtml](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie22/Pages/Movies/Create.cshtml) with a `Rating` field.</span></span>
* <span data-ttu-id="9d396-280">Aggiungere il campo `Rating` alla pagina Edit (Modifica).</span><span class="sxs-lookup"><span data-stu-id="9d396-280">Add the `Rating` field to the Edit Page.</span></span>

<span data-ttu-id="9d396-281">L'app non funzionerà finché il database non verrà aggiornato in modo da includere il nuovo campo.</span><span class="sxs-lookup"><span data-stu-id="9d396-281">The app won't work until the database is updated to include the new field.</span></span> <span data-ttu-id="9d396-282">Se l'app viene eseguita adesso, l'app genera un'eccezione `SqlException` :</span><span class="sxs-lookup"><span data-stu-id="9d396-282">If the app is run now, the app throws a `SqlException`:</span></span>

`SqlException: Invalid column name 'Rating'.`

<span data-ttu-id="9d396-283">Questo errore viene visualizzato perché la classe del modello Movie aggiornata è diversa rispetto allo schema della tabella Movie nel database.</span><span class="sxs-lookup"><span data-stu-id="9d396-283">This error is caused by the updated Movie model class being different than the schema of the Movie table of the database.</span></span> <span data-ttu-id="9d396-284">Non è presente alcuna `Rating` colonna nella tabella di database.</span><span class="sxs-lookup"><span data-stu-id="9d396-284">There's no `Rating` column in the database table.</span></span>

<span data-ttu-id="9d396-285">Per correggere questo errore, esistono alcuni approcci:</span><span class="sxs-lookup"><span data-stu-id="9d396-285">There are a few approaches to resolving the error:</span></span>

1. <span data-ttu-id="9d396-286">Fare in modo che Entity Framework elimini e crei di nuovo automaticamente il database usando il nuovo schema di classi del modello.</span><span class="sxs-lookup"><span data-stu-id="9d396-286">Have the Entity Framework automatically drop and re-create the database using the new model class schema.</span></span> <span data-ttu-id="9d396-287">Questo approccio è utile nelle prime fasi del ciclo di sviluppo e consente di sviluppare rapidamente lo schema del modello e del database insieme.</span><span class="sxs-lookup"><span data-stu-id="9d396-287">This approach is convenient early in the development cycle, it allows you to quickly evolve the model and database schema together.</span></span> <span data-ttu-id="9d396-288">Lo svantaggio è che si perdono i dati esistenti nel database.</span><span class="sxs-lookup"><span data-stu-id="9d396-288">The downside is that you lose existing data in the database.</span></span> <span data-ttu-id="9d396-289">Non usare questo approccio in un database di produzione.</span><span class="sxs-lookup"><span data-stu-id="9d396-289">Don't use this approach on a production database!</span></span> <span data-ttu-id="9d396-290">L'eliminazione del database in caso di modifiche dello schema e l'utilizzo di un inizializzatore per eseguire automaticamente il seeding del database con dati di test è spesso un modo produttivo per sviluppare un'app.</span><span class="sxs-lookup"><span data-stu-id="9d396-290">Dropping the database on schema changes and using an initializer to automatically seed the database with test data is often a productive way to develop an app.</span></span>

2. <span data-ttu-id="9d396-291">Modificare esplicitamente lo schema del database esistente in modo che corrisponda alle classi del modello.</span><span class="sxs-lookup"><span data-stu-id="9d396-291">Explicitly modify the schema of the existing database so that it matches the model classes.</span></span> <span data-ttu-id="9d396-292">Il vantaggio di questo approccio consiste nel preservare i dati.</span><span class="sxs-lookup"><span data-stu-id="9d396-292">The advantage of this approach is to keep the data.</span></span> <span data-ttu-id="9d396-293">Apportare questa modifica manualmente o creando uno script di modifica del database.</span><span class="sxs-lookup"><span data-stu-id="9d396-293">Make this change either manually or by creating a database change script.</span></span>

3. <span data-ttu-id="9d396-294">Usare Migrazioni Code First per aggiornare lo schema del database.</span><span class="sxs-lookup"><span data-stu-id="9d396-294">Use Code First Migrations to update the database schema.</span></span>

<span data-ttu-id="9d396-295">Per questa esercitazione usare Migrazioni Code First.</span><span class="sxs-lookup"><span data-stu-id="9d396-295">For this tutorial, use Code First Migrations.</span></span>

<span data-ttu-id="9d396-296">Aggiornare la classe `SeedData` in modo che fornisca un valore per la nuova colonna.</span><span class="sxs-lookup"><span data-stu-id="9d396-296">Update the `SeedData` class so that it provides a value for the new column.</span></span> <span data-ttu-id="9d396-297">Di seguito viene illustrata una modifica di esempio, ma questa modifica viene apportata per ogni `new Movie` blocco.</span><span class="sxs-lookup"><span data-stu-id="9d396-297">A sample change is shown below, but make this change for each `new Movie` block.</span></span>

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie22/Models/SeedDataRating.cs?name=snippet1&highlight=8)]

<span data-ttu-id="9d396-298">Vedere il [file SeedData.cs completato](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie22/Models/SeedDataRating.cs).</span><span class="sxs-lookup"><span data-stu-id="9d396-298">See the [completed SeedData.cs file](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie22/Models/SeedDataRating.cs).</span></span>

<span data-ttu-id="9d396-299">Compilare la soluzione.</span><span class="sxs-lookup"><span data-stu-id="9d396-299">Build the solution.</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="9d396-300">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="9d396-300">Visual Studio</span></span>](#tab/visual-studio)

<a name="pmc"></a>

### <a name="add-a-migration-for-the-rating-field"></a><span data-ttu-id="9d396-301">Aggiungere una migrazione per il campo Rating</span><span class="sxs-lookup"><span data-stu-id="9d396-301">Add a migration for the rating field</span></span>

<span data-ttu-id="9d396-302">Dal menu **Strumenti** selezionare **Gestione pacchetti NuGet > Console di Gestione pacchetti**.</span><span class="sxs-lookup"><span data-stu-id="9d396-302">From the **Tools** menu, select **NuGet Package Manager > Package Manager Console**.</span></span>
<span data-ttu-id="9d396-303">Nella Console di Gestione pacchetti immettere i comandi seguenti:</span><span class="sxs-lookup"><span data-stu-id="9d396-303">In the PMC, enter the following commands:</span></span>

```powershell
Add-Migration Rating
Update-Database
```

<span data-ttu-id="9d396-304">Il comando `Add-Migration` indica al framework di:</span><span class="sxs-lookup"><span data-stu-id="9d396-304">The `Add-Migration` command tells the framework to:</span></span>

* <span data-ttu-id="9d396-305">Confrontare il `Movie` modello con lo `Movie` schema del database.</span><span class="sxs-lookup"><span data-stu-id="9d396-305">Compare the `Movie` model with the `Movie` database schema.</span></span>
* <span data-ttu-id="9d396-306">Creare il codice per eseguire la migrazione dello schema del database al nuovo modello.</span><span class="sxs-lookup"><span data-stu-id="9d396-306">Create code to migrate the database schema to the new model.</span></span>

<span data-ttu-id="9d396-307">Il nome "Rating" è arbitrario e viene usato per denominare il file di migrazione.</span><span class="sxs-lookup"><span data-stu-id="9d396-307">The name "Rating" is arbitrary and is used to name the migration file.</span></span> <span data-ttu-id="9d396-308">È consigliabile usare un nome significativo per il file di migrazione.</span><span class="sxs-lookup"><span data-stu-id="9d396-308">It's helpful to use a meaningful name for the migration file.</span></span>

<span data-ttu-id="9d396-309">Il comando `Update-Database` indica al framework di applicare le modifiche dello schema al database.</span><span class="sxs-lookup"><span data-stu-id="9d396-309">The `Update-Database` command tells the framework to apply the schema changes to the database.</span></span>

<a name="ssox"></a>

<span data-ttu-id="9d396-310">Se si eliminano tutti i record nel database, l'inizializzatore eseguirà il seeding del database e includerà il `Rating` campo.</span><span class="sxs-lookup"><span data-stu-id="9d396-310">If you delete all the records in the database, the initializer will seed the database and include the `Rating` field.</span></span> <span data-ttu-id="9d396-311">È possibile eseguire questa operazione con i collegamenti di eliminazione nel browser o da [Sql Server Object Explorer](xref:tutorials/razor-pages/sql#ssox) (SSOX).</span><span class="sxs-lookup"><span data-stu-id="9d396-311">You can do this with the delete links in the browser or from [Sql Server Object Explorer](xref:tutorials/razor-pages/sql#ssox) (SSOX).</span></span>

<span data-ttu-id="9d396-312">Un'altra opzione è quella di eliminare il database e usare le migrazioni per ricreare il database.</span><span class="sxs-lookup"><span data-stu-id="9d396-312">Another option is to delete the database and use migrations to re-create the database.</span></span> <span data-ttu-id="9d396-313">Per eliminare il database da SSOX:</span><span class="sxs-lookup"><span data-stu-id="9d396-313">To delete the database in SSOX:</span></span>

* <span data-ttu-id="9d396-314">Selezionare il database in SSOX.</span><span class="sxs-lookup"><span data-stu-id="9d396-314">Select the database in SSOX.</span></span>
* <span data-ttu-id="9d396-315">Fare clic con il pulsante destro del mouse sul database e scegliere **Elimina**.</span><span class="sxs-lookup"><span data-stu-id="9d396-315">Right-click on the database, and select **Delete**.</span></span>
* <span data-ttu-id="9d396-316">Selezionare **Chiudi connessioni esistenti**.</span><span class="sxs-lookup"><span data-stu-id="9d396-316">Check **Close existing connections**.</span></span>
* <span data-ttu-id="9d396-317">Selezionare **OK**.</span><span class="sxs-lookup"><span data-stu-id="9d396-317">Select **OK**.</span></span>
* <span data-ttu-id="9d396-318">In [PMC](xref:tutorials/razor-pages/new-field#pmc)aggiornare il database:</span><span class="sxs-lookup"><span data-stu-id="9d396-318">In the [PMC](xref:tutorials/razor-pages/new-field#pmc), update the database:</span></span>

  ```powershell
  Update-Database
  ```

# <a name="visual-studio-code--visual-studio-for-mac"></a>[<span data-ttu-id="9d396-319">Visual Studio Code / Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="9d396-319">Visual Studio Code / Visual Studio for Mac</span></span>](#tab/visual-studio-code+visual-studio-mac)

### <a name="drop-and-re-create-the-database"></a><span data-ttu-id="9d396-320">Eliminare e ricreare il database</span><span class="sxs-lookup"><span data-stu-id="9d396-320">Drop and re-create the database</span></span>

> [!NOTE]
> <span data-ttu-id="9d396-321">Per questa esercitazione, usare la funzionalità *migrazioni* di Entity Framework Core laddove possibile.</span><span class="sxs-lookup"><span data-stu-id="9d396-321">For this tutorial you, use the Entity Framework Core *migrations* feature where possible.</span></span> <span data-ttu-id="9d396-322">Le migrazioni aggiornano lo schema del database in base alle modifiche nel modello di dati.</span><span class="sxs-lookup"><span data-stu-id="9d396-322">Migrations updates the database schema to match changes in the data model.</span></span> <span data-ttu-id="9d396-323">Tuttavia, le migrazioni possono eseguire solo i tipi di modifiche supportate dal provider EF Core e le funzionalità del provider SQLite sono limitate.</span><span class="sxs-lookup"><span data-stu-id="9d396-323">However, migrations can only do the kinds of changes that the EF Core provider supports, and the SQLite provider's capabilities are limited.</span></span> <span data-ttu-id="9d396-324">Ad esempio l'aggiunta di una colonna è supportata, ma la rimozione o la modifica di una colonna non è supportata.</span><span class="sxs-lookup"><span data-stu-id="9d396-324">For example, adding a column is supported, but removing or changing a column is not supported.</span></span> <span data-ttu-id="9d396-325">Se si crea una migrazione per rimuovere o modificare una colonna, il comando `ef migrations add` ha esito positivo, ma il comando `ef database update` ha esito negativo.</span><span class="sxs-lookup"><span data-stu-id="9d396-325">If a migration is created to remove or change a column, the `ef migrations add` command succeeds but the `ef database update` command fails.</span></span> <span data-ttu-id="9d396-326">A causa di queste limitazioni, in questa esercitazione non vengono usate le migrazioni per le modifiche dello schema di SQLite.</span><span class="sxs-lookup"><span data-stu-id="9d396-326">Due to these limitations, this tutorial doesn't use migrations for SQLite schema changes.</span></span> <span data-ttu-id="9d396-327">In caso di modifiche dello schema, il database viene invece eliminato e ricreato.</span><span class="sxs-lookup"><span data-stu-id="9d396-327">Instead, when the schema changes, you drop and re-create the database.</span></span>
>
><span data-ttu-id="9d396-328">La soluzione per ovviare alle limitazioni di SQLite consiste nello scrivere manualmente il codice delle migrazioni per eseguire una ricompilazione della tabella in caso di modifiche nella tabella.</span><span class="sxs-lookup"><span data-stu-id="9d396-328">The workaround for the SQLite limitations is to manually write migrations code to perform a table rebuild when something in the table changes.</span></span> <span data-ttu-id="9d396-329">La ricompilazione della tabella comporta:</span><span class="sxs-lookup"><span data-stu-id="9d396-329">A table rebuild involves:</span></span>
>
>* <span data-ttu-id="9d396-330">Creazione di una nuova tabella.</span><span class="sxs-lookup"><span data-stu-id="9d396-330">Creating a new table.</span></span>
>* <span data-ttu-id="9d396-331">Copia dei dati dalla vecchia tabella alla nuova tabella.</span><span class="sxs-lookup"><span data-stu-id="9d396-331">Copying data from the old table to the new table.</span></span>
>* <span data-ttu-id="9d396-332">Eliminazione della tabella precedente.</span><span class="sxs-lookup"><span data-stu-id="9d396-332">Dropping the old table.</span></span>
>* <span data-ttu-id="9d396-333">Ridenominazione della nuova tabella.</span><span class="sxs-lookup"><span data-stu-id="9d396-333">Renaming the new table.</span></span>
>
><span data-ttu-id="9d396-334">Per altre informazioni, vedere le risorse seguenti:</span><span class="sxs-lookup"><span data-stu-id="9d396-334">For more information, see the following resources:</span></span>
>
> * [<span data-ttu-id="9d396-335">Limitazioni del provider di database SQLite per EF Core</span><span class="sxs-lookup"><span data-stu-id="9d396-335">SQLite EF Core Database Provider Limitations</span></span>](/ef/core/providers/sqlite/limitations)
> * [<span data-ttu-id="9d396-336">Personalizzare il codice di migrazione</span><span class="sxs-lookup"><span data-stu-id="9d396-336">Customize migration code</span></span>](/ef/core/managing-schemas/migrations/#customize-migration-code)
> * [<span data-ttu-id="9d396-337">Seeding dei dati</span><span class="sxs-lookup"><span data-stu-id="9d396-337">Data seeding</span></span>](/ef/core/modeling/data-seeding)
> * [<span data-ttu-id="9d396-338">Istruzione ALTER TABLE di SQLite</span><span class="sxs-lookup"><span data-stu-id="9d396-338">SQLite ALTER TABLE statement</span></span>](https://sqlite.org/lang_altertable.html)

<span data-ttu-id="9d396-339">Eliminare il database e usare le migrazioni per ricreare il database.</span><span class="sxs-lookup"><span data-stu-id="9d396-339">Delete the database and use migrations to re-create the database.</span></span> <span data-ttu-id="9d396-340">Per eliminare il database, eliminare il file di database (*MvcMovie.db*).</span><span class="sxs-lookup"><span data-stu-id="9d396-340">To delete the database, delete the database file (*MvcMovie.db*).</span></span> <span data-ttu-id="9d396-341">Eseguire quindi il comando `ef database update`:</span><span class="sxs-lookup"><span data-stu-id="9d396-341">Then run the `ef database update` command:</span></span>

```dotnetcli
dotnet ef database update
```

---

<span data-ttu-id="9d396-342">Eseguire l'app e verificare che sia possibile creare/modificare/visualizzare i film con un campo `Rating`.</span><span class="sxs-lookup"><span data-stu-id="9d396-342">Run the app and verify you can create/edit/display movies with a `Rating` field.</span></span> <span data-ttu-id="9d396-343">Se il database non è inizializzato, impostare un punto di interruzione nel metodo `SeedData.Initialize`.</span><span class="sxs-lookup"><span data-stu-id="9d396-343">If the database isn't seeded, set a break point in the `SeedData.Initialize` method.</span></span>

## <a name="additional-resources"></a><span data-ttu-id="9d396-344">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="9d396-344">Additional resources</span></span>

* [<span data-ttu-id="9d396-345">Versione YouTube dell'esercitazione</span><span class="sxs-lookup"><span data-stu-id="9d396-345">YouTube version of this tutorial</span></span>](https://youtu.be/3i7uMxiGGR8)

> [!div class="step-by-step"]
> <span data-ttu-id="9d396-346">[Precedente: Aggiungi ricerca](xref:tutorials/razor-pages/search) 
>  Passaggio [successivo: aggiungere la convalida](xref:tutorials/razor-pages/validation)</span><span class="sxs-lookup"><span data-stu-id="9d396-346">[Previous: Add Search](xref:tutorials/razor-pages/search)
[Next: Add Validation](xref:tutorials/razor-pages/validation)</span></span>

::: moniker-end