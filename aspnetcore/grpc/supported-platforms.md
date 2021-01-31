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
# <a name="grpc-on-net-supported-platforms"></a><span data-ttu-id="00a6a-103">gRPC su piattaforme supportate da .NET</span><span class="sxs-lookup"><span data-stu-id="00a6a-103">gRPC on .NET supported platforms</span></span>

<span data-ttu-id="00a6a-104">Di [James Newton-King](https://twitter.com/jamesnk)</span><span class="sxs-lookup"><span data-stu-id="00a6a-104">By [James Newton-King](https://twitter.com/jamesnk)</span></span>

<span data-ttu-id="00a6a-105">Questo articolo illustra i requisiti e le piattaforme supportate per l'uso di gRPC con .NET.</span><span class="sxs-lookup"><span data-stu-id="00a6a-105">This article discusses the requirements and supported platforms for using gRPC with .NET.</span></span>

<span data-ttu-id="00a6a-106">gRPC sfrutta le funzionalità avanzate disponibili in HTTP/2.</span><span class="sxs-lookup"><span data-stu-id="00a6a-106">gRPC takes advantage of advanced features available in  HTTP/2.</span></span> <span data-ttu-id="00a6a-107">HTTP/2 non è supportato in tutto il mondo, ma per gRPC è disponibile un secondo formato wire che usa HTTP/1.1:</span><span class="sxs-lookup"><span data-stu-id="00a6a-107">HTTP/2 isn't supported everywhere, but a second wire-format using HTTP/1.1 is available for gRPC:</span></span>

* <span data-ttu-id="00a6a-108">[`application/grpc`](https://github.com/grpc/grpc/blob/master/doc/PROTOCOL-HTTP2.md) -gRPC su HTTP/2 è il modo in cui viene in genere usato gRPC.</span><span class="sxs-lookup"><span data-stu-id="00a6a-108">[`application/grpc`](https://github.com/grpc/grpc/blob/master/doc/PROTOCOL-HTTP2.md) - gRPC over HTTP/2 is how gRPC is typically used.</span></span>
* <span data-ttu-id="00a6a-109">[`application/grpc-web`](https://github.com/grpc/grpc/blob/master/doc/PROTOCOL-WEB.md) -gRPC-Web modifica il protocollo gRPC per la compatibilità con HTTP/1.1.</span><span class="sxs-lookup"><span data-stu-id="00a6a-109">[`application/grpc-web`](https://github.com/grpc/grpc/blob/master/doc/PROTOCOL-WEB.md) - gRPC-Web modifies the gRPC protocol to be compatible with HTTP/1.1.</span></span> <span data-ttu-id="00a6a-110">gRPC-Web può essere usato in più luoghi, in particolare è richiamabile dalle app del browser.</span><span class="sxs-lookup"><span data-stu-id="00a6a-110">gRPC-Web can be used in more places, notably it is callable by browser apps.</span></span> <span data-ttu-id="00a6a-111">Due funzionalità avanzate di gRPC non sono più supportate: flusso client e flusso bidirezionale.</span><span class="sxs-lookup"><span data-stu-id="00a6a-111">Two advanced gRPC features are no longer supported: client streaming and bidirectional streaming.</span></span>

<span data-ttu-id="00a6a-112">gRPC su .NET supporta entrambi i formati wire.</span><span class="sxs-lookup"><span data-stu-id="00a6a-112">gRPC on .NET supports both wire-formats.</span></span> <span data-ttu-id="00a6a-113">per impostazione predefinita viene usato gRPC su HTTP/2.</span><span class="sxs-lookup"><span data-stu-id="00a6a-113">gRPC over HTTP/2 is used by default.</span></span> <span data-ttu-id="00a6a-114">Per informazioni sulla configurazione di gRPC-Web, vedere <xref:grpc/browser> .</span><span class="sxs-lookup"><span data-stu-id="00a6a-114">For information on setting up gRPC-Web, see <xref:grpc/browser>.</span></span>

## <a name="device-requirements"></a><span data-ttu-id="00a6a-115">Requisiti dei dispositivi</span><span class="sxs-lookup"><span data-stu-id="00a6a-115">Device requirements</span></span>

<span data-ttu-id="00a6a-116">gRPC su .NET supporta qualsiasi dispositivo supportato da .NET Core.</span><span class="sxs-lookup"><span data-stu-id="00a6a-116">gRPC on .NET supports any device that .NET Core supports.</span></span>

> [!div class="checklist"]
>
> * <span data-ttu-id="00a6a-117">Windows</span><span class="sxs-lookup"><span data-stu-id="00a6a-117">Windows</span></span>
> * <span data-ttu-id="00a6a-118">Linux</span><span class="sxs-lookup"><span data-stu-id="00a6a-118">Linux</span></span>
> * <span data-ttu-id="00a6a-119">macOS&dagger;</span><span class="sxs-lookup"><span data-stu-id="00a6a-119">macOS&dagger;</span></span>
> * <span data-ttu-id="00a6a-120">Browser&Dagger;</span><span class="sxs-lookup"><span data-stu-id="00a6a-120">Browsers&Dagger;</span></span>

<span data-ttu-id="00a6a-121">&dagger;[MacOS non supporta l'hosting di app ASP.NET Core con https](xref:grpc/troubleshoot#unable-to-start-aspnet-core-grpc-app-on-macos).</span><span class="sxs-lookup"><span data-stu-id="00a6a-121">&dagger;[macOS doesn't support hosting ASP.NET Core apps with HTTPS](xref:grpc/troubleshoot#unable-to-start-aspnet-core-grpc-app-on-macos).</span></span> <span data-ttu-id="00a6a-122">i client gRPC in macOS possono chiamare i servizi remoti che usano HTTPS.</span><span class="sxs-lookup"><span data-stu-id="00a6a-122">gRPC clients on macOS can call remote services that use HTTPS.</span></span>

<span data-ttu-id="00a6a-123">&Dagger;Blazor WebAssembly le app possono chiamare i servizi gRPC con gRPC-Web.</span><span class="sxs-lookup"><span data-stu-id="00a6a-123">&Dagger;Blazor WebAssembly apps can call gRPC services with gRPC-Web.</span></span>

## <a name="aspnet-core-server-requirements"></a><span data-ttu-id="00a6a-124">Requisiti del server ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="00a6a-124">ASP.NET Core server requirements</span></span>

<span data-ttu-id="00a6a-125">i servizi gRPC possono essere ospitati in tutti i server ASP.NET Core predefiniti.</span><span class="sxs-lookup"><span data-stu-id="00a6a-125">gRPC services can be hosted on all built-in ASP.NET Core servers.</span></span>

> [!div class="checklist"]
>
> * <span data-ttu-id="00a6a-126">Kestrel</span><span class="sxs-lookup"><span data-stu-id="00a6a-126">Kestrel</span></span>
> * <span data-ttu-id="00a6a-127">TestServer</span><span class="sxs-lookup"><span data-stu-id="00a6a-127">TestServer</span></span>
> * <span data-ttu-id="00a6a-128">IIS&dagger;</span><span class="sxs-lookup"><span data-stu-id="00a6a-128">IIS&dagger;</span></span>
> * <span data-ttu-id="00a6a-129">HTTP.sys&dagger;</span><span class="sxs-lookup"><span data-stu-id="00a6a-129">HTTP.sys&dagger;</span></span>

<span data-ttu-id="00a6a-130">&dagger;IIS e HTTP.sys richiedono .NET 5 e Windows 10 Build 20241 o versione successiva.</span><span class="sxs-lookup"><span data-stu-id="00a6a-130">&dagger;IIS and HTTP.sys require .NET 5 and Windows 10 Build 20241 or later.</span></span>

<span data-ttu-id="00a6a-131">Per altre informazioni, vedere <xref:grpc/aspnetcore>.</span><span class="sxs-lookup"><span data-stu-id="00a6a-131">For more information, see <xref:grpc/aspnetcore>.</span></span>

## <a name="net-version-requirements"></a><span data-ttu-id="00a6a-132">Requisiti della versione di .NET</span><span class="sxs-lookup"><span data-stu-id="00a6a-132">.NET version requirements</span></span>

<span data-ttu-id="00a6a-133">gRPC su .NET supporta .NET Core 3 e .NET 5 o versione successiva.</span><span class="sxs-lookup"><span data-stu-id="00a6a-133">gRPC on .NET supports .NET Core 3 and .NET 5 or later.</span></span>

> [!div class="checklist"]
>
> * <span data-ttu-id="00a6a-134">.NET 5 o versione successiva</span><span class="sxs-lookup"><span data-stu-id="00a6a-134">.NET 5 or later</span></span>
> * <span data-ttu-id="00a6a-135">.NET Core 3</span><span class="sxs-lookup"><span data-stu-id="00a6a-135">.NET Core 3</span></span>

<span data-ttu-id="00a6a-136">gRPC in .NET non supporta l'esecuzione in .NET Framework e Novell.</span><span class="sxs-lookup"><span data-stu-id="00a6a-136">gRPC on .NET doesn't support running on .NET Framework and Xamarin.</span></span> <span data-ttu-id="00a6a-137">[GRPC C# Core-Library](https://grpc.io/docs/languages/csharp/quickstart/) è una libreria di terze parti che supporta .NET Framework e Novell.</span><span class="sxs-lookup"><span data-stu-id="00a6a-137">[gRPC C# core-library](https://grpc.io/docs/languages/csharp/quickstart/) is a third party library that supports .NET Framework and Xamarin.</span></span> <span data-ttu-id="00a6a-138">gRPC C-core non è supportato da Microsoft.</span><span class="sxs-lookup"><span data-stu-id="00a6a-138">gRPC C-core is not supported by Microsoft.</span></span>

## <a name="azure-services"></a><span data-ttu-id="00a6a-139">Servizi di Azure</span><span class="sxs-lookup"><span data-stu-id="00a6a-139">Azure services</span></span>

> [!div class="checklist"]
>
> * [<span data-ttu-id="00a6a-140">Servizio Azure Kubernetes</span><span class="sxs-lookup"><span data-stu-id="00a6a-140">Azure Kubernetes Service (AKS)</span></span>](https://azure.microsoft.com/services/kubernetes-service/)
> * <span data-ttu-id="00a6a-141">[Servizio app Azure](https://azure.microsoft.com/services/app-service/)&dagger;</span><span class="sxs-lookup"><span data-stu-id="00a6a-141">[Azure App Service](https://azure.microsoft.com/services/app-service/)&dagger;</span></span>

<span data-ttu-id="00a6a-142">&dagger;App Azure servizio non supporta l'hosting di gRPC su HTTP/2.</span><span class="sxs-lookup"><span data-stu-id="00a6a-142">&dagger;Azure App Service doesn't support hosting gRPC over HTTP/2.</span></span> <span data-ttu-id="00a6a-143">gRPC-Web è un'alternativa compatibile.</span><span class="sxs-lookup"><span data-stu-id="00a6a-143">gRPC-Web is a compatible alternative.</span></span>

<span data-ttu-id="00a6a-144">Il lavoro è in corso per migliorare il supporto per gRPC con HTTP/2 nel servizio app Azure.</span><span class="sxs-lookup"><span data-stu-id="00a6a-144">Work is in-progress to improve support for gRPC with HTTP/2 in Azure App Service.</span></span> <span data-ttu-id="00a6a-145">Per altre informazioni, vedere [questo problema di GitHub](https://github.com/dotnet/AspNetCore/issues/9020).</span><span class="sxs-lookup"><span data-stu-id="00a6a-145">For more information, see [this GitHub issue](https://github.com/dotnet/AspNetCore/issues/9020).</span></span>

## <a name="additional-resources"></a><span data-ttu-id="00a6a-146">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="00a6a-146">Additional resources</span></span>

* [<span data-ttu-id="00a6a-147">gRPC C# Core-Library</span><span class="sxs-lookup"><span data-stu-id="00a6a-147">gRPC C# core-library</span></span>](https://grpc.io/docs/languages/csharp/quickstart/)
