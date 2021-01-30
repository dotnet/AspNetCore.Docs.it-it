---
title: Servizi gRPC con ASP.NET Core
author: juntaoluo
description: Informazioni sui concetti di base per la scrittura di servizi gRPC con ASP.NET Core.
monikerRange: '>= aspnetcore-3.0'
ms.author: johluo
ms.date: 01/29/2021
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
uid: grpc/aspnetcore
ms.openlocfilehash: 57edfa31079cb3fca6e9e8d0fa55bcbb8bbfefca
ms.sourcegitcommit: 83524f739dd25fbfa95ee34e95342afb383b49fe
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 01/29/2021
ms.locfileid: "99057480"
---
# <a name="grpc-services-with-aspnet-core"></a><span data-ttu-id="420a1-103">Servizi gRPC con ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="420a1-103">gRPC services with ASP.NET Core</span></span>

<span data-ttu-id="420a1-104">Questo documento illustra come iniziare a usare i servizi di gRPC con ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="420a1-104">This document shows how to get started with gRPC services using ASP.NET Core.</span></span>

[!INCLUDE[](~/includes/gRPCazure.md)]

## <a name="prerequisites"></a><span data-ttu-id="420a1-105">Prerequisiti</span><span class="sxs-lookup"><span data-stu-id="420a1-105">Prerequisites</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="420a1-106">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="420a1-106">Visual Studio</span></span>](#tab/visual-studio)

[!INCLUDE[](~/includes/net-core-prereqs-vs-3.0.md)]

# <a name="visual-studio-code"></a>[<span data-ttu-id="420a1-107">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="420a1-107">Visual Studio Code</span></span>](#tab/visual-studio-code)

[!INCLUDE[](~/includes/net-core-prereqs-vsc-3.0.md)]

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="420a1-108">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="420a1-108">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

[!INCLUDE[](~/includes/net-core-prereqs-mac-3.0.md)]

---

## <a name="get-started-with-grpc-service-in-aspnet-core"></a><span data-ttu-id="420a1-109">Introduzione all'uso del servizio gRPC in ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="420a1-109">Get started with gRPC service in ASP.NET Core</span></span>

<span data-ttu-id="420a1-110">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/tutorials/grpc/grpc-start/sample) ([procedura per il download](xref:index#how-to-download-a-sample)).</span><span class="sxs-lookup"><span data-stu-id="420a1-110">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/tutorials/grpc/grpc-start/sample) ([how to download](xref:index#how-to-download-a-sample)).</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="420a1-111">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="420a1-111">Visual Studio</span></span>](#tab/visual-studio)

<span data-ttu-id="420a1-112">Per istruzioni dettagliate su come creare un progetto gRPC, vedere [Introduzione ai servizi gRPC](xref:tutorials/grpc/grpc-start) .</span><span class="sxs-lookup"><span data-stu-id="420a1-112">See [Get started with gRPC services](xref:tutorials/grpc/grpc-start) for detailed instructions on how to create a gRPC project.</span></span>

# <a name="visual-studio-code--visual-studio-for-mac"></a>[<span data-ttu-id="420a1-113">Visual Studio Code / Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="420a1-113">Visual Studio Code / Visual Studio for Mac</span></span>](#tab/visual-studio-code+visual-studio-mac)

<span data-ttu-id="420a1-114">Eseguire `dotnet new grpc -o GrpcGreeter` dalla riga di comando.</span><span class="sxs-lookup"><span data-stu-id="420a1-114">Run `dotnet new grpc -o GrpcGreeter` from the command line.</span></span>

---

## <a name="add-grpc-services-to-an-aspnet-core-app"></a><span data-ttu-id="420a1-115">Aggiungere servizi gRPC a un'app ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="420a1-115">Add gRPC services to an ASP.NET Core app</span></span>

<span data-ttu-id="420a1-116">gRPC richiede il pacchetto [gRPC. AspNetCore](https://www.nuget.org/packages/Grpc.AspNetCore) .</span><span class="sxs-lookup"><span data-stu-id="420a1-116">gRPC requires the [Grpc.AspNetCore](https://www.nuget.org/packages/Grpc.AspNetCore) package.</span></span>

### <a name="configure-grpc"></a><span data-ttu-id="420a1-117">Configurare gRPC</span><span class="sxs-lookup"><span data-stu-id="420a1-117">Configure gRPC</span></span>

<span data-ttu-id="420a1-118">In *Startup.cs*:</span><span class="sxs-lookup"><span data-stu-id="420a1-118">In *Startup.cs*:</span></span>

* <span data-ttu-id="420a1-119">gRPC è abilitato con il `AddGrpc` metodo.</span><span class="sxs-lookup"><span data-stu-id="420a1-119">gRPC is enabled with the `AddGrpc` method.</span></span>
* <span data-ttu-id="420a1-120">Ogni servizio gRPC viene aggiunto alla pipeline di routing tramite il `MapGrpcService` metodo.</span><span class="sxs-lookup"><span data-stu-id="420a1-120">Each gRPC service is added to the routing pipeline through the `MapGrpcService` method.</span></span>

[!code-csharp[](~/tutorials/grpc/grpc-start/sample/GrpcGreeter/Startup.cs?name=snippet&highlight=7,24)]
[!INCLUDE[about the series](~/includes/code-comments-loc.md)]

<span data-ttu-id="420a1-121">ASP.NET Core middleware e le funzionalità condividono la pipeline di routing, di conseguenza è possibile configurare un'app per gestire gestori di richieste aggiuntivi.</span><span class="sxs-lookup"><span data-stu-id="420a1-121">ASP.NET Core middleware and features share the routing pipeline, therefore an app can be configured to serve additional request handlers.</span></span> <span data-ttu-id="420a1-122">I gestori di richieste aggiuntivi, ad esempio i controller MVC, funzionano in parallelo con i servizi gRPC configurati.</span><span class="sxs-lookup"><span data-stu-id="420a1-122">The additional request handlers, such as MVC controllers, work in parallel with the configured gRPC services.</span></span>

## <a name="server-options"></a><span data-ttu-id="420a1-123">Opzioni server</span><span class="sxs-lookup"><span data-stu-id="420a1-123">Server options</span></span>

<span data-ttu-id="420a1-124">i servizi gRPC possono essere ospitati da tutti i server ASP.NET Core predefiniti.</span><span class="sxs-lookup"><span data-stu-id="420a1-124">gRPC services can be hosted by all built-in ASP.NET Core servers.</span></span>

> [!div class="checklist"]
>
> * <span data-ttu-id="420a1-125">Kestrel</span><span class="sxs-lookup"><span data-stu-id="420a1-125">Kestrel</span></span>
> * <span data-ttu-id="420a1-126">TestServer</span><span class="sxs-lookup"><span data-stu-id="420a1-126">TestServer</span></span>
> * <span data-ttu-id="420a1-127">IIS&dagger;</span><span class="sxs-lookup"><span data-stu-id="420a1-127">IIS&dagger;</span></span>
> * <span data-ttu-id="420a1-128">HTTP.sys&dagger;</span><span class="sxs-lookup"><span data-stu-id="420a1-128">HTTP.sys&dagger;</span></span>

<span data-ttu-id="420a1-129">&dagger;IIS e HTTP.sys richiedono .NET 5 e Windows 10 Build 20241 o versione successiva.</span><span class="sxs-lookup"><span data-stu-id="420a1-129">&dagger;IIS and HTTP.sys require .NET 5 and Windows 10 Build 20241 or later.</span></span>

<span data-ttu-id="420a1-130">Per ulteriori informazioni sulla scelta del server appropriato per un'app ASP.NET Core, vedere <xref:fundamentals/servers/index> .</span><span class="sxs-lookup"><span data-stu-id="420a1-130">For more information about choosing the right server for an ASP.NET Core app, see <xref:fundamentals/servers/index>.</span></span>

::: moniker range=">= aspnetcore-5.0"

## <a name="kestrel"></a><span data-ttu-id="420a1-131">Kestrel</span><span class="sxs-lookup"><span data-stu-id="420a1-131">Kestrel</span></span>

<span data-ttu-id="420a1-132">[Gheppio](xref:fundamentals/servers/kestrel) è un server Web multipiattaforma per ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="420a1-132">[Kestrel](xref:fundamentals/servers/kestrel) is a cross-platform web server for ASP.NET Core.</span></span> <span data-ttu-id="420a1-133">Gheppio fornisce le prestazioni migliori e l'utilizzo della memoria, ma non alcune delle funzionalità avanzate di HTTP.sys come la condivisione delle porte.</span><span class="sxs-lookup"><span data-stu-id="420a1-133">Kestrel provides the best performance and memory utilization, but it doesn't have some of the advanced features in HTTP.sys such as port sharing.</span></span>

<span data-ttu-id="420a1-134">Endpoint gRPC di Gheppio:</span><span class="sxs-lookup"><span data-stu-id="420a1-134">Kestrel gRPC endpoints:</span></span>

* <span data-ttu-id="420a1-135">Richiedi HTTP/2.</span><span class="sxs-lookup"><span data-stu-id="420a1-135">Require HTTP/2.</span></span>
* <span data-ttu-id="420a1-136">Deve essere protetto con [Transport Layer Security (TLS)](https://tools.ietf.org/html/rfc5246).</span><span class="sxs-lookup"><span data-stu-id="420a1-136">Should be secured with [Transport Layer Security (TLS)](https://tools.ietf.org/html/rfc5246).</span></span>

### <a name="http2"></a><span data-ttu-id="420a1-137">HTTP/2</span><span class="sxs-lookup"><span data-stu-id="420a1-137">HTTP/2</span></span>

<span data-ttu-id="420a1-138">gRPC richiede HTTP/2.</span><span class="sxs-lookup"><span data-stu-id="420a1-138">gRPC requires HTTP/2.</span></span> <span data-ttu-id="420a1-139">gRPC per ASP.NET Core convalida [HttpRequest. Protocol](xref:Microsoft.AspNetCore.Http.HttpRequest.Protocol%2A) è `HTTP/2` .</span><span class="sxs-lookup"><span data-stu-id="420a1-139">gRPC for ASP.NET Core validates [HttpRequest.Protocol](xref:Microsoft.AspNetCore.Http.HttpRequest.Protocol%2A) is `HTTP/2`.</span></span>

<span data-ttu-id="420a1-140">Gheppio [supporta http/2](xref:fundamentals/servers/kestrel/http2) nei sistemi operativi più recenti.</span><span class="sxs-lookup"><span data-stu-id="420a1-140">Kestrel [supports HTTP/2](xref:fundamentals/servers/kestrel/http2) on most modern operating systems.</span></span> <span data-ttu-id="420a1-141">Per impostazione predefinita, gli endpoint gheppio sono configurati per supportare connessioni HTTP/1.1 e HTTP/2.</span><span class="sxs-lookup"><span data-stu-id="420a1-141">Kestrel endpoints are configured to support HTTP/1.1 and HTTP/2 connections by default.</span></span>

### <a name="tls"></a><span data-ttu-id="420a1-142">TLS</span><span class="sxs-lookup"><span data-stu-id="420a1-142">TLS</span></span>

<span data-ttu-id="420a1-143">Gli endpoint gheppio usati per gRPC devono essere protetti con TLS.</span><span class="sxs-lookup"><span data-stu-id="420a1-143">Kestrel endpoints used for gRPC should be secured with TLS.</span></span> <span data-ttu-id="420a1-144">In fase di sviluppo, un endpoint protetto con TLS viene creato automaticamente in corrispondenza del momento in `https://localhost:5001` cui è presente il certificato di sviluppo ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="420a1-144">In development, an endpoint secured with TLS is automatically created at `https://localhost:5001` when the ASP.NET Core development certificate is present.</span></span> <span data-ttu-id="420a1-145">Non è richiesta alcuna configurazione.</span><span class="sxs-lookup"><span data-stu-id="420a1-145">No configuration is required.</span></span> <span data-ttu-id="420a1-146">Un `https` prefisso verifica che l'endpoint gheppio stia usando TLS.</span><span class="sxs-lookup"><span data-stu-id="420a1-146">An `https` prefix verifies the Kestrel endpoint is using TLS.</span></span>

<span data-ttu-id="420a1-147">In produzione, è necessario configurare in modo esplicito TLS.</span><span class="sxs-lookup"><span data-stu-id="420a1-147">In production, TLS must be explicitly configured.</span></span> <span data-ttu-id="420a1-148">Nell'esempio seguente *appsettings.json* viene fornito un endpoint HTTP/2 protetto con TLS:</span><span class="sxs-lookup"><span data-stu-id="420a1-148">In the following *appsettings.json* example, an HTTP/2 endpoint secured with TLS is provided:</span></span>

[!code-json[](~/grpc/aspnetcore/sample/appsettings.json?highlight=4)]

<span data-ttu-id="420a1-149">In alternativa, è possibile configurare gli endpoint gheppio in *Program.cs*:</span><span class="sxs-lookup"><span data-stu-id="420a1-149">Alternatively, Kestrel endpoints can be configured in *Program.cs*:</span></span>

[!code-csharp[](~/grpc/aspnetcore/sample/Program.cs?highlight=7&name=snippet)]

### <a name="protocol-negotiation"></a><span data-ttu-id="420a1-150">Negoziazione del protocollo</span><span class="sxs-lookup"><span data-stu-id="420a1-150">Protocol negotiation</span></span>

<span data-ttu-id="420a1-151">TLS viene usato per una maggiore sicurezza della comunicazione.</span><span class="sxs-lookup"><span data-stu-id="420a1-151">TLS is used for more than securing communication.</span></span> <span data-ttu-id="420a1-152">L'handshake TLS [(ALPN)](https://tools.ietf.org/html/rfc7301#section-3) viene utilizzato per negoziare il protocollo di connessione tra il client e il server quando un endpoint supporta più protocolli.</span><span class="sxs-lookup"><span data-stu-id="420a1-152">The TLS [Application-Layer Protocol Negotiation (ALPN)](https://tools.ietf.org/html/rfc7301#section-3) handshake is used to negotiate the connection protocol between the client and the server when an endpoint supports multiple protocols.</span></span> <span data-ttu-id="420a1-153">Questa negoziazione determina se la connessione utilizza HTTP/1.1 o HTTP/2.</span><span class="sxs-lookup"><span data-stu-id="420a1-153">This negotiation determines whether the connection uses HTTP/1.1 or HTTP/2.</span></span>

<span data-ttu-id="420a1-154">Se un endpoint HTTP/2 viene configurato senza TLS, l'endpoint [ListenOptions. Protocols](xref:fundamentals/servers/kestrel/endpoints#listenoptionsprotocols) deve essere impostato su `HttpProtocols.Http2` .</span><span class="sxs-lookup"><span data-stu-id="420a1-154">If an HTTP/2 endpoint is configured without TLS, the endpoint's [ListenOptions.Protocols](xref:fundamentals/servers/kestrel/endpoints#listenoptionsprotocols) must be set to `HttpProtocols.Http2`.</span></span> <span data-ttu-id="420a1-155">Un endpoint con più protocolli (ad esempio, `HttpProtocols.Http1AndHttp2` ) non può essere usato senza TLS perché non esiste alcuna negoziazione.</span><span class="sxs-lookup"><span data-stu-id="420a1-155">An endpoint with multiple protocols (for example, `HttpProtocols.Http1AndHttp2`) can't be used without TLS because there's no negotiation.</span></span> <span data-ttu-id="420a1-156">Per impostazione predefinita, tutte le connessioni all'endpoint non protetto sono HTTP/1.1 e le chiamate a gRPC hanno esito negativo.</span><span class="sxs-lookup"><span data-stu-id="420a1-156">All connections to the unsecured endpoint default to HTTP/1.1, and gRPC calls fail.</span></span>

<span data-ttu-id="420a1-157">Per ulteriori informazioni sull'abilitazione di HTTP/2 e TLS con gheppio, vedere la pagina relativa alla [configurazione dell'endpoint gheppio](xref:fundamentals/servers/kestrel/endpoints).</span><span class="sxs-lookup"><span data-stu-id="420a1-157">For more information on enabling HTTP/2 and TLS with Kestrel, see [Kestrel endpoint configuration](xref:fundamentals/servers/kestrel/endpoints).</span></span>

> [!NOTE]
> <span data-ttu-id="420a1-158">macOS non supporta ASP.NET Core gRPC con TLS.</span><span class="sxs-lookup"><span data-stu-id="420a1-158">macOS doesn't support ASP.NET Core gRPC with TLS.</span></span> <span data-ttu-id="420a1-159">Per eseguire correttamente i servizi gRPC in macOS, è necessaria una configurazione aggiuntiva.</span><span class="sxs-lookup"><span data-stu-id="420a1-159">Additional configuration is required to successfully run gRPC services on macOS.</span></span> <span data-ttu-id="420a1-160">Per altre informazioni, vedere [Non è possibile avviare un'app ASP.NET Core gRPC in macOS](xref:grpc/troubleshoot#unable-to-start-aspnet-core-grpc-app-on-macos).</span><span class="sxs-lookup"><span data-stu-id="420a1-160">For more information, see [Unable to start ASP.NET Core gRPC app on macOS](xref:grpc/troubleshoot#unable-to-start-aspnet-core-grpc-app-on-macos).</span></span>

## <a name="iis"></a><span data-ttu-id="420a1-161">IIS</span><span class="sxs-lookup"><span data-stu-id="420a1-161">IIS</span></span>

<span data-ttu-id="420a1-162">[Internet Information Services (IIS)](xref:host-and-deploy/iis/index) è un server Web flessibile, sicuro e gestibile per l'hosting di app Web, incluso ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="420a1-162">[Internet Information Services (IIS)](xref:host-and-deploy/iis/index) is a flexible, secure and manageable Web Server for hosting web apps, including ASP.NET Core.</span></span> <span data-ttu-id="420a1-163">.NET 5 e Windows 10 Build 20241 o versione successiva sono necessari per ospitare i servizi gRPC con IIS.</span><span class="sxs-lookup"><span data-stu-id="420a1-163">.NET 5 and Windows 10 Build 20241 or later are required to host gRPC services with IIS.</span></span>

<span data-ttu-id="420a1-164">IIS deve essere configurato per l'uso di TLS e HTTP/2.</span><span class="sxs-lookup"><span data-stu-id="420a1-164">IIS must be configured to use TLS and HTTP/2.</span></span> <span data-ttu-id="420a1-165">Per altre informazioni, vedere <xref:host-and-deploy/iis/protocols>.</span><span class="sxs-lookup"><span data-stu-id="420a1-165">For more information, see <xref:host-and-deploy/iis/protocols>.</span></span>

## <a name="httpsys"></a><span data-ttu-id="420a1-166">HTTP.sys</span><span class="sxs-lookup"><span data-stu-id="420a1-166">HTTP.sys</span></span>

<span data-ttu-id="420a1-167">[HTTP.sys](xref:fundamentals/servers/httpsys) è un server Web per ASP.NET Core che può essere eseguito solo in Windows.</span><span class="sxs-lookup"><span data-stu-id="420a1-167">[HTTP.sys](xref:fundamentals/servers/httpsys) is a web server for ASP.NET Core that only runs on Windows.</span></span> <span data-ttu-id="420a1-168">.NET 5 e Windows 10 Build 20241 o versione successiva sono necessari per ospitare i servizi gRPC con HTTP.sys.</span><span class="sxs-lookup"><span data-stu-id="420a1-168">.NET 5 and Windows 10 Build 20241 or later are required to host gRPC services with HTTP.sys.</span></span>

<span data-ttu-id="420a1-169">HTTP.sys deve essere configurato per l'uso di TLS e HTTP/2.</span><span class="sxs-lookup"><span data-stu-id="420a1-169">HTTP.sys must be configured to use TLS and HTTP/2.</span></span> <span data-ttu-id="420a1-170">Per ulteriori informazioni, vedere  [HTTP.sys supporto http/2 del server Web](xref:fundamentals/servers/httpsys#http2-support).</span><span class="sxs-lookup"><span data-stu-id="420a1-170">For more information, see  [HTTP.sys web server HTTP/2 support](xref:fundamentals/servers/httpsys#http2-support).</span></span>

::: moniker-end

::: moniker range="< aspnetcore-5.0"

## <a name="kestrel"></a><span data-ttu-id="420a1-171">Kestrel</span><span class="sxs-lookup"><span data-stu-id="420a1-171">Kestrel</span></span>

<span data-ttu-id="420a1-172">[Gheppio](xref:fundamentals/servers/kestrel) è un server Web multipiattaforma per ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="420a1-172">[Kestrel](xref:fundamentals/servers/kestrel) is a cross-platform web server for ASP.NET Core.</span></span> <span data-ttu-id="420a1-173">Gheppio fornisce le prestazioni migliori e l'utilizzo della memoria, ma non alcune delle funzionalità avanzate di HTTP.sys come la condivisione delle porte.</span><span class="sxs-lookup"><span data-stu-id="420a1-173">Kestrel provides the best performance and memory utilization, but it doesn't have some of the advanced features in HTTP.sys such as port sharing.</span></span>

<span data-ttu-id="420a1-174">Endpoint gRPC di Gheppio:</span><span class="sxs-lookup"><span data-stu-id="420a1-174">Kestrel gRPC endpoints:</span></span>

* <span data-ttu-id="420a1-175">Richiedi HTTP/2.</span><span class="sxs-lookup"><span data-stu-id="420a1-175">Require HTTP/2.</span></span>
* <span data-ttu-id="420a1-176">Deve essere protetto con [Transport Layer Security (TLS)](https://tools.ietf.org/html/rfc5246).</span><span class="sxs-lookup"><span data-stu-id="420a1-176">Should be secured with [Transport Layer Security (TLS)](https://tools.ietf.org/html/rfc5246).</span></span>

### <a name="http2"></a><span data-ttu-id="420a1-177">HTTP/2</span><span class="sxs-lookup"><span data-stu-id="420a1-177">HTTP/2</span></span>

<span data-ttu-id="420a1-178">gRPC richiede HTTP/2.</span><span class="sxs-lookup"><span data-stu-id="420a1-178">gRPC requires HTTP/2.</span></span> <span data-ttu-id="420a1-179">gRPC per ASP.NET Core convalida [HttpRequest. Protocol](xref:Microsoft.AspNetCore.Http.HttpRequest.Protocol%2A) è `HTTP/2` .</span><span class="sxs-lookup"><span data-stu-id="420a1-179">gRPC for ASP.NET Core validates [HttpRequest.Protocol](xref:Microsoft.AspNetCore.Http.HttpRequest.Protocol%2A) is `HTTP/2`.</span></span>

<span data-ttu-id="420a1-180">Gheppio [supporta http/2](xref:fundamentals/servers/kestrel#http2-support) nei sistemi operativi più recenti.</span><span class="sxs-lookup"><span data-stu-id="420a1-180">Kestrel [supports HTTP/2](xref:fundamentals/servers/kestrel#http2-support) on most modern operating systems.</span></span> <span data-ttu-id="420a1-181">Per impostazione predefinita, gli endpoint gheppio sono configurati per supportare connessioni HTTP/1.1 e HTTP/2.</span><span class="sxs-lookup"><span data-stu-id="420a1-181">Kestrel endpoints are configured to support HTTP/1.1 and HTTP/2 connections by default.</span></span>

### <a name="tls"></a><span data-ttu-id="420a1-182">TLS</span><span class="sxs-lookup"><span data-stu-id="420a1-182">TLS</span></span>

<span data-ttu-id="420a1-183">Gli endpoint gheppio usati per gRPC devono essere protetti con TLS.</span><span class="sxs-lookup"><span data-stu-id="420a1-183">Kestrel endpoints used for gRPC should be secured with TLS.</span></span> <span data-ttu-id="420a1-184">In fase di sviluppo, un endpoint protetto con TLS viene creato automaticamente in corrispondenza del momento in `https://localhost:5001` cui è presente il certificato di sviluppo ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="420a1-184">In development, an endpoint secured with TLS is automatically created at `https://localhost:5001` when the ASP.NET Core development certificate is present.</span></span> <span data-ttu-id="420a1-185">Non è richiesta alcuna configurazione.</span><span class="sxs-lookup"><span data-stu-id="420a1-185">No configuration is required.</span></span> <span data-ttu-id="420a1-186">Un `https` prefisso verifica che l'endpoint gheppio stia usando TLS.</span><span class="sxs-lookup"><span data-stu-id="420a1-186">An `https` prefix verifies the Kestrel endpoint is using TLS.</span></span>

<span data-ttu-id="420a1-187">In produzione, è necessario configurare in modo esplicito TLS.</span><span class="sxs-lookup"><span data-stu-id="420a1-187">In production, TLS must be explicitly configured.</span></span> <span data-ttu-id="420a1-188">Nell'esempio seguente *appsettings.json* viene fornito un endpoint HTTP/2 protetto con TLS:</span><span class="sxs-lookup"><span data-stu-id="420a1-188">In the following *appsettings.json* example, an HTTP/2 endpoint secured with TLS is provided:</span></span>

[!code-json[](~/grpc/aspnetcore/sample/appsettings.json?highlight=4)]

<span data-ttu-id="420a1-189">In alternativa, è possibile configurare gli endpoint gheppio in *Program.cs*:</span><span class="sxs-lookup"><span data-stu-id="420a1-189">Alternatively, Kestrel endpoints can be configured in *Program.cs*:</span></span>

[!code-csharp[](~/grpc/aspnetcore/sample/Program.cs?highlight=7&name=snippet)]

### <a name="protocol-negotiation"></a><span data-ttu-id="420a1-190">Negoziazione del protocollo</span><span class="sxs-lookup"><span data-stu-id="420a1-190">Protocol negotiation</span></span>

<span data-ttu-id="420a1-191">TLS viene usato per una maggiore sicurezza della comunicazione.</span><span class="sxs-lookup"><span data-stu-id="420a1-191">TLS is used for more than securing communication.</span></span> <span data-ttu-id="420a1-192">L'handshake TLS [(ALPN)](https://tools.ietf.org/html/rfc7301#section-3) viene utilizzato per negoziare il protocollo di connessione tra il client e il server quando un endpoint supporta più protocolli.</span><span class="sxs-lookup"><span data-stu-id="420a1-192">The TLS [Application-Layer Protocol Negotiation (ALPN)](https://tools.ietf.org/html/rfc7301#section-3) handshake is used to negotiate the connection protocol between the client and the server when an endpoint supports multiple protocols.</span></span> <span data-ttu-id="420a1-193">Questa negoziazione determina se la connessione utilizza HTTP/1.1 o HTTP/2.</span><span class="sxs-lookup"><span data-stu-id="420a1-193">This negotiation determines whether the connection uses HTTP/1.1 or HTTP/2.</span></span>

<span data-ttu-id="420a1-194">Se un endpoint HTTP/2 viene configurato senza TLS, l'endpoint [ListenOptions. Protocols](xref:fundamentals/servers/kestrel#listenoptionsprotocols) deve essere impostato su `HttpProtocols.Http2` .</span><span class="sxs-lookup"><span data-stu-id="420a1-194">If an HTTP/2 endpoint is configured without TLS, the endpoint's [ListenOptions.Protocols](xref:fundamentals/servers/kestrel#listenoptionsprotocols) must be set to `HttpProtocols.Http2`.</span></span> <span data-ttu-id="420a1-195">Un endpoint con più protocolli (ad esempio, `HttpProtocols.Http1AndHttp2` ) non può essere usato senza TLS perché non esiste alcuna negoziazione.</span><span class="sxs-lookup"><span data-stu-id="420a1-195">An endpoint with multiple protocols (for example, `HttpProtocols.Http1AndHttp2`) can't be used without TLS because there's no negotiation.</span></span> <span data-ttu-id="420a1-196">Per impostazione predefinita, tutte le connessioni all'endpoint non protetto sono HTTP/1.1 e le chiamate a gRPC hanno esito negativo.</span><span class="sxs-lookup"><span data-stu-id="420a1-196">All connections to the unsecured endpoint default to HTTP/1.1, and gRPC calls fail.</span></span>

<span data-ttu-id="420a1-197">Per ulteriori informazioni sull'abilitazione di HTTP/2 e TLS con gheppio, vedere la pagina relativa alla [configurazione dell'endpoint gheppio](xref:fundamentals/servers/kestrel#endpoint-configuration).</span><span class="sxs-lookup"><span data-stu-id="420a1-197">For more information on enabling HTTP/2 and TLS with Kestrel, see [Kestrel endpoint configuration](xref:fundamentals/servers/kestrel#endpoint-configuration).</span></span>

> [!NOTE]
> <span data-ttu-id="420a1-198">macOS non supporta ASP.NET Core gRPC con TLS.</span><span class="sxs-lookup"><span data-stu-id="420a1-198">macOS doesn't support ASP.NET Core gRPC with TLS.</span></span> <span data-ttu-id="420a1-199">Per eseguire correttamente i servizi gRPC in macOS, è necessaria una configurazione aggiuntiva.</span><span class="sxs-lookup"><span data-stu-id="420a1-199">Additional configuration is required to successfully run gRPC services on macOS.</span></span> <span data-ttu-id="420a1-200">Per altre informazioni, vedere [Non è possibile avviare un'app ASP.NET Core gRPC in macOS](xref:grpc/troubleshoot#unable-to-start-aspnet-core-grpc-app-on-macos).</span><span class="sxs-lookup"><span data-stu-id="420a1-200">For more information, see [Unable to start ASP.NET Core gRPC app on macOS](xref:grpc/troubleshoot#unable-to-start-aspnet-core-grpc-app-on-macos).</span></span>

::: moniker-end

## <a name="integration-with-aspnet-core-apis"></a><span data-ttu-id="420a1-201">Integrazione con API ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="420a1-201">Integration with ASP.NET Core APIs</span></span>

<span data-ttu-id="420a1-202">i servizi gRPC hanno accesso completo alle funzionalità di ASP.NET Core come l' [inserimento delle dipendenze](xref:fundamentals/dependency-injection) e la [registrazione](xref:fundamentals/logging/index).</span><span class="sxs-lookup"><span data-stu-id="420a1-202">gRPC services have full access to the ASP.NET Core features such as [Dependency Injection](xref:fundamentals/dependency-injection) (DI) and [Logging](xref:fundamentals/logging/index).</span></span> <span data-ttu-id="420a1-203">Ad esempio, l'implementazione del servizio può risolvere un servizio logger dal contenitore DI inserimento delle dipendenze tramite il costruttore:</span><span class="sxs-lookup"><span data-stu-id="420a1-203">For example, the service implementation can resolve a logger service from the DI container via the constructor:</span></span>

```csharp
public class GreeterService : Greeter.GreeterBase
{
    public GreeterService(ILogger<GreeterService> logger)
    {
    }
}
```

<span data-ttu-id="420a1-204">Per impostazione predefinita, l'implementazione del servizio gRPC è in grado di risolvere altri servizi DI con qualsiasi durata (singleton, ambito o temporaneo).</span><span class="sxs-lookup"><span data-stu-id="420a1-204">By default, the gRPC service implementation can resolve other DI services with any lifetime (Singleton, Scoped, or Transient).</span></span>

### <a name="resolve-httpcontext-in-grpc-methods"></a><span data-ttu-id="420a1-205">Risolvere HttpContext nei metodi gRPC</span><span class="sxs-lookup"><span data-stu-id="420a1-205">Resolve HttpContext in gRPC methods</span></span>

<span data-ttu-id="420a1-206">L'API gRPC consente di accedere ad alcuni dati del messaggio HTTP/2, ad esempio il metodo, l'host, l'intestazione e i trailer.</span><span class="sxs-lookup"><span data-stu-id="420a1-206">The gRPC API provides access to some HTTP/2 message data, such as the method, host, header, and trailers.</span></span> <span data-ttu-id="420a1-207">L'accesso avviene tramite l' `ServerCallContext` argomento passato a ogni metodo gRPC:</span><span class="sxs-lookup"><span data-stu-id="420a1-207">Access is through the `ServerCallContext` argument passed to each gRPC method:</span></span>

[!code-csharp[](~/grpc/aspnetcore/sample/GrcpService/GreeterService.cs?highlight=3-4&name=snippet)]

<span data-ttu-id="420a1-208">`ServerCallContext` non fornisce l'accesso completo a `HttpContext` in tutte le API di ASP.NET.</span><span class="sxs-lookup"><span data-stu-id="420a1-208">`ServerCallContext` doesn't provide full access to `HttpContext` in all ASP.NET APIs.</span></span> <span data-ttu-id="420a1-209">Il `GetHttpContext` metodo di estensione fornisce accesso completo a `HttpContext` che rappresenta il messaggio http/2 sottostante nelle API ASP.NET:</span><span class="sxs-lookup"><span data-stu-id="420a1-209">The `GetHttpContext` extension method provides full access to the `HttpContext` representing the underlying HTTP/2 message in ASP.NET APIs:</span></span>

[!code-csharp[](~/grpc/aspnetcore/sample/GrcpService/GreeterService2.cs?highlight=6-7&name=snippet)]


## <a name="additional-resources"></a><span data-ttu-id="420a1-210">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="420a1-210">Additional resources</span></span>

* <xref:tutorials/grpc/grpc-start>
* <xref:grpc/index>
* <xref:grpc/basics>
* <xref:fundamentals/servers/kestrel>
