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
# <a name="aspnet-core-signalr-java-client"></a><span data-ttu-id="337d9-103">SignalRClient Java ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="337d9-103">ASP.NET Core SignalR Java client</span></span>

<span data-ttu-id="337d9-104">Di [Mikael Mengistu](https://twitter.com/MikaelM_12)</span><span class="sxs-lookup"><span data-stu-id="337d9-104">By [Mikael Mengistu](https://twitter.com/MikaelM_12)</span></span>

<span data-ttu-id="337d9-105">Il client Java consente la connessione a un SignalR server di ASP.NET Core dal codice Java, incluse le app Android.</span><span class="sxs-lookup"><span data-stu-id="337d9-105">The Java client enables connecting to an ASP.NET Core SignalR server from Java code, including Android apps.</span></span> <span data-ttu-id="337d9-106">Come il client [JavaScript](xref:signalr/javascript-client) e il [client .NET](xref:signalr/dotnet-client), il client Java consente di ricevere e inviare messaggi a un hub in tempo reale.</span><span class="sxs-lookup"><span data-stu-id="337d9-106">Like the [JavaScript client](xref:signalr/javascript-client) and the [.NET client](xref:signalr/dotnet-client), the Java client enables you to receive and send messages to a hub in real time.</span></span> <span data-ttu-id="337d9-107">Il client Java è disponibile in ASP.NET Core 2,2 e versioni successive.</span><span class="sxs-lookup"><span data-stu-id="337d9-107">The Java client is available in ASP.NET Core 2.2 and later.</span></span>

<span data-ttu-id="337d9-108">L'app console Java di esempio a cui si fa riferimento in questo articolo usa il SignalR client Java.</span><span class="sxs-lookup"><span data-stu-id="337d9-108">The sample Java console app referenced in this article uses the SignalR Java client.</span></span>

<span data-ttu-id="337d9-109">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/signalr/java-client/sample) ([procedura per il download](xref:index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="337d9-109">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/signalr/java-client/sample) ([how to download](xref:index#how-to-download-a-sample))</span></span>

## <a name="install-the-signalr-java-client-package"></a><span data-ttu-id="337d9-110">Installare il SignalR pacchetto client Java</span><span class="sxs-lookup"><span data-stu-id="337d9-110">Install the SignalR Java client package</span></span>

<span data-ttu-id="337d9-111">Il file jar *SignalR-1.0.0* consente ai client di connettersi agli SignalR Hub.</span><span class="sxs-lookup"><span data-stu-id="337d9-111">The *signalr-1.0.0* JAR file allows clients to connect to SignalR hubs.</span></span> <span data-ttu-id="337d9-112">Per trovare il numero di versione del file JAR più recente, vedere i [Risultati della ricerca di Maven](https://search.maven.org/search?q=g:com.microsoft.signalr%20AND%20a:signalr).</span><span class="sxs-lookup"><span data-stu-id="337d9-112">To find the latest JAR file version number, see the [Maven search results](https://search.maven.org/search?q=g:com.microsoft.signalr%20AND%20a:signalr).</span></span>

<span data-ttu-id="337d9-113">Se si usa Gradle, aggiungere la riga seguente alla `dependencies` sezione del file *Build. Gradle* :</span><span class="sxs-lookup"><span data-stu-id="337d9-113">If using Gradle, add the following line to the `dependencies` section of your *build.gradle* file:</span></span>

```gradle
implementation 'com.microsoft.signalr:signalr:1.0.0'
```

<span data-ttu-id="337d9-114">Se si usa Maven, aggiungere le righe seguenti all'interno dell' `<dependencies>` elemento del file di *pom.xml* :</span><span class="sxs-lookup"><span data-stu-id="337d9-114">If using Maven, add the following lines inside the `<dependencies>` element of your *pom.xml* file:</span></span>

[!code-xml[pom.xml dependency element](java-client/sample/pom.xml?name=snippet_dependencyElement)]

## <a name="connect-to-a-hub"></a><span data-ttu-id="337d9-115">Connettersi a un hub</span><span class="sxs-lookup"><span data-stu-id="337d9-115">Connect to a hub</span></span>

<span data-ttu-id="337d9-116">Per stabilire un oggetto `HubConnection` , `HubConnectionBuilder` deve essere utilizzato.</span><span class="sxs-lookup"><span data-stu-id="337d9-116">To establish a `HubConnection`, the `HubConnectionBuilder` should be used.</span></span> <span data-ttu-id="337d9-117">È possibile configurare l'URL dell'hub e il livello di registrazione durante la compilazione di una connessione.</span><span class="sxs-lookup"><span data-stu-id="337d9-117">The hub URL and log level can be configured while building a connection.</span></span> <span data-ttu-id="337d9-118">Configurare le opzioni necessarie chiamando uno dei `HubConnectionBuilder` metodi precedenti a `build` .</span><span class="sxs-lookup"><span data-stu-id="337d9-118">Configure any required options by calling any of the `HubConnectionBuilder` methods before `build`.</span></span> <span data-ttu-id="337d9-119">Avviare la connessione con `start` .</span><span class="sxs-lookup"><span data-stu-id="337d9-119">Start the connection with `start`.</span></span>

[!code-java[Build hub connection](java-client/sample/src/main/java/Chat.java?range=16-17)]

## <a name="call-hub-methods-from-client"></a><span data-ttu-id="337d9-120">Chiamare i metodi dell'hub dal client</span><span class="sxs-lookup"><span data-stu-id="337d9-120">Call hub methods from client</span></span>

<span data-ttu-id="337d9-121">Una chiamata a `send` richiama un metodo dell'hub.</span><span class="sxs-lookup"><span data-stu-id="337d9-121">A call to `send` invokes a hub method.</span></span> <span data-ttu-id="337d9-122">Passare il nome del metodo dell'hub e gli eventuali argomenti definiti nel metodo Hub a `send` .</span><span class="sxs-lookup"><span data-stu-id="337d9-122">Pass the hub method name and any arguments defined in the hub method to `send`.</span></span>

[!code-java[send method](java-client/sample/src/main/java/Chat.java?range=28)]

> [!NOTE]
> <span data-ttu-id="337d9-123">La chiamata di metodi dell'hub da un client è supportata solo quando si usa il SignalR servizio di Azure in modalità *predefinita* .</span><span class="sxs-lookup"><span data-stu-id="337d9-123">Calling hub methods from a client is only supported when using the Azure SignalR Service in *Default* mode.</span></span> <span data-ttu-id="337d9-124">Per altre informazioni, vedere [domande frequenti (repository GitHub di Azure-SignalR)](https://github.com/Azure/azure-signalr/blob/dev/docs/faq.md#what-is-the-meaning-of-service-mode-defaultserverlessclassic-how-can-i-choose).</span><span class="sxs-lookup"><span data-stu-id="337d9-124">For more information, see [Frequently Asked Questions (azure-signalr GitHub repository)](https://github.com/Azure/azure-signalr/blob/dev/docs/faq.md#what-is-the-meaning-of-service-mode-defaultserverlessclassic-how-can-i-choose).</span></span>

## <a name="call-client-methods-from-hub"></a><span data-ttu-id="337d9-125">Chiamare i metodi client dall'hub</span><span class="sxs-lookup"><span data-stu-id="337d9-125">Call client methods from hub</span></span>

<span data-ttu-id="337d9-126">Usare `hubConnection.on` per definire i metodi sul client che l'hub può chiamare.</span><span class="sxs-lookup"><span data-stu-id="337d9-126">Use `hubConnection.on` to define methods on the client that the hub can call.</span></span> <span data-ttu-id="337d9-127">Definire i metodi dopo la compilazione, ma prima di avviare la connessione.</span><span class="sxs-lookup"><span data-stu-id="337d9-127">Define the methods after building but before starting the connection.</span></span>

[!code-java[Define client methods](java-client/sample/src/main/java/Chat.java?range=19-21)]

## <a name="add-logging"></a><span data-ttu-id="337d9-128">Aggiungere la registrazione</span><span class="sxs-lookup"><span data-stu-id="337d9-128">Add logging</span></span>

<span data-ttu-id="337d9-129">Il SignalR client Java usa la libreria [SLF4J](https://www.slf4j.org/) per la registrazione.</span><span class="sxs-lookup"><span data-stu-id="337d9-129">The SignalR Java client uses the [SLF4J](https://www.slf4j.org/) library for logging.</span></span> <span data-ttu-id="337d9-130">Si tratta di un'API di registrazione di alto livello che consente agli utenti della libreria di scegliere una propria implementazione di registrazione specifica inserendo una dipendenza di registrazione specifica.</span><span class="sxs-lookup"><span data-stu-id="337d9-130">It's a high-level logging API that allows users of the library to chose their own specific logging implementation by bringing in a specific logging dependency.</span></span> <span data-ttu-id="337d9-131">Il frammento di codice seguente illustra come usare `java.util.logging` con il SignalR client Java.</span><span class="sxs-lookup"><span data-stu-id="337d9-131">The following code snippet shows how to use `java.util.logging` with the SignalR Java client.</span></span>

```gradle
implementation 'org.slf4j:slf4j-jdk14:1.7.25'
```

<span data-ttu-id="337d9-132">Se non si configura la registrazione nelle dipendenze, SLF4J carica un logger senza operazioni predefinito con il seguente messaggio di avviso:</span><span class="sxs-lookup"><span data-stu-id="337d9-132">If you don't configure logging in your dependencies, SLF4J loads a default no-operation logger with the following warning message:</span></span>

```
SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
SLF4J: Defaulting to no-operation (NOP) logger implementation
SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for further details.
```

<span data-ttu-id="337d9-133">Questa operazione può essere ignorata in modo sicuro.</span><span class="sxs-lookup"><span data-stu-id="337d9-133">This can safely be ignored.</span></span>

## <a name="android-development-notes"></a><span data-ttu-id="337d9-134">Note sullo sviluppo per Android</span><span class="sxs-lookup"><span data-stu-id="337d9-134">Android development notes</span></span>

<span data-ttu-id="337d9-135">Per quanto riguarda Android SDK compatibilità per le SignalR funzionalità client di, considerare gli elementi seguenti quando si specifica la versione di destinazione Android SDK:</span><span class="sxs-lookup"><span data-stu-id="337d9-135">With regards to Android SDK compatibility for the SignalR client features, consider the following items when specifying your target Android SDK version:</span></span>

* <span data-ttu-id="337d9-136">Il SignalR client Java viene eseguito sul livello 16 dell'API Android e versioni successive.</span><span class="sxs-lookup"><span data-stu-id="337d9-136">The SignalR Java Client will run on Android API Level 16 and later.</span></span>
* <span data-ttu-id="337d9-137">La connessione tramite il servizio di Azure richiede il SignalR livello 20 dell'API Android e versioni successive perché il [ SignalR servizio di azure](/azure/azure-signalr/signalr-overview) richiede TLS 1,2 e non supporta i pacchetti di crittografia basati su SHA-1.</span><span class="sxs-lookup"><span data-stu-id="337d9-137">Connecting through the Azure SignalR Service will require Android API Level 20 and later because the [Azure SignalR Service](/azure/azure-signalr/signalr-overview) requires TLS 1.2 and doesn't support SHA-1-based cipher suites.</span></span> <span data-ttu-id="337d9-138">Android ha [aggiunto il supporto per i pacchetti di crittografia SHA-256 (e versioni successive)](https://developer.android.com/reference/javax/net/ssl/SSLSocket) nel livello API 20.</span><span class="sxs-lookup"><span data-stu-id="337d9-138">Android [added support for SHA-256 (and above) cipher suites](https://developer.android.com/reference/javax/net/ssl/SSLSocket) in API Level 20.</span></span>

## <a name="configure-bearer-token-authentication"></a><span data-ttu-id="337d9-139">Configurare l'autenticazione bearer token</span><span class="sxs-lookup"><span data-stu-id="337d9-139">Configure bearer token authentication</span></span>

<span data-ttu-id="337d9-140">Nel SignalR client Java è possibile configurare un Bearer token da usare per l'autenticazione fornendo una "Factory di token di accesso" a [HttpHubConnectionBuilder](/java/api/com.microsoft.signalr.httphubconnectionbuilder?view=aspnet-signalr-java).</span><span class="sxs-lookup"><span data-stu-id="337d9-140">In the SignalR Java client, you can configure a bearer token to use for authentication by providing an "access token factory" to the [HttpHubConnectionBuilder](/java/api/com.microsoft.signalr.httphubconnectionbuilder?view=aspnet-signalr-java).</span></span> <span data-ttu-id="337d9-141">Usare [withAccessTokenFactory](/java/api/com.microsoft.signalr.httphubconnectionbuilder.withaccesstokenprovider?view=aspnet-signalr-java#com_microsoft_signalr__http_hub_connection_builder_withAccessTokenProvider_Single_String__) per fornire un [](https://github.com/ReactiveX/RxJava) [singolo \<String> ](https://reactivex.io/documentation/single.html)RxJava.</span><span class="sxs-lookup"><span data-stu-id="337d9-141">Use [withAccessTokenFactory](/java/api/com.microsoft.signalr.httphubconnectionbuilder.withaccesstokenprovider?view=aspnet-signalr-java#com_microsoft_signalr__http_hub_connection_builder_withAccessTokenProvider_Single_String__) to provide an [RxJava](https://github.com/ReactiveX/RxJava) [Single\<String>](https://reactivex.io/documentation/single.html).</span></span> <span data-ttu-id="337d9-142">Con una chiamata a [Single. rinvia](https://reactivex.io/RxJava/javadoc/io/reactivex/Single.html#defer-java.util.concurrent.Callable-), è possibile scrivere la logica per produrre i token di accesso per il client.</span><span class="sxs-lookup"><span data-stu-id="337d9-142">With a call to [Single.defer](https://reactivex.io/RxJava/javadoc/io/reactivex/Single.html#defer-java.util.concurrent.Callable-), you can write logic to produce access tokens for your client.</span></span>

```java
HubConnection hubConnection = HubConnectionBuilder.create("YOUR HUB URL HERE")
    .withAccessTokenProvider(Single.defer(() -> {
        // Your logic here.
        return Single.just("An Access Token");
    })).build();
```

::: moniker range=">= aspnetcore-5.0"

### <a name="passing-class-information-in-java"></a><span data-ttu-id="337d9-143">Passaggio di informazioni sulle classi in Java</span><span class="sxs-lookup"><span data-stu-id="337d9-143">Passing Class information in Java</span></span>

<span data-ttu-id="337d9-144">Quando si chiamano `on` i `invoke` metodi, o `stream` di `HubConnection` nel client Java, gli utenti devono passare un `Type` oggetto anziché un `Class<?>` oggetto per descrivere qualsiasi oggetto generico `Object` passato al metodo.</span><span class="sxs-lookup"><span data-stu-id="337d9-144">When calling the `on`, `invoke`, or `stream` methods of `HubConnection` in the Java client, users should pass a `Type` object rather than a `Class<?>` object to describe any generic `Object` passed to the method.</span></span> <span data-ttu-id="337d9-145">`Type`È possibile acquisire un oggetto utilizzando la `TypeReference` classe fornita.</span><span class="sxs-lookup"><span data-stu-id="337d9-145">A `Type` can be acquired using the provided `TypeReference` class.</span></span> <span data-ttu-id="337d9-146">Usando, ad esempio, una classe generica personalizzata denominata `Foo<T>` , il codice seguente ottiene `Type` :</span><span class="sxs-lookup"><span data-stu-id="337d9-146">For example, using a custom generic class named `Foo<T>`, the following code gets the `Type`:</span></span>

```java
Type fooType = new TypeReference<Foo<String>>() { }).getType();
```

<span data-ttu-id="337d9-147">Per i tipi non generici, ad esempio le primitive o altri tipi senza parametri come `String` , è possibile usare semplicemente l'oggetto incorporato `.class` .</span><span class="sxs-lookup"><span data-stu-id="337d9-147">For non-generics, such as primitives or other non-parameterized types like `String`, you can simply use the built-in `.class`.</span></span>

<span data-ttu-id="337d9-148">Quando si chiama uno di questi metodi con uno o più tipi di oggetto, usare la sintassi generics quando si richiama il metodo.</span><span class="sxs-lookup"><span data-stu-id="337d9-148">When calling one of these methods with one or more object types, use the generics syntax when invoking the method.</span></span> <span data-ttu-id="337d9-149">Ad esempio, quando si registra un `on` gestore per un metodo denominato `func` , che accetta come argomenti una stringa e un `Foo<String>` oggetto, usare il codice seguente per impostare un'azione per la stampa degli argomenti:</span><span class="sxs-lookup"><span data-stu-id="337d9-149">For example, when registering an `on` handler for a method named `func`, which takes as arguments a String and a `Foo<String>` object, use the following code to set an action to print the arguments:</span></span>

```java
hubConnection.<String, Foo<String>>on("func", (param1, param2) ->{
    System.out.println(param1);
    System.out.println(param2);
}, String.class, fooType);
```

<span data-ttu-id="337d9-150">Questa convenzione è necessaria perché non è possibile recuperare informazioni complete sui tipi complessi con il `Object.getClass` metodo a causa della cancellazione dei tipi in Java.</span><span class="sxs-lookup"><span data-stu-id="337d9-150">This convention is necessary because we can not retrieve complete information about complex types with the `Object.getClass` method due to type erasure in Java.</span></span> <span data-ttu-id="337d9-151">Ad esempio, la chiamata di `getClass` su un `ArrayList<String>` non restituisce `Class<ArrayList<String>>` , ma piuttosto `Class<ArrayList>` , che non fornisce al deserializzatore informazioni sufficienti per deserializzare correttamente un messaggio in arrivo.</span><span class="sxs-lookup"><span data-stu-id="337d9-151">For example, calling `getClass` on an `ArrayList<String>` would not return `Class<ArrayList<String>>`, but rather `Class<ArrayList>`, which does not give the deserializer enough information to correctly deserialize an incoming message.</span></span> <span data-ttu-id="337d9-152">Lo stesso vale per gli oggetti personalizzati.</span><span class="sxs-lookup"><span data-stu-id="337d9-152">The same is true for custom objects.</span></span>

::: moniker-end

## <a name="known-limitations"></a><span data-ttu-id="337d9-153">Limitazioni note</span><span class="sxs-lookup"><span data-stu-id="337d9-153">Known limitations</span></span>

::: moniker range=">= aspnetcore-5.0"

* <span data-ttu-id="337d9-154">Il trasporto del fallback del trasporto e degli eventi inviati dal server non è supportato.</span><span class="sxs-lookup"><span data-stu-id="337d9-154">Transport fallback and the Server Sent Events transport aren't supported.</span></span>

::: moniker-end

::: moniker range=">= aspnetcore-3.0 < aspnetcore-5.0"

* <span data-ttu-id="337d9-155">Il trasporto del fallback del trasporto e degli eventi inviati dal server non è supportato.</span><span class="sxs-lookup"><span data-stu-id="337d9-155">Transport fallback and the Server Sent Events transport aren't supported.</span></span>
* <span data-ttu-id="337d9-156">È supportato solo il protocollo JSON.</span><span class="sxs-lookup"><span data-stu-id="337d9-156">Only the JSON protocol is supported.</span></span>

::: moniker-end

::: moniker range="< aspnetcore-3.0"

* <span data-ttu-id="337d9-157">È supportato solo il protocollo JSON.</span><span class="sxs-lookup"><span data-stu-id="337d9-157">Only the JSON protocol is supported.</span></span>
* <span data-ttu-id="337d9-158">È supportato solo il trasporto WebSocket.</span><span class="sxs-lookup"><span data-stu-id="337d9-158">Only the WebSockets transport is supported.</span></span>
* <span data-ttu-id="337d9-159">Il flusso non è ancora supportato.</span><span class="sxs-lookup"><span data-stu-id="337d9-159">Streaming isn't supported yet.</span></span>

::: moniker-end

## <a name="additional-resources"></a><span data-ttu-id="337d9-160">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="337d9-160">Additional resources</span></span>

* [<span data-ttu-id="337d9-161">Informazioni di riferimento sulle API Java</span><span class="sxs-lookup"><span data-stu-id="337d9-161">Java API reference</span></span>](/java/api/com.microsoft.signalr?view=aspnet-signalr-java)
* <xref:signalr/hubs>
* <xref:signalr/javascript-client>
* <xref:signalr/publish-to-azure-web-app>
* [<span data-ttu-id="337d9-162">Documentazione senza server per i servizi di Azure SignalR</span><span class="sxs-lookup"><span data-stu-id="337d9-162">Azure SignalR Service serverless documentation</span></span>](/azure/azure-signalr/signalr-concept-serverless-development-config)
