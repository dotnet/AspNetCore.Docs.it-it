---
title: gRPC per piattaforme supportate da .NET
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
ms.openlocfilehash: 92ca38875c6618c8630a66af16548d32bc469a62
ms.sourcegitcommit: 83524f739dd25fbfa95ee34e95342afb383b49fe
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 01/29/2021
ms.locfileid: "99057715"
---
# <a name="grpc-for-net-supported-platforms"></a>gRPC per piattaforme supportate da .NET

Di [James Newton-King](https://twitter.com/jamesnk)

Questo articolo illustra i requisiti e le piattaforme supportate per l'uso di gRPC con .NET.

gRPC è progettato per usare HTTP/2 per alcune delle funzionalità più avanzate. HTTP/2 non è supportato in tutti i quali può impedire l'uso di gRPC. Per questo motivo è disponibile un secondo formato wire compatibile con HTTP/1.1 per l'invio di chiamate gRPC tra client e server:

* [`application/grpc`](https://github.com/grpc/grpc/blob/master/doc/PROTOCOL-HTTP2.md) -gRPC su HTTP/2 è il modo in cui viene in genere usato gRPC.
* [`application/grpc-web`](https://github.com/grpc/grpc/blob/master/doc/PROTOCOL-WEB.md) -gRPC-Web modifica il protocollo gRPC per la compatibilità con HTTP/1.1. gRPC-Web può essere usato in più luoghi, in particolare è richiamabile dalle app del browser. Due funzionalità avanzate di gRPC non sono più supportate: flusso client e flusso bidirezionale.

gRPC per .NET supporta entrambi i formati wire. Per informazioni sulla configurazione di gRPC-Web, vedere <xref:grpc/browser> .

## <a name="device-requirements"></a>Requisiti dei dispositivi

gRPC per .NET supporta qualsiasi dispositivo supportato da .NET Core.

> [!div class="checklist"]
>
> * Windows
> * Linux
> * macOS&dagger;
> * Browser&Dagger;

&dagger;ASP.NET Core app ospitate in macOS non supportano HTTPS. i client gRPC in macOS possono comunque usare HTTPS quando chiamano i servizi remoti.

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

gRPC per .NET supporta .NET Core 3 e .NET 5 o versioni successive.

> [!div class="checklist"]
>
> * .NET 5 o versione successiva
> * .NET Core 3

gRPC per .NET non supporta l'esecuzione in .NET Framework e Novell. [GRPC C# Core-Library](https://grpc.io/docs/languages/csharp/quickstart/) è una libreria di terze parti che supporta .NET Framework e Novell. gRPC C-core non è supportato da Microsoft.

## <a name="azure-services"></a>Servizi di Azure

> [!div class="checklist"]
>
> * [Servizio Azure Kubernetes](https://azure.microsoft.com/services/kubernetes-service/)
> * [Servizio app Azure](https://azure.microsoft.com/services/app-service/)&dagger;

&dagger;App Azure servizio non supporta l'hosting di gRPC su HTTP/2. gRPC-Web è un'alternativa compatibile.

Il lavoro è in corso per migliorare il supporto per gRPC con HTTP/2 nel servizio app Azure. Per altre informazioni, vedere [questo problema di GitHub](https://github.com/dotnet/AspNetCore/issues/9020).

## <a name="additional-resources"></a>Risorse aggiuntive

* [gRPC C# Core-Library](https://grpc.io/docs/languages/csharp/quickstart/)
