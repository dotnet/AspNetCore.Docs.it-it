---
title: Parte 2, aggiungere un modello
author: rick-anderson
description: Parte 2 della serie di esercitazioni sulle Razor pagine. In questa sezione vengono aggiunte le classi del modello.
ms.author: riande
ms.date: 03/10/2021
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
uid: tutorials/razor-pages/model
ms.openlocfilehash: defbc73d0c1d6aac30360cd7b83cc518a407bf98
ms.sourcegitcommit: 07e7ee573fe4e12be93249a385db745d714ff6ae
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/12/2021
ms.locfileid: "103413444"
---
# <a name="part-2-add-a-model-to-a-razor-pages-app-in-aspnet-core"></a><span data-ttu-id="5792c-104">Parte 2, aggiungere un modello a un' Razor app di pagine in ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="5792c-104">Part 2, add a model to a Razor Pages app in ASP.NET Core</span></span>

<span data-ttu-id="5792c-105">Autore: [Rick Anderson](https://twitter.com/RickAndMSFT)</span><span class="sxs-lookup"><span data-stu-id="5792c-105">By [Rick Anderson](https://twitter.com/RickAndMSFT)</span></span>

::: moniker range=">= aspnetcore-5.0"

<!-- In the next update on the CLI version, let the scaffolder do the same work the VS driven scaffolder does. That is, create the DB context, etc -->

<span data-ttu-id="5792c-106">In questa sezione si aggiungono alcune classi per la gestione di filmati in un database.</span><span class="sxs-lookup"><span data-stu-id="5792c-106">In this section, classes are added for managing movies in a database.</span></span> <span data-ttu-id="5792c-107">Le classi modello dell'app usano [Entity Framework Core (EF Core)](/ef/core) per lavorare con il database.</span><span class="sxs-lookup"><span data-stu-id="5792c-107">The app's model classes use [Entity Framework Core (EF Core)](/ef/core) to work with the database.</span></span> <span data-ttu-id="5792c-108">EF Core è un mapper relazionale a oggetti (O/RM) che semplifica l'accesso ai dati.</span><span class="sxs-lookup"><span data-stu-id="5792c-108">EF Core is an object-relational mapper (O/RM) that simplifies data access.</span></span> <span data-ttu-id="5792c-109">Prima di tutto si scrivono le classi del modello e EF Core crea il database.</span><span class="sxs-lookup"><span data-stu-id="5792c-109">You write the model classes first, and EF Core creates the database.</span></span>

<span data-ttu-id="5792c-110">Le classi del modello sono note come classi POCO (da "**P** Lain-**o** LD **C** LR **o** gli oggetti") perché non hanno una dipendenza da EF core.</span><span class="sxs-lookup"><span data-stu-id="5792c-110">The model classes are known as POCO classes (from "**P** lain-**O** ld **C** LR **O** bjects") because they don't have a dependency on EF Core.</span></span> <span data-ttu-id="5792c-111">Definiscono le proprietà dei dati archiviati nel database.</span><span class="sxs-lookup"><span data-stu-id="5792c-111">They define the properties of the data that are stored in the database.</span></span>

<span data-ttu-id="5792c-112">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie50) ([procedura per il download](xref:index#how-to-download-a-sample)).</span><span class="sxs-lookup"><span data-stu-id="5792c-112">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie50) ([how to download](xref:index#how-to-download-a-sample)).</span></span>

## <a name="add-a-data-model"></a><span data-ttu-id="5792c-113">Aggiungere un modello di dati</span><span class="sxs-lookup"><span data-stu-id="5792c-113">Add a data model</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="5792c-114">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="5792c-114">Visual Studio</span></span>](#tab/visual-studio)

1. <span data-ttu-id="5792c-115">In **Esplora soluzioni** fare clic con il pulsante destro del mouse sul progetto *Razor PagesMovie* > **Aggiungi**  >  **nuova cartella**.</span><span class="sxs-lookup"><span data-stu-id="5792c-115">In **Solution Explorer**, right-click the *RazorPagesMovie* project > **Add** > **New Folder**.</span></span> <span data-ttu-id="5792c-116">Assegnare il nome *Modelli* alla cartella.</span><span class="sxs-lookup"><span data-stu-id="5792c-116">Name the folder *Models*.</span></span>
1. <span data-ttu-id="5792c-117">Fare clic con il pulsante destro del mouse sulla cartella *modelli* .</span><span class="sxs-lookup"><span data-stu-id="5792c-117">Right-click the *Models* folder.</span></span> <span data-ttu-id="5792c-118">Selezionare **Aggiungi**  >  **classe**.</span><span class="sxs-lookup"><span data-stu-id="5792c-118">Select **Add** > **Class**.</span></span> <span data-ttu-id="5792c-119">Denominare la classe *Movie*.</span><span class="sxs-lookup"><span data-stu-id="5792c-119">Name the class *Movie*.</span></span>
1. <span data-ttu-id="5792c-120">Aggiungere le proprietà seguenti alla classe `Movie`:</span><span class="sxs-lookup"><span data-stu-id="5792c-120">Add the following properties to the `Movie` class:</span></span>

   [!code-csharp[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie22/Models/Movie.cs?name=snippet1)]

<span data-ttu-id="5792c-121">La classe `Movie` contiene:</span><span class="sxs-lookup"><span data-stu-id="5792c-121">The `Movie` class contains:</span></span>

* <span data-ttu-id="5792c-122">Il campo `ID` è richiesto dal database per la chiave primaria.</span><span class="sxs-lookup"><span data-stu-id="5792c-122">The `ID` field is required by the database for the primary key.</span></span>
* <span data-ttu-id="5792c-123">`[DataType(DataType.Date)]`: L'attributo [[DataType]](xref:System.ComponentModel.DataAnnotations.DataTypeAttribute) specifica il tipo di dati ( `Date` ).</span><span class="sxs-lookup"><span data-stu-id="5792c-123">`[DataType(DataType.Date)]`: The [[DataType]](xref:System.ComponentModel.DataAnnotations.DataTypeAttribute) attribute specifies the type of the data (`Date`).</span></span> <span data-ttu-id="5792c-124">Con questo attributo:</span><span class="sxs-lookup"><span data-stu-id="5792c-124">With this attribute:</span></span>

  * <span data-ttu-id="5792c-125">L'utente non deve immettere le informazioni sull'ora nel campo Data.</span><span class="sxs-lookup"><span data-stu-id="5792c-125">The user isn't required to enter time information in the date field.</span></span>
  * <span data-ttu-id="5792c-126">Viene visualizzata solo la data, non le informazioni temporali.</span><span class="sxs-lookup"><span data-stu-id="5792c-126">Only the date is displayed, not time information.</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="5792c-127">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="5792c-127">Visual Studio Code</span></span>](#tab/visual-studio-code)

1. <span data-ttu-id="5792c-128">Aggiungere una cartella denominata *Modelli*.</span><span class="sxs-lookup"><span data-stu-id="5792c-128">Add a folder named *Models*.</span></span>
1. <span data-ttu-id="5792c-129">Aggiungere una classe alla cartella *Modello* denominata *Movie.cs*.</span><span class="sxs-lookup"><span data-stu-id="5792c-129">Add a class to the *Models* folder named *Movie.cs*.</span></span>

<span data-ttu-id="5792c-130">Aggiungere le proprietà seguenti alla classe `Movie`:</span><span class="sxs-lookup"><span data-stu-id="5792c-130">Add the following properties to the `Movie` class:</span></span>

[!code-csharp[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie22/Models/Movie.cs?name=snippet1)]

<span data-ttu-id="5792c-131">La classe `Movie` contiene:</span><span class="sxs-lookup"><span data-stu-id="5792c-131">The `Movie` class contains:</span></span>

* <span data-ttu-id="5792c-132">Il campo `ID` è richiesto dal database per la chiave primaria.</span><span class="sxs-lookup"><span data-stu-id="5792c-132">The `ID` field is required by the database for the primary key.</span></span>
* <span data-ttu-id="5792c-133">`[DataType(DataType.Date)]`: L'attributo [[DataType]](xref:System.ComponentModel.DataAnnotations.DataTypeAttribute) specifica il tipo di dati ( `Date` ).</span><span class="sxs-lookup"><span data-stu-id="5792c-133">`[DataType(DataType.Date)]`: The [[DataType]](xref:System.ComponentModel.DataAnnotations.DataTypeAttribute) attribute specifies the type of the data (`Date`).</span></span> <span data-ttu-id="5792c-134">Con questo attributo:</span><span class="sxs-lookup"><span data-stu-id="5792c-134">With this attribute:</span></span>

  * <span data-ttu-id="5792c-135">l'utente non deve immettere le informazioni temporali nel campo della data.</span><span class="sxs-lookup"><span data-stu-id="5792c-135">The user is not required to enter time information in the date field.</span></span>
  * <span data-ttu-id="5792c-136">Viene visualizzata solo la data, non le informazioni temporali.</span><span class="sxs-lookup"><span data-stu-id="5792c-136">Only the date is displayed, not time information.</span></span>

<a name="dc"></a>

### <a name="add-nuget-packages-and-ef-tools"></a><span data-ttu-id="5792c-137">Aggiungere i pacchetti NuGet e gli strumenti EF</span><span class="sxs-lookup"><span data-stu-id="5792c-137">Add NuGet packages and EF tools</span></span>

[!INCLUDE[](~/includes/add-EF-NuGet-SQLite-CLI-5.md)]

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="5792c-138">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="5792c-138">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

1. <span data-ttu-id="5792c-139">Nella **finestra degli strumenti della soluzione**, fare clic sul progetto *Razor PagesMovie* e quindi selezionare **Aggiungi** > **nuova cartella.** Assegnare un nome ai *modelli* di cartella.</span><span class="sxs-lookup"><span data-stu-id="5792c-139">In the **Solution Tool Window**, control-click the *RazorPagesMovie* project, and then select **Add** > **New Folder...**. Name the folder *Models*.</span></span>
1. <span data-ttu-id="5792c-140">CTRL-fare clic sulla cartella *modelli* , quindi selezionare **Aggiungi** > **nuovo file.**...</span><span class="sxs-lookup"><span data-stu-id="5792c-140">Control-click the *Models* folder, and then select **Add** > **New File...**.</span></span>
1. <span data-ttu-id="5792c-141">Nel finestra di dialogo **Nuovo file**:</span><span class="sxs-lookup"><span data-stu-id="5792c-141">In the **New File** dialog:</span></span>
   1. <span data-ttu-id="5792c-142">Selezionare **Generale** nel riquadro a sinistra.</span><span class="sxs-lookup"><span data-stu-id="5792c-142">Select **General** in the left pane.</span></span>
   1. <span data-ttu-id="5792c-143">Selezionare **Classe vuota** nel riquadro centrale.</span><span class="sxs-lookup"><span data-stu-id="5792c-143">Select **Empty Class** in the center pane.</span></span>
   1. <span data-ttu-id="5792c-144">Denominare la classe **Filmato** e selezionare **Nuovo**.</span><span class="sxs-lookup"><span data-stu-id="5792c-144">Name the class **Movie** and select **New**.</span></span>

1. <span data-ttu-id="5792c-145">Aggiungere le proprietà seguenti alla classe `Movie`:</span><span class="sxs-lookup"><span data-stu-id="5792c-145">Add the following properties to the `Movie` class:</span></span>

   [!code-csharp[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie22/Models/Movie.cs?name=snippet1)]

<span data-ttu-id="5792c-146">La classe `Movie` contiene:</span><span class="sxs-lookup"><span data-stu-id="5792c-146">The `Movie` class contains:</span></span>

* <span data-ttu-id="5792c-147">Il campo `ID` è richiesto dal database per la chiave primaria.</span><span class="sxs-lookup"><span data-stu-id="5792c-147">The `ID` field is required by the database for the primary key.</span></span>
* <span data-ttu-id="5792c-148">`[DataType(DataType.Date)]`: L'attributo [[DataType]](xref:System.ComponentModel.DataAnnotations.DataTypeAttribute) specifica il tipo di dati ( `Date` ).</span><span class="sxs-lookup"><span data-stu-id="5792c-148">`[DataType(DataType.Date)]`: The [[DataType]](xref:System.ComponentModel.DataAnnotations.DataTypeAttribute) attribute specifies the type of the data (`Date`).</span></span> <span data-ttu-id="5792c-149">Con questo attributo:</span><span class="sxs-lookup"><span data-stu-id="5792c-149">With this attribute:</span></span>

  * <span data-ttu-id="5792c-150">l'utente non deve immettere le informazioni temporali nel campo della data.</span><span class="sxs-lookup"><span data-stu-id="5792c-150">The user is not required to enter time information in the date field.</span></span>
  * <span data-ttu-id="5792c-151">Viene visualizzata solo la data, non le informazioni temporali.</span><span class="sxs-lookup"><span data-stu-id="5792c-151">Only the date is displayed, not time information.</span></span>

---

<span data-ttu-id="5792c-152">L'attributo [DataAnnotations](xref:System.ComponentModel.DataAnnotations) viene analizzato in un'esercitazione successiva.</span><span class="sxs-lookup"><span data-stu-id="5792c-152">[DataAnnotations](xref:System.ComponentModel.DataAnnotations) are covered in a later tutorial.</span></span>

<span data-ttu-id="5792c-153">Compilare il progetto per verificare che non siano presenti errori di compilazione.</span><span class="sxs-lookup"><span data-stu-id="5792c-153">Build the project to verify there are no compilation errors.</span></span>

## <a name="scaffold-the-movie-model"></a><span data-ttu-id="5792c-154">Eseguire lo scaffolding del modello di filmato</span><span class="sxs-lookup"><span data-stu-id="5792c-154">Scaffold the movie model</span></span>

<span data-ttu-id="5792c-155">In questa sezione viene eseguito lo scaffolding del modello *Movie*.</span><span class="sxs-lookup"><span data-stu-id="5792c-155">In this section, the movie model is scaffolded.</span></span> <span data-ttu-id="5792c-156">Lo strumento di scaffolding crea quindi le pagine per le operazioni CRUD (creazione, lettura, aggiornamento ed eliminazione) per il modello *Movie*.</span><span class="sxs-lookup"><span data-stu-id="5792c-156">That is, the scaffolding tool produces pages for Create, Read, Update, and Delete (CRUD) operations for the movie model.</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="5792c-157">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="5792c-157">Visual Studio</span></span>](#tab/visual-studio)

1. <span data-ttu-id="5792c-158">Creare una cartella *Pages/Movies*:</span><span class="sxs-lookup"><span data-stu-id="5792c-158">Create a *Pages/Movies* folder:</span></span>
   1. <span data-ttu-id="5792c-159">Fare clic con il pulsante destro del mouse sulla cartella *pagine* > **Aggiungi** > **nuova cartella**.</span><span class="sxs-lookup"><span data-stu-id="5792c-159">Right-click on the *Pages* folder > **Add** > **New Folder**.</span></span>
   1. <span data-ttu-id="5792c-160">Denominare la cartella *Movies*.</span><span class="sxs-lookup"><span data-stu-id="5792c-160">Name the folder *Movies*.</span></span>

1. <span data-ttu-id="5792c-161">Fare clic con il pulsante destro del mouse sulla cartella *pages/Movies* > **Aggiungi** > **nuovo elemento con impalcatura**.</span><span class="sxs-lookup"><span data-stu-id="5792c-161">Right-click on the *Pages/Movies* folder > **Add** > **New Scaffolded Item**.</span></span>

   ![Immagine relativa alle istruzioni precedenti.](model/_static/5/sca.png)

1. <span data-ttu-id="5792c-163">Nella finestra di dialogo **Aggiungi impalcatura** selezionare **Razor pages using Entity Framework (CRUD)** > **Add**.</span><span class="sxs-lookup"><span data-stu-id="5792c-163">In the **Add Scaffold** dialog, select **Razor Pages using Entity Framework (CRUD)** > **Add**.</span></span>

   ![Immagine relativa alle istruzioni precedenti.](model/_static/add_scaffold.png)

1. <span data-ttu-id="5792c-165">Completare la finestra di dialogo **Aggiungi Razor pagine con Entity Framework (CRUD)** :</span><span class="sxs-lookup"><span data-stu-id="5792c-165">Complete the **Add Razor Pages using Entity Framework (CRUD)** dialog:</span></span>
   1. <span data-ttu-id="5792c-166">Nell'elenco a discesa **classe modello** selezionare **Movie ( Razor PagesMovie. Models)**.</span><span class="sxs-lookup"><span data-stu-id="5792c-166">In the **Model class** drop down, select **Movie (RazorPagesMovie.Models)**.</span></span>
   1. <span data-ttu-id="5792c-167">Nella riga **Classe contesto di dati** selezionare il segno più **+**.</span><span class="sxs-lookup"><span data-stu-id="5792c-167">In the **Data context class** row, select the **+** (plus) sign.</span></span>
      1. <span data-ttu-id="5792c-168">Nella finestra di dialogo **Aggiungi contesto dati** viene generato il nome della classe `RazorPagesMovie.Data.RazorPagesMovieContext` .</span><span class="sxs-lookup"><span data-stu-id="5792c-168">In the **Add Data Context** dialog, the class name `RazorPagesMovie.Data.RazorPagesMovieContext` is generated.</span></span>
   1. <span data-ttu-id="5792c-169">Selezionare **Aggiungi**.</span><span class="sxs-lookup"><span data-stu-id="5792c-169">Select **Add**.</span></span>

   ![Immagine relativa alle istruzioni precedenti.](model/_static/3/arp.png)

<span data-ttu-id="5792c-171">Il *appsettings.json* file viene aggiornato con la stringa di connessione utilizzata per connettersi a un database locale.</span><span class="sxs-lookup"><span data-stu-id="5792c-171">The *appsettings.json* file is updated with the connection string used to connect to a local database.</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="5792c-172">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="5792c-172">Visual Studio Code</span></span>](#tab/visual-studio-code)

<!--  Until https://github.com/aspnet/Scaffolding/issues/582 is fixed windows needs backslash or the namespace is namespace RazorPagesMovie.Pages_Movies rather than namespace RazorPagesMovie.Pages.Movies
-->

* <span data-ttu-id="5792c-173">Aprire una shell dei comandi nella directory del progetto che contiene i file *Program.cs*, *Startup.cs* e *. csproj* .</span><span class="sxs-lookup"><span data-stu-id="5792c-173">Open a command shell to the project directory, which contains the *Program.cs*, *Startup.cs*, and *.csproj* files.</span></span>

* <span data-ttu-id="5792c-174">**Per Windows**: eseguire il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="5792c-174">**For Windows**: Run the following command:</span></span>

  ```dotnetcli
  dotnet-aspnet-codegenerator razorpage -m Movie -dc RazorPagesMovieContext -udl -outDir Pages\Movies --referenceScriptLibraries
  ```

* <span data-ttu-id="5792c-175">**Per MacOS e Linux**: eseguire il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="5792c-175">**For macOS and Linux**: Run the following command:</span></span>

  ```dotnetcli
  dotnet-aspnet-codegenerator razorpage -m Movie -dc RazorPagesMovieContext -udl -outDir Pages/Movies --referenceScriptLibraries
  ```

<a name="codegenerator"></a> <span data-ttu-id="5792c-176">La tabella seguente illustra in dettaglio le opzioni del generatore di codice ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="5792c-176">The following table details the ASP.NET Core code generator options.</span></span>

| <span data-ttu-id="5792c-177">Opzione</span><span class="sxs-lookup"><span data-stu-id="5792c-177">Option</span></span>               | <span data-ttu-id="5792c-178">Descrizione</span><span class="sxs-lookup"><span data-stu-id="5792c-178">Description</span></span>|
| ----------------- | ------------ |
| `-m`  | <span data-ttu-id="5792c-179">Nome del modello.</span><span class="sxs-lookup"><span data-stu-id="5792c-179">The name of the model.</span></span> |
| `-dc`  | <span data-ttu-id="5792c-180">Classe `DbContext` da usare.</span><span class="sxs-lookup"><span data-stu-id="5792c-180">The `DbContext` class to use.</span></span> |
| `-udl` | <span data-ttu-id="5792c-181">Uso del layout predefinito.</span><span class="sxs-lookup"><span data-stu-id="5792c-181">Use the default layout.</span></span> |
| `-outDir` | <span data-ttu-id="5792c-182">Percorso relativo della cartella di output per creare le viste.</span><span class="sxs-lookup"><span data-stu-id="5792c-182">The relative output folder path to create the views.</span></span> |
| `--referenceScriptLibraries` | <span data-ttu-id="5792c-183">Aggiunge `_ValidationScriptsPartial` per modificare e creare pagine</span><span class="sxs-lookup"><span data-stu-id="5792c-183">Adds `_ValidationScriptsPartial` to Edit and Create pages</span></span> |

<span data-ttu-id="5792c-184">Usare l' `-h` opzione per ottenere informazioni sul `aspnet-codegenerator razorpage` comando:</span><span class="sxs-lookup"><span data-stu-id="5792c-184">Use the `-h` option to get help on the `aspnet-codegenerator razorpage` command:</span></span>

```dotnetcli
dotnet-aspnet-codegenerator razorpage -h
```

<span data-ttu-id="5792c-185">Per ulteriori informazioni, vedere [DotNet-ASPNET-CodeGenerator](xref:fundamentals/tools/dotnet-aspnet-codegenerator).</span><span class="sxs-lookup"><span data-stu-id="5792c-185">For more information, see [dotnet-aspnet-codegenerator](xref:fundamentals/tools/dotnet-aspnet-codegenerator).</span></span>

### <a name="use-sqlite-for-development-sql-server-for-production"></a><span data-ttu-id="5792c-186">Usare SQLite per lo sviluppo, SQL Server per la produzione</span><span class="sxs-lookup"><span data-stu-id="5792c-186">Use SQLite for development, SQL Server for production</span></span>

<span data-ttu-id="5792c-187">Quando si seleziona SQLite, il codice generato dal modello è pronto per lo sviluppo.</span><span class="sxs-lookup"><span data-stu-id="5792c-187">When SQLite is selected, the template generated code is ready for development.</span></span> <span data-ttu-id="5792c-188">Nel codice seguente viene illustrato come inserire <xref:Microsoft.AspNetCore.Hosting.IWebHostEnvironment> in `Startup` .</span><span class="sxs-lookup"><span data-stu-id="5792c-188">The following code shows how to inject <xref:Microsoft.AspNetCore.Hosting.IWebHostEnvironment> into `Startup`.</span></span> <span data-ttu-id="5792c-189">`IWebHostEnvironment` viene inserito in modo che l'app possa usare SQLite in fase di sviluppo e SQL Server nell'ambiente di produzione.</span><span class="sxs-lookup"><span data-stu-id="5792c-189">`IWebHostEnvironment` is injected so the app can use SQLite in development and SQL Server in production.</span></span>

[!code-csharp[](~/includes/RP/code/StartupDevProd.cs?name=snippet&highlight=5,10,14)]

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="5792c-190">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="5792c-190">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

1. <span data-ttu-id="5792c-191">Creare una cartella *Pages/Movies*:</span><span class="sxs-lookup"><span data-stu-id="5792c-191">Create a *Pages/Movies* folder:</span></span>
   1. <span data-ttu-id="5792c-192">CTRL: fare clic sulla cartella *pagine* > **Aggiungi** > **nuova cartella**.</span><span class="sxs-lookup"><span data-stu-id="5792c-192">Control-click on the *Pages* folder > **Add** > **New Folder**.</span></span>
   1. <span data-ttu-id="5792c-193">Denominare la cartella *Movies*.</span><span class="sxs-lookup"><span data-stu-id="5792c-193">Name the folder *Movies*.</span></span>

1. <span data-ttu-id="5792c-194">CTRL: fare clic sulla cartella *pagine/filmati* > **Aggiungi** > **Nuova impalcatura...**.</span><span class="sxs-lookup"><span data-stu-id="5792c-194">Control-click on the *Pages/Movies* folder > **Add** > **New Scaffolding...**.</span></span>

   ![Immagine relativa alle istruzioni precedenti.](model/_static/scaMac.png)

1. <span data-ttu-id="5792c-196">Nella finestra di dialogo **Nuova impalcatura** selezionare **Razor pagine con Entity Framework (CRUD)** > **Avanti**.</span><span class="sxs-lookup"><span data-stu-id="5792c-196">In the **New Scaffolding** dialog, select **Razor Pages using Entity Framework (CRUD)** > **Next**.</span></span>

   ![Immagine relativa alle istruzioni precedenti.](model/_static/add_scaffoldMac.png)

1. <span data-ttu-id="5792c-198">Completare la finestra di dialogo **Aggiungi Razor pagine con Entity Framework (CRUD)** :</span><span class="sxs-lookup"><span data-stu-id="5792c-198">Complete the **Add Razor Pages using Entity Framework (CRUD)** dialog:</span></span>
   1. <span data-ttu-id="5792c-199">Nella **classe DbContext da usare:** Row, assegnare un nome alla classe `RazorPagesMovie.Data.RazorPagesMovieContext` .</span><span class="sxs-lookup"><span data-stu-id="5792c-199">In the **DbContext Class to use:** row, name the class `RazorPagesMovie.Data.RazorPagesMovieContext`.</span></span>
   1. <span data-ttu-id="5792c-200">Selezionare **Fine**.</span><span class="sxs-lookup"><span data-stu-id="5792c-200">Select **Finish**.</span></span>

   ![Immagine relativa alle istruzioni precedenti.](model/_static/5/arpMac.png)

<span data-ttu-id="5792c-202">Il *appsettings.json* file viene aggiornato con la stringa di connessione utilizzata per connettersi a un database locale.</span><span class="sxs-lookup"><span data-stu-id="5792c-202">The *appsettings.json* file is updated with the connection string used to connect to a local database.</span></span>

### <a name="use-sqlite-for-development-sql-server-for-production"></a><span data-ttu-id="5792c-203">Usare SQLite per lo sviluppo, SQL Server per la produzione</span><span class="sxs-lookup"><span data-stu-id="5792c-203">Use SQLite for development, SQL Server for production</span></span>

<span data-ttu-id="5792c-204">Quando si seleziona SQLite, il codice generato dal modello è pronto per lo sviluppo.</span><span class="sxs-lookup"><span data-stu-id="5792c-204">When SQLite is selected, the template generated code is ready for development.</span></span> <span data-ttu-id="5792c-205">Nel codice seguente viene illustrato come inserire <xref:Microsoft.AspNetCore.Hosting.IWebHostEnvironment> in `Startup` .</span><span class="sxs-lookup"><span data-stu-id="5792c-205">The following code shows how to inject <xref:Microsoft.AspNetCore.Hosting.IWebHostEnvironment> into `Startup`.</span></span> <span data-ttu-id="5792c-206">`IWebHostEnvironment` viene inserito in modo che l'app possa usare SQLite in fase di sviluppo e SQL Server nell'ambiente di produzione.</span><span class="sxs-lookup"><span data-stu-id="5792c-206">`IWebHostEnvironment` is injected so the app can use SQLite in development and SQL Server in production.</span></span>

[!code-csharp[](~/includes/RP/code/StartupDevProd.cs?name=snippet&highlight=5,10,14)]

---

### <a name="files-created"></a><span data-ttu-id="5792c-207">File creati</span><span class="sxs-lookup"><span data-stu-id="5792c-207">Files created</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="5792c-208">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="5792c-208">Visual Studio</span></span>](#tab/visual-studio)

<span data-ttu-id="5792c-209">Il processo di scaffolding crea e aggiorna i file seguenti:</span><span class="sxs-lookup"><span data-stu-id="5792c-209">The scaffold process creates and updates the following files:</span></span>

* <span data-ttu-id="5792c-210">*Pagine/filmati*: creare, eliminare, Details, Edit e Index .</span><span class="sxs-lookup"><span data-stu-id="5792c-210">*Pages/Movies*: Create, Delete, Details, Edit, and Index.</span></span>
* <span data-ttu-id="5792c-211">*Data/ Razor PagesMovieContext.cs*</span><span class="sxs-lookup"><span data-stu-id="5792c-211">*Data/RazorPagesMovieContext.cs*</span></span>

### <a name="updated"></a><span data-ttu-id="5792c-212">Aggiornato</span><span class="sxs-lookup"><span data-stu-id="5792c-212">Updated</span></span>

* <span data-ttu-id="5792c-213">*Startup.cs*</span><span class="sxs-lookup"><span data-stu-id="5792c-213">*Startup.cs*</span></span>

<span data-ttu-id="5792c-214">I file creati e aggiornati sono illustrati nella sezione successiva.</span><span class="sxs-lookup"><span data-stu-id="5792c-214">The created and updated files are explained in the next section.</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="5792c-215">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="5792c-215">Visual Studio Code</span></span>](#tab/visual-studio-code)

<span data-ttu-id="5792c-216">Il processo di scaffolding crea i file seguenti:</span><span class="sxs-lookup"><span data-stu-id="5792c-216">The scaffold process creates the following files:</span></span>

* <span data-ttu-id="5792c-217">*Pagine/filmati*: creare, eliminare, Details, Edit e Index .</span><span class="sxs-lookup"><span data-stu-id="5792c-217">*Pages/Movies*: Create, Delete, Details, Edit, and Index.</span></span>

<span data-ttu-id="5792c-218">I file creati sono illustrati nella sezione successiva.</span><span class="sxs-lookup"><span data-stu-id="5792c-218">The created files are explained in the next section.</span></span>

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="5792c-219">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="5792c-219">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

<span data-ttu-id="5792c-220">Il processo di scaffolding crea e aggiorna i file seguenti:</span><span class="sxs-lookup"><span data-stu-id="5792c-220">The scaffold process creates and updates the following files:</span></span>

* <span data-ttu-id="5792c-221">*Pagine/filmati*: creare, eliminare, Details, Edit e Index .</span><span class="sxs-lookup"><span data-stu-id="5792c-221">*Pages/Movies*: Create, Delete, Details, Edit, and Index.</span></span>
* <span data-ttu-id="5792c-222">*Data/ Razor PagesMovieContext.cs*</span><span class="sxs-lookup"><span data-stu-id="5792c-222">*Data/RazorPagesMovieContext.cs*</span></span>

### <a name="updated"></a><span data-ttu-id="5792c-223">Aggiornato</span><span class="sxs-lookup"><span data-stu-id="5792c-223">Updated</span></span>

* <span data-ttu-id="5792c-224">*Startup.cs*</span><span class="sxs-lookup"><span data-stu-id="5792c-224">*Startup.cs*</span></span>

<span data-ttu-id="5792c-225">I file creati e aggiornati sono illustrati nella sezione successiva.</span><span class="sxs-lookup"><span data-stu-id="5792c-225">The created and updated files are explained in the next section.</span></span>

---

<a name="pmc"></a>

## <a name="create-the-initial-database-schema-using-efs-migration-feature"></a><span data-ttu-id="5792c-226">Creare lo schema del database iniziale usando la funzionalità di migrazione di EF</span><span class="sxs-lookup"><span data-stu-id="5792c-226">Create the initial database schema using EF's migration feature</span></span>

<span data-ttu-id="5792c-227">La funzionalità migrazioni di Entity Framework Core fornisce un modo per:</span><span class="sxs-lookup"><span data-stu-id="5792c-227">The migrations feature in Entity Framework Core provides a way to:</span></span>

* <span data-ttu-id="5792c-228">Creare lo schema del database iniziale.</span><span class="sxs-lookup"><span data-stu-id="5792c-228">Create the initial database schema.</span></span>
* <span data-ttu-id="5792c-229">Aggiornare in modo incrementale lo schema del database per mantenerlo sincronizzato con il modello di dati dell'applicazione.</span><span class="sxs-lookup"><span data-stu-id="5792c-229">Incrementally update the database schema to keep it in sync with the application's data model.</span></span>  <span data-ttu-id="5792c-230">Vengono conservati i dati esistenti nel database.</span><span class="sxs-lookup"><span data-stu-id="5792c-230">Existing data in the database is preserved.</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="5792c-231">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="5792c-231">Visual Studio</span></span>](#tab/visual-studio)

<span data-ttu-id="5792c-232">In questa sezione viene usata la finestra **console di gestione pacchetti** (PMC) per:</span><span class="sxs-lookup"><span data-stu-id="5792c-232">In this section, the **Package Manager Console** (PMC) window is used to:</span></span>

* <span data-ttu-id="5792c-233">Aggiungere una migrazione iniziale.</span><span class="sxs-lookup"><span data-stu-id="5792c-233">Add an initial migration.</span></span>
* <span data-ttu-id="5792c-234">Aggiornare il database con la migrazione iniziale.</span><span class="sxs-lookup"><span data-stu-id="5792c-234">Update the database with the initial migration.</span></span>

1. <span data-ttu-id="5792c-235">Nel menu **Strumenti** selezionare **Gestione pacchetti NuGet** > **Console di Gestione pacchetti**.</span><span class="sxs-lookup"><span data-stu-id="5792c-235">From the **Tools** menu, select **NuGet Package Manager** > **Package Manager Console**.</span></span>

   ![Menu della Console di Gestione pacchetti](model/_static/5/pmc.png)

1. <span data-ttu-id="5792c-237">Nella Console di Gestione pacchetti immettere i comandi seguenti:</span><span class="sxs-lookup"><span data-stu-id="5792c-237">In the PMC, enter the following commands:</span></span>

   ```powershell
   Add-Migration InitialCreate
   Update-Database
   ```

# <a name="visual-studio-code--visual-studio-for-mac"></a>[<span data-ttu-id="5792c-238">Visual Studio Code / Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="5792c-238">Visual Studio Code / Visual Studio for Mac</span></span>](#tab/visual-studio-code+visual-studio-mac)

[!INCLUDE [more information on the CLI for EF Core](~/includes/ef-cli.md)]

* <span data-ttu-id="5792c-239">Eseguire i seguenti comandi dell'interfaccia della riga di comando .NET:</span><span class="sxs-lookup"><span data-stu-id="5792c-239">Run the following .NET CLI commands:</span></span>

  ```dotnetcli
  dotnet ef migrations add InitialCreate
  dotnet ef database update
  ```

---

<span data-ttu-id="5792c-240">I comandi precedenti generano l'avviso seguente: "nessun tipo specificato per la colonna decimale ' Price ' nel tipo di entità' Movie '.</span><span class="sxs-lookup"><span data-stu-id="5792c-240">The preceding commands generate the following warning: "No type was specified for the decimal column 'Price' on entity type 'Movie'.</span></span> <span data-ttu-id="5792c-241">This will cause values to be silently truncated if they do not fit in the default precision and scale. (I valori saranno quindi automaticamente troncati se non rispettano la precisione e la scala predefinite).</span><span class="sxs-lookup"><span data-stu-id="5792c-241">This will cause values to be silently truncated if they do not fit in the default precision and scale.</span></span> <span data-ttu-id="5792c-242">Explicitly specify the SQL server column type that can accommodate all the values using 'HasColumnType()'. (Specificare in modo esplicito il tipo di colonna di SQL Server che può supportare tutti i valori usando 'HasColumnType()')"</span><span class="sxs-lookup"><span data-stu-id="5792c-242">Explicitly specify the SQL server column type that can accommodate all the values using 'HasColumnType()'."</span></span>

<span data-ttu-id="5792c-243">Ignorare l'avviso poiché verrà risolto in un passaggio successivo.</span><span class="sxs-lookup"><span data-stu-id="5792c-243">Ignore the warning, as it will be addressed in a later step.</span></span>

<span data-ttu-id="5792c-244">Il comando `migrations` genera un codice per creare lo schema del database iniziale.</span><span class="sxs-lookup"><span data-stu-id="5792c-244">The `migrations` command generates code to create the initial database schema.</span></span> <span data-ttu-id="5792c-245">Lo schema è basato sul modello specificato in `DbContext` .</span><span class="sxs-lookup"><span data-stu-id="5792c-245">The schema is based on the model specified in `DbContext`.</span></span> <span data-ttu-id="5792c-246">L'argomento `InitialCreate` viene usato per denominare le migrazioni.</span><span class="sxs-lookup"><span data-stu-id="5792c-246">The `InitialCreate` argument is used to name the migrations.</span></span> <span data-ttu-id="5792c-247">È possibile usare qualsiasi nome, ma per convenzione viene selezionato un nome che descrive la migrazione.</span><span class="sxs-lookup"><span data-stu-id="5792c-247">Any name can be used, but by convention a name is selected that describes the migration.</span></span>

<span data-ttu-id="5792c-248">Il `update` comando esegue il `Up` Metodo nelle migrazioni che non sono state applicate.</span><span class="sxs-lookup"><span data-stu-id="5792c-248">The `update` command runs the `Up` method in migrations that have not been applied.</span></span> <span data-ttu-id="5792c-249">In tal caso, `update` esegue il `Up` metodo nel file *migrations/ \<time-stamp> _InitialCreate. cs* , che crea il database.</span><span class="sxs-lookup"><span data-stu-id="5792c-249">In this case, `update` runs the `Up` method in the *Migrations/\<time-stamp>_InitialCreate.cs* file, which creates the database.</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="5792c-250">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="5792c-250">Visual Studio</span></span>](#tab/visual-studio)

### <a name="examine-the-context-registered-with-dependency-injection"></a><span data-ttu-id="5792c-251">Esaminare il contesto registrato con l'inserimento di dipendenze</span><span class="sxs-lookup"><span data-stu-id="5792c-251">Examine the context registered with dependency injection</span></span>

<span data-ttu-id="5792c-252">ASP.NET Core viene compilato tramite [dependency injection](xref:fundamentals/dependency-injection).</span><span class="sxs-lookup"><span data-stu-id="5792c-252">ASP.NET Core is built with [dependency injection](xref:fundamentals/dependency-injection).</span></span> <span data-ttu-id="5792c-253">I servizi, ad esempio il contesto di database EF Core, vengono registrati con l'inserimento delle dipendenze durante l'avvio dell'applicazione.</span><span class="sxs-lookup"><span data-stu-id="5792c-253">Services, such as the EF Core database context, are registered with dependency injection during application startup.</span></span> <span data-ttu-id="5792c-254">I componenti che richiedono questi servizi, ad esempio Razor le pagine, vengono forniti tramite i parametri del costruttore.</span><span class="sxs-lookup"><span data-stu-id="5792c-254">Components that require these services (such as Razor Pages) are provided these services via constructor parameters.</span></span> <span data-ttu-id="5792c-255">Più avanti nell'esercitazione viene illustrato il codice del costruttore che ottiene un'istanza del contesto di database.</span><span class="sxs-lookup"><span data-stu-id="5792c-255">The constructor code that gets a database context instance is shown later in the tutorial.</span></span>

<span data-ttu-id="5792c-256">Lo strumento di ponteggi crea automaticamente un contesto di database e lo registra con il contenitore di inserimento delle dipendenze.</span><span class="sxs-lookup"><span data-stu-id="5792c-256">The scaffolding tool automatically created a database context and registered it with the dependency injection container.</span></span>

<span data-ttu-id="5792c-257">Esaminare il metodo `Startup.ConfigureServices`.</span><span class="sxs-lookup"><span data-stu-id="5792c-257">Examine the `Startup.ConfigureServices` method.</span></span> <span data-ttu-id="5792c-258">La riga evidenziata è stata aggiunta dallo scaffolder:</span><span class="sxs-lookup"><span data-stu-id="5792c-258">The highlighted line was added by the scaffolder:</span></span>

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie30/Startup.cs?name=snippet_ConfigureServices&highlight=5-6)]

<span data-ttu-id="5792c-259">Le `RazorPagesMovieContext` coordinate EF Core funzionalità, ad esempio creazione, lettura, aggiornamento ed eliminazione, per il `Movie` modello.</span><span class="sxs-lookup"><span data-stu-id="5792c-259">The `RazorPagesMovieContext` coordinates EF Core functionality, such as Create, Read, Update and Delete, for the `Movie` model.</span></span> <span data-ttu-id="5792c-260">Il contesto dei dati (`RazorPagesMovieContext`) è derivato da [Microsoft.EntityFrameworkCore.DbContext](xref:Microsoft.EntityFrameworkCore.DbContext).</span><span class="sxs-lookup"><span data-stu-id="5792c-260">The data context (`RazorPagesMovieContext`) is derived from [Microsoft.EntityFrameworkCore.DbContext](xref:Microsoft.EntityFrameworkCore.DbContext).</span></span> <span data-ttu-id="5792c-261">Il contesto dei dati specifica le entità incluse nel modello di dati.</span><span class="sxs-lookup"><span data-stu-id="5792c-261">The data context specifies which entities are included in the data model.</span></span>

[!code-csharp[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie50/Data/RazorPagesMovieContext.cs)]

<span data-ttu-id="5792c-262">Il codice precedente crea una [proprietà \<Movie> DbSet](xref:Microsoft.EntityFrameworkCore.DbSet%601) per il set di entità.</span><span class="sxs-lookup"><span data-stu-id="5792c-262">The preceding code creates a [DbSet\<Movie>](xref:Microsoft.EntityFrameworkCore.DbSet%601) property for the entity set.</span></span> <span data-ttu-id="5792c-263">Nella terminologia di Entity Framework, un set di entità corrisponde in genere alla tabella di un database.</span><span class="sxs-lookup"><span data-stu-id="5792c-263">In Entity Framework terminology, an entity set typically corresponds to a database table.</span></span> <span data-ttu-id="5792c-264">Un'entità corrisponde a una riga nella tabella.</span><span class="sxs-lookup"><span data-stu-id="5792c-264">An entity corresponds to a row in the table.</span></span>

<span data-ttu-id="5792c-265">Il nome della stringa di connessione viene passato al contesto chiamando un metodo in un oggetto [DbContextOptions](xref:Microsoft.EntityFrameworkCore.DbContextOptions).</span><span class="sxs-lookup"><span data-stu-id="5792c-265">The name of the connection string is passed in to the context by calling a method on a [DbContextOptions](xref:Microsoft.EntityFrameworkCore.DbContextOptions) object.</span></span> <span data-ttu-id="5792c-266">Per lo sviluppo locale, il [sistema di configurazione](xref:fundamentals/configuration/index) legge la stringa di connessione dal *appsettings.json* file.</span><span class="sxs-lookup"><span data-stu-id="5792c-266">For local development, the [Configuration system](xref:fundamentals/configuration/index) reads the connection string from the *appsettings.json* file.</span></span>

# <a name="visual-studio-code--visual-studio-for-mac"></a>[<span data-ttu-id="5792c-267">Visual Studio Code / Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="5792c-267">Visual Studio Code / Visual Studio for Mac</span></span>](#tab/visual-studio-code+visual-studio-mac)

<span data-ttu-id="5792c-268">Esaminare il metodo `Up`.</span><span class="sxs-lookup"><span data-stu-id="5792c-268">Examine the `Up` method.</span></span>

---

<a name="test"></a>

## <a name="test-the-app"></a><span data-ttu-id="5792c-269">Testare l'app</span><span class="sxs-lookup"><span data-stu-id="5792c-269">Test the app</span></span>

1. <span data-ttu-id="5792c-270">Eseguire l'app e accodare `/Movies` all'URL nel browser (`http://localhost:port/movies`).</span><span class="sxs-lookup"><span data-stu-id="5792c-270">Run the app and append `/Movies` to the URL in the browser (`http://localhost:port/movies`).</span></span>

   <span data-ttu-id="5792c-271">Se si riceve un messaggio di errore che indica che</span><span class="sxs-lookup"><span data-stu-id="5792c-271">If you receive the following error:</span></span>

   ```console
   SqlException: Cannot open database "RazorPagesMovieContext-GUID" requested by the login. The login failed.
   Login failed for user 'User-name'.
   ```

   <span data-ttu-id="5792c-272">Non è stato eseguita la [migrazione](#pmc).</span><span class="sxs-lookup"><span data-stu-id="5792c-272">You missed the [migrations step](#pmc).</span></span>

1. <span data-ttu-id="5792c-273">Eseguire il test del collegamento **Crea**.</span><span class="sxs-lookup"><span data-stu-id="5792c-273">Test the **Create** link.</span></span>

   ![Pagina di creazione](model/_static/conan5.png)

   > [!NOTE]
   > <span data-ttu-id="5792c-275">Potrebbe non essere possibile immettere virgole decimali nel campo `Price`.</span><span class="sxs-lookup"><span data-stu-id="5792c-275">You may not be able to enter decimal commas in the `Price` field.</span></span> <span data-ttu-id="5792c-276">Per supportare la [convalida jQuery](https://jqueryvalidation.org/) per impostazioni locali diverse dall'inglese che usano la virgola (",") come separatore decimale e per formati di data diversi da quello dell'inglese (Stati Uniti), è necessario localizzare l'app.</span><span class="sxs-lookup"><span data-stu-id="5792c-276">To support [jQuery validation](https://jqueryvalidation.org/) for non-English locales that use a comma (",") for a decimal point and for non US-English date formats, the app must be globalized.</span></span> <span data-ttu-id="5792c-277">Per istruzioni sulla globalizzazione, vedere [questo problema su GitHub](https://github.com/dotnet/AspNetCore.Docs/issues/4076#issuecomment-326590420).</span><span class="sxs-lookup"><span data-stu-id="5792c-277">For globalization instructions, see [this GitHub issue](https://github.com/dotnet/AspNetCore.Docs/issues/4076#issuecomment-326590420).</span></span>

1. <span data-ttu-id="5792c-278">Testare i collegamenti **Modifica**, **Dettagli** ed **Elimina**.</span><span class="sxs-lookup"><span data-stu-id="5792c-278">Test the **Edit**, **Details**, and **Delete** links.</span></span>

<span data-ttu-id="5792c-279">L'esercitazione successiva illustra i file creati tramite scaffolding.</span><span class="sxs-lookup"><span data-stu-id="5792c-279">The next tutorial explains the files created by scaffolding.</span></span>

## <a name="additional-resources"></a><span data-ttu-id="5792c-280">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="5792c-280">Additional resources</span></span>

> [!div class="step-by-step"]
> <span data-ttu-id="5792c-281">[Precedente: introduzione](xref:tutorials/razor-pages/razor-pages-start) 
>  Passaggio [successivo: impalcatura Razor Pagine](xref:tutorials/razor-pages/page) di</span><span class="sxs-lookup"><span data-stu-id="5792c-281">[Previous: Get Started](xref:tutorials/razor-pages/razor-pages-start)
[Next: Scaffolded Razor Pages](xref:tutorials/razor-pages/page)</span></span>

::: moniker-end

<!--  ::: End of >= 5.0 moniker version   -->

::: moniker range=">= aspnetcore-3.0 < aspnetcore-5.0"

<!-- In the next update on the CLI version, let the scaffolder do the same work the VS driven scaffolder does. That is, create the DB context, etc -->

<span data-ttu-id="5792c-282">In questa sezione vengono aggiunte le classi per la gestione dei filmati.</span><span class="sxs-lookup"><span data-stu-id="5792c-282">In this section, classes are added for managing movies.</span></span> <span data-ttu-id="5792c-283">Le classi modello dell'app usano [Entity Framework Core (EF Core)](/ef/core) per lavorare con il database.</span><span class="sxs-lookup"><span data-stu-id="5792c-283">The app's model classes use [Entity Framework Core (EF Core)](/ef/core) to work with the database.</span></span> <span data-ttu-id="5792c-284">EF Core è un mapper relazionale a oggetti (O/RM) che semplifica l'accesso ai dati.</span><span class="sxs-lookup"><span data-stu-id="5792c-284">EF Core is an object-relational mapper (O/RM) that simplifies data access.</span></span>

<span data-ttu-id="5792c-285">Le classi di modello sono dette classi POCO (da "plain-old CLR objects") perché non hanno alcuna dipendenza in EF Core.</span><span class="sxs-lookup"><span data-stu-id="5792c-285">The model classes are known as POCO classes (from "plain-old CLR objects") because they don't have any dependency on EF Core.</span></span> <span data-ttu-id="5792c-286">Definiscono le proprietà dei dati archiviati nel database.</span><span class="sxs-lookup"><span data-stu-id="5792c-286">They define the properties of the data that are stored in the database.</span></span>

<span data-ttu-id="5792c-287">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie30) ([procedura per il download](xref:index#how-to-download-a-sample)).</span><span class="sxs-lookup"><span data-stu-id="5792c-287">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie30) ([how to download](xref:index#how-to-download-a-sample)).</span></span>

## <a name="add-a-data-model"></a><span data-ttu-id="5792c-288">Aggiungere un modello di dati</span><span class="sxs-lookup"><span data-stu-id="5792c-288">Add a data model</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="5792c-289">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="5792c-289">Visual Studio</span></span>](#tab/visual-studio)

<span data-ttu-id="5792c-290">Fare clic con il pulsante destro del mouse sul progetto **Razor PagesMovie** > **Aggiungi**  >  **nuova cartella**.</span><span class="sxs-lookup"><span data-stu-id="5792c-290">Right-click the **RazorPagesMovie** project > **Add** > **New Folder**.</span></span> <span data-ttu-id="5792c-291">Assegnare il nome *Modelli* alla cartella.</span><span class="sxs-lookup"><span data-stu-id="5792c-291">Name the folder *Models*.</span></span>

<span data-ttu-id="5792c-292">Fare clic con il pulsante destro del mouse sulla cartella *modelli* .</span><span class="sxs-lookup"><span data-stu-id="5792c-292">Right-click the *Models* folder.</span></span> <span data-ttu-id="5792c-293">Selezionare **Aggiungi**  >  **classe**.</span><span class="sxs-lookup"><span data-stu-id="5792c-293">Select **Add** > **Class**.</span></span> <span data-ttu-id="5792c-294">Denominare la classe **Movie**.</span><span class="sxs-lookup"><span data-stu-id="5792c-294">Name the class **Movie**.</span></span>

<span data-ttu-id="5792c-295">Aggiungere le proprietà seguenti alla classe `Movie`:</span><span class="sxs-lookup"><span data-stu-id="5792c-295">Add the following properties to the `Movie` class:</span></span>

[!code-csharp[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie22/Models/Movie.cs?name=snippet1)]

<span data-ttu-id="5792c-296">La classe `Movie` contiene:</span><span class="sxs-lookup"><span data-stu-id="5792c-296">The `Movie` class contains:</span></span>

* <span data-ttu-id="5792c-297">Il campo `ID` è richiesto dal database per la chiave primaria.</span><span class="sxs-lookup"><span data-stu-id="5792c-297">The `ID` field is required by the database for the primary key.</span></span>
* <span data-ttu-id="5792c-298">`[DataType(DataType.Date)]`: L'attributo [DataType](xref:System.ComponentModel.DataAnnotations.DataTypeAttribute) specifica il tipo di dati ( `Date` ).</span><span class="sxs-lookup"><span data-stu-id="5792c-298">`[DataType(DataType.Date)]`: The [DataType](xref:System.ComponentModel.DataAnnotations.DataTypeAttribute) attribute specifies the type of the data (`Date`).</span></span> <span data-ttu-id="5792c-299">Con questo attributo:</span><span class="sxs-lookup"><span data-stu-id="5792c-299">With this attribute:</span></span>

  * <span data-ttu-id="5792c-300">l'utente non deve immettere le informazioni temporali nel campo della data.</span><span class="sxs-lookup"><span data-stu-id="5792c-300">The user is not required to enter time information in the date field.</span></span>
  * <span data-ttu-id="5792c-301">Viene visualizzata solo la data, non le informazioni temporali.</span><span class="sxs-lookup"><span data-stu-id="5792c-301">Only the date is displayed, not time information.</span></span>

<span data-ttu-id="5792c-302">L'attributo [DataAnnotations](xref:System.ComponentModel.DataAnnotations) viene analizzato in un'esercitazione successiva.</span><span class="sxs-lookup"><span data-stu-id="5792c-302">[DataAnnotations](xref:System.ComponentModel.DataAnnotations) are covered in a later tutorial.</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="5792c-303">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="5792c-303">Visual Studio Code</span></span>](#tab/visual-studio-code)

* <span data-ttu-id="5792c-304">Aggiungere una cartella denominata *Modelli*.</span><span class="sxs-lookup"><span data-stu-id="5792c-304">Add a folder named *Models*.</span></span>
* <span data-ttu-id="5792c-305">Aggiungere una classe alla cartella *Modello* denominata *Movie.cs*.</span><span class="sxs-lookup"><span data-stu-id="5792c-305">Add a class to the *Models* folder named *Movie.cs*.</span></span>

<span data-ttu-id="5792c-306">Aggiungere le proprietà seguenti alla classe `Movie`:</span><span class="sxs-lookup"><span data-stu-id="5792c-306">Add the following properties to the `Movie` class:</span></span>

[!code-csharp[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie22/Models/Movie.cs?name=snippet1)]

<span data-ttu-id="5792c-307">La classe `Movie` contiene:</span><span class="sxs-lookup"><span data-stu-id="5792c-307">The `Movie` class contains:</span></span>

* <span data-ttu-id="5792c-308">Il campo `ID` è richiesto dal database per la chiave primaria.</span><span class="sxs-lookup"><span data-stu-id="5792c-308">The `ID` field is required by the database for the primary key.</span></span>
* <span data-ttu-id="5792c-309">`[DataType(DataType.Date)]`: L'attributo [DataType](xref:System.ComponentModel.DataAnnotations.DataTypeAttribute) specifica il tipo di dati ( `Date` ).</span><span class="sxs-lookup"><span data-stu-id="5792c-309">`[DataType(DataType.Date)]`: The [DataType](xref:System.ComponentModel.DataAnnotations.DataTypeAttribute) attribute specifies the type of the data (`Date`).</span></span> <span data-ttu-id="5792c-310">Con questo attributo:</span><span class="sxs-lookup"><span data-stu-id="5792c-310">With this attribute:</span></span>

  * <span data-ttu-id="5792c-311">l'utente non deve immettere le informazioni temporali nel campo della data.</span><span class="sxs-lookup"><span data-stu-id="5792c-311">The user is not required to enter time information in the date field.</span></span>
  * <span data-ttu-id="5792c-312">Viene visualizzata solo la data, non le informazioni temporali.</span><span class="sxs-lookup"><span data-stu-id="5792c-312">Only the date is displayed, not time information.</span></span>

<span data-ttu-id="5792c-313">L'attributo [DataAnnotations](xref:System.ComponentModel.DataAnnotations) viene analizzato in un'esercitazione successiva.</span><span class="sxs-lookup"><span data-stu-id="5792c-313">[DataAnnotations](xref:System.ComponentModel.DataAnnotations) are covered in a later tutorial.</span></span>

<a name="dc"></a>

### <a name="add-nuget-packages-and-ef-tools"></a><span data-ttu-id="5792c-314">Aggiungere i pacchetti NuGet e gli strumenti EF</span><span class="sxs-lookup"><span data-stu-id="5792c-314">Add NuGet packages and EF tools</span></span>

[!INCLUDE[](~/includes/add-EF-NuGet-SQLite-CLI.md)]

### <a name="add-a-database-context-class"></a><span data-ttu-id="5792c-315">Aggiungere una classe di contesto dei dati</span><span class="sxs-lookup"><span data-stu-id="5792c-315">Add a database context class</span></span>

* <span data-ttu-id="5792c-316">Nel progetto *Razor PagesMovie* creare una nuova cartella denominata *Data*.</span><span class="sxs-lookup"><span data-stu-id="5792c-316">In the *RazorPagesMovie* project, create a new folder named *Data*.</span></span>
* <span data-ttu-id="5792c-317">Aggiungere la classe `RazorPagesMovieContext` seguente alla cartella *Data*:</span><span class="sxs-lookup"><span data-stu-id="5792c-317">Add the following `RazorPagesMovieContext` class to the *Data* folder:</span></span>

  [!code-csharp[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie30/Data/RazorPagesMovieContext.cs)]

<span data-ttu-id="5792c-318">Il codice precedente crea una proprietà `DbSet` per il set di entità.</span><span class="sxs-lookup"><span data-stu-id="5792c-318">The preceding code creates a `DbSet` property for the entity set.</span></span> <span data-ttu-id="5792c-319">Nella terminologia di Entity Framework, un set di entità corrisponde in genere alla tabella di un database e un'entità corrisponde a una riga della tabella.</span><span class="sxs-lookup"><span data-stu-id="5792c-319">In Entity Framework terminology, an entity set typically corresponds to a database table, and an entity corresponds to a row in the table.</span></span> <span data-ttu-id="5792c-320">Il codice non verrà compilato fino a quando le dipendenze non verranno aggiunte in un passaggio successivo.</span><span class="sxs-lookup"><span data-stu-id="5792c-320">The code won't compile until dependencies are added in a later step.</span></span>

<a name="cs"></a>

### <a name="add-a-database-connection-string"></a><span data-ttu-id="5792c-321">Aggiungere una stringa di connessione del database</span><span class="sxs-lookup"><span data-stu-id="5792c-321">Add a database connection string</span></span>

<span data-ttu-id="5792c-322">Aggiungere una stringa di connessione al *appsettings.json* file come illustrato nel codice evidenziato seguente:</span><span class="sxs-lookup"><span data-stu-id="5792c-322">Add a connection string to the *appsettings.json* file as shown in the following highlighted code:</span></span>

[!code-json[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie30/appsettings_SQLite.json?highlight=10-12)]

<a name="reg"></a>

### <a name="register-the-database-context"></a><span data-ttu-id="5792c-323">Registrare il contesto del database</span><span class="sxs-lookup"><span data-stu-id="5792c-323">Register the database context</span></span>

<span data-ttu-id="5792c-324">Aggiungere le istruzioni `using` seguenti all'inizio di *Startup.cs*:</span><span class="sxs-lookup"><span data-stu-id="5792c-324">Add the following `using` statements at the top of *Startup.cs*:</span></span>

```csharp
using RazorPagesMovie.Data;
using Microsoft.EntityFrameworkCore;
```

<span data-ttu-id="5792c-325">Registrare il contesto del database con il contenitore di [inserimento delle dipendenze](xref:fundamentals/dependency-injection) in `Startup.ConfigureServices`.</span><span class="sxs-lookup"><span data-stu-id="5792c-325">Register the database context with the [dependency injection](xref:fundamentals/dependency-injection) container in `Startup.ConfigureServices`.</span></span>

[!code-csharp[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie30/Startup.cs?name=snippet_UseSqlite&highlight=5-6)]

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="5792c-326">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="5792c-326">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

* <span data-ttu-id="5792c-327">Nella **finestra degli strumenti della soluzione**, fare clic sul progetto **Razor PagesMovie** e quindi selezionare **Aggiungi** > **nuova cartella.** Assegnare un nome ai *modelli* di cartella.</span><span class="sxs-lookup"><span data-stu-id="5792c-327">In the **Solution Tool Window**, control-click the **RazorPagesMovie** project, and then select **Add** > **New Folder...**. Name the folder *Models*.</span></span>
* <span data-ttu-id="5792c-328">Fare clic con il pulsante destro del mouse sulla cartella *Models* , quindi scegliere **Aggiungi** > **nuovo file**.</span><span class="sxs-lookup"><span data-stu-id="5792c-328">Right-click the *Models* folder, and then select **Add** > **New File...**.</span></span>
* <span data-ttu-id="5792c-329">Nel finestra di dialogo **Nuovo file**:</span><span class="sxs-lookup"><span data-stu-id="5792c-329">In the **New File** dialog:</span></span>

  * <span data-ttu-id="5792c-330">Selezionare **Generale** nel riquadro a sinistra.</span><span class="sxs-lookup"><span data-stu-id="5792c-330">Select **General** in the left pane.</span></span>
  * <span data-ttu-id="5792c-331">Selezionare **Classe vuota** nel riquadro centrale.</span><span class="sxs-lookup"><span data-stu-id="5792c-331">Select **Empty Class** in the center pane.</span></span>
  * <span data-ttu-id="5792c-332">Denominare la classe **Filmato** e selezionare **Nuovo**.</span><span class="sxs-lookup"><span data-stu-id="5792c-332">Name the class **Movie** and select **New**.</span></span>

<span data-ttu-id="5792c-333">Aggiungere le proprietà seguenti alla classe `Movie`:</span><span class="sxs-lookup"><span data-stu-id="5792c-333">Add the following properties to the `Movie` class:</span></span>

[!code-csharp[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie22/Models/Movie.cs?name=snippet1)]

<span data-ttu-id="5792c-334">La classe `Movie` contiene:</span><span class="sxs-lookup"><span data-stu-id="5792c-334">The `Movie` class contains:</span></span>

* <span data-ttu-id="5792c-335">Il campo `ID` è richiesto dal database per la chiave primaria.</span><span class="sxs-lookup"><span data-stu-id="5792c-335">The `ID` field is required by the database for the primary key.</span></span>
* <span data-ttu-id="5792c-336">`[DataType(DataType.Date)]`: L'attributo [DataType](xref:System.ComponentModel.DataAnnotations.DataTypeAttribute) specifica il tipo di dati ( `Date` ).</span><span class="sxs-lookup"><span data-stu-id="5792c-336">`[DataType(DataType.Date)]`: The [DataType](xref:System.ComponentModel.DataAnnotations.DataTypeAttribute) attribute specifies the type of the data (`Date`).</span></span> <span data-ttu-id="5792c-337">Con questo attributo:</span><span class="sxs-lookup"><span data-stu-id="5792c-337">With this attribute:</span></span>

  * <span data-ttu-id="5792c-338">l'utente non deve immettere le informazioni temporali nel campo della data.</span><span class="sxs-lookup"><span data-stu-id="5792c-338">The user is not required to enter time information in the date field.</span></span>
  * <span data-ttu-id="5792c-339">Viene visualizzata solo la data, non le informazioni temporali.</span><span class="sxs-lookup"><span data-stu-id="5792c-339">Only the date is displayed, not time information.</span></span>

---

<span data-ttu-id="5792c-340">L'attributo [DataAnnotations](xref:System.ComponentModel.DataAnnotations) viene analizzato in un'esercitazione successiva.</span><span class="sxs-lookup"><span data-stu-id="5792c-340">[DataAnnotations](xref:System.ComponentModel.DataAnnotations) are covered in a later tutorial.</span></span>

<span data-ttu-id="5792c-341">Compilare il progetto per verificare che non siano presenti errori di compilazione.</span><span class="sxs-lookup"><span data-stu-id="5792c-341">Build the project to verify there are no compilation errors.</span></span>

## <a name="scaffold-the-movie-model"></a><span data-ttu-id="5792c-342">Eseguire lo scaffolding del modello di filmato</span><span class="sxs-lookup"><span data-stu-id="5792c-342">Scaffold the movie model</span></span>

<span data-ttu-id="5792c-343">In questa sezione viene eseguito lo scaffolding del modello *Movie*.</span><span class="sxs-lookup"><span data-stu-id="5792c-343">In this section, the movie model is scaffolded.</span></span> <span data-ttu-id="5792c-344">Lo strumento di scaffolding crea quindi le pagine per le operazioni CRUD (creazione, lettura, aggiornamento ed eliminazione) per il modello *Movie*.</span><span class="sxs-lookup"><span data-stu-id="5792c-344">That is, the scaffolding tool produces pages for Create, Read, Update, and Delete (CRUD) operations for the movie model.</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="5792c-345">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="5792c-345">Visual Studio</span></span>](#tab/visual-studio)

<span data-ttu-id="5792c-346">Creare una cartella *Pages/Movies*:</span><span class="sxs-lookup"><span data-stu-id="5792c-346">Create a *Pages/Movies* folder:</span></span>

* <span data-ttu-id="5792c-347">Fare clic con il pulsante destro del mouse sulla cartella *pagine* > **Aggiungi** > **nuova cartella**.</span><span class="sxs-lookup"><span data-stu-id="5792c-347">Right-click on the *Pages* folder > **Add** > **New Folder**.</span></span>
* <span data-ttu-id="5792c-348">Denominare la cartella *Movies*.</span><span class="sxs-lookup"><span data-stu-id="5792c-348">Name the folder *Movies*.</span></span>

<span data-ttu-id="5792c-349">Fare clic con il pulsante destro del mouse sulla cartella *pages/Movies* > **Aggiungi** > **nuovo elemento con impalcatura**.</span><span class="sxs-lookup"><span data-stu-id="5792c-349">Right-click on the *Pages/Movies* folder > **Add** > **New Scaffolded Item**.</span></span>

![Immagine relativa alle istruzioni precedenti.](model/_static/sca.png)

<span data-ttu-id="5792c-351">Nella finestra di dialogo **Aggiungi impalcatura** selezionare **Razor pages using Entity Framework (CRUD)** > **Add**.</span><span class="sxs-lookup"><span data-stu-id="5792c-351">In the **Add Scaffold** dialog, select **Razor Pages using Entity Framework (CRUD)** > **Add**.</span></span>

![Immagine relativa alle istruzioni precedenti.](model/_static/add_scaffold.png)

<span data-ttu-id="5792c-353">Completare la finestra di dialogo **Aggiungi Razor pagine con Entity Framework (CRUD)** :</span><span class="sxs-lookup"><span data-stu-id="5792c-353">Complete the **Add Razor Pages using Entity Framework (CRUD)** dialog:</span></span>

* <span data-ttu-id="5792c-354">Nell'elenco a discesa **classe modello** selezionare **Movie ( Razor PagesMovie. Models)**.</span><span class="sxs-lookup"><span data-stu-id="5792c-354">In the **Model class** drop down, select **Movie (RazorPagesMovie.Models)**.</span></span>
* <span data-ttu-id="5792c-355">Nella riga della **classe del contesto dati** selezionare il **+** segno (segno più) e modificare il nome generato da Razor PagesMovie.**Modelli**. Razor Da PagesMovieContext a Razor PagesMovie.**Dati**. Razor PagesMovieContext.</span><span class="sxs-lookup"><span data-stu-id="5792c-355">In the **Data context class** row, select the **+** (plus) sign and change the generated name from RazorPagesMovie.**Models**.RazorPagesMovieContext to RazorPagesMovie.**Data**.RazorPagesMovieContext.</span></span> <span data-ttu-id="5792c-356">[Questa modifica](https://developercommunity.visualstudio.com/content/problem/652166/aspnet-core-ef-scaffolder-uses-incorrect-namespace.html) non è obbligatoria.</span><span class="sxs-lookup"><span data-stu-id="5792c-356">[This change](https://developercommunity.visualstudio.com/content/problem/652166/aspnet-core-ef-scaffolder-uses-incorrect-namespace.html) is not required.</span></span> <span data-ttu-id="5792c-357">Crea la classe del contesto di database con lo spazio dei nomi corretto.</span><span class="sxs-lookup"><span data-stu-id="5792c-357">It creates the database context class with the correct namespace.</span></span>
* <span data-ttu-id="5792c-358">Selezionare **Aggiungi**.</span><span class="sxs-lookup"><span data-stu-id="5792c-358">Select **Add**.</span></span>

![Immagine relativa alle istruzioni precedenti.](model/_static/3/arp.png)

<span data-ttu-id="5792c-360">Il *appsettings.json* file viene aggiornato con la stringa di connessione utilizzata per connettersi a un database locale.</span><span class="sxs-lookup"><span data-stu-id="5792c-360">The *appsettings.json* file is updated with the connection string used to connect to a local database.</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="5792c-361">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="5792c-361">Visual Studio Code</span></span>](#tab/visual-studio-code)

<!--  Until https://github.com/aspnet/Scaffolding/issues/582 is fixed windows needs backslash or the namespace is namespace RazorPagesMovie.Pages_Movies rather than namespace RazorPagesMovie.Pages.Movies
-->

* <span data-ttu-id="5792c-362">Aprire una finestra di comando nella directory del progetto che contiene i file *Program.cs*, *Startup.cs* e *. csproj* .</span><span class="sxs-lookup"><span data-stu-id="5792c-362">Open a command window in the project directory, which contains the *Program.cs*, *Startup.cs*, and *.csproj* files.</span></span>

* <span data-ttu-id="5792c-363">**Per Windows**: eseguire il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="5792c-363">**For Windows**: Run the following command:</span></span>

  ```dotnetcli
  dotnet-aspnet-codegenerator razorpage -m Movie -dc RazorPagesMovieContext -udl -outDir Pages\Movies --referenceScriptLibraries
  ```

* <span data-ttu-id="5792c-364">**Per MacOS e Linux**: eseguire il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="5792c-364">**For macOS and Linux**: Run the following command:</span></span>

  ```dotnetcli
  dotnet-aspnet-codegenerator razorpage -m Movie -dc RazorPagesMovieContext -udl -outDir Pages/Movies --referenceScriptLibraries
  ```

<a name="codegenerator"></a> <span data-ttu-id="5792c-365">La tabella seguente illustra in dettaglio le opzioni del generatore di codice ASP.NET Core:</span><span class="sxs-lookup"><span data-stu-id="5792c-365">The following table details the ASP.NET Core code generator options:</span></span>

| <span data-ttu-id="5792c-366">Opzione</span><span class="sxs-lookup"><span data-stu-id="5792c-366">Option</span></span>               | <span data-ttu-id="5792c-367">Descrizione</span><span class="sxs-lookup"><span data-stu-id="5792c-367">Description</span></span>|
| ----------------- | ------------ |
| `-m`  | <span data-ttu-id="5792c-368">Nome del modello.</span><span class="sxs-lookup"><span data-stu-id="5792c-368">The name of the model.</span></span> |
| `-dc`  | <span data-ttu-id="5792c-369">Classe `DbContext` da usare.</span><span class="sxs-lookup"><span data-stu-id="5792c-369">The `DbContext` class to use.</span></span> |
| `-udl` | <span data-ttu-id="5792c-370">Uso del layout predefinito.</span><span class="sxs-lookup"><span data-stu-id="5792c-370">Use the default layout.</span></span> |
| `-outDir` | <span data-ttu-id="5792c-371">Percorso relativo della cartella di output per creare le viste.</span><span class="sxs-lookup"><span data-stu-id="5792c-371">The relative output folder path to create the views.</span></span> |
| `--referenceScriptLibraries` | <span data-ttu-id="5792c-372">Aggiunge `_ValidationScriptsPartial` per modificare e creare pagine</span><span class="sxs-lookup"><span data-stu-id="5792c-372">Adds `_ValidationScriptsPartial` to Edit and Create pages</span></span> |

<span data-ttu-id="5792c-373">Usare l' `-h` opzione per ottenere informazioni sul `aspnet-codegenerator razorpage` comando:</span><span class="sxs-lookup"><span data-stu-id="5792c-373">Use the `-h` option to get help on the `aspnet-codegenerator razorpage` command:</span></span>

```dotnetcli
dotnet-aspnet-codegenerator razorpage -h
```

<span data-ttu-id="5792c-374">Per ulteriori informazioni, vedere [DotNet-ASPNET-CodeGenerator](xref:fundamentals/tools/dotnet-aspnet-codegenerator).</span><span class="sxs-lookup"><span data-stu-id="5792c-374">For more information, see [dotnet-aspnet-codegenerator](xref:fundamentals/tools/dotnet-aspnet-codegenerator).</span></span>

### <a name="use-sqlite-for-development-sql-server-for-production"></a><span data-ttu-id="5792c-375">Usare SQLite per lo sviluppo, SQL Server per la produzione</span><span class="sxs-lookup"><span data-stu-id="5792c-375">Use SQLite for development, SQL Server for production</span></span>

<span data-ttu-id="5792c-376">Quando si seleziona SQLite, il codice generato dal modello è pronto per lo sviluppo.</span><span class="sxs-lookup"><span data-stu-id="5792c-376">When SQLite is selected, the template generated code is ready for development.</span></span> <span data-ttu-id="5792c-377">Il codice seguente illustra come inserire all' <xref:Microsoft.AspNetCore.Hosting.IWebHostEnvironment> avvio.</span><span class="sxs-lookup"><span data-stu-id="5792c-377">The following code shows how to inject <xref:Microsoft.AspNetCore.Hosting.IWebHostEnvironment> into Startup.</span></span> <span data-ttu-id="5792c-378">`IWebHostEnvironment` viene inserito in modo `ConfigureServices` che possa usare SQLite per lo sviluppo e la SQL Server nell'ambiente di produzione.</span><span class="sxs-lookup"><span data-stu-id="5792c-378">`IWebHostEnvironment` is injected so `ConfigureServices` can use SQLite in development and SQL Server in production.</span></span>

[!code-csharp[](~/includes/RP/code/StartupDevProd.cs?name=snippet&highlight=5,10,14)]

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="5792c-379">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="5792c-379">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

<span data-ttu-id="5792c-380">Creare una cartella *Pages/Movies*:</span><span class="sxs-lookup"><span data-stu-id="5792c-380">Create a *Pages/Movies* folder:</span></span>

* <span data-ttu-id="5792c-381">Fare clic con il pulsante destro del mouse sulla cartella *pagine* > **Aggiungi** > **nuova cartella**.</span><span class="sxs-lookup"><span data-stu-id="5792c-381">Right-click on the *Pages* folder > **Add** > **New Folder**.</span></span>
* <span data-ttu-id="5792c-382">Denominare la cartella *Movies*.</span><span class="sxs-lookup"><span data-stu-id="5792c-382">Name the folder *Movies*.</span></span>

<span data-ttu-id="5792c-383">Fare clic con il pulsante destro del mouse sulla cartella *pages/Movies* > **Aggiungi** > **Nuova impalcatura...**.</span><span class="sxs-lookup"><span data-stu-id="5792c-383">Right-click on the *Pages/Movies* folder > **Add** > **New Scaffolding...**.</span></span>

![Immagine relativa alle istruzioni precedenti.](model/_static/scaMac.png)

<span data-ttu-id="5792c-385">Nella finestra di dialogo **Nuova impalcatura** selezionare **Razor pagine con Entity Framework (CRUD)** > **Avanti**.</span><span class="sxs-lookup"><span data-stu-id="5792c-385">In the **New Scaffolding** dialog, select **Razor Pages using Entity Framework (CRUD)** > **Next**.</span></span>

![Immagine relativa alle istruzioni precedenti.](model/_static/add_scaffoldMac.png)

<span data-ttu-id="5792c-387">Completare la finestra di dialogo **Aggiungi Razor pagine con Entity Framework (CRUD)** :</span><span class="sxs-lookup"><span data-stu-id="5792c-387">Complete the **Add Razor Pages using Entity Framework (CRUD)** dialog:</span></span>

* <span data-ttu-id="5792c-388">Nell'elenco a discesa **classe modello** selezionare o digitare **Movie ( Razor PagesMovie. Models)**.</span><span class="sxs-lookup"><span data-stu-id="5792c-388">In the **Model class** drop down, select, or type, **Movie (RazorPagesMovie.Models)**.</span></span>
* <span data-ttu-id="5792c-389">Nella riga della **classe del contesto dati** Digitare il nome della nuova classe, Razor PagesMovie.**Dati**. Razor PagesMovieContext.</span><span class="sxs-lookup"><span data-stu-id="5792c-389">In the **Data context class** row, type the name for the new class, RazorPagesMovie.**Data**.RazorPagesMovieContext.</span></span> <span data-ttu-id="5792c-390">[Questa modifica](https://developercommunity.visualstudio.com/content/problem/652166/aspnet-core-ef-scaffolder-uses-incorrect-namespace.html) non è obbligatoria.</span><span class="sxs-lookup"><span data-stu-id="5792c-390">[This change](https://developercommunity.visualstudio.com/content/problem/652166/aspnet-core-ef-scaffolder-uses-incorrect-namespace.html) is not required.</span></span> <span data-ttu-id="5792c-391">Crea la classe del contesto di database con lo spazio dei nomi corretto.</span><span class="sxs-lookup"><span data-stu-id="5792c-391">It creates the database context class with the correct namespace.</span></span>
* <span data-ttu-id="5792c-392">Selezionare **Aggiungi**.</span><span class="sxs-lookup"><span data-stu-id="5792c-392">Select **Add**.</span></span>

![Immagine relativa alle istruzioni precedenti.](model/_static/arpMac.png)

<span data-ttu-id="5792c-394">Il *appsettings.json* file viene aggiornato con la stringa di connessione utilizzata per connettersi a un database locale.</span><span class="sxs-lookup"><span data-stu-id="5792c-394">The *appsettings.json* file is updated with the connection string used to connect to a local database.</span></span>

### <a name="add-ef-tools"></a><span data-ttu-id="5792c-395">Aggiungi strumenti EF</span><span class="sxs-lookup"><span data-stu-id="5792c-395">Add EF tools</span></span>

<span data-ttu-id="5792c-396">Eseguire il comando interfaccia della riga di comando di .NET Core seguente:</span><span class="sxs-lookup"><span data-stu-id="5792c-396">Run the following .NET Core CLI command:</span></span>

```dotnetcli
dotnet tool install --global dotnet-ef
```

<span data-ttu-id="5792c-397">Il comando precedente aggiunge gli strumenti Entity Framework Core per l'interfaccia della riga di comando di .NET Core.</span><span class="sxs-lookup"><span data-stu-id="5792c-397">The preceding command adds the Entity Framework Core Tools for the .NET Core CLI.</span></span> <span data-ttu-id="5792c-398">Per ulteriori informazioni, vedere [riferimento agli strumenti Entity Framework Core-interfaccia della riga di comando di .NET Core](/ef/core/miscellaneous/cli/dotnet).</span><span class="sxs-lookup"><span data-stu-id="5792c-398">For more information, see [Entity Framework Core tools reference - .NET Core CLI](/ef/core/miscellaneous/cli/dotnet).</span></span>

### <a name="use-sqlite-for-development-sql-server-for-production"></a><span data-ttu-id="5792c-399">Usare SQLite per lo sviluppo, SQL Server per la produzione</span><span class="sxs-lookup"><span data-stu-id="5792c-399">Use SQLite for development, SQL Server for production</span></span>

<span data-ttu-id="5792c-400">Quando si seleziona SQLite, il codice generato dal modello è pronto per lo sviluppo.</span><span class="sxs-lookup"><span data-stu-id="5792c-400">When SQLite is selected, the template generated code is ready for development.</span></span> <span data-ttu-id="5792c-401">Il codice seguente illustra come inserire all' <xref:Microsoft.AspNetCore.Hosting.IWebHostEnvironment> avvio.</span><span class="sxs-lookup"><span data-stu-id="5792c-401">The following code shows how to inject <xref:Microsoft.AspNetCore.Hosting.IWebHostEnvironment> into Startup.</span></span> <span data-ttu-id="5792c-402">`IWebHostEnvironment` viene inserito in modo `ConfigureServices` che possa usare SQLite per lo sviluppo e la SQL Server nell'ambiente di produzione.</span><span class="sxs-lookup"><span data-stu-id="5792c-402">`IWebHostEnvironment` is injected so `ConfigureServices` can use SQLite in development and SQL Server in production.</span></span>

[!code-csharp[](~/includes/RP/code/StartupDevProd.cs?name=snippet&highlight=5,10,14)]

---

### <a name="files-created"></a><span data-ttu-id="5792c-403">File creati</span><span class="sxs-lookup"><span data-stu-id="5792c-403">Files created</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="5792c-404">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="5792c-404">Visual Studio</span></span>](#tab/visual-studio)

<span data-ttu-id="5792c-405">Il processo di scaffolding crea e aggiorna i file seguenti:</span><span class="sxs-lookup"><span data-stu-id="5792c-405">The scaffold process creates and updates the following files:</span></span>

* <span data-ttu-id="5792c-406">*Pagine/filmati*: creare, eliminare, Details, Edit e Index .</span><span class="sxs-lookup"><span data-stu-id="5792c-406">*Pages/Movies*: Create, Delete, Details, Edit, and Index.</span></span>
* <span data-ttu-id="5792c-407">*Data/ Razor PagesMovieContext.cs*</span><span class="sxs-lookup"><span data-stu-id="5792c-407">*Data/RazorPagesMovieContext.cs*</span></span>

### <a name="updated"></a><span data-ttu-id="5792c-408">Aggiornato</span><span class="sxs-lookup"><span data-stu-id="5792c-408">Updated</span></span>

* <span data-ttu-id="5792c-409">*Startup.cs*</span><span class="sxs-lookup"><span data-stu-id="5792c-409">*Startup.cs*</span></span>

<span data-ttu-id="5792c-410">I file creati e aggiornati sono illustrati nella sezione successiva.</span><span class="sxs-lookup"><span data-stu-id="5792c-410">The created and updated files are explained in the next section.</span></span>

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="5792c-411">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="5792c-411">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

<span data-ttu-id="5792c-412">Il processo di scaffolding crea e aggiorna i file seguenti:</span><span class="sxs-lookup"><span data-stu-id="5792c-412">The scaffold process creates and updates the following files:</span></span>

* <span data-ttu-id="5792c-413">*Pagine/filmati*: creare, eliminare, Details, Edit e Index .</span><span class="sxs-lookup"><span data-stu-id="5792c-413">*Pages/Movies*: Create, Delete, Details, Edit, and Index.</span></span>
* <span data-ttu-id="5792c-414">*Data/ Razor PagesMovieContext.cs*</span><span class="sxs-lookup"><span data-stu-id="5792c-414">*Data/RazorPagesMovieContext.cs*</span></span>

### <a name="updated"></a><span data-ttu-id="5792c-415">Aggiornato</span><span class="sxs-lookup"><span data-stu-id="5792c-415">Updated</span></span>

* <span data-ttu-id="5792c-416">*Startup.cs*</span><span class="sxs-lookup"><span data-stu-id="5792c-416">*Startup.cs*</span></span>

<span data-ttu-id="5792c-417">I file creati e aggiornati sono illustrati nella sezione successiva.</span><span class="sxs-lookup"><span data-stu-id="5792c-417">The created and updated files are explained in the next section.</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="5792c-418">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="5792c-418">Visual Studio Code</span></span>](#tab/visual-studio-code)

<span data-ttu-id="5792c-419">Il processo di scaffolding crea i file seguenti:</span><span class="sxs-lookup"><span data-stu-id="5792c-419">The scaffold process creates the following files:</span></span>

* <span data-ttu-id="5792c-420">*Pagine/filmati*: creare, eliminare, Details, Edit e Index .</span><span class="sxs-lookup"><span data-stu-id="5792c-420">*Pages/Movies*: Create, Delete, Details, Edit, and Index.</span></span>

<span data-ttu-id="5792c-421">I file creati sono illustrati nella sezione successiva.</span><span class="sxs-lookup"><span data-stu-id="5792c-421">The created files are explained in the next section.</span></span>

---

<a name="pmc"></a>

## <a name="initial-migration"></a><span data-ttu-id="5792c-422">Migrazione iniziale</span><span class="sxs-lookup"><span data-stu-id="5792c-422">Initial migration</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="5792c-423">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="5792c-423">Visual Studio</span></span>](#tab/visual-studio)

<span data-ttu-id="5792c-424">In questa sezione viene usata la Console di Gestione pacchetti (PMC) per:</span><span class="sxs-lookup"><span data-stu-id="5792c-424">In this section, the Package Manager Console (PMC) is used to:</span></span>

* <span data-ttu-id="5792c-425">Aggiungere una migrazione iniziale.</span><span class="sxs-lookup"><span data-stu-id="5792c-425">Add an initial migration.</span></span>
* <span data-ttu-id="5792c-426">Aggiornare il database con la migrazione iniziale.</span><span class="sxs-lookup"><span data-stu-id="5792c-426">Update the database with the initial migration.</span></span>

<span data-ttu-id="5792c-427">Nel menu **Strumenti** selezionare **Gestione pacchetti NuGet** > **Console di Gestione pacchetti**.</span><span class="sxs-lookup"><span data-stu-id="5792c-427">From the **Tools** menu, select **NuGet Package Manager** > **Package Manager Console**.</span></span>

  ![Menu della Console di Gestione pacchetti](../first-mvc-app/adding-model/_static/pmc.png)

<span data-ttu-id="5792c-429">Nella Console di Gestione pacchetti immettere i comandi seguenti:</span><span class="sxs-lookup"><span data-stu-id="5792c-429">In the PMC, enter the following commands:</span></span>

```powershell
Add-Migration InitialCreate
Update-Database
```

# <a name="visual-studio-code--visual-studio-for-mac"></a>[<span data-ttu-id="5792c-430">Visual Studio Code / Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="5792c-430">Visual Studio Code / Visual Studio for Mac</span></span>](#tab/visual-studio-code+visual-studio-mac)

[!INCLUDE [more information on the CLI for EF Core](~/includes/ef-cli.md)]

<span data-ttu-id="5792c-431">Eseguire i seguenti comandi dell'interfaccia della riga di comando di .NET Core:</span><span class="sxs-lookup"><span data-stu-id="5792c-431">Run the following .NET Core CLI commands:</span></span>

```dotnetcli
dotnet ef migrations add InitialCreate
dotnet ef database update
```

---

<span data-ttu-id="5792c-432">I comandi precedenti generano l'avviso seguente: "nessun tipo specificato per la colonna decimale ' Price ' nel tipo di entità' Movie '.</span><span class="sxs-lookup"><span data-stu-id="5792c-432">The preceding commands generate the following warning: "No type was specified for the decimal column 'Price' on entity type 'Movie'.</span></span> <span data-ttu-id="5792c-433">This will cause values to be silently truncated if they do not fit in the default precision and scale. (I valori saranno quindi automaticamente troncati se non rispettano la precisione e la scala predefinite).</span><span class="sxs-lookup"><span data-stu-id="5792c-433">This will cause values to be silently truncated if they do not fit in the default precision and scale.</span></span> <span data-ttu-id="5792c-434">Explicitly specify the SQL server column type that can accommodate all the values using 'HasColumnType()'. (Specificare in modo esplicito il tipo di colonna di SQL Server che può supportare tutti i valori usando 'HasColumnType()')"</span><span class="sxs-lookup"><span data-stu-id="5792c-434">Explicitly specify the SQL server column type that can accommodate all the values using 'HasColumnType()'."</span></span>

<span data-ttu-id="5792c-435">Ignorare l'avviso poiché verrà risolto in un passaggio successivo.</span><span class="sxs-lookup"><span data-stu-id="5792c-435">Ignore the warning, as it will be addressed in a later step.</span></span>

<span data-ttu-id="5792c-436">Il comando Migrations genera il codice per creare lo schema del database iniziale.</span><span class="sxs-lookup"><span data-stu-id="5792c-436">The migrations command generates code to create the initial database schema.</span></span> <span data-ttu-id="5792c-437">Lo schema è basato sul modello specificato in `DbContext` .</span><span class="sxs-lookup"><span data-stu-id="5792c-437">The schema is based on the model specified in `DbContext`.</span></span> <span data-ttu-id="5792c-438">L'argomento `InitialCreate` viene usato per denominare le migrazioni.</span><span class="sxs-lookup"><span data-stu-id="5792c-438">The `InitialCreate` argument is used to name the migrations.</span></span> <span data-ttu-id="5792c-439">È possibile usare qualsiasi nome, ma per convenzione viene selezionato un nome che descrive la migrazione.</span><span class="sxs-lookup"><span data-stu-id="5792c-439">Any name can be used, but by convention a name is selected that describes the migration.</span></span>

<span data-ttu-id="5792c-440">Il `update` comando esegue il `Up` Metodo nelle migrazioni che non sono state applicate.</span><span class="sxs-lookup"><span data-stu-id="5792c-440">The `update` command runs the `Up` method in migrations that have not been applied.</span></span> <span data-ttu-id="5792c-441">In tal caso, `update` esegue il `Up` metodo nel file  *migrations/ \<time-stamp> _InitialCreate. cs* , che crea il database.</span><span class="sxs-lookup"><span data-stu-id="5792c-441">In this case, `update` runs the `Up` method in  *Migrations/\<time-stamp>_InitialCreate.cs* file, which creates the database.</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="5792c-442">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="5792c-442">Visual Studio</span></span>](#tab/visual-studio)

### <a name="examine-the-context-registered-with-dependency-injection"></a><span data-ttu-id="5792c-443">Esaminare il contesto registrato con l'inserimento di dipendenze</span><span class="sxs-lookup"><span data-stu-id="5792c-443">Examine the context registered with dependency injection</span></span>

<span data-ttu-id="5792c-444">ASP.NET Core viene compilato tramite [dependency injection](xref:fundamentals/dependency-injection).</span><span class="sxs-lookup"><span data-stu-id="5792c-444">ASP.NET Core is built with [dependency injection](xref:fundamentals/dependency-injection).</span></span> <span data-ttu-id="5792c-445">I servizi, ad esempio il contesto del contesto del database EF Core, vengono registrati con l'inserimento delle dipendenze durante l'avvio dell'applicazione.</span><span class="sxs-lookup"><span data-stu-id="5792c-445">Services, such as the EF Core database context context, are registered with dependency injection during application startup.</span></span> <span data-ttu-id="5792c-446">I componenti che richiedono questi servizi, ad esempio Razor le pagine, vengono forniti tramite i parametri del costruttore.</span><span class="sxs-lookup"><span data-stu-id="5792c-446">Components that require these services, such as Razor Pages, are provided these services via constructor parameters.</span></span> <span data-ttu-id="5792c-447">Il codice del costruttore che ottiene un'istanza del contesto del contesto del database viene illustrato più avanti nell'esercitazione.</span><span class="sxs-lookup"><span data-stu-id="5792c-447">The constructor code that gets a database context context instance is shown later in the tutorial.</span></span>

<span data-ttu-id="5792c-448">Lo strumento di ponteggi crea automaticamente un contesto del contesto del database e lo registra con il contenitore di inserimento delle dipendenze.</span><span class="sxs-lookup"><span data-stu-id="5792c-448">The scaffolding tool automatically created a database context context and registered it with the dependency injection container.</span></span>

<span data-ttu-id="5792c-449">Esaminare il metodo `Startup.ConfigureServices`.</span><span class="sxs-lookup"><span data-stu-id="5792c-449">Examine the `Startup.ConfigureServices` method.</span></span> <span data-ttu-id="5792c-450">La riga evidenziata è stata aggiunta dallo scaffolder:</span><span class="sxs-lookup"><span data-stu-id="5792c-450">The highlighted line was added by the scaffolder:</span></span>

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie30/Startup.cs?name=snippet_ConfigureServices&highlight=5-6)]

<span data-ttu-id="5792c-451">Le `RazorPagesMovieContext` coordinate EF Core funzionalità, ad esempio creazione, lettura, aggiornamento ed eliminazione, per il `Movie` modello.</span><span class="sxs-lookup"><span data-stu-id="5792c-451">The `RazorPagesMovieContext` coordinates EF Core functionality, such as Create, Read, Update and Delete, for the `Movie` model.</span></span> <span data-ttu-id="5792c-452">Il contesto dei dati (`RazorPagesMovieContext`) è derivato da [Microsoft.EntityFrameworkCore.DbContext](xref:Microsoft.EntityFrameworkCore.DbContext).</span><span class="sxs-lookup"><span data-stu-id="5792c-452">The data context (`RazorPagesMovieContext`) is derived from [Microsoft.EntityFrameworkCore.DbContext](xref:Microsoft.EntityFrameworkCore.DbContext).</span></span> <span data-ttu-id="5792c-453">Il contesto dei dati specifica le entità incluse nel modello di dati.</span><span class="sxs-lookup"><span data-stu-id="5792c-453">The data context specifies which entities are included in the data model.</span></span>

[!code-csharp[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie30/Data/RazorPagesMovieContext.cs)]

<span data-ttu-id="5792c-454">Il codice precedente crea una [proprietà \<Movie> DbSet](/dotnet/api/microsoft.entityframeworkcore.dbset-1) per il set di entità.</span><span class="sxs-lookup"><span data-stu-id="5792c-454">The preceding code creates a [DbSet\<Movie>](/dotnet/api/microsoft.entityframeworkcore.dbset-1) property for the entity set.</span></span> <span data-ttu-id="5792c-455">Nella terminologia di Entity Framework, un set di entità corrisponde in genere alla tabella di un database.</span><span class="sxs-lookup"><span data-stu-id="5792c-455">In Entity Framework terminology, an entity set typically corresponds to a database table.</span></span> <span data-ttu-id="5792c-456">Un'entità corrisponde a una riga nella tabella.</span><span class="sxs-lookup"><span data-stu-id="5792c-456">An entity corresponds to a row in the table.</span></span>

<span data-ttu-id="5792c-457">Il nome della stringa di connessione viene passato al contesto chiamando un metodo in un oggetto [DbContextOptions](xref:Microsoft.EntityFrameworkCore.DbContextOptions).</span><span class="sxs-lookup"><span data-stu-id="5792c-457">The name of the connection string is passed in to the context by calling a method on a [DbContextOptions](xref:Microsoft.EntityFrameworkCore.DbContextOptions) object.</span></span> <span data-ttu-id="5792c-458">Per lo sviluppo locale, il [sistema di configurazione](xref:fundamentals/configuration/index) legge la stringa di connessione dal *appsettings.json* file.</span><span class="sxs-lookup"><span data-stu-id="5792c-458">For local development, the [Configuration system](xref:fundamentals/configuration/index) reads the connection string from the *appsettings.json* file.</span></span>

# <a name="visual-studio-code--visual-studio-for-mac"></a>[<span data-ttu-id="5792c-459">Visual Studio Code / Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="5792c-459">Visual Studio Code / Visual Studio for Mac</span></span>](#tab/visual-studio-code+visual-studio-mac)

<span data-ttu-id="5792c-460">Esaminare il metodo `Up`.</span><span class="sxs-lookup"><span data-stu-id="5792c-460">Examine the `Up` method.</span></span>

---

<a name="test"></a>

### <a name="test-the-app"></a><span data-ttu-id="5792c-461">Testare l'app</span><span class="sxs-lookup"><span data-stu-id="5792c-461">Test the app</span></span>

* <span data-ttu-id="5792c-462">Eseguire l'app e accodare `/Movies` all'URL nel browser (`http://localhost:port/movies`).</span><span class="sxs-lookup"><span data-stu-id="5792c-462">Run the app and append `/Movies` to the URL in the browser (`http://localhost:port/movies`).</span></span>

<span data-ttu-id="5792c-463">Se viene visualizzato l'errore:</span><span class="sxs-lookup"><span data-stu-id="5792c-463">If you get the error:</span></span>

```console
SqlException: Cannot open database "RazorPagesMovieContext-GUID" requested by the login. The login failed.
Login failed for user 'User-name'.
```

<span data-ttu-id="5792c-464">Non è stato eseguita la [migrazione](#pmc).</span><span class="sxs-lookup"><span data-stu-id="5792c-464">You missed the [migrations step](#pmc).</span></span>

* <span data-ttu-id="5792c-465">Eseguire il test del collegamento **Crea**.</span><span class="sxs-lookup"><span data-stu-id="5792c-465">Test the **Create** link.</span></span>

  ![Pagina di creazione](model/_static/conan5.png)

  > [!NOTE]
  > <span data-ttu-id="5792c-467">Potrebbe non essere possibile immettere virgole decimali nel campo `Price`.</span><span class="sxs-lookup"><span data-stu-id="5792c-467">You may not be able to enter decimal commas in the `Price` field.</span></span> <span data-ttu-id="5792c-468">Per supportare la [convalida jQuery](https://jqueryvalidation.org/) per impostazioni locali diverse dall'inglese che usano la virgola (",") come separatore decimale e per formati di data diversi da quello dell'inglese (Stati Uniti), è necessario localizzare l'app.</span><span class="sxs-lookup"><span data-stu-id="5792c-468">To support [jQuery validation](https://jqueryvalidation.org/) for non-English locales that use a comma (",") for a decimal point and for non US-English date formats, the app must be globalized.</span></span> <span data-ttu-id="5792c-469">Per istruzioni sulla globalizzazione, vedere [questo problema su GitHub](https://github.com/dotnet/AspNetCore.Docs/issues/4076#issuecomment-326590420).</span><span class="sxs-lookup"><span data-stu-id="5792c-469">For globalization instructions, see [this GitHub issue](https://github.com/dotnet/AspNetCore.Docs/issues/4076#issuecomment-326590420).</span></span>

* <span data-ttu-id="5792c-470">Testare i collegamenti **Modifica**, **Dettagli** ed **Elimina**.</span><span class="sxs-lookup"><span data-stu-id="5792c-470">Test the **Edit**, **Details**, and **Delete** links.</span></span>

<span data-ttu-id="5792c-471">L'esercitazione successiva illustra i file creati tramite scaffolding.</span><span class="sxs-lookup"><span data-stu-id="5792c-471">The next tutorial explains the files created by scaffolding.</span></span>

## <a name="additional-resources"></a><span data-ttu-id="5792c-472">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="5792c-472">Additional resources</span></span>

> [!div class="step-by-step"]
> <span data-ttu-id="5792c-473">[Precedente: introduzione](xref:tutorials/razor-pages/razor-pages-start) 
>  Passaggio [successivo: impalcatura Razor Pagine](xref:tutorials/razor-pages/page) di</span><span class="sxs-lookup"><span data-stu-id="5792c-473">[Previous: Get Started](xref:tutorials/razor-pages/razor-pages-start)
[Next: Scaffolded Razor Pages](xref:tutorials/razor-pages/page)</span></span>

::: moniker-end

<!--  ::: moniker previous version   -->
::: moniker range="< aspnetcore-3.0"

<span data-ttu-id="5792c-474">In questa sezione vengono aggiunte le classi per la gestione dei film in un [database SQLite](https://www.sqlite.org/index.html)multipiattaforma.</span><span class="sxs-lookup"><span data-stu-id="5792c-474">In this section, classes are added for managing movies in a cross-platform [SQLite database](https://www.sqlite.org/index.html).</span></span> <span data-ttu-id="5792c-475">Le app create da un modello di ASP.NET Core usano un database SQLite.</span><span class="sxs-lookup"><span data-stu-id="5792c-475">Apps created from an ASP.NET Core template use a SQLite database.</span></span> <span data-ttu-id="5792c-476">Le classi modello dell'app vengono usate con [Entity Framework Core (EF Core)](/ef/core) ([SQLite EF Core provider di database](/ef/core/providers/sqlite)) per lavorare con il database.</span><span class="sxs-lookup"><span data-stu-id="5792c-476">The app's model classes are used with [Entity Framework Core (EF Core)](/ef/core) ([SQLite EF Core Database Provider](/ef/core/providers/sqlite)) to work with the database.</span></span> <span data-ttu-id="5792c-477">EF Core è un framework ORM (Object-Relational Mapping) che semplifica l'accesso ai dati.</span><span class="sxs-lookup"><span data-stu-id="5792c-477">EF Core is an object-relational mapping (ORM) framework that simplifies data access.</span></span>

<span data-ttu-id="5792c-478">Le classi di modello sono dette classi POCO (da "plain-old CLR objects") perché non hanno alcuna dipendenza in EF Core.</span><span class="sxs-lookup"><span data-stu-id="5792c-478">The model classes are known as POCO classes (from "plain-old CLR objects") because they don't have any dependency on EF Core.</span></span> <span data-ttu-id="5792c-479">Definiscono le proprietà dei dati archiviati nel database.</span><span class="sxs-lookup"><span data-stu-id="5792c-479">They define the properties of the data that are stored in the database.</span></span>

<span data-ttu-id="5792c-480">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/razor-pages/razor-pages-start) ([procedura per il download](xref:index#how-to-download-a-sample)).</span><span class="sxs-lookup"><span data-stu-id="5792c-480">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/razor-pages/razor-pages-start) ([how to download](xref:index#how-to-download-a-sample)).</span></span>

## <a name="add-a-data-model"></a><span data-ttu-id="5792c-481">Aggiungere un modello di dati</span><span class="sxs-lookup"><span data-stu-id="5792c-481">Add a data model</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="5792c-482">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="5792c-482">Visual Studio</span></span>](#tab/visual-studio)

<span data-ttu-id="5792c-483">Fare clic con il pulsante destro del mouse sul progetto **Razor PagesMovie** > **Aggiungi**  >  **nuova cartella**.</span><span class="sxs-lookup"><span data-stu-id="5792c-483">Right-click the **RazorPagesMovie** project > **Add** > **New Folder**.</span></span> <span data-ttu-id="5792c-484">Assegnare il nome *Modelli* alla cartella.</span><span class="sxs-lookup"><span data-stu-id="5792c-484">Name the folder *Models*.</span></span>

<span data-ttu-id="5792c-485">Fare clic con il pulsante destro del mouse sulla cartella *modelli* .</span><span class="sxs-lookup"><span data-stu-id="5792c-485">Right-click the *Models* folder.</span></span> <span data-ttu-id="5792c-486">Selezionare **Aggiungi**  >  **classe**.</span><span class="sxs-lookup"><span data-stu-id="5792c-486">Select **Add** > **Class**.</span></span> <span data-ttu-id="5792c-487">Denominare la classe **Movie**.</span><span class="sxs-lookup"><span data-stu-id="5792c-487">Name the class **Movie**.</span></span>

<span data-ttu-id="5792c-488">Aggiungere le proprietà seguenti alla classe `Movie`:</span><span class="sxs-lookup"><span data-stu-id="5792c-488">Add the following properties to the `Movie` class:</span></span>

[!code-csharp[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie22/Models/Movie.cs?name=snippet1)]

<span data-ttu-id="5792c-489">La classe `Movie` contiene:</span><span class="sxs-lookup"><span data-stu-id="5792c-489">The `Movie` class contains:</span></span>

* <span data-ttu-id="5792c-490">Il campo `ID` è richiesto dal database per la chiave primaria.</span><span class="sxs-lookup"><span data-stu-id="5792c-490">The `ID` field is required by the database for the primary key.</span></span>
* <span data-ttu-id="5792c-491">`[DataType(DataType.Date)]`: L'attributo [DataType](xref:System.ComponentModel.DataAnnotations.DataTypeAttribute) specifica il tipo di dati ( `Date` ).</span><span class="sxs-lookup"><span data-stu-id="5792c-491">`[DataType(DataType.Date)]`: The [DataType](xref:System.ComponentModel.DataAnnotations.DataTypeAttribute) attribute specifies the type of the data (`Date`).</span></span> <span data-ttu-id="5792c-492">Con questo attributo:</span><span class="sxs-lookup"><span data-stu-id="5792c-492">With this attribute:</span></span>

  * <span data-ttu-id="5792c-493">l'utente non deve immettere le informazioni temporali nel campo della data.</span><span class="sxs-lookup"><span data-stu-id="5792c-493">The user is not required to enter time information in the date field.</span></span>
  * <span data-ttu-id="5792c-494">Viene visualizzata solo la data, non le informazioni temporali.</span><span class="sxs-lookup"><span data-stu-id="5792c-494">Only the date is displayed, not time information.</span></span>

<span data-ttu-id="5792c-495">L'attributo [DataAnnotations](xref:System.ComponentModel.DataAnnotations) viene analizzato in un'esercitazione successiva.</span><span class="sxs-lookup"><span data-stu-id="5792c-495">[DataAnnotations](xref:System.ComponentModel.DataAnnotations) are covered in a later tutorial.</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="5792c-496">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="5792c-496">Visual Studio Code</span></span>](#tab/visual-studio-code)

* <span data-ttu-id="5792c-497">Aggiungere una cartella denominata *Modelli*.</span><span class="sxs-lookup"><span data-stu-id="5792c-497">Add a folder named *Models*.</span></span>
* <span data-ttu-id="5792c-498">Aggiungere una classe alla cartella *Modello* denominata *Movie.cs*.</span><span class="sxs-lookup"><span data-stu-id="5792c-498">Add a class to the *Models* folder named *Movie.cs*.</span></span>

<span data-ttu-id="5792c-499">Aggiungere le proprietà seguenti alla classe `Movie`:</span><span class="sxs-lookup"><span data-stu-id="5792c-499">Add the following properties to the `Movie` class:</span></span>

[!code-csharp[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie22/Models/Movie.cs?name=snippet1)]

<span data-ttu-id="5792c-500">La classe `Movie` contiene:</span><span class="sxs-lookup"><span data-stu-id="5792c-500">The `Movie` class contains:</span></span>

* <span data-ttu-id="5792c-501">Il campo `ID` è richiesto dal database per la chiave primaria.</span><span class="sxs-lookup"><span data-stu-id="5792c-501">The `ID` field is required by the database for the primary key.</span></span>
* <span data-ttu-id="5792c-502">`[DataType(DataType.Date)]`: L'attributo [DataType](xref:System.ComponentModel.DataAnnotations.DataTypeAttribute) specifica il tipo di dati ( `Date` ).</span><span class="sxs-lookup"><span data-stu-id="5792c-502">`[DataType(DataType.Date)]`: The [DataType](xref:System.ComponentModel.DataAnnotations.DataTypeAttribute) attribute specifies the type of the data (`Date`).</span></span> <span data-ttu-id="5792c-503">Con questo attributo:</span><span class="sxs-lookup"><span data-stu-id="5792c-503">With this attribute:</span></span>

  * <span data-ttu-id="5792c-504">l'utente non deve immettere le informazioni temporali nel campo della data.</span><span class="sxs-lookup"><span data-stu-id="5792c-504">The user is not required to enter time information in the date field.</span></span>
  * <span data-ttu-id="5792c-505">Viene visualizzata solo la data, non le informazioni temporali.</span><span class="sxs-lookup"><span data-stu-id="5792c-505">Only the date is displayed, not time information.</span></span>

<span data-ttu-id="5792c-506">L'attributo [DataAnnotations](xref:System.ComponentModel.DataAnnotations) viene analizzato in un'esercitazione successiva.</span><span class="sxs-lookup"><span data-stu-id="5792c-506">[DataAnnotations](xref:System.ComponentModel.DataAnnotations) are covered in a later tutorial.</span></span>

<a name="dc"></a>

### <a name="add-nuget-packages-and-ef-tools"></a><span data-ttu-id="5792c-507">Aggiungere i pacchetti NuGet e gli strumenti EF</span><span class="sxs-lookup"><span data-stu-id="5792c-507">Add NuGet packages and EF tools</span></span>

[!INCLUDE[](~/includes/add-EF-NuGet-SQLite-CLI.md)]

### <a name="add-a-database-context-class"></a><span data-ttu-id="5792c-508">Aggiungere una classe di contesto dei dati</span><span class="sxs-lookup"><span data-stu-id="5792c-508">Add a database context class</span></span>

<span data-ttu-id="5792c-509">Nel Razor progetto PagesMovie creare una nuova cartella denominata *Data*.</span><span class="sxs-lookup"><span data-stu-id="5792c-509">In the RazorPagesMovie project, create a new folder named *Data*.</span></span> 
<span data-ttu-id="5792c-510">Aggiungere la classe `RazorPagesMovieContext` seguente alla cartella *Data*:</span><span class="sxs-lookup"><span data-stu-id="5792c-510">Add the following `RazorPagesMovieContext` class to the *Data* folder:</span></span>

[!code-csharp[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie30/Data/RazorPagesMovieContext.cs)]

<span data-ttu-id="5792c-511">Il codice precedente crea una proprietà `DbSet` per il set di entità.</span><span class="sxs-lookup"><span data-stu-id="5792c-511">The preceding code creates a `DbSet` property for the entity set.</span></span> <span data-ttu-id="5792c-512">Nella terminologia di Entity Framework, un set di entità corrisponde in genere alla tabella di un database e un'entità corrisponde a una riga della tabella.</span><span class="sxs-lookup"><span data-stu-id="5792c-512">In Entity Framework terminology, an entity set typically corresponds to a database table, and an entity corresponds to a row in the table.</span></span> <span data-ttu-id="5792c-513">Il codice non verrà compilato fino a quando le dipendenze non verranno aggiunte in un passaggio successivo.</span><span class="sxs-lookup"><span data-stu-id="5792c-513">The code won't compile until dependencies are added in a later step.</span></span>

<a name="cs"></a>

### <a name="add-a-database-connection-string"></a><span data-ttu-id="5792c-514">Aggiungere una stringa di connessione del database</span><span class="sxs-lookup"><span data-stu-id="5792c-514">Add a database connection string</span></span>

<span data-ttu-id="5792c-515">Aggiungere una stringa di connessione al *appsettings.json* file come illustrato nel codice evidenziato seguente:</span><span class="sxs-lookup"><span data-stu-id="5792c-515">Add a connection string to the *appsettings.json* file as shown in the following highlighted code:</span></span>

[!code-json[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie22/appsettings_SQLite.json?highlight=8-10)]

### <a name="add-required-nuget-packages"></a><span data-ttu-id="5792c-516">Aggiungere i pacchetti NuGet necessari</span><span class="sxs-lookup"><span data-stu-id="5792c-516">Add required NuGet packages</span></span>

<span data-ttu-id="5792c-517">Eseguire il comando interfaccia della riga di comando di .NET Core seguente per aggiungere SQLite e CodeGeneration. Design al progetto:</span><span class="sxs-lookup"><span data-stu-id="5792c-517">Run the following .NET Core CLI command to add SQLite and CodeGeneration.Design to the project:</span></span>

```dotnetcli
dotnet add package Microsoft.EntityFrameworkCore.Sqlite --version 2.2.6
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design --version 2.2.4
dotnet add package Microsoft.EntityFrameworkCore.Design --version 2.2.6
```

<span data-ttu-id="5792c-518">Il pacchetto `Microsoft.VisualStudio.Web.CodeGeneration.Design` è necessario per lo scaffolding.</span><span class="sxs-lookup"><span data-stu-id="5792c-518">The `Microsoft.VisualStudio.Web.CodeGeneration.Design` package is required for scaffolding.</span></span>

<a name="reg"></a>

### <a name="register-the-database-context"></a><span data-ttu-id="5792c-519">Registrare il contesto del database</span><span class="sxs-lookup"><span data-stu-id="5792c-519">Register the database context</span></span>

<span data-ttu-id="5792c-520">Aggiungere le istruzioni `using` seguenti all'inizio di *Startup.cs*:</span><span class="sxs-lookup"><span data-stu-id="5792c-520">Add the following `using` statements at the top of *Startup.cs*:</span></span>

```csharp
using RazorPagesMovie.Models;
using Microsoft.EntityFrameworkCore;
```

<span data-ttu-id="5792c-521">Registrare il contesto del database con il contenitore di [inserimento delle dipendenze](xref:fundamentals/dependency-injection) in `Startup.ConfigureServices`.</span><span class="sxs-lookup"><span data-stu-id="5792c-521">Register the database context with the [dependency injection](xref:fundamentals/dependency-injection) container in `Startup.ConfigureServices`.</span></span>

[!code-csharp[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie22/Startup.cs?name=snippet_UseSqlite&highlight=11-12)]

<span data-ttu-id="5792c-522">Compilare il progetto per il controllo di eventuali errori.</span><span class="sxs-lookup"><span data-stu-id="5792c-522">Build the project as a check for errors.</span></span>

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="5792c-523">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="5792c-523">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

* <span data-ttu-id="5792c-524">Nella **finestra degli strumenti della soluzione**, fare clic sul progetto *Razor PagesMovie* e quindi selezionare **Aggiungi**  >  **nuova cartella**.</span><span class="sxs-lookup"><span data-stu-id="5792c-524">In **Solution Tool Window**, control-click the *RazorPagesMovie* project, and then select **Add** > **New Folder**.</span></span> <span data-ttu-id="5792c-525">Assegnare il nome *Modelli* alla cartella.</span><span class="sxs-lookup"><span data-stu-id="5792c-525">Name the folder *Models*.</span></span>
* <span data-ttu-id="5792c-526">CTRL: fare clic sulla cartella *modelli* , quindi selezionare **Aggiungi** > **nuovo file**.</span><span class="sxs-lookup"><span data-stu-id="5792c-526">Control-click the *Models* folder, and then select **Add** > **New File**.</span></span>
* <span data-ttu-id="5792c-527">Nel finestra di dialogo **Nuovo file**:</span><span class="sxs-lookup"><span data-stu-id="5792c-527">In the **New File** dialog:</span></span>

  * <span data-ttu-id="5792c-528">Selezionare **Generale** nel riquadro a sinistra.</span><span class="sxs-lookup"><span data-stu-id="5792c-528">Select **General** in the left pane.</span></span>
  * <span data-ttu-id="5792c-529">Selezionare **Classe vuota** nel riquadro centrale.</span><span class="sxs-lookup"><span data-stu-id="5792c-529">Select **Empty Class** in the center pane.</span></span>
  * <span data-ttu-id="5792c-530">Denominare la classe **Filmato** e selezionare **Nuovo**.</span><span class="sxs-lookup"><span data-stu-id="5792c-530">Name the class **Movie** and select **New**.</span></span>

<span data-ttu-id="5792c-531">Aggiungere le proprietà seguenti alla classe `Movie`:</span><span class="sxs-lookup"><span data-stu-id="5792c-531">Add the following properties to the `Movie` class:</span></span>

[!code-csharp[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie22/Models/Movie.cs?name=snippet1)]

<span data-ttu-id="5792c-532">La classe `Movie` contiene:</span><span class="sxs-lookup"><span data-stu-id="5792c-532">The `Movie` class contains:</span></span>

* <span data-ttu-id="5792c-533">Il campo `ID` è richiesto dal database per la chiave primaria.</span><span class="sxs-lookup"><span data-stu-id="5792c-533">The `ID` field is required by the database for the primary key.</span></span>
* <span data-ttu-id="5792c-534">`[DataType(DataType.Date)]`: L'attributo [DataType](xref:System.ComponentModel.DataAnnotations.DataTypeAttribute) specifica il tipo di dati ( `Date` ).</span><span class="sxs-lookup"><span data-stu-id="5792c-534">`[DataType(DataType.Date)]`: The [DataType](xref:System.ComponentModel.DataAnnotations.DataTypeAttribute) attribute specifies the type of the data (`Date`).</span></span> <span data-ttu-id="5792c-535">Con questo attributo:</span><span class="sxs-lookup"><span data-stu-id="5792c-535">With this attribute:</span></span>

  * <span data-ttu-id="5792c-536">l'utente non deve immettere le informazioni temporali nel campo della data.</span><span class="sxs-lookup"><span data-stu-id="5792c-536">The user is not required to enter time information in the date field.</span></span>
  * <span data-ttu-id="5792c-537">Viene visualizzata solo la data, non le informazioni temporali.</span><span class="sxs-lookup"><span data-stu-id="5792c-537">Only the date is displayed, not time information.</span></span>

<span data-ttu-id="5792c-538">L'attributo [DataAnnotations](xref:System.ComponentModel.DataAnnotations) viene analizzato in un'esercitazione successiva.</span><span class="sxs-lookup"><span data-stu-id="5792c-538">[DataAnnotations](xref:System.ComponentModel.DataAnnotations) are covered in a later tutorial.</span></span>

---

<span data-ttu-id="5792c-539">Compilare il progetto per verificare che non siano presenti errori di compilazione.</span><span class="sxs-lookup"><span data-stu-id="5792c-539">Build the project to verify there are no compilation errors.</span></span>

## <a name="scaffold-the-movie-model"></a><span data-ttu-id="5792c-540">Eseguire lo scaffolding del modello di filmato</span><span class="sxs-lookup"><span data-stu-id="5792c-540">Scaffold the movie model</span></span>

<span data-ttu-id="5792c-541">In questa sezione viene eseguito lo scaffolding del modello *Movie*.</span><span class="sxs-lookup"><span data-stu-id="5792c-541">In this section, the movie model is scaffolded.</span></span> <span data-ttu-id="5792c-542">Lo strumento di scaffolding crea quindi le pagine per le operazioni CRUD (creazione, lettura, aggiornamento ed eliminazione) per il modello *Movie*.</span><span class="sxs-lookup"><span data-stu-id="5792c-542">That is, the scaffolding tool produces pages for Create, Read, Update, and Delete (CRUD) operations for the movie model.</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="5792c-543">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="5792c-543">Visual Studio</span></span>](#tab/visual-studio)

<span data-ttu-id="5792c-544">Creare una cartella *Pages/Movies*:</span><span class="sxs-lookup"><span data-stu-id="5792c-544">Create a *Pages/Movies* folder:</span></span>

* <span data-ttu-id="5792c-545">Fare clic con il pulsante destro del mouse sulla cartella *pagine* > **Aggiungi** > **nuova cartella**.</span><span class="sxs-lookup"><span data-stu-id="5792c-545">Right-click on the *Pages* folder > **Add** > **New Folder**.</span></span>
* <span data-ttu-id="5792c-546">Denominare la cartella *Movies*.</span><span class="sxs-lookup"><span data-stu-id="5792c-546">Name the folder *Movies*.</span></span>

<span data-ttu-id="5792c-547">Fare clic con il pulsante destro del mouse sulla cartella *pages/Movies* > **Aggiungi** > **nuovo elemento con impalcatura**.</span><span class="sxs-lookup"><span data-stu-id="5792c-547">Right-click on the *Pages/Movies* folder > **Add** > **New Scaffolded Item**.</span></span>

![Immagine relativa alle istruzioni precedenti.](model/_static/sca.png)

<span data-ttu-id="5792c-549">Nella finestra di dialogo **Aggiungi impalcatura** selezionare **Razor pages using Entity Framework (CRUD)** > **Add**.</span><span class="sxs-lookup"><span data-stu-id="5792c-549">In the **Add Scaffold** dialog, select **Razor Pages using Entity Framework (CRUD)** > **Add**.</span></span>

![Immagine relativa alle istruzioni precedenti.](model/_static/add_scaffold.png)

<span data-ttu-id="5792c-551">Completare la finestra di dialogo **Aggiungi Razor pagine con Entity Framework (CRUD)** :</span><span class="sxs-lookup"><span data-stu-id="5792c-551">Complete the **Add Razor Pages using Entity Framework (CRUD)** dialog:</span></span>
<!-- In the next section, change 
(plus) sign and accept the generated name 
to use Data, it should not use models. That will make the namespace the same for the VS version and the CLI version
-->

* <span data-ttu-id="5792c-552">Nell'elenco a discesa **classe modello** selezionare **Movie ( Razor PagesMovie. Models)**.</span><span class="sxs-lookup"><span data-stu-id="5792c-552">In the **Model class** drop down, select **Movie (RazorPagesMovie.Models)**.</span></span>
* <span data-ttu-id="5792c-553">Nella riga della **classe del contesto dati** selezionare il **+** segno (più) e accettare il nome generato **Razor PagesMovie. Models. Razor PagesMovieContext**.</span><span class="sxs-lookup"><span data-stu-id="5792c-553">In the **Data context class** row, select the **+** (plus) sign and accept the generated name **RazorPagesMovie.Models.RazorPagesMovieContext**.</span></span>
* <span data-ttu-id="5792c-554">Selezionare **Aggiungi**.</span><span class="sxs-lookup"><span data-stu-id="5792c-554">Select **Add**.</span></span>

![Immagine relativa alle istruzioni precedenti.](model/_static/arp.png)

<span data-ttu-id="5792c-556">Il *appsettings.json* file viene aggiornato con la stringa di connessione utilizzata per connettersi a un database locale.</span><span class="sxs-lookup"><span data-stu-id="5792c-556">The *appsettings.json* file is updated with the connection string used to connect to a local database.</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="5792c-557">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="5792c-557">Visual Studio Code</span></span>](#tab/visual-studio-code)

<!--  Until https://github.com/aspnet/Scaffolding/issues/582 is fixed windows needs backslash or the namespace is namespace RazorPagesMovie.Pages_Movies rather than namespace RazorPagesMovie.Pages.Movies
-->

* <span data-ttu-id="5792c-558">Aprire una finestra di comando nella directory del progetto che contiene i file *Program.cs*, *Startup.cs* e *. csproj* .</span><span class="sxs-lookup"><span data-stu-id="5792c-558">Open a command window in the project directory, which contains the *Program.cs*, *Startup.cs*, and *.csproj* files.</span></span>

* <span data-ttu-id="5792c-559">**Per Windows**: eseguire il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="5792c-559">**For Windows**: Run the following command:</span></span>

  ```dotnetcli
  dotnet-aspnet-codegenerator razorpage -m Movie -dc RazorPagesMovieContext -udl -outDir Pages\Movies --referenceScriptLibraries
  ```

* <span data-ttu-id="5792c-560">**Per MacOS e Linux**: eseguire il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="5792c-560">**For macOS and Linux**: Run the following command:</span></span>

  ```dotnetcli
  dotnet-aspnet-codegenerator razorpage -m Movie -dc RazorPagesMovieContext -udl -outDir Pages/Movies --referenceScriptLibraries
  ```

<a name="codegenerator"></a> <span data-ttu-id="5792c-561">La tabella seguente illustra in dettaglio le opzioni del generatore di codice ASP.NET Core:</span><span class="sxs-lookup"><span data-stu-id="5792c-561">The following table details the ASP.NET Core code generator options:</span></span>

| <span data-ttu-id="5792c-562">Opzione</span><span class="sxs-lookup"><span data-stu-id="5792c-562">Option</span></span>               | <span data-ttu-id="5792c-563">Descrizione</span><span class="sxs-lookup"><span data-stu-id="5792c-563">Description</span></span>|
| ----------------- | ------------ |
| `-m`  | <span data-ttu-id="5792c-564">Nome del modello.</span><span class="sxs-lookup"><span data-stu-id="5792c-564">The name of the model.</span></span> |
| `-dc`  | <span data-ttu-id="5792c-565">Classe `DbContext` da usare.</span><span class="sxs-lookup"><span data-stu-id="5792c-565">The `DbContext` class to use.</span></span> |
| `-udl` | <span data-ttu-id="5792c-566">Uso del layout predefinito.</span><span class="sxs-lookup"><span data-stu-id="5792c-566">Use the default layout.</span></span> |
| `-outDir` | <span data-ttu-id="5792c-567">Percorso relativo della cartella di output per creare le viste.</span><span class="sxs-lookup"><span data-stu-id="5792c-567">The relative output folder path to create the views.</span></span> |
| `--referenceScriptLibraries` | <span data-ttu-id="5792c-568">Aggiunge `_ValidationScriptsPartial` per modificare e creare pagine</span><span class="sxs-lookup"><span data-stu-id="5792c-568">Adds `_ValidationScriptsPartial` to Edit and Create pages</span></span> |

<span data-ttu-id="5792c-569">Usare l' `-h` opzione per ottenere informazioni sul `aspnet-codegenerator razorpage` comando:</span><span class="sxs-lookup"><span data-stu-id="5792c-569">Use the `-h` option to get help on the `aspnet-codegenerator razorpage` command:</span></span>

```dotnetcli
dotnet-aspnet-codegenerator razorpage -h
```

<span data-ttu-id="5792c-570">Per ulteriori informazioni, vedere [DotNet-ASPNET-CodeGenerator](xref:fundamentals/tools/dotnet-aspnet-codegenerator).</span><span class="sxs-lookup"><span data-stu-id="5792c-570">For more information, see [dotnet-aspnet-codegenerator](xref:fundamentals/tools/dotnet-aspnet-codegenerator).</span></span>

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="5792c-571">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="5792c-571">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

<span data-ttu-id="5792c-572">Creare una cartella *Pages/Movies*:</span><span class="sxs-lookup"><span data-stu-id="5792c-572">Create a *Pages/Movies* folder:</span></span>

* <span data-ttu-id="5792c-573">CTRL: fare clic sulla cartella *pagine* > **Aggiungi** > **nuova cartella**.</span><span class="sxs-lookup"><span data-stu-id="5792c-573">Control-click on the *Pages* folder > **Add** > **New Folder**.</span></span>
* <span data-ttu-id="5792c-574">Denominare la cartella *Movies*.</span><span class="sxs-lookup"><span data-stu-id="5792c-574">Name the folder *Movies*.</span></span>

<span data-ttu-id="5792c-575">Control: fare clic sulla cartella *pages/Movies* > **aggiungere** un > **nuovo elemento con impalcatura**.</span><span class="sxs-lookup"><span data-stu-id="5792c-575">Control-click on the *Pages/Movies* folder > **Add** > **New Scaffolded Item**.</span></span>

![Immagine relativa alle istruzioni precedenti.](model/_static/scaMac.png)

<span data-ttu-id="5792c-577">Nella finestra di dialogo **Aggiungi nuova impalcatura** selezionare **Razor pages using Entity Framework (CRUD)** > **Add**.</span><span class="sxs-lookup"><span data-stu-id="5792c-577">In the **Add New Scaffolding** dialog, select **Razor Pages using Entity Framework (CRUD)** > **Add**.</span></span>

![Immagine relativa alle istruzioni precedenti.](model/_static/add_scaffoldMac.png)

<span data-ttu-id="5792c-579">Completare la finestra di dialogo **Aggiungi Razor pagine con Entity Framework (CRUD)** :</span><span class="sxs-lookup"><span data-stu-id="5792c-579">Complete the **Add Razor Pages using Entity Framework (CRUD)** dialog:</span></span>

* <span data-ttu-id="5792c-580">Nell'elenco a discesa **classe modello** selezionare o digitare **Movie**.</span><span class="sxs-lookup"><span data-stu-id="5792c-580">In the **Model class** drop down, select or type **Movie**.</span></span>
* <span data-ttu-id="5792c-581">Nella riga della **classe del contesto dati** Digitare select the **Razor PagesMovieContext** . verrà creata una nuova classe del contesto di database con lo spazio dei nomi corretto.</span><span class="sxs-lookup"><span data-stu-id="5792c-581">In the **Data context class** row, type select the **RazorPagesMovieContext** this will create a new database context class with the correct namespace.</span></span> <span data-ttu-id="5792c-582">In questo caso sarà **Razor PagesMovie. Models. Razor PagesMovieContext**.</span><span class="sxs-lookup"><span data-stu-id="5792c-582">In this case it will be  **RazorPagesMovie.Models.RazorPagesMovieContext**.</span></span>
* <span data-ttu-id="5792c-583">Selezionare **Aggiungi**.</span><span class="sxs-lookup"><span data-stu-id="5792c-583">Select **Add**.</span></span>

![Immagine relativa alle istruzioni precedenti.](model/_static/arpMac.png)

<span data-ttu-id="5792c-585">Il *appsettings.json* file viene aggiornato con la stringa di connessione utilizzata per connettersi a un database locale.</span><span class="sxs-lookup"><span data-stu-id="5792c-585">The *appsettings.json* file is updated with the connection string used to connect to a local database.</span></span>

---

<span data-ttu-id="5792c-586">Il processo di scaffolding crea e aggiorna i file seguenti:</span><span class="sxs-lookup"><span data-stu-id="5792c-586">The scaffold process creates and updates the following files:</span></span>

### <a name="files-created"></a><span data-ttu-id="5792c-587">File creati</span><span class="sxs-lookup"><span data-stu-id="5792c-587">Files created</span></span>

* <span data-ttu-id="5792c-588">*Pagine/filmati*: creare, eliminare, Details, Edit e Index .</span><span class="sxs-lookup"><span data-stu-id="5792c-588">*Pages/Movies*: Create, Delete, Details, Edit, and Index.</span></span>
* <span data-ttu-id="5792c-589">*Data/ Razor PagesMovieContext.cs*</span><span class="sxs-lookup"><span data-stu-id="5792c-589">*Data/RazorPagesMovieContext.cs*</span></span>

### <a name="file-updated"></a><span data-ttu-id="5792c-590">File aggiornato</span><span class="sxs-lookup"><span data-stu-id="5792c-590">File updated</span></span>

* <span data-ttu-id="5792c-591">*Startup.cs*</span><span class="sxs-lookup"><span data-stu-id="5792c-591">*Startup.cs*</span></span>

<span data-ttu-id="5792c-592">I file creati e aggiornati sono illustrati nella sezione successiva.</span><span class="sxs-lookup"><span data-stu-id="5792c-592">The created and updated files are explained in the next section.</span></span>

<a name="pmc"></a>

## <a name="initial-migration"></a><span data-ttu-id="5792c-593">Migrazione iniziale</span><span class="sxs-lookup"><span data-stu-id="5792c-593">Initial migration</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="5792c-594">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="5792c-594">Visual Studio</span></span>](#tab/visual-studio)

<span data-ttu-id="5792c-595">In questa sezione viene usata la Console di Gestione pacchetti (PMC) per:</span><span class="sxs-lookup"><span data-stu-id="5792c-595">In this section, the Package Manager Console (PMC) is used to:</span></span>

* <span data-ttu-id="5792c-596">Aggiungere una migrazione iniziale.</span><span class="sxs-lookup"><span data-stu-id="5792c-596">Add an initial migration.</span></span>
* <span data-ttu-id="5792c-597">Aggiornare il database con la migrazione iniziale.</span><span class="sxs-lookup"><span data-stu-id="5792c-597">Update the database with the initial migration.</span></span>

<span data-ttu-id="5792c-598">Nel menu **Strumenti** selezionare **Gestione pacchetti NuGet** > **Console di Gestione pacchetti**.</span><span class="sxs-lookup"><span data-stu-id="5792c-598">From the **Tools** menu, select **NuGet Package Manager** > **Package Manager Console**.</span></span>

  ![Menu della Console di Gestione pacchetti](../first-mvc-app/adding-model/_static/pmc.png)

<span data-ttu-id="5792c-600">Nella Console di Gestione pacchetti immettere i comandi seguenti:</span><span class="sxs-lookup"><span data-stu-id="5792c-600">In the PMC, enter the following commands:</span></span>

```powershell
Add-Migration Initial
Update-Database
```

<span data-ttu-id="5792c-601">Il comando `Add-Migration` genera un codice per creare lo schema del database iniziale.</span><span class="sxs-lookup"><span data-stu-id="5792c-601">The `Add-Migration` command generates code to create the initial database schema.</span></span> <span data-ttu-id="5792c-602">Lo schema è basato sul modello specificato in `DbContext` , nel file *Razor PagesMovieContext.cs* .</span><span class="sxs-lookup"><span data-stu-id="5792c-602">The schema is based on the model specified in the `DbContext`, in the *RazorPagesMovieContext.cs* file.</span></span> <span data-ttu-id="5792c-603">L' `InitialCreate` argomento viene usato per assegnare un nome alla migrazione.</span><span class="sxs-lookup"><span data-stu-id="5792c-603">The `InitialCreate` argument is used to name the migration.</span></span> <span data-ttu-id="5792c-604">È possibile usare qualsiasi nome, ma per convenzione viene usato un nome che descrive la migrazione.</span><span class="sxs-lookup"><span data-stu-id="5792c-604">Any name can be used, but by convention a name that describes the migration is used.</span></span> <span data-ttu-id="5792c-605">Per altre informazioni, vedere <xref:data/ef-mvc/migrations>.</span><span class="sxs-lookup"><span data-stu-id="5792c-605">For more information, see <xref:data/ef-mvc/migrations>.</span></span>

<span data-ttu-id="5792c-606">Il `Update-Database` comando esegue il `Up` metodo nel file *migrations/ \<time-stamp> _InitialCreate. cs* .</span><span class="sxs-lookup"><span data-stu-id="5792c-606">The `Update-Database` command runs the `Up` method in the *Migrations/\<time-stamp>_InitialCreate.cs* file.</span></span> <span data-ttu-id="5792c-607">Il metodo `Up` crea il database.</span><span class="sxs-lookup"><span data-stu-id="5792c-607">The `Up` method creates the database.</span></span>

# <a name="visual-studio-code--visual-studio-for-mac"></a>[<span data-ttu-id="5792c-608">Visual Studio Code / Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="5792c-608">Visual Studio Code / Visual Studio for Mac</span></span>](#tab/visual-studio-code+visual-studio-mac)

[!INCLUDE [more information on the CLI for EF Core](~/includes/ef-cli.md)]

<span data-ttu-id="5792c-609">Eseguire i seguenti comandi dell'interfaccia della riga di comando di .NET Core:</span><span class="sxs-lookup"><span data-stu-id="5792c-609">Run the following .NET Core CLI commands:</span></span>

```dotnetcli
dotnet ef migrations add InitialCreate
dotnet ef database update
```

---

<span data-ttu-id="5792c-610">I comandi precedenti generano l'avviso seguente: "nessun tipo specificato per la colonna decimale ' Price ' nel tipo di entità' Movie '.</span><span class="sxs-lookup"><span data-stu-id="5792c-610">The preceding commands generate the following warning: "No type was specified for the decimal column 'Price' on entity type 'Movie'.</span></span> <span data-ttu-id="5792c-611">This will cause values to be silently truncated if they do not fit in the default precision and scale. (I valori saranno quindi automaticamente troncati se non rispettano la precisione e la scala predefinite).</span><span class="sxs-lookup"><span data-stu-id="5792c-611">This will cause values to be silently truncated if they do not fit in the default precision and scale.</span></span> <span data-ttu-id="5792c-612">Explicitly specify the SQL server column type that can accommodate all the values using 'HasColumnType()'. (Specificare in modo esplicito il tipo di colonna di SQL Server che può supportare tutti i valori usando 'HasColumnType()')"</span><span class="sxs-lookup"><span data-stu-id="5792c-612">Explicitly specify the SQL server column type that can accommodate all the values using 'HasColumnType()'."</span></span>

<span data-ttu-id="5792c-613">Ignorare l'avviso poiché verrà risolto in un passaggio successivo.</span><span class="sxs-lookup"><span data-stu-id="5792c-613">Ignore the warning, as it will be addressed in a later step.</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="5792c-614">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="5792c-614">Visual Studio</span></span>](#tab/visual-studio)

### <a name="examine-the-context-registered-with-dependency-injection"></a><span data-ttu-id="5792c-615">Esaminare il contesto registrato con l'inserimento di dipendenze</span><span class="sxs-lookup"><span data-stu-id="5792c-615">Examine the context registered with dependency injection</span></span>

<span data-ttu-id="5792c-616">ASP.NET Core viene compilato tramite [dependency injection](xref:fundamentals/dependency-injection).</span><span class="sxs-lookup"><span data-stu-id="5792c-616">ASP.NET Core is built with [dependency injection](xref:fundamentals/dependency-injection).</span></span> <span data-ttu-id="5792c-617">I servizi (ad esempio il contesto del contesto del database EF Core) vengono registrati con l'inserimento delle dipendenze durante l'avvio dell'applicazione.</span><span class="sxs-lookup"><span data-stu-id="5792c-617">Services (such as the EF Core database context context) are registered with dependency injection during application startup.</span></span> <span data-ttu-id="5792c-618">I componenti che richiedono questi servizi, ad esempio Razor le pagine, vengono forniti tramite i parametri del costruttore.</span><span class="sxs-lookup"><span data-stu-id="5792c-618">Components that require these services (such as Razor Pages) are provided these services via constructor parameters.</span></span> <span data-ttu-id="5792c-619">Il codice del costruttore che ottiene un'istanza del contesto contextB del contesto di database viene illustrato più avanti nell'esercitazione.</span><span class="sxs-lookup"><span data-stu-id="5792c-619">The constructor code that gets a database context contextB context instance is shown later in the tutorial.</span></span>

<span data-ttu-id="5792c-620">Lo strumento di ponteggi crea automaticamente un contesto del contesto del database e lo registra con il contenitore di inserimento delle dipendenze.</span><span class="sxs-lookup"><span data-stu-id="5792c-620">The scaffolding tool automatically created a database context context and registered it with the dependency injection container.</span></span>

<span data-ttu-id="5792c-621">Esaminare il metodo `Startup.ConfigureServices`.</span><span class="sxs-lookup"><span data-stu-id="5792c-621">Examine the `Startup.ConfigureServices` method.</span></span> <span data-ttu-id="5792c-622">La riga evidenziata è stata aggiunta dallo scaffolder:</span><span class="sxs-lookup"><span data-stu-id="5792c-622">The highlighted line was added by the scaffolder:</span></span>

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie22/Startup.cs?name=snippet_ConfigureServices&highlight=15-18)]

<span data-ttu-id="5792c-623">Le `RazorPagesMovieContext` coordinate EF Core funzionalità, ad esempio creazione, lettura, aggiornamento ed eliminazione, per il `Movie` modello.</span><span class="sxs-lookup"><span data-stu-id="5792c-623">The `RazorPagesMovieContext` coordinates EF Core functionality, such as Create, Read, Update, and Delete, for the `Movie` model.</span></span> <span data-ttu-id="5792c-624">Il contesto dei dati (`RazorPagesMovieContext`) è derivato da [Microsoft.EntityFrameworkCore.DbContext](xref:Microsoft.EntityFrameworkCore.DbContext).</span><span class="sxs-lookup"><span data-stu-id="5792c-624">The data context (`RazorPagesMovieContext`) is derived from [Microsoft.EntityFrameworkCore.DbContext](xref:Microsoft.EntityFrameworkCore.DbContext).</span></span> <span data-ttu-id="5792c-625">Il contesto dei dati specifica le entità incluse nel modello di dati.</span><span class="sxs-lookup"><span data-stu-id="5792c-625">The data context specifies which entities are included in the data model.</span></span>

[!code-csharp[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie22/Data/RazorPagesMovieContext.cs)]

<span data-ttu-id="5792c-626">Il codice precedente crea una [proprietà \<Movie> DbSet](/dotnet/api/microsoft.entityframeworkcore.dbset-1) per il set di entità.</span><span class="sxs-lookup"><span data-stu-id="5792c-626">The preceding code creates a [DbSet\<Movie>](/dotnet/api/microsoft.entityframeworkcore.dbset-1) property for the entity set.</span></span> <span data-ttu-id="5792c-627">Nella terminologia di Entity Framework, un set di entità corrisponde in genere alla tabella di un database.</span><span class="sxs-lookup"><span data-stu-id="5792c-627">In Entity Framework terminology, an entity set typically corresponds to a database table.</span></span> <span data-ttu-id="5792c-628">Un'entità corrisponde a una riga nella tabella.</span><span class="sxs-lookup"><span data-stu-id="5792c-628">An entity corresponds to a row in the table.</span></span>

<span data-ttu-id="5792c-629">Il nome della stringa di connessione viene passato al contesto chiamando un metodo in un oggetto [DbContextOptions](xref:Microsoft.EntityFrameworkCore.DbContextOptions).</span><span class="sxs-lookup"><span data-stu-id="5792c-629">The name of the connection string is passed in to the context by calling a method on a [DbContextOptions](xref:Microsoft.EntityFrameworkCore.DbContextOptions) object.</span></span> <span data-ttu-id="5792c-630">Per lo sviluppo locale, il [sistema di configurazione](xref:fundamentals/configuration/index) legge la stringa di connessione dal *appsettings.json* file.</span><span class="sxs-lookup"><span data-stu-id="5792c-630">For local development, the [Configuration system](xref:fundamentals/configuration/index) reads the connection string from the *appsettings.json* file.</span></span>

# <a name="visual-studio-code--visual-studio-for-mac"></a>[<span data-ttu-id="5792c-631">Visual Studio Code / Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="5792c-631">Visual Studio Code / Visual Studio for Mac</span></span>](#tab/visual-studio-code+visual-studio-mac)

<span data-ttu-id="5792c-632">Esaminare il metodo `Up`.</span><span class="sxs-lookup"><span data-stu-id="5792c-632">Examine the `Up` method.</span></span>

---

<a name="test"></a>

### <a name="test-the-app"></a><span data-ttu-id="5792c-633">Testare l'app</span><span class="sxs-lookup"><span data-stu-id="5792c-633">Test the app</span></span>

* <span data-ttu-id="5792c-634">Eseguire l'app e accodare `/Movies` all'URL nel browser (`http://localhost:port/movies`).</span><span class="sxs-lookup"><span data-stu-id="5792c-634">Run the app and append `/Movies` to the URL in the browser (`http://localhost:port/movies`).</span></span>

<span data-ttu-id="5792c-635">Se viene visualizzato l'errore:</span><span class="sxs-lookup"><span data-stu-id="5792c-635">If you get the error:</span></span>

```console
SqlException: Cannot open database "RazorPagesMovieContext-GUID" requested by the login. The login failed.
Login failed for user 'User-name'.
```

<span data-ttu-id="5792c-636">Non è stato eseguita la [migrazione](#pmc).</span><span class="sxs-lookup"><span data-stu-id="5792c-636">You missed the [migrations step](#pmc).</span></span>

* <span data-ttu-id="5792c-637">Eseguire il test del collegamento **Crea**.</span><span class="sxs-lookup"><span data-stu-id="5792c-637">Test the **Create** link.</span></span>

  ![Pagina di creazione](model/_static/conan.png)

  > [!NOTE]
  > <span data-ttu-id="5792c-639">Potrebbe non essere possibile immettere virgole decimali nel campo `Price`.</span><span class="sxs-lookup"><span data-stu-id="5792c-639">You may not be able to enter decimal commas in the `Price` field.</span></span> <span data-ttu-id="5792c-640">Per supportare la [convalida jQuery](https://jqueryvalidation.org/) per impostazioni locali diverse dall'inglese che usano la virgola (",") come separatore decimale e per formati di data diversi da quello dell'inglese (Stati Uniti), è necessario localizzare l'app.</span><span class="sxs-lookup"><span data-stu-id="5792c-640">To support [jQuery validation](https://jqueryvalidation.org/) for non-English locales that use a comma (",") for a decimal point and for non US-English date formats, the app must be globalized.</span></span> <span data-ttu-id="5792c-641">Per istruzioni sulla globalizzazione, vedere [questo problema su GitHub](https://github.com/dotnet/AspNetCore.Docs/issues/4076#issuecomment-326590420).</span><span class="sxs-lookup"><span data-stu-id="5792c-641">For globalization instructions, see [this GitHub issue](https://github.com/dotnet/AspNetCore.Docs/issues/4076#issuecomment-326590420).</span></span>

* <span data-ttu-id="5792c-642">Testare i collegamenti **Modifica**, **Dettagli** ed **Elimina**.</span><span class="sxs-lookup"><span data-stu-id="5792c-642">Test the **Edit**, **Details**, and **Delete** links.</span></span>

<span data-ttu-id="5792c-643">L'esercitazione successiva illustra i file creati tramite scaffolding.</span><span class="sxs-lookup"><span data-stu-id="5792c-643">The next tutorial explains the files created by scaffolding.</span></span>

## <a name="additional-resources"></a><span data-ttu-id="5792c-644">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="5792c-644">Additional resources</span></span>

> [!div class="step-by-step"]
> <span data-ttu-id="5792c-645">[Precedente: introduzione](xref:tutorials/razor-pages/razor-pages-start) 
>  Passaggio [successivo: impalcatura Razor Pagine](xref:tutorials/razor-pages/page) di</span><span class="sxs-lookup"><span data-stu-id="5792c-645">[Previous: Get Started](xref:tutorials/razor-pages/razor-pages-start)
[Next: Scaffolded Razor Pages](xref:tutorials/razor-pages/page)</span></span>

::: moniker-end
