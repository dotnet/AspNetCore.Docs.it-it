---
title: Parte 4, utilizzo di un database
author: rick-anderson
description: Parte 4 della serie di esercitazioni sulle Razor pagine.
ms.author: riande
ms.date: 01/05/2021
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
uid: tutorials/razor-pages/sql
ms.openlocfilehash: 3133ff7a192f5e3bdec4f6f1e6e6d55f966d7e64
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102589764"
---
# <a name="part-4-of-tutorial-series-on-razor-pages"></a><span data-ttu-id="276dd-103">Parte 4 della serie di esercitazioni sulle Razor pagine</span><span class="sxs-lookup"><span data-stu-id="276dd-103">Part 4 of tutorial series on Razor Pages</span></span>

<span data-ttu-id="276dd-104">Di [Rick Anderson](https://twitter.com/RickAndMSFT) e [Joe Audette](https://twitter.com/joeaudette)</span><span class="sxs-lookup"><span data-stu-id="276dd-104">By [Rick Anderson](https://twitter.com/RickAndMSFT) and [Joe Audette](https://twitter.com/joeaudette)</span></span>

::: moniker range=">= aspnetcore-5.0"

<span data-ttu-id="276dd-105">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie50) ([procedura per il download](xref:index#how-to-download-a-sample)).</span><span class="sxs-lookup"><span data-stu-id="276dd-105">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie50) ([how to download](xref:index#how-to-download-a-sample)).</span></span>

<span data-ttu-id="276dd-106">L'oggetto `RazorPagesMovieContext` gestisce l'attività di connessione al database e di mapping degli oggetti `Movie` ai record di database.</span><span class="sxs-lookup"><span data-stu-id="276dd-106">The `RazorPagesMovieContext` object handles the task of connecting to the database and mapping `Movie` objects to database records.</span></span> <span data-ttu-id="276dd-107">Il contesto del database viene registrato con il contenitore di [inserimento delle dipendenze](xref:fundamentals/dependency-injection) nel metodo `ConfigureServices` di *Startup.cs*:</span><span class="sxs-lookup"><span data-stu-id="276dd-107">The database context is registered with the [Dependency Injection](xref:fundamentals/dependency-injection) container in the `ConfigureServices` method in *Startup.cs*:</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="276dd-108">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="276dd-108">Visual Studio</span></span>](#tab/visual-studio)

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie50/Startup.cs?name=snippet_ConfigureServices&highlight=5-6)]

# <a name="visual-studio-code--visual-studio-for-mac"></a>[<span data-ttu-id="276dd-109">Visual Studio Code / Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="276dd-109">Visual Studio Code / Visual Studio for Mac</span></span>](#tab/visual-studio-code+visual-studio-mac)

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie50/Startup.cs?name=snippet_UseSqlite&highlight=5-6)]

---

<span data-ttu-id="276dd-110">La chiave viene letta dal sistema di [configurazione](xref:fundamentals/configuration/index) ASP.NET Core `ConnectionString` .</span><span class="sxs-lookup"><span data-stu-id="276dd-110">The ASP.NET Core [Configuration](xref:fundamentals/configuration/index) system reads the `ConnectionString` key.</span></span> <span data-ttu-id="276dd-111">Per lo sviluppo locale, la configurazione ottiene la stringa di connessione dal *appsettings.json* file.</span><span class="sxs-lookup"><span data-stu-id="276dd-111">For local development, configuration gets the connection string from the *appsettings.json* file.</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="276dd-112">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="276dd-112">Visual Studio</span></span>](#tab/visual-studio)

<span data-ttu-id="276dd-113">La stringa di connessione generata è simile al codice JSON seguente:</span><span class="sxs-lookup"><span data-stu-id="276dd-113">The generated connection string is similar to the following JSON:</span></span>

[!code-json[](razor-pages-start/sample/RazorPagesMovie50/appsettings.json?highlight=10-12)]

# <a name="visual-studio-code--visual-studio-for-mac"></a>[<span data-ttu-id="276dd-114">Visual Studio Code / Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="276dd-114">Visual Studio Code / Visual Studio for Mac</span></span>](#tab/visual-studio-code+visual-studio-mac)

[!code-json[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie50/appsettings_SQLite.json?highlight=10-12)]

---

<span data-ttu-id="276dd-115">Quando l'app viene distribuita in un server di test o di produzione, è possibile usare una variabile di ambiente per impostare la stringa di connessione su un server di database di test o di produzione.</span><span class="sxs-lookup"><span data-stu-id="276dd-115">When the app is deployed to a test or production server, an environment variable can be used to set the connection string to a test or production database server.</span></span> <span data-ttu-id="276dd-116">Per ulteriori informazioni, vedere [configurazione](xref:fundamentals/configuration/index).</span><span class="sxs-lookup"><span data-stu-id="276dd-116">For more information, see [Configuration](xref:fundamentals/configuration/index).</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="276dd-117">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="276dd-117">Visual Studio</span></span>](#tab/visual-studio)

## <a name="sql-server-express-localdb"></a><span data-ttu-id="276dd-118">SQL Server Express LocalDB</span><span class="sxs-lookup"><span data-stu-id="276dd-118">SQL Server Express LocalDB</span></span>

<span data-ttu-id="276dd-119">Local DB è una versione leggera del motore di database di SQL Server Express appositamente pensato per lo sviluppo di programmi.</span><span class="sxs-lookup"><span data-stu-id="276dd-119">LocalDB is a lightweight version of the SQL Server Express database engine that's targeted for program development.</span></span> <span data-ttu-id="276dd-120">Local DB viene avviato su richiesta ed eseguito in modalità utente; non richiede quindi una configurazione complessa.</span><span class="sxs-lookup"><span data-stu-id="276dd-120">LocalDB starts on demand and runs in user mode, so there's no complex configuration.</span></span> <span data-ttu-id="276dd-121">Per impostazione predefinita, Local DB crea file con estensione `*.mdf` nella directory `C:\Users\<user>\`.</span><span class="sxs-lookup"><span data-stu-id="276dd-121">By default, LocalDB database creates `*.mdf` files in the `C:\Users\<user>\` directory.</span></span>

<a name="ssox"></a>
1. <span data-ttu-id="276dd-122">Dal menu **Visualizzazione** aprire **Esplora oggetti di SQL Server** (SSOX).</span><span class="sxs-lookup"><span data-stu-id="276dd-122">From the **View** menu, open **SQL Server Object Explorer** (SSOX).</span></span>

   ![Menu Visualizza](sql/_static/5/ssox.png)

1. <span data-ttu-id="276dd-124">Fare clic con il pulsante destro del mouse sulla `Movie` tabella e scegliere **Visualizza finestra di progettazione**:</span><span class="sxs-lookup"><span data-stu-id="276dd-124">Right-click on the `Movie` table and select **View Designer**:</span></span>

   ![Menu di scelta rapida aperti per la tabella Movie](sql/_static/5/design.png)

   ![Tabelle Movie aperte nella finestra di progettazione](sql/_static/dv.png)

   <span data-ttu-id="276dd-127">Si noti l'icona a forma di chiave accanto a `ID`.</span><span class="sxs-lookup"><span data-stu-id="276dd-127">Note the key icon next to `ID`.</span></span> <span data-ttu-id="276dd-128">Per impostazione predefinita, Entity Framework crea una proprietà denominata `ID` per la chiave primaria.</span><span class="sxs-lookup"><span data-stu-id="276dd-128">By default, EF creates a property named `ID` for the primary key.</span></span>

1. <span data-ttu-id="276dd-129">Fare clic con il pulsante destro del mouse sulla `Movie` tabella e selezionare **Visualizza dati**:</span><span class="sxs-lookup"><span data-stu-id="276dd-129">Right-click on the `Movie` table and select **View Data**:</span></span>

   ![Tabella Movie aperta con i dati della tabella](sql/_static/vd22.png)

# <a name="visual-studio-code--visual-studio-for-mac"></a>[<span data-ttu-id="276dd-131">Visual Studio Code / Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="276dd-131">Visual Studio Code / Visual Studio for Mac</span></span>](#tab/visual-studio-code+visual-studio-mac)

## <a name="sqlite"></a><span data-ttu-id="276dd-132">SQLite</span><span class="sxs-lookup"><span data-stu-id="276dd-132">SQLite</span></span>

<span data-ttu-id="276dd-133">Nel sito Web di [SQLite](https://www.sqlite.org/) si dichiara:</span><span class="sxs-lookup"><span data-stu-id="276dd-133">The [SQLite](https://www.sqlite.org/) website states:</span></span>

> <span data-ttu-id="276dd-134">SQLite è un motore di database SQL autonomo, a elevata affidabilità, incorporato, completo e di dominio pubblico.</span><span class="sxs-lookup"><span data-stu-id="276dd-134">SQLite is a self-contained, high-reliability, embedded, full-featured, public-domain, SQL database engine.</span></span> <span data-ttu-id="276dd-135">SQLite è il motore di database più usato in tutto il mondo.</span><span class="sxs-lookup"><span data-stu-id="276dd-135">SQLite is the most used database engine in the world.</span></span>

<span data-ttu-id="276dd-136">Sono disponibili molti strumenti di terze parti che è possibile scaricare per gestire e visualizzare un database SQLite.</span><span class="sxs-lookup"><span data-stu-id="276dd-136">There are many third-party tools you can download to manage and view a SQLite database.</span></span> <span data-ttu-id="276dd-137">L'immagine seguente è stata tratta da [DB Browser per SQLite](https://sqlitebrowser.org/).</span><span class="sxs-lookup"><span data-stu-id="276dd-137">The image below is from [DB Browser for SQLite](https://sqlitebrowser.org/).</span></span> <span data-ttu-id="276dd-138">Se si preferisce uno strumento SQLite in particolare, aggiungere un commento sui motivi della preferenza.</span><span class="sxs-lookup"><span data-stu-id="276dd-138">If you have a favorite SQLite tool, leave a comment on what you like about it.</span></span>

![Browser DB per SQLite con database di film](~/tutorials/first-mvc-app-xplat/working-with-sql/_static/dbb.png)

> [!NOTE]
> <span data-ttu-id="276dd-140">Per questa esercitazione, viene usata la funzionalità *migrazioni* di Entity Framework Core, laddove possibile.</span><span class="sxs-lookup"><span data-stu-id="276dd-140">For this tutorial, the Entity Framework Core *migrations* feature is used where possible.</span></span> <span data-ttu-id="276dd-141">Le migrazioni aggiornano lo schema del database in base alle modifiche nel modello di dati.</span><span class="sxs-lookup"><span data-stu-id="276dd-141">Migrations updates the database schema to match changes in the data model.</span></span> <span data-ttu-id="276dd-142">Tuttavia, le migrazioni possono eseguire solo i tipi di modifiche supportate dal provider EF Core e le funzionalità del provider SQLite sono limitate.</span><span class="sxs-lookup"><span data-stu-id="276dd-142">However, migrations can only do the kinds of changes that the EF Core provider supports, and the SQLite provider's capabilities are limited.</span></span> <span data-ttu-id="276dd-143">Ad esempio l'aggiunta di una colonna è supportata, ma la rimozione o la modifica di una colonna non è supportata.</span><span class="sxs-lookup"><span data-stu-id="276dd-143">For example, adding a column is supported, but removing or changing a column is not supported.</span></span> <span data-ttu-id="276dd-144">Se si crea una migrazione per rimuovere o modificare una colonna, il comando `ef migrations add` ha esito positivo, ma il comando `ef database update` ha esito negativo.</span><span class="sxs-lookup"><span data-stu-id="276dd-144">If a migration is created to remove or change a column, the `ef migrations add` command succeeds but the `ef database update` command fails.</span></span> <span data-ttu-id="276dd-145">A causa di queste limitazioni, in questa esercitazione non vengono usate le migrazioni per le modifiche dello schema di SQLite.</span><span class="sxs-lookup"><span data-stu-id="276dd-145">Due to these limitations, this tutorial doesn't use migrations for SQLite schema changes.</span></span> <span data-ttu-id="276dd-146">Al contrario, quando lo schema viene modificato, il database viene eliminato e ricreato.</span><span class="sxs-lookup"><span data-stu-id="276dd-146">Instead, when the schema changes, the database is dropped and re-created.</span></span>
>
><span data-ttu-id="276dd-147">La soluzione per ovviare alle limitazioni di SQLite consiste nello scrivere manualmente il codice delle migrazioni per eseguire una ricompilazione della tabella in caso di modifiche nella tabella.</span><span class="sxs-lookup"><span data-stu-id="276dd-147">The workaround for the SQLite limitations is to manually write migrations code to perform a table rebuild when something in the table changes.</span></span> <span data-ttu-id="276dd-148">La ricompilazione della tabella comporta:</span><span class="sxs-lookup"><span data-stu-id="276dd-148">A table rebuild involves:</span></span>
>
>* <span data-ttu-id="276dd-149">Creazione di una nuova tabella.</span><span class="sxs-lookup"><span data-stu-id="276dd-149">Creating a new table.</span></span>
>* <span data-ttu-id="276dd-150">Copia dei dati dalla vecchia tabella alla nuova tabella.</span><span class="sxs-lookup"><span data-stu-id="276dd-150">Copying data from the old table to the new table.</span></span>
>* <span data-ttu-id="276dd-151">Eliminazione della tabella precedente.</span><span class="sxs-lookup"><span data-stu-id="276dd-151">Dropping the old table.</span></span>
>* <span data-ttu-id="276dd-152">Ridenominazione della nuova tabella.</span><span class="sxs-lookup"><span data-stu-id="276dd-152">Renaming the new table.</span></span>
>
><span data-ttu-id="276dd-153">Per altre informazioni, vedere le risorse seguenti:</span><span class="sxs-lookup"><span data-stu-id="276dd-153">For more information, see the following resources:</span></span>
> * [<span data-ttu-id="276dd-154">Limitazioni del provider di database SQLite per EF Core</span><span class="sxs-lookup"><span data-stu-id="276dd-154">SQLite EF Core Database Provider Limitations</span></span>](/ef/core/providers/sqlite/limitations)
> * [<span data-ttu-id="276dd-155">Personalizzare il codice di migrazione</span><span class="sxs-lookup"><span data-stu-id="276dd-155">Customize migration code</span></span>](/ef/core/managing-schemas/migrations/#customize-migration-code)
> * [<span data-ttu-id="276dd-156">Seeding dei dati</span><span class="sxs-lookup"><span data-stu-id="276dd-156">Data seeding</span></span>](/ef/core/modeling/data-seeding)
> * [<span data-ttu-id="276dd-157">Istruzione ALTER TABLE di SQLite</span><span class="sxs-lookup"><span data-stu-id="276dd-157">SQLite ALTER TABLE statement</span></span>](https://sqlite.org/lang_altertable.html)

---

## <a name="seed-the-database"></a><span data-ttu-id="276dd-158">Specificare il valore di inizializzazione del database</span><span class="sxs-lookup"><span data-stu-id="276dd-158">Seed the database</span></span>

<span data-ttu-id="276dd-159">Creare una nuova classe denominata `SeedData` nella cartella *Models* usando il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="276dd-159">Create a new class named `SeedData` in the *Models* folder with the following code:</span></span>

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie30/Models/SeedData.cs?name=snippet_1)]

<span data-ttu-id="276dd-160">Se sono presenti film nel database, l'inizializzatore del valore di inizializzazione viene restituito e non viene aggiunto alcun film.</span><span class="sxs-lookup"><span data-stu-id="276dd-160">If there are any movies in the database, the seed initializer returns and no movies are added.</span></span>

```csharp
if (context.Movie.Any())
{
    return;
}
```

<a name="si"></a>

### <a name="add-the-seed-initializer"></a><span data-ttu-id="276dd-161">Aggiungere l'inizializzatore del valore di inizializzazione</span><span class="sxs-lookup"><span data-stu-id="276dd-161">Add the seed initializer</span></span>

<span data-ttu-id="276dd-162">Sostituire il contenuto di *Program.cs* con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="276dd-162">Replace the contents of the *Program.cs* with the following code:</span></span>

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie50/Program.cs)]

<span data-ttu-id="276dd-163">Nel codice precedente, il `Main` metodo è stato modificato per eseguire le operazioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="276dd-163">In the previous code, the `Main` method has been modified to do the following:</span></span>

* <span data-ttu-id="276dd-164">Ottenere un'istanza del contesto di database dal contenitore di inserimento delle dipendenze.</span><span class="sxs-lookup"><span data-stu-id="276dd-164">Get a database context instance from the dependency injection container.</span></span>
* <span data-ttu-id="276dd-165">Chiamare il `seedData.Initialize` metodo passando all'istanza del contesto di database.</span><span class="sxs-lookup"><span data-stu-id="276dd-165">Call the `seedData.Initialize` method, passing to it the database context instance.</span></span>
* <span data-ttu-id="276dd-166">Eliminare il contesto dopo che il metodo di inizializzazione è stato completato.</span><span class="sxs-lookup"><span data-stu-id="276dd-166">Dispose the context when the seed method completes.</span></span> <span data-ttu-id="276dd-167">L' [istruzione using](/dotnet/csharp/language-reference/keywords/using-statement) garantisce che il contesto venga eliminato.</span><span class="sxs-lookup"><span data-stu-id="276dd-167">The [using statement](/dotnet/csharp/language-reference/keywords/using-statement) ensures the context is disposed.</span></span>

<span data-ttu-id="276dd-168">Quando `Update-Database` non è stato eseguito, si verifica l'eccezione seguente:</span><span class="sxs-lookup"><span data-stu-id="276dd-168">The following exception occurs when `Update-Database` has not been run:</span></span>

> `SqlException: Cannot open database "RazorPagesMovieContext-" requested by the login. The login failed.`
> `Login failed for user 'user name'.`

### <a name="test-the-app"></a><span data-ttu-id="276dd-169">Testare l'app</span><span class="sxs-lookup"><span data-stu-id="276dd-169">Test the app</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="276dd-170">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="276dd-170">Visual Studio</span></span>](#tab/visual-studio)

1. <span data-ttu-id="276dd-171">Eliminare tutti i record nel database.</span><span class="sxs-lookup"><span data-stu-id="276dd-171">Delete all the records in the database.</span></span> <span data-ttu-id="276dd-172">Usare i collegamenti di eliminazione nel browser o da [SSOX](xref:tutorials/razor-pages/new-field#ssox)</span><span class="sxs-lookup"><span data-stu-id="276dd-172">Use the delete links in the browser or from [SSOX](xref:tutorials/razor-pages/new-field#ssox)</span></span>

1. <span data-ttu-id="276dd-173">Forzare l'inizializzazione dell'app chiamando i metodi nella `Startup` classe, in modo che venga eseguito il metodo di inizializzazione.</span><span class="sxs-lookup"><span data-stu-id="276dd-173">Force the app to initialize by calling the methods in the `Startup` class, so the seed method runs.</span></span> <span data-ttu-id="276dd-174">Per forzare l'inizializzazione, IIS Express deve essere arrestato e riavviato.</span><span class="sxs-lookup"><span data-stu-id="276dd-174">To force initialization, IIS Express must be stopped and restarted.</span></span> <span data-ttu-id="276dd-175">Arrestare e riavviare IIS con uno degli approcci seguenti:</span><span class="sxs-lookup"><span data-stu-id="276dd-175">Stop and restart IIS with any of the following approaches:</span></span>

   1. <span data-ttu-id="276dd-176">Fare clic con il pulsante destro del mouse sull'icona della barra di sistema IIS Express nell'area di notifica e selezionare **Esci** o **Arresta sito**:</span><span class="sxs-lookup"><span data-stu-id="276dd-176">Right-click the IIS Express system tray icon in the notification area and select **Exit** or **Stop Site**:</span></span>

      ![Icona dell'area di notifica di IIS Express](../first-mvc-app/working-with-sql/_static/iisExIcon.png)

      ![Menu di scelta rapida](sql/_static/stopIIS.png)

   1. <span data-ttu-id="276dd-179">Se l'app è in esecuzione in modalità non di debug, premere <kbd>F5</kbd> per l'esecuzione in modalità di debug.</span><span class="sxs-lookup"><span data-stu-id="276dd-179">If the app is running in non-debug mode, press <kbd>F5</kbd> to run in debug mode.</span></span>
   1. <span data-ttu-id="276dd-180">Se l'app è in modalità di debug, arrestare il debugger e premere <kbd>F5</kbd>.</span><span class="sxs-lookup"><span data-stu-id="276dd-180">If the app in debug mode, stop the debugger and press <kbd>F5</kbd>.</span></span>

# <a name="visual-studio-code--visual-studio-for-mac"></a>[<span data-ttu-id="276dd-181">Visual Studio Code / Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="276dd-181">Visual Studio Code / Visual Studio for Mac</span></span>](#tab/visual-studio-code+visual-studio-mac)

<span data-ttu-id="276dd-182">Eliminare tutti i record nel database, in modo che venga eseguito il metodo Seed.</span><span class="sxs-lookup"><span data-stu-id="276dd-182">Delete all the records in the database, so the seed method will run.</span></span> <span data-ttu-id="276dd-183">Arrestare e avviare l'app per inizializzare il database.</span><span class="sxs-lookup"><span data-stu-id="276dd-183">Stop and start the app to seed the database.</span></span>

---

<span data-ttu-id="276dd-184">L'app visualizza i dati sottoposti a seed:</span><span class="sxs-lookup"><span data-stu-id="276dd-184">The app shows the seeded data:</span></span>

![Applicazione film aperta nel browser che mostra i dati dei film](sql/_static/5/m55.png)

## <a name="additional-resources"></a><span data-ttu-id="276dd-186">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="276dd-186">Additional resources</span></span>

> [!div class="step-by-step"]
> <span data-ttu-id="276dd-187">[Precedente: con ponteggi Razor Pagine](xref:tutorials/razor-pages/page) 
>  [successive: aggiornare le pagine](xref:tutorials/razor-pages/da1)</span><span class="sxs-lookup"><span data-stu-id="276dd-187">[Previous: Scaffolded Razor Pages](xref:tutorials/razor-pages/page)
[Next: Update the pages](xref:tutorials/razor-pages/da1)</span></span>

::: moniker-end

::: moniker range="< aspnetcore-5.0 >= aspnetcore-3.0"

<span data-ttu-id="276dd-188">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie30) ([procedura per il download](xref:index#how-to-download-a-sample)).</span><span class="sxs-lookup"><span data-stu-id="276dd-188">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie30) ([how to download](xref:index#how-to-download-a-sample)).</span></span>

<span data-ttu-id="276dd-189">L'oggetto `RazorPagesMovieContext` gestisce l'attività di connessione al database e di mapping degli oggetti `Movie` ai record di database.</span><span class="sxs-lookup"><span data-stu-id="276dd-189">The `RazorPagesMovieContext` object handles the task of connecting to the database and mapping `Movie` objects to database records.</span></span> <span data-ttu-id="276dd-190">Il contesto del database viene registrato con il contenitore di [inserimento delle dipendenze](xref:fundamentals/dependency-injection) nel metodo `ConfigureServices` di *Startup.cs*:</span><span class="sxs-lookup"><span data-stu-id="276dd-190">The database context is registered with the [Dependency Injection](xref:fundamentals/dependency-injection) container in the `ConfigureServices` method in *Startup.cs*:</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="276dd-191">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="276dd-191">Visual Studio</span></span>](#tab/visual-studio)

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie30/Startup.cs?name=snippet_ConfigureServices&highlight=5-6)]

# <a name="visual-studio-code--visual-studio-for-mac"></a>[<span data-ttu-id="276dd-192">Visual Studio Code / Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="276dd-192">Visual Studio Code / Visual Studio for Mac</span></span>](#tab/visual-studio-code+visual-studio-mac)

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie30/Startup.cs?name=snippet_UseSqlite&highlight=5-6)]

---

<span data-ttu-id="276dd-193">La chiave viene letta dal sistema di [configurazione](xref:fundamentals/configuration/index) ASP.NET Core `ConnectionString` .</span><span class="sxs-lookup"><span data-stu-id="276dd-193">The ASP.NET Core [Configuration](xref:fundamentals/configuration/index) system reads the `ConnectionString` key.</span></span> <span data-ttu-id="276dd-194">Per lo sviluppo locale, la configurazione ottiene la stringa di connessione dal *appsettings.json* file.</span><span class="sxs-lookup"><span data-stu-id="276dd-194">For local development, configuration gets the connection string from the *appsettings.json* file.</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="276dd-195">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="276dd-195">Visual Studio</span></span>](#tab/visual-studio)

<span data-ttu-id="276dd-196">La stringa di connessione generata sarà simile alla seguente:</span><span class="sxs-lookup"><span data-stu-id="276dd-196">The generated connection string will be similar to the following:</span></span>

[!code-json[](razor-pages-start/sample/RazorPagesMovie30/appsettings.json?highlight=10-12)]

# <a name="visual-studio-code--visual-studio-for-mac"></a>[<span data-ttu-id="276dd-197">Visual Studio Code / Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="276dd-197">Visual Studio Code / Visual Studio for Mac</span></span>](#tab/visual-studio-code+visual-studio-mac)

[!code-json[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie30/appsettings_SQLite.json?highlight=10-12)]

---

<span data-ttu-id="276dd-198">Quando l'app viene distribuita in un server di test o di produzione, è possibile usare una variabile di ambiente per impostare la stringa di connessione su un server di database di test o di produzione.</span><span class="sxs-lookup"><span data-stu-id="276dd-198">When the app is deployed to a test or production server, an environment variable can be used to set the connection string to a test or production database server.</span></span> <span data-ttu-id="276dd-199">Per altre informazioni, vedere [Configurazione](xref:fundamentals/configuration/index).</span><span class="sxs-lookup"><span data-stu-id="276dd-199">See [Configuration](xref:fundamentals/configuration/index) for more information.</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="276dd-200">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="276dd-200">Visual Studio</span></span>](#tab/visual-studio)

## <a name="sql-server-express-localdb"></a><span data-ttu-id="276dd-201">SQL Server Express LocalDB</span><span class="sxs-lookup"><span data-stu-id="276dd-201">SQL Server Express LocalDB</span></span>

<span data-ttu-id="276dd-202">Local DB è una versione leggera del motore di database di SQL Server Express appositamente pensato per lo sviluppo di programmi.</span><span class="sxs-lookup"><span data-stu-id="276dd-202">LocalDB is a lightweight version of the SQL Server Express database engine that's targeted for program development.</span></span> <span data-ttu-id="276dd-203">Local DB viene avviato su richiesta ed eseguito in modalità utente; non richiede quindi una configurazione complessa.</span><span class="sxs-lookup"><span data-stu-id="276dd-203">LocalDB starts on demand and runs in user mode, so there's no complex configuration.</span></span> <span data-ttu-id="276dd-204">Per impostazione predefinita, Local DB crea file con estensione `*.mdf` nella directory `C:\Users\<user>\`.</span><span class="sxs-lookup"><span data-stu-id="276dd-204">By default, LocalDB database creates `*.mdf` files in the `C:\Users\<user>\` directory.</span></span>

<a name="ssox"></a>
* <span data-ttu-id="276dd-205">Dal menu **Visualizzazione** aprire **Esplora oggetti di SQL Server** (SSOX).</span><span class="sxs-lookup"><span data-stu-id="276dd-205">From the **View** menu, open **SQL Server Object Explorer** (SSOX).</span></span>

  ![Menu Visualizza](sql/_static/ssox.png)

* <span data-ttu-id="276dd-207">Fare clic con il pulsante destro del mouse sulla `Movie` tabella e scegliere **Visualizza finestra di progettazione**:</span><span class="sxs-lookup"><span data-stu-id="276dd-207">Right-click on the `Movie` table and select **View Designer**:</span></span>

  ![Menu di scelta rapida aperti per la tabella Movie](sql/_static/design.png)

  ![Tabelle Movie aperte nella finestra di progettazione](sql/_static/dv.png)

<span data-ttu-id="276dd-210">Si noti l'icona a forma di chiave accanto a `ID`.</span><span class="sxs-lookup"><span data-stu-id="276dd-210">Note the key icon next to `ID`.</span></span> <span data-ttu-id="276dd-211">Per impostazione predefinita, Entity Framework crea una proprietà denominata `ID` per la chiave primaria.</span><span class="sxs-lookup"><span data-stu-id="276dd-211">By default, EF creates a property named `ID` for the primary key.</span></span>

* <span data-ttu-id="276dd-212">Fare clic con il pulsante destro del mouse sulla `Movie` tabella e selezionare **Visualizza dati**:</span><span class="sxs-lookup"><span data-stu-id="276dd-212">Right-click on the `Movie` table and select **View Data**:</span></span>

  ![Tabella Movie aperta con i dati della tabella](sql/_static/vd22.png)

# <a name="visual-studio-code--visual-studio-for-mac"></a>[<span data-ttu-id="276dd-214">Visual Studio Code / Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="276dd-214">Visual Studio Code / Visual Studio for Mac</span></span>](#tab/visual-studio-code+visual-studio-mac)

## <a name="sqlite"></a><span data-ttu-id="276dd-215">SQLite</span><span class="sxs-lookup"><span data-stu-id="276dd-215">SQLite</span></span>

<span data-ttu-id="276dd-216">Nel sito Web di [SQLite](https://www.sqlite.org/) si dichiara:</span><span class="sxs-lookup"><span data-stu-id="276dd-216">The [SQLite](https://www.sqlite.org/) website states:</span></span>

> <span data-ttu-id="276dd-217">SQLite è un motore di database SQL autonomo, a elevata affidabilità, incorporato, completo e di dominio pubblico.</span><span class="sxs-lookup"><span data-stu-id="276dd-217">SQLite is a self-contained, high-reliability, embedded, full-featured, public-domain, SQL database engine.</span></span> <span data-ttu-id="276dd-218">SQLite è il motore di database più usato in tutto il mondo.</span><span class="sxs-lookup"><span data-stu-id="276dd-218">SQLite is the most used database engine in the world.</span></span>

<span data-ttu-id="276dd-219">Sono disponibili molti strumenti di terze parti che è possibile scaricare per gestire e visualizzare un database SQLite.</span><span class="sxs-lookup"><span data-stu-id="276dd-219">There are many third-party tools you can download to manage and view a SQLite database.</span></span> <span data-ttu-id="276dd-220">L'immagine seguente è stata tratta da [DB Browser per SQLite](https://sqlitebrowser.org/).</span><span class="sxs-lookup"><span data-stu-id="276dd-220">The image below is from [DB Browser for SQLite](https://sqlitebrowser.org/).</span></span> <span data-ttu-id="276dd-221">Se si preferisce uno strumento SQLite in particolare, aggiungere un commento sui motivi della preferenza.</span><span class="sxs-lookup"><span data-stu-id="276dd-221">If you have a favorite SQLite tool, leave a comment on what you like about it.</span></span>

![Browser DB per SQLite con database di film](~/tutorials/first-mvc-app-xplat/working-with-sql/_static/dbb.png)

> [!NOTE]
> <span data-ttu-id="276dd-223">Per questa esercitazione, viene usata la funzionalità *migrazioni* di Entity Framework Core, laddove possibile.</span><span class="sxs-lookup"><span data-stu-id="276dd-223">For this tutorial, the Entity Framework Core *migrations* feature is used where possible.</span></span> <span data-ttu-id="276dd-224">Le migrazioni aggiornano lo schema del database in base alle modifiche nel modello di dati.</span><span class="sxs-lookup"><span data-stu-id="276dd-224">Migrations updates the database schema to match changes in the data model.</span></span> <span data-ttu-id="276dd-225">Tuttavia, le migrazioni possono eseguire solo i tipi di modifiche supportate dal provider EF Core e le funzionalità del provider SQLite sono limitate.</span><span class="sxs-lookup"><span data-stu-id="276dd-225">However, migrations can only do the kinds of changes that the EF Core provider supports, and the SQLite provider's capabilities are limited.</span></span> <span data-ttu-id="276dd-226">Ad esempio l'aggiunta di una colonna è supportata, ma la rimozione o la modifica di una colonna non è supportata.</span><span class="sxs-lookup"><span data-stu-id="276dd-226">For example, adding a column is supported, but removing or changing a column is not supported.</span></span> <span data-ttu-id="276dd-227">Se si crea una migrazione per rimuovere o modificare una colonna, il comando `ef migrations add` ha esito positivo, ma il comando `ef database update` ha esito negativo.</span><span class="sxs-lookup"><span data-stu-id="276dd-227">If a migration is created to remove or change a column, the `ef migrations add` command succeeds but the `ef database update` command fails.</span></span> <span data-ttu-id="276dd-228">A causa di queste limitazioni, in questa esercitazione non vengono usate le migrazioni per le modifiche dello schema di SQLite.</span><span class="sxs-lookup"><span data-stu-id="276dd-228">Due to these limitations, this tutorial doesn't use migrations for SQLite schema changes.</span></span> <span data-ttu-id="276dd-229">Al contrario, quando lo schema viene modificato, il database viene eliminato e ricreato.</span><span class="sxs-lookup"><span data-stu-id="276dd-229">Instead, when the schema changes, the database is dropped and re-created.</span></span>
>
><span data-ttu-id="276dd-230">La soluzione per ovviare alle limitazioni di SQLite consiste nello scrivere manualmente il codice delle migrazioni per eseguire una ricompilazione della tabella in caso di modifiche nella tabella.</span><span class="sxs-lookup"><span data-stu-id="276dd-230">The workaround for the SQLite limitations is to manually write migrations code to perform a table rebuild when something in the table changes.</span></span> <span data-ttu-id="276dd-231">La ricompilazione della tabella comporta:</span><span class="sxs-lookup"><span data-stu-id="276dd-231">A table rebuild involves:</span></span>
>
>* <span data-ttu-id="276dd-232">Creazione di una nuova tabella.</span><span class="sxs-lookup"><span data-stu-id="276dd-232">Creating a new table.</span></span>
>* <span data-ttu-id="276dd-233">Copia dei dati dalla vecchia tabella alla nuova tabella.</span><span class="sxs-lookup"><span data-stu-id="276dd-233">Copying data from the old table to the new table.</span></span>
>* <span data-ttu-id="276dd-234">Eliminazione della tabella precedente.</span><span class="sxs-lookup"><span data-stu-id="276dd-234">Dropping the old table.</span></span>
>* <span data-ttu-id="276dd-235">Ridenominazione della nuova tabella.</span><span class="sxs-lookup"><span data-stu-id="276dd-235">Renaming the new table.</span></span>
>
><span data-ttu-id="276dd-236">Per altre informazioni, vedere le risorse seguenti:</span><span class="sxs-lookup"><span data-stu-id="276dd-236">For more information, see the following resources:</span></span>
> * [<span data-ttu-id="276dd-237">Limitazioni del provider di database SQLite per EF Core</span><span class="sxs-lookup"><span data-stu-id="276dd-237">SQLite EF Core Database Provider Limitations</span></span>](/ef/core/providers/sqlite/limitations)
> * [<span data-ttu-id="276dd-238">Personalizzare il codice di migrazione</span><span class="sxs-lookup"><span data-stu-id="276dd-238">Customize migration code</span></span>](/ef/core/managing-schemas/migrations/#customize-migration-code)
> * [<span data-ttu-id="276dd-239">Seeding dei dati</span><span class="sxs-lookup"><span data-stu-id="276dd-239">Data seeding</span></span>](/ef/core/modeling/data-seeding)
> * [<span data-ttu-id="276dd-240">Istruzione ALTER TABLE di SQLite</span><span class="sxs-lookup"><span data-stu-id="276dd-240">SQLite ALTER TABLE statement</span></span>](https://sqlite.org/lang_altertable.html)

---

## <a name="seed-the-database"></a><span data-ttu-id="276dd-241">Specificare il valore di inizializzazione del database</span><span class="sxs-lookup"><span data-stu-id="276dd-241">Seed the database</span></span>

<span data-ttu-id="276dd-242">Creare una nuova classe denominata `SeedData` nella cartella *Models* usando il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="276dd-242">Create a new class named `SeedData` in the *Models* folder with the following code:</span></span>

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie30/Models/SeedData.cs?name=snippet_1)]

<span data-ttu-id="276dd-243">Se sono presenti film nel database, l'inizializzatore del valore di inizializzazione viene restituito e non viene aggiunto alcun film.</span><span class="sxs-lookup"><span data-stu-id="276dd-243">If there are any movies in the database, the seed initializer returns and no movies are added.</span></span>

```csharp
if (context.Movie.Any())
{
    return;
}
```

<a name="si"></a>

### <a name="add-the-seed-initializer"></a><span data-ttu-id="276dd-244">Aggiungere l'inizializzatore del valore di inizializzazione</span><span class="sxs-lookup"><span data-stu-id="276dd-244">Add the seed initializer</span></span>

<span data-ttu-id="276dd-245">Sostituire il contenuto di *Program.cs* con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="276dd-245">Replace the contents of the *Program.cs* with the following code:</span></span>

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie30/Program.cs)]

<span data-ttu-id="276dd-246">Nel codice precedente, il `Main` metodo è stato modificato per eseguire le operazioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="276dd-246">In the previous code, the `Main` method has been modified to do the following:</span></span>

* <span data-ttu-id="276dd-247">Ottenere un'istanza del contesto di database dal contenitore di inserimento delle dipendenze.</span><span class="sxs-lookup"><span data-stu-id="276dd-247">Get a database context instance from the dependency injection container.</span></span>
* <span data-ttu-id="276dd-248">Chiamare il `seedData.Initialize` metodo passando all'istanza del contesto di database.</span><span class="sxs-lookup"><span data-stu-id="276dd-248">Call the `seedData.Initialize` method, passing to it the database context instance.</span></span>
* <span data-ttu-id="276dd-249">Eliminare il contesto dopo che il metodo di inizializzazione è stato completato.</span><span class="sxs-lookup"><span data-stu-id="276dd-249">Dispose the context when the seed method completes.</span></span> <span data-ttu-id="276dd-250">L' [istruzione using](/dotnet/csharp/language-reference/keywords/using-statement) garantisce che il contesto venga eliminato.</span><span class="sxs-lookup"><span data-stu-id="276dd-250">The [using statement](/dotnet/csharp/language-reference/keywords/using-statement) ensures the context is disposed.</span></span>

<span data-ttu-id="276dd-251">Quando `Update-Database` non è stato eseguito, si verifica l'eccezione seguente:</span><span class="sxs-lookup"><span data-stu-id="276dd-251">The following exception occurs when `Update-Database` has not been run:</span></span>

> `SqlException: Cannot open database "RazorPagesMovieContext-" requested by the login. The login failed.`
> `Login failed for user 'user name'.`

### <a name="test-the-app"></a><span data-ttu-id="276dd-252">Testare l'app</span><span class="sxs-lookup"><span data-stu-id="276dd-252">Test the app</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="276dd-253">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="276dd-253">Visual Studio</span></span>](#tab/visual-studio)

* <span data-ttu-id="276dd-254">Eliminare tutti i record nel database.</span><span class="sxs-lookup"><span data-stu-id="276dd-254">Delete all the records in the database.</span></span> <span data-ttu-id="276dd-255">Usare i collegamenti di eliminazione nel browser o da [SSOX](xref:tutorials/razor-pages/new-field#ssox).</span><span class="sxs-lookup"><span data-stu-id="276dd-255">Use the delete links in the browser or from [SSOX](xref:tutorials/razor-pages/new-field#ssox).</span></span>
* <span data-ttu-id="276dd-256">Forzare l'inizializzazione dell'app chiamando i metodi nella `Startup` classe, in modo che venga eseguito il metodo di inizializzazione.</span><span class="sxs-lookup"><span data-stu-id="276dd-256">Force the app to initialize by calling the methods in the `Startup` class, so the seed method runs.</span></span> <span data-ttu-id="276dd-257">Per forzare l'inizializzazione, IIS Express deve essere arrestato e riavviato.</span><span class="sxs-lookup"><span data-stu-id="276dd-257">To force initialization, IIS Express must be stopped and restarted.</span></span> <span data-ttu-id="276dd-258">Arrestare e riavviare IIS con uno degli approcci seguenti:</span><span class="sxs-lookup"><span data-stu-id="276dd-258">Stop and restart IIS with any of the following approaches:</span></span>

  * <span data-ttu-id="276dd-259">Fare clic con il pulsante destro del mouse sull'icona dell'area di notifica di IIS Express e toccare **Esci** o **Arresta sito**:</span><span class="sxs-lookup"><span data-stu-id="276dd-259">Right-click the IIS Express system tray icon in the notification area and tap **Exit** or **Stop Site**:</span></span>

    ![Icona dell'area di notifica di IIS Express](../first-mvc-app/working-with-sql/_static/iisExIcon.png)

    ![Menu di scelta rapida](sql/_static/stopIIS.png)

    * <span data-ttu-id="276dd-262">Se l'app è in esecuzione in modalità non di debug, premere <kbd>F5</kbd> per l'esecuzione in modalità di debug.</span><span class="sxs-lookup"><span data-stu-id="276dd-262">If the app is running in non-debug mode, press <kbd>F5</kbd> to run in debug mode.</span></span>
    * <span data-ttu-id="276dd-263">Se l'app è in modalità di debug, arrestare il debugger e premere <kbd>F5</kbd>.</span><span class="sxs-lookup"><span data-stu-id="276dd-263">If the app in debug mode, stop the debugger and press <kbd>F5</kbd>.</span></span>

# <a name="visual-studio-code--visual-studio-for-mac"></a>[<span data-ttu-id="276dd-264">Visual Studio Code / Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="276dd-264">Visual Studio Code / Visual Studio for Mac</span></span>](#tab/visual-studio-code+visual-studio-mac)

<span data-ttu-id="276dd-265">Eliminare tutti i record nel database, in modo che venga eseguito il metodo Seed.</span><span class="sxs-lookup"><span data-stu-id="276dd-265">Delete all the records in the database, so the seed method will run.</span></span> <span data-ttu-id="276dd-266">Arrestare e avviare l'app per inizializzare il database.</span><span class="sxs-lookup"><span data-stu-id="276dd-266">Stop and start the app to seed the database.</span></span>

---

<span data-ttu-id="276dd-267">L'app visualizza i dati sottoposti a seed:</span><span class="sxs-lookup"><span data-stu-id="276dd-267">The app shows the seeded data:</span></span>

![App per i film aperta in Chrome con i dati sui film](sql/_static/m55https.png)

## <a name="additional-resources"></a><span data-ttu-id="276dd-269">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="276dd-269">Additional resources</span></span>

> [!div class="step-by-step"]
> <span data-ttu-id="276dd-270">[Precedente: con ponteggi Razor Pagine](xref:tutorials/razor-pages/page) 
>  [successive: aggiornare le pagine](xref:tutorials/razor-pages/da1)</span><span class="sxs-lookup"><span data-stu-id="276dd-270">[Previous: Scaffolded Razor Pages](xref:tutorials/razor-pages/page)
[Next: Update the pages](xref:tutorials/razor-pages/da1)</span></span>

::: moniker-end

::: moniker range="< aspnetcore-3.0"

<span data-ttu-id="276dd-271">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/razor-pages/razor-pages-start) ([procedura per il download](xref:index#how-to-download-a-sample)).</span><span class="sxs-lookup"><span data-stu-id="276dd-271">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/razor-pages/razor-pages-start) ([how to download](xref:index#how-to-download-a-sample)).</span></span>

<span data-ttu-id="276dd-272">L'oggetto `RazorPagesMovieContext` gestisce l'attività di connessione al database e di mapping degli oggetti `Movie` ai record di database.</span><span class="sxs-lookup"><span data-stu-id="276dd-272">The `RazorPagesMovieContext` object handles the task of connecting to the database and mapping `Movie` objects to database records.</span></span> <span data-ttu-id="276dd-273">Il contesto del database viene registrato con il contenitore di [inserimento delle dipendenze](xref:fundamentals/dependency-injection) nel metodo `ConfigureServices` di *Startup.cs*:</span><span class="sxs-lookup"><span data-stu-id="276dd-273">The database context is registered with the [Dependency Injection](xref:fundamentals/dependency-injection) container in the `ConfigureServices` method in *Startup.cs*:</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="276dd-274">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="276dd-274">Visual Studio</span></span>](#tab/visual-studio)

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie22/Startup.cs?name=snippet_ConfigureServices&highlight=15-17)]

# <a name="visual-studio-code--visual-studio-for-mac"></a>[<span data-ttu-id="276dd-275">Visual Studio Code / Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="276dd-275">Visual Studio Code / Visual Studio for Mac</span></span>](#tab/visual-studio-code+visual-studio-mac)

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie22/Startup.cs?name=snippet_UseSqlite&highlight=11-12)]

---

<span data-ttu-id="276dd-276">Per altre informazioni sui metodi usati in `ConfigureServices`, vedere:</span><span class="sxs-lookup"><span data-stu-id="276dd-276">For more information on the methods used in `ConfigureServices`, see:</span></span>

* <span data-ttu-id="276dd-277">[Supporto per il Regolamento generale sulla protezione dei dati (GDPR) dell'Unione Europea in ASP.NET Core](xref:security/gdpr) per `CookiePolicyOptions`.</span><span class="sxs-lookup"><span data-stu-id="276dd-277">[EU General Data Protection Regulation (GDPR) support in ASP.NET Core](xref:security/gdpr) for `CookiePolicyOptions`.</span></span>
* [<span data-ttu-id="276dd-278">SetCompatibilityVersion</span><span class="sxs-lookup"><span data-stu-id="276dd-278">SetCompatibilityVersion</span></span>](xref:mvc/compatibility-version)

<span data-ttu-id="276dd-279">La chiave viene letta dal sistema di [configurazione](xref:fundamentals/configuration/index) ASP.NET Core `ConnectionString` .</span><span class="sxs-lookup"><span data-stu-id="276dd-279">The ASP.NET Core [Configuration](xref:fundamentals/configuration/index) system reads the `ConnectionString` key.</span></span> <span data-ttu-id="276dd-280">Per lo sviluppo locale, la configurazione ottiene la stringa di connessione dal *appsettings.json* file.</span><span class="sxs-lookup"><span data-stu-id="276dd-280">For local development, configuration gets the connection string from the *appsettings.json* file.</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="276dd-281">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="276dd-281">Visual Studio</span></span>](#tab/visual-studio)

<span data-ttu-id="276dd-282">La stringa di connessione generata sarà simile alla seguente:</span><span class="sxs-lookup"><span data-stu-id="276dd-282">The generated connection string will be similar to the following:</span></span>

[!code-json[](razor-pages-start/sample/RazorPagesMovie22/appsettings.json?highlight=8-10)]

# <a name="visual-studio-code"></a>[<span data-ttu-id="276dd-283">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="276dd-283">Visual Studio Code</span></span>](#tab/visual-studio-code)

[!code-json[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie22/appsettings_SQLite.json?highlight=8-10)]

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="276dd-284">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="276dd-284">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

[!code-json[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie22/appsettings_SQLite.json?highlight=8-10)]

---

<span data-ttu-id="276dd-285">Quando l'app viene distribuita in un server di test o di produzione, è possibile usare una variabile di ambiente per impostare la stringa di connessione su un server di database di test o di produzione.</span><span class="sxs-lookup"><span data-stu-id="276dd-285">When the app is deployed to a test or production server, an environment variable can be used to set the connection string to a test or production database server.</span></span> <span data-ttu-id="276dd-286">Per altre informazioni, vedere [Configurazione](xref:fundamentals/configuration/index).</span><span class="sxs-lookup"><span data-stu-id="276dd-286">See [Configuration](xref:fundamentals/configuration/index) for more information.</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="276dd-287">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="276dd-287">Visual Studio</span></span>](#tab/visual-studio)

## <a name="sql-server-express-localdb"></a><span data-ttu-id="276dd-288">SQL Server Express LocalDB</span><span class="sxs-lookup"><span data-stu-id="276dd-288">SQL Server Express LocalDB</span></span>

<span data-ttu-id="276dd-289">Local DB è una versione leggera del motore di database di SQL Server Express appositamente pensato per lo sviluppo di programmi.</span><span class="sxs-lookup"><span data-stu-id="276dd-289">LocalDB is a lightweight version of the SQL Server Express database engine that's targeted for program development.</span></span> <span data-ttu-id="276dd-290">Local DB viene avviato su richiesta ed eseguito in modalità utente; non richiede quindi una configurazione complessa.</span><span class="sxs-lookup"><span data-stu-id="276dd-290">LocalDB starts on demand and runs in user mode, so there's no complex configuration.</span></span> <span data-ttu-id="276dd-291">Per impostazione predefinita, Local DB crea file con estensione `*.mdf` nella directory `C:/Users/<user/>`.</span><span class="sxs-lookup"><span data-stu-id="276dd-291">By default, LocalDB database creates `*.mdf` files in the `C:/Users/<user/>` directory.</span></span>

<a name="ssox"></a>
* <span data-ttu-id="276dd-292">Dal menu **Visualizzazione** aprire **Esplora oggetti di SQL Server** (SSOX).</span><span class="sxs-lookup"><span data-stu-id="276dd-292">From the **View** menu, open **SQL Server Object Explorer** (SSOX).</span></span>

  ![Menu Visualizza](sql/_static/ssox.png)

* <span data-ttu-id="276dd-294">Fare clic con il pulsante destro del mouse sulla `Movie` tabella e scegliere **Visualizza finestra di progettazione**:</span><span class="sxs-lookup"><span data-stu-id="276dd-294">Right-click on the `Movie` table and select **View Designer**:</span></span>

  ![Menu di scelta rapida aperto per la tabella Movie](sql/_static/design.png)

  ![Tabella Movie aperta nella finestra di progettazione](sql/_static/dv.png)

<span data-ttu-id="276dd-297">Si noti l'icona a forma di chiave accanto a `ID`.</span><span class="sxs-lookup"><span data-stu-id="276dd-297">Note the key icon next to `ID`.</span></span> <span data-ttu-id="276dd-298">Per impostazione predefinita, Entity Framework crea una proprietà denominata `ID` per la chiave primaria.</span><span class="sxs-lookup"><span data-stu-id="276dd-298">By default, EF creates a property named `ID` for the primary key.</span></span>

* <span data-ttu-id="276dd-299">Fare clic con il pulsante destro del mouse sulla `Movie` tabella e selezionare **Visualizza dati**:</span><span class="sxs-lookup"><span data-stu-id="276dd-299">Right-click on the `Movie` table and select **View Data**:</span></span>

  ![Tabella Movie aperta con i dati della tabella](sql/_static/vd22.png)

# <a name="visual-studio-code"></a>[<span data-ttu-id="276dd-301">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="276dd-301">Visual Studio Code</span></span>](#tab/visual-studio-code)

[!INCLUDE[](~/includes/rp/sqlite.md)]
[!INCLUDE[](~/includes/RP-mvc-shared/sqlite-warn.md)]

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="276dd-302">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="276dd-302">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

[!INCLUDE[](~/includes/rp/sqlite.md)]
[!INCLUDE[](~/includes/RP-mvc-shared/sqlite-warn.md)]

---

## <a name="seed-the-database"></a><span data-ttu-id="276dd-303">Specificare il valore di inizializzazione del database</span><span class="sxs-lookup"><span data-stu-id="276dd-303">Seed the database</span></span>

<span data-ttu-id="276dd-304">Creare una nuova classe denominata `SeedData` nella cartella *Models* usando il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="276dd-304">Create a new class named `SeedData` in the *Models* folder with the following code:</span></span>

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie22/Models/SeedData.cs?name=snippet_1)]

<span data-ttu-id="276dd-305">Se sono presenti film nel database, l'inizializzatore del valore di inizializzazione viene restituito e non viene aggiunto alcun film.</span><span class="sxs-lookup"><span data-stu-id="276dd-305">If there are any movies in the database, the seed initializer returns and no movies are added.</span></span>

```csharp
if (context.Movie.Any())
{
    return;
}
```

<a name="si"></a>

### <a name="add-the-seed-initializer"></a><span data-ttu-id="276dd-306">Aggiungere l'inizializzatore del valore di inizializzazione</span><span class="sxs-lookup"><span data-stu-id="276dd-306">Add the seed initializer</span></span>

<span data-ttu-id="276dd-307">Sostituire il contenuto di *Program.cs* con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="276dd-307">Replace the contents of the *Program.cs* with the following code:</span></span>

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie22/Program.cs)]

<span data-ttu-id="276dd-308">Nel codice precedente, il `Main` metodo è stato modificato per eseguire le operazioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="276dd-308">In the previous code, the `Main` method has been modified to do the following:</span></span>

* <span data-ttu-id="276dd-309">Ottenere un'istanza del contesto di database dal contenitore di inserimento delle dipendenze.</span><span class="sxs-lookup"><span data-stu-id="276dd-309">Get a database context instance from the dependency injection container.</span></span>
* <span data-ttu-id="276dd-310">Chiamare il `seedData.Initialize` metodo passando all'istanza del contesto di database.</span><span class="sxs-lookup"><span data-stu-id="276dd-310">Call the `seedData.Initialize` method, passing to it the database context instance.</span></span>
* <span data-ttu-id="276dd-311">Eliminare il contesto dopo che il metodo di inizializzazione è stato completato.</span><span class="sxs-lookup"><span data-stu-id="276dd-311">Dispose the context when the seed method completes.</span></span> <span data-ttu-id="276dd-312">L' [istruzione using](/dotnet/csharp/language-reference/keywords/using-statement) garantisce che il contesto venga eliminato.</span><span class="sxs-lookup"><span data-stu-id="276dd-312">The [using statement](/dotnet/csharp/language-reference/keywords/using-statement) ensures the context is disposed.</span></span>

<span data-ttu-id="276dd-313">Un'app di produzione non chiamerà `Database.Migrate`.</span><span class="sxs-lookup"><span data-stu-id="276dd-313">A production app would not call `Database.Migrate`.</span></span> <span data-ttu-id="276dd-314">Sarà aggiunto al codice precedente per evitare l'eccezione seguente quando `Update-Database` non è stato eseguito:</span><span class="sxs-lookup"><span data-stu-id="276dd-314">It's added to the preceding code to prevent the following exception when `Update-Database` has not been run:</span></span>

<span data-ttu-id="276dd-315">SqlException: Impossibile aprire il database " Razor PagesMovieContext-21" richiesto dall'account di accesso.</span><span class="sxs-lookup"><span data-stu-id="276dd-315">SqlException: Cannot open database "RazorPagesMovieContext-21" requested by the login.</span></span> <span data-ttu-id="276dd-316">Accesso non riuscito.</span><span class="sxs-lookup"><span data-stu-id="276dd-316">The login failed.</span></span>
<span data-ttu-id="276dd-317">Accesso non riuscito per l'utente "nome-utente".</span><span class="sxs-lookup"><span data-stu-id="276dd-317">Login failed for user 'user name'.</span></span>

### <a name="test-the-app"></a><span data-ttu-id="276dd-318">Testare l'app</span><span class="sxs-lookup"><span data-stu-id="276dd-318">Test the app</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="276dd-319">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="276dd-319">Visual Studio</span></span>](#tab/visual-studio)

* <span data-ttu-id="276dd-320">Eliminare tutti i record nel database.</span><span class="sxs-lookup"><span data-stu-id="276dd-320">Delete all the records in the database.</span></span> <span data-ttu-id="276dd-321">Questa operazione può essere eseguita con i collegamenti di eliminazione nel browser o da [SSOX](xref:tutorials/razor-pages/new-field#ssox)</span><span class="sxs-lookup"><span data-stu-id="276dd-321">You can do this with the delete links in the browser or from [SSOX](xref:tutorials/razor-pages/new-field#ssox)</span></span>
* <span data-ttu-id="276dd-322">Forzare l'inizializzazione dell'app chiamando i metodi nella `Startup` classe, in modo che venga eseguito il metodo di inizializzazione.</span><span class="sxs-lookup"><span data-stu-id="276dd-322">Force the app to initialize by calling the methods in the `Startup` class, so the seed method runs.</span></span> <span data-ttu-id="276dd-323">Per forzare l'inizializzazione, IIS Express deve essere arrestato e riavviato.</span><span class="sxs-lookup"><span data-stu-id="276dd-323">To force initialization, IIS Express must be stopped and restarted.</span></span> <span data-ttu-id="276dd-324">È possibile eseguire questa operazione adottando uno degli approcci seguenti:</span><span class="sxs-lookup"><span data-stu-id="276dd-324">You can do this with any of the following approaches:</span></span>

  * <span data-ttu-id="276dd-325">Fare clic con il pulsante destro del mouse sull'icona dell'area di notifica di IIS Express e toccare **Esci** o **Arresta sito**:</span><span class="sxs-lookup"><span data-stu-id="276dd-325">Right-click the IIS Express system tray icon in the notification area and tap **Exit** or **Stop Site**:</span></span>

    ![Icona dell'area di notifica di IIS Express](../first-mvc-app/working-with-sql/_static/iisExIcon.png)

    ![Menu di scelta rapida](sql/_static/stopIIS.png)

    * <span data-ttu-id="276dd-328">Se l'app è in esecuzione in modalità non di debug, premere <kbd>F5</kbd> per l'esecuzione in modalità di debug.</span><span class="sxs-lookup"><span data-stu-id="276dd-328">If the app is running in non-debug mode, press <kbd>F5</kbd> to run in debug mode.</span></span>
    * <span data-ttu-id="276dd-329">Se l'app è in modalità di debug, arrestare il debugger e premere <kbd>F5</kbd>.</span><span class="sxs-lookup"><span data-stu-id="276dd-329">If the app in debug mode, stop the debugger and press <kbd>F5</kbd>.</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="276dd-330">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="276dd-330">Visual Studio Code</span></span>](#tab/visual-studio-code)

<span data-ttu-id="276dd-331">Eliminare tutti i record nel database, in modo che venga eseguito il metodo Seed.</span><span class="sxs-lookup"><span data-stu-id="276dd-331">Delete all the records in the database, so the seed method will run.</span></span> <span data-ttu-id="276dd-332">Arrestare e avviare l'app per inizializzare il database.</span><span class="sxs-lookup"><span data-stu-id="276dd-332">Stop and start the app to seed the database.</span></span>

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="276dd-333">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="276dd-333">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

<span data-ttu-id="276dd-334">Eliminare tutti i record nel database, in modo che venga eseguito il metodo Seed.</span><span class="sxs-lookup"><span data-stu-id="276dd-334">Delete all the records in the database, so the seed method will run.</span></span> <span data-ttu-id="276dd-335">Arrestare e avviare l'app per inizializzare il database.</span><span class="sxs-lookup"><span data-stu-id="276dd-335">Stop and start the app to seed the database.</span></span>

---

<span data-ttu-id="276dd-336">L'app visualizza i dati sottoposti a seed:</span><span class="sxs-lookup"><span data-stu-id="276dd-336">The app shows the seeded data:</span></span>

![App per i film aperta in Chrome con i dati sui film](sql/_static/m55https.png)

<span data-ttu-id="276dd-338">L'esercitazione successiva consentirà di pulire la presentazione dei dati.</span><span class="sxs-lookup"><span data-stu-id="276dd-338">The next tutorial will clean up the presentation of the data.</span></span>

## <a name="additional-resources"></a><span data-ttu-id="276dd-339">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="276dd-339">Additional resources</span></span>

* [<span data-ttu-id="276dd-340">Versione YouTube dell'esercitazione</span><span class="sxs-lookup"><span data-stu-id="276dd-340">YouTube version of this tutorial</span></span>](https://youtu.be/A_5ff11sDHY)

> [!div class="step-by-step"]
> <span data-ttu-id="276dd-341">[Precedente: con ponteggi Razor Pagine](xref:tutorials/razor-pages/page) 
>  [successive: aggiornare le pagine](xref:tutorials/razor-pages/da1)</span><span class="sxs-lookup"><span data-stu-id="276dd-341">[Previous: Scaffolded Razor Pages](xref:tutorials/razor-pages/page)
[Next: Update the pages](xref:tutorials/razor-pages/da1)</span></span>

::: moniker-end