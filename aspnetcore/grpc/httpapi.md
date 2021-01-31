---
title: Create API Web JSON da gRPC
author: jamesnk
description: Informazioni su come creare API HTTP JSON per i servizi gRPC.
monikerRange: '>= aspnetcore-3.0'
ms.author: jamesnk
ms.date: 08/28/2020
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
uid: grpc/httpapi
ms.openlocfilehash: 78247a9b775ec8c9a3bc4a58209f09e5b714c07d
ms.sourcegitcommit: 7e394a8527c9818caebb940f692ae4fcf2f1b277
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 01/31/2021
ms.locfileid: "99217518"
---
# <a name="create-json-web-apis-from-grpc"></a>Create API Web JSON da gRPC

Di [James Newton-King](https://twitter.com/jamesnk)

> [!IMPORTANT]
> l'API HTTP gRPC è un progetto sperimentale, non un prodotto di cui è stato eseguito il commit. Si desidera:
>
> * Verificare che l'approccio alla creazione di API Web JSON per i servizi gRPC funzioni.
> * Ottenere commenti e suggerimenti su se questo approccio è utile per gli sviluppatori .NET.
>
> [Invia commenti e suggerimenti](https://github.com/grpc/grpc-dotnet/issues/167) per assicurarsi di creare qualcosa che gli sviluppatori desiderano e siano produttivi.

gRPC è un modo moderno per comunicare tra le app. gRPC Usa HTTP/2, flusso, protobuf e contratti di messaggio per creare servizi in tempo reale ad alte prestazioni.

Una limitazione di gRPC non è la piattaforma che può essere usata da tutte le piattaforme. I browser non supportano completamente HTTP/2, rendendo REST e JSON il modo principale per ottenere i dati nelle app del browser. Anche con i vantaggi offerti da gRPC, REST e JSON hanno un posto importante nelle app moderne. La compilazione di API Web gRPC ***e** _ JSON aggiunge un sovraccarico indesiderato per lo sviluppo di app.

Questo documento illustra come creare API Web JSON usando i servizi gRPC.

## <a name="grpc-http-api"></a>API HTTP gRPC

l'API HTTP di gRPC è un'estensione sperimentale per ASP.NET Core che crea API JSON RESTful per i servizi gRPC. Una volta configurata, l'API HTTP gRPC consente alle app di chiamare i servizi gRPC con concetti HTTP noti:

_ Verbi HTTP
* Binding parametro URL
* Richieste/risposte JSON

gRPC può comunque essere usato per chiamare i servizi.

### <a name="usage"></a>Utilizzo

1. Aggiungere un riferimento al pacchetto a [Microsoft. AspNetCore. Grpc. HttpApi](https://www.nuget.org/packages/Microsoft.AspNetCore.Grpc.HttpApi).
1. Registrare i servizi in *Startup.cs* con `AddGrpcHttpApi` .
1. Aggiungere i file [Google/API/HTTP. proto](https://github.com/aspnet/AspLabs/blob/c1e59cacf7b9606650d6ec38e54fa3a82377f360/src/GrpcHttpApi/sample/Proto/google/api/http.proto) e [Google/API/Annotations. proto](https://github.com/aspnet/AspLabs/blob/c1e59cacf7b9606650d6ec38e54fa3a82377f360/src/GrpcHttpApi/sample/Proto/google/api/annotations.proto) al progetto.
1. Annotare i metodi gRPC nei file *. proto* con binding e route http:

[!code-protobuf[](~/grpc/httpapi/greet.proto?highlight=3,9-11)]

Il `SayHello` Metodo gRPC può ora essere richiamato come gRPC + protobuf e come API http:

* Richiesta: `HTTP/1.1 GET /v1/greeter/world`
* Risposta: `{ "message": "Hello world" }`

I log del server indicano che la chiamata HTTP viene eseguita da un servizio gRPC. l'API HTTP gRPC esegue il mapping della richiesta HTTP in ingresso a un messaggio gRPC e quindi converte il messaggio di risposta in JSON.

```
info: Microsoft.AspNetCore.Hosting.Diagnostics[1]
      Request starting HTTP/1.1 GET https://localhost:5001/v1/greeter/world
info: Microsoft.AspNetCore.Routing.EndpointMiddleware[0]
      Executing endpoint 'gRPC - /v1/greeter/{name}'
info: Server.GreeterService[0]
      Sending hello to world
info: Microsoft.AspNetCore.Routing.EndpointMiddleware[1]
      Executed endpoint 'gRPC - /v1/greeter/{name}'
info: Microsoft.AspNetCore.Hosting.Diagnostics[2]
      Request finished in 1.996ms 200 application/json
```

Questo è un esempio di base. Per altre opzioni di personalizzazione, vedere [HttpRule](https://cloud.google.com/service-infrastructure/docs/service-management/reference/rpc/google.api#google.api.HttpRule) .

### <a name="enable-swaggeropenapi-support"></a>Abilita supporto di spavalderia/OpenAPI

Spavalderia (OpenAPI) è una specifica indipendente dal linguaggio per la descrizione delle API REST. l'API HTTP gRPC può essere integrata con [Swashbuckle](https://github.com/domaindrivendev/Swashbuckle.AspNetCore) per generare un endpoint di spavalderia per i servizi gRPC RESTful. L'endpoint di spavalderia può quindi essere usato con l' [interfaccia utente di spavalderia](https://swagger.io/swagger-ui/) e altri strumenti.

Per abilitare la spavalderia con l'API HTTP gRPC:

1. Aggiungere un riferimento al pacchetto a [Microsoft. AspNetCore. Grpc. spavalderia](https://www.nuget.org/packages/Microsoft.AspNetCore.Grpc.Swagger).
2. Configurare Swashbuckle in *Startup.cs*. Il `AddGrpcSwagger` metodo configura Swashbuckle in modo da includere gli endpoint dell'API HTTP di gRPC.

[!code-csharp[](~/grpc/httpapi/Startup.cs?name=snippet_1&highlight=6-10,15-19)]

Per confermare che Swashbuckle genera spavalderia per i servizi gRPC RESTful, avviare l'app e passare alla pagina dell'interfaccia utente di spavalderia:

![Interfaccia utente di Swagger](~/grpc/httpapi/static/swaggerui.png)

### <a name="grpc-http-api-vs-grpc-web"></a>API gRPC HTTP vs gRPC-Web

Sia l'API HTTP gRPC che gRPC-Web consentono di chiamare i servizi gRPC da un browser. Tuttavia, il modo in cui ogni operazione è diversa:

* gRPC-Web consente alle app del browser di chiamare i servizi gRPC dal browser con gRPC-Web client e protobuf. gRPC-Web richiede che l'app browser generi un client gRPC e abbia il vantaggio di inviare messaggi protobuf piccoli e veloci.
* l'API HTTP gRPC consente alle app browser di chiamare i servizi gRPC come se fossero API RESTful con JSON. Non è necessario che l'app browser generi un client gRPC o non conosca alcun valore di gRPC.

Non viene creato alcun client generato per l'API HTTP gRPC. Il `Greeter` servizio precedente può essere chiamato usando le API JavaScript del browser:

```javascript
var name = nameInput.value;

fetch("/v1/greeter/" + name).then(function (response) {
  response.json().then(function (data) {
    console.log("Result: " + data.message);
  });
});
```

### <a name="experimental-status"></a>Stato sperimentale

l'API HTTP di gRPC è un esperimento. Non è completa e non è supportata. Siamo interessati a questa tecnologia e alla possibilità che gli sviluppatori di app creino rapidamente i servizi gRPC e JSON nello stesso momento. Non sono previsti impegni per completare l'API HTTP di gRPC.

Si vuole misurare gli interessi degli sviluppatori nell'API HTTP di gRPC. Se l'API HTTP di gRPC è interessante, inviare [commenti e suggerimenti](https://github.com/grpc/grpc-dotnet/issues/167).

## <a name="grpc-gateway"></a>grpc-gateway

[grpc-gateway](https://grpc-ecosystem.github.io/grpc-gateway/) è un'altra tecnologia per la creazione di API JSON RESTful dai servizi di grpc. Usa le stesse annotazioni *. proto* per eseguire il mapping dei concetti http ai servizi gRPC.

La differenza principale tra grpc-gateway e l'API HTTP gRPC è grpc-gateway usa la generazione del codice per creare un server proxy inverso. Il proxy inverso converte le chiamate RESTful in gRPC e quindi le invia al servizio gRPC.

Per l'installazione e l'uso di grpc-gateway, vedere il [file Leggimi di grpc-gateway](https://github.com/grpc-ecosystem/grpc-gateway/#grpc-gateway).

## <a name="additional-resources"></a>Risorse aggiuntive

* [documentazione di Google. API. HttpRule](https://cloud.google.com/service-infrastructure/docs/service-management/reference/rpc/google.api#google.api.HttpRule)
* <xref:grpc/browser>
