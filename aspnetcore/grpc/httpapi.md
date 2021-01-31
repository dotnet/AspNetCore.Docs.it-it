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
# <a name="create-json-web-apis-from-grpc"></a><span data-ttu-id="06ea3-103">Create API Web JSON da gRPC</span><span class="sxs-lookup"><span data-stu-id="06ea3-103">Create JSON Web APIs from gRPC</span></span>

<span data-ttu-id="06ea3-104">Di [James Newton-King](https://twitter.com/jamesnk)</span><span class="sxs-lookup"><span data-stu-id="06ea3-104">By [James Newton-King](https://twitter.com/jamesnk)</span></span>

> [!IMPORTANT]
> <span data-ttu-id="06ea3-105">l'API HTTP gRPC è un progetto sperimentale, non un prodotto di cui è stato eseguito il commit.</span><span class="sxs-lookup"><span data-stu-id="06ea3-105">gRPC HTTP API is an experimental project, not a committed product.</span></span> <span data-ttu-id="06ea3-106">Si desidera:</span><span class="sxs-lookup"><span data-stu-id="06ea3-106">We want to:</span></span>
>
> * <span data-ttu-id="06ea3-107">Verificare che l'approccio alla creazione di API Web JSON per i servizi gRPC funzioni.</span><span class="sxs-lookup"><span data-stu-id="06ea3-107">Test that our approach to creating JSON Web APIs for gRPC services works.</span></span>
> * <span data-ttu-id="06ea3-108">Ottenere commenti e suggerimenti su se questo approccio è utile per gli sviluppatori .NET.</span><span class="sxs-lookup"><span data-stu-id="06ea3-108">Get feedback on if this approach is useful to .NET developers.</span></span>
>
> <span data-ttu-id="06ea3-109">[Invia commenti e suggerimenti](https://github.com/grpc/grpc-dotnet/issues/167) per assicurarsi di creare qualcosa che gli sviluppatori desiderano e siano produttivi.</span><span class="sxs-lookup"><span data-stu-id="06ea3-109">Please [leave feedback](https://github.com/grpc/grpc-dotnet/issues/167) to ensure we build something that developers like and are productive with.</span></span>

<span data-ttu-id="06ea3-110">gRPC è un modo moderno per comunicare tra le app.</span><span class="sxs-lookup"><span data-stu-id="06ea3-110">gRPC is a modern way to communicate between apps.</span></span> <span data-ttu-id="06ea3-111">gRPC Usa HTTP/2, flusso, protobuf e contratti di messaggio per creare servizi in tempo reale ad alte prestazioni.</span><span class="sxs-lookup"><span data-stu-id="06ea3-111">gRPC uses HTTP/2, streaming, Protobuf and message contracts to create high-performance, real-time services.</span></span>

<span data-ttu-id="06ea3-112">Una limitazione di gRPC non è la piattaforma che può essere usata da tutte le piattaforme.</span><span class="sxs-lookup"><span data-stu-id="06ea3-112">One limitation with gRPC is not every platform can use it.</span></span> <span data-ttu-id="06ea3-113">I browser non supportano completamente HTTP/2, rendendo REST e JSON il modo principale per ottenere i dati nelle app del browser.</span><span class="sxs-lookup"><span data-stu-id="06ea3-113">Browsers don't fully support HTTP/2, making REST and JSON the primary way to get data into browser apps.</span></span> <span data-ttu-id="06ea3-114">Anche con i vantaggi offerti da gRPC, REST e JSON hanno un posto importante nelle app moderne.</span><span class="sxs-lookup"><span data-stu-id="06ea3-114">Even with the benefits that gRPC brings, REST and JSON have an important place in modern apps.</span></span> <span data-ttu-id="06ea3-115">La compilazione di API Web gRPC \***e** _ JSON aggiunge un sovraccarico indesiderato per lo sviluppo di app.</span><span class="sxs-lookup"><span data-stu-id="06ea3-115">Building gRPC \***and** _ JSON Web APIs adds unwanted overhead to app development.</span></span>

<span data-ttu-id="06ea3-116">Questo documento illustra come creare API Web JSON usando i servizi gRPC.</span><span class="sxs-lookup"><span data-stu-id="06ea3-116">This document discusses how to create JSON Web APIs using gRPC services.</span></span>

## <a name="grpc-http-api"></a><span data-ttu-id="06ea3-117">API HTTP gRPC</span><span class="sxs-lookup"><span data-stu-id="06ea3-117">gRPC HTTP API</span></span>

<span data-ttu-id="06ea3-118">l'API HTTP di gRPC è un'estensione sperimentale per ASP.NET Core che crea API JSON RESTful per i servizi gRPC.</span><span class="sxs-lookup"><span data-stu-id="06ea3-118">gRPC HTTP API is an experimental extension for ASP.NET Core that creates RESTful JSON APIs for gRPC services.</span></span> <span data-ttu-id="06ea3-119">Una volta configurata, l'API HTTP gRPC consente alle app di chiamare i servizi gRPC con concetti HTTP noti:</span><span class="sxs-lookup"><span data-stu-id="06ea3-119">Once configured, gRPC HTTP API allows apps to call gRPC services with familiar HTTP concepts:</span></span>

<span data-ttu-id="06ea3-120">_ Verbi HTTP</span><span class="sxs-lookup"><span data-stu-id="06ea3-120">_ HTTP verbs</span></span>
* <span data-ttu-id="06ea3-121">Binding parametro URL</span><span class="sxs-lookup"><span data-stu-id="06ea3-121">URL parameter binding</span></span>
* <span data-ttu-id="06ea3-122">Richieste/risposte JSON</span><span class="sxs-lookup"><span data-stu-id="06ea3-122">JSON requests/responses</span></span>

<span data-ttu-id="06ea3-123">gRPC può comunque essere usato per chiamare i servizi.</span><span class="sxs-lookup"><span data-stu-id="06ea3-123">gRPC can still be used to call services.</span></span>

### <a name="usage"></a><span data-ttu-id="06ea3-124">Utilizzo</span><span class="sxs-lookup"><span data-stu-id="06ea3-124">Usage</span></span>

1. <span data-ttu-id="06ea3-125">Aggiungere un riferimento al pacchetto a [Microsoft. AspNetCore. Grpc. HttpApi](https://www.nuget.org/packages/Microsoft.AspNetCore.Grpc.HttpApi).</span><span class="sxs-lookup"><span data-stu-id="06ea3-125">Add a package reference to [Microsoft.AspNetCore.Grpc.HttpApi](https://www.nuget.org/packages/Microsoft.AspNetCore.Grpc.HttpApi).</span></span>
1. <span data-ttu-id="06ea3-126">Registrare i servizi in *Startup.cs* con `AddGrpcHttpApi` .</span><span class="sxs-lookup"><span data-stu-id="06ea3-126">Register services in *Startup.cs* with `AddGrpcHttpApi`.</span></span>
1. <span data-ttu-id="06ea3-127">Aggiungere i file [Google/API/HTTP. proto](https://github.com/aspnet/AspLabs/blob/c1e59cacf7b9606650d6ec38e54fa3a82377f360/src/GrpcHttpApi/sample/Proto/google/api/http.proto) e [Google/API/Annotations. proto](https://github.com/aspnet/AspLabs/blob/c1e59cacf7b9606650d6ec38e54fa3a82377f360/src/GrpcHttpApi/sample/Proto/google/api/annotations.proto) al progetto.</span><span class="sxs-lookup"><span data-stu-id="06ea3-127">Add [google/api/http.proto](https://github.com/aspnet/AspLabs/blob/c1e59cacf7b9606650d6ec38e54fa3a82377f360/src/GrpcHttpApi/sample/Proto/google/api/http.proto) and [google/api/annotations.proto](https://github.com/aspnet/AspLabs/blob/c1e59cacf7b9606650d6ec38e54fa3a82377f360/src/GrpcHttpApi/sample/Proto/google/api/annotations.proto) files to your project.</span></span>
1. <span data-ttu-id="06ea3-128">Annotare i metodi gRPC nei file *. proto* con binding e route http:</span><span class="sxs-lookup"><span data-stu-id="06ea3-128">Annotate gRPC methods in your *.proto* files with HTTP bindings and routes:</span></span>

[!code-protobuf[](~/grpc/httpapi/greet.proto?highlight=3,9-11)]

<span data-ttu-id="06ea3-129">Il `SayHello` Metodo gRPC può ora essere richiamato come gRPC + protobuf e come API http:</span><span class="sxs-lookup"><span data-stu-id="06ea3-129">The `SayHello` gRPC method can now be invoked as gRPC+Protobuf and as an HTTP API:</span></span>

* <span data-ttu-id="06ea3-130">Richiesta: `HTTP/1.1 GET /v1/greeter/world`</span><span class="sxs-lookup"><span data-stu-id="06ea3-130">Request: `HTTP/1.1 GET /v1/greeter/world`</span></span>
* <span data-ttu-id="06ea3-131">Risposta: `{ "message": "Hello world" }`</span><span class="sxs-lookup"><span data-stu-id="06ea3-131">Response: `{ "message": "Hello world" }`</span></span>

<span data-ttu-id="06ea3-132">I log del server indicano che la chiamata HTTP viene eseguita da un servizio gRPC.</span><span class="sxs-lookup"><span data-stu-id="06ea3-132">Server logs show that the HTTP call is executed by a gRPC service.</span></span> <span data-ttu-id="06ea3-133">l'API HTTP gRPC esegue il mapping della richiesta HTTP in ingresso a un messaggio gRPC e quindi converte il messaggio di risposta in JSON.</span><span class="sxs-lookup"><span data-stu-id="06ea3-133">gRPC HTTP API maps the incoming HTTP request to a gRPC message, and then converts the response message to JSON.</span></span>

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

<span data-ttu-id="06ea3-134">Questo è un esempio di base.</span><span class="sxs-lookup"><span data-stu-id="06ea3-134">This is a basic example.</span></span> <span data-ttu-id="06ea3-135">Per altre opzioni di personalizzazione, vedere [HttpRule](https://cloud.google.com/service-infrastructure/docs/service-management/reference/rpc/google.api#google.api.HttpRule) .</span><span class="sxs-lookup"><span data-stu-id="06ea3-135">See [HttpRule](https://cloud.google.com/service-infrastructure/docs/service-management/reference/rpc/google.api#google.api.HttpRule) for more customization options.</span></span>

### <a name="enable-swaggeropenapi-support"></a><span data-ttu-id="06ea3-136">Abilita supporto di spavalderia/OpenAPI</span><span class="sxs-lookup"><span data-stu-id="06ea3-136">Enable Swagger/OpenAPI support</span></span>

<span data-ttu-id="06ea3-137">Spavalderia (OpenAPI) è una specifica indipendente dal linguaggio per la descrizione delle API REST.</span><span class="sxs-lookup"><span data-stu-id="06ea3-137">Swagger (OpenAPI) is a language-agnostic specification for describing REST APIs.</span></span> <span data-ttu-id="06ea3-138">l'API HTTP gRPC può essere integrata con [Swashbuckle](https://github.com/domaindrivendev/Swashbuckle.AspNetCore) per generare un endpoint di spavalderia per i servizi gRPC RESTful.</span><span class="sxs-lookup"><span data-stu-id="06ea3-138">gRPC HTTP API can integrate with [Swashbuckle](https://github.com/domaindrivendev/Swashbuckle.AspNetCore) to generate a Swagger endpoint for RESTful gRPC services.</span></span> <span data-ttu-id="06ea3-139">L'endpoint di spavalderia può quindi essere usato con l' [interfaccia utente di spavalderia](https://swagger.io/swagger-ui/) e altri strumenti.</span><span class="sxs-lookup"><span data-stu-id="06ea3-139">The Swagger endpoint can then be used with [Swagger UI](https://swagger.io/swagger-ui/) and other tooling.</span></span>

<span data-ttu-id="06ea3-140">Per abilitare la spavalderia con l'API HTTP gRPC:</span><span class="sxs-lookup"><span data-stu-id="06ea3-140">To enable Swagger with gRPC HTTP API:</span></span>

1. <span data-ttu-id="06ea3-141">Aggiungere un riferimento al pacchetto a [Microsoft. AspNetCore. Grpc. spavalderia](https://www.nuget.org/packages/Microsoft.AspNetCore.Grpc.Swagger).</span><span class="sxs-lookup"><span data-stu-id="06ea3-141">Add a package reference to [Microsoft.AspNetCore.Grpc.Swagger](https://www.nuget.org/packages/Microsoft.AspNetCore.Grpc.Swagger).</span></span>
2. <span data-ttu-id="06ea3-142">Configurare Swashbuckle in *Startup.cs*.</span><span class="sxs-lookup"><span data-stu-id="06ea3-142">Configure Swashbuckle in *Startup.cs*.</span></span> <span data-ttu-id="06ea3-143">Il `AddGrpcSwagger` metodo configura Swashbuckle in modo da includere gli endpoint dell'API HTTP di gRPC.</span><span class="sxs-lookup"><span data-stu-id="06ea3-143">The `AddGrpcSwagger` method configures Swashbuckle to include gRPC HTTP API endpoints.</span></span>

[!code-csharp[](~/grpc/httpapi/Startup.cs?name=snippet_1&highlight=6-10,15-19)]

<span data-ttu-id="06ea3-144">Per confermare che Swashbuckle genera spavalderia per i servizi gRPC RESTful, avviare l'app e passare alla pagina dell'interfaccia utente di spavalderia:</span><span class="sxs-lookup"><span data-stu-id="06ea3-144">To confirm that Swashbuckle is generating Swagger for the RESTful gRPC services, start the app and navigate to the Swagger UI page:</span></span>

![Interfaccia utente di Swagger](~/grpc/httpapi/static/swaggerui.png)

### <a name="grpc-http-api-vs-grpc-web"></a><span data-ttu-id="06ea3-146">API gRPC HTTP vs gRPC-Web</span><span class="sxs-lookup"><span data-stu-id="06ea3-146">gRPC HTTP API vs gRPC-Web</span></span>

<span data-ttu-id="06ea3-147">Sia l'API HTTP gRPC che gRPC-Web consentono di chiamare i servizi gRPC da un browser.</span><span class="sxs-lookup"><span data-stu-id="06ea3-147">Both gRPC HTTP API and gRPC-Web allow gRPC services to be called from a browser.</span></span> <span data-ttu-id="06ea3-148">Tuttavia, il modo in cui ogni operazione è diversa:</span><span class="sxs-lookup"><span data-stu-id="06ea3-148">However, the way each does this is different:</span></span>

* <span data-ttu-id="06ea3-149">gRPC-Web consente alle app del browser di chiamare i servizi gRPC dal browser con gRPC-Web client e protobuf.</span><span class="sxs-lookup"><span data-stu-id="06ea3-149">gRPC-Web lets browser apps call gRPC services from the browser with the gRPC-Web client and Protobuf.</span></span> <span data-ttu-id="06ea3-150">gRPC-Web richiede che l'app browser generi un client gRPC e abbia il vantaggio di inviare messaggi protobuf piccoli e veloci.</span><span class="sxs-lookup"><span data-stu-id="06ea3-150">gRPC-Web requires the browser app generate a gRPC client, and has the advantage of sending small, fast Protobuf messages.</span></span>
* <span data-ttu-id="06ea3-151">l'API HTTP gRPC consente alle app browser di chiamare i servizi gRPC come se fossero API RESTful con JSON.</span><span class="sxs-lookup"><span data-stu-id="06ea3-151">gRPC HTTP API allows browser apps to call gRPC services as if they were RESTful APIs with JSON.</span></span> <span data-ttu-id="06ea3-152">Non è necessario che l'app browser generi un client gRPC o non conosca alcun valore di gRPC.</span><span class="sxs-lookup"><span data-stu-id="06ea3-152">The browser app doesn't need to generate a gRPC client or know anything about gRPC.</span></span>

<span data-ttu-id="06ea3-153">Non viene creato alcun client generato per l'API HTTP gRPC.</span><span class="sxs-lookup"><span data-stu-id="06ea3-153">No generated client is created for gRPC HTTP API.</span></span> <span data-ttu-id="06ea3-154">Il `Greeter` servizio precedente può essere chiamato usando le API JavaScript del browser:</span><span class="sxs-lookup"><span data-stu-id="06ea3-154">The previous `Greeter` service can be called using browser JavaScript APIs:</span></span>

```javascript
var name = nameInput.value;

fetch("/v1/greeter/" + name).then(function (response) {
  response.json().then(function (data) {
    console.log("Result: " + data.message);
  });
});
```

### <a name="experimental-status"></a><span data-ttu-id="06ea3-155">Stato sperimentale</span><span class="sxs-lookup"><span data-stu-id="06ea3-155">Experimental status</span></span>

<span data-ttu-id="06ea3-156">l'API HTTP di gRPC è un esperimento.</span><span class="sxs-lookup"><span data-stu-id="06ea3-156">gRPC HTTP API is an experiment.</span></span> <span data-ttu-id="06ea3-157">Non è completa e non è supportata.</span><span class="sxs-lookup"><span data-stu-id="06ea3-157">It is not complete and it is not supported.</span></span> <span data-ttu-id="06ea3-158">Siamo interessati a questa tecnologia e alla possibilità che gli sviluppatori di app creino rapidamente i servizi gRPC e JSON nello stesso momento.</span><span class="sxs-lookup"><span data-stu-id="06ea3-158">We're interested in this technology, and the ability it gives app developers to quickly create gRPC and JSON services at the same time.</span></span> <span data-ttu-id="06ea3-159">Non sono previsti impegni per completare l'API HTTP di gRPC.</span><span class="sxs-lookup"><span data-stu-id="06ea3-159">There is no commitment to completing the gRPC HTTP API.</span></span>

<span data-ttu-id="06ea3-160">Si vuole misurare gli interessi degli sviluppatori nell'API HTTP di gRPC.</span><span class="sxs-lookup"><span data-stu-id="06ea3-160">We want to gauge developer interest in gRPC HTTP API.</span></span> <span data-ttu-id="06ea3-161">Se l'API HTTP di gRPC è interessante, inviare [commenti e suggerimenti](https://github.com/grpc/grpc-dotnet/issues/167).</span><span class="sxs-lookup"><span data-stu-id="06ea3-161">If gRPC HTTP API is interesting to you then please [give feedback](https://github.com/grpc/grpc-dotnet/issues/167).</span></span>

## <a name="grpc-gateway"></a><span data-ttu-id="06ea3-162">grpc-gateway</span><span class="sxs-lookup"><span data-stu-id="06ea3-162">grpc-gateway</span></span>

<span data-ttu-id="06ea3-163">[grpc-gateway](https://grpc-ecosystem.github.io/grpc-gateway/) è un'altra tecnologia per la creazione di API JSON RESTful dai servizi di grpc.</span><span class="sxs-lookup"><span data-stu-id="06ea3-163">[grpc-gateway](https://grpc-ecosystem.github.io/grpc-gateway/) is another technology for creating RESTful JSON APIs from gRPC services.</span></span> <span data-ttu-id="06ea3-164">Usa le stesse annotazioni *. proto* per eseguire il mapping dei concetti http ai servizi gRPC.</span><span class="sxs-lookup"><span data-stu-id="06ea3-164">It uses the same *.proto* annotations to map HTTP concepts to gRPC services.</span></span>

<span data-ttu-id="06ea3-165">La differenza principale tra grpc-gateway e l'API HTTP gRPC è grpc-gateway usa la generazione del codice per creare un server proxy inverso.</span><span class="sxs-lookup"><span data-stu-id="06ea3-165">The biggest difference between grpc-gateway and gRPC HTTP API is grpc-gateway uses code generation to create a reverse-proxy server.</span></span> <span data-ttu-id="06ea3-166">Il proxy inverso converte le chiamate RESTful in gRPC e quindi le invia al servizio gRPC.</span><span class="sxs-lookup"><span data-stu-id="06ea3-166">The reverse-proxy translates RESTful calls into gRPC and then sends them on to the gRPC service.</span></span>

<span data-ttu-id="06ea3-167">Per l'installazione e l'uso di grpc-gateway, vedere il [file Leggimi di grpc-gateway](https://github.com/grpc-ecosystem/grpc-gateway/#grpc-gateway).</span><span class="sxs-lookup"><span data-stu-id="06ea3-167">For installation and usage of grpc-gateway, see the [grpc-gateway README](https://github.com/grpc-ecosystem/grpc-gateway/#grpc-gateway).</span></span>

## <a name="additional-resources"></a><span data-ttu-id="06ea3-168">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="06ea3-168">Additional resources</span></span>

* [<span data-ttu-id="06ea3-169">documentazione di Google. API. HttpRule</span><span class="sxs-lookup"><span data-stu-id="06ea3-169">google.api.HttpRule documentation</span></span>](https://cloud.google.com/service-infrastructure/docs/service-management/reference/rpc/google.api#google.api.HttpRule)
* <xref:grpc/browser>
