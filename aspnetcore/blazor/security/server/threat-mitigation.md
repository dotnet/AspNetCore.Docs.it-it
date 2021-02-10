---
title: Linee guida per la mitigazione delle minacce per ASP.NET Core Blazor Server
author: guardrex
description: Informazioni su come ridurre le minacce per la sicurezza alle Blazor Server app.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 11/09/2020
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
uid: blazor/security/server/threat-mitigation
ms.openlocfilehash: 270589b1effc8d82f54d2760b315fa2686699f67
ms.sourcegitcommit: 04ad9cd26fcaa8bd11e261d3661f375f5f343cdc
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/10/2021
ms.locfileid: "100106752"
---
# <a name="threat-mitigation-guidance-for-aspnet-core-blazor-server"></a>Linee guida per la mitigazione delle minacce per ASP.NET Core Blazor Server

Di [Javier Calvarro Nelson](https://github.com/javiercn)

Blazor Server le app adottano un modello di elaborazione dati con *stato* , in cui il server e il client gestiscono una relazione di lunga durata. Lo stato persistente è gestito da un [circuito](xref:blazor/state-management), che può estendersi anche a una durata potenzialmente prolungata.

Quando un utente visita un Blazor Server sito, il server crea un circuito nella memoria del server. Il circuito indica al browser il contenuto di cui eseguire il rendering e risponde agli eventi, ad esempio quando l'utente seleziona un pulsante nell'interfaccia utente. Per eseguire queste azioni, un circuito richiama le funzioni JavaScript nel browser dell'utente e nei metodi .NET sul server. Questa interazione basata su JavaScript bidirezionale è detta interoperabilità [JavaScript (interoperabilità js)](xref:blazor/call-javascript-from-dotnet).

Poiché l'interoperabilità JS viene eseguita su Internet e il client usa un browser remoto, le Blazor Server app condividono la maggior parte dei problemi di sicurezza delle app Web. Questo argomento descrive le minacce più comuni per Blazor Server le app e fornisce indicazioni per la mitigazione delle minacce incentrate sulle app con connessione Internet.

Negli ambienti vincolati, ad esempio all'interno di reti aziendali o Intranet, alcune delle linee guida per la mitigazione:

* Non si applica nell'ambiente vincolato.
* Non vale la pena di implementare perché il rischio per la sicurezza è ridotto in un ambiente vincolato.

## <a name="blazor-and-shared-state"></a>Blazor e stato condiviso

[!INCLUDE[](~/blazor/includes/security/blazor-shared-state.md)]

## <a name="resource-exhaustion"></a>Esaurimento delle risorse

L'esaurimento delle risorse può verificarsi quando un client interagisce con il server e fa in modo che il server consumi risorse eccessive. Un utilizzo eccessivo delle risorse interessa principalmente:

* [CPU](#cpu)
* [Memoria](#memory)
* [Connessioni client](#client-connections)

Gli attacchi Denial of Service (DoS) in genere cercano di esaurire le risorse di un'app o di un server. Tuttavia, l'esaurimento delle risorse non è necessariamente il risultato di un attacco al sistema. Ad esempio, le risorse finite possono essere esaurite a causa della richiesta di un utente elevato. Il DoS viene trattato ulteriormente nella sezione degli [attacchi Denial of Service (DOS)](#denial-of-service-dos-attacks) .

Le risorse esterne al Blazor Framework, ad esempio i database e gli handle di file, usati per leggere e scrivere file, possono anche riscontrare un esaurimento delle risorse. Per altre informazioni, vedere <xref:performance/performance-best-practices>.

### <a name="cpu"></a>CPU

L'esaurimento della CPU può verificarsi quando uno o più client forzano il server a eseguire un lavoro intensivo della CPU.

Si consideri, ad esempio, un' Blazor Server app che calcola un *numero Fibonnacci*. Un numero Fibonnacci viene generato da una sequenza Fibonnacci, dove ogni numero nella sequenza corrisponde alla somma dei due numeri precedenti. La quantità di lavoro necessaria per raggiungere la risposta dipende dalla lunghezza della sequenza e dalle dimensioni del valore iniziale. Se l'app non inserisce limiti per la richiesta di un client, i calcoli con utilizzo intensivo della CPU possono dominare il tempo della CPU e diminuire le prestazioni di altre attività. Un utilizzo eccessivo delle risorse è un problema di sicurezza che influisca sulla disponibilità.

L'esaurimento della CPU è un problema per tutte le app pubbliche. Nelle normali app Web, le richieste e le connessioni si assicurano come misure di sicurezza, ma Blazor Server le app non forniscono le stesse misure di sicurezza. Blazor Server le app devono includere i controlli e i limiti appropriati prima di eseguire potenzialmente un lavoro con utilizzo intensivo della CPU.

### <a name="memory"></a>Memoria

L'esaurimento della memoria può verificarsi quando uno o più client forzano l'utilizzo di una grande quantità di memoria da parte del server.

Si consideri, ad esempio, un' Blazor app lato server con un componente che accetta e visualizza un elenco di elementi. Se l' Blazor app non limita il numero di elementi consentiti o il numero di elementi di cui è stato eseguito il rendering nel client, l'elaborazione e il rendering con utilizzo intensivo della memoria potrebbero dominare la memoria del server fino al punto in cui si verificano le prestazioni del server. Il server potrebbe arrestarsi in modo anomalo o rallentare fino al momento in cui si è verificato un arresto anomalo.

Si consideri lo scenario seguente per la gestione e la visualizzazione di un elenco di elementi relativi a un potenziale scenario di esaurimento della memoria nel server:

* Gli elementi di una `List<MyItem>` proprietà o di un campo utilizzano la memoria del server. Se l'app consente di aumentare il limite dell'elenco di elementi, si rischia che il server esaurisca la memoria. L'esaurimento della memoria causa la fine della sessione corrente (arresto anomalo) e tutte le sessioni simultanee in tale istanza del server ricevono un'eccezione di memoria insufficiente. Per evitare che si verifichi questo scenario, l'app deve usare una struttura di dati che impone un limite di elementi per gli utenti simultanei.
* Se per il rendering non viene utilizzato uno schema di paging, il server utilizza memoria aggiuntiva per gli oggetti che non sono visibili nell'interfaccia utente. Senza un limite al numero di elementi, le richieste di memoria potrebbero esaurire la memoria del server disponibile. Per evitare questo scenario, usare uno degli approcci seguenti:
  * Usare elenchi impaginati durante il rendering.
  * Visualizza solo i primi 100 e 1.000 elementi e richiede all'utente di immettere i criteri di ricerca per trovare gli elementi oltre gli elementi visualizzati.
  * Per uno scenario di rendering più avanzato, implementare elenchi o griglie che supportano la *virtualizzazione*. Con la virtualizzazione, gli elenchi eseguono solo il rendering di un subset di elementi attualmente visibili all'utente. Quando l'utente interagisce con la barra di scorrimento nell'interfaccia utente, il componente esegue il rendering solo degli elementi necessari per la visualizzazione. Gli elementi che non sono attualmente necessari per la visualizzazione possono essere conservati nell'archiviazione secondaria, che è l'approccio ideale. Gli elementi non visualizzati possono anche essere mantenuti in memoria, il che è meno ideale.

Blazor Server le app offrono un modello di programmazione simile ad altri Framework dell'interfaccia utente per le app con stato, ad esempio WPF, Windows Form o Blazor WebAssembly . La differenza principale consiste nel fatto che in diversi framework dell'interfaccia utente la memoria utilizzata dall'app appartiene al client e ha effetto solo su tale client. Ad esempio, un' Blazor WebAssembly app viene eseguita interamente nel client e usa solo le risorse di memoria del client. Nello Blazor Server scenario, la memoria usata dall'app appartiene al server e viene condivisa tra i client nell'istanza del server.

Le richieste di memoria sul lato server sono una considerazione per tutte le Blazor Server app. Tuttavia, la maggior parte delle app Web sono senza stato e la memoria usata durante l'elaborazione di una richiesta viene rilasciata quando viene restituita la risposta. Come raccomandazione generale, non consentire ai client di allocare una quantità di memoria non associata come in qualsiasi altra app sul lato server che rende permanente le connessioni client. La memoria utilizzata da un' Blazor Server applicazione viene mantenute per un periodo di tempo più lungo rispetto a una singola richiesta.

> [!NOTE]
> Durante lo sviluppo, è possibile utilizzare un profiler o una traccia acquisita per valutare le richieste di memoria dei client. Un profiler o una traccia non acquisisce la memoria allocata a un client specifico. Per acquisire l'utilizzo della memoria di un client specifico durante lo sviluppo, acquisire un dump ed esaminare la richiesta di memoria di tutti gli oggetti radice nel circuito di un utente.

### <a name="client-connections"></a>Connessioni client

L'esaurimento della connessione può verificarsi quando uno o più client aprono troppe connessioni simultanee al server, impedendo ad altri client di stabilire nuove connessioni.

Blazor i client stabiliscono una singola connessione per sessione e mantengono aperta la connessione fino a quando la finestra del browser è aperta. Le richieste sul server di gestione di tutte le connessioni non sono specifiche delle Blazor app. Data la natura persistente delle connessioni e la natura con stato delle Blazor Server app, l'esaurimento della connessione costituisce un rischio maggiore per la disponibilità dell'app.

Per impostazione predefinita, non esiste alcun limite al numero di connessioni per utente per un' Blazor Server app. Se l'app richiede un limite di connessione, adottare uno o più degli approcci seguenti:

* Richiedere l'autenticazione, che limita naturalmente la capacità degli utenti non autorizzati di connettersi all'app. Affinché questo scenario sia efficace, è necessario impedire agli utenti di effettuare il provisioning di nuovi utenti in base a.
* Limitare il numero di connessioni per utente. La limitazione delle connessioni può essere eseguita tramite gli approcci seguenti. Prestare attenzione a consentire agli utenti autorizzati di accedere all'app, ad esempio quando viene stabilito un limite di connessione in base all'indirizzo IP del client.
  * A livello di app:
    * Estensibilità del routing dell'endpoint.
    * Richiedere l'autenticazione per la connessione all'app e per tenere traccia delle sessioni attive per ogni utente.
    * Rifiutare le nuove sessioni al raggiungimento di un limite.
    * Connessioni WebSocket proxy a un'app tramite l'uso di un proxy, ad esempio il [ SignalR servizio di Azure](/azure/azure-signalr/signalr-overview) che multiplexerà le connessioni dai client a un'app. In questo modo si fornisce un'app con maggiore capacità di connessione rispetto a un singolo client che può stabilire, impedendo a un client di esaurire le connessioni al server.
  * A livello di server: usare un proxy/gateway davanti all'app. Ad esempio, la [porta anteriore di Azure](/azure/frontdoor/front-door-overview) consente di definire, gestire e monitorare il routing globale del traffico Web a un'app e funziona quando le Blazor Server app sono configurate per l'uso del polling prolungato.
  
    > [!NOTE]
    > Sebbene il polling prolungato sia supportato per Blazor Server le app, [WebSocket è il protocollo di trasporto consigliato](xref:blazor/host-and-deploy/server#azure-signalr-service). Il front-end di [Azure](/azure/frontdoor/front-door-overview) non supporta WebSocket in questo momento, ma il supporto per WebSocket è in considerazione per una versione futura del servizio.

## <a name="denial-of-service-dos-attacks"></a>Attacchi Denial of Service (DoS)

Gli attacchi di tipo Denial of Service (DoS) coinvolgono un client che induce il server a esaurire una o più risorse rendendo l'app non disponibile. Blazor Server le app includono i limiti predefiniti e si basano su altri ASP.NET Core e SignalR limiti per la protezione da attacchi DOS impostati su <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions> :

* <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions.DisconnectedCircuitMaxRetained?displayProperty=nameWithType>
* <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions.DisconnectedCircuitRetentionPeriod?displayProperty=nameWithType>
* <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions.JSInteropDefaultCallTimeout?displayProperty=nameWithType>
* <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions.MaxBufferedUnacknowledgedRenderBatches?displayProperty=nameWithType>
* <xref:Microsoft.AspNetCore.SignalR.HubConnectionContextOptions.MaximumReceiveMessageSize?displayProperty=nameWithType>

Per ulteriori informazioni ed esempi di codice di configurazione, vedere gli articoli seguenti:

* <xref:blazor/fundamentals/signalr>
* <xref:signalr/configuration>

## <a name="interactions-with-the-browser-client"></a>Interazioni con il browser (client)

Un client interagisce con il server tramite l'invio di eventi di interoperabilità JS e il completamento del rendering. La comunicazione di interoperabilità JS passa da JavaScript a .NET e viceversa:

* Gli eventi del browser vengono inviati dal client al server in modo asincrono.
* Il server risponde in modo asincrono al rendering dell'interfaccia utente, se necessario.

### <a name="javascript-functions-invoked-from-net"></a>Funzioni JavaScript richiamate da .NET

Per le chiamate da metodi .NET a JavaScript:

* Tutte le chiamate hanno un timeout configurabile dopo il quale hanno esito negativo, restituendo un oggetto <xref:System.OperationCanceledException> al chiamante.
  * È previsto un timeout predefinito per le chiamate ( <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions.JSInteropDefaultCallTimeout?displayProperty=nameWithType> ) di un minuto. Per configurare questo limite, vedere <xref:blazor/call-javascript-from-dotnet#harden-js-interop-calls> .
  * È possibile fornire un token di annullamento per controllare l'annullamento in base a ogni chiamata. Si basano sul timeout di chiamata predefinito, dove possibile e con associazione temporale qualsiasi chiamata al client se viene fornito un token di annullamento.
* Il risultato di una chiamata JavaScript non può essere considerato attendibile. Il Blazor client dell'app in esecuzione nel browser cerca la funzione JavaScript da richiamare. La funzione viene richiamata e viene generato il risultato o un errore. Un client dannoso può tentare di eseguire le operazioni seguenti:
  * Causa un problema nell'app restituendo un errore dalla funzione JavaScript.
  * Indurre un comportamento imprevisto sul server restituendo un risultato imprevisto dalla funzione JavaScript.

Adottare le seguenti precauzioni per evitare gli scenari precedenti:

* Eseguire il wrapping delle chiamate di interoperabilità JS all'interno [`try-catch`](/dotnet/csharp/language-reference/keywords/try-catch) delle istruzioni per tenere conto degli errori che potrebbero verificarsi durante la chiamata. Per altre informazioni, vedere <xref:blazor/fundamentals/handle-errors#javascript-interop>.
* Convalidare i dati restituiti dalle chiamate di interoperabilità JS, inclusi i messaggi di errore, prima di intraprendere qualsiasi azione.

### <a name="net-methods-invoked-from-the-browser"></a>Metodi .NET richiamati dal browser

Non considerare attendibili le chiamate da JavaScript ai metodi .NET. Quando un metodo .NET viene esposto a JavaScript, prendere in considerazione il modo in cui viene richiamato il metodo .NET:

* Considerare qualsiasi metodo .NET esposto a JavaScript come un endpoint pubblico per l'app.
  * Convalida input.
    * Verificare che i valori siano compresi negli intervalli previsti.
    * Verificare che l'utente disponga delle autorizzazioni necessarie per eseguire l'azione richiesta.
  * Non allocare una quantità eccessiva di risorse come parte della chiamata al metodo .NET. Eseguire ad esempio i controlli e i limiti di utilizzo della CPU e della memoria.
  * Tenere presente che i metodi statici e di istanza possono essere esposti ai client JavaScript. Evitare di condividere lo stato tra le sessioni a meno che la progettazione non chiami per lo stato di condivisione con vincoli appropriati.
    * Per i metodi di istanza esposti tramite `DotNetReference` oggetti creati in origine tramite l'inserimento di dipendenze, gli oggetti devono essere registrati come oggetti con ambito. Si applica a qualsiasi servizio DI DI cui l' Blazor Server app usa.
    * Per i metodi statici, evitare di stabilire uno stato che non può essere definito come ambito del client a meno che l'app non condivida in modo esplicito lo stato in base alla progettazione per tutti gli utenti in un'istanza del server.
  * Evitare di passare i dati forniti dall'utente nei parametri alle chiamate JavaScript. Se il passaggio dei dati nei parametri è assolutamente necessario, assicurarsi che il codice JavaScript gestisca il passaggio dei dati senza introdurre vulnerabilità di [Scripting (XSS) tra siti](#cross-site-scripting-xss) . Ad esempio, non scrivere dati specificati dall'utente nel Document Object Model (DOM) impostando la `innerHTML` proprietà di un elemento. Provare a usare i [criteri di sicurezza del contenuto (CSP)](https://developer.mozilla.org/docs/Web/HTTP/CSP) per disabilitare `eval` e altre primitive JavaScript non sicure.
* Evitare di implementare l'invio personalizzato delle chiamate .NET all'implementazione dell'invio del Framework. L'esposizione di metodi .NET al browser è uno scenario avanzato, non consigliato per Blazor lo sviluppo generale.

### <a name="events"></a>Eventi

Gli eventi forniscono un punto di ingresso a un' Blazor Server app. Le stesse regole per la salvaguardia degli endpoint nelle app Web si applicano alla gestione degli eventi nelle Blazor Server app. Un client dannoso può inviare tutti i dati che desidera inviare come payload per un evento.

Ad esempio:

* Un evento di modifica per un `<select>` può inviare un valore che non rientra nelle opzioni presentate dall'app al client.
* Un oggetto `<input>` può inviare dati di testo al server, ignorando la convalida lato client.

L'app deve convalidare i dati per qualsiasi evento gestito dall'app. I Blazor [componenti dei moduli](xref:blazor/forms-validation) del Framework eseguono convalide di base. Se l'app usa componenti dei moduli personalizzati, è necessario scrivere codice personalizzato per convalidare i dati degli eventi in base alle esigenze.

Blazor Server gli eventi sono asincroni, quindi è possibile inviare più eventi al server prima che l'app abbia tempo per rispondere producendo un nuovo rendering. Questo ha alcune implicazioni sulla sicurezza da prendere in considerazione. La limitazione delle azioni client nell'app deve essere eseguita all'interno di gestori eventi e non dipende dallo stato di visualizzazione di cui è stato eseguito il rendering corrente.

Si consideri un componente contatore che deve consentire a un utente di incrementare un contatore un massimo di tre volte. Il pulsante per incrementare il contatore è in modo condizionale in base al valore di `count` :

```razor
<p>Count: @count<p>

@if (count < 3)
{
    <button @onclick="IncrementCount" value="Increment count" />
}

@code 
{
    private int count = 0;

    private void IncrementCount()
    {
        count++;
    }
}
```

Un client può inviare uno o più eventi di incremento prima che il Framework produca un nuovo rendering di questo componente. Il risultato è che `count` può essere incrementato di *tre volte* dall'utente perché il pulsante non viene rimosso dall'interfaccia utente abbastanza rapidamente. Il modo corretto per raggiungere il limite di tre `count` incrementi è illustrato nell'esempio seguente:

```razor
<p>Count: @count<p>

@if (count < 3)
{
    <button @onclick="IncrementCount" value="Increment count" />
}

@code 
{
    private int count = 0;

    private void IncrementCount()
    {
        if (count < 3)
        {
            count++;
        }
    }
}
```

Aggiungendo il `if (count < 3) { ... }` controllo all'interno del gestore, la decisione di incremento `count` si basa sullo stato corrente dell'app. La decisione non è basata sullo stato dell'interfaccia utente come nell'esempio precedente, che potrebbe essere temporaneamente obsoleto.

### <a name="guard-against-multiple-dispatches"></a>Protezione da più invii

Se un callback di evento richiama un'operazione a esecuzione prolungata in modo asincrono, ad esempio il recupero di dati da un servizio o da un database esterno, provare a usare un GUARD. Il Guard può impedire all'utente di accodare più operazioni mentre è in corso l'operazione con commenti visivi. Il codice componente seguente imposta `isLoading` su `true` mentre `GetForecastAsync` ottiene i dati dal server. Mentre `isLoading` è `true` , il pulsante è disabilitato nell'interfaccia utente:

```razor
@page "/fetchdata"
@using BlazorServerSample.Data
@inject WeatherForecastService ForecastService

<button disabled="@isLoading" @onclick="UpdateForecasts">Update</button>

@code {
    private bool isLoading;
    private WeatherForecast[] forecasts;

    private async Task UpdateForecasts()
    {
        if (!isLoading)
        {
            isLoading = true;
            forecasts = await ForecastService.GetForecastAsync(DateTime.Now);
            isLoading = false;
        }
    }
}
```

Il modello Guard illustrato nell'esempio precedente funziona se l'operazione in background viene eseguita in modo asincrono con il `async` - `await` modello.

### <a name="cancel-early-and-avoid-use-after-dispose"></a>Annulla anticipatamente ed evita l'uso di After-Dispose

Oltre a usare un GUARD come descritto nella sezione [Guard da più invii](#guard-against-multiple-dispatches) , provare a usare un <xref:System.Threading.CancellationToken> per annullare le operazioni a esecuzione prolungata quando il componente viene eliminato. Questo approccio offre il vantaggio aggiuntivo di evitare *use-after-Dispose* nei componenti:

```razor
@implements IDisposable

...

@code {
    private readonly CancellationTokenSource TokenSource = 
        new CancellationTokenSource();

    private async Task UpdateForecasts()
    {
        ...

        forecasts = await ForecastService.GetForecastAsync(DateTime.Now, 
            TokenSource.Token);

        if (TokenSource.Token.IsCancellationRequested)
        {
           return;
        }

        ...
    }

    public void Dispose()
    {
        TokenSource.Cancel();
    }
}
```

### <a name="avoid-events-that-produce-large-amounts-of-data"></a>Evitare eventi che producono grandi quantità di dati

Alcuni eventi DOM, ad esempio `oninput` o `onscroll` , possono produrre una grande quantità di dati. Evitare di usare questi eventi nelle Blazor app Server.

## <a name="additional-security-guidance"></a>Ulteriori indicazioni sulla sicurezza

Le linee guida per la protezione delle app ASP.NET Core si applicano alle Blazor Server app e vengono descritte nelle sezioni seguenti:

* [Registrazione e dati sensibili](#logging-and-sensitive-data)
* [Proteggere le informazioni in transito con HTTPS](#protect-information-in-transit-with-https)
* [Scripting tra siti (XSS)](#cross-site-scripting-xss)
* [Protezione tra le origini](#cross-origin-protection)
* [Clic su jacking](#click-jacking)
* [Reindirizzamenti aperti](#open-redirects)

### <a name="logging-and-sensitive-data"></a>Registrazione e dati sensibili

Le interazioni di interoperabilità JS tra il client e il server vengono registrate nei log del server con le <xref:Microsoft.Extensions.Logging.ILogger> istanze. Blazor evita la registrazione di informazioni riservate, ad esempio gli eventi effettivi o gli input e gli output di interoperabilità JS.

Quando si verifica un errore nel server, il framework invia una notifica al client e rimuove la sessione. Per impostazione predefinita, il client riceve un messaggio di errore generico che può essere visualizzato negli strumenti di sviluppo del browser.

L'errore sul lato client non include il stack e non fornisce dettagli sulla ragione dell'errore, ma i log del server contengono tali informazioni. A scopo di sviluppo, le informazioni sensibili sugli errori possono essere rese disponibili al client [abilitando errori dettagliati](xref:blazor/fundamentals/handle-errors#blazor-server-detailed-circuit-errors).

> [!WARNING]
> L'esposizione delle informazioni sugli errori ai client su Internet costituisce un rischio per la sicurezza che deve essere sempre evitata.

### <a name="protect-information-in-transit-with-https"></a>Proteggere le informazioni in transito con HTTPS

Blazor Server USA SignalR per la comunicazione tra il client e il server. Blazor Server in genere usa il trasporto che SignalR negozia, che in genere è WebSocket.

Blazor Server non garantisce l'integrità e la riservatezza dei dati inviati tra il server e il client. Usare sempre HTTPS.

### <a name="cross-site-scripting-xss"></a>Scripting tra siti (XSS)

Il cross-site scripting (XSS) consente a un'entità non autorizzata di eseguire logica arbitraria nel contesto del browser. Un'app compromessa potrebbe potenzialmente eseguire codice arbitrario nel client. La vulnerabilità può essere utilizzata per eseguire potenzialmente una serie di azioni dannose sul server:

* Inviare eventi Fake/non validi al server.
* Completamenti di rendering non riusciti/non validi.
* Evitare di inviare i completamenti di rendering.
* Invia le chiamate di interoperabilità da JavaScript a .NET.
* Modificare la risposta delle chiamate di interoperabilità da .NET a JavaScript.
* Evitare di inviare i risultati di interoperabilità da .NET a JS.

Il Blazor Server Framework esegue misure per la protezione da alcune delle minacce precedenti:

* Interrompe la produzione di nuovi aggiornamenti dell'interfaccia utente se il client non riconosce i batch di rendering. Configurato con <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions.MaxBufferedUnacknowledgedRenderBatches?displayProperty=nameWithType> .
* Timeout di qualsiasi chiamata da .NET a JavaScript dopo un minuto senza ricevere una risposta dal client. Configurato con <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions.JSInteropDefaultCallTimeout?displayProperty=nameWithType> .
* Esegue la convalida di base per tutti gli input provenienti dal browser durante l'interoperabilità JS:
  * I riferimenti .NET sono validi e del tipo previsto dal metodo .NET.
  * I dati non sono in formato non corretto.
  * Il numero corretto di argomenti per il metodo è presente nel payload.
  * Gli argomenti o i risultati possono essere deserializzati correttamente prima di richiamare il metodo.
* Esegue la convalida di base in tutti gli input provenienti dal browser dagli eventi inviati:
  * Il tipo dell'evento è valido.
  * I dati per l'evento possono essere deserializzati.
  * All'evento è associato un gestore eventi.

Oltre alle misure di sicurezza implementate dal Framework, l'app deve essere codificata dallo sviluppatore per proteggersi dalle minacce e intraprendere le azioni appropriate:

* Convalidare sempre i dati durante la gestione degli eventi.
* Eseguire le azioni appropriate alla ricezione di dati non validi:
  * Ignorare i dati e restituire. Ciò consente all'app di continuare l'elaborazione delle richieste.
  * Se l'app determina che l'input è illegittimo e non può essere prodotto da client legittimo, generare un'eccezione. La generazione di un'eccezione rimuove il circuito e termina la sessione.
* Non considerare attendibile il messaggio di errore fornito dai completamenti dei batch di rendering inclusi nei log. L'errore viene fornito dal client e in genere non può essere considerato attendibile, perché il client potrebbe essere compromesso.
* Non considerare attendibile l'input sulle chiamate di interoperabilità JS in entrambe le direzioni tra i metodi JavaScript e .NET.
* L'app è responsabile della convalida della validità del contenuto di argomenti e risultati, anche se gli argomenti o i risultati vengono deserializzati correttamente.

Per poter esistere una vulnerabilità XSS, l'app deve incorporare l'input dell'utente nella pagina di cui è stato eseguito il rendering. Blazor Server i componenti eseguono un passaggio della fase di compilazione in cui il markup in un `.razor` file viene trasformato in logica C# procedurale. In fase di esecuzione, la logica C# compila un *albero di rendering* che descrive gli elementi, il testo e i componenti figlio. Viene applicato al DOM del browser tramite una sequenza di istruzioni JavaScript (o viene serializzato in HTML in caso di prerendering):

* L'input dell'utente di cui è stato eseguito il rendering tramite Razor la sintassi normale (ad esempio, `@someStringValue` ) non espone una vulnerabilità XSS perché la Razor sintassi viene aggiunta al Dom tramite comandi che possono scrivere solo testo. Anche se il valore include il markup HTML, il valore viene visualizzato come testo statico. Quando si esegue il prerendering, l'output è codificato in formato HTML, che visualizza anche il contenuto come testo statico.
* I tag di script non sono consentiti e non devono essere inclusi nell'albero di rendering del componente dell'app. Se un tag di script è incluso nel markup di un componente, viene generato un errore in fase di compilazione.
* Gli autori dei componenti possono creare componenti in C# senza usare Razor . L'autore del componente è responsabile dell'uso delle API corrette durante la creazione dell'output. Usare, ad esempio, `builder.AddContent(0, someUserSuppliedString)` e *non* `builder.AddMarkupContent(0, someUserSuppliedString)` , perché quest'ultimo potrebbe creare una vulnerabilità XSS.

Come parte della protezione da attacchi XSS, valutare la possibilità di implementare mitigazioni XSS, ad esempio i [criteri di sicurezza del contenuto (CSP)](https://developer.mozilla.org/docs/Web/HTTP/CSP).

Per altre informazioni, vedere <xref:security/cross-site-scripting>.

### <a name="cross-origin-protection"></a>Protezione tra le origini

Gli attacchi tra le origini coinvolgono un client di un'origine diversa che esegue un'azione sul server. L'azione dannosa è in genere una richiesta GET o un modulo POST (richiesta tra siti falsa, CSRF), ma è anche possibile aprire un WebSocket dannoso. Blazor Server le app offrono [le stesse garanzie che qualsiasi altra SignalR app che usa l'offerta del protocollo Hub](xref:signalr/security):

* Blazor Server è possibile accedere alle app tra le origini, a meno che non vengano adottate misure aggiuntive per impedirlo. Per disabilitare l'accesso tra le origini, disabilitare CORS nell'endpoint aggiungendo il middleware CORS alla pipeline e aggiungendo ai <xref:Microsoft.AspNetCore.Cors.DisableCorsAttribute> Blazor metadati dell'endpoint o limitare il set di origini consentite [configurando SignalR per la condivisione di risorse tra](xref:signalr/security#cross-origin-resource-sharing)le origini.
* Se CORS è abilitato, potrebbero essere necessari passaggi aggiuntivi per proteggere l'app a seconda della configurazione di CORS. Se la CORS è abilitata a livello globale, è possibile disabilitare CORS per l' Blazor Server Hub aggiungendo i <xref:Microsoft.AspNetCore.Cors.DisableCorsAttribute> metadati ai metadati dell'endpoint dopo aver chiamato il <xref:Microsoft.AspNetCore.Builder.ComponentEndpointRouteBuilderExtensions.MapBlazorHub%2A> Generatore di route dell'endpoint.

Per altre informazioni, vedere <xref:security/anti-request-forgery>.

### <a name="click-jacking"></a>Clic su jacking

Il click-Jack prevede il rendering di un sito come `<iframe>` all'interno di un sito da un'origine diversa per consentire all'utente di eseguire azioni sul sito sotto attacco.

Per proteggere un'app dal rendering all'interno di un `<iframe>` , usare i [criteri di sicurezza del contenuto (CSP)](https://developer.mozilla.org/docs/Web/HTTP/CSP) e l' `X-Frame-Options` intestazione. Per ulteriori informazioni, vedere la pagina relativa alla [documentazione Web MDN: X-frame-options](https://developer.mozilla.org/docs/Web/HTTP/Headers/X-Frame-Options).

### <a name="open-redirects"></a>Reindirizzamenti aperti

Quando Blazor Server viene avviata una sessione dell'app, il server esegue la convalida di base degli URL inviati come parte dell'avvio della sessione. Il Framework verifica che l'URL di base sia un elemento padre dell'URL corrente prima di stabilire il circuito. Il Framework non esegue alcun controllo aggiuntivo.

Quando un utente seleziona un collegamento sul client, l'URL del collegamento viene inviato al server, che determina l'azione da eseguire. Ad esempio, l'app può eseguire una navigazione sul lato client o indicare al browser di passare alla nuova posizione.

I componenti possono inoltre attivare richieste di navigazione a livello tramite l'utilizzo di <xref:Microsoft.AspNetCore.Components.NavigationManager> . In questi scenari, l'app può eseguire una navigazione sul lato client o indicare al browser di passare alla nuova posizione.

I componenti devono:

* Evitare di usare l'input dell'utente come parte degli argomenti della chiamata di navigazione.
* Convalidare gli argomenti per assicurarsi che la destinazione sia consentita dall'app.

In caso contrario, un utente malintenzionato può forzare il browser ad accedere a un sito controllato da un utente malintenzionato. In questo scenario l'utente malintenzionato fa in modo che l'app usi un input utente come parte della chiamata del <xref:Microsoft.AspNetCore.Components.NavigationManager.NavigateTo%2A?displayProperty=nameWithType> metodo.

Questo Consiglio si applica anche quando si esegue il rendering dei collegamenti come parte dell'app:

* Se possibile, usare collegamenti relativi.
* Verificare che le destinazioni di collegamento assolute siano valide prima di includerle in una pagina.

Per altre informazioni, vedere <xref:security/preventing-open-redirects>.

## <a name="security-checklist"></a>Elenco di controllo relativo alla sicurezza

Il seguente elenco di considerazioni sulla sicurezza non è completo:

* Convalidare gli argomenti dagli eventi.
* Convalidare gli input e i risultati dalle chiamate di interoperabilità JS.
* Evitare di usare (o convalidare in anticipo) l'input utente per le chiamate di interoperabilità da .NET a JS.
* Impedire al client di allocare una quantità di memoria non associata.
  * Dati all'interno del componente.
  * `DotNetObject` riferimenti restituiti al client.
* Proteggersi da più invii.
* Annulla le operazioni a esecuzione prolungata quando il componente viene eliminato.
* Evitare eventi che producono grandi quantità di dati.
* Evitare di usare l'input dell'utente come parte delle chiamate a <xref:Microsoft.AspNetCore.Components.NavigationManager.NavigateTo%2A?displayProperty=nameWithType> e convalidare l'input dell'utente per gli URL rispetto a un set di origini consentite prima se non è possibile evitare
* Non prendere decisioni di autorizzazione in base allo stato dell'interfaccia utente, ma solo dallo stato del componente.
* Prendere in considerazione l'uso dei [criteri di sicurezza del contenuto (CSP)](https://developer.mozilla.org/docs/Web/HTTP/CSP) per la protezione da attacchi XSS.
* Si consiglia di usare CSP e [X-frame-options](https://developer.mozilla.org/docs/Web/HTTP/Headers/X-Frame-Options) per la protezione da clic-jacking.
* Verificare che le impostazioni CORS siano appropriate quando si Abilita CORS o disabilitare in modo esplicito CORS per le Blazor app.
* Eseguire un test per assicurarsi che i limiti lato server per l' Blazor app forniscano un'esperienza utente accettabile senza livelli di rischio inaccettabili.
