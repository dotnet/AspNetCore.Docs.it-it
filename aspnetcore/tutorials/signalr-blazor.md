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
ms.openlocfilehash: 226813f3b975e207a51eba24d6cbb57f00464073
ms.sourcegitcommit: 04ad9cd26fcaa8bd11e261d3661f375f5f343cdc
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/10/2021
ms.locfileid: "100107207"
---
# <a name="use-aspnet-core-signalr-with-blazor"></a><span data-ttu-id="3fbd9-103">Usare ASP.NET Core SignalR con Blazor</span><span class="sxs-lookup"><span data-stu-id="3fbd9-103">Use ASP.NET Core SignalR with Blazor</span></span>

<span data-ttu-id="3fbd9-104">Di [Daniel Roth](https://github.com/danroth27) e [Luke Latham](https://github.com/guardrex)</span><span class="sxs-lookup"><span data-stu-id="3fbd9-104">By [Daniel Roth](https://github.com/danroth27) and [Luke Latham](https://github.com/guardrex)</span></span>

<span data-ttu-id="3fbd9-105">Questa esercitazione illustra le nozioni di base per la creazione di un'app in tempo reale usando SignalR con Blazor .</span><span class="sxs-lookup"><span data-stu-id="3fbd9-105">This tutorial teaches the basics of building a real-time app using SignalR with Blazor.</span></span> <span data-ttu-id="3fbd9-106">Si apprenderà come:</span><span class="sxs-lookup"><span data-stu-id="3fbd9-106">You learn how to:</span></span>

> [!div class="checklist"]
> * <span data-ttu-id="3fbd9-107">Creazione di un Blazor progetto</span><span class="sxs-lookup"><span data-stu-id="3fbd9-107">Create a Blazor project</span></span>
> * <span data-ttu-id="3fbd9-108">Aggiungere la SignalR libreria client</span><span class="sxs-lookup"><span data-stu-id="3fbd9-108">Add the SignalR client library</span></span>
> * <span data-ttu-id="3fbd9-109">Aggiungere un SignalR Hub</span><span class="sxs-lookup"><span data-stu-id="3fbd9-109">Add a SignalR hub</span></span>
> * <span data-ttu-id="3fbd9-110">Aggiungere SignalR Servizi e un endpoint per l' SignalR Hub</span><span class="sxs-lookup"><span data-stu-id="3fbd9-110">Add SignalR services and an endpoint for the SignalR hub</span></span>
> * <span data-ttu-id="3fbd9-111">Aggiungi Razor codice componente per la chat</span><span class="sxs-lookup"><span data-stu-id="3fbd9-111">Add Razor component code for chat</span></span>

<span data-ttu-id="3fbd9-112">Al termine di questa esercitazione, si disporrà di un'app di chat funzionante.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-112">At the end of this tutorial, you'll have a working chat app.</span></span>

<span data-ttu-id="3fbd9-113">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/tutorials/signalr-blazor/samples/) ([procedura per il download](xref:index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="3fbd9-113">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/tutorials/signalr-blazor/samples/) ([how to download](xref:index#how-to-download-a-sample))</span></span>

## <a name="prerequisites"></a><span data-ttu-id="3fbd9-114">Prerequisiti</span><span class="sxs-lookup"><span data-stu-id="3fbd9-114">Prerequisites</span></span>

::: moniker range=">= aspnetcore-5.0"

# <a name="visual-studio"></a>[<span data-ttu-id="3fbd9-115">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="3fbd9-115">Visual Studio</span></span>](#tab/visual-studio)

* <span data-ttu-id="3fbd9-116">[Visual Studio 2019 16,8 o versione successiva](https://visualstudio.microsoft.com/downloads/?utm_medium=microsoft&utm_source=docs.microsoft.com&utm_campaign=inline+link&utm_content=download+vs2019) con il carico di lavoro **sviluppo di ASP.NET e Web**</span><span class="sxs-lookup"><span data-stu-id="3fbd9-116">[Visual Studio 2019 16.8 or later](https://visualstudio.microsoft.com/downloads/?utm_medium=microsoft&utm_source=docs.microsoft.com&utm_campaign=inline+link&utm_content=download+vs2019) with the **ASP.NET and web development** workload</span></span>
* [!INCLUDE [.NET Core 5.0 SDK](~/includes/5.0-SDK.md)]

# <a name="visual-studio-code"></a>[<span data-ttu-id="3fbd9-117">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="3fbd9-117">Visual Studio Code</span></span>](#tab/visual-studio-code)

[!INCLUDE[](~/includes/net-core-prereqs-vsc-5.0.md)]

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="3fbd9-118">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="3fbd9-118">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

* [<span data-ttu-id="3fbd9-119">Visual Studio per Mac versione 8,8 o successiva</span><span class="sxs-lookup"><span data-stu-id="3fbd9-119">Visual Studio for Mac version 8.8 or later</span></span>](https://visualstudio.microsoft.com/vs/mac/)
* [!INCLUDE [.NET Core 5.0 SDK](~/includes/5.0-SDK.md)]

# <a name="net-core-cli"></a>[<span data-ttu-id="3fbd9-120">Interfaccia della riga di comando di .NET Core</span><span class="sxs-lookup"><span data-stu-id="3fbd9-120">.NET Core CLI</span></span>](#tab/netcore-cli/)

[!INCLUDE[](~/includes/5.0-SDK.md)]

---

::: moniker-end

::: moniker range="< aspnetcore-5.0"

# <a name="visual-studio"></a>[<span data-ttu-id="3fbd9-121">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="3fbd9-121">Visual Studio</span></span>](#tab/visual-studio)

* <span data-ttu-id="3fbd9-122">[Visual Studio 2019 16,6 o versione successiva](https://visualstudio.microsoft.com/downloads/?utm_medium=microsoft&utm_source=docs.microsoft.com&utm_campaign=inline+link&utm_content=download+vs2019) con il carico di lavoro **sviluppo di ASP.NET e Web**</span><span class="sxs-lookup"><span data-stu-id="3fbd9-122">[Visual Studio 2019 16.6 or later](https://visualstudio.microsoft.com/downloads/?utm_medium=microsoft&utm_source=docs.microsoft.com&utm_campaign=inline+link&utm_content=download+vs2019) with the **ASP.NET and web development** workload</span></span>
* [!INCLUDE [.NET Core 3.1 SDK](~/includes/3.1-SDK.md)]

# <a name="visual-studio-code"></a>[<span data-ttu-id="3fbd9-123">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="3fbd9-123">Visual Studio Code</span></span>](#tab/visual-studio-code)

[!INCLUDE[](~/includes/net-core-prereqs-vsc-3.1.md)]

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="3fbd9-124">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="3fbd9-124">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

* [<span data-ttu-id="3fbd9-125">Visual Studio per Mac versione 8,6 o successiva</span><span class="sxs-lookup"><span data-stu-id="3fbd9-125">Visual Studio for Mac version 8.6 or later</span></span>](https://visualstudio.microsoft.com/vs/mac/)
* [!INCLUDE [.NET Core 3.1 SDK](~/includes/3.1-SDK.md)]

# <a name="net-core-cli"></a>[<span data-ttu-id="3fbd9-126">Interfaccia della riga di comando di .NET Core</span><span class="sxs-lookup"><span data-stu-id="3fbd9-126">.NET Core CLI</span></span>](#tab/netcore-cli/)

[!INCLUDE[](~/includes/3.1-SDK.md)]

---

::: moniker-end

::: zone pivot="webassembly"

## <a name="create-a-hosted-blazor-webassembly-app"></a><span data-ttu-id="3fbd9-127">Creare un'app ospitata Blazor WebAssembly</span><span class="sxs-lookup"><span data-stu-id="3fbd9-127">Create a hosted Blazor WebAssembly app</span></span>

<span data-ttu-id="3fbd9-128">Seguire le istruzioni per la scelta degli strumenti:</span><span class="sxs-lookup"><span data-stu-id="3fbd9-128">Follow the guidance for your choice of tooling:</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="3fbd9-129">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="3fbd9-129">Visual Studio</span></span>](#tab/visual-studio)

::: moniker range=">= aspnetcore-5.0"

> [!NOTE]
> <span data-ttu-id="3fbd9-130">Sono necessari Visual Studio 16,8 o versioni successive e .NET Core SDK 5.0.0 o versioni successive.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-130">Visual Studio 16.8 or later and .NET Core SDK 5.0.0 or later are required.</span></span>

::: moniker-end

::: moniker range="< aspnetcore-5.0"

> [!NOTE]
> <span data-ttu-id="3fbd9-131">Sono necessari Visual Studio 16,6 o versioni successive e .NET Core SDK 3.1.300 o versioni successive.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-131">Visual Studio 16.6 or later and .NET Core SDK 3.1.300 or later are required.</span></span>

::: moniker-end

1. <span data-ttu-id="3fbd9-132">Creare un nuovo progetto.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-132">Create a new project.</span></span>

1. <span data-ttu-id="3fbd9-133">Selezionare **Blazor app** e fare clic su **Next (avanti**).</span><span class="sxs-lookup"><span data-stu-id="3fbd9-133">Select **Blazor App** and select **Next**.</span></span>

1. <span data-ttu-id="3fbd9-134">Digitare `BlazorWebAssemblySignalRApp` nel campo **nome progetto** .</span><span class="sxs-lookup"><span data-stu-id="3fbd9-134">Type `BlazorWebAssemblySignalRApp` in the **Project name** field.</span></span> <span data-ttu-id="3fbd9-135">Confermare che la voce relativa al **percorso** sia corretta o specificare un percorso per il progetto.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-135">Confirm the **Location** entry is correct or provide a location for the project.</span></span> <span data-ttu-id="3fbd9-136">Selezionare **Crea**.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-136">Select **Create**.</span></span>

1. <span data-ttu-id="3fbd9-137">Scegliere il modello di **Blazor WebAssembly app** .</span><span class="sxs-lookup"><span data-stu-id="3fbd9-137">Choose the **Blazor WebAssembly App** template.</span></span>

1. <span data-ttu-id="3fbd9-138">In **Avanzate** selezionare la casella di controllo **ASP.NET Core Hosted** .</span><span class="sxs-lookup"><span data-stu-id="3fbd9-138">Under **Advanced**, select the **ASP.NET Core hosted** check box.</span></span>

1. <span data-ttu-id="3fbd9-139">Selezionare **Crea**.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-139">Select **Create**.</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="3fbd9-140">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="3fbd9-140">Visual Studio Code</span></span>](#tab/visual-studio-code)

1. <span data-ttu-id="3fbd9-141">In una shell dei comandi eseguire il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="3fbd9-141">In a command shell, execute the following command:</span></span>

   ```dotnetcli
   dotnet new blazorwasm -ho -o BlazorWebAssemblySignalRApp
   ```

   <span data-ttu-id="3fbd9-142">L' `-ho|--hosted` opzione Crea una soluzione ospitata Blazor WebAssembly .</span><span class="sxs-lookup"><span data-stu-id="3fbd9-142">The `-ho|--hosted` option creates a hosted Blazor WebAssembly solution.</span></span>

   <span data-ttu-id="3fbd9-143">L' `-o|--output` opzione Crea una cartella per la soluzione.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-143">The `-o|--output` option creates a folder for the solution.</span></span> <span data-ttu-id="3fbd9-144">Se è stata creata una cartella per la soluzione e la shell dei comandi è aperta in tale cartella, omettere l' `-o|--output` opzione e il valore per creare la soluzione.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-144">If you've created a folder for the solution and the command shell is open in that folder, omit the `-o|--output` option and value to create the solution.</span></span>

1. <span data-ttu-id="3fbd9-145">In Visual Studio Code aprire la cartella del progetto dell'app.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-145">In Visual Studio Code, open the app's project folder.</span></span>

1. <span data-ttu-id="3fbd9-146">Quando viene visualizzata la finestra di dialogo per aggiungere asset per compilare ed eseguire il debug dell'app, selezionare **Sì**.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-146">When the dialog appears to add assets to build and debug the app, select **Yes**.</span></span> <span data-ttu-id="3fbd9-147">Visual Studio Code aggiunge automaticamente la `.vscode` cartella con `launch.json` `tasks.json` i file e generati.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-147">Visual Studio Code automatically adds the `.vscode` folder with generated `launch.json` and `tasks.json` files.</span></span>

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="3fbd9-148">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="3fbd9-148">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

1. <span data-ttu-id="3fbd9-149">Installare la versione più recente di [Visual Studio per Mac](https://visualstudio.microsoft.com/vs/mac/) e seguire questa procedura:</span><span class="sxs-lookup"><span data-stu-id="3fbd9-149">Install the latest version of [Visual Studio for Mac](https://visualstudio.microsoft.com/vs/mac/) and perform the following steps:</span></span>

1. <span data-ttu-id="3fbd9-150">Selezionare **file**  >  **nuova soluzione** o creare un **nuovo** progetto dalla **finestra Start**.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-150">Select **File** > **New Solution** or create a **New** project from the **Start Window**.</span></span>

1. <span data-ttu-id="3fbd9-151">Nella barra laterale selezionare app **Web e console**  >  .</span><span class="sxs-lookup"><span data-stu-id="3fbd9-151">In the sidebar, select **Web and Console** > **App**.</span></span>

1. <span data-ttu-id="3fbd9-152">Scegliere il modello di **Blazor WebAssembly app** .</span><span class="sxs-lookup"><span data-stu-id="3fbd9-152">Choose the **Blazor WebAssembly App** template.</span></span> <span data-ttu-id="3fbd9-153">Selezionare **Avanti**.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-153">Select **Next**.</span></span>

1. <span data-ttu-id="3fbd9-154">Verificare che **l'autenticazione** sia impostata su **Nessuna autenticazione**.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-154">Confirm that **Authentication** is set to **No Authentication**.</span></span> <span data-ttu-id="3fbd9-155">Selezionare la casella di controllo **ASP.NET Core Hosted** .</span><span class="sxs-lookup"><span data-stu-id="3fbd9-155">Select the **ASP.NET Core Hosted** check box.</span></span> <span data-ttu-id="3fbd9-156">Selezionare **Avanti**.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-156">Select **Next**.</span></span>

1. <span data-ttu-id="3fbd9-157">Nel campo **nome progetto** assegnare un nome all'app `BlazorWebAssemblySignalRApp` .</span><span class="sxs-lookup"><span data-stu-id="3fbd9-157">In the **Project Name** field, name the app `BlazorWebAssemblySignalRApp`.</span></span> <span data-ttu-id="3fbd9-158">Selezionare **Crea**.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-158">Select **Create**.</span></span>

   <span data-ttu-id="3fbd9-159">Se viene visualizzato un messaggio per considerare attendibile il certificato di sviluppo, considerare attendibile il certificato e continuare.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-159">If a prompt appears to trust the development certificate, trust the certificate and continue.</span></span> <span data-ttu-id="3fbd9-160">Per considerare attendibile il certificato, è necessario specificare le password dell'utente e del keychain.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-160">The user and keychain passwords are required to trust the certificate.</span></span>

1. <span data-ttu-id="3fbd9-161">Aprire il progetto passando alla cartella del progetto e aprendo il file di soluzione del progetto ( `.sln` ).</span><span class="sxs-lookup"><span data-stu-id="3fbd9-161">Open the project by navigating to the project folder and opening the project's solution file (`.sln`).</span></span>

# <a name="net-core-cli"></a>[<span data-ttu-id="3fbd9-162">Interfaccia della riga di comando di .NET Core</span><span class="sxs-lookup"><span data-stu-id="3fbd9-162">.NET Core CLI</span></span>](#tab/netcore-cli/)

<span data-ttu-id="3fbd9-163">In una shell dei comandi eseguire il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="3fbd9-163">In a command shell, execute the following command:</span></span>

```dotnetcli
dotnet new blazorwasm -ho -o BlazorWebAssemblySignalRApp
```

<span data-ttu-id="3fbd9-164">L' `-ho|--hosted` opzione Crea una soluzione ospitata Blazor WebAssembly .</span><span class="sxs-lookup"><span data-stu-id="3fbd9-164">The `-ho|--hosted` option creates a hosted Blazor WebAssembly solution.</span></span>

<span data-ttu-id="3fbd9-165">L' `-o|--output` opzione Crea una cartella per la soluzione.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-165">The `-o|--output` option creates a folder for the solution.</span></span> <span data-ttu-id="3fbd9-166">Se è stata creata una cartella per la soluzione e la shell dei comandi è aperta in tale cartella, omettere l' `-o|--output` opzione e il valore per creare la soluzione.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-166">If you've created a folder for the solution and the command shell is open in that folder, omit the `-o|--output` option and value to create the solution.</span></span>

---

## <a name="add-the-signalr-client-library"></a><span data-ttu-id="3fbd9-167">Aggiungere la SignalR libreria client</span><span class="sxs-lookup"><span data-stu-id="3fbd9-167">Add the SignalR client library</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="3fbd9-168">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="3fbd9-168">Visual Studio</span></span>](#tab/visual-studio/)

1. <span data-ttu-id="3fbd9-169">In **Esplora soluzioni** fare clic con il pulsante destro del mouse sul `BlazorWebAssemblySignalRApp.Client` progetto e scegliere **Gestisci pacchetti NuGet**.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-169">In **Solution Explorer**, right-click the `BlazorWebAssemblySignalRApp.Client` project and select **Manage NuGet Packages**.</span></span>

1. <span data-ttu-id="3fbd9-170">Nella finestra di dialogo **Gestisci pacchetti NuGet** verificare che l' **origine del pacchetto** sia impostata su `nuget.org` .</span><span class="sxs-lookup"><span data-stu-id="3fbd9-170">In the **Manage NuGet Packages** dialog, confirm that the **Package source** is set to `nuget.org`.</span></span>

1. <span data-ttu-id="3fbd9-171">Con **Sfoglia** selezionato, digitare `Microsoft.AspNetCore.SignalR.Client` nella casella di ricerca.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-171">With **Browse** selected, type `Microsoft.AspNetCore.SignalR.Client` in the search box.</span></span>

1. <span data-ttu-id="3fbd9-172">Nei risultati della ricerca selezionare il [`Microsoft.AspNetCore.SignalR.Client`](https://www.nuget.org/packages/Microsoft.AspNetCore.SignalR.Client) pacchetto.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-172">In the search results, select the [`Microsoft.AspNetCore.SignalR.Client`](https://www.nuget.org/packages/Microsoft.AspNetCore.SignalR.Client) package.</span></span> <span data-ttu-id="3fbd9-173">Impostare la versione in modo che corrisponda al Framework condiviso dell'app.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-173">Set the version to match the shared framework of the app.</span></span> <span data-ttu-id="3fbd9-174">Selezionare **Installa**.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-174">Select **Install**.</span></span>

1. <span data-ttu-id="3fbd9-175">Se viene visualizzata la finestra di dialogo **Anteprima modifiche** , selezionare **OK**.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-175">If the **Preview Changes** dialog appears, select **OK**.</span></span>

1. <span data-ttu-id="3fbd9-176">Se viene visualizzata la finestra di dialogo **accettazione della licenza** , selezionare Accetto se **si accettano le** condizioni di licenza.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-176">If the **License Acceptance** dialog appears, select **I Accept** if you agree with the license terms.</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="3fbd9-177">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="3fbd9-177">Visual Studio Code</span></span>](#tab/visual-studio-code/)

<span data-ttu-id="3fbd9-178">Nel **terminale integrato** (**visualizzare** il  >  **terminale** dalla barra degli strumenti) eseguire il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="3fbd9-178">In the **Integrated Terminal** (**View** > **Terminal** from the toolbar), execute the following command:</span></span>

```dotnetcli
dotnet add Client package Microsoft.AspNetCore.SignalR.Client
```

<span data-ttu-id="3fbd9-179">Per aggiungere una versione precedente del pacchetto, specificare l' `--version {VERSION}` opzione, dove il `{VERSION}` segnaposto è la versione del pacchetto da aggiungere.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-179">To add an earlier version of the package, supply the `--version {VERSION}` option, where the `{VERSION}` placeholder is the version of the package to add.</span></span>

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="3fbd9-180">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="3fbd9-180">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

1. <span data-ttu-id="3fbd9-181">Nella barra laterale **soluzione** fare clic con il pulsante destro del mouse sul `BlazorWebAssemblySignalRApp.Client` progetto e scegliere **Gestisci pacchetti NuGet**.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-181">In the **Solution** sidebar, right-click the `BlazorWebAssemblySignalRApp.Client` project and select **Manage NuGet Packages**.</span></span>

1. <span data-ttu-id="3fbd9-182">Nella finestra di dialogo **Gestisci pacchetti NuGet** verificare che l'elenco a discesa origine sia impostato su `nuget.org` .</span><span class="sxs-lookup"><span data-stu-id="3fbd9-182">In the **Manage NuGet Packages** dialog, confirm that the source drop-down is set to `nuget.org`.</span></span>

1. <span data-ttu-id="3fbd9-183">Con **Sfoglia** selezionato, digitare `Microsoft.AspNetCore.SignalR.Client` nella casella di ricerca.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-183">With **Browse** selected, type `Microsoft.AspNetCore.SignalR.Client` in the search box.</span></span>

1. <span data-ttu-id="3fbd9-184">Nei risultati della ricerca selezionare la casella di controllo accanto al [`Microsoft.AspNetCore.SignalR.Client`](https://www.nuget.org/packages/Microsoft.AspNetCore.SignalR.Client) pacchetto.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-184">In the search results, select the check box next to the [`Microsoft.AspNetCore.SignalR.Client`](https://www.nuget.org/packages/Microsoft.AspNetCore.SignalR.Client) package.</span></span> <span data-ttu-id="3fbd9-185">Impostare la versione in modo che corrisponda al Framework condiviso dell'app.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-185">Set the version to match the shared framework of the app.</span></span> <span data-ttu-id="3fbd9-186">Selezionare **Aggiungi pacchetto**.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-186">Select **Add Package**.</span></span>

1. <span data-ttu-id="3fbd9-187">Se viene visualizzata la finestra di dialogo **accettazione della licenza** , selezionare **Accetto** se si accettano le condizioni di licenza.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-187">If the **License Acceptance** dialog appears, select **Accept** if you agree with the license terms.</span></span>

# <a name="net-core-cli"></a>[<span data-ttu-id="3fbd9-188">Interfaccia della riga di comando di .NET Core</span><span class="sxs-lookup"><span data-stu-id="3fbd9-188">.NET Core CLI</span></span>](#tab/netcore-cli/)

<span data-ttu-id="3fbd9-189">In una shell dei comandi dalla cartella della soluzione, eseguire il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="3fbd9-189">In a command shell from the solution's folder, execute the following command:</span></span>

```dotnetcli
dotnet add Client package Microsoft.AspNetCore.SignalR.Client
```

<span data-ttu-id="3fbd9-190">Per aggiungere una versione precedente del pacchetto, specificare l' `--version {VERSION}` opzione, dove il `{VERSION}` segnaposto è la versione del pacchetto da aggiungere.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-190">To add an earlier version of the package, supply the `--version {VERSION}` option, where the `{VERSION}` placeholder is the version of the package to add.</span></span>

---

::: moniker range="< aspnetcore-5.0"

## <a name="add-the-systemtextencodingsweb-package"></a><span data-ttu-id="3fbd9-191">Aggiungere il pacchetto System. Text. Encodings. Web</span><span class="sxs-lookup"><span data-stu-id="3fbd9-191">Add the System.Text.Encodings.Web package</span></span>

<span data-ttu-id="3fbd9-192">*Questa sezione si applica solo alle app per ASP.NET Core versione 3. x.*</span><span class="sxs-lookup"><span data-stu-id="3fbd9-192">*This section only applies to apps for ASP.NET Core version 3.x.*</span></span>

<span data-ttu-id="3fbd9-193">A causa di un problema di risoluzione del pacchetto quando si usa [`System.Text.Json`](https://www.nuget.org/packages/System.Text.Json) 5. x in un'app ASP.NET Core 3. x, il `BlazorWebAssemblySignalRApp.Server` progetto richiede un riferimento al pacchetto per [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web) .</span><span class="sxs-lookup"><span data-stu-id="3fbd9-193">Due to a package resolution issue when using [`System.Text.Json`](https://www.nuget.org/packages/System.Text.Json) 5.x in an ASP.NET Core 3.x app, the `BlazorWebAssemblySignalRApp.Server` project requires a package reference for [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web).</span></span> <span data-ttu-id="3fbd9-194">Il problema sottostante verrà risolto in una versione futura della patch di .NET 5.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-194">The underlying issue will be resolved in a future patch release of .NET 5.</span></span> <span data-ttu-id="3fbd9-195">Per altre informazioni, vedere [System.Text.Json definisce netcoreapp 3.0 senza dipendenze (DotNet/runtime #45560)](https://github.com/dotnet/runtime/issues/45560).</span><span class="sxs-lookup"><span data-stu-id="3fbd9-195">For more information, see [System.Text.Json defines netcoreapp3.0 with no dependencies (dotnet/runtime #45560)](https://github.com/dotnet/runtime/issues/45560).</span></span>

<span data-ttu-id="3fbd9-196">Per aggiungere [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web) al `BlazorWebAssemblySignalRApp.Server` progetto della soluzione ospitata ASP.NET Core 3,1 Blazor , seguire le istruzioni per la scelta degli strumenti:</span><span class="sxs-lookup"><span data-stu-id="3fbd9-196">To add [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web) to the `BlazorWebAssemblySignalRApp.Server` project of the ASP.NET Core 3.1 hosted Blazor solution, follow the guidance for your choice of tooling:</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="3fbd9-197">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="3fbd9-197">Visual Studio</span></span>](#tab/visual-studio/)

1. <span data-ttu-id="3fbd9-198">In **Esplora soluzioni** fare clic con il pulsante destro del mouse sul `BlazorWebAssemblySignalRApp.Server` progetto e scegliere **Gestisci pacchetti NuGet**.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-198">In **Solution Explorer**, right-click the `BlazorWebAssemblySignalRApp.Server` project and select **Manage NuGet Packages**.</span></span>

1. <span data-ttu-id="3fbd9-199">Nella finestra di dialogo **Gestisci pacchetti NuGet** verificare che l' **origine del pacchetto** sia impostata su `nuget.org` .</span><span class="sxs-lookup"><span data-stu-id="3fbd9-199">In the **Manage NuGet Packages** dialog, confirm that the **Package source** is set to `nuget.org`.</span></span>

1. <span data-ttu-id="3fbd9-200">Con **Sfoglia** selezionato, digitare `System.Text.Encodings.Web` nella casella di ricerca.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-200">With **Browse** selected, type `System.Text.Encodings.Web` in the search box.</span></span>

1. <span data-ttu-id="3fbd9-201">Nei risultati della ricerca selezionare il [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web) pacchetto.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-201">In the search results, select the [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web) package.</span></span> <span data-ttu-id="3fbd9-202">Consente di selezionare la versione del pacchetto corrispondente al Framework condiviso in uso.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-202">Select the version of the package that matches the shared framework in use.</span></span> <span data-ttu-id="3fbd9-203">Selezionare **Installa**.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-203">Select **Install**.</span></span>

1. <span data-ttu-id="3fbd9-204">Se viene visualizzata la finestra di dialogo **Anteprima modifiche** , selezionare **OK**.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-204">If the **Preview Changes** dialog appears, select **OK**.</span></span>

1. <span data-ttu-id="3fbd9-205">Se viene visualizzata la finestra di dialogo **accettazione della licenza** , selezionare Accetto se **si accettano le** condizioni di licenza.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-205">If the **License Acceptance** dialog appears, select **I Accept** if you agree with the license terms.</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="3fbd9-206">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="3fbd9-206">Visual Studio Code</span></span>](#tab/visual-studio-code/)

<span data-ttu-id="3fbd9-207">Nel **terminale integrato** (**visualizzare** il > **terminale** dalla barra degli strumenti) eseguire i comandi seguenti:</span><span class="sxs-lookup"><span data-stu-id="3fbd9-207">In the **Integrated Terminal** (**View** > **Terminal** from the toolbar), execute the following commands:</span></span>

```dotnetcli
dotnet add Server package System.Text.Encodings.Web
```

<span data-ttu-id="3fbd9-208">Per aggiungere una versione precedente del pacchetto, specificare l' `--version {VERSION}` opzione, dove il `{VERSION}` segnaposto è la versione del pacchetto da aggiungere.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-208">To add an earlier version of the package, supply the `--version {VERSION}` option, where the `{VERSION}` placeholder is the version of the package to add.</span></span>

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="3fbd9-209">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="3fbd9-209">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

1. <span data-ttu-id="3fbd9-210">Nella barra laterale **soluzione** fare clic con il pulsante destro del mouse sul `BlazorWebAssemblySignalRApp.Server` progetto e scegliere **Gestisci pacchetti NuGet**.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-210">In the **Solution** sidebar, right-click the `BlazorWebAssemblySignalRApp.Server` project and select **Manage NuGet Packages**.</span></span>

1. <span data-ttu-id="3fbd9-211">Nella finestra di dialogo **Gestisci pacchetti NuGet** verificare che l'elenco a discesa origine sia impostato su `nuget.org` .</span><span class="sxs-lookup"><span data-stu-id="3fbd9-211">In the **Manage NuGet Packages** dialog, confirm that the source drop-down is set to `nuget.org`.</span></span>

1. <span data-ttu-id="3fbd9-212">Con **Sfoglia** selezionato, digitare `System.Text.Encodings.Web` nella casella di ricerca.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-212">With **Browse** selected, type `System.Text.Encodings.Web` in the search box.</span></span>

1. <span data-ttu-id="3fbd9-213">Nei risultati della ricerca selezionare la casella di controllo accanto al [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web) pacchetto, selezionare la versione corretta del pacchetto corrispondente al Framework condiviso in uso e selezionare **Aggiungi pacchetto**.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-213">In the search results, select the check box next to the [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web) package, select the correct version of the package that matches the shared framework in use, and select **Add Package**.</span></span>

1. <span data-ttu-id="3fbd9-214">Se viene visualizzata la finestra di dialogo **accettazione della licenza** , selezionare **Accetto** se si accettano le condizioni di licenza.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-214">If the **License Acceptance** dialog appears, select **Accept** if you agree with the license terms.</span></span>

# <a name="net-core-cli"></a>[<span data-ttu-id="3fbd9-215">Interfaccia della riga di comando di .NET Core</span><span class="sxs-lookup"><span data-stu-id="3fbd9-215">.NET Core CLI</span></span>](#tab/netcore-cli/)

<span data-ttu-id="3fbd9-216">In una shell dei comandi dalla cartella della soluzione, eseguire il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="3fbd9-216">In a command shell from the solution's folder, execute the following command:</span></span>

```dotnetcli
dotnet add Server package System.Text.Encodings.Web
```

<span data-ttu-id="3fbd9-217">Per aggiungere una versione precedente del pacchetto, specificare l' `--version {VERSION}` opzione, dove il `{VERSION}` segnaposto è la versione del pacchetto da aggiungere.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-217">To add an earlier version of the package, supply the `--version {VERSION}` option, where the `{VERSION}` placeholder is the version of the package to add.</span></span>

---

::: moniker-end

## <a name="add-a-signalr-hub"></a><span data-ttu-id="3fbd9-218">Aggiungere un SignalR Hub</span><span class="sxs-lookup"><span data-stu-id="3fbd9-218">Add a SignalR hub</span></span>

<span data-ttu-id="3fbd9-219">Nel `BlazorWebAssemblySignalRApp.Server` progetto creare una `Hubs` cartella (plurale) e aggiungere la classe seguente `ChatHub` ( `Hubs/ChatHub.cs` ):</span><span class="sxs-lookup"><span data-stu-id="3fbd9-219">In the `BlazorWebAssemblySignalRApp.Server` project, create a `Hubs` (plural) folder and add the following `ChatHub` class (`Hubs/ChatHub.cs`):</span></span>

::: moniker range=">= aspnetcore-5.0"

[!code-csharp[](signalr-blazor/samples/5.x/BlazorWebAssemblySignalRApp/Server/Hubs/ChatHub.cs)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-csharp[](signalr-blazor/samples/3.x/BlazorWebAssemblySignalRApp/Server/Hubs/ChatHub.cs)]

::: moniker-end

## <a name="add-services-and-an-endpoint-for-the-signalr-hub"></a><span data-ttu-id="3fbd9-220">Aggiungere servizi e un endpoint per l' SignalR Hub</span><span class="sxs-lookup"><span data-stu-id="3fbd9-220">Add services and an endpoint for the SignalR hub</span></span>

1. <span data-ttu-id="3fbd9-221">Nel progetto `BlazorWebAssemblySignalRApp.Server` aprire il file `Startup.cs`.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-221">In the `BlazorWebAssemblySignalRApp.Server` project, open the `Startup.cs` file.</span></span>

1. <span data-ttu-id="3fbd9-222">Aggiungere lo spazio dei nomi per la `ChatHub` classe all'inizio del file:</span><span class="sxs-lookup"><span data-stu-id="3fbd9-222">Add the namespace for the `ChatHub` class to the top of the file:</span></span>

   ```csharp
   using BlazorWebAssemblySignalRApp.Server.Hubs;
   ```

::: moniker range=">= aspnetcore-5.0"

1. <span data-ttu-id="3fbd9-223">Aggiungere SignalR e rispondere ai servizi middleware di compressione per `Startup.ConfigureServices` :</span><span class="sxs-lookup"><span data-stu-id="3fbd9-223">Add SignalR and Response Compression Middleware services to `Startup.ConfigureServices`:</span></span>

   [!code-csharp[](signalr-blazor/samples/5.x/BlazorWebAssemblySignalRApp/Server/Startup.cs?name=snippet_ConfigureServices&highlight=3,6-10)]

1. <span data-ttu-id="3fbd9-224">In `Startup.Configure`:</span><span class="sxs-lookup"><span data-stu-id="3fbd9-224">In `Startup.Configure`:</span></span>

   * <span data-ttu-id="3fbd9-225">Usare il middleware della compressione della risposta nella parte superiore della configurazione della pipeline di elaborazione.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-225">Use Response Compression Middleware at the top of the processing pipeline's configuration.</span></span>
   * <span data-ttu-id="3fbd9-226">Tra gli endpoint per i controller e il fallback sul lato client, aggiungere un endpoint per l'hub.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-226">Between the endpoints for controllers and the client-side fallback, add an endpoint for the hub.</span></span>

   [!code-csharp[](signalr-blazor/samples/5.x/BlazorWebAssemblySignalRApp/Server/Startup.cs?name=snippet_Configure&highlight=3,26)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

1. <span data-ttu-id="3fbd9-227">Aggiungere SignalR e rispondere ai servizi middleware di compressione per `Startup.ConfigureServices` :</span><span class="sxs-lookup"><span data-stu-id="3fbd9-227">Add SignalR and Response Compression Middleware services to `Startup.ConfigureServices`:</span></span>

   [!code-csharp[](signalr-blazor/samples/3.x/BlazorWebAssemblySignalRApp/Server/Startup.cs?name=snippet_ConfigureServices&highlight=3,5-9)]

1. <span data-ttu-id="3fbd9-228">In `Startup.Configure`:</span><span class="sxs-lookup"><span data-stu-id="3fbd9-228">In `Startup.Configure`:</span></span>

   * <span data-ttu-id="3fbd9-229">Usare il middleware della compressione della risposta nella parte superiore della configurazione della pipeline di elaborazione.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-229">Use Response Compression Middleware at the top of the processing pipeline's configuration.</span></span>
   * <span data-ttu-id="3fbd9-230">Tra gli endpoint per i controller e il fallback sul lato client, aggiungere un endpoint per l'hub.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-230">Between the endpoints for controllers and the client-side fallback, add an endpoint for the hub.</span></span>

   [!code-csharp[](signalr-blazor/samples/3.x/BlazorWebAssemblySignalRApp/Server/Startup.cs?name=snippet_Configure&highlight=3,25)]

::: moniker-end

## <a name="add-razor-component-code-for-chat"></a><span data-ttu-id="3fbd9-231">Aggiungi Razor codice componente per la chat</span><span class="sxs-lookup"><span data-stu-id="3fbd9-231">Add Razor component code for chat</span></span>

1. <span data-ttu-id="3fbd9-232">Nel progetto `BlazorWebAssemblySignalRApp.Client` aprire il file `Pages/Index.razor`.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-232">In the `BlazorWebAssemblySignalRApp.Client` project, open the `Pages/Index.razor` file.</span></span>

::: moniker range=">= aspnetcore-5.0"

1. <span data-ttu-id="3fbd9-233">Sostituire il markup con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="3fbd9-233">Replace the markup with the following code:</span></span>

   [!code-razor[](signalr-blazor/samples/5.x/BlazorWebAssemblySignalRApp/Client/Pages/Index.razor)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

1. <span data-ttu-id="3fbd9-234">Sostituire il markup con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="3fbd9-234">Replace the markup with the following code:</span></span>

   [!code-razor[](signalr-blazor/samples/3.x/BlazorWebAssemblySignalRApp/Client/Pages/Index.razor)]

::: moniker-end

## <a name="run-the-app"></a><span data-ttu-id="3fbd9-235">Eseguire l'app</span><span class="sxs-lookup"><span data-stu-id="3fbd9-235">Run the app</span></span>

<span data-ttu-id="3fbd9-236">Seguire le istruzioni per gli strumenti:</span><span class="sxs-lookup"><span data-stu-id="3fbd9-236">Follow the guidance for your tooling:</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="3fbd9-237">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="3fbd9-237">Visual Studio</span></span>](#tab/visual-studio)

1. <span data-ttu-id="3fbd9-238">In **Esplora soluzioni** selezionare il `BlazorWebAssemblySignalRApp.Server` progetto.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-238">In **Solution Explorer**, select the `BlazorWebAssemblySignalRApp.Server` project.</span></span> <span data-ttu-id="3fbd9-239">Premere <kbd>F5</kbd> per eseguire l'app con il debug o <kbd>CTRL</kbd> + <kbd>F5</kbd> per eseguire l'app senza debug.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-239">Press <kbd>F5</kbd> to run the app with debugging or <kbd>Ctrl</kbd>+<kbd>F5</kbd> to run the app without debugging.</span></span>

1. <span data-ttu-id="3fbd9-240">Copiare l'URL dalla barra degli indirizzi, aprire un'altra istanza o scheda del browser e incollare l'URL nella barra degli indirizzi.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-240">Copy the URL from the address bar, open another browser instance or tab, and paste the URL in the address bar.</span></span>

1. <span data-ttu-id="3fbd9-241">Scegliere browser, immettere un nome e un messaggio e selezionare il pulsante per l'invio del messaggio.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-241">Choose either browser, enter a name and message, and select the button to send the message.</span></span> <span data-ttu-id="3fbd9-242">Il nome e il messaggio vengono visualizzati immediatamente in entrambe le pagine:</span><span class="sxs-lookup"><span data-stu-id="3fbd9-242">The name and message are displayed on both pages instantly:</span></span>

   ![::: NO-LOC (SignalR):::::: NO-LOC (Blazer)::: app di esempio aperta in due finestre del browser che mostrano i messaggi scambiati.](signalr-blazor/_static/signalr-blazor-finished.png)

   <span data-ttu-id="3fbd9-244">Virgolette: *Star Trek VI: il paese non individuato* &copy; 1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)</span><span class="sxs-lookup"><span data-stu-id="3fbd9-244">Quotes: *Star Trek VI: The Undiscovered Country* &copy;1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="3fbd9-245">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="3fbd9-245">Visual Studio Code</span></span>](#tab/visual-studio-code)

1. <span data-ttu-id="3fbd9-246">Premere <kbd>F5</kbd> per eseguire l'app con il debug o <kbd>CTRL</kbd> + <kbd>F5</kbd> per eseguire l'app senza debug.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-246">Press <kbd>F5</kbd> to run the app with debugging or <kbd>Ctrl</kbd>+<kbd>F5</kbd> to run the app without debugging.</span></span>

1. <span data-ttu-id="3fbd9-247">Copiare l'URL dalla barra degli indirizzi, aprire un'altra istanza o scheda del browser e incollare l'URL nella barra degli indirizzi.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-247">Copy the URL from the address bar, open another browser instance or tab, and paste the URL in the address bar.</span></span>

1. <span data-ttu-id="3fbd9-248">Scegliere browser, immettere un nome e un messaggio e selezionare il pulsante per l'invio del messaggio.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-248">Choose either browser, enter a name and message, and select the button to send the message.</span></span> <span data-ttu-id="3fbd9-249">Il nome e il messaggio vengono visualizzati immediatamente in entrambe le pagine:</span><span class="sxs-lookup"><span data-stu-id="3fbd9-249">The name and message are displayed on both pages instantly:</span></span>

   ![::: NO-LOC (SignalR):::::: NO-LOC (Blazer)::: app di esempio aperta in due finestre del browser che mostrano i messaggi scambiati.](signalr-blazor/_static/signalr-blazor-finished.png)

   <span data-ttu-id="3fbd9-251">Virgolette: *Star Trek VI: il paese non individuato* &copy; 1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)</span><span class="sxs-lookup"><span data-stu-id="3fbd9-251">Quotes: *Star Trek VI: The Undiscovered Country* &copy;1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)</span></span>

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="3fbd9-252">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="3fbd9-252">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

1. <span data-ttu-id="3fbd9-253">Nella barra laterale della **soluzione** selezionare il `BlazorWebAssemblySignalRApp.Server` progetto.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-253">In the **Solution** sidebar, select the `BlazorWebAssemblySignalRApp.Server` project.</span></span> <span data-ttu-id="3fbd9-254">Premere <kbd>⌘</kbd> + <kbd>↩</kbd> per eseguire l'app con debug o <kbd>⌥</kbd> + <kbd>⌘</kbd> + <kbd>↩</kbd> per eseguire l'app senza debug.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-254">Press <kbd>⌘</kbd>+<kbd>↩</kbd> to run the app with debugging or <kbd>⌥</kbd>+<kbd>⌘</kbd>+<kbd>↩</kbd> to run the app without debugging.</span></span>

1. <span data-ttu-id="3fbd9-255">Copiare l'URL dalla barra degli indirizzi, aprire un'altra istanza o scheda del browser e incollare l'URL nella barra degli indirizzi.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-255">Copy the URL from the address bar, open another browser instance or tab, and paste the URL in the address bar.</span></span>

1. <span data-ttu-id="3fbd9-256">Scegliere browser, immettere un nome e un messaggio e selezionare il pulsante per l'invio del messaggio.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-256">Choose either browser, enter a name and message, and select the button to send the message.</span></span> <span data-ttu-id="3fbd9-257">Il nome e il messaggio vengono visualizzati immediatamente in entrambe le pagine:</span><span class="sxs-lookup"><span data-stu-id="3fbd9-257">The name and message are displayed on both pages instantly:</span></span>

   ![::: NO-LOC (SignalR):::::: NO-LOC (Blazer)::: app di esempio aperta in due finestre del browser che mostrano i messaggi scambiati.](signalr-blazor/_static/signalr-blazor-finished.png)

   <span data-ttu-id="3fbd9-259">Virgolette: *Star Trek VI: il paese non individuato* &copy; 1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)</span><span class="sxs-lookup"><span data-stu-id="3fbd9-259">Quotes: *Star Trek VI: The Undiscovered Country* &copy;1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)</span></span>

# <a name="net-core-cli"></a>[<span data-ttu-id="3fbd9-260">Interfaccia della riga di comando di .NET Core</span><span class="sxs-lookup"><span data-stu-id="3fbd9-260">.NET Core CLI</span></span>](#tab/netcore-cli/)

1. <span data-ttu-id="3fbd9-261">In una shell dei comandi dalla cartella della soluzione eseguire i comandi seguenti:</span><span class="sxs-lookup"><span data-stu-id="3fbd9-261">In a command shell from the solution's folder, execute the following commands:</span></span>

   ```dotnetcli
   cd Server
   dotnet run
   ```

1. <span data-ttu-id="3fbd9-262">Copiare l'URL dalla barra degli indirizzi, aprire un'altra istanza o scheda del browser e incollare l'URL nella barra degli indirizzi.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-262">Copy the URL from the address bar, open another browser instance or tab, and paste the URL in the address bar.</span></span>

1. <span data-ttu-id="3fbd9-263">Scegliere browser, immettere un nome e un messaggio e selezionare il pulsante per l'invio del messaggio.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-263">Choose either browser, enter a name and message, and select the button to send the message.</span></span> <span data-ttu-id="3fbd9-264">Il nome e il messaggio vengono visualizzati immediatamente in entrambe le pagine:</span><span class="sxs-lookup"><span data-stu-id="3fbd9-264">The name and message are displayed on both pages instantly:</span></span>

   ![::: NO-LOC (SignalR):::::: NO-LOC (Blazer)::: app di esempio aperta in due finestre del browser che mostrano i messaggi scambiati.](signalr-blazor/_static/signalr-blazor-finished.png)

   <span data-ttu-id="3fbd9-266">Virgolette: *Star Trek VI: il paese non individuato* &copy; 1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)</span><span class="sxs-lookup"><span data-stu-id="3fbd9-266">Quotes: *Star Trek VI: The Undiscovered Country* &copy;1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)</span></span>

---

::: zone-end

::: zone pivot="server"

## <a name="create-a-blazor-server-app"></a><span data-ttu-id="3fbd9-267">Creare un' Blazor Server app</span><span class="sxs-lookup"><span data-stu-id="3fbd9-267">Create a Blazor Server app</span></span>

<span data-ttu-id="3fbd9-268">Seguire le istruzioni per la scelta degli strumenti:</span><span class="sxs-lookup"><span data-stu-id="3fbd9-268">Follow the guidance for your choice of tooling:</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="3fbd9-269">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="3fbd9-269">Visual Studio</span></span>](#tab/visual-studio)

::: moniker range=">= aspnetcore-5.0"

> [!NOTE]
> <span data-ttu-id="3fbd9-270">Sono necessari Visual Studio 16,8 o versioni successive e .NET Core SDK 5.0.0 o versioni successive.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-270">Visual Studio 16.8 or later and .NET Core SDK 5.0.0 or later are required.</span></span>

::: moniker-end

::: moniker range="< aspnetcore-5.0"

> [!NOTE]
> <span data-ttu-id="3fbd9-271">Sono necessari Visual Studio 16,6 o versioni successive e .NET Core SDK 3.1.300 o versioni successive.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-271">Visual Studio 16.6 or later and .NET Core SDK 3.1.300 or later are required.</span></span>

::: moniker-end

1. <span data-ttu-id="3fbd9-272">Creare un nuovo progetto.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-272">Create a new project.</span></span>

1. <span data-ttu-id="3fbd9-273">Selezionare **Blazor app** e fare clic su **Next (avanti**).</span><span class="sxs-lookup"><span data-stu-id="3fbd9-273">Select **Blazor App** and select **Next**.</span></span>

1. <span data-ttu-id="3fbd9-274">Digitare `BlazorServerSignalRApp` nel campo **nome progetto** .</span><span class="sxs-lookup"><span data-stu-id="3fbd9-274">Type `BlazorServerSignalRApp` in the **Project name** field.</span></span> <span data-ttu-id="3fbd9-275">Confermare che la voce relativa al **percorso** sia corretta o specificare un percorso per il progetto.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-275">Confirm the **Location** entry is correct or provide a location for the project.</span></span> <span data-ttu-id="3fbd9-276">Selezionare **Crea**.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-276">Select **Create**.</span></span>

1. <span data-ttu-id="3fbd9-277">Scegliere il modello di **Blazor Server app** .</span><span class="sxs-lookup"><span data-stu-id="3fbd9-277">Choose the **Blazor Server App** template.</span></span>

1. <span data-ttu-id="3fbd9-278">Selezionare **Crea**.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-278">Select **Create**.</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="3fbd9-279">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="3fbd9-279">Visual Studio Code</span></span>](#tab/visual-studio-code)

1. <span data-ttu-id="3fbd9-280">In una shell dei comandi eseguire il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="3fbd9-280">In a command shell, execute the following command:</span></span>

   ```dotnetcli
   dotnet new blazorserver -o BlazorServerSignalRApp
   ```

   <span data-ttu-id="3fbd9-281">L' `-o|--output` opzione Crea una cartella per il progetto.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-281">The `-o|--output` option creates a folder for the project.</span></span> <span data-ttu-id="3fbd9-282">Se è stata creata una cartella per il progetto e la shell dei comandi è aperta in tale cartella, omettere l' `-o|--output` opzione e il valore per creare il progetto.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-282">If you've created a folder for the project and the command shell is open in that folder, omit the `-o|--output` option and value to create the project.</span></span>

1. <span data-ttu-id="3fbd9-283">In Visual Studio Code aprire la cartella del progetto dell'app.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-283">In Visual Studio Code, open the app's project folder.</span></span>

1. <span data-ttu-id="3fbd9-284">Quando viene visualizzata la finestra di dialogo per aggiungere asset per compilare ed eseguire il debug dell'app, selezionare **Sì**.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-284">When the dialog appears to add assets to build and debug the app, select **Yes**.</span></span> <span data-ttu-id="3fbd9-285">Visual Studio Code aggiunge automaticamente la `.vscode` cartella con `launch.json` `tasks.json` i file e generati.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-285">Visual Studio Code automatically adds the `.vscode` folder with generated `launch.json` and `tasks.json` files.</span></span>

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="3fbd9-286">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="3fbd9-286">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

1. <span data-ttu-id="3fbd9-287">Installare la versione più recente di [Visual Studio per Mac](https://visualstudio.microsoft.com/vs/mac/) e seguire questa procedura:</span><span class="sxs-lookup"><span data-stu-id="3fbd9-287">Install the latest version of [Visual Studio for Mac](https://visualstudio.microsoft.com/vs/mac/) and perform the following steps:</span></span>

1. <span data-ttu-id="3fbd9-288">Selezionare **file**  >  **nuova soluzione** o creare un **nuovo** progetto dalla **finestra Start**.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-288">Select **File** > **New Solution** or create a **New** project from the **Start Window**.</span></span>

1. <span data-ttu-id="3fbd9-289">Nella barra laterale selezionare app **Web e console**  >  .</span><span class="sxs-lookup"><span data-stu-id="3fbd9-289">In the sidebar, select **Web and Console** > **App**.</span></span>

1. <span data-ttu-id="3fbd9-290">Scegliere il modello di **Blazor Server app** .</span><span class="sxs-lookup"><span data-stu-id="3fbd9-290">Choose the **Blazor Server App** template.</span></span> <span data-ttu-id="3fbd9-291">Selezionare **Avanti**.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-291">Select **Next**.</span></span>

1. <span data-ttu-id="3fbd9-292">Verificare che **l'autenticazione** sia impostata su **Nessuna autenticazione**.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-292">Confirm that **Authentication** is set to **No Authentication**.</span></span> <span data-ttu-id="3fbd9-293">Selezionare **Avanti**.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-293">Select **Next**.</span></span>

1. <span data-ttu-id="3fbd9-294">Nel campo **nome progetto** assegnare un nome all'app `BlazorServerSignalRApp` .</span><span class="sxs-lookup"><span data-stu-id="3fbd9-294">In the **Project Name** field, name the app `BlazorServerSignalRApp`.</span></span> <span data-ttu-id="3fbd9-295">Selezionare **Crea**.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-295">Select **Create**.</span></span>

   <span data-ttu-id="3fbd9-296">Se viene visualizzato un messaggio per considerare attendibile il certificato di sviluppo, considerare attendibile il certificato e continuare.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-296">If a prompt appears to trust the development certificate, trust the certificate and continue.</span></span> <span data-ttu-id="3fbd9-297">Per considerare attendibile il certificato, è necessario specificare le password dell'utente e del keychain.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-297">The user and keychain passwords are required to trust the certificate.</span></span>

1. <span data-ttu-id="3fbd9-298">Aprire il progetto passando alla cartella del progetto e aprendo il file di soluzione del progetto ( `.sln` ).</span><span class="sxs-lookup"><span data-stu-id="3fbd9-298">Open the project by navigating to the project folder and opening the project's solution file (`.sln`).</span></span>

# <a name="net-core-cli"></a>[<span data-ttu-id="3fbd9-299">Interfaccia della riga di comando di .NET Core</span><span class="sxs-lookup"><span data-stu-id="3fbd9-299">.NET Core CLI</span></span>](#tab/netcore-cli/)

<span data-ttu-id="3fbd9-300">In una shell dei comandi eseguire il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="3fbd9-300">In a command shell, execute the following command:</span></span>

```dotnetcli
dotnet new blazorserver -o BlazorServerSignalRApp
```

<span data-ttu-id="3fbd9-301">L' `-o|--output` opzione Crea una cartella per il progetto.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-301">The `-o|--output` option creates a folder for the project.</span></span> <span data-ttu-id="3fbd9-302">Se è stata creata una cartella per il progetto e la shell dei comandi è aperta in tale cartella, omettere l' `-o|--output` opzione e il valore per creare il progetto.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-302">If you've created a folder for the project and the command shell is open in that folder, omit the `-o|--output` option and value to create the project.</span></span>

---

## <a name="add-the-signalr-client-library"></a><span data-ttu-id="3fbd9-303">Aggiungere la SignalR libreria client</span><span class="sxs-lookup"><span data-stu-id="3fbd9-303">Add the SignalR client library</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="3fbd9-304">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="3fbd9-304">Visual Studio</span></span>](#tab/visual-studio/)

1. <span data-ttu-id="3fbd9-305">In **Esplora soluzioni** fare clic con il pulsante destro del mouse sul `BlazorServerSignalRApp` progetto e scegliere **Gestisci pacchetti NuGet**.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-305">In **Solution Explorer**, right-click the `BlazorServerSignalRApp` project and select **Manage NuGet Packages**.</span></span>

1. <span data-ttu-id="3fbd9-306">Nella finestra di dialogo **Gestisci pacchetti NuGet** verificare che l' **origine del pacchetto** sia impostata su `nuget.org` .</span><span class="sxs-lookup"><span data-stu-id="3fbd9-306">In the **Manage NuGet Packages** dialog, confirm that the **Package source** is set to `nuget.org`.</span></span>

1. <span data-ttu-id="3fbd9-307">Con **Sfoglia** selezionato, digitare `Microsoft.AspNetCore.SignalR.Client` nella casella di ricerca.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-307">With **Browse** selected, type `Microsoft.AspNetCore.SignalR.Client` in the search box.</span></span>

1. <span data-ttu-id="3fbd9-308">Nei risultati della ricerca selezionare il [`Microsoft.AspNetCore.SignalR.Client`](https://www.nuget.org/packages/Microsoft.AspNetCore.SignalR.Client) pacchetto.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-308">In the search results, select the [`Microsoft.AspNetCore.SignalR.Client`](https://www.nuget.org/packages/Microsoft.AspNetCore.SignalR.Client) package.</span></span> <span data-ttu-id="3fbd9-309">Impostare la versione in modo che corrisponda al Framework condiviso dell'app.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-309">Set the version to match the shared framework of the app.</span></span> <span data-ttu-id="3fbd9-310">Selezionare **Installa**.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-310">Select **Install**.</span></span>

1. <span data-ttu-id="3fbd9-311">Se viene visualizzata la finestra di dialogo **Anteprima modifiche** , selezionare **OK**.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-311">If the **Preview Changes** dialog appears, select **OK**.</span></span>

1. <span data-ttu-id="3fbd9-312">Se viene visualizzata la finestra di dialogo **accettazione della licenza** , selezionare Accetto se **si accettano le** condizioni di licenza.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-312">If the **License Acceptance** dialog appears, select **I Accept** if you agree with the license terms.</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="3fbd9-313">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="3fbd9-313">Visual Studio Code</span></span>](#tab/visual-studio-code/)

<span data-ttu-id="3fbd9-314">Nel **terminale integrato** (**visualizzare** il  >  **terminale** dalla barra degli strumenti) eseguire il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="3fbd9-314">In the **Integrated Terminal** (**View** > **Terminal** from the toolbar), execute the following command:</span></span>

```dotnetcli
dotnet add package Microsoft.AspNetCore.SignalR.Client
```

<span data-ttu-id="3fbd9-315">Per aggiungere una versione precedente del pacchetto, specificare l' `--version {VERSION}` opzione, dove il `{VERSION}` segnaposto è la versione del pacchetto da aggiungere.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-315">To add an earlier version of the package, supply the `--version {VERSION}` option, where the `{VERSION}` placeholder is the version of the package to add.</span></span>

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="3fbd9-316">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="3fbd9-316">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

1. <span data-ttu-id="3fbd9-317">Nella barra laterale **soluzione** fare clic con il pulsante destro del mouse sul `BlazorServerSignalRApp` progetto e scegliere **Gestisci pacchetti NuGet**.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-317">In the **Solution** sidebar, right-click the `BlazorServerSignalRApp` project and select **Manage NuGet Packages**.</span></span>

1. <span data-ttu-id="3fbd9-318">Nella finestra di dialogo **Gestisci pacchetti NuGet** verificare che l'elenco a discesa origine sia impostato su `nuget.org` .</span><span class="sxs-lookup"><span data-stu-id="3fbd9-318">In the **Manage NuGet Packages** dialog, confirm that the source drop-down is set to `nuget.org`.</span></span>

1. <span data-ttu-id="3fbd9-319">Con **Sfoglia** selezionato, digitare `Microsoft.AspNetCore.SignalR.Client` nella casella di ricerca.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-319">With **Browse** selected, type `Microsoft.AspNetCore.SignalR.Client` in the search box.</span></span>

1. <span data-ttu-id="3fbd9-320">Nei risultati della ricerca selezionare la casella di controllo accanto al [`Microsoft.AspNetCore.SignalR.Client`](https://www.nuget.org/packages/Microsoft.AspNetCore.SignalR.Client) pacchetto.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-320">In the search results, select the check box next to the [`Microsoft.AspNetCore.SignalR.Client`](https://www.nuget.org/packages/Microsoft.AspNetCore.SignalR.Client) package.</span></span> <span data-ttu-id="3fbd9-321">Impostare la versione in modo che corrisponda al Framework condiviso dell'app.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-321">Set the version to match the shared framework of the app.</span></span> <span data-ttu-id="3fbd9-322">Selezionare **Aggiungi pacchetto**.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-322">Select **Add Package**.</span></span>

1. <span data-ttu-id="3fbd9-323">Se viene visualizzata la finestra di dialogo **accettazione della licenza** , selezionare **Accetto** se si accettano le condizioni di licenza.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-323">If the **License Acceptance** dialog appears, select **Accept** if you agree with the license terms.</span></span>

# <a name="net-core-cli"></a>[<span data-ttu-id="3fbd9-324">Interfaccia della riga di comando di .NET Core</span><span class="sxs-lookup"><span data-stu-id="3fbd9-324">.NET Core CLI</span></span>](#tab/netcore-cli/)

<span data-ttu-id="3fbd9-325">In una shell dei comandi dalla cartella del progetto, eseguire il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="3fbd9-325">In a command shell from the project's folder, execute the following command:</span></span>

```dotnetcli
dotnet add package Microsoft.AspNetCore.SignalR.Client
```

<span data-ttu-id="3fbd9-326">Per aggiungere una versione precedente del pacchetto, specificare l' `--version {VERSION}` opzione, dove il `{VERSION}` segnaposto è la versione del pacchetto da aggiungere.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-326">To add an earlier version of the package, supply the `--version {VERSION}` option, where the `{VERSION}` placeholder is the version of the package to add.</span></span>

---

::: moniker range="< aspnetcore-5.0"

## <a name="add-the-systemtextencodingsweb-package"></a><span data-ttu-id="3fbd9-327">Aggiungere il pacchetto System. Text. Encodings. Web</span><span class="sxs-lookup"><span data-stu-id="3fbd9-327">Add the System.Text.Encodings.Web package</span></span>

<span data-ttu-id="3fbd9-328">*Questa sezione si applica solo alle app per ASP.NET Core versione 3. x.*</span><span class="sxs-lookup"><span data-stu-id="3fbd9-328">*This section only applies to apps for ASP.NET Core version 3.x.*</span></span>

<span data-ttu-id="3fbd9-329">A causa di un problema di risoluzione del pacchetto quando si usa [`System.Text.Json`](https://www.nuget.org/packages/System.Text.Json) 5. x in un'app ASP.NET Core 3. x, il progetto richiede un riferimento al pacchetto per [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web) .</span><span class="sxs-lookup"><span data-stu-id="3fbd9-329">Due to a package resolution issue when using [`System.Text.Json`](https://www.nuget.org/packages/System.Text.Json) 5.x in an ASP.NET Core 3.x app, the project requires a package reference for [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web).</span></span> <span data-ttu-id="3fbd9-330">Il problema sottostante verrà risolto in una versione futura della patch di .NET 5.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-330">The underlying issue will be resolved in a future patch release of .NET 5.</span></span> <span data-ttu-id="3fbd9-331">Per altre informazioni, vedere [System.Text.Json definisce netcoreapp 3.0 senza dipendenze (DotNet/runtime #45560)](https://github.com/dotnet/runtime/issues/45560).</span><span class="sxs-lookup"><span data-stu-id="3fbd9-331">For more information, see [System.Text.Json defines netcoreapp3.0 with no dependencies (dotnet/runtime #45560)](https://github.com/dotnet/runtime/issues/45560).</span></span>

<span data-ttu-id="3fbd9-332">Per aggiungere [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web) al progetto, seguire le istruzioni per la scelta degli strumenti:</span><span class="sxs-lookup"><span data-stu-id="3fbd9-332">To add [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web) to the project, follow the guidance for your choice of tooling:</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="3fbd9-333">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="3fbd9-333">Visual Studio</span></span>](#tab/visual-studio/)

1. <span data-ttu-id="3fbd9-334">In **Esplora soluzioni** fare clic con il pulsante destro del mouse sul `BlazorServerSignalRApp` progetto e scegliere **Gestisci pacchetti NuGet**.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-334">In **Solution Explorer**, right-click the `BlazorServerSignalRApp` project and select **Manage NuGet Packages**.</span></span>

1. <span data-ttu-id="3fbd9-335">Nella finestra di dialogo **Gestisci pacchetti NuGet** verificare che l' **origine del pacchetto** sia impostata su `nuget.org` .</span><span class="sxs-lookup"><span data-stu-id="3fbd9-335">In the **Manage NuGet Packages** dialog, confirm that the **Package source** is set to `nuget.org`.</span></span>

1. <span data-ttu-id="3fbd9-336">Con **Sfoglia** selezionato, digitare `System.Text.Encodings.Web` nella casella di ricerca.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-336">With **Browse** selected, type `System.Text.Encodings.Web` in the search box.</span></span>

1. <span data-ttu-id="3fbd9-337">Nei risultati della ricerca selezionare il [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web) pacchetto.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-337">In the search results, select the [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web) package.</span></span> <span data-ttu-id="3fbd9-338">Consente di selezionare la versione del pacchetto corrispondente al Framework condiviso in uso.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-338">Select the version of the package that matches the shared framework in use.</span></span> <span data-ttu-id="3fbd9-339">Selezionare **Installa**.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-339">Select **Install**.</span></span>

1. <span data-ttu-id="3fbd9-340">Se viene visualizzata la finestra di dialogo **Anteprima modifiche** , selezionare **OK**.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-340">If the **Preview Changes** dialog appears, select **OK**.</span></span>

1. <span data-ttu-id="3fbd9-341">Se viene visualizzata la finestra di dialogo **accettazione della licenza** , selezionare Accetto se **si accettano le** condizioni di licenza.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-341">If the **License Acceptance** dialog appears, select **I Accept** if you agree with the license terms.</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="3fbd9-342">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="3fbd9-342">Visual Studio Code</span></span>](#tab/visual-studio-code/)

<span data-ttu-id="3fbd9-343">Nel **terminale integrato** (**visualizzare** il > **terminale** dalla barra degli strumenti) eseguire i comandi seguenti:</span><span class="sxs-lookup"><span data-stu-id="3fbd9-343">In the **Integrated Terminal** (**View** > **Terminal** from the toolbar), execute the following commands:</span></span>

```dotnetcli
dotnet add package System.Text.Encodings.Web
```

<span data-ttu-id="3fbd9-344">Per aggiungere una versione precedente del pacchetto, specificare l' `--version {VERSION}` opzione, dove il `{VERSION}` segnaposto è la versione del pacchetto da aggiungere.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-344">To add an earlier version of the package, supply the `--version {VERSION}` option, where the `{VERSION}` placeholder is the version of the package to add.</span></span>

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="3fbd9-345">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="3fbd9-345">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

1. <span data-ttu-id="3fbd9-346">Nella barra laterale **soluzione** fare clic con il pulsante destro del mouse sul `BlazorServerSignalRApp` progetto e scegliere **Gestisci pacchetti NuGet**.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-346">In the **Solution** sidebar, right-click the `BlazorServerSignalRApp` project and select **Manage NuGet Packages**.</span></span>

1. <span data-ttu-id="3fbd9-347">Nella finestra di dialogo **Gestisci pacchetti NuGet** verificare che l'elenco a discesa origine sia impostato su `nuget.org` .</span><span class="sxs-lookup"><span data-stu-id="3fbd9-347">In the **Manage NuGet Packages** dialog, confirm that the source drop-down is set to `nuget.org`.</span></span>

1. <span data-ttu-id="3fbd9-348">Con **Sfoglia** selezionato, digitare `System.Text.Encodings.Web` nella casella di ricerca.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-348">With **Browse** selected, type `System.Text.Encodings.Web` in the search box.</span></span>

1. <span data-ttu-id="3fbd9-349">Nei risultati della ricerca selezionare la casella di controllo accanto al [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web) pacchetto, selezionare la versione corretta del pacchetto corrispondente al Framework condiviso in uso e selezionare **Aggiungi pacchetto**.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-349">In the search results, select the check box next to the [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web) package, select the correct version of the package that matches the shared framework in use, and select **Add Package**.</span></span>

1. <span data-ttu-id="3fbd9-350">Se viene visualizzata la finestra di dialogo **accettazione della licenza** , selezionare **Accetto** se si accettano le condizioni di licenza.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-350">If the **License Acceptance** dialog appears, select **Accept** if you agree with the license terms.</span></span>

# <a name="net-core-cli"></a>[<span data-ttu-id="3fbd9-351">Interfaccia della riga di comando di .NET Core</span><span class="sxs-lookup"><span data-stu-id="3fbd9-351">.NET Core CLI</span></span>](#tab/netcore-cli/)

<span data-ttu-id="3fbd9-352">In una shell dei comandi dalla cartella del progetto, eseguire il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="3fbd9-352">In a command shell from the project's folder, execute the following command:</span></span>

```dotnetcli
dotnet add package System.Text.Encodings.Web
```

<span data-ttu-id="3fbd9-353">Per aggiungere una versione precedente del pacchetto, specificare l' `--version {VERSION}` opzione, dove il `{VERSION}` segnaposto è la versione del pacchetto da aggiungere.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-353">To add an earlier version of the package, supply the `--version {VERSION}` option, where the `{VERSION}` placeholder is the version of the package to add.</span></span>

---

::: moniker-end

## <a name="add-a-signalr-hub"></a><span data-ttu-id="3fbd9-354">Aggiungere un SignalR Hub</span><span class="sxs-lookup"><span data-stu-id="3fbd9-354">Add a SignalR hub</span></span>

<span data-ttu-id="3fbd9-355">Creare una `Hubs` cartella (plurale) e aggiungere la `ChatHub` classe seguente ( `Hubs/ChatHub.cs` ):</span><span class="sxs-lookup"><span data-stu-id="3fbd9-355">Create a `Hubs` (plural) folder and add the following `ChatHub` class (`Hubs/ChatHub.cs`):</span></span>

::: moniker range=">= aspnetcore-5.0"

[!code-csharp[](signalr-blazor/samples/5.x/BlazorServerSignalRApp/Hubs/ChatHub.cs)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-csharp[](signalr-blazor/samples/3.x/BlazorServerSignalRApp/Hubs/ChatHub.cs)]

::: moniker-end

## <a name="add-services-and-an-endpoint-for-the-signalr-hub"></a><span data-ttu-id="3fbd9-356">Aggiungere servizi e un endpoint per l' SignalR Hub</span><span class="sxs-lookup"><span data-stu-id="3fbd9-356">Add services and an endpoint for the SignalR hub</span></span>

1. <span data-ttu-id="3fbd9-357">Aprire il file `Startup.cs`.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-357">Open the `Startup.cs` file.</span></span>

1. <span data-ttu-id="3fbd9-358">Aggiungere gli spazi dei nomi per <xref:Microsoft.AspNetCore.ResponseCompression?displayProperty=fullName> e la `ChatHub` classe all'inizio del file:</span><span class="sxs-lookup"><span data-stu-id="3fbd9-358">Add the namespaces for <xref:Microsoft.AspNetCore.ResponseCompression?displayProperty=fullName> and the `ChatHub` class to the top of the file:</span></span>

   ```csharp
   using Microsoft.AspNetCore.ResponseCompression;
   using BlazorServerSignalRApp.Server.Hubs;
   ```

::: moniker range=">= aspnetcore-5.0"

1. <span data-ttu-id="3fbd9-359">Aggiungere i servizi middleware della compressione della risposta a `Startup.ConfigureServices` :</span><span class="sxs-lookup"><span data-stu-id="3fbd9-359">Add Response Compression Middleware services to `Startup.ConfigureServices`:</span></span>

   [!code-csharp[](signalr-blazor/samples/5.x/BlazorServerSignalRApp/Startup.cs?name=snippet_ConfigureServices&highlight=6-10)]

1. <span data-ttu-id="3fbd9-360">In `Startup.Configure`:</span><span class="sxs-lookup"><span data-stu-id="3fbd9-360">In `Startup.Configure`:</span></span>

   * <span data-ttu-id="3fbd9-361">Usare il middleware della compressione della risposta nella parte superiore della configurazione della pipeline di elaborazione.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-361">Use Response Compression Middleware at the top of the processing pipeline's configuration.</span></span>
   * <span data-ttu-id="3fbd9-362">Tra gli endpoint per il mapping dell' Blazor Hub e del fallback lato client, aggiungere un endpoint per l'hub.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-362">Between the endpoints for mapping the Blazor hub and the client-side fallback, add an endpoint for the hub.</span></span>

   [!code-csharp[](signalr-blazor/samples/5.x/BlazorServerSignalRApp/Startup.cs?name=snippet_Configure&highlight=3,23)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

1. <span data-ttu-id="3fbd9-363">Aggiungere i servizi middleware della compressione della risposta a `Startup.ConfigureServices` :</span><span class="sxs-lookup"><span data-stu-id="3fbd9-363">Add Response Compression Middleware services to `Startup.ConfigureServices`:</span></span>

   [!code-csharp[](signalr-blazor/samples/3.x/BlazorServerSignalRApp/Startup.cs?name=snippet_ConfigureServices&highlight=6-10)]

1. <span data-ttu-id="3fbd9-364">In `Startup.Configure`:</span><span class="sxs-lookup"><span data-stu-id="3fbd9-364">In `Startup.Configure`:</span></span>

   * <span data-ttu-id="3fbd9-365">Usare il middleware della compressione della risposta nella parte superiore della configurazione della pipeline di elaborazione.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-365">Use Response Compression Middleware at the top of the processing pipeline's configuration.</span></span>
   * <span data-ttu-id="3fbd9-366">Tra gli endpoint per il mapping dell' Blazor Hub e del fallback lato client, aggiungere un endpoint per l'hub.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-366">Between the endpoints for mapping the Blazor hub and the client-side fallback, add an endpoint for the hub.</span></span>

   [!code-csharp[](signalr-blazor/samples/3.x/BlazorServerSignalRApp/Startup.cs?name=snippet_Configure&highlight=3,23)]

::: moniker-end

## <a name="add-razor-component-code-for-chat"></a><span data-ttu-id="3fbd9-367">Aggiungi Razor codice componente per la chat</span><span class="sxs-lookup"><span data-stu-id="3fbd9-367">Add Razor component code for chat</span></span>

1. <span data-ttu-id="3fbd9-368">Aprire il file `Pages/Index.razor`.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-368">Open the `Pages/Index.razor` file.</span></span>

::: moniker range=">= aspnetcore-5.0"

1. <span data-ttu-id="3fbd9-369">Sostituire il markup con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="3fbd9-369">Replace the markup with the following code:</span></span>

   [!code-razor[](signalr-blazor/samples/5.x/BlazorServerSignalRApp/Pages/Index.razor)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

1. <span data-ttu-id="3fbd9-370">Sostituire il markup con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="3fbd9-370">Replace the markup with the following code:</span></span>

   [!code-razor[](signalr-blazor/samples/3.x/BlazorServerSignalRApp/Pages/Index.razor)]

::: moniker-end

## <a name="run-the-app"></a><span data-ttu-id="3fbd9-371">Eseguire l'app</span><span class="sxs-lookup"><span data-stu-id="3fbd9-371">Run the app</span></span>

<span data-ttu-id="3fbd9-372">Seguire le istruzioni per gli strumenti:</span><span class="sxs-lookup"><span data-stu-id="3fbd9-372">Follow the guidance for your tooling:</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="3fbd9-373">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="3fbd9-373">Visual Studio</span></span>](#tab/visual-studio)

1. <span data-ttu-id="3fbd9-374">Premere <kbd>F5</kbd> per eseguire l'app con il debug o <kbd>CTRL</kbd> + <kbd>F5</kbd> per eseguire l'app senza debug.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-374">Press <kbd>F5</kbd> to run the app with debugging or <kbd>Ctrl</kbd>+<kbd>F5</kbd> to run the app without debugging.</span></span>

1. <span data-ttu-id="3fbd9-375">Copiare l'URL dalla barra degli indirizzi, aprire un'altra istanza o scheda del browser e incollare l'URL nella barra degli indirizzi.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-375">Copy the URL from the address bar, open another browser instance or tab, and paste the URL in the address bar.</span></span>

1. <span data-ttu-id="3fbd9-376">Scegliere browser, immettere un nome e un messaggio e selezionare il pulsante per l'invio del messaggio.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-376">Choose either browser, enter a name and message, and select the button to send the message.</span></span> <span data-ttu-id="3fbd9-377">Il nome e il messaggio vengono visualizzati immediatamente in entrambe le pagine:</span><span class="sxs-lookup"><span data-stu-id="3fbd9-377">The name and message are displayed on both pages instantly:</span></span>

   ![::: NO-LOC (SignalR):::::: NO-LOC (Blazer)::: app di esempio aperta in due finestre del browser che mostrano i messaggi scambiati.](signalr-blazor/_static/signalr-blazor-finished.png)

   <span data-ttu-id="3fbd9-379">Virgolette: *Star Trek VI: il paese non individuato* &copy; 1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)</span><span class="sxs-lookup"><span data-stu-id="3fbd9-379">Quotes: *Star Trek VI: The Undiscovered Country* &copy;1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="3fbd9-380">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="3fbd9-380">Visual Studio Code</span></span>](#tab/visual-studio-code)

1. <span data-ttu-id="3fbd9-381">Premere <kbd>F5</kbd> per eseguire l'app con il debug o <kbd>CTRL</kbd> + <kbd>F5</kbd> per eseguire l'app senza debug.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-381">Press <kbd>F5</kbd> to run the app with debugging or <kbd>Ctrl</kbd>+<kbd>F5</kbd> to run the app without debugging.</span></span>

1. <span data-ttu-id="3fbd9-382">Copiare l'URL dalla barra degli indirizzi, aprire un'altra istanza o scheda del browser e incollare l'URL nella barra degli indirizzi.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-382">Copy the URL from the address bar, open another browser instance or tab, and paste the URL in the address bar.</span></span>

1. <span data-ttu-id="3fbd9-383">Scegliere browser, immettere un nome e un messaggio e selezionare il pulsante per l'invio del messaggio.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-383">Choose either browser, enter a name and message, and select the button to send the message.</span></span> <span data-ttu-id="3fbd9-384">Il nome e il messaggio vengono visualizzati immediatamente in entrambe le pagine:</span><span class="sxs-lookup"><span data-stu-id="3fbd9-384">The name and message are displayed on both pages instantly:</span></span>

   ![::: NO-LOC (SignalR):::::: NO-LOC (Blazer)::: app di esempio aperta in due finestre del browser che mostrano i messaggi scambiati.](signalr-blazor/_static/signalr-blazor-finished.png)

   <span data-ttu-id="3fbd9-386">Virgolette: *Star Trek VI: il paese non individuato* &copy; 1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)</span><span class="sxs-lookup"><span data-stu-id="3fbd9-386">Quotes: *Star Trek VI: The Undiscovered Country* &copy;1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)</span></span>

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="3fbd9-387">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="3fbd9-387">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

1. <span data-ttu-id="3fbd9-388">Premere <kbd>⌘</kbd> + <kbd>↩</kbd> per eseguire l'app con debug o <kbd>⌥</kbd> + <kbd>⌘</kbd> + <kbd>↩</kbd> per eseguire l'app senza debug.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-388">Press <kbd>⌘</kbd>+<kbd>↩</kbd> to run the app with debugging or <kbd>⌥</kbd>+<kbd>⌘</kbd>+<kbd>↩</kbd> to run the app without debugging.</span></span>

1. <span data-ttu-id="3fbd9-389">Copiare l'URL dalla barra degli indirizzi, aprire un'altra istanza o scheda del browser e incollare l'URL nella barra degli indirizzi.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-389">Copy the URL from the address bar, open another browser instance or tab, and paste the URL in the address bar.</span></span>

1. <span data-ttu-id="3fbd9-390">Scegliere browser, immettere un nome e un messaggio e selezionare il pulsante per l'invio del messaggio.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-390">Choose either browser, enter a name and message, and select the button to send the message.</span></span> <span data-ttu-id="3fbd9-391">Il nome e il messaggio vengono visualizzati immediatamente in entrambe le pagine:</span><span class="sxs-lookup"><span data-stu-id="3fbd9-391">The name and message are displayed on both pages instantly:</span></span>

   ![::: NO-LOC (SignalR):::::: NO-LOC (Blazer)::: app di esempio aperta in due finestre del browser che mostrano i messaggi scambiati.](signalr-blazor/_static/signalr-blazor-finished.png)

   <span data-ttu-id="3fbd9-393">Virgolette: *Star Trek VI: il paese non individuato* &copy; 1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)</span><span class="sxs-lookup"><span data-stu-id="3fbd9-393">Quotes: *Star Trek VI: The Undiscovered Country* &copy;1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)</span></span>

# <a name="net-core-cli"></a>[<span data-ttu-id="3fbd9-394">Interfaccia della riga di comando di .NET Core</span><span class="sxs-lookup"><span data-stu-id="3fbd9-394">.NET Core CLI</span></span>](#tab/netcore-cli/)

1. <span data-ttu-id="3fbd9-395">In una shell dei comandi dalla cartella del progetto, eseguire i comandi seguenti:</span><span class="sxs-lookup"><span data-stu-id="3fbd9-395">In a command shell from the project's folder, execute the following commands:</span></span>

   ```dotnetcli
   dotnet run
   ```

1. <span data-ttu-id="3fbd9-396">Copiare l'URL dalla barra degli indirizzi, aprire un'altra istanza o scheda del browser e incollare l'URL nella barra degli indirizzi.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-396">Copy the URL from the address bar, open another browser instance or tab, and paste the URL in the address bar.</span></span>

1. <span data-ttu-id="3fbd9-397">Scegliere browser, immettere un nome e un messaggio e selezionare il pulsante per l'invio del messaggio.</span><span class="sxs-lookup"><span data-stu-id="3fbd9-397">Choose either browser, enter a name and message, and select the button to send the message.</span></span> <span data-ttu-id="3fbd9-398">Il nome e il messaggio vengono visualizzati immediatamente in entrambe le pagine:</span><span class="sxs-lookup"><span data-stu-id="3fbd9-398">The name and message are displayed on both pages instantly:</span></span>

   ![::: NO-LOC (SignalR):::::: NO-LOC (Blazer)::: app di esempio aperta in due finestre del browser che mostrano i messaggi scambiati.](signalr-blazor/_static/signalr-blazor-finished.png)

   <span data-ttu-id="3fbd9-400">Virgolette: *Star Trek VI: il paese non individuato* &copy; 1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)</span><span class="sxs-lookup"><span data-stu-id="3fbd9-400">Quotes: *Star Trek VI: The Undiscovered Country* &copy;1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)</span></span>

---

::: zone-end

## <a name="next-steps"></a><span data-ttu-id="3fbd9-401">Passaggi successivi</span><span class="sxs-lookup"><span data-stu-id="3fbd9-401">Next steps</span></span>

<span data-ttu-id="3fbd9-402">In questa esercitazione sono state illustrate le procedure per:</span><span class="sxs-lookup"><span data-stu-id="3fbd9-402">In this tutorial, you learned how to:</span></span>

> [!div class="checklist"]
> * <span data-ttu-id="3fbd9-403">Creazione di un Blazor progetto</span><span class="sxs-lookup"><span data-stu-id="3fbd9-403">Create a Blazor project</span></span>
> * <span data-ttu-id="3fbd9-404">Aggiungere la SignalR libreria client</span><span class="sxs-lookup"><span data-stu-id="3fbd9-404">Add the SignalR client library</span></span>
> * <span data-ttu-id="3fbd9-405">Aggiungere un SignalR Hub</span><span class="sxs-lookup"><span data-stu-id="3fbd9-405">Add a SignalR hub</span></span>
> * <span data-ttu-id="3fbd9-406">Aggiungere SignalR Servizi e un endpoint per l' SignalR Hub</span><span class="sxs-lookup"><span data-stu-id="3fbd9-406">Add SignalR services and an endpoint for the SignalR hub</span></span>
> * <span data-ttu-id="3fbd9-407">Aggiungi Razor codice componente per la chat</span><span class="sxs-lookup"><span data-stu-id="3fbd9-407">Add Razor component code for chat</span></span>

<span data-ttu-id="3fbd9-408">Per altre informazioni sulla creazione di Blazor app, vedere la Blazor documentazione:</span><span class="sxs-lookup"><span data-stu-id="3fbd9-408">To learn more about building Blazor apps, see the Blazor documentation:</span></span>

> [!div class="nextstepaction"]
> <span data-ttu-id="3fbd9-409"><xref:blazor/index>
> [Autenticazione del token di porta con Identity Server, WebSocket ed eventi di Server-Sent](xref:signalr/authn-and-authz#bearer-token-authentication)</span><span class="sxs-lookup"><span data-stu-id="3fbd9-409"><xref:blazor/index>
[Bearer token authentication with Identity Server, WebSockets, and Server-Sent Events](xref:signalr/authn-and-authz#bearer-token-authentication)</span></span>

## <a name="additional-resources"></a><span data-ttu-id="3fbd9-410">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="3fbd9-410">Additional resources</span></span>

* <xref:signalr/introduction>
* [<span data-ttu-id="3fbd9-411">SignalR negoziazione tra le origini per l'autenticazione</span><span class="sxs-lookup"><span data-stu-id="3fbd9-411">SignalR cross-origin negotiation for authentication</span></span>](xref:blazor/fundamentals/signalr#signalr-cross-origin-negotiation-for-authentication)
* <xref:blazor/debug>
* <xref:blazor/security/server/threat-mitigation>
