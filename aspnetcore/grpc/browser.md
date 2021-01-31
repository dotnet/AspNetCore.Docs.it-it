---
title: Usare gRPC nelle app del browser
author: jamesnk
description: Informazioni su come configurare i servizi gRPC in ASP.NET Core per essere richiamabili dalle app del browser usando gRPC-Web.
monikerRange: '>= aspnetcore-3.0'
ms.author: jamesnk
ms.date: 06/30/2020
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
uid: grpc/browser
ms.openlocfilehash: 0967a70b498156d9c4ea8818ee1c80b37d9f2d87
ms.sourcegitcommit: 7e394a8527c9818caebb940f692ae4fcf2f1b277
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 01/31/2021
ms.locfileid: "99217479"
---
# <a name="use-grpc-in-browser-apps"></a>Usare gRPC nelle app del browser

Di [James Newton-King](https://twitter.com/jamesnk)

 Informazioni su come configurare un servizio di ASP.NET Core gRPC esistente da chiamare dalle app del browser, usando il protocollo [gRPC-Web](https://github.com/grpc/grpc/blob/2a388793792cc80944334535b7c729494d209a7e/doc/PROTOCOL-WEB.md) . gRPC-Web consente alle app e JavaScript del browser Blazor di chiamare i servizi gRPC. Non è possibile chiamare un servizio HTTP/2 gRPC da un'app basata su browser. i servizi gRPC ospitati in ASP.NET Core possono essere configurati per supportare gRPC-Web insieme a HTTP/2 gRPC.


Per istruzioni sull'aggiunta di un servizio gRPC a un'app ASP.NET Core esistente, vedere [aggiungere servizi gRPC a un'app ASP.NET Core](xref:grpc/aspnetcore#add-grpc-services-to-an-aspnet-core-app).

Per istruzioni sulla creazione di un progetto gRPC, vedere <xref:tutorials/grpc/grpc-start> .

## <a name="grpc-web-in-aspnet-core-vs-envoy"></a>gRPC-Web in ASP.NET Core e inviato

Sono disponibili due opzioni per l'aggiunta di gRPC-Web a un'app ASP.NET Core:

* Supportare gRPC-Web insieme a gRPC HTTP/2 in ASP.NET Core. Questa opzione utilizza il middleware fornito dal `Grpc.AspNetCore.Web` pacchetto.
* Usare il supporto gRPC-Web del [proxy inviato](https://www.envoyproxy.io/) per tradurre GRPC-Web in gRPC http/2. La chiamata tradotta viene quindi trasmessa nell'app ASP.NET Core.

Ogni approccio presenta vantaggi e svantaggi. Se l'ambiente di un'app usa già l'inviato come proxy, potrebbe essere utile usare anche l'inviato per fornire il supporto gRPC-Web. Per una soluzione di base per gRPC-Web che richiede solo ASP.NET Core, `Grpc.AspNetCore.Web` è una scelta ottimale.

## <a name="configure-grpc-web-in-aspnet-core"></a>Configurare gRPC-Web in ASP.NET Core

i servizi gRPC ospitati in ASP.NET Core possono essere configurati per supportare gRPC-Web insieme a HTTP/2 gRPC. gRPC-Web non richiede alcuna modifica ai servizi. L'unica modifica è la configurazione di avvio.

Per abilitare gRPC-Web con un servizio gRPC ASP.NET Core:

* Aggiungere un riferimento al pacchetto [Grpc. AspNetCore. Web](https://www.nuget.org/packages/Grpc.AspNetCore.Web) .
* Configurare l'app per l'uso di gRPC-Web aggiungendo `UseGrpcWeb` e `EnableGrpcWeb` a *Startup.cs*:

[!code-csharp[](~/grpc/browser/sample/Startup.cs?name=snippet_1&highlight=10,14)]

Il codice precedente:

* Aggiunge il middleware gRPC-Web, `UseGrpcWeb` , dopo il routing e prima degli endpoint.
* Specifica che il `endpoints.MapGrpcService<GreeterService>()` metodo supporta gRPC-Web con `EnableGrpcWeb` . 

In alternativa, è possibile configurare il middleware gRPC-Web in modo che tutti i servizi supportino gRPC-Web per impostazione predefinita e `EnableGrpcWeb` non siano necessari. Specificare `new GrpcWebOptions { DefaultEnabled = true }` quando viene aggiunto il middleware.

[!code-csharp[](~/grpc/browser/sample/AllServicesSupportExample_Startup.cs?name=snippet_1&highlight=12)]

> [!NOTE]
> Esiste un problema noto che causa l'esito negativo di gRPC-Web quando è [ospitato da HTTP.sys](xref:fundamentals/servers/httpsys) in .NET Core 3. x.
>
> Una soluzione alternativa per l'uso di gRPC-Web in HTTP.sys è disponibile [qui](https://github.com/grpc/grpc-dotnet/issues/853#issuecomment-610078202).

### <a name="grpc-web-and-cors"></a>gRPC-Web e CORS

La sicurezza del browser impedisce a una pagina Web di eseguire richieste a un dominio diverso da quello che ha gestito la pagina Web. Questa restrizione si applica all'esecuzione di chiamate gRPC-Web con le app browser. Ad esempio, un'app browser servita da `https://www.contoso.com` viene bloccata dalla chiamata a gRPC-Web Services ospitata in `https://services.contoso.com` . La condivisione di risorse tra le origini (CORS) può essere usata per attenuare questa restrizione.

Per consentire a un'app del browser di effettuare chiamate web gRPC tra le origini, configurare [CORS in ASP.NET Core](xref:security/cors). Usare il supporto CORS incorporato ed esporre intestazioni specifiche di gRPC con <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.WithExposedHeaders%2A> .

[!code-csharp[](~/grpc/browser/sample/CORS_Startup.cs?name=snippet_1&highlight=5-11,19,24)]

Il codice precedente:

* Chiama `AddCors` per aggiungere servizi CORS e configura un criterio CORS che espone intestazioni specifiche di gRPC.
* Chiama `UseCors` per aggiungere il middleware CORS dopo il routing e prima degli endpoint.
* Specifica che il `endpoints.MapGrpcService<GreeterService>()` metodo supporta cors con `RequiresCors` .

### <a name="grpc-web-and-streaming"></a>gRPC-Web e streaming

Il gRPC tradizionale su HTTP/2 supporta lo streaming in tutte le direzioni. gRPC-Web offre un supporto limitato per lo streaming:

* i client gRPC-Web browser non supportano la chiamata di flussi client e metodi di streaming bidirezionale.
* ASP.NET Core Servizi gRPC ospitati nel servizio app Azure e IIS non supportano lo streaming bidirezionale.

Quando si usa gRPC-Web, è consigliabile usare solo metodi unari e metodi di streaming del server.

## <a name="call-grpc-web-from-the-browser"></a>Chiamare gRPC-Web dal browser

Le app browser possono usare gRPC-Web per chiamare i servizi gRPC. Esistono alcuni requisiti e limitazioni quando si chiamano i servizi gRPC con gRPC-Web dal browser:

* Il server deve essere stato configurato per supportare gRPC-Web.
* Il flusso client e le chiamate di streaming bidirezionali non sono supportate. Il flusso del server è supportato.
* Per chiamare i servizi gRPC in un dominio diverso è necessario configurare [CORS](xref:security/cors) nel server.

### <a name="javascript-grpc-web-client"></a>JavaScript gRPC-client Web

È disponibile un client JavaScript gRPC-Web. Per istruzioni su come usare gRPC-Web da JavaScript, vedere [scrivere codice client JavaScript con gRPC-Web](https://github.com/grpc/grpc-web/tree/master/net/grpc/gateway/examples/helloworld#write-client-code).

### <a name="configure-grpc-web-with-the-net-grpc-client"></a>Configurare gRPC-Web con il client gRPC .NET

Il client gRPC .NET può essere configurato per effettuare chiamate gRPC-Web. Questa operazione è utile per [Blazor WebAssembly](xref:blazor/index#blazor-webassembly) le app, che sono ospitate nel browser e hanno le stesse limitazioni http del codice JavaScript. La chiamata di gRPC-Web con un client .NET equivale a [http/2 gRPC](xref:grpc/client). L'unica modifica è la modalità di creazione del canale.

Per usare gRPC-Web:

* Aggiungere un riferimento al pacchetto [Grpc .NET. client. Web](https://www.nuget.org/packages/Grpc.Net.Client.Web) .
* Verificare che il riferimento a [Grpc .NET. client](https://www.nuget.org/packages/Grpc.Net.Client) Package sia 2.29.0 o versione successiva.
* Configurare il canale per l'utilizzo di `GrpcWebHandler` :

[!code-csharp[](~/grpc/browser/sample/Handler.cs?name=snippet_1)]

Il codice precedente:

* Configura un canale per l'uso di gRPC-Web.
* Crea un client ed effettua una chiamata utilizzando il canale.

`GrpcWebHandler` in sono disponibili le opzioni di configurazione seguenti:

* **InnerHandler**: oggetto sottostante <xref:System.Net.Http.HttpMessageHandler> che esegue la richiesta http gRPC, ad esempio `HttpClientHandler` .
* **GrpcWebMode**: tipo di enumerazione che specifica se la richiesta http gRPC `Content-Type` è `application/grpc-web` o `application/grpc-web-text` .
    * `GrpcWebMode.GrpcWeb` Configura il contenuto da inviare senza codifica. Valore predefinito.
    * `GrpcWebMode.GrpcWebText` Configura il contenuto con codifica Base64. Obbligatorio per le chiamate di streaming del server nei browser.
* **HttpVersion**: protocollo http `Version` usato per impostare [HttpRequestMessage. Version](xref:System.Net.Http.HttpRequestMessage.Version) sulla richiesta http gRPC sottostante. gRPC-Web non richiede una versione specifica e non esegue l'override dell'impostazione predefinita, a meno che non sia specificato.

> [!IMPORTANT]
> I client gRPC generati hanno metodi Sync e Async per la chiamata di metodi unaria. Ad esempio, `SayHello` è Sync ed `SayHelloAsync` è Async. La chiamata di un metodo di sincronizzazione in un'app provocherà la mancata Blazor WebAssembly risposta dell'app. I metodi asincroni devono sempre essere utilizzati in Blazor WebAssembly .

### <a name="use-grpc-client-factory-with-grpc-web"></a>Usare la factory del client gRPC con gRPC-Web

Un client .NET compatibile con gRPC può essere creato usando l'integrazione di gRPC con [HttpClientFactory](xref:System.Net.Http.IHttpClientFactory).

Per usare gRPC-Web con client Factory:

* Aggiungere i riferimenti ai pacchetti al file di progetto per i pacchetti seguenti:
  * [Grpc .NET. client. Web](https://www.nuget.org/packages/Grpc.Net.Client.Web)
  * [Grpc .NET. ClientFactory](https://www.nuget.org/packages/Grpc.Net.ClientFactory)
* Registrare un client gRPC con l'inserimento DI dipendenze usando il `AddGrpcClient` metodo di estensione generico. In un' Blazor WebAssembly app, i servizi vengono registrati con l'inserimento di `Program.cs` .
* Configurare `GrpcWebHandler` usando il <xref:Microsoft.Extensions.DependencyInjection.HttpClientBuilderExtensions.ConfigurePrimaryHttpMessageHandler%2A> metodo di estensione.

```csharp
builder.Services
    .AddGrpcClient<Greet.GreeterClient>((services, options) =>
    {
        options.Address = new Uri("https://localhost:5001");
    })
    .ConfigurePrimaryHttpMessageHandler(
        () => new GrpcWebHandler(GrpcWebMode.GrpcWebText, new HttpClientHandler()));
```

Per altre informazioni, vedere <xref:grpc/clientfactory>.

## <a name="additional-resources"></a>Risorse aggiuntive

* [progetto GitHub di gRPC per client Web](https://github.com/grpc/grpc-web)
* <xref:security/cors>
* <xref:grpc/httpapi>
