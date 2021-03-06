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
ms.openlocfilehash: 50e5f2d5b1f9d0bf229bc57e6a1f599f9574b27c
ms.sourcegitcommit: 1436bd4d70937d6ec3140da56d96caab33c4320b
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/06/2021
ms.locfileid: "102395227"
---
# <a name="use-aspnet-core-signalr-with-blazor"></a><span data-ttu-id="a9d28-103">Usare ASP.NET Core SignalR con Blazor</span><span class="sxs-lookup"><span data-stu-id="a9d28-103">Use ASP.NET Core SignalR with Blazor</span></span>

<span data-ttu-id="a9d28-104">Questa esercitazione illustra le nozioni di base per la creazione di un'app in tempo reale usando SignalR con Blazor .</span><span class="sxs-lookup"><span data-stu-id="a9d28-104">This tutorial teaches the basics of building a real-time app using SignalR with Blazor.</span></span> <span data-ttu-id="a9d28-105">Si apprenderà come:</span><span class="sxs-lookup"><span data-stu-id="a9d28-105">You learn how to:</span></span>

> [!div class="checklist"]
> * <span data-ttu-id="a9d28-106">Creazione di un Blazor progetto</span><span class="sxs-lookup"><span data-stu-id="a9d28-106">Create a Blazor project</span></span>
> * <span data-ttu-id="a9d28-107">Aggiungere la SignalR libreria client</span><span class="sxs-lookup"><span data-stu-id="a9d28-107">Add the SignalR client library</span></span>
> * <span data-ttu-id="a9d28-108">Aggiungere un SignalR Hub</span><span class="sxs-lookup"><span data-stu-id="a9d28-108">Add a SignalR hub</span></span>
> * <span data-ttu-id="a9d28-109">Aggiungere SignalR Servizi e un endpoint per l' SignalR Hub</span><span class="sxs-lookup"><span data-stu-id="a9d28-109">Add SignalR services and an endpoint for the SignalR hub</span></span>
> * <span data-ttu-id="a9d28-110">Aggiungi Razor codice componente per la chat</span><span class="sxs-lookup"><span data-stu-id="a9d28-110">Add Razor component code for chat</span></span>

<span data-ttu-id="a9d28-111">Al termine di questa esercitazione, si disporrà di un'app di chat funzionante.</span><span class="sxs-lookup"><span data-stu-id="a9d28-111">At the end of this tutorial, you'll have a working chat app.</span></span>

<span data-ttu-id="a9d28-112">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/tutorials/signalr-blazor/samples/) ([procedura per il download](xref:index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="a9d28-112">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/tutorials/signalr-blazor/samples/) ([how to download](xref:index#how-to-download-a-sample))</span></span>

## <a name="prerequisites"></a><span data-ttu-id="a9d28-113">Prerequisiti</span><span class="sxs-lookup"><span data-stu-id="a9d28-113">Prerequisites</span></span>

::: moniker range=">= aspnetcore-5.0"

# <a name="visual-studio"></a>[<span data-ttu-id="a9d28-114">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="a9d28-114">Visual Studio</span></span>](#tab/visual-studio)

* <span data-ttu-id="a9d28-115">[Visual Studio 2019 16,8 o versione successiva](https://visualstudio.microsoft.com/downloads/?utm_medium=microsoft&utm_source=docs.microsoft.com&utm_campaign=inline+link&utm_content=download+vs2019) con il carico di lavoro **sviluppo di ASP.NET e Web**</span><span class="sxs-lookup"><span data-stu-id="a9d28-115">[Visual Studio 2019 16.8 or later](https://visualstudio.microsoft.com/downloads/?utm_medium=microsoft&utm_source=docs.microsoft.com&utm_campaign=inline+link&utm_content=download+vs2019) with the **ASP.NET and web development** workload</span></span>
* [!INCLUDE [.NET Core 5.0 SDK](~/includes/5.0-SDK.md)]

# <a name="visual-studio-code"></a>[<span data-ttu-id="a9d28-116">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="a9d28-116">Visual Studio Code</span></span>](#tab/visual-studio-code)

[!INCLUDE[](~/includes/net-core-prereqs-vsc-5.0.md)]

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="a9d28-117">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="a9d28-117">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

* [<span data-ttu-id="a9d28-118">Visual Studio per Mac versione 8,8 o successiva</span><span class="sxs-lookup"><span data-stu-id="a9d28-118">Visual Studio for Mac version 8.8 or later</span></span>](https://visualstudio.microsoft.com/vs/mac/)
* [!INCLUDE [.NET Core 5.0 SDK](~/includes/5.0-SDK.md)]

# <a name="net-core-cli"></a>[<span data-ttu-id="a9d28-119">Interfaccia della riga di comando di .NET Core</span><span class="sxs-lookup"><span data-stu-id="a9d28-119">.NET Core CLI</span></span>](#tab/netcore-cli/)

[!INCLUDE[](~/includes/5.0-SDK.md)]

---

::: moniker-end

::: moniker range="< aspnetcore-5.0"

# <a name="visual-studio"></a>[<span data-ttu-id="a9d28-120">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="a9d28-120">Visual Studio</span></span>](#tab/visual-studio)

* <span data-ttu-id="a9d28-121">[Visual Studio 2019 16,6 o versione successiva](https://visualstudio.microsoft.com/downloads/?utm_medium=microsoft&utm_source=docs.microsoft.com&utm_campaign=inline+link&utm_content=download+vs2019) con il carico di lavoro **sviluppo di ASP.NET e Web**</span><span class="sxs-lookup"><span data-stu-id="a9d28-121">[Visual Studio 2019 16.6 or later](https://visualstudio.microsoft.com/downloads/?utm_medium=microsoft&utm_source=docs.microsoft.com&utm_campaign=inline+link&utm_content=download+vs2019) with the **ASP.NET and web development** workload</span></span>
* [!INCLUDE [.NET Core 3.1 SDK](~/includes/3.1-SDK.md)]

# <a name="visual-studio-code"></a>[<span data-ttu-id="a9d28-122">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="a9d28-122">Visual Studio Code</span></span>](#tab/visual-studio-code)

[!INCLUDE[](~/includes/net-core-prereqs-vsc-3.1.md)]

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="a9d28-123">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="a9d28-123">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

* [<span data-ttu-id="a9d28-124">Visual Studio per Mac versione 8,6 o successiva</span><span class="sxs-lookup"><span data-stu-id="a9d28-124">Visual Studio for Mac version 8.6 or later</span></span>](https://visualstudio.microsoft.com/vs/mac/)
* [!INCLUDE [.NET Core 3.1 SDK](~/includes/3.1-SDK.md)]

# <a name="net-core-cli"></a>[<span data-ttu-id="a9d28-125">Interfaccia della riga di comando di .NET Core</span><span class="sxs-lookup"><span data-stu-id="a9d28-125">.NET Core CLI</span></span>](#tab/netcore-cli/)

[!INCLUDE[](~/includes/3.1-SDK.md)]

---

::: moniker-end

::: zone pivot="webassembly"

## <a name="create-a-hosted-blazor-webassembly-app"></a><span data-ttu-id="a9d28-126">Creare un'app ospitata Blazor WebAssembly</span><span class="sxs-lookup"><span data-stu-id="a9d28-126">Create a hosted Blazor WebAssembly app</span></span>

<span data-ttu-id="a9d28-127">Seguire le istruzioni per la scelta degli strumenti:</span><span class="sxs-lookup"><span data-stu-id="a9d28-127">Follow the guidance for your choice of tooling:</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="a9d28-128">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="a9d28-128">Visual Studio</span></span>](#tab/visual-studio)

::: moniker range=">= aspnetcore-5.0"

> [!NOTE]
> <span data-ttu-id="a9d28-129">Sono necessari Visual Studio 16,8 o versioni successive e .NET Core SDK 5.0.0 o versioni successive.</span><span class="sxs-lookup"><span data-stu-id="a9d28-129">Visual Studio 16.8 or later and .NET Core SDK 5.0.0 or later are required.</span></span>

::: moniker-end

::: moniker range="< aspnetcore-5.0"

> [!NOTE]
> <span data-ttu-id="a9d28-130">Sono necessari Visual Studio 16,6 o versioni successive e .NET Core SDK 3.1.300 o versioni successive.</span><span class="sxs-lookup"><span data-stu-id="a9d28-130">Visual Studio 16.6 or later and .NET Core SDK 3.1.300 or later are required.</span></span>

::: moniker-end

1. <span data-ttu-id="a9d28-131">Creare un nuovo progetto.</span><span class="sxs-lookup"><span data-stu-id="a9d28-131">Create a new project.</span></span>

1. <span data-ttu-id="a9d28-132">Selezionare **Blazor app** e fare clic su **Next (avanti**).</span><span class="sxs-lookup"><span data-stu-id="a9d28-132">Select **Blazor App** and select **Next**.</span></span>

1. <span data-ttu-id="a9d28-133">Digitare `BlazorWebAssemblySignalRApp` nel campo **nome progetto** .</span><span class="sxs-lookup"><span data-stu-id="a9d28-133">Type `BlazorWebAssemblySignalRApp` in the **Project name** field.</span></span> <span data-ttu-id="a9d28-134">Confermare che la voce relativa al **percorso** sia corretta o specificare un percorso per il progetto.</span><span class="sxs-lookup"><span data-stu-id="a9d28-134">Confirm the **Location** entry is correct or provide a location for the project.</span></span> <span data-ttu-id="a9d28-135">Selezionare **Crea**.</span><span class="sxs-lookup"><span data-stu-id="a9d28-135">Select **Create**.</span></span>

1. <span data-ttu-id="a9d28-136">Scegliere il modello di **Blazor WebAssembly app** .</span><span class="sxs-lookup"><span data-stu-id="a9d28-136">Choose the **Blazor WebAssembly App** template.</span></span>

1. <span data-ttu-id="a9d28-137">In **Avanzate** selezionare la casella di controllo **ASP.NET Core Hosted** .</span><span class="sxs-lookup"><span data-stu-id="a9d28-137">Under **Advanced**, select the **ASP.NET Core hosted** check box.</span></span>

1. <span data-ttu-id="a9d28-138">Selezionare **Crea**.</span><span class="sxs-lookup"><span data-stu-id="a9d28-138">Select **Create**.</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="a9d28-139">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="a9d28-139">Visual Studio Code</span></span>](#tab/visual-studio-code)

1. <span data-ttu-id="a9d28-140">In una shell dei comandi eseguire il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="a9d28-140">In a command shell, execute the following command:</span></span>

   ```dotnetcli
   dotnet new blazorwasm -ho -o BlazorWebAssemblySignalRApp
   ```

   <span data-ttu-id="a9d28-141">L' `-ho|--hosted` opzione Crea una soluzione ospitata Blazor WebAssembly .</span><span class="sxs-lookup"><span data-stu-id="a9d28-141">The `-ho|--hosted` option creates a hosted Blazor WebAssembly solution.</span></span> <span data-ttu-id="a9d28-142">Per informazioni sulla configurazione di asset VS Code nella `.vscode` cartella, vedere la guida del sistema operativo **Linux** in <xref:blazor/tooling> .</span><span class="sxs-lookup"><span data-stu-id="a9d28-142">For information on configuring VS Code assets in the `.vscode` folder, see the **Linux** operating system guidance in <xref:blazor/tooling>.</span></span>

   <span data-ttu-id="a9d28-143">L' `-o|--output` opzione Crea una cartella per la soluzione.</span><span class="sxs-lookup"><span data-stu-id="a9d28-143">The `-o|--output` option creates a folder for the solution.</span></span> <span data-ttu-id="a9d28-144">Se è stata creata una cartella per la soluzione e la shell dei comandi è aperta in tale cartella, omettere l' `-o|--output` opzione e il valore per creare la soluzione.</span><span class="sxs-lookup"><span data-stu-id="a9d28-144">If you've created a folder for the solution and the command shell is open in that folder, omit the `-o|--output` option and value to create the solution.</span></span>

1. <span data-ttu-id="a9d28-145">In Visual Studio Code aprire la cartella del progetto dell'app.</span><span class="sxs-lookup"><span data-stu-id="a9d28-145">In Visual Studio Code, open the app's project folder.</span></span>

1. <span data-ttu-id="a9d28-146">Quando viene visualizzata la finestra di dialogo per aggiungere asset per compilare ed eseguire il debug dell'app, selezionare **Sì**.</span><span class="sxs-lookup"><span data-stu-id="a9d28-146">When the dialog appears to add assets to build and debug the app, select **Yes**.</span></span> <span data-ttu-id="a9d28-147">Visual Studio Code aggiunge automaticamente la `.vscode` cartella con `launch.json` `tasks.json` i file e generati.</span><span class="sxs-lookup"><span data-stu-id="a9d28-147">Visual Studio Code automatically adds the `.vscode` folder with generated `launch.json` and `tasks.json` files.</span></span>

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="a9d28-148">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="a9d28-148">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

1. <span data-ttu-id="a9d28-149">Installare la versione più recente di [Visual Studio per Mac](https://visualstudio.microsoft.com/vs/mac/) e seguire questa procedura:</span><span class="sxs-lookup"><span data-stu-id="a9d28-149">Install the latest version of [Visual Studio for Mac](https://visualstudio.microsoft.com/vs/mac/) and perform the following steps:</span></span>

1. <span data-ttu-id="a9d28-150">Selezionare **file**  >  **nuova soluzione** o creare un **nuovo** progetto dalla **finestra Start**.</span><span class="sxs-lookup"><span data-stu-id="a9d28-150">Select **File** > **New Solution** or create a **New** project from the **Start Window**.</span></span>

1. <span data-ttu-id="a9d28-151">Nella barra laterale selezionare app **Web e console**  >  .</span><span class="sxs-lookup"><span data-stu-id="a9d28-151">In the sidebar, select **Web and Console** > **App**.</span></span>

1. <span data-ttu-id="a9d28-152">Scegliere il modello di **Blazor WebAssembly app** .</span><span class="sxs-lookup"><span data-stu-id="a9d28-152">Choose the **Blazor WebAssembly App** template.</span></span> <span data-ttu-id="a9d28-153">Selezionare **Avanti**.</span><span class="sxs-lookup"><span data-stu-id="a9d28-153">Select **Next**.</span></span>

1. <span data-ttu-id="a9d28-154">Verificare che **l'autenticazione** sia impostata su **Nessuna autenticazione**.</span><span class="sxs-lookup"><span data-stu-id="a9d28-154">Confirm that **Authentication** is set to **No Authentication**.</span></span> <span data-ttu-id="a9d28-155">Selezionare la casella di controllo **ASP.NET Core Hosted** .</span><span class="sxs-lookup"><span data-stu-id="a9d28-155">Select the **ASP.NET Core Hosted** check box.</span></span> <span data-ttu-id="a9d28-156">Selezionare **Avanti**.</span><span class="sxs-lookup"><span data-stu-id="a9d28-156">Select **Next**.</span></span>

1. <span data-ttu-id="a9d28-157">Nel campo **nome progetto** assegnare un nome all'app `BlazorWebAssemblySignalRApp` .</span><span class="sxs-lookup"><span data-stu-id="a9d28-157">In the **Project Name** field, name the app `BlazorWebAssemblySignalRApp`.</span></span> <span data-ttu-id="a9d28-158">Selezionare **Crea**.</span><span class="sxs-lookup"><span data-stu-id="a9d28-158">Select **Create**.</span></span>

   <span data-ttu-id="a9d28-159">Se viene visualizzato un messaggio per considerare attendibile il certificato di sviluppo, considerare attendibile il certificato e continuare.</span><span class="sxs-lookup"><span data-stu-id="a9d28-159">If a prompt appears to trust the development certificate, trust the certificate and continue.</span></span> <span data-ttu-id="a9d28-160">Per considerare attendibile il certificato, è necessario specificare le password dell'utente e del keychain.</span><span class="sxs-lookup"><span data-stu-id="a9d28-160">The user and keychain passwords are required to trust the certificate.</span></span>

1. <span data-ttu-id="a9d28-161">Aprire il progetto passando alla cartella del progetto e aprendo il file di soluzione del progetto ( `.sln` ).</span><span class="sxs-lookup"><span data-stu-id="a9d28-161">Open the project by navigating to the project folder and opening the project's solution file (`.sln`).</span></span>

# <a name="net-core-cli"></a>[<span data-ttu-id="a9d28-162">Interfaccia della riga di comando di .NET Core</span><span class="sxs-lookup"><span data-stu-id="a9d28-162">.NET Core CLI</span></span>](#tab/netcore-cli/)

<span data-ttu-id="a9d28-163">In una shell dei comandi eseguire il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="a9d28-163">In a command shell, execute the following command:</span></span>

```dotnetcli
dotnet new blazorwasm -ho -o BlazorWebAssemblySignalRApp
```

<span data-ttu-id="a9d28-164">L' `-ho|--hosted` opzione Crea una soluzione ospitata Blazor WebAssembly .</span><span class="sxs-lookup"><span data-stu-id="a9d28-164">The `-ho|--hosted` option creates a hosted Blazor WebAssembly solution.</span></span>

<span data-ttu-id="a9d28-165">L' `-o|--output` opzione Crea una cartella per la soluzione.</span><span class="sxs-lookup"><span data-stu-id="a9d28-165">The `-o|--output` option creates a folder for the solution.</span></span> <span data-ttu-id="a9d28-166">Se è stata creata una cartella per la soluzione e la shell dei comandi è aperta in tale cartella, omettere l' `-o|--output` opzione e il valore per creare la soluzione.</span><span class="sxs-lookup"><span data-stu-id="a9d28-166">If you've created a folder for the solution and the command shell is open in that folder, omit the `-o|--output` option and value to create the solution.</span></span>

---

## <a name="add-the-signalr-client-library"></a><span data-ttu-id="a9d28-167">Aggiungere la SignalR libreria client</span><span class="sxs-lookup"><span data-stu-id="a9d28-167">Add the SignalR client library</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="a9d28-168">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="a9d28-168">Visual Studio</span></span>](#tab/visual-studio/)

1. <span data-ttu-id="a9d28-169">In **Esplora soluzioni** fare clic con il pulsante destro del mouse sul `BlazorWebAssemblySignalRApp.Client` progetto e scegliere **Gestisci pacchetti NuGet**.</span><span class="sxs-lookup"><span data-stu-id="a9d28-169">In **Solution Explorer**, right-click the `BlazorWebAssemblySignalRApp.Client` project and select **Manage NuGet Packages**.</span></span>

1. <span data-ttu-id="a9d28-170">Nella finestra di dialogo **Gestisci pacchetti NuGet** verificare che l' **origine del pacchetto** sia impostata su `nuget.org` .</span><span class="sxs-lookup"><span data-stu-id="a9d28-170">In the **Manage NuGet Packages** dialog, confirm that the **Package source** is set to `nuget.org`.</span></span>

1. <span data-ttu-id="a9d28-171">Con **Sfoglia** selezionato, digitare `Microsoft.AspNetCore.SignalR.Client` nella casella di ricerca.</span><span class="sxs-lookup"><span data-stu-id="a9d28-171">With **Browse** selected, type `Microsoft.AspNetCore.SignalR.Client` in the search box.</span></span>

1. <span data-ttu-id="a9d28-172">Nei risultati della ricerca selezionare il [`Microsoft.AspNetCore.SignalR.Client`](https://www.nuget.org/packages/Microsoft.AspNetCore.SignalR.Client) pacchetto.</span><span class="sxs-lookup"><span data-stu-id="a9d28-172">In the search results, select the [`Microsoft.AspNetCore.SignalR.Client`](https://www.nuget.org/packages/Microsoft.AspNetCore.SignalR.Client) package.</span></span> <span data-ttu-id="a9d28-173">Impostare la versione in modo che corrisponda al Framework condiviso dell'app.</span><span class="sxs-lookup"><span data-stu-id="a9d28-173">Set the version to match the shared framework of the app.</span></span> <span data-ttu-id="a9d28-174">Selezionare **Installa**.</span><span class="sxs-lookup"><span data-stu-id="a9d28-174">Select **Install**.</span></span>

1. <span data-ttu-id="a9d28-175">Se viene visualizzata la finestra di dialogo **Anteprima modifiche** , selezionare **OK**.</span><span class="sxs-lookup"><span data-stu-id="a9d28-175">If the **Preview Changes** dialog appears, select **OK**.</span></span>

1. <span data-ttu-id="a9d28-176">Se viene visualizzata la finestra di dialogo **accettazione della licenza** , selezionare Accetto se **si accettano le** condizioni di licenza.</span><span class="sxs-lookup"><span data-stu-id="a9d28-176">If the **License Acceptance** dialog appears, select **I Accept** if you agree with the license terms.</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="a9d28-177">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="a9d28-177">Visual Studio Code</span></span>](#tab/visual-studio-code/)

<span data-ttu-id="a9d28-178">Nel **terminale integrato** (**visualizzare** il  >  **terminale** dalla barra degli strumenti) eseguire il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="a9d28-178">In the **Integrated Terminal** (**View** > **Terminal** from the toolbar), execute the following command:</span></span>

```dotnetcli
dotnet add Client package Microsoft.AspNetCore.SignalR.Client
```

<span data-ttu-id="a9d28-179">Per aggiungere una versione precedente del pacchetto, specificare l' `--version {VERSION}` opzione, dove il `{VERSION}` segnaposto è la versione del pacchetto da aggiungere.</span><span class="sxs-lookup"><span data-stu-id="a9d28-179">To add an earlier version of the package, supply the `--version {VERSION}` option, where the `{VERSION}` placeholder is the version of the package to add.</span></span>

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="a9d28-180">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="a9d28-180">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

1. <span data-ttu-id="a9d28-181">Nella barra laterale **soluzione** fare clic con il pulsante destro del mouse sul `BlazorWebAssemblySignalRApp.Client` progetto e scegliere **Gestisci pacchetti NuGet**.</span><span class="sxs-lookup"><span data-stu-id="a9d28-181">In the **Solution** sidebar, right-click the `BlazorWebAssemblySignalRApp.Client` project and select **Manage NuGet Packages**.</span></span>

1. <span data-ttu-id="a9d28-182">Nella finestra di dialogo **Gestisci pacchetti NuGet** verificare che l'elenco a discesa origine sia impostato su `nuget.org` .</span><span class="sxs-lookup"><span data-stu-id="a9d28-182">In the **Manage NuGet Packages** dialog, confirm that the source drop-down is set to `nuget.org`.</span></span>

1. <span data-ttu-id="a9d28-183">Con **Sfoglia** selezionato, digitare `Microsoft.AspNetCore.SignalR.Client` nella casella di ricerca.</span><span class="sxs-lookup"><span data-stu-id="a9d28-183">With **Browse** selected, type `Microsoft.AspNetCore.SignalR.Client` in the search box.</span></span>

1. <span data-ttu-id="a9d28-184">Nei risultati della ricerca selezionare la casella di controllo accanto al [`Microsoft.AspNetCore.SignalR.Client`](https://www.nuget.org/packages/Microsoft.AspNetCore.SignalR.Client) pacchetto.</span><span class="sxs-lookup"><span data-stu-id="a9d28-184">In the search results, select the check box next to the [`Microsoft.AspNetCore.SignalR.Client`](https://www.nuget.org/packages/Microsoft.AspNetCore.SignalR.Client) package.</span></span> <span data-ttu-id="a9d28-185">Impostare la versione in modo che corrisponda al Framework condiviso dell'app.</span><span class="sxs-lookup"><span data-stu-id="a9d28-185">Set the version to match the shared framework of the app.</span></span> <span data-ttu-id="a9d28-186">Selezionare **Aggiungi pacchetto**.</span><span class="sxs-lookup"><span data-stu-id="a9d28-186">Select **Add Package**.</span></span>

1. <span data-ttu-id="a9d28-187">Se viene visualizzata la finestra di dialogo **accettazione della licenza** , selezionare **Accetto** se si accettano le condizioni di licenza.</span><span class="sxs-lookup"><span data-stu-id="a9d28-187">If the **License Acceptance** dialog appears, select **Accept** if you agree with the license terms.</span></span>

# <a name="net-core-cli"></a>[<span data-ttu-id="a9d28-188">Interfaccia della riga di comando di .NET Core</span><span class="sxs-lookup"><span data-stu-id="a9d28-188">.NET Core CLI</span></span>](#tab/netcore-cli/)

<span data-ttu-id="a9d28-189">In una shell dei comandi dalla cartella della soluzione, eseguire il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="a9d28-189">In a command shell from the solution's folder, execute the following command:</span></span>

```dotnetcli
dotnet add Client package Microsoft.AspNetCore.SignalR.Client
```

<span data-ttu-id="a9d28-190">Per aggiungere una versione precedente del pacchetto, specificare l' `--version {VERSION}` opzione, dove il `{VERSION}` segnaposto è la versione del pacchetto da aggiungere.</span><span class="sxs-lookup"><span data-stu-id="a9d28-190">To add an earlier version of the package, supply the `--version {VERSION}` option, where the `{VERSION}` placeholder is the version of the package to add.</span></span>

---

::: moniker range="< aspnetcore-5.0"

## <a name="add-the-systemtextencodingsweb-package"></a><span data-ttu-id="a9d28-191">Aggiungere il pacchetto System. Text. Encodings. Web</span><span class="sxs-lookup"><span data-stu-id="a9d28-191">Add the System.Text.Encodings.Web package</span></span>

<span data-ttu-id="a9d28-192">*Questa sezione si applica solo alle app per ASP.NET Core versione 3. x.*</span><span class="sxs-lookup"><span data-stu-id="a9d28-192">*This section only applies to apps for ASP.NET Core version 3.x.*</span></span>

<span data-ttu-id="a9d28-193">A causa di un problema di risoluzione del pacchetto quando si usa [`System.Text.Json`](https://www.nuget.org/packages/System.Text.Json) 5. x in un'app ASP.NET Core 3. x, il `BlazorWebAssemblySignalRApp.Server` progetto richiede un riferimento al pacchetto per [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web) .</span><span class="sxs-lookup"><span data-stu-id="a9d28-193">Due to a package resolution issue when using [`System.Text.Json`](https://www.nuget.org/packages/System.Text.Json) 5.x in an ASP.NET Core 3.x app, the `BlazorWebAssemblySignalRApp.Server` project requires a package reference for [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web).</span></span> <span data-ttu-id="a9d28-194">Il problema sottostante verrà risolto in una versione futura della patch di .NET 5.</span><span class="sxs-lookup"><span data-stu-id="a9d28-194">The underlying issue will be resolved in a future patch release of .NET 5.</span></span> <span data-ttu-id="a9d28-195">Per altre informazioni, vedere [System.Text.Json definisce netcoreapp 3.0 senza dipendenze (DotNet/runtime #45560)](https://github.com/dotnet/runtime/issues/45560).</span><span class="sxs-lookup"><span data-stu-id="a9d28-195">For more information, see [System.Text.Json defines netcoreapp3.0 with no dependencies (dotnet/runtime #45560)](https://github.com/dotnet/runtime/issues/45560).</span></span>

<span data-ttu-id="a9d28-196">Per aggiungere [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web) al `BlazorWebAssemblySignalRApp.Server` progetto della soluzione ospitata ASP.NET Core 3,1 Blazor , seguire le istruzioni per la scelta degli strumenti:</span><span class="sxs-lookup"><span data-stu-id="a9d28-196">To add [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web) to the `BlazorWebAssemblySignalRApp.Server` project of the ASP.NET Core 3.1 hosted Blazor solution, follow the guidance for your choice of tooling:</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="a9d28-197">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="a9d28-197">Visual Studio</span></span>](#tab/visual-studio/)

1. <span data-ttu-id="a9d28-198">In **Esplora soluzioni** fare clic con il pulsante destro del mouse sul `BlazorWebAssemblySignalRApp.Server` progetto e scegliere **Gestisci pacchetti NuGet**.</span><span class="sxs-lookup"><span data-stu-id="a9d28-198">In **Solution Explorer**, right-click the `BlazorWebAssemblySignalRApp.Server` project and select **Manage NuGet Packages**.</span></span>

1. <span data-ttu-id="a9d28-199">Nella finestra di dialogo **Gestisci pacchetti NuGet** verificare che l' **origine del pacchetto** sia impostata su `nuget.org` .</span><span class="sxs-lookup"><span data-stu-id="a9d28-199">In the **Manage NuGet Packages** dialog, confirm that the **Package source** is set to `nuget.org`.</span></span>

1. <span data-ttu-id="a9d28-200">Con **Sfoglia** selezionato, digitare `System.Text.Encodings.Web` nella casella di ricerca.</span><span class="sxs-lookup"><span data-stu-id="a9d28-200">With **Browse** selected, type `System.Text.Encodings.Web` in the search box.</span></span>

1. <span data-ttu-id="a9d28-201">Nei risultati della ricerca selezionare il [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web) pacchetto.</span><span class="sxs-lookup"><span data-stu-id="a9d28-201">In the search results, select the [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web) package.</span></span> <span data-ttu-id="a9d28-202">Consente di selezionare la versione del pacchetto corrispondente al Framework condiviso in uso.</span><span class="sxs-lookup"><span data-stu-id="a9d28-202">Select the version of the package that matches the shared framework in use.</span></span> <span data-ttu-id="a9d28-203">Selezionare **Installa**.</span><span class="sxs-lookup"><span data-stu-id="a9d28-203">Select **Install**.</span></span>

1. <span data-ttu-id="a9d28-204">Se viene visualizzata la finestra di dialogo **Anteprima modifiche** , selezionare **OK**.</span><span class="sxs-lookup"><span data-stu-id="a9d28-204">If the **Preview Changes** dialog appears, select **OK**.</span></span>

1. <span data-ttu-id="a9d28-205">Se viene visualizzata la finestra di dialogo **accettazione della licenza** , selezionare Accetto se **si accettano le** condizioni di licenza.</span><span class="sxs-lookup"><span data-stu-id="a9d28-205">If the **License Acceptance** dialog appears, select **I Accept** if you agree with the license terms.</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="a9d28-206">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="a9d28-206">Visual Studio Code</span></span>](#tab/visual-studio-code/)

<span data-ttu-id="a9d28-207">Nel **terminale integrato** (**visualizzare** il > **terminale** dalla barra degli strumenti) eseguire i comandi seguenti:</span><span class="sxs-lookup"><span data-stu-id="a9d28-207">In the **Integrated Terminal** (**View** > **Terminal** from the toolbar), execute the following commands:</span></span>

```dotnetcli
dotnet add Server package System.Text.Encodings.Web
```

<span data-ttu-id="a9d28-208">Per aggiungere una versione precedente del pacchetto, specificare l' `--version {VERSION}` opzione, dove il `{VERSION}` segnaposto è la versione del pacchetto da aggiungere.</span><span class="sxs-lookup"><span data-stu-id="a9d28-208">To add an earlier version of the package, supply the `--version {VERSION}` option, where the `{VERSION}` placeholder is the version of the package to add.</span></span>

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="a9d28-209">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="a9d28-209">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

1. <span data-ttu-id="a9d28-210">Nella barra laterale **soluzione** fare clic con il pulsante destro del mouse sul `BlazorWebAssemblySignalRApp.Server` progetto e scegliere **Gestisci pacchetti NuGet**.</span><span class="sxs-lookup"><span data-stu-id="a9d28-210">In the **Solution** sidebar, right-click the `BlazorWebAssemblySignalRApp.Server` project and select **Manage NuGet Packages**.</span></span>

1. <span data-ttu-id="a9d28-211">Nella finestra di dialogo **Gestisci pacchetti NuGet** verificare che l'elenco a discesa origine sia impostato su `nuget.org` .</span><span class="sxs-lookup"><span data-stu-id="a9d28-211">In the **Manage NuGet Packages** dialog, confirm that the source drop-down is set to `nuget.org`.</span></span>

1. <span data-ttu-id="a9d28-212">Con **Sfoglia** selezionato, digitare `System.Text.Encodings.Web` nella casella di ricerca.</span><span class="sxs-lookup"><span data-stu-id="a9d28-212">With **Browse** selected, type `System.Text.Encodings.Web` in the search box.</span></span>

1. <span data-ttu-id="a9d28-213">Nei risultati della ricerca selezionare la casella di controllo accanto al [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web) pacchetto, selezionare la versione corretta del pacchetto corrispondente al Framework condiviso in uso e selezionare **Aggiungi pacchetto**.</span><span class="sxs-lookup"><span data-stu-id="a9d28-213">In the search results, select the check box next to the [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web) package, select the correct version of the package that matches the shared framework in use, and select **Add Package**.</span></span>

1. <span data-ttu-id="a9d28-214">Se viene visualizzata la finestra di dialogo **accettazione della licenza** , selezionare **Accetto** se si accettano le condizioni di licenza.</span><span class="sxs-lookup"><span data-stu-id="a9d28-214">If the **License Acceptance** dialog appears, select **Accept** if you agree with the license terms.</span></span>

# <a name="net-core-cli"></a>[<span data-ttu-id="a9d28-215">Interfaccia della riga di comando di .NET Core</span><span class="sxs-lookup"><span data-stu-id="a9d28-215">.NET Core CLI</span></span>](#tab/netcore-cli/)

<span data-ttu-id="a9d28-216">In una shell dei comandi dalla cartella della soluzione, eseguire il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="a9d28-216">In a command shell from the solution's folder, execute the following command:</span></span>

```dotnetcli
dotnet add Server package System.Text.Encodings.Web
```

<span data-ttu-id="a9d28-217">Per aggiungere una versione precedente del pacchetto, specificare l' `--version {VERSION}` opzione, dove il `{VERSION}` segnaposto è la versione del pacchetto da aggiungere.</span><span class="sxs-lookup"><span data-stu-id="a9d28-217">To add an earlier version of the package, supply the `--version {VERSION}` option, where the `{VERSION}` placeholder is the version of the package to add.</span></span>

---

::: moniker-end

## <a name="add-a-signalr-hub"></a><span data-ttu-id="a9d28-218">Aggiungere un SignalR Hub</span><span class="sxs-lookup"><span data-stu-id="a9d28-218">Add a SignalR hub</span></span>

<span data-ttu-id="a9d28-219">Nel `BlazorWebAssemblySignalRApp.Server` progetto creare una `Hubs` cartella (plurale) e aggiungere la classe seguente `ChatHub` ( `Hubs/ChatHub.cs` ):</span><span class="sxs-lookup"><span data-stu-id="a9d28-219">In the `BlazorWebAssemblySignalRApp.Server` project, create a `Hubs` (plural) folder and add the following `ChatHub` class (`Hubs/ChatHub.cs`):</span></span>

::: moniker range=">= aspnetcore-5.0"

[!code-csharp[](~/tutorials/signalr-blazor/samples/5.x/BlazorWebAssemblySignalRApp/Server/Hubs/ChatHub.cs)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-csharp[](~/tutorials/signalr-blazor/samples/3.x/BlazorWebAssemblySignalRApp/Server/Hubs/ChatHub.cs)]

::: moniker-end

## <a name="add-services-and-an-endpoint-for-the-signalr-hub"></a><span data-ttu-id="a9d28-220">Aggiungere servizi e un endpoint per l' SignalR Hub</span><span class="sxs-lookup"><span data-stu-id="a9d28-220">Add services and an endpoint for the SignalR hub</span></span>

1. <span data-ttu-id="a9d28-221">Nel progetto `BlazorWebAssemblySignalRApp.Server` aprire il file `Startup.cs`.</span><span class="sxs-lookup"><span data-stu-id="a9d28-221">In the `BlazorWebAssemblySignalRApp.Server` project, open the `Startup.cs` file.</span></span>

1. <span data-ttu-id="a9d28-222">Aggiungere lo spazio dei nomi per la `ChatHub` classe all'inizio del file:</span><span class="sxs-lookup"><span data-stu-id="a9d28-222">Add the namespace for the `ChatHub` class to the top of the file:</span></span>

   ```csharp
   using BlazorWebAssemblySignalRApp.Server.Hubs;
   ```

::: moniker range=">= aspnetcore-5.0"

1. <span data-ttu-id="a9d28-223">Aggiungere SignalR e rispondere ai servizi middleware di compressione per `Startup.ConfigureServices` :</span><span class="sxs-lookup"><span data-stu-id="a9d28-223">Add SignalR and Response Compression Middleware services to `Startup.ConfigureServices`:</span></span>

   [!code-csharp[](~/tutorials/signalr-blazor/samples/5.x/BlazorWebAssemblySignalRApp/Server/Startup.cs?name=snippet_ConfigureServices&highlight=3,6-10)]

1. <span data-ttu-id="a9d28-224">In `Startup.Configure`:</span><span class="sxs-lookup"><span data-stu-id="a9d28-224">In `Startup.Configure`:</span></span>

   * <span data-ttu-id="a9d28-225">Usare il middleware della compressione della risposta nella parte superiore della configurazione della pipeline di elaborazione.</span><span class="sxs-lookup"><span data-stu-id="a9d28-225">Use Response Compression Middleware at the top of the processing pipeline's configuration.</span></span>
   * <span data-ttu-id="a9d28-226">Tra gli endpoint per i controller e il fallback sul lato client, aggiungere un endpoint per l'hub.</span><span class="sxs-lookup"><span data-stu-id="a9d28-226">Between the endpoints for controllers and the client-side fallback, add an endpoint for the hub.</span></span>

   [!code-csharp[](~/tutorials/signalr-blazor/samples/5.x/BlazorWebAssemblySignalRApp/Server/Startup.cs?name=snippet_Configure&highlight=3,26)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

1. <span data-ttu-id="a9d28-227">Aggiungere SignalR e rispondere ai servizi middleware di compressione per `Startup.ConfigureServices` :</span><span class="sxs-lookup"><span data-stu-id="a9d28-227">Add SignalR and Response Compression Middleware services to `Startup.ConfigureServices`:</span></span>

   [!code-csharp[](~/tutorials/signalr-blazor/samples/3.x/BlazorWebAssemblySignalRApp/Server/Startup.cs?name=snippet_ConfigureServices&highlight=3,5-9)]

1. <span data-ttu-id="a9d28-228">In `Startup.Configure`:</span><span class="sxs-lookup"><span data-stu-id="a9d28-228">In `Startup.Configure`:</span></span>

   * <span data-ttu-id="a9d28-229">Usare il middleware della compressione della risposta nella parte superiore della configurazione della pipeline di elaborazione.</span><span class="sxs-lookup"><span data-stu-id="a9d28-229">Use Response Compression Middleware at the top of the processing pipeline's configuration.</span></span>
   * <span data-ttu-id="a9d28-230">Tra gli endpoint per i controller e il fallback sul lato client, aggiungere un endpoint per l'hub.</span><span class="sxs-lookup"><span data-stu-id="a9d28-230">Between the endpoints for controllers and the client-side fallback, add an endpoint for the hub.</span></span>

   [!code-csharp[](~/tutorials/signalr-blazor/samples/3.x/BlazorWebAssemblySignalRApp/Server/Startup.cs?name=snippet_Configure&highlight=3,25)]

::: moniker-end

## <a name="add-razor-component-code-for-chat"></a><span data-ttu-id="a9d28-231">Aggiungi Razor codice componente per la chat</span><span class="sxs-lookup"><span data-stu-id="a9d28-231">Add Razor component code for chat</span></span>

1. <span data-ttu-id="a9d28-232">Nel progetto `BlazorWebAssemblySignalRApp.Client` aprire il file `Pages/Index.razor`.</span><span class="sxs-lookup"><span data-stu-id="a9d28-232">In the `BlazorWebAssemblySignalRApp.Client` project, open the `Pages/Index.razor` file.</span></span>

::: moniker range=">= aspnetcore-5.0"

1. <span data-ttu-id="a9d28-233">Sostituire il markup con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="a9d28-233">Replace the markup with the following code:</span></span>

   [!code-razor[](~/tutorials/signalr-blazor/samples/5.x/BlazorWebAssemblySignalRApp/Client/Pages/Index.razor)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

1. <span data-ttu-id="a9d28-234">Sostituire il markup con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="a9d28-234">Replace the markup with the following code:</span></span>

   [!code-razor[](~/tutorials/signalr-blazor/samples/3.x/BlazorWebAssemblySignalRApp/Client/Pages/Index.razor)]

::: moniker-end

## <a name="run-the-app"></a><span data-ttu-id="a9d28-235">Eseguire l'app</span><span class="sxs-lookup"><span data-stu-id="a9d28-235">Run the app</span></span>

<span data-ttu-id="a9d28-236">Seguire le istruzioni per gli strumenti:</span><span class="sxs-lookup"><span data-stu-id="a9d28-236">Follow the guidance for your tooling:</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="a9d28-237">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="a9d28-237">Visual Studio</span></span>](#tab/visual-studio)

1. <span data-ttu-id="a9d28-238">In **Esplora soluzioni** selezionare il `BlazorWebAssemblySignalRApp.Server` progetto.</span><span class="sxs-lookup"><span data-stu-id="a9d28-238">In **Solution Explorer**, select the `BlazorWebAssemblySignalRApp.Server` project.</span></span> <span data-ttu-id="a9d28-239">Premere <kbd>F5</kbd> per eseguire l'app con il debug o <kbd>CTRL</kbd> + <kbd>F5</kbd> per eseguire l'app senza debug.</span><span class="sxs-lookup"><span data-stu-id="a9d28-239">Press <kbd>F5</kbd> to run the app with debugging or <kbd>Ctrl</kbd>+<kbd>F5</kbd> to run the app without debugging.</span></span>

1. <span data-ttu-id="a9d28-240">Copiare l'URL dalla barra degli indirizzi, aprire un'altra istanza o scheda del browser e incollare l'URL nella barra degli indirizzi.</span><span class="sxs-lookup"><span data-stu-id="a9d28-240">Copy the URL from the address bar, open another browser instance or tab, and paste the URL in the address bar.</span></span>

1. <span data-ttu-id="a9d28-241">Scegliere browser, immettere un nome e un messaggio e selezionare il pulsante per l'invio del messaggio.</span><span class="sxs-lookup"><span data-stu-id="a9d28-241">Choose either browser, enter a name and message, and select the button to send the message.</span></span> <span data-ttu-id="a9d28-242">Il nome e il messaggio vengono visualizzati immediatamente in entrambe le pagine:</span><span class="sxs-lookup"><span data-stu-id="a9d28-242">The name and message are displayed on both pages instantly:</span></span>

   ![::: NO-LOC (SignalR):::::: NO-LOC (Blazer)::: app di esempio aperta in due finestre del browser che mostrano i messaggi scambiati.](signalr-blazor/_static/signalr-blazor-finished.png)

   <span data-ttu-id="a9d28-244">Virgolette: *Star Trek VI: il paese non individuato* &copy; 1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)</span><span class="sxs-lookup"><span data-stu-id="a9d28-244">Quotes: *Star Trek VI: The Undiscovered Country* &copy;1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="a9d28-245">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="a9d28-245">Visual Studio Code</span></span>](#tab/visual-studio-code)

<span data-ttu-id="a9d28-246">Per informazioni sulla configurazione di asset VS Code nella `.vscode` cartella, vedere la guida del sistema operativo **Linux** in <xref:blazor/tooling> .</span><span class="sxs-lookup"><span data-stu-id="a9d28-246">For information on configuring VS Code assets in the `.vscode` folder, see the **Linux** operating system guidance in <xref:blazor/tooling>.</span></span>

1. <span data-ttu-id="a9d28-247">Premere <kbd>F5</kbd> per eseguire l'app con il debug o <kbd>CTRL</kbd> + <kbd>F5</kbd> per eseguire l'app senza debug.</span><span class="sxs-lookup"><span data-stu-id="a9d28-247">Press <kbd>F5</kbd> to run the app with debugging or <kbd>Ctrl</kbd>+<kbd>F5</kbd> to run the app without debugging.</span></span>

1. <span data-ttu-id="a9d28-248">Copiare l'URL dalla barra degli indirizzi, aprire un'altra istanza o scheda del browser e incollare l'URL nella barra degli indirizzi.</span><span class="sxs-lookup"><span data-stu-id="a9d28-248">Copy the URL from the address bar, open another browser instance or tab, and paste the URL in the address bar.</span></span>

1. <span data-ttu-id="a9d28-249">Scegliere browser, immettere un nome e un messaggio e selezionare il pulsante per l'invio del messaggio.</span><span class="sxs-lookup"><span data-stu-id="a9d28-249">Choose either browser, enter a name and message, and select the button to send the message.</span></span> <span data-ttu-id="a9d28-250">Il nome e il messaggio vengono visualizzati immediatamente in entrambe le pagine:</span><span class="sxs-lookup"><span data-stu-id="a9d28-250">The name and message are displayed on both pages instantly:</span></span>

   ![::: NO-LOC (SignalR):::::: NO-LOC (Blazer)::: app di esempio aperta in due finestre del browser che mostrano i messaggi scambiati.](signalr-blazor/_static/signalr-blazor-finished.png)

   <span data-ttu-id="a9d28-252">Virgolette: *Star Trek VI: il paese non individuato* &copy; 1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)</span><span class="sxs-lookup"><span data-stu-id="a9d28-252">Quotes: *Star Trek VI: The Undiscovered Country* &copy;1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)</span></span>

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="a9d28-253">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="a9d28-253">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

1. <span data-ttu-id="a9d28-254">Nella barra laterale della **soluzione** selezionare il `BlazorWebAssemblySignalRApp.Server` progetto.</span><span class="sxs-lookup"><span data-stu-id="a9d28-254">In the **Solution** sidebar, select the `BlazorWebAssemblySignalRApp.Server` project.</span></span> <span data-ttu-id="a9d28-255">Premere <kbd>⌘</kbd> + <kbd>↩</kbd> per eseguire l'app con debug o <kbd>⌥</kbd> + <kbd>⌘</kbd> + <kbd>↩</kbd> per eseguire l'app senza debug.</span><span class="sxs-lookup"><span data-stu-id="a9d28-255">Press <kbd>⌘</kbd>+<kbd>↩</kbd> to run the app with debugging or <kbd>⌥</kbd>+<kbd>⌘</kbd>+<kbd>↩</kbd> to run the app without debugging.</span></span>

1. <span data-ttu-id="a9d28-256">Copiare l'URL dalla barra degli indirizzi, aprire un'altra istanza o scheda del browser e incollare l'URL nella barra degli indirizzi.</span><span class="sxs-lookup"><span data-stu-id="a9d28-256">Copy the URL from the address bar, open another browser instance or tab, and paste the URL in the address bar.</span></span>

1. <span data-ttu-id="a9d28-257">Scegliere browser, immettere un nome e un messaggio e selezionare il pulsante per l'invio del messaggio.</span><span class="sxs-lookup"><span data-stu-id="a9d28-257">Choose either browser, enter a name and message, and select the button to send the message.</span></span> <span data-ttu-id="a9d28-258">Il nome e il messaggio vengono visualizzati immediatamente in entrambe le pagine:</span><span class="sxs-lookup"><span data-stu-id="a9d28-258">The name and message are displayed on both pages instantly:</span></span>

   ![::: NO-LOC (SignalR):::::: NO-LOC (Blazer)::: app di esempio aperta in due finestre del browser che mostrano i messaggi scambiati.](signalr-blazor/_static/signalr-blazor-finished.png)

   <span data-ttu-id="a9d28-260">Virgolette: *Star Trek VI: il paese non individuato* &copy; 1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)</span><span class="sxs-lookup"><span data-stu-id="a9d28-260">Quotes: *Star Trek VI: The Undiscovered Country* &copy;1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)</span></span>

# <a name="net-core-cli"></a>[<span data-ttu-id="a9d28-261">Interfaccia della riga di comando di .NET Core</span><span class="sxs-lookup"><span data-stu-id="a9d28-261">.NET Core CLI</span></span>](#tab/netcore-cli/)

1. <span data-ttu-id="a9d28-262">In una shell dei comandi dalla cartella della soluzione eseguire i comandi seguenti:</span><span class="sxs-lookup"><span data-stu-id="a9d28-262">In a command shell from the solution's folder, execute the following commands:</span></span>

   ```dotnetcli
   cd Server
   dotnet run
   ```

1. <span data-ttu-id="a9d28-263">Copiare l'URL dalla barra degli indirizzi, aprire un'altra istanza o scheda del browser e incollare l'URL nella barra degli indirizzi.</span><span class="sxs-lookup"><span data-stu-id="a9d28-263">Copy the URL from the address bar, open another browser instance or tab, and paste the URL in the address bar.</span></span>

1. <span data-ttu-id="a9d28-264">Scegliere browser, immettere un nome e un messaggio e selezionare il pulsante per l'invio del messaggio.</span><span class="sxs-lookup"><span data-stu-id="a9d28-264">Choose either browser, enter a name and message, and select the button to send the message.</span></span> <span data-ttu-id="a9d28-265">Il nome e il messaggio vengono visualizzati immediatamente in entrambe le pagine:</span><span class="sxs-lookup"><span data-stu-id="a9d28-265">The name and message are displayed on both pages instantly:</span></span>

   ![::: NO-LOC (SignalR):::::: NO-LOC (Blazer)::: app di esempio aperta in due finestre del browser che mostrano i messaggi scambiati.](signalr-blazor/_static/signalr-blazor-finished.png)

   <span data-ttu-id="a9d28-267">Virgolette: *Star Trek VI: il paese non individuato* &copy; 1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)</span><span class="sxs-lookup"><span data-stu-id="a9d28-267">Quotes: *Star Trek VI: The Undiscovered Country* &copy;1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)</span></span>

---

::: zone-end

::: zone pivot="server"

## <a name="create-a-blazor-server-app"></a><span data-ttu-id="a9d28-268">Creare un' Blazor Server app</span><span class="sxs-lookup"><span data-stu-id="a9d28-268">Create a Blazor Server app</span></span>

<span data-ttu-id="a9d28-269">Seguire le istruzioni per la scelta degli strumenti:</span><span class="sxs-lookup"><span data-stu-id="a9d28-269">Follow the guidance for your choice of tooling:</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="a9d28-270">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="a9d28-270">Visual Studio</span></span>](#tab/visual-studio)

::: moniker range=">= aspnetcore-5.0"

> [!NOTE]
> <span data-ttu-id="a9d28-271">Sono necessari Visual Studio 16,8 o versioni successive e .NET Core SDK 5.0.0 o versioni successive.</span><span class="sxs-lookup"><span data-stu-id="a9d28-271">Visual Studio 16.8 or later and .NET Core SDK 5.0.0 or later are required.</span></span>

::: moniker-end

::: moniker range="< aspnetcore-5.0"

> [!NOTE]
> <span data-ttu-id="a9d28-272">Sono necessari Visual Studio 16,6 o versioni successive e .NET Core SDK 3.1.300 o versioni successive.</span><span class="sxs-lookup"><span data-stu-id="a9d28-272">Visual Studio 16.6 or later and .NET Core SDK 3.1.300 or later are required.</span></span>

::: moniker-end

1. <span data-ttu-id="a9d28-273">Creare un nuovo progetto.</span><span class="sxs-lookup"><span data-stu-id="a9d28-273">Create a new project.</span></span>

1. <span data-ttu-id="a9d28-274">Selezionare **Blazor app** e fare clic su **Next (avanti**).</span><span class="sxs-lookup"><span data-stu-id="a9d28-274">Select **Blazor App** and select **Next**.</span></span>

1. <span data-ttu-id="a9d28-275">Digitare `BlazorServerSignalRApp` nel campo **nome progetto** .</span><span class="sxs-lookup"><span data-stu-id="a9d28-275">Type `BlazorServerSignalRApp` in the **Project name** field.</span></span> <span data-ttu-id="a9d28-276">Confermare che la voce relativa al **percorso** sia corretta o specificare un percorso per il progetto.</span><span class="sxs-lookup"><span data-stu-id="a9d28-276">Confirm the **Location** entry is correct or provide a location for the project.</span></span> <span data-ttu-id="a9d28-277">Selezionare **Crea**.</span><span class="sxs-lookup"><span data-stu-id="a9d28-277">Select **Create**.</span></span>

1. <span data-ttu-id="a9d28-278">Scegliere il modello di **Blazor Server app** .</span><span class="sxs-lookup"><span data-stu-id="a9d28-278">Choose the **Blazor Server App** template.</span></span>

1. <span data-ttu-id="a9d28-279">Selezionare **Crea**.</span><span class="sxs-lookup"><span data-stu-id="a9d28-279">Select **Create**.</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="a9d28-280">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="a9d28-280">Visual Studio Code</span></span>](#tab/visual-studio-code)

1. <span data-ttu-id="a9d28-281">In una shell dei comandi eseguire il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="a9d28-281">In a command shell, execute the following command:</span></span>

   ```dotnetcli
   dotnet new blazorserver -o BlazorServerSignalRApp
   ```

   <span data-ttu-id="a9d28-282">L' `-o|--output` opzione Crea una cartella per il progetto.</span><span class="sxs-lookup"><span data-stu-id="a9d28-282">The `-o|--output` option creates a folder for the project.</span></span> <span data-ttu-id="a9d28-283">Se è stata creata una cartella per il progetto e la shell dei comandi è aperta in tale cartella, omettere l' `-o|--output` opzione e il valore per creare il progetto.</span><span class="sxs-lookup"><span data-stu-id="a9d28-283">If you've created a folder for the project and the command shell is open in that folder, omit the `-o|--output` option and value to create the project.</span></span>

1. <span data-ttu-id="a9d28-284">In Visual Studio Code aprire la cartella del progetto dell'app.</span><span class="sxs-lookup"><span data-stu-id="a9d28-284">In Visual Studio Code, open the app's project folder.</span></span>

1. <span data-ttu-id="a9d28-285">Quando viene visualizzata la finestra di dialogo per aggiungere asset per compilare ed eseguire il debug dell'app, selezionare **Sì**.</span><span class="sxs-lookup"><span data-stu-id="a9d28-285">When the dialog appears to add assets to build and debug the app, select **Yes**.</span></span> <span data-ttu-id="a9d28-286">Visual Studio Code aggiunge automaticamente la `.vscode` cartella con `launch.json` `tasks.json` i file e generati.</span><span class="sxs-lookup"><span data-stu-id="a9d28-286">Visual Studio Code automatically adds the `.vscode` folder with generated `launch.json` and `tasks.json` files.</span></span> <span data-ttu-id="a9d28-287">Per informazioni sulla configurazione di asset VS Code nella `.vscode` cartella, vedere la guida del sistema operativo **Linux** in <xref:blazor/tooling> .</span><span class="sxs-lookup"><span data-stu-id="a9d28-287">For information on configuring VS Code assets in the `.vscode` folder, see the **Linux** operating system guidance in <xref:blazor/tooling>.</span></span>

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="a9d28-288">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="a9d28-288">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

1. <span data-ttu-id="a9d28-289">Installare la versione più recente di [Visual Studio per Mac](https://visualstudio.microsoft.com/vs/mac/) e seguire questa procedura:</span><span class="sxs-lookup"><span data-stu-id="a9d28-289">Install the latest version of [Visual Studio for Mac](https://visualstudio.microsoft.com/vs/mac/) and perform the following steps:</span></span>

1. <span data-ttu-id="a9d28-290">Selezionare **file**  >  **nuova soluzione** o creare un **nuovo** progetto dalla **finestra Start**.</span><span class="sxs-lookup"><span data-stu-id="a9d28-290">Select **File** > **New Solution** or create a **New** project from the **Start Window**.</span></span>

1. <span data-ttu-id="a9d28-291">Nella barra laterale selezionare app **Web e console**  >  .</span><span class="sxs-lookup"><span data-stu-id="a9d28-291">In the sidebar, select **Web and Console** > **App**.</span></span>

1. <span data-ttu-id="a9d28-292">Scegliere il modello di **Blazor Server app** .</span><span class="sxs-lookup"><span data-stu-id="a9d28-292">Choose the **Blazor Server App** template.</span></span> <span data-ttu-id="a9d28-293">Selezionare **Avanti**.</span><span class="sxs-lookup"><span data-stu-id="a9d28-293">Select **Next**.</span></span>

1. <span data-ttu-id="a9d28-294">Verificare che **l'autenticazione** sia impostata su **Nessuna autenticazione**.</span><span class="sxs-lookup"><span data-stu-id="a9d28-294">Confirm that **Authentication** is set to **No Authentication**.</span></span> <span data-ttu-id="a9d28-295">Selezionare **Avanti**.</span><span class="sxs-lookup"><span data-stu-id="a9d28-295">Select **Next**.</span></span>

1. <span data-ttu-id="a9d28-296">Nel campo **nome progetto** assegnare un nome all'app `BlazorServerSignalRApp` .</span><span class="sxs-lookup"><span data-stu-id="a9d28-296">In the **Project Name** field, name the app `BlazorServerSignalRApp`.</span></span> <span data-ttu-id="a9d28-297">Selezionare **Crea**.</span><span class="sxs-lookup"><span data-stu-id="a9d28-297">Select **Create**.</span></span>

   <span data-ttu-id="a9d28-298">Se viene visualizzato un messaggio per considerare attendibile il certificato di sviluppo, considerare attendibile il certificato e continuare.</span><span class="sxs-lookup"><span data-stu-id="a9d28-298">If a prompt appears to trust the development certificate, trust the certificate and continue.</span></span> <span data-ttu-id="a9d28-299">Per considerare attendibile il certificato, è necessario specificare le password dell'utente e del keychain.</span><span class="sxs-lookup"><span data-stu-id="a9d28-299">The user and keychain passwords are required to trust the certificate.</span></span>

1. <span data-ttu-id="a9d28-300">Aprire il progetto passando alla cartella del progetto e aprendo il file di soluzione del progetto ( `.sln` ).</span><span class="sxs-lookup"><span data-stu-id="a9d28-300">Open the project by navigating to the project folder and opening the project's solution file (`.sln`).</span></span>

# <a name="net-core-cli"></a>[<span data-ttu-id="a9d28-301">Interfaccia della riga di comando di .NET Core</span><span class="sxs-lookup"><span data-stu-id="a9d28-301">.NET Core CLI</span></span>](#tab/netcore-cli/)

<span data-ttu-id="a9d28-302">In una shell dei comandi eseguire il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="a9d28-302">In a command shell, execute the following command:</span></span>

```dotnetcli
dotnet new blazorserver -o BlazorServerSignalRApp
```

<span data-ttu-id="a9d28-303">L' `-o|--output` opzione Crea una cartella per il progetto.</span><span class="sxs-lookup"><span data-stu-id="a9d28-303">The `-o|--output` option creates a folder for the project.</span></span> <span data-ttu-id="a9d28-304">Se è stata creata una cartella per il progetto e la shell dei comandi è aperta in tale cartella, omettere l' `-o|--output` opzione e il valore per creare il progetto.</span><span class="sxs-lookup"><span data-stu-id="a9d28-304">If you've created a folder for the project and the command shell is open in that folder, omit the `-o|--output` option and value to create the project.</span></span>

---

## <a name="add-the-signalr-client-library"></a><span data-ttu-id="a9d28-305">Aggiungere la SignalR libreria client</span><span class="sxs-lookup"><span data-stu-id="a9d28-305">Add the SignalR client library</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="a9d28-306">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="a9d28-306">Visual Studio</span></span>](#tab/visual-studio/)

1. <span data-ttu-id="a9d28-307">In **Esplora soluzioni** fare clic con il pulsante destro del mouse sul `BlazorServerSignalRApp` progetto e scegliere **Gestisci pacchetti NuGet**.</span><span class="sxs-lookup"><span data-stu-id="a9d28-307">In **Solution Explorer**, right-click the `BlazorServerSignalRApp` project and select **Manage NuGet Packages**.</span></span>

1. <span data-ttu-id="a9d28-308">Nella finestra di dialogo **Gestisci pacchetti NuGet** verificare che l' **origine del pacchetto** sia impostata su `nuget.org` .</span><span class="sxs-lookup"><span data-stu-id="a9d28-308">In the **Manage NuGet Packages** dialog, confirm that the **Package source** is set to `nuget.org`.</span></span>

1. <span data-ttu-id="a9d28-309">Con **Sfoglia** selezionato, digitare `Microsoft.AspNetCore.SignalR.Client` nella casella di ricerca.</span><span class="sxs-lookup"><span data-stu-id="a9d28-309">With **Browse** selected, type `Microsoft.AspNetCore.SignalR.Client` in the search box.</span></span>

1. <span data-ttu-id="a9d28-310">Nei risultati della ricerca selezionare il [`Microsoft.AspNetCore.SignalR.Client`](https://www.nuget.org/packages/Microsoft.AspNetCore.SignalR.Client) pacchetto.</span><span class="sxs-lookup"><span data-stu-id="a9d28-310">In the search results, select the [`Microsoft.AspNetCore.SignalR.Client`](https://www.nuget.org/packages/Microsoft.AspNetCore.SignalR.Client) package.</span></span> <span data-ttu-id="a9d28-311">Impostare la versione in modo che corrisponda al Framework condiviso dell'app.</span><span class="sxs-lookup"><span data-stu-id="a9d28-311">Set the version to match the shared framework of the app.</span></span> <span data-ttu-id="a9d28-312">Selezionare **Installa**.</span><span class="sxs-lookup"><span data-stu-id="a9d28-312">Select **Install**.</span></span>

1. <span data-ttu-id="a9d28-313">Se viene visualizzata la finestra di dialogo **Anteprima modifiche** , selezionare **OK**.</span><span class="sxs-lookup"><span data-stu-id="a9d28-313">If the **Preview Changes** dialog appears, select **OK**.</span></span>

1. <span data-ttu-id="a9d28-314">Se viene visualizzata la finestra di dialogo **accettazione della licenza** , selezionare Accetto se **si accettano le** condizioni di licenza.</span><span class="sxs-lookup"><span data-stu-id="a9d28-314">If the **License Acceptance** dialog appears, select **I Accept** if you agree with the license terms.</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="a9d28-315">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="a9d28-315">Visual Studio Code</span></span>](#tab/visual-studio-code/)

<span data-ttu-id="a9d28-316">Nel **terminale integrato** (**visualizzare** il  >  **terminale** dalla barra degli strumenti) eseguire il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="a9d28-316">In the **Integrated Terminal** (**View** > **Terminal** from the toolbar), execute the following command:</span></span>

```dotnetcli
dotnet add package Microsoft.AspNetCore.SignalR.Client
```

<span data-ttu-id="a9d28-317">Per aggiungere una versione precedente del pacchetto, specificare l' `--version {VERSION}` opzione, dove il `{VERSION}` segnaposto è la versione del pacchetto da aggiungere.</span><span class="sxs-lookup"><span data-stu-id="a9d28-317">To add an earlier version of the package, supply the `--version {VERSION}` option, where the `{VERSION}` placeholder is the version of the package to add.</span></span>

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="a9d28-318">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="a9d28-318">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

1. <span data-ttu-id="a9d28-319">Nella barra laterale **soluzione** fare clic con il pulsante destro del mouse sul `BlazorServerSignalRApp` progetto e scegliere **Gestisci pacchetti NuGet**.</span><span class="sxs-lookup"><span data-stu-id="a9d28-319">In the **Solution** sidebar, right-click the `BlazorServerSignalRApp` project and select **Manage NuGet Packages**.</span></span>

1. <span data-ttu-id="a9d28-320">Nella finestra di dialogo **Gestisci pacchetti NuGet** verificare che l'elenco a discesa origine sia impostato su `nuget.org` .</span><span class="sxs-lookup"><span data-stu-id="a9d28-320">In the **Manage NuGet Packages** dialog, confirm that the source drop-down is set to `nuget.org`.</span></span>

1. <span data-ttu-id="a9d28-321">Con **Sfoglia** selezionato, digitare `Microsoft.AspNetCore.SignalR.Client` nella casella di ricerca.</span><span class="sxs-lookup"><span data-stu-id="a9d28-321">With **Browse** selected, type `Microsoft.AspNetCore.SignalR.Client` in the search box.</span></span>

1. <span data-ttu-id="a9d28-322">Nei risultati della ricerca selezionare la casella di controllo accanto al [`Microsoft.AspNetCore.SignalR.Client`](https://www.nuget.org/packages/Microsoft.AspNetCore.SignalR.Client) pacchetto.</span><span class="sxs-lookup"><span data-stu-id="a9d28-322">In the search results, select the check box next to the [`Microsoft.AspNetCore.SignalR.Client`](https://www.nuget.org/packages/Microsoft.AspNetCore.SignalR.Client) package.</span></span> <span data-ttu-id="a9d28-323">Impostare la versione in modo che corrisponda al Framework condiviso dell'app.</span><span class="sxs-lookup"><span data-stu-id="a9d28-323">Set the version to match the shared framework of the app.</span></span> <span data-ttu-id="a9d28-324">Selezionare **Aggiungi pacchetto**.</span><span class="sxs-lookup"><span data-stu-id="a9d28-324">Select **Add Package**.</span></span>

1. <span data-ttu-id="a9d28-325">Se viene visualizzata la finestra di dialogo **accettazione della licenza** , selezionare **Accetto** se si accettano le condizioni di licenza.</span><span class="sxs-lookup"><span data-stu-id="a9d28-325">If the **License Acceptance** dialog appears, select **Accept** if you agree with the license terms.</span></span>

# <a name="net-core-cli"></a>[<span data-ttu-id="a9d28-326">Interfaccia della riga di comando di .NET Core</span><span class="sxs-lookup"><span data-stu-id="a9d28-326">.NET Core CLI</span></span>](#tab/netcore-cli/)

<span data-ttu-id="a9d28-327">In una shell dei comandi dalla cartella del progetto, eseguire il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="a9d28-327">In a command shell from the project's folder, execute the following command:</span></span>

```dotnetcli
dotnet add package Microsoft.AspNetCore.SignalR.Client
```

<span data-ttu-id="a9d28-328">Per aggiungere una versione precedente del pacchetto, specificare l' `--version {VERSION}` opzione, dove il `{VERSION}` segnaposto è la versione del pacchetto da aggiungere.</span><span class="sxs-lookup"><span data-stu-id="a9d28-328">To add an earlier version of the package, supply the `--version {VERSION}` option, where the `{VERSION}` placeholder is the version of the package to add.</span></span>

---

::: moniker range="< aspnetcore-5.0"

## <a name="add-the-systemtextencodingsweb-package"></a><span data-ttu-id="a9d28-329">Aggiungere il pacchetto System. Text. Encodings. Web</span><span class="sxs-lookup"><span data-stu-id="a9d28-329">Add the System.Text.Encodings.Web package</span></span>

<span data-ttu-id="a9d28-330">*Questa sezione si applica solo alle app per ASP.NET Core versione 3. x.*</span><span class="sxs-lookup"><span data-stu-id="a9d28-330">*This section only applies to apps for ASP.NET Core version 3.x.*</span></span>

<span data-ttu-id="a9d28-331">A causa di un problema di risoluzione del pacchetto quando si usa [`System.Text.Json`](https://www.nuget.org/packages/System.Text.Json) 5. x in un'app ASP.NET Core 3. x, il progetto richiede un riferimento al pacchetto per [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web) .</span><span class="sxs-lookup"><span data-stu-id="a9d28-331">Due to a package resolution issue when using [`System.Text.Json`](https://www.nuget.org/packages/System.Text.Json) 5.x in an ASP.NET Core 3.x app, the project requires a package reference for [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web).</span></span> <span data-ttu-id="a9d28-332">Il problema sottostante verrà risolto in una versione futura della patch di .NET 5.</span><span class="sxs-lookup"><span data-stu-id="a9d28-332">The underlying issue will be resolved in a future patch release of .NET 5.</span></span> <span data-ttu-id="a9d28-333">Per altre informazioni, vedere [System.Text.Json definisce netcoreapp 3.0 senza dipendenze (DotNet/runtime #45560)](https://github.com/dotnet/runtime/issues/45560).</span><span class="sxs-lookup"><span data-stu-id="a9d28-333">For more information, see [System.Text.Json defines netcoreapp3.0 with no dependencies (dotnet/runtime #45560)](https://github.com/dotnet/runtime/issues/45560).</span></span>

<span data-ttu-id="a9d28-334">Per aggiungere [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web) al progetto, seguire le istruzioni per la scelta degli strumenti:</span><span class="sxs-lookup"><span data-stu-id="a9d28-334">To add [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web) to the project, follow the guidance for your choice of tooling:</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="a9d28-335">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="a9d28-335">Visual Studio</span></span>](#tab/visual-studio/)

1. <span data-ttu-id="a9d28-336">In **Esplora soluzioni** fare clic con il pulsante destro del mouse sul `BlazorServerSignalRApp` progetto e scegliere **Gestisci pacchetti NuGet**.</span><span class="sxs-lookup"><span data-stu-id="a9d28-336">In **Solution Explorer**, right-click the `BlazorServerSignalRApp` project and select **Manage NuGet Packages**.</span></span>

1. <span data-ttu-id="a9d28-337">Nella finestra di dialogo **Gestisci pacchetti NuGet** verificare che l' **origine del pacchetto** sia impostata su `nuget.org` .</span><span class="sxs-lookup"><span data-stu-id="a9d28-337">In the **Manage NuGet Packages** dialog, confirm that the **Package source** is set to `nuget.org`.</span></span>

1. <span data-ttu-id="a9d28-338">Con **Sfoglia** selezionato, digitare `System.Text.Encodings.Web` nella casella di ricerca.</span><span class="sxs-lookup"><span data-stu-id="a9d28-338">With **Browse** selected, type `System.Text.Encodings.Web` in the search box.</span></span>

1. <span data-ttu-id="a9d28-339">Nei risultati della ricerca selezionare il [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web) pacchetto.</span><span class="sxs-lookup"><span data-stu-id="a9d28-339">In the search results, select the [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web) package.</span></span> <span data-ttu-id="a9d28-340">Consente di selezionare la versione del pacchetto corrispondente al Framework condiviso in uso.</span><span class="sxs-lookup"><span data-stu-id="a9d28-340">Select the version of the package that matches the shared framework in use.</span></span> <span data-ttu-id="a9d28-341">Selezionare **Installa**.</span><span class="sxs-lookup"><span data-stu-id="a9d28-341">Select **Install**.</span></span>

1. <span data-ttu-id="a9d28-342">Se viene visualizzata la finestra di dialogo **Anteprima modifiche** , selezionare **OK**.</span><span class="sxs-lookup"><span data-stu-id="a9d28-342">If the **Preview Changes** dialog appears, select **OK**.</span></span>

1. <span data-ttu-id="a9d28-343">Se viene visualizzata la finestra di dialogo **accettazione della licenza** , selezionare Accetto se **si accettano le** condizioni di licenza.</span><span class="sxs-lookup"><span data-stu-id="a9d28-343">If the **License Acceptance** dialog appears, select **I Accept** if you agree with the license terms.</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="a9d28-344">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="a9d28-344">Visual Studio Code</span></span>](#tab/visual-studio-code/)

<span data-ttu-id="a9d28-345">Nel **terminale integrato** (**visualizzare** il > **terminale** dalla barra degli strumenti) eseguire i comandi seguenti:</span><span class="sxs-lookup"><span data-stu-id="a9d28-345">In the **Integrated Terminal** (**View** > **Terminal** from the toolbar), execute the following commands:</span></span>

```dotnetcli
dotnet add package System.Text.Encodings.Web
```

<span data-ttu-id="a9d28-346">Per aggiungere una versione precedente del pacchetto, specificare l' `--version {VERSION}` opzione, dove il `{VERSION}` segnaposto è la versione del pacchetto da aggiungere.</span><span class="sxs-lookup"><span data-stu-id="a9d28-346">To add an earlier version of the package, supply the `--version {VERSION}` option, where the `{VERSION}` placeholder is the version of the package to add.</span></span>

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="a9d28-347">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="a9d28-347">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

1. <span data-ttu-id="a9d28-348">Nella barra laterale **soluzione** fare clic con il pulsante destro del mouse sul `BlazorServerSignalRApp` progetto e scegliere **Gestisci pacchetti NuGet**.</span><span class="sxs-lookup"><span data-stu-id="a9d28-348">In the **Solution** sidebar, right-click the `BlazorServerSignalRApp` project and select **Manage NuGet Packages**.</span></span>

1. <span data-ttu-id="a9d28-349">Nella finestra di dialogo **Gestisci pacchetti NuGet** verificare che l'elenco a discesa origine sia impostato su `nuget.org` .</span><span class="sxs-lookup"><span data-stu-id="a9d28-349">In the **Manage NuGet Packages** dialog, confirm that the source drop-down is set to `nuget.org`.</span></span>

1. <span data-ttu-id="a9d28-350">Con **Sfoglia** selezionato, digitare `System.Text.Encodings.Web` nella casella di ricerca.</span><span class="sxs-lookup"><span data-stu-id="a9d28-350">With **Browse** selected, type `System.Text.Encodings.Web` in the search box.</span></span>

1. <span data-ttu-id="a9d28-351">Nei risultati della ricerca selezionare la casella di controllo accanto al [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web) pacchetto, selezionare la versione corretta del pacchetto corrispondente al Framework condiviso in uso e selezionare **Aggiungi pacchetto**.</span><span class="sxs-lookup"><span data-stu-id="a9d28-351">In the search results, select the check box next to the [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web) package, select the correct version of the package that matches the shared framework in use, and select **Add Package**.</span></span>

1. <span data-ttu-id="a9d28-352">Se viene visualizzata la finestra di dialogo **accettazione della licenza** , selezionare **Accetto** se si accettano le condizioni di licenza.</span><span class="sxs-lookup"><span data-stu-id="a9d28-352">If the **License Acceptance** dialog appears, select **Accept** if you agree with the license terms.</span></span>

# <a name="net-core-cli"></a>[<span data-ttu-id="a9d28-353">Interfaccia della riga di comando di .NET Core</span><span class="sxs-lookup"><span data-stu-id="a9d28-353">.NET Core CLI</span></span>](#tab/netcore-cli/)

<span data-ttu-id="a9d28-354">In una shell dei comandi dalla cartella del progetto, eseguire il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="a9d28-354">In a command shell from the project's folder, execute the following command:</span></span>

```dotnetcli
dotnet add package System.Text.Encodings.Web
```

<span data-ttu-id="a9d28-355">Per aggiungere una versione precedente del pacchetto, specificare l' `--version {VERSION}` opzione, dove il `{VERSION}` segnaposto è la versione del pacchetto da aggiungere.</span><span class="sxs-lookup"><span data-stu-id="a9d28-355">To add an earlier version of the package, supply the `--version {VERSION}` option, where the `{VERSION}` placeholder is the version of the package to add.</span></span>

---

::: moniker-end

## <a name="add-a-signalr-hub"></a><span data-ttu-id="a9d28-356">Aggiungere un SignalR Hub</span><span class="sxs-lookup"><span data-stu-id="a9d28-356">Add a SignalR hub</span></span>

<span data-ttu-id="a9d28-357">Creare una `Hubs` cartella (plurale) e aggiungere la `ChatHub` classe seguente ( `Hubs/ChatHub.cs` ):</span><span class="sxs-lookup"><span data-stu-id="a9d28-357">Create a `Hubs` (plural) folder and add the following `ChatHub` class (`Hubs/ChatHub.cs`):</span></span>

::: moniker range=">= aspnetcore-5.0"

[!code-csharp[](~/tutorials/signalr-blazor/samples/5.x/BlazorServerSignalRApp/Hubs/ChatHub.cs)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-csharp[](~/tutorials/signalr-blazor/samples/3.x/BlazorServerSignalRApp/Hubs/ChatHub.cs)]

::: moniker-end

## <a name="add-services-and-an-endpoint-for-the-signalr-hub"></a><span data-ttu-id="a9d28-358">Aggiungere servizi e un endpoint per l' SignalR Hub</span><span class="sxs-lookup"><span data-stu-id="a9d28-358">Add services and an endpoint for the SignalR hub</span></span>

1. <span data-ttu-id="a9d28-359">Aprire il file `Startup.cs`.</span><span class="sxs-lookup"><span data-stu-id="a9d28-359">Open the `Startup.cs` file.</span></span>

1. <span data-ttu-id="a9d28-360">Aggiungere gli spazi dei nomi per <xref:Microsoft.AspNetCore.ResponseCompression?displayProperty=fullName> e la `ChatHub` classe all'inizio del file:</span><span class="sxs-lookup"><span data-stu-id="a9d28-360">Add the namespaces for <xref:Microsoft.AspNetCore.ResponseCompression?displayProperty=fullName> and the `ChatHub` class to the top of the file:</span></span>

   ```csharp
   using Microsoft.AspNetCore.ResponseCompression;
   using BlazorServerSignalRApp.Server.Hubs;
   ```

::: moniker range=">= aspnetcore-5.0"

1. <span data-ttu-id="a9d28-361">Aggiungere i servizi middleware della compressione della risposta a `Startup.ConfigureServices` :</span><span class="sxs-lookup"><span data-stu-id="a9d28-361">Add Response Compression Middleware services to `Startup.ConfigureServices`:</span></span>

   [!code-csharp[](~/tutorials/signalr-blazor/samples/5.x/BlazorServerSignalRApp/Startup.cs?name=snippet_ConfigureServices&highlight=6-10)]

1. <span data-ttu-id="a9d28-362">In `Startup.Configure`:</span><span class="sxs-lookup"><span data-stu-id="a9d28-362">In `Startup.Configure`:</span></span>

   * <span data-ttu-id="a9d28-363">Usare il middleware della compressione della risposta nella parte superiore della configurazione della pipeline di elaborazione.</span><span class="sxs-lookup"><span data-stu-id="a9d28-363">Use Response Compression Middleware at the top of the processing pipeline's configuration.</span></span>
   * <span data-ttu-id="a9d28-364">Tra gli endpoint per il mapping dell' Blazor Hub e del fallback lato client, aggiungere un endpoint per l'hub.</span><span class="sxs-lookup"><span data-stu-id="a9d28-364">Between the endpoints for mapping the Blazor hub and the client-side fallback, add an endpoint for the hub.</span></span>

   [!code-csharp[](~/tutorials/signalr-blazor/samples/5.x/BlazorServerSignalRApp/Startup.cs?name=snippet_Configure&highlight=3,23)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

1. <span data-ttu-id="a9d28-365">Aggiungere i servizi middleware della compressione della risposta a `Startup.ConfigureServices` :</span><span class="sxs-lookup"><span data-stu-id="a9d28-365">Add Response Compression Middleware services to `Startup.ConfigureServices`:</span></span>

   [!code-csharp[](~/tutorials/signalr-blazor/samples/3.x/BlazorServerSignalRApp/Startup.cs?name=snippet_ConfigureServices&highlight=6-10)]

1. <span data-ttu-id="a9d28-366">In `Startup.Configure`:</span><span class="sxs-lookup"><span data-stu-id="a9d28-366">In `Startup.Configure`:</span></span>

   * <span data-ttu-id="a9d28-367">Usare il middleware della compressione della risposta nella parte superiore della configurazione della pipeline di elaborazione.</span><span class="sxs-lookup"><span data-stu-id="a9d28-367">Use Response Compression Middleware at the top of the processing pipeline's configuration.</span></span>
   * <span data-ttu-id="a9d28-368">Tra gli endpoint per il mapping dell' Blazor Hub e del fallback lato client, aggiungere un endpoint per l'hub.</span><span class="sxs-lookup"><span data-stu-id="a9d28-368">Between the endpoints for mapping the Blazor hub and the client-side fallback, add an endpoint for the hub.</span></span>

   [!code-csharp[](~/tutorials/signalr-blazor/samples/3.x/BlazorServerSignalRApp/Startup.cs?name=snippet_Configure&highlight=3,23)]

::: moniker-end

## <a name="add-razor-component-code-for-chat"></a><span data-ttu-id="a9d28-369">Aggiungi Razor codice componente per la chat</span><span class="sxs-lookup"><span data-stu-id="a9d28-369">Add Razor component code for chat</span></span>

1. <span data-ttu-id="a9d28-370">Aprire il file `Pages/Index.razor`.</span><span class="sxs-lookup"><span data-stu-id="a9d28-370">Open the `Pages/Index.razor` file.</span></span>

::: moniker range=">= aspnetcore-5.0"

1. <span data-ttu-id="a9d28-371">Sostituire il markup con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="a9d28-371">Replace the markup with the following code:</span></span>

   [!code-razor[](~/tutorials/signalr-blazor/samples/5.x/BlazorServerSignalRApp/Pages/Index.razor)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

1. <span data-ttu-id="a9d28-372">Sostituire il markup con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="a9d28-372">Replace the markup with the following code:</span></span>

   [!code-razor[](~/tutorials/signalr-blazor/samples/3.x/BlazorServerSignalRApp/Pages/Index.razor)]

::: moniker-end

## <a name="run-the-app"></a><span data-ttu-id="a9d28-373">Eseguire l'app</span><span class="sxs-lookup"><span data-stu-id="a9d28-373">Run the app</span></span>

<span data-ttu-id="a9d28-374">Seguire le istruzioni per gli strumenti:</span><span class="sxs-lookup"><span data-stu-id="a9d28-374">Follow the guidance for your tooling:</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="a9d28-375">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="a9d28-375">Visual Studio</span></span>](#tab/visual-studio)

1. <span data-ttu-id="a9d28-376">Premere <kbd>F5</kbd> per eseguire l'app con il debug o <kbd>CTRL</kbd> + <kbd>F5</kbd> per eseguire l'app senza debug.</span><span class="sxs-lookup"><span data-stu-id="a9d28-376">Press <kbd>F5</kbd> to run the app with debugging or <kbd>Ctrl</kbd>+<kbd>F5</kbd> to run the app without debugging.</span></span>

1. <span data-ttu-id="a9d28-377">Copiare l'URL dalla barra degli indirizzi, aprire un'altra istanza o scheda del browser e incollare l'URL nella barra degli indirizzi.</span><span class="sxs-lookup"><span data-stu-id="a9d28-377">Copy the URL from the address bar, open another browser instance or tab, and paste the URL in the address bar.</span></span>

1. <span data-ttu-id="a9d28-378">Scegliere browser, immettere un nome e un messaggio e selezionare il pulsante per l'invio del messaggio.</span><span class="sxs-lookup"><span data-stu-id="a9d28-378">Choose either browser, enter a name and message, and select the button to send the message.</span></span> <span data-ttu-id="a9d28-379">Il nome e il messaggio vengono visualizzati immediatamente in entrambe le pagine:</span><span class="sxs-lookup"><span data-stu-id="a9d28-379">The name and message are displayed on both pages instantly:</span></span>

   ![::: NO-LOC (SignalR):::::: NO-LOC (Blazer)::: app di esempio aperta in due finestre del browser che mostrano i messaggi scambiati.](signalr-blazor/_static/signalr-blazor-finished.png)

   <span data-ttu-id="a9d28-381">Virgolette: *Star Trek VI: il paese non individuato* &copy; 1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)</span><span class="sxs-lookup"><span data-stu-id="a9d28-381">Quotes: *Star Trek VI: The Undiscovered Country* &copy;1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="a9d28-382">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="a9d28-382">Visual Studio Code</span></span>](#tab/visual-studio-code)

1. <span data-ttu-id="a9d28-383">Premere <kbd>F5</kbd> per eseguire l'app con il debug o <kbd>CTRL</kbd> + <kbd>F5</kbd> per eseguire l'app senza debug.</span><span class="sxs-lookup"><span data-stu-id="a9d28-383">Press <kbd>F5</kbd> to run the app with debugging or <kbd>Ctrl</kbd>+<kbd>F5</kbd> to run the app without debugging.</span></span>

1. <span data-ttu-id="a9d28-384">Copiare l'URL dalla barra degli indirizzi, aprire un'altra istanza o scheda del browser e incollare l'URL nella barra degli indirizzi.</span><span class="sxs-lookup"><span data-stu-id="a9d28-384">Copy the URL from the address bar, open another browser instance or tab, and paste the URL in the address bar.</span></span>

1. <span data-ttu-id="a9d28-385">Scegliere browser, immettere un nome e un messaggio e selezionare il pulsante per l'invio del messaggio.</span><span class="sxs-lookup"><span data-stu-id="a9d28-385">Choose either browser, enter a name and message, and select the button to send the message.</span></span> <span data-ttu-id="a9d28-386">Il nome e il messaggio vengono visualizzati immediatamente in entrambe le pagine:</span><span class="sxs-lookup"><span data-stu-id="a9d28-386">The name and message are displayed on both pages instantly:</span></span>

   ![::: NO-LOC (SignalR):::::: NO-LOC (Blazer)::: app di esempio aperta in due finestre del browser che mostrano i messaggi scambiati.](signalr-blazor/_static/signalr-blazor-finished.png)

   <span data-ttu-id="a9d28-388">Virgolette: *Star Trek VI: il paese non individuato* &copy; 1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)</span><span class="sxs-lookup"><span data-stu-id="a9d28-388">Quotes: *Star Trek VI: The Undiscovered Country* &copy;1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)</span></span>

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="a9d28-389">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="a9d28-389">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

1. <span data-ttu-id="a9d28-390">Premere <kbd>⌘</kbd> + <kbd>↩</kbd> per eseguire l'app con debug o <kbd>⌥</kbd> + <kbd>⌘</kbd> + <kbd>↩</kbd> per eseguire l'app senza debug.</span><span class="sxs-lookup"><span data-stu-id="a9d28-390">Press <kbd>⌘</kbd>+<kbd>↩</kbd> to run the app with debugging or <kbd>⌥</kbd>+<kbd>⌘</kbd>+<kbd>↩</kbd> to run the app without debugging.</span></span>

1. <span data-ttu-id="a9d28-391">Copiare l'URL dalla barra degli indirizzi, aprire un'altra istanza o scheda del browser e incollare l'URL nella barra degli indirizzi.</span><span class="sxs-lookup"><span data-stu-id="a9d28-391">Copy the URL from the address bar, open another browser instance or tab, and paste the URL in the address bar.</span></span>

1. <span data-ttu-id="a9d28-392">Scegliere browser, immettere un nome e un messaggio e selezionare il pulsante per l'invio del messaggio.</span><span class="sxs-lookup"><span data-stu-id="a9d28-392">Choose either browser, enter a name and message, and select the button to send the message.</span></span> <span data-ttu-id="a9d28-393">Il nome e il messaggio vengono visualizzati immediatamente in entrambe le pagine:</span><span class="sxs-lookup"><span data-stu-id="a9d28-393">The name and message are displayed on both pages instantly:</span></span>

   ![::: NO-LOC (SignalR):::::: NO-LOC (Blazer)::: app di esempio aperta in due finestre del browser che mostrano i messaggi scambiati.](signalr-blazor/_static/signalr-blazor-finished.png)

   <span data-ttu-id="a9d28-395">Virgolette: *Star Trek VI: il paese non individuato* &copy; 1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)</span><span class="sxs-lookup"><span data-stu-id="a9d28-395">Quotes: *Star Trek VI: The Undiscovered Country* &copy;1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)</span></span>

# <a name="net-core-cli"></a>[<span data-ttu-id="a9d28-396">Interfaccia della riga di comando di .NET Core</span><span class="sxs-lookup"><span data-stu-id="a9d28-396">.NET Core CLI</span></span>](#tab/netcore-cli/)

1. <span data-ttu-id="a9d28-397">In una shell dei comandi dalla cartella del progetto, eseguire i comandi seguenti:</span><span class="sxs-lookup"><span data-stu-id="a9d28-397">In a command shell from the project's folder, execute the following commands:</span></span>

   ```dotnetcli
   dotnet run
   ```

1. <span data-ttu-id="a9d28-398">Copiare l'URL dalla barra degli indirizzi, aprire un'altra istanza o scheda del browser e incollare l'URL nella barra degli indirizzi.</span><span class="sxs-lookup"><span data-stu-id="a9d28-398">Copy the URL from the address bar, open another browser instance or tab, and paste the URL in the address bar.</span></span>

1. <span data-ttu-id="a9d28-399">Scegliere browser, immettere un nome e un messaggio e selezionare il pulsante per l'invio del messaggio.</span><span class="sxs-lookup"><span data-stu-id="a9d28-399">Choose either browser, enter a name and message, and select the button to send the message.</span></span> <span data-ttu-id="a9d28-400">Il nome e il messaggio vengono visualizzati immediatamente in entrambe le pagine:</span><span class="sxs-lookup"><span data-stu-id="a9d28-400">The name and message are displayed on both pages instantly:</span></span>

   ![::: NO-LOC (SignalR):::::: NO-LOC (Blazer)::: app di esempio aperta in due finestre del browser che mostrano i messaggi scambiati.](signalr-blazor/_static/signalr-blazor-finished.png)

   <span data-ttu-id="a9d28-402">Virgolette: *Star Trek VI: il paese non individuato* &copy; 1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)</span><span class="sxs-lookup"><span data-stu-id="a9d28-402">Quotes: *Star Trek VI: The Undiscovered Country* &copy;1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)</span></span>

---

::: zone-end

## <a name="next-steps"></a><span data-ttu-id="a9d28-403">Passaggi successivi</span><span class="sxs-lookup"><span data-stu-id="a9d28-403">Next steps</span></span>

<span data-ttu-id="a9d28-404">In questa esercitazione sono state illustrate le procedure per:</span><span class="sxs-lookup"><span data-stu-id="a9d28-404">In this tutorial, you learned how to:</span></span>

> [!div class="checklist"]
> * <span data-ttu-id="a9d28-405">Creazione di un Blazor progetto</span><span class="sxs-lookup"><span data-stu-id="a9d28-405">Create a Blazor project</span></span>
> * <span data-ttu-id="a9d28-406">Aggiungere la SignalR libreria client</span><span class="sxs-lookup"><span data-stu-id="a9d28-406">Add the SignalR client library</span></span>
> * <span data-ttu-id="a9d28-407">Aggiungere un SignalR Hub</span><span class="sxs-lookup"><span data-stu-id="a9d28-407">Add a SignalR hub</span></span>
> * <span data-ttu-id="a9d28-408">Aggiungere SignalR Servizi e un endpoint per l' SignalR Hub</span><span class="sxs-lookup"><span data-stu-id="a9d28-408">Add SignalR services and an endpoint for the SignalR hub</span></span>
> * <span data-ttu-id="a9d28-409">Aggiungi Razor codice componente per la chat</span><span class="sxs-lookup"><span data-stu-id="a9d28-409">Add Razor component code for chat</span></span>

<span data-ttu-id="a9d28-410">Per altre informazioni sulla creazione di Blazor app, vedere la Blazor documentazione:</span><span class="sxs-lookup"><span data-stu-id="a9d28-410">To learn more about building Blazor apps, see the Blazor documentation:</span></span>

> [!div class="nextstepaction"]
> <span data-ttu-id="a9d28-411"><xref:blazor/index>
> [Autenticazione del token di porta con Identity Server, WebSocket ed eventi di Server-Sent](xref:signalr/authn-and-authz#bearer-token-authentication)</span><span class="sxs-lookup"><span data-stu-id="a9d28-411"><xref:blazor/index>
[Bearer token authentication with Identity Server, WebSockets, and Server-Sent Events](xref:signalr/authn-and-authz#bearer-token-authentication)</span></span>

## <a name="additional-resources"></a><span data-ttu-id="a9d28-412">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="a9d28-412">Additional resources</span></span>

* <xref:signalr/introduction>
* [<span data-ttu-id="a9d28-413">SignalR negoziazione tra le origini per l'autenticazione</span><span class="sxs-lookup"><span data-stu-id="a9d28-413">SignalR cross-origin negotiation for authentication</span></span>](xref:blazor/fundamentals/signalr#signalr-cross-origin-negotiation-for-authentication)
* <xref:blazor/debug>
* <xref:blazor/security/server/threat-mitigation>
