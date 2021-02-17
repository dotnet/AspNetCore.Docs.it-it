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
# <a name="aspnet-core-signalr-hosting-and-scaling"></a>ASP.NET Core SignalR hosting e scalabilità

Di [Andrew Stanton-Nurse](https://twitter.com/anurse), [Brady Gaster](https://twitter.com/bradygaster)e [Tom Dykstra](https://github.com/tdykstra)

Questo articolo illustra le considerazioni di hosting e scalabilità per le app con traffico elevato che usano ASP.NET Core SignalR .

## <a name="sticky-sessions"></a>Sessioni permanenti

SignalR richiede che tutte le richieste HTTP per una connessione specifica siano gestite dallo stesso processo server. Quando SignalR è in esecuzione in un server farm (più server), è necessario usare "sessioni permanenti". Le "sessioni permanenti" sono definite anche affinità di sessione da parte di alcuni bilanciamenti del carico. Il servizio app Azure usa [Application Request Routing](/iis/extensions/planning-for-arr/application-request-routing-version-2-overview) (arr) per indirizzare le richieste. L'abilitazione dell'impostazione "affinità ARR" nel servizio app Azure consentirà di "sessioni permanenti". Le uniche circostanze in cui non sono richieste sessioni permanenti sono:

1. Quando si esegue l'hosting in un singolo server, in un singolo processo.
1. Quando si usa il SignalR servizio di Azure.
1. Quando tutti i client sono configurati in modo da usare **solo** WebSocket **e** l' [impostazione SkipNegotiation](xref:signalr/configuration#configure-additional-options) è abilitata nella configurazione client.

In tutti gli altri casi (incluso quando viene usato il backplane Redis), è necessario configurare l'ambiente server per le sessioni permanenti.

Per informazioni sulla configurazione del servizio app Azure per SignalR , vedere <xref:signalr/publish-to-azure-web-app> .

## <a name="tcp-connection-resources"></a>Risorse di connessione TCP

Il numero di connessioni TCP simultanee che un server Web è in grado di supportare è limitato. I client HTTP standard *utilizzano connessioni* temporanee. Queste connessioni possono essere chiuse quando il client diventa inattivo e riaperto in un secondo momento. D'altra parte, una SignalR connessione è *persistente*. SignalR le connessioni resteranno aperte anche quando il client diventa inattivo. In un'app a traffico elevato che serve molti client, queste connessioni permanenti possono provocare il raggiungimento del numero massimo di connessioni da parte dei server.

Le connessioni permanenti utilizzano anche una quantità di memoria aggiuntiva per tenere traccia di ogni connessione.

L'utilizzo intensivo delle risorse correlate alla connessione SignalR può influire su altre app Web ospitate nello stesso server. Quando SignalR si apre e contiene le ultime connessioni TCP disponibili, anche altre app Web nello stesso server non dispongono di altre connessioni disponibili.

Se un server esaurisce le connessioni, verranno visualizzati gli errori di socket casuali e gli errori di reimpostazione della connessione. Ad esempio:

```
An attempt was made to access a socket in a way forbidden by its access permissions...
```

Per evitare SignalR che l'utilizzo delle risorse causi errori in altre app Web, eseguire SignalR su server diversi rispetto alle altre app Web.

Per evitare che SignalR l'utilizzo delle risorse causi errori in un' SignalR app, aumentare il numero di connessioni per limitare il numero di connessioni che un server deve gestire.

## <a name="scale-out"></a>Aumentare il numero di istanze

Un'app che usa SignalR deve tenere traccia di tutte le connessioni, che creano problemi per un server farm. Aggiungere un server e ottenere nuove connessioni che gli altri server non conoscono. Ad esempio, in SignalR ogni server del diagramma seguente non è a conoscenza delle connessioni sugli altri server. Quando SignalR in uno dei server si desidera inviare un messaggio a tutti i client, il messaggio passa solo ai client connessi a tale server.

![Ridimensionamento::: NO-LOC (SignalR)::: senza un backplane](scale/_static/scale-no-backplane.png)

Le opzioni per la risoluzione di questo problema sono [il SignalR servizio di Azure](#azure-signalr-service) e il [backplane di redis](#redis-backplane).

## <a name="azure-signalr-service"></a>Servizio di Azure SignalR

Il SignalR servizio di Azure è un proxy anziché un backplane. Ogni volta che un client avvia una connessione al server, il client viene reindirizzato per connettersi al servizio. Questo processo è illustrato nel diagramma seguente:

![Stabilire una connessione al servizio Azure::: NO-LOC (SignalR)::: Service](scale/_static/azure-signalr-service-one-connection.png)

Il risultato è che il servizio gestisce tutte le connessioni client, mentre per ogni server è necessario solo un piccolo numero costante di connessioni al servizio, come illustrato nel diagramma seguente:

![Client connessi al servizio, server connessi al servizio](scale/_static/azure-signalr-service-multiple-connections.png)

Questo approccio per la scalabilità orizzontale presenta diversi vantaggi rispetto all'alternativa del backplane redis:

* Le sessioni permanenti, note anche come [affinità client](/iis/extensions/configuring-application-request-routing-arr/http-load-balancing-using-application-request-routing#step-3---configure-client-affinity), non sono necessarie, perché i client vengono immediatamente reindirizzati al servizio di Azure SignalR quando si connettono.
* Un' SignalR app può aumentare la scalabilità orizzontale in base al numero di messaggi inviati, mentre il servizio di Azure viene SignalR ridimensionato per gestire un numero qualsiasi di connessioni. Ad esempio, potrebbero essere presenti migliaia di client, ma se vengono inviati solo pochi messaggi al secondo, l' SignalR app non deve essere scalata in più server solo per gestire le connessioni stesse.
* Un' SignalR app non userà significativamente più risorse di connessione rispetto a un'app Web senza SignalR .

Per questi motivi, è consigliabile usare il SignalR servizio Azure per tutte le SignalR app ASP.NET Core ospitate in Azure, tra cui il servizio app, le macchine virtuali e i contenitori.

Per altre informazioni, vedere [la SignalR documentazione dei servizi di Azure](/azure/azure-signalr/signalr-overview).

## <a name="redis-backplane"></a>Backplane Redis

[Redis](https://redis.io/) è un archivio chiave-valore in memoria che supporta un sistema di messaggistica con un modello di pubblicazione/sottoscrizione. Il SignalR backplane Redis usa la funzionalità di pubblicazione/sottoscrizione per l'invio di messaggi ad altri server. Quando un client effettua una connessione, le informazioni di connessione vengono passate al backplane. Quando un server vuole inviare un messaggio a tutti i client, lo invia al backplane. Il backplane conosce tutti i client connessi e i server in cui si trovano. Invia il messaggio a tutti i client tramite i rispettivi server. Questo processo è illustrato nel diagramma seguente:

![Backplane Redis, messaggio inviato da un server a tutti i client](scale/_static/redis-backplane.png)

Il backplane di redis è l'approccio di scalabilità orizzontale consigliato per le app ospitate nella propria infrastruttura. Se è presente una latenza di connessione significativa tra il data center e un data center di Azure, il SignalR servizio di Azure potrebbe non essere un'opzione pratica per le app locali con requisiti di bassa latenza o velocità effettiva elevata.

I SignalR vantaggi dei servizi di Azure indicati in precedenza sono gli svantaggi per il backplane di redis:

* Le sessioni permanenti, note anche come [affinità client](/iis/extensions/configuring-application-request-routing-arr/http-load-balancing-using-application-request-routing#step-3---configure-client-affinity), sono necessarie, tranne quando si verificano **entrambe** le condizioni seguenti:
  * Tutti i client sono configurati in modo da usare **solo** WebSocket.
  * L' [impostazione SkipNegotiation](xref:signalr/configuration#configure-additional-options) è abilitata nella configurazione client. 
   Una volta avviata una connessione in un server, la connessione deve rimanere nel server.
* Un' SignalR applicazione deve essere scalata in base al numero di client anche se sono stati inviati pochi messaggi.
* Un' SignalR app usa significativamente più risorse di connessione rispetto a un'app Web senza SignalR .

## <a name="iis-limitations-on-windows-client-os"></a>Limitazioni di IIS sul sistema operativo client Windows

Windows 10 e Windows 8. x sono sistemi operativi client. IIS nei sistemi operativi client ha un limite di 10 connessioni simultanee. SignalRle connessioni sono:

* Temporaneo e ristabilito di frequente.
* **Non** viene eliminato immediatamente quando non viene più utilizzato.

Le condizioni precedenti consentono di raggiungere il limite di 10 connessioni in un sistema operativo client. Quando un sistema operativo client viene usato per lo sviluppo, si consiglia quanto segue:

* Evitare IIS.
* Usare gheppio o IIS Express come destinazioni di distribuzione.

## <a name="linux-with-nginx"></a>Linux con Nginx

Di seguito sono riportate le impostazioni minime obbligatorie per abilitare WebSockets, ServerSentEvents e LongPolling per SignalR :

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

Quando si utilizzano più server back-end, è necessario aggiungere sessioni permanenti per impedire che le SignalR connessioni cambino server durante la connessione. Sono disponibili diversi modi per aggiungere sessioni permanenti in nginx. Di seguito sono riportati due approcci, a seconda di ciò che è disponibile.

Oltre alla configurazione precedente, viene aggiunto quanto segue. Negli esempi seguenti `backend` è il nome del gruppo di server.

Con l' [Open Source nginx](https://nginx.org/en/), usare `ip_hash` per instradare le connessioni a un server in base all'indirizzo IP del client:

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

Con [nginx Plus](https://www.nginx.com/products/nginx), usare `sticky` per aggiungere un cookie a richieste e aggiungere le richieste dell'utente a un server:

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

Infine, modificare `proxy_pass http://localhost:5000` la `server` sezione in `proxy_pass http://backend` .

Per altre informazioni sui WebSocket su nginx, vedere [nginx come proxy WebSocket](https://www.nginx.com/blog/websocket-nginx).

Per altre informazioni sul bilanciamento del carico e le sessioni permanenti, vedere [nginx Load Balancing](https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/).

Per ulteriori informazioni su ASP.NET Core con nginx, vedere l'articolo seguente:
* <xref:host-and-deploy/linux-nginx>

## <a name="third-party-signalr-backplane-providers"></a>Provider backplane di terze parti SignalR

* [NCache](https://www.alachisoft.com/ncache/asp-net-core-signalr.html)
* [Orleans](https://github.com/OrleansContrib/SignalR.Orleans)
* [Rebus](https://github.com/rebus-org/Rebus.SignalR)

## <a name="next-steps"></a>Passaggi successivi

Per altre informazioni, vedere le seguenti risorse:

* [SignalRDocumentazione del servizio Azure](/azure/azure-signalr/signalr-overview)
* [Configurare un backplane Redis](xref:signalr/redis-backplane)
