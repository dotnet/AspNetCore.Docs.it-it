---
title: Hosting di immagini ASP.NET Core con Docker su HTTPS
author: rick-anderson
description: Informazioni su come ospitare ASP.NET Core immagini con Docker su HTTPS
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 07/05/2019
no-loc:
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
uid: security/docker-https
ms.openlocfilehash: 3af2aff477604eb19ac211753f848d08d0c67c72
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102588640"
---
# <a name="hosting-aspnet-core-images-with-docker-over-https"></a><span data-ttu-id="4cf93-103">Hosting di immagini ASP.NET Core con Docker su HTTPS</span><span class="sxs-lookup"><span data-stu-id="4cf93-103">Hosting ASP.NET Core images with Docker over HTTPS</span></span>

<span data-ttu-id="4cf93-104">Autore: [Rick Anderson](https://twitter.com/RickAndMSFT)</span><span class="sxs-lookup"><span data-stu-id="4cf93-104">By [Rick Anderson](https://twitter.com/RickAndMSFT)</span></span>

<span data-ttu-id="4cf93-105">ASP.NET Core utilizza [https per impostazione predefinita](./enforcing-ssl.md).</span><span class="sxs-lookup"><span data-stu-id="4cf93-105">ASP.NET Core uses [HTTPS by default](./enforcing-ssl.md).</span></span> <span data-ttu-id="4cf93-106">[Https](https://en.wikipedia.org/wiki/HTTPS) si basa su [certificati](https://en.wikipedia.org/wiki/Public_key_certificate) per l'attendibilità, l'identità e la crittografia.</span><span class="sxs-lookup"><span data-stu-id="4cf93-106">[HTTPS](https://en.wikipedia.org/wiki/HTTPS) relies on [certificates](https://en.wikipedia.org/wiki/Public_key_certificate) for trust, identity, and encryption.</span></span>

<span data-ttu-id="4cf93-107">Questo documento illustra come eseguire le immagini del contenitore predefinite con HTTPS.</span><span class="sxs-lookup"><span data-stu-id="4cf93-107">This document explains how to run pre-built container images with HTTPS.</span></span>

<span data-ttu-id="4cf93-108">Per gli scenari di sviluppo, vedere [sviluppo di applicazioni ASP.NET Core con Docker su HTTPS](https://github.com/dotnet/dotnet-docker/blob/main/samples/run-aspnetcore-https-development.md) .</span><span class="sxs-lookup"><span data-stu-id="4cf93-108">See [Developing ASP.NET Core Applications with Docker over HTTPS](https://github.com/dotnet/dotnet-docker/blob/main/samples/run-aspnetcore-https-development.md) for development scenarios.</span></span>

<span data-ttu-id="4cf93-109">Questo esempio richiede [docker 17,06](https://docs.docker.com/release-notes/docker-ce) o versione successiva del [client Docker](https://www.docker.com/products/docker).</span><span class="sxs-lookup"><span data-stu-id="4cf93-109">This sample requires [Docker 17.06](https://docs.docker.com/release-notes/docker-ce) or later of the [Docker client](https://www.docker.com/products/docker).</span></span>

## <a name="prerequisites"></a><span data-ttu-id="4cf93-110">Prerequisiti</span><span class="sxs-lookup"><span data-stu-id="4cf93-110">Prerequisites</span></span>

<span data-ttu-id="4cf93-111">Per alcune istruzioni di questo documento è necessario [.NET Core 2,2 SDK](https://dotnet.microsoft.com/download) o versione successiva.</span><span class="sxs-lookup"><span data-stu-id="4cf93-111">The [.NET Core 2.2 SDK](https://dotnet.microsoft.com/download) or later is required for some of the instructions in this document.</span></span>

## <a name="certificates"></a><span data-ttu-id="4cf93-112">Certificati</span><span class="sxs-lookup"><span data-stu-id="4cf93-112">Certificates</span></span>

<span data-ttu-id="4cf93-113">Un certificato di un' [autorità di certificazione](https://wikipedia.org/wiki/Certificate_authority) è necessario per l' [hosting di produzione](https://blogs.msdn.microsoft.com/webdev/2017/11/29/configuring-https-in-asp-net-core-across-different-platforms/) per un dominio.</span><span class="sxs-lookup"><span data-stu-id="4cf93-113">A certificate from a [certificate authority](https://wikipedia.org/wiki/Certificate_authority) is required for [production hosting](https://blogs.msdn.microsoft.com/webdev/2017/11/29/configuring-https-in-asp-net-core-across-different-platforms/) for a domain.</span></span> <span data-ttu-id="4cf93-114">[Let's Encrypt](https://letsencrypt.org/) è un'autorità di certificazione che offre certificati gratuiti.</span><span class="sxs-lookup"><span data-stu-id="4cf93-114">[Let's Encrypt](https://letsencrypt.org/) is a certificate authority that offers free certificates.</span></span>

<span data-ttu-id="4cf93-115">Questo documento usa [certificati di sviluppo autofirmati](https://en.wikipedia.org/wiki/Self-signed_certificate) per l'hosting di immagini predefinite rispetto a `localhost` .</span><span class="sxs-lookup"><span data-stu-id="4cf93-115">This document uses [self-signed development certificates](https://en.wikipedia.org/wiki/Self-signed_certificate) for hosting pre-built images over `localhost`.</span></span> <span data-ttu-id="4cf93-116">Le istruzioni sono simili all'uso dei certificati di produzione.</span><span class="sxs-lookup"><span data-stu-id="4cf93-116">The instructions are similar to using production certificates.</span></span>

<span data-ttu-id="4cf93-117">Usare [DotNet dev-certs](/dotnet/core/additional-tools/self-signed-certificates-guide) per creare certificati autofirmati per lo sviluppo e il test.</span><span class="sxs-lookup"><span data-stu-id="4cf93-117">Use [dotnet dev-certs](/dotnet/core/additional-tools/self-signed-certificates-guide) to create self-signed certificates for development and testing.</span></span>

<span data-ttu-id="4cf93-118">Per i certificati di produzione:</span><span class="sxs-lookup"><span data-stu-id="4cf93-118">For production certs:</span></span>

* <span data-ttu-id="4cf93-119">Lo `dotnet dev-certs` strumento non è obbligatorio.</span><span class="sxs-lookup"><span data-stu-id="4cf93-119">The `dotnet dev-certs` tool is not required.</span></span>
* <span data-ttu-id="4cf93-120">Non è necessario archiviare i certificati nel percorso utilizzato nelle istruzioni.</span><span class="sxs-lookup"><span data-stu-id="4cf93-120">Certificates do not need to be stored in the location used in the instructions.</span></span> <span data-ttu-id="4cf93-121">Qualsiasi località dovrebbe funzionare, anche se non è consigliabile archiviare i certificati nella directory del sito.</span><span class="sxs-lookup"><span data-stu-id="4cf93-121">Any location should work, although storing certs within your site directory is not recommended.</span></span>

<span data-ttu-id="4cf93-122">Le istruzioni contenute nella sezione seguente comportano i certificati di montaggio nei contenitori con l'opzione della riga di comando di Docker `-v` .</span><span class="sxs-lookup"><span data-stu-id="4cf93-122">The instructions contained in the following section volume mount certificates into containers using Docker's `-v` command-line option.</span></span> <span data-ttu-id="4cf93-123">È possibile aggiungere certificati alle immagini del contenitore con un `COPY` comando in un *Dockerfile*, ma non è consigliabile.</span><span class="sxs-lookup"><span data-stu-id="4cf93-123">You could add certificates into container images with a `COPY` command in a *Dockerfile*, but it's not recommended.</span></span> <span data-ttu-id="4cf93-124">La copia di certificati in un'immagine non è consigliata per i motivi seguenti:</span><span class="sxs-lookup"><span data-stu-id="4cf93-124">Copying certificates into an image isn't recommended for the following reasons:</span></span>

* <span data-ttu-id="4cf93-125">Risulta difficile usare la stessa immagine per i test con certificati per sviluppatori.</span><span class="sxs-lookup"><span data-stu-id="4cf93-125">It makes difficult to use the same image for testing with developer certificates.</span></span>
* <span data-ttu-id="4cf93-126">Risulta difficile utilizzare la stessa immagine per l'hosting con certificati di produzione.</span><span class="sxs-lookup"><span data-stu-id="4cf93-126">It makes difficult to use the same image for Hosting with production certificates.</span></span>
* <span data-ttu-id="4cf93-127">Il rischio di divulgazione dei certificati è significativo.</span><span class="sxs-lookup"><span data-stu-id="4cf93-127">There is significant risk of certificate disclosure.</span></span>

## <a name="running-pre-built-container-images-with-https"></a><span data-ttu-id="4cf93-128">Esecuzione di immagini del contenitore predefinite con HTTPS</span><span class="sxs-lookup"><span data-stu-id="4cf93-128">Running pre-built container images with HTTPS</span></span>

<span data-ttu-id="4cf93-129">Usare le istruzioni seguenti per la configurazione del sistema operativo.</span><span class="sxs-lookup"><span data-stu-id="4cf93-129">Use the following instructions for your operating system configuration.</span></span>

### <a name="windows-using-linux-containers"></a><span data-ttu-id="4cf93-130">Windows con contenitori Linux</span><span class="sxs-lookup"><span data-stu-id="4cf93-130">Windows using Linux containers</span></span>

<span data-ttu-id="4cf93-131">Generare il certificato e configurare il computer locale:</span><span class="sxs-lookup"><span data-stu-id="4cf93-131">Generate certificate and configure local machine:</span></span>

```dotnetcli
dotnet dev-certs https -ep %USERPROFILE%\.aspnet\https\aspnetapp.pfx -p { password here }
dotnet dev-certs https --trust
```

<span data-ttu-id="4cf93-132">Nei comandi precedenti sostituire `{ password here }` con una password.</span><span class="sxs-lookup"><span data-stu-id="4cf93-132">In the preceding commands, replace `{ password here }` with a password.</span></span>

<span data-ttu-id="4cf93-133">Eseguire l'immagine del contenitore con ASP.NET Core configurato per HTTPS in una shell dei comandi:</span><span class="sxs-lookup"><span data-stu-id="4cf93-133">Run the container image with ASP.NET Core configured for HTTPS in a command shell:</span></span>

```console
docker pull mcr.microsoft.com/dotnet/core/samples:aspnetapp
docker run --rm -it -p 8000:80 -p 8001:443 -e ASPNETCORE_URLS="https://+;http://+" -e ASPNETCORE_HTTPS_PORT=8001 -e ASPNETCORE_Kestrel__Certificates__Default__Password="password" -e ASPNETCORE_Kestrel__Certificates__Default__Path=/https/aspnetapp.pfx -v %USERPROFILE%\.aspnet\https:/https/ mcr.microsoft.com/dotnet/core/samples:aspnetapp
```

<span data-ttu-id="4cf93-134">Quando si usa [PowerShell](/powershell/scripting/overview), sostituire `%USERPROFILE%` con `$env:USERPROFILE` .</span><span class="sxs-lookup"><span data-stu-id="4cf93-134">When using [PowerShell](/powershell/scripting/overview), replace `%USERPROFILE%` with `$env:USERPROFILE`.</span></span>

<span data-ttu-id="4cf93-135">La password deve corrispondere alla password utilizzata per il certificato.</span><span class="sxs-lookup"><span data-stu-id="4cf93-135">The password must match the password used for the certificate.</span></span>


<span data-ttu-id="4cf93-136">Nota: in questo caso il certificato deve essere un `.pfx` file.</span><span class="sxs-lookup"><span data-stu-id="4cf93-136">Note: The certificate in this case must be a `.pfx` file.</span></span>  <span data-ttu-id="4cf93-137">L'uso di `.crt` un `.key` file o con o senza la password non è supportato con il contenitore di esempio.</span><span class="sxs-lookup"><span data-stu-id="4cf93-137">Utilizing a `.crt` or `.key` file with or without the password isn't supported with the sample container.</span></span>  <span data-ttu-id="4cf93-138">Ad esempio, quando si specifica un `.crt` file, il contenitore può restituire messaggi di errore, ad esempio "la modalità server SSL deve usare un certificato con la chiave privata associata".</span><span class="sxs-lookup"><span data-stu-id="4cf93-138">For example, when specifying a `.crt` file, the container may return error messages such as 'The server mode SSL must use a certificate with the associated private key.'.</span></span> <span data-ttu-id="4cf93-139">Quando si usa [WSL](/windows/wsl/about), convalidare il percorso di montaggio per assicurarsi che il certificato venga caricato correttamente.</span><span class="sxs-lookup"><span data-stu-id="4cf93-139">When using [WSL](/windows/wsl/about), validate the mount path to ensure that the certificate loads correctly.</span></span>

### <a name="macos-or-linux"></a><span data-ttu-id="4cf93-140">macOS o Linux</span><span class="sxs-lookup"><span data-stu-id="4cf93-140">macOS or Linux</span></span>

<span data-ttu-id="4cf93-141">Generare il certificato e configurare il computer locale:</span><span class="sxs-lookup"><span data-stu-id="4cf93-141">Generate certificate and configure local machine:</span></span>

```dotnetcli
dotnet dev-certs https -ep ${HOME}/.aspnet/https/aspnetapp.pfx -p { password here }
dotnet dev-certs https --trust
```

<span data-ttu-id="4cf93-142">`dotnet dev-certs https --trust` è supportato solo in macOS e Windows.</span><span class="sxs-lookup"><span data-stu-id="4cf93-142">`dotnet dev-certs https --trust` is only supported on macOS and Windows.</span></span> <span data-ttu-id="4cf93-143">È necessario considerare attendibili i certificati in Linux nel modo supportato dalla distribuzione.</span><span class="sxs-lookup"><span data-stu-id="4cf93-143">You need to trust certs on Linux in the way that is supported by your distribution.</span></span> <span data-ttu-id="4cf93-144">È probabile che sia necessario considerare attendibile il certificato nel browser.</span><span class="sxs-lookup"><span data-stu-id="4cf93-144">It is likely that you need to trust the certificate in your browser.</span></span>

<span data-ttu-id="4cf93-145">Nei comandi precedenti sostituire `{ password here }` con una password.</span><span class="sxs-lookup"><span data-stu-id="4cf93-145">In the preceding commands, replace `{ password here }` with a password.</span></span>

<span data-ttu-id="4cf93-146">Eseguire l'immagine del contenitore con ASP.NET Core configurato per HTTPS:</span><span class="sxs-lookup"><span data-stu-id="4cf93-146">Run the container image with ASP.NET Core configured for HTTPS:</span></span>

```console
docker pull mcr.microsoft.com/dotnet/core/samples:aspnetapp
docker run --rm -it -p 8000:80 -p 8001:443 -e ASPNETCORE_URLS="https://+;http://+" -e ASPNETCORE_HTTPS_PORT=8001 -e ASPNETCORE_Kestrel__Certificates__Default__Password="password" -e ASPNETCORE_Kestrel__Certificates__Default__Path=/https/aspnetapp.pfx -v ${HOME}/.aspnet/https:/https/ mcr.microsoft.com/dotnet/core/samples:aspnetapp
```

<span data-ttu-id="4cf93-147">La password deve corrispondere alla password utilizzata per il certificato.</span><span class="sxs-lookup"><span data-stu-id="4cf93-147">The password must match the password used for the certificate.</span></span>

### <a name="windows-using-windows-containers"></a><span data-ttu-id="4cf93-148">Windows con i contenitori di Windows</span><span class="sxs-lookup"><span data-stu-id="4cf93-148">Windows using Windows containers</span></span>

<span data-ttu-id="4cf93-149">Generare il certificato e configurare il computer locale:</span><span class="sxs-lookup"><span data-stu-id="4cf93-149">Generate certificate and configure local machine:</span></span>

```dotnetcli
dotnet dev-certs https -ep %USERPROFILE%\.aspnet\https\aspnetapp.pfx -p { password here }
dotnet dev-certs https --trust
```

<span data-ttu-id="4cf93-150">Nei comandi precedenti sostituire `{ password here }` con una password.</span><span class="sxs-lookup"><span data-stu-id="4cf93-150">In the preceding commands, replace `{ password here }` with a password.</span></span> <span data-ttu-id="4cf93-151">Quando si usa [PowerShell](/powershell/scripting/overview), sostituire `%USERPROFILE%` con `$env:USERPROFILE` .</span><span class="sxs-lookup"><span data-stu-id="4cf93-151">When using [PowerShell](/powershell/scripting/overview), replace `%USERPROFILE%` with `$env:USERPROFILE`.</span></span>

<span data-ttu-id="4cf93-152">Eseguire l'immagine del contenitore con ASP.NET Core configurato per HTTPS:</span><span class="sxs-lookup"><span data-stu-id="4cf93-152">Run the container image with ASP.NET Core configured for HTTPS:</span></span>

```console
docker pull mcr.microsoft.com/dotnet/core/samples:aspnetapp
docker run --rm -it -p 8000:80 -p 8001:443 -e ASPNETCORE_URLS="https://+;http://+" -e ASPNETCORE_HTTPS_PORT=8001 -e ASPNETCORE_Kestrel__Certificates__Default__Password="password" -e ASPNETCORE_Kestrel__Certificates__Default__Path=\https\aspnetapp.pfx -v %USERPROFILE%\.aspnet\https:C:\https\ mcr.microsoft.com/dotnet/core/samples:aspnetapp
```

<span data-ttu-id="4cf93-153">La password deve corrispondere alla password utilizzata per il certificato.</span><span class="sxs-lookup"><span data-stu-id="4cf93-153">The password must match the password used for the certificate.</span></span> <span data-ttu-id="4cf93-154">Quando si usa [PowerShell](/powershell/scripting/overview), sostituire `%USERPROFILE%` con `$env:USERPROFILE` .</span><span class="sxs-lookup"><span data-stu-id="4cf93-154">When using [PowerShell](/powershell/scripting/overview), replace `%USERPROFILE%` with `$env:USERPROFILE`.</span></span>
