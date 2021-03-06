---
title: BlazorModelli di hosting ASP.NET Core
author: guardrex
description: Comprendere Blazor WebAssembly e Blazor Server ospitare i modelli.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 12/07/2020
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
uid: blazor/hosting-models
ms.openlocfilehash: 8dd11251358bbeea444661970fadf19cb1390fd3
ms.sourcegitcommit: 1436bd4d70937d6ec3140da56d96caab33c4320b
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/06/2021
ms.locfileid: "102394928"
---
# <a name="aspnet-core-blazor-hosting-models"></a>BlazorModelli di hosting ASP.NET Core

Blazor è un framework web progettato per l'esecuzione sul lato client nel browser in un Runtime .NET basato su [webassembly](https://webassembly.org/)( *Blazor WebAssembly* ) o sul lato server in ASP.NET Core ( *Blazor Server* ). Indipendentemente dal modello di hosting, i modelli di app e componenti *sono gli stessi*.

## Blazor WebAssembly

Il Blazor modello di hosting primario è in esecuzione sul lato client nel browser del webassembly. L' Blazor app, le relative dipendenze e il Runtime .NET vengono scaricati nel browser. L'app viene eseguita direttamente nel thread dell'interfaccia utente del browser. Gli aggiornamenti dell'interfaccia utente e la gestione degli eventi si verificano nello stesso processo. Le risorse dell'app vengono distribuite come file statici in un server Web o in un servizio in grado di servire contenuto statico ai client.

![::: NO-LOC (webassembly Blaze):::: l'oggetto::: NO-LOC (Blazer)::: app viene eseguito su un thread dell'interfaccia utente all'interno del browser.](hosting-models/_static/blazor-webassembly.png)

Quando l' Blazor WebAssembly app viene creata per la distribuzione senza un'app ASP.NET Core back-end per gestire i file, l'app viene chiamata app *autonoma* Blazor WebAssembly . Quando l'app viene creata per la distribuzione con un'app back-end per gestire i file, l'app viene chiamata app *ospitata* Blazor WebAssembly . Un'app ospitata Blazor WebAssembly **`Client`** in genere interagisce con l'app back-end tramite **`Server`** la rete usando chiamate API Web o [SignalR](xref:signalr/introduction) ( <xref:tutorials/signalr-blazor> ).

Lo `blazor.webassembly.js` script viene fornito dal Framework e gestisce:

* Download del runtime .NET, dell'app e delle dipendenze dell'app.
* Inizializzazione del runtime per l'esecuzione dell'app.

Il Blazor WebAssembly modello di hosting offre diversi vantaggi:

* Non esiste alcuna dipendenza lato server .NET. L'app funziona completamente dopo il download nel client.
* Le risorse e le funzionalità client sono completamente sfruttate.
* Il lavoro viene scaricato dal server al client.
* Non è necessario un server Web ASP.NET Core per ospitare l'app. Gli scenari di distribuzione senza server sono possibili, ad esempio per servire l'app da una rete per la distribuzione di contenuti (CDN).

Il Blazor WebAssembly modello di hosting presenta le limitazioni seguenti:

* L'app è limitata alle funzionalità del browser.
* È necessario disporre di hardware e software client idonei (ad esempio, il supporto di webassembly).
* Le dimensioni del download sono maggiori e le app importano più tempo per il caricamento.
* Il supporto di runtime e strumenti .NET è meno maturo. Ad esempio, esistono limitazioni in [.NET standard](/dotnet/standard/net-standard) supporto e debug.

Per creare un' Blazor WebAssembly app, vedere <xref:blazor/tooling> .

Il modello di app ospitata Blazor supporta i [contenitori Docker](/dotnet/standard/microservices-architecture/container-docker-introduction/index). Per il supporto di Docker in Visual Studio, fare clic con il pulsante destro del mouse sul **`Server`** progetto di una soluzione ospitata Blazor WebAssembly e scegliere **Aggiungi**  >  **supporto Docker**.

## Blazor Server

Con il Blazor Server modello di hosting, l'app viene eseguita nel server dall'interno di un'app ASP.NET Core. Gli aggiornamenti dell'interfaccia utente, la gestione degli eventi e le chiamate JavaScript vengono gestiti tramite una [SignalR](xref:signalr/introduction) connessione.

![Il browser interagisce con l'app (ospitata all'interno di un'app ASP.NET Core) nel server tramite::: NO-LOC (SignalR)::: Connection.](hosting-models/_static/blazor-server.png)

L'app ASP.NET Core fa riferimento alla classe dell'app `Startup` per aggiungere:

* Servizi lato server.
* App per la pipeline di gestione delle richieste.

Nel client lo `blazor.server.js` script stabilisce la connessione al SignalR Server. Lo script viene servito all'app sul lato client da una risorsa incorporata nel Framework condiviso ASP.NET Core. L'app sul lato client è responsabile del salvataggio permanente e del ripristino dello stato dell'app in modo obbligatorio. 

Il Blazor Server modello di hosting offre diversi vantaggi:

* Le dimensioni del download sono notevolmente inferiori rispetto a un' Blazor WebAssembly app e l'app viene caricata molto più velocemente.
* L'app sfrutta appieno le funzionalità del server, incluso l'uso di tutte le API compatibili con .NET Core.
* .NET Core nel server viene usato per eseguire l'app, in modo che gli strumenti .NET esistenti, ad esempio il debug, funzionino come previsto.
* Sono supportati i thin client. Ad esempio, Blazor Server le app funzionano con i browser che non supportano webassembly e nei dispositivi con vincoli di risorse.
* La codebase .NET/C# dell'app, incluso il codice componente dell'app, non è servita ai client.

> [!IMPORTANT]
> Blazor ServerViene eseguito il rendering di un'app in risposta alla prima richiesta client, che crea lo stato dell'interfaccia utente nel server. Quando il client tenta di creare una SignalR connessione, **il client deve riconnettersi allo stesso server**. Blazor Server le app che usano più di un server back-end devono implementare *sessioni permanenti* per le SignalR connessioni. Per ulteriori informazioni, vedere la sezione [connessione al server](#connection-to-the-server) .

Il Blazor Server modello di hosting presenta le limitazioni seguenti:

* In genere esiste una latenza più elevata. Ogni interazione dell'utente comporta un hop di rete.
* Non è disponibile alcun supporto offline. Se la connessione client non riesce, l'app smette di funzionare.
* La scalabilità è complessa per le app con molti utenti. Il server deve gestire più connessioni client e gestire lo stato del client.
* Per gestire l'app, è necessario un server ASP.NET Core. Gli scenari di distribuzione senza server non sono possibili, ad esempio per servire l'app da una rete per la distribuzione di contenuti (CDN).

Per creare un' Blazor Server app, vedere <xref:blazor/tooling> .

Il Blazor Server modello di app supporta i [contenitori Docker](/dotnet/standard/microservices-architecture/container-docker-introduction/index). Per il supporto di Docker in Visual Studio, fare clic con il pulsante destro del mouse sul progetto in Visual Studio e scegliere **Aggiungi**  >  **supporto Docker**.

### <a name="comparison-to-server-rendered-ui"></a>Confronto con interfaccia utente sottoposta a rendering server

Un modo per comprendere le Blazor Server app consiste nel comprendere come si differenzia dai modelli tradizionali per il rendering dell'interfaccia utente nelle app ASP.NET Core usando Razor viste o Razor pagine. Entrambi i modelli utilizzano il [ Razor linguaggio](xref:mvc/views/razor) per descrivere il contenuto HTML per il rendering, ma presentano differenze significative *per il rendering del markup.*

Quando Razor viene eseguito il rendering di una pagina o di una vista, ogni riga di Razor codice emette codice HTML in formato testo. Dopo il rendering, il server Elimina l'istanza della pagina o della vista, incluso qualsiasi stato prodotto. Quando si verifica un'altra richiesta della pagina, ad esempio quando la convalida del server non riesce e viene visualizzato il riepilogo di convalida:

* Viene nuovamente eseguito il rendering dell'intera pagina in testo HTML.
* La pagina viene inviata al client.

Un' Blazor app è costituita da elementi riutilizzabili di un'interfaccia utente denominata *Components*. Un componente contiene codice C#, markup e altri componenti. Quando viene eseguito il rendering di un componente, Blazor in viene generato un grafico dei componenti inclusi in modo analogo a un codice HTML o XML Document Object Model (Dom). Questo grafico include lo stato del componente contenuto in proprietà e campi. Blazor valuta il grafico dei componenti per produrre una rappresentazione binaria del markup. Il formato binario può essere:

* Trasformato in testo HTML (durante il prerendering &dagger; ).
* Utilizzato per aggiornare in modo efficiente il markup durante il normale rendering.

&dagger;*Prerendering*: il Razor componente richiesto viene compilato nel server in HTML statico e inviato al client, dove viene sottoposto a rendering all'utente. Una volta stattiva la connessione tra il client e il server, gli elementi statici di cui è stato eseguito il rendering sono sostituiti da elementi interattivi. Il prerendering rende l'app più rispondente all'utente.

Un aggiornamento dell'interfaccia utente in Blazor viene attivato da:

* Interazione dell'utente, ad esempio la selezione di un pulsante.
* Trigger dell'app, ad esempio un timer.

Viene eseguito il rendering del grafico dei componenti e viene calcolata *una differenza dell'interfaccia utente* (differenza). Questa differenza è il set più piccolo di modifiche DOM necessarie per aggiornare l'interfaccia utente nel client. Il diff viene inviato al client in un formato binario e applicato dal browser.

Un componente viene eliminato dopo che l'utente si è spostato dal client. Mentre un utente interagisce con un componente, lo stato del componente (servizi, risorse) deve essere mantenuto nella memoria del server. Poiché lo stato di molti componenti può essere gestito simultaneamente dal server, l'esaurimento della memoria è un problema che deve essere risolto. Per istruzioni su come creare un' Blazor Server app per garantire l'utilizzo ottimale della memoria del server, vedere <xref:blazor/security/server/threat-mitigation> .

### <a name="circuits"></a>Circuiti

Un' Blazor Server app è basata su [ASP.NET Core SignalR ](xref:signalr/introduction). Ogni client comunica al server tramite una o più SignalR connessioni dette *circuito*. Un circuito è Blazor l'astrazione delle SignalR connessioni che può tollerare interruzioni di rete temporanee. Quando un Blazor client rileva che la SignalR connessione è disconnessa, tenta di riconnettersi al server utilizzando una nuova SignalR connessione.

Ogni schermata del browser (scheda del browser o iframe) connessa a un' Blazor Server app usa una SignalR connessione. Si tratta ancora di un'altra distinzione importante rispetto alle app tipiche sottoposte a rendering server. In un'app sottoposta a rendering server, l'apertura della stessa app in più schermate del browser in genere non viene convertita in richieste di risorse aggiuntive sul server. In un' Blazor Server app, ogni schermata del browser richiede un circuito separato e istanze separate dello stato dei componenti che devono essere gestite dal server.

Blazor Considera la chiusura di una scheda del browser o l'esplorazione di un URL esterno a una chiusura *normale* . In caso di chiusura normale, il circuito e le risorse associate vengono immediatamente rilasciati. Un client può anche disconnettersi in modo non corretto, ad esempio a causa di un'interruzione della rete. Blazor Server Archivia i circuiti disconnessi per un intervallo configurabile in modo da consentire la riconnessione del client.

Blazor Server consente al codice di definire un *gestore di circuito*, che consente l'esecuzione di codice in base alle modifiche apportate allo stato del circuito di un utente. Per altre informazioni, vedere <xref:blazor/advanced-scenarios#blazor-server-circuit-handler>.

### <a name="ui-latency"></a>Latenza interfaccia utente

La latenza dell'interfaccia utente è il tempo necessario da un'azione iniziata al momento dell'aggiornamento dell'interfaccia utente. I valori più bassi per la latenza dell'interfaccia utente sono imperativi perché un'app risponda a un utente. In un' Blazor Server app ogni azione viene inviata al server, elaborata e viene restituita una diff dell'interfaccia utente. Di conseguenza, la latenza dell'interfaccia utente è la somma della latenza di rete e la latenza del server nell'elaborazione dell'azione.

Per un'app aziendale limitata a una rete aziendale privata, l'effetto sulle percezioni utente della latenza a causa della latenza di rete è in genere impercettibile. Per un'app distribuita su Internet, la latenza può diventare evidente per gli utenti, in particolare se gli utenti sono ampiamente distribuiti geograficamente.

L'utilizzo della memoria può anche contribuire alla latenza dell'app. Un aumento dell'utilizzo della memoria comporta un frequente Garbage Collection o il paging della memoria su disco, che comportano una riduzione delle prestazioni dell'app e quindi aumentano la latenza dell'interfaccia

Blazor Server le app devono essere ottimizzate per ridurre la latenza dell'interfaccia utente riducendo la latenza di rete e l'utilizzo della memoria Per un approccio alla misurazione della latenza di rete, vedere <xref:blazor/host-and-deploy/server#measure-network-latency> . Per ulteriori informazioni su SignalR e Blazor , vedere:

* <xref:blazor/host-and-deploy/server>
* <xref:blazor/security/server/threat-mitigation>

### <a name="connection-to-the-server"></a>Connessione al server

Blazor Server le app richiedono una SignalR connessione attiva al server. Se la connessione viene persa, l'app tenta di riconnettersi al server. Finché lo stato del client rimane nella memoria del server, la sessione client riprende senza perdere lo stato.

Blazor ServerViene eseguito il rendering di un'app in risposta alla prima richiesta client, che crea lo stato dell'interfaccia utente nel server. Quando il client tenta di creare una SignalR connessione, il client deve riconnettersi allo stesso server. Blazor Server le app che usano più di un server back-end devono implementare *sessioni permanenti* per le SignalR connessioni.

È consigliabile usare il [ SignalR servizio Azure](/azure/azure-signalr) per le Blazor Server app. Il servizio consente di ridimensionare un' Blazor Server app a un numero elevato di connessioni simultanee SignalR . Le sessioni permanenti sono abilitate per il SignalR servizio Azure impostando l' `ServerStickyMode` opzione o il valore di configurazione del servizio su `Required` . Per altre informazioni, vedere <xref:blazor/host-and-deploy/server#signalr-configuration>.

Quando si usa IIS, le sessioni permanenti sono abilitate con il *routing delle richieste di applicazioni*. Per altre informazioni, vedere [bilanciamento del carico http usando il routing delle richieste di applicazioni](/iis/extensions/configuring-application-request-routing-arr/http-load-balancing-using-application-request-routing).

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:blazor/tooling>
* <xref:blazor/project-structure>
* <xref:signalr/introduction>
* <xref:blazor/fundamentals/signalr>
* <xref:tutorials/signalr-blazor>
