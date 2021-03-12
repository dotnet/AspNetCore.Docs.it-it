---
title: Creare un client e un server gRPC .NET Core in ASP.NET Core
author: juntaoluo
description: Questa esercitazione illustra come creare un servizio gRPC e un client gRPC in ASP.NET Core. Informazioni su come creare un progetto di servizio gRPC, modificare un file con estensione proto e aggiungere una chiamata con flusso bidirezionale.
ms.author: johluo
ms.date: 10/23/2020
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
uid: tutorials/grpc/grpc-start
ms.openlocfilehash: fed25eeacf57504810d41fcf002dcaa9927a21af
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102588601"
---
# <a name="tutorial-create-a-grpc-client-and-server-in-aspnet-core"></a><span data-ttu-id="57077-104">Esercitazione: creare un client e un server gRPC in ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="57077-104">Tutorial: Create a gRPC client and server in ASP.NET Core</span></span>

<span data-ttu-id="57077-105">A cura di [John Luo](https://github.com/juntaoluo)</span><span class="sxs-lookup"><span data-stu-id="57077-105">By [John Luo](https://github.com/juntaoluo)</span></span>

<span data-ttu-id="57077-106">Questa esercitazione illustra come creare un client [gRPC](https://grpc.io/docs/guides/) .NET Core e un server gRPC ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="57077-106">This tutorial shows how to create a .NET Core [gRPC](https://grpc.io/docs/guides/) client and an ASP.NET Core gRPC Server.</span></span>

<span data-ttu-id="57077-107">Al termine, si otterrà un client gRPC che comunica con il servizio Greeter gRPC.</span><span class="sxs-lookup"><span data-stu-id="57077-107">At the end, you'll have a gRPC client that communicates with the gRPC Greeter service.</span></span>

<span data-ttu-id="57077-108">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/grpc/grpc-start/sample) ([procedura per il download](xref:index#how-to-download-a-sample)).</span><span class="sxs-lookup"><span data-stu-id="57077-108">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/grpc/grpc-start/sample) ([how to download](xref:index#how-to-download-a-sample)).</span></span>

<span data-ttu-id="57077-109">In questa esercitazione:</span><span class="sxs-lookup"><span data-stu-id="57077-109">In this tutorial, you:</span></span>

> [!div class="checklist"]
> * <span data-ttu-id="57077-110">Creare un server gRPC.</span><span class="sxs-lookup"><span data-stu-id="57077-110">Create a gRPC Server.</span></span>
> * <span data-ttu-id="57077-111">Creare un client gRPC.</span><span class="sxs-lookup"><span data-stu-id="57077-111">Create a gRPC client.</span></span>
> * <span data-ttu-id="57077-112">Testare il servizio client gRPC con il servizio Greeter gRPC.</span><span class="sxs-lookup"><span data-stu-id="57077-112">Test the gRPC client service with the gRPC Greeter service.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="57077-113">Prerequisiti</span><span class="sxs-lookup"><span data-stu-id="57077-113">Prerequisites</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="57077-114">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="57077-114">Visual Studio</span></span>](#tab/visual-studio)

[!INCLUDE[](~/includes/net-core-prereqs-vs-3.1.md)]

# <a name="visual-studio-code"></a>[<span data-ttu-id="57077-115">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="57077-115">Visual Studio Code</span></span>](#tab/visual-studio-code)

[!INCLUDE[](~/includes/net-core-prereqs-vsc-3.1.md)]

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="57077-116">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="57077-116">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

* [<span data-ttu-id="57077-117">Visual Studio per Mac versione 8,7 o successiva</span><span class="sxs-lookup"><span data-stu-id="57077-117">Visual Studio for Mac version 8.7 or later</span></span>](/visualstudio/releasenotes/vs2019-mac-relnotes)
* [!INCLUDE [.NET Core 3.1 SDK](~/includes/3.1-SDK.md)]
---

## <a name="create-a-grpc-service"></a><span data-ttu-id="57077-118">Creare un servizio gRPC</span><span class="sxs-lookup"><span data-stu-id="57077-118">Create a gRPC service</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="57077-119">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="57077-119">Visual Studio</span></span>](#tab/visual-studio)

* <span data-ttu-id="57077-120">Avviare Visual Studio e selezionare **Crea un nuovo progetto**.</span><span class="sxs-lookup"><span data-stu-id="57077-120">Start Visual Studio and select **Create a new project**.</span></span> <span data-ttu-id="57077-121">In alternativa, dal menu **File** di Visual Studio scegliere **Nuovo** > **Progetto**.</span><span class="sxs-lookup"><span data-stu-id="57077-121">Alternatively, from the Visual Studio **File** menu, select **New** > **Project**.</span></span>
* <span data-ttu-id="57077-122">Nella finestra di dialogo **Crea un nuovo progetto** selezionare **servizio GRPC** e selezionare **Avanti**:</span><span class="sxs-lookup"><span data-stu-id="57077-122">In the **Create a new project** dialog, select **gRPC Service** and select **Next**:</span></span>

  ![Crea una finestra di dialogo nuovo progetto in Visual Studio](~/tutorials/grpc/grpc-start/static/cnp.png)

* <span data-ttu-id="57077-124">Denominare il progetto **GrpcGreeter**.</span><span class="sxs-lookup"><span data-stu-id="57077-124">Name the project **GrpcGreeter**.</span></span> <span data-ttu-id="57077-125">È importante denominare il progetto *GrpcGreeter* in modo che gli spazi dei nomi corrispondano quando si copia e incolla il codice.</span><span class="sxs-lookup"><span data-stu-id="57077-125">It's important to name the project *GrpcGreeter* so the namespaces match when you copy and paste code.</span></span>
* <span data-ttu-id="57077-126">Selezionare **Crea**.</span><span class="sxs-lookup"><span data-stu-id="57077-126">Select **Create**.</span></span>
* <span data-ttu-id="57077-127">Nella finestra di dialogo **Crea un nuovo servizio gRPC**:</span><span class="sxs-lookup"><span data-stu-id="57077-127">In the **Create a new gRPC service** dialog:</span></span>
  * <span data-ttu-id="57077-128">È selezionato il modello **Servizio gRPC**.</span><span class="sxs-lookup"><span data-stu-id="57077-128">The **gRPC Service** template is selected.</span></span>
  * <span data-ttu-id="57077-129">Selezionare **Crea**.</span><span class="sxs-lookup"><span data-stu-id="57077-129">Select **Create**.</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="57077-130">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="57077-130">Visual Studio Code</span></span>](#tab/visual-studio-code)

* <span data-ttu-id="57077-131">Aprire il [terminale integrato](https://code.visualstudio.com/docs/editor/integrated-terminal).</span><span class="sxs-lookup"><span data-stu-id="57077-131">Open the [integrated terminal](https://code.visualstudio.com/docs/editor/integrated-terminal).</span></span>
* <span data-ttu-id="57077-132">Modificare le directory ( `cd` ) in una cartella per il progetto.</span><span class="sxs-lookup"><span data-stu-id="57077-132">Change directories (`cd`) to a folder for the project.</span></span>
* <span data-ttu-id="57077-133">Eseguire i comandi seguenti:</span><span class="sxs-lookup"><span data-stu-id="57077-133">Run the following commands:</span></span>

  ```dotnetcli
  dotnet new grpc -o GrpcGreeter
  code -r GrpcGreeter
  ```

  * <span data-ttu-id="57077-134">Il comando `dotnet new` crea un nuovo servizio gRPC nella cartella *GrpcGreeter*.</span><span class="sxs-lookup"><span data-stu-id="57077-134">The `dotnet new` command creates a new gRPC service in the *GrpcGreeter* folder.</span></span>
  * <span data-ttu-id="57077-135">Il comando `code` apre la cartella *GrpcGreeter* in una nuova istanza di Visual Studio Code.</span><span class="sxs-lookup"><span data-stu-id="57077-135">The `code` command opens the *GrpcGreeter* folder in a new instance of Visual Studio Code.</span></span>

  <span data-ttu-id="57077-136">Viene visualizzata una finestra di dialogo con le **risorse necessarie per la compilazione e il debug non è presente in ' GrpcGreeter '. Aggiungerli?**</span><span class="sxs-lookup"><span data-stu-id="57077-136">A dialog box appears with **Required assets to build and debug are missing from 'GrpcGreeter'. Add them?**</span></span>
* <span data-ttu-id="57077-137">Selezionare **Sì**.</span><span class="sxs-lookup"><span data-stu-id="57077-137">Select **Yes**.</span></span>

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="57077-138">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="57077-138">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

* <span data-ttu-id="57077-139">Avviare Visual Studio per Mac e selezionare **Crea un nuovo progetto**.</span><span class="sxs-lookup"><span data-stu-id="57077-139">Start Visual Studio for Mac and select **Create a new project**.</span></span> <span data-ttu-id="57077-140">In alternativa, dal menu **File** di Visual Studio scegliere **Nuovo** > **Progetto**.</span><span class="sxs-lookup"><span data-stu-id="57077-140">Alternatively, from the Visual Studio **File** menu, select **New** > **Project**.</span></span>
* <span data-ttu-id="57077-141">Nella finestra di dialogo **Crea un nuovo progetto** selezionare **Web e**  >  **app** console  >  **servizio gRPC** e selezionare **Avanti**:</span><span class="sxs-lookup"><span data-stu-id="57077-141">In the **Create a new project** dialog, select **Web and Console** > **App** > **gRPC Service** and select **Next**:</span></span>

  ![Crea una finestra di dialogo nuovo progetto in macOS](~/tutorials/grpc/grpc-start/static/cnp-mac.png)

* <span data-ttu-id="57077-143">Selezionare **.NET Core 3,1** per il Framework di destinazione e selezionare **Avanti**.</span><span class="sxs-lookup"><span data-stu-id="57077-143">Select **.NET Core 3.1** for the target framework and select **Next**.</span></span>
* <span data-ttu-id="57077-144">Denominare il progetto **GrpcGreeter**.</span><span class="sxs-lookup"><span data-stu-id="57077-144">Name the project **GrpcGreeter**.</span></span> <span data-ttu-id="57077-145">È importante denominare il progetto *GrpcGreeter* in modo che gli spazi dei nomi corrispondano quando si copia e incolla il codice.</span><span class="sxs-lookup"><span data-stu-id="57077-145">It's important to name the project *GrpcGreeter* so the namespaces match when you copy and paste code.</span></span>
* <span data-ttu-id="57077-146">Selezionare **Crea**.</span><span class="sxs-lookup"><span data-stu-id="57077-146">Select **Create**.</span></span>
---

### <a name="run-the-service"></a><span data-ttu-id="57077-147">Eseguire il servizio</span><span class="sxs-lookup"><span data-stu-id="57077-147">Run the service</span></span>

  [!INCLUDE[](~/includes/run-the-app.md)]

<span data-ttu-id="57077-148">I log indicano che il servizio è in ascolto su `https://localhost:5001`.</span><span class="sxs-lookup"><span data-stu-id="57077-148">The logs show the service listening on `https://localhost:5001`.</span></span>

```console
info: Microsoft.Hosting.Lifetime[0]
      Now listening on: https://localhost:5001
info: Microsoft.Hosting.Lifetime[0]
      Application started. Press Ctrl+C to shut down.
info: Microsoft.Hosting.Lifetime[0]
      Hosting environment: Development
```

> [!NOTE]
> <span data-ttu-id="57077-149">Il modello gRPC è configurato per l'uso di [Transport Layer Security (TLS)](https://tools.ietf.org/html/rfc5246).</span><span class="sxs-lookup"><span data-stu-id="57077-149">The gRPC template is configured to use [Transport Layer Security (TLS)](https://tools.ietf.org/html/rfc5246).</span></span> <span data-ttu-id="57077-150">i client gRPC devono usare HTTPS per chiamare il server.</span><span class="sxs-lookup"><span data-stu-id="57077-150">gRPC clients need to use HTTPS to call the server.</span></span>
>
> <span data-ttu-id="57077-151">macOS non supporta ASP.NET Core gRPC con TLS.</span><span class="sxs-lookup"><span data-stu-id="57077-151">macOS doesn't support ASP.NET Core gRPC with TLS.</span></span> <span data-ttu-id="57077-152">Per eseguire correttamente i servizi gRPC in macOS, è necessaria una configurazione aggiuntiva.</span><span class="sxs-lookup"><span data-stu-id="57077-152">Additional configuration is required to successfully run gRPC services on macOS.</span></span> <span data-ttu-id="57077-153">Per altre informazioni, vedere [Non è possibile avviare un'app ASP.NET Core gRPC in macOS](xref:grpc/troubleshoot#unable-to-start-aspnet-core-grpc-app-on-macos).</span><span class="sxs-lookup"><span data-stu-id="57077-153">For more information, see [Unable to start ASP.NET Core gRPC app on macOS](xref:grpc/troubleshoot#unable-to-start-aspnet-core-grpc-app-on-macos).</span></span>

### <a name="examine-the-project-files"></a><span data-ttu-id="57077-154">Esaminare i file di progetto</span><span class="sxs-lookup"><span data-stu-id="57077-154">Examine the project files</span></span>

<span data-ttu-id="57077-155">File di progetto *GrpcGreeter*:</span><span class="sxs-lookup"><span data-stu-id="57077-155">*GrpcGreeter* project files:</span></span>

* <span data-ttu-id="57077-156">*greet. proto*: il file *Protos/greet. proto* definisce il `Greeter` gRPC e viene usato per generare gli asset del server gRPC.</span><span class="sxs-lookup"><span data-stu-id="57077-156">*greet.proto*: The *Protos/greet.proto* file defines the `Greeter` gRPC and is used to generate the gRPC server assets.</span></span> <span data-ttu-id="57077-157">Per altre informazioni, vedere [Introduzione a gRPC](xref:grpc/index).</span><span class="sxs-lookup"><span data-stu-id="57077-157">For more information, see [Introduction to gRPC](xref:grpc/index).</span></span>
* <span data-ttu-id="57077-158">Cartella dei *Servizi* : contiene l'implementazione del `Greeter` servizio.</span><span class="sxs-lookup"><span data-stu-id="57077-158">*Services* folder: Contains the implementation of the `Greeter` service.</span></span>
* <span data-ttu-id="57077-159">*appSettings.json*: contiene i dati di configurazione, ad esempio il protocollo usato da gheppio.</span><span class="sxs-lookup"><span data-stu-id="57077-159">*appSettings.json*: Contains configuration data, such as protocol used by Kestrel.</span></span> <span data-ttu-id="57077-160">Per altre informazioni, vedere <xref:fundamentals/configuration/index>.</span><span class="sxs-lookup"><span data-stu-id="57077-160">For more information, see <xref:fundamentals/configuration/index>.</span></span>
* <span data-ttu-id="57077-161">*Program.cs*: contiene il punto di ingresso per il servizio gRPC.</span><span class="sxs-lookup"><span data-stu-id="57077-161">*Program.cs*: Contains the entry point for the gRPC service.</span></span> <span data-ttu-id="57077-162">Per altre informazioni, vedere <xref:fundamentals/host/generic-host>.</span><span class="sxs-lookup"><span data-stu-id="57077-162">For more information, see <xref:fundamentals/host/generic-host>.</span></span>
* <span data-ttu-id="57077-163">*Startup.cs*: contiene il codice che configura il comportamento dell'app.</span><span class="sxs-lookup"><span data-stu-id="57077-163">*Startup.cs*: Contains code that configures app behavior.</span></span> <span data-ttu-id="57077-164">Per altre informazioni, vedere [Avvio dell'app](xref:fundamentals/startup).</span><span class="sxs-lookup"><span data-stu-id="57077-164">For more information, see [App startup](xref:fundamentals/startup).</span></span>

## <a name="create-the-grpc-client-in-a-net-console-app"></a><span data-ttu-id="57077-165">Creare il client gRPC in un'app console .NET</span><span class="sxs-lookup"><span data-stu-id="57077-165">Create the gRPC client in a .NET console app</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="57077-166">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="57077-166">Visual Studio</span></span>](#tab/visual-studio)

* <span data-ttu-id="57077-167">Aprire una seconda istanza di Visual Studio e selezionare **Crea un nuovo progetto**.</span><span class="sxs-lookup"><span data-stu-id="57077-167">Open a second instance of Visual Studio and select **Create a new project**.</span></span>
* <span data-ttu-id="57077-168">Nella finestra di dialogo **Crea un nuovo progetto** selezionare **App console (.NET Core)** e selezionare **Avanti**.</span><span class="sxs-lookup"><span data-stu-id="57077-168">In the **Create a new project** dialog, select **Console App (.NET Core)** and select **Next**.</span></span>
* <span data-ttu-id="57077-169">Nella casella di testo **nome progetto** immettere **GrpcGreeterClient** e selezionare **Crea**.</span><span class="sxs-lookup"><span data-stu-id="57077-169">In the **Project name** text box, enter **GrpcGreeterClient** and select **Create**.</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="57077-170">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="57077-170">Visual Studio Code</span></span>](#tab/visual-studio-code)

* <span data-ttu-id="57077-171">Aprire il [terminale integrato](https://code.visualstudio.com/docs/editor/integrated-terminal).</span><span class="sxs-lookup"><span data-stu-id="57077-171">Open the [integrated terminal](https://code.visualstudio.com/docs/editor/integrated-terminal).</span></span>
* <span data-ttu-id="57077-172">Modificare le directory ( `cd` ) in una cartella per il progetto.</span><span class="sxs-lookup"><span data-stu-id="57077-172">Change directories (`cd`) to a folder for the project.</span></span>
* <span data-ttu-id="57077-173">Eseguire i comandi seguenti:</span><span class="sxs-lookup"><span data-stu-id="57077-173">Run the following commands:</span></span>

  ```dotnetcli
  dotnet new console -o GrpcGreeterClient
  code -r GrpcGreeterClient
  ```

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="57077-174">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="57077-174">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

<span data-ttu-id="57077-175">Seguire le istruzioni riportate in [Creazione di una soluzione .NET Core completa in macOS con Visual Studio per Mac](/dotnet/core/tutorials/using-on-mac-vs-full-solution) per creare un'app console con il nome *GrpcGreeterClient*.</span><span class="sxs-lookup"><span data-stu-id="57077-175">Follow the instructions in [Building a complete .NET Core solution on macOS using Visual Studio for Mac](/dotnet/core/tutorials/using-on-mac-vs-full-solution) to create a console app with the name *GrpcGreeterClient*.</span></span>

---

### <a name="add-required-packages"></a><span data-ttu-id="57077-176">Aggiungere i pacchetti necessari</span><span class="sxs-lookup"><span data-stu-id="57077-176">Add required packages</span></span>

<span data-ttu-id="57077-177">Per il progetto client gRPC sono richiesti i pacchetti seguenti:</span><span class="sxs-lookup"><span data-stu-id="57077-177">The gRPC client project requires the following packages:</span></span>

* <span data-ttu-id="57077-178">[Grpc.Net.Client](https://www.nuget.org/packages/Grpc.Net.Client), che contiene il client .NET Core.</span><span class="sxs-lookup"><span data-stu-id="57077-178">[Grpc.Net.Client](https://www.nuget.org/packages/Grpc.Net.Client), which contains the .NET Core client.</span></span>
* <span data-ttu-id="57077-179">[Google.Protobuf](https://www.nuget.org/packages/Google.Protobuf/), che contiene le API per i messaggi protobuf per C#.</span><span class="sxs-lookup"><span data-stu-id="57077-179">[Google.Protobuf](https://www.nuget.org/packages/Google.Protobuf/), which contains protobuf message APIs for C#.</span></span>
* <span data-ttu-id="57077-180">[Grpc.Tools](https://www.nuget.org/packages/Grpc.Tools/), che contiene il supporto di strumenti C# per i file protobuf.</span><span class="sxs-lookup"><span data-stu-id="57077-180">[Grpc.Tools](https://www.nuget.org/packages/Grpc.Tools/), which contains C# tooling support for protobuf files.</span></span> <span data-ttu-id="57077-181">Il pacchetto di strumenti non è necessario in fase di esecuzione, quindi la dipendenza è contrassegnata come `PrivateAssets="All"`.</span><span class="sxs-lookup"><span data-stu-id="57077-181">The tooling package isn't required at runtime, so the dependency is marked with `PrivateAssets="All"`.</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="57077-182">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="57077-182">Visual Studio</span></span>](#tab/visual-studio)

<span data-ttu-id="57077-183">Installare i pacchetti tramite la Console di gestione pacchetti (PMC) o Gestisci pacchetti NuGet.</span><span class="sxs-lookup"><span data-stu-id="57077-183">Install the packages using either the Package Manager Console (PMC) or Manage NuGet Packages.</span></span>

#### <a name="pmc-option-to-install-packages"></a><span data-ttu-id="57077-184">Opzione con la console di Gestione pacchetti per installare i pacchetti</span><span class="sxs-lookup"><span data-stu-id="57077-184">PMC option to install packages</span></span>

* <span data-ttu-id="57077-185">In Visual Studio selezionare **strumenti**  >  **Gestione pacchetti NuGet**  >  **console di gestione pacchetti**</span><span class="sxs-lookup"><span data-stu-id="57077-185">From Visual Studio, select **Tools** > **NuGet Package Manager** > **Package Manager Console**</span></span>
* <span data-ttu-id="57077-186">Dalla finestra **Console di Gestione pacchetti** eseguire `cd GrpcGreeterClient` per passare alla cartella contenente i file *GrpcGreeterClient.csproj*.</span><span class="sxs-lookup"><span data-stu-id="57077-186">From the **Package Manager Console** window, run `cd GrpcGreeterClient` to change directories to the folder containing the *GrpcGreeterClient.csproj* files.</span></span>
* <span data-ttu-id="57077-187">Eseguire i comandi seguenti:</span><span class="sxs-lookup"><span data-stu-id="57077-187">Run the following commands:</span></span>

  ```powershell
  Install-Package Grpc.Net.Client
  Install-Package Google.Protobuf
  Install-Package Grpc.Tools
  ```

#### <a name="manage-nuget-packages-option-to-install-packages"></a><span data-ttu-id="57077-188">Opzione Gestisci pacchetti NuGet per installare i pacchetti</span><span class="sxs-lookup"><span data-stu-id="57077-188">Manage NuGet Packages option to install packages</span></span>

* <span data-ttu-id="57077-189">Fare clic con il pulsante destro del mouse sul progetto in **Esplora soluzioni**  >  **Gestisci pacchetti NuGet**.</span><span class="sxs-lookup"><span data-stu-id="57077-189">Right-click the project in **Solution Explorer** > **Manage NuGet Packages**.</span></span>
* <span data-ttu-id="57077-190">Selezionare la scheda **Sfoglia**.</span><span class="sxs-lookup"><span data-stu-id="57077-190">Select the **Browse** tab.</span></span>
* <span data-ttu-id="57077-191">Immettere **Grpc.Net.Client** nella casella di ricerca.</span><span class="sxs-lookup"><span data-stu-id="57077-191">Enter **Grpc.Net.Client** in the search box.</span></span>
* <span data-ttu-id="57077-192">Selezionare il pacchetto **Grpc.Net.Client** dalla scheda **Sfoglia** e fare clic su **Installa**.</span><span class="sxs-lookup"><span data-stu-id="57077-192">Select the **Grpc.Net.Client** package from the **Browse** tab and select **Install**.</span></span>
* <span data-ttu-id="57077-193">Ripetere la procedura per `Google.Protobuf` e `Grpc.Tools`.</span><span class="sxs-lookup"><span data-stu-id="57077-193">Repeat for `Google.Protobuf` and `Grpc.Tools`.</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="57077-194">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="57077-194">Visual Studio Code</span></span>](#tab/visual-studio-code)

<span data-ttu-id="57077-195">Eseguire i comandi seguenti dal **terminale integrato**:</span><span class="sxs-lookup"><span data-stu-id="57077-195">Run the following commands from the **Integrated Terminal**:</span></span>

```dotnetcli
dotnet add GrpcGreeterClient.csproj package Grpc.Net.Client
dotnet add GrpcGreeterClient.csproj package Google.Protobuf
dotnet add GrpcGreeterClient.csproj package Grpc.Tools
```

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="57077-196">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="57077-196">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

* <span data-ttu-id="57077-197">Fare clic con il pulsante destro del mouse su progetto **GrpcGreeterClient** nel **riquadro della soluzione** e scegliere **Gestisci pacchetti NuGet**.</span><span class="sxs-lookup"><span data-stu-id="57077-197">Right-click **GrpcGreeterClient** project in the **Solution Pad** and select **Manage NuGet Packages**.</span></span>
* <span data-ttu-id="57077-198">Immettere **Grpc.Net.Client** nella casella di ricerca.</span><span class="sxs-lookup"><span data-stu-id="57077-198">Enter **Grpc.Net.Client** in the search box.</span></span>
* <span data-ttu-id="57077-199">Selezionare il pacchetto **Grpc .NET. client** dal riquadro dei risultati e selezionare **Aggiungi pacchetto**.</span><span class="sxs-lookup"><span data-stu-id="57077-199">Select the **Grpc.Net.Client** package from the results pane and select **Add Package**.</span></span>
* <span data-ttu-id="57077-200">Selezionare il pulsante **accetta** nella finestra di dialogo **Accept License** .</span><span class="sxs-lookup"><span data-stu-id="57077-200">Select the **Accept** button on the **Accept License** dialog.</span></span>
* <span data-ttu-id="57077-201">Ripetere la procedura per `Google.Protobuf` e `Grpc.Tools`.</span><span class="sxs-lookup"><span data-stu-id="57077-201">Repeat for `Google.Protobuf` and `Grpc.Tools`.</span></span>

---

### <a name="add-greetproto"></a><span data-ttu-id="57077-202">Aggiungere greet.proto</span><span class="sxs-lookup"><span data-stu-id="57077-202">Add greet.proto</span></span>

* <span data-ttu-id="57077-203">Creare una cartella *Protos* nel progetto client gRPC.</span><span class="sxs-lookup"><span data-stu-id="57077-203">Create a *Protos* folder in the gRPC client project.</span></span>
* <span data-ttu-id="57077-204">Copiare il file *Protos\greet.proto* dal servizio Greeter gRPC al progetto client gRPC.</span><span class="sxs-lookup"><span data-stu-id="57077-204">Copy the *Protos\greet.proto* file from the gRPC Greeter service to the gRPC client project.</span></span>
* <span data-ttu-id="57077-205">Aggiornare lo spazio dei nomi all'interno del `greet.proto` file allo spazio dei nomi del progetto:</span><span class="sxs-lookup"><span data-stu-id="57077-205">Update the namespace inside the `greet.proto` file to the project's namespace:</span></span>

  ```
  option csharp_namespace = "GrpcGreeterClient";
  ```

* <span data-ttu-id="57077-206">Modificare il file di progetto *GrpcGreeterClient.csproj*:</span><span class="sxs-lookup"><span data-stu-id="57077-206">Edit the *GrpcGreeterClient.csproj* project file:</span></span>

  # <a name="visual-studio"></a>[<span data-ttu-id="57077-207">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="57077-207">Visual Studio</span></span>](#tab/visual-studio)

  <span data-ttu-id="57077-208">Fare clic con il pulsante destro del mouse sul progetto e scegliere **Modifica file di progetto**.</span><span class="sxs-lookup"><span data-stu-id="57077-208">Right-click the project and select **Edit Project File**.</span></span>

  # <a name="visual-studio-code"></a>[<span data-ttu-id="57077-209">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="57077-209">Visual Studio Code</span></span>](#tab/visual-studio-code)

  <span data-ttu-id="57077-210">Selezionare il file *GrpcGreeterClient.csproj*.</span><span class="sxs-lookup"><span data-stu-id="57077-210">Select the *GrpcGreeterClient.csproj* file.</span></span>

  # <a name="visual-studio-for-mac"></a>[<span data-ttu-id="57077-211">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="57077-211">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

  <span data-ttu-id="57077-212">Fare clic con il pulsante destro del mouse sul progetto e scegliere **Modifica file di progetto**.</span><span class="sxs-lookup"><span data-stu-id="57077-212">Right-click the project and select **Edit Project File**.</span></span>

  ---

* <span data-ttu-id="57077-213">Aggiungere un gruppo di elementi con un elemento `<Protobuf>` che fa riferimento al file *greet.proto*:</span><span class="sxs-lookup"><span data-stu-id="57077-213">Add an item group with a `<Protobuf>` element that refers to the *greet.proto* file:</span></span>

  ```xml
  <ItemGroup>
    <Protobuf Include="Protos\greet.proto" GrpcServices="Client" />
  </ItemGroup>
  ```

### <a name="create-the-greeter-client"></a><span data-ttu-id="57077-214">Creare il client Greeter</span><span class="sxs-lookup"><span data-stu-id="57077-214">Create the Greeter client</span></span>

<span data-ttu-id="57077-215">Compilare il progetto client per creare i tipi nello `GrpcGreeter` spazio dei nomi.</span><span class="sxs-lookup"><span data-stu-id="57077-215">Build the client project to create the types in the `GrpcGreeter` namespace.</span></span> <span data-ttu-id="57077-216">I tipi `GrpcGreeter` vengono generati automaticamente dal processo di compilazione.</span><span class="sxs-lookup"><span data-stu-id="57077-216">The `GrpcGreeter` types are generated automatically by the build process.</span></span>

<span data-ttu-id="57077-217">Aggiornare il file *Program.cs* del client gRPC con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="57077-217">Update the gRPC client *Program.cs* file with the following code:</span></span>

[!code-csharp[](~/tutorials/grpc/grpc-start/sample/GrpcGreeterClient/Program.cs?name=snippet2)]

<span data-ttu-id="57077-218">*Program.cs* contiene la logica e il punto di ingresso per il client gRPC.</span><span class="sxs-lookup"><span data-stu-id="57077-218">*Program.cs* contains the entry point and logic for the gRPC client.</span></span>

<span data-ttu-id="57077-219">Il client Greeter viene creato come descritto di seguito:</span><span class="sxs-lookup"><span data-stu-id="57077-219">The Greeter client is created by:</span></span>

* <span data-ttu-id="57077-220">Creare un'istanza di `GrpcChannel` contenente le informazioni per creare la connessione al servizio gRPC.</span><span class="sxs-lookup"><span data-stu-id="57077-220">Instantiating a `GrpcChannel` containing the information for creating the connection to the gRPC service.</span></span>
* <span data-ttu-id="57077-221">Usare `GrpcChannel` per costruire il client Greeter:</span><span class="sxs-lookup"><span data-stu-id="57077-221">Using the `GrpcChannel` to construct the Greeter client:</span></span>

[!code-csharp[](~/tutorials/grpc/grpc-start/sample/GrpcGreeterClient/Program.cs?name=snippet&highlight=3-5)]

<span data-ttu-id="57077-222">Il client Greeter chiama il metodo `SayHello` asincrono.</span><span class="sxs-lookup"><span data-stu-id="57077-222">The Greeter client calls the asynchronous `SayHello` method.</span></span> <span data-ttu-id="57077-223">Viene visualizzato il risultato della chiamata `SayHello`:</span><span class="sxs-lookup"><span data-stu-id="57077-223">The result of the `SayHello` call is displayed:</span></span>

[!code-csharp[](~/tutorials/grpc/grpc-start/sample/GrpcGreeterClient/Program.cs?name=snippet&highlight=6-8)]

## <a name="test-the-grpc-client-with-the-grpc-greeter-service"></a><span data-ttu-id="57077-224">Testare il client gRPC con il servizio Greeter gRPC</span><span class="sxs-lookup"><span data-stu-id="57077-224">Test the gRPC client with the gRPC Greeter service</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="57077-225">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="57077-225">Visual Studio</span></span>](#tab/visual-studio)

* <span data-ttu-id="57077-226">Nel servizio Greeter premere `Ctrl+F5` per avviare il server senza il debugger.</span><span class="sxs-lookup"><span data-stu-id="57077-226">In the Greeter service, press `Ctrl+F5` to start the server without the debugger.</span></span>
* <span data-ttu-id="57077-227">Nel progetto `GrpcGreeterClient` premere `Ctrl+F5` per avviare il client senza il debugger.</span><span class="sxs-lookup"><span data-stu-id="57077-227">In the `GrpcGreeterClient` project, press `Ctrl+F5` to start the client without the debugger.</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="57077-228">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="57077-228">Visual Studio Code</span></span>](#tab/visual-studio-code)

* <span data-ttu-id="57077-229">Avviare il servizio Greeter.</span><span class="sxs-lookup"><span data-stu-id="57077-229">Start the Greeter service.</span></span>
* <span data-ttu-id="57077-230">Avviare il client.</span><span class="sxs-lookup"><span data-stu-id="57077-230">Start the client.</span></span>


# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="57077-231">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="57077-231">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

* <span data-ttu-id="57077-232">A causa del [problema http/2 TLS](xref:grpc/troubleshoot#unable-to-start-aspnet-core-grpc-app-on-macos)indicato in precedenza nella soluzione MacOS, è necessario aggiornare l'indirizzo del canale nel client a " http://localhost:5000 ".</span><span class="sxs-lookup"><span data-stu-id="57077-232">Due to the previously mentioned [HTTP/2 TLS issue on macOS workaround](xref:grpc/troubleshoot#unable-to-start-aspnet-core-grpc-app-on-macos), you'll need to update the channel address in the client to "http://localhost:5000".</span></span> <span data-ttu-id="57077-233">Aggiornare la riga 13 di **GrpcGreeterClient/Program. cs** da leggere:</span><span class="sxs-lookup"><span data-stu-id="57077-233">Update line 13 of **GrpcGreeterClient/Program.cs** to read:</span></span>
  ```csharp
  using var channel = GrpcChannel.ForAddress("http://localhost:5000");
  ``` 
* <span data-ttu-id="57077-234">Avviare il servizio Greeter.</span><span class="sxs-lookup"><span data-stu-id="57077-234">Start the Greeter service.</span></span>
* <span data-ttu-id="57077-235">Avviare il client.</span><span class="sxs-lookup"><span data-stu-id="57077-235">Start the client.</span></span>

---

<span data-ttu-id="57077-236">Il client invia un messaggio di saluto al servizio con un messaggio contenente il nome *GreeterClient*.</span><span class="sxs-lookup"><span data-stu-id="57077-236">The client sends a greeting to the service with a message containing its name, *GreeterClient*.</span></span> <span data-ttu-id="57077-237">Il servizio invia il messaggio "Hello GreeterClient" come risposta.</span><span class="sxs-lookup"><span data-stu-id="57077-237">The service sends the message "Hello GreeterClient" as a response.</span></span> <span data-ttu-id="57077-238">La risposta "Hello GreeterClient" viene visualizzata al prompt dei comandi:</span><span class="sxs-lookup"><span data-stu-id="57077-238">The "Hello GreeterClient" response is displayed in the command prompt:</span></span>

```console
Greeting: Hello GreeterClient
Press any key to exit...
```

<span data-ttu-id="57077-239">Il servizio gRPC registra i dettagli della chiamata con esito positivo nei log scritti al prompt dei comandi:</span><span class="sxs-lookup"><span data-stu-id="57077-239">The gRPC service records the details of the successful call in the logs written to the command prompt:</span></span>

```console
info: Microsoft.Hosting.Lifetime[0]
      Now listening on: https://localhost:5001
info: Microsoft.Hosting.Lifetime[0]
      Application started. Press Ctrl+C to shut down.
info: Microsoft.Hosting.Lifetime[0]
      Hosting environment: Development
info: Microsoft.Hosting.Lifetime[0]
      Content root path: C:\GH\aspnet\docs\4\Docs\aspnetcore\tutorials\grpc\grpc-start\sample\GrpcGreeter
info: Microsoft.AspNetCore.Hosting.Diagnostics[1]
      Request starting HTTP/2 POST https://localhost:5001/Greet.Greeter/SayHello application/grpc
info: Microsoft.AspNetCore.Routing.EndpointMiddleware[0]
      Executing endpoint 'gRPC - /Greet.Greeter/SayHello'
info: Microsoft.AspNetCore.Routing.EndpointMiddleware[1]
      Executed endpoint 'gRPC - /Greet.Greeter/SayHello'
info: Microsoft.AspNetCore.Hosting.Diagnostics[2]
      Request finished in 78.32260000000001ms 200 application/grpc
```

> [!NOTE]
> <span data-ttu-id="57077-240">Il codice in questo articolo richiede il certificato di sviluppo HTTPS ASP.NET Core per proteggere il servizio gRPC.</span><span class="sxs-lookup"><span data-stu-id="57077-240">The code in this article requires the ASP.NET Core HTTPS development certificate to secure the gRPC service.</span></span> <span data-ttu-id="57077-241">Se il client gRPC .NET ha esito negativo con il messaggio `The remote certificate is invalid according to the validation procedure.` o `The SSL connection could not be established.` , il certificato di sviluppo non è attendibile.</span><span class="sxs-lookup"><span data-stu-id="57077-241">If the .NET gRPC client fails with the message `The remote certificate is invalid according to the validation procedure.` or `The SSL connection could not be established.`, the development certificate isn't trusted.</span></span> <span data-ttu-id="57077-242">Per risolvere questo problema, vedere [chiamare un servizio gRPC con un certificato non attendibile/non valido](xref:grpc/troubleshoot#call-a-grpc-service-with-an-untrustedinvalid-certificate).</span><span class="sxs-lookup"><span data-stu-id="57077-242">To fix this issue, see [Call a gRPC service with an untrusted/invalid certificate](xref:grpc/troubleshoot#call-a-grpc-service-with-an-untrustedinvalid-certificate).</span></span>

[!INCLUDE[](~/includes/gRPCazure.md)]

### <a name="next-steps"></a><span data-ttu-id="57077-243">Passaggi successivi</span><span class="sxs-lookup"><span data-stu-id="57077-243">Next steps</span></span>

* <xref:grpc/index>
* <xref:grpc/basics>
* <xref:grpc/migration>
