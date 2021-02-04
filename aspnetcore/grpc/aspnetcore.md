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
ms.openlocfilehash: f17ba247747f906cf026fc0f7bc04d51f4c8cb2a
ms.sourcegitcommit: e311cfb77f26a0a23681019bd334929d1aaeda20
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/03/2021
ms.locfileid: "99530203"
---
# <a name="grpc-services-with-aspnet-core"></a><span data-ttu-id="c6be3-103">Servizi gRPC con ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="c6be3-103">gRPC services with ASP.NET Core</span></span>

<span data-ttu-id="c6be3-104">Questo documento illustra come iniziare a usare i servizi di gRPC con ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="c6be3-104">This document shows how to get started with gRPC services using ASP.NET Core.</span></span>

[!INCLUDE[](~/includes/gRPCazure.md)]

## <a name="prerequisites"></a><span data-ttu-id="c6be3-105">Prerequisiti</span><span class="sxs-lookup"><span data-stu-id="c6be3-105">Prerequisites</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="c6be3-106">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="c6be3-106">Visual Studio</span></span>](#tab/visual-studio)

[!INCLUDE[](~/includes/net-core-prereqs-vs-3.0.md)]

# <a name="visual-studio-code"></a>[<span data-ttu-id="c6be3-107">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="c6be3-107">Visual Studio Code</span></span>](#tab/visual-studio-code)

[!INCLUDE[](~/includes/net-core-prereqs-vsc-3.0.md)]

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="c6be3-108">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="c6be3-108">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

[!INCLUDE[](~/includes/net-core-prereqs-mac-3.0.md)]

---

## <a name="get-started-with-grpc-service-in-aspnet-core"></a><span data-ttu-id="c6be3-109">Introduzione all'uso del servizio gRPC in ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="c6be3-109">Get started with gRPC service in ASP.NET Core</span></span>

<span data-ttu-id="c6be3-110">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/tutorials/grpc/grpc-start/sample) ([procedura per il download](xref:index#how-to-download-a-sample)).</span><span class="sxs-lookup"><span data-stu-id="c6be3-110">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/tutorials/grpc/grpc-start/sample) ([how to download](xref:index#how-to-download-a-sample)).</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="c6be3-111">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="c6be3-111">Visual Studio</span></span>](#tab/visual-studio)

<span data-ttu-id="c6be3-112">Per istruzioni dettagliate su come creare un progetto gRPC, vedere [Introduzione ai servizi gRPC](xref:tutorials/grpc/grpc-start) .</span><span class="sxs-lookup"><span data-stu-id="c6be3-112">See [Get started with gRPC services](xref:tutorials/grpc/grpc-start) for detailed instructions on how to create a gRPC project.</span></span>

# <a name="visual-studio-code--visual-studio-for-mac"></a>[<span data-ttu-id="c6be3-113">Visual Studio Code / Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="c6be3-113">Visual Studio Code / Visual Studio for Mac</span></span>](#tab/visual-studio-code+visual-studio-mac)

<span data-ttu-id="c6be3-114">Eseguire `dotnet new grpc -o GrpcGreeter` dalla riga di comando.</span><span class="sxs-lookup"><span data-stu-id="c6be3-114">Run `dotnet new grpc -o GrpcGreeter` from the command line.</span></span>

---

## <a name="add-grpc-services-to-an-aspnet-core-app"></a><span data-ttu-id="c6be3-115">Aggiungere servizi gRPC a un'app ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="c6be3-115">Add gRPC services to an ASP.NET Core app</span></span>

<span data-ttu-id="c6be3-116">gRPC richiede il pacchetto [gRPC. AspNetCore](https://www.nuget.org/packages/Grpc.AspNetCore) .</span><span class="sxs-lookup"><span data-stu-id="c6be3-116">gRPC requires the [Grpc.AspNetCore](https://www.nuget.org/packages/Grpc.AspNetCore) package.</span></span>

### <a name="configure-grpc"></a><span data-ttu-id="c6be3-117">Configurare gRPC</span><span class="sxs-lookup"><span data-stu-id="c6be3-117">Configure gRPC</span></span>

<span data-ttu-id="c6be3-118">In *Startup.cs*:</span><span class="sxs-lookup"><span data-stu-id="c6be3-118">In *Startup.cs*:</span></span>

* <span data-ttu-id="c6be3-119">gRPC è abilitato con il `AddGrpc` metodo.</span><span class="sxs-lookup"><span data-stu-id="c6be3-119">gRPC is enabled with the `AddGrpc` method.</span></span>
* <span data-ttu-id="c6be3-120">Ogni servizio gRPC viene aggiunto alla pipeline di routing tramite il `MapGrpcService` metodo.</span><span class="sxs-lookup"><span data-stu-id="c6be3-120">Each gRPC service is added to the routing pipeline through the `MapGrpcService` method.</span></span>

[!code-csharp[](~/tutorials/grpc/grpc-start/sample/GrpcGreeter/Startup.cs?name=snippet&highlight=7,24)]
[!INCLUDE[about the series](~/includes/code-comments-loc.md)]

<span data-ttu-id="c6be3-121">ASP.NET Core middleware e le funzionalità condividono la pipeline di routing, di conseguenza è possibile configurare un'app per gestire gestori di richieste aggiuntivi.</span><span class="sxs-lookup"><span data-stu-id="c6be3-121">ASP.NET Core middleware and features share the routing pipeline, therefore an app can be configured to serve additional request handlers.</span></span> <span data-ttu-id="c6be3-122">I gestori di richieste aggiuntivi, ad esempio i controller MVC, funzionano in parallelo con i servizi gRPC configurati.</span><span class="sxs-lookup"><span data-stu-id="c6be3-122">The additional request handlers, such as MVC controllers, work in parallel with the configured gRPC services.</span></span>

## <a name="server-options"></a><span data-ttu-id="c6be3-123">Opzioni server</span><span class="sxs-lookup"><span data-stu-id="c6be3-123">Server options</span></span>

<span data-ttu-id="c6be3-124">i servizi gRPC possono essere ospitati da tutti i server ASP.NET Core predefiniti.</span><span class="sxs-lookup"><span data-stu-id="c6be3-124">gRPC services can be hosted by all built-in ASP.NET Core servers.</span></span>

> [!div class="checklist"]
>
> * <span data-ttu-id="c6be3-125">Kestrel</span><span class="sxs-lookup"><span data-stu-id="c6be3-125">Kestrel</span></span>
> * <span data-ttu-id="c6be3-126">TestServer</span><span class="sxs-lookup"><span data-stu-id="c6be3-126">TestServer</span></span>
> * <span data-ttu-id="c6be3-127">IIS&dagger;</span><span class="sxs-lookup"><span data-stu-id="c6be3-127">IIS&dagger;</span></span>
> * <span data-ttu-id="c6be3-128">HTTP.sys&Dagger;</span><span class="sxs-lookup"><span data-stu-id="c6be3-128">HTTP.sys&Dagger;</span></span>

<span data-ttu-id="c6be3-129">&dagger;IIS richiede .NET 5 e Windows 10 Build 20241 o versione successiva.</span><span class="sxs-lookup"><span data-stu-id="c6be3-129">&dagger;IIS requires .NET 5 and Windows 10 Build 20241 or later.</span></span>

<span data-ttu-id="c6be3-130">&Dagger;HTTP.sys richiede .NET 5 e Windows 10 Build 19529 o versione successiva.</span><span class="sxs-lookup"><span data-stu-id="c6be3-130">&Dagger;HTTP.sys requires .NET 5 and Windows 10 Build 19529 or later.</span></span>

<span data-ttu-id="c6be3-131">Per ulteriori informazioni sulla scelta del server appropriato per un'app ASP.NET Core, vedere <xref:fundamentals/servers/index> .</span><span class="sxs-lookup"><span data-stu-id="c6be3-131">For more information about choosing the right server for an ASP.NET Core app, see <xref:fundamentals/servers/index>.</span></span>

::: moniker range=">= aspnetcore-5.0"

## <a name="kestrel"></a><span data-ttu-id="c6be3-132">Kestrel</span><span class="sxs-lookup"><span data-stu-id="c6be3-132">Kestrel</span></span>

<span data-ttu-id="c6be3-133">[Gheppio](xref:fundamentals/servers/kestrel) è un server Web multipiattaforma per ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="c6be3-133">[Kestrel](xref:fundamentals/servers/kestrel) is a cross-platform web server for ASP.NET Core.</span></span> <span data-ttu-id="c6be3-134">Gheppio fornisce le prestazioni migliori e l'utilizzo della memoria, ma non alcune delle funzionalità avanzate di HTTP.sys come la condivisione delle porte.</span><span class="sxs-lookup"><span data-stu-id="c6be3-134">Kestrel provides the best performance and memory utilization, but it doesn't have some of the advanced features in HTTP.sys such as port sharing.</span></span>

<span data-ttu-id="c6be3-135">Endpoint gRPC di Gheppio:</span><span class="sxs-lookup"><span data-stu-id="c6be3-135">Kestrel gRPC endpoints:</span></span>

* <span data-ttu-id="c6be3-136">Richiedi HTTP/2.</span><span class="sxs-lookup"><span data-stu-id="c6be3-136">Require HTTP/2.</span></span>
* <span data-ttu-id="c6be3-137">Deve essere protetto con [Transport Layer Security (TLS)](https://tools.ietf.org/html/rfc5246).</span><span class="sxs-lookup"><span data-stu-id="c6be3-137">Should be secured with [Transport Layer Security (TLS)](https://tools.ietf.org/html/rfc5246).</span></span>

### <a name="http2"></a><span data-ttu-id="c6be3-138">HTTP/2</span><span class="sxs-lookup"><span data-stu-id="c6be3-138">HTTP/2</span></span>

<span data-ttu-id="c6be3-139">gRPC richiede HTTP/2.</span><span class="sxs-lookup"><span data-stu-id="c6be3-139">gRPC requires HTTP/2.</span></span> <span data-ttu-id="c6be3-140">gRPC per ASP.NET Core convalida [HttpRequest. Protocol](xref:Microsoft.AspNetCore.Http.HttpRequest.Protocol%2A) è `HTTP/2` .</span><span class="sxs-lookup"><span data-stu-id="c6be3-140">gRPC for ASP.NET Core validates [HttpRequest.Protocol](xref:Microsoft.AspNetCore.Http.HttpRequest.Protocol%2A) is `HTTP/2`.</span></span>

<span data-ttu-id="c6be3-141">Gheppio [supporta http/2](xref:fundamentals/servers/kestrel/http2) nei sistemi operativi più recenti.</span><span class="sxs-lookup"><span data-stu-id="c6be3-141">Kestrel [supports HTTP/2](xref:fundamentals/servers/kestrel/http2) on most modern operating systems.</span></span> <span data-ttu-id="c6be3-142">Per impostazione predefinita, gli endpoint gheppio sono configurati per supportare connessioni HTTP/1.1 e HTTP/2.</span><span class="sxs-lookup"><span data-stu-id="c6be3-142">Kestrel endpoints are configured to support HTTP/1.1 and HTTP/2 connections by default.</span></span>

### <a name="tls"></a><span data-ttu-id="c6be3-143">TLS</span><span class="sxs-lookup"><span data-stu-id="c6be3-143">TLS</span></span>

<span data-ttu-id="c6be3-144">Gli endpoint gheppio usati per gRPC devono essere protetti con TLS.</span><span class="sxs-lookup"><span data-stu-id="c6be3-144">Kestrel endpoints used for gRPC should be secured with TLS.</span></span> <span data-ttu-id="c6be3-145">In fase di sviluppo, un endpoint protetto con TLS viene creato automaticamente in corrispondenza del momento in `https://localhost:5001` cui è presente il certificato di sviluppo ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="c6be3-145">In development, an endpoint secured with TLS is automatically created at `https://localhost:5001` when the ASP.NET Core development certificate is present.</span></span> <span data-ttu-id="c6be3-146">Non è richiesta alcuna configurazione.</span><span class="sxs-lookup"><span data-stu-id="c6be3-146">No configuration is required.</span></span> <span data-ttu-id="c6be3-147">Un `https` prefisso verifica che l'endpoint gheppio stia usando TLS.</span><span class="sxs-lookup"><span data-stu-id="c6be3-147">An `https` prefix verifies the Kestrel endpoint is using TLS.</span></span>

<span data-ttu-id="c6be3-148">In produzione, è necessario configurare in modo esplicito TLS.</span><span class="sxs-lookup"><span data-stu-id="c6be3-148">In production, TLS must be explicitly configured.</span></span> <span data-ttu-id="c6be3-149">Nell'esempio seguente *appsettings.json* viene fornito un endpoint HTTP/2 protetto con TLS:</span><span class="sxs-lookup"><span data-stu-id="c6be3-149">In the following *appsettings.json* example, an HTTP/2 endpoint secured with TLS is provided:</span></span>

[!code-json[](~/grpc/aspnetcore/sample/appsettings.json?highlight=4)]

<span data-ttu-id="c6be3-150">In alternativa, è possibile configurare gli endpoint gheppio in *Program.cs*:</span><span class="sxs-lookup"><span data-stu-id="c6be3-150">Alternatively, Kestrel endpoints can be configured in *Program.cs*:</span></span>

[!code-csharp[](~/grpc/aspnetcore/sample/Program.cs?highlight=7&name=snippet)]

### <a name="protocol-negotiation"></a><span data-ttu-id="c6be3-151">Negoziazione del protocollo</span><span class="sxs-lookup"><span data-stu-id="c6be3-151">Protocol negotiation</span></span>

<span data-ttu-id="c6be3-152">TLS viene usato per una maggiore sicurezza della comunicazione.</span><span class="sxs-lookup"><span data-stu-id="c6be3-152">TLS is used for more than securing communication.</span></span> <span data-ttu-id="c6be3-153">L'handshake TLS [(ALPN)](https://tools.ietf.org/html/rfc7301#section-3) viene utilizzato per negoziare il protocollo di connessione tra il client e il server quando un endpoint supporta più protocolli.</span><span class="sxs-lookup"><span data-stu-id="c6be3-153">The TLS [Application-Layer Protocol Negotiation (ALPN)](https://tools.ietf.org/html/rfc7301#section-3) handshake is used to negotiate the connection protocol between the client and the server when an endpoint supports multiple protocols.</span></span> <span data-ttu-id="c6be3-154">Questa negoziazione determina se la connessione utilizza HTTP/1.1 o HTTP/2.</span><span class="sxs-lookup"><span data-stu-id="c6be3-154">This negotiation determines whether the connection uses HTTP/1.1 or HTTP/2.</span></span>

<span data-ttu-id="c6be3-155">Se un endpoint HTTP/2 viene configurato senza TLS, l'endpoint [ListenOptions. Protocols](xref:fundamentals/servers/kestrel/endpoints#listenoptionsprotocols) deve essere impostato su `HttpProtocols.Http2` .</span><span class="sxs-lookup"><span data-stu-id="c6be3-155">If an HTTP/2 endpoint is configured without TLS, the endpoint's [ListenOptions.Protocols](xref:fundamentals/servers/kestrel/endpoints#listenoptionsprotocols) must be set to `HttpProtocols.Http2`.</span></span> <span data-ttu-id="c6be3-156">Un endpoint con più protocolli (ad esempio, `HttpProtocols.Http1AndHttp2` ) non può essere usato senza TLS perché non esiste alcuna negoziazione.</span><span class="sxs-lookup"><span data-stu-id="c6be3-156">An endpoint with multiple protocols (for example, `HttpProtocols.Http1AndHttp2`) can't be used without TLS because there's no negotiation.</span></span> <span data-ttu-id="c6be3-157">Per impostazione predefinita, tutte le connessioni all'endpoint non protetto sono HTTP/1.1 e le chiamate a gRPC hanno esito negativo.</span><span class="sxs-lookup"><span data-stu-id="c6be3-157">All connections to the unsecured endpoint default to HTTP/1.1, and gRPC calls fail.</span></span>

<span data-ttu-id="c6be3-158">Per ulteriori informazioni sull'abilitazione di HTTP/2 e TLS con gheppio, vedere la pagina relativa alla [configurazione dell'endpoint gheppio](xref:fundamentals/servers/kestrel/endpoints).</span><span class="sxs-lookup"><span data-stu-id="c6be3-158">For more information on enabling HTTP/2 and TLS with Kestrel, see [Kestrel endpoint configuration](xref:fundamentals/servers/kestrel/endpoints).</span></span>

> [!NOTE]
> <span data-ttu-id="c6be3-159">macOS non supporta ASP.NET Core gRPC con TLS.</span><span class="sxs-lookup"><span data-stu-id="c6be3-159">macOS doesn't support ASP.NET Core gRPC with TLS.</span></span> <span data-ttu-id="c6be3-160">Per eseguire correttamente i servizi gRPC in macOS, è necessaria una configurazione aggiuntiva.</span><span class="sxs-lookup"><span data-stu-id="c6be3-160">Additional configuration is required to successfully run gRPC services on macOS.</span></span> <span data-ttu-id="c6be3-161">Per altre informazioni, vedere [Non è possibile avviare un'app ASP.NET Core gRPC in macOS](xref:grpc/troubleshoot#unable-to-start-aspnet-core-grpc-app-on-macos).</span><span class="sxs-lookup"><span data-stu-id="c6be3-161">For more information, see [Unable to start ASP.NET Core gRPC app on macOS](xref:grpc/troubleshoot#unable-to-start-aspnet-core-grpc-app-on-macos).</span></span>

## <a name="iis"></a><span data-ttu-id="c6be3-162">IIS</span><span class="sxs-lookup"><span data-stu-id="c6be3-162">IIS</span></span>

<span data-ttu-id="c6be3-163">[Internet Information Services (IIS)](xref:host-and-deploy/iis/index) è un server Web flessibile, sicuro e gestibile per l'hosting di app Web, incluso ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="c6be3-163">[Internet Information Services (IIS)](xref:host-and-deploy/iis/index) is a flexible, secure and manageable Web Server for hosting web apps, including ASP.NET Core.</span></span> <span data-ttu-id="c6be3-164">.NET 5 e Windows 10 Build 20241 o versione successiva sono necessari per ospitare i servizi gRPC con IIS.</span><span class="sxs-lookup"><span data-stu-id="c6be3-164">.NET 5 and Windows 10 Build 20241 or later are required to host gRPC services with IIS.</span></span>

<span data-ttu-id="c6be3-165">IIS deve essere configurato per l'uso di TLS e HTTP/2.</span><span class="sxs-lookup"><span data-stu-id="c6be3-165">IIS must be configured to use TLS and HTTP/2.</span></span> <span data-ttu-id="c6be3-166">Per altre informazioni, vedere <xref:host-and-deploy/iis/protocols>.</span><span class="sxs-lookup"><span data-stu-id="c6be3-166">For more information, see <xref:host-and-deploy/iis/protocols>.</span></span>

## <a name="httpsys"></a><span data-ttu-id="c6be3-167">HTTP.sys</span><span class="sxs-lookup"><span data-stu-id="c6be3-167">HTTP.sys</span></span>

<span data-ttu-id="c6be3-168">[HTTP.sys](xref:fundamentals/servers/httpsys) è un server Web per ASP.NET Core che può essere eseguito solo in Windows.</span><span class="sxs-lookup"><span data-stu-id="c6be3-168">[HTTP.sys](xref:fundamentals/servers/httpsys) is a web server for ASP.NET Core that only runs on Windows.</span></span> <span data-ttu-id="c6be3-169">.NET 5 e Windows 10 Build 20241 o versione successiva sono necessari per ospitare i servizi gRPC con HTTP.sys.</span><span class="sxs-lookup"><span data-stu-id="c6be3-169">.NET 5 and Windows 10 Build 20241 or later are required to host gRPC services with HTTP.sys.</span></span>

<span data-ttu-id="c6be3-170">HTTP.sys deve essere configurato per l'uso di TLS e HTTP/2.</span><span class="sxs-lookup"><span data-stu-id="c6be3-170">HTTP.sys must be configured to use TLS and HTTP/2.</span></span> <span data-ttu-id="c6be3-171">Per ulteriori informazioni, vedere  [HTTP.sys supporto http/2 del server Web](xref:fundamentals/servers/httpsys#http2-support).</span><span class="sxs-lookup"><span data-stu-id="c6be3-171">For more information, see  [HTTP.sys web server HTTP/2 support](xref:fundamentals/servers/httpsys#http2-support).</span></span>

::: moniker-end

::: moniker range="< aspnetcore-5.0"

## <a name="kestrel"></a><span data-ttu-id="c6be3-172">Kestrel</span><span class="sxs-lookup"><span data-stu-id="c6be3-172">Kestrel</span></span>

<span data-ttu-id="c6be3-173">[Gheppio](xref:fundamentals/servers/kestrel) è un server Web multipiattaforma per ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="c6be3-173">[Kestrel](xref:fundamentals/servers/kestrel) is a cross-platform web server for ASP.NET Core.</span></span> <span data-ttu-id="c6be3-174">Gheppio fornisce le prestazioni migliori e l'utilizzo della memoria, ma non alcune delle funzionalità avanzate di HTTP.sys come la condivisione delle porte.</span><span class="sxs-lookup"><span data-stu-id="c6be3-174">Kestrel provides the best performance and memory utilization, but it doesn't have some of the advanced features in HTTP.sys such as port sharing.</span></span>

<span data-ttu-id="c6be3-175">Endpoint gRPC di Gheppio:</span><span class="sxs-lookup"><span data-stu-id="c6be3-175">Kestrel gRPC endpoints:</span></span>

* <span data-ttu-id="c6be3-176">Richiedi HTTP/2.</span><span class="sxs-lookup"><span data-stu-id="c6be3-176">Require HTTP/2.</span></span>
* <span data-ttu-id="c6be3-177">Deve essere protetto con [Transport Layer Security (TLS)](https://tools.ietf.org/html/rfc5246).</span><span class="sxs-lookup"><span data-stu-id="c6be3-177">Should be secured with [Transport Layer Security (TLS)](https://tools.ietf.org/html/rfc5246).</span></span>

### <a name="http2"></a><span data-ttu-id="c6be3-178">HTTP/2</span><span class="sxs-lookup"><span data-stu-id="c6be3-178">HTTP/2</span></span>

<span data-ttu-id="c6be3-179">gRPC richiede HTTP/2.</span><span class="sxs-lookup"><span data-stu-id="c6be3-179">gRPC requires HTTP/2.</span></span> <span data-ttu-id="c6be3-180">gRPC per ASP.NET Core convalida [HttpRequest. Protocol](xref:Microsoft.AspNetCore.Http.HttpRequest.Protocol%2A) è `HTTP/2` .</span><span class="sxs-lookup"><span data-stu-id="c6be3-180">gRPC for ASP.NET Core validates [HttpRequest.Protocol](xref:Microsoft.AspNetCore.Http.HttpRequest.Protocol%2A) is `HTTP/2`.</span></span>

<span data-ttu-id="c6be3-181">Gheppio [supporta http/2](xref:fundamentals/servers/kestrel#http2-support) nei sistemi operativi più recenti.</span><span class="sxs-lookup"><span data-stu-id="c6be3-181">Kestrel [supports HTTP/2](xref:fundamentals/servers/kestrel#http2-support) on most modern operating systems.</span></span> <span data-ttu-id="c6be3-182">Per impostazione predefinita, gli endpoint gheppio sono configurati per supportare connessioni HTTP/1.1 e HTTP/2.</span><span class="sxs-lookup"><span data-stu-id="c6be3-182">Kestrel endpoints are configured to support HTTP/1.1 and HTTP/2 connections by default.</span></span>

### <a name="tls"></a><span data-ttu-id="c6be3-183">TLS</span><span class="sxs-lookup"><span data-stu-id="c6be3-183">TLS</span></span>

<span data-ttu-id="c6be3-184">Gli endpoint gheppio usati per gRPC devono essere protetti con TLS.</span><span class="sxs-lookup"><span data-stu-id="c6be3-184">Kestrel endpoints used for gRPC should be secured with TLS.</span></span> <span data-ttu-id="c6be3-185">In fase di sviluppo, un endpoint protetto con TLS viene creato automaticamente in corrispondenza del momento in `https://localhost:5001` cui è presente il certificato di sviluppo ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="c6be3-185">In development, an endpoint secured with TLS is automatically created at `https://localhost:5001` when the ASP.NET Core development certificate is present.</span></span> <span data-ttu-id="c6be3-186">Non è richiesta alcuna configurazione.</span><span class="sxs-lookup"><span data-stu-id="c6be3-186">No configuration is required.</span></span> <span data-ttu-id="c6be3-187">Un `https` prefisso verifica che l'endpoint gheppio stia usando TLS.</span><span class="sxs-lookup"><span data-stu-id="c6be3-187">An `https` prefix verifies the Kestrel endpoint is using TLS.</span></span>

<span data-ttu-id="c6be3-188">In produzione, è necessario configurare in modo esplicito TLS.</span><span class="sxs-lookup"><span data-stu-id="c6be3-188">In production, TLS must be explicitly configured.</span></span> <span data-ttu-id="c6be3-189">Nell'esempio seguente *appsettings.json* viene fornito un endpoint HTTP/2 protetto con TLS:</span><span class="sxs-lookup"><span data-stu-id="c6be3-189">In the following *appsettings.json* example, an HTTP/2 endpoint secured with TLS is provided:</span></span>

[!code-json[](~/grpc/aspnetcore/sample/appsettings.json?highlight=4)]

<span data-ttu-id="c6be3-190">In alternativa, è possibile configurare gli endpoint gheppio in *Program.cs*:</span><span class="sxs-lookup"><span data-stu-id="c6be3-190">Alternatively, Kestrel endpoints can be configured in *Program.cs*:</span></span>

[!code-csharp[](~/grpc/aspnetcore/sample/Program.cs?highlight=7&name=snippet)]

### <a name="protocol-negotiation"></a><span data-ttu-id="c6be3-191">Negoziazione del protocollo</span><span class="sxs-lookup"><span data-stu-id="c6be3-191">Protocol negotiation</span></span>

<span data-ttu-id="c6be3-192">TLS viene usato per una maggiore sicurezza della comunicazione.</span><span class="sxs-lookup"><span data-stu-id="c6be3-192">TLS is used for more than securing communication.</span></span> <span data-ttu-id="c6be3-193">L'handshake TLS [(ALPN)](https://tools.ietf.org/html/rfc7301#section-3) viene utilizzato per negoziare il protocollo di connessione tra il client e il server quando un endpoint supporta più protocolli.</span><span class="sxs-lookup"><span data-stu-id="c6be3-193">The TLS [Application-Layer Protocol Negotiation (ALPN)](https://tools.ietf.org/html/rfc7301#section-3) handshake is used to negotiate the connection protocol between the client and the server when an endpoint supports multiple protocols.</span></span> <span data-ttu-id="c6be3-194">Questa negoziazione determina se la connessione utilizza HTTP/1.1 o HTTP/2.</span><span class="sxs-lookup"><span data-stu-id="c6be3-194">This negotiation determines whether the connection uses HTTP/1.1 or HTTP/2.</span></span>

<span data-ttu-id="c6be3-195">Se un endpoint HTTP/2 viene configurato senza TLS, l'endpoint [ListenOptions. Protocols](xref:fundamentals/servers/kestrel#listenoptionsprotocols) deve essere impostato su `HttpProtocols.Http2` .</span><span class="sxs-lookup"><span data-stu-id="c6be3-195">If an HTTP/2 endpoint is configured without TLS, the endpoint's [ListenOptions.Protocols](xref:fundamentals/servers/kestrel#listenoptionsprotocols) must be set to `HttpProtocols.Http2`.</span></span> <span data-ttu-id="c6be3-196">Un endpoint con più protocolli (ad esempio, `HttpProtocols.Http1AndHttp2` ) non può essere usato senza TLS perché non esiste alcuna negoziazione.</span><span class="sxs-lookup"><span data-stu-id="c6be3-196">An endpoint with multiple protocols (for example, `HttpProtocols.Http1AndHttp2`) can't be used without TLS because there's no negotiation.</span></span> <span data-ttu-id="c6be3-197">Per impostazione predefinita, tutte le connessioni all'endpoint non protetto sono HTTP/1.1 e le chiamate a gRPC hanno esito negativo.</span><span class="sxs-lookup"><span data-stu-id="c6be3-197">All connections to the unsecured endpoint default to HTTP/1.1, and gRPC calls fail.</span></span>

<span data-ttu-id="c6be3-198">Per ulteriori informazioni sull'abilitazione di HTTP/2 e TLS con gheppio, vedere la pagina relativa alla [configurazione dell'endpoint gheppio](xref:fundamentals/servers/kestrel#endpoint-configuration).</span><span class="sxs-lookup"><span data-stu-id="c6be3-198">For more information on enabling HTTP/2 and TLS with Kestrel, see [Kestrel endpoint configuration](xref:fundamentals/servers/kestrel#endpoint-configuration).</span></span>

> [!NOTE]
> <span data-ttu-id="c6be3-199">macOS non supporta ASP.NET Core gRPC con TLS.</span><span class="sxs-lookup"><span data-stu-id="c6be3-199">macOS doesn't support ASP.NET Core gRPC with TLS.</span></span> <span data-ttu-id="c6be3-200">Per eseguire correttamente i servizi gRPC in macOS, è necessaria una configurazione aggiuntiva.</span><span class="sxs-lookup"><span data-stu-id="c6be3-200">Additional configuration is required to successfully run gRPC services on macOS.</span></span> <span data-ttu-id="c6be3-201">Per altre informazioni, vedere [Non è possibile avviare un'app ASP.NET Core gRPC in macOS](xref:grpc/troubleshoot#unable-to-start-aspnet-core-grpc-app-on-macos).</span><span class="sxs-lookup"><span data-stu-id="c6be3-201">For more information, see [Unable to start ASP.NET Core gRPC app on macOS](xref:grpc/troubleshoot#unable-to-start-aspnet-core-grpc-app-on-macos).</span></span>

::: moniker-end

## <a name="integration-with-aspnet-core-apis"></a><span data-ttu-id="c6be3-202">Integrazione con API ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="c6be3-202">Integration with ASP.NET Core APIs</span></span>

<span data-ttu-id="c6be3-203">i servizi gRPC hanno accesso completo alle funzionalità di ASP.NET Core come l' [inserimento delle dipendenze](xref:fundamentals/dependency-injection) e la [registrazione](xref:fundamentals/logging/index).</span><span class="sxs-lookup"><span data-stu-id="c6be3-203">gRPC services have full access to the ASP.NET Core features such as [Dependency Injection](xref:fundamentals/dependency-injection) (DI) and [Logging](xref:fundamentals/logging/index).</span></span> <span data-ttu-id="c6be3-204">Ad esempio, l'implementazione del servizio può risolvere un servizio logger dal contenitore DI inserimento delle dipendenze tramite il costruttore:</span><span class="sxs-lookup"><span data-stu-id="c6be3-204">For example, the service implementation can resolve a logger service from the DI container via the constructor:</span></span>

```csharp
public class GreeterService : Greeter.GreeterBase
{
    public GreeterService(ILogger<GreeterService> logger)
    {
    }
}
```

<span data-ttu-id="c6be3-205">Per impostazione predefinita, l'implementazione del servizio gRPC è in grado di risolvere altri servizi DI con qualsiasi durata (singleton, ambito o temporaneo).</span><span class="sxs-lookup"><span data-stu-id="c6be3-205">By default, the gRPC service implementation can resolve other DI services with any lifetime (Singleton, Scoped, or Transient).</span></span>

### <a name="resolve-httpcontext-in-grpc-methods"></a><span data-ttu-id="c6be3-206">Risolvere HttpContext nei metodi gRPC</span><span class="sxs-lookup"><span data-stu-id="c6be3-206">Resolve HttpContext in gRPC methods</span></span>

<span data-ttu-id="c6be3-207">L'API gRPC consente di accedere ad alcuni dati del messaggio HTTP/2, ad esempio il metodo, l'host, l'intestazione e i trailer.</span><span class="sxs-lookup"><span data-stu-id="c6be3-207">The gRPC API provides access to some HTTP/2 message data, such as the method, host, header, and trailers.</span></span> <span data-ttu-id="c6be3-208">L'accesso avviene tramite l' `ServerCallContext` argomento passato a ogni metodo gRPC:</span><span class="sxs-lookup"><span data-stu-id="c6be3-208">Access is through the `ServerCallContext` argument passed to each gRPC method:</span></span>

[!code-csharp[](~/grpc/aspnetcore/sample/GrcpService/GreeterService.cs?highlight=3-4&name=snippet)]

<span data-ttu-id="c6be3-209">`ServerCallContext` non fornisce l'accesso completo a `HttpContext` in tutte le API di ASP.NET.</span><span class="sxs-lookup"><span data-stu-id="c6be3-209">`ServerCallContext` doesn't provide full access to `HttpContext` in all ASP.NET APIs.</span></span> <span data-ttu-id="c6be3-210">Il `GetHttpContext` metodo di estensione fornisce accesso completo a `HttpContext` che rappresenta il messaggio http/2 sottostante nelle API ASP.NET:</span><span class="sxs-lookup"><span data-stu-id="c6be3-210">The `GetHttpContext` extension method provides full access to the `HttpContext` representing the underlying HTTP/2 message in ASP.NET APIs:</span></span>

[!code-csharp[](~/grpc/aspnetcore/sample/GrcpService/GreeterService2.cs?highlight=6-7&name=snippet)]


## <a name="additional-resources"></a><span data-ttu-id="c6be3-211">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="c6be3-211">Additional resources</span></span>

* <xref:tutorials/grpc/grpc-start>
* <xref:grpc/index>
* <xref:grpc/basics>
* <xref:fundamentals/servers/kestrel>
