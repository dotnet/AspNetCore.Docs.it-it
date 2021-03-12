---
title: Strumenti contenitore di Visual Studio con ASP.NET Core
author: spboyer
description: Informazioni su come usare gli strumenti di Visual Studio e Docker per Windows per aggiungere un'app ASP.NET Core in un contenitore.
ms.author: scaddie
ms.custom: mvc
ms.date: 09/12/2018
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
uid: host-and-deploy/docker/visual-studio-tools-for-docker
ms.openlocfilehash: c11261fef84234a9caaf872ae9871932d02617fa
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102585994"
---
# <a name="visual-studio-container-tools-with-aspnet-core"></a>Strumenti contenitore di Visual Studio con ASP.NET Core

Visual Studio 2017 e versioni successive supporta la compilazione, il debug e l'esecuzione di app ASP.NET Core incluse in contenitori destinate a .NET Core. Sono supportati sia contenitori Windows che contenitori Linux.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/host-and-deploy/docker/visual-studio-tools-for-docker/samples) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="prerequisites"></a>Prerequisiti

* [Docker per Windows](https://docs.docker.com/docker-for-windows/install/)
* [Visual Studio 2019](https://visualstudio.microsoft.com/downloads/?utm_medium=microsoft&utm_source=docs.microsoft.com&utm_campaign=inline+link&utm_content=download+vs2019) con il carico di lavoro **Sviluppo multipiattaforma .NET Core**

## <a name="installation-and-setup"></a>Installazione e configurazione

Per l'installazione di Docker, rivedere prima le informazioni riportate in [Docker for Windows: What to know before you install](https://docs.docker.com/docker-for-windows/install/#what-to-know-before-you-install) (Docker per Windows: informazioni da conoscere prima dell'installazione). Installare quindi [Docker per Windows](https://docs.docker.com/docker-for-windows/install/).

**[Le unità condivise](https://docs.docker.com/docker-for-windows/#shared-drives)** in Docker per Windows devono essere configurate per supportare il mapping e il debug del volume. Fare clic con il pulsante destro del mouse sull'icona di Docker sulla barra delle applicazioni, selezionare **Settings** (Impostazioni) e quindi selezionare **Shared Drives** (Unità condivise). Selezionare l'unità in cui Docker archivia i file. Fare clic su **Applica**.

![Finestra di dialogo per la selezione della condivisione dell'unità C locale per i contenitori](visual-studio-tools-for-docker/_static/settings-shared-drives-win.png)

> [!TIP]
> Visual Studio 2017 15.6 e versioni successive visualizzano un messaggio quando l'opzione **Shared Drives** (Unità condivise) non è configurata.

## <a name="add-a-project-to-a-docker-container"></a>Aggiungere un progetto in un contenitore Docker

Per inserire in un contenitore un progetto ASP.NET Core, il progetto deve avere come destinazione .NET Core. Sono supportati i contenitori Linux e Windows.

Quando si aggiunge il supporto di Docker a un progetto, scegliere un contenitore Windows o Linux. L'host Docker deve eseguire lo stesso tipo di contenitore. Per modificare il tipo di contenitore nell'istanza di Docker in esecuzione, fare clic con il pulsante destro del mouse sull'icona di Docker sulla barra delle applicazioni e scegliere **Switch to Windows containers** (Passa ai contenitori Windows) o **Switch to Linux containers** (Passa ai contenitori Linux).

### <a name="new-app"></a>Nuova app

Quando si crea una nuova app con i modelli di progetto **Applicazione Web ASP.NET Core** selezionare la casella di controllo **Abilita supporto Docker**:

![Casella di controllo Abilita supporto Docker](visual-studio-tools-for-docker/_static/enable-docker-support-check-box.png)

Se il framework di destinazione è .NET Core, l'elenco a discesa **Sistema operativo** consente la selezione di un tipo di contenitore.

### <a name="existing-app"></a>App esistente

Per i progetti ASP.NET Core destinati a .NET Core, esistono due opzioni per l'aggiunta del supporto di Docker tramite gli strumenti. Aprire il progetto in Visual Studio e scegliere una delle opzioni seguenti:

* Scegliere **Supporto Docker** dal menu **Progetto**.
* Fare clic con il pulsante destro del mouse sul progetto in **Esplora soluzioni** e scegliere **Aggiungi**  >  **supporto Docker**.

Gli strumenti contenitore in Visual Studio non supportano l'aggiunta di Docker a un progetto di ASP.NET Core esistente destinato a .NET Framework.

## <a name="dockerfile-overview"></a>Panoramica di Dockerfile

Alla radice del progetto viene aggiunto un *Dockerfile*, ovvero il file recipe per la creazione di un'immagine Docker finale. Per informazioni sui comandi al suo interno, vedere la Guida di [riferimento a Dockerfile](https://docs.docker.com/engine/reference/builder/) . Questo *Dockerfile* specifico usa una [compilazione in più fasi](https://docs.docker.com/engine/userguide/eng-image/multistage-build/) suddivisa in quattro fasi di compilazione distinte e denominate:

::: moniker range=">= aspnetcore-2.1"

[!code-dockerfile[](visual-studio-tools-for-docker/samples/2.1/HelloDockerTools/Dockerfile.original?highlight=1,6,14,17)]

Il *Dockerfile* precedente si basa sull'immagine [microsoft/dotnet](https://hub.docker.com/r/microsoft/dotnet/). Questa immagine di base include il runtime di ASP.NET Core e pacchetti NuGet. I pacchetti sono compilati tramite JIT per ottenere prestazioni di avvio migliori.

Se la casella di controllo **Configura per HTTPS** della finestra di dialogo Nuovo progetto è selezionata, il *Dockerfile* espone due porte. Una porta viene usata per il traffico HTTP e l'altra viene usata per il traffico HTTPS. Se la casella di controllo non è selezionata, viene esposta una sola porta (80) per il traffico HTTP.

::: moniker-end

::: moniker range="<= aspnetcore-2.0"

[!code-dockerfile[](visual-studio-tools-for-docker/samples/2.0/HelloDockerTools/Dockerfile?highlight=1,5,13,16)]

Il *Dockerfile* precedente si basa sull'immagine [microsoft/aspnetcore](https://hub.docker.com/r/microsoft/aspnetcore/). Questa immagine di base include i pacchetti NuGet ASP.NET Core che sono stati compilati tramite JIT per ottenere prestazioni di avvio migliori.

::: moniker-end

## <a name="add-container-orchestrator-support-to-an-app"></a>Aggiungere il supporto di un agente di orchestrazione a un'app

Visual Studio 2017 15.7 e versioni precedenti supporta [Docker Compose](https://docs.docker.com/compose/overview/) come unica soluzione di orchestrazione dei contenitori. Gli elementi del Docker compose vengono aggiunti tramite il supporto di **Aggiungi**  >  **Docker**.

Visual Studio 2017 versione 15.8 e successive aggiunge una soluzione di orchestrazione solo quando specificamente richiesto. Fare clic con il pulsante destro del mouse su **Esplora soluzioni** e selezionare **Aggiungi** > **Container Orchestrator Support** (Supporto agente di orchestrazione contenitori). Sono disponibili le opzioni seguenti: 

* [Docker Compose](#docker-compose)
* [Service Fabric](#service-fabric)
* [Kubernetes/Helm ](https://helm.sh/)

### <a name="docker-compose"></a>Modello di Docker Compose

Gli strumenti contenitore in Visual Studio aggiungono un progetto *docker-compose* alla soluzione con i file seguenti:

* *Docker-compose. dcproj*: il file che rappresenta il progetto. Include un elemento `<DockerTargetOS>` che specifica il sistema operativo da usare.
* *. dockerignore*: elenca i modelli di file e directory da escludere durante la generazione di un contesto di compilazione.
* *Docker-compose. yml*: il file di base [Docker compose](https://docs.docker.com/compose/overview/) usato per definire la raccolta di immagini compilate ed eseguite `docker-compose build` rispettivamente con e `docker-compose run` .
* *Docker-compose. override. yml*: file facoltativo, letto da Docker compose, con override della configurazione per i servizi. Visual Studio esegue `docker-compose -f "docker-compose.yml" -f "docker-compose.override.yml"` per unire questi file.

Il file *docker-compose.yml* fa riferimento al nome dell'immagine creata quando il progetto viene eseguito:

[!code-yaml[](visual-studio-tools-for-docker/samples/2.0/docker-compose.yml?highlight=5)]

Nell'esempio precedente, `image: hellodockertools` genera l'immagine `hellodockertools:dev` quando l'app viene eseguita in modalità **Debug**. L'immagine `hellodockertools:latest` viene generata quando l'app viene eseguita in modalità **Versione**.

Se viene effettuato il push dell'immagine nel registro, anteporre il nome utente dell'[hub Docker](https://hub.docker.com/) al nome dell'immagine, ad esempio `dockerhubusername/hellodockertools`. In alternativa, modificare il nome dell'immagine per includere l'URL del registro privato (ad esempio, `privateregistry.domain.com/hellodockertools`) a seconda della configurazione.

Se si vuole un comportamento diverso in base alla configurazione della build (ad esempio, debug o versione), aggiungere i file *docker-compose* specifici della configurazione. I file devono essere denominati in base alla configurazione della build (ad esempio, *docker-compose.vs.debug.yml* e *docker-compose.vs.release.yml*) e inseriti nella stessa posizione del file *docker-compose-override.yml*. 

Usando i file di override specifici della configurazione, è possibile specificare impostazioni di configurazione diverse (ad esempio, variabili di ambiente o punti di ingresso) per le configurazioni delle build di debug e di versione.

Per Docker Compose visualizzare un'opzione per l'esecuzione in Visual Studio, il progetto Docker deve essere il progetto di avvio.

### <a name="service-fabric"></a>Service Fabric

Oltre ai [prerequisiti](#prerequisites) di base, la soluzione di orchestrazione [Service Fabric](/azure/service-fabric/) richiede i prerequisiti seguenti:

* [Microsoft Azure Service Fabric SDK](https://www.microsoft.com/web/handlers/webpi.ashx?command=getinstallerredirect&appid=MicrosoftAzure-ServiceFabric-CoreSDK) versione 2.6 o successiva
* Carico di lavoro **Sviluppo di Azure** di Visual Studio

Service Fabric non supporta l'esecuzione di contenitori Linux nel cluster di sviluppo locale in Windows. Se il progetto usa già un contenitore Linux, Visual Studio richiede di passare a contenitori Windows.

Gli strumenti contenitore in Visual Studio eseguono le attività seguenti:

* Aggiunge un' *&lt; &gt; applicazione PROJECT_NAME* Service Fabric progetto di **applicazione** alla soluzione.
* Aggiunge un *Dockerfile* e un file con estensione *dockerignore* al progetto ASP.NET Core. Se nel progetto ASP.NET Core è già presente un *Dockerfile*, questo viene rinominato in *Dockerfile.original*. Viene creato un nuovo *Dockerfile* simile al seguente:

    [!code-dockerfile[](visual-studio-tools-for-docker/samples/2.1/HelloDockerTools/Dockerfile)]

* Aggiunge un elemento `<IsServiceFabricServiceProject>` al file con estensione *csproj* del progetto ASP.NET Core:

    [!code-xml[](visual-studio-tools-for-docker/samples/2.1/HelloDockerTools/HelloDockerTools.csproj?name=snippet_IsServiceFabricServiceProject)]

* Aggiunge una cartella *PackageRoot* al progetto ASP.NET Core. La cartella contiene il manifesto del servizio e le impostazioni per il nuovo servizio.

Per altre informazioni, vedere [Distribuire un'app .NET in un contenitore Windows in Azure Service Fabric](/azure/service-fabric/service-fabric-host-app-in-a-container).

## <a name="debug"></a>Debug

Selezionare **Docker** nell'elenco a discesa Debug nella barra degli strumenti e avviare il debug dell'app. La visualizzazione **Docker** della finestra **Output** mostra le azioni seguenti:

::: moniker range=">= aspnetcore-2.1"

* Viene acquisito il tag *2.1-aspnetcore-runtime* dell'immagine di runtime *microsoft/dotnet*, se non è già presente nella cache. L'immagine installa i runtime di ASP.NET Core e di .NET Core, nonché le librerie associate. È ottimizzata per l'esecuzione di app ASP.NET Core in un ambiente di produzione.
* La variabile di ambiente `ASPNETCORE_ENVIRONMENT` viene impostata su `Development` all'interno del contenitore.
* Vengono esposte due porte assegnate dinamicamente: una per il protocollo HTTP e una per il protocollo HTTPS. È possibile eseguire query sulla porta assegnata a localhost con il comando `docker ps`.
* L'app viene copiata nel contenitore.
* Usando la porta assegnata in modo dinamico, viene avviato il browser predefinito con il debugger collegato al contenitore.

L'immagine Docker risultante dell'app, contrassegnata con *dev*, è basata sul tag *2.1-aspnetcore-runtime* dell'immagine di base *microsoft/dotnet*. Eseguire il comando `docker images` nella finestra **Console di Gestione pacchetti**. Vengono visualizzate le immagini nel computer in uso:

```console
REPOSITORY        TAG                     IMAGE ID      CREATED         SIZE
hellodockertools  dev                     d72ce0f1dfe7  30 seconds ago  255MB
microsoft/dotnet  2.1-aspnetcore-runtime  fcc3887985bb  6 days ago      255MB
```

::: moniker-end

::: moniker range="<= aspnetcore-2.0"

* Se non è già presente nella cache, viene acquisita l'immagine di runtime *microsoft/aspnetcore*.
* La variabile di ambiente `ASPNETCORE_ENVIRONMENT` viene impostata su `Development` all'interno del contenitore.
* La porta 80 viene esposta e mappata a una porta assegnata in modo dinamico per localhost. La porta è determinata dall'host Docker ed è possibile eseguirvi query con il comando `docker ps`.
* L'app viene copiata nel contenitore.
* Usando la porta assegnata in modo dinamico, viene avviato il browser predefinito con il debugger collegato al contenitore.

L'immagine Docker risultante dell'app, contrassegnata con *dev*, è basata sull'immagine di base *microsoft/aspnetcore*. Eseguire il comando `docker images` nella finestra **Console di Gestione pacchetti**. Vengono visualizzate le immagini nel computer in uso:

```console
REPOSITORY            TAG  IMAGE ID      CREATED        SIZE
hellodockertools      dev  5fafe5d1ad5b  4 minutes ago  347MB
microsoft/aspnetcore  2.0  c69d39472da9  13 days ago    347MB
```

::: moniker-end

> [!NOTE]
> Nell'immagine *dev* non è presente il contenuto dell'app, poiché le configurazioni **Debug** usano il montaggio su volume per garantire un'esperienza iterativa. Per eseguire il push di un'immagine, usare la configurazione di **rilascio** .

Eseguire il comando `docker ps` nella console di Gestione pacchetti. Si noti che l'app viene eseguita usando il contenitore:

```console
CONTAINER ID        IMAGE                  COMMAND                   CREATED             STATUS              PORTS                   NAMES
baf9a678c88d        hellodockertools:dev   "C:\\remote_debugge..."   21 seconds ago      Up 19 seconds       0.0.0.0:37630->80/tcp   dockercompose4642749010770307127_hellodockertools_1
```

## <a name="edit-and-continue"></a>Modifica e continuazione

Le modifiche apportate ai file statici e alle Razor viste vengono aggiornate automaticamente senza la necessità di un passaggio di compilazione. Dopo aver apportato una modifica, salvarla e aggiornare il browser per visualizzare l'aggiornamento.

Le modifiche ai file del codice richiedono la compilazione e il riavvio di Kestrel all'interno del contenitore. Dopo aver apportato una modifica, usare `CTRL+F5` per eseguire il processo e avviare l'app all'interno del contenitore. Il contenitore Docker non viene ricompilato o arrestato. Eseguire il comando `docker ps` nella console di Gestione pacchetti. Si noti che il contenitore originale è ancora in esecuzione da 10 minuti:

```console
CONTAINER ID        IMAGE                  COMMAND                   CREATED             STATUS              PORTS                   NAMES
baf9a678c88d        hellodockertools:dev   "C:\\remote_debugge..."   10 minutes ago      Up 10 minutes       0.0.0.0:37630->80/tcp   dockercompose4642749010770307127_hellodockertools_1
```

## <a name="publish-docker-images"></a>Pubblicare immagini Docker

Dopo il completamento del ciclo di sviluppo e debug dell'app, gli strumenti contenitore in Visual Studio consentono di creare l'immagine di produzione dell'app. Selezionare **Versione** nell'elenco a discesa della configurazione ed eseguire l'app. Gli strumenti acquisiscono l'immagine di compilazione/pubblicazione dall'hub Docker, se non è già presente nella cache. Viene generata un'immagine con il tag *latest*. È possibile effettuare il push dell'immagine nel registro privato o nell'hub Docker.

Eseguire il comando `docker images` nella Console di Gestione pacchetti per visualizzare l'elenco delle immagini. Viene visualizzato output simile al seguente:

::: moniker range=">= aspnetcore-2.1"

```console
REPOSITORY        TAG                     IMAGE ID      CREATED             SIZE
hellodockertools  latest                  e3984a64230c  About a minute ago  258MB
hellodockertools  dev                     d72ce0f1dfe7  4 minutes ago       255MB
microsoft/dotnet  2.1-sdk                 9e243db15f91  6 days ago          1.7GB
microsoft/dotnet  2.1-aspnetcore-runtime  fcc3887985bb  6 days ago          255MB
```

::: moniker-end

::: moniker range="<= aspnetcore-2.0"

```console
REPOSITORY                  TAG     IMAGE ID      CREATED         SIZE
hellodockertools            latest  cd28f0d4abbd  12 seconds ago  349MB
hellodockertools            dev     5fafe5d1ad5b  23 minutes ago  347MB
microsoft/aspnetcore-build  2.0     7fed40fbb647  13 days ago     2.02GB
microsoft/aspnetcore        2.0     c69d39472da9  13 days ago     347MB
```

Le immagini `microsoft/aspnetcore-build` e `microsoft/aspnetcore` indicate nell'output precedente vengono sostituite da immagini `microsoft/dotnet` a partire da .NET Core 2.1. Per altre informazioni, vedere [l'annuncio della migrazione dei repository Docker](https://github.com/aspnet/Announcements/issues/298).

::: moniker-end

> [!NOTE]
> Il `docker images` comando restituisce le immagini intermedie con i nomi dei repository e i tag identificati come *\<none>* (non elencati in precedenza). Queste immagini senza nome vengono generate dalla compilazione in più [fasi](https://docs.docker.com/engine/userguide/eng-image/multistage-build/) *Dockerfile*. L'efficienza della creazione dell'immagine finale risulta migliorata, dato che vengono ricompilati solo i livelli necessari in seguito a modifiche. Quando le immagini intermedie non sono più necessarie, eliminarle usando il comando [docker rmi](https://docs.docker.com/engine/reference/commandline/rmi/).

Ci si potrebbe aspettare che l'immagine di produzione o di versione abbia dimensioni minori rispetto all'immagine *dev*. A causa del mapping del volume, il debugger e l'app sono stati eseguiti dal computer locale e non all'interno del contenitore. L'immagine *latest* include il codice dell'app necessario per eseguire l'app in un computer host. Pertanto, il delta è la dimensione del codice dell'app.

## <a name="additional-resources"></a>Risorse aggiuntive

* [Sviluppare con i contenitori in Visual Studio](/visualstudio/containers)
* [Azure Service Fabric: preparare l'ambiente di sviluppo](/azure/service-fabric/service-fabric-get-started)
* [Distribuire un'app .NET in un contenitore Windows in Azure Service Fabric](/azure/service-fabric/service-fabric-host-app-in-a-container)
* [Risolvere i problemi di sviluppo di Visual Studio con Docker](/azure/vs-azure-tools-docker-troubleshooting-docker-errors)
* [Repository GitHub degli strumenti contenitore di Visual Studio](https://github.com/Microsoft/DockerTools)
* [GC con contenitori Docker e Small](xref:performance/memory#sc)
