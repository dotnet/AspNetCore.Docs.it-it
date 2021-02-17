---
title: ASP.NET Core l' SignalR hosting e la scalabilità di produzione
author: bradygaster
description: Informazioni su come evitare problemi di prestazioni e scalabilità nelle app che usano ASP.NET Core SignalR .
monikerRange: '>= aspnetcore-2.1'
ms.author: bradyg
ms.custom: mvc
ms.date: 01/17/2020
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
uid: signalr/scale
ms.openlocfilehash: e70f3143159a1817e326a95b30e7369a5c9ab025
ms.sourcegitcommit: f77a7467651bab61b24261da9dc5c1dd75fc1fa9
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/17/2021
ms.locfileid: "100564007"
---
# <a name="aspnet-core-signalr-hosting-and-scaling"></a><span data-ttu-id="2433e-103">ASP.NET Core SignalR hosting e scalabilità</span><span class="sxs-lookup"><span data-stu-id="2433e-103">ASP.NET Core SignalR hosting and scaling</span></span>

<span data-ttu-id="2433e-104">Di [Andrew Stanton-Nurse](https://twitter.com/anurse), [Brady Gaster](https://twitter.com/bradygaster)e [Tom Dykstra](https://github.com/tdykstra)</span><span class="sxs-lookup"><span data-stu-id="2433e-104">By [Andrew Stanton-Nurse](https://twitter.com/anurse), [Brady Gaster](https://twitter.com/bradygaster), and [Tom Dykstra](https://github.com/tdykstra)</span></span>

<span data-ttu-id="2433e-105">Questo articolo illustra le considerazioni di hosting e scalabilità per le app con traffico elevato che usano ASP.NET Core SignalR .</span><span class="sxs-lookup"><span data-stu-id="2433e-105">This article explains hosting and scaling considerations for high-traffic apps that use ASP.NET Core SignalR.</span></span>

## <a name="sticky-sessions"></a><span data-ttu-id="2433e-106">Sessioni permanenti</span><span class="sxs-lookup"><span data-stu-id="2433e-106">Sticky Sessions</span></span>

<span data-ttu-id="2433e-107">SignalR richiede che tutte le richieste HTTP per una connessione specifica siano gestite dallo stesso processo server.</span><span class="sxs-lookup"><span data-stu-id="2433e-107">SignalR requires that all HTTP requests for a specific connection be handled by the same server process.</span></span> <span data-ttu-id="2433e-108">Quando SignalR è in esecuzione in un server farm (più server), è necessario usare "sessioni permanenti".</span><span class="sxs-lookup"><span data-stu-id="2433e-108">When SignalR is running on a server farm (multiple servers), "sticky sessions" must be used.</span></span> <span data-ttu-id="2433e-109">Le "sessioni permanenti" sono definite anche affinità di sessione da parte di alcuni bilanciamenti del carico.</span><span class="sxs-lookup"><span data-stu-id="2433e-109">"Sticky sessions" are also called session affinity by some load balancers.</span></span> <span data-ttu-id="2433e-110">Il servizio app Azure usa [Application Request Routing](/iis/extensions/planning-for-arr/application-request-routing-version-2-overview) (arr) per indirizzare le richieste.</span><span class="sxs-lookup"><span data-stu-id="2433e-110">Azure App Service uses [Application Request Routing](/iis/extensions/planning-for-arr/application-request-routing-version-2-overview) (ARR) to route requests.</span></span> <span data-ttu-id="2433e-111">L'abilitazione dell'impostazione "affinità ARR" nel servizio app Azure consentirà di "sessioni permanenti".</span><span class="sxs-lookup"><span data-stu-id="2433e-111">Enabling the "ARR Affinity" setting in your Azure App Service will enable "sticky sessions".</span></span> <span data-ttu-id="2433e-112">Le uniche circostanze in cui non sono richieste sessioni permanenti sono:</span><span class="sxs-lookup"><span data-stu-id="2433e-112">The only circumstances in which sticky sessions are not required are:</span></span>

1. <span data-ttu-id="2433e-113">Quando si esegue l'hosting in un singolo server, in un singolo processo.</span><span class="sxs-lookup"><span data-stu-id="2433e-113">When hosting on a single server, in a single process.</span></span>
1. <span data-ttu-id="2433e-114">Quando si usa il SignalR servizio di Azure.</span><span class="sxs-lookup"><span data-stu-id="2433e-114">When using the Azure SignalR Service.</span></span>
1. <span data-ttu-id="2433e-115">Quando tutti i client sono configurati in modo da usare **solo** WebSocket **e** l' [impostazione SkipNegotiation](xref:signalr/configuration#configure-additional-options) è abilitata nella configurazione client.</span><span class="sxs-lookup"><span data-stu-id="2433e-115">When all clients are configured to **only** use WebSockets, **and** the [SkipNegotiation setting](xref:signalr/configuration#configure-additional-options) is enabled in the client configuration.</span></span>

<span data-ttu-id="2433e-116">In tutti gli altri casi (incluso quando viene usato il backplane Redis), è necessario configurare l'ambiente server per le sessioni permanenti.</span><span class="sxs-lookup"><span data-stu-id="2433e-116">In all other circumstances (including when the Redis backplane is used), the server environment must be configured for sticky sessions.</span></span>

<span data-ttu-id="2433e-117">Per informazioni sulla configurazione del servizio app Azure per SignalR , vedere <xref:signalr/publish-to-azure-web-app> .</span><span class="sxs-lookup"><span data-stu-id="2433e-117">For guidance on configuring Azure App Service for SignalR, see <xref:signalr/publish-to-azure-web-app>.</span></span>

## <a name="tcp-connection-resources"></a><span data-ttu-id="2433e-118">Risorse di connessione TCP</span><span class="sxs-lookup"><span data-stu-id="2433e-118">TCP connection resources</span></span>

<span data-ttu-id="2433e-119">Il numero di connessioni TCP simultanee che un server Web è in grado di supportare è limitato.</span><span class="sxs-lookup"><span data-stu-id="2433e-119">The number of concurrent TCP connections that a web server can support is limited.</span></span> <span data-ttu-id="2433e-120">I client HTTP standard *utilizzano connessioni* temporanee.</span><span class="sxs-lookup"><span data-stu-id="2433e-120">Standard HTTP clients use *ephemeral* connections.</span></span> <span data-ttu-id="2433e-121">Queste connessioni possono essere chiuse quando il client diventa inattivo e riaperto in un secondo momento.</span><span class="sxs-lookup"><span data-stu-id="2433e-121">These connections can be closed when the client goes idle and reopened later.</span></span> <span data-ttu-id="2433e-122">D'altra parte, una SignalR connessione è *persistente*.</span><span class="sxs-lookup"><span data-stu-id="2433e-122">On the other hand, a SignalR connection is *persistent*.</span></span> <span data-ttu-id="2433e-123">SignalR le connessioni resteranno aperte anche quando il client diventa inattivo.</span><span class="sxs-lookup"><span data-stu-id="2433e-123">SignalR connections stay open even when the client goes idle.</span></span> <span data-ttu-id="2433e-124">In un'app a traffico elevato che serve molti client, queste connessioni permanenti possono provocare il raggiungimento del numero massimo di connessioni da parte dei server.</span><span class="sxs-lookup"><span data-stu-id="2433e-124">In a high-traffic app that serves many clients, these persistent connections can cause servers to hit their maximum number of connections.</span></span>

<span data-ttu-id="2433e-125">Le connessioni permanenti utilizzano anche una quantità di memoria aggiuntiva per tenere traccia di ogni connessione.</span><span class="sxs-lookup"><span data-stu-id="2433e-125">Persistent connections also consume some additional memory, to track each connection.</span></span>

<span data-ttu-id="2433e-126">L'utilizzo intensivo delle risorse correlate alla connessione SignalR può influire su altre app Web ospitate nello stesso server.</span><span class="sxs-lookup"><span data-stu-id="2433e-126">The heavy use of connection-related resources by SignalR can affect other web apps that are hosted on the same server.</span></span> <span data-ttu-id="2433e-127">Quando SignalR si apre e contiene le ultime connessioni TCP disponibili, anche altre app Web nello stesso server non dispongono di altre connessioni disponibili.</span><span class="sxs-lookup"><span data-stu-id="2433e-127">When SignalR opens and holds the last available TCP connections, other web apps on the same server also have no more connections available to them.</span></span>

<span data-ttu-id="2433e-128">Se un server esaurisce le connessioni, verranno visualizzati gli errori di socket casuali e gli errori di reimpostazione della connessione.</span><span class="sxs-lookup"><span data-stu-id="2433e-128">If a server runs out of connections, you'll see random socket errors and connection reset errors.</span></span> <span data-ttu-id="2433e-129">Ad esempio:</span><span class="sxs-lookup"><span data-stu-id="2433e-129">For example:</span></span>

```
An attempt was made to access a socket in a way forbidden by its access permissions...
```

<span data-ttu-id="2433e-130">Per evitare SignalR che l'utilizzo delle risorse causi errori in altre app Web, eseguire SignalR su server diversi rispetto alle altre app Web.</span><span class="sxs-lookup"><span data-stu-id="2433e-130">To keep SignalR resource usage from causing errors in other web apps, run SignalR on different servers than your other web apps.</span></span>

<span data-ttu-id="2433e-131">Per evitare che SignalR l'utilizzo delle risorse causi errori in un' SignalR app, aumentare il numero di connessioni per limitare il numero di connessioni che un server deve gestire.</span><span class="sxs-lookup"><span data-stu-id="2433e-131">To keep SignalR resource usage from causing errors in a SignalR app, scale out to limit the number of connections a server has to handle.</span></span>

## <a name="scale-out"></a><span data-ttu-id="2433e-132">Aumentare il numero di istanze</span><span class="sxs-lookup"><span data-stu-id="2433e-132">Scale out</span></span>

<span data-ttu-id="2433e-133">Un'app che usa SignalR deve tenere traccia di tutte le connessioni, che creano problemi per un server farm.</span><span class="sxs-lookup"><span data-stu-id="2433e-133">An app that uses SignalR needs to keep track of all its connections, which creates problems for a server farm.</span></span> <span data-ttu-id="2433e-134">Aggiungere un server e ottenere nuove connessioni che gli altri server non conoscono.</span><span class="sxs-lookup"><span data-stu-id="2433e-134">Add a server, and it gets new connections that the other servers don't know about.</span></span> <span data-ttu-id="2433e-135">Ad esempio, in SignalR ogni server del diagramma seguente non è a conoscenza delle connessioni sugli altri server.</span><span class="sxs-lookup"><span data-stu-id="2433e-135">For example, SignalR on each server in the following diagram is unaware of the connections on the other servers.</span></span> <span data-ttu-id="2433e-136">Quando SignalR in uno dei server si desidera inviare un messaggio a tutti i client, il messaggio passa solo ai client connessi a tale server.</span><span class="sxs-lookup"><span data-stu-id="2433e-136">When SignalR on one of the servers wants to send a message to all clients, the message only goes to the clients connected to that server.</span></span>

![Ridimensionamento::: NO-LOC (SignalR)::: senza un backplane](scale/_static/scale-no-backplane.png)

<span data-ttu-id="2433e-138">Le opzioni per la risoluzione di questo problema sono [il SignalR servizio di Azure](#azure-signalr-service) e il [backplane di redis](#redis-backplane).</span><span class="sxs-lookup"><span data-stu-id="2433e-138">The options for solving this problem are the [Azure SignalR Service](#azure-signalr-service) and [Redis backplane](#redis-backplane).</span></span>

## <a name="azure-signalr-service"></a><span data-ttu-id="2433e-139">Servizio di Azure SignalR</span><span class="sxs-lookup"><span data-stu-id="2433e-139">Azure SignalR Service</span></span>

<span data-ttu-id="2433e-140">Il SignalR servizio di Azure è un proxy anziché un backplane.</span><span class="sxs-lookup"><span data-stu-id="2433e-140">The Azure SignalR Service is a proxy rather than a backplane.</span></span> <span data-ttu-id="2433e-141">Ogni volta che un client avvia una connessione al server, il client viene reindirizzato per connettersi al servizio.</span><span class="sxs-lookup"><span data-stu-id="2433e-141">Each time a client initiates a connection to the server, the client is redirected to connect to the service.</span></span> <span data-ttu-id="2433e-142">Questo processo è illustrato nel diagramma seguente:</span><span class="sxs-lookup"><span data-stu-id="2433e-142">That process is illustrated in the following diagram:</span></span>

![Stabilire una connessione al servizio Azure::: NO-LOC (SignalR)::: Service](scale/_static/azure-signalr-service-one-connection.png)

<span data-ttu-id="2433e-144">Il risultato è che il servizio gestisce tutte le connessioni client, mentre per ogni server è necessario solo un piccolo numero costante di connessioni al servizio, come illustrato nel diagramma seguente:</span><span class="sxs-lookup"><span data-stu-id="2433e-144">The result is that the service manages all of the client connections, while each server needs only a small constant number of connections to the service, as shown in the following diagram:</span></span>

![Client connessi al servizio, server connessi al servizio](scale/_static/azure-signalr-service-multiple-connections.png)

<span data-ttu-id="2433e-146">Questo approccio per la scalabilità orizzontale presenta diversi vantaggi rispetto all'alternativa del backplane redis:</span><span class="sxs-lookup"><span data-stu-id="2433e-146">This approach to scale-out has several advantages over the Redis backplane alternative:</span></span>

* <span data-ttu-id="2433e-147">Le sessioni permanenti, note anche come [affinità client](/iis/extensions/configuring-application-request-routing-arr/http-load-balancing-using-application-request-routing#step-3---configure-client-affinity), non sono necessarie, perché i client vengono immediatamente reindirizzati al servizio di Azure SignalR quando si connettono.</span><span class="sxs-lookup"><span data-stu-id="2433e-147">Sticky sessions, also known as [client affinity](/iis/extensions/configuring-application-request-routing-arr/http-load-balancing-using-application-request-routing#step-3---configure-client-affinity), is not required, because clients are immediately redirected to the Azure SignalR Service when they connect.</span></span>
* <span data-ttu-id="2433e-148">Un' SignalR app può aumentare la scalabilità orizzontale in base al numero di messaggi inviati, mentre il servizio di Azure viene SignalR ridimensionato per gestire un numero qualsiasi di connessioni.</span><span class="sxs-lookup"><span data-stu-id="2433e-148">A SignalR app can scale out based on the number of messages sent, while the Azure SignalR Service scales to handle any number of connections.</span></span> <span data-ttu-id="2433e-149">Ad esempio, potrebbero essere presenti migliaia di client, ma se vengono inviati solo pochi messaggi al secondo, l' SignalR app non deve essere scalata in più server solo per gestire le connessioni stesse.</span><span class="sxs-lookup"><span data-stu-id="2433e-149">For example, there could be thousands of clients, but if only a few messages per second are sent, the SignalR app won't need to scale out to multiple servers just to handle the connections themselves.</span></span>
* <span data-ttu-id="2433e-150">Un' SignalR app non userà significativamente più risorse di connessione rispetto a un'app Web senza SignalR .</span><span class="sxs-lookup"><span data-stu-id="2433e-150">A SignalR app won't use significantly more connection resources than a web app without SignalR.</span></span>

<span data-ttu-id="2433e-151">Per questi motivi, è consigliabile usare il SignalR servizio Azure per tutte le SignalR app ASP.NET Core ospitate in Azure, tra cui il servizio app, le macchine virtuali e i contenitori.</span><span class="sxs-lookup"><span data-stu-id="2433e-151">For these reasons, we recommend the Azure SignalR Service for all ASP.NET Core SignalR apps hosted on Azure, including App Service, VMs, and containers.</span></span>

<span data-ttu-id="2433e-152">Per altre informazioni, vedere [la SignalR documentazione dei servizi di Azure](/azure/azure-signalr/signalr-overview).</span><span class="sxs-lookup"><span data-stu-id="2433e-152">For more information see the [Azure SignalR Service documentation](/azure/azure-signalr/signalr-overview).</span></span>

## <a name="redis-backplane"></a><span data-ttu-id="2433e-153">Backplane Redis</span><span class="sxs-lookup"><span data-stu-id="2433e-153">Redis backplane</span></span>

<span data-ttu-id="2433e-154">[Redis](https://redis.io/) è un archivio chiave-valore in memoria che supporta un sistema di messaggistica con un modello di pubblicazione/sottoscrizione.</span><span class="sxs-lookup"><span data-stu-id="2433e-154">[Redis](https://redis.io/) is an in-memory key-value store that supports a messaging system with a publish/subscribe model.</span></span> <span data-ttu-id="2433e-155">Il SignalR backplane Redis usa la funzionalità di pubblicazione/sottoscrizione per l'invio di messaggi ad altri server.</span><span class="sxs-lookup"><span data-stu-id="2433e-155">The SignalR Redis backplane uses the pub/sub feature to forward messages to other servers.</span></span> <span data-ttu-id="2433e-156">Quando un client effettua una connessione, le informazioni di connessione vengono passate al backplane.</span><span class="sxs-lookup"><span data-stu-id="2433e-156">When a client makes a connection, the connection information is passed to the backplane.</span></span> <span data-ttu-id="2433e-157">Quando un server vuole inviare un messaggio a tutti i client, lo invia al backplane.</span><span class="sxs-lookup"><span data-stu-id="2433e-157">When a server wants to send a message to all clients, it sends to the backplane.</span></span> <span data-ttu-id="2433e-158">Il backplane conosce tutti i client connessi e i server in cui si trovano.</span><span class="sxs-lookup"><span data-stu-id="2433e-158">The backplane knows all connected clients and which servers they're on.</span></span> <span data-ttu-id="2433e-159">Invia il messaggio a tutti i client tramite i rispettivi server.</span><span class="sxs-lookup"><span data-stu-id="2433e-159">It sends the message to all clients via their respective servers.</span></span> <span data-ttu-id="2433e-160">Questo processo è illustrato nel diagramma seguente:</span><span class="sxs-lookup"><span data-stu-id="2433e-160">This process is illustrated in the following diagram:</span></span>

![Backplane Redis, messaggio inviato da un server a tutti i client](scale/_static/redis-backplane.png)

<span data-ttu-id="2433e-162">Il backplane di redis è l'approccio di scalabilità orizzontale consigliato per le app ospitate nella propria infrastruttura.</span><span class="sxs-lookup"><span data-stu-id="2433e-162">The Redis backplane is the recommended scale-out approach for apps hosted on your own infrastructure.</span></span> <span data-ttu-id="2433e-163">Se è presente una latenza di connessione significativa tra il data center e un data center di Azure, il SignalR servizio di Azure potrebbe non essere un'opzione pratica per le app locali con requisiti di bassa latenza o velocità effettiva elevata.</span><span class="sxs-lookup"><span data-stu-id="2433e-163">If there is significant connection latency between your data center and an Azure data center, Azure SignalR Service may not be a practical option for on-premises apps with low latency or high throughput requirements.</span></span>

<span data-ttu-id="2433e-164">I SignalR vantaggi dei servizi di Azure indicati in precedenza sono gli svantaggi per il backplane di redis:</span><span class="sxs-lookup"><span data-stu-id="2433e-164">The Azure SignalR Service advantages noted earlier are disadvantages for the Redis backplane:</span></span>

* <span data-ttu-id="2433e-165">Le sessioni permanenti, note anche come [affinità client](/iis/extensions/configuring-application-request-routing-arr/http-load-balancing-using-application-request-routing#step-3---configure-client-affinity), sono necessarie, tranne quando si verificano **entrambe** le condizioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="2433e-165">Sticky sessions, also known as [client affinity](/iis/extensions/configuring-application-request-routing-arr/http-load-balancing-using-application-request-routing#step-3---configure-client-affinity), is required, except when **both** of the following are true:</span></span>
  * <span data-ttu-id="2433e-166">Tutti i client sono configurati in modo da usare **solo** WebSocket.</span><span class="sxs-lookup"><span data-stu-id="2433e-166">All clients are configured to **only** use WebSockets.</span></span>
  * <span data-ttu-id="2433e-167">L' [impostazione SkipNegotiation](xref:signalr/configuration#configure-additional-options) è abilitata nella configurazione client.</span><span class="sxs-lookup"><span data-stu-id="2433e-167">The [SkipNegotiation setting](xref:signalr/configuration#configure-additional-options) is enabled in the client configuration.</span></span> 
   <span data-ttu-id="2433e-168">Una volta avviata una connessione in un server, la connessione deve rimanere nel server.</span><span class="sxs-lookup"><span data-stu-id="2433e-168">Once a connection is initiated on a server, the connection has to stay on that server.</span></span>
* <span data-ttu-id="2433e-169">Un' SignalR applicazione deve essere scalata in base al numero di client anche se sono stati inviati pochi messaggi.</span><span class="sxs-lookup"><span data-stu-id="2433e-169">A SignalR app must scale out based on number of clients even if few messages are being sent.</span></span>
* <span data-ttu-id="2433e-170">Un' SignalR app usa significativamente più risorse di connessione rispetto a un'app Web senza SignalR .</span><span class="sxs-lookup"><span data-stu-id="2433e-170">A SignalR app uses significantly more connection resources than a web app without SignalR.</span></span>

## <a name="iis-limitations-on-windows-client-os"></a><span data-ttu-id="2433e-171">Limitazioni di IIS sul sistema operativo client Windows</span><span class="sxs-lookup"><span data-stu-id="2433e-171">IIS limitations on Windows client OS</span></span>

<span data-ttu-id="2433e-172">Windows 10 e Windows 8. x sono sistemi operativi client.</span><span class="sxs-lookup"><span data-stu-id="2433e-172">Windows 10 and Windows 8.x are client operating systems.</span></span> <span data-ttu-id="2433e-173">IIS nei sistemi operativi client ha un limite di 10 connessioni simultanee.</span><span class="sxs-lookup"><span data-stu-id="2433e-173">IIS on client operating systems has a limit of 10 concurrent connections.</span></span> <span data-ttu-id="2433e-174">SignalRle connessioni sono:</span><span class="sxs-lookup"><span data-stu-id="2433e-174">SignalR's connections are:</span></span>

* <span data-ttu-id="2433e-175">Temporaneo e ristabilito di frequente.</span><span class="sxs-lookup"><span data-stu-id="2433e-175">Transient and frequently re-established.</span></span>
* <span data-ttu-id="2433e-176">**Non** viene eliminato immediatamente quando non viene più utilizzato.</span><span class="sxs-lookup"><span data-stu-id="2433e-176">**Not** disposed immediately when no longer used.</span></span>

<span data-ttu-id="2433e-177">Le condizioni precedenti consentono di raggiungere il limite di 10 connessioni in un sistema operativo client.</span><span class="sxs-lookup"><span data-stu-id="2433e-177">The preceding conditions make it likely to hit the 10 connection limit on a client OS.</span></span> <span data-ttu-id="2433e-178">Quando un sistema operativo client viene usato per lo sviluppo, si consiglia quanto segue:</span><span class="sxs-lookup"><span data-stu-id="2433e-178">When a client OS is used for development, we recommend:</span></span>

* <span data-ttu-id="2433e-179">Evitare IIS.</span><span class="sxs-lookup"><span data-stu-id="2433e-179">Avoid IIS.</span></span>
* <span data-ttu-id="2433e-180">Usare gheppio o IIS Express come destinazioni di distribuzione.</span><span class="sxs-lookup"><span data-stu-id="2433e-180">Use Kestrel or IIS Express as deployment targets.</span></span>

## <a name="linux-with-nginx"></a><span data-ttu-id="2433e-181">Linux con Nginx</span><span class="sxs-lookup"><span data-stu-id="2433e-181">Linux with Nginx</span></span>

<span data-ttu-id="2433e-182">Di seguito sono riportate le impostazioni minime obbligatorie per abilitare WebSockets, ServerSentEvents e LongPolling per SignalR :</span><span class="sxs-lookup"><span data-stu-id="2433e-182">The following contains the minimum required settings to enable WebSockets, ServerSentEvents, and LongPolling for SignalR:</span></span>

```nginx
http {
  map $http_connection $connection_upgrade {
    "~*Upgrade" $http_connection;
    default keep-alive;
}

  server {
    listen 80;
    server_name example.com *.example.com;

    # Configure the SignalR Endpoint
    location /hubroute {
      # App server url
      proxy_pass http://localhost:5000;

      # Configuration for WebSockets
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection $connection_upgrade;
      proxy_cache off;

      # Configuration for ServerSentEvents
      proxy_buffering off;

      # Configuration for LongPolling or if your KeepAliveInterval is longer than 60 seconds
      proxy_read_timeout 100s;

      proxy_set_header Host $host;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header X-Forwarded-Proto $scheme;
    }
  }
}
```

<span data-ttu-id="2433e-183">Quando si utilizzano più server back-end, è necessario aggiungere sessioni permanenti per impedire che le SignalR connessioni cambino server durante la connessione.</span><span class="sxs-lookup"><span data-stu-id="2433e-183">When multiple backend servers are used, sticky sessions must be added to prevent SignalR connections from switching servers when connecting.</span></span> <span data-ttu-id="2433e-184">Sono disponibili diversi modi per aggiungere sessioni permanenti in nginx.</span><span class="sxs-lookup"><span data-stu-id="2433e-184">There are multiple ways to add sticky sessions in Nginx.</span></span> <span data-ttu-id="2433e-185">Di seguito sono riportati due approcci, a seconda di ciò che è disponibile.</span><span class="sxs-lookup"><span data-stu-id="2433e-185">Two approaches are shown below depending on what you have available.</span></span>

<span data-ttu-id="2433e-186">Oltre alla configurazione precedente, viene aggiunto quanto segue.</span><span class="sxs-lookup"><span data-stu-id="2433e-186">The following is added in addition to the previous configuration.</span></span> <span data-ttu-id="2433e-187">Negli esempi seguenti `backend` è il nome del gruppo di server.</span><span class="sxs-lookup"><span data-stu-id="2433e-187">In the following examples, `backend` is the name of the group of servers.</span></span>

<span data-ttu-id="2433e-188">Con l' [Open Source nginx](https://nginx.org/en/), usare `ip_hash` per instradare le connessioni a un server in base all'indirizzo IP del client:</span><span class="sxs-lookup"><span data-stu-id="2433e-188">With [Nginx Open Source](https://nginx.org/en/), use `ip_hash` to route connections to a server based on the client's IP address:</span></span>

```nginx
http {
  upstream backend {
    # App server 1
    server http://localhost:5000;
    # App server 2
    server http://localhost:5002;

    ip_hash;
  }
}
```

<span data-ttu-id="2433e-189">Con [nginx Plus](https://www.nginx.com/products/nginx), usare `sticky` per aggiungere un cookie a richieste e aggiungere le richieste dell'utente a un server:</span><span class="sxs-lookup"><span data-stu-id="2433e-189">With [Nginx Plus](https://www.nginx.com/products/nginx), use `sticky` to add a cookie to requests and pin the user's requests to a server:</span></span>

```nginx
http {
  upstream backend {
    # App server 1
    server http://localhost:5000;
    # App server 2
    server http://localhost:5002;

    sticky cookie srv_id expires=max domain=.example.com path=/ httponly;
  }
}
```

<span data-ttu-id="2433e-190">Infine, modificare `proxy_pass http://localhost:5000` la `server` sezione in `proxy_pass http://backend` .</span><span class="sxs-lookup"><span data-stu-id="2433e-190">Finally, change `proxy_pass http://localhost:5000` in the `server` section to `proxy_pass http://backend`.</span></span>

<span data-ttu-id="2433e-191">Per altre informazioni sui WebSocket su nginx, vedere [nginx come proxy WebSocket](https://www.nginx.com/blog/websocket-nginx).</span><span class="sxs-lookup"><span data-stu-id="2433e-191">For more information on WebSockets over Nginx, see [NGINX as a WebSocket Proxy](https://www.nginx.com/blog/websocket-nginx).</span></span>

<span data-ttu-id="2433e-192">Per altre informazioni sul bilanciamento del carico e le sessioni permanenti, vedere [nginx Load Balancing](https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/).</span><span class="sxs-lookup"><span data-stu-id="2433e-192">For more information on load balancing and sticky sessions, see [NGINX load balancing](https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/).</span></span>

<span data-ttu-id="2433e-193">Per ulteriori informazioni su ASP.NET Core con nginx, vedere l'articolo seguente:</span><span class="sxs-lookup"><span data-stu-id="2433e-193">For more information about ASP.NET Core with Nginx see the following article:</span></span>
* <xref:host-and-deploy/linux-nginx>

## <a name="third-party-signalr-backplane-providers"></a><span data-ttu-id="2433e-194">Provider backplane di terze parti SignalR</span><span class="sxs-lookup"><span data-stu-id="2433e-194">Third-party SignalR backplane providers</span></span>

* [<span data-ttu-id="2433e-195">NCache</span><span class="sxs-lookup"><span data-stu-id="2433e-195">NCache</span></span>](https://www.alachisoft.com/ncache/asp-net-core-signalr.html)
* <span data-ttu-id="2433e-196">[Orleans](https://github.com/OrleansContrib/SignalR.Orleans)</span><span class="sxs-lookup"><span data-stu-id="2433e-196">[Orleans](https://github.com/OrleansContrib/SignalR.Orleans)</span></span>
* <span data-ttu-id="2433e-197">[Rebus](https://github.com/rebus-org/Rebus.SignalR)</span><span class="sxs-lookup"><span data-stu-id="2433e-197">[Rebus](https://github.com/rebus-org/Rebus.SignalR)</span></span>

## <a name="next-steps"></a><span data-ttu-id="2433e-198">Passaggi successivi</span><span class="sxs-lookup"><span data-stu-id="2433e-198">Next steps</span></span>

<span data-ttu-id="2433e-199">Per altre informazioni, vedere le seguenti risorse:</span><span class="sxs-lookup"><span data-stu-id="2433e-199">For more information, see the following resources:</span></span>

* [<span data-ttu-id="2433e-200">SignalRDocumentazione del servizio Azure</span><span class="sxs-lookup"><span data-stu-id="2433e-200">Azure SignalR Service documentation</span></span>](/azure/azure-signalr/signalr-overview)
* [<span data-ttu-id="2433e-201">Configurare un backplane Redis</span><span class="sxs-lookup"><span data-stu-id="2433e-201">Set up a Redis backplane</span></span>](xref:signalr/redis-backplane)
