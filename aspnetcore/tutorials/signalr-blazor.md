---
title: Usare ASP.NET Core SignalR con Blazor
author: guardrex
description: Creare un'app di chat che usa ASP.NET Core SignalR con Blazor .
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 01/25/2021
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
uid: tutorials/signalr-blazor
zone_pivot_groups: blazor-hosting-models
ms.openlocfilehash: e9bb79eb5bd8f56fe8014981de76e210eb35c2de
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102589530"
---
# <a name="use-aspnet-core-signalr-with-blazor"></a><span data-ttu-id="49370-103">Usare ASP.NET Core SignalR con Blazor</span><span class="sxs-lookup"><span data-stu-id="49370-103">Use ASP.NET Core SignalR with Blazor</span></span>

<span data-ttu-id="49370-104">Questa esercitazione illustra le nozioni di base per la creazione di un'app in tempo reale usando SignalR con Blazor .</span><span class="sxs-lookup"><span data-stu-id="49370-104">This tutorial teaches the basics of building a real-time app using SignalR with Blazor.</span></span> <span data-ttu-id="49370-105">Si apprenderà come:</span><span class="sxs-lookup"><span data-stu-id="49370-105">You learn how to:</span></span>

> [!div class="checklist"]
> * <span data-ttu-id="49370-106">Creazione di un Blazor progetto</span><span class="sxs-lookup"><span data-stu-id="49370-106">Create a Blazor project</span></span>
> * <span data-ttu-id="49370-107">Aggiungere la SignalR libreria client</span><span class="sxs-lookup"><span data-stu-id="49370-107">Add the SignalR client library</span></span>
> * <span data-ttu-id="49370-108">Aggiungere un SignalR Hub</span><span class="sxs-lookup"><span data-stu-id="49370-108">Add a SignalR hub</span></span>
> * <span data-ttu-id="49370-109">Aggiungere SignalR Servizi e un endpoint per l' SignalR Hub</span><span class="sxs-lookup"><span data-stu-id="49370-109">Add SignalR services and an endpoint for the SignalR hub</span></span>
> * <span data-ttu-id="49370-110">Aggiungi Razor codice componente per la chat</span><span class="sxs-lookup"><span data-stu-id="49370-110">Add Razor component code for chat</span></span>

<span data-ttu-id="49370-111">Al termine di questa esercitazione, si disporrà di un'app di chat funzionante.</span><span class="sxs-lookup"><span data-stu-id="49370-111">At the end of this tutorial, you'll have a working chat app.</span></span>

<span data-ttu-id="49370-112">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/signalr-blazor/samples/) ([procedura per il download](xref:index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="49370-112">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/signalr-blazor/samples/) ([how to download](xref:index#how-to-download-a-sample))</span></span>

## <a name="prerequisites"></a><span data-ttu-id="49370-113">Prerequisiti</span><span class="sxs-lookup"><span data-stu-id="49370-113">Prerequisites</span></span>

::: moniker range=">= aspnetcore-5.0"

# <a name="visual-studio"></a>[<span data-ttu-id="49370-114">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="49370-114">Visual Studio</span></span>](#tab/visual-studio)

* <span data-ttu-id="49370-115">[Visual Studio 2019 16,8 o versione successiva](https://visualstudio.microsoft.com/downloads/?utm_medium=microsoft&utm_source=docs.microsoft.com&utm_campaign=inline+link&utm_content=download+vs2019) con il carico di lavoro **sviluppo di ASP.NET e Web**</span><span class="sxs-lookup"><span data-stu-id="49370-115">[Visual Studio 2019 16.8 or later](https://visualstudio.microsoft.com/downloads/?utm_medium=microsoft&utm_source=docs.microsoft.com&utm_campaign=inline+link&utm_content=download+vs2019) with the **ASP.NET and web development** workload</span></span>
* [!INCLUDE [.NET Core 5.0 SDK](~/includes/5.0-SDK.md)]

# <a name="visual-studio-code"></a>[<span data-ttu-id="49370-116">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="49370-116">Visual Studio Code</span></span>](#tab/visual-studio-code)

[!INCLUDE[](~/includes/net-core-prereqs-vsc-5.0.md)]

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="49370-117">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="49370-117">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

* [<span data-ttu-id="49370-118">Visual Studio per Mac versione 8,8 o successiva</span><span class="sxs-lookup"><span data-stu-id="49370-118">Visual Studio for Mac version 8.8 or later</span></span>](https://visualstudio.microsoft.com/vs/mac/)
* [!INCLUDE [.NET Core 5.0 SDK](~/includes/5.0-SDK.md)]

# <a name="net-core-cli"></a>[<span data-ttu-id="49370-119">Interfaccia della riga di comando di .NET Core</span><span class="sxs-lookup"><span data-stu-id="49370-119">.NET Core CLI</span></span>](#tab/netcore-cli/)

[!INCLUDE[](~/includes/5.0-SDK.md)]

---

::: moniker-end

::: moniker range="< aspnetcore-5.0"

# <a name="visual-studio"></a>[<span data-ttu-id="49370-120">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="49370-120">Visual Studio</span></span>](#tab/visual-studio)

* <span data-ttu-id="49370-121">[Visual Studio 2019 16,6 o versione successiva](https://visualstudio.microsoft.com/downloads/?utm_medium=microsoft&utm_source=docs.microsoft.com&utm_campaign=inline+link&utm_content=download+vs2019) con il carico di lavoro **sviluppo di ASP.NET e Web**</span><span class="sxs-lookup"><span data-stu-id="49370-121">[Visual Studio 2019 16.6 or later](https://visualstudio.microsoft.com/downloads/?utm_medium=microsoft&utm_source=docs.microsoft.com&utm_campaign=inline+link&utm_content=download+vs2019) with the **ASP.NET and web development** workload</span></span>
* [!INCLUDE [.NET Core 3.1 SDK](~/includes/3.1-SDK.md)]

# <a name="visual-studio-code"></a>[<span data-ttu-id="49370-122">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="49370-122">Visual Studio Code</span></span>](#tab/visual-studio-code)

[!INCLUDE[](~/includes/net-core-prereqs-vsc-3.1.md)]

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="49370-123">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="49370-123">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

* [<span data-ttu-id="49370-124">Visual Studio per Mac versione 8,6 o successiva</span><span class="sxs-lookup"><span data-stu-id="49370-124">Visual Studio for Mac version 8.6 or later</span></span>](https://visualstudio.microsoft.com/vs/mac/)
* [!INCLUDE [.NET Core 3.1 SDK](~/includes/3.1-SDK.md)]

# <a name="net-core-cli"></a>[<span data-ttu-id="49370-125">Interfaccia della riga di comando di .NET Core</span><span class="sxs-lookup"><span data-stu-id="49370-125">.NET Core CLI</span></span>](#tab/netcore-cli/)

[!INCLUDE[](~/includes/3.1-SDK.md)]

---

::: moniker-end

::: zone pivot="webassembly"

## <a name="create-a-hosted-blazor-webassembly-app"></a><span data-ttu-id="49370-126">Creare un'app ospitata Blazor WebAssembly</span><span class="sxs-lookup"><span data-stu-id="49370-126">Create a hosted Blazor WebAssembly app</span></span>

<span data-ttu-id="49370-127">Seguire le istruzioni per la scelta degli strumenti:</span><span class="sxs-lookup"><span data-stu-id="49370-127">Follow the guidance for your choice of tooling:</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="49370-128">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="49370-128">Visual Studio</span></span>](#tab/visual-studio)

::: moniker range=">= aspnetcore-5.0"

> [!NOTE]
> <span data-ttu-id="49370-129">Sono necessari Visual Studio 16,8 o versioni successive e .NET Core SDK 5.0.0 o versioni successive.</span><span class="sxs-lookup"><span data-stu-id="49370-129">Visual Studio 16.8 or later and .NET Core SDK 5.0.0 or later are required.</span></span>

::: moniker-end

::: moniker range="< aspnetcore-5.0"

> [!NOTE]
> <span data-ttu-id="49370-130">Sono necessari Visual Studio 16,6 o versioni successive e .NET Core SDK 3.1.300 o versioni successive.</span><span class="sxs-lookup"><span data-stu-id="49370-130">Visual Studio 16.6 or later and .NET Core SDK 3.1.300 or later are required.</span></span>

::: moniker-end

1. <span data-ttu-id="49370-131">Creare un nuovo progetto.</span><span class="sxs-lookup"><span data-stu-id="49370-131">Create a new project.</span></span>

1. <span data-ttu-id="49370-132">Selezionare **Blazor app** e fare clic su **Next (avanti**).</span><span class="sxs-lookup"><span data-stu-id="49370-132">Select **Blazor App** and select **Next**.</span></span>

1. <span data-ttu-id="49370-133">Digitare `BlazorWebAssemblySignalRApp` nel campo **nome progetto** .</span><span class="sxs-lookup"><span data-stu-id="49370-133">Type `BlazorWebAssemblySignalRApp` in the **Project name** field.</span></span> <span data-ttu-id="49370-134">Confermare che la voce relativa al **percorso** sia corretta o specificare un percorso per il progetto.</span><span class="sxs-lookup"><span data-stu-id="49370-134">Confirm the **Location** entry is correct or provide a location for the project.</span></span> <span data-ttu-id="49370-135">Selezionare **Crea**.</span><span class="sxs-lookup"><span data-stu-id="49370-135">Select **Create**.</span></span>

1. <span data-ttu-id="49370-136">Scegliere il modello di **Blazor WebAssembly app** .</span><span class="sxs-lookup"><span data-stu-id="49370-136">Choose the **Blazor WebAssembly App** template.</span></span>

1. <span data-ttu-id="49370-137">In **Avanzate** selezionare la casella di controllo **ASP.NET Core Hosted** .</span><span class="sxs-lookup"><span data-stu-id="49370-137">Under **Advanced**, select the **ASP.NET Core hosted** check box.</span></span>

1. <span data-ttu-id="49370-138">Selezionare **Crea**.</span><span class="sxs-lookup"><span data-stu-id="49370-138">Select **Create**.</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="49370-139">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="49370-139">Visual Studio Code</span></span>](#tab/visual-studio-code)

1. <span data-ttu-id="49370-140">In una shell dei comandi eseguire il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="49370-140">In a command shell, execute the following command:</span></span>

   ```dotnetcli
   dotnet new blazorwasm -ho -o BlazorWebAssemblySignalRApp
   ```

   <span data-ttu-id="49370-141">L' `-ho|--hosted` opzione Crea una soluzione ospitata Blazor WebAssembly .</span><span class="sxs-lookup"><span data-stu-id="49370-141">The `-ho|--hosted` option creates a hosted Blazor WebAssembly solution.</span></span> <span data-ttu-id="49370-142">Per informazioni sulla configurazione di asset VS Code nella `.vscode` cartella, vedere la guida del sistema operativo **Linux** in <xref:blazor/tooling> .</span><span class="sxs-lookup"><span data-stu-id="49370-142">For information on configuring VS Code assets in the `.vscode` folder, see the **Linux** operating system guidance in <xref:blazor/tooling>.</span></span>

   <span data-ttu-id="49370-143">L' `-o|--output` opzione Crea una cartella per la soluzione.</span><span class="sxs-lookup"><span data-stu-id="49370-143">The `-o|--output` option creates a folder for the solution.</span></span> <span data-ttu-id="49370-144">Se è stata creata una cartella per la soluzione e la shell dei comandi è aperta in tale cartella, omettere l' `-o|--output` opzione e il valore per creare la soluzione.</span><span class="sxs-lookup"><span data-stu-id="49370-144">If you've created a folder for the solution and the command shell is open in that folder, omit the `-o|--output` option and value to create the solution.</span></span>

1. <span data-ttu-id="49370-145">In Visual Studio Code aprire la cartella del progetto dell'app.</span><span class="sxs-lookup"><span data-stu-id="49370-145">In Visual Studio Code, open the app's project folder.</span></span>

1. <span data-ttu-id="49370-146">Quando viene visualizzata la finestra di dialogo per aggiungere asset per compilare ed eseguire il debug dell'app, selezionare **Sì**.</span><span class="sxs-lookup"><span data-stu-id="49370-146">When the dialog appears to add assets to build and debug the app, select **Yes**.</span></span> <span data-ttu-id="49370-147">Visual Studio Code aggiunge automaticamente la `.vscode` cartella con `launch.json` `tasks.json` i file e generati.</span><span class="sxs-lookup"><span data-stu-id="49370-147">Visual Studio Code automatically adds the `.vscode` folder with generated `launch.json` and `tasks.json` files.</span></span>

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="49370-148">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="49370-148">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

1. <span data-ttu-id="49370-149">Installare la versione più recente di [Visual Studio per Mac](https://visualstudio.microsoft.com/vs/mac/) e seguire questa procedura:</span><span class="sxs-lookup"><span data-stu-id="49370-149">Install the latest version of [Visual Studio for Mac](https://visualstudio.microsoft.com/vs/mac/) and perform the following steps:</span></span>

1. <span data-ttu-id="49370-150">Selezionare **file**  >  **nuova soluzione** o creare un **nuovo** progetto dalla **finestra Start**.</span><span class="sxs-lookup"><span data-stu-id="49370-150">Select **File** > **New Solution** or create a **New** project from the **Start Window**.</span></span>

1. <span data-ttu-id="49370-151">Nella barra laterale selezionare app **Web e console**  >  .</span><span class="sxs-lookup"><span data-stu-id="49370-151">In the sidebar, select **Web and Console** > **App**.</span></span>

1. <span data-ttu-id="49370-152">Scegliere il modello di **Blazor WebAssembly app** .</span><span class="sxs-lookup"><span data-stu-id="49370-152">Choose the **Blazor WebAssembly App** template.</span></span> <span data-ttu-id="49370-153">Selezionare **Avanti**.</span><span class="sxs-lookup"><span data-stu-id="49370-153">Select **Next**.</span></span>

1. <span data-ttu-id="49370-154">Verificare che **l'autenticazione** sia impostata su **Nessuna autenticazione**.</span><span class="sxs-lookup"><span data-stu-id="49370-154">Confirm that **Authentication** is set to **No Authentication**.</span></span> <span data-ttu-id="49370-155">Selezionare la casella di controllo **ASP.NET Core Hosted** .</span><span class="sxs-lookup"><span data-stu-id="49370-155">Select the **ASP.NET Core Hosted** check box.</span></span> <span data-ttu-id="49370-156">Selezionare **Avanti**.</span><span class="sxs-lookup"><span data-stu-id="49370-156">Select **Next**.</span></span>

1. <span data-ttu-id="49370-157">Nel campo **nome progetto** assegnare un nome all'app `BlazorWebAssemblySignalRApp` .</span><span class="sxs-lookup"><span data-stu-id="49370-157">In the **Project Name** field, name the app `BlazorWebAssemblySignalRApp`.</span></span> <span data-ttu-id="49370-158">Selezionare **Crea**.</span><span class="sxs-lookup"><span data-stu-id="49370-158">Select **Create**.</span></span>

   <span data-ttu-id="49370-159">Se viene visualizzato un messaggio per considerare attendibile il certificato di sviluppo, considerare attendibile il certificato e continuare.</span><span class="sxs-lookup"><span data-stu-id="49370-159">If a prompt appears to trust the development certificate, trust the certificate and continue.</span></span> <span data-ttu-id="49370-160">Per considerare attendibile il certificato, è necessario specificare le password dell'utente e del keychain.</span><span class="sxs-lookup"><span data-stu-id="49370-160">The user and keychain passwords are required to trust the certificate.</span></span>

1. <span data-ttu-id="49370-161">Aprire il progetto passando alla cartella del progetto e aprendo il file di soluzione del progetto ( `.sln` ).</span><span class="sxs-lookup"><span data-stu-id="49370-161">Open the project by navigating to the project folder and opening the project's solution file (`.sln`).</span></span>

# <a name="net-core-cli"></a>[<span data-ttu-id="49370-162">Interfaccia della riga di comando di .NET Core</span><span class="sxs-lookup"><span data-stu-id="49370-162">.NET Core CLI</span></span>](#tab/netcore-cli/)

<span data-ttu-id="49370-163">In una shell dei comandi eseguire il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="49370-163">In a command shell, execute the following command:</span></span>

```dotnetcli
dotnet new blazorwasm -ho -o BlazorWebAssemblySignalRApp
```

<span data-ttu-id="49370-164">L' `-ho|--hosted` opzione Crea una soluzione ospitata Blazor WebAssembly .</span><span class="sxs-lookup"><span data-stu-id="49370-164">The `-ho|--hosted` option creates a hosted Blazor WebAssembly solution.</span></span>

<span data-ttu-id="49370-165">L' `-o|--output` opzione Crea una cartella per la soluzione.</span><span class="sxs-lookup"><span data-stu-id="49370-165">The `-o|--output` option creates a folder for the solution.</span></span> <span data-ttu-id="49370-166">Se è stata creata una cartella per la soluzione e la shell dei comandi è aperta in tale cartella, omettere l' `-o|--output` opzione e il valore per creare la soluzione.</span><span class="sxs-lookup"><span data-stu-id="49370-166">If you've created a folder for the solution and the command shell is open in that folder, omit the `-o|--output` option and value to create the solution.</span></span>

---

## <a name="add-the-signalr-client-library"></a><span data-ttu-id="49370-167">Aggiungere la SignalR libreria client</span><span class="sxs-lookup"><span data-stu-id="49370-167">Add the SignalR client library</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="49370-168">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="49370-168">Visual Studio</span></span>](#tab/visual-studio/)

1. <span data-ttu-id="49370-169">In **Esplora soluzioni** fare clic con il pulsante destro del mouse sul `BlazorWebAssemblySignalRApp.Client` progetto e scegliere **Gestisci pacchetti NuGet**.</span><span class="sxs-lookup"><span data-stu-id="49370-169">In **Solution Explorer**, right-click the `BlazorWebAssemblySignalRApp.Client` project and select **Manage NuGet Packages**.</span></span>

1. <span data-ttu-id="49370-170">Nella finestra di dialogo **Gestisci pacchetti NuGet** verificare che l' **origine del pacchetto** sia impostata su `nuget.org` .</span><span class="sxs-lookup"><span data-stu-id="49370-170">In the **Manage NuGet Packages** dialog, confirm that the **Package source** is set to `nuget.org`.</span></span>

1. <span data-ttu-id="49370-171">Con **Sfoglia** selezionato, digitare `Microsoft.AspNetCore.SignalR.Client` nella casella di ricerca.</span><span class="sxs-lookup"><span data-stu-id="49370-171">With **Browse** selected, type `Microsoft.AspNetCore.SignalR.Client` in the search box.</span></span>

1. <span data-ttu-id="49370-172">Nei risultati della ricerca selezionare il [`Microsoft.AspNetCore.SignalR.Client`](https://www.nuget.org/packages/Microsoft.AspNetCore.SignalR.Client) pacchetto.</span><span class="sxs-lookup"><span data-stu-id="49370-172">In the search results, select the [`Microsoft.AspNetCore.SignalR.Client`](https://www.nuget.org/packages/Microsoft.AspNetCore.SignalR.Client) package.</span></span> <span data-ttu-id="49370-173">Impostare la versione in modo che corrisponda al Framework condiviso dell'app.</span><span class="sxs-lookup"><span data-stu-id="49370-173">Set the version to match the shared framework of the app.</span></span> <span data-ttu-id="49370-174">Selezionare **Installa**.</span><span class="sxs-lookup"><span data-stu-id="49370-174">Select **Install**.</span></span>

1. <span data-ttu-id="49370-175">Se viene visualizzata la finestra di dialogo **Anteprima modifiche** , selezionare **OK**.</span><span class="sxs-lookup"><span data-stu-id="49370-175">If the **Preview Changes** dialog appears, select **OK**.</span></span>

1. <span data-ttu-id="49370-176">Se viene visualizzata la finestra di dialogo **accettazione della licenza** , selezionare Accetto se **si accettano le** condizioni di licenza.</span><span class="sxs-lookup"><span data-stu-id="49370-176">If the **License Acceptance** dialog appears, select **I Accept** if you agree with the license terms.</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="49370-177">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="49370-177">Visual Studio Code</span></span>](#tab/visual-studio-code/)

<span data-ttu-id="49370-178">Nel **terminale integrato** (**visualizzare** il  >  **terminale** dalla barra degli strumenti) eseguire il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="49370-178">In the **Integrated Terminal** (**View** > **Terminal** from the toolbar), execute the following command:</span></span>

```dotnetcli
dotnet add Client package Microsoft.AspNetCore.SignalR.Client
```

<span data-ttu-id="49370-179">Per aggiungere una versione precedente del pacchetto, specificare l' `--version {VERSION}` opzione, dove il `{VERSION}` segnaposto è la versione del pacchetto da aggiungere.</span><span class="sxs-lookup"><span data-stu-id="49370-179">To add an earlier version of the package, supply the `--version {VERSION}` option, where the `{VERSION}` placeholder is the version of the package to add.</span></span>

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="49370-180">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="49370-180">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

1. <span data-ttu-id="49370-181">Nella barra laterale **soluzione** fare clic con il pulsante destro del mouse sul `BlazorWebAssemblySignalRApp.Client` progetto e scegliere **Gestisci pacchetti NuGet**.</span><span class="sxs-lookup"><span data-stu-id="49370-181">In the **Solution** sidebar, right-click the `BlazorWebAssemblySignalRApp.Client` project and select **Manage NuGet Packages**.</span></span>

1. <span data-ttu-id="49370-182">Nella finestra di dialogo **Gestisci pacchetti NuGet** verificare che l'elenco a discesa origine sia impostato su `nuget.org` .</span><span class="sxs-lookup"><span data-stu-id="49370-182">In the **Manage NuGet Packages** dialog, confirm that the source drop-down is set to `nuget.org`.</span></span>

1. <span data-ttu-id="49370-183">Con **Sfoglia** selezionato, digitare `Microsoft.AspNetCore.SignalR.Client` nella casella di ricerca.</span><span class="sxs-lookup"><span data-stu-id="49370-183">With **Browse** selected, type `Microsoft.AspNetCore.SignalR.Client` in the search box.</span></span>

1. <span data-ttu-id="49370-184">Nei risultati della ricerca selezionare la casella di controllo accanto al [`Microsoft.AspNetCore.SignalR.Client`](https://www.nuget.org/packages/Microsoft.AspNetCore.SignalR.Client) pacchetto.</span><span class="sxs-lookup"><span data-stu-id="49370-184">In the search results, select the check box next to the [`Microsoft.AspNetCore.SignalR.Client`](https://www.nuget.org/packages/Microsoft.AspNetCore.SignalR.Client) package.</span></span> <span data-ttu-id="49370-185">Impostare la versione in modo che corrisponda al Framework condiviso dell'app.</span><span class="sxs-lookup"><span data-stu-id="49370-185">Set the version to match the shared framework of the app.</span></span> <span data-ttu-id="49370-186">Selezionare **Aggiungi pacchetto**.</span><span class="sxs-lookup"><span data-stu-id="49370-186">Select **Add Package**.</span></span>

1. <span data-ttu-id="49370-187">Se viene visualizzata la finestra di dialogo **accettazione della licenza** , selezionare **Accetto** se si accettano le condizioni di licenza.</span><span class="sxs-lookup"><span data-stu-id="49370-187">If the **License Acceptance** dialog appears, select **Accept** if you agree with the license terms.</span></span>

# <a name="net-core-cli"></a>[<span data-ttu-id="49370-188">Interfaccia della riga di comando di .NET Core</span><span class="sxs-lookup"><span data-stu-id="49370-188">.NET Core CLI</span></span>](#tab/netcore-cli/)

<span data-ttu-id="49370-189">In una shell dei comandi dalla cartella della soluzione, eseguire il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="49370-189">In a command shell from the solution's folder, execute the following command:</span></span>

```dotnetcli
dotnet add Client package Microsoft.AspNetCore.SignalR.Client
```

<span data-ttu-id="49370-190">Per aggiungere una versione precedente del pacchetto, specificare l' `--version {VERSION}` opzione, dove il `{VERSION}` segnaposto è la versione del pacchetto da aggiungere.</span><span class="sxs-lookup"><span data-stu-id="49370-190">To add an earlier version of the package, supply the `--version {VERSION}` option, where the `{VERSION}` placeholder is the version of the package to add.</span></span>

---

::: moniker range="< aspnetcore-5.0"

## <a name="add-the-systemtextencodingsweb-package"></a><span data-ttu-id="49370-191">Aggiungere il pacchetto System. Text. Encodings. Web</span><span class="sxs-lookup"><span data-stu-id="49370-191">Add the System.Text.Encodings.Web package</span></span>

<span data-ttu-id="49370-192">*Questa sezione si applica solo alle app per ASP.NET Core versione 3. x.*</span><span class="sxs-lookup"><span data-stu-id="49370-192">*This section only applies to apps for ASP.NET Core version 3.x.*</span></span>

<span data-ttu-id="49370-193">A causa di un problema di risoluzione del pacchetto quando si usa [`System.Text.Json`](https://www.nuget.org/packages/System.Text.Json) 5. x in un'app ASP.NET Core 3. x, il `BlazorWebAssemblySignalRApp.Server` progetto richiede un riferimento al pacchetto per [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web) .</span><span class="sxs-lookup"><span data-stu-id="49370-193">Due to a package resolution issue when using [`System.Text.Json`](https://www.nuget.org/packages/System.Text.Json) 5.x in an ASP.NET Core 3.x app, the `BlazorWebAssemblySignalRApp.Server` project requires a package reference for [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web).</span></span> <span data-ttu-id="49370-194">Il problema sottostante verrà risolto in una versione futura della patch di .NET 5.</span><span class="sxs-lookup"><span data-stu-id="49370-194">The underlying issue will be resolved in a future patch release of .NET 5.</span></span> <span data-ttu-id="49370-195">Per altre informazioni, vedere [System.Text.Json definisce netcoreapp 3.0 senza dipendenze (DotNet/runtime #45560)](https://github.com/dotnet/runtime/issues/45560).</span><span class="sxs-lookup"><span data-stu-id="49370-195">For more information, see [System.Text.Json defines netcoreapp3.0 with no dependencies (dotnet/runtime #45560)](https://github.com/dotnet/runtime/issues/45560).</span></span>

<span data-ttu-id="49370-196">Per aggiungere [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web) al `BlazorWebAssemblySignalRApp.Server` progetto della soluzione ospitata ASP.NET Core 3,1 Blazor , seguire le istruzioni per la scelta degli strumenti:</span><span class="sxs-lookup"><span data-stu-id="49370-196">To add [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web) to the `BlazorWebAssemblySignalRApp.Server` project of the ASP.NET Core 3.1 hosted Blazor solution, follow the guidance for your choice of tooling:</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="49370-197">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="49370-197">Visual Studio</span></span>](#tab/visual-studio/)

1. <span data-ttu-id="49370-198">In **Esplora soluzioni** fare clic con il pulsante destro del mouse sul `BlazorWebAssemblySignalRApp.Server` progetto e scegliere **Gestisci pacchetti NuGet**.</span><span class="sxs-lookup"><span data-stu-id="49370-198">In **Solution Explorer**, right-click the `BlazorWebAssemblySignalRApp.Server` project and select **Manage NuGet Packages**.</span></span>

1. <span data-ttu-id="49370-199">Nella finestra di dialogo **Gestisci pacchetti NuGet** verificare che l' **origine del pacchetto** sia impostata su `nuget.org` .</span><span class="sxs-lookup"><span data-stu-id="49370-199">In the **Manage NuGet Packages** dialog, confirm that the **Package source** is set to `nuget.org`.</span></span>

1. <span data-ttu-id="49370-200">Con **Sfoglia** selezionato, digitare `System.Text.Encodings.Web` nella casella di ricerca.</span><span class="sxs-lookup"><span data-stu-id="49370-200">With **Browse** selected, type `System.Text.Encodings.Web` in the search box.</span></span>

1. <span data-ttu-id="49370-201">Nei risultati della ricerca selezionare il [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web) pacchetto.</span><span class="sxs-lookup"><span data-stu-id="49370-201">In the search results, select the [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web) package.</span></span> <span data-ttu-id="49370-202">Consente di selezionare la versione del pacchetto corrispondente al Framework condiviso in uso.</span><span class="sxs-lookup"><span data-stu-id="49370-202">Select the version of the package that matches the shared framework in use.</span></span> <span data-ttu-id="49370-203">Selezionare **Installa**.</span><span class="sxs-lookup"><span data-stu-id="49370-203">Select **Install**.</span></span>

1. <span data-ttu-id="49370-204">Se viene visualizzata la finestra di dialogo **Anteprima modifiche** , selezionare **OK**.</span><span class="sxs-lookup"><span data-stu-id="49370-204">If the **Preview Changes** dialog appears, select **OK**.</span></span>

1. <span data-ttu-id="49370-205">Se viene visualizzata la finestra di dialogo **accettazione della licenza** , selezionare Accetto se **si accettano le** condizioni di licenza.</span><span class="sxs-lookup"><span data-stu-id="49370-205">If the **License Acceptance** dialog appears, select **I Accept** if you agree with the license terms.</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="49370-206">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="49370-206">Visual Studio Code</span></span>](#tab/visual-studio-code/)

<span data-ttu-id="49370-207">Nel **terminale integrato** (**visualizzare** il > **terminale** dalla barra degli strumenti) eseguire i comandi seguenti:</span><span class="sxs-lookup"><span data-stu-id="49370-207">In the **Integrated Terminal** (**View** > **Terminal** from the toolbar), execute the following commands:</span></span>

```dotnetcli
dotnet add Server package System.Text.Encodings.Web
```

<span data-ttu-id="49370-208">Per aggiungere una versione precedente del pacchetto, specificare l' `--version {VERSION}` opzione, dove il `{VERSION}` segnaposto è la versione del pacchetto da aggiungere.</span><span class="sxs-lookup"><span data-stu-id="49370-208">To add an earlier version of the package, supply the `--version {VERSION}` option, where the `{VERSION}` placeholder is the version of the package to add.</span></span>

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="49370-209">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="49370-209">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

1. <span data-ttu-id="49370-210">Nella barra laterale **soluzione** fare clic con il pulsante destro del mouse sul `BlazorWebAssemblySignalRApp.Server` progetto e scegliere **Gestisci pacchetti NuGet**.</span><span class="sxs-lookup"><span data-stu-id="49370-210">In the **Solution** sidebar, right-click the `BlazorWebAssemblySignalRApp.Server` project and select **Manage NuGet Packages**.</span></span>

1. <span data-ttu-id="49370-211">Nella finestra di dialogo **Gestisci pacchetti NuGet** verificare che l'elenco a discesa origine sia impostato su `nuget.org` .</span><span class="sxs-lookup"><span data-stu-id="49370-211">In the **Manage NuGet Packages** dialog, confirm that the source drop-down is set to `nuget.org`.</span></span>

1. <span data-ttu-id="49370-212">Con **Sfoglia** selezionato, digitare `System.Text.Encodings.Web` nella casella di ricerca.</span><span class="sxs-lookup"><span data-stu-id="49370-212">With **Browse** selected, type `System.Text.Encodings.Web` in the search box.</span></span>

1. <span data-ttu-id="49370-213">Nei risultati della ricerca selezionare la casella di controllo accanto al [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web) pacchetto, selezionare la versione corretta del pacchetto corrispondente al Framework condiviso in uso e selezionare **Aggiungi pacchetto**.</span><span class="sxs-lookup"><span data-stu-id="49370-213">In the search results, select the check box next to the [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web) package, select the correct version of the package that matches the shared framework in use, and select **Add Package**.</span></span>

1. <span data-ttu-id="49370-214">Se viene visualizzata la finestra di dialogo **accettazione della licenza** , selezionare **Accetto** se si accettano le condizioni di licenza.</span><span class="sxs-lookup"><span data-stu-id="49370-214">If the **License Acceptance** dialog appears, select **Accept** if you agree with the license terms.</span></span>

# <a name="net-core-cli"></a>[<span data-ttu-id="49370-215">Interfaccia della riga di comando di .NET Core</span><span class="sxs-lookup"><span data-stu-id="49370-215">.NET Core CLI</span></span>](#tab/netcore-cli/)

<span data-ttu-id="49370-216">In una shell dei comandi dalla cartella della soluzione, eseguire il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="49370-216">In a command shell from the solution's folder, execute the following command:</span></span>

```dotnetcli
dotnet add Server package System.Text.Encodings.Web
```

<span data-ttu-id="49370-217">Per aggiungere una versione precedente del pacchetto, specificare l' `--version {VERSION}` opzione, dove il `{VERSION}` segnaposto è la versione del pacchetto da aggiungere.</span><span class="sxs-lookup"><span data-stu-id="49370-217">To add an earlier version of the package, supply the `--version {VERSION}` option, where the `{VERSION}` placeholder is the version of the package to add.</span></span>

---

::: moniker-end

## <a name="add-a-signalr-hub"></a><span data-ttu-id="49370-218">Aggiungere un SignalR Hub</span><span class="sxs-lookup"><span data-stu-id="49370-218">Add a SignalR hub</span></span>

<span data-ttu-id="49370-219">Nel `BlazorWebAssemblySignalRApp.Server` progetto creare una `Hubs` cartella (plurale) e aggiungere la classe seguente `ChatHub` ( `Hubs/ChatHub.cs` ):</span><span class="sxs-lookup"><span data-stu-id="49370-219">In the `BlazorWebAssemblySignalRApp.Server` project, create a `Hubs` (plural) folder and add the following `ChatHub` class (`Hubs/ChatHub.cs`):</span></span>

::: moniker range=">= aspnetcore-5.0"

[!code-csharp[](~/tutorials/signalr-blazor/samples/5.x/BlazorWebAssemblySignalRApp/Server/Hubs/ChatHub.cs)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-csharp[](~/tutorials/signalr-blazor/samples/3.x/BlazorWebAssemblySignalRApp/Server/Hubs/ChatHub.cs)]

::: moniker-end

## <a name="add-services-and-an-endpoint-for-the-signalr-hub"></a><span data-ttu-id="49370-220">Aggiungere servizi e un endpoint per l' SignalR Hub</span><span class="sxs-lookup"><span data-stu-id="49370-220">Add services and an endpoint for the SignalR hub</span></span>

1. <span data-ttu-id="49370-221">Nel progetto `BlazorWebAssemblySignalRApp.Server` aprire il file `Startup.cs`.</span><span class="sxs-lookup"><span data-stu-id="49370-221">In the `BlazorWebAssemblySignalRApp.Server` project, open the `Startup.cs` file.</span></span>

1. <span data-ttu-id="49370-222">Aggiungere lo spazio dei nomi per la `ChatHub` classe all'inizio del file:</span><span class="sxs-lookup"><span data-stu-id="49370-222">Add the namespace for the `ChatHub` class to the top of the file:</span></span>

   ```csharp
   using BlazorWebAssemblySignalRApp.Server.Hubs;
   ```

::: moniker range=">= aspnetcore-5.0"

1. <span data-ttu-id="49370-223">Aggiungere SignalR e rispondere ai servizi middleware di compressione per `Startup.ConfigureServices` :</span><span class="sxs-lookup"><span data-stu-id="49370-223">Add SignalR and Response Compression Middleware services to `Startup.ConfigureServices`:</span></span>

   [!code-csharp[](~/tutorials/signalr-blazor/samples/5.x/BlazorWebAssemblySignalRApp/Server/Startup.cs?name=snippet_ConfigureServices&highlight=3,6-10)]

1. <span data-ttu-id="49370-224">In `Startup.Configure`:</span><span class="sxs-lookup"><span data-stu-id="49370-224">In `Startup.Configure`:</span></span>

   * <span data-ttu-id="49370-225">Usare il middleware della compressione della risposta nella parte superiore della configurazione della pipeline di elaborazione.</span><span class="sxs-lookup"><span data-stu-id="49370-225">Use Response Compression Middleware at the top of the processing pipeline's configuration.</span></span>
   * <span data-ttu-id="49370-226">Tra gli endpoint per i controller e il fallback sul lato client, aggiungere un endpoint per l'hub.</span><span class="sxs-lookup"><span data-stu-id="49370-226">Between the endpoints for controllers and the client-side fallback, add an endpoint for the hub.</span></span>

   [!code-csharp[](~/tutorials/signalr-blazor/samples/5.x/BlazorWebAssemblySignalRApp/Server/Startup.cs?name=snippet_Configure&highlight=3,26)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

1. <span data-ttu-id="49370-227">Aggiungere SignalR e rispondere ai servizi middleware di compressione per `Startup.ConfigureServices` :</span><span class="sxs-lookup"><span data-stu-id="49370-227">Add SignalR and Response Compression Middleware services to `Startup.ConfigureServices`:</span></span>

   [!code-csharp[](~/tutorials/signalr-blazor/samples/3.x/BlazorWebAssemblySignalRApp/Server/Startup.cs?name=snippet_ConfigureServices&highlight=3,5-9)]

1. <span data-ttu-id="49370-228">In `Startup.Configure`:</span><span class="sxs-lookup"><span data-stu-id="49370-228">In `Startup.Configure`:</span></span>

   * <span data-ttu-id="49370-229">Usare il middleware della compressione della risposta nella parte superiore della configurazione della pipeline di elaborazione.</span><span class="sxs-lookup"><span data-stu-id="49370-229">Use Response Compression Middleware at the top of the processing pipeline's configuration.</span></span>
   * <span data-ttu-id="49370-230">Tra gli endpoint per i controller e il fallback sul lato client, aggiungere un endpoint per l'hub.</span><span class="sxs-lookup"><span data-stu-id="49370-230">Between the endpoints for controllers and the client-side fallback, add an endpoint for the hub.</span></span>

   [!code-csharp[](~/tutorials/signalr-blazor/samples/3.x/BlazorWebAssemblySignalRApp/Server/Startup.cs?name=snippet_Configure&highlight=3,25)]

::: moniker-end

## <a name="add-razor-component-code-for-chat"></a><span data-ttu-id="49370-231">Aggiungi Razor codice componente per la chat</span><span class="sxs-lookup"><span data-stu-id="49370-231">Add Razor component code for chat</span></span>

1. <span data-ttu-id="49370-232">Nel progetto `BlazorWebAssemblySignalRApp.Client` aprire il file `Pages/Index.razor`.</span><span class="sxs-lookup"><span data-stu-id="49370-232">In the `BlazorWebAssemblySignalRApp.Client` project, open the `Pages/Index.razor` file.</span></span>

::: moniker range=">= aspnetcore-5.0"

1. <span data-ttu-id="49370-233">Sostituire il markup con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="49370-233">Replace the markup with the following code:</span></span>

   [!code-razor[](~/tutorials/signalr-blazor/samples/5.x/BlazorWebAssemblySignalRApp/Client/Pages/Index.razor)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

1. <span data-ttu-id="49370-234">Sostituire il markup con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="49370-234">Replace the markup with the following code:</span></span>

   [!code-razor[](~/tutorials/signalr-blazor/samples/3.x/BlazorWebAssemblySignalRApp/Client/Pages/Index.razor)]

::: moniker-end

## <a name="run-the-app"></a><span data-ttu-id="49370-235">Eseguire l'app</span><span class="sxs-lookup"><span data-stu-id="49370-235">Run the app</span></span>

<span data-ttu-id="49370-236">Seguire le istruzioni per gli strumenti:</span><span class="sxs-lookup"><span data-stu-id="49370-236">Follow the guidance for your tooling:</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="49370-237">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="49370-237">Visual Studio</span></span>](#tab/visual-studio)

1. <span data-ttu-id="49370-238">In **Esplora soluzioni** selezionare il `BlazorWebAssemblySignalRApp.Server` progetto.</span><span class="sxs-lookup"><span data-stu-id="49370-238">In **Solution Explorer**, select the `BlazorWebAssemblySignalRApp.Server` project.</span></span> <span data-ttu-id="49370-239">Premere <kbd>F5</kbd> per eseguire l'app con il debug o <kbd>CTRL</kbd> + <kbd>F5</kbd> per eseguire l'app senza debug.</span><span class="sxs-lookup"><span data-stu-id="49370-239">Press <kbd>F5</kbd> to run the app with debugging or <kbd>Ctrl</kbd>+<kbd>F5</kbd> to run the app without debugging.</span></span>

1. <span data-ttu-id="49370-240">Copiare l'URL dalla barra degli indirizzi, aprire un'altra istanza o scheda del browser e incollare l'URL nella barra degli indirizzi.</span><span class="sxs-lookup"><span data-stu-id="49370-240">Copy the URL from the address bar, open another browser instance or tab, and paste the URL in the address bar.</span></span>

1. <span data-ttu-id="49370-241">Scegliere browser, immettere un nome e un messaggio e selezionare il pulsante per l'invio del messaggio.</span><span class="sxs-lookup"><span data-stu-id="49370-241">Choose either browser, enter a name and message, and select the button to send the message.</span></span> <span data-ttu-id="49370-242">Il nome e il messaggio vengono visualizzati immediatamente in entrambe le pagine:</span><span class="sxs-lookup"><span data-stu-id="49370-242">The name and message are displayed on both pages instantly:</span></span>

   ![::: NO-LOC (SignalR):::::: NO-LOC (Blazer)::: app di esempio aperta in due finestre del browser che mostrano i messaggi scambiati.](signalr-blazor/_static/signalr-blazor-finished.png)

   <span data-ttu-id="49370-244">Virgolette: *Star Trek VI: il paese non individuato* &copy; 1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)</span><span class="sxs-lookup"><span data-stu-id="49370-244">Quotes: *Star Trek VI: The Undiscovered Country* &copy;1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="49370-245">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="49370-245">Visual Studio Code</span></span>](#tab/visual-studio-code)

<span data-ttu-id="49370-246">Per informazioni sulla configurazione di asset VS Code nella `.vscode` cartella, vedere la guida del sistema operativo **Linux** in <xref:blazor/tooling> .</span><span class="sxs-lookup"><span data-stu-id="49370-246">For information on configuring VS Code assets in the `.vscode` folder, see the **Linux** operating system guidance in <xref:blazor/tooling>.</span></span>

1. <span data-ttu-id="49370-247">Premere <kbd>F5</kbd> per eseguire l'app con il debug o <kbd>CTRL</kbd> + <kbd>F5</kbd> per eseguire l'app senza debug.</span><span class="sxs-lookup"><span data-stu-id="49370-247">Press <kbd>F5</kbd> to run the app with debugging or <kbd>Ctrl</kbd>+<kbd>F5</kbd> to run the app without debugging.</span></span>

1. <span data-ttu-id="49370-248">Copiare l'URL dalla barra degli indirizzi, aprire un'altra istanza o scheda del browser e incollare l'URL nella barra degli indirizzi.</span><span class="sxs-lookup"><span data-stu-id="49370-248">Copy the URL from the address bar, open another browser instance or tab, and paste the URL in the address bar.</span></span>

1. <span data-ttu-id="49370-249">Scegliere browser, immettere un nome e un messaggio e selezionare il pulsante per l'invio del messaggio.</span><span class="sxs-lookup"><span data-stu-id="49370-249">Choose either browser, enter a name and message, and select the button to send the message.</span></span> <span data-ttu-id="49370-250">Il nome e il messaggio vengono visualizzati immediatamente in entrambe le pagine:</span><span class="sxs-lookup"><span data-stu-id="49370-250">The name and message are displayed on both pages instantly:</span></span>

   ![::: NO-LOC (SignalR):::::: NO-LOC (Blazer)::: app di esempio aperta in due finestre del browser che mostrano i messaggi scambiati.](signalr-blazor/_static/signalr-blazor-finished.png)

   <span data-ttu-id="49370-252">Virgolette: *Star Trek VI: il paese non individuato* &copy; 1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)</span><span class="sxs-lookup"><span data-stu-id="49370-252">Quotes: *Star Trek VI: The Undiscovered Country* &copy;1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)</span></span>

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="49370-253">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="49370-253">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

1. <span data-ttu-id="49370-254">Nella barra laterale della **soluzione** selezionare il `BlazorWebAssemblySignalRApp.Server` progetto.</span><span class="sxs-lookup"><span data-stu-id="49370-254">In the **Solution** sidebar, select the `BlazorWebAssemblySignalRApp.Server` project.</span></span> <span data-ttu-id="49370-255">Premere <kbd>⌘</kbd> + <kbd>↩</kbd> per eseguire l'app con debug o <kbd>⌥</kbd> + <kbd>⌘</kbd> + <kbd>↩</kbd> per eseguire l'app senza debug.</span><span class="sxs-lookup"><span data-stu-id="49370-255">Press <kbd>⌘</kbd>+<kbd>↩</kbd> to run the app with debugging or <kbd>⌥</kbd>+<kbd>⌘</kbd>+<kbd>↩</kbd> to run the app without debugging.</span></span>

1. <span data-ttu-id="49370-256">Copiare l'URL dalla barra degli indirizzi, aprire un'altra istanza o scheda del browser e incollare l'URL nella barra degli indirizzi.</span><span class="sxs-lookup"><span data-stu-id="49370-256">Copy the URL from the address bar, open another browser instance or tab, and paste the URL in the address bar.</span></span>

1. <span data-ttu-id="49370-257">Scegliere browser, immettere un nome e un messaggio e selezionare il pulsante per l'invio del messaggio.</span><span class="sxs-lookup"><span data-stu-id="49370-257">Choose either browser, enter a name and message, and select the button to send the message.</span></span> <span data-ttu-id="49370-258">Il nome e il messaggio vengono visualizzati immediatamente in entrambe le pagine:</span><span class="sxs-lookup"><span data-stu-id="49370-258">The name and message are displayed on both pages instantly:</span></span>

   ![::: NO-LOC (SignalR):::::: NO-LOC (Blazer)::: app di esempio aperta in due finestre del browser che mostrano i messaggi scambiati.](signalr-blazor/_static/signalr-blazor-finished.png)

   <span data-ttu-id="49370-260">Virgolette: *Star Trek VI: il paese non individuato* &copy; 1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)</span><span class="sxs-lookup"><span data-stu-id="49370-260">Quotes: *Star Trek VI: The Undiscovered Country* &copy;1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)</span></span>

# <a name="net-core-cli"></a>[<span data-ttu-id="49370-261">Interfaccia della riga di comando di .NET Core</span><span class="sxs-lookup"><span data-stu-id="49370-261">.NET Core CLI</span></span>](#tab/netcore-cli/)

1. <span data-ttu-id="49370-262">In una shell dei comandi dalla cartella della soluzione eseguire i comandi seguenti:</span><span class="sxs-lookup"><span data-stu-id="49370-262">In a command shell from the solution's folder, execute the following commands:</span></span>

   ```dotnetcli
   cd Server
   dotnet run
   ```

1. <span data-ttu-id="49370-263">Copiare l'URL dalla barra degli indirizzi, aprire un'altra istanza o scheda del browser e incollare l'URL nella barra degli indirizzi.</span><span class="sxs-lookup"><span data-stu-id="49370-263">Copy the URL from the address bar, open another browser instance or tab, and paste the URL in the address bar.</span></span>

1. <span data-ttu-id="49370-264">Scegliere browser, immettere un nome e un messaggio e selezionare il pulsante per l'invio del messaggio.</span><span class="sxs-lookup"><span data-stu-id="49370-264">Choose either browser, enter a name and message, and select the button to send the message.</span></span> <span data-ttu-id="49370-265">Il nome e il messaggio vengono visualizzati immediatamente in entrambe le pagine:</span><span class="sxs-lookup"><span data-stu-id="49370-265">The name and message are displayed on both pages instantly:</span></span>

   ![::: NO-LOC (SignalR):::::: NO-LOC (Blazer)::: app di esempio aperta in due finestre del browser che mostrano i messaggi scambiati.](signalr-blazor/_static/signalr-blazor-finished.png)

   <span data-ttu-id="49370-267">Virgolette: *Star Trek VI: il paese non individuato* &copy; 1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)</span><span class="sxs-lookup"><span data-stu-id="49370-267">Quotes: *Star Trek VI: The Undiscovered Country* &copy;1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)</span></span>

---

::: zone-end

::: zone pivot="server"

## <a name="create-a-blazor-server-app"></a><span data-ttu-id="49370-268">Creare un' Blazor Server app</span><span class="sxs-lookup"><span data-stu-id="49370-268">Create a Blazor Server app</span></span>

<span data-ttu-id="49370-269">Seguire le istruzioni per la scelta degli strumenti:</span><span class="sxs-lookup"><span data-stu-id="49370-269">Follow the guidance for your choice of tooling:</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="49370-270">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="49370-270">Visual Studio</span></span>](#tab/visual-studio)

::: moniker range=">= aspnetcore-5.0"

> [!NOTE]
> <span data-ttu-id="49370-271">Sono necessari Visual Studio 16,8 o versioni successive e .NET Core SDK 5.0.0 o versioni successive.</span><span class="sxs-lookup"><span data-stu-id="49370-271">Visual Studio 16.8 or later and .NET Core SDK 5.0.0 or later are required.</span></span>

::: moniker-end

::: moniker range="< aspnetcore-5.0"

> [!NOTE]
> <span data-ttu-id="49370-272">Sono necessari Visual Studio 16,6 o versioni successive e .NET Core SDK 3.1.300 o versioni successive.</span><span class="sxs-lookup"><span data-stu-id="49370-272">Visual Studio 16.6 or later and .NET Core SDK 3.1.300 or later are required.</span></span>

::: moniker-end

1. <span data-ttu-id="49370-273">Creare un nuovo progetto.</span><span class="sxs-lookup"><span data-stu-id="49370-273">Create a new project.</span></span>

1. <span data-ttu-id="49370-274">Selezionare **Blazor app** e fare clic su **Next (avanti**).</span><span class="sxs-lookup"><span data-stu-id="49370-274">Select **Blazor App** and select **Next**.</span></span>

1. <span data-ttu-id="49370-275">Digitare `BlazorServerSignalRApp` nel campo **nome progetto** .</span><span class="sxs-lookup"><span data-stu-id="49370-275">Type `BlazorServerSignalRApp` in the **Project name** field.</span></span> <span data-ttu-id="49370-276">Confermare che la voce relativa al **percorso** sia corretta o specificare un percorso per il progetto.</span><span class="sxs-lookup"><span data-stu-id="49370-276">Confirm the **Location** entry is correct or provide a location for the project.</span></span> <span data-ttu-id="49370-277">Selezionare **Crea**.</span><span class="sxs-lookup"><span data-stu-id="49370-277">Select **Create**.</span></span>

1. <span data-ttu-id="49370-278">Scegliere il modello di **Blazor Server app** .</span><span class="sxs-lookup"><span data-stu-id="49370-278">Choose the **Blazor Server App** template.</span></span>

1. <span data-ttu-id="49370-279">Selezionare **Crea**.</span><span class="sxs-lookup"><span data-stu-id="49370-279">Select **Create**.</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="49370-280">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="49370-280">Visual Studio Code</span></span>](#tab/visual-studio-code)

1. <span data-ttu-id="49370-281">In una shell dei comandi eseguire il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="49370-281">In a command shell, execute the following command:</span></span>

   ```dotnetcli
   dotnet new blazorserver -o BlazorServerSignalRApp
   ```

   <span data-ttu-id="49370-282">L' `-o|--output` opzione Crea una cartella per il progetto.</span><span class="sxs-lookup"><span data-stu-id="49370-282">The `-o|--output` option creates a folder for the project.</span></span> <span data-ttu-id="49370-283">Se è stata creata una cartella per il progetto e la shell dei comandi è aperta in tale cartella, omettere l' `-o|--output` opzione e il valore per creare il progetto.</span><span class="sxs-lookup"><span data-stu-id="49370-283">If you've created a folder for the project and the command shell is open in that folder, omit the `-o|--output` option and value to create the project.</span></span>

1. <span data-ttu-id="49370-284">In Visual Studio Code aprire la cartella del progetto dell'app.</span><span class="sxs-lookup"><span data-stu-id="49370-284">In Visual Studio Code, open the app's project folder.</span></span>

1. <span data-ttu-id="49370-285">Quando viene visualizzata la finestra di dialogo per aggiungere asset per compilare ed eseguire il debug dell'app, selezionare **Sì**.</span><span class="sxs-lookup"><span data-stu-id="49370-285">When the dialog appears to add assets to build and debug the app, select **Yes**.</span></span> <span data-ttu-id="49370-286">Visual Studio Code aggiunge automaticamente la `.vscode` cartella con `launch.json` `tasks.json` i file e generati.</span><span class="sxs-lookup"><span data-stu-id="49370-286">Visual Studio Code automatically adds the `.vscode` folder with generated `launch.json` and `tasks.json` files.</span></span> <span data-ttu-id="49370-287">Per informazioni sulla configurazione di asset VS Code nella `.vscode` cartella, vedere la guida del sistema operativo **Linux** in <xref:blazor/tooling> .</span><span class="sxs-lookup"><span data-stu-id="49370-287">For information on configuring VS Code assets in the `.vscode` folder, see the **Linux** operating system guidance in <xref:blazor/tooling>.</span></span>

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="49370-288">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="49370-288">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

1. <span data-ttu-id="49370-289">Installare la versione più recente di [Visual Studio per Mac](https://visualstudio.microsoft.com/vs/mac/) e seguire questa procedura:</span><span class="sxs-lookup"><span data-stu-id="49370-289">Install the latest version of [Visual Studio for Mac](https://visualstudio.microsoft.com/vs/mac/) and perform the following steps:</span></span>

1. <span data-ttu-id="49370-290">Selezionare **file**  >  **nuova soluzione** o creare un **nuovo** progetto dalla **finestra Start**.</span><span class="sxs-lookup"><span data-stu-id="49370-290">Select **File** > **New Solution** or create a **New** project from the **Start Window**.</span></span>

1. <span data-ttu-id="49370-291">Nella barra laterale selezionare app **Web e console**  >  .</span><span class="sxs-lookup"><span data-stu-id="49370-291">In the sidebar, select **Web and Console** > **App**.</span></span>

1. <span data-ttu-id="49370-292">Scegliere il modello di **Blazor Server app** .</span><span class="sxs-lookup"><span data-stu-id="49370-292">Choose the **Blazor Server App** template.</span></span> <span data-ttu-id="49370-293">Selezionare **Avanti**.</span><span class="sxs-lookup"><span data-stu-id="49370-293">Select **Next**.</span></span>

1. <span data-ttu-id="49370-294">Verificare che **l'autenticazione** sia impostata su **Nessuna autenticazione**.</span><span class="sxs-lookup"><span data-stu-id="49370-294">Confirm that **Authentication** is set to **No Authentication**.</span></span> <span data-ttu-id="49370-295">Selezionare **Avanti**.</span><span class="sxs-lookup"><span data-stu-id="49370-295">Select **Next**.</span></span>

1. <span data-ttu-id="49370-296">Nel campo **nome progetto** assegnare un nome all'app `BlazorServerSignalRApp` .</span><span class="sxs-lookup"><span data-stu-id="49370-296">In the **Project Name** field, name the app `BlazorServerSignalRApp`.</span></span> <span data-ttu-id="49370-297">Selezionare **Crea**.</span><span class="sxs-lookup"><span data-stu-id="49370-297">Select **Create**.</span></span>

   <span data-ttu-id="49370-298">Se viene visualizzato un messaggio per considerare attendibile il certificato di sviluppo, considerare attendibile il certificato e continuare.</span><span class="sxs-lookup"><span data-stu-id="49370-298">If a prompt appears to trust the development certificate, trust the certificate and continue.</span></span> <span data-ttu-id="49370-299">Per considerare attendibile il certificato, è necessario specificare le password dell'utente e del keychain.</span><span class="sxs-lookup"><span data-stu-id="49370-299">The user and keychain passwords are required to trust the certificate.</span></span>

1. <span data-ttu-id="49370-300">Aprire il progetto passando alla cartella del progetto e aprendo il file di soluzione del progetto ( `.sln` ).</span><span class="sxs-lookup"><span data-stu-id="49370-300">Open the project by navigating to the project folder and opening the project's solution file (`.sln`).</span></span>

# <a name="net-core-cli"></a>[<span data-ttu-id="49370-301">Interfaccia della riga di comando di .NET Core</span><span class="sxs-lookup"><span data-stu-id="49370-301">.NET Core CLI</span></span>](#tab/netcore-cli/)

<span data-ttu-id="49370-302">In una shell dei comandi eseguire il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="49370-302">In a command shell, execute the following command:</span></span>

```dotnetcli
dotnet new blazorserver -o BlazorServerSignalRApp
```

<span data-ttu-id="49370-303">L' `-o|--output` opzione Crea una cartella per il progetto.</span><span class="sxs-lookup"><span data-stu-id="49370-303">The `-o|--output` option creates a folder for the project.</span></span> <span data-ttu-id="49370-304">Se è stata creata una cartella per il progetto e la shell dei comandi è aperta in tale cartella, omettere l' `-o|--output` opzione e il valore per creare il progetto.</span><span class="sxs-lookup"><span data-stu-id="49370-304">If you've created a folder for the project and the command shell is open in that folder, omit the `-o|--output` option and value to create the project.</span></span>

---

## <a name="add-the-signalr-client-library"></a><span data-ttu-id="49370-305">Aggiungere la SignalR libreria client</span><span class="sxs-lookup"><span data-stu-id="49370-305">Add the SignalR client library</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="49370-306">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="49370-306">Visual Studio</span></span>](#tab/visual-studio/)

1. <span data-ttu-id="49370-307">In **Esplora soluzioni** fare clic con il pulsante destro del mouse sul `BlazorServerSignalRApp` progetto e scegliere **Gestisci pacchetti NuGet**.</span><span class="sxs-lookup"><span data-stu-id="49370-307">In **Solution Explorer**, right-click the `BlazorServerSignalRApp` project and select **Manage NuGet Packages**.</span></span>

1. <span data-ttu-id="49370-308">Nella finestra di dialogo **Gestisci pacchetti NuGet** verificare che l' **origine del pacchetto** sia impostata su `nuget.org` .</span><span class="sxs-lookup"><span data-stu-id="49370-308">In the **Manage NuGet Packages** dialog, confirm that the **Package source** is set to `nuget.org`.</span></span>

1. <span data-ttu-id="49370-309">Con **Sfoglia** selezionato, digitare `Microsoft.AspNetCore.SignalR.Client` nella casella di ricerca.</span><span class="sxs-lookup"><span data-stu-id="49370-309">With **Browse** selected, type `Microsoft.AspNetCore.SignalR.Client` in the search box.</span></span>

1. <span data-ttu-id="49370-310">Nei risultati della ricerca selezionare il [`Microsoft.AspNetCore.SignalR.Client`](https://www.nuget.org/packages/Microsoft.AspNetCore.SignalR.Client) pacchetto.</span><span class="sxs-lookup"><span data-stu-id="49370-310">In the search results, select the [`Microsoft.AspNetCore.SignalR.Client`](https://www.nuget.org/packages/Microsoft.AspNetCore.SignalR.Client) package.</span></span> <span data-ttu-id="49370-311">Impostare la versione in modo che corrisponda al Framework condiviso dell'app.</span><span class="sxs-lookup"><span data-stu-id="49370-311">Set the version to match the shared framework of the app.</span></span> <span data-ttu-id="49370-312">Selezionare **Installa**.</span><span class="sxs-lookup"><span data-stu-id="49370-312">Select **Install**.</span></span>

1. <span data-ttu-id="49370-313">Se viene visualizzata la finestra di dialogo **Anteprima modifiche** , selezionare **OK**.</span><span class="sxs-lookup"><span data-stu-id="49370-313">If the **Preview Changes** dialog appears, select **OK**.</span></span>

1. <span data-ttu-id="49370-314">Se viene visualizzata la finestra di dialogo **accettazione della licenza** , selezionare Accetto se **si accettano le** condizioni di licenza.</span><span class="sxs-lookup"><span data-stu-id="49370-314">If the **License Acceptance** dialog appears, select **I Accept** if you agree with the license terms.</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="49370-315">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="49370-315">Visual Studio Code</span></span>](#tab/visual-studio-code/)

<span data-ttu-id="49370-316">Nel **terminale integrato** (**visualizzare** il  >  **terminale** dalla barra degli strumenti) eseguire il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="49370-316">In the **Integrated Terminal** (**View** > **Terminal** from the toolbar), execute the following command:</span></span>

```dotnetcli
dotnet add package Microsoft.AspNetCore.SignalR.Client
```

<span data-ttu-id="49370-317">Per aggiungere una versione precedente del pacchetto, specificare l' `--version {VERSION}` opzione, dove il `{VERSION}` segnaposto è la versione del pacchetto da aggiungere.</span><span class="sxs-lookup"><span data-stu-id="49370-317">To add an earlier version of the package, supply the `--version {VERSION}` option, where the `{VERSION}` placeholder is the version of the package to add.</span></span>

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="49370-318">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="49370-318">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

1. <span data-ttu-id="49370-319">Nella barra laterale **soluzione** fare clic con il pulsante destro del mouse sul `BlazorServerSignalRApp` progetto e scegliere **Gestisci pacchetti NuGet**.</span><span class="sxs-lookup"><span data-stu-id="49370-319">In the **Solution** sidebar, right-click the `BlazorServerSignalRApp` project and select **Manage NuGet Packages**.</span></span>

1. <span data-ttu-id="49370-320">Nella finestra di dialogo **Gestisci pacchetti NuGet** verificare che l'elenco a discesa origine sia impostato su `nuget.org` .</span><span class="sxs-lookup"><span data-stu-id="49370-320">In the **Manage NuGet Packages** dialog, confirm that the source drop-down is set to `nuget.org`.</span></span>

1. <span data-ttu-id="49370-321">Con **Sfoglia** selezionato, digitare `Microsoft.AspNetCore.SignalR.Client` nella casella di ricerca.</span><span class="sxs-lookup"><span data-stu-id="49370-321">With **Browse** selected, type `Microsoft.AspNetCore.SignalR.Client` in the search box.</span></span>

1. <span data-ttu-id="49370-322">Nei risultati della ricerca selezionare la casella di controllo accanto al [`Microsoft.AspNetCore.SignalR.Client`](https://www.nuget.org/packages/Microsoft.AspNetCore.SignalR.Client) pacchetto.</span><span class="sxs-lookup"><span data-stu-id="49370-322">In the search results, select the check box next to the [`Microsoft.AspNetCore.SignalR.Client`](https://www.nuget.org/packages/Microsoft.AspNetCore.SignalR.Client) package.</span></span> <span data-ttu-id="49370-323">Impostare la versione in modo che corrisponda al Framework condiviso dell'app.</span><span class="sxs-lookup"><span data-stu-id="49370-323">Set the version to match the shared framework of the app.</span></span> <span data-ttu-id="49370-324">Selezionare **Aggiungi pacchetto**.</span><span class="sxs-lookup"><span data-stu-id="49370-324">Select **Add Package**.</span></span>

1. <span data-ttu-id="49370-325">Se viene visualizzata la finestra di dialogo **accettazione della licenza** , selezionare **Accetto** se si accettano le condizioni di licenza.</span><span class="sxs-lookup"><span data-stu-id="49370-325">If the **License Acceptance** dialog appears, select **Accept** if you agree with the license terms.</span></span>

# <a name="net-core-cli"></a>[<span data-ttu-id="49370-326">Interfaccia della riga di comando di .NET Core</span><span class="sxs-lookup"><span data-stu-id="49370-326">.NET Core CLI</span></span>](#tab/netcore-cli/)

<span data-ttu-id="49370-327">In una shell dei comandi dalla cartella del progetto, eseguire il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="49370-327">In a command shell from the project's folder, execute the following command:</span></span>

```dotnetcli
dotnet add package Microsoft.AspNetCore.SignalR.Client
```

<span data-ttu-id="49370-328">Per aggiungere una versione precedente del pacchetto, specificare l' `--version {VERSION}` opzione, dove il `{VERSION}` segnaposto è la versione del pacchetto da aggiungere.</span><span class="sxs-lookup"><span data-stu-id="49370-328">To add an earlier version of the package, supply the `--version {VERSION}` option, where the `{VERSION}` placeholder is the version of the package to add.</span></span>

---

::: moniker range="< aspnetcore-5.0"

## <a name="add-the-systemtextencodingsweb-package"></a><span data-ttu-id="49370-329">Aggiungere il pacchetto System. Text. Encodings. Web</span><span class="sxs-lookup"><span data-stu-id="49370-329">Add the System.Text.Encodings.Web package</span></span>

<span data-ttu-id="49370-330">*Questa sezione si applica solo alle app per ASP.NET Core versione 3. x.*</span><span class="sxs-lookup"><span data-stu-id="49370-330">*This section only applies to apps for ASP.NET Core version 3.x.*</span></span>

<span data-ttu-id="49370-331">A causa di un problema di risoluzione del pacchetto quando si usa [`System.Text.Json`](https://www.nuget.org/packages/System.Text.Json) 5. x in un'app ASP.NET Core 3. x, il progetto richiede un riferimento al pacchetto per [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web) .</span><span class="sxs-lookup"><span data-stu-id="49370-331">Due to a package resolution issue when using [`System.Text.Json`](https://www.nuget.org/packages/System.Text.Json) 5.x in an ASP.NET Core 3.x app, the project requires a package reference for [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web).</span></span> <span data-ttu-id="49370-332">Il problema sottostante verrà risolto in una versione futura della patch di .NET 5.</span><span class="sxs-lookup"><span data-stu-id="49370-332">The underlying issue will be resolved in a future patch release of .NET 5.</span></span> <span data-ttu-id="49370-333">Per altre informazioni, vedere [System.Text.Json definisce netcoreapp 3.0 senza dipendenze (DotNet/runtime #45560)](https://github.com/dotnet/runtime/issues/45560).</span><span class="sxs-lookup"><span data-stu-id="49370-333">For more information, see [System.Text.Json defines netcoreapp3.0 with no dependencies (dotnet/runtime #45560)](https://github.com/dotnet/runtime/issues/45560).</span></span>

<span data-ttu-id="49370-334">Per aggiungere [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web) al progetto, seguire le istruzioni per la scelta degli strumenti:</span><span class="sxs-lookup"><span data-stu-id="49370-334">To add [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web) to the project, follow the guidance for your choice of tooling:</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="49370-335">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="49370-335">Visual Studio</span></span>](#tab/visual-studio/)

1. <span data-ttu-id="49370-336">In **Esplora soluzioni** fare clic con il pulsante destro del mouse sul `BlazorServerSignalRApp` progetto e scegliere **Gestisci pacchetti NuGet**.</span><span class="sxs-lookup"><span data-stu-id="49370-336">In **Solution Explorer**, right-click the `BlazorServerSignalRApp` project and select **Manage NuGet Packages**.</span></span>

1. <span data-ttu-id="49370-337">Nella finestra di dialogo **Gestisci pacchetti NuGet** verificare che l' **origine del pacchetto** sia impostata su `nuget.org` .</span><span class="sxs-lookup"><span data-stu-id="49370-337">In the **Manage NuGet Packages** dialog, confirm that the **Package source** is set to `nuget.org`.</span></span>

1. <span data-ttu-id="49370-338">Con **Sfoglia** selezionato, digitare `System.Text.Encodings.Web` nella casella di ricerca.</span><span class="sxs-lookup"><span data-stu-id="49370-338">With **Browse** selected, type `System.Text.Encodings.Web` in the search box.</span></span>

1. <span data-ttu-id="49370-339">Nei risultati della ricerca selezionare il [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web) pacchetto.</span><span class="sxs-lookup"><span data-stu-id="49370-339">In the search results, select the [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web) package.</span></span> <span data-ttu-id="49370-340">Consente di selezionare la versione del pacchetto corrispondente al Framework condiviso in uso.</span><span class="sxs-lookup"><span data-stu-id="49370-340">Select the version of the package that matches the shared framework in use.</span></span> <span data-ttu-id="49370-341">Selezionare **Installa**.</span><span class="sxs-lookup"><span data-stu-id="49370-341">Select **Install**.</span></span>

1. <span data-ttu-id="49370-342">Se viene visualizzata la finestra di dialogo **Anteprima modifiche** , selezionare **OK**.</span><span class="sxs-lookup"><span data-stu-id="49370-342">If the **Preview Changes** dialog appears, select **OK**.</span></span>

1. <span data-ttu-id="49370-343">Se viene visualizzata la finestra di dialogo **accettazione della licenza** , selezionare Accetto se **si accettano le** condizioni di licenza.</span><span class="sxs-lookup"><span data-stu-id="49370-343">If the **License Acceptance** dialog appears, select **I Accept** if you agree with the license terms.</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="49370-344">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="49370-344">Visual Studio Code</span></span>](#tab/visual-studio-code/)

<span data-ttu-id="49370-345">Nel **terminale integrato** (**visualizzare** il > **terminale** dalla barra degli strumenti) eseguire i comandi seguenti:</span><span class="sxs-lookup"><span data-stu-id="49370-345">In the **Integrated Terminal** (**View** > **Terminal** from the toolbar), execute the following commands:</span></span>

```dotnetcli
dotnet add package System.Text.Encodings.Web
```

<span data-ttu-id="49370-346">Per aggiungere una versione precedente del pacchetto, specificare l' `--version {VERSION}` opzione, dove il `{VERSION}` segnaposto è la versione del pacchetto da aggiungere.</span><span class="sxs-lookup"><span data-stu-id="49370-346">To add an earlier version of the package, supply the `--version {VERSION}` option, where the `{VERSION}` placeholder is the version of the package to add.</span></span>

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="49370-347">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="49370-347">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

1. <span data-ttu-id="49370-348">Nella barra laterale **soluzione** fare clic con il pulsante destro del mouse sul `BlazorServerSignalRApp` progetto e scegliere **Gestisci pacchetti NuGet**.</span><span class="sxs-lookup"><span data-stu-id="49370-348">In the **Solution** sidebar, right-click the `BlazorServerSignalRApp` project and select **Manage NuGet Packages**.</span></span>

1. <span data-ttu-id="49370-349">Nella finestra di dialogo **Gestisci pacchetti NuGet** verificare che l'elenco a discesa origine sia impostato su `nuget.org` .</span><span class="sxs-lookup"><span data-stu-id="49370-349">In the **Manage NuGet Packages** dialog, confirm that the source drop-down is set to `nuget.org`.</span></span>

1. <span data-ttu-id="49370-350">Con **Sfoglia** selezionato, digitare `System.Text.Encodings.Web` nella casella di ricerca.</span><span class="sxs-lookup"><span data-stu-id="49370-350">With **Browse** selected, type `System.Text.Encodings.Web` in the search box.</span></span>

1. <span data-ttu-id="49370-351">Nei risultati della ricerca selezionare la casella di controllo accanto al [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web) pacchetto, selezionare la versione corretta del pacchetto corrispondente al Framework condiviso in uso e selezionare **Aggiungi pacchetto**.</span><span class="sxs-lookup"><span data-stu-id="49370-351">In the search results, select the check box next to the [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web) package, select the correct version of the package that matches the shared framework in use, and select **Add Package**.</span></span>

1. <span data-ttu-id="49370-352">Se viene visualizzata la finestra di dialogo **accettazione della licenza** , selezionare **Accetto** se si accettano le condizioni di licenza.</span><span class="sxs-lookup"><span data-stu-id="49370-352">If the **License Acceptance** dialog appears, select **Accept** if you agree with the license terms.</span></span>

# <a name="net-core-cli"></a>[<span data-ttu-id="49370-353">Interfaccia della riga di comando di .NET Core</span><span class="sxs-lookup"><span data-stu-id="49370-353">.NET Core CLI</span></span>](#tab/netcore-cli/)

<span data-ttu-id="49370-354">In una shell dei comandi dalla cartella del progetto, eseguire il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="49370-354">In a command shell from the project's folder, execute the following command:</span></span>

```dotnetcli
dotnet add package System.Text.Encodings.Web
```

<span data-ttu-id="49370-355">Per aggiungere una versione precedente del pacchetto, specificare l' `--version {VERSION}` opzione, dove il `{VERSION}` segnaposto è la versione del pacchetto da aggiungere.</span><span class="sxs-lookup"><span data-stu-id="49370-355">To add an earlier version of the package, supply the `--version {VERSION}` option, where the `{VERSION}` placeholder is the version of the package to add.</span></span>

---

::: moniker-end

## <a name="add-a-signalr-hub"></a><span data-ttu-id="49370-356">Aggiungere un SignalR Hub</span><span class="sxs-lookup"><span data-stu-id="49370-356">Add a SignalR hub</span></span>

<span data-ttu-id="49370-357">Creare una `Hubs` cartella (plurale) e aggiungere la `ChatHub` classe seguente ( `Hubs/ChatHub.cs` ):</span><span class="sxs-lookup"><span data-stu-id="49370-357">Create a `Hubs` (plural) folder and add the following `ChatHub` class (`Hubs/ChatHub.cs`):</span></span>

::: moniker range=">= aspnetcore-5.0"

[!code-csharp[](~/tutorials/signalr-blazor/samples/5.x/BlazorServerSignalRApp/Hubs/ChatHub.cs)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-csharp[](~/tutorials/signalr-blazor/samples/3.x/BlazorServerSignalRApp/Hubs/ChatHub.cs)]

::: moniker-end

## <a name="add-services-and-an-endpoint-for-the-signalr-hub"></a><span data-ttu-id="49370-358">Aggiungere servizi e un endpoint per l' SignalR Hub</span><span class="sxs-lookup"><span data-stu-id="49370-358">Add services and an endpoint for the SignalR hub</span></span>

1. <span data-ttu-id="49370-359">Aprire il file `Startup.cs`.</span><span class="sxs-lookup"><span data-stu-id="49370-359">Open the `Startup.cs` file.</span></span>

1. <span data-ttu-id="49370-360">Aggiungere gli spazi dei nomi per <xref:Microsoft.AspNetCore.ResponseCompression?displayProperty=fullName> e la `ChatHub` classe all'inizio del file:</span><span class="sxs-lookup"><span data-stu-id="49370-360">Add the namespaces for <xref:Microsoft.AspNetCore.ResponseCompression?displayProperty=fullName> and the `ChatHub` class to the top of the file:</span></span>

   ```csharp
   using Microsoft.AspNetCore.ResponseCompression;
   using BlazorServerSignalRApp.Server.Hubs;
   ```

::: moniker range=">= aspnetcore-5.0"

1. <span data-ttu-id="49370-361">Aggiungere i servizi middleware della compressione della risposta a `Startup.ConfigureServices` :</span><span class="sxs-lookup"><span data-stu-id="49370-361">Add Response Compression Middleware services to `Startup.ConfigureServices`:</span></span>

   [!code-csharp[](~/tutorials/signalr-blazor/samples/5.x/BlazorServerSignalRApp/Startup.cs?name=snippet_ConfigureServices&highlight=6-10)]

1. <span data-ttu-id="49370-362">In `Startup.Configure`:</span><span class="sxs-lookup"><span data-stu-id="49370-362">In `Startup.Configure`:</span></span>

   * <span data-ttu-id="49370-363">Usare il middleware della compressione della risposta nella parte superiore della configurazione della pipeline di elaborazione.</span><span class="sxs-lookup"><span data-stu-id="49370-363">Use Response Compression Middleware at the top of the processing pipeline's configuration.</span></span>
   * <span data-ttu-id="49370-364">Tra gli endpoint per il mapping dell' Blazor Hub e del fallback lato client, aggiungere un endpoint per l'hub.</span><span class="sxs-lookup"><span data-stu-id="49370-364">Between the endpoints for mapping the Blazor hub and the client-side fallback, add an endpoint for the hub.</span></span>

   [!code-csharp[](~/tutorials/signalr-blazor/samples/5.x/BlazorServerSignalRApp/Startup.cs?name=snippet_Configure&highlight=3,23)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

1. <span data-ttu-id="49370-365">Aggiungere i servizi middleware della compressione della risposta a `Startup.ConfigureServices` :</span><span class="sxs-lookup"><span data-stu-id="49370-365">Add Response Compression Middleware services to `Startup.ConfigureServices`:</span></span>

   [!code-csharp[](~/tutorials/signalr-blazor/samples/3.x/BlazorServerSignalRApp/Startup.cs?name=snippet_ConfigureServices&highlight=6-10)]

1. <span data-ttu-id="49370-366">In `Startup.Configure`:</span><span class="sxs-lookup"><span data-stu-id="49370-366">In `Startup.Configure`:</span></span>

   * <span data-ttu-id="49370-367">Usare il middleware della compressione della risposta nella parte superiore della configurazione della pipeline di elaborazione.</span><span class="sxs-lookup"><span data-stu-id="49370-367">Use Response Compression Middleware at the top of the processing pipeline's configuration.</span></span>
   * <span data-ttu-id="49370-368">Tra gli endpoint per il mapping dell' Blazor Hub e del fallback lato client, aggiungere un endpoint per l'hub.</span><span class="sxs-lookup"><span data-stu-id="49370-368">Between the endpoints for mapping the Blazor hub and the client-side fallback, add an endpoint for the hub.</span></span>

   [!code-csharp[](~/tutorials/signalr-blazor/samples/3.x/BlazorServerSignalRApp/Startup.cs?name=snippet_Configure&highlight=3,23)]

::: moniker-end

## <a name="add-razor-component-code-for-chat"></a><span data-ttu-id="49370-369">Aggiungi Razor codice componente per la chat</span><span class="sxs-lookup"><span data-stu-id="49370-369">Add Razor component code for chat</span></span>

1. <span data-ttu-id="49370-370">Aprire il file `Pages/Index.razor`.</span><span class="sxs-lookup"><span data-stu-id="49370-370">Open the `Pages/Index.razor` file.</span></span>

::: moniker range=">= aspnetcore-5.0"

1. <span data-ttu-id="49370-371">Sostituire il markup con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="49370-371">Replace the markup with the following code:</span></span>

   [!code-razor[](~/tutorials/signalr-blazor/samples/5.x/BlazorServerSignalRApp/Pages/Index.razor)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

1. <span data-ttu-id="49370-372">Sostituire il markup con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="49370-372">Replace the markup with the following code:</span></span>

   [!code-razor[](~/tutorials/signalr-blazor/samples/3.x/BlazorServerSignalRApp/Pages/Index.razor)]

::: moniker-end

## <a name="run-the-app"></a><span data-ttu-id="49370-373">Eseguire l'app</span><span class="sxs-lookup"><span data-stu-id="49370-373">Run the app</span></span>

<span data-ttu-id="49370-374">Seguire le istruzioni per gli strumenti:</span><span class="sxs-lookup"><span data-stu-id="49370-374">Follow the guidance for your tooling:</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="49370-375">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="49370-375">Visual Studio</span></span>](#tab/visual-studio)

1. <span data-ttu-id="49370-376">Premere <kbd>F5</kbd> per eseguire l'app con il debug o <kbd>CTRL</kbd> + <kbd>F5</kbd> per eseguire l'app senza debug.</span><span class="sxs-lookup"><span data-stu-id="49370-376">Press <kbd>F5</kbd> to run the app with debugging or <kbd>Ctrl</kbd>+<kbd>F5</kbd> to run the app without debugging.</span></span>

1. <span data-ttu-id="49370-377">Copiare l'URL dalla barra degli indirizzi, aprire un'altra istanza o scheda del browser e incollare l'URL nella barra degli indirizzi.</span><span class="sxs-lookup"><span data-stu-id="49370-377">Copy the URL from the address bar, open another browser instance or tab, and paste the URL in the address bar.</span></span>

1. <span data-ttu-id="49370-378">Scegliere browser, immettere un nome e un messaggio e selezionare il pulsante per l'invio del messaggio.</span><span class="sxs-lookup"><span data-stu-id="49370-378">Choose either browser, enter a name and message, and select the button to send the message.</span></span> <span data-ttu-id="49370-379">Il nome e il messaggio vengono visualizzati immediatamente in entrambe le pagine:</span><span class="sxs-lookup"><span data-stu-id="49370-379">The name and message are displayed on both pages instantly:</span></span>

   ![::: NO-LOC (SignalR):::::: NO-LOC (Blazer)::: app di esempio aperta in due finestre del browser che mostrano i messaggi scambiati.](signalr-blazor/_static/signalr-blazor-finished.png)

   <span data-ttu-id="49370-381">Virgolette: *Star Trek VI: il paese non individuato* &copy; 1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)</span><span class="sxs-lookup"><span data-stu-id="49370-381">Quotes: *Star Trek VI: The Undiscovered Country* &copy;1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="49370-382">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="49370-382">Visual Studio Code</span></span>](#tab/visual-studio-code)

1. <span data-ttu-id="49370-383">Premere <kbd>F5</kbd> per eseguire l'app con il debug o <kbd>CTRL</kbd> + <kbd>F5</kbd> per eseguire l'app senza debug.</span><span class="sxs-lookup"><span data-stu-id="49370-383">Press <kbd>F5</kbd> to run the app with debugging or <kbd>Ctrl</kbd>+<kbd>F5</kbd> to run the app without debugging.</span></span>

1. <span data-ttu-id="49370-384">Copiare l'URL dalla barra degli indirizzi, aprire un'altra istanza o scheda del browser e incollare l'URL nella barra degli indirizzi.</span><span class="sxs-lookup"><span data-stu-id="49370-384">Copy the URL from the address bar, open another browser instance or tab, and paste the URL in the address bar.</span></span>

1. <span data-ttu-id="49370-385">Scegliere browser, immettere un nome e un messaggio e selezionare il pulsante per l'invio del messaggio.</span><span class="sxs-lookup"><span data-stu-id="49370-385">Choose either browser, enter a name and message, and select the button to send the message.</span></span> <span data-ttu-id="49370-386">Il nome e il messaggio vengono visualizzati immediatamente in entrambe le pagine:</span><span class="sxs-lookup"><span data-stu-id="49370-386">The name and message are displayed on both pages instantly:</span></span>

   ![::: NO-LOC (SignalR):::::: NO-LOC (Blazer)::: app di esempio aperta in due finestre del browser che mostrano i messaggi scambiati.](signalr-blazor/_static/signalr-blazor-finished.png)

   <span data-ttu-id="49370-388">Virgolette: *Star Trek VI: il paese non individuato* &copy; 1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)</span><span class="sxs-lookup"><span data-stu-id="49370-388">Quotes: *Star Trek VI: The Undiscovered Country* &copy;1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)</span></span>

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="49370-389">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="49370-389">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

1. <span data-ttu-id="49370-390">Premere <kbd>⌘</kbd> + <kbd>↩</kbd> per eseguire l'app con debug o <kbd>⌥</kbd> + <kbd>⌘</kbd> + <kbd>↩</kbd> per eseguire l'app senza debug.</span><span class="sxs-lookup"><span data-stu-id="49370-390">Press <kbd>⌘</kbd>+<kbd>↩</kbd> to run the app with debugging or <kbd>⌥</kbd>+<kbd>⌘</kbd>+<kbd>↩</kbd> to run the app without debugging.</span></span>

1. <span data-ttu-id="49370-391">Copiare l'URL dalla barra degli indirizzi, aprire un'altra istanza o scheda del browser e incollare l'URL nella barra degli indirizzi.</span><span class="sxs-lookup"><span data-stu-id="49370-391">Copy the URL from the address bar, open another browser instance or tab, and paste the URL in the address bar.</span></span>

1. <span data-ttu-id="49370-392">Scegliere browser, immettere un nome e un messaggio e selezionare il pulsante per l'invio del messaggio.</span><span class="sxs-lookup"><span data-stu-id="49370-392">Choose either browser, enter a name and message, and select the button to send the message.</span></span> <span data-ttu-id="49370-393">Il nome e il messaggio vengono visualizzati immediatamente in entrambe le pagine:</span><span class="sxs-lookup"><span data-stu-id="49370-393">The name and message are displayed on both pages instantly:</span></span>

   ![::: NO-LOC (SignalR):::::: NO-LOC (Blazer)::: app di esempio aperta in due finestre del browser che mostrano i messaggi scambiati.](signalr-blazor/_static/signalr-blazor-finished.png)

   <span data-ttu-id="49370-395">Virgolette: *Star Trek VI: il paese non individuato* &copy; 1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)</span><span class="sxs-lookup"><span data-stu-id="49370-395">Quotes: *Star Trek VI: The Undiscovered Country* &copy;1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)</span></span>

# <a name="net-core-cli"></a>[<span data-ttu-id="49370-396">Interfaccia della riga di comando di .NET Core</span><span class="sxs-lookup"><span data-stu-id="49370-396">.NET Core CLI</span></span>](#tab/netcore-cli/)

1. <span data-ttu-id="49370-397">In una shell dei comandi dalla cartella del progetto, eseguire i comandi seguenti:</span><span class="sxs-lookup"><span data-stu-id="49370-397">In a command shell from the project's folder, execute the following commands:</span></span>

   ```dotnetcli
   dotnet run
   ```

1. <span data-ttu-id="49370-398">Copiare l'URL dalla barra degli indirizzi, aprire un'altra istanza o scheda del browser e incollare l'URL nella barra degli indirizzi.</span><span class="sxs-lookup"><span data-stu-id="49370-398">Copy the URL from the address bar, open another browser instance or tab, and paste the URL in the address bar.</span></span>

1. <span data-ttu-id="49370-399">Scegliere browser, immettere un nome e un messaggio e selezionare il pulsante per l'invio del messaggio.</span><span class="sxs-lookup"><span data-stu-id="49370-399">Choose either browser, enter a name and message, and select the button to send the message.</span></span> <span data-ttu-id="49370-400">Il nome e il messaggio vengono visualizzati immediatamente in entrambe le pagine:</span><span class="sxs-lookup"><span data-stu-id="49370-400">The name and message are displayed on both pages instantly:</span></span>

   ![::: NO-LOC (SignalR):::::: NO-LOC (Blazer)::: app di esempio aperta in due finestre del browser che mostrano i messaggi scambiati.](signalr-blazor/_static/signalr-blazor-finished.png)

   <span data-ttu-id="49370-402">Virgolette: *Star Trek VI: il paese non individuato* &copy; 1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)</span><span class="sxs-lookup"><span data-stu-id="49370-402">Quotes: *Star Trek VI: The Undiscovered Country* &copy;1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)</span></span>

---

::: zone-end

## <a name="next-steps"></a><span data-ttu-id="49370-403">Passaggi successivi</span><span class="sxs-lookup"><span data-stu-id="49370-403">Next steps</span></span>

<span data-ttu-id="49370-404">In questa esercitazione sono state illustrate le procedure per:</span><span class="sxs-lookup"><span data-stu-id="49370-404">In this tutorial, you learned how to:</span></span>

> [!div class="checklist"]
> * <span data-ttu-id="49370-405">Creazione di un Blazor progetto</span><span class="sxs-lookup"><span data-stu-id="49370-405">Create a Blazor project</span></span>
> * <span data-ttu-id="49370-406">Aggiungere la SignalR libreria client</span><span class="sxs-lookup"><span data-stu-id="49370-406">Add the SignalR client library</span></span>
> * <span data-ttu-id="49370-407">Aggiungere un SignalR Hub</span><span class="sxs-lookup"><span data-stu-id="49370-407">Add a SignalR hub</span></span>
> * <span data-ttu-id="49370-408">Aggiungere SignalR Servizi e un endpoint per l' SignalR Hub</span><span class="sxs-lookup"><span data-stu-id="49370-408">Add SignalR services and an endpoint for the SignalR hub</span></span>
> * <span data-ttu-id="49370-409">Aggiungi Razor codice componente per la chat</span><span class="sxs-lookup"><span data-stu-id="49370-409">Add Razor component code for chat</span></span>

<span data-ttu-id="49370-410">Per altre informazioni sulla creazione di Blazor app, vedere la Blazor documentazione:</span><span class="sxs-lookup"><span data-stu-id="49370-410">To learn more about building Blazor apps, see the Blazor documentation:</span></span>

> [!div class="nextstepaction"]
> <span data-ttu-id="49370-411"><xref:blazor/index>
> [Autenticazione del token di porta con Identity Server, WebSocket ed eventi di Server-Sent](xref:signalr/authn-and-authz#bearer-token-authentication)</span><span class="sxs-lookup"><span data-stu-id="49370-411"><xref:blazor/index>
[Bearer token authentication with Identity Server, WebSockets, and Server-Sent Events](xref:signalr/authn-and-authz#bearer-token-authentication)</span></span>

## <a name="additional-resources"></a><span data-ttu-id="49370-412">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="49370-412">Additional resources</span></span>

* <xref:signalr/introduction>
* [<span data-ttu-id="49370-413">SignalR negoziazione tra le origini per l'autenticazione</span><span class="sxs-lookup"><span data-stu-id="49370-413">SignalR cross-origin negotiation for authentication</span></span>](xref:blazor/fundamentals/signalr#signalr-cross-origin-negotiation-for-authentication)
* <xref:blazor/debug>
* <xref:blazor/security/server/threat-mitigation>
