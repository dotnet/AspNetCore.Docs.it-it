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
ms.openlocfilehash: 355db5ad5462747be0058096bc0132ed1071f290
ms.sourcegitcommit: 1166b0ff3828418559510c661e8240e5c5717bb7
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/12/2021
ms.locfileid: "100280988"
---
# <a name="use-aspnet-core-signalr-with-blazor"></a><span data-ttu-id="db903-103">Usare ASP.NET Core SignalR con Blazor</span><span class="sxs-lookup"><span data-stu-id="db903-103">Use ASP.NET Core SignalR with Blazor</span></span>

<span data-ttu-id="db903-104">Questa esercitazione illustra le nozioni di base per la creazione di un'app in tempo reale usando SignalR con Blazor .</span><span class="sxs-lookup"><span data-stu-id="db903-104">This tutorial teaches the basics of building a real-time app using SignalR with Blazor.</span></span> <span data-ttu-id="db903-105">Si apprenderà come:</span><span class="sxs-lookup"><span data-stu-id="db903-105">You learn how to:</span></span>

> [!div class="checklist"]
> * <span data-ttu-id="db903-106">Creazione di un Blazor progetto</span><span class="sxs-lookup"><span data-stu-id="db903-106">Create a Blazor project</span></span>
> * <span data-ttu-id="db903-107">Aggiungere la SignalR libreria client</span><span class="sxs-lookup"><span data-stu-id="db903-107">Add the SignalR client library</span></span>
> * <span data-ttu-id="db903-108">Aggiungere un SignalR Hub</span><span class="sxs-lookup"><span data-stu-id="db903-108">Add a SignalR hub</span></span>
> * <span data-ttu-id="db903-109">Aggiungere SignalR Servizi e un endpoint per l' SignalR Hub</span><span class="sxs-lookup"><span data-stu-id="db903-109">Add SignalR services and an endpoint for the SignalR hub</span></span>
> * <span data-ttu-id="db903-110">Aggiungi Razor codice componente per la chat</span><span class="sxs-lookup"><span data-stu-id="db903-110">Add Razor component code for chat</span></span>

<span data-ttu-id="db903-111">Al termine di questa esercitazione, si disporrà di un'app di chat funzionante.</span><span class="sxs-lookup"><span data-stu-id="db903-111">At the end of this tutorial, you'll have a working chat app.</span></span>

<span data-ttu-id="db903-112">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/tutorials/signalr-blazor/samples/) ([procedura per il download](xref:index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="db903-112">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/tutorials/signalr-blazor/samples/) ([how to download](xref:index#how-to-download-a-sample))</span></span>

## <a name="prerequisites"></a><span data-ttu-id="db903-113">Prerequisiti</span><span class="sxs-lookup"><span data-stu-id="db903-113">Prerequisites</span></span>

::: moniker range=">= aspnetcore-5.0"

# <a name="visual-studio"></a>[<span data-ttu-id="db903-114">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="db903-114">Visual Studio</span></span>](#tab/visual-studio)

* <span data-ttu-id="db903-115">[Visual Studio 2019 16,8 o versione successiva](https://visualstudio.microsoft.com/downloads/?utm_medium=microsoft&utm_source=docs.microsoft.com&utm_campaign=inline+link&utm_content=download+vs2019) con il carico di lavoro **sviluppo di ASP.NET e Web**</span><span class="sxs-lookup"><span data-stu-id="db903-115">[Visual Studio 2019 16.8 or later](https://visualstudio.microsoft.com/downloads/?utm_medium=microsoft&utm_source=docs.microsoft.com&utm_campaign=inline+link&utm_content=download+vs2019) with the **ASP.NET and web development** workload</span></span>
* [!INCLUDE [.NET Core 5.0 SDK](~/includes/5.0-SDK.md)]

# <a name="visual-studio-code"></a>[<span data-ttu-id="db903-116">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="db903-116">Visual Studio Code</span></span>](#tab/visual-studio-code)

[!INCLUDE[](~/includes/net-core-prereqs-vsc-5.0.md)]

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="db903-117">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="db903-117">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

* [<span data-ttu-id="db903-118">Visual Studio per Mac versione 8,8 o successiva</span><span class="sxs-lookup"><span data-stu-id="db903-118">Visual Studio for Mac version 8.8 or later</span></span>](https://visualstudio.microsoft.com/vs/mac/)
* [!INCLUDE [.NET Core 5.0 SDK](~/includes/5.0-SDK.md)]

# <a name="net-core-cli"></a>[<span data-ttu-id="db903-119">Interfaccia della riga di comando di .NET Core</span><span class="sxs-lookup"><span data-stu-id="db903-119">.NET Core CLI</span></span>](#tab/netcore-cli/)

[!INCLUDE[](~/includes/5.0-SDK.md)]

---

::: moniker-end

::: moniker range="< aspnetcore-5.0"

# <a name="visual-studio"></a>[<span data-ttu-id="db903-120">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="db903-120">Visual Studio</span></span>](#tab/visual-studio)

* <span data-ttu-id="db903-121">[Visual Studio 2019 16,6 o versione successiva](https://visualstudio.microsoft.com/downloads/?utm_medium=microsoft&utm_source=docs.microsoft.com&utm_campaign=inline+link&utm_content=download+vs2019) con il carico di lavoro **sviluppo di ASP.NET e Web**</span><span class="sxs-lookup"><span data-stu-id="db903-121">[Visual Studio 2019 16.6 or later](https://visualstudio.microsoft.com/downloads/?utm_medium=microsoft&utm_source=docs.microsoft.com&utm_campaign=inline+link&utm_content=download+vs2019) with the **ASP.NET and web development** workload</span></span>
* [!INCLUDE [.NET Core 3.1 SDK](~/includes/3.1-SDK.md)]

# <a name="visual-studio-code"></a>[<span data-ttu-id="db903-122">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="db903-122">Visual Studio Code</span></span>](#tab/visual-studio-code)

[!INCLUDE[](~/includes/net-core-prereqs-vsc-3.1.md)]

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="db903-123">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="db903-123">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

* [<span data-ttu-id="db903-124">Visual Studio per Mac versione 8,6 o successiva</span><span class="sxs-lookup"><span data-stu-id="db903-124">Visual Studio for Mac version 8.6 or later</span></span>](https://visualstudio.microsoft.com/vs/mac/)
* [!INCLUDE [.NET Core 3.1 SDK](~/includes/3.1-SDK.md)]

# <a name="net-core-cli"></a>[<span data-ttu-id="db903-125">Interfaccia della riga di comando di .NET Core</span><span class="sxs-lookup"><span data-stu-id="db903-125">.NET Core CLI</span></span>](#tab/netcore-cli/)

[!INCLUDE[](~/includes/3.1-SDK.md)]

---

::: moniker-end

::: zone pivot="webassembly"

## <a name="create-a-hosted-blazor-webassembly-app"></a><span data-ttu-id="db903-126">Creare un'app ospitata Blazor WebAssembly</span><span class="sxs-lookup"><span data-stu-id="db903-126">Create a hosted Blazor WebAssembly app</span></span>

<span data-ttu-id="db903-127">Seguire le istruzioni per la scelta degli strumenti:</span><span class="sxs-lookup"><span data-stu-id="db903-127">Follow the guidance for your choice of tooling:</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="db903-128">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="db903-128">Visual Studio</span></span>](#tab/visual-studio)

::: moniker range=">= aspnetcore-5.0"

> [!NOTE]
> <span data-ttu-id="db903-129">Sono necessari Visual Studio 16,8 o versioni successive e .NET Core SDK 5.0.0 o versioni successive.</span><span class="sxs-lookup"><span data-stu-id="db903-129">Visual Studio 16.8 or later and .NET Core SDK 5.0.0 or later are required.</span></span>

::: moniker-end

::: moniker range="< aspnetcore-5.0"

> [!NOTE]
> <span data-ttu-id="db903-130">Sono necessari Visual Studio 16,6 o versioni successive e .NET Core SDK 3.1.300 o versioni successive.</span><span class="sxs-lookup"><span data-stu-id="db903-130">Visual Studio 16.6 or later and .NET Core SDK 3.1.300 or later are required.</span></span>

::: moniker-end

1. <span data-ttu-id="db903-131">Creare un nuovo progetto.</span><span class="sxs-lookup"><span data-stu-id="db903-131">Create a new project.</span></span>

1. <span data-ttu-id="db903-132">Selezionare **Blazor app** e fare clic su **Next (avanti**).</span><span class="sxs-lookup"><span data-stu-id="db903-132">Select **Blazor App** and select **Next**.</span></span>

1. <span data-ttu-id="db903-133">Digitare `BlazorWebAssemblySignalRApp` nel campo **nome progetto** .</span><span class="sxs-lookup"><span data-stu-id="db903-133">Type `BlazorWebAssemblySignalRApp` in the **Project name** field.</span></span> <span data-ttu-id="db903-134">Confermare che la voce relativa al **percorso** sia corretta o specificare un percorso per il progetto.</span><span class="sxs-lookup"><span data-stu-id="db903-134">Confirm the **Location** entry is correct or provide a location for the project.</span></span> <span data-ttu-id="db903-135">Selezionare **Crea**.</span><span class="sxs-lookup"><span data-stu-id="db903-135">Select **Create**.</span></span>

1. <span data-ttu-id="db903-136">Scegliere il modello di **Blazor WebAssembly app** .</span><span class="sxs-lookup"><span data-stu-id="db903-136">Choose the **Blazor WebAssembly App** template.</span></span>

1. <span data-ttu-id="db903-137">In **Avanzate** selezionare la casella di controllo **ASP.NET Core Hosted** .</span><span class="sxs-lookup"><span data-stu-id="db903-137">Under **Advanced**, select the **ASP.NET Core hosted** check box.</span></span>

1. <span data-ttu-id="db903-138">Selezionare **Crea**.</span><span class="sxs-lookup"><span data-stu-id="db903-138">Select **Create**.</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="db903-139">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="db903-139">Visual Studio Code</span></span>](#tab/visual-studio-code)

1. <span data-ttu-id="db903-140">In una shell dei comandi eseguire il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="db903-140">In a command shell, execute the following command:</span></span>

   ```dotnetcli
   dotnet new blazorwasm -ho -o BlazorWebAssemblySignalRApp
   ```

   <span data-ttu-id="db903-141">L' `-ho|--hosted` opzione Crea una soluzione ospitata Blazor WebAssembly .</span><span class="sxs-lookup"><span data-stu-id="db903-141">The `-ho|--hosted` option creates a hosted Blazor WebAssembly solution.</span></span>

   <span data-ttu-id="db903-142">L' `-o|--output` opzione Crea una cartella per la soluzione.</span><span class="sxs-lookup"><span data-stu-id="db903-142">The `-o|--output` option creates a folder for the solution.</span></span> <span data-ttu-id="db903-143">Se è stata creata una cartella per la soluzione e la shell dei comandi è aperta in tale cartella, omettere l' `-o|--output` opzione e il valore per creare la soluzione.</span><span class="sxs-lookup"><span data-stu-id="db903-143">If you've created a folder for the solution and the command shell is open in that folder, omit the `-o|--output` option and value to create the solution.</span></span>

1. <span data-ttu-id="db903-144">In Visual Studio Code aprire la cartella del progetto dell'app.</span><span class="sxs-lookup"><span data-stu-id="db903-144">In Visual Studio Code, open the app's project folder.</span></span>

1. <span data-ttu-id="db903-145">Quando viene visualizzata la finestra di dialogo per aggiungere asset per compilare ed eseguire il debug dell'app, selezionare **Sì**.</span><span class="sxs-lookup"><span data-stu-id="db903-145">When the dialog appears to add assets to build and debug the app, select **Yes**.</span></span> <span data-ttu-id="db903-146">Visual Studio Code aggiunge automaticamente la `.vscode` cartella con `launch.json` `tasks.json` i file e generati.</span><span class="sxs-lookup"><span data-stu-id="db903-146">Visual Studio Code automatically adds the `.vscode` folder with generated `launch.json` and `tasks.json` files.</span></span>

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="db903-147">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="db903-147">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

1. <span data-ttu-id="db903-148">Installare la versione più recente di [Visual Studio per Mac](https://visualstudio.microsoft.com/vs/mac/) e seguire questa procedura:</span><span class="sxs-lookup"><span data-stu-id="db903-148">Install the latest version of [Visual Studio for Mac](https://visualstudio.microsoft.com/vs/mac/) and perform the following steps:</span></span>

1. <span data-ttu-id="db903-149">Selezionare **file**  >  **nuova soluzione** o creare un **nuovo** progetto dalla **finestra Start**.</span><span class="sxs-lookup"><span data-stu-id="db903-149">Select **File** > **New Solution** or create a **New** project from the **Start Window**.</span></span>

1. <span data-ttu-id="db903-150">Nella barra laterale selezionare app **Web e console**  >  .</span><span class="sxs-lookup"><span data-stu-id="db903-150">In the sidebar, select **Web and Console** > **App**.</span></span>

1. <span data-ttu-id="db903-151">Scegliere il modello di **Blazor WebAssembly app** .</span><span class="sxs-lookup"><span data-stu-id="db903-151">Choose the **Blazor WebAssembly App** template.</span></span> <span data-ttu-id="db903-152">Selezionare **Avanti**.</span><span class="sxs-lookup"><span data-stu-id="db903-152">Select **Next**.</span></span>

1. <span data-ttu-id="db903-153">Verificare che **l'autenticazione** sia impostata su **Nessuna autenticazione**.</span><span class="sxs-lookup"><span data-stu-id="db903-153">Confirm that **Authentication** is set to **No Authentication**.</span></span> <span data-ttu-id="db903-154">Selezionare la casella di controllo **ASP.NET Core Hosted** .</span><span class="sxs-lookup"><span data-stu-id="db903-154">Select the **ASP.NET Core Hosted** check box.</span></span> <span data-ttu-id="db903-155">Selezionare **Avanti**.</span><span class="sxs-lookup"><span data-stu-id="db903-155">Select **Next**.</span></span>

1. <span data-ttu-id="db903-156">Nel campo **nome progetto** assegnare un nome all'app `BlazorWebAssemblySignalRApp` .</span><span class="sxs-lookup"><span data-stu-id="db903-156">In the **Project Name** field, name the app `BlazorWebAssemblySignalRApp`.</span></span> <span data-ttu-id="db903-157">Selezionare **Crea**.</span><span class="sxs-lookup"><span data-stu-id="db903-157">Select **Create**.</span></span>

   <span data-ttu-id="db903-158">Se viene visualizzato un messaggio per considerare attendibile il certificato di sviluppo, considerare attendibile il certificato e continuare.</span><span class="sxs-lookup"><span data-stu-id="db903-158">If a prompt appears to trust the development certificate, trust the certificate and continue.</span></span> <span data-ttu-id="db903-159">Per considerare attendibile il certificato, è necessario specificare le password dell'utente e del keychain.</span><span class="sxs-lookup"><span data-stu-id="db903-159">The user and keychain passwords are required to trust the certificate.</span></span>

1. <span data-ttu-id="db903-160">Aprire il progetto passando alla cartella del progetto e aprendo il file di soluzione del progetto ( `.sln` ).</span><span class="sxs-lookup"><span data-stu-id="db903-160">Open the project by navigating to the project folder and opening the project's solution file (`.sln`).</span></span>

# <a name="net-core-cli"></a>[<span data-ttu-id="db903-161">Interfaccia della riga di comando di .NET Core</span><span class="sxs-lookup"><span data-stu-id="db903-161">.NET Core CLI</span></span>](#tab/netcore-cli/)

<span data-ttu-id="db903-162">In una shell dei comandi eseguire il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="db903-162">In a command shell, execute the following command:</span></span>

```dotnetcli
dotnet new blazorwasm -ho -o BlazorWebAssemblySignalRApp
```

<span data-ttu-id="db903-163">L' `-ho|--hosted` opzione Crea una soluzione ospitata Blazor WebAssembly .</span><span class="sxs-lookup"><span data-stu-id="db903-163">The `-ho|--hosted` option creates a hosted Blazor WebAssembly solution.</span></span>

<span data-ttu-id="db903-164">L' `-o|--output` opzione Crea una cartella per la soluzione.</span><span class="sxs-lookup"><span data-stu-id="db903-164">The `-o|--output` option creates a folder for the solution.</span></span> <span data-ttu-id="db903-165">Se è stata creata una cartella per la soluzione e la shell dei comandi è aperta in tale cartella, omettere l' `-o|--output` opzione e il valore per creare la soluzione.</span><span class="sxs-lookup"><span data-stu-id="db903-165">If you've created a folder for the solution and the command shell is open in that folder, omit the `-o|--output` option and value to create the solution.</span></span>

---

## <a name="add-the-signalr-client-library"></a><span data-ttu-id="db903-166">Aggiungere la SignalR libreria client</span><span class="sxs-lookup"><span data-stu-id="db903-166">Add the SignalR client library</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="db903-167">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="db903-167">Visual Studio</span></span>](#tab/visual-studio/)

1. <span data-ttu-id="db903-168">In **Esplora soluzioni** fare clic con il pulsante destro del mouse sul `BlazorWebAssemblySignalRApp.Client` progetto e scegliere **Gestisci pacchetti NuGet**.</span><span class="sxs-lookup"><span data-stu-id="db903-168">In **Solution Explorer**, right-click the `BlazorWebAssemblySignalRApp.Client` project and select **Manage NuGet Packages**.</span></span>

1. <span data-ttu-id="db903-169">Nella finestra di dialogo **Gestisci pacchetti NuGet** verificare che l' **origine del pacchetto** sia impostata su `nuget.org` .</span><span class="sxs-lookup"><span data-stu-id="db903-169">In the **Manage NuGet Packages** dialog, confirm that the **Package source** is set to `nuget.org`.</span></span>

1. <span data-ttu-id="db903-170">Con **Sfoglia** selezionato, digitare `Microsoft.AspNetCore.SignalR.Client` nella casella di ricerca.</span><span class="sxs-lookup"><span data-stu-id="db903-170">With **Browse** selected, type `Microsoft.AspNetCore.SignalR.Client` in the search box.</span></span>

1. <span data-ttu-id="db903-171">Nei risultati della ricerca selezionare il [`Microsoft.AspNetCore.SignalR.Client`](https://www.nuget.org/packages/Microsoft.AspNetCore.SignalR.Client) pacchetto.</span><span class="sxs-lookup"><span data-stu-id="db903-171">In the search results, select the [`Microsoft.AspNetCore.SignalR.Client`](https://www.nuget.org/packages/Microsoft.AspNetCore.SignalR.Client) package.</span></span> <span data-ttu-id="db903-172">Impostare la versione in modo che corrisponda al Framework condiviso dell'app.</span><span class="sxs-lookup"><span data-stu-id="db903-172">Set the version to match the shared framework of the app.</span></span> <span data-ttu-id="db903-173">Selezionare **Installa**.</span><span class="sxs-lookup"><span data-stu-id="db903-173">Select **Install**.</span></span>

1. <span data-ttu-id="db903-174">Se viene visualizzata la finestra di dialogo **Anteprima modifiche** , selezionare **OK**.</span><span class="sxs-lookup"><span data-stu-id="db903-174">If the **Preview Changes** dialog appears, select **OK**.</span></span>

1. <span data-ttu-id="db903-175">Se viene visualizzata la finestra di dialogo **accettazione della licenza** , selezionare Accetto se **si accettano le** condizioni di licenza.</span><span class="sxs-lookup"><span data-stu-id="db903-175">If the **License Acceptance** dialog appears, select **I Accept** if you agree with the license terms.</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="db903-176">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="db903-176">Visual Studio Code</span></span>](#tab/visual-studio-code/)

<span data-ttu-id="db903-177">Nel **terminale integrato** (**visualizzare** il  >  **terminale** dalla barra degli strumenti) eseguire il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="db903-177">In the **Integrated Terminal** (**View** > **Terminal** from the toolbar), execute the following command:</span></span>

```dotnetcli
dotnet add Client package Microsoft.AspNetCore.SignalR.Client
```

<span data-ttu-id="db903-178">Per aggiungere una versione precedente del pacchetto, specificare l' `--version {VERSION}` opzione, dove il `{VERSION}` segnaposto è la versione del pacchetto da aggiungere.</span><span class="sxs-lookup"><span data-stu-id="db903-178">To add an earlier version of the package, supply the `--version {VERSION}` option, where the `{VERSION}` placeholder is the version of the package to add.</span></span>

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="db903-179">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="db903-179">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

1. <span data-ttu-id="db903-180">Nella barra laterale **soluzione** fare clic con il pulsante destro del mouse sul `BlazorWebAssemblySignalRApp.Client` progetto e scegliere **Gestisci pacchetti NuGet**.</span><span class="sxs-lookup"><span data-stu-id="db903-180">In the **Solution** sidebar, right-click the `BlazorWebAssemblySignalRApp.Client` project and select **Manage NuGet Packages**.</span></span>

1. <span data-ttu-id="db903-181">Nella finestra di dialogo **Gestisci pacchetti NuGet** verificare che l'elenco a discesa origine sia impostato su `nuget.org` .</span><span class="sxs-lookup"><span data-stu-id="db903-181">In the **Manage NuGet Packages** dialog, confirm that the source drop-down is set to `nuget.org`.</span></span>

1. <span data-ttu-id="db903-182">Con **Sfoglia** selezionato, digitare `Microsoft.AspNetCore.SignalR.Client` nella casella di ricerca.</span><span class="sxs-lookup"><span data-stu-id="db903-182">With **Browse** selected, type `Microsoft.AspNetCore.SignalR.Client` in the search box.</span></span>

1. <span data-ttu-id="db903-183">Nei risultati della ricerca selezionare la casella di controllo accanto al [`Microsoft.AspNetCore.SignalR.Client`](https://www.nuget.org/packages/Microsoft.AspNetCore.SignalR.Client) pacchetto.</span><span class="sxs-lookup"><span data-stu-id="db903-183">In the search results, select the check box next to the [`Microsoft.AspNetCore.SignalR.Client`](https://www.nuget.org/packages/Microsoft.AspNetCore.SignalR.Client) package.</span></span> <span data-ttu-id="db903-184">Impostare la versione in modo che corrisponda al Framework condiviso dell'app.</span><span class="sxs-lookup"><span data-stu-id="db903-184">Set the version to match the shared framework of the app.</span></span> <span data-ttu-id="db903-185">Selezionare **Aggiungi pacchetto**.</span><span class="sxs-lookup"><span data-stu-id="db903-185">Select **Add Package**.</span></span>

1. <span data-ttu-id="db903-186">Se viene visualizzata la finestra di dialogo **accettazione della licenza** , selezionare **Accetto** se si accettano le condizioni di licenza.</span><span class="sxs-lookup"><span data-stu-id="db903-186">If the **License Acceptance** dialog appears, select **Accept** if you agree with the license terms.</span></span>

# <a name="net-core-cli"></a>[<span data-ttu-id="db903-187">Interfaccia della riga di comando di .NET Core</span><span class="sxs-lookup"><span data-stu-id="db903-187">.NET Core CLI</span></span>](#tab/netcore-cli/)

<span data-ttu-id="db903-188">In una shell dei comandi dalla cartella della soluzione, eseguire il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="db903-188">In a command shell from the solution's folder, execute the following command:</span></span>

```dotnetcli
dotnet add Client package Microsoft.AspNetCore.SignalR.Client
```

<span data-ttu-id="db903-189">Per aggiungere una versione precedente del pacchetto, specificare l' `--version {VERSION}` opzione, dove il `{VERSION}` segnaposto è la versione del pacchetto da aggiungere.</span><span class="sxs-lookup"><span data-stu-id="db903-189">To add an earlier version of the package, supply the `--version {VERSION}` option, where the `{VERSION}` placeholder is the version of the package to add.</span></span>

---

::: moniker range="< aspnetcore-5.0"

## <a name="add-the-systemtextencodingsweb-package"></a><span data-ttu-id="db903-190">Aggiungere il pacchetto System. Text. Encodings. Web</span><span class="sxs-lookup"><span data-stu-id="db903-190">Add the System.Text.Encodings.Web package</span></span>

<span data-ttu-id="db903-191">*Questa sezione si applica solo alle app per ASP.NET Core versione 3. x.*</span><span class="sxs-lookup"><span data-stu-id="db903-191">*This section only applies to apps for ASP.NET Core version 3.x.*</span></span>

<span data-ttu-id="db903-192">A causa di un problema di risoluzione del pacchetto quando si usa [`System.Text.Json`](https://www.nuget.org/packages/System.Text.Json) 5. x in un'app ASP.NET Core 3. x, il `BlazorWebAssemblySignalRApp.Server` progetto richiede un riferimento al pacchetto per [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web) .</span><span class="sxs-lookup"><span data-stu-id="db903-192">Due to a package resolution issue when using [`System.Text.Json`](https://www.nuget.org/packages/System.Text.Json) 5.x in an ASP.NET Core 3.x app, the `BlazorWebAssemblySignalRApp.Server` project requires a package reference for [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web).</span></span> <span data-ttu-id="db903-193">Il problema sottostante verrà risolto in una versione futura della patch di .NET 5.</span><span class="sxs-lookup"><span data-stu-id="db903-193">The underlying issue will be resolved in a future patch release of .NET 5.</span></span> <span data-ttu-id="db903-194">Per altre informazioni, vedere [System.Text.Json definisce netcoreapp 3.0 senza dipendenze (DotNet/runtime #45560)](https://github.com/dotnet/runtime/issues/45560).</span><span class="sxs-lookup"><span data-stu-id="db903-194">For more information, see [System.Text.Json defines netcoreapp3.0 with no dependencies (dotnet/runtime #45560)](https://github.com/dotnet/runtime/issues/45560).</span></span>

<span data-ttu-id="db903-195">Per aggiungere [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web) al `BlazorWebAssemblySignalRApp.Server` progetto della soluzione ospitata ASP.NET Core 3,1 Blazor , seguire le istruzioni per la scelta degli strumenti:</span><span class="sxs-lookup"><span data-stu-id="db903-195">To add [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web) to the `BlazorWebAssemblySignalRApp.Server` project of the ASP.NET Core 3.1 hosted Blazor solution, follow the guidance for your choice of tooling:</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="db903-196">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="db903-196">Visual Studio</span></span>](#tab/visual-studio/)

1. <span data-ttu-id="db903-197">In **Esplora soluzioni** fare clic con il pulsante destro del mouse sul `BlazorWebAssemblySignalRApp.Server` progetto e scegliere **Gestisci pacchetti NuGet**.</span><span class="sxs-lookup"><span data-stu-id="db903-197">In **Solution Explorer**, right-click the `BlazorWebAssemblySignalRApp.Server` project and select **Manage NuGet Packages**.</span></span>

1. <span data-ttu-id="db903-198">Nella finestra di dialogo **Gestisci pacchetti NuGet** verificare che l' **origine del pacchetto** sia impostata su `nuget.org` .</span><span class="sxs-lookup"><span data-stu-id="db903-198">In the **Manage NuGet Packages** dialog, confirm that the **Package source** is set to `nuget.org`.</span></span>

1. <span data-ttu-id="db903-199">Con **Sfoglia** selezionato, digitare `System.Text.Encodings.Web` nella casella di ricerca.</span><span class="sxs-lookup"><span data-stu-id="db903-199">With **Browse** selected, type `System.Text.Encodings.Web` in the search box.</span></span>

1. <span data-ttu-id="db903-200">Nei risultati della ricerca selezionare il [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web) pacchetto.</span><span class="sxs-lookup"><span data-stu-id="db903-200">In the search results, select the [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web) package.</span></span> <span data-ttu-id="db903-201">Consente di selezionare la versione del pacchetto corrispondente al Framework condiviso in uso.</span><span class="sxs-lookup"><span data-stu-id="db903-201">Select the version of the package that matches the shared framework in use.</span></span> <span data-ttu-id="db903-202">Selezionare **Installa**.</span><span class="sxs-lookup"><span data-stu-id="db903-202">Select **Install**.</span></span>

1. <span data-ttu-id="db903-203">Se viene visualizzata la finestra di dialogo **Anteprima modifiche** , selezionare **OK**.</span><span class="sxs-lookup"><span data-stu-id="db903-203">If the **Preview Changes** dialog appears, select **OK**.</span></span>

1. <span data-ttu-id="db903-204">Se viene visualizzata la finestra di dialogo **accettazione della licenza** , selezionare Accetto se **si accettano le** condizioni di licenza.</span><span class="sxs-lookup"><span data-stu-id="db903-204">If the **License Acceptance** dialog appears, select **I Accept** if you agree with the license terms.</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="db903-205">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="db903-205">Visual Studio Code</span></span>](#tab/visual-studio-code/)

<span data-ttu-id="db903-206">Nel **terminale integrato** (**visualizzare** il > **terminale** dalla barra degli strumenti) eseguire i comandi seguenti:</span><span class="sxs-lookup"><span data-stu-id="db903-206">In the **Integrated Terminal** (**View** > **Terminal** from the toolbar), execute the following commands:</span></span>

```dotnetcli
dotnet add Server package System.Text.Encodings.Web
```

<span data-ttu-id="db903-207">Per aggiungere una versione precedente del pacchetto, specificare l' `--version {VERSION}` opzione, dove il `{VERSION}` segnaposto è la versione del pacchetto da aggiungere.</span><span class="sxs-lookup"><span data-stu-id="db903-207">To add an earlier version of the package, supply the `--version {VERSION}` option, where the `{VERSION}` placeholder is the version of the package to add.</span></span>

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="db903-208">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="db903-208">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

1. <span data-ttu-id="db903-209">Nella barra laterale **soluzione** fare clic con il pulsante destro del mouse sul `BlazorWebAssemblySignalRApp.Server` progetto e scegliere **Gestisci pacchetti NuGet**.</span><span class="sxs-lookup"><span data-stu-id="db903-209">In the **Solution** sidebar, right-click the `BlazorWebAssemblySignalRApp.Server` project and select **Manage NuGet Packages**.</span></span>

1. <span data-ttu-id="db903-210">Nella finestra di dialogo **Gestisci pacchetti NuGet** verificare che l'elenco a discesa origine sia impostato su `nuget.org` .</span><span class="sxs-lookup"><span data-stu-id="db903-210">In the **Manage NuGet Packages** dialog, confirm that the source drop-down is set to `nuget.org`.</span></span>

1. <span data-ttu-id="db903-211">Con **Sfoglia** selezionato, digitare `System.Text.Encodings.Web` nella casella di ricerca.</span><span class="sxs-lookup"><span data-stu-id="db903-211">With **Browse** selected, type `System.Text.Encodings.Web` in the search box.</span></span>

1. <span data-ttu-id="db903-212">Nei risultati della ricerca selezionare la casella di controllo accanto al [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web) pacchetto, selezionare la versione corretta del pacchetto corrispondente al Framework condiviso in uso e selezionare **Aggiungi pacchetto**.</span><span class="sxs-lookup"><span data-stu-id="db903-212">In the search results, select the check box next to the [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web) package, select the correct version of the package that matches the shared framework in use, and select **Add Package**.</span></span>

1. <span data-ttu-id="db903-213">Se viene visualizzata la finestra di dialogo **accettazione della licenza** , selezionare **Accetto** se si accettano le condizioni di licenza.</span><span class="sxs-lookup"><span data-stu-id="db903-213">If the **License Acceptance** dialog appears, select **Accept** if you agree with the license terms.</span></span>

# <a name="net-core-cli"></a>[<span data-ttu-id="db903-214">Interfaccia della riga di comando di .NET Core</span><span class="sxs-lookup"><span data-stu-id="db903-214">.NET Core CLI</span></span>](#tab/netcore-cli/)

<span data-ttu-id="db903-215">In una shell dei comandi dalla cartella della soluzione, eseguire il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="db903-215">In a command shell from the solution's folder, execute the following command:</span></span>

```dotnetcli
dotnet add Server package System.Text.Encodings.Web
```

<span data-ttu-id="db903-216">Per aggiungere una versione precedente del pacchetto, specificare l' `--version {VERSION}` opzione, dove il `{VERSION}` segnaposto è la versione del pacchetto da aggiungere.</span><span class="sxs-lookup"><span data-stu-id="db903-216">To add an earlier version of the package, supply the `--version {VERSION}` option, where the `{VERSION}` placeholder is the version of the package to add.</span></span>

---

::: moniker-end

## <a name="add-a-signalr-hub"></a><span data-ttu-id="db903-217">Aggiungere un SignalR Hub</span><span class="sxs-lookup"><span data-stu-id="db903-217">Add a SignalR hub</span></span>

<span data-ttu-id="db903-218">Nel `BlazorWebAssemblySignalRApp.Server` progetto creare una `Hubs` cartella (plurale) e aggiungere la classe seguente `ChatHub` ( `Hubs/ChatHub.cs` ):</span><span class="sxs-lookup"><span data-stu-id="db903-218">In the `BlazorWebAssemblySignalRApp.Server` project, create a `Hubs` (plural) folder and add the following `ChatHub` class (`Hubs/ChatHub.cs`):</span></span>

::: moniker range=">= aspnetcore-5.0"

[!code-csharp[](signalr-blazor/samples/5.x/BlazorWebAssemblySignalRApp/Server/Hubs/ChatHub.cs)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-csharp[](signalr-blazor/samples/3.x/BlazorWebAssemblySignalRApp/Server/Hubs/ChatHub.cs)]

::: moniker-end

## <a name="add-services-and-an-endpoint-for-the-signalr-hub"></a><span data-ttu-id="db903-219">Aggiungere servizi e un endpoint per l' SignalR Hub</span><span class="sxs-lookup"><span data-stu-id="db903-219">Add services and an endpoint for the SignalR hub</span></span>

1. <span data-ttu-id="db903-220">Nel progetto `BlazorWebAssemblySignalRApp.Server` aprire il file `Startup.cs`.</span><span class="sxs-lookup"><span data-stu-id="db903-220">In the `BlazorWebAssemblySignalRApp.Server` project, open the `Startup.cs` file.</span></span>

1. <span data-ttu-id="db903-221">Aggiungere lo spazio dei nomi per la `ChatHub` classe all'inizio del file:</span><span class="sxs-lookup"><span data-stu-id="db903-221">Add the namespace for the `ChatHub` class to the top of the file:</span></span>

   ```csharp
   using BlazorWebAssemblySignalRApp.Server.Hubs;
   ```

::: moniker range=">= aspnetcore-5.0"

1. <span data-ttu-id="db903-222">Aggiungere SignalR e rispondere ai servizi middleware di compressione per `Startup.ConfigureServices` :</span><span class="sxs-lookup"><span data-stu-id="db903-222">Add SignalR and Response Compression Middleware services to `Startup.ConfigureServices`:</span></span>

   [!code-csharp[](signalr-blazor/samples/5.x/BlazorWebAssemblySignalRApp/Server/Startup.cs?name=snippet_ConfigureServices&highlight=3,6-10)]

1. <span data-ttu-id="db903-223">In `Startup.Configure`:</span><span class="sxs-lookup"><span data-stu-id="db903-223">In `Startup.Configure`:</span></span>

   * <span data-ttu-id="db903-224">Usare il middleware della compressione della risposta nella parte superiore della configurazione della pipeline di elaborazione.</span><span class="sxs-lookup"><span data-stu-id="db903-224">Use Response Compression Middleware at the top of the processing pipeline's configuration.</span></span>
   * <span data-ttu-id="db903-225">Tra gli endpoint per i controller e il fallback sul lato client, aggiungere un endpoint per l'hub.</span><span class="sxs-lookup"><span data-stu-id="db903-225">Between the endpoints for controllers and the client-side fallback, add an endpoint for the hub.</span></span>

   [!code-csharp[](signalr-blazor/samples/5.x/BlazorWebAssemblySignalRApp/Server/Startup.cs?name=snippet_Configure&highlight=3,26)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

1. <span data-ttu-id="db903-226">Aggiungere SignalR e rispondere ai servizi middleware di compressione per `Startup.ConfigureServices` :</span><span class="sxs-lookup"><span data-stu-id="db903-226">Add SignalR and Response Compression Middleware services to `Startup.ConfigureServices`:</span></span>

   [!code-csharp[](signalr-blazor/samples/3.x/BlazorWebAssemblySignalRApp/Server/Startup.cs?name=snippet_ConfigureServices&highlight=3,5-9)]

1. <span data-ttu-id="db903-227">In `Startup.Configure`:</span><span class="sxs-lookup"><span data-stu-id="db903-227">In `Startup.Configure`:</span></span>

   * <span data-ttu-id="db903-228">Usare il middleware della compressione della risposta nella parte superiore della configurazione della pipeline di elaborazione.</span><span class="sxs-lookup"><span data-stu-id="db903-228">Use Response Compression Middleware at the top of the processing pipeline's configuration.</span></span>
   * <span data-ttu-id="db903-229">Tra gli endpoint per i controller e il fallback sul lato client, aggiungere un endpoint per l'hub.</span><span class="sxs-lookup"><span data-stu-id="db903-229">Between the endpoints for controllers and the client-side fallback, add an endpoint for the hub.</span></span>

   [!code-csharp[](signalr-blazor/samples/3.x/BlazorWebAssemblySignalRApp/Server/Startup.cs?name=snippet_Configure&highlight=3,25)]

::: moniker-end

## <a name="add-razor-component-code-for-chat"></a><span data-ttu-id="db903-230">Aggiungi Razor codice componente per la chat</span><span class="sxs-lookup"><span data-stu-id="db903-230">Add Razor component code for chat</span></span>

1. <span data-ttu-id="db903-231">Nel progetto `BlazorWebAssemblySignalRApp.Client` aprire il file `Pages/Index.razor`.</span><span class="sxs-lookup"><span data-stu-id="db903-231">In the `BlazorWebAssemblySignalRApp.Client` project, open the `Pages/Index.razor` file.</span></span>

::: moniker range=">= aspnetcore-5.0"

1. <span data-ttu-id="db903-232">Sostituire il markup con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="db903-232">Replace the markup with the following code:</span></span>

   [!code-razor[](signalr-blazor/samples/5.x/BlazorWebAssemblySignalRApp/Client/Pages/Index.razor)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

1. <span data-ttu-id="db903-233">Sostituire il markup con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="db903-233">Replace the markup with the following code:</span></span>

   [!code-razor[](signalr-blazor/samples/3.x/BlazorWebAssemblySignalRApp/Client/Pages/Index.razor)]

::: moniker-end

## <a name="run-the-app"></a><span data-ttu-id="db903-234">Eseguire l'app</span><span class="sxs-lookup"><span data-stu-id="db903-234">Run the app</span></span>

<span data-ttu-id="db903-235">Seguire le istruzioni per gli strumenti:</span><span class="sxs-lookup"><span data-stu-id="db903-235">Follow the guidance for your tooling:</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="db903-236">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="db903-236">Visual Studio</span></span>](#tab/visual-studio)

1. <span data-ttu-id="db903-237">In **Esplora soluzioni** selezionare il `BlazorWebAssemblySignalRApp.Server` progetto.</span><span class="sxs-lookup"><span data-stu-id="db903-237">In **Solution Explorer**, select the `BlazorWebAssemblySignalRApp.Server` project.</span></span> <span data-ttu-id="db903-238">Premere <kbd>F5</kbd> per eseguire l'app con il debug o <kbd>CTRL</kbd> + <kbd>F5</kbd> per eseguire l'app senza debug.</span><span class="sxs-lookup"><span data-stu-id="db903-238">Press <kbd>F5</kbd> to run the app with debugging or <kbd>Ctrl</kbd>+<kbd>F5</kbd> to run the app without debugging.</span></span>

1. <span data-ttu-id="db903-239">Copiare l'URL dalla barra degli indirizzi, aprire un'altra istanza o scheda del browser e incollare l'URL nella barra degli indirizzi.</span><span class="sxs-lookup"><span data-stu-id="db903-239">Copy the URL from the address bar, open another browser instance or tab, and paste the URL in the address bar.</span></span>

1. <span data-ttu-id="db903-240">Scegliere browser, immettere un nome e un messaggio e selezionare il pulsante per l'invio del messaggio.</span><span class="sxs-lookup"><span data-stu-id="db903-240">Choose either browser, enter a name and message, and select the button to send the message.</span></span> <span data-ttu-id="db903-241">Il nome e il messaggio vengono visualizzati immediatamente in entrambe le pagine:</span><span class="sxs-lookup"><span data-stu-id="db903-241">The name and message are displayed on both pages instantly:</span></span>

   ![::: NO-LOC (SignalR):::::: NO-LOC (Blazer)::: app di esempio aperta in due finestre del browser che mostrano i messaggi scambiati.](signalr-blazor/_static/signalr-blazor-finished.png)

   <span data-ttu-id="db903-243">Virgolette: *Star Trek VI: il paese non individuato* &copy; 1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)</span><span class="sxs-lookup"><span data-stu-id="db903-243">Quotes: *Star Trek VI: The Undiscovered Country* &copy;1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="db903-244">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="db903-244">Visual Studio Code</span></span>](#tab/visual-studio-code)

1. <span data-ttu-id="db903-245">Premere <kbd>F5</kbd> per eseguire l'app con il debug o <kbd>CTRL</kbd> + <kbd>F5</kbd> per eseguire l'app senza debug.</span><span class="sxs-lookup"><span data-stu-id="db903-245">Press <kbd>F5</kbd> to run the app with debugging or <kbd>Ctrl</kbd>+<kbd>F5</kbd> to run the app without debugging.</span></span>

1. <span data-ttu-id="db903-246">Copiare l'URL dalla barra degli indirizzi, aprire un'altra istanza o scheda del browser e incollare l'URL nella barra degli indirizzi.</span><span class="sxs-lookup"><span data-stu-id="db903-246">Copy the URL from the address bar, open another browser instance or tab, and paste the URL in the address bar.</span></span>

1. <span data-ttu-id="db903-247">Scegliere browser, immettere un nome e un messaggio e selezionare il pulsante per l'invio del messaggio.</span><span class="sxs-lookup"><span data-stu-id="db903-247">Choose either browser, enter a name and message, and select the button to send the message.</span></span> <span data-ttu-id="db903-248">Il nome e il messaggio vengono visualizzati immediatamente in entrambe le pagine:</span><span class="sxs-lookup"><span data-stu-id="db903-248">The name and message are displayed on both pages instantly:</span></span>

   ![::: NO-LOC (SignalR):::::: NO-LOC (Blazer)::: app di esempio aperta in due finestre del browser che mostrano i messaggi scambiati.](signalr-blazor/_static/signalr-blazor-finished.png)

   <span data-ttu-id="db903-250">Virgolette: *Star Trek VI: il paese non individuato* &copy; 1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)</span><span class="sxs-lookup"><span data-stu-id="db903-250">Quotes: *Star Trek VI: The Undiscovered Country* &copy;1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)</span></span>

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="db903-251">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="db903-251">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

1. <span data-ttu-id="db903-252">Nella barra laterale della **soluzione** selezionare il `BlazorWebAssemblySignalRApp.Server` progetto.</span><span class="sxs-lookup"><span data-stu-id="db903-252">In the **Solution** sidebar, select the `BlazorWebAssemblySignalRApp.Server` project.</span></span> <span data-ttu-id="db903-253">Premere <kbd>⌘</kbd> + <kbd>↩</kbd> per eseguire l'app con debug o <kbd>⌥</kbd> + <kbd>⌘</kbd> + <kbd>↩</kbd> per eseguire l'app senza debug.</span><span class="sxs-lookup"><span data-stu-id="db903-253">Press <kbd>⌘</kbd>+<kbd>↩</kbd> to run the app with debugging or <kbd>⌥</kbd>+<kbd>⌘</kbd>+<kbd>↩</kbd> to run the app without debugging.</span></span>

1. <span data-ttu-id="db903-254">Copiare l'URL dalla barra degli indirizzi, aprire un'altra istanza o scheda del browser e incollare l'URL nella barra degli indirizzi.</span><span class="sxs-lookup"><span data-stu-id="db903-254">Copy the URL from the address bar, open another browser instance or tab, and paste the URL in the address bar.</span></span>

1. <span data-ttu-id="db903-255">Scegliere browser, immettere un nome e un messaggio e selezionare il pulsante per l'invio del messaggio.</span><span class="sxs-lookup"><span data-stu-id="db903-255">Choose either browser, enter a name and message, and select the button to send the message.</span></span> <span data-ttu-id="db903-256">Il nome e il messaggio vengono visualizzati immediatamente in entrambe le pagine:</span><span class="sxs-lookup"><span data-stu-id="db903-256">The name and message are displayed on both pages instantly:</span></span>

   ![::: NO-LOC (SignalR):::::: NO-LOC (Blazer)::: app di esempio aperta in due finestre del browser che mostrano i messaggi scambiati.](signalr-blazor/_static/signalr-blazor-finished.png)

   <span data-ttu-id="db903-258">Virgolette: *Star Trek VI: il paese non individuato* &copy; 1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)</span><span class="sxs-lookup"><span data-stu-id="db903-258">Quotes: *Star Trek VI: The Undiscovered Country* &copy;1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)</span></span>

# <a name="net-core-cli"></a>[<span data-ttu-id="db903-259">Interfaccia della riga di comando di .NET Core</span><span class="sxs-lookup"><span data-stu-id="db903-259">.NET Core CLI</span></span>](#tab/netcore-cli/)

1. <span data-ttu-id="db903-260">In una shell dei comandi dalla cartella della soluzione eseguire i comandi seguenti:</span><span class="sxs-lookup"><span data-stu-id="db903-260">In a command shell from the solution's folder, execute the following commands:</span></span>

   ```dotnetcli
   cd Server
   dotnet run
   ```

1. <span data-ttu-id="db903-261">Copiare l'URL dalla barra degli indirizzi, aprire un'altra istanza o scheda del browser e incollare l'URL nella barra degli indirizzi.</span><span class="sxs-lookup"><span data-stu-id="db903-261">Copy the URL from the address bar, open another browser instance or tab, and paste the URL in the address bar.</span></span>

1. <span data-ttu-id="db903-262">Scegliere browser, immettere un nome e un messaggio e selezionare il pulsante per l'invio del messaggio.</span><span class="sxs-lookup"><span data-stu-id="db903-262">Choose either browser, enter a name and message, and select the button to send the message.</span></span> <span data-ttu-id="db903-263">Il nome e il messaggio vengono visualizzati immediatamente in entrambe le pagine:</span><span class="sxs-lookup"><span data-stu-id="db903-263">The name and message are displayed on both pages instantly:</span></span>

   ![::: NO-LOC (SignalR):::::: NO-LOC (Blazer)::: app di esempio aperta in due finestre del browser che mostrano i messaggi scambiati.](signalr-blazor/_static/signalr-blazor-finished.png)

   <span data-ttu-id="db903-265">Virgolette: *Star Trek VI: il paese non individuato* &copy; 1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)</span><span class="sxs-lookup"><span data-stu-id="db903-265">Quotes: *Star Trek VI: The Undiscovered Country* &copy;1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)</span></span>

---

::: zone-end

::: zone pivot="server"

## <a name="create-a-blazor-server-app"></a><span data-ttu-id="db903-266">Creare un' Blazor Server app</span><span class="sxs-lookup"><span data-stu-id="db903-266">Create a Blazor Server app</span></span>

<span data-ttu-id="db903-267">Seguire le istruzioni per la scelta degli strumenti:</span><span class="sxs-lookup"><span data-stu-id="db903-267">Follow the guidance for your choice of tooling:</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="db903-268">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="db903-268">Visual Studio</span></span>](#tab/visual-studio)

::: moniker range=">= aspnetcore-5.0"

> [!NOTE]
> <span data-ttu-id="db903-269">Sono necessari Visual Studio 16,8 o versioni successive e .NET Core SDK 5.0.0 o versioni successive.</span><span class="sxs-lookup"><span data-stu-id="db903-269">Visual Studio 16.8 or later and .NET Core SDK 5.0.0 or later are required.</span></span>

::: moniker-end

::: moniker range="< aspnetcore-5.0"

> [!NOTE]
> <span data-ttu-id="db903-270">Sono necessari Visual Studio 16,6 o versioni successive e .NET Core SDK 3.1.300 o versioni successive.</span><span class="sxs-lookup"><span data-stu-id="db903-270">Visual Studio 16.6 or later and .NET Core SDK 3.1.300 or later are required.</span></span>

::: moniker-end

1. <span data-ttu-id="db903-271">Creare un nuovo progetto.</span><span class="sxs-lookup"><span data-stu-id="db903-271">Create a new project.</span></span>

1. <span data-ttu-id="db903-272">Selezionare **Blazor app** e fare clic su **Next (avanti**).</span><span class="sxs-lookup"><span data-stu-id="db903-272">Select **Blazor App** and select **Next**.</span></span>

1. <span data-ttu-id="db903-273">Digitare `BlazorServerSignalRApp` nel campo **nome progetto** .</span><span class="sxs-lookup"><span data-stu-id="db903-273">Type `BlazorServerSignalRApp` in the **Project name** field.</span></span> <span data-ttu-id="db903-274">Confermare che la voce relativa al **percorso** sia corretta o specificare un percorso per il progetto.</span><span class="sxs-lookup"><span data-stu-id="db903-274">Confirm the **Location** entry is correct or provide a location for the project.</span></span> <span data-ttu-id="db903-275">Selezionare **Crea**.</span><span class="sxs-lookup"><span data-stu-id="db903-275">Select **Create**.</span></span>

1. <span data-ttu-id="db903-276">Scegliere il modello di **Blazor Server app** .</span><span class="sxs-lookup"><span data-stu-id="db903-276">Choose the **Blazor Server App** template.</span></span>

1. <span data-ttu-id="db903-277">Selezionare **Crea**.</span><span class="sxs-lookup"><span data-stu-id="db903-277">Select **Create**.</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="db903-278">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="db903-278">Visual Studio Code</span></span>](#tab/visual-studio-code)

1. <span data-ttu-id="db903-279">In una shell dei comandi eseguire il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="db903-279">In a command shell, execute the following command:</span></span>

   ```dotnetcli
   dotnet new blazorserver -o BlazorServerSignalRApp
   ```

   <span data-ttu-id="db903-280">L' `-o|--output` opzione Crea una cartella per il progetto.</span><span class="sxs-lookup"><span data-stu-id="db903-280">The `-o|--output` option creates a folder for the project.</span></span> <span data-ttu-id="db903-281">Se è stata creata una cartella per il progetto e la shell dei comandi è aperta in tale cartella, omettere l' `-o|--output` opzione e il valore per creare il progetto.</span><span class="sxs-lookup"><span data-stu-id="db903-281">If you've created a folder for the project and the command shell is open in that folder, omit the `-o|--output` option and value to create the project.</span></span>

1. <span data-ttu-id="db903-282">In Visual Studio Code aprire la cartella del progetto dell'app.</span><span class="sxs-lookup"><span data-stu-id="db903-282">In Visual Studio Code, open the app's project folder.</span></span>

1. <span data-ttu-id="db903-283">Quando viene visualizzata la finestra di dialogo per aggiungere asset per compilare ed eseguire il debug dell'app, selezionare **Sì**.</span><span class="sxs-lookup"><span data-stu-id="db903-283">When the dialog appears to add assets to build and debug the app, select **Yes**.</span></span> <span data-ttu-id="db903-284">Visual Studio Code aggiunge automaticamente la `.vscode` cartella con `launch.json` `tasks.json` i file e generati.</span><span class="sxs-lookup"><span data-stu-id="db903-284">Visual Studio Code automatically adds the `.vscode` folder with generated `launch.json` and `tasks.json` files.</span></span>

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="db903-285">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="db903-285">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

1. <span data-ttu-id="db903-286">Installare la versione più recente di [Visual Studio per Mac](https://visualstudio.microsoft.com/vs/mac/) e seguire questa procedura:</span><span class="sxs-lookup"><span data-stu-id="db903-286">Install the latest version of [Visual Studio for Mac](https://visualstudio.microsoft.com/vs/mac/) and perform the following steps:</span></span>

1. <span data-ttu-id="db903-287">Selezionare **file**  >  **nuova soluzione** o creare un **nuovo** progetto dalla **finestra Start**.</span><span class="sxs-lookup"><span data-stu-id="db903-287">Select **File** > **New Solution** or create a **New** project from the **Start Window**.</span></span>

1. <span data-ttu-id="db903-288">Nella barra laterale selezionare app **Web e console**  >  .</span><span class="sxs-lookup"><span data-stu-id="db903-288">In the sidebar, select **Web and Console** > **App**.</span></span>

1. <span data-ttu-id="db903-289">Scegliere il modello di **Blazor Server app** .</span><span class="sxs-lookup"><span data-stu-id="db903-289">Choose the **Blazor Server App** template.</span></span> <span data-ttu-id="db903-290">Selezionare **Avanti**.</span><span class="sxs-lookup"><span data-stu-id="db903-290">Select **Next**.</span></span>

1. <span data-ttu-id="db903-291">Verificare che **l'autenticazione** sia impostata su **Nessuna autenticazione**.</span><span class="sxs-lookup"><span data-stu-id="db903-291">Confirm that **Authentication** is set to **No Authentication**.</span></span> <span data-ttu-id="db903-292">Selezionare **Avanti**.</span><span class="sxs-lookup"><span data-stu-id="db903-292">Select **Next**.</span></span>

1. <span data-ttu-id="db903-293">Nel campo **nome progetto** assegnare un nome all'app `BlazorServerSignalRApp` .</span><span class="sxs-lookup"><span data-stu-id="db903-293">In the **Project Name** field, name the app `BlazorServerSignalRApp`.</span></span> <span data-ttu-id="db903-294">Selezionare **Crea**.</span><span class="sxs-lookup"><span data-stu-id="db903-294">Select **Create**.</span></span>

   <span data-ttu-id="db903-295">Se viene visualizzato un messaggio per considerare attendibile il certificato di sviluppo, considerare attendibile il certificato e continuare.</span><span class="sxs-lookup"><span data-stu-id="db903-295">If a prompt appears to trust the development certificate, trust the certificate and continue.</span></span> <span data-ttu-id="db903-296">Per considerare attendibile il certificato, è necessario specificare le password dell'utente e del keychain.</span><span class="sxs-lookup"><span data-stu-id="db903-296">The user and keychain passwords are required to trust the certificate.</span></span>

1. <span data-ttu-id="db903-297">Aprire il progetto passando alla cartella del progetto e aprendo il file di soluzione del progetto ( `.sln` ).</span><span class="sxs-lookup"><span data-stu-id="db903-297">Open the project by navigating to the project folder and opening the project's solution file (`.sln`).</span></span>

# <a name="net-core-cli"></a>[<span data-ttu-id="db903-298">Interfaccia della riga di comando di .NET Core</span><span class="sxs-lookup"><span data-stu-id="db903-298">.NET Core CLI</span></span>](#tab/netcore-cli/)

<span data-ttu-id="db903-299">In una shell dei comandi eseguire il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="db903-299">In a command shell, execute the following command:</span></span>

```dotnetcli
dotnet new blazorserver -o BlazorServerSignalRApp
```

<span data-ttu-id="db903-300">L' `-o|--output` opzione Crea una cartella per il progetto.</span><span class="sxs-lookup"><span data-stu-id="db903-300">The `-o|--output` option creates a folder for the project.</span></span> <span data-ttu-id="db903-301">Se è stata creata una cartella per il progetto e la shell dei comandi è aperta in tale cartella, omettere l' `-o|--output` opzione e il valore per creare il progetto.</span><span class="sxs-lookup"><span data-stu-id="db903-301">If you've created a folder for the project and the command shell is open in that folder, omit the `-o|--output` option and value to create the project.</span></span>

---

## <a name="add-the-signalr-client-library"></a><span data-ttu-id="db903-302">Aggiungere la SignalR libreria client</span><span class="sxs-lookup"><span data-stu-id="db903-302">Add the SignalR client library</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="db903-303">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="db903-303">Visual Studio</span></span>](#tab/visual-studio/)

1. <span data-ttu-id="db903-304">In **Esplora soluzioni** fare clic con il pulsante destro del mouse sul `BlazorServerSignalRApp` progetto e scegliere **Gestisci pacchetti NuGet**.</span><span class="sxs-lookup"><span data-stu-id="db903-304">In **Solution Explorer**, right-click the `BlazorServerSignalRApp` project and select **Manage NuGet Packages**.</span></span>

1. <span data-ttu-id="db903-305">Nella finestra di dialogo **Gestisci pacchetti NuGet** verificare che l' **origine del pacchetto** sia impostata su `nuget.org` .</span><span class="sxs-lookup"><span data-stu-id="db903-305">In the **Manage NuGet Packages** dialog, confirm that the **Package source** is set to `nuget.org`.</span></span>

1. <span data-ttu-id="db903-306">Con **Sfoglia** selezionato, digitare `Microsoft.AspNetCore.SignalR.Client` nella casella di ricerca.</span><span class="sxs-lookup"><span data-stu-id="db903-306">With **Browse** selected, type `Microsoft.AspNetCore.SignalR.Client` in the search box.</span></span>

1. <span data-ttu-id="db903-307">Nei risultati della ricerca selezionare il [`Microsoft.AspNetCore.SignalR.Client`](https://www.nuget.org/packages/Microsoft.AspNetCore.SignalR.Client) pacchetto.</span><span class="sxs-lookup"><span data-stu-id="db903-307">In the search results, select the [`Microsoft.AspNetCore.SignalR.Client`](https://www.nuget.org/packages/Microsoft.AspNetCore.SignalR.Client) package.</span></span> <span data-ttu-id="db903-308">Impostare la versione in modo che corrisponda al Framework condiviso dell'app.</span><span class="sxs-lookup"><span data-stu-id="db903-308">Set the version to match the shared framework of the app.</span></span> <span data-ttu-id="db903-309">Selezionare **Installa**.</span><span class="sxs-lookup"><span data-stu-id="db903-309">Select **Install**.</span></span>

1. <span data-ttu-id="db903-310">Se viene visualizzata la finestra di dialogo **Anteprima modifiche** , selezionare **OK**.</span><span class="sxs-lookup"><span data-stu-id="db903-310">If the **Preview Changes** dialog appears, select **OK**.</span></span>

1. <span data-ttu-id="db903-311">Se viene visualizzata la finestra di dialogo **accettazione della licenza** , selezionare Accetto se **si accettano le** condizioni di licenza.</span><span class="sxs-lookup"><span data-stu-id="db903-311">If the **License Acceptance** dialog appears, select **I Accept** if you agree with the license terms.</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="db903-312">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="db903-312">Visual Studio Code</span></span>](#tab/visual-studio-code/)

<span data-ttu-id="db903-313">Nel **terminale integrato** (**visualizzare** il  >  **terminale** dalla barra degli strumenti) eseguire il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="db903-313">In the **Integrated Terminal** (**View** > **Terminal** from the toolbar), execute the following command:</span></span>

```dotnetcli
dotnet add package Microsoft.AspNetCore.SignalR.Client
```

<span data-ttu-id="db903-314">Per aggiungere una versione precedente del pacchetto, specificare l' `--version {VERSION}` opzione, dove il `{VERSION}` segnaposto è la versione del pacchetto da aggiungere.</span><span class="sxs-lookup"><span data-stu-id="db903-314">To add an earlier version of the package, supply the `--version {VERSION}` option, where the `{VERSION}` placeholder is the version of the package to add.</span></span>

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="db903-315">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="db903-315">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

1. <span data-ttu-id="db903-316">Nella barra laterale **soluzione** fare clic con il pulsante destro del mouse sul `BlazorServerSignalRApp` progetto e scegliere **Gestisci pacchetti NuGet**.</span><span class="sxs-lookup"><span data-stu-id="db903-316">In the **Solution** sidebar, right-click the `BlazorServerSignalRApp` project and select **Manage NuGet Packages**.</span></span>

1. <span data-ttu-id="db903-317">Nella finestra di dialogo **Gestisci pacchetti NuGet** verificare che l'elenco a discesa origine sia impostato su `nuget.org` .</span><span class="sxs-lookup"><span data-stu-id="db903-317">In the **Manage NuGet Packages** dialog, confirm that the source drop-down is set to `nuget.org`.</span></span>

1. <span data-ttu-id="db903-318">Con **Sfoglia** selezionato, digitare `Microsoft.AspNetCore.SignalR.Client` nella casella di ricerca.</span><span class="sxs-lookup"><span data-stu-id="db903-318">With **Browse** selected, type `Microsoft.AspNetCore.SignalR.Client` in the search box.</span></span>

1. <span data-ttu-id="db903-319">Nei risultati della ricerca selezionare la casella di controllo accanto al [`Microsoft.AspNetCore.SignalR.Client`](https://www.nuget.org/packages/Microsoft.AspNetCore.SignalR.Client) pacchetto.</span><span class="sxs-lookup"><span data-stu-id="db903-319">In the search results, select the check box next to the [`Microsoft.AspNetCore.SignalR.Client`](https://www.nuget.org/packages/Microsoft.AspNetCore.SignalR.Client) package.</span></span> <span data-ttu-id="db903-320">Impostare la versione in modo che corrisponda al Framework condiviso dell'app.</span><span class="sxs-lookup"><span data-stu-id="db903-320">Set the version to match the shared framework of the app.</span></span> <span data-ttu-id="db903-321">Selezionare **Aggiungi pacchetto**.</span><span class="sxs-lookup"><span data-stu-id="db903-321">Select **Add Package**.</span></span>

1. <span data-ttu-id="db903-322">Se viene visualizzata la finestra di dialogo **accettazione della licenza** , selezionare **Accetto** se si accettano le condizioni di licenza.</span><span class="sxs-lookup"><span data-stu-id="db903-322">If the **License Acceptance** dialog appears, select **Accept** if you agree with the license terms.</span></span>

# <a name="net-core-cli"></a>[<span data-ttu-id="db903-323">Interfaccia della riga di comando di .NET Core</span><span class="sxs-lookup"><span data-stu-id="db903-323">.NET Core CLI</span></span>](#tab/netcore-cli/)

<span data-ttu-id="db903-324">In una shell dei comandi dalla cartella del progetto, eseguire il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="db903-324">In a command shell from the project's folder, execute the following command:</span></span>

```dotnetcli
dotnet add package Microsoft.AspNetCore.SignalR.Client
```

<span data-ttu-id="db903-325">Per aggiungere una versione precedente del pacchetto, specificare l' `--version {VERSION}` opzione, dove il `{VERSION}` segnaposto è la versione del pacchetto da aggiungere.</span><span class="sxs-lookup"><span data-stu-id="db903-325">To add an earlier version of the package, supply the `--version {VERSION}` option, where the `{VERSION}` placeholder is the version of the package to add.</span></span>

---

::: moniker range="< aspnetcore-5.0"

## <a name="add-the-systemtextencodingsweb-package"></a><span data-ttu-id="db903-326">Aggiungere il pacchetto System. Text. Encodings. Web</span><span class="sxs-lookup"><span data-stu-id="db903-326">Add the System.Text.Encodings.Web package</span></span>

<span data-ttu-id="db903-327">*Questa sezione si applica solo alle app per ASP.NET Core versione 3. x.*</span><span class="sxs-lookup"><span data-stu-id="db903-327">*This section only applies to apps for ASP.NET Core version 3.x.*</span></span>

<span data-ttu-id="db903-328">A causa di un problema di risoluzione del pacchetto quando si usa [`System.Text.Json`](https://www.nuget.org/packages/System.Text.Json) 5. x in un'app ASP.NET Core 3. x, il progetto richiede un riferimento al pacchetto per [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web) .</span><span class="sxs-lookup"><span data-stu-id="db903-328">Due to a package resolution issue when using [`System.Text.Json`](https://www.nuget.org/packages/System.Text.Json) 5.x in an ASP.NET Core 3.x app, the project requires a package reference for [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web).</span></span> <span data-ttu-id="db903-329">Il problema sottostante verrà risolto in una versione futura della patch di .NET 5.</span><span class="sxs-lookup"><span data-stu-id="db903-329">The underlying issue will be resolved in a future patch release of .NET 5.</span></span> <span data-ttu-id="db903-330">Per altre informazioni, vedere [System.Text.Json definisce netcoreapp 3.0 senza dipendenze (DotNet/runtime #45560)](https://github.com/dotnet/runtime/issues/45560).</span><span class="sxs-lookup"><span data-stu-id="db903-330">For more information, see [System.Text.Json defines netcoreapp3.0 with no dependencies (dotnet/runtime #45560)](https://github.com/dotnet/runtime/issues/45560).</span></span>

<span data-ttu-id="db903-331">Per aggiungere [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web) al progetto, seguire le istruzioni per la scelta degli strumenti:</span><span class="sxs-lookup"><span data-stu-id="db903-331">To add [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web) to the project, follow the guidance for your choice of tooling:</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="db903-332">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="db903-332">Visual Studio</span></span>](#tab/visual-studio/)

1. <span data-ttu-id="db903-333">In **Esplora soluzioni** fare clic con il pulsante destro del mouse sul `BlazorServerSignalRApp` progetto e scegliere **Gestisci pacchetti NuGet**.</span><span class="sxs-lookup"><span data-stu-id="db903-333">In **Solution Explorer**, right-click the `BlazorServerSignalRApp` project and select **Manage NuGet Packages**.</span></span>

1. <span data-ttu-id="db903-334">Nella finestra di dialogo **Gestisci pacchetti NuGet** verificare che l' **origine del pacchetto** sia impostata su `nuget.org` .</span><span class="sxs-lookup"><span data-stu-id="db903-334">In the **Manage NuGet Packages** dialog, confirm that the **Package source** is set to `nuget.org`.</span></span>

1. <span data-ttu-id="db903-335">Con **Sfoglia** selezionato, digitare `System.Text.Encodings.Web` nella casella di ricerca.</span><span class="sxs-lookup"><span data-stu-id="db903-335">With **Browse** selected, type `System.Text.Encodings.Web` in the search box.</span></span>

1. <span data-ttu-id="db903-336">Nei risultati della ricerca selezionare il [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web) pacchetto.</span><span class="sxs-lookup"><span data-stu-id="db903-336">In the search results, select the [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web) package.</span></span> <span data-ttu-id="db903-337">Consente di selezionare la versione del pacchetto corrispondente al Framework condiviso in uso.</span><span class="sxs-lookup"><span data-stu-id="db903-337">Select the version of the package that matches the shared framework in use.</span></span> <span data-ttu-id="db903-338">Selezionare **Installa**.</span><span class="sxs-lookup"><span data-stu-id="db903-338">Select **Install**.</span></span>

1. <span data-ttu-id="db903-339">Se viene visualizzata la finestra di dialogo **Anteprima modifiche** , selezionare **OK**.</span><span class="sxs-lookup"><span data-stu-id="db903-339">If the **Preview Changes** dialog appears, select **OK**.</span></span>

1. <span data-ttu-id="db903-340">Se viene visualizzata la finestra di dialogo **accettazione della licenza** , selezionare Accetto se **si accettano le** condizioni di licenza.</span><span class="sxs-lookup"><span data-stu-id="db903-340">If the **License Acceptance** dialog appears, select **I Accept** if you agree with the license terms.</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="db903-341">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="db903-341">Visual Studio Code</span></span>](#tab/visual-studio-code/)

<span data-ttu-id="db903-342">Nel **terminale integrato** (**visualizzare** il > **terminale** dalla barra degli strumenti) eseguire i comandi seguenti:</span><span class="sxs-lookup"><span data-stu-id="db903-342">In the **Integrated Terminal** (**View** > **Terminal** from the toolbar), execute the following commands:</span></span>

```dotnetcli
dotnet add package System.Text.Encodings.Web
```

<span data-ttu-id="db903-343">Per aggiungere una versione precedente del pacchetto, specificare l' `--version {VERSION}` opzione, dove il `{VERSION}` segnaposto è la versione del pacchetto da aggiungere.</span><span class="sxs-lookup"><span data-stu-id="db903-343">To add an earlier version of the package, supply the `--version {VERSION}` option, where the `{VERSION}` placeholder is the version of the package to add.</span></span>

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="db903-344">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="db903-344">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

1. <span data-ttu-id="db903-345">Nella barra laterale **soluzione** fare clic con il pulsante destro del mouse sul `BlazorServerSignalRApp` progetto e scegliere **Gestisci pacchetti NuGet**.</span><span class="sxs-lookup"><span data-stu-id="db903-345">In the **Solution** sidebar, right-click the `BlazorServerSignalRApp` project and select **Manage NuGet Packages**.</span></span>

1. <span data-ttu-id="db903-346">Nella finestra di dialogo **Gestisci pacchetti NuGet** verificare che l'elenco a discesa origine sia impostato su `nuget.org` .</span><span class="sxs-lookup"><span data-stu-id="db903-346">In the **Manage NuGet Packages** dialog, confirm that the source drop-down is set to `nuget.org`.</span></span>

1. <span data-ttu-id="db903-347">Con **Sfoglia** selezionato, digitare `System.Text.Encodings.Web` nella casella di ricerca.</span><span class="sxs-lookup"><span data-stu-id="db903-347">With **Browse** selected, type `System.Text.Encodings.Web` in the search box.</span></span>

1. <span data-ttu-id="db903-348">Nei risultati della ricerca selezionare la casella di controllo accanto al [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web) pacchetto, selezionare la versione corretta del pacchetto corrispondente al Framework condiviso in uso e selezionare **Aggiungi pacchetto**.</span><span class="sxs-lookup"><span data-stu-id="db903-348">In the search results, select the check box next to the [`System.Text.Encodings.Web`](https://www.nuget.org/packages/System.Text.Encodings.Web) package, select the correct version of the package that matches the shared framework in use, and select **Add Package**.</span></span>

1. <span data-ttu-id="db903-349">Se viene visualizzata la finestra di dialogo **accettazione della licenza** , selezionare **Accetto** se si accettano le condizioni di licenza.</span><span class="sxs-lookup"><span data-stu-id="db903-349">If the **License Acceptance** dialog appears, select **Accept** if you agree with the license terms.</span></span>

# <a name="net-core-cli"></a>[<span data-ttu-id="db903-350">Interfaccia della riga di comando di .NET Core</span><span class="sxs-lookup"><span data-stu-id="db903-350">.NET Core CLI</span></span>](#tab/netcore-cli/)

<span data-ttu-id="db903-351">In una shell dei comandi dalla cartella del progetto, eseguire il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="db903-351">In a command shell from the project's folder, execute the following command:</span></span>

```dotnetcli
dotnet add package System.Text.Encodings.Web
```

<span data-ttu-id="db903-352">Per aggiungere una versione precedente del pacchetto, specificare l' `--version {VERSION}` opzione, dove il `{VERSION}` segnaposto è la versione del pacchetto da aggiungere.</span><span class="sxs-lookup"><span data-stu-id="db903-352">To add an earlier version of the package, supply the `--version {VERSION}` option, where the `{VERSION}` placeholder is the version of the package to add.</span></span>

---

::: moniker-end

## <a name="add-a-signalr-hub"></a><span data-ttu-id="db903-353">Aggiungere un SignalR Hub</span><span class="sxs-lookup"><span data-stu-id="db903-353">Add a SignalR hub</span></span>

<span data-ttu-id="db903-354">Creare una `Hubs` cartella (plurale) e aggiungere la `ChatHub` classe seguente ( `Hubs/ChatHub.cs` ):</span><span class="sxs-lookup"><span data-stu-id="db903-354">Create a `Hubs` (plural) folder and add the following `ChatHub` class (`Hubs/ChatHub.cs`):</span></span>

::: moniker range=">= aspnetcore-5.0"

[!code-csharp[](signalr-blazor/samples/5.x/BlazorServerSignalRApp/Hubs/ChatHub.cs)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-csharp[](signalr-blazor/samples/3.x/BlazorServerSignalRApp/Hubs/ChatHub.cs)]

::: moniker-end

## <a name="add-services-and-an-endpoint-for-the-signalr-hub"></a><span data-ttu-id="db903-355">Aggiungere servizi e un endpoint per l' SignalR Hub</span><span class="sxs-lookup"><span data-stu-id="db903-355">Add services and an endpoint for the SignalR hub</span></span>

1. <span data-ttu-id="db903-356">Aprire il file `Startup.cs`.</span><span class="sxs-lookup"><span data-stu-id="db903-356">Open the `Startup.cs` file.</span></span>

1. <span data-ttu-id="db903-357">Aggiungere gli spazi dei nomi per <xref:Microsoft.AspNetCore.ResponseCompression?displayProperty=fullName> e la `ChatHub` classe all'inizio del file:</span><span class="sxs-lookup"><span data-stu-id="db903-357">Add the namespaces for <xref:Microsoft.AspNetCore.ResponseCompression?displayProperty=fullName> and the `ChatHub` class to the top of the file:</span></span>

   ```csharp
   using Microsoft.AspNetCore.ResponseCompression;
   using BlazorServerSignalRApp.Server.Hubs;
   ```

::: moniker range=">= aspnetcore-5.0"

1. <span data-ttu-id="db903-358">Aggiungere i servizi middleware della compressione della risposta a `Startup.ConfigureServices` :</span><span class="sxs-lookup"><span data-stu-id="db903-358">Add Response Compression Middleware services to `Startup.ConfigureServices`:</span></span>

   [!code-csharp[](signalr-blazor/samples/5.x/BlazorServerSignalRApp/Startup.cs?name=snippet_ConfigureServices&highlight=6-10)]

1. <span data-ttu-id="db903-359">In `Startup.Configure`:</span><span class="sxs-lookup"><span data-stu-id="db903-359">In `Startup.Configure`:</span></span>

   * <span data-ttu-id="db903-360">Usare il middleware della compressione della risposta nella parte superiore della configurazione della pipeline di elaborazione.</span><span class="sxs-lookup"><span data-stu-id="db903-360">Use Response Compression Middleware at the top of the processing pipeline's configuration.</span></span>
   * <span data-ttu-id="db903-361">Tra gli endpoint per il mapping dell' Blazor Hub e del fallback lato client, aggiungere un endpoint per l'hub.</span><span class="sxs-lookup"><span data-stu-id="db903-361">Between the endpoints for mapping the Blazor hub and the client-side fallback, add an endpoint for the hub.</span></span>

   [!code-csharp[](signalr-blazor/samples/5.x/BlazorServerSignalRApp/Startup.cs?name=snippet_Configure&highlight=3,23)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

1. <span data-ttu-id="db903-362">Aggiungere i servizi middleware della compressione della risposta a `Startup.ConfigureServices` :</span><span class="sxs-lookup"><span data-stu-id="db903-362">Add Response Compression Middleware services to `Startup.ConfigureServices`:</span></span>

   [!code-csharp[](signalr-blazor/samples/3.x/BlazorServerSignalRApp/Startup.cs?name=snippet_ConfigureServices&highlight=6-10)]

1. <span data-ttu-id="db903-363">In `Startup.Configure`:</span><span class="sxs-lookup"><span data-stu-id="db903-363">In `Startup.Configure`:</span></span>

   * <span data-ttu-id="db903-364">Usare il middleware della compressione della risposta nella parte superiore della configurazione della pipeline di elaborazione.</span><span class="sxs-lookup"><span data-stu-id="db903-364">Use Response Compression Middleware at the top of the processing pipeline's configuration.</span></span>
   * <span data-ttu-id="db903-365">Tra gli endpoint per il mapping dell' Blazor Hub e del fallback lato client, aggiungere un endpoint per l'hub.</span><span class="sxs-lookup"><span data-stu-id="db903-365">Between the endpoints for mapping the Blazor hub and the client-side fallback, add an endpoint for the hub.</span></span>

   [!code-csharp[](signalr-blazor/samples/3.x/BlazorServerSignalRApp/Startup.cs?name=snippet_Configure&highlight=3,23)]

::: moniker-end

## <a name="add-razor-component-code-for-chat"></a><span data-ttu-id="db903-366">Aggiungi Razor codice componente per la chat</span><span class="sxs-lookup"><span data-stu-id="db903-366">Add Razor component code for chat</span></span>

1. <span data-ttu-id="db903-367">Aprire il file `Pages/Index.razor`.</span><span class="sxs-lookup"><span data-stu-id="db903-367">Open the `Pages/Index.razor` file.</span></span>

::: moniker range=">= aspnetcore-5.0"

1. <span data-ttu-id="db903-368">Sostituire il markup con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="db903-368">Replace the markup with the following code:</span></span>

   [!code-razor[](signalr-blazor/samples/5.x/BlazorServerSignalRApp/Pages/Index.razor)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

1. <span data-ttu-id="db903-369">Sostituire il markup con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="db903-369">Replace the markup with the following code:</span></span>

   [!code-razor[](signalr-blazor/samples/3.x/BlazorServerSignalRApp/Pages/Index.razor)]

::: moniker-end

## <a name="run-the-app"></a><span data-ttu-id="db903-370">Eseguire l'app</span><span class="sxs-lookup"><span data-stu-id="db903-370">Run the app</span></span>

<span data-ttu-id="db903-371">Seguire le istruzioni per gli strumenti:</span><span class="sxs-lookup"><span data-stu-id="db903-371">Follow the guidance for your tooling:</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="db903-372">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="db903-372">Visual Studio</span></span>](#tab/visual-studio)

1. <span data-ttu-id="db903-373">Premere <kbd>F5</kbd> per eseguire l'app con il debug o <kbd>CTRL</kbd> + <kbd>F5</kbd> per eseguire l'app senza debug.</span><span class="sxs-lookup"><span data-stu-id="db903-373">Press <kbd>F5</kbd> to run the app with debugging or <kbd>Ctrl</kbd>+<kbd>F5</kbd> to run the app without debugging.</span></span>

1. <span data-ttu-id="db903-374">Copiare l'URL dalla barra degli indirizzi, aprire un'altra istanza o scheda del browser e incollare l'URL nella barra degli indirizzi.</span><span class="sxs-lookup"><span data-stu-id="db903-374">Copy the URL from the address bar, open another browser instance or tab, and paste the URL in the address bar.</span></span>

1. <span data-ttu-id="db903-375">Scegliere browser, immettere un nome e un messaggio e selezionare il pulsante per l'invio del messaggio.</span><span class="sxs-lookup"><span data-stu-id="db903-375">Choose either browser, enter a name and message, and select the button to send the message.</span></span> <span data-ttu-id="db903-376">Il nome e il messaggio vengono visualizzati immediatamente in entrambe le pagine:</span><span class="sxs-lookup"><span data-stu-id="db903-376">The name and message are displayed on both pages instantly:</span></span>

   ![::: NO-LOC (SignalR):::::: NO-LOC (Blazer)::: app di esempio aperta in due finestre del browser che mostrano i messaggi scambiati.](signalr-blazor/_static/signalr-blazor-finished.png)

   <span data-ttu-id="db903-378">Virgolette: *Star Trek VI: il paese non individuato* &copy; 1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)</span><span class="sxs-lookup"><span data-stu-id="db903-378">Quotes: *Star Trek VI: The Undiscovered Country* &copy;1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="db903-379">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="db903-379">Visual Studio Code</span></span>](#tab/visual-studio-code)

1. <span data-ttu-id="db903-380">Premere <kbd>F5</kbd> per eseguire l'app con il debug o <kbd>CTRL</kbd> + <kbd>F5</kbd> per eseguire l'app senza debug.</span><span class="sxs-lookup"><span data-stu-id="db903-380">Press <kbd>F5</kbd> to run the app with debugging or <kbd>Ctrl</kbd>+<kbd>F5</kbd> to run the app without debugging.</span></span>

1. <span data-ttu-id="db903-381">Copiare l'URL dalla barra degli indirizzi, aprire un'altra istanza o scheda del browser e incollare l'URL nella barra degli indirizzi.</span><span class="sxs-lookup"><span data-stu-id="db903-381">Copy the URL from the address bar, open another browser instance or tab, and paste the URL in the address bar.</span></span>

1. <span data-ttu-id="db903-382">Scegliere browser, immettere un nome e un messaggio e selezionare il pulsante per l'invio del messaggio.</span><span class="sxs-lookup"><span data-stu-id="db903-382">Choose either browser, enter a name and message, and select the button to send the message.</span></span> <span data-ttu-id="db903-383">Il nome e il messaggio vengono visualizzati immediatamente in entrambe le pagine:</span><span class="sxs-lookup"><span data-stu-id="db903-383">The name and message are displayed on both pages instantly:</span></span>

   ![::: NO-LOC (SignalR):::::: NO-LOC (Blazer)::: app di esempio aperta in due finestre del browser che mostrano i messaggi scambiati.](signalr-blazor/_static/signalr-blazor-finished.png)

   <span data-ttu-id="db903-385">Virgolette: *Star Trek VI: il paese non individuato* &copy; 1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)</span><span class="sxs-lookup"><span data-stu-id="db903-385">Quotes: *Star Trek VI: The Undiscovered Country* &copy;1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)</span></span>

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="db903-386">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="db903-386">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

1. <span data-ttu-id="db903-387">Premere <kbd>⌘</kbd> + <kbd>↩</kbd> per eseguire l'app con debug o <kbd>⌥</kbd> + <kbd>⌘</kbd> + <kbd>↩</kbd> per eseguire l'app senza debug.</span><span class="sxs-lookup"><span data-stu-id="db903-387">Press <kbd>⌘</kbd>+<kbd>↩</kbd> to run the app with debugging or <kbd>⌥</kbd>+<kbd>⌘</kbd>+<kbd>↩</kbd> to run the app without debugging.</span></span>

1. <span data-ttu-id="db903-388">Copiare l'URL dalla barra degli indirizzi, aprire un'altra istanza o scheda del browser e incollare l'URL nella barra degli indirizzi.</span><span class="sxs-lookup"><span data-stu-id="db903-388">Copy the URL from the address bar, open another browser instance or tab, and paste the URL in the address bar.</span></span>

1. <span data-ttu-id="db903-389">Scegliere browser, immettere un nome e un messaggio e selezionare il pulsante per l'invio del messaggio.</span><span class="sxs-lookup"><span data-stu-id="db903-389">Choose either browser, enter a name and message, and select the button to send the message.</span></span> <span data-ttu-id="db903-390">Il nome e il messaggio vengono visualizzati immediatamente in entrambe le pagine:</span><span class="sxs-lookup"><span data-stu-id="db903-390">The name and message are displayed on both pages instantly:</span></span>

   ![::: NO-LOC (SignalR):::::: NO-LOC (Blazer)::: app di esempio aperta in due finestre del browser che mostrano i messaggi scambiati.](signalr-blazor/_static/signalr-blazor-finished.png)

   <span data-ttu-id="db903-392">Virgolette: *Star Trek VI: il paese non individuato* &copy; 1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)</span><span class="sxs-lookup"><span data-stu-id="db903-392">Quotes: *Star Trek VI: The Undiscovered Country* &copy;1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)</span></span>

# <a name="net-core-cli"></a>[<span data-ttu-id="db903-393">Interfaccia della riga di comando di .NET Core</span><span class="sxs-lookup"><span data-stu-id="db903-393">.NET Core CLI</span></span>](#tab/netcore-cli/)

1. <span data-ttu-id="db903-394">In una shell dei comandi dalla cartella del progetto, eseguire i comandi seguenti:</span><span class="sxs-lookup"><span data-stu-id="db903-394">In a command shell from the project's folder, execute the following commands:</span></span>

   ```dotnetcli
   dotnet run
   ```

1. <span data-ttu-id="db903-395">Copiare l'URL dalla barra degli indirizzi, aprire un'altra istanza o scheda del browser e incollare l'URL nella barra degli indirizzi.</span><span class="sxs-lookup"><span data-stu-id="db903-395">Copy the URL from the address bar, open another browser instance or tab, and paste the URL in the address bar.</span></span>

1. <span data-ttu-id="db903-396">Scegliere browser, immettere un nome e un messaggio e selezionare il pulsante per l'invio del messaggio.</span><span class="sxs-lookup"><span data-stu-id="db903-396">Choose either browser, enter a name and message, and select the button to send the message.</span></span> <span data-ttu-id="db903-397">Il nome e il messaggio vengono visualizzati immediatamente in entrambe le pagine:</span><span class="sxs-lookup"><span data-stu-id="db903-397">The name and message are displayed on both pages instantly:</span></span>

   ![::: NO-LOC (SignalR):::::: NO-LOC (Blazer)::: app di esempio aperta in due finestre del browser che mostrano i messaggi scambiati.](signalr-blazor/_static/signalr-blazor-finished.png)

   <span data-ttu-id="db903-399">Virgolette: *Star Trek VI: il paese non individuato* &copy; 1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)</span><span class="sxs-lookup"><span data-stu-id="db903-399">Quotes: *Star Trek VI: The Undiscovered Country* &copy;1991 [Paramount](https://www.paramountmovies.com/movies/star-trek-vi-the-undiscovered-country)</span></span>

---

::: zone-end

## <a name="next-steps"></a><span data-ttu-id="db903-400">Passaggi successivi</span><span class="sxs-lookup"><span data-stu-id="db903-400">Next steps</span></span>

<span data-ttu-id="db903-401">In questa esercitazione sono state illustrate le procedure per:</span><span class="sxs-lookup"><span data-stu-id="db903-401">In this tutorial, you learned how to:</span></span>

> [!div class="checklist"]
> * <span data-ttu-id="db903-402">Creazione di un Blazor progetto</span><span class="sxs-lookup"><span data-stu-id="db903-402">Create a Blazor project</span></span>
> * <span data-ttu-id="db903-403">Aggiungere la SignalR libreria client</span><span class="sxs-lookup"><span data-stu-id="db903-403">Add the SignalR client library</span></span>
> * <span data-ttu-id="db903-404">Aggiungere un SignalR Hub</span><span class="sxs-lookup"><span data-stu-id="db903-404">Add a SignalR hub</span></span>
> * <span data-ttu-id="db903-405">Aggiungere SignalR Servizi e un endpoint per l' SignalR Hub</span><span class="sxs-lookup"><span data-stu-id="db903-405">Add SignalR services and an endpoint for the SignalR hub</span></span>
> * <span data-ttu-id="db903-406">Aggiungi Razor codice componente per la chat</span><span class="sxs-lookup"><span data-stu-id="db903-406">Add Razor component code for chat</span></span>

<span data-ttu-id="db903-407">Per altre informazioni sulla creazione di Blazor app, vedere la Blazor documentazione:</span><span class="sxs-lookup"><span data-stu-id="db903-407">To learn more about building Blazor apps, see the Blazor documentation:</span></span>

> [!div class="nextstepaction"]
> <span data-ttu-id="db903-408"><xref:blazor/index>
> [Autenticazione del token di porta con Identity Server, WebSocket ed eventi di Server-Sent](xref:signalr/authn-and-authz#bearer-token-authentication)</span><span class="sxs-lookup"><span data-stu-id="db903-408"><xref:blazor/index>
[Bearer token authentication with Identity Server, WebSockets, and Server-Sent Events](xref:signalr/authn-and-authz#bearer-token-authentication)</span></span>

## <a name="additional-resources"></a><span data-ttu-id="db903-409">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="db903-409">Additional resources</span></span>

* <xref:signalr/introduction>
* [<span data-ttu-id="db903-410">SignalR negoziazione tra le origini per l'autenticazione</span><span class="sxs-lookup"><span data-stu-id="db903-410">SignalR cross-origin negotiation for authentication</span></span>](xref:blazor/fundamentals/signalr#signalr-cross-origin-negotiation-for-authentication)
* <xref:blazor/debug>
* <xref:blazor/security/server/threat-mitigation>
