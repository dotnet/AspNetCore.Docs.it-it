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
# <a name="hosting-aspnet-core-images-with-docker-over-https"></a>Hosting di immagini ASP.NET Core con Docker su HTTPS

Autore: [Rick Anderson](https://twitter.com/RickAndMSFT)

ASP.NET Core utilizza [https per impostazione predefinita](./enforcing-ssl.md). [Https](https://en.wikipedia.org/wiki/HTTPS) si basa su [certificati](https://en.wikipedia.org/wiki/Public_key_certificate) per l'attendibilità, l'identità e la crittografia.

Questo documento illustra come eseguire le immagini del contenitore predefinite con HTTPS.

Per gli scenari di sviluppo, vedere [sviluppo di applicazioni ASP.NET Core con Docker su HTTPS](https://github.com/dotnet/dotnet-docker/blob/main/samples/run-aspnetcore-https-development.md) .

Questo esempio richiede [docker 17,06](https://docs.docker.com/release-notes/docker-ce) o versione successiva del [client Docker](https://www.docker.com/products/docker).

## <a name="prerequisites"></a>Prerequisiti

Per alcune istruzioni di questo documento è necessario [.NET Core 2,2 SDK](https://dotnet.microsoft.com/download) o versione successiva.

## <a name="certificates"></a>Certificati

Un certificato di un' [autorità di certificazione](https://wikipedia.org/wiki/Certificate_authority) è necessario per l' [hosting di produzione](https://blogs.msdn.microsoft.com/webdev/2017/11/29/configuring-https-in-asp-net-core-across-different-platforms/) per un dominio. [Let's Encrypt](https://letsencrypt.org/) è un'autorità di certificazione che offre certificati gratuiti.

Questo documento usa [certificati di sviluppo autofirmati](https://en.wikipedia.org/wiki/Self-signed_certificate) per l'hosting di immagini predefinite rispetto a `localhost` . Le istruzioni sono simili all'uso dei certificati di produzione.

Usare [DotNet dev-certs](/dotnet/core/additional-tools/self-signed-certificates-guide) per creare certificati autofirmati per lo sviluppo e il test.

Per i certificati di produzione:

* Lo `dotnet dev-certs` strumento non è obbligatorio.
* Non è necessario archiviare i certificati nel percorso utilizzato nelle istruzioni. Qualsiasi località dovrebbe funzionare, anche se non è consigliabile archiviare i certificati nella directory del sito.

Le istruzioni contenute nella sezione seguente comportano i certificati di montaggio nei contenitori con l'opzione della riga di comando di Docker `-v` . È possibile aggiungere certificati alle immagini del contenitore con un `COPY` comando in un *Dockerfile*, ma non è consigliabile. La copia di certificati in un'immagine non è consigliata per i motivi seguenti:

* Risulta difficile usare la stessa immagine per i test con certificati per sviluppatori.
* Risulta difficile utilizzare la stessa immagine per l'hosting con certificati di produzione.
* Il rischio di divulgazione dei certificati è significativo.

## <a name="running-pre-built-container-images-with-https"></a>Esecuzione di immagini del contenitore predefinite con HTTPS

Usare le istruzioni seguenti per la configurazione del sistema operativo.

### <a name="windows-using-linux-containers"></a>Windows con contenitori Linux

Generare il certificato e configurare il computer locale:

```dotnetcli
dotnet dev-certs https -ep %USERPROFILE%\.aspnet\https\aspnetapp.pfx -p { password here }
dotnet dev-certs https --trust
```

Nei comandi precedenti sostituire `{ password here }` con una password.

Eseguire l'immagine del contenitore con ASP.NET Core configurato per HTTPS in una shell dei comandi:

```console
docker pull mcr.microsoft.com/dotnet/core/samples:aspnetapp
docker run --rm -it -p 8000:80 -p 8001:443 -e ASPNETCORE_URLS="https://+;http://+" -e ASPNETCORE_HTTPS_PORT=8001 -e ASPNETCORE_Kestrel__Certificates__Default__Password="password" -e ASPNETCORE_Kestrel__Certificates__Default__Path=/https/aspnetapp.pfx -v %USERPROFILE%\.aspnet\https:/https/ mcr.microsoft.com/dotnet/core/samples:aspnetapp
```

Quando si usa [PowerShell](/powershell/scripting/overview), sostituire `%USERPROFILE%` con `$env:USERPROFILE` .

La password deve corrispondere alla password utilizzata per il certificato.


Nota: in questo caso il certificato deve essere un `.pfx` file.  L'uso di `.crt` un `.key` file o con o senza la password non è supportato con il contenitore di esempio.  Ad esempio, quando si specifica un `.crt` file, il contenitore può restituire messaggi di errore, ad esempio "la modalità server SSL deve usare un certificato con la chiave privata associata". Quando si usa [WSL](/windows/wsl/about), convalidare il percorso di montaggio per assicurarsi che il certificato venga caricato correttamente.

### <a name="macos-or-linux"></a>macOS o Linux

Generare il certificato e configurare il computer locale:

```dotnetcli
dotnet dev-certs https -ep ${HOME}/.aspnet/https/aspnetapp.pfx -p { password here }
dotnet dev-certs https --trust
```

`dotnet dev-certs https --trust` è supportato solo in macOS e Windows. È necessario considerare attendibili i certificati in Linux nel modo supportato dalla distribuzione. È probabile che sia necessario considerare attendibile il certificato nel browser.

Nei comandi precedenti sostituire `{ password here }` con una password.

Eseguire l'immagine del contenitore con ASP.NET Core configurato per HTTPS:

```console
docker pull mcr.microsoft.com/dotnet/core/samples:aspnetapp
docker run --rm -it -p 8000:80 -p 8001:443 -e ASPNETCORE_URLS="https://+;http://+" -e ASPNETCORE_HTTPS_PORT=8001 -e ASPNETCORE_Kestrel__Certificates__Default__Password="password" -e ASPNETCORE_Kestrel__Certificates__Default__Path=/https/aspnetapp.pfx -v ${HOME}/.aspnet/https:/https/ mcr.microsoft.com/dotnet/core/samples:aspnetapp
```

La password deve corrispondere alla password utilizzata per il certificato.

### <a name="windows-using-windows-containers"></a>Windows con i contenitori di Windows

Generare il certificato e configurare il computer locale:

```dotnetcli
dotnet dev-certs https -ep %USERPROFILE%\.aspnet\https\aspnetapp.pfx -p { password here }
dotnet dev-certs https --trust
```

Nei comandi precedenti sostituire `{ password here }` con una password. Quando si usa [PowerShell](/powershell/scripting/overview), sostituire `%USERPROFILE%` con `$env:USERPROFILE` .

Eseguire l'immagine del contenitore con ASP.NET Core configurato per HTTPS:

```console
docker pull mcr.microsoft.com/dotnet/core/samples:aspnetapp
docker run --rm -it -p 8000:80 -p 8001:443 -e ASPNETCORE_URLS="https://+;http://+" -e ASPNETCORE_HTTPS_PORT=8001 -e ASPNETCORE_Kestrel__Certificates__Default__Password="password" -e ASPNETCORE_Kestrel__Certificates__Default__Path=\https\aspnetapp.pfx -v %USERPROFILE%\.aspnet\https:C:\https\ mcr.microsoft.com/dotnet/core/samples:aspnetapp
```

La password deve corrispondere alla password utilizzata per il certificato. Quando si usa [PowerShell](/powershell/scripting/overview), sostituire `%USERPROFILE%` con `$env:USERPROFILE` .
