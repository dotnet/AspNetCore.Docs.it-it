---
title: gRPC su piattaforme supportate da .NET
author: jamesnk
description: Informazioni sulle piattaforme supportate per gRPC in .NET.
monikerRange: '>= aspnetcore-3.0'
ms.author: jamesnk
ms.date: 01/22/2021
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
uid: grpc/supported-platforms
ms.openlocfilehash: 88d371f460839261b618a32564a723c257b0b119
ms.sourcegitcommit: 7e394a8527c9818caebb940f692ae4fcf2f1b277
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 01/31/2021
ms.locfileid: "99217492"
---
# <a name="grpc-on-net-supported-platforms"></a>gRPC su piattaforme supportate da .NET

Di [James Newton-King](https://twitter.com/jamesnk)

Questo articolo illustra i requisiti e le piattaforme supportate per l'uso di gRPC con .NET.

gRPC sfrutta le funzionalità avanzate disponibili in HTTP/2. HTTP/2 non è supportato in tutto il mondo, ma per gRPC è disponibile un secondo formato wire che usa HTTP/1.1:

* [`application/grpc`](https://github.com/grpc/grpc/blob/master/doc/PROTOCOL-HTTP2.md) -gRPC su HTTP/2 è il modo in cui viene in genere usato gRPC.
* [`application/grpc-web`](https://github.com/grpc/grpc/blob/master/doc/PROTOCOL-WEB.md) -gRPC-Web modifica il protocollo gRPC per la compatibilità con HTTP/1.1. gRPC-Web può essere usato in più luoghi, in particolare è richiamabile dalle app del browser. Due funzionalità avanzate di gRPC non sono più supportate: flusso client e flusso bidirezionale.

gRPC su .NET supporta entrambi i formati wire. per impostazione predefinita viene usato gRPC su HTTP/2. Per informazioni sulla configurazione di gRPC-Web, vedere <xref:grpc/browser> .

## <a name="device-requirements"></a>Requisiti dei dispositivi

gRPC su .NET supporta qualsiasi dispositivo supportato da .NET Core.

> [!div class="checklist"]
>
> * Windows
> * Linux
> * macOS&dagger;
> * Browser&Dagger;

&dagger;[MacOS non supporta l'hosting di app ASP.NET Core con https](xref:grpc/troubleshoot#unable-to-start-aspnet-core-grpc-app-on-macos). i client gRPC in macOS possono chiamare i servizi remoti che usano HTTPS.

&Dagger;Blazor WebAssembly le app possono chiamare i servizi gRPC con gRPC-Web.

## <a name="aspnet-core-server-requirements"></a>Requisiti del server ASP.NET Core

i servizi gRPC possono essere ospitati in tutti i server ASP.NET Core predefiniti.

> [!div class="checklist"]
>
> * Kestrel
> * TestServer
> * IIS&dagger;
> * HTTP.sys&dagger;

&dagger;IIS e HTTP.sys richiedono .NET 5 e Windows 10 Build 20241 o versione successiva.

Per altre informazioni, vedere <xref:grpc/aspnetcore>.

## <a name="net-version-requirements"></a>Requisiti della versione di .NET

gRPC su .NET supporta .NET Core 3 e .NET 5 o versione successiva.

> [!div class="checklist"]
>
> * .NET 5 o versione successiva
> * .NET Core 3

gRPC in .NET non supporta l'esecuzione in .NET Framework e Novell. [GRPC C# Core-Library](https://grpc.io/docs/languages/csharp/quickstart/) è una libreria di terze parti che supporta .NET Framework e Novell. gRPC C-core non è supportato da Microsoft.

## <a name="azure-services"></a>Servizi di Azure

> [!div class="checklist"]
>
> * [Servizio Azure Kubernetes](https://azure.microsoft.com/services/kubernetes-service/)
> * [Servizio app Azure](https://azure.microsoft.com/services/app-service/)&dagger;

&dagger;App Azure servizio non supporta l'hosting di gRPC su HTTP/2. gRPC-Web è un'alternativa compatibile.

Il lavoro è in corso per migliorare il supporto per gRPC con HTTP/2 nel servizio app Azure. Per altre informazioni, vedere [questo problema di GitHub](https://github.com/dotnet/AspNetCore/issues/9020).

## <a name="additional-resources"></a>Risorse aggiuntive

* [gRPC C# Core-Library](https://grpc.io/docs/languages/csharp/quickstart/)
