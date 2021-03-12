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
# <a name="docker-images-for-aspnet-core"></a>Immagini Docker per ASP.NET Core

Questa esercitazione mostra come eseguire un'app ASP.NET Core in contenitori Docker.

In questa esercitazione:
> [!div class="checklist"]
> * Informazioni sulle ASP.NET Core immagini Docker
> * Download di un'app di esempio ASP.NET Core
> * Eseguire l'app di esempio in locale
> * Esecuzione dell'app di esempio in contenitori Linux
> * Esecuzione dell'app di esempio in contenitori Windows
> * Compilazione e distribuzione manuali

## <a name="aspnet-core-docker-images"></a>Immagini Docker per ASP.NET Core

Per questa esercitazione si scaricherà un'app di esempio ASP.NET Core, che verrà eseguita in contenitori Docker. L'esempio usa contenitori sia Linux che Windows.

Il documento Dockerfile di esempio usa la [funzionalità di compilazione in più fasi di Docker](https://docs.docker.com/engine/userguide/eng-image/multistage-build/) per la compilazione e l'esecuzione in contenitori diversi. I contenitori di compilazione ed esecuzione vengono creati da immagini fornite nell'hub Docker da Microsoft:

::: moniker range=">= aspnetcore-5.0"

* `dotnet/sdk`

  L'esempio usa questa immagine per la compilazione dell'app. L'immagine contiene .NET SDK, che include gli strumenti da riga di comando (CLI). L'immagine è ottimizzata per lo sviluppo, il debug e il testing unità locali. Gli strumenti installati per lo sviluppo e la compilazione rendono l'immagine relativamente grande.

::: moniker-end

::: moniker range="< aspnetcore-5.0"

* `dotnet/core/sdk`

  L'esempio usa questa immagine per la compilazione dell'app. L'immagine contiene .NET Core SDK, che include strumenti da riga di comando (interfaccia della riga di comando). L'immagine è ottimizzata per lo sviluppo, il debug e il testing unità locali. Gli strumenti installati per lo sviluppo e la compilazione rendono l'immagine relativamente grande.

::: moniker-end

::: moniker range=">= aspnetcore-5.0"

* `dotnet/aspnet`

::: moniker-end

::: moniker range="< aspnetcore-5.0"

* `dotnet/core/aspnet`

::: moniker-end

   L'esempio usa questa immagine per l'esecuzione dell'app. L'immagine contiene il runtime e le librerie ASP.NET Core ed è ottimizzata per l'esecuzione di app nell'ambiente di produzione. Progettata per offrire velocità di distribuzione e avvio dell'app, l'immagine è relativamente piccola, in modo da ottimizzare le prestazioni di rete dal registro Docker all'host Docker. Solo i file binari e i contenuti necessari per eseguire un'app vengono copiati nel contenitore. I contenuti sono pronti per l'esecuzione, fornendo i tempi più rapidi da `docker run` all'avvio dell'app. La compilazione dinamica del codice non è necessaria nel modello di Docker.

## <a name="prerequisites"></a>Prerequisiti

::: moniker range=">= aspnetcore-5.0"

* [.NET SDK 5.0](https://dotnet.microsoft.com/download)

::: moniker-end

::: moniker range=">= aspnetcore-3.0 < aspnetcore-5.0"

* [.NET Core SDK 3.1](https://dotnet.microsoft.com/download)

::: moniker-end

::: moniker range="< aspnetcore-3.0"

* [.NET Core 2.2 SDK](https://dotnet.microsoft.com/download/dotnet-core)

::: moniker-end

* Client di Docker 18.03 o versione successiva

  * Distribuzioni di Linux
    * [CentOS](https://docs.docker.com/install/linux/docker-ce/centos/)
    * [Debian](https://docs.docker.com/install/linux/docker-ce/debian/)
    * [Fedora](https://docs.docker.com/install/linux/docker-ce/fedora/)
    * [Ubuntu](https://docs.docker.com/install/linux/docker-ce/ubuntu/)
  * [macOS](https://docs.docker.com/docker-for-mac/install/)
  * [Windows](https://docs.docker.com/docker-for-windows/install/)

* [Git](https://git-scm.com/download)

## <a name="download-the-sample-app"></a>Scaricare l'app di esempio

* Scaricare l'esempio clonando il [repository docker .NET](https://github.com/dotnet/dotnet-docker): 

  ```console
  git clone https://github.com/dotnet/dotnet-docker
  ```

## <a name="run-the-app-locally"></a>Eseguire l'app in locale

* Passare alla cartella del progetto in *dotnet-docker/samples/aspnetapp/aspnetapp*.

* Eseguire il comando seguente per compilare ed eseguire l'app in locale:

  ```dotnetcli
  dotnet run
  ```

* Passare a `http://localhost:5000` in un browser per testare l'app.

* Premere CTRL+C al prompt dei comandi per arrestare l'app.

## <a name="run-in-a-linux-container"></a>Eseguire l'app in un contenitore Linux

* Nel client Docker [passare a contenitori Linux](https://docs.docker.com/docker-for-windows/#switch-between-windows-and-linux-containers).

* Passare alla cartella di Dockerfile in *dotnet-docker/samples/aspnetapp*.

* Eseguire i comandi seguenti per compilare ed eseguire l'esempio in Docker:

  ```console
  docker build -t aspnetapp .
  docker run -it --rm -p 5000:80 --name aspnetcore_sample aspnetapp
  ```

  Gli argomenti del comando `build`:
  * Denominano l'immagine aspnetapp.
  * Cercano il documento Dockerfile nella cartella corrente (il punto alla fine).

  Gli argomenti del comando run:
  * Allocano uno pseudo TTY e lo tengono aperto anche se non è collegato (stesso effetto di `--interactive --tty`).
  * Rimuovono automaticamente il contenitore quando viene chiuso.
  * Eseguono il mapping della porta 5000 nel computer locale alla porta 80 nel contenitore.
  * Denominano il contenitore aspnetcore_sample.
  * Specificano l'immagine aspnetapp.

* Passare a `http://localhost:5000` in un browser per testare l'app.

## <a name="run-in-a-windows-container"></a>Eseguire l'app in un contenitore Windows

* Nel client Docker [passare a contenitori di Windows](https://docs.docker.com/docker-for-windows/#switch-between-windows-and-linux-containers).

Passare alla cartella di Dockerfile in `dotnet-docker/samples/aspnetapp`.

* Eseguire i comandi seguenti per compilare ed eseguire l'esempio in Docker:

  ```console
  docker build -t aspnetapp .
  docker run -it --rm --name aspnetcore_sample aspnetapp
  ```

* Per i contenitori Windows, è necessario l'indirizzo IP del contenitore (il passaggio a `http://localhost:5000` non funziona):
  * Aprire un altro prompt dei comandi.
  * Eseguire `docker ps` per visualizzare i contenitori in esecuzione. Verificare che sia presente il contenitore "aspnetcore_sample".
  * Eseguire `docker exec aspnetcore_sample ipconfig` per visualizzare l'indirizzo IP del contenitore. L'output del comando è simile a questo esempio:

    ```console
    Ethernet adapter Ethernet:

       Connection-specific DNS Suffix  . : contoso.com
       Link-local IPv6 Address . . . . . : fe80::1967:6598:124:cfa3%4
       IPv4 Address. . . . . . . . . . . : 172.29.245.43
       Subnet Mask . . . . . . . . . . . : 255.255.240.0
       Default Gateway . . . . . . . . . : 172.29.240.1
    ```

* Copiare l'indirizzo IPv4 del contenitore, ad esempio 172.29.245.43, e incollarlo nella barra degli indirizzi del browser per testare l'app.

## <a name="build-and-deploy-manually"></a>Compilazione e distribuzione manuali

In alcuni scenari potrebbe essere necessario distribuire un'app in un contenitore copiando gli asset necessari in fase di esecuzione. Questa sezione mostra come eseguire manualmente la distribuzione.

* Passare alla cartella del progetto in *dotnet-docker/samples/aspnetapp/aspnetapp*.

* Eseguire il comando [dotnet publish](/dotnet/core/tools/dotnet-publish):

  ```dotnetcli
  dotnet publish -c Release -o published
  ```

  Gli argomenti del comando:
  * Compilare l'app in modalità di rilascio (il valore predefinito è modalità di debug).
  * Creare gli asset nella cartella *pubblicata* .

* Eseguire l'app.

  * Windows:

    ```dotnetcli
    dotnet published\aspnetapp.dll
    ```

  * Linux:

    ```dotnetcli
    dotnet published/aspnetapp.dll
    ```

* Passare a `http://localhost:5000`per visualizzare la home page.

Per usare l'app pubblicata manualmente in un contenitore Docker, creare un nuovo *Dockerfile* e usare il `docker build .` comando per compilare il contenitore.

::: moniker range=">= aspnetcore-5.0"

```dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:5.0 AS runtime
WORKDIR /app
COPY published/aspnetapp.dll ./
ENTRYPOINT ["dotnet", "aspnetapp.dll"]
```

### <a name="the-dockerfile"></a>Dockerfile

Ecco il *Dockerfile* usato dal `docker build` comando che è stato eseguito in precedenza.  Usa `dotnet publish` allo stesso modo in cui è stato usato in questa sezione per la compilazione e la distribuzione.  

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

Nel *Dockerfile* precedente, i `*.csproj` file vengono copiati e ripristinati come *livelli* distinti. Quando il `docker build` comando compila un'immagine, usa una cache predefinita. Se i `*.csproj` file non sono stati modificati dopo l'ultima esecuzione del `docker build` comando, `dotnet restore` non è necessario eseguire di nuovo il comando. Viene invece riutilizzata la cache predefinita per il `dotnet restore` livello corrispondente. Per ulteriori informazioni, vedere [procedure consigliate per la scrittura di dockerfile](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#leverage-build-cache).

::: moniker-end

::: moniker range=">= aspnetcore-3.0 < aspnetcore-5.0"

```dockerfile
FROM mcr.microsoft.com/dotnet/core/aspnet:3.0 AS runtime
WORKDIR /app
COPY published/aspnetapp.dll ./
ENTRYPOINT ["dotnet", "aspnetapp.dll"]
```

### <a name="the-dockerfile"></a>Dockerfile

Ecco il *Dockerfile* usato dal `docker build` comando che è stato eseguito in precedenza.  Usa `dotnet publish` allo stesso modo in cui è stato usato in questa sezione per la compilazione e la distribuzione.  

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

Come indicato nel Dockerfile precedente, i `*.csproj` file vengono copiati e ripristinati come *livelli* distinti. Quando il `docker build` comando compila un'immagine, usa una cache predefinita. Se i `*.csproj` file non sono stati modificati dopo l'ultima esecuzione del `docker build` comando, `dotnet restore` non è necessario eseguire di nuovo il comando. Viene invece riutilizzata la cache predefinita per il `dotnet restore` livello corrispondente. Per ulteriori informazioni, vedere [procedure consigliate per la scrittura di dockerfile](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#leverage-build-cache).

::: moniker-end

::: moniker range="< aspnetcore-3.0"

```dockerfile
FROM mcr.microsoft.com/dotnet/core/aspnet:2.2 AS runtime
WORKDIR /app
COPY published/aspnetapp.dll ./
ENTRYPOINT ["dotnet", "aspnetapp.dll"]
```

### <a name="the-dockerfile"></a>Dockerfile

Ecco il *Dockerfile* usato dal `docker build` comando che è stato eseguito in precedenza. Usa `dotnet publish` allo stesso modo in cui è stato usato in questa sezione per la compilazione e la distribuzione.  

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

## <a name="additional-resources"></a>Risorse aggiuntive

* [Comando build di Docker](https://docs.docker.com/engine/reference/commandline/build)
* [Comando run di Docker](https://docs.docker.com/engine/reference/commandline/run)
* [Esempio di Docker per ASP.NET Core](https://github.com/dotnet/dotnet-docker) (usato in questa esercitazione)
* [Configurare ASP.NET Core per l'utilizzo di server proxy e servizi di bilanciamento del carico](../proxy-load-balancer.md)
* [Working with Visual Studio Docker Tools](./visual-studio-tools-for-docker.md) (Uso degli strumenti Docker per Visual Studio)
* [Debug con Visual Studio Code](https://code.visualstudio.com/docs/nodejs/debugging-recipes#_debug-nodejs-in-docker-containers)
* [GC con contenitori Docker e Small](xref:performance/memory#sc)

## <a name="next-steps"></a>Passaggi successivi

Il repository Git che contiene l'app di esempio include anche la documentazione. Per una panoramica delle risorse disponibili nel repository, vedere il [file LEGGIMI](https://github.com/dotnet/dotnet-docker/blob/main/samples/aspnetapp/README.md). In particolare, vedere le informazioni su come implementare HTTPS:

> [!div class="nextstepaction"]
> [Developing ASP.NET Core Applications with Docker over HTTPS](https://github.com/dotnet/dotnet-docker/blob/main/samples/run-aspnetcore-https-development.md) (Sviluppo di applicazioni ASP.NET Core con Docker su HTTPS)
