---
title: BlazorInserimento di dipendenze ASP.NET Core
author: guardrex
description: Informazioni su come le Blazor app possono inserire i servizi nei componenti.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 12/19/2020
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
uid: blazor/fundamentals/dependency-injection
zone_pivot_groups: blazor-hosting-models
ms.openlocfilehash: aefeac2f3a68a669b7c84cbeee2f6a4ec0621f6f
ms.sourcegitcommit: a1db01b4d3bd8c57d7a9c94ce122a6db68002d66
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/04/2021
ms.locfileid: "102109676"
---
# <a name="aspnet-core-blazor-dependency-injection"></a><span data-ttu-id="76459-103">BlazorInserimento di dipendenze ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="76459-103">ASP.NET Core Blazor dependency injection</span></span>

<span data-ttu-id="76459-104">Di [Rainer Stropek](https://www.timecockpit.com) e [Mike entusiasmanti](https://github.com/mjrousos)</span><span class="sxs-lookup"><span data-stu-id="76459-104">By [Rainer Stropek](https://www.timecockpit.com) and [Mike Rousos](https://github.com/mjrousos)</span></span>

<span data-ttu-id="76459-105">L' [inserimento di dipendenze](xref:fundamentals/dependency-injection) è una tecnica per accedere ai servizi configurati in una posizione centrale:</span><span class="sxs-lookup"><span data-stu-id="76459-105">[Dependency injection (DI)](xref:fundamentals/dependency-injection) is a technique for accessing services configured in a central location:</span></span>

* <span data-ttu-id="76459-106">I servizi registrati dal Framework possono essere inseriti direttamente in componenti di Blazor app.</span><span class="sxs-lookup"><span data-stu-id="76459-106">Framework-registered services can be injected directly into components of Blazor apps.</span></span>
* <span data-ttu-id="76459-107">Blazor le app definiscono e registrano servizi personalizzati e le rendono disponibili nell'app tramite DI.</span><span class="sxs-lookup"><span data-stu-id="76459-107">Blazor apps define and register custom services and make them available throughout the app via DI.</span></span>

## <a name="default-services"></a><span data-ttu-id="76459-108">Servizi predefiniti</span><span class="sxs-lookup"><span data-stu-id="76459-108">Default services</span></span>

<span data-ttu-id="76459-109">I servizi illustrati nella tabella seguente sono comunemente usati nelle Blazor app.</span><span class="sxs-lookup"><span data-stu-id="76459-109">The services shown in the following table are commonly used in Blazor apps.</span></span>

| <span data-ttu-id="76459-110">Servizio</span><span class="sxs-lookup"><span data-stu-id="76459-110">Service</span></span> | <span data-ttu-id="76459-111">Durata</span><span class="sxs-lookup"><span data-stu-id="76459-111">Lifetime</span></span> | <span data-ttu-id="76459-112">Descrizione</span><span class="sxs-lookup"><span data-stu-id="76459-112">Description</span></span> |
| ------- | -------- | ----------- |
| <xref:System.Net.Http.HttpClient> | <span data-ttu-id="76459-113">Con ambito</span><span class="sxs-lookup"><span data-stu-id="76459-113">Scoped</span></span> | <p><span data-ttu-id="76459-114">Fornisce metodi per l'invio di richieste HTTP e la ricezione di risposte HTTP da una risorsa identificata da un URI.</span><span class="sxs-lookup"><span data-stu-id="76459-114">Provides methods for sending HTTP requests and receiving HTTP responses from a resource identified by a URI.</span></span></p><p><span data-ttu-id="76459-115">L'istanza di <xref:System.Net.Http.HttpClient> in un' Blazor WebAssembly applicazione utilizza il browser per gestire il traffico HTTP in background.</span><span class="sxs-lookup"><span data-stu-id="76459-115">The instance of <xref:System.Net.Http.HttpClient> in a Blazor WebAssembly app uses the browser for handling the HTTP traffic in the background.</span></span></p><p><span data-ttu-id="76459-116">Blazor Server per impostazione predefinita, le app non includono un <xref:System.Net.Http.HttpClient> configurato come servizio.</span><span class="sxs-lookup"><span data-stu-id="76459-116">Blazor Server apps don't include an <xref:System.Net.Http.HttpClient> configured as a service by default.</span></span> <span data-ttu-id="76459-117">Fornire un <xref:System.Net.Http.HttpClient> a un' Blazor Server app.</span><span class="sxs-lookup"><span data-stu-id="76459-117">Provide an <xref:System.Net.Http.HttpClient> to a Blazor Server app.</span></span></p><p><span data-ttu-id="76459-118">Per altre informazioni, vedere <xref:blazor/call-web-api>.</span><span class="sxs-lookup"><span data-stu-id="76459-118">For more information, see <xref:blazor/call-web-api>.</span></span></p><p><span data-ttu-id="76459-119">Un <xref:System.Net.Http.HttpClient> viene registrato come servizio con ambito, non come singleton.</span><span class="sxs-lookup"><span data-stu-id="76459-119">An <xref:System.Net.Http.HttpClient> is registered as a scoped service, not singleton.</span></span> <span data-ttu-id="76459-120">Per ulteriori informazioni, vedere la sezione [durata del servizio](#service-lifetime) .</span><span class="sxs-lookup"><span data-stu-id="76459-120">For more information, see the [Service lifetime](#service-lifetime) section.</span></span></p> |
| <xref:Microsoft.JSInterop.IJSRuntime> | <p><span data-ttu-id="76459-121">**Blazor WebAssembly**: Singleton</span><span class="sxs-lookup"><span data-stu-id="76459-121">**Blazor WebAssembly**: Singleton</span></span></p><p><span data-ttu-id="76459-122">**Blazor Server**: Con ambito</span><span class="sxs-lookup"><span data-stu-id="76459-122">**Blazor Server**: Scoped</span></span></p> | <span data-ttu-id="76459-123">Rappresenta un'istanza di un runtime JavaScript in cui vengono inviate le chiamate a JavaScript.</span><span class="sxs-lookup"><span data-stu-id="76459-123">Represents an instance of a JavaScript runtime where JavaScript calls are dispatched.</span></span> <span data-ttu-id="76459-124">Per altre informazioni, vedere <xref:blazor/call-javascript-from-dotnet>.</span><span class="sxs-lookup"><span data-stu-id="76459-124">For more information, see <xref:blazor/call-javascript-from-dotnet>.</span></span> |
| <xref:Microsoft.AspNetCore.Components.NavigationManager> | <p><span data-ttu-id="76459-125">**Blazor WebAssembly**: Singleton</span><span class="sxs-lookup"><span data-stu-id="76459-125">**Blazor WebAssembly**: Singleton</span></span></p><p><span data-ttu-id="76459-126">**Blazor Server**: Con ambito</span><span class="sxs-lookup"><span data-stu-id="76459-126">**Blazor Server**: Scoped</span></span></p> | <span data-ttu-id="76459-127">Contiene gli helper per lavorare con gli URI e lo stato di navigazione.</span><span class="sxs-lookup"><span data-stu-id="76459-127">Contains helpers for working with URIs and navigation state.</span></span> <span data-ttu-id="76459-128">Per ulteriori informazioni, vedere [URI e Helper dello stato di navigazione](xref:blazor/fundamentals/routing#uri-and-navigation-state-helpers).</span><span class="sxs-lookup"><span data-stu-id="76459-128">For more information, see [URI and navigation state helpers](xref:blazor/fundamentals/routing#uri-and-navigation-state-helpers).</span></span> |

<span data-ttu-id="76459-129">Un provider di servizi personalizzato non fornisce automaticamente i servizi predefiniti elencati nella tabella.</span><span class="sxs-lookup"><span data-stu-id="76459-129">A custom service provider doesn't automatically provide the default services listed in the table.</span></span> <span data-ttu-id="76459-130">Se si usa un provider di servizi personalizzato e si richiede uno dei servizi indicati nella tabella, aggiungere i servizi necessari al nuovo provider di servizi.</span><span class="sxs-lookup"><span data-stu-id="76459-130">If you use a custom service provider and require any of the services shown in the table, add the required services to the new service provider.</span></span>

## <a name="add-services-to-an-app"></a><span data-ttu-id="76459-131">Aggiungere servizi a un'app</span><span class="sxs-lookup"><span data-stu-id="76459-131">Add services to an app</span></span>

::: zone pivot="webassembly"

<span data-ttu-id="76459-132">Configurare i servizi per la raccolta di servizi dell'app nel `Program.Main` metodo di `Program.cs` .</span><span class="sxs-lookup"><span data-stu-id="76459-132">Configure services for the app's service collection in the `Program.Main` method of `Program.cs`.</span></span> <span data-ttu-id="76459-133">Nell'esempio seguente l' `MyDependency` implementazione è registrata per `IMyDependency` :</span><span class="sxs-lookup"><span data-stu-id="76459-133">In the following example, the `MyDependency` implementation is registered for `IMyDependency`:</span></span>

```csharp
public class Program
{
    public static async Task Main(string[] args)
    {
        var builder = WebAssemblyHostBuilder.CreateDefault(args);
        ...
        builder.Services.AddSingleton<IMyDependency, MyDependency>();
        ...

        await builder.Build().RunAsync();
    }
}
```

<span data-ttu-id="76459-134">Al termine della compilazione dell'host, i servizi sono disponibili nell'ambito radice prima DI eseguire il rendering dei componenti.</span><span class="sxs-lookup"><span data-stu-id="76459-134">After the host is built, services are available from the root DI scope before any components are rendered.</span></span> <span data-ttu-id="76459-135">Questa operazione può essere utile per eseguire la logica di inizializzazione prima del rendering del contenuto:</span><span class="sxs-lookup"><span data-stu-id="76459-135">This can be useful for running initialization logic before rendering content:</span></span>

```csharp
public class Program
{
    public static async Task Main(string[] args)
    {
        var builder = WebAssemblyHostBuilder.CreateDefault(args);
        ...
        builder.Services.AddSingleton<WeatherService>();
        ...

        var host = builder.Build();

        var weatherService = host.Services.GetRequiredService<WeatherService>();
        await weatherService.InitializeWeatherAsync();

        await host.RunAsync();
    }
}
```

<span data-ttu-id="76459-136">L'host fornisce un'istanza di configurazione centrale per l'app.</span><span class="sxs-lookup"><span data-stu-id="76459-136">The host provides a central configuration instance for the app.</span></span> <span data-ttu-id="76459-137">Basandosi sull'esempio precedente, l'URL del servizio meteo viene passato da un'origine di configurazione predefinita (ad esempio, `appsettings.json` ) a `InitializeWeatherAsync` :</span><span class="sxs-lookup"><span data-stu-id="76459-137">Building on the preceding example, the weather service's URL is passed from a default configuration source (for example, `appsettings.json`) to `InitializeWeatherAsync`:</span></span>

```csharp
public class Program
{
    public static async Task Main(string[] args)
    {
        var builder = WebAssemblyHostBuilder.CreateDefault(args);
        ...
        builder.Services.AddSingleton<WeatherService>();
        ...

        var host = builder.Build();

        var weatherService = host.Services.GetRequiredService<WeatherService>();
        await weatherService.InitializeWeatherAsync(
            host.Configuration["WeatherServiceUrl"]);

        await host.RunAsync();
    }
}
```

::: zone-end

::: zone pivot="server"

<span data-ttu-id="76459-138">Dopo aver creato una nuova app, esaminare il `Startup.ConfigureServices` metodo in `Startup.cs` :</span><span class="sxs-lookup"><span data-stu-id="76459-138">After creating a new app, examine the `Startup.ConfigureServices` method in `Startup.cs`:</span></span>

```csharp
using Microsoft.Extensions.DependencyInjection;

...

public void ConfigureServices(IServiceCollection services)
{
    ...
}
```

<span data-ttu-id="76459-139">Al <xref:Microsoft.Extensions.Hosting.IHostBuilder.ConfigureServices%2A> metodo viene passato un oggetto <xref:Microsoft.Extensions.DependencyInjection.IServiceCollection> , che è un elenco di oggetti del [descrittore del servizio](xref:Microsoft.Extensions.DependencyInjection.ServiceDescriptor) .</span><span class="sxs-lookup"><span data-stu-id="76459-139">The <xref:Microsoft.Extensions.Hosting.IHostBuilder.ConfigureServices%2A> method is passed an <xref:Microsoft.Extensions.DependencyInjection.IServiceCollection>, which is a list of [service descriptor](xref:Microsoft.Extensions.DependencyInjection.ServiceDescriptor) objects.</span></span> <span data-ttu-id="76459-140">I servizi vengono aggiunti nel `ConfigureServices` Metodo fornendo descrittori del servizio alla raccolta di servizi.</span><span class="sxs-lookup"><span data-stu-id="76459-140">Services are added in the `ConfigureServices` method by providing service descriptors to the service collection.</span></span> <span data-ttu-id="76459-141">Nell'esempio seguente viene illustrato il concetto con l' `IDataAccess` interfaccia e la relativa implementazione concreta `DataAccess` :</span><span class="sxs-lookup"><span data-stu-id="76459-141">The following example demonstrates the concept with the `IDataAccess` interface and its concrete implementation `DataAccess`:</span></span>

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddSingleton<IDataAccess, DataAccess>();
}
```

::: zone-end

### <a name="service-lifetime"></a><span data-ttu-id="76459-142">Durata del servizio</span><span class="sxs-lookup"><span data-stu-id="76459-142">Service lifetime</span></span>

<span data-ttu-id="76459-143">I servizi possono essere configurati con le durate mostrate nella tabella seguente.</span><span class="sxs-lookup"><span data-stu-id="76459-143">Services can be configured with the lifetimes shown in the following table.</span></span>

| <span data-ttu-id="76459-144">Durata</span><span class="sxs-lookup"><span data-stu-id="76459-144">Lifetime</span></span> | <span data-ttu-id="76459-145">Descrizione</span><span class="sxs-lookup"><span data-stu-id="76459-145">Description</span></span> |
| -------- | ----------- |
| <xref:Microsoft.Extensions.DependencyInjection.ServiceDescriptor.Scoped%2A> | <p><span data-ttu-id="76459-146">Blazor WebAssembly Attualmente le app non hanno un concetto di ambiti di.</span><span class="sxs-lookup"><span data-stu-id="76459-146">Blazor WebAssembly apps don't currently have a concept of DI scopes.</span></span> <span data-ttu-id="76459-147">`Scoped`-i servizi registrati si comportano come `Singleton` servizi.</span><span class="sxs-lookup"><span data-stu-id="76459-147">`Scoped`-registered services behave like `Singleton` services.</span></span></p><p><span data-ttu-id="76459-148">Il Blazor Server modello di hosting supporta la `Scoped` durata tra le richieste HTTP, ma non tra SignalR i messaggi di connessione/circuito tra i componenti caricati nel client.</span><span class="sxs-lookup"><span data-stu-id="76459-148">The Blazor Server hosting model supports the `Scoped` lifetime across HTTP requests but not across SignalR connection/circuit messages among components that are loaded on the client.</span></span> <span data-ttu-id="76459-149">Le Razor pagine o la parte MVC dell'app considera i servizi con ambito normalmente e ricrea i servizi in *ogni richiesta http* quando si naviga tra pagine o viste oppure da una pagina o da una vista a un componente.</span><span class="sxs-lookup"><span data-stu-id="76459-149">The Razor Pages or MVC portion of the app treats scoped services normally and recreates the services on *each HTTP request* when navigating among pages or views or from a page or view to a component.</span></span> <span data-ttu-id="76459-150">I servizi con ambito non vengono ricostruiti quando ci si sposta tra i componenti nel client, dove la comunicazione con il server avviene tramite la SignalR connessione del circuito dell'utente, non tramite richieste HTTP.</span><span class="sxs-lookup"><span data-stu-id="76459-150">Scoped services aren't reconstructed when navigating among components on the client, where the communication to the server takes place over the SignalR connection of the user's circuit, not via HTTP requests.</span></span> <span data-ttu-id="76459-151">Negli scenari dei componenti seguenti nel client, i servizi con ambito vengono ricostruiti perché viene creato un nuovo circuito per l'utente:</span><span class="sxs-lookup"><span data-stu-id="76459-151">In the following component scenarios on the client, scoped services are reconstructed because a new circuit is created for the user:</span></span></p><ul><li><span data-ttu-id="76459-152">L'utente chiude la finestra del browser.</span><span class="sxs-lookup"><span data-stu-id="76459-152">The user closes the browser's window.</span></span> <span data-ttu-id="76459-153">L'utente apre una nuova finestra e torna all'app.</span><span class="sxs-lookup"><span data-stu-id="76459-153">The user opens a new window and navigates back to the app.</span></span></li><li><span data-ttu-id="76459-154">L'utente chiude l'ultima scheda dell'app in una finestra del browser.</span><span class="sxs-lookup"><span data-stu-id="76459-154">The user closes the last tab of the app in a browser window.</span></span> <span data-ttu-id="76459-155">L'utente apre una nuova scheda e torna all'app.</span><span class="sxs-lookup"><span data-stu-id="76459-155">The user opens a new tab and navigates back to the app.</span></span></li><li><span data-ttu-id="76459-156">L'utente seleziona il pulsante ricarica/Aggiorna del browser.</span><span class="sxs-lookup"><span data-stu-id="76459-156">The user selects the browser's reload/refresh button.</span></span></li></ul><p><span data-ttu-id="76459-157">Per ulteriori informazioni sul mantenimento dello stato utente tra i servizi con ambito nelle Blazor Server app, vedere <xref:blazor/hosting-models?pivots=server> .</span><span class="sxs-lookup"><span data-stu-id="76459-157">For more information on preserving user state across scoped services in Blazor Server apps, see <xref:blazor/hosting-models?pivots=server>.</span></span></p> |
| <xref:Microsoft.Extensions.DependencyInjection.ServiceDescriptor.Singleton%2A> | <span data-ttu-id="76459-158">La creazione di una *singola istanza* del servizio.</span><span class="sxs-lookup"><span data-stu-id="76459-158">DI creates a *single instance* of the service.</span></span> <span data-ttu-id="76459-159">Tutti i componenti che richiedono un `Singleton` servizio ricevono un'istanza dello stesso servizio.</span><span class="sxs-lookup"><span data-stu-id="76459-159">All components requiring a `Singleton` service receive an instance of the same service.</span></span> |
| <xref:Microsoft.Extensions.DependencyInjection.ServiceDescriptor.Transient%2A> | <span data-ttu-id="76459-160">Ogni volta che un componente ottiene un'istanza di un `Transient` servizio dal contenitore del servizio, riceve una *nuova istanza* del servizio.</span><span class="sxs-lookup"><span data-stu-id="76459-160">Whenever a component obtains an instance of a `Transient` service from the service container, it receives a *new instance* of the service.</span></span> |

<span data-ttu-id="76459-161">Il sistema DI è basato sul sistema DI ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="76459-161">The DI system is based on the DI system in ASP.NET Core.</span></span> <span data-ttu-id="76459-162">Per altre informazioni, vedere <xref:fundamentals/dependency-injection>.</span><span class="sxs-lookup"><span data-stu-id="76459-162">For more information, see <xref:fundamentals/dependency-injection>.</span></span>

## <a name="request-a-service-in-a-component"></a><span data-ttu-id="76459-163">Richiedere un servizio in un componente</span><span class="sxs-lookup"><span data-stu-id="76459-163">Request a service in a component</span></span>

<span data-ttu-id="76459-164">Una volta aggiunti i servizi alla raccolta di servizi, inserire i servizi nei componenti usando la [`@inject`](xref:mvc/views/razor#inject) Razor direttiva, che ha due parametri:</span><span class="sxs-lookup"><span data-stu-id="76459-164">After services are added to the service collection, inject the services into the components using the [`@inject`](xref:mvc/views/razor#inject) Razor directive, which has two parameters:</span></span>

* <span data-ttu-id="76459-165">Tipo: tipo di servizio da inserire.</span><span class="sxs-lookup"><span data-stu-id="76459-165">Type: The type of the service to inject.</span></span>
* <span data-ttu-id="76459-166">Property: nome della proprietà che riceve il servizio app inserito.</span><span class="sxs-lookup"><span data-stu-id="76459-166">Property: The name of the property receiving the injected app service.</span></span> <span data-ttu-id="76459-167">La proprietà non richiede la creazione manuale.</span><span class="sxs-lookup"><span data-stu-id="76459-167">The property doesn't require manual creation.</span></span> <span data-ttu-id="76459-168">Il compilatore crea la proprietà.</span><span class="sxs-lookup"><span data-stu-id="76459-168">The compiler creates the property.</span></span>

<span data-ttu-id="76459-169">Per altre informazioni, vedere <xref:mvc/views/dependency-injection>.</span><span class="sxs-lookup"><span data-stu-id="76459-169">For more information, see <xref:mvc/views/dependency-injection>.</span></span>

<span data-ttu-id="76459-170">Usare più [`@inject`](xref:mvc/views/razor#inject) istruzioni per inserire servizi diversi.</span><span class="sxs-lookup"><span data-stu-id="76459-170">Use multiple [`@inject`](xref:mvc/views/razor#inject) statements to inject different services.</span></span>

<span data-ttu-id="76459-171">Nell'esempio seguente viene illustrato come utilizzare [`@inject`](xref:mvc/views/razor#inject) .</span><span class="sxs-lookup"><span data-stu-id="76459-171">The following example shows how to use [`@inject`](xref:mvc/views/razor#inject).</span></span> <span data-ttu-id="76459-172">Il servizio che implementa `Services.IDataAccess` viene inserito nella proprietà del componente `DataRepository` .</span><span class="sxs-lookup"><span data-stu-id="76459-172">The service implementing `Services.IDataAccess` is injected into the component's property `DataRepository`.</span></span> <span data-ttu-id="76459-173">Si noti come il codice usa solo l' `IDataAccess` astrazione:</span><span class="sxs-lookup"><span data-stu-id="76459-173">Note how the code is only using the `IDataAccess` abstraction:</span></span>

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_Server/Pages/dependency-injection/CustomerList.razor?name=snippet&highlight=2,19)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_Server/Pages/dependency-injection/CustomerList.razor?name=snippet&highlight=2,19)]

::: moniker-end

<span data-ttu-id="76459-174">Internamente, la proprietà generata ( `DataRepository` ) utilizza l' [ `[Inject]` attributo](xref:Microsoft.AspNetCore.Components.InjectAttribute).</span><span class="sxs-lookup"><span data-stu-id="76459-174">Internally, the generated property (`DataRepository`) uses the [`[Inject]` attribute](xref:Microsoft.AspNetCore.Components.InjectAttribute).</span></span> <span data-ttu-id="76459-175">In genere, questo attributo non viene utilizzato direttamente.</span><span class="sxs-lookup"><span data-stu-id="76459-175">Typically, this attribute isn't used directly.</span></span> <span data-ttu-id="76459-176">Se è necessaria una classe base per i componenti e le proprietà inserite sono necessarie anche per la classe base, aggiungere manualmente l' [ `[Inject]` attributo](xref:Microsoft.AspNetCore.Components.InjectAttribute):</span><span class="sxs-lookup"><span data-stu-id="76459-176">If a base class is required for components and injected properties are also required for the base class, manually add the [`[Inject]` attribute](xref:Microsoft.AspNetCore.Components.InjectAttribute):</span></span>

```csharp
using Microsoft.AspNetCore.Components;

public class ComponentBase : IComponent
{
    [Inject]
    protected IDataAccess DataRepository { get; set; }

    ...
}
```

<span data-ttu-id="76459-177">Nei componenti derivati dalla classe di base, la [`@inject`](xref:mvc/views/razor#inject) direttiva non è obbligatoria.</span><span class="sxs-lookup"><span data-stu-id="76459-177">In components derived from the base class, the [`@inject`](xref:mvc/views/razor#inject) directive isn't required.</span></span> <span data-ttu-id="76459-178">La <xref:Microsoft.AspNetCore.Components.InjectAttribute> classe della classe base è sufficiente:</span><span class="sxs-lookup"><span data-stu-id="76459-178">The <xref:Microsoft.AspNetCore.Components.InjectAttribute> of the base class is sufficient:</span></span>

```razor
@page "/demo"
@inherits ComponentBase

<h1>Demo Component</h1>
```

## <a name="use-di-in-services"></a><span data-ttu-id="76459-179">Usare l'inserimento DI dipendenze nei servizi</span><span class="sxs-lookup"><span data-stu-id="76459-179">Use DI in services</span></span>

<span data-ttu-id="76459-180">Servizi complessi potrebbe richiedere servizi aggiuntivi.</span><span class="sxs-lookup"><span data-stu-id="76459-180">Complex services might require additional services.</span></span> <span data-ttu-id="76459-181">Nell'esempio seguente `DataAccess` richiede il <xref:System.Net.Http.HttpClient> servizio predefinito.</span><span class="sxs-lookup"><span data-stu-id="76459-181">In the following example, `DataAccess` requires the <xref:System.Net.Http.HttpClient> default service.</span></span> <span data-ttu-id="76459-182">[`@inject`](xref:mvc/views/razor#inject)(o l' [ `[Inject]` attributo](xref:Microsoft.AspNetCore.Components.InjectAttribute)) non è disponibile per l'uso nei servizi.</span><span class="sxs-lookup"><span data-stu-id="76459-182">[`@inject`](xref:mvc/views/razor#inject) (or the [`[Inject]` attribute](xref:Microsoft.AspNetCore.Components.InjectAttribute)) isn't available for use in services.</span></span> <span data-ttu-id="76459-183">È necessario usare invece l' *inserimento del costruttore* .</span><span class="sxs-lookup"><span data-stu-id="76459-183">*Constructor injection* must be used instead.</span></span> <span data-ttu-id="76459-184">I servizi necessari vengono aggiunti aggiungendo parametri al costruttore del servizio.</span><span class="sxs-lookup"><span data-stu-id="76459-184">Required services are added by adding parameters to the service's constructor.</span></span> <span data-ttu-id="76459-185">Quando si crea il servizio, vengono riconosciuti i servizi richiesti nel costruttore e forniti DI conseguenza.</span><span class="sxs-lookup"><span data-stu-id="76459-185">When DI creates the service, it recognizes the services it requires in the constructor and provides them accordingly.</span></span> <span data-ttu-id="76459-186">Nell'esempio seguente, il costruttore riceve una <xref:System.Net.Http.HttpClient> via di.</span><span class="sxs-lookup"><span data-stu-id="76459-186">In the following example, the constructor receives an <xref:System.Net.Http.HttpClient> via DI.</span></span> <span data-ttu-id="76459-187"><xref:System.Net.Http.HttpClient> è un servizio predefinito.</span><span class="sxs-lookup"><span data-stu-id="76459-187"><xref:System.Net.Http.HttpClient> is a default service.</span></span>

```csharp
using System.Net.Http;

public class DataAccess : IDataAccess
{
    public DataAccess(HttpClient http)
    {
        ...
    }
}
```

<span data-ttu-id="76459-188">Prerequisiti per l'inserimento del costruttore:</span><span class="sxs-lookup"><span data-stu-id="76459-188">Prerequisites for constructor injection:</span></span>

* <span data-ttu-id="76459-189">È necessario che esista un costruttore i cui argomenti possono essere tutti soddisfatti da DI.</span><span class="sxs-lookup"><span data-stu-id="76459-189">One constructor must exist whose arguments can all be fulfilled by DI.</span></span> <span data-ttu-id="76459-190">Sono consentiti parametri aggiuntivi non analizzati da DI se specificano i valori predefiniti.</span><span class="sxs-lookup"><span data-stu-id="76459-190">Additional parameters not covered by DI are allowed if they specify default values.</span></span>
* <span data-ttu-id="76459-191">Il costruttore applicabile deve essere `public` .</span><span class="sxs-lookup"><span data-stu-id="76459-191">The applicable constructor must be `public`.</span></span>
* <span data-ttu-id="76459-192">È necessario che esista un costruttore applicabile.</span><span class="sxs-lookup"><span data-stu-id="76459-192">One applicable constructor must exist.</span></span> <span data-ttu-id="76459-193">In caso di ambiguità, viene generata un'eccezione.</span><span class="sxs-lookup"><span data-stu-id="76459-193">In case of an ambiguity, DI throws an exception.</span></span>

## <a name="utility-base-component-classes-to-manage-a-di-scope"></a><span data-ttu-id="76459-194">Classi di componenti di base dell'utilità per gestire un ambito DI</span><span class="sxs-lookup"><span data-stu-id="76459-194">Utility base component classes to manage a DI scope</span></span>

<span data-ttu-id="76459-195">Nelle app ASP.NET Core, i servizi con ambito hanno in genere come ambito la richiesta corrente.</span><span class="sxs-lookup"><span data-stu-id="76459-195">In ASP.NET Core apps, scoped services are typically scoped to the current request.</span></span> <span data-ttu-id="76459-196">Al termine della richiesta, tutti i servizi con ambito o temporaneo vengono eliminati dal sistema DI.</span><span class="sxs-lookup"><span data-stu-id="76459-196">After the request completes, any scoped or transient services are disposed by the DI system.</span></span> <span data-ttu-id="76459-197">Nelle Blazor Server app, l'ambito della richiesta dura per la durata della connessione client, che può comportare la permanenza di servizi temporanei e con ambito più a lungo del previsto.</span><span class="sxs-lookup"><span data-stu-id="76459-197">In Blazor Server apps, the request scope lasts for the duration of the client connection, which can result in transient and scoped services living much longer than expected.</span></span> <span data-ttu-id="76459-198">Nelle Blazor WebAssembly app i servizi registrati con una durata con ambito vengono considerati come singleton, quindi vivono più a lungo rispetto ai servizi con ambito nelle app ASP.NET Core tipiche.</span><span class="sxs-lookup"><span data-stu-id="76459-198">In Blazor WebAssembly apps, services registered with a scoped lifetime are treated as singletons, so they live longer than scoped services in typical ASP.NET Core apps.</span></span>

> [!NOTE]
> <span data-ttu-id="76459-199">Per rilevare servizi temporanei monouso in un'app, vedere la sezione [rilevare gli eliminabili temporanei](#detect-transient-disposables) .</span><span class="sxs-lookup"><span data-stu-id="76459-199">To detect disposable transient services in an app, see the [Detect transient disposables](#detect-transient-disposables) section.</span></span>

<span data-ttu-id="76459-200">Un approccio che limita la durata di un servizio nelle Blazor app è l'uso del <xref:Microsoft.AspNetCore.Components.OwningComponentBase> tipo.</span><span class="sxs-lookup"><span data-stu-id="76459-200">An approach that limits a service lifetime in Blazor apps is use of the <xref:Microsoft.AspNetCore.Components.OwningComponentBase> type.</span></span> <span data-ttu-id="76459-201"><xref:Microsoft.AspNetCore.Components.OwningComponentBase> è un tipo astratto derivato da <xref:Microsoft.AspNetCore.Components.ComponentBase> che consente di creare un ambito di che corrisponde alla durata del componente.</span><span class="sxs-lookup"><span data-stu-id="76459-201"><xref:Microsoft.AspNetCore.Components.OwningComponentBase> is an abstract type derived from <xref:Microsoft.AspNetCore.Components.ComponentBase> that creates a DI scope corresponding to the lifetime of the component.</span></span> <span data-ttu-id="76459-202">Con questo ambito, è possibile usare i servizi DI i con una durata con ambito e fare in modo che siano attivi fino a quando il componente.</span><span class="sxs-lookup"><span data-stu-id="76459-202">Using this scope, it's possible to use DI services with a scoped lifetime and have them live as long as the component.</span></span> <span data-ttu-id="76459-203">Quando il componente viene eliminato definitivamente, vengono eliminati anche i servizi del provider di servizi con ambito.</span><span class="sxs-lookup"><span data-stu-id="76459-203">When the component is destroyed, services from the component's scoped service provider are disposed as well.</span></span> <span data-ttu-id="76459-204">Questa operazione può essere utile per i servizi che:</span><span class="sxs-lookup"><span data-stu-id="76459-204">This can be useful for services that:</span></span>

* <span data-ttu-id="76459-205">È necessario riutilizzarlo all'interno di un componente, perché la durata temporanea non è appropriata.</span><span class="sxs-lookup"><span data-stu-id="76459-205">Should be reused within a component, as the transient lifetime is inappropriate.</span></span>
* <span data-ttu-id="76459-206">Non devono essere condivise tra i componenti, perché la durata singleton non è appropriata.</span><span class="sxs-lookup"><span data-stu-id="76459-206">Shouldn't be shared across components, as the singleton lifetime is inappropriate.</span></span>

<span data-ttu-id="76459-207">Sono disponibili due versioni del <xref:Microsoft.AspNetCore.Components.OwningComponentBase> tipo:</span><span class="sxs-lookup"><span data-stu-id="76459-207">Two versions of the <xref:Microsoft.AspNetCore.Components.OwningComponentBase> type are available:</span></span>

* <span data-ttu-id="76459-208"><xref:Microsoft.AspNetCore.Components.OwningComponentBase> è un elemento figlio astratto e disposable del <xref:Microsoft.AspNetCore.Components.ComponentBase> tipo con una <xref:Microsoft.AspNetCore.Components.OwningComponentBase.ScopedServices> proprietà protetta di tipo <xref:System.IServiceProvider> .</span><span class="sxs-lookup"><span data-stu-id="76459-208"><xref:Microsoft.AspNetCore.Components.OwningComponentBase> is an abstract, disposable child of the <xref:Microsoft.AspNetCore.Components.ComponentBase> type with a protected <xref:Microsoft.AspNetCore.Components.OwningComponentBase.ScopedServices> property of type <xref:System.IServiceProvider>.</span></span> <span data-ttu-id="76459-209">Questo provider può essere utilizzato per risolvere i servizi che hanno come ambito la durata del componente.</span><span class="sxs-lookup"><span data-stu-id="76459-209">This provider can be used to resolve services that are scoped to the lifetime of the component.</span></span>

  <span data-ttu-id="76459-210">I servizi DI inserimento nel componente usando [`@inject`](xref:mvc/views/razor#inject) o l' [ `[Inject]` attributo](xref:Microsoft.AspNetCore.Components.InjectAttribute) non vengono creati nell'ambito del componente.</span><span class="sxs-lookup"><span data-stu-id="76459-210">DI services injected into the component using [`@inject`](xref:mvc/views/razor#inject) or the [`[Inject]` attribute](xref:Microsoft.AspNetCore.Components.InjectAttribute) aren't created in the component's scope.</span></span> <span data-ttu-id="76459-211">Per utilizzare l'ambito del componente, i servizi devono essere risolti utilizzando <xref:Microsoft.Extensions.DependencyInjection.ServiceProviderServiceExtensions.GetRequiredService%2A> o <xref:System.IServiceProvider.GetService%2A> .</span><span class="sxs-lookup"><span data-stu-id="76459-211">To use the component's scope, services must be resolved using <xref:Microsoft.Extensions.DependencyInjection.ServiceProviderServiceExtensions.GetRequiredService%2A> or <xref:System.IServiceProvider.GetService%2A>.</span></span> <span data-ttu-id="76459-212">Tutti i servizi risolti utilizzando il <xref:Microsoft.AspNetCore.Components.OwningComponentBase.ScopedServices> provider hanno le dipendenze fornite dallo stesso ambito.</span><span class="sxs-lookup"><span data-stu-id="76459-212">Any services resolved using the <xref:Microsoft.AspNetCore.Components.OwningComponentBase.ScopedServices> provider have their dependencies provided from that same scope.</span></span>

  ::: moniker range=">= aspnetcore-5.0"

  [!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/dependency-injection/Preferences.razor?name=snippet&highlight=3,20-21)]

  ::: moniker-end

  ::: moniker range="< aspnetcore-5.0"

  [!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Pages/dependency-injection/Preferences.razor?name=snippet&highlight=3,20-21)]

  ::: moniker-end

* <span data-ttu-id="76459-213"><xref:Microsoft.AspNetCore.Components.OwningComponentBase%601> deriva da <xref:Microsoft.AspNetCore.Components.OwningComponentBase> e aggiunge una <xref:Microsoft.AspNetCore.Components.OwningComponentBase%601.Service%2A> proprietà che restituisce un'istanza di `T` dal provider dell'ambito di.</span><span class="sxs-lookup"><span data-stu-id="76459-213"><xref:Microsoft.AspNetCore.Components.OwningComponentBase%601> derives from <xref:Microsoft.AspNetCore.Components.OwningComponentBase> and adds a <xref:Microsoft.AspNetCore.Components.OwningComponentBase%601.Service%2A> property that returns an instance of `T` from the scoped DI provider.</span></span> <span data-ttu-id="76459-214">Questo tipo è un modo pratico per accedere ai servizi con ambito senza usare un'istanza di <xref:System.IServiceProvider> quando è presente un servizio primario richiesto dall'app dal contenitore di inserimento delle dipendenze usando l'ambito del componente.</span><span class="sxs-lookup"><span data-stu-id="76459-214">This type is a convenient way to access scoped services without using an instance of <xref:System.IServiceProvider> when there's one primary service the app requires from the DI container using the component's scope.</span></span> <span data-ttu-id="76459-215">La <xref:Microsoft.AspNetCore.Components.OwningComponentBase.ScopedServices> proprietà è disponibile, in modo che l'app possa ottenere i servizi di altri tipi, se necessario.</span><span class="sxs-lookup"><span data-stu-id="76459-215">The <xref:Microsoft.AspNetCore.Components.OwningComponentBase.ScopedServices> property is available, so the app can get services of other types, if necessary.</span></span>

  ```razor
  @page "/users"
  @attribute [Authorize]
  @inherits OwningComponentBase<AppDbContext>

  <h1>Users (@Service.Users.Count())</h1>

  <ul>
      @foreach (var user in Service.Users)
      {
          <li>@user.UserName</li>
      }
  </ul>
  ```

## <a name="use-of-an-entity-framework-core-ef-core-dbcontext-from-di"></a><span data-ttu-id="76459-216">Uso di un Entity Framework Core (EF Core) DbContext da DI</span><span class="sxs-lookup"><span data-stu-id="76459-216">Use of an Entity Framework Core (EF Core) DbContext from DI</span></span>

<span data-ttu-id="76459-217">Per altre informazioni, vedere <xref:blazor/blazor-server-ef-core>.</span><span class="sxs-lookup"><span data-stu-id="76459-217">For more information, see <xref:blazor/blazor-server-ef-core>.</span></span>

## <a name="detect-transient-disposables"></a><span data-ttu-id="76459-218">Rilevare eliminabili temporanei</span><span class="sxs-lookup"><span data-stu-id="76459-218">Detect transient disposables</span></span>

<span data-ttu-id="76459-219">Gli esempi seguenti illustrano come rilevare servizi temporanei monouso in un'app che deve usare <xref:Microsoft.AspNetCore.Components.OwningComponentBase> .</span><span class="sxs-lookup"><span data-stu-id="76459-219">The following examples show how to detect disposable transient services in an app that should use <xref:Microsoft.AspNetCore.Components.OwningComponentBase>.</span></span> <span data-ttu-id="76459-220">Per ulteriori informazioni, vedere la sezione relativa [alle classi dei componenti di base dell'utilità per gestire una](#utility-base-component-classes-to-manage-a-di-scope) sezione dell'ambito.</span><span class="sxs-lookup"><span data-stu-id="76459-220">For more information, see the [Utility base component classes to manage a DI scope](#utility-base-component-classes-to-manage-a-di-scope) section.</span></span>

::: zone pivot="webassembly"

<span data-ttu-id="76459-221">`DetectIncorrectUsagesOfTransientDisposables.cs`:</span><span class="sxs-lookup"><span data-stu-id="76459-221">`DetectIncorrectUsagesOfTransientDisposables.cs`:</span></span>

::: moniker range=">= aspnetcore-5.0"

[!code-csharp[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/dependency-injection/DetectIncorrectUsagesOfTransientDisposables.cs)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-csharp[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/dependency-injection/DetectIncorrectUsagesOfTransientDisposables.cs)]

::: moniker-end

<span data-ttu-id="76459-222">Nell' `TransientDisposable` esempio seguente viene rilevato ( `Program.cs` ):</span><span class="sxs-lookup"><span data-stu-id="76459-222">The `TransientDisposable` in the following example is detected (`Program.cs`):</span></span>

::: moniker range=">= aspnetcore-5.0"

```csharp
public class Program
{
    public static async Task Main(string[] args)
    {
        var builder = WebAssemblyHostBuilder.CreateDefault(args);
        builder.DetectIncorrectUsageOfTransients();
        builder.RootComponents.Add<App>("#app");

        builder.Services.AddTransient<TransientDisposable>();
        builder.Services.AddScoped(sp =>
            new HttpClient
            {
                BaseAddress = new Uri(builder.HostEnvironment.BaseAddress)
            });

        var host = builder.Build();
        host.EnableTransientDisposableDetection();
        await host.RunAsync();
    }
}

public class TransientDisposable : IDisposable
{
    public void Dispose() => throw new NotImplementedException();
}
```

::: moniker-end

::: moniker range="< aspnetcore-5.0"

```csharp
public class Program
{
    public static async Task Main(string[] args)
    {
        var builder = WebAssemblyHostBuilder.CreateDefault(args);
        builder.DetectIncorrectUsageOfTransients();
        builder.RootComponents.Add<App>("app");

        builder.Services.AddTransient<TransientDisposable>();
        builder.Services.AddScoped(sp =>
            new HttpClient
            {
                BaseAddress = new Uri(builder.HostEnvironment.BaseAddress)
            });

        var host = builder.Build();
        host.EnableTransientDisposableDetection();
        await host.RunAsync();
    }
}

public class TransientDisposable : IDisposable
{
    public void Dispose() => throw new NotImplementedException();
}
```

::: moniker-end

::: zone-end

::: zone pivot="server"

<span data-ttu-id="76459-223">`DetectIncorrectUsagesOfTransientDisposables.cs`:</span><span class="sxs-lookup"><span data-stu-id="76459-223">`DetectIncorrectUsagesOfTransientDisposables.cs`:</span></span>

::: moniker range=">= aspnetcore-5.0"

[!code-csharp[](~/blazor/common/samples/5.x/BlazorSample_Server/dependency-injection/DetectIncorrectUsagesOfTransientDisposables.cs)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-csharp[](~/blazor/common/samples/3.x/BlazorSample_Server/dependency-injection/DetectIncorrectUsagesOfTransientDisposables.cs)]

::: moniker-end

<span data-ttu-id="76459-224">Aggiungere lo spazio dei nomi per <xref:Microsoft.Extensions.DependencyInjection?displayProperty=fullName> a `Program.cs` :</span><span class="sxs-lookup"><span data-stu-id="76459-224">Add the namespace for <xref:Microsoft.Extensions.DependencyInjection?displayProperty=fullName> to `Program.cs`:</span></span>

```csharp
using Microsoft.Extensions.DependencyInjection;
```

<span data-ttu-id="76459-225">In `Program.CreateHostBuilder` di `Program.cs` :</span><span class="sxs-lookup"><span data-stu-id="76459-225">In `Program.CreateHostBuilder` of `Program.cs`:</span></span>

```csharp
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .DetectIncorrectUsageOfTransients()
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.UseStartup<Startup>();
        });
```

<span data-ttu-id="76459-226">Nell' `TransientDependency` esempio seguente viene rilevato ( `Startup.cs` ):</span><span class="sxs-lookup"><span data-stu-id="76459-226">The `TransientDependency` in the following example is detected (`Startup.cs`):</span></span>

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddRazorPages();
    services.AddServerSideBlazor();
    services.AddSingleton<WeatherForecastService>();
    services.AddTransient<TransientDependency>();
    services.AddTransient<ITransitiveTransientDisposableDependency, 
        TransitiveTransientDisposableDependency>();
}

public class TransitiveTransientDisposableDependency 
    : ITransitiveTransientDisposableDependency, IDisposable
{
    public void Dispose() { }
}

public interface ITransitiveTransientDisposableDependency
{
}

public class TransientDependency
{
    private readonly ITransitiveTransientDisposableDependency 
        _transitiveTransientDisposableDependency;

    public TransientDependency(ITransitiveTransientDisposableDependency 
        transitiveTransientDisposableDependency)
    {
        _transitiveTransientDisposableDependency = 
            transitiveTransientDisposableDependency;
    }
}
```

::: zone-end

<span data-ttu-id="76459-227">L'app può registrare gli eliminabili temporanei senza generare un'eccezione.</span><span class="sxs-lookup"><span data-stu-id="76459-227">The app can register transient disposables without throwing an exception.</span></span> <span data-ttu-id="76459-228">Tuttavia, se si tenta di risolvere i risultati monouso temporanei in un oggetto <xref:System.InvalidOperationException> , come illustrato nell'esempio riportato di seguito.</span><span class="sxs-lookup"><span data-stu-id="76459-228">However, attempting to resolve a transient disposable results in an <xref:System.InvalidOperationException>, as the following example shows.</span></span>

<span data-ttu-id="76459-229">`Pages/TransientDisposable.razor`:</span><span class="sxs-lookup"><span data-stu-id="76459-229">`Pages/TransientDisposable.razor`:</span></span>

```razor
@page "/transient-disposable"
@inject TransientDisposable TransientDisposable

<h1>Transient Disposable Detection</h1>
```

<span data-ttu-id="76459-230">Passare al `TransientDisposable` componente in `/transient-disposable` e <xref:System.InvalidOperationException> viene generata un'eccezione quando il Framework tenta di costruire un'istanza di `TransientDisposable` :</span><span class="sxs-lookup"><span data-stu-id="76459-230">Navigate to the `TransientDisposable` component at `/transient-disposable` and an <xref:System.InvalidOperationException> is thrown when the framework attempts to construct an instance of `TransientDisposable`:</span></span>

> <span data-ttu-id="76459-231">System. InvalidOperationException: si sta tentando di risolvere TransientDisposable di servizio Disposable temporaneo nell'ambito errato.</span><span class="sxs-lookup"><span data-stu-id="76459-231">System.InvalidOperationException: Trying to resolve transient disposable service TransientDisposable in the wrong scope.</span></span> <span data-ttu-id="76459-232">Usare una \<T> classe di base del componente ' OwningComponentBase ' per il servizio ' t'che si sta tentando di risolvere.</span><span class="sxs-lookup"><span data-stu-id="76459-232">Use an 'OwningComponentBase\<T>' component base class for the service 'T' you are trying to resolve.</span></span>

## <a name="additional-resources"></a><span data-ttu-id="76459-233">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="76459-233">Additional resources</span></span>

* <xref:fundamentals/dependency-injection>
* [<span data-ttu-id="76459-234">`IDisposable` linee guida per le istanze temporanee e condivise</span><span class="sxs-lookup"><span data-stu-id="76459-234">`IDisposable` guidance for Transient and shared instances</span></span>](xref:fundamentals/dependency-injection#idisposable-guidance-for-transient-and-shared-instances)
* <xref:mvc/views/dependency-injection>
