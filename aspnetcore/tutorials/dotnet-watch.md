---
title: Sviluppare app ASP.NET Core usando un watcher per file
author: rick-anderson
description: Questa esercitazione illustra come installare e usare lo strumento watcher per file (dotnet watch) dell'interfaccia della riga di comando di .NET Core in un'app ASP.NET Core.
ms.author: riande
ms.date: 05/31/2018
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
uid: tutorials/dotnet-watch
ms.openlocfilehash: 84cae3b3babe28c2ebf6dba50023b020112d1bb3
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102587580"
---
# <a name="develop-aspnet-core-apps-using-a-file-watcher"></a><span data-ttu-id="006e5-103">Sviluppare app ASP.NET Core usando un watcher per file</span><span class="sxs-lookup"><span data-stu-id="006e5-103">Develop ASP.NET Core apps using a file watcher</span></span>

<span data-ttu-id="006e5-104">Di [Rick Anderson](https://twitter.com/RickAndMSFT) e [Victor Hurdugaci](https://twitter.com/victorhurdugaci)</span><span class="sxs-lookup"><span data-stu-id="006e5-104">By [Rick Anderson](https://twitter.com/RickAndMSFT) and [Victor Hurdugaci](https://twitter.com/victorhurdugaci)</span></span>

<span data-ttu-id="006e5-105">`dotnet watch` è uno strumento che esegue un comando [interfaccia della riga di comando di .NET Core](/dotnet/core/tools) quando i file di origine cambiano.</span><span class="sxs-lookup"><span data-stu-id="006e5-105">`dotnet watch` is a tool that runs a [.NET Core CLI](/dotnet/core/tools) command when source files change.</span></span> <span data-ttu-id="006e5-106">Ad esempio, una modifica di file può attivare la compilazione, l'esecuzione di test o la distribuzione.</span><span class="sxs-lookup"><span data-stu-id="006e5-106">For example, a file change can trigger compilation, test execution, or deployment.</span></span>

<span data-ttu-id="006e5-107">Questa esercitazione usa un'API Web esistente con due endpoint, di cui uno restituisce una somma e l'altro un prodotto.</span><span class="sxs-lookup"><span data-stu-id="006e5-107">This tutorial uses an existing web API with two endpoints: one that returns a sum and one that returns a product.</span></span> <span data-ttu-id="006e5-108">Il metodo del prodotto ha un bug, che verrà corretto in questa esercitazione.</span><span class="sxs-lookup"><span data-stu-id="006e5-108">The product method has a bug, which is fixed in this tutorial.</span></span>

<span data-ttu-id="006e5-109">Scaricare l' [app di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/dotnet-watch/sample).</span><span class="sxs-lookup"><span data-stu-id="006e5-109">Download the [sample app](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/dotnet-watch/sample).</span></span> <span data-ttu-id="006e5-110">Questa è costituita da due progetti: *WebApp* (un'API Web di ASP.NET Core) e *WebAppTests* (unit test per l'API Web).</span><span class="sxs-lookup"><span data-stu-id="006e5-110">It consists of two projects: *WebApp* (an ASP.NET Core web API) and *WebAppTests* (unit tests for the web API).</span></span>

<span data-ttu-id="006e5-111">In una shell dei comandi passare alla cartella *WebApp*.</span><span class="sxs-lookup"><span data-stu-id="006e5-111">In a command shell, navigate to the *WebApp* folder.</span></span> <span data-ttu-id="006e5-112">Eseguire il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="006e5-112">Run the following command:</span></span>

```dotnetcli
dotnet run
```

> [!NOTE]
> <span data-ttu-id="006e5-113">È possibile usare `dotnet run --project <PROJECT>` per specificare un progetto da eseguire.</span><span class="sxs-lookup"><span data-stu-id="006e5-113">You can use `dotnet run --project <PROJECT>` to specify a project to run.</span></span> <span data-ttu-id="006e5-114">Ad esempio, l'esecuzione di `dotnet run --project WebApp` dalla radice dell'app di esempio consentirà di eseguire anche il progetto *WebApp*.</span><span class="sxs-lookup"><span data-stu-id="006e5-114">For example, running `dotnet run --project WebApp` from the root of the sample app will also run the *WebApp* project.</span></span>

<span data-ttu-id="006e5-115">L'output della console visualizza messaggi simili al seguente che indicano che l'app è in esecuzione e in attesa di richieste:</span><span class="sxs-lookup"><span data-stu-id="006e5-115">The console output shows messages similar to the following (indicating that the app is running and awaiting requests):</span></span>

```console
$ dotnet run
Hosting environment: Development
Content root path: C:/Docs/aspnetcore/tutorials/dotnet-watch/sample/WebApp
Now listening on: http://localhost:5000
Application started. Press Ctrl+C to shut down.
```

<span data-ttu-id="006e5-116">In un Web browser passare a `http://localhost:<port number>/api/math/sum?a=4&b=5`.</span><span class="sxs-lookup"><span data-stu-id="006e5-116">In a web browser, navigate to `http://localhost:<port number>/api/math/sum?a=4&b=5`.</span></span> <span data-ttu-id="006e5-117">Dovrebbe essere visualizzato il risultato `9`.</span><span class="sxs-lookup"><span data-stu-id="006e5-117">You should see the result of `9`.</span></span>

<span data-ttu-id="006e5-118">Passare all'API del prodotto (`http://localhost:<port number>/api/math/product?a=4&b=5`).</span><span class="sxs-lookup"><span data-stu-id="006e5-118">Navigate to the product API (`http://localhost:<port number>/api/math/product?a=4&b=5`).</span></span> <span data-ttu-id="006e5-119">Restituisce `9` e non `20` come previsto.</span><span class="sxs-lookup"><span data-stu-id="006e5-119">It returns `9`, not `20` as you'd expect.</span></span> <span data-ttu-id="006e5-120">Questo problema verrà corretto più avanti nell'esercitazione.</span><span class="sxs-lookup"><span data-stu-id="006e5-120">That problem is fixed later in the tutorial.</span></span>

::: moniker range="<= aspnetcore-2.0"

## <a name="add-dotnet-watch-to-a-project"></a><span data-ttu-id="006e5-121">Aggiungere `dotnet watch` a un progetto</span><span class="sxs-lookup"><span data-stu-id="006e5-121">Add `dotnet watch` to a project</span></span>

<span data-ttu-id="006e5-122">Lo strumento watcher per file `dotnet watch` è incluso nella versione 2.1.300 di .NET Core SDK.</span><span class="sxs-lookup"><span data-stu-id="006e5-122">The `dotnet watch` file watcher tool is included with version 2.1.300 of the .NET Core SDK.</span></span> <span data-ttu-id="006e5-123">Se si usa una versione precedente di .NET Core SDK, i passaggi seguenti sono obbligatori.</span><span class="sxs-lookup"><span data-stu-id="006e5-123">The following steps are required when using an earlier version of the .NET Core SDK.</span></span>

1. <span data-ttu-id="006e5-124">Aggiungere un riferimento al pacchetto `Microsoft.DotNet.Watcher.Tools` nel file *.csproj*:</span><span class="sxs-lookup"><span data-stu-id="006e5-124">Add a `Microsoft.DotNet.Watcher.Tools` package reference to the *.csproj* file:</span></span>

    ```xml
    <ItemGroup>
        <DotNetCliToolReference Include="Microsoft.DotNet.Watcher.Tools" Version="2.0.0" />
    </ItemGroup>
    ```

1. <span data-ttu-id="006e5-125">Installare il pacchetto `Microsoft.DotNet.Watcher.Tools` eseguendo il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="006e5-125">Install the `Microsoft.DotNet.Watcher.Tools` package by running the following command:</span></span>

    ```dotnetcli
    dotnet restore
    ```

::: moniker-end

## <a name="run-net-core-cli-commands-using-dotnet-watch"></a><span data-ttu-id="006e5-126">Eseguire i comandi dell'interfaccia della riga di comando di .NET Core con `dotnet watch`</span><span class="sxs-lookup"><span data-stu-id="006e5-126">Run .NET Core CLI commands using `dotnet watch`</span></span>

<span data-ttu-id="006e5-127">Qualsiasi [comando dell'interfaccia della riga di comando di .NET Core](/dotnet/core/tools#cli-commands) può essere eseguito con `dotnet watch`.</span><span class="sxs-lookup"><span data-stu-id="006e5-127">Any [.NET Core CLI command](/dotnet/core/tools#cli-commands) can be run with `dotnet watch`.</span></span> <span data-ttu-id="006e5-128">Ad esempio:</span><span class="sxs-lookup"><span data-stu-id="006e5-128">For example:</span></span>

| <span data-ttu-id="006e5-129">Comando</span><span class="sxs-lookup"><span data-stu-id="006e5-129">Command</span></span> | <span data-ttu-id="006e5-130">Comando con watch</span><span class="sxs-lookup"><span data-stu-id="006e5-130">Command with watch</span></span> |
| ---- | ----- |
| <span data-ttu-id="006e5-131">dotnet run</span><span class="sxs-lookup"><span data-stu-id="006e5-131">dotnet run</span></span> | <span data-ttu-id="006e5-132">dotnet watch run</span><span class="sxs-lookup"><span data-stu-id="006e5-132">dotnet watch run</span></span> |
| <span data-ttu-id="006e5-133">esecuzione DotNet-f netcoreapp 3.1</span><span class="sxs-lookup"><span data-stu-id="006e5-133">dotnet run -f netcoreapp3.1</span></span> | <span data-ttu-id="006e5-134">esecuzione di DotNet Watch-f netcoreapp 3.1</span><span class="sxs-lookup"><span data-stu-id="006e5-134">dotnet watch run -f netcoreapp3.1</span></span> |
| <span data-ttu-id="006e5-135">DotNet Run-f netcoreapp 3.1----arg1</span><span class="sxs-lookup"><span data-stu-id="006e5-135">dotnet run -f netcoreapp3.1 -- --arg1</span></span> | <span data-ttu-id="006e5-136">esecuzione di DotNet Watch-f netcoreapp 3.1----arg1</span><span class="sxs-lookup"><span data-stu-id="006e5-136">dotnet watch run -f netcoreapp3.1 -- --arg1</span></span> |
| <span data-ttu-id="006e5-137">dotnet test</span><span class="sxs-lookup"><span data-stu-id="006e5-137">dotnet test</span></span> | <span data-ttu-id="006e5-138">dotnet watch test</span><span class="sxs-lookup"><span data-stu-id="006e5-138">dotnet watch test</span></span> |

<span data-ttu-id="006e5-139">Eseguire `dotnet watch run` nella cartella *WebApp*.</span><span class="sxs-lookup"><span data-stu-id="006e5-139">Run `dotnet watch run` in the *WebApp* folder.</span></span> <span data-ttu-id="006e5-140">L'output della console indica che `watch` è stato avviato.</span><span class="sxs-lookup"><span data-stu-id="006e5-140">The console output indicates `watch` has started.</span></span>

::: moniker range=">= aspnetcore-5.0"
<span data-ttu-id="006e5-141">In esecuzione `dotnet watch run` in un'app Web viene avviato un browser che consente di passare all'URL dell'app al termine della preparazione.</span><span class="sxs-lookup"><span data-stu-id="006e5-141">Running `dotnet watch run` on a web app launches a browser that navigates to the app's URL once ready.</span></span> <span data-ttu-id="006e5-142">`dotnet watch` a tale scopo, leggere l'output della console dell'app e attendere il messaggio pronto visualizzato da <xref:Microsoft.AspNetCore.WebHost> .</span><span class="sxs-lookup"><span data-stu-id="006e5-142">`dotnet watch` does this by reading the app's console output and waiting for the the ready message displayed by <xref:Microsoft.AspNetCore.WebHost>.</span></span>

<span data-ttu-id="006e5-143">`dotnet watch` Aggiorna il browser quando rileva le modifiche apportate ai file osservati.</span><span class="sxs-lookup"><span data-stu-id="006e5-143">`dotnet watch` refreshes the browser when it detects changes to watched files.</span></span> <span data-ttu-id="006e5-144">A tale scopo, il comando Watch inserisce un middleware nell'app che modifica le risposte HTML create dall'app.</span><span class="sxs-lookup"><span data-stu-id="006e5-144">To do this, the watch command injects a middleware to the app that modifies HTML responses created by the app.</span></span> <span data-ttu-id="006e5-145">Il middleware aggiunge un blocco di script JavaScript alla pagina che consente `dotnet watch` di impostare il browser per l'aggiornamento.</span><span class="sxs-lookup"><span data-stu-id="006e5-145">The middleware adds a JavaScript script block to the page that allows `dotnet watch` to instruct the browser to refresh.</span></span> <span data-ttu-id="006e5-146">Attualmente, le modifiche apportate a tutti i file controllati, incluso il contenuto statico, ad esempio i file *HTML* e *CSS* , determinano la ricompilazione dell'app.</span><span class="sxs-lookup"><span data-stu-id="006e5-146">Currently, changes to all watched files, including static content such as *.html* and *.css* files cause the app to be rebuilt.</span></span>

<span data-ttu-id="006e5-147">`dotnet watch`:</span><span class="sxs-lookup"><span data-stu-id="006e5-147">`dotnet watch`:</span></span>

* <span data-ttu-id="006e5-148">Controlla solo i file che incidono sulle compilazioni per impostazione predefinita.</span><span class="sxs-lookup"><span data-stu-id="006e5-148">Only watches files that impact builds by default.</span></span>
* <span data-ttu-id="006e5-149">Eventuali altri file controllati (tramite configurazione) continuano a generare una compilazione.</span><span class="sxs-lookup"><span data-stu-id="006e5-149">Any additionally watched files (via configuration) still results in a build taking place.</span></span>

<span data-ttu-id="006e5-150">Per ulteriori informazioni sulla configurazione, vedere la pagina relativa alla [configurazione di DotNet-Watch](#dotnet-watch-configuration) in questo documento.</span><span class="sxs-lookup"><span data-stu-id="006e5-150">For more information on configuration, see [dotnet-watch configuration](#dotnet-watch-configuration) in this document</span></span>

::: moniker-end

> [!NOTE]
> <span data-ttu-id="006e5-151">È possibile usare `dotnet watch --project <PROJECT>` per specificare un progetto da controllare.</span><span class="sxs-lookup"><span data-stu-id="006e5-151">You can use `dotnet watch --project <PROJECT>` to specify a project to watch.</span></span> <span data-ttu-id="006e5-152">Ad esempio, l'esecuzione di `dotnet watch --project WebApp run` dalla radice dell'app di esempio consentirà di eseguire e controllare anche il progetto *WebApp*.</span><span class="sxs-lookup"><span data-stu-id="006e5-152">For example, running `dotnet watch --project WebApp run` from the root of the sample app will also run and watch the *WebApp* project.</span></span>

## <a name="make-changes-with-dotnet-watch"></a><span data-ttu-id="006e5-153">Apportare modifiche con `dotnet watch`</span><span class="sxs-lookup"><span data-stu-id="006e5-153">Make changes with `dotnet watch`</span></span>

<span data-ttu-id="006e5-154">Assicurarsi che `dotnet watch` sia in esecuzione.</span><span class="sxs-lookup"><span data-stu-id="006e5-154">Make sure `dotnet watch` is running.</span></span>

<span data-ttu-id="006e5-155">Correggere il bug nel metodo `Product` di *MathController.cs* in modo che restituisca il prodotto e non la somma:</span><span class="sxs-lookup"><span data-stu-id="006e5-155">Fix the bug in the `Product` method of *MathController.cs* so it returns the product and not the sum:</span></span>

```csharp
public static int Product(int a, int b)
{
    return a * b;
}
```

<span data-ttu-id="006e5-156">Salvare il file.</span><span class="sxs-lookup"><span data-stu-id="006e5-156">Save the file.</span></span> <span data-ttu-id="006e5-157">L'output della console indica che `dotnet watch` ha rilevato una modifica del file e ha riavviato l'app.</span><span class="sxs-lookup"><span data-stu-id="006e5-157">The console output indicates that `dotnet watch` detected a file change and restarted the app.</span></span>

<span data-ttu-id="006e5-158">Verificare che `http://localhost:<port number>/api/math/product?a=4&b=5` restituisca il risultato corretto.</span><span class="sxs-lookup"><span data-stu-id="006e5-158">Verify `http://localhost:<port number>/api/math/product?a=4&b=5` returns the correct result.</span></span>

## <a name="run-tests-using-dotnet-watch"></a><span data-ttu-id="006e5-159">Eseguire test con `dotnet watch`</span><span class="sxs-lookup"><span data-stu-id="006e5-159">Run tests using `dotnet watch`</span></span>

1. <span data-ttu-id="006e5-160">Modificare il metodo `Product` di *MathController.cs* in modo che restituisca la somma.</span><span class="sxs-lookup"><span data-stu-id="006e5-160">Change the `Product` method of *MathController.cs* back to returning the sum.</span></span> <span data-ttu-id="006e5-161">Salvare il file.</span><span class="sxs-lookup"><span data-stu-id="006e5-161">Save the file.</span></span>
1. <span data-ttu-id="006e5-162">In una shell dei comandi passare alla cartella *WebAppTests*.</span><span class="sxs-lookup"><span data-stu-id="006e5-162">In a command shell, navigate to the *WebAppTests* folder.</span></span>
1. <span data-ttu-id="006e5-163">Eseguire [dotnet restore](/dotnet/core/tools/dotnet-restore).</span><span class="sxs-lookup"><span data-stu-id="006e5-163">Run [dotnet restore](/dotnet/core/tools/dotnet-restore).</span></span>
1. <span data-ttu-id="006e5-164">Eseguire `dotnet watch test`.</span><span class="sxs-lookup"><span data-stu-id="006e5-164">Run `dotnet watch test`.</span></span> <span data-ttu-id="006e5-165">L'output indica che un test non è stato superato e che il watcher è in attesa di modifiche ai file:</span><span class="sxs-lookup"><span data-stu-id="006e5-165">Its output indicates that a test failed and that the watcher is awaiting file changes:</span></span>

     ```console
     Total tests: 2. Passed: 1. Failed: 1. Skipped: 0.
     Test Run Failed.
     ```

1. <span data-ttu-id="006e5-166">Correggere il codice del metodo `Product` in modo che restituisca il prodotto.</span><span class="sxs-lookup"><span data-stu-id="006e5-166">Fix the `Product` method code so it returns the product.</span></span> <span data-ttu-id="006e5-167">Salvare il file.</span><span class="sxs-lookup"><span data-stu-id="006e5-167">Save the file.</span></span>

<span data-ttu-id="006e5-168">`dotnet watch` rileva la modifica ai file ed esegue di nuovo i test.</span><span class="sxs-lookup"><span data-stu-id="006e5-168">`dotnet watch` detects the file change and reruns the tests.</span></span> <span data-ttu-id="006e5-169">L'output della console indica che i test sono stati superati.</span><span class="sxs-lookup"><span data-stu-id="006e5-169">The console output indicates the tests passed.</span></span>

## <a name="customize-files-list-to-watch"></a><span data-ttu-id="006e5-170">Personalizzare l'elenco dei file da controllare</span><span class="sxs-lookup"><span data-stu-id="006e5-170">Customize files list to watch</span></span>

<span data-ttu-id="006e5-171">Per impostazione predefinita, `dotnet-watch` tiene traccia di tutti i file che soddisfano i criteri GLOB seguenti:</span><span class="sxs-lookup"><span data-stu-id="006e5-171">By default, `dotnet-watch` tracks all files matching the following glob patterns:</span></span>

* `**/*.cs`
* `*.csproj`
* `**/*.resx`

<span data-ttu-id="006e5-172">È possibile aggiungere altri elementi all'elenco di controllo modificando il file con estensione *csproj*.</span><span class="sxs-lookup"><span data-stu-id="006e5-172">More items can be added to the watch list by editing the *.csproj* file.</span></span> <span data-ttu-id="006e5-173">Gli elementi possono essere specificati singolarmente o tramite criteri GLOB.</span><span class="sxs-lookup"><span data-stu-id="006e5-173">Items can be specified individually or by using glob patterns.</span></span>

```xml
<ItemGroup>
    <!-- extends watching group to include *.js files -->
    <Watch Include="**\*.js" Exclude="node_modules\**\*;**\*.js.map;obj\**\*;bin\**\*" />
</ItemGroup>
```

## <a name="opt-out-of-files-to-be-watched"></a><span data-ttu-id="006e5-174">Esclusione di file dal controllo</span><span class="sxs-lookup"><span data-stu-id="006e5-174">Opt-out of files to be watched</span></span>

<span data-ttu-id="006e5-175">È possibile configurare `dotnet-watch` in modo che ignori le impostazioni predefinite.</span><span class="sxs-lookup"><span data-stu-id="006e5-175">`dotnet-watch` can be configured to ignore its default settings.</span></span> <span data-ttu-id="006e5-176">Per ignorare file specifici, aggiungere l'attributo `Watch="false"` alla definizione di un elemento nel file con estensione *csproj*:</span><span class="sxs-lookup"><span data-stu-id="006e5-176">To ignore specific files, add the `Watch="false"` attribute to an item's definition in the *.csproj* file:</span></span>

```xml
<ItemGroup>
    <!-- exclude Generated.cs from dotnet-watch -->
    <Compile Include="Generated.cs" Watch="false" />

    <!-- exclude Strings.resx from dotnet-watch -->
    <EmbeddedResource Include="Strings.resx" Watch="false" />

    <!-- exclude changes in this referenced project -->
    <ProjectReference Include="..\ClassLibrary1\ClassLibrary1.csproj" Watch="false" />
</ItemGroup>
```

## <a name="custom-watch-projects"></a><span data-ttu-id="006e5-177">Progetti di controllo personalizzati</span><span class="sxs-lookup"><span data-stu-id="006e5-177">Custom watch projects</span></span>

<span data-ttu-id="006e5-178">`dotnet-watch` non è limitato a progetti C#.</span><span class="sxs-lookup"><span data-stu-id="006e5-178">`dotnet-watch` isn't restricted to C# projects.</span></span> <span data-ttu-id="006e5-179">È possibile creare progetti di controllo personalizzati per gestire scenari diversi.</span><span class="sxs-lookup"><span data-stu-id="006e5-179">Custom watch projects can be created to handle different scenarios.</span></span> <span data-ttu-id="006e5-180">Si consideri il layout di progetto seguente:</span><span class="sxs-lookup"><span data-stu-id="006e5-180">Consider the following project layout:</span></span>

* <span data-ttu-id="006e5-181">**test**</span><span class="sxs-lookup"><span data-stu-id="006e5-181">**test/**</span></span>
  * <span data-ttu-id="006e5-182">*UnitTests/UnitTests.csproj*</span><span class="sxs-lookup"><span data-stu-id="006e5-182">*UnitTests/UnitTests.csproj*</span></span>
  * <span data-ttu-id="006e5-183">*IntegrationTests/IntegrationTests.csproj*</span><span class="sxs-lookup"><span data-stu-id="006e5-183">*IntegrationTests/IntegrationTests.csproj*</span></span>

<span data-ttu-id="006e5-184">Se l'obiettivo consiste nel controllare entrambi i progetti, creare un file di progetto personalizzato configurato per controllarli entrambi:</span><span class="sxs-lookup"><span data-stu-id="006e5-184">If the goal is to watch both projects, create a custom project file configured to watch both projects:</span></span>

```xml
<Project>
    <ItemGroup>
        <TestProjects Include="**\*.csproj" />
        <Watch Include="**\*.cs" />
    </ItemGroup>

    <Target Name="Test">
        <MSBuild Targets="VSTest" Projects="@(TestProjects)" />
    </Target>

    <Import Project="$(MSBuildExtensionsPath)\Microsoft.Common.targets" />
</Project>
```

<span data-ttu-id="006e5-185">Per avviare il controllo dei file per entrambi i progetti, passare alla cartella *test*.</span><span class="sxs-lookup"><span data-stu-id="006e5-185">To start file watching on both projects, change to the *test* folder.</span></span> <span data-ttu-id="006e5-186">Eseguire il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="006e5-186">Execute the following command:</span></span>

```dotnetcli
dotnet watch msbuild /t:Test
```

<span data-ttu-id="006e5-187">VSTest viene eseguito quando un qualsiasi file viene modificato in uno dei progetti di test.</span><span class="sxs-lookup"><span data-stu-id="006e5-187">VSTest executes when any file changes in either test project.</span></span>

## <a name="dotnet-watch-configuration"></a><span data-ttu-id="006e5-188">configurazione di DotNet-Watch</span><span class="sxs-lookup"><span data-stu-id="006e5-188">dotnet-watch configuration</span></span>

<span data-ttu-id="006e5-189">Alcune opzioni di configurazione possono essere passate a `dotnet watch` tramite le variabili di ambiente.</span><span class="sxs-lookup"><span data-stu-id="006e5-189">Some configuration options can be passed to `dotnet watch` through environment variables.</span></span> <span data-ttu-id="006e5-190">Le variabili disponibili sono:</span><span class="sxs-lookup"><span data-stu-id="006e5-190">The available variables are:</span></span>

| <span data-ttu-id="006e5-191">Impostazione</span><span class="sxs-lookup"><span data-stu-id="006e5-191">Setting</span></span>  | <span data-ttu-id="006e5-192">Descrizione</span><span class="sxs-lookup"><span data-stu-id="006e5-192">Description</span></span> |
| ------------- | ------------- |
| `DOTNET_USE_POLLING_FILE_WATCHER`                | <span data-ttu-id="006e5-193">Se impostato su "1" o "true", `dotnet watch` utilizza un Watcher del file di polling anziché CoreFx `FileSystemWatcher` .</span><span class="sxs-lookup"><span data-stu-id="006e5-193">If set to "1" or "true", `dotnet watch` uses a polling file watcher instead of CoreFx's `FileSystemWatcher`.</span></span> <span data-ttu-id="006e5-194">Usato quando si controllano i file nelle condivisioni di rete o nei volumi montati docker.</span><span class="sxs-lookup"><span data-stu-id="006e5-194">Used when watching files on network shares or Docker mounted volumes.</span></span>                       |
| `DOTNET_WATCH_SUPPRESS_MSBUILD_INCREMENTALISM`   | <span data-ttu-id="006e5-195">Per impostazione predefinita, `dotnet watch` ottimizza la compilazione evitando determinate operazioni, ad esempio l'esecuzione del ripristino o la rivalutazione del set di file controllati per ogni modifica apportata al file.</span><span class="sxs-lookup"><span data-stu-id="006e5-195">By default, `dotnet watch` optimizes the build by avoiding certain operations such as running restore or re-evaluating the set of watched files on every file change.</span></span> <span data-ttu-id="006e5-196">Se impostato su "1" o "true", queste ottimizzazioni sono disabilitate.</span><span class="sxs-lookup"><span data-stu-id="006e5-196">If set to "1" or "true",  these optimizations are disabled.</span></span> |
| `DOTNET_WATCH_SUPPRESS_LAUNCH_BROWSER`   | <span data-ttu-id="006e5-197">`dotnet watch run` tenta di avviare browser per le app Web con `launchBrowser` configurato in *launchSettings.js*.</span><span class="sxs-lookup"><span data-stu-id="006e5-197">`dotnet watch run` attempts to launch browsers for web apps with `launchBrowser` configured in *launchSettings.json*.</span></span> <span data-ttu-id="006e5-198">Se impostato su "1" o "true", questo comportamento viene eliminato.</span><span class="sxs-lookup"><span data-stu-id="006e5-198">If set to "1" or "true", this behavior is suppressed.</span></span> |
| `DOTNET_WATCH_SUPPRESS_BROWSER_REFRESH`   | <span data-ttu-id="006e5-199">`dotnet watch run` tenta di aggiornare i browser quando rileva le modifiche al file.</span><span class="sxs-lookup"><span data-stu-id="006e5-199">`dotnet watch run` attempts to refresh browsers when it detects file changes.</span></span> <span data-ttu-id="006e5-200">Se impostato su "1" o "true", questo comportamento viene eliminato.</span><span class="sxs-lookup"><span data-stu-id="006e5-200">If set to "1" or "true", this behavior is suppressed.</span></span> <span data-ttu-id="006e5-201">Questo comportamento viene anche eliminato se `DOTNET_WATCH_SUPPRESS_LAUNCH_BROWSER` è impostato.</span><span class="sxs-lookup"><span data-stu-id="006e5-201">This behavior is also suppressed if `DOTNET_WATCH_SUPPRESS_LAUNCH_BROWSER` is set.</span></span> |
