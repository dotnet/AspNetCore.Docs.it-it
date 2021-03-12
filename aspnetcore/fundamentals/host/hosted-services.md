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
# <a name="background-tasks-with-hosted-services-in-aspnet-core"></a><span data-ttu-id="ec8a1-103">Attività in background con servizi ospitati in ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="ec8a1-103">Background tasks with hosted services in ASP.NET Core</span></span>

<span data-ttu-id="ec8a1-104">Di [Jeow li Huan](https://github.com/huan086)</span><span class="sxs-lookup"><span data-stu-id="ec8a1-104">By [Jeow Li Huan](https://github.com/huan086)</span></span>

::: moniker range=">= aspnetcore-3.0"

<span data-ttu-id="ec8a1-105">In ASP.NET Core le attività in background possono essere implementate come *servizi ospitati*.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-105">In ASP.NET Core, background tasks can be implemented as *hosted services*.</span></span> <span data-ttu-id="ec8a1-106">Un servizio ospitato è una classe con logica di attività in background che implementa l'interfaccia <xref:Microsoft.Extensions.Hosting.IHostedService>.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-106">A hosted service is a class with background task logic that implements the <xref:Microsoft.Extensions.Hosting.IHostedService> interface.</span></span> <span data-ttu-id="ec8a1-107">In questo argomento vengono forniti tre esempi di servizio ospitato:</span><span class="sxs-lookup"><span data-stu-id="ec8a1-107">This topic provides three hosted service examples:</span></span>

* <span data-ttu-id="ec8a1-108">Attività in background eseguita su un timer.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-108">Background task that runs on a timer.</span></span>
* <span data-ttu-id="ec8a1-109">Servizio ospitato che attiva un [servizio con ambito](xref:fundamentals/dependency-injection#service-lifetimes).</span><span class="sxs-lookup"><span data-stu-id="ec8a1-109">Hosted service that activates a [scoped service](xref:fundamentals/dependency-injection#service-lifetimes).</span></span> <span data-ttu-id="ec8a1-110">Il servizio con ambito può usare l' [inserimento di dipendenze](xref:fundamentals/dependency-injection).</span><span class="sxs-lookup"><span data-stu-id="ec8a1-110">The scoped service can use [dependency injection (DI)](xref:fundamentals/dependency-injection).</span></span>
* <span data-ttu-id="ec8a1-111">Attività in background in coda che vengono eseguite in sequenza.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-111">Queued background tasks that run sequentially.</span></span>

<span data-ttu-id="ec8a1-112">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/host/hosted-services/samples/) ([procedura per il download](xref:index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="ec8a1-112">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/host/hosted-services/samples/) ([how to download](xref:index#how-to-download-a-sample))</span></span>

## <a name="worker-service-template"></a><span data-ttu-id="ec8a1-113">Modello di servizio di ruolo di lavoro</span><span class="sxs-lookup"><span data-stu-id="ec8a1-113">Worker Service template</span></span>

<span data-ttu-id="ec8a1-114">Il modello di servizio di ruolo di lavoro di ASP.NET Core rappresenta un punto di partenza per la scrittura di app di servizi a esecuzione prolungata.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-114">The ASP.NET Core Worker Service template provides a starting point for writing long running service apps.</span></span> <span data-ttu-id="ec8a1-115">Un'app creata dal modello del servizio Worker specifica l'SDK del ruolo di lavoro nel file di progetto:</span><span class="sxs-lookup"><span data-stu-id="ec8a1-115">An app created from the Worker Service template specifies the Worker SDK in its project file:</span></span>

```xml
<Project Sdk="Microsoft.NET.Sdk.Worker">
```

<span data-ttu-id="ec8a1-116">Per usare il modello come base per un'app di servizi ospitati:</span><span class="sxs-lookup"><span data-stu-id="ec8a1-116">To use the template as a basis for a hosted services app:</span></span>

[!INCLUDE[](~/includes/worker-template-instructions.md)]

## <a name="package"></a><span data-ttu-id="ec8a1-117">Pacchetto</span><span class="sxs-lookup"><span data-stu-id="ec8a1-117">Package</span></span>

<span data-ttu-id="ec8a1-118">Un'app basata sul modello del servizio Worker usa l' `Microsoft.NET.Sdk.Worker` SDK e contiene un riferimento esplicito al pacchetto per il pacchetto [Microsoft. Extensions. host](https://www.nuget.org/packages/Microsoft.Extensions.Hosting) .</span><span class="sxs-lookup"><span data-stu-id="ec8a1-118">An app based on the Worker Service template uses the `Microsoft.NET.Sdk.Worker` SDK and has an explicit package reference to the [Microsoft.Extensions.Hosting](https://www.nuget.org/packages/Microsoft.Extensions.Hosting) package.</span></span> <span data-ttu-id="ec8a1-119">Ad esempio, vedere il file di progetto dell'app di esempio (*BackgroundTasksSample. csproj*).</span><span class="sxs-lookup"><span data-stu-id="ec8a1-119">For example, see the sample app's project file (*BackgroundTasksSample.csproj*).</span></span>

<span data-ttu-id="ec8a1-120">Per le app Web che usano l' `Microsoft.NET.Sdk.Web` SDK, viene fatto riferimento in modo implicito al pacchetto [Microsoft. Extensions. host](https://www.nuget.org/packages/Microsoft.Extensions.Hosting) dal Framework condiviso.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-120">For web apps that use the `Microsoft.NET.Sdk.Web` SDK, the [Microsoft.Extensions.Hosting](https://www.nuget.org/packages/Microsoft.Extensions.Hosting) package is referenced implicitly from the shared framework.</span></span> <span data-ttu-id="ec8a1-121">Un riferimento esplicito al pacchetto nel file di progetto dell'app non è obbligatorio.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-121">An explicit package reference in the app's project file isn't required.</span></span>

## <a name="ihostedservice-interface"></a><span data-ttu-id="ec8a1-122">Interfaccia IHostedService</span><span class="sxs-lookup"><span data-stu-id="ec8a1-122">IHostedService interface</span></span>

<span data-ttu-id="ec8a1-123">L' <xref:Microsoft.Extensions.Hosting.IHostedService> interfaccia definisce due metodi per gli oggetti gestiti dall'host:</span><span class="sxs-lookup"><span data-stu-id="ec8a1-123">The <xref:Microsoft.Extensions.Hosting.IHostedService> interface defines two methods for objects that are managed by the host:</span></span>

* <span data-ttu-id="ec8a1-124">[StartAsync (CancellationToken)](xref:Microsoft.Extensions.Hosting.IHostedService.StartAsync*): `StartAsync` contiene la logica per avviare l'attività in background.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-124">[StartAsync(CancellationToken)](xref:Microsoft.Extensions.Hosting.IHostedService.StartAsync*): `StartAsync` contains the logic to start the background task.</span></span> <span data-ttu-id="ec8a1-125">`StartAsync` viene chiamato *prima* di:</span><span class="sxs-lookup"><span data-stu-id="ec8a1-125">`StartAsync` is called *before*:</span></span>

  * <span data-ttu-id="ec8a1-126">La pipeline di elaborazione delle richieste dell'app è configurata ( `Startup.Configure` ).</span><span class="sxs-lookup"><span data-stu-id="ec8a1-126">The app's request processing pipeline is configured (`Startup.Configure`).</span></span>
  * <span data-ttu-id="ec8a1-127">Il server viene avviato e viene attivato [IApplicationLifetime. ApplicationStarted](xref:Microsoft.AspNetCore.Hosting.IApplicationLifetime.ApplicationStarted*) .</span><span class="sxs-lookup"><span data-stu-id="ec8a1-127">The server is started and [IApplicationLifetime.ApplicationStarted](xref:Microsoft.AspNetCore.Hosting.IApplicationLifetime.ApplicationStarted*) is triggered.</span></span>

  <span data-ttu-id="ec8a1-128">Il comportamento predefinito può essere modificato in modo che il servizio ospitato `StartAsync` venga eseguito dopo che la pipeline dell'app è stata configurata e `ApplicationStarted` chiamata.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-128">The default behavior can be changed so that the hosted service's `StartAsync` runs after the app's pipeline has been configured and `ApplicationStarted` is called.</span></span> <span data-ttu-id="ec8a1-129">Per modificare il comportamento predefinito, aggiungere il servizio ospitato ( `VideosWatcher` nell'esempio seguente) dopo la chiamata a `ConfigureWebHostDefaults` :</span><span class="sxs-lookup"><span data-stu-id="ec8a1-129">To change the default behavior, add the hosted service (`VideosWatcher` in the following example) after calling `ConfigureWebHostDefaults`:</span></span>

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

* <span data-ttu-id="ec8a1-130">[StopAsync (CancellationToken)](xref:Microsoft.Extensions.Hosting.IHostedService.StopAsync*): attivato quando l'host esegue un arresto normale.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-130">[StopAsync(CancellationToken)](xref:Microsoft.Extensions.Hosting.IHostedService.StopAsync*): Triggered when the host is performing a graceful shutdown.</span></span> <span data-ttu-id="ec8a1-131">`StopAsync` contiene la logica per terminare l'attività in background.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-131">`StopAsync` contains the logic to end the background task.</span></span> <span data-ttu-id="ec8a1-132">Implementare <xref:System.IDisposable> e i [finalizzatori (distruttori)](/dotnet/csharp/programming-guide/classes-and-structs/destructors) per eliminare tutte le risorse non gestite.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-132">Implement <xref:System.IDisposable> and [finalizers (destructors)](/dotnet/csharp/programming-guide/classes-and-structs/destructors) to dispose of any unmanaged resources.</span></span>

  <span data-ttu-id="ec8a1-133">Il token di annullamento ha un timeout predefinito di cinque secondi che indica che il processo di arresto non è più normale.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-133">The cancellation token has a default five second timeout to indicate that the shutdown process should no longer be graceful.</span></span> <span data-ttu-id="ec8a1-134">Quando viene richiesto l'annullamento sul token:</span><span class="sxs-lookup"><span data-stu-id="ec8a1-134">When cancellation is requested on the token:</span></span>

  * <span data-ttu-id="ec8a1-135">Tutte le operazioni in background rimanenti che sta eseguendo l'app devono essere interrotte.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-135">Any remaining background operations that the app is performing should be aborted.</span></span>
  * <span data-ttu-id="ec8a1-136">Tutti i metodi eventuali chiamati in `StopAsync` devono essere completati rapidamente.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-136">Any methods called in `StopAsync` should return promptly.</span></span>

  <span data-ttu-id="ec8a1-137">Tuttavia, dopo la richiesta di annullamento le attività non vengono abbandonate: il chiamante attende il completamento di tutte le attività.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-137">However, tasks aren't abandoned after cancellation is requested&mdash;the caller awaits all tasks to complete.</span></span>

  <span data-ttu-id="ec8a1-138">Se l'app si arresta in modo imprevisto, ad esempio, il processo dell'app ha esito negativo, il metodo `StopAsync` potrebbe non essere chiamato.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-138">If the app shuts down unexpectedly (for example, the app's process fails), `StopAsync` might not be called.</span></span> <span data-ttu-id="ec8a1-139">Pertanto è possibile che i metodi chiamati o le operazioni effettuate in `StopAsync` non vengano eseguiti.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-139">Therefore, any methods called or operations conducted in `StopAsync` might not occur.</span></span>

  <span data-ttu-id="ec8a1-140">Per estendere il timeout di arresto predefinito di cinque secondi, impostare:</span><span class="sxs-lookup"><span data-stu-id="ec8a1-140">To extend the default five second shutdown timeout, set:</span></span>

  * <span data-ttu-id="ec8a1-141"><xref:Microsoft.Extensions.Hosting.HostOptions.ShutdownTimeout*> quando si usa l'host generico.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-141"><xref:Microsoft.Extensions.Hosting.HostOptions.ShutdownTimeout*> when using Generic Host.</span></span> <span data-ttu-id="ec8a1-142">Per altre informazioni, vedere <xref:fundamentals/host/generic-host#shutdown-timeout>.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-142">For more information, see <xref:fundamentals/host/generic-host#shutdown-timeout>.</span></span>
  * <span data-ttu-id="ec8a1-143">Impostazione di configurazione dell'host del timeout di arresto quando si usa l'host Web.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-143">Shutdown timeout host configuration setting when using Web Host.</span></span> <span data-ttu-id="ec8a1-144">Per altre informazioni, vedere <xref:fundamentals/host/web-host#shutdown-timeout>.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-144">For more information, see <xref:fundamentals/host/web-host#shutdown-timeout>.</span></span>

<span data-ttu-id="ec8a1-145">Il servizio ospitato viene attivato una volta all'avvio dell'app e arrestato normalmente all'arresto dell'applicazione.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-145">The hosted service is activated once at app startup and gracefully shut down at app shutdown.</span></span> <span data-ttu-id="ec8a1-146">Se viene generato un errore durante l'esecuzione dell'attività in background, deve essere chiamato `Dispose` anche se `StopAsync` non viene chiamato.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-146">If an error is thrown during background task execution, `Dispose` should be called even if `StopAsync` isn't called.</span></span>

## <a name="backgroundservice-base-class"></a><span data-ttu-id="ec8a1-147">Classe di base BackgroundService</span><span class="sxs-lookup"><span data-stu-id="ec8a1-147">BackgroundService base class</span></span>

<span data-ttu-id="ec8a1-148"><xref:Microsoft.Extensions.Hosting.BackgroundService> è una classe di base per l'implementazione di un oggetto a esecuzione prolungata <xref:Microsoft.Extensions.Hosting.IHostedService> .</span><span class="sxs-lookup"><span data-stu-id="ec8a1-148"><xref:Microsoft.Extensions.Hosting.BackgroundService> is a base class for implementing a long running <xref:Microsoft.Extensions.Hosting.IHostedService>.</span></span>

<span data-ttu-id="ec8a1-149">[ExecuteAsync (CancellationToken)](xref:Microsoft.Extensions.Hosting.BackgroundService.ExecuteAsync*) viene chiamato per eseguire il servizio in background.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-149">[ExecuteAsync(CancellationToken)](xref:Microsoft.Extensions.Hosting.BackgroundService.ExecuteAsync*) is called to run the background service.</span></span> <span data-ttu-id="ec8a1-150">L'implementazione restituisce un oggetto <xref:System.Threading.Tasks.Task> che rappresenta l'intera durata del servizio in background.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-150">The implementation returns a <xref:System.Threading.Tasks.Task> that represents the entire lifetime of the background service.</span></span> <span data-ttu-id="ec8a1-151">Non vengono avviati altri servizi fino a quando [ExecuteAsync non diventa asincrono](https://github.com/dotnet/extensions/issues/2149), ad esempio chiamando `await` .</span><span class="sxs-lookup"><span data-stu-id="ec8a1-151">No further services are started until [ExecuteAsync becomes asynchronous](https://github.com/dotnet/extensions/issues/2149), such as by calling `await`.</span></span> <span data-ttu-id="ec8a1-152">Evitare di eseguire operazioni di inizializzazione lunghe e bloccate in `ExecuteAsync` .</span><span class="sxs-lookup"><span data-stu-id="ec8a1-152">Avoid performing long, blocking initialization work in `ExecuteAsync`.</span></span> <span data-ttu-id="ec8a1-153">Blocchi host in [StopAsync (CancellationToken)](xref:Microsoft.Extensions.Hosting.BackgroundService.StopAsync*) in attesa del `ExecuteAsync` completamento di.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-153">The host blocks in [StopAsync(CancellationToken)](xref:Microsoft.Extensions.Hosting.BackgroundService.StopAsync*) waiting for `ExecuteAsync` to complete.</span></span>

<span data-ttu-id="ec8a1-154">Il token di annullamento viene attivato quando viene chiamato [IHostedService. StopAsync](xref:Microsoft.Extensions.Hosting.IHostedService.StopAsync*) .</span><span class="sxs-lookup"><span data-stu-id="ec8a1-154">The cancellation token is triggered when [IHostedService.StopAsync](xref:Microsoft.Extensions.Hosting.IHostedService.StopAsync*) is called.</span></span> <span data-ttu-id="ec8a1-155">L'implementazione di `ExecuteAsync` deve terminare tempestivamente quando viene generato il token di annullamento per arrestare correttamente il servizio.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-155">Your implementation of `ExecuteAsync` should finish promptly when the cancellation token is fired in order to gracefully shut down the service.</span></span> <span data-ttu-id="ec8a1-156">In caso contrario, il servizio si arresta normalmente in corrispondenza del timeout di arresto.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-156">Otherwise, the service ungracefully shuts down at the shutdown timeout.</span></span> <span data-ttu-id="ec8a1-157">Per ulteriori informazioni, vedere la sezione [interfaccia IHostedService](#ihostedservice-interface) .</span><span class="sxs-lookup"><span data-stu-id="ec8a1-157">For more information, see the [IHostedService interface](#ihostedservice-interface) section.</span></span>

## <a name="timed-background-tasks"></a><span data-ttu-id="ec8a1-158">Attività in background programmate</span><span class="sxs-lookup"><span data-stu-id="ec8a1-158">Timed background tasks</span></span>

<span data-ttu-id="ec8a1-159">Un'attività programmata in background utilizza la classe [System.Threading.Timer](xref:System.Threading.Timer).</span><span class="sxs-lookup"><span data-stu-id="ec8a1-159">A timed background task makes use of the [System.Threading.Timer](xref:System.Threading.Timer) class.</span></span> <span data-ttu-id="ec8a1-160">Il timer attiva il metodo `DoWork` dell'attività.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-160">The timer triggers the task's `DoWork` method.</span></span> <span data-ttu-id="ec8a1-161">Il timer viene disabilitato con `StopAsync` ed eliminato quando il contenitore dei servizi è eliminato con `Dispose`:</span><span class="sxs-lookup"><span data-stu-id="ec8a1-161">The timer is disabled on `StopAsync` and disposed when the service container is disposed on `Dispose`:</span></span>

[!code-csharp[](hosted-services/samples/3.x/BackgroundTasksSample/Services/TimedHostedService.cs?name=snippet1&highlight=16-17,34,41)]

<span data-ttu-id="ec8a1-162">Il <xref:System.Threading.Timer> non attende il completamento delle esecuzioni precedenti di `DoWork` , pertanto l'approccio illustrato potrebbe non essere adatto per ogni scenario.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-162">The <xref:System.Threading.Timer> doesn't wait for previous executions of `DoWork` to finish, so the approach shown might not be suitable for every scenario.</span></span> <span data-ttu-id="ec8a1-163">[Interlocked. Increment](xref:System.Threading.Interlocked.Increment*) viene usato per incrementare il contatore di esecuzione come operazione atomica, in modo da garantire che più thread non vengano aggiornati `executionCount` contemporaneamente.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-163">[Interlocked.Increment](xref:System.Threading.Interlocked.Increment*) is used to increment the execution counter as an atomic operation, which ensures that multiple threads don't update `executionCount` concurrently.</span></span>

<span data-ttu-id="ec8a1-164">Il servizio è registrato in `IHostBuilder.ConfigureServices` (*Program.cs*) con il `AddHostedService` metodo di estensione:</span><span class="sxs-lookup"><span data-stu-id="ec8a1-164">The service is registered in `IHostBuilder.ConfigureServices` (*Program.cs*) with the `AddHostedService` extension method:</span></span>

[!code-csharp[](hosted-services/samples/3.x/BackgroundTasksSample/Program.cs?name=snippet1)]

## <a name="consuming-a-scoped-service-in-a-background-task"></a><span data-ttu-id="ec8a1-165">Utilizzo di un servizio con ambito in un'attività in background</span><span class="sxs-lookup"><span data-stu-id="ec8a1-165">Consuming a scoped service in a background task</span></span>

<span data-ttu-id="ec8a1-166">Per usare i [servizi con ambito](xref:fundamentals/dependency-injection#service-lifetimes) all'interno di un [BackgroundService](#backgroundservice-base-class), creare un ambito.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-166">To use [scoped services](xref:fundamentals/dependency-injection#service-lifetimes) within a [BackgroundService](#backgroundservice-base-class), create a scope.</span></span> <span data-ttu-id="ec8a1-167">Non viene creato automaticamente alcun ambito per un servizio ospitato.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-167">No scope is created for a hosted service by default.</span></span>

<span data-ttu-id="ec8a1-168">Il servizio dell'attività in background con ambito contiene la logica dell'attività in background.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-168">The scoped background task service contains the background task's logic.</span></span> <span data-ttu-id="ec8a1-169">Nell'esempio seguente:</span><span class="sxs-lookup"><span data-stu-id="ec8a1-169">In the following example:</span></span>

* <span data-ttu-id="ec8a1-170">Il servizio è asincrono.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-170">The service is asynchronous.</span></span> <span data-ttu-id="ec8a1-171">Il metodo `DoWork` restituisce un oggetto `Task`.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-171">The `DoWork` method returns a `Task`.</span></span> <span data-ttu-id="ec8a1-172">A scopo dimostrativo, nel metodo è atteso un ritardo di dieci secondi `DoWork` .</span><span class="sxs-lookup"><span data-stu-id="ec8a1-172">For demonstration purposes, a delay of ten seconds is awaited in the `DoWork` method.</span></span>
* <span data-ttu-id="ec8a1-173">Un oggetto <xref:Microsoft.Extensions.Logging.ILogger> viene inserito nel servizio.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-173">An <xref:Microsoft.Extensions.Logging.ILogger> is injected into the service.</span></span>

[!code-csharp[](hosted-services/samples/3.x/BackgroundTasksSample/Services/ScopedProcessingService.cs?name=snippet1)]

<span data-ttu-id="ec8a1-174">Il servizio ospitato crea un ambito per risolvere il servizio attività in background con ambito per chiamare il relativo `DoWork` metodo.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-174">The hosted service creates a scope to resolve the scoped background task service to call its `DoWork` method.</span></span> <span data-ttu-id="ec8a1-175">`DoWork` Restituisce un oggetto `Task` , che è atteso in `ExecuteAsync` :</span><span class="sxs-lookup"><span data-stu-id="ec8a1-175">`DoWork` returns a `Task`, which is awaited in `ExecuteAsync`:</span></span>

[!code-csharp[](hosted-services/samples/3.x/BackgroundTasksSample/Services/ConsumeScopedServiceHostedService.cs?name=snippet1&highlight=19,22-35)]

<span data-ttu-id="ec8a1-176">I servizi sono registrati in `IHostBuilder.ConfigureServices` (*Program.cs*).</span><span class="sxs-lookup"><span data-stu-id="ec8a1-176">The services are registered in `IHostBuilder.ConfigureServices` (*Program.cs*).</span></span> <span data-ttu-id="ec8a1-177">Il servizio ospitato viene registrato con il `AddHostedService` metodo di estensione:</span><span class="sxs-lookup"><span data-stu-id="ec8a1-177">The hosted service is registered with the `AddHostedService` extension method:</span></span>

[!code-csharp[](hosted-services/samples/3.x/BackgroundTasksSample/Program.cs?name=snippet2)]

## <a name="queued-background-tasks"></a><span data-ttu-id="ec8a1-178">Attività in background in coda</span><span class="sxs-lookup"><span data-stu-id="ec8a1-178">Queued background tasks</span></span>

<span data-ttu-id="ec8a1-179">Una coda delle attività in background è basata su .NET 4. x <xref:System.Web.Hosting.HostingEnvironment.QueueBackgroundWorkItem*> :</span><span class="sxs-lookup"><span data-stu-id="ec8a1-179">A background task queue is based on the .NET 4.x <xref:System.Web.Hosting.HostingEnvironment.QueueBackgroundWorkItem*>:</span></span>

[!code-csharp[](hosted-services/samples/3.x/BackgroundTasksSample/Services/BackgroundTaskQueue.cs?name=snippet1)]

<span data-ttu-id="ec8a1-180">Nell'esempio seguente `QueueHostedService` :</span><span class="sxs-lookup"><span data-stu-id="ec8a1-180">In the following `QueueHostedService` example:</span></span>

* <span data-ttu-id="ec8a1-181">Il `BackgroundProcessing` metodo restituisce un oggetto `Task` , che è atteso in `ExecuteAsync` .</span><span class="sxs-lookup"><span data-stu-id="ec8a1-181">The `BackgroundProcessing` method returns a `Task`, which is awaited in `ExecuteAsync`.</span></span>
* <span data-ttu-id="ec8a1-182">Le attività in background nella coda vengono rimesse in coda ed eseguite in `BackgroundProcessing` .</span><span class="sxs-lookup"><span data-stu-id="ec8a1-182">Background tasks in the queue are dequeued and executed in `BackgroundProcessing`.</span></span>
* <span data-ttu-id="ec8a1-183">Gli elementi di lavoro sono in attesa prima che il servizio si arresti in `StopAsync` .</span><span class="sxs-lookup"><span data-stu-id="ec8a1-183">Work items are awaited before the service stops in `StopAsync`.</span></span>

[!code-csharp[](hosted-services/samples/3.x/BackgroundTasksSample/Services/QueuedHostedService.cs?name=snippet1&highlight=28-29,33)]

<span data-ttu-id="ec8a1-184">Un `MonitorLoop` servizio gestisce le attività di Accodamento per il servizio ospitato ogni volta che la `w` chiave viene selezionata in un dispositivo di input:</span><span class="sxs-lookup"><span data-stu-id="ec8a1-184">A `MonitorLoop` service handles enqueuing tasks for the hosted service whenever the `w` key is selected on an input device:</span></span>

* <span data-ttu-id="ec8a1-185">`IBackgroundTaskQueue`Viene inserito nel `MonitorLoop` servizio.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-185">The `IBackgroundTaskQueue` is injected into the `MonitorLoop` service.</span></span>
* <span data-ttu-id="ec8a1-186">`IBackgroundTaskQueue.QueueBackgroundWorkItem` viene chiamato per accodare un elemento di lavoro.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-186">`IBackgroundTaskQueue.QueueBackgroundWorkItem` is called to enqueue a work item.</span></span>
* <span data-ttu-id="ec8a1-187">L'elemento di lavoro simula un'attività in background con esecuzione prolungata:</span><span class="sxs-lookup"><span data-stu-id="ec8a1-187">The work item simulates a long-running background task:</span></span>
  * <span data-ttu-id="ec8a1-188">3 5 secondi di ritardo vengono eseguiti ( `Task.Delay` ).</span><span class="sxs-lookup"><span data-stu-id="ec8a1-188">Three 5-second delays are executed (`Task.Delay`).</span></span>
  * <span data-ttu-id="ec8a1-189">`try-catch` <xref:System.OperationCanceledException> Se l'attività viene annullata, viene intercettata un'istruzione.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-189">A `try-catch` statement traps <xref:System.OperationCanceledException> if the task is cancelled.</span></span>

[!code-csharp[](hosted-services/samples/3.x/BackgroundTasksSample/Services/MonitorLoop.cs?name=snippet_Monitor&highlight=7,33)]

<span data-ttu-id="ec8a1-190">I servizi sono registrati in `IHostBuilder.ConfigureServices` (*Program.cs*).</span><span class="sxs-lookup"><span data-stu-id="ec8a1-190">The services are registered in `IHostBuilder.ConfigureServices` (*Program.cs*).</span></span> <span data-ttu-id="ec8a1-191">Il servizio ospitato viene registrato con il `AddHostedService` metodo di estensione:</span><span class="sxs-lookup"><span data-stu-id="ec8a1-191">The hosted service is registered with the `AddHostedService` extension method:</span></span>

[!code-csharp[](hosted-services/samples/3.x/BackgroundTasksSample/Program.cs?name=snippet3)]

<span data-ttu-id="ec8a1-192">`MonitorLoop` viene avviato in `Program.Main` :</span><span class="sxs-lookup"><span data-stu-id="ec8a1-192">`MonitorLoop` is started in `Program.Main`:</span></span>

[!code-csharp[](hosted-services/samples/3.x/BackgroundTasksSample/Program.cs?name=snippet4)]

::: moniker-end

::: moniker range="< aspnetcore-3.0"

<span data-ttu-id="ec8a1-193">In ASP.NET Core le attività in background possono essere implementate come *servizi ospitati*.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-193">In ASP.NET Core, background tasks can be implemented as *hosted services*.</span></span> <span data-ttu-id="ec8a1-194">Un servizio ospitato è una classe con logica di attività in background che implementa l'interfaccia <xref:Microsoft.Extensions.Hosting.IHostedService>.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-194">A hosted service is a class with background task logic that implements the <xref:Microsoft.Extensions.Hosting.IHostedService> interface.</span></span> <span data-ttu-id="ec8a1-195">In questo argomento vengono forniti tre esempi di servizio ospitato:</span><span class="sxs-lookup"><span data-stu-id="ec8a1-195">This topic provides three hosted service examples:</span></span>

* <span data-ttu-id="ec8a1-196">Attività in background eseguita su un timer.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-196">Background task that runs on a timer.</span></span>
* <span data-ttu-id="ec8a1-197">Servizio ospitato che attiva un [servizio con ambito](xref:fundamentals/dependency-injection#service-lifetimes).</span><span class="sxs-lookup"><span data-stu-id="ec8a1-197">Hosted service that activates a [scoped service](xref:fundamentals/dependency-injection#service-lifetimes).</span></span> <span data-ttu-id="ec8a1-198">Il servizio con ambito può usare l' [inserimento di dipendenze](xref:fundamentals/dependency-injection)</span><span class="sxs-lookup"><span data-stu-id="ec8a1-198">The scoped service can use [dependency injection (DI)](xref:fundamentals/dependency-injection)</span></span>
* <span data-ttu-id="ec8a1-199">Attività in background in coda che vengono eseguite in sequenza.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-199">Queued background tasks that run sequentially.</span></span>

<span data-ttu-id="ec8a1-200">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/host/hosted-services/samples/) ([procedura per il download](xref:index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="ec8a1-200">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/host/hosted-services/samples/) ([how to download](xref:index#how-to-download-a-sample))</span></span>

## <a name="package"></a><span data-ttu-id="ec8a1-201">Pacchetto</span><span class="sxs-lookup"><span data-stu-id="ec8a1-201">Package</span></span>

<span data-ttu-id="ec8a1-202">Fare riferimento al [metapacchetto Microsoft.AspNetCore.App](xref:fundamentals/metapackage-app) oppure aggiungere un riferimento al pacchetto [Microsoft.Extensions.Hosting](https://www.nuget.org/packages/Microsoft.Extensions.Hosting).</span><span class="sxs-lookup"><span data-stu-id="ec8a1-202">Reference the [Microsoft.AspNetCore.App metapackage](xref:fundamentals/metapackage-app) or add a package reference to the [Microsoft.Extensions.Hosting](https://www.nuget.org/packages/Microsoft.Extensions.Hosting) package.</span></span>

## <a name="ihostedservice-interface"></a><span data-ttu-id="ec8a1-203">Interfaccia IHostedService</span><span class="sxs-lookup"><span data-stu-id="ec8a1-203">IHostedService interface</span></span>

<span data-ttu-id="ec8a1-204">I servizi ospitati implementano l'interfaccia <xref:Microsoft.Extensions.Hosting.IHostedService>.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-204">Hosted services implement the <xref:Microsoft.Extensions.Hosting.IHostedService> interface.</span></span> <span data-ttu-id="ec8a1-205">L'interfaccia definisce due metodi riservati agli oggetti gestiti dall'host:</span><span class="sxs-lookup"><span data-stu-id="ec8a1-205">The interface defines two methods for objects that are managed by the host:</span></span>

* <span data-ttu-id="ec8a1-206">[StartAsync (CancellationToken)](xref:Microsoft.Extensions.Hosting.IHostedService.StartAsync*): `StartAsync` contiene la logica per avviare l'attività in background.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-206">[StartAsync(CancellationToken)](xref:Microsoft.Extensions.Hosting.IHostedService.StartAsync*): `StartAsync` contains the logic to start the background task.</span></span> <span data-ttu-id="ec8a1-207">Quando si usa l' [host Web](xref:fundamentals/host/web-host), `StartAsync` viene chiamato dopo che il server è stato avviato e viene attivato [IApplicationLifetime. ApplicationStarted](xref:Microsoft.AspNetCore.Hosting.IApplicationLifetime.ApplicationStarted*) .</span><span class="sxs-lookup"><span data-stu-id="ec8a1-207">When using the [Web Host](xref:fundamentals/host/web-host), `StartAsync` is called after the server has started and [IApplicationLifetime.ApplicationStarted](xref:Microsoft.AspNetCore.Hosting.IApplicationLifetime.ApplicationStarted*) is triggered.</span></span> <span data-ttu-id="ec8a1-208">Quando si usa l' [host generico](xref:fundamentals/host/generic-host), `StartAsync` viene chiamato prima di `ApplicationStarted` viene attivato.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-208">When using the [Generic Host](xref:fundamentals/host/generic-host), `StartAsync` is called before `ApplicationStarted` is triggered.</span></span>

* <span data-ttu-id="ec8a1-209">[StopAsync (CancellationToken)](xref:Microsoft.Extensions.Hosting.IHostedService.StopAsync*): attivato quando l'host esegue un arresto normale.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-209">[StopAsync(CancellationToken)](xref:Microsoft.Extensions.Hosting.IHostedService.StopAsync*): Triggered when the host is performing a graceful shutdown.</span></span> <span data-ttu-id="ec8a1-210">`StopAsync` contiene la logica per terminare l'attività in background.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-210">`StopAsync` contains the logic to end the background task.</span></span> <span data-ttu-id="ec8a1-211">Implementare <xref:System.IDisposable> e i [finalizzatori (distruttori)](/dotnet/csharp/programming-guide/classes-and-structs/destructors) per eliminare tutte le risorse non gestite.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-211">Implement <xref:System.IDisposable> and [finalizers (destructors)](/dotnet/csharp/programming-guide/classes-and-structs/destructors) to dispose of any unmanaged resources.</span></span>

  <span data-ttu-id="ec8a1-212">Il token di annullamento ha un timeout predefinito di cinque secondi che indica che il processo di arresto non è più normale.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-212">The cancellation token has a default five second timeout to indicate that the shutdown process should no longer be graceful.</span></span> <span data-ttu-id="ec8a1-213">Quando viene richiesto l'annullamento sul token:</span><span class="sxs-lookup"><span data-stu-id="ec8a1-213">When cancellation is requested on the token:</span></span>

  * <span data-ttu-id="ec8a1-214">Tutte le operazioni in background rimanenti che sta eseguendo l'app devono essere interrotte.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-214">Any remaining background operations that the app is performing should be aborted.</span></span>
  * <span data-ttu-id="ec8a1-215">Tutti i metodi eventuali chiamati in `StopAsync` devono essere completati rapidamente.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-215">Any methods called in `StopAsync` should return promptly.</span></span>

  <span data-ttu-id="ec8a1-216">Tuttavia, dopo la richiesta di annullamento le attività non vengono abbandonate: il chiamante attende il completamento di tutte le attività.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-216">However, tasks aren't abandoned after cancellation is requested&mdash;the caller awaits all tasks to complete.</span></span>

  <span data-ttu-id="ec8a1-217">Se l'app si arresta in modo imprevisto, ad esempio, il processo dell'app ha esito negativo, il metodo `StopAsync` potrebbe non essere chiamato.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-217">If the app shuts down unexpectedly (for example, the app's process fails), `StopAsync` might not be called.</span></span> <span data-ttu-id="ec8a1-218">Pertanto è possibile che i metodi chiamati o le operazioni effettuate in `StopAsync` non vengano eseguiti.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-218">Therefore, any methods called or operations conducted in `StopAsync` might not occur.</span></span>

  <span data-ttu-id="ec8a1-219">Per estendere il timeout di arresto predefinito di cinque secondi, impostare:</span><span class="sxs-lookup"><span data-stu-id="ec8a1-219">To extend the default five second shutdown timeout, set:</span></span>

  * <span data-ttu-id="ec8a1-220"><xref:Microsoft.Extensions.Hosting.HostOptions.ShutdownTimeout*> quando si usa l'host generico.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-220"><xref:Microsoft.Extensions.Hosting.HostOptions.ShutdownTimeout*> when using Generic Host.</span></span> <span data-ttu-id="ec8a1-221">Per altre informazioni, vedere <xref:fundamentals/host/generic-host#shutdown-timeout>.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-221">For more information, see <xref:fundamentals/host/generic-host#shutdown-timeout>.</span></span>
  * <span data-ttu-id="ec8a1-222">Impostazione di configurazione dell'host del timeout di arresto quando si usa l'host Web.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-222">Shutdown timeout host configuration setting when using Web Host.</span></span> <span data-ttu-id="ec8a1-223">Per altre informazioni, vedere <xref:fundamentals/host/web-host#shutdown-timeout>.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-223">For more information, see <xref:fundamentals/host/web-host#shutdown-timeout>.</span></span>

<span data-ttu-id="ec8a1-224">Il servizio ospitato viene attivato una volta all'avvio dell'app e arrestato normalmente all'arresto dell'applicazione.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-224">The hosted service is activated once at app startup and gracefully shut down at app shutdown.</span></span> <span data-ttu-id="ec8a1-225">Se viene generato un errore durante l'esecuzione dell'attività in background, deve essere chiamato `Dispose` anche se `StopAsync` non viene chiamato.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-225">If an error is thrown during background task execution, `Dispose` should be called even if `StopAsync` isn't called.</span></span>

## <a name="timed-background-tasks"></a><span data-ttu-id="ec8a1-226">Attività in background programmate</span><span class="sxs-lookup"><span data-stu-id="ec8a1-226">Timed background tasks</span></span>

<span data-ttu-id="ec8a1-227">Un'attività programmata in background utilizza la classe [System.Threading.Timer](xref:System.Threading.Timer).</span><span class="sxs-lookup"><span data-stu-id="ec8a1-227">A timed background task makes use of the [System.Threading.Timer](xref:System.Threading.Timer) class.</span></span> <span data-ttu-id="ec8a1-228">Il timer attiva il metodo `DoWork` dell'attività.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-228">The timer triggers the task's `DoWork` method.</span></span> <span data-ttu-id="ec8a1-229">Il timer viene disabilitato con `StopAsync` ed eliminato quando il contenitore dei servizi è eliminato con `Dispose`:</span><span class="sxs-lookup"><span data-stu-id="ec8a1-229">The timer is disabled on `StopAsync` and disposed when the service container is disposed on `Dispose`:</span></span>

[!code-csharp[](hosted-services/samples/2.x/BackgroundTasksSample/Services/TimedHostedService.cs?name=snippet1&highlight=15-16,30,37)]

<span data-ttu-id="ec8a1-230">Il <xref:System.Threading.Timer> non attende il completamento delle esecuzioni precedenti di `DoWork` , pertanto l'approccio illustrato potrebbe non essere adatto per ogni scenario.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-230">The <xref:System.Threading.Timer> doesn't wait for previous executions of `DoWork` to finish, so the approach shown might not be suitable for every scenario.</span></span>

<span data-ttu-id="ec8a1-231">Il servizio registrato in `Startup.ConfigureServices` con il metodo di estensione `AddHostedService`:</span><span class="sxs-lookup"><span data-stu-id="ec8a1-231">The service is registered in `Startup.ConfigureServices` with the `AddHostedService` extension method:</span></span>

[!code-csharp[](hosted-services/samples/2.x/BackgroundTasksSample/Startup.cs?name=snippet1)]

## <a name="consuming-a-scoped-service-in-a-background-task"></a><span data-ttu-id="ec8a1-232">Utilizzo di un servizio con ambito in un'attività in background</span><span class="sxs-lookup"><span data-stu-id="ec8a1-232">Consuming a scoped service in a background task</span></span>

<span data-ttu-id="ec8a1-233">Per usare i [servizi con ambito](xref:fundamentals/dependency-injection#service-lifetimes) all'interno di un `IHostedService` , creare un ambito.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-233">To use [scoped services](xref:fundamentals/dependency-injection#service-lifetimes) within an `IHostedService`, create a scope.</span></span> <span data-ttu-id="ec8a1-234">Non viene creato automaticamente alcun ambito per un servizio ospitato.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-234">No scope is created for a hosted service by default.</span></span>

<span data-ttu-id="ec8a1-235">Il servizio dell'attività in background con ambito contiene la logica dell'attività in background.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-235">The scoped background task service contains the background task's logic.</span></span> <span data-ttu-id="ec8a1-236">Nell'esempio seguente, <xref:Microsoft.Extensions.Logging.ILogger> viene inserito nel servizio:</span><span class="sxs-lookup"><span data-stu-id="ec8a1-236">In the following example, an <xref:Microsoft.Extensions.Logging.ILogger> is injected into the service:</span></span>

[!code-csharp[](hosted-services/samples/2.x/BackgroundTasksSample/Services/ScopedProcessingService.cs?name=snippet1)]

<span data-ttu-id="ec8a1-237">Il servizio ospitato crea un ambito per risolvere il servizio dell'attività in background con ambito e chiamare il relativo metodo `DoWork`:</span><span class="sxs-lookup"><span data-stu-id="ec8a1-237">The hosted service creates a scope to resolve the scoped background task service to call its `DoWork` method:</span></span>

[!code-csharp[](hosted-services/samples/2.x/BackgroundTasksSample/Services/ConsumeScopedServiceHostedService.cs?name=snippet1&highlight=29-36)]

<span data-ttu-id="ec8a1-238">I servizi vengono registrati in `Startup.ConfigureServices`.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-238">The services are registered in `Startup.ConfigureServices`.</span></span> <span data-ttu-id="ec8a1-239">L'implementazione di `IHostedService`viene registrata con il metodo di estensione `AddHostedService`:</span><span class="sxs-lookup"><span data-stu-id="ec8a1-239">The `IHostedService` implementation is registered with the `AddHostedService` extension method:</span></span>

[!code-csharp[](hosted-services/samples/2.x/BackgroundTasksSample/Startup.cs?name=snippet2)]

## <a name="queued-background-tasks"></a><span data-ttu-id="ec8a1-240">Attività in background in coda</span><span class="sxs-lookup"><span data-stu-id="ec8a1-240">Queued background tasks</span></span>

<span data-ttu-id="ec8a1-241">Una coda delle attività in background è basata sulla .NET Framework 4. x <xref:System.Web.Hosting.HostingEnvironment.QueueBackgroundWorkItem*> ([pianificata provvisoriamente per essere incorporata per ASP.NET Core](https://github.com/aspnet/Hosting/issues/1280)):</span><span class="sxs-lookup"><span data-stu-id="ec8a1-241">A background task queue is based on the .NET Framework 4.x <xref:System.Web.Hosting.HostingEnvironment.QueueBackgroundWorkItem*> ([tentatively scheduled to be built-in for ASP.NET Core](https://github.com/aspnet/Hosting/issues/1280)):</span></span>

[!code-csharp[](hosted-services/samples/2.x/BackgroundTasksSample/Services/BackgroundTaskQueue.cs?name=snippet1)]

<span data-ttu-id="ec8a1-242">In `QueueHostedService`, le attività in background in coda vengono rimosse dalla coda ed eseguite come [BackgroundService](#backgroundservice-base-class), ovvero una classe di base per l'implementazione di un `IHostedService` a esecuzione prolungata:</span><span class="sxs-lookup"><span data-stu-id="ec8a1-242">In `QueueHostedService`, background tasks in the queue are dequeued and executed as a [BackgroundService](#backgroundservice-base-class), which is a base class for implementing a long running `IHostedService`:</span></span>

[!code-csharp[](hosted-services/samples/2.x/BackgroundTasksSample/Services/QueuedHostedService.cs?name=snippet1&highlight=21,25)]

<span data-ttu-id="ec8a1-243">I servizi vengono registrati in `Startup.ConfigureServices`.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-243">The services are registered in `Startup.ConfigureServices`.</span></span> <span data-ttu-id="ec8a1-244">L'implementazione di `IHostedService`viene registrata con il metodo di estensione `AddHostedService`:</span><span class="sxs-lookup"><span data-stu-id="ec8a1-244">The `IHostedService` implementation is registered with the `AddHostedService` extension method:</span></span>

[!code-csharp[](hosted-services/samples/2.x/BackgroundTasksSample/Startup.cs?name=snippet3)]

<span data-ttu-id="ec8a1-245">Nella classe modello della pagina di indice:</span><span class="sxs-lookup"><span data-stu-id="ec8a1-245">In the Index page model class:</span></span>

* <span data-ttu-id="ec8a1-246">`IBackgroundTaskQueue` viene inserito nel costruttore e assegnato a `Queue`.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-246">The `IBackgroundTaskQueue` is injected into the constructor and assigned to `Queue`.</span></span>
* <span data-ttu-id="ec8a1-247">Un'interfaccia <xref:Microsoft.Extensions.DependencyInjection.IServiceScopeFactory> viene inserita e assegnata a `_serviceScopeFactory`.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-247">An <xref:Microsoft.Extensions.DependencyInjection.IServiceScopeFactory> is injected and assigned to `_serviceScopeFactory`.</span></span> <span data-ttu-id="ec8a1-248">La factory viene usata per creare istanze di <xref:Microsoft.Extensions.DependencyInjection.IServiceScope>, che consente di creare servizi all'interno di un ambito.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-248">The factory is used to create instances of <xref:Microsoft.Extensions.DependencyInjection.IServiceScope>, which is used to create services within a scope.</span></span> <span data-ttu-id="ec8a1-249">Viene creato un ambito per usare la classe `AppDbContext` dell'app (un [servizio con ambito](xref:fundamentals/dependency-injection#service-lifetimes)) per scrivere record di database in `IBackgroundTaskQueue` (un servizio singleton).</span><span class="sxs-lookup"><span data-stu-id="ec8a1-249">A scope is created in order to use the app's `AppDbContext` (a [scoped service](xref:fundamentals/dependency-injection#service-lifetimes)) to write database records in the `IBackgroundTaskQueue` (a singleton service).</span></span>

[!code-csharp[](hosted-services/samples/2.x/BackgroundTasksSample/Pages/Index.cshtml.cs?name=snippet1)]

<span data-ttu-id="ec8a1-250">Quando si seleziona il pulsante **Aggiungi attività** nella pagina di indice, viene eseguito il metodo `OnPostAddTask`.</span><span class="sxs-lookup"><span data-stu-id="ec8a1-250">When the **Add Task** button is selected on the Index page, the `OnPostAddTask` method is executed.</span></span> <span data-ttu-id="ec8a1-251">`QueueBackgroundWorkItem` viene chiamato per accodare un elemento di lavoro:</span><span class="sxs-lookup"><span data-stu-id="ec8a1-251">`QueueBackgroundWorkItem` is called to enqueue a work item:</span></span>

[!code-csharp[](hosted-services/samples/2.x/BackgroundTasksSample/Pages/Index.cshtml.cs?name=snippet2)]

::: moniker-end

## <a name="additional-resources"></a><span data-ttu-id="ec8a1-252">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="ec8a1-252">Additional resources</span></span>

* [<span data-ttu-id="ec8a1-253">Implementare attività in background in microservizi con IHostedService e la classe BackgroundService</span><span class="sxs-lookup"><span data-stu-id="ec8a1-253">Implement background tasks in microservices with IHostedService and the BackgroundService class</span></span>](/dotnet/standard/microservices-architecture/multi-container-microservice-net-applications/background-tasks-with-ihostedservice)
* [<span data-ttu-id="ec8a1-254">Eseguire attività in background con processi Web nel servizio app Azure</span><span class="sxs-lookup"><span data-stu-id="ec8a1-254">Run background tasks with WebJobs in Azure App Service</span></span>](/azure/app-service/webjobs-create)
* <xref:System.Threading.Timer>
