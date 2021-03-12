---
title: Implementazione del server Web Kestrel in ASP.NET Core
author: rick-anderson
description: Informazioni su Kestrel, il server Web multipiattaforma per ASP.NET Core.
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 05/04/2020
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
uid: fundamentals/servers/kestrel
ms.openlocfilehash: 54d7304430c2a77347b6e3279f638cf6d7add77e
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102589043"
---
# <a name="kestrel-web-server-implementation-in-aspnet-core"></a><span data-ttu-id="13792-103">Implementazione del server Web Kestrel in ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="13792-103">Kestrel web server implementation in ASP.NET Core</span></span>

<span data-ttu-id="13792-104">Di [Tom Dykstra](https://github.com/tdykstra), [Chris Ross](https://github.com/Tratcher) e [Stephen Halter](https://twitter.com/halter73)</span><span class="sxs-lookup"><span data-stu-id="13792-104">By [Tom Dykstra](https://github.com/tdykstra), [Chris Ross](https://github.com/Tratcher), and [Stephen Halter](https://twitter.com/halter73)</span></span>

::: moniker range=">= aspnetcore-5.0"

<span data-ttu-id="13792-105">Kestrel è un [server Web per ASP.NET Core](xref:fundamentals/servers/index) multipiattaforma.</span><span class="sxs-lookup"><span data-stu-id="13792-105">Kestrel is a cross-platform [web server for ASP.NET Core](xref:fundamentals/servers/index).</span></span> <span data-ttu-id="13792-106">Gheppio è il server Web incluso e abilitato per impostazione predefinita nei modelli di progetto ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="13792-106">Kestrel is the web server that's included and enabled by default in ASP.NET Core project templates.</span></span>

<span data-ttu-id="13792-107">Kestrel supporta gli scenari seguenti:</span><span class="sxs-lookup"><span data-stu-id="13792-107">Kestrel supports the following scenarios:</span></span>

* <span data-ttu-id="13792-108">HTTPS</span><span class="sxs-lookup"><span data-stu-id="13792-108">HTTPS</span></span>
* <span data-ttu-id="13792-109">[Http/2](xref:fundamentals/servers/kestrel/http2) (eccetto MacOS &dagger; )</span><span class="sxs-lookup"><span data-stu-id="13792-109">[HTTP/2](xref:fundamentals/servers/kestrel/http2) (except on macOS&dagger;)</span></span>
* <span data-ttu-id="13792-110">Aggiornamento opaco usato per abilitare [WebSocket](xref:fundamentals/websockets)</span><span class="sxs-lookup"><span data-stu-id="13792-110">Opaque upgrade used to enable [WebSockets](xref:fundamentals/websockets)</span></span>
* <span data-ttu-id="13792-111">Socket Unix ad alte prestazioni dietro Nginx</span><span class="sxs-lookup"><span data-stu-id="13792-111">Unix sockets for high performance behind Nginx</span></span>

<span data-ttu-id="13792-112">&dagger;HTTP/2 verrà supportato in macOS in una versione futura.</span><span class="sxs-lookup"><span data-stu-id="13792-112">&dagger;HTTP/2 will be supported on macOS in a future release.</span></span>

<span data-ttu-id="13792-113">Kestrel è supportato in tutte le piattaforme e le versioni supportate da .NET Core.</span><span class="sxs-lookup"><span data-stu-id="13792-113">Kestrel is supported on all platforms and versions that .NET Core supports.</span></span>

<span data-ttu-id="13792-114">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/servers/kestrel/samples/5.x) ([procedura per il download](xref:index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="13792-114">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/servers/kestrel/samples/5.x) ([how to download](xref:index#how-to-download-a-sample))</span></span>

## <a name="get-started"></a><span data-ttu-id="13792-115">Introduzione</span><span class="sxs-lookup"><span data-stu-id="13792-115">Get started</span></span>

<span data-ttu-id="13792-116">I modelli di progetto ASP.NET Core usano Kestrel per impostazione predefinita.</span><span class="sxs-lookup"><span data-stu-id="13792-116">ASP.NET Core project templates use Kestrel by default.</span></span> <span data-ttu-id="13792-117">In *Program.cs* il <xref:Microsoft.Extensions.Hosting.GenericHostBuilderExtensions.ConfigureWebHostDefaults*> metodo chiama <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderKestrelExtensions.UseKestrel*> :</span><span class="sxs-lookup"><span data-stu-id="13792-117">In *Program.cs*, the <xref:Microsoft.Extensions.Hosting.GenericHostBuilderExtensions.ConfigureWebHostDefaults*> method calls <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderKestrelExtensions.UseKestrel*>:</span></span>

[!code-csharp[](kestrel/samples/5.x/KestrelSample/Program.cs?name=snippet_DefaultBuilder&highlight=8)]

<span data-ttu-id="13792-118">Per ulteriori informazioni sulla compilazione dell'host, vedere le sezioni configurare le impostazioni di *un host* e di un *generatore predefinito* di <xref:fundamentals/host/generic-host#set-up-a-host> .</span><span class="sxs-lookup"><span data-stu-id="13792-118">For more information on building the host, see the *Set up a host* and *Default builder settings* sections of <xref:fundamentals/host/generic-host#set-up-a-host>.</span></span>

## <a name="additional-resources"></a><span data-ttu-id="13792-119">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="13792-119">Additional resources</span></span>

<a name="endpoint-configuration"></a>
* <xref:fundamentals/servers/kestrel/endpoints>
<a name="kestrel-options"></a>
* <xref:fundamentals/servers/kestrel/options>
<a name="http2-support"></a>
* <xref:fundamentals/servers/kestrel/http2>
<a name="when-to-use-kestrel-with-a-reverse-proxy"></a>
* <xref:fundamentals/servers/kestrel/when-to-use-a-reverse-proxy>
<a name="host-filtering"></a>
* <xref:fundamentals/servers/kestrel/host-filtering>
* <xref:test/troubleshoot>
* <xref:security/enforcing-ssl>
* <xref:host-and-deploy/proxy-load-balancer>
* <span data-ttu-id="13792-120">[RFC 7230: Message Syntax and Routing (Section 5.4: Host)](https://tools.ietf.org/html/rfc7230#section-5.4) (RFC 7230: Sintassi e routing dei messaggi (sezione 5.4: Host))</span><span class="sxs-lookup"><span data-stu-id="13792-120">[RFC 7230: Message Syntax and Routing (Section 5.4: Host)](https://tools.ietf.org/html/rfc7230#section-5.4)</span></span>
* <span data-ttu-id="13792-121">Quando si usano i socket UNIX in Linux, il socket non viene eliminato automaticamente all'arresto dell'app.</span><span class="sxs-lookup"><span data-stu-id="13792-121">When using UNIX sockets on Linux, the socket is not automatically deleted on app shut down.</span></span> <span data-ttu-id="13792-122">Per altre informazioni, vedere [questo problema di GitHub](https://github.com/dotnet/aspnetcore/issues/14134).</span><span class="sxs-lookup"><span data-stu-id="13792-122">For more information, see [this GitHub issue](https://github.com/dotnet/aspnetcore/issues/14134).</span></span>

> [!NOTE]
> <span data-ttu-id="13792-123">A partire da ASP.NET Core 5,0, il trasporto libuv del gheppio è obsoleto.</span><span class="sxs-lookup"><span data-stu-id="13792-123">As of ASP.NET Core 5.0, Kestrel's libuv transport is obsolete.</span></span> <span data-ttu-id="13792-124">Il trasporto libuv non riceve gli aggiornamenti per supportare nuove piattaforme del sistema operativo, ad esempio Windows ARM64, e verrà rimosso in una versione futura.</span><span class="sxs-lookup"><span data-stu-id="13792-124">The libuv transport doesn't receive updates to support new OS platforms, such as Windows ARM64, and will be removed in a future release.</span></span> <span data-ttu-id="13792-125">Rimuovere tutte le chiamate al metodo obsoleto <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderLibuvExtensions.UseLibuv%2A> e usare invece il trasporto socket predefinito di gheppio.</span><span class="sxs-lookup"><span data-stu-id="13792-125">Remove any calls to the obsolete <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderLibuvExtensions.UseLibuv%2A> method and use Kestrel's default Socket transport instead.</span></span>

::: moniker-end

::: moniker range=">= aspnetcore-3.0 < aspnetcore-5.0"

<span data-ttu-id="13792-126">Kestrel è un [server Web per ASP.NET Core](xref:fundamentals/servers/index) multipiattaforma.</span><span class="sxs-lookup"><span data-stu-id="13792-126">Kestrel is a cross-platform [web server for ASP.NET Core](xref:fundamentals/servers/index).</span></span> <span data-ttu-id="13792-127">Kestrel è il server Web incluso per impostazione predefinita nei modelli di progetto di ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="13792-127">Kestrel is the web server that's included by default in ASP.NET Core project templates.</span></span>

<span data-ttu-id="13792-128">Kestrel supporta gli scenari seguenti:</span><span class="sxs-lookup"><span data-stu-id="13792-128">Kestrel supports the following scenarios:</span></span>

* <span data-ttu-id="13792-129">HTTPS</span><span class="sxs-lookup"><span data-stu-id="13792-129">HTTPS</span></span>
* <span data-ttu-id="13792-130">Aggiornamento opaco usato per abilitare [WebSocket](https://github.com/aspnet/websockets)</span><span class="sxs-lookup"><span data-stu-id="13792-130">Opaque upgrade used to enable [WebSockets](https://github.com/aspnet/websockets)</span></span>
* <span data-ttu-id="13792-131">Socket Unix ad alte prestazioni dietro Nginx</span><span class="sxs-lookup"><span data-stu-id="13792-131">Unix sockets for high performance behind Nginx</span></span>
* <span data-ttu-id="13792-132">HTTP/2 (tranne che in macOS&dagger;)</span><span class="sxs-lookup"><span data-stu-id="13792-132">HTTP/2 (except on macOS&dagger;)</span></span>

<span data-ttu-id="13792-133">&dagger;HTTP/2 verrà supportato in macOS in una versione futura.</span><span class="sxs-lookup"><span data-stu-id="13792-133">&dagger;HTTP/2 will be supported on macOS in a future release.</span></span>

<span data-ttu-id="13792-134">Kestrel è supportato in tutte le piattaforme e le versioni supportate da .NET Core.</span><span class="sxs-lookup"><span data-stu-id="13792-134">Kestrel is supported on all platforms and versions that .NET Core supports.</span></span>

<span data-ttu-id="13792-135">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/servers/kestrel/samples/3.x) ([procedura per il download](xref:index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="13792-135">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/servers/kestrel/samples/3.x) ([how to download](xref:index#how-to-download-a-sample))</span></span>

## <a name="http2-support"></a><span data-ttu-id="13792-136">Supporto HTTP/2</span><span class="sxs-lookup"><span data-stu-id="13792-136">HTTP/2 support</span></span>

<span data-ttu-id="13792-137">[HTTP/2](https://httpwg.org/specs/rfc7540.html) è disponibile per le app ASP.NET Core se vengono soddisfatti i requisiti di base seguenti:</span><span class="sxs-lookup"><span data-stu-id="13792-137">[HTTP/2](https://httpwg.org/specs/rfc7540.html) is available for ASP.NET Core apps if the following base requirements are met:</span></span>

* <span data-ttu-id="13792-138">Sistema operativo&dagger;</span><span class="sxs-lookup"><span data-stu-id="13792-138">Operating system&dagger;</span></span>
  * <span data-ttu-id="13792-139">Windows Server 2016/Windows 10 o versioni successive&Dagger;</span><span class="sxs-lookup"><span data-stu-id="13792-139">Windows Server 2016/Windows 10 or later&Dagger;</span></span>
  * <span data-ttu-id="13792-140">Linux con OpenSSL 1.0.2 o versioni successive (ad esempio, Ubuntu 16.04 o versioni successive)</span><span class="sxs-lookup"><span data-stu-id="13792-140">Linux with OpenSSL 1.0.2 or later (for example, Ubuntu 16.04 or later)</span></span>
* <span data-ttu-id="13792-141">Framework di destinazione: .NET Core 2.2 o versioni successive</span><span class="sxs-lookup"><span data-stu-id="13792-141">Target framework: .NET Core 2.2 or later</span></span>
* <span data-ttu-id="13792-142">Connessione [ALPN (Application-Layer Protocol Negotiation)](https://tools.ietf.org/html/rfc7301#section-3)</span><span class="sxs-lookup"><span data-stu-id="13792-142">[Application-Layer Protocol Negotiation (ALPN)](https://tools.ietf.org/html/rfc7301#section-3) connection</span></span>
* <span data-ttu-id="13792-143">Connessione TLS 1.2 o successiva</span><span class="sxs-lookup"><span data-stu-id="13792-143">TLS 1.2 or later connection</span></span>

<span data-ttu-id="13792-144">&dagger;HTTP/2 verrà supportato in macOS in una versione futura.</span><span class="sxs-lookup"><span data-stu-id="13792-144">&dagger;HTTP/2 will be supported on macOS in a future release.</span></span>
<span data-ttu-id="13792-145">&Dagger;Kestrel ha un supporto limitato per HTTP/2 in Windows Server 2012 R2 e Windows 8.1.</span><span class="sxs-lookup"><span data-stu-id="13792-145">&Dagger;Kestrel has limited support for HTTP/2 on Windows Server 2012 R2 and Windows 8.1.</span></span> <span data-ttu-id="13792-146">Il supporto è limitato perché l'elenco di suite di crittografia TLS supportate disponibili in questi sistemi operativi è limitato.</span><span class="sxs-lookup"><span data-stu-id="13792-146">Support is limited because the list of supported TLS cipher suites available on these operating systems is limited.</span></span> <span data-ttu-id="13792-147">Un certificato generato con un algoritmo ECDSA potrebbe essere necessario per proteggere le connessioni TLS.</span><span class="sxs-lookup"><span data-stu-id="13792-147">A certificate generated using an Elliptic Curve Digital Signature Algorithm (ECDSA) may be required to secure TLS connections.</span></span>

<span data-ttu-id="13792-148">Se viene stabilita una connessione HTTP/2, [HttpRequest.Protocol](xref:Microsoft.AspNetCore.Http.HttpRequest.Protocol*) corrisponde a `HTTP/2`.</span><span class="sxs-lookup"><span data-stu-id="13792-148">If an HTTP/2 connection is established, [HttpRequest.Protocol](xref:Microsoft.AspNetCore.Http.HttpRequest.Protocol*) reports `HTTP/2`.</span></span>

<span data-ttu-id="13792-149">A partire da .NET Core 3,0, HTTP/2 è abilitato per impostazione predefinita.</span><span class="sxs-lookup"><span data-stu-id="13792-149">Starting with .NET Core 3.0, HTTP/2 is enabled by default.</span></span> <span data-ttu-id="13792-150">Per altre informazioni sulla configurazione, vedere le sezioni [Opzioni Kestrel](#kestrel-options) e [ListenOptions.Protocols](#listenoptionsprotocols).</span><span class="sxs-lookup"><span data-stu-id="13792-150">For more information on configuration, see the [Kestrel options](#kestrel-options) and [ListenOptions.Protocols](#listenoptionsprotocols) sections.</span></span>

## <a name="when-to-use-kestrel-with-a-reverse-proxy"></a><span data-ttu-id="13792-151">Quando usare Kestrel con un proxy inverso</span><span class="sxs-lookup"><span data-stu-id="13792-151">When to use Kestrel with a reverse proxy</span></span>

<span data-ttu-id="13792-152">È possibile usare Kestrel da solo o in combinazione con un *server proxy inverso*, ad esempio [Internet Information Services (IIS)](https://www.iis.net/), [Nginx](https://nginx.org) o [Apache](https://httpd.apache.org/).</span><span class="sxs-lookup"><span data-stu-id="13792-152">Kestrel can be used by itself or with a *reverse proxy server*, such as [Internet Information Services (IIS)](https://www.iis.net/), [Nginx](https://nginx.org), or [Apache](https://httpd.apache.org/).</span></span> <span data-ttu-id="13792-153">Il server proxy inverso riceve le richieste HTTP dalla rete e le inoltra a Kestrel.</span><span class="sxs-lookup"><span data-stu-id="13792-153">A reverse proxy server receives HTTP requests from the network and forwards them to Kestrel.</span></span>

<span data-ttu-id="13792-154">Kestrel usato come server Web perimetrale (esposto a Internet):</span><span class="sxs-lookup"><span data-stu-id="13792-154">Kestrel used as an edge (Internet-facing) web server:</span></span>

![Kestrel comunica direttamente con Internet senza un server proxy inverso](kestrel/_static/kestrel-to-internet2.png)

<span data-ttu-id="13792-156">Kestrel usato in una configurazione proxy inverso:</span><span class="sxs-lookup"><span data-stu-id="13792-156">Kestrel used in a reverse proxy configuration:</span></span>

![Kestrel comunica indirettamente con Internet attraverso un server proxy inverso, ad esempio IIS, Nginx o Apache](kestrel/_static/kestrel-to-internet.png)

<span data-ttu-id="13792-158">La configurazione, con o senza un server proxy inverso, è una configurazione di hosting supportata.</span><span class="sxs-lookup"><span data-stu-id="13792-158">Either configuration, with or without a reverse proxy server, is a supported hosting configuration.</span></span>

<span data-ttu-id="13792-159">Se viene utilizzato come server perimetrale in assenza di un server proxy inverso, Kestrel non supporta la condivisione tra più processi dell'IP e della porta.</span><span class="sxs-lookup"><span data-stu-id="13792-159">Kestrel used as an edge server without a reverse proxy server doesn't support sharing the same IP and port among multiple processes.</span></span> <span data-ttu-id="13792-160">Quando è configurato per restare in ascolto su una porta, Kestrel gestisce tutto il traffico per tale porta indipendentemente dalle intestazioni `Host` delle richieste.</span><span class="sxs-lookup"><span data-stu-id="13792-160">When Kestrel is configured to listen on a port, Kestrel handles all of the traffic for that port regardless of requests' `Host` headers.</span></span> <span data-ttu-id="13792-161">Un proxy inverso che può condividere le porte può eseguire l'inoltro delle richieste a Kestrel su un unico IP e un'unica porta.</span><span class="sxs-lookup"><span data-stu-id="13792-161">A reverse proxy that can share ports has the ability to forward requests to Kestrel on a unique IP and port.</span></span>

<span data-ttu-id="13792-162">Anche se la presenza di un server proxy inverso non è necessaria, può risultare utile per i motivi seguenti.</span><span class="sxs-lookup"><span data-stu-id="13792-162">Even if a reverse proxy server isn't required, using a reverse proxy server might be a good choice.</span></span>

<span data-ttu-id="13792-163">Un proxy inverso:</span><span class="sxs-lookup"><span data-stu-id="13792-163">A reverse proxy:</span></span>

* <span data-ttu-id="13792-164">Può limitare l'area della superficie di attacco pubblica esposta delle app ospitate.</span><span class="sxs-lookup"><span data-stu-id="13792-164">Can limit the exposed public surface area of the apps that it hosts.</span></span>
* <span data-ttu-id="13792-165">Offre un livello aggiuntivo di configurazione e protezione.</span><span class="sxs-lookup"><span data-stu-id="13792-165">Provide an additional layer of configuration and defense.</span></span>
* <span data-ttu-id="13792-166">Potrebbe offrire un'integrazione migliore con l'infrastruttura esistente.</span><span class="sxs-lookup"><span data-stu-id="13792-166">Might integrate better with existing infrastructure.</span></span>
* <span data-ttu-id="13792-167">Semplifica il bilanciamento del carico e la configurazione di comunicazioni protette (HTTPS).</span><span class="sxs-lookup"><span data-stu-id="13792-167">Simplify load balancing and secure communication (HTTPS) configuration.</span></span> <span data-ttu-id="13792-168">Solo il server proxy inverso richiede un certificato X. 509 e il server è in grado di comunicare con i server dell'app nella rete interna tramite HTTP normale.</span><span class="sxs-lookup"><span data-stu-id="13792-168">Only the reverse proxy server requires an X.509 certificate, and that server can communicate with the app's servers on the internal network using plain HTTP.</span></span>

> [!WARNING]
> <span data-ttu-id="13792-169">Una configurazione che prevede un proxy inverso richiede il [filtro host](#host-filtering).</span><span class="sxs-lookup"><span data-stu-id="13792-169">Hosting in a reverse proxy configuration requires [host filtering](#host-filtering).</span></span>

## <a name="kestrel-in-aspnet-core-apps"></a><span data-ttu-id="13792-170">Gheppio nelle app ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="13792-170">Kestrel in ASP.NET Core apps</span></span>

<span data-ttu-id="13792-171">I modelli di progetto ASP.NET Core usano Kestrel per impostazione predefinita.</span><span class="sxs-lookup"><span data-stu-id="13792-171">ASP.NET Core project templates use Kestrel by default.</span></span> <span data-ttu-id="13792-172">In *Program.cs* il <xref:Microsoft.Extensions.Hosting.GenericHostBuilderExtensions.ConfigureWebHostDefaults*> metodo chiama <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderKestrelExtensions.UseKestrel*> :</span><span class="sxs-lookup"><span data-stu-id="13792-172">In *Program.cs*, the <xref:Microsoft.Extensions.Hosting.GenericHostBuilderExtensions.ConfigureWebHostDefaults*> method calls <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderKestrelExtensions.UseKestrel*>:</span></span>

[!code-csharp[](kestrel/samples/3.x/KestrelSample/Program.cs?name=snippet_DefaultBuilder&highlight=8)]

<span data-ttu-id="13792-173">Per ulteriori informazioni sulla compilazione dell'host, vedere le sezioni configurare le impostazioni di *un host* e di un *generatore predefinito* di <xref:fundamentals/host/generic-host#set-up-a-host> .</span><span class="sxs-lookup"><span data-stu-id="13792-173">For more information on building the host, see the *Set up a host* and *Default builder settings* sections of <xref:fundamentals/host/generic-host#set-up-a-host>.</span></span>

<span data-ttu-id="13792-174">Per fornire una configurazione aggiuntiva dopo la chiamata di `ConfigureWebHostDefaults`, usare `ConfigureKestrel`:</span><span class="sxs-lookup"><span data-stu-id="13792-174">To provide additional configuration after calling `ConfigureWebHostDefaults`, use `ConfigureKestrel`:</span></span>

```csharp
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.ConfigureKestrel(serverOptions =>
            {
                // Set properties and call methods on options
            })
            .UseStartup<Startup>();
        });
```

## <a name="kestrel-options"></a><span data-ttu-id="13792-175">Opzioni Kestrel</span><span class="sxs-lookup"><span data-stu-id="13792-175">Kestrel options</span></span>

<span data-ttu-id="13792-176">Il server Web Kestrel dispone di opzioni di configurazione dei vincoli che risultano particolarmente utili nelle distribuzioni con connessione Internet.</span><span class="sxs-lookup"><span data-stu-id="13792-176">The Kestrel web server has constraint configuration options that are especially useful in Internet-facing deployments.</span></span>

<span data-ttu-id="13792-177">Impostare i vincoli per la proprietà <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Limits> della classe <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions>.</span><span class="sxs-lookup"><span data-stu-id="13792-177">Set constraints on the <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Limits> property of the <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions> class.</span></span> <span data-ttu-id="13792-178">La proprietà `Limits` contiene un'istanza della classe <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits>.</span><span class="sxs-lookup"><span data-stu-id="13792-178">The `Limits` property holds an instance of the <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits> class.</span></span>

<span data-ttu-id="13792-179">Negli esempi seguenti viene usato lo spazio dei nomi <xref:Microsoft.AspNetCore.Server.Kestrel.Core>:</span><span class="sxs-lookup"><span data-stu-id="13792-179">The following examples use the <xref:Microsoft.AspNetCore.Server.Kestrel.Core> namespace:</span></span>

```csharp
using Microsoft.AspNetCore.Server.Kestrel.Core;
```

<span data-ttu-id="13792-180">Negli esempi illustrati più avanti in questo articolo, le opzioni di gheppio sono configurate nel codice C#.</span><span class="sxs-lookup"><span data-stu-id="13792-180">In examples shown later in this article, Kestrel options are configured in C# code.</span></span> <span data-ttu-id="13792-181">È inoltre possibile impostare le opzioni di Gheppio utilizzando un [provider di configurazione](xref:fundamentals/configuration/index).</span><span class="sxs-lookup"><span data-stu-id="13792-181">Kestrel options can also be set using a [configuration provider](xref:fundamentals/configuration/index).</span></span> <span data-ttu-id="13792-182">Il [provider di configurazione file](xref:fundamentals/configuration/index#file-configuration-provider) , ad esempio, può caricare la configurazione di Gheppio da un oggetto *appsettings.json* o *appSettings. { File Environment}. JSON* :</span><span class="sxs-lookup"><span data-stu-id="13792-182">For example, the [File Configuration Provider](xref:fundamentals/configuration/index#file-configuration-provider) can load Kestrel configuration from an *appsettings.json* or *appsettings.{Environment}.json* file:</span></span>

```json
{
  "Kestrel": {
    "Limits": {
      "MaxConcurrentConnections": 100,
      "MaxConcurrentUpgradedConnections": 100
    },
    "DisableStringReuse": true
  }
}
```

> [!NOTE]
> <span data-ttu-id="13792-183"><xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions> e la [configurazione dell'endpoint](#endpoint-configuration) sono configurabili dai provider di configurazione.</span><span class="sxs-lookup"><span data-stu-id="13792-183"><xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions> and [endpoint configuration](#endpoint-configuration) are configurable from configuration providers.</span></span> <span data-ttu-id="13792-184">La configurazione del gheppio rimanente deve essere configurata nel codice C#.</span><span class="sxs-lookup"><span data-stu-id="13792-184">Remaining Kestrel configuration must be configured in C# code.</span></span>

<span data-ttu-id="13792-185">Usare **uno** degli approcci seguenti:</span><span class="sxs-lookup"><span data-stu-id="13792-185">Use **one** of the following approaches:</span></span>

* <span data-ttu-id="13792-186">Configurare gheppio in `Startup.ConfigureServices` :</span><span class="sxs-lookup"><span data-stu-id="13792-186">Configure Kestrel in `Startup.ConfigureServices`:</span></span>

  1. <span data-ttu-id="13792-187">Inserire un'istanza di `IConfiguration` nella `Startup` classe.</span><span class="sxs-lookup"><span data-stu-id="13792-187">Inject an instance of `IConfiguration` into the `Startup` class.</span></span> <span data-ttu-id="13792-188">Nell'esempio seguente si presuppone che la configurazione inserita venga assegnata alla `Configuration` Proprietà.</span><span class="sxs-lookup"><span data-stu-id="13792-188">The following example assumes that the injected configuration is assigned to the `Configuration` property.</span></span>
  2. <span data-ttu-id="13792-189">In `Startup.ConfigureServices` caricare la `Kestrel` sezione della configurazione nella configurazione di Gheppio:</span><span class="sxs-lookup"><span data-stu-id="13792-189">In `Startup.ConfigureServices`, load the `Kestrel` section of configuration into Kestrel's configuration:</span></span>

     ```csharp
     using Microsoft.Extensions.Configuration
     
     public class Startup
     {
         public Startup(IConfiguration configuration)
         {
             Configuration = configuration;
         }

         public IConfiguration Configuration { get; }

         public void ConfigureServices(IServiceCollection services)
         {
             services.Configure<KestrelServerOptions>(
                 Configuration.GetSection("Kestrel"));
         }

         public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
         {
             ...
         }
     }
     ```

* <span data-ttu-id="13792-190">Configurare il gheppio durante la compilazione dell'host:</span><span class="sxs-lookup"><span data-stu-id="13792-190">Configure Kestrel when building the host:</span></span>

  <span data-ttu-id="13792-191">In *Program.cs* caricare la `Kestrel` sezione della configurazione nella configurazione di Gheppio:</span><span class="sxs-lookup"><span data-stu-id="13792-191">In *Program.cs*, load the `Kestrel` section of configuration into Kestrel's configuration:</span></span>

  ```csharp
  // using Microsoft.Extensions.DependencyInjection;

  public static IHostBuilder CreateHostBuilder(string[] args) =>
      Host.CreateDefaultBuilder(args)
          .ConfigureServices((context, services) =>
          {
              services.Configure<KestrelServerOptions>(
                  context.Configuration.GetSection("Kestrel"));
          })
          .ConfigureWebHostDefaults(webBuilder =>
          {
              webBuilder.UseStartup<Startup>();
          });
  ```

<span data-ttu-id="13792-192">Entrambi gli approcci precedenti funzionano con qualsiasi [provider di configurazione](xref:fundamentals/configuration/index).</span><span class="sxs-lookup"><span data-stu-id="13792-192">Both of the preceding approaches work with any [configuration provider](xref:fundamentals/configuration/index).</span></span>

### <a name="keep-alive-timeout"></a><span data-ttu-id="13792-193">Timeout keep-alive</span><span class="sxs-lookup"><span data-stu-id="13792-193">Keep-alive timeout</span></span>

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.KeepAliveTimeout>

<span data-ttu-id="13792-194">Ottiene o imposta il [timeout keep-alive](https://tools.ietf.org/html/rfc7230#section-6.5).</span><span class="sxs-lookup"><span data-stu-id="13792-194">Gets or sets the [keep-alive timeout](https://tools.ietf.org/html/rfc7230#section-6.5).</span></span> <span data-ttu-id="13792-195">Il valore predefinito è 2 minuti.</span><span class="sxs-lookup"><span data-stu-id="13792-195">Defaults to 2 minutes.</span></span>

[!code-csharp[](kestrel/samples/3.x/KestrelSample/Program.cs?name=snippet_Limits&highlight=19-20)]

### <a name="maximum-client-connections"></a><span data-ttu-id="13792-196">Numero massimo di connessioni client</span><span class="sxs-lookup"><span data-stu-id="13792-196">Maximum client connections</span></span>

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.MaxConcurrentConnections>
<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.MaxConcurrentUpgradedConnections>

<span data-ttu-id="13792-197">È possibile impostare il numero massimo di connessioni TCP aperte simultaneamente per l'intera app con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="13792-197">The maximum number of concurrent open TCP connections can be set for the entire app with the following code:</span></span>

[!code-csharp[](kestrel/samples/3.x/KestrelSample/Program.cs?name=snippet_Limits&highlight=3)]

<span data-ttu-id="13792-198">È previsto un limite separato per le connessioni che sono state aggiornate da HTTP o HTTPS a un altro protocollo (ad esempio su una richiesta WebSocket).</span><span class="sxs-lookup"><span data-stu-id="13792-198">There's a separate limit for connections that have been upgraded from HTTP or HTTPS to another protocol (for example, on a WebSockets request).</span></span> <span data-ttu-id="13792-199">Dopo l'aggiornamento di una connessione, questa non viene conteggiata per il limite `MaxConcurrentConnections`.</span><span class="sxs-lookup"><span data-stu-id="13792-199">After a connection is upgraded, it isn't counted against the `MaxConcurrentConnections` limit.</span></span>

[!code-csharp[](kestrel/samples/3.x/KestrelSample/Program.cs?name=snippet_Limits&highlight=4)]

<span data-ttu-id="13792-200">Per impostazione predefinita, il numero massimo di connessioni è illimitato (null).</span><span class="sxs-lookup"><span data-stu-id="13792-200">The maximum number of connections is unlimited (null) by default.</span></span>

### <a name="maximum-request-body-size"></a><span data-ttu-id="13792-201">Dimensione massima del corpo della richiesta</span><span class="sxs-lookup"><span data-stu-id="13792-201">Maximum request body size</span></span>

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.MaxRequestBodySize>

<span data-ttu-id="13792-202">La dimensione massima predefinita del corpo della richiesta è pari a 30.000.000 di byte, equivalenti a circa 28,6 MB.</span><span class="sxs-lookup"><span data-stu-id="13792-202">The default maximum request body size is 30,000,000 bytes, which is approximately 28.6 MB.</span></span>

<span data-ttu-id="13792-203">Il metodo consigliato per ignorare il limite in un'app ASP.NET Core MVC è l'uso dell'attributo <xref:Microsoft.AspNetCore.Mvc.RequestSizeLimitAttribute> in un metodo di azione:</span><span class="sxs-lookup"><span data-stu-id="13792-203">The recommended approach to override the limit in an ASP.NET Core MVC app is to use the <xref:Microsoft.AspNetCore.Mvc.RequestSizeLimitAttribute> attribute on an action method:</span></span>

```csharp
[RequestSizeLimit(100000000)]
public IActionResult MyActionMethod()
```

<span data-ttu-id="13792-204">L'esempio seguente illustra come configurare il vincolo per l'app in ogni richiesta:</span><span class="sxs-lookup"><span data-stu-id="13792-204">Here's an example that shows how to configure the constraint for the app on every request:</span></span>

[!code-csharp[](kestrel/samples/3.x/KestrelSample/Program.cs?name=snippet_Limits&highlight=5)]

<span data-ttu-id="13792-205">Eseguire l'override dell'impostazione per una richiesta specifica nel middleware:</span><span class="sxs-lookup"><span data-stu-id="13792-205">Override the setting on a specific request in middleware:</span></span>

[!code-csharp[](kestrel/samples/3.x/KestrelSample/Startup.cs?name=snippet_Limits&highlight=3-4)]

<span data-ttu-id="13792-206">Viene generata un'eccezione se l'app configura il limite per una richiesta dopo che l'app ha iniziato a leggere la richiesta.</span><span class="sxs-lookup"><span data-stu-id="13792-206">An exception is thrown if the app configures the limit on a request after the app has started to read the request.</span></span> <span data-ttu-id="13792-207">Una proprietà `IsReadOnly` indica se la proprietà `MaxRequestBodySize` è in stato di sola lettura e pertanto è troppo tardi per configurare il limite.</span><span class="sxs-lookup"><span data-stu-id="13792-207">There's an `IsReadOnly` property that indicates if the `MaxRequestBodySize` property is in read-only state, meaning it's too late to configure the limit.</span></span>

<span data-ttu-id="13792-208">Quando un'app viene eseguita [out-of-process](xref:host-and-deploy/iis/index#out-of-process-hosting-model) dietro il [modulo ASP.NET Core](xref:host-and-deploy/aspnet-core-module), il limite della dimensione del corpo della richiesta di Kestrel è disabilitato perché viene già impostato da IIS.</span><span class="sxs-lookup"><span data-stu-id="13792-208">When an app is run [out-of-process](xref:host-and-deploy/iis/index#out-of-process-hosting-model) behind the [ASP.NET Core Module](xref:host-and-deploy/aspnet-core-module), Kestrel's request body size limit is disabled because IIS already sets the limit.</span></span>

### <a name="minimum-request-body-data-rate"></a><span data-ttu-id="13792-209">Velocità minima dei dati del corpo della richiesta</span><span class="sxs-lookup"><span data-stu-id="13792-209">Minimum request body data rate</span></span>

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.MinRequestBodyDataRate>
<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.MinResponseDataRate>

<span data-ttu-id="13792-210">Kestrel controlla ogni secondo se i dati arrivano alla velocità in byte al secondo specificata.</span><span class="sxs-lookup"><span data-stu-id="13792-210">Kestrel checks every second if data is arriving at the specified rate in bytes/second.</span></span> <span data-ttu-id="13792-211">Se la frequenza scende sotto il valore minimo, si è verificato il timeout della connessione. Il periodo di tolleranza è la quantità di tempo che il gheppio concede al client di aumentare la velocità di invio fino al minimo. la frequenza non viene controllata durante tale periodo di tempo.</span><span class="sxs-lookup"><span data-stu-id="13792-211">If the rate drops below the minimum, the connection is timed out. The grace period is the amount of time that Kestrel gives the client to increase its send rate up to the minimum; the rate isn't checked during that time.</span></span> <span data-ttu-id="13792-212">Il periodo di prova consente di evitare l'interruzione di connessioni che inizialmente inviano i dati a velocità ridotta a causa dell'avvio lento del protocollo TCP.</span><span class="sxs-lookup"><span data-stu-id="13792-212">The grace period helps avoid dropping connections that are initially sending data at a slow rate due to TCP slow-start.</span></span>

<span data-ttu-id="13792-213">La velocità minima predefinita è di 240 byte al secondo, con un periodo di tolleranza di 5 secondi.</span><span class="sxs-lookup"><span data-stu-id="13792-213">The default minimum rate is 240 bytes/second with a 5 second grace period.</span></span>

<span data-ttu-id="13792-214">Anche per la risposta è prevista una velocità minima.</span><span class="sxs-lookup"><span data-stu-id="13792-214">A minimum rate also applies to the response.</span></span> <span data-ttu-id="13792-215">Il codice per impostare il limite della richiesta e della risposta è identico e varia solo per `RequestBody` o `Response` nei nomi di proprietà e di interfaccia.</span><span class="sxs-lookup"><span data-stu-id="13792-215">The code to set the request limit and the response limit is the same except for having `RequestBody` or `Response` in the property and interface names.</span></span>

<span data-ttu-id="13792-216">L'esempio seguente visualizza come configurare la velocità minima dei dati in *Program.cs*:</span><span class="sxs-lookup"><span data-stu-id="13792-216">Here's an example that shows how to configure the minimum data rates in *Program.cs*:</span></span>

[!code-csharp[](kestrel/samples/3.x/KestrelSample/Program.cs?name=snippet_Limits&highlight=6-11)]

<span data-ttu-id="13792-217">Ignorare i limiti di velocità minima per ogni richiesta nel middleware:</span><span class="sxs-lookup"><span data-stu-id="13792-217">Override the minimum rate limits per request in middleware:</span></span>

[!code-csharp[](kestrel/samples/3.x/KestrelSample/Startup.cs?name=snippet_Limits&highlight=6-21)]

<span data-ttu-id="13792-218">La funzionalità <xref:Microsoft.AspNetCore.Server.Kestrel.Core.Features.IHttpMinResponseDataRateFeature> a cui si fa riferimento nell'esempio precedente non è presente in `HttpContext.Features` per le richieste HTTP/2, perché la modifica dei limiti di velocità per ogni richiesta in genere non è supportata per HTTP/2 a causa del supporto del protocollo per il multiplexing delle richieste.</span><span class="sxs-lookup"><span data-stu-id="13792-218">The <xref:Microsoft.AspNetCore.Server.Kestrel.Core.Features.IHttpMinResponseDataRateFeature> referenced in the prior sample is not present in `HttpContext.Features` for HTTP/2 requests because modifying rate limits on a per-request basis is generally not supported for HTTP/2 due to the protocol's support for request multiplexing.</span></span> <span data-ttu-id="13792-219">La funzionalità <xref:Microsoft.AspNetCore.Server.Kestrel.Core.Features.IHttpMinRequestBodyDataRateFeature> è tuttavia ancora presente in `HttpContext.Features` per le richieste HTTP/2, perché il limite di velocità di lettura può comunque essere *disabilitato interamente* per ogni singola richiesta impostando `IHttpMinRequestBodyDataRateFeature.MinDataRate` su `null` anche per una richiesta HTTP/2.</span><span class="sxs-lookup"><span data-stu-id="13792-219">However, the <xref:Microsoft.AspNetCore.Server.Kestrel.Core.Features.IHttpMinRequestBodyDataRateFeature> is still present `HttpContext.Features` for HTTP/2 requests, because the read rate limit can still be *disabled entirely* on a per-request basis by setting `IHttpMinRequestBodyDataRateFeature.MinDataRate` to `null` even for an HTTP/2 request.</span></span> <span data-ttu-id="13792-220">Il tentativo di leggere `IHttpMinRequestBodyDataRateFeature.MinDataRate` o il tentativo di impostazione di un valore diverso da `null` causerà la generazione di una `NotSupportedException` per una richiesta HTTP/2.</span><span class="sxs-lookup"><span data-stu-id="13792-220">Attempting to read `IHttpMinRequestBodyDataRateFeature.MinDataRate` or attempting to set it to a value other than `null` will result in a `NotSupportedException` being thrown given an HTTP/2 request.</span></span>

<span data-ttu-id="13792-221">I limiti di velocità a livello di server configurati tramite `KestrelServerOptions.Limits` vengono tuttavia applicati alle connessioni HTTP/1.x e HTTP/2.</span><span class="sxs-lookup"><span data-stu-id="13792-221">Server-wide rate limits configured via `KestrelServerOptions.Limits` still apply to both HTTP/1.x and HTTP/2 connections.</span></span>

### <a name="request-headers-timeout"></a><span data-ttu-id="13792-222">Timeout delle intestazioni delle richieste</span><span class="sxs-lookup"><span data-stu-id="13792-222">Request headers timeout</span></span>

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.RequestHeadersTimeout>

<span data-ttu-id="13792-223">Ottiene o imposta la quantità massima di tempo che il server dedica alla ricezione delle intestazioni delle richieste.</span><span class="sxs-lookup"><span data-stu-id="13792-223">Gets or sets the maximum amount of time the server spends receiving request headers.</span></span> <span data-ttu-id="13792-224">Il valore predefinito è 30 secondi.</span><span class="sxs-lookup"><span data-stu-id="13792-224">Defaults to 30 seconds.</span></span>

[!code-csharp[](kestrel/samples/3.x/KestrelSample/Program.cs?name=snippet_Limits&highlight=21-22)]

### <a name="maximum-streams-per-connection"></a><span data-ttu-id="13792-225">Numero massimo di flussi per connessione</span><span class="sxs-lookup"><span data-stu-id="13792-225">Maximum streams per connection</span></span>

<span data-ttu-id="13792-226">`Http2.MaxStreamsPerConnection` limita il numero di flussi di richieste simultanee per ogni connessione HTTP/2.</span><span class="sxs-lookup"><span data-stu-id="13792-226">`Http2.MaxStreamsPerConnection` limits the number of concurrent request streams per HTTP/2 connection.</span></span> <span data-ttu-id="13792-227">I flussi in eccesso vengono rifiutati.</span><span class="sxs-lookup"><span data-stu-id="13792-227">Excess streams are refused.</span></span>

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.Limits.Http2.MaxStreamsPerConnection = 100;
});
```

<span data-ttu-id="13792-228">Il valore predefinito è 100.</span><span class="sxs-lookup"><span data-stu-id="13792-228">The default value is 100.</span></span>

### <a name="header-table-size"></a><span data-ttu-id="13792-229">Dimensioni della tabella delle intestazioni</span><span class="sxs-lookup"><span data-stu-id="13792-229">Header table size</span></span>

<span data-ttu-id="13792-230">Il decodificatore HPACK decomprime le intestazioni HTTP per le connessioni HTTP/2.</span><span class="sxs-lookup"><span data-stu-id="13792-230">The HPACK decoder decompresses HTTP headers for HTTP/2 connections.</span></span> <span data-ttu-id="13792-231">`Http2.HeaderTableSize` limita le dimensioni della tabella di compressione delle intestazioni usata dal decodificatore HPACK.</span><span class="sxs-lookup"><span data-stu-id="13792-231">`Http2.HeaderTableSize` limits the size of the header compression table that the HPACK decoder uses.</span></span> <span data-ttu-id="13792-232">Il valore viene specificato in ottetti e deve essere maggiore di zero (0).</span><span class="sxs-lookup"><span data-stu-id="13792-232">The value is provided in octets and must be greater than zero (0).</span></span>

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.Limits.Http2.HeaderTableSize = 4096;
});
```

<span data-ttu-id="13792-233">Il valore predefinito è 4096.</span><span class="sxs-lookup"><span data-stu-id="13792-233">The default value is 4096.</span></span>

### <a name="maximum-frame-size"></a><span data-ttu-id="13792-234">Dimensione massima del frame</span><span class="sxs-lookup"><span data-stu-id="13792-234">Maximum frame size</span></span>

<span data-ttu-id="13792-235">`Http2.MaxFrameSize` indica la dimensione massima consentita di un payload del frame di connessione HTTP/2 ricevuto o inviato dal server.</span><span class="sxs-lookup"><span data-stu-id="13792-235">`Http2.MaxFrameSize` indicates the maximum allowed size of an HTTP/2 connection frame payload received or sent by the server.</span></span> <span data-ttu-id="13792-236">Il valore viene specificato in ottetti e deve essere compreso tra 2^14 (16.384) e 2^24-1 (16.777.215).</span><span class="sxs-lookup"><span data-stu-id="13792-236">The value is provided in octets and must be between 2^14 (16,384) and 2^24-1 (16,777,215).</span></span>

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.Limits.Http2.MaxFrameSize = 16384;
});
```

<span data-ttu-id="13792-237">Il valore predefinito è 2^14 (16.384).</span><span class="sxs-lookup"><span data-stu-id="13792-237">The default value is 2^14 (16,384).</span></span>

### <a name="maximum-request-header-size"></a><span data-ttu-id="13792-238">Dimensioni massime dell'intestazione della richiesta</span><span class="sxs-lookup"><span data-stu-id="13792-238">Maximum request header size</span></span>

<span data-ttu-id="13792-239">`Http2.MaxRequestHeaderFieldSize` indica le dimensioni massime consentite in ottetti di valori di intestazione di richiesta.</span><span class="sxs-lookup"><span data-stu-id="13792-239">`Http2.MaxRequestHeaderFieldSize` indicates the maximum allowed size in octets of request header values.</span></span> <span data-ttu-id="13792-240">Questo limite si applica sia al nome che al valore nelle rappresentazioni compresse e non compresse.</span><span class="sxs-lookup"><span data-stu-id="13792-240">This limit applies to both name and value in their compressed and uncompressed representations.</span></span> <span data-ttu-id="13792-241">Il valore deve essere maggiore di zero (0).</span><span class="sxs-lookup"><span data-stu-id="13792-241">The value must be greater than zero (0).</span></span>

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.Limits.Http2.MaxRequestHeaderFieldSize = 8192;
});
```

<span data-ttu-id="13792-242">Il valore predefinito è 8.192.</span><span class="sxs-lookup"><span data-stu-id="13792-242">The default value is 8,192.</span></span>

### <a name="initial-connection-window-size"></a><span data-ttu-id="13792-243">Dimensioni della finestra di connessione iniziali</span><span class="sxs-lookup"><span data-stu-id="13792-243">Initial connection window size</span></span>

<span data-ttu-id="13792-244">`Http2.InitialConnectionWindowSize` indica il numero massimo di byte di dati del corpo della richiesta che il server memorizza nel buffer in una volta, aggregati in tutte le richieste (flussi), per ogni connessione.</span><span class="sxs-lookup"><span data-stu-id="13792-244">`Http2.InitialConnectionWindowSize` indicates the maximum request body data in bytes the server buffers at one time aggregated across all requests (streams) per connection.</span></span> <span data-ttu-id="13792-245">Le richieste sono anche limitate da `Http2.InitialStreamWindowSize`.</span><span class="sxs-lookup"><span data-stu-id="13792-245">Requests are also limited by `Http2.InitialStreamWindowSize`.</span></span> <span data-ttu-id="13792-246">Il valore deve essere maggiore di o uguale a 65.535 e minore di 2^31 (2.147.483.648).</span><span class="sxs-lookup"><span data-stu-id="13792-246">The value must be greater than or equal to 65,535 and less than 2^31 (2,147,483,648).</span></span>

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.Limits.Http2.InitialConnectionWindowSize = 131072;
});
```

<span data-ttu-id="13792-247">Il valore predefinito è 128 KB (131.072).</span><span class="sxs-lookup"><span data-stu-id="13792-247">The default value is 128 KB (131,072).</span></span>

### <a name="initial-stream-window-size"></a><span data-ttu-id="13792-248">Dimensioni della finestra di flusso iniziali</span><span class="sxs-lookup"><span data-stu-id="13792-248">Initial stream window size</span></span>

<span data-ttu-id="13792-249">`Http2.InitialStreamWindowSize` indica il numero massimo di byte di dati del corpo della richiesta che il server memorizza nel buffer in una volta per ogni richiesta (flusso).</span><span class="sxs-lookup"><span data-stu-id="13792-249">`Http2.InitialStreamWindowSize` indicates the maximum request body data in bytes the server buffers at one time per request (stream).</span></span> <span data-ttu-id="13792-250">Le richieste sono anche limitate da `Http2.InitialConnectionWindowSize`.</span><span class="sxs-lookup"><span data-stu-id="13792-250">Requests are also limited by `Http2.InitialConnectionWindowSize`.</span></span> <span data-ttu-id="13792-251">Il valore deve essere maggiore di o uguale a 65.535 e minore di 2^31 (2.147.483.648).</span><span class="sxs-lookup"><span data-stu-id="13792-251">The value must be greater than or equal to 65,535 and less than 2^31 (2,147,483,648).</span></span>

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.Limits.Http2.InitialStreamWindowSize = 98304;
});
```

<span data-ttu-id="13792-252">Il valore predefinito è 96 KB (98.304).</span><span class="sxs-lookup"><span data-stu-id="13792-252">The default value is 96 KB (98,304).</span></span>

### <a name="trailers"></a><span data-ttu-id="13792-253">Trailer</span><span class="sxs-lookup"><span data-stu-id="13792-253">Trailers</span></span>

[!INCLUDE[](~/includes/trailers.md)]

### <a name="reset"></a><span data-ttu-id="13792-254">Reset</span><span class="sxs-lookup"><span data-stu-id="13792-254">Reset</span></span>

[!INCLUDE[](~/includes/reset.md)]

### <a name="synchronous-io"></a><span data-ttu-id="13792-255">I/O sincrono</span><span class="sxs-lookup"><span data-stu-id="13792-255">Synchronous I/O</span></span>

<span data-ttu-id="13792-256"><xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.AllowSynchronousIO> Controlla se è consentito l'I/O sincrono per la richiesta e la risposta.</span><span class="sxs-lookup"><span data-stu-id="13792-256"><xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.AllowSynchronousIO> controls whether synchronous I/O is allowed for the request and response.</span></span> <span data-ttu-id="13792-257">Il valore predefinito è `false`.</span><span class="sxs-lookup"><span data-stu-id="13792-257">The default value is `false`.</span></span>

> [!WARNING]
> <span data-ttu-id="13792-258">Un numero elevato di operazioni di I/O sincrone bloccanti può causare l'esaurimento del pool di thread, il che rende l'app non risponde.</span><span class="sxs-lookup"><span data-stu-id="13792-258">A large number of blocking synchronous I/O operations can lead to thread pool starvation, which makes the app unresponsive.</span></span> <span data-ttu-id="13792-259">Abilitare solo `AllowSynchronousIO` quando si usa una libreria che non supporta l'I/O asincrono.</span><span class="sxs-lookup"><span data-stu-id="13792-259">Only enable `AllowSynchronousIO` when using a library that doesn't support asynchronous I/O.</span></span>

<span data-ttu-id="13792-260">L'esempio seguente abilita l'I/O sincrono:</span><span class="sxs-lookup"><span data-stu-id="13792-260">The following example enables synchronous I/O:</span></span>

[!code-csharp[](kestrel/samples/3.x/KestrelSample/Program.cs?name=snippet_SyncIO)]

<span data-ttu-id="13792-261">Per informazioni su altre opzioni e limiti di Kestrel, vedere:</span><span class="sxs-lookup"><span data-stu-id="13792-261">For information about other Kestrel options and limits, see:</span></span>

* <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions>
* <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits>
* <xref:Microsoft.AspNetCore.Server.Kestrel.Core.ListenOptions>

## <a name="endpoint-configuration"></a><span data-ttu-id="13792-262">Configurazione dell'endpoint</span><span class="sxs-lookup"><span data-stu-id="13792-262">Endpoint configuration</span></span>

<span data-ttu-id="13792-263">Per impostazione predefinita, ASP.NET Core è associato a:</span><span class="sxs-lookup"><span data-stu-id="13792-263">By default, ASP.NET Core binds to:</span></span>

* `http://localhost:5000`
* <span data-ttu-id="13792-264">`https://localhost:5001` (quando è presente un certificato di sviluppo locale)</span><span class="sxs-lookup"><span data-stu-id="13792-264">`https://localhost:5001` (when a local development certificate is present)</span></span>

<span data-ttu-id="13792-265">Specificare gli URL usando gli elementi seguenti:</span><span class="sxs-lookup"><span data-stu-id="13792-265">Specify URLs using the:</span></span>

* <span data-ttu-id="13792-266">La variabile di ambiente `ASPNETCORE_URLS`.</span><span class="sxs-lookup"><span data-stu-id="13792-266">`ASPNETCORE_URLS` environment variable.</span></span>
* <span data-ttu-id="13792-267">L'argomento della riga di comando `--urls`.</span><span class="sxs-lookup"><span data-stu-id="13792-267">`--urls` command-line argument.</span></span>
* <span data-ttu-id="13792-268">La chiave di configurazione dell'host `urls`.</span><span class="sxs-lookup"><span data-stu-id="13792-268">`urls` host configuration key.</span></span>
* <span data-ttu-id="13792-269">Il metodo di estensione `UseUrls`.</span><span class="sxs-lookup"><span data-stu-id="13792-269">`UseUrls` extension method.</span></span>

<span data-ttu-id="13792-270">Il valore specificato usando i metodi seguenti può essere uno o più endpoint HTTP e HTTPS (HTTPS se è disponibile un certificato predefinito).</span><span class="sxs-lookup"><span data-stu-id="13792-270">The value provided using these approaches can be one or more HTTP and HTTPS endpoints (HTTPS if a default cert is available).</span></span> <span data-ttu-id="13792-271">Configurare il valore come un elenco delimitato da punto e virgola (ad esempio, `"Urls": "http://localhost:8000;http://localhost:8001"`).</span><span class="sxs-lookup"><span data-stu-id="13792-271">Configure the value as a semicolon-separated list (for example, `"Urls": "http://localhost:8000;http://localhost:8001"`).</span></span>

<span data-ttu-id="13792-272">Per altre informazioni su questi approcci, vedere [URL del server](xref:fundamentals/host/web-host#server-urls) e [Override della configurazione](xref:fundamentals/host/web-host#override-configuration).</span><span class="sxs-lookup"><span data-stu-id="13792-272">For more information on these approaches, see [Server URLs](xref:fundamentals/host/web-host#server-urls) and [Override configuration](xref:fundamentals/host/web-host#override-configuration).</span></span>

<span data-ttu-id="13792-273">Viene creato un certificato di sviluppo nei casi seguenti:</span><span class="sxs-lookup"><span data-stu-id="13792-273">A development certificate is created:</span></span>

* <span data-ttu-id="13792-274">Quando viene installato [.NET Core SDK](/dotnet/core/sdk).</span><span class="sxs-lookup"><span data-stu-id="13792-274">When the [.NET Core SDK](/dotnet/core/sdk) is installed.</span></span>
* <span data-ttu-id="13792-275">Quando per creare un certificato viene usato [lo strumento dev-certs](xref:aspnetcore-2.1#https).</span><span class="sxs-lookup"><span data-stu-id="13792-275">The [dev-certs tool](xref:aspnetcore-2.1#https) is used to create a certificate.</span></span>

<span data-ttu-id="13792-276">Alcuni browser richiedono la concessione di autorizzazioni esplicite per considerare attendibile il certificato di sviluppo locale.</span><span class="sxs-lookup"><span data-stu-id="13792-276">Some browsers require granting explicit permission to trust the local development certificate.</span></span>

<span data-ttu-id="13792-277">I modelli di progetto consentono di configurare le app per l'esecuzione su HTTPS per impostazione predefinita e includono il [reindirizzamento HTTPS e il supporto HSTS](xref:security/enforcing-ssl).</span><span class="sxs-lookup"><span data-stu-id="13792-277">Project templates configure apps to run on HTTPS by default and include [HTTPS redirection and HSTS support](xref:security/enforcing-ssl).</span></span>

<span data-ttu-id="13792-278">Chiamare i metodi <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Listen*> oppure <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ListenUnixSocket*> su <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions> per configurare le porte e i prefissi URL per Kestrel.</span><span class="sxs-lookup"><span data-stu-id="13792-278">Call <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Listen*> or <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ListenUnixSocket*> methods on <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions> to configure URL prefixes and ports for Kestrel.</span></span>

<span data-ttu-id="13792-279">Anche `UseUrls`, l'argomento della riga di comando `--urls`, la chiave di configurazione dell'host `urls` e la variabile di ambiente `ASPNETCORE_URLS` funzionano, ma con le limitazioni indicate più avanti nella sezione (deve essere disponibile un certificato predefinito per la configurazione dell'endopoint HTTPS).</span><span class="sxs-lookup"><span data-stu-id="13792-279">`UseUrls`, the `--urls` command-line argument, `urls` host configuration key, and the `ASPNETCORE_URLS` environment variable also work but have the limitations noted later in this section (a default certificate must be available for HTTPS endpoint configuration).</span></span>

<span data-ttu-id="13792-280">`KestrelServerOptions` configurazione</span><span class="sxs-lookup"><span data-stu-id="13792-280">`KestrelServerOptions` configuration:</span></span>

### <a name="configureendpointdefaultsactionlistenoptions"></a><span data-ttu-id="13792-281">ConfigureEndpointDefaults (azione \<ListenOptions> )</span><span class="sxs-lookup"><span data-stu-id="13792-281">ConfigureEndpointDefaults(Action\<ListenOptions>)</span></span>

<span data-ttu-id="13792-282">Specifica un elemento di configurazione `Action` da eseguire per ogni endpoint specificato.</span><span class="sxs-lookup"><span data-stu-id="13792-282">Specifies a configuration `Action` to run for each specified endpoint.</span></span> <span data-ttu-id="13792-283">Se si chiama `ConfigureEndpointDefaults` più volte, gli elementi `Action` precedenti vengono sostituiti con l'ultimo elemento `Action` specificato.</span><span class="sxs-lookup"><span data-stu-id="13792-283">Calling `ConfigureEndpointDefaults` multiple times replaces prior `Action`s with the last `Action` specified.</span></span>

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.ConfigureEndpointDefaults(listenOptions =>
    {
        // Configure endpoint defaults
    });
});
```

> [!NOTE]
> <span data-ttu-id="13792-284">Per gli endpoint creati chiamando <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Listen*> **prima** della chiamata a <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ConfigureEndpointDefaults*> non verranno applicati i valori predefiniti.</span><span class="sxs-lookup"><span data-stu-id="13792-284">Endpoints created by calling <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Listen*> **before** calling <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ConfigureEndpointDefaults*> won't have the defaults applied.</span></span>

### <a name="configurehttpsdefaultsactionhttpsconnectionadapteroptions"></a><span data-ttu-id="13792-285">ConfigureHttpsDefaults (azione \<HttpsConnectionAdapterOptions> )</span><span class="sxs-lookup"><span data-stu-id="13792-285">ConfigureHttpsDefaults(Action\<HttpsConnectionAdapterOptions>)</span></span>

<span data-ttu-id="13792-286">Specifica un elemento di configurazione `Action` da eseguire per ogni endpoint HTTPS.</span><span class="sxs-lookup"><span data-stu-id="13792-286">Specifies a configuration `Action` to run for each HTTPS endpoint.</span></span> <span data-ttu-id="13792-287">Se si chiama `ConfigureHttpsDefaults` più volte, gli elementi `Action` precedenti vengono sostituiti con l'ultimo elemento `Action` specificato.</span><span class="sxs-lookup"><span data-stu-id="13792-287">Calling `ConfigureHttpsDefaults` multiple times replaces prior `Action`s with the last `Action` specified.</span></span>

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.ConfigureHttpsDefaults(listenOptions =>
    {
        // certificate is an X509Certificate2
        listenOptions.ServerCertificate = certificate;
    });
});
```

> [!NOTE]
> <span data-ttu-id="13792-288">Per gli endpoint creati chiamando <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Listen*> **prima** della chiamata a <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ConfigureHttpsDefaults*> non verranno applicati i valori predefiniti.</span><span class="sxs-lookup"><span data-stu-id="13792-288">Endpoints created by calling <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Listen*> **before** calling <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ConfigureHttpsDefaults*> won't have the defaults applied.</span></span>

### <a name="configureiconfiguration"></a><span data-ttu-id="13792-289">Configure(IConfiguration)</span><span class="sxs-lookup"><span data-stu-id="13792-289">Configure(IConfiguration)</span></span>

<span data-ttu-id="13792-290">Crea un loader di configurazione per la configurazione di Kestrel che accetta un elemento <xref:Microsoft.Extensions.Configuration.IConfiguration> come input.</span><span class="sxs-lookup"><span data-stu-id="13792-290">Creates a configuration loader for setting up Kestrel that takes an <xref:Microsoft.Extensions.Configuration.IConfiguration> as input.</span></span> <span data-ttu-id="13792-291">L'ambito della configurazione deve essere la sezione di configurazione per Kestrel.</span><span class="sxs-lookup"><span data-stu-id="13792-291">The configuration must be scoped to the configuration section for Kestrel.</span></span>

### <a name="listenoptionsusehttps"></a><span data-ttu-id="13792-292">ListenOptions.UseHttps</span><span class="sxs-lookup"><span data-stu-id="13792-292">ListenOptions.UseHttps</span></span>

<span data-ttu-id="13792-293">Configura Kestrel per l'uso di HTTPS.</span><span class="sxs-lookup"><span data-stu-id="13792-293">Configure Kestrel to use HTTPS.</span></span>

<span data-ttu-id="13792-294">Estensioni `ListenOptions.UseHttps`:</span><span class="sxs-lookup"><span data-stu-id="13792-294">`ListenOptions.UseHttps` extensions:</span></span>

* <span data-ttu-id="13792-295">`UseHttps`: Configurare gheppio per l'uso di HTTPS con il certificato predefinito.</span><span class="sxs-lookup"><span data-stu-id="13792-295">`UseHttps`: Configure Kestrel to use HTTPS with the default certificate.</span></span> <span data-ttu-id="13792-296">Genera un'eccezione se non è stato configurato alcun certificato predefinito.</span><span class="sxs-lookup"><span data-stu-id="13792-296">Throws an exception if no default certificate is configured.</span></span>
* `UseHttps(string fileName)`
* `UseHttps(string fileName, string password)`
* `UseHttps(string fileName, string password, Action<HttpsConnectionAdapterOptions> configureOptions)`
* `UseHttps(StoreName storeName, string subject)`
* `UseHttps(StoreName storeName, string subject, bool allowInvalid)`
* `UseHttps(StoreName storeName, string subject, bool allowInvalid, StoreLocation location)`
* `UseHttps(StoreName storeName, string subject, bool allowInvalid, StoreLocation location, Action<HttpsConnectionAdapterOptions> configureOptions)`
* `UseHttps(X509Certificate2 serverCertificate)`
* `UseHttps(X509Certificate2 serverCertificate, Action<HttpsConnectionAdapterOptions> configureOptions)`
* `UseHttps(Action<HttpsConnectionAdapterOptions> configureOptions)`

<span data-ttu-id="13792-297">Parametri `ListenOptions.UseHttps`:</span><span class="sxs-lookup"><span data-stu-id="13792-297">`ListenOptions.UseHttps` parameters:</span></span>

* <span data-ttu-id="13792-298">`filename` è il percorso e il nome file di un file di certificato, relativo alla directory che contiene i file di contenuto dell'app.</span><span class="sxs-lookup"><span data-stu-id="13792-298">`filename` is the path and file name of a certificate file, relative to the directory that contains the app's content files.</span></span>
* <span data-ttu-id="13792-299">`password` è la password richiesta per accedere ai dati del certificato X.509.</span><span class="sxs-lookup"><span data-stu-id="13792-299">`password` is the password required to access the X.509 certificate data.</span></span>
* <span data-ttu-id="13792-300">`configureOptions` è un elemento `Action` per configurare `HttpsConnectionAdapterOptions`.</span><span class="sxs-lookup"><span data-stu-id="13792-300">`configureOptions` is an `Action` to configure the `HttpsConnectionAdapterOptions`.</span></span> <span data-ttu-id="13792-301">Restituisce `ListenOptions`.</span><span class="sxs-lookup"><span data-stu-id="13792-301">Returns the `ListenOptions`.</span></span>
* <span data-ttu-id="13792-302">`storeName` è l'archivio certificati da cui caricare il certificato.</span><span class="sxs-lookup"><span data-stu-id="13792-302">`storeName` is the certificate store from which to load the certificate.</span></span>
* <span data-ttu-id="13792-303">`subject` è il nome dell'oggetto del certificato.</span><span class="sxs-lookup"><span data-stu-id="13792-303">`subject` is the subject name for the certificate.</span></span>
* <span data-ttu-id="13792-304">`allowInvalid` indica se devono essere considerati i certificati non validi, come ad esempio i certificati autofirmati.</span><span class="sxs-lookup"><span data-stu-id="13792-304">`allowInvalid` indicates if invalid certificates should be considered, such as self-signed certificates.</span></span>
* <span data-ttu-id="13792-305">`location` è il percorso dell'archivio da cui caricare il certificato.</span><span class="sxs-lookup"><span data-stu-id="13792-305">`location` is the store location to load the certificate from.</span></span>
* <span data-ttu-id="13792-306">`serverCertificate` è il certificato X.509.</span><span class="sxs-lookup"><span data-stu-id="13792-306">`serverCertificate` is the X.509 certificate.</span></span>

<span data-ttu-id="13792-307">Nell'ambiente di produzione, HTTPS deve essere configurato in modo esplicito.</span><span class="sxs-lookup"><span data-stu-id="13792-307">In production, HTTPS must be explicitly configured.</span></span> <span data-ttu-id="13792-308">Come minimo, è necessario specificare un certificato predefinito.</span><span class="sxs-lookup"><span data-stu-id="13792-308">At a minimum, a default certificate must be provided.</span></span>

<span data-ttu-id="13792-309">Configurazioni supportate descritte in seguito:</span><span class="sxs-lookup"><span data-stu-id="13792-309">Supported configurations described next:</span></span>

* <span data-ttu-id="13792-310">Nessuna configurazione</span><span class="sxs-lookup"><span data-stu-id="13792-310">No configuration</span></span>
* <span data-ttu-id="13792-311">Sostituire il certificato predefinito della configurazione</span><span class="sxs-lookup"><span data-stu-id="13792-311">Replace the default certificate from configuration</span></span>
* <span data-ttu-id="13792-312">Modificare le impostazioni predefinite nel codice</span><span class="sxs-lookup"><span data-stu-id="13792-312">Change the defaults in code</span></span>

<span data-ttu-id="13792-313">*Nessuna configurazione*</span><span class="sxs-lookup"><span data-stu-id="13792-313">*No configuration*</span></span>

<span data-ttu-id="13792-314">Kestrel è in ascolto su `http://localhost:5000` e `https://localhost:5001` (se è disponibile un certificato predefinito).</span><span class="sxs-lookup"><span data-stu-id="13792-314">Kestrel listens on `http://localhost:5000` and `https://localhost:5001` (if a default cert is available).</span></span>

<a name="configuration"></a>

<span data-ttu-id="13792-315">*Sostituire il certificato predefinito della configurazione*</span><span class="sxs-lookup"><span data-stu-id="13792-315">*Replace the default certificate from configuration*</span></span>

<span data-ttu-id="13792-316">`CreateDefaultBuilder` chiama `Configure(context.Configuration.GetSection("Kestrel"))` per impostazione predefinita per caricare la configurazione di Kestrel.</span><span class="sxs-lookup"><span data-stu-id="13792-316">`CreateDefaultBuilder` calls `Configure(context.Configuration.GetSection("Kestrel"))` by default to load Kestrel configuration.</span></span> <span data-ttu-id="13792-317">È disponibile per Kestrel uno schema di configurazione delle impostazioni delle app HTTPS predefinito.</span><span class="sxs-lookup"><span data-stu-id="13792-317">A default HTTPS app settings configuration schema is available for Kestrel.</span></span> <span data-ttu-id="13792-318">Configurare più endpoint, inclusi gli URL e i certificati da usare, da un file su disco o da un archivio certificati.</span><span class="sxs-lookup"><span data-stu-id="13792-318">Configure multiple endpoints, including the URLs and the certificates to use, either from a file on disk or from a certificate store.</span></span>

<span data-ttu-id="13792-319">Nell'esempio seguente *appsettings.json* :</span><span class="sxs-lookup"><span data-stu-id="13792-319">In the following *appsettings.json* example:</span></span>

* <span data-ttu-id="13792-320">Impostare **AllowInvalid** su `true` per consentire l'uso di certificati non validi, come ad esempio i certificati autofirmati.</span><span class="sxs-lookup"><span data-stu-id="13792-320">Set **AllowInvalid** to `true` to permit the use of invalid certificates (for example, self-signed certificates).</span></span>
* <span data-ttu-id="13792-321">Tutti gli endpoint HTTPS che non specificano un certificato (**HttpsDefaultCert** nell'esempio che segue) usano il certificato definito in **Certificates** > **Default** o il certificato di sviluppo.</span><span class="sxs-lookup"><span data-stu-id="13792-321">Any HTTPS endpoint that doesn't specify a certificate (**HttpsDefaultCert** in the example that follows) falls back to the cert defined under **Certificates** > **Default** or the development certificate.</span></span>

```json
{
  "Kestrel": {
    "Endpoints": {
      "Http": {
        "Url": "http://localhost:5000"
      },
      "HttpsInlineCertFile": {
        "Url": "https://localhost:5001",
        "Certificate": {
          "Path": "<path to .pfx file>",
          "Password": "<certificate password>"
        }
      },
      "HttpsInlineCertStore": {
        "Url": "https://localhost:5002",
        "Certificate": {
          "Subject": "<subject; required>",
          "Store": "<certificate store; required>",
          "Location": "<location; defaults to CurrentUser>",
          "AllowInvalid": "<true or false; defaults to false>"
        }
      },
      "HttpsDefaultCert": {
        "Url": "https://localhost:5003"
      },
      "Https": {
        "Url": "https://*:5004",
        "Certificate": {
          "Path": "<path to .pfx file>",
          "Password": "<certificate password>"
        }
      }
    },
    "Certificates": {
      "Default": {
        "Path": "<path to .pfx file>",
        "Password": "<certificate password>"
      }
    }
  }
}
```

<span data-ttu-id="13792-322">Un'alternativa all'uso di **Path** e **Password** per qualsiasi nodo del certificato consiste nello specificare il certificato usando i campi dell'archivio certificati.</span><span class="sxs-lookup"><span data-stu-id="13792-322">An alternative to using **Path** and **Password** for any certificate node is to specify the certificate using certificate store fields.</span></span> <span data-ttu-id="13792-323">Ad esempio, il certificato predefinito **Certificates**  >   può essere specificato come:</span><span class="sxs-lookup"><span data-stu-id="13792-323">For example, the **Certificates** > **Default** certificate can be specified as:</span></span>

```json
"Default": {
  "Subject": "<subject; required>",
  "Store": "<cert store; required>",
  "Location": "<location; defaults to CurrentUser>",
  "AllowInvalid": "<true or false; defaults to false>"
}
```

<span data-ttu-id="13792-324">Note di schema:</span><span class="sxs-lookup"><span data-stu-id="13792-324">Schema notes:</span></span>

* <span data-ttu-id="13792-325">I nomi degli endpoint non applicano la distinzione tra maiuscole e minuscole.</span><span class="sxs-lookup"><span data-stu-id="13792-325">Endpoints names are case-insensitive.</span></span> <span data-ttu-id="13792-326">Ad esempio, sono validi sia `HTTPS` che `Https`.</span><span class="sxs-lookup"><span data-stu-id="13792-326">For example, `HTTPS` and `Https` are valid.</span></span>
* <span data-ttu-id="13792-327">Il parametro `Url` è obbligatorio per ogni endpoint.</span><span class="sxs-lookup"><span data-stu-id="13792-327">The `Url` parameter is required for each endpoint.</span></span> <span data-ttu-id="13792-328">Il formato per questo parametro è uguale a quello del parametro di configurazione di primo livello `Urls`, con la differenza che si limita a un singolo valore.</span><span class="sxs-lookup"><span data-stu-id="13792-328">The format for this parameter is the same as the top-level `Urls` configuration parameter except that it's limited to a single value.</span></span>
* <span data-ttu-id="13792-329">Questi endpoint sostituiscono quelli definiti nella configurazione di primo livello `Urls`, non vi si aggiungono.</span><span class="sxs-lookup"><span data-stu-id="13792-329">These endpoints replace those defined in the top-level `Urls` configuration rather than adding to them.</span></span> <span data-ttu-id="13792-330">Gli endpoint definiti nel codice tramite `Listen` si aggiungono agli endpoint definiti nella sezione di configurazione.</span><span class="sxs-lookup"><span data-stu-id="13792-330">Endpoints defined in code via `Listen` are cumulative with the endpoints defined in the configuration section.</span></span>
* <span data-ttu-id="13792-331">La sezione `Certificate` è facoltativa.</span><span class="sxs-lookup"><span data-stu-id="13792-331">The `Certificate` section is optional.</span></span> <span data-ttu-id="13792-332">Se la sezione `Certificate` non è specificata, vengono usati i valori predefiniti definiti negli scenari precedenti.</span><span class="sxs-lookup"><span data-stu-id="13792-332">If the `Certificate` section isn't specified, the defaults defined in earlier scenarios are used.</span></span> <span data-ttu-id="13792-333">Se non è disponibile alcun valore predefinito, il server genera un'eccezione e non viene avviato.</span><span class="sxs-lookup"><span data-stu-id="13792-333">If no defaults are available, the server throws an exception and fails to start.</span></span>
* <span data-ttu-id="13792-334">La `Certificate` sezione supporta sia la password del **percorso** &ndash;  che i certificati dell'archivio dell' **oggetto** &ndash;  .</span><span class="sxs-lookup"><span data-stu-id="13792-334">The `Certificate` section supports both **Path**&ndash;**Password** and **Subject**&ndash;**Store** certificates.</span></span>
* <span data-ttu-id="13792-335">In questo modo è possibile definire un numero qualsiasi di endpoint, purché non provochino conflitti di porte.</span><span class="sxs-lookup"><span data-stu-id="13792-335">Any number of endpoints may be defined in this way so long as they don't cause port conflicts.</span></span>
* <span data-ttu-id="13792-336">`options.Configure(context.Configuration.GetSection("{SECTION}"))` restituisce un oggetto `KestrelConfigurationLoader` con un metodo `.Endpoint(string name, listenOptions => { })` che può essere usato per integrare le impostazioni dell'endpoint configurato:</span><span class="sxs-lookup"><span data-stu-id="13792-336">`options.Configure(context.Configuration.GetSection("{SECTION}"))` returns a `KestrelConfigurationLoader` with an `.Endpoint(string name, listenOptions => { })` method that can be used to supplement a configured endpoint's settings:</span></span>

```csharp
webBuilder.UseKestrel((context, serverOptions) =>
{
    serverOptions.Configure(context.Configuration.GetSection("Kestrel"))
        .Endpoint("HTTPS", listenOptions =>
        {
            listenOptions.HttpsOptions.SslProtocols = SslProtocols.Tls12;
        });
});
```

<span data-ttu-id="13792-337">`KestrelServerOptions.ConfigurationLoader` è possibile accedere direttamente a per continuare a scorrere sul caricatore esistente, ad esempio quello fornito da <xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder*> .</span><span class="sxs-lookup"><span data-stu-id="13792-337">`KestrelServerOptions.ConfigurationLoader` can be directly accessed to continue iterating on the existing loader, such as the one provided by <xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder*>.</span></span>

* <span data-ttu-id="13792-338">La sezione di configurazione per ogni endpoint è disponibile nelle opzioni del `Endpoint` metodo in modo che sia possibile leggere le impostazioni personalizzate.</span><span class="sxs-lookup"><span data-stu-id="13792-338">The configuration section for each endpoint is available on the options in the `Endpoint` method so that custom settings may be read.</span></span>
* <span data-ttu-id="13792-339">È possibile caricare più configurazioni chiamando ancora `options.Configure(context.Configuration.GetSection("{SECTION}"))` con un'altra sezione.</span><span class="sxs-lookup"><span data-stu-id="13792-339">Multiple configurations may be loaded by calling `options.Configure(context.Configuration.GetSection("{SECTION}"))` again with another section.</span></span> <span data-ttu-id="13792-340">Viene usata solo l'ultima configurazione, a meno che `Load` sia stato chiamato esplicitamente in istanze precedenti.</span><span class="sxs-lookup"><span data-stu-id="13792-340">Only the last configuration is used, unless `Load` is explicitly called on prior instances.</span></span> <span data-ttu-id="13792-341">Il metapacchetto non chiama `Load` in modo che la sezione di configurazione predefinita venga sostituita.</span><span class="sxs-lookup"><span data-stu-id="13792-341">The metapackage doesn't call `Load` so that its default configuration section may be replaced.</span></span>
* <span data-ttu-id="13792-342">`KestrelConfigurationLoader` riflette la famiglia di API `Listen` da `KestrelServerOptions` all'overload di `Endpoint`, in modo che gli endpoint di codice e di configurazione possano essere configurati nella stessa posizione.</span><span class="sxs-lookup"><span data-stu-id="13792-342">`KestrelConfigurationLoader` mirrors the `Listen` family of APIs from `KestrelServerOptions` as `Endpoint` overloads, so code and config endpoints may be configured in the same place.</span></span> <span data-ttu-id="13792-343">Questi overload non usano nomi e usano solo le impostazioni predefinite della configurazione.</span><span class="sxs-lookup"><span data-stu-id="13792-343">These overloads don't use names and only consume default settings from configuration.</span></span>

<span data-ttu-id="13792-344">*Modificare le impostazioni predefinite nel codice*</span><span class="sxs-lookup"><span data-stu-id="13792-344">*Change the defaults in code*</span></span>

<span data-ttu-id="13792-345">`ConfigureEndpointDefaults` e `ConfigureHttpsDefaults` possono essere usati per modificare le impostazioni predefinite per `ListenOptions` e `HttpsConnectionAdapterOptions`, inclusa l'esecuzione dell'override del certificato predefinito specificato nello scenario precedente.</span><span class="sxs-lookup"><span data-stu-id="13792-345">`ConfigureEndpointDefaults` and `ConfigureHttpsDefaults` can be used to change default settings for `ListenOptions` and `HttpsConnectionAdapterOptions`, including overriding the default certificate specified in the prior scenario.</span></span> <span data-ttu-id="13792-346">`ConfigureEndpointDefaults` e `ConfigureHttpsDefaults` devono essere chiamati prima che venga configurato qualsiasi endpoint.</span><span class="sxs-lookup"><span data-stu-id="13792-346">`ConfigureEndpointDefaults` and `ConfigureHttpsDefaults` should be called before any endpoints are configured.</span></span>

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.ConfigureEndpointDefaults(listenOptions =>
    {
        // Configure endpoint defaults
    });

    serverOptions.ConfigureHttpsDefaults(listenOptions =>
    {
        listenOptions.SslProtocols = SslProtocols.Tls12;
    });
});
```

<span data-ttu-id="13792-347">*Supporto kestrel per SNI*</span><span class="sxs-lookup"><span data-stu-id="13792-347">*Kestrel support for SNI*</span></span>

<span data-ttu-id="13792-348">[L'Indicazione nome server (SNI)](https://tools.ietf.org/html/rfc6066#section-3) può essere usata per ospitare più domini sulla stessa porta e indirizzo IP.</span><span class="sxs-lookup"><span data-stu-id="13792-348">[Server Name Indication (SNI)](https://tools.ietf.org/html/rfc6066#section-3) can be used to host multiple domains on the same IP address and port.</span></span> <span data-ttu-id="13792-349">Perché SNI funzioni è necessario che il client invii al server il nome host per la sessione protetta durante l'handshake TLS in modo che il server possa specificare il certificato corretto.</span><span class="sxs-lookup"><span data-stu-id="13792-349">For SNI to function, the client sends the host name for the secure session to the server during the TLS handshake so that the server can provide the correct certificate.</span></span> <span data-ttu-id="13792-350">Il client usa il certificato specificato per la comunicazione crittografata con il server durante la sessione protetta che segue l'handshake TLS.</span><span class="sxs-lookup"><span data-stu-id="13792-350">The client uses the furnished certificate for encrypted communication with the server during the secure session that follows the TLS handshake.</span></span>

<span data-ttu-id="13792-351">Kestrel supporta SNI tramite il callback `ServerCertificateSelector`.</span><span class="sxs-lookup"><span data-stu-id="13792-351">Kestrel supports SNI via the `ServerCertificateSelector` callback.</span></span> <span data-ttu-id="13792-352">Il callback viene richiamato una volta per ogni connessione per consentire all'app di controllare il nome host e selezionare il certificato appropriato.</span><span class="sxs-lookup"><span data-stu-id="13792-352">The callback is invoked once per connection to allow the app to inspect the host name and select the appropriate certificate.</span></span>

<span data-ttu-id="13792-353">Il supporto SNI richiede:</span><span class="sxs-lookup"><span data-stu-id="13792-353">SNI support requires:</span></span>

* <span data-ttu-id="13792-354">In esecuzione su Framework di destinazione `netcoreapp2.1` o versione successiva.</span><span class="sxs-lookup"><span data-stu-id="13792-354">Running on target framework `netcoreapp2.1` or later.</span></span> <span data-ttu-id="13792-355">In `net461` o versioni successive, il callback viene richiamato, ma `name` è sempre `null` .</span><span class="sxs-lookup"><span data-stu-id="13792-355">On `net461` or later, the callback is invoked but the `name` is always `null`.</span></span> <span data-ttu-id="13792-356">L'elemento `name` è `null` anche se il client non specifica il parametro del nome host nell'handshake TLS.</span><span class="sxs-lookup"><span data-stu-id="13792-356">The `name` is also `null` if the client doesn't provide the host name parameter in the TLS handshake.</span></span>
* <span data-ttu-id="13792-357">Esecuzione di tutti i siti Web nella stessa istanza di Kestrel.</span><span class="sxs-lookup"><span data-stu-id="13792-357">All websites run on the same Kestrel instance.</span></span> <span data-ttu-id="13792-358">Kestrel supporta la condivisione di un indirizzo IP e di una porta tra più istanze solo con un proxy inverso.</span><span class="sxs-lookup"><span data-stu-id="13792-358">Kestrel doesn't support sharing an IP address and port across multiple instances without a reverse proxy.</span></span>

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.ListenAnyIP(5005, listenOptions =>
    {
        listenOptions.UseHttps(httpsOptions =>
        {
            var localhostCert = CertificateLoader.LoadFromStoreCert(
                "localhost", "My", StoreLocation.CurrentUser,
                allowInvalid: true);
            var exampleCert = CertificateLoader.LoadFromStoreCert(
                "example.com", "My", StoreLocation.CurrentUser,
                allowInvalid: true);
            var subExampleCert = CertificateLoader.LoadFromStoreCert(
                "sub.example.com", "My", StoreLocation.CurrentUser,
                allowInvalid: true);
            var certs = new Dictionary<string, X509Certificate2>(
                StringComparer.OrdinalIgnoreCase);
            certs["localhost"] = localhostCert;
            certs["example.com"] = exampleCert;
            certs["sub.example.com"] = subExampleCert;

            httpsOptions.ServerCertificateSelector = (connectionContext, name) =>
            {
                if (name != null && certs.TryGetValue(name, out var cert))
                {
                    return cert;
                }

                return exampleCert;
            };
        });
    });
});
```

### <a name="connection-logging"></a><span data-ttu-id="13792-359">Registrazione connessione</span><span class="sxs-lookup"><span data-stu-id="13792-359">Connection logging</span></span>

<span data-ttu-id="13792-360">Chiamare <xref:Microsoft.AspNetCore.Hosting.ListenOptionsConnectionLoggingExtensions.UseConnectionLogging*> per creare log a livello di debug per la comunicazione a livello di byte in una connessione.</span><span class="sxs-lookup"><span data-stu-id="13792-360">Call <xref:Microsoft.AspNetCore.Hosting.ListenOptionsConnectionLoggingExtensions.UseConnectionLogging*> to emit Debug level logs for byte-level communication on a connection.</span></span> <span data-ttu-id="13792-361">La registrazione delle connessioni è utile per la risoluzione dei problemi nella comunicazione di basso livello, ad esempio durante la crittografia TLS e dietro i proxy.</span><span class="sxs-lookup"><span data-stu-id="13792-361">Connection logging is helpful for troubleshooting problems in low-level communication, such as during TLS encryption and behind proxies.</span></span> <span data-ttu-id="13792-362">Se `UseConnectionLogging` viene inserito prima `UseHttps` , viene registrato il traffico crittografato.</span><span class="sxs-lookup"><span data-stu-id="13792-362">If `UseConnectionLogging` is placed before `UseHttps`, encrypted traffic is logged.</span></span> <span data-ttu-id="13792-363">Se `UseConnectionLogging` viene inserito dopo `UseHttps` , viene registrato il traffico decrittografato.</span><span class="sxs-lookup"><span data-stu-id="13792-363">If `UseConnectionLogging` is placed after `UseHttps`, decrypted traffic is logged.</span></span>

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.Listen(IPAddress.Any, 8000, listenOptions =>
    {
        listenOptions.UseConnectionLogging();
    });
});
```

### <a name="bind-to-a-tcp-socket"></a><span data-ttu-id="13792-364">Associazione a un socket TCP</span><span class="sxs-lookup"><span data-stu-id="13792-364">Bind to a TCP socket</span></span>

<span data-ttu-id="13792-365">Il metodo <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Listen*> esegue l'associazione a un socket TCP e un'espressione lambda per le opzioni consente di configurare un certificato X.509:</span><span class="sxs-lookup"><span data-stu-id="13792-365">The <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Listen*> method binds to a TCP socket, and an options lambda permits X.509 certificate configuration:</span></span>

[!code-csharp[](kestrel/samples/3.x/KestrelSample/Program.cs?name=snippet_TCPSocket&highlight=12-18)]

<span data-ttu-id="13792-366">L'esempio configura HTTPS per un endpoint con <xref:Microsoft.AspNetCore.Server.Kestrel.Core.ListenOptions>.</span><span class="sxs-lookup"><span data-stu-id="13792-366">The example configures HTTPS for an endpoint with <xref:Microsoft.AspNetCore.Server.Kestrel.Core.ListenOptions>.</span></span> <span data-ttu-id="13792-367">Usare la stessa API per configurare altre impostazioni di Kestrel per endpoint specifici.</span><span class="sxs-lookup"><span data-stu-id="13792-367">Use the same API to configure other Kestrel settings for specific endpoints.</span></span>

[!INCLUDE [How to make an X.509 cert](~/includes/make-x509-cert.md)]

### <a name="bind-to-a-unix-socket"></a><span data-ttu-id="13792-368">Associazione a un socket Unix</span><span class="sxs-lookup"><span data-stu-id="13792-368">Bind to a Unix socket</span></span>

<span data-ttu-id="13792-369">Impostare l'ascolto su un socket Unix con <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ListenUnixSocket*> per migliorare le prestazioni con Nginx, come visualizzato nell'esempio seguente:</span><span class="sxs-lookup"><span data-stu-id="13792-369">Listen on a Unix socket with <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ListenUnixSocket*> for improved performance with Nginx, as shown in this example:</span></span>

[!code-csharp[](kestrel/samples/3.x/KestrelSample/Program.cs?name=snippet_UnixSocket)]

* <span data-ttu-id="13792-370">Nel file di configurazione nginx impostare la `server`  >  `location`  >  `proxy_pass` voce su `http://unix:/tmp/{KESTREL SOCKET}:/;` .</span><span class="sxs-lookup"><span data-stu-id="13792-370">In the Nginx configuration file, set the `server` > `location` > `proxy_pass` entry to `http://unix:/tmp/{KESTREL SOCKET}:/;`.</span></span> <span data-ttu-id="13792-371">`{KESTREL SOCKET}` nome del socket fornito a <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ListenUnixSocket*> (ad esempio, `kestrel-test.sock` nell'esempio precedente).</span><span class="sxs-lookup"><span data-stu-id="13792-371">`{KESTREL SOCKET}` is the name of the socket provided to <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ListenUnixSocket*> (for example, `kestrel-test.sock` in the preceding example).</span></span>
* <span data-ttu-id="13792-372">Verificare che il socket sia scrivibile da nginx (ad esempio, `chmod go+w /tmp/kestrel-test.sock` ).</span><span class="sxs-lookup"><span data-stu-id="13792-372">Ensure that the socket is writeable by Nginx (for example, `chmod go+w /tmp/kestrel-test.sock`).</span></span>

### <a name="port-0"></a><span data-ttu-id="13792-373">Porta 0</span><span class="sxs-lookup"><span data-stu-id="13792-373">Port 0</span></span>

<span data-ttu-id="13792-374">Quando si specifica il numero di porta `0`, Kestrel esegue l'associazione dinamica a una porta disponibile.</span><span class="sxs-lookup"><span data-stu-id="13792-374">When the port number `0` is specified, Kestrel dynamically binds to an available port.</span></span> <span data-ttu-id="13792-375">L'esempio seguente indica come determinare la porta alla quale Kestrel ha eseguito l'associazione in fase di esecuzione:</span><span class="sxs-lookup"><span data-stu-id="13792-375">The following example shows how to determine which port Kestrel actually bound at runtime:</span></span>

[!code-csharp[](kestrel/samples/3.x/KestrelSample/Startup.cs?name=snippet_Configure&highlight=3-4,15-21)]

<span data-ttu-id="13792-376">Quando si esegue l'app, l'output della finestra della console indica la porta dinamica in cui l'app può essere raggiunta:</span><span class="sxs-lookup"><span data-stu-id="13792-376">When the app is run, the console window output indicates the dynamic port where the app can be reached:</span></span>

```console
Listening on the following addresses: http://127.0.0.1:48508
```

### <a name="limitations"></a><span data-ttu-id="13792-377">Limitazioni</span><span class="sxs-lookup"><span data-stu-id="13792-377">Limitations</span></span>

<span data-ttu-id="13792-378">Configurare gli endpoint con i metodi seguenti:</span><span class="sxs-lookup"><span data-stu-id="13792-378">Configure endpoints with the following approaches:</span></span>

* <xref:Microsoft.AspNetCore.Hosting.HostingAbstractionsWebHostBuilderExtensions.UseUrls*>
* <span data-ttu-id="13792-379">L'argomento della riga di comando `--urls`</span><span class="sxs-lookup"><span data-stu-id="13792-379">`--urls` command-line argument</span></span>
* <span data-ttu-id="13792-380">La chiave di configurazione dell'host `urls`</span><span class="sxs-lookup"><span data-stu-id="13792-380">`urls` host configuration key</span></span>
* <span data-ttu-id="13792-381">La variabile di ambiente `ASPNETCORE_URLS`</span><span class="sxs-lookup"><span data-stu-id="13792-381">`ASPNETCORE_URLS` environment variable</span></span>

<span data-ttu-id="13792-382">Questi metodi sono utili se si vuole che il codice funzioni con server diversi da Kestrel.</span><span class="sxs-lookup"><span data-stu-id="13792-382">These methods are useful for making code work with servers other than Kestrel.</span></span> <span data-ttu-id="13792-383">Tenere presenti, tuttavia, le limitazioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="13792-383">However, be aware of the following limitations:</span></span>

* <span data-ttu-id="13792-384">HTTPS non può essere usato con questi approcci, a meno che non venga specificato un certificato predefinito nella configurazione dell'endpoint HTTPS (ad esempio, usando la configurazione `KestrelServerOptions` o un file di configurazione come illustrato in precedenza in questo argomento).</span><span class="sxs-lookup"><span data-stu-id="13792-384">HTTPS can't be used with these approaches unless a default certificate is provided in the HTTPS endpoint configuration (for example, using `KestrelServerOptions` configuration or a configuration file as shown earlier in this topic).</span></span>
* <span data-ttu-id="13792-385">Quando sia l'approccio `Listen` che l'approccio `UseUrls` vengono usati contemporaneamente, gli endpoint `Listen` eseguono l'override degli endpoint `UseUrls`.</span><span class="sxs-lookup"><span data-stu-id="13792-385">When both the `Listen` and `UseUrls` approaches are used simultaneously, the `Listen` endpoints override the `UseUrls` endpoints.</span></span>

### <a name="iis-endpoint-configuration"></a><span data-ttu-id="13792-386">Configurazione dell'endpoint IIS</span><span class="sxs-lookup"><span data-stu-id="13792-386">IIS endpoint configuration</span></span>

<span data-ttu-id="13792-387">Quando si usa IIS, le associazioni di URL per le associazioni di override di IIS vengono impostate mediante `Listen` o `UseUrls`.</span><span class="sxs-lookup"><span data-stu-id="13792-387">When using IIS, the URL bindings for IIS override bindings are set by either `Listen` or `UseUrls`.</span></span> <span data-ttu-id="13792-388">Per altre informazioni, vedere l'articolo [Introduzione al modulo ASP.NET Core](xref:host-and-deploy/aspnet-core-module).</span><span class="sxs-lookup"><span data-stu-id="13792-388">For more information, see the [ASP.NET Core Module](xref:host-and-deploy/aspnet-core-module) topic.</span></span>

### <a name="listenoptionsprotocols"></a><span data-ttu-id="13792-389">ListenOptions.Protocols</span><span class="sxs-lookup"><span data-stu-id="13792-389">ListenOptions.Protocols</span></span>

<span data-ttu-id="13792-390">La proprietà `Protocols` stabilisce i protocolli HTTP (`HttpProtocols`) abilitati in un endpoint di connessione o per il server.</span><span class="sxs-lookup"><span data-stu-id="13792-390">The `Protocols` property establishes the HTTP protocols (`HttpProtocols`) enabled on a connection endpoint or for the server.</span></span> <span data-ttu-id="13792-391">Assegnare un valore alla proprietà `Protocols` dall'enumerazione `HttpProtocols`.</span><span class="sxs-lookup"><span data-stu-id="13792-391">Assign a value to the `Protocols` property from the `HttpProtocols` enum.</span></span>

| <span data-ttu-id="13792-392">Valore di enumerazione `HttpProtocols`</span><span class="sxs-lookup"><span data-stu-id="13792-392">`HttpProtocols` enum value</span></span> | <span data-ttu-id="13792-393">Protocollo di connessione consentito</span><span class="sxs-lookup"><span data-stu-id="13792-393">Connection protocol permitted</span></span> |
| -------------------------- | ----------------------------- |
| `Http1`                    | <span data-ttu-id="13792-394">Solo HTTP/1.1.</span><span class="sxs-lookup"><span data-stu-id="13792-394">HTTP/1.1 only.</span></span> <span data-ttu-id="13792-395">Può essere usato con o senza TLS.</span><span class="sxs-lookup"><span data-stu-id="13792-395">Can be used with or without TLS.</span></span> |
| `Http2`                    | <span data-ttu-id="13792-396">Solo HTTP/2.</span><span class="sxs-lookup"><span data-stu-id="13792-396">HTTP/2 only.</span></span> <span data-ttu-id="13792-397">Può essere usato senza TLS solo se il client supporta una [modalità di conoscenza pregressa](https://tools.ietf.org/html/rfc7540#section-3.4).</span><span class="sxs-lookup"><span data-stu-id="13792-397">May be used without TLS only if the client supports a [Prior Knowledge mode](https://tools.ietf.org/html/rfc7540#section-3.4).</span></span> |
| `Http1AndHttp2`            | <span data-ttu-id="13792-398">HTTP/1.1 e HTTP/2.</span><span class="sxs-lookup"><span data-stu-id="13792-398">HTTP/1.1 and HTTP/2.</span></span> <span data-ttu-id="13792-399">HTTP/2 richiede che il client selezioni HTTP/2 nell'handshake TLS [(Application-Layer Protocol negotiation) ALPN](https://tools.ietf.org/html/rfc7301#section-3) in caso contrario, il valore predefinito per la connessione è HTTP/1.1.</span><span class="sxs-lookup"><span data-stu-id="13792-399">HTTP/2 requires the client to select HTTP/2 in the TLS [Application-Layer Protocol Negotiation (ALPN)](https://tools.ietf.org/html/rfc7301#section-3) handshake; otherwise, the connection defaults to HTTP/1.1.</span></span> |

<span data-ttu-id="13792-400">Il `ListenOptions.Protocols` valore predefinito per qualsiasi endpoint è `HttpProtocols.Http1AndHttp2` .</span><span class="sxs-lookup"><span data-stu-id="13792-400">The default `ListenOptions.Protocols` value for any endpoint is `HttpProtocols.Http1AndHttp2`.</span></span>

<span data-ttu-id="13792-401">Restrizioni relative a TLS per HTTP/2:</span><span class="sxs-lookup"><span data-stu-id="13792-401">TLS restrictions for HTTP/2:</span></span>

* <span data-ttu-id="13792-402">TLS versione 1.2 o successiva</span><span class="sxs-lookup"><span data-stu-id="13792-402">TLS version 1.2 or later</span></span>
* <span data-ttu-id="13792-403">Rinegoziazione disabilitata</span><span class="sxs-lookup"><span data-stu-id="13792-403">Renegotiation disabled</span></span>
* <span data-ttu-id="13792-404">Compressione disabilitata</span><span class="sxs-lookup"><span data-stu-id="13792-404">Compression disabled</span></span>
* <span data-ttu-id="13792-405">Dimensioni minime per lo scambio di chiavi temporanee:</span><span class="sxs-lookup"><span data-stu-id="13792-405">Minimum ephemeral key exchange sizes:</span></span>
  * <span data-ttu-id="13792-406">Curva ellittica Diffie-Hellman (ECDHE) &lbrack; [RFC4492](https://www.ietf.org/rfc/rfc4492.txt) &rbrack; : minimo 224 bit</span><span class="sxs-lookup"><span data-stu-id="13792-406">Elliptic curve Diffie-Hellman (ECDHE) &lbrack;[RFC4492](https://www.ietf.org/rfc/rfc4492.txt)&rbrack;: 224 bits minimum</span></span>
  * <span data-ttu-id="13792-407">Diffie-Hellman campo finito (DHE) &lbrack; `TLS12` &rbrack; : minimo 2048 bit</span><span class="sxs-lookup"><span data-stu-id="13792-407">Finite field Diffie-Hellman (DHE) &lbrack;`TLS12`&rbrack;: 2048 bits minimum</span></span>
* <span data-ttu-id="13792-408">Il pacchetto di crittografia non è consentito.</span><span class="sxs-lookup"><span data-stu-id="13792-408">Cipher suite not prohibited.</span></span> 

<span data-ttu-id="13792-409">`TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256`&lbrack;`TLS-ECDHE`&rbrack; con la curva ellittica P-256 &lbrack; `FIPS186` &rbrack; è supportata per impostazione predefinita.</span><span class="sxs-lookup"><span data-stu-id="13792-409">`TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256` &lbrack;`TLS-ECDHE`&rbrack; with the P-256 elliptic curve &lbrack;`FIPS186`&rbrack; is supported by default.</span></span>

<span data-ttu-id="13792-410">Nell'esempio seguente sono consentite connessioni HTTP/1.1 e HTTP/2 sulla porta 8000.</span><span class="sxs-lookup"><span data-stu-id="13792-410">The following example permits HTTP/1.1 and HTTP/2 connections on port 8000.</span></span> <span data-ttu-id="13792-411">Le connessioni sono protette da TLS con un certificato incluso:</span><span class="sxs-lookup"><span data-stu-id="13792-411">Connections are secured by TLS with a supplied certificate:</span></span>

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.Listen(IPAddress.Any, 8000, listenOptions =>
    {
        listenOptions.UseHttps("testCert.pfx", "testPassword");
    });
});
```

<span data-ttu-id="13792-412">Usare il middleware di connessione per filtrare gli handshake TLS in base alla connessione per le crittografie specifiche, se necessario.</span><span class="sxs-lookup"><span data-stu-id="13792-412">Use Connection Middleware to filter TLS handshakes on a per-connection basis for specific ciphers if required.</span></span>

<span data-ttu-id="13792-413">L'esempio seguente genera un'eccezione <xref:System.NotSupportedException> per qualsiasi algoritmo crittografico non supportato dall'app.</span><span class="sxs-lookup"><span data-stu-id="13792-413">The following example throws <xref:System.NotSupportedException> for any cipher algorithm that the app doesn't support.</span></span> <span data-ttu-id="13792-414">In alternativa, definire e confrontare [ITlsHandshakeFeature. CipherAlgorithm](xref:Microsoft.AspNetCore.Connections.Features.ITlsHandshakeFeature.CipherAlgorithm) con un elenco di pacchetti di crittografia accettabili.</span><span class="sxs-lookup"><span data-stu-id="13792-414">Alternatively, define and compare [ITlsHandshakeFeature.CipherAlgorithm](xref:Microsoft.AspNetCore.Connections.Features.ITlsHandshakeFeature.CipherAlgorithm) to a list of acceptable cipher suites.</span></span>

<span data-ttu-id="13792-415">Non viene utilizzata alcuna crittografia con un algoritmo di crittografia [CipherAlgorithmType. null](xref:System.Security.Authentication.CipherAlgorithmType) .</span><span class="sxs-lookup"><span data-stu-id="13792-415">No encryption is used with a [CipherAlgorithmType.Null](xref:System.Security.Authentication.CipherAlgorithmType) cipher algorithm.</span></span>

```csharp
// using System.Net;
// using Microsoft.AspNetCore.Connections;

webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.Listen(IPAddress.Any, 8000, listenOptions =>
    {
        listenOptions.UseHttps("testCert.pfx", "testPassword");
        listenOptions.UseTlsFilter();
    });
});
```

```csharp
using System;
using System.Security.Authentication;
using Microsoft.AspNetCore.Connections.Features;

namespace Microsoft.AspNetCore.Connections
{
    public static class TlsFilterConnectionMiddlewareExtensions
    {
        public static IConnectionBuilder UseTlsFilter(
            this IConnectionBuilder builder)
        {
            return builder.Use((connection, next) =>
            {
                var tlsFeature = connection.Features.Get<ITlsHandshakeFeature>();

                if (tlsFeature.CipherAlgorithm == CipherAlgorithmType.Null)
                {
                    throw new NotSupportedException("Prohibited cipher: " +
                        tlsFeature.CipherAlgorithm);
                }

                return next();
            });
        }
    }
}
```

<span data-ttu-id="13792-416">Il filtro della connessione può essere configurato anche tramite un' <xref:Microsoft.AspNetCore.Connections.IConnectionBuilder> espressione lambda:</span><span class="sxs-lookup"><span data-stu-id="13792-416">Connection filtering can also be configured via an <xref:Microsoft.AspNetCore.Connections.IConnectionBuilder> lambda:</span></span>

```csharp
// using System;
// using System.Net;
// using System.Security.Authentication;
// using Microsoft.AspNetCore.Connections;
// using Microsoft.AspNetCore.Connections.Features;

webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.Listen(IPAddress.Any, 8000, listenOptions =>
    {
        listenOptions.UseHttps("testCert.pfx", "testPassword");
        listenOptions.Use((context, next) =>
        {
            var tlsFeature = context.Features.Get<ITlsHandshakeFeature>();

            if (tlsFeature.CipherAlgorithm == CipherAlgorithmType.Null)
            {
                throw new NotSupportedException(
                    $"Prohibited cipher: {tlsFeature.CipherAlgorithm}");
            }

            return next();
        });
    });
});
```

<span data-ttu-id="13792-417">In Linux <xref:System.Net.Security.CipherSuitesPolicy> è possibile usare per filtrare gli handshake TLS in base alla connessione:</span><span class="sxs-lookup"><span data-stu-id="13792-417">On Linux, <xref:System.Net.Security.CipherSuitesPolicy> can be used to filter TLS handshakes on a per-connection basis:</span></span>

```csharp
// using System.Net.Security;
// using Microsoft.AspNetCore.Hosting;
// using Microsoft.AspNetCore.Server.Kestrel.Core;
// using Microsoft.Extensions.DependencyInjection;
// using Microsoft.Extensions.Hosting;

webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.ConfigureHttpsDefaults(listenOptions =>
    {
        listenOptions.OnAuthenticate = (context, sslOptions) =>
        {
            sslOptions.CipherSuitesPolicy = new CipherSuitesPolicy(
                new[]
                {
                    TlsCipherSuite.TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,
                    TlsCipherSuite.TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384,
                    // ...
                });
        };
    });
});
```

<span data-ttu-id="13792-418">*Impostare il protocollo dalla configurazione*</span><span class="sxs-lookup"><span data-stu-id="13792-418">*Set the protocol from configuration*</span></span>

<span data-ttu-id="13792-419">`CreateDefaultBuilder` chiama `serverOptions.Configure(context.Configuration.GetSection("Kestrel"))` per impostazione predefinita per caricare la configurazione di Kestrel.</span><span class="sxs-lookup"><span data-stu-id="13792-419">`CreateDefaultBuilder` calls `serverOptions.Configure(context.Configuration.GetSection("Kestrel"))` by default to load Kestrel configuration.</span></span>

<span data-ttu-id="13792-420">Nell' *appsettings.json* esempio seguente viene stabilito http/1.1 come protocollo di connessione predefinito per tutti gli endpoint:</span><span class="sxs-lookup"><span data-stu-id="13792-420">The following *appsettings.json* example establishes HTTP/1.1 as the default connection protocol for all endpoints:</span></span>

```json
{
  "Kestrel": {
    "EndpointDefaults": {
      "Protocols": "Http1"
    }
  }
}
```

<span data-ttu-id="13792-421">Nell' *appsettings.json* esempio seguente viene stabilito il protocollo di connessione HTTP/1.1 per un endpoint specifico:</span><span class="sxs-lookup"><span data-stu-id="13792-421">The following *appsettings.json* example establishes the HTTP/1.1 connection protocol for a specific endpoint:</span></span>

```json
{
  "Kestrel": {
    "Endpoints": {
      "HttpsDefaultCert": {
        "Url": "https://localhost:5001",
        "Protocols": "Http1"
      }
    }
  }
}
```

<span data-ttu-id="13792-422">I protocolli specificati nei valori di override del codice sono impostati dalla configurazione.</span><span class="sxs-lookup"><span data-stu-id="13792-422">Protocols specified in code override values set by configuration.</span></span>

### <a name="url-prefixes"></a><span data-ttu-id="13792-423">Prefissi URL</span><span class="sxs-lookup"><span data-stu-id="13792-423">URL prefixes</span></span>

<span data-ttu-id="13792-424">Se si usa `UseUrls`, l'argomento della riga di comando `--urls`, la chiave di configurazione dell'host `urls` o la variabile di ambiente `ASPNETCORE_URLS`, i prefissi URL possono avere uno dei formati seguenti.</span><span class="sxs-lookup"><span data-stu-id="13792-424">When using `UseUrls`, `--urls` command-line argument, `urls` host configuration key, or `ASPNETCORE_URLS` environment variable, the URL prefixes can be in any of the following formats.</span></span>

<span data-ttu-id="13792-425">Sono validi solo i prefissi URL HTTP.</span><span class="sxs-lookup"><span data-stu-id="13792-425">Only HTTP URL prefixes are valid.</span></span> <span data-ttu-id="13792-426">Kestrel non supporta HTTPS quando la configurazione delle associazioni URL viene eseguita con `UseUrls`.</span><span class="sxs-lookup"><span data-stu-id="13792-426">Kestrel doesn't support HTTPS when configuring URL bindings using `UseUrls`.</span></span>

* <span data-ttu-id="13792-427">Indirizzo IPv4 con numero di porta</span><span class="sxs-lookup"><span data-stu-id="13792-427">IPv4 address with port number</span></span>

  ```
  http://65.55.39.10:80/
  ```

  <span data-ttu-id="13792-428">`0.0.0.0` è un caso speciale che esegue l'associazione a tutti gli indirizzi IPv4.</span><span class="sxs-lookup"><span data-stu-id="13792-428">`0.0.0.0` is a special case that binds to all IPv4 addresses.</span></span>

* <span data-ttu-id="13792-429">Indirizzo IPv6 con numero di porta</span><span class="sxs-lookup"><span data-stu-id="13792-429">IPv6 address with port number</span></span>

  ```
  http://[0:0:0:0:0:ffff:4137:270a]:80/
  ```

  <span data-ttu-id="13792-430">`[::]` è l'equivalente IPv6 di `0.0.0.0` per IPv4.</span><span class="sxs-lookup"><span data-stu-id="13792-430">`[::]` is the IPv6 equivalent of IPv4 `0.0.0.0`.</span></span>

* <span data-ttu-id="13792-431">Nome host con numero di porta</span><span class="sxs-lookup"><span data-stu-id="13792-431">Host name with port number</span></span>

  ```
  http://contoso.com:80/
  http://*:80/
  ```

  <span data-ttu-id="13792-432">I nomi host, `*` e `+` non sono casi particolari.</span><span class="sxs-lookup"><span data-stu-id="13792-432">Host names, `*`, and `+`, aren't special.</span></span> <span data-ttu-id="13792-433">Tutto ciò che non è riconosciuto come un indirizzo IP o un elemento `localhost` valido esegue l'associazione a tutti gli IP IPv4 e IPv6.</span><span class="sxs-lookup"><span data-stu-id="13792-433">Anything not recognized as a valid IP address or `localhost` binds to all IPv4 and IPv6 IPs.</span></span> <span data-ttu-id="13792-434">Per associare nomi host diversi ad app ASP.NET Core diverse sulla stessa porta, usare [HTTP.sys](xref:fundamentals/servers/httpsys) o un server proxy inverso come IIS, Nginx o Apache.</span><span class="sxs-lookup"><span data-stu-id="13792-434">To bind different host names to different ASP.NET Core apps on the same port, use [HTTP.sys](xref:fundamentals/servers/httpsys) or a reverse proxy server, such as IIS, Nginx, or Apache.</span></span>

  > [!WARNING]
  > <span data-ttu-id="13792-435">Una configurazione che prevede un proxy inverso richiede il [filtro host](#host-filtering).</span><span class="sxs-lookup"><span data-stu-id="13792-435">Hosting in a reverse proxy configuration requires [host filtering](#host-filtering).</span></span>

* <span data-ttu-id="13792-436">Nome host `localhost` con numero di porta o IP di loopback con numero di porta</span><span class="sxs-lookup"><span data-stu-id="13792-436">Host `localhost` name with port number or loopback IP with port number</span></span>

  ```
  http://localhost:5000/
  http://127.0.0.1:5000/
  http://[::1]:5000/
  ```

  <span data-ttu-id="13792-437">Se è specificato `localhost`, Kestrel tenta l'associazione alle interfacce di loopback sia IPv4 che IPv6.</span><span class="sxs-lookup"><span data-stu-id="13792-437">When `localhost` is specified, Kestrel attempts to bind to both IPv4 and IPv6 loopback interfaces.</span></span> <span data-ttu-id="13792-438">Se la porta richiesta è in uso da parte di un altro servizio in una delle due interfacce di loopback, Kestrel non viene avviato.</span><span class="sxs-lookup"><span data-stu-id="13792-438">If the requested port is in use by another service on either loopback interface, Kestrel fails to start.</span></span> <span data-ttu-id="13792-439">Se una delle due interfacce di loopback non è disponibile per qualsiasi altro motivo (in genere perché IPv6 non è supportato), Kestrel registra un avviso.</span><span class="sxs-lookup"><span data-stu-id="13792-439">If either loopback interface is unavailable for any other reason (most commonly because IPv6 isn't supported), Kestrel logs a warning.</span></span>

## <a name="host-filtering"></a><span data-ttu-id="13792-440">Filtro host</span><span class="sxs-lookup"><span data-stu-id="13792-440">Host filtering</span></span>

<span data-ttu-id="13792-441">Mentre supporta la configurazione in base ai prefissi, ad esempio `http://example.com:5000`, Kestrel ignora quasi sempre il nome host.</span><span class="sxs-lookup"><span data-stu-id="13792-441">While Kestrel supports configuration based on prefixes such as `http://example.com:5000`, Kestrel largely ignores the host name.</span></span> <span data-ttu-id="13792-442">L'host `localhost` è un caso speciale usato per l'associazione agli indirizzi di loopback.</span><span class="sxs-lookup"><span data-stu-id="13792-442">Host `localhost` is a special case used for binding to loopback addresses.</span></span> <span data-ttu-id="13792-443">Qualsiasi host che non sia un indirizzo IP esplicito esegue l'associazione a tutti gli indirizzi IP pubblici.</span><span class="sxs-lookup"><span data-stu-id="13792-443">Any host other than an explicit IP address binds to all public IP addresses.</span></span> <span data-ttu-id="13792-444">Le intestazioni `Host` non vengono convalidate.</span><span class="sxs-lookup"><span data-stu-id="13792-444">`Host` headers aren't validated.</span></span>

<span data-ttu-id="13792-445">Come soluzione alternativa, usare il middleware di filtro host.</span><span class="sxs-lookup"><span data-stu-id="13792-445">As a workaround, use Host Filtering Middleware.</span></span> <span data-ttu-id="13792-446">Il middleware di filtro host viene fornito dal pacchetto [Microsoft. AspNetCore. HostFiltering](https://www.nuget.org/packages/Microsoft.AspNetCore.HostFiltering) , fornito in modo implicito per le app ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="13792-446">Host Filtering Middleware is provided by the [Microsoft.AspNetCore.HostFiltering](https://www.nuget.org/packages/Microsoft.AspNetCore.HostFiltering) package, which is implicitly provided for ASP.NET Core apps.</span></span> <span data-ttu-id="13792-447">Il middleware viene aggiunto da <xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder*>, che chiama <xref:Microsoft.AspNetCore.Builder.HostFilteringServicesExtensions.AddHostFiltering*>:</span><span class="sxs-lookup"><span data-stu-id="13792-447">The middleware is added by <xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder*>, which calls <xref:Microsoft.AspNetCore.Builder.HostFilteringServicesExtensions.AddHostFiltering*>:</span></span>

[!code-csharp[](kestrel/samples-snapshot/2.x/KestrelSample/Program.cs?name=snippet_Program&highlight=9)]

<span data-ttu-id="13792-448">Per impostazione predefinita, il middleware di filtro host è disabilitato per impostazione predefinita.</span><span class="sxs-lookup"><span data-stu-id="13792-448">Host Filtering Middleware is disabled by default.</span></span> <span data-ttu-id="13792-449">Per abilitare il middleware, definire una `AllowedHosts` chiave in *appsettings.json* / *appSettings. \<EnvironmentName> JSON*.</span><span class="sxs-lookup"><span data-stu-id="13792-449">To enable the middleware, define an `AllowedHosts` key in *appsettings.json*/*appsettings.\<EnvironmentName>.json*.</span></span> <span data-ttu-id="13792-450">Il valore è un elenco con valori delimitati da punto e virgola di nomi host senza numeri di porta:</span><span class="sxs-lookup"><span data-stu-id="13792-450">The value is a semicolon-delimited list of host names without port numbers:</span></span>

<span data-ttu-id="13792-451">*appsettings.json*:</span><span class="sxs-lookup"><span data-stu-id="13792-451">*appsettings.json*:</span></span>

```json
{
  "AllowedHosts": "example.com;localhost"
}
```

> [!NOTE]
> <span data-ttu-id="13792-452">Il [middleware delle intestazioni inoltrate](xref:host-and-deploy/proxy-load-balancer) include anche un'opzione <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.AllowedHosts>.</span><span class="sxs-lookup"><span data-stu-id="13792-452">[Forwarded Headers Middleware](xref:host-and-deploy/proxy-load-balancer) also has an <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.AllowedHosts> option.</span></span> <span data-ttu-id="13792-453">Il middleware di intestazioni inoltrate e il middleware di filtro host hanno funzionalità simili per diversi scenari.</span><span class="sxs-lookup"><span data-stu-id="13792-453">Forwarded Headers Middleware and Host Filtering Middleware have similar functionality for different scenarios.</span></span> <span data-ttu-id="13792-454">L'impostazione di `AllowedHosts` con il middleware delle intestazioni inoltrate è appropriato se l'intestazione `Host` non viene mantenuta durante l'inoltro delle richieste con un server proxy inverso o il bilanciamento del carico.</span><span class="sxs-lookup"><span data-stu-id="13792-454">Setting `AllowedHosts` with Forwarded Headers Middleware is appropriate when the `Host` header isn't preserved while forwarding requests with a reverse proxy server or load balancer.</span></span> <span data-ttu-id="13792-455">L'impostazione di `AllowedHosts` con il middleware del filtro host è appropriata se si usa Kestrel come server perimetrale rivolto al pubblico o se l'intestazione `Host` viene inoltrata direttamente.</span><span class="sxs-lookup"><span data-stu-id="13792-455">Setting `AllowedHosts` with Host Filtering Middleware is appropriate when Kestrel is used as a public-facing edge server or when the `Host` header is directly forwarded.</span></span>
>
> <span data-ttu-id="13792-456">Per altre informazioni sul middleware delle intestazioni inoltrate, vedere <xref:host-and-deploy/proxy-load-balancer>.</span><span class="sxs-lookup"><span data-stu-id="13792-456">For more information on Forwarded Headers Middleware, see <xref:host-and-deploy/proxy-load-balancer>.</span></span>

## <a name="libuv-transport-configuration"></a><span data-ttu-id="13792-457">Configurazione del trasporto libuv</span><span class="sxs-lookup"><span data-stu-id="13792-457">Libuv transport configuration</span></span>

<span data-ttu-id="13792-458">Per i progetti che richiedono l'utilizzo di libuv ( <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderLibuvExtensions.UseLibuv%2A> ):</span><span class="sxs-lookup"><span data-stu-id="13792-458">For projects that require the use of Libuv (<xref:Microsoft.AspNetCore.Hosting.WebHostBuilderLibuvExtensions.UseLibuv%2A>):</span></span>

* <span data-ttu-id="13792-459">Aggiungere una dipendenza per il [`Microsoft.AspNetCore.Server.Kestrel.Transport.Libuv`](https://www.nuget.org/packages/Microsoft.AspNetCore.Server.Kestrel.Transport.Libuv) pacchetto al file di progetto dell'app:</span><span class="sxs-lookup"><span data-stu-id="13792-459">Add a dependency for the [`Microsoft.AspNetCore.Server.Kestrel.Transport.Libuv`](https://www.nuget.org/packages/Microsoft.AspNetCore.Server.Kestrel.Transport.Libuv) package to the app's project file:</span></span>

  ```xml
  <PackageReference Include="Microsoft.AspNetCore.Server.Kestrel.Transport.Libuv"
                    Version="{VERSION}" />
  ```

* <span data-ttu-id="13792-460">Chiamare <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderLibuvExtensions.UseLibuv%2A> su `IWebHostBuilder` :</span><span class="sxs-lookup"><span data-stu-id="13792-460">Call <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderLibuvExtensions.UseLibuv%2A> on the `IWebHostBuilder`:</span></span>

  ```csharp
  public class Program
  {
      public static void Main(string[] args)
      {
          CreateHostBuilder(args).Build().Run();
      }

      public static IHostBuilder CreateHostBuilder(string[] args) =>
          Host.CreateDefaultBuilder(args)
              .ConfigureWebHostDefaults(webBuilder =>
              {
                  webBuilder.UseLibuv();
                  webBuilder.UseStartup<Startup>();
              });
  }
  ```

## <a name="http11-request-draining"></a><span data-ttu-id="13792-461">Svuotamento richieste HTTP/1.1</span><span class="sxs-lookup"><span data-stu-id="13792-461">HTTP/1.1 request draining</span></span>

<span data-ttu-id="13792-462">L'apertura di connessioni HTTP richiede molto tempo.</span><span class="sxs-lookup"><span data-stu-id="13792-462">Opening HTTP connections is time consuming.</span></span> <span data-ttu-id="13792-463">Per HTTPS, è anche a elevato utilizzo di risorse.</span><span class="sxs-lookup"><span data-stu-id="13792-463">For HTTPS, it's also resource intensive.</span></span> <span data-ttu-id="13792-464">Pertanto, gheppio tenta di riutilizzare le connessioni per il protocollo HTTP/1.1.</span><span class="sxs-lookup"><span data-stu-id="13792-464">Therefore, Kestrel tries to reuse connections per the HTTP/1.1 protocol.</span></span> <span data-ttu-id="13792-465">Il corpo della richiesta deve essere completamente utilizzato per consentire il riutilizzo della connessione.</span><span class="sxs-lookup"><span data-stu-id="13792-465">A request body must be fully consumed to allow the connection to be reused.</span></span> <span data-ttu-id="13792-466">L'app non usa sempre il corpo della richiesta, ad esempio le `POST` richieste in cui il server restituisce una risposta di reindirizzamento o 404.</span><span class="sxs-lookup"><span data-stu-id="13792-466">The app doesn't always consume the request body, such as a `POST` requests where the server returns a redirect or 404 response.</span></span> <span data-ttu-id="13792-467">Nel `POST` caso-Reindirizzamento:</span><span class="sxs-lookup"><span data-stu-id="13792-467">In the `POST`-redirect case:</span></span>

* <span data-ttu-id="13792-468">Il client potrebbe avere già inviato parte dei `POST` dati.</span><span class="sxs-lookup"><span data-stu-id="13792-468">The client may already have sent part of the `POST` data.</span></span>
* <span data-ttu-id="13792-469">Il server scrive la risposta 301.</span><span class="sxs-lookup"><span data-stu-id="13792-469">The server writes the 301 response.</span></span>
* <span data-ttu-id="13792-470">Non è possibile usare la connessione per una nuova richiesta fino a quando i `POST` dati del corpo della richiesta precedente non sono stati completamente letti.</span><span class="sxs-lookup"><span data-stu-id="13792-470">The connection can't be used for a new request until the `POST` data from the previous request body has been fully read.</span></span>
* <span data-ttu-id="13792-471">Gheppio tenta di svuotare il corpo della richiesta.</span><span class="sxs-lookup"><span data-stu-id="13792-471">Kestrel tries to drain the request body.</span></span> <span data-ttu-id="13792-472">Lo svuotamento del corpo della richiesta implica la lettura e l'eliminazione dei dati senza elaborarli.</span><span class="sxs-lookup"><span data-stu-id="13792-472">Draining the request body means reading and discarding the data without processing it.</span></span>

<span data-ttu-id="13792-473">Il processo di svuotamento crea un tradoff tra la possibilità di riutilizzare la connessione e il tempo necessario per svuotare i dati rimanenti:</span><span class="sxs-lookup"><span data-stu-id="13792-473">The draining process makes a tradoff between allowing the connection to be reused and the time it takes to drain any remaining data:</span></span>

* <span data-ttu-id="13792-474">Lo svuotamento ha un timeout di cinque secondi, che non è configurabile.</span><span class="sxs-lookup"><span data-stu-id="13792-474">Draining has a timeout of five seconds, which isn't configurable.</span></span>
* <span data-ttu-id="13792-475">Se tutti i dati specificati dall' `Content-Length` `Transfer-Encoding` intestazione o non sono stati letti prima del timeout, la connessione viene chiusa.</span><span class="sxs-lookup"><span data-stu-id="13792-475">If all of the data specified by the `Content-Length` or `Transfer-Encoding` header hasn't been read before the timeout, the connection is closed.</span></span>

<span data-ttu-id="13792-476">In alcuni casi può essere necessario terminare immediatamente la richiesta, prima o dopo la scrittura della risposta.</span><span class="sxs-lookup"><span data-stu-id="13792-476">Sometimes you may want to terminate the request immediately, before or after writing the response.</span></span> <span data-ttu-id="13792-477">È ad esempio possibile che i client dispongano di protezioni dati restrittive, pertanto la limitazione dei dati caricati potrebbe avere una priorità.</span><span class="sxs-lookup"><span data-stu-id="13792-477">For example, clients may have restrictive data caps, so limiting uploaded data might be a priority.</span></span> <span data-ttu-id="13792-478">In questi casi, per terminare una richiesta, chiamare [HttpContext. Abort](xref:Microsoft.AspNetCore.Http.HttpContext.Abort%2A) da un controller, una Razor pagina o un middleware.</span><span class="sxs-lookup"><span data-stu-id="13792-478">In such cases to terminate a request, call [HttpContext.Abort](xref:Microsoft.AspNetCore.Http.HttpContext.Abort%2A) from a controller, Razor Page, or middleware.</span></span>

<span data-ttu-id="13792-479">Ci sono alcune avvertenze per chiamare `Abort` :</span><span class="sxs-lookup"><span data-stu-id="13792-479">There are caveats to calling `Abort`:</span></span>

* <span data-ttu-id="13792-480">La creazione di nuove connessioni può essere lenta e dispendiosa.</span><span class="sxs-lookup"><span data-stu-id="13792-480">Creating new connections can be slow and expensive.</span></span>
* <span data-ttu-id="13792-481">Non vi è alcuna garanzia che il client abbia letto la risposta prima che la connessione venga chiusa.</span><span class="sxs-lookup"><span data-stu-id="13792-481">There's no guarantee that the client has read the response before the connection closes.</span></span>
* <span data-ttu-id="13792-482">`Abort`La chiamata a deve essere rara e riservata per i casi di errore gravi, non per errori comuni.</span><span class="sxs-lookup"><span data-stu-id="13792-482">Calling `Abort` should be rare and reserved for severe error cases, not common errors.</span></span>
  * <span data-ttu-id="13792-483">Chiamare solo `Abort` quando è necessario risolvere un problema specifico.</span><span class="sxs-lookup"><span data-stu-id="13792-483">Only call `Abort` when a specific problem needs to be solved.</span></span> <span data-ttu-id="13792-484">Ad esempio, chiamare `Abort` se i client malintenzionati tentano di eseguire `POST` dati o quando si verifica un bug nel codice client che genera richieste di grandi dimensioni o numerose.</span><span class="sxs-lookup"><span data-stu-id="13792-484">For example, call `Abort` if malicious clients are trying to `POST` data or when there's a bug in client code that causes large or numerous requests.</span></span>
  * <span data-ttu-id="13792-485">Non chiamare le `Abort` situazioni di errore comuni, ad esempio HTTP 404 (non trovato).</span><span class="sxs-lookup"><span data-stu-id="13792-485">Don't call `Abort` for common error situations, such as HTTP 404 (Not Found).</span></span>

<span data-ttu-id="13792-486">Quando si chiama [HttpResponse. CompleteAsync](xref:Microsoft.AspNetCore.Http.HttpResponse.CompleteAsync%2A) , prima `Abort` di chiamare si garantisce che il server abbia completato la scrittura della risposta.</span><span class="sxs-lookup"><span data-stu-id="13792-486">Calling [HttpResponse.CompleteAsync](xref:Microsoft.AspNetCore.Http.HttpResponse.CompleteAsync%2A) before calling `Abort` ensures that the server has completed writing the response.</span></span> <span data-ttu-id="13792-487">Tuttavia, il comportamento del client non è prevedibile e potrebbe non essere in grado di leggere la risposta prima che la connessione venga interrotta.</span><span class="sxs-lookup"><span data-stu-id="13792-487">However, client behavior isn't predictable and they may not read the response before the connection is aborted.</span></span>

<span data-ttu-id="13792-488">Questo processo è diverso per HTTP/2 poiché il protocollo supporta l'interruzione dei singoli flussi di richiesta senza chiudere la connessione.</span><span class="sxs-lookup"><span data-stu-id="13792-488">This process is different for HTTP/2 because the protocol supports aborting individual request streams without closing the connection.</span></span> <span data-ttu-id="13792-489">Il timeout di svuotamento di cinque secondi non è applicabile.</span><span class="sxs-lookup"><span data-stu-id="13792-489">The five second drain timeout doesn't apply.</span></span> <span data-ttu-id="13792-490">Se dopo il completamento di una risposta sono presenti dati del corpo della richiesta non letti, il server invia un frame RST HTTP/2.</span><span class="sxs-lookup"><span data-stu-id="13792-490">If there's any unread request body data after completing a response, then the server sends an HTTP/2 RST frame.</span></span> <span data-ttu-id="13792-491">I frame di dati del corpo della richiesta aggiuntivi vengono ignorati.</span><span class="sxs-lookup"><span data-stu-id="13792-491">Additional request body data frames are ignored.</span></span>

<span data-ttu-id="13792-492">Se possibile, è preferibile che i client utilizzino l'intestazione della richiesta [expect: 100-continue](https://developer.mozilla.org/docs/Web/HTTP/Status/100) e attendano la risposta del server prima di iniziare a inviare il corpo della richiesta.</span><span class="sxs-lookup"><span data-stu-id="13792-492">If possible, it's better for clients to utilize the [Expect: 100-continue](https://developer.mozilla.org/docs/Web/HTTP/Status/100) request header and wait for the server to respond before starting to send the request body.</span></span> <span data-ttu-id="13792-493">Che fornisce al client la possibilità di esaminare la risposta e interrompere prima di inviare dati non necessari.</span><span class="sxs-lookup"><span data-stu-id="13792-493">That gives the client an opportunity to examine the response and abort before sending unneeded data.</span></span>

## <a name="additional-resources"></a><span data-ttu-id="13792-494">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="13792-494">Additional resources</span></span>

* <span data-ttu-id="13792-495">Quando si usano i socket UNIX in Linux, il socket non viene eliminato automaticamente all'arresto dell'app.</span><span class="sxs-lookup"><span data-stu-id="13792-495">When using UNIX sockets on Linux, the socket is not automatically deleted on app shut down.</span></span> <span data-ttu-id="13792-496">Per altre informazioni, vedere [questo problema di GitHub](https://github.com/dotnet/aspnetcore/issues/14134).</span><span class="sxs-lookup"><span data-stu-id="13792-496">For more information, see [this GitHub issue](https://github.com/dotnet/aspnetcore/issues/14134).</span></span>
* <xref:test/troubleshoot>
* <xref:security/enforcing-ssl>
* <xref:host-and-deploy/proxy-load-balancer>
* <span data-ttu-id="13792-497">[RFC 7230: Message Syntax and Routing (Section 5.4: Host)](https://tools.ietf.org/html/rfc7230#section-5.4) (RFC 7230: Sintassi e routing dei messaggi (sezione 5.4: Host))</span><span class="sxs-lookup"><span data-stu-id="13792-497">[RFC 7230: Message Syntax and Routing (Section 5.4: Host)](https://tools.ietf.org/html/rfc7230#section-5.4)</span></span>

::: moniker-end

::: moniker range="= aspnetcore-2.2"

<span data-ttu-id="13792-498">Kestrel è un [server Web per ASP.NET Core](xref:fundamentals/servers/index) multipiattaforma.</span><span class="sxs-lookup"><span data-stu-id="13792-498">Kestrel is a cross-platform [web server for ASP.NET Core](xref:fundamentals/servers/index).</span></span> <span data-ttu-id="13792-499">Kestrel è il server Web incluso per impostazione predefinita nei modelli di progetto di ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="13792-499">Kestrel is the web server that's included by default in ASP.NET Core project templates.</span></span>

<span data-ttu-id="13792-500">Kestrel supporta gli scenari seguenti:</span><span class="sxs-lookup"><span data-stu-id="13792-500">Kestrel supports the following scenarios:</span></span>

* <span data-ttu-id="13792-501">HTTPS</span><span class="sxs-lookup"><span data-stu-id="13792-501">HTTPS</span></span>
* <span data-ttu-id="13792-502">Aggiornamento opaco usato per abilitare [WebSocket](https://github.com/aspnet/websockets)</span><span class="sxs-lookup"><span data-stu-id="13792-502">Opaque upgrade used to enable [WebSockets](https://github.com/aspnet/websockets)</span></span>
* <span data-ttu-id="13792-503">Socket Unix ad alte prestazioni dietro Nginx</span><span class="sxs-lookup"><span data-stu-id="13792-503">Unix sockets for high performance behind Nginx</span></span>
* <span data-ttu-id="13792-504">HTTP/2 (tranne che in macOS&dagger;)</span><span class="sxs-lookup"><span data-stu-id="13792-504">HTTP/2 (except on macOS&dagger;)</span></span>

<span data-ttu-id="13792-505">&dagger;HTTP/2 verrà supportato in macOS in una versione futura.</span><span class="sxs-lookup"><span data-stu-id="13792-505">&dagger;HTTP/2 will be supported on macOS in a future release.</span></span>

<span data-ttu-id="13792-506">Kestrel è supportato in tutte le piattaforme e le versioni supportate da .NET Core.</span><span class="sxs-lookup"><span data-stu-id="13792-506">Kestrel is supported on all platforms and versions that .NET Core supports.</span></span>

<span data-ttu-id="13792-507">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/servers/kestrel/samples/2.x) ([procedura per il download](xref:index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="13792-507">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/servers/kestrel/samples/2.x) ([how to download](xref:index#how-to-download-a-sample))</span></span>

## <a name="http2-support"></a><span data-ttu-id="13792-508">Supporto HTTP/2</span><span class="sxs-lookup"><span data-stu-id="13792-508">HTTP/2 support</span></span>

<span data-ttu-id="13792-509">[HTTP/2](https://httpwg.org/specs/rfc7540.html) è disponibile per le app ASP.NET Core se vengono soddisfatti i requisiti di base seguenti:</span><span class="sxs-lookup"><span data-stu-id="13792-509">[HTTP/2](https://httpwg.org/specs/rfc7540.html) is available for ASP.NET Core apps if the following base requirements are met:</span></span>

* <span data-ttu-id="13792-510">Sistema operativo&dagger;</span><span class="sxs-lookup"><span data-stu-id="13792-510">Operating system&dagger;</span></span>
  * <span data-ttu-id="13792-511">Windows Server 2016/Windows 10 o versioni successive&Dagger;</span><span class="sxs-lookup"><span data-stu-id="13792-511">Windows Server 2016/Windows 10 or later&Dagger;</span></span>
  * <span data-ttu-id="13792-512">Linux con OpenSSL 1.0.2 o versioni successive (ad esempio, Ubuntu 16.04 o versioni successive)</span><span class="sxs-lookup"><span data-stu-id="13792-512">Linux with OpenSSL 1.0.2 or later (for example, Ubuntu 16.04 or later)</span></span>
* <span data-ttu-id="13792-513">Framework di destinazione: .NET Core 2.2 o versioni successive</span><span class="sxs-lookup"><span data-stu-id="13792-513">Target framework: .NET Core 2.2 or later</span></span>
* <span data-ttu-id="13792-514">Connessione [ALPN (Application-Layer Protocol Negotiation)](https://tools.ietf.org/html/rfc7301#section-3)</span><span class="sxs-lookup"><span data-stu-id="13792-514">[Application-Layer Protocol Negotiation (ALPN)](https://tools.ietf.org/html/rfc7301#section-3) connection</span></span>
* <span data-ttu-id="13792-515">Connessione TLS 1.2 o successiva</span><span class="sxs-lookup"><span data-stu-id="13792-515">TLS 1.2 or later connection</span></span>

<span data-ttu-id="13792-516">&dagger;HTTP/2 verrà supportato in macOS in una versione futura.</span><span class="sxs-lookup"><span data-stu-id="13792-516">&dagger;HTTP/2 will be supported on macOS in a future release.</span></span>
<span data-ttu-id="13792-517">&Dagger;Kestrel ha un supporto limitato per HTTP/2 in Windows Server 2012 R2 e Windows 8.1.</span><span class="sxs-lookup"><span data-stu-id="13792-517">&Dagger;Kestrel has limited support for HTTP/2 on Windows Server 2012 R2 and Windows 8.1.</span></span> <span data-ttu-id="13792-518">Il supporto è limitato perché l'elenco di suite di crittografia TLS supportate disponibili in questi sistemi operativi è limitato.</span><span class="sxs-lookup"><span data-stu-id="13792-518">Support is limited because the list of supported TLS cipher suites available on these operating systems is limited.</span></span> <span data-ttu-id="13792-519">Un certificato generato con un algoritmo ECDSA potrebbe essere necessario per proteggere le connessioni TLS.</span><span class="sxs-lookup"><span data-stu-id="13792-519">A certificate generated using an Elliptic Curve Digital Signature Algorithm (ECDSA) may be required to secure TLS connections.</span></span>

<span data-ttu-id="13792-520">Se viene stabilita una connessione HTTP/2, [HttpRequest.Protocol](xref:Microsoft.AspNetCore.Http.HttpRequest.Protocol*) corrisponde a `HTTP/2`.</span><span class="sxs-lookup"><span data-stu-id="13792-520">If an HTTP/2 connection is established, [HttpRequest.Protocol](xref:Microsoft.AspNetCore.Http.HttpRequest.Protocol*) reports `HTTP/2`.</span></span>

<span data-ttu-id="13792-521">HTTP/2 è disabilitato per impostazione predefinita.</span><span class="sxs-lookup"><span data-stu-id="13792-521">HTTP/2 is disabled by default.</span></span> <span data-ttu-id="13792-522">Per altre informazioni sulla configurazione, vedere le sezioni [Opzioni Kestrel](#kestrel-options) e [ListenOptions.Protocols](#listenoptionsprotocols).</span><span class="sxs-lookup"><span data-stu-id="13792-522">For more information on configuration, see the [Kestrel options](#kestrel-options) and [ListenOptions.Protocols](#listenoptionsprotocols) sections.</span></span>

## <a name="when-to-use-kestrel-with-a-reverse-proxy"></a><span data-ttu-id="13792-523">Quando usare Kestrel con un proxy inverso</span><span class="sxs-lookup"><span data-stu-id="13792-523">When to use Kestrel with a reverse proxy</span></span>

<span data-ttu-id="13792-524">È possibile usare Kestrel da solo o in combinazione con un *server proxy inverso*, ad esempio [Internet Information Services (IIS)](https://www.iis.net/), [Nginx](https://nginx.org) o [Apache](https://httpd.apache.org/).</span><span class="sxs-lookup"><span data-stu-id="13792-524">Kestrel can be used by itself or with a *reverse proxy server*, such as [Internet Information Services (IIS)](https://www.iis.net/), [Nginx](https://nginx.org), or [Apache](https://httpd.apache.org/).</span></span> <span data-ttu-id="13792-525">Il server proxy inverso riceve le richieste HTTP dalla rete e le inoltra a Kestrel.</span><span class="sxs-lookup"><span data-stu-id="13792-525">A reverse proxy server receives HTTP requests from the network and forwards them to Kestrel.</span></span>

<span data-ttu-id="13792-526">Kestrel usato come server Web perimetrale (esposto a Internet):</span><span class="sxs-lookup"><span data-stu-id="13792-526">Kestrel used as an edge (Internet-facing) web server:</span></span>

![Kestrel comunica direttamente con Internet senza un server proxy inverso](kestrel/_static/kestrel-to-internet2.png)

<span data-ttu-id="13792-528">Kestrel usato in una configurazione proxy inverso:</span><span class="sxs-lookup"><span data-stu-id="13792-528">Kestrel used in a reverse proxy configuration:</span></span>

![Kestrel comunica indirettamente con Internet attraverso un server proxy inverso, ad esempio IIS, Nginx o Apache](kestrel/_static/kestrel-to-internet.png)

<span data-ttu-id="13792-530">La configurazione, con o senza un server proxy inverso, è una configurazione di hosting supportata.</span><span class="sxs-lookup"><span data-stu-id="13792-530">Either configuration, with or without a reverse proxy server, is a supported hosting configuration.</span></span>

<span data-ttu-id="13792-531">Se viene utilizzato come server perimetrale in assenza di un server proxy inverso, Kestrel non supporta la condivisione tra più processi dell'IP e della porta.</span><span class="sxs-lookup"><span data-stu-id="13792-531">Kestrel used as an edge server without a reverse proxy server doesn't support sharing the same IP and port among multiple processes.</span></span> <span data-ttu-id="13792-532">Quando è configurato per restare in ascolto su una porta, Kestrel gestisce tutto il traffico per tale porta indipendentemente dalle intestazioni `Host` delle richieste.</span><span class="sxs-lookup"><span data-stu-id="13792-532">When Kestrel is configured to listen on a port, Kestrel handles all of the traffic for that port regardless of requests' `Host` headers.</span></span> <span data-ttu-id="13792-533">Un proxy inverso che può condividere le porte può eseguire l'inoltro delle richieste a Kestrel su un unico IP e un'unica porta.</span><span class="sxs-lookup"><span data-stu-id="13792-533">A reverse proxy that can share ports has the ability to forward requests to Kestrel on a unique IP and port.</span></span>

<span data-ttu-id="13792-534">Anche se la presenza di un server proxy inverso non è necessaria, può risultare utile per i motivi seguenti.</span><span class="sxs-lookup"><span data-stu-id="13792-534">Even if a reverse proxy server isn't required, using a reverse proxy server might be a good choice.</span></span>

<span data-ttu-id="13792-535">Un proxy inverso:</span><span class="sxs-lookup"><span data-stu-id="13792-535">A reverse proxy:</span></span>

* <span data-ttu-id="13792-536">Può limitare l'area della superficie di attacco pubblica esposta delle app ospitate.</span><span class="sxs-lookup"><span data-stu-id="13792-536">Can limit the exposed public surface area of the apps that it hosts.</span></span>
* <span data-ttu-id="13792-537">Offre un livello aggiuntivo di configurazione e protezione.</span><span class="sxs-lookup"><span data-stu-id="13792-537">Provide an additional layer of configuration and defense.</span></span>
* <span data-ttu-id="13792-538">Potrebbe offrire un'integrazione migliore con l'infrastruttura esistente.</span><span class="sxs-lookup"><span data-stu-id="13792-538">Might integrate better with existing infrastructure.</span></span>
* <span data-ttu-id="13792-539">Semplifica il bilanciamento del carico e la configurazione di comunicazioni protette (HTTPS).</span><span class="sxs-lookup"><span data-stu-id="13792-539">Simplify load balancing and secure communication (HTTPS) configuration.</span></span> <span data-ttu-id="13792-540">Solo il server proxy inverso richiede un certificato X. 509 e il server è in grado di comunicare con i server dell'app nella rete interna tramite HTTP normale.</span><span class="sxs-lookup"><span data-stu-id="13792-540">Only the reverse proxy server requires an X.509 certificate, and that server can communicate with the app's servers on the internal network using plain HTTP.</span></span>

> [!WARNING]
> <span data-ttu-id="13792-541">Una configurazione che prevede un proxy inverso richiede il [filtro host](#host-filtering).</span><span class="sxs-lookup"><span data-stu-id="13792-541">Hosting in a reverse proxy configuration requires [host filtering](#host-filtering).</span></span>

## <a name="how-to-use-kestrel-in-aspnet-core-apps"></a><span data-ttu-id="13792-542">Come usare Kestrel nelle app ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="13792-542">How to use Kestrel in ASP.NET Core apps</span></span>

<span data-ttu-id="13792-543">Il pacchetto [Microsoft. AspNetCore. Server. gheppio](https://www.nuget.org/packages/Microsoft.AspNetCore.Server.Kestrel/) è incluso nel [metapacchetto Microsoft. AspNetCore. app](xref:fundamentals/metapackage-app).</span><span class="sxs-lookup"><span data-stu-id="13792-543">The [Microsoft.AspNetCore.Server.Kestrel](https://www.nuget.org/packages/Microsoft.AspNetCore.Server.Kestrel/) package is included in the [Microsoft.AspNetCore.App metapackage](xref:fundamentals/metapackage-app).</span></span>

<span data-ttu-id="13792-544">I modelli di progetto ASP.NET Core usano Kestrel per impostazione predefinita.</span><span class="sxs-lookup"><span data-stu-id="13792-544">ASP.NET Core project templates use Kestrel by default.</span></span> <span data-ttu-id="13792-545">In *Program.cs* il codice del modello chiama <xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder*>, che chiama <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderKestrelExtensions.UseKestrel*> in background.</span><span class="sxs-lookup"><span data-stu-id="13792-545">In *Program.cs*, the template code calls <xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder*>, which calls <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderKestrelExtensions.UseKestrel*> behind the scenes.</span></span>

[!code-csharp[](kestrel/samples/2.x/KestrelSample/Program.cs?name=snippet_DefaultBuilder&highlight=7)]

<span data-ttu-id="13792-546">Per ulteriori informazioni su `CreateDefaultBuilder` e sulla compilazione dell'host, vedere la sezione *configurare un host* di <xref:fundamentals/host/web-host#set-up-a-host> .</span><span class="sxs-lookup"><span data-stu-id="13792-546">For more information on `CreateDefaultBuilder` and building the host, see the *Set up a host* section of <xref:fundamentals/host/web-host#set-up-a-host>.</span></span>

<span data-ttu-id="13792-547">Per fornire una configurazione aggiuntiva dopo la chiamata di `CreateDefaultBuilder`, usare `ConfigureKestrel`:</span><span class="sxs-lookup"><span data-stu-id="13792-547">To provide additional configuration after calling `CreateDefaultBuilder`, use `ConfigureKestrel`:</span></span>

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .ConfigureKestrel((context, serverOptions) =>
        {
            // Set properties and call methods on serverOptions
        });
```

<span data-ttu-id="13792-548">Se l'app non chiama `CreateDefaultBuilder` per configurare l'host, chiamare <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderKestrelExtensions.UseKestrel*> **prima** di chiamare `ConfigureKestrel`:</span><span class="sxs-lookup"><span data-stu-id="13792-548">If the app doesn't call `CreateDefaultBuilder` to set up the host, call <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderKestrelExtensions.UseKestrel*> **before** calling `ConfigureKestrel`:</span></span>

```csharp
public static void Main(string[] args)
{
    var host = new WebHostBuilder()
        .UseContentRoot(Directory.GetCurrentDirectory())
        .UseKestrel()
        .UseIISIntegration()
        .UseStartup<Startup>()
        .ConfigureKestrel((context, serverOptions) =>
        {
            // Set properties and call methods on serverOptions
        })
        .Build();

    host.Run();
}
```

## <a name="kestrel-options"></a><span data-ttu-id="13792-549">Opzioni Kestrel</span><span class="sxs-lookup"><span data-stu-id="13792-549">Kestrel options</span></span>

<span data-ttu-id="13792-550">Il server Web Kestrel dispone di opzioni di configurazione dei vincoli che risultano particolarmente utili nelle distribuzioni con connessione Internet.</span><span class="sxs-lookup"><span data-stu-id="13792-550">The Kestrel web server has constraint configuration options that are especially useful in Internet-facing deployments.</span></span>

<span data-ttu-id="13792-551">Impostare i vincoli per la proprietà <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Limits> della classe <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions>.</span><span class="sxs-lookup"><span data-stu-id="13792-551">Set constraints on the <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Limits> property of the <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions> class.</span></span> <span data-ttu-id="13792-552">La proprietà `Limits` contiene un'istanza della classe <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits>.</span><span class="sxs-lookup"><span data-stu-id="13792-552">The `Limits` property holds an instance of the <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits> class.</span></span>

<span data-ttu-id="13792-553">Negli esempi seguenti viene usato lo spazio dei nomi <xref:Microsoft.AspNetCore.Server.Kestrel.Core>:</span><span class="sxs-lookup"><span data-stu-id="13792-553">The following examples use the <xref:Microsoft.AspNetCore.Server.Kestrel.Core> namespace:</span></span>

```csharp
using Microsoft.AspNetCore.Server.Kestrel.Core;
```

<span data-ttu-id="13792-554">Le opzioni gheppio, che sono configurate nel codice C# negli esempi seguenti, possono essere impostate anche usando un [provider di configurazione](xref:fundamentals/configuration/index).</span><span class="sxs-lookup"><span data-stu-id="13792-554">Kestrel options, which are configured in C# code in the following examples, can also be set using a [configuration provider](xref:fundamentals/configuration/index).</span></span> <span data-ttu-id="13792-555">Il provider di configurazione file, ad esempio, può caricare la configurazione di Gheppio da un oggetto *appsettings.json* o *appSettings. { File Environment}. JSON* :</span><span class="sxs-lookup"><span data-stu-id="13792-555">For example, the File Configuration Provider can load Kestrel configuration from an *appsettings.json* or *appsettings.{Environment}.json* file:</span></span>

```json
{
  "Kestrel": {
    "Limits": {
      "MaxConcurrentConnections": 100,
      "MaxConcurrentUpgradedConnections": 100
    }
  }
}
```

<span data-ttu-id="13792-556">Usare **uno** degli approcci seguenti:</span><span class="sxs-lookup"><span data-stu-id="13792-556">Use **one** of the following approaches:</span></span>

* <span data-ttu-id="13792-557">Configurare gheppio in `Startup.ConfigureServices` :</span><span class="sxs-lookup"><span data-stu-id="13792-557">Configure Kestrel in `Startup.ConfigureServices`:</span></span>

  1. <span data-ttu-id="13792-558">Inserire un'istanza di `IConfiguration` nella `Startup` classe.</span><span class="sxs-lookup"><span data-stu-id="13792-558">Inject an instance of `IConfiguration` into the `Startup` class.</span></span> <span data-ttu-id="13792-559">Nell'esempio seguente si presuppone che la configurazione inserita venga assegnata alla `Configuration` Proprietà.</span><span class="sxs-lookup"><span data-stu-id="13792-559">The following example assumes that the injected configuration is assigned to the `Configuration` property.</span></span>
  2. <span data-ttu-id="13792-560">In `Startup.ConfigureServices` caricare la `Kestrel` sezione della configurazione nella configurazione di Gheppio:</span><span class="sxs-lookup"><span data-stu-id="13792-560">In `Startup.ConfigureServices`, load the `Kestrel` section of configuration into Kestrel's configuration:</span></span>

     ```csharp
     using Microsoft.Extensions.Configuration
     
     public class Startup
     {
         public Startup(IConfiguration configuration)
         {
             Configuration = configuration;
         }

         public IConfiguration Configuration { get; }

         public void ConfigureServices(IServiceCollection services)
         {
             services.Configure<KestrelServerOptions>(
                 Configuration.GetSection("Kestrel"));
         }

         public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
         {
             ...
         }
     }
     ```

* <span data-ttu-id="13792-561">Configurare il gheppio durante la compilazione dell'host:</span><span class="sxs-lookup"><span data-stu-id="13792-561">Configure Kestrel when building the host:</span></span>

  <span data-ttu-id="13792-562">In *Program.cs* caricare la `Kestrel` sezione della configurazione nella configurazione di Gheppio:</span><span class="sxs-lookup"><span data-stu-id="13792-562">In *Program.cs*, load the `Kestrel` section of configuration into Kestrel's configuration:</span></span>

  ```csharp
  // using Microsoft.Extensions.DependencyInjection;

  public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
      WebHost.CreateDefaultBuilder(args)
          .ConfigureServices((context, services) =>
          {
              services.Configure<KestrelServerOptions>(
                  context.Configuration.GetSection("Kestrel"));
          })
          .UseStartup<Startup>();
  ```

<span data-ttu-id="13792-563">Entrambi gli approcci precedenti funzionano con qualsiasi [provider di configurazione](xref:fundamentals/configuration/index).</span><span class="sxs-lookup"><span data-stu-id="13792-563">Both of the preceding approaches work with any [configuration provider](xref:fundamentals/configuration/index).</span></span>

### <a name="keep-alive-timeout"></a><span data-ttu-id="13792-564">Timeout keep-alive</span><span class="sxs-lookup"><span data-stu-id="13792-564">Keep-alive timeout</span></span>

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.KeepAliveTimeout>

<span data-ttu-id="13792-565">Ottiene o imposta il [timeout keep-alive](https://tools.ietf.org/html/rfc7230#section-6.5).</span><span class="sxs-lookup"><span data-stu-id="13792-565">Gets or sets the [keep-alive timeout](https://tools.ietf.org/html/rfc7230#section-6.5).</span></span> <span data-ttu-id="13792-566">Il valore predefinito è 2 minuti.</span><span class="sxs-lookup"><span data-stu-id="13792-566">Defaults to 2 minutes.</span></span>

[!code-csharp[](kestrel/samples/2.x/KestrelSample/Program.cs?name=snippet_Limits&highlight=15)]

### <a name="maximum-client-connections"></a><span data-ttu-id="13792-567">Numero massimo di connessioni client</span><span class="sxs-lookup"><span data-stu-id="13792-567">Maximum client connections</span></span>

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.MaxConcurrentConnections>
<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.MaxConcurrentUpgradedConnections>

<span data-ttu-id="13792-568">È possibile impostare il numero massimo di connessioni TCP aperte simultaneamente per l'intera app con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="13792-568">The maximum number of concurrent open TCP connections can be set for the entire app with the following code:</span></span>

[!code-csharp[](kestrel/samples/2.x/KestrelSample/Program.cs?name=snippet_Limits&highlight=3)]

<span data-ttu-id="13792-569">È previsto un limite separato per le connessioni che sono state aggiornate da HTTP o HTTPS a un altro protocollo (ad esempio su una richiesta WebSocket).</span><span class="sxs-lookup"><span data-stu-id="13792-569">There's a separate limit for connections that have been upgraded from HTTP or HTTPS to another protocol (for example, on a WebSockets request).</span></span> <span data-ttu-id="13792-570">Dopo l'aggiornamento di una connessione, questa non viene conteggiata per il limite `MaxConcurrentConnections`.</span><span class="sxs-lookup"><span data-stu-id="13792-570">After a connection is upgraded, it isn't counted against the `MaxConcurrentConnections` limit.</span></span>

[!code-csharp[](kestrel/samples/2.x/KestrelSample/Program.cs?name=snippet_Limits&highlight=4)]

<span data-ttu-id="13792-571">Per impostazione predefinita, il numero massimo di connessioni è illimitato (null).</span><span class="sxs-lookup"><span data-stu-id="13792-571">The maximum number of connections is unlimited (null) by default.</span></span>

### <a name="maximum-request-body-size"></a><span data-ttu-id="13792-572">Dimensione massima del corpo della richiesta</span><span class="sxs-lookup"><span data-stu-id="13792-572">Maximum request body size</span></span>

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.MaxRequestBodySize>

<span data-ttu-id="13792-573">La dimensione massima predefinita del corpo della richiesta è pari a 30.000.000 di byte, equivalenti a circa 28,6 MB.</span><span class="sxs-lookup"><span data-stu-id="13792-573">The default maximum request body size is 30,000,000 bytes, which is approximately 28.6 MB.</span></span>

<span data-ttu-id="13792-574">Il metodo consigliato per ignorare il limite in un'app ASP.NET Core MVC è l'uso dell'attributo <xref:Microsoft.AspNetCore.Mvc.RequestSizeLimitAttribute> in un metodo di azione:</span><span class="sxs-lookup"><span data-stu-id="13792-574">The recommended approach to override the limit in an ASP.NET Core MVC app is to use the <xref:Microsoft.AspNetCore.Mvc.RequestSizeLimitAttribute> attribute on an action method:</span></span>

```csharp
[RequestSizeLimit(100000000)]
public IActionResult MyActionMethod()
```

<span data-ttu-id="13792-575">L'esempio seguente illustra come configurare il vincolo per l'app in ogni richiesta:</span><span class="sxs-lookup"><span data-stu-id="13792-575">Here's an example that shows how to configure the constraint for the app on every request:</span></span>

[!code-csharp[](kestrel/samples/2.x/KestrelSample/Program.cs?name=snippet_Limits&highlight=5)]

<span data-ttu-id="13792-576">Eseguire l'override dell'impostazione per una richiesta specifica nel middleware:</span><span class="sxs-lookup"><span data-stu-id="13792-576">Override the setting on a specific request in middleware:</span></span>

[!code-csharp[](kestrel/samples/2.x/KestrelSample/Startup.cs?name=snippet_Limits&highlight=3-4)]

<span data-ttu-id="13792-577">Viene generata un'eccezione se l'app configura il limite per una richiesta dopo che l'app ha iniziato a leggere la richiesta.</span><span class="sxs-lookup"><span data-stu-id="13792-577">An exception is thrown if the app configures the limit on a request after the app has started to read the request.</span></span> <span data-ttu-id="13792-578">Una proprietà `IsReadOnly` indica se la proprietà `MaxRequestBodySize` è in stato di sola lettura e pertanto è troppo tardi per configurare il limite.</span><span class="sxs-lookup"><span data-stu-id="13792-578">There's an `IsReadOnly` property that indicates if the `MaxRequestBodySize` property is in read-only state, meaning it's too late to configure the limit.</span></span>

<span data-ttu-id="13792-579">Quando un'app viene eseguita [out-of-process](xref:host-and-deploy/iis/index#out-of-process-hosting-model) dietro il [modulo ASP.NET Core](xref:host-and-deploy/aspnet-core-module), il limite della dimensione del corpo della richiesta di Kestrel è disabilitato perché viene già impostato da IIS.</span><span class="sxs-lookup"><span data-stu-id="13792-579">When an app is run [out-of-process](xref:host-and-deploy/iis/index#out-of-process-hosting-model) behind the [ASP.NET Core Module](xref:host-and-deploy/aspnet-core-module), Kestrel's request body size limit is disabled because IIS already sets the limit.</span></span>

### <a name="minimum-request-body-data-rate"></a><span data-ttu-id="13792-580">Velocità minima dei dati del corpo della richiesta</span><span class="sxs-lookup"><span data-stu-id="13792-580">Minimum request body data rate</span></span>

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.MinRequestBodyDataRate>
<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.MinResponseDataRate>

<span data-ttu-id="13792-581">Kestrel controlla ogni secondo se i dati arrivano alla velocità in byte al secondo specificata.</span><span class="sxs-lookup"><span data-stu-id="13792-581">Kestrel checks every second if data is arriving at the specified rate in bytes/second.</span></span> <span data-ttu-id="13792-582">Se la frequenza scende sotto il valore minimo, si è verificato il timeout della connessione. Il periodo di tolleranza è la quantità di tempo che il gheppio concede al client di aumentare la velocità di invio fino al minimo. la frequenza non viene controllata durante tale periodo di tempo.</span><span class="sxs-lookup"><span data-stu-id="13792-582">If the rate drops below the minimum, the connection is timed out. The grace period is the amount of time that Kestrel gives the client to increase its send rate up to the minimum; the rate isn't checked during that time.</span></span> <span data-ttu-id="13792-583">Il periodo di prova consente di evitare l'interruzione di connessioni che inizialmente inviano i dati a velocità ridotta a causa dell'avvio lento del protocollo TCP.</span><span class="sxs-lookup"><span data-stu-id="13792-583">The grace period helps avoid dropping connections that are initially sending data at a slow rate due to TCP slow-start.</span></span>

<span data-ttu-id="13792-584">La velocità minima predefinita è di 240 byte al secondo, con un periodo di tolleranza di 5 secondi.</span><span class="sxs-lookup"><span data-stu-id="13792-584">The default minimum rate is 240 bytes/second with a 5 second grace period.</span></span>

<span data-ttu-id="13792-585">Anche per la risposta è prevista una velocità minima.</span><span class="sxs-lookup"><span data-stu-id="13792-585">A minimum rate also applies to the response.</span></span> <span data-ttu-id="13792-586">Il codice per impostare il limite della richiesta e della risposta è identico e varia solo per `RequestBody` o `Response` nei nomi di proprietà e di interfaccia.</span><span class="sxs-lookup"><span data-stu-id="13792-586">The code to set the request limit and the response limit is the same except for having `RequestBody` or `Response` in the property and interface names.</span></span>

<span data-ttu-id="13792-587">L'esempio seguente visualizza come configurare la velocità minima dei dati in *Program.cs*:</span><span class="sxs-lookup"><span data-stu-id="13792-587">Here's an example that shows how to configure the minimum data rates in *Program.cs*:</span></span>

[!code-csharp[](kestrel/samples/2.x/KestrelSample/Program.cs?name=snippet_Limits&highlight=6-9)]

<span data-ttu-id="13792-588">Ignorare i limiti di velocità minima per ogni richiesta nel middleware:</span><span class="sxs-lookup"><span data-stu-id="13792-588">Override the minimum rate limits per request in middleware:</span></span>

[!code-csharp[](kestrel/samples/2.x/KestrelSample/Startup.cs?name=snippet_Limits&highlight=6-21)]

<span data-ttu-id="13792-589">Nessuna delle due funzionalità relative alla velocità, a cui si fa riferimento nell'esempio precedente, è presente in `HttpContext.Features` per le richieste HTTP/2 perché la modifica dei limiti di velocità per ogni richiesta non è supportata per HTTP/2 a causa del supporto del protocollo per il multiplexing delle richieste.</span><span class="sxs-lookup"><span data-stu-id="13792-589">Neither rate feature referenced in the prior sample are present in `HttpContext.Features` for HTTP/2 requests because modifying rate limits on a per-request basis isn't supported for HTTP/2 due to the protocol's support for request multiplexing.</span></span> <span data-ttu-id="13792-590">I limiti di velocità a livello di server configurati tramite `KestrelServerOptions.Limits` vengono tuttavia applicati alle connessioni HTTP/1.x e HTTP/2.</span><span class="sxs-lookup"><span data-stu-id="13792-590">Server-wide rate limits configured via `KestrelServerOptions.Limits` still apply to both HTTP/1.x and HTTP/2 connections.</span></span>

### <a name="request-headers-timeout"></a><span data-ttu-id="13792-591">Timeout delle intestazioni delle richieste</span><span class="sxs-lookup"><span data-stu-id="13792-591">Request headers timeout</span></span>

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.RequestHeadersTimeout>

<span data-ttu-id="13792-592">Ottiene o imposta la quantità massima di tempo che il server dedica alla ricezione delle intestazioni delle richieste.</span><span class="sxs-lookup"><span data-stu-id="13792-592">Gets or sets the maximum amount of time the server spends receiving request headers.</span></span> <span data-ttu-id="13792-593">Il valore predefinito è 30 secondi.</span><span class="sxs-lookup"><span data-stu-id="13792-593">Defaults to 30 seconds.</span></span>

[!code-csharp[](kestrel/samples/2.x/KestrelSample/Program.cs?name=snippet_Limits&highlight=16)]

### <a name="maximum-streams-per-connection"></a><span data-ttu-id="13792-594">Numero massimo di flussi per connessione</span><span class="sxs-lookup"><span data-stu-id="13792-594">Maximum streams per connection</span></span>

<span data-ttu-id="13792-595">`Http2.MaxStreamsPerConnection` limita il numero di flussi di richieste simultanee per ogni connessione HTTP/2.</span><span class="sxs-lookup"><span data-stu-id="13792-595">`Http2.MaxStreamsPerConnection` limits the number of concurrent request streams per HTTP/2 connection.</span></span> <span data-ttu-id="13792-596">I flussi in eccesso vengono rifiutati.</span><span class="sxs-lookup"><span data-stu-id="13792-596">Excess streams are refused.</span></span>

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .ConfigureKestrel((context, serverOptions) =>
        {
            serverOptions.Limits.Http2.MaxStreamsPerConnection = 100;
        });
```

<span data-ttu-id="13792-597">Il valore predefinito è 100.</span><span class="sxs-lookup"><span data-stu-id="13792-597">The default value is 100.</span></span>

### <a name="header-table-size"></a><span data-ttu-id="13792-598">Dimensioni della tabella delle intestazioni</span><span class="sxs-lookup"><span data-stu-id="13792-598">Header table size</span></span>

<span data-ttu-id="13792-599">Il decodificatore HPACK decomprime le intestazioni HTTP per le connessioni HTTP/2.</span><span class="sxs-lookup"><span data-stu-id="13792-599">The HPACK decoder decompresses HTTP headers for HTTP/2 connections.</span></span> <span data-ttu-id="13792-600">`Http2.HeaderTableSize` limita le dimensioni della tabella di compressione delle intestazioni usata dal decodificatore HPACK.</span><span class="sxs-lookup"><span data-stu-id="13792-600">`Http2.HeaderTableSize` limits the size of the header compression table that the HPACK decoder uses.</span></span> <span data-ttu-id="13792-601">Il valore viene specificato in ottetti e deve essere maggiore di zero (0).</span><span class="sxs-lookup"><span data-stu-id="13792-601">The value is provided in octets and must be greater than zero (0).</span></span>

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .ConfigureKestrel((context, serverOptions) =>
        {
            serverOptions.Limits.Http2.HeaderTableSize = 4096;
        });
```

<span data-ttu-id="13792-602">Il valore predefinito è 4096.</span><span class="sxs-lookup"><span data-stu-id="13792-602">The default value is 4096.</span></span>

### <a name="maximum-frame-size"></a><span data-ttu-id="13792-603">Dimensione massima del frame</span><span class="sxs-lookup"><span data-stu-id="13792-603">Maximum frame size</span></span>

<span data-ttu-id="13792-604">`Http2.MaxFrameSize` indica le dimensioni massime del payload del frame di connessione HTTP/2 da ricevere.</span><span class="sxs-lookup"><span data-stu-id="13792-604">`Http2.MaxFrameSize` indicates the maximum size of the HTTP/2 connection frame payload to receive.</span></span> <span data-ttu-id="13792-605">Il valore viene specificato in ottetti e deve essere compreso tra 2^14 (16.384) e 2^24-1 (16.777.215).</span><span class="sxs-lookup"><span data-stu-id="13792-605">The value is provided in octets and must be between 2^14 (16,384) and 2^24-1 (16,777,215).</span></span>

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .ConfigureKestrel((context, serverOptions) =>
        {
            serverOptions.Limits.Http2.MaxFrameSize = 16384;
        });
```

<span data-ttu-id="13792-606">Il valore predefinito è 2^14 (16.384).</span><span class="sxs-lookup"><span data-stu-id="13792-606">The default value is 2^14 (16,384).</span></span>

### <a name="maximum-request-header-size"></a><span data-ttu-id="13792-607">Dimensioni massime dell'intestazione della richiesta</span><span class="sxs-lookup"><span data-stu-id="13792-607">Maximum request header size</span></span>

<span data-ttu-id="13792-608">`Http2.MaxRequestHeaderFieldSize` indica le dimensioni massime consentite in ottetti di valori di intestazione di richiesta.</span><span class="sxs-lookup"><span data-stu-id="13792-608">`Http2.MaxRequestHeaderFieldSize` indicates the maximum allowed size in octets of request header values.</span></span> <span data-ttu-id="13792-609">Questo limite si applica insieme sia al nome che al valore nelle rappresentazioni compresse e non compresse.</span><span class="sxs-lookup"><span data-stu-id="13792-609">This limit applies to both name and value together in their compressed and uncompressed representations.</span></span> <span data-ttu-id="13792-610">Il valore deve essere maggiore di zero (0).</span><span class="sxs-lookup"><span data-stu-id="13792-610">The value must be greater than zero (0).</span></span>

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .ConfigureKestrel((context, serverOptions) =>
        {
            serverOptions.Limits.Http2.MaxRequestHeaderFieldSize = 8192;
        });
```

<span data-ttu-id="13792-611">Il valore predefinito è 8.192.</span><span class="sxs-lookup"><span data-stu-id="13792-611">The default value is 8,192.</span></span>

### <a name="initial-connection-window-size"></a><span data-ttu-id="13792-612">Dimensioni della finestra di connessione iniziali</span><span class="sxs-lookup"><span data-stu-id="13792-612">Initial connection window size</span></span>

<span data-ttu-id="13792-613">`Http2.InitialConnectionWindowSize` indica il numero massimo di byte di dati del corpo della richiesta che il server memorizza nel buffer in una volta, aggregati in tutte le richieste (flussi), per ogni connessione.</span><span class="sxs-lookup"><span data-stu-id="13792-613">`Http2.InitialConnectionWindowSize` indicates the maximum request body data in bytes the server buffers at one time aggregated across all requests (streams) per connection.</span></span> <span data-ttu-id="13792-614">Le richieste sono anche limitate da `Http2.InitialStreamWindowSize`.</span><span class="sxs-lookup"><span data-stu-id="13792-614">Requests are also limited by `Http2.InitialStreamWindowSize`.</span></span> <span data-ttu-id="13792-615">Il valore deve essere maggiore di o uguale a 65.535 e minore di 2^31 (2.147.483.648).</span><span class="sxs-lookup"><span data-stu-id="13792-615">The value must be greater than or equal to 65,535 and less than 2^31 (2,147,483,648).</span></span>

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .ConfigureKestrel((context, serverOptions) =>
        {
            serverOptions.Limits.Http2.InitialConnectionWindowSize = 131072;
        });
```

<span data-ttu-id="13792-616">Il valore predefinito è 128 KB (131.072).</span><span class="sxs-lookup"><span data-stu-id="13792-616">The default value is 128 KB (131,072).</span></span>

### <a name="initial-stream-window-size"></a><span data-ttu-id="13792-617">Dimensioni della finestra di flusso iniziali</span><span class="sxs-lookup"><span data-stu-id="13792-617">Initial stream window size</span></span>

<span data-ttu-id="13792-618">`Http2.InitialStreamWindowSize` indica il numero massimo di byte di dati del corpo della richiesta che il server memorizza nel buffer in una volta per ogni richiesta (flusso).</span><span class="sxs-lookup"><span data-stu-id="13792-618">`Http2.InitialStreamWindowSize` indicates the maximum request body data in bytes the server buffers at one time per request (stream).</span></span> <span data-ttu-id="13792-619">Le richieste sono anche limitate da `Http2.InitialStreamWindowSize`.</span><span class="sxs-lookup"><span data-stu-id="13792-619">Requests are also limited by `Http2.InitialStreamWindowSize`.</span></span> <span data-ttu-id="13792-620">Il valore deve essere maggiore di o uguale a 65.535 e minore di 2^31 (2.147.483.648).</span><span class="sxs-lookup"><span data-stu-id="13792-620">The value must be greater than or equal to 65,535 and less than 2^31 (2,147,483,648).</span></span>

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .ConfigureKestrel((context, serverOptions) =>
        {
            serverOptions.Limits.Http2.InitialStreamWindowSize = 98304;
        });
```

<span data-ttu-id="13792-621">Il valore predefinito è 96 KB (98.304).</span><span class="sxs-lookup"><span data-stu-id="13792-621">The default value is 96 KB (98,304).</span></span>

### <a name="synchronous-io"></a><span data-ttu-id="13792-622">I/O sincrono</span><span class="sxs-lookup"><span data-stu-id="13792-622">Synchronous I/O</span></span>

<span data-ttu-id="13792-623"><xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.AllowSynchronousIO> Controlla se è consentito l'I/O sincrono per la richiesta e la risposta.</span><span class="sxs-lookup"><span data-stu-id="13792-623"><xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.AllowSynchronousIO> controls whether synchronous I/O is allowed for the request and response.</span></span> <span data-ttu-id="13792-624">Il valore predefinito è `true`.</span><span class="sxs-lookup"><span data-stu-id="13792-624">The  default value is `true`.</span></span>

> [!WARNING]
> <span data-ttu-id="13792-625">Un numero elevato di operazioni di I/O sincrone bloccanti può causare l'esaurimento del pool di thread, il che rende l'app non risponde.</span><span class="sxs-lookup"><span data-stu-id="13792-625">A large number of blocking synchronous I/O operations can lead to thread pool starvation, which makes the app unresponsive.</span></span> <span data-ttu-id="13792-626">Abilitare solo `AllowSynchronousIO` quando si usa una libreria che non supporta l'I/O asincrono.</span><span class="sxs-lookup"><span data-stu-id="13792-626">Only enable `AllowSynchronousIO` when using a library that doesn't support asynchronous I/O.</span></span>

<span data-ttu-id="13792-627">L'esempio seguente abilita l'I/O sincrono:</span><span class="sxs-lookup"><span data-stu-id="13792-627">The following example enables synchronous I/O:</span></span>

[!code-csharp[](kestrel/samples/2.x/KestrelSample/Program.cs?name=snippet_SyncIO)]

<span data-ttu-id="13792-628">Per informazioni su altre opzioni e limiti di Kestrel, vedere:</span><span class="sxs-lookup"><span data-stu-id="13792-628">For information about other Kestrel options and limits, see:</span></span>

* <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions>
* <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits>
* <xref:Microsoft.AspNetCore.Server.Kestrel.Core.ListenOptions>

## <a name="endpoint-configuration"></a><span data-ttu-id="13792-629">Configurazione dell'endpoint</span><span class="sxs-lookup"><span data-stu-id="13792-629">Endpoint configuration</span></span>

<span data-ttu-id="13792-630">Per impostazione predefinita, ASP.NET Core è associato a:</span><span class="sxs-lookup"><span data-stu-id="13792-630">By default, ASP.NET Core binds to:</span></span>

* `http://localhost:5000`
* <span data-ttu-id="13792-631">`https://localhost:5001` (quando è presente un certificato di sviluppo locale)</span><span class="sxs-lookup"><span data-stu-id="13792-631">`https://localhost:5001` (when a local development certificate is present)</span></span>

<span data-ttu-id="13792-632">Specificare gli URL usando gli elementi seguenti:</span><span class="sxs-lookup"><span data-stu-id="13792-632">Specify URLs using the:</span></span>

* <span data-ttu-id="13792-633">La variabile di ambiente `ASPNETCORE_URLS`.</span><span class="sxs-lookup"><span data-stu-id="13792-633">`ASPNETCORE_URLS` environment variable.</span></span>
* <span data-ttu-id="13792-634">L'argomento della riga di comando `--urls`.</span><span class="sxs-lookup"><span data-stu-id="13792-634">`--urls` command-line argument.</span></span>
* <span data-ttu-id="13792-635">La chiave di configurazione dell'host `urls`.</span><span class="sxs-lookup"><span data-stu-id="13792-635">`urls` host configuration key.</span></span>
* <span data-ttu-id="13792-636">Il metodo di estensione `UseUrls`.</span><span class="sxs-lookup"><span data-stu-id="13792-636">`UseUrls` extension method.</span></span>

<span data-ttu-id="13792-637">Il valore specificato usando i metodi seguenti può essere uno o più endpoint HTTP e HTTPS (HTTPS se è disponibile un certificato predefinito).</span><span class="sxs-lookup"><span data-stu-id="13792-637">The value provided using these approaches can be one or more HTTP and HTTPS endpoints (HTTPS if a default cert is available).</span></span> <span data-ttu-id="13792-638">Configurare il valore come un elenco delimitato da punto e virgola (ad esempio, `"Urls": "http://localhost:8000;http://localhost:8001"`).</span><span class="sxs-lookup"><span data-stu-id="13792-638">Configure the value as a semicolon-separated list (for example, `"Urls": "http://localhost:8000;http://localhost:8001"`).</span></span>

<span data-ttu-id="13792-639">Per altre informazioni su questi approcci, vedere [URL del server](xref:fundamentals/host/web-host#server-urls) e [Override della configurazione](xref:fundamentals/host/web-host#override-configuration).</span><span class="sxs-lookup"><span data-stu-id="13792-639">For more information on these approaches, see [Server URLs](xref:fundamentals/host/web-host#server-urls) and [Override configuration](xref:fundamentals/host/web-host#override-configuration).</span></span>

<span data-ttu-id="13792-640">Viene creato un certificato di sviluppo nei casi seguenti:</span><span class="sxs-lookup"><span data-stu-id="13792-640">A development certificate is created:</span></span>

* <span data-ttu-id="13792-641">Quando viene installato [.NET Core SDK](/dotnet/core/sdk).</span><span class="sxs-lookup"><span data-stu-id="13792-641">When the [.NET Core SDK](/dotnet/core/sdk) is installed.</span></span>
* <span data-ttu-id="13792-642">Quando per creare un certificato viene usato [lo strumento dev-certs](xref:aspnetcore-2.1#https).</span><span class="sxs-lookup"><span data-stu-id="13792-642">The [dev-certs tool](xref:aspnetcore-2.1#https) is used to create a certificate.</span></span>

<span data-ttu-id="13792-643">Alcuni browser richiedono la concessione di autorizzazioni esplicite per considerare attendibile il certificato di sviluppo locale.</span><span class="sxs-lookup"><span data-stu-id="13792-643">Some browsers require granting explicit permission to trust the local development certificate.</span></span>

<span data-ttu-id="13792-644">I modelli di progetto consentono di configurare le app per l'esecuzione su HTTPS per impostazione predefinita e includono il [reindirizzamento HTTPS e il supporto HSTS](xref:security/enforcing-ssl).</span><span class="sxs-lookup"><span data-stu-id="13792-644">Project templates configure apps to run on HTTPS by default and include [HTTPS redirection and HSTS support](xref:security/enforcing-ssl).</span></span>

<span data-ttu-id="13792-645">Chiamare i metodi <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Listen*> oppure <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ListenUnixSocket*> su <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions> per configurare le porte e i prefissi URL per Kestrel.</span><span class="sxs-lookup"><span data-stu-id="13792-645">Call <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Listen*> or <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ListenUnixSocket*> methods on <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions> to configure URL prefixes and ports for Kestrel.</span></span>

<span data-ttu-id="13792-646">Anche `UseUrls`, l'argomento della riga di comando `--urls`, la chiave di configurazione dell'host `urls` e la variabile di ambiente `ASPNETCORE_URLS` funzionano, ma con le limitazioni indicate più avanti nella sezione (deve essere disponibile un certificato predefinito per la configurazione dell'endopoint HTTPS).</span><span class="sxs-lookup"><span data-stu-id="13792-646">`UseUrls`, the `--urls` command-line argument, `urls` host configuration key, and the `ASPNETCORE_URLS` environment variable also work but have the limitations noted later in this section (a default certificate must be available for HTTPS endpoint configuration).</span></span>

<span data-ttu-id="13792-647">`KestrelServerOptions` configurazione</span><span class="sxs-lookup"><span data-stu-id="13792-647">`KestrelServerOptions` configuration:</span></span>

### <a name="configureendpointdefaultsactionlistenoptions"></a><span data-ttu-id="13792-648">ConfigureEndpointDefaults (azione \<ListenOptions> )</span><span class="sxs-lookup"><span data-stu-id="13792-648">ConfigureEndpointDefaults(Action\<ListenOptions>)</span></span>

<span data-ttu-id="13792-649">Specifica un elemento di configurazione `Action` da eseguire per ogni endpoint specificato.</span><span class="sxs-lookup"><span data-stu-id="13792-649">Specifies a configuration `Action` to run for each specified endpoint.</span></span> <span data-ttu-id="13792-650">Se si chiama `ConfigureEndpointDefaults` più volte, gli elementi `Action` precedenti vengono sostituiti con l'ultimo elemento `Action` specificato.</span><span class="sxs-lookup"><span data-stu-id="13792-650">Calling `ConfigureEndpointDefaults` multiple times replaces prior `Action`s with the last `Action` specified.</span></span>

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .ConfigureKestrel((context, serverOptions) =>
        {
            serverOptions.ConfigureEndpointDefaults(listenOptions =>
            {
                // Configure endpoint defaults
            });
        });
```

> [!NOTE]
> <span data-ttu-id="13792-651">Per gli endpoint creati chiamando <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Listen*> **prima** della chiamata a <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ConfigureEndpointDefaults*> non verranno applicati i valori predefiniti.</span><span class="sxs-lookup"><span data-stu-id="13792-651">Endpoints created by calling <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Listen*> **before** calling <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ConfigureEndpointDefaults*> won't have the defaults applied.</span></span>

### <a name="configurehttpsdefaultsactionhttpsconnectionadapteroptions"></a><span data-ttu-id="13792-652">ConfigureHttpsDefaults (azione \<HttpsConnectionAdapterOptions> )</span><span class="sxs-lookup"><span data-stu-id="13792-652">ConfigureHttpsDefaults(Action\<HttpsConnectionAdapterOptions>)</span></span>

<span data-ttu-id="13792-653">Specifica un elemento di configurazione `Action` da eseguire per ogni endpoint HTTPS.</span><span class="sxs-lookup"><span data-stu-id="13792-653">Specifies a configuration `Action` to run for each HTTPS endpoint.</span></span> <span data-ttu-id="13792-654">Se si chiama `ConfigureHttpsDefaults` più volte, gli elementi `Action` precedenti vengono sostituiti con l'ultimo elemento `Action` specificato.</span><span class="sxs-lookup"><span data-stu-id="13792-654">Calling `ConfigureHttpsDefaults` multiple times replaces prior `Action`s with the last `Action` specified.</span></span>

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .ConfigureKestrel((context, serverOptions) =>
        {
            serverOptions.ConfigureHttpsDefaults(listenOptions =>
            {
                // certificate is an X509Certificate2
                listenOptions.ServerCertificate = certificate;
            });
        });
```

> [!NOTE]
> <span data-ttu-id="13792-655">Per gli endpoint creati chiamando <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Listen*> **prima** della chiamata a <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ConfigureHttpsDefaults*> non verranno applicati i valori predefiniti.</span><span class="sxs-lookup"><span data-stu-id="13792-655">Endpoints created by calling <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Listen*> **before** calling <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ConfigureHttpsDefaults*> won't have the defaults applied.</span></span>


### <a name="configureiconfiguration"></a><span data-ttu-id="13792-656">Configure(IConfiguration)</span><span class="sxs-lookup"><span data-stu-id="13792-656">Configure(IConfiguration)</span></span>

<span data-ttu-id="13792-657">Crea un loader di configurazione per la configurazione di Kestrel che accetta un elemento <xref:Microsoft.Extensions.Configuration.IConfiguration> come input.</span><span class="sxs-lookup"><span data-stu-id="13792-657">Creates a configuration loader for setting up Kestrel that takes an <xref:Microsoft.Extensions.Configuration.IConfiguration> as input.</span></span> <span data-ttu-id="13792-658">L'ambito della configurazione deve essere la sezione di configurazione per Kestrel.</span><span class="sxs-lookup"><span data-stu-id="13792-658">The configuration must be scoped to the configuration section for Kestrel.</span></span>

### <a name="listenoptionsusehttps"></a><span data-ttu-id="13792-659">ListenOptions.UseHttps</span><span class="sxs-lookup"><span data-stu-id="13792-659">ListenOptions.UseHttps</span></span>

<span data-ttu-id="13792-660">Configura Kestrel per l'uso di HTTPS.</span><span class="sxs-lookup"><span data-stu-id="13792-660">Configure Kestrel to use HTTPS.</span></span>

<span data-ttu-id="13792-661">Estensioni `ListenOptions.UseHttps`:</span><span class="sxs-lookup"><span data-stu-id="13792-661">`ListenOptions.UseHttps` extensions:</span></span>

* <span data-ttu-id="13792-662">`UseHttps`: Configurare gheppio per l'uso di HTTPS con il certificato predefinito.</span><span class="sxs-lookup"><span data-stu-id="13792-662">`UseHttps`: Configure Kestrel to use HTTPS with the default certificate.</span></span> <span data-ttu-id="13792-663">Genera un'eccezione se non è stato configurato alcun certificato predefinito.</span><span class="sxs-lookup"><span data-stu-id="13792-663">Throws an exception if no default certificate is configured.</span></span>
* `UseHttps(string fileName)`
* `UseHttps(string fileName, string password)`
* `UseHttps(string fileName, string password, Action<HttpsConnectionAdapterOptions> configureOptions)`
* `UseHttps(StoreName storeName, string subject)`
* `UseHttps(StoreName storeName, string subject, bool allowInvalid)`
* `UseHttps(StoreName storeName, string subject, bool allowInvalid, StoreLocation location)`
* `UseHttps(StoreName storeName, string subject, bool allowInvalid, StoreLocation location, Action<HttpsConnectionAdapterOptions> configureOptions)`
* `UseHttps(X509Certificate2 serverCertificate)`
* `UseHttps(X509Certificate2 serverCertificate, Action<HttpsConnectionAdapterOptions> configureOptions)`
* `UseHttps(Action<HttpsConnectionAdapterOptions> configureOptions)`

<span data-ttu-id="13792-664">Parametri `ListenOptions.UseHttps`:</span><span class="sxs-lookup"><span data-stu-id="13792-664">`ListenOptions.UseHttps` parameters:</span></span>

* <span data-ttu-id="13792-665">`filename` è il percorso e il nome file di un file di certificato, relativo alla directory che contiene i file di contenuto dell'app.</span><span class="sxs-lookup"><span data-stu-id="13792-665">`filename` is the path and file name of a certificate file, relative to the directory that contains the app's content files.</span></span>
* <span data-ttu-id="13792-666">`password` è la password richiesta per accedere ai dati del certificato X.509.</span><span class="sxs-lookup"><span data-stu-id="13792-666">`password` is the password required to access the X.509 certificate data.</span></span>
* <span data-ttu-id="13792-667">`configureOptions` è un elemento `Action` per configurare `HttpsConnectionAdapterOptions`.</span><span class="sxs-lookup"><span data-stu-id="13792-667">`configureOptions` is an `Action` to configure the `HttpsConnectionAdapterOptions`.</span></span> <span data-ttu-id="13792-668">Restituisce `ListenOptions`.</span><span class="sxs-lookup"><span data-stu-id="13792-668">Returns the `ListenOptions`.</span></span>
* <span data-ttu-id="13792-669">`storeName` è l'archivio certificati da cui caricare il certificato.</span><span class="sxs-lookup"><span data-stu-id="13792-669">`storeName` is the certificate store from which to load the certificate.</span></span>
* <span data-ttu-id="13792-670">`subject` è il nome dell'oggetto del certificato.</span><span class="sxs-lookup"><span data-stu-id="13792-670">`subject` is the subject name for the certificate.</span></span>
* <span data-ttu-id="13792-671">`allowInvalid` indica se devono essere considerati i certificati non validi, come ad esempio i certificati autofirmati.</span><span class="sxs-lookup"><span data-stu-id="13792-671">`allowInvalid` indicates if invalid certificates should be considered, such as self-signed certificates.</span></span>
* <span data-ttu-id="13792-672">`location` è il percorso dell'archivio da cui caricare il certificato.</span><span class="sxs-lookup"><span data-stu-id="13792-672">`location` is the store location to load the certificate from.</span></span>
* <span data-ttu-id="13792-673">`serverCertificate` è il certificato X.509.</span><span class="sxs-lookup"><span data-stu-id="13792-673">`serverCertificate` is the X.509 certificate.</span></span>

<span data-ttu-id="13792-674">Nell'ambiente di produzione, HTTPS deve essere configurato in modo esplicito.</span><span class="sxs-lookup"><span data-stu-id="13792-674">In production, HTTPS must be explicitly configured.</span></span> <span data-ttu-id="13792-675">Come minimo, è necessario specificare un certificato predefinito.</span><span class="sxs-lookup"><span data-stu-id="13792-675">At a minimum, a default certificate must be provided.</span></span>

<span data-ttu-id="13792-676">Configurazioni supportate descritte in seguito:</span><span class="sxs-lookup"><span data-stu-id="13792-676">Supported configurations described next:</span></span>

* <span data-ttu-id="13792-677">Nessuna configurazione</span><span class="sxs-lookup"><span data-stu-id="13792-677">No configuration</span></span>
* <span data-ttu-id="13792-678">Sostituire il certificato predefinito della configurazione</span><span class="sxs-lookup"><span data-stu-id="13792-678">Replace the default certificate from configuration</span></span>
* <span data-ttu-id="13792-679">Modificare le impostazioni predefinite nel codice</span><span class="sxs-lookup"><span data-stu-id="13792-679">Change the defaults in code</span></span>

<span data-ttu-id="13792-680">*Nessuna configurazione*</span><span class="sxs-lookup"><span data-stu-id="13792-680">*No configuration*</span></span>

<span data-ttu-id="13792-681">Kestrel è in ascolto su `http://localhost:5000` e `https://localhost:5001` (se è disponibile un certificato predefinito).</span><span class="sxs-lookup"><span data-stu-id="13792-681">Kestrel listens on `http://localhost:5000` and `https://localhost:5001` (if a default cert is available).</span></span>

<a name="configuration"></a>

<span data-ttu-id="13792-682">*Sostituire il certificato predefinito della configurazione*</span><span class="sxs-lookup"><span data-stu-id="13792-682">*Replace the default certificate from configuration*</span></span>

<span data-ttu-id="13792-683">`CreateDefaultBuilder` chiama `Configure(context.Configuration.GetSection("Kestrel"))` per impostazione predefinita per caricare la configurazione di Kestrel.</span><span class="sxs-lookup"><span data-stu-id="13792-683">`CreateDefaultBuilder` calls `Configure(context.Configuration.GetSection("Kestrel"))` by default to load Kestrel configuration.</span></span> <span data-ttu-id="13792-684">È disponibile per Kestrel uno schema di configurazione delle impostazioni delle app HTTPS predefinito.</span><span class="sxs-lookup"><span data-stu-id="13792-684">A default HTTPS app settings configuration schema is available for Kestrel.</span></span> <span data-ttu-id="13792-685">Configurare più endpoint, inclusi gli URL e i certificati da usare, da un file su disco o da un archivio certificati.</span><span class="sxs-lookup"><span data-stu-id="13792-685">Configure multiple endpoints, including the URLs and the certificates to use, either from a file on disk or from a certificate store.</span></span>

<span data-ttu-id="13792-686">Nell'esempio seguente *appsettings.json* :</span><span class="sxs-lookup"><span data-stu-id="13792-686">In the following *appsettings.json* example:</span></span>

* <span data-ttu-id="13792-687">Impostare **AllowInvalid** su `true` per consentire l'uso di certificati non validi, come ad esempio i certificati autofirmati.</span><span class="sxs-lookup"><span data-stu-id="13792-687">Set **AllowInvalid** to `true` to permit the use of invalid certificates (for example, self-signed certificates).</span></span>
* <span data-ttu-id="13792-688">Tutti gli endpoint HTTPS che non specificano un certificato (**HttpsDefaultCert** nell'esempio che segue) usano il certificato definito in **Certificates** > **Default** o il certificato di sviluppo.</span><span class="sxs-lookup"><span data-stu-id="13792-688">Any HTTPS endpoint that doesn't specify a certificate (**HttpsDefaultCert** in the example that follows) falls back to the cert defined under **Certificates** > **Default** or the development certificate.</span></span>

```json
{
  "Kestrel": {
    "Endpoints": {
      "Http": {
        "Url": "http://localhost:5000"
      },

      "HttpsInlineCertFile": {
        "Url": "https://localhost:5001",
        "Certificate": {
          "Path": "<path to .pfx file>",
          "Password": "<certificate password>"
        }
      },

      "HttpsInlineCertStore": {
        "Url": "https://localhost:5002",
        "Certificate": {
          "Subject": "<subject; required>",
          "Store": "<certificate store; required>",
          "Location": "<location; defaults to CurrentUser>",
          "AllowInvalid": "<true or false; defaults to false>"
        }
      },

      "HttpsDefaultCert": {
        "Url": "https://localhost:5003"
      },

      "Https": {
        "Url": "https://*:5004",
        "Certificate": {
          "Path": "<path to .pfx file>",
          "Password": "<certificate password>"
        }
      }
    },
    "Certificates": {
      "Default": {
        "Path": "<path to .pfx file>",
        "Password": "<certificate password>"
      }
    }
  }
}
```

<span data-ttu-id="13792-689">Un'alternativa all'uso di **Path** e **Password** per qualsiasi nodo del certificato consiste nello specificare il certificato usando i campi dell'archivio certificati.</span><span class="sxs-lookup"><span data-stu-id="13792-689">An alternative to using **Path** and **Password** for any certificate node is to specify the certificate using certificate store fields.</span></span> <span data-ttu-id="13792-690">Ad esempio, il certificato predefinito **Certificates**  >   può essere specificato come:</span><span class="sxs-lookup"><span data-stu-id="13792-690">For example, the **Certificates** > **Default** certificate can be specified as:</span></span>

```json
"Default": {
  "Subject": "<subject; required>",
  "Store": "<cert store; required>",
  "Location": "<location; defaults to CurrentUser>",
  "AllowInvalid": "<true or false; defaults to false>"
}
```

<span data-ttu-id="13792-691">Note di schema:</span><span class="sxs-lookup"><span data-stu-id="13792-691">Schema notes:</span></span>

* <span data-ttu-id="13792-692">I nomi degli endpoint non applicano la distinzione tra maiuscole e minuscole.</span><span class="sxs-lookup"><span data-stu-id="13792-692">Endpoints names are case-insensitive.</span></span> <span data-ttu-id="13792-693">Ad esempio, sono validi sia `HTTPS` che `Https`.</span><span class="sxs-lookup"><span data-stu-id="13792-693">For example, `HTTPS` and `Https` are valid.</span></span>
* <span data-ttu-id="13792-694">Il parametro `Url` è obbligatorio per ogni endpoint.</span><span class="sxs-lookup"><span data-stu-id="13792-694">The `Url` parameter is required for each endpoint.</span></span> <span data-ttu-id="13792-695">Il formato per questo parametro è uguale a quello del parametro di configurazione di primo livello `Urls`, con la differenza che si limita a un singolo valore.</span><span class="sxs-lookup"><span data-stu-id="13792-695">The format for this parameter is the same as the top-level `Urls` configuration parameter except that it's limited to a single value.</span></span>
* <span data-ttu-id="13792-696">Questi endpoint sostituiscono quelli definiti nella configurazione di primo livello `Urls`, non vi si aggiungono.</span><span class="sxs-lookup"><span data-stu-id="13792-696">These endpoints replace those defined in the top-level `Urls` configuration rather than adding to them.</span></span> <span data-ttu-id="13792-697">Gli endpoint definiti nel codice tramite `Listen` si aggiungono agli endpoint definiti nella sezione di configurazione.</span><span class="sxs-lookup"><span data-stu-id="13792-697">Endpoints defined in code via `Listen` are cumulative with the endpoints defined in the configuration section.</span></span>
* <span data-ttu-id="13792-698">La sezione `Certificate` è facoltativa.</span><span class="sxs-lookup"><span data-stu-id="13792-698">The `Certificate` section is optional.</span></span> <span data-ttu-id="13792-699">Se la sezione `Certificate` non è specificata, vengono usati i valori predefiniti definiti negli scenari precedenti.</span><span class="sxs-lookup"><span data-stu-id="13792-699">If the `Certificate` section isn't specified, the defaults defined in earlier scenarios are used.</span></span> <span data-ttu-id="13792-700">Se non è disponibile alcun valore predefinito, il server genera un'eccezione e non viene avviato.</span><span class="sxs-lookup"><span data-stu-id="13792-700">If no defaults are available, the server throws an exception and fails to start.</span></span>
* <span data-ttu-id="13792-701">La `Certificate` sezione supporta sia la password del **percorso** &ndash;  che i certificati dell'archivio dell' **oggetto** &ndash;  .</span><span class="sxs-lookup"><span data-stu-id="13792-701">The `Certificate` section supports both **Path**&ndash;**Password** and **Subject**&ndash;**Store** certificates.</span></span>
* <span data-ttu-id="13792-702">In questo modo è possibile definire un numero qualsiasi di endpoint, purché non provochino conflitti di porte.</span><span class="sxs-lookup"><span data-stu-id="13792-702">Any number of endpoints may be defined in this way so long as they don't cause port conflicts.</span></span>
* <span data-ttu-id="13792-703">`options.Configure(context.Configuration.GetSection("{SECTION}"))` restituisce un oggetto `KestrelConfigurationLoader` con un metodo `.Endpoint(string name, listenOptions => { })` che può essere usato per integrare le impostazioni dell'endpoint configurato:</span><span class="sxs-lookup"><span data-stu-id="13792-703">`options.Configure(context.Configuration.GetSection("{SECTION}"))` returns a `KestrelConfigurationLoader` with an `.Endpoint(string name, listenOptions => { })` method that can be used to supplement a configured endpoint's settings:</span></span>

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .UseKestrel((context, serverOptions) =>
        {
            serverOptions.Configure(context.Configuration.GetSection("Kestrel"))
                .Endpoint("HTTPS", listenOptions =>
                {
                    listenOptions.HttpsOptions.SslProtocols = SslProtocols.Tls12;
                });
        });
```

<span data-ttu-id="13792-704">`KestrelServerOptions.ConfigurationLoader` è possibile accedere direttamente a per continuare a scorrere sul caricatore esistente, ad esempio quello fornito da <xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder*> .</span><span class="sxs-lookup"><span data-stu-id="13792-704">`KestrelServerOptions.ConfigurationLoader` can be directly accessed to continue iterating on the existing loader, such as the one provided by <xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder*>.</span></span>

* <span data-ttu-id="13792-705">La sezione di configurazione per ogni endpoint è disponibile nelle opzioni del `Endpoint` metodo in modo che sia possibile leggere le impostazioni personalizzate.</span><span class="sxs-lookup"><span data-stu-id="13792-705">The configuration section for each endpoint is available on the options in the `Endpoint` method so that custom settings may be read.</span></span>
* <span data-ttu-id="13792-706">È possibile caricare più configurazioni chiamando ancora `options.Configure(context.Configuration.GetSection("{SECTION}"))` con un'altra sezione.</span><span class="sxs-lookup"><span data-stu-id="13792-706">Multiple configurations may be loaded by calling `options.Configure(context.Configuration.GetSection("{SECTION}"))` again with another section.</span></span> <span data-ttu-id="13792-707">Viene usata solo l'ultima configurazione, a meno che `Load` sia stato chiamato esplicitamente in istanze precedenti.</span><span class="sxs-lookup"><span data-stu-id="13792-707">Only the last configuration is used, unless `Load` is explicitly called on prior instances.</span></span> <span data-ttu-id="13792-708">Il metapacchetto non chiama `Load` in modo che la sezione di configurazione predefinita venga sostituita.</span><span class="sxs-lookup"><span data-stu-id="13792-708">The metapackage doesn't call `Load` so that its default configuration section may be replaced.</span></span>
* <span data-ttu-id="13792-709">`KestrelConfigurationLoader` riflette la famiglia di API `Listen` da `KestrelServerOptions` all'overload di `Endpoint`, in modo che gli endpoint di codice e di configurazione possano essere configurati nella stessa posizione.</span><span class="sxs-lookup"><span data-stu-id="13792-709">`KestrelConfigurationLoader` mirrors the `Listen` family of APIs from `KestrelServerOptions` as `Endpoint` overloads, so code and config endpoints may be configured in the same place.</span></span> <span data-ttu-id="13792-710">Questi overload non usano nomi e usano solo le impostazioni predefinite della configurazione.</span><span class="sxs-lookup"><span data-stu-id="13792-710">These overloads don't use names and only consume default settings from configuration.</span></span>

<span data-ttu-id="13792-711">*Modificare le impostazioni predefinite nel codice*</span><span class="sxs-lookup"><span data-stu-id="13792-711">*Change the defaults in code*</span></span>

<span data-ttu-id="13792-712">`ConfigureEndpointDefaults` e `ConfigureHttpsDefaults` possono essere usati per modificare le impostazioni predefinite per `ListenOptions` e `HttpsConnectionAdapterOptions`, inclusa l'esecuzione dell'override del certificato predefinito specificato nello scenario precedente.</span><span class="sxs-lookup"><span data-stu-id="13792-712">`ConfigureEndpointDefaults` and `ConfigureHttpsDefaults` can be used to change default settings for `ListenOptions` and `HttpsConnectionAdapterOptions`, including overriding the default certificate specified in the prior scenario.</span></span> <span data-ttu-id="13792-713">`ConfigureEndpointDefaults` e `ConfigureHttpsDefaults` devono essere chiamati prima che venga configurato qualsiasi endpoint.</span><span class="sxs-lookup"><span data-stu-id="13792-713">`ConfigureEndpointDefaults` and `ConfigureHttpsDefaults` should be called before any endpoints are configured.</span></span>

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .UseKestrel((context, serverOptions) =>
        {
            serverOptions.ConfigureEndpointDefaults(listenOptions =>
            {
                // Configure endpoint defaults
            });
            
            serverOptions.ConfigureHttpsDefaults(listenOptions =>
            {
                listenOptions.SslProtocols = SslProtocols.Tls12;
            });
        });
```

<span data-ttu-id="13792-714">*Supporto kestrel per SNI*</span><span class="sxs-lookup"><span data-stu-id="13792-714">*Kestrel support for SNI*</span></span>

<span data-ttu-id="13792-715">[L'Indicazione nome server (SNI)](https://tools.ietf.org/html/rfc6066#section-3) può essere usata per ospitare più domini sulla stessa porta e indirizzo IP.</span><span class="sxs-lookup"><span data-stu-id="13792-715">[Server Name Indication (SNI)](https://tools.ietf.org/html/rfc6066#section-3) can be used to host multiple domains on the same IP address and port.</span></span> <span data-ttu-id="13792-716">Perché SNI funzioni è necessario che il client invii al server il nome host per la sessione protetta durante l'handshake TLS in modo che il server possa specificare il certificato corretto.</span><span class="sxs-lookup"><span data-stu-id="13792-716">For SNI to function, the client sends the host name for the secure session to the server during the TLS handshake so that the server can provide the correct certificate.</span></span> <span data-ttu-id="13792-717">Il client usa il certificato specificato per la comunicazione crittografata con il server durante la sessione protetta che segue l'handshake TLS.</span><span class="sxs-lookup"><span data-stu-id="13792-717">The client uses the furnished certificate for encrypted communication with the server during the secure session that follows the TLS handshake.</span></span>

<span data-ttu-id="13792-718">Kestrel supporta SNI tramite il callback `ServerCertificateSelector`.</span><span class="sxs-lookup"><span data-stu-id="13792-718">Kestrel supports SNI via the `ServerCertificateSelector` callback.</span></span> <span data-ttu-id="13792-719">Il callback viene richiamato una volta per ogni connessione per consentire all'app di controllare il nome host e selezionare il certificato appropriato.</span><span class="sxs-lookup"><span data-stu-id="13792-719">The callback is invoked once per connection to allow the app to inspect the host name and select the appropriate certificate.</span></span>

<span data-ttu-id="13792-720">Il supporto SNI richiede:</span><span class="sxs-lookup"><span data-stu-id="13792-720">SNI support requires:</span></span>

* <span data-ttu-id="13792-721">In esecuzione su Framework di destinazione `netcoreapp2.1` o versione successiva.</span><span class="sxs-lookup"><span data-stu-id="13792-721">Running on target framework `netcoreapp2.1` or later.</span></span> <span data-ttu-id="13792-722">In `net461` o versioni successive, il callback viene richiamato, ma `name` è sempre `null` .</span><span class="sxs-lookup"><span data-stu-id="13792-722">On `net461` or later, the callback is invoked but the `name` is always `null`.</span></span> <span data-ttu-id="13792-723">L'elemento `name` è `null` anche se il client non specifica il parametro del nome host nell'handshake TLS.</span><span class="sxs-lookup"><span data-stu-id="13792-723">The `name` is also `null` if the client doesn't provide the host name parameter in the TLS handshake.</span></span>
* <span data-ttu-id="13792-724">Esecuzione di tutti i siti Web nella stessa istanza di Kestrel.</span><span class="sxs-lookup"><span data-stu-id="13792-724">All websites run on the same Kestrel instance.</span></span> <span data-ttu-id="13792-725">Kestrel supporta la condivisione di un indirizzo IP e di una porta tra più istanze solo con un proxy inverso.</span><span class="sxs-lookup"><span data-stu-id="13792-725">Kestrel doesn't support sharing an IP address and port across multiple instances without a reverse proxy.</span></span>

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .ConfigureKestrel((context, serverOptions) =>
        {
            serverOptions.ListenAnyIP(5005, listenOptions =>
            {
                listenOptions.UseHttps(httpsOptions =>
                {
                    var localhostCert = CertificateLoader.LoadFromStoreCert(
                        "localhost", "My", StoreLocation.CurrentUser,
                        allowInvalid: true);
                    var exampleCert = CertificateLoader.LoadFromStoreCert(
                        "example.com", "My", StoreLocation.CurrentUser,
                        allowInvalid: true);
                    var subExampleCert = CertificateLoader.LoadFromStoreCert(
                        "sub.example.com", "My", StoreLocation.CurrentUser,
                        allowInvalid: true);
                    var certs = new Dictionary<string, X509Certificate2>(
                        StringComparer.OrdinalIgnoreCase);
                    certs["localhost"] = localhostCert;
                    certs["example.com"] = exampleCert;
                    certs["sub.example.com"] = subExampleCert;

                    httpsOptions.ServerCertificateSelector = (connectionContext, name) =>
                    {
                        if (name != null && certs.TryGetValue(name, out var cert))
                        {
                            return cert;
                        }

                        return exampleCert;
                    };
                });
            });
        });
```

### <a name="connection-logging"></a><span data-ttu-id="13792-726">Registrazione connessione</span><span class="sxs-lookup"><span data-stu-id="13792-726">Connection logging</span></span>

<span data-ttu-id="13792-727">Chiamare <xref:Microsoft.AspNetCore.Hosting.ListenOptionsConnectionLoggingExtensions.UseConnectionLogging*> per creare log a livello di debug per la comunicazione a livello di byte in una connessione.</span><span class="sxs-lookup"><span data-stu-id="13792-727">Call <xref:Microsoft.AspNetCore.Hosting.ListenOptionsConnectionLoggingExtensions.UseConnectionLogging*> to emit Debug level logs for byte-level communication on a connection.</span></span> <span data-ttu-id="13792-728">La registrazione delle connessioni è utile per la risoluzione dei problemi nella comunicazione di basso livello, ad esempio durante la crittografia TLS e dietro i proxy.</span><span class="sxs-lookup"><span data-stu-id="13792-728">Connection logging is helpful for troubleshooting problems in low-level communication, such as during TLS encryption and behind proxies.</span></span> <span data-ttu-id="13792-729">Se `UseConnectionLogging` viene inserito prima `UseHttps` , viene registrato il traffico crittografato.</span><span class="sxs-lookup"><span data-stu-id="13792-729">If `UseConnectionLogging` is placed before `UseHttps`, encrypted traffic is logged.</span></span> <span data-ttu-id="13792-730">Se `UseConnectionLogging` viene inserito dopo `UseHttps` , viene registrato il traffico decrittografato.</span><span class="sxs-lookup"><span data-stu-id="13792-730">If `UseConnectionLogging` is placed after `UseHttps`, decrypted traffic is logged.</span></span>

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.Listen(IPAddress.Any, 8000, listenOptions =>
    {
        listenOptions.UseConnectionLogging();
    });
});
```

### <a name="bind-to-a-tcp-socket"></a><span data-ttu-id="13792-731">Associazione a un socket TCP</span><span class="sxs-lookup"><span data-stu-id="13792-731">Bind to a TCP socket</span></span>

<span data-ttu-id="13792-732">Il metodo <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Listen*> esegue l'associazione a un socket TCP e un'espressione lambda per le opzioni consente di configurare un certificato X.509:</span><span class="sxs-lookup"><span data-stu-id="13792-732">The <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Listen*> method binds to a TCP socket, and an options lambda permits X.509 certificate configuration:</span></span>

[!code-csharp[](kestrel/samples/2.x/KestrelSample/Program.cs?name=snippet_TCPSocket&highlight=9-16)]

<span data-ttu-id="13792-733">L'esempio configura HTTPS per un endpoint con <xref:Microsoft.AspNetCore.Server.Kestrel.Core.ListenOptions>.</span><span class="sxs-lookup"><span data-stu-id="13792-733">The example configures HTTPS for an endpoint with <xref:Microsoft.AspNetCore.Server.Kestrel.Core.ListenOptions>.</span></span> <span data-ttu-id="13792-734">Usare la stessa API per configurare altre impostazioni di Kestrel per endpoint specifici.</span><span class="sxs-lookup"><span data-stu-id="13792-734">Use the same API to configure other Kestrel settings for specific endpoints.</span></span>

[!INCLUDE [How to make an X.509 cert](~/includes/make-x509-cert.md)]

### <a name="bind-to-a-unix-socket"></a><span data-ttu-id="13792-735">Associazione a un socket Unix</span><span class="sxs-lookup"><span data-stu-id="13792-735">Bind to a Unix socket</span></span>

<span data-ttu-id="13792-736">Impostare l'ascolto su un socket Unix con <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ListenUnixSocket*> per migliorare le prestazioni con Nginx, come visualizzato nell'esempio seguente:</span><span class="sxs-lookup"><span data-stu-id="13792-736">Listen on a Unix socket with <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ListenUnixSocket*> for improved performance with Nginx, as shown in this example:</span></span>

[!code-csharp[](kestrel/samples/2.x/KestrelSample/Program.cs?name=snippet_UnixSocket)]

* <span data-ttu-id="13792-737">Nel file nginx confiuguration impostare la `server`  >  `location`  >  `proxy_pass` voce su `http://unix:/tmp/{KESTREL SOCKET}:/;` .</span><span class="sxs-lookup"><span data-stu-id="13792-737">In the Nginx confiuguration file, set the `server` > `location` > `proxy_pass` entry to `http://unix:/tmp/{KESTREL SOCKET}:/;`.</span></span> <span data-ttu-id="13792-738">`{KESTREL SOCKET}` nome del socket fornito a <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ListenUnixSocket*> (ad esempio, `kestrel-test.sock` nell'esempio precedente).</span><span class="sxs-lookup"><span data-stu-id="13792-738">`{KESTREL SOCKET}` is the name of the socket provided to <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ListenUnixSocket*> (for example, `kestrel-test.sock` in the preceding example).</span></span>
* <span data-ttu-id="13792-739">Verificare che il socket sia scrivibile da nginx (ad esempio, `chmod go+w /tmp/kestrel-test.sock` ).</span><span class="sxs-lookup"><span data-stu-id="13792-739">Ensure that the socket is writeable by Nginx (for example, `chmod go+w /tmp/kestrel-test.sock`).</span></span> 

### <a name="port-0"></a><span data-ttu-id="13792-740">Porta 0</span><span class="sxs-lookup"><span data-stu-id="13792-740">Port 0</span></span>

<span data-ttu-id="13792-741">Quando si specifica il numero di porta `0`, Kestrel esegue l'associazione dinamica a una porta disponibile.</span><span class="sxs-lookup"><span data-stu-id="13792-741">When the port number `0` is specified, Kestrel dynamically binds to an available port.</span></span> <span data-ttu-id="13792-742">L'esempio seguente indica come determinare la porta alla quale Kestrel ha eseguito l'associazione in fase di esecuzione:</span><span class="sxs-lookup"><span data-stu-id="13792-742">The following example shows how to determine which port Kestrel actually bound at runtime:</span></span>

[!code-csharp[](kestrel/samples/2.x/KestrelSample/Startup.cs?name=snippet_Configure&highlight=3-4,15-21)]

<span data-ttu-id="13792-743">Quando si esegue l'app, l'output della finestra della console indica la porta dinamica in cui l'app può essere raggiunta:</span><span class="sxs-lookup"><span data-stu-id="13792-743">When the app is run, the console window output indicates the dynamic port where the app can be reached:</span></span>

```console
Listening on the following addresses: http://127.0.0.1:48508
```

### <a name="limitations"></a><span data-ttu-id="13792-744">Limitazioni</span><span class="sxs-lookup"><span data-stu-id="13792-744">Limitations</span></span>

<span data-ttu-id="13792-745">Configurare gli endpoint con i metodi seguenti:</span><span class="sxs-lookup"><span data-stu-id="13792-745">Configure endpoints with the following approaches:</span></span>

* <xref:Microsoft.AspNetCore.Hosting.HostingAbstractionsWebHostBuilderExtensions.UseUrls*>
* <span data-ttu-id="13792-746">L'argomento della riga di comando `--urls`</span><span class="sxs-lookup"><span data-stu-id="13792-746">`--urls` command-line argument</span></span>
* <span data-ttu-id="13792-747">La chiave di configurazione dell'host `urls`</span><span class="sxs-lookup"><span data-stu-id="13792-747">`urls` host configuration key</span></span>
* <span data-ttu-id="13792-748">La variabile di ambiente `ASPNETCORE_URLS`</span><span class="sxs-lookup"><span data-stu-id="13792-748">`ASPNETCORE_URLS` environment variable</span></span>

<span data-ttu-id="13792-749">Questi metodi sono utili se si vuole che il codice funzioni con server diversi da Kestrel.</span><span class="sxs-lookup"><span data-stu-id="13792-749">These methods are useful for making code work with servers other than Kestrel.</span></span> <span data-ttu-id="13792-750">Tenere presenti, tuttavia, le limitazioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="13792-750">However, be aware of the following limitations:</span></span>

* <span data-ttu-id="13792-751">HTTPS non può essere usato con questi approcci, a meno che non venga specificato un certificato predefinito nella configurazione dell'endpoint HTTPS (ad esempio, usando la configurazione `KestrelServerOptions` o un file di configurazione come illustrato in precedenza in questo argomento).</span><span class="sxs-lookup"><span data-stu-id="13792-751">HTTPS can't be used with these approaches unless a default certificate is provided in the HTTPS endpoint configuration (for example, using `KestrelServerOptions` configuration or a configuration file as shown earlier in this topic).</span></span>
* <span data-ttu-id="13792-752">Quando sia l'approccio `Listen` che l'approccio `UseUrls` vengono usati contemporaneamente, gli endpoint `Listen` eseguono l'override degli endpoint `UseUrls`.</span><span class="sxs-lookup"><span data-stu-id="13792-752">When both the `Listen` and `UseUrls` approaches are used simultaneously, the `Listen` endpoints override the `UseUrls` endpoints.</span></span>

### <a name="iis-endpoint-configuration"></a><span data-ttu-id="13792-753">Configurazione dell'endpoint IIS</span><span class="sxs-lookup"><span data-stu-id="13792-753">IIS endpoint configuration</span></span>

<span data-ttu-id="13792-754">Quando si usa IIS, le associazioni di URL per le associazioni di override di IIS vengono impostate mediante `Listen` o `UseUrls`.</span><span class="sxs-lookup"><span data-stu-id="13792-754">When using IIS, the URL bindings for IIS override bindings are set by either `Listen` or `UseUrls`.</span></span> <span data-ttu-id="13792-755">Per altre informazioni, vedere l'articolo [Introduzione al modulo ASP.NET Core](xref:host-and-deploy/aspnet-core-module).</span><span class="sxs-lookup"><span data-stu-id="13792-755">For more information, see the [ASP.NET Core Module](xref:host-and-deploy/aspnet-core-module) topic.</span></span>

### <a name="listenoptionsprotocols"></a><span data-ttu-id="13792-756">ListenOptions.Protocols</span><span class="sxs-lookup"><span data-stu-id="13792-756">ListenOptions.Protocols</span></span>

<span data-ttu-id="13792-757">La proprietà `Protocols` stabilisce i protocolli HTTP (`HttpProtocols`) abilitati in un endpoint di connessione o per il server.</span><span class="sxs-lookup"><span data-stu-id="13792-757">The `Protocols` property establishes the HTTP protocols (`HttpProtocols`) enabled on a connection endpoint or for the server.</span></span> <span data-ttu-id="13792-758">Assegnare un valore alla proprietà `Protocols` dall'enumerazione `HttpProtocols`.</span><span class="sxs-lookup"><span data-stu-id="13792-758">Assign a value to the `Protocols` property from the `HttpProtocols` enum.</span></span>

| <span data-ttu-id="13792-759">Valore di enumerazione `HttpProtocols`</span><span class="sxs-lookup"><span data-stu-id="13792-759">`HttpProtocols` enum value</span></span> | <span data-ttu-id="13792-760">Protocollo di connessione consentito</span><span class="sxs-lookup"><span data-stu-id="13792-760">Connection protocol permitted</span></span> |
| -------------------------- | ----------------------------- |
| `Http1`                    | <span data-ttu-id="13792-761">Solo HTTP/1.1.</span><span class="sxs-lookup"><span data-stu-id="13792-761">HTTP/1.1 only.</span></span> <span data-ttu-id="13792-762">Può essere usato con o senza TLS.</span><span class="sxs-lookup"><span data-stu-id="13792-762">Can be used with or without TLS.</span></span> |
| `Http2`                    | <span data-ttu-id="13792-763">Solo HTTP/2.</span><span class="sxs-lookup"><span data-stu-id="13792-763">HTTP/2 only.</span></span> <span data-ttu-id="13792-764">Può essere usato senza TLS solo se il client supporta una [modalità di conoscenza pregressa](https://tools.ietf.org/html/rfc7540#section-3.4).</span><span class="sxs-lookup"><span data-stu-id="13792-764">May be used without TLS only if the client supports a [Prior Knowledge mode](https://tools.ietf.org/html/rfc7540#section-3.4).</span></span> |
| `Http1AndHttp2`            | <span data-ttu-id="13792-765">HTTP/1.1 e HTTP/2.</span><span class="sxs-lookup"><span data-stu-id="13792-765">HTTP/1.1 and HTTP/2.</span></span> <span data-ttu-id="13792-766">HTTP/2 richiede una connessione TLS e [ALPN (Application-Layer Protocol negotiation)](https://tools.ietf.org/html/rfc7301#section-3) ; in caso contrario, il valore predefinito per la connessione è HTTP/1.1.</span><span class="sxs-lookup"><span data-stu-id="13792-766">HTTP/2 requires a TLS and [Application-Layer Protocol Negotiation (ALPN)](https://tools.ietf.org/html/rfc7301#section-3) connection; otherwise, the connection defaults to HTTP/1.1.</span></span> |

<span data-ttu-id="13792-767">Il protocollo predefinito è HTTP/1.1.</span><span class="sxs-lookup"><span data-stu-id="13792-767">The default protocol is HTTP/1.1.</span></span>

<span data-ttu-id="13792-768">Restrizioni relative a TLS per HTTP/2:</span><span class="sxs-lookup"><span data-stu-id="13792-768">TLS restrictions for HTTP/2:</span></span>

* <span data-ttu-id="13792-769">TLS versione 1.2 o successiva</span><span class="sxs-lookup"><span data-stu-id="13792-769">TLS version 1.2 or later</span></span>
* <span data-ttu-id="13792-770">Rinegoziazione disabilitata</span><span class="sxs-lookup"><span data-stu-id="13792-770">Renegotiation disabled</span></span>
* <span data-ttu-id="13792-771">Compressione disabilitata</span><span class="sxs-lookup"><span data-stu-id="13792-771">Compression disabled</span></span>
* <span data-ttu-id="13792-772">Dimensioni minime per lo scambio di chiavi temporanee:</span><span class="sxs-lookup"><span data-stu-id="13792-772">Minimum ephemeral key exchange sizes:</span></span>
  * <span data-ttu-id="13792-773">Curva ellittica Diffie-Hellman (ECDHE) &lbrack; [RFC4492](https://www.ietf.org/rfc/rfc4492.txt) &rbrack; : minimo 224 bit</span><span class="sxs-lookup"><span data-stu-id="13792-773">Elliptic curve Diffie-Hellman (ECDHE) &lbrack;[RFC4492](https://www.ietf.org/rfc/rfc4492.txt)&rbrack;: 224 bits minimum</span></span>
  * <span data-ttu-id="13792-774">Diffie-Hellman campo finito (DHE) &lbrack; `TLS12` &rbrack; : minimo 2048 bit</span><span class="sxs-lookup"><span data-stu-id="13792-774">Finite field Diffie-Hellman (DHE) &lbrack;`TLS12`&rbrack;: 2048 bits minimum</span></span>
* <span data-ttu-id="13792-775">Pacchetto di crittografia non bloccato</span><span class="sxs-lookup"><span data-stu-id="13792-775">Cipher suite not blocked</span></span>

<span data-ttu-id="13792-776">`TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256`&lbrack;`TLS-ECDHE`&rbrack; con la curva ellittica P-256 &lbrack; `FIPS186` &rbrack; è supportata per impostazione predefinita.</span><span class="sxs-lookup"><span data-stu-id="13792-776">`TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256` &lbrack;`TLS-ECDHE`&rbrack; with the P-256 elliptic curve &lbrack;`FIPS186`&rbrack; is supported by default.</span></span>

<span data-ttu-id="13792-777">Nell'esempio seguente sono consentite connessioni HTTP/1.1 e HTTP/2 sulla porta 8000.</span><span class="sxs-lookup"><span data-stu-id="13792-777">The following example permits HTTP/1.1 and HTTP/2 connections on port 8000.</span></span> <span data-ttu-id="13792-778">Le connessioni sono protette da TLS con un certificato incluso:</span><span class="sxs-lookup"><span data-stu-id="13792-778">Connections are secured by TLS with a supplied certificate:</span></span>

```csharp
.ConfigureKestrel((context, serverOptions) =>
{
    serverOptions.Listen(IPAddress.Any, 8000, listenOptions =>
    {
        listenOptions.Protocols = HttpProtocols.Http1AndHttp2;
        listenOptions.UseHttps("testCert.pfx", "testPassword");
    });
});
```

<span data-ttu-id="13792-779">Facoltativamente, creare un'implementazione `IConnectionAdapter` per filtrare gli handshake TLS in base alla connessione in caso di crittografie specifiche:</span><span class="sxs-lookup"><span data-stu-id="13792-779">Optionally create an `IConnectionAdapter` implementation to filter TLS handshakes on a per-connection basis for specific ciphers:</span></span>

```csharp
.ConfigureKestrel((context, serverOptions) =>
{
    serverOptions.Listen(IPAddress.Any, 8000, listenOptions =>
    {
        listenOptions.Protocols = HttpProtocols.Http1AndHttp2;
        listenOptions.UseHttps("testCert.pfx", "testPassword");
        listenOptions.ConnectionAdapters.Add(new TlsFilterAdapter());
    });
});
```

```csharp
private class TlsFilterAdapter : IConnectionAdapter
{
    public bool IsHttps => false;

    public Task<IAdaptedConnection> OnConnectionAsync(ConnectionAdapterContext context)
    {
        var tlsFeature = context.Features.Get<ITlsHandshakeFeature>();

        // Throw NotSupportedException for any cipher algorithm that the app doesn't
        // wish to support. Alternatively, define and compare
        // ITlsHandshakeFeature.CipherAlgorithm to a list of acceptable cipher
        // suites.
        //
        // No encryption is used with a CipherAlgorithmType.Null cipher algorithm.
        if (tlsFeature.CipherAlgorithm == CipherAlgorithmType.Null)
        {
            throw new NotSupportedException("Prohibited cipher: " + tlsFeature.CipherAlgorithm);
        }

        return Task.FromResult<IAdaptedConnection>(new AdaptedConnection(context.ConnectionStream));
    }

    private class AdaptedConnection : IAdaptedConnection
    {
        public AdaptedConnection(Stream adaptedStream)
        {
            ConnectionStream = adaptedStream;
        }

        public Stream ConnectionStream { get; }

        public void Dispose()
        {
        }
    }
}
```

<span data-ttu-id="13792-780">*Impostare il protocollo dalla configurazione*</span><span class="sxs-lookup"><span data-stu-id="13792-780">*Set the protocol from configuration*</span></span>

<span data-ttu-id="13792-781"><xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder*> chiama `serverOptions.Configure(context.Configuration.GetSection("Kestrel"))` per impostazione predefinita per caricare la configurazione di Kestrel.</span><span class="sxs-lookup"><span data-stu-id="13792-781"><xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder*> calls `serverOptions.Configure(context.Configuration.GetSection("Kestrel"))` by default to load Kestrel configuration.</span></span>

<span data-ttu-id="13792-782">Nell'esempio seguente *appsettings.json* viene stabilito un protocollo di connessione predefinito (http/1.1 e http/2) per tutti gli endpoint di Gheppio:</span><span class="sxs-lookup"><span data-stu-id="13792-782">In the following *appsettings.json* example, a default connection protocol (HTTP/1.1 and HTTP/2) is established for all of Kestrel's endpoints:</span></span>

```json
{
  "Kestrel": {
    "EndpointDefaults": {
      "Protocols": "Http1AndHttp2"
    }
  }
}
```

<span data-ttu-id="13792-783">Il file di configurazione di esempio seguente stabilisce un protocollo di connessione per un endpoint specifico:</span><span class="sxs-lookup"><span data-stu-id="13792-783">The following configuration file example establishes a connection protocol for a specific endpoint:</span></span>

```json
{
  "Kestrel": {
    "Endpoints": {
      "HttpsDefaultCert": {
        "Url": "https://localhost:5001",
        "Protocols": "Http1AndHttp2"
      }
    }
  }
}
```

<span data-ttu-id="13792-784">I protocolli specificati nei valori di override del codice sono impostati dalla configurazione.</span><span class="sxs-lookup"><span data-stu-id="13792-784">Protocols specified in code override values set by configuration.</span></span>

## <a name="libuv-transport-configuration"></a><span data-ttu-id="13792-785">Configurazione del trasporto libuv</span><span class="sxs-lookup"><span data-stu-id="13792-785">Libuv transport configuration</span></span>

<span data-ttu-id="13792-786">Con la versione ASP.NET Core 2.1, il trasporto predefinito di Kestrel non si basa più su Libuv, ma su socket gestiti.</span><span class="sxs-lookup"><span data-stu-id="13792-786">With the release of ASP.NET Core 2.1, Kestrel's default transport is no longer based on Libuv but instead based on managed sockets.</span></span> <span data-ttu-id="13792-787">Si tratta di una modifica importante per le app ASP.NET 2.0 Core che vengono aggiornate alla versione 2.1, che chiamano <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderLibuvExtensions.UseLibuv*> e dipendono da uno dei pacchetti seguenti:</span><span class="sxs-lookup"><span data-stu-id="13792-787">This is a breaking change for ASP.NET Core 2.0 apps upgrading to 2.1 that call <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderLibuvExtensions.UseLibuv*> and depend on either of the following packages:</span></span>

* <span data-ttu-id="13792-788">[Microsoft.AspNetCore.Server.Kestrel](https://www.nuget.org/packages/Microsoft.AspNetCore.Server.Kestrel/) (riferimento diretto al pacchetto)</span><span class="sxs-lookup"><span data-stu-id="13792-788">[Microsoft.AspNetCore.Server.Kestrel](https://www.nuget.org/packages/Microsoft.AspNetCore.Server.Kestrel/) (direct package reference)</span></span>
* [<span data-ttu-id="13792-789">Microsoft.AspNetCore.App</span><span class="sxs-lookup"><span data-stu-id="13792-789">Microsoft.AspNetCore.App</span></span>](https://www.nuget.org/packages/Microsoft.AspNetCore.App/)

<span data-ttu-id="13792-790">Per i progetti che richiedono l'uso di libuv:</span><span class="sxs-lookup"><span data-stu-id="13792-790">For projects that require the use of Libuv:</span></span>

* <span data-ttu-id="13792-791">Aggiungere una dipendenza per il pacchetto [Microsoft.AspNetCore.Server.Kestrel.Transport.Libuv](https://www.nuget.org/packages/Microsoft.AspNetCore.Server.Kestrel.Transport.Libuv/) al file di progetto dell'app:</span><span class="sxs-lookup"><span data-stu-id="13792-791">Add a dependency for the [Microsoft.AspNetCore.Server.Kestrel.Transport.Libuv](https://www.nuget.org/packages/Microsoft.AspNetCore.Server.Kestrel.Transport.Libuv/) package to the app's project file:</span></span>

  ```xml
  <PackageReference Include="Microsoft.AspNetCore.Server.Kestrel.Transport.Libuv"
                    Version="{VERSION}" />
  ```

* <span data-ttu-id="13792-792">Chiamare <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderLibuvExtensions.UseLibuv*>:</span><span class="sxs-lookup"><span data-stu-id="13792-792">Call <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderLibuvExtensions.UseLibuv*>:</span></span>

  ```csharp
  public class Program
  {
      public static void Main(string[] args)
      {
          CreateWebHostBuilder(args).Build().Run();
      }

      public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
          WebHost.CreateDefaultBuilder(args)
              .UseLibuv()
              .UseStartup<Startup>();
  }
  ```

### <a name="url-prefixes"></a><span data-ttu-id="13792-793">Prefissi URL</span><span class="sxs-lookup"><span data-stu-id="13792-793">URL prefixes</span></span>

<span data-ttu-id="13792-794">Se si usa `UseUrls`, l'argomento della riga di comando `--urls`, la chiave di configurazione dell'host `urls` o la variabile di ambiente `ASPNETCORE_URLS`, i prefissi URL possono avere uno dei formati seguenti.</span><span class="sxs-lookup"><span data-stu-id="13792-794">When using `UseUrls`, `--urls` command-line argument, `urls` host configuration key, or `ASPNETCORE_URLS` environment variable, the URL prefixes can be in any of the following formats.</span></span>

<span data-ttu-id="13792-795">Sono validi solo i prefissi URL HTTP.</span><span class="sxs-lookup"><span data-stu-id="13792-795">Only HTTP URL prefixes are valid.</span></span> <span data-ttu-id="13792-796">Kestrel non supporta HTTPS quando la configurazione delle associazioni URL viene eseguita con `UseUrls`.</span><span class="sxs-lookup"><span data-stu-id="13792-796">Kestrel doesn't support HTTPS when configuring URL bindings using `UseUrls`.</span></span>

* <span data-ttu-id="13792-797">Indirizzo IPv4 con numero di porta</span><span class="sxs-lookup"><span data-stu-id="13792-797">IPv4 address with port number</span></span>

  ```
  http://65.55.39.10:80/
  ```

  <span data-ttu-id="13792-798">`0.0.0.0` è un caso speciale che esegue l'associazione a tutti gli indirizzi IPv4.</span><span class="sxs-lookup"><span data-stu-id="13792-798">`0.0.0.0` is a special case that binds to all IPv4 addresses.</span></span>

* <span data-ttu-id="13792-799">Indirizzo IPv6 con numero di porta</span><span class="sxs-lookup"><span data-stu-id="13792-799">IPv6 address with port number</span></span>

  ```
  http://[0:0:0:0:0:ffff:4137:270a]:80/
  ```

  <span data-ttu-id="13792-800">`[::]` è l'equivalente IPv6 di `0.0.0.0` per IPv4.</span><span class="sxs-lookup"><span data-stu-id="13792-800">`[::]` is the IPv6 equivalent of IPv4 `0.0.0.0`.</span></span>

* <span data-ttu-id="13792-801">Nome host con numero di porta</span><span class="sxs-lookup"><span data-stu-id="13792-801">Host name with port number</span></span>

  ```
  http://contoso.com:80/
  http://*:80/
  ```

  <span data-ttu-id="13792-802">I nomi host, `*` e `+` non sono casi particolari.</span><span class="sxs-lookup"><span data-stu-id="13792-802">Host names, `*`, and `+`, aren't special.</span></span> <span data-ttu-id="13792-803">Tutto ciò che non è riconosciuto come un indirizzo IP o un elemento `localhost` valido esegue l'associazione a tutti gli IP IPv4 e IPv6.</span><span class="sxs-lookup"><span data-stu-id="13792-803">Anything not recognized as a valid IP address or `localhost` binds to all IPv4 and IPv6 IPs.</span></span> <span data-ttu-id="13792-804">Per associare nomi host diversi ad app ASP.NET Core diverse sulla stessa porta, usare [HTTP.sys](xref:fundamentals/servers/httpsys) o un server proxy inverso come IIS, Nginx o Apache.</span><span class="sxs-lookup"><span data-stu-id="13792-804">To bind different host names to different ASP.NET Core apps on the same port, use [HTTP.sys](xref:fundamentals/servers/httpsys) or a reverse proxy server, such as IIS, Nginx, or Apache.</span></span>

  > [!WARNING]
  > <span data-ttu-id="13792-805">Una configurazione che prevede un proxy inverso richiede il [filtro host](#host-filtering).</span><span class="sxs-lookup"><span data-stu-id="13792-805">Hosting in a reverse proxy configuration requires [host filtering](#host-filtering).</span></span>

* <span data-ttu-id="13792-806">Nome host `localhost` con numero di porta o IP di loopback con numero di porta</span><span class="sxs-lookup"><span data-stu-id="13792-806">Host `localhost` name with port number or loopback IP with port number</span></span>

  ```
  http://localhost:5000/
  http://127.0.0.1:5000/
  http://[::1]:5000/
  ```

  <span data-ttu-id="13792-807">Se è specificato `localhost`, Kestrel tenta l'associazione alle interfacce di loopback sia IPv4 che IPv6.</span><span class="sxs-lookup"><span data-stu-id="13792-807">When `localhost` is specified, Kestrel attempts to bind to both IPv4 and IPv6 loopback interfaces.</span></span> <span data-ttu-id="13792-808">Se la porta richiesta è in uso da parte di un altro servizio in una delle due interfacce di loopback, Kestrel non viene avviato.</span><span class="sxs-lookup"><span data-stu-id="13792-808">If the requested port is in use by another service on either loopback interface, Kestrel fails to start.</span></span> <span data-ttu-id="13792-809">Se una delle due interfacce di loopback non è disponibile per qualsiasi altro motivo (in genere perché IPv6 non è supportato), Kestrel registra un avviso.</span><span class="sxs-lookup"><span data-stu-id="13792-809">If either loopback interface is unavailable for any other reason (most commonly because IPv6 isn't supported), Kestrel logs a warning.</span></span>

## <a name="host-filtering"></a><span data-ttu-id="13792-810">Filtro host</span><span class="sxs-lookup"><span data-stu-id="13792-810">Host filtering</span></span>

<span data-ttu-id="13792-811">Mentre supporta la configurazione in base ai prefissi, ad esempio `http://example.com:5000`, Kestrel ignora quasi sempre il nome host.</span><span class="sxs-lookup"><span data-stu-id="13792-811">While Kestrel supports configuration based on prefixes such as `http://example.com:5000`, Kestrel largely ignores the host name.</span></span> <span data-ttu-id="13792-812">L'host `localhost` è un caso speciale usato per l'associazione agli indirizzi di loopback.</span><span class="sxs-lookup"><span data-stu-id="13792-812">Host `localhost` is a special case used for binding to loopback addresses.</span></span> <span data-ttu-id="13792-813">Qualsiasi host che non sia un indirizzo IP esplicito esegue l'associazione a tutti gli indirizzi IP pubblici.</span><span class="sxs-lookup"><span data-stu-id="13792-813">Any host other than an explicit IP address binds to all public IP addresses.</span></span> <span data-ttu-id="13792-814">Le intestazioni `Host` non vengono convalidate.</span><span class="sxs-lookup"><span data-stu-id="13792-814">`Host` headers aren't validated.</span></span>

<span data-ttu-id="13792-815">Come soluzione alternativa, usare il middleware di filtro host.</span><span class="sxs-lookup"><span data-stu-id="13792-815">As a workaround, use Host Filtering Middleware.</span></span> <span data-ttu-id="13792-816">Il middleware di filtro host viene fornito dal pacchetto [Microsoft. AspNetCore. HostFiltering](https://www.nuget.org/packages/Microsoft.AspNetCore.HostFiltering) , incluso nel [metapacchetto Microsoft. AspNetCore. App](xref:fundamentals/metapackage-app) (ASP.NET Core 2,1 o 2,2).</span><span class="sxs-lookup"><span data-stu-id="13792-816">Host Filtering Middleware is provided by the [Microsoft.AspNetCore.HostFiltering](https://www.nuget.org/packages/Microsoft.AspNetCore.HostFiltering) package, which is included in the [Microsoft.AspNetCore.App metapackage](xref:fundamentals/metapackage-app) (ASP.NET Core 2.1 or 2.2).</span></span> <span data-ttu-id="13792-817">Il middleware viene aggiunto da <xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder*>, che chiama <xref:Microsoft.AspNetCore.Builder.HostFilteringServicesExtensions.AddHostFiltering*>:</span><span class="sxs-lookup"><span data-stu-id="13792-817">The middleware is added by <xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder*>, which calls <xref:Microsoft.AspNetCore.Builder.HostFilteringServicesExtensions.AddHostFiltering*>:</span></span>

[!code-csharp[](kestrel/samples-snapshot/2.x/KestrelSample/Program.cs?name=snippet_Program&highlight=9)]

<span data-ttu-id="13792-818">Per impostazione predefinita, il middleware di filtro host è disabilitato per impostazione predefinita.</span><span class="sxs-lookup"><span data-stu-id="13792-818">Host Filtering Middleware is disabled by default.</span></span> <span data-ttu-id="13792-819">Per abilitare il middleware, definire una `AllowedHosts` chiave in *appsettings.json* / *appSettings. \<EnvironmentName> JSON*.</span><span class="sxs-lookup"><span data-stu-id="13792-819">To enable the middleware, define an `AllowedHosts` key in *appsettings.json*/*appsettings.\<EnvironmentName>.json*.</span></span> <span data-ttu-id="13792-820">Il valore è un elenco con valori delimitati da punto e virgola di nomi host senza numeri di porta:</span><span class="sxs-lookup"><span data-stu-id="13792-820">The value is a semicolon-delimited list of host names without port numbers:</span></span>

<span data-ttu-id="13792-821">*appsettings.json*:</span><span class="sxs-lookup"><span data-stu-id="13792-821">*appsettings.json*:</span></span>

```json
{
  "AllowedHosts": "example.com;localhost"
}
```

> [!NOTE]
> <span data-ttu-id="13792-822">Il [middleware delle intestazioni inoltrate](xref:host-and-deploy/proxy-load-balancer) include anche un'opzione <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.AllowedHosts>.</span><span class="sxs-lookup"><span data-stu-id="13792-822">[Forwarded Headers Middleware](xref:host-and-deploy/proxy-load-balancer) also has an <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.AllowedHosts> option.</span></span> <span data-ttu-id="13792-823">Il middleware di intestazioni inoltrate e il middleware di filtro host hanno funzionalità simili per diversi scenari.</span><span class="sxs-lookup"><span data-stu-id="13792-823">Forwarded Headers Middleware and Host Filtering Middleware have similar functionality for different scenarios.</span></span> <span data-ttu-id="13792-824">L'impostazione di `AllowedHosts` con il middleware delle intestazioni inoltrate è appropriato se l'intestazione `Host` non viene mantenuta durante l'inoltro delle richieste con un server proxy inverso o il bilanciamento del carico.</span><span class="sxs-lookup"><span data-stu-id="13792-824">Setting `AllowedHosts` with Forwarded Headers Middleware is appropriate when the `Host` header isn't preserved while forwarding requests with a reverse proxy server or load balancer.</span></span> <span data-ttu-id="13792-825">L'impostazione di `AllowedHosts` con il middleware del filtro host è appropriata se si usa Kestrel come server perimetrale rivolto al pubblico o se l'intestazione `Host` viene inoltrata direttamente.</span><span class="sxs-lookup"><span data-stu-id="13792-825">Setting `AllowedHosts` with Host Filtering Middleware is appropriate when Kestrel is used as a public-facing edge server or when the `Host` header is directly forwarded.</span></span>
>
> <span data-ttu-id="13792-826">Per altre informazioni sul middleware delle intestazioni inoltrate, vedere <xref:host-and-deploy/proxy-load-balancer>.</span><span class="sxs-lookup"><span data-stu-id="13792-826">For more information on Forwarded Headers Middleware, see <xref:host-and-deploy/proxy-load-balancer>.</span></span>

## <a name="http11-request-draining"></a><span data-ttu-id="13792-827">Svuotamento richieste HTTP/1.1</span><span class="sxs-lookup"><span data-stu-id="13792-827">HTTP/1.1 request draining</span></span>

<span data-ttu-id="13792-828">L'apertura di connessioni HTTP richiede molto tempo.</span><span class="sxs-lookup"><span data-stu-id="13792-828">Opening HTTP connections is time consuming.</span></span> <span data-ttu-id="13792-829">Per HTTPS, è anche a elevato utilizzo di risorse.</span><span class="sxs-lookup"><span data-stu-id="13792-829">For HTTPS, it's also resource intensive.</span></span> <span data-ttu-id="13792-830">Pertanto, gheppio tenta di riutilizzare le connessioni per il protocollo HTTP/1.1.</span><span class="sxs-lookup"><span data-stu-id="13792-830">Therefore, Kestrel tries to reuse connections per the HTTP/1.1 protocol.</span></span> <span data-ttu-id="13792-831">Il corpo della richiesta deve essere completamente utilizzato per consentire il riutilizzo della connessione.</span><span class="sxs-lookup"><span data-stu-id="13792-831">A request body must be fully consumed to allow the connection to be reused.</span></span> <span data-ttu-id="13792-832">L'app non usa sempre il corpo della richiesta, ad esempio le `POST` richieste in cui il server restituisce una risposta di reindirizzamento o 404.</span><span class="sxs-lookup"><span data-stu-id="13792-832">The app doesn't always consume the request body, such as a `POST` requests where the server returns a redirect or 404 response.</span></span> <span data-ttu-id="13792-833">Nel `POST` caso-Reindirizzamento:</span><span class="sxs-lookup"><span data-stu-id="13792-833">In the `POST`-redirect case:</span></span>

* <span data-ttu-id="13792-834">Il client potrebbe avere già inviato parte dei `POST` dati.</span><span class="sxs-lookup"><span data-stu-id="13792-834">The client may already have sent part of the `POST` data.</span></span>
* <span data-ttu-id="13792-835">Il server scrive la risposta 301.</span><span class="sxs-lookup"><span data-stu-id="13792-835">The server writes the 301 response.</span></span>
* <span data-ttu-id="13792-836">Non è possibile usare la connessione per una nuova richiesta fino a quando i `POST` dati del corpo della richiesta precedente non sono stati completamente letti.</span><span class="sxs-lookup"><span data-stu-id="13792-836">The connection can't be used for a new request until the `POST` data from the previous request body has been fully read.</span></span>
* <span data-ttu-id="13792-837">Gheppio tenta di svuotare il corpo della richiesta.</span><span class="sxs-lookup"><span data-stu-id="13792-837">Kestrel tries to drain the request body.</span></span> <span data-ttu-id="13792-838">Lo svuotamento del corpo della richiesta implica la lettura e l'eliminazione dei dati senza elaborarli.</span><span class="sxs-lookup"><span data-stu-id="13792-838">Draining the request body means reading and discarding the data without processing it.</span></span>

<span data-ttu-id="13792-839">Il processo di svuotamento crea un tradoff tra la possibilità di riutilizzare la connessione e il tempo necessario per svuotare i dati rimanenti:</span><span class="sxs-lookup"><span data-stu-id="13792-839">The draining process makes a tradoff between allowing the connection to be reused and the time it takes to drain any remaining data:</span></span>

* <span data-ttu-id="13792-840">Lo svuotamento ha un timeout di cinque secondi, che non è configurabile.</span><span class="sxs-lookup"><span data-stu-id="13792-840">Draining has a timeout of five seconds, which isn't configurable.</span></span>
* <span data-ttu-id="13792-841">Se tutti i dati specificati dall' `Content-Length` `Transfer-Encoding` intestazione o non sono stati letti prima del timeout, la connessione viene chiusa.</span><span class="sxs-lookup"><span data-stu-id="13792-841">If all of the data specified by the `Content-Length` or `Transfer-Encoding` header hasn't been read before the timeout, the connection is closed.</span></span>

<span data-ttu-id="13792-842">In alcuni casi può essere necessario terminare immediatamente la richiesta, prima o dopo la scrittura della risposta.</span><span class="sxs-lookup"><span data-stu-id="13792-842">Sometimes you may want to terminate the request immediately, before or after writing the response.</span></span> <span data-ttu-id="13792-843">È ad esempio possibile che i client dispongano di protezioni dati restrittive, pertanto la limitazione dei dati caricati potrebbe avere una priorità.</span><span class="sxs-lookup"><span data-stu-id="13792-843">For example, clients may have restrictive data caps, so limiting uploaded data might be a priority.</span></span> <span data-ttu-id="13792-844">In questi casi, per terminare una richiesta, chiamare [HttpContext. Abort](xref:Microsoft.AspNetCore.Http.HttpContext.Abort%2A) da un controller, una Razor pagina o un middleware.</span><span class="sxs-lookup"><span data-stu-id="13792-844">In such cases to terminate a request, call [HttpContext.Abort](xref:Microsoft.AspNetCore.Http.HttpContext.Abort%2A) from a controller, Razor Page, or middleware.</span></span>

<span data-ttu-id="13792-845">Ci sono alcune avvertenze per chiamare `Abort` :</span><span class="sxs-lookup"><span data-stu-id="13792-845">There are caveats to calling `Abort`:</span></span>

* <span data-ttu-id="13792-846">La creazione di nuove connessioni può essere lenta e dispendiosa.</span><span class="sxs-lookup"><span data-stu-id="13792-846">Creating new connections can be slow and expensive.</span></span>
* <span data-ttu-id="13792-847">Non vi è alcuna garanzia che il client abbia letto la risposta prima che la connessione venga chiusa.</span><span class="sxs-lookup"><span data-stu-id="13792-847">There's no guarantee that the client has read the response before the connection closes.</span></span>
* <span data-ttu-id="13792-848">`Abort`La chiamata a deve essere rara e riservata per i casi di errore gravi, non per errori comuni.</span><span class="sxs-lookup"><span data-stu-id="13792-848">Calling `Abort` should be rare and reserved for severe error cases, not common errors.</span></span>
  * <span data-ttu-id="13792-849">Chiamare solo `Abort` quando è necessario risolvere un problema specifico.</span><span class="sxs-lookup"><span data-stu-id="13792-849">Only call `Abort` when a specific problem needs to be solved.</span></span> <span data-ttu-id="13792-850">Ad esempio, chiamare `Abort` se i client malintenzionati tentano di eseguire `POST` dati o quando si verifica un bug nel codice client che genera richieste di grandi dimensioni o numerose.</span><span class="sxs-lookup"><span data-stu-id="13792-850">For example, call `Abort` if malicious clients are trying to `POST` data or when there's a bug in client code that causes large or numerous requests.</span></span>
  * <span data-ttu-id="13792-851">Non chiamare le `Abort` situazioni di errore comuni, ad esempio HTTP 404 (non trovato).</span><span class="sxs-lookup"><span data-stu-id="13792-851">Don't call `Abort` for common error situations, such as HTTP 404 (Not Found).</span></span>

<span data-ttu-id="13792-852">Quando si chiama [HttpResponse. CompleteAsync](xref:Microsoft.AspNetCore.Http.HttpResponse.CompleteAsync%2A) , prima `Abort` di chiamare si garantisce che il server abbia completato la scrittura della risposta.</span><span class="sxs-lookup"><span data-stu-id="13792-852">Calling [HttpResponse.CompleteAsync](xref:Microsoft.AspNetCore.Http.HttpResponse.CompleteAsync%2A) before calling `Abort` ensures that the server has completed writing the response.</span></span> <span data-ttu-id="13792-853">Tuttavia, il comportamento del client non è prevedibile e potrebbe non essere in grado di leggere la risposta prima che la connessione venga interrotta.</span><span class="sxs-lookup"><span data-stu-id="13792-853">However, client behavior isn't predictable and they may not read the response before the connection is aborted.</span></span>

<span data-ttu-id="13792-854">Questo processo è diverso per HTTP/2 poiché il protocollo supporta l'interruzione dei singoli flussi di richiesta senza chiudere la connessione.</span><span class="sxs-lookup"><span data-stu-id="13792-854">This process is different for HTTP/2 because the protocol supports aborting individual request streams without closing the connection.</span></span> <span data-ttu-id="13792-855">Il timeout di svuotamento di cinque secondi non è applicabile.</span><span class="sxs-lookup"><span data-stu-id="13792-855">The five second drain timeout doesn't apply.</span></span> <span data-ttu-id="13792-856">Se dopo il completamento di una risposta sono presenti dati del corpo della richiesta non letti, il server invia un frame RST HTTP/2.</span><span class="sxs-lookup"><span data-stu-id="13792-856">If there's any unread request body data after completing a response, then the server sends an HTTP/2 RST frame.</span></span> <span data-ttu-id="13792-857">I frame di dati del corpo della richiesta aggiuntivi vengono ignorati.</span><span class="sxs-lookup"><span data-stu-id="13792-857">Additional request body data frames are ignored.</span></span>

<span data-ttu-id="13792-858">Se possibile, è preferibile che i client utilizzino l'intestazione della richiesta [expect: 100-continue](https://developer.mozilla.org/docs/Web/HTTP/Status/100) e attendano la risposta del server prima di iniziare a inviare il corpo della richiesta.</span><span class="sxs-lookup"><span data-stu-id="13792-858">If possible, it's better for clients to utilize the [Expect: 100-continue](https://developer.mozilla.org/docs/Web/HTTP/Status/100) request header and wait for the server to respond before starting to send the request body.</span></span> <span data-ttu-id="13792-859">Che fornisce al client la possibilità di esaminare la risposta e interrompere prima di inviare dati non necessari.</span><span class="sxs-lookup"><span data-stu-id="13792-859">That gives the client an opportunity to examine the response and abort before sending unneeded data.</span></span>

## <a name="additional-resources"></a><span data-ttu-id="13792-860">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="13792-860">Additional resources</span></span>

* <span data-ttu-id="13792-861">Quando si usano i socket UNIX in Linux, il socket non viene eliminato automaticamente all'arresto dell'app.</span><span class="sxs-lookup"><span data-stu-id="13792-861">When using UNIX sockets on Linux, the socket is not automatically deleted on app shut down.</span></span> <span data-ttu-id="13792-862">Per altre informazioni, vedere [questo problema di GitHub](https://github.com/dotnet/aspnetcore/issues/14134).</span><span class="sxs-lookup"><span data-stu-id="13792-862">For more information, see [this GitHub issue](https://github.com/dotnet/aspnetcore/issues/14134).</span></span>
* <xref:test/troubleshoot>
* <xref:security/enforcing-ssl>
* <xref:host-and-deploy/proxy-load-balancer>
* <span data-ttu-id="13792-863">[RFC 7230: Message Syntax and Routing (Section 5.4: Host)](https://tools.ietf.org/html/rfc7230#section-5.4) (RFC 7230: Sintassi e routing dei messaggi (sezione 5.4: Host))</span><span class="sxs-lookup"><span data-stu-id="13792-863">[RFC 7230: Message Syntax and Routing (Section 5.4: Host)](https://tools.ietf.org/html/rfc7230#section-5.4)</span></span>

::: moniker-end

::: moniker range="= aspnetcore-2.1"

<span data-ttu-id="13792-864">Kestrel è un [server Web per ASP.NET Core](xref:fundamentals/servers/index) multipiattaforma.</span><span class="sxs-lookup"><span data-stu-id="13792-864">Kestrel is a cross-platform [web server for ASP.NET Core](xref:fundamentals/servers/index).</span></span> <span data-ttu-id="13792-865">Kestrel è il server Web incluso per impostazione predefinita nei modelli di progetto di ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="13792-865">Kestrel is the web server that's included by default in ASP.NET Core project templates.</span></span>

<span data-ttu-id="13792-866">Kestrel supporta gli scenari seguenti:</span><span class="sxs-lookup"><span data-stu-id="13792-866">Kestrel supports the following scenarios:</span></span>

* <span data-ttu-id="13792-867">HTTPS</span><span class="sxs-lookup"><span data-stu-id="13792-867">HTTPS</span></span>
* <span data-ttu-id="13792-868">Aggiornamento opaco usato per abilitare [WebSocket](https://github.com/aspnet/websockets)</span><span class="sxs-lookup"><span data-stu-id="13792-868">Opaque upgrade used to enable [WebSockets](https://github.com/aspnet/websockets)</span></span>
* <span data-ttu-id="13792-869">Socket Unix ad alte prestazioni dietro Nginx</span><span class="sxs-lookup"><span data-stu-id="13792-869">Unix sockets for high performance behind Nginx</span></span>

<span data-ttu-id="13792-870">Kestrel è supportato in tutte le piattaforme e le versioni supportate da .NET Core.</span><span class="sxs-lookup"><span data-stu-id="13792-870">Kestrel is supported on all platforms and versions that .NET Core supports.</span></span>

<span data-ttu-id="13792-871">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/servers/kestrel/samples/2.x) ([procedura per il download](xref:index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="13792-871">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/servers/kestrel/samples/2.x) ([how to download](xref:index#how-to-download-a-sample))</span></span>

## <a name="when-to-use-kestrel-with-a-reverse-proxy"></a><span data-ttu-id="13792-872">Quando usare Kestrel con un proxy inverso</span><span class="sxs-lookup"><span data-stu-id="13792-872">When to use Kestrel with a reverse proxy</span></span>

<span data-ttu-id="13792-873">È possibile usare Kestrel da solo o in combinazione con un *server proxy inverso*, ad esempio [Internet Information Services (IIS)](https://www.iis.net/), [Nginx](https://nginx.org) o [Apache](https://httpd.apache.org/).</span><span class="sxs-lookup"><span data-stu-id="13792-873">Kestrel can be used by itself or with a *reverse proxy server*, such as [Internet Information Services (IIS)](https://www.iis.net/), [Nginx](https://nginx.org), or [Apache](https://httpd.apache.org/).</span></span> <span data-ttu-id="13792-874">Il server proxy inverso riceve le richieste HTTP dalla rete e le inoltra a Kestrel.</span><span class="sxs-lookup"><span data-stu-id="13792-874">A reverse proxy server receives HTTP requests from the network and forwards them to Kestrel.</span></span>

<span data-ttu-id="13792-875">Kestrel usato come server Web perimetrale (esposto a Internet):</span><span class="sxs-lookup"><span data-stu-id="13792-875">Kestrel used as an edge (Internet-facing) web server:</span></span>

![Kestrel comunica direttamente con Internet senza un server proxy inverso](kestrel/_static/kestrel-to-internet2.png)

<span data-ttu-id="13792-877">Kestrel usato in una configurazione proxy inverso:</span><span class="sxs-lookup"><span data-stu-id="13792-877">Kestrel used in a reverse proxy configuration:</span></span>

![Kestrel comunica indirettamente con Internet attraverso un server proxy inverso, ad esempio IIS, Nginx o Apache](kestrel/_static/kestrel-to-internet.png)

<span data-ttu-id="13792-879">La configurazione, con o senza un server proxy inverso, è una configurazione di hosting supportata.</span><span class="sxs-lookup"><span data-stu-id="13792-879">Either configuration, with or without a reverse proxy server, is a supported hosting configuration.</span></span>

<span data-ttu-id="13792-880">Se viene utilizzato come server perimetrale in assenza di un server proxy inverso, Kestrel non supporta la condivisione tra più processi dell'IP e della porta.</span><span class="sxs-lookup"><span data-stu-id="13792-880">Kestrel used as an edge server without a reverse proxy server doesn't support sharing the same IP and port among multiple processes.</span></span> <span data-ttu-id="13792-881">Quando è configurato per restare in ascolto su una porta, Kestrel gestisce tutto il traffico per tale porta indipendentemente dalle intestazioni `Host` delle richieste.</span><span class="sxs-lookup"><span data-stu-id="13792-881">When Kestrel is configured to listen on a port, Kestrel handles all of the traffic for that port regardless of requests' `Host` headers.</span></span> <span data-ttu-id="13792-882">Un proxy inverso che può condividere le porte può eseguire l'inoltro delle richieste a Kestrel su un unico IP e un'unica porta.</span><span class="sxs-lookup"><span data-stu-id="13792-882">A reverse proxy that can share ports has the ability to forward requests to Kestrel on a unique IP and port.</span></span>

<span data-ttu-id="13792-883">Anche se la presenza di un server proxy inverso non è necessaria, può risultare utile per i motivi seguenti.</span><span class="sxs-lookup"><span data-stu-id="13792-883">Even if a reverse proxy server isn't required, using a reverse proxy server might be a good choice.</span></span>

<span data-ttu-id="13792-884">Un proxy inverso:</span><span class="sxs-lookup"><span data-stu-id="13792-884">A reverse proxy:</span></span>

* <span data-ttu-id="13792-885">Può limitare l'area della superficie di attacco pubblica esposta delle app ospitate.</span><span class="sxs-lookup"><span data-stu-id="13792-885">Can limit the exposed public surface area of the apps that it hosts.</span></span>
* <span data-ttu-id="13792-886">Offre un livello aggiuntivo di configurazione e protezione.</span><span class="sxs-lookup"><span data-stu-id="13792-886">Provide an additional layer of configuration and defense.</span></span>
* <span data-ttu-id="13792-887">Potrebbe offrire un'integrazione migliore con l'infrastruttura esistente.</span><span class="sxs-lookup"><span data-stu-id="13792-887">Might integrate better with existing infrastructure.</span></span>
* <span data-ttu-id="13792-888">Semplifica il bilanciamento del carico e la configurazione di comunicazioni protette (HTTPS).</span><span class="sxs-lookup"><span data-stu-id="13792-888">Simplify load balancing and secure communication (HTTPS) configuration.</span></span> <span data-ttu-id="13792-889">Solo il server proxy inverso richiede un certificato X. 509 e il server è in grado di comunicare con i server dell'app nella rete interna tramite HTTP normale.</span><span class="sxs-lookup"><span data-stu-id="13792-889">Only the reverse proxy server requires an X.509 certificate, and that server can communicate with the app's servers on the internal network using plain HTTP.</span></span>

> [!WARNING]
> <span data-ttu-id="13792-890">Una configurazione che prevede un proxy inverso richiede il [filtro host](#host-filtering).</span><span class="sxs-lookup"><span data-stu-id="13792-890">Hosting in a reverse proxy configuration requires [host filtering](#host-filtering).</span></span>

## <a name="how-to-use-kestrel-in-aspnet-core-apps"></a><span data-ttu-id="13792-891">Come usare Kestrel nelle app ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="13792-891">How to use Kestrel in ASP.NET Core apps</span></span>

<span data-ttu-id="13792-892">Il pacchetto [Microsoft. AspNetCore. Server. gheppio](https://www.nuget.org/packages/Microsoft.AspNetCore.Server.Kestrel/) è incluso nel [metapacchetto Microsoft. AspNetCore. app](xref:fundamentals/metapackage-app).</span><span class="sxs-lookup"><span data-stu-id="13792-892">The [Microsoft.AspNetCore.Server.Kestrel](https://www.nuget.org/packages/Microsoft.AspNetCore.Server.Kestrel/) package is included in the [Microsoft.AspNetCore.App metapackage](xref:fundamentals/metapackage-app).</span></span>

<span data-ttu-id="13792-893">I modelli di progetto ASP.NET Core usano Kestrel per impostazione predefinita.</span><span class="sxs-lookup"><span data-stu-id="13792-893">ASP.NET Core project templates use Kestrel by default.</span></span> <span data-ttu-id="13792-894">In *Program.cs* il codice del modello chiama <xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder*>, che chiama <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderKestrelExtensions.UseKestrel*> in background.</span><span class="sxs-lookup"><span data-stu-id="13792-894">In *Program.cs*, the template code calls <xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder*>, which calls <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderKestrelExtensions.UseKestrel*> behind the scenes.</span></span>

<span data-ttu-id="13792-895">Per fornire una configurazione aggiuntiva dopo la chiamata di `CreateDefaultBuilder`, chiamare <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderKestrelExtensions.UseKestrel*>:</span><span class="sxs-lookup"><span data-stu-id="13792-895">To provide additional configuration after calling `CreateDefaultBuilder`, call <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderKestrelExtensions.UseKestrel*>:</span></span>

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .UseKestrel(serverOptions =>
        {
            // Set properties and call methods on serverOptions
        });
```

<span data-ttu-id="13792-896">Per ulteriori informazioni su `CreateDefaultBuilder` e sulla compilazione dell'host, vedere la sezione *configurare un host* di <xref:fundamentals/host/web-host#set-up-a-host> .</span><span class="sxs-lookup"><span data-stu-id="13792-896">For more information on `CreateDefaultBuilder` and building the host, see the *Set up a host* section of <xref:fundamentals/host/web-host#set-up-a-host>.</span></span>

## <a name="kestrel-options"></a><span data-ttu-id="13792-897">Opzioni Kestrel</span><span class="sxs-lookup"><span data-stu-id="13792-897">Kestrel options</span></span>

<span data-ttu-id="13792-898">Il server Web Kestrel dispone di opzioni di configurazione dei vincoli che risultano particolarmente utili nelle distribuzioni con connessione Internet.</span><span class="sxs-lookup"><span data-stu-id="13792-898">The Kestrel web server has constraint configuration options that are especially useful in Internet-facing deployments.</span></span>

<span data-ttu-id="13792-899">Impostare i vincoli per la proprietà <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Limits> della classe <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions>.</span><span class="sxs-lookup"><span data-stu-id="13792-899">Set constraints on the <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Limits> property of the <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions> class.</span></span> <span data-ttu-id="13792-900">La proprietà `Limits` contiene un'istanza della classe <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits>.</span><span class="sxs-lookup"><span data-stu-id="13792-900">The `Limits` property holds an instance of the <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits> class.</span></span>

<span data-ttu-id="13792-901">Negli esempi seguenti viene usato lo spazio dei nomi <xref:Microsoft.AspNetCore.Server.Kestrel.Core>:</span><span class="sxs-lookup"><span data-stu-id="13792-901">The following examples use the <xref:Microsoft.AspNetCore.Server.Kestrel.Core> namespace:</span></span>

```csharp
using Microsoft.AspNetCore.Server.Kestrel.Core;
```

<span data-ttu-id="13792-902">Le opzioni gheppio, che sono configurate nel codice C# negli esempi seguenti, possono essere impostate anche usando un [provider di configurazione](xref:fundamentals/configuration/index).</span><span class="sxs-lookup"><span data-stu-id="13792-902">Kestrel options, which are configured in C# code in the following examples, can also be set using a [configuration provider](xref:fundamentals/configuration/index).</span></span> <span data-ttu-id="13792-903">Il provider di configurazione file, ad esempio, può caricare la configurazione di Gheppio da un oggetto *appsettings.json* o *appSettings. { File Environment}. JSON* :</span><span class="sxs-lookup"><span data-stu-id="13792-903">For example, the File Configuration Provider can load Kestrel configuration from an *appsettings.json* or *appsettings.{Environment}.json* file:</span></span>

```json
{
  "Kestrel": {
    "Limits": {
      "MaxConcurrentConnections": 100,
      "MaxConcurrentUpgradedConnections": 100
    }
  }
}
```

<span data-ttu-id="13792-904">Usare **uno** degli approcci seguenti:</span><span class="sxs-lookup"><span data-stu-id="13792-904">Use **one** of the following approaches:</span></span>

* <span data-ttu-id="13792-905">Configurare gheppio in `Startup.ConfigureServices` :</span><span class="sxs-lookup"><span data-stu-id="13792-905">Configure Kestrel in `Startup.ConfigureServices`:</span></span>

  1. <span data-ttu-id="13792-906">Inserire un'istanza di `IConfiguration` nella `Startup` classe.</span><span class="sxs-lookup"><span data-stu-id="13792-906">Inject an instance of `IConfiguration` into the `Startup` class.</span></span> <span data-ttu-id="13792-907">Nell'esempio seguente si presuppone che la configurazione inserita venga assegnata alla `Configuration` Proprietà.</span><span class="sxs-lookup"><span data-stu-id="13792-907">The following example assumes that the injected configuration is assigned to the `Configuration` property.</span></span>
  2. <span data-ttu-id="13792-908">In `Startup.ConfigureServices` caricare la `Kestrel` sezione della configurazione nella configurazione di Gheppio:</span><span class="sxs-lookup"><span data-stu-id="13792-908">In `Startup.ConfigureServices`, load the `Kestrel` section of configuration into Kestrel's configuration:</span></span>

     ```csharp
     using Microsoft.Extensions.Configuration
     
     public class Startup
     {
         public Startup(IConfiguration configuration)
         {
             Configuration = configuration;
         }

         public IConfiguration Configuration { get; }

         public void ConfigureServices(IServiceCollection services)
         {
             services.Configure<KestrelServerOptions>(
                 Configuration.GetSection("Kestrel"));
         }

         public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
         {
             ...
         }
     }
     ```

* <span data-ttu-id="13792-909">Configurare il gheppio durante la compilazione dell'host:</span><span class="sxs-lookup"><span data-stu-id="13792-909">Configure Kestrel when building the host:</span></span>

  <span data-ttu-id="13792-910">In *Program.cs* caricare la `Kestrel` sezione della configurazione nella configurazione di Gheppio:</span><span class="sxs-lookup"><span data-stu-id="13792-910">In *Program.cs*, load the `Kestrel` section of configuration into Kestrel's configuration:</span></span>

  ```csharp
  // using Microsoft.Extensions.DependencyInjection;

  public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
      WebHost.CreateDefaultBuilder(args)
          .ConfigureServices((context, services) =>
          {
              services.Configure<KestrelServerOptions>(
                  context.Configuration.GetSection("Kestrel"));
          })
          .UseStartup<Startup>();
  ```

<span data-ttu-id="13792-911">Entrambi gli approcci precedenti funzionano con qualsiasi [provider di configurazione](xref:fundamentals/configuration/index).</span><span class="sxs-lookup"><span data-stu-id="13792-911">Both of the preceding approaches work with any [configuration provider](xref:fundamentals/configuration/index).</span></span>

### <a name="keep-alive-timeout"></a><span data-ttu-id="13792-912">Timeout keep-alive</span><span class="sxs-lookup"><span data-stu-id="13792-912">Keep-alive timeout</span></span>

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.KeepAliveTimeout>

<span data-ttu-id="13792-913">Ottiene o imposta il [timeout keep-alive](https://tools.ietf.org/html/rfc7230#section-6.5).</span><span class="sxs-lookup"><span data-stu-id="13792-913">Gets or sets the [keep-alive timeout](https://tools.ietf.org/html/rfc7230#section-6.5).</span></span> <span data-ttu-id="13792-914">Il valore predefinito è 2 minuti.</span><span class="sxs-lookup"><span data-stu-id="13792-914">Defaults to 2 minutes.</span></span>

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .UseKestrel(serverOptions =>
        {
            serverOptions.Limits.KeepAliveTimeout = TimeSpan.FromMinutes(2);
        });
```

### <a name="maximum-client-connections"></a><span data-ttu-id="13792-915">Numero massimo di connessioni client</span><span class="sxs-lookup"><span data-stu-id="13792-915">Maximum client connections</span></span>

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.MaxConcurrentConnections>
<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.MaxConcurrentUpgradedConnections>

<span data-ttu-id="13792-916">È possibile impostare il numero massimo di connessioni TCP aperte simultaneamente per l'intera app con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="13792-916">The maximum number of concurrent open TCP connections can be set for the entire app with the following code:</span></span>

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .UseKestrel(serverOptions =>
        {
            serverOptions.Limits.MaxConcurrentConnections = 100;
        });
```

<span data-ttu-id="13792-917">È previsto un limite separato per le connessioni che sono state aggiornate da HTTP o HTTPS a un altro protocollo (ad esempio su una richiesta WebSocket).</span><span class="sxs-lookup"><span data-stu-id="13792-917">There's a separate limit for connections that have been upgraded from HTTP or HTTPS to another protocol (for example, on a WebSockets request).</span></span> <span data-ttu-id="13792-918">Dopo l'aggiornamento di una connessione, questa non viene conteggiata per il limite `MaxConcurrentConnections`.</span><span class="sxs-lookup"><span data-stu-id="13792-918">After a connection is upgraded, it isn't counted against the `MaxConcurrentConnections` limit.</span></span>

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .UseKestrel(serverOptions =>
        {
            serverOptions.Limits.MaxConcurrentUpgradedConnections = 100;
        });
```

<span data-ttu-id="13792-919">Per impostazione predefinita, il numero massimo di connessioni è illimitato (null).</span><span class="sxs-lookup"><span data-stu-id="13792-919">The maximum number of connections is unlimited (null) by default.</span></span>

### <a name="maximum-request-body-size"></a><span data-ttu-id="13792-920">Dimensione massima del corpo della richiesta</span><span class="sxs-lookup"><span data-stu-id="13792-920">Maximum request body size</span></span>

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.MaxRequestBodySize>

<span data-ttu-id="13792-921">La dimensione massima predefinita del corpo della richiesta è pari a 30.000.000 di byte, equivalenti a circa 28,6 MB.</span><span class="sxs-lookup"><span data-stu-id="13792-921">The default maximum request body size is 30,000,000 bytes, which is approximately 28.6 MB.</span></span>

<span data-ttu-id="13792-922">Il metodo consigliato per ignorare il limite in un'app ASP.NET Core MVC è l'uso dell'attributo <xref:Microsoft.AspNetCore.Mvc.RequestSizeLimitAttribute> in un metodo di azione:</span><span class="sxs-lookup"><span data-stu-id="13792-922">The recommended approach to override the limit in an ASP.NET Core MVC app is to use the <xref:Microsoft.AspNetCore.Mvc.RequestSizeLimitAttribute> attribute on an action method:</span></span>

```csharp
[RequestSizeLimit(100000000)]
public IActionResult MyActionMethod()
```

<span data-ttu-id="13792-923">L'esempio seguente illustra come configurare il vincolo per l'app in ogni richiesta:</span><span class="sxs-lookup"><span data-stu-id="13792-923">Here's an example that shows how to configure the constraint for the app on every request:</span></span>

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .UseKestrel(serverOptions =>
        {
            serverOptions.Limits.MaxRequestBodySize = 10 * 1024;
        });
```

<span data-ttu-id="13792-924">Eseguire l'override dell'impostazione per una richiesta specifica nel middleware:</span><span class="sxs-lookup"><span data-stu-id="13792-924">Override the setting on a specific request in middleware:</span></span>

[!code-csharp[](kestrel/samples/2.x/KestrelSample/Startup.cs?name=snippet_Limits&highlight=3-4)]

<span data-ttu-id="13792-925">Viene generata un'eccezione se l'app configura il limite per una richiesta dopo che l'app ha iniziato a leggere la richiesta.</span><span class="sxs-lookup"><span data-stu-id="13792-925">An exception is thrown if the app configures the limit on a request after the app has started to read the request.</span></span> <span data-ttu-id="13792-926">Una proprietà `IsReadOnly` indica se la proprietà `MaxRequestBodySize` è in stato di sola lettura e pertanto è troppo tardi per configurare il limite.</span><span class="sxs-lookup"><span data-stu-id="13792-926">There's an `IsReadOnly` property that indicates if the `MaxRequestBodySize` property is in read-only state, meaning it's too late to configure the limit.</span></span>

<span data-ttu-id="13792-927">Quando un'app viene eseguita [out-of-process](xref:host-and-deploy/iis/index#out-of-process-hosting-model) dietro il [modulo ASP.NET Core](xref:host-and-deploy/aspnet-core-module), il limite della dimensione del corpo della richiesta di Kestrel è disabilitato perché viene già impostato da IIS.</span><span class="sxs-lookup"><span data-stu-id="13792-927">When an app is run [out-of-process](xref:host-and-deploy/iis/index#out-of-process-hosting-model) behind the [ASP.NET Core Module](xref:host-and-deploy/aspnet-core-module), Kestrel's request body size limit is disabled because IIS already sets the limit.</span></span>

### <a name="minimum-request-body-data-rate"></a><span data-ttu-id="13792-928">Velocità minima dei dati del corpo della richiesta</span><span class="sxs-lookup"><span data-stu-id="13792-928">Minimum request body data rate</span></span>

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.MinRequestBodyDataRate>
<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.MinResponseDataRate>

<span data-ttu-id="13792-929">Kestrel controlla ogni secondo se i dati arrivano alla velocità in byte al secondo specificata.</span><span class="sxs-lookup"><span data-stu-id="13792-929">Kestrel checks every second if data is arriving at the specified rate in bytes/second.</span></span> <span data-ttu-id="13792-930">Se la frequenza scende sotto il valore minimo, si è verificato il timeout della connessione. Il periodo di tolleranza è la quantità di tempo che il gheppio concede al client di aumentare la velocità di invio fino al minimo. la frequenza non viene controllata durante tale periodo di tempo.</span><span class="sxs-lookup"><span data-stu-id="13792-930">If the rate drops below the minimum, the connection is timed out. The grace period is the amount of time that Kestrel gives the client to increase its send rate up to the minimum; the rate isn't checked during that time.</span></span> <span data-ttu-id="13792-931">Il periodo di prova consente di evitare l'interruzione di connessioni che inizialmente inviano i dati a velocità ridotta a causa dell'avvio lento del protocollo TCP.</span><span class="sxs-lookup"><span data-stu-id="13792-931">The grace period helps avoid dropping connections that are initially sending data at a slow rate due to TCP slow-start.</span></span>

<span data-ttu-id="13792-932">La velocità minima predefinita è di 240 byte al secondo, con un periodo di tolleranza di 5 secondi.</span><span class="sxs-lookup"><span data-stu-id="13792-932">The default minimum rate is 240 bytes/second with a 5 second grace period.</span></span>

<span data-ttu-id="13792-933">Anche per la risposta è prevista una velocità minima.</span><span class="sxs-lookup"><span data-stu-id="13792-933">A minimum rate also applies to the response.</span></span> <span data-ttu-id="13792-934">Il codice per impostare il limite della richiesta e della risposta è identico e varia solo per `RequestBody` o `Response` nei nomi di proprietà e di interfaccia.</span><span class="sxs-lookup"><span data-stu-id="13792-934">The code to set the request limit and the response limit is the same except for having `RequestBody` or `Response` in the property and interface names.</span></span>

<span data-ttu-id="13792-935">L'esempio seguente visualizza come configurare la velocità minima dei dati in *Program.cs*:</span><span class="sxs-lookup"><span data-stu-id="13792-935">Here's an example that shows how to configure the minimum data rates in *Program.cs*:</span></span>

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .UseKestrel(serverOptions =>
        {
            serverOptions.Limits.MinRequestBodyDataRate =
                new MinDataRate(bytesPerSecond: 100, gracePeriod: TimeSpan.FromSeconds(10));
            serverOptions.Limits.MinResponseDataRate =
                new MinDataRate(bytesPerSecond: 100, gracePeriod: TimeSpan.FromSeconds(10));
        });
```

### <a name="request-headers-timeout"></a><span data-ttu-id="13792-936">Timeout delle intestazioni delle richieste</span><span class="sxs-lookup"><span data-stu-id="13792-936">Request headers timeout</span></span>

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.RequestHeadersTimeout>

<span data-ttu-id="13792-937">Ottiene o imposta la quantità massima di tempo che il server dedica alla ricezione delle intestazioni delle richieste.</span><span class="sxs-lookup"><span data-stu-id="13792-937">Gets or sets the maximum amount of time the server spends receiving request headers.</span></span> <span data-ttu-id="13792-938">Il valore predefinito è 30 secondi.</span><span class="sxs-lookup"><span data-stu-id="13792-938">Defaults to 30 seconds.</span></span>

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .UseKestrel(serverOptions =>
        {
            serverOptions.Limits.RequestHeadersTimeout = TimeSpan.FromMinutes(1);
        });
```

### <a name="synchronous-io"></a><span data-ttu-id="13792-939">I/O sincrono</span><span class="sxs-lookup"><span data-stu-id="13792-939">Synchronous I/O</span></span>

<span data-ttu-id="13792-940"><xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.AllowSynchronousIO> Controlla se è consentito l'I/O sincrono per la richiesta e la risposta.</span><span class="sxs-lookup"><span data-stu-id="13792-940"><xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.AllowSynchronousIO> controls whether synchronous I/O is allowed for the request and response.</span></span> <span data-ttu-id="13792-941">Il valore predefinito è `true`.</span><span class="sxs-lookup"><span data-stu-id="13792-941">The  default value is `true`.</span></span>

> [!WARNING]
> <span data-ttu-id="13792-942">Un numero elevato di operazioni di I/O sincrone bloccanti può causare l'esaurimento del pool di thread, il che rende l'app non risponde.</span><span class="sxs-lookup"><span data-stu-id="13792-942">A large number of blocking synchronous I/O operations can lead to thread pool starvation, which makes the app unresponsive.</span></span> <span data-ttu-id="13792-943">Abilitare solo `AllowSynchronousIO` quando si usa una libreria che non supporta l'I/O asincrono.</span><span class="sxs-lookup"><span data-stu-id="13792-943">Only enable `AllowSynchronousIO` when using a library that doesn't support asynchronous I/O.</span></span>

<span data-ttu-id="13792-944">Nell'esempio seguente viene disabilitato l'I/O sincrono:</span><span class="sxs-lookup"><span data-stu-id="13792-944">The following example disables synchronous I/O:</span></span>

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .UseKestrel(serverOptions =>
        {
            serverOptions.AllowSynchronousIO = false;
        });
```

<span data-ttu-id="13792-945">Per informazioni su altre opzioni e limiti di Kestrel, vedere:</span><span class="sxs-lookup"><span data-stu-id="13792-945">For information about other Kestrel options and limits, see:</span></span>

* <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions>
* <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits>
* <xref:Microsoft.AspNetCore.Server.Kestrel.Core.ListenOptions>

## <a name="endpoint-configuration"></a><span data-ttu-id="13792-946">Configurazione dell'endpoint</span><span class="sxs-lookup"><span data-stu-id="13792-946">Endpoint configuration</span></span>

<span data-ttu-id="13792-947">Per impostazione predefinita, ASP.NET Core è associato a:</span><span class="sxs-lookup"><span data-stu-id="13792-947">By default, ASP.NET Core binds to:</span></span>

* `http://localhost:5000`
* <span data-ttu-id="13792-948">`https://localhost:5001` (quando è presente un certificato di sviluppo locale)</span><span class="sxs-lookup"><span data-stu-id="13792-948">`https://localhost:5001` (when a local development certificate is present)</span></span>

<span data-ttu-id="13792-949">Specificare gli URL usando gli elementi seguenti:</span><span class="sxs-lookup"><span data-stu-id="13792-949">Specify URLs using the:</span></span>

* <span data-ttu-id="13792-950">La variabile di ambiente `ASPNETCORE_URLS`.</span><span class="sxs-lookup"><span data-stu-id="13792-950">`ASPNETCORE_URLS` environment variable.</span></span>
* <span data-ttu-id="13792-951">L'argomento della riga di comando `--urls`.</span><span class="sxs-lookup"><span data-stu-id="13792-951">`--urls` command-line argument.</span></span>
* <span data-ttu-id="13792-952">La chiave di configurazione dell'host `urls`.</span><span class="sxs-lookup"><span data-stu-id="13792-952">`urls` host configuration key.</span></span>
* <span data-ttu-id="13792-953">Il metodo di estensione `UseUrls`.</span><span class="sxs-lookup"><span data-stu-id="13792-953">`UseUrls` extension method.</span></span>

<span data-ttu-id="13792-954">Il valore specificato usando i metodi seguenti può essere uno o più endpoint HTTP e HTTPS (HTTPS se è disponibile un certificato predefinito).</span><span class="sxs-lookup"><span data-stu-id="13792-954">The value provided using these approaches can be one or more HTTP and HTTPS endpoints (HTTPS if a default cert is available).</span></span> <span data-ttu-id="13792-955">Configurare il valore come un elenco delimitato da punto e virgola (ad esempio, `"Urls": "http://localhost:8000;http://localhost:8001"`).</span><span class="sxs-lookup"><span data-stu-id="13792-955">Configure the value as a semicolon-separated list (for example, `"Urls": "http://localhost:8000;http://localhost:8001"`).</span></span>

<span data-ttu-id="13792-956">Per altre informazioni su questi approcci, vedere [URL del server](xref:fundamentals/host/web-host#server-urls) e [Override della configurazione](xref:fundamentals/host/web-host#override-configuration).</span><span class="sxs-lookup"><span data-stu-id="13792-956">For more information on these approaches, see [Server URLs](xref:fundamentals/host/web-host#server-urls) and [Override configuration](xref:fundamentals/host/web-host#override-configuration).</span></span>

<span data-ttu-id="13792-957">Viene creato un certificato di sviluppo nei casi seguenti:</span><span class="sxs-lookup"><span data-stu-id="13792-957">A development certificate is created:</span></span>

* <span data-ttu-id="13792-958">Quando viene installato [.NET Core SDK](/dotnet/core/sdk).</span><span class="sxs-lookup"><span data-stu-id="13792-958">When the [.NET Core SDK](/dotnet/core/sdk) is installed.</span></span>
* <span data-ttu-id="13792-959">Quando per creare un certificato viene usato [lo strumento dev-certs](xref:aspnetcore-2.1#https).</span><span class="sxs-lookup"><span data-stu-id="13792-959">The [dev-certs tool](xref:aspnetcore-2.1#https) is used to create a certificate.</span></span>

<span data-ttu-id="13792-960">Alcuni browser richiedono la concessione di autorizzazioni esplicite per considerare attendibile il certificato di sviluppo locale.</span><span class="sxs-lookup"><span data-stu-id="13792-960">Some browsers require granting explicit permission to trust the local development certificate.</span></span>

<span data-ttu-id="13792-961">I modelli di progetto consentono di configurare le app per l'esecuzione su HTTPS per impostazione predefinita e includono il [reindirizzamento HTTPS e il supporto HSTS](xref:security/enforcing-ssl).</span><span class="sxs-lookup"><span data-stu-id="13792-961">Project templates configure apps to run on HTTPS by default and include [HTTPS redirection and HSTS support](xref:security/enforcing-ssl).</span></span>

<span data-ttu-id="13792-962">Chiamare i metodi <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Listen*> oppure <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ListenUnixSocket*> su <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions> per configurare le porte e i prefissi URL per Kestrel.</span><span class="sxs-lookup"><span data-stu-id="13792-962">Call <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Listen*> or <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ListenUnixSocket*> methods on <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions> to configure URL prefixes and ports for Kestrel.</span></span>

<span data-ttu-id="13792-963">Anche `UseUrls`, l'argomento della riga di comando `--urls`, la chiave di configurazione dell'host `urls` e la variabile di ambiente `ASPNETCORE_URLS` funzionano, ma con le limitazioni indicate più avanti nella sezione (deve essere disponibile un certificato predefinito per la configurazione dell'endopoint HTTPS).</span><span class="sxs-lookup"><span data-stu-id="13792-963">`UseUrls`, the `--urls` command-line argument, `urls` host configuration key, and the `ASPNETCORE_URLS` environment variable also work but have the limitations noted later in this section (a default certificate must be available for HTTPS endpoint configuration).</span></span>

<span data-ttu-id="13792-964">`KestrelServerOptions` configurazione</span><span class="sxs-lookup"><span data-stu-id="13792-964">`KestrelServerOptions` configuration:</span></span>

### <a name="configureendpointdefaultsactionlistenoptions"></a><span data-ttu-id="13792-965">ConfigureEndpointDefaults (azione \<ListenOptions> )</span><span class="sxs-lookup"><span data-stu-id="13792-965">ConfigureEndpointDefaults(Action\<ListenOptions>)</span></span>

<span data-ttu-id="13792-966">Specifica un elemento di configurazione `Action` da eseguire per ogni endpoint specificato.</span><span class="sxs-lookup"><span data-stu-id="13792-966">Specifies a configuration `Action` to run for each specified endpoint.</span></span> <span data-ttu-id="13792-967">Se si chiama `ConfigureEndpointDefaults` più volte, gli elementi `Action` precedenti vengono sostituiti con l'ultimo elemento `Action` specificato.</span><span class="sxs-lookup"><span data-stu-id="13792-967">Calling `ConfigureEndpointDefaults` multiple times replaces prior `Action`s with the last `Action` specified.</span></span>

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .ConfigureKestrel((context, serverOptions) =>
        {
            serverOptions.ConfigureEndpointDefaults(listenOptions =>
            {
                // Configure endpoint defaults
            });
        });
```

> [!NOTE]
> <span data-ttu-id="13792-968">Per gli endpoint creati chiamando <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Listen*> **prima** della chiamata a <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ConfigureEndpointDefaults*> non verranno applicati i valori predefiniti.</span><span class="sxs-lookup"><span data-stu-id="13792-968">Endpoints created by calling <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Listen*> **before** calling <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ConfigureEndpointDefaults*> won't have the defaults applied.</span></span>

### <a name="configurehttpsdefaultsactionhttpsconnectionadapteroptions"></a><span data-ttu-id="13792-969">ConfigureHttpsDefaults (azione \<HttpsConnectionAdapterOptions> )</span><span class="sxs-lookup"><span data-stu-id="13792-969">ConfigureHttpsDefaults(Action\<HttpsConnectionAdapterOptions>)</span></span>

<span data-ttu-id="13792-970">Specifica un elemento di configurazione `Action` da eseguire per ogni endpoint HTTPS.</span><span class="sxs-lookup"><span data-stu-id="13792-970">Specifies a configuration `Action` to run for each HTTPS endpoint.</span></span> <span data-ttu-id="13792-971">Se si chiama `ConfigureHttpsDefaults` più volte, gli elementi `Action` precedenti vengono sostituiti con l'ultimo elemento `Action` specificato.</span><span class="sxs-lookup"><span data-stu-id="13792-971">Calling `ConfigureHttpsDefaults` multiple times replaces prior `Action`s with the last `Action` specified.</span></span>

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .ConfigureKestrel((context, serverOptions) =>
        {
            serverOptions.ConfigureHttpsDefaults(listenOptions =>
            {
                // certificate is an X509Certificate2
                listenOptions.ServerCertificate = certificate;
            });
        });
```

> [!NOTE]
> <span data-ttu-id="13792-972">Per gli endpoint creati chiamando <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Listen*> **prima** della chiamata a <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ConfigureHttpsDefaults*> non verranno applicati i valori predefiniti.</span><span class="sxs-lookup"><span data-stu-id="13792-972">Endpoints created by calling <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Listen*> **before** calling <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ConfigureHttpsDefaults*> won't have the defaults applied.</span></span>

### <a name="configureiconfiguration"></a><span data-ttu-id="13792-973">Configure(IConfiguration)</span><span class="sxs-lookup"><span data-stu-id="13792-973">Configure(IConfiguration)</span></span>

<span data-ttu-id="13792-974">Crea un loader di configurazione per la configurazione di Kestrel che accetta un elemento <xref:Microsoft.Extensions.Configuration.IConfiguration> come input.</span><span class="sxs-lookup"><span data-stu-id="13792-974">Creates a configuration loader for setting up Kestrel that takes an <xref:Microsoft.Extensions.Configuration.IConfiguration> as input.</span></span> <span data-ttu-id="13792-975">L'ambito della configurazione deve essere la sezione di configurazione per Kestrel.</span><span class="sxs-lookup"><span data-stu-id="13792-975">The configuration must be scoped to the configuration section for Kestrel.</span></span>

### <a name="listenoptionsusehttps"></a><span data-ttu-id="13792-976">ListenOptions.UseHttps</span><span class="sxs-lookup"><span data-stu-id="13792-976">ListenOptions.UseHttps</span></span>

<span data-ttu-id="13792-977">Configura Kestrel per l'uso di HTTPS.</span><span class="sxs-lookup"><span data-stu-id="13792-977">Configure Kestrel to use HTTPS.</span></span>

<span data-ttu-id="13792-978">Estensioni `ListenOptions.UseHttps`:</span><span class="sxs-lookup"><span data-stu-id="13792-978">`ListenOptions.UseHttps` extensions:</span></span>

* <span data-ttu-id="13792-979">`UseHttps`: Configurare gheppio per l'uso di HTTPS con il certificato predefinito.</span><span class="sxs-lookup"><span data-stu-id="13792-979">`UseHttps`: Configure Kestrel to use HTTPS with the default certificate.</span></span> <span data-ttu-id="13792-980">Genera un'eccezione se non è stato configurato alcun certificato predefinito.</span><span class="sxs-lookup"><span data-stu-id="13792-980">Throws an exception if no default certificate is configured.</span></span>
* `UseHttps(string fileName)`
* `UseHttps(string fileName, string password)`
* `UseHttps(string fileName, string password, Action<HttpsConnectionAdapterOptions> configureOptions)`
* `UseHttps(StoreName storeName, string subject)`
* `UseHttps(StoreName storeName, string subject, bool allowInvalid)`
* `UseHttps(StoreName storeName, string subject, bool allowInvalid, StoreLocation location)`
* `UseHttps(StoreName storeName, string subject, bool allowInvalid, StoreLocation location, Action<HttpsConnectionAdapterOptions> configureOptions)`
* `UseHttps(X509Certificate2 serverCertificate)`
* `UseHttps(X509Certificate2 serverCertificate, Action<HttpsConnectionAdapterOptions> configureOptions)`
* `UseHttps(Action<HttpsConnectionAdapterOptions> configureOptions)`

<span data-ttu-id="13792-981">Parametri `ListenOptions.UseHttps`:</span><span class="sxs-lookup"><span data-stu-id="13792-981">`ListenOptions.UseHttps` parameters:</span></span>

* <span data-ttu-id="13792-982">`filename` è il percorso e il nome file di un file di certificato, relativo alla directory che contiene i file di contenuto dell'app.</span><span class="sxs-lookup"><span data-stu-id="13792-982">`filename` is the path and file name of a certificate file, relative to the directory that contains the app's content files.</span></span>
* <span data-ttu-id="13792-983">`password` è la password richiesta per accedere ai dati del certificato X.509.</span><span class="sxs-lookup"><span data-stu-id="13792-983">`password` is the password required to access the X.509 certificate data.</span></span>
* <span data-ttu-id="13792-984">`configureOptions` è un elemento `Action` per configurare `HttpsConnectionAdapterOptions`.</span><span class="sxs-lookup"><span data-stu-id="13792-984">`configureOptions` is an `Action` to configure the `HttpsConnectionAdapterOptions`.</span></span> <span data-ttu-id="13792-985">Restituisce `ListenOptions`.</span><span class="sxs-lookup"><span data-stu-id="13792-985">Returns the `ListenOptions`.</span></span>
* <span data-ttu-id="13792-986">`storeName` è l'archivio certificati da cui caricare il certificato.</span><span class="sxs-lookup"><span data-stu-id="13792-986">`storeName` is the certificate store from which to load the certificate.</span></span>
* <span data-ttu-id="13792-987">`subject` è il nome dell'oggetto del certificato.</span><span class="sxs-lookup"><span data-stu-id="13792-987">`subject` is the subject name for the certificate.</span></span>
* <span data-ttu-id="13792-988">`allowInvalid` indica se devono essere considerati i certificati non validi, come ad esempio i certificati autofirmati.</span><span class="sxs-lookup"><span data-stu-id="13792-988">`allowInvalid` indicates if invalid certificates should be considered, such as self-signed certificates.</span></span>
* <span data-ttu-id="13792-989">`location` è il percorso dell'archivio da cui caricare il certificato.</span><span class="sxs-lookup"><span data-stu-id="13792-989">`location` is the store location to load the certificate from.</span></span>
* <span data-ttu-id="13792-990">`serverCertificate` è il certificato X.509.</span><span class="sxs-lookup"><span data-stu-id="13792-990">`serverCertificate` is the X.509 certificate.</span></span>

<span data-ttu-id="13792-991">Nell'ambiente di produzione, HTTPS deve essere configurato in modo esplicito.</span><span class="sxs-lookup"><span data-stu-id="13792-991">In production, HTTPS must be explicitly configured.</span></span> <span data-ttu-id="13792-992">Come minimo, è necessario specificare un certificato predefinito.</span><span class="sxs-lookup"><span data-stu-id="13792-992">At a minimum, a default certificate must be provided.</span></span>

<span data-ttu-id="13792-993">Configurazioni supportate descritte in seguito:</span><span class="sxs-lookup"><span data-stu-id="13792-993">Supported configurations described next:</span></span>

* <span data-ttu-id="13792-994">Nessuna configurazione</span><span class="sxs-lookup"><span data-stu-id="13792-994">No configuration</span></span>
* <span data-ttu-id="13792-995">Sostituire il certificato predefinito della configurazione</span><span class="sxs-lookup"><span data-stu-id="13792-995">Replace the default certificate from configuration</span></span>
* <span data-ttu-id="13792-996">Modificare le impostazioni predefinite nel codice</span><span class="sxs-lookup"><span data-stu-id="13792-996">Change the defaults in code</span></span>

<span data-ttu-id="13792-997">*Nessuna configurazione*</span><span class="sxs-lookup"><span data-stu-id="13792-997">*No configuration*</span></span>

<span data-ttu-id="13792-998">Kestrel è in ascolto su `http://localhost:5000` e `https://localhost:5001` (se è disponibile un certificato predefinito).</span><span class="sxs-lookup"><span data-stu-id="13792-998">Kestrel listens on `http://localhost:5000` and `https://localhost:5001` (if a default cert is available).</span></span>

<a name="configuration"></a>

<span data-ttu-id="13792-999">*Sostituire il certificato predefinito della configurazione*</span><span class="sxs-lookup"><span data-stu-id="13792-999">*Replace the default certificate from configuration*</span></span>

<span data-ttu-id="13792-1000">`CreateDefaultBuilder` chiama `Configure(context.Configuration.GetSection("Kestrel"))` per impostazione predefinita per caricare la configurazione di Kestrel.</span><span class="sxs-lookup"><span data-stu-id="13792-1000">`CreateDefaultBuilder` calls `Configure(context.Configuration.GetSection("Kestrel"))` by default to load Kestrel configuration.</span></span> <span data-ttu-id="13792-1001">È disponibile per Kestrel uno schema di configurazione delle impostazioni delle app HTTPS predefinito.</span><span class="sxs-lookup"><span data-stu-id="13792-1001">A default HTTPS app settings configuration schema is available for Kestrel.</span></span> <span data-ttu-id="13792-1002">Configurare più endpoint, inclusi gli URL e i certificati da usare, da un file su disco o da un archivio certificati.</span><span class="sxs-lookup"><span data-stu-id="13792-1002">Configure multiple endpoints, including the URLs and the certificates to use, either from a file on disk or from a certificate store.</span></span>

<span data-ttu-id="13792-1003">Nell'esempio seguente *appsettings.json* :</span><span class="sxs-lookup"><span data-stu-id="13792-1003">In the following *appsettings.json* example:</span></span>

* <span data-ttu-id="13792-1004">Impostare **AllowInvalid** su `true` per consentire l'uso di certificati non validi, come ad esempio i certificati autofirmati.</span><span class="sxs-lookup"><span data-stu-id="13792-1004">Set **AllowInvalid** to `true` to permit the use of invalid certificates (for example, self-signed certificates).</span></span>
* <span data-ttu-id="13792-1005">Tutti gli endpoint HTTPS che non specificano un certificato (**HttpsDefaultCert** nell'esempio che segue) usano il certificato definito in **Certificates** > **Default** o il certificato di sviluppo.</span><span class="sxs-lookup"><span data-stu-id="13792-1005">Any HTTPS endpoint that doesn't specify a certificate (**HttpsDefaultCert** in the example that follows) falls back to the cert defined under **Certificates** > **Default** or the development certificate.</span></span>

```json
{
  "Kestrel": {
    "Endpoints": {
      "Http": {
        "Url": "http://localhost:5000"
      },

      "HttpsInlineCertFile": {
        "Url": "https://localhost:5001",
        "Certificate": {
          "Path": "<path to .pfx file>",
          "Password": "<certificate password>"
        }
      },

      "HttpsInlineCertStore": {
        "Url": "https://localhost:5002",
        "Certificate": {
          "Subject": "<subject; required>",
          "Store": "<certificate store; required>",
          "Location": "<location; defaults to CurrentUser>",
          "AllowInvalid": "<true or false; defaults to false>"
        }
      },

      "HttpsDefaultCert": {
        "Url": "https://localhost:5003"
      },

      "Https": {
        "Url": "https://*:5004",
        "Certificate": {
          "Path": "<path to .pfx file>",
          "Password": "<certificate password>"
        }
      }
    },
    "Certificates": {
      "Default": {
        "Path": "<path to .pfx file>",
        "Password": "<certificate password>"
      }
    }
  }
}
```

<span data-ttu-id="13792-1006">Un'alternativa all'uso di **Path** e **Password** per qualsiasi nodo del certificato consiste nello specificare il certificato usando i campi dell'archivio certificati.</span><span class="sxs-lookup"><span data-stu-id="13792-1006">An alternative to using **Path** and **Password** for any certificate node is to specify the certificate using certificate store fields.</span></span> <span data-ttu-id="13792-1007">Ad esempio, il certificato predefinito **Certificates**  >   può essere specificato come:</span><span class="sxs-lookup"><span data-stu-id="13792-1007">For example, the **Certificates** > **Default** certificate can be specified as:</span></span>

```json
"Default": {
  "Subject": "<subject; required>",
  "Store": "<cert store; required>",
  "Location": "<location; defaults to CurrentUser>",
  "AllowInvalid": "<true or false; defaults to false>"
}
```

<span data-ttu-id="13792-1008">Note di schema:</span><span class="sxs-lookup"><span data-stu-id="13792-1008">Schema notes:</span></span>

* <span data-ttu-id="13792-1009">I nomi degli endpoint non applicano la distinzione tra maiuscole e minuscole.</span><span class="sxs-lookup"><span data-stu-id="13792-1009">Endpoints names are case-insensitive.</span></span> <span data-ttu-id="13792-1010">Ad esempio, sono validi sia `HTTPS` che `Https`.</span><span class="sxs-lookup"><span data-stu-id="13792-1010">For example, `HTTPS` and `Https` are valid.</span></span>
* <span data-ttu-id="13792-1011">Il parametro `Url` è obbligatorio per ogni endpoint.</span><span class="sxs-lookup"><span data-stu-id="13792-1011">The `Url` parameter is required for each endpoint.</span></span> <span data-ttu-id="13792-1012">Il formato per questo parametro è uguale a quello del parametro di configurazione di primo livello `Urls`, con la differenza che si limita a un singolo valore.</span><span class="sxs-lookup"><span data-stu-id="13792-1012">The format for this parameter is the same as the top-level `Urls` configuration parameter except that it's limited to a single value.</span></span>
* <span data-ttu-id="13792-1013">Questi endpoint sostituiscono quelli definiti nella configurazione di primo livello `Urls`, non vi si aggiungono.</span><span class="sxs-lookup"><span data-stu-id="13792-1013">These endpoints replace those defined in the top-level `Urls` configuration rather than adding to them.</span></span> <span data-ttu-id="13792-1014">Gli endpoint definiti nel codice tramite `Listen` si aggiungono agli endpoint definiti nella sezione di configurazione.</span><span class="sxs-lookup"><span data-stu-id="13792-1014">Endpoints defined in code via `Listen` are cumulative with the endpoints defined in the configuration section.</span></span>
* <span data-ttu-id="13792-1015">La sezione `Certificate` è facoltativa.</span><span class="sxs-lookup"><span data-stu-id="13792-1015">The `Certificate` section is optional.</span></span> <span data-ttu-id="13792-1016">Se la sezione `Certificate` non è specificata, vengono usati i valori predefiniti definiti negli scenari precedenti.</span><span class="sxs-lookup"><span data-stu-id="13792-1016">If the `Certificate` section isn't specified, the defaults defined in earlier scenarios are used.</span></span> <span data-ttu-id="13792-1017">Se non è disponibile alcun valore predefinito, il server genera un'eccezione e non viene avviato.</span><span class="sxs-lookup"><span data-stu-id="13792-1017">If no defaults are available, the server throws an exception and fails to start.</span></span>
* <span data-ttu-id="13792-1018">La `Certificate` sezione supporta sia la password del **percorso** &ndash;  che i certificati dell'archivio dell' **oggetto** &ndash;  .</span><span class="sxs-lookup"><span data-stu-id="13792-1018">The `Certificate` section supports both **Path**&ndash;**Password** and **Subject**&ndash;**Store** certificates.</span></span>
* <span data-ttu-id="13792-1019">In questo modo è possibile definire un numero qualsiasi di endpoint, purché non provochino conflitti di porte.</span><span class="sxs-lookup"><span data-stu-id="13792-1019">Any number of endpoints may be defined in this way so long as they don't cause port conflicts.</span></span>
* <span data-ttu-id="13792-1020">`options.Configure(context.Configuration.GetSection("{SECTION}"))` restituisce un oggetto `KestrelConfigurationLoader` con un metodo `.Endpoint(string name, listenOptions => { })` che può essere usato per integrare le impostazioni dell'endpoint configurato:</span><span class="sxs-lookup"><span data-stu-id="13792-1020">`options.Configure(context.Configuration.GetSection("{SECTION}"))` returns a `KestrelConfigurationLoader` with an `.Endpoint(string name, listenOptions => { })` method that can be used to supplement a configured endpoint's settings:</span></span>

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .UseKestrel((context, serverOptions) =>
        {
            serverOptions.Configure(context.Configuration.GetSection("Kestrel"))
                .Endpoint("HTTPS", listenOptions =>
                {
                    listenOptions.HttpsOptions.SslProtocols = SslProtocols.Tls12;
                });
        });
```

<span data-ttu-id="13792-1021">`KestrelServerOptions.ConfigurationLoader` è possibile accedere direttamente a per continuare a scorrere sul caricatore esistente, ad esempio quello fornito da <xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder*> .</span><span class="sxs-lookup"><span data-stu-id="13792-1021">`KestrelServerOptions.ConfigurationLoader` can be directly accessed to continue iterating on the existing loader, such as the one provided by <xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder*>.</span></span>

* <span data-ttu-id="13792-1022">La sezione di configurazione per ogni endpoint è disponibile nelle opzioni del `Endpoint` metodo in modo che sia possibile leggere le impostazioni personalizzate.</span><span class="sxs-lookup"><span data-stu-id="13792-1022">The configuration section for each endpoint is available on the options in the `Endpoint` method so that custom settings may be read.</span></span>
* <span data-ttu-id="13792-1023">È possibile caricare più configurazioni chiamando ancora `options.Configure(context.Configuration.GetSection("{SECTION}"))` con un'altra sezione.</span><span class="sxs-lookup"><span data-stu-id="13792-1023">Multiple configurations may be loaded by calling `options.Configure(context.Configuration.GetSection("{SECTION}"))` again with another section.</span></span> <span data-ttu-id="13792-1024">Viene usata solo l'ultima configurazione, a meno che `Load` sia stato chiamato esplicitamente in istanze precedenti.</span><span class="sxs-lookup"><span data-stu-id="13792-1024">Only the last configuration is used, unless `Load` is explicitly called on prior instances.</span></span> <span data-ttu-id="13792-1025">Il metapacchetto non chiama `Load` in modo che la sezione di configurazione predefinita venga sostituita.</span><span class="sxs-lookup"><span data-stu-id="13792-1025">The metapackage doesn't call `Load` so that its default configuration section may be replaced.</span></span>
* <span data-ttu-id="13792-1026">`KestrelConfigurationLoader` riflette la famiglia di API `Listen` da `KestrelServerOptions` all'overload di `Endpoint`, in modo che gli endpoint di codice e di configurazione possano essere configurati nella stessa posizione.</span><span class="sxs-lookup"><span data-stu-id="13792-1026">`KestrelConfigurationLoader` mirrors the `Listen` family of APIs from `KestrelServerOptions` as `Endpoint` overloads, so code and config endpoints may be configured in the same place.</span></span> <span data-ttu-id="13792-1027">Questi overload non usano nomi e usano solo le impostazioni predefinite della configurazione.</span><span class="sxs-lookup"><span data-stu-id="13792-1027">These overloads don't use names and only consume default settings from configuration.</span></span>

<span data-ttu-id="13792-1028">*Modificare le impostazioni predefinite nel codice*</span><span class="sxs-lookup"><span data-stu-id="13792-1028">*Change the defaults in code*</span></span>

<span data-ttu-id="13792-1029">`ConfigureEndpointDefaults` e `ConfigureHttpsDefaults` possono essere usati per modificare le impostazioni predefinite per `ListenOptions` e `HttpsConnectionAdapterOptions`, inclusa l'esecuzione dell'override del certificato predefinito specificato nello scenario precedente.</span><span class="sxs-lookup"><span data-stu-id="13792-1029">`ConfigureEndpointDefaults` and `ConfigureHttpsDefaults` can be used to change default settings for `ListenOptions` and `HttpsConnectionAdapterOptions`, including overriding the default certificate specified in the prior scenario.</span></span> <span data-ttu-id="13792-1030">`ConfigureEndpointDefaults` e `ConfigureHttpsDefaults` devono essere chiamati prima che venga configurato qualsiasi endpoint.</span><span class="sxs-lookup"><span data-stu-id="13792-1030">`ConfigureEndpointDefaults` and `ConfigureHttpsDefaults` should be called before any endpoints are configured.</span></span>

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .UseKestrel((context, serverOptions) =>
        {
            serverOptions.ConfigureEndpointDefaults(listenOptions =>
            {
                // Configure endpoint defaults
            });
            
            serverOptions.ConfigureHttpsDefaults(listenOptions =>
            {
                listenOptions.SslProtocols = SslProtocols.Tls12;
            });
        });
```

<span data-ttu-id="13792-1031">*Supporto kestrel per SNI*</span><span class="sxs-lookup"><span data-stu-id="13792-1031">*Kestrel support for SNI*</span></span>

<span data-ttu-id="13792-1032">[L'Indicazione nome server (SNI)](https://tools.ietf.org/html/rfc6066#section-3) può essere usata per ospitare più domini sulla stessa porta e indirizzo IP.</span><span class="sxs-lookup"><span data-stu-id="13792-1032">[Server Name Indication (SNI)](https://tools.ietf.org/html/rfc6066#section-3) can be used to host multiple domains on the same IP address and port.</span></span> <span data-ttu-id="13792-1033">Perché SNI funzioni è necessario che il client invii al server il nome host per la sessione protetta durante l'handshake TLS in modo che il server possa specificare il certificato corretto.</span><span class="sxs-lookup"><span data-stu-id="13792-1033">For SNI to function, the client sends the host name for the secure session to the server during the TLS handshake so that the server can provide the correct certificate.</span></span> <span data-ttu-id="13792-1034">Il client usa il certificato specificato per la comunicazione crittografata con il server durante la sessione protetta che segue l'handshake TLS.</span><span class="sxs-lookup"><span data-stu-id="13792-1034">The client uses the furnished certificate for encrypted communication with the server during the secure session that follows the TLS handshake.</span></span>

<span data-ttu-id="13792-1035">Kestrel supporta SNI tramite il callback `ServerCertificateSelector`.</span><span class="sxs-lookup"><span data-stu-id="13792-1035">Kestrel supports SNI via the `ServerCertificateSelector` callback.</span></span> <span data-ttu-id="13792-1036">Il callback viene richiamato una volta per ogni connessione per consentire all'app di controllare il nome host e selezionare il certificato appropriato.</span><span class="sxs-lookup"><span data-stu-id="13792-1036">The callback is invoked once per connection to allow the app to inspect the host name and select the appropriate certificate.</span></span>

<span data-ttu-id="13792-1037">Il supporto SNI richiede:</span><span class="sxs-lookup"><span data-stu-id="13792-1037">SNI support requires:</span></span>

* <span data-ttu-id="13792-1038">In esecuzione su Framework di destinazione `netcoreapp2.1` o versione successiva.</span><span class="sxs-lookup"><span data-stu-id="13792-1038">Running on target framework `netcoreapp2.1` or later.</span></span> <span data-ttu-id="13792-1039">In `net461` o versioni successive, il callback viene richiamato, ma `name` è sempre `null` .</span><span class="sxs-lookup"><span data-stu-id="13792-1039">On `net461` or later, the callback is invoked but the `name` is always `null`.</span></span> <span data-ttu-id="13792-1040">L'elemento `name` è `null` anche se il client non specifica il parametro del nome host nell'handshake TLS.</span><span class="sxs-lookup"><span data-stu-id="13792-1040">The `name` is also `null` if the client doesn't provide the host name parameter in the TLS handshake.</span></span>
* <span data-ttu-id="13792-1041">Esecuzione di tutti i siti Web nella stessa istanza di Kestrel.</span><span class="sxs-lookup"><span data-stu-id="13792-1041">All websites run on the same Kestrel instance.</span></span> <span data-ttu-id="13792-1042">Kestrel supporta la condivisione di un indirizzo IP e di una porta tra più istanze solo con un proxy inverso.</span><span class="sxs-lookup"><span data-stu-id="13792-1042">Kestrel doesn't support sharing an IP address and port across multiple instances without a reverse proxy.</span></span>

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .UseKestrel((context, serverOptions) =>
        {
            serverOptions.ListenAnyIP(5005, listenOptions =>
            {
                listenOptions.UseHttps(httpsOptions =>
                {
                    var localhostCert = CertificateLoader.LoadFromStoreCert(
                        "localhost", "My", StoreLocation.CurrentUser,
                        allowInvalid: true);
                    var exampleCert = CertificateLoader.LoadFromStoreCert(
                        "example.com", "My", StoreLocation.CurrentUser,
                        allowInvalid: true);
                    var subExampleCert = CertificateLoader.LoadFromStoreCert(
                        "sub.example.com", "My", StoreLocation.CurrentUser,
                        allowInvalid: true);
                    var certs = new Dictionary<string, X509Certificate2>(
                        StringComparer.OrdinalIgnoreCase);
                    certs["localhost"] = localhostCert;
                    certs["example.com"] = exampleCert;
                    certs["sub.example.com"] = subExampleCert;

                    httpsOptions.ServerCertificateSelector = (connectionContext, name) =>
                    {
                        if (name != null && certs.TryGetValue(name, out var cert))
                        {
                            return cert;
                        }

                        return exampleCert;
                    };
                });
            });
        })
        .Build();
```

### <a name="connection-logging"></a><span data-ttu-id="13792-1043">Registrazione connessione</span><span class="sxs-lookup"><span data-stu-id="13792-1043">Connection logging</span></span>

<span data-ttu-id="13792-1044">Chiamare <xref:Microsoft.AspNetCore.Hosting.ListenOptionsConnectionLoggingExtensions.UseConnectionLogging*> per creare log a livello di debug per la comunicazione a livello di byte in una connessione.</span><span class="sxs-lookup"><span data-stu-id="13792-1044">Call <xref:Microsoft.AspNetCore.Hosting.ListenOptionsConnectionLoggingExtensions.UseConnectionLogging*> to emit Debug level logs for byte-level communication on a connection.</span></span> <span data-ttu-id="13792-1045">La registrazione delle connessioni è utile per la risoluzione dei problemi nella comunicazione di basso livello, ad esempio durante la crittografia TLS e dietro i proxy.</span><span class="sxs-lookup"><span data-stu-id="13792-1045">Connection logging is helpful for troubleshooting problems in low-level communication, such as during TLS encryption and behind proxies.</span></span> <span data-ttu-id="13792-1046">Se `UseConnectionLogging` viene inserito prima `UseHttps` , viene registrato il traffico crittografato.</span><span class="sxs-lookup"><span data-stu-id="13792-1046">If `UseConnectionLogging` is placed before `UseHttps`, encrypted traffic is logged.</span></span> <span data-ttu-id="13792-1047">Se `UseConnectionLogging` viene inserito dopo `UseHttps` , viene registrato il traffico decrittografato.</span><span class="sxs-lookup"><span data-stu-id="13792-1047">If `UseConnectionLogging` is placed after `UseHttps`, decrypted traffic is logged.</span></span>

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.Listen(IPAddress.Any, 8000, listenOptions =>
    {
        listenOptions.UseConnectionLogging();
    });
});
```

### <a name="bind-to-a-tcp-socket"></a><span data-ttu-id="13792-1048">Associazione a un socket TCP</span><span class="sxs-lookup"><span data-stu-id="13792-1048">Bind to a TCP socket</span></span>

<span data-ttu-id="13792-1049">Il metodo <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Listen*> esegue l'associazione a un socket TCP e un'espressione lambda per le opzioni consente di configurare un certificato X.509:</span><span class="sxs-lookup"><span data-stu-id="13792-1049">The <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Listen*> method binds to a TCP socket, and an options lambda permits X.509 certificate configuration:</span></span>

```csharp
public static void Main(string[] args)
{
    CreateWebHostBuilder(args).Build().Run();
}

public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .UseKestrel(serverOptions =>
        {
            serverOptions.Listen(IPAddress.Loopback, 5000);
            serverOptions.Listen(IPAddress.Loopback, 5001, listenOptions =>
            {
                listenOptions.UseHttps("testCert.pfx", "testPassword");
            });
        });
```

```csharp
public static void Main(string[] args)
{
    CreateWebHostBuilder(args).Build().Run();
}

public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .UseKestrel(serverOptions =>
        {
            serverOptions.Listen(IPAddress.Loopback, 5000);
            serverOptions.Listen(IPAddress.Loopback, 5001, listenOptions =>
            {
                listenOptions.UseHttps("testCert.pfx", "testPassword");
            });
        });
```

<span data-ttu-id="13792-1050">L'esempio configura HTTPS per un endpoint con <xref:Microsoft.AspNetCore.Server.Kestrel.Core.ListenOptions>.</span><span class="sxs-lookup"><span data-stu-id="13792-1050">The example configures HTTPS for an endpoint with <xref:Microsoft.AspNetCore.Server.Kestrel.Core.ListenOptions>.</span></span> <span data-ttu-id="13792-1051">Usare la stessa API per configurare altre impostazioni di Kestrel per endpoint specifici.</span><span class="sxs-lookup"><span data-stu-id="13792-1051">Use the same API to configure other Kestrel settings for specific endpoints.</span></span>

[!INCLUDE [How to make an X.509 cert](~/includes/make-x509-cert.md)]

### <a name="bind-to-a-unix-socket"></a><span data-ttu-id="13792-1052">Associazione a un socket Unix</span><span class="sxs-lookup"><span data-stu-id="13792-1052">Bind to a Unix socket</span></span>

<span data-ttu-id="13792-1053">Impostare l'ascolto su un socket Unix con <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ListenUnixSocket*> per migliorare le prestazioni con Nginx, come visualizzato nell'esempio seguente:</span><span class="sxs-lookup"><span data-stu-id="13792-1053">Listen on a Unix socket with <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ListenUnixSocket*> for improved performance with Nginx, as shown in this example:</span></span>

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .UseKestrel(serverOptions =>
        {
            serverOptions.ListenUnixSocket("/tmp/kestrel-test.sock");
            serverOptions.ListenUnixSocket("/tmp/kestrel-test.sock", listenOptions =>
            {
                listenOptions.UseHttps("testCert.pfx", "testpassword");
            });
        });
```

* <span data-ttu-id="13792-1054">Nel file nginx confiuguration impostare la `server`  >  `location`  >  `proxy_pass` voce su `http://unix:/tmp/{KESTREL SOCKET}:/;` .</span><span class="sxs-lookup"><span data-stu-id="13792-1054">In the Nginx confiuguration file, set the `server` > `location` > `proxy_pass` entry to `http://unix:/tmp/{KESTREL SOCKET}:/;`.</span></span> <span data-ttu-id="13792-1055">`{KESTREL SOCKET}` nome del socket fornito a <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ListenUnixSocket*> (ad esempio, `kestrel-test.sock` nell'esempio precedente).</span><span class="sxs-lookup"><span data-stu-id="13792-1055">`{KESTREL SOCKET}` is the name of the socket provided to <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ListenUnixSocket*> (for example, `kestrel-test.sock` in the preceding example).</span></span>
* <span data-ttu-id="13792-1056">Verificare che il socket sia scrivibile da nginx (ad esempio, `chmod go+w /tmp/kestrel-test.sock` ).</span><span class="sxs-lookup"><span data-stu-id="13792-1056">Ensure that the socket is writeable by Nginx (for example, `chmod go+w /tmp/kestrel-test.sock`).</span></span> 

### <a name="port-0"></a><span data-ttu-id="13792-1057">Porta 0</span><span class="sxs-lookup"><span data-stu-id="13792-1057">Port 0</span></span>

<span data-ttu-id="13792-1058">Quando si specifica il numero di porta `0`, Kestrel esegue l'associazione dinamica a una porta disponibile.</span><span class="sxs-lookup"><span data-stu-id="13792-1058">When the port number `0` is specified, Kestrel dynamically binds to an available port.</span></span> <span data-ttu-id="13792-1059">L'esempio seguente indica come determinare la porta alla quale Kestrel ha eseguito l'associazione in fase di esecuzione:</span><span class="sxs-lookup"><span data-stu-id="13792-1059">The following example shows how to determine which port Kestrel actually bound at runtime:</span></span>

[!code-csharp[](kestrel/samples/2.x/KestrelSample/Startup.cs?name=snippet_Configure&highlight=3-4,15-21)]

<span data-ttu-id="13792-1060">Quando si esegue l'app, l'output della finestra della console indica la porta dinamica in cui l'app può essere raggiunta:</span><span class="sxs-lookup"><span data-stu-id="13792-1060">When the app is run, the console window output indicates the dynamic port where the app can be reached:</span></span>

```console
Listening on the following addresses: http://127.0.0.1:48508
```

### <a name="limitations"></a><span data-ttu-id="13792-1061">Limitazioni</span><span class="sxs-lookup"><span data-stu-id="13792-1061">Limitations</span></span>

<span data-ttu-id="13792-1062">Configurare gli endpoint con i metodi seguenti:</span><span class="sxs-lookup"><span data-stu-id="13792-1062">Configure endpoints with the following approaches:</span></span>

* <xref:Microsoft.AspNetCore.Hosting.HostingAbstractionsWebHostBuilderExtensions.UseUrls*>
* <span data-ttu-id="13792-1063">L'argomento della riga di comando `--urls`</span><span class="sxs-lookup"><span data-stu-id="13792-1063">`--urls` command-line argument</span></span>
* <span data-ttu-id="13792-1064">La chiave di configurazione dell'host `urls`</span><span class="sxs-lookup"><span data-stu-id="13792-1064">`urls` host configuration key</span></span>
* <span data-ttu-id="13792-1065">La variabile di ambiente `ASPNETCORE_URLS`</span><span class="sxs-lookup"><span data-stu-id="13792-1065">`ASPNETCORE_URLS` environment variable</span></span>

<span data-ttu-id="13792-1066">Questi metodi sono utili se si vuole che il codice funzioni con server diversi da Kestrel.</span><span class="sxs-lookup"><span data-stu-id="13792-1066">These methods are useful for making code work with servers other than Kestrel.</span></span> <span data-ttu-id="13792-1067">Tenere presenti, tuttavia, le limitazioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="13792-1067">However, be aware of the following limitations:</span></span>

* <span data-ttu-id="13792-1068">HTTPS non può essere usato con questi approcci, a meno che non venga specificato un certificato predefinito nella configurazione dell'endpoint HTTPS (ad esempio, usando la configurazione `KestrelServerOptions` o un file di configurazione come illustrato in precedenza in questo argomento).</span><span class="sxs-lookup"><span data-stu-id="13792-1068">HTTPS can't be used with these approaches unless a default certificate is provided in the HTTPS endpoint configuration (for example, using `KestrelServerOptions` configuration or a configuration file as shown earlier in this topic).</span></span>
* <span data-ttu-id="13792-1069">Quando sia l'approccio `Listen` che l'approccio `UseUrls` vengono usati contemporaneamente, gli endpoint `Listen` eseguono l'override degli endpoint `UseUrls`.</span><span class="sxs-lookup"><span data-stu-id="13792-1069">When both the `Listen` and `UseUrls` approaches are used simultaneously, the `Listen` endpoints override the `UseUrls` endpoints.</span></span>

### <a name="iis-endpoint-configuration"></a><span data-ttu-id="13792-1070">Configurazione dell'endpoint IIS</span><span class="sxs-lookup"><span data-stu-id="13792-1070">IIS endpoint configuration</span></span>

<span data-ttu-id="13792-1071">Quando si usa IIS, le associazioni di URL per le associazioni di override di IIS vengono impostate mediante `Listen` o `UseUrls`.</span><span class="sxs-lookup"><span data-stu-id="13792-1071">When using IIS, the URL bindings for IIS override bindings are set by either `Listen` or `UseUrls`.</span></span> <span data-ttu-id="13792-1072">Per altre informazioni, vedere l'articolo [Introduzione al modulo ASP.NET Core](xref:host-and-deploy/aspnet-core-module).</span><span class="sxs-lookup"><span data-stu-id="13792-1072">For more information, see the [ASP.NET Core Module](xref:host-and-deploy/aspnet-core-module) topic.</span></span>

## <a name="libuv-transport-configuration"></a><span data-ttu-id="13792-1073">Configurazione del trasporto libuv</span><span class="sxs-lookup"><span data-stu-id="13792-1073">Libuv transport configuration</span></span>

<span data-ttu-id="13792-1074">Con la versione ASP.NET Core 2.1, il trasporto predefinito di Kestrel non si basa più su Libuv, ma su socket gestiti.</span><span class="sxs-lookup"><span data-stu-id="13792-1074">With the release of ASP.NET Core 2.1, Kestrel's default transport is no longer based on Libuv but instead based on managed sockets.</span></span> <span data-ttu-id="13792-1075">Si tratta di una modifica importante per le app ASP.NET 2.0 Core che vengono aggiornate alla versione 2.1, che chiamano <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderLibuvExtensions.UseLibuv*> e dipendono da uno dei pacchetti seguenti:</span><span class="sxs-lookup"><span data-stu-id="13792-1075">This is a breaking change for ASP.NET Core 2.0 apps upgrading to 2.1 that call <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderLibuvExtensions.UseLibuv*> and depend on either of the following packages:</span></span>

* <span data-ttu-id="13792-1076">[Microsoft.AspNetCore.Server.Kestrel](https://www.nuget.org/packages/Microsoft.AspNetCore.Server.Kestrel/) (riferimento diretto al pacchetto)</span><span class="sxs-lookup"><span data-stu-id="13792-1076">[Microsoft.AspNetCore.Server.Kestrel](https://www.nuget.org/packages/Microsoft.AspNetCore.Server.Kestrel/) (direct package reference)</span></span>
* [<span data-ttu-id="13792-1077">Microsoft.AspNetCore.App</span><span class="sxs-lookup"><span data-stu-id="13792-1077">Microsoft.AspNetCore.App</span></span>](https://www.nuget.org/packages/Microsoft.AspNetCore.App/)

<span data-ttu-id="13792-1078">Per i progetti che richiedono l'uso di libuv:</span><span class="sxs-lookup"><span data-stu-id="13792-1078">For projects that require the use of Libuv:</span></span>

* <span data-ttu-id="13792-1079">Aggiungere una dipendenza per il pacchetto [Microsoft.AspNetCore.Server.Kestrel.Transport.Libuv](https://www.nuget.org/packages/Microsoft.AspNetCore.Server.Kestrel.Transport.Libuv/) al file di progetto dell'app:</span><span class="sxs-lookup"><span data-stu-id="13792-1079">Add a dependency for the [Microsoft.AspNetCore.Server.Kestrel.Transport.Libuv](https://www.nuget.org/packages/Microsoft.AspNetCore.Server.Kestrel.Transport.Libuv/) package to the app's project file:</span></span>

  ```xml
  <PackageReference Include="Microsoft.AspNetCore.Server.Kestrel.Transport.Libuv"
                    Version="{VERSION}" />
  ```

* <span data-ttu-id="13792-1080">Chiamare <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderLibuvExtensions.UseLibuv*>:</span><span class="sxs-lookup"><span data-stu-id="13792-1080">Call <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderLibuvExtensions.UseLibuv*>:</span></span>

  ```csharp
  public class Program
  {
      public static void Main(string[] args)
      {
          CreateWebHostBuilder(args).Build().Run();
      }

      public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
          WebHost.CreateDefaultBuilder(args)
              .UseLibuv()
              .UseStartup<Startup>();
  }
  ```

### <a name="url-prefixes"></a><span data-ttu-id="13792-1081">Prefissi URL</span><span class="sxs-lookup"><span data-stu-id="13792-1081">URL prefixes</span></span>

<span data-ttu-id="13792-1082">Se si usa `UseUrls`, l'argomento della riga di comando `--urls`, la chiave di configurazione dell'host `urls` o la variabile di ambiente `ASPNETCORE_URLS`, i prefissi URL possono avere uno dei formati seguenti.</span><span class="sxs-lookup"><span data-stu-id="13792-1082">When using `UseUrls`, `--urls` command-line argument, `urls` host configuration key, or `ASPNETCORE_URLS` environment variable, the URL prefixes can be in any of the following formats.</span></span>

<span data-ttu-id="13792-1083">Sono validi solo i prefissi URL HTTP.</span><span class="sxs-lookup"><span data-stu-id="13792-1083">Only HTTP URL prefixes are valid.</span></span> <span data-ttu-id="13792-1084">Kestrel non supporta HTTPS quando la configurazione delle associazioni URL viene eseguita con `UseUrls`.</span><span class="sxs-lookup"><span data-stu-id="13792-1084">Kestrel doesn't support HTTPS when configuring URL bindings using `UseUrls`.</span></span>

* <span data-ttu-id="13792-1085">Indirizzo IPv4 con numero di porta</span><span class="sxs-lookup"><span data-stu-id="13792-1085">IPv4 address with port number</span></span>

  ```
  http://65.55.39.10:80/
  ```

  <span data-ttu-id="13792-1086">`0.0.0.0` è un caso speciale che esegue l'associazione a tutti gli indirizzi IPv4.</span><span class="sxs-lookup"><span data-stu-id="13792-1086">`0.0.0.0` is a special case that binds to all IPv4 addresses.</span></span>

* <span data-ttu-id="13792-1087">Indirizzo IPv6 con numero di porta</span><span class="sxs-lookup"><span data-stu-id="13792-1087">IPv6 address with port number</span></span>

  ```
  http://[0:0:0:0:0:ffff:4137:270a]:80/
  ```

  <span data-ttu-id="13792-1088">`[::]` è l'equivalente IPv6 di `0.0.0.0` per IPv4.</span><span class="sxs-lookup"><span data-stu-id="13792-1088">`[::]` is the IPv6 equivalent of IPv4 `0.0.0.0`.</span></span>

* <span data-ttu-id="13792-1089">Nome host con numero di porta</span><span class="sxs-lookup"><span data-stu-id="13792-1089">Host name with port number</span></span>

  ```
  http://contoso.com:80/
  http://*:80/
  ```

  <span data-ttu-id="13792-1090">I nomi host, `*` e `+` non sono casi particolari.</span><span class="sxs-lookup"><span data-stu-id="13792-1090">Host names, `*`, and `+`, aren't special.</span></span> <span data-ttu-id="13792-1091">Tutto ciò che non è riconosciuto come un indirizzo IP o un elemento `localhost` valido esegue l'associazione a tutti gli IP IPv4 e IPv6.</span><span class="sxs-lookup"><span data-stu-id="13792-1091">Anything not recognized as a valid IP address or `localhost` binds to all IPv4 and IPv6 IPs.</span></span> <span data-ttu-id="13792-1092">Per associare nomi host diversi ad app ASP.NET Core diverse sulla stessa porta, usare [HTTP.sys](xref:fundamentals/servers/httpsys) o un server proxy inverso come IIS, Nginx o Apache.</span><span class="sxs-lookup"><span data-stu-id="13792-1092">To bind different host names to different ASP.NET Core apps on the same port, use [HTTP.sys](xref:fundamentals/servers/httpsys) or a reverse proxy server, such as IIS, Nginx, or Apache.</span></span>

  > [!WARNING]
  > <span data-ttu-id="13792-1093">Una configurazione che prevede un proxy inverso richiede il [filtro host](#host-filtering).</span><span class="sxs-lookup"><span data-stu-id="13792-1093">Hosting in a reverse proxy configuration requires [host filtering](#host-filtering).</span></span>

* <span data-ttu-id="13792-1094">Nome host `localhost` con numero di porta o IP di loopback con numero di porta</span><span class="sxs-lookup"><span data-stu-id="13792-1094">Host `localhost` name with port number or loopback IP with port number</span></span>

  ```
  http://localhost:5000/
  http://127.0.0.1:5000/
  http://[::1]:5000/
  ```

  <span data-ttu-id="13792-1095">Se è specificato `localhost`, Kestrel tenta l'associazione alle interfacce di loopback sia IPv4 che IPv6.</span><span class="sxs-lookup"><span data-stu-id="13792-1095">When `localhost` is specified, Kestrel attempts to bind to both IPv4 and IPv6 loopback interfaces.</span></span> <span data-ttu-id="13792-1096">Se la porta richiesta è in uso da parte di un altro servizio in una delle due interfacce di loopback, Kestrel non viene avviato.</span><span class="sxs-lookup"><span data-stu-id="13792-1096">If the requested port is in use by another service on either loopback interface, Kestrel fails to start.</span></span> <span data-ttu-id="13792-1097">Se una delle due interfacce di loopback non è disponibile per qualsiasi altro motivo (in genere perché IPv6 non è supportato), Kestrel registra un avviso.</span><span class="sxs-lookup"><span data-stu-id="13792-1097">If either loopback interface is unavailable for any other reason (most commonly because IPv6 isn't supported), Kestrel logs a warning.</span></span>

## <a name="host-filtering"></a><span data-ttu-id="13792-1098">Filtro host</span><span class="sxs-lookup"><span data-stu-id="13792-1098">Host filtering</span></span>

<span data-ttu-id="13792-1099">Mentre supporta la configurazione in base ai prefissi, ad esempio `http://example.com:5000`, Kestrel ignora quasi sempre il nome host.</span><span class="sxs-lookup"><span data-stu-id="13792-1099">While Kestrel supports configuration based on prefixes such as `http://example.com:5000`, Kestrel largely ignores the host name.</span></span> <span data-ttu-id="13792-1100">L'host `localhost` è un caso speciale usato per l'associazione agli indirizzi di loopback.</span><span class="sxs-lookup"><span data-stu-id="13792-1100">Host `localhost` is a special case used for binding to loopback addresses.</span></span> <span data-ttu-id="13792-1101">Qualsiasi host che non sia un indirizzo IP esplicito esegue l'associazione a tutti gli indirizzi IP pubblici.</span><span class="sxs-lookup"><span data-stu-id="13792-1101">Any host other than an explicit IP address binds to all public IP addresses.</span></span> <span data-ttu-id="13792-1102">Le intestazioni `Host` non vengono convalidate.</span><span class="sxs-lookup"><span data-stu-id="13792-1102">`Host` headers aren't validated.</span></span>

<span data-ttu-id="13792-1103">Come soluzione alternativa, usare il middleware di filtro host.</span><span class="sxs-lookup"><span data-stu-id="13792-1103">As a workaround, use Host Filtering Middleware.</span></span> <span data-ttu-id="13792-1104">Il middleware di filtro host viene fornito dal pacchetto [Microsoft. AspNetCore. HostFiltering](https://www.nuget.org/packages/Microsoft.AspNetCore.HostFiltering) , incluso nel [metapacchetto Microsoft. AspNetCore. App](xref:fundamentals/metapackage-app) (ASP.NET Core 2,1 o 2,2).</span><span class="sxs-lookup"><span data-stu-id="13792-1104">Host Filtering Middleware is provided by the [Microsoft.AspNetCore.HostFiltering](https://www.nuget.org/packages/Microsoft.AspNetCore.HostFiltering) package, which is included in the [Microsoft.AspNetCore.App metapackage](xref:fundamentals/metapackage-app) (ASP.NET Core 2.1 or 2.2).</span></span> <span data-ttu-id="13792-1105">Il middleware viene aggiunto da <xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder*>, che chiama <xref:Microsoft.AspNetCore.Builder.HostFilteringServicesExtensions.AddHostFiltering*>:</span><span class="sxs-lookup"><span data-stu-id="13792-1105">The middleware is added by <xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder*>, which calls <xref:Microsoft.AspNetCore.Builder.HostFilteringServicesExtensions.AddHostFiltering*>:</span></span>

[!code-csharp[](kestrel/samples-snapshot/2.x/KestrelSample/Program.cs?name=snippet_Program&highlight=9)]

<span data-ttu-id="13792-1106">Per impostazione predefinita, il middleware di filtro host è disabilitato per impostazione predefinita.</span><span class="sxs-lookup"><span data-stu-id="13792-1106">Host Filtering Middleware is disabled by default.</span></span> <span data-ttu-id="13792-1107">Per abilitare il middleware, definire una `AllowedHosts` chiave in *appsettings.json* / *appSettings. \<EnvironmentName> JSON*.</span><span class="sxs-lookup"><span data-stu-id="13792-1107">To enable the middleware, define an `AllowedHosts` key in *appsettings.json*/*appsettings.\<EnvironmentName>.json*.</span></span> <span data-ttu-id="13792-1108">Il valore è un elenco con valori delimitati da punto e virgola di nomi host senza numeri di porta:</span><span class="sxs-lookup"><span data-stu-id="13792-1108">The value is a semicolon-delimited list of host names without port numbers:</span></span>

<span data-ttu-id="13792-1109">*appsettings.json*:</span><span class="sxs-lookup"><span data-stu-id="13792-1109">*appsettings.json*:</span></span>

```json
{
  "AllowedHosts": "example.com;localhost"
}
```

> [!NOTE]
> <span data-ttu-id="13792-1110">Il [middleware delle intestazioni inoltrate](xref:host-and-deploy/proxy-load-balancer) include anche un'opzione <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.AllowedHosts>.</span><span class="sxs-lookup"><span data-stu-id="13792-1110">[Forwarded Headers Middleware](xref:host-and-deploy/proxy-load-balancer) also has an <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.AllowedHosts> option.</span></span> <span data-ttu-id="13792-1111">Il middleware di intestazioni inoltrate e il middleware di filtro host hanno funzionalità simili per diversi scenari.</span><span class="sxs-lookup"><span data-stu-id="13792-1111">Forwarded Headers Middleware and Host Filtering Middleware have similar functionality for different scenarios.</span></span> <span data-ttu-id="13792-1112">L'impostazione di `AllowedHosts` con il middleware delle intestazioni inoltrate è appropriato se l'intestazione `Host` non viene mantenuta durante l'inoltro delle richieste con un server proxy inverso o il bilanciamento del carico.</span><span class="sxs-lookup"><span data-stu-id="13792-1112">Setting `AllowedHosts` with Forwarded Headers Middleware is appropriate when the `Host` header isn't preserved while forwarding requests with a reverse proxy server or load balancer.</span></span> <span data-ttu-id="13792-1113">L'impostazione di `AllowedHosts` con il middleware del filtro host è appropriata se si usa Kestrel come server perimetrale rivolto al pubblico o se l'intestazione `Host` viene inoltrata direttamente.</span><span class="sxs-lookup"><span data-stu-id="13792-1113">Setting `AllowedHosts` with Host Filtering Middleware is appropriate when Kestrel is used as a public-facing edge server or when the `Host` header is directly forwarded.</span></span>
>
> <span data-ttu-id="13792-1114">Per altre informazioni sul middleware delle intestazioni inoltrate, vedere <xref:host-and-deploy/proxy-load-balancer>.</span><span class="sxs-lookup"><span data-stu-id="13792-1114">For more information on Forwarded Headers Middleware, see <xref:host-and-deploy/proxy-load-balancer>.</span></span>

## <a name="http11-request-draining"></a><span data-ttu-id="13792-1115">Svuotamento richieste HTTP/1.1</span><span class="sxs-lookup"><span data-stu-id="13792-1115">HTTP/1.1 request draining</span></span>

<span data-ttu-id="13792-1116">L'apertura di connessioni HTTP richiede molto tempo.</span><span class="sxs-lookup"><span data-stu-id="13792-1116">Opening HTTP connections is time consuming.</span></span> <span data-ttu-id="13792-1117">Per HTTPS, è anche a elevato utilizzo di risorse.</span><span class="sxs-lookup"><span data-stu-id="13792-1117">For HTTPS, it's also resource intensive.</span></span> <span data-ttu-id="13792-1118">Pertanto, gheppio tenta di riutilizzare le connessioni per il protocollo HTTP/1.1.</span><span class="sxs-lookup"><span data-stu-id="13792-1118">Therefore, Kestrel tries to reuse connections per the HTTP/1.1 protocol.</span></span> <span data-ttu-id="13792-1119">Il corpo della richiesta deve essere completamente utilizzato per consentire il riutilizzo della connessione.</span><span class="sxs-lookup"><span data-stu-id="13792-1119">A request body must be fully consumed to allow the connection to be reused.</span></span> <span data-ttu-id="13792-1120">L'app non usa sempre il corpo della richiesta, ad esempio le `POST` richieste in cui il server restituisce una risposta di reindirizzamento o 404.</span><span class="sxs-lookup"><span data-stu-id="13792-1120">The app doesn't always consume the request body, such as a `POST` requests where the server returns a redirect or 404 response.</span></span> <span data-ttu-id="13792-1121">Nel `POST` caso-Reindirizzamento:</span><span class="sxs-lookup"><span data-stu-id="13792-1121">In the `POST`-redirect case:</span></span>

* <span data-ttu-id="13792-1122">Il client potrebbe avere già inviato parte dei `POST` dati.</span><span class="sxs-lookup"><span data-stu-id="13792-1122">The client may already have sent part of the `POST` data.</span></span>
* <span data-ttu-id="13792-1123">Il server scrive la risposta 301.</span><span class="sxs-lookup"><span data-stu-id="13792-1123">The server writes the 301 response.</span></span>
* <span data-ttu-id="13792-1124">Non è possibile usare la connessione per una nuova richiesta fino a quando i `POST` dati del corpo della richiesta precedente non sono stati completamente letti.</span><span class="sxs-lookup"><span data-stu-id="13792-1124">The connection can't be used for a new request until the `POST` data from the previous request body has been fully read.</span></span>
* <span data-ttu-id="13792-1125">Gheppio tenta di svuotare il corpo della richiesta.</span><span class="sxs-lookup"><span data-stu-id="13792-1125">Kestrel tries to drain the request body.</span></span> <span data-ttu-id="13792-1126">Lo svuotamento del corpo della richiesta implica la lettura e l'eliminazione dei dati senza elaborarli.</span><span class="sxs-lookup"><span data-stu-id="13792-1126">Draining the request body means reading and discarding the data without processing it.</span></span>

<span data-ttu-id="13792-1127">Il processo di svuotamento crea un tradoff tra la possibilità di riutilizzare la connessione e il tempo necessario per svuotare i dati rimanenti:</span><span class="sxs-lookup"><span data-stu-id="13792-1127">The draining process makes a tradoff between allowing the connection to be reused and the time it takes to drain any remaining data:</span></span>

* <span data-ttu-id="13792-1128">Lo svuotamento ha un timeout di cinque secondi, che non è configurabile.</span><span class="sxs-lookup"><span data-stu-id="13792-1128">Draining has a timeout of five seconds, which isn't configurable.</span></span>
* <span data-ttu-id="13792-1129">Se tutti i dati specificati dall' `Content-Length` `Transfer-Encoding` intestazione o non sono stati letti prima del timeout, la connessione viene chiusa.</span><span class="sxs-lookup"><span data-stu-id="13792-1129">If all of the data specified by the `Content-Length` or `Transfer-Encoding` header hasn't been read before the timeout, the connection is closed.</span></span>

<span data-ttu-id="13792-1130">In alcuni casi può essere necessario terminare immediatamente la richiesta, prima o dopo la scrittura della risposta.</span><span class="sxs-lookup"><span data-stu-id="13792-1130">Sometimes you may want to terminate the request immediately, before or after writing the response.</span></span> <span data-ttu-id="13792-1131">È ad esempio possibile che i client dispongano di protezioni dati restrittive, pertanto la limitazione dei dati caricati potrebbe avere una priorità.</span><span class="sxs-lookup"><span data-stu-id="13792-1131">For example, clients may have restrictive data caps, so limiting uploaded data might be a priority.</span></span> <span data-ttu-id="13792-1132">In questi casi, per terminare una richiesta, chiamare [HttpContext. Abort](xref:Microsoft.AspNetCore.Http.HttpContext.Abort%2A) da un controller, una Razor pagina o un middleware.</span><span class="sxs-lookup"><span data-stu-id="13792-1132">In such cases to terminate a request, call [HttpContext.Abort](xref:Microsoft.AspNetCore.Http.HttpContext.Abort%2A) from a controller, Razor Page, or middleware.</span></span>

<span data-ttu-id="13792-1133">Ci sono alcune avvertenze per chiamare `Abort` :</span><span class="sxs-lookup"><span data-stu-id="13792-1133">There are caveats to calling `Abort`:</span></span>

* <span data-ttu-id="13792-1134">La creazione di nuove connessioni può essere lenta e dispendiosa.</span><span class="sxs-lookup"><span data-stu-id="13792-1134">Creating new connections can be slow and expensive.</span></span>
* <span data-ttu-id="13792-1135">Non vi è alcuna garanzia che il client abbia letto la risposta prima che la connessione venga chiusa.</span><span class="sxs-lookup"><span data-stu-id="13792-1135">There's no guarantee that the client has read the response before the connection closes.</span></span>
* <span data-ttu-id="13792-1136">`Abort`La chiamata a deve essere rara e riservata per i casi di errore gravi, non per errori comuni.</span><span class="sxs-lookup"><span data-stu-id="13792-1136">Calling `Abort` should be rare and reserved for severe error cases, not common errors.</span></span>
  * <span data-ttu-id="13792-1137">Chiamare solo `Abort` quando è necessario risolvere un problema specifico.</span><span class="sxs-lookup"><span data-stu-id="13792-1137">Only call `Abort` when a specific problem needs to be solved.</span></span> <span data-ttu-id="13792-1138">Ad esempio, chiamare `Abort` se i client malintenzionati tentano di eseguire `POST` dati o quando si verifica un bug nel codice client che genera richieste di grandi dimensioni o numerose.</span><span class="sxs-lookup"><span data-stu-id="13792-1138">For example, call `Abort` if malicious clients are trying to `POST` data or when there's a bug in client code that causes large or numerous requests.</span></span>
  * <span data-ttu-id="13792-1139">Non chiamare le `Abort` situazioni di errore comuni, ad esempio HTTP 404 (non trovato).</span><span class="sxs-lookup"><span data-stu-id="13792-1139">Don't call `Abort` for common error situations, such as HTTP 404 (Not Found).</span></span>

<span data-ttu-id="13792-1140">Quando si chiama [HttpResponse. CompleteAsync](xref:Microsoft.AspNetCore.Http.HttpResponse.CompleteAsync%2A) , prima `Abort` di chiamare si garantisce che il server abbia completato la scrittura della risposta.</span><span class="sxs-lookup"><span data-stu-id="13792-1140">Calling [HttpResponse.CompleteAsync](xref:Microsoft.AspNetCore.Http.HttpResponse.CompleteAsync%2A) before calling `Abort` ensures that the server has completed writing the response.</span></span> <span data-ttu-id="13792-1141">Tuttavia, il comportamento del client non è prevedibile e potrebbe non essere in grado di leggere la risposta prima che la connessione venga interrotta.</span><span class="sxs-lookup"><span data-stu-id="13792-1141">However, client behavior isn't predictable and they may not read the response before the connection is aborted.</span></span>

<span data-ttu-id="13792-1142">Questo processo è diverso per HTTP/2 poiché il protocollo supporta l'interruzione dei singoli flussi di richiesta senza chiudere la connessione.</span><span class="sxs-lookup"><span data-stu-id="13792-1142">This process is different for HTTP/2 because the protocol supports aborting individual request streams without closing the connection.</span></span> <span data-ttu-id="13792-1143">Il timeout di svuotamento di cinque secondi non è applicabile.</span><span class="sxs-lookup"><span data-stu-id="13792-1143">The five second drain timeout doesn't apply.</span></span> <span data-ttu-id="13792-1144">Se dopo il completamento di una risposta sono presenti dati del corpo della richiesta non letti, il server invia un frame RST HTTP/2.</span><span class="sxs-lookup"><span data-stu-id="13792-1144">If there's any unread request body data after completing a response, then the server sends an HTTP/2 RST frame.</span></span> <span data-ttu-id="13792-1145">I frame di dati del corpo della richiesta aggiuntivi vengono ignorati.</span><span class="sxs-lookup"><span data-stu-id="13792-1145">Additional request body data frames are ignored.</span></span>

<span data-ttu-id="13792-1146">Se possibile, è preferibile che i client utilizzino l'intestazione della richiesta [expect: 100-continue](https://developer.mozilla.org/docs/Web/HTTP/Status/100) e attendano la risposta del server prima di iniziare a inviare il corpo della richiesta.</span><span class="sxs-lookup"><span data-stu-id="13792-1146">If possible, it's better for clients to utilize the [Expect: 100-continue](https://developer.mozilla.org/docs/Web/HTTP/Status/100) request header and wait for the server to respond before starting to send the request body.</span></span> <span data-ttu-id="13792-1147">Che fornisce al client la possibilità di esaminare la risposta e interrompere prima di inviare dati non necessari.</span><span class="sxs-lookup"><span data-stu-id="13792-1147">That gives the client an opportunity to examine the response and abort before sending unneeded data.</span></span>

## <a name="additional-resources"></a><span data-ttu-id="13792-1148">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="13792-1148">Additional resources</span></span>

* <span data-ttu-id="13792-1149">Quando si usano i socket UNIX in Linux, il socket non viene eliminato automaticamente all'arresto dell'app.</span><span class="sxs-lookup"><span data-stu-id="13792-1149">When using UNIX sockets on Linux, the socket is not automatically deleted on app shut down.</span></span> <span data-ttu-id="13792-1150">Per altre informazioni, vedere [questo problema di GitHub](https://github.com/dotnet/aspnetcore/issues/14134).</span><span class="sxs-lookup"><span data-stu-id="13792-1150">For more information, see [this GitHub issue](https://github.com/dotnet/aspnetcore/issues/14134).</span></span>
* <xref:test/troubleshoot>
* <xref:security/enforcing-ssl>
* <xref:host-and-deploy/proxy-load-balancer>
* <span data-ttu-id="13792-1151">[RFC 7230: Message Syntax and Routing (Section 5.4: Host)](https://tools.ietf.org/html/rfc7230#section-5.4) (RFC 7230: Sintassi e routing dei messaggi (sezione 5.4: Host))</span><span class="sxs-lookup"><span data-stu-id="13792-1151">[RFC 7230: Message Syntax and Routing (Section 5.4: Host)](https://tools.ietf.org/html/rfc7230#section-5.4)</span></span>

::: moniker-end
