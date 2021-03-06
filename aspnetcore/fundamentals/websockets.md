---
title: Supporto di WebSocket in ASP.NET Core
author: rick-anderson
description: Introduzione all'uso di oggetti WebSocket in ASP.NET Core.
monikerRange: '>= aspnetcore-1.1'
ms.author: riande
ms.custom: mvc
ms.date: 11/1/2020
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
uid: fundamentals/websockets
ms.openlocfilehash: 2864d98c1e3cff4474ce38d05c47f7cd0e8b3cc5
ms.sourcegitcommit: 1436bd4d70937d6ec3140da56d96caab33c4320b
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/06/2021
ms.locfileid: "102395240"
---
# <a name="websockets-support-in-aspnet-core"></a>Supporto di WebSocket in ASP.NET Core

Di [Tom Dykstra](https://github.com/tdykstra) e [Andrew Stanton-Nurse](https://github.com/anurse)

L'articolo contiene l'introduzione all'uso di oggetti WebSocket in ASP.NET Core. [WebSocket](https://wikipedia.org/wiki/WebSocket) ([RFC 6455](https://tools.ietf.org/html/rfc6455)) è un protocollo che consente canali di comunicazione persistente bidirezionale su connessioni TCP. Viene usato nelle app che sfruttano comunicazioni veloci e in tempo reale, ad esempio le app di chat, i dashboard e le app di gioco.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/websockets/samples) ([procedura per il download](xref:index#how-to-download-a-sample)). [Modalità di esecuzione](#sample-app).

## SignalR

[ASP.NET Core SignalR ](xref:signalr/introduction) è una libreria che semplifica l'aggiunta di funzionalità Web in tempo reale alle app. Laddove possibile, usa oggetti WebSocket.

Per la maggior parte delle applicazioni, è consigliabile usare SignalR WebSocket non elaborati. SignalR fornisce il fallback del trasporto per gli ambienti in cui WebSocket non è disponibile. Fornisce anche un modello di app Remote Procedure Call. E nella maggior parte degli scenari, SignalR non ha un notevole svantaggio in merito alle prestazioni rispetto all'uso di WebSocket non elaborati.

Per alcune app, [gRPC in .NET](xref:grpc/index) offre un'alternativa ai WebSocket.

## <a name="prerequisites"></a>Prerequisiti

* Qualsiasi sistema operativo che supporta ASP.NET Core:  
  * Windows 7/Windows Server 2008 o versioni successive
  * Linux
  * macOS  
* Se l'app viene eseguita su Windows con IIS:
  * Windows 8/Windows Server 2012 o versioni successive
  * IIS 8/IIS 8 Express
  * I WebSocket devono essere abilitati. Vedere la sezione [supporto per IIS/IIS Express](#iisiis-express-support) .  
* Se l'app viene eseguita su [HTTP.sys](xref:fundamentals/servers/httpsys):
  * Windows 8/Windows Server 2012 o versioni successive
* Per i browser supportati, vedere https://caniuse.com/#feat=websockets.

## <a name="configure-the-middleware"></a>Configurare il middleware

Aggiungere il middleware degli oggetti WebSocket nel metodo `Configure` della classe `Startup`:

[!code-csharp[](websockets/samples/2.x/WebSocketsSample/Startup.cs?name=UseWebSockets)]

> [!NOTE]
> Se si desidera accettare le richieste WebSocket in un controller, la chiamata a `app.UseWebSockets` deve essere eseguita prima di `app.UseEndpoints` .

::: moniker range="< aspnetcore-2.2"

È possibile configurare le impostazioni seguenti:

* `KeepAliveInterval`: la frequenza di invio di frame "ping" al client per garantire che i proxy tengano aperta la connessione. Il valore predefinito è due minuti.

::: moniker-end

::: moniker range=">= aspnetcore-2.2"

È possibile configurare le impostazioni seguenti:

* `KeepAliveInterval`: la frequenza di invio di frame "ping" al client per garantire che i proxy tengano aperta la connessione. Il valore predefinito è due minuti.
* `AllowedOrigins`: elenco di valori dell'intestazione Origin consentiti per le richieste WebSocket. Per impostazione predefinita, tutte le origini sono consentite. Per informazioni dettagliate, vedere più avanti "Restrizione per le origini WebSocket".

::: moniker-end

[!code-csharp[](websockets/samples/2.x/WebSocketsSample/Startup.cs?name=UseWebSocketsOptions)]

## <a name="accept-websocket-requests"></a>Accettare le richieste WebSocket

In un momento successivo nel ciclo di vita della richiesta, più avanti nel metodo `Configure` o in un metodo di azione, ad esempio, verificare che si tratti di una richiesta WebSocket e accettarla.

L'esempio seguente è tratto da un punto successivo nel metodo `Configure`:

[!code-csharp[](websockets/samples/2.x/WebSocketsSample/Startup.cs?name=AcceptWebSocket&highlight=7)]

Una richiesta WebSocket può arrivare in qualsiasi URL, ma questo codice di esempio accetta solo le richieste per `/ws`.

Quando si usa un oggetto WebSocket, è **necessario** mantenere la pipeline middleware in esecuzione per la durata della connessione. Se si cerca di inviare o ricevere un messaggio WebSocket dopo che la pipeline middleware è terminata, si potrebbe ottenere un'eccezione simile alla seguente:

```
System.Net.WebSockets.WebSocketException (0x80004005): The remote party closed the WebSocket connection without completing the close handshake. ---> System.ObjectDisposedException: Cannot write to the response body, the response has completed.
Object name: 'HttpResponseStream'.
```

Se si sta usando un servizio in background per scrivere i dati in un oggetto WebSocket, assicurarsi di mantenere la pipeline middleware in esecuzione. A tale scopo, usare <xref:System.Threading.Tasks.TaskCompletionSource%601>. Passare `TaskCompletionSource` al servizio in background e fare in modo che chiami <xref:System.Threading.Tasks.TaskCompletionSource%601.TrySetResult%2A> al termine dell'uso dell'oggetto WebSocket. Quindi assegnare `await` alla proprietà <xref:System.Threading.Tasks.TaskCompletionSource%601.Task> durante la richiesta, come mostrato nell'esempio seguente:

[!code-csharp[](websockets/samples/2.x/WebSocketsSample/Startup2.cs?name=AcceptWebSocket)]

L'eccezione di WebSocket closed può verificarsi anche quando viene restituito troppo presto da un metodo di azione. Quando si accetta un socket in un metodo di azione, attendere il completamento del codice che usa il socket prima di restituire il metodo di azione.

Per attendere il completamento del socket, non utilizzare mai `Task.Wait`, `Task.Result` o chiamate di blocco simili perché ciò può provocare gravi problemi di threading. Usare sempre `await`.

## <a name="send-and-receive-messages"></a>Inviare e ricevere messaggi

Il metodo `AcceptWebSocketAsync` consente di aggiornare la connessione TCP a una connessione WebSocket e offre un oggetto [WebSocket](/dotnet/core/api/system.net.websockets.websocket). Usare l'oggetto `WebSocket` per inviare e ricevere messaggi.

Il codice illustrato in precedenza che accetta la richiesta WebSocket passa l'oggetto `WebSocket` a un metodo `Echo`. Il codice riceve un messaggio e lo reinvia immediatamente. I messaggi vengono inviati e ricevuti in un ciclo, fino a quando il client non chiude la connessione:

[!code-csharp[](websockets/samples/2.x/WebSocketsSample/Startup.cs?name=Echo)]

Quando si accetta la connessione WebSocket prima che inizi il ciclo, la pipeline del middleware termina. Dopo la chiusura del socket, la pipeline si arresta, In altri termini, quando viene accettato il WebSocket, lo spostamento in avanti della richiesta nella pipeline si interrompe. Quando il ciclo viene completato e il socket viene chiuso, la richiesta torna ad avanzare nella pipeline.

::: moniker range=">= aspnetcore-2.2"

## <a name="handle-client-disconnects"></a>Gestire le disconnessioni del client

Il server non viene informato automaticamente quando il client si disconnette a causa della perdita di connettività. Il server riceve un messaggio di disconnessione solo se inviato dal client, ma questo non è possibile in caso di interruzione della connessione Internet. Se si vuole intervenire quando si verifica una situazione di questo tipo, impostare un timeout per segnalare che non sono stati ricevuti messaggi dal client entro un determinato intervallo di tempo.

Se il client non invia messaggi con una certa frequenza e non si vuole impostare un timeout solo perché la connessione diventa inattiva, configurare un timer nel client in modo da inviare un messaggio ping ogni X secondi. Nel server, se un messaggio non è arrivato entro 2\*X secondi dal precedente, terminare la connessione e segnalare che il client si è disconnesso. Attendere il doppio del tempo previsto per tenere conto di eventuali ritardi della rete che potrebbero trattenere il messaggio ping.

## <a name="websocket-origin-restriction"></a>Restrizione per le origini WebSocket

La protezione fornita da CORS non si applica agli oggetti WebSocket. I browser **non**:

* Eseguono richieste CORS preventive.
* Rispettano le restrizioni specificate nelle intestazioni `Access-Control` quando eseguono richieste WebSocket.

I browser, tuttavia, inviano l'intestazione `Origin` quando rilasciano richieste WebSocket. Le applicazioni devono essere configurate per la convalida di queste intestazioni per assicurarsi che siano consentiti solo WebSocket provenienti dalle origini previste.

Se si ospita il server in "https://server.com" e il client in "https://client.com", aggiungere "https://client.com" all'elenco `AllowedOrigins` per consentire la verifica dei WebSocket.

[!code-csharp[](websockets/samples/2.x/WebSocketsSample/Startup.cs?name=UseWebSocketsOptionsAO&highlight=6-7)]

> [!NOTE]
> L'intestazione `Origin` viene controllata dal client e, come l'intestazione `Referer`, può essere falsificata. **Non** usare queste intestazioni come meccanismo di autenticazione.

::: moniker-end

## <a name="iisiis-express-support"></a>Supporto di IIS/IIS Express

Windows Server 2012 o versioni successive e Windows 8 o versioni successive con IIS/IIS Express 8 o versioni successive includono il supporto per il protocollo WebSocket.

> [!NOTE]
> Gli oggetti WebSocket sono sempre abilitati quando si usa IIS Express.

### <a name="enabling-websockets-on-iis"></a>Abilitazione di oggetti WebSocket in IIS

Per abilitare il supporto per il protocollo WebSocket in Windows Server 2012 o versioni successive:

> [!NOTE]
> Questi passaggi non sono necessari quando si usa IIS Express

1. Usare la procedura guidata **Aggiungi ruoli e funzionalità** accessibile tramite il menu **Gestisci** o il collegamento in **Server Manager**.
1. Selezionare **Installazione basata su ruoli o basata su funzionalità**. Selezionare **Avanti**.
1. Selezionare il server appropriato (il server locale è selezionato per impostazione predefinita). Selezionare **Avanti**.
1. Espandere **Server Web (IIS)** nella struttura **Ruoli**, espandere **Server Web**, quindi **Sviluppo applicazioni**.
1. Selezionare **Protocollo WebSocket**. Selezionare **Avanti**.
1. Se non sono necessarie le funzionalità aggiuntive, selezionare **Avanti**.
1. Selezionare **Installa**.
1. Al termine dell'installazione, selezionare **Chiudi** per chiudere la procedura guidata.

Per abilitare il supporto per il protocollo WebSocket in Windows 8 o versioni successive:

> [!NOTE]
> Questi passaggi non sono necessari quando si usa IIS Express

1. Passare a **Pannello di controllo**  >  **programmi** programmi  >  **e funzionalità**  >  **attiva o disattiva funzionalità Windows** (lato sinistro dello schermo).
1. Aprire i nodi seguenti: **Internet Information Services**  >  funzionalità di sviluppo di applicazioni **World Wide Web Services**  >  .
1. Selezionare la funzionalità **Protocollo WebSocket**. Selezionare **OK**.

### <a name="disable-websocket-when-using-socketio-on-nodejs"></a>Disabilitare WebSocket quando si usa socket.io su node.js

Se si usa il supporto WebSocket in [Socket.io](https://socket.io/) in [Node.js](https://nodejs.org/), disabilitare il modulo WebSocket IIS predefinito usando l' `webSocket` elemento in *web.config* o *applicationHost.config*. Se questo passaggio non viene eseguito, il modulo WebSocket di IIS tenta di gestire la comunicazione WebSocket anziché Node.js e l'app.

```xml
<system.webServer>
  <webSocket enabled="false" />
</system.webServer>
```

## <a name="sample-app"></a>App di esempio

L'[app di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/websockets/samples) inclusa in questo articolo è un'app echo. Dispone di una pagina Web che esegue le connessioni WebSocket e il server invia nuovamente tutti i messaggi ricevuti al client. L'app di esempio non è configurata per l'esecuzione da Visual Studio con IIS Express, quindi eseguire l'app in una shell dei comandi con [`dotnet run`](/dotnet/core/tools/dotnet-run) e spostarsi in un browser a `http://localhost:5000` . La pagina Web Mostra lo stato della connessione:

![Stato iniziale della pagina Web prima della connessione a WebSocket](websockets/_static/start.png)

Selezionare **Connect** (Connetti) per inviare una richiesta WebSocket per l'URL indicato. Immettere un messaggio di prova e selezionare **Send** (Invia). Al termine dell'operazione, selezionare **Close Socket** (Chiudi socket). La sezione **Comminication Log** (Registrazione comunicazione) riporta tutte le azioni di apertura, invio e chiusura nel momento in cui vengono eseguite.

![Stato finale della pagina Web dopo l'invio e la ricezione dei messaggi di prova e di connessione di WebSockets](websockets/_static/end.png)
