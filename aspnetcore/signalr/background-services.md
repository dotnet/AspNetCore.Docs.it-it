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
# <a name="host-aspnet-core-signalr-in-background-services"></a>ASP.NET Core host SignalR nei servizi in background

Di [Brady Gaster](https://twitter.com/bradygaster)

Questo articolo fornisce indicazioni per:

* SignalRHub di hosting usando un processo di lavoro in background ospitato con ASP.NET Core.
* Invio di messaggi ai client connessi dall'interno di un [BackgroundService](xref:Microsoft.Extensions.Hosting.BackgroundService)di .NET Core.

::: moniker range=">= aspnetcore-3.0"

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/signalr/background-service/samples/3.x) [(procedura per il download)](xref:index#how-to-download-a-sample)

::: moniker-end
::: moniker range="<= aspnetcore-2.2"

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/signalr/background-service/samples/2.2) [(procedura per il download)](xref:index#how-to-download-a-sample)

::: moniker-end

## <a name="enable-signalr-in-startup"></a>Abilita SignalR all'avvio

::: moniker range=">= aspnetcore-3.0"

L'hosting di SignalR hub ASP.NET Core nel contesto di un processo di lavoro in background è identico all'hosting di un hub in un'app web ASP.NET Core. Nel `Startup.ConfigureServices` metodo, la chiamata di `services.AddSignalR` aggiunge i servizi richiesti al livello di inserimento delle dipendenze di ASP.NET Core per supportare SignalR . In il `Startup.Configure` `MapHub` metodo viene chiamato nel `UseEndpoints` callback per connettere gli endpoint dell'hub nella pipeline della richiesta ASP.NET Core.

[!code-csharp[Startup](background-service/samples/3.x/Server/Startup.cs?name=Startup)]

::: moniker-end
::: moniker range="<= aspnetcore-2.2"

L'hosting di SignalR hub ASP.NET Core nel contesto di un processo di lavoro in background è identico all'hosting di un hub in un'app web ASP.NET Core. Nel `Startup.ConfigureServices` metodo, la chiamata di `services.AddSignalR` aggiunge i servizi richiesti al livello di inserimento delle dipendenze di ASP.NET Core per supportare SignalR . In `Startup.Configure` `UseSignalR` viene chiamato il metodo per connettere gli endpoint dell'hub nella pipeline della richiesta ASP.NET Core.

[!code-csharp[Startup](background-service/samples/2.2/Server/Startup.cs?name=Startup)]

::: moniker-end

Nell'esempio precedente, la `ClockHub` classe implementa la `Hub<T>` classe per creare un hub fortemente tipizzato. `ClockHub`È stato configurato nella `Startup` classe per rispondere alle richieste all'endpoint `/hubs/clock` .

Per altre informazioni sugli hub fortemente tipizzati, vedere [usare hub in SignalR per ASP.NET Core](xref:signalr/hubs#strongly-typed-hubs).

> [!NOTE]
> Questa funzionalità non è limitata alla [classe \<T> Hub](xref:Microsoft.AspNetCore.SignalR.Hub`1) . Tutte le classi che ereditano dall' [Hub](xref:Microsoft.AspNetCore.SignalR.Hub), ad esempio [DynamicHub](xref:Microsoft.AspNetCore.SignalR.DynamicHub), funzionano.

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[Startup](background-service/samples/3.x/Server/ClockHub.cs?name=ClockHub)]

::: moniker-end
::: moniker range="<= aspnetcore-2.2"

[!code-csharp[Startup](background-service/samples/2.2/Server/ClockHub.cs?name=ClockHub)]

::: moniker-end

L'interfaccia utilizzata dalla fortemente tipizzata `ClockHub` è l' `IClock` interfaccia.

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[Startup](background-service/samples/3.x/HubServiceInterfaces/IClock.cs?name=IClock)]

::: moniker-end
::: moniker range="<= aspnetcore-2.2"

[!code-csharp[Startup](background-service/samples/2.2/HubServiceInterfaces/IClock.cs?name=IClock)]

::: moniker-end

## <a name="call-a-signalr-hub-from-a-background-service"></a>Chiamare un SignalR Hub da un servizio in background

Durante l'avvio, la `Worker` classe, a `BackgroundService` , viene abilitata utilizzando `AddHostedService` .

```csharp
services.AddHostedService<Worker>();
```

Poiché SignalR è abilitato anche durante la `Startup` fase, in cui ogni hub è collegato a un singolo endpoint nella pipeline di richieste HTTP di ASP.NET Core, ogni hub è rappresentato da un `IHubContext<T>` nel server. Utilizzando le funzionalità DI DI ASP.NET Core, altre classi di cui è stata creata un'istanza dal livello di hosting, come `BackgroundService` le classi, le classi controller MVC o i Razor modelli di pagina, possono ottenere riferimenti a Hub sul lato server accettando istanze di `IHubContext<ClockHub, IClock>` durante la costruzione.

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[Startup](background-service/samples/3.x/Server/Worker.cs?name=Worker)]

::: moniker-end
::: moniker range="<= aspnetcore-2.2"

[!code-csharp[Startup](background-service/samples/2.2/Server/Worker.cs?name=Worker)]

::: moniker-end

Poiché il `ExecuteAsync` metodo viene chiamato in modo iterativo nel servizio in background, la data e l'ora correnti del server vengono inviate ai client connessi tramite `ClockHub` .

## <a name="react-to-signalr-events-with-background-services"></a>Reagire agli SignalR eventi con servizi in background

Analogamente a un'app a pagina singola che usa il client JavaScript per SignalR o un'app desktop .NET <xref:signalr/dotnet-client> che usa, un' `BackgroundService` implementazione di o `IHostedService` può essere usata anche per connettersi agli SignalR Hub e rispondere agli eventi.

La `ClockHubClient` classe implementa l' `IClock` interfaccia e l'interfaccia `IHostedService` . In questo modo può essere abilitata durante l' `Startup` esecuzione continua e rispondere agli eventi dell'hub dal server.

```csharp
public partial class ClockHubClient : IClock, IHostedService
{
}
```

Durante l'inizializzazione, `ClockHubClient` Crea un'istanza di un oggetto `HubConnection` e Abilita il `IClock.ShowTime` metodo come gestore per l'evento dell'hub `ShowTime` .

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[The ClockHubClient constructor](background-service/samples/3.x/Clients.ConsoleTwo/ClockHubClient.cs?name=ClockHubClientCtor)]

Nell' `IHostedService.StartAsync` implementazione di `HubConnection` viene avviato in modo asincrono.

[!code-csharp[StartAsync method](background-service/samples/3.x/Clients.ConsoleTwo/ClockHubClient.cs?name=StartAsync)]

Durante il `IHostedService.StopAsync` metodo, l'oggetto `HubConnection` viene eliminato in modo asincrono.

[!code-csharp[StopAsync method](background-service/samples/3.x/Clients.ConsoleTwo/ClockHubClient.cs?name=StopAsync)]

::: moniker-end
::: moniker range="<= aspnetcore-2.2"

[!code-csharp[The ClockHubClient constructor](background-service/samples/2.2/Clients.ConsoleTwo/ClockHubClient.cs?name=ClockHubClientCtor)]

Nell' `IHostedService.StartAsync` implementazione di `HubConnection` viene avviato in modo asincrono.

[!code-csharp[StartAsync method](background-service/samples/2.2/Clients.ConsoleTwo/ClockHubClient.cs?name=StartAsync)]

Durante il `IHostedService.StopAsync` metodo, l'oggetto `HubConnection` viene eliminato in modo asincrono.

[!code-csharp[StopAsync method](background-service/samples/2.2/Clients.ConsoleTwo/ClockHubClient.cs?name=StopAsync)]

::: moniker-end

## <a name="additional-resources"></a>Risorse aggiuntive

* [Operazioni preliminari](xref:tutorials/signalr)
* [Hub](xref:signalr/hubs)
* [Pubblicare in Azure](xref:signalr/publish-to-azure-web-app)
* [Hub fortemente tipizzati](xref:signalr/hubs#strongly-typed-hubs)
