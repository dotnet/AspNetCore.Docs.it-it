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
# <a name="hosting-aspnet-core-images-with-docker-compose-over-https"></a>Hosting di immagini ASP.NET Core con Docker Compose su HTTPS


ASP.NET Core utilizza [https per impostazione predefinita](./enforcing-ssl.md). [Https](https://en.wikipedia.org/wiki/HTTPS) si basa su [certificati](https://en.wikipedia.org/wiki/Public_key_certificate) per l'attendibilità, l'identità e la crittografia.

Questo documento illustra come eseguire le immagini del contenitore predefinite con HTTPS.

Per gli scenari di sviluppo, vedere [sviluppo di applicazioni ASP.NET Core con Docker su HTTPS](https://github.com/dotnet/dotnet-docker/blob/main/samples/run-aspnetcore-https-development.md) .

Questo esempio richiede [docker 17,06](https://docs.docker.com/release-notes/docker-ce) o versione successiva del [client Docker](https://www.docker.com/products/docker).

## <a name="prerequisites"></a>Prerequisiti

Per alcune istruzioni di questo documento è necessario [.NET Core 2,2 SDK](https://dotnet.microsoft.com/download) o versione successiva.

## <a name="certificates"></a>Certificati

Un certificato di un' [autorità di certificazione](https://wikipedia.org/wiki/Certificate_authority) è necessario per l' [hosting di produzione](https://blogs.msdn.microsoft.com/webdev/2017/11/29/configuring-https-in-asp-net-core-across-different-platforms/) per un dominio. [Let's Encrypt](https://letsencrypt.org/) è un'autorità di certificazione che offre certificati gratuiti.

Questo documento usa [certificati di sviluppo autofirmati](https://wikipedia.org/wiki/Self-signed_certificate) per l'hosting di immagini predefinite rispetto a `localhost` . Le istruzioni sono simili all'uso dei certificati di produzione.

Per i certificati di produzione:

* Lo `dotnet dev-certs` strumento non è obbligatorio.
* I certificati non devono essere archiviati nel percorso utilizzato nelle istruzioni. Archiviare i certificati in qualsiasi posizione all'esterno della directory del sito.

Le istruzioni contenute nella sezione seguente comportano i certificati di montaggio nei contenitori usando la `volumes` Proprietà in *Docker-compose. yml.* È possibile aggiungere certificati alle immagini del contenitore con un `COPY` comando in un *Dockerfile*, ma non è consigliabile. La copia di certificati in un'immagine non è consigliata per i motivi seguenti:

* Risulta difficile usare la stessa immagine per i test con certificati per sviluppatori.
* Rende difficile usare la stessa immagine per l'hosting con certificati di produzione.
* Il rischio di divulgazione dei certificati è significativo.

## <a name="starting-a-container-with-https-support-using-docker-compose"></a>Avvio di un contenitore con supporto HTTPS con Docker compose

Usare le istruzioni seguenti per la configurazione del sistema operativo.

### <a name="windows-using-linux-containers"></a>Windows con contenitori Linux

Generare il certificato e configurare il computer locale:

```dotnetcli
dotnet dev-certs https -ep %USERPROFILE%\.aspnet\https\aspnetapp.pfx -p { password here }
dotnet dev-certs https --trust
```

Nei comandi precedenti sostituire `{ password here }` con una password.

Creare un file _Docker-compose. debug. yml_ con il contenuto seguente:

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
La password specificata nel file Docker compose deve corrispondere alla password usata per il certificato.

Avviare il contenitore con ASP.NET Core configurato per HTTPS:

```console
docker-compose -f "docker-compose.debug.yml" up -d
```

### <a name="macos-or-linux"></a>macOS o Linux

Generare il certificato e configurare il computer locale:

```dotnetcli
dotnet dev-certs https -ep ${HOME}/.aspnet/https/aspnetapp.pfx -p { password here }
dotnet dev-certs https --trust
```

`dotnet dev-certs https --trust` è supportato solo in macOS e Windows. È necessario considerare attendibili i certificati in Linux nel modo supportato dalla distribuzione. È probabile che sia necessario considerare attendibile il certificato nel browser.

Nei comandi precedenti sostituire `{ password here }` con una password.

Creare un file _Docker-compose. debug. yml_ con il contenuto seguente:

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
La password specificata nel file Docker compose deve corrispondere alla password usata per il certificato.

Avviare il contenitore con ASP.NET Core configurato per HTTPS:

```console
docker-compose -f "docker-compose.debug.yml" up -d
```

### <a name="windows-using-windows-containers"></a>Windows con i contenitori di Windows

Generare il certificato e configurare il computer locale:

```dotnetcli
dotnet dev-certs https -ep %USERPROFILE%\.aspnet\https\aspnetapp.pfx -p { password here }
dotnet dev-certs https --trust
```

Nei comandi precedenti sostituire `{ password here }` con una password.

Creare un file _Docker-compose. debug. yml_ con il contenuto seguente:

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
La password specificata nel file Docker compose deve corrispondere alla password usata per il certificato.

Avviare il contenitore con ASP.NET Core configurato per HTTPS:

```console
docker-compose -f "docker-compose.debug.yml" up -d
```
