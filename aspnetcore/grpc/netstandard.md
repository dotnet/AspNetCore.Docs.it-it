---
title: Usare il client di gRPC con .NET Standard 2,0
author: jamesnk
description: Informazioni su come usare il client gRPC .NET in app e librerie che supportano .NET Standard 2,0.
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
uid: grpc/netstandard
ms.openlocfilehash: a6b066979dcdcdf648b8b0326bef47fe0e466266
ms.sourcegitcommit: 07e7ee573fe4e12be93249a385db745d714ff6ae
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/12/2021
ms.locfileid: "103422501"
---
# <a name="use-grpc-client-with-net-standard-20"></a>Usare il client di gRPC con .NET Standard 2,0

Di [James Newton-King](https://twitter.com/jamesnk)

Questo articolo illustra come usare il client .NET gRPC con implementazioni .NET che supportano [.NET Standard 2,0](/dotnet/standard/net-standard).

## <a name="net-implementations"></a>Implementazioni di .NET

Le seguenti implementazioni di .NET (o versioni successive) supportano [Grpc .NET. client](https://www.nuget.org/packages/Grpc.Net.Client/) ma non dispongono del supporto completo per http/2:

* .NET Core 2.1
* .NET Framework 4.6.1
* Mono 5.4
* Xamarin.iOS 10.14
* Xamarin.Android 8.0
* Piattaforma UWP (Universal Windows Platform) 10.0.16299
* Unity 2018,1

Il client gRPC .NET può chiamare i servizi da queste implementazioni di .NET con alcune configurazioni aggiuntive.

## <a name="httphandler-configuration"></a>Configurazione di HttpHandler

È necessario configurare un provider HTTP utilizzando `GrpcChannelOptions.HttpHandler` . Se un gestore non è configurato, viene generato un errore:

> `System.PlatformNotSupportedException`: gRPC richiede una configurazione aggiuntiva per eseguire correttamente le chiamate RPC sulle implementazioni di .NET che non supportano gRPC su HTTP/2. È necessario specificare un provider HTTP usando `GrpcChannelOptions.HttpHandler` . Il provider HTTP configurato deve supportare HTTP/2 o essere configurato per l'uso di gRPC-Web.

Le implementazioni .NET che non supportano HTTP/2, ad esempio UWP, Novell e Unity, possono usare gRPC-Web come alternativa.

```csharp
var channel = GrpcChannel.ForAddress("https://localhost:5001", new GrpcChannelOptions
    {
        HttpHandler = new GrpcWebHandler(new HttpClientHandler())
    });

var client = new Greeter.GreeterClient(channel);
var response = await client.SayHelloAsync(new HelloRequest { Name = ".NET" });
```

Per altre informazioni, vedere [Configure gRPC-Web with the .NET gRPC client](xref:grpc/browser#configure-grpc-web-with-the-net-grpc-client).

## <a name="net-framework"></a>.NET Framework

.NET Framework ha un supporto limitato per gRPC su HTTP/2. Per abilitare gRPC su HTTP/2 in .NET Framework, configurare il canale da usare <xref:System.Net.Http.WinHttpHandler> .

Requisiti e restrizioni per l'utilizzo di `WinHttpHandler` :

* Windows 10 Build 19622 o versione successiva.
* Un riferimento al pacchetto NuGet [System .NET. http. WinHttpHandler](https://www.nuget.org/packages/System.Net.Http.WinHttpHandler/) .
* Sono supportate solo le chiamate gRPC di flusso unario e server.
* Sono supportate solo le chiamate gRPC su TLS.

```csharp
var channel = GrpcChannel.ForAddress("https://localhost:5001", new GrpcChannelOptions
    {
        HttpHandler = new WinHttpHandler()
    });

var client = new Greeter.GreeterClient(channel);
var response = await client.SayHelloAsync(new HelloRequest { Name = ".NET" });
```

> [!NOTE]
> Il supporto .NET Framework è nelle fasi iniziali e richiede l'uso di software provvisorio.
> * Windows 10 Build 19622 o versione successiva è disponibile come build di [Windows Insider](https://insider.windows.com/) .
> * La versione richiesta di `System.Net.Http.WinHttpHandler` non è attualmente disponibile in NuGet.org. La versione provvisoria più recente è disponibile in [questo feed NuGet](https://pkgs.dev.azure.com/dnceng/public/_packaging/dotnet6/nuget/v3/index.json) da usare.

## <a name="grpc-c-core-library"></a>gRPC C# Core-Library

Un'opzione alternativa per .NET Framework e Novell consiste nell'usare [GRPC C# Core-Library](https://grpc.io/docs/languages/csharp/quickstart/) per effettuare chiamate gRPC. Si tratta di una libreria di terze parti che supporta l'esecuzione di chiamate gRPC su HTTP/2 su .NET Framework e Novell. gRPC C-core non è supportato da Microsoft.

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:grpc/client>
* <xref:grpc/browser>
* [gRPC C# Core-Library](https://grpc.io/docs/languages/csharp/quickstart/)
