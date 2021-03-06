---
title: Ospitare e distribuire ASP.NET Core Blazor Server
author: guardrex
description: Informazioni su come ospitare e distribuire un' Blazor Server app usando ASP.NET Core.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 08/26/2020
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
uid: blazor/host-and-deploy/server
ms.openlocfilehash: bac6f4a6595558f32779286bcc09c266663a2d24
ms.sourcegitcommit: 1436bd4d70937d6ec3140da56d96caab33c4320b
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/06/2021
ms.locfileid: "102394980"
---
# <a name="host-and-deploy-blazor-server"></a><span data-ttu-id="710fc-103">Ospitare e distribuire Blazor Server</span><span class="sxs-lookup"><span data-stu-id="710fc-103">Host and deploy Blazor Server</span></span>

## <a name="host-configuration-values"></a><span data-ttu-id="710fc-104">Valori di configurazione dell'host</span><span class="sxs-lookup"><span data-stu-id="710fc-104">Host configuration values</span></span>

<span data-ttu-id="710fc-105">le [ Blazor Server app](xref:blazor/hosting-models#blazor-server) possono accettare [valori di configurazione host generici](xref:fundamentals/host/generic-host#host-configuration).</span><span class="sxs-lookup"><span data-stu-id="710fc-105">[Blazor Server apps](xref:blazor/hosting-models#blazor-server) can accept [Generic Host configuration values](xref:fundamentals/host/generic-host#host-configuration).</span></span>

## <a name="deployment"></a><span data-ttu-id="710fc-106">Distribuzione</span><span class="sxs-lookup"><span data-stu-id="710fc-106">Deployment</span></span>

<span data-ttu-id="710fc-107">L'uso del [ Blazor Server modello di hosting](xref:blazor/hosting-models#blazor-server) Blazor viene eseguito sul server dall'interno di un'app ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="710fc-107">Using the [Blazor Server hosting model](xref:blazor/hosting-models#blazor-server), Blazor is executed on the server from within an ASP.NET Core app.</span></span> <span data-ttu-id="710fc-108">Gli aggiornamenti dell'interfaccia utente, la gestione degli eventi e le chiamate JavaScript vengono gestiti tramite una [SignalR](xref:signalr/introduction) connessione.</span><span class="sxs-lookup"><span data-stu-id="710fc-108">UI updates, event handling, and JavaScript calls are handled over a [SignalR](xref:signalr/introduction) connection.</span></span>

<span data-ttu-id="710fc-109">È necessario un server Web in grado di ospitare un'app ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="710fc-109">A web server capable of hosting an ASP.NET Core app is required.</span></span> <span data-ttu-id="710fc-110">Visual Studio include il modello di progetto **Blazor Server app** ( `blazorserverside` modello quando si usa il [`dotnet new`](/dotnet/core/tools/dotnet-new) comando).</span><span class="sxs-lookup"><span data-stu-id="710fc-110">Visual Studio includes the **Blazor Server App** project template (`blazorserverside` template when using the [`dotnet new`](/dotnet/core/tools/dotnet-new) command).</span></span> <span data-ttu-id="710fc-111">Per ulteriori informazioni sui Blazor modelli di progetto di, vedere <xref:blazor/project-structure> .</span><span class="sxs-lookup"><span data-stu-id="710fc-111">For more information on Blazor project templates, see <xref:blazor/project-structure>.</span></span>

## <a name="scalability"></a><span data-ttu-id="710fc-112">Scalabilità</span><span class="sxs-lookup"><span data-stu-id="710fc-112">Scalability</span></span>

<span data-ttu-id="710fc-113">Pianificare una distribuzione per sfruttare al meglio l'infrastruttura disponibile per un' Blazor Server app.</span><span class="sxs-lookup"><span data-stu-id="710fc-113">Plan a deployment to make the best use of the available infrastructure for a Blazor Server app.</span></span> <span data-ttu-id="710fc-114">Per risolvere la scalabilità delle app, vedere le risorse seguenti Blazor Server :</span><span class="sxs-lookup"><span data-stu-id="710fc-114">See the following resources to address Blazor Server app scalability:</span></span>

* [<span data-ttu-id="710fc-115">Nozioni fondamentali sulle Blazor Server app</span><span class="sxs-lookup"><span data-stu-id="710fc-115">Fundamentals of Blazor Server apps</span></span>](xref:blazor/hosting-models#blazor-server)
* <xref:blazor/security/server/threat-mitigation>

### <a name="deployment-server"></a><span data-ttu-id="710fc-116">Server di distribuzione</span><span class="sxs-lookup"><span data-stu-id="710fc-116">Deployment server</span></span>

<span data-ttu-id="710fc-117">Quando si considera la scalabilità di un singolo server (scalabilità verticale), la memoria disponibile per un'app è probabilmente la prima risorsa che verrà esaurita dall'app quando le richieste degli utenti aumentano.</span><span class="sxs-lookup"><span data-stu-id="710fc-117">When considering the scalability of a single server (scale up), the memory available to an app is likely the first resource that the app will exhaust as user demands increase.</span></span> <span data-ttu-id="710fc-118">La memoria disponibile sul server influiscono su:</span><span class="sxs-lookup"><span data-stu-id="710fc-118">The available memory on the server affects the:</span></span>

* <span data-ttu-id="710fc-119">Numero di circuiti attivi che un server è in grado di supportare.</span><span class="sxs-lookup"><span data-stu-id="710fc-119">Number of active circuits that a server can support.</span></span>
* <span data-ttu-id="710fc-120">Latenza dell'interfaccia utente nel client.</span><span class="sxs-lookup"><span data-stu-id="710fc-120">UI latency on the client.</span></span>

<span data-ttu-id="710fc-121">Per istruzioni sulla creazione di app Server sicure e scalabili Blazor , vedere <xref:blazor/security/server/threat-mitigation> .</span><span class="sxs-lookup"><span data-stu-id="710fc-121">For guidance on building secure and scalable Blazor server apps, see <xref:blazor/security/server/threat-mitigation>.</span></span>

<span data-ttu-id="710fc-122">Ogni circuito utilizza circa 250 KB di memoria per un'app di tipo *Hello World* minima.</span><span class="sxs-lookup"><span data-stu-id="710fc-122">Each circuit uses approximately 250 KB of memory for a minimal *Hello World*-style app.</span></span> <span data-ttu-id="710fc-123">La dimensione di un circuito dipende dal codice dell'app e dai requisiti di manutenzione dello stato associati a ogni componente.</span><span class="sxs-lookup"><span data-stu-id="710fc-123">The size of a circuit depends on the app's code and the state maintenance requirements associated with each component.</span></span> <span data-ttu-id="710fc-124">Si consiglia di misurare le richieste di risorse durante lo sviluppo dell'applicazione e dell'infrastruttura, ma la linea di base seguente può essere un punto di partenza per la pianificazione della destinazione di distribuzione: se si prevede che l'app supporti 5.000 utenti simultanei, valutare la possibilità di prevedere almeno 1,3 GB di memoria del server per l'app (o ~ 273 KB per utente).</span><span class="sxs-lookup"><span data-stu-id="710fc-124">We recommend that you measure resource demands during development for your app and infrastructure, but the following baseline can be a starting point in planning your deployment target: If you expect your app to support 5,000 concurrent users, consider budgeting at least 1.3 GB of server memory to the app (or ~273 KB per user).</span></span>

### <a name="signalr-configuration"></a><span data-ttu-id="710fc-125">SignalR configurazione</span><span class="sxs-lookup"><span data-stu-id="710fc-125">SignalR configuration</span></span>

<span data-ttu-id="710fc-126">Blazor Server le app usano ASP.NET Core SignalR per comunicare con il browser.</span><span class="sxs-lookup"><span data-stu-id="710fc-126">Blazor Server apps use ASP.NET Core SignalR to communicate with the browser.</span></span> <span data-ttu-id="710fc-127">[ SignalR le condizioni di hosting e scalabilità sono](xref:signalr/publish-to-azure-web-app) valide per le Blazor Server app.</span><span class="sxs-lookup"><span data-stu-id="710fc-127">[SignalR's hosting and scaling conditions](xref:signalr/publish-to-azure-web-app) apply to Blazor Server apps.</span></span>

<span data-ttu-id="710fc-128">Blazor funziona meglio quando si usano WebSocket come il SignalR trasporto a causa di una latenza, affidabilità e [sicurezza](xref:signalr/security)più bassa.</span><span class="sxs-lookup"><span data-stu-id="710fc-128">Blazor works best when using WebSockets as the SignalR transport due to lower latency, reliability, and [security](xref:signalr/security).</span></span> <span data-ttu-id="710fc-129">Il polling prolungato viene usato da SignalR quando i WebSocket non sono disponibili o quando l'app è configurata in modo esplicito per l'uso del polling prolungato.</span><span class="sxs-lookup"><span data-stu-id="710fc-129">Long Polling is used by SignalR when WebSockets isn't available or when the app is explicitly configured to use Long Polling.</span></span> <span data-ttu-id="710fc-130">Quando si esegue la distribuzione nel servizio app Azure, configurare l'app per l'uso di WebSocket nelle impostazioni portale di Azure per il servizio.</span><span class="sxs-lookup"><span data-stu-id="710fc-130">When deploying to Azure App Service, configure the app to use WebSockets in the Azure portal settings for the service.</span></span> <span data-ttu-id="710fc-131">Per informazioni dettagliate sulla configurazione dell'app per il servizio app Azure, vedere le [ SignalR linee guida](xref:signalr/publish-to-azure-web-app)per la pubblicazione.</span><span class="sxs-lookup"><span data-stu-id="710fc-131">For details on configuring the app for Azure App Service, see the [SignalR publishing guidelines](xref:signalr/publish-to-azure-web-app).</span></span>

#### <a name="azure-signalr-service"></a><span data-ttu-id="710fc-132">Servizio di Azure SignalR</span><span class="sxs-lookup"><span data-stu-id="710fc-132">Azure SignalR Service</span></span>

<span data-ttu-id="710fc-133">È consigliabile usare il [ SignalR servizio Azure](xref:signalr/scale#azure-signalr-service) per le Blazor Server app.</span><span class="sxs-lookup"><span data-stu-id="710fc-133">We recommend using the [Azure SignalR Service](xref:signalr/scale#azure-signalr-service) for Blazor Server apps.</span></span> <span data-ttu-id="710fc-134">Il servizio consente di ridimensionare un' Blazor Server app a un numero elevato di connessioni simultanee SignalR .</span><span class="sxs-lookup"><span data-stu-id="710fc-134">The service allows for scaling up a Blazor Server app to a large number of concurrent SignalR connections.</span></span> <span data-ttu-id="710fc-135">Inoltre, la SignalR portata globale del servizio e i Data Center a prestazioni elevate facilitano significativamente la riduzione della latenza dovuta all'area geografica.</span><span class="sxs-lookup"><span data-stu-id="710fc-135">In addition, the SignalR service's global reach and high-performance data centers significantly aid in reducing latency due to geography.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="710fc-136">Quando i [WebSocket](https://wikipedia.org/wiki/WebSocket) sono disabilitati, app Azure servizio simula una connessione in tempo reale usando il polling lungo http.</span><span class="sxs-lookup"><span data-stu-id="710fc-136">When [WebSockets](https://wikipedia.org/wiki/WebSocket) are disabled, Azure App Service simulates a real-time connection using HTTP Long Polling.</span></span> <span data-ttu-id="710fc-137">Il polling lungo HTTP è notevolmente più lento rispetto all'esecuzione con WebSocket abilitati, che non usa il polling per simulare una connessione client-server.</span><span class="sxs-lookup"><span data-stu-id="710fc-137">HTTP Long Polling is noticeably slower than running with WebSockets enabled, which doesn't use polling to simulate a client-server connection.</span></span>
>
> <span data-ttu-id="710fc-138">È consigliabile usare WebSocket per le Blazor Server app distribuite nel servizio app Azure.</span><span class="sxs-lookup"><span data-stu-id="710fc-138">We recommend using WebSockets for Blazor Server apps deployed to Azure App Service.</span></span> <span data-ttu-id="710fc-139">Per impostazione predefinita, il [ SignalR servizio di Azure](xref:signalr/scale#azure-signalr-service) Usa WebSocket.</span><span class="sxs-lookup"><span data-stu-id="710fc-139">The [Azure SignalR Service](xref:signalr/scale#azure-signalr-service) uses WebSockets by default.</span></span> <span data-ttu-id="710fc-140">Se l'app non usa il servizio di Azure SignalR , vedere <xref:signalr/publish-to-azure-web-app#configure-the-app-in-azure-app-service> .</span><span class="sxs-lookup"><span data-stu-id="710fc-140">If the app doesn't use the Azure SignalR Service, see <xref:signalr/publish-to-azure-web-app#configure-the-app-in-azure-app-service>.</span></span>
>
> <span data-ttu-id="710fc-141">Per altre informazioni, vedere:</span><span class="sxs-lookup"><span data-stu-id="710fc-141">For more information, see:</span></span>
>
> * [<span data-ttu-id="710fc-142">Che cos'è il SignalR servizio Azure?</span><span class="sxs-lookup"><span data-stu-id="710fc-142">What is Azure SignalR Service?</span></span>](/azure/azure-signalr/signalr-overview)
> * [<span data-ttu-id="710fc-143">Guida alle prestazioni per il SignalR servizio Azure</span><span class="sxs-lookup"><span data-stu-id="710fc-143">Performance guide for Azure SignalR Service</span></span>](/azure-signalr/signalr-concept-performance#performance-factors)

### <a name="configuration"></a><span data-ttu-id="710fc-144">Configurazione</span><span class="sxs-lookup"><span data-stu-id="710fc-144">Configuration</span></span>

<span data-ttu-id="710fc-145">Per configurare un'app per il servizio di Azure SignalR , l'app deve supportare le *sessioni permanenti*, in cui i client vengono [reindirizzati allo stesso server durante il prerendering](xref:blazor/hosting-models#connection-to-the-server).</span><span class="sxs-lookup"><span data-stu-id="710fc-145">To configure an app for the Azure SignalR Service, the app must support *sticky sessions*, where clients are [redirected back to the same server when prerendering](xref:blazor/hosting-models#connection-to-the-server).</span></span> <span data-ttu-id="710fc-146">L' `ServerStickyMode` opzione o il valore di configurazione è impostato su `Required` .</span><span class="sxs-lookup"><span data-stu-id="710fc-146">The `ServerStickyMode` option or configuration value is set to `Required`.</span></span> <span data-ttu-id="710fc-147">In genere, un'app crea la configurazione usando **_uno_** degli approcci seguenti:</span><span class="sxs-lookup"><span data-stu-id="710fc-147">Typically, an app creates the configuration using **_ONE_** of the following approaches:</span></span>

   * <span data-ttu-id="710fc-148">`Startup.ConfigureServices`:</span><span class="sxs-lookup"><span data-stu-id="710fc-148">`Startup.ConfigureServices`:</span></span>
  
     ```csharp
     services.AddSignalR().AddAzureSignalR(options =>
     {
         options.ServerStickyMode = 
             Microsoft.Azure.SignalR.ServerStickyMode.Required;
     });
     ```

   * <span data-ttu-id="710fc-149">Configurazione (usare **_uno_** degli approcci seguenti):</span><span class="sxs-lookup"><span data-stu-id="710fc-149">Configuration (use **_ONE_** of the following approaches):</span></span>
  
     * <span data-ttu-id="710fc-150">In `appsettings.json`:</span><span class="sxs-lookup"><span data-stu-id="710fc-150">In `appsettings.json`:</span></span>

       ```json
       "Azure:SignalR:StickyServerMode": "Required"
       ```

     * <span data-ttu-id="710fc-151">Impostazioni dell'applicazione di **configurazione** del servizio app  >   nel portale di Azure (**nome**: `Azure__SignalR__StickyServerMode` , **valore**: `Required` ).</span><span class="sxs-lookup"><span data-stu-id="710fc-151">The app service's **Configuration** > **Application settings** in the Azure portal (**Name**: `Azure__SignalR__StickyServerMode`, **Value**: `Required`).</span></span> <span data-ttu-id="710fc-152">Questo approccio viene adottato automaticamente per l'app se si esegue [il provisioning del SignalR servizio di Azure](#provision-the-azure-signalr-service).</span><span class="sxs-lookup"><span data-stu-id="710fc-152">This approach is adopted for the app automatically if you [provision the Azure SignalR Service](#provision-the-azure-signalr-service).</span></span>

### <a name="provision-the-azure-signalr-service"></a><span data-ttu-id="710fc-153">Effettuare il provisioning del SignalR servizio Azure</span><span class="sxs-lookup"><span data-stu-id="710fc-153">Provision the Azure SignalR Service</span></span>

<span data-ttu-id="710fc-154">Per eseguire il provisioning del SignalR servizio Azure per un'app in Visual Studio:</span><span class="sxs-lookup"><span data-stu-id="710fc-154">To provision the Azure SignalR Service for an app in Visual Studio:</span></span>

1. <span data-ttu-id="710fc-155">Creare un profilo di pubblicazione per le app di Azure in Visual Studio per l' Blazor Server app.</span><span class="sxs-lookup"><span data-stu-id="710fc-155">Create an Azure Apps publish profile in Visual Studio for the Blazor Server app.</span></span>
1. <span data-ttu-id="710fc-156">Aggiungere la dipendenza del **SignalR servizio di Azure** al profilo.</span><span class="sxs-lookup"><span data-stu-id="710fc-156">Add the **Azure SignalR Service** dependency to the profile.</span></span> <span data-ttu-id="710fc-157">Se la sottoscrizione di Azure non dispone di un'istanza del servizio di Azure preesistente SignalR da assegnare all'app, selezionare **Crea una nuova SignalR istanza del servizio di Azure** per eseguire il provisioning di una nuova istanza del servizio.</span><span class="sxs-lookup"><span data-stu-id="710fc-157">If the Azure subscription doesn't have a pre-existing Azure SignalR Service instance to assign to the app, select **Create a new Azure SignalR Service instance** to provision a new service instance.</span></span>
1. <span data-ttu-id="710fc-158">Pubblicare l'app in Azure.</span><span class="sxs-lookup"><span data-stu-id="710fc-158">Publish the app to Azure.</span></span>

<span data-ttu-id="710fc-159">Il provisioning del SignalR servizio di Azure in Visual Studio Abilita automaticamente le [ *sessioni permanenti*](#configuration) e aggiunge la SignalR stringa di connessione alla configurazione del servizio app.</span><span class="sxs-lookup"><span data-stu-id="710fc-159">Provisioning the Azure SignalR Service in Visual Studio automatically [enables *sticky sessions*](#configuration) and adds the SignalR connection string to the app service's configuration.</span></span>

#### <a name="iis"></a><span data-ttu-id="710fc-160">IIS</span><span class="sxs-lookup"><span data-stu-id="710fc-160">IIS</span></span>

<span data-ttu-id="710fc-161">Quando si usa IIS, abilitare:</span><span class="sxs-lookup"><span data-stu-id="710fc-161">When using IIS, enable:</span></span>

* <span data-ttu-id="710fc-162">[WebSocket in IIS](xref:fundamentals/websockets#enabling-websockets-on-iis).</span><span class="sxs-lookup"><span data-stu-id="710fc-162">[WebSockets on IIS](xref:fundamentals/websockets#enabling-websockets-on-iis).</span></span>
* <span data-ttu-id="710fc-163">[Sessioni permanenti con routing della richiesta dell'applicazione](/iis/extensions/configuring-application-request-routing-arr/http-load-balancing-using-application-request-routing).</span><span class="sxs-lookup"><span data-stu-id="710fc-163">[Sticky sessions with Application Request Routing](/iis/extensions/configuring-application-request-routing-arr/http-load-balancing-using-application-request-routing).</span></span>

#### <a name="kubernetes"></a><span data-ttu-id="710fc-164">Kubernetes</span><span class="sxs-lookup"><span data-stu-id="710fc-164">Kubernetes</span></span>

<span data-ttu-id="710fc-165">Creare una definizione di ingresso con le [annotazioni Kubernetes seguenti per le sessioni permanenti](https://kubernetes.github.io/ingress-nginx/examples/affinity/cookie/):</span><span class="sxs-lookup"><span data-stu-id="710fc-165">Create an ingress definition with the following [Kubernetes annotations for sticky sessions](https://kubernetes.github.io/ingress-nginx/examples/affinity/cookie/):</span></span>

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: <ingress-name>
  annotations:
    nginx.ingress.kubernetes.io/affinity: "cookie"
    nginx.ingress.kubernetes.io/session-cookie-name: "affinity"
    nginx.ingress.kubernetes.io/session-cookie-expires: "14400"
    nginx.ingress.kubernetes.io/session-cookie-max-age: "14400"
```

#### <a name="linux-with-nginx"></a><span data-ttu-id="710fc-166">Linux con Nginx</span><span class="sxs-lookup"><span data-stu-id="710fc-166">Linux with Nginx</span></span>

<span data-ttu-id="710fc-167">Per SignalR il corretto funzionamento di WebSocket, verificare che le `Upgrade` intestazioni e del proxy `Connection` siano impostate sui valori seguenti e che `$connection_upgrade` sia mappato a uno dei seguenti valori:</span><span class="sxs-lookup"><span data-stu-id="710fc-167">For SignalR WebSockets to function properly, confirm that the proxy's `Upgrade` and `Connection` headers are set to the following values and that `$connection_upgrade` is mapped to either:</span></span>

* <span data-ttu-id="710fc-168">Valore dell'intestazione di aggiornamento per impostazione predefinita.</span><span class="sxs-lookup"><span data-stu-id="710fc-168">The Upgrade header value by default.</span></span>
* <span data-ttu-id="710fc-169">`close` Quando l'intestazione di aggiornamento è mancante o vuota.</span><span class="sxs-lookup"><span data-stu-id="710fc-169">`close` when the Upgrade header is missing or empty.</span></span>

```
http {
    map $http_upgrade $connection_upgrade {
        default Upgrade;
        ''      close;
    }

    server {
        listen      80;
        server_name example.com *.example.com
        location / {
            proxy_pass         http://localhost:5000;
            proxy_http_version 1.1;
            proxy_set_header   Upgrade $http_upgrade;
            proxy_set_header   Connection $connection_upgrade;
            proxy_set_header   Host $host;
            proxy_cache_bypass $http_upgrade;
            proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header   X-Forwarded-Proto $scheme;
        }
    }
}
```

<span data-ttu-id="710fc-170">Per altre informazioni, vedere gli articoli seguenti:</span><span class="sxs-lookup"><span data-stu-id="710fc-170">For more information, see the following articles:</span></span>

* [<span data-ttu-id="710fc-171">NGINX come proxy WebSocket</span><span class="sxs-lookup"><span data-stu-id="710fc-171">NGINX as a WebSocket Proxy</span></span>](https://www.nginx.com/blog/websocket-nginx/)
* [<span data-ttu-id="710fc-172">Proxy WebSocket</span><span class="sxs-lookup"><span data-stu-id="710fc-172">WebSocket proxying</span></span>](http://nginx.org/docs/http/websocket.html)
* <xref:host-and-deploy/linux-nginx>

## <a name="linux-with-apache"></a><span data-ttu-id="710fc-173">Linux con Apache</span><span class="sxs-lookup"><span data-stu-id="710fc-173">Linux with Apache</span></span>

<span data-ttu-id="710fc-174">Per ospitare un' Blazor app dietro Apache in Linux, configurare `ProxyPass` per il traffico HTTP e WebSocket.</span><span class="sxs-lookup"><span data-stu-id="710fc-174">To host a Blazor app behind Apache on Linux, configure `ProxyPass` for HTTP and WebSockets traffic.</span></span>

<span data-ttu-id="710fc-175">Nell'esempio seguente:</span><span class="sxs-lookup"><span data-stu-id="710fc-175">In the following example:</span></span>

* <span data-ttu-id="710fc-176">Il server gheppio è in esecuzione nel computer host.</span><span class="sxs-lookup"><span data-stu-id="710fc-176">Kestrel server is running on the host machine.</span></span>
* <span data-ttu-id="710fc-177">L'app è in ascolto del traffico sulla porta 5000.</span><span class="sxs-lookup"><span data-stu-id="710fc-177">The app listens for traffic on port 5000.</span></span>

```
ProxyRequests       On
ProxyPreserveHost   On
ProxyPassMatch      ^/_blazor/(.*) http://localhost:5000/_blazor/$1
ProxyPass           /_blazor ws://localhost:5000/_blazor
ProxyPass           / http://localhost:5000/
ProxyPassReverse    / http://localhost:5000/
```

<span data-ttu-id="710fc-178">Abilitare i moduli seguenti:</span><span class="sxs-lookup"><span data-stu-id="710fc-178">Enable the following modules:</span></span>

```
a2enmod   proxy
a2enmod   proxy_wstunnel
```

<span data-ttu-id="710fc-179">Controllare gli errori di WebSocket nella console del browser.</span><span class="sxs-lookup"><span data-stu-id="710fc-179">Check the browser console for WebSockets errors.</span></span> <span data-ttu-id="710fc-180">Errori di esempio:</span><span class="sxs-lookup"><span data-stu-id="710fc-180">Example errors:</span></span>

* <span data-ttu-id="710fc-181">Firefox non è in grado di stabilire una connessione al server in ws://the-domain-name.tld/_blazor?id=XXX.</span><span class="sxs-lookup"><span data-stu-id="710fc-181">Firefox can't establish a connection to the server at ws://the-domain-name.tld/_blazor?id=XXX.</span></span>
* <span data-ttu-id="710fc-182">Errore: non è stato possibile avviare il trasporto ' WebSocket ': errore: si è verificato un errore con il trasporto.</span><span class="sxs-lookup"><span data-stu-id="710fc-182">Error: Failed to start the transport 'WebSockets': Error: There was an error with the transport.</span></span>
* <span data-ttu-id="710fc-183">Errore: non è stato possibile avviare il trasporto ' LongPolling ': TypeError: il trasporto non è definito</span><span class="sxs-lookup"><span data-stu-id="710fc-183">Error: Failed to start the transport 'LongPolling': TypeError: this.transport is undefined</span></span>
* <span data-ttu-id="710fc-184">Errore: Impossibile connettersi al server con i trasporti disponibili.</span><span class="sxs-lookup"><span data-stu-id="710fc-184">Error: Unable to connect to the server with any of the available transports.</span></span> <span data-ttu-id="710fc-185">WebSocket non riusciti</span><span class="sxs-lookup"><span data-stu-id="710fc-185">WebSockets failed</span></span>
* <span data-ttu-id="710fc-186">Errore: Impossibile inviare dati se la connessione non è nello stato "connesso".</span><span class="sxs-lookup"><span data-stu-id="710fc-186">Error: Cannot send data if the connection is not in the 'Connected' State.</span></span>

<span data-ttu-id="710fc-187">Per ulteriori informazioni, vedere la [documentazione di Apache](https://httpd.apache.org/docs/current/mod/mod_proxy.html).</span><span class="sxs-lookup"><span data-stu-id="710fc-187">For more information, see the [Apache documentation](https://httpd.apache.org/docs/current/mod/mod_proxy.html).</span></span>

### <a name="measure-network-latency"></a><span data-ttu-id="710fc-188">Misurare la latenza di rete</span><span class="sxs-lookup"><span data-stu-id="710fc-188">Measure network latency</span></span>

<span data-ttu-id="710fc-189">L' [interoperabilità JS](xref:blazor/call-javascript-from-dotnet) può essere usata per misurare la latenza di rete, come illustrato nell'esempio seguente:</span><span class="sxs-lookup"><span data-stu-id="710fc-189">[JS interop](xref:blazor/call-javascript-from-dotnet) can be used to measure network latency, as the following example demonstrates:</span></span>

```razor
@inject IJSRuntime JS

@if (latency is null)
{
    <span>Calculating...</span>
}
else
{
    <span>@(latency.Value.TotalMilliseconds)ms</span>
}

@code {
    private DateTime startTime;
    private TimeSpan? latency;

    protected override async Task OnAfterRenderAsync(bool firstRender)
    {
        if (firstRender)
        {
            startTime = DateTime.UtcNow;
            var _ = await JS.InvokeAsync<string>("toString");
            latency = DateTime.UtcNow - startTime;
            StateHasChanged();
        }
    }
}
```

<span data-ttu-id="710fc-190">Per un'esperienza dell'interfaccia utente ragionevole, è consigliabile una latenza di interfaccia utente prolungata di 250ms o meno.</span><span class="sxs-lookup"><span data-stu-id="710fc-190">For a reasonable UI experience, we recommend a sustained UI latency of 250ms or less.</span></span>
