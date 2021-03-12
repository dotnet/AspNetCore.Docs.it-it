---
title: Effettuare richieste HTTP usando IHttpClientFactory in ASP.NET Core
author: stevejgordon
description: Informazioni sull'uso dell'interfaccia IHttpClientFactory per gestire le istanze di HttpClient logiche in ASP.NET Core.
monikerRange: '>= aspnetcore-2.1'
ms.author: scaddie
ms.custom: mvc
ms.date: 1/21/2021
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
uid: fundamentals/http-requests
ms.openlocfilehash: 2bc093af63f305dd9808e37011223043646852d5
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102588048"
---
# <a name="make-http-requests-using-ihttpclientfactory-in-aspnet-core"></a><span data-ttu-id="4b675-103">Effettuare richieste HTTP usando IHttpClientFactory in ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="4b675-103">Make HTTP requests using IHttpClientFactory in ASP.NET Core</span></span>

::: moniker range=">= aspnetcore-3.0"

<span data-ttu-id="4b675-104">Di [Kirk Larkin](https://github.com/serpent5), [Steve Gordon](https://github.com/stevejgordon), [Glenn Condron](https://github.com/glennc)e [Ryan Nowak](https://github.com/rynowak).</span><span class="sxs-lookup"><span data-stu-id="4b675-104">By [Kirk Larkin](https://github.com/serpent5), [Steve Gordon](https://github.com/stevejgordon), [Glenn Condron](https://github.com/glennc), and [Ryan Nowak](https://github.com/rynowak).</span></span>

<span data-ttu-id="4b675-105">È possibile registrare e usare un'interfaccia <xref:System.Net.Http.IHttpClientFactory> per configurare e creare istanze di <xref:System.Net.Http.HttpClient> in un'app.</span><span class="sxs-lookup"><span data-stu-id="4b675-105">An <xref:System.Net.Http.IHttpClientFactory> can be registered and used to configure and create <xref:System.Net.Http.HttpClient> instances in an app.</span></span> <span data-ttu-id="4b675-106">`IHttpClientFactory` offre i vantaggi seguenti:</span><span class="sxs-lookup"><span data-stu-id="4b675-106">`IHttpClientFactory` offers the following benefits:</span></span>

* <span data-ttu-id="4b675-107">Offre una posizione centrale per la denominazione e la configurazione di istanze di `HttpClient` logiche.</span><span class="sxs-lookup"><span data-stu-id="4b675-107">Provides a central location for naming and configuring logical `HttpClient` instances.</span></span> <span data-ttu-id="4b675-108">Ad esempio, un client denominato  *GitHub* potrebbe essere registrato e configurato per accedere a [GitHub](https://github.com/).</span><span class="sxs-lookup"><span data-stu-id="4b675-108">For example, a client named  *github* could be registered and configured to access [GitHub](https://github.com/).</span></span> <span data-ttu-id="4b675-109">Un client predefinito può essere registrato per l'accesso generale.</span><span class="sxs-lookup"><span data-stu-id="4b675-109">A default client can be registered for general access.</span></span>
* <span data-ttu-id="4b675-110">Codifica il concetto di middleware in uscita tramite la delega dei gestori in `HttpClient` .</span><span class="sxs-lookup"><span data-stu-id="4b675-110">Codifies the concept of outgoing middleware via delegating handlers in `HttpClient`.</span></span> <span data-ttu-id="4b675-111">Fornisce le estensioni per il middleware basato su Polly per sfruttare i vantaggi delegare i gestori in `HttpClient` .</span><span class="sxs-lookup"><span data-stu-id="4b675-111">Provides extensions for Polly-based middleware to take advantage of delegating handlers in `HttpClient`.</span></span>
* <span data-ttu-id="4b675-112">Gestisce il pool e la durata delle istanze sottostanti `HttpClientMessageHandler` .</span><span class="sxs-lookup"><span data-stu-id="4b675-112">Manages the pooling and lifetime of underlying `HttpClientMessageHandler` instances.</span></span> <span data-ttu-id="4b675-113">La gestione automatica evita i problemi comuni di DNS (Domain Name System) che si verificano quando si gestiscono manualmente le `HttpClient` durate.</span><span class="sxs-lookup"><span data-stu-id="4b675-113">Automatic management avoids common DNS (Domain Name System) problems that occur when manually managing `HttpClient` lifetimes.</span></span>
* <span data-ttu-id="4b675-114">Aggiunge un'esperienza di registrazione configurabile, tramite `ILogger`, per tutte le richieste inviate attraverso i client creati dalla factory.</span><span class="sxs-lookup"><span data-stu-id="4b675-114">Adds a configurable logging experience (via `ILogger`) for all requests sent through clients created by the factory.</span></span>

<span data-ttu-id="4b675-115">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/http-requests/samples) ([procedura per il download](xref:index#how-to-download-a-sample)).</span><span class="sxs-lookup"><span data-stu-id="4b675-115">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/http-requests/samples) ([how to download](xref:index#how-to-download-a-sample)).</span></span>

<span data-ttu-id="4b675-116">Il codice di esempio in questo argomento version USA <xref:System.Text.Json> per deserializzare il contenuto JSON restituito nelle risposte http.</span><span class="sxs-lookup"><span data-stu-id="4b675-116">The sample code in this topic version uses <xref:System.Text.Json> to deserialize JSON content returned in HTTP responses.</span></span> <span data-ttu-id="4b675-117">Per esempi che usano `Json.NET` e `ReadAsAsync<T>` , usare il selettore di versione per selezionare una versione 2. x di questo argomento.</span><span class="sxs-lookup"><span data-stu-id="4b675-117">For samples that use `Json.NET` and `ReadAsAsync<T>`, use the version selector to select a 2.x version of this topic.</span></span>

## <a name="consumption-patterns"></a><span data-ttu-id="4b675-118">Modelli di consumo</span><span class="sxs-lookup"><span data-stu-id="4b675-118">Consumption patterns</span></span>

<span data-ttu-id="4b675-119">`IHttpClientFactory` può essere usato in un'app in diversi modi:</span><span class="sxs-lookup"><span data-stu-id="4b675-119">There are several ways `IHttpClientFactory` can be used in an app:</span></span>

* [<span data-ttu-id="4b675-120">Uso di base</span><span class="sxs-lookup"><span data-stu-id="4b675-120">Basic usage</span></span>](#basic-usage)
* [<span data-ttu-id="4b675-121">Client denominati</span><span class="sxs-lookup"><span data-stu-id="4b675-121">Named clients</span></span>](#named-clients)
* [<span data-ttu-id="4b675-122">Client tipizzati</span><span class="sxs-lookup"><span data-stu-id="4b675-122">Typed clients</span></span>](#typed-clients)
* [<span data-ttu-id="4b675-123">Client generati</span><span class="sxs-lookup"><span data-stu-id="4b675-123">Generated clients</span></span>](#generated-clients)

<span data-ttu-id="4b675-124">L'approccio migliore dipende dai requisiti dell'app.</span><span class="sxs-lookup"><span data-stu-id="4b675-124">The best approach depends upon the app's requirements.</span></span>

### <a name="basic-usage"></a><span data-ttu-id="4b675-125">Uso di base</span><span class="sxs-lookup"><span data-stu-id="4b675-125">Basic usage</span></span>

<span data-ttu-id="4b675-126">`IHttpClientFactory` può essere registrato chiamando `AddHttpClient` :</span><span class="sxs-lookup"><span data-stu-id="4b675-126">`IHttpClientFactory` can be registered by calling `AddHttpClient`:</span></span>

[!code-csharp[](http-requests/samples/3.x/HttpClientFactorySample/Startup.cs?name=snippet1&highlight=13)]

<span data-ttu-id="4b675-127">`IHttpClientFactory`È possibile richiedere un oggetto usando l' [inserimento di dipendenze](xref:fundamentals/dependency-injection).</span><span class="sxs-lookup"><span data-stu-id="4b675-127">An `IHttpClientFactory` can be requested using [dependency injection (DI)](xref:fundamentals/dependency-injection).</span></span> <span data-ttu-id="4b675-128">Il codice seguente usa `IHttpClientFactory` per creare un' `HttpClient` istanza:</span><span class="sxs-lookup"><span data-stu-id="4b675-128">The following code uses `IHttpClientFactory` to create an `HttpClient` instance:</span></span>

[!code-csharp[](http-requests/samples/3.x/HttpClientFactorySample/Pages/BasicUsage.cshtml.cs?name=snippet1&highlight=9-12,21)]

<span data-ttu-id="4b675-129">L'uso di `IHttpClientFactory` like nell'esempio precedente è un modo efficace per effettuare il refactoring di un'app esistente.</span><span class="sxs-lookup"><span data-stu-id="4b675-129">Using `IHttpClientFactory` like in the preceding example is a good way to refactor an existing app.</span></span> <span data-ttu-id="4b675-130">Non ha alcun effetto sul modo in cui `HttpClient` viene usato.</span><span class="sxs-lookup"><span data-stu-id="4b675-130">It has no impact on how `HttpClient` is used.</span></span> <span data-ttu-id="4b675-131">Nelle posizioni in cui `HttpClient` vengono create le istanze in un'app esistente, sostituire tali occorrenze con le chiamate a <xref:System.Net.Http.IHttpClientFactory.CreateClient*> .</span><span class="sxs-lookup"><span data-stu-id="4b675-131">In places where `HttpClient` instances are created in an existing app, replace those occurrences with calls to <xref:System.Net.Http.IHttpClientFactory.CreateClient*>.</span></span>

### <a name="named-clients"></a><span data-ttu-id="4b675-132">Client denominati</span><span class="sxs-lookup"><span data-stu-id="4b675-132">Named clients</span></span>

<span data-ttu-id="4b675-133">I client denominati sono una scelta ottimale nei casi seguenti:</span><span class="sxs-lookup"><span data-stu-id="4b675-133">Named clients are a good choice when:</span></span>

* <span data-ttu-id="4b675-134">L'app richiede molti usi distinti di `HttpClient` .</span><span class="sxs-lookup"><span data-stu-id="4b675-134">The app requires many distinct uses of `HttpClient`.</span></span>
* <span data-ttu-id="4b675-135">Molti `HttpClient` hanno una configurazione diversa.</span><span class="sxs-lookup"><span data-stu-id="4b675-135">Many `HttpClient`s have different configuration.</span></span>

<span data-ttu-id="4b675-136">La configurazione di un oggetto denominato `HttpClient` può essere specificata durante la registrazione in `Startup.ConfigureServices` :</span><span class="sxs-lookup"><span data-stu-id="4b675-136">Configuration for a named `HttpClient` can be specified during registration in `Startup.ConfigureServices`:</span></span>

[!code-csharp[](http-requests/samples/3.x/HttpClientFactorySample/Startup.cs?name=snippet2)]

<span data-ttu-id="4b675-137">Nel codice precedente il client viene configurato con:</span><span class="sxs-lookup"><span data-stu-id="4b675-137">In the preceding code the client is configured with:</span></span>

* <span data-ttu-id="4b675-138">Indirizzo di base `https://api.github.com/` .</span><span class="sxs-lookup"><span data-stu-id="4b675-138">The base address `https://api.github.com/`.</span></span>
* <span data-ttu-id="4b675-139">Due intestazioni necessarie per lavorare con l'API GitHub.</span><span class="sxs-lookup"><span data-stu-id="4b675-139">Two headers required to work with the GitHub API.</span></span>

#### <a name="createclient"></a><span data-ttu-id="4b675-140">CreateClient</span><span class="sxs-lookup"><span data-stu-id="4b675-140">CreateClient</span></span>

<span data-ttu-id="4b675-141">Ogni volta che <xref:System.Net.Http.IHttpClientFactory.CreateClient*> viene chiamato:</span><span class="sxs-lookup"><span data-stu-id="4b675-141">Each time <xref:System.Net.Http.IHttpClientFactory.CreateClient*> is called:</span></span>

* <span data-ttu-id="4b675-142">Viene creata una nuova istanza di `HttpClient` .</span><span class="sxs-lookup"><span data-stu-id="4b675-142">A new instance of `HttpClient` is created.</span></span>
* <span data-ttu-id="4b675-143">Viene chiamata l'azione di configurazione.</span><span class="sxs-lookup"><span data-stu-id="4b675-143">The configuration action is called.</span></span>

<span data-ttu-id="4b675-144">Per creare un client denominato, passare il nome in `CreateClient` :</span><span class="sxs-lookup"><span data-stu-id="4b675-144">To create a named client, pass its name into `CreateClient`:</span></span>

[!code-csharp[](http-requests/samples/3.x/HttpClientFactorySample/Pages/NamedClient.cshtml.cs?name=snippet1&highlight=21)]

<span data-ttu-id="4b675-145">Nel codice precedente non è necessario che la richiesta specifichi un nome host.</span><span class="sxs-lookup"><span data-stu-id="4b675-145">In the preceding code, the request doesn't need to specify a hostname.</span></span> <span data-ttu-id="4b675-146">Il codice può passare solo il percorso, perché viene usato l'indirizzo di base configurato per il client.</span><span class="sxs-lookup"><span data-stu-id="4b675-146">The code can pass just the path, since the base address configured for the client is used.</span></span>

### <a name="typed-clients"></a><span data-ttu-id="4b675-147">Client tipizzati</span><span class="sxs-lookup"><span data-stu-id="4b675-147">Typed clients</span></span>

<span data-ttu-id="4b675-148">Client tipizzati:</span><span class="sxs-lookup"><span data-stu-id="4b675-148">Typed clients:</span></span>

* <span data-ttu-id="4b675-149">Offrono le stesse funzionalità dei client denominati senza la necessità di usare le stringhe come chiavi.</span><span class="sxs-lookup"><span data-stu-id="4b675-149">Provide the same capabilities as named clients without the need to use strings as keys.</span></span>
* <span data-ttu-id="4b675-150">Offrono l'aiuto di IntelliSense e del compilatore quando si usano i client.</span><span class="sxs-lookup"><span data-stu-id="4b675-150">Provides IntelliSense and compiler help when consuming clients.</span></span>
* <span data-ttu-id="4b675-151">La configurazione e l'interazione con un particolare `HttpClient` può avvenire in un'unica posizione.</span><span class="sxs-lookup"><span data-stu-id="4b675-151">Provide a single location to configure and interact with a particular `HttpClient`.</span></span> <span data-ttu-id="4b675-152">Ad esempio, è possibile usare un singolo client tipizzato:</span><span class="sxs-lookup"><span data-stu-id="4b675-152">For example, a single typed client might be used:</span></span>
  * <span data-ttu-id="4b675-153">Per un singolo endpoint back-end.</span><span class="sxs-lookup"><span data-stu-id="4b675-153">For a single backend endpoint.</span></span>
  * <span data-ttu-id="4b675-154">Per incapsulare tutta la logica che riguarda l'endpoint.</span><span class="sxs-lookup"><span data-stu-id="4b675-154">To encapsulate all logic dealing with the endpoint.</span></span>
* <span data-ttu-id="4b675-155">Usare DI e può essere inserito laddove necessario nell'app.</span><span class="sxs-lookup"><span data-stu-id="4b675-155">Work with DI and can be injected where required in the app.</span></span>

<span data-ttu-id="4b675-156">Un client tipizzato accetta un `HttpClient` parametro nel relativo costruttore:</span><span class="sxs-lookup"><span data-stu-id="4b675-156">A typed client accepts an `HttpClient` parameter in its constructor:</span></span>

[!code-csharp[](http-requests/samples/3.x/HttpClientFactorySample/GitHub/GitHubService.cs?name=snippet1&highlight=5)]
[!INCLUDE[about the series](~/includes/code-comments-loc.md)]

<span data-ttu-id="4b675-157">Nel codice precedente:</span><span class="sxs-lookup"><span data-stu-id="4b675-157">In the preceding code:</span></span>

* <span data-ttu-id="4b675-158">La configurazione viene spostata nel client tipizzato.</span><span class="sxs-lookup"><span data-stu-id="4b675-158">The configuration is moved into the typed client.</span></span>
* <span data-ttu-id="4b675-159">L'oggetto `HttpClient` viene esposto come una proprietà pubblica.</span><span class="sxs-lookup"><span data-stu-id="4b675-159">The `HttpClient` object is exposed as a public property.</span></span>

<span data-ttu-id="4b675-160">È possibile creare metodi specifici dell'API che espongono `HttpClient` funzionalità.</span><span class="sxs-lookup"><span data-stu-id="4b675-160">API-specific methods can be created that expose `HttpClient` functionality.</span></span> <span data-ttu-id="4b675-161">Ad esempio, il `GetAspNetDocsIssues` metodo incapsula il codice per recuperare i problemi aperti.</span><span class="sxs-lookup"><span data-stu-id="4b675-161">For example, the `GetAspNetDocsIssues` method encapsulates code to retrieve open issues.</span></span>

<span data-ttu-id="4b675-162">Il codice seguente chiama <xref:Microsoft.Extensions.DependencyInjection.HttpClientFactoryServiceCollectionExtensions.AddHttpClient*> in `Startup.ConfigureServices` per registrare una classe client tipizzata:</span><span class="sxs-lookup"><span data-stu-id="4b675-162">The following code calls <xref:Microsoft.Extensions.DependencyInjection.HttpClientFactoryServiceCollectionExtensions.AddHttpClient*> in `Startup.ConfigureServices` to register a typed client class:</span></span>

[!code-csharp[](http-requests/samples/3.x/HttpClientFactorySample/Startup.cs?name=snippet3)]

<span data-ttu-id="4b675-163">Il client tipizzato viene registrato come temporaneo nell'inserimento di dipendenze.</span><span class="sxs-lookup"><span data-stu-id="4b675-163">The typed client is registered as transient with DI.</span></span> <span data-ttu-id="4b675-164">Nel codice precedente, viene `AddHttpClient` registrato `GitHubService` come servizio temporaneo.</span><span class="sxs-lookup"><span data-stu-id="4b675-164">In the preceding code, `AddHttpClient` registers `GitHubService` as a transient service.</span></span> <span data-ttu-id="4b675-165">Questa registrazione usa un metodo factory per:</span><span class="sxs-lookup"><span data-stu-id="4b675-165">This registration uses a factory method to:</span></span>

1. <span data-ttu-id="4b675-166">Creare un'istanza di `HttpClient`.</span><span class="sxs-lookup"><span data-stu-id="4b675-166">Create an instance of `HttpClient`.</span></span>
1. <span data-ttu-id="4b675-167">Creare un'istanza di `GitHubService` passando l'istanza di `HttpClient` al relativo costruttore.</span><span class="sxs-lookup"><span data-stu-id="4b675-167">Create an instance of `GitHubService`, passing in the instance of `HttpClient` to its constructor.</span></span>

<span data-ttu-id="4b675-168">Il client tipizzato può essere inserito e usato direttamente:</span><span class="sxs-lookup"><span data-stu-id="4b675-168">The typed client can be injected and consumed directly:</span></span>

[!code-csharp[](http-requests/samples/3.x/HttpClientFactorySample/Pages/TypedClient.cshtml.cs?name=snippet1&highlight=11-14,20)]

<span data-ttu-id="4b675-169">La configurazione di un client tipizzato può essere specificata durante la registrazione in `Startup.ConfigureServices` , anziché nel costruttore del client tipizzato:</span><span class="sxs-lookup"><span data-stu-id="4b675-169">The configuration for a typed client can be specified during registration in `Startup.ConfigureServices`, rather than in the typed client's constructor:</span></span>

[!code-csharp[](http-requests/samples/3.x/HttpClientFactorySample/Startup.cs?name=snippet4)]

<span data-ttu-id="4b675-170">L'oggetto `HttpClient` può essere incapsulato all'interno di un client tipizzato.</span><span class="sxs-lookup"><span data-stu-id="4b675-170">The `HttpClient` can be encapsulated within a typed client.</span></span> <span data-ttu-id="4b675-171">Anziché esporla come proprietà, definire un metodo che chiama l' `HttpClient` istanza internamente:</span><span class="sxs-lookup"><span data-stu-id="4b675-171">Rather than exposing it as a property, define a method which calls the `HttpClient` instance internally:</span></span>

[!code-csharp[](http-requests/samples/3.x/HttpClientFactorySample/GitHub/RepoService.cs?name=snippet1&highlight=4)]

<span data-ttu-id="4b675-172">Nel codice precedente, l'oggetto `HttpClient` viene archiviato in un campo privato.</span><span class="sxs-lookup"><span data-stu-id="4b675-172">In the preceding code, the `HttpClient` is stored in a private field.</span></span> <span data-ttu-id="4b675-173">L'accesso a `HttpClient` è tramite il `GetRepos` Metodo pubblico.</span><span class="sxs-lookup"><span data-stu-id="4b675-173">Access to the `HttpClient` is by the public `GetRepos` method.</span></span>

### <a name="generated-clients"></a><span data-ttu-id="4b675-174">Client generati</span><span class="sxs-lookup"><span data-stu-id="4b675-174">Generated clients</span></span>

<span data-ttu-id="4b675-175">`IHttpClientFactory` può essere usato in combinazione con librerie di terze parti, ad esempio il [refitting](https://github.com/paulcbetts/refit).</span><span class="sxs-lookup"><span data-stu-id="4b675-175">`IHttpClientFactory` can be used in combination with third-party libraries such as [Refit](https://github.com/paulcbetts/refit).</span></span> <span data-ttu-id="4b675-176">Refit è una libreria REST per .NET.</span><span class="sxs-lookup"><span data-stu-id="4b675-176">Refit is a REST library for .NET.</span></span> <span data-ttu-id="4b675-177">Converte le API REST in interfacce live.</span><span class="sxs-lookup"><span data-stu-id="4b675-177">It converts REST APIs into live interfaces.</span></span> <span data-ttu-id="4b675-178">Un'implementazione dell'interfaccia viene generata dinamicamente da `RestService`, usando `HttpClient` per effettuare le chiamate HTTP esterne.</span><span class="sxs-lookup"><span data-stu-id="4b675-178">An implementation of the interface is generated dynamically by the `RestService`, using `HttpClient` to make the external HTTP calls.</span></span>

<span data-ttu-id="4b675-179">Per rappresentare l'API esterna e la relativa risposta vengono definite un'interfaccia e una risposta:</span><span class="sxs-lookup"><span data-stu-id="4b675-179">An interface and a reply are defined to represent the external API and its response:</span></span>

```csharp
public interface IHelloClient
{
    [Get("/helloworld")]
    Task<Reply> GetMessageAsync();
}

public class Reply
{
    public string Message { get; set; }
}
```

<span data-ttu-id="4b675-180">È possibile aggiungere un client tipizzato usando Refit per generare l'implementazione:</span><span class="sxs-lookup"><span data-stu-id="4b675-180">A typed client can be added, using Refit to generate the implementation:</span></span>

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddHttpClient("hello", c =>
    {
        c.BaseAddress = new Uri("http://localhost:5000");
    })
    .AddTypedClient(c => Refit.RestService.For<IHelloClient>(c));

    services.AddControllers();
}
```

<span data-ttu-id="4b675-181">L'interfaccia definita può essere usata dove necessario, con l'implementazione offerta dall'inserimento di dipendenze e da Refit:</span><span class="sxs-lookup"><span data-stu-id="4b675-181">The defined interface can be consumed where necessary, with the implementation provided by DI and Refit:</span></span>

```csharp
[ApiController]
public class ValuesController : ControllerBase
{
    private readonly IHelloClient _client;

    public ValuesController(IHelloClient client)
    {
        _client = client;
    }

    [HttpGet("/")]
    public async Task<ActionResult<Reply>> Index()
    {
        return await _client.GetMessageAsync();
    }
}
```

## <a name="make-post-put-and-delete-requests"></a><span data-ttu-id="4b675-182">Creare richieste POST, PUT e DELETE</span><span class="sxs-lookup"><span data-stu-id="4b675-182">Make POST, PUT, and DELETE requests</span></span>

<span data-ttu-id="4b675-183">Negli esempi precedenti, tutte le richieste HTTP usano il verbo GET HTTP.</span><span class="sxs-lookup"><span data-stu-id="4b675-183">In the preceding examples, all HTTP requests use the GET HTTP verb.</span></span> <span data-ttu-id="4b675-184">`HttpClient` supporta anche altri verbi HTTP, tra cui:</span><span class="sxs-lookup"><span data-stu-id="4b675-184">`HttpClient` also supports other HTTP verbs, including:</span></span>

* <span data-ttu-id="4b675-185">POST</span><span class="sxs-lookup"><span data-stu-id="4b675-185">POST</span></span>
* <span data-ttu-id="4b675-186">PUT</span><span class="sxs-lookup"><span data-stu-id="4b675-186">PUT</span></span>
* <span data-ttu-id="4b675-187">DELETE</span><span class="sxs-lookup"><span data-stu-id="4b675-187">DELETE</span></span>
* <span data-ttu-id="4b675-188">PATCH</span><span class="sxs-lookup"><span data-stu-id="4b675-188">PATCH</span></span>

<span data-ttu-id="4b675-189">Per un elenco completo dei verbi HTTP supportati, vedere <xref:System.Net.Http.HttpMethod> .</span><span class="sxs-lookup"><span data-stu-id="4b675-189">For a complete list of supported HTTP verbs, see <xref:System.Net.Http.HttpMethod>.</span></span>

<span data-ttu-id="4b675-190">Nell'esempio seguente viene illustrato come effettuare una richiesta HTTP POST:</span><span class="sxs-lookup"><span data-stu-id="4b675-190">The following example shows how to make an HTTP POST request:</span></span>

[!code-csharp[](http-requests/samples/3.x/HttpRequestsSample/Models/TodoClient.cs?name=snippet_POST)]

<span data-ttu-id="4b675-191">Nel codice precedente, il `CreateItemAsync` Metodo:</span><span class="sxs-lookup"><span data-stu-id="4b675-191">In the preceding code, the `CreateItemAsync` method:</span></span>

* <span data-ttu-id="4b675-192">Serializza il `TodoItem` parametro in JSON usando `System.Text.Json` .</span><span class="sxs-lookup"><span data-stu-id="4b675-192">Serializes the `TodoItem` parameter to JSON using `System.Text.Json`.</span></span> <span data-ttu-id="4b675-193">Viene utilizzata un'istanza di <xref:System.Text.Json.JsonSerializerOptions> per configurare il processo di serializzazione.</span><span class="sxs-lookup"><span data-stu-id="4b675-193">This uses an instance of <xref:System.Text.Json.JsonSerializerOptions> to configure the serialization process.</span></span>
* <span data-ttu-id="4b675-194">Crea un'istanza di <xref:System.Net.Http.StringContent> per comprimere il JSON serializzato per l'invio nel corpo della richiesta HTTP.</span><span class="sxs-lookup"><span data-stu-id="4b675-194">Creates an instance of <xref:System.Net.Http.StringContent> to package the serialized JSON for sending in the HTTP request's body.</span></span>
* <span data-ttu-id="4b675-195">Chiama <xref:System.Net.Http.HttpClient.PostAsync%2A> per inviare il contenuto JSON all'URL specificato.</span><span class="sxs-lookup"><span data-stu-id="4b675-195">Calls <xref:System.Net.Http.HttpClient.PostAsync%2A> to send the JSON content to the specified URL.</span></span> <span data-ttu-id="4b675-196">Si tratta di un URL relativo che viene aggiunto a [HttpClient. BaseAddress](xref:System.Net.Http.HttpClient.BaseAddress).</span><span class="sxs-lookup"><span data-stu-id="4b675-196">This is a relative URL that gets added to the [HttpClient.BaseAddress](xref:System.Net.Http.HttpClient.BaseAddress).</span></span>
* <span data-ttu-id="4b675-197">Chiama <xref:System.Net.Http.HttpResponseMessage.EnsureSuccessStatusCode%2A> per generare un'eccezione se il codice di stato della risposta non indica esito positivo.</span><span class="sxs-lookup"><span data-stu-id="4b675-197">Calls <xref:System.Net.Http.HttpResponseMessage.EnsureSuccessStatusCode%2A> to throw an exception if the response status code does not indicate success.</span></span>

<span data-ttu-id="4b675-198">`HttpClient` supporta anche altri tipi di contenuto.</span><span class="sxs-lookup"><span data-stu-id="4b675-198">`HttpClient` also supports other types of content.</span></span> <span data-ttu-id="4b675-199">Ad esempio, <xref:System.Net.Http.MultipartContent> e <xref:System.Net.Http.StreamContent>.</span><span class="sxs-lookup"><span data-stu-id="4b675-199">For example, <xref:System.Net.Http.MultipartContent> and <xref:System.Net.Http.StreamContent>.</span></span> <span data-ttu-id="4b675-200">Per un elenco completo dei contenuti supportati, vedere <xref:System.Net.Http.HttpContent> .</span><span class="sxs-lookup"><span data-stu-id="4b675-200">For a complete list of supported content, see <xref:System.Net.Http.HttpContent>.</span></span>

<span data-ttu-id="4b675-201">Nell'esempio seguente viene illustrata una richiesta HTTP PUT:</span><span class="sxs-lookup"><span data-stu-id="4b675-201">The following example shows an HTTP PUT request:</span></span>

[!code-csharp[](http-requests/samples/3.x/HttpRequestsSample/Models/TodoClient.cs?name=snippet_PUT)]

<span data-ttu-id="4b675-202">Il codice precedente è molto simile all'esempio di POST.</span><span class="sxs-lookup"><span data-stu-id="4b675-202">The preceding code is very similar to the POST example.</span></span> <span data-ttu-id="4b675-203">Il `SaveItemAsync` metodo chiama <xref:System.Net.Http.HttpClient.PutAsync%2A> anziché `PostAsync` .</span><span class="sxs-lookup"><span data-stu-id="4b675-203">The `SaveItemAsync` method calls <xref:System.Net.Http.HttpClient.PutAsync%2A> instead of `PostAsync`.</span></span>

<span data-ttu-id="4b675-204">Nell'esempio seguente viene illustrata una richiesta HTTP DELETE:</span><span class="sxs-lookup"><span data-stu-id="4b675-204">The following example shows an HTTP DELETE request:</span></span>

[!code-csharp[](http-requests/samples/3.x/HttpRequestsSample/Models/TodoClient.cs?name=snippet_DELETE)]

<span data-ttu-id="4b675-205">Nel codice precedente, il `DeleteItemAsync` metodo chiama <xref:System.Net.Http.HttpClient.DeleteAsync%2A> .</span><span class="sxs-lookup"><span data-stu-id="4b675-205">In the preceding code, the `DeleteItemAsync` method calls <xref:System.Net.Http.HttpClient.DeleteAsync%2A>.</span></span> <span data-ttu-id="4b675-206">Poiché le richieste DELETE HTTP non contengono in genere alcun corpo, il `DeleteAsync` metodo non fornisce un overload che accetta un'istanza di `HttpContent` .</span><span class="sxs-lookup"><span data-stu-id="4b675-206">Because HTTP DELETE requests typically contain no body, the `DeleteAsync` method doesn't provide an overload that accepts an instance of `HttpContent`.</span></span>

<span data-ttu-id="4b675-207">Per ulteriori informazioni sull'utilizzo di verbi HTTP diversi con `HttpClient` , vedere <xref:System.Net.Http.HttpClient> .</span><span class="sxs-lookup"><span data-stu-id="4b675-207">To learn more about using different HTTP verbs with `HttpClient`, see <xref:System.Net.Http.HttpClient>.</span></span>

## <a name="outgoing-request-middleware"></a><span data-ttu-id="4b675-208">Middleware per richieste in uscita</span><span class="sxs-lookup"><span data-stu-id="4b675-208">Outgoing request middleware</span></span>

<span data-ttu-id="4b675-209">`HttpClient` ha il concetto di delegare i gestori che possono essere collegati insieme per le richieste HTTP in uscita.</span><span class="sxs-lookup"><span data-stu-id="4b675-209">`HttpClient` has the concept of delegating handlers that can be linked together for outgoing HTTP requests.</span></span> <span data-ttu-id="4b675-210">`IHttpClientFactory`:</span><span class="sxs-lookup"><span data-stu-id="4b675-210">`IHttpClientFactory`:</span></span>

  * <span data-ttu-id="4b675-211">Semplifica la definizione dei gestori da applicare per ogni client denominato.</span><span class="sxs-lookup"><span data-stu-id="4b675-211">Simplifies defining the handlers to apply for each named client.</span></span>
  * <span data-ttu-id="4b675-212">Supporta la registrazione e il concatenamento di più gestori per compilare una pipeline middleware per le richieste in uscita.</span><span class="sxs-lookup"><span data-stu-id="4b675-212">Supports registration and chaining of multiple handlers to build an outgoing request   middleware pipeline.</span></span> <span data-ttu-id="4b675-213">Ognuno di questi gestori è in grado di eseguire operazioni prima e dopo la richiesta in uscita.</span><span class="sxs-lookup"><span data-stu-id="4b675-213">Each of these handlers is able to perform work before and after the   outgoing request.</span></span> <span data-ttu-id="4b675-214">Questo modello:</span><span class="sxs-lookup"><span data-stu-id="4b675-214">This pattern:</span></span>
  
    * <span data-ttu-id="4b675-215">È simile alla pipeline del middleware in ingresso in ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="4b675-215">Is similar to the inbound middleware pipeline in ASP.NET Core.</span></span>
    * <span data-ttu-id="4b675-216">Fornisce un meccanismo per gestire le problematiche trasversali relative alle richieste HTTP, ad esempio:</span><span class="sxs-lookup"><span data-stu-id="4b675-216">Provides a mechanism to manage cross-cutting concerns around HTTP requests, such as:</span></span>
      * <span data-ttu-id="4b675-217">caching</span><span class="sxs-lookup"><span data-stu-id="4b675-217">caching</span></span>
      * <span data-ttu-id="4b675-218">gestione degli errori</span><span class="sxs-lookup"><span data-stu-id="4b675-218">error handling</span></span>
      * <span data-ttu-id="4b675-219">serializzazione</span><span class="sxs-lookup"><span data-stu-id="4b675-219">serialization</span></span>
      * <span data-ttu-id="4b675-220">registrazione</span><span class="sxs-lookup"><span data-stu-id="4b675-220">logging</span></span>

<span data-ttu-id="4b675-221">Per creare un gestore di delega:</span><span class="sxs-lookup"><span data-stu-id="4b675-221">To create a delegating handler:</span></span>

* <span data-ttu-id="4b675-222">Derivare da <xref:System.Net.Http.DelegatingHandler> .</span><span class="sxs-lookup"><span data-stu-id="4b675-222">Derive from <xref:System.Net.Http.DelegatingHandler>.</span></span>
* <span data-ttu-id="4b675-223">Eseguire l'override di <xref:System.Net.Http.DelegatingHandler.SendAsync*>.</span><span class="sxs-lookup"><span data-stu-id="4b675-223">Override <xref:System.Net.Http.DelegatingHandler.SendAsync*>.</span></span> <span data-ttu-id="4b675-224">Eseguire il codice prima di passare la richiesta al gestore successivo nella pipeline:</span><span class="sxs-lookup"><span data-stu-id="4b675-224">Execute code before passing the request to the next handler in the pipeline:</span></span>

[!code-csharp[](http-requests/samples/3.x/HttpClientFactorySample/Handlers/ValidateHeaderHandler.cs?name=snippet1)]

<span data-ttu-id="4b675-225">Il codice precedente controlla se l' `X-API-KEY` intestazione è presente nella richiesta.</span><span class="sxs-lookup"><span data-stu-id="4b675-225">The preceding code checks if the `X-API-KEY` header is in the request.</span></span> <span data-ttu-id="4b675-226">Se `X-API-KEY` è mancante, <xref:System.Net.HttpStatusCode.BadRequest> viene restituito.</span><span class="sxs-lookup"><span data-stu-id="4b675-226">If `X-API-KEY` is missing, <xref:System.Net.HttpStatusCode.BadRequest> is returned.</span></span>

<span data-ttu-id="4b675-227">È possibile aggiungere più di un gestore alla configurazione per un oggetto `HttpClient` con <xref:Microsoft.Extensions.DependencyInjection.HttpClientBuilderExtensions.AddHttpMessageHandler*?displayProperty=fullName> :</span><span class="sxs-lookup"><span data-stu-id="4b675-227">More than one handler can be added to the configuration for an `HttpClient` with <xref:Microsoft.Extensions.DependencyInjection.HttpClientBuilderExtensions.AddHttpMessageHandler*?displayProperty=fullName>:</span></span>

[!code-csharp[](http-requests/samples/3.x/HttpClientFactorySample/Startup2.cs?name=snippet1)]

<span data-ttu-id="4b675-228">Nel codice precedente `ValidateHeaderHandler` viene registrato nell'inserimento di dipendenze.</span><span class="sxs-lookup"><span data-stu-id="4b675-228">In the preceding code, the `ValidateHeaderHandler` is registered with DI.</span></span> <span data-ttu-id="4b675-229">Dopo la registrazione è possibile chiamare <xref:Microsoft.Extensions.DependencyInjection.HttpClientBuilderExtensions.AddHttpMessageHandler*>, passando il tipo di gestore.</span><span class="sxs-lookup"><span data-stu-id="4b675-229">Once registered, <xref:Microsoft.Extensions.DependencyInjection.HttpClientBuilderExtensions.AddHttpMessageHandler*> can be called, passing in the type for the handler.</span></span>

<span data-ttu-id="4b675-230">È possibile registrare più gestori nell'ordine di esecuzione.</span><span class="sxs-lookup"><span data-stu-id="4b675-230">Multiple handlers can be registered in the order that they should execute.</span></span> <span data-ttu-id="4b675-231">Ogni gestore esegue il wrapping del gestore successivo fino a quando l'elemento `HttpClientHandler` finale non esegue la richiesta:</span><span class="sxs-lookup"><span data-stu-id="4b675-231">Each handler wraps the next handler until the final `HttpClientHandler` executes the request:</span></span>

[!code-csharp[](http-requests/samples/3.x/HttpClientFactorySample/Startup.cs?name=snippet6)]

### <a name="use-di-in-outgoing-request-middleware"></a><span data-ttu-id="4b675-232">Usa il middleware DI richiesta in uscita</span><span class="sxs-lookup"><span data-stu-id="4b675-232">Use DI in outgoing request middleware</span></span>

<span data-ttu-id="4b675-233">Quando `IHttpClientFactory` Crea un nuovo gestore di delega, USA di per soddisfare i parametri del costruttore del gestore.</span><span class="sxs-lookup"><span data-stu-id="4b675-233">When `IHttpClientFactory` creates a new delegating handler, it uses DI to fulfill the handler's constructor parameters.</span></span> <span data-ttu-id="4b675-234">`IHttpClientFactory` consente di creare un ambito **separato** per ogni gestore, che può causare un comportamento sorprendente quando un gestore utilizza un servizio con *ambito* .</span><span class="sxs-lookup"><span data-stu-id="4b675-234">`IHttpClientFactory` creates a **separate** DI scope for each handler, which can lead to surprising behavior when a handler consumes a *scoped* service.</span></span>

<span data-ttu-id="4b675-235">Si consideri, ad esempio, l'interfaccia seguente e la relativa implementazione, che rappresenta un'attività come operazione con un identificatore `OperationId` :</span><span class="sxs-lookup"><span data-stu-id="4b675-235">For example, consider the following interface and its implementation, which represents a task as an operation with an identifier, `OperationId`:</span></span>

[!code-csharp[](http-requests/samples/3.x/HttpRequestsSample/Models/OperationScoped.cs?name=snippet_Types)]

<span data-ttu-id="4b675-236">Come suggerisce il nome, `IOperationScoped` viene registrato con l'uso di una durata con *ambito* :</span><span class="sxs-lookup"><span data-stu-id="4b675-236">As its name suggests, `IOperationScoped` is registered with DI using a *scoped* lifetime:</span></span>

[!code-csharp[](http-requests/samples/3.x/HttpRequestsSample/Startup.cs?name=snippet_IOperationScoped&highlight=18,26)]

<span data-ttu-id="4b675-237">Il gestore delegato seguente utilizza e USA `IOperationScoped` per impostare l' `X-OPERATION-ID` intestazione per la richiesta in uscita:</span><span class="sxs-lookup"><span data-stu-id="4b675-237">The following delegating handler consumes and uses `IOperationScoped` to set the `X-OPERATION-ID` header for the outgoing request:</span></span>

[!code-csharp[](http-requests/samples/3.x/HttpRequestsSample/Handlers/OperationHandler.cs?name=snippet_Class&highlight=13)]

<span data-ttu-id="4b675-238">Nel [ `HttpRequestsSample` download](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/http-requests/samples/3.x/HttpRequestsSample)] passare a `/Operation` e aggiornare la pagina.</span><span class="sxs-lookup"><span data-stu-id="4b675-238">In the [`HttpRequestsSample` download](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/http-requests/samples/3.x/HttpRequestsSample)], navigate to `/Operation` and refresh the page.</span></span> <span data-ttu-id="4b675-239">Il valore dell'ambito della richiesta cambia per ogni richiesta, ma il valore dell'ambito del gestore cambia solo ogni 5 secondi.</span><span class="sxs-lookup"><span data-stu-id="4b675-239">The request scope value changes for each request, but the handler scope value only changes every 5 seconds.</span></span>

<span data-ttu-id="4b675-240">I gestori possono dipendere da servizi di qualsiasi ambito.</span><span class="sxs-lookup"><span data-stu-id="4b675-240">Handlers can depend upon services of any scope.</span></span> <span data-ttu-id="4b675-241">I servizi da cui dipendono i gestori vengono eliminati al momento dell'eliminazione del gestore.</span><span class="sxs-lookup"><span data-stu-id="4b675-241">Services that handlers depend upon are disposed when the handler is disposed.</span></span>

<span data-ttu-id="4b675-242">Usare uno degli approcci seguenti per condividere lo stato in base alla richiesta con i gestori di messaggi:</span><span class="sxs-lookup"><span data-stu-id="4b675-242">Use one of the following approaches to share per-request state with message handlers:</span></span>

* <span data-ttu-id="4b675-243">Passare i dati nel gestore usando [HttpRequestMessage. Properties](xref:System.Net.Http.HttpRequestMessage.Properties).</span><span class="sxs-lookup"><span data-stu-id="4b675-243">Pass data into the handler using [HttpRequestMessage.Properties](xref:System.Net.Http.HttpRequestMessage.Properties).</span></span>
* <span data-ttu-id="4b675-244">Usare <xref:Microsoft.AspNetCore.Http.IHttpContextAccessor> per accedere alla richiesta corrente.</span><span class="sxs-lookup"><span data-stu-id="4b675-244">Use <xref:Microsoft.AspNetCore.Http.IHttpContextAccessor> to access the current request.</span></span>
* <span data-ttu-id="4b675-245">Creare un oggetto di archiviazione <xref:System.Threading.AsyncLocal`1> personalizzato per passare i dati.</span><span class="sxs-lookup"><span data-stu-id="4b675-245">Create a custom <xref:System.Threading.AsyncLocal`1> storage object to pass the data.</span></span>

## <a name="use-polly-based-handlers"></a><span data-ttu-id="4b675-246">Usare gestori basati su Polly</span><span class="sxs-lookup"><span data-stu-id="4b675-246">Use Polly-based handlers</span></span>

<span data-ttu-id="4b675-247">`IHttpClientFactory` si integra con la libreria di terze parti [Polly](https://github.com/App-vNext/Polly).</span><span class="sxs-lookup"><span data-stu-id="4b675-247">`IHttpClientFactory` integrates with the third-party library [Polly](https://github.com/App-vNext/Polly).</span></span> <span data-ttu-id="4b675-248">Polly è una libreria di gestione degli errori temporanei e di resilienza completa per .NET.</span><span class="sxs-lookup"><span data-stu-id="4b675-248">Polly is a comprehensive resilience and transient fault-handling library for .NET.</span></span> <span data-ttu-id="4b675-249">Consente agli sviluppatori di esprimere criteri quali Retry, Circuit Breaker, Timeout, Bulkhead Isolation e Fallback in modo fluido e thread-safe.</span><span class="sxs-lookup"><span data-stu-id="4b675-249">It allows developers to express policies such as Retry, Circuit Breaker, Timeout, Bulkhead Isolation, and Fallback in a fluent and thread-safe manner.</span></span>

<span data-ttu-id="4b675-250">Per consentire l'uso dei criteri Polly con le istanze configurate di `HttpClient` sono disponibili metodi di estensione.</span><span class="sxs-lookup"><span data-stu-id="4b675-250">Extension methods are provided to enable the use of Polly policies with configured `HttpClient` instances.</span></span> <span data-ttu-id="4b675-251">Le estensioni di Polly supportano l'aggiunta di gestori basati su Polly ai client.</span><span class="sxs-lookup"><span data-stu-id="4b675-251">The Polly extensions support adding Polly-based handlers to clients.</span></span> <span data-ttu-id="4b675-252">Polly richiede il pacchetto NuGet [Microsoft. Extensions. http. Polly](https://www.nuget.org/packages/Microsoft.Extensions.Http.Polly/) .</span><span class="sxs-lookup"><span data-stu-id="4b675-252">Polly requires the [Microsoft.Extensions.Http.Polly](https://www.nuget.org/packages/Microsoft.Extensions.Http.Polly/) NuGet package.</span></span>

### <a name="handle-transient-faults"></a><span data-ttu-id="4b675-253">Gestire gli errori temporanei</span><span class="sxs-lookup"><span data-stu-id="4b675-253">Handle transient faults</span></span>

<span data-ttu-id="4b675-254">Gli errori si verificano in genere quando le chiamate HTTP esterne sono temporanee.</span><span class="sxs-lookup"><span data-stu-id="4b675-254">Faults typically occur when external HTTP calls are transient.</span></span> <span data-ttu-id="4b675-255"><xref:Microsoft.Extensions.DependencyInjection.PollyHttpClientBuilderExtensions.AddTransientHttpErrorPolicy*> consente di definire un criterio per gestire gli errori temporanei.</span><span class="sxs-lookup"><span data-stu-id="4b675-255"><xref:Microsoft.Extensions.DependencyInjection.PollyHttpClientBuilderExtensions.AddTransientHttpErrorPolicy*> allows a policy to be defined to handle transient errors.</span></span> <span data-ttu-id="4b675-256">I criteri configurati con `AddTransientHttpErrorPolicy` gestiscono le risposte seguenti:</span><span class="sxs-lookup"><span data-stu-id="4b675-256">Policies configured with `AddTransientHttpErrorPolicy` handle the following responses:</span></span>

* <xref:System.Net.Http.HttpRequestException>
* <span data-ttu-id="4b675-257">HTTP 5xx</span><span class="sxs-lookup"><span data-stu-id="4b675-257">HTTP 5xx</span></span>
* <span data-ttu-id="4b675-258">HTTP 408</span><span class="sxs-lookup"><span data-stu-id="4b675-258">HTTP 408</span></span>

<span data-ttu-id="4b675-259">`AddTransientHttpErrorPolicy` consente di accedere a un `PolicyBuilder` oggetto configurato per gestire gli errori che rappresentano un possibile errore temporaneo:</span><span class="sxs-lookup"><span data-stu-id="4b675-259">`AddTransientHttpErrorPolicy` provides access to a `PolicyBuilder` object configured to handle errors representing a possible transient fault:</span></span>

[!code-csharp[](http-requests/samples/3.x/HttpClientFactorySample/Startup3.cs?name=snippet1)]

<span data-ttu-id="4b675-260">Nel codice precedente viene definito un criterio `WaitAndRetryAsync`.</span><span class="sxs-lookup"><span data-stu-id="4b675-260">In the preceding code, a `WaitAndRetryAsync` policy is defined.</span></span> <span data-ttu-id="4b675-261">Le richieste non riuscite vengono ritentate fino a tre volte con un ritardo di 600 millisecondi tra i tentativi.</span><span class="sxs-lookup"><span data-stu-id="4b675-261">Failed requests are retried up to three times with a delay of 600 ms between attempts.</span></span>

### <a name="dynamically-select-policies"></a><span data-ttu-id="4b675-262">Selezionare i criteri in modo dinamico</span><span class="sxs-lookup"><span data-stu-id="4b675-262">Dynamically select policies</span></span>

<span data-ttu-id="4b675-263">I metodi di estensione vengono forniti per aggiungere gestori basati su Polly, ad esempio <xref:Microsoft.Extensions.DependencyInjection.PollyHttpClientBuilderExtensions.AddPolicyHandler*> .</span><span class="sxs-lookup"><span data-stu-id="4b675-263">Extension methods are provided to add Polly-based handlers, for example, <xref:Microsoft.Extensions.DependencyInjection.PollyHttpClientBuilderExtensions.AddPolicyHandler*>.</span></span> <span data-ttu-id="4b675-264">L' `AddPolicyHandler` Overload seguente controlla la richiesta per decidere quali criteri applicare:</span><span class="sxs-lookup"><span data-stu-id="4b675-264">The following `AddPolicyHandler` overload inspects the request to decide which policy to apply:</span></span>

[!code-csharp[](http-requests/samples/3.x/HttpClientFactorySample/Startup.cs?name=snippet8)]

<span data-ttu-id="4b675-265">Nel codice precedente, se la richiesta in uscita è una richiesta HTTP GET viene applicato un timeout di 10 secondi.</span><span class="sxs-lookup"><span data-stu-id="4b675-265">In the preceding code, if the outgoing request is an HTTP GET, a 10-second timeout is applied.</span></span> <span data-ttu-id="4b675-266">Per qualsiasi altro metodo HTTP viene usato un timeout di 30 secondi.</span><span class="sxs-lookup"><span data-stu-id="4b675-266">For any other HTTP method, a 30-second timeout is used.</span></span>

### <a name="add-multiple-polly-handlers"></a><span data-ttu-id="4b675-267">Aggiungere più gestori Polly</span><span class="sxs-lookup"><span data-stu-id="4b675-267">Add multiple Polly handlers</span></span>

<span data-ttu-id="4b675-268">È comune annidare i criteri Polly:</span><span class="sxs-lookup"><span data-stu-id="4b675-268">It's common to nest Polly policies:</span></span>

[!code-csharp[](http-requests/samples/3.x/HttpClientFactorySample/Startup.cs?name=snippet9)]

<span data-ttu-id="4b675-269">Nell'esempio precedente:</span><span class="sxs-lookup"><span data-stu-id="4b675-269">In the preceding example:</span></span>

* <span data-ttu-id="4b675-270">Vengono aggiunti due gestori.</span><span class="sxs-lookup"><span data-stu-id="4b675-270">Two handlers are added.</span></span>
* <span data-ttu-id="4b675-271">Il primo gestore USA <xref:Microsoft.Extensions.DependencyInjection.PollyHttpClientBuilderExtensions.AddTransientHttpErrorPolicy*> per aggiungere un criterio di ripetizione.</span><span class="sxs-lookup"><span data-stu-id="4b675-271">The first handler uses <xref:Microsoft.Extensions.DependencyInjection.PollyHttpClientBuilderExtensions.AddTransientHttpErrorPolicy*> to add a retry policy.</span></span> <span data-ttu-id="4b675-272">Le richieste non riuscite vengono ritentate fino a tre volte.</span><span class="sxs-lookup"><span data-stu-id="4b675-272">Failed requests are retried up to three times.</span></span>
* <span data-ttu-id="4b675-273">La seconda `AddTransientHttpErrorPolicy` chiamata aggiunge un criterio di interruttore.</span><span class="sxs-lookup"><span data-stu-id="4b675-273">The second `AddTransientHttpErrorPolicy` call adds a circuit breaker policy.</span></span> <span data-ttu-id="4b675-274">Altre richieste esterne vengono bloccate per 30 secondi se 5 tentativi non riusciti si verificano in sequenza.</span><span class="sxs-lookup"><span data-stu-id="4b675-274">Further external requests are blocked for 30 seconds if 5 failed attempts occur sequentially.</span></span> <span data-ttu-id="4b675-275">I criteri dell'interruttore di circuito sono con stato.</span><span class="sxs-lookup"><span data-stu-id="4b675-275">Circuit breaker policies are stateful.</span></span> <span data-ttu-id="4b675-276">Tutte le chiamate tramite questo client condividono lo stesso stato di circuito.</span><span class="sxs-lookup"><span data-stu-id="4b675-276">All calls through this client share the same circuit state.</span></span>

### <a name="add-policies-from-the-polly-registry"></a><span data-ttu-id="4b675-277">Aggiungere criteri dal registro Polly</span><span class="sxs-lookup"><span data-stu-id="4b675-277">Add policies from the Polly registry</span></span>

<span data-ttu-id="4b675-278">Un approccio alla gestione dei criteri usati di frequente consiste nel definirli una volta e registrarli in un elemento `PolicyRegistry`.</span><span class="sxs-lookup"><span data-stu-id="4b675-278">An approach to managing regularly used policies is to define them once and register them with a `PolicyRegistry`.</span></span>

<span data-ttu-id="4b675-279">Nel codice seguente:</span><span class="sxs-lookup"><span data-stu-id="4b675-279">In the following code:</span></span>

* <span data-ttu-id="4b675-280">Vengono aggiunti i criteri "regular" e "Long".</span><span class="sxs-lookup"><span data-stu-id="4b675-280">The "regular" and "long" policies are added.</span></span>
* <span data-ttu-id="4b675-281"><xref:Microsoft.Extensions.DependencyInjection.PollyHttpClientBuilderExtensions.AddPolicyHandlerFromRegistry*> aggiunge i criteri "regular" e "Long" dal registro di sistema.</span><span class="sxs-lookup"><span data-stu-id="4b675-281"><xref:Microsoft.Extensions.DependencyInjection.PollyHttpClientBuilderExtensions.AddPolicyHandlerFromRegistry*> adds the "regular" and "long" policies from the registry.</span></span>

[!code-csharp[](http-requests/samples/3.x/HttpClientFactorySample/Startup4.cs?name=snippet1)]

<span data-ttu-id="4b675-282">Per ulteriori informazioni su `IHttpClientFactory` e le integrazioni di Polly, vedere il [wiki di Polly](https://github.com/App-vNext/Polly/wiki/Polly-and-HttpClientFactory).</span><span class="sxs-lookup"><span data-stu-id="4b675-282">For more information on `IHttpClientFactory` and Polly integrations, see the [Polly wiki](https://github.com/App-vNext/Polly/wiki/Polly-and-HttpClientFactory).</span></span>

## <a name="httpclient-and-lifetime-management"></a><span data-ttu-id="4b675-283">Gestione di HttpClient e durata</span><span class="sxs-lookup"><span data-stu-id="4b675-283">HttpClient and lifetime management</span></span>

<span data-ttu-id="4b675-284">Viene restituita una nuova istanza di `HttpClient` per ogni chiamata di `CreateClient` per `IHttpClientFactory`.</span><span class="sxs-lookup"><span data-stu-id="4b675-284">A new `HttpClient` instance is returned each time `CreateClient` is called on the `IHttpClientFactory`.</span></span> <span data-ttu-id="4b675-285"><xref:System.Net.Http.HttpMessageHandler>Viene creato un oggetto per ogni client denominato.</span><span class="sxs-lookup"><span data-stu-id="4b675-285">An <xref:System.Net.Http.HttpMessageHandler> is created per named client.</span></span> <span data-ttu-id="4b675-286">La factory gestisce le durate delle istanze di `HttpMessageHandler`.</span><span class="sxs-lookup"><span data-stu-id="4b675-286">The factory manages the lifetimes of the `HttpMessageHandler` instances.</span></span>

<span data-ttu-id="4b675-287">`IHttpClientFactory` esegue il pooling delle istanze di `HttpMessageHandler` create dalla factory per ridurre il consumo di risorse.</span><span class="sxs-lookup"><span data-stu-id="4b675-287">`IHttpClientFactory` pools the `HttpMessageHandler` instances created by the factory to reduce resource consumption.</span></span> <span data-ttu-id="4b675-288">Un'istanza di `HttpMessageHandler` può essere riusata dal pool quando si crea una nuova istanza di `HttpClient` se la relativa durata non è scaduta.</span><span class="sxs-lookup"><span data-stu-id="4b675-288">An `HttpMessageHandler` instance may be reused from the pool when creating a new `HttpClient` instance if its lifetime hasn't expired.</span></span>

<span data-ttu-id="4b675-289">Il pooling dei gestori è consigliabile in quanto ogni gestore gestisce in genere le proprie connessioni HTTP sottostanti.</span><span class="sxs-lookup"><span data-stu-id="4b675-289">Pooling of handlers is desirable as each handler typically manages its own underlying HTTP connections.</span></span> <span data-ttu-id="4b675-290">La creazione di più gestori del necessario può causare ritardi di connessione.</span><span class="sxs-lookup"><span data-stu-id="4b675-290">Creating more handlers than necessary can result in connection delays.</span></span> <span data-ttu-id="4b675-291">Alcuni gestori mantengono inoltre le connessioni aperte a tempo indefinito, che possono impedire al gestore di reagire alle modifiche DNS (Domain Name System).</span><span class="sxs-lookup"><span data-stu-id="4b675-291">Some handlers also keep connections open indefinitely, which can prevent the handler from reacting to DNS (Domain Name System) changes.</span></span>

<span data-ttu-id="4b675-292">La durata del gestore predefinito è di due minuti.</span><span class="sxs-lookup"><span data-stu-id="4b675-292">The default handler lifetime is two minutes.</span></span> <span data-ttu-id="4b675-293">È possibile eseguire l'override del valore predefinito in base al client denominato:</span><span class="sxs-lookup"><span data-stu-id="4b675-293">The default value can be overridden on a per named client basis:</span></span>

[!code-csharp[](http-requests/samples/3.x/HttpClientFactorySample/Startup5.cs?name=snippet1)]

<span data-ttu-id="4b675-294">`HttpClient` le istanze possono in genere essere considerate come oggetti .NET che **non** richiedono l'eliminazione.</span><span class="sxs-lookup"><span data-stu-id="4b675-294">`HttpClient` instances can generally be treated as .NET objects **not** requiring disposal.</span></span> <span data-ttu-id="4b675-295">L'eliminazione annulla le richieste in uscita e garantisce che l'istanza di `HttpClient` specificata non possa essere usata dopo la chiamata a <xref:System.IDisposable.Dispose*>.</span><span class="sxs-lookup"><span data-stu-id="4b675-295">Disposal cancels outgoing requests and guarantees the given `HttpClient` instance can't be used after calling <xref:System.IDisposable.Dispose*>.</span></span> <span data-ttu-id="4b675-296">`IHttpClientFactory` tiene traccia ed elimina le risorse usate dalle istanze di `HttpClient`.</span><span class="sxs-lookup"><span data-stu-id="4b675-296">`IHttpClientFactory` tracks and disposes resources used by `HttpClient` instances.</span></span>

<span data-ttu-id="4b675-297">Mantenere una sola istanza di `HttpClient` attiva per un lungo periodo di tempo è un modello comune usato prima dell'avvento di `IHttpClientFactory`.</span><span class="sxs-lookup"><span data-stu-id="4b675-297">Keeping a single `HttpClient` instance alive for a long duration is a common pattern used before the inception of `IHttpClientFactory`.</span></span> <span data-ttu-id="4b675-298">Questo modello non è più necessario dopo la migrazione a `IHttpClientFactory`.</span><span class="sxs-lookup"><span data-stu-id="4b675-298">This pattern becomes unnecessary after migrating to `IHttpClientFactory`.</span></span>

### <a name="alternatives-to-ihttpclientfactory"></a><span data-ttu-id="4b675-299">Alternative a IHttpClientFactory</span><span class="sxs-lookup"><span data-stu-id="4b675-299">Alternatives to IHttpClientFactory</span></span>

<span data-ttu-id="4b675-300">`IHttpClientFactory`L'uso di in un'app abilitata per la funzionalità consente di evitare:</span><span class="sxs-lookup"><span data-stu-id="4b675-300">Using `IHttpClientFactory` in a DI-enabled app avoids:</span></span>

* <span data-ttu-id="4b675-301">Problemi di esaurimento delle risorse in base alle istanze di pool `HttpMessageHandler` .</span><span class="sxs-lookup"><span data-stu-id="4b675-301">Resource exhaustion problems by pooling `HttpMessageHandler` instances.</span></span>
* <span data-ttu-id="4b675-302">Problemi di DNS obsoleti tramite il ciclo di `HttpMessageHandler` istanze a intervalli regolari.</span><span class="sxs-lookup"><span data-stu-id="4b675-302">Stale DNS problems by cycling `HttpMessageHandler` instances at regular intervals.</span></span>

<span data-ttu-id="4b675-303">Esistono modi alternativi per risolvere i problemi precedenti utilizzando un'istanza di lunga durata <xref:System.Net.Http.SocketsHttpHandler> .</span><span class="sxs-lookup"><span data-stu-id="4b675-303">There are alternative ways to solve the preceding problems using a long-lived <xref:System.Net.Http.SocketsHttpHandler> instance.</span></span>

- <span data-ttu-id="4b675-304">Creare un'istanza di all' `SocketsHttpHandler` avvio dell'app e usarla per il ciclo di vita dell'app.</span><span class="sxs-lookup"><span data-stu-id="4b675-304">Create an instance of `SocketsHttpHandler` when the app starts and use it for the life of the app.</span></span>
- <span data-ttu-id="4b675-305">Configurare su <xref:System.Net.Http.SocketsHttpHandler.PooledConnectionLifetime> un valore appropriato in base alle ore di aggiornamento DNS.</span><span class="sxs-lookup"><span data-stu-id="4b675-305">Configure <xref:System.Net.Http.SocketsHttpHandler.PooledConnectionLifetime> to an appropriate value based on DNS refresh times.</span></span>
- <span data-ttu-id="4b675-306">Creare `HttpClient` istanze `new HttpClient(handler, disposeHandler: false)` di usando in base alle esigenze.</span><span class="sxs-lookup"><span data-stu-id="4b675-306">Create `HttpClient` instances using `new HttpClient(handler, disposeHandler: false)` as needed.</span></span>

<span data-ttu-id="4b675-307">Gli approcci precedenti risolvono i problemi di gestione delle risorse che vengono `IHttpClientFactory` risolti in modo analogo.</span><span class="sxs-lookup"><span data-stu-id="4b675-307">The preceding approaches solve the resource management problems that `IHttpClientFactory` solves in a similar way.</span></span>

- <span data-ttu-id="4b675-308">Il `SocketsHttpHandler` condivide le connessioni tra le `HttpClient` istanze.</span><span class="sxs-lookup"><span data-stu-id="4b675-308">The `SocketsHttpHandler` shares connections across `HttpClient` instances.</span></span> <span data-ttu-id="4b675-309">Questa condivisione impedisce l'esaurimento del socket.</span><span class="sxs-lookup"><span data-stu-id="4b675-309">This sharing prevents socket exhaustion.</span></span>
- <span data-ttu-id="4b675-310">Il `SocketsHttpHandler` ciclo di connessioni in base a `PooledConnectionLifetime` per evitare problemi DNS non aggiornati.</span><span class="sxs-lookup"><span data-stu-id="4b675-310">The `SocketsHttpHandler` cycles connections according to `PooledConnectionLifetime` to avoid stale DNS problems.</span></span>

### <a name="cookies"></a><span data-ttu-id="4b675-311">Cookies</span><span class="sxs-lookup"><span data-stu-id="4b675-311">Cookies</span></span>

<span data-ttu-id="4b675-312">Le `HttpMessageHandler` istanze in pool generano `CookieContainer` oggetti condivisi.</span><span class="sxs-lookup"><span data-stu-id="4b675-312">The pooled `HttpMessageHandler` instances results in `CookieContainer` objects being shared.</span></span> <span data-ttu-id="4b675-313">La `CookieContainer` condivisione di oggetti imprevista spesso genera codice errato.</span><span class="sxs-lookup"><span data-stu-id="4b675-313">Unanticipated `CookieContainer` object sharing often results in incorrect code.</span></span> <span data-ttu-id="4b675-314">Per le app che richiedono cookie s, prendere in considerazione quanto segue:</span><span class="sxs-lookup"><span data-stu-id="4b675-314">For apps that require cookies, consider either:</span></span>

 - <span data-ttu-id="4b675-315">Disabilitazione della cookie gestione automatica</span><span class="sxs-lookup"><span data-stu-id="4b675-315">Disabling automatic cookie handling</span></span>
 - <span data-ttu-id="4b675-316">Evitando `IHttpClientFactory`</span><span class="sxs-lookup"><span data-stu-id="4b675-316">Avoiding `IHttpClientFactory`</span></span>

<span data-ttu-id="4b675-317">Chiamare <xref:Microsoft.Extensions.DependencyInjection.HttpClientBuilderExtensions.ConfigurePrimaryHttpMessageHandler*> per disabilitare la cookie gestione automatica:</span><span class="sxs-lookup"><span data-stu-id="4b675-317">Call <xref:Microsoft.Extensions.DependencyInjection.HttpClientBuilderExtensions.ConfigurePrimaryHttpMessageHandler*> to disable automatic cookie handling:</span></span>

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet13)]

## <a name="logging"></a><span data-ttu-id="4b675-318">Registrazione</span><span class="sxs-lookup"><span data-stu-id="4b675-318">Logging</span></span>

<span data-ttu-id="4b675-319">I client creati tramite `IHttpClientFactory` registrano i messaggi di log per tutte le richieste.</span><span class="sxs-lookup"><span data-stu-id="4b675-319">Clients created via `IHttpClientFactory` record log messages for all requests.</span></span> <span data-ttu-id="4b675-320">Abilitare il livello di informazioni appropriato nella configurazione di registrazione per visualizzare i messaggi di log predefiniti.</span><span class="sxs-lookup"><span data-stu-id="4b675-320">Enable the appropriate information level in the logging configuration to see the default log messages.</span></span> <span data-ttu-id="4b675-321">La registrazione aggiuntiva, ad esempio quella delle intestazioni delle richieste, è inclusa solo a livello di traccia.</span><span class="sxs-lookup"><span data-stu-id="4b675-321">Additional logging, such as the logging of request headers, is only included at trace level.</span></span>

<span data-ttu-id="4b675-322">La categoria di log usata per ogni client include il nome del client.</span><span class="sxs-lookup"><span data-stu-id="4b675-322">The log category used for each client includes the name of the client.</span></span> <span data-ttu-id="4b675-323">Un client denominato *MyNamedClient*, ad esempio, registra i messaggi con una categoria "System .NET. http. HttpClient. **MyNamedClient**. LogicalHandler".</span><span class="sxs-lookup"><span data-stu-id="4b675-323">A client named *MyNamedClient*, for example, logs messages with a category of "System.Net.Http.HttpClient.**MyNamedClient**.LogicalHandler".</span></span> <span data-ttu-id="4b675-324">I messaggi con suffisso *LogicalHandler* sono esterni alla pipeline del gestore delle richieste.</span><span class="sxs-lookup"><span data-stu-id="4b675-324">Messages suffixed with *LogicalHandler* occur outside the request handler pipeline.</span></span> <span data-ttu-id="4b675-325">Nella richiesta i messaggi vengono registrati prima che qualsiasi altro gestore nella pipeline l'abbia elaborata.</span><span class="sxs-lookup"><span data-stu-id="4b675-325">On the request, messages are logged before any other handlers in the pipeline have processed it.</span></span> <span data-ttu-id="4b675-326">Nella risposta i messaggi vengono registrati dopo che qualsiasi altro gestore nella pipeline ha ricevuto la risposta.</span><span class="sxs-lookup"><span data-stu-id="4b675-326">On the response, messages are logged after any other pipeline handlers have received the response.</span></span>

<span data-ttu-id="4b675-327">La registrazione avviene anche all'interno della pipeline del gestore delle richieste.</span><span class="sxs-lookup"><span data-stu-id="4b675-327">Logging also occurs inside the request handler pipeline.</span></span> <span data-ttu-id="4b675-328">Nell'esempio *MyNamedClient* i messaggi vengono registrati con la categoria di log "System .NET. http. HttpClient. **MyNamedClient**. ClientHandler".</span><span class="sxs-lookup"><span data-stu-id="4b675-328">In the *MyNamedClient* example, those messages are logged with the log category "System.Net.Http.HttpClient.**MyNamedClient**.ClientHandler".</span></span> <span data-ttu-id="4b675-329">Per la richiesta, questo errore si verifica dopo l'esecuzione di tutti gli altri gestori e immediatamente prima dell'invio della richiesta.</span><span class="sxs-lookup"><span data-stu-id="4b675-329">For the request, this occurs after all other handlers have run and immediately before the request is sent.</span></span> <span data-ttu-id="4b675-330">Nella risposta la registrazione include lo stato della risposta prima di restituirla attraverso la pipeline del gestore.</span><span class="sxs-lookup"><span data-stu-id="4b675-330">On the response, this logging includes the state of the response before it passes back through the handler pipeline.</span></span>

<span data-ttu-id="4b675-331">L'abilitazione della registrazione all'esterno e all'interno della pipeline consente l'ispezione delle modifiche apportate da altri gestori nella pipeline.</span><span class="sxs-lookup"><span data-stu-id="4b675-331">Enabling logging outside and inside the pipeline enables inspection of the changes made by the other pipeline handlers.</span></span> <span data-ttu-id="4b675-332">Questo può includere modifiche alle intestazioni delle richieste o al codice di stato della risposta.</span><span class="sxs-lookup"><span data-stu-id="4b675-332">This may include changes to request headers or to the response status code.</span></span>

<span data-ttu-id="4b675-333">L'inclusione del nome del client nella categoria log consente il filtraggio dei log per specifici client denominati.</span><span class="sxs-lookup"><span data-stu-id="4b675-333">Including the name of the client in the log category enables log filtering for specific named clients.</span></span>

## <a name="configure-the-httpmessagehandler"></a><span data-ttu-id="4b675-334">Configurare HttpMessageHandler</span><span class="sxs-lookup"><span data-stu-id="4b675-334">Configure the HttpMessageHandler</span></span>

<span data-ttu-id="4b675-335">Può essere necessario controllare la configurazione dell'elemento `HttpMessageHandler` interno usato da un client.</span><span class="sxs-lookup"><span data-stu-id="4b675-335">It may be necessary to control the configuration of the inner `HttpMessageHandler` used by a client.</span></span>

<span data-ttu-id="4b675-336">Quando si aggiungono client denominati o tipizzati viene restituito un elemento `IHttpClientBuilder`.</span><span class="sxs-lookup"><span data-stu-id="4b675-336">An `IHttpClientBuilder` is returned when adding named or typed clients.</span></span> <span data-ttu-id="4b675-337">È possibile usare il metodo di estensione <xref:Microsoft.Extensions.DependencyInjection.HttpClientBuilderExtensions.ConfigurePrimaryHttpMessageHandler*> per definire un delegato.</span><span class="sxs-lookup"><span data-stu-id="4b675-337">The <xref:Microsoft.Extensions.DependencyInjection.HttpClientBuilderExtensions.ConfigurePrimaryHttpMessageHandler*> extension method can be used to define a delegate.</span></span> <span data-ttu-id="4b675-338">Il delegato viene usato per creare e configurare l'elemento `HttpMessageHandler` primario usato dal client:</span><span class="sxs-lookup"><span data-stu-id="4b675-338">The delegate is used to create and configure the primary `HttpMessageHandler` used by that client:</span></span>

[!code-csharp[](http-requests/samples/3.x/HttpClientFactorySample/Startup6.cs?name=snippet1)]

## <a name="use-ihttpclientfactory-in-a-console-app"></a><span data-ttu-id="4b675-339">Usare IHttpClientFactory in un'app console</span><span class="sxs-lookup"><span data-stu-id="4b675-339">Use IHttpClientFactory in a console app</span></span>

<span data-ttu-id="4b675-340">In un'app console aggiungere al progetto i riferimenti ai pacchetti seguenti:</span><span class="sxs-lookup"><span data-stu-id="4b675-340">In a console app, add the following package references to the project:</span></span>

* [<span data-ttu-id="4b675-341">Microsoft.Extensions.Hosting</span><span class="sxs-lookup"><span data-stu-id="4b675-341">Microsoft.Extensions.Hosting</span></span>](https://www.nuget.org/packages/Microsoft.Extensions.Hosting)
* [<span data-ttu-id="4b675-342">Microsoft.Extensions.Http</span><span class="sxs-lookup"><span data-stu-id="4b675-342">Microsoft.Extensions.Http</span></span>](https://www.nuget.org/packages/Microsoft.Extensions.Http)

<span data-ttu-id="4b675-343">Nell'esempio seguente:</span><span class="sxs-lookup"><span data-stu-id="4b675-343">In the following example:</span></span>

* <span data-ttu-id="4b675-344"><xref:System.Net.Http.IHttpClientFactory> è registrato nel contenitore di servizi dell'[host generico](xref:fundamentals/host/generic-host).</span><span class="sxs-lookup"><span data-stu-id="4b675-344"><xref:System.Net.Http.IHttpClientFactory> is registered in the [Generic Host's](xref:fundamentals/host/generic-host) service container.</span></span>
* <span data-ttu-id="4b675-345">`MyService` crea un'istanza della factory client dal servizio, che viene usata per creare una classe `HttpClient`.</span><span class="sxs-lookup"><span data-stu-id="4b675-345">`MyService` creates a client factory instance from the service, which is used to create an `HttpClient`.</span></span> <span data-ttu-id="4b675-346">`HttpClient` viene usato per recuperare una pagina Web.</span><span class="sxs-lookup"><span data-stu-id="4b675-346">`HttpClient` is used to retrieve a webpage.</span></span>
* <span data-ttu-id="4b675-347">`Main` crea un ambito per eseguire il metodo `GetPage` del servizio e scrivere i primi 500 caratteri del contenuto della pagina Web nella console.</span><span class="sxs-lookup"><span data-stu-id="4b675-347">`Main` creates a scope to execute the service's `GetPage` method and write the first 500 characters of the webpage content to the console.</span></span>

[!code-csharp[](http-requests/samples/3.x/HttpClientFactoryConsoleSample/Program.cs?highlight=14-15,20,26-27,59-62)]

## <a name="header-propagation-middleware"></a><span data-ttu-id="4b675-348">Middleware di propagazione delle intestazioni</span><span class="sxs-lookup"><span data-stu-id="4b675-348">Header propagation middleware</span></span>

<span data-ttu-id="4b675-349">La propagazione dell'intestazione è un middleware ASP.NET Core per propagare le intestazioni HTTP dalla richiesta in ingresso alle richieste del client HTTP in uscita.</span><span class="sxs-lookup"><span data-stu-id="4b675-349">Header propagation is an ASP.NET Core middleware to propagate HTTP headers from the incoming request to the outgoing HTTP Client requests.</span></span> <span data-ttu-id="4b675-350">Per usare la propagazione delle intestazioni:</span><span class="sxs-lookup"><span data-stu-id="4b675-350">To use header propagation:</span></span>

* <span data-ttu-id="4b675-351">Fare riferimento al pacchetto [Microsoft. AspNetCore. HeaderPropagation](https://www.nuget.org/packages/Microsoft.AspNetCore.HeaderPropagation) .</span><span class="sxs-lookup"><span data-stu-id="4b675-351">Reference the [Microsoft.AspNetCore.HeaderPropagation](https://www.nuget.org/packages/Microsoft.AspNetCore.HeaderPropagation) package.</span></span>
* <span data-ttu-id="4b675-352">Configurare il middleware e `HttpClient` in `Startup` :</span><span class="sxs-lookup"><span data-stu-id="4b675-352">Configure the middleware and `HttpClient` in `Startup`:</span></span>

  [!code-csharp[](http-requests/samples/3.x/Startup.cs?highlight=5-9,21&name=snippet)]

* <span data-ttu-id="4b675-353">Il client include le intestazioni configurate nelle richieste in uscita:</span><span class="sxs-lookup"><span data-stu-id="4b675-353">The client includes the configured headers on outbound requests:</span></span>

  ```csharp
  var client = clientFactory.CreateClient("MyForwardingClient");
  var response = client.GetAsync(...);
  ```

## <a name="additional-resources"></a><span data-ttu-id="4b675-354">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="4b675-354">Additional resources</span></span>

* [<span data-ttu-id="4b675-355">Usare HttpClientFactory per l'implementazione di richieste HTTP resilienti</span><span class="sxs-lookup"><span data-stu-id="4b675-355">Use HttpClientFactory to implement resilient HTTP requests</span></span>](/dotnet/standard/microservices-architecture/implement-resilient-applications/use-httpclientfactory-to-implement-resilient-http-requests)
* [<span data-ttu-id="4b675-356">Implementazione dei tentativi di chiamate HTTP con backoff esponenziale con i criteri di Polly e HttpClientFactory</span><span class="sxs-lookup"><span data-stu-id="4b675-356">Implement HTTP call retries with exponential backoff with HttpClientFactory and Polly policies</span></span>](/dotnet/standard/microservices-architecture/implement-resilient-applications/implement-http-call-retries-exponential-backoff-polly)
* [<span data-ttu-id="4b675-357">Implementazione dello schema Circuit Breaker</span><span class="sxs-lookup"><span data-stu-id="4b675-357">Implement the Circuit Breaker pattern</span></span>](/dotnet/standard/microservices-architecture/implement-resilient-applications/implement-circuit-breaker-pattern)
* [<span data-ttu-id="4b675-358">Come serializzare e deserializzare JSON in .NET</span><span class="sxs-lookup"><span data-stu-id="4b675-358">How to serialize and deserialize JSON in .NET</span></span>](/dotnet/standard/serialization/system-text-json-how-to)

::: moniker-end

::: moniker range="= aspnetcore-2.2"

<span data-ttu-id="4b675-359">Di [Glenn Condron](https://github.com/glennc), [Ryan Nowak](https://github.com/rynowak) e [Steve Gordon](https://github.com/stevejgordon)</span><span class="sxs-lookup"><span data-stu-id="4b675-359">By [Glenn Condron](https://github.com/glennc), [Ryan Nowak](https://github.com/rynowak), and [Steve Gordon](https://github.com/stevejgordon)</span></span>

<span data-ttu-id="4b675-360">È possibile registrare e usare un'interfaccia <xref:System.Net.Http.IHttpClientFactory> per configurare e creare istanze di <xref:System.Net.Http.HttpClient> in un'app.</span><span class="sxs-lookup"><span data-stu-id="4b675-360">An <xref:System.Net.Http.IHttpClientFactory> can be registered and used to configure and create <xref:System.Net.Http.HttpClient> instances in an app.</span></span> <span data-ttu-id="4b675-361">I vantaggi offerti sono i seguenti:</span><span class="sxs-lookup"><span data-stu-id="4b675-361">It offers the following benefits:</span></span>

* <span data-ttu-id="4b675-362">Offre una posizione centrale per la denominazione e la configurazione di istanze di `HttpClient` logiche.</span><span class="sxs-lookup"><span data-stu-id="4b675-362">Provides a central location for naming and configuring logical `HttpClient` instances.</span></span> <span data-ttu-id="4b675-363">Ad esempio, è possibile registrare e configurare un client *github* per accedere a [GitHub](https://github.com/).</span><span class="sxs-lookup"><span data-stu-id="4b675-363">For example, a *github* client can be registered and configured to access [GitHub](https://github.com/).</span></span> <span data-ttu-id="4b675-364">È possibile registrare un client predefinito per altri scopi.</span><span class="sxs-lookup"><span data-stu-id="4b675-364">A default client can be registered for other purposes.</span></span>
* <span data-ttu-id="4b675-365">Codifica il concetto di middleware in uscita tramite la delega di gestori in `HttpClient` e offre estensioni per il middleware basato su Polly per sfruttarne i vantaggi.</span><span class="sxs-lookup"><span data-stu-id="4b675-365">Codifies the concept of outgoing middleware via delegating handlers in `HttpClient` and provides extensions for Polly-based middleware to take advantage of that.</span></span>
* <span data-ttu-id="4b675-366">Gestisce il pooling e la durata delle istanze di `HttpClientMessageHandler` sottostanti per evitare problemi DNS comuni che si verificano quando le durate di `HttpClient` vengono gestite manualmente.</span><span class="sxs-lookup"><span data-stu-id="4b675-366">Manages the pooling and lifetime of underlying `HttpClientMessageHandler` instances to avoid common DNS problems that occur when manually managing `HttpClient` lifetimes.</span></span>
* <span data-ttu-id="4b675-367">Aggiunge un'esperienza di registrazione configurabile, tramite `ILogger`, per tutte le richieste inviate attraverso i client creati dalla factory.</span><span class="sxs-lookup"><span data-stu-id="4b675-367">Adds a configurable logging experience (via `ILogger`) for all requests sent through clients created by the factory.</span></span>

<span data-ttu-id="4b675-368">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/http-requests/samples) ([procedura per il download](xref:index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="4b675-368">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/http-requests/samples) ([how to download](xref:index#how-to-download-a-sample))</span></span>

## <a name="consumption-patterns"></a><span data-ttu-id="4b675-369">Modelli di consumo</span><span class="sxs-lookup"><span data-stu-id="4b675-369">Consumption patterns</span></span>

<span data-ttu-id="4b675-370">`IHttpClientFactory` può essere usato in un'app in diversi modi:</span><span class="sxs-lookup"><span data-stu-id="4b675-370">There are several ways `IHttpClientFactory` can be used in an app:</span></span>

* [<span data-ttu-id="4b675-371">Uso di base</span><span class="sxs-lookup"><span data-stu-id="4b675-371">Basic usage</span></span>](#basic-usage)
* [<span data-ttu-id="4b675-372">Client denominati</span><span class="sxs-lookup"><span data-stu-id="4b675-372">Named clients</span></span>](#named-clients)
* [<span data-ttu-id="4b675-373">Client tipizzati</span><span class="sxs-lookup"><span data-stu-id="4b675-373">Typed clients</span></span>](#typed-clients)
* [<span data-ttu-id="4b675-374">Client generati</span><span class="sxs-lookup"><span data-stu-id="4b675-374">Generated clients</span></span>](#generated-clients)

<span data-ttu-id="4b675-375">Nessuno di questi modi può essere considerato superiore a un altro.</span><span class="sxs-lookup"><span data-stu-id="4b675-375">None of them are strictly superior to another.</span></span> <span data-ttu-id="4b675-376">L'approccio migliore dipende dai vincoli dell'app.</span><span class="sxs-lookup"><span data-stu-id="4b675-376">The best approach depends upon the app's constraints.</span></span>

### <a name="basic-usage"></a><span data-ttu-id="4b675-377">Uso di base</span><span class="sxs-lookup"><span data-stu-id="4b675-377">Basic usage</span></span>

<span data-ttu-id="4b675-378">È possibile registrare `IHttpClientFactory` chiamando il metodo di estensione `AddHttpClient` in `IServiceCollection`, all'interno del metodo `Startup.ConfigureServices`.</span><span class="sxs-lookup"><span data-stu-id="4b675-378">The `IHttpClientFactory` can be registered by calling the `AddHttpClient` extension method on the `IServiceCollection`, inside the `Startup.ConfigureServices` method.</span></span>

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet1)]

<span data-ttu-id="4b675-379">Dopo la registrazione, il codice può accettare un'interfaccia `IHttpClientFactory` ovunque sia possibile inserire servizi con l'[inserimento di dipendenze](xref:fundamentals/dependency-injection).</span><span class="sxs-lookup"><span data-stu-id="4b675-379">Once registered, code can accept an `IHttpClientFactory` anywhere services can be injected with [dependency injection (DI)](xref:fundamentals/dependency-injection).</span></span> <span data-ttu-id="4b675-380">`IHttpClientFactory`È possibile utilizzare per creare un' `HttpClient` istanza di:</span><span class="sxs-lookup"><span data-stu-id="4b675-380">The `IHttpClientFactory` can be used to create an `HttpClient` instance:</span></span>

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Pages/BasicUsage.cshtml.cs?name=snippet1&highlight=9-12,21)]

<span data-ttu-id="4b675-381">Questo uso di `IHttpClientFactory` è un modo efficace per effettuare il refactoring di un'app esistente.</span><span class="sxs-lookup"><span data-stu-id="4b675-381">Using `IHttpClientFactory` in this fashion is a good way to refactor an existing app.</span></span> <span data-ttu-id="4b675-382">Non influisce in alcun modo sulle modalità d'uso di `HttpClient`.</span><span class="sxs-lookup"><span data-stu-id="4b675-382">It has no impact on the way `HttpClient` is used.</span></span> <span data-ttu-id="4b675-383">Nelle posizioni in cui vengono attualmente create istanze di `HttpClient`, sostituire le occorrenze con una chiamata a <xref:System.Net.Http.IHttpClientFactory.CreateClient*>.</span><span class="sxs-lookup"><span data-stu-id="4b675-383">In places where `HttpClient` instances are currently created, replace those occurrences with a call to <xref:System.Net.Http.IHttpClientFactory.CreateClient*>.</span></span>

### <a name="named-clients"></a><span data-ttu-id="4b675-384">Client denominati</span><span class="sxs-lookup"><span data-stu-id="4b675-384">Named clients</span></span>

<span data-ttu-id="4b675-385">Se un'app richiede molti usi distinti di `HttpClient`, ognuno con una configurazione diversa, un'opzione è l'uso di **client denominati**.</span><span class="sxs-lookup"><span data-stu-id="4b675-385">If an app requires many distinct uses of `HttpClient`, each with a different configuration, an option is to use **named clients**.</span></span> <span data-ttu-id="4b675-386">La configurazione di un `HttpClient` denominato può essere specificata durante la registrazione in `Startup.ConfigureServices`.</span><span class="sxs-lookup"><span data-stu-id="4b675-386">Configuration for a named `HttpClient` can be specified during registration in `Startup.ConfigureServices`.</span></span>

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet2)]

<span data-ttu-id="4b675-387">Nel codice precedente viene chiamato `AddHttpClient`, in cui viene specificato il nome *github*.</span><span class="sxs-lookup"><span data-stu-id="4b675-387">In the preceding code, `AddHttpClient` is called, providing the name *github*.</span></span> <span data-ttu-id="4b675-388">Al client viene applicata una configurazione predefinita, ovvero l'indirizzo di base e due intestazioni necessari per l'uso dell'API GitHub.</span><span class="sxs-lookup"><span data-stu-id="4b675-388">This client has some default configuration applied&mdash;namely the base address and two headers required to work with the GitHub API.</span></span>

<span data-ttu-id="4b675-389">Ogni volta che `CreateClient` viene chiamato, verrà creata una nuova istanza di `HttpClient` e verrà chiamata l'azione di configurazione.</span><span class="sxs-lookup"><span data-stu-id="4b675-389">Each time `CreateClient` is called, a new instance of `HttpClient` is created and the configuration action is called.</span></span>

<span data-ttu-id="4b675-390">Per usare un client denominato, è possibile passare un parametro di stringa a `CreateClient`.</span><span class="sxs-lookup"><span data-stu-id="4b675-390">To consume a named client, a string parameter can be passed to `CreateClient`.</span></span> <span data-ttu-id="4b675-391">Specificare il nome del client da creare:</span><span class="sxs-lookup"><span data-stu-id="4b675-391">Specify the name of the client to be created:</span></span>

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Pages/NamedClient.cshtml.cs?name=snippet1&highlight=21)]

<span data-ttu-id="4b675-392">Nel codice precedente non è necessario che la richiesta specifichi un nome host.</span><span class="sxs-lookup"><span data-stu-id="4b675-392">In the preceding code, the request doesn't need to specify a hostname.</span></span> <span data-ttu-id="4b675-393">Poiché viene usato l'indirizzo di base configurato per il client, è possibile passare solo il percorso.</span><span class="sxs-lookup"><span data-stu-id="4b675-393">It can pass just the path, since the base address configured for the client is used.</span></span>

### <a name="typed-clients"></a><span data-ttu-id="4b675-394">Client tipizzati</span><span class="sxs-lookup"><span data-stu-id="4b675-394">Typed clients</span></span>

<span data-ttu-id="4b675-395">Client tipizzati:</span><span class="sxs-lookup"><span data-stu-id="4b675-395">Typed clients:</span></span>

* <span data-ttu-id="4b675-396">Offrono le stesse funzionalità dei client denominati senza la necessità di usare le stringhe come chiavi.</span><span class="sxs-lookup"><span data-stu-id="4b675-396">Provide the same capabilities as named clients without the need to use strings as keys.</span></span>
* <span data-ttu-id="4b675-397">Offrono l'aiuto di IntelliSense e del compilatore quando si usano i client.</span><span class="sxs-lookup"><span data-stu-id="4b675-397">Provides IntelliSense and compiler help when consuming clients.</span></span>
* <span data-ttu-id="4b675-398">La configurazione e l'interazione con un particolare `HttpClient` può avvenire in un'unica posizione.</span><span class="sxs-lookup"><span data-stu-id="4b675-398">Provide a single location to configure and interact with a particular `HttpClient`.</span></span> <span data-ttu-id="4b675-399">Per un singolo endpoint back-end è ad esempio possibile usare un unico client tipizzato che incapsuli tutta la logica relativa all'endpoint.</span><span class="sxs-lookup"><span data-stu-id="4b675-399">For example, a single typed client might be used for a single backend endpoint and encapsulate all logic dealing with that endpoint.</span></span>
* <span data-ttu-id="4b675-400">Usano l'inserimento di dipendenze e possono essere inseriti dove necessario nell'app.</span><span class="sxs-lookup"><span data-stu-id="4b675-400">Work with DI and can be injected where required in your app.</span></span>

<span data-ttu-id="4b675-401">Un client tipizzato accetta un `HttpClient` parametro nel relativo costruttore:</span><span class="sxs-lookup"><span data-stu-id="4b675-401">A typed client accepts an `HttpClient` parameter in its constructor:</span></span>

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/GitHub/GitHubService.cs?name=snippet1&highlight=5)]

<span data-ttu-id="4b675-402">Nel codice precedente la configurazione viene spostata nel client tipizzato.</span><span class="sxs-lookup"><span data-stu-id="4b675-402">In the preceding code, the configuration is moved into the typed client.</span></span> <span data-ttu-id="4b675-403">L'oggetto `HttpClient` viene esposto come una proprietà pubblica.</span><span class="sxs-lookup"><span data-stu-id="4b675-403">The `HttpClient` object is exposed as a public property.</span></span> <span data-ttu-id="4b675-404">È possibile definire metodi di API specifiche che espongono la funzionalità `HttpClient`.</span><span class="sxs-lookup"><span data-stu-id="4b675-404">It's possible to define API-specific methods that expose `HttpClient` functionality.</span></span> <span data-ttu-id="4b675-405">Il metodo `GetAspNetDocsIssues` incapsula il codice necessario per eseguire una query e analizzare gli ultimi problemi aperti in un repository GitHub.</span><span class="sxs-lookup"><span data-stu-id="4b675-405">The `GetAspNetDocsIssues` method encapsulates the code needed to query for and parse out the latest open issues from a GitHub repository.</span></span>

<span data-ttu-id="4b675-406">Per registrare un client tipizzato è possibile usare il metodo di estensione <xref:Microsoft.Extensions.DependencyInjection.HttpClientFactoryServiceCollectionExtensions.AddHttpClient*> generico all'interno di `Startup.ConfigureServices`, specificando la classe del client tipizzato:</span><span class="sxs-lookup"><span data-stu-id="4b675-406">To register a typed client, the generic <xref:Microsoft.Extensions.DependencyInjection.HttpClientFactoryServiceCollectionExtensions.AddHttpClient*> extension method can be used within `Startup.ConfigureServices`, specifying the typed client class:</span></span>

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet3)]

<span data-ttu-id="4b675-407">Il client tipizzato viene registrato come temporaneo nell'inserimento di dipendenze.</span><span class="sxs-lookup"><span data-stu-id="4b675-407">The typed client is registered as transient with DI.</span></span> <span data-ttu-id="4b675-408">Il client tipizzato può essere inserito e usato direttamente:</span><span class="sxs-lookup"><span data-stu-id="4b675-408">The typed client can be injected and consumed directly:</span></span>

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Pages/TypedClient.cshtml.cs?name=snippet1&highlight=11-14,20)]

<span data-ttu-id="4b675-409">Se si preferisce, è possibile specificare la configurazione di un client tipizzato durante la registrazione in `Startup.ConfigureServices`, anziché nel relativo costruttore:</span><span class="sxs-lookup"><span data-stu-id="4b675-409">If preferred, the configuration for a typed client can be specified during registration in `Startup.ConfigureServices`, rather than in the typed client's constructor:</span></span>

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet4)]

<span data-ttu-id="4b675-410">È possibile incapsulare interamente `HttpClient` all'interno di un client tipizzato.</span><span class="sxs-lookup"><span data-stu-id="4b675-410">It's possible to entirely encapsulate the `HttpClient` within a typed client.</span></span> <span data-ttu-id="4b675-411">Anziché esporlo come una proprietà, è possibile specificare metodi pubblici che chiamano l'istanza di `HttpClient` internamente.</span><span class="sxs-lookup"><span data-stu-id="4b675-411">Rather than exposing it as a property, public methods can be provided which call the `HttpClient` instance internally.</span></span>

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/GitHub/RepoService.cs?name=snippet1&highlight=4)]

<span data-ttu-id="4b675-412">Nel codice precedente `HttpClient` viene archiviato come un campo privato.</span><span class="sxs-lookup"><span data-stu-id="4b675-412">In the preceding code, the `HttpClient` is stored as a private field.</span></span> <span data-ttu-id="4b675-413">L'accesso per effettuare chiamate esterne passa attraverso il metodo `GetRepos`.</span><span class="sxs-lookup"><span data-stu-id="4b675-413">All access to make external calls goes through the `GetRepos` method.</span></span>

### <a name="generated-clients"></a><span data-ttu-id="4b675-414">Client generati</span><span class="sxs-lookup"><span data-stu-id="4b675-414">Generated clients</span></span>

<span data-ttu-id="4b675-415">È possibile usare `IHttpClientFactory` in combinazione con altre librerie di terze parti, ad esempio [Refit](https://github.com/paulcbetts/refit).</span><span class="sxs-lookup"><span data-stu-id="4b675-415">`IHttpClientFactory` can be used in combination with other third-party libraries such as [Refit](https://github.com/paulcbetts/refit).</span></span> <span data-ttu-id="4b675-416">Refit è una libreria REST per .NET.</span><span class="sxs-lookup"><span data-stu-id="4b675-416">Refit is a REST library for .NET.</span></span> <span data-ttu-id="4b675-417">Converte le API REST in interfacce live.</span><span class="sxs-lookup"><span data-stu-id="4b675-417">It converts REST APIs into live interfaces.</span></span> <span data-ttu-id="4b675-418">Un'implementazione dell'interfaccia viene generata dinamicamente da `RestService`, usando `HttpClient` per effettuare le chiamate HTTP esterne.</span><span class="sxs-lookup"><span data-stu-id="4b675-418">An implementation of the interface is generated dynamically by the `RestService`, using `HttpClient` to make the external HTTP calls.</span></span>

<span data-ttu-id="4b675-419">Per rappresentare l'API esterna e la relativa risposta vengono definite un'interfaccia e una risposta:</span><span class="sxs-lookup"><span data-stu-id="4b675-419">An interface and a reply are defined to represent the external API and its response:</span></span>

```csharp
public interface IHelloClient
{
    [Get("/helloworld")]
    Task<Reply> GetMessageAsync();
}

public class Reply
{
    public string Message { get; set; }
}
```

<span data-ttu-id="4b675-420">È possibile aggiungere un client tipizzato usando Refit per generare l'implementazione:</span><span class="sxs-lookup"><span data-stu-id="4b675-420">A typed client can be added, using Refit to generate the implementation:</span></span>

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddHttpClient("hello", c =>
    {
        c.BaseAddress = new Uri("https://localhost:5001");
    })
    .AddTypedClient(c => Refit.RestService.For<IHelloClient>(c));

    services.AddMvc();
}
```

<span data-ttu-id="4b675-421">L'interfaccia definita può essere usata dove necessario, con l'implementazione offerta dall'inserimento di dipendenze e da Refit:</span><span class="sxs-lookup"><span data-stu-id="4b675-421">The defined interface can be consumed where necessary, with the implementation provided by DI and Refit:</span></span>

```csharp
[ApiController]
public class ValuesController : ControllerBase
{
    private readonly IHelloClient _client;

    public ValuesController(IHelloClient client)
    {
        _client = client;
    }

    [HttpGet("/")]
    public async Task<ActionResult<Reply>> Index()
    {
        return await _client.GetMessageAsync();
    }
}
```

## <a name="outgoing-request-middleware"></a><span data-ttu-id="4b675-422">Middleware per richieste in uscita</span><span class="sxs-lookup"><span data-stu-id="4b675-422">Outgoing request middleware</span></span>

<span data-ttu-id="4b675-423">`HttpClient` include già il concetto di delega di gestori concatenati per le richieste HTTP in uscita.</span><span class="sxs-lookup"><span data-stu-id="4b675-423">`HttpClient` already has the concept of delegating handlers that can be linked together for outgoing HTTP requests.</span></span> <span data-ttu-id="4b675-424">`IHttpClientFactory` semplifica la definizione dei gestori da applicare per ogni client denominato.</span><span class="sxs-lookup"><span data-stu-id="4b675-424">The `IHttpClientFactory` makes it easy to define the handlers to apply for each named client.</span></span> <span data-ttu-id="4b675-425">Supporta la registrazione e il concatenamento di più gestori per creare una pipeline di middleware per le richieste in uscita.</span><span class="sxs-lookup"><span data-stu-id="4b675-425">It supports registration and chaining of multiple handlers to build an outgoing request middleware pipeline.</span></span> <span data-ttu-id="4b675-426">Ognuno di questi gestori è in grado di eseguire operazioni prima e dopo la richiesta in uscita.</span><span class="sxs-lookup"><span data-stu-id="4b675-426">Each of these handlers is able to perform work before and after the outgoing request.</span></span> <span data-ttu-id="4b675-427">Questo modello è simile alla pipeline di middleware in ingresso in ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="4b675-427">This pattern is similar to the inbound middleware pipeline in ASP.NET Core.</span></span> <span data-ttu-id="4b675-428">Il modello offre un meccanismo per gestire le problematiche trasversali relative alle richieste HTTP, tra cui memorizzazione nella cache, gestione degli errori, serializzazione e registrazione.</span><span class="sxs-lookup"><span data-stu-id="4b675-428">The pattern provides a mechanism to manage cross-cutting concerns around HTTP requests, including caching, error handling, serialization, and logging.</span></span>

<span data-ttu-id="4b675-429">Per creare un gestore, definire una classe che deriva da <xref:System.Net.Http.DelegatingHandler>.</span><span class="sxs-lookup"><span data-stu-id="4b675-429">To create a handler, define a class deriving from <xref:System.Net.Http.DelegatingHandler>.</span></span> <span data-ttu-id="4b675-430">Eseguire l'override del metodo `SendAsync` per eseguire il codice prima di passare la richiesta al gestore successivo nella pipeline:</span><span class="sxs-lookup"><span data-stu-id="4b675-430">Override the `SendAsync` method to execute code before passing the request to the next handler in the pipeline:</span></span>

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Handlers/ValidateHeaderHandler.cs?name=snippet1)]

<span data-ttu-id="4b675-431">Il codice precedente definisce un gestore di base.</span><span class="sxs-lookup"><span data-stu-id="4b675-431">The preceding code defines a basic handler.</span></span> <span data-ttu-id="4b675-432">Verifica se un'intestazione `X-API-KEY` è stata inclusa nella richiesta.</span><span class="sxs-lookup"><span data-stu-id="4b675-432">It checks to see if an `X-API-KEY` header has been included on the request.</span></span> <span data-ttu-id="4b675-433">Se l'intestazione non è presente, può evitare la chiamata HTTP e restituire una risposta appropriata.</span><span class="sxs-lookup"><span data-stu-id="4b675-433">If the header is missing, it can avoid the HTTP call and return a suitable response.</span></span>

<span data-ttu-id="4b675-434">Durante la registrazione, è possibile aggiungere uno o più gestori alla configurazione per un `HttpClient` .</span><span class="sxs-lookup"><span data-stu-id="4b675-434">During registration, one or more handlers can be added to the configuration for an `HttpClient`.</span></span> <span data-ttu-id="4b675-435">Questa attività viene eseguita tramite metodi di estensione in <xref:Microsoft.Extensions.DependencyInjection.IHttpClientBuilder>.</span><span class="sxs-lookup"><span data-stu-id="4b675-435">This task is accomplished via extension methods on the <xref:Microsoft.Extensions.DependencyInjection.IHttpClientBuilder>.</span></span>

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet5)]

<span data-ttu-id="4b675-436">Nel codice precedente `ValidateHeaderHandler` viene registrato nell'inserimento di dipendenze.</span><span class="sxs-lookup"><span data-stu-id="4b675-436">In the preceding code, the `ValidateHeaderHandler` is registered with DI.</span></span> <span data-ttu-id="4b675-437">L'interfaccia `IHttpClientFactory` crea un ambito di inserimento delle dipendenze separato per ogni gestore.</span><span class="sxs-lookup"><span data-stu-id="4b675-437">The `IHttpClientFactory` creates a separate DI scope for each handler.</span></span> <span data-ttu-id="4b675-438">I gestori possono dipendere da servizi di qualsiasi ambito.</span><span class="sxs-lookup"><span data-stu-id="4b675-438">Handlers are free to depend upon services of any scope.</span></span> <span data-ttu-id="4b675-439">I servizi da cui dipendono i gestori vengono eliminati al momento dell'eliminazione del gestore.</span><span class="sxs-lookup"><span data-stu-id="4b675-439">Services that handlers depend upon are disposed when the handler is disposed.</span></span>

<span data-ttu-id="4b675-440">Dopo la registrazione è possibile chiamare <xref:Microsoft.Extensions.DependencyInjection.HttpClientBuilderExtensions.AddHttpMessageHandler*>, passando il tipo di gestore.</span><span class="sxs-lookup"><span data-stu-id="4b675-440">Once registered, <xref:Microsoft.Extensions.DependencyInjection.HttpClientBuilderExtensions.AddHttpMessageHandler*> can be called, passing in the type for the handler.</span></span>

<span data-ttu-id="4b675-441">È possibile registrare più gestori nell'ordine di esecuzione.</span><span class="sxs-lookup"><span data-stu-id="4b675-441">Multiple handlers can be registered in the order that they should execute.</span></span> <span data-ttu-id="4b675-442">Ogni gestore esegue il wrapping del gestore successivo fino a quando l'elemento `HttpClientHandler` finale non esegue la richiesta:</span><span class="sxs-lookup"><span data-stu-id="4b675-442">Each handler wraps the next handler until the final `HttpClientHandler` executes the request:</span></span>

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet6)]

<span data-ttu-id="4b675-443">Usare uno degli approcci seguenti per condividere lo stato in base alla richiesta con i gestori di messaggi:</span><span class="sxs-lookup"><span data-stu-id="4b675-443">Use one of the following approaches to share per-request state with message handlers:</span></span>

* <span data-ttu-id="4b675-444">Passare i dati al gestore usando `HttpRequestMessage.Properties`.</span><span class="sxs-lookup"><span data-stu-id="4b675-444">Pass data into the handler using `HttpRequestMessage.Properties`.</span></span>
* <span data-ttu-id="4b675-445">Usare `IHttpContextAccessor` per accedere alla richiesta corrente.</span><span class="sxs-lookup"><span data-stu-id="4b675-445">Use `IHttpContextAccessor` to access the current request.</span></span>
* <span data-ttu-id="4b675-446">Creare un oggetto di archiviazione `AsyncLocal` personalizzato per passare i dati.</span><span class="sxs-lookup"><span data-stu-id="4b675-446">Create a custom `AsyncLocal` storage object to pass the data.</span></span>

## <a name="use-polly-based-handlers"></a><span data-ttu-id="4b675-447">Usare gestori basati su Polly</span><span class="sxs-lookup"><span data-stu-id="4b675-447">Use Polly-based handlers</span></span>

<span data-ttu-id="4b675-448">`IHttpClientFactory` si integra con una libreria di terze parti piuttosto diffusa denominata [Polly](https://github.com/App-vNext/Polly).</span><span class="sxs-lookup"><span data-stu-id="4b675-448">`IHttpClientFactory` integrates with a popular third-party library called [Polly](https://github.com/App-vNext/Polly).</span></span> <span data-ttu-id="4b675-449">Polly è una libreria di gestione degli errori temporanei e di resilienza completa per .NET.</span><span class="sxs-lookup"><span data-stu-id="4b675-449">Polly is a comprehensive resilience and transient fault-handling library for .NET.</span></span> <span data-ttu-id="4b675-450">Consente agli sviluppatori di esprimere criteri quali Retry, Circuit Breaker, Timeout, Bulkhead Isolation e Fallback in modo fluido e thread-safe.</span><span class="sxs-lookup"><span data-stu-id="4b675-450">It allows developers to express policies such as Retry, Circuit Breaker, Timeout, Bulkhead Isolation, and Fallback in a fluent and thread-safe manner.</span></span>

<span data-ttu-id="4b675-451">Per consentire l'uso dei criteri Polly con le istanze configurate di `HttpClient` sono disponibili metodi di estensione.</span><span class="sxs-lookup"><span data-stu-id="4b675-451">Extension methods are provided to enable the use of Polly policies with configured `HttpClient` instances.</span></span> <span data-ttu-id="4b675-452">Le estensioni Polly:</span><span class="sxs-lookup"><span data-stu-id="4b675-452">The Polly extensions:</span></span>

* <span data-ttu-id="4b675-453">Supportano l'aggiunta di gestori basati su Polly ai client.</span><span class="sxs-lookup"><span data-stu-id="4b675-453">Support adding Polly-based handlers to clients.</span></span>
* <span data-ttu-id="4b675-454">Possono essere usate dopo l'installazione del pacchetto NuGet [Microsoft.Extensions.Http.Polly](https://www.nuget.org/packages/Microsoft.Extensions.Http.Polly/).</span><span class="sxs-lookup"><span data-stu-id="4b675-454">Can be used after installing the [Microsoft.Extensions.Http.Polly](https://www.nuget.org/packages/Microsoft.Extensions.Http.Polly/) NuGet package.</span></span> <span data-ttu-id="4b675-455">Il pacchetto non è incluso nel framework condiviso ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="4b675-455">The package isn't included in the ASP.NET Core shared framework.</span></span>

### <a name="handle-transient-faults"></a><span data-ttu-id="4b675-456">Gestire gli errori temporanei</span><span class="sxs-lookup"><span data-stu-id="4b675-456">Handle transient faults</span></span>

<span data-ttu-id="4b675-457">La maggior parte degli errori comuni si verifica quando le chiamate HTTP esterne sono temporanee.</span><span class="sxs-lookup"><span data-stu-id="4b675-457">Most common faults occur when external HTTP calls are transient.</span></span> <span data-ttu-id="4b675-458">Per definire un criterio in grado di gestire gli errori temporanei è disponibile un pratico metodo di estensione denominato `AddTransientHttpErrorPolicy`.</span><span class="sxs-lookup"><span data-stu-id="4b675-458">A convenient extension method called `AddTransientHttpErrorPolicy` is included which allows a policy to be defined to handle transient errors.</span></span> <span data-ttu-id="4b675-459">I criteri configurati con questo metodo di estensione gestiscono `HttpRequestException`, risposte HTTP 5xx e risposte HTTP 408.</span><span class="sxs-lookup"><span data-stu-id="4b675-459">Policies configured with this extension method handle `HttpRequestException`, HTTP 5xx responses, and HTTP 408 responses.</span></span>

<span data-ttu-id="4b675-460">L'estensione `AddTransientHttpErrorPolicy` può essere usata all'interno di `Startup.ConfigureServices`.</span><span class="sxs-lookup"><span data-stu-id="4b675-460">The `AddTransientHttpErrorPolicy` extension can be used within `Startup.ConfigureServices`.</span></span> <span data-ttu-id="4b675-461">L'estensione consente l'accesso a un oggetto `PolicyBuilder` configurato per gestire gli errori che rappresentano un possibile errore temporaneo:</span><span class="sxs-lookup"><span data-stu-id="4b675-461">The extension provides access to a `PolicyBuilder` object configured to handle errors representing a possible transient fault:</span></span>

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet7)]

<span data-ttu-id="4b675-462">Nel codice precedente viene definito un criterio `WaitAndRetryAsync`.</span><span class="sxs-lookup"><span data-stu-id="4b675-462">In the preceding code, a `WaitAndRetryAsync` policy is defined.</span></span> <span data-ttu-id="4b675-463">Le richieste non riuscite vengono ritentate fino a tre volte con un ritardo di 600 millisecondi tra i tentativi.</span><span class="sxs-lookup"><span data-stu-id="4b675-463">Failed requests are retried up to three times with a delay of 600 ms between attempts.</span></span>

### <a name="dynamically-select-policies"></a><span data-ttu-id="4b675-464">Selezionare i criteri in modo dinamico</span><span class="sxs-lookup"><span data-stu-id="4b675-464">Dynamically select policies</span></span>

<span data-ttu-id="4b675-465">Per aggiungere gestori basati su Polly è possibile usare altri metodi di estensione.</span><span class="sxs-lookup"><span data-stu-id="4b675-465">Additional extension methods exist which can be used to add Polly-based handlers.</span></span> <span data-ttu-id="4b675-466">Una di queste estensioni è `AddPolicyHandler`, che include più overload.</span><span class="sxs-lookup"><span data-stu-id="4b675-466">One such extension is `AddPolicyHandler`, which has multiple overloads.</span></span> <span data-ttu-id="4b675-467">Un overload consente l'ispezione della richiesta al momento della definizione del criterio da applicare:</span><span class="sxs-lookup"><span data-stu-id="4b675-467">One overload allows the request to be inspected when defining which policy to apply:</span></span>

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet8)]

<span data-ttu-id="4b675-468">Nel codice precedente, se la richiesta in uscita è una richiesta HTTP GET viene applicato un timeout di 10 secondi.</span><span class="sxs-lookup"><span data-stu-id="4b675-468">In the preceding code, if the outgoing request is an HTTP GET, a 10-second timeout is applied.</span></span> <span data-ttu-id="4b675-469">Per qualsiasi altro metodo HTTP viene usato un timeout di 30 secondi.</span><span class="sxs-lookup"><span data-stu-id="4b675-469">For any other HTTP method, a 30-second timeout is used.</span></span>

### <a name="add-multiple-polly-handlers"></a><span data-ttu-id="4b675-470">Aggiungere più gestori Polly</span><span class="sxs-lookup"><span data-stu-id="4b675-470">Add multiple Polly handlers</span></span>

<span data-ttu-id="4b675-471">In molti casi i criteri Polly vengono annidati per offrire funzionalità avanzate:</span><span class="sxs-lookup"><span data-stu-id="4b675-471">It's common to nest Polly policies to provide enhanced functionality:</span></span>

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet9)]

<span data-ttu-id="4b675-472">Nell'esempio precedente vengono aggiunti due gestori.</span><span class="sxs-lookup"><span data-stu-id="4b675-472">In the preceding example, two handlers are added.</span></span> <span data-ttu-id="4b675-473">Il primo usa l'estensione `AddTransientHttpErrorPolicy` per aggiungere criteri di ripetizione.</span><span class="sxs-lookup"><span data-stu-id="4b675-473">The first uses the `AddTransientHttpErrorPolicy` extension to add a retry policy.</span></span> <span data-ttu-id="4b675-474">Le richieste non riuscite vengono ritentate fino a tre volte.</span><span class="sxs-lookup"><span data-stu-id="4b675-474">Failed requests are retried up to three times.</span></span> <span data-ttu-id="4b675-475">La seconda chiamata a `AddTransientHttpErrorPolicy` aggiunge criteri dell'interruttore di circuito.</span><span class="sxs-lookup"><span data-stu-id="4b675-475">The second call to `AddTransientHttpErrorPolicy` adds a circuit breaker policy.</span></span> <span data-ttu-id="4b675-476">Ulteriori richieste esterne vengono bloccate per 30 secondi nel caso si verifichino cinque tentativi non riusciti consecutivi.</span><span class="sxs-lookup"><span data-stu-id="4b675-476">Further external requests are blocked for 30 seconds if five failed attempts occur sequentially.</span></span> <span data-ttu-id="4b675-477">I criteri dell'interruttore di circuito sono con stato.</span><span class="sxs-lookup"><span data-stu-id="4b675-477">Circuit breaker policies are stateful.</span></span> <span data-ttu-id="4b675-478">Tutte le chiamate tramite questo client condividono lo stesso stato di circuito.</span><span class="sxs-lookup"><span data-stu-id="4b675-478">All calls through this client share the same circuit state.</span></span>

### <a name="add-policies-from-the-polly-registry"></a><span data-ttu-id="4b675-479">Aggiungere criteri dal registro Polly</span><span class="sxs-lookup"><span data-stu-id="4b675-479">Add policies from the Polly registry</span></span>

<span data-ttu-id="4b675-480">Un approccio alla gestione dei criteri usati di frequente consiste nel definirli una volta e registrarli in un elemento `PolicyRegistry`.</span><span class="sxs-lookup"><span data-stu-id="4b675-480">An approach to managing regularly used policies is to define them once and register them with a `PolicyRegistry`.</span></span> <span data-ttu-id="4b675-481">Per aggiungere un gestore usando un criterio del registro è disponibile un metodo di estensione:</span><span class="sxs-lookup"><span data-stu-id="4b675-481">An extension method is provided which allows a handler to be added using a policy from the registry:</span></span>

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet10)]

<span data-ttu-id="4b675-482">Nel codice precedente vengono registrati due criteri quando si aggiunge `PolicyRegistry` a `ServiceCollection`.</span><span class="sxs-lookup"><span data-stu-id="4b675-482">In the preceding code, two policies are registered when the `PolicyRegistry` is added to the `ServiceCollection`.</span></span> <span data-ttu-id="4b675-483">Per usare un criterio dal registro viene usato il metodo `AddPolicyHandlerFromRegistry` passando il nome del criterio da applicare.</span><span class="sxs-lookup"><span data-stu-id="4b675-483">To use a policy from the registry, the `AddPolicyHandlerFromRegistry` method is used, passing the name of the policy to apply.</span></span>

<span data-ttu-id="4b675-484">Altre informazioni su `IHttpClientFactory` e le integrazioni con Polly sono disponibili nel [wiki di Polly](https://github.com/App-vNext/Polly/wiki/Polly-and-HttpClientFactory).</span><span class="sxs-lookup"><span data-stu-id="4b675-484">Further information about `IHttpClientFactory` and Polly integrations can be found on the [Polly wiki](https://github.com/App-vNext/Polly/wiki/Polly-and-HttpClientFactory).</span></span>

## <a name="httpclient-and-lifetime-management"></a><span data-ttu-id="4b675-485">Gestione di HttpClient e durata</span><span class="sxs-lookup"><span data-stu-id="4b675-485">HttpClient and lifetime management</span></span>

<span data-ttu-id="4b675-486">Viene restituita una nuova istanza di `HttpClient` per ogni chiamata di `CreateClient` per `IHttpClientFactory`.</span><span class="sxs-lookup"><span data-stu-id="4b675-486">A new `HttpClient` instance is returned each time `CreateClient` is called on the `IHttpClientFactory`.</span></span> <span data-ttu-id="4b675-487">È presente un <xref:System.Net.Http.HttpMessageHandler> per ogni client denominato.</span><span class="sxs-lookup"><span data-stu-id="4b675-487">There's an <xref:System.Net.Http.HttpMessageHandler> per named client.</span></span> <span data-ttu-id="4b675-488">La factory gestisce le durate delle istanze di `HttpMessageHandler`.</span><span class="sxs-lookup"><span data-stu-id="4b675-488">The factory manages the lifetimes of the `HttpMessageHandler` instances.</span></span>

<span data-ttu-id="4b675-489">`IHttpClientFactory` esegue il pooling delle istanze di `HttpMessageHandler` create dalla factory per ridurre il consumo di risorse.</span><span class="sxs-lookup"><span data-stu-id="4b675-489">`IHttpClientFactory` pools the `HttpMessageHandler` instances created by the factory to reduce resource consumption.</span></span> <span data-ttu-id="4b675-490">Un'istanza di `HttpMessageHandler` può essere riusata dal pool quando si crea una nuova istanza di `HttpClient` se la relativa durata non è scaduta.</span><span class="sxs-lookup"><span data-stu-id="4b675-490">An `HttpMessageHandler` instance may be reused from the pool when creating a new `HttpClient` instance if its lifetime hasn't expired.</span></span>

<span data-ttu-id="4b675-491">Il pooling dei gestori è consigliabile in quanto ogni gestore gestisce in genere le proprie connessioni HTTP sottostanti.</span><span class="sxs-lookup"><span data-stu-id="4b675-491">Pooling of handlers is desirable as each handler typically manages its own underlying HTTP connections.</span></span> <span data-ttu-id="4b675-492">La creazione di più gestori del necessario può causare ritardi di connessione.</span><span class="sxs-lookup"><span data-stu-id="4b675-492">Creating more handlers than necessary can result in connection delays.</span></span> <span data-ttu-id="4b675-493">Alcuni gestori mantengono inoltre le connessioni aperte a tempo indefinito. Ciò può impedire al gestore di reagire alle modifiche DNS.</span><span class="sxs-lookup"><span data-stu-id="4b675-493">Some handlers also keep connections open indefinitely, which can prevent the handler from reacting to DNS changes.</span></span>

<span data-ttu-id="4b675-494">La durata del gestore predefinito è di due minuti.</span><span class="sxs-lookup"><span data-stu-id="4b675-494">The default handler lifetime is two minutes.</span></span> <span data-ttu-id="4b675-495">Il valore predefinito può essere sottoposto a override per ogni client denominato.</span><span class="sxs-lookup"><span data-stu-id="4b675-495">The default value can be overridden on a per named client basis.</span></span> <span data-ttu-id="4b675-496">Per eseguire l'override, chiamare <xref:Microsoft.Extensions.DependencyInjection.HttpClientBuilderExtensions.SetHandlerLifetime*> nell'elemento `IHttpClientBuilder` restituito al momento della creazione del client:</span><span class="sxs-lookup"><span data-stu-id="4b675-496">To override it, call <xref:Microsoft.Extensions.DependencyInjection.HttpClientBuilderExtensions.SetHandlerLifetime*> on the `IHttpClientBuilder` that is returned when creating the client:</span></span>

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet11)]

<span data-ttu-id="4b675-497">L'eliminazione del client non è obbligatoria.</span><span class="sxs-lookup"><span data-stu-id="4b675-497">Disposal of the client isn't required.</span></span> <span data-ttu-id="4b675-498">L'eliminazione annulla le richieste in uscita e garantisce che l'istanza di `HttpClient` specificata non possa essere usata dopo la chiamata a <xref:System.IDisposable.Dispose*>.</span><span class="sxs-lookup"><span data-stu-id="4b675-498">Disposal cancels outgoing requests and guarantees the given `HttpClient` instance can't be used after calling <xref:System.IDisposable.Dispose*>.</span></span> <span data-ttu-id="4b675-499">`IHttpClientFactory` tiene traccia ed elimina le risorse usate dalle istanze di `HttpClient`.</span><span class="sxs-lookup"><span data-stu-id="4b675-499">`IHttpClientFactory` tracks and disposes resources used by `HttpClient` instances.</span></span> <span data-ttu-id="4b675-500">Le istanze di `HttpClient` possono essere considerate a livello generale come oggetti .NET che non richiedono l'eliminazione.</span><span class="sxs-lookup"><span data-stu-id="4b675-500">The `HttpClient` instances can generally be treated as .NET objects not requiring disposal.</span></span>

<span data-ttu-id="4b675-501">Mantenere una sola istanza di `HttpClient` attiva per un lungo periodo di tempo è un modello comune usato prima dell'avvento di `IHttpClientFactory`.</span><span class="sxs-lookup"><span data-stu-id="4b675-501">Keeping a single `HttpClient` instance alive for a long duration is a common pattern used before the inception of `IHttpClientFactory`.</span></span> <span data-ttu-id="4b675-502">Questo modello non è più necessario dopo la migrazione a `IHttpClientFactory`.</span><span class="sxs-lookup"><span data-stu-id="4b675-502">This pattern becomes unnecessary after migrating to `IHttpClientFactory`.</span></span>

### <a name="alternatives-to-ihttpclientfactory"></a><span data-ttu-id="4b675-503">Alternative a IHttpClientFactory</span><span class="sxs-lookup"><span data-stu-id="4b675-503">Alternatives to IHttpClientFactory</span></span>

<span data-ttu-id="4b675-504">`IHttpClientFactory`L'uso di in un'app abilitata per la funzionalità consente di evitare:</span><span class="sxs-lookup"><span data-stu-id="4b675-504">Using `IHttpClientFactory` in a DI-enabled app avoids:</span></span>

* <span data-ttu-id="4b675-505">Problemi di esaurimento delle risorse in base alle istanze di pool `HttpMessageHandler` .</span><span class="sxs-lookup"><span data-stu-id="4b675-505">Resource exhaustion problems by pooling `HttpMessageHandler` instances.</span></span>
* <span data-ttu-id="4b675-506">Problemi di DNS obsoleti tramite il ciclo di `HttpMessageHandler` istanze a intervalli regolari.</span><span class="sxs-lookup"><span data-stu-id="4b675-506">Stale DNS problems by cycling `HttpMessageHandler` instances at regular intervals.</span></span>

<span data-ttu-id="4b675-507">Esistono modi alternativi per risolvere i problemi precedenti utilizzando un'istanza di lunga durata <xref:System.Net.Http.SocketsHttpHandler> .</span><span class="sxs-lookup"><span data-stu-id="4b675-507">There are alternative ways to solve the preceding problems using a long-lived <xref:System.Net.Http.SocketsHttpHandler> instance.</span></span>

- <span data-ttu-id="4b675-508">Creare un'istanza di all' `SocketsHttpHandler` avvio dell'app e usarla per il ciclo di vita dell'app.</span><span class="sxs-lookup"><span data-stu-id="4b675-508">Create an instance of `SocketsHttpHandler` when the app starts and use it for the life of the app.</span></span>
- <span data-ttu-id="4b675-509">Configurare su <xref:System.Net.Http.SocketsHttpHandler.PooledConnectionLifetime> un valore appropriato in base alle ore di aggiornamento DNS.</span><span class="sxs-lookup"><span data-stu-id="4b675-509">Configure <xref:System.Net.Http.SocketsHttpHandler.PooledConnectionLifetime> to an appropriate value based on DNS refresh times.</span></span>
- <span data-ttu-id="4b675-510">Creare `HttpClient` istanze `new HttpClient(handler, disposeHandler: false)` di usando in base alle esigenze.</span><span class="sxs-lookup"><span data-stu-id="4b675-510">Create `HttpClient` instances using `new HttpClient(handler, disposeHandler: false)` as needed.</span></span>

<span data-ttu-id="4b675-511">Gli approcci precedenti risolvono i problemi di gestione delle risorse che vengono `IHttpClientFactory` risolti in modo analogo.</span><span class="sxs-lookup"><span data-stu-id="4b675-511">The preceding approaches solve the resource management problems that `IHttpClientFactory` solves in a similar way.</span></span>

- <span data-ttu-id="4b675-512">Il `SocketsHttpHandler` condivide le connessioni tra le `HttpClient` istanze.</span><span class="sxs-lookup"><span data-stu-id="4b675-512">The `SocketsHttpHandler` shares connections across `HttpClient` instances.</span></span> <span data-ttu-id="4b675-513">Questa condivisione impedisce l'esaurimento del socket.</span><span class="sxs-lookup"><span data-stu-id="4b675-513">This sharing prevents socket exhaustion.</span></span>
- <span data-ttu-id="4b675-514">Il `SocketsHttpHandler` ciclo di connessioni in base a `PooledConnectionLifetime` per evitare problemi DNS non aggiornati.</span><span class="sxs-lookup"><span data-stu-id="4b675-514">The `SocketsHttpHandler` cycles connections according to `PooledConnectionLifetime` to avoid stale DNS problems.</span></span>

### <a name="cookies"></a><span data-ttu-id="4b675-515">Cookies</span><span class="sxs-lookup"><span data-stu-id="4b675-515">Cookies</span></span>

<span data-ttu-id="4b675-516">Le `HttpMessageHandler` istanze in pool generano `CookieContainer` oggetti condivisi.</span><span class="sxs-lookup"><span data-stu-id="4b675-516">The pooled `HttpMessageHandler` instances results in `CookieContainer` objects being shared.</span></span> <span data-ttu-id="4b675-517">La `CookieContainer` condivisione di oggetti imprevista spesso genera codice errato.</span><span class="sxs-lookup"><span data-stu-id="4b675-517">Unanticipated `CookieContainer` object sharing often results in incorrect code.</span></span> <span data-ttu-id="4b675-518">Per le app che richiedono cookie s, prendere in considerazione quanto segue:</span><span class="sxs-lookup"><span data-stu-id="4b675-518">For apps that require cookies, consider either:</span></span>

 - <span data-ttu-id="4b675-519">Disabilitazione della cookie gestione automatica</span><span class="sxs-lookup"><span data-stu-id="4b675-519">Disabling automatic cookie handling</span></span>
 - <span data-ttu-id="4b675-520">Evitando `IHttpClientFactory`</span><span class="sxs-lookup"><span data-stu-id="4b675-520">Avoiding `IHttpClientFactory`</span></span>

<span data-ttu-id="4b675-521">Chiamare <xref:Microsoft.Extensions.DependencyInjection.HttpClientBuilderExtensions.ConfigurePrimaryHttpMessageHandler*> per disabilitare la cookie gestione automatica:</span><span class="sxs-lookup"><span data-stu-id="4b675-521">Call <xref:Microsoft.Extensions.DependencyInjection.HttpClientBuilderExtensions.ConfigurePrimaryHttpMessageHandler*> to disable automatic cookie handling:</span></span>

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet13)]

## <a name="logging"></a><span data-ttu-id="4b675-522">Registrazione</span><span class="sxs-lookup"><span data-stu-id="4b675-522">Logging</span></span>

<span data-ttu-id="4b675-523">I client creati tramite `IHttpClientFactory` registrano i messaggi di log per tutte le richieste.</span><span class="sxs-lookup"><span data-stu-id="4b675-523">Clients created via `IHttpClientFactory` record log messages for all requests.</span></span> <span data-ttu-id="4b675-524">Per visualizzare i messaggi di log predefiniti, abilitare il livello di informazioni appropriato nella configurazione di registrazione.</span><span class="sxs-lookup"><span data-stu-id="4b675-524">Enable the appropriate information level in your logging configuration to see the default log messages.</span></span> <span data-ttu-id="4b675-525">La registrazione aggiuntiva, ad esempio quella delle intestazioni delle richieste, è inclusa solo a livello di traccia.</span><span class="sxs-lookup"><span data-stu-id="4b675-525">Additional logging, such as the logging of request headers, is only included at trace level.</span></span>

<span data-ttu-id="4b675-526">La categoria di log usata per ogni client include il nome del client.</span><span class="sxs-lookup"><span data-stu-id="4b675-526">The log category used for each client includes the name of the client.</span></span> <span data-ttu-id="4b675-527">Un client denominato *MyNamedClient*, ad esempio, registra i messaggi con una categoria `System.Net.Http.HttpClient.MyNamedClient.LogicalHandler`.</span><span class="sxs-lookup"><span data-stu-id="4b675-527">A client named *MyNamedClient*, for example, logs messages with a category of `System.Net.Http.HttpClient.MyNamedClient.LogicalHandler`.</span></span> <span data-ttu-id="4b675-528">I messaggi con suffisso *LogicalHandler* sono esterni alla pipeline del gestore delle richieste.</span><span class="sxs-lookup"><span data-stu-id="4b675-528">Messages suffixed with *LogicalHandler* occur outside the request handler pipeline.</span></span> <span data-ttu-id="4b675-529">Nella richiesta i messaggi vengono registrati prima che qualsiasi altro gestore nella pipeline l'abbia elaborata.</span><span class="sxs-lookup"><span data-stu-id="4b675-529">On the request, messages are logged before any other handlers in the pipeline have processed it.</span></span> <span data-ttu-id="4b675-530">Nella risposta i messaggi vengono registrati dopo che qualsiasi altro gestore nella pipeline ha ricevuto la risposta.</span><span class="sxs-lookup"><span data-stu-id="4b675-530">On the response, messages are logged after any other pipeline handlers have received the response.</span></span>

<span data-ttu-id="4b675-531">La registrazione avviene anche all'interno della pipeline del gestore delle richieste.</span><span class="sxs-lookup"><span data-stu-id="4b675-531">Logging also occurs inside the request handler pipeline.</span></span> <span data-ttu-id="4b675-532">Nell'esempio *MyNamedClient*, i messaggi vengono registrati nella categoria di log `System.Net.Http.HttpClient.MyNamedClient.ClientHandler`.</span><span class="sxs-lookup"><span data-stu-id="4b675-532">In the *MyNamedClient* example, those messages are logged against the log category `System.Net.Http.HttpClient.MyNamedClient.ClientHandler`.</span></span> <span data-ttu-id="4b675-533">Per la richiesta, ciò avviene dopo l'esecuzione di tutti gli altri gestori e immediatamente prima che la richiesta sia inviata in rete.</span><span class="sxs-lookup"><span data-stu-id="4b675-533">For the request, this occurs after all other handlers have run and immediately before the request is sent out on the network.</span></span> <span data-ttu-id="4b675-534">Nella risposta la registrazione include lo stato della risposta prima di restituirla attraverso la pipeline del gestore.</span><span class="sxs-lookup"><span data-stu-id="4b675-534">On the response, this logging includes the state of the response before it passes back through the handler pipeline.</span></span>

<span data-ttu-id="4b675-535">L'abilitazione della registrazione all'esterno e all'interno della pipeline consente l'ispezione delle modifiche apportate da altri gestori nella pipeline.</span><span class="sxs-lookup"><span data-stu-id="4b675-535">Enabling logging outside and inside the pipeline enables inspection of the changes made by the other pipeline handlers.</span></span> <span data-ttu-id="4b675-536">Le modifiche possono ad esempio riguardare le intestazioni delle richieste o il codice di stato della risposta.</span><span class="sxs-lookup"><span data-stu-id="4b675-536">This may include changes to request headers, for example, or to the response status code.</span></span>

<span data-ttu-id="4b675-537">L'inclusione del nome del client nella categoria di log consente di filtrare i log in base a client denominati specifici, se necessario.</span><span class="sxs-lookup"><span data-stu-id="4b675-537">Including the name of the client in the log category enables log filtering for specific named clients where necessary.</span></span>

## <a name="configure-the-httpmessagehandler"></a><span data-ttu-id="4b675-538">Configurare HttpMessageHandler</span><span class="sxs-lookup"><span data-stu-id="4b675-538">Configure the HttpMessageHandler</span></span>

<span data-ttu-id="4b675-539">Può essere necessario controllare la configurazione dell'elemento `HttpMessageHandler` interno usato da un client.</span><span class="sxs-lookup"><span data-stu-id="4b675-539">It may be necessary to control the configuration of the inner `HttpMessageHandler` used by a client.</span></span>

<span data-ttu-id="4b675-540">Quando si aggiungono client denominati o tipizzati viene restituito un elemento `IHttpClientBuilder`.</span><span class="sxs-lookup"><span data-stu-id="4b675-540">An `IHttpClientBuilder` is returned when adding named or typed clients.</span></span> <span data-ttu-id="4b675-541">È possibile usare il metodo di estensione <xref:Microsoft.Extensions.DependencyInjection.HttpClientBuilderExtensions.ConfigurePrimaryHttpMessageHandler*> per definire un delegato.</span><span class="sxs-lookup"><span data-stu-id="4b675-541">The <xref:Microsoft.Extensions.DependencyInjection.HttpClientBuilderExtensions.ConfigurePrimaryHttpMessageHandler*> extension method can be used to define a delegate.</span></span> <span data-ttu-id="4b675-542">Il delegato viene usato per creare e configurare l'elemento `HttpMessageHandler` primario usato dal client:</span><span class="sxs-lookup"><span data-stu-id="4b675-542">The delegate is used to create and configure the primary `HttpMessageHandler` used by that client:</span></span>

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet12)]

## <a name="use-ihttpclientfactory-in-a-console-app"></a><span data-ttu-id="4b675-543">Usare IHttpClientFactory in un'app console</span><span class="sxs-lookup"><span data-stu-id="4b675-543">Use IHttpClientFactory in a console app</span></span>

<span data-ttu-id="4b675-544">In un'app console aggiungere al progetto i riferimenti ai pacchetti seguenti:</span><span class="sxs-lookup"><span data-stu-id="4b675-544">In a console app, add the following package references to the project:</span></span>

* [<span data-ttu-id="4b675-545">Microsoft.Extensions.Hosting</span><span class="sxs-lookup"><span data-stu-id="4b675-545">Microsoft.Extensions.Hosting</span></span>](https://www.nuget.org/packages/Microsoft.Extensions.Hosting)
* [<span data-ttu-id="4b675-546">Microsoft.Extensions.Http</span><span class="sxs-lookup"><span data-stu-id="4b675-546">Microsoft.Extensions.Http</span></span>](https://www.nuget.org/packages/Microsoft.Extensions.Http)

<span data-ttu-id="4b675-547">Nell'esempio seguente:</span><span class="sxs-lookup"><span data-stu-id="4b675-547">In the following example:</span></span>

* <span data-ttu-id="4b675-548"><xref:System.Net.Http.IHttpClientFactory> è registrato nel contenitore di servizi dell'[host generico](xref:fundamentals/host/generic-host).</span><span class="sxs-lookup"><span data-stu-id="4b675-548"><xref:System.Net.Http.IHttpClientFactory> is registered in the [Generic Host's](xref:fundamentals/host/generic-host) service container.</span></span>
* <span data-ttu-id="4b675-549">`MyService` crea un'istanza della factory client dal servizio, che viene usata per creare una classe `HttpClient`.</span><span class="sxs-lookup"><span data-stu-id="4b675-549">`MyService` creates a client factory instance from the service, which is used to create an `HttpClient`.</span></span> <span data-ttu-id="4b675-550">`HttpClient` viene usato per recuperare una pagina Web.</span><span class="sxs-lookup"><span data-stu-id="4b675-550">`HttpClient` is used to retrieve a webpage.</span></span>
* <span data-ttu-id="4b675-551">`Main` crea un ambito per eseguire il metodo `GetPage` del servizio e scrivere i primi 500 caratteri del contenuto della pagina Web nella console.</span><span class="sxs-lookup"><span data-stu-id="4b675-551">`Main` creates a scope to execute the service's `GetPage` method and write the first 500 characters of the webpage content to the console.</span></span>

[!code-csharp[](http-requests/samples/2.x/HttpClientFactoryConsoleSample/Program.cs?highlight=14-15,20,26-27,59-62)]

## <a name="additional-resources"></a><span data-ttu-id="4b675-552">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="4b675-552">Additional resources</span></span>

* [<span data-ttu-id="4b675-553">Usare HttpClientFactory per l'implementazione di richieste HTTP resilienti</span><span class="sxs-lookup"><span data-stu-id="4b675-553">Use HttpClientFactory to implement resilient HTTP requests</span></span>](/dotnet/standard/microservices-architecture/implement-resilient-applications/use-httpclientfactory-to-implement-resilient-http-requests)
* [<span data-ttu-id="4b675-554">Implementazione dei tentativi di chiamate HTTP con backoff esponenziale con i criteri di Polly e HttpClientFactory</span><span class="sxs-lookup"><span data-stu-id="4b675-554">Implement HTTP call retries with exponential backoff with HttpClientFactory and Polly policies</span></span>](/dotnet/standard/microservices-architecture/implement-resilient-applications/implement-http-call-retries-exponential-backoff-polly)
* [<span data-ttu-id="4b675-555">Implementazione dello schema Circuit Breaker</span><span class="sxs-lookup"><span data-stu-id="4b675-555">Implement the Circuit Breaker pattern</span></span>](/dotnet/standard/microservices-architecture/implement-resilient-applications/implement-circuit-breaker-pattern)

::: moniker-end

::: moniker range="= aspnetcore-2.1"

<span data-ttu-id="4b675-556">Di [Glenn Condron](https://github.com/glennc), [Ryan Nowak](https://github.com/rynowak) e [Steve Gordon](https://github.com/stevejgordon)</span><span class="sxs-lookup"><span data-stu-id="4b675-556">By [Glenn Condron](https://github.com/glennc), [Ryan Nowak](https://github.com/rynowak), and [Steve Gordon](https://github.com/stevejgordon)</span></span>

<span data-ttu-id="4b675-557">È possibile registrare e usare un'interfaccia <xref:System.Net.Http.IHttpClientFactory> per configurare e creare istanze di <xref:System.Net.Http.HttpClient> in un'app.</span><span class="sxs-lookup"><span data-stu-id="4b675-557">An <xref:System.Net.Http.IHttpClientFactory> can be registered and used to configure and create <xref:System.Net.Http.HttpClient> instances in an app.</span></span> <span data-ttu-id="4b675-558">I vantaggi offerti sono i seguenti:</span><span class="sxs-lookup"><span data-stu-id="4b675-558">It offers the following benefits:</span></span>

* <span data-ttu-id="4b675-559">Offre una posizione centrale per la denominazione e la configurazione di istanze di `HttpClient` logiche.</span><span class="sxs-lookup"><span data-stu-id="4b675-559">Provides a central location for naming and configuring logical `HttpClient` instances.</span></span> <span data-ttu-id="4b675-560">Ad esempio, è possibile registrare e configurare un client *github* per accedere a [GitHub](https://github.com/).</span><span class="sxs-lookup"><span data-stu-id="4b675-560">For example, a *github* client can be registered and configured to access [GitHub](https://github.com/).</span></span> <span data-ttu-id="4b675-561">È possibile registrare un client predefinito per altri scopi.</span><span class="sxs-lookup"><span data-stu-id="4b675-561">A default client can be registered for other purposes.</span></span>
* <span data-ttu-id="4b675-562">Codifica il concetto di middleware in uscita tramite la delega di gestori in `HttpClient` e offre estensioni per il middleware basato su Polly per sfruttarne i vantaggi.</span><span class="sxs-lookup"><span data-stu-id="4b675-562">Codifies the concept of outgoing middleware via delegating handlers in `HttpClient` and provides extensions for Polly-based middleware to take advantage of that.</span></span>
* <span data-ttu-id="4b675-563">Gestisce il pooling e la durata delle istanze di `HttpClientMessageHandler` sottostanti per evitare problemi DNS comuni che si verificano quando le durate di `HttpClient` vengono gestite manualmente.</span><span class="sxs-lookup"><span data-stu-id="4b675-563">Manages the pooling and lifetime of underlying `HttpClientMessageHandler` instances to avoid common DNS problems that occur when manually managing `HttpClient` lifetimes.</span></span>
* <span data-ttu-id="4b675-564">Aggiunge un'esperienza di registrazione configurabile, tramite `ILogger`, per tutte le richieste inviate attraverso i client creati dalla factory.</span><span class="sxs-lookup"><span data-stu-id="4b675-564">Adds a configurable logging experience (via `ILogger`) for all requests sent through clients created by the factory.</span></span>

<span data-ttu-id="4b675-565">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/http-requests/samples) ([procedura per il download](xref:index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="4b675-565">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/http-requests/samples) ([how to download](xref:index#how-to-download-a-sample))</span></span>

## <a name="prerequisites"></a><span data-ttu-id="4b675-566">Prerequisiti</span><span class="sxs-lookup"><span data-stu-id="4b675-566">Prerequisites</span></span>

<span data-ttu-id="4b675-567">I progetti destinati a .NET Framework richiedono l'installazione del pacchetto NuGet [Microsoft.Extensions.Http](https://www.nuget.org/packages/Microsoft.Extensions.Http/).</span><span class="sxs-lookup"><span data-stu-id="4b675-567">Projects targeting .NET Framework require installation of the [Microsoft.Extensions.Http](https://www.nuget.org/packages/Microsoft.Extensions.Http/) NuGet package.</span></span> <span data-ttu-id="4b675-568">I progetti destinati a .NET Core che fanno riferimento al [metapacchetto Microsoft.AspNetCore.All](xref:fundamentals/metapackage-app) sono già inclusi nel pacchetto `Microsoft.Extensions.Http`.</span><span class="sxs-lookup"><span data-stu-id="4b675-568">Projects that target .NET Core and reference the [Microsoft.AspNetCore.App metapackage](xref:fundamentals/metapackage-app) already include the `Microsoft.Extensions.Http` package.</span></span>

## <a name="consumption-patterns"></a><span data-ttu-id="4b675-569">Modelli di consumo</span><span class="sxs-lookup"><span data-stu-id="4b675-569">Consumption patterns</span></span>

<span data-ttu-id="4b675-570">`IHttpClientFactory` può essere usato in un'app in diversi modi:</span><span class="sxs-lookup"><span data-stu-id="4b675-570">There are several ways `IHttpClientFactory` can be used in an app:</span></span>

* [<span data-ttu-id="4b675-571">Uso di base</span><span class="sxs-lookup"><span data-stu-id="4b675-571">Basic usage</span></span>](#basic-usage)
* [<span data-ttu-id="4b675-572">Client denominati</span><span class="sxs-lookup"><span data-stu-id="4b675-572">Named clients</span></span>](#named-clients)
* [<span data-ttu-id="4b675-573">Client tipizzati</span><span class="sxs-lookup"><span data-stu-id="4b675-573">Typed clients</span></span>](#typed-clients)
* [<span data-ttu-id="4b675-574">Client generati</span><span class="sxs-lookup"><span data-stu-id="4b675-574">Generated clients</span></span>](#generated-clients)

<span data-ttu-id="4b675-575">Nessuno di questi modi può essere considerato superiore a un altro.</span><span class="sxs-lookup"><span data-stu-id="4b675-575">None of them are strictly superior to another.</span></span> <span data-ttu-id="4b675-576">L'approccio migliore dipende dai vincoli dell'app.</span><span class="sxs-lookup"><span data-stu-id="4b675-576">The best approach depends upon the app's constraints.</span></span>

### <a name="basic-usage"></a><span data-ttu-id="4b675-577">Uso di base</span><span class="sxs-lookup"><span data-stu-id="4b675-577">Basic usage</span></span>

<span data-ttu-id="4b675-578">È possibile registrare `IHttpClientFactory` chiamando il metodo di estensione `AddHttpClient` in `IServiceCollection`, all'interno del metodo `Startup.ConfigureServices`.</span><span class="sxs-lookup"><span data-stu-id="4b675-578">The `IHttpClientFactory` can be registered by calling the `AddHttpClient` extension method on the `IServiceCollection`, inside the `Startup.ConfigureServices` method.</span></span>

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet1)]

<span data-ttu-id="4b675-579">Dopo la registrazione, il codice può accettare un'interfaccia `IHttpClientFactory` ovunque sia possibile inserire servizi con l'[inserimento di dipendenze](xref:fundamentals/dependency-injection).</span><span class="sxs-lookup"><span data-stu-id="4b675-579">Once registered, code can accept an `IHttpClientFactory` anywhere services can be injected with [dependency injection (DI)](xref:fundamentals/dependency-injection).</span></span> <span data-ttu-id="4b675-580">`IHttpClientFactory`È possibile utilizzare per creare un' `HttpClient` istanza di:</span><span class="sxs-lookup"><span data-stu-id="4b675-580">The `IHttpClientFactory` can be used to create an `HttpClient` instance:</span></span>

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Pages/BasicUsage.cshtml.cs?name=snippet1&highlight=9-12,21)]

<span data-ttu-id="4b675-581">Questo uso di `IHttpClientFactory` è un modo efficace per effettuare il refactoring di un'app esistente.</span><span class="sxs-lookup"><span data-stu-id="4b675-581">Using `IHttpClientFactory` in this fashion is a good way to refactor an existing app.</span></span> <span data-ttu-id="4b675-582">Non influisce in alcun modo sulle modalità d'uso di `HttpClient`.</span><span class="sxs-lookup"><span data-stu-id="4b675-582">It has no impact on the way `HttpClient` is used.</span></span> <span data-ttu-id="4b675-583">Nelle posizioni in cui vengono attualmente create istanze di `HttpClient`, sostituire le occorrenze con una chiamata a <xref:System.Net.Http.IHttpClientFactory.CreateClient*>.</span><span class="sxs-lookup"><span data-stu-id="4b675-583">In places where `HttpClient` instances are currently created, replace those occurrences with a call to <xref:System.Net.Http.IHttpClientFactory.CreateClient*>.</span></span>

### <a name="named-clients"></a><span data-ttu-id="4b675-584">Client denominati</span><span class="sxs-lookup"><span data-stu-id="4b675-584">Named clients</span></span>

<span data-ttu-id="4b675-585">Se un'app richiede molti usi distinti di `HttpClient`, ognuno con una configurazione diversa, un'opzione è l'uso di **client denominati**.</span><span class="sxs-lookup"><span data-stu-id="4b675-585">If an app requires many distinct uses of `HttpClient`, each with a different configuration, an option is to use **named clients**.</span></span> <span data-ttu-id="4b675-586">La configurazione di un `HttpClient` denominato può essere specificata durante la registrazione in `Startup.ConfigureServices`.</span><span class="sxs-lookup"><span data-stu-id="4b675-586">Configuration for a named `HttpClient` can be specified during registration in `Startup.ConfigureServices`.</span></span>

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet2)]

<span data-ttu-id="4b675-587">Nel codice precedente viene chiamato `AddHttpClient`, in cui viene specificato il nome *github*.</span><span class="sxs-lookup"><span data-stu-id="4b675-587">In the preceding code, `AddHttpClient` is called, providing the name *github*.</span></span> <span data-ttu-id="4b675-588">Al client viene applicata una configurazione predefinita, ovvero l'indirizzo di base e due intestazioni necessari per l'uso dell'API GitHub.</span><span class="sxs-lookup"><span data-stu-id="4b675-588">This client has some default configuration applied&mdash;namely the base address and two headers required to work with the GitHub API.</span></span>

<span data-ttu-id="4b675-589">Ogni volta che `CreateClient` viene chiamato, verrà creata una nuova istanza di `HttpClient` e verrà chiamata l'azione di configurazione.</span><span class="sxs-lookup"><span data-stu-id="4b675-589">Each time `CreateClient` is called, a new instance of `HttpClient` is created and the configuration action is called.</span></span>

<span data-ttu-id="4b675-590">Per usare un client denominato, è possibile passare un parametro di stringa a `CreateClient`.</span><span class="sxs-lookup"><span data-stu-id="4b675-590">To consume a named client, a string parameter can be passed to `CreateClient`.</span></span> <span data-ttu-id="4b675-591">Specificare il nome del client da creare:</span><span class="sxs-lookup"><span data-stu-id="4b675-591">Specify the name of the client to be created:</span></span>

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Pages/NamedClient.cshtml.cs?name=snippet1&highlight=21)]

<span data-ttu-id="4b675-592">Nel codice precedente non è necessario che la richiesta specifichi un nome host.</span><span class="sxs-lookup"><span data-stu-id="4b675-592">In the preceding code, the request doesn't need to specify a hostname.</span></span> <span data-ttu-id="4b675-593">Poiché viene usato l'indirizzo di base configurato per il client, è possibile passare solo il percorso.</span><span class="sxs-lookup"><span data-stu-id="4b675-593">It can pass just the path, since the base address configured for the client is used.</span></span>

### <a name="typed-clients"></a><span data-ttu-id="4b675-594">Client tipizzati</span><span class="sxs-lookup"><span data-stu-id="4b675-594">Typed clients</span></span>

<span data-ttu-id="4b675-595">Client tipizzati:</span><span class="sxs-lookup"><span data-stu-id="4b675-595">Typed clients:</span></span>

* <span data-ttu-id="4b675-596">Offrono le stesse funzionalità dei client denominati senza la necessità di usare le stringhe come chiavi.</span><span class="sxs-lookup"><span data-stu-id="4b675-596">Provide the same capabilities as named clients without the need to use strings as keys.</span></span>
* <span data-ttu-id="4b675-597">Offrono l'aiuto di IntelliSense e del compilatore quando si usano i client.</span><span class="sxs-lookup"><span data-stu-id="4b675-597">Provides IntelliSense and compiler help when consuming clients.</span></span>
* <span data-ttu-id="4b675-598">La configurazione e l'interazione con un particolare `HttpClient` può avvenire in un'unica posizione.</span><span class="sxs-lookup"><span data-stu-id="4b675-598">Provide a single location to configure and interact with a particular `HttpClient`.</span></span> <span data-ttu-id="4b675-599">Per un singolo endpoint back-end è ad esempio possibile usare un unico client tipizzato che incapsuli tutta la logica relativa all'endpoint.</span><span class="sxs-lookup"><span data-stu-id="4b675-599">For example, a single typed client might be used for a single backend endpoint and encapsulate all logic dealing with that endpoint.</span></span>
* <span data-ttu-id="4b675-600">Usano l'inserimento di dipendenze e possono essere inseriti dove necessario nell'app.</span><span class="sxs-lookup"><span data-stu-id="4b675-600">Work with DI and can be injected where required in your app.</span></span>

<span data-ttu-id="4b675-601">Un client tipizzato accetta un `HttpClient` parametro nel relativo costruttore:</span><span class="sxs-lookup"><span data-stu-id="4b675-601">A typed client accepts an `HttpClient` parameter in its constructor:</span></span>

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/GitHub/GitHubService.cs?name=snippet1&highlight=5)]

<span data-ttu-id="4b675-602">Nel codice precedente la configurazione viene spostata nel client tipizzato.</span><span class="sxs-lookup"><span data-stu-id="4b675-602">In the preceding code, the configuration is moved into the typed client.</span></span> <span data-ttu-id="4b675-603">L'oggetto `HttpClient` viene esposto come una proprietà pubblica.</span><span class="sxs-lookup"><span data-stu-id="4b675-603">The `HttpClient` object is exposed as a public property.</span></span> <span data-ttu-id="4b675-604">È possibile definire metodi di API specifiche che espongono la funzionalità `HttpClient`.</span><span class="sxs-lookup"><span data-stu-id="4b675-604">It's possible to define API-specific methods that expose `HttpClient` functionality.</span></span> <span data-ttu-id="4b675-605">Il metodo `GetAspNetDocsIssues` incapsula il codice necessario per eseguire una query e analizzare gli ultimi problemi aperti in un repository GitHub.</span><span class="sxs-lookup"><span data-stu-id="4b675-605">The `GetAspNetDocsIssues` method encapsulates the code needed to query for and parse out the latest open issues from a GitHub repository.</span></span>

<span data-ttu-id="4b675-606">Per registrare un client tipizzato è possibile usare il metodo di estensione <xref:Microsoft.Extensions.DependencyInjection.HttpClientFactoryServiceCollectionExtensions.AddHttpClient*> generico all'interno di `Startup.ConfigureServices`, specificando la classe del client tipizzato:</span><span class="sxs-lookup"><span data-stu-id="4b675-606">To register a typed client, the generic <xref:Microsoft.Extensions.DependencyInjection.HttpClientFactoryServiceCollectionExtensions.AddHttpClient*> extension method can be used within `Startup.ConfigureServices`, specifying the typed client class:</span></span>

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet3)]

<span data-ttu-id="4b675-607">Il client tipizzato viene registrato come temporaneo nell'inserimento di dipendenze.</span><span class="sxs-lookup"><span data-stu-id="4b675-607">The typed client is registered as transient with DI.</span></span> <span data-ttu-id="4b675-608">Il client tipizzato può essere inserito e usato direttamente:</span><span class="sxs-lookup"><span data-stu-id="4b675-608">The typed client can be injected and consumed directly:</span></span>

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Pages/TypedClient.cshtml.cs?name=snippet1&highlight=11-14,20)]

<span data-ttu-id="4b675-609">Se si preferisce, è possibile specificare la configurazione di un client tipizzato durante la registrazione in `Startup.ConfigureServices`, anziché nel relativo costruttore:</span><span class="sxs-lookup"><span data-stu-id="4b675-609">If preferred, the configuration for a typed client can be specified during registration in `Startup.ConfigureServices`, rather than in the typed client's constructor:</span></span>

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet4)]

<span data-ttu-id="4b675-610">È possibile incapsulare interamente `HttpClient` all'interno di un client tipizzato.</span><span class="sxs-lookup"><span data-stu-id="4b675-610">It's possible to entirely encapsulate the `HttpClient` within a typed client.</span></span> <span data-ttu-id="4b675-611">Anziché esporlo come una proprietà, è possibile specificare metodi pubblici che chiamano l'istanza di `HttpClient` internamente.</span><span class="sxs-lookup"><span data-stu-id="4b675-611">Rather than exposing it as a property, public methods can be provided which call the `HttpClient` instance internally.</span></span>

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/GitHub/RepoService.cs?name=snippet1&highlight=4)]

<span data-ttu-id="4b675-612">Nel codice precedente `HttpClient` viene archiviato come un campo privato.</span><span class="sxs-lookup"><span data-stu-id="4b675-612">In the preceding code, the `HttpClient` is stored as a private field.</span></span> <span data-ttu-id="4b675-613">L'accesso per effettuare chiamate esterne passa attraverso il metodo `GetRepos`.</span><span class="sxs-lookup"><span data-stu-id="4b675-613">All access to make external calls goes through the `GetRepos` method.</span></span>

### <a name="generated-clients"></a><span data-ttu-id="4b675-614">Client generati</span><span class="sxs-lookup"><span data-stu-id="4b675-614">Generated clients</span></span>

<span data-ttu-id="4b675-615">È possibile usare `IHttpClientFactory` in combinazione con altre librerie di terze parti, ad esempio [Refit](https://github.com/paulcbetts/refit).</span><span class="sxs-lookup"><span data-stu-id="4b675-615">`IHttpClientFactory` can be used in combination with other third-party libraries such as [Refit](https://github.com/paulcbetts/refit).</span></span> <span data-ttu-id="4b675-616">Refit è una libreria REST per .NET.</span><span class="sxs-lookup"><span data-stu-id="4b675-616">Refit is a REST library for .NET.</span></span> <span data-ttu-id="4b675-617">Converte le API REST in interfacce live.</span><span class="sxs-lookup"><span data-stu-id="4b675-617">It converts REST APIs into live interfaces.</span></span> <span data-ttu-id="4b675-618">Un'implementazione dell'interfaccia viene generata dinamicamente da `RestService`, usando `HttpClient` per effettuare le chiamate HTTP esterne.</span><span class="sxs-lookup"><span data-stu-id="4b675-618">An implementation of the interface is generated dynamically by the `RestService`, using `HttpClient` to make the external HTTP calls.</span></span>

<span data-ttu-id="4b675-619">Per rappresentare l'API esterna e la relativa risposta vengono definite un'interfaccia e una risposta:</span><span class="sxs-lookup"><span data-stu-id="4b675-619">An interface and a reply are defined to represent the external API and its response:</span></span>

```csharp
public interface IHelloClient
{
    [Get("/helloworld")]
    Task<Reply> GetMessageAsync();
}

public class Reply
{
    public string Message { get; set; }
}
```

<span data-ttu-id="4b675-620">È possibile aggiungere un client tipizzato usando Refit per generare l'implementazione:</span><span class="sxs-lookup"><span data-stu-id="4b675-620">A typed client can be added, using Refit to generate the implementation:</span></span>

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddHttpClient("hello", c =>
    {
        c.BaseAddress = new Uri("http://localhost:5000");
    })
    .AddTypedClient(c => Refit.RestService.For<IHelloClient>(c));

    services.AddMvc();
}
```

<span data-ttu-id="4b675-621">L'interfaccia definita può essere usata dove necessario, con l'implementazione offerta dall'inserimento di dipendenze e da Refit:</span><span class="sxs-lookup"><span data-stu-id="4b675-621">The defined interface can be consumed where necessary, with the implementation provided by DI and Refit:</span></span>

```csharp
[ApiController]
public class ValuesController : ControllerBase
{
    private readonly IHelloClient _client;

    public ValuesController(IHelloClient client)
    {
        _client = client;
    }

    [HttpGet("/")]
    public async Task<ActionResult<Reply>> Index()
    {
        return await _client.GetMessageAsync();
    }
}
```

## <a name="outgoing-request-middleware"></a><span data-ttu-id="4b675-622">Middleware per richieste in uscita</span><span class="sxs-lookup"><span data-stu-id="4b675-622">Outgoing request middleware</span></span>

<span data-ttu-id="4b675-623">`HttpClient` include già il concetto di delega di gestori concatenati per le richieste HTTP in uscita.</span><span class="sxs-lookup"><span data-stu-id="4b675-623">`HttpClient` already has the concept of delegating handlers that can be linked together for outgoing HTTP requests.</span></span> <span data-ttu-id="4b675-624">`IHttpClientFactory` semplifica la definizione dei gestori da applicare per ogni client denominato.</span><span class="sxs-lookup"><span data-stu-id="4b675-624">The `IHttpClientFactory` makes it easy to define the handlers to apply for each named client.</span></span> <span data-ttu-id="4b675-625">Supporta la registrazione e il concatenamento di più gestori per creare una pipeline di middleware per le richieste in uscita.</span><span class="sxs-lookup"><span data-stu-id="4b675-625">It supports registration and chaining of multiple handlers to build an outgoing request middleware pipeline.</span></span> <span data-ttu-id="4b675-626">Ognuno di questi gestori è in grado di eseguire operazioni prima e dopo la richiesta in uscita.</span><span class="sxs-lookup"><span data-stu-id="4b675-626">Each of these handlers is able to perform work before and after the outgoing request.</span></span> <span data-ttu-id="4b675-627">Questo modello è simile alla pipeline di middleware in ingresso in ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="4b675-627">This pattern is similar to the inbound middleware pipeline in ASP.NET Core.</span></span> <span data-ttu-id="4b675-628">Il modello offre un meccanismo per gestire le problematiche trasversali relative alle richieste HTTP, tra cui memorizzazione nella cache, gestione degli errori, serializzazione e registrazione.</span><span class="sxs-lookup"><span data-stu-id="4b675-628">The pattern provides a mechanism to manage cross-cutting concerns around HTTP requests, including caching, error handling, serialization, and logging.</span></span>

<span data-ttu-id="4b675-629">Per creare un gestore, definire una classe che deriva da <xref:System.Net.Http.DelegatingHandler>.</span><span class="sxs-lookup"><span data-stu-id="4b675-629">To create a handler, define a class deriving from <xref:System.Net.Http.DelegatingHandler>.</span></span> <span data-ttu-id="4b675-630">Eseguire l'override del metodo `SendAsync` per eseguire il codice prima di passare la richiesta al gestore successivo nella pipeline:</span><span class="sxs-lookup"><span data-stu-id="4b675-630">Override the `SendAsync` method to execute code before passing the request to the next handler in the pipeline:</span></span>

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Handlers/ValidateHeaderHandler.cs?name=snippet1)]

<span data-ttu-id="4b675-631">Il codice precedente definisce un gestore di base.</span><span class="sxs-lookup"><span data-stu-id="4b675-631">The preceding code defines a basic handler.</span></span> <span data-ttu-id="4b675-632">Verifica se un'intestazione `X-API-KEY` è stata inclusa nella richiesta.</span><span class="sxs-lookup"><span data-stu-id="4b675-632">It checks to see if an `X-API-KEY` header has been included on the request.</span></span> <span data-ttu-id="4b675-633">Se l'intestazione non è presente, può evitare la chiamata HTTP e restituire una risposta appropriata.</span><span class="sxs-lookup"><span data-stu-id="4b675-633">If the header is missing, it can avoid the HTTP call and return a suitable response.</span></span>

<span data-ttu-id="4b675-634">Durante la registrazione, è possibile aggiungere uno o più gestori alla configurazione per un `HttpClient` .</span><span class="sxs-lookup"><span data-stu-id="4b675-634">During registration, one or more handlers can be added to the configuration for an `HttpClient`.</span></span> <span data-ttu-id="4b675-635">Questa attività viene eseguita tramite metodi di estensione in <xref:Microsoft.Extensions.DependencyInjection.IHttpClientBuilder>.</span><span class="sxs-lookup"><span data-stu-id="4b675-635">This task is accomplished via extension methods on the <xref:Microsoft.Extensions.DependencyInjection.IHttpClientBuilder>.</span></span>

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet5)]

<span data-ttu-id="4b675-636">Nel codice precedente `ValidateHeaderHandler` viene registrato nell'inserimento di dipendenze.</span><span class="sxs-lookup"><span data-stu-id="4b675-636">In the preceding code, the `ValidateHeaderHandler` is registered with DI.</span></span> <span data-ttu-id="4b675-637">Il gestore **deve** essere registrato nell'inserimento di dipendenze come servizio temporaneo, senza definizione di ambito.</span><span class="sxs-lookup"><span data-stu-id="4b675-637">The handler **must** be registered in DI as a transient service, never scoped.</span></span> <span data-ttu-id="4b675-638">Se il gestore viene registrato come un servizio con ambito e i servizi da cui dipende il gestore sono eliminabili:</span><span class="sxs-lookup"><span data-stu-id="4b675-638">If the handler is registered as a scoped service and any services that the handler depends upon are disposable:</span></span>

* <span data-ttu-id="4b675-639">I servizi del gestore potrebbero essere eliminati prima che il gestore esca dall'ambito.</span><span class="sxs-lookup"><span data-stu-id="4b675-639">The handler's services could be disposed before the handler goes out of scope.</span></span>
* <span data-ttu-id="4b675-640">I servizi del gestore eliminati causano un errore del gestore.</span><span class="sxs-lookup"><span data-stu-id="4b675-640">The disposed handler services causes the handler to fail.</span></span>

<span data-ttu-id="4b675-641">Dopo la registrazione è possibile chiamare <xref:Microsoft.Extensions.DependencyInjection.HttpClientBuilderExtensions.AddHttpMessageHandler*>, passando il tipo di gestore.</span><span class="sxs-lookup"><span data-stu-id="4b675-641">Once registered, <xref:Microsoft.Extensions.DependencyInjection.HttpClientBuilderExtensions.AddHttpMessageHandler*> can be called, passing in the handler type.</span></span>

<span data-ttu-id="4b675-642">È possibile registrare più gestori nell'ordine di esecuzione.</span><span class="sxs-lookup"><span data-stu-id="4b675-642">Multiple handlers can be registered in the order that they should execute.</span></span> <span data-ttu-id="4b675-643">Ogni gestore esegue il wrapping del gestore successivo fino a quando l'elemento `HttpClientHandler` finale non esegue la richiesta:</span><span class="sxs-lookup"><span data-stu-id="4b675-643">Each handler wraps the next handler until the final `HttpClientHandler` executes the request:</span></span>

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet6)]

<span data-ttu-id="4b675-644">Usare uno degli approcci seguenti per condividere lo stato in base alla richiesta con i gestori di messaggi:</span><span class="sxs-lookup"><span data-stu-id="4b675-644">Use one of the following approaches to share per-request state with message handlers:</span></span>

* <span data-ttu-id="4b675-645">Passare i dati al gestore usando `HttpRequestMessage.Properties`.</span><span class="sxs-lookup"><span data-stu-id="4b675-645">Pass data into the handler using `HttpRequestMessage.Properties`.</span></span>
* <span data-ttu-id="4b675-646">Usare `IHttpContextAccessor` per accedere alla richiesta corrente.</span><span class="sxs-lookup"><span data-stu-id="4b675-646">Use `IHttpContextAccessor` to access the current request.</span></span>
* <span data-ttu-id="4b675-647">Creare un oggetto di archiviazione `AsyncLocal` personalizzato per passare i dati.</span><span class="sxs-lookup"><span data-stu-id="4b675-647">Create a custom `AsyncLocal` storage object to pass the data.</span></span>

## <a name="use-polly-based-handlers"></a><span data-ttu-id="4b675-648">Usare gestori basati su Polly</span><span class="sxs-lookup"><span data-stu-id="4b675-648">Use Polly-based handlers</span></span>

<span data-ttu-id="4b675-649">`IHttpClientFactory` si integra con una libreria di terze parti piuttosto diffusa denominata [Polly](https://github.com/App-vNext/Polly).</span><span class="sxs-lookup"><span data-stu-id="4b675-649">`IHttpClientFactory` integrates with a popular third-party library called [Polly](https://github.com/App-vNext/Polly).</span></span> <span data-ttu-id="4b675-650">Polly è una libreria di gestione degli errori temporanei e di resilienza completa per .NET.</span><span class="sxs-lookup"><span data-stu-id="4b675-650">Polly is a comprehensive resilience and transient fault-handling library for .NET.</span></span> <span data-ttu-id="4b675-651">Consente agli sviluppatori di esprimere criteri quali Retry, Circuit Breaker, Timeout, Bulkhead Isolation e Fallback in modo fluido e thread-safe.</span><span class="sxs-lookup"><span data-stu-id="4b675-651">It allows developers to express policies such as Retry, Circuit Breaker, Timeout, Bulkhead Isolation, and Fallback in a fluent and thread-safe manner.</span></span>

<span data-ttu-id="4b675-652">Per consentire l'uso dei criteri Polly con le istanze configurate di `HttpClient` sono disponibili metodi di estensione.</span><span class="sxs-lookup"><span data-stu-id="4b675-652">Extension methods are provided to enable the use of Polly policies with configured `HttpClient` instances.</span></span> <span data-ttu-id="4b675-653">Le estensioni Polly:</span><span class="sxs-lookup"><span data-stu-id="4b675-653">The Polly extensions:</span></span>

* <span data-ttu-id="4b675-654">Supportano l'aggiunta di gestori basati su Polly ai client.</span><span class="sxs-lookup"><span data-stu-id="4b675-654">Support adding Polly-based handlers to clients.</span></span>
* <span data-ttu-id="4b675-655">Possono essere usate dopo l'installazione del pacchetto NuGet [Microsoft.Extensions.Http.Polly](https://www.nuget.org/packages/Microsoft.Extensions.Http.Polly/).</span><span class="sxs-lookup"><span data-stu-id="4b675-655">Can be used after installing the [Microsoft.Extensions.Http.Polly](https://www.nuget.org/packages/Microsoft.Extensions.Http.Polly/) NuGet package.</span></span> <span data-ttu-id="4b675-656">Il pacchetto non è incluso nel framework condiviso ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="4b675-656">The package isn't included in the ASP.NET Core shared framework.</span></span>

### <a name="handle-transient-faults"></a><span data-ttu-id="4b675-657">Gestire gli errori temporanei</span><span class="sxs-lookup"><span data-stu-id="4b675-657">Handle transient faults</span></span>

<span data-ttu-id="4b675-658">La maggior parte degli errori comuni si verifica quando le chiamate HTTP esterne sono temporanee.</span><span class="sxs-lookup"><span data-stu-id="4b675-658">Most common faults occur when external HTTP calls are transient.</span></span> <span data-ttu-id="4b675-659">Per definire un criterio in grado di gestire gli errori temporanei è disponibile un pratico metodo di estensione denominato `AddTransientHttpErrorPolicy`.</span><span class="sxs-lookup"><span data-stu-id="4b675-659">A convenient extension method called `AddTransientHttpErrorPolicy` is included which allows a policy to be defined to handle transient errors.</span></span> <span data-ttu-id="4b675-660">I criteri configurati con questo metodo di estensione gestiscono `HttpRequestException`, risposte HTTP 5xx e risposte HTTP 408.</span><span class="sxs-lookup"><span data-stu-id="4b675-660">Policies configured with this extension method handle `HttpRequestException`, HTTP 5xx responses, and HTTP 408 responses.</span></span>

<span data-ttu-id="4b675-661">L'estensione `AddTransientHttpErrorPolicy` può essere usata all'interno di `Startup.ConfigureServices`.</span><span class="sxs-lookup"><span data-stu-id="4b675-661">The `AddTransientHttpErrorPolicy` extension can be used within `Startup.ConfigureServices`.</span></span> <span data-ttu-id="4b675-662">L'estensione consente l'accesso a un oggetto `PolicyBuilder` configurato per gestire gli errori che rappresentano un possibile errore temporaneo:</span><span class="sxs-lookup"><span data-stu-id="4b675-662">The extension provides access to a `PolicyBuilder` object configured to handle errors representing a possible transient fault:</span></span>

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet7)]

<span data-ttu-id="4b675-663">Nel codice precedente viene definito un criterio `WaitAndRetryAsync`.</span><span class="sxs-lookup"><span data-stu-id="4b675-663">In the preceding code, a `WaitAndRetryAsync` policy is defined.</span></span> <span data-ttu-id="4b675-664">Le richieste non riuscite vengono ritentate fino a tre volte con un ritardo di 600 millisecondi tra i tentativi.</span><span class="sxs-lookup"><span data-stu-id="4b675-664">Failed requests are retried up to three times with a delay of 600 ms between attempts.</span></span>

### <a name="dynamically-select-policies"></a><span data-ttu-id="4b675-665">Selezionare i criteri in modo dinamico</span><span class="sxs-lookup"><span data-stu-id="4b675-665">Dynamically select policies</span></span>

<span data-ttu-id="4b675-666">Per aggiungere gestori basati su Polly è possibile usare altri metodi di estensione.</span><span class="sxs-lookup"><span data-stu-id="4b675-666">Additional extension methods exist which can be used to add Polly-based handlers.</span></span> <span data-ttu-id="4b675-667">Una di queste estensioni è `AddPolicyHandler`, che include più overload.</span><span class="sxs-lookup"><span data-stu-id="4b675-667">One such extension is `AddPolicyHandler`, which has multiple overloads.</span></span> <span data-ttu-id="4b675-668">Un overload consente l'ispezione della richiesta al momento della definizione del criterio da applicare:</span><span class="sxs-lookup"><span data-stu-id="4b675-668">One overload allows the request to be inspected when defining which policy to apply:</span></span>

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet8)]

<span data-ttu-id="4b675-669">Nel codice precedente, se la richiesta in uscita è una richiesta HTTP GET viene applicato un timeout di 10 secondi.</span><span class="sxs-lookup"><span data-stu-id="4b675-669">In the preceding code, if the outgoing request is an HTTP GET, a 10-second timeout is applied.</span></span> <span data-ttu-id="4b675-670">Per qualsiasi altro metodo HTTP viene usato un timeout di 30 secondi.</span><span class="sxs-lookup"><span data-stu-id="4b675-670">For any other HTTP method, a 30-second timeout is used.</span></span>

### <a name="add-multiple-polly-handlers"></a><span data-ttu-id="4b675-671">Aggiungere più gestori Polly</span><span class="sxs-lookup"><span data-stu-id="4b675-671">Add multiple Polly handlers</span></span>

<span data-ttu-id="4b675-672">In molti casi i criteri Polly vengono annidati per offrire funzionalità avanzate:</span><span class="sxs-lookup"><span data-stu-id="4b675-672">It's common to nest Polly policies to provide enhanced functionality:</span></span>

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet9)]

<span data-ttu-id="4b675-673">Nell'esempio precedente vengono aggiunti due gestori.</span><span class="sxs-lookup"><span data-stu-id="4b675-673">In the preceding example, two handlers are added.</span></span> <span data-ttu-id="4b675-674">Il primo usa l'estensione `AddTransientHttpErrorPolicy` per aggiungere criteri di ripetizione.</span><span class="sxs-lookup"><span data-stu-id="4b675-674">The first uses the `AddTransientHttpErrorPolicy` extension to add a retry policy.</span></span> <span data-ttu-id="4b675-675">Le richieste non riuscite vengono ritentate fino a tre volte.</span><span class="sxs-lookup"><span data-stu-id="4b675-675">Failed requests are retried up to three times.</span></span> <span data-ttu-id="4b675-676">La seconda chiamata a `AddTransientHttpErrorPolicy` aggiunge criteri dell'interruttore di circuito.</span><span class="sxs-lookup"><span data-stu-id="4b675-676">The second call to `AddTransientHttpErrorPolicy` adds a circuit breaker policy.</span></span> <span data-ttu-id="4b675-677">Ulteriori richieste esterne vengono bloccate per 30 secondi nel caso si verifichino cinque tentativi non riusciti consecutivi.</span><span class="sxs-lookup"><span data-stu-id="4b675-677">Further external requests are blocked for 30 seconds if five failed attempts occur sequentially.</span></span> <span data-ttu-id="4b675-678">I criteri dell'interruttore di circuito sono con stato.</span><span class="sxs-lookup"><span data-stu-id="4b675-678">Circuit breaker policies are stateful.</span></span> <span data-ttu-id="4b675-679">Tutte le chiamate tramite questo client condividono lo stesso stato di circuito.</span><span class="sxs-lookup"><span data-stu-id="4b675-679">All calls through this client share the same circuit state.</span></span>

### <a name="add-policies-from-the-polly-registry"></a><span data-ttu-id="4b675-680">Aggiungere criteri dal registro Polly</span><span class="sxs-lookup"><span data-stu-id="4b675-680">Add policies from the Polly registry</span></span>

<span data-ttu-id="4b675-681">Un approccio alla gestione dei criteri usati di frequente consiste nel definirli una volta e registrarli in un elemento `PolicyRegistry`.</span><span class="sxs-lookup"><span data-stu-id="4b675-681">An approach to managing regularly used policies is to define them once and register them with a `PolicyRegistry`.</span></span> <span data-ttu-id="4b675-682">Per aggiungere un gestore usando un criterio del registro è disponibile un metodo di estensione:</span><span class="sxs-lookup"><span data-stu-id="4b675-682">An extension method is provided which allows a handler to be added using a policy from the registry:</span></span>

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet10)]

<span data-ttu-id="4b675-683">Nel codice precedente vengono registrati due criteri quando si aggiunge `PolicyRegistry` a `ServiceCollection`.</span><span class="sxs-lookup"><span data-stu-id="4b675-683">In the preceding code, two policies are registered when the `PolicyRegistry` is added to the `ServiceCollection`.</span></span> <span data-ttu-id="4b675-684">Per usare un criterio dal registro viene usato il metodo `AddPolicyHandlerFromRegistry` passando il nome del criterio da applicare.</span><span class="sxs-lookup"><span data-stu-id="4b675-684">To use a policy from the registry, the `AddPolicyHandlerFromRegistry` method is used, passing the name of the policy to apply.</span></span>

<span data-ttu-id="4b675-685">Altre informazioni su `IHttpClientFactory` e le integrazioni con Polly sono disponibili nel [wiki di Polly](https://github.com/App-vNext/Polly/wiki/Polly-and-HttpClientFactory).</span><span class="sxs-lookup"><span data-stu-id="4b675-685">Further information about `IHttpClientFactory` and Polly integrations can be found on the [Polly wiki](https://github.com/App-vNext/Polly/wiki/Polly-and-HttpClientFactory).</span></span>

## <a name="httpclient-and-lifetime-management"></a><span data-ttu-id="4b675-686">Gestione di HttpClient e durata</span><span class="sxs-lookup"><span data-stu-id="4b675-686">HttpClient and lifetime management</span></span>

<span data-ttu-id="4b675-687">Viene restituita una nuova istanza di `HttpClient` per ogni chiamata di `CreateClient` per `IHttpClientFactory`.</span><span class="sxs-lookup"><span data-stu-id="4b675-687">A new `HttpClient` instance is returned each time `CreateClient` is called on the `IHttpClientFactory`.</span></span> <span data-ttu-id="4b675-688">È presente un <xref:System.Net.Http.HttpMessageHandler> per ogni client denominato.</span><span class="sxs-lookup"><span data-stu-id="4b675-688">There's an <xref:System.Net.Http.HttpMessageHandler> per named client.</span></span> <span data-ttu-id="4b675-689">La factory gestisce le durate delle istanze di `HttpMessageHandler`.</span><span class="sxs-lookup"><span data-stu-id="4b675-689">The factory manages the lifetimes of the `HttpMessageHandler` instances.</span></span>

<span data-ttu-id="4b675-690">`IHttpClientFactory` esegue il pooling delle istanze di `HttpMessageHandler` create dalla factory per ridurre il consumo di risorse.</span><span class="sxs-lookup"><span data-stu-id="4b675-690">`IHttpClientFactory` pools the `HttpMessageHandler` instances created by the factory to reduce resource consumption.</span></span> <span data-ttu-id="4b675-691">Un'istanza di `HttpMessageHandler` può essere riusata dal pool quando si crea una nuova istanza di `HttpClient` se la relativa durata non è scaduta.</span><span class="sxs-lookup"><span data-stu-id="4b675-691">An `HttpMessageHandler` instance may be reused from the pool when creating a new `HttpClient` instance if its lifetime hasn't expired.</span></span>

<span data-ttu-id="4b675-692">Il pooling dei gestori è consigliabile in quanto ogni gestore gestisce in genere le proprie connessioni HTTP sottostanti.</span><span class="sxs-lookup"><span data-stu-id="4b675-692">Pooling of handlers is desirable as each handler typically manages its own underlying HTTP connections.</span></span> <span data-ttu-id="4b675-693">La creazione di più gestori del necessario può causare ritardi di connessione.</span><span class="sxs-lookup"><span data-stu-id="4b675-693">Creating more handlers than necessary can result in connection delays.</span></span> <span data-ttu-id="4b675-694">Alcuni gestori mantengono inoltre le connessioni aperte a tempo indefinito. Ciò può impedire al gestore di reagire alle modifiche DNS.</span><span class="sxs-lookup"><span data-stu-id="4b675-694">Some handlers also keep connections open indefinitely, which can prevent the handler from reacting to DNS changes.</span></span>

<span data-ttu-id="4b675-695">La durata del gestore predefinito è di due minuti.</span><span class="sxs-lookup"><span data-stu-id="4b675-695">The default handler lifetime is two minutes.</span></span> <span data-ttu-id="4b675-696">Il valore predefinito può essere sottoposto a override per ogni client denominato.</span><span class="sxs-lookup"><span data-stu-id="4b675-696">The default value can be overridden on a per named client basis.</span></span> <span data-ttu-id="4b675-697">Per eseguire l'override, chiamare <xref:Microsoft.Extensions.DependencyInjection.HttpClientBuilderExtensions.SetHandlerLifetime*> nell'elemento `IHttpClientBuilder` restituito al momento della creazione del client:</span><span class="sxs-lookup"><span data-stu-id="4b675-697">To override it, call <xref:Microsoft.Extensions.DependencyInjection.HttpClientBuilderExtensions.SetHandlerLifetime*> on the `IHttpClientBuilder` that is returned when creating the client:</span></span>

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet11)]

<span data-ttu-id="4b675-698">L'eliminazione del client non è obbligatoria.</span><span class="sxs-lookup"><span data-stu-id="4b675-698">Disposal of the client isn't required.</span></span> <span data-ttu-id="4b675-699">L'eliminazione annulla le richieste in uscita e garantisce che l'istanza di `HttpClient` specificata non possa essere usata dopo la chiamata a <xref:System.IDisposable.Dispose*>.</span><span class="sxs-lookup"><span data-stu-id="4b675-699">Disposal cancels outgoing requests and guarantees the given `HttpClient` instance can't be used after calling <xref:System.IDisposable.Dispose*>.</span></span> <span data-ttu-id="4b675-700">`IHttpClientFactory` tiene traccia ed elimina le risorse usate dalle istanze di `HttpClient`.</span><span class="sxs-lookup"><span data-stu-id="4b675-700">`IHttpClientFactory` tracks and disposes resources used by `HttpClient` instances.</span></span> <span data-ttu-id="4b675-701">Le istanze di `HttpClient` possono essere considerate a livello generale come oggetti .NET che non richiedono l'eliminazione.</span><span class="sxs-lookup"><span data-stu-id="4b675-701">The `HttpClient` instances can generally be treated as .NET objects not requiring disposal.</span></span>

<span data-ttu-id="4b675-702">Mantenere una sola istanza di `HttpClient` attiva per un lungo periodo di tempo è un modello comune usato prima dell'avvento di `IHttpClientFactory`.</span><span class="sxs-lookup"><span data-stu-id="4b675-702">Keeping a single `HttpClient` instance alive for a long duration is a common pattern used before the inception of `IHttpClientFactory`.</span></span> <span data-ttu-id="4b675-703">Questo modello non è più necessario dopo la migrazione a `IHttpClientFactory`.</span><span class="sxs-lookup"><span data-stu-id="4b675-703">This pattern becomes unnecessary after migrating to `IHttpClientFactory`.</span></span>

### <a name="alternatives-to-ihttpclientfactory"></a><span data-ttu-id="4b675-704">Alternative a IHttpClientFactory</span><span class="sxs-lookup"><span data-stu-id="4b675-704">Alternatives to IHttpClientFactory</span></span>

<span data-ttu-id="4b675-705">`IHttpClientFactory`L'uso di in un'app abilitata per la funzionalità consente di evitare:</span><span class="sxs-lookup"><span data-stu-id="4b675-705">Using `IHttpClientFactory` in a DI-enabled app avoids:</span></span>

* <span data-ttu-id="4b675-706">Problemi di esaurimento delle risorse in base alle istanze di pool `HttpMessageHandler` .</span><span class="sxs-lookup"><span data-stu-id="4b675-706">Resource exhaustion problems by pooling `HttpMessageHandler` instances.</span></span>
* <span data-ttu-id="4b675-707">Problemi di DNS obsoleti tramite il ciclo di `HttpMessageHandler` istanze a intervalli regolari.</span><span class="sxs-lookup"><span data-stu-id="4b675-707">Stale DNS problems by cycling `HttpMessageHandler` instances at regular intervals.</span></span>

<span data-ttu-id="4b675-708">Esistono modi alternativi per risolvere i problemi precedenti utilizzando un'istanza di lunga durata <xref:System.Net.Http.SocketsHttpHandler> .</span><span class="sxs-lookup"><span data-stu-id="4b675-708">There are alternative ways to solve the preceding problems using a long-lived <xref:System.Net.Http.SocketsHttpHandler> instance.</span></span>

- <span data-ttu-id="4b675-709">Creare un'istanza di all' `SocketsHttpHandler` avvio dell'app e usarla per il ciclo di vita dell'app.</span><span class="sxs-lookup"><span data-stu-id="4b675-709">Create an instance of `SocketsHttpHandler` when the app starts and use it for the life of the app.</span></span>
- <span data-ttu-id="4b675-710">Configurare su <xref:System.Net.Http.SocketsHttpHandler.PooledConnectionLifetime> un valore appropriato in base alle ore di aggiornamento DNS.</span><span class="sxs-lookup"><span data-stu-id="4b675-710">Configure <xref:System.Net.Http.SocketsHttpHandler.PooledConnectionLifetime> to an appropriate value based on DNS refresh times.</span></span>
- <span data-ttu-id="4b675-711">Creare `HttpClient` istanze `new HttpClient(handler, disposeHandler: false)` di usando in base alle esigenze.</span><span class="sxs-lookup"><span data-stu-id="4b675-711">Create `HttpClient` instances using `new HttpClient(handler, disposeHandler: false)` as needed.</span></span>

<span data-ttu-id="4b675-712">Gli approcci precedenti risolvono i problemi di gestione delle risorse che vengono `IHttpClientFactory` risolti in modo analogo.</span><span class="sxs-lookup"><span data-stu-id="4b675-712">The preceding approaches solve the resource management problems that `IHttpClientFactory` solves in a similar way.</span></span>

- <span data-ttu-id="4b675-713">Il `SocketsHttpHandler` condivide le connessioni tra le `HttpClient` istanze.</span><span class="sxs-lookup"><span data-stu-id="4b675-713">The `SocketsHttpHandler` shares connections across `HttpClient` instances.</span></span> <span data-ttu-id="4b675-714">Questa condivisione impedisce l'esaurimento del socket.</span><span class="sxs-lookup"><span data-stu-id="4b675-714">This sharing prevents socket exhaustion.</span></span>
- <span data-ttu-id="4b675-715">Il `SocketsHttpHandler` ciclo di connessioni in base a `PooledConnectionLifetime` per evitare problemi DNS non aggiornati.</span><span class="sxs-lookup"><span data-stu-id="4b675-715">The `SocketsHttpHandler` cycles connections according to `PooledConnectionLifetime` to avoid stale DNS problems.</span></span>

### <a name="cookies"></a><span data-ttu-id="4b675-716">Cookies</span><span class="sxs-lookup"><span data-stu-id="4b675-716">Cookies</span></span>

<span data-ttu-id="4b675-717">Le `HttpMessageHandler` istanze in pool generano `CookieContainer` oggetti condivisi.</span><span class="sxs-lookup"><span data-stu-id="4b675-717">The pooled `HttpMessageHandler` instances results in `CookieContainer` objects being shared.</span></span> <span data-ttu-id="4b675-718">La `CookieContainer` condivisione di oggetti imprevista spesso genera codice errato.</span><span class="sxs-lookup"><span data-stu-id="4b675-718">Unanticipated `CookieContainer` object sharing often results in incorrect code.</span></span> <span data-ttu-id="4b675-719">Per le app che richiedono cookie s, prendere in considerazione quanto segue:</span><span class="sxs-lookup"><span data-stu-id="4b675-719">For apps that require cookies, consider either:</span></span>

 - <span data-ttu-id="4b675-720">Disabilitazione della cookie gestione automatica</span><span class="sxs-lookup"><span data-stu-id="4b675-720">Disabling automatic cookie handling</span></span>
 - <span data-ttu-id="4b675-721">Evitando `IHttpClientFactory`</span><span class="sxs-lookup"><span data-stu-id="4b675-721">Avoiding `IHttpClientFactory`</span></span>

<span data-ttu-id="4b675-722">Chiamare <xref:Microsoft.Extensions.DependencyInjection.HttpClientBuilderExtensions.ConfigurePrimaryHttpMessageHandler*> per disabilitare la cookie gestione automatica:</span><span class="sxs-lookup"><span data-stu-id="4b675-722">Call <xref:Microsoft.Extensions.DependencyInjection.HttpClientBuilderExtensions.ConfigurePrimaryHttpMessageHandler*> to disable automatic cookie handling:</span></span>

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet13)]

## <a name="logging"></a><span data-ttu-id="4b675-723">Registrazione</span><span class="sxs-lookup"><span data-stu-id="4b675-723">Logging</span></span>

<span data-ttu-id="4b675-724">I client creati tramite `IHttpClientFactory` registrano i messaggi di log per tutte le richieste.</span><span class="sxs-lookup"><span data-stu-id="4b675-724">Clients created via `IHttpClientFactory` record log messages for all requests.</span></span> <span data-ttu-id="4b675-725">Per visualizzare i messaggi di log predefiniti, abilitare il livello di informazioni appropriato nella configurazione di registrazione.</span><span class="sxs-lookup"><span data-stu-id="4b675-725">Enable the appropriate information level in your logging configuration to see the default log messages.</span></span> <span data-ttu-id="4b675-726">La registrazione aggiuntiva, ad esempio quella delle intestazioni delle richieste, è inclusa solo a livello di traccia.</span><span class="sxs-lookup"><span data-stu-id="4b675-726">Additional logging, such as the logging of request headers, is only included at trace level.</span></span>

<span data-ttu-id="4b675-727">La categoria di log usata per ogni client include il nome del client.</span><span class="sxs-lookup"><span data-stu-id="4b675-727">The log category used for each client includes the name of the client.</span></span> <span data-ttu-id="4b675-728">Un client denominato *MyNamedClient*, ad esempio, registra i messaggi con una categoria `System.Net.Http.HttpClient.MyNamedClient.LogicalHandler`.</span><span class="sxs-lookup"><span data-stu-id="4b675-728">A client named *MyNamedClient*, for example, logs messages with a category of `System.Net.Http.HttpClient.MyNamedClient.LogicalHandler`.</span></span> <span data-ttu-id="4b675-729">I messaggi con suffisso *LogicalHandler* sono esterni alla pipeline del gestore delle richieste.</span><span class="sxs-lookup"><span data-stu-id="4b675-729">Messages suffixed with *LogicalHandler* occur outside the request handler pipeline.</span></span> <span data-ttu-id="4b675-730">Nella richiesta i messaggi vengono registrati prima che qualsiasi altro gestore nella pipeline l'abbia elaborata.</span><span class="sxs-lookup"><span data-stu-id="4b675-730">On the request, messages are logged before any other handlers in the pipeline have processed it.</span></span> <span data-ttu-id="4b675-731">Nella risposta i messaggi vengono registrati dopo che qualsiasi altro gestore nella pipeline ha ricevuto la risposta.</span><span class="sxs-lookup"><span data-stu-id="4b675-731">On the response, messages are logged after any other pipeline handlers have received the response.</span></span>

<span data-ttu-id="4b675-732">La registrazione avviene anche all'interno della pipeline del gestore delle richieste.</span><span class="sxs-lookup"><span data-stu-id="4b675-732">Logging also occurs inside the request handler pipeline.</span></span> <span data-ttu-id="4b675-733">Nell'esempio *MyNamedClient*, i messaggi vengono registrati nella categoria di log `System.Net.Http.HttpClient.MyNamedClient.ClientHandler`.</span><span class="sxs-lookup"><span data-stu-id="4b675-733">In the *MyNamedClient* example, those messages are logged against the log category `System.Net.Http.HttpClient.MyNamedClient.ClientHandler`.</span></span> <span data-ttu-id="4b675-734">Per la richiesta, ciò avviene dopo l'esecuzione di tutti gli altri gestori e immediatamente prima che la richiesta sia inviata in rete.</span><span class="sxs-lookup"><span data-stu-id="4b675-734">For the request, this occurs after all other handlers have run and immediately before the request is sent out on the network.</span></span> <span data-ttu-id="4b675-735">Nella risposta la registrazione include lo stato della risposta prima di restituirla attraverso la pipeline del gestore.</span><span class="sxs-lookup"><span data-stu-id="4b675-735">On the response, this logging includes the state of the response before it passes back through the handler pipeline.</span></span>

<span data-ttu-id="4b675-736">L'abilitazione della registrazione all'esterno e all'interno della pipeline consente l'ispezione delle modifiche apportate da altri gestori nella pipeline.</span><span class="sxs-lookup"><span data-stu-id="4b675-736">Enabling logging outside and inside the pipeline enables inspection of the changes made by the other pipeline handlers.</span></span> <span data-ttu-id="4b675-737">Le modifiche possono ad esempio riguardare le intestazioni delle richieste o il codice di stato della risposta.</span><span class="sxs-lookup"><span data-stu-id="4b675-737">This may include changes to request headers, for example, or to the response status code.</span></span>

<span data-ttu-id="4b675-738">L'inclusione del nome del client nella categoria di log consente di filtrare i log in base a client denominati specifici, se necessario.</span><span class="sxs-lookup"><span data-stu-id="4b675-738">Including the name of the client in the log category enables log filtering for specific named clients where necessary.</span></span>

## <a name="configure-the-httpmessagehandler"></a><span data-ttu-id="4b675-739">Configurare HttpMessageHandler</span><span class="sxs-lookup"><span data-stu-id="4b675-739">Configure the HttpMessageHandler</span></span>

<span data-ttu-id="4b675-740">Può essere necessario controllare la configurazione dell'elemento `HttpMessageHandler` interno usato da un client.</span><span class="sxs-lookup"><span data-stu-id="4b675-740">It may be necessary to control the configuration of the inner `HttpMessageHandler` used by a client.</span></span>

<span data-ttu-id="4b675-741">Quando si aggiungono client denominati o tipizzati viene restituito un elemento `IHttpClientBuilder`.</span><span class="sxs-lookup"><span data-stu-id="4b675-741">An `IHttpClientBuilder` is returned when adding named or typed clients.</span></span> <span data-ttu-id="4b675-742">È possibile usare il metodo di estensione <xref:Microsoft.Extensions.DependencyInjection.HttpClientBuilderExtensions.ConfigurePrimaryHttpMessageHandler*> per definire un delegato.</span><span class="sxs-lookup"><span data-stu-id="4b675-742">The <xref:Microsoft.Extensions.DependencyInjection.HttpClientBuilderExtensions.ConfigurePrimaryHttpMessageHandler*> extension method can be used to define a delegate.</span></span> <span data-ttu-id="4b675-743">Il delegato viene usato per creare e configurare l'elemento `HttpMessageHandler` primario usato dal client:</span><span class="sxs-lookup"><span data-stu-id="4b675-743">The delegate is used to create and configure the primary `HttpMessageHandler` used by that client:</span></span>

[!code-csharp[](http-requests/samples/2.x/HttpClientFactorySample/Startup.cs?name=snippet12)]

## <a name="use-ihttpclientfactory-in-a-console-app"></a><span data-ttu-id="4b675-744">Usare IHttpClientFactory in un'app console</span><span class="sxs-lookup"><span data-stu-id="4b675-744">Use IHttpClientFactory in a console app</span></span>

<span data-ttu-id="4b675-745">In un'app console aggiungere al progetto i riferimenti ai pacchetti seguenti:</span><span class="sxs-lookup"><span data-stu-id="4b675-745">In a console app, add the following package references to the project:</span></span>

* [<span data-ttu-id="4b675-746">Microsoft.Extensions.Hosting</span><span class="sxs-lookup"><span data-stu-id="4b675-746">Microsoft.Extensions.Hosting</span></span>](https://www.nuget.org/packages/Microsoft.Extensions.Hosting)
* [<span data-ttu-id="4b675-747">Microsoft.Extensions.Http</span><span class="sxs-lookup"><span data-stu-id="4b675-747">Microsoft.Extensions.Http</span></span>](https://www.nuget.org/packages/Microsoft.Extensions.Http)

<span data-ttu-id="4b675-748">Nell'esempio seguente:</span><span class="sxs-lookup"><span data-stu-id="4b675-748">In the following example:</span></span>

* <span data-ttu-id="4b675-749"><xref:System.Net.Http.IHttpClientFactory> è registrato nel contenitore di servizi dell'[host generico](xref:fundamentals/host/generic-host).</span><span class="sxs-lookup"><span data-stu-id="4b675-749"><xref:System.Net.Http.IHttpClientFactory> is registered in the [Generic Host's](xref:fundamentals/host/generic-host) service container.</span></span>
* <span data-ttu-id="4b675-750">`MyService` crea un'istanza della factory client dal servizio, che viene usata per creare una classe `HttpClient`.</span><span class="sxs-lookup"><span data-stu-id="4b675-750">`MyService` creates a client factory instance from the service, which is used to create an `HttpClient`.</span></span> <span data-ttu-id="4b675-751">`HttpClient` viene usato per recuperare una pagina Web.</span><span class="sxs-lookup"><span data-stu-id="4b675-751">`HttpClient` is used to retrieve a webpage.</span></span>
* <span data-ttu-id="4b675-752">`Main` crea un ambito per eseguire il metodo `GetPage` del servizio e scrivere i primi 500 caratteri del contenuto della pagina Web nella console.</span><span class="sxs-lookup"><span data-stu-id="4b675-752">`Main` creates a scope to execute the service's `GetPage` method and write the first 500 characters of the webpage content to the console.</span></span>

[!code-csharp[](http-requests/samples/2.x/HttpClientFactoryConsoleSample/Program.cs?highlight=14-15,20,26-27,59-62)]

## <a name="header-propagation-middleware"></a><span data-ttu-id="4b675-753">Middleware di propagazione delle intestazioni</span><span class="sxs-lookup"><span data-stu-id="4b675-753">Header propagation middleware</span></span>

<span data-ttu-id="4b675-754">La propagazione dell'intestazione è un middleware supportato dalla community per propagare le intestazioni HTTP dalla richiesta in ingresso alle richieste del client HTTP in uscita.</span><span class="sxs-lookup"><span data-stu-id="4b675-754">Header propagation is a community supported middleware to propagate HTTP headers from the incoming request to the outgoing HTTP Client requests.</span></span> <span data-ttu-id="4b675-755">Per usare la propagazione delle intestazioni:</span><span class="sxs-lookup"><span data-stu-id="4b675-755">To use header propagation:</span></span>

* <span data-ttu-id="4b675-756">Fare riferimento alla porta supportata dalla community del pacchetto [HeaderPropagation](https://www.nuget.org/packages/HeaderPropagation).</span><span class="sxs-lookup"><span data-stu-id="4b675-756">Reference the community supported port of the package [HeaderPropagation](https://www.nuget.org/packages/HeaderPropagation).</span></span> <span data-ttu-id="4b675-757">ASP.NET Core 3,1 e versioni successive supporta [Microsoft. AspNetCore. HeaderPropagation](https://www.nuget.org/packages/Microsoft.AspNetCore.HeaderPropagation).</span><span class="sxs-lookup"><span data-stu-id="4b675-757">ASP.NET Core 3.1 and later supports [Microsoft.AspNetCore.HeaderPropagation](https://www.nuget.org/packages/Microsoft.AspNetCore.HeaderPropagation).</span></span>

* <span data-ttu-id="4b675-758">Configurare il middleware e `HttpClient` in `Startup` :</span><span class="sxs-lookup"><span data-stu-id="4b675-758">Configure the middleware and `HttpClient` in `Startup`:</span></span>

  [!code-csharp[](http-requests/samples/2.x/Startup21.cs?highlight=5-9,25&name=snippet)]

* <span data-ttu-id="4b675-759">Il client include le intestazioni configurate nelle richieste in uscita:</span><span class="sxs-lookup"><span data-stu-id="4b675-759">The client includes the configured headers on outbound requests:</span></span>

  ```csharp
  var client = clientFactory.CreateClient("MyForwardingClient");
  var response = client.GetAsync(...);
  ```

## <a name="additional-resources"></a><span data-ttu-id="4b675-760">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="4b675-760">Additional resources</span></span>

* [<span data-ttu-id="4b675-761">Usare HttpClientFactory per l'implementazione di richieste HTTP resilienti</span><span class="sxs-lookup"><span data-stu-id="4b675-761">Use HttpClientFactory to implement resilient HTTP requests</span></span>](/dotnet/standard/microservices-architecture/implement-resilient-applications/use-httpclientfactory-to-implement-resilient-http-requests)
* [<span data-ttu-id="4b675-762">Implementazione dei tentativi di chiamate HTTP con backoff esponenziale con i criteri di Polly e HttpClientFactory</span><span class="sxs-lookup"><span data-stu-id="4b675-762">Implement HTTP call retries with exponential backoff with HttpClientFactory and Polly policies</span></span>](/dotnet/standard/microservices-architecture/implement-resilient-applications/implement-http-call-retries-exponential-backoff-polly)
* [<span data-ttu-id="4b675-763">Implementazione dello schema Circuit Breaker</span><span class="sxs-lookup"><span data-stu-id="4b675-763">Implement the Circuit Breaker pattern</span></span>](/dotnet/standard/microservices-architecture/implement-resilient-applications/implement-circuit-breaker-pattern)

::: moniker-end
