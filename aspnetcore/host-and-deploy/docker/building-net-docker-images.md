---
title: Immagini Docker per ASP.NET Core
author: rick-anderson
description: Informazioni su come usare le immagini Docker ASP.NET Core pubblicate dal registro docker. Esegui il pull e crea immagini personalizzate.
ms.author: riande
ms.custom: mvc
ms.date: 01/04/2021
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
uid: host-and-deploy/docker/building-net-docker-images
ms.openlocfilehash: b29ce03366e5c0e815de0874f5b96efb9ba5326c
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102585955"
---
# <a name="docker-images-for-aspnet-core"></a><span data-ttu-id="79f6b-104">Immagini Docker per ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="79f6b-104">Docker images for ASP.NET Core</span></span>

<span data-ttu-id="79f6b-105">Questa esercitazione mostra come eseguire un'app ASP.NET Core in contenitori Docker.</span><span class="sxs-lookup"><span data-stu-id="79f6b-105">This tutorial shows how to run an ASP.NET Core app in Docker containers.</span></span>

<span data-ttu-id="79f6b-106">In questa esercitazione:</span><span class="sxs-lookup"><span data-stu-id="79f6b-106">In this tutorial, you:</span></span>
> [!div class="checklist"]
> * <span data-ttu-id="79f6b-107">Informazioni sulle ASP.NET Core immagini Docker</span><span class="sxs-lookup"><span data-stu-id="79f6b-107">Learn about ASP.NET Core Docker images</span></span>
> * <span data-ttu-id="79f6b-108">Download di un'app di esempio ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="79f6b-108">Download an ASP.NET Core sample app</span></span>
> * <span data-ttu-id="79f6b-109">Eseguire l'app di esempio in locale</span><span class="sxs-lookup"><span data-stu-id="79f6b-109">Run the sample app locally</span></span>
> * <span data-ttu-id="79f6b-110">Esecuzione dell'app di esempio in contenitori Linux</span><span class="sxs-lookup"><span data-stu-id="79f6b-110">Run the sample app in Linux containers</span></span>
> * <span data-ttu-id="79f6b-111">Esecuzione dell'app di esempio in contenitori Windows</span><span class="sxs-lookup"><span data-stu-id="79f6b-111">Run the sample app in Windows containers</span></span>
> * <span data-ttu-id="79f6b-112">Compilazione e distribuzione manuali</span><span class="sxs-lookup"><span data-stu-id="79f6b-112">Build and deploy manually</span></span>

## <a name="aspnet-core-docker-images"></a><span data-ttu-id="79f6b-113">Immagini Docker per ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="79f6b-113">ASP.NET Core Docker images</span></span>

<span data-ttu-id="79f6b-114">Per questa esercitazione si scaricherà un'app di esempio ASP.NET Core, che verrà eseguita in contenitori Docker.</span><span class="sxs-lookup"><span data-stu-id="79f6b-114">For this tutorial, you download an ASP.NET Core sample app and run it in Docker containers.</span></span> <span data-ttu-id="79f6b-115">L'esempio usa contenitori sia Linux che Windows.</span><span class="sxs-lookup"><span data-stu-id="79f6b-115">The sample works with both Linux and Windows containers.</span></span>

<span data-ttu-id="79f6b-116">Il documento Dockerfile di esempio usa la [funzionalità di compilazione in più fasi di Docker](https://docs.docker.com/engine/userguide/eng-image/multistage-build/) per la compilazione e l'esecuzione in contenitori diversi.</span><span class="sxs-lookup"><span data-stu-id="79f6b-116">The sample Dockerfile uses the [Docker multi-stage build feature](https://docs.docker.com/engine/userguide/eng-image/multistage-build/) to build and run in different containers.</span></span> <span data-ttu-id="79f6b-117">I contenitori di compilazione ed esecuzione vengono creati da immagini fornite nell'hub Docker da Microsoft:</span><span class="sxs-lookup"><span data-stu-id="79f6b-117">The build and run containers are created from images that are provided in Docker Hub by Microsoft:</span></span>

::: moniker range=">= aspnetcore-5.0"

* `dotnet/sdk`

  <span data-ttu-id="79f6b-118">L'esempio usa questa immagine per la compilazione dell'app.</span><span class="sxs-lookup"><span data-stu-id="79f6b-118">The sample uses this image for building the app.</span></span> <span data-ttu-id="79f6b-119">L'immagine contiene .NET SDK, che include gli strumenti da riga di comando (CLI).</span><span class="sxs-lookup"><span data-stu-id="79f6b-119">The image contains the .NET SDK, which includes the Command Line Tools (CLI).</span></span> <span data-ttu-id="79f6b-120">L'immagine è ottimizzata per lo sviluppo, il debug e il testing unità locali.</span><span class="sxs-lookup"><span data-stu-id="79f6b-120">The image is optimized for local development, debugging, and unit testing.</span></span> <span data-ttu-id="79f6b-121">Gli strumenti installati per lo sviluppo e la compilazione rendono l'immagine relativamente grande.</span><span class="sxs-lookup"><span data-stu-id="79f6b-121">The tools installed for development and compilation make the image relatively large.</span></span>

::: moniker-end

::: moniker range="< aspnetcore-5.0"

* `dotnet/core/sdk`

  <span data-ttu-id="79f6b-122">L'esempio usa questa immagine per la compilazione dell'app.</span><span class="sxs-lookup"><span data-stu-id="79f6b-122">The sample uses this image for building the app.</span></span> <span data-ttu-id="79f6b-123">L'immagine contiene .NET Core SDK, che include strumenti da riga di comando (interfaccia della riga di comando).</span><span class="sxs-lookup"><span data-stu-id="79f6b-123">The image contains the .NET Core SDK, which includes the Command Line Tools (CLI).</span></span> <span data-ttu-id="79f6b-124">L'immagine è ottimizzata per lo sviluppo, il debug e il testing unità locali.</span><span class="sxs-lookup"><span data-stu-id="79f6b-124">The image is optimized for local development, debugging, and unit testing.</span></span> <span data-ttu-id="79f6b-125">Gli strumenti installati per lo sviluppo e la compilazione rendono l'immagine relativamente grande.</span><span class="sxs-lookup"><span data-stu-id="79f6b-125">The tools installed for development and compilation make the image relatively large.</span></span>

::: moniker-end

::: moniker range=">= aspnetcore-5.0"

* `dotnet/aspnet`

::: moniker-end

::: moniker range="< aspnetcore-5.0"

* `dotnet/core/aspnet`

::: moniker-end

   <span data-ttu-id="79f6b-126">L'esempio usa questa immagine per l'esecuzione dell'app.</span><span class="sxs-lookup"><span data-stu-id="79f6b-126">The sample uses this image for running the app.</span></span> <span data-ttu-id="79f6b-127">L'immagine contiene il runtime e le librerie ASP.NET Core ed è ottimizzata per l'esecuzione di app nell'ambiente di produzione.</span><span class="sxs-lookup"><span data-stu-id="79f6b-127">The image contains the ASP.NET Core runtime and libraries and is optimized for running apps in production.</span></span> <span data-ttu-id="79f6b-128">Progettata per offrire velocità di distribuzione e avvio dell'app, l'immagine è relativamente piccola, in modo da ottimizzare le prestazioni di rete dal registro Docker all'host Docker.</span><span class="sxs-lookup"><span data-stu-id="79f6b-128">Designed for speed of deployment and app startup, the image is relatively small, so network performance from Docker Registry to Docker host is optimized.</span></span> <span data-ttu-id="79f6b-129">Solo i file binari e i contenuti necessari per eseguire un'app vengono copiati nel contenitore.</span><span class="sxs-lookup"><span data-stu-id="79f6b-129">Only the binaries and content needed to run an app are copied to the container.</span></span> <span data-ttu-id="79f6b-130">I contenuti sono pronti per l'esecuzione, fornendo i tempi più rapidi da `docker run` all'avvio dell'app.</span><span class="sxs-lookup"><span data-stu-id="79f6b-130">The contents are ready to run, enabling the fastest time from `docker run` to app startup.</span></span> <span data-ttu-id="79f6b-131">La compilazione dinamica del codice non è necessaria nel modello di Docker.</span><span class="sxs-lookup"><span data-stu-id="79f6b-131">Dynamic code compilation isn't needed in the Docker model.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="79f6b-132">Prerequisiti</span><span class="sxs-lookup"><span data-stu-id="79f6b-132">Prerequisites</span></span>

::: moniker range=">= aspnetcore-5.0"

* [<span data-ttu-id="79f6b-133">.NET SDK 5.0</span><span class="sxs-lookup"><span data-stu-id="79f6b-133">.NET SDK 5.0</span></span>](https://dotnet.microsoft.com/download)

::: moniker-end

::: moniker range=">= aspnetcore-3.0 < aspnetcore-5.0"

* [<span data-ttu-id="79f6b-134">.NET Core SDK 3.1</span><span class="sxs-lookup"><span data-stu-id="79f6b-134">.NET Core SDK 3.1</span></span>](https://dotnet.microsoft.com/download)

::: moniker-end

::: moniker range="< aspnetcore-3.0"

* [<span data-ttu-id="79f6b-135">.NET Core 2.2 SDK</span><span class="sxs-lookup"><span data-stu-id="79f6b-135">.NET Core 2.2 SDK</span></span>](https://dotnet.microsoft.com/download/dotnet-core)

::: moniker-end

* <span data-ttu-id="79f6b-136">Client di Docker 18.03 o versione successiva</span><span class="sxs-lookup"><span data-stu-id="79f6b-136">Docker client 18.03 or later</span></span>

  * <span data-ttu-id="79f6b-137">Distribuzioni di Linux</span><span class="sxs-lookup"><span data-stu-id="79f6b-137">Linux distributions</span></span>
    * [<span data-ttu-id="79f6b-138">CentOS</span><span class="sxs-lookup"><span data-stu-id="79f6b-138">CentOS</span></span>](https://docs.docker.com/install/linux/docker-ce/centos/)
    * [<span data-ttu-id="79f6b-139">Debian</span><span class="sxs-lookup"><span data-stu-id="79f6b-139">Debian</span></span>](https://docs.docker.com/install/linux/docker-ce/debian/)
    * [<span data-ttu-id="79f6b-140">Fedora</span><span class="sxs-lookup"><span data-stu-id="79f6b-140">Fedora</span></span>](https://docs.docker.com/install/linux/docker-ce/fedora/)
    * [<span data-ttu-id="79f6b-141">Ubuntu</span><span class="sxs-lookup"><span data-stu-id="79f6b-141">Ubuntu</span></span>](https://docs.docker.com/install/linux/docker-ce/ubuntu/)
  * [<span data-ttu-id="79f6b-142">macOS</span><span class="sxs-lookup"><span data-stu-id="79f6b-142">macOS</span></span>](https://docs.docker.com/docker-for-mac/install/)
  * [<span data-ttu-id="79f6b-143">Windows</span><span class="sxs-lookup"><span data-stu-id="79f6b-143">Windows</span></span>](https://docs.docker.com/docker-for-windows/install/)

* [<span data-ttu-id="79f6b-144">Git</span><span class="sxs-lookup"><span data-stu-id="79f6b-144">Git</span></span>](https://git-scm.com/download)

## <a name="download-the-sample-app"></a><span data-ttu-id="79f6b-145">Scaricare l'app di esempio</span><span class="sxs-lookup"><span data-stu-id="79f6b-145">Download the sample app</span></span>

* <span data-ttu-id="79f6b-146">Scaricare l'esempio clonando il [repository docker .NET](https://github.com/dotnet/dotnet-docker):</span><span class="sxs-lookup"><span data-stu-id="79f6b-146">Download the sample by cloning the [.NET Docker repository](https://github.com/dotnet/dotnet-docker):</span></span> 

  ```console
  git clone https://github.com/dotnet/dotnet-docker
  ```

## <a name="run-the-app-locally"></a><span data-ttu-id="79f6b-147">Eseguire l'app in locale</span><span class="sxs-lookup"><span data-stu-id="79f6b-147">Run the app locally</span></span>

* <span data-ttu-id="79f6b-148">Passare alla cartella del progetto in *dotnet-docker/samples/aspnetapp/aspnetapp*.</span><span class="sxs-lookup"><span data-stu-id="79f6b-148">Navigate to the project folder at *dotnet-docker/samples/aspnetapp/aspnetapp*.</span></span>

* <span data-ttu-id="79f6b-149">Eseguire il comando seguente per compilare ed eseguire l'app in locale:</span><span class="sxs-lookup"><span data-stu-id="79f6b-149">Run the following command to build and run the app locally:</span></span>

  ```dotnetcli
  dotnet run
  ```

* <span data-ttu-id="79f6b-150">Passare a `http://localhost:5000` in un browser per testare l'app.</span><span class="sxs-lookup"><span data-stu-id="79f6b-150">Go to `http://localhost:5000` in a browser to test the app.</span></span>

* <span data-ttu-id="79f6b-151">Premere CTRL+C al prompt dei comandi per arrestare l'app.</span><span class="sxs-lookup"><span data-stu-id="79f6b-151">Press Ctrl+C at the command prompt to stop the app.</span></span>

## <a name="run-in-a-linux-container"></a><span data-ttu-id="79f6b-152">Eseguire l'app in un contenitore Linux</span><span class="sxs-lookup"><span data-stu-id="79f6b-152">Run in a Linux container</span></span>

* <span data-ttu-id="79f6b-153">Nel client Docker [passare a contenitori Linux](https://docs.docker.com/docker-for-windows/#switch-between-windows-and-linux-containers).</span><span class="sxs-lookup"><span data-stu-id="79f6b-153">In the Docker client, [switch to Linux containers](https://docs.docker.com/docker-for-windows/#switch-between-windows-and-linux-containers).</span></span>

* <span data-ttu-id="79f6b-154">Passare alla cartella di Dockerfile in *dotnet-docker/samples/aspnetapp*.</span><span class="sxs-lookup"><span data-stu-id="79f6b-154">Navigate to the Dockerfile folder at *dotnet-docker/samples/aspnetapp*.</span></span>

* <span data-ttu-id="79f6b-155">Eseguire i comandi seguenti per compilare ed eseguire l'esempio in Docker:</span><span class="sxs-lookup"><span data-stu-id="79f6b-155">Run the following commands to build and run the sample in Docker:</span></span>

  ```console
  docker build -t aspnetapp .
  docker run -it --rm -p 5000:80 --name aspnetcore_sample aspnetapp
  ```

  <span data-ttu-id="79f6b-156">Gli argomenti del comando `build`:</span><span class="sxs-lookup"><span data-stu-id="79f6b-156">The `build` command arguments:</span></span>
  * <span data-ttu-id="79f6b-157">Denominano l'immagine aspnetapp.</span><span class="sxs-lookup"><span data-stu-id="79f6b-157">Name the image aspnetapp.</span></span>
  * <span data-ttu-id="79f6b-158">Cercano il documento Dockerfile nella cartella corrente (il punto alla fine).</span><span class="sxs-lookup"><span data-stu-id="79f6b-158">Look for the Dockerfile in the current folder (the period at the end).</span></span>

  <span data-ttu-id="79f6b-159">Gli argomenti del comando run:</span><span class="sxs-lookup"><span data-stu-id="79f6b-159">The run command arguments:</span></span>
  * <span data-ttu-id="79f6b-160">Allocano uno pseudo TTY e lo tengono aperto anche se non è collegato</span><span class="sxs-lookup"><span data-stu-id="79f6b-160">Allocate a pseudo-TTY and keep it open even if not attached.</span></span> <span data-ttu-id="79f6b-161">(stesso effetto di `--interactive --tty`).</span><span class="sxs-lookup"><span data-stu-id="79f6b-161">(Same effect as `--interactive --tty`.)</span></span>
  * <span data-ttu-id="79f6b-162">Rimuovono automaticamente il contenitore quando viene chiuso.</span><span class="sxs-lookup"><span data-stu-id="79f6b-162">Automatically remove the container when it exits.</span></span>
  * <span data-ttu-id="79f6b-163">Eseguono il mapping della porta 5000 nel computer locale alla porta 80 nel contenitore.</span><span class="sxs-lookup"><span data-stu-id="79f6b-163">Map port 5000 on the local machine to port 80 in the container.</span></span>
  * <span data-ttu-id="79f6b-164">Denominano il contenitore aspnetcore_sample.</span><span class="sxs-lookup"><span data-stu-id="79f6b-164">Name the container aspnetcore_sample.</span></span>
  * <span data-ttu-id="79f6b-165">Specificano l'immagine aspnetapp.</span><span class="sxs-lookup"><span data-stu-id="79f6b-165">Specify the aspnetapp image.</span></span>

* <span data-ttu-id="79f6b-166">Passare a `http://localhost:5000` in un browser per testare l'app.</span><span class="sxs-lookup"><span data-stu-id="79f6b-166">Go to `http://localhost:5000` in a browser to test the app.</span></span>

## <a name="run-in-a-windows-container"></a><span data-ttu-id="79f6b-167">Eseguire l'app in un contenitore Windows</span><span class="sxs-lookup"><span data-stu-id="79f6b-167">Run in a Windows container</span></span>

* <span data-ttu-id="79f6b-168">Nel client Docker [passare a contenitori di Windows](https://docs.docker.com/docker-for-windows/#switch-between-windows-and-linux-containers).</span><span class="sxs-lookup"><span data-stu-id="79f6b-168">In the Docker client, [switch to Windows containers](https://docs.docker.com/docker-for-windows/#switch-between-windows-and-linux-containers).</span></span>

<span data-ttu-id="79f6b-169">Passare alla cartella di Dockerfile in `dotnet-docker/samples/aspnetapp`.</span><span class="sxs-lookup"><span data-stu-id="79f6b-169">Navigate to the docker file folder at `dotnet-docker/samples/aspnetapp`.</span></span>

* <span data-ttu-id="79f6b-170">Eseguire i comandi seguenti per compilare ed eseguire l'esempio in Docker:</span><span class="sxs-lookup"><span data-stu-id="79f6b-170">Run the following commands to build and run the sample in Docker:</span></span>

  ```console
  docker build -t aspnetapp .
  docker run -it --rm --name aspnetcore_sample aspnetapp
  ```

* <span data-ttu-id="79f6b-171">Per i contenitori Windows, è necessario l'indirizzo IP del contenitore (il passaggio a `http://localhost:5000` non funziona):</span><span class="sxs-lookup"><span data-stu-id="79f6b-171">For Windows containers, you need the IP address of the container (browsing to `http://localhost:5000` won't work):</span></span>
  * <span data-ttu-id="79f6b-172">Aprire un altro prompt dei comandi.</span><span class="sxs-lookup"><span data-stu-id="79f6b-172">Open up another command prompt.</span></span>
  * <span data-ttu-id="79f6b-173">Eseguire `docker ps` per visualizzare i contenitori in esecuzione.</span><span class="sxs-lookup"><span data-stu-id="79f6b-173">Run `docker ps` to see the running containers.</span></span> <span data-ttu-id="79f6b-174">Verificare che sia presente il contenitore "aspnetcore_sample".</span><span class="sxs-lookup"><span data-stu-id="79f6b-174">Verify that the "aspnetcore_sample" container is there.</span></span>
  * <span data-ttu-id="79f6b-175">Eseguire `docker exec aspnetcore_sample ipconfig` per visualizzare l'indirizzo IP del contenitore.</span><span class="sxs-lookup"><span data-stu-id="79f6b-175">Run `docker exec aspnetcore_sample ipconfig` to display the IP address of the container.</span></span> <span data-ttu-id="79f6b-176">L'output del comando è simile a questo esempio:</span><span class="sxs-lookup"><span data-stu-id="79f6b-176">The output from the command looks like this example:</span></span>

    ```console
    Ethernet adapter Ethernet:

       Connection-specific DNS Suffix  . : contoso.com
       Link-local IPv6 Address . . . . . : fe80::1967:6598:124:cfa3%4
       IPv4 Address. . . . . . . . . . . : 172.29.245.43
       Subnet Mask . . . . . . . . . . . : 255.255.240.0
       Default Gateway . . . . . . . . . : 172.29.240.1
    ```

* <span data-ttu-id="79f6b-177">Copiare l'indirizzo IPv4 del contenitore, ad esempio 172.29.245.43, e incollarlo nella barra degli indirizzi del browser per testare l'app.</span><span class="sxs-lookup"><span data-stu-id="79f6b-177">Copy the container IPv4 address (for example, 172.29.245.43) and paste into the browser address bar to test the app.</span></span>

## <a name="build-and-deploy-manually"></a><span data-ttu-id="79f6b-178">Compilazione e distribuzione manuali</span><span class="sxs-lookup"><span data-stu-id="79f6b-178">Build and deploy manually</span></span>

<span data-ttu-id="79f6b-179">In alcuni scenari potrebbe essere necessario distribuire un'app in un contenitore copiando gli asset necessari in fase di esecuzione.</span><span class="sxs-lookup"><span data-stu-id="79f6b-179">In some scenarios, you might want to deploy an app to a container by copying its assets that are needed at run time.</span></span> <span data-ttu-id="79f6b-180">Questa sezione mostra come eseguire manualmente la distribuzione.</span><span class="sxs-lookup"><span data-stu-id="79f6b-180">This section shows how to deploy manually.</span></span>

* <span data-ttu-id="79f6b-181">Passare alla cartella del progetto in *dotnet-docker/samples/aspnetapp/aspnetapp*.</span><span class="sxs-lookup"><span data-stu-id="79f6b-181">Navigate to the project folder at *dotnet-docker/samples/aspnetapp/aspnetapp*.</span></span>

* <span data-ttu-id="79f6b-182">Eseguire il comando [dotnet publish](/dotnet/core/tools/dotnet-publish):</span><span class="sxs-lookup"><span data-stu-id="79f6b-182">Run the [dotnet publish](/dotnet/core/tools/dotnet-publish) command:</span></span>

  ```dotnetcli
  dotnet publish -c Release -o published
  ```

  <span data-ttu-id="79f6b-183">Gli argomenti del comando:</span><span class="sxs-lookup"><span data-stu-id="79f6b-183">The command arguments:</span></span>
  * <span data-ttu-id="79f6b-184">Compilare l'app in modalità di rilascio (il valore predefinito è modalità di debug).</span><span class="sxs-lookup"><span data-stu-id="79f6b-184">Build the app in release mode (the default is debug mode).</span></span>
  * <span data-ttu-id="79f6b-185">Creare gli asset nella cartella *pubblicata* .</span><span class="sxs-lookup"><span data-stu-id="79f6b-185">Create the assets in the *published* folder.</span></span>

* <span data-ttu-id="79f6b-186">Eseguire l'app.</span><span class="sxs-lookup"><span data-stu-id="79f6b-186">Run the app.</span></span>

  * <span data-ttu-id="79f6b-187">Windows:</span><span class="sxs-lookup"><span data-stu-id="79f6b-187">Windows:</span></span>

    ```dotnetcli
    dotnet published\aspnetapp.dll
    ```

  * <span data-ttu-id="79f6b-188">Linux:</span><span class="sxs-lookup"><span data-stu-id="79f6b-188">Linux:</span></span>

    ```dotnetcli
    dotnet published/aspnetapp.dll
    ```

* <span data-ttu-id="79f6b-189">Passare a `http://localhost:5000`per visualizzare la home page.</span><span class="sxs-lookup"><span data-stu-id="79f6b-189">Browse to `http://localhost:5000` to see the home page.</span></span>

<span data-ttu-id="79f6b-190">Per usare l'app pubblicata manualmente in un contenitore Docker, creare un nuovo *Dockerfile* e usare il `docker build .` comando per compilare il contenitore.</span><span class="sxs-lookup"><span data-stu-id="79f6b-190">To use the manually published app within a Docker container, create a new *Dockerfile* and use the `docker build .` command to build the container.</span></span>

::: moniker range=">= aspnetcore-5.0"

```dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:5.0 AS runtime
WORKDIR /app
COPY published/aspnetapp.dll ./
ENTRYPOINT ["dotnet", "aspnetapp.dll"]
```

### <a name="the-dockerfile"></a><span data-ttu-id="79f6b-191">Dockerfile</span><span class="sxs-lookup"><span data-stu-id="79f6b-191">The Dockerfile</span></span>

<span data-ttu-id="79f6b-192">Ecco il *Dockerfile* usato dal `docker build` comando che è stato eseguito in precedenza.</span><span class="sxs-lookup"><span data-stu-id="79f6b-192">Here's the *Dockerfile* used by the `docker build` command you ran earlier.</span></span>  <span data-ttu-id="79f6b-193">Usa `dotnet publish` allo stesso modo in cui è stato usato in questa sezione per la compilazione e la distribuzione.</span><span class="sxs-lookup"><span data-stu-id="79f6b-193">It uses `dotnet publish` the same way you did in this section to build and deploy.</span></span>  

```dockerfile
# https://hub.docker.com/_/microsoft-dotnet
FROM mcr.microsoft.com/dotnet/sdk:5.0 AS build
WORKDIR /source

# copy csproj and restore as distinct layers
COPY *.sln .
COPY aspnetapp/*.csproj ./aspnetapp/
RUN dotnet restore

# copy everything else and build app
COPY aspnetapp/. ./aspnetapp/
WORKDIR /source/aspnetapp
RUN dotnet publish -c release -o /app --no-restore

# final stage/image
FROM mcr.microsoft.com/dotnet/aspnet:5.0
WORKDIR /app
COPY --from=build /app ./
ENTRYPOINT ["dotnet", "aspnetapp.dll"]
```

<span data-ttu-id="79f6b-194">Nel *Dockerfile* precedente, i `*.csproj` file vengono copiati e ripristinati come *livelli* distinti.</span><span class="sxs-lookup"><span data-stu-id="79f6b-194">In the preceding *Dockerfile*, the `*.csproj` files are copied and restored as distinct *layers*.</span></span> <span data-ttu-id="79f6b-195">Quando il `docker build` comando compila un'immagine, usa una cache predefinita.</span><span class="sxs-lookup"><span data-stu-id="79f6b-195">When the `docker build` command builds an image, it uses a built-in cache.</span></span> <span data-ttu-id="79f6b-196">Se i `*.csproj` file non sono stati modificati dopo l'ultima esecuzione del `docker build` comando, `dotnet restore` non è necessario eseguire di nuovo il comando.</span><span class="sxs-lookup"><span data-stu-id="79f6b-196">If the `*.csproj` files haven't changed since the `docker build` command last ran, the `dotnet restore` command doesn't need to run again.</span></span> <span data-ttu-id="79f6b-197">Viene invece riutilizzata la cache predefinita per il `dotnet restore` livello corrispondente.</span><span class="sxs-lookup"><span data-stu-id="79f6b-197">Instead, the built-in cache for the corresponding `dotnet restore` layer is reused.</span></span> <span data-ttu-id="79f6b-198">Per ulteriori informazioni, vedere [procedure consigliate per la scrittura di dockerfile](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#leverage-build-cache).</span><span class="sxs-lookup"><span data-stu-id="79f6b-198">For more information, see [Best practices for writing Dockerfiles](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#leverage-build-cache).</span></span>

::: moniker-end

::: moniker range=">= aspnetcore-3.0 < aspnetcore-5.0"

```dockerfile
FROM mcr.microsoft.com/dotnet/core/aspnet:3.0 AS runtime
WORKDIR /app
COPY published/aspnetapp.dll ./
ENTRYPOINT ["dotnet", "aspnetapp.dll"]
```

### <a name="the-dockerfile"></a><span data-ttu-id="79f6b-199">Dockerfile</span><span class="sxs-lookup"><span data-stu-id="79f6b-199">The Dockerfile</span></span>

<span data-ttu-id="79f6b-200">Ecco il *Dockerfile* usato dal `docker build` comando che è stato eseguito in precedenza.</span><span class="sxs-lookup"><span data-stu-id="79f6b-200">Here's the *Dockerfile* used by the `docker build` command you ran earlier.</span></span>  <span data-ttu-id="79f6b-201">Usa `dotnet publish` allo stesso modo in cui è stato usato in questa sezione per la compilazione e la distribuzione.</span><span class="sxs-lookup"><span data-stu-id="79f6b-201">It uses `dotnet publish` the same way you did in this section to build and deploy.</span></span>  

```dockerfile
FROM mcr.microsoft.com/dotnet/core/sdk:3.0 AS build
WORKDIR /app

# copy csproj and restore as distinct layers
COPY *.sln .
COPY aspnetapp/*.csproj ./aspnetapp/
RUN dotnet restore

# copy everything else and build app
COPY aspnetapp/. ./aspnetapp/
WORKDIR /app/aspnetapp
RUN dotnet publish -c Release -o out

FROM mcr.microsoft.com/dotnet/core/aspnet:3.0 AS runtime
WORKDIR /app
COPY --from=build /app/aspnetapp/out ./
ENTRYPOINT ["dotnet", "aspnetapp.dll"]
```

<span data-ttu-id="79f6b-202">Come indicato nel Dockerfile precedente, i `*.csproj` file vengono copiati e ripristinati come *livelli* distinti.</span><span class="sxs-lookup"><span data-stu-id="79f6b-202">As noted in the preceding Dockerfile, the `*.csproj` files are copied and restored as distinct *layers*.</span></span> <span data-ttu-id="79f6b-203">Quando il `docker build` comando compila un'immagine, usa una cache predefinita.</span><span class="sxs-lookup"><span data-stu-id="79f6b-203">When the `docker build` command builds an image, it uses a built-in cache.</span></span> <span data-ttu-id="79f6b-204">Se i `*.csproj` file non sono stati modificati dopo l'ultima esecuzione del `docker build` comando, `dotnet restore` non è necessario eseguire di nuovo il comando.</span><span class="sxs-lookup"><span data-stu-id="79f6b-204">If the `*.csproj` files haven't changed since the `docker build` command last ran, the `dotnet restore` command doesn't need to run again.</span></span> <span data-ttu-id="79f6b-205">Viene invece riutilizzata la cache predefinita per il `dotnet restore` livello corrispondente.</span><span class="sxs-lookup"><span data-stu-id="79f6b-205">Instead, the built-in cache for the corresponding `dotnet restore` layer is reused.</span></span> <span data-ttu-id="79f6b-206">Per ulteriori informazioni, vedere [procedure consigliate per la scrittura di dockerfile](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#leverage-build-cache).</span><span class="sxs-lookup"><span data-stu-id="79f6b-206">For more information, see [Best practices for writing Dockerfiles](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#leverage-build-cache).</span></span>

::: moniker-end

::: moniker range="< aspnetcore-3.0"

```dockerfile
FROM mcr.microsoft.com/dotnet/core/aspnet:2.2 AS runtime
WORKDIR /app
COPY published/aspnetapp.dll ./
ENTRYPOINT ["dotnet", "aspnetapp.dll"]
```

### <a name="the-dockerfile"></a><span data-ttu-id="79f6b-207">Dockerfile</span><span class="sxs-lookup"><span data-stu-id="79f6b-207">The Dockerfile</span></span>

<span data-ttu-id="79f6b-208">Ecco il *Dockerfile* usato dal `docker build` comando che è stato eseguito in precedenza.</span><span class="sxs-lookup"><span data-stu-id="79f6b-208">Here's the *Dockerfile* used by the `docker build` command you ran earlier.</span></span> <span data-ttu-id="79f6b-209">Usa `dotnet publish` allo stesso modo in cui è stato usato in questa sezione per la compilazione e la distribuzione.</span><span class="sxs-lookup"><span data-stu-id="79f6b-209">It uses `dotnet publish` the same way you did in this section to build and deploy.</span></span>  

```dockerfile
FROM mcr.microsoft.com/dotnet/core/sdk:2.2 AS build
WORKDIR /app

# copy csproj and restore as distinct layers
COPY *.sln .
COPY aspnetapp/*.csproj ./aspnetapp/
RUN dotnet restore

# copy everything else and build app
COPY aspnetapp/. ./aspnetapp/
WORKDIR /app/aspnetapp
RUN dotnet publish -c Release -o out

FROM mcr.microsoft.com/dotnet/core/aspnet:2.2 AS runtime
WORKDIR /app
COPY --from=build /app/aspnetapp/out ./
ENTRYPOINT ["dotnet", "aspnetapp.dll"]
```

::: moniker-end

## <a name="additional-resources"></a><span data-ttu-id="79f6b-210">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="79f6b-210">Additional resources</span></span>

* [<span data-ttu-id="79f6b-211">Comando build di Docker</span><span class="sxs-lookup"><span data-stu-id="79f6b-211">Docker build command</span></span>](https://docs.docker.com/engine/reference/commandline/build)
* [<span data-ttu-id="79f6b-212">Comando run di Docker</span><span class="sxs-lookup"><span data-stu-id="79f6b-212">Docker run command</span></span>](https://docs.docker.com/engine/reference/commandline/run)
* <span data-ttu-id="79f6b-213">[Esempio di Docker per ASP.NET Core](https://github.com/dotnet/dotnet-docker) (usato in questa esercitazione)</span><span class="sxs-lookup"><span data-stu-id="79f6b-213">[ASP.NET Core Docker sample](https://github.com/dotnet/dotnet-docker) (The one used in this tutorial.)</span></span>
* [<span data-ttu-id="79f6b-214">Configurare ASP.NET Core per l'utilizzo di server proxy e servizi di bilanciamento del carico</span><span class="sxs-lookup"><span data-stu-id="79f6b-214">Configure ASP.NET Core to work with proxy servers and load balancers</span></span>](../proxy-load-balancer.md)
* <span data-ttu-id="79f6b-215">[Working with Visual Studio Docker Tools](./visual-studio-tools-for-docker.md) (Uso degli strumenti Docker per Visual Studio)</span><span class="sxs-lookup"><span data-stu-id="79f6b-215">[Working with Visual Studio Docker Tools](./visual-studio-tools-for-docker.md)</span></span>
* [<span data-ttu-id="79f6b-216">Debug con Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="79f6b-216">Debugging with Visual Studio Code</span></span>](https://code.visualstudio.com/docs/nodejs/debugging-recipes#_debug-nodejs-in-docker-containers)
* [<span data-ttu-id="79f6b-217">GC con contenitori Docker e Small</span><span class="sxs-lookup"><span data-stu-id="79f6b-217">GC using Docker and small containers</span></span>](xref:performance/memory#sc)

## <a name="next-steps"></a><span data-ttu-id="79f6b-218">Passaggi successivi</span><span class="sxs-lookup"><span data-stu-id="79f6b-218">Next steps</span></span>

<span data-ttu-id="79f6b-219">Il repository Git che contiene l'app di esempio include anche la documentazione.</span><span class="sxs-lookup"><span data-stu-id="79f6b-219">The Git repository that contains the sample app also includes documentation.</span></span> <span data-ttu-id="79f6b-220">Per una panoramica delle risorse disponibili nel repository, vedere il [file LEGGIMI](https://github.com/dotnet/dotnet-docker/blob/main/samples/aspnetapp/README.md).</span><span class="sxs-lookup"><span data-stu-id="79f6b-220">For an overview of the resources available in the repository, see [the README file](https://github.com/dotnet/dotnet-docker/blob/main/samples/aspnetapp/README.md).</span></span> <span data-ttu-id="79f6b-221">In particolare, vedere le informazioni su come implementare HTTPS:</span><span class="sxs-lookup"><span data-stu-id="79f6b-221">In particular, learn how to implement HTTPS:</span></span>

> [!div class="nextstepaction"]
> <span data-ttu-id="79f6b-222">[Developing ASP.NET Core Applications with Docker over HTTPS](https://github.com/dotnet/dotnet-docker/blob/main/samples/run-aspnetcore-https-development.md) (Sviluppo di applicazioni ASP.NET Core con Docker su HTTPS)</span><span class="sxs-lookup"><span data-stu-id="79f6b-222">[Developing ASP.NET Core Applications with Docker over HTTPS](https://github.com/dotnet/dotnet-docker/blob/main/samples/run-aspnetcore-https-development.md)</span></span>
