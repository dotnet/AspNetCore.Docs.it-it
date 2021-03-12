---
title: Usare LibMan con ASP.NET Core in Visual Studio
author: scottaddie
description: Informazioni su come usare LibMan in un progetto ASP.NET Core con Visual Studio.
ms.author: scaddie
ms.custom: mvc
ms.date: 08/20/2018
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
uid: client-side/libman/libman-vs
ms.openlocfilehash: ae2bc34edaea2df6f329e47b00c7c02cc59d03bd
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102589270"
---
# <a name="use-libman-with-aspnet-core-in-visual-studio"></a><span data-ttu-id="29df2-103">Usare LibMan con ASP.NET Core in Visual Studio</span><span class="sxs-lookup"><span data-stu-id="29df2-103">Use LibMan with ASP.NET Core in Visual Studio</span></span>

<span data-ttu-id="29df2-104">Di [Scott Addie](https://twitter.com/Scott_Addie)</span><span class="sxs-lookup"><span data-stu-id="29df2-104">By [Scott Addie](https://twitter.com/Scott_Addie)</span></span>

<span data-ttu-id="29df2-105">Visual Studio include il supporto incorporato per [LibMan](xref:client-side/libman/index) in progetti di ASP.NET Core, tra cui:</span><span class="sxs-lookup"><span data-stu-id="29df2-105">Visual Studio has built-in support for [LibMan](xref:client-side/libman/index) in ASP.NET Core projects, including:</span></span>

* <span data-ttu-id="29df2-106">Supporto per la configurazione e l'esecuzione di operazioni di ripristino di LibMan durante la compilazione.</span><span class="sxs-lookup"><span data-stu-id="29df2-106">Support for configuring and running LibMan restore operations on build.</span></span>
* <span data-ttu-id="29df2-107">Voci di menu per l'attivazione delle operazioni di ripristino e pulizia del LibMan.</span><span class="sxs-lookup"><span data-stu-id="29df2-107">Menu items for triggering LibMan restore and clean operations.</span></span>
* <span data-ttu-id="29df2-108">Finestra di dialogo Cerca per la ricerca di librerie e l'aggiunta di file a un progetto.</span><span class="sxs-lookup"><span data-stu-id="29df2-108">Search dialog for finding libraries and adding the files to a project.</span></span>
* <span data-ttu-id="29df2-109">Modifica del supporto per *libman.jsnel* &mdash; file manifesto LibMan.</span><span class="sxs-lookup"><span data-stu-id="29df2-109">Editing support for *libman.json*&mdash;the LibMan manifest file.</span></span>

<span data-ttu-id="29df2-110">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/client-side/libman/samples/) [(procedura per il download)](xref:index#how-to-download-a-sample)</span><span class="sxs-lookup"><span data-stu-id="29df2-110">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/client-side/libman/samples/) [(how to download)](xref:index#how-to-download-a-sample)</span></span>

## <a name="prerequisites"></a><span data-ttu-id="29df2-111">Prerequisiti</span><span class="sxs-lookup"><span data-stu-id="29df2-111">Prerequisites</span></span>

* <span data-ttu-id="29df2-112">[Visual Studio 2019](https://visualstudio.microsoft.com/downloads/?utm_medium=microsoft&utm_source=docs.microsoft.com&utm_campaign=inline+link&utm_content=download+vs2019) con il carico di lavoro **sviluppo di ASP.NET e Web**</span><span class="sxs-lookup"><span data-stu-id="29df2-112">[Visual Studio 2019](https://visualstudio.microsoft.com/downloads/?utm_medium=microsoft&utm_source=docs.microsoft.com&utm_campaign=inline+link&utm_content=download+vs2019) with the **ASP.NET and web development** workload</span></span>

## <a name="add-library-files"></a><span data-ttu-id="29df2-113">Aggiungi file di libreria</span><span class="sxs-lookup"><span data-stu-id="29df2-113">Add library files</span></span>

<span data-ttu-id="29df2-114">I file di libreria possono essere aggiunti a un progetto ASP.NET Core in due modi diversi:</span><span class="sxs-lookup"><span data-stu-id="29df2-114">Library files can be added to an ASP.NET Core project in two different ways:</span></span>

1. [<span data-ttu-id="29df2-115">Utilizzare la finestra di dialogo Aggiungi libreria Client-Side</span><span class="sxs-lookup"><span data-stu-id="29df2-115">Use the Add Client-Side Library dialog</span></span>](#use-the-add-client-side-library-dialog)
1. [<span data-ttu-id="29df2-116">Configurare manualmente le voci del file manifesto LibMan</span><span class="sxs-lookup"><span data-stu-id="29df2-116">Manually configure LibMan manifest file entries</span></span>](#manually-configure-libman-manifest-file-entries)

### <a name="use-the-add-client-side-library-dialog"></a><span data-ttu-id="29df2-117">Utilizzare la finestra di dialogo Aggiungi libreria Client-Side</span><span class="sxs-lookup"><span data-stu-id="29df2-117">Use the Add Client-Side Library dialog</span></span>

<span data-ttu-id="29df2-118">Per installare una libreria sul lato client, attenersi alla procedura seguente:</span><span class="sxs-lookup"><span data-stu-id="29df2-118">Follow these steps to install a client-side library:</span></span>

* <span data-ttu-id="29df2-119">In **Esplora soluzioni** fare clic con il pulsante destro del mouse sulla cartella del progetto in cui devono essere aggiunti i file.</span><span class="sxs-lookup"><span data-stu-id="29df2-119">In **Solution Explorer**, right-click the project folder in which the files should be added.</span></span> <span data-ttu-id="29df2-120">Scegliere **Aggiungi**  >  **libreria sul lato client**.</span><span class="sxs-lookup"><span data-stu-id="29df2-120">Choose **Add** > **Client-Side Library**.</span></span> <span data-ttu-id="29df2-121">Viene visualizzata la finestra di dialogo **Aggiungi libreria Client-Side** :</span><span class="sxs-lookup"><span data-stu-id="29df2-121">The **Add Client-Side Library** dialog appears:</span></span>

  ![Finestra di dialogo Aggiungi libreria Client-Side](_static/add-library-dialog.png)

* <span data-ttu-id="29df2-123">Selezionare il provider di librerie dall'elenco a discesa **provider** .</span><span class="sxs-lookup"><span data-stu-id="29df2-123">Select the library provider from the **Provider** drop down.</span></span> <span data-ttu-id="29df2-124">CDNJS è il provider predefinito.</span><span class="sxs-lookup"><span data-stu-id="29df2-124">CDNJS is the default provider.</span></span>
* <span data-ttu-id="29df2-125">Digitare il nome della libreria da recuperare nella casella di testo **libreria** .</span><span class="sxs-lookup"><span data-stu-id="29df2-125">Type the library name to fetch in the **Library** text box.</span></span> <span data-ttu-id="29df2-126">IntelliSense fornisce un elenco di librerie che iniziano con il testo fornito.</span><span class="sxs-lookup"><span data-stu-id="29df2-126">IntelliSense provides a list of libraries beginning with the provided text.</span></span>
* <span data-ttu-id="29df2-127">Selezionare la libreria dall'elenco IntelliSense.</span><span class="sxs-lookup"><span data-stu-id="29df2-127">Select the library from the IntelliSense list.</span></span> <span data-ttu-id="29df2-128">Si noti che il nome della libreria è suffisso con il `@` simbolo e la versione stabile più recente nota al provider selezionato.</span><span class="sxs-lookup"><span data-stu-id="29df2-128">Notice the library name is suffixed with the `@` symbol and the latest stable version known to the selected provider.</span></span>
* <span data-ttu-id="29df2-129">Decidere quali file includere:</span><span class="sxs-lookup"><span data-stu-id="29df2-129">Decide which files to include:</span></span>
  * <span data-ttu-id="29df2-130">Selezionare il pulsante di opzione **Includi tutti i file di libreria** per includere tutti i file della libreria.</span><span class="sxs-lookup"><span data-stu-id="29df2-130">Select the **Include all library files** radio button to include all of the library's files.</span></span>
  * <span data-ttu-id="29df2-131">Selezionare il pulsante di opzione **Scegli file specifici** per includere un subset dei file della libreria.</span><span class="sxs-lookup"><span data-stu-id="29df2-131">Select the **Choose specific files** radio button to include a subset of the library's files.</span></span> <span data-ttu-id="29df2-132">Quando il pulsante di opzione è selezionato, la struttura ad albero del selettore file è abilitata.</span><span class="sxs-lookup"><span data-stu-id="29df2-132">When the radio button is selected, the file selector tree is enabled.</span></span> <span data-ttu-id="29df2-133">Selezionare le caselle a sinistra dei nomi file da scaricare.</span><span class="sxs-lookup"><span data-stu-id="29df2-133">Check the boxes to the left of the file names to download.</span></span>
* <span data-ttu-id="29df2-134">Specificare la cartella di progetto per l'archiviazione dei file nella casella di testo **percorso di destinazione** .</span><span class="sxs-lookup"><span data-stu-id="29df2-134">Specify the project folder for storing the files in the **Target Location** text box.</span></span> <span data-ttu-id="29df2-135">Come raccomandazione, archiviare ogni libreria in una cartella separata.</span><span class="sxs-lookup"><span data-stu-id="29df2-135">As a recommendation, store each library in a separate folder.</span></span>

  <span data-ttu-id="29df2-136">La cartella del **percorso di destinazione** suggerita è basata sul percorso da cui è stata avviata la finestra di dialogo:</span><span class="sxs-lookup"><span data-stu-id="29df2-136">The suggested **Target Location** folder is based on the location from which the dialog launched:</span></span>

  * <span data-ttu-id="29df2-137">Se avviato dalla radice del progetto:</span><span class="sxs-lookup"><span data-stu-id="29df2-137">If launched from the project root:</span></span>
    * <span data-ttu-id="29df2-138">Se *wwwroot* esiste, viene usato *wwwroot/lib* .</span><span class="sxs-lookup"><span data-stu-id="29df2-138">*wwwroot/lib* is used if *wwwroot* exists.</span></span>
    * <span data-ttu-id="29df2-139">*lib* viene usato se *wwwroot* non esiste.</span><span class="sxs-lookup"><span data-stu-id="29df2-139">*lib* is used if *wwwroot* doesn't exist.</span></span>
  * <span data-ttu-id="29df2-140">Se avviato da una cartella del progetto, viene usato il nome della cartella corrispondente.</span><span class="sxs-lookup"><span data-stu-id="29df2-140">If launched from a project folder, the corresponding folder name is used.</span></span>

  <span data-ttu-id="29df2-141">Il suggerimento della cartella è con suffisso con il nome della libreria.</span><span class="sxs-lookup"><span data-stu-id="29df2-141">The folder suggestion is suffixed with the library name.</span></span> <span data-ttu-id="29df2-142">Nella tabella seguente vengono illustrati i suggerimenti per le cartelle quando si installa jQuery in un Razor progetto di pagine.</span><span class="sxs-lookup"><span data-stu-id="29df2-142">The following table illustrates folder suggestions when installing jQuery in a Razor Pages project.</span></span>
  
  |<span data-ttu-id="29df2-143">Percorso di avvio</span><span class="sxs-lookup"><span data-stu-id="29df2-143">Launch location</span></span>                           |<span data-ttu-id="29df2-144">Cartella consigliata</span><span class="sxs-lookup"><span data-stu-id="29df2-144">Suggested folder</span></span>      |
  |------------------------------------------|----------------------|
  |<span data-ttu-id="29df2-145">radice del progetto (se *wwwroot* esiste)</span><span class="sxs-lookup"><span data-stu-id="29df2-145">project root (if *wwwroot* exists)</span></span>        |<span data-ttu-id="29df2-146">*Wwwroot/lib/jQuery/*</span><span class="sxs-lookup"><span data-stu-id="29df2-146">*wwwroot/lib/jquery/*</span></span> |
  |<span data-ttu-id="29df2-147">radice del progetto (se *wwwroot* non esiste)</span><span class="sxs-lookup"><span data-stu-id="29df2-147">project root (if *wwwroot* doesn't exist)</span></span> |<span data-ttu-id="29df2-148">*lib/jQuery/*</span><span class="sxs-lookup"><span data-stu-id="29df2-148">*lib/jquery/*</span></span>         |
  |<span data-ttu-id="29df2-149">Cartella *pages* nel progetto</span><span class="sxs-lookup"><span data-stu-id="29df2-149">*Pages* folder in project</span></span>                 |<span data-ttu-id="29df2-150">*Pagine/jQuery/*</span><span class="sxs-lookup"><span data-stu-id="29df2-150">*Pages/jquery/*</span></span>       |

* <span data-ttu-id="29df2-151">Fare clic sul pulsante **Install (installa** ) per scaricare i file, in base alla configurazione in *libman.js*.</span><span class="sxs-lookup"><span data-stu-id="29df2-151">Click the **Install** button to download the files, per the configuration in *libman.json*.</span></span>
* <span data-ttu-id="29df2-152">Per informazioni sull'installazione, vedere il feed di **Gestione librerie** della finestra di **output** .</span><span class="sxs-lookup"><span data-stu-id="29df2-152">Review the **Library Manager** feed of the **Output** window for installation details.</span></span> <span data-ttu-id="29df2-153">Ad esempio:</span><span class="sxs-lookup"><span data-stu-id="29df2-153">For example:</span></span>

  ```console
  Restore operation started...
  Restoring libraries for project LibManSample
  Restoring library jquery@3.3.1... (LibManSample)
  wwwroot/lib/jquery/jquery.min.js written to destination (LibManSample)
  wwwroot/lib/jquery/jquery.js written to destination (LibManSample)
  wwwroot/lib/jquery/jquery.min.map written to destination (LibManSample)
  Restore operation completed
  1 libraries restored in 2.32 seconds
  ```

### <a name="manually-configure-libman-manifest-file-entries"></a><span data-ttu-id="29df2-154">Configurare manualmente le voci del file manifesto LibMan</span><span class="sxs-lookup"><span data-stu-id="29df2-154">Manually configure LibMan manifest file entries</span></span>

<span data-ttu-id="29df2-155">Tutte le operazioni LibMan in Visual Studio sono basate sul contenuto del manifesto LibMan della radice del progetto (*libman.json*).</span><span class="sxs-lookup"><span data-stu-id="29df2-155">All LibMan operations in Visual Studio are based on the content of the project root's LibMan manifest (*libman.json*).</span></span> <span data-ttu-id="29df2-156">È possibile modificare manualmente *libman.jssu* per configurare i file di libreria per il progetto.</span><span class="sxs-lookup"><span data-stu-id="29df2-156">You can manually edit *libman.json* to configure library files for the project.</span></span> <span data-ttu-id="29df2-157">Quandolibman.jsviene salvato, Visual Studio Ripristina tutti i file *di* libreria.</span><span class="sxs-lookup"><span data-stu-id="29df2-157">Visual Studio restores all library files once *libman.json* is saved.</span></span>

<span data-ttu-id="29df2-158">Per aprire *libman.js* per la modifica, sono disponibili le opzioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="29df2-158">To open *libman.json* for editing, the following options exist:</span></span>

* <span data-ttu-id="29df2-159">Fare doppio clic sul file *libman.js* in **Esplora soluzioni**.</span><span class="sxs-lookup"><span data-stu-id="29df2-159">Double-click the *libman.json* file in **Solution Explorer**.</span></span>
* <span data-ttu-id="29df2-160">Fare clic con il pulsante destro del mouse sul progetto **Esplora soluzioni** e scegliere **gestisci librerie Client-Side**.</span><span class="sxs-lookup"><span data-stu-id="29df2-160">Right-click the project in **Solution Explorer** and select **Manage Client-Side Libraries**.</span></span> <span data-ttu-id="29df2-161">**&#8224;**</span><span class="sxs-lookup"><span data-stu-id="29df2-161">**&#8224;**</span></span>
* <span data-ttu-id="29df2-162">Selezionare **Gestisci librerie Client-Side** dal menu **progetto** di Visual Studio.</span><span class="sxs-lookup"><span data-stu-id="29df2-162">Select **Manage Client-Side Libraries** from the Visual Studio **Project** menu.</span></span> <span data-ttu-id="29df2-163">**&#8224;**</span><span class="sxs-lookup"><span data-stu-id="29df2-163">**&#8224;**</span></span>

<span data-ttu-id="29df2-164">**&#8224;** Se il *libman.js* nel file non esiste già nella radice del progetto, verrà creato con il contenuto del modello di elemento predefinito.</span><span class="sxs-lookup"><span data-stu-id="29df2-164">**&#8224;** If the *libman.json* file doesn't already exist in the project root, it will be created with the default item template content.</span></span>

<span data-ttu-id="29df2-165">Visual Studio offre supporto avanzato per la modifica di JSON, ad esempio la colorazione, la formattazione, IntelliSense e la convalida dello schema.</span><span class="sxs-lookup"><span data-stu-id="29df2-165">Visual Studio offers rich JSON editing support such as colorization, formatting, IntelliSense, and schema validation.</span></span> <span data-ttu-id="29df2-166">Lo schema JSON del manifesto LibMan si trova in [https://json.schemastore.org/libman](https://json.schemastore.org/libman) .</span><span class="sxs-lookup"><span data-stu-id="29df2-166">The LibMan manifest's JSON schema is found at [https://json.schemastore.org/libman](https://json.schemastore.org/libman).</span></span>

<span data-ttu-id="29df2-167">Con il file manifesto seguente, LibMan recupera i file in base alla configurazione definita nella `libraries` Proprietà.</span><span class="sxs-lookup"><span data-stu-id="29df2-167">With the following manifest file, LibMan retrieves files per the configuration defined in the `libraries` property.</span></span> <span data-ttu-id="29df2-168">Una spiegazione dei valori letterali dell'oggetto definiti nei `libraries` seguenti modi:</span><span class="sxs-lookup"><span data-stu-id="29df2-168">An explanation of the object literals defined within `libraries` follows:</span></span>

* <span data-ttu-id="29df2-169">Un subset di [jQuery](https://jquery.com/) versione 3.3.1 viene recuperato dal provider CDNJS.</span><span class="sxs-lookup"><span data-stu-id="29df2-169">A subset of [jQuery](https://jquery.com/) version 3.3.1 is retrieved from the CDNJS provider.</span></span> <span data-ttu-id="29df2-170">Il subset viene definito nella `files` proprietà &mdash; *jquery.min.js*, *jquery.js* e *jQuery. min. map*.</span><span class="sxs-lookup"><span data-stu-id="29df2-170">The subset is defined in the `files` property&mdash;*jquery.min.js*, *jquery.js*, and *jquery.min.map*.</span></span> <span data-ttu-id="29df2-171">I file vengono inseriti nella cartella *wwwroot/lib/jQuery* del progetto.</span><span class="sxs-lookup"><span data-stu-id="29df2-171">The files are placed in the project's *wwwroot/lib/jquery* folder.</span></span>
* <span data-ttu-id="29df2-172">L'intera versione di [bootstrap](https://getbootstrap.com/) 4.1.3 viene recuperata e inserita in una cartella *wwwroot/lib/bootstrap* .</span><span class="sxs-lookup"><span data-stu-id="29df2-172">The entirety of [Bootstrap](https://getbootstrap.com/) version 4.1.3 is retrieved and placed in a *wwwroot/lib/bootstrap* folder.</span></span> <span data-ttu-id="29df2-173">La proprietà del valore letterale dell'oggetto `provider` esegue l'override del `defaultProvider` valore della proprietà.</span><span class="sxs-lookup"><span data-stu-id="29df2-173">The object literal's `provider` property overrides the `defaultProvider` property value.</span></span> <span data-ttu-id="29df2-174">LibMan recupera i file bootstrap dal provider unpkg.</span><span class="sxs-lookup"><span data-stu-id="29df2-174">LibMan retrieves the Bootstrap files from the unpkg provider.</span></span>
* <span data-ttu-id="29df2-175">Un subset di [Lodash](https://lodash.com/) è stato approvato da un corpo di governance all'interno dell'organizzazione.</span><span class="sxs-lookup"><span data-stu-id="29df2-175">A subset of [Lodash](https://lodash.com/) was approved by a governing body within the organization.</span></span> <span data-ttu-id="29df2-176">I file di *lodash.js* e di *lodash.min.js* vengono recuperati dal file system locale in *C: \\ temp \\ lodash \\*.</span><span class="sxs-lookup"><span data-stu-id="29df2-176">The *lodash.js* and *lodash.min.js* files are retrieved from the local file system at *C:\\temp\\lodash\\*.</span></span> <span data-ttu-id="29df2-177">I file vengono copiati nella cartella *wwwroot/lib/lodash* del progetto.</span><span class="sxs-lookup"><span data-stu-id="29df2-177">The files are copied to the project's *wwwroot/lib/lodash* folder.</span></span>

[!code-json[](samples/LibManSample/libman.json)]

> [!NOTE]
> <span data-ttu-id="29df2-178">LibMan supporta solo una versione di ogni libreria di ogni provider.</span><span class="sxs-lookup"><span data-stu-id="29df2-178">LibMan only supports one version of each library from each provider.</span></span> <span data-ttu-id="29df2-179">La convalida dello schema del *libman.jssu* file non riesce se contiene due librerie con lo stesso nome di libreria per un determinato provider.</span><span class="sxs-lookup"><span data-stu-id="29df2-179">The *libman.json* file fails schema validation if it contains two libraries with the same library name for a given provider.</span></span>

## <a name="restore-library-files"></a><span data-ttu-id="29df2-180">Ripristinare i file di libreria</span><span class="sxs-lookup"><span data-stu-id="29df2-180">Restore library files</span></span>

<span data-ttu-id="29df2-181">Per ripristinare i file di libreria da Visual Studio, è necessario che sia presente un *libman.js* valido nel file nella radice del progetto.</span><span class="sxs-lookup"><span data-stu-id="29df2-181">To restore library files from within Visual Studio, there must be a valid *libman.json* file in the project root.</span></span> <span data-ttu-id="29df2-182">I file ripristinati vengono inseriti nel progetto nel percorso specificato per ogni libreria.</span><span class="sxs-lookup"><span data-stu-id="29df2-182">Restored files are placed in the project at the location specified for each library.</span></span>

<span data-ttu-id="29df2-183">I file di libreria possono essere ripristinati in un progetto ASP.NET Core in due modi:</span><span class="sxs-lookup"><span data-stu-id="29df2-183">Library files can be restored in an ASP.NET Core project in two ways:</span></span>

1. [<span data-ttu-id="29df2-184">Ripristinare i file durante la compilazione</span><span class="sxs-lookup"><span data-stu-id="29df2-184">Restore files during build</span></span>](#restore-files-during-build)
1. [<span data-ttu-id="29df2-185">Ripristino manuale dei file</span><span class="sxs-lookup"><span data-stu-id="29df2-185">Restore files manually</span></span>](#restore-files-manually)

### <a name="restore-files-during-build"></a><span data-ttu-id="29df2-186">Ripristinare i file durante la compilazione</span><span class="sxs-lookup"><span data-stu-id="29df2-186">Restore files during build</span></span>

<span data-ttu-id="29df2-187">LibMan consente di ripristinare i file di libreria definiti come parte del processo di compilazione.</span><span class="sxs-lookup"><span data-stu-id="29df2-187">LibMan can restore the defined library files as part of the build process.</span></span> <span data-ttu-id="29df2-188">Per impostazione predefinita, il comportamento di *ripristino su compilazione* è disabilitato.</span><span class="sxs-lookup"><span data-stu-id="29df2-188">By default, the *restore-on-build* behavior is disabled.</span></span>

<span data-ttu-id="29df2-189">Per abilitare e testare il comportamento di ripristino in base alla compilazione:</span><span class="sxs-lookup"><span data-stu-id="29df2-189">To enable and test the restore-on-build behavior:</span></span>

* <span data-ttu-id="29df2-190">Fare clic con il pulsante destro del mouse *sulibman.js* in **Esplora soluzioni** e selezionare **Abilita ripristino Client-Side librerie in compilazione** dal menu di scelta rapida.</span><span class="sxs-lookup"><span data-stu-id="29df2-190">Right-click *libman.json* in **Solution Explorer** and select **Enable Restore Client-Side Libraries on Build** from the context menu.</span></span>
* <span data-ttu-id="29df2-191">Quando viene richiesto di installare un pacchetto NuGet, fare clic sul pulsante **Sì** .</span><span class="sxs-lookup"><span data-stu-id="29df2-191">Click the **Yes** button when prompted to install a NuGet package.</span></span> <span data-ttu-id="29df2-192">Al progetto viene aggiunto il pacchetto NuGet [Microsoft. Web. librarymanager. Build](https://www.nuget.org/packages/Microsoft.Web.LibraryManager.Build/) :</span><span class="sxs-lookup"><span data-stu-id="29df2-192">The [Microsoft.Web.LibraryManager.Build](https://www.nuget.org/packages/Microsoft.Web.LibraryManager.Build/) NuGet package is added to the project:</span></span>

  [!code-xml[](samples/LibManSample/LibManSample.csproj?name=snippet_RestoreOnBuildPackage)]

* <span data-ttu-id="29df2-193">Compilare il progetto per confermare che si verifica il ripristino del file LibMan.</span><span class="sxs-lookup"><span data-stu-id="29df2-193">Build the project to confirm LibMan file restoration occurs.</span></span> <span data-ttu-id="29df2-194">Il `Microsoft.Web.LibraryManager.Build` pacchetto inserisce una destinazione MSBuild che esegue LibMan durante l'operazione di compilazione del progetto.</span><span class="sxs-lookup"><span data-stu-id="29df2-194">The `Microsoft.Web.LibraryManager.Build` package injects an MSBuild target that runs LibMan during the project's build operation.</span></span>
* <span data-ttu-id="29df2-195">Esaminare il feed di **compilazione** della finestra di **output** per un log attività LibMan:</span><span class="sxs-lookup"><span data-stu-id="29df2-195">Review the **Build** feed of the **Output** window for a LibMan activity log:</span></span>

  ```console
  1>------ Build started: Project: LibManSample, Configuration: Debug Any CPU ------
  1>
  1>Restore operation started...
  1>Restoring library jquery@3.3.1...
  1>Restoring library bootstrap@4.1.3...
  1>
  1>2 libraries restored in 10.66 seconds
  1>LibManSample -> C:\LibManSample\bin\Debug\netcoreapp2.1\LibManSample.dll
  ========== Build: 1 succeeded, 0 failed, 0 up-to-date, 0 skipped ==========
  ```

<span data-ttu-id="29df2-196">Quando è abilitato il comportamento di ripristino in fase di compilazione, nel menu di scelta rapida *libman.js* viene visualizzata l'opzione **Disable Restore Client-Side Libraries on Build** .</span><span class="sxs-lookup"><span data-stu-id="29df2-196">When the restore-on-build behavior is enabled, the *libman.json* context menu displays a **Disable Restore Client-Side Libraries on Build** option.</span></span> <span data-ttu-id="29df2-197">Se si seleziona questa opzione `Microsoft.Web.LibraryManager.Build` , il riferimento al pacchetto verrà rimosso dal file di progetto.</span><span class="sxs-lookup"><span data-stu-id="29df2-197">Selecting this option removes the `Microsoft.Web.LibraryManager.Build` package reference from the project file.</span></span> <span data-ttu-id="29df2-198">Di conseguenza, le librerie sul lato client non vengono più ripristinate in ogni compilazione.</span><span class="sxs-lookup"><span data-stu-id="29df2-198">Consequently, the client-side libraries are no longer restored on each build.</span></span>

<span data-ttu-id="29df2-199">Indipendentemente dall'impostazione di ripristino in base alla compilazione, è possibile eseguire manualmente il ripristino in qualsiasi momento dal menu di scelta rapida *libman.js* .</span><span class="sxs-lookup"><span data-stu-id="29df2-199">Regardless of the restore-on-build setting, you can manually restore at any time from the *libman.json* context menu.</span></span> <span data-ttu-id="29df2-200">Per ulteriori informazioni, vedere [ripristino manuale dei file](#restore-files-manually).</span><span class="sxs-lookup"><span data-stu-id="29df2-200">For more information, see [Restore files manually](#restore-files-manually).</span></span>

### <a name="restore-files-manually"></a><span data-ttu-id="29df2-201">Ripristino manuale dei file</span><span class="sxs-lookup"><span data-stu-id="29df2-201">Restore files manually</span></span>

<span data-ttu-id="29df2-202">Per ripristinare manualmente i file di libreria:</span><span class="sxs-lookup"><span data-stu-id="29df2-202">To manually restore library files:</span></span>

* <span data-ttu-id="29df2-203">Per tutti i progetti nella soluzione:</span><span class="sxs-lookup"><span data-stu-id="29df2-203">For all projects in the solution:</span></span>
  * <span data-ttu-id="29df2-204">Fare clic con il pulsante destro del mouse sul nome della soluzione in **Esplora soluzioni**.</span><span class="sxs-lookup"><span data-stu-id="29df2-204">Right-click the solution name in **Solution Explorer**.</span></span>
  * <span data-ttu-id="29df2-205">Selezionare l'opzione **restore Client-Side Libraries** .</span><span class="sxs-lookup"><span data-stu-id="29df2-205">Select the **Restore Client-Side Libraries** option.</span></span>
* <span data-ttu-id="29df2-206">Per un progetto specifico:</span><span class="sxs-lookup"><span data-stu-id="29df2-206">For a specific project:</span></span>
  * <span data-ttu-id="29df2-207">Fare clic con il pulsante destro del mouse sul file *libman.js* in **Esplora soluzioni**.</span><span class="sxs-lookup"><span data-stu-id="29df2-207">Right-click the *libman.json* file in **Solution Explorer**.</span></span>
  * <span data-ttu-id="29df2-208">Selezionare l'opzione **restore Client-Side Libraries** .</span><span class="sxs-lookup"><span data-stu-id="29df2-208">Select the **Restore Client-Side Libraries** option.</span></span>

<span data-ttu-id="29df2-209">Durante l'esecuzione dell'operazione di ripristino:</span><span class="sxs-lookup"><span data-stu-id="29df2-209">While the restore operation is running:</span></span>

* <span data-ttu-id="29df2-210">L'icona centro stato attività (TSC) sulla barra di stato di Visual Studio verrà animata e l'operazione di ripristino verrà letta *avviata*.</span><span class="sxs-lookup"><span data-stu-id="29df2-210">The Task Status Center (TSC) icon on the Visual Studio status bar will be animated and will read *Restore operation started*.</span></span> <span data-ttu-id="29df2-211">Facendo clic sull'icona si apre una descrizione comando che elenca le attività in background note.</span><span class="sxs-lookup"><span data-stu-id="29df2-211">Clicking the icon opens a tooltip listing the known background tasks.</span></span>
* <span data-ttu-id="29df2-212">I messaggi verranno inviati alla barra di stato e al feed di **Gestione librerie** della finestra di **output** .</span><span class="sxs-lookup"><span data-stu-id="29df2-212">Messages will be sent to the status bar and the **Library Manager** feed of the **Output** window.</span></span> <span data-ttu-id="29df2-213">Ad esempio:</span><span class="sxs-lookup"><span data-stu-id="29df2-213">For example:</span></span>

  ```console
  Restore operation started...
  Restoring libraries for project LibManSample
  Restoring library jquery@3.3.1... (LibManSample)
  wwwroot/lib/jquery/jquery.min.js written to destination (LibManSample)
  wwwroot/lib/jquery/jquery.js written to destination (LibManSample)
  wwwroot/lib/jquery/jquery.min.map written to destination (LibManSample)
  Restore operation completed
  1 libraries restored in 2.32 seconds
  ```

## <a name="delete-library-files"></a><span data-ttu-id="29df2-214">Elimina file di libreria</span><span class="sxs-lookup"><span data-stu-id="29df2-214">Delete library files</span></span>

<span data-ttu-id="29df2-215">Per eseguire l'operazione di *pulizia* , che elimina i file di libreria ripristinati in precedenza in Visual Studio:</span><span class="sxs-lookup"><span data-stu-id="29df2-215">To perform the *clean* operation, which deletes library files previously restored in Visual Studio:</span></span>

* <span data-ttu-id="29df2-216">Fare clic con il pulsante destro del mouse sul file *libman.js* in **Esplora soluzioni**.</span><span class="sxs-lookup"><span data-stu-id="29df2-216">Right-click the *libman.json* file in **Solution Explorer**.</span></span>
* <span data-ttu-id="29df2-217">Selezionare l'opzione **pulisci Client-Side librerie** .</span><span class="sxs-lookup"><span data-stu-id="29df2-217">Select the **Clean Client-Side Libraries** option.</span></span>

<span data-ttu-id="29df2-218">Per evitare la rimozione accidentale di file non di libreria, l'operazione di pulizia non Elimina intere directory.</span><span class="sxs-lookup"><span data-stu-id="29df2-218">To prevent unintentional removal of non-library files, the clean operation doesn't delete whole directories.</span></span> <span data-ttu-id="29df2-219">Rimuove solo i file inclusi nel ripristino precedente.</span><span class="sxs-lookup"><span data-stu-id="29df2-219">It only removes files that were included in the previous restore.</span></span>

<span data-ttu-id="29df2-220">Durante l'esecuzione dell'operazione di pulizia:</span><span class="sxs-lookup"><span data-stu-id="29df2-220">While the clean operation is running:</span></span>

* <span data-ttu-id="29df2-221">L'icona TSC sulla barra di stato di Visual Studio verrà animata e verrà letta l' *operazione delle librerie client avviata*.</span><span class="sxs-lookup"><span data-stu-id="29df2-221">The TSC icon on the Visual Studio status bar will be animated and will read *Client libraries operation started*.</span></span> <span data-ttu-id="29df2-222">Facendo clic sull'icona si apre una descrizione comando che elenca le attività in background note.</span><span class="sxs-lookup"><span data-stu-id="29df2-222">Clicking the icon opens a tooltip listing the known background tasks.</span></span>
* <span data-ttu-id="29df2-223">I messaggi vengono inviati alla barra di stato e al feed di **Gestione librerie** della finestra di **output** .</span><span class="sxs-lookup"><span data-stu-id="29df2-223">Messages are sent to the status bar and the **Library Manager** feed of the **Output** window.</span></span> <span data-ttu-id="29df2-224">Ad esempio:</span><span class="sxs-lookup"><span data-stu-id="29df2-224">For example:</span></span>

```console
Clean libraries operation started...
Clean libraries operation completed
2 libraries were successfully deleted in 1.91 secs
```

<span data-ttu-id="29df2-225">L'operazione di pulizia elimina solo i file dal progetto.</span><span class="sxs-lookup"><span data-stu-id="29df2-225">The clean operation only deletes files from the project.</span></span> <span data-ttu-id="29df2-226">I file di libreria vengono mantenuti nella cache per un recupero più rapido delle operazioni di ripristino future.</span><span class="sxs-lookup"><span data-stu-id="29df2-226">Library files stay in the cache for faster retrieval on future restore operations.</span></span> <span data-ttu-id="29df2-227">Per gestire i file di libreria archiviati nella cache del computer locale, usare l'interfaccia della riga di comando di [LibMan](xref:client-side/libman/libman-cli).</span><span class="sxs-lookup"><span data-stu-id="29df2-227">To manage library files stored in the local machine's cache, use the [LibMan CLI](xref:client-side/libman/libman-cli).</span></span>

## <a name="uninstall-library-files"></a><span data-ttu-id="29df2-228">Disinstalla file di libreria</span><span class="sxs-lookup"><span data-stu-id="29df2-228">Uninstall library files</span></span>

<span data-ttu-id="29df2-229">Per disinstallare i file di libreria:</span><span class="sxs-lookup"><span data-stu-id="29df2-229">To uninstall library files:</span></span>

* <span data-ttu-id="29df2-230">Aprire *libman.js*.</span><span class="sxs-lookup"><span data-stu-id="29df2-230">Open *libman.json*.</span></span>
* <span data-ttu-id="29df2-231">Posizionare il punto di inserimento all'interno del `libraries` valore letterale dell'oggetto corrispondente.</span><span class="sxs-lookup"><span data-stu-id="29df2-231">Position the caret inside the corresponding `libraries` object literal.</span></span>
* <span data-ttu-id="29df2-232">Fare clic sull'icona a bulbo chiaro visualizzata nel margine sinistro e selezionare **Disinstalla \<library_name> @ \<library_version>**:</span><span class="sxs-lookup"><span data-stu-id="29df2-232">Click the light bulb icon that appears in the left margin, and select **Uninstall \<library_name>@\<library_version>**:</span></span>

  ![Opzione del menu di scelta rapida Disinstalla libreria](_static/uninstall-menu-option.png)

<span data-ttu-id="29df2-234">In alternativa, è possibile modificare e salvare manualmente il manifesto LibMan (*libman.json*).</span><span class="sxs-lookup"><span data-stu-id="29df2-234">Alternatively, you can manually edit and save the LibMan manifest (*libman.json*).</span></span> <span data-ttu-id="29df2-235">L' [operazione di ripristino](#restore-library-files) viene eseguita quando il file viene salvato.</span><span class="sxs-lookup"><span data-stu-id="29df2-235">The [restore operation](#restore-library-files) runs when the file is saved.</span></span> <span data-ttu-id="29df2-236">I file di libreria che non sono più definiti in *libman.json* vengono rimossi dal progetto.</span><span class="sxs-lookup"><span data-stu-id="29df2-236">Library files that are no longer defined in *libman.json* are removed from the project.</span></span>

## <a name="update-library-version"></a><span data-ttu-id="29df2-237">Aggiornare la versione della libreria</span><span class="sxs-lookup"><span data-stu-id="29df2-237">Update library version</span></span>

<span data-ttu-id="29df2-238">Per verificare la presenza di una versione aggiornata della libreria:</span><span class="sxs-lookup"><span data-stu-id="29df2-238">To check for an updated library version:</span></span>

* <span data-ttu-id="29df2-239">Aprire *libman.js*.</span><span class="sxs-lookup"><span data-stu-id="29df2-239">Open *libman.json*.</span></span>
* <span data-ttu-id="29df2-240">Posizionare il punto di inserimento all'interno del `libraries` valore letterale dell'oggetto corrispondente.</span><span class="sxs-lookup"><span data-stu-id="29df2-240">Position the caret inside the corresponding `libraries` object literal.</span></span>
* <span data-ttu-id="29df2-241">Fare clic sull'icona a bulbo chiaro visualizzata nel margine sinistro.</span><span class="sxs-lookup"><span data-stu-id="29df2-241">Click the light bulb icon that appears in the left margin.</span></span> <span data-ttu-id="29df2-242">Passare il mouse su **Verifica disponibilità aggiornamenti**.</span><span class="sxs-lookup"><span data-stu-id="29df2-242">Hover over **Check for updates**.</span></span>

<span data-ttu-id="29df2-243">LibMan verifica la presenza di una versione della libreria più recente della versione installata.</span><span class="sxs-lookup"><span data-stu-id="29df2-243">LibMan checks for a library version newer than the version installed.</span></span> <span data-ttu-id="29df2-244">Possono verificarsi i risultati seguenti:</span><span class="sxs-lookup"><span data-stu-id="29df2-244">The following outcomes can occur:</span></span>

* <span data-ttu-id="29df2-245">Se è già installata la versione più recente, viene visualizzato un messaggio di **Nessun aggiornamento trovato** .</span><span class="sxs-lookup"><span data-stu-id="29df2-245">A **No updates found** message is displayed if the latest version is already installed.</span></span>
* <span data-ttu-id="29df2-246">Se non è già installata, viene visualizzata la versione stabile più recente.</span><span class="sxs-lookup"><span data-stu-id="29df2-246">The latest stable version is displayed if not already installed.</span></span>

  ![Opzione del menu di scelta rapida Controlla aggiornamenti](_static/update-menu-option.png)

* <span data-ttu-id="29df2-248">Se è disponibile una versione provvisoria più recente di quella installata, viene visualizzata la versione non definitiva.</span><span class="sxs-lookup"><span data-stu-id="29df2-248">If a pre-release newer than the installed version is available, the pre-release is displayed.</span></span>

<span data-ttu-id="29df2-249">Per eseguire il downgrade a una versione precedente della libreria, modificare manualmente il *libman.jssu* file.</span><span class="sxs-lookup"><span data-stu-id="29df2-249">To downgrade to an older library version, manually edit the *libman.json* file.</span></span> <span data-ttu-id="29df2-250">Quando il file viene salvato, l' [operazione di ripristino](#restore-library-files)LibMan:</span><span class="sxs-lookup"><span data-stu-id="29df2-250">When the file is saved, the LibMan [restore operation](#restore-library-files):</span></span>

* <span data-ttu-id="29df2-251">Rimuove i file ridondanti dalla versione precedente.</span><span class="sxs-lookup"><span data-stu-id="29df2-251">Removes redundant files from the previous version.</span></span>
* <span data-ttu-id="29df2-252">Aggiunge file nuovi e aggiornati dalla nuova versione.</span><span class="sxs-lookup"><span data-stu-id="29df2-252">Adds new and updated files from the new version.</span></span>

## <a name="additional-resources"></a><span data-ttu-id="29df2-253">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="29df2-253">Additional resources</span></span>

* <xref:client-side/libman/libman-cli>
* [<span data-ttu-id="29df2-254">Repository GitHub di LibMan</span><span class="sxs-lookup"><span data-stu-id="29df2-254">LibMan GitHub repository</span></span>](https://github.com/aspnet/LibraryManager)
