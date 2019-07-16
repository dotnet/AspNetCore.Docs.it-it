---
title: Comando dotnet aspnet-codegenerator
author: rick-anderson
description: Il comando dotnet aspnet-codegenerator esegue lo scaffolding dei progetti ASP.NET Core.
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.date: 07/04/2019
uid: fundamentals/tools/dotnet-aspnet-codegenerator
ms.openlocfilehash: c96362f320efd84c35dc07294a2968a2c687ee94
ms.sourcegitcommit: b9e914ef274b5ec359582f299724af6234dce135
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 07/05/2019
ms.locfileid: "67596135"
---
# <a name="dotnet-aspnet-codegenerator"></a><span data-ttu-id="d16f9-103">dotnet aspnet-codegenerator</span><span class="sxs-lookup"><span data-stu-id="d16f9-103">dotnet aspnet-codegenerator</span></span>

<span data-ttu-id="d16f9-104">Di [Rick Anderson](https://twitter.com/RickAndMSFT)</span><span class="sxs-lookup"><span data-stu-id="d16f9-104">By [Rick Anderson](https://twitter.com/RickAndMSFT)</span></span>

<span data-ttu-id="d16f9-105">`dotnet aspnet-codegenerator` - Esegue il motore di scaffolding di ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="d16f9-105">`dotnet aspnet-codegenerator` - Runs the ASP.NET Core scaffolding engine.</span></span> <span data-ttu-id="d16f9-106">`dotnet aspnet-codegenerator` è necessario solo per eseguire lo scaffolding dalla riga di comando. Non è necessario per usare lo scaffolding con Visual Studio.</span><span class="sxs-lookup"><span data-stu-id="d16f9-106">`dotnet aspnet-codegenerator` is only required to scaffold from the command line, it's not need to use scaffolding with Visual Studio.</span></span>

<span data-ttu-id="d16f9-107">Questo articolo si applica a [.NET Core 2.1 SDK](https://dotnet.microsoft.com/download/dotnet-core/2.1) e versioni successive.</span><span class="sxs-lookup"><span data-stu-id="d16f9-107">This article applies to [.NET Core 2.1 SDK](https://dotnet.microsoft.com/download/dotnet-core/2.1) and later.</span></span>

## <a name="installing-aspnet-codegenerator"></a><span data-ttu-id="d16f9-108">Installazione di aspnet-codegenerator</span><span class="sxs-lookup"><span data-stu-id="d16f9-108">Installing aspnet-codegenerator</span></span>

<span data-ttu-id="d16f9-109">`dotnet-aspnet-codegenerator` è uno [strumento global](/dotnet/core/tools/global-tools) che deve essere installato.</span><span class="sxs-lookup"><span data-stu-id="d16f9-109">`dotnet-aspnet-codegenerator` is a [global tool](/dotnet/core/tools/global-tools) that must be installed.</span></span> <span data-ttu-id="d16f9-110">Il comando seguente installa l'ultima versione stabile dello strumento `dotnet-aspnet-codegenerator`:</span><span class="sxs-lookup"><span data-stu-id="d16f9-110">The following command installs the latest stable version of the `dotnet-aspnet-codegenerator` tool:</span></span>

```console
dotnet tool install -g dotnet-aspnet-codegenerator
```

<span data-ttu-id="d16f9-111">Il comando seguente aggiorna `dotnet-aspnet-codegenerator` alla versione stabile più recente disponibile dai .NET Core SDK installati:</span><span class="sxs-lookup"><span data-stu-id="d16f9-111">The following command updates `dotnet-aspnet-codegenerator` to the latest stable version available from the installed .NET Core SDKs:</span></span>

```console
dotnet tool update -g dotnet-aspnet-codegenerator
```

## <a name="synopsis"></a><span data-ttu-id="d16f9-112">Riepilogo</span><span class="sxs-lookup"><span data-stu-id="d16f9-112">Synopsis</span></span>

```
dotnet aspnet-codegenerator [arguments] [-p|--project] [-n|--nuget-package-dir] [-c|--configuration] [-tfm|--target-framework] [-b|--build-base-path] [--no-build] 
dotnet aspnet-codegenerator [-h|--help]
```

## <a name="description"></a><span data-ttu-id="d16f9-113">DESCRIZIONE</span><span class="sxs-lookup"><span data-stu-id="d16f9-113">Description</span></span>

<span data-ttu-id="d16f9-114">Il comando globale `dotnet aspnet-codegenerator` esegue il generatore di codice e il motore di scaffolding di ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="d16f9-114">The `dotnet aspnet-codegenerator` global command runs the ASP.NET Core code generator and scaffolding engine.</span></span>

## <a name="arguments"></a><span data-ttu-id="d16f9-115">Argomenti</span><span class="sxs-lookup"><span data-stu-id="d16f9-115">Arguments</span></span>

`generator`

<span data-ttu-id="d16f9-116">Il generatore di codice da eseguire.</span><span class="sxs-lookup"><span data-stu-id="d16f9-116">The code generator to run.</span></span> <span data-ttu-id="d16f9-117">Sono disponibili i generatori seguenti:</span><span class="sxs-lookup"><span data-stu-id="d16f9-117">The following generators are available:</span></span>

| <span data-ttu-id="d16f9-118">Generator</span><span class="sxs-lookup"><span data-stu-id="d16f9-118">Generator</span></span> | <span data-ttu-id="d16f9-119">Operazione</span><span class="sxs-lookup"><span data-stu-id="d16f9-119">Operation</span></span> |
| ----------------- | ------------ | 
| <span data-ttu-id="d16f9-120">area</span><span class="sxs-lookup"><span data-stu-id="d16f9-120">area</span></span>      | [<span data-ttu-id="d16f9-121">Esegue lo scaffolding di un'area</span><span class="sxs-lookup"><span data-stu-id="d16f9-121">Scaffolds an Area</span></span>](/aspnet/core/mvc/controllers/areas) |
  <span data-ttu-id="d16f9-122">controller</span><span class="sxs-lookup"><span data-stu-id="d16f9-122">controller</span></span>| [<span data-ttu-id="d16f9-123">Esegue lo scaffolding di un controller</span><span class="sxs-lookup"><span data-stu-id="d16f9-123">Scaffolds a controller</span></span>](/aspnet/core/tutorials/first-mvc-app/adding-model) |
  <span data-ttu-id="d16f9-124">identity</span><span class="sxs-lookup"><span data-stu-id="d16f9-124">identity</span></span>  | [<span data-ttu-id="d16f9-125">Esegue lo scaffolding dell'identità</span><span class="sxs-lookup"><span data-stu-id="d16f9-125">Scaffolds Identity</span></span>](/aspnet/core/security/authentication/scaffold-identity) |
  <span data-ttu-id="d16f9-126">razorpage</span><span class="sxs-lookup"><span data-stu-id="d16f9-126">razorpage</span></span> | [<span data-ttu-id="d16f9-127">Esegue lo scaffolding di Razor Pages</span><span class="sxs-lookup"><span data-stu-id="d16f9-127">Scaffolds Razor Pages</span></span>](/aspnet/core/tutorials/razor-pages/model) |
  <span data-ttu-id="d16f9-128">view</span><span class="sxs-lookup"><span data-stu-id="d16f9-128">view</span></span>      | [<span data-ttu-id="d16f9-129">Esegue lo scaffolding di una visualizzazione</span><span class="sxs-lookup"><span data-stu-id="d16f9-129">Scaffolds a view</span></span>](/aspnet/core/mvc/views/overview) |

## <a name="options"></a><span data-ttu-id="d16f9-130">Opzioni</span><span class="sxs-lookup"><span data-stu-id="d16f9-130">Options</span></span>

`-n|--nuget-package-dir`

<span data-ttu-id="d16f9-131">Specifica la directory del pacchetto NuGet.</span><span class="sxs-lookup"><span data-stu-id="d16f9-131">Specifies the NuGet package directory.</span></span>

`-c|--configuration {Debug|Release}`

<span data-ttu-id="d16f9-132">Definisce la configurazione di compilazione.</span><span class="sxs-lookup"><span data-stu-id="d16f9-132">Defines the build configuration.</span></span> <span data-ttu-id="d16f9-133">Il valore predefinito è `Debug`.</span><span class="sxs-lookup"><span data-stu-id="d16f9-133">The default value is `Debug`.</span></span>

`-tfm|--target-framework`

<span data-ttu-id="d16f9-134">[Framework](/dotnet/standard/frameworks) di destinazione da usare.</span><span class="sxs-lookup"><span data-stu-id="d16f9-134">Target [Framework](/dotnet/standard/frameworks) to use.</span></span> <span data-ttu-id="d16f9-135">Ad esempio `net46`.</span><span class="sxs-lookup"><span data-stu-id="d16f9-135">For example, `net46`.</span></span>

`-b|--build-base-path`

<span data-ttu-id="d16f9-136">Percorso di base di compilazione.</span><span class="sxs-lookup"><span data-stu-id="d16f9-136">The build base path.</span></span>

`-h|--help`

<span data-ttu-id="d16f9-137">Stampa una breve guida per il comando.</span><span class="sxs-lookup"><span data-stu-id="d16f9-137">Prints out a short help for the command.</span></span>

`--no-build`

<span data-ttu-id="d16f9-138">Non compila il progetto prima dell'esecuzione.</span><span class="sxs-lookup"><span data-stu-id="d16f9-138">Doesn't build the project before running.</span></span> <span data-ttu-id="d16f9-139">Imposta anche in modo implicito il flag `--no-restore`.</span><span class="sxs-lookup"><span data-stu-id="d16f9-139">It also implicitly sets the `--no-restore` flag.</span></span>

`-p|--project <PATH>`

<span data-ttu-id="d16f9-140">Specifica il percorso del file di progetto da eseguire: nome della cartella o percorso completo.</span><span class="sxs-lookup"><span data-stu-id="d16f9-140">Specifies the path of the project file to run (folder name or full path).</span></span> <span data-ttu-id="d16f9-141">Se non specificato, per impostazione predefinita il percorso corrisponde alla directory corrente.</span><span class="sxs-lookup"><span data-stu-id="d16f9-141">If not specified, it defaults to the current directory.</span></span>

## <a name="generator-options"></a><span data-ttu-id="d16f9-142">Opzioni del generatore</span><span class="sxs-lookup"><span data-stu-id="d16f9-142">Generator options</span></span>

<span data-ttu-id="d16f9-143">Nelle sezioni seguenti vengono descritte in dettaglio le opzioni disponibili per i generatori supportati:</span><span class="sxs-lookup"><span data-stu-id="d16f9-143">The following sections detail the options available for the supported generators:</span></span>

* <span data-ttu-id="d16f9-144">Area</span><span class="sxs-lookup"><span data-stu-id="d16f9-144">Area</span></span>
* <span data-ttu-id="d16f9-145">Controller</span><span class="sxs-lookup"><span data-stu-id="d16f9-145">Controller</span></span>
* <span data-ttu-id="d16f9-146">identità</span><span class="sxs-lookup"><span data-stu-id="d16f9-146">Identity</span></span>  
* <span data-ttu-id="d16f9-147">Razorpage</span><span class="sxs-lookup"><span data-stu-id="d16f9-147">Razorpage</span></span>
* <span data-ttu-id="d16f9-148">Visualizza</span><span class="sxs-lookup"><span data-stu-id="d16f9-148">View</span></span>

<a name="area"></a>

### <a name="area-options"></a><span data-ttu-id="d16f9-149">Opzioni per area</span><span class="sxs-lookup"><span data-stu-id="d16f9-149">Area options</span></span>

<span data-ttu-id="d16f9-150">Questo strumento è progettato per i progetti Web ASP.NET Core con controller e visualizzazioni.</span><span class="sxs-lookup"><span data-stu-id="d16f9-150">This tool is intended for ASP.NET Core web projects with controllers and views.</span></span> <span data-ttu-id="d16f9-151">Non è destinato alle app Razor Pages.</span><span class="sxs-lookup"><span data-stu-id="d16f9-151">It's not intended for Razor Pages apps.</span></span>

<span data-ttu-id="d16f9-152">Sintassi: `dotnet aspnet-codegenerator area AreaNameToGenerate`</span><span class="sxs-lookup"><span data-stu-id="d16f9-152">Usage: `dotnet aspnet-codegenerator area AreaNameToGenerate`</span></span>

<span data-ttu-id="d16f9-153">Il comando precedente genera le cartelle seguenti:</span><span class="sxs-lookup"><span data-stu-id="d16f9-153">The preceding command generates the following folders:</span></span>

* <span data-ttu-id="d16f9-154">*Aree*</span><span class="sxs-lookup"><span data-stu-id="d16f9-154">*Areas*</span></span>
  * <span data-ttu-id="d16f9-155">*AreaNameToGenerate*</span><span class="sxs-lookup"><span data-stu-id="d16f9-155">*AreaNameToGenerate*</span></span>
    * <span data-ttu-id="d16f9-156">*Controller*</span><span class="sxs-lookup"><span data-stu-id="d16f9-156">*Controllers*</span></span>
    * <span data-ttu-id="d16f9-157">*Dati*</span><span class="sxs-lookup"><span data-stu-id="d16f9-157">*Data*</span></span>
    * <span data-ttu-id="d16f9-158">*Models*</span><span class="sxs-lookup"><span data-stu-id="d16f9-158">*Models*</span></span>
    * <span data-ttu-id="d16f9-159">*Visualizzazioni*</span><span class="sxs-lookup"><span data-stu-id="d16f9-159">*Views*</span></span>

<a name="ctl"></a>

### <a name="controller-options"></a><span data-ttu-id="d16f9-160">Opzioni per controller</span><span class="sxs-lookup"><span data-stu-id="d16f9-160">Controller options</span></span>

<span data-ttu-id="d16f9-161">La tabella seguente contiene un elenco di opzioni per `aspnet-codegenerator` `controller` e `razorpage`:</span><span class="sxs-lookup"><span data-stu-id="d16f9-161">The following table lists options for  `aspnet-codegenerator` `controller` and `razorpage`:</span></span>

[!INCLUDE [aspnet-codegenerator-args-md.md](~/includes/aspnet-codegenerator-args-md.md)]

<span data-ttu-id="d16f9-162">La tabella seguente contiene un elenco di opzioni specifiche per `aspnet-codegenerator controller`:</span><span class="sxs-lookup"><span data-stu-id="d16f9-162">The following table lists options unique to  `aspnet-codegenerator controller`:</span></span>

| <span data-ttu-id="d16f9-163">Opzione</span><span class="sxs-lookup"><span data-stu-id="d16f9-163">Option</span></span>               | <span data-ttu-id="d16f9-164">DESCRIZIONE</span><span class="sxs-lookup"><span data-stu-id="d16f9-164">Description</span></span>|
| ----------------- | ------------ |
| <span data-ttu-id="d16f9-165">--controllerName o -name</span><span class="sxs-lookup"><span data-stu-id="d16f9-165">--controllerName or -name</span></span> | <span data-ttu-id="d16f9-166">Nome del controller.</span><span class="sxs-lookup"><span data-stu-id="d16f9-166">Name of the controller.</span></span> |
| <span data-ttu-id="d16f9-167">--useAsyncActions o -async</span><span class="sxs-lookup"><span data-stu-id="d16f9-167">--useAsyncActions or -async</span></span> | <span data-ttu-id="d16f9-168">Genera le azioni del controller asincrone.</span><span class="sxs-lookup"><span data-stu-id="d16f9-168">Generate async controller actions.</span></span> |
| <span data-ttu-id="d16f9-169">--noViews or -nv</span><span class="sxs-lookup"><span data-stu-id="d16f9-169">--noViews or -nv</span></span> | <span data-ttu-id="d16f9-170">**Non** genera alcuna visualizzazione.</span><span class="sxs-lookup"><span data-stu-id="d16f9-170">Generate **no** views.</span></span> |
| <span data-ttu-id="d16f9-171">--restWithNoViews o -api</span><span class="sxs-lookup"><span data-stu-id="d16f9-171">--restWithNoViews or -api</span></span>  | <span data-ttu-id="d16f9-172">Genera un controller con l'API in stile REST.</span><span class="sxs-lookup"><span data-stu-id="d16f9-172">Generate a Controller with REST style API.</span></span> <span data-ttu-id="d16f9-173">Si presuppone `noViews` e qualsiasi opzione correlata alla visualizzazione viene ignorata.</span><span class="sxs-lookup"><span data-stu-id="d16f9-173">`noViews` is assumed and any view related options are ignored.</span></span> |
| <span data-ttu-id="d16f9-174">--readWriteActions o -actions</span><span class="sxs-lookup"><span data-stu-id="d16f9-174">--readWriteActions or -actions</span></span> | <span data-ttu-id="d16f9-175">Genera un controller con azioni di lettura/scrittura senza un modello.</span><span class="sxs-lookup"><span data-stu-id="d16f9-175">Generate controller with read/write actions without a model.</span></span> |

<span data-ttu-id="d16f9-176">Usare l'opzione `-h` per ottenere informazioni della Guida per il comando `aspnet-codegenerator controller`:</span><span class="sxs-lookup"><span data-stu-id="d16f9-176">Use the `-h` switch for help on the `aspnet-codegenerator controller` command:</span></span>

```console
dotnet aspnet-codegenerator controller -h
```

<span data-ttu-id="d16f9-177">Vedere [Eseguire lo scaffolding del modello di filmato](/aspnet/core/tutorials/razor-pages/model) per un esempio di `dotnet aspnet-codegenerator controller`.</span><span class="sxs-lookup"><span data-stu-id="d16f9-177">See [Scaffold the movie model](/aspnet/core/tutorials/razor-pages/model) for an example of `dotnet aspnet-codegenerator controller`.</span></span>

### <a name="razorpage"></a><span data-ttu-id="d16f9-178">Razorpage</span><span class="sxs-lookup"><span data-stu-id="d16f9-178">Razorpage</span></span>

<a name="rp"></a>

<span data-ttu-id="d16f9-179">È possibile eseguire lo scaffolding di singole pagine Razor specificando il nome della nuova pagina e il modello da usare.</span><span class="sxs-lookup"><span data-stu-id="d16f9-179">Razor Pages can be individually scaffolded by specifying the name of the new page and the template to use.</span></span> <span data-ttu-id="d16f9-180">I modelli supportati sono:</span><span class="sxs-lookup"><span data-stu-id="d16f9-180">The supported templates are:</span></span>

* `Empty`
* `Create`
* `Edit`
* `Delete`
* `Details`
* `List`

<span data-ttu-id="d16f9-181">Ad esempio, il comando seguente usa il modello Edit per generare *MyEdit.cshtml* e *MyEdit.cshtml.cs*:</span><span class="sxs-lookup"><span data-stu-id="d16f9-181">For example, the following command uses the Edit template to generate *MyEdit.cshtml* and *MyEdit.cshtml.cs*:</span></span>

```console
dotnet aspnet-codegenerator razorpage MyEdit Edit -m Movie -dc RazorPagesMovieContext -outDir Pages/Movies
```

<span data-ttu-id="d16f9-182">In genere, il modello e il nome di file generato non vengono specificati e vengono creati i modelli seguenti:</span><span class="sxs-lookup"><span data-stu-id="d16f9-182">Typically, the template and generated file name is not specified, and the following templates are created:</span></span>

* `Create`
* `Edit`
* `Delete`
* `Details`
* `List`

<span data-ttu-id="d16f9-183">La tabella seguente contiene un elenco di opzioni per `aspnet-codegenerator` `razorpage` e `controller`:</span><span class="sxs-lookup"><span data-stu-id="d16f9-183">The following table lists options for  `aspnet-codegenerator` `razorpage` and `controller`:</span></span>

[!INCLUDE [aspnet-codegenerator-args-md.md](~/includes/aspnet-codegenerator-args-md.md)]

<span data-ttu-id="d16f9-184">La tabella seguente contiene un elenco di opzioni specifiche per `aspnet-codegenerator razorpage`:</span><span class="sxs-lookup"><span data-stu-id="d16f9-184">The following table lists options unique to  `aspnet-codegenerator razorpage`:</span></span>

| <span data-ttu-id="d16f9-185">Opzione</span><span class="sxs-lookup"><span data-stu-id="d16f9-185">Option</span></span>               | <span data-ttu-id="d16f9-186">DESCRIZIONE</span><span class="sxs-lookup"><span data-stu-id="d16f9-186">Description</span></span>|
| ----------------- | ------------ |
|   <span data-ttu-id="d16f9-187">--namespaceName o -namespace</span><span class="sxs-lookup"><span data-stu-id="d16f9-187">--namespaceName or -namespace</span></span> | <span data-ttu-id="d16f9-188">Nome dello spazio dei nomi da usare per il PageModel generato</span><span class="sxs-lookup"><span data-stu-id="d16f9-188">The name of the namespace to use for the generated PageModel</span></span> |
| <span data-ttu-id="d16f9-189">--partialView o -partial</span><span class="sxs-lookup"><span data-stu-id="d16f9-189">--partialView or -partial</span></span> | <span data-ttu-id="d16f9-190">Genera una visualizzazione parziale.</span><span class="sxs-lookup"><span data-stu-id="d16f9-190">Generate a partial view.</span></span> <span data-ttu-id="d16f9-191">Le opzioni di layout -l e -udl vengono ignorate se viene specificata questa opzione.</span><span class="sxs-lookup"><span data-stu-id="d16f9-191">Layout options -l and -udl are ignored if this is specified.</span></span> |
| <span data-ttu-id="d16f9-192">--noPageModel o -npm</span><span class="sxs-lookup"><span data-stu-id="d16f9-192">--noPageModel or -npm</span></span> | <span data-ttu-id="d16f9-193">Opzione per non generare una classe PageModel per un modello vuoto</span><span class="sxs-lookup"><span data-stu-id="d16f9-193">Switch to not generate a PageModel class for Empty template</span></span> |

<span data-ttu-id="d16f9-194">Usare l'opzione `-h` per ottenere informazioni della Guida per il comando `aspnet-codegenerator razorpage`:</span><span class="sxs-lookup"><span data-stu-id="d16f9-194">Use the `-h` switch for help on the `aspnet-codegenerator razorpage` command:</span></span>

```console
dotnet aspnet-codegenerator razorpage -h
```

<span data-ttu-id="d16f9-195">Vedere [Eseguire lo scaffolding del modello di filmato](/aspnet/core/tutorials/razor-pages/model) per un esempio di `dotnet aspnet-codegenerator razorpage`.</span><span class="sxs-lookup"><span data-stu-id="d16f9-195">See [Scaffold the movie model](/aspnet/core/tutorials/razor-pages/model) for an example of `dotnet aspnet-codegenerator razorpage`.</span></span>

### <a name="identity"></a><span data-ttu-id="d16f9-196">identità</span><span class="sxs-lookup"><span data-stu-id="d16f9-196">Identity</span></span>

<span data-ttu-id="d16f9-197">Vedere [Scaffolding dell'identità](/aspnet/core/security/authentication/scaffold-identity)</span><span class="sxs-lookup"><span data-stu-id="d16f9-197">See [Scaffold Identity](/aspnet/core/security/authentication/scaffold-identity)</span></span>