---
title: ASP.NET Core host SignalR nei servizi in background
author: bradygaster
description: Informazioni su come inviare messaggi ai SignalR client da classi BackgroundService di .NET Core.
monikerRange: '>= aspnetcore-2.2'
ms.author: bradyg
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
uid: signalr/background-services
ms.openlocfilehash: 9d95d7c9a33bcf2f4a603d815269752124133ee6
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102589699"
---
# <a name="host-aspnet-core-signalr-in-background-services"></a><span data-ttu-id="670a8-103">ASP.NET Core host SignalR nei servizi in background</span><span class="sxs-lookup"><span data-stu-id="670a8-103">Host ASP.NET Core SignalR in background services</span></span>

<span data-ttu-id="670a8-104">Di [Brady Gaster](https://twitter.com/bradygaster)</span><span class="sxs-lookup"><span data-stu-id="670a8-104">By [Brady Gaster](https://twitter.com/bradygaster)</span></span>

<span data-ttu-id="670a8-105">Questo articolo fornisce indicazioni per:</span><span class="sxs-lookup"><span data-stu-id="670a8-105">This article provides guidance for:</span></span>

* <span data-ttu-id="670a8-106">SignalRHub di hosting usando un processo di lavoro in background ospitato con ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="670a8-106">Hosting SignalR Hubs using a background worker process hosted with ASP.NET Core.</span></span>
* <span data-ttu-id="670a8-107">Invio di messaggi ai client connessi dall'interno di un [BackgroundService](xref:Microsoft.Extensions.Hosting.BackgroundService)di .NET Core.</span><span class="sxs-lookup"><span data-stu-id="670a8-107">Sending messages to connected clients from within a .NET Core [BackgroundService](xref:Microsoft.Extensions.Hosting.BackgroundService).</span></span>

::: moniker range=">= aspnetcore-3.0"

<span data-ttu-id="670a8-108">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/signalr/background-service/samples/3.x) [(procedura per il download)](xref:index#how-to-download-a-sample)</span><span class="sxs-lookup"><span data-stu-id="670a8-108">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/signalr/background-service/samples/3.x) [(how to download)](xref:index#how-to-download-a-sample)</span></span>

::: moniker-end
::: moniker range="<= aspnetcore-2.2"

<span data-ttu-id="670a8-109">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/signalr/background-service/samples/2.2) [(procedura per il download)](xref:index#how-to-download-a-sample)</span><span class="sxs-lookup"><span data-stu-id="670a8-109">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/signalr/background-service/samples/2.2) [(how to download)](xref:index#how-to-download-a-sample)</span></span>

::: moniker-end

## <a name="enable-signalr-in-startup"></a><span data-ttu-id="670a8-110">Abilita SignalR all'avvio</span><span class="sxs-lookup"><span data-stu-id="670a8-110">Enable SignalR in startup</span></span>

::: moniker range=">= aspnetcore-3.0"

<span data-ttu-id="670a8-111">L'hosting di SignalR hub ASP.NET Core nel contesto di un processo di lavoro in background è identico all'hosting di un hub in un'app web ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="670a8-111">Hosting ASP.NET Core SignalR Hubs in the context of a background worker process is identical to hosting a Hub in an ASP.NET Core web app.</span></span> <span data-ttu-id="670a8-112">Nel `Startup.ConfigureServices` metodo, la chiamata di `services.AddSignalR` aggiunge i servizi richiesti al livello di inserimento delle dipendenze di ASP.NET Core per supportare SignalR .</span><span class="sxs-lookup"><span data-stu-id="670a8-112">In the `Startup.ConfigureServices` method, calling `services.AddSignalR` adds the required services to the ASP.NET Core Dependency Injection (DI) layer to support SignalR.</span></span> <span data-ttu-id="670a8-113">In il `Startup.Configure` `MapHub` metodo viene chiamato nel `UseEndpoints` callback per connettere gli endpoint dell'hub nella pipeline della richiesta ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="670a8-113">In `Startup.Configure`, the `MapHub` method is called in the `UseEndpoints` callback to connect the Hub endpoints in the ASP.NET Core request pipeline.</span></span>

[!code-csharp[Startup](background-service/samples/3.x/Server/Startup.cs?name=Startup)]

::: moniker-end
::: moniker range="<= aspnetcore-2.2"

<span data-ttu-id="670a8-114">L'hosting di SignalR hub ASP.NET Core nel contesto di un processo di lavoro in background è identico all'hosting di un hub in un'app web ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="670a8-114">Hosting ASP.NET Core SignalR Hubs in the context of a background worker process is identical to hosting a Hub in an ASP.NET Core web app.</span></span> <span data-ttu-id="670a8-115">Nel `Startup.ConfigureServices` metodo, la chiamata di `services.AddSignalR` aggiunge i servizi richiesti al livello di inserimento delle dipendenze di ASP.NET Core per supportare SignalR .</span><span class="sxs-lookup"><span data-stu-id="670a8-115">In the `Startup.ConfigureServices` method, calling `services.AddSignalR` adds the required services to the ASP.NET Core Dependency Injection (DI) layer to support SignalR.</span></span> <span data-ttu-id="670a8-116">In `Startup.Configure` `UseSignalR` viene chiamato il metodo per connettere gli endpoint dell'hub nella pipeline della richiesta ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="670a8-116">In `Startup.Configure`, the `UseSignalR` method is called to connect the Hub endpoint(s) in the ASP.NET Core request pipeline.</span></span>

[!code-csharp[Startup](background-service/samples/2.2/Server/Startup.cs?name=Startup)]

::: moniker-end

<span data-ttu-id="670a8-117">Nell'esempio precedente, la `ClockHub` classe implementa la `Hub<T>` classe per creare un hub fortemente tipizzato.</span><span class="sxs-lookup"><span data-stu-id="670a8-117">In the preceding example, the `ClockHub` class implements the `Hub<T>` class to create a strongly typed Hub.</span></span> <span data-ttu-id="670a8-118">`ClockHub`È stato configurato nella `Startup` classe per rispondere alle richieste all'endpoint `/hubs/clock` .</span><span class="sxs-lookup"><span data-stu-id="670a8-118">The `ClockHub` has been configured in the `Startup` class to respond to requests at the endpoint `/hubs/clock`.</span></span>

<span data-ttu-id="670a8-119">Per altre informazioni sugli hub fortemente tipizzati, vedere [usare hub in SignalR per ASP.NET Core](xref:signalr/hubs#strongly-typed-hubs).</span><span class="sxs-lookup"><span data-stu-id="670a8-119">For more information on strongly typed Hubs, see [Use hubs in SignalR for ASP.NET Core](xref:signalr/hubs#strongly-typed-hubs).</span></span>

> [!NOTE]
> <span data-ttu-id="670a8-120">Questa funzionalità non è limitata alla [classe \<T> Hub](xref:Microsoft.AspNetCore.SignalR.Hub`1) .</span><span class="sxs-lookup"><span data-stu-id="670a8-120">This functionality isn't limited to the [Hub\<T>](xref:Microsoft.AspNetCore.SignalR.Hub`1) class.</span></span> <span data-ttu-id="670a8-121">Tutte le classi che ereditano dall' [Hub](xref:Microsoft.AspNetCore.SignalR.Hub), ad esempio [DynamicHub](xref:Microsoft.AspNetCore.SignalR.DynamicHub), funzionano.</span><span class="sxs-lookup"><span data-stu-id="670a8-121">Any class that inherits from [Hub](xref:Microsoft.AspNetCore.SignalR.Hub), such as [DynamicHub](xref:Microsoft.AspNetCore.SignalR.DynamicHub), works.</span></span>

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[Startup](background-service/samples/3.x/Server/ClockHub.cs?name=ClockHub)]

::: moniker-end
::: moniker range="<= aspnetcore-2.2"

[!code-csharp[Startup](background-service/samples/2.2/Server/ClockHub.cs?name=ClockHub)]

::: moniker-end

<span data-ttu-id="670a8-122">L'interfaccia utilizzata dalla fortemente tipizzata `ClockHub` è l' `IClock` interfaccia.</span><span class="sxs-lookup"><span data-stu-id="670a8-122">The interface used by the strongly typed `ClockHub` is the `IClock` interface.</span></span>

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[Startup](background-service/samples/3.x/HubServiceInterfaces/IClock.cs?name=IClock)]

::: moniker-end
::: moniker range="<= aspnetcore-2.2"

[!code-csharp[Startup](background-service/samples/2.2/HubServiceInterfaces/IClock.cs?name=IClock)]

::: moniker-end

## <a name="call-a-signalr-hub-from-a-background-service"></a><span data-ttu-id="670a8-123">Chiamare un SignalR Hub da un servizio in background</span><span class="sxs-lookup"><span data-stu-id="670a8-123">Call a SignalR Hub from a background service</span></span>

<span data-ttu-id="670a8-124">Durante l'avvio, la `Worker` classe, a `BackgroundService` , viene abilitata utilizzando `AddHostedService` .</span><span class="sxs-lookup"><span data-stu-id="670a8-124">During startup, the `Worker` class, a `BackgroundService`, is enabled using `AddHostedService`.</span></span>

```csharp
services.AddHostedService<Worker>();
```

<span data-ttu-id="670a8-125">Poiché SignalR è abilitato anche durante la `Startup` fase, in cui ogni hub è collegato a un singolo endpoint nella pipeline di richieste HTTP di ASP.NET Core, ogni hub è rappresentato da un `IHubContext<T>` nel server.</span><span class="sxs-lookup"><span data-stu-id="670a8-125">Since SignalR is also enabled up during the `Startup` phase, in which each Hub is attached to an individual endpoint in ASP.NET Core's HTTP request pipeline, each Hub is represented by an `IHubContext<T>` on the server.</span></span> <span data-ttu-id="670a8-126">Utilizzando le funzionalità DI DI ASP.NET Core, altre classi di cui è stata creata un'istanza dal livello di hosting, come `BackgroundService` le classi, le classi controller MVC o i Razor modelli di pagina, possono ottenere riferimenti a Hub sul lato server accettando istanze di `IHubContext<ClockHub, IClock>` durante la costruzione.</span><span class="sxs-lookup"><span data-stu-id="670a8-126">Using ASP.NET Core's DI features, other classes instantiated by the hosting layer, like `BackgroundService` classes, MVC Controller classes, or Razor page models, can get references to server-side Hubs by accepting instances of `IHubContext<ClockHub, IClock>` during construction.</span></span>

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[Startup](background-service/samples/3.x/Server/Worker.cs?name=Worker)]

::: moniker-end
::: moniker range="<= aspnetcore-2.2"

[!code-csharp[Startup](background-service/samples/2.2/Server/Worker.cs?name=Worker)]

::: moniker-end

<span data-ttu-id="670a8-127">Poiché il `ExecuteAsync` metodo viene chiamato in modo iterativo nel servizio in background, la data e l'ora correnti del server vengono inviate ai client connessi tramite `ClockHub` .</span><span class="sxs-lookup"><span data-stu-id="670a8-127">As the `ExecuteAsync` method is called iteratively in the background service, the server's current date and time are sent to the connected clients using the `ClockHub`.</span></span>

## <a name="react-to-signalr-events-with-background-services"></a><span data-ttu-id="670a8-128">Reagire agli SignalR eventi con servizi in background</span><span class="sxs-lookup"><span data-stu-id="670a8-128">React to SignalR events with background services</span></span>

<span data-ttu-id="670a8-129">Analogamente a un'app a pagina singola che usa il client JavaScript per SignalR o un'app desktop .NET <xref:signalr/dotnet-client> che usa, un' `BackgroundService` implementazione di o `IHostedService` può essere usata anche per connettersi agli SignalR Hub e rispondere agli eventi.</span><span class="sxs-lookup"><span data-stu-id="670a8-129">Like a Single Page App using the JavaScript client for SignalR, or a .NET desktop app using the <xref:signalr/dotnet-client>, a `BackgroundService` or `IHostedService` implementation can also be used to connect to SignalR Hubs and respond to events.</span></span>

<span data-ttu-id="670a8-130">La `ClockHubClient` classe implementa l' `IClock` interfaccia e l'interfaccia `IHostedService` .</span><span class="sxs-lookup"><span data-stu-id="670a8-130">The `ClockHubClient` class implements both the `IClock` interface and the `IHostedService` interface.</span></span> <span data-ttu-id="670a8-131">In questo modo può essere abilitata durante l' `Startup` esecuzione continua e rispondere agli eventi dell'hub dal server.</span><span class="sxs-lookup"><span data-stu-id="670a8-131">This way it can be enabled during `Startup` to run continuously and respond to Hub events from the server.</span></span>

```csharp
public partial class ClockHubClient : IClock, IHostedService
{
}
```

<span data-ttu-id="670a8-132">Durante l'inizializzazione, `ClockHubClient` Crea un'istanza di un oggetto `HubConnection` e Abilita il `IClock.ShowTime` metodo come gestore per l'evento dell'hub `ShowTime` .</span><span class="sxs-lookup"><span data-stu-id="670a8-132">During initialization, the `ClockHubClient` creates an instance of a `HubConnection` and enables the `IClock.ShowTime` method as the handler for the Hub's `ShowTime` event.</span></span>

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[The ClockHubClient constructor](background-service/samples/3.x/Clients.ConsoleTwo/ClockHubClient.cs?name=ClockHubClientCtor)]

<span data-ttu-id="670a8-133">Nell' `IHostedService.StartAsync` implementazione di `HubConnection` viene avviato in modo asincrono.</span><span class="sxs-lookup"><span data-stu-id="670a8-133">In the `IHostedService.StartAsync` implementation, the `HubConnection` is started asynchronously.</span></span>

[!code-csharp[StartAsync method](background-service/samples/3.x/Clients.ConsoleTwo/ClockHubClient.cs?name=StartAsync)]

<span data-ttu-id="670a8-134">Durante il `IHostedService.StopAsync` metodo, l'oggetto `HubConnection` viene eliminato in modo asincrono.</span><span class="sxs-lookup"><span data-stu-id="670a8-134">During the `IHostedService.StopAsync` method, the `HubConnection` is disposed of asynchronously.</span></span>

[!code-csharp[StopAsync method](background-service/samples/3.x/Clients.ConsoleTwo/ClockHubClient.cs?name=StopAsync)]

::: moniker-end
::: moniker range="<= aspnetcore-2.2"

[!code-csharp[The ClockHubClient constructor](background-service/samples/2.2/Clients.ConsoleTwo/ClockHubClient.cs?name=ClockHubClientCtor)]

<span data-ttu-id="670a8-135">Nell' `IHostedService.StartAsync` implementazione di `HubConnection` viene avviato in modo asincrono.</span><span class="sxs-lookup"><span data-stu-id="670a8-135">In the `IHostedService.StartAsync` implementation, the `HubConnection` is started asynchronously.</span></span>

[!code-csharp[StartAsync method](background-service/samples/2.2/Clients.ConsoleTwo/ClockHubClient.cs?name=StartAsync)]

<span data-ttu-id="670a8-136">Durante il `IHostedService.StopAsync` metodo, l'oggetto `HubConnection` viene eliminato in modo asincrono.</span><span class="sxs-lookup"><span data-stu-id="670a8-136">During the `IHostedService.StopAsync` method, the `HubConnection` is disposed of asynchronously.</span></span>

[!code-csharp[StopAsync method](background-service/samples/2.2/Clients.ConsoleTwo/ClockHubClient.cs?name=StopAsync)]

::: moniker-end

## <a name="additional-resources"></a><span data-ttu-id="670a8-137">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="670a8-137">Additional resources</span></span>

* [<span data-ttu-id="670a8-138">Operazioni preliminari</span><span class="sxs-lookup"><span data-stu-id="670a8-138">Get started</span></span>](xref:tutorials/signalr)
* [<span data-ttu-id="670a8-139">Hub</span><span class="sxs-lookup"><span data-stu-id="670a8-139">Hubs</span></span>](xref:signalr/hubs)
* [<span data-ttu-id="670a8-140">Pubblicare in Azure</span><span class="sxs-lookup"><span data-stu-id="670a8-140">Publish to Azure</span></span>](xref:signalr/publish-to-azure-web-app)
* [<span data-ttu-id="670a8-141">Hub fortemente tipizzati</span><span class="sxs-lookup"><span data-stu-id="670a8-141">Strongly typed Hubs</span></span>](xref:signalr/hubs#strongly-typed-hubs)
