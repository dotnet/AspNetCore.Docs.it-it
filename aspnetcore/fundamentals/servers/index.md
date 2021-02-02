---
title: Implementazioni di server Web in ASP.NET Core
author: rick-anderson
description: Individuare i server Web Kestrel e HTTP.sys per ASP.NET Core. Informazioni su come scegliere un server e quando usare un server proxy inverso.
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 11/07/2019
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
uid: fundamentals/servers/index
ms.openlocfilehash: a49388c73f2ec0ea03b35dbba1575ce70a4e1512
ms.sourcegitcommit: 75db2f684a9302b0be7925eab586aa091c6bd19f
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/01/2021
ms.locfileid: "99238327"
---
# <a name="web-server-implementations-in-aspnet-core"></a><span data-ttu-id="e702c-104">Implementazioni di server Web in ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="e702c-104">Web server implementations in ASP.NET Core</span></span>

<span data-ttu-id="e702c-105">Di [Tom Dykstra](https://github.com/tdykstra), [Steve Smith](https://ardalis.com/), [Stephen Halter](https://twitter.com/halter73) e [Chris Ross](https://github.com/Tratcher)</span><span class="sxs-lookup"><span data-stu-id="e702c-105">By [Tom Dykstra](https://github.com/tdykstra), [Steve Smith](https://ardalis.com/), [Stephen Halter](https://twitter.com/halter73), and [Chris Ross](https://github.com/Tratcher)</span></span>

<span data-ttu-id="e702c-106">Un'app ASP.NET Core viene eseguita con un'implementazione del server HTTP in-process.</span><span class="sxs-lookup"><span data-stu-id="e702c-106">An ASP.NET Core app runs with an in-process HTTP server implementation.</span></span> <span data-ttu-id="e702c-107">L'implementazione del server attende le richieste HTTP e le rende visibili all'app come un set di [funzionalità di richiesta](xref:fundamentals/request-features) combinate in un <xref:Microsoft.AspNetCore.Http.HttpContext>.</span><span class="sxs-lookup"><span data-stu-id="e702c-107">The server implementation listens for HTTP requests and surfaces them to the app as a set of [request features](xref:fundamentals/request-features) composed into an <xref:Microsoft.AspNetCore.Http.HttpContext>.</span></span>

::: moniker range=">= aspnetcore-2.2"

# <a name="windows"></a>[<span data-ttu-id="e702c-108">Windows</span><span class="sxs-lookup"><span data-stu-id="e702c-108">Windows</span></span>](#tab/windows)

<span data-ttu-id="e702c-109">ASP.NET Core include quanto segue:</span><span class="sxs-lookup"><span data-stu-id="e702c-109">ASP.NET Core ships with the following:</span></span>

* <span data-ttu-id="e702c-110">Il [server Kestrel](xref:fundamentals/servers/kestrel) è l'implementazione di server HTTP multipiattaforma predefinita.</span><span class="sxs-lookup"><span data-stu-id="e702c-110">[Kestrel server](xref:fundamentals/servers/kestrel) is the default, cross-platform HTTP server implementation.</span></span> <span data-ttu-id="e702c-111">Gheppio fornisce le prestazioni migliori e l'utilizzo della memoria, ma non alcune delle funzionalità avanzate di HTTP.sys.</span><span class="sxs-lookup"><span data-stu-id="e702c-111">Kestrel provides the best performance and memory utilization, but it doesn't have some of the advanced features in HTTP.sys.</span></span> <span data-ttu-id="e702c-112">Per ulteriori informazioni, vedere [gheppio rispetto a HTTP.sys](#korh) nella sezione successiva.</span><span class="sxs-lookup"><span data-stu-id="e702c-112">For more information, see [Kestrel vs. HTTP.sys](#korh) in the next section.</span></span>
* <span data-ttu-id="e702c-113">Il server HTTP IIS è un [server in-process](#hosting-models) per IIS.</span><span class="sxs-lookup"><span data-stu-id="e702c-113">IIS HTTP Server is an [in-process server](#hosting-models) for IIS.</span></span>
* <span data-ttu-id="e702c-114">Il [server HTTP.sys](xref:fundamentals/servers/httpsys) è un server HTTP solo per Windows basato sul [driver del kernel HTTP.sys e l'API HTTP Server](/windows/desktop/Http/http-api-start-page).</span><span class="sxs-lookup"><span data-stu-id="e702c-114">[HTTP.sys server](xref:fundamentals/servers/httpsys) is a Windows-only HTTP server based on the [HTTP.sys kernel driver and HTTP Server API](/windows/desktop/Http/http-api-start-page).</span></span>

<span data-ttu-id="e702c-115">Quando si usa [IIS](/iis/get-started/introduction-to-iis/introduction-to-iis-architecture) oppure [IIS Express](/iis/extensions/introduction-to-iis-express/iis-express-overview), l'app viene eseguita:</span><span class="sxs-lookup"><span data-stu-id="e702c-115">When using [IIS](/iis/get-started/introduction-to-iis/introduction-to-iis-architecture) or [IIS Express](/iis/extensions/introduction-to-iis-express/iis-express-overview), the app either runs:</span></span>

* <span data-ttu-id="e702c-116">Nello stesso processo del processo di lavoro IIS (il [modello di hosting in-process](#hosting-models)) con il server http IIS.</span><span class="sxs-lookup"><span data-stu-id="e702c-116">In the same process as the IIS worker process (the [in-process hosting model](#hosting-models)) with the IIS HTTP Server.</span></span> <span data-ttu-id="e702c-117">*In-process* è la configurazione consigliata.</span><span class="sxs-lookup"><span data-stu-id="e702c-117">*In-process* is the recommended configuration.</span></span>
* <span data-ttu-id="e702c-118">In un processo separato dal processo di lavoro IIS ([modello di hosting out-of-process](#hosting-models)) con il [server Kestrel](#kestrel).</span><span class="sxs-lookup"><span data-stu-id="e702c-118">In a process separate from the IIS worker process (the [out-of-process hosting model](#hosting-models)) with the [Kestrel server](#kestrel).</span></span>

<span data-ttu-id="e702c-119">Il [modulo ASP.NET Core](xref:host-and-deploy/aspnet-core-module) è un modulo IIS nativo che gestisce le richieste IIS native tra IIS e il server HTTP IIS in-process o il server Kestrel.</span><span class="sxs-lookup"><span data-stu-id="e702c-119">The [ASP.NET Core Module](xref:host-and-deploy/aspnet-core-module) is a native IIS module that handles native IIS requests between IIS and the in-process IIS HTTP Server or Kestrel.</span></span> <span data-ttu-id="e702c-120">Per altre informazioni, vedere <xref:host-and-deploy/aspnet-core-module>.</span><span class="sxs-lookup"><span data-stu-id="e702c-120">For more information, see <xref:host-and-deploy/aspnet-core-module>.</span></span>

<a name="korh"></a>

## <a name="kestrel-vs-httpsys"></a><span data-ttu-id="e702c-121">Gheppio rispetto a HTTP.sys</span><span class="sxs-lookup"><span data-stu-id="e702c-121">Kestrel vs. HTTP.sys</span></span>

<span data-ttu-id="e702c-122">Il gheppio presenta i vantaggi seguenti rispetto a HTTP.sys:</span><span class="sxs-lookup"><span data-stu-id="e702c-122">Kestrel has the following advantages over HTTP.sys:</span></span>

  * <span data-ttu-id="e702c-123">Miglioramento delle prestazioni e dell'utilizzo della memoria.</span><span class="sxs-lookup"><span data-stu-id="e702c-123">Better performance and memory utilization.</span></span>
  * <span data-ttu-id="e702c-124">Multipiattaforma</span><span class="sxs-lookup"><span data-stu-id="e702c-124">Cross platform</span></span>
  * <span data-ttu-id="e702c-125">Agilità, sviluppato e con patch indipendente dal sistema operativo.</span><span class="sxs-lookup"><span data-stu-id="e702c-125">Agility, it's developed and patched independent of the OS.</span></span>
  * <span data-ttu-id="e702c-126">Configurazione di porta e TLS a livello di codice</span><span class="sxs-lookup"><span data-stu-id="e702c-126">Programmatic port and TLS configuration</span></span>
  * <span data-ttu-id="e702c-127">Estensibilità che consente protocolli quali [PPv2](https://github.com/aspnet/AspLabs/blob/master/src/ProxyProtocol/ProxyProtocol.Sample/ProxyProtocol.cs) e trasporti alternativi.</span><span class="sxs-lookup"><span data-stu-id="e702c-127">Extensibility that allows for protocols like [PPv2](https://github.com/aspnet/AspLabs/blob/master/src/ProxyProtocol/ProxyProtocol.Sample/ProxyProtocol.cs) and alternate transports.</span></span>

<span data-ttu-id="e702c-128">Http.Sys funziona come un componente in modalità kernel condiviso con le seguenti funzionalità non disponibili in gheppio:</span><span class="sxs-lookup"><span data-stu-id="e702c-128">Http.Sys operates as a shared kernel mode component with the following features that kestrel does not have:</span></span>

  * <span data-ttu-id="e702c-129">Condivisione delle porte</span><span class="sxs-lookup"><span data-stu-id="e702c-129">Port sharing</span></span>
  * <span data-ttu-id="e702c-130">Autenticazione di Windows in modalità kernel.</span><span class="sxs-lookup"><span data-stu-id="e702c-130">Kernel mode windows authentication.</span></span> <span data-ttu-id="e702c-131">[Gheppio supporta solo l'autenticazione in modalità utente](xref:security/authentication/windowsauth#kestrel).</span><span class="sxs-lookup"><span data-stu-id="e702c-131">[Kestrel supports only user-mode authentication](xref:security/authentication/windowsauth#kestrel).</span></span>
  * <span data-ttu-id="e702c-132">Inoltro rapido tramite trasferimenti di coda</span><span class="sxs-lookup"><span data-stu-id="e702c-132">Fast proxying via queue transfers</span></span>
  * <span data-ttu-id="e702c-133">Trasmissione diretta dei file</span><span class="sxs-lookup"><span data-stu-id="e702c-133">Direct file transmission</span></span>
  * <span data-ttu-id="e702c-134">Memorizzazione nella cache delle risposte</span><span class="sxs-lookup"><span data-stu-id="e702c-134">Response caching</span></span>

## <a name="hosting-models"></a><span data-ttu-id="e702c-135">Modelli di hosting</span><span class="sxs-lookup"><span data-stu-id="e702c-135">Hosting models</span></span>

<span data-ttu-id="e702c-136">Se si usa l'hosting in-process, un'app ASP.NET Core esegue lo stesso processo del processo di lavoro IIS.</span><span class="sxs-lookup"><span data-stu-id="e702c-136">Using in-process hosting, an ASP.NET Core app runs in the same process as its IIS worker process.</span></span> <span data-ttu-id="e702c-137">L'hosting in-process offre prestazioni migliori rispetto all'hosting out-of-process perché le richieste non vengono inserite in proxy sulla scheda di loopback, un'interfaccia di rete che restituisce il traffico di rete in uscita allo stesso computer.</span><span class="sxs-lookup"><span data-stu-id="e702c-137">In-process hosting provides improved performance over out-of-process hosting because requests aren't proxied over the loopback adapter, a network interface that returns outgoing network traffic back to the same machine.</span></span> <span data-ttu-id="e702c-138">Per gestire il processo, IIS usa il [servizio Attivazione processo Windows (WAS)](/iis/manage/provisioning-and-managing-iis/features-of-the-windows-process-activation-service-was).</span><span class="sxs-lookup"><span data-stu-id="e702c-138">IIS handles process management with the [Windows Process Activation Service (WAS)](/iis/manage/provisioning-and-managing-iis/features-of-the-windows-process-activation-service-was).</span></span>

<span data-ttu-id="e702c-139">Quando si usa l'hosting out-of-process, le app ASP.NET Core vengono eseguite in un processo distinto dal processo di lavoro IIS e il modulo esegue la gestione dei processi.</span><span class="sxs-lookup"><span data-stu-id="e702c-139">Using out-of-process hosting, ASP.NET Core apps run in a process separate from the IIS worker process, and the module handles process management.</span></span> <span data-ttu-id="e702c-140">Il modulo avvia il processo per l'app ASP.NET Core quando arriva la prima richiesta e riavvia l'app se viene arrestata o si arresta in modo anomalo.</span><span class="sxs-lookup"><span data-stu-id="e702c-140">The module starts the process for the ASP.NET Core app when the first request arrives and restarts the app if it shuts down or crashes.</span></span> <span data-ttu-id="e702c-141">Si tratta essenzialmente dello stesso comportamento delle app eseguite in-process e gestite dal [servizio Attivazione processo Windows](/iis/manage/provisioning-and-managing-iis/features-of-the-windows-process-activation-service-was).</span><span class="sxs-lookup"><span data-stu-id="e702c-141">This is essentially the same behavior as seen with apps that run in-process that are managed by the [Windows Process Activation Service (WAS)](/iis/manage/provisioning-and-managing-iis/features-of-the-windows-process-activation-service-was).</span></span>

<span data-ttu-id="e702c-142">Per altre informazioni e indicazioni per la configurazione, vedere gli argomenti seguenti:</span><span class="sxs-lookup"><span data-stu-id="e702c-142">For more information and configuration guidance, see the following topics:</span></span>

* <xref:host-and-deploy/iis/index>
* <xref:host-and-deploy/aspnet-core-module>

# <a name="macos"></a>[<span data-ttu-id="e702c-143">macOS</span><span class="sxs-lookup"><span data-stu-id="e702c-143">macOS</span></span>](#tab/macos)

<span data-ttu-id="e702c-144">ASP.NET Core viene fornito con il [server Kestrel](xref:fundamentals/servers/kestrel), ovvero il server HTTP multipiattaforma predefinito.</span><span class="sxs-lookup"><span data-stu-id="e702c-144">ASP.NET Core ships with [Kestrel server](xref:fundamentals/servers/kestrel), which is the default, cross-platform HTTP server.</span></span>

# <a name="linux"></a>[<span data-ttu-id="e702c-145">Linux</span><span class="sxs-lookup"><span data-stu-id="e702c-145">Linux</span></span>](#tab/linux)

<span data-ttu-id="e702c-146">ASP.NET Core viene fornito con il [server Kestrel](xref:fundamentals/servers/kestrel), ovvero il server HTTP multipiattaforma predefinito.</span><span class="sxs-lookup"><span data-stu-id="e702c-146">ASP.NET Core ships with [Kestrel server](xref:fundamentals/servers/kestrel), which is the default, cross-platform HTTP server.</span></span>

---

::: moniker-end

## <a name="kestrel"></a><span data-ttu-id="e702c-147">Kestrel</span><span class="sxs-lookup"><span data-stu-id="e702c-147">Kestrel</span></span>

 <span data-ttu-id="e702c-148">Il [server Kestrel](xref:fundamentals/servers/kestrel) è l'implementazione di server HTTP multipiattaforma predefinita.</span><span class="sxs-lookup"><span data-stu-id="e702c-148">[Kestrel server](xref:fundamentals/servers/kestrel) is the default, cross-platform HTTP server implementation.</span></span> <span data-ttu-id="e702c-149">Gheppio fornisce le prestazioni migliori e l'utilizzo della memoria, ma non alcune delle funzionalità avanzate di HTTP.sys.</span><span class="sxs-lookup"><span data-stu-id="e702c-149">Kestrel provides the best performance and memory utilization, but it doesn't have some of the advanced features in HTTP.sys.</span></span> <span data-ttu-id="e702c-150">Per ulteriori informazioni, vedere [gheppio rispetto a HTTP.sys](#korh) in questo documento.</span><span class="sxs-lookup"><span data-stu-id="e702c-150">For more information, see [Kestrel vs. HTTP.sys](#korh) in this document.</span></span>

<span data-ttu-id="e702c-151">Usare Kestrel:</span><span class="sxs-lookup"><span data-stu-id="e702c-151">Use Kestrel:</span></span>

* <span data-ttu-id="e702c-152">Da solo come server perimetrale che elabora le richieste direttamente da una rete, inclusa Internet.</span><span class="sxs-lookup"><span data-stu-id="e702c-152">By itself as an edge server processing requests directly from a network, including the Internet.</span></span>

  ![Kestrel comunica direttamente con Internet senza un server proxy inverso](kestrel/_static/kestrel-to-internet2.png)

* <span data-ttu-id="e702c-154">Con un *server proxy inverso*, ad esempio [Internet Information Services (IIS)](https://www.iis.net/), [Nginx](https://nginx.org) o [Apache](https://httpd.apache.org/).</span><span class="sxs-lookup"><span data-stu-id="e702c-154">With a *reverse proxy server*, such as [Internet Information Services (IIS)](https://www.iis.net/), [Nginx](https://nginx.org), or [Apache](https://httpd.apache.org/).</span></span> <span data-ttu-id="e702c-155">Il server proxy inverso riceve le richieste HTTP da Internet e le inoltra a Kestrel.</span><span class="sxs-lookup"><span data-stu-id="e702c-155">A reverse proxy server receives HTTP requests from the Internet and forwards them to Kestrel.</span></span>

  ![Kestrel comunica indirettamente con Internet attraverso un server proxy inverso, ad esempio IIS, Nginx o Apache](kestrel/_static/kestrel-to-internet.png)

<span data-ttu-id="e702c-157">È supportata una configurazione host &mdash; con o senza un server proxy inverso &mdash; .</span><span class="sxs-lookup"><span data-stu-id="e702c-157">Either hosting configuration&mdash;with or without a reverse proxy server&mdash;is supported.</span></span>

<span data-ttu-id="e702c-158">Per indicazioni per la configurazione di Kestrel e per informazioni su quando usare Kestrel in una configurazione con proxy inverso, vedere <xref:fundamentals/servers/kestrel>.</span><span class="sxs-lookup"><span data-stu-id="e702c-158">For Kestrel configuration guidance and information on when to use Kestrel in a reverse proxy configuration, see <xref:fundamentals/servers/kestrel>.</span></span>

::: moniker range="< aspnetcore-2.2"

# <a name="windows"></a>[<span data-ttu-id="e702c-159">Windows</span><span class="sxs-lookup"><span data-stu-id="e702c-159">Windows</span></span>](#tab/windows)

<span data-ttu-id="e702c-160">ASP.NET Core include quanto segue:</span><span class="sxs-lookup"><span data-stu-id="e702c-160">ASP.NET Core ships with the following:</span></span>

* <span data-ttu-id="e702c-161">Il [server kestrel](xref:fundamentals/servers/kestrel) è il server HTTP multipiattaforma predefinito.</span><span class="sxs-lookup"><span data-stu-id="e702c-161">[Kestrel server](xref:fundamentals/servers/kestrel) is the default, cross-platform HTTP server.</span></span>
* <span data-ttu-id="e702c-162">Il [server HTTP.sys](xref:fundamentals/servers/httpsys) è un server HTTP solo per Windows basato sul [driver del kernel HTTP.sys e l'API HTTP Server](/windows/desktop/Http/http-api-start-page).</span><span class="sxs-lookup"><span data-stu-id="e702c-162">[HTTP.sys server](xref:fundamentals/servers/httpsys) is a Windows-only HTTP server based on the [HTTP.sys kernel driver and HTTP Server API](/windows/desktop/Http/http-api-start-page).</span></span>

<span data-ttu-id="e702c-163">Quando si usa [IIS](/iis/get-started/introduction-to-iis/introduction-to-iis-architecture) o [IIS Express](/iis/extensions/introduction-to-iis-express/iis-express-overview), l'app viene eseguita in un processo separato dal processo di lavoro IIS (*out-of-process*) con il [server Kestrel](#kestrel).</span><span class="sxs-lookup"><span data-stu-id="e702c-163">When using [IIS](/iis/get-started/introduction-to-iis/introduction-to-iis-architecture) or [IIS Express](/iis/extensions/introduction-to-iis-express/iis-express-overview), the app runs in a process separate from the IIS worker process (*out-of-process*) with the [Kestrel server](#kestrel).</span></span>

<span data-ttu-id="e702c-164">Poiché le app ASP.NET Core vengono eseguite in un processo distinto dal processo di lavoro IIS, il modulo esegue la gestione dei processi.</span><span class="sxs-lookup"><span data-stu-id="e702c-164">Because ASP.NET Core apps run in a process separate from the IIS worker process, the module handles process management.</span></span> <span data-ttu-id="e702c-165">Il modulo avvia il processo per l'app ASP.NET Core quando arriva la prima richiesta e riavvia l'app se viene arrestata o si arresta in modo anomalo.</span><span class="sxs-lookup"><span data-stu-id="e702c-165">The module starts the process for the ASP.NET Core app when the first request arrives and restarts the app if it shuts down or crashes.</span></span> <span data-ttu-id="e702c-166">Si tratta essenzialmente dello stesso comportamento delle app eseguite in-process e gestite dal [servizio Attivazione processo Windows](/iis/manage/provisioning-and-managing-iis/features-of-the-windows-process-activation-service-was).</span><span class="sxs-lookup"><span data-stu-id="e702c-166">This is essentially the same behavior as seen with apps that run in-process that are managed by the [Windows Process Activation Service (WAS)](/iis/manage/provisioning-and-managing-iis/features-of-the-windows-process-activation-service-was).</span></span>

<span data-ttu-id="e702c-167">Il diagramma seguente illustra la relazione tra IIS, il modulo ASP.NET Core e un'app ospitata out-of-process:</span><span class="sxs-lookup"><span data-stu-id="e702c-167">The following diagram illustrates the relationship between IIS, the ASP.NET Core Module, and an app hosted out-of-process:</span></span>

![Modulo ASP.NET Core](_static/ancm-outofprocess.png)

<span data-ttu-id="e702c-169">Le richieste arrivano dal Web al driver HTTP.sys in modalità kernel.</span><span class="sxs-lookup"><span data-stu-id="e702c-169">Requests arrive from the web to the kernel-mode HTTP.sys driver.</span></span> <span data-ttu-id="e702c-170">Il driver instrada le richieste a IIS sulla porta configurata per il sito Web, in genere 80 (HTTP) o 443 (HTTPS).</span><span class="sxs-lookup"><span data-stu-id="e702c-170">The driver routes the requests to IIS on the website's configured port, usually 80 (HTTP) or 443 (HTTPS).</span></span> <span data-ttu-id="e702c-171">Il modulo inoltra le richieste a Kestrel su una porta casuale per l'app non corrispondente alla porta 80 o 443.</span><span class="sxs-lookup"><span data-stu-id="e702c-171">The module forwards the requests to Kestrel on a random port for the app, which isn't port 80 or 443.</span></span>

<span data-ttu-id="e702c-172">Il modulo specifica la porta tramite una variabile di ambiente all'avvio e il [middleware di integrazione di IIS](xref:host-and-deploy/iis/index#enable-the-iisintegration-components) configura il server per l'ascolto `http://localhost:{port}` .</span><span class="sxs-lookup"><span data-stu-id="e702c-172">The module specifies the port via an environment variable at startup, and the [IIS Integration Middleware](xref:host-and-deploy/iis/index#enable-the-iisintegration-components) configures the server to listen on `http://localhost:{port}`.</span></span> <span data-ttu-id="e702c-173">Vengono eseguiti controlli aggiuntivi e le richieste che non provengono dal modulo vengono rifiutate.</span><span class="sxs-lookup"><span data-stu-id="e702c-173">Additional checks are performed, and requests that don't originate from the module are rejected.</span></span> <span data-ttu-id="e702c-174">Il modulo non supporta l'inoltro HTTPS, pertanto le richieste vengono inoltrate tramite HTTP anche se sono state ricevute da IIS tramite HTTPS.</span><span class="sxs-lookup"><span data-stu-id="e702c-174">The module doesn't support HTTPS forwarding, so requests are forwarded over HTTP even if received by IIS over HTTPS.</span></span>

<span data-ttu-id="e702c-175">Dopo che Kestrel ha prelevato la richiesta dal modulo, viene eseguito il push della richiesta nella pipeline middleware ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="e702c-175">After Kestrel picks up the request from the module, the request is pushed into the ASP.NET Core middleware pipeline.</span></span> <span data-ttu-id="e702c-176">La pipeline middleware gestisce la richiesta e la passa come istanza di `HttpContext` alla logica dell'app.</span><span class="sxs-lookup"><span data-stu-id="e702c-176">The middleware pipeline handles the request and passes it on as an `HttpContext` instance to the app's logic.</span></span> <span data-ttu-id="e702c-177">Il middleware aggiunto dall'integrazione di IIS aggiorna lo schema, l'IP remoto e il percorso di base all'account per l'inoltro della richiesta a Kestrel.</span><span class="sxs-lookup"><span data-stu-id="e702c-177">Middleware added by IIS Integration updates the scheme, remote IP, and pathbase to account for forwarding the request to Kestrel.</span></span> <span data-ttu-id="e702c-178">La risposta dell'app viene quindi passata a IIS, che ne esegue di nuovo il push al client HTTP che ha avviato la richiesta.</span><span class="sxs-lookup"><span data-stu-id="e702c-178">The app's response is passed back to IIS, which pushes it back out to the HTTP client that initiated the request.</span></span>

<span data-ttu-id="e702c-179">Per indicazioni sulla configurazione di IIS e del modulo ASP.NET Core, vedere gli argomenti seguenti:</span><span class="sxs-lookup"><span data-stu-id="e702c-179">For IIS and ASP.NET Core Module configuration guidance, see the following topics:</span></span>

* <xref:host-and-deploy/iis/index>
* <xref:host-and-deploy/aspnet-core-module>

# <a name="macos"></a>[<span data-ttu-id="e702c-180">macOS</span><span class="sxs-lookup"><span data-stu-id="e702c-180">macOS</span></span>](#tab/macos)

<span data-ttu-id="e702c-181">ASP.NET Core viene fornito con il [server Kestrel](xref:fundamentals/servers/kestrel), ovvero il server HTTP multipiattaforma predefinito.</span><span class="sxs-lookup"><span data-stu-id="e702c-181">ASP.NET Core ships with [Kestrel server](xref:fundamentals/servers/kestrel), which is the default, cross-platform HTTP server.</span></span>

# <a name="linux"></a>[<span data-ttu-id="e702c-182">Linux</span><span class="sxs-lookup"><span data-stu-id="e702c-182">Linux</span></span>](#tab/linux)

<span data-ttu-id="e702c-183">ASP.NET Core viene fornito con il [server Kestrel](xref:fundamentals/servers/kestrel), ovvero il server HTTP multipiattaforma predefinito.</span><span class="sxs-lookup"><span data-stu-id="e702c-183">ASP.NET Core ships with [Kestrel server](xref:fundamentals/servers/kestrel), which is the default, cross-platform HTTP server.</span></span>

---

::: moniker-end

### <a name="nginx-with-kestrel"></a><span data-ttu-id="e702c-184">Nginx con Kestrel</span><span class="sxs-lookup"><span data-stu-id="e702c-184">Nginx with Kestrel</span></span>

<span data-ttu-id="e702c-185">Per informazioni su come usare Nginx in Linux come server proxy inverso per Kestrel, vedere <xref:host-and-deploy/linux-nginx>.</span><span class="sxs-lookup"><span data-stu-id="e702c-185">For information on how to use Nginx on Linux as a reverse proxy server for Kestrel, see <xref:host-and-deploy/linux-nginx>.</span></span>

### <a name="apache-with-kestrel"></a><span data-ttu-id="e702c-186">Apache con Kestrel</span><span class="sxs-lookup"><span data-stu-id="e702c-186">Apache with Kestrel</span></span>

<span data-ttu-id="e702c-187">Per informazioni su come usare Apache in Linux come server proxy inverso per Kestrel, vedere <xref:host-and-deploy/linux-apache>.</span><span class="sxs-lookup"><span data-stu-id="e702c-187">For information on how to use Apache on Linux as a reverse proxy server for Kestrel, see <xref:host-and-deploy/linux-apache>.</span></span>

## <a name="httpsys"></a><span data-ttu-id="e702c-188">HTTP.sys</span><span class="sxs-lookup"><span data-stu-id="e702c-188">HTTP.sys</span></span>

<span data-ttu-id="e702c-189">Se si eseguono app ASP.NET Core in Windows, HTTP.sys è un'alternativa a Kestrel.</span><span class="sxs-lookup"><span data-stu-id="e702c-189">If ASP.NET Core apps are run on Windows, HTTP.sys is an alternative to Kestrel.</span></span> <span data-ttu-id="e702c-190">È consigliabile usare gheppio su HTTP.sys, a meno che l'app non richieda funzionalità non disponibili in gheppio.</span><span class="sxs-lookup"><span data-stu-id="e702c-190">Kestrel is recommended over HTTP.sys unless the app requires features not available in Kestrel.</span></span> <span data-ttu-id="e702c-191">Per altre informazioni, vedere <xref:fundamentals/servers/httpsys>.</span><span class="sxs-lookup"><span data-stu-id="e702c-191">For more information, see <xref:fundamentals/servers/httpsys>.</span></span>

![HTTP.sys comunica direttamente con Internet](httpsys/_static/httpsys-to-internet.png)

<span data-ttu-id="e702c-193">HTTP.sys può essere usato anche per le app che vengono esposte solo a una rete interna.</span><span class="sxs-lookup"><span data-stu-id="e702c-193">HTTP.sys can also be used for apps that are only exposed to an internal network.</span></span>

![HTTP.sys comunica direttamente con la rete interna](httpsys/_static/httpsys-to-internal.png)

<span data-ttu-id="e702c-195">Per indicazioni per la configurazione di HTTP.sys, vedere <xref:fundamentals/servers/httpsys>.</span><span class="sxs-lookup"><span data-stu-id="e702c-195">For HTTP.sys configuration guidance, see <xref:fundamentals/servers/httpsys>.</span></span>

## <a name="aspnet-core-server-infrastructure"></a><span data-ttu-id="e702c-196">Infrastruttura del server ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="e702c-196">ASP.NET Core server infrastructure</span></span>

<span data-ttu-id="e702c-197">L'oggetto <xref:Microsoft.AspNetCore.Builder.IApplicationBuilder> disponibile nel metodo `Startup.Configure` espone la proprietà <xref:Microsoft.AspNetCore.Builder.IApplicationBuilder.ServerFeatures> del tipo <xref:Microsoft.AspNetCore.Http.Features.IFeatureCollection>.</span><span class="sxs-lookup"><span data-stu-id="e702c-197">The <xref:Microsoft.AspNetCore.Builder.IApplicationBuilder> available in the `Startup.Configure` method exposes the <xref:Microsoft.AspNetCore.Builder.IApplicationBuilder.ServerFeatures> property of type <xref:Microsoft.AspNetCore.Http.Features.IFeatureCollection>.</span></span> <span data-ttu-id="e702c-198">Kestrel e HTTP.sys espongono una singola funzionalità ciascuno, <xref:Microsoft.AspNetCore.Hosting.Server.Features.IServerAddressesFeature>, ma implementazioni server diverse potrebbero esporre funzionalità aggiuntive.</span><span class="sxs-lookup"><span data-stu-id="e702c-198">Kestrel and HTTP.sys only expose a single feature each, <xref:Microsoft.AspNetCore.Hosting.Server.Features.IServerAddressesFeature>, but different server implementations may expose additional functionality.</span></span>

<span data-ttu-id="e702c-199">È possibile usare `IServerAddressesFeature` per individuare la porta a cui è associata l'implementazione server in fase di esecuzione.</span><span class="sxs-lookup"><span data-stu-id="e702c-199">`IServerAddressesFeature` can be used to find out which port the server implementation has bound at runtime.</span></span>

## <a name="custom-servers"></a><span data-ttu-id="e702c-200">Server personalizzati</span><span class="sxs-lookup"><span data-stu-id="e702c-200">Custom servers</span></span>

<span data-ttu-id="e702c-201">Se i server predefiniti non soddisfano i requisiti dell'app, è possibile creare un'implementazione server personalizzata.</span><span class="sxs-lookup"><span data-stu-id="e702c-201">If the built-in servers don't meet the app's requirements, a custom server implementation can be created.</span></span> <span data-ttu-id="e702c-202">La [guida a Open Web Interface for .NET (OWIN)](xref:fundamentals/owin) spiega come scrivere un'implementazione [Nowin](https://github.com/Bobris/Nowin) di <xref:Microsoft.AspNetCore.Hosting.Server.IServer>.</span><span class="sxs-lookup"><span data-stu-id="e702c-202">The [Open Web Interface for .NET (OWIN) guide](xref:fundamentals/owin) demonstrates how to write a [Nowin](https://github.com/Bobris/Nowin)-based <xref:Microsoft.AspNetCore.Hosting.Server.IServer> implementation.</span></span> <span data-ttu-id="e702c-203">È necessario implementare solo le interfacce delle funzionalità usate dall'app, anche se è richiesto come minimo il supporto di <xref:Microsoft.AspNetCore.Http.Features.IHttpRequestFeature> e <xref:Microsoft.AspNetCore.Http.Features.IHttpResponseFeature>.</span><span class="sxs-lookup"><span data-stu-id="e702c-203">Only the feature interfaces that the app uses require implementation, though at a minimum <xref:Microsoft.AspNetCore.Http.Features.IHttpRequestFeature> and <xref:Microsoft.AspNetCore.Http.Features.IHttpResponseFeature> must be supported.</span></span>

## <a name="server-startup"></a><span data-ttu-id="e702c-204">Avvio del server</span><span class="sxs-lookup"><span data-stu-id="e702c-204">Server startup</span></span>

<span data-ttu-id="e702c-205">Il server viene avviato quando l'editor o l'ambiente di sviluppo integrato (IDE) avvia l'app:</span><span class="sxs-lookup"><span data-stu-id="e702c-205">The server is launched when the Integrated Development Environment (IDE) or editor starts the app:</span></span>

* <span data-ttu-id="e702c-206">[Visual Studio](https://visualstudio.microsoft.com): i profili di avvio possono essere usati per avviare l'app e il server con [IIS Express](/iis/extensions/introduction-to-iis-express/iis-express-overview) / [ASP.NET Core modulo](xref:host-and-deploy/aspnet-core-module) o la console.</span><span class="sxs-lookup"><span data-stu-id="e702c-206">[Visual Studio](https://visualstudio.microsoft.com): Launch profiles can be used to start the app and server with either [IIS Express](/iis/extensions/introduction-to-iis-express/iis-express-overview)/[ASP.NET Core Module](xref:host-and-deploy/aspnet-core-module) or the console.</span></span>
* <span data-ttu-id="e702c-207">[Visual Studio Code](https://code.visualstudio.com/): l'app e il server vengono avviati da [Omnisharp](https://github.com/OmniSharp/omnisharp-vscode), che attiva il debugger di CoreCLR.</span><span class="sxs-lookup"><span data-stu-id="e702c-207">[Visual Studio Code](https://code.visualstudio.com/): The app and server are started by [Omnisharp](https://github.com/OmniSharp/omnisharp-vscode), which activates the CoreCLR debugger.</span></span>
* <span data-ttu-id="e702c-208">[Visual Studio per Mac](https://visualstudio.microsoft.com/vs/mac/): l'app e il server vengono avviati dal [Debugger di mono Soft-Mode](https://www.mono-project.com/docs/advanced/runtime/docs/soft-debugger/).</span><span class="sxs-lookup"><span data-stu-id="e702c-208">[Visual Studio for Mac](https://visualstudio.microsoft.com/vs/mac/): The app and server are started by the [Mono Soft-Mode Debugger](https://www.mono-project.com/docs/advanced/runtime/docs/soft-debugger/).</span></span>

<span data-ttu-id="e702c-209">All'avvio dell'app dal prompt dei comandi nella cartella del progetto, [dotnet run](/dotnet/core/tools/dotnet-run) avvia l'app e il server (solo Kestrel e HTTP.sys).</span><span class="sxs-lookup"><span data-stu-id="e702c-209">When launching the app from a command prompt in the project's folder, [dotnet run](/dotnet/core/tools/dotnet-run) launches the app and server (Kestrel and HTTP.sys only).</span></span> <span data-ttu-id="e702c-210">La configurazione viene specificata dall'opzione `-c|--configuration`, impostata su `Debug` (valore predefinito) o su `Release`.</span><span class="sxs-lookup"><span data-stu-id="e702c-210">The configuration is specified by the `-c|--configuration` option, which is set to either `Debug` (default) or `Release`.</span></span>

<span data-ttu-id="e702c-211">Un *launchSettings.jssu* file fornisce la configurazione quando si avvia un'app con `dotnet run` o con un debugger incorporato in strumenti, ad esempio Visual Studio.</span><span class="sxs-lookup"><span data-stu-id="e702c-211">A *launchSettings.json* file provides configuration when launching an app with `dotnet run` or with a debugger built into tooling, such as Visual Studio.</span></span> <span data-ttu-id="e702c-212">Se i profili di avvio sono presenti in un *launchSettings.jssu* file, usare l' `--launch-profile {PROFILE NAME}` opzione con il `dotnet run` comando o selezionare il profilo in Visual Studio.</span><span class="sxs-lookup"><span data-stu-id="e702c-212">If launch profiles are present in a *launchSettings.json* file, use the `--launch-profile {PROFILE NAME}` option with the `dotnet run` command or select the profile in Visual Studio.</span></span> <span data-ttu-id="e702c-213">Per altre informazioni, vedere [dotnet run](/dotnet/core/tools/dotnet-run) e [Creazione di pacchetti di distribuzione di .NET Core](/dotnet/core/build/distribution-packaging).</span><span class="sxs-lookup"><span data-stu-id="e702c-213">For more information, see [dotnet run](/dotnet/core/tools/dotnet-run) and [.NET Core distribution packaging](/dotnet/core/build/distribution-packaging).</span></span>

## <a name="http2-support"></a><span data-ttu-id="e702c-214">Supporto HTTP/2</span><span class="sxs-lookup"><span data-stu-id="e702c-214">HTTP/2 support</span></span>

<span data-ttu-id="e702c-215">[HTTP/2](https://httpwg.org/specs/rfc7540.html) è supportato con ASP.NET Core negli scenari di distribuzione seguenti:</span><span class="sxs-lookup"><span data-stu-id="e702c-215">[HTTP/2](https://httpwg.org/specs/rfc7540.html) is supported with ASP.NET Core in the following deployment scenarios:</span></span>

::: moniker range=">= aspnetcore-5.0"

* [<span data-ttu-id="e702c-216">Kestrel</span><span class="sxs-lookup"><span data-stu-id="e702c-216">Kestrel</span></span>](xref:fundamentals/servers/kestrel/http2)
  * <span data-ttu-id="e702c-217">Sistema operativo</span><span class="sxs-lookup"><span data-stu-id="e702c-217">Operating system</span></span>
    * <span data-ttu-id="e702c-218">Windows Server 2016/Windows 10 o versioni successive&dagger;</span><span class="sxs-lookup"><span data-stu-id="e702c-218">Windows Server 2016/Windows 10 or later&dagger;</span></span>
    * <span data-ttu-id="e702c-219">Linux con OpenSSL 1.0.2 o versioni successive (ad esempio, Ubuntu 16.04 o versioni successive)</span><span class="sxs-lookup"><span data-stu-id="e702c-219">Linux with OpenSSL 1.0.2 or later (for example, Ubuntu 16.04 or later)</span></span>
    * <span data-ttu-id="e702c-220">HTTP/2 verrà supportato in macOS in una versione futura.</span><span class="sxs-lookup"><span data-stu-id="e702c-220">HTTP/2 will be supported on macOS in a future release.</span></span>
  * <span data-ttu-id="e702c-221">Framework di destinazione: .NET Core 2.2 o versioni successive</span><span class="sxs-lookup"><span data-stu-id="e702c-221">Target framework: .NET Core 2.2 or later</span></span>
* [<span data-ttu-id="e702c-222">HTTP.sys</span><span class="sxs-lookup"><span data-stu-id="e702c-222">HTTP.sys</span></span>](xref:fundamentals/servers/httpsys#http2-support)
  * <span data-ttu-id="e702c-223">Windows Server 2016/Windows 10 o versioni successive</span><span class="sxs-lookup"><span data-stu-id="e702c-223">Windows Server 2016/Windows 10 or later</span></span>
  * <span data-ttu-id="e702c-224">Framework di destinazione: non applicabile alle distribuzioni HTTP.sys.</span><span class="sxs-lookup"><span data-stu-id="e702c-224">Target framework: Not applicable to HTTP.sys deployments.</span></span>
* [<span data-ttu-id="e702c-225">IIS (in-process)</span><span class="sxs-lookup"><span data-stu-id="e702c-225">IIS (in-process)</span></span>](xref:host-and-deploy/iis/index#http2-support)
  * <span data-ttu-id="e702c-226">Windows Server 2016/Windows 10 o versioni successive; IIS 10 o versioni successive</span><span class="sxs-lookup"><span data-stu-id="e702c-226">Windows Server 2016/Windows 10 or later; IIS 10 or later</span></span>
  * <span data-ttu-id="e702c-227">Framework di destinazione: .NET Core 2.2 o versioni successive</span><span class="sxs-lookup"><span data-stu-id="e702c-227">Target framework: .NET Core 2.2 or later</span></span>
* [<span data-ttu-id="e702c-228">IIS (out-of-process)</span><span class="sxs-lookup"><span data-stu-id="e702c-228">IIS (out-of-process)</span></span>](xref:host-and-deploy/iis/index#http2-support)
  * <span data-ttu-id="e702c-229">Windows Server 2016/Windows 10 o versioni successive; IIS 10 o versioni successive</span><span class="sxs-lookup"><span data-stu-id="e702c-229">Windows Server 2016/Windows 10 or later; IIS 10 or later</span></span>
  * <span data-ttu-id="e702c-230">Le connessioni server perimetrali pubbliche usano HTTP/2, mentre la connessione con proxy inverso a Kestrel usa HTTP/1.1.</span><span class="sxs-lookup"><span data-stu-id="e702c-230">Public-facing edge server connections use HTTP/2, but the reverse proxy connection to Kestrel uses HTTP/1.1.</span></span>
  * <span data-ttu-id="e702c-231">Framework di destinazione: non applicabile alle distribuzioni IIS out-of-process.</span><span class="sxs-lookup"><span data-stu-id="e702c-231">Target framework: Not applicable to IIS out-of-process deployments.</span></span>

<span data-ttu-id="e702c-232">&dagger;Kestrel ha un supporto limitato per HTTP/2 in Windows Server 2012 R2 e Windows 8.1.</span><span class="sxs-lookup"><span data-stu-id="e702c-232">&dagger;Kestrel has limited support for HTTP/2 on Windows Server 2012 R2 and Windows 8.1.</span></span> <span data-ttu-id="e702c-233">Il supporto è limitato perché l'elenco di suite di crittografia TLS supportate disponibili in questi sistemi operativi è limitato.</span><span class="sxs-lookup"><span data-stu-id="e702c-233">Support is limited because the list of supported TLS cipher suites available on these operating systems is limited.</span></span> <span data-ttu-id="e702c-234">Un certificato generato con un algoritmo ECDSA potrebbe essere necessario per proteggere le connessioni TLS.</span><span class="sxs-lookup"><span data-stu-id="e702c-234">A certificate generated using an Elliptic Curve Digital Signature Algorithm (ECDSA) may be required to secure TLS connections.</span></span>

::: moniker-end

::: moniker range=">= aspnetcore-2.2 < aspnetcore-5.0"

* [<span data-ttu-id="e702c-235">Kestrel</span><span class="sxs-lookup"><span data-stu-id="e702c-235">Kestrel</span></span>](xref:fundamentals/servers/kestrel#http2-support)
  * <span data-ttu-id="e702c-236">Sistema operativo</span><span class="sxs-lookup"><span data-stu-id="e702c-236">Operating system</span></span>
    * <span data-ttu-id="e702c-237">Windows Server 2016/Windows 10 o versioni successive&dagger;</span><span class="sxs-lookup"><span data-stu-id="e702c-237">Windows Server 2016/Windows 10 or later&dagger;</span></span>
    * <span data-ttu-id="e702c-238">Linux con OpenSSL 1.0.2 o versioni successive (ad esempio, Ubuntu 16.04 o versioni successive)</span><span class="sxs-lookup"><span data-stu-id="e702c-238">Linux with OpenSSL 1.0.2 or later (for example, Ubuntu 16.04 or later)</span></span>
    * <span data-ttu-id="e702c-239">HTTP/2 verrà supportato in macOS in una versione futura.</span><span class="sxs-lookup"><span data-stu-id="e702c-239">HTTP/2 will be supported on macOS in a future release.</span></span>
  * <span data-ttu-id="e702c-240">Framework di destinazione: .NET Core 2.2 o versioni successive</span><span class="sxs-lookup"><span data-stu-id="e702c-240">Target framework: .NET Core 2.2 or later</span></span>
* [<span data-ttu-id="e702c-241">HTTP.sys</span><span class="sxs-lookup"><span data-stu-id="e702c-241">HTTP.sys</span></span>](xref:fundamentals/servers/httpsys#http2-support)
  * <span data-ttu-id="e702c-242">Windows Server 2016/Windows 10 o versioni successive</span><span class="sxs-lookup"><span data-stu-id="e702c-242">Windows Server 2016/Windows 10 or later</span></span>
  * <span data-ttu-id="e702c-243">Framework di destinazione: non applicabile alle distribuzioni HTTP.sys.</span><span class="sxs-lookup"><span data-stu-id="e702c-243">Target framework: Not applicable to HTTP.sys deployments.</span></span>
* [<span data-ttu-id="e702c-244">IIS (in-process)</span><span class="sxs-lookup"><span data-stu-id="e702c-244">IIS (in-process)</span></span>](xref:host-and-deploy/iis/index#http2-support)
  * <span data-ttu-id="e702c-245">Windows Server 2016/Windows 10 o versioni successive; IIS 10 o versioni successive</span><span class="sxs-lookup"><span data-stu-id="e702c-245">Windows Server 2016/Windows 10 or later; IIS 10 or later</span></span>
  * <span data-ttu-id="e702c-246">Framework di destinazione: .NET Core 2.2 o versioni successive</span><span class="sxs-lookup"><span data-stu-id="e702c-246">Target framework: .NET Core 2.2 or later</span></span>
* [<span data-ttu-id="e702c-247">IIS (out-of-process)</span><span class="sxs-lookup"><span data-stu-id="e702c-247">IIS (out-of-process)</span></span>](xref:host-and-deploy/iis/index#http2-support)
  * <span data-ttu-id="e702c-248">Windows Server 2016/Windows 10 o versioni successive; IIS 10 o versioni successive</span><span class="sxs-lookup"><span data-stu-id="e702c-248">Windows Server 2016/Windows 10 or later; IIS 10 or later</span></span>
  * <span data-ttu-id="e702c-249">Le connessioni server perimetrali pubbliche usano HTTP/2, mentre la connessione con proxy inverso a Kestrel usa HTTP/1.1.</span><span class="sxs-lookup"><span data-stu-id="e702c-249">Public-facing edge server connections use HTTP/2, but the reverse proxy connection to Kestrel uses HTTP/1.1.</span></span>
  * <span data-ttu-id="e702c-250">Framework di destinazione: non applicabile alle distribuzioni IIS out-of-process.</span><span class="sxs-lookup"><span data-stu-id="e702c-250">Target framework: Not applicable to IIS out-of-process deployments.</span></span>

<span data-ttu-id="e702c-251">&dagger;Kestrel ha un supporto limitato per HTTP/2 in Windows Server 2012 R2 e Windows 8.1.</span><span class="sxs-lookup"><span data-stu-id="e702c-251">&dagger;Kestrel has limited support for HTTP/2 on Windows Server 2012 R2 and Windows 8.1.</span></span> <span data-ttu-id="e702c-252">Il supporto è limitato perché l'elenco di suite di crittografia TLS supportate disponibili in questi sistemi operativi è limitato.</span><span class="sxs-lookup"><span data-stu-id="e702c-252">Support is limited because the list of supported TLS cipher suites available on these operating systems is limited.</span></span> <span data-ttu-id="e702c-253">Un certificato generato con un algoritmo ECDSA potrebbe essere necessario per proteggere le connessioni TLS.</span><span class="sxs-lookup"><span data-stu-id="e702c-253">A certificate generated using an Elliptic Curve Digital Signature Algorithm (ECDSA) may be required to secure TLS connections.</span></span>

::: moniker-end

::: moniker range="< aspnetcore-2.2"

* [<span data-ttu-id="e702c-254">HTTP.sys</span><span class="sxs-lookup"><span data-stu-id="e702c-254">HTTP.sys</span></span>](xref:fundamentals/servers/httpsys#http2-support)
  * <span data-ttu-id="e702c-255">Windows Server 2016/Windows 10 o versioni successive</span><span class="sxs-lookup"><span data-stu-id="e702c-255">Windows Server 2016/Windows 10 or later</span></span>
  * <span data-ttu-id="e702c-256">Framework di destinazione: non applicabile alle distribuzioni HTTP.sys.</span><span class="sxs-lookup"><span data-stu-id="e702c-256">Target framework: Not applicable to HTTP.sys deployments.</span></span>
* [<span data-ttu-id="e702c-257">IIS (out-of-process)</span><span class="sxs-lookup"><span data-stu-id="e702c-257">IIS (out-of-process)</span></span>](xref:host-and-deploy/iis/index#http2-support)
  * <span data-ttu-id="e702c-258">Windows Server 2016/Windows 10 o versioni successive; IIS 10 o versioni successive</span><span class="sxs-lookup"><span data-stu-id="e702c-258">Windows Server 2016/Windows 10 or later; IIS 10 or later</span></span>
  * <span data-ttu-id="e702c-259">Le connessioni server perimetrali pubbliche usano HTTP/2, mentre la connessione con proxy inverso a Kestrel usa HTTP/1.1.</span><span class="sxs-lookup"><span data-stu-id="e702c-259">Public-facing edge server connections use HTTP/2, but the reverse proxy connection to Kestrel uses HTTP/1.1.</span></span>
  * <span data-ttu-id="e702c-260">Framework di destinazione: non applicabile alle distribuzioni IIS out-of-process.</span><span class="sxs-lookup"><span data-stu-id="e702c-260">Target framework: Not applicable to IIS out-of-process deployments.</span></span>

::: moniker-end

<span data-ttu-id="e702c-261">Una connessione HTTP/2 deve usare [ALPN (Application-Layer Protocol Negotiation)](https://tools.ietf.org/html/rfc7301#section-3) e TLS 1.2 o versioni successive.</span><span class="sxs-lookup"><span data-stu-id="e702c-261">An HTTP/2 connection must use [Application-Layer Protocol Negotiation (ALPN)](https://tools.ietf.org/html/rfc7301#section-3) and TLS 1.2 or later.</span></span> <span data-ttu-id="e702c-262">Per altre informazioni, vedere gli argomenti relativi agli scenari di distribuzione server.</span><span class="sxs-lookup"><span data-stu-id="e702c-262">For more information, see the topics that pertain to your server deployment scenarios.</span></span>

## <a name="additional-resources"></a><span data-ttu-id="e702c-263">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="e702c-263">Additional resources</span></span>

* <xref:fundamentals/servers/kestrel>
* <xref:host-and-deploy/aspnet-core-module>
* <xref:host-and-deploy/iis/index>
* <xref:host-and-deploy/azure-apps/index>
* <xref:host-and-deploy/linux-nginx>
* <xref:host-and-deploy/linux-apache>
* <xref:fundamentals/servers/httpsys>
