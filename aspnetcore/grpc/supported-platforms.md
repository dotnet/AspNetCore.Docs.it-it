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
# <a name="grpc-on-net-supported-platforms"></a><span data-ttu-id="4114a-103">gRPC su piattaforme supportate da .NET</span><span class="sxs-lookup"><span data-stu-id="4114a-103">gRPC on .NET supported platforms</span></span>

<span data-ttu-id="4114a-104">Di [James Newton-King](https://twitter.com/jamesnk)</span><span class="sxs-lookup"><span data-stu-id="4114a-104">By [James Newton-King](https://twitter.com/jamesnk)</span></span>

<span data-ttu-id="4114a-105">Questo articolo illustra i requisiti e le piattaforme supportate per l'uso di gRPC con .NET.</span><span class="sxs-lookup"><span data-stu-id="4114a-105">This article discusses the requirements and supported platforms for using gRPC with .NET.</span></span> <span data-ttu-id="4114a-106">Esistono diversi requisiti per i due carichi di lavoro gRPC principali:</span><span class="sxs-lookup"><span data-stu-id="4114a-106">There are different requirements for the two major gRPC workloads:</span></span>

* [<span data-ttu-id="4114a-107">Hosting di servizi gRPC in ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="4114a-107">Hosting gRPC services in ASP.NET Core</span></span>](#aspnet-core-grpc-server-requirements)
* [<span data-ttu-id="4114a-108">Chiamata di gRPC da app client .NET</span><span class="sxs-lookup"><span data-stu-id="4114a-108">Calling gRPC from .NET client apps</span></span>](#net-grpc-client-requirements)

## <a name="wire-formats"></a><span data-ttu-id="4114a-109">Formati wire</span><span class="sxs-lookup"><span data-stu-id="4114a-109">Wire-formats</span></span>

<span data-ttu-id="4114a-110">gRPC sfrutta le funzionalità avanzate disponibili in HTTP/2.</span><span class="sxs-lookup"><span data-stu-id="4114a-110">gRPC takes advantage of advanced features available in HTTP/2.</span></span> <span data-ttu-id="4114a-111">HTTP/2 non è supportato in tutto il mondo, ma per gRPC è disponibile un secondo formato wire che usa HTTP/1.1:</span><span class="sxs-lookup"><span data-stu-id="4114a-111">HTTP/2 isn't supported everywhere, but a second wire-format using HTTP/1.1 is available for gRPC:</span></span>

* <span data-ttu-id="4114a-112">[`application/grpc`](https://github.com/grpc/grpc/blob/master/doc/PROTOCOL-HTTP2.md) -gRPC su HTTP/2 è il modo in cui viene in genere usato gRPC.</span><span class="sxs-lookup"><span data-stu-id="4114a-112">[`application/grpc`](https://github.com/grpc/grpc/blob/master/doc/PROTOCOL-HTTP2.md) - gRPC over HTTP/2 is how gRPC is typically used.</span></span>
* <span data-ttu-id="4114a-113">[`application/grpc-web`](https://github.com/grpc/grpc/blob/master/doc/PROTOCOL-WEB.md) -gRPC-Web modifica il protocollo gRPC per la compatibilità con HTTP/1.1.</span><span class="sxs-lookup"><span data-stu-id="4114a-113">[`application/grpc-web`](https://github.com/grpc/grpc/blob/master/doc/PROTOCOL-WEB.md) - gRPC-Web modifies the gRPC protocol to be compatible with HTTP/1.1.</span></span> <span data-ttu-id="4114a-114">gRPC-Web può essere usato in più posizioni.</span><span class="sxs-lookup"><span data-stu-id="4114a-114">gRPC-Web can be used in more places.</span></span> <span data-ttu-id="4114a-115">gRPC-Web può essere usato dalle app del browser e dalle reti senza supporto completo per HTTP/2.</span><span class="sxs-lookup"><span data-stu-id="4114a-115">gRPC-Web can be used by browser apps and in networks without complete support for HTTP/2.</span></span> <span data-ttu-id="4114a-116">Due funzionalità avanzate di gRPC non sono più supportate: flusso client e flusso bidirezionale.</span><span class="sxs-lookup"><span data-stu-id="4114a-116">Two advanced gRPC features are no longer supported: client streaming and bidirectional streaming.</span></span>

<span data-ttu-id="4114a-117">gRPC su .NET supporta entrambi i formati wire.</span><span class="sxs-lookup"><span data-stu-id="4114a-117">gRPC on .NET supports both wire-formats.</span></span> <span data-ttu-id="4114a-118">`application/grpc` viene usato per impostazione predefinita.</span><span class="sxs-lookup"><span data-stu-id="4114a-118">`application/grpc` is used by default.</span></span> <span data-ttu-id="4114a-119">gRPC-Web deve essere configurato nel client e nel server per le chiamate gRPC-web riuscite.</span><span class="sxs-lookup"><span data-stu-id="4114a-119">gRPC-Web must be configured on the client and the server for successful gRPC-Web calls.</span></span> <span data-ttu-id="4114a-120">Per informazioni sulla configurazione di gRPC-Web, vedere <xref:grpc/browser> .</span><span class="sxs-lookup"><span data-stu-id="4114a-120">For information on setting up gRPC-Web, see <xref:grpc/browser>.</span></span>

## <a name="aspnet-core-grpc-server-requirements"></a><span data-ttu-id="4114a-121">Requisiti del server gRPC ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="4114a-121">ASP.NET Core gRPC server requirements</span></span>

<span data-ttu-id="4114a-122">L'hosting di servizi gRPC con ASP.NET Core richiede .NET Core 3. x o versione successiva.</span><span class="sxs-lookup"><span data-stu-id="4114a-122">Hosting gRPC services with ASP.NET Core requires .NET Core 3.x or later.</span></span>

> [!div class="checklist"]
>
> * <span data-ttu-id="4114a-123">.NET 5 o versione successiva</span><span class="sxs-lookup"><span data-stu-id="4114a-123">.NET 5 or later</span></span>
> * <span data-ttu-id="4114a-124">.NET Core 3</span><span class="sxs-lookup"><span data-stu-id="4114a-124">.NET Core 3</span></span>

<span data-ttu-id="4114a-125">ASP.NET Core i servizi gRPC possono essere ospitati in tutti i sistemi operativi supportati da .NET Core.</span><span class="sxs-lookup"><span data-stu-id="4114a-125">ASP.NET Core gRPC services can be hosted on all operating system that .NET Core supports.</span></span>

> [!div class="checklist"]
>
> * <span data-ttu-id="4114a-126">Windows</span><span class="sxs-lookup"><span data-stu-id="4114a-126">Windows</span></span>
> * <span data-ttu-id="4114a-127">Linux</span><span class="sxs-lookup"><span data-stu-id="4114a-127">Linux</span></span>
> * <span data-ttu-id="4114a-128">macOS&dagger;</span><span class="sxs-lookup"><span data-stu-id="4114a-128">macOS&dagger;</span></span>

<span data-ttu-id="4114a-129">&dagger;[MacOS non supporta l'hosting di app ASP.NET Core con https](xref:grpc/troubleshoot#unable-to-start-aspnet-core-grpc-app-on-macos).</span><span class="sxs-lookup"><span data-stu-id="4114a-129">&dagger;[macOS doesn't support hosting ASP.NET Core apps with HTTPS](xref:grpc/troubleshoot#unable-to-start-aspnet-core-grpc-app-on-macos).</span></span>

### <a name="supported-aspnet-core-servers"></a><span data-ttu-id="4114a-130">Server ASP.NET Core supportati</span><span class="sxs-lookup"><span data-stu-id="4114a-130">Supported ASP.NET Core servers</span></span>

<span data-ttu-id="4114a-131">Sono supportati tutti i server ASP.NET Core predefiniti.</span><span class="sxs-lookup"><span data-stu-id="4114a-131">All built-in ASP.NET Core servers are supported.</span></span>

> [!div class="checklist"]
>
> * <span data-ttu-id="4114a-132">Kestrel</span><span class="sxs-lookup"><span data-stu-id="4114a-132">Kestrel</span></span>
> * <span data-ttu-id="4114a-133">TestServer</span><span class="sxs-lookup"><span data-stu-id="4114a-133">TestServer</span></span>
> * <span data-ttu-id="4114a-134">IIS&dagger;</span><span class="sxs-lookup"><span data-stu-id="4114a-134">IIS&dagger;</span></span>
> * <span data-ttu-id="4114a-135">HTTP.sys&Dagger;</span><span class="sxs-lookup"><span data-stu-id="4114a-135">HTTP.sys&Dagger;</span></span>

<span data-ttu-id="4114a-136">&dagger;IIS richiede .NET 5 e Windows 10 Build 20241 o versione successiva.</span><span class="sxs-lookup"><span data-stu-id="4114a-136">&dagger;IIS requires .NET 5 and Windows 10 Build 20241 or later.</span></span>

<span data-ttu-id="4114a-137">&Dagger;HTTP.sys richiede .NET 5 e Windows 10 Build 19529 o versione successiva.</span><span class="sxs-lookup"><span data-stu-id="4114a-137">&Dagger;HTTP.sys requires .NET 5 and Windows 10 Build 19529 or later.</span></span>

<span data-ttu-id="4114a-138">Per informazioni sulla configurazione dei server ASP.NET Core per l'esecuzione di gRPC, vedere <xref:grpc/aspnetcore#server-options> .</span><span class="sxs-lookup"><span data-stu-id="4114a-138">For information about configuring ASP.NET Core servers to run gRPC, see <xref:grpc/aspnetcore#server-options>.</span></span>

### <a name="azure-services"></a><span data-ttu-id="4114a-139">Servizi di Azure</span><span class="sxs-lookup"><span data-stu-id="4114a-139">Azure services</span></span>

> [!div class="checklist"]
>
> * [<span data-ttu-id="4114a-140">Servizio Azure Kubernetes</span><span class="sxs-lookup"><span data-stu-id="4114a-140">Azure Kubernetes Service (AKS)</span></span>](https://azure.microsoft.com/services/kubernetes-service/)
> * <span data-ttu-id="4114a-141">[Servizio app Azure](https://azure.microsoft.com/services/app-service/)&dagger;</span><span class="sxs-lookup"><span data-stu-id="4114a-141">[Azure App Service](https://azure.microsoft.com/services/app-service/)&dagger;</span></span>

<span data-ttu-id="4114a-142">&dagger;App Azure servizio non supporta l'hosting di gRPC su HTTP/2.</span><span class="sxs-lookup"><span data-stu-id="4114a-142">&dagger;Azure App Service doesn't support hosting gRPC over HTTP/2.</span></span> <span data-ttu-id="4114a-143">gRPC-Web è un'alternativa compatibile.</span><span class="sxs-lookup"><span data-stu-id="4114a-143">gRPC-Web is a compatible alternative.</span></span>

<span data-ttu-id="4114a-144">Il lavoro è in corso per migliorare il supporto per gRPC con HTTP/2 nel servizio app Azure.</span><span class="sxs-lookup"><span data-stu-id="4114a-144">Work is in-progress to improve support for gRPC with HTTP/2 in Azure App Service.</span></span> <span data-ttu-id="4114a-145">Per altre informazioni, vedere [questo problema di GitHub](https://github.com/dotnet/AspNetCore/issues/9020).</span><span class="sxs-lookup"><span data-stu-id="4114a-145">For more information, see [this GitHub issue](https://github.com/dotnet/AspNetCore/issues/9020).</span></span>

## <a name="net-grpc-client-requirements"></a><span data-ttu-id="4114a-146">Requisiti del client gRPC .NET</span><span class="sxs-lookup"><span data-stu-id="4114a-146">.NET gRPC client requirements</span></span>

<span data-ttu-id="4114a-147">Il pacchetto [Grpc .NET. client](https://www.nuget.org/packages/Grpc.Net.Client/) supporta le chiamate GRPC su http/2 in .NET Core 3 e .NET 5 o versioni successive.</span><span class="sxs-lookup"><span data-stu-id="4114a-147">The [Grpc.Net.Client](https://www.nuget.org/packages/Grpc.Net.Client/) package supports gRPC calls over HTTP/2 on .NET Core 3 and .NET 5 or later.</span></span>

<span data-ttu-id="4114a-148">È disponibile un supporto limitato per gRPC su HTTP/2 in .NET Framework.</span><span class="sxs-lookup"><span data-stu-id="4114a-148">Limited support is available for gRPC over HTTP/2 on .NET Framework.</span></span> <span data-ttu-id="4114a-149">Altre versioni di .NET, ad esempio UWP, Novell e Unity, non hanno il supporto richiesto per HTTP/2 e devono invece usare gRPC-Web.</span><span class="sxs-lookup"><span data-stu-id="4114a-149">Other .NET versions such as UWP, Xamarin and Unity don't have required HTTP/2 support, and must use gRPC-Web instead.</span></span>

<span data-ttu-id="4114a-150">La tabella seguente elenca le implementazioni di .NET e il relativo supporto client gRPC:</span><span class="sxs-lookup"><span data-stu-id="4114a-150">The following table lists .NET implementations and their gRPC client support:</span></span>

| <span data-ttu-id="4114a-151">Implementazione .NET</span><span class="sxs-lookup"><span data-stu-id="4114a-151">.NET implementation</span></span>                          | <span data-ttu-id="4114a-152">gRPC su HTTP/2</span><span class="sxs-lookup"><span data-stu-id="4114a-152">gRPC over HTTP/2</span></span>   | <span data-ttu-id="4114a-153">gRPC-Web</span><span class="sxs-lookup"><span data-stu-id="4114a-153">gRPC-Web</span></span>   |
|----------------------------------------------|--------------------|------------|
| <span data-ttu-id="4114a-154">.NET 5 o versione successiva</span><span class="sxs-lookup"><span data-stu-id="4114a-154">.NET 5 or later</span></span>                              | <span data-ttu-id="4114a-155">✔️</span><span class="sxs-lookup"><span data-stu-id="4114a-155">✔️</span></span>                | <span data-ttu-id="4114a-156">✔️</span><span class="sxs-lookup"><span data-stu-id="4114a-156">✔️</span></span>         |
| <span data-ttu-id="4114a-157">.NET Core 3</span><span class="sxs-lookup"><span data-stu-id="4114a-157">.NET Core 3</span></span>                                  | <span data-ttu-id="4114a-158">✔️</span><span class="sxs-lookup"><span data-stu-id="4114a-158">✔️</span></span>                | <span data-ttu-id="4114a-159">✔️</span><span class="sxs-lookup"><span data-stu-id="4114a-159">✔️</span></span>         |
| <span data-ttu-id="4114a-160">.NET Core 2.1</span><span class="sxs-lookup"><span data-stu-id="4114a-160">.NET Core 2.1</span></span>                                | ❌                | <span data-ttu-id="4114a-161">✔️</span><span class="sxs-lookup"><span data-stu-id="4114a-161">✔️</span></span>         |
| <span data-ttu-id="4114a-162">.NET Framework 4.6.1</span><span class="sxs-lookup"><span data-stu-id="4114a-162">.NET Framework 4.6.1</span></span>                         | ⚠️&dagger;        | <span data-ttu-id="4114a-163">✔️</span><span class="sxs-lookup"><span data-stu-id="4114a-163">✔️</span></span>         |
| Blazor WebAssembly                           | ❌                | <span data-ttu-id="4114a-164">✔️</span><span class="sxs-lookup"><span data-stu-id="4114a-164">✔️</span></span>         |
| <span data-ttu-id="4114a-165">Mono 5.4</span><span class="sxs-lookup"><span data-stu-id="4114a-165">Mono 5.4</span></span>                                     | ❌                | <span data-ttu-id="4114a-166">✔️</span><span class="sxs-lookup"><span data-stu-id="4114a-166">✔️</span></span>         |
| <span data-ttu-id="4114a-167">Xamarin.iOS 10.14</span><span class="sxs-lookup"><span data-stu-id="4114a-167">Xamarin.iOS 10.14</span></span>                            | ❌                | <span data-ttu-id="4114a-168">✔️</span><span class="sxs-lookup"><span data-stu-id="4114a-168">✔️</span></span>         |
| <span data-ttu-id="4114a-169">Xamarin.Android 8.0</span><span class="sxs-lookup"><span data-stu-id="4114a-169">Xamarin.Android 8.0</span></span>                          | ❌                | <span data-ttu-id="4114a-170">✔️</span><span class="sxs-lookup"><span data-stu-id="4114a-170">✔️</span></span>         |
| <span data-ttu-id="4114a-171">Piattaforma UWP (Universal Windows Platform) 10.0.16299</span><span class="sxs-lookup"><span data-stu-id="4114a-171">Universal Windows Platform 10.0.16299</span></span>        | ❌                | <span data-ttu-id="4114a-172">✔️</span><span class="sxs-lookup"><span data-stu-id="4114a-172">✔️</span></span>         |
| <span data-ttu-id="4114a-173">Unity 2018,1</span><span class="sxs-lookup"><span data-stu-id="4114a-173">Unity 2018.1</span></span>                                 | ❌                | <span data-ttu-id="4114a-174">✔️</span><span class="sxs-lookup"><span data-stu-id="4114a-174">✔️</span></span>         |

<span data-ttu-id="4114a-175">&dagger;.NET Framework deve <xref:System.Net.Http.WinHttpHandler> essere configurato e Windows 10 Build 19622 o versione successiva.</span><span class="sxs-lookup"><span data-stu-id="4114a-175">&dagger;.NET Framework requires <xref:System.Net.Http.WinHttpHandler> to be configured and Windows 10 Build 19622 or later.</span></span>

<span data-ttu-id="4114a-176">L'uso `Grpc.Net.Client` di su .NET Framework o con gRPC-Web richiede una configurazione aggiuntiva.</span><span class="sxs-lookup"><span data-stu-id="4114a-176">Using `Grpc.Net.Client` on .NET Framework or with gRPC-Web requires additional configuration.</span></span> <span data-ttu-id="4114a-177">Per altre informazioni, vedere <xref:grpc/netstandard>.</span><span class="sxs-lookup"><span data-stu-id="4114a-177">For more information, see <xref:grpc/netstandard>.</span></span>

## <a name="additional-resources"></a><span data-ttu-id="4114a-178">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="4114a-178">Additional resources</span></span>

* <xref:grpc/netstandard>
* [<span data-ttu-id="4114a-179">gRPC C# Core-Library</span><span class="sxs-lookup"><span data-stu-id="4114a-179">gRPC C# core-library</span></span>](https://grpc.io/docs/languages/csharp/quickstart/)
