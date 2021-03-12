---
title: Attività in background con servizi ospitati in ASP.NET Core
author: rick-anderson
description: Informazioni su come implementare attività in background con servizi ospitati in ASP.NET Core.
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 02/10/2020
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
uid: fundamentals/host/hosted-services
ms.openlocfilehash: c0492c0c5b660e1387b0d0a4f6be405ded49ee92
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102587476"
---
# <a name="background-tasks-with-hosted-services-in-aspnet-core"></a>Attività in background con servizi ospitati in ASP.NET Core

Di [Jeow li Huan](https://github.com/huan086)

::: moniker range=">= aspnetcore-3.0"

In ASP.NET Core le attività in background possono essere implementate come *servizi ospitati*. Un servizio ospitato è una classe con logica di attività in background che implementa l'interfaccia <xref:Microsoft.Extensions.Hosting.IHostedService>. In questo argomento vengono forniti tre esempi di servizio ospitato:

* Attività in background eseguita su un timer.
* Servizio ospitato che attiva un [servizio con ambito](xref:fundamentals/dependency-injection#service-lifetimes). Il servizio con ambito può usare l' [inserimento di dipendenze](xref:fundamentals/dependency-injection).
* Attività in background in coda che vengono eseguite in sequenza.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/host/hosted-services/samples/) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="worker-service-template"></a>Modello di servizio di ruolo di lavoro

Il modello di servizio di ruolo di lavoro di ASP.NET Core rappresenta un punto di partenza per la scrittura di app di servizi a esecuzione prolungata. Un'app creata dal modello del servizio Worker specifica l'SDK del ruolo di lavoro nel file di progetto:

```xml
<Project Sdk="Microsoft.NET.Sdk.Worker">
```

Per usare il modello come base per un'app di servizi ospitati:

[!INCLUDE[](~/includes/worker-template-instructions.md)]

## <a name="package"></a>Pacchetto

Un'app basata sul modello del servizio Worker usa l' `Microsoft.NET.Sdk.Worker` SDK e contiene un riferimento esplicito al pacchetto per il pacchetto [Microsoft. Extensions. host](https://www.nuget.org/packages/Microsoft.Extensions.Hosting) . Ad esempio, vedere il file di progetto dell'app di esempio (*BackgroundTasksSample. csproj*).

Per le app Web che usano l' `Microsoft.NET.Sdk.Web` SDK, viene fatto riferimento in modo implicito al pacchetto [Microsoft. Extensions. host](https://www.nuget.org/packages/Microsoft.Extensions.Hosting) dal Framework condiviso. Un riferimento esplicito al pacchetto nel file di progetto dell'app non è obbligatorio.

## <a name="ihostedservice-interface"></a>Interfaccia IHostedService

L' <xref:Microsoft.Extensions.Hosting.IHostedService> interfaccia definisce due metodi per gli oggetti gestiti dall'host:

* [StartAsync (CancellationToken)](xref:Microsoft.Extensions.Hosting.IHostedService.StartAsync*): `StartAsync` contiene la logica per avviare l'attività in background. `StartAsync` viene chiamato *prima* di:

  * La pipeline di elaborazione delle richieste dell'app è configurata ( `Startup.Configure` ).
  * Il server viene avviato e viene attivato [IApplicationLifetime. ApplicationStarted](xref:Microsoft.AspNetCore.Hosting.IApplicationLifetime.ApplicationStarted*) .

  Il comportamento predefinito può essere modificato in modo che il servizio ospitato `StartAsync` venga eseguito dopo che la pipeline dell'app è stata configurata e `ApplicationStarted` chiamata. Per modificare il comportamento predefinito, aggiungere il servizio ospitato ( `VideosWatcher` nell'esempio seguente) dopo la chiamata a `ConfigureWebHostDefaults` :

  ```csharp
  using Microsoft.AspNetCore.Hosting;
  using Microsoft.Extensions.DependencyInjection;
  using Microsoft.Extensions.Hosting;

  public class Program
  {
      public static void Main(string[] args)
      {
          CreateHostBuilder(args).Build().Run();
      }

      public static IHostBuilder CreateHostBuilder(string[] args) =>
          Host.CreateDefaultBuilder(args)
              .ConfigureWebHostDefaults(webBuilder =>
              {
                  webBuilder.UseStartup<Startup>();
              })
              .ConfigureServices(services =>
              {
                  services.AddHostedService<VideosWatcher>();
              });
  }
  ```

* [StopAsync (CancellationToken)](xref:Microsoft.Extensions.Hosting.IHostedService.StopAsync*): attivato quando l'host esegue un arresto normale. `StopAsync` contiene la logica per terminare l'attività in background. Implementare <xref:System.IDisposable> e i [finalizzatori (distruttori)](/dotnet/csharp/programming-guide/classes-and-structs/destructors) per eliminare tutte le risorse non gestite.

  Il token di annullamento ha un timeout predefinito di cinque secondi che indica che il processo di arresto non è più normale. Quando viene richiesto l'annullamento sul token:

  * Tutte le operazioni in background rimanenti che sta eseguendo l'app devono essere interrotte.
  * Tutti i metodi eventuali chiamati in `StopAsync` devono essere completati rapidamente.

  Tuttavia, dopo la richiesta di annullamento le attività non vengono abbandonate: il chiamante attende il completamento di tutte le attività.

  Se l'app si arresta in modo imprevisto, ad esempio, il processo dell'app ha esito negativo, il metodo `StopAsync` potrebbe non essere chiamato. Pertanto è possibile che i metodi chiamati o le operazioni effettuate in `StopAsync` non vengano eseguiti.

  Per estendere il timeout di arresto predefinito di cinque secondi, impostare:

  * <xref:Microsoft.Extensions.Hosting.HostOptions.ShutdownTimeout*> quando si usa l'host generico. Per altre informazioni, vedere <xref:fundamentals/host/generic-host#shutdown-timeout>.
  * Impostazione di configurazione dell'host del timeout di arresto quando si usa l'host Web. Per altre informazioni, vedere <xref:fundamentals/host/web-host#shutdown-timeout>.

Il servizio ospitato viene attivato una volta all'avvio dell'app e arrestato normalmente all'arresto dell'applicazione. Se viene generato un errore durante l'esecuzione dell'attività in background, deve essere chiamato `Dispose` anche se `StopAsync` non viene chiamato.

## <a name="backgroundservice-base-class"></a>Classe di base BackgroundService

<xref:Microsoft.Extensions.Hosting.BackgroundService> è una classe di base per l'implementazione di un oggetto a esecuzione prolungata <xref:Microsoft.Extensions.Hosting.IHostedService> .

[ExecuteAsync (CancellationToken)](xref:Microsoft.Extensions.Hosting.BackgroundService.ExecuteAsync*) viene chiamato per eseguire il servizio in background. L'implementazione restituisce un oggetto <xref:System.Threading.Tasks.Task> che rappresenta l'intera durata del servizio in background. Non vengono avviati altri servizi fino a quando [ExecuteAsync non diventa asincrono](https://github.com/dotnet/extensions/issues/2149), ad esempio chiamando `await` . Evitare di eseguire operazioni di inizializzazione lunghe e bloccate in `ExecuteAsync` . Blocchi host in [StopAsync (CancellationToken)](xref:Microsoft.Extensions.Hosting.BackgroundService.StopAsync*) in attesa del `ExecuteAsync` completamento di.

Il token di annullamento viene attivato quando viene chiamato [IHostedService. StopAsync](xref:Microsoft.Extensions.Hosting.IHostedService.StopAsync*) . L'implementazione di `ExecuteAsync` deve terminare tempestivamente quando viene generato il token di annullamento per arrestare correttamente il servizio. In caso contrario, il servizio si arresta normalmente in corrispondenza del timeout di arresto. Per ulteriori informazioni, vedere la sezione [interfaccia IHostedService](#ihostedservice-interface) .

## <a name="timed-background-tasks"></a>Attività in background programmate

Un'attività programmata in background utilizza la classe [System.Threading.Timer](xref:System.Threading.Timer). Il timer attiva il metodo `DoWork` dell'attività. Il timer viene disabilitato con `StopAsync` ed eliminato quando il contenitore dei servizi è eliminato con `Dispose`:

[!code-csharp[](hosted-services/samples/3.x/BackgroundTasksSample/Services/TimedHostedService.cs?name=snippet1&highlight=16-17,34,41)]

Il <xref:System.Threading.Timer> non attende il completamento delle esecuzioni precedenti di `DoWork` , pertanto l'approccio illustrato potrebbe non essere adatto per ogni scenario. [Interlocked. Increment](xref:System.Threading.Interlocked.Increment*) viene usato per incrementare il contatore di esecuzione come operazione atomica, in modo da garantire che più thread non vengano aggiornati `executionCount` contemporaneamente.

Il servizio è registrato in `IHostBuilder.ConfigureServices` (*Program.cs*) con il `AddHostedService` metodo di estensione:

[!code-csharp[](hosted-services/samples/3.x/BackgroundTasksSample/Program.cs?name=snippet1)]

## <a name="consuming-a-scoped-service-in-a-background-task"></a>Utilizzo di un servizio con ambito in un'attività in background

Per usare i [servizi con ambito](xref:fundamentals/dependency-injection#service-lifetimes) all'interno di un [BackgroundService](#backgroundservice-base-class), creare un ambito. Non viene creato automaticamente alcun ambito per un servizio ospitato.

Il servizio dell'attività in background con ambito contiene la logica dell'attività in background. Nell'esempio seguente:

* Il servizio è asincrono. Il metodo `DoWork` restituisce un oggetto `Task`. A scopo dimostrativo, nel metodo è atteso un ritardo di dieci secondi `DoWork` .
* Un oggetto <xref:Microsoft.Extensions.Logging.ILogger> viene inserito nel servizio.

[!code-csharp[](hosted-services/samples/3.x/BackgroundTasksSample/Services/ScopedProcessingService.cs?name=snippet1)]

Il servizio ospitato crea un ambito per risolvere il servizio attività in background con ambito per chiamare il relativo `DoWork` metodo. `DoWork` Restituisce un oggetto `Task` , che è atteso in `ExecuteAsync` :

[!code-csharp[](hosted-services/samples/3.x/BackgroundTasksSample/Services/ConsumeScopedServiceHostedService.cs?name=snippet1&highlight=19,22-35)]

I servizi sono registrati in `IHostBuilder.ConfigureServices` (*Program.cs*). Il servizio ospitato viene registrato con il `AddHostedService` metodo di estensione:

[!code-csharp[](hosted-services/samples/3.x/BackgroundTasksSample/Program.cs?name=snippet2)]

## <a name="queued-background-tasks"></a>Attività in background in coda

Una coda delle attività in background è basata su .NET 4. x <xref:System.Web.Hosting.HostingEnvironment.QueueBackgroundWorkItem*> :

[!code-csharp[](hosted-services/samples/3.x/BackgroundTasksSample/Services/BackgroundTaskQueue.cs?name=snippet1)]

Nell'esempio seguente `QueueHostedService` :

* Il `BackgroundProcessing` metodo restituisce un oggetto `Task` , che è atteso in `ExecuteAsync` .
* Le attività in background nella coda vengono rimesse in coda ed eseguite in `BackgroundProcessing` .
* Gli elementi di lavoro sono in attesa prima che il servizio si arresti in `StopAsync` .

[!code-csharp[](hosted-services/samples/3.x/BackgroundTasksSample/Services/QueuedHostedService.cs?name=snippet1&highlight=28-29,33)]

Un `MonitorLoop` servizio gestisce le attività di Accodamento per il servizio ospitato ogni volta che la `w` chiave viene selezionata in un dispositivo di input:

* `IBackgroundTaskQueue`Viene inserito nel `MonitorLoop` servizio.
* `IBackgroundTaskQueue.QueueBackgroundWorkItem` viene chiamato per accodare un elemento di lavoro.
* L'elemento di lavoro simula un'attività in background con esecuzione prolungata:
  * 3 5 secondi di ritardo vengono eseguiti ( `Task.Delay` ).
  * `try-catch` <xref:System.OperationCanceledException> Se l'attività viene annullata, viene intercettata un'istruzione.

[!code-csharp[](hosted-services/samples/3.x/BackgroundTasksSample/Services/MonitorLoop.cs?name=snippet_Monitor&highlight=7,33)]

I servizi sono registrati in `IHostBuilder.ConfigureServices` (*Program.cs*). Il servizio ospitato viene registrato con il `AddHostedService` metodo di estensione:

[!code-csharp[](hosted-services/samples/3.x/BackgroundTasksSample/Program.cs?name=snippet3)]

`MonitorLoop` viene avviato in `Program.Main` :

[!code-csharp[](hosted-services/samples/3.x/BackgroundTasksSample/Program.cs?name=snippet4)]

::: moniker-end

::: moniker range="< aspnetcore-3.0"

In ASP.NET Core le attività in background possono essere implementate come *servizi ospitati*. Un servizio ospitato è una classe con logica di attività in background che implementa l'interfaccia <xref:Microsoft.Extensions.Hosting.IHostedService>. In questo argomento vengono forniti tre esempi di servizio ospitato:

* Attività in background eseguita su un timer.
* Servizio ospitato che attiva un [servizio con ambito](xref:fundamentals/dependency-injection#service-lifetimes). Il servizio con ambito può usare l' [inserimento di dipendenze](xref:fundamentals/dependency-injection)
* Attività in background in coda che vengono eseguite in sequenza.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/host/hosted-services/samples/) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="package"></a>Pacchetto

Fare riferimento al [metapacchetto Microsoft.AspNetCore.App](xref:fundamentals/metapackage-app) oppure aggiungere un riferimento al pacchetto [Microsoft.Extensions.Hosting](https://www.nuget.org/packages/Microsoft.Extensions.Hosting).

## <a name="ihostedservice-interface"></a>Interfaccia IHostedService

I servizi ospitati implementano l'interfaccia <xref:Microsoft.Extensions.Hosting.IHostedService>. L'interfaccia definisce due metodi riservati agli oggetti gestiti dall'host:

* [StartAsync (CancellationToken)](xref:Microsoft.Extensions.Hosting.IHostedService.StartAsync*): `StartAsync` contiene la logica per avviare l'attività in background. Quando si usa l' [host Web](xref:fundamentals/host/web-host), `StartAsync` viene chiamato dopo che il server è stato avviato e viene attivato [IApplicationLifetime. ApplicationStarted](xref:Microsoft.AspNetCore.Hosting.IApplicationLifetime.ApplicationStarted*) . Quando si usa l' [host generico](xref:fundamentals/host/generic-host), `StartAsync` viene chiamato prima di `ApplicationStarted` viene attivato.

* [StopAsync (CancellationToken)](xref:Microsoft.Extensions.Hosting.IHostedService.StopAsync*): attivato quando l'host esegue un arresto normale. `StopAsync` contiene la logica per terminare l'attività in background. Implementare <xref:System.IDisposable> e i [finalizzatori (distruttori)](/dotnet/csharp/programming-guide/classes-and-structs/destructors) per eliminare tutte le risorse non gestite.

  Il token di annullamento ha un timeout predefinito di cinque secondi che indica che il processo di arresto non è più normale. Quando viene richiesto l'annullamento sul token:

  * Tutte le operazioni in background rimanenti che sta eseguendo l'app devono essere interrotte.
  * Tutti i metodi eventuali chiamati in `StopAsync` devono essere completati rapidamente.

  Tuttavia, dopo la richiesta di annullamento le attività non vengono abbandonate: il chiamante attende il completamento di tutte le attività.

  Se l'app si arresta in modo imprevisto, ad esempio, il processo dell'app ha esito negativo, il metodo `StopAsync` potrebbe non essere chiamato. Pertanto è possibile che i metodi chiamati o le operazioni effettuate in `StopAsync` non vengano eseguiti.

  Per estendere il timeout di arresto predefinito di cinque secondi, impostare:

  * <xref:Microsoft.Extensions.Hosting.HostOptions.ShutdownTimeout*> quando si usa l'host generico. Per altre informazioni, vedere <xref:fundamentals/host/generic-host#shutdown-timeout>.
  * Impostazione di configurazione dell'host del timeout di arresto quando si usa l'host Web. Per altre informazioni, vedere <xref:fundamentals/host/web-host#shutdown-timeout>.

Il servizio ospitato viene attivato una volta all'avvio dell'app e arrestato normalmente all'arresto dell'applicazione. Se viene generato un errore durante l'esecuzione dell'attività in background, deve essere chiamato `Dispose` anche se `StopAsync` non viene chiamato.

## <a name="timed-background-tasks"></a>Attività in background programmate

Un'attività programmata in background utilizza la classe [System.Threading.Timer](xref:System.Threading.Timer). Il timer attiva il metodo `DoWork` dell'attività. Il timer viene disabilitato con `StopAsync` ed eliminato quando il contenitore dei servizi è eliminato con `Dispose`:

[!code-csharp[](hosted-services/samples/2.x/BackgroundTasksSample/Services/TimedHostedService.cs?name=snippet1&highlight=15-16,30,37)]

Il <xref:System.Threading.Timer> non attende il completamento delle esecuzioni precedenti di `DoWork` , pertanto l'approccio illustrato potrebbe non essere adatto per ogni scenario.

Il servizio registrato in `Startup.ConfigureServices` con il metodo di estensione `AddHostedService`:

[!code-csharp[](hosted-services/samples/2.x/BackgroundTasksSample/Startup.cs?name=snippet1)]

## <a name="consuming-a-scoped-service-in-a-background-task"></a>Utilizzo di un servizio con ambito in un'attività in background

Per usare i [servizi con ambito](xref:fundamentals/dependency-injection#service-lifetimes) all'interno di un `IHostedService` , creare un ambito. Non viene creato automaticamente alcun ambito per un servizio ospitato.

Il servizio dell'attività in background con ambito contiene la logica dell'attività in background. Nell'esempio seguente, <xref:Microsoft.Extensions.Logging.ILogger> viene inserito nel servizio:

[!code-csharp[](hosted-services/samples/2.x/BackgroundTasksSample/Services/ScopedProcessingService.cs?name=snippet1)]

Il servizio ospitato crea un ambito per risolvere il servizio dell'attività in background con ambito e chiamare il relativo metodo `DoWork`:

[!code-csharp[](hosted-services/samples/2.x/BackgroundTasksSample/Services/ConsumeScopedServiceHostedService.cs?name=snippet1&highlight=29-36)]

I servizi vengono registrati in `Startup.ConfigureServices`. L'implementazione di `IHostedService`viene registrata con il metodo di estensione `AddHostedService`:

[!code-csharp[](hosted-services/samples/2.x/BackgroundTasksSample/Startup.cs?name=snippet2)]

## <a name="queued-background-tasks"></a>Attività in background in coda

Una coda delle attività in background è basata sulla .NET Framework 4. x <xref:System.Web.Hosting.HostingEnvironment.QueueBackgroundWorkItem*> ([pianificata provvisoriamente per essere incorporata per ASP.NET Core](https://github.com/aspnet/Hosting/issues/1280)):

[!code-csharp[](hosted-services/samples/2.x/BackgroundTasksSample/Services/BackgroundTaskQueue.cs?name=snippet1)]

In `QueueHostedService`, le attività in background in coda vengono rimosse dalla coda ed eseguite come [BackgroundService](#backgroundservice-base-class), ovvero una classe di base per l'implementazione di un `IHostedService` a esecuzione prolungata:

[!code-csharp[](hosted-services/samples/2.x/BackgroundTasksSample/Services/QueuedHostedService.cs?name=snippet1&highlight=21,25)]

I servizi vengono registrati in `Startup.ConfigureServices`. L'implementazione di `IHostedService`viene registrata con il metodo di estensione `AddHostedService`:

[!code-csharp[](hosted-services/samples/2.x/BackgroundTasksSample/Startup.cs?name=snippet3)]

Nella classe modello della pagina di indice:

* `IBackgroundTaskQueue` viene inserito nel costruttore e assegnato a `Queue`.
* Un'interfaccia <xref:Microsoft.Extensions.DependencyInjection.IServiceScopeFactory> viene inserita e assegnata a `_serviceScopeFactory`. La factory viene usata per creare istanze di <xref:Microsoft.Extensions.DependencyInjection.IServiceScope>, che consente di creare servizi all'interno di un ambito. Viene creato un ambito per usare la classe `AppDbContext` dell'app (un [servizio con ambito](xref:fundamentals/dependency-injection#service-lifetimes)) per scrivere record di database in `IBackgroundTaskQueue` (un servizio singleton).

[!code-csharp[](hosted-services/samples/2.x/BackgroundTasksSample/Pages/Index.cshtml.cs?name=snippet1)]

Quando si seleziona il pulsante **Aggiungi attività** nella pagina di indice, viene eseguito il metodo `OnPostAddTask`. `QueueBackgroundWorkItem` viene chiamato per accodare un elemento di lavoro:

[!code-csharp[](hosted-services/samples/2.x/BackgroundTasksSample/Pages/Index.cshtml.cs?name=snippet2)]

::: moniker-end

## <a name="additional-resources"></a>Risorse aggiuntive

* [Implementare attività in background in microservizi con IHostedService e la classe BackgroundService](/dotnet/standard/microservices-architecture/multi-container-microservice-net-applications/background-tasks-with-ihostedservice)
* [Eseguire attività in background con processi Web nel servizio app Azure](/azure/app-service/webjobs-create)
* <xref:System.Threading.Timer>
