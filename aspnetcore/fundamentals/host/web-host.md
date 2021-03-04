---
title: Host Web ASP.NET Core
author: rick-anderson
description: Informazioni sull'host Web in ASP.NET Core, responsabile della gestione dell'avvio e della durata delle app.
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 10/07/2019
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
uid: fundamentals/host/web-host
ms.openlocfilehash: fa9b1941d6dcda30855a4729dfa1cd78f897d9b6
ms.sourcegitcommit: a1db01b4d3bd8c57d7a9c94ce122a6db68002d66
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/04/2021
ms.locfileid: "102109975"
---
# <a name="aspnet-core-web-host"></a><span data-ttu-id="2662b-103">Host Web ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="2662b-103">ASP.NET Core Web Host</span></span>

<span data-ttu-id="2662b-104">ASP.NET Core le app configurano e avviano un *host*.</span><span class="sxs-lookup"><span data-stu-id="2662b-104">ASP.NET Core apps configure and launch a *host*.</span></span> <span data-ttu-id="2662b-105">L'host è responsabile della gestione dell'avvio e della durata delle app.</span><span class="sxs-lookup"><span data-stu-id="2662b-105">The host is responsible for app startup and lifetime management.</span></span> <span data-ttu-id="2662b-106">L'host configura almeno un server e una pipeline di elaborazione delle richieste.</span><span class="sxs-lookup"><span data-stu-id="2662b-106">At a minimum, the host configures a server and a request processing pipeline.</span></span> <span data-ttu-id="2662b-107">L'host può anche configurare la registrazione, l'inserimento delle dipendenze e la configurazione.</span><span class="sxs-lookup"><span data-stu-id="2662b-107">The host can also set up logging, dependency injection, and configuration.</span></span>

::: moniker range=">= aspnetcore-3.0"

<span data-ttu-id="2662b-108">Questo articolo descrive l'host Web, che rimane disponibile solo per compatibilità con le versioni precedenti.</span><span class="sxs-lookup"><span data-stu-id="2662b-108">This article covers the Web Host, which remains available only for backward compatibility.</span></span> <span data-ttu-id="2662b-109">I modelli di ASP.NET Core creano un [host generico .NET](<xref:fundamentals/host/generic-host>), consigliato per tutti i tipi di app.</span><span class="sxs-lookup"><span data-stu-id="2662b-109">The ASP.NET Core templates create a [.NET Generic Host](<xref:fundamentals/host/generic-host>), which is recommended for all app types.</span></span>

::: moniker-end

::: moniker range="< aspnetcore-3.0"

<span data-ttu-id="2662b-110">Questo articolo descrive l'host Web destinato all'hosting di app Web.</span><span class="sxs-lookup"><span data-stu-id="2662b-110">This article covers the Web Host, which is for hosting web apps.</span></span> <span data-ttu-id="2662b-111">Per altri tipi di app, usare l'[host generico](xref:fundamentals/host/generic-host).</span><span class="sxs-lookup"><span data-stu-id="2662b-111">For other kinds of apps, use the [Generic Host](xref:fundamentals/host/generic-host).</span></span>

::: moniker-end

## <a name="set-up-a-host"></a><span data-ttu-id="2662b-112">Configurare un host</span><span class="sxs-lookup"><span data-stu-id="2662b-112">Set up a host</span></span>

<span data-ttu-id="2662b-113">Creare un host usando un'istanza di [IWebHostBuilder](/dotnet/api/microsoft.aspnetcore.hosting.iwebhostbuilder).</span><span class="sxs-lookup"><span data-stu-id="2662b-113">Create a host using an instance of [IWebHostBuilder](/dotnet/api/microsoft.aspnetcore.hosting.iwebhostbuilder).</span></span> <span data-ttu-id="2662b-114">Questa operazione viene in genere eseguita nel punto di ingresso dell'app, il metodo `Main`.</span><span class="sxs-lookup"><span data-stu-id="2662b-114">This is typically performed in the app's entry point, the `Main` method.</span></span>

<span data-ttu-id="2662b-115">Nei modelli di progetto `Main` si trova in *Program.cs*.</span><span class="sxs-lookup"><span data-stu-id="2662b-115">In the project templates, `Main` is located in *Program.cs*.</span></span> <span data-ttu-id="2662b-116">Un'app tipica chiama [CreateDefaultBuilder](/dotnet/api/microsoft.aspnetcore.webhost.createdefaultbuilder) per avviare la configurazione di un host:</span><span class="sxs-lookup"><span data-stu-id="2662b-116">A typical app calls [CreateDefaultBuilder](/dotnet/api/microsoft.aspnetcore.webhost.createdefaultbuilder) to start setting up a host:</span></span>

```csharp
public class Program
{
    public static void Main(string[] args)
    {
        CreateWebHostBuilder(args).Build().Run();
    }

    public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
        WebHost.CreateDefaultBuilder(args)
            .UseStartup<Startup>();
}
```

<span data-ttu-id="2662b-117">Il codice che chiama `CreateDefaultBuilder` è incluso in un metodo denominato `CreateWebHostBuilder`, che lo separa dal codice in `Main` che chiama `Run` per l'oggetto generatore.</span><span class="sxs-lookup"><span data-stu-id="2662b-117">The code that calls `CreateDefaultBuilder` is in a method named `CreateWebHostBuilder`, which separates it from the code in `Main` that calls `Run` on the builder object.</span></span> <span data-ttu-id="2662b-118">Questa separazione è necessaria se si usano gli [strumenti di Entity Framework Core](/ef/core/miscellaneous/cli/).</span><span class="sxs-lookup"><span data-stu-id="2662b-118">This separation is required if you use [Entity Framework Core tools](/ef/core/miscellaneous/cli/).</span></span> <span data-ttu-id="2662b-119">Gli strumenti si aspettano di trovare un metodo `CreateWebHostBuilder` che possono chiamare in fase di progettazione per configurare l'host senza eseguire l'app.</span><span class="sxs-lookup"><span data-stu-id="2662b-119">The tools expect to find a `CreateWebHostBuilder` method that they can call at design time to configure the host without running the app.</span></span> <span data-ttu-id="2662b-120">Un'alternativa consiste nell'implementare `IDesignTimeDbContextFactory`.</span><span class="sxs-lookup"><span data-stu-id="2662b-120">An alternative is to implement `IDesignTimeDbContextFactory`.</span></span> <span data-ttu-id="2662b-121">Per altre informazioni, vedere [Creazione DbContext in fase di progettazione](/ef/core/miscellaneous/cli/dbcontext-creation).</span><span class="sxs-lookup"><span data-stu-id="2662b-121">For more information, see [Design-time DbContext Creation](/ef/core/miscellaneous/cli/dbcontext-creation).</span></span>

<span data-ttu-id="2662b-122">`CreateDefaultBuilder` esegue queste attività:</span><span class="sxs-lookup"><span data-stu-id="2662b-122">`CreateDefaultBuilder` performs the following tasks:</span></span>

::: moniker range=">= aspnetcore-5.0"
* <span data-ttu-id="2662b-123">Configura il server [Kestrel](xref:fundamentals/servers/kestrel) come server Web usando i provider di configurazione dell'host dell'app.</span><span class="sxs-lookup"><span data-stu-id="2662b-123">Configures [Kestrel](xref:fundamentals/servers/kestrel) server as the web server using the app's hosting configuration providers.</span></span> <span data-ttu-id="2662b-124">Per le opzioni predefinite del server Kestrel, vedere <xref:fundamentals/servers/kestrel/options>.</span><span class="sxs-lookup"><span data-stu-id="2662b-124">For the Kestrel server's default options, see <xref:fundamentals/servers/kestrel/options>.</span></span>
::: moniker-end
::: moniker range="< aspnetcore-5.0"
* <span data-ttu-id="2662b-125">Configura il server [Kestrel](xref:fundamentals/servers/kestrel) come server Web usando i provider di configurazione dell'host dell'app.</span><span class="sxs-lookup"><span data-stu-id="2662b-125">Configures [Kestrel](xref:fundamentals/servers/kestrel) server as the web server using the app's hosting configuration providers.</span></span> <span data-ttu-id="2662b-126">Per le opzioni predefinite del server Kestrel, vedere <xref:fundamentals/servers/kestrel#kestrel-options>.</span><span class="sxs-lookup"><span data-stu-id="2662b-126">For the Kestrel server's default options, see <xref:fundamentals/servers/kestrel#kestrel-options>.</span></span>
::: moniker-end
* <span data-ttu-id="2662b-127">Imposta la [radice del contenuto](xref:fundamentals/index#content-root) sul percorso restituito da [Directory. GetCurrentDirectory](/dotnet/api/system.io.directory.getcurrentdirectory).</span><span class="sxs-lookup"><span data-stu-id="2662b-127">Sets the [content root](xref:fundamentals/index#content-root) to the path returned by [Directory.GetCurrentDirectory](/dotnet/api/system.io.directory.getcurrentdirectory).</span></span>
* <span data-ttu-id="2662b-128">Carica la [configurazione dell'host](#host-configuration-values) da:</span><span class="sxs-lookup"><span data-stu-id="2662b-128">Loads [host configuration](#host-configuration-values) from:</span></span>
  * <span data-ttu-id="2662b-129">Le variabili di ambiente con prefisso `ASPNETCORE_` (ad esempio, `ASPNETCORE_ENVIRONMENT`).</span><span class="sxs-lookup"><span data-stu-id="2662b-129">Environment variables prefixed with `ASPNETCORE_` (for example, `ASPNETCORE_ENVIRONMENT`).</span></span>
  * <span data-ttu-id="2662b-130">Argomenti della riga di comando.</span><span class="sxs-lookup"><span data-stu-id="2662b-130">Command-line arguments.</span></span>
* <span data-ttu-id="2662b-131">Carica la configurazione dell'app nell'ordine seguente da:</span><span class="sxs-lookup"><span data-stu-id="2662b-131">Loads app configuration in the following order from:</span></span>
  * <span data-ttu-id="2662b-132">*appsettings.json*.</span><span class="sxs-lookup"><span data-stu-id="2662b-132">*appsettings.json*.</span></span>
  * <span data-ttu-id="2662b-133">*appsettings.{Environment}.json*.</span><span class="sxs-lookup"><span data-stu-id="2662b-133">*appsettings.{Environment}.json*.</span></span>
  * <span data-ttu-id="2662b-134">[Segreti dell'utente](xref:security/app-secrets) quando l'app viene eseguita nell'ambiente `Development` usando l'assembly di ingresso.</span><span class="sxs-lookup"><span data-stu-id="2662b-134">[User secrets](xref:security/app-secrets) when the app runs in the `Development` environment using the entry assembly.</span></span>
  * <span data-ttu-id="2662b-135">Variabili di ambiente.</span><span class="sxs-lookup"><span data-stu-id="2662b-135">Environment variables.</span></span>
  * <span data-ttu-id="2662b-136">Argomenti della riga di comando.</span><span class="sxs-lookup"><span data-stu-id="2662b-136">Command-line arguments.</span></span>
* <span data-ttu-id="2662b-137">Configura la [registrazione](xref:fundamentals/logging/index) per l'output della console e del debug.</span><span class="sxs-lookup"><span data-stu-id="2662b-137">Configures [logging](xref:fundamentals/logging/index) for console and debug output.</span></span> <span data-ttu-id="2662b-138">La registrazione include le regole di [filtro dei log](xref:fundamentals/logging/index#log-filtering) specificate in una sezione di configurazione di registrazione di un oggetto *appsettings.json* o *appSettings. { Environment}. file JSON* .</span><span class="sxs-lookup"><span data-stu-id="2662b-138">Logging includes [log filtering](xref:fundamentals/logging/index#log-filtering) rules specified in a Logging configuration section of an *appsettings.json* or *appsettings.{Environment}.json* file.</span></span>
* <span data-ttu-id="2662b-139">Quando è in esecuzione dietro IIS con il [modulo ASP.NET Core](xref:host-and-deploy/aspnet-core-module), `CreateDefaultBuilder` Abilita l' [integrazione di IIS](xref:host-and-deploy/iis/index), che configura l'indirizzo di base e la porta dell'app.</span><span class="sxs-lookup"><span data-stu-id="2662b-139">When running behind IIS with the [ASP.NET Core Module](xref:host-and-deploy/aspnet-core-module), `CreateDefaultBuilder` enables [IIS Integration](xref:host-and-deploy/iis/index), which configures the app's base address and port.</span></span> <span data-ttu-id="2662b-140">L'integrazione di IIS configura inoltre l'app per l'[acquisizione degli errori di avvio](#capture-startup-errors).</span><span class="sxs-lookup"><span data-stu-id="2662b-140">IIS Integration also configures the app to [capture startup errors](#capture-startup-errors).</span></span> <span data-ttu-id="2662b-141">Per le opzioni predefinite di IIS, vedere <xref:host-and-deploy/iis/index#iis-options>.</span><span class="sxs-lookup"><span data-stu-id="2662b-141">For the IIS default options, see <xref:host-and-deploy/iis/index#iis-options>.</span></span>
* <span data-ttu-id="2662b-142">Imposta [ServiceProviderOptions.ValidateScopes](/dotnet/api/microsoft.extensions.dependencyinjection.serviceprovideroptions.validatescopes) to `true` se l'ambiente dell'app è lo sviluppo.</span><span class="sxs-lookup"><span data-stu-id="2662b-142">Sets [ServiceProviderOptions.ValidateScopes](/dotnet/api/microsoft.extensions.dependencyinjection.serviceprovideroptions.validatescopes) to `true` if the app's environment is Development.</span></span> <span data-ttu-id="2662b-143">Per ulteriori informazioni, vedere [Convalida dell'ambito](#scope-validation).</span><span class="sxs-lookup"><span data-stu-id="2662b-143">For more information, see [Scope validation](#scope-validation).</span></span>

<span data-ttu-id="2662b-144">La configurazione definita da `CreateDefaultBuilder` può essere sottoposta a override e aumentata tramite [ConfigureAppConfiguration](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilderextensions.configureappconfiguration), [ConfigureLogging](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilderextensions.configurelogging) e altri metodi e metodi di estensione di [IWebHostBuilder](/dotnet/api/microsoft.aspnetcore.hosting.iwebhostbuilder).</span><span class="sxs-lookup"><span data-stu-id="2662b-144">The configuration defined by `CreateDefaultBuilder` can be overridden and augmented by [ConfigureAppConfiguration](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilderextensions.configureappconfiguration), [ConfigureLogging](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilderextensions.configurelogging), and other methods and extension methods of [IWebHostBuilder](/dotnet/api/microsoft.aspnetcore.hosting.iwebhostbuilder).</span></span> <span data-ttu-id="2662b-145">Di seguito sono riportati alcuni esempi:</span><span class="sxs-lookup"><span data-stu-id="2662b-145">A few examples follow:</span></span>

* <span data-ttu-id="2662b-146">[ConfigureAppConfiguration](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilderextensions.configureappconfiguration) viene usato per specificare altri elementi `IConfiguration` per l'app.</span><span class="sxs-lookup"><span data-stu-id="2662b-146">[ConfigureAppConfiguration](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilderextensions.configureappconfiguration) is used to specify additional `IConfiguration` for the app.</span></span> <span data-ttu-id="2662b-147">La chiamata a `ConfigureAppConfiguration` seguente consente di aggiungere un delegato per includere la configurazione dell'app nel file *appsettings.xml*.</span><span class="sxs-lookup"><span data-stu-id="2662b-147">The following `ConfigureAppConfiguration` call adds a delegate to include app configuration in the *appsettings.xml* file.</span></span> <span data-ttu-id="2662b-148">`ConfigureAppConfiguration` può essere chiamato più volte.</span><span class="sxs-lookup"><span data-stu-id="2662b-148">`ConfigureAppConfiguration` may be called multiple times.</span></span> <span data-ttu-id="2662b-149">Si noti che questa configurazione non è valida per l'host (ad esempio, gli URL del server o l'ambiente).</span><span class="sxs-lookup"><span data-stu-id="2662b-149">Note that this configuration doesn't apply to the host (for example, server URLs or environment).</span></span> <span data-ttu-id="2662b-150">Vedere la sezione [Valori di configurazione dell'host](#host-configuration-values).</span><span class="sxs-lookup"><span data-stu-id="2662b-150">See the [Host configuration values](#host-configuration-values) section.</span></span>

    ```csharp
    WebHost.CreateDefaultBuilder(args)
        .ConfigureAppConfiguration((hostingContext, config) =>
        {
            config.AddXmlFile("appsettings.xml", optional: true, reloadOnChange: true);
        })
        ...
    ```

* <span data-ttu-id="2662b-151">La chiamata a `ConfigureLogging` seguente consente di aggiungere un delegato per configurare il livello di registrazione minimo ([SetMinimumLevel](/dotnet/api/microsoft.extensions.logging.loggingbuilderextensions.setminimumlevel)) [LogLevel.Warning](/dotnet/api/microsoft.extensions.logging.loglevel).</span><span class="sxs-lookup"><span data-stu-id="2662b-151">The following `ConfigureLogging` call adds a delegate to configure the minimum logging level ([SetMinimumLevel](/dotnet/api/microsoft.extensions.logging.loggingbuilderextensions.setminimumlevel)) to [LogLevel.Warning](/dotnet/api/microsoft.extensions.logging.loglevel).</span></span> <span data-ttu-id="2662b-152">Questa impostazione sostituisce le impostazioni *appsettings.Development.json* ( `LogLevel.Debug` ) e *appsettings.Production.json* ( `LogLevel.Error` ) configurate da `CreateDefaultBuilder` .</span><span class="sxs-lookup"><span data-stu-id="2662b-152">This setting overrides the settings in *appsettings.Development.json* (`LogLevel.Debug`) and *appsettings.Production.json* (`LogLevel.Error`) configured by `CreateDefaultBuilder`.</span></span> <span data-ttu-id="2662b-153">`ConfigureLogging` può essere chiamato più volte.</span><span class="sxs-lookup"><span data-stu-id="2662b-153">`ConfigureLogging` may be called multiple times.</span></span>

    ```csharp
    WebHost.CreateDefaultBuilder(args)
        .ConfigureLogging(logging => 
        {
            logging.SetMinimumLevel(LogLevel.Warning);
        })
        ...
    ```

::: moniker range=">= aspnetcore-2.2"

* <span data-ttu-id="2662b-154">La chiamata a `ConfigureKestrel` seguente sostituisce il valore predefinito [Limits.MaxRequestBodySize](/dotnet/api/microsoft.aspnetcore.server.kestrel.core.kestrelserverlimits.maxrequestbodysize) di 30.000.000 di byte stabilito durante la configurazione del Kestrel da `CreateDefaultBuilder`:</span><span class="sxs-lookup"><span data-stu-id="2662b-154">The following call to `ConfigureKestrel` overrides the default [Limits.MaxRequestBodySize](/dotnet/api/microsoft.aspnetcore.server.kestrel.core.kestrelserverlimits.maxrequestbodysize) of 30,000,000 bytes established when Kestrel was configured by `CreateDefaultBuilder`:</span></span>

    ```csharp
    WebHost.CreateDefaultBuilder(args)
        .ConfigureKestrel((context, options) =>
        {
            options.Limits.MaxRequestBodySize = 20000000;
        });
    ```

::: moniker-end

::: moniker range="< aspnetcore-2.2"

* <span data-ttu-id="2662b-155">La chiamata a [UseKestrel](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilderkestrelextensions.usekestrel) seguente sostituisce il valore predefinito [Limits.MaxRequestBodySize](/dotnet/api/microsoft.aspnetcore.server.kestrel.core.kestrelserverlimits.maxrequestbodysize) di 30.000.000 di byte stabilito durante la configurazione del Kestrel da `CreateDefaultBuilder`:</span><span class="sxs-lookup"><span data-stu-id="2662b-155">The following call to [UseKestrel](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilderkestrelextensions.usekestrel) overrides the default [Limits.MaxRequestBodySize](/dotnet/api/microsoft.aspnetcore.server.kestrel.core.kestrelserverlimits.maxrequestbodysize) of 30,000,000 bytes established when Kestrel was configured by `CreateDefaultBuilder`:</span></span>

    ```csharp
    WebHost.CreateDefaultBuilder(args)
        .UseKestrel(options =>
        {
            options.Limits.MaxRequestBodySize = 20000000;
        });
    ```

::: moniker-end

<span data-ttu-id="2662b-156">La [radice del contenuto](xref:fundamentals/index#content-root) determina la posizione in cui l'host cerca i file dei contenuti, ad esempio i file di visualizzazione MVC.</span><span class="sxs-lookup"><span data-stu-id="2662b-156">The [content root](xref:fundamentals/index#content-root) determines where the host searches for content files, such as MVC view files.</span></span> <span data-ttu-id="2662b-157">Quando l'app viene avviata dalla cartella radice del progetto, la cartella radice del progetto viene usata come radice del contenuto.</span><span class="sxs-lookup"><span data-stu-id="2662b-157">When the app is started from the project's root folder, the project's root folder is used as the content root.</span></span> <span data-ttu-id="2662b-158">Si tratta dell'impostazione predefinita usata in [Visual Studio](https://visualstudio.microsoft.com) e nei [nuovi modelli dotnet](/dotnet/core/tools/dotnet-new).</span><span class="sxs-lookup"><span data-stu-id="2662b-158">This is the default used in [Visual Studio](https://visualstudio.microsoft.com) and the [dotnet new templates](/dotnet/core/tools/dotnet-new).</span></span>

<span data-ttu-id="2662b-159">Per altre informazioni sulla configurazione dell'app, vedere <xref:fundamentals/configuration/index>.</span><span class="sxs-lookup"><span data-stu-id="2662b-159">For more information on app configuration, see <xref:fundamentals/configuration/index>.</span></span>

> [!NOTE]
> <span data-ttu-id="2662b-160">In alternativa all'uso del metodo `CreateDefaultBuilder` statico, un approccio supportato in ASP.NET Core 2.x consiste nel creare un host da [WebHostBuilder](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilder).</span><span class="sxs-lookup"><span data-stu-id="2662b-160">As an alternative to using the static `CreateDefaultBuilder` method, creating a host from [WebHostBuilder](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilder) is a supported approach with ASP.NET Core 2.x.</span></span>

<span data-ttu-id="2662b-161">Quando si configura un host, è possibile specificare i metodi [Configure](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilderextensions.configure) e [ConfigureServices](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilder.configureservices).</span><span class="sxs-lookup"><span data-stu-id="2662b-161">When setting up a host, [Configure](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilderextensions.configure) and [ConfigureServices](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilder.configureservices) methods can be provided.</span></span> <span data-ttu-id="2662b-162">Se viene specificata una classe `Startup`, la classe deve definire un metodo `Configure`.</span><span class="sxs-lookup"><span data-stu-id="2662b-162">If a `Startup` class is specified, it must define a `Configure` method.</span></span> <span data-ttu-id="2662b-163">Per altre informazioni, vedere <xref:fundamentals/startup>.</span><span class="sxs-lookup"><span data-stu-id="2662b-163">For more information, see <xref:fundamentals/startup>.</span></span> <span data-ttu-id="2662b-164">Se vengono effettuate più chiamate a `ConfigureServices`, le chiamate vengono aggiunte l'una all'altra.</span><span class="sxs-lookup"><span data-stu-id="2662b-164">Multiple calls to `ConfigureServices` append to one another.</span></span> <span data-ttu-id="2662b-165">Più chiamate a `Configure` o `UseStartup` in `WebHostBuilder` sostituiscono le impostazioni precedenti.</span><span class="sxs-lookup"><span data-stu-id="2662b-165">Multiple calls to `Configure` or `UseStartup` on the `WebHostBuilder` replace previous settings.</span></span>

## <a name="host-configuration-values"></a><span data-ttu-id="2662b-166">Valori di configurazione dell'host</span><span class="sxs-lookup"><span data-stu-id="2662b-166">Host configuration values</span></span>

<span data-ttu-id="2662b-167">[WebHostBuilder](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilder) si basa sugli approcci seguenti per impostare i valori di configurazione dell'host:</span><span class="sxs-lookup"><span data-stu-id="2662b-167">[WebHostBuilder](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilder) relies on the following approaches to set the host configuration values:</span></span>

* <span data-ttu-id="2662b-168">Configurazione del generatore di host, che include le variabili di ambiente nel formato `ASPNETCORE_{configurationKey}`.</span><span class="sxs-lookup"><span data-stu-id="2662b-168">Host builder configuration, which includes environment variables with the format `ASPNETCORE_{configurationKey}`.</span></span> <span data-ttu-id="2662b-169">Ad esempio: `ASPNETCORE_ENVIRONMENT`.</span><span class="sxs-lookup"><span data-stu-id="2662b-169">For example, `ASPNETCORE_ENVIRONMENT`.</span></span>
* <span data-ttu-id="2662b-170">Le estensioni come [UseContentRoot](/dotnet/api/microsoft.aspnetcore.hosting.hostingabstractionswebhostbuilderextensions.usecontentroot) e [UseConfiguration](/dotnet/api/microsoft.aspnetcore.hosting.hostingabstractionswebhostbuilderextensions.useconfiguration) (vedere la sezione [Override della configurazione](#override-configuration)).</span><span class="sxs-lookup"><span data-stu-id="2662b-170">Extensions such as [UseContentRoot](/dotnet/api/microsoft.aspnetcore.hosting.hostingabstractionswebhostbuilderextensions.usecontentroot) and [UseConfiguration](/dotnet/api/microsoft.aspnetcore.hosting.hostingabstractionswebhostbuilderextensions.useconfiguration) (see the [Override configuration](#override-configuration) section).</span></span>
* <span data-ttu-id="2662b-171">[UseSetting](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilder.usesetting) e chiave associata.</span><span class="sxs-lookup"><span data-stu-id="2662b-171">[UseSetting](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilder.usesetting) and the associated key.</span></span> <span data-ttu-id="2662b-172">Quando si imposta un valore con `UseSetting`, il valore viene impostato come stringa indipendentemente dal tipo.</span><span class="sxs-lookup"><span data-stu-id="2662b-172">When setting a value with `UseSetting`, the value is set as a string regardless of the type.</span></span>

<span data-ttu-id="2662b-173">L'host usa l'ultima opzione che imposta un valore.</span><span class="sxs-lookup"><span data-stu-id="2662b-173">The host uses whichever option sets a value last.</span></span> <span data-ttu-id="2662b-174">Per altre informazioni, vedere [Override della configurazione](#override-configuration) nella prossima sezione.</span><span class="sxs-lookup"><span data-stu-id="2662b-174">For more information, see [Override configuration](#override-configuration) in the next section.</span></span>

### <a name="application-key-name"></a><span data-ttu-id="2662b-175">Chiave applicazione (nome)</span><span class="sxs-lookup"><span data-stu-id="2662b-175">Application Key (Name)</span></span>

::: moniker range=">= aspnetcore-3.0"

<span data-ttu-id="2662b-176">La `IWebHostEnvironment.ApplicationName` proprietà viene impostata automaticamente quando [UseStartup](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilderextensions.usestartup) o [Configure](/dotnet/api/microsoft.aspnetcore.hosting.istartup.configure) viene chiamato durante la costruzione dell'host.</span><span class="sxs-lookup"><span data-stu-id="2662b-176">The `IWebHostEnvironment.ApplicationName` property is automatically set when [UseStartup](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilderextensions.usestartup) or [Configure](/dotnet/api/microsoft.aspnetcore.hosting.istartup.configure) is called during host construction.</span></span> <span data-ttu-id="2662b-177">Il valore viene impostato sul nome dell'assembly contenente il punto di ingresso dell'app.</span><span class="sxs-lookup"><span data-stu-id="2662b-177">The value is set to the name of the assembly containing the app's entry point.</span></span> <span data-ttu-id="2662b-178">Per impostare il valore in modo esplicito, usare [WebHostDefaults.ApplicationKey](/dotnet/api/microsoft.aspnetcore.hosting.webhostdefaults.applicationkey):</span><span class="sxs-lookup"><span data-stu-id="2662b-178">To set the value explicitly, use the [WebHostDefaults.ApplicationKey](/dotnet/api/microsoft.aspnetcore.hosting.webhostdefaults.applicationkey):</span></span>

::: moniker-end

::: moniker range="< aspnetcore-3.0"

<span data-ttu-id="2662b-179">La proprietà [IHostingEnvironment.ApplicationName](/dotnet/api/microsoft.extensions.hosting.ihostingenvironment.applicationname) viene impostata automaticamente quando si chiama [UseStartup](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilderextensions.usestartup) o [Configure](/dotnet/api/microsoft.aspnetcore.hosting.istartup.configure) durante la costruzione dell'host.</span><span class="sxs-lookup"><span data-stu-id="2662b-179">The [IHostingEnvironment.ApplicationName](/dotnet/api/microsoft.extensions.hosting.ihostingenvironment.applicationname) property is automatically set when [UseStartup](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilderextensions.usestartup) or [Configure](/dotnet/api/microsoft.aspnetcore.hosting.istartup.configure) is called during host construction.</span></span> <span data-ttu-id="2662b-180">Il valore viene impostato sul nome dell'assembly contenente il punto di ingresso dell'app.</span><span class="sxs-lookup"><span data-stu-id="2662b-180">The value is set to the name of the assembly containing the app's entry point.</span></span> <span data-ttu-id="2662b-181">Per impostare il valore in modo esplicito, usare [WebHostDefaults.ApplicationKey](/dotnet/api/microsoft.aspnetcore.hosting.webhostdefaults.applicationkey):</span><span class="sxs-lookup"><span data-stu-id="2662b-181">To set the value explicitly, use the [WebHostDefaults.ApplicationKey](/dotnet/api/microsoft.aspnetcore.hosting.webhostdefaults.applicationkey):</span></span>

::: moniker-end

<span data-ttu-id="2662b-182">**Chiave**: applicationName</span><span class="sxs-lookup"><span data-stu-id="2662b-182">**Key**: applicationName</span></span>  
<span data-ttu-id="2662b-183">**Tipo**: *stringa*</span><span class="sxs-lookup"><span data-stu-id="2662b-183">**Type**: *string*</span></span>  
<span data-ttu-id="2662b-184">**Impostazione predefinita**: il nome dell'assembly contenente il punto di ingresso dell'app.</span><span class="sxs-lookup"><span data-stu-id="2662b-184">**Default**: The name of the assembly containing the app's entry point.</span></span>  
<span data-ttu-id="2662b-185">**Imposta utilizzando**: `UseSetting`</span><span class="sxs-lookup"><span data-stu-id="2662b-185">**Set using**: `UseSetting`</span></span>  
<span data-ttu-id="2662b-186">**Variabile di ambiente**: `ASPNETCORE_APPLICATIONNAME`</span><span class="sxs-lookup"><span data-stu-id="2662b-186">**Environment variable**: `ASPNETCORE_APPLICATIONNAME`</span></span>

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseSetting(WebHostDefaults.ApplicationKey, "CustomApplicationName")
```

### <a name="capture-startup-errors"></a><span data-ttu-id="2662b-187">Acquisizione degli errori di avvio</span><span class="sxs-lookup"><span data-stu-id="2662b-187">Capture Startup Errors</span></span>

<span data-ttu-id="2662b-188">Questa impostazione controlla l'acquisizione degli errori di avvio.</span><span class="sxs-lookup"><span data-stu-id="2662b-188">This setting controls the capture of startup errors.</span></span>

<span data-ttu-id="2662b-189">**Chiave**: captureStartupErrors</span><span class="sxs-lookup"><span data-stu-id="2662b-189">**Key**: captureStartupErrors</span></span>  
<span data-ttu-id="2662b-190">**Tipo**: *bool* (`true` o `1`)</span><span class="sxs-lookup"><span data-stu-id="2662b-190">**Type**: *bool* (`true` or `1`)</span></span>  
<span data-ttu-id="2662b-191">**Impostazione predefinita**: il valore predefinito è `false` a meno che l'app non venga eseguita con Kestrel in IIS, in tal caso il valore predefinito è `true`.</span><span class="sxs-lookup"><span data-stu-id="2662b-191">**Default**: Defaults to `false` unless the app runs with Kestrel behind IIS, where the default is `true`.</span></span>  
<span data-ttu-id="2662b-192">**Imposta utilizzando**: `CaptureStartupErrors`</span><span class="sxs-lookup"><span data-stu-id="2662b-192">**Set using**: `CaptureStartupErrors`</span></span>  
<span data-ttu-id="2662b-193">**Variabile di ambiente**: `ASPNETCORE_CAPTURESTARTUPERRORS`</span><span class="sxs-lookup"><span data-stu-id="2662b-193">**Environment variable**: `ASPNETCORE_CAPTURESTARTUPERRORS`</span></span>

<span data-ttu-id="2662b-194">Quando è `false`, gli errori durante l'avvio causano la chiusura dell'host.</span><span class="sxs-lookup"><span data-stu-id="2662b-194">When `false`, errors during startup result in the host exiting.</span></span> <span data-ttu-id="2662b-195">Quando è `true`, l'host acquisisce le eccezioni durante l'avvio e tenta di avviare il server.</span><span class="sxs-lookup"><span data-stu-id="2662b-195">When `true`, the host captures exceptions during startup and attempts to start the server.</span></span>

```csharp
WebHost.CreateDefaultBuilder(args)
    .CaptureStartupErrors(true)
```

### <a name="content-root"></a><span data-ttu-id="2662b-196">Radice del contenuto</span><span class="sxs-lookup"><span data-stu-id="2662b-196">Content root</span></span>

<span data-ttu-id="2662b-197">Questa impostazione determina la posizione in cui ASP.NET Core inizia la ricerca dei file di contenuto.</span><span class="sxs-lookup"><span data-stu-id="2662b-197">This setting determines where ASP.NET Core begins searching for content files.</span></span>

<span data-ttu-id="2662b-198">**Chiave**: contentRoot</span><span class="sxs-lookup"><span data-stu-id="2662b-198">**Key**: contentRoot</span></span>  
<span data-ttu-id="2662b-199">**Tipo**: *stringa*</span><span class="sxs-lookup"><span data-stu-id="2662b-199">**Type**: *string*</span></span>  
<span data-ttu-id="2662b-200">**Impostazione predefinita**: il valore predefinito corrisponde alla cartella contenente l'assembly dell'app.</span><span class="sxs-lookup"><span data-stu-id="2662b-200">**Default**: Defaults to the folder where the app assembly resides.</span></span>  
<span data-ttu-id="2662b-201">**Imposta utilizzando**: `UseContentRoot`</span><span class="sxs-lookup"><span data-stu-id="2662b-201">**Set using**: `UseContentRoot`</span></span>  
<span data-ttu-id="2662b-202">**Variabile di ambiente**: `ASPNETCORE_CONTENTROOT`</span><span class="sxs-lookup"><span data-stu-id="2662b-202">**Environment variable**: `ASPNETCORE_CONTENTROOT`</span></span>

<span data-ttu-id="2662b-203">La radice del contenuto viene usata anche come percorso di base per la [radice Web](xref:fundamentals/index#web-root).</span><span class="sxs-lookup"><span data-stu-id="2662b-203">The content root is also used as the base path for the [web root](xref:fundamentals/index#web-root).</span></span> <span data-ttu-id="2662b-204">Se il percorso radice del contenuto non esiste, l'host non viene avviato.</span><span class="sxs-lookup"><span data-stu-id="2662b-204">If the content root path doesn't exist, the host fails to start.</span></span>

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseContentRoot("c:\\<content-root>")
```

<span data-ttu-id="2662b-205">Per altre informazioni, vedere:</span><span class="sxs-lookup"><span data-stu-id="2662b-205">For more information, see:</span></span>

* [<span data-ttu-id="2662b-206">Nozioni fondamentali: radice del contenuto</span><span class="sxs-lookup"><span data-stu-id="2662b-206">Fundamentals: Content root</span></span>](xref:fundamentals/index#content-root)
* [<span data-ttu-id="2662b-207">Radice Web</span><span class="sxs-lookup"><span data-stu-id="2662b-207">Web root</span></span>](#web-root)

### <a name="detailed-errors"></a><span data-ttu-id="2662b-208">Errori dettagliati</span><span class="sxs-lookup"><span data-stu-id="2662b-208">Detailed Errors</span></span>

<span data-ttu-id="2662b-209">Determina l'acquisizione degli errori dettagliati.</span><span class="sxs-lookup"><span data-stu-id="2662b-209">Determines if detailed errors should be captured.</span></span>

<span data-ttu-id="2662b-210">**Chiave**: detailedErrors</span><span class="sxs-lookup"><span data-stu-id="2662b-210">**Key**: detailedErrors</span></span>  
<span data-ttu-id="2662b-211">**Tipo**: *bool* (`true` o `1`)</span><span class="sxs-lookup"><span data-stu-id="2662b-211">**Type**: *bool* (`true` or `1`)</span></span>  
<span data-ttu-id="2662b-212">**Valore predefinito**: false</span><span class="sxs-lookup"><span data-stu-id="2662b-212">**Default**: false</span></span>  
<span data-ttu-id="2662b-213">**Imposta utilizzando**: `UseSetting`</span><span class="sxs-lookup"><span data-stu-id="2662b-213">**Set using**: `UseSetting`</span></span>  
<span data-ttu-id="2662b-214">**Variabile di ambiente**: `ASPNETCORE_DETAILEDERRORS`</span><span class="sxs-lookup"><span data-stu-id="2662b-214">**Environment variable**: `ASPNETCORE_DETAILEDERRORS`</span></span>

<span data-ttu-id="2662b-215">Quando è abilitata (o quando l'<a href="#environment">ambiente</a> è impostato su `Development`), l'app acquisisce le eccezioni dettagliate.</span><span class="sxs-lookup"><span data-stu-id="2662b-215">When enabled (or when the <a href="#environment">Environment</a> is set to `Development`), the app captures detailed exceptions.</span></span>

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseSetting(WebHostDefaults.DetailedErrorsKey, "true")
```

### <a name="environment"></a><span data-ttu-id="2662b-216">Ambiente</span><span class="sxs-lookup"><span data-stu-id="2662b-216">Environment</span></span>

<span data-ttu-id="2662b-217">Imposta l'ambiente dell'app.</span><span class="sxs-lookup"><span data-stu-id="2662b-217">Sets the app's environment.</span></span>

<span data-ttu-id="2662b-218">**Chiave**: environment</span><span class="sxs-lookup"><span data-stu-id="2662b-218">**Key**: environment</span></span>  
<span data-ttu-id="2662b-219">**Tipo**: *stringa*</span><span class="sxs-lookup"><span data-stu-id="2662b-219">**Type**: *string*</span></span>  
<span data-ttu-id="2662b-220">**Impostazione predefinita**: Production</span><span class="sxs-lookup"><span data-stu-id="2662b-220">**Default**: Production</span></span>  
<span data-ttu-id="2662b-221">**Imposta utilizzando**: `UseEnvironment`</span><span class="sxs-lookup"><span data-stu-id="2662b-221">**Set using**: `UseEnvironment`</span></span>  
<span data-ttu-id="2662b-222">**Variabile di ambiente**: `ASPNETCORE_ENVIRONMENT`</span><span class="sxs-lookup"><span data-stu-id="2662b-222">**Environment variable**: `ASPNETCORE_ENVIRONMENT`</span></span>

<span data-ttu-id="2662b-223">L'ambiente può essere impostato su qualsiasi valore.</span><span class="sxs-lookup"><span data-stu-id="2662b-223">The environment can be set to any value.</span></span> <span data-ttu-id="2662b-224">I valori definiti dal framework includono `Development`, `Staging` e `Production`.</span><span class="sxs-lookup"><span data-stu-id="2662b-224">Framework-defined values include `Development`, `Staging`, and `Production`.</span></span> <span data-ttu-id="2662b-225">Nei valori non viene fatta distinzione tra maiuscole e minuscole.</span><span class="sxs-lookup"><span data-stu-id="2662b-225">Values aren't case sensitive.</span></span> <span data-ttu-id="2662b-226">Per impostazione predefinita, l'*ambiente* viene letto dalla variabile di ambiente `ASPNETCORE_ENVIRONMENT`.</span><span class="sxs-lookup"><span data-stu-id="2662b-226">By default, the *Environment* is read from the `ASPNETCORE_ENVIRONMENT` environment variable.</span></span> <span data-ttu-id="2662b-227">Quando si usa [Visual Studio](https://visualstudio.microsoft.com), le variabili di ambiente possono essere impostate nel file *launchSettings.json*.</span><span class="sxs-lookup"><span data-stu-id="2662b-227">When using [Visual Studio](https://visualstudio.microsoft.com), environment variables may be set in the *launchSettings.json* file.</span></span> <span data-ttu-id="2662b-228">Per altre informazioni, vedere <xref:fundamentals/environments>.</span><span class="sxs-lookup"><span data-stu-id="2662b-228">For more information, see <xref:fundamentals/environments>.</span></span>

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseEnvironment(EnvironmentName.Development)
```

### <a name="hosting-startup-assemblies"></a><span data-ttu-id="2662b-229">Assembly di avvio dell'hosting</span><span class="sxs-lookup"><span data-stu-id="2662b-229">Hosting Startup Assemblies</span></span>

<span data-ttu-id="2662b-230">Imposta gli assembly di avvio dell'hosting dell'app.</span><span class="sxs-lookup"><span data-stu-id="2662b-230">Sets the app's hosting startup assemblies.</span></span>

<span data-ttu-id="2662b-231">**Chiave**: hostingStartupAssemblies</span><span class="sxs-lookup"><span data-stu-id="2662b-231">**Key**: hostingStartupAssemblies</span></span>  
<span data-ttu-id="2662b-232">**Tipo**: *stringa*</span><span class="sxs-lookup"><span data-stu-id="2662b-232">**Type**: *string*</span></span>  
<span data-ttu-id="2662b-233">**Impostazione predefinita**: stringa vuota</span><span class="sxs-lookup"><span data-stu-id="2662b-233">**Default**: Empty string</span></span>  
<span data-ttu-id="2662b-234">**Imposta utilizzando**: `UseSetting`</span><span class="sxs-lookup"><span data-stu-id="2662b-234">**Set using**: `UseSetting`</span></span>  
<span data-ttu-id="2662b-235">**Variabile di ambiente**: `ASPNETCORE_HOSTINGSTARTUPASSEMBLIES`</span><span class="sxs-lookup"><span data-stu-id="2662b-235">**Environment variable**: `ASPNETCORE_HOSTINGSTARTUPASSEMBLIES`</span></span>

<span data-ttu-id="2662b-236">Una stringa delimitata da punto e virgola di assembly di avvio dell'hosting da caricare all'avvio.</span><span class="sxs-lookup"><span data-stu-id="2662b-236">A semicolon-delimited string of hosting startup assemblies to load on startup.</span></span>

<span data-ttu-id="2662b-237">Sebbene il valore di configurazione predefinito sia una stringa vuota, gli assembly di avvio dell'hosting includono sempre l'assembly dell'app.</span><span class="sxs-lookup"><span data-stu-id="2662b-237">Although the configuration value defaults to an empty string, the hosting startup assemblies always include the app's assembly.</span></span> <span data-ttu-id="2662b-238">Quando vengono specificati, gli assembly di avvio dell'hosting vengono aggiunti all'assembly dell'app per essere caricati quando l'app compila i servizi comuni durante l'avvio.</span><span class="sxs-lookup"><span data-stu-id="2662b-238">When hosting startup assemblies are provided, they're added to the app's assembly for loading when the app builds its common services during startup.</span></span>

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseSetting(WebHostDefaults.HostingStartupAssembliesKey, "assembly1;assembly2")
```

### <a name="https-port"></a><span data-ttu-id="2662b-239">Porta HTTPS</span><span class="sxs-lookup"><span data-stu-id="2662b-239">HTTPS Port</span></span>

<span data-ttu-id="2662b-240">Impostare la porta di reindirizzamento HTTPS.</span><span class="sxs-lookup"><span data-stu-id="2662b-240">Set the HTTPS redirect port.</span></span> <span data-ttu-id="2662b-241">Usata per [imporre HTTPS](xref:security/enforcing-ssl).</span><span class="sxs-lookup"><span data-stu-id="2662b-241">Used in [enforcing HTTPS](xref:security/enforcing-ssl).</span></span>

<span data-ttu-id="2662b-242">**Chiave**: https_port **Tipo**: *string*
**Valore predefinito**: non è impostato un valore predefinito.</span><span class="sxs-lookup"><span data-stu-id="2662b-242">**Key**: https_port **Type**: *string*
**Default**: A default value isn't set.</span></span>
<span data-ttu-id="2662b-243">**Imposta utilizzando**: `UseSetting` 
 **variabile di ambiente**:`ASPNETCORE_HTTPS_PORT`</span><span class="sxs-lookup"><span data-stu-id="2662b-243">**Set using**: `UseSetting`
**Environment variable**: `ASPNETCORE_HTTPS_PORT`</span></span>

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseSetting("https_port", "8080")
```

### <a name="hosting-startup-exclude-assemblies"></a><span data-ttu-id="2662b-244">Assembly di avvio dell'hosting da escludere</span><span class="sxs-lookup"><span data-stu-id="2662b-244">Hosting Startup Exclude Assemblies</span></span>

<span data-ttu-id="2662b-245">Una stringa delimitata da punto e virgola di assembly di avvio dell'hosting da escludere all'avvio.</span><span class="sxs-lookup"><span data-stu-id="2662b-245">A semicolon-delimited string of hosting startup assemblies to exclude on startup.</span></span>

<span data-ttu-id="2662b-246">**Chiave**: hostingStartupExcludeAssemblies</span><span class="sxs-lookup"><span data-stu-id="2662b-246">**Key**: hostingStartupExcludeAssemblies</span></span>  
<span data-ttu-id="2662b-247">**Tipo**: *stringa*</span><span class="sxs-lookup"><span data-stu-id="2662b-247">**Type**: *string*</span></span>  
<span data-ttu-id="2662b-248">**Impostazione predefinita**: stringa vuota</span><span class="sxs-lookup"><span data-stu-id="2662b-248">**Default**: Empty string</span></span>  
<span data-ttu-id="2662b-249">**Imposta utilizzando**: `UseSetting`</span><span class="sxs-lookup"><span data-stu-id="2662b-249">**Set using**: `UseSetting`</span></span>  
<span data-ttu-id="2662b-250">**Variabile di ambiente**: `ASPNETCORE_HOSTINGSTARTUPEXCLUDEASSEMBLIES`</span><span class="sxs-lookup"><span data-stu-id="2662b-250">**Environment variable**: `ASPNETCORE_HOSTINGSTARTUPEXCLUDEASSEMBLIES`</span></span>

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseSetting(WebHostDefaults.HostingStartupExcludeAssembliesKey, "assembly1;assembly2")
```

### <a name="prefer-hosting-urls"></a><span data-ttu-id="2662b-251">Preferire gli URL di hosting</span><span class="sxs-lookup"><span data-stu-id="2662b-251">Prefer Hosting URLs</span></span>

<span data-ttu-id="2662b-252">Indica se l'host deve eseguire l'ascolto sugli URL configurati con `WebHostBuilder` anziché su quelli configurati con l'implementazione `IServer`.</span><span class="sxs-lookup"><span data-stu-id="2662b-252">Indicates whether the host should listen on the URLs configured with the `WebHostBuilder` instead of those configured with the `IServer` implementation.</span></span>

<span data-ttu-id="2662b-253">**Chiave**: preferHostingUrls</span><span class="sxs-lookup"><span data-stu-id="2662b-253">**Key**: preferHostingUrls</span></span>  
<span data-ttu-id="2662b-254">**Tipo**: *bool* (`true` o `1`)</span><span class="sxs-lookup"><span data-stu-id="2662b-254">**Type**: *bool* (`true` or `1`)</span></span>  
<span data-ttu-id="2662b-255">**Impostazione predefinita**: true</span><span class="sxs-lookup"><span data-stu-id="2662b-255">**Default**: true</span></span>  
<span data-ttu-id="2662b-256">**Imposta utilizzando**: `PreferHostingUrls`</span><span class="sxs-lookup"><span data-stu-id="2662b-256">**Set using**: `PreferHostingUrls`</span></span>  
<span data-ttu-id="2662b-257">**Variabile di ambiente**: `ASPNETCORE_PREFERHOSTINGURLS`</span><span class="sxs-lookup"><span data-stu-id="2662b-257">**Environment variable**: `ASPNETCORE_PREFERHOSTINGURLS`</span></span>

```csharp
WebHost.CreateDefaultBuilder(args)
    .PreferHostingUrls(false)
```

### <a name="prevent-hosting-startup"></a><span data-ttu-id="2662b-258">Impedire l'avvio dell'hosting</span><span class="sxs-lookup"><span data-stu-id="2662b-258">Prevent Hosting Startup</span></span>

<span data-ttu-id="2662b-259">Impedisce il caricamento automatico degli assembly di avvio dell'hosting, inclusi gli assembly di avvio dell'hosting configurati dall'assembly dell'app.</span><span class="sxs-lookup"><span data-stu-id="2662b-259">Prevents the automatic loading of hosting startup assemblies, including hosting startup assemblies configured by the app's assembly.</span></span> <span data-ttu-id="2662b-260">Per altre informazioni, vedere <xref:fundamentals/configuration/platform-specific-configuration>.</span><span class="sxs-lookup"><span data-stu-id="2662b-260">For more information, see <xref:fundamentals/configuration/platform-specific-configuration>.</span></span>

<span data-ttu-id="2662b-261">**Chiave**: preventHostingStartup</span><span class="sxs-lookup"><span data-stu-id="2662b-261">**Key**: preventHostingStartup</span></span>  
<span data-ttu-id="2662b-262">**Tipo**: *bool* (`true` o `1`)</span><span class="sxs-lookup"><span data-stu-id="2662b-262">**Type**: *bool* (`true` or `1`)</span></span>  
<span data-ttu-id="2662b-263">**Valore predefinito**: false</span><span class="sxs-lookup"><span data-stu-id="2662b-263">**Default**: false</span></span>  
<span data-ttu-id="2662b-264">**Imposta utilizzando**: `UseSetting`</span><span class="sxs-lookup"><span data-stu-id="2662b-264">**Set using**: `UseSetting`</span></span>  
<span data-ttu-id="2662b-265">**Variabile di ambiente**: `ASPNETCORE_PREVENTHOSTINGSTARTUP`</span><span class="sxs-lookup"><span data-stu-id="2662b-265">**Environment variable**: `ASPNETCORE_PREVENTHOSTINGSTARTUP`</span></span>

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseSetting(WebHostDefaults.PreventHostingStartupKey, "true")
```

### <a name="server-urls"></a><span data-ttu-id="2662b-266">URL del server</span><span class="sxs-lookup"><span data-stu-id="2662b-266">Server URLs</span></span>

<span data-ttu-id="2662b-267">Indica gli indirizzi IP o gli indirizzi host con le porte e protocolli su cui il server deve eseguire l'ascolto per le richieste.</span><span class="sxs-lookup"><span data-stu-id="2662b-267">Indicates the IP addresses or host addresses with ports and protocols that the server should listen on for requests.</span></span>

<span data-ttu-id="2662b-268">**Chiave**: urls</span><span class="sxs-lookup"><span data-stu-id="2662b-268">**Key**: urls</span></span>  
<span data-ttu-id="2662b-269">**Tipo**: *stringa*</span><span class="sxs-lookup"><span data-stu-id="2662b-269">**Type**: *string*</span></span>  
<span data-ttu-id="2662b-270">**Impostazione predefinita**: http://localhost:5000</span><span class="sxs-lookup"><span data-stu-id="2662b-270">**Default**: http://localhost:5000</span></span>  
<span data-ttu-id="2662b-271">**Imposta utilizzando**: `UseUrls`</span><span class="sxs-lookup"><span data-stu-id="2662b-271">**Set using**: `UseUrls`</span></span>  
<span data-ttu-id="2662b-272">**Variabile di ambiente**: `ASPNETCORE_URLS`</span><span class="sxs-lookup"><span data-stu-id="2662b-272">**Environment variable**: `ASPNETCORE_URLS`</span></span>

<span data-ttu-id="2662b-273">Impostare su un elenco di prefissi URL separati da punto e virgola (;) ai quali il server deve rispondere.</span><span class="sxs-lookup"><span data-stu-id="2662b-273">Set to a semicolon-separated (;) list of URL prefixes to which the server should respond.</span></span> <span data-ttu-id="2662b-274">Ad esempio: `http://localhost:123`.</span><span class="sxs-lookup"><span data-stu-id="2662b-274">For example, `http://localhost:123`.</span></span> <span data-ttu-id="2662b-275">Usare "\*" per indicare che il server deve eseguire l'ascolto per le richieste su tutti gli indirizzi IP o nomi host usando la porta e il protocollo specificati (ad esempio, `http://*:5000`).</span><span class="sxs-lookup"><span data-stu-id="2662b-275">Use "\*" to indicate that the server should listen for requests on any IP address or hostname using the specified port and protocol (for example, `http://*:5000`).</span></span> <span data-ttu-id="2662b-276">Il protocollo (`http://` o `https://`) deve essere incluso con ogni URL.</span><span class="sxs-lookup"><span data-stu-id="2662b-276">The protocol (`http://` or `https://`) must be included with each URL.</span></span> <span data-ttu-id="2662b-277">I formati supportati variano a seconda del server.</span><span class="sxs-lookup"><span data-stu-id="2662b-277">Supported formats vary among servers.</span></span>

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseUrls("http://*:5000;http://localhost:5001;https://hostname:5002")
```

::: moniker range=">= aspnetcore-5.0"
<span data-ttu-id="2662b-278">Kestrel ha una propria API di configurazione degli endpoint.</span><span class="sxs-lookup"><span data-stu-id="2662b-278">Kestrel has its own endpoint configuration API.</span></span> <span data-ttu-id="2662b-279">Per altre informazioni, vedere <xref:fundamentals/servers/kestrel/endpoints>.</span><span class="sxs-lookup"><span data-stu-id="2662b-279">For more information, see <xref:fundamentals/servers/kestrel/endpoints>.</span></span>
::: moniker-end
::: moniker range="< aspnetcore-5.0"
<span data-ttu-id="2662b-280">Kestrel ha una propria API di configurazione degli endpoint.</span><span class="sxs-lookup"><span data-stu-id="2662b-280">Kestrel has its own endpoint configuration API.</span></span> <span data-ttu-id="2662b-281">Per altre informazioni, vedere <xref:fundamentals/servers/kestrel#endpoint-configuration>.</span><span class="sxs-lookup"><span data-stu-id="2662b-281">For more information, see <xref:fundamentals/servers/kestrel#endpoint-configuration>.</span></span>
::: moniker-end

### <a name="shutdown-timeout"></a><span data-ttu-id="2662b-282">Timeout di arresto</span><span class="sxs-lookup"><span data-stu-id="2662b-282">Shutdown Timeout</span></span>

<span data-ttu-id="2662b-283">Specifica il tempo di attesa per l'arresto dell'host Web.</span><span class="sxs-lookup"><span data-stu-id="2662b-283">Specifies the amount of time to wait for Web Host to shut down.</span></span>

<span data-ttu-id="2662b-284">**Chiave**: shutdownTimeoutSeconds</span><span class="sxs-lookup"><span data-stu-id="2662b-284">**Key**: shutdownTimeoutSeconds</span></span>  
<span data-ttu-id="2662b-285">**Tipo**: *int*</span><span class="sxs-lookup"><span data-stu-id="2662b-285">**Type**: *int*</span></span>  
<span data-ttu-id="2662b-286">**Valore predefinito**: 5</span><span class="sxs-lookup"><span data-stu-id="2662b-286">**Default**: 5</span></span>  
<span data-ttu-id="2662b-287">**Imposta utilizzando**: `UseShutdownTimeout`</span><span class="sxs-lookup"><span data-stu-id="2662b-287">**Set using**: `UseShutdownTimeout`</span></span>  
<span data-ttu-id="2662b-288">**Variabile di ambiente**: `ASPNETCORE_SHUTDOWNTIMEOUTSECONDS`</span><span class="sxs-lookup"><span data-stu-id="2662b-288">**Environment variable**: `ASPNETCORE_SHUTDOWNTIMEOUTSECONDS`</span></span>

<span data-ttu-id="2662b-289">Sebbene la chiave accetti *int* con `UseSetting` (ad esempio, `.UseSetting(WebHostDefaults.ShutdownTimeoutKey, "10")`), il metodo di estensione [UseShutdownTimeout](/dotnet/api/microsoft.aspnetcore.hosting.hostingabstractionswebhostbuilderextensions.useshutdowntimeout) accetta [TimeSpan](/dotnet/api/system.timespan).</span><span class="sxs-lookup"><span data-stu-id="2662b-289">Although the key accepts an *int* with `UseSetting` (for example, `.UseSetting(WebHostDefaults.ShutdownTimeoutKey, "10")`), the [UseShutdownTimeout](/dotnet/api/microsoft.aspnetcore.hosting.hostingabstractionswebhostbuilderextensions.useshutdowntimeout) extension method takes a [TimeSpan](/dotnet/api/system.timespan).</span></span>

<span data-ttu-id="2662b-290">Durante il periodo di timeout, l'hosting:</span><span class="sxs-lookup"><span data-stu-id="2662b-290">During the timeout period, hosting:</span></span>

* <span data-ttu-id="2662b-291">Attiva [IApplicationLifetime.ApplicationStopping](/dotnet/api/microsoft.aspnetcore.hosting.iapplicationlifetime.applicationstopping).</span><span class="sxs-lookup"><span data-stu-id="2662b-291">Triggers [IApplicationLifetime.ApplicationStopping](/dotnet/api/microsoft.aspnetcore.hosting.iapplicationlifetime.applicationstopping).</span></span>
* <span data-ttu-id="2662b-292">Tenta di arrestare i servizi ospitati, registrando eventuali errori per i servizi che non si interrompono.</span><span class="sxs-lookup"><span data-stu-id="2662b-292">Attempts to stop hosted services, logging any errors for services that fail to stop.</span></span>

<span data-ttu-id="2662b-293">Se il periodo di timeout scade prima che siano stati arrestati tutti i servizi ospitati, gli eventuali servizi attivi rimanenti vengono interrotti quando l'app viene arrestata.</span><span class="sxs-lookup"><span data-stu-id="2662b-293">If the timeout period expires before all of the hosted services stop, any remaining active services are stopped when the app shuts down.</span></span> <span data-ttu-id="2662b-294">I servizi si arrestano anche se non hanno completato l'elaborazione.</span><span class="sxs-lookup"><span data-stu-id="2662b-294">The services stop even if they haven't finished processing.</span></span> <span data-ttu-id="2662b-295">Se l'arresto dei servizi richiede più tempo, aumentare il valore di timeout.</span><span class="sxs-lookup"><span data-stu-id="2662b-295">If services require additional time to stop, increase the timeout.</span></span>

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseShutdownTimeout(TimeSpan.FromSeconds(10))
```

### <a name="startup-assembly"></a><span data-ttu-id="2662b-296">Assembly di avvio</span><span class="sxs-lookup"><span data-stu-id="2662b-296">Startup Assembly</span></span>

<span data-ttu-id="2662b-297">Determina l'assembly per la ricerca della classe `Startup`.</span><span class="sxs-lookup"><span data-stu-id="2662b-297">Determines the assembly to search for the `Startup` class.</span></span>

<span data-ttu-id="2662b-298">**Chiave**: startupAssembly</span><span class="sxs-lookup"><span data-stu-id="2662b-298">**Key**: startupAssembly</span></span>  
<span data-ttu-id="2662b-299">**Tipo**: *stringa*</span><span class="sxs-lookup"><span data-stu-id="2662b-299">**Type**: *string*</span></span>  
<span data-ttu-id="2662b-300">**Impostazione predefinita**: assembly dell'app</span><span class="sxs-lookup"><span data-stu-id="2662b-300">**Default**: The app's assembly</span></span>  
<span data-ttu-id="2662b-301">**Imposta utilizzando**: `UseStartup`</span><span class="sxs-lookup"><span data-stu-id="2662b-301">**Set using**: `UseStartup`</span></span>  
<span data-ttu-id="2662b-302">**Variabile di ambiente**: `ASPNETCORE_STARTUPASSEMBLY`</span><span class="sxs-lookup"><span data-stu-id="2662b-302">**Environment variable**: `ASPNETCORE_STARTUPASSEMBLY`</span></span>

<span data-ttu-id="2662b-303">È possibile fare riferimento all'assembly per nome (`string`) o per tipo (`TStartup`).</span><span class="sxs-lookup"><span data-stu-id="2662b-303">The assembly by name (`string`) or type (`TStartup`) can be referenced.</span></span> <span data-ttu-id="2662b-304">Se vengono chiamati più metodi `UseStartup`, l'ultimo metodo ha la precedenza.</span><span class="sxs-lookup"><span data-stu-id="2662b-304">If multiple `UseStartup` methods are called, the last one takes precedence.</span></span>

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseStartup("StartupAssemblyName")
```

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseStartup<TStartup>()
```

### <a name="web-root"></a><span data-ttu-id="2662b-305">Radice Web</span><span class="sxs-lookup"><span data-stu-id="2662b-305">Web root</span></span>

<span data-ttu-id="2662b-306">Imposta il percorso relativo degli asset statici dell'app.</span><span class="sxs-lookup"><span data-stu-id="2662b-306">Sets the relative path to the app's static assets.</span></span>

<span data-ttu-id="2662b-307">**Chiave**: webroot</span><span class="sxs-lookup"><span data-stu-id="2662b-307">**Key**: webroot</span></span>  
<span data-ttu-id="2662b-308">**Tipo**: *stringa*</span><span class="sxs-lookup"><span data-stu-id="2662b-308">**Type**: *string*</span></span>  
<span data-ttu-id="2662b-309">**Impostazione predefinita**: il valore predefinito è `wwwroot` .</span><span class="sxs-lookup"><span data-stu-id="2662b-309">**Default**: The default is `wwwroot`.</span></span> <span data-ttu-id="2662b-310">Il percorso di *{radice del contenuto}/wwwroot* deve esistere.</span><span class="sxs-lookup"><span data-stu-id="2662b-310">The path to *{content root}/wwwroot* must exist.</span></span> <span data-ttu-id="2662b-311">Se il percorso non esiste, viene usato un provider di file no-op.</span><span class="sxs-lookup"><span data-stu-id="2662b-311">If the path doesn't exist, a no-op file provider is used.</span></span>  
<span data-ttu-id="2662b-312">**Imposta utilizzando**: `UseWebRoot`</span><span class="sxs-lookup"><span data-stu-id="2662b-312">**Set using**: `UseWebRoot`</span></span>  
<span data-ttu-id="2662b-313">**Variabile di ambiente**: `ASPNETCORE_WEBROOT`</span><span class="sxs-lookup"><span data-stu-id="2662b-313">**Environment variable**: `ASPNETCORE_WEBROOT`</span></span>

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseWebRoot("public")
```

<span data-ttu-id="2662b-314">Per altre informazioni, vedere:</span><span class="sxs-lookup"><span data-stu-id="2662b-314">For more information, see:</span></span>

* [<span data-ttu-id="2662b-315">Nozioni fondamentali: radice Web</span><span class="sxs-lookup"><span data-stu-id="2662b-315">Fundamentals: Web root</span></span>](xref:fundamentals/index#web-root)
* [<span data-ttu-id="2662b-316">Radice del contenuto</span><span class="sxs-lookup"><span data-stu-id="2662b-316">Content root</span></span>](#content-root)

## <a name="override-configuration"></a><span data-ttu-id="2662b-317">Override della configurazione</span><span class="sxs-lookup"><span data-stu-id="2662b-317">Override configuration</span></span>

<span data-ttu-id="2662b-318">Usare la [Configurazione](xref:fundamentals/configuration/index) per configurare l'host Web.</span><span class="sxs-lookup"><span data-stu-id="2662b-318">Use [Configuration](xref:fundamentals/configuration/index) to configure Web Host.</span></span> <span data-ttu-id="2662b-319">Nell'esempio seguente la configurazione dell'host è specificata facoltativamente nel file *hostsettings.json*.</span><span class="sxs-lookup"><span data-stu-id="2662b-319">In the following example, host configuration is optionally specified in a *hostsettings.json* file.</span></span> <span data-ttu-id="2662b-320">Qualsiasi configurazione caricata dal file *hostsettings.json* può essere sostituita dagli argomenti della riga di comando.</span><span class="sxs-lookup"><span data-stu-id="2662b-320">Any configuration loaded from the *hostsettings.json* file may be overridden by command-line arguments.</span></span> <span data-ttu-id="2662b-321">La configurazione compilata (in `config`) viene usata per configurare l'host con [UseConfiguration](/dotnet/api/microsoft.aspnetcore.hosting.hostingabstractionswebhostbuilderextensions.useconfiguration).</span><span class="sxs-lookup"><span data-stu-id="2662b-321">The built configuration (in `config`) is used to configure the host with [UseConfiguration](/dotnet/api/microsoft.aspnetcore.hosting.hostingabstractionswebhostbuilderextensions.useconfiguration).</span></span> <span data-ttu-id="2662b-322">La configurazione di `IWebHostBuilder` viene aggiunta alla configurazione dell'app, ma non il contrario: &mdash;`ConfigureAppConfiguration` non influenza la configurazione di `IWebHostBuilder`.</span><span class="sxs-lookup"><span data-stu-id="2662b-322">`IWebHostBuilder` configuration is added to the app's configuration, but the converse isn't true&mdash;`ConfigureAppConfiguration` doesn't affect the `IWebHostBuilder` configuration.</span></span>

<span data-ttu-id="2662b-323">Esecuzione dell'override della configurazione specificata da `UseUrls` con prima la configurazione *hostsettings.json* e quindi con la configurazione dell'argomento della riga di comando:</span><span class="sxs-lookup"><span data-stu-id="2662b-323">Overriding the configuration provided by `UseUrls` with *hostsettings.json* config first, command-line argument config second:</span></span>

```csharp
public class Program
{
    public static void Main(string[] args)
    {
        CreateWebHostBuilder(args).Build().Run();
    }

    public static IWebHostBuilder CreateWebHostBuilder(string[] args)
    {
        var config = new ConfigurationBuilder()
            .SetBasePath(Directory.GetCurrentDirectory())
            .AddJsonFile("hostsettings.json", optional: true)
            .AddCommandLine(args)
            .Build();

        return WebHost.CreateDefaultBuilder(args)
            .UseUrls("http://*:5000")
            .UseConfiguration(config)
            .Configure(app =>
            {
                app.Run(context => 
                    context.Response.WriteAsync("Hello, World!"));
            });
    }
}
```

<span data-ttu-id="2662b-324">*hostsettings.json*:</span><span class="sxs-lookup"><span data-stu-id="2662b-324">*hostsettings.json*:</span></span>

```json
{
    urls: "http://*:5005"
}
```

> [!NOTE]
> <span data-ttu-id="2662b-325">[UseConfiguration](/dotnet/api/microsoft.aspnetcore.hosting.hostingabstractionswebhostbuilderextensions.useconfiguration) copia solo le chiavi dall'oggetto fornito `IConfiguration` alla configurazione del generatore host.</span><span class="sxs-lookup"><span data-stu-id="2662b-325">[UseConfiguration](/dotnet/api/microsoft.aspnetcore.hosting.hostingabstractionswebhostbuilderextensions.useconfiguration) only copies keys from the provided `IConfiguration` to the host builder configuration.</span></span> <span data-ttu-id="2662b-326">Pertanto, l'impostazione di `reloadOnChange: true` per i file JSON, INI e XML non ha alcun effetto.</span><span class="sxs-lookup"><span data-stu-id="2662b-326">Therefore, setting `reloadOnChange: true` for JSON, INI, and XML settings files has no effect.</span></span>

<span data-ttu-id="2662b-327">Per specificare l'host eseguito in un URL specifico, il valore desiderato può essere passato da un prompt dei comandi durante l'esecuzione di [dotnet run](/dotnet/core/tools/dotnet-run).</span><span class="sxs-lookup"><span data-stu-id="2662b-327">To specify the host run on a particular URL, the desired value can be passed in from a command prompt when executing [dotnet run](/dotnet/core/tools/dotnet-run).</span></span> <span data-ttu-id="2662b-328">L'argomento della riga di comando esegue l'override del valore `urls` dal file *hostsettings.json*, mentre il server esegue l'ascolto sulla porta 8080:</span><span class="sxs-lookup"><span data-stu-id="2662b-328">The command-line argument overrides the `urls` value from the *hostsettings.json* file, and the server listens on port 8080:</span></span>

```dotnetcli
dotnet run --urls "http://*:8080"
```

## <a name="manage-the-host"></a><span data-ttu-id="2662b-329">Gestire l'host</span><span class="sxs-lookup"><span data-stu-id="2662b-329">Manage the host</span></span>

<span data-ttu-id="2662b-330">**Esegui**</span><span class="sxs-lookup"><span data-stu-id="2662b-330">**Run**</span></span>

<span data-ttu-id="2662b-331">Il metodo `Run` avvia l'app Web e blocca il thread di chiamata fino all'arresto dell'host:</span><span class="sxs-lookup"><span data-stu-id="2662b-331">The `Run` method starts the web app and blocks the calling thread until the host is shut down:</span></span>

```csharp
host.Run();
```

<span data-ttu-id="2662b-332">**Inizia**</span><span class="sxs-lookup"><span data-stu-id="2662b-332">**Start**</span></span>

<span data-ttu-id="2662b-333">Eseguire l'host in modo non bloccante chiamandone il metodo `Start`:</span><span class="sxs-lookup"><span data-stu-id="2662b-333">Run the host in a non-blocking manner by calling its `Start` method:</span></span>

```csharp
using (host)
{
    host.Start();
    Console.ReadLine();
}
```

<span data-ttu-id="2662b-334">Se viene passato un elenco di URL al metodo `Start`, viene eseguito l'ascolto sugli URL specificati:</span><span class="sxs-lookup"><span data-stu-id="2662b-334">If a list of URLs is passed to the `Start` method, it listens on the URLs specified:</span></span>

```csharp
var urls = new List<string>()
{
    "http://*:5000",
    "http://localhost:5001"
};

var host = new WebHostBuilder()
    .UseKestrel()
    .UseStartup<Startup>()
    .Start(urls.ToArray());

using (host)
{
    Console.ReadLine();
}
```

<span data-ttu-id="2662b-335">L'app può inizializzare e avviare un nuovo host usando i valori predefiniti preconfigurati di `CreateDefaultBuilder` con un metodo pratico statico.</span><span class="sxs-lookup"><span data-stu-id="2662b-335">The app can initialize and start a new host using the pre-configured defaults of `CreateDefaultBuilder` using a static convenience method.</span></span> <span data-ttu-id="2662b-336">Questi metodi avviano il server senza l'output della console e con l'attesa [WaitForShutdown](/dotnet/api/microsoft.aspnetcore.hosting.webhostextensions.waitforshutdown) di un'interruzione (Ctrl-C/SIGINT o SIGTERM):</span><span class="sxs-lookup"><span data-stu-id="2662b-336">These methods start the server without console output and with [WaitForShutdown](/dotnet/api/microsoft.aspnetcore.hosting.webhostextensions.waitforshutdown) wait for a break (Ctrl-C/SIGINT or SIGTERM):</span></span>

<span data-ttu-id="2662b-337">**Start(RequestDelegate app)**</span><span class="sxs-lookup"><span data-stu-id="2662b-337">**Start(RequestDelegate app)**</span></span>

<span data-ttu-id="2662b-338">Start con `RequestDelegate`:</span><span class="sxs-lookup"><span data-stu-id="2662b-338">Start with a `RequestDelegate`:</span></span>

```csharp
using (var host = WebHost.Start(app => app.Response.WriteAsync("Hello, World!")))
{
    Console.WriteLine("Use Ctrl-C to shutdown the host...");
    host.WaitForShutdown();
}
```

<span data-ttu-id="2662b-339">Creare una richiesta nel browser a `http://localhost:5000` per ricevere la risposta "Hello World!"</span><span class="sxs-lookup"><span data-stu-id="2662b-339">Make a request in the browser to `http://localhost:5000` to receive the response "Hello World!"</span></span> <span data-ttu-id="2662b-340">`WaitForShutdown` rimane bloccato fino a quando non viene eseguita un'interruzione (Ctrl-C/SIGINT o SIGTERM).</span><span class="sxs-lookup"><span data-stu-id="2662b-340">`WaitForShutdown` blocks until a break (Ctrl-C/SIGINT or SIGTERM) is issued.</span></span> <span data-ttu-id="2662b-341">L'app visualizza il messaggio `Console.WriteLine` e attende la pressione di un tasto per chiudersi.</span><span class="sxs-lookup"><span data-stu-id="2662b-341">The app displays the `Console.WriteLine` message and waits for a keypress to exit.</span></span>

<span data-ttu-id="2662b-342">**Start(string url, RequestDelegate app)**</span><span class="sxs-lookup"><span data-stu-id="2662b-342">**Start(string url, RequestDelegate app)**</span></span>

<span data-ttu-id="2662b-343">Start con un URL e `RequestDelegate`:</span><span class="sxs-lookup"><span data-stu-id="2662b-343">Start with a URL and `RequestDelegate`:</span></span>

```csharp
using (var host = WebHost.Start("http://localhost:8080", app => app.Response.WriteAsync("Hello, World!")))
{
    Console.WriteLine("Use Ctrl-C to shutdown the host...");
    host.WaitForShutdown();
}
```

<span data-ttu-id="2662b-344">Produce lo stesso risultato di **Start(app RequestDelegate)**, ad eccezione del fatto che l'app risponde su `http://localhost:8080`.</span><span class="sxs-lookup"><span data-stu-id="2662b-344">Produces the same result as **Start(RequestDelegate app)**, except the app responds on `http://localhost:8080`.</span></span>

<span data-ttu-id="2662b-345">**Start(Action\<IRouteBuilder> routeBuilder)**</span><span class="sxs-lookup"><span data-stu-id="2662b-345">**Start(Action\<IRouteBuilder> routeBuilder)**</span></span>

<span data-ttu-id="2662b-346">Usare un'istanza di `IRouteBuilder` ([Microsoft.AspNetCore.Routing](https://www.nuget.org/packages/Microsoft.AspNetCore.Routing/)) per usare il middleware di routing:</span><span class="sxs-lookup"><span data-stu-id="2662b-346">Use an instance of `IRouteBuilder` ([Microsoft.AspNetCore.Routing](https://www.nuget.org/packages/Microsoft.AspNetCore.Routing/)) to use routing middleware:</span></span>

```csharp
using (var host = WebHost.Start(router => router
    .MapGet("hello/{name}", (req, res, data) => 
        res.WriteAsync($"Hello, {data.Values["name"]}!"))
    .MapGet("buenosdias/{name}", (req, res, data) => 
        res.WriteAsync($"Buenos dias, {data.Values["name"]}!"))
    .MapGet("throw/{message?}", (req, res, data) => 
        throw new Exception((string)data.Values["message"] ?? "Uh oh!"))
    .MapGet("{greeting}/{name}", (req, res, data) => 
        res.WriteAsync($"{data.Values["greeting"]}, {data.Values["name"]}!"))
    .MapGet("", (req, res, data) => res.WriteAsync("Hello, World!"))))
{
    Console.WriteLine("Use Ctrl-C to shutdown the host...");
    host.WaitForShutdown();
}
```

<span data-ttu-id="2662b-347">Usare le richieste del browser seguenti con l'esempio:</span><span class="sxs-lookup"><span data-stu-id="2662b-347">Use the following browser requests with the example:</span></span>

| <span data-ttu-id="2662b-348">Richiesta</span><span class="sxs-lookup"><span data-stu-id="2662b-348">Request</span></span>                                    | <span data-ttu-id="2662b-349">Risposta</span><span class="sxs-lookup"><span data-stu-id="2662b-349">Response</span></span>                                 |
| ------------------------------------------ | ---------------------------------------- |
| `http://localhost:5000/hello/Martin`       | <span data-ttu-id="2662b-350">Hello, Martin!</span><span class="sxs-lookup"><span data-stu-id="2662b-350">Hello, Martin!</span></span>                           |
| `http://localhost:5000/buenosdias/Catrina` | <span data-ttu-id="2662b-351">Buenos dias, Catrina!</span><span class="sxs-lookup"><span data-stu-id="2662b-351">Buenos dias, Catrina!</span></span>                    |
| `http://localhost:5000/throw/ooops!`       | <span data-ttu-id="2662b-352">Genera un'eccezione con la stringa "ooops!"</span><span class="sxs-lookup"><span data-stu-id="2662b-352">Throws an exception with string "ooops!"</span></span> |
| `http://localhost:5000/throw`              | <span data-ttu-id="2662b-353">Genera un'eccezione con la stringa "Uh oh!"</span><span class="sxs-lookup"><span data-stu-id="2662b-353">Throws an exception with string "Uh oh!"</span></span> |
| `http://localhost:5000/Sante/Kevin`        | <span data-ttu-id="2662b-354">Sante, Kevin!</span><span class="sxs-lookup"><span data-stu-id="2662b-354">Sante, Kevin!</span></span>                            |
| `http://localhost:5000`                    | <span data-ttu-id="2662b-355">Hello World!</span><span class="sxs-lookup"><span data-stu-id="2662b-355">Hello World!</span></span>                             |

<span data-ttu-id="2662b-356">`WaitForShutdown` rimane bloccato fino a quando non viene eseguita un'interruzione (Ctrl-C/SIGINT o SIGTERM).</span><span class="sxs-lookup"><span data-stu-id="2662b-356">`WaitForShutdown` blocks until a break (Ctrl-C/SIGINT or SIGTERM) is issued.</span></span> <span data-ttu-id="2662b-357">L'app visualizza il messaggio `Console.WriteLine` e attende la pressione di un tasto per chiudersi.</span><span class="sxs-lookup"><span data-stu-id="2662b-357">The app displays the `Console.WriteLine` message and waits for a keypress to exit.</span></span>

<span data-ttu-id="2662b-358">**Start(string url, Action\<IRouteBuilder> routeBuilder)**</span><span class="sxs-lookup"><span data-stu-id="2662b-358">**Start(string url, Action\<IRouteBuilder> routeBuilder)**</span></span>

<span data-ttu-id="2662b-359">Usare un URL e un'istanza di `IRouteBuilder`:</span><span class="sxs-lookup"><span data-stu-id="2662b-359">Use a URL and an instance of `IRouteBuilder`:</span></span>

```csharp
using (var host = WebHost.Start("http://localhost:8080", router => router
    .MapGet("hello/{name}", (req, res, data) => 
        res.WriteAsync($"Hello, {data.Values["name"]}!"))
    .MapGet("buenosdias/{name}", (req, res, data) => 
        res.WriteAsync($"Buenos dias, {data.Values["name"]}!"))
    .MapGet("throw/{message?}", (req, res, data) => 
        throw new Exception((string)data.Values["message"] ?? "Uh oh!"))
    .MapGet("{greeting}/{name}", (req, res, data) => 
        res.WriteAsync($"{data.Values["greeting"]}, {data.Values["name"]}!"))
    .MapGet("", (req, res, data) => res.WriteAsync("Hello, World!"))))
{
    Console.WriteLine("Use Ctrl-C to shut down the host...");
    host.WaitForShutdown();
}
```

<span data-ttu-id="2662b-360">Produce lo stesso risultato di **Start(Action\<IRouteBuilder> routeBuilder)**, ad eccezione del fatto che l'app risponde in `http://localhost:8080`.</span><span class="sxs-lookup"><span data-stu-id="2662b-360">Produces the same result as **Start(Action\<IRouteBuilder> routeBuilder)**, except the app responds at `http://localhost:8080`.</span></span>

<span data-ttu-id="2662b-361">**StartWith(Action\<IApplicationBuilder> app)**</span><span class="sxs-lookup"><span data-stu-id="2662b-361">**StartWith(Action\<IApplicationBuilder> app)**</span></span>

<span data-ttu-id="2662b-362">Specificare un delegato per configurare `IApplicationBuilder`:</span><span class="sxs-lookup"><span data-stu-id="2662b-362">Provide a delegate to configure an `IApplicationBuilder`:</span></span>

```csharp
using (var host = WebHost.StartWith(app => 
    app.Use(next => 
    {
        return async context => 
        {
            await context.Response.WriteAsync("Hello World!");
        };
    })))
{
    Console.WriteLine("Use Ctrl-C to shut down the host...");
    host.WaitForShutdown();
}
```

<span data-ttu-id="2662b-363">Creare una richiesta nel browser a `http://localhost:5000` per ricevere la risposta "Hello World!"</span><span class="sxs-lookup"><span data-stu-id="2662b-363">Make a request in the browser to `http://localhost:5000` to receive the response "Hello World!"</span></span> <span data-ttu-id="2662b-364">`WaitForShutdown` rimane bloccato fino a quando non viene eseguita un'interruzione (Ctrl-C/SIGINT o SIGTERM).</span><span class="sxs-lookup"><span data-stu-id="2662b-364">`WaitForShutdown` blocks until a break (Ctrl-C/SIGINT or SIGTERM) is issued.</span></span> <span data-ttu-id="2662b-365">L'app visualizza il messaggio `Console.WriteLine` e attende la pressione di un tasto per chiudersi.</span><span class="sxs-lookup"><span data-stu-id="2662b-365">The app displays the `Console.WriteLine` message and waits for a keypress to exit.</span></span>

<span data-ttu-id="2662b-366">**StartWith(string url, Action\<IApplicationBuilder> app)**</span><span class="sxs-lookup"><span data-stu-id="2662b-366">**StartWith(string url, Action\<IApplicationBuilder> app)**</span></span>

<span data-ttu-id="2662b-367">Specificare un URL e un delegato per configurare `IApplicationBuilder`:</span><span class="sxs-lookup"><span data-stu-id="2662b-367">Provide a URL and a delegate to configure an `IApplicationBuilder`:</span></span>

```csharp
using (var host = WebHost.StartWith("http://localhost:8080", app => 
    app.Use(next => 
    {
        return async context => 
        {
            await context.Response.WriteAsync("Hello World!");
        };
    })))
{
    Console.WriteLine("Use Ctrl-C to shut down the host...");
    host.WaitForShutdown();
}
```

<span data-ttu-id="2662b-368">Produce lo stesso risultato di **StartWith(Action\<IApplicationBuilder> app)**, ad eccezione del fatto che l'app risponde su `http://localhost:8080`.</span><span class="sxs-lookup"><span data-stu-id="2662b-368">Produces the same result as **StartWith(Action\<IApplicationBuilder> app)**, except the app responds on `http://localhost:8080`.</span></span>

::: moniker range=">= aspnetcore-3.0"

## <a name="iwebhostenvironment-interface"></a><span data-ttu-id="2662b-369">Interfaccia IWebHostEnvironment</span><span class="sxs-lookup"><span data-stu-id="2662b-369">IWebHostEnvironment interface</span></span>

<span data-ttu-id="2662b-370">L' `IWebHostEnvironment` interfaccia fornisce informazioni sull'ambiente di hosting Web dell'app.</span><span class="sxs-lookup"><span data-stu-id="2662b-370">The `IWebHostEnvironment` interface provides information about the app's web hosting environment.</span></span> <span data-ttu-id="2662b-371">Usare l'[inserimento di un costruttore](xref:fundamentals/dependency-injection) per ottenere `IWebHostEnvironment` per poterne usare le proprietà e i metodi di estensione:</span><span class="sxs-lookup"><span data-stu-id="2662b-371">Use [constructor injection](xref:fundamentals/dependency-injection) to obtain the `IWebHostEnvironment` in order to use its properties and extension methods:</span></span>

```csharp
public class CustomFileReader
{
    private readonly IWebHostEnvironment _env;

    public CustomFileReader(IWebHostEnvironment env)
    {
        _env = env;
    }

    public string ReadFile(string filePath)
    {
        var fileProvider = _env.WebRootFileProvider;
        // Process the file here
    }
}
```

<span data-ttu-id="2662b-372">È possibile usare un [approccio basato su convenzione](xref:fundamentals/environments#environment-based-startup-class-and-methods) per configurare l'app all'avvio in base all'ambiente.</span><span class="sxs-lookup"><span data-stu-id="2662b-372">A [convention-based approach](xref:fundamentals/environments#environment-based-startup-class-and-methods) can be used to configure the app at startup based on the environment.</span></span> <span data-ttu-id="2662b-373">In alternativa, inserire `IWebHostEnvironment` nel costruttore `Startup` per l'utilizzo in `ConfigureServices`:</span><span class="sxs-lookup"><span data-stu-id="2662b-373">Alternatively, inject the `IWebHostEnvironment` into the `Startup` constructor for use in `ConfigureServices`:</span></span>

```csharp
public class Startup
{
    public Startup(IWebHostEnvironment env)
    {
        HostingEnvironment = env;
    }

    public IWebHostEnvironment HostingEnvironment { get; }

    public void ConfigureServices(IServiceCollection services)
    {
        if (HostingEnvironment.IsDevelopment())
        {
            // Development configuration
        }
        else
        {
            // Staging/Production configuration
        }

        var contentRootPath = HostingEnvironment.ContentRootPath;
    }
}
```

> [!NOTE]
> <span data-ttu-id="2662b-374">Oltre al metodo di estensione `IsDevelopment`, `IWebHostEnvironment` offre i metodi `IsStaging`, `IsProduction` e `IsEnvironment(string environmentName)`.</span><span class="sxs-lookup"><span data-stu-id="2662b-374">In addition to the `IsDevelopment` extension method, `IWebHostEnvironment` offers `IsStaging`, `IsProduction`, and `IsEnvironment(string environmentName)` methods.</span></span> <span data-ttu-id="2662b-375">Per altre informazioni, vedere <xref:fundamentals/environments>.</span><span class="sxs-lookup"><span data-stu-id="2662b-375">For more information, see <xref:fundamentals/environments>.</span></span>

<span data-ttu-id="2662b-376">Il servizio `IWebHostEnvironment` può anche essere inserito direttamente nel metodo `Configure` per configurare la pipeline di elaborazione:</span><span class="sxs-lookup"><span data-stu-id="2662b-376">The `IWebHostEnvironment` service can also be injected directly into the `Configure` method for setting up the processing pipeline:</span></span>

```csharp
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    if (env.IsDevelopment())
    {
        // In Development, use the Developer Exception Page
        app.UseDeveloperExceptionPage();
    }
    else
    {
        // In Staging/Production, route exceptions to /error
        app.UseExceptionHandler("/error");
    }

    var contentRootPath = env.ContentRootPath;
}
```

<span data-ttu-id="2662b-377">`IWebHostEnvironment` può essere inserito nel metodo `Invoke` durante la creazione di [middleware](xref:fundamentals/middleware/write) personalizzato:</span><span class="sxs-lookup"><span data-stu-id="2662b-377">`IWebHostEnvironment` can be injected into the `Invoke` method when creating custom [middleware](xref:fundamentals/middleware/write):</span></span>

```csharp
public async Task Invoke(HttpContext context, IWebHostEnvironment env)
{
    if (env.IsDevelopment())
    {
        // Configure middleware for Development
    }
    else
    {
        // Configure middleware for Staging/Production
    }

    var contentRootPath = env.ContentRootPath;
}
```

::: moniker-end

::: moniker range="< aspnetcore-3.0"

## <a name="ihostingenvironment-interface"></a><span data-ttu-id="2662b-378">Interfaccia IHostingEnvironment</span><span class="sxs-lookup"><span data-stu-id="2662b-378">IHostingEnvironment interface</span></span>

<span data-ttu-id="2662b-379">L'[interfaccia IHostingEnvironment](/dotnet/api/microsoft.aspnetcore.hosting.ihostingenvironment) fornisce informazioni sull'ambiente di hosting Web dell'app.</span><span class="sxs-lookup"><span data-stu-id="2662b-379">The [IHostingEnvironment interface](/dotnet/api/microsoft.aspnetcore.hosting.ihostingenvironment) provides information about the app's web hosting environment.</span></span> <span data-ttu-id="2662b-380">Usare l'[inserimento di un costruttore](xref:fundamentals/dependency-injection) per ottenere `IHostingEnvironment` per poterne usare le proprietà e i metodi di estensione:</span><span class="sxs-lookup"><span data-stu-id="2662b-380">Use [constructor injection](xref:fundamentals/dependency-injection) to obtain the `IHostingEnvironment` in order to use its properties and extension methods:</span></span>

```csharp
public class CustomFileReader
{
    private readonly IHostingEnvironment _env;

    public CustomFileReader(IHostingEnvironment env)
    {
        _env = env;
    }

    public string ReadFile(string filePath)
    {
        var fileProvider = _env.WebRootFileProvider;
        // Process the file here
    }
}
```

<span data-ttu-id="2662b-381">È possibile usare un [approccio basato su convenzione](xref:fundamentals/environments#environment-based-startup-class-and-methods) per configurare l'app all'avvio in base all'ambiente.</span><span class="sxs-lookup"><span data-stu-id="2662b-381">A [convention-based approach](xref:fundamentals/environments#environment-based-startup-class-and-methods) can be used to configure the app at startup based on the environment.</span></span> <span data-ttu-id="2662b-382">In alternativa, inserire `IHostingEnvironment` nel costruttore `Startup` per l'utilizzo in `ConfigureServices`:</span><span class="sxs-lookup"><span data-stu-id="2662b-382">Alternatively, inject the `IHostingEnvironment` into the `Startup` constructor for use in `ConfigureServices`:</span></span>

```csharp
public class Startup
{
    public Startup(IHostingEnvironment env)
    {
        HostingEnvironment = env;
    }

    public IHostingEnvironment HostingEnvironment { get; }

    public void ConfigureServices(IServiceCollection services)
    {
        if (HostingEnvironment.IsDevelopment())
        {
            // Development configuration
        }
        else
        {
            // Staging/Production configuration
        }

        var contentRootPath = HostingEnvironment.ContentRootPath;
    }
}
```

> [!NOTE]
> <span data-ttu-id="2662b-383">Oltre al metodo di estensione `IsDevelopment`, `IHostingEnvironment` offre i metodi `IsStaging`, `IsProduction` e `IsEnvironment(string environmentName)`.</span><span class="sxs-lookup"><span data-stu-id="2662b-383">In addition to the `IsDevelopment` extension method, `IHostingEnvironment` offers `IsStaging`, `IsProduction`, and `IsEnvironment(string environmentName)` methods.</span></span> <span data-ttu-id="2662b-384">Per altre informazioni, vedere <xref:fundamentals/environments>.</span><span class="sxs-lookup"><span data-stu-id="2662b-384">For more information, see <xref:fundamentals/environments>.</span></span>

<span data-ttu-id="2662b-385">Il servizio `IHostingEnvironment` può anche essere inserito direttamente nel metodo `Configure` per configurare la pipeline di elaborazione:</span><span class="sxs-lookup"><span data-stu-id="2662b-385">The `IHostingEnvironment` service can also be injected directly into the `Configure` method for setting up the processing pipeline:</span></span>

```csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    if (env.IsDevelopment())
    {
        // In Development, use the Developer Exception Page
        app.UseDeveloperExceptionPage();
    }
    else
    {
        // In Staging/Production, route exceptions to /error
        app.UseExceptionHandler("/error");
    }

    var contentRootPath = env.ContentRootPath;
}
```

<span data-ttu-id="2662b-386">`IHostingEnvironment` può essere inserito nel metodo `Invoke` durante la creazione di [middleware](xref:fundamentals/middleware/write) personalizzato:</span><span class="sxs-lookup"><span data-stu-id="2662b-386">`IHostingEnvironment` can be injected into the `Invoke` method when creating custom [middleware](xref:fundamentals/middleware/write):</span></span>

```csharp
public async Task Invoke(HttpContext context, IHostingEnvironment env)
{
    if (env.IsDevelopment())
    {
        // Configure middleware for Development
    }
    else
    {
        // Configure middleware for Staging/Production
    }

    var contentRootPath = env.ContentRootPath;
}
```

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

## <a name="ihostapplicationlifetime-interface"></a><span data-ttu-id="2662b-387">Interfaccia IHostApplicationLifetime</span><span class="sxs-lookup"><span data-stu-id="2662b-387">IHostApplicationLifetime interface</span></span>

<span data-ttu-id="2662b-388">`IHostApplicationLifetime` consente attività post-avvio e di arresto.</span><span class="sxs-lookup"><span data-stu-id="2662b-388">`IHostApplicationLifetime` allows for post-startup and shutdown activities.</span></span> <span data-ttu-id="2662b-389">Tre proprietà nell'interfaccia sono token di annullamento usati per registrare i metodi `Action` che definiscono gli eventi di avvio e arresto.</span><span class="sxs-lookup"><span data-stu-id="2662b-389">Three properties on the interface are cancellation tokens used to register `Action` methods that define startup and shutdown events.</span></span>

| <span data-ttu-id="2662b-390">Token di annullamento</span><span class="sxs-lookup"><span data-stu-id="2662b-390">Cancellation Token</span></span>    | <span data-ttu-id="2662b-391">Attivato quando&#8230;</span><span class="sxs-lookup"><span data-stu-id="2662b-391">Triggered when&#8230;</span></span> |
| --------------------- | --------------------- |
| `ApplicationStarted`  | <span data-ttu-id="2662b-392">L'host è stato completamente avviato.</span><span class="sxs-lookup"><span data-stu-id="2662b-392">The host has fully started.</span></span> |
| `ApplicationStopped`  | <span data-ttu-id="2662b-393">L'host sta completando un arresto normale.</span><span class="sxs-lookup"><span data-stu-id="2662b-393">The host is completing a graceful shutdown.</span></span> <span data-ttu-id="2662b-394">Tutte le richieste devono essere elaborate.</span><span class="sxs-lookup"><span data-stu-id="2662b-394">All requests should be processed.</span></span> <span data-ttu-id="2662b-395">L'arresto è bloccato fino al completamento di questo evento.</span><span class="sxs-lookup"><span data-stu-id="2662b-395">Shutdown blocks until this event completes.</span></span> |
| `ApplicationStopping` | <span data-ttu-id="2662b-396">L'host sta eseguendo un arresto normale.</span><span class="sxs-lookup"><span data-stu-id="2662b-396">The host is performing a graceful shutdown.</span></span> <span data-ttu-id="2662b-397">È possibile che le richieste siano ancora in esecuzione.</span><span class="sxs-lookup"><span data-stu-id="2662b-397">Requests may still be processing.</span></span> <span data-ttu-id="2662b-398">L'arresto è bloccato fino al completamento di questo evento.</span><span class="sxs-lookup"><span data-stu-id="2662b-398">Shutdown blocks until this event completes.</span></span> |

```csharp
public class Startup
{
    public void Configure(IApplicationBuilder app, IHostApplicationLifetime appLifetime)
    {
        appLifetime.ApplicationStarted.Register(OnStarted);
        appLifetime.ApplicationStopping.Register(OnStopping);
        appLifetime.ApplicationStopped.Register(OnStopped);

        Console.CancelKeyPress += (sender, eventArgs) =>
        {
            appLifetime.StopApplication();
            // Don't terminate the process immediately, wait for the Main thread to exit gracefully.
            eventArgs.Cancel = true;
        };
    }

    private void OnStarted()
    {
        // Perform post-startup activities here
    }

    private void OnStopping()
    {
        // Perform on-stopping activities here
    }

    private void OnStopped()
    {
        // Perform post-stopped activities here
    }
}
```

<span data-ttu-id="2662b-399">`StopApplication` richiede la chiusura dell'applicazione corrente.</span><span class="sxs-lookup"><span data-stu-id="2662b-399">`StopApplication` requests termination of the app.</span></span> <span data-ttu-id="2662b-400">La classe seguente usa `StopApplication` per arrestare normalmente un'app quando viene chiamato il metodo `Shutdown` della classe:</span><span class="sxs-lookup"><span data-stu-id="2662b-400">The following class uses `StopApplication` to gracefully shut down an app when the class's `Shutdown` method is called:</span></span>

```csharp
public class MyClass
{
    private readonly IHostApplicationLifetime _appLifetime;

    public MyClass(IHostApplicationLifetime appLifetime)
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

::: moniker range="< aspnetcore-3.0"

## <a name="iapplicationlifetime-interface"></a><span data-ttu-id="2662b-401">Interfaccia IApplicationLifetime</span><span class="sxs-lookup"><span data-stu-id="2662b-401">IApplicationLifetime interface</span></span>

<span data-ttu-id="2662b-402">[IApplicationLifetime](/dotnet/api/microsoft.aspnetcore.hosting.iapplicationlifetime) consente attività post-avvio e di arresto.</span><span class="sxs-lookup"><span data-stu-id="2662b-402">[IApplicationLifetime](/dotnet/api/microsoft.aspnetcore.hosting.iapplicationlifetime) allows for post-startup and shutdown activities.</span></span> <span data-ttu-id="2662b-403">Tre proprietà nell'interfaccia sono token di annullamento usati per registrare i metodi `Action` che definiscono gli eventi di avvio e arresto.</span><span class="sxs-lookup"><span data-stu-id="2662b-403">Three properties on the interface are cancellation tokens used to register `Action` methods that define startup and shutdown events.</span></span>

| <span data-ttu-id="2662b-404">Token di annullamento</span><span class="sxs-lookup"><span data-stu-id="2662b-404">Cancellation Token</span></span>    | <span data-ttu-id="2662b-405">Attivato quando&#8230;</span><span class="sxs-lookup"><span data-stu-id="2662b-405">Triggered when&#8230;</span></span> |
| --------------------- | --------------------- |
| [<span data-ttu-id="2662b-406">ApplicationStarted</span><span class="sxs-lookup"><span data-stu-id="2662b-406">ApplicationStarted</span></span>](/dotnet/api/microsoft.extensions.hosting.iapplicationlifetime.applicationstarted) | <span data-ttu-id="2662b-407">L'host è stato completamente avviato.</span><span class="sxs-lookup"><span data-stu-id="2662b-407">The host has fully started.</span></span> |
| [<span data-ttu-id="2662b-408">ApplicationStopped</span><span class="sxs-lookup"><span data-stu-id="2662b-408">ApplicationStopped</span></span>](/dotnet/api/microsoft.extensions.hosting.iapplicationlifetime.applicationstopped) | <span data-ttu-id="2662b-409">L'host sta completando un arresto normale.</span><span class="sxs-lookup"><span data-stu-id="2662b-409">The host is completing a graceful shutdown.</span></span> <span data-ttu-id="2662b-410">Tutte le richieste devono essere elaborate.</span><span class="sxs-lookup"><span data-stu-id="2662b-410">All requests should be processed.</span></span> <span data-ttu-id="2662b-411">L'arresto è bloccato fino al completamento di questo evento.</span><span class="sxs-lookup"><span data-stu-id="2662b-411">Shutdown blocks until this event completes.</span></span> |
| [<span data-ttu-id="2662b-412">ApplicationStopping</span><span class="sxs-lookup"><span data-stu-id="2662b-412">ApplicationStopping</span></span>](/dotnet/api/microsoft.extensions.hosting.iapplicationlifetime.applicationstopping) | <span data-ttu-id="2662b-413">L'host sta eseguendo un arresto normale.</span><span class="sxs-lookup"><span data-stu-id="2662b-413">The host is performing a graceful shutdown.</span></span> <span data-ttu-id="2662b-414">È possibile che le richieste siano ancora in esecuzione.</span><span class="sxs-lookup"><span data-stu-id="2662b-414">Requests may still be processing.</span></span> <span data-ttu-id="2662b-415">L'arresto è bloccato fino al completamento di questo evento.</span><span class="sxs-lookup"><span data-stu-id="2662b-415">Shutdown blocks until this event completes.</span></span> |

```csharp
public class Startup
{
    public void Configure(IApplicationBuilder app, IApplicationLifetime appLifetime)
    {
        appLifetime.ApplicationStarted.Register(OnStarted);
        appLifetime.ApplicationStopping.Register(OnStopping);
        appLifetime.ApplicationStopped.Register(OnStopped);

        Console.CancelKeyPress += (sender, eventArgs) =>
        {
            appLifetime.StopApplication();
            // Don't terminate the process immediately, wait for the Main thread to exit gracefully.
            eventArgs.Cancel = true;
        };
    }

    private void OnStarted()
    {
        // Perform post-startup activities here
    }

    private void OnStopping()
    {
        // Perform on-stopping activities here
    }

    private void OnStopped()
    {
        // Perform post-stopped activities here
    }
}
```

<span data-ttu-id="2662b-416">[StopApplication](/dotnet/api/microsoft.aspnetcore.hosting.iapplicationlifetime.stopapplication) richiede la chiusura dell'applicazione corrente.</span><span class="sxs-lookup"><span data-stu-id="2662b-416">[StopApplication](/dotnet/api/microsoft.aspnetcore.hosting.iapplicationlifetime.stopapplication) requests termination of the app.</span></span> <span data-ttu-id="2662b-417">La classe seguente usa `StopApplication` per arrestare normalmente un'app quando viene chiamato il metodo `Shutdown` della classe:</span><span class="sxs-lookup"><span data-stu-id="2662b-417">The following class uses `StopApplication` to gracefully shut down an app when the class's `Shutdown` method is called:</span></span>

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

## <a name="scope-validation"></a><span data-ttu-id="2662b-418">Convalida dell'ambito</span><span class="sxs-lookup"><span data-stu-id="2662b-418">Scope validation</span></span>

<span data-ttu-id="2662b-419">[CreateDefaultBuilder](/dotnet/api/microsoft.aspnetcore.webhost.createdefaultbuilder) imposta [ServiceProviderOptions.ValidateScopes](/dotnet/api/microsoft.extensions.dependencyinjection.serviceprovideroptions.validatescopes) su `true` se l'ambiente dell'app è lo sviluppo.</span><span class="sxs-lookup"><span data-stu-id="2662b-419">[CreateDefaultBuilder](/dotnet/api/microsoft.aspnetcore.webhost.createdefaultbuilder) sets [ServiceProviderOptions.ValidateScopes](/dotnet/api/microsoft.extensions.dependencyinjection.serviceprovideroptions.validatescopes) to `true` if the app's environment is Development.</span></span>

<span data-ttu-id="2662b-420">Quando `ValidateScopes` è impostato su `true`, il provider di servizi predefinito esegue dei controlli per verificare che:</span><span class="sxs-lookup"><span data-stu-id="2662b-420">When `ValidateScopes` is set to `true`, the default service provider performs checks to verify that:</span></span>

* <span data-ttu-id="2662b-421">I servizi con ambito non vengano risolti direttamente o indirettamente dal provider di servizi radice.</span><span class="sxs-lookup"><span data-stu-id="2662b-421">Scoped services aren't directly or indirectly resolved from the root service provider.</span></span>
* <span data-ttu-id="2662b-422">I servizi con ambito non vengano inseriti direttamente o indirettamente in singleton.</span><span class="sxs-lookup"><span data-stu-id="2662b-422">Scoped services aren't directly or indirectly injected into singletons.</span></span>

<span data-ttu-id="2662b-423">Il provider di servizi radice venga creato con la chiamata di [BuildServiceProvider](/dotnet/api/microsoft.extensions.dependencyinjection.servicecollectioncontainerbuilderextensions.buildserviceprovider).</span><span class="sxs-lookup"><span data-stu-id="2662b-423">The root service provider is created when [BuildServiceProvider](/dotnet/api/microsoft.extensions.dependencyinjection.servicecollectioncontainerbuilderextensions.buildserviceprovider) is called.</span></span> <span data-ttu-id="2662b-424">La durata del provider di servizi radice corrisponde alla durata dell'app/server quando il provider viene avviato con l'app e viene eliminato alla chiusura dell'app.</span><span class="sxs-lookup"><span data-stu-id="2662b-424">The root service provider's lifetime corresponds to the app/server's lifetime when the provider starts with the app and is disposed when the app shuts down.</span></span>

<span data-ttu-id="2662b-425">I servizi con ambito vengono eliminati dal contenitore che li ha creati.</span><span class="sxs-lookup"><span data-stu-id="2662b-425">Scoped services are disposed by the container that created them.</span></span> <span data-ttu-id="2662b-426">Se un servizio con ambito viene creato nel contenitore radice, la durata del servizio viene di fatto convertita in singleton, perché il servizio viene eliminato solo dal contenitore radice alla chiusura dell'app/server.</span><span class="sxs-lookup"><span data-stu-id="2662b-426">If a scoped service is created in the root container, the service's lifetime is effectively promoted to singleton because it's only disposed by the root container when app/server is shut down.</span></span> <span data-ttu-id="2662b-427">La convalida degli ambiti servizio rileva queste situazioni nel contesto della chiamata di `BuildServiceProvider`.</span><span class="sxs-lookup"><span data-stu-id="2662b-427">Validating service scopes catches these situations when `BuildServiceProvider` is called.</span></span>

<span data-ttu-id="2662b-428">Per convalidare sempre gli ambiti, inclusi quelli dell'ambiente di produzione, configurare [ServiceProviderOptions](/dotnet/api/microsoft.extensions.dependencyinjection.serviceprovideroptions) con [UseDefaultServiceProvider](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilderextensions.usedefaultserviceprovider) nel generatore di host:</span><span class="sxs-lookup"><span data-stu-id="2662b-428">To always validate scopes, including in the Production environment, configure the [ServiceProviderOptions](/dotnet/api/microsoft.extensions.dependencyinjection.serviceprovideroptions) with [UseDefaultServiceProvider](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilderextensions.usedefaultserviceprovider) on the host builder:</span></span>

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseDefaultServiceProvider((context, options) => {
        options.ValidateScopes = true;
    })
```

## <a name="additional-resources"></a><span data-ttu-id="2662b-429">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="2662b-429">Additional resources</span></span>

* <xref:host-and-deploy/iis/index>
* <xref:host-and-deploy/linux-nginx>
* <xref:host-and-deploy/linux-apache>
* <xref:host-and-deploy/windows-service>
