---
title: Host generico .NET in ASP.NET Core
author: rick-anderson
description: Usare l'host generico di .NET Core nelle app ASP.NET Core.  L'host generico è responsabile della gestione dell'avvio e della durata delle app.
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 4/17/2020
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
uid: fundamentals/host/generic-host
ms.openlocfilehash: ceb766cae1f8a1735f5b578a44f0c481927e47a5
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102586748"
---
# <a name="net-generic-host-in-aspnet-core"></a><span data-ttu-id="a2704-104">Host generico .NET in ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="a2704-104">.NET Generic Host in ASP.NET Core</span></span>

::: moniker range=">= aspnetcore-5.0"

<span data-ttu-id="a2704-105">I modelli di ASP.NET Core creano un host generico .NET Core ( <xref:Microsoft.Extensions.Hosting.HostBuilder> ).</span><span class="sxs-lookup"><span data-stu-id="a2704-105">The ASP.NET Core templates create a .NET Core Generic Host (<xref:Microsoft.Extensions.Hosting.HostBuilder>).</span></span>

<span data-ttu-id="a2704-106">In questo argomento vengono fornite informazioni sull'utilizzo di host generico .NET in ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="a2704-106">This topic provides information on using .NET Generic Host in ASP.NET Core.</span></span> <span data-ttu-id="a2704-107">Per informazioni sull'uso di un host generico .NET nelle app console, vedere [.NET Generic Host](/dotnet/core/extensions/generic-host).</span><span class="sxs-lookup"><span data-stu-id="a2704-107">For information on using .NET Generic Host in console apps, see [.NET Generic Host](/dotnet/core/extensions/generic-host).</span></span>

## <a name="host-definition"></a><span data-ttu-id="a2704-108">Definizione host</span><span class="sxs-lookup"><span data-stu-id="a2704-108">Host definition</span></span>

<span data-ttu-id="a2704-109">Un *host* è un oggetto che incapsula le risorse di un'app, ad esempio:</span><span class="sxs-lookup"><span data-stu-id="a2704-109">A *host* is an object that encapsulates an app's resources, such as:</span></span>

* <span data-ttu-id="a2704-110">Inserimento di dipendenze (DI)</span><span class="sxs-lookup"><span data-stu-id="a2704-110">Dependency injection (DI)</span></span>
* <span data-ttu-id="a2704-111">Registrazione</span><span class="sxs-lookup"><span data-stu-id="a2704-111">Logging</span></span>
* <span data-ttu-id="a2704-112">Configurazione</span><span class="sxs-lookup"><span data-stu-id="a2704-112">Configuration</span></span>
* <span data-ttu-id="a2704-113">`IHostedService` Implementazioni</span><span class="sxs-lookup"><span data-stu-id="a2704-113">`IHostedService` implementations</span></span>

<span data-ttu-id="a2704-114">Quando viene avviato, un host chiama <xref:Microsoft.Extensions.Hosting.IHostedService.StartAsync%2A?displayProperty=nameWithType> su ogni implementazione di <xref:Microsoft.Extensions.Hosting.IHostedService> registrata nella raccolta di servizi ospitati del contenitore dei servizi.</span><span class="sxs-lookup"><span data-stu-id="a2704-114">When a host starts, it calls <xref:Microsoft.Extensions.Hosting.IHostedService.StartAsync%2A?displayProperty=nameWithType> on each implementation of <xref:Microsoft.Extensions.Hosting.IHostedService> registered in the service container's collection of hosted services.</span></span> <span data-ttu-id="a2704-115">In un'app Web, una delle implementazioni di `IHostedService` è un servizio web che avvia un'[implementazione del server HTTP](xref:fundamentals/index#servers).</span><span class="sxs-lookup"><span data-stu-id="a2704-115">In a web app, one of the `IHostedService` implementations is a web service that starts an [HTTP server implementation](xref:fundamentals/index#servers).</span></span>

<span data-ttu-id="a2704-116">Il motivo principale per cui tutte le risorse interdipendenti dell'app sono incluse in un unico oggetto è la gestione del ciclo di vita, vale a dire il controllo sull'avvio dell'app e sull'arresto normale.</span><span class="sxs-lookup"><span data-stu-id="a2704-116">The main reason for including all of the app's interdependent resources in one object is lifetime management: control over app startup and graceful shutdown.</span></span>

## <a name="set-up-a-host"></a><span data-ttu-id="a2704-117">Configurare un host</span><span class="sxs-lookup"><span data-stu-id="a2704-117">Set up a host</span></span>

<span data-ttu-id="a2704-118">L'host è in genere configurato, compilato ed eseguito da codice nella classe `Program`.</span><span class="sxs-lookup"><span data-stu-id="a2704-118">The host is typically configured, built, and run by code in the `Program` class.</span></span> <span data-ttu-id="a2704-119">Il metodo `Main`:</span><span class="sxs-lookup"><span data-stu-id="a2704-119">The `Main` method:</span></span>

* <span data-ttu-id="a2704-120">Chiama un metodo `CreateHostBuilder` per creare e configurare un oggetto generatore.</span><span class="sxs-lookup"><span data-stu-id="a2704-120">Calls a `CreateHostBuilder` method to create and configure a builder object.</span></span>
* <span data-ttu-id="a2704-121">Chiamate `Build` e metodi `Run` nell'oggetto generatore.</span><span class="sxs-lookup"><span data-stu-id="a2704-121">Calls `Build` and `Run` methods on the builder object.</span></span>

<span data-ttu-id="a2704-122">I modelli Web ASP.NET Core generano il codice seguente per creare un host:</span><span class="sxs-lookup"><span data-stu-id="a2704-122">The ASP.NET Core web templates generate the following code to create a host:</span></span>

```csharp
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
            });
}
```

<span data-ttu-id="a2704-123">Il codice seguente crea un carico di lavoro non HTTP con un' `IHostedService` implementazione aggiunta al contenitore di inserimento delle dipendenze.</span><span class="sxs-lookup"><span data-stu-id="a2704-123">The following code creates a non-HTTP workload with an `IHostedService` implementation added to the DI container.</span></span>

```csharp
public class Program
{
    public static void Main(string[] args)
    {
        CreateHostBuilder(args).Build().Run();
    }

    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .ConfigureServices((hostContext, services) =>
            {
               services.AddHostedService<Worker>();
            });
}
```

<span data-ttu-id="a2704-124">Per un carico di lavoro HTTP, il metodo `Main` è lo stesso ma `CreateHostBuilder` chiama `ConfigureWebHostDefaults`:</span><span class="sxs-lookup"><span data-stu-id="a2704-124">For an HTTP workload, the `Main` method is the same but `CreateHostBuilder` calls `ConfigureWebHostDefaults`:</span></span>

```csharp
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.UseStartup<Startup>();
        });
```

<span data-ttu-id="a2704-125">Se l'app usa Entity Framework Core, non modificare il nome o la firma del metodo `CreateHostBuilder`.</span><span class="sxs-lookup"><span data-stu-id="a2704-125">If the app uses Entity Framework Core, don't change the name or signature of the `CreateHostBuilder` method.</span></span> <span data-ttu-id="a2704-126">Gli [strumenti di Entity Framework Core](/ef/core/miscellaneous/cli/) si aspettano di trovare un metodo `CreateHostBuilder` che configura l'host senza eseguire l'app.</span><span class="sxs-lookup"><span data-stu-id="a2704-126">The [Entity Framework Core tools](/ef/core/miscellaneous/cli/) expect to find a `CreateHostBuilder` method that configures the host without running the app.</span></span> <span data-ttu-id="a2704-127">Per altre informazioni, vedere [Creazione DbContext in fase di progettazione](/ef/core/miscellaneous/cli/dbcontext-creation).</span><span class="sxs-lookup"><span data-stu-id="a2704-127">For more information, see [Design-time DbContext Creation](/ef/core/miscellaneous/cli/dbcontext-creation).</span></span>

## <a name="default-builder-settings"></a><span data-ttu-id="a2704-128">Impostazioni predefinite del generatore</span><span class="sxs-lookup"><span data-stu-id="a2704-128">Default builder settings</span></span>

<span data-ttu-id="a2704-129">Il metodo <xref:Microsoft.Extensions.Hosting.Host.CreateDefaultBuilder*>:</span><span class="sxs-lookup"><span data-stu-id="a2704-129">The <xref:Microsoft.Extensions.Hosting.Host.CreateDefaultBuilder*> method:</span></span>

* <span data-ttu-id="a2704-130">Imposta la [radice del contenuto](xref:fundamentals/index#content-root) sul percorso restituito da <xref:System.IO.Directory.GetCurrentDirectory*> .</span><span class="sxs-lookup"><span data-stu-id="a2704-130">Sets the [content root](xref:fundamentals/index#content-root) to the path returned by <xref:System.IO.Directory.GetCurrentDirectory*>.</span></span>
* <span data-ttu-id="a2704-131">Carica la configurazione dell'host da:</span><span class="sxs-lookup"><span data-stu-id="a2704-131">Loads host configuration from:</span></span>
  * <span data-ttu-id="a2704-132">Variabili di ambiente con prefisso `DOTNET_` .</span><span class="sxs-lookup"><span data-stu-id="a2704-132">Environment variables prefixed with `DOTNET_`.</span></span>
  * <span data-ttu-id="a2704-133">Argomenti della riga di comando.</span><span class="sxs-lookup"><span data-stu-id="a2704-133">Command-line arguments.</span></span>
* <span data-ttu-id="a2704-134">Carica la configurazione dell'app da:</span><span class="sxs-lookup"><span data-stu-id="a2704-134">Loads app configuration from:</span></span>
  * <span data-ttu-id="a2704-135">*appsettings.json*.</span><span class="sxs-lookup"><span data-stu-id="a2704-135">*appsettings.json*.</span></span>
  * <span data-ttu-id="a2704-136">*appsettings.{Environment}.json*.</span><span class="sxs-lookup"><span data-stu-id="a2704-136">*appsettings.{Environment}.json*.</span></span>
  * <span data-ttu-id="a2704-137">[Segreti dell'utente](xref:security/app-secrets) quando l'app viene eseguita nell'ambiente `Development`.</span><span class="sxs-lookup"><span data-stu-id="a2704-137">[User secrets](xref:security/app-secrets) when the app runs in the `Development` environment.</span></span>
  * <span data-ttu-id="a2704-138">Variabili di ambiente.</span><span class="sxs-lookup"><span data-stu-id="a2704-138">Environment variables.</span></span>
  * <span data-ttu-id="a2704-139">Argomenti della riga di comando.</span><span class="sxs-lookup"><span data-stu-id="a2704-139">Command-line arguments.</span></span>
* <span data-ttu-id="a2704-140">Aggiunge i provider di [log](xref:fundamentals/logging/index) seguenti:</span><span class="sxs-lookup"><span data-stu-id="a2704-140">Adds the following [logging](xref:fundamentals/logging/index) providers:</span></span>
  * <span data-ttu-id="a2704-141">Console</span><span class="sxs-lookup"><span data-stu-id="a2704-141">Console</span></span>
  * <span data-ttu-id="a2704-142">Debug</span><span class="sxs-lookup"><span data-stu-id="a2704-142">Debug</span></span>
  * <span data-ttu-id="a2704-143">EventSource</span><span class="sxs-lookup"><span data-stu-id="a2704-143">EventSource</span></span>
  * <span data-ttu-id="a2704-144">EventLog (solo quando è in esecuzione su Windows)</span><span class="sxs-lookup"><span data-stu-id="a2704-144">EventLog (only when running on Windows)</span></span>
* <span data-ttu-id="a2704-145">Abilita la [convalida dell'ambito](xref:fundamentals/dependency-injection#scope-validation) e la [convalida delle dipendenze](xref:Microsoft.Extensions.DependencyInjection.ServiceProviderOptions.ValidateOnBuild) quando l'ambiente è lo sviluppo.</span><span class="sxs-lookup"><span data-stu-id="a2704-145">Enables [scope validation](xref:fundamentals/dependency-injection#scope-validation) and [dependency validation](xref:Microsoft.Extensions.DependencyInjection.ServiceProviderOptions.ValidateOnBuild) when the environment is Development.</span></span>

<span data-ttu-id="a2704-146">Il metodo <xref:Microsoft.Extensions.Hosting.GenericHostBuilderExtensions.ConfigureWebHostDefaults*>:</span><span class="sxs-lookup"><span data-stu-id="a2704-146">The <xref:Microsoft.Extensions.Hosting.GenericHostBuilderExtensions.ConfigureWebHostDefaults*> method:</span></span>

* <span data-ttu-id="a2704-147">Carica la configurazione host dalle variabili di ambiente con prefisso `ASPNETCORE_` .</span><span class="sxs-lookup"><span data-stu-id="a2704-147">Loads host configuration from environment variables prefixed with `ASPNETCORE_`.</span></span>
* <span data-ttu-id="a2704-148">Imposta il server [Kestrel](xref:fundamentals/servers/kestrel) come server Web e lo configura usando i provider di configurazione dell'host dell'app.</span><span class="sxs-lookup"><span data-stu-id="a2704-148">Sets [Kestrel](xref:fundamentals/servers/kestrel) server as the web server and configures it using the app's hosting configuration providers.</span></span> <span data-ttu-id="a2704-149">Per le opzioni predefinite del server Kestrel, vedere <xref:fundamentals/servers/kestrel/options>.</span><span class="sxs-lookup"><span data-stu-id="a2704-149">For the Kestrel server's default options, see <xref:fundamentals/servers/kestrel/options>.</span></span>
* <span data-ttu-id="a2704-150">Aggiunge [il middleware dell'applicazione di filtri dell'host](xref:fundamentals/servers/kestrel/host-filtering).</span><span class="sxs-lookup"><span data-stu-id="a2704-150">Adds [Host Filtering middleware](xref:fundamentals/servers/kestrel/host-filtering).</span></span>
* <span data-ttu-id="a2704-151">Aggiunge il [middleware delle intestazioni con inoltri](xref:host-and-deploy/proxy-load-balancer#forwarded-headers) se `ASPNETCORE_FORWARDEDHEADERS_ENABLED` è uguale a `true` .</span><span class="sxs-lookup"><span data-stu-id="a2704-151">Adds [Forwarded Headers middleware](xref:host-and-deploy/proxy-load-balancer#forwarded-headers) if `ASPNETCORE_FORWARDEDHEADERS_ENABLED` equals `true`.</span></span>
* <span data-ttu-id="a2704-152">Abilita l'integrazione di IIS.</span><span class="sxs-lookup"><span data-stu-id="a2704-152">Enables IIS integration.</span></span> <span data-ttu-id="a2704-153">Per le opzioni predefinite di IIS, vedere <xref:host-and-deploy/iis/index#iis-options>.</span><span class="sxs-lookup"><span data-stu-id="a2704-153">For the IIS default options, see <xref:host-and-deploy/iis/index#iis-options>.</span></span>

<span data-ttu-id="a2704-154">Le sezioni [Impostazioni per tutti i tipi di app](#settings-for-all-app-types) e [Impostazioni per le app Web](#settings-for-web-apps) più avanti in questo articolo mostrano come eseguire l'override delle impostazioni predefinite del generatore.</span><span class="sxs-lookup"><span data-stu-id="a2704-154">The [Settings for all app types](#settings-for-all-app-types) and [Settings for web apps](#settings-for-web-apps) sections later in this article show how to override default builder settings.</span></span>

## <a name="framework-provided-services"></a><span data-ttu-id="a2704-155">Servizi forniti dal framework</span><span class="sxs-lookup"><span data-stu-id="a2704-155">Framework-provided services</span></span>

<span data-ttu-id="a2704-156">I servizi seguenti vengono registrati automaticamente:</span><span class="sxs-lookup"><span data-stu-id="a2704-156">The following services are registered automatically:</span></span>

* [<span data-ttu-id="a2704-157">IHostApplicationLifetime</span><span class="sxs-lookup"><span data-stu-id="a2704-157">IHostApplicationLifetime</span></span>](#ihostapplicationlifetime)
* [<span data-ttu-id="a2704-158">IHostLifetime</span><span class="sxs-lookup"><span data-stu-id="a2704-158">IHostLifetime</span></span>](#ihostlifetime)
* [<span data-ttu-id="a2704-159">IHostEnvironment / IWebHostEnvironment</span><span class="sxs-lookup"><span data-stu-id="a2704-159">IHostEnvironment / IWebHostEnvironment</span></span>](#ihostenvironment)

<span data-ttu-id="a2704-160">Per ulteriori informazioni sui servizi forniti dal Framework, vedere <xref:fundamentals/dependency-injection#framework-provided-services> .</span><span class="sxs-lookup"><span data-stu-id="a2704-160">For more information on framework-provided services, see <xref:fundamentals/dependency-injection#framework-provided-services>.</span></span>

## <a name="ihostapplicationlifetime"></a><span data-ttu-id="a2704-161">IHostApplicationLifetime</span><span class="sxs-lookup"><span data-stu-id="a2704-161">IHostApplicationLifetime</span></span>

<span data-ttu-id="a2704-162">Inserire il servizio <xref:Microsoft.Extensions.Hosting.IHostApplicationLifetime> (in precedenza `IApplicationLifetime`) in qualsiasi classe per gestire le attività post-avvio e di arresto normale.</span><span class="sxs-lookup"><span data-stu-id="a2704-162">Inject the <xref:Microsoft.Extensions.Hosting.IHostApplicationLifetime> (formerly `IApplicationLifetime`) service into any class to handle post-startup and graceful shutdown tasks.</span></span> <span data-ttu-id="a2704-163">Tre proprietà nell'interfaccia sono token di annullamento usati per registrare i metodi del gestore dell'evento di avvio e di arresto.</span><span class="sxs-lookup"><span data-stu-id="a2704-163">Three properties on the interface are cancellation tokens used to register app start and app stop event handler methods.</span></span> <span data-ttu-id="a2704-164">L'interfaccia include anche un metodo `StopApplication`.</span><span class="sxs-lookup"><span data-stu-id="a2704-164">The interface also includes a `StopApplication` method.</span></span>

<span data-ttu-id="a2704-165">L'esempio seguente è un' `IHostedService` implementazione di che registra `IHostApplicationLifetime` gli eventi:</span><span class="sxs-lookup"><span data-stu-id="a2704-165">The following example is an `IHostedService` implementation that registers `IHostApplicationLifetime` events:</span></span>

[!code-csharp[](generic-host/samples-snapshot/3.x/LifetimeEventsHostedService.cs?name=snippet_LifetimeEvents)]

## <a name="ihostlifetime"></a><span data-ttu-id="a2704-166">IHostLifetime</span><span class="sxs-lookup"><span data-stu-id="a2704-166">IHostLifetime</span></span>

<span data-ttu-id="a2704-167">L'implementazione <xref:Microsoft.Extensions.Hosting.IHostLifetime> controlla quando l'host viene avviato e quando si arresta.</span><span class="sxs-lookup"><span data-stu-id="a2704-167">The <xref:Microsoft.Extensions.Hosting.IHostLifetime> implementation controls when the host starts and when it stops.</span></span> <span data-ttu-id="a2704-168">Viene usata l'ultima implementazione registrata.</span><span class="sxs-lookup"><span data-stu-id="a2704-168">The last implementation registered is used.</span></span>

<span data-ttu-id="a2704-169">`Microsoft.Extensions.Hosting.Internal.ConsoleLifetime` è l'implementazione `IHostLifetime` predefinita.</span><span class="sxs-lookup"><span data-stu-id="a2704-169">`Microsoft.Extensions.Hosting.Internal.ConsoleLifetime` is the default `IHostLifetime` implementation.</span></span> <span data-ttu-id="a2704-170">`ConsoleLifetime`:</span><span class="sxs-lookup"><span data-stu-id="a2704-170">`ConsoleLifetime`:</span></span>

* <span data-ttu-id="a2704-171">Ascolta la <kbd>combinazione di tasti CTRL</kbd> + <kbd>C</kbd>/SIGINT o SIGTERM e chiama <xref:Microsoft.Extensions.Hosting.IHostApplicationLifetime.StopApplication*> per avviare il processo di arresto.</span><span class="sxs-lookup"><span data-stu-id="a2704-171">Listens for <kbd>Ctrl</kbd>+<kbd>C</kbd>/SIGINT or SIGTERM and calls <xref:Microsoft.Extensions.Hosting.IHostApplicationLifetime.StopApplication*> to start the shutdown process.</span></span>
* <span data-ttu-id="a2704-172">Sblocca estensioni come [RunAsync](#runasync) e [WaitForShutdownAsync](#waitforshutdownasync).</span><span class="sxs-lookup"><span data-stu-id="a2704-172">Unblocks extensions such as [RunAsync](#runasync) and [WaitForShutdownAsync](#waitforshutdownasync).</span></span>

## <a name="ihostenvironment"></a><span data-ttu-id="a2704-173">IHostEnvironment</span><span class="sxs-lookup"><span data-stu-id="a2704-173">IHostEnvironment</span></span>

<span data-ttu-id="a2704-174">Inserire il <xref:Microsoft.Extensions.Hosting.IHostEnvironment> servizio in una classe per ottenere informazioni sulle impostazioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="a2704-174">Inject the <xref:Microsoft.Extensions.Hosting.IHostEnvironment> service into a class to get information about the following settings:</span></span>

* [<span data-ttu-id="a2704-175">ApplicationName</span><span class="sxs-lookup"><span data-stu-id="a2704-175">ApplicationName</span></span>](#applicationname)
* [<span data-ttu-id="a2704-176">EnvironmentName</span><span class="sxs-lookup"><span data-stu-id="a2704-176">EnvironmentName</span></span>](#environmentname)
* [<span data-ttu-id="a2704-177">ContentRootPath</span><span class="sxs-lookup"><span data-stu-id="a2704-177">ContentRootPath</span></span>](#contentroot)

<span data-ttu-id="a2704-178">Le app Web implementano l' `IWebHostEnvironment` interfaccia, che eredita `IHostEnvironment` e aggiunge [WebRootPath](#webroot).</span><span class="sxs-lookup"><span data-stu-id="a2704-178">Web apps implement the `IWebHostEnvironment` interface, which inherits `IHostEnvironment` and adds the [WebRootPath](#webroot).</span></span>

## <a name="host-configuration"></a><span data-ttu-id="a2704-179">Configurazione dell'host</span><span class="sxs-lookup"><span data-stu-id="a2704-179">Host configuration</span></span>

<span data-ttu-id="a2704-180">La configurazione dell'host viene usata per le proprietà dell'implementazione <xref:Microsoft.Extensions.Hosting.IHostEnvironment>.</span><span class="sxs-lookup"><span data-stu-id="a2704-180">Host configuration is used for the properties of the <xref:Microsoft.Extensions.Hosting.IHostEnvironment> implementation.</span></span>

<span data-ttu-id="a2704-181">La configurazione dell'host è disponibile da [HostBuilderContext.Configuration](xref:Microsoft.Extensions.Hosting.HostBuilderContext.Configuration) all'interno di <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureAppConfiguration*>.</span><span class="sxs-lookup"><span data-stu-id="a2704-181">Host configuration is available from [HostBuilderContext.Configuration](xref:Microsoft.Extensions.Hosting.HostBuilderContext.Configuration) inside <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureAppConfiguration*>.</span></span> <span data-ttu-id="a2704-182">Dopo `ConfigureAppConfiguration`, `HostBuilderContext.Configuration` viene sostituito con la configurazione dell'app.</span><span class="sxs-lookup"><span data-stu-id="a2704-182">After `ConfigureAppConfiguration`, `HostBuilderContext.Configuration` is replaced with the app config.</span></span>

<span data-ttu-id="a2704-183">Per aggiungere la configurazione dell'host, chiamare <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureHostConfiguration*> su `IHostBuilder`.</span><span class="sxs-lookup"><span data-stu-id="a2704-183">To add host configuration, call <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureHostConfiguration*> on `IHostBuilder`.</span></span> <span data-ttu-id="a2704-184">`ConfigureHostConfiguration` può essere chiamato più volte e i risultati si sommano.</span><span class="sxs-lookup"><span data-stu-id="a2704-184">`ConfigureHostConfiguration` can be called multiple times with additive results.</span></span> <span data-ttu-id="a2704-185">L'host usa l'ultima opzione che imposta un valore in una determinata chiave.</span><span class="sxs-lookup"><span data-stu-id="a2704-185">The host uses whichever option sets a value last on a given key.</span></span>

<span data-ttu-id="a2704-186">Il provider di variabili di ambiente con prefisso `DOTNET_` e argomenti della riga di comando sono inclusi in `CreateDefaultBuilder` .</span><span class="sxs-lookup"><span data-stu-id="a2704-186">The environment variable provider with prefix `DOTNET_` and command-line arguments are included by `CreateDefaultBuilder`.</span></span> <span data-ttu-id="a2704-187">Per le app Web, viene aggiunto il provider di variabili di ambiente con prefisso `ASPNETCORE_`.</span><span class="sxs-lookup"><span data-stu-id="a2704-187">For web apps, the environment variable provider with prefix `ASPNETCORE_` is added.</span></span> <span data-ttu-id="a2704-188">Il prefisso viene rimosso quando vengono lette le variabili di ambiente.</span><span class="sxs-lookup"><span data-stu-id="a2704-188">The prefix is removed when the environment variables are read.</span></span> <span data-ttu-id="a2704-189">Ad esempio, il valore della variabile di ambiente per `ASPNETCORE_ENVIRONMENT` diventa il valore di configurazione dell'host per la chiave `environment`.</span><span class="sxs-lookup"><span data-stu-id="a2704-189">For example, the environment variable value for `ASPNETCORE_ENVIRONMENT` becomes the host configuration value for the `environment` key.</span></span>

<span data-ttu-id="a2704-190">L'esempio seguente crea la configurazione host:</span><span class="sxs-lookup"><span data-stu-id="a2704-190">The following example creates host configuration:</span></span>

[!code-csharp[](generic-host/samples-snapshot/3.x/Program.cs?name=snippet_HostConfig)]

## <a name="app-configuration"></a><span data-ttu-id="a2704-191">Configurazione delle app</span><span class="sxs-lookup"><span data-stu-id="a2704-191">App configuration</span></span>

<span data-ttu-id="a2704-192">La configurazione dell'app viene creata chiamando <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureAppConfiguration*> su `IHostBuilder`.</span><span class="sxs-lookup"><span data-stu-id="a2704-192">App configuration is created by calling <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureAppConfiguration*> on `IHostBuilder`.</span></span> <span data-ttu-id="a2704-193">`ConfigureAppConfiguration` può essere chiamato più volte e i risultati si sommano.</span><span class="sxs-lookup"><span data-stu-id="a2704-193">`ConfigureAppConfiguration` can be called multiple times with additive results.</span></span> <span data-ttu-id="a2704-194">L'app usa l'ultima opzione che imposta un valore in una determinata chiave.</span><span class="sxs-lookup"><span data-stu-id="a2704-194">The app uses whichever option sets a value last on a given key.</span></span> 

<span data-ttu-id="a2704-195">La configurazione creata da `ConfigureAppConfiguration` è disponibile in [HostBuilderContext.Configuration](xref:Microsoft.Extensions.Hosting.HostBuilderContext.Configuration*) per le operazioni successive e come servizio da DI.</span><span class="sxs-lookup"><span data-stu-id="a2704-195">The configuration created by `ConfigureAppConfiguration` is available at [HostBuilderContext.Configuration](xref:Microsoft.Extensions.Hosting.HostBuilderContext.Configuration*) for subsequent operations and as a service from DI.</span></span> <span data-ttu-id="a2704-196">La configurazione dell'host viene aggiunta anche alla configurazione dell'app.</span><span class="sxs-lookup"><span data-stu-id="a2704-196">The host configuration is also added to the app configuration.</span></span>

<span data-ttu-id="a2704-197">Per altre informazioni, vedere [Configurazione in ASP.NET Core](xref:fundamentals/configuration/index#configureappconfiguration).</span><span class="sxs-lookup"><span data-stu-id="a2704-197">For more information, see [Configuration in ASP.NET Core](xref:fundamentals/configuration/index#configureappconfiguration).</span></span>

## <a name="settings-for-all-app-types"></a><span data-ttu-id="a2704-198">Impostazioni per tutti i tipi di app</span><span class="sxs-lookup"><span data-stu-id="a2704-198">Settings for all app types</span></span>

<span data-ttu-id="a2704-199">Questa sezione elenca le impostazioni dell'host che si applicano ai carichi di lavoro HTTP e non-HTTP.</span><span class="sxs-lookup"><span data-stu-id="a2704-199">This section lists host settings that apply to both HTTP and non-HTTP workloads.</span></span> <span data-ttu-id="a2704-200">Per impostazione predefinita, le variabili di ambiente usate per configurare queste impostazioni possono avere un prefisso `DOTNET_` o `ASPNETCORE_`.</span><span class="sxs-lookup"><span data-stu-id="a2704-200">By default, environment variables used to configure these settings can have a `DOTNET_` or `ASPNETCORE_` prefix.</span></span> <span data-ttu-id="a2704-201">Per ulteriori informazioni, vedere la sezione [impostazioni predefinite del generatore](#default-builder-settings) .</span><span class="sxs-lookup"><span data-stu-id="a2704-201">For more information, see the [Default builder settings](#default-builder-settings) section.</span></span>

<!-- In the following sections, two spaces at end of line are used to force line breaks in the rendered page. -->

### <a name="applicationname"></a><span data-ttu-id="a2704-202">ApplicationName</span><span class="sxs-lookup"><span data-stu-id="a2704-202">ApplicationName</span></span>

<span data-ttu-id="a2704-203">La proprietà [IHostEnvironment.ApplicationName](xref:Microsoft.Extensions.Hosting.IHostEnvironment.ApplicationName*) viene impostata dalla configurazione dell'host durante la costruzione dell'host.</span><span class="sxs-lookup"><span data-stu-id="a2704-203">The [IHostEnvironment.ApplicationName](xref:Microsoft.Extensions.Hosting.IHostEnvironment.ApplicationName*) property is set from host configuration during host construction.</span></span>

<span data-ttu-id="a2704-204">**Chiave**: `applicationName`</span><span class="sxs-lookup"><span data-stu-id="a2704-204">**Key**: `applicationName`</span></span>  
<span data-ttu-id="a2704-205">**Tipo**: `string`</span><span class="sxs-lookup"><span data-stu-id="a2704-205">**Type**: `string`</span></span>  
<span data-ttu-id="a2704-206">**Impostazione predefinita**: il nome dell'assembly che contiene il punto di ingresso dell'app.</span><span class="sxs-lookup"><span data-stu-id="a2704-206">**Default**: The name of the assembly that contains the app's entry point.</span></span>  
<span data-ttu-id="a2704-207">**Variabile di ambiente**: `<PREFIX_>APPLICATIONNAME`</span><span class="sxs-lookup"><span data-stu-id="a2704-207">**Environment variable**: `<PREFIX_>APPLICATIONNAME`</span></span>

<span data-ttu-id="a2704-208">per impostare questo valore usare la variabile di ambiente.</span><span class="sxs-lookup"><span data-stu-id="a2704-208">To set this value, use the environment variable.</span></span> 

### <a name="contentroot"></a><span data-ttu-id="a2704-209">ContentRoot</span><span class="sxs-lookup"><span data-stu-id="a2704-209">ContentRoot</span></span>

<span data-ttu-id="a2704-210">La proprietà [IHostEnvironment.ContentRootPath](xref:Microsoft.Extensions.Hosting.IHostEnvironment.ContentRootPath*) determina il punto in cui l'host inizia a cercare i file di contenuto.</span><span class="sxs-lookup"><span data-stu-id="a2704-210">The [IHostEnvironment.ContentRootPath](xref:Microsoft.Extensions.Hosting.IHostEnvironment.ContentRootPath*) property determines where the host begins searching for content files.</span></span> <span data-ttu-id="a2704-211">Se il percorso non esiste, l'host non verrà avviato.</span><span class="sxs-lookup"><span data-stu-id="a2704-211">If the path doesn't exist, the host fails to start.</span></span>

<span data-ttu-id="a2704-212">**Chiave**: `contentRoot`</span><span class="sxs-lookup"><span data-stu-id="a2704-212">**Key**: `contentRoot`</span></span>  
<span data-ttu-id="a2704-213">**Tipo**: `string`</span><span class="sxs-lookup"><span data-stu-id="a2704-213">**Type**: `string`</span></span>  
<span data-ttu-id="a2704-214">**Impostazione predefinita**: la cartella in cui risiede l'assembly dell'app.</span><span class="sxs-lookup"><span data-stu-id="a2704-214">**Default**: The folder where the app assembly resides.</span></span>  
<span data-ttu-id="a2704-215">**Variabile di ambiente**: `<PREFIX_>CONTENTROOT`</span><span class="sxs-lookup"><span data-stu-id="a2704-215">**Environment variable**: `<PREFIX_>CONTENTROOT`</span></span>

<span data-ttu-id="a2704-216">Per impostare questo valore, usare la variabile di ambiente o chiamare `UseContentRoot` su `IHostBuilder`:</span><span class="sxs-lookup"><span data-stu-id="a2704-216">To set this value, use the environment variable or call `UseContentRoot` on `IHostBuilder`:</span></span>

```csharp
Host.CreateDefaultBuilder(args)
    .UseContentRoot("c:\\content-root")
    //...
```

<span data-ttu-id="a2704-217">Per altre informazioni, vedere:</span><span class="sxs-lookup"><span data-stu-id="a2704-217">For more information, see:</span></span>

* [<span data-ttu-id="a2704-218">Nozioni fondamentali: radice del contenuto</span><span class="sxs-lookup"><span data-stu-id="a2704-218">Fundamentals: Content root</span></span>](xref:fundamentals/index#content-root)
* [<span data-ttu-id="a2704-219">WebRoot</span><span class="sxs-lookup"><span data-stu-id="a2704-219">WebRoot</span></span>](#webroot)

### <a name="environmentname"></a><span data-ttu-id="a2704-220">EnvironmentName</span><span class="sxs-lookup"><span data-stu-id="a2704-220">EnvironmentName</span></span>

<span data-ttu-id="a2704-221">La proprietà [IHostEnvironment.EnvironmentName](xref:Microsoft.Extensions.Hosting.IHostEnvironment.EnvironmentName*) può essere impostata su qualsiasi valore.</span><span class="sxs-lookup"><span data-stu-id="a2704-221">The [IHostEnvironment.EnvironmentName](xref:Microsoft.Extensions.Hosting.IHostEnvironment.EnvironmentName*) property can be set to any value.</span></span> <span data-ttu-id="a2704-222">I valori definiti dal framework includono `Development`, `Staging` e `Production`.</span><span class="sxs-lookup"><span data-stu-id="a2704-222">Framework-defined values include `Development`, `Staging`, and `Production`.</span></span> <span data-ttu-id="a2704-223">I valori non fanno distinzione tra maiuscole e minuscole.</span><span class="sxs-lookup"><span data-stu-id="a2704-223">Values aren't case-sensitive.</span></span>

<span data-ttu-id="a2704-224">**Chiave**: `environment`</span><span class="sxs-lookup"><span data-stu-id="a2704-224">**Key**: `environment`</span></span>  
<span data-ttu-id="a2704-225">**Tipo**: `string`</span><span class="sxs-lookup"><span data-stu-id="a2704-225">**Type**: `string`</span></span>  
<span data-ttu-id="a2704-226">**Impostazione predefinita**: `Production`</span><span class="sxs-lookup"><span data-stu-id="a2704-226">**Default**: `Production`</span></span>  
<span data-ttu-id="a2704-227">**Variabile di ambiente**: `<PREFIX_>ENVIRONMENT`</span><span class="sxs-lookup"><span data-stu-id="a2704-227">**Environment variable**: `<PREFIX_>ENVIRONMENT`</span></span>

<span data-ttu-id="a2704-228">Per impostare questo valore, usare la variabile di ambiente o chiamare `UseEnvironment` su `IHostBuilder`:</span><span class="sxs-lookup"><span data-stu-id="a2704-228">To set this value, use the environment variable or call `UseEnvironment` on `IHostBuilder`:</span></span>

```csharp
Host.CreateDefaultBuilder(args)
    .UseEnvironment("Development")
    //...
```

### <a name="shutdowntimeout"></a><span data-ttu-id="a2704-229">ShutdownTimeout</span><span class="sxs-lookup"><span data-stu-id="a2704-229">ShutdownTimeout</span></span>

<span data-ttu-id="a2704-230">[HostOptions.ShutdownTimeout](xref:Microsoft.Extensions.Hosting.HostOptions.ShutdownTimeout*) imposta il timeout per <xref:Microsoft.Extensions.Hosting.IHost.StopAsync*>.</span><span class="sxs-lookup"><span data-stu-id="a2704-230">[HostOptions.ShutdownTimeout](xref:Microsoft.Extensions.Hosting.HostOptions.ShutdownTimeout*) sets the timeout for <xref:Microsoft.Extensions.Hosting.IHost.StopAsync*>.</span></span> <span data-ttu-id="a2704-231">Il valore predefinito è cinque secondi.</span><span class="sxs-lookup"><span data-stu-id="a2704-231">The default value is five seconds.</span></span>  <span data-ttu-id="a2704-232">Durante il periodo di timeout, l'host:</span><span class="sxs-lookup"><span data-stu-id="a2704-232">During the timeout period, the host:</span></span>

* <span data-ttu-id="a2704-233">attiva [IHostApplicationLifetime.ApplicationStopping](/dotnet/api/microsoft.extensions.hosting.ihostapplicationlifetime.applicationstopping).</span><span class="sxs-lookup"><span data-stu-id="a2704-233">Triggers [IHostApplicationLifetime.ApplicationStopping](/dotnet/api/microsoft.extensions.hosting.ihostapplicationlifetime.applicationstopping).</span></span>
* <span data-ttu-id="a2704-234">Tenta di arrestare i servizi ospitati, registrando gli errori per i servizi che non si interrompono.</span><span class="sxs-lookup"><span data-stu-id="a2704-234">Attempts to stop hosted services, logging errors for services that fail to stop.</span></span>

<span data-ttu-id="a2704-235">Se il periodo di timeout scade prima che siano stati arrestati tutti i servizi ospitati, gli eventuali servizi attivi rimanenti vengono interrotti quando l'app viene arrestata.</span><span class="sxs-lookup"><span data-stu-id="a2704-235">If the timeout period expires before all of the hosted services stop, any remaining active services are stopped when the app shuts down.</span></span> <span data-ttu-id="a2704-236">I servizi si arrestano anche se non hanno completato l'elaborazione.</span><span class="sxs-lookup"><span data-stu-id="a2704-236">The services stop even if they haven't finished processing.</span></span> <span data-ttu-id="a2704-237">Se l'arresto dei servizi richiede più tempo, aumentare il valore di timeout.</span><span class="sxs-lookup"><span data-stu-id="a2704-237">If services require additional time to stop, increase the timeout.</span></span>

<span data-ttu-id="a2704-238">**Chiave**: `shutdownTimeoutSeconds`</span><span class="sxs-lookup"><span data-stu-id="a2704-238">**Key**: `shutdownTimeoutSeconds`</span></span>  
<span data-ttu-id="a2704-239">**Tipo**: `int`</span><span class="sxs-lookup"><span data-stu-id="a2704-239">**Type**: `int`</span></span>  
<span data-ttu-id="a2704-240">**Impostazione predefinita**: 5 secondi</span><span class="sxs-lookup"><span data-stu-id="a2704-240">**Default**: 5 seconds</span></span>  
<span data-ttu-id="a2704-241">**Variabile di ambiente**: `<PREFIX_>SHUTDOWNTIMEOUTSECONDS`</span><span class="sxs-lookup"><span data-stu-id="a2704-241">**Environment variable**: `<PREFIX_>SHUTDOWNTIMEOUTSECONDS`</span></span>

<span data-ttu-id="a2704-242">Per impostare questo valore, usare la variabile di ambiente o configurare `HostOptions`.</span><span class="sxs-lookup"><span data-stu-id="a2704-242">To set this value, use the environment variable or configure `HostOptions`.</span></span> <span data-ttu-id="a2704-243">Nell'esempio seguente il timeout viene impostato su 20 secondi:</span><span class="sxs-lookup"><span data-stu-id="a2704-243">The following example sets the timeout to 20 seconds:</span></span>

[!code-csharp[](generic-host/samples-snapshot/3.x/Program.cs?name=snippet_HostOptions)]

### <a name="disable-app-configuration-reload-on-change"></a><span data-ttu-id="a2704-244">Disabilitare il ricaricamento della configurazione dell'app durante la modifica</span><span class="sxs-lookup"><span data-stu-id="a2704-244">Disable app configuration reload on change</span></span>

<span data-ttu-id="a2704-245">Per [impostazione predefinita](xref:fundamentals/configuration/index#default), *appsettings.json* e *appSettings. { L'ambiente}. JSON* viene ricaricato quando il file viene modificato.</span><span class="sxs-lookup"><span data-stu-id="a2704-245">By [default](xref:fundamentals/configuration/index#default), *appsettings.json* and *appsettings.{Environment}.json* are reloaded when the file changes.</span></span> <span data-ttu-id="a2704-246">Per disabilitare questo comportamento di ricaricamento in ASP.NET Core 5,0 o versione successiva, impostare la `hostBuilder:reloadConfigOnChange` chiave su `false` .</span><span class="sxs-lookup"><span data-stu-id="a2704-246">To disable this reload behavior in ASP.NET Core 5.0 or later, set the `hostBuilder:reloadConfigOnChange` key to `false`.</span></span>

<span data-ttu-id="a2704-247">**Chiave**: `hostBuilder:reloadConfigOnChange`</span><span class="sxs-lookup"><span data-stu-id="a2704-247">**Key**: `hostBuilder:reloadConfigOnChange`</span></span>  
<span data-ttu-id="a2704-248">**Tipo**: `bool` ( `true` o `1` )</span><span class="sxs-lookup"><span data-stu-id="a2704-248">**Type**: `bool` (`true` or `1`)</span></span>  
<span data-ttu-id="a2704-249">**Impostazione predefinita**: `true`</span><span class="sxs-lookup"><span data-stu-id="a2704-249">**Default**: `true`</span></span>  
<span data-ttu-id="a2704-250">**Argomento della riga di comando**: `hostBuilder:reloadConfigOnChange`</span><span class="sxs-lookup"><span data-stu-id="a2704-250">**Command-line argument**: `hostBuilder:reloadConfigOnChange`</span></span>  
<span data-ttu-id="a2704-251">**Variabile di ambiente**: `<PREFIX_>hostBuilder:reloadConfigOnChange`</span><span class="sxs-lookup"><span data-stu-id="a2704-251">**Environment variable**: `<PREFIX_>hostBuilder:reloadConfigOnChange`</span></span>

> [!WARNING]
> <span data-ttu-id="a2704-252">Il separatore dei due punti ( `:` ) non funziona con le chiavi gerarchiche delle variabili di ambiente in tutte le piattaforme.</span><span class="sxs-lookup"><span data-stu-id="a2704-252">The colon (`:`) separator doesn't work with environment variable hierarchical keys on all platforms.</span></span> <span data-ttu-id="a2704-253">Per altre informazioni, vedere [variabili di ambiente](xref:fundamentals/configuration/index#environment-variables).</span><span class="sxs-lookup"><span data-stu-id="a2704-253">For more information, see [Environment variables](xref:fundamentals/configuration/index#environment-variables).</span></span>

## <a name="settings-for-web-apps"></a><span data-ttu-id="a2704-254">Impostazioni per le app Web</span><span class="sxs-lookup"><span data-stu-id="a2704-254">Settings for web apps</span></span>

<span data-ttu-id="a2704-255">Alcune impostazioni host si applicano solo ai carichi di lavoro HTTP.</span><span class="sxs-lookup"><span data-stu-id="a2704-255">Some host settings apply only to HTTP workloads.</span></span> <span data-ttu-id="a2704-256">Per impostazione predefinita, le variabili di ambiente usate per configurare queste impostazioni possono avere un prefisso `DOTNET_` o `ASPNETCORE_`.</span><span class="sxs-lookup"><span data-stu-id="a2704-256">By default, environment variables used to configure these settings can have a `DOTNET_` or `ASPNETCORE_` prefix.</span></span>

<span data-ttu-id="a2704-257">Per queste impostazioni sono disponibili i metodi di estensione su `IWebHostBuilder`.</span><span class="sxs-lookup"><span data-stu-id="a2704-257">Extension methods on `IWebHostBuilder` are available for these settings.</span></span> <span data-ttu-id="a2704-258">Gli esempi di codice che illustrano come chiamare i metodi di estensione presumono che `webBuilder` sia un'istanza di `IWebHostBuilder`, come nell'esempio seguente:</span><span class="sxs-lookup"><span data-stu-id="a2704-258">Code samples that show how to call the extension methods assume `webBuilder` is an instance of `IWebHostBuilder`, as in the following example:</span></span>

```csharp
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.CaptureStartupErrors(true);
            webBuilder.UseStartup<Startup>();
        });
```

### <a name="capturestartuperrors"></a><span data-ttu-id="a2704-259">CaptureStartupErrors</span><span class="sxs-lookup"><span data-stu-id="a2704-259">CaptureStartupErrors</span></span>

<span data-ttu-id="a2704-260">Quando è `false`, gli errori durante l'avvio causano la chiusura dell'host.</span><span class="sxs-lookup"><span data-stu-id="a2704-260">When `false`, errors during startup result in the host exiting.</span></span> <span data-ttu-id="a2704-261">Quando è `true`, l'host acquisisce le eccezioni durante l'avvio e tenta di avviare il server.</span><span class="sxs-lookup"><span data-stu-id="a2704-261">When `true`, the host captures exceptions during startup and attempts to start the server.</span></span>

<span data-ttu-id="a2704-262">**Chiave**: `captureStartupErrors`</span><span class="sxs-lookup"><span data-stu-id="a2704-262">**Key**: `captureStartupErrors`</span></span>  
<span data-ttu-id="a2704-263">**Tipo**: `bool` ( `true` o `1` )</span><span class="sxs-lookup"><span data-stu-id="a2704-263">**Type**: `bool` (`true` or `1`)</span></span>  
<span data-ttu-id="a2704-264">**Impostazione predefinita**: il valore predefinito è `false` a meno che l'app non venga eseguita con Kestrel in IIS, in tal caso il valore predefinito è `true`.</span><span class="sxs-lookup"><span data-stu-id="a2704-264">**Default**: Defaults to `false` unless the app runs with Kestrel behind IIS, where the default is `true`.</span></span>  
<span data-ttu-id="a2704-265">**Variabile di ambiente**: `<PREFIX_>CAPTURESTARTUPERRORS`</span><span class="sxs-lookup"><span data-stu-id="a2704-265">**Environment variable**: `<PREFIX_>CAPTURESTARTUPERRORS`</span></span>

<span data-ttu-id="a2704-266">Per impostare questo valore, usare la configurazione o chiamare `CaptureStartupErrors`:</span><span class="sxs-lookup"><span data-stu-id="a2704-266">To set this value, use configuration or call `CaptureStartupErrors`:</span></span>

```csharp
webBuilder.CaptureStartupErrors(true);
```

### <a name="detailederrors"></a><span data-ttu-id="a2704-267">DetailedErrors</span><span class="sxs-lookup"><span data-stu-id="a2704-267">DetailedErrors</span></span>

<span data-ttu-id="a2704-268">Quando è abilitata o quando l'ambiente è `Development`, l'app acquisisce gli errori dettagliati.</span><span class="sxs-lookup"><span data-stu-id="a2704-268">When enabled, or when the environment is `Development`, the app captures detailed errors.</span></span>

<span data-ttu-id="a2704-269">**Chiave**: `detailedErrors`</span><span class="sxs-lookup"><span data-stu-id="a2704-269">**Key**: `detailedErrors`</span></span>  
<span data-ttu-id="a2704-270">**Tipo**: `bool` ( `true` o `1` )</span><span class="sxs-lookup"><span data-stu-id="a2704-270">**Type**: `bool` (`true` or `1`)</span></span>  
<span data-ttu-id="a2704-271">**Impostazione predefinita**: `false`</span><span class="sxs-lookup"><span data-stu-id="a2704-271">**Default**: `false`</span></span>  
<span data-ttu-id="a2704-272">**Variabile di ambiente**: `<PREFIX_>_DETAILEDERRORS`</span><span class="sxs-lookup"><span data-stu-id="a2704-272">**Environment variable**: `<PREFIX_>_DETAILEDERRORS`</span></span>

<span data-ttu-id="a2704-273">Per impostare questo valore, usare la configurazione o chiamare `UseSetting`:</span><span class="sxs-lookup"><span data-stu-id="a2704-273">To set this value, use configuration or call `UseSetting`:</span></span>

```csharp
webBuilder.UseSetting(WebHostDefaults.DetailedErrorsKey, "true");
```

### <a name="hostingstartupassemblies"></a><span data-ttu-id="a2704-274">HostingStartupAssemblies</span><span class="sxs-lookup"><span data-stu-id="a2704-274">HostingStartupAssemblies</span></span>

<span data-ttu-id="a2704-275">Una stringa delimitata da punto e virgola di assembly di avvio dell'hosting da caricare all'avvio.</span><span class="sxs-lookup"><span data-stu-id="a2704-275">A semicolon-delimited string of hosting startup assemblies to load on startup.</span></span> <span data-ttu-id="a2704-276">Sebbene il valore di configurazione predefinito sia una stringa vuota, gli assembly di avvio dell'hosting includono sempre l'assembly dell'app.</span><span class="sxs-lookup"><span data-stu-id="a2704-276">Although the configuration value defaults to an empty string, the hosting startup assemblies always include the app's assembly.</span></span> <span data-ttu-id="a2704-277">Quando vengono specificati, gli assembly di avvio dell'hosting vengono aggiunti all'assembly dell'app per essere caricati quando l'app compila i servizi comuni durante l'avvio.</span><span class="sxs-lookup"><span data-stu-id="a2704-277">When hosting startup assemblies are provided, they're added to the app's assembly for loading when the app builds its common services during startup.</span></span>

<span data-ttu-id="a2704-278">**Chiave**: `hostingStartupAssemblies`</span><span class="sxs-lookup"><span data-stu-id="a2704-278">**Key**: `hostingStartupAssemblies`</span></span>  
<span data-ttu-id="a2704-279">**Tipo**: `string`</span><span class="sxs-lookup"><span data-stu-id="a2704-279">**Type**: `string`</span></span>  
<span data-ttu-id="a2704-280">**Impostazione predefinita**: stringa vuota</span><span class="sxs-lookup"><span data-stu-id="a2704-280">**Default**: Empty string</span></span>  
<span data-ttu-id="a2704-281">**Variabile di ambiente**: `<PREFIX_>_HOSTINGSTARTUPASSEMBLIES`</span><span class="sxs-lookup"><span data-stu-id="a2704-281">**Environment variable**: `<PREFIX_>_HOSTINGSTARTUPASSEMBLIES`</span></span>

<span data-ttu-id="a2704-282">Per impostare questo valore, usare la configurazione o chiamare `UseSetting`:</span><span class="sxs-lookup"><span data-stu-id="a2704-282">To set this value, use configuration or call `UseSetting`:</span></span>

```csharp
webBuilder.UseSetting(WebHostDefaults.HostingStartupAssembliesKey, "assembly1;assembly2");
```

### <a name="hostingstartupexcludeassemblies"></a><span data-ttu-id="a2704-283">HostingStartupExcludeAssemblies</span><span class="sxs-lookup"><span data-stu-id="a2704-283">HostingStartupExcludeAssemblies</span></span>

<span data-ttu-id="a2704-284">Una stringa delimitata da punto e virgola di assembly di avvio dell'hosting da escludere all'avvio.</span><span class="sxs-lookup"><span data-stu-id="a2704-284">A semicolon-delimited string of hosting startup assemblies to exclude on startup.</span></span>

<span data-ttu-id="a2704-285">**Chiave**: `hostingStartupExcludeAssemblies`</span><span class="sxs-lookup"><span data-stu-id="a2704-285">**Key**: `hostingStartupExcludeAssemblies`</span></span>  
<span data-ttu-id="a2704-286">**Tipo**: `string`</span><span class="sxs-lookup"><span data-stu-id="a2704-286">**Type**: `string`</span></span>  
<span data-ttu-id="a2704-287">**Impostazione predefinita**: stringa vuota</span><span class="sxs-lookup"><span data-stu-id="a2704-287">**Default**: Empty string</span></span>  
<span data-ttu-id="a2704-288">**Variabile di ambiente**: `<PREFIX_>_HOSTINGSTARTUPEXCLUDEASSEMBLIES`</span><span class="sxs-lookup"><span data-stu-id="a2704-288">**Environment variable**: `<PREFIX_>_HOSTINGSTARTUPEXCLUDEASSEMBLIES`</span></span>

<span data-ttu-id="a2704-289">Per impostare questo valore, usare la configurazione o chiamare `UseSetting`:</span><span class="sxs-lookup"><span data-stu-id="a2704-289">To set this value, use configuration or call `UseSetting`:</span></span>

```csharp
webBuilder.UseSetting(WebHostDefaults.HostingStartupExcludeAssembliesKey, "assembly1;assembly2");
```

### <a name="https_port"></a><span data-ttu-id="a2704-290">HTTPS_Port</span><span class="sxs-lookup"><span data-stu-id="a2704-290">HTTPS_Port</span></span>

<span data-ttu-id="a2704-291">Porta di reindirizzamento HTTPS.</span><span class="sxs-lookup"><span data-stu-id="a2704-291">The HTTPS redirect port.</span></span> <span data-ttu-id="a2704-292">Usata per [imporre HTTPS](xref:security/enforcing-ssl).</span><span class="sxs-lookup"><span data-stu-id="a2704-292">Used in [enforcing HTTPS](xref:security/enforcing-ssl).</span></span>

<span data-ttu-id="a2704-293">**Chiave**: `https_port`</span><span class="sxs-lookup"><span data-stu-id="a2704-293">**Key**: `https_port`</span></span>  
<span data-ttu-id="a2704-294">**Tipo**: `string`</span><span class="sxs-lookup"><span data-stu-id="a2704-294">**Type**: `string`</span></span>  
<span data-ttu-id="a2704-295">**Impostazione predefinita**: un valore predefinito non è impostato.</span><span class="sxs-lookup"><span data-stu-id="a2704-295">**Default**: A default value isn't set.</span></span>  
<span data-ttu-id="a2704-296">**Variabile di ambiente**: `<PREFIX_>HTTPS_PORT`</span><span class="sxs-lookup"><span data-stu-id="a2704-296">**Environment variable**: `<PREFIX_>HTTPS_PORT`</span></span>

<span data-ttu-id="a2704-297">Per impostare questo valore, usare la configurazione o chiamare `UseSetting`:</span><span class="sxs-lookup"><span data-stu-id="a2704-297">To set this value, use configuration or call `UseSetting`:</span></span>

```csharp
webBuilder.UseSetting("https_port", "8080");
```

### <a name="preferhostingurls"></a><span data-ttu-id="a2704-298">PreferHostingUrls</span><span class="sxs-lookup"><span data-stu-id="a2704-298">PreferHostingUrls</span></span>

<span data-ttu-id="a2704-299">Indica se l'host deve essere in ascolto sugli URL configurati con `IWebHostBuilder` anziché gli URL configurati con l' `IServer` implementazione di.</span><span class="sxs-lookup"><span data-stu-id="a2704-299">Indicates whether the host should listen on the URLs configured with the `IWebHostBuilder` instead of those URLs configured with the `IServer` implementation.</span></span>

<span data-ttu-id="a2704-300">**Chiave**: `preferHostingUrls`</span><span class="sxs-lookup"><span data-stu-id="a2704-300">**Key**: `preferHostingUrls`</span></span>  
<span data-ttu-id="a2704-301">**Tipo**: `bool` ( `true` o `1` )</span><span class="sxs-lookup"><span data-stu-id="a2704-301">**Type**: `bool` (`true` or `1`)</span></span>  
<span data-ttu-id="a2704-302">**Impostazione predefinita**: `true`</span><span class="sxs-lookup"><span data-stu-id="a2704-302">**Default**: `true`</span></span>  
<span data-ttu-id="a2704-303">**Variabile di ambiente**: `<PREFIX_>_PREFERHOSTINGURLS`</span><span class="sxs-lookup"><span data-stu-id="a2704-303">**Environment variable**: `<PREFIX_>_PREFERHOSTINGURLS`</span></span>

<span data-ttu-id="a2704-304">Per impostare questo valore, usare la variabile di ambiente o chiamare `PreferHostingUrls`:</span><span class="sxs-lookup"><span data-stu-id="a2704-304">To set this value, use the environment variable or call `PreferHostingUrls`:</span></span>

```csharp
webBuilder.PreferHostingUrls(false);
```

### <a name="preventhostingstartup"></a><span data-ttu-id="a2704-305">PreventHostingStartup</span><span class="sxs-lookup"><span data-stu-id="a2704-305">PreventHostingStartup</span></span>

<span data-ttu-id="a2704-306">Impedisce il caricamento automatico degli assembly di avvio dell'hosting, inclusi gli assembly di avvio dell'hosting configurati dall'assembly dell'app.</span><span class="sxs-lookup"><span data-stu-id="a2704-306">Prevents the automatic loading of hosting startup assemblies, including hosting startup assemblies configured by the app's assembly.</span></span> <span data-ttu-id="a2704-307">Per altre informazioni, vedere <xref:fundamentals/configuration/platform-specific-configuration>.</span><span class="sxs-lookup"><span data-stu-id="a2704-307">For more information, see <xref:fundamentals/configuration/platform-specific-configuration>.</span></span>

<span data-ttu-id="a2704-308">**Chiave**: `preventHostingStartup`</span><span class="sxs-lookup"><span data-stu-id="a2704-308">**Key**: `preventHostingStartup`</span></span>  
<span data-ttu-id="a2704-309">**Tipo**: `bool` ( `true` o `1` )</span><span class="sxs-lookup"><span data-stu-id="a2704-309">**Type**: `bool` (`true` or `1`)</span></span>  
<span data-ttu-id="a2704-310">**Impostazione predefinita**: `false`</span><span class="sxs-lookup"><span data-stu-id="a2704-310">**Default**: `false`</span></span>  
<span data-ttu-id="a2704-311">**Variabile di ambiente**: `<PREFIX_>_PREVENTHOSTINGSTARTUP`</span><span class="sxs-lookup"><span data-stu-id="a2704-311">**Environment variable**: `<PREFIX_>_PREVENTHOSTINGSTARTUP`</span></span>

<span data-ttu-id="a2704-312">Per impostare questo valore, usare la variabile di ambiente o chiamare `UseSetting`:</span><span class="sxs-lookup"><span data-stu-id="a2704-312">To set this value, use the environment variable or call `UseSetting` :</span></span>

```csharp
webBuilder.UseSetting(WebHostDefaults.PreventHostingStartupKey, "true");
```

### <a name="startupassembly"></a><span data-ttu-id="a2704-313">StartupAssembly</span><span class="sxs-lookup"><span data-stu-id="a2704-313">StartupAssembly</span></span>

<span data-ttu-id="a2704-314">L'assembly per la ricerca della classe `Startup`.</span><span class="sxs-lookup"><span data-stu-id="a2704-314">The assembly to search for the `Startup` class.</span></span>

<span data-ttu-id="a2704-315">**Chiave**: `startupAssembly`</span><span class="sxs-lookup"><span data-stu-id="a2704-315">**Key**: `startupAssembly`</span></span>  
<span data-ttu-id="a2704-316">**Tipo**: `string`</span><span class="sxs-lookup"><span data-stu-id="a2704-316">**Type**: `string`</span></span>  
<span data-ttu-id="a2704-317">**Impostazione predefinita**: assembly dell'app</span><span class="sxs-lookup"><span data-stu-id="a2704-317">**Default**: The app's assembly</span></span>  
<span data-ttu-id="a2704-318">**Variabile di ambiente**: `<PREFIX_>STARTUPASSEMBLY`</span><span class="sxs-lookup"><span data-stu-id="a2704-318">**Environment variable**: `<PREFIX_>STARTUPASSEMBLY`</span></span>

<span data-ttu-id="a2704-319">Per impostare questo valore, usare la variabile di ambiente o chiamare `UseStartup`.</span><span class="sxs-lookup"><span data-stu-id="a2704-319">To set this value, use the environment variable or call `UseStartup`.</span></span> <span data-ttu-id="a2704-320">`UseStartup` può richiedere un nome di assembly (`string`) o un tipo (`TStartup`).</span><span class="sxs-lookup"><span data-stu-id="a2704-320">`UseStartup` can take an assembly name (`string`) or a type (`TStartup`).</span></span> <span data-ttu-id="a2704-321">Se vengono chiamati più metodi `UseStartup`, l'ultimo metodo ha la precedenza.</span><span class="sxs-lookup"><span data-stu-id="a2704-321">If multiple `UseStartup` methods are called, the last one takes precedence.</span></span>

```csharp
webBuilder.UseStartup("StartupAssemblyName");
```

```csharp
webBuilder.UseStartup<Startup>();
```

### <a name="urls"></a><span data-ttu-id="a2704-322">URL</span><span class="sxs-lookup"><span data-stu-id="a2704-322">URLs</span></span>

<span data-ttu-id="a2704-323">Elenco delimitato da punto e virgola degli indirizzi IP o gli indirizzi host con le porte e protocolli su cui il server deve eseguire l'ascolto per le richieste.</span><span class="sxs-lookup"><span data-stu-id="a2704-323">A semicolon-delimited list of IP addresses or host addresses with ports and protocols that the server should listen on for requests.</span></span> <span data-ttu-id="a2704-324">Ad esempio: `http://localhost:123`.</span><span class="sxs-lookup"><span data-stu-id="a2704-324">For example, `http://localhost:123`.</span></span> <span data-ttu-id="a2704-325">Usare "\*" per indicare che il server deve eseguire l'ascolto per le richieste su tutti gli indirizzi IP o nomi host usando la porta e il protocollo specificati (ad esempio, `http://*:5000`).</span><span class="sxs-lookup"><span data-stu-id="a2704-325">Use "\*" to indicate that the server should listen for requests on any IP address or hostname using the specified port and protocol (for example, `http://*:5000`).</span></span> <span data-ttu-id="a2704-326">Il protocollo (`http://` o `https://`) deve essere incluso con ogni URL.</span><span class="sxs-lookup"><span data-stu-id="a2704-326">The protocol (`http://` or `https://`) must be included with each URL.</span></span> <span data-ttu-id="a2704-327">I formati supportati variano a seconda del server.</span><span class="sxs-lookup"><span data-stu-id="a2704-327">Supported formats vary among servers.</span></span>

<span data-ttu-id="a2704-328">**Chiave**: `urls`</span><span class="sxs-lookup"><span data-stu-id="a2704-328">**Key**: `urls`</span></span>  
<span data-ttu-id="a2704-329">**Tipo**: `string`</span><span class="sxs-lookup"><span data-stu-id="a2704-329">**Type**: `string`</span></span>  
<span data-ttu-id="a2704-330">**Impostazione predefinita**: `http://localhost:5000` e `https://localhost:5001`</span><span class="sxs-lookup"><span data-stu-id="a2704-330">**Default**: `http://localhost:5000` and `https://localhost:5001`</span></span>  
<span data-ttu-id="a2704-331">**Variabile di ambiente**: `<PREFIX_>URLS`</span><span class="sxs-lookup"><span data-stu-id="a2704-331">**Environment variable**: `<PREFIX_>URLS`</span></span>

<span data-ttu-id="a2704-332">Per impostare questo valore, usare la variabile di ambiente o chiamare `UseUrls`:</span><span class="sxs-lookup"><span data-stu-id="a2704-332">To set this value, use the environment variable or call `UseUrls`:</span></span>

```csharp
webBuilder.UseUrls("http://*:5000;http://localhost:5001;https://hostname:5002");
```

<span data-ttu-id="a2704-333">Kestrel ha una propria API di configurazione degli endpoint.</span><span class="sxs-lookup"><span data-stu-id="a2704-333">Kestrel has its own endpoint configuration API.</span></span> <span data-ttu-id="a2704-334">Per altre informazioni, vedere <xref:fundamentals/servers/kestrel/endpoints>.</span><span class="sxs-lookup"><span data-stu-id="a2704-334">For more information, see <xref:fundamentals/servers/kestrel/endpoints>.</span></span>

### <a name="webroot"></a><span data-ttu-id="a2704-335">WebRoot</span><span class="sxs-lookup"><span data-stu-id="a2704-335">WebRoot</span></span>

<span data-ttu-id="a2704-336">La proprietà [IWebHostEnvironment. WebRootPath](xref:Microsoft.AspNetCore.Hosting.IWebHostEnvironment.WebRootPath) determina il percorso relativo degli asset statici dell'app.</span><span class="sxs-lookup"><span data-stu-id="a2704-336">The [IWebHostEnvironment.WebRootPath](xref:Microsoft.AspNetCore.Hosting.IWebHostEnvironment.WebRootPath) property determines the relative path to the app's static assets.</span></span> <span data-ttu-id="a2704-337">Se il percorso non esiste, viene usato un provider di file no-op.</span><span class="sxs-lookup"><span data-stu-id="a2704-337">If the path doesn't exist, a no-op file provider is used.</span></span>  

<span data-ttu-id="a2704-338">**Chiave**: `webroot`</span><span class="sxs-lookup"><span data-stu-id="a2704-338">**Key**: `webroot`</span></span>  
<span data-ttu-id="a2704-339">**Tipo**: `string`</span><span class="sxs-lookup"><span data-stu-id="a2704-339">**Type**: `string`</span></span>  
<span data-ttu-id="a2704-340">**Impostazione predefinita**: il valore predefinito è `wwwroot` .</span><span class="sxs-lookup"><span data-stu-id="a2704-340">**Default**: The default is `wwwroot`.</span></span> <span data-ttu-id="a2704-341">Il percorso di *{radice del contenuto}/wwwroot* deve esistere.</span><span class="sxs-lookup"><span data-stu-id="a2704-341">The path to *{content root}/wwwroot* must exist.</span></span>  
<span data-ttu-id="a2704-342">**Variabile di ambiente**: `<PREFIX_>WEBROOT`</span><span class="sxs-lookup"><span data-stu-id="a2704-342">**Environment variable**: `<PREFIX_>WEBROOT`</span></span>

<span data-ttu-id="a2704-343">Per impostare questo valore, usare la variabile di ambiente o chiamare `UseWebRoot` su `IWebHostBuilder`:</span><span class="sxs-lookup"><span data-stu-id="a2704-343">To set this value, use the environment variable or call `UseWebRoot` on `IWebHostBuilder`:</span></span>

```csharp
webBuilder.UseWebRoot("public");
```

<span data-ttu-id="a2704-344">Per altre informazioni, vedere:</span><span class="sxs-lookup"><span data-stu-id="a2704-344">For more information, see:</span></span>

* [<span data-ttu-id="a2704-345">Nozioni fondamentali: radice Web</span><span class="sxs-lookup"><span data-stu-id="a2704-345">Fundamentals: Web root</span></span>](xref:fundamentals/index#web-root)
* [<span data-ttu-id="a2704-346">ContentRoot</span><span class="sxs-lookup"><span data-stu-id="a2704-346">ContentRoot</span></span>](#contentroot)

## <a name="manage-the-host-lifetime"></a><span data-ttu-id="a2704-347">Gestire la durata dell'host</span><span class="sxs-lookup"><span data-stu-id="a2704-347">Manage the host lifetime</span></span>

<span data-ttu-id="a2704-348">Chiamare metodi sull'implementazione <xref:Microsoft.Extensions.Hosting.IHost> incorporata per avviare e arrestare l'app.</span><span class="sxs-lookup"><span data-stu-id="a2704-348">Call methods on the built <xref:Microsoft.Extensions.Hosting.IHost> implementation to start and stop the app.</span></span> <span data-ttu-id="a2704-349">Questi metodi influiscono su tutte le implementazioni <xref:Microsoft.Extensions.Hosting.IHostedService> che vengono registrate nel contenitore dei servizi.</span><span class="sxs-lookup"><span data-stu-id="a2704-349">These methods affect all  <xref:Microsoft.Extensions.Hosting.IHostedService> implementations that are registered in the service container.</span></span>

### <a name="run"></a><span data-ttu-id="a2704-350">Esegui</span><span class="sxs-lookup"><span data-stu-id="a2704-350">Run</span></span>

<span data-ttu-id="a2704-351"><xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.Run*> esegue l'app e blocca il thread di chiamata fino all'arresto dell'host.</span><span class="sxs-lookup"><span data-stu-id="a2704-351"><xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.Run*> runs the app and blocks the calling thread until the host is shut down.</span></span>

### <a name="runasync"></a><span data-ttu-id="a2704-352">RunAsync</span><span class="sxs-lookup"><span data-stu-id="a2704-352">RunAsync</span></span>

<span data-ttu-id="a2704-353"><xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.RunAsync*> esegue l'app e restituisce un elemento <xref:System.Threading.Tasks.Task> che viene completato quando viene attivato il token di annullamento o l'arresto.</span><span class="sxs-lookup"><span data-stu-id="a2704-353"><xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.RunAsync*> runs the app and returns a <xref:System.Threading.Tasks.Task> that completes when the cancellation token or shutdown is triggered.</span></span>

### <a name="runconsoleasync"></a><span data-ttu-id="a2704-354">RunConsoleAsync</span><span class="sxs-lookup"><span data-stu-id="a2704-354">RunConsoleAsync</span></span>

<span data-ttu-id="a2704-355"><xref:Microsoft.Extensions.Hosting.HostingHostBuilderExtensions.RunConsoleAsync*>Abilita il supporto della console, compila e avvia l'host e attende la <kbd></kbd> + chiusura di CTRL<kbd>C</kbd>/SIGINT o SIGTERM.</span><span class="sxs-lookup"><span data-stu-id="a2704-355"><xref:Microsoft.Extensions.Hosting.HostingHostBuilderExtensions.RunConsoleAsync*> enables console support, builds and starts the host, and waits for <kbd>Ctrl</kbd>+<kbd>C</kbd>/SIGINT or SIGTERM to shut down.</span></span>

### <a name="start"></a><span data-ttu-id="a2704-356">Inizia</span><span class="sxs-lookup"><span data-stu-id="a2704-356">Start</span></span>

<span data-ttu-id="a2704-357"><xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.Start*> avvia l'host in modo sincrono.</span><span class="sxs-lookup"><span data-stu-id="a2704-357"><xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.Start*> starts the host synchronously.</span></span>

### <a name="startasync"></a><span data-ttu-id="a2704-358">StartAsync</span><span class="sxs-lookup"><span data-stu-id="a2704-358">StartAsync</span></span>

<span data-ttu-id="a2704-359"><xref:Microsoft.Extensions.Hosting.IHost.StartAsync*> avvia l'host e restituisce un elemento <xref:System.Threading.Tasks.Task> che viene completato quando viene attivato il token di annullamento o l'arresto.</span><span class="sxs-lookup"><span data-stu-id="a2704-359"><xref:Microsoft.Extensions.Hosting.IHost.StartAsync*> starts the host and returns a <xref:System.Threading.Tasks.Task> that completes when the cancellation token or shutdown is triggered.</span></span> 

<span data-ttu-id="a2704-360"><xref:Microsoft.Extensions.Hosting.IHostLifetime.WaitForStartAsync*> viene chiamato all'avvio di `StartAsync`, che attende il completamento prima di continuare.</span><span class="sxs-lookup"><span data-stu-id="a2704-360"><xref:Microsoft.Extensions.Hosting.IHostLifetime.WaitForStartAsync*> is called at the start of `StartAsync`, which waits until it's complete before continuing.</span></span> <span data-ttu-id="a2704-361">Questo è utile per ritardare l'avvio fino al segnale trasmesso da un evento esterno.</span><span class="sxs-lookup"><span data-stu-id="a2704-361">This can be used to delay startup until signaled by an external event.</span></span>

### <a name="stopasync"></a><span data-ttu-id="a2704-362">StopAsync</span><span class="sxs-lookup"><span data-stu-id="a2704-362">StopAsync</span></span>

<span data-ttu-id="a2704-363"><xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.StopAsync*> prova ad arrestare l'host entro il timeout specificato.</span><span class="sxs-lookup"><span data-stu-id="a2704-363"><xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.StopAsync*> attempts to stop the host within the provided timeout.</span></span>

### <a name="waitforshutdown"></a><span data-ttu-id="a2704-364">WaitForShutdown</span><span class="sxs-lookup"><span data-stu-id="a2704-364">WaitForShutdown</span></span>

<span data-ttu-id="a2704-365"><xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.WaitForShutdown*>blocca il thread chiamante finché l'arresto non viene attivato da IHostLifetime, ad esempio tramite <kbd>CTRL</kbd> + <kbd>C</kbd>/SIGINT o SIGTERM.</span><span class="sxs-lookup"><span data-stu-id="a2704-365"><xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.WaitForShutdown*> blocks the calling thread until shutdown is triggered by the IHostLifetime, such as via <kbd>Ctrl</kbd>+<kbd>C</kbd>/SIGINT or SIGTERM.</span></span>

### <a name="waitforshutdownasync"></a><span data-ttu-id="a2704-366">WaitForShutdownAsync</span><span class="sxs-lookup"><span data-stu-id="a2704-366">WaitForShutdownAsync</span></span>

<span data-ttu-id="a2704-367"><xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.WaitForShutdownAsync*> restituisce un elemento <xref:System.Threading.Tasks.Task> che viene completato quando viene attivato l'arresto tramite il token specificato, quindi chiama <xref:Microsoft.Extensions.Hosting.IHost.StopAsync*>.</span><span class="sxs-lookup"><span data-stu-id="a2704-367"><xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.WaitForShutdownAsync*> returns a <xref:System.Threading.Tasks.Task> that completes when shutdown is triggered via the given token and calls <xref:Microsoft.Extensions.Hosting.IHost.StopAsync*>.</span></span>

### <a name="external-control"></a><span data-ttu-id="a2704-368">Controllo esterno</span><span class="sxs-lookup"><span data-stu-id="a2704-368">External control</span></span>

<span data-ttu-id="a2704-369">Il controllo diretto della durata dell'host può essere ottenuto usando metodi che possono essere chiamati dall'esterno:</span><span class="sxs-lookup"><span data-stu-id="a2704-369">Direct control of the host lifetime can be achieved using methods that can be called externally:</span></span>

```csharp
public class Program
{
    private IHost _host;

    public Program()
    {
        _host = new HostBuilder()
            .Build();
    }

    public async Task StartAsync()
    {
        _host.StartAsync();
    }

    public async Task StopAsync()
    {
        using (_host)
        {
            await _host.StopAsync(TimeSpan.FromSeconds(5));
        }
    }
}
```

::: moniker-end

::: moniker range=">= aspnetcore-3.0 < aspnetcore-5.0"

<span data-ttu-id="a2704-370">I modelli di ASP.NET Core creano un host generico .NET Core ( <xref:Microsoft.Extensions.Hosting.HostBuilder> ).</span><span class="sxs-lookup"><span data-stu-id="a2704-370">The ASP.NET Core templates create a .NET Core Generic Host (<xref:Microsoft.Extensions.Hosting.HostBuilder>).</span></span>

<span data-ttu-id="a2704-371">In questo argomento vengono fornite informazioni sull'utilizzo di host generico .NET in ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="a2704-371">This topic provides information on using .NET Generic Host in ASP.NET Core.</span></span> <span data-ttu-id="a2704-372">Per informazioni sull'uso di un host generico .NET nelle app console, vedere [.NET Generic Host](/dotnet/core/extensions/generic-host).</span><span class="sxs-lookup"><span data-stu-id="a2704-372">For information on using .NET Generic Host in console apps, see [.NET Generic Host](/dotnet/core/extensions/generic-host).</span></span>

## <a name="host-definition"></a><span data-ttu-id="a2704-373">Definizione host</span><span class="sxs-lookup"><span data-stu-id="a2704-373">Host definition</span></span>

<span data-ttu-id="a2704-374">Un *host* è un oggetto che incapsula le risorse di un'app, ad esempio:</span><span class="sxs-lookup"><span data-stu-id="a2704-374">A *host* is an object that encapsulates an app's resources, such as:</span></span>

* <span data-ttu-id="a2704-375">Inserimento di dipendenze (DI)</span><span class="sxs-lookup"><span data-stu-id="a2704-375">Dependency injection (DI)</span></span>
* <span data-ttu-id="a2704-376">Registrazione</span><span class="sxs-lookup"><span data-stu-id="a2704-376">Logging</span></span>
* <span data-ttu-id="a2704-377">Configurazione</span><span class="sxs-lookup"><span data-stu-id="a2704-377">Configuration</span></span>
* <span data-ttu-id="a2704-378">`IHostedService` Implementazioni</span><span class="sxs-lookup"><span data-stu-id="a2704-378">`IHostedService` implementations</span></span>

<span data-ttu-id="a2704-379">Quando viene avviato, un host chiama <xref:Microsoft.Extensions.Hosting.IHostedService.StartAsync%2A?displayProperty=nameWithType> su ogni implementazione di <xref:Microsoft.Extensions.Hosting.IHostedService> registrata nella raccolta di servizi ospitati del contenitore dei servizi.</span><span class="sxs-lookup"><span data-stu-id="a2704-379">When a host starts, it calls <xref:Microsoft.Extensions.Hosting.IHostedService.StartAsync%2A?displayProperty=nameWithType> on each implementation of <xref:Microsoft.Extensions.Hosting.IHostedService> registered in the service container's collection of hosted services.</span></span> <span data-ttu-id="a2704-380">In un'app Web, una delle implementazioni di `IHostedService` è un servizio web che avvia un'[implementazione del server HTTP](xref:fundamentals/index#servers).</span><span class="sxs-lookup"><span data-stu-id="a2704-380">In a web app, one of the `IHostedService` implementations is a web service that starts an [HTTP server implementation](xref:fundamentals/index#servers).</span></span>

<span data-ttu-id="a2704-381">Il motivo principale per cui tutte le risorse interdipendenti dell'app sono incluse in un unico oggetto è la gestione del ciclo di vita, vale a dire il controllo sull'avvio dell'app e sull'arresto normale.</span><span class="sxs-lookup"><span data-stu-id="a2704-381">The main reason for including all of the app's interdependent resources in one object is lifetime management: control over app startup and graceful shutdown.</span></span>

## <a name="set-up-a-host"></a><span data-ttu-id="a2704-382">Configurare un host</span><span class="sxs-lookup"><span data-stu-id="a2704-382">Set up a host</span></span>

<span data-ttu-id="a2704-383">L'host è in genere configurato, compilato ed eseguito da codice nella classe `Program`.</span><span class="sxs-lookup"><span data-stu-id="a2704-383">The host is typically configured, built, and run by code in the `Program` class.</span></span> <span data-ttu-id="a2704-384">Il metodo `Main`:</span><span class="sxs-lookup"><span data-stu-id="a2704-384">The `Main` method:</span></span>

* <span data-ttu-id="a2704-385">Chiama un metodo `CreateHostBuilder` per creare e configurare un oggetto generatore.</span><span class="sxs-lookup"><span data-stu-id="a2704-385">Calls a `CreateHostBuilder` method to create and configure a builder object.</span></span>
* <span data-ttu-id="a2704-386">Chiamate `Build` e metodi `Run` nell'oggetto generatore.</span><span class="sxs-lookup"><span data-stu-id="a2704-386">Calls `Build` and `Run` methods on the builder object.</span></span>

<span data-ttu-id="a2704-387">I modelli Web ASP.NET Core generano il codice seguente per creare un host generico:</span><span class="sxs-lookup"><span data-stu-id="a2704-387">The ASP.NET Core web templates generate the following code to create a Generic Host:</span></span>

```csharp
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
            });
}
```

<span data-ttu-id="a2704-388">Il codice seguente crea un host generico utilizzando un carico di lavoro non HTTP.</span><span class="sxs-lookup"><span data-stu-id="a2704-388">The following code creates a Generic Host using non-HTTP workload.</span></span> <span data-ttu-id="a2704-389">L' `IHostedService` implementazione viene aggiunta al contenitore DI di:</span><span class="sxs-lookup"><span data-stu-id="a2704-389">The `IHostedService` implementation is added to the DI container:</span></span>

```csharp
public class Program
{
    public static void Main(string[] args)
    {
        CreateHostBuilder(args).Build().Run();
    }

    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .ConfigureServices((hostContext, services) =>
            {
               services.AddHostedService<Worker>();
            });
}
```

<span data-ttu-id="a2704-390">Per un carico di lavoro HTTP, il metodo `Main` è lo stesso ma `CreateHostBuilder` chiama `ConfigureWebHostDefaults`:</span><span class="sxs-lookup"><span data-stu-id="a2704-390">For an HTTP workload, the `Main` method is the same but `CreateHostBuilder` calls `ConfigureWebHostDefaults`:</span></span>

```csharp
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.UseStartup<Startup>();
        });
```

<span data-ttu-id="a2704-391">Il codice precedente viene generato dai modelli di ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="a2704-391">The preceding code is generated by the ASP.NET Core templates.</span></span>

<span data-ttu-id="a2704-392">Se l'app usa Entity Framework Core, non modificare il nome o la firma del metodo `CreateHostBuilder`.</span><span class="sxs-lookup"><span data-stu-id="a2704-392">If the app uses Entity Framework Core, don't change the name or signature of the `CreateHostBuilder` method.</span></span> <span data-ttu-id="a2704-393">Gli [strumenti di Entity Framework Core](/ef/core/miscellaneous/cli/) si aspettano di trovare un metodo `CreateHostBuilder` che configura l'host senza eseguire l'app.</span><span class="sxs-lookup"><span data-stu-id="a2704-393">The [Entity Framework Core tools](/ef/core/miscellaneous/cli/) expect to find a `CreateHostBuilder` method that configures the host without running the app.</span></span> <span data-ttu-id="a2704-394">Per altre informazioni, vedere [Creazione DbContext in fase di progettazione](/ef/core/miscellaneous/cli/dbcontext-creation).</span><span class="sxs-lookup"><span data-stu-id="a2704-394">For more information, see [Design-time DbContext Creation](/ef/core/miscellaneous/cli/dbcontext-creation).</span></span>

## <a name="default-builder-settings"></a><span data-ttu-id="a2704-395">Impostazioni predefinite del generatore</span><span class="sxs-lookup"><span data-stu-id="a2704-395">Default builder settings</span></span>

<span data-ttu-id="a2704-396">Il metodo <xref:Microsoft.Extensions.Hosting.Host.CreateDefaultBuilder%2A>:</span><span class="sxs-lookup"><span data-stu-id="a2704-396">The <xref:Microsoft.Extensions.Hosting.Host.CreateDefaultBuilder%2A> method:</span></span>

* <span data-ttu-id="a2704-397">Imposta la [radice del contenuto](xref:fundamentals/index#content-root) sul percorso restituito da <xref:System.IO.Directory.GetCurrentDirectory%2A> .</span><span class="sxs-lookup"><span data-stu-id="a2704-397">Sets the [content root](xref:fundamentals/index#content-root) to the path returned by <xref:System.IO.Directory.GetCurrentDirectory%2A>.</span></span>
* <span data-ttu-id="a2704-398">Carica la configurazione dell'host da:</span><span class="sxs-lookup"><span data-stu-id="a2704-398">Loads host configuration from:</span></span>
  * <span data-ttu-id="a2704-399">Variabili di ambiente con prefisso `DOTNET_` .</span><span class="sxs-lookup"><span data-stu-id="a2704-399">Environment variables prefixed with `DOTNET_`.</span></span>
  * <span data-ttu-id="a2704-400">Argomenti della riga di comando.</span><span class="sxs-lookup"><span data-stu-id="a2704-400">Command-line arguments.</span></span>
* <span data-ttu-id="a2704-401">Carica la configurazione dell'app da:</span><span class="sxs-lookup"><span data-stu-id="a2704-401">Loads app configuration from:</span></span>
  * <span data-ttu-id="a2704-402">*appsettings.json*.</span><span class="sxs-lookup"><span data-stu-id="a2704-402">*appsettings.json*.</span></span>
  * <span data-ttu-id="a2704-403">*appsettings.{Environment}.json*.</span><span class="sxs-lookup"><span data-stu-id="a2704-403">*appsettings.{Environment}.json*.</span></span>
  * <span data-ttu-id="a2704-404">[Segreti dell'utente](xref:security/app-secrets) quando l'app viene eseguita nell'ambiente `Development`.</span><span class="sxs-lookup"><span data-stu-id="a2704-404">[User secrets](xref:security/app-secrets) when the app runs in the `Development` environment.</span></span>
  * <span data-ttu-id="a2704-405">Variabili di ambiente.</span><span class="sxs-lookup"><span data-stu-id="a2704-405">Environment variables.</span></span>
  * <span data-ttu-id="a2704-406">Argomenti della riga di comando.</span><span class="sxs-lookup"><span data-stu-id="a2704-406">Command-line arguments.</span></span>
* <span data-ttu-id="a2704-407">Aggiunge i provider di [log](xref:fundamentals/logging/index) seguenti:</span><span class="sxs-lookup"><span data-stu-id="a2704-407">Adds the following [logging](xref:fundamentals/logging/index) providers:</span></span>
  * <span data-ttu-id="a2704-408">Console</span><span class="sxs-lookup"><span data-stu-id="a2704-408">Console</span></span>
  * <span data-ttu-id="a2704-409">Debug</span><span class="sxs-lookup"><span data-stu-id="a2704-409">Debug</span></span>
  * <span data-ttu-id="a2704-410">EventSource</span><span class="sxs-lookup"><span data-stu-id="a2704-410">EventSource</span></span>
  * <span data-ttu-id="a2704-411">EventLog (solo quando è in esecuzione su Windows)</span><span class="sxs-lookup"><span data-stu-id="a2704-411">EventLog (only when running on Windows)</span></span>
* <span data-ttu-id="a2704-412">Abilita la [convalida dell'ambito](xref:fundamentals/dependency-injection#scope-validation) e la [convalida delle dipendenze](xref:Microsoft.Extensions.DependencyInjection.ServiceProviderOptions.ValidateOnBuild) quando l'ambiente è lo sviluppo.</span><span class="sxs-lookup"><span data-stu-id="a2704-412">Enables [scope validation](xref:fundamentals/dependency-injection#scope-validation) and [dependency validation](xref:Microsoft.Extensions.DependencyInjection.ServiceProviderOptions.ValidateOnBuild) when the environment is Development.</span></span>

<span data-ttu-id="a2704-413">Il metodo `ConfigureWebHostDefaults`:</span><span class="sxs-lookup"><span data-stu-id="a2704-413">The `ConfigureWebHostDefaults` method:</span></span>

* <span data-ttu-id="a2704-414">Carica la configurazione host dalle variabili di ambiente con prefisso `ASPNETCORE_` .</span><span class="sxs-lookup"><span data-stu-id="a2704-414">Loads host configuration from environment variables prefixed with `ASPNETCORE_`.</span></span>
* <span data-ttu-id="a2704-415">Imposta il server [Kestrel](xref:fundamentals/servers/kestrel) come server Web e lo configura usando i provider di configurazione dell'host dell'app.</span><span class="sxs-lookup"><span data-stu-id="a2704-415">Sets [Kestrel](xref:fundamentals/servers/kestrel) server as the web server and configures it using the app's hosting configuration providers.</span></span> <span data-ttu-id="a2704-416">Per le opzioni predefinite del server Kestrel, vedere <xref:fundamentals/servers/kestrel#kestrel-options>.</span><span class="sxs-lookup"><span data-stu-id="a2704-416">For the Kestrel server's default options, see <xref:fundamentals/servers/kestrel#kestrel-options>.</span></span>
* <span data-ttu-id="a2704-417">Aggiunge [il middleware dell'applicazione di filtri dell'host](xref:fundamentals/servers/kestrel#host-filtering).</span><span class="sxs-lookup"><span data-stu-id="a2704-417">Adds [Host Filtering middleware](xref:fundamentals/servers/kestrel#host-filtering).</span></span>
* <span data-ttu-id="a2704-418">Aggiunge il [middleware delle intestazioni con inoltri](xref:host-and-deploy/proxy-load-balancer#forwarded-headers) se `ASPNETCORE_FORWARDEDHEADERS_ENABLED` è uguale a `true` .</span><span class="sxs-lookup"><span data-stu-id="a2704-418">Adds [Forwarded Headers middleware](xref:host-and-deploy/proxy-load-balancer#forwarded-headers) if `ASPNETCORE_FORWARDEDHEADERS_ENABLED` equals `true`.</span></span>
* <span data-ttu-id="a2704-419">Abilita l'integrazione di IIS.</span><span class="sxs-lookup"><span data-stu-id="a2704-419">Enables IIS integration.</span></span> <span data-ttu-id="a2704-420">Per le opzioni predefinite di IIS, vedere <xref:host-and-deploy/iis/index#iis-options>.</span><span class="sxs-lookup"><span data-stu-id="a2704-420">For the IIS default options, see <xref:host-and-deploy/iis/index#iis-options>.</span></span>

<span data-ttu-id="a2704-421">Le sezioni [Impostazioni per tutti i tipi di app](#settings-for-all-app-types) e [Impostazioni per le app Web](#settings-for-web-apps) più avanti in questo articolo mostrano come eseguire l'override delle impostazioni predefinite del generatore.</span><span class="sxs-lookup"><span data-stu-id="a2704-421">The [Settings for all app types](#settings-for-all-app-types) and [Settings for web apps](#settings-for-web-apps) sections later in this article show how to override default builder settings.</span></span>

## <a name="framework-provided-services"></a><span data-ttu-id="a2704-422">Servizi forniti dal framework</span><span class="sxs-lookup"><span data-stu-id="a2704-422">Framework-provided services</span></span>

<span data-ttu-id="a2704-423">I servizi seguenti vengono registrati automaticamente:</span><span class="sxs-lookup"><span data-stu-id="a2704-423">The following services are registered automatically:</span></span>

* [<span data-ttu-id="a2704-424">IHostApplicationLifetime</span><span class="sxs-lookup"><span data-stu-id="a2704-424">IHostApplicationLifetime</span></span>](#ihostapplicationlifetime)
* [<span data-ttu-id="a2704-425">IHostLifetime</span><span class="sxs-lookup"><span data-stu-id="a2704-425">IHostLifetime</span></span>](#ihostlifetime)
* [<span data-ttu-id="a2704-426">IHostEnvironment / IWebHostEnvironment</span><span class="sxs-lookup"><span data-stu-id="a2704-426">IHostEnvironment / IWebHostEnvironment</span></span>](#ihostenvironment)

<span data-ttu-id="a2704-427">Per ulteriori informazioni sui servizi forniti dal Framework, vedere <xref:fundamentals/dependency-injection#framework-provided-services> .</span><span class="sxs-lookup"><span data-stu-id="a2704-427">For more information on framework-provided services, see <xref:fundamentals/dependency-injection#framework-provided-services>.</span></span>

## <a name="ihostapplicationlifetime"></a><span data-ttu-id="a2704-428">IHostApplicationLifetime</span><span class="sxs-lookup"><span data-stu-id="a2704-428">IHostApplicationLifetime</span></span>

<span data-ttu-id="a2704-429">Inserire il servizio <xref:Microsoft.Extensions.Hosting.IHostApplicationLifetime> (in precedenza `IApplicationLifetime`) in qualsiasi classe per gestire le attività post-avvio e di arresto normale.</span><span class="sxs-lookup"><span data-stu-id="a2704-429">Inject the <xref:Microsoft.Extensions.Hosting.IHostApplicationLifetime> (formerly `IApplicationLifetime`) service into any class to handle post-startup and graceful shutdown tasks.</span></span> <span data-ttu-id="a2704-430">Tre proprietà nell'interfaccia sono token di annullamento usati per registrare i metodi del gestore dell'evento di avvio e di arresto.</span><span class="sxs-lookup"><span data-stu-id="a2704-430">Three properties on the interface are cancellation tokens used to register app start and app stop event handler methods.</span></span> <span data-ttu-id="a2704-431">L'interfaccia include anche un metodo `StopApplication`.</span><span class="sxs-lookup"><span data-stu-id="a2704-431">The interface also includes a `StopApplication` method.</span></span>

<span data-ttu-id="a2704-432">L'esempio seguente è un' `IHostedService` implementazione di che registra `IHostApplicationLifetime` gli eventi:</span><span class="sxs-lookup"><span data-stu-id="a2704-432">The following example is an `IHostedService` implementation that registers `IHostApplicationLifetime` events:</span></span>

[!code-csharp[](generic-host/samples-snapshot/3.x/LifetimeEventsHostedService.cs?name=snippet_LifetimeEvents)]

## <a name="ihostlifetime"></a><span data-ttu-id="a2704-433">IHostLifetime</span><span class="sxs-lookup"><span data-stu-id="a2704-433">IHostLifetime</span></span>

<span data-ttu-id="a2704-434">L'implementazione <xref:Microsoft.Extensions.Hosting.IHostLifetime> controlla quando l'host viene avviato e quando si arresta.</span><span class="sxs-lookup"><span data-stu-id="a2704-434">The <xref:Microsoft.Extensions.Hosting.IHostLifetime> implementation controls when the host starts and when it stops.</span></span> <span data-ttu-id="a2704-435">Viene usata l'ultima implementazione registrata.</span><span class="sxs-lookup"><span data-stu-id="a2704-435">The last implementation registered is used.</span></span>

<span data-ttu-id="a2704-436">`Microsoft.Extensions.Hosting.Internal.ConsoleLifetime` è l'implementazione `IHostLifetime` predefinita.</span><span class="sxs-lookup"><span data-stu-id="a2704-436">`Microsoft.Extensions.Hosting.Internal.ConsoleLifetime` is the default `IHostLifetime` implementation.</span></span> <span data-ttu-id="a2704-437">`ConsoleLifetime`:</span><span class="sxs-lookup"><span data-stu-id="a2704-437">`ConsoleLifetime`:</span></span>

* <span data-ttu-id="a2704-438">Ascolta la <kbd>combinazione di tasti CTRL</kbd> + <kbd>C</kbd>/SIGINT o SIGTERM e chiama <xref:Microsoft.Extensions.Hosting.IHostApplicationLifetime.StopApplication%2A> per avviare il processo di arresto.</span><span class="sxs-lookup"><span data-stu-id="a2704-438">Listens for <kbd>Ctrl</kbd>+<kbd>C</kbd>/SIGINT or SIGTERM and calls <xref:Microsoft.Extensions.Hosting.IHostApplicationLifetime.StopApplication%2A> to start the shutdown process.</span></span>
* <span data-ttu-id="a2704-439">Sblocca estensioni come [RunAsync](#runasync) e [WaitForShutdownAsync](#waitforshutdownasync).</span><span class="sxs-lookup"><span data-stu-id="a2704-439">Unblocks extensions such as [RunAsync](#runasync) and [WaitForShutdownAsync](#waitforshutdownasync).</span></span>

## <a name="ihostenvironment"></a><span data-ttu-id="a2704-440">IHostEnvironment</span><span class="sxs-lookup"><span data-stu-id="a2704-440">IHostEnvironment</span></span>

<span data-ttu-id="a2704-441">Inserire il <xref:Microsoft.Extensions.Hosting.IHostEnvironment> servizio in una classe per ottenere informazioni sulle impostazioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="a2704-441">Inject the <xref:Microsoft.Extensions.Hosting.IHostEnvironment> service into a class to get information about the following settings:</span></span>

* [<span data-ttu-id="a2704-442">ApplicationName</span><span class="sxs-lookup"><span data-stu-id="a2704-442">ApplicationName</span></span>](#applicationname)
* [<span data-ttu-id="a2704-443">EnvironmentName</span><span class="sxs-lookup"><span data-stu-id="a2704-443">EnvironmentName</span></span>](#environmentname)
* [<span data-ttu-id="a2704-444">ContentRootPath</span><span class="sxs-lookup"><span data-stu-id="a2704-444">ContentRootPath</span></span>](#contentroot)

<span data-ttu-id="a2704-445">Le app Web implementano l' `IWebHostEnvironment` interfaccia, che eredita `IHostEnvironment` e aggiunge [WebRootPath](#webroot).</span><span class="sxs-lookup"><span data-stu-id="a2704-445">Web apps implement the `IWebHostEnvironment` interface, which inherits `IHostEnvironment` and adds the [WebRootPath](#webroot).</span></span>

## <a name="host-configuration"></a><span data-ttu-id="a2704-446">Configurazione dell'host</span><span class="sxs-lookup"><span data-stu-id="a2704-446">Host configuration</span></span>

<span data-ttu-id="a2704-447">La configurazione dell'host viene usata per le proprietà dell'implementazione <xref:Microsoft.Extensions.Hosting.IHostEnvironment>.</span><span class="sxs-lookup"><span data-stu-id="a2704-447">Host configuration is used for the properties of the <xref:Microsoft.Extensions.Hosting.IHostEnvironment> implementation.</span></span>

<span data-ttu-id="a2704-448">La configurazione dell'host è disponibile da [HostBuilderContext.Configuration](xref:Microsoft.Extensions.Hosting.HostBuilderContext.Configuration) all'interno di <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureAppConfiguration%2A>.</span><span class="sxs-lookup"><span data-stu-id="a2704-448">Host configuration is available from [HostBuilderContext.Configuration](xref:Microsoft.Extensions.Hosting.HostBuilderContext.Configuration) inside <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureAppConfiguration%2A>.</span></span> <span data-ttu-id="a2704-449">Dopo `ConfigureAppConfiguration`, `HostBuilderContext.Configuration` viene sostituito con la configurazione dell'app.</span><span class="sxs-lookup"><span data-stu-id="a2704-449">After `ConfigureAppConfiguration`, `HostBuilderContext.Configuration` is replaced with the app config.</span></span>

<span data-ttu-id="a2704-450">Per aggiungere la configurazione dell'host, chiamare <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureHostConfiguration%2A> su `IHostBuilder`.</span><span class="sxs-lookup"><span data-stu-id="a2704-450">To add host configuration, call <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureHostConfiguration%2A> on `IHostBuilder`.</span></span> <span data-ttu-id="a2704-451">`ConfigureHostConfiguration` può essere chiamato più volte e i risultati si sommano.</span><span class="sxs-lookup"><span data-stu-id="a2704-451">`ConfigureHostConfiguration` can be called multiple times with additive results.</span></span> <span data-ttu-id="a2704-452">L'host usa l'ultima opzione che imposta un valore in una determinata chiave.</span><span class="sxs-lookup"><span data-stu-id="a2704-452">The host uses whichever option sets a value last on a given key.</span></span>

<span data-ttu-id="a2704-453">Il provider di variabili di ambiente con prefisso `DOTNET_` e argomenti della riga di comando sono inclusi in `CreateDefaultBuilder` .</span><span class="sxs-lookup"><span data-stu-id="a2704-453">The environment variable provider with prefix `DOTNET_` and command-line arguments are included by `CreateDefaultBuilder`.</span></span> <span data-ttu-id="a2704-454">Per le app Web, viene aggiunto il provider di variabili di ambiente con prefisso `ASPNETCORE_`.</span><span class="sxs-lookup"><span data-stu-id="a2704-454">For web apps, the environment variable provider with prefix `ASPNETCORE_` is added.</span></span> <span data-ttu-id="a2704-455">Il prefisso viene rimosso quando vengono lette le variabili di ambiente.</span><span class="sxs-lookup"><span data-stu-id="a2704-455">The prefix is removed when the environment variables are read.</span></span> <span data-ttu-id="a2704-456">Ad esempio, il valore della variabile di ambiente per `ASPNETCORE_ENVIRONMENT` diventa il valore di configurazione dell'host per la chiave `environment`.</span><span class="sxs-lookup"><span data-stu-id="a2704-456">For example, the environment variable value for `ASPNETCORE_ENVIRONMENT` becomes the host configuration value for the `environment` key.</span></span>

<span data-ttu-id="a2704-457">L'esempio seguente crea la configurazione host:</span><span class="sxs-lookup"><span data-stu-id="a2704-457">The following example creates host configuration:</span></span>

[!code-csharp[](generic-host/samples-snapshot/3.x/Program.cs?name=snippet_HostConfig)]

## <a name="app-configuration"></a><span data-ttu-id="a2704-458">Configurazione delle app</span><span class="sxs-lookup"><span data-stu-id="a2704-458">App configuration</span></span>

<span data-ttu-id="a2704-459">La configurazione dell'app viene creata chiamando <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureAppConfiguration*> su `IHostBuilder`.</span><span class="sxs-lookup"><span data-stu-id="a2704-459">App configuration is created by calling <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureAppConfiguration*> on `IHostBuilder`.</span></span> <span data-ttu-id="a2704-460">`ConfigureAppConfiguration` può essere chiamato più volte e i risultati si sommano.</span><span class="sxs-lookup"><span data-stu-id="a2704-460">`ConfigureAppConfiguration` can be called multiple times with additive results.</span></span> <span data-ttu-id="a2704-461">L'app usa l'ultima opzione che imposta un valore in una determinata chiave.</span><span class="sxs-lookup"><span data-stu-id="a2704-461">The app uses whichever option sets a value last on a given key.</span></span> 

<span data-ttu-id="a2704-462">La configurazione creata da `ConfigureAppConfiguration` è disponibile in [HostBuilderContext.Configuration](xref:Microsoft.Extensions.Hosting.HostBuilderContext.Configuration*) per le operazioni successive e come servizio da DI.</span><span class="sxs-lookup"><span data-stu-id="a2704-462">The configuration created by `ConfigureAppConfiguration` is available at [HostBuilderContext.Configuration](xref:Microsoft.Extensions.Hosting.HostBuilderContext.Configuration*) for subsequent operations and as a service from DI.</span></span> <span data-ttu-id="a2704-463">La configurazione dell'host viene aggiunta anche alla configurazione dell'app.</span><span class="sxs-lookup"><span data-stu-id="a2704-463">The host configuration is also added to the app configuration.</span></span>

<span data-ttu-id="a2704-464">Per altre informazioni, vedere [Configurazione in ASP.NET Core](xref:fundamentals/configuration/index#configureappconfiguration).</span><span class="sxs-lookup"><span data-stu-id="a2704-464">For more information, see [Configuration in ASP.NET Core](xref:fundamentals/configuration/index#configureappconfiguration).</span></span>

## <a name="settings-for-all-app-types"></a><span data-ttu-id="a2704-465">Impostazioni per tutti i tipi di app</span><span class="sxs-lookup"><span data-stu-id="a2704-465">Settings for all app types</span></span>

<span data-ttu-id="a2704-466">Questa sezione elenca le impostazioni dell'host che si applicano ai carichi di lavoro HTTP e non-HTTP.</span><span class="sxs-lookup"><span data-stu-id="a2704-466">This section lists host settings that apply to both HTTP and non-HTTP workloads.</span></span> <span data-ttu-id="a2704-467">Per impostazione predefinita, le variabili di ambiente usate per configurare queste impostazioni possono avere un prefisso `DOTNET_` o `ASPNETCORE_`.</span><span class="sxs-lookup"><span data-stu-id="a2704-467">By default, environment variables used to configure these settings can have a `DOTNET_` or `ASPNETCORE_` prefix.</span></span>

<!-- In the following sections, two spaces at end of line are used to force line breaks in the rendered page. -->

### <a name="applicationname"></a><span data-ttu-id="a2704-468">ApplicationName</span><span class="sxs-lookup"><span data-stu-id="a2704-468">ApplicationName</span></span>

<span data-ttu-id="a2704-469">La proprietà [IHostEnvironment.ApplicationName](xref:Microsoft.Extensions.Hosting.IHostEnvironment.ApplicationName*) viene impostata dalla configurazione dell'host durante la costruzione dell'host.</span><span class="sxs-lookup"><span data-stu-id="a2704-469">The [IHostEnvironment.ApplicationName](xref:Microsoft.Extensions.Hosting.IHostEnvironment.ApplicationName*) property is set from host configuration during host construction.</span></span>

<span data-ttu-id="a2704-470">**Chiave**: `applicationName`</span><span class="sxs-lookup"><span data-stu-id="a2704-470">**Key**: `applicationName`</span></span>  
<span data-ttu-id="a2704-471">**Tipo**: `string`</span><span class="sxs-lookup"><span data-stu-id="a2704-471">**Type**: `string`</span></span>  
<span data-ttu-id="a2704-472">**Impostazione predefinita**: il nome dell'assembly che contiene il punto di ingresso dell'app.</span><span class="sxs-lookup"><span data-stu-id="a2704-472">**Default**: The name of the assembly that contains the app's entry point.</span></span>  
<span data-ttu-id="a2704-473">**Variabile di ambiente**: `<PREFIX_>APPLICATIONNAME`</span><span class="sxs-lookup"><span data-stu-id="a2704-473">**Environment variable**: `<PREFIX_>APPLICATIONNAME`</span></span>

<span data-ttu-id="a2704-474">per impostare questo valore usare la variabile di ambiente.</span><span class="sxs-lookup"><span data-stu-id="a2704-474">To set this value, use the environment variable.</span></span> 

### <a name="contentroot"></a><span data-ttu-id="a2704-475">ContentRoot</span><span class="sxs-lookup"><span data-stu-id="a2704-475">ContentRoot</span></span>

<span data-ttu-id="a2704-476">La proprietà [IHostEnvironment.ContentRootPath](xref:Microsoft.Extensions.Hosting.IHostEnvironment.ContentRootPath*) determina il punto in cui l'host inizia a cercare i file di contenuto.</span><span class="sxs-lookup"><span data-stu-id="a2704-476">The [IHostEnvironment.ContentRootPath](xref:Microsoft.Extensions.Hosting.IHostEnvironment.ContentRootPath*) property determines where the host begins searching for content files.</span></span> <span data-ttu-id="a2704-477">Se il percorso non esiste, l'host non verrà avviato.</span><span class="sxs-lookup"><span data-stu-id="a2704-477">If the path doesn't exist, the host fails to start.</span></span>

<span data-ttu-id="a2704-478">**Chiave**: `contentRoot`</span><span class="sxs-lookup"><span data-stu-id="a2704-478">**Key**: `contentRoot`</span></span>  
<span data-ttu-id="a2704-479">**Tipo**: `string`</span><span class="sxs-lookup"><span data-stu-id="a2704-479">**Type**: `string`</span></span>  
<span data-ttu-id="a2704-480">**Impostazione predefinita**: la cartella in cui risiede l'assembly dell'app.</span><span class="sxs-lookup"><span data-stu-id="a2704-480">**Default**: The folder where the app assembly resides.</span></span>  
<span data-ttu-id="a2704-481">**Variabile di ambiente**: `<PREFIX_>CONTENTROOT`</span><span class="sxs-lookup"><span data-stu-id="a2704-481">**Environment variable**: `<PREFIX_>CONTENTROOT`</span></span>

<span data-ttu-id="a2704-482">Per impostare questo valore, usare la variabile di ambiente o chiamare `UseContentRoot` su `IHostBuilder`:</span><span class="sxs-lookup"><span data-stu-id="a2704-482">To set this value, use the environment variable or call `UseContentRoot` on `IHostBuilder`:</span></span>

```csharp
Host.CreateDefaultBuilder(args)
    .UseContentRoot("c:\\content-root")
    //...
```

<span data-ttu-id="a2704-483">Per altre informazioni, vedere:</span><span class="sxs-lookup"><span data-stu-id="a2704-483">For more information, see:</span></span>

* [<span data-ttu-id="a2704-484">Nozioni fondamentali: radice del contenuto</span><span class="sxs-lookup"><span data-stu-id="a2704-484">Fundamentals: Content root</span></span>](xref:fundamentals/index#content-root)
* [<span data-ttu-id="a2704-485">WebRoot</span><span class="sxs-lookup"><span data-stu-id="a2704-485">WebRoot</span></span>](#webroot)

### <a name="environmentname"></a><span data-ttu-id="a2704-486">EnvironmentName</span><span class="sxs-lookup"><span data-stu-id="a2704-486">EnvironmentName</span></span>

<span data-ttu-id="a2704-487">La proprietà [IHostEnvironment.EnvironmentName](xref:Microsoft.Extensions.Hosting.IHostEnvironment.EnvironmentName*) può essere impostata su qualsiasi valore.</span><span class="sxs-lookup"><span data-stu-id="a2704-487">The [IHostEnvironment.EnvironmentName](xref:Microsoft.Extensions.Hosting.IHostEnvironment.EnvironmentName*) property can be set to any value.</span></span> <span data-ttu-id="a2704-488">I valori definiti dal framework includono `Development`, `Staging` e `Production`.</span><span class="sxs-lookup"><span data-stu-id="a2704-488">Framework-defined values include `Development`, `Staging`, and `Production`.</span></span> <span data-ttu-id="a2704-489">I valori non fanno distinzione tra maiuscole e minuscole.</span><span class="sxs-lookup"><span data-stu-id="a2704-489">Values aren't case-sensitive.</span></span>

<span data-ttu-id="a2704-490">**Chiave**: `environment`</span><span class="sxs-lookup"><span data-stu-id="a2704-490">**Key**: `environment`</span></span>  
<span data-ttu-id="a2704-491">**Tipo**: `string`</span><span class="sxs-lookup"><span data-stu-id="a2704-491">**Type**: `string`</span></span>  
<span data-ttu-id="a2704-492">**Impostazione predefinita**: `Production`</span><span class="sxs-lookup"><span data-stu-id="a2704-492">**Default**: `Production`</span></span>  
<span data-ttu-id="a2704-493">**Variabile di ambiente**: `<PREFIX_>ENVIRONMENT`</span><span class="sxs-lookup"><span data-stu-id="a2704-493">**Environment variable**: `<PREFIX_>ENVIRONMENT`</span></span>

<span data-ttu-id="a2704-494">Per impostare questo valore, usare la variabile di ambiente o chiamare `UseEnvironment` su `IHostBuilder`:</span><span class="sxs-lookup"><span data-stu-id="a2704-494">To set this value, use the environment variable or call `UseEnvironment` on `IHostBuilder`:</span></span>

```csharp
Host.CreateDefaultBuilder(args)
    .UseEnvironment("Development")
    //...
```

### <a name="shutdowntimeout"></a><span data-ttu-id="a2704-495">ShutdownTimeout</span><span class="sxs-lookup"><span data-stu-id="a2704-495">ShutdownTimeout</span></span>

<span data-ttu-id="a2704-496">[HostOptions.ShutdownTimeout](xref:Microsoft.Extensions.Hosting.HostOptions.ShutdownTimeout*) imposta il timeout per <xref:Microsoft.Extensions.Hosting.IHost.StopAsync*>.</span><span class="sxs-lookup"><span data-stu-id="a2704-496">[HostOptions.ShutdownTimeout](xref:Microsoft.Extensions.Hosting.HostOptions.ShutdownTimeout*) sets the timeout for <xref:Microsoft.Extensions.Hosting.IHost.StopAsync*>.</span></span> <span data-ttu-id="a2704-497">Il valore predefinito è cinque secondi.</span><span class="sxs-lookup"><span data-stu-id="a2704-497">The default value is five seconds.</span></span>  <span data-ttu-id="a2704-498">Durante il periodo di timeout, l'host:</span><span class="sxs-lookup"><span data-stu-id="a2704-498">During the timeout period, the host:</span></span>

* <span data-ttu-id="a2704-499">attiva [IHostApplicationLifetime.ApplicationStopping](/dotnet/api/microsoft.extensions.hosting.ihostapplicationlifetime.applicationstopping).</span><span class="sxs-lookup"><span data-stu-id="a2704-499">Triggers [IHostApplicationLifetime.ApplicationStopping](/dotnet/api/microsoft.extensions.hosting.ihostapplicationlifetime.applicationstopping).</span></span>
* <span data-ttu-id="a2704-500">Tenta di arrestare i servizi ospitati, registrando gli errori per i servizi che non si interrompono.</span><span class="sxs-lookup"><span data-stu-id="a2704-500">Attempts to stop hosted services, logging errors for services that fail to stop.</span></span>

<span data-ttu-id="a2704-501">Se il periodo di timeout scade prima che siano stati arrestati tutti i servizi ospitati, gli eventuali servizi attivi rimanenti vengono interrotti quando l'app viene arrestata.</span><span class="sxs-lookup"><span data-stu-id="a2704-501">If the timeout period expires before all of the hosted services stop, any remaining active services are stopped when the app shuts down.</span></span> <span data-ttu-id="a2704-502">I servizi si arrestano anche se non hanno completato l'elaborazione.</span><span class="sxs-lookup"><span data-stu-id="a2704-502">The services stop even if they haven't finished processing.</span></span> <span data-ttu-id="a2704-503">Se l'arresto dei servizi richiede più tempo, aumentare il valore di timeout.</span><span class="sxs-lookup"><span data-stu-id="a2704-503">If services require additional time to stop, increase the timeout.</span></span>

<span data-ttu-id="a2704-504">**Chiave**: `shutdownTimeoutSeconds`</span><span class="sxs-lookup"><span data-stu-id="a2704-504">**Key**: `shutdownTimeoutSeconds`</span></span>  
<span data-ttu-id="a2704-505">**Tipo**: `int`</span><span class="sxs-lookup"><span data-stu-id="a2704-505">**Type**: `int`</span></span>  
<span data-ttu-id="a2704-506">**Impostazione predefinita**: 5 secondi</span><span class="sxs-lookup"><span data-stu-id="a2704-506">**Default**: 5 seconds</span></span>  
<span data-ttu-id="a2704-507">**Variabile di ambiente**: `<PREFIX_>SHUTDOWNTIMEOUTSECONDS`</span><span class="sxs-lookup"><span data-stu-id="a2704-507">**Environment variable**: `<PREFIX_>SHUTDOWNTIMEOUTSECONDS`</span></span>

<span data-ttu-id="a2704-508">Per impostare questo valore, usare la variabile di ambiente o configurare `HostOptions`.</span><span class="sxs-lookup"><span data-stu-id="a2704-508">To set this value, use the environment variable or configure `HostOptions`.</span></span> <span data-ttu-id="a2704-509">Nell'esempio seguente il timeout viene impostato su 20 secondi:</span><span class="sxs-lookup"><span data-stu-id="a2704-509">The following example sets the timeout to 20 seconds:</span></span>

[!code-csharp[](generic-host/samples-snapshot/3.x/Program.cs?name=snippet_HostOptions)]

## <a name="settings-for-web-apps"></a><span data-ttu-id="a2704-510">Impostazioni per le app Web</span><span class="sxs-lookup"><span data-stu-id="a2704-510">Settings for web apps</span></span>

<span data-ttu-id="a2704-511">Alcune impostazioni host si applicano solo ai carichi di lavoro HTTP.</span><span class="sxs-lookup"><span data-stu-id="a2704-511">Some host settings apply only to HTTP workloads.</span></span> <span data-ttu-id="a2704-512">Per impostazione predefinita, le variabili di ambiente usate per configurare queste impostazioni possono avere un prefisso `DOTNET_` o `ASPNETCORE_`.</span><span class="sxs-lookup"><span data-stu-id="a2704-512">By default, environment variables used to configure these settings can have a `DOTNET_` or `ASPNETCORE_` prefix.</span></span>

<span data-ttu-id="a2704-513">Per queste impostazioni sono disponibili i metodi di estensione su `IWebHostBuilder`.</span><span class="sxs-lookup"><span data-stu-id="a2704-513">Extension methods on `IWebHostBuilder` are available for these settings.</span></span> <span data-ttu-id="a2704-514">Gli esempi di codice che illustrano come chiamare i metodi di estensione presumono che `webBuilder` sia un'istanza di `IWebHostBuilder`, come nell'esempio seguente:</span><span class="sxs-lookup"><span data-stu-id="a2704-514">Code samples that show how to call the extension methods assume `webBuilder` is an instance of `IWebHostBuilder`, as in the following example:</span></span>

```csharp
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.CaptureStartupErrors(true);
            webBuilder.UseStartup<Startup>();
        });
```

### <a name="capturestartuperrors"></a><span data-ttu-id="a2704-515">CaptureStartupErrors</span><span class="sxs-lookup"><span data-stu-id="a2704-515">CaptureStartupErrors</span></span>

<span data-ttu-id="a2704-516">Quando è `false`, gli errori durante l'avvio causano la chiusura dell'host.</span><span class="sxs-lookup"><span data-stu-id="a2704-516">When `false`, errors during startup result in the host exiting.</span></span> <span data-ttu-id="a2704-517">Quando è `true`, l'host acquisisce le eccezioni durante l'avvio e tenta di avviare il server.</span><span class="sxs-lookup"><span data-stu-id="a2704-517">When `true`, the host captures exceptions during startup and attempts to start the server.</span></span>

<span data-ttu-id="a2704-518">**Chiave**: `captureStartupErrors`</span><span class="sxs-lookup"><span data-stu-id="a2704-518">**Key**: `captureStartupErrors`</span></span>  
<span data-ttu-id="a2704-519">**Tipo**: `bool` ( `true` o `1` )</span><span class="sxs-lookup"><span data-stu-id="a2704-519">**Type**: `bool` (`true` or `1`)</span></span>  
<span data-ttu-id="a2704-520">**Impostazione predefinita**: il valore predefinito è `false` a meno che l'app non venga eseguita con Kestrel in IIS, in tal caso il valore predefinito è `true`.</span><span class="sxs-lookup"><span data-stu-id="a2704-520">**Default**: Defaults to `false` unless the app runs with Kestrel behind IIS, where the default is `true`.</span></span>  
<span data-ttu-id="a2704-521">**Variabile di ambiente**: `<PREFIX_>CAPTURESTARTUPERRORS`</span><span class="sxs-lookup"><span data-stu-id="a2704-521">**Environment variable**: `<PREFIX_>CAPTURESTARTUPERRORS`</span></span>

<span data-ttu-id="a2704-522">Per impostare questo valore, usare la configurazione o chiamare `CaptureStartupErrors`:</span><span class="sxs-lookup"><span data-stu-id="a2704-522">To set this value, use configuration or call `CaptureStartupErrors`:</span></span>

```csharp
webBuilder.CaptureStartupErrors(true);
```

### <a name="detailederrors"></a><span data-ttu-id="a2704-523">DetailedErrors</span><span class="sxs-lookup"><span data-stu-id="a2704-523">DetailedErrors</span></span>

<span data-ttu-id="a2704-524">Quando è abilitata o quando l'ambiente è `Development`, l'app acquisisce gli errori dettagliati.</span><span class="sxs-lookup"><span data-stu-id="a2704-524">When enabled, or when the environment is `Development`, the app captures detailed errors.</span></span>

<span data-ttu-id="a2704-525">**Chiave**: `detailedErrors`</span><span class="sxs-lookup"><span data-stu-id="a2704-525">**Key**: `detailedErrors`</span></span>  
<span data-ttu-id="a2704-526">**Tipo**: `bool` ( `true` o `1` )</span><span class="sxs-lookup"><span data-stu-id="a2704-526">**Type**: `bool` (`true` or `1`)</span></span>  
<span data-ttu-id="a2704-527">**Impostazione predefinita**: `false`</span><span class="sxs-lookup"><span data-stu-id="a2704-527">**Default**: `false`</span></span>  
<span data-ttu-id="a2704-528">**Variabile di ambiente**: `<PREFIX_>_DETAILEDERRORS`</span><span class="sxs-lookup"><span data-stu-id="a2704-528">**Environment variable**: `<PREFIX_>_DETAILEDERRORS`</span></span>

<span data-ttu-id="a2704-529">Per impostare questo valore, usare la configurazione o chiamare `UseSetting`:</span><span class="sxs-lookup"><span data-stu-id="a2704-529">To set this value, use configuration or call `UseSetting`:</span></span>

```csharp
webBuilder.UseSetting(WebHostDefaults.DetailedErrorsKey, "true");
```

### <a name="hostingstartupassemblies"></a><span data-ttu-id="a2704-530">HostingStartupAssemblies</span><span class="sxs-lookup"><span data-stu-id="a2704-530">HostingStartupAssemblies</span></span>

<span data-ttu-id="a2704-531">Una stringa delimitata da punto e virgola di assembly di avvio dell'hosting da caricare all'avvio.</span><span class="sxs-lookup"><span data-stu-id="a2704-531">A semicolon-delimited string of hosting startup assemblies to load on startup.</span></span> <span data-ttu-id="a2704-532">Sebbene il valore di configurazione predefinito sia una stringa vuota, gli assembly di avvio dell'hosting includono sempre l'assembly dell'app.</span><span class="sxs-lookup"><span data-stu-id="a2704-532">Although the configuration value defaults to an empty string, the hosting startup assemblies always include the app's assembly.</span></span> <span data-ttu-id="a2704-533">Quando vengono specificati, gli assembly di avvio dell'hosting vengono aggiunti all'assembly dell'app per essere caricati quando l'app compila i servizi comuni durante l'avvio.</span><span class="sxs-lookup"><span data-stu-id="a2704-533">When hosting startup assemblies are provided, they're added to the app's assembly for loading when the app builds its common services during startup.</span></span>

<span data-ttu-id="a2704-534">**Chiave**: `hostingStartupAssemblies`</span><span class="sxs-lookup"><span data-stu-id="a2704-534">**Key**: `hostingStartupAssemblies`</span></span>  
<span data-ttu-id="a2704-535">**Tipo**: `string`</span><span class="sxs-lookup"><span data-stu-id="a2704-535">**Type**: `string`</span></span>  
<span data-ttu-id="a2704-536">**Impostazione predefinita**: stringa vuota</span><span class="sxs-lookup"><span data-stu-id="a2704-536">**Default**: Empty string</span></span>  
<span data-ttu-id="a2704-537">**Variabile di ambiente**: `<PREFIX_>_HOSTINGSTARTUPASSEMBLIES`</span><span class="sxs-lookup"><span data-stu-id="a2704-537">**Environment variable**: `<PREFIX_>_HOSTINGSTARTUPASSEMBLIES`</span></span>

<span data-ttu-id="a2704-538">Per impostare questo valore, usare la configurazione o chiamare `UseSetting`:</span><span class="sxs-lookup"><span data-stu-id="a2704-538">To set this value, use configuration or call `UseSetting`:</span></span>

```csharp
webBuilder.UseSetting(WebHostDefaults.HostingStartupAssembliesKey, "assembly1;assembly2");
```

### <a name="hostingstartupexcludeassemblies"></a><span data-ttu-id="a2704-539">HostingStartupExcludeAssemblies</span><span class="sxs-lookup"><span data-stu-id="a2704-539">HostingStartupExcludeAssemblies</span></span>

<span data-ttu-id="a2704-540">Una stringa delimitata da punto e virgola di assembly di avvio dell'hosting da escludere all'avvio.</span><span class="sxs-lookup"><span data-stu-id="a2704-540">A semicolon-delimited string of hosting startup assemblies to exclude on startup.</span></span>

<span data-ttu-id="a2704-541">**Chiave**: `hostingStartupExcludeAssemblies`</span><span class="sxs-lookup"><span data-stu-id="a2704-541">**Key**: `hostingStartupExcludeAssemblies`</span></span>  
<span data-ttu-id="a2704-542">**Tipo**: `string`</span><span class="sxs-lookup"><span data-stu-id="a2704-542">**Type**: `string`</span></span>  
<span data-ttu-id="a2704-543">**Impostazione predefinita**: stringa vuota</span><span class="sxs-lookup"><span data-stu-id="a2704-543">**Default**: Empty string</span></span>  
<span data-ttu-id="a2704-544">**Variabile di ambiente**: `<PREFIX_>_HOSTINGSTARTUPEXCLUDEASSEMBLIES`</span><span class="sxs-lookup"><span data-stu-id="a2704-544">**Environment variable**: `<PREFIX_>_HOSTINGSTARTUPEXCLUDEASSEMBLIES`</span></span>

<span data-ttu-id="a2704-545">Per impostare questo valore, usare la configurazione o chiamare `UseSetting`:</span><span class="sxs-lookup"><span data-stu-id="a2704-545">To set this value, use configuration or call `UseSetting`:</span></span>

```csharp
webBuilder.UseSetting(WebHostDefaults.HostingStartupExcludeAssembliesKey, "assembly1;assembly2");
```

### <a name="https_port"></a><span data-ttu-id="a2704-546">HTTPS_Port</span><span class="sxs-lookup"><span data-stu-id="a2704-546">HTTPS_Port</span></span>

<span data-ttu-id="a2704-547">Porta di reindirizzamento HTTPS.</span><span class="sxs-lookup"><span data-stu-id="a2704-547">The HTTPS redirect port.</span></span> <span data-ttu-id="a2704-548">Usata per [imporre HTTPS](xref:security/enforcing-ssl).</span><span class="sxs-lookup"><span data-stu-id="a2704-548">Used in [enforcing HTTPS](xref:security/enforcing-ssl).</span></span>

<span data-ttu-id="a2704-549">**Chiave**: `https_port`</span><span class="sxs-lookup"><span data-stu-id="a2704-549">**Key**: `https_port`</span></span>  
<span data-ttu-id="a2704-550">**Tipo**: `string`</span><span class="sxs-lookup"><span data-stu-id="a2704-550">**Type**: `string`</span></span>  
<span data-ttu-id="a2704-551">**Impostazione predefinita**: un valore predefinito non è impostato.</span><span class="sxs-lookup"><span data-stu-id="a2704-551">**Default**: A default value isn't set.</span></span>  
<span data-ttu-id="a2704-552">**Variabile di ambiente**: `<PREFIX_>HTTPS_PORT`</span><span class="sxs-lookup"><span data-stu-id="a2704-552">**Environment variable**: `<PREFIX_>HTTPS_PORT`</span></span>

<span data-ttu-id="a2704-553">Per impostare questo valore, usare la configurazione o chiamare `UseSetting`:</span><span class="sxs-lookup"><span data-stu-id="a2704-553">To set this value, use configuration or call `UseSetting`:</span></span>

```csharp
webBuilder.UseSetting("https_port", "8080");
```

### <a name="preferhostingurls"></a><span data-ttu-id="a2704-554">PreferHostingUrls</span><span class="sxs-lookup"><span data-stu-id="a2704-554">PreferHostingUrls</span></span>

<span data-ttu-id="a2704-555">Indica se l'host deve essere in ascolto sugli URL configurati con `IWebHostBuilder` anziché gli URL configurati con l' `IServer` implementazione di.</span><span class="sxs-lookup"><span data-stu-id="a2704-555">Indicates whether the host should listen on the URLs configured with the `IWebHostBuilder` instead of those URLs configured with the `IServer` implementation.</span></span>

<span data-ttu-id="a2704-556">**Chiave**: `preferHostingUrls`</span><span class="sxs-lookup"><span data-stu-id="a2704-556">**Key**: `preferHostingUrls`</span></span>  
<span data-ttu-id="a2704-557">**Tipo**: `bool` ( `true` o `1` )</span><span class="sxs-lookup"><span data-stu-id="a2704-557">**Type**: `bool` (`true` or `1`)</span></span>  
<span data-ttu-id="a2704-558">**Impostazione predefinita**: `true`</span><span class="sxs-lookup"><span data-stu-id="a2704-558">**Default**: `true`</span></span>  
<span data-ttu-id="a2704-559">**Variabile di ambiente**: `<PREFIX_>_PREFERHOSTINGURLS`</span><span class="sxs-lookup"><span data-stu-id="a2704-559">**Environment variable**: `<PREFIX_>_PREFERHOSTINGURLS`</span></span>

<span data-ttu-id="a2704-560">Per impostare questo valore, usare la variabile di ambiente o chiamare `PreferHostingUrls`:</span><span class="sxs-lookup"><span data-stu-id="a2704-560">To set this value, use the environment variable or call `PreferHostingUrls`:</span></span>

```csharp
webBuilder.PreferHostingUrls(false);
```

### <a name="preventhostingstartup"></a><span data-ttu-id="a2704-561">PreventHostingStartup</span><span class="sxs-lookup"><span data-stu-id="a2704-561">PreventHostingStartup</span></span>

<span data-ttu-id="a2704-562">Impedisce il caricamento automatico degli assembly di avvio dell'hosting, inclusi gli assembly di avvio dell'hosting configurati dall'assembly dell'app.</span><span class="sxs-lookup"><span data-stu-id="a2704-562">Prevents the automatic loading of hosting startup assemblies, including hosting startup assemblies configured by the app's assembly.</span></span> <span data-ttu-id="a2704-563">Per altre informazioni, vedere <xref:fundamentals/configuration/platform-specific-configuration>.</span><span class="sxs-lookup"><span data-stu-id="a2704-563">For more information, see <xref:fundamentals/configuration/platform-specific-configuration>.</span></span>

<span data-ttu-id="a2704-564">**Chiave**: `preventHostingStartup`</span><span class="sxs-lookup"><span data-stu-id="a2704-564">**Key**: `preventHostingStartup`</span></span>  
<span data-ttu-id="a2704-565">**Tipo**: `bool` ( `true` o `1` )</span><span class="sxs-lookup"><span data-stu-id="a2704-565">**Type**: `bool` (`true` or `1`)</span></span>  
<span data-ttu-id="a2704-566">**Impostazione predefinita**: `false`</span><span class="sxs-lookup"><span data-stu-id="a2704-566">**Default**: `false`</span></span>  
<span data-ttu-id="a2704-567">**Variabile di ambiente**: `<PREFIX_>_PREVENTHOSTINGSTARTUP`</span><span class="sxs-lookup"><span data-stu-id="a2704-567">**Environment variable**: `<PREFIX_>_PREVENTHOSTINGSTARTUP`</span></span>

<span data-ttu-id="a2704-568">Per impostare questo valore, usare la variabile di ambiente o chiamare `UseSetting`:</span><span class="sxs-lookup"><span data-stu-id="a2704-568">To set this value, use the environment variable or call `UseSetting` :</span></span>

```csharp
webBuilder.UseSetting(WebHostDefaults.PreventHostingStartupKey, "true");
```

### <a name="startupassembly"></a><span data-ttu-id="a2704-569">StartupAssembly</span><span class="sxs-lookup"><span data-stu-id="a2704-569">StartupAssembly</span></span>

<span data-ttu-id="a2704-570">L'assembly per la ricerca della classe `Startup`.</span><span class="sxs-lookup"><span data-stu-id="a2704-570">The assembly to search for the `Startup` class.</span></span>

<span data-ttu-id="a2704-571">**Chiave**: `startupAssembly`</span><span class="sxs-lookup"><span data-stu-id="a2704-571">**Key**: `startupAssembly`</span></span>  
<span data-ttu-id="a2704-572">**Tipo**: `string`</span><span class="sxs-lookup"><span data-stu-id="a2704-572">**Type**: `string`</span></span>  
<span data-ttu-id="a2704-573">**Impostazione predefinita**: assembly dell'app</span><span class="sxs-lookup"><span data-stu-id="a2704-573">**Default**: The app's assembly</span></span>  
<span data-ttu-id="a2704-574">**Variabile di ambiente**: `<PREFIX_>STARTUPASSEMBLY`</span><span class="sxs-lookup"><span data-stu-id="a2704-574">**Environment variable**: `<PREFIX_>STARTUPASSEMBLY`</span></span>

<span data-ttu-id="a2704-575">Per impostare questo valore, usare la variabile di ambiente o chiamare `UseStartup`.</span><span class="sxs-lookup"><span data-stu-id="a2704-575">To set this value, use the environment variable or call `UseStartup`.</span></span> <span data-ttu-id="a2704-576">`UseStartup` può richiedere un nome di assembly (`string`) o un tipo (`TStartup`).</span><span class="sxs-lookup"><span data-stu-id="a2704-576">`UseStartup` can take an assembly name (`string`) or a type (`TStartup`).</span></span> <span data-ttu-id="a2704-577">Se vengono chiamati più metodi `UseStartup`, l'ultimo metodo ha la precedenza.</span><span class="sxs-lookup"><span data-stu-id="a2704-577">If multiple `UseStartup` methods are called, the last one takes precedence.</span></span>

```csharp
webBuilder.UseStartup("StartupAssemblyName");
```

```csharp
webBuilder.UseStartup<Startup>();
```

### <a name="urls"></a><span data-ttu-id="a2704-578">URL</span><span class="sxs-lookup"><span data-stu-id="a2704-578">URLs</span></span>

<span data-ttu-id="a2704-579">Elenco delimitato da punto e virgola degli indirizzi IP o gli indirizzi host con le porte e protocolli su cui il server deve eseguire l'ascolto per le richieste.</span><span class="sxs-lookup"><span data-stu-id="a2704-579">A semicolon-delimited list of IP addresses or host addresses with ports and protocols that the server should listen on for requests.</span></span> <span data-ttu-id="a2704-580">Ad esempio: `http://localhost:123`.</span><span class="sxs-lookup"><span data-stu-id="a2704-580">For example, `http://localhost:123`.</span></span> <span data-ttu-id="a2704-581">Usare "\*" per indicare che il server deve eseguire l'ascolto per le richieste su tutti gli indirizzi IP o nomi host usando la porta e il protocollo specificati (ad esempio, `http://*:5000`).</span><span class="sxs-lookup"><span data-stu-id="a2704-581">Use "\*" to indicate that the server should listen for requests on any IP address or hostname using the specified port and protocol (for example, `http://*:5000`).</span></span> <span data-ttu-id="a2704-582">Il protocollo (`http://` o `https://`) deve essere incluso con ogni URL.</span><span class="sxs-lookup"><span data-stu-id="a2704-582">The protocol (`http://` or `https://`) must be included with each URL.</span></span> <span data-ttu-id="a2704-583">I formati supportati variano a seconda del server.</span><span class="sxs-lookup"><span data-stu-id="a2704-583">Supported formats vary among servers.</span></span>

<span data-ttu-id="a2704-584">**Chiave**: `urls`</span><span class="sxs-lookup"><span data-stu-id="a2704-584">**Key**: `urls`</span></span>  
<span data-ttu-id="a2704-585">**Tipo**: `string`</span><span class="sxs-lookup"><span data-stu-id="a2704-585">**Type**: `string`</span></span>  
<span data-ttu-id="a2704-586">**Impostazione predefinita**: `http://localhost:5000` e `https://localhost:5001`</span><span class="sxs-lookup"><span data-stu-id="a2704-586">**Default**: `http://localhost:5000` and `https://localhost:5001`</span></span>  
<span data-ttu-id="a2704-587">**Variabile di ambiente**: `<PREFIX_>URLS`</span><span class="sxs-lookup"><span data-stu-id="a2704-587">**Environment variable**: `<PREFIX_>URLS`</span></span>

<span data-ttu-id="a2704-588">Per impostare questo valore, usare la variabile di ambiente o chiamare `UseUrls`:</span><span class="sxs-lookup"><span data-stu-id="a2704-588">To set this value, use the environment variable or call `UseUrls`:</span></span>

```csharp
webBuilder.UseUrls("http://*:5000;http://localhost:5001;https://hostname:5002");
```

<span data-ttu-id="a2704-589">Kestrel ha una propria API di configurazione degli endpoint.</span><span class="sxs-lookup"><span data-stu-id="a2704-589">Kestrel has its own endpoint configuration API.</span></span> <span data-ttu-id="a2704-590">Per altre informazioni, vedere <xref:fundamentals/servers/kestrel#endpoint-configuration>.</span><span class="sxs-lookup"><span data-stu-id="a2704-590">For more information, see <xref:fundamentals/servers/kestrel#endpoint-configuration>.</span></span>

### <a name="webroot"></a><span data-ttu-id="a2704-591">WebRoot</span><span class="sxs-lookup"><span data-stu-id="a2704-591">WebRoot</span></span>

<span data-ttu-id="a2704-592">La proprietà [IWebHostEnvironment. WebRootPath](xref:Microsoft.AspNetCore.Hosting.IWebHostEnvironment.WebRootPath) determina il percorso relativo degli asset statici dell'app.</span><span class="sxs-lookup"><span data-stu-id="a2704-592">The [IWebHostEnvironment.WebRootPath](xref:Microsoft.AspNetCore.Hosting.IWebHostEnvironment.WebRootPath) property determines the relative path to the app's static assets.</span></span> <span data-ttu-id="a2704-593">Se il percorso non esiste, viene usato un provider di file no-op.</span><span class="sxs-lookup"><span data-stu-id="a2704-593">If the path doesn't exist, a no-op file provider is used.</span></span>  

<span data-ttu-id="a2704-594">**Chiave**: `webroot`</span><span class="sxs-lookup"><span data-stu-id="a2704-594">**Key**: `webroot`</span></span>  
<span data-ttu-id="a2704-595">**Tipo**: `string`</span><span class="sxs-lookup"><span data-stu-id="a2704-595">**Type**: `string`</span></span>  
<span data-ttu-id="a2704-596">**Impostazione predefinita**: il valore predefinito è `wwwroot` .</span><span class="sxs-lookup"><span data-stu-id="a2704-596">**Default**: The default is `wwwroot`.</span></span> <span data-ttu-id="a2704-597">Il percorso di *{radice del contenuto}/wwwroot* deve esistere.</span><span class="sxs-lookup"><span data-stu-id="a2704-597">The path to *{content root}/wwwroot* must exist.</span></span>  
<span data-ttu-id="a2704-598">**Variabile di ambiente**: `<PREFIX_>WEBROOT`</span><span class="sxs-lookup"><span data-stu-id="a2704-598">**Environment variable**: `<PREFIX_>WEBROOT`</span></span>

<span data-ttu-id="a2704-599">Per impostare questo valore, usare la variabile di ambiente o chiamare `UseWebRoot` su `IWebHostBuilder`:</span><span class="sxs-lookup"><span data-stu-id="a2704-599">To set this value, use the environment variable or call `UseWebRoot` on `IWebHostBuilder`:</span></span>

```csharp
webBuilder.UseWebRoot("public");
```

<span data-ttu-id="a2704-600">Per altre informazioni, vedere:</span><span class="sxs-lookup"><span data-stu-id="a2704-600">For more information, see:</span></span>

* [<span data-ttu-id="a2704-601">Nozioni fondamentali: radice Web</span><span class="sxs-lookup"><span data-stu-id="a2704-601">Fundamentals: Web root</span></span>](xref:fundamentals/index#web-root)
* [<span data-ttu-id="a2704-602">ContentRoot</span><span class="sxs-lookup"><span data-stu-id="a2704-602">ContentRoot</span></span>](#contentroot)

## <a name="manage-the-host-lifetime"></a><span data-ttu-id="a2704-603">Gestire la durata dell'host</span><span class="sxs-lookup"><span data-stu-id="a2704-603">Manage the host lifetime</span></span>

<span data-ttu-id="a2704-604">Chiamare metodi sull'implementazione <xref:Microsoft.Extensions.Hosting.IHost> incorporata per avviare e arrestare l'app.</span><span class="sxs-lookup"><span data-stu-id="a2704-604">Call methods on the built <xref:Microsoft.Extensions.Hosting.IHost> implementation to start and stop the app.</span></span> <span data-ttu-id="a2704-605">Questi metodi influiscono su tutte le implementazioni <xref:Microsoft.Extensions.Hosting.IHostedService> che vengono registrate nel contenitore dei servizi.</span><span class="sxs-lookup"><span data-stu-id="a2704-605">These methods affect all  <xref:Microsoft.Extensions.Hosting.IHostedService> implementations that are registered in the service container.</span></span>

### <a name="run"></a><span data-ttu-id="a2704-606">Esegui</span><span class="sxs-lookup"><span data-stu-id="a2704-606">Run</span></span>

<span data-ttu-id="a2704-607"><xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.Run*> esegue l'app e blocca il thread di chiamata fino all'arresto dell'host.</span><span class="sxs-lookup"><span data-stu-id="a2704-607"><xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.Run*> runs the app and blocks the calling thread until the host is shut down.</span></span>

### <a name="runasync"></a><span data-ttu-id="a2704-608">RunAsync</span><span class="sxs-lookup"><span data-stu-id="a2704-608">RunAsync</span></span>

<span data-ttu-id="a2704-609"><xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.RunAsync*> esegue l'app e restituisce un elemento <xref:System.Threading.Tasks.Task> che viene completato quando viene attivato il token di annullamento o l'arresto.</span><span class="sxs-lookup"><span data-stu-id="a2704-609"><xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.RunAsync*> runs the app and returns a <xref:System.Threading.Tasks.Task> that completes when the cancellation token or shutdown is triggered.</span></span>

### <a name="runconsoleasync"></a><span data-ttu-id="a2704-610">RunConsoleAsync</span><span class="sxs-lookup"><span data-stu-id="a2704-610">RunConsoleAsync</span></span>

<span data-ttu-id="a2704-611"><xref:Microsoft.Extensions.Hosting.HostingHostBuilderExtensions.RunConsoleAsync*>Abilita il supporto della console, compila e avvia l'host e attende la <kbd></kbd> + chiusura di CTRL<kbd>C</kbd>/SIGINT o SIGTERM.</span><span class="sxs-lookup"><span data-stu-id="a2704-611"><xref:Microsoft.Extensions.Hosting.HostingHostBuilderExtensions.RunConsoleAsync*> enables console support, builds and starts the host, and waits for <kbd>Ctrl</kbd>+<kbd>C</kbd>/SIGINT or SIGTERM to shut down.</span></span>

### <a name="start"></a><span data-ttu-id="a2704-612">Inizia</span><span class="sxs-lookup"><span data-stu-id="a2704-612">Start</span></span>

<span data-ttu-id="a2704-613"><xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.Start*> avvia l'host in modo sincrono.</span><span class="sxs-lookup"><span data-stu-id="a2704-613"><xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.Start*> starts the host synchronously.</span></span>

### <a name="startasync"></a><span data-ttu-id="a2704-614">StartAsync</span><span class="sxs-lookup"><span data-stu-id="a2704-614">StartAsync</span></span>

<span data-ttu-id="a2704-615"><xref:Microsoft.Extensions.Hosting.IHost.StartAsync*> avvia l'host e restituisce un elemento <xref:System.Threading.Tasks.Task> che viene completato quando viene attivato il token di annullamento o l'arresto.</span><span class="sxs-lookup"><span data-stu-id="a2704-615"><xref:Microsoft.Extensions.Hosting.IHost.StartAsync*> starts the host and returns a <xref:System.Threading.Tasks.Task> that completes when the cancellation token or shutdown is triggered.</span></span> 

<span data-ttu-id="a2704-616"><xref:Microsoft.Extensions.Hosting.IHostLifetime.WaitForStartAsync*> viene chiamato all'avvio di `StartAsync`, che attende il completamento prima di continuare.</span><span class="sxs-lookup"><span data-stu-id="a2704-616"><xref:Microsoft.Extensions.Hosting.IHostLifetime.WaitForStartAsync*> is called at the start of `StartAsync`, which waits until it's complete before continuing.</span></span> <span data-ttu-id="a2704-617">Questo è utile per ritardare l'avvio fino al segnale trasmesso da un evento esterno.</span><span class="sxs-lookup"><span data-stu-id="a2704-617">This can be used to delay startup until signaled by an external event.</span></span>

### <a name="stopasync"></a><span data-ttu-id="a2704-618">StopAsync</span><span class="sxs-lookup"><span data-stu-id="a2704-618">StopAsync</span></span>

<span data-ttu-id="a2704-619"><xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.StopAsync*> prova ad arrestare l'host entro il timeout specificato.</span><span class="sxs-lookup"><span data-stu-id="a2704-619"><xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.StopAsync*> attempts to stop the host within the provided timeout.</span></span>

### <a name="waitforshutdown"></a><span data-ttu-id="a2704-620">WaitForShutdown</span><span class="sxs-lookup"><span data-stu-id="a2704-620">WaitForShutdown</span></span>

<span data-ttu-id="a2704-621"><xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.WaitForShutdown*>blocca il thread chiamante finché l'arresto non viene attivato da IHostLifetime, ad esempio tramite <kbd>CTRL</kbd> + <kbd>C</kbd>/SIGINT o SIGTERM.</span><span class="sxs-lookup"><span data-stu-id="a2704-621"><xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.WaitForShutdown*> blocks the calling thread until shutdown is triggered by the IHostLifetime, such as via <kbd>Ctrl</kbd>+<kbd>C</kbd>/SIGINT or SIGTERM.</span></span>

### <a name="waitforshutdownasync"></a><span data-ttu-id="a2704-622">WaitForShutdownAsync</span><span class="sxs-lookup"><span data-stu-id="a2704-622">WaitForShutdownAsync</span></span>

<span data-ttu-id="a2704-623"><xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.WaitForShutdownAsync*> restituisce un elemento <xref:System.Threading.Tasks.Task> che viene completato quando viene attivato l'arresto tramite il token specificato, quindi chiama <xref:Microsoft.Extensions.Hosting.IHost.StopAsync*>.</span><span class="sxs-lookup"><span data-stu-id="a2704-623"><xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.WaitForShutdownAsync*> returns a <xref:System.Threading.Tasks.Task> that completes when shutdown is triggered via the given token and calls <xref:Microsoft.Extensions.Hosting.IHost.StopAsync*>.</span></span>

### <a name="external-control"></a><span data-ttu-id="a2704-624">Controllo esterno</span><span class="sxs-lookup"><span data-stu-id="a2704-624">External control</span></span>

<span data-ttu-id="a2704-625">Il controllo diretto della durata dell'host può essere ottenuto usando metodi che possono essere chiamati dall'esterno:</span><span class="sxs-lookup"><span data-stu-id="a2704-625">Direct control of the host lifetime can be achieved using methods that can be called externally:</span></span>

```csharp
public class Program
{
    private IHost _host;

    public Program()
    {
        _host = new HostBuilder()
            .Build();
    }

    public async Task StartAsync()
    {
        _host.StartAsync();
    }

    public async Task StopAsync()
    {
        using (_host)
        {
            await _host.StopAsync(TimeSpan.FromSeconds(5));
        }
    }
}
```

::: moniker-end

::: moniker range="< aspnetcore-3.0"

<span data-ttu-id="a2704-626">Le app ASP.NET Core configurano e avviano un host.</span><span class="sxs-lookup"><span data-stu-id="a2704-626">ASP.NET Core apps configure and launch a host.</span></span> <span data-ttu-id="a2704-627">L'host è responsabile della gestione dell'avvio e della durata delle app.</span><span class="sxs-lookup"><span data-stu-id="a2704-627">The host is responsible for app startup and lifetime management.</span></span>

<span data-ttu-id="a2704-628">Questo articolo illustra l'host generico di ASP.NET Core (<xref:Microsoft.Extensions.Hosting.HostBuilder>), che si usa per le app che non elaborano le richieste HTTP.</span><span class="sxs-lookup"><span data-stu-id="a2704-628">This article covers the ASP.NET Core Generic Host (<xref:Microsoft.Extensions.Hosting.HostBuilder>), which is used for apps that don't process HTTP requests.</span></span>

<span data-ttu-id="a2704-629">Lo scopo dell'host generico è separare la pipeline HTTP dall'API dell'host Web, per abilitare una gamma più ampia di scenari host.</span><span class="sxs-lookup"><span data-stu-id="a2704-629">The purpose of Generic Host is to decouple the HTTP pipeline from the Web Host API to enable a wider array of host scenarios.</span></span> <span data-ttu-id="a2704-630">La messaggistica, le attività in background e altri carichi di lavoro non HTTP basati sull'host generico traggono vantaggio dalle funzionalità a montaggio incrociato come la configurazione, l'inserimento di dipendenze (DI) e la registrazione.</span><span class="sxs-lookup"><span data-stu-id="a2704-630">Messaging, background tasks, and other non-HTTP workloads based on Generic Host benefit from cross-cutting capabilities, such as configuration, dependency injection (DI), and logging.</span></span>

<span data-ttu-id="a2704-631">L'host generico è una nuova funzionalità introdotta in ASP.NET Core 2.1 e non è adatto agli scenari di hosting Web.</span><span class="sxs-lookup"><span data-stu-id="a2704-631">Generic Host is new in ASP.NET Core 2.1 and isn't suitable for web hosting scenarios.</span></span> <span data-ttu-id="a2704-632">Per gli scenari di hosting Web usare l'[host Web](xref:fundamentals/host/web-host).</span><span class="sxs-lookup"><span data-stu-id="a2704-632">For web hosting scenarios, use the [Web Host](xref:fundamentals/host/web-host).</span></span> <span data-ttu-id="a2704-633">L'host generico sostituirà l'host Web in una versione futura, funzionando come API host principale negli scenari sia HTTP che non HTTP.</span><span class="sxs-lookup"><span data-stu-id="a2704-633">Generic Host will replace Web Host in a future release and act as the primary host API in both HTTP and non-HTTP scenarios.</span></span>

<span data-ttu-id="a2704-634">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/host/generic-host/samples/) ([procedura per il download](xref:index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="a2704-634">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/host/generic-host/samples/) ([how to download](xref:index#how-to-download-a-sample))</span></span>

<span data-ttu-id="a2704-635">Quando si esegue l'app di esempio in [Visual Studio Code](https://code.visualstudio.com/), usare un *terminale esterno o integrato*.</span><span class="sxs-lookup"><span data-stu-id="a2704-635">When running the sample app in [Visual Studio Code](https://code.visualstudio.com/), use an *external or integrated terminal*.</span></span> <span data-ttu-id="a2704-636">Non eseguire l'esempio in un ambiente `internalConsole`.</span><span class="sxs-lookup"><span data-stu-id="a2704-636">Don't run the sample in an `internalConsole`.</span></span>

<span data-ttu-id="a2704-637">Per impostare la console in Visual Studio Code:</span><span class="sxs-lookup"><span data-stu-id="a2704-637">To set the console in Visual Studio Code:</span></span>

1. <span data-ttu-id="a2704-638">Aprire il file *.vscode/launch.json*.</span><span class="sxs-lookup"><span data-stu-id="a2704-638">Open the *.vscode/launch.json* file.</span></span>
1. <span data-ttu-id="a2704-639">Nella configurazione **.NET Core Launch (console)** trovare la voce **console**.</span><span class="sxs-lookup"><span data-stu-id="a2704-639">In the **.NET Core Launch (console)** configuration, locate the **console** entry.</span></span> <span data-ttu-id="a2704-640">Impostare il valore su `externalTerminal` o `integratedTerminal`.</span><span class="sxs-lookup"><span data-stu-id="a2704-640">Set the value to either `externalTerminal` or `integratedTerminal`.</span></span>

## <a name="introduction"></a><span data-ttu-id="a2704-641">Introduzione</span><span class="sxs-lookup"><span data-stu-id="a2704-641">Introduction</span></span>

<span data-ttu-id="a2704-642">La libreria host generico è disponibile nello spazio dei nomi <xref:Microsoft.Extensions.Hosting> e viene specificata dal pacchetto [Microsoft.Extensions.Hosting](https://www.nuget.org/packages/Microsoft.Extensions.Hosting/).</span><span class="sxs-lookup"><span data-stu-id="a2704-642">The Generic Host library is available in the <xref:Microsoft.Extensions.Hosting> namespace and provided by the [Microsoft.Extensions.Hosting](https://www.nuget.org/packages/Microsoft.Extensions.Hosting/) package.</span></span> <span data-ttu-id="a2704-643">Il pacchetto `Microsoft.Extensions.Hosting` è incluso nel [metapacchetto Microsoft.AspNetCore.App](xref:fundamentals/metapackage-app) (ASP.NET Core 2.1 o versioni successive).</span><span class="sxs-lookup"><span data-stu-id="a2704-643">The `Microsoft.Extensions.Hosting` package is included in the [Microsoft.AspNetCore.App metapackage](xref:fundamentals/metapackage-app) (ASP.NET Core 2.1 or later).</span></span>

<span data-ttu-id="a2704-644"><xref:Microsoft.Extensions.Hosting.IHostedService> è il punto di ingresso per l'esecuzione del codice.</span><span class="sxs-lookup"><span data-stu-id="a2704-644"><xref:Microsoft.Extensions.Hosting.IHostedService> is the entry point to code execution.</span></span> <span data-ttu-id="a2704-645">Ogni implementazione <xref:Microsoft.Extensions.Hosting.IHostedService> viene eseguita nell'ordine di [registrazione del servizio in ConfigureServices](#configureservices).</span><span class="sxs-lookup"><span data-stu-id="a2704-645">Each <xref:Microsoft.Extensions.Hosting.IHostedService> implementation is executed in the order of [service registration in ConfigureServices](#configureservices).</span></span> <span data-ttu-id="a2704-646"><xref:Microsoft.Extensions.Hosting.IHostedService.StartAsync*> viene chiamato su ogni <xref:Microsoft.Extensions.Hosting.IHostedService> quando viene avviato l'host e <xref:Microsoft.Extensions.Hosting.IHostedService.StopAsync*> viene chiamato in ordine di registrazione inverso quando l'host viene chiuso normalmente.</span><span class="sxs-lookup"><span data-stu-id="a2704-646"><xref:Microsoft.Extensions.Hosting.IHostedService.StartAsync*> is called on each <xref:Microsoft.Extensions.Hosting.IHostedService> when the host starts, and <xref:Microsoft.Extensions.Hosting.IHostedService.StopAsync*> is called in reverse registration order when the host shuts down gracefully.</span></span>

## <a name="set-up-a-host"></a><span data-ttu-id="a2704-647">Configurare un host</span><span class="sxs-lookup"><span data-stu-id="a2704-647">Set up a host</span></span>

<span data-ttu-id="a2704-648"><xref:Microsoft.Extensions.Hosting.IHostBuilder> è il componente principale che le librerie e le app usano per inizializzare, compilare ed eseguire l'host:</span><span class="sxs-lookup"><span data-stu-id="a2704-648"><xref:Microsoft.Extensions.Hosting.IHostBuilder> is the main component that libraries and apps use to initialize, build, and run the host:</span></span>

[!code-csharp[](generic-host/samples-snapshot/2.x/GenericHostSample/Program.cs?name=snippet_HostBuilder)]

## <a name="options"></a><span data-ttu-id="a2704-649">Opzioni</span><span class="sxs-lookup"><span data-stu-id="a2704-649">Options</span></span>

<span data-ttu-id="a2704-650"><xref:Microsoft.Extensions.Hosting.HostOptions> configura le opzioni per <xref:Microsoft.Extensions.Hosting.IHost>.</span><span class="sxs-lookup"><span data-stu-id="a2704-650"><xref:Microsoft.Extensions.Hosting.HostOptions> configure options for the <xref:Microsoft.Extensions.Hosting.IHost>.</span></span>

### <a name="shutdown-timeout"></a><span data-ttu-id="a2704-651">Timeout di arresto</span><span class="sxs-lookup"><span data-stu-id="a2704-651">Shutdown timeout</span></span>

<span data-ttu-id="a2704-652"><xref:Microsoft.Extensions.Hosting.HostOptions.ShutdownTimeout*> imposta il timeout per <xref:Microsoft.Extensions.Hosting.IHost.StopAsync*>.</span><span class="sxs-lookup"><span data-stu-id="a2704-652"><xref:Microsoft.Extensions.Hosting.HostOptions.ShutdownTimeout*> sets the timeout for <xref:Microsoft.Extensions.Hosting.IHost.StopAsync*>.</span></span> <span data-ttu-id="a2704-653">Il valore predefinito è cinque secondi.</span><span class="sxs-lookup"><span data-stu-id="a2704-653">The default value is five seconds.</span></span>

<span data-ttu-id="a2704-654">La seguente configurazione di opzione in `Program.Main` aumenta il timeout predefinito di cinque secondi di chiusura a 20 secondi:</span><span class="sxs-lookup"><span data-stu-id="a2704-654">The following option configuration in `Program.Main` increases the default five-second shutdown timeout to 20 seconds:</span></span>

```csharp
var host = new HostBuilder()
    .ConfigureServices((hostContext, services) =>
    {
        services.Configure<HostOptions>(option =>
        {
            option.ShutdownTimeout = System.TimeSpan.FromSeconds(20);
        });
    })
    .Build();
```

## <a name="default-services"></a><span data-ttu-id="a2704-655">Servizi predefiniti</span><span class="sxs-lookup"><span data-stu-id="a2704-655">Default services</span></span>

<span data-ttu-id="a2704-656">I servizi seguenti vengono registrati durante l'inizializzazione dell'host:</span><span class="sxs-lookup"><span data-stu-id="a2704-656">The following services are registered during host initialization:</span></span>

* <span data-ttu-id="a2704-657">[Ambiente](xref:fundamentals/environments) ( <xref:Microsoft.Extensions.Hosting.IHostingEnvironment> )</span><span class="sxs-lookup"><span data-stu-id="a2704-657">[Environment](xref:fundamentals/environments) (<xref:Microsoft.Extensions.Hosting.IHostingEnvironment>)</span></span>
* <xref:Microsoft.Extensions.Hosting.HostBuilderContext>
* <span data-ttu-id="a2704-658">[Configurazione](xref:fundamentals/configuration/index) ( <xref:Microsoft.Extensions.Configuration.IConfiguration> )</span><span class="sxs-lookup"><span data-stu-id="a2704-658">[Configuration](xref:fundamentals/configuration/index) (<xref:Microsoft.Extensions.Configuration.IConfiguration>)</span></span>
* <span data-ttu-id="a2704-659"><xref:Microsoft.Extensions.Hosting.IApplicationLifetime> (`Microsoft.Extensions.Hosting.Internal.ApplicationLifetime`)</span><span class="sxs-lookup"><span data-stu-id="a2704-659"><xref:Microsoft.Extensions.Hosting.IApplicationLifetime> (`Microsoft.Extensions.Hosting.Internal.ApplicationLifetime`)</span></span>
* <span data-ttu-id="a2704-660"><xref:Microsoft.Extensions.Hosting.IHostLifetime> (`Microsoft.Extensions.Hosting.Internal.ConsoleLifetime`)</span><span class="sxs-lookup"><span data-stu-id="a2704-660"><xref:Microsoft.Extensions.Hosting.IHostLifetime> (`Microsoft.Extensions.Hosting.Internal.ConsoleLifetime`)</span></span>
* <xref:Microsoft.Extensions.Hosting.IHost>
* <span data-ttu-id="a2704-661">[Opzioni](xref:fundamentals/configuration/options) ( <xref:Microsoft.Extensions.DependencyInjection.OptionsServiceCollectionExtensions.AddOptions*> )</span><span class="sxs-lookup"><span data-stu-id="a2704-661">[Options](xref:fundamentals/configuration/options) (<xref:Microsoft.Extensions.DependencyInjection.OptionsServiceCollectionExtensions.AddOptions*>)</span></span>
* <span data-ttu-id="a2704-662">[Registrazione](xref:fundamentals/logging/index) ( <xref:Microsoft.Extensions.DependencyInjection.LoggingServiceCollectionExtensions.AddLogging*> )</span><span class="sxs-lookup"><span data-stu-id="a2704-662">[Logging](xref:fundamentals/logging/index) (<xref:Microsoft.Extensions.DependencyInjection.LoggingServiceCollectionExtensions.AddLogging*>)</span></span>

## <a name="host-configuration"></a><span data-ttu-id="a2704-663">Configurazione dell'host</span><span class="sxs-lookup"><span data-stu-id="a2704-663">Host configuration</span></span>

<span data-ttu-id="a2704-664">La configurazione dell'host viene creata tramite:</span><span class="sxs-lookup"><span data-stu-id="a2704-664">Host configuration is created by:</span></span>

* <span data-ttu-id="a2704-665">Chiamata ai metodi di estensione su <xref:Microsoft.Extensions.Hosting.IHostBuilder> per impostare la [radice del contenuto](#content-root) e l'[ambiente](#environment).</span><span class="sxs-lookup"><span data-stu-id="a2704-665">Calling extension methods on <xref:Microsoft.Extensions.Hosting.IHostBuilder> to set the [content root](#content-root) and [environment](#environment).</span></span>
* <span data-ttu-id="a2704-666">Lettura della configurazione dai provider di configurazione in <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureHostConfiguration*>.</span><span class="sxs-lookup"><span data-stu-id="a2704-666">Reading configuration from configuration providers in <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureHostConfiguration*>.</span></span>

### <a name="extension-methods"></a><span data-ttu-id="a2704-667">Metodi di estensione</span><span class="sxs-lookup"><span data-stu-id="a2704-667">Extension methods</span></span>

### <a name="application-key-name"></a><span data-ttu-id="a2704-668">Chiave applicazione (nome)</span><span class="sxs-lookup"><span data-stu-id="a2704-668">Application key (name)</span></span>

<span data-ttu-id="a2704-669">La proprietà [IHostingEnvironment.ApplicationName](xref:Microsoft.Extensions.Hosting.IHostingEnvironment.ApplicationName*) viene impostata dalla configurazione dell'host durante la costruzione dell'host.</span><span class="sxs-lookup"><span data-stu-id="a2704-669">The [IHostingEnvironment.ApplicationName](xref:Microsoft.Extensions.Hosting.IHostingEnvironment.ApplicationName*) property is set from host configuration during host construction.</span></span> <span data-ttu-id="a2704-670">Per impostare il valore in modo esplicito, usare [HostDefaults.ApplicationKey](xref:Microsoft.Extensions.Hosting.HostDefaults.ApplicationKey):</span><span class="sxs-lookup"><span data-stu-id="a2704-670">To set the value explicitly, use the [HostDefaults.ApplicationKey](xref:Microsoft.Extensions.Hosting.HostDefaults.ApplicationKey):</span></span>

<span data-ttu-id="a2704-671">**Chiave**: `applicationName`</span><span class="sxs-lookup"><span data-stu-id="a2704-671">**Key**: `applicationName`</span></span>  
<span data-ttu-id="a2704-672">**Tipo**: `string`</span><span class="sxs-lookup"><span data-stu-id="a2704-672">**Type**: `string`</span></span>  
<span data-ttu-id="a2704-673">**Impostazione predefinita**: il nome dell'assembly contenente il punto di ingresso dell'app.</span><span class="sxs-lookup"><span data-stu-id="a2704-673">**Default**: The name of the assembly containing the app's entry point.</span></span>  
<span data-ttu-id="a2704-674">**Imposta utilizzando**: `HostBuilderContext.HostingEnvironment.ApplicationName`</span><span class="sxs-lookup"><span data-stu-id="a2704-674">**Set using**: `HostBuilderContext.HostingEnvironment.ApplicationName`</span></span>  
<span data-ttu-id="a2704-675">**Variabile di ambiente**: `<PREFIX_>APPLICATIONNAME` ( `<PREFIX_>` è [facoltativo e definito dall'utente](#configurehostconfiguration))</span><span class="sxs-lookup"><span data-stu-id="a2704-675">**Environment variable**: `<PREFIX_>APPLICATIONNAME` (`<PREFIX_>` is [optional and user-defined](#configurehostconfiguration))</span></span>

### <a name="content-root"></a><span data-ttu-id="a2704-676">Radice del contenuto</span><span class="sxs-lookup"><span data-stu-id="a2704-676">Content root</span></span>

<span data-ttu-id="a2704-677">Questa impostazione determina la posizione da cui l'host inizia la ricerca dei file di contenuto.</span><span class="sxs-lookup"><span data-stu-id="a2704-677">This setting determines where the host begins searching for content files.</span></span>

<span data-ttu-id="a2704-678">**Chiave**: `contentRoot`</span><span class="sxs-lookup"><span data-stu-id="a2704-678">**Key**: `contentRoot`</span></span>  
<span data-ttu-id="a2704-679">**Tipo**: `string`</span><span class="sxs-lookup"><span data-stu-id="a2704-679">**Type**: `string`</span></span>  
<span data-ttu-id="a2704-680">**Impostazione predefinita**: il valore predefinito corrisponde alla cartella contenente l'assembly dell'app.</span><span class="sxs-lookup"><span data-stu-id="a2704-680">**Default**: Defaults to the folder where the app assembly resides.</span></span>  
<span data-ttu-id="a2704-681">**Imposta utilizzando**: `UseContentRoot`</span><span class="sxs-lookup"><span data-stu-id="a2704-681">**Set using**: `UseContentRoot`</span></span>  
<span data-ttu-id="a2704-682">**Variabile di ambiente**: `<PREFIX_>CONTENTROOT` ( `<PREFIX_>` è [facoltativo e definito dall'utente](#configurehostconfiguration))</span><span class="sxs-lookup"><span data-stu-id="a2704-682">**Environment variable**: `<PREFIX_>CONTENTROOT` (`<PREFIX_>` is [optional and user-defined](#configurehostconfiguration))</span></span>

<span data-ttu-id="a2704-683">Se il percorso non esiste, l'host non verrà avviato.</span><span class="sxs-lookup"><span data-stu-id="a2704-683">If the path doesn't exist, the host fails to start.</span></span>

[!code-csharp[](generic-host/samples-snapshot/2.x/GenericHostSample/Program.cs?name=snippet_UseContentRoot)]

<span data-ttu-id="a2704-684">Per altre informazioni, vedere [nozioni fondamentali: radice del contenuto](xref:fundamentals/index#content-root).</span><span class="sxs-lookup"><span data-stu-id="a2704-684">For more information, see [Fundamentals: Content root](xref:fundamentals/index#content-root).</span></span>

### <a name="environment"></a><span data-ttu-id="a2704-685">Ambiente</span><span class="sxs-lookup"><span data-stu-id="a2704-685">Environment</span></span>

<span data-ttu-id="a2704-686">Imposta l' [ambiente](xref:fundamentals/environments)dell'app.</span><span class="sxs-lookup"><span data-stu-id="a2704-686">Sets the app's [environment](xref:fundamentals/environments).</span></span>

<span data-ttu-id="a2704-687">**Chiave**: `environment`</span><span class="sxs-lookup"><span data-stu-id="a2704-687">**Key**: `environment`</span></span>  
<span data-ttu-id="a2704-688">**Tipo**: `string`</span><span class="sxs-lookup"><span data-stu-id="a2704-688">**Type**: `string`</span></span>  
<span data-ttu-id="a2704-689">**Impostazione predefinita**: `Production`</span><span class="sxs-lookup"><span data-stu-id="a2704-689">**Default**: `Production`</span></span>  
<span data-ttu-id="a2704-690">**Imposta utilizzando**: `UseEnvironment`</span><span class="sxs-lookup"><span data-stu-id="a2704-690">**Set using**: `UseEnvironment`</span></span>  
<span data-ttu-id="a2704-691">**Variabile di ambiente**: `<PREFIX_>ENVIRONMENT` ( `<PREFIX_>` è [facoltativo e definito dall'utente](#configurehostconfiguration))</span><span class="sxs-lookup"><span data-stu-id="a2704-691">**Environment variable**: `<PREFIX_>ENVIRONMENT` (`<PREFIX_>` is [optional and user-defined](#configurehostconfiguration))</span></span>

<span data-ttu-id="a2704-692">L'ambiente può essere impostato su qualsiasi valore.</span><span class="sxs-lookup"><span data-stu-id="a2704-692">The environment can be set to any value.</span></span> <span data-ttu-id="a2704-693">I valori definiti dal framework includono `Development`, `Staging` e `Production`.</span><span class="sxs-lookup"><span data-stu-id="a2704-693">Framework-defined values include `Development`, `Staging`, and `Production`.</span></span> <span data-ttu-id="a2704-694">I valori non fanno distinzione tra maiuscole e minuscole.</span><span class="sxs-lookup"><span data-stu-id="a2704-694">Values aren't case-sensitive.</span></span>

[!code-csharp[](generic-host/samples-snapshot/2.x/GenericHostSample/Program.cs?name=snippet_UseEnvironment)]

### <a name="configurehostconfiguration"></a><span data-ttu-id="a2704-695">ConfigureHostConfiguration</span><span class="sxs-lookup"><span data-stu-id="a2704-695">ConfigureHostConfiguration</span></span>

<span data-ttu-id="a2704-696"><xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureHostConfiguration*> usa un'interfaccia <xref:Microsoft.Extensions.Configuration.IConfigurationBuilder> per creare un'interfaccia <xref:Microsoft.Extensions.Configuration.IConfiguration> per l'host.</span><span class="sxs-lookup"><span data-stu-id="a2704-696"><xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureHostConfiguration*> uses an <xref:Microsoft.Extensions.Configuration.IConfigurationBuilder> to create an <xref:Microsoft.Extensions.Configuration.IConfiguration> for the host.</span></span> <span data-ttu-id="a2704-697">La configurazione dell'host viene usata per inizializzare l'interfaccia <xref:Microsoft.Extensions.Hosting.IHostingEnvironment> da usare nel processo di compilazione dell'app.</span><span class="sxs-lookup"><span data-stu-id="a2704-697">The host configuration is used to initialize the <xref:Microsoft.Extensions.Hosting.IHostingEnvironment> for use in the app's build process.</span></span>

<span data-ttu-id="a2704-698"><xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureHostConfiguration*> può essere chiamato più volte e i risultati si sommano.</span><span class="sxs-lookup"><span data-stu-id="a2704-698"><xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureHostConfiguration*> can be called multiple times with additive results.</span></span> <span data-ttu-id="a2704-699">L'host usa l'ultima opzione che imposta un valore in una determinata chiave.</span><span class="sxs-lookup"><span data-stu-id="a2704-699">The host uses whichever option sets a value last on a given key.</span></span>

<span data-ttu-id="a2704-700">Nessun provider è incluso per impostazione predefinita.</span><span class="sxs-lookup"><span data-stu-id="a2704-700">No providers are included by default.</span></span> <span data-ttu-id="a2704-701">È necessario specificare in modo esplicito tutti i provider di configurazione richiesti dall'app in <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureHostConfiguration*>, inclusi:</span><span class="sxs-lookup"><span data-stu-id="a2704-701">You must explicitly specify whatever configuration providers the app requires in <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureHostConfiguration*>, including:</span></span>

* <span data-ttu-id="a2704-702">Configurazione dei file (ad esempio, da un file *hostsettings.json*).</span><span class="sxs-lookup"><span data-stu-id="a2704-702">File configuration (for example, from a *hostsettings.json* file).</span></span>
* <span data-ttu-id="a2704-703">Configurazione delle variabili di ambiente.</span><span class="sxs-lookup"><span data-stu-id="a2704-703">Environment variable configuration.</span></span>
* <span data-ttu-id="a2704-704">Configurazione degli argomenti della riga di comando.</span><span class="sxs-lookup"><span data-stu-id="a2704-704">Command-line argument configuration.</span></span>
* <span data-ttu-id="a2704-705">Tutti gli altri provider di configurazione necessari.</span><span class="sxs-lookup"><span data-stu-id="a2704-705">Any other required configuration providers.</span></span>

<span data-ttu-id="a2704-706">La configurazione file dell'host viene abilitata specificando il percorso di base dell'app con `SetBasePath` seguito da una chiamata a uno dei [provider di configurazione dei file](xref:fundamentals/configuration/index#file-configuration-provider).</span><span class="sxs-lookup"><span data-stu-id="a2704-706">File configuration of the host is enabled by specifying the app's base path with `SetBasePath` followed by a call to one of the [file configuration providers](xref:fundamentals/configuration/index#file-configuration-provider).</span></span> <span data-ttu-id="a2704-707">L'app di esempio usa un file JSON, *hostsettings.json*, e chiama <xref:Microsoft.Extensions.Configuration.JsonConfigurationExtensions.AddJsonFile*> per utilizzare le impostazioni di configurazione host del file.</span><span class="sxs-lookup"><span data-stu-id="a2704-707">The sample app uses a JSON file, *hostsettings.json*, and calls <xref:Microsoft.Extensions.Configuration.JsonConfigurationExtensions.AddJsonFile*> to consume the file's host configuration settings.</span></span>

<span data-ttu-id="a2704-708">Per aggiungere la [configurazione delle variabili di ambiente](xref:fundamentals/configuration/index#environment-variables-configuration-provider) dell'host, chiamare <xref:Microsoft.Extensions.Configuration.EnvironmentVariablesExtensions.AddEnvironmentVariables*> sul generatore di host.</span><span class="sxs-lookup"><span data-stu-id="a2704-708">To add [environment variable configuration](xref:fundamentals/configuration/index#environment-variables-configuration-provider) of the host, call <xref:Microsoft.Extensions.Configuration.EnvironmentVariablesExtensions.AddEnvironmentVariables*> on the host builder.</span></span> <span data-ttu-id="a2704-709">`AddEnvironmentVariables` accetta un prefisso facoltativo definito dall'utente.</span><span class="sxs-lookup"><span data-stu-id="a2704-709">`AddEnvironmentVariables` accepts an optional user-defined prefix.</span></span> <span data-ttu-id="a2704-710">L'app di esempio usa un prefisso di `PREFIX_`.</span><span class="sxs-lookup"><span data-stu-id="a2704-710">The sample app uses a prefix of `PREFIX_`.</span></span> <span data-ttu-id="a2704-711">Il prefisso viene rimosso quando vengono lette le variabili di ambiente.</span><span class="sxs-lookup"><span data-stu-id="a2704-711">The prefix is removed when the environment variables are read.</span></span> <span data-ttu-id="a2704-712">Quando viene configurato l'host dell'app di esempio, il valore della variabile di ambiente per `PREFIX_ENVIRONMENT` diventa il valore di configurazione dell'host per la chiave `environment`.</span><span class="sxs-lookup"><span data-stu-id="a2704-712">When the sample app's host is configured, the environment variable value for `PREFIX_ENVIRONMENT` becomes the host configuration value for the `environment` key.</span></span>

<span data-ttu-id="a2704-713">Durante lo sviluppo quando si usa [Visual Studio](https://visualstudio.microsoft.com) o si esegue un'app con `dotnet run`, le variabili di ambiente possono essere impostate nel file *Properties/launchSettings.json*.</span><span class="sxs-lookup"><span data-stu-id="a2704-713">During development when using [Visual Studio](https://visualstudio.microsoft.com) or running an app with `dotnet run`, environment variables may be set in the *Properties/launchSettings.json* file.</span></span> <span data-ttu-id="a2704-714">Durante lo sviluppo in [Visual Studio Code](https://code.visualstudio.com/), le variabili di ambiente possono essere impostate nel file *.vscode/launch.json*.</span><span class="sxs-lookup"><span data-stu-id="a2704-714">In [Visual Studio Code](https://code.visualstudio.com/), environment variables may be set in the *.vscode/launch.json* file during development.</span></span> <span data-ttu-id="a2704-715">Per altre informazioni, vedere <xref:fundamentals/environments>.</span><span class="sxs-lookup"><span data-stu-id="a2704-715">For more information, see <xref:fundamentals/environments>.</span></span>

<span data-ttu-id="a2704-716">La [configurazione della riga di comando](xref:fundamentals/configuration/index#command-line-configuration-provider) viene aggiunta chiamando <xref:Microsoft.Extensions.Configuration.CommandLineConfigurationExtensions.AddCommandLine*>.</span><span class="sxs-lookup"><span data-stu-id="a2704-716">[Command-line configuration](xref:fundamentals/configuration/index#command-line-configuration-provider) is added by calling <xref:Microsoft.Extensions.Configuration.CommandLineConfigurationExtensions.AddCommandLine*>.</span></span> <span data-ttu-id="a2704-717">La configurazione della riga di comando viene aggiunta per ultima per consentire agli argomenti della riga di comando di eseguire l'override della configurazione fornita dai provider di configurazione precedenti.</span><span class="sxs-lookup"><span data-stu-id="a2704-717">Command-line configuration is added last to permit command-line arguments to override configuration provided by the earlier configuration providers.</span></span>

<span data-ttu-id="a2704-718">*hostsettings.json*:</span><span class="sxs-lookup"><span data-stu-id="a2704-718">*hostsettings.json*:</span></span>

[!code-json[](generic-host/samples/2.x/GenericHostSample/hostsettings.json)]

<span data-ttu-id="a2704-719">È possibile fornire una configurazione aggiuntiva con le chiavi [applicationName](#application-key-name) e [contentRoot](#content-root).</span><span class="sxs-lookup"><span data-stu-id="a2704-719">Additional configuration can be provided with the [applicationName](#application-key-name) and [contentRoot](#content-root) keys.</span></span>

<span data-ttu-id="a2704-720">Configurazione di esempio di `HostBuilder` che usa <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureHostConfiguration*>:</span><span class="sxs-lookup"><span data-stu-id="a2704-720">Example `HostBuilder` configuration using <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureHostConfiguration*>:</span></span>

[!code-csharp[](generic-host/samples-snapshot/2.x/GenericHostSample/Program.cs?name=snippet_ConfigureHostConfiguration)]

## <a name="configureappconfiguration"></a><span data-ttu-id="a2704-721">ConfigureAppConfiguration</span><span class="sxs-lookup"><span data-stu-id="a2704-721">ConfigureAppConfiguration</span></span>

<span data-ttu-id="a2704-722">La configurazione dell'app viene creata chiamando <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureAppConfiguration*> sull'implementazione <xref:Microsoft.Extensions.Hosting.IHostBuilder>.</span><span class="sxs-lookup"><span data-stu-id="a2704-722">App configuration is created by calling <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureAppConfiguration*> on the <xref:Microsoft.Extensions.Hosting.IHostBuilder> implementation.</span></span> <span data-ttu-id="a2704-723"><xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureAppConfiguration*> usa un'interfaccia <xref:Microsoft.Extensions.Configuration.IConfigurationBuilder> per creare un'interfaccia <xref:Microsoft.Extensions.Configuration.IConfiguration> per l'app.</span><span class="sxs-lookup"><span data-stu-id="a2704-723"><xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureAppConfiguration*> uses an <xref:Microsoft.Extensions.Configuration.IConfigurationBuilder> to create an <xref:Microsoft.Extensions.Configuration.IConfiguration> for the app.</span></span> <span data-ttu-id="a2704-724"><xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureAppConfiguration*> può essere chiamato più volte e i risultati si sommano.</span><span class="sxs-lookup"><span data-stu-id="a2704-724"><xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureAppConfiguration*> can be called multiple times with additive results.</span></span> <span data-ttu-id="a2704-725">L'app usa l'ultima opzione che imposta un valore in una determinata chiave.</span><span class="sxs-lookup"><span data-stu-id="a2704-725">The app uses whichever option sets a value last on a given key.</span></span> <span data-ttu-id="a2704-726">La configurazione creata da <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureAppConfiguration*> è disponibile in [HostBuilderContext.Configuration](xref:Microsoft.Extensions.Hosting.HostBuilderContext.Configuration*) per le operazioni successive e in <xref:Microsoft.Extensions.Hosting.IHost.Services*>.</span><span class="sxs-lookup"><span data-stu-id="a2704-726">The configuration created by <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureAppConfiguration*> is available at [HostBuilderContext.Configuration](xref:Microsoft.Extensions.Hosting.HostBuilderContext.Configuration*) for subsequent operations and in <xref:Microsoft.Extensions.Hosting.IHost.Services*>.</span></span>

<span data-ttu-id="a2704-727">La configurazione dell'app riceve automaticamente la configurazione dell'host fornita da [ConfigureHostConfiguration](#configurehostconfiguration).</span><span class="sxs-lookup"><span data-stu-id="a2704-727">App configuration automatically receives host configuration provided by [ConfigureHostConfiguration](#configurehostconfiguration).</span></span>

<span data-ttu-id="a2704-728">Configurazione app di esempio che usa <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureAppConfiguration*>:</span><span class="sxs-lookup"><span data-stu-id="a2704-728">Example app configuration using <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureAppConfiguration*>:</span></span>

[!code-csharp[](generic-host/samples-snapshot/2.x/GenericHostSample/Program.cs?name=snippet_ConfigureAppConfiguration)]

<span data-ttu-id="a2704-729">*appsettings.json*:</span><span class="sxs-lookup"><span data-stu-id="a2704-729">*appsettings.json*:</span></span>

[!code-json[](generic-host/samples/2.x/GenericHostSample/appsettings.json)]

<span data-ttu-id="a2704-730">*appsettings.Development.json*:</span><span class="sxs-lookup"><span data-stu-id="a2704-730">*appsettings.Development.json*:</span></span>

[!code-json[](generic-host/samples/2.x/GenericHostSample/appsettings.Development.json)]

<span data-ttu-id="a2704-731">*appsettings.Production.json*:</span><span class="sxs-lookup"><span data-stu-id="a2704-731">*appsettings.Production.json*:</span></span>

[!code-json[](generic-host/samples/2.x/GenericHostSample/appsettings.Production.json)]

<span data-ttu-id="a2704-732">Per spostare i file delle impostazioni nella directory di output, specificare i file delle impostazioni come [elementi di progetto MSBuild](/visualstudio/msbuild/common-msbuild-project-items) nel file di progetto.</span><span class="sxs-lookup"><span data-stu-id="a2704-732">To move settings files to the output directory, specify the settings files as [MSBuild project items](/visualstudio/msbuild/common-msbuild-project-items) in the project file.</span></span> <span data-ttu-id="a2704-733">L'app di esempio sposta i file delle impostazioni dell'app JSON e *hostsettings.json* con l'elemento `<Content>` seguente:</span><span class="sxs-lookup"><span data-stu-id="a2704-733">The sample app moves its JSON app settings files and *hostsettings.json* with the following `<Content>` item:</span></span>

```xml
<ItemGroup>
  <Content Include="**\*.json" Exclude="bin\**\*;obj\**\*" 
      CopyToOutputDirectory="PreserveNewest" />
</ItemGroup>
```

> [!NOTE]
> <span data-ttu-id="a2704-734">I metodi di estensione di configurazione, ad esempio <xref:Microsoft.Extensions.Configuration.JsonConfigurationExtensions.AddJsonFile*> e <xref:Microsoft.Extensions.Configuration.EnvironmentVariablesExtensions.AddEnvironmentVariables*>, necessitano di altri pacchetti NuGet, ad esempio [Microsoft.Extensions.Configuration.Json](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.Json) e [ Microsoft.Extensions.Configuration.EnvironmentVariables](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.EnvironmentVariables).</span><span class="sxs-lookup"><span data-stu-id="a2704-734">Configuration extension methods, such as <xref:Microsoft.Extensions.Configuration.JsonConfigurationExtensions.AddJsonFile*> and <xref:Microsoft.Extensions.Configuration.EnvironmentVariablesExtensions.AddEnvironmentVariables*> require additional NuGet packages, such as [Microsoft.Extensions.Configuration.Json](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.Json) and [Microsoft.Extensions.Configuration.EnvironmentVariables](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.EnvironmentVariables).</span></span> <span data-ttu-id="a2704-735">A meno che l'app non usi il [metapacchetto Microsoft.AspNetCore.App](xref:fundamentals/metapackage-app), questi pacchetti devono essere aggiunti al progetto oltre al pacchetto [Microsoft.Extensions.Configuration](https://www.nuget.org/packages/Microsoft.Extensions.Configuration) di base.</span><span class="sxs-lookup"><span data-stu-id="a2704-735">Unless the app uses the [Microsoft.AspNetCore.App metapackage](xref:fundamentals/metapackage-app), these packages must be added to the project in addition to the core [Microsoft.Extensions.Configuration](https://www.nuget.org/packages/Microsoft.Extensions.Configuration) package.</span></span> <span data-ttu-id="a2704-736">Per altre informazioni, vedere <xref:fundamentals/configuration/index>.</span><span class="sxs-lookup"><span data-stu-id="a2704-736">For more information, see <xref:fundamentals/configuration/index>.</span></span>

## <a name="configureservices"></a><span data-ttu-id="a2704-737">ConfigureServices</span><span class="sxs-lookup"><span data-stu-id="a2704-737">ConfigureServices</span></span>

<span data-ttu-id="a2704-738"><xref:Microsoft.Extensions.Hosting.HostingHostBuilderExtensions.ConfigureServices*> aggiunge servizi al contenitore di [inserimento delle dipendenze](xref:fundamentals/dependency-injection) dell'app.</span><span class="sxs-lookup"><span data-stu-id="a2704-738"><xref:Microsoft.Extensions.Hosting.HostingHostBuilderExtensions.ConfigureServices*> adds services to the app's [dependency injection](xref:fundamentals/dependency-injection) container.</span></span> <span data-ttu-id="a2704-739"><xref:Microsoft.Extensions.Hosting.HostingHostBuilderExtensions.ConfigureServices*> può essere chiamato più volte e i risultati si sommano.</span><span class="sxs-lookup"><span data-stu-id="a2704-739"><xref:Microsoft.Extensions.Hosting.HostingHostBuilderExtensions.ConfigureServices*> can be called multiple times with additive results.</span></span>

<span data-ttu-id="a2704-740">Un servizio ospitato è una classe con logica di attività in background che implementa l'interfaccia <xref:Microsoft.Extensions.Hosting.IHostedService>.</span><span class="sxs-lookup"><span data-stu-id="a2704-740">A hosted service is a class with background task logic that implements the <xref:Microsoft.Extensions.Hosting.IHostedService> interface.</span></span> <span data-ttu-id="a2704-741">Per altre informazioni, vedere <xref:fundamentals/host/hosted-services>.</span><span class="sxs-lookup"><span data-stu-id="a2704-741">For more information, see <xref:fundamentals/host/hosted-services>.</span></span>

<span data-ttu-id="a2704-742">L'[app di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/host/generic-host/samples/) usa il metodo di estensione `AddHostedService` per aggiungere all'app un servizio per gli eventi di durata (`LifetimeEventsHostedService`) e un'attività in background con timeout (`TimedHostedService`):</span><span class="sxs-lookup"><span data-stu-id="a2704-742">The [sample app](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/host/generic-host/samples/) uses the `AddHostedService` extension method to add a service for lifetime events, `LifetimeEventsHostedService`, and a timed background task, `TimedHostedService`, to the app:</span></span>

[!code-csharp[](generic-host/samples-snapshot/2.x/GenericHostSample/Program.cs?name=snippet_ConfigureServices)]

## <a name="configurelogging"></a><span data-ttu-id="a2704-743">ConfigureLogging</span><span class="sxs-lookup"><span data-stu-id="a2704-743">ConfigureLogging</span></span>

<span data-ttu-id="a2704-744"><xref:Microsoft.Extensions.Hosting.HostingHostBuilderExtensions.ConfigureLogging*> aggiunge un delegato per configurare l'interfaccia <xref:Microsoft.Extensions.Logging.ILoggingBuilder> fornita.</span><span class="sxs-lookup"><span data-stu-id="a2704-744"><xref:Microsoft.Extensions.Hosting.HostingHostBuilderExtensions.ConfigureLogging*> adds a delegate for configuring the provided <xref:Microsoft.Extensions.Logging.ILoggingBuilder>.</span></span> <span data-ttu-id="a2704-745"><xref:Microsoft.Extensions.Hosting.HostingHostBuilderExtensions.ConfigureLogging*> può essere chiamato più volte e i risultati si sommano.</span><span class="sxs-lookup"><span data-stu-id="a2704-745"><xref:Microsoft.Extensions.Hosting.HostingHostBuilderExtensions.ConfigureLogging*> may be called multiple times with additive results.</span></span>

[!code-csharp[](generic-host/samples-snapshot/2.x/GenericHostSample/Program.cs?name=snippet_ConfigureLogging)]

### <a name="useconsolelifetime"></a><span data-ttu-id="a2704-746">UseConsoleLifetime</span><span class="sxs-lookup"><span data-stu-id="a2704-746">UseConsoleLifetime</span></span>

<span data-ttu-id="a2704-747"><xref:Microsoft.Extensions.Hosting.HostingHostBuilderExtensions.UseConsoleLifetime*>Ascolta la <kbd>combinazione di tasti CTRL</kbd> + <kbd>C</kbd>/SIGINT o SIGTERM e chiama <xref:Microsoft.Extensions.Hosting.IApplicationLifetime.StopApplication*> per avviare il processo di arresto.</span><span class="sxs-lookup"><span data-stu-id="a2704-747"><xref:Microsoft.Extensions.Hosting.HostingHostBuilderExtensions.UseConsoleLifetime*> listens for <kbd>Ctrl</kbd>+<kbd>C</kbd>/SIGINT or SIGTERM and calls <xref:Microsoft.Extensions.Hosting.IApplicationLifetime.StopApplication*> to start the shutdown process.</span></span> <span data-ttu-id="a2704-748"><xref:Microsoft.Extensions.Hosting.HostingHostBuilderExtensions.UseConsoleLifetime*> Sblocca le estensioni, ad esempio [RunAsync](#runasync) e [WaitForShutdownAsync](#waitforshutdownasync).</span><span class="sxs-lookup"><span data-stu-id="a2704-748"><xref:Microsoft.Extensions.Hosting.HostingHostBuilderExtensions.UseConsoleLifetime*> unblocks extensions such as [RunAsync](#runasync) and [WaitForShutdownAsync](#waitforshutdownasync).</span></span> <span data-ttu-id="a2704-749">`Microsoft.Extensions.Hosting.Internal.ConsoleLifetime` è preregistrata come implementazione di durata predefinita.</span><span class="sxs-lookup"><span data-stu-id="a2704-749">`Microsoft.Extensions.Hosting.Internal.ConsoleLifetime` is pre-registered as the default lifetime implementation.</span></span> <span data-ttu-id="a2704-750">Viene usata l'ultima durata registrata.</span><span class="sxs-lookup"><span data-stu-id="a2704-750">The last lifetime registered is used.</span></span>

[!code-csharp[](generic-host/samples-snapshot/2.x/GenericHostSample/Program.cs?name=snippet_UseConsoleLifetime)]

## <a name="container-configuration"></a><span data-ttu-id="a2704-751">Configurazione contenitore</span><span class="sxs-lookup"><span data-stu-id="a2704-751">Container configuration</span></span>

<span data-ttu-id="a2704-752">Per supportare l'inserimento di altri contenitori, l'host può accettare un elemento <xref:Microsoft.Extensions.DependencyInjection.IServiceProviderFactory%601>.</span><span class="sxs-lookup"><span data-stu-id="a2704-752">To support plugging in other containers, the host can accept an <xref:Microsoft.Extensions.DependencyInjection.IServiceProviderFactory%601>.</span></span> <span data-ttu-id="a2704-753">La disponibilità di una factory non fa parte della registrazione del contenitore DI, ma è una funzionalità intrinseca dell'host usata per creare il contenitore DI concreto.</span><span class="sxs-lookup"><span data-stu-id="a2704-753">Providing a factory isn't part of the DI container registration but is instead a host intrinsic used to create the concrete DI container.</span></span> <span data-ttu-id="a2704-754">[UseServiceProviderFactory(IServiceProviderFactory&lt;TContainerBuilder&gt;)](xref:Microsoft.Extensions.Hosting.HostBuilder.UseServiceProviderFactory*) esegue l'override della factory predefinita usata per la creazione del provider di servizi dell'app.</span><span class="sxs-lookup"><span data-stu-id="a2704-754">[UseServiceProviderFactory(IServiceProviderFactory&lt;TContainerBuilder&gt;)](xref:Microsoft.Extensions.Hosting.HostBuilder.UseServiceProviderFactory*) overrides the default factory used to create the app's service provider.</span></span>

<span data-ttu-id="a2704-755">La configurazione del contenitore personalizzato è gestita dal metodo <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureContainer*>.</span><span class="sxs-lookup"><span data-stu-id="a2704-755">Custom container configuration is managed by the <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureContainer*> method.</span></span> <span data-ttu-id="a2704-756"><xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureContainer*> offre un'esperienza fortemente tipizzata per la configurazione del contenitore sulla base dell'API host sottostante.</span><span class="sxs-lookup"><span data-stu-id="a2704-756"><xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureContainer*> provides a strongly-typed experience for configuring the container on top of the underlying host API.</span></span> <span data-ttu-id="a2704-757"><xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureContainer*> può essere chiamato più volte e i risultati si sommano.</span><span class="sxs-lookup"><span data-stu-id="a2704-757"><xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureContainer*> can be called multiple times with additive results.</span></span>

<span data-ttu-id="a2704-758">Creare un contenitore di servizi per l'app:</span><span class="sxs-lookup"><span data-stu-id="a2704-758">Create a service container for the app:</span></span>

[!code-csharp[](generic-host/samples-snapshot/2.x/GenericHostSample/ServiceContainer.cs)]

<span data-ttu-id="a2704-759">Specificare una factory per il contenitore di servizi:</span><span class="sxs-lookup"><span data-stu-id="a2704-759">Provide a service container factory:</span></span>

[!code-csharp[](generic-host/samples-snapshot/2.x/GenericHostSample/ServiceContainerFactory.cs)]

<span data-ttu-id="a2704-760">Usare la factory e configurare il contenitore di servizi personalizzato per l'app:</span><span class="sxs-lookup"><span data-stu-id="a2704-760">Use the factory and configure the custom service container for the app:</span></span>

[!code-csharp[](generic-host/samples-snapshot/2.x/GenericHostSample/Program.cs?name=snippet_ContainerConfiguration)]

## <a name="extensibility"></a><span data-ttu-id="a2704-761">Estendibilità</span><span class="sxs-lookup"><span data-stu-id="a2704-761">Extensibility</span></span>

<span data-ttu-id="a2704-762">L'estensibilità host viene eseguita con i metodi di estensione su <xref:Microsoft.Extensions.Hosting.IHostBuilder>.</span><span class="sxs-lookup"><span data-stu-id="a2704-762">Host extensibility is performed with extension methods on <xref:Microsoft.Extensions.Hosting.IHostBuilder>.</span></span> <span data-ttu-id="a2704-763">L'esempio seguente illustra come un metodo di estensione estende un'implementazione <xref:Microsoft.Extensions.Hosting.IHostBuilder> con l'esempio [TimedHostedService](xref:fundamentals/host/hosted-services#timed-background-tasks) dimostrato in <xref:fundamentals/host/hosted-services>.</span><span class="sxs-lookup"><span data-stu-id="a2704-763">The following example shows how an extension method extends an <xref:Microsoft.Extensions.Hosting.IHostBuilder> implementation with the [TimedHostedService](xref:fundamentals/host/hosted-services#timed-background-tasks) example demonstrated in <xref:fundamentals/host/hosted-services>.</span></span>

```csharp
var host = new HostBuilder()
    .UseHostedService<TimedHostedService>()
    .Build();

await host.StartAsync();
```

<span data-ttu-id="a2704-764">Un'app stabilisce il metodo di estensione `UseHostedService` per registrare il servizio ospitato passato in `T`:</span><span class="sxs-lookup"><span data-stu-id="a2704-764">An app establishes the `UseHostedService` extension method to register the hosted service passed in `T`:</span></span>

```csharp
using System;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;

public static class Extensions
{
    public static IHostBuilder UseHostedService<T>(this IHostBuilder hostBuilder)
        where T : class, IHostedService, IDisposable
    {
        return hostBuilder.ConfigureServices(services =>
            services.AddHostedService<T>());
    }
}
```

## <a name="manage-the-host"></a><span data-ttu-id="a2704-765">Gestire l'host</span><span class="sxs-lookup"><span data-stu-id="a2704-765">Manage the host</span></span>

<span data-ttu-id="a2704-766">L'implementazione <xref:Microsoft.Extensions.Hosting.IHost> è responsabile dell'avvio e arresto delle implementazioni <xref:Microsoft.Extensions.Hosting.IHostedService> che sono registrate nel contenitore di servizi.</span><span class="sxs-lookup"><span data-stu-id="a2704-766">The <xref:Microsoft.Extensions.Hosting.IHost> implementation is responsible for starting and stopping the <xref:Microsoft.Extensions.Hosting.IHostedService> implementations that are registered in the service container.</span></span>

### <a name="run"></a><span data-ttu-id="a2704-767">Esegui</span><span class="sxs-lookup"><span data-stu-id="a2704-767">Run</span></span>

<span data-ttu-id="a2704-768"><xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.Run*> esegue l'app e blocca il thread di chiamata fino all'arresto dell'host:</span><span class="sxs-lookup"><span data-stu-id="a2704-768"><xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.Run*> runs the app and blocks the calling thread until the host is shut down:</span></span>

```csharp
public class Program
{
    public void Main(string[] args)
    {
        var host = new HostBuilder()
            .Build();

        host.Run();
    }
}
```

### <a name="runasync"></a><span data-ttu-id="a2704-769">RunAsync</span><span class="sxs-lookup"><span data-stu-id="a2704-769">RunAsync</span></span>

<span data-ttu-id="a2704-770"><xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.RunAsync*> esegue l'app e restituisce un elemento <xref:System.Threading.Tasks.Task> che viene completato quando viene attivato il token di annullamento o l'arresto:</span><span class="sxs-lookup"><span data-stu-id="a2704-770"><xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.RunAsync*> runs the app and returns a <xref:System.Threading.Tasks.Task> that completes when the cancellation token or shutdown is triggered:</span></span>

```csharp
public class Program
{
    public static async Task Main(string[] args)
    {
        var host = new HostBuilder()
            .Build();

        await host.RunAsync();
    }
}
```

### <a name="runconsoleasync"></a><span data-ttu-id="a2704-771">RunConsoleAsync</span><span class="sxs-lookup"><span data-stu-id="a2704-771">RunConsoleAsync</span></span>

<span data-ttu-id="a2704-772"><xref:Microsoft.Extensions.Hosting.HostingHostBuilderExtensions.RunConsoleAsync*>Abilita il supporto della console, compila e avvia l'host e attende la <kbd></kbd> + chiusura di CTRL<kbd>C</kbd>/SIGINT o SIGTERM.</span><span class="sxs-lookup"><span data-stu-id="a2704-772"><xref:Microsoft.Extensions.Hosting.HostingHostBuilderExtensions.RunConsoleAsync*> enables console support, builds and starts the host, and waits for <kbd>Ctrl</kbd>+<kbd>C</kbd>/SIGINT or SIGTERM to shut down.</span></span>

```csharp
public class Program
{
    public static async Task Main(string[] args)
    {
        var hostBuilder = new HostBuilder();

        await hostBuilder.RunConsoleAsync();
    }
}
```

### <a name="start-and-stopasync"></a><span data-ttu-id="a2704-773">Start e StopAsync</span><span class="sxs-lookup"><span data-stu-id="a2704-773">Start and StopAsync</span></span>

<span data-ttu-id="a2704-774"><xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.Start*> avvia l'host in modo sincrono.</span><span class="sxs-lookup"><span data-stu-id="a2704-774"><xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.Start*> starts the host synchronously.</span></span>

<span data-ttu-id="a2704-775"><xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.StopAsync*> prova ad arrestare l'host entro il timeout specificato.</span><span class="sxs-lookup"><span data-stu-id="a2704-775"><xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.StopAsync*> attempts to stop the host within the provided timeout.</span></span>

```csharp
public class Program
{
    public static async Task Main(string[] args)
    {
        var host = new HostBuilder()
            .Build();

        using (host)
        {
            host.Start();

            await host.StopAsync(TimeSpan.FromSeconds(5));
        }
    }
}
```

### <a name="startasync-and-stopasync"></a><span data-ttu-id="a2704-776">StartAsync e StopAsync</span><span class="sxs-lookup"><span data-stu-id="a2704-776">StartAsync and StopAsync</span></span>

<span data-ttu-id="a2704-777"><xref:Microsoft.Extensions.Hosting.IHost.StartAsync*> avvia l'app.</span><span class="sxs-lookup"><span data-stu-id="a2704-777"><xref:Microsoft.Extensions.Hosting.IHost.StartAsync*> starts the app.</span></span>

<span data-ttu-id="a2704-778"><xref:Microsoft.Extensions.Hosting.IHost.StopAsync*> arresta l'app.</span><span class="sxs-lookup"><span data-stu-id="a2704-778"><xref:Microsoft.Extensions.Hosting.IHost.StopAsync*> stops the app.</span></span>

```csharp
public class Program
{
    public static async Task Main(string[] args)
    {
        var host = new HostBuilder()
            .Build();

        using (host)
        {
            await host.StartAsync();

            await host.StopAsync();
        }
    }
}
```

### <a name="waitforshutdown"></a><span data-ttu-id="a2704-779">WaitForShutdown</span><span class="sxs-lookup"><span data-stu-id="a2704-779">WaitForShutdown</span></span>

<span data-ttu-id="a2704-780"><xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.WaitForShutdown*>viene attivato tramite <xref:Microsoft.Extensions.Hosting.IHostLifetime> , ad esempio (in `Microsoft.Extensions.Hosting.Internal.ConsoleLifetime` attesa di <kbd>CTRL</kbd> + <kbd>C</kbd>/SIGINT o SIGTERM).</span><span class="sxs-lookup"><span data-stu-id="a2704-780"><xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.WaitForShutdown*> is triggered via the <xref:Microsoft.Extensions.Hosting.IHostLifetime>, such as `Microsoft.Extensions.Hosting.Internal.ConsoleLifetime` (listens for <kbd>Ctrl</kbd>+<kbd>C</kbd>/SIGINT or SIGTERM).</span></span> <span data-ttu-id="a2704-781"><xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.WaitForShutdown*> chiama <xref:Microsoft.Extensions.Hosting.IHost.StopAsync*>.</span><span class="sxs-lookup"><span data-stu-id="a2704-781"><xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.WaitForShutdown*> calls <xref:Microsoft.Extensions.Hosting.IHost.StopAsync*>.</span></span>

```csharp
public class Program
{
    public void Main(string[] args)
    {
        var host = new HostBuilder()
            .Build();

        using (host)
        {
            host.Start();

            host.WaitForShutdown();
        }
    }
}
```

### <a name="waitforshutdownasync"></a><span data-ttu-id="a2704-782">WaitForShutdownAsync</span><span class="sxs-lookup"><span data-stu-id="a2704-782">WaitForShutdownAsync</span></span>

<span data-ttu-id="a2704-783"><xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.WaitForShutdownAsync*> restituisce un elemento <xref:System.Threading.Tasks.Task> che viene completato quando viene attivato l'arresto tramite il token specificato, quindi chiama <xref:Microsoft.Extensions.Hosting.IHost.StopAsync*>.</span><span class="sxs-lookup"><span data-stu-id="a2704-783"><xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.WaitForShutdownAsync*> returns a <xref:System.Threading.Tasks.Task> that completes when shutdown is triggered via the given token and calls <xref:Microsoft.Extensions.Hosting.IHost.StopAsync*>.</span></span>

```csharp
public class Program
{
    public static async Task Main(string[] args)
    {
        var host = new HostBuilder()
            .Build();

        using (host)
        {
            await host.StartAsync();

            await host.WaitForShutdownAsync();
        }

    }
}
```

### <a name="external-control"></a><span data-ttu-id="a2704-784">Controllo esterno</span><span class="sxs-lookup"><span data-stu-id="a2704-784">External control</span></span>

<span data-ttu-id="a2704-785">Il controllo esterno dell'host può essere ottenuto usando metodi che possono essere chiamati dall'esterno:</span><span class="sxs-lookup"><span data-stu-id="a2704-785">External control of the host can be achieved using methods that can be called externally:</span></span>

```csharp
public class Program
{
    private IHost _host;

    public Program()
    {
        _host = new HostBuilder()
            .Build();
    }

    public async Task StartAsync()
    {
        _host.StartAsync();
    }

    public async Task StopAsync()
    {
        using (_host)
        {
            await _host.StopAsync(TimeSpan.FromSeconds(5));
        }
    }
}
```

<span data-ttu-id="a2704-786"><xref:Microsoft.Extensions.Hosting.IHostLifetime.WaitForStartAsync*> viene chiamato all'avvio di <xref:Microsoft.Extensions.Hosting.IHost.StartAsync*>, che attende il completamento prima di continuare.</span><span class="sxs-lookup"><span data-stu-id="a2704-786"><xref:Microsoft.Extensions.Hosting.IHostLifetime.WaitForStartAsync*> is called at the start of <xref:Microsoft.Extensions.Hosting.IHost.StartAsync*>, which waits until it's complete before continuing.</span></span> <span data-ttu-id="a2704-787">Questo è utile per ritardare l'avvio fino al segnale trasmesso da un evento esterno.</span><span class="sxs-lookup"><span data-stu-id="a2704-787">This can be used to delay startup until signaled by an external event.</span></span>

## <a name="ihostingenvironment-interface"></a><span data-ttu-id="a2704-788">Interfaccia IHostingEnvironment</span><span class="sxs-lookup"><span data-stu-id="a2704-788">IHostingEnvironment interface</span></span>

<span data-ttu-id="a2704-789"><xref:Microsoft.Extensions.Hosting.IHostingEnvironment> include informazioni sull'ambiente di hosting dell'app.</span><span class="sxs-lookup"><span data-stu-id="a2704-789"><xref:Microsoft.Extensions.Hosting.IHostingEnvironment> provides information about the app's hosting environment.</span></span> <span data-ttu-id="a2704-790">Usare l'[inserimento di un costruttore](xref:fundamentals/dependency-injection) per ottenere <xref:Microsoft.Extensions.Hosting.IHostingEnvironment> per poterne usare le proprietà e i metodi di estensione:</span><span class="sxs-lookup"><span data-stu-id="a2704-790">Use [constructor injection](xref:fundamentals/dependency-injection) to obtain the <xref:Microsoft.Extensions.Hosting.IHostingEnvironment> in order to use its properties and extension methods:</span></span>

```csharp
public class MyClass
{
    private readonly IHostingEnvironment _env;

    public MyClass(IHostingEnvironment env)
    {
        _env = env;
    }

    public void DoSomething()
    {
        var environmentName = _env.EnvironmentName;
    }
}
```

<span data-ttu-id="a2704-791">Per altre informazioni, vedere <xref:fundamentals/environments>.</span><span class="sxs-lookup"><span data-stu-id="a2704-791">For more information, see <xref:fundamentals/environments>.</span></span>

## <a name="iapplicationlifetime-interface"></a><span data-ttu-id="a2704-792">Interfaccia IApplicationLifetime</span><span class="sxs-lookup"><span data-stu-id="a2704-792">IApplicationLifetime interface</span></span>

<span data-ttu-id="a2704-793"><xref:Microsoft.Extensions.Hosting.IApplicationLifetime> consente attività post-avvio e di arresto, incluse le richieste di arresto normale.</span><span class="sxs-lookup"><span data-stu-id="a2704-793"><xref:Microsoft.Extensions.Hosting.IApplicationLifetime> allows for post-startup and shutdown activities, including graceful shutdown requests.</span></span> <span data-ttu-id="a2704-794">Tre proprietà nell'interfaccia sono token di annullamento usati per registrare i metodi <xref:System.Action> che definiscono gli eventi di avvio e arresto.</span><span class="sxs-lookup"><span data-stu-id="a2704-794">Three properties on the interface are cancellation tokens used to register <xref:System.Action> methods that define startup and shutdown events.</span></span>

| <span data-ttu-id="a2704-795">Token di annullamento</span><span class="sxs-lookup"><span data-stu-id="a2704-795">Cancellation Token</span></span> | <span data-ttu-id="a2704-796">Attivato quando&#8230;</span><span class="sxs-lookup"><span data-stu-id="a2704-796">Triggered when&#8230;</span></span> |
| ------------------ | --------------------- |
| <xref:Microsoft.Extensions.Hosting.IApplicationLifetime.ApplicationStarted*> | <span data-ttu-id="a2704-797">L'host è stato completamente avviato.</span><span class="sxs-lookup"><span data-stu-id="a2704-797">The host has fully started.</span></span> |
| <xref:Microsoft.Extensions.Hosting.IApplicationLifetime.ApplicationStopped*> | <span data-ttu-id="a2704-798">L'host sta completando un arresto normale.</span><span class="sxs-lookup"><span data-stu-id="a2704-798">The host is completing a graceful shutdown.</span></span> <span data-ttu-id="a2704-799">Tutte le richieste devono essere elaborate.</span><span class="sxs-lookup"><span data-stu-id="a2704-799">All requests should be processed.</span></span> <span data-ttu-id="a2704-800">L'arresto è bloccato fino al completamento di questo evento.</span><span class="sxs-lookup"><span data-stu-id="a2704-800">Shutdown blocks until this event completes.</span></span> |
| <xref:Microsoft.Extensions.Hosting.IApplicationLifetime.ApplicationStopping*> | <span data-ttu-id="a2704-801">L'host sta eseguendo un arresto normale.</span><span class="sxs-lookup"><span data-stu-id="a2704-801">The host is performing a graceful shutdown.</span></span> <span data-ttu-id="a2704-802">È possibile che le richieste siano ancora in esecuzione.</span><span class="sxs-lookup"><span data-stu-id="a2704-802">Requests may still be processing.</span></span> <span data-ttu-id="a2704-803">L'arresto è bloccato fino al completamento di questo evento.</span><span class="sxs-lookup"><span data-stu-id="a2704-803">Shutdown blocks until this event completes.</span></span> |

<span data-ttu-id="a2704-804">Inserire tramite costruttore il servizio <xref:Microsoft.Extensions.Hosting.IApplicationLifetime> in qualsiasi classe.</span><span class="sxs-lookup"><span data-stu-id="a2704-804">Constructor-inject the <xref:Microsoft.Extensions.Hosting.IApplicationLifetime> service into any class.</span></span> <span data-ttu-id="a2704-805">L'[app di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/host/generic-host/samples/) usa l'inserimento di costruttori in una classe `LifetimeEventsHostedService` (implementazione <xref:Microsoft.Extensions.Hosting.IHostedService>) per registrare gli eventi.</span><span class="sxs-lookup"><span data-stu-id="a2704-805">The [sample app](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/host/generic-host/samples/) uses constructor injection into a `LifetimeEventsHostedService` class (an <xref:Microsoft.Extensions.Hosting.IHostedService> implementation) to register the events.</span></span>

<span data-ttu-id="a2704-806">*LifetimeEventsHostedService.cs*:</span><span class="sxs-lookup"><span data-stu-id="a2704-806">*LifetimeEventsHostedService.cs*:</span></span>

[!code-csharp[](generic-host/samples/2.x/GenericHostSample/LifetimeEventsHostedService.cs?name=snippet1)]

<span data-ttu-id="a2704-807"><xref:Microsoft.Extensions.Hosting.IApplicationLifetime.StopApplication*> richiede la chiusura dell'applicazione corrente.</span><span class="sxs-lookup"><span data-stu-id="a2704-807"><xref:Microsoft.Extensions.Hosting.IApplicationLifetime.StopApplication*> requests termination of the app.</span></span> <span data-ttu-id="a2704-808">La classe seguente usa <xref:Microsoft.Extensions.Hosting.IApplicationLifetime.StopApplication*> per arrestare normalmente un'app quando viene chiamato il metodo `Shutdown` della classe:</span><span class="sxs-lookup"><span data-stu-id="a2704-808">The following class uses <xref:Microsoft.Extensions.Hosting.IApplicationLifetime.StopApplication*> to gracefully shut down an app when the class's `Shutdown` method is called:</span></span>

```csharp
public class MyClass
{
    private readonly IApplicationLifetime _appLifetime;

    public MyClass(IApplicationLifetime appLifetime)
    {
        _appLifetime = appLifetime;
    }

    public void Shutdown()
    {
        _appLifetime.StopApplication();
    }
}
```

::: moniker-end

## <a name="additional-resources"></a><span data-ttu-id="a2704-809">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="a2704-809">Additional resources</span></span>

* <xref:fundamentals/host/hosted-services>
