---
title: Strumenti per ASP.NET Core Blazor
author: guardrex
description: Informazioni sugli strumenti disponibili per la compilazione di Blazor app.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 02/11/2021
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
uid: blazor/tooling
zone_pivot_groups: operating-systems
ms.openlocfilehash: 19270bb74326dccfee9466b7c1fa61daeab805a2
ms.sourcegitcommit: 1436bd4d70937d6ec3140da56d96caab33c4320b
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/06/2021
ms.locfileid: "102394460"
---
# <a name="tooling-for-aspnet-core-blazor"></a><span data-ttu-id="59f18-103">Strumenti per ASP.NET Core Blazor</span><span class="sxs-lookup"><span data-stu-id="59f18-103">Tooling for ASP.NET Core Blazor</span></span>

::: zone pivot="windows"

1. <span data-ttu-id="59f18-104">Installare la versione più recente di [Visual Studio 2019](https://visualstudio.microsoft.com/downloads/) con il carico di lavoro **sviluppo di ASP.NET e Web** .</span><span class="sxs-lookup"><span data-stu-id="59f18-104">Install the latest version of [Visual Studio 2019](https://visualstudio.microsoft.com/downloads/) with the **ASP.NET and web development** workload.</span></span>

1. <span data-ttu-id="59f18-105">Creare un nuovo progetto.</span><span class="sxs-lookup"><span data-stu-id="59f18-105">Create a new project.</span></span>

1. <span data-ttu-id="59f18-106">Selezionare **Blazor app**.</span><span class="sxs-lookup"><span data-stu-id="59f18-106">Select **Blazor App**.</span></span> <span data-ttu-id="59f18-107">Selezionare **Avanti**.</span><span class="sxs-lookup"><span data-stu-id="59f18-107">Select **Next**.</span></span>

1. <span data-ttu-id="59f18-108">Specificare il nome di un progetto nel campo **Nome progetto** oppure accettare il nome predefinito.</span><span class="sxs-lookup"><span data-stu-id="59f18-108">Provide a project name in the **Project name** field or accept the default project name.</span></span> <span data-ttu-id="59f18-109">Confermare che la voce relativa al **percorso** sia corretta o specificare un percorso per il progetto.</span><span class="sxs-lookup"><span data-stu-id="59f18-109">Confirm the **Location** entry is correct or provide a location for the project.</span></span> <span data-ttu-id="59f18-110">Selezionare **Crea**.</span><span class="sxs-lookup"><span data-stu-id="59f18-110">Select **Create**.</span></span>

1. <span data-ttu-id="59f18-111">Per un' Blazor WebAssembly esperienza, scegliere il modello **Blazor WebAssembly app** .</span><span class="sxs-lookup"><span data-stu-id="59f18-111">For a Blazor WebAssembly experience, choose the **Blazor WebAssembly App** template.</span></span> <span data-ttu-id="59f18-112">Per un' Blazor Server esperienza, scegliere il modello **Blazor Server app** .</span><span class="sxs-lookup"><span data-stu-id="59f18-112">For a Blazor Server experience, choose the **Blazor Server App** template.</span></span> <span data-ttu-id="59f18-113">Selezionare **Crea**.</span><span class="sxs-lookup"><span data-stu-id="59f18-113">Select **Create**.</span></span>

   <span data-ttu-id="59f18-114">Per un'esperienza ospitata Blazor WebAssembly , selezionare la casella di controllo **ASP.NET Core Hosted** .</span><span class="sxs-lookup"><span data-stu-id="59f18-114">For a hosted Blazor WebAssembly experience, select the **ASP.NET Core hosted** check box.</span></span>

   <span data-ttu-id="59f18-115">Per informazioni sui due Blazor modelli di hosting *Blazor WebAssembly* (standalone e Hosted) e *Blazor Server* , vedere <xref:blazor/hosting-models> .</span><span class="sxs-lookup"><span data-stu-id="59f18-115">For information on the two Blazor hosting models, *Blazor WebAssembly* (standalone and hosted) and *Blazor Server*, see <xref:blazor/hosting-models>.</span></span>

1. <span data-ttu-id="59f18-116">Premere <kbd>CTRL</kbd> + <kbd>F5</kbd> per eseguire l'app.</span><span class="sxs-lookup"><span data-stu-id="59f18-116">Press <kbd>Ctrl</kbd>+<kbd>F5</kbd> to run the app.</span></span>

<span data-ttu-id="59f18-117">Per ulteriori informazioni su come considerare attendibile il certificato di sviluppo HTTPS ASP.NET Core, vedere <xref:security/enforcing-ssl#trust-the-aspnet-core-https-development-certificate-on-windows-and-macos> .</span><span class="sxs-lookup"><span data-stu-id="59f18-117">For more information on trusting the ASP.NET Core HTTPS development certificate, see <xref:security/enforcing-ssl#trust-the-aspnet-core-https-development-certificate-on-windows-and-macos>.</span></span>

<span data-ttu-id="59f18-118">Quando si esegue un'app ospitata Blazor WebAssembly , eseguire l'app dal progetto della soluzione **`Server`** .</span><span class="sxs-lookup"><span data-stu-id="59f18-118">When executing a hosted Blazor WebAssembly app, run the app from the solution's **`Server`** project.</span></span>

::: zone-end

::: zone pivot="linux"

1. <span data-ttu-id="59f18-119">Installare la versione più recente del [.NET Core SDK](https://dotnet.microsoft.com/download).</span><span class="sxs-lookup"><span data-stu-id="59f18-119">Install the latest version of the [.NET Core SDK](https://dotnet.microsoft.com/download).</span></span> <span data-ttu-id="59f18-120">Se in precedenza è stato installato l'SDK, è possibile determinare la versione installata eseguendo il comando seguente in una shell dei comandi:</span><span class="sxs-lookup"><span data-stu-id="59f18-120">If you previously installed the SDK, you can determine your installed version by executing the following command in a command shell:</span></span>

   ```dotnetcli
   dotnet --version
   ```

1. <span data-ttu-id="59f18-121">Installare la versione più recente di [Visual Studio Code](https://code.visualstudio.com).</span><span class="sxs-lookup"><span data-stu-id="59f18-121">Install the latest version of [Visual Studio Code](https://code.visualstudio.com).</span></span>

1. <span data-ttu-id="59f18-122">Installare la versione più recente [di C# per Visual Studio Code estensione](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp).</span><span class="sxs-lookup"><span data-stu-id="59f18-122">Install the latest [C# for Visual Studio Code extension](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp).</span></span>

1. <span data-ttu-id="59f18-123">Per un' Blazor WebAssembly esperienza, eseguire il comando seguente in una shell dei comandi:</span><span class="sxs-lookup"><span data-stu-id="59f18-123">For a Blazor WebAssembly experience, execute the following command in a command shell:</span></span>

   ```dotnetcli
   dotnet new blazorwasm -o WebApplication1
   ```

   <span data-ttu-id="59f18-124">Per un'esperienza ospitata Blazor WebAssembly , aggiungere l'opzione Hosted ( `-ho` o `--hosted` ) al comando:</span><span class="sxs-lookup"><span data-stu-id="59f18-124">For a hosted Blazor WebAssembly experience, add the hosted option (`-ho` or `--hosted`) option to the command:</span></span>

   ```dotnetcli
   dotnet new blazorwasm -o WebApplication1 -ho
   ```

   <span data-ttu-id="59f18-125">Per un' Blazor Server esperienza, eseguire il comando seguente in una shell dei comandi:</span><span class="sxs-lookup"><span data-stu-id="59f18-125">For a Blazor Server experience, execute the following command in a command shell:</span></span>

   ```dotnetcli
   dotnet new blazorserver -o WebApplication1
   ```

   <span data-ttu-id="59f18-126">Per informazioni sui due Blazor modelli di hosting *Blazor WebAssembly* (standalone e Hosted) e *Blazor Server* , vedere <xref:blazor/hosting-models> .</span><span class="sxs-lookup"><span data-stu-id="59f18-126">For information on the two Blazor hosting models, *Blazor WebAssembly* (standalone and hosted) and *Blazor Server*, see <xref:blazor/hosting-models>.</span></span>

1. <span data-ttu-id="59f18-127">Aprire la cartella `WebApplication1` in Visual Studio Code.</span><span class="sxs-lookup"><span data-stu-id="59f18-127">Open the `WebApplication1` folder in Visual Studio Code.</span></span>

1. <span data-ttu-id="59f18-128">Le richieste dell'IDE aggiungono risorse per compilare ed eseguire il debug del progetto.</span><span class="sxs-lookup"><span data-stu-id="59f18-128">The IDE requests that you add assets to build and debug the project.</span></span> <span data-ttu-id="59f18-129">Selezionare **Sì**.</span><span class="sxs-lookup"><span data-stu-id="59f18-129">Select **Yes**.</span></span>

   <span data-ttu-id="59f18-130">**Blazor WebAssemblyAvvio e configurazione attività ospitate**</span><span class="sxs-lookup"><span data-stu-id="59f18-130">**Hosted Blazor WebAssembly launch and task configuration**</span></span>

   <span data-ttu-id="59f18-131">Per le Blazor WebAssembly soluzioni ospitate, aggiungere (o spostare) la `.vscode` cartella con i `launch.json` file e nella `tasks.json` cartella padre della soluzione, ovvero la cartella che contiene i nomi tipici delle cartelle di progetto `Client` , `Server` e `Shared` .</span><span class="sxs-lookup"><span data-stu-id="59f18-131">For hosted Blazor WebAssembly solutions, add (or move) the `.vscode` folder with `launch.json` and `tasks.json` files to the solution's parent folder, which is the folder that contains the typical project folder names of `Client`, `Server`, and `Shared`.</span></span> <span data-ttu-id="59f18-132">Aggiornare o verificare che la configurazione nei `launch.json` file e `tasks.json` esegua un'app ospitata Blazor WebAssembly dal **`Server`** progetto.</span><span class="sxs-lookup"><span data-stu-id="59f18-132">Update or confirm that the configuration in the `launch.json` and `tasks.json` files execute a hosted Blazor WebAssembly app from the **`Server`** project.</span></span>

   <span data-ttu-id="59f18-133">**`.vscode/launch.json`** ( `launch` configurazione):</span><span class="sxs-lookup"><span data-stu-id="59f18-133">**`.vscode/launch.json`** (`launch` configuration):</span></span>

   ```json
   ...
   "cwd": "${workspaceFolder}/{SERVER APP FOLDER}",
   ...
   ```

   <span data-ttu-id="59f18-134">Nella configurazione precedente per la directory di lavoro corrente ( `cwd` ), il `{SERVER APP FOLDER}` segnaposto è la **`Server`** cartella del progetto, in genere " `Server` ".</span><span class="sxs-lookup"><span data-stu-id="59f18-134">In the preceding configuration for the current working directory (`cwd`), the `{SERVER APP FOLDER}` placeholder is the **`Server`** project's folder, typically "`Server`".</span></span>

   <span data-ttu-id="59f18-135">Se viene usato Microsoft Edge e Google Chrome non è installato nel sistema, aggiungere una proprietà aggiuntiva di `"browser": "edge"` alla configurazione.</span><span class="sxs-lookup"><span data-stu-id="59f18-135">If Microsoft Edge is used and Google Chrome isn't installed on the system, add an additional property of `"browser": "edge"` to the configuration.</span></span>

   <span data-ttu-id="59f18-136">Esempio per una cartella di progetto di `Server` e che genera Microsoft Edge come browser per le esecuzioni di debug anziché il browser predefinito Google Chrome:</span><span class="sxs-lookup"><span data-stu-id="59f18-136">Example for a project folder of `Server` and that spawns Microsoft Edge as the browser for debug runs instead of the default browser Google Chrome:</span></span>

   ```json
   ...
   "cwd": "${workspaceFolder}/Server",
   "browser": "edge"
   ...
   ```

   <span data-ttu-id="59f18-137">**`.vscode/tasks.json`**(argomenti del [ `dotnet` comando](/dotnet/core/tools/dotnet) ):</span><span class="sxs-lookup"><span data-stu-id="59f18-137">**`.vscode/tasks.json`** ([`dotnet` command](/dotnet/core/tools/dotnet) arguments):</span></span>

   ```json
   ...
   "${workspaceFolder}/{SERVER APP FOLDER}/{PROJECT NAME}.csproj",
   ...
   ```

   <span data-ttu-id="59f18-138">Nell'argomento precedente:</span><span class="sxs-lookup"><span data-stu-id="59f18-138">In the preceding argument:</span></span>

   * <span data-ttu-id="59f18-139">Il `{SERVER APP FOLDER}` segnaposto è la **`Server`** cartella del progetto, in genere " `Server` ".</span><span class="sxs-lookup"><span data-stu-id="59f18-139">The `{SERVER APP FOLDER}` placeholder is the **`Server`** project's folder, typically "`Server`".</span></span>
   * <span data-ttu-id="59f18-140">Il `{PROJECT NAME}` segnaposto è il nome dell'app, in genere in base al nome della soluzione seguito da " `.Server` " in un'app generata dal [ Blazor modello di progetto](xref:blazor/project-structure).</span><span class="sxs-lookup"><span data-stu-id="59f18-140">The `{PROJECT NAME}` placeholder is the app's name, typically based on the solution's name followed by "`.Server`" in an app generated from the [Blazor project template](xref:blazor/project-structure).</span></span>

   <span data-ttu-id="59f18-141">Nell'esempio seguente dell' [esercitazione per l'uso di SignalR con un' Blazor WebAssembly app](xref:tutorials/signalr-blazor) viene usato il nome della cartella del progetto `Server` e il nome di un progetto `BlazorWebAssemblySignalRApp.Server` :</span><span class="sxs-lookup"><span data-stu-id="59f18-141">The following example from the [tutorial for using SignalR with a Blazor WebAssembly app](xref:tutorials/signalr-blazor) uses a project folder name of `Server` and a project name of `BlazorWebAssemblySignalRApp.Server`:</span></span>

   ```json
   ...
   "args": [
     "build",
       "${workspaceFolder}/Server/BlazorWebAssemblySignalRApp.Server.csproj",
       "/property:GenerateFullPaths=true",
       "/consoleloggerparameters:NoSummary"
   ],
   ...
   ```

1. <span data-ttu-id="59f18-142">Premere <kbd>CTRL</kbd> + <kbd>F5</kbd> per eseguire l'app.</span><span class="sxs-lookup"><span data-stu-id="59f18-142">Press <kbd>Ctrl</kbd>+<kbd>F5</kbd> to run the app.</span></span>

## <a name="trust-a-development-certificate"></a><span data-ttu-id="59f18-143">Attendibilità di un certificato di sviluppo</span><span class="sxs-lookup"><span data-stu-id="59f18-143">Trust a development certificate</span></span>

<span data-ttu-id="59f18-144">Non esiste un modo centralizzato per considerare attendibile un certificato in Linux.</span><span class="sxs-lookup"><span data-stu-id="59f18-144">There's no centralized way to trust a certificate on Linux.</span></span> <span data-ttu-id="59f18-145">Viene in genere adottato uno degli approcci seguenti:</span><span class="sxs-lookup"><span data-stu-id="59f18-145">Typically, one of the following approaches is adopted:</span></span>

* <span data-ttu-id="59f18-146">Escludere l'URL dell'app nell'elenco di esclusione del browser.</span><span class="sxs-lookup"><span data-stu-id="59f18-146">Exclude the app's URL in browser's exclude list.</span></span>
* <span data-ttu-id="59f18-147">Considerare attendibili tutti i certificati autofirmati per `localhost` .</span><span class="sxs-lookup"><span data-stu-id="59f18-147">Trust all self-signed certificates for `localhost`.</span></span>
* <span data-ttu-id="59f18-148">Aggiungere il certificato all'elenco dei certificati attendibili nel browser.</span><span class="sxs-lookup"><span data-stu-id="59f18-148">Add the certificate to the list of trusted certificates in the browser.</span></span>

<span data-ttu-id="59f18-149">Per ulteriori informazioni, vedere le indicazioni fornite dal produttore del browser e dalla distribuzione di Linux.</span><span class="sxs-lookup"><span data-stu-id="59f18-149">For more information, see the guidance provided by your browser manufacturer and Linux distribution.</span></span>

::: zone-end

::: zone pivot="macos"

1. <span data-ttu-id="59f18-150">Installare [Visual Studio per Mac](https://visualstudio.microsoft.com/vs/mac/).</span><span class="sxs-lookup"><span data-stu-id="59f18-150">Install [Visual Studio for Mac](https://visualstudio.microsoft.com/vs/mac/).</span></span>

1. <span data-ttu-id="59f18-151">Selezionare **file**  >  **nuova soluzione** o creare un **nuovo** progetto dalla **finestra Start**.</span><span class="sxs-lookup"><span data-stu-id="59f18-151">Select **File** > **New Solution** or create a **New** project from the **Start Window**.</span></span>

1. <span data-ttu-id="59f18-152">Nella barra laterale selezionare app **Web e console**  >  .</span><span class="sxs-lookup"><span data-stu-id="59f18-152">In the sidebar, select **Web and Console** > **App**.</span></span>

   <span data-ttu-id="59f18-153">Per un' Blazor WebAssembly esperienza, scegliere il modello **Blazor WebAssembly app** .</span><span class="sxs-lookup"><span data-stu-id="59f18-153">For a Blazor WebAssembly experience, choose the **Blazor WebAssembly App** template.</span></span> <span data-ttu-id="59f18-154">Per un' Blazor Server esperienza, scegliere il modello **Blazor Server app** .</span><span class="sxs-lookup"><span data-stu-id="59f18-154">For a Blazor Server experience, choose the **Blazor Server App** template.</span></span> <span data-ttu-id="59f18-155">Selezionare **Avanti**.</span><span class="sxs-lookup"><span data-stu-id="59f18-155">Select **Next**.</span></span>

   <span data-ttu-id="59f18-156">Per informazioni sui due Blazor modelli di hosting *Blazor WebAssembly* (standalone e Hosted) e *Blazor Server* , vedere <xref:blazor/hosting-models> .</span><span class="sxs-lookup"><span data-stu-id="59f18-156">For information on the two Blazor hosting models, *Blazor WebAssembly* (standalone and hosted) and *Blazor Server*, see <xref:blazor/hosting-models>.</span></span>

1. <span data-ttu-id="59f18-157">Verificare che **l'autenticazione** sia impostata su **Nessuna autenticazione**.</span><span class="sxs-lookup"><span data-stu-id="59f18-157">Confirm that **Authentication** is set to **No Authentication**.</span></span> <span data-ttu-id="59f18-158">Selezionare **Avanti**.</span><span class="sxs-lookup"><span data-stu-id="59f18-158">Select **Next**.</span></span>

1. <span data-ttu-id="59f18-159">Per un'esperienza ospitata Blazor WebAssembly , selezionare la casella di controllo **ASP.NET Core Hosted** .</span><span class="sxs-lookup"><span data-stu-id="59f18-159">For a hosted Blazor WebAssembly experience, select the **ASP.NET Core hosted** check box.</span></span>

1. <span data-ttu-id="59f18-160">Nel campo **nome progetto** assegnare un nome all'app `WebApplication1` .</span><span class="sxs-lookup"><span data-stu-id="59f18-160">In the **Project Name** field, name the app `WebApplication1`.</span></span> <span data-ttu-id="59f18-161">Selezionare **Crea**.</span><span class="sxs-lookup"><span data-stu-id="59f18-161">Select **Create**.</span></span>

1. <span data-ttu-id="59f18-162">Selezionare **Esegui**  >  **Avvia senza eseguire debug** per eseguire l'app *senza il debugger*.</span><span class="sxs-lookup"><span data-stu-id="59f18-162">Select **Run** > **Start Without Debugging** to run the app *without the debugger*.</span></span> <span data-ttu-id="59f18-163">Eseguire l'app con **Esegui**  >  **debug Avvia debug** o il pulsante Esegui (&#9654;) per eseguire l'app *con il debugger*.</span><span class="sxs-lookup"><span data-stu-id="59f18-163">Run the app with **Run** > **Start Debugging** or the Run (&#9654;) button to run the app *with the debugger*.</span></span>

<span data-ttu-id="59f18-164">Se viene visualizzato un messaggio per considerare attendibile il certificato di sviluppo, considerare attendibile il certificato e continuare.</span><span class="sxs-lookup"><span data-stu-id="59f18-164">If a prompt appears to trust the development certificate, trust the certificate and continue.</span></span> <span data-ttu-id="59f18-165">Per considerare attendibile il certificato, è necessario specificare le password dell'utente e del keychain.</span><span class="sxs-lookup"><span data-stu-id="59f18-165">The user and keychain passwords are required to trust the certificate.</span></span> <span data-ttu-id="59f18-166">Per ulteriori informazioni su come considerare attendibile il certificato di sviluppo HTTPS ASP.NET Core, vedere <xref:security/enforcing-ssl#trust-the-aspnet-core-https-development-certificate-on-windows-and-macos> .</span><span class="sxs-lookup"><span data-stu-id="59f18-166">For more information on trusting the ASP.NET Core HTTPS development certificate, see <xref:security/enforcing-ssl#trust-the-aspnet-core-https-development-certificate-on-windows-and-macos>.</span></span>

<span data-ttu-id="59f18-167">Quando si esegue un'app ospitata Blazor WebAssembly , eseguire l'app dal progetto della soluzione **`Server`** .</span><span class="sxs-lookup"><span data-stu-id="59f18-167">When executing a hosted Blazor WebAssembly app, run the app from the solution's **`Server`** project.</span></span>

::: zone-end

## <a name="use-visual-studio-code-for-cross-platform-blazor-development"></a><span data-ttu-id="59f18-168">Usare Visual Studio Code per Blazor lo sviluppo multipiattaforma</span><span class="sxs-lookup"><span data-stu-id="59f18-168">Use Visual Studio Code for cross-platform Blazor development</span></span>

<span data-ttu-id="59f18-169">[Visual Studio Code](https://code.visualstudio.com/) è un ambiente di sviluppo integrato (IDE) open source e multipiattaforma che può essere usato per sviluppare Blazor app.</span><span class="sxs-lookup"><span data-stu-id="59f18-169">[Visual Studio Code](https://code.visualstudio.com/) is an open source, cross-platform Integrated Development Environment (IDE) that can be used to develop Blazor apps.</span></span> <span data-ttu-id="59f18-170">Usare l'interfaccia della riga di comando di .NET per creare una nuova Blazor app per lo sviluppo con Visual Studio Code.</span><span class="sxs-lookup"><span data-stu-id="59f18-170">Use the .NET CLI to create a new Blazor app for development with Visual Studio Code.</span></span> <span data-ttu-id="59f18-171">Per altre informazioni, vedere la [versione di questo articolo](?pivots=linux)relativa a Linux.</span><span class="sxs-lookup"><span data-stu-id="59f18-171">For more information, see the [Linux version of this article](?pivots=linux).</span></span>

## <a name="blazor-template-options"></a><span data-ttu-id="59f18-172">Blazor opzioni del modello</span><span class="sxs-lookup"><span data-stu-id="59f18-172">Blazor template options</span></span>

<span data-ttu-id="59f18-173">Il Blazor Framework fornisce modelli per la creazione di nuove app per ognuno dei due Blazor modelli di hosting.</span><span class="sxs-lookup"><span data-stu-id="59f18-173">The Blazor framework provides templates for creating new apps for each of the two Blazor hosting models.</span></span> <span data-ttu-id="59f18-174">I modelli vengono usati per creare nuovi Blazor progetti e soluzioni indipendentemente dagli strumenti selezionati per Blazor lo sviluppo (Visual Studio, Visual Studio per Mac, Visual Studio Code o l'interfaccia della riga di comando di .NET):</span><span class="sxs-lookup"><span data-stu-id="59f18-174">The templates are used to create new Blazor projects and solutions regardless of the tooling that you select for Blazor development (Visual Studio, Visual Studio for Mac, Visual Studio Code, or the .NET CLI):</span></span>

* <span data-ttu-id="59f18-175">Blazor WebAssembly modello di progetto: `blazorwasm`</span><span class="sxs-lookup"><span data-stu-id="59f18-175">Blazor WebAssembly project template: `blazorwasm`</span></span>
* <span data-ttu-id="59f18-176">Blazor Server modello di progetto: `blazorserver`</span><span class="sxs-lookup"><span data-stu-id="59f18-176">Blazor Server project template: `blazorserver`</span></span>

<span data-ttu-id="59f18-177">Per ulteriori informazioni sui Blazor modelli di hosting di, vedere <xref:blazor/hosting-models> .</span><span class="sxs-lookup"><span data-stu-id="59f18-177">For more information on Blazor's hosting models, see <xref:blazor/hosting-models>.</span></span> <span data-ttu-id="59f18-178">Per ulteriori informazioni sui Blazor modelli di progetto di, vedere <xref:blazor/project-structure> .</span><span class="sxs-lookup"><span data-stu-id="59f18-178">For more information on Blazor project templates, see <xref:blazor/project-structure>.</span></span>

<span data-ttu-id="59f18-179">Le opzioni del modello sono disponibili passando l'opzione Help ( `-h` o `--help` ) al [`dotnet new`](/dotnet/core/tools/dotnet-new) comando CLI in una shell dei comandi:</span><span class="sxs-lookup"><span data-stu-id="59f18-179">Template options are available by passing the help option (`-h` or `--help`) to the [`dotnet new`](/dotnet/core/tools/dotnet-new) CLI command in a command shell:</span></span>

```dotnetcli
dotnet new blazorwasm -h
dotnet new blazorserver -h
```

## <a name="additional-resources"></a><span data-ttu-id="59f18-180">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="59f18-180">Additional resources</span></span>

* <xref:blazor/hosting-models>
* <xref:blazor/project-structure>
