---
title: SignalRClient Java ASP.NET Core
author: mikaelm12
description: Informazioni su come usare il SignalR client Java di ASP.NET Core.
monikerRange: '>= aspnetcore-2.2'
ms.author: mimengis
ms.custom: mvc
ms.date: 11/12/2019
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
uid: signalr/java-client
ms.openlocfilehash: bdfaf50895612e739eb5ca068a76755f97cf24c2
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102588074"
---
# <a name="aspnet-core-signalr-java-client"></a>SignalRClient Java ASP.NET Core

Di [Mikael Mengistu](https://twitter.com/MikaelM_12)

Il client Java consente la connessione a un SignalR server di ASP.NET Core dal codice Java, incluse le app Android. Come il client [JavaScript](xref:signalr/javascript-client) e il [client .NET](xref:signalr/dotnet-client), il client Java consente di ricevere e inviare messaggi a un hub in tempo reale. Il client Java è disponibile in ASP.NET Core 2,2 e versioni successive.

L'app console Java di esempio a cui si fa riferimento in questo articolo usa il SignalR client Java.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/signalr/java-client/sample) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="install-the-signalr-java-client-package"></a>Installare il SignalR pacchetto client Java

Il file jar *SignalR-1.0.0* consente ai client di connettersi agli SignalR Hub. Per trovare il numero di versione del file JAR più recente, vedere i [Risultati della ricerca di Maven](https://search.maven.org/search?q=g:com.microsoft.signalr%20AND%20a:signalr).

Se si usa Gradle, aggiungere la riga seguente alla `dependencies` sezione del file *Build. Gradle* :

```gradle
implementation 'com.microsoft.signalr:signalr:1.0.0'
```

Se si usa Maven, aggiungere le righe seguenti all'interno dell' `<dependencies>` elemento del file di *pom.xml* :

[!code-xml[pom.xml dependency element](java-client/sample/pom.xml?name=snippet_dependencyElement)]

## <a name="connect-to-a-hub"></a>Connettersi a un hub

Per stabilire un oggetto `HubConnection` , `HubConnectionBuilder` deve essere utilizzato. È possibile configurare l'URL dell'hub e il livello di registrazione durante la compilazione di una connessione. Configurare le opzioni necessarie chiamando uno dei `HubConnectionBuilder` metodi precedenti a `build` . Avviare la connessione con `start` .

[!code-java[Build hub connection](java-client/sample/src/main/java/Chat.java?range=16-17)]

## <a name="call-hub-methods-from-client"></a>Chiamare i metodi dell'hub dal client

Una chiamata a `send` richiama un metodo dell'hub. Passare il nome del metodo dell'hub e gli eventuali argomenti definiti nel metodo Hub a `send` .

[!code-java[send method](java-client/sample/src/main/java/Chat.java?range=28)]

> [!NOTE]
> La chiamata di metodi dell'hub da un client è supportata solo quando si usa il SignalR servizio di Azure in modalità *predefinita* . Per altre informazioni, vedere [domande frequenti (repository GitHub di Azure-SignalR)](https://github.com/Azure/azure-signalr/blob/dev/docs/faq.md#what-is-the-meaning-of-service-mode-defaultserverlessclassic-how-can-i-choose).

## <a name="call-client-methods-from-hub"></a>Chiamare i metodi client dall'hub

Usare `hubConnection.on` per definire i metodi sul client che l'hub può chiamare. Definire i metodi dopo la compilazione, ma prima di avviare la connessione.

[!code-java[Define client methods](java-client/sample/src/main/java/Chat.java?range=19-21)]

## <a name="add-logging"></a>Aggiungere la registrazione

Il SignalR client Java usa la libreria [SLF4J](https://www.slf4j.org/) per la registrazione. Si tratta di un'API di registrazione di alto livello che consente agli utenti della libreria di scegliere una propria implementazione di registrazione specifica inserendo una dipendenza di registrazione specifica. Il frammento di codice seguente illustra come usare `java.util.logging` con il SignalR client Java.

```gradle
implementation 'org.slf4j:slf4j-jdk14:1.7.25'
```

Se non si configura la registrazione nelle dipendenze, SLF4J carica un logger senza operazioni predefinito con il seguente messaggio di avviso:

```
SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
SLF4J: Defaulting to no-operation (NOP) logger implementation
SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for further details.
```

Questa operazione può essere ignorata in modo sicuro.

## <a name="android-development-notes"></a>Note sullo sviluppo per Android

Per quanto riguarda Android SDK compatibilità per le SignalR funzionalità client di, considerare gli elementi seguenti quando si specifica la versione di destinazione Android SDK:

* Il SignalR client Java viene eseguito sul livello 16 dell'API Android e versioni successive.
* La connessione tramite il servizio di Azure richiede il SignalR livello 20 dell'API Android e versioni successive perché il [ SignalR servizio di azure](/azure/azure-signalr/signalr-overview) richiede TLS 1,2 e non supporta i pacchetti di crittografia basati su SHA-1. Android ha [aggiunto il supporto per i pacchetti di crittografia SHA-256 (e versioni successive)](https://developer.android.com/reference/javax/net/ssl/SSLSocket) nel livello API 20.

## <a name="configure-bearer-token-authentication"></a>Configurare l'autenticazione bearer token

Nel SignalR client Java è possibile configurare un Bearer token da usare per l'autenticazione fornendo una "Factory di token di accesso" a [HttpHubConnectionBuilder](/java/api/com.microsoft.signalr.httphubconnectionbuilder?view=aspnet-signalr-java). Usare [withAccessTokenFactory](/java/api/com.microsoft.signalr.httphubconnectionbuilder.withaccesstokenprovider?view=aspnet-signalr-java#com_microsoft_signalr__http_hub_connection_builder_withAccessTokenProvider_Single_String__) per fornire un [](https://github.com/ReactiveX/RxJava) [singolo \<String> ](https://reactivex.io/documentation/single.html)RxJava. Con una chiamata a [Single. rinvia](https://reactivex.io/RxJava/javadoc/io/reactivex/Single.html#defer-java.util.concurrent.Callable-), è possibile scrivere la logica per produrre i token di accesso per il client.

```java
HubConnection hubConnection = HubConnectionBuilder.create("YOUR HUB URL HERE")
    .withAccessTokenProvider(Single.defer(() -> {
        // Your logic here.
        return Single.just("An Access Token");
    })).build();
```

::: moniker range=">= aspnetcore-5.0"

### <a name="passing-class-information-in-java"></a>Passaggio di informazioni sulle classi in Java

Quando si chiamano `on` i `invoke` metodi, o `stream` di `HubConnection` nel client Java, gli utenti devono passare un `Type` oggetto anziché un `Class<?>` oggetto per descrivere qualsiasi oggetto generico `Object` passato al metodo. `Type`È possibile acquisire un oggetto utilizzando la `TypeReference` classe fornita. Usando, ad esempio, una classe generica personalizzata denominata `Foo<T>` , il codice seguente ottiene `Type` :

```java
Type fooType = new TypeReference<Foo<String>>() { }).getType();
```

Per i tipi non generici, ad esempio le primitive o altri tipi senza parametri come `String` , è possibile usare semplicemente l'oggetto incorporato `.class` .

Quando si chiama uno di questi metodi con uno o più tipi di oggetto, usare la sintassi generics quando si richiama il metodo. Ad esempio, quando si registra un `on` gestore per un metodo denominato `func` , che accetta come argomenti una stringa e un `Foo<String>` oggetto, usare il codice seguente per impostare un'azione per la stampa degli argomenti:

```java
hubConnection.<String, Foo<String>>on("func", (param1, param2) ->{
    System.out.println(param1);
    System.out.println(param2);
}, String.class, fooType);
```

Questa convenzione è necessaria perché non è possibile recuperare informazioni complete sui tipi complessi con il `Object.getClass` metodo a causa della cancellazione dei tipi in Java. Ad esempio, la chiamata di `getClass` su un `ArrayList<String>` non restituisce `Class<ArrayList<String>>` , ma piuttosto `Class<ArrayList>` , che non fornisce al deserializzatore informazioni sufficienti per deserializzare correttamente un messaggio in arrivo. Lo stesso vale per gli oggetti personalizzati.

::: moniker-end

## <a name="known-limitations"></a>Limitazioni note

::: moniker range=">= aspnetcore-5.0"

* Il trasporto del fallback del trasporto e degli eventi inviati dal server non è supportato.

::: moniker-end

::: moniker range=">= aspnetcore-3.0 < aspnetcore-5.0"

* Il trasporto del fallback del trasporto e degli eventi inviati dal server non è supportato.
* È supportato solo il protocollo JSON.

::: moniker-end

::: moniker range="< aspnetcore-3.0"

* È supportato solo il protocollo JSON.
* È supportato solo il trasporto WebSocket.
* Il flusso non è ancora supportato.

::: moniker-end

## <a name="additional-resources"></a>Risorse aggiuntive

* [Informazioni di riferimento sulle API Java](/java/api/com.microsoft.signalr?view=aspnet-signalr-java)
* <xref:signalr/hubs>
* <xref:signalr/javascript-client>
* <xref:signalr/publish-to-azure-web-app>
* [Documentazione senza server per i servizi di Azure SignalR](/azure/azure-signalr/signalr-concept-serverless-development-config)
