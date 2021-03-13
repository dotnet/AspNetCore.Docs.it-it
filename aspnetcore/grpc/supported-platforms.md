---
title: gRPC su piattaforme supportate da .NET
author: jamesnk
description: Informazioni sulle piattaforme supportate per gRPC in .NET.
monikerRange: '>= aspnetcore-3.0'
ms.author: jamesnk
ms.date: 3/11/2021
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
ms.openlocfilehash: c2bd808d16f11077e39aada829d79e8aedf2755b
ms.sourcegitcommit: 07e7ee573fe4e12be93249a385db745d714ff6ae
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/12/2021
ms.locfileid: "103413418"
---
# <a name="grpc-on-net-supported-platforms"></a>gRPC su piattaforme supportate da .NET

Di [James Newton-King](https://twitter.com/jamesnk)

Questo articolo illustra i requisiti e le piattaforme supportate per l'uso di gRPC con .NET. Esistono diversi requisiti per i due carichi di lavoro gRPC principali:

* [Hosting di servizi gRPC in ASP.NET Core](#aspnet-core-grpc-server-requirements)
* [Chiamata di gRPC da app client .NET](#net-grpc-client-requirements)

## <a name="wire-formats"></a>Formati wire

gRPC sfrutta le funzionalità avanzate disponibili in HTTP/2. HTTP/2 non è supportato in tutto il mondo, ma per gRPC è disponibile un secondo formato wire che usa HTTP/1.1:

* [`application/grpc`](https://github.com/grpc/grpc/blob/master/doc/PROTOCOL-HTTP2.md) -gRPC su HTTP/2 è il modo in cui viene in genere usato gRPC.
* [`application/grpc-web`](https://github.com/grpc/grpc/blob/master/doc/PROTOCOL-WEB.md) -gRPC-Web modifica il protocollo gRPC per la compatibilità con HTTP/1.1. gRPC-Web può essere usato in più posizioni. gRPC-Web può essere usato dalle app del browser e dalle reti senza supporto completo per HTTP/2. Due funzionalità avanzate di gRPC non sono più supportate: flusso client e flusso bidirezionale.

gRPC su .NET supporta entrambi i formati wire. `application/grpc` viene usato per impostazione predefinita. gRPC-Web deve essere configurato nel client e nel server per le chiamate gRPC-web riuscite. Per informazioni sulla configurazione di gRPC-Web, vedere <xref:grpc/browser> .

## <a name="aspnet-core-grpc-server-requirements"></a>Requisiti del server gRPC ASP.NET Core

L'hosting di servizi gRPC con ASP.NET Core richiede .NET Core 3. x o versione successiva.

> [!div class="checklist"]
>
> * .NET 5 o versione successiva
> * .NET Core 3

ASP.NET Core i servizi gRPC possono essere ospitati in tutti i sistemi operativi supportati da .NET Core.

> [!div class="checklist"]
>
> * Windows
> * Linux
> * macOS&dagger;

&dagger;[MacOS non supporta l'hosting di app ASP.NET Core con https](xref:grpc/troubleshoot#unable-to-start-aspnet-core-grpc-app-on-macos).

### <a name="supported-aspnet-core-servers"></a>Server ASP.NET Core supportati

Sono supportati tutti i server ASP.NET Core predefiniti.

> [!div class="checklist"]
>
> * Kestrel
> * TestServer
> * IIS&dagger;
> * HTTP.sys&Dagger;

&dagger;IIS richiede .NET 5 e Windows 10 Build 20241 o versione successiva.

&Dagger;HTTP.sys richiede .NET 5 e Windows 10 Build 19529 o versione successiva.

Per informazioni sulla configurazione dei server ASP.NET Core per l'esecuzione di gRPC, vedere <xref:grpc/aspnetcore#server-options> .

### <a name="azure-services"></a>Servizi di Azure

> [!div class="checklist"]
>
> * [Servizio Azure Kubernetes](https://azure.microsoft.com/services/kubernetes-service/)
> * [Servizio app Azure](https://azure.microsoft.com/services/app-service/)&dagger;

&dagger;App Azure servizio non supporta l'hosting di gRPC su HTTP/2. gRPC-Web è un'alternativa compatibile.

Il lavoro è in corso per migliorare il supporto per gRPC con HTTP/2 nel servizio app Azure. Per altre informazioni, vedere [questo problema di GitHub](https://github.com/dotnet/AspNetCore/issues/9020).

## <a name="net-grpc-client-requirements"></a>Requisiti del client gRPC .NET

Il pacchetto [Grpc .NET. client](https://www.nuget.org/packages/Grpc.Net.Client/) supporta le chiamate GRPC su http/2 in .NET Core 3 e .NET 5 o versioni successive.

È disponibile un supporto limitato per gRPC su HTTP/2 in .NET Framework. Altre versioni di .NET, ad esempio UWP, Novell e Unity, non hanno il supporto richiesto per HTTP/2 e devono invece usare gRPC-Web.

La tabella seguente elenca le implementazioni di .NET e il relativo supporto client gRPC:

| Implementazione .NET                          | gRPC su HTTP/2   | gRPC-Web   |
|----------------------------------------------|--------------------|------------|
| .NET 5 o versione successiva                              | ✔️                | ✔️         |
| .NET Core 3                                  | ✔️                | ✔️         |
| .NET Core 2.1                                | ❌                | ✔️         |
| .NET Framework 4.6.1                         | ⚠️&dagger;        | ✔️         |
| Blazor WebAssembly                           | ❌                | ✔️         |
| Mono 5.4                                     | ❌                | ✔️         |
| Xamarin.iOS 10.14                            | ❌                | ✔️         |
| Xamarin.Android 8.0                          | ❌                | ✔️         |
| Piattaforma UWP (Universal Windows Platform) 10.0.16299        | ❌                | ✔️         |
| Unity 2018,1                                 | ❌                | ✔️         |

&dagger;.NET Framework deve <xref:System.Net.Http.WinHttpHandler> essere configurato e Windows 10 Build 19622 o versione successiva.

L'uso `Grpc.Net.Client` di su .NET Framework o con gRPC-Web richiede una configurazione aggiuntiva. Per altre informazioni, vedere <xref:grpc/netstandard>.

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:grpc/netstandard>
* [gRPC C# Core-Library](https://grpc.io/docs/languages/csharp/quickstart/)
