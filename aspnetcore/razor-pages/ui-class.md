---
title: Interfaccia utente riutilizzabile Razor nelle librerie di classi con ASP.NET Core
author: Rick-Anderson
description: Viene illustrato come creare Razor un'interfaccia utente riutilizzabile usando visualizzazioni parziali in una libreria di classi in ASP.NET Core.
ms.author: riande
ms.date: 01/19/2021
ms.custom: mvc, seodec18
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
uid: razor-pages/ui-class
ms.openlocfilehash: 88b3bae5d52e334dea7ffcfe245f49d2816047c3
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102588178"
---
# <a name="create-reusable-ui-using-the-razor-class-library-project-in-aspnet-core"></a><span data-ttu-id="ce0ad-103">Creare un'interfaccia utente riutilizzabile usando il Razor progetto libreria di classi in ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="ce0ad-103">Create reusable UI using the Razor class library project in ASP.NET Core</span></span>

<span data-ttu-id="ce0ad-104">Autore: [Rick Anderson](https://twitter.com/RickAndMSFT)</span><span class="sxs-lookup"><span data-stu-id="ce0ad-104">By [Rick Anderson](https://twitter.com/RickAndMSFT)</span></span>

::: moniker range=">= aspnetcore-3.0"

<span data-ttu-id="ce0ad-105">Razorle visualizzazioni, le pagine, i controller, i modelli di pagina, i [ Razor componenti](xref:blazor/components/class-libraries), i [componenti di visualizzazione](xref:mvc/views/view-components)e i modelli di dati possono essere incorporati in una Razor libreria di classi (RCL).</span><span class="sxs-lookup"><span data-stu-id="ce0ad-105">Razor views, pages, controllers, page models, [Razor components](xref:blazor/components/class-libraries), [View components](xref:mvc/views/view-components), and data models can be built into a Razor class library (RCL).</span></span> <span data-ttu-id="ce0ad-106">La libreria di classi Razor può essere inclusa nel pacchetto e usata nuovamente.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-106">The RCL can be packaged and reused.</span></span> <span data-ttu-id="ce0ad-107">Le applicazioni possono includere la libreria di classi Razor ed eseguire l'override delle visualizzazioni e pagine in essa contenute.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-107">Applications can include the RCL and override the views and pages it contains.</span></span> <span data-ttu-id="ce0ad-108">Quando si trova una visualizzazione, una visualizzazione parziale o una Razor pagina nell'app Web e nel RCL, il Razor markup (file con *estensione cshtml* ) nell'app Web ha la precedenza.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-108">When a view, partial view, or Razor Page is found in both the web app and the RCL, the Razor markup (*.cshtml* file) in the web app takes precedence.</span></span>

<span data-ttu-id="ce0ad-109">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/razor-pages/ui-class/samples) ([procedura per il download](xref:index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="ce0ad-109">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/razor-pages/ui-class/samples) ([how to download](xref:index#how-to-download-a-sample))</span></span>

## <a name="create-a-class-library-containing-razor-ui"></a><span data-ttu-id="ce0ad-110">Creare una libreria di classi contenente l' Razor interfaccia utente</span><span class="sxs-lookup"><span data-stu-id="ce0ad-110">Create a class library containing Razor UI</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="ce0ad-111">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="ce0ad-111">Visual Studio</span></span>](#tab/visual-studio)

* <span data-ttu-id="ce0ad-112">In Visual Studio selezionare **Crea nuovo nuovo progetto**.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-112">From Visual Studio select **Create new a new project**.</span></span>
* <span data-ttu-id="ce0ad-113">Selezionare **Razor libreria di classi** > **Avanti**.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-113">Select **Razor Class Library** > **Next**.</span></span>
* <span data-ttu-id="ce0ad-114">Assegnare un nome alla libreria (ad esempio, " Razor CLASSLIB") > **creare**.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-114">Name the library (for example, "RazorClassLib"), > **Create**.</span></span> <span data-ttu-id="ce0ad-115">Per evitare un conflitto di nomi di file con la libreria di visualizzazione generata, verificare che il nome della libreria non finisca per `.Views`.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-115">To avoid a file name collision with the generated view library, ensure the library name doesn't end in `.Views`.</span></span>
* <span data-ttu-id="ce0ad-116">Se è necessario supportare le visualizzazioni, selezionare **pagine e visualizzazioni di supporto** .</span><span class="sxs-lookup"><span data-stu-id="ce0ad-116">Select **Support pages and views** if you need to support views.</span></span> <span data-ttu-id="ce0ad-117">Per impostazione predefinita, Razor sono supportate solo le pagine.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-117">By default, only Razor Pages are supported.</span></span> <span data-ttu-id="ce0ad-118">Selezionare **Crea**.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-118">Select **Create**.</span></span>

<span data-ttu-id="ce0ad-119">Per impostazione predefinita, il Razor modello libreria di classi (RCL) prevede Razor lo sviluppo di componenti.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-119">The Razor class library (RCL) template defaults to Razor component development by default.</span></span> <span data-ttu-id="ce0ad-120">L'opzione **pagine e visualizzazioni di supporto** supporta pagine e viste.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-120">The **Support pages and views** option supports pages and views.</span></span>

# <a name="net-core-cli"></a>[<span data-ttu-id="ce0ad-121">Interfaccia della riga di comando di .NET Core</span><span class="sxs-lookup"><span data-stu-id="ce0ad-121">.NET Core CLI</span></span>](#tab/netcore-cli)

<span data-ttu-id="ce0ad-122">Eseguire `dotnet new razorclasslib` dalla riga di comando.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-122">From the command line, run `dotnet new razorclasslib`.</span></span> <span data-ttu-id="ce0ad-123">Ad esempio:</span><span class="sxs-lookup"><span data-stu-id="ce0ad-123">For example:</span></span>

```dotnetcli
dotnet new razorclasslib -o RazorUIClassLib
```

<span data-ttu-id="ce0ad-124">Per impostazione predefinita, il Razor modello libreria di classi (RCL) prevede Razor lo sviluppo di componenti.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-124">The Razor class library (RCL) template defaults to Razor component development by default.</span></span> <span data-ttu-id="ce0ad-125">Passare l' `--support-pages-and-views` opzione ( `dotnet new razorclasslib --support-pages-and-views` ) per fornire il supporto per le pagine e le visualizzazioni.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-125">Pass the `--support-pages-and-views` option (`dotnet new razorclasslib --support-pages-and-views`) to provide support for pages and views.</span></span>

<span data-ttu-id="ce0ad-126">Per altre informazioni, vedere [dotnet new](/dotnet/core/tools/dotnet-new).</span><span class="sxs-lookup"><span data-stu-id="ce0ad-126">For more information, see [dotnet new](/dotnet/core/tools/dotnet-new).</span></span> <span data-ttu-id="ce0ad-127">Per evitare un conflitto di nomi di file con la libreria di visualizzazione generata, verificare che il nome della libreria non finisca per `.Views`.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-127">To avoid a file name collision with the generated view library, ensure the library name doesn't end in `.Views`.</span></span>

---

<span data-ttu-id="ce0ad-128">Aggiungere Razor i file a RCL.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-128">Add Razor files to the RCL.</span></span>

<span data-ttu-id="ce0ad-129">I modelli di ASP.NET Core presuppongono che il contenuto RCL si trovi nella cartella *aree* .</span><span class="sxs-lookup"><span data-stu-id="ce0ad-129">The ASP.NET Core templates assume the RCL content is in the *Areas* folder.</span></span> <span data-ttu-id="ce0ad-130">Vedere [layout di pagine RCL](#rcl-pages-layout) per creare un RCL che espone il contenuto in `~/Pages` anziché `~/Areas/Pages` .</span><span class="sxs-lookup"><span data-stu-id="ce0ad-130">See [RCL Pages layout](#rcl-pages-layout) to create an RCL that exposes content in `~/Pages` rather than `~/Areas/Pages`.</span></span>

## <a name="reference-rcl-content"></a><span data-ttu-id="ce0ad-131">Contenuto RCL di riferimento</span><span class="sxs-lookup"><span data-stu-id="ce0ad-131">Reference RCL content</span></span>

<span data-ttu-id="ce0ad-132">Il riferimento alla libreria di classi Razor può essere eseguito da:</span><span class="sxs-lookup"><span data-stu-id="ce0ad-132">The RCL can be referenced by:</span></span>

* <span data-ttu-id="ce0ad-133">Pacchetto NuGet.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-133">NuGet package.</span></span> <span data-ttu-id="ce0ad-134">Vedere [Creazione di pacchetti NuGet](/nuget/create-packages/creating-a-package), [dotnet add package](/dotnet/core/tools/dotnet-add-package) e [Creare e pubblicare un pacchetto NuGet](/nuget/quickstart/create-and-publish-a-package-using-visual-studio).</span><span class="sxs-lookup"><span data-stu-id="ce0ad-134">See [Creating NuGet packages](/nuget/create-packages/creating-a-package) and [dotnet add package](/dotnet/core/tools/dotnet-add-package) and [Create and publish a NuGet package](/nuget/quickstart/create-and-publish-a-package-using-visual-studio).</span></span>
* <span data-ttu-id="ce0ad-135">*{ProjectName}.csproj*.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-135">*{ProjectName}.csproj*.</span></span> <span data-ttu-id="ce0ad-136">Vedere [dotnet-add reference](/dotnet/core/tools/dotnet-add-reference).</span><span class="sxs-lookup"><span data-stu-id="ce0ad-136">See [dotnet-add reference](/dotnet/core/tools/dotnet-add-reference).</span></span>

## <a name="override-views-partial-views-and-pages"></a><span data-ttu-id="ce0ad-137">Eseguire l'override di visualizzazioni, visualizzazioni parziali e pagine</span><span class="sxs-lookup"><span data-stu-id="ce0ad-137">Override views, partial views, and pages</span></span>

<span data-ttu-id="ce0ad-138">Quando si trova una visualizzazione, una visualizzazione parziale o una Razor pagina nell'app Web e nel RCL, il Razor markup (file con *estensione cshtml* ) nell'app Web ha la precedenza.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-138">When a view, partial view, or Razor Page is found in both the web app and the RCL, the Razor markup (*.cshtml* file) in the web app takes precedence.</span></span> <span data-ttu-id="ce0ad-139">Ad esempio, aggiungere *app Web 1/areas/la funzionalità/Pages/Pagina1. cshtml* a app Web 1 e Pagina1 in app Web 1 avrà la precedenza su PAGINA1 in RCL.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-139">For example, add *WebApp1/Areas/MyFeature/Pages/Page1.cshtml* to WebApp1, and Page1 in the WebApp1 will take precedence over Page1 in the RCL.</span></span>

<span data-ttu-id="ce0ad-140">Nel download di esempio, rinominare *WebApp1/aree/MyFeature2* in *WebApp1/aree/MyFeature* per testare la precedenza.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-140">In the sample download, rename *WebApp1/Areas/MyFeature2* to *WebApp1/Areas/MyFeature* to test precedence.</span></span>

<span data-ttu-id="ce0ad-141">Copiare la visualizzazione parziale *Razor UIClassLib/areas/la funzionalità/Pages/Shared/_Message. cshtml* in *app Web 1/areas/my feature/pages/Shared/_Message. cshtml*.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-141">Copy the *RazorUIClassLib/Areas/MyFeature/Pages/Shared/_Message.cshtml* partial view to *WebApp1/Areas/MyFeature/Pages/Shared/_Message.cshtml*.</span></span> <span data-ttu-id="ce0ad-142">Aggiornare il markup per indicare la nuova posizione.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-142">Update the markup to indicate the new location.</span></span> <span data-ttu-id="ce0ad-143">Compilare ed eseguire l'app per verificare quale versione del parziale sia in uso.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-143">Build and run the app to verify the app's version of the partial is being used.</span></span>

### <a name="rcl-pages-layout"></a><span data-ttu-id="ce0ad-144">Layout di pagine RCL</span><span class="sxs-lookup"><span data-stu-id="ce0ad-144">RCL Pages layout</span></span>

<span data-ttu-id="ce0ad-145">Per fare riferimento al contenuto RCL come se facesse parte della cartella *pagine* dell'app Web, creare il progetto RCL con la struttura di file seguente:</span><span class="sxs-lookup"><span data-stu-id="ce0ad-145">To reference RCL content as though it is part of the web app's *Pages* folder, create the RCL project with the following file structure:</span></span>

* <span data-ttu-id="ce0ad-146">*RazorUIClassLib/pagine*</span><span class="sxs-lookup"><span data-stu-id="ce0ad-146">*RazorUIClassLib/Pages*</span></span>
* <span data-ttu-id="ce0ad-147">*RazorUIClassLib/pagine/condiviso*</span><span class="sxs-lookup"><span data-stu-id="ce0ad-147">*RazorUIClassLib/Pages/Shared*</span></span>

<span data-ttu-id="ce0ad-148">Si supponga che *Razor UIClassLib/Pages/Shared* contenga due file parziali: *_Header. cshtml* e *_Footer. cshtml*.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-148">Suppose *RazorUIClassLib/Pages/Shared* contains two partial files: *_Header.cshtml* and *_Footer.cshtml*.</span></span> <span data-ttu-id="ce0ad-149">`<partial>`È possibile aggiungere i tag al file *_Layout. cshtml* :</span><span class="sxs-lookup"><span data-stu-id="ce0ad-149">The `<partial>` tags could be added to *_Layout.cshtml* file:</span></span>

```cshtml
<body>
  <partial name="_Header">
  @RenderBody()
  <partial name="_Footer">
</body>
```

<span data-ttu-id="ce0ad-150">Aggiungere il file *_ViewStart. cshtml* alla cartella *pagine* del progetto RCL per usare il file *_Layout. cshtml* dall'app Web host:</span><span class="sxs-lookup"><span data-stu-id="ce0ad-150">Add the *_ViewStart.cshtml* file to the RCL project's *Pages* folder to use the *_Layout.cshtml* file from the host web app:</span></span>

```cshtml
@{
    Layout = "_Layout";
}
```

## <a name="create-an-rcl-with-static-assets"></a><span data-ttu-id="ce0ad-151">Creare un RCL con asset statici</span><span class="sxs-lookup"><span data-stu-id="ce0ad-151">Create an RCL with static assets</span></span>

<span data-ttu-id="ce0ad-152">Un RCL può richiedere risorse statiche complementari a cui è possibile fare riferimento tramite il RCL o l'app consumer di RCL.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-152">An RCL may require companion static assets that can be referenced by either the RCL or the consuming app of the RCL.</span></span> <span data-ttu-id="ce0ad-153">ASP.NET Core consente di creare RCL che includono risorse statiche disponibili per un'app di consumo.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-153">ASP.NET Core allows creating RCLs that include static assets that are available to a consuming app.</span></span>

<span data-ttu-id="ce0ad-154">Per includere asset complementari come parte di un RCL, creare una cartella *wwwroot* nella libreria di classi e includere tutti i file necessari in tale cartella.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-154">To include companion assets as part of an RCL, create a *wwwroot* folder in the class library and include any required files in that folder.</span></span>

<span data-ttu-id="ce0ad-155">Quando si esegue la compressione di un RCL, tutti gli asset complementari nella cartella *wwwroot* vengono inclusi automaticamente nel pacchetto.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-155">When packing an RCL, all companion assets in the *wwwroot* folder are automatically included in the package.</span></span>

<span data-ttu-id="ce0ad-156">Usare il `dotnet pack` comando anziché la versione NuGet.exe `nuget pack` .</span><span class="sxs-lookup"><span data-stu-id="ce0ad-156">Use the `dotnet pack` command rather than the NuGet.exe version `nuget pack`.</span></span>

### <a name="exclude-static-assets"></a><span data-ttu-id="ce0ad-157">Escludi asset statici</span><span class="sxs-lookup"><span data-stu-id="ce0ad-157">Exclude static assets</span></span>

<span data-ttu-id="ce0ad-158">Per escludere asset statici, aggiungere il percorso di esclusione desiderato al `$(DefaultItemExcludes)` gruppo di proprietà nel file di progetto.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-158">To exclude static assets, add the desired exclusion path to the `$(DefaultItemExcludes)` property group in the project file.</span></span> <span data-ttu-id="ce0ad-159">Separare le voci con un punto e virgola ( `;` ).</span><span class="sxs-lookup"><span data-stu-id="ce0ad-159">Separate entries with a semicolon (`;`).</span></span>

<span data-ttu-id="ce0ad-160">Nell'esempio seguente, il foglio di stile *lib. CSS* nella cartella *wwwroot* non è considerato un asset statico e non è incluso nel RCL pubblicato:</span><span class="sxs-lookup"><span data-stu-id="ce0ad-160">In the following example, the *lib.css* stylesheet in the *wwwroot* folder isn't considered a static asset and isn't included in the published RCL:</span></span>

```xml
<PropertyGroup>
  <DefaultItemExcludes>$(DefaultItemExcludes);wwwroot\lib.css</DefaultItemExcludes>
</PropertyGroup>
```

### <a name="typescript-integration"></a><span data-ttu-id="ce0ad-161">Integrazione typescript</span><span class="sxs-lookup"><span data-stu-id="ce0ad-161">Typescript integration</span></span>

<span data-ttu-id="ce0ad-162">Per includere i file TypeScript in un RCL:</span><span class="sxs-lookup"><span data-stu-id="ce0ad-162">To include TypeScript files in an RCL:</span></span>

1. <span data-ttu-id="ce0ad-163">Inserire i file TypeScript (*. TS*) all'esterno della cartella *wwwroot*</span><span class="sxs-lookup"><span data-stu-id="ce0ad-163">Place the TypeScript files (*.ts*) outside of the *wwwroot* folder.</span></span> <span data-ttu-id="ce0ad-164">Inserire, ad esempio, i file in una cartella *client* .</span><span class="sxs-lookup"><span data-stu-id="ce0ad-164">For example, place the files in a *Client* folder.</span></span>

1. <span data-ttu-id="ce0ad-165">Configurare l'output di compilazione TypeScript per la cartella *wwwroot* .</span><span class="sxs-lookup"><span data-stu-id="ce0ad-165">Configure the TypeScript build output for the *wwwroot* folder.</span></span> <span data-ttu-id="ce0ad-166">Impostare la `TypescriptOutDir` proprietà all'interno di un oggetto `PropertyGroup` nel file di progetto:</span><span class="sxs-lookup"><span data-stu-id="ce0ad-166">Set the `TypescriptOutDir` property inside of a `PropertyGroup` in the project file:</span></span>

   ```xml
   <TypescriptOutDir>wwwroot</TypescriptOutDir>
   ```

1. <span data-ttu-id="ce0ad-167">Includere la destinazione TypeScript come dipendenza della destinazione aggiungendo `ResolveCurrentProjectStaticWebAssets` la destinazione seguente all'interno di un oggetto `PropertyGroup` nel file di progetto:</span><span class="sxs-lookup"><span data-stu-id="ce0ad-167">Include the TypeScript target as a dependency of the `ResolveCurrentProjectStaticWebAssets` target by adding the following target inside of a `PropertyGroup` in the project file:</span></span>

   ```xml
   <ResolveCurrentProjectStaticWebAssetsInputsDependsOn>
     CompileTypeScript;
     $(ResolveCurrentProjectStaticWebAssetsInputs)
   </ResolveCurrentProjectStaticWebAssetsInputsDependsOn>
   ```

### <a name="consume-content-from-a-referenced-rcl"></a><span data-ttu-id="ce0ad-168">Utilizzare il contenuto da un RCL a cui si fa riferimento</span><span class="sxs-lookup"><span data-stu-id="ce0ad-168">Consume content from a referenced RCL</span></span>

<span data-ttu-id="ce0ad-169">I file inclusi nella cartella *wwwroot* di RCL vengono esposti a RCL o all'app consumer sotto il prefisso `_content/{LIBRARY NAME}/` .</span><span class="sxs-lookup"><span data-stu-id="ce0ad-169">The files included in the *wwwroot* folder of the RCL are exposed to either the RCL or the consuming app under the prefix `_content/{LIBRARY NAME}/`.</span></span> <span data-ttu-id="ce0ad-170">Ad esempio, una libreria denominata *Razor . Class. lib* restituisce un percorso al contenuto statico in `_content/Razor.Class.Lib/` .</span><span class="sxs-lookup"><span data-stu-id="ce0ad-170">For example, a library named *Razor.Class.Lib* results in a path to static content at `_content/Razor.Class.Lib/`.</span></span> <span data-ttu-id="ce0ad-171">Quando si produce un pacchetto NuGet e il nome dell'assembly non è uguale all'ID del pacchetto, usare l'ID del pacchetto per `{LIBRARY NAME}` .</span><span class="sxs-lookup"><span data-stu-id="ce0ad-171">When producing a NuGet package and the assembly name isn't the same as the package ID, use the package ID for `{LIBRARY NAME}`.</span></span>

<span data-ttu-id="ce0ad-172">L'app consumer fa riferimento a risorse statiche fornite dalla libreria con `<script>` ,, `<style>` `<img>` e altri tag HTML.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-172">The consuming app references static assets provided by the library with `<script>`, `<style>`, `<img>`, and other HTML tags.</span></span> <span data-ttu-id="ce0ad-173">Per l'app consumer deve essere abilitato il [supporto file statico](xref:fundamentals/static-files) in `Startup.Configure` :</span><span class="sxs-lookup"><span data-stu-id="ce0ad-173">The consuming app must have [static file support](xref:fundamentals/static-files) enabled in `Startup.Configure`:</span></span>

```csharp
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    ...

    app.UseStaticFiles();

    ...
}
```

<span data-ttu-id="ce0ad-174">Quando si esegue l'app consumer dall'output di compilazione ( `dotnet run` ), le risorse Web statiche sono abilitate per impostazione predefinita nell'ambiente di sviluppo.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-174">When running the consuming app from build output (`dotnet run`), static web assets are enabled by default in the Development environment.</span></span> <span data-ttu-id="ce0ad-175">Per supportare asset in altri ambienti durante l'esecuzione dall'output di compilazione, chiamare `UseStaticWebAssets` sul generatore host in *Program.cs*:</span><span class="sxs-lookup"><span data-stu-id="ce0ad-175">To support assets in other environments when running from build output, call `UseStaticWebAssets` on the host builder in *Program.cs*:</span></span>

```csharp
using Microsoft.AspNetCore.Hosting;
using Microsoft.Extensions.Hosting;

public class Program
{
    public static void Main(string[] args)
    {
        CreateHostBuilder(args).Build().Run();
    }

    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .ConfigureWebHostDefaults(webBuilder =>
            {
                webBuilder.UseStaticWebAssets();
                webBuilder.UseStartup<Startup>();
            });
}
```

<span data-ttu-id="ce0ad-176">`UseStaticWebAssets`Quando si esegue un'app dall'output pubblicato (), la chiamata non è obbligatoria `dotnet publish` .</span><span class="sxs-lookup"><span data-stu-id="ce0ad-176">Calling `UseStaticWebAssets` isn't required when running an app from published output (`dotnet publish`).</span></span>

### <a name="multi-project-development-flow"></a><span data-ttu-id="ce0ad-177">Flusso di sviluppo per più progetti</span><span class="sxs-lookup"><span data-stu-id="ce0ad-177">Multi-project development flow</span></span>

<span data-ttu-id="ce0ad-178">Quando viene eseguita l'app consumer:</span><span class="sxs-lookup"><span data-stu-id="ce0ad-178">When the consuming app runs:</span></span>

* <span data-ttu-id="ce0ad-179">Gli asset nel RCL rimanere nelle cartelle originali.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-179">The assets in the RCL stay in their original folders.</span></span> <span data-ttu-id="ce0ad-180">Gli asset non vengono spostati nell'app consumer.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-180">The assets aren't moved to the consuming app.</span></span>
* <span data-ttu-id="ce0ad-181">Tutte le modifiche apportate all'interno della cartella *wwwroot* di RCL vengono riflesse nell'app consumer dopo che il RCL viene ricompilato e senza ricompilare l'app consumer.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-181">Any change within the RCL's *wwwroot* folder is reflected in the consuming app after the RCL is rebuilt and without rebuilding the consuming app.</span></span>

<span data-ttu-id="ce0ad-182">Quando viene compilato il RCL, viene generato un manifesto che descrive i percorsi di asset Web statici.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-182">When the RCL is built, a manifest is produced that describes the static web asset locations.</span></span> <span data-ttu-id="ce0ad-183">L'app consumer legge il manifesto in fase di esecuzione per usare gli asset dei progetti e dei pacchetti a cui si fa riferimento.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-183">The consuming app reads the manifest at runtime to consume the assets from referenced projects and packages.</span></span> <span data-ttu-id="ce0ad-184">Quando si aggiunge un nuovo asset a un RCL, è necessario ricompilare il RCL per aggiornare il manifesto prima che un'app che utilizza possa accedere al nuovo asset.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-184">When a new asset is added to an RCL, the RCL must be rebuilt to update its manifest before a consuming app can access the new asset.</span></span>

### <a name="publish"></a><span data-ttu-id="ce0ad-185">Pubblica</span><span class="sxs-lookup"><span data-stu-id="ce0ad-185">Publish</span></span>

<span data-ttu-id="ce0ad-186">Quando viene pubblicata l'app, gli asset complementari di tutti i progetti e i pacchetti a cui viene fatto riferimento vengono copiati nella cartella *wwwroot* dell'app pubblicata in `_content/{LIBRARY NAME}/` .</span><span class="sxs-lookup"><span data-stu-id="ce0ad-186">When the app is published, the companion assets from all referenced projects and packages are copied into the *wwwroot* folder of the published app under `_content/{LIBRARY NAME}/`.</span></span>

::: moniker-end

::: moniker range="< aspnetcore-3.0"

<span data-ttu-id="ce0ad-187">Razorle visualizzazioni, le pagine, i controller, i modelli di pagina, i [ Razor componenti](xref:blazor/components/class-libraries), i [componenti di visualizzazione](xref:mvc/views/view-components)e i modelli di dati possono essere incorporati in una Razor libreria di classi (RCL).</span><span class="sxs-lookup"><span data-stu-id="ce0ad-187">Razor views, pages, controllers, page models, [Razor components](xref:blazor/components/class-libraries), [View components](xref:mvc/views/view-components), and data models can be built into a Razor class library (RCL).</span></span> <span data-ttu-id="ce0ad-188">La libreria di classi Razor può essere inclusa nel pacchetto e usata nuovamente.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-188">The RCL can be packaged and reused.</span></span> <span data-ttu-id="ce0ad-189">Le applicazioni possono includere la libreria di classi Razor ed eseguire l'override delle visualizzazioni e pagine in essa contenute.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-189">Applications can include the RCL and override the views and pages it contains.</span></span> <span data-ttu-id="ce0ad-190">Quando si trova una visualizzazione, una visualizzazione parziale o una Razor pagina nell'app Web e nel RCL, il Razor markup (file con *estensione cshtml* ) nell'app Web ha la precedenza.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-190">When a view, partial view, or Razor Page is found in both the web app and the RCL, the Razor markup (*.cshtml* file) in the web app takes precedence.</span></span>

<span data-ttu-id="ce0ad-191">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/razor-pages/ui-class/samples) ([procedura per il download](xref:index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="ce0ad-191">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/razor-pages/ui-class/samples) ([how to download](xref:index#how-to-download-a-sample))</span></span>

## <a name="create-a-class-library-containing-razor-ui"></a><span data-ttu-id="ce0ad-192">Creare una libreria di classi contenente l' Razor interfaccia utente</span><span class="sxs-lookup"><span data-stu-id="ce0ad-192">Create a class library containing Razor UI</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="ce0ad-193">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="ce0ad-193">Visual Studio</span></span>](#tab/visual-studio)

* <span data-ttu-id="ce0ad-194">Dal menu **File** di Visual Studio selezionare **Nuovo** > **Progetto**.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-194">From the Visual Studio **File** menu, select **New** > **Project**.</span></span>
* <span data-ttu-id="ce0ad-195">Selezionare **Applicazione Web ASP.NET Core**.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-195">Select **ASP.NET Core Web Application**.</span></span>
* <span data-ttu-id="ce0ad-196">Assegnare un nome alla libreria (ad esempio, " Razor CLASSLIB") > **OK**.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-196">Name the library (for example, "RazorClassLib") > **OK**.</span></span> <span data-ttu-id="ce0ad-197">Per evitare un conflitto di nomi di file con la libreria di visualizzazione generata, verificare che il nome della libreria non finisca per `.Views`.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-197">To avoid a file name collision with the generated view library, ensure the library name doesn't end in `.Views`.</span></span>
* <span data-ttu-id="ce0ad-198">Verificare che sia selezionato **ASP.NET Core 2.1** o versioni successive.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-198">Verify **ASP.NET Core 2.1** or later is selected.</span></span>
* <span data-ttu-id="ce0ad-199">Selezionare **Razor libreria di classi** > **OK**.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-199">Select **Razor Class Library** > **OK**.</span></span>

<span data-ttu-id="ce0ad-200">Un RCL ha il seguente file di progetto:</span><span class="sxs-lookup"><span data-stu-id="ce0ad-200">An RCL has the following project file:</span></span>

[!code-xml[](ui-class/samples/cli/RazorUIClassLib/RazorUIClassLib.csproj)]

# <a name="net-core-cli"></a>[<span data-ttu-id="ce0ad-201">Interfaccia della riga di comando di .NET Core</span><span class="sxs-lookup"><span data-stu-id="ce0ad-201">.NET Core CLI</span></span>](#tab/netcore-cli)

<span data-ttu-id="ce0ad-202">Eseguire `dotnet new razorclasslib` dalla riga di comando.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-202">From the command line, run `dotnet new razorclasslib`.</span></span> <span data-ttu-id="ce0ad-203">Ad esempio:</span><span class="sxs-lookup"><span data-stu-id="ce0ad-203">For example:</span></span>

```dotnetcli
dotnet new razorclasslib -o RazorUIClassLib
```

<span data-ttu-id="ce0ad-204">Per altre informazioni, vedere [dotnet new](/dotnet/core/tools/dotnet-new).</span><span class="sxs-lookup"><span data-stu-id="ce0ad-204">For more information, see [dotnet new](/dotnet/core/tools/dotnet-new).</span></span> <span data-ttu-id="ce0ad-205">Per evitare un conflitto di nomi di file con la libreria di visualizzazione generata, verificare che il nome della libreria non finisca per `.Views`.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-205">To avoid a file name collision with the generated view library, ensure the library name doesn't end in `.Views`.</span></span>

---

<span data-ttu-id="ce0ad-206">Aggiungere Razor i file a RCL.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-206">Add Razor files to the RCL.</span></span>

<span data-ttu-id="ce0ad-207">I modelli di ASP.NET Core presuppongono che il contenuto RCL si trovi nella cartella *aree* .</span><span class="sxs-lookup"><span data-stu-id="ce0ad-207">The ASP.NET Core templates assume the RCL content is in the *Areas* folder.</span></span> <span data-ttu-id="ce0ad-208">Vedere [layout di pagine RCL](#rcl-pages-layout) per creare un RCL che espone il contenuto in `~/Pages` anziché `~/Areas/Pages` .</span><span class="sxs-lookup"><span data-stu-id="ce0ad-208">See [RCL Pages layout](#rcl-pages-layout) to create an RCL that exposes content in `~/Pages` rather than `~/Areas/Pages`.</span></span>

## <a name="reference-rcl-content"></a><span data-ttu-id="ce0ad-209">Contenuto RCL di riferimento</span><span class="sxs-lookup"><span data-stu-id="ce0ad-209">Reference RCL content</span></span>

<span data-ttu-id="ce0ad-210">Il riferimento alla libreria di classi Razor può essere eseguito da:</span><span class="sxs-lookup"><span data-stu-id="ce0ad-210">The RCL can be referenced by:</span></span>

* <span data-ttu-id="ce0ad-211">Pacchetto NuGet.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-211">NuGet package.</span></span> <span data-ttu-id="ce0ad-212">Vedere [Creazione di pacchetti NuGet](/nuget/create-packages/creating-a-package), [dotnet add package](/dotnet/core/tools/dotnet-add-package) e [Creare e pubblicare un pacchetto NuGet](/nuget/quickstart/create-and-publish-a-package-using-visual-studio).</span><span class="sxs-lookup"><span data-stu-id="ce0ad-212">See [Creating NuGet packages](/nuget/create-packages/creating-a-package) and [dotnet add package](/dotnet/core/tools/dotnet-add-package) and [Create and publish a NuGet package](/nuget/quickstart/create-and-publish-a-package-using-visual-studio).</span></span>
* <span data-ttu-id="ce0ad-213">*{ProjectName}.csproj*.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-213">*{ProjectName}.csproj*.</span></span> <span data-ttu-id="ce0ad-214">Vedere [dotnet-add reference](/dotnet/core/tools/dotnet-add-reference).</span><span class="sxs-lookup"><span data-stu-id="ce0ad-214">See [dotnet-add reference](/dotnet/core/tools/dotnet-add-reference).</span></span>

## <a name="walkthrough-create-an-rcl-project-and-use-from-a-razor-pages-project"></a><span data-ttu-id="ce0ad-215">Procedura dettagliata: creare un progetto RCL e usarlo da un Razor progetto Pages</span><span class="sxs-lookup"><span data-stu-id="ce0ad-215">Walkthrough: Create an RCL project and use from a Razor Pages project</span></span>

<span data-ttu-id="ce0ad-216">È possibile scaricare il [progetto completo](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/razor-pages/ui-class/samples) e testarlo anziché crearlo.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-216">You can download the [complete project](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/razor-pages/ui-class/samples) and test it rather than creating it.</span></span> <span data-ttu-id="ce0ad-217">Il download di esempio contiene codice aggiuntivo e collegamenti che rendono più semplice testare il progetto.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-217">The sample download contains additional code and links that make the project easy to test.</span></span> <span data-ttu-id="ce0ad-218">È possibile lasciare commenti e suggerimenti in [questa discussione su GitHub](https://github.com/dotnet/AspNetCore.Docs/issues/6098) con i commenti sui download di esempio rispetto alle istruzioni dettagliate.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-218">You can leave feedback in [this GitHub issue](https://github.com/dotnet/AspNetCore.Docs/issues/6098) with your comments on download samples versus step-by-step instructions.</span></span>

### <a name="test-the-download-app"></a><span data-ttu-id="ce0ad-219">Testare l'app di download</span><span class="sxs-lookup"><span data-stu-id="ce0ad-219">Test the download app</span></span>

<span data-ttu-id="ce0ad-220">Se non è stata scaricata l'app completa e si vuole invece creare il progetto della procedura dettagliata, passare alla [prossima sezione](#create-an-rcl).</span><span class="sxs-lookup"><span data-stu-id="ce0ad-220">If you haven't downloaded the completed app and would rather create the walkthrough project, skip to the [next section](#create-an-rcl).</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="ce0ad-221">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="ce0ad-221">Visual Studio</span></span>](#tab/visual-studio)

<span data-ttu-id="ce0ad-222">Aprire il file con estensione *sln* in Visual Studio.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-222">Open the *.sln* file in Visual Studio.</span></span> <span data-ttu-id="ce0ad-223">Eseguire l'app.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-223">Run the app.</span></span>

# <a name="net-core-cli"></a>[<span data-ttu-id="ce0ad-224">Interfaccia della riga di comando di .NET Core</span><span class="sxs-lookup"><span data-stu-id="ce0ad-224">.NET Core CLI</span></span>](#tab/netcore-cli)

<span data-ttu-id="ce0ad-225">Al prompt dei comandi nella directory *cli*, compilare la libreria di classi Razor e l'app Web.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-225">From a command prompt in the *cli* directory, build the RCL and web app.</span></span>

```dotnetcli
dotnet build
```

<span data-ttu-id="ce0ad-226">Passare alla directory *WebApp1* ed eseguire l'app:</span><span class="sxs-lookup"><span data-stu-id="ce0ad-226">Move to the *WebApp1* directory and run the app:</span></span>

```dotnetcli
dotnet run
```

---

<span data-ttu-id="ce0ad-227">Seguire le istruzioni indicate in [Testare WebApp1](#test-webapp1)</span><span class="sxs-lookup"><span data-stu-id="ce0ad-227">Follow the instructions in [Test WebApp1](#test-webapp1)</span></span>

## <a name="create-an-rcl"></a><span data-ttu-id="ce0ad-228">Creare un RCL</span><span class="sxs-lookup"><span data-stu-id="ce0ad-228">Create an RCL</span></span>

<span data-ttu-id="ce0ad-229">In questa sezione viene creato un RCL.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-229">In this section, an RCL is created.</span></span> <span data-ttu-id="ce0ad-230">Razor i file vengono aggiunti a RCL.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-230">Razor files are added to the RCL.</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="ce0ad-231">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="ce0ad-231">Visual Studio</span></span>](#tab/visual-studio)

<span data-ttu-id="ce0ad-232">Creare il progetto di libreria di classi Razor:</span><span class="sxs-lookup"><span data-stu-id="ce0ad-232">Create the RCL project:</span></span>

* <span data-ttu-id="ce0ad-233">Dal menu **File** di Visual Studio selezionare **Nuovo** > **Progetto**.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-233">From the Visual Studio **File** menu, select **New** > **Project**.</span></span>
* <span data-ttu-id="ce0ad-234">Selezionare **Applicazione Web ASP.NET Core**.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-234">Select **ASP.NET Core Web Application**.</span></span>
* <span data-ttu-id="ce0ad-235">Denominare l'app **Razor UIClassLib** > **OK**.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-235">Name the app **RazorUIClassLib** > **OK**.</span></span>
* <span data-ttu-id="ce0ad-236">Verificare che sia selezionato **ASP.NET Core 2.1** o versioni successive.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-236">Verify **ASP.NET Core 2.1** or later is selected.</span></span>
* <span data-ttu-id="ce0ad-237">Selezionare **Razor libreria di classi** > **OK**.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-237">Select **Razor Class Library** > **OK**.</span></span>
* <span data-ttu-id="ce0ad-238">Aggiungere un Razor file di visualizzazione parziale denominato *Razor UIClassLib/areas/la funzionalità/Pages/Shared/_Message. cshtml*.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-238">Add a Razor partial view file named *RazorUIClassLib/Areas/MyFeature/Pages/Shared/_Message.cshtml*.</span></span>

# <a name="net-core-cli"></a>[<span data-ttu-id="ce0ad-239">Interfaccia della riga di comando di .NET Core</span><span class="sxs-lookup"><span data-stu-id="ce0ad-239">.NET Core CLI</span></span>](#tab/netcore-cli)

<span data-ttu-id="ce0ad-240">Eseguire il comando seguente dalla riga di comando:</span><span class="sxs-lookup"><span data-stu-id="ce0ad-240">From the command line, run the following:</span></span>

```dotnetcli
dotnet new razorclasslib -o RazorUIClassLib
dotnet new page -n _Message -np -o RazorUIClassLib/Areas/MyFeature/Pages/Shared
dotnet new viewstart -o RazorUIClassLib/Areas/MyFeature/Pages
```

<span data-ttu-id="ce0ad-241">I comandi precedenti:</span><span class="sxs-lookup"><span data-stu-id="ce0ad-241">The preceding commands:</span></span>

* <span data-ttu-id="ce0ad-242">Crea l'oggetto `RazorUIClassLib` RCL.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-242">Creates the `RazorUIClassLib` RCL.</span></span>
* <span data-ttu-id="ce0ad-243">Crea una Razor pagina di _Message e la aggiunge a RCL.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-243">Creates a Razor _Message page, and adds it to the RCL.</span></span> <span data-ttu-id="ce0ad-244">Il parametro `-np` crea la pagina senza un `PageModel`.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-244">The `-np` parameter creates the page without a `PageModel`.</span></span>
* <span data-ttu-id="ce0ad-245">Crea un file [_ViewStart. cshtml](xref:mvc/views/layout#running-code-before-each-view) e lo aggiunge al RCL.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-245">Creates a [_ViewStart.cshtml](xref:mvc/views/layout#running-code-before-each-view) file and adds it to the RCL.</span></span>

<span data-ttu-id="ce0ad-246">Il file *_ViewStart. cshtml* è necessario per usare il layout del Razor progetto Pages, che viene aggiunto nella sezione successiva.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-246">The *_ViewStart.cshtml* file is required to use the layout of the Razor Pages project (which is added in the next section).</span></span>

---

### <a name="add-razor-files-and-folders-to-the-project"></a><span data-ttu-id="ce0ad-247">Aggiungere Razor file e cartelle al progetto</span><span class="sxs-lookup"><span data-stu-id="ce0ad-247">Add Razor files and folders to the project</span></span>

* <span data-ttu-id="ce0ad-248">Sostituire il markup in *Razor UIClassLib/areas/la funzionalità/Pages/Shared/_Message. cshtml* con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="ce0ad-248">Replace the markup in *RazorUIClassLib/Areas/MyFeature/Pages/Shared/_Message.cshtml* with the following code:</span></span>

  [!code-cshtml[](ui-class/samples/cli/RazorUIClassLib/Areas/MyFeature/Pages/Shared/_Message.cshtml)]

* <span data-ttu-id="ce0ad-249">Sostituire il markup in *Razor UIClassLib/areas/la funzionalità/Pages/Pagina1. cshtml* con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="ce0ad-249">Replace the markup in *RazorUIClassLib/Areas/MyFeature/Pages/Page1.cshtml* with the following code:</span></span>

  [!code-cshtml[](ui-class/samples/cli/RazorUIClassLib/Areas/MyFeature/Pages/Page1.cshtml)]

  <span data-ttu-id="ce0ad-250">`@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers` è necessario per usare la visualizzazione parziale (`<partial name="_Message" />`).</span><span class="sxs-lookup"><span data-stu-id="ce0ad-250">`@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers` is required to use the partial view (`<partial name="_Message" />`).</span></span> <span data-ttu-id="ce0ad-251">Anziché includere la direttiva `@addTagHelper`, è possibile aggiungere un file *_ViewImports.cshtml*.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-251">Rather than including the `@addTagHelper` directive, you can add a *_ViewImports.cshtml* file.</span></span> <span data-ttu-id="ce0ad-252">Ad esempio:</span><span class="sxs-lookup"><span data-stu-id="ce0ad-252">For example:</span></span>

  ```dotnetcli
  dotnet new viewimports -o RazorUIClassLib/Areas/MyFeature/Pages
  ```

  <span data-ttu-id="ce0ad-253">Per altre informazioni su *_ViewImports. cshtml*, vedere [importazione di direttive condivise](xref:mvc/views/layout#importing-shared-directives)</span><span class="sxs-lookup"><span data-stu-id="ce0ad-253">For more information on *_ViewImports.cshtml*, see [Importing Shared Directives](xref:mvc/views/layout#importing-shared-directives)</span></span>

* <span data-ttu-id="ce0ad-254">Compilare la libreria di classi per verificare che non siano presenti errori del compilatore:</span><span class="sxs-lookup"><span data-stu-id="ce0ad-254">Build the class library to verify there are no compiler errors:</span></span>

  ```dotnetcli
  dotnet build RazorUIClassLib
  ```

<span data-ttu-id="ce0ad-255">L'output di compilazione contiene *RazorUIClassLib.dll* e *RazorUIClassLib.Views.dll*.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-255">The build output contains *RazorUIClassLib.dll* and *RazorUIClassLib.Views.dll*.</span></span> <span data-ttu-id="ce0ad-256">*RazorUIClassLib.Views.dll* contiene il Razor contenuto compilato.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-256">*RazorUIClassLib.Views.dll* contains the compiled Razor content.</span></span>

### <a name="use-the-razor-ui-library-from-a-razor-pages-project"></a><span data-ttu-id="ce0ad-257">Usare la Razor libreria dell'interfaccia utente da un Razor progetto di pagine</span><span class="sxs-lookup"><span data-stu-id="ce0ad-257">Use the Razor UI library from a Razor Pages project</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="ce0ad-258">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="ce0ad-258">Visual Studio</span></span>](#tab/visual-studio)

<span data-ttu-id="ce0ad-259">Creare l' Razor app Web Pages:</span><span class="sxs-lookup"><span data-stu-id="ce0ad-259">Create the Razor Pages web app:</span></span>

* <span data-ttu-id="ce0ad-260">In **Esplora soluzioni** fare clic con il pulsante destro del mouse sulla soluzione > **Aggiungi** >  **Nuovo progetto**.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-260">From **Solution Explorer**, right-click the solution > **Add** >  **New Project**.</span></span>
* <span data-ttu-id="ce0ad-261">Selezionare **Applicazione Web ASP.NET Core**.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-261">Select **ASP.NET Core Web Application**.</span></span>
* <span data-ttu-id="ce0ad-262">Denominare l'app **WebApp1**.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-262">Name the app **WebApp1**.</span></span>
* <span data-ttu-id="ce0ad-263">Verificare che sia selezionato **ASP.NET Core 2.1** o versioni successive.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-263">Verify **ASP.NET Core 2.1** or later is selected.</span></span>
* <span data-ttu-id="ce0ad-264">Selezionare **Applicazione Web** > **OK**.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-264">Select **Web Application** > **OK**.</span></span>

* <span data-ttu-id="ce0ad-265">In **Esplora soluzioni** fare clic con il pulsante destro del mouse su **WebApp1** e selezionare **Imposta come progetto di avvio**.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-265">From **Solution Explorer**, right-click on **WebApp1** and select **Set as StartUp Project**.</span></span>
* <span data-ttu-id="ce0ad-266">In **Esplora soluzioni** fare clic con il pulsante destro del mouse su **WebApp1** e selezionare **Dipendenze di compilazione** > **Dipendenze progetto**.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-266">From **Solution Explorer**, right-click on **WebApp1** and select **Build Dependencies** > **Project Dependencies**.</span></span>
* <span data-ttu-id="ce0ad-267">Controllare **Razor UIClassLib** come dipendenza di **app Web 1**.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-267">Check **RazorUIClassLib** as a dependency of **WebApp1**.</span></span>
* <span data-ttu-id="ce0ad-268">In **Esplora soluzioni** fare clic con il pulsante destro del mouse su **WebApp1** e scegliere **Aggiungi** > **Riferimento**.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-268">From **Solution Explorer**, right-click on **WebApp1** and select **Add** > **Reference**.</span></span>
* <span data-ttu-id="ce0ad-269">Nella finestra di dialogo **Gestione riferimenti** selezionare **Razor UIClassLib** > **OK**.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-269">In the **Reference Manager** dialog, check **RazorUIClassLib** > **OK**.</span></span>

<span data-ttu-id="ce0ad-270">Eseguire l'app.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-270">Run the app.</span></span>

# <a name="net-core-cli"></a>[<span data-ttu-id="ce0ad-271">Interfaccia della riga di comando di .NET Core</span><span class="sxs-lookup"><span data-stu-id="ce0ad-271">.NET Core CLI</span></span>](#tab/netcore-cli)

<span data-ttu-id="ce0ad-272">Creare un' Razor app Web di pagine e un file di soluzione contenente l' Razor app pagine e il RCL:</span><span class="sxs-lookup"><span data-stu-id="ce0ad-272">Create a Razor Pages web app and a solution file containing the Razor Pages app and the RCL:</span></span>

```dotnetcli
dotnet new webapp -o WebApp1
dotnet new sln
dotnet sln add WebApp1
dotnet sln add RazorUIClassLib
dotnet add WebApp1 reference RazorUIClassLib
```

<span data-ttu-id="ce0ad-273">Compilare ed eseguire l'app Web:</span><span class="sxs-lookup"><span data-stu-id="ce0ad-273">Build and run the web app:</span></span>

```dotnetcli
cd WebApp1
dotnet run
```

---

### <a name="test-webapp1"></a><span data-ttu-id="ce0ad-274">Testare WebApp1</span><span class="sxs-lookup"><span data-stu-id="ce0ad-274">Test WebApp1</span></span>

<span data-ttu-id="ce0ad-275">Passare a `/MyFeature/Page1` per verificare che la Razor libreria di classi dell'interfaccia utente sia in uso.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-275">Browse to `/MyFeature/Page1` to verify that the Razor UI class library is in use.</span></span>

## <a name="override-views-partial-views-and-pages"></a><span data-ttu-id="ce0ad-276">Eseguire l'override di visualizzazioni, visualizzazioni parziali e pagine</span><span class="sxs-lookup"><span data-stu-id="ce0ad-276">Override views, partial views, and pages</span></span>

<span data-ttu-id="ce0ad-277">Quando si trova una visualizzazione, una visualizzazione parziale o una Razor pagina nell'app Web e nel RCL, il Razor markup (file con *estensione cshtml* ) nell'app Web ha la precedenza.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-277">When a view, partial view, or Razor Page is found in both the web app and the RCL, the Razor markup (*.cshtml* file) in the web app takes precedence.</span></span> <span data-ttu-id="ce0ad-278">Ad esempio, aggiungere *app Web 1/areas/la funzionalità/Pages/Pagina1. cshtml* a app Web 1 e Pagina1 in app Web 1 avrà la precedenza su PAGINA1 in RCL.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-278">For example, add *WebApp1/Areas/MyFeature/Pages/Page1.cshtml* to WebApp1, and Page1 in the WebApp1 will take precedence over Page1 in the RCL.</span></span>

<span data-ttu-id="ce0ad-279">Nel download di esempio, rinominare *WebApp1/aree/MyFeature2* in *WebApp1/aree/MyFeature* per testare la precedenza.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-279">In the sample download, rename *WebApp1/Areas/MyFeature2* to *WebApp1/Areas/MyFeature* to test precedence.</span></span>

<span data-ttu-id="ce0ad-280">Copiare la visualizzazione parziale *Razor UIClassLib/areas/la funzionalità/Pages/Shared/_Message. cshtml* in *app Web 1/areas/my feature/pages/Shared/_Message. cshtml*.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-280">Copy the *RazorUIClassLib/Areas/MyFeature/Pages/Shared/_Message.cshtml* partial view to *WebApp1/Areas/MyFeature/Pages/Shared/_Message.cshtml*.</span></span> <span data-ttu-id="ce0ad-281">Aggiornare il markup per indicare la nuova posizione.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-281">Update the markup to indicate the new location.</span></span> <span data-ttu-id="ce0ad-282">Compilare ed eseguire l'app per verificare quale versione del parziale sia in uso.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-282">Build and run the app to verify the app's version of the partial is being used.</span></span>

### <a name="rcl-pages-layout"></a><span data-ttu-id="ce0ad-283">Layout di pagine RCL</span><span class="sxs-lookup"><span data-stu-id="ce0ad-283">RCL Pages layout</span></span>

<span data-ttu-id="ce0ad-284">Per fare riferimento al contenuto RCL come se facesse parte della cartella *pagine* dell'app Web, creare il progetto RCL con la struttura di file seguente:</span><span class="sxs-lookup"><span data-stu-id="ce0ad-284">To reference RCL content as though it is part of the web app's *Pages* folder, create the RCL project with the following file structure:</span></span>

* <span data-ttu-id="ce0ad-285">*RazorUIClassLib/pagine*</span><span class="sxs-lookup"><span data-stu-id="ce0ad-285">*RazorUIClassLib/Pages*</span></span>
* <span data-ttu-id="ce0ad-286">*RazorUIClassLib/pagine/condiviso*</span><span class="sxs-lookup"><span data-stu-id="ce0ad-286">*RazorUIClassLib/Pages/Shared*</span></span>

<span data-ttu-id="ce0ad-287">Si supponga che *Razor UIClassLib/Pages/Shared* contenga due file parziali: *_Header. cshtml* e *_Footer. cshtml*.</span><span class="sxs-lookup"><span data-stu-id="ce0ad-287">Suppose *RazorUIClassLib/Pages/Shared* contains two partial files: *_Header.cshtml* and *_Footer.cshtml*.</span></span> <span data-ttu-id="ce0ad-288">`<partial>`È possibile aggiungere i tag al file *_Layout. cshtml* :</span><span class="sxs-lookup"><span data-stu-id="ce0ad-288">The `<partial>` tags could be added to *_Layout.cshtml* file:</span></span>

```cshtml
<body>
  <partial name="_Header">
  @RenderBody()
  <partial name="_Footer">
</body>
```

::: moniker-end

## <a name="additional-resources"></a><span data-ttu-id="ce0ad-289">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="ce0ad-289">Additional resources</span></span>

::: moniker range=">= aspnetcore-5.0"

* <xref:blazor/components/class-libraries>
* <xref:blazor/components/css-isolation#razor-class-library-rcl-support>

::: moniker-end

::: moniker range="< aspnetcore-5.0"

* <xref:blazor/components/class-libraries>

::: moniker-end
