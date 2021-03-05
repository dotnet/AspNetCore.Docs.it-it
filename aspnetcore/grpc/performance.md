---
title: Procedure consigliate per le prestazioni con gRPC
author: jamesnk
description: Informazioni sulle procedure consigliate per la creazione di servizi gRPC ad alte prestazioni.
monikerRange: '>= aspnetcore-3.0'
ms.author: jamesnk
ms.date: 08/23/2020
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
uid: grpc/performance
ms.openlocfilehash: 386359adfe0e876fa5c067dc82153fd3de0f3fba
ms.sourcegitcommit: 3982ff9dabb5b12aeb0a61cde2686b5253364f5d
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/04/2021
ms.locfileid: "102118941"
---
# <a name="performance-best-practices-with-grpc"></a><span data-ttu-id="53114-103">Procedure consigliate per le prestazioni con gRPC</span><span class="sxs-lookup"><span data-stu-id="53114-103">Performance best practices with gRPC</span></span>

<span data-ttu-id="53114-104">Di [James Newton-King](https://twitter.com/jamesnk)</span><span class="sxs-lookup"><span data-stu-id="53114-104">By [James Newton-King](https://twitter.com/jamesnk)</span></span>

<span data-ttu-id="53114-105">gRPC è progettato per servizi a prestazioni elevate.</span><span class="sxs-lookup"><span data-stu-id="53114-105">gRPC is designed for high-performance services.</span></span> <span data-ttu-id="53114-106">Questo documento illustra come ottenere le migliori prestazioni possibili da gRPC.</span><span class="sxs-lookup"><span data-stu-id="53114-106">This document explains how to get the best performance possible from gRPC.</span></span>

## <a name="reuse-grpc-channels"></a><span data-ttu-id="53114-107">Riutilizzo di canali gRPC</span><span class="sxs-lookup"><span data-stu-id="53114-107">Reuse gRPC channels</span></span>

<span data-ttu-id="53114-108">Quando si effettuano chiamate gRPC è necessario riutilizzare un canale gRPC.</span><span class="sxs-lookup"><span data-stu-id="53114-108">A gRPC channel should be reused when making gRPC calls.</span></span> <span data-ttu-id="53114-109">Il riutilizzo di un canale consente il multiplexing delle chiamate tramite una connessione HTTP/2 esistente.</span><span class="sxs-lookup"><span data-stu-id="53114-109">Reusing a channel allows calls to be multiplexed through an existing HTTP/2 connection.</span></span>

<span data-ttu-id="53114-110">Se viene creato un nuovo canale per ogni chiamata gRPC, la quantità di tempo necessaria per il completamento può aumentare in modo significativo.</span><span class="sxs-lookup"><span data-stu-id="53114-110">If a new channel is created for each gRPC call then the amount of time it takes to complete can increase significantly.</span></span> <span data-ttu-id="53114-111">Per ogni chiamata sono necessari più round trip di rete tra il client e il server per creare una nuova connessione HTTP/2:</span><span class="sxs-lookup"><span data-stu-id="53114-111">Each call will require multiple network round-trips between the client and the server to create a new HTTP/2 connection:</span></span>

1. <span data-ttu-id="53114-112">Apertura di un socket</span><span class="sxs-lookup"><span data-stu-id="53114-112">Opening a socket</span></span>
2. <span data-ttu-id="53114-113">Creazione della connessione TCP</span><span class="sxs-lookup"><span data-stu-id="53114-113">Establishing TCP connection</span></span>
3. <span data-ttu-id="53114-114">Negoziazione di TLS</span><span class="sxs-lookup"><span data-stu-id="53114-114">Negotiating TLS</span></span>
4. <span data-ttu-id="53114-115">Avvio della connessione HTTP/2</span><span class="sxs-lookup"><span data-stu-id="53114-115">Starting HTTP/2 connection</span></span>
5. <span data-ttu-id="53114-116">Creazione della chiamata gRPC</span><span class="sxs-lookup"><span data-stu-id="53114-116">Making the gRPC call</span></span>

<span data-ttu-id="53114-117">I canali possono essere condivisi e riutilizzati in modo sicuro tra le chiamate a gRPC:</span><span class="sxs-lookup"><span data-stu-id="53114-117">Channels are safe to share and reuse between gRPC calls:</span></span>

* <span data-ttu-id="53114-118">i client gRPC vengono creati con i canali.</span><span class="sxs-lookup"><span data-stu-id="53114-118">gRPC clients are created with channels.</span></span> <span data-ttu-id="53114-119">i client gRPC sono oggetti leggeri e non devono essere memorizzati nella cache o riutilizzati.</span><span class="sxs-lookup"><span data-stu-id="53114-119">gRPC clients are lightweight objects and don't need to be cached or reused.</span></span>
* <span data-ttu-id="53114-120">È possibile creare più client gRPC da un canale, inclusi tipi diversi di client.</span><span class="sxs-lookup"><span data-stu-id="53114-120">Multiple gRPC clients can be created from a channel, including different types of clients.</span></span>
* <span data-ttu-id="53114-121">Un canale e i client creati dal canale possono essere usati in modo sicuro da più thread.</span><span class="sxs-lookup"><span data-stu-id="53114-121">A channel and clients created from the channel can safely be used by multiple threads.</span></span>
* <span data-ttu-id="53114-122">I client creati dal canale possono effettuare più chiamate simultanee.</span><span class="sxs-lookup"><span data-stu-id="53114-122">Clients created from the channel can make multiple simultaneous calls.</span></span>

<span data-ttu-id="53114-123">la factory client di gRPC offre un modo centralizzato per configurare i canali.</span><span class="sxs-lookup"><span data-stu-id="53114-123">gRPC client factory offers a centralized way to configure channels.</span></span> <span data-ttu-id="53114-124">Riutilizza automaticamente i canali sottostanti.</span><span class="sxs-lookup"><span data-stu-id="53114-124">It automatically reuses underlying channels.</span></span> <span data-ttu-id="53114-125">Per altre informazioni, vedere <xref:grpc/clientfactory>.</span><span class="sxs-lookup"><span data-stu-id="53114-125">For more information, see <xref:grpc/clientfactory>.</span></span>

## <a name="connection-concurrency"></a><span data-ttu-id="53114-126">Concorrenza di connessione</span><span class="sxs-lookup"><span data-stu-id="53114-126">Connection concurrency</span></span>

<span data-ttu-id="53114-127">Le connessioni HTTP/2 in genere presentano un limite al numero [massimo di flussi simultanei (richieste HTTP attive)](https://httpwg.github.io/specs/rfc7540.html#rfc.section.5.1.2) in una connessione in una sola volta.</span><span class="sxs-lookup"><span data-stu-id="53114-127">HTTP/2 connections typically have a limit on the number of [maximum concurrent streams (active HTTP requests)](https://httpwg.github.io/specs/rfc7540.html#rfc.section.5.1.2) on a connection at one time.</span></span> <span data-ttu-id="53114-128">Per impostazione predefinita, la maggior parte dei server imposta questo limite su 100 flussi simultanei.</span><span class="sxs-lookup"><span data-stu-id="53114-128">By default, most servers set this limit to 100 concurrent streams.</span></span>

<span data-ttu-id="53114-129">Un canale gRPC usa una singola connessione HTTP/2 e le chiamate simultanee vengono multiplexate su tale connessione.</span><span class="sxs-lookup"><span data-stu-id="53114-129">A gRPC channel uses a single HTTP/2 connection, and concurrent calls are multiplexed on that connection.</span></span> <span data-ttu-id="53114-130">Quando il numero di chiamate attive raggiunge il limite del flusso di connessione, le chiamate aggiuntive vengono accodate nel client.</span><span class="sxs-lookup"><span data-stu-id="53114-130">When the number of active calls reaches the connection stream limit, additional calls are queued in the client.</span></span> <span data-ttu-id="53114-131">Le chiamate in coda attendono il completamento delle chiamate attive prima di essere inviate.</span><span class="sxs-lookup"><span data-stu-id="53114-131">Queued calls wait for active calls to complete before they are sent.</span></span> <span data-ttu-id="53114-132">Le applicazioni con carico elevato, o chiamate gRPC di streaming con esecuzione prolungata, potrebbero riscontrare problemi di prestazioni causati dall'accodamento delle chiamate a causa di questo limite.</span><span class="sxs-lookup"><span data-stu-id="53114-132">Applications with high load, or long running streaming gRPC calls, could see performance issues caused by calls queuing because of this limit.</span></span>

::: moniker range=">= aspnetcore-5.0"

<span data-ttu-id="53114-133">In .NET 5 è stata introdotta la `SocketsHttpHandler.EnableMultipleHttp2Connections` Proprietà.</span><span class="sxs-lookup"><span data-stu-id="53114-133">.NET 5 introduces the `SocketsHttpHandler.EnableMultipleHttp2Connections` property.</span></span> <span data-ttu-id="53114-134">Quando è impostato su `true` , le connessioni http/2 aggiuntive vengono create da un canale quando viene raggiunto il limite di flusso simultaneo.</span><span class="sxs-lookup"><span data-stu-id="53114-134">When set to `true`, additional HTTP/2 connections are created by a channel when the concurrent stream limit is reached.</span></span> <span data-ttu-id="53114-135">Quando `GrpcChannel` viene creato un oggetto, il relativo interno `SocketsHttpHandler` viene configurato automaticamente per creare connessioni http/2 aggiuntive.</span><span class="sxs-lookup"><span data-stu-id="53114-135">When a `GrpcChannel` is created its internal `SocketsHttpHandler` is automatically configured to create additional HTTP/2 connections.</span></span> <span data-ttu-id="53114-136">Se un'app configura il proprio gestore, provare a impostare `EnableMultipleHttp2Connections` su `true` :</span><span class="sxs-lookup"><span data-stu-id="53114-136">If an app configures its own handler, consider setting `EnableMultipleHttp2Connections` to `true`:</span></span>

```csharp
var channel = GrpcChannel.ForAddress("https://localhost", new GrpcChannelOptions
{
    HttpHandler = new SocketsHttpHandler
    {
        EnableMultipleHttp2Connections = true,

        // ...configure other handler settings
    }
});
```

::: moniker-end

<span data-ttu-id="53114-137">Esistono un paio di soluzioni alternative per le app .NET Core 3,1:</span><span class="sxs-lookup"><span data-stu-id="53114-137">There are a couple of workarounds for .NET Core 3.1 apps:</span></span>

* <span data-ttu-id="53114-138">Creare canali gRPC distinti per le aree dell'app con carico elevato.</span><span class="sxs-lookup"><span data-stu-id="53114-138">Create separate gRPC channels for areas of the app with high load.</span></span> <span data-ttu-id="53114-139">Ad esempio, il `Logger` servizio gRPC potrebbe avere un carico elevato.</span><span class="sxs-lookup"><span data-stu-id="53114-139">For example, the `Logger` gRPC service might have a high load.</span></span> <span data-ttu-id="53114-140">Usare un canale separato per creare il `LoggerClient` nell'app.</span><span class="sxs-lookup"><span data-stu-id="53114-140">Use a separate channel to create the `LoggerClient` in the app.</span></span>
* <span data-ttu-id="53114-141">Usare un pool di canali gRPC, ad esempio, creare un elenco di canali gRPC.</span><span class="sxs-lookup"><span data-stu-id="53114-141">Use a pool of gRPC channels, for example,  create a list of gRPC channels.</span></span> <span data-ttu-id="53114-142">`Random` viene usato per selezionare un canale dall'elenco ogni volta che è necessario un canale gRPC.</span><span class="sxs-lookup"><span data-stu-id="53114-142">`Random` is used to pick a channel from the list each time a gRPC channel is needed.</span></span> <span data-ttu-id="53114-143">L'uso di `Random` distribuisce in modo casuale le chiamate su più connessioni.</span><span class="sxs-lookup"><span data-stu-id="53114-143">Using `Random` randomly distributes calls over multiple connections.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="53114-144">Aumentare il limite massimo di flussi simultanei sul server è un altro modo per risolvere questo problema.</span><span class="sxs-lookup"><span data-stu-id="53114-144">Increasing the maximum concurrent stream limit on the server is another way to solve this problem.</span></span> <span data-ttu-id="53114-145">In gheppio è configurato con <xref:Microsoft.AspNetCore.Server.Kestrel.Core.Http2Limits.MaxStreamsPerConnection> .</span><span class="sxs-lookup"><span data-stu-id="53114-145">In Kestrel this is configured with <xref:Microsoft.AspNetCore.Server.Kestrel.Core.Http2Limits.MaxStreamsPerConnection>.</span></span>
>
> <span data-ttu-id="53114-146">Non è consigliabile aumentare il limite massimo di flussi simultanei.</span><span class="sxs-lookup"><span data-stu-id="53114-146">Increasing the maximum concurrent stream limit is not recommended.</span></span> <span data-ttu-id="53114-147">Un numero eccessivo di flussi in una singola connessione HTTP/2 introduce nuovi problemi di prestazioni:</span><span class="sxs-lookup"><span data-stu-id="53114-147">Too many streams on a single HTTP/2 connection introduces new performance issues:</span></span>
>
> * <span data-ttu-id="53114-148">Contesa di thread tra i flussi che tentano di scrivere nella connessione.</span><span class="sxs-lookup"><span data-stu-id="53114-148">Thread contention between streams trying to write to the connection.</span></span>
> * <span data-ttu-id="53114-149">La perdita di pacchetti di connessione causa il blocco di tutte le chiamate a livello TCP.</span><span class="sxs-lookup"><span data-stu-id="53114-149">Connection packet loss causes all calls to be blocked at the TCP layer.</span></span>

## <a name="load-balancing"></a><span data-ttu-id="53114-150">Bilanciamento del carico</span><span class="sxs-lookup"><span data-stu-id="53114-150">Load balancing</span></span>

<span data-ttu-id="53114-151">Alcuni bilanciamenti del carico non funzionano in modo efficace con gRPC.</span><span class="sxs-lookup"><span data-stu-id="53114-151">Some load balancers don't work effectively with gRPC.</span></span> <span data-ttu-id="53114-152">I bilanciamenti del carico L4 (trasporto) operano a livello di connessione, distribuendo le connessioni TCP tra gli endpoint.</span><span class="sxs-lookup"><span data-stu-id="53114-152">L4 (transport) load balancers operate at a connection level, by distributing TCP connections across endpoints.</span></span> <span data-ttu-id="53114-153">Questo approccio funziona bene per il caricamento delle chiamate API di bilanciamento effettuate con HTTP/1.1.</span><span class="sxs-lookup"><span data-stu-id="53114-153">This approach works well for loading balancing API calls made with HTTP/1.1.</span></span> <span data-ttu-id="53114-154">Le chiamate simultanee effettuate con HTTP/1.1 vengono inviate su connessioni diverse, consentendo il bilanciamento del carico delle chiamate tra gli endpoint.</span><span class="sxs-lookup"><span data-stu-id="53114-154">Concurrent calls made with HTTP/1.1 are sent on different connections, allowing calls to be load balanced across endpoints.</span></span>

<span data-ttu-id="53114-155">Poiché i bilanciamenti del carico L4 operano a livello di connessione, non funzionano bene con gRPC.</span><span class="sxs-lookup"><span data-stu-id="53114-155">Because L4 load balancers operate at a connection level, they don't work well with gRPC.</span></span> <span data-ttu-id="53114-156">gRPC Usa HTTP/2, che esegue il multiplexing di più chiamate su una singola connessione TCP.</span><span class="sxs-lookup"><span data-stu-id="53114-156">gRPC uses HTTP/2, which multiplexes multiple calls on a single TCP connection.</span></span> <span data-ttu-id="53114-157">Tutte le chiamate gRPC sulla connessione passano a un endpoint.</span><span class="sxs-lookup"><span data-stu-id="53114-157">All gRPC calls over that connection go to one endpoint.</span></span>

<span data-ttu-id="53114-158">Per il bilanciamento del carico gRPC sono disponibili due opzioni:</span><span class="sxs-lookup"><span data-stu-id="53114-158">There are two options to effectively load balance gRPC:</span></span>

* <span data-ttu-id="53114-159">Bilanciamento del carico lato client</span><span class="sxs-lookup"><span data-stu-id="53114-159">Client-side load balancing</span></span>
* <span data-ttu-id="53114-160">Bilanciamento del carico del proxy L7 (applicazione)</span><span class="sxs-lookup"><span data-stu-id="53114-160">L7 (application) proxy load balancing</span></span>

> [!NOTE]
> <span data-ttu-id="53114-161">Solo le chiamate gRPC possono essere sottoposte a bilanciamento del carico tra gli endpoint.</span><span class="sxs-lookup"><span data-stu-id="53114-161">Only gRPC calls can be load balanced between endpoints.</span></span> <span data-ttu-id="53114-162">Una volta stabilita una chiamata di streaming gRPC, tutti i messaggi inviati sul flusso passano a un endpoint.</span><span class="sxs-lookup"><span data-stu-id="53114-162">Once a streaming gRPC call is established, all messages sent over the stream go to one endpoint.</span></span>

### <a name="client-side-load-balancing"></a><span data-ttu-id="53114-163">Bilanciamento del carico lato client</span><span class="sxs-lookup"><span data-stu-id="53114-163">Client-side load balancing</span></span>

<span data-ttu-id="53114-164">Con il bilanciamento del carico lato client, il client conosce gli endpoint.</span><span class="sxs-lookup"><span data-stu-id="53114-164">With client-side load balancing, the client knows about endpoints.</span></span> <span data-ttu-id="53114-165">Per ogni chiamata a gRPC, viene selezionato un endpoint diverso a cui inviare la chiamata.</span><span class="sxs-lookup"><span data-stu-id="53114-165">For each gRPC call, it selects a different endpoint to send the call to.</span></span> <span data-ttu-id="53114-166">Il bilanciamento del carico lato client è una scelta ottimale quando la latenza è importante.</span><span class="sxs-lookup"><span data-stu-id="53114-166">Client-side load balancing is a good choice when latency is important.</span></span> <span data-ttu-id="53114-167">Non esiste alcun proxy tra il client e il servizio, quindi la chiamata viene inviata direttamente al servizio.</span><span class="sxs-lookup"><span data-stu-id="53114-167">There's no proxy between the client and the service, so the call is sent to the service directly.</span></span> <span data-ttu-id="53114-168">Lo svantaggio del bilanciamento del carico lato client è che ogni client deve tenere traccia degli endpoint disponibili che deve usare.</span><span class="sxs-lookup"><span data-stu-id="53114-168">The downside to client-side load balancing is that each client must keep track of the available endpoints that it should use.</span></span>

<span data-ttu-id="53114-169">Il bilanciamento del carico del client lookaside è una tecnica in cui lo stato del bilanciamento del carico viene archiviato in una posizione centrale.</span><span class="sxs-lookup"><span data-stu-id="53114-169">Lookaside client load balancing is a technique where load balancing state is stored in a central location.</span></span> <span data-ttu-id="53114-170">I client eseguono periodicamente una query sulla posizione centrale per ottenere informazioni da usare quando si prendono decisioni di bilanciamento del carico.</span><span class="sxs-lookup"><span data-stu-id="53114-170">Clients periodically query the central location for information to use when making load balancing decisions.</span></span>

<span data-ttu-id="53114-171">`Grpc.Net.Client` Attualmente non supporta il bilanciamento del carico lato client.</span><span class="sxs-lookup"><span data-stu-id="53114-171">`Grpc.Net.Client` currently doesn't support client-side load balancing.</span></span> <span data-ttu-id="53114-172">[Grpc. Core](https://www.nuget.org/packages/Grpc.Core) è una scelta ottimale se il bilanciamento del carico lato client è necessario in .NET.</span><span class="sxs-lookup"><span data-stu-id="53114-172">[Grpc.Core](https://www.nuget.org/packages/Grpc.Core) is a good choice if client-side load balancing is required in .NET.</span></span>

### <a name="proxy-load-balancing"></a><span data-ttu-id="53114-173">Bilanciamento del carico del proxy</span><span class="sxs-lookup"><span data-stu-id="53114-173">Proxy load balancing</span></span>

<span data-ttu-id="53114-174">Un proxy L7 (applicazione) funziona a un livello superiore rispetto a un proxy L4 (trasporto).</span><span class="sxs-lookup"><span data-stu-id="53114-174">An L7 (application) proxy works at a higher level than an L4 (transport) proxy.</span></span> <span data-ttu-id="53114-175">I proxy L7 comprendono HTTP/2 e sono in grado di distribuire le chiamate gRPC multiplexing al proxy in una connessione HTTP/2 tra più endpoint.</span><span class="sxs-lookup"><span data-stu-id="53114-175">L7 proxies understand HTTP/2, and are able to distribute gRPC calls multiplexed to the proxy on one HTTP/2 connection across multiple endpoints.</span></span> <span data-ttu-id="53114-176">L'uso di un proxy è più semplice del bilanciamento del carico lato client, ma può aggiungere ulteriore latenza alle chiamate gRPC.</span><span class="sxs-lookup"><span data-stu-id="53114-176">Using a proxy is simpler than client-side load balancing, but can add extra latency to gRPC calls.</span></span>

<span data-ttu-id="53114-177">Sono disponibili molti proxy L7.</span><span class="sxs-lookup"><span data-stu-id="53114-177">There are many L7 proxies available.</span></span> <span data-ttu-id="53114-178">Di seguito sono riportate alcune opzioni:</span><span class="sxs-lookup"><span data-stu-id="53114-178">Some options are:</span></span>

* <span data-ttu-id="53114-179">[Inviato](https://www.envoyproxy.io/) : un proxy open source noto.</span><span class="sxs-lookup"><span data-stu-id="53114-179">[Envoy](https://www.envoyproxy.io/) - A popular open source proxy.</span></span>
* <span data-ttu-id="53114-180">[Linkerd](https://linkerd.io/) -mesh del servizio per Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="53114-180">[Linkerd](https://linkerd.io/) - Service mesh for Kubernetes.</span></span>
* <span data-ttu-id="53114-181">[YARP: proxy inverso](https://microsoft.github.io/reverse-proxy/) -un proxy open source di anteprima scritto in .NET.</span><span class="sxs-lookup"><span data-stu-id="53114-181">[YARP: A Reverse Proxy](https://microsoft.github.io/reverse-proxy/) - A preview open source proxy written in .NET.</span></span>

::: moniker range=">= aspnetcore-5.0"

## <a name="inter-process-communication"></a><span data-ttu-id="53114-182">Comunicazione tra processi</span><span class="sxs-lookup"><span data-stu-id="53114-182">Inter-process communication</span></span>

<span data-ttu-id="53114-183">le chiamate gRPC tra un client e un servizio vengono in genere inviate tramite socket TCP.</span><span class="sxs-lookup"><span data-stu-id="53114-183">gRPC calls between a client and service are usually sent over TCP sockets.</span></span> <span data-ttu-id="53114-184">TCP è ideale per la comunicazione attraverso una rete, ma la [comunicazione interprocesso (IPC)](https://wikipedia.org/wiki/Inter-process_communication) è più efficiente quando il client e il servizio si trovano nello stesso computer.</span><span class="sxs-lookup"><span data-stu-id="53114-184">TCP is great for communicating across a network, but [inter-process communication (IPC)](https://wikipedia.org/wiki/Inter-process_communication) is more efficient when the client and service are on the same machine.</span></span>

<span data-ttu-id="53114-185">Prendere in considerazione l'uso di un trasporto come i socket di dominio UNIX o named pipe per le chiamate gRPC tra processi nello stesso computer.</span><span class="sxs-lookup"><span data-stu-id="53114-185">Consider using a transport like Unix domain sockets or named pipes for gRPC calls between processes on the same machine.</span></span> <span data-ttu-id="53114-186">Per altre informazioni, vedere <xref:grpc/interprocess>.</span><span class="sxs-lookup"><span data-stu-id="53114-186">For more information, see <xref:grpc/interprocess>.</span></span>

## <a name="keep-alive-pings"></a><span data-ttu-id="53114-187">Ping keep-alive</span><span class="sxs-lookup"><span data-stu-id="53114-187">Keep alive pings</span></span>

<span data-ttu-id="53114-188">È possibile usare i ping keep-alive per tenere attive le connessioni HTTP/2 durante i periodi di inattività.</span><span class="sxs-lookup"><span data-stu-id="53114-188">Keep alive pings can be used to keep HTTP/2 connections alive during periods of inactivity.</span></span> <span data-ttu-id="53114-189">Con una connessione HTTP/2 esistente pronta quando un'attività riprende l'attività consente di eseguire rapidamente le chiamate gRPC iniziali, senza un ritardo causato dalla reimpostazione della connessione.</span><span class="sxs-lookup"><span data-stu-id="53114-189">Having an existing HTTP/2 connection ready when an app resumes activity allows for the initial gRPC calls to be made quickly, without a delay caused by the connection being reestablished.</span></span>

<span data-ttu-id="53114-190">I ping Keep-Alive sono configurati in <xref:System.Net.Http.SocketsHttpHandler> :</span><span class="sxs-lookup"><span data-stu-id="53114-190">Keep alive pings are configured on <xref:System.Net.Http.SocketsHttpHandler>:</span></span>

```csharp
var handler = new SocketsHttpHandler
{
    PooledConnectionIdleTimeout = Timeout.InfiniteTimeSpan,
    KeepAlivePingDelay = TimeSpan.FromSeconds(60),
    KeepAlivePingTimeout = TimeSpan.FromSeconds(30),
    EnableMultipleHttp2Connections = true
};

var channel = GrpcChannel.ForAddress("https://localhost:5001", new GrpcChannelOptions
{
    HttpHandler = handler
});
```

<span data-ttu-id="53114-191">Il codice precedente configura un canale che invia un ping keep-alive al server ogni 60 secondi durante i periodi di inattività.</span><span class="sxs-lookup"><span data-stu-id="53114-191">The preceding code configures a channel that sends a keep alive ping to the server every 60 seconds during periods of inactivity.</span></span> <span data-ttu-id="53114-192">Il ping garantisce che il server e gli eventuali proxy in uso non chiudano la connessione a causa di inattività.</span><span class="sxs-lookup"><span data-stu-id="53114-192">The ping ensures the server and any proxies in use won't close the connection because of inactivity.</span></span>

::: moniker-end

## <a name="streaming"></a><span data-ttu-id="53114-193">Streaming</span><span class="sxs-lookup"><span data-stu-id="53114-193">Streaming</span></span>

<span data-ttu-id="53114-194">il flusso bidirezionale gRPC può essere usato per sostituire le chiamate gRPC unarie in scenari a prestazioni elevate.</span><span class="sxs-lookup"><span data-stu-id="53114-194">gRPC bidirectional streaming can be used to replace unary gRPC calls in high-performance scenarios.</span></span> <span data-ttu-id="53114-195">Una volta avviato un flusso bidirezionale, lo streaming dei messaggi è più veloce rispetto all'invio di messaggi con più chiamate gRPC unarie.</span><span class="sxs-lookup"><span data-stu-id="53114-195">Once a bidirectional stream has started, streaming messages back and forth is faster than sending messages with multiple unary gRPC calls.</span></span> <span data-ttu-id="53114-196">I messaggi trasmessi come dati in una richiesta HTTP/2 esistente e eliminano il sovraccarico della creazione di una nuova richiesta HTTP/2 per ogni chiamata unaria.</span><span class="sxs-lookup"><span data-stu-id="53114-196">Streamed messages are sent as data on an existing HTTP/2 request and eliminates the overhead of creating a new HTTP/2 request for each unary call.</span></span>

<span data-ttu-id="53114-197">Servizio di esempio:</span><span class="sxs-lookup"><span data-stu-id="53114-197">Example service:</span></span>

```csharp
public override async Task SayHello(IAsyncStreamReader<HelloRequest> requestStream,
    IServerStreamWriter<HelloReply> responseStream, ServerCallContext context)
{
    await foreach (var request in requestStream.ReadAllAsync())
    {
        var helloReply = new HelloReply { Message = "Hello " + request.Name };

        await responseStream.WriteAsync(helloReply);
    }
}
```

<span data-ttu-id="53114-198">Client di esempio:</span><span class="sxs-lookup"><span data-stu-id="53114-198">Example client:</span></span>

```csharp
var client = new Greet.GreeterClient(channel);
using var call = client.SayHello();

Console.WriteLine("Type a name then press enter.");
while (true)
{
    var text = Console.ReadLine();

    // Send and receive messages over the stream
    await call.RequestStream.WriteAsync(new HelloRequest { Name = text });
    await call.ResponseStream.MoveNext();

    Console.WriteLine($"Greeting: {call.ResponseStream.Current.Message}");
}
```

<span data-ttu-id="53114-199">La sostituzione di chiamate unarie con flusso bidirezionale per motivi di prestazioni è una tecnica avanzata e non è appropriata in molte situazioni.</span><span class="sxs-lookup"><span data-stu-id="53114-199">Replacing unary calls with bidirectional streaming for performance reasons is an advanced technique and is not appropriate in many situations.</span></span>

<span data-ttu-id="53114-200">L'uso delle chiamate di streaming è una scelta ottimale nei casi seguenti:</span><span class="sxs-lookup"><span data-stu-id="53114-200">Using streaming calls is a good choice when:</span></span>

1. <span data-ttu-id="53114-201">È necessaria una velocità effettiva elevata o bassa latenza.</span><span class="sxs-lookup"><span data-stu-id="53114-201">High throughput or low latency is required.</span></span>
2. <span data-ttu-id="53114-202">gRPC e HTTP/2 sono identificati come un collo di bottiglia delle prestazioni.</span><span class="sxs-lookup"><span data-stu-id="53114-202">gRPC and HTTP/2 are identified as a performance bottleneck.</span></span>
3. <span data-ttu-id="53114-203">Un thread di lavoro nel client sta inviando o ricevendo messaggi regolari con un servizio gRPC.</span><span class="sxs-lookup"><span data-stu-id="53114-203">A worker in the client is sending or receiving regular messages with a gRPC service.</span></span>

<span data-ttu-id="53114-204">Tenere presente la complessità e le limitazioni aggiuntive dell'utilizzo di chiamate di streaming anziché unario:</span><span class="sxs-lookup"><span data-stu-id="53114-204">Be aware of the additional complexity and limitations of using streaming calls instead of unary:</span></span>

1. <span data-ttu-id="53114-205">Un flusso può essere interrotto da un errore del servizio o della connessione.</span><span class="sxs-lookup"><span data-stu-id="53114-205">A stream can be interrupted by a service or connection error.</span></span> <span data-ttu-id="53114-206">La logica è necessaria per riavviare il flusso se si verifica un errore.</span><span class="sxs-lookup"><span data-stu-id="53114-206">Logic is required to restart stream if there is an error.</span></span>
2. <span data-ttu-id="53114-207">`RequestStream.WriteAsync` non è sicuro per il multithread.</span><span class="sxs-lookup"><span data-stu-id="53114-207">`RequestStream.WriteAsync` is not safe for multi-threading.</span></span> <span data-ttu-id="53114-208">Solo un messaggio può essere scritto in un flusso alla volta.</span><span class="sxs-lookup"><span data-stu-id="53114-208">Only one message can be written to a stream at a time.</span></span> <span data-ttu-id="53114-209">L'invio di messaggi da più thread su un singolo flusso richiede una coda producer/consumer <xref:System.Threading.Channels.Channel%601> , ad esempio i messaggi Marshall.</span><span class="sxs-lookup"><span data-stu-id="53114-209">Sending messages from multiple threads over a single stream requires a producer/consumer queue like <xref:System.Threading.Channels.Channel%601> to marshall messages.</span></span>
3. <span data-ttu-id="53114-210">Un metodo di streaming gRPC è limitato a ricevere un tipo di messaggio e a inviare un tipo di messaggio.</span><span class="sxs-lookup"><span data-stu-id="53114-210">A gRPC streaming method is limited to receiving one type of message and sending one type of message.</span></span> <span data-ttu-id="53114-211">Ad esempio, `rpc StreamingCall(stream RequestMessage) returns (stream ResponseMessage)` riceve `RequestMessage` e invia `ResponseMessage` .</span><span class="sxs-lookup"><span data-stu-id="53114-211">For example, `rpc StreamingCall(stream RequestMessage) returns (stream ResponseMessage)` receives `RequestMessage` and sends `ResponseMessage`.</span></span> <span data-ttu-id="53114-212">Il supporto di protobuf per i messaggi sconosciuti o condizionali `Any` che usano e `oneof` può ovviare a questa limitazione.</span><span class="sxs-lookup"><span data-stu-id="53114-212">Protobuf's support for unknown or conditional messages using `Any` and `oneof` can work around this limitation.</span></span>
