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
# <a name="use-grpc-client-with-net-standard-20"></a><span data-ttu-id="ec611-103">Usare il client di gRPC con .NET Standard 2,0</span><span class="sxs-lookup"><span data-stu-id="ec611-103">Use gRPC client with .NET Standard 2.0</span></span>

<span data-ttu-id="ec611-104">Di [James Newton-King](https://twitter.com/jamesnk)</span><span class="sxs-lookup"><span data-stu-id="ec611-104">By [James Newton-King](https://twitter.com/jamesnk)</span></span>

<span data-ttu-id="ec611-105">Questo articolo illustra come usare il client .NET gRPC con implementazioni .NET che supportano [.NET Standard 2,0](/dotnet/standard/net-standard).</span><span class="sxs-lookup"><span data-stu-id="ec611-105">This article discusses how to use the .NET gRPC client with .NET implementations that support [.NET Standard 2.0](/dotnet/standard/net-standard).</span></span>

## <a name="net-implementations"></a><span data-ttu-id="ec611-106">Implementazioni di .NET</span><span class="sxs-lookup"><span data-stu-id="ec611-106">.NET implementations</span></span>

<span data-ttu-id="ec611-107">Le seguenti implementazioni di .NET (o versioni successive) supportano [Grpc .NET. client](https://www.nuget.org/packages/Grpc.Net.Client/) ma non dispongono del supporto completo per http/2:</span><span class="sxs-lookup"><span data-stu-id="ec611-107">The following .NET implementations (or later) support [Grpc.Net.Client](https://www.nuget.org/packages/Grpc.Net.Client/) but don't have full support for HTTP/2:</span></span>

* <span data-ttu-id="ec611-108">.NET Core 2.1</span><span class="sxs-lookup"><span data-stu-id="ec611-108">.NET Core 2.1</span></span>
* <span data-ttu-id="ec611-109">.NET Framework 4.6.1</span><span class="sxs-lookup"><span data-stu-id="ec611-109">.NET Framework 4.6.1</span></span>
* <span data-ttu-id="ec611-110">Mono 5.4</span><span class="sxs-lookup"><span data-stu-id="ec611-110">Mono 5.4</span></span>
* <span data-ttu-id="ec611-111">Xamarin.iOS 10.14</span><span class="sxs-lookup"><span data-stu-id="ec611-111">Xamarin.iOS 10.14</span></span>
* <span data-ttu-id="ec611-112">Xamarin.Android 8.0</span><span class="sxs-lookup"><span data-stu-id="ec611-112">Xamarin.Android 8.0</span></span>
* <span data-ttu-id="ec611-113">Piattaforma UWP (Universal Windows Platform) 10.0.16299</span><span class="sxs-lookup"><span data-stu-id="ec611-113">Universal Windows Platform 10.0.16299</span></span>
* <span data-ttu-id="ec611-114">Unity 2018,1</span><span class="sxs-lookup"><span data-stu-id="ec611-114">Unity 2018.1</span></span>

<span data-ttu-id="ec611-115">Il client gRPC .NET può chiamare i servizi da queste implementazioni di .NET con alcune configurazioni aggiuntive.</span><span class="sxs-lookup"><span data-stu-id="ec611-115">The .NET gRPC client can call services from these .NET implementations with some additional configuration.</span></span>

## <a name="httphandler-configuration"></a><span data-ttu-id="ec611-116">Configurazione di HttpHandler</span><span class="sxs-lookup"><span data-stu-id="ec611-116">HttpHandler configuration</span></span>

<span data-ttu-id="ec611-117">È necessario configurare un provider HTTP utilizzando `GrpcChannelOptions.HttpHandler` .</span><span class="sxs-lookup"><span data-stu-id="ec611-117">An HTTP provider must be configured using `GrpcChannelOptions.HttpHandler`.</span></span> <span data-ttu-id="ec611-118">Se un gestore non è configurato, viene generato un errore:</span><span class="sxs-lookup"><span data-stu-id="ec611-118">If a handler isn't configured, an error is thrown:</span></span>

> <span data-ttu-id="ec611-119">`System.PlatformNotSupportedException`: gRPC richiede una configurazione aggiuntiva per eseguire correttamente le chiamate RPC sulle implementazioni di .NET che non supportano gRPC su HTTP/2.</span><span class="sxs-lookup"><span data-stu-id="ec611-119">`System.PlatformNotSupportedException`: gRPC requires extra configuration to successfully make RPC calls on .NET implementations that don't have support for gRPC over HTTP/2.</span></span> <span data-ttu-id="ec611-120">È necessario specificare un provider HTTP usando `GrpcChannelOptions.HttpHandler` .</span><span class="sxs-lookup"><span data-stu-id="ec611-120">An HTTP provider must be specified using `GrpcChannelOptions.HttpHandler`.</span></span> <span data-ttu-id="ec611-121">Il provider HTTP configurato deve supportare HTTP/2 o essere configurato per l'uso di gRPC-Web.</span><span class="sxs-lookup"><span data-stu-id="ec611-121">The configured HTTP provider must either support HTTP/2 or be configured to use gRPC-Web.</span></span>

<span data-ttu-id="ec611-122">Le implementazioni .NET che non supportano HTTP/2, ad esempio UWP, Novell e Unity, possono usare gRPC-Web come alternativa.</span><span class="sxs-lookup"><span data-stu-id="ec611-122">.NET implementations that don't support HTTP/2, such as UWP, Xamarin, and Unity, can use gRPC-Web as an alternative.</span></span>

```csharp
var channel = GrpcChannel.ForAddress("https://localhost:5001", new GrpcChannelOptions
    {
        HttpHandler = new GrpcWebHandler(new HttpClientHandler())
    });

var client = new Greeter.GreeterClient(channel);
var response = await client.SayHelloAsync(new HelloRequest { Name = ".NET" });
```

<span data-ttu-id="ec611-123">Per altre informazioni, vedere [Configure gRPC-Web with the .NET gRPC client](xref:grpc/browser#configure-grpc-web-with-the-net-grpc-client).</span><span class="sxs-lookup"><span data-stu-id="ec611-123">For more information, see [Configure gRPC-Web with the .NET gRPC client](xref:grpc/browser#configure-grpc-web-with-the-net-grpc-client).</span></span>

## <a name="net-framework"></a><span data-ttu-id="ec611-124">.NET Framework</span><span class="sxs-lookup"><span data-stu-id="ec611-124">.NET Framework</span></span>

<span data-ttu-id="ec611-125">.NET Framework ha un supporto limitato per gRPC su HTTP/2.</span><span class="sxs-lookup"><span data-stu-id="ec611-125">.NET Framework has limited support for gRPC over HTTP/2.</span></span> <span data-ttu-id="ec611-126">Per abilitare gRPC su HTTP/2 in .NET Framework, configurare il canale da usare <xref:System.Net.Http.WinHttpHandler> .</span><span class="sxs-lookup"><span data-stu-id="ec611-126">To enable gRPC over HTTP/2 on .NET Framework, configure the channel to use <xref:System.Net.Http.WinHttpHandler>.</span></span>

<span data-ttu-id="ec611-127">Requisiti e restrizioni per l'utilizzo di `WinHttpHandler` :</span><span class="sxs-lookup"><span data-stu-id="ec611-127">Requirements and restrictions to using `WinHttpHandler`:</span></span>

* <span data-ttu-id="ec611-128">Windows 10 Build 19622 o versione successiva.</span><span class="sxs-lookup"><span data-stu-id="ec611-128">Windows 10 Build 19622 or later.</span></span>
* <span data-ttu-id="ec611-129">Un riferimento al pacchetto NuGet [System .NET. http. WinHttpHandler](https://www.nuget.org/packages/System.Net.Http.WinHttpHandler/) .</span><span class="sxs-lookup"><span data-stu-id="ec611-129">A reference to the [System.Net.Http.WinHttpHandler](https://www.nuget.org/packages/System.Net.Http.WinHttpHandler/) NuGet package.</span></span>
* <span data-ttu-id="ec611-130">Sono supportate solo le chiamate gRPC di flusso unario e server.</span><span class="sxs-lookup"><span data-stu-id="ec611-130">Only unary and server streaming gRPC calls are supported.</span></span>
* <span data-ttu-id="ec611-131">Sono supportate solo le chiamate gRPC su TLS.</span><span class="sxs-lookup"><span data-stu-id="ec611-131">Only gRPC calls over TLS are supported.</span></span>

```csharp
var channel = GrpcChannel.ForAddress("https://localhost:5001", new GrpcChannelOptions
    {
        HttpHandler = new WinHttpHandler()
    });

var client = new Greeter.GreeterClient(channel);
var response = await client.SayHelloAsync(new HelloRequest { Name = ".NET" });
```

> [!NOTE]
> <span data-ttu-id="ec611-132">Il supporto .NET Framework è nelle fasi iniziali e richiede l'uso di software provvisorio.</span><span class="sxs-lookup"><span data-stu-id="ec611-132">.NET Framework support is in its early stages and requires using pre-release software.</span></span>
> * <span data-ttu-id="ec611-133">Windows 10 Build 19622 o versione successiva è disponibile come build di [Windows Insider](https://insider.windows.com/) .</span><span class="sxs-lookup"><span data-stu-id="ec611-133">Windows 10 Build 19622 or later is available as a [Windows Insiders](https://insider.windows.com/) build.</span></span>
> * <span data-ttu-id="ec611-134">La versione richiesta di `System.Net.Http.WinHttpHandler` non è attualmente disponibile in NuGet.org. La versione provvisoria più recente è disponibile in [questo feed NuGet](https://pkgs.dev.azure.com/dnceng/public/_packaging/dotnet6/nuget/v3/index.json) da usare.</span><span class="sxs-lookup"><span data-stu-id="ec611-134">The required version of `System.Net.Http.WinHttpHandler` is not currently available on NuGet.org. The latest pre-release version is available on [this NuGet feed](https://pkgs.dev.azure.com/dnceng/public/_packaging/dotnet6/nuget/v3/index.json) should be used.</span></span>

## <a name="grpc-c-core-library"></a><span data-ttu-id="ec611-135">gRPC C# Core-Library</span><span class="sxs-lookup"><span data-stu-id="ec611-135">gRPC C# core-library</span></span>

<span data-ttu-id="ec611-136">Un'opzione alternativa per .NET Framework e Novell consiste nell'usare [GRPC C# Core-Library](https://grpc.io/docs/languages/csharp/quickstart/) per effettuare chiamate gRPC.</span><span class="sxs-lookup"><span data-stu-id="ec611-136">An alternative option for .NET Framework and Xamarin is to use [gRPC C# core-library](https://grpc.io/docs/languages/csharp/quickstart/) to make gRPC calls.</span></span> <span data-ttu-id="ec611-137">Si tratta di una libreria di terze parti che supporta l'esecuzione di chiamate gRPC su HTTP/2 su .NET Framework e Novell.</span><span class="sxs-lookup"><span data-stu-id="ec611-137">It's a third party library that supports making gRPC calls over HTTP/2 on .NET Framework and Xamarin.</span></span> <span data-ttu-id="ec611-138">gRPC C-core non è supportato da Microsoft.</span><span class="sxs-lookup"><span data-stu-id="ec611-138">gRPC C-core is not supported by Microsoft.</span></span>

## <a name="additional-resources"></a><span data-ttu-id="ec611-139">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="ec611-139">Additional resources</span></span>

* <xref:grpc/client>
* <xref:grpc/browser>
* [<span data-ttu-id="ec611-140">gRPC C# Core-Library</span><span class="sxs-lookup"><span data-stu-id="ec611-140">gRPC C# core-library</span></span>](https://grpc.io/docs/languages/csharp/quickstart/)
