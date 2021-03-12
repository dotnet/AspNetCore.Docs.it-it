---
title: "Esercitazione parte 5: applicare le migrazioni all'esempio di Contoso University"
description: Parte 5 della serie di esercitazioni di Contoso University. Usare la funzionalità migrazioni di EF Core per gestire le modifiche al modello di dati in un'app ASP.NET Core MVC.
author: rick-anderson
ms.author: riande
ms.custom: contperf-fy21q2
ms.date: 11/13/2020
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
uid: data/ef-mvc/migrations
ms.openlocfilehash: aebbc3f29b0356c7993abd83869ab21d3613bf61
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102589348"
---
# <a name="tutorial-part-5-apply-migrations-to-the-contoso-university-sample"></a><span data-ttu-id="27e4a-104">Esercitazione: parte 5, applicare le migrazioni all'esempio di Contoso University</span><span class="sxs-lookup"><span data-stu-id="27e4a-104">Tutorial: Part 5, apply migrations to the Contoso University sample</span></span>

<span data-ttu-id="27e4a-105">In questa esercitazione si inizia a usare la funzionalità delle migrazioni EF Core per la gestione delle modifiche al modello di dati.</span><span class="sxs-lookup"><span data-stu-id="27e4a-105">In this tutorial, you start using the EF Core migrations feature for managing data model changes.</span></span> <span data-ttu-id="27e4a-106">Nelle esercitazioni successive si aggiungeranno altre migrazioni quando si modifica il modello di dati.</span><span class="sxs-lookup"><span data-stu-id="27e4a-106">In later tutorials, you'll add more migrations as you change the data model.</span></span>

<span data-ttu-id="27e4a-107">In questa esercitazione:</span><span class="sxs-lookup"><span data-stu-id="27e4a-107">In this tutorial, you:</span></span>

> [!div class="checklist"]
> * <span data-ttu-id="27e4a-108">Ottenere informazioni sulle migrazioni</span><span class="sxs-lookup"><span data-stu-id="27e4a-108">Learn about migrations</span></span>
> * <span data-ttu-id="27e4a-109">Creare una migrazione iniziale</span><span class="sxs-lookup"><span data-stu-id="27e4a-109">Create an initial migration</span></span>
> * <span data-ttu-id="27e4a-110">Esaminare i metodi Up e Down</span><span class="sxs-lookup"><span data-stu-id="27e4a-110">Examine Up and Down methods</span></span>
> * <span data-ttu-id="27e4a-111">Esaminare lo snapshot del modello di dati</span><span class="sxs-lookup"><span data-stu-id="27e4a-111">Learn about the data model snapshot</span></span>
> * <span data-ttu-id="27e4a-112">Applicare la migrazione</span><span class="sxs-lookup"><span data-stu-id="27e4a-112">Apply the migration</span></span>

## <a name="prerequisites"></a><span data-ttu-id="27e4a-113">Prerequisiti</span><span class="sxs-lookup"><span data-stu-id="27e4a-113">Prerequisites</span></span>

* [<span data-ttu-id="27e4a-114">Ordinamento, filtro e paging</span><span class="sxs-lookup"><span data-stu-id="27e4a-114">Sorting, filtering, and paging</span></span>](sort-filter-page.md)

## <a name="about-migrations"></a><span data-ttu-id="27e4a-115">Informazioni sulle migrazioni</span><span class="sxs-lookup"><span data-stu-id="27e4a-115">About migrations</span></span>

<span data-ttu-id="27e4a-116">Quando si sviluppa una nuova applicazione, il modello di dati cambia di frequente e, a ogni cambiamento, non è più sincronizzato con il database.</span><span class="sxs-lookup"><span data-stu-id="27e4a-116">When you develop a new application, your data model changes frequently, and each time the model changes, it gets out of sync with the database.</span></span> <span data-ttu-id="27e4a-117">La serie di esercitazioni è iniziata con la configurazione di Entity Framework per creare il database se non esiste già.</span><span class="sxs-lookup"><span data-stu-id="27e4a-117">You started these tutorials by configuring the Entity Framework to create the database if it doesn't exist.</span></span> <span data-ttu-id="27e4a-118">Quindi, ogni volta che si modifica il modello di dati, ovvero si aggiungono, rimuovono, o modificano le classi di entità o si modifica la classe DbContext, è possibile eliminare il database. In seguito EF ne crea uno nuovo corrispondente al modello ed esegue il seeding con dati di test.</span><span class="sxs-lookup"><span data-stu-id="27e4a-118">Then each time you change the data model -- add, remove, or change entity classes or change your DbContext class -- you can delete the database and EF creates a new one that matches the model, and seeds it with test data.</span></span>

<span data-ttu-id="27e4a-119">Questo metodo che consiste nel mantenere il database sincronizzato con il modello di dati funziona bene fino a quando non si distribuisce l'applicazione nell'ambiente di produzione.</span><span class="sxs-lookup"><span data-stu-id="27e4a-119">This method of keeping the database in sync with the data model works well until you deploy the application to production.</span></span> <span data-ttu-id="27e4a-120">Quando l'applicazione è in esecuzione nell'ambiente di produzione in genere archivia dati che devono essere mantenuti ed è necessario evitare di perdere tutto ad ogni modifica, come ad esempio all'aggiunta di una colonna.</span><span class="sxs-lookup"><span data-stu-id="27e4a-120">When the application is running in production it's usually storing data that you want to keep, and you don't want to lose everything each time you make a change such as adding a new column.</span></span> <span data-ttu-id="27e4a-121">La funzionalità delle migrazioni di EF Core risolve questo problema consentendo a EF Core di aggiornare lo schema del database anziché crearne uno nuovo.</span><span class="sxs-lookup"><span data-stu-id="27e4a-121">The EF Core Migrations feature solves this problem by enabling EF to update the database schema instead of creating  a new database.</span></span>

<span data-ttu-id="27e4a-122">Per lavorare con le migrazioni, è possibile usare la **console di gestione pacchetti** (PMC) o l'interfaccia della riga di comando.</span><span class="sxs-lookup"><span data-stu-id="27e4a-122">To work with migrations, you can use the **Package Manager Console** (PMC) or the CLI.</span></span>  <span data-ttu-id="27e4a-123">Queste esercitazioni illustrano come usare i comandi dell'interfaccia della riga di comando.</span><span class="sxs-lookup"><span data-stu-id="27e4a-123">These tutorials show how to use CLI commands.</span></span> <span data-ttu-id="27e4a-124">Per informazioni sulla Console di Gestione pacchetti, vedere [la fine di questa esercitazione](#pmc).</span><span class="sxs-lookup"><span data-stu-id="27e4a-124">Information about the PMC is at [the end of this tutorial](#pmc).</span></span>

## <a name="drop-the-database"></a><span data-ttu-id="27e4a-125">Eliminare il database</span><span class="sxs-lookup"><span data-stu-id="27e4a-125">Drop the database</span></span>

<span data-ttu-id="27e4a-126">Installare EF Core Tools come [strumento globale](/ef/core/miscellaneous/cli/dotnet) ed eliminare il database:</span><span class="sxs-lookup"><span data-stu-id="27e4a-126">Install EF Core tools as a [global tool](/ef/core/miscellaneous/cli/dotnet) and delete the database:</span></span>

 ```dotnetcli
 dotnet tool install --global dotnet-ef
 dotnet ef database drop
 ```

<span data-ttu-id="27e4a-127">Nella sezione seguente viene illustrato come eseguire i comandi della CLI.</span><span class="sxs-lookup"><span data-stu-id="27e4a-127">The following section explains how to run CLI commands.</span></span>

## <a name="create-an-initial-migration"></a><span data-ttu-id="27e4a-128">Creare una migrazione iniziale</span><span class="sxs-lookup"><span data-stu-id="27e4a-128">Create an initial migration</span></span>

<span data-ttu-id="27e4a-129">Salvare le modifiche e compilare il progetto.</span><span class="sxs-lookup"><span data-stu-id="27e4a-129">Save your changes and build the project.</span></span> <span data-ttu-id="27e4a-130">Aprire una finestra di comando e passare alla cartella del progetto.</span><span class="sxs-lookup"><span data-stu-id="27e4a-130">Then open a command window and navigate to the project folder.</span></span> <span data-ttu-id="27e4a-131">Ecco un modo rapido per eseguire questa operazione:</span><span class="sxs-lookup"><span data-stu-id="27e4a-131">Here's a quick way to do that:</span></span>

* <span data-ttu-id="27e4a-132">In **Esplora soluzioni** fare clic con il pulsante destro del mouse sul progetto e scegliere **Apri cartella in Esplora File** dal menu di scelta rapida.</span><span class="sxs-lookup"><span data-stu-id="27e4a-132">In **Solution Explorer**, right-click the project and choose **Open Folder in File Explorer** from the context menu.</span></span>

  ![Voce di menu Apri in Esplora file](migrations/_static/open-in-file-explorer.png)

* <span data-ttu-id="27e4a-134">Immettere "cmd" nella barra degli indirizzi e premere INVIO.</span><span class="sxs-lookup"><span data-stu-id="27e4a-134">Enter "cmd" in the address bar and press Enter.</span></span>

  ![Finestra di comando](migrations/_static/open-command-window.png)

<span data-ttu-id="27e4a-136">Immettere il comando seguente nella finestra Comando:</span><span class="sxs-lookup"><span data-stu-id="27e4a-136">Enter the following command in the command window:</span></span>

```dotnetcli
dotnet ef migrations add InitialCreate
```

<span data-ttu-id="27e4a-137">Nei comandi precedenti viene visualizzato un output simile al seguente:</span><span class="sxs-lookup"><span data-stu-id="27e4a-137">In the preceding commands, output similar to the following is displayed:</span></span>

```console
info: Microsoft.EntityFrameworkCore.Infrastructure[10403]
      Entity Framework Core initialized 'SchoolContext' using provider 'Microsoft.EntityFrameworkCore.SqlServer' with options: None
Done. To undo this action, use 'ef migrations remove'
```

<span data-ttu-id="27e4a-138">Se viene visualizzato un messaggio di errore che *indica che non è possibile accedere al file... ContosoUniversity.dll perché è in uso da un altro processo.*", trovare l'icona IIS Express nella barra delle applicazioni di Windows e fare clic con il pulsante destro del mouse su di essa, quindi fare clic su **ContosoUniversity > arresta sito**.</span><span class="sxs-lookup"><span data-stu-id="27e4a-138">If you see an error message "*cannot access the file ... ContosoUniversity.dll because it is being used by another process.*", find the IIS Express icon in the Windows System Tray, and right-click it, then click **ContosoUniversity > Stop Site**.</span></span>

## <a name="examine-up-and-down-methods"></a><span data-ttu-id="27e4a-139">Esaminare i metodi Up e Down</span><span class="sxs-lookup"><span data-stu-id="27e4a-139">Examine Up and Down methods</span></span>

<span data-ttu-id="27e4a-140">Quando è stato eseguito il comando `migrations add`, EF ha generato il codice che crea il database da zero.</span><span class="sxs-lookup"><span data-stu-id="27e4a-140">When you executed the `migrations add` command, EF generated the code that will create the database from scratch.</span></span> <span data-ttu-id="27e4a-141">Questo codice si trova nella cartella *migrazioni* , nel file denominato *\<timestamp> _InitialCreate. cs*.</span><span class="sxs-lookup"><span data-stu-id="27e4a-141">This code is in the *Migrations* folder, in the file named *\<timestamp>_InitialCreate.cs*.</span></span> <span data-ttu-id="27e4a-142">Il metodo `Up` della classe `InitialCreate` crea le tabelle di database che corrispondono ai set di entità del modello di dati, e il metodo `Down` le elimina, come illustrato nell'esempio seguente.</span><span class="sxs-lookup"><span data-stu-id="27e4a-142">The `Up` method of the `InitialCreate` class creates the database tables that correspond to the data model entity sets, and the `Down` method deletes them, as shown in the following example.</span></span>

[!code-csharp[](intro/samples/cu/Migrations/20170215220724_InitialCreate.cs?range=92-118)]

<span data-ttu-id="27e4a-143">Le migrazioni chiamano il metodo `Up` per implementare le modifiche al modello di dati per una migrazione.</span><span class="sxs-lookup"><span data-stu-id="27e4a-143">Migrations calls the `Up` method to implement the data model changes for a migration.</span></span> <span data-ttu-id="27e4a-144">Quando si immette un comando per annullare l'aggiornamento, le migrazioni chiamano il metodo `Down`.</span><span class="sxs-lookup"><span data-stu-id="27e4a-144">When you enter a command to roll back the update, Migrations calls the `Down` method.</span></span>

<span data-ttu-id="27e4a-145">Questo codice è per la migrazione iniziale creata al momento dell'immissione del comando `migrations add InitialCreate`.</span><span class="sxs-lookup"><span data-stu-id="27e4a-145">This code is for the initial migration that was created when you entered the `migrations add InitialCreate` command.</span></span> <span data-ttu-id="27e4a-146">Il parametro del nome della migrazione, nell'esempio "InitialCreate", viene usato per il nome del file e può essere ciò che si vuole.</span><span class="sxs-lookup"><span data-stu-id="27e4a-146">The migration name parameter ("InitialCreate" in the example) is used for the file name and can be whatever you want.</span></span> <span data-ttu-id="27e4a-147">È consigliabile scegliere una parola o una frase che riepiloghi cosa viene fatto nella migrazione.</span><span class="sxs-lookup"><span data-stu-id="27e4a-147">It's best to choose a word or phrase that summarizes what is being done in the migration.</span></span> <span data-ttu-id="27e4a-148">Ad esempio, è possibile denominare una migrazione successiva "AddDepartmentTable".</span><span class="sxs-lookup"><span data-stu-id="27e4a-148">For example, you might name a later migration "AddDepartmentTable".</span></span>

<span data-ttu-id="27e4a-149">Se la migrazione iniziale è stata creata quando il database esisteva già, il codice di creazione del database viene generato ma non è necessario eseguirlo perché il database corrisponde già al modello di dati.</span><span class="sxs-lookup"><span data-stu-id="27e4a-149">If you created the initial migration when the database already exists, the database creation code is generated but it doesn't have to run because the database already matches the data model.</span></span> <span data-ttu-id="27e4a-150">Quando si distribuisce l'app in un altro ambiente in cui il database non esiste ancora, questo codice verrà eseguito per creare il database, è consigliabile quindi testarlo prima.</span><span class="sxs-lookup"><span data-stu-id="27e4a-150">When you deploy the app to another environment where the database doesn't exist yet, this code will run to create your database, so it's a good idea to test it first.</span></span> <span data-ttu-id="27e4a-151">Ecco perché in precedenza è stato modificato il nome del database nella stringa di connessione: per far sì che le migrazioni possano crearne uno nuovo da zero.</span><span class="sxs-lookup"><span data-stu-id="27e4a-151">That's why you changed the name of the database in the connection string earlier -- so that migrations can create a new one from scratch.</span></span>

## <a name="the-data-model-snapshot"></a><span data-ttu-id="27e4a-152">Snapshot del modello di dati</span><span class="sxs-lookup"><span data-stu-id="27e4a-152">The data model snapshot</span></span>

<span data-ttu-id="27e4a-153">Le migrazioni creano uno *snapshot* dello schema del database corrente in *Migrations/SchoolContextModelSnapshot.cs*.</span><span class="sxs-lookup"><span data-stu-id="27e4a-153">Migrations creates a *snapshot* of the current database schema in *Migrations/SchoolContextModelSnapshot.cs*.</span></span> <span data-ttu-id="27e4a-154">Quando si aggiunge una migrazione, EF determina le modifiche apportate confrontando il modello di dati con il file dello snapshot.</span><span class="sxs-lookup"><span data-stu-id="27e4a-154">When you add a migration, EF determines what changed by comparing the data model to the snapshot file.</span></span>

<span data-ttu-id="27e4a-155">Usare il comando [DotNet EF Migrations Remove](/ef/core/miscellaneous/cli/dotnet#dotnet-ef-migrations-remove) per rimuovere una migrazione.</span><span class="sxs-lookup"><span data-stu-id="27e4a-155">Use the [dotnet ef migrations remove](/ef/core/miscellaneous/cli/dotnet#dotnet-ef-migrations-remove) command to remove a migration.</span></span> <span data-ttu-id="27e4a-156">`dotnet ef migrations remove` elimina la migrazione e garantisce che lo snapshot venga reimpostato correttamente.</span><span class="sxs-lookup"><span data-stu-id="27e4a-156">`dotnet ef migrations remove` deletes the migration and ensures the snapshot is correctly reset.</span></span> <span data-ttu-id="27e4a-157">Se ha `dotnet ef migrations remove` esito negativo, usare `dotnet ef migrations remove -v` per ottenere altre informazioni sull'errore.</span><span class="sxs-lookup"><span data-stu-id="27e4a-157">If `dotnet ef migrations remove` fails, use `dotnet ef migrations remove -v` to get more information on the failure.</span></span>

<span data-ttu-id="27e4a-158">Per altre informazioni sull'uso del file di snapshot, vedere [EF Core Migrations in Team Environments](/ef/core/managing-schemas/migrations/teams) (Migrazioni EF Core in ambienti team).</span><span class="sxs-lookup"><span data-stu-id="27e4a-158">See [EF Core Migrations in Team Environments](/ef/core/managing-schemas/migrations/teams) for more information about how the snapshot file is used.</span></span>

## <a name="apply-the-migration"></a><span data-ttu-id="27e4a-159">Applicare la migrazione</span><span class="sxs-lookup"><span data-stu-id="27e4a-159">Apply the migration</span></span>

<span data-ttu-id="27e4a-160">Nella finestra di comando immettere il comando seguente per creare il database e le tabelle al suo interno.</span><span class="sxs-lookup"><span data-stu-id="27e4a-160">In the command window, enter the following command to create the database and tables in it.</span></span>

```dotnetcli
dotnet ef database update
```

<span data-ttu-id="27e4a-161">L'output del comando è simile al comando `migrations add`, a eccezione del fatto che vengono visualizzati i log per i comandi SQL che configurano il database.</span><span class="sxs-lookup"><span data-stu-id="27e4a-161">The output from the command is similar to the `migrations add` command, except that you see logs for the SQL commands that set up the database.</span></span> <span data-ttu-id="27e4a-162">La maggior parte dei log viene omessa nell'output di esempio seguente.</span><span class="sxs-lookup"><span data-stu-id="27e4a-162">Most of the logs are omitted in the following sample output.</span></span> <span data-ttu-id="27e4a-163">Se si vuole ridurre il livello di dettaglio nei messaggi di log, è possibile modificare i livelli di log nel file *appsettings.Development.json*.</span><span class="sxs-lookup"><span data-stu-id="27e4a-163">If you prefer not to see this level of detail in log messages, you can change the log level in the *appsettings.Development.json* file.</span></span> <span data-ttu-id="27e4a-164">Per altre informazioni, vedere <xref:fundamentals/logging/index>.</span><span class="sxs-lookup"><span data-stu-id="27e4a-164">For more information, see <xref:fundamentals/logging/index>.</span></span>

```text
info: Microsoft.EntityFrameworkCore.Infrastructure[10403]
      Entity Framework Core initialized 'SchoolContext' using provider 'Microsoft.EntityFrameworkCore.SqlServer' with options: None
info: Microsoft.EntityFrameworkCore.Database.Command[20101]
      Executed DbCommand (274ms) [Parameters=[], CommandType='Text', CommandTimeout='60']
      CREATE DATABASE [ContosoUniversity2];
info: Microsoft.EntityFrameworkCore.Database.Command[20101]
      Executed DbCommand (60ms) [Parameters=[], CommandType='Text', CommandTimeout='60']
      IF SERVERPROPERTY('EngineEdition') <> 5
      BEGIN
          ALTER DATABASE [ContosoUniversity2] SET READ_COMMITTED_SNAPSHOT ON;
      END;
info: Microsoft.EntityFrameworkCore.Database.Command[20101]
      Executed DbCommand (15ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
      CREATE TABLE [__EFMigrationsHistory] (
          [MigrationId] nvarchar(150) NOT NULL,
          [ProductVersion] nvarchar(32) NOT NULL,
          CONSTRAINT [PK___EFMigrationsHistory] PRIMARY KEY ([MigrationId])
      );

<logs omitted for brevity>

info: Microsoft.EntityFrameworkCore.Database.Command[20101]
      Executed DbCommand (3ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
      INSERT INTO [__EFMigrationsHistory] ([MigrationId], [ProductVersion])
      VALUES (N'20190327172701_InitialCreate', N'5.0-rtm');
Done.
```

<span data-ttu-id="27e4a-165">Per controllare il database come è stato fatto nella prima esercitazione, usare **Esplora oggetti di SQL Server**.</span><span class="sxs-lookup"><span data-stu-id="27e4a-165">Use **SQL Server Object Explorer** to inspect the database as you did in the first tutorial.</span></span>  <span data-ttu-id="27e4a-166">Si noterà l'aggiunta di una tabella \_\_EFMigrationsHistory che tiene traccia di quali migrazioni sono state applicate al database.</span><span class="sxs-lookup"><span data-stu-id="27e4a-166">You'll notice the addition of an \_\_EFMigrationsHistory table that keeps track of which migrations have been applied to the database.</span></span> <span data-ttu-id="27e4a-167">Visualizzare i dati nella tabella: si noterà la presenza di una riga per la prima migrazione.</span><span class="sxs-lookup"><span data-stu-id="27e4a-167">View the data in that table and you'll see one row for the first migration.</span></span> <span data-ttu-id="27e4a-168">Nell'ultimo log nell'esempio di output della CLI precedente viene visualizzata l'istruzione INSERT che crea tale riga.</span><span class="sxs-lookup"><span data-stu-id="27e4a-168">(The last log in the preceding CLI output example shows the INSERT statement that creates this row.)</span></span>

<span data-ttu-id="27e4a-169">Eseguire l'applicazione per verificare che tutto funzioni come prima.</span><span class="sxs-lookup"><span data-stu-id="27e4a-169">Run the application to verify that everything still works the same as before.</span></span>

![Pagina Student Index (Indice degli studenti)](migrations/_static/students-index.png)

<a id="pmc"></a>

## <a name="compare-cli-and-pmc"></a><span data-ttu-id="27e4a-171">Interfaccia della riga di comando e console di gestione pacchetto a confronto</span><span class="sxs-lookup"><span data-stu-id="27e4a-171">Compare CLI and PMC</span></span>

<span data-ttu-id="27e4a-172">Gli strumenti di EF per la gestione delle migrazioni sono disponibili dai comandi della CLI di .NET Core o dai cmdlet di PowerShell in Visual Studio nella finestra **Console di Gestione pacchetti**.</span><span class="sxs-lookup"><span data-stu-id="27e4a-172">The EF tooling for managing migrations is available from .NET Core CLI commands or from PowerShell cmdlets in the Visual Studio **Package Manager Console** (PMC) window.</span></span> <span data-ttu-id="27e4a-173">In questa esercitazione viene illustrato come usare la CLI, ma se si preferisce è possibile usare la console di Gestione pacchetti.</span><span class="sxs-lookup"><span data-stu-id="27e4a-173">This tutorial shows how to use the CLI, but you can use the PMC if you prefer.</span></span>

<span data-ttu-id="27e4a-174">I comandi di EF per la console di Gestione pacchetti sono inclusi nel pacchetto [Microsoft.EntityFrameworkCore.Tools](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Tools).</span><span class="sxs-lookup"><span data-stu-id="27e4a-174">The EF commands for the PMC commands are in the [Microsoft.EntityFrameworkCore.Tools](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Tools) package.</span></span> <span data-ttu-id="27e4a-175">Questo pacchetto è incluso nel [metapacchetto Microsoft.AspNetCore.App](xref:fundamentals/metapackage-app), quindi non è necessario aggiungere un riferimento al pacchetto se l'app dispone di un riferimento al pacchetto per `Microsoft.AspNetCore.App`.</span><span class="sxs-lookup"><span data-stu-id="27e4a-175">This package is included in the [Microsoft.AspNetCore.App metapackage](xref:fundamentals/metapackage-app), so you don't need to add a package reference if your app has a package reference for `Microsoft.AspNetCore.App`.</span></span>

<span data-ttu-id="27e4a-176">**Importante:** non si tratta dello stesso pacchetto che si installa per l'interfaccia della riga di comando modificando il file con estensione *.csproj*.</span><span class="sxs-lookup"><span data-stu-id="27e4a-176">**Important:** This isn't the same package as the one you install for the CLI by editing the *.csproj* file.</span></span> <span data-ttu-id="27e4a-177">Il nome di questo pacchetto termina con `Tools`, a differenza del nome del pacchetto della CLI che termina con `Tools.DotNet`.</span><span class="sxs-lookup"><span data-stu-id="27e4a-177">The name of this one ends in `Tools`, unlike the CLI package name which ends in `Tools.DotNet`.</span></span>

<span data-ttu-id="27e4a-178">Per altre informazioni sui comandi della CLI, vedere [Strumenti da riga di comando di EF Core .NET](/ef/core/miscellaneous/cli/dotnet).</span><span class="sxs-lookup"><span data-stu-id="27e4a-178">For more information about the CLI commands, see [.NET Core CLI](/ef/core/miscellaneous/cli/dotnet).</span></span>

<span data-ttu-id="27e4a-179">Per altre informazioni sui comandi della console di Gestione pacchetti, vedere [Console di Gestione pacchetti (Visual Studio)](/ef/core/miscellaneous/cli/powershell).</span><span class="sxs-lookup"><span data-stu-id="27e4a-179">For more information about the PMC commands, see [Package Manager Console (Visual Studio)](/ef/core/miscellaneous/cli/powershell).</span></span>

## <a name="get-the-code"></a><span data-ttu-id="27e4a-180">Ottenere il codice</span><span class="sxs-lookup"><span data-stu-id="27e4a-180">Get the code</span></span>

[<span data-ttu-id="27e4a-181">Scaricare o visualizzare l'applicazione completata.</span><span class="sxs-lookup"><span data-stu-id="27e4a-181">Download or view the completed application.</span></span>](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/data/ef-mvc/intro/samples)

## <a name="next-step"></a><span data-ttu-id="27e4a-182">Passaggio successivo</span><span class="sxs-lookup"><span data-stu-id="27e4a-182">Next step</span></span>

<span data-ttu-id="27e4a-183">Passare all'esercitazione successiva per iniziare a esaminare argomenti più avanzati sull'espansione del modello di dati.</span><span class="sxs-lookup"><span data-stu-id="27e4a-183">Advance to the next tutorial to begin looking at more advanced topics about expanding the data model.</span></span> <span data-ttu-id="27e4a-184">Lungo il percorso verranno create e applicate altre migrazioni.</span><span class="sxs-lookup"><span data-stu-id="27e4a-184">Along the way you'll create and apply additional migrations.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="27e4a-185">Creare e applicare altre migrazioni</span><span class="sxs-lookup"><span data-stu-id="27e4a-185">Create and apply additional migrations</span></span>](complex-data-model.md)
