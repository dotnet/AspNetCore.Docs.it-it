---
title: Hosting di ASP.NET Core immagine nel contenitore con Docker compose con HTTPS
author: ravipal
description: Informazioni su come ospitare ASP.NET Core immagini con Docker Compose su HTTPS
monikerRange: '>= aspnetcore-2.1'
ms.author: ravipal
ms.custom: mvc
ms.date: 03/28/2020
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
uid: security/docker-compose-https
ms.openlocfilehash: 69c449527a64ec928d436245e9e971ec49303489
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102588757"
---
# <a name="hosting-aspnet-core-images-with-docker-compose-over-https"></a><span data-ttu-id="1c7b0-103">Hosting di immagini ASP.NET Core con Docker Compose su HTTPS</span><span class="sxs-lookup"><span data-stu-id="1c7b0-103">Hosting ASP.NET Core images with Docker Compose over HTTPS</span></span>


<span data-ttu-id="1c7b0-104">ASP.NET Core utilizza [https per impostazione predefinita](./enforcing-ssl.md).</span><span class="sxs-lookup"><span data-stu-id="1c7b0-104">ASP.NET Core uses [HTTPS by default](./enforcing-ssl.md).</span></span> <span data-ttu-id="1c7b0-105">[Https](https://en.wikipedia.org/wiki/HTTPS) si basa su [certificati](https://en.wikipedia.org/wiki/Public_key_certificate) per l'attendibilità, l'identità e la crittografia.</span><span class="sxs-lookup"><span data-stu-id="1c7b0-105">[HTTPS](https://en.wikipedia.org/wiki/HTTPS) relies on [certificates](https://en.wikipedia.org/wiki/Public_key_certificate) for trust, identity, and encryption.</span></span>

<span data-ttu-id="1c7b0-106">Questo documento illustra come eseguire le immagini del contenitore predefinite con HTTPS.</span><span class="sxs-lookup"><span data-stu-id="1c7b0-106">This document explains how to run pre-built container images with HTTPS.</span></span>

<span data-ttu-id="1c7b0-107">Per gli scenari di sviluppo, vedere [sviluppo di applicazioni ASP.NET Core con Docker su HTTPS](https://github.com/dotnet/dotnet-docker/blob/main/samples/run-aspnetcore-https-development.md) .</span><span class="sxs-lookup"><span data-stu-id="1c7b0-107">See [Developing ASP.NET Core Applications with Docker over HTTPS](https://github.com/dotnet/dotnet-docker/blob/main/samples/run-aspnetcore-https-development.md) for development scenarios.</span></span>

<span data-ttu-id="1c7b0-108">Questo esempio richiede [docker 17,06](https://docs.docker.com/release-notes/docker-ce) o versione successiva del [client Docker](https://www.docker.com/products/docker).</span><span class="sxs-lookup"><span data-stu-id="1c7b0-108">This sample requires [Docker 17.06](https://docs.docker.com/release-notes/docker-ce) or later of the [Docker client](https://www.docker.com/products/docker).</span></span>

## <a name="prerequisites"></a><span data-ttu-id="1c7b0-109">Prerequisiti</span><span class="sxs-lookup"><span data-stu-id="1c7b0-109">Prerequisites</span></span>

<span data-ttu-id="1c7b0-110">Per alcune istruzioni di questo documento è necessario [.NET Core 2,2 SDK](https://dotnet.microsoft.com/download) o versione successiva.</span><span class="sxs-lookup"><span data-stu-id="1c7b0-110">The [.NET Core 2.2 SDK](https://dotnet.microsoft.com/download) or later is required for some of the instructions in this document.</span></span>

## <a name="certificates"></a><span data-ttu-id="1c7b0-111">Certificati</span><span class="sxs-lookup"><span data-stu-id="1c7b0-111">Certificates</span></span>

<span data-ttu-id="1c7b0-112">Un certificato di un' [autorità di certificazione](https://wikipedia.org/wiki/Certificate_authority) è necessario per l' [hosting di produzione](https://blogs.msdn.microsoft.com/webdev/2017/11/29/configuring-https-in-asp-net-core-across-different-platforms/) per un dominio.</span><span class="sxs-lookup"><span data-stu-id="1c7b0-112">A certificate from a [certificate authority](https://wikipedia.org/wiki/Certificate_authority) is required for [production hosting](https://blogs.msdn.microsoft.com/webdev/2017/11/29/configuring-https-in-asp-net-core-across-different-platforms/) for a domain.</span></span> <span data-ttu-id="1c7b0-113">[Let's Encrypt](https://letsencrypt.org/) è un'autorità di certificazione che offre certificati gratuiti.</span><span class="sxs-lookup"><span data-stu-id="1c7b0-113">[Let's Encrypt](https://letsencrypt.org/) is a certificate authority that offers free certificates.</span></span>

<span data-ttu-id="1c7b0-114">Questo documento usa [certificati di sviluppo autofirmati](https://wikipedia.org/wiki/Self-signed_certificate) per l'hosting di immagini predefinite rispetto a `localhost` .</span><span class="sxs-lookup"><span data-stu-id="1c7b0-114">This document uses [self-signed development certificates](https://wikipedia.org/wiki/Self-signed_certificate) for hosting pre-built images over `localhost`.</span></span> <span data-ttu-id="1c7b0-115">Le istruzioni sono simili all'uso dei certificati di produzione.</span><span class="sxs-lookup"><span data-stu-id="1c7b0-115">The instructions are similar to using production certificates.</span></span>

<span data-ttu-id="1c7b0-116">Per i certificati di produzione:</span><span class="sxs-lookup"><span data-stu-id="1c7b0-116">For production certificates:</span></span>

* <span data-ttu-id="1c7b0-117">Lo `dotnet dev-certs` strumento non è obbligatorio.</span><span class="sxs-lookup"><span data-stu-id="1c7b0-117">The `dotnet dev-certs` tool is not required.</span></span>
* <span data-ttu-id="1c7b0-118">I certificati non devono essere archiviati nel percorso utilizzato nelle istruzioni.</span><span class="sxs-lookup"><span data-stu-id="1c7b0-118">Certificates don't need to be stored in the location used in the instructions.</span></span> <span data-ttu-id="1c7b0-119">Archiviare i certificati in qualsiasi posizione all'esterno della directory del sito.</span><span class="sxs-lookup"><span data-stu-id="1c7b0-119">Store the certificates in any location outside the site directory.</span></span>

<span data-ttu-id="1c7b0-120">Le istruzioni contenute nella sezione seguente comportano i certificati di montaggio nei contenitori usando la `volumes` Proprietà in *Docker-compose. yml.*</span><span class="sxs-lookup"><span data-stu-id="1c7b0-120">The instructions contained in the following section volume mount certificates into containers using the `volumes` property in *docker-compose.yml.*</span></span> <span data-ttu-id="1c7b0-121">È possibile aggiungere certificati alle immagini del contenitore con un `COPY` comando in un *Dockerfile*, ma non è consigliabile.</span><span class="sxs-lookup"><span data-stu-id="1c7b0-121">You could add certificates into container images with a `COPY` command in a *Dockerfile*, but it's not recommended.</span></span> <span data-ttu-id="1c7b0-122">La copia di certificati in un'immagine non è consigliata per i motivi seguenti:</span><span class="sxs-lookup"><span data-stu-id="1c7b0-122">Copying certificates into an image isn't recommended for the following reasons:</span></span>

* <span data-ttu-id="1c7b0-123">Risulta difficile usare la stessa immagine per i test con certificati per sviluppatori.</span><span class="sxs-lookup"><span data-stu-id="1c7b0-123">It makes it difficult to use the same image for testing with developer certificates.</span></span>
* <span data-ttu-id="1c7b0-124">Rende difficile usare la stessa immagine per l'hosting con certificati di produzione.</span><span class="sxs-lookup"><span data-stu-id="1c7b0-124">It makes it difficult to use the same image for Hosting with production certificates.</span></span>
* <span data-ttu-id="1c7b0-125">Il rischio di divulgazione dei certificati è significativo.</span><span class="sxs-lookup"><span data-stu-id="1c7b0-125">There is significant risk of certificate disclosure.</span></span>

## <a name="starting-a-container-with-https-support-using-docker-compose"></a><span data-ttu-id="1c7b0-126">Avvio di un contenitore con supporto HTTPS con Docker compose</span><span class="sxs-lookup"><span data-stu-id="1c7b0-126">Starting a container with https support using docker compose</span></span>

<span data-ttu-id="1c7b0-127">Usare le istruzioni seguenti per la configurazione del sistema operativo.</span><span class="sxs-lookup"><span data-stu-id="1c7b0-127">Use the following instructions for your operating system configuration.</span></span>

### <a name="windows-using-linux-containers"></a><span data-ttu-id="1c7b0-128">Windows con contenitori Linux</span><span class="sxs-lookup"><span data-stu-id="1c7b0-128">Windows using Linux containers</span></span>

<span data-ttu-id="1c7b0-129">Generare il certificato e configurare il computer locale:</span><span class="sxs-lookup"><span data-stu-id="1c7b0-129">Generate certificate and configure local machine:</span></span>

```dotnetcli
dotnet dev-certs https -ep %USERPROFILE%\.aspnet\https\aspnetapp.pfx -p { password here }
dotnet dev-certs https --trust
```

<span data-ttu-id="1c7b0-130">Nei comandi precedenti sostituire `{ password here }` con una password.</span><span class="sxs-lookup"><span data-stu-id="1c7b0-130">In the preceding commands, replace `{ password here }` with a password.</span></span>

<span data-ttu-id="1c7b0-131">Creare un file _Docker-compose. debug. yml_ con il contenuto seguente:</span><span class="sxs-lookup"><span data-stu-id="1c7b0-131">Create a _docker-compose.debug.yml_ file with the following content:</span></span>

```yaml
version: '3.4'

services:
  webapp:
    image: mcr.microsoft.com/dotnet/core/samples:aspnetapp
    ports:
      - 80
      - 443
    environment:
      - ASPNETCORE_ENVIRONMENT=Development
      - ASPNETCORE_URLS=https://+:443;http://+:80
      - ASPNETCORE_Kestrel__Certificates__Default__Password=password
      - ASPNETCORE_Kestrel__Certificates__Default__Path=/https/aspnetapp.pfx
    volumes:
      - ~/.aspnet/https:/https:ro
```
<span data-ttu-id="1c7b0-132">La password specificata nel file Docker compose deve corrispondere alla password usata per il certificato.</span><span class="sxs-lookup"><span data-stu-id="1c7b0-132">The password specified in the docker compose file must match the password used for the certificate.</span></span>

<span data-ttu-id="1c7b0-133">Avviare il contenitore con ASP.NET Core configurato per HTTPS:</span><span class="sxs-lookup"><span data-stu-id="1c7b0-133">Start the container with ASP.NET Core configured for HTTPS:</span></span>

```console
docker-compose -f "docker-compose.debug.yml" up -d
```

### <a name="macos-or-linux"></a><span data-ttu-id="1c7b0-134">macOS o Linux</span><span class="sxs-lookup"><span data-stu-id="1c7b0-134">macOS or Linux</span></span>

<span data-ttu-id="1c7b0-135">Generare il certificato e configurare il computer locale:</span><span class="sxs-lookup"><span data-stu-id="1c7b0-135">Generate certificate and configure local machine:</span></span>

```dotnetcli
dotnet dev-certs https -ep ${HOME}/.aspnet/https/aspnetapp.pfx -p { password here }
dotnet dev-certs https --trust
```

<span data-ttu-id="1c7b0-136">`dotnet dev-certs https --trust` è supportato solo in macOS e Windows.</span><span class="sxs-lookup"><span data-stu-id="1c7b0-136">`dotnet dev-certs https --trust` is only supported on macOS and Windows.</span></span> <span data-ttu-id="1c7b0-137">È necessario considerare attendibili i certificati in Linux nel modo supportato dalla distribuzione.</span><span class="sxs-lookup"><span data-stu-id="1c7b0-137">You need to trust certificates on Linux in the way that is supported by your distribution.</span></span> <span data-ttu-id="1c7b0-138">È probabile che sia necessario considerare attendibile il certificato nel browser.</span><span class="sxs-lookup"><span data-stu-id="1c7b0-138">It is likely that you need to trust the certificate in your browser.</span></span>

<span data-ttu-id="1c7b0-139">Nei comandi precedenti sostituire `{ password here }` con una password.</span><span class="sxs-lookup"><span data-stu-id="1c7b0-139">In the preceding commands, replace `{ password here }` with a password.</span></span>

<span data-ttu-id="1c7b0-140">Creare un file _Docker-compose. debug. yml_ con il contenuto seguente:</span><span class="sxs-lookup"><span data-stu-id="1c7b0-140">Create a _docker-compose.debug.yml_ file with the following content:</span></span>

```yaml
version: '3.4'

services:
  webapp:
    image: mcr.microsoft.com/dotnet/core/samples:aspnetapp
    ports:
      - 80
      - 443
    environment:
      - ASPNETCORE_ENVIRONMENT=Development
      - ASPNETCORE_URLS=https://+:443;http://+:80
      - ASPNETCORE_Kestrel__Certificates__Default__Password=password
      - ASPNETCORE_Kestrel__Certificates__Default__Path=/https/aspnetapp.pfx
    volumes:
      - ~/.aspnet/https:/https:ro
```
<span data-ttu-id="1c7b0-141">La password specificata nel file Docker compose deve corrispondere alla password usata per il certificato.</span><span class="sxs-lookup"><span data-stu-id="1c7b0-141">The password specified in the docker compose file must match the password used for the certificate.</span></span>

<span data-ttu-id="1c7b0-142">Avviare il contenitore con ASP.NET Core configurato per HTTPS:</span><span class="sxs-lookup"><span data-stu-id="1c7b0-142">Start the container with ASP.NET Core configured for HTTPS:</span></span>

```console
docker-compose -f "docker-compose.debug.yml" up -d
```

### <a name="windows-using-windows-containers"></a><span data-ttu-id="1c7b0-143">Windows con i contenitori di Windows</span><span class="sxs-lookup"><span data-stu-id="1c7b0-143">Windows using Windows containers</span></span>

<span data-ttu-id="1c7b0-144">Generare il certificato e configurare il computer locale:</span><span class="sxs-lookup"><span data-stu-id="1c7b0-144">Generate certificate and configure local machine:</span></span>

```dotnetcli
dotnet dev-certs https -ep %USERPROFILE%\.aspnet\https\aspnetapp.pfx -p { password here }
dotnet dev-certs https --trust
```

<span data-ttu-id="1c7b0-145">Nei comandi precedenti sostituire `{ password here }` con una password.</span><span class="sxs-lookup"><span data-stu-id="1c7b0-145">In the preceding commands, replace `{ password here }` with a password.</span></span>

<span data-ttu-id="1c7b0-146">Creare un file _Docker-compose. debug. yml_ con il contenuto seguente:</span><span class="sxs-lookup"><span data-stu-id="1c7b0-146">Create a _docker-compose.debug.yml_ file with the following content:</span></span>

```yaml
version: '3.4'

services:
  webapp:
    image: mcr.microsoft.com/dotnet/core/samples:aspnetapp
    ports:
      - 80
      - 443
    environment:
      - ASPNETCORE_ENVIRONMENT=Development
      - ASPNETCORE_URLS=https://+:443;http://+:80
      - ASPNETCORE_Kestrel__Certificates__Default__Password=password
      - ASPNETCORE_Kestrel__Certificates__Default__Path=C:\https\aspnetapp.pfx
    volumes:
      - ${USERPROFILE}\.aspnet\https:C:\https:ro
```
<span data-ttu-id="1c7b0-147">La password specificata nel file Docker compose deve corrispondere alla password usata per il certificato.</span><span class="sxs-lookup"><span data-stu-id="1c7b0-147">The password specified in the docker compose file must match the password used for the certificate.</span></span>

<span data-ttu-id="1c7b0-148">Avviare il contenitore con ASP.NET Core configurato per HTTPS:</span><span class="sxs-lookup"><span data-stu-id="1c7b0-148">Start the container with ASP.NET Core configured for HTTPS:</span></span>

```console
docker-compose -f "docker-compose.debug.yml" up -d
```
