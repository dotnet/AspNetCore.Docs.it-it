---
title: ASP.NET Core di debug Blazor WebAssembly
author: guardrex
description: Informazioni su come eseguire il debug delle Blazor app.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 08/26/2020
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
uid: blazor/debug
ms.openlocfilehash: adf22001e7d9b8ee4f36456cd4b07d2791a7331f
ms.sourcegitcommit: 1436bd4d70937d6ec3140da56d96caab33c4320b
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/06/2021
ms.locfileid: "102395150"
---
# <a name="debug-aspnet-core-blazor-webassembly"></a><span data-ttu-id="b96d2-103">ASP.NET Core di debug Blazor WebAssembly</span><span class="sxs-lookup"><span data-stu-id="b96d2-103">Debug ASP.NET Core Blazor WebAssembly</span></span>

<span data-ttu-id="b96d2-104">Blazor WebAssembly è possibile eseguire il debug delle app usando gli strumenti di sviluppo del browser nei browser basati su cromo (Edge/Chrome).</span><span class="sxs-lookup"><span data-stu-id="b96d2-104">Blazor WebAssembly apps can be debugged using the browser dev tools in Chromium-based browsers (Edge/Chrome).</span></span> <span data-ttu-id="b96d2-105">È anche possibile eseguire il debug dell'app usando gli ambienti di sviluppo integrato (IDE) seguenti:</span><span class="sxs-lookup"><span data-stu-id="b96d2-105">You can also debug your app using the following integrated development environments (IDEs):</span></span>

* <span data-ttu-id="b96d2-106">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="b96d2-106">Visual Studio</span></span>
* <span data-ttu-id="b96d2-107">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="b96d2-107">Visual Studio for Mac</span></span>
* <span data-ttu-id="b96d2-108">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="b96d2-108">Visual Studio Code</span></span>

<span data-ttu-id="b96d2-109">Gli scenari disponibili includono:</span><span class="sxs-lookup"><span data-stu-id="b96d2-109">Available scenarios include:</span></span>

* <span data-ttu-id="b96d2-110">Impostare e rimuovere punti di interruzione.</span><span class="sxs-lookup"><span data-stu-id="b96d2-110">Set and remove breakpoints.</span></span>
* <span data-ttu-id="b96d2-111">Eseguire l'app con supporto per il debug in IDE.</span><span class="sxs-lookup"><span data-stu-id="b96d2-111">Run the app with debugging support in IDEs.</span></span>
* <span data-ttu-id="b96d2-112">Singolo passaggio del codice.</span><span class="sxs-lookup"><span data-stu-id="b96d2-112">Single-step through the code.</span></span>
* <span data-ttu-id="b96d2-113">Riprendere l'esecuzione del codice con un tasto di scelta rapida negli IDE.</span><span class="sxs-lookup"><span data-stu-id="b96d2-113">Resume code execution with a keyboard shortcut in IDEs.</span></span>
* <span data-ttu-id="b96d2-114">Nella finestra variabili *locali* osservare i valori delle variabili locali.</span><span class="sxs-lookup"><span data-stu-id="b96d2-114">In the *Locals* window, observe the values of local variables.</span></span>
* <span data-ttu-id="b96d2-115">Vedere lo stack di chiamate, incluse le catene di chiamate tra JavaScript e .NET.</span><span class="sxs-lookup"><span data-stu-id="b96d2-115">See the call stack, including call chains between JavaScript and .NET.</span></span>

<span data-ttu-id="b96d2-116">Per il momento *non è possibile*:</span><span class="sxs-lookup"><span data-stu-id="b96d2-116">For now, you *can't*:</span></span>

* <span data-ttu-id="b96d2-117">Interrompi in corrispondenza di eccezioni non gestite.</span><span class="sxs-lookup"><span data-stu-id="b96d2-117">Break on unhandled exceptions.</span></span>
* <span data-ttu-id="b96d2-118">Premere i punti di interruzione durante l'avvio dell'app prima che il proxy di debug sia in esecuzione.</span><span class="sxs-lookup"><span data-stu-id="b96d2-118">Hit breakpoints during app startup before the debug proxy is running.</span></span> <span data-ttu-id="b96d2-119">Sono inclusi i punti di interruzione in `Program.Main` ( `Program.cs` ) e i punti di interruzione nei [ `OnInitialized{Async}` Metodi](xref:blazor/components/lifecycle#component-initialization-methods) dei componenti caricati dalla prima pagina richiesta dall'app.</span><span class="sxs-lookup"><span data-stu-id="b96d2-119">This includes breakpoints in `Program.Main` (`Program.cs`) and breakpoints in the [`OnInitialized{Async}` methods](xref:blazor/components/lifecycle#component-initialization-methods) of components that are loaded by the first page requested from the app.</span></span>
* <span data-ttu-id="b96d2-120">Eseguire il debug in scenari non locali (ad esempio, il [sottosistema Windows per Linux (WSL)](/windows/wsl/) o gli spazi dei nomi di [Visual Studio](/visualstudio/codespaces/overview/what-is-vsonline)).</span><span class="sxs-lookup"><span data-stu-id="b96d2-120">Debug in non-local scenarios (for example, [Windows Subsystem for Linux (WSL)](/windows/wsl/) or [Visual Studio Codespaces](/visualstudio/codespaces/overview/what-is-vsonline)).</span></span>
* <span data-ttu-id="b96d2-121">Ricompilare automaticamente l'app back-end `*Server*` di una soluzione ospitata Blazor WebAssembly durante il debug, ad esempio eseguendo l'app con [`dotnet watch run`](xref:tutorials/dotnet-watch) .</span><span class="sxs-lookup"><span data-stu-id="b96d2-121">Automatically rebuild the backend `*Server*` app of a hosted Blazor WebAssembly solution during debugging, for example by running the app with [`dotnet watch run`](xref:tutorials/dotnet-watch).</span></span>

## <a name="prerequisites"></a><span data-ttu-id="b96d2-122">Prerequisiti</span><span class="sxs-lookup"><span data-stu-id="b96d2-122">Prerequisites</span></span>

<span data-ttu-id="b96d2-123">Il debug richiede uno dei seguenti browser:</span><span class="sxs-lookup"><span data-stu-id="b96d2-123">Debugging requires either of the following browsers:</span></span>

* <span data-ttu-id="b96d2-124">Google Chrome (versione 70 o successiva) (impostazione predefinita)</span><span class="sxs-lookup"><span data-stu-id="b96d2-124">Google Chrome (version 70 or later) (default)</span></span>
* <span data-ttu-id="b96d2-125">Microsoft Edge (versione 80 o successiva)</span><span class="sxs-lookup"><span data-stu-id="b96d2-125">Microsoft Edge (version 80 or later)</span></span>

<span data-ttu-id="b96d2-126">Assicurarsi che i firewall o i proxy non blocchino le comunicazioni con il proxy di debug ( `NodeJS` processo).</span><span class="sxs-lookup"><span data-stu-id="b96d2-126">Ensure that firewalls or proxies don't block communication with the debug proxy (`NodeJS` process).</span></span> <span data-ttu-id="b96d2-127">Per ulteriori informazioni, vedere la sezione relativa alla [configurazione del firewall](#firewall-configuration) .</span><span class="sxs-lookup"><span data-stu-id="b96d2-127">For more information, see the [Firewall configuration](#firewall-configuration) section.</span></span>

<span data-ttu-id="b96d2-128">Visual Studio Code utenti richiedono le seguenti estensioni:</span><span class="sxs-lookup"><span data-stu-id="b96d2-128">Visual Studio Code users require the following extensions:</span></span>

* [<span data-ttu-id="b96d2-129">C# per estensione Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="b96d2-129">C# for Visual Studio Code Extension</span></span>](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp)
* <span data-ttu-id="b96d2-130">[ Blazor Estensione di debug WASM](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.blazorwasm-companion) (quando si usa C# per Visual Studio Code estensione 1.23.9 o versione successiva)</span><span class="sxs-lookup"><span data-stu-id="b96d2-130">[Blazor WASM Debugging Extension](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.blazorwasm-companion) (when using the C# for Visual Studio Code Extension version 1.23.9 or later)</span></span>

<span data-ttu-id="b96d2-131">Dopo l'apertura di un progetto in VS Code, è possibile che venga ricevuta una notifica che è necessaria un'installazione aggiuntiva per abilitare il debug.</span><span class="sxs-lookup"><span data-stu-id="b96d2-131">After opening a project in VS Code, you may receive a notification that additional setup is required to enable debugging.</span></span> <span data-ttu-id="b96d2-132">Se richiesto, installare le estensioni richieste dal Visual Studio Marketplace.</span><span class="sxs-lookup"><span data-stu-id="b96d2-132">If requested, install the required extensions from the Visual Studio Marketplace.</span></span> <span data-ttu-id="b96d2-133">Per esaminare le estensioni installate, aprire **View**  >  **Extensions** dalla barra dei menu o selezionare l'icona **Extensions** nella barra laterale **Activity** .</span><span class="sxs-lookup"><span data-stu-id="b96d2-133">To inspect the installed extensions, open **View** > **Extensions** from the menu bar or select the **Extensions** icon in the **Activity** sidebar.</span></span>

<span data-ttu-id="b96d2-134">Visual Studio per Mac richiede la versione 8,8 (Build 1532) o versioni successive:</span><span class="sxs-lookup"><span data-stu-id="b96d2-134">Visual Studio for Mac requires version 8.8 (build 1532) or later:</span></span>

1. <span data-ttu-id="b96d2-135">Installare la versione più recente di Visual Studio per Mac selezionando il pulsante **scarica Visual Studio per Mac** in [Microsoft: Visual Studio per Mac](https://visualstudio.microsoft.com/vs/mac/).</span><span class="sxs-lookup"><span data-stu-id="b96d2-135">Install the latest release of Visual Studio for Mac by selecting the **Download Visual Studio for Mac** button at [Microsoft: Visual Studio for Mac](https://visualstudio.microsoft.com/vs/mac/).</span></span>
1. <span data-ttu-id="b96d2-136">Selezionare il canale di *Anteprima* dall'interno di Visual Studio.</span><span class="sxs-lookup"><span data-stu-id="b96d2-136">Select the *Preview* channel from within Visual Studio.</span></span> <span data-ttu-id="b96d2-137">Per ulteriori informazioni, vedere [installare una versione di anteprima di Visual Studio per Mac](/visualstudio/mac/install-preview).</span><span class="sxs-lookup"><span data-stu-id="b96d2-137">For more information, see [Install a preview version of Visual Studio for Mac](/visualstudio/mac/install-preview).</span></span>

> [!NOTE]
> <span data-ttu-id="b96d2-138">Apple Safari in macOS attualmente non è supportato.</span><span class="sxs-lookup"><span data-stu-id="b96d2-138">Apple Safari on macOS isn't currently supported.</span></span>

## <a name="enable-debugging"></a><span data-ttu-id="b96d2-139">Abilitare il debug</span><span class="sxs-lookup"><span data-stu-id="b96d2-139">Enable debugging</span></span>

<span data-ttu-id="b96d2-140">Per abilitare il debug per un' Blazor WebAssembly app esistente, aggiornare il `launchSettings.json` file nel progetto di avvio per includere la `inspectUri` proprietà seguente in ogni profilo di avvio:</span><span class="sxs-lookup"><span data-stu-id="b96d2-140">To enable debugging for an existing Blazor WebAssembly app, update the `launchSettings.json` file in the startup project to include the following `inspectUri` property in each launch profile:</span></span>

```json
"inspectUri": "{wsProtocol}://{url.hostname}:{url.port}/_framework/debug/ws-proxy?browser={browserInspectUri}"
```

<span data-ttu-id="b96d2-141">Al termine dell'aggiornamento, il `launchSettings.json` file dovrebbe essere simile all'esempio seguente:</span><span class="sxs-lookup"><span data-stu-id="b96d2-141">Once updated, the `launchSettings.json` file should look similar to the following example:</span></span>

[!code-json[](debug/launchSettings.json?highlight=14,22)]

<span data-ttu-id="b96d2-142">`inspectUri`Proprietà:</span><span class="sxs-lookup"><span data-stu-id="b96d2-142">The `inspectUri` property:</span></span>

* <span data-ttu-id="b96d2-143">Consente all'IDE di rilevare che l'app è un' Blazor WebAssembly app.</span><span class="sxs-lookup"><span data-stu-id="b96d2-143">Enables the IDE to detect that the app is a Blazor WebAssembly app.</span></span>
* <span data-ttu-id="b96d2-144">Indica all'infrastruttura di debug degli script di connettersi al browser tramite il Blazor proxy di debug.</span><span class="sxs-lookup"><span data-stu-id="b96d2-144">Instructs the script debugging infrastructure to connect to the browser through Blazor's debugging proxy.</span></span>

<span data-ttu-id="b96d2-145">I valori segnaposto per i protocolli WebSockets ( `wsProtocol` ), host ( `url.hostname` ), Port ( `url.port` ) e Inspector URI nel browser avviato ( `browserInspectUri` ) sono forniti dal Framework.</span><span class="sxs-lookup"><span data-stu-id="b96d2-145">The placeholder values for the WebSockets protocol (`wsProtocol`), host (`url.hostname`), port (`url.port`), and inspector URI on the launched browser (`browserInspectUri`) are provided by the framework.</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="b96d2-146">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="b96d2-146">Visual Studio</span></span>](#tab/visual-studio)

<span data-ttu-id="b96d2-147">Per eseguire il debug di un' Blazor WebAssembly app in Visual Studio:</span><span class="sxs-lookup"><span data-stu-id="b96d2-147">To debug a Blazor WebAssembly app in Visual Studio:</span></span>

1. <span data-ttu-id="b96d2-148">Creare una nuova soluzione ospitata Blazor WebAssembly .</span><span class="sxs-lookup"><span data-stu-id="b96d2-148">Create a new hosted Blazor WebAssembly solution.</span></span>
1. <span data-ttu-id="b96d2-149">Premere <kbd>F5</kbd> per eseguire l'app nel debugger.</span><span class="sxs-lookup"><span data-stu-id="b96d2-149">Press <kbd>F5</kbd> to run the app in the debugger.</span></span>

   > [!NOTE]
   > <span data-ttu-id="b96d2-150">**Avvia senza eseguire debug** (<kbd>CTRL</kbd> + <kbd>F5</kbd>) non è supportato.</span><span class="sxs-lookup"><span data-stu-id="b96d2-150">**Start Without Debugging** (<kbd>Ctrl</kbd>+<kbd>F5</kbd>) isn't supported.</span></span> <span data-ttu-id="b96d2-151">Quando l'app viene eseguita nella configurazione di debug, il sovraccarico del debug comporta sempre una riduzione delle prestazioni.</span><span class="sxs-lookup"><span data-stu-id="b96d2-151">When the app is run in Debug configuration, debugging overhead always results in a small performance reduction.</span></span>

1. <span data-ttu-id="b96d2-152">Nell' `*Client*` app, impostare un punto di interruzione nella `currentCount++;` riga in `Pages/Counter.razor` .</span><span class="sxs-lookup"><span data-stu-id="b96d2-152">In the `*Client*` app, set a breakpoint on the `currentCount++;` line in `Pages/Counter.razor`.</span></span>
1. <span data-ttu-id="b96d2-153">Nel browser passare alla `Counter` pagina e selezionare il pulsante **fare clic su me** per raggiungere il punto di interruzione.</span><span class="sxs-lookup"><span data-stu-id="b96d2-153">In the browser, navigate to `Counter` page and select the **Click me** button to hit the breakpoint.</span></span>
1. <span data-ttu-id="b96d2-154">In Visual Studio, controllare il valore del `currentCount` campo nella finestra **variabili locali** .</span><span class="sxs-lookup"><span data-stu-id="b96d2-154">In Visual Studio, inspect the value of the `currentCount` field in the **Locals** window.</span></span>
1. <span data-ttu-id="b96d2-155">Premere <kbd>F5</kbd> per continuare l'esecuzione.</span><span class="sxs-lookup"><span data-stu-id="b96d2-155">Press <kbd>F5</kbd> to continue execution.</span></span>

<span data-ttu-id="b96d2-156">Durante il debug di un' Blazor WebAssembly app, è anche possibile eseguire il debug del codice del server:</span><span class="sxs-lookup"><span data-stu-id="b96d2-156">While debugging a Blazor WebAssembly app, you can also debug server code:</span></span>

1. <span data-ttu-id="b96d2-157">Impostare un punto di interruzione nella `Pages/FetchData.razor` pagina in <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> .</span><span class="sxs-lookup"><span data-stu-id="b96d2-157">Set a breakpoint in the `Pages/FetchData.razor` page in <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A>.</span></span>
1. <span data-ttu-id="b96d2-158">Impostare un punto di interruzione in `WeatherForecastController` nel `Get` metodo di azione.</span><span class="sxs-lookup"><span data-stu-id="b96d2-158">Set a breakpoint in the `WeatherForecastController` in the `Get` action method.</span></span>
1. <span data-ttu-id="b96d2-159">Passare alla `Fetch Data` pagina per raggiungere il primo punto di interruzione nel `FetchData` componente immediatamente prima di eseguire una richiesta HTTP al server.</span><span class="sxs-lookup"><span data-stu-id="b96d2-159">Browse to the `Fetch Data` page to hit the first breakpoint in the `FetchData` component just before it issues an HTTP request to the server.</span></span>
1. <span data-ttu-id="b96d2-160">Premere <kbd>F5</kbd> per continuare l'esecuzione e quindi raggiungere il punto di interruzione sul server nel `WeatherForecastController` .</span><span class="sxs-lookup"><span data-stu-id="b96d2-160">Press <kbd>F5</kbd> to continue execution and then hit the breakpoint on the server in the `WeatherForecastController`.</span></span>
1. <span data-ttu-id="b96d2-161">Premere di nuovo <kbd>F5</kbd> per consentire l'esecuzione continua e visualizzare la tabella delle previsioni meteorologiche sottoposta a rendering nel browser.</span><span class="sxs-lookup"><span data-stu-id="b96d2-161">Press <kbd>F5</kbd> again to let execution continue and see the weather forecast table rendered in the browser.</span></span>

> [!NOTE]
> <span data-ttu-id="b96d2-162">I punti di interruzione **non** vengono raggiunti durante l'avvio dell'app prima che il proxy di debug sia in esecuzione.</span><span class="sxs-lookup"><span data-stu-id="b96d2-162">Breakpoints are **not** hit during app startup before the debug proxy is running.</span></span> <span data-ttu-id="b96d2-163">Sono inclusi i punti di interruzione in `Program.Main` ( `Program.cs` ) e i punti di interruzione nei [ `OnInitialized{Async}` Metodi](xref:blazor/components/lifecycle#component-initialization-methods) dei componenti caricati dalla prima pagina richiesta dall'app.</span><span class="sxs-lookup"><span data-stu-id="b96d2-163">This includes breakpoints in `Program.Main` (`Program.cs`) and breakpoints in the [`OnInitialized{Async}` methods](xref:blazor/components/lifecycle#component-initialization-methods) of components that are loaded by the first page requested from the app.</span></span>

<span data-ttu-id="b96d2-164">Se l'app è ospitata in un [percorso di base dell'app](xref:blazor/host-and-deploy/index#app-base-path) diverso da `/` , aggiornare le proprietà seguenti in `Properties/launchSettings.json` per riflettere il percorso di base dell'app:</span><span class="sxs-lookup"><span data-stu-id="b96d2-164">If the app is hosted at a different [app base path](xref:blazor/host-and-deploy/index#app-base-path) than `/`, update the following properties in `Properties/launchSettings.json` to reflect the app's base path:</span></span>

* <span data-ttu-id="b96d2-165">`applicationUrl`:</span><span class="sxs-lookup"><span data-stu-id="b96d2-165">`applicationUrl`:</span></span>

  ```json
  "iisSettings": {
    ...
    "iisExpress": {
      "applicationUrl": "http://localhost:{INSECURE PORT}/{APP BASE PATH}/",
      "sslPort": {SECURE PORT}
    }
  },
  ```

* <span data-ttu-id="b96d2-166">`inspectUri` di ogni profilo:</span><span class="sxs-lookup"><span data-stu-id="b96d2-166">`inspectUri` of each profile:</span></span>

  ```json
  "profiles": {
    ...
    "{PROFILE 1, 2, ... N}": {
      ...
      "inspectUri": "{wsProtocol}://{url.hostname}:{url.port}/{APP BASE PATH}/_framework/debug/ws-proxy?browser={browserInspectUri}",
      ...
    }
  }
  ```

<span data-ttu-id="b96d2-167">I segnaposto nelle impostazioni precedenti:</span><span class="sxs-lookup"><span data-stu-id="b96d2-167">The placeholders in the preceding settings:</span></span>

* <span data-ttu-id="b96d2-168">`{INSECURE PORT}`: Porta non protetta.</span><span class="sxs-lookup"><span data-stu-id="b96d2-168">`{INSECURE PORT}`: The insecure port.</span></span> <span data-ttu-id="b96d2-169">Per impostazione predefinita viene fornito un valore casuale, ma è consentita una porta personalizzata.</span><span class="sxs-lookup"><span data-stu-id="b96d2-169">A random value is provided by default, but a custom port is permitted.</span></span>
* <span data-ttu-id="b96d2-170">`{APP BASE PATH}`: Percorso di base dell'app.</span><span class="sxs-lookup"><span data-stu-id="b96d2-170">`{APP BASE PATH}`: The app's base path.</span></span>
* <span data-ttu-id="b96d2-171">`{SECURE PORT}`: Porta protetta.</span><span class="sxs-lookup"><span data-stu-id="b96d2-171">`{SECURE PORT}`: The secure port.</span></span> <span data-ttu-id="b96d2-172">Per impostazione predefinita viene fornito un valore casuale, ma è consentita una porta personalizzata.</span><span class="sxs-lookup"><span data-stu-id="b96d2-172">A random value is provided by default, but a custom port is permitted.</span></span>
* <span data-ttu-id="b96d2-173">`{PROFILE 1, 2, ... N}`: Avvia profili impostazioni.</span><span class="sxs-lookup"><span data-stu-id="b96d2-173">`{PROFILE 1, 2, ... N}`: Launch settings profiles.</span></span> <span data-ttu-id="b96d2-174">In genere, un'app specifica più di un profilo per impostazione predefinita (ad esempio, un profilo per IIS Express e un profilo di progetto, che viene usato dal server gheppio).</span><span class="sxs-lookup"><span data-stu-id="b96d2-174">Usually, an app specifies more than one profile by default (for example, a profile for IIS Express and a project profile, which is used by Kestrel server).</span></span>

<span data-ttu-id="b96d2-175">Negli esempi seguenti l'app è ospitata in `/OAT` con un percorso di base dell'app configurato in `wwwroot/index.html` come `<base href="/OAT/">` :</span><span class="sxs-lookup"><span data-stu-id="b96d2-175">In the following examples, the app is hosted at `/OAT` with an app base path configured in `wwwroot/index.html` as `<base href="/OAT/">`:</span></span>

```json
"applicationUrl": "http://localhost:{INSECURE PORT}/OAT/",
```

```json
"inspectUri": "{wsProtocol}://{url.hostname}:{url.port}/OAT/_framework/debug/ws-proxy?browser={browserInspectUri}",
```

<span data-ttu-id="b96d2-176">Per informazioni sull'uso di un percorso di base dell'app personalizzato per le Blazor WebAssembly app, vedere <xref:blazor/host-and-deploy/index#app-base-path> .</span><span class="sxs-lookup"><span data-stu-id="b96d2-176">For information on using a custom app base path for Blazor WebAssembly apps, see <xref:blazor/host-and-deploy/index#app-base-path>.</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="b96d2-177">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="b96d2-177">Visual Studio Code</span></span>](#tab/visual-studio-code)

<h2 id="vscode"><span data-ttu-id="b96d2-178">Debug autonomo Blazor WebAssembly</span><span class="sxs-lookup"><span data-stu-id="b96d2-178">Debug standalone Blazor WebAssembly</span></span></h2>

<span data-ttu-id="b96d2-179">Per informazioni sulla configurazione di asset VS Code nella `.vscode` cartella, vedere la guida del sistema operativo **Linux** in <xref:blazor/tooling> .</span><span class="sxs-lookup"><span data-stu-id="b96d2-179">For information on configuring VS Code assets in the `.vscode` folder, see the **Linux** operating system guidance in <xref:blazor/tooling>.</span></span>

1. <span data-ttu-id="b96d2-180">Aprire l'app autonoma Blazor WebAssembly in vs code.</span><span class="sxs-lookup"><span data-stu-id="b96d2-180">Open the standalone Blazor WebAssembly app in VS Code.</span></span>

   <span data-ttu-id="b96d2-181">È possibile che venga ricevuta una notifica che è necessaria un'installazione aggiuntiva per abilitare il debug:</span><span class="sxs-lookup"><span data-stu-id="b96d2-181">You may receive a notification that additional setup is required to enable debugging:</span></span>

   > <span data-ttu-id="b96d2-182">Per eseguire il debug delle applicazioni, è necessaria un'installazione aggiuntiva Blazor WebAssembly .</span><span class="sxs-lookup"><span data-stu-id="b96d2-182">Additional setup is required to debug Blazor WebAssembly applications.</span></span>

   <span data-ttu-id="b96d2-183">Se si riceve la notifica:</span><span class="sxs-lookup"><span data-stu-id="b96d2-183">If you receive the notification:</span></span>

   * <span data-ttu-id="b96d2-184">Verificare che sia installata la versione più recente [di C# per Visual Studio Code estensione](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp) .</span><span class="sxs-lookup"><span data-stu-id="b96d2-184">Confirm that the latest [C# for Visual Studio Code Extension](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp) is installed.</span></span> <span data-ttu-id="b96d2-185">Per esaminare le estensioni installate, aprire **View**  >  **Extensions** dalla barra dei menu o selezionare l'icona **Extensions** nella barra laterale **Activity** .</span><span class="sxs-lookup"><span data-stu-id="b96d2-185">To inspect the installed extensions, open **View** > **Extensions** from the menu bar or select the **Extensions** icon in the **Activity** sidebar.</span></span>
   * <span data-ttu-id="b96d2-186">Quando si usa [C# per Visual Studio Code estensione](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp) **1.23.9 o versione successiva**, verificare che sia installata l' [ Blazor estensione di debug WASM](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.blazorwasm-companion) più recente.</span><span class="sxs-lookup"><span data-stu-id="b96d2-186">When using the [C# for Visual Studio Code Extension](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp) **version 1.23.9 or later**, confirm that the latest [Blazor WASM Debugging Extension](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.blazorwasm-companion) is installed.</span></span> <span data-ttu-id="b96d2-187">Per esaminare le estensioni installate, aprire **View**  >  **Extensions** dalla barra dei menu o selezionare l'icona **Extensions** nella barra laterale **Activity** .</span><span class="sxs-lookup"><span data-stu-id="b96d2-187">To inspect the installed extensions, open **View** > **Extensions** from the menu bar or select the **Extensions** icon in the **Activity** sidebar.</span></span>
   * <span data-ttu-id="b96d2-188">Verificare che l'anteprima JavaScript sia abilitata.</span><span class="sxs-lookup"><span data-stu-id="b96d2-188">Confirm that JavaScript preview debugging is enabled.</span></span> <span data-ttu-id="b96d2-189">Aprire le impostazioni dalla barra dei menu (  >    >  **Impostazioni** preferenze file).</span><span class="sxs-lookup"><span data-stu-id="b96d2-189">Open the settings from the menu bar (**File** > **Preferences** > **Settings**).</span></span> <span data-ttu-id="b96d2-190">Eseguire la ricerca usando le parole chiave `debug preview` .</span><span class="sxs-lookup"><span data-stu-id="b96d2-190">Search using the keywords `debug preview`.</span></span> <span data-ttu-id="b96d2-191">Nei risultati della ricerca impostare o verificare che la casella di controllo per **Debug > JavaScript: USA anteprima** sia selezionata.</span><span class="sxs-lookup"><span data-stu-id="b96d2-191">In the search results, set or confirm that the check box for **Debug > JavaScript: Use Preview** is checked.</span></span> <span data-ttu-id="b96d2-192">Se l'opzione per abilitare il debug in anteprima non è presente, eseguire l'aggiornamento alla versione più recente di VS Code o installare l' [estensione del debugger JavaScript](https://marketplace.visualstudio.com/items?itemName=ms-vscode.js-debug-nightly) (VS Code versioni 1,46 o precedenti).</span><span class="sxs-lookup"><span data-stu-id="b96d2-192">If the option to enable preview debugging isn't present, either upgrade to the latest version of VS Code or install the [JavaScript Debugger Extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode.js-debug-nightly) (VS Code versions 1.46 or earlier).</span></span>
   * <span data-ttu-id="b96d2-193">Ricaricare la finestra.</span><span class="sxs-lookup"><span data-stu-id="b96d2-193">Reload the window.</span></span>

1. <span data-ttu-id="b96d2-194">Avviare il debug usando il tasto di scelta rapida <kbd>F5</kbd> o la voce di menu.</span><span class="sxs-lookup"><span data-stu-id="b96d2-194">Start debugging using the <kbd>F5</kbd> keyboard shortcut or the menu item.</span></span>

   > [!NOTE]
   > <span data-ttu-id="b96d2-195">**Avvia senza eseguire debug** (<kbd>CTRL</kbd> + <kbd>F5</kbd>) non è supportato.</span><span class="sxs-lookup"><span data-stu-id="b96d2-195">**Start Without Debugging** (<kbd>Ctrl</kbd>+<kbd>F5</kbd>) isn't supported.</span></span> <span data-ttu-id="b96d2-196">Quando l'app viene eseguita nella configurazione di debug, il sovraccarico del debug comporta sempre una riduzione delle prestazioni.</span><span class="sxs-lookup"><span data-stu-id="b96d2-196">When the app is run in Debug configuration, debugging overhead always results in a small performance reduction.</span></span>

1. <span data-ttu-id="b96d2-197">Quando richiesto, selezionare l'opzione **Blazor WebAssembly debug** per avviare il debug.</span><span class="sxs-lookup"><span data-stu-id="b96d2-197">When prompted, select the **Blazor WebAssembly Debug** option to start debugging.</span></span>

1. <span data-ttu-id="b96d2-198">Viene avviata l'app autonoma e viene aperto un browser di debug.</span><span class="sxs-lookup"><span data-stu-id="b96d2-198">The standalone app is launched, and a debugging browser is opened.</span></span>

1. <span data-ttu-id="b96d2-199">Nell' `*Client*` app, impostare un punto di interruzione nella `currentCount++;` riga in `Pages/Counter.razor` .</span><span class="sxs-lookup"><span data-stu-id="b96d2-199">In the `*Client*` app, set a breakpoint on the `currentCount++;` line in `Pages/Counter.razor`.</span></span>

1. <span data-ttu-id="b96d2-200">Nel browser passare alla `Counter` pagina e selezionare il pulsante **fare clic su me** per raggiungere il punto di interruzione.</span><span class="sxs-lookup"><span data-stu-id="b96d2-200">In the browser, navigate to `Counter` page and select the **Click me** button to hit the breakpoint.</span></span>

> [!NOTE]
> <span data-ttu-id="b96d2-201">I punti di interruzione **non** vengono raggiunti durante l'avvio dell'app prima che il proxy di debug sia in esecuzione.</span><span class="sxs-lookup"><span data-stu-id="b96d2-201">Breakpoints are **not** hit during app startup before the debug proxy is running.</span></span> <span data-ttu-id="b96d2-202">Sono inclusi i punti di interruzione in `Program.Main` ( `Program.cs` ) e i punti di interruzione nei [ `OnInitialized{Async}` Metodi](xref:blazor/components/lifecycle#component-initialization-methods) dei componenti caricati dalla prima pagina richiesta dall'app.</span><span class="sxs-lookup"><span data-stu-id="b96d2-202">This includes breakpoints in `Program.Main` (`Program.cs`) and breakpoints in the [`OnInitialized{Async}` methods](xref:blazor/components/lifecycle#component-initialization-methods) of components that are loaded by the first page requested from the app.</span></span>

## <a name="debug-hosted-blazor-webassembly"></a><span data-ttu-id="b96d2-203">Debug ospitato Blazor WebAssembly</span><span class="sxs-lookup"><span data-stu-id="b96d2-203">Debug hosted Blazor WebAssembly</span></span>

1. <span data-ttu-id="b96d2-204">Aprire la **`Client`** cartella di progetto della cartella della soluzione ospitata Blazor in vs code.</span><span class="sxs-lookup"><span data-stu-id="b96d2-204">Open the **`Client`** project folder of the hosted Blazor solution folder in VS Code.</span></span>

1. <span data-ttu-id="b96d2-205">Se non è impostata alcuna configurazione di avvio per il progetto, viene visualizzata la notifica seguente.</span><span class="sxs-lookup"><span data-stu-id="b96d2-205">If there's no launch configuration set for the project, the following notification appears.</span></span> <span data-ttu-id="b96d2-206">Selezionare **Sì**.</span><span class="sxs-lookup"><span data-stu-id="b96d2-206">Select **Yes**.</span></span>

   > <span data-ttu-id="b96d2-207">Gli asset necessari per la compilazione e il debug non sono presenti in ' {nome applicazione}'.</span><span class="sxs-lookup"><span data-stu-id="b96d2-207">Required assets to build and debug are missing from '{APPLICATION NAME}'.</span></span> <span data-ttu-id="b96d2-208">e se si vuole aggiungerle.</span><span class="sxs-lookup"><span data-stu-id="b96d2-208">Add them?</span></span>

   <span data-ttu-id="b96d2-209">Per informazioni sulla configurazione di asset VS Code nella `.vscode` cartella, vedere la guida del sistema operativo **Linux** in <xref:blazor/tooling> .</span><span class="sxs-lookup"><span data-stu-id="b96d2-209">For information on configuring VS Code assets in the `.vscode` folder, see the **Linux** operating system guidance in <xref:blazor/tooling>.</span></span>

1. <span data-ttu-id="b96d2-210">Nel riquadro comandi nella parte superiore della finestra selezionare il progetto *Server* all'interno della soluzione ospitata.</span><span class="sxs-lookup"><span data-stu-id="b96d2-210">In the command palette at the top of the window, select the *Server* project within the hosted solution.</span></span>

<span data-ttu-id="b96d2-211">`launch.json`Viene generato un file con la configurazione di avvio per l'avvio del debugger.</span><span class="sxs-lookup"><span data-stu-id="b96d2-211">A `launch.json` file is generated with the launch configuration for launching the debugger.</span></span>

## <a name="attach-to-an-existing-debugging-session"></a><span data-ttu-id="b96d2-212">Connetti a una sessione di debug esistente</span><span class="sxs-lookup"><span data-stu-id="b96d2-212">Attach to an existing debugging session</span></span>

<span data-ttu-id="b96d2-213">Per connettersi a un'app in esecuzione Blazor , creare un `launch.json` file con la configurazione seguente:</span><span class="sxs-lookup"><span data-stu-id="b96d2-213">To attach to a running Blazor app, create a `launch.json` file with the following configuration:</span></span>

```json
{
  "type": "blazorwasm",
  "request": "attach",
  "name": "Attach to Existing Blazor WebAssembly Application"
}
```

> [!NOTE]
> <span data-ttu-id="b96d2-214">Il fissaggio a una sessione di debug è supportato solo per le app autonome.</span><span class="sxs-lookup"><span data-stu-id="b96d2-214">Attaching to a debugging session is only supported for standalone apps.</span></span> <span data-ttu-id="b96d2-215">Per usare il debug dello stack completo, è necessario avviare l'app da VS Code.</span><span class="sxs-lookup"><span data-stu-id="b96d2-215">To use full-stack debugging, you must launch the app from VS Code.</span></span>

## <a name="launch-configuration-options"></a><span data-ttu-id="b96d2-216">Opzioni di configurazione di avvio</span><span class="sxs-lookup"><span data-stu-id="b96d2-216">Launch configuration options</span></span>

<span data-ttu-id="b96d2-217">Le seguenti opzioni di configurazione di avvio sono supportate per il `blazorwasm` tipo di debug ( `.vscode/launch.json` ).</span><span class="sxs-lookup"><span data-stu-id="b96d2-217">The following launch configuration options are supported for the `blazorwasm` debug type (`.vscode/launch.json`).</span></span>

| <span data-ttu-id="b96d2-218">Opzione</span><span class="sxs-lookup"><span data-stu-id="b96d2-218">Option</span></span>    | <span data-ttu-id="b96d2-219">Descrizione</span><span class="sxs-lookup"><span data-stu-id="b96d2-219">Description</span></span> |
| --------- | ----------- |
| `request` | <span data-ttu-id="b96d2-220">Usare `launch` per avviare e alleghi una sessione di debug a un' Blazor WebAssembly app o `attach` per alleghi una sessione di debug a un'app già in esecuzione.</span><span class="sxs-lookup"><span data-stu-id="b96d2-220">Use `launch` to launch and attach a debugging session to a Blazor WebAssembly app or `attach` to attach a debugging session to an already-running app.</span></span> |
| `url`     | <span data-ttu-id="b96d2-221">URL da aprire nel browser durante il debug.</span><span class="sxs-lookup"><span data-stu-id="b96d2-221">The URL to open in the browser when debugging.</span></span> <span data-ttu-id="b96d2-222">Il valore predefinito è `https://localhost:5001`.</span><span class="sxs-lookup"><span data-stu-id="b96d2-222">Defaults to `https://localhost:5001`.</span></span> |
| `browser` | <span data-ttu-id="b96d2-223">Browser da avviare per la sessione di debug.</span><span class="sxs-lookup"><span data-stu-id="b96d2-223">The browser to launch for the debugging session.</span></span> <span data-ttu-id="b96d2-224">Impostare su `edge` o `chrome`.</span><span class="sxs-lookup"><span data-stu-id="b96d2-224">Set to `edge` or `chrome`.</span></span> <span data-ttu-id="b96d2-225">Il valore predefinito è `chrome`.</span><span class="sxs-lookup"><span data-stu-id="b96d2-225">Defaults to `chrome`.</span></span> |
| `trace`   | <span data-ttu-id="b96d2-226">Usato per generare log dal debugger JS.</span><span class="sxs-lookup"><span data-stu-id="b96d2-226">Used to generate logs from the JS debugger.</span></span> <span data-ttu-id="b96d2-227">Impostare su `true` per generare i log.</span><span class="sxs-lookup"><span data-stu-id="b96d2-227">Set to `true` to generate logs.</span></span> |
| `hosted`  | <span data-ttu-id="b96d2-228">Deve essere impostato su `true` se si avvia e si esegue il debug di un'app ospitata Blazor WebAssembly .</span><span class="sxs-lookup"><span data-stu-id="b96d2-228">Must be set to `true` if launching and debugging a hosted Blazor WebAssembly app.</span></span> |
| `webRoot` | <span data-ttu-id="b96d2-229">Specifica il percorso assoluto del server Web.</span><span class="sxs-lookup"><span data-stu-id="b96d2-229">Specifies the absolute path of the web server.</span></span> <span data-ttu-id="b96d2-230">Deve essere impostato se un'app viene gestita da una route secondaria.</span><span class="sxs-lookup"><span data-stu-id="b96d2-230">Should be set if an app is served from a sub-route.</span></span> |
| `timeout` | <span data-ttu-id="b96d2-231">Numero di millisecondi di attesa per il fissaggio della sessione di debug.</span><span class="sxs-lookup"><span data-stu-id="b96d2-231">The number of milliseconds to wait for the debugging session to attach.</span></span> <span data-ttu-id="b96d2-232">Il valore predefinito è 30.000 millisecondi (30 secondi).</span><span class="sxs-lookup"><span data-stu-id="b96d2-232">Defaults to 30,000 milliseconds (30 seconds).</span></span> |
| `program` | <span data-ttu-id="b96d2-233">Riferimento al file eseguibile per eseguire il server dell'app ospitata.</span><span class="sxs-lookup"><span data-stu-id="b96d2-233">A reference to the executable to run the server of the hosted app.</span></span> <span data-ttu-id="b96d2-234">Deve essere impostato se `hosted` è `true` .</span><span class="sxs-lookup"><span data-stu-id="b96d2-234">Must be set if `hosted` is `true`.</span></span> |
| `cwd`     | <span data-ttu-id="b96d2-235">Directory di lavoro in cui avviare l'app.</span><span class="sxs-lookup"><span data-stu-id="b96d2-235">The working directory to launch the app under.</span></span> <span data-ttu-id="b96d2-236">Deve essere impostato se `hosted` è `true` .</span><span class="sxs-lookup"><span data-stu-id="b96d2-236">Must be set if `hosted` is `true`.</span></span> |
| `env`     | <span data-ttu-id="b96d2-237">Variabili di ambiente da fornire al processo avviato.</span><span class="sxs-lookup"><span data-stu-id="b96d2-237">The environment variables to provide to the launched process.</span></span> <span data-ttu-id="b96d2-238">Applicabile solo se `hosted` è impostato su `true` .</span><span class="sxs-lookup"><span data-stu-id="b96d2-238">Only applicable if `hosted` is set to `true`.</span></span> |

## <a name="example-launch-configurations"></a><span data-ttu-id="b96d2-239">Configurazioni di avvio di esempio</span><span class="sxs-lookup"><span data-stu-id="b96d2-239">Example launch configurations</span></span>

### <a name="launch-and-debug-a-standalone-blazor-webassembly-app"></a><span data-ttu-id="b96d2-240">Avviare ed eseguire il debug di un'app autonoma Blazor WebAssembly</span><span class="sxs-lookup"><span data-stu-id="b96d2-240">Launch and debug a standalone Blazor WebAssembly app</span></span>

```json
{
  "type": "blazorwasm",
  "request": "launch",
  "name": "Launch and Debug"
}
```

### <a name="attach-to-a-running-app-at-a-specified-url"></a><span data-ttu-id="b96d2-241">Connettersi a un'app in esecuzione in un URL specificato</span><span class="sxs-lookup"><span data-stu-id="b96d2-241">Attach to a running app at a specified URL</span></span>

```json
{
  "type": "blazorwasm",
  "request": "attach",
  "name": "Attach and Debug",
  "url": "http://localhost:5000"
}
```

### <a name="launch-and-debug-a-hosted-blazor-webassembly-app-with-microsoft-edge"></a><span data-ttu-id="b96d2-242">Avviare ed eseguire il debug di un'app ospitata Blazor WebAssembly con Microsoft Edge</span><span class="sxs-lookup"><span data-stu-id="b96d2-242">Launch and debug a hosted Blazor WebAssembly app with Microsoft Edge</span></span>

<span data-ttu-id="b96d2-243">Per impostazione predefinita, la configurazione del browser è Google Chrome.</span><span class="sxs-lookup"><span data-stu-id="b96d2-243">Browser configuration defaults to Google Chrome.</span></span> <span data-ttu-id="b96d2-244">Quando si usa Microsoft Edge per il debug, impostare `browser` su `edge` .</span><span class="sxs-lookup"><span data-stu-id="b96d2-244">When using Microsoft Edge for debugging, set `browser` to `edge`.</span></span> <span data-ttu-id="b96d2-245">Per usare Google Chrome, non impostare l' `browser` opzione o impostare il valore dell'opzione su `chrome` .</span><span class="sxs-lookup"><span data-stu-id="b96d2-245">To use Google Chrome, either don't set the `browser` option or set the option's value to `chrome`.</span></span>

```json
{
  "name": "Launch and Debug Hosted Blazor WebAssembly App",
  "type": "blazorwasm",
  "request": "launch",
  "hosted": true,
  "program": "${workspaceFolder}/Server/bin/Debug/netcoreapp3.1/MyHostedApp.Server.dll",
  "cwd": "${workspaceFolder}/Server",
  "browser": "edge"
}
```

<span data-ttu-id="b96d2-246">Nell'esempio precedente, `MyHostedApp.Server.dll` è l'assembly dell'app *Server* .</span><span class="sxs-lookup"><span data-stu-id="b96d2-246">In the preceding example, `MyHostedApp.Server.dll` is the *Server* app's assembly.</span></span> <span data-ttu-id="b96d2-247">La `.vscode` cartella si trova nella cartella della soluzione accanto alle `Client` `Server` cartelle, e `Shared` .</span><span class="sxs-lookup"><span data-stu-id="b96d2-247">The `.vscode` folder is located in the solution's folder next to the `Client`, `Server`, and `Shared` folders.</span></span>

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="b96d2-248">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="b96d2-248">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

<span data-ttu-id="b96d2-249">Per eseguire il debug di un' Blazor WebAssembly app in Visual Studio per Mac:</span><span class="sxs-lookup"><span data-stu-id="b96d2-249">To debug a Blazor WebAssembly app in Visual Studio for Mac:</span></span>

1. <span data-ttu-id="b96d2-250">Creare una nuova app ospitata ASP.NET Core Blazor WebAssembly .</span><span class="sxs-lookup"><span data-stu-id="b96d2-250">Create a new ASP.NET Core hosted Blazor WebAssembly app.</span></span>
1. <span data-ttu-id="b96d2-251">Premere <kbd>&#8984;</kbd> + <kbd>&#8617;</kbd> per eseguire l'applicazione nel debugger.</span><span class="sxs-lookup"><span data-stu-id="b96d2-251">Press <kbd>&#8984;</kbd>+<kbd>&#8617;</kbd> to run the app in the debugger.</span></span>

   > [!NOTE]
   > <span data-ttu-id="b96d2-252">**Avvia senza eseguire debug** (<kbd>&#8997;</kbd> + <kbd>&#8984;</kbd> + <kbd>&#8617;</kbd>) non è supportato.</span><span class="sxs-lookup"><span data-stu-id="b96d2-252">**Start Without Debugging** (<kbd>&#8997;</kbd>+<kbd>&#8984;</kbd>+<kbd>&#8617;</kbd>) isn't supported.</span></span> <span data-ttu-id="b96d2-253">Quando l'app viene eseguita nella configurazione di debug, il sovraccarico del debug comporta sempre una riduzione delle prestazioni.</span><span class="sxs-lookup"><span data-stu-id="b96d2-253">When the app is run in Debug configuration, debugging overhead always results in a small performance reduction.</span></span>

   > [!IMPORTANT]
   > <span data-ttu-id="b96d2-254">Google Chrome o Microsoft Edge deve essere il browser selezionato per la sessione di debug.</span><span class="sxs-lookup"><span data-stu-id="b96d2-254">Google Chrome or Microsoft Edge must be the selected browser for the debugging session.</span></span>

1. <span data-ttu-id="b96d2-255">Nell' `*Client*` app, impostare un punto di interruzione nella `currentCount++;` riga in `Pages/Counter.razor` .</span><span class="sxs-lookup"><span data-stu-id="b96d2-255">In the `*Client*` app, set a breakpoint on the `currentCount++;` line in `Pages/Counter.razor`.</span></span>
1. <span data-ttu-id="b96d2-256">Nel browser passare alla `Counter` pagina e selezionare il pulsante **fare clic su me** per raggiungere il punto di interruzione:</span><span class="sxs-lookup"><span data-stu-id="b96d2-256">In the browser, navigate to `Counter` page and select the **Click me** button to hit the breakpoint:</span></span>
1. <span data-ttu-id="b96d2-257">In Visual Studio, controllare il valore del `currentCount` campo nella finestra **variabili locali** .</span><span class="sxs-lookup"><span data-stu-id="b96d2-257">In Visual Studio, inspect the value of the `currentCount` field in the **Locals** window.</span></span>
1. <span data-ttu-id="b96d2-258">Premere <kbd>&#8984;</kbd> + <kbd>&#8617;</kbd> per continuare l'esecuzione.</span><span class="sxs-lookup"><span data-stu-id="b96d2-258">Press <kbd>&#8984;</kbd>+<kbd>&#8617;</kbd> to continue execution.</span></span>

<span data-ttu-id="b96d2-259">Durante il debug di un' Blazor WebAssembly app, è anche possibile eseguire il debug del codice del server:</span><span class="sxs-lookup"><span data-stu-id="b96d2-259">While debugging a Blazor WebAssembly app, you can also debug server code:</span></span>

1. <span data-ttu-id="b96d2-260">Impostare un punto di interruzione nella `Pages/FetchData.razor` pagina in <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> .</span><span class="sxs-lookup"><span data-stu-id="b96d2-260">Set a breakpoint in the `Pages/FetchData.razor` page in <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A>.</span></span>
1. <span data-ttu-id="b96d2-261">Impostare un punto di interruzione in `WeatherForecastController` nel `Get` metodo di azione.</span><span class="sxs-lookup"><span data-stu-id="b96d2-261">Set a breakpoint in the `WeatherForecastController` in the `Get` action method.</span></span>
1. <span data-ttu-id="b96d2-262">Passare alla `Fetch Data` pagina per raggiungere il primo punto di interruzione nel `FetchData` componente immediatamente prima di eseguire una richiesta HTTP al server.</span><span class="sxs-lookup"><span data-stu-id="b96d2-262">Browse to the `Fetch Data` page to hit the first breakpoint in the `FetchData` component just before it issues an HTTP request to the server.</span></span>
1. <span data-ttu-id="b96d2-263">Premere <kbd>&#8984;</kbd> + <kbd>&#8617;</kbd> per continuare l'esecuzione e quindi raggiungere il punto di interruzione sul server nel `WeatherForecastController` .</span><span class="sxs-lookup"><span data-stu-id="b96d2-263">Press <kbd>&#8984;</kbd>+<kbd>&#8617;</kbd> to continue execution and then hit the breakpoint on the server in the `WeatherForecastController`.</span></span>
1. <span data-ttu-id="b96d2-264">Premere <kbd></kbd> + di nuovo&#8984;<kbd>&#8617;</kbd> per consentire l'esecuzione continua e visualizzare la tabella delle previsioni meteorologiche sottoposta a rendering nel browser.</span><span class="sxs-lookup"><span data-stu-id="b96d2-264">Press <kbd>&#8984;</kbd>+<kbd>&#8617;</kbd> again to let execution continue and see the weather forecast table rendered in the browser.</span></span>

> [!NOTE]
> <span data-ttu-id="b96d2-265">I punti di interruzione **non** vengono raggiunti durante l'avvio dell'app prima che il proxy di debug sia in esecuzione.</span><span class="sxs-lookup"><span data-stu-id="b96d2-265">Breakpoints are **not** hit during app startup before the debug proxy is running.</span></span> <span data-ttu-id="b96d2-266">Sono inclusi i punti di interruzione in `Program.Main` ( `Program.cs` ) e i punti di interruzione nei [ `OnInitialized{Async}` Metodi](xref:blazor/components/lifecycle#component-initialization-methods) dei componenti caricati dalla prima pagina richiesta dall'app.</span><span class="sxs-lookup"><span data-stu-id="b96d2-266">This includes breakpoints in `Program.Main` (`Program.cs`) and breakpoints in the [`OnInitialized{Async}` methods](xref:blazor/components/lifecycle#component-initialization-methods) of components that are loaded by the first page requested from the app.</span></span>

<span data-ttu-id="b96d2-267">Per ulteriori informazioni, vedere [debug con Visual Studio per Mac](/visualstudio/mac/debugging).</span><span class="sxs-lookup"><span data-stu-id="b96d2-267">For more information, see [Debugging with Visual Studio for Mac](/visualstudio/mac/debugging).</span></span>

---

## <a name="debug-in-the-browser"></a><span data-ttu-id="b96d2-268">Debug nel browser</span><span class="sxs-lookup"><span data-stu-id="b96d2-268">Debug in the browser</span></span>

<span data-ttu-id="b96d2-269">*Le indicazioni fornite in questa sezione si applicano a Google Chrome e Microsoft Edge in esecuzione su Windows.*</span><span class="sxs-lookup"><span data-stu-id="b96d2-269">*The guidance in this section applies to Google Chrome and Microsoft Edge running on Windows.*</span></span>

1. <span data-ttu-id="b96d2-270">Eseguire una build di debug dell'app nell'ambiente di sviluppo.</span><span class="sxs-lookup"><span data-stu-id="b96d2-270">Run a Debug build of the app in the Development environment.</span></span>

1. <span data-ttu-id="b96d2-271">Avviare un browser e passare all'URL dell'app, ad esempio `https://localhost:5001` .</span><span class="sxs-lookup"><span data-stu-id="b96d2-271">Launch a browser and navigate to the app's URL (for example, `https://localhost:5001`).</span></span>

1. <span data-ttu-id="b96d2-272">Nel browser provare ad avviare il debug remoto premendo <kbd>MAIUSC</kbd> + <kbd>ALT</kbd> + <kbd>d</kbd>.</span><span class="sxs-lookup"><span data-stu-id="b96d2-272">In the browser, attempt to commence remote debugging by pressing <kbd>Shift</kbd>+<kbd>Alt</kbd>+<kbd>d</kbd>.</span></span>

   <span data-ttu-id="b96d2-273">Il browser deve essere in esecuzione con il debug remoto abilitato, che non è il valore predefinito.</span><span class="sxs-lookup"><span data-stu-id="b96d2-273">The browser must be running with remote debugging enabled, which isn't the default.</span></span> <span data-ttu-id="b96d2-274">Se il debug remoto è disabilitato, viene eseguito il rendering della pagina di errore della **scheda del browser di cui è possibile eseguire** il debug con le istruzioni per avviare il browser con la porta di debug aperta.</span><span class="sxs-lookup"><span data-stu-id="b96d2-274">If remote debugging is disabled, an **Unable to find debuggable browser tab** error page is rendered with instructions for launching the browser with the debugging port open.</span></span> <span data-ttu-id="b96d2-275">Seguire le istruzioni per il browser, che consente di aprire una nuova finestra del browser.</span><span class="sxs-lookup"><span data-stu-id="b96d2-275">Follow the instructions for your browser, which opens a new browser window.</span></span> <span data-ttu-id="b96d2-276">Chiudere la finestra del browser precedente.</span><span class="sxs-lookup"><span data-stu-id="b96d2-276">Close the previous browser window.</span></span>

<!-- HOLD 
1. In the browser, attempt to commence remote debugging by pressing:

   * <kbd>Shift</kbd>+<kbd>Alt</kbd>+<kbd>d</kbd> on Windows.
   * <kbd>Shift</kbd>+<kbd>&#8984;</kbd>+<kbd>d</kbd> on macOS.

   The browser must be running with remote debugging enabled, which isn't the default. If remote debugging is disabled, an **Unable to find debuggable browser tab** error page is rendered with instructions for launching the browser with the debugging port open. Follow the instructions for your browser, which opens a new browser window. Close the previous browser window.
-->

1. <span data-ttu-id="b96d2-277">Quando il browser è in esecuzione con il debug remoto abilitato, il tasto di scelta rapida di debug nel passaggio precedente apre una nuova scheda del debugger.</span><span class="sxs-lookup"><span data-stu-id="b96d2-277">Once the browser is running with remote debugging enabled, the debugging keyboard shortcut in the previous step opens a new debugger tab.</span></span>

1. <span data-ttu-id="b96d2-278">Dopo qualche istante, la scheda **origini** Mostra un elenco degli assembly .NET dell'app all'interno del `file://` nodo.</span><span class="sxs-lookup"><span data-stu-id="b96d2-278">After a moment, the **Sources** tab shows a list of the app's .NET assemblies within the `file://` node.</span></span>

1. <span data-ttu-id="b96d2-279">Nel codice componente ( `.razor` file) e nei file di codice C# ( `.cs` ), i punti di interruzione impostati vengono raggiunti durante l'esecuzione del codice.</span><span class="sxs-lookup"><span data-stu-id="b96d2-279">In component code (`.razor` files) and C# code files (`.cs`), breakpoints that you set are hit when code executes.</span></span> <span data-ttu-id="b96d2-280">Quando viene raggiunto un punto di interruzione, l'esecuzione<kbd></kbd>del codice in un singolo passaggio (F10<kbd>) viene</kbd>eseguito normalmente.</span><span class="sxs-lookup"><span data-stu-id="b96d2-280">After a breakpoint is hit, single-step (<kbd>F10</kbd>) through the code or resume (<kbd>F8</kbd>) code execution normally.</span></span>

<span data-ttu-id="b96d2-281">Blazor fornisce un proxy di debug che implementa il [protocollo devtools di Chrome](https://chromedevtools.github.io/devtools-protocol/) e potenzia il protocollo con. Informazioni specifiche del NET.</span><span class="sxs-lookup"><span data-stu-id="b96d2-281">Blazor provides a debugging proxy that implements the [Chrome DevTools Protocol](https://chromedevtools.github.io/devtools-protocol/) and augments the protocol with .NET-specific information.</span></span> <span data-ttu-id="b96d2-282">Quando si preme il tasto di scelta rapida Blazor per il debug, punta il devtools di Chrome sul proxy.</span><span class="sxs-lookup"><span data-stu-id="b96d2-282">When debugging keyboard shortcut is pressed, Blazor points the Chrome DevTools at the proxy.</span></span> <span data-ttu-id="b96d2-283">Il proxy si connette alla finestra del browser che si sta tentando di eseguire il debug, quindi è necessario abilitare il debug remoto.</span><span class="sxs-lookup"><span data-stu-id="b96d2-283">The proxy connects to the browser window you're seeking to debug (hence the need to enable remote debugging).</span></span>

## <a name="browser-source-maps"></a><span data-ttu-id="b96d2-284">Mappe di origine del browser</span><span class="sxs-lookup"><span data-stu-id="b96d2-284">Browser source maps</span></span>

<span data-ttu-id="b96d2-285">Le mappe di origine del browser consentono al browser di eseguire il mapping dei file compilati ai file di origine originali e vengono comunemente usati per il debug sul lato client.</span><span class="sxs-lookup"><span data-stu-id="b96d2-285">Browser source maps allow the browser to map compiled files back to their original source files and are commonly used for client-side debugging.</span></span> <span data-ttu-id="b96d2-286">Tuttavia, Blazor attualmente non esegue il mapping di C# direttamente a JavaScript/WASM.</span><span class="sxs-lookup"><span data-stu-id="b96d2-286">However, Blazor doesn't currently map C# directly to JavaScript/WASM.</span></span> <span data-ttu-id="b96d2-287">Al contrario, Blazor l'interpretazione il nel browser, quindi le mappe di origine non sono rilevanti.</span><span class="sxs-lookup"><span data-stu-id="b96d2-287">Instead, Blazor does IL interpretation within the browser, so source maps aren't relevant.</span></span>

## <a name="firewall-configuration"></a><span data-ttu-id="b96d2-288">Configurazione del firewall</span><span class="sxs-lookup"><span data-stu-id="b96d2-288">Firewall configuration</span></span>

<span data-ttu-id="b96d2-289">Se un firewall blocca la comunicazione con il proxy di debug, creare una regola di eccezione del firewall che consenta la comunicazione tra il browser e il `NodeJS` processo.</span><span class="sxs-lookup"><span data-stu-id="b96d2-289">If a firewall blocks communication with the debug proxy, create a firewall exception rule that permits communication between the browser and the `NodeJS` process.</span></span>

> [!WARNING]
> <span data-ttu-id="b96d2-290">La modifica di una configurazione del firewall deve essere eseguita con cautela per evitare la creazione di vulnerablities di sicurezza.</span><span class="sxs-lookup"><span data-stu-id="b96d2-290">Modification of a firewall configuration must be made with care to avoid creating security vulnerablities.</span></span> <span data-ttu-id="b96d2-291">Applicare con attenzione le linee guida per la sicurezza, attenersi alle procedure di sicurezza consigliate e rispettare gli avvisi emessi dal produttore del firewall.</span><span class="sxs-lookup"><span data-stu-id="b96d2-291">Carefully apply security guidance, follow best security practices, and respect warnings issued by the firewall's manufacturer.</span></span>
>
> <span data-ttu-id="b96d2-292">Consentire la comunicazione aperta con il `NodeJS` processo:</span><span class="sxs-lookup"><span data-stu-id="b96d2-292">Permitting open communication with the `NodeJS` process:</span></span>
>
> * <span data-ttu-id="b96d2-293">Apre il server del nodo a qualsiasi connessione, a seconda delle funzionalità e della configurazione del firewall.</span><span class="sxs-lookup"><span data-stu-id="b96d2-293">Opens up the Node server to any connection, depending on the firewall's capabilities and configuration.</span></span>
> * <span data-ttu-id="b96d2-294">Potrebbe essere rischioso a seconda della rete.</span><span class="sxs-lookup"><span data-stu-id="b96d2-294">Might be risky depending on your network.</span></span>
> * <span data-ttu-id="b96d2-295">**È consigliato solo nei computer di sviluppo.**</span><span class="sxs-lookup"><span data-stu-id="b96d2-295">**Is only recommended on developer machines.**</span></span>
>
> <span data-ttu-id="b96d2-296">Se possibile, consentire la comunicazione aperta solo con il `NodeJS` processo **su reti private o attendibili**.</span><span class="sxs-lookup"><span data-stu-id="b96d2-296">If possible, only allow open communication with the `NodeJS` process **on trusted or private networks**.</span></span>

<span data-ttu-id="b96d2-297">Per informazioni aggiuntive sulla configurazione di [Windows Firewall](/windows/security/threat-protection/windows-firewall/windows-firewall-with-advanced-security) , vedere [creare una regola di servizio o programma in ingresso](/windows/security/threat-protection/windows-firewall/create-an-inbound-program-or-service-rule).</span><span class="sxs-lookup"><span data-stu-id="b96d2-297">For [Windows Firewall](/windows/security/threat-protection/windows-firewall/windows-firewall-with-advanced-security) configuration guidance, see [Create an Inbound Program or Service Rule](/windows/security/threat-protection/windows-firewall/create-an-inbound-program-or-service-rule).</span></span> <span data-ttu-id="b96d2-298">Per ulteriori informazioni, vedere [Windows Defender Firewall con protezione avanzata](/windows/security/threat-protection/windows-firewall/windows-firewall-with-advanced-security) e articoli correlati nel set di documentazione Windows Firewall.</span><span class="sxs-lookup"><span data-stu-id="b96d2-298">For more information, see [Windows Defender Firewall with Advanced Security](/windows/security/threat-protection/windows-firewall/windows-firewall-with-advanced-security) and related articles in the Windows Firewall documentation set.</span></span>

## <a name="troubleshoot"></a><span data-ttu-id="b96d2-299">Risolvere problemi</span><span class="sxs-lookup"><span data-stu-id="b96d2-299">Troubleshoot</span></span>

<span data-ttu-id="b96d2-300">Se si verificano errori, è possibile che vengano visualizzati i suggerimenti seguenti:</span><span class="sxs-lookup"><span data-stu-id="b96d2-300">If you're running into errors, the following tips may help:</span></span>

* <span data-ttu-id="b96d2-301">Nella scheda **debugger** aprire gli strumenti di sviluppo nel browser.</span><span class="sxs-lookup"><span data-stu-id="b96d2-301">In the **Debugger** tab, open the developer tools in your browser.</span></span> <span data-ttu-id="b96d2-302">Nella console eseguire `localStorage.clear()` per rimuovere tutti i punti di interruzione.</span><span class="sxs-lookup"><span data-stu-id="b96d2-302">In the console, execute `localStorage.clear()` to remove any breakpoints.</span></span>
* <span data-ttu-id="b96d2-303">Verificare di aver installato e considerato attendibile il certificato di sviluppo ASP.NET Core HTTPS.</span><span class="sxs-lookup"><span data-stu-id="b96d2-303">Confirm that you've installed and trusted the ASP.NET Core HTTPS development certificate.</span></span> <span data-ttu-id="b96d2-304">Per altre informazioni, vedere <xref:security/enforcing-ssl#troubleshoot-certificate-problems>.</span><span class="sxs-lookup"><span data-stu-id="b96d2-304">For more information, see <xref:security/enforcing-ssl#troubleshoot-certificate-problems>.</span></span>
* <span data-ttu-id="b96d2-305">Visual Studio richiede l'opzione **Abilita debug JavaScript per ASP.NET (Chrome, Edge e IE)** in **strumenti**  >  **Opzioni**  >  **debug**  >  **generale**.</span><span class="sxs-lookup"><span data-stu-id="b96d2-305">Visual Studio requires the **Enable JavaScript debugging for ASP.NET (Chrome, Edge and IE)** option in **Tools** > **Options** > **Debugging** > **General**.</span></span> <span data-ttu-id="b96d2-306">Questa è l'impostazione predefinita per Visual Studio.</span><span class="sxs-lookup"><span data-stu-id="b96d2-306">This is the default setting for Visual Studio.</span></span> <span data-ttu-id="b96d2-307">Se il debug non funziona, verificare che sia selezionata l'opzione.</span><span class="sxs-lookup"><span data-stu-id="b96d2-307">If debugging isn't working, confirm that the option is selected.</span></span>
* <span data-ttu-id="b96d2-308">Se l'ambiente usa un proxy HTTP, assicurarsi che `localhost` sia incluso nelle impostazioni di bypass del proxy.</span><span class="sxs-lookup"><span data-stu-id="b96d2-308">If your environment uses an HTTP proxy, make sure that `localhost` is included in the proxy bypass settings.</span></span> <span data-ttu-id="b96d2-309">Questa operazione può essere eseguita impostando la `NO_PROXY` variabile di ambiente in uno dei seguenti valori:</span><span class="sxs-lookup"><span data-stu-id="b96d2-309">This can be done by setting the `NO_PROXY` environment variable in either:</span></span>
  * <span data-ttu-id="b96d2-310">`launchSettings.json`File per il progetto.</span><span class="sxs-lookup"><span data-stu-id="b96d2-310">The `launchSettings.json` file for the project.</span></span>
  * <span data-ttu-id="b96d2-311">A livello di utente o di variabili di ambiente di sistema per applicarlo a tutte le app.</span><span class="sxs-lookup"><span data-stu-id="b96d2-311">At the user or system environment variables level for it to apply to all apps.</span></span> <span data-ttu-id="b96d2-312">Quando si usa una variabile di ambiente, riavviare Visual Studio per rendere effettive le modifiche.</span><span class="sxs-lookup"><span data-stu-id="b96d2-312">When using an environment variable, restart Visual Studio for the change to take effect.</span></span>
* <span data-ttu-id="b96d2-313">Assicurarsi che i firewall o i proxy non blocchino le comunicazioni con il proxy di debug ( `NodeJS` processo).</span><span class="sxs-lookup"><span data-stu-id="b96d2-313">Ensure that firewalls or proxies don't block communication with the debug proxy (`NodeJS` process).</span></span> <span data-ttu-id="b96d2-314">Per ulteriori informazioni, vedere la sezione relativa alla [configurazione del firewall](#firewall-configuration) .</span><span class="sxs-lookup"><span data-stu-id="b96d2-314">For more information, see the [Firewall configuration](#firewall-configuration) section.</span></span>

### <a name="breakpoints-in-oninitializedasync-not-hit"></a><span data-ttu-id="b96d2-315">Punti di interruzione in `OnInitialized{Async}` non riscontri</span><span class="sxs-lookup"><span data-stu-id="b96d2-315">Breakpoints in `OnInitialized{Async}` not hit</span></span>

<span data-ttu-id="b96d2-316">Il Blazor proxy di debug del Framework richiede un breve intervallo di tempo, quindi i punti di interruzione nel [ `OnInitialized{Async}` metodo del ciclo](xref:blazor/components/lifecycle#component-initialization-methods) di vita potrebbero non essere raggiunti.</span><span class="sxs-lookup"><span data-stu-id="b96d2-316">The Blazor framework's debugging proxy takes a short time to launch, so breakpoints in the [`OnInitialized{Async}` lifecycle method](xref:blazor/components/lifecycle#component-initialization-methods) might not be hit.</span></span> <span data-ttu-id="b96d2-317">Si consiglia di aggiungere un ritardo all'inizio del corpo del metodo per consentire al proxy di debug di avviarsi prima che venga raggiunto il punto di interruzione.</span><span class="sxs-lookup"><span data-stu-id="b96d2-317">We recommend adding a delay at the start of the method body to give the debug proxy some time to launch before the breakpoint is hit.</span></span> <span data-ttu-id="b96d2-318">È possibile includere il ritardo in base a una [ `if` direttiva del compilatore](/dotnet/csharp/language-reference/preprocessor-directives/preprocessor-if) per assicurarsi che il ritardo non sia presente per una build di rilascio dell'app.</span><span class="sxs-lookup"><span data-stu-id="b96d2-318">You can include the delay based on an [`if` compiler directive](/dotnet/csharp/language-reference/preprocessor-directives/preprocessor-if) to ensure that the delay isn't present for a release build of the app.</span></span>

<span data-ttu-id="b96d2-319"><xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitialized%2A>:</span><span class="sxs-lookup"><span data-stu-id="b96d2-319"><xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitialized%2A>:</span></span>

```csharp
protected override void OnInitialized()
{
#if DEBUG
    Thread.Sleep(10000)
#endif

    ...
}
```

<span data-ttu-id="b96d2-320"><xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A>:</span><span class="sxs-lookup"><span data-stu-id="b96d2-320"><xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A>:</span></span>

```csharp
protected override async Task OnInitializedAsync()
{
#if DEBUG
    await Task.Delay(10000)
#endif

    ...
}
```

### <a name="visual-studio-windows-timeout"></a><span data-ttu-id="b96d2-321">Timeout di Visual Studio (Windows)</span><span class="sxs-lookup"><span data-stu-id="b96d2-321">Visual Studio (Windows) timeout</span></span>

<span data-ttu-id="b96d2-322">Se Visual Studio genera un'eccezione che non è stato possibile avviare l'adattatore di debug, indicando che è stato raggiunto il timeout, è possibile modificare il timeout con un'impostazione del registro di sistema:</span><span class="sxs-lookup"><span data-stu-id="b96d2-322">If Visual Studio throws an exception that the debug adapter failed to launch mentioning that the timeout was reached, you can adjust the timeout with a Registry setting:</span></span>

```console
VsRegEdit.exe set "<VSInstallFolder>" HKCU JSDebugger\Options\Debugging "BlazorTimeoutInMilliseconds" dword {TIMEOUT}
```

<span data-ttu-id="b96d2-323">Il `{TIMEOUT}` segnaposto nel comando precedente è in millisecondi.</span><span class="sxs-lookup"><span data-stu-id="b96d2-323">The `{TIMEOUT}` placeholder in the preceding command is in milliseconds.</span></span> <span data-ttu-id="b96d2-324">Ad esempio, un minuto viene assegnato come `60000` .</span><span class="sxs-lookup"><span data-stu-id="b96d2-324">For example, one minute is assigned as `60000`.</span></span>
