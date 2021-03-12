---
title: ASP.NET Core SignalR client JavaScript
author: bradygaster
description: Panoramica di ASP.NET Core SignalR client JavaScript.
monikerRange: '>= aspnetcore-2.1'
ms.author: bradyg
ms.custom: mvc, devx-track-js
ms.date: 04/08/2020
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
uid: signalr/javascript-client
ms.openlocfilehash: 1b5c89c1beaf36912766b3e08b9f16c4d33945f8
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102588061"
---
# <a name="aspnet-core-signalr-javascript-client"></a>ASP.NET Core SignalR client JavaScript

::: moniker range=">= aspnetcore-3.0"

Di [Rachel Appel](https://twitter.com/rachelappel)

La SignalR libreria client JavaScript ASP.NET Core consente agli sviluppatori di chiamare il codice dell'hub sul lato server.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/signalr/javascript-client/samples) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="install-the-signalr-client-package"></a>Installare il SignalR pacchetto client

La SignalR libreria client JavaScript viene distribuita come pacchetto [NPM](https://www.npmjs.com/) . Le sezioni seguenti illustrano diversi modi per installare la libreria client.

### <a name="install-with-npm"></a>Eseguire l'installazione con NPM

Per Visual Studio, eseguire i comandi seguenti dalla **console di gestione pacchetti** nella cartella radice. Per Visual Studio Code, eseguire i comandi seguenti dal **terminale integrato**.

```bash
npm init -y
npm install @microsoft/signalr
```

NPM installa il contenuto del pacchetto nella *cartella \\ @microsoft\signalr\dist\browser node_modules* . Creare una nuova cartella denominata *SignalR* nella cartella *wwwroot \\ lib* . Copiare il file *signalr.js* nella cartella *wwwroot\lib\signalr* .

Fare riferimento al SignalR client JavaScript nell' `<script>` elemento. Ad esempio:

```html
<script src="~/lib/signalr/signalr.js"></script>
```

### <a name="use-a-content-delivery-network-cdn"></a>Usare una rete per la distribuzione di contenuti (CDN)

Per usare la libreria client senza il prerequisito NPM, fare riferimento a una copia ospitata dalla rete CDN della libreria client. Ad esempio:

[!code-html[](javascript-client/samples/3.x/SignalRChat/Pages/Index.cshtml?name=snippet_CDN)]

La libreria client è disponibile nei seguenti CDNs:

* [cdnjs](https://cdnjs.com/libraries/microsoft-signalr)
* [jsDelivr](https://www.jsdelivr.com/package/npm/@microsoft/signalr)
* [unpkg](https://unpkg.com/@microsoft/signalr@next/dist/browser/signalr.min.js)

### <a name="install-with-libman"></a>Installare con LibMan

[LibMan](xref:client-side/libman/index) può essere usato per installare file di libreria client specifici dalla libreria client ospitata nella rete CDN. Ad esempio, aggiungere solo il file JavaScript minimizzati al progetto. Per informazioni dettagliate su questo approccio, vedere [aggiungere la SignalR libreria client](xref:tutorials/signalr#add-the-signalr-client-library).

## <a name="connect-to-a-hub"></a>Connettersi a un hub

Il codice seguente crea e avvia una connessione. Il nome dell'hub non fa distinzione tra maiuscole e minuscole:

[!code-javascript[](javascript-client/samples/3.x/SignalRChat/wwwroot/chat.js?range=3-6,29-43)]

### <a name="cross-origin-connections"></a>Connessioni tra le origini

In genere, i browser caricano le connessioni dallo stesso dominio della pagina richiesta. In alcuni casi, tuttavia, è necessaria una connessione a un altro dominio.

> [!IMPORTANT]
> Il codice client deve usare un URL assoluto anziché un URL relativo. Cambiare `.withUrl("/chathub")` in `.withUrl("https://myappurl/chathub")`.

Per impedire a un sito dannoso di leggere dati sensibili da un altro sito, le [connessioni tra le origini](xref:security/cors) sono disabilitate per impostazione predefinita. Per consentire una richiesta tra origini, abilitarla nella `Startup` classe:

[!code-csharp[](javascript-client/samples/3.x/SignalRChat/Startup.cs?highlight=16-23,40)]

## <a name="call-hub-methods-from-the-client"></a>Chiamare i metodi dell'hub dal client

I client JavaScript chiamano metodi pubblici sugli hub tramite il metodo [Invoke](/javascript/api/%40microsoft/signalr/hubconnection#invoke-string--any---) di [HubConnection](/javascript/api/%40microsoft/signalr/hubconnection). Il `invoke` metodo accetta:

* Nome del metodo dell'hub.
* Qualsiasi argomento definito nel metodo dell'hub.

Nell'esempio seguente il nome del metodo nell'hub è `SendMessage` . Il secondo e il terzo argomento passati per eseguire il `invoke` mapping agli argomenti e del metodo Hub `user` `message` :

[!code-javascript[](javascript-client/samples/3.x/SignalRChat/wwwroot/chat.js?name=snippet_Invoke&highlight=2)]

> [!NOTE]
> La chiamata di metodi dell'hub da un client è supportata solo quando si usa il SignalR servizio di Azure in modalità *predefinita* . Per altre informazioni, vedere [domande frequenti (repository GitHub di Azure-SignalR)](https://github.com/Azure/azure-signalr/blob/dev/docs/faq.md#what-is-the-meaning-of-service-mode-defaultserverlessclassic-how-can-i-choose).

Il `invoke` metodo restituisce una [promessa](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Promise)JavaScript. `Promise`Viene risolto con il valore restituito (se presente) quando il metodo nel server restituisce. Se il metodo sul server genera un errore, `Promise` viene rifiutato con il messaggio di errore. Usare `async` e `await` o i `Promise` `then` metodi e `catch` per gestire questi casi.

I client JavaScript possono anche chiamare metodi pubblici sugli hub tramite il metodo [Send](/javascript/api/%40microsoft/signalr/hubconnection#send-string--any---) di `HubConnection` . A differenza del `invoke` metodo, il `send` metodo non attende una risposta dal server. Il `send` metodo restituisce un oggetto JavaScript `Promise` . Il `Promise` viene risolto quando il messaggio è stato inviato al server. Se si verifica un errore durante l'invio del messaggio, `Promise` viene rifiutato con il messaggio di errore. Usare `async` e `await` o i `Promise` `then` metodi e `catch` per gestire questi casi.

> [!NOTE]
> L'utilizzo di `send` non attende fino a quando il server non ha ricevuto il messaggio. Di conseguenza, non è possibile restituire dati o errori dal server.

## <a name="call-client-methods-from-the-hub"></a>Chiamare i metodi client dall'hub

Per ricevere messaggi dall'hub, definire un metodo usando il metodo [on](/javascript/api/%40microsoft/signalr/hubconnection#on-string---args--any-------void-) di `HubConnection` .

* Nome del metodo client JavaScript.
* Argomenti passati dall'hub al metodo.

Nell'esempio seguente il nome del metodo è `ReceiveMessage` . I nomi degli argomenti sono `user` e `message` :

[!code-javascript[](javascript-client/samples/3.x/SignalRChat/wwwroot/chat.js?name=snippet_ReceiveMessage)]

Il codice precedente in `connection.on` viene eseguito quando il codice lato server lo chiama usando il <xref:Microsoft.AspNetCore.SignalR.ClientProxyExtensions.SendAsync%2A> Metodo:

[!code-csharp[Call client-side](javascript-client/samples/3.x/SignalRChat/Hubs/ChatHub.cs?name=snippet_SendMessage)]

SignalR determina il metodo client da chiamare mediante la corrispondenza tra il nome del metodo e gli argomenti definiti in `SendAsync` e `connection.on` .

> [!NOTE]
> Come procedura consigliata, chiamare il metodo [Start](/javascript/api/%40aspnet/signalr/hubconnection#start) sul valore `HubConnection` after `on` . In questo modo si garantisce che i gestori vengano registrati prima della ricezione dei messaggi.

## <a name="error-handling-and-logging"></a>Registrazione e gestione degli errori

Usare `try` e `catch` con `async` e `await` o il `Promise` `catch` metodo per gestire gli errori sul lato client. Usare `console.error` per restituire gli errori alla console del browser:

[!code-javascript[](javascript-client/samples/3.x/SignalRChat/wwwroot/chat.js?name=snippet_Invoke&highlight=1,3-5)]

Configurare la traccia del log sul lato client passando un logger e un tipo di evento da registrare quando viene stabilita la connessione. I messaggi vengono registrati con il livello di registrazione specificato e un valore superiore. I livelli di log disponibili sono i seguenti:

* `signalR.LogLevel.Error`: Messaggi di errore. Registra `Error` solo i messaggi.
* `signalR.LogLevel.Warning`: Messaggi di avviso relativi a potenziali errori. Log `Warning` e `Error` messaggi.
* `signalR.LogLevel.Information`: Messaggi di stato senza errori. Registra `Information` `Warning` `Error` i messaggi, e.
* `signalR.LogLevel.Trace`: Messaggi di traccia. Registra tutti gli elementi, inclusi i dati trasportati tra l'hub e il client.

Usare il metodo [configureLogging](/javascript/api/%40aspnet/signalr/hubconnectionbuilder#configurelogging) in [HubConnectionBuilder](/javascript/api/%40aspnet/signalr/hubconnectionbuilder) per configurare il livello di registrazione. I messaggi vengono registrati nella console del browser:

[!code-javascript[](javascript-client/samples/3.x/SignalRChat/wwwroot/chat.js?name=snippet_Connection&highlight=3)]

## <a name="reconnect-clients"></a>Riconnettere i client

### <a name="automatically-reconnect"></a>Riconnetti automaticamente

Il client JavaScript per SignalR può essere configurato in modo da riconnettersi automaticamente usando il `withAutomaticReconnect` metodo in [HubConnectionBuilder](/javascript/api/%40aspnet/signalr/hubconnectionbuilder). Per impostazione predefinita, non verrà riconnessa automaticamente.

```javascript
const connection = new signalR.HubConnectionBuilder()
    .withUrl("/chathub")
    .withAutomaticReconnect()
    .build();
```

Senza parametri, `withAutomaticReconnect()` Configura il client in modo che attenda rispettivamente 0, 2, 10 e 30 secondi prima di provare ogni tentativo di riconnessione, arrestandosi dopo quattro tentativi non riusciti.

Prima di iniziare tutti i tentativi di riconnessione, il `HubConnection` passerà allo stato e genererà i `HubConnectionState.Reconnecting` `onreconnecting` callback anziché eseguire la transizione allo `Disconnected` stato e attivare i callback, ad `onclose` esempio `HubConnection` senza riconnessione automatica configurata. Questo consente di avvisare gli utenti che la connessione è stata persa e di disabilitare gli elementi dell'interfaccia utente.

```javascript
connection.onreconnecting(error => {
    console.assert(connection.state === signalR.HubConnectionState.Reconnecting);

    document.getElementById("messageInput").disabled = true;

    const li = document.createElement("li");
    li.textContent = `Connection lost due to error "${error}". Reconnecting.`;
    document.getElementById("messagesList").appendChild(li);
});
```

Se il client si riconnette correttamente entro i primi quattro tentativi, il `HubConnection` eseguirà la transizione allo `Connected` stato e attiverà i `onreconnected` callback. Questo consente di informare gli utenti che è stata ristabilita la connessione.

Poiché la connessione ha un aspetto completamente nuovo per il server, `connectionId` al callback verrà fornito un nuovo `onreconnected` .

> [!WARNING]
> Il `onreconnected` parametro del callback non sarà `connectionId` definito se `HubConnection` è stato configurato per [ignorare la negoziazione](xref:signalr/configuration#configure-client-options).

```javascript
connection.onreconnected(connectionId => {
    console.assert(connection.state === signalR.HubConnectionState.Connected);

    document.getElementById("messageInput").disabled = false;

    const li = document.createElement("li");
    li.textContent = `Connection reestablished. Connected with connectionId "${connectionId}".`;
    document.getElementById("messagesList").appendChild(li);
});
```

`withAutomaticReconnect()` non configurerà `HubConnection` per ritentare gli errori iniziali iniziali, quindi gli errori di avvio devono essere gestiti manualmente:

```javascript
async function start() {
    try {
        await connection.start();
        console.assert(connection.state === signalR.HubConnectionState.Connected);
        console.log("SignalR Connected.");
    } catch (err) {
        console.assert(connection.state === signalR.HubConnectionState.Disconnected);
        console.log(err);
        setTimeout(() => start(), 5000);
    }
};
```

Se il client non si riconnette correttamente entro i primi quattro tentativi, il `HubConnection` passerà allo `Disconnected` stato e attiverà i callback [OnClose](/javascript/api/%40aspnet/signalr/hubconnection#onclose) . Questo consente di informare gli utenti che la connessione è stata persa definitivamente e si consiglia di aggiornare la pagina:

```javascript
connection.onclose(error => {
    console.assert(connection.state === signalR.HubConnectionState.Disconnected);

    document.getElementById("messageInput").disabled = true;

    const li = document.createElement("li");
    li.textContent = `Connection closed due to error "${error}". Try refreshing this page to restart the connection.`;
    document.getElementById("messagesList").appendChild(li);
});
```

Per configurare un numero personalizzato di tentativi di riconnessione prima di disconnettere o modificare la temporizzazione di riconnessione, `withAutomaticReconnect` accetta una matrice di numeri che rappresenta il ritardo in millisecondi di attesa prima di avviare ogni tentativo di riconnessione.

```javascript
const connection = new signalR.HubConnectionBuilder()
    .withUrl("/chathub")
    .withAutomaticReconnect([0, 0, 10000])
    .build();

    // .withAutomaticReconnect([0, 2000, 10000, 30000]) yields the default behavior
```

Nell'esempio precedente `HubConnection` viene configurato per avviare il tentativo di riconnessione immediatamente dopo la perdita della connessione. Questo vale anche per la configurazione predefinita.

Se il primo tentativo di riconnessione ha esito negativo, anche il secondo tentativo di riconnessione verrà avviato immediatamente anziché attendere 2 secondi come nella configurazione predefinita.

Se il secondo tentativo di riconnessione ha esito negativo, il terzo tentativo di riconnessione si avvierà entro 10 secondi, che è simile alla configurazione predefinita.

Il comportamento personalizzato, quindi, differisce di nuovo dal comportamento predefinito arrestando il terzo tentativo di riconnessione anziché tentare un ulteriore tentativo di riconnessione in altri 30 secondi, come nella configurazione predefinita.

Se si desidera un maggiore controllo sull'intervallo e sul numero di tentativi di riconnessione automatici, `withAutomaticReconnect` accetta un oggetto che implementa l' `IRetryPolicy` interfaccia, che dispone di un singolo metodo denominato `nextRetryDelayInMilliseconds` .

`nextRetryDelayInMilliseconds` accetta un solo argomento con il tipo `RetryContext` . `RetryContext`Dispone di tre proprietà: `previousRetryCount` , `elapsedMilliseconds` e `retryReason` che sono rispettivamente un oggetto `number` `number` e `Error` un oggetto. Prima del primo tentativo di riconnessione, sia che `previousRetryCount` `elapsedMilliseconds` saranno zero e sarà `retryReason` l'errore che ha causato la perdita della connessione. Dopo ogni tentativo non riuscito, verrà `previousRetryCount` incrementato di uno, `elapsedMilliseconds` verrà aggiornato in modo da riflettere la quantità di tempo impiegato per la riconnessione fino a questo punto, in millisecondi, e `retryReason` sarà l'errore che ha causato l'esito negativo dell'ultimo tentativo di riconnessione.

`nextRetryDelayInMilliseconds` deve restituire un numero che rappresenta il numero di millisecondi di attesa prima del successivo tentativo di riconnessione o `null` se l'oggetto `HubConnection` deve arrestare la riconnessione.

```javascript
const connection = new signalR.HubConnectionBuilder()
    .withUrl("/chathub")
    .withAutomaticReconnect({
        nextRetryDelayInMilliseconds: retryContext => {
            if (retryContext.elapsedMilliseconds < 60000) {
                // If we've been reconnecting for less than 60 seconds so far,
                // wait between 0 and 10 seconds before the next reconnect attempt.
                return Math.random() * 10000;
            } else {
                // If we've been reconnecting for more than 60 seconds so far, stop reconnecting.
                return null;
            }
        }
    })
    .build();
```

In alternativa, è possibile scrivere codice per riconnettere manualmente il client, come illustrato in [riconnessione manuale](#manually-reconnect).

### <a name="manually-reconnect"></a>Riconnessione manuale

Il codice seguente illustra un tipico approccio di riconnessione manuale:

1. Una funzione (in questo caso, la `start` funzione) viene creata per avviare la connessione.
1. Chiamare la `start` funzione nel `onclose` gestore eventi della connessione.

[!code-javascript[](javascript-client/samples/3.x/SignalRChat/wwwroot/chat.js?range=30-40)]

Un'implementazione reale utilizzerebbe un backup esponenziale o ritenterà un numero di volte specificato prima di rinunciare.

## <a name="additional-resources"></a>Risorse aggiuntive

* [Informazioni di riferimento sulle API JavaScript](/javascript/api/?view=signalr-js-latest&preserve-view=true )
* [Esercitazione su JavaScript](xref:tutorials/signalr)
* [Esercitazione su Webpack e TypeScript](xref:tutorials/signalr-typescript-webpack)
* [Hub](xref:signalr/hubs)
* [Client .NET](xref:signalr/dotnet-client)
* [Pubblicare in Azure](xref:signalr/publish-to-azure-web-app)
* [Richieste tra le origini (CORS)](xref:security/cors)
* [Documentazione senza server per i servizi di Azure SignalR](/azure/azure-signalr/signalr-concept-serverless-development-config)
* [Risolvere gli errori di connessione](xref:signalr/troubleshoot)

::: moniker-end

::: moniker range="< aspnetcore-3.0"

Di [Rachel Appel](https://twitter.com/rachelappel)

La SignalR libreria client JavaScript ASP.NET Core consente agli sviluppatori di chiamare il codice dell'hub sul lato server.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/signalr/javascript-client/samples) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="install-the-signalr-client-package"></a>Installare il SignalR pacchetto client

La SignalR libreria client JavaScript viene distribuita come pacchetto [NPM](https://www.npmjs.com/) . Le sezioni seguenti illustrano diversi modi per installare la libreria client.

### <a name="install-with-npm"></a>Eseguire l'installazione con NPM

Se si usa Visual Studio, eseguire i comandi seguenti dalla **console di gestione pacchetti** nella cartella radice. Per Visual Studio Code, eseguire i comandi seguenti dal **terminale integrato**.

```bash
npm init -y
npm install @aspnet/signalr
```

NPM installa il contenuto del pacchetto nella *cartella \\ @aspnet\signalr\dist\browser node_modules* . Creare una nuova cartella denominata *SignalR* nella cartella *wwwroot \\ lib* . Copiare il file *signalr.js* nella cartella *wwwroot\lib\signalr* .

Fare riferimento al SignalR client JavaScript nell' `<script>` elemento. Ad esempio:

```html
<script src="~/lib/signalr/signalr.js"></script>
```

### <a name="use-a-content-delivery-network-cdn"></a>Usare una rete per la distribuzione di contenuti (CDN)

Per usare la libreria client senza il prerequisito NPM, fare riferimento a una copia ospitata dalla rete CDN della libreria client. Ad esempio:

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/microsoft-signalr/3.1.3/signalr.min.js"></script>
```

La libreria client è disponibile nei seguenti CDNs:

* [cdnjs](https://cdnjs.com/libraries/aspnet-signalr)
* [jsDelivr](https://www.jsdelivr.com/package/npm/@aspnet/signalr)
* [unpkg](https://unpkg.com/@aspnet/signalr@next/dist/browser/signalr.min.js)

### <a name="install-with-libman"></a>Installare con LibMan

[LibMan](xref:client-side/libman/index) può essere usato per installare file di libreria client specifici dalla libreria client ospitata nella rete CDN. Ad esempio, aggiungere solo il file JavaScript minimizzati al progetto. Per informazioni dettagliate su questo approccio, vedere [aggiungere la SignalR libreria client](xref:tutorials/signalr#add-the-signalr-client-library).

## <a name="connect-to-a-hub"></a>Connettersi a un hub

Il codice seguente crea e avvia una connessione. Il nome dell'hub non fa distinzione tra maiuscole e minuscole.

[!code-javascript[Call hub methods](javascript-client/samples/2.x/SignalRChat/wwwroot/js/chat.js?range=9-13,28-51)]

### <a name="cross-origin-connections"></a>Connessioni tra le origini

In genere, i browser caricano le connessioni dallo stesso dominio della pagina richiesta. In alcuni casi, tuttavia, è necessaria una connessione a un altro dominio.

Per impedire a un sito dannoso di leggere dati sensibili da un altro sito, le [connessioni tra le origini](xref:security/cors) sono disabilitate per impostazione predefinita. Per consentire una richiesta tra origini, abilitarla nella `Startup` classe.

[!code-csharp[Cross-origin connections](javascript-client/samples/2.x/SignalRChat/Startup.cs?highlight=29-35,56)]

## <a name="call-hub-methods-from-client"></a>Chiamare i metodi dell'hub dal client

I client JavaScript chiamano metodi pubblici sugli hub tramite il metodo [Invoke](/javascript/api/%40aspnet/signalr/hubconnection#invoke) di [HubConnection](/javascript/api/%40aspnet/signalr/hubconnection). Il `invoke` metodo accetta due argomenti:

* Nome del metodo dell'hub. Nell'esempio seguente il nome del metodo nell'hub è `SendMessage` .
* Qualsiasi argomento definito nel metodo dell'hub. Nell'esempio seguente il nome dell'argomento è `message` . Il codice di esempio usa la sintassi della funzione freccia supportata nelle versioni correnti di tutti i browser principali, ad eccezione di Internet Explorer.

  [!code-javascript[Call hub methods](javascript-client/samples/2.x/SignalRChat/wwwroot/js/chat.js?range=24)]

> [!NOTE]
> La chiamata di metodi dell'hub da un client è supportata solo quando si usa il SignalR servizio di Azure in modalità *predefinita* . Per altre informazioni, vedere [domande frequenti (repository GitHub di Azure-SignalR)](https://github.com/Azure/azure-signalr/blob/dev/docs/faq.md#what-is-the-meaning-of-service-mode-defaultserverlessclassic-how-can-i-choose).

Il `invoke` metodo restituisce una [promessa](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Promise)JavaScript. `Promise`Viene risolto con il valore restituito (se presente) quando il metodo nel server restituisce. Se il metodo sul server genera un errore, `Promise` viene rifiutato con il messaggio di errore. Usare i `then` `catch` metodi e sull'oggetto `Promise` stesso per gestire questi casi (o la `await` sintassi).

Il `send` metodo restituisce un oggetto JavaScript `Promise` . Il `Promise` viene risolto quando il messaggio è stato inviato al server. Se si verifica un errore durante l'invio del messaggio, `Promise` viene rifiutato con il messaggio di errore. Usare i `then` `catch` metodi e sull'oggetto `Promise` stesso per gestire questi casi (o la `await` sintassi).

> [!NOTE]
> L'utilizzo di `send` non attende fino a quando il server non ha ricevuto il messaggio. Di conseguenza, non è possibile restituire dati o errori dal server.

## <a name="call-client-methods-from-hub"></a>Chiamare i metodi client dall'hub

Per ricevere messaggi dall'hub, definire un metodo usando il metodo [on](/javascript/api/%40aspnet/signalr/hubconnection#on) di `HubConnection` .

* Nome del metodo client JavaScript. Nell'esempio seguente il nome del metodo è `ReceiveMessage` .
* Argomenti passati dall'hub al metodo. Nell'esempio seguente il valore dell'argomento è `message` .

[!code-javascript[Receive calls from hub](javascript-client/samples/2.x/SignalRChat/wwwroot/js/chat.js?range=14-19)]

Il codice precedente in `connection.on` viene eseguito quando il codice lato server lo chiama usando il <xref:Microsoft.AspNetCore.SignalR.ClientProxyExtensions.SendAsync%2A> metodo.

[!code-csharp[Call client-side](javascript-client/samples/2.x/SignalRChat/hubs/chathub.cs?range=8-11)]

SignalR determina il metodo client da chiamare mediante la corrispondenza tra il nome del metodo e gli argomenti definiti in `SendAsync` e `connection.on` .

> [!NOTE]
> Come procedura consigliata, chiamare il metodo [Start](/javascript/api/%40aspnet/signalr/hubconnection#start) sul valore `HubConnection` after `on` . In questo modo si garantisce che i gestori vengano registrati prima della ricezione dei messaggi.

## <a name="error-handling-and-logging"></a>Registrazione e gestione degli errori

Concatenare un `catch` metodo alla fine del `start` metodo per gestire gli errori sul lato client. Usare `console.error` per restituire gli errori alla console del browser.

[!code-javascript[Error handling](javascript-client/samples/2.x/SignalRChat/wwwroot/js/chat.js?range=50)]

Configurare la traccia del log sul lato client passando un logger e un tipo di evento da registrare quando viene stabilita la connessione. I messaggi vengono registrati con il livello di registrazione specificato e un valore superiore. I livelli di log disponibili sono i seguenti:

* `signalR.LogLevel.Error`: Messaggi di errore. Registra `Error` solo i messaggi.
* `signalR.LogLevel.Warning`: Messaggi di avviso relativi a potenziali errori. Log `Warning` e `Error` messaggi.
* `signalR.LogLevel.Information`: Messaggi di stato senza errori. Registra `Information` `Warning` `Error` i messaggi, e.
* `signalR.LogLevel.Trace`: Messaggi di traccia. Registra tutti gli elementi, inclusi i dati trasportati tra l'hub e il client.

Usare il metodo [configureLogging](/javascript/api/%40aspnet/signalr/hubconnectionbuilder#configurelogging) in [HubConnectionBuilder](/javascript/api/%40aspnet/signalr/hubconnectionbuilder) per configurare il livello di registrazione. I messaggi vengono registrati nella console del browser.

[!code-javascript[Logging levels](javascript-client/samples/2.x/SignalRChat/wwwroot/js/chat.js?range=9-12)]

## <a name="reconnect-clients"></a>Riconnettere i client

### <a name="manually-reconnect"></a>Riconnessione manuale

> [!WARNING]
> Prima del 3,0, il client JavaScript per SignalR non si riconnette automaticamente. È necessario scrivere codice che riconnetta il client manualmente.

Il codice seguente illustra un tipico approccio di riconnessione manuale:

1. Una funzione (in questo caso, la `start` funzione) viene creata per avviare la connessione.
1. Chiamare la `start` funzione nel `onclose` gestore eventi della connessione.

[!code-javascript[Reconnect the JavaScript client](javascript-client/samples/2.x/SignalRChat/wwwroot/js/chat.js?range=28-40)]

Un'implementazione reale utilizzerebbe un backup esponenziale o ritenterà un numero di volte specificato prima di rinunciare.

## <a name="additional-resources"></a>Risorse aggiuntive

* [Informazioni di riferimento sulle API JavaScript](/javascript/api/?view=signalr-js-latest)
* [Esercitazione su JavaScript](xref:tutorials/signalr)
* [Esercitazione su Webpack e TypeScript](xref:tutorials/signalr-typescript-webpack)
* [Hub](xref:signalr/hubs)
* [Client .NET](xref:signalr/dotnet-client)
* [Pubblicare in Azure](xref:signalr/publish-to-azure-web-app)
* [Richieste tra le origini (CORS)](xref:security/cors)
* [Documentazione senza server per i servizi di Azure SignalR](/azure/azure-signalr/signalr-concept-serverless-development-config)

::: moniker-end
