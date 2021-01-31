---
title: Usare gRPC nelle app del browser
author: jamesnk
description: Informazioni su come configurare i servizi gRPC in ASP.NET Core per essere richiamabili dalle app del browser usando gRPC-Web.
monikerRange: '>= aspnetcore-3.0'
ms.author: jamesnk
ms.date: 06/30/2020
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
uid: grpc/browser
ms.openlocfilehash: 0967a70b498156d9c4ea8818ee1c80b37d9f2d87
ms.sourcegitcommit: 7e394a8527c9818caebb940f692ae4fcf2f1b277
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 01/31/2021
ms.locfileid: "99217479"
---
# <a name="use-grpc-in-browser-apps"></a><span data-ttu-id="dd2c8-103">Usare gRPC nelle app del browser</span><span class="sxs-lookup"><span data-stu-id="dd2c8-103">Use gRPC in browser apps</span></span>

<span data-ttu-id="dd2c8-104">Di [James Newton-King](https://twitter.com/jamesnk)</span><span class="sxs-lookup"><span data-stu-id="dd2c8-104">By [James Newton-King](https://twitter.com/jamesnk)</span></span>

 <span data-ttu-id="dd2c8-105">Informazioni su come configurare un servizio di ASP.NET Core gRPC esistente da chiamare dalle app del browser, usando il protocollo [gRPC-Web](https://github.com/grpc/grpc/blob/2a388793792cc80944334535b7c729494d209a7e/doc/PROTOCOL-WEB.md) .</span><span class="sxs-lookup"><span data-stu-id="dd2c8-105">Learn how to configure an existing ASP.NET Core gRPC service to be callable from browser apps, using the [gRPC-Web](https://github.com/grpc/grpc/blob/2a388793792cc80944334535b7c729494d209a7e/doc/PROTOCOL-WEB.md) protocol.</span></span> <span data-ttu-id="dd2c8-106">gRPC-Web consente alle app e JavaScript del browser Blazor di chiamare i servizi gRPC.</span><span class="sxs-lookup"><span data-stu-id="dd2c8-106">gRPC-Web allows browser JavaScript and Blazor apps to call gRPC services.</span></span> <span data-ttu-id="dd2c8-107">Non è possibile chiamare un servizio HTTP/2 gRPC da un'app basata su browser.</span><span class="sxs-lookup"><span data-stu-id="dd2c8-107">It's not possible to call an HTTP/2 gRPC service from a browser-based app.</span></span> <span data-ttu-id="dd2c8-108">i servizi gRPC ospitati in ASP.NET Core possono essere configurati per supportare gRPC-Web insieme a HTTP/2 gRPC.</span><span class="sxs-lookup"><span data-stu-id="dd2c8-108">gRPC services hosted in ASP.NET Core can be configured to support gRPC-Web alongside HTTP/2 gRPC.</span></span>


<span data-ttu-id="dd2c8-109">Per istruzioni sull'aggiunta di un servizio gRPC a un'app ASP.NET Core esistente, vedere [aggiungere servizi gRPC a un'app ASP.NET Core](xref:grpc/aspnetcore#add-grpc-services-to-an-aspnet-core-app).</span><span class="sxs-lookup"><span data-stu-id="dd2c8-109">For instructions on adding a gRPC service to an existing ASP.NET Core app, see [Add gRPC services to an ASP.NET Core app](xref:grpc/aspnetcore#add-grpc-services-to-an-aspnet-core-app).</span></span>

<span data-ttu-id="dd2c8-110">Per istruzioni sulla creazione di un progetto gRPC, vedere <xref:tutorials/grpc/grpc-start> .</span><span class="sxs-lookup"><span data-stu-id="dd2c8-110">For instructions on creating a gRPC project, see <xref:tutorials/grpc/grpc-start>.</span></span>

## <a name="grpc-web-in-aspnet-core-vs-envoy"></a><span data-ttu-id="dd2c8-111">gRPC-Web in ASP.NET Core e inviato</span><span class="sxs-lookup"><span data-stu-id="dd2c8-111">gRPC-Web in ASP.NET Core vs. Envoy</span></span>

<span data-ttu-id="dd2c8-112">Sono disponibili due opzioni per l'aggiunta di gRPC-Web a un'app ASP.NET Core:</span><span class="sxs-lookup"><span data-stu-id="dd2c8-112">There are two choices for how to add gRPC-Web to an ASP.NET Core app:</span></span>

* <span data-ttu-id="dd2c8-113">Supportare gRPC-Web insieme a gRPC HTTP/2 in ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="dd2c8-113">Support gRPC-Web alongside gRPC HTTP/2 in ASP.NET Core.</span></span> <span data-ttu-id="dd2c8-114">Questa opzione utilizza il middleware fornito dal `Grpc.AspNetCore.Web` pacchetto.</span><span class="sxs-lookup"><span data-stu-id="dd2c8-114">This option uses middleware provided by the `Grpc.AspNetCore.Web` package.</span></span>
* <span data-ttu-id="dd2c8-115">Usare il supporto gRPC-Web del [proxy inviato](https://www.envoyproxy.io/) per tradurre GRPC-Web in gRPC http/2.</span><span class="sxs-lookup"><span data-stu-id="dd2c8-115">Use the [Envoy proxy's](https://www.envoyproxy.io/) gRPC-Web support to translate gRPC-Web to gRPC HTTP/2.</span></span> <span data-ttu-id="dd2c8-116">La chiamata tradotta viene quindi trasmessa nell'app ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="dd2c8-116">The translated call is then forwarded onto the ASP.NET Core app.</span></span>

<span data-ttu-id="dd2c8-117">Ogni approccio presenta vantaggi e svantaggi.</span><span class="sxs-lookup"><span data-stu-id="dd2c8-117">There are pros and cons to each approach.</span></span> <span data-ttu-id="dd2c8-118">Se l'ambiente di un'app usa già l'inviato come proxy, potrebbe essere utile usare anche l'inviato per fornire il supporto gRPC-Web.</span><span class="sxs-lookup"><span data-stu-id="dd2c8-118">If an app's environment is already using Envoy as a proxy, it might make sense to also use Envoy to provide gRPC-Web support.</span></span> <span data-ttu-id="dd2c8-119">Per una soluzione di base per gRPC-Web che richiede solo ASP.NET Core, `Grpc.AspNetCore.Web` è una scelta ottimale.</span><span class="sxs-lookup"><span data-stu-id="dd2c8-119">For a basic solution for gRPC-Web that only requires ASP.NET Core, `Grpc.AspNetCore.Web` is a good choice.</span></span>

## <a name="configure-grpc-web-in-aspnet-core"></a><span data-ttu-id="dd2c8-120">Configurare gRPC-Web in ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="dd2c8-120">Configure gRPC-Web in ASP.NET Core</span></span>

<span data-ttu-id="dd2c8-121">i servizi gRPC ospitati in ASP.NET Core possono essere configurati per supportare gRPC-Web insieme a HTTP/2 gRPC.</span><span class="sxs-lookup"><span data-stu-id="dd2c8-121">gRPC services hosted in ASP.NET Core can be configured to support gRPC-Web alongside HTTP/2 gRPC.</span></span> <span data-ttu-id="dd2c8-122">gRPC-Web non richiede alcuna modifica ai servizi.</span><span class="sxs-lookup"><span data-stu-id="dd2c8-122">gRPC-Web does not require any changes to services.</span></span> <span data-ttu-id="dd2c8-123">L'unica modifica è la configurazione di avvio.</span><span class="sxs-lookup"><span data-stu-id="dd2c8-123">The only modification is startup configuration.</span></span>

<span data-ttu-id="dd2c8-124">Per abilitare gRPC-Web con un servizio gRPC ASP.NET Core:</span><span class="sxs-lookup"><span data-stu-id="dd2c8-124">To enable gRPC-Web with an ASP.NET Core gRPC service:</span></span>

* <span data-ttu-id="dd2c8-125">Aggiungere un riferimento al pacchetto [Grpc. AspNetCore. Web](https://www.nuget.org/packages/Grpc.AspNetCore.Web) .</span><span class="sxs-lookup"><span data-stu-id="dd2c8-125">Add a reference to the [Grpc.AspNetCore.Web](https://www.nuget.org/packages/Grpc.AspNetCore.Web) package.</span></span>
* <span data-ttu-id="dd2c8-126">Configurare l'app per l'uso di gRPC-Web aggiungendo `UseGrpcWeb` e `EnableGrpcWeb` a *Startup.cs*:</span><span class="sxs-lookup"><span data-stu-id="dd2c8-126">Configure the app to use gRPC-Web by adding `UseGrpcWeb` and `EnableGrpcWeb` to *Startup.cs*:</span></span>

[!code-csharp[](~/grpc/browser/sample/Startup.cs?name=snippet_1&highlight=10,14)]

<span data-ttu-id="dd2c8-127">Il codice precedente:</span><span class="sxs-lookup"><span data-stu-id="dd2c8-127">The preceding code:</span></span>

* <span data-ttu-id="dd2c8-128">Aggiunge il middleware gRPC-Web, `UseGrpcWeb` , dopo il routing e prima degli endpoint.</span><span class="sxs-lookup"><span data-stu-id="dd2c8-128">Adds the gRPC-Web middleware, `UseGrpcWeb`, after routing and before endpoints.</span></span>
* <span data-ttu-id="dd2c8-129">Specifica che il `endpoints.MapGrpcService<GreeterService>()` metodo supporta gRPC-Web con `EnableGrpcWeb` .</span><span class="sxs-lookup"><span data-stu-id="dd2c8-129">Specifies the `endpoints.MapGrpcService<GreeterService>()` method supports gRPC-Web with `EnableGrpcWeb`.</span></span> 

<span data-ttu-id="dd2c8-130">In alternativa, è possibile configurare il middleware gRPC-Web in modo che tutti i servizi supportino gRPC-Web per impostazione predefinita e `EnableGrpcWeb` non siano necessari.</span><span class="sxs-lookup"><span data-stu-id="dd2c8-130">Alternatively, the gRPC-Web middleware can be configured so all services support gRPC-Web by default and `EnableGrpcWeb` isn't required.</span></span> <span data-ttu-id="dd2c8-131">Specificare `new GrpcWebOptions { DefaultEnabled = true }` quando viene aggiunto il middleware.</span><span class="sxs-lookup"><span data-stu-id="dd2c8-131">Specify `new GrpcWebOptions { DefaultEnabled = true }` when the middleware is added.</span></span>

[!code-csharp[](~/grpc/browser/sample/AllServicesSupportExample_Startup.cs?name=snippet_1&highlight=12)]

> [!NOTE]
> <span data-ttu-id="dd2c8-132">Esiste un problema noto che causa l'esito negativo di gRPC-Web quando è [ospitato da HTTP.sys](xref:fundamentals/servers/httpsys) in .NET Core 3. x.</span><span class="sxs-lookup"><span data-stu-id="dd2c8-132">There is a known issue that causes gRPC-Web to fail when [hosted by HTTP.sys](xref:fundamentals/servers/httpsys) in .NET Core 3.x.</span></span>
>
> <span data-ttu-id="dd2c8-133">Una soluzione alternativa per l'uso di gRPC-Web in HTTP.sys è disponibile [qui](https://github.com/grpc/grpc-dotnet/issues/853#issuecomment-610078202).</span><span class="sxs-lookup"><span data-stu-id="dd2c8-133">A workaround to get gRPC-Web working on HTTP.sys is available [here](https://github.com/grpc/grpc-dotnet/issues/853#issuecomment-610078202).</span></span>

### <a name="grpc-web-and-cors"></a><span data-ttu-id="dd2c8-134">gRPC-Web e CORS</span><span class="sxs-lookup"><span data-stu-id="dd2c8-134">gRPC-Web and CORS</span></span>

<span data-ttu-id="dd2c8-135">La sicurezza del browser impedisce a una pagina Web di eseguire richieste a un dominio diverso da quello che ha gestito la pagina Web.</span><span class="sxs-lookup"><span data-stu-id="dd2c8-135">Browser security prevents a web page from making requests to a different domain than the one that served the web page.</span></span> <span data-ttu-id="dd2c8-136">Questa restrizione si applica all'esecuzione di chiamate gRPC-Web con le app browser.</span><span class="sxs-lookup"><span data-stu-id="dd2c8-136">This restriction applies to making gRPC-Web calls with browser apps.</span></span> <span data-ttu-id="dd2c8-137">Ad esempio, un'app browser servita da `https://www.contoso.com` viene bloccata dalla chiamata a gRPC-Web Services ospitata in `https://services.contoso.com` .</span><span class="sxs-lookup"><span data-stu-id="dd2c8-137">For example, a browser app served by `https://www.contoso.com` is blocked from calling gRPC-Web services hosted on `https://services.contoso.com`.</span></span> <span data-ttu-id="dd2c8-138">La condivisione di risorse tra le origini (CORS) può essere usata per attenuare questa restrizione.</span><span class="sxs-lookup"><span data-stu-id="dd2c8-138">Cross Origin Resource Sharing (CORS) can be used to relax this restriction.</span></span>

<span data-ttu-id="dd2c8-139">Per consentire a un'app del browser di effettuare chiamate web gRPC tra le origini, configurare [CORS in ASP.NET Core](xref:security/cors).</span><span class="sxs-lookup"><span data-stu-id="dd2c8-139">To allow a browser app to make cross-origin gRPC-Web calls, set up [CORS in ASP.NET Core](xref:security/cors).</span></span> <span data-ttu-id="dd2c8-140">Usare il supporto CORS incorporato ed esporre intestazioni specifiche di gRPC con <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.WithExposedHeaders%2A> .</span><span class="sxs-lookup"><span data-stu-id="dd2c8-140">Use the built-in CORS support, and expose gRPC-specific headers with <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.WithExposedHeaders%2A>.</span></span>

[!code-csharp[](~/grpc/browser/sample/CORS_Startup.cs?name=snippet_1&highlight=5-11,19,24)]

<span data-ttu-id="dd2c8-141">Il codice precedente:</span><span class="sxs-lookup"><span data-stu-id="dd2c8-141">The preceding code:</span></span>

* <span data-ttu-id="dd2c8-142">Chiama `AddCors` per aggiungere servizi CORS e configura un criterio CORS che espone intestazioni specifiche di gRPC.</span><span class="sxs-lookup"><span data-stu-id="dd2c8-142">Calls `AddCors` to add CORS services and configures a CORS policy that exposes gRPC-specific headers.</span></span>
* <span data-ttu-id="dd2c8-143">Chiama `UseCors` per aggiungere il middleware CORS dopo il routing e prima degli endpoint.</span><span class="sxs-lookup"><span data-stu-id="dd2c8-143">Calls `UseCors` to add the CORS middleware after routing and before endpoints.</span></span>
* <span data-ttu-id="dd2c8-144">Specifica che il `endpoints.MapGrpcService<GreeterService>()` metodo supporta cors con `RequiresCors` .</span><span class="sxs-lookup"><span data-stu-id="dd2c8-144">Specifies the `endpoints.MapGrpcService<GreeterService>()` method supports CORS with `RequiresCors`.</span></span>

### <a name="grpc-web-and-streaming"></a><span data-ttu-id="dd2c8-145">gRPC-Web e streaming</span><span class="sxs-lookup"><span data-stu-id="dd2c8-145">gRPC-Web and streaming</span></span>

<span data-ttu-id="dd2c8-146">Il gRPC tradizionale su HTTP/2 supporta lo streaming in tutte le direzioni.</span><span class="sxs-lookup"><span data-stu-id="dd2c8-146">Traditional gRPC over HTTP/2 supports streaming in all directions.</span></span> <span data-ttu-id="dd2c8-147">gRPC-Web offre un supporto limitato per lo streaming:</span><span class="sxs-lookup"><span data-stu-id="dd2c8-147">gRPC-Web offers limited support for streaming:</span></span>

* <span data-ttu-id="dd2c8-148">i client gRPC-Web browser non supportano la chiamata di flussi client e metodi di streaming bidirezionale.</span><span class="sxs-lookup"><span data-stu-id="dd2c8-148">gRPC-Web browser clients don't support calling client streaming and bidirectional streaming methods.</span></span>
* <span data-ttu-id="dd2c8-149">ASP.NET Core Servizi gRPC ospitati nel servizio app Azure e IIS non supportano lo streaming bidirezionale.</span><span class="sxs-lookup"><span data-stu-id="dd2c8-149">ASP.NET Core gRPC services hosted on Azure App Service and IIS don't support bidirectional streaming.</span></span>

<span data-ttu-id="dd2c8-150">Quando si usa gRPC-Web, è consigliabile usare solo metodi unari e metodi di streaming del server.</span><span class="sxs-lookup"><span data-stu-id="dd2c8-150">When using gRPC-Web, we only recommend the use of unary methods and server streaming methods.</span></span>

## <a name="call-grpc-web-from-the-browser"></a><span data-ttu-id="dd2c8-151">Chiamare gRPC-Web dal browser</span><span class="sxs-lookup"><span data-stu-id="dd2c8-151">Call gRPC-Web from the browser</span></span>

<span data-ttu-id="dd2c8-152">Le app browser possono usare gRPC-Web per chiamare i servizi gRPC.</span><span class="sxs-lookup"><span data-stu-id="dd2c8-152">Browser apps can use gRPC-Web to call gRPC services.</span></span> <span data-ttu-id="dd2c8-153">Esistono alcuni requisiti e limitazioni quando si chiamano i servizi gRPC con gRPC-Web dal browser:</span><span class="sxs-lookup"><span data-stu-id="dd2c8-153">There are some requirements and limitations when calling gRPC services with gRPC-Web from the browser:</span></span>

* <span data-ttu-id="dd2c8-154">Il server deve essere stato configurato per supportare gRPC-Web.</span><span class="sxs-lookup"><span data-stu-id="dd2c8-154">The server must have been configured to support gRPC-Web.</span></span>
* <span data-ttu-id="dd2c8-155">Il flusso client e le chiamate di streaming bidirezionali non sono supportate.</span><span class="sxs-lookup"><span data-stu-id="dd2c8-155">Client streaming and bidirectional streaming calls aren't supported.</span></span> <span data-ttu-id="dd2c8-156">Il flusso del server è supportato.</span><span class="sxs-lookup"><span data-stu-id="dd2c8-156">Server streaming is supported.</span></span>
* <span data-ttu-id="dd2c8-157">Per chiamare i servizi gRPC in un dominio diverso è necessario configurare [CORS](xref:security/cors) nel server.</span><span class="sxs-lookup"><span data-stu-id="dd2c8-157">Calling gRPC services on a different domain requires [CORS](xref:security/cors) to be configured on the server.</span></span>

### <a name="javascript-grpc-web-client"></a><span data-ttu-id="dd2c8-158">JavaScript gRPC-client Web</span><span class="sxs-lookup"><span data-stu-id="dd2c8-158">JavaScript gRPC-Web client</span></span>

<span data-ttu-id="dd2c8-159">È disponibile un client JavaScript gRPC-Web.</span><span class="sxs-lookup"><span data-stu-id="dd2c8-159">There is a JavaScript gRPC-Web client.</span></span> <span data-ttu-id="dd2c8-160">Per istruzioni su come usare gRPC-Web da JavaScript, vedere [scrivere codice client JavaScript con gRPC-Web](https://github.com/grpc/grpc-web/tree/master/net/grpc/gateway/examples/helloworld#write-client-code).</span><span class="sxs-lookup"><span data-stu-id="dd2c8-160">For instructions on how to use gRPC-Web from JavaScript, see [write JavaScript client code with gRPC-Web](https://github.com/grpc/grpc-web/tree/master/net/grpc/gateway/examples/helloworld#write-client-code).</span></span>

### <a name="configure-grpc-web-with-the-net-grpc-client"></a><span data-ttu-id="dd2c8-161">Configurare gRPC-Web con il client gRPC .NET</span><span class="sxs-lookup"><span data-stu-id="dd2c8-161">Configure gRPC-Web with the .NET gRPC client</span></span>

<span data-ttu-id="dd2c8-162">Il client gRPC .NET può essere configurato per effettuare chiamate gRPC-Web.</span><span class="sxs-lookup"><span data-stu-id="dd2c8-162">The .NET gRPC client can be configured to make gRPC-Web calls.</span></span> <span data-ttu-id="dd2c8-163">Questa operazione è utile per [Blazor WebAssembly](xref:blazor/index#blazor-webassembly) le app, che sono ospitate nel browser e hanno le stesse limitazioni http del codice JavaScript.</span><span class="sxs-lookup"><span data-stu-id="dd2c8-163">This is useful for [Blazor WebAssembly](xref:blazor/index#blazor-webassembly) apps, which are hosted in the browser and have the same HTTP limitations of JavaScript code.</span></span> <span data-ttu-id="dd2c8-164">La chiamata di gRPC-Web con un client .NET equivale a [http/2 gRPC](xref:grpc/client).</span><span class="sxs-lookup"><span data-stu-id="dd2c8-164">Calling gRPC-Web with a .NET client is [the same as HTTP/2 gRPC](xref:grpc/client).</span></span> <span data-ttu-id="dd2c8-165">L'unica modifica è la modalità di creazione del canale.</span><span class="sxs-lookup"><span data-stu-id="dd2c8-165">The only modification is how the channel is created.</span></span>

<span data-ttu-id="dd2c8-166">Per usare gRPC-Web:</span><span class="sxs-lookup"><span data-stu-id="dd2c8-166">To use gRPC-Web:</span></span>

* <span data-ttu-id="dd2c8-167">Aggiungere un riferimento al pacchetto [Grpc .NET. client. Web](https://www.nuget.org/packages/Grpc.Net.Client.Web) .</span><span class="sxs-lookup"><span data-stu-id="dd2c8-167">Add a reference to the [Grpc.Net.Client.Web](https://www.nuget.org/packages/Grpc.Net.Client.Web) package.</span></span>
* <span data-ttu-id="dd2c8-168">Verificare che il riferimento a [Grpc .NET. client](https://www.nuget.org/packages/Grpc.Net.Client) Package sia 2.29.0 o versione successiva.</span><span class="sxs-lookup"><span data-stu-id="dd2c8-168">Ensure the reference to [Grpc.Net.Client](https://www.nuget.org/packages/Grpc.Net.Client) package is 2.29.0 or greater.</span></span>
* <span data-ttu-id="dd2c8-169">Configurare il canale per l'utilizzo di `GrpcWebHandler` :</span><span class="sxs-lookup"><span data-stu-id="dd2c8-169">Configure the channel to use the `GrpcWebHandler`:</span></span>

[!code-csharp[](~/grpc/browser/sample/Handler.cs?name=snippet_1)]

<span data-ttu-id="dd2c8-170">Il codice precedente:</span><span class="sxs-lookup"><span data-stu-id="dd2c8-170">The preceding code:</span></span>

* <span data-ttu-id="dd2c8-171">Configura un canale per l'uso di gRPC-Web.</span><span class="sxs-lookup"><span data-stu-id="dd2c8-171">Configures a channel to use gRPC-Web.</span></span>
* <span data-ttu-id="dd2c8-172">Crea un client ed effettua una chiamata utilizzando il canale.</span><span class="sxs-lookup"><span data-stu-id="dd2c8-172">Creates a client and makes a call using the channel.</span></span>

<span data-ttu-id="dd2c8-173">`GrpcWebHandler` in sono disponibili le opzioni di configurazione seguenti:</span><span class="sxs-lookup"><span data-stu-id="dd2c8-173">`GrpcWebHandler` has the following configuration options:</span></span>

* <span data-ttu-id="dd2c8-174">**InnerHandler**: oggetto sottostante <xref:System.Net.Http.HttpMessageHandler> che esegue la richiesta http gRPC, ad esempio `HttpClientHandler` .</span><span class="sxs-lookup"><span data-stu-id="dd2c8-174">**InnerHandler**: The underlying <xref:System.Net.Http.HttpMessageHandler> that makes the gRPC HTTP request, for example, `HttpClientHandler`.</span></span>
* <span data-ttu-id="dd2c8-175">**GrpcWebMode**: tipo di enumerazione che specifica se la richiesta http gRPC `Content-Type` è `application/grpc-web` o `application/grpc-web-text` .</span><span class="sxs-lookup"><span data-stu-id="dd2c8-175">**GrpcWebMode**: An enumeration type that specifies whether the gRPC HTTP request `Content-Type` is `application/grpc-web` or `application/grpc-web-text`.</span></span>
    * <span data-ttu-id="dd2c8-176">`GrpcWebMode.GrpcWeb` Configura il contenuto da inviare senza codifica.</span><span class="sxs-lookup"><span data-stu-id="dd2c8-176">`GrpcWebMode.GrpcWeb` configures content to be sent without encoding.</span></span> <span data-ttu-id="dd2c8-177">Valore predefinito.</span><span class="sxs-lookup"><span data-stu-id="dd2c8-177">Default value.</span></span>
    * <span data-ttu-id="dd2c8-178">`GrpcWebMode.GrpcWebText` Configura il contenuto con codifica Base64.</span><span class="sxs-lookup"><span data-stu-id="dd2c8-178">`GrpcWebMode.GrpcWebText` configures content to be base64 encoded.</span></span> <span data-ttu-id="dd2c8-179">Obbligatorio per le chiamate di streaming del server nei browser.</span><span class="sxs-lookup"><span data-stu-id="dd2c8-179">Required for server streaming calls in browsers.</span></span>
* <span data-ttu-id="dd2c8-180">**HttpVersion**: protocollo http `Version` usato per impostare [HttpRequestMessage. Version](xref:System.Net.Http.HttpRequestMessage.Version) sulla richiesta http gRPC sottostante.</span><span class="sxs-lookup"><span data-stu-id="dd2c8-180">**HttpVersion**: HTTP protocol `Version` used to set [HttpRequestMessage.Version](xref:System.Net.Http.HttpRequestMessage.Version) on the underlying gRPC HTTP request.</span></span> <span data-ttu-id="dd2c8-181">gRPC-Web non richiede una versione specifica e non esegue l'override dell'impostazione predefinita, a meno che non sia specificato.</span><span class="sxs-lookup"><span data-stu-id="dd2c8-181">gRPC-Web doesn't require a specific version and doesn't override the default unless specified.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="dd2c8-182">I client gRPC generati hanno metodi Sync e Async per la chiamata di metodi unaria.</span><span class="sxs-lookup"><span data-stu-id="dd2c8-182">Generated gRPC clients have sync and async methods for calling unary methods.</span></span> <span data-ttu-id="dd2c8-183">Ad esempio, `SayHello` è Sync ed `SayHelloAsync` è Async.</span><span class="sxs-lookup"><span data-stu-id="dd2c8-183">For example, `SayHello` is sync and `SayHelloAsync` is async.</span></span> <span data-ttu-id="dd2c8-184">La chiamata di un metodo di sincronizzazione in un'app provocherà la mancata Blazor WebAssembly risposta dell'app.</span><span class="sxs-lookup"><span data-stu-id="dd2c8-184">Calling a sync method in a Blazor WebAssembly app will cause the app to become unresponsive.</span></span> <span data-ttu-id="dd2c8-185">I metodi asincroni devono sempre essere utilizzati in Blazor WebAssembly .</span><span class="sxs-lookup"><span data-stu-id="dd2c8-185">Async methods must always be used in Blazor WebAssembly.</span></span>

### <a name="use-grpc-client-factory-with-grpc-web"></a><span data-ttu-id="dd2c8-186">Usare la factory del client gRPC con gRPC-Web</span><span class="sxs-lookup"><span data-stu-id="dd2c8-186">Use gRPC client factory with gRPC-Web</span></span>

<span data-ttu-id="dd2c8-187">Un client .NET compatibile con gRPC può essere creato usando l'integrazione di gRPC con [HttpClientFactory](xref:System.Net.Http.IHttpClientFactory).</span><span class="sxs-lookup"><span data-stu-id="dd2c8-187">A gRPC-Web compatible .NET client can be created using gRPC's integration with [HttpClientFactory](xref:System.Net.Http.IHttpClientFactory).</span></span>

<span data-ttu-id="dd2c8-188">Per usare gRPC-Web con client Factory:</span><span class="sxs-lookup"><span data-stu-id="dd2c8-188">To use gRPC-Web with client factory:</span></span>

* <span data-ttu-id="dd2c8-189">Aggiungere i riferimenti ai pacchetti al file di progetto per i pacchetti seguenti:</span><span class="sxs-lookup"><span data-stu-id="dd2c8-189">Add package references to the project file for the following packages:</span></span>
  * [<span data-ttu-id="dd2c8-190">Grpc .NET. client. Web</span><span class="sxs-lookup"><span data-stu-id="dd2c8-190">Grpc.Net.Client.Web</span></span>](https://www.nuget.org/packages/Grpc.Net.Client.Web)
  * [<span data-ttu-id="dd2c8-191">Grpc .NET. ClientFactory</span><span class="sxs-lookup"><span data-stu-id="dd2c8-191">Grpc.Net.ClientFactory</span></span>](https://www.nuget.org/packages/Grpc.Net.ClientFactory)
* <span data-ttu-id="dd2c8-192">Registrare un client gRPC con l'inserimento DI dipendenze usando il `AddGrpcClient` metodo di estensione generico.</span><span class="sxs-lookup"><span data-stu-id="dd2c8-192">Register a gRPC client with dependency injection (DI) using the generic `AddGrpcClient` extension method.</span></span> <span data-ttu-id="dd2c8-193">In un' Blazor WebAssembly app, i servizi vengono registrati con l'inserimento di `Program.cs` .</span><span class="sxs-lookup"><span data-stu-id="dd2c8-193">In a Blazor WebAssembly app, services are registered with DI in `Program.cs`.</span></span>
* <span data-ttu-id="dd2c8-194">Configurare `GrpcWebHandler` usando il <xref:Microsoft.Extensions.DependencyInjection.HttpClientBuilderExtensions.ConfigurePrimaryHttpMessageHandler%2A> metodo di estensione.</span><span class="sxs-lookup"><span data-stu-id="dd2c8-194">Configure `GrpcWebHandler` using the <xref:Microsoft.Extensions.DependencyInjection.HttpClientBuilderExtensions.ConfigurePrimaryHttpMessageHandler%2A> extension method.</span></span>

```csharp
builder.Services
    .AddGrpcClient<Greet.GreeterClient>((services, options) =>
    {
        options.Address = new Uri("https://localhost:5001");
    })
    .ConfigurePrimaryHttpMessageHandler(
        () => new GrpcWebHandler(GrpcWebMode.GrpcWebText, new HttpClientHandler()));
```

<span data-ttu-id="dd2c8-195">Per altre informazioni, vedere <xref:grpc/clientfactory>.</span><span class="sxs-lookup"><span data-stu-id="dd2c8-195">For more information, see <xref:grpc/clientfactory>.</span></span>

## <a name="additional-resources"></a><span data-ttu-id="dd2c8-196">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="dd2c8-196">Additional resources</span></span>

* [<span data-ttu-id="dd2c8-197">progetto GitHub di gRPC per client Web</span><span class="sxs-lookup"><span data-stu-id="dd2c8-197">gRPC for Web Clients GitHub project</span></span>](https://github.com/grpc/grpc-web)
* <xref:security/cors>
* <xref:grpc/httpapi>
