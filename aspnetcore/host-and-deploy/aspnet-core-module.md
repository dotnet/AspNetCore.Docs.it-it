---
title: Modulo ASP.NET Core
author: rick-anderson
description: Informazioni sul modulo ASP.NET Core per l'hosting di app ASP.NET Core con IIS.
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 01/13/2020
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
uid: host-and-deploy/aspnet-core-module
ms.openlocfilehash: ffa2a7c73e00ccfb5dd726a084352c0b80cc41b0
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102586969"
---
# <a name="aspnet-core-module"></a><span data-ttu-id="bf97d-103">Modulo ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="bf97d-103">ASP.NET Core Module</span></span>

<span data-ttu-id="bf97d-104">Di [Tom Dykstra](https://github.com/tdykstra), [Rick Strahl](https://github.com/RickStrahl), [Chris Ross](https://github.com/Tratcher), [Rick Anderson](https://twitter.com/RickAndMSFT), [Shirhatti](https://twitter.com/sshirhatti)e [Justin Kotalik](https://github.com/jkotalik)</span><span class="sxs-lookup"><span data-stu-id="bf97d-104">By [Tom Dykstra](https://github.com/tdykstra), [Rick Strahl](https://github.com/RickStrahl), [Chris Ross](https://github.com/Tratcher), [Rick Anderson](https://twitter.com/RickAndMSFT), [Sourabh Shirhatti](https://twitter.com/sshirhatti), and [Justin Kotalik](https://github.com/jkotalik)</span></span>

::: moniker range=">= aspnetcore-5.0"

<span data-ttu-id="bf97d-105">Il modulo ASP.NET Core è un modulo IIS nativo che si connette alla pipeline IIS, consentendo a ASP.NET Core applicazioni di utilizzare IIS.</span><span class="sxs-lookup"><span data-stu-id="bf97d-105">The ASP.NET Core Module is a native IIS module that plugs into the IIS pipeline, allowing ASP.NET Core applications to work with IIS.</span></span> <span data-ttu-id="bf97d-106">Eseguire ASP.NET Core app con IIS in uno dei seguenti casi:</span><span class="sxs-lookup"><span data-stu-id="bf97d-106">Run ASP.NET Core apps with IIS by either:</span></span> 

* <span data-ttu-id="bf97d-107">Hosting di un'app ASP.NET Core all'interno del processo di lavoro IIS ( `w3wp.exe` ), denominato [modello di hosting in-process](xref:host-and-deploy/iis/in-process-hosting).</span><span class="sxs-lookup"><span data-stu-id="bf97d-107">Hosting an ASP.NET Core app inside of the IIS worker process (`w3wp.exe`), called the [in-process hosting model](xref:host-and-deploy/iis/in-process-hosting).</span></span>
* <span data-ttu-id="bf97d-108">L'invio di richieste Web a un'app ASP.NET Core back-end che esegue il server gheppio, denominato [modello di hosting out-of-process](xref:host-and-deploy/iis/out-of-process-hosting).</span><span class="sxs-lookup"><span data-stu-id="bf97d-108">Forwarding web requests to a backend ASP.NET Core app running the Kestrel server, called the [out-of-process hosting model](xref:host-and-deploy/iis/out-of-process-hosting).</span></span>

<span data-ttu-id="bf97d-109">Esistono compromessi tra i modelli di hosting.</span><span class="sxs-lookup"><span data-stu-id="bf97d-109">There are trade-offs between each of the hosting models.</span></span> <span data-ttu-id="bf97d-110">Per impostazione predefinita, il modello di hosting in-process viene usato a causa di prestazioni e diagnostica migliori.</span><span class="sxs-lookup"><span data-stu-id="bf97d-110">By default, the in-process hosting model is used due to better performance and diagnostics.</span></span>

## <a name="install-aspnet-core-module"></a><span data-ttu-id="bf97d-111">Installare ASP.NET Core modulo</span><span class="sxs-lookup"><span data-stu-id="bf97d-111">Install ASP.NET Core Module</span></span>

<span data-ttu-id="bf97d-112">Scaricare il programma di installazione mediante il collegamento seguente:</span><span class="sxs-lookup"><span data-stu-id="bf97d-112">Download the installer using the following link:</span></span>

[<span data-ttu-id="bf97d-113">Programma di installazione del bundle di hosting .NET Core corrente (download diretto)</span><span class="sxs-lookup"><span data-stu-id="bf97d-113">Current .NET Core Hosting Bundle installer (direct download)</span></span>](https://dotnet.microsoft.com/permalink/dotnetcore-current-windows-runtime-bundle-installer)

<span data-ttu-id="bf97d-114">Per altre informazioni dettagliate su come installare il modulo ASP.NET Core o installare versioni diverse del modulo, vedere [Install the .NET Core hosting bundle](xref:host-and-deploy/iis/hosting-bundle).</span><span class="sxs-lookup"><span data-stu-id="bf97d-114">For more details instructions on how to install the ASP.NET Core Module, or installing different versions of the module, see [Install the .NET Core Hosting Bundle](xref:host-and-deploy/iis/hosting-bundle).</span></span>

<span data-ttu-id="bf97d-115">Per un'esercitazione sulla pubblicazione di un'app ASP.NET Core in un server IIS, vedere <xref:tutorials/publish-to-iis>.</span><span class="sxs-lookup"><span data-stu-id="bf97d-115">For a tutorial experience on publishing an ASP.NET Core app to an IIS server, see <xref:tutorials/publish-to-iis>.</span></span>

::: moniker-end

::: moniker range=">= aspnetcore-3.0 < aspnetcore-5.0"

<span data-ttu-id="bf97d-116">Il modulo ASP.NET Core è un modulo IIS nativo collegato alla pipeline di IIS per eseguire le operazioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="bf97d-116">The ASP.NET Core Module is a native IIS module that plugs into the IIS pipeline to either:</span></span>

* <span data-ttu-id="bf97d-117">Ospitare un app ASP.NET Core all'interno del processo di lavoro IIS (`w3wp.exe`), denominato [modello di hosting in-process](#in-process-hosting-model).</span><span class="sxs-lookup"><span data-stu-id="bf97d-117">Host an ASP.NET Core app inside of the IIS worker process (`w3wp.exe`), called the [in-process hosting model](#in-process-hosting-model).</span></span>
* <span data-ttu-id="bf97d-118">Inoltrare le richieste Web all'app back-end ASP.NET Core che esegue il [server Kestrel](xref:fundamentals/servers/kestrel), denominato [modello di hosting out-of-process](#out-of-process-hosting-model).</span><span class="sxs-lookup"><span data-stu-id="bf97d-118">Forward web requests to a backend ASP.NET Core app running the [Kestrel server](xref:fundamentals/servers/kestrel), called the [out-of-process hosting model](#out-of-process-hosting-model).</span></span>

<span data-ttu-id="bf97d-119">Versioni supportate di Windows:</span><span class="sxs-lookup"><span data-stu-id="bf97d-119">Supported Windows versions:</span></span>

* <span data-ttu-id="bf97d-120">Windows 7 o versione successiva</span><span class="sxs-lookup"><span data-stu-id="bf97d-120">Windows 7 or later</span></span>
* <span data-ttu-id="bf97d-121">Windows Server 2012 R2 o versione successiva</span><span class="sxs-lookup"><span data-stu-id="bf97d-121">Windows Server 2012 R2 or later</span></span>

<span data-ttu-id="bf97d-122">In caso di hosting in-process, il modulo usa un'implementazione di server in-process per IIS detta server HTTP di IIS (`IISHttpServer`).</span><span class="sxs-lookup"><span data-stu-id="bf97d-122">When hosting in-process, the module uses an in-process server implementation for IIS, called IIS HTTP Server (`IISHttpServer`).</span></span>

<span data-ttu-id="bf97d-123">In caso di hosting out-of-process, il modulo funziona solo con Kestrel.</span><span class="sxs-lookup"><span data-stu-id="bf97d-123">When hosting out-of-process, the module only works with Kestrel.</span></span> <span data-ttu-id="bf97d-124">Il modulo non funziona con [HTTP.sys](xref:fundamentals/servers/httpsys).</span><span class="sxs-lookup"><span data-stu-id="bf97d-124">The module doesn't function with [HTTP.sys](xref:fundamentals/servers/httpsys).</span></span>

## <a name="hosting-models"></a><span data-ttu-id="bf97d-125">Modelli di hosting</span><span class="sxs-lookup"><span data-stu-id="bf97d-125">Hosting models</span></span>

### <a name="in-process-hosting-model"></a><span data-ttu-id="bf97d-126">Modello di hosting in-process</span><span class="sxs-lookup"><span data-stu-id="bf97d-126">In-process hosting model</span></span>

<span data-ttu-id="bf97d-127">ASP.NET Core app predefinite per il modello di hosting in-process.</span><span class="sxs-lookup"><span data-stu-id="bf97d-127">ASP.NET Core apps default to the in-process hosting model.</span></span>

<span data-ttu-id="bf97d-128">In caso di hosting in-process, vengono applicate le caratteristiche seguenti:</span><span class="sxs-lookup"><span data-stu-id="bf97d-128">The following characteristics apply when hosting in-process:</span></span>

* <span data-ttu-id="bf97d-129">Viene usato un server HTTP di IIS (`IISHttpServer`) al posto del server [Kestrel](xref:fundamentals/servers/kestrel).</span><span class="sxs-lookup"><span data-stu-id="bf97d-129">IIS HTTP Server (`IISHttpServer`) is used instead of [Kestrel](xref:fundamentals/servers/kestrel) server.</span></span> <span data-ttu-id="bf97d-130">Per in-process, [CreateDefaultBuilder](xref:fundamentals/host/generic-host#default-builder-settings) chiama <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderIISExtensions.UseIIS*> per:</span><span class="sxs-lookup"><span data-stu-id="bf97d-130">For in-process, [CreateDefaultBuilder](xref:fundamentals/host/generic-host#default-builder-settings) calls <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderIISExtensions.UseIIS*> to:</span></span>

  * <span data-ttu-id="bf97d-131">Registrare `IISHttpServer`.</span><span class="sxs-lookup"><span data-stu-id="bf97d-131">Register the `IISHttpServer`.</span></span>
  * <span data-ttu-id="bf97d-132">Configurare la porta e il percorso di base su cui il server deve eseguire l'ascolto in caso di esecuzione dietro il modulo ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="bf97d-132">Configure the port and base path the server should listen on when running behind the ASP.NET Core Module.</span></span>
  * <span data-ttu-id="bf97d-133">Configurare l'host per l'acquisizione degli errori di avvio.</span><span class="sxs-lookup"><span data-stu-id="bf97d-133">Configure the host to capture startup errors.</span></span>

* <span data-ttu-id="bf97d-134">L'[attributo requestTimeout ](#attributes-of-the-aspnetcore-element) non viene applicato all'hosting in-process.</span><span class="sxs-lookup"><span data-stu-id="bf97d-134">The [requestTimeout attribute](#attributes-of-the-aspnetcore-element) doesn't apply to in-process hosting.</span></span>

* <span data-ttu-id="bf97d-135">Non è supportata la condivisione di un pool di app tra le app.</span><span class="sxs-lookup"><span data-stu-id="bf97d-135">Sharing an app pool among apps isn't supported.</span></span> <span data-ttu-id="bf97d-136">Usare un pool di app per ogni app.</span><span class="sxs-lookup"><span data-stu-id="bf97d-136">Use one app pool per app.</span></span>

* <span data-ttu-id="bf97d-137">Quando si usa [distribuzione Web](/iis/publish/using-web-deploy/introduction-to-web-deploy) o si posiziona manualmente un [ `app_offline.htm` file nella distribuzione](xref:host-and-deploy/iis/index#locked-deployment-files), l'app potrebbe non arrestarsi immediatamente se è presente una connessione aperta.</span><span class="sxs-lookup"><span data-stu-id="bf97d-137">When using [Web Deploy](/iis/publish/using-web-deploy/introduction-to-web-deploy) or manually placing an [`app_offline.htm` file in the deployment](xref:host-and-deploy/iis/index#locked-deployment-files), the app might not shut down immediately if there's an open connection.</span></span> <span data-ttu-id="bf97d-138">Ad esempio, una connessione WebSocket potrebbe ritardare l'arresto dell'app.</span><span class="sxs-lookup"><span data-stu-id="bf97d-138">For example, a WebSocket connection may delay app shut down.</span></span>

* <span data-ttu-id="bf97d-139">L'architettura, vale a dire il numero di bit dell'app, e il runtime installato (x64 o x86) devono corrispondere all'architettura del pool di app.</span><span class="sxs-lookup"><span data-stu-id="bf97d-139">The architecture (bitness) of the app and installed runtime (x64 or x86) must match the architecture of the app pool.</span></span>

* <span data-ttu-id="bf97d-140">Le disconnessioni del client vengono rilevate.</span><span class="sxs-lookup"><span data-stu-id="bf97d-140">Client disconnects are detected.</span></span> <span data-ttu-id="bf97d-141">Il [`HttpContext.RequestAborted`](xref:Microsoft.AspNetCore.Http.HttpContext.RequestAborted*) token di annullamento viene annullato quando il client si disconnette.</span><span class="sxs-lookup"><span data-stu-id="bf97d-141">The [`HttpContext.RequestAborted`](xref:Microsoft.AspNetCore.Http.HttpContext.RequestAborted*) cancellation token is cancelled when the client disconnects.</span></span>

* <span data-ttu-id="bf97d-142">In ASP.NET Core 2.2.1 o versioni precedenti, <xref:System.IO.Directory.GetCurrentDirectory*> restituisce la directory di lavoro del processo avviato da IIS anziché la directory dell'app, ad esempio `C:\Windows\System32\inetsrv` per `w3wp.exe` .</span><span class="sxs-lookup"><span data-stu-id="bf97d-142">In ASP.NET Core 2.2.1 or earlier, <xref:System.IO.Directory.GetCurrentDirectory*> returns the worker directory of the process started by IIS rather than the app's directory (for example, `C:\Windows\System32\inetsrv` for `w3wp.exe`).</span></span>

  <span data-ttu-id="bf97d-143">Per il codice di esempio che imposta la directory corrente dell'app, vedere la [ `CurrentDirectoryHelpers` classe](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/host-and-deploy/aspnet-core-module/samples_snapshot/3.x/CurrentDirectoryHelpers.cs).</span><span class="sxs-lookup"><span data-stu-id="bf97d-143">For sample code that sets the app's current directory, see the [`CurrentDirectoryHelpers` class](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/host-and-deploy/aspnet-core-module/samples_snapshot/3.x/CurrentDirectoryHelpers.cs).</span></span> <span data-ttu-id="bf97d-144">Chiamare il metodo `SetCurrentDirectory` .</span><span class="sxs-lookup"><span data-stu-id="bf97d-144">Call the `SetCurrentDirectory` method.</span></span> <span data-ttu-id="bf97d-145">Le chiamate successive a <xref:System.IO.Directory.GetCurrentDirectory*> specificano la directory dell'app.</span><span class="sxs-lookup"><span data-stu-id="bf97d-145">Subsequent calls to <xref:System.IO.Directory.GetCurrentDirectory*> provide the app's directory.</span></span>

* <span data-ttu-id="bf97d-146">In caso di hosting in-process, <xref:Microsoft.AspNetCore.Authentication.AuthenticationService.AuthenticateAsync*> non viene chiamato internamente per inizializzare un utente.</span><span class="sxs-lookup"><span data-stu-id="bf97d-146">When hosting in-process, <xref:Microsoft.AspNetCore.Authentication.AuthenticationService.AuthenticateAsync*> isn't called internally to initialize a user.</span></span> <span data-ttu-id="bf97d-147">Pertanto, un'implementazione di <xref:Microsoft.AspNetCore.Authentication.IClaimsTransformation> usate per trasformare le attestazioni dopo ogni autenticazione non viene attivata per impostazione predefinita.</span><span class="sxs-lookup"><span data-stu-id="bf97d-147">Therefore, an <xref:Microsoft.AspNetCore.Authentication.IClaimsTransformation> implementation used to transform claims after every authentication isn't activated by default.</span></span> <span data-ttu-id="bf97d-148">Quando si trasformano le attestazioni con un'implementazione di <xref:Microsoft.AspNetCore.Authentication.IClaimsTransformation>, chiamare <xref:Microsoft.Extensions.DependencyInjection.AuthenticationServiceCollectionExtensions.AddAuthentication*> per aggiungere i servizi di autenticazione:</span><span class="sxs-lookup"><span data-stu-id="bf97d-148">When transforming claims with an <xref:Microsoft.AspNetCore.Authentication.IClaimsTransformation> implementation, call <xref:Microsoft.Extensions.DependencyInjection.AuthenticationServiceCollectionExtensions.AddAuthentication*> to add authentication services:</span></span>

  ```csharp
  public void ConfigureServices(IServiceCollection services)
  {
      services.AddTransient<IClaimsTransformation, ClaimsTransformer>();
      services.AddAuthentication(IISServerDefaults.AuthenticationScheme);
  }

  public void Configure(IApplicationBuilder app)
  {
      app.UseAuthentication();
  }
  ```
  
  * <span data-ttu-id="bf97d-149">Le [distribuzioni di pacchetti Web (file singolo)](/aspnet/web-forms/overview/deployment/web-deployment-in-the-enterprise/deploying-web-packages) non sono supportate.</span><span class="sxs-lookup"><span data-stu-id="bf97d-149">[Web Package (single-file) deployments](/aspnet/web-forms/overview/deployment/web-deployment-in-the-enterprise/deploying-web-packages) aren't supported.</span></span>

### <a name="out-of-process-hosting-model"></a><span data-ttu-id="bf97d-150">Modello di hosting out-of-process</span><span class="sxs-lookup"><span data-stu-id="bf97d-150">Out-of-process hosting model</span></span>

<span data-ttu-id="bf97d-151">Per configurare un'app per l'hosting out-of-process, impostare il valore della `<AspNetCoreHostingModel>` proprietà su `OutOfProcess` nel file di progetto ( `.csproj` ):</span><span class="sxs-lookup"><span data-stu-id="bf97d-151">To configure an app for out-of-process hosting, set the value of the `<AspNetCoreHostingModel>` property to `OutOfProcess` in the project file (`.csproj`):</span></span>

```xml
<PropertyGroup>
  <AspNetCoreHostingModel>OutOfProcess</AspNetCoreHostingModel>
</PropertyGroup>
```

<span data-ttu-id="bf97d-152">L'hosting in-process è impostato su `InProcess` , che corrisponde al valore predefinito.</span><span class="sxs-lookup"><span data-stu-id="bf97d-152">In-process hosting is set with `InProcess`, which is the default value.</span></span>

<span data-ttu-id="bf97d-153">Il valore di non `<AspNetCoreHostingModel>` fa distinzione tra maiuscole e minuscole, pertanto `inprocess` e `outofprocess` sono valori validi.</span><span class="sxs-lookup"><span data-stu-id="bf97d-153">The value of `<AspNetCoreHostingModel>` is case insensitive, so `inprocess` and `outofprocess` are valid values.</span></span>

<span data-ttu-id="bf97d-154">Viene usato il server [Kestrel](xref:fundamentals/servers/kestrel) al posto di un server HTTP di IIS (`IISHttpServer`).</span><span class="sxs-lookup"><span data-stu-id="bf97d-154">[Kestrel](xref:fundamentals/servers/kestrel) server is used instead of IIS HTTP Server (`IISHttpServer`).</span></span>

<span data-ttu-id="bf97d-155">Per out-of-process, le [`CreateDefaultBuilder`](xref:fundamentals/host/generic-host#default-builder-settings) chiamate <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderIISExtensions.UseIISIntegration*> a:</span><span class="sxs-lookup"><span data-stu-id="bf97d-155">For out-of-process, [`CreateDefaultBuilder`](xref:fundamentals/host/generic-host#default-builder-settings) calls <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderIISExtensions.UseIISIntegration*> to:</span></span>

* <span data-ttu-id="bf97d-156">Configurare la porta e il percorso di base su cui il server deve eseguire l'ascolto in caso di esecuzione dietro il modulo ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="bf97d-156">Configure the port and base path the server should listen on when running behind the ASP.NET Core Module.</span></span>
* <span data-ttu-id="bf97d-157">Configurare l'host per l'acquisizione degli errori di avvio.</span><span class="sxs-lookup"><span data-stu-id="bf97d-157">Configure the host to capture startup errors.</span></span>

### <a name="hosting-model-changes"></a><span data-ttu-id="bf97d-158">Modifiche del modello di hosting</span><span class="sxs-lookup"><span data-stu-id="bf97d-158">Hosting model changes</span></span>

<span data-ttu-id="bf97d-159">Se l' `hostingModel` impostazione viene modificata nel `web.config` file (illustrata nella sezione [configurazione con `web.config` ](#configuration-with-webconfig) ), il modulo ricicla il processo di lavoro per IIS.</span><span class="sxs-lookup"><span data-stu-id="bf97d-159">If the `hostingModel` setting is changed in the `web.config` file (explained in the [Configuration with `web.config`](#configuration-with-webconfig) section), the module recycles the worker process for IIS.</span></span>

<span data-ttu-id="bf97d-160">Per IIS Express, il modulo non ricicla il processo di lavoro. Attiva invece un arresto normale del processo di IIS Express corrente.</span><span class="sxs-lookup"><span data-stu-id="bf97d-160">For IIS Express, the module doesn't recycle the worker process but instead triggers a graceful shutdown of the current IIS Express process.</span></span> <span data-ttu-id="bf97d-161">La richiesta successiva all'app attiva un nuovo processo di IIS Express.</span><span class="sxs-lookup"><span data-stu-id="bf97d-161">The next request to the app spawns a new IIS Express process.</span></span>

### <a name="process-name"></a><span data-ttu-id="bf97d-162">Nome del processo</span><span class="sxs-lookup"><span data-stu-id="bf97d-162">Process name</span></span>

<span data-ttu-id="bf97d-163">`Process.GetCurrentProcess().ProcessName` dichiara `w3wp`/`iisexpress` (In-Process) o `dotnet` (out-of-process).</span><span class="sxs-lookup"><span data-stu-id="bf97d-163">`Process.GetCurrentProcess().ProcessName` reports `w3wp`/`iisexpress` (in-process) or `dotnet` (out-of-process).</span></span>

<span data-ttu-id="bf97d-164">Molti moduli nativi, ad esempio l'autenticazione di Windows, rimangono attivi.</span><span class="sxs-lookup"><span data-stu-id="bf97d-164">Many native modules, such as Windows Authentication, remain active.</span></span> <span data-ttu-id="bf97d-165">Per altre informazioni sui moduli IIS attivi con il modulo ASP.NET Core, vedere <xref:host-and-deploy/iis/modules>.</span><span class="sxs-lookup"><span data-stu-id="bf97d-165">To learn more about IIS modules active with the ASP.NET Core Module, see <xref:host-and-deploy/iis/modules>.</span></span>

<span data-ttu-id="bf97d-166">Il modulo ASP.NET Core può anche:</span><span class="sxs-lookup"><span data-stu-id="bf97d-166">The ASP.NET Core Module can also:</span></span>

* <span data-ttu-id="bf97d-167">Impostare variabili di ambiente per il processo di lavoro.</span><span class="sxs-lookup"><span data-stu-id="bf97d-167">Set environment variables for the worker process.</span></span>
* <span data-ttu-id="bf97d-168">Registrare output stdout in una risorsa di archiviazione di file per la risoluzione di problemi di avvio.</span><span class="sxs-lookup"><span data-stu-id="bf97d-168">Log stdout output to file storage for troubleshooting startup issues.</span></span>
* <span data-ttu-id="bf97d-169">Inoltrare token di autenticazione di Windows.</span><span class="sxs-lookup"><span data-stu-id="bf97d-169">Forward Windows authentication tokens.</span></span>

## <a name="how-to-install-and-use-the-aspnet-core-module"></a><span data-ttu-id="bf97d-170">Come installare e usare il modulo ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="bf97d-170">How to install and use the ASP.NET Core Module</span></span>

<span data-ttu-id="bf97d-171">Per istruzioni su come installare e usare il modulo ASP.NET Core, vedere [Installare il bundle di hosting .NET Core](xref:host-and-deploy/iis/index#install-the-net-core-hosting-bundle).</span><span class="sxs-lookup"><span data-stu-id="bf97d-171">For instructions on how to install the ASP.NET Core Module, see [Install the .NET Core Hosting Bundle](xref:host-and-deploy/iis/index#install-the-net-core-hosting-bundle).</span></span>

## <a name="configuration-with-webconfig"></a><span data-ttu-id="bf97d-172">Configurazione con web.config</span><span class="sxs-lookup"><span data-stu-id="bf97d-172">Configuration with web.config</span></span>

<span data-ttu-id="bf97d-173">Il modulo ASP.NET Core viene configurato tramite la sezione `aspNetCore` del nodo `system.webServer` nel file *web.config* del sito.</span><span class="sxs-lookup"><span data-stu-id="bf97d-173">The ASP.NET Core Module is configured with the `aspNetCore` section of the `system.webServer` node in the site's *web.config* file.</span></span>

<span data-ttu-id="bf97d-174">Il `web.config` file seguente viene pubblicato per una [distribuzione dipendente dal Framework](/dotnet/articles/core/deploying/#framework-dependent-deployments-fdd) e configura il modulo ASP.NET Core per gestire le richieste del sito:</span><span class="sxs-lookup"><span data-stu-id="bf97d-174">The following `web.config` file is published for a [framework-dependent deployment](/dotnet/articles/core/deploying/#framework-dependent-deployments-fdd) and configures the ASP.NET Core Module to handle site requests:</span></span>

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <location path="." inheritInChildApplications="false">
    <system.webServer>
      <handlers>
        <add name="aspNetCore" path="*" verb="*" modules="AspNetCoreModuleV2" resourceType="Unspecified" />
      </handlers>
      <aspNetCore processPath="dotnet"
                  arguments=".\MyApp.dll"
                  stdoutLogEnabled="false"
                  stdoutLogFile=".\logs\stdout"
                  hostingModel="inprocess" />
    </system.webServer>
  </location>
</configuration>
```

<span data-ttu-id="bf97d-175">Il file *web.config* seguente viene pubblicato per una [distribuzione autonoma](/dotnet/articles/core/deploying/#self-contained-deployments-scd):</span><span class="sxs-lookup"><span data-stu-id="bf97d-175">The following *web.config* is published for a [self-contained deployment](/dotnet/articles/core/deploying/#self-contained-deployments-scd):</span></span>

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <location path="." inheritInChildApplications="false">
    <system.webServer>
      <handlers>
        <add name="aspNetCore" path="*" verb="*" modules="AspNetCoreModuleV2" resourceType="Unspecified" />
      </handlers>
      <aspNetCore processPath=".\MyApp.exe"
                  stdoutLogEnabled="false"
                  stdoutLogFile=".\logs\stdout"
                  hostingModel="inprocess" />
    </system.webServer>
  </location>
</configuration>
```

<span data-ttu-id="bf97d-176">La <xref:System.Configuration.SectionInformation.InheritInChildApplications*> proprietà è impostata su `false` per indicare che le impostazioni specificate all'interno dell' [`<location>`](/iis/manage/managing-your-configuration-settings/understanding-iis-configuration-delegation#the-concept-of-location) elemento non vengono ereditate da app che si trovano in una sottodirectory dell'app.</span><span class="sxs-lookup"><span data-stu-id="bf97d-176">The <xref:System.Configuration.SectionInformation.InheritInChildApplications*> property is set to `false` to indicate that the settings specified within the [`<location>`](/iis/manage/managing-your-configuration-settings/understanding-iis-configuration-delegation#the-concept-of-location) element aren't inherited by apps that reside in a subdirectory of the app.</span></span>

<span data-ttu-id="bf97d-177">Quando un'app viene distribuita in [Servizio app di Azure](https://azure.microsoft.com/services/app-service/), il percorso `stdoutLogFile` è impostato su `\\?\%home%\LogFiles\stdout`.</span><span class="sxs-lookup"><span data-stu-id="bf97d-177">When an app is deployed to [Azure App Service](https://azure.microsoft.com/services/app-service/), the `stdoutLogFile` path is set to `\\?\%home%\LogFiles\stdout`.</span></span> <span data-ttu-id="bf97d-178">Il percorso Salva i log stdout nella `LogFiles` cartella, che è un percorso creato automaticamente dal servizio.</span><span class="sxs-lookup"><span data-stu-id="bf97d-178">The path saves stdout logs to the `LogFiles` folder, which is a location automatically created by the service.</span></span>

<span data-ttu-id="bf97d-179">Per informazioni sulla configurazione delle applicazioni secondarie IIS, vedere <xref:host-and-deploy/iis/index#sub-applications>.</span><span class="sxs-lookup"><span data-stu-id="bf97d-179">For information on IIS sub-application configuration, see <xref:host-and-deploy/iis/index#sub-applications>.</span></span>

### <a name="attributes-of-the-aspnetcore-element"></a><span data-ttu-id="bf97d-180">Attributi dell'elemento aspNetCore</span><span class="sxs-lookup"><span data-stu-id="bf97d-180">Attributes of the aspNetCore element</span></span>

| <span data-ttu-id="bf97d-181">Attributo</span><span class="sxs-lookup"><span data-stu-id="bf97d-181">Attribute</span></span> | <span data-ttu-id="bf97d-182">Descrizione</span><span class="sxs-lookup"><span data-stu-id="bf97d-182">Description</span></span> | <span data-ttu-id="bf97d-183">Predefinito</span><span class="sxs-lookup"><span data-stu-id="bf97d-183">Default</span></span> |
| --------- | ----------- | :-----: |
| `arguments` | <p><span data-ttu-id="bf97d-184">Attributo stringa facoltativo.</span><span class="sxs-lookup"><span data-stu-id="bf97d-184">Optional string attribute.</span></span></p><p><span data-ttu-id="bf97d-185">Argomenti per l'eseguibile specificato in **processPath**.</span><span class="sxs-lookup"><span data-stu-id="bf97d-185">Arguments to the executable specified in **processPath**.</span></span></p> | |
| `disableStartUpErrorPage` | <p><span data-ttu-id="bf97d-186">Attributo booleano facoltativo.</span><span class="sxs-lookup"><span data-stu-id="bf97d-186">Optional Boolean attribute.</span></span></p><p><span data-ttu-id="bf97d-187">Se true, la pagina **502.5 - Errore del processo** non viene visualizzata e la tabella codici di stato 502 configurata in *web.config* ha la precedenza.</span><span class="sxs-lookup"><span data-stu-id="bf97d-187">If true, the **502.5 - Process Failure** page is suppressed, and the 502 status code page configured in the *web.config* takes precedence.</span></span></p> | `false` |
| `forwardWindowsAuthToken` | <p><span data-ttu-id="bf97d-188">Attributo booleano facoltativo.</span><span class="sxs-lookup"><span data-stu-id="bf97d-188">Optional Boolean attribute.</span></span></p><p><span data-ttu-id="bf97d-189">Se true, il token viene inviato al processo figlio in ascolto `%ASPNETCORE_PORT%` come intestazione `'MS-ASPNETCORE-WINAUTHTOKEN'` per ogni richiesta.</span><span class="sxs-lookup"><span data-stu-id="bf97d-189">If true, the token is forwarded to the child process listening on `%ASPNETCORE_PORT%` as a header `'MS-ASPNETCORE-WINAUTHTOKEN'` per request.</span></span> <span data-ttu-id="bf97d-190">È responsabilità del processo chiamare CloseHandle su questo token per ogni richiesta.</span><span class="sxs-lookup"><span data-stu-id="bf97d-190">It's the responsibility of that process to call CloseHandle on this token per request.</span></span></p> | `true` |
| `hostingModel` | <p><span data-ttu-id="bf97d-191">Attributo stringa facoltativo.</span><span class="sxs-lookup"><span data-stu-id="bf97d-191">Optional string attribute.</span></span></p><p><span data-ttu-id="bf97d-192">Specifica il modello di hosting come in-process ( `InProcess` / `inprocess` ) o out-of-process ( `OutOfProcess` / `outofprocess` ).</span><span class="sxs-lookup"><span data-stu-id="bf97d-192">Specifies the hosting model as in-process (`InProcess`/`inprocess`) or out-of-process (`OutOfProcess`/`outofprocess`).</span></span></p> | `InProcess`<br>`inprocess` |
| `processesPerApplication` | <p><span data-ttu-id="bf97d-193">Attributo Integer facoltativo.</span><span class="sxs-lookup"><span data-stu-id="bf97d-193">Optional integer attribute.</span></span></p><p><span data-ttu-id="bf97d-194">Specifica il numero di istanze del processo specificato nell'impostazione **processPath** che può essere riattivato per ogni app.</span><span class="sxs-lookup"><span data-stu-id="bf97d-194">Specifies the number of instances of the process specified in the **processPath** setting that can be spun up per app.</span></span></p><p><span data-ttu-id="bf97d-195">&dagger;Per l'hosting in-process, il valore è limitato a `1`.</span><span class="sxs-lookup"><span data-stu-id="bf97d-195">&dagger;For in-process hosting, the value is limited to `1`.</span></span></p><p><span data-ttu-id="bf97d-196">L'impostazione di `processesPerApplication` è sconsigliata.</span><span class="sxs-lookup"><span data-stu-id="bf97d-196">Setting `processesPerApplication` is discouraged.</span></span> <span data-ttu-id="bf97d-197">Questo attributo sarà rimosso nelle versioni future.</span><span class="sxs-lookup"><span data-stu-id="bf97d-197">This attribute will be removed in a future release.</span></span></p> | <span data-ttu-id="bf97d-198">Valore predefinito: `1`</span><span class="sxs-lookup"><span data-stu-id="bf97d-198">Default: `1`</span></span><br><span data-ttu-id="bf97d-199">Min: `1`</span><span class="sxs-lookup"><span data-stu-id="bf97d-199">Min: `1`</span></span><br><span data-ttu-id="bf97d-200">Max: `100`&dagger;</span><span class="sxs-lookup"><span data-stu-id="bf97d-200">Max: `100`&dagger;</span></span> |
| `processPath` | <p><span data-ttu-id="bf97d-201">Attributo stringa obbligatorio.</span><span class="sxs-lookup"><span data-stu-id="bf97d-201">Required string attribute.</span></span></p><p><span data-ttu-id="bf97d-202">Percorso del file eseguibile che avvia un processo in ascolto delle richieste HTTP.</span><span class="sxs-lookup"><span data-stu-id="bf97d-202">Path to the executable that launches a process listening for HTTP requests.</span></span> <span data-ttu-id="bf97d-203">I percorsi relativi sono supportati.</span><span class="sxs-lookup"><span data-stu-id="bf97d-203">Relative paths are supported.</span></span> <span data-ttu-id="bf97d-204">Se il percorso inizia con `.`, viene considerato relativo alla radice del sito.</span><span class="sxs-lookup"><span data-stu-id="bf97d-204">If the path begins with `.`, the path is considered to be relative to the site root.</span></span></p> | |
| `rapidFailsPerMinute` | <p><span data-ttu-id="bf97d-205">Attributo Integer facoltativo.</span><span class="sxs-lookup"><span data-stu-id="bf97d-205">Optional integer attribute.</span></span></p><p><span data-ttu-id="bf97d-206">Specifica il numero di arresti anomali al minuto per il processo specificato in **processPath**.</span><span class="sxs-lookup"><span data-stu-id="bf97d-206">Specifies the number of times the process specified in **processPath** is allowed to crash per minute.</span></span> <span data-ttu-id="bf97d-207">Se questo limite viene superato, il modulo smette di avviare il processo per la parte restante del minuto.</span><span class="sxs-lookup"><span data-stu-id="bf97d-207">If this limit is exceeded, the module stops launching the process for the remainder of the minute.</span></span></p><p><span data-ttu-id="bf97d-208">Non supportato con l'hosting in-process.</span><span class="sxs-lookup"><span data-stu-id="bf97d-208">Not supported with in-process hosting.</span></span></p> | <span data-ttu-id="bf97d-209">Valore predefinito: `10`</span><span class="sxs-lookup"><span data-stu-id="bf97d-209">Default: `10`</span></span><br><span data-ttu-id="bf97d-210">Min: `0`</span><span class="sxs-lookup"><span data-stu-id="bf97d-210">Min: `0`</span></span><br><span data-ttu-id="bf97d-211">Max: `100`</span><span class="sxs-lookup"><span data-stu-id="bf97d-211">Max: `100`</span></span> |
| `requestTimeout` | <p><span data-ttu-id="bf97d-212">Attributo Timespan facoltativo.</span><span class="sxs-lookup"><span data-stu-id="bf97d-212">Optional timespan attribute.</span></span></p><p><span data-ttu-id="bf97d-213">Specifica la durata per cui il modulo ASP.NET Core attende una risposta dal processo in ascolto su %ASPNETCORE_PORT%.</span><span class="sxs-lookup"><span data-stu-id="bf97d-213">Specifies the duration for which the ASP.NET Core Module waits for a response from the process listening on %ASPNETCORE_PORT%.</span></span></p><p><span data-ttu-id="bf97d-214">Nelle versioni del modulo ASP.NET Core fornito con ASP.NET Core 2.1 o versioni successive, `requestTimeout` viene specificato in ore, minuti e secondi.</span><span class="sxs-lookup"><span data-stu-id="bf97d-214">In versions of the ASP.NET Core Module that shipped with the release of ASP.NET Core 2.1 or later, the `requestTimeout` is specified in hours, minutes, and seconds.</span></span></p><p><span data-ttu-id="bf97d-215">Non è applicabile all'hosting in-process.</span><span class="sxs-lookup"><span data-stu-id="bf97d-215">Doesn't apply to in-process hosting.</span></span> <span data-ttu-id="bf97d-216">Per l'hosting in-process, il modulo resta in attesa che l'app elabori la richiesta.</span><span class="sxs-lookup"><span data-stu-id="bf97d-216">For in-process hosting, the module waits for the app to process the request.</span></span></p><p><span data-ttu-id="bf97d-217">I valori validi per i segmenti della stringa relativi a minuti e secondi sono compresi nell'intervallo tra 0 e 59.</span><span class="sxs-lookup"><span data-stu-id="bf97d-217">Valid values for minutes and seconds segments of the string are in the range 0-59.</span></span> <span data-ttu-id="bf97d-218">Se si usa **60** come valore per i minuti o i secondi, viene generato un errore *500 - Errore interno del server*.</span><span class="sxs-lookup"><span data-stu-id="bf97d-218">Use of **60** in the value for minutes or seconds results in a *500 - Internal Server Error*.</span></span></p> | <span data-ttu-id="bf97d-219">Valore predefinito: `00:02:00`</span><span class="sxs-lookup"><span data-stu-id="bf97d-219">Default: `00:02:00`</span></span><br><span data-ttu-id="bf97d-220">Min: `00:00:00`</span><span class="sxs-lookup"><span data-stu-id="bf97d-220">Min: `00:00:00`</span></span><br><span data-ttu-id="bf97d-221">Max: `360:00:00`</span><span class="sxs-lookup"><span data-stu-id="bf97d-221">Max: `360:00:00`</span></span> |
| `shutdownTimeLimit` | <p><span data-ttu-id="bf97d-222">Attributo Integer facoltativo.</span><span class="sxs-lookup"><span data-stu-id="bf97d-222">Optional integer attribute.</span></span></p><p><span data-ttu-id="bf97d-223">Durata in secondi per cui il modulo attende che il file eseguibile venga arrestato normalmente quando viene rilevato il file *app_offline.htm*.</span><span class="sxs-lookup"><span data-stu-id="bf97d-223">Duration in seconds that the module waits for the executable to gracefully shutdown when the *app_offline.htm* file is detected.</span></span></p> | <span data-ttu-id="bf97d-224">Valore predefinito: `10`</span><span class="sxs-lookup"><span data-stu-id="bf97d-224">Default: `10`</span></span><br><span data-ttu-id="bf97d-225">Min: `0`</span><span class="sxs-lookup"><span data-stu-id="bf97d-225">Min: `0`</span></span><br><span data-ttu-id="bf97d-226">Max: `600`</span><span class="sxs-lookup"><span data-stu-id="bf97d-226">Max: `600`</span></span> |
| `startupTimeLimit` | <p><span data-ttu-id="bf97d-227">Attributo Integer facoltativo.</span><span class="sxs-lookup"><span data-stu-id="bf97d-227">Optional integer attribute.</span></span></p><p><span data-ttu-id="bf97d-228">Durata in secondi per cui il modulo attende l'avvio di un processo in ascolto sulla porta da parte del file eseguibile.</span><span class="sxs-lookup"><span data-stu-id="bf97d-228">Duration in seconds that the module waits for the executable to start a process listening on the port.</span></span> <span data-ttu-id="bf97d-229">Se questo limite di tempo viene superato, il modulo termina il processo.</span><span class="sxs-lookup"><span data-stu-id="bf97d-229">If this time limit is exceeded, the module kills the process.</span></span></p><p><span data-ttu-id="bf97d-230">Quando si esegue l'hosting *in-process*: il processo **non** viene riavviato **e non usa** l'impostazione **rapidFailsPerMinute** .</span><span class="sxs-lookup"><span data-stu-id="bf97d-230">When hosting *in-process*: The process is **not** restarted and does **not** use the **rapidFailsPerMinute** setting.</span></span></p><p><span data-ttu-id="bf97d-231">Quando si ospita *out-of-process*: il modulo tenta di riavviare il processo quando riceve una nuova richiesta e continua a provare a riavviare il processo per le successive richieste in ingresso, a meno che l'app non riesca ad avviare **rapidFailsPerMinute** numero di volte nell'ultimo minuto in sequenza.</span><span class="sxs-lookup"><span data-stu-id="bf97d-231">When hosting *out-of-process*: The module attempts to relaunch the process when it receives a new request and continues to attempt to restart the process on subsequent incoming requests unless the app fails to start **rapidFailsPerMinute** number of times in the last rolling minute.</span></span></p><p><span data-ttu-id="bf97d-232">Un valore pari a 0 (zero) **non** è considerato un timeout infinito.</span><span class="sxs-lookup"><span data-stu-id="bf97d-232">A value of 0 (zero) is **not** considered an infinite timeout.</span></span></p> | <span data-ttu-id="bf97d-233">Valore predefinito: `120`</span><span class="sxs-lookup"><span data-stu-id="bf97d-233">Default: `120`</span></span><br><span data-ttu-id="bf97d-234">Min: `0`</span><span class="sxs-lookup"><span data-stu-id="bf97d-234">Min: `0`</span></span><br><span data-ttu-id="bf97d-235">Max: `3600`</span><span class="sxs-lookup"><span data-stu-id="bf97d-235">Max: `3600`</span></span> |
| `stdoutLogEnabled` | <p><span data-ttu-id="bf97d-236">Attributo booleano facoltativo.</span><span class="sxs-lookup"><span data-stu-id="bf97d-236">Optional Boolean attribute.</span></span></p><p><span data-ttu-id="bf97d-237">Se true, **stdout** e **stderr** per il processo specificato in **processPath** vengono reindirizzati al file specificato in **stdoutLogFile**.</span><span class="sxs-lookup"><span data-stu-id="bf97d-237">If true, **stdout** and **stderr** for the process specified in **processPath** are redirected to the file specified in **stdoutLogFile**.</span></span></p> | `false` |
| `stdoutLogFile` | <p><span data-ttu-id="bf97d-238">Attributo stringa facoltativo.</span><span class="sxs-lookup"><span data-stu-id="bf97d-238">Optional string attribute.</span></span></p><p><span data-ttu-id="bf97d-239">Specifica il percorso relativo o assoluto per cui vengono registrati **stdout** e **stderr** dal processo specificato in **processPath**.</span><span class="sxs-lookup"><span data-stu-id="bf97d-239">Specifies the relative or absolute file path for which **stdout** and **stderr** from the process specified in **processPath** are logged.</span></span> <span data-ttu-id="bf97d-240">I percorsi relativi sono relativi alla radice del sito.</span><span class="sxs-lookup"><span data-stu-id="bf97d-240">Relative paths are relative to the root of the site.</span></span> <span data-ttu-id="bf97d-241">Qualsiasi percorso che inizia con `.` è relativo al sito radice e tutti gli altri percorsi vengono trattati come percorsi assoluti.</span><span class="sxs-lookup"><span data-stu-id="bf97d-241">Any path starting with `.` are relative to the site root and all other paths are treated as absolute paths.</span></span> <span data-ttu-id="bf97d-242">Le eventuali cartelle specificate nel percorso vengono create dal modulo quando viene creato il file di log.</span><span class="sxs-lookup"><span data-stu-id="bf97d-242">Any folders provided in the path are created by the module when the log file is created.</span></span> <span data-ttu-id="bf97d-243">L'utilizzo di delimitatori di sottolineatura, timestamp, ID processo ed estensione di file ( `.log` ) viene aggiunto all'ultimo segmento del percorso **stdoutLogFile** .</span><span class="sxs-lookup"><span data-stu-id="bf97d-243">Using underscore delimiters, a timestamp, process ID, and file extension (`.log`) are added to the last segment of the **stdoutLogFile** path.</span></span> <span data-ttu-id="bf97d-244">Se `.\logs\stdout` viene fornito come valore, un esempio di log stdout viene salvato come `stdout_20180205194132_1934.log` nella `logs` cartella quando viene salvato il 2/5/2018 alle 19:41:32 con l'ID di processo 1934.</span><span class="sxs-lookup"><span data-stu-id="bf97d-244">If `.\logs\stdout` is supplied as a value, an example stdout log is saved as `stdout_20180205194132_1934.log` in the `logs` folder when saved on 2/5/2018 at 19:41:32 with a process ID of 1934.</span></span></p> | `aspnetcore-stdout` |

### <a name="set-environment-variables"></a><span data-ttu-id="bf97d-245">Impostare le variabili di ambiente</span><span class="sxs-lookup"><span data-stu-id="bf97d-245">Set environment variables</span></span>

<span data-ttu-id="bf97d-246">È possibile specificare le variabili di ambiente per il processo nell'attributo `processPath`.</span><span class="sxs-lookup"><span data-stu-id="bf97d-246">Environment variables can be specified for the process in the `processPath` attribute.</span></span> <span data-ttu-id="bf97d-247">Specificare una variabile di ambiente con l'elemento figlio `<environmentVariable>` di un elemento della raccolta `<environmentVariables>`.</span><span class="sxs-lookup"><span data-stu-id="bf97d-247">Specify an environment variable with the `<environmentVariable>` child element of an `<environmentVariables>` collection element.</span></span> <span data-ttu-id="bf97d-248">Le variabili di ambiente impostate in questa sezione hanno la precedenza sulle variabili di ambiente di sistema.</span><span class="sxs-lookup"><span data-stu-id="bf97d-248">Environment variables set in this section take precedence over system environment variables.</span></span>

<span data-ttu-id="bf97d-249">Nell'esempio seguente vengono impostate due variabili di ambiente in `web.config` .</span><span class="sxs-lookup"><span data-stu-id="bf97d-249">The following example sets two environment variables in `web.config`.</span></span> <span data-ttu-id="bf97d-250">`ASPNETCORE_ENVIRONMENT` configura l'ambiente dell'app su `Development`.</span><span class="sxs-lookup"><span data-stu-id="bf97d-250">`ASPNETCORE_ENVIRONMENT` configures the app's environment to `Development`.</span></span> <span data-ttu-id="bf97d-251">Uno sviluppatore può impostare temporaneamente questo valore nel `web.config` file per forzare il caricamento della [pagina delle eccezioni](xref:fundamentals/error-handling) per gli sviluppatori durante il debug di un'eccezione dell'app.</span><span class="sxs-lookup"><span data-stu-id="bf97d-251">A developer may temporarily set this value in the `web.config` file in order to force the [Developer Exception Page](xref:fundamentals/error-handling) to load when debugging an app exception.</span></span> <span data-ttu-id="bf97d-252">`CONFIG_DIR` è un esempio di variabile di ambiente definita dall'utente, in cui lo sviluppatore ha scritto il codice che legge il valore all'avvio in modo da formare un percorso per il caricamento del file di configurazione dell'app.</span><span class="sxs-lookup"><span data-stu-id="bf97d-252">`CONFIG_DIR` is an example of a user-defined environment variable, where the developer has written code that reads the value on startup to form a path for loading the app's configuration file.</span></span>

```xml
<aspNetCore processPath="dotnet"
      arguments=".\MyApp.dll"
      stdoutLogEnabled="false"
      stdoutLogFile=".\logs\stdout"
      hostingModel="inprocess">
  <environmentVariables>
    <environmentVariable name="ASPNETCORE_ENVIRONMENT" value="Development" />
    <environmentVariable name="CONFIG_DIR" value="f:\application_config" />
  </environmentVariables>
</aspNetCore>
```

> [!NOTE]
> <span data-ttu-id="bf97d-253">Un'alternativa all'impostazione dell'ambiente direttamente in `web.config` consiste nell'includere la `<EnvironmentName>` proprietà nel file di progetto o nel [profilo di pubblicazione `.pubxml` ](xref:host-and-deploy/visual-studio-publish-profiles) .</span><span class="sxs-lookup"><span data-stu-id="bf97d-253">An alternative to setting the environment directly in `web.config` is to include the `<EnvironmentName>` property in the [publish profile (`.pubxml`)](xref:host-and-deploy/visual-studio-publish-profiles) or project file.</span></span> <span data-ttu-id="bf97d-254">Questo approccio imposta l'ambiente in `web.config` quando viene pubblicato il progetto:</span><span class="sxs-lookup"><span data-stu-id="bf97d-254">This approach sets the environment in `web.config` when the project is published:</span></span>
>
> ```xml
> <PropertyGroup>
>   <EnvironmentName>Development</EnvironmentName>
> </PropertyGroup>
> ```

> [!WARNING]
> <span data-ttu-id="bf97d-255">Impostare la variabile di ambiente `ASPNETCORE_ENVIRONMENT` su `Development` solo in server di gestione temporanea e test che non sono accessibili da reti non attendibili, ad esempio Internet.</span><span class="sxs-lookup"><span data-stu-id="bf97d-255">Only set the `ASPNETCORE_ENVIRONMENT` environment variable to `Development` on staging and testing servers that aren't accessible to untrusted networks, such as the Internet.</span></span>

## `app_offline.htm`

<span data-ttu-id="bf97d-256">Se viene rilevato un file con il nome `app_offline.htm` nella directory radice di un'app, il modulo ASP.NET Core tenta di arrestare normalmente l'app e di arrestare l'elaborazione delle richieste in ingresso.</span><span class="sxs-lookup"><span data-stu-id="bf97d-256">If a file with the name `app_offline.htm` is detected in the root directory of an app, the ASP.NET Core Module attempts to gracefully shutdown the app and stop processing incoming requests.</span></span> <span data-ttu-id="bf97d-257">Se l'app è ancora in esecuzione dopo il numero di secondi definito in `shutdownTimeLimit`, il modulo ASP.NET Core termina il processo in esecuzione.</span><span class="sxs-lookup"><span data-stu-id="bf97d-257">If the app is still running after the number of seconds defined in `shutdownTimeLimit`, the ASP.NET Core Module kills the running process.</span></span>

<span data-ttu-id="bf97d-258">Mentre il `app_offline.htm` file è presente, il modulo ASP.NET Core risponde alle richieste inviando di nuovo il contenuto del `app_offline.htm` file.</span><span class="sxs-lookup"><span data-stu-id="bf97d-258">While the `app_offline.htm` file is present, the ASP.NET Core Module responds to requests by sending back the contents of the `app_offline.htm` file.</span></span> <span data-ttu-id="bf97d-259">Quando il `app_offline.htm` file viene rimosso, la richiesta successiva avvia l'app.</span><span class="sxs-lookup"><span data-stu-id="bf97d-259">When the `app_offline.htm` file is removed, the next request starts the app.</span></span>

<span data-ttu-id="bf97d-260">Quando si usa il modello di hosting out-of-process, l'app potrebbe non arrestarsi immediatamente se una connessione è aperta.</span><span class="sxs-lookup"><span data-stu-id="bf97d-260">When using the out-of-process hosting model, the app might not shut down immediately if there's an open connection.</span></span> <span data-ttu-id="bf97d-261">Ad esempio, una connessione WebSocket potrebbe ritardare l'arresto dell'app.</span><span class="sxs-lookup"><span data-stu-id="bf97d-261">For example, a WebSocket connection may delay app shut down.</span></span>

## <a name="start-up-error-page"></a><span data-ttu-id="bf97d-262">Pagina di errore di avvio</span><span class="sxs-lookup"><span data-stu-id="bf97d-262">Start-up error page</span></span>

<span data-ttu-id="bf97d-263">Sia l'hosting In-Process che quello out-of-process producono pagine di errore personalizzate quando non riescono ad avviare l'app.</span><span class="sxs-lookup"><span data-stu-id="bf97d-263">Both in-process and out-of-process hosting produce custom error pages when they fail to start the app.</span></span>

<span data-ttu-id="bf97d-264">Se il modulo ASP.NET Core non riesce a trovare il gestore richieste In-Process o out-of-process, viene visualizzata una tabella codici di stato *500.0 - Errore di caricamento del gestore In-Process/out-of-process*.</span><span class="sxs-lookup"><span data-stu-id="bf97d-264">If the ASP.NET Core Module fails to find either the in-process or out-of-process request handler, a *500.0 - In-Process/Out-Of-Process Handler Load Failure* status code page appears.</span></span>

<span data-ttu-id="bf97d-265">Per l'hosting In-Process, se il modulo ASP.NET Core non riesce ad avviare l'app, viene visualizzata una tabella codici di stato *500.30 - Errore di avvio*.</span><span class="sxs-lookup"><span data-stu-id="bf97d-265">For in-process hosting if the ASP.NET Core Module fails to start the app, a *500.30 - Start Failure* status code page appears.</span></span>

<span data-ttu-id="bf97d-266">Per l'hosting out-of-process, se il modulo ASP.NET Core non riesce ad avviare il processo di back-end o il processo di back-end viene avviato ma non riesce a restare in ascolto sulla porta configurata, verrà visualizzata la tabella codici di stato *502.5 - Errore del processo*.</span><span class="sxs-lookup"><span data-stu-id="bf97d-266">For out-of-process hosting if the ASP.NET Core Module fails to launch the backend process or the backend process starts but fails to listen on the configured port, a *502.5 - Process Failure* status code page appears.</span></span>

<span data-ttu-id="bf97d-267">Per non visualizzare questa pagina e tornare alla tabella codici di stato 5xx predefinita di IIS, usare l'attributo `disableStartUpErrorPage`.</span><span class="sxs-lookup"><span data-stu-id="bf97d-267">To suppress this page and revert to the default IIS 5xx status code page, use the `disableStartUpErrorPage` attribute.</span></span> <span data-ttu-id="bf97d-268">Per altre informazioni sulla configurazione di messaggi di errore personalizzati, vedere [Errori HTTP `<httpErrors>`](/iis/configuration/system.webServer/httpErrors/).</span><span class="sxs-lookup"><span data-stu-id="bf97d-268">For more information on configuring custom error messages, see [HTTP Errors `<httpErrors>`](/iis/configuration/system.webServer/httpErrors/).</span></span>

## <a name="log-creation-and-redirection"></a><span data-ttu-id="bf97d-269">Creazione e reindirizzamento dei log</span><span class="sxs-lookup"><span data-stu-id="bf97d-269">Log creation and redirection</span></span>

<span data-ttu-id="bf97d-270">Il modulo ASP.NET Core reindirizza su disco l'output della console stdout e stderr se sono impostati gli attributi `stdoutLogEnabled` e `stdoutLogFile` dell'elemento `aspNetCore`.</span><span class="sxs-lookup"><span data-stu-id="bf97d-270">The ASP.NET Core Module redirects stdout and stderr console output to disk if the `stdoutLogEnabled` and `stdoutLogFile` attributes of the `aspNetCore` element are set.</span></span> <span data-ttu-id="bf97d-271">Le eventuali le cartelle nel percorso `stdoutLogFile` vengono create dal modulo quando viene creato il file di log.</span><span class="sxs-lookup"><span data-stu-id="bf97d-271">Any folders in the `stdoutLogFile` path are created by the module when the log file is created.</span></span> <span data-ttu-id="bf97d-272">Il pool di app deve avere accesso in scrittura alla posizione in cui vengono scritti i log (usare `IIS AppPool\<app_pool_name>` per specificare l'autorizzazione di scrittura).</span><span class="sxs-lookup"><span data-stu-id="bf97d-272">The app pool must have write access to the location where the logs are written (use `IIS AppPool\<app_pool_name>` to provide write permission).</span></span>

<span data-ttu-id="bf97d-273">I log non vengono ruotati, a meno che non si verifichi il riciclo/riavvio del processo.</span><span class="sxs-lookup"><span data-stu-id="bf97d-273">Logs aren't rotated, unless process recycling/restart occurs.</span></span> <span data-ttu-id="bf97d-274">È responsabilità del provider di servizi di hosting limitare lo spazio su disco usato dai log.</span><span class="sxs-lookup"><span data-stu-id="bf97d-274">It's the responsibility of the hoster to limit the disk space the logs consume.</span></span>

<span data-ttu-id="bf97d-275">L'uso del log stdout è consigliato solo per la risoluzione dei problemi di avvio dell'app durante l'hosting in IIS o quando si usa il [supporto in fase di sviluppo per IIS con Visual Studio](xref:host-and-deploy/iis/development-time-iis-support), non durante il debug in locale e l'esecuzione dell'app con IIS Express.</span><span class="sxs-lookup"><span data-stu-id="bf97d-275">Using the stdout log is only recommended for troubleshooting app startup issues when hosting on IIS or when using [development-time support for IIS with Visual Studio](xref:host-and-deploy/iis/development-time-iis-support), not while debugging locally and running the app with IIS Express.</span></span>

<span data-ttu-id="bf97d-276">Non usare il log stdout per scopi di registrazione generale delle app.</span><span class="sxs-lookup"><span data-stu-id="bf97d-276">Don't use the stdout log for general app logging purposes.</span></span> <span data-ttu-id="bf97d-277">Per la registrazione di routine in un'app ASP.NET Core, usare una libreria di registrazione che limita le dimensioni dei file di log e ne esegue la rotazione.</span><span class="sxs-lookup"><span data-stu-id="bf97d-277">For routine logging in an ASP.NET Core app, use a logging library that limits log file size and rotates logs.</span></span> <span data-ttu-id="bf97d-278">Per altre informazioni, vedere [Provider di registrazione di terze parti](xref:fundamentals/logging/index#third-party-logging-providers).</span><span class="sxs-lookup"><span data-stu-id="bf97d-278">For more information, see [third-party logging providers](xref:fundamentals/logging/index#third-party-logging-providers).</span></span>

<span data-ttu-id="bf97d-279">Un timestamp e l'estensione del file vengono aggiunti automaticamente al momento della creazione del file di log.</span><span class="sxs-lookup"><span data-stu-id="bf97d-279">A timestamp and file extension are added automatically when the log file is created.</span></span> <span data-ttu-id="bf97d-280">Il nome del file di log è costituito dall'aggiunta del timestamp, dell'ID del processo e dell'estensione di file ( `.log` ) all'ultimo segmento del `stdoutLogFile` percorso, in genere `stdout` delimitato da caratteri di sottolineatura.</span><span class="sxs-lookup"><span data-stu-id="bf97d-280">The log file name is composed by appending the timestamp, process ID, and file extension (`.log`) to the last segment of the `stdoutLogFile` path (typically `stdout`) delimited by underscores.</span></span> <span data-ttu-id="bf97d-281">Se il `stdoutLogFile` percorso termina con `stdout` , il nome file di un log per un'app con PID 1934 creato in 2/5/2018 a 19:42:32 è `stdout_20180205194132_1934.log` .</span><span class="sxs-lookup"><span data-stu-id="bf97d-281">If the `stdoutLogFile` path ends with `stdout`, a log for an app with a PID of 1934 created on 2/5/2018 at 19:42:32 has the file name `stdout_20180205194132_1934.log`.</span></span>

<span data-ttu-id="bf97d-282">Se `stdoutLogEnabled` è false, gli errori che si verificano all'avvio dell'app vengono acquisiti ed emessi nel log eventi fino a 30 KB.</span><span class="sxs-lookup"><span data-stu-id="bf97d-282">If `stdoutLogEnabled` is false, errors that occur on app startup are captured and emitted to the event log up to 30 KB.</span></span> <span data-ttu-id="bf97d-283">Dopo l'avvio, tutti i log aggiuntivi vengono rimossi.</span><span class="sxs-lookup"><span data-stu-id="bf97d-283">After startup, all additional logs are discarded.</span></span>

<span data-ttu-id="bf97d-284">Nell'elemento di esempio seguente viene `aspNetCore` configurata la registrazione di stdout nel percorso relativo `.\log\` .</span><span class="sxs-lookup"><span data-stu-id="bf97d-284">The following sample `aspNetCore` element configures stdout logging at the relative path `.\log\`.</span></span> <span data-ttu-id="bf97d-285">Verificare che l'identità dell'utente AppPool disponga dell'autorizzazione di scrittura per il percorso specificato.</span><span class="sxs-lookup"><span data-stu-id="bf97d-285">Confirm that the AppPool user identity has permission to write to the path provided.</span></span>

```xml
<aspNetCore processPath="dotnet"
    arguments=".\MyApp.dll"
    stdoutLogEnabled="true"
    stdoutLogFile=".\logs\stdout"
    hostingModel="inprocess">
</aspNetCore>
```

<span data-ttu-id="bf97d-286">Quando si pubblica un'app per la distribuzione di app Azure Service, il valore viene impostato da Web SDK `stdoutLogFile` `\\?\%home%\LogFiles\stdout` .</span><span class="sxs-lookup"><span data-stu-id="bf97d-286">When publishing an app for Azure App Service deployment, the Web SDK sets the `stdoutLogFile` value to `\\?\%home%\LogFiles\stdout`.</span></span> <span data-ttu-id="bf97d-287">La `%home` variabile di ambiente è predefinita per le app ospitate dal servizio app Azure.</span><span class="sxs-lookup"><span data-stu-id="bf97d-287">The `%home` environment variable is predefined for apps hosted by Azure App Service.</span></span>

<span data-ttu-id="bf97d-288">Per creare regole di filtro di registrazione, vedere le sezioni [configurazione](xref:fundamentals/logging/index#log-filtering) e [filtro dei log](xref:fundamentals/logging/index#log-filtering) della documentazione relativa alla registrazione del ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="bf97d-288">To create logging filter rules, see the [Configuration](xref:fundamentals/logging/index#log-filtering) and [Log filtering](xref:fundamentals/logging/index#log-filtering) sections of the ASP.NET Core logging documentation.</span></span>

<span data-ttu-id="bf97d-289">Per ulteriori informazioni sui formati di percorso, vedere [formati di percorso dei file nei sistemi Windows](/dotnet/standard/io/file-path-formats).</span><span class="sxs-lookup"><span data-stu-id="bf97d-289">For more information on path formats, see [File path formats on Windows systems](/dotnet/standard/io/file-path-formats).</span></span>

## <a name="enhanced-diagnostic-logs"></a><span data-ttu-id="bf97d-290">Log di diagnostica avanzati</span><span class="sxs-lookup"><span data-stu-id="bf97d-290">Enhanced diagnostic logs</span></span>

<span data-ttu-id="bf97d-291">Il modulo ASP.NET Core può essere configurato per restituire log di diagnostica avanzata.</span><span class="sxs-lookup"><span data-stu-id="bf97d-291">The ASP.NET Core Module is configurable to provide enhanced diagnostics logs.</span></span> <span data-ttu-id="bf97d-292">Aggiungere l' `<handlerSettings>` elemento all' `<aspNetCore>` elemento in `web.config` .</span><span class="sxs-lookup"><span data-stu-id="bf97d-292">Add the `<handlerSettings>` element to the `<aspNetCore>` element in `web.config`.</span></span> <span data-ttu-id="bf97d-293">L'impostazione di `debugLevel` su `TRACE` restituisce una maggior fedeltà dei dati diagnostici:</span><span class="sxs-lookup"><span data-stu-id="bf97d-293">Setting the `debugLevel` to `TRACE` exposes a higher fidelity of diagnostic information:</span></span>

```xml
<aspNetCore processPath="dotnet"
    arguments=".\MyApp.dll"
    stdoutLogEnabled="false"
    stdoutLogFile="\\?\%home%\LogFiles\stdout"
    hostingModel="inprocess">
  <handlerSettings>
    <handlerSetting name="debugFile" value=".\logs\aspnetcore-debug.log" />
    <handlerSetting name="debugLevel" value="FILE,TRACE" />
  </handlerSettings>
</aspNetCore>
```

<span data-ttu-id="bf97d-294">Tutte le cartelle nel percorso ( `logs` nell'esempio precedente) vengono create dal modulo quando viene creato il file di log.</span><span class="sxs-lookup"><span data-stu-id="bf97d-294">Any folders in the path (`logs` in the preceding example) are created by the module when the log file is created.</span></span> <span data-ttu-id="bf97d-295">Il pool di applicazioni deve avere accesso in scrittura al percorso in cui vengono scritti i log (usare `IIS AppPool\{APP POOL NAME}` , dove il segnaposto `{APP POOL NAME}` è il nome del pool di applicazioni, per fornire l'autorizzazione di scrittura).</span><span class="sxs-lookup"><span data-stu-id="bf97d-295">The app pool must have write access to the location where the logs are written (use `IIS AppPool\{APP POOL NAME}`, where the placeholder `{APP POOL NAME}` is the app pool name, to provide write permission).</span></span>

<span data-ttu-id="bf97d-296">I valori di livello debug (`debugLevel`) possono includere sia il livello che il percorso.</span><span class="sxs-lookup"><span data-stu-id="bf97d-296">Debug level (`debugLevel`) values can include both the level and the location.</span></span>

<span data-ttu-id="bf97d-297">Livelli (in ordine dal meno dettagliato al più dettagliato):</span><span class="sxs-lookup"><span data-stu-id="bf97d-297">Levels (in order from least to most verbose):</span></span>

* <span data-ttu-id="bf97d-298">ERRORE</span><span class="sxs-lookup"><span data-stu-id="bf97d-298">ERROR</span></span>
* <span data-ttu-id="bf97d-299">WARNING</span><span class="sxs-lookup"><span data-stu-id="bf97d-299">WARNING</span></span>
* <span data-ttu-id="bf97d-300">INFO</span><span class="sxs-lookup"><span data-stu-id="bf97d-300">INFO</span></span>
* <span data-ttu-id="bf97d-301">TRACE</span><span class="sxs-lookup"><span data-stu-id="bf97d-301">TRACE</span></span>

<span data-ttu-id="bf97d-302">Posizioni (sono consentite più posizioni):</span><span class="sxs-lookup"><span data-stu-id="bf97d-302">Locations (multiple locations are permitted):</span></span>

* <span data-ttu-id="bf97d-303">CONSOLE</span><span class="sxs-lookup"><span data-stu-id="bf97d-303">CONSOLE</span></span>
* <span data-ttu-id="bf97d-304">EVENTLOG</span><span class="sxs-lookup"><span data-stu-id="bf97d-304">EVENTLOG</span></span>
* <span data-ttu-id="bf97d-305">FILE</span><span class="sxs-lookup"><span data-stu-id="bf97d-305">FILE</span></span>

<span data-ttu-id="bf97d-306">Le impostazioni del gestore possono essere specificate anche tramite le variabili di ambiente:</span><span class="sxs-lookup"><span data-stu-id="bf97d-306">The handler settings can also be provided via environment variables:</span></span>

* <span data-ttu-id="bf97d-307">`ASPNETCORE_MODULE_DEBUG_FILE`: Percorso del file di log di debug.</span><span class="sxs-lookup"><span data-stu-id="bf97d-307">`ASPNETCORE_MODULE_DEBUG_FILE`: Path to the debug log file.</span></span> <span data-ttu-id="bf97d-308">(Impostazione predefinita: `aspnetcore-debug.log` )</span><span class="sxs-lookup"><span data-stu-id="bf97d-308">(Default: `aspnetcore-debug.log`)</span></span>
* <span data-ttu-id="bf97d-309">`ASPNETCORE_MODULE_DEBUG`: Impostazione del livello di debug.</span><span class="sxs-lookup"><span data-stu-id="bf97d-309">`ASPNETCORE_MODULE_DEBUG`: Debug level setting.</span></span>

> [!WARNING]
> <span data-ttu-id="bf97d-310">**Non** lasciare la registrazione del debug abilitata nella distribuzione per un tempo superiore a quello necessario alla risoluzione di problema.</span><span class="sxs-lookup"><span data-stu-id="bf97d-310">Do **not** leave debug logging enabled in the deployment for longer than required to troubleshoot an issue.</span></span> <span data-ttu-id="bf97d-311">Le dimensioni del log non sono limitate.</span><span class="sxs-lookup"><span data-stu-id="bf97d-311">The size of the log isn't limited.</span></span> <span data-ttu-id="bf97d-312">Se si lascia abilitato il log di debug, lo spazio disponibile su disco può esaurirsi e il server o il servizio app può registrare un arresto anomalo.</span><span class="sxs-lookup"><span data-stu-id="bf97d-312">Leaving the debug log enabled can exhaust the available disk space and crash the server or app service.</span></span>

<span data-ttu-id="bf97d-313">Per un esempio dell'elemento nel file, vedere la pagina relativa alla [configurazione con web.config](#configuration-with-webconfig) `aspNetCore` `web.config` .</span><span class="sxs-lookup"><span data-stu-id="bf97d-313">See [Configuration with web.config](#configuration-with-webconfig) for an example of the `aspNetCore` element in the `web.config` file.</span></span>

## <a name="modify-the-stack-size"></a><span data-ttu-id="bf97d-314">Modificare le dimensioni dello stack</span><span class="sxs-lookup"><span data-stu-id="bf97d-314">Modify the stack size</span></span>

<span data-ttu-id="bf97d-315">*Si applica solo quando si usa il modello di hosting in-process.*</span><span class="sxs-lookup"><span data-stu-id="bf97d-315">*Only applies when using the in-process hosting model.*</span></span>

<span data-ttu-id="bf97d-316">Configurare la dimensione dello stack gestito usando l' `stackSize` impostazione in byte in `web.config` .</span><span class="sxs-lookup"><span data-stu-id="bf97d-316">Configure the managed stack size using the `stackSize` setting in bytes in `web.config`.</span></span> <span data-ttu-id="bf97d-317">Le dimensioni predefinite sono pari a 1.048.576 byte (1 MB).</span><span class="sxs-lookup"><span data-stu-id="bf97d-317">The default size is 1,048,576 bytes (1 MB).</span></span>

```xml
<aspNetCore processPath="dotnet"
    arguments=".\MyApp.dll"
    stdoutLogEnabled="false"
    stdoutLogFile="\\?\%home%\LogFiles\stdout"
    hostingModel="inprocess">
  <handlerSettings>
    <handlerSetting name="stackSize" value="2097152" />
  </handlerSettings>
</aspNetCore>
```

## <a name="proxy-configuration-uses-http-protocol-and-a-pairing-token"></a><span data-ttu-id="bf97d-318">La configurazione del proxy usa il protocollo HTTP e un token di associazione</span><span class="sxs-lookup"><span data-stu-id="bf97d-318">Proxy configuration uses HTTP protocol and a pairing token</span></span>

<span data-ttu-id="bf97d-319">*Si applica solo all'hosting out-of-process.*</span><span class="sxs-lookup"><span data-stu-id="bf97d-319">*Only applies to out-of-process hosting.*</span></span>

<span data-ttu-id="bf97d-320">Il proxy creato tra il modulo ASP.NET Core e Kestrel usa il protocollo HTTP.</span><span class="sxs-lookup"><span data-stu-id="bf97d-320">The proxy created between the ASP.NET Core Module and Kestrel uses the HTTP protocol.</span></span> <span data-ttu-id="bf97d-321">Non vi è alcun rischio di intercettazione del traffico tra il modulo e Kestrel da una posizione esterna al server.</span><span class="sxs-lookup"><span data-stu-id="bf97d-321">There's no risk of eavesdropping the traffic between the module and Kestrel from a location off of the server.</span></span>

<span data-ttu-id="bf97d-322">Viene usato un token di associazione per garantire che le richieste ricevute da Kestrel siano state trasmesse tramite proxy da IIS e non provengano da un'altra origine.</span><span class="sxs-lookup"><span data-stu-id="bf97d-322">A pairing token is used to guarantee that the requests received by Kestrel were proxied by IIS and didn't come from some other source.</span></span> <span data-ttu-id="bf97d-323">Il token di associazione viene creato e impostato in una variabile di ambiente (`ASPNETCORE_TOKEN`) dal modulo.</span><span class="sxs-lookup"><span data-stu-id="bf97d-323">The pairing token is created and set into an environment variable (`ASPNETCORE_TOKEN`) by the module.</span></span> <span data-ttu-id="bf97d-324">Il token di associazione viene impostato anche in un'intestazione (`MS-ASPNETCORE-TOKEN`) in tutte le richieste trasmesse tramite proxy.</span><span class="sxs-lookup"><span data-stu-id="bf97d-324">The pairing token is also set into a header (`MS-ASPNETCORE-TOKEN`) on every proxied request.</span></span> <span data-ttu-id="bf97d-325">Il middleware di IIS controlla ogni richiesta che riceve in modo da confermare che il valore dell'intestazione del token di associazione corrisponda al valore della variabile di ambiente.</span><span class="sxs-lookup"><span data-stu-id="bf97d-325">IIS Middleware checks each request it receives to confirm that the pairing token header value matches the environment variable value.</span></span> <span data-ttu-id="bf97d-326">Se i valori del token non corrispondono, la richiesta viene registrata e rifiutata.</span><span class="sxs-lookup"><span data-stu-id="bf97d-326">If the token values are mismatched, the request is logged and rejected.</span></span> <span data-ttu-id="bf97d-327">La variabile di ambiente del token di associazione e il traffico tra il modulo e Kestrel non sono accessibili da una posizione esterna al server.</span><span class="sxs-lookup"><span data-stu-id="bf97d-327">The pairing token environment variable and the traffic between the module and Kestrel aren't accessible from a location off of the server.</span></span> <span data-ttu-id="bf97d-328">Senza conoscere il valore del token di associazione, un utente malintenzionato non può inviare richieste che ignorano il controllo nel middleware di IIS.</span><span class="sxs-lookup"><span data-stu-id="bf97d-328">Without knowing the pairing token value, an attacker can't submit requests that bypass the check in the IIS Middleware.</span></span>

## <a name="aspnet-core-module-with-an-iis-shared-configuration"></a><span data-ttu-id="bf97d-329">Modulo ASP.NET Core con configurazione condivisa di IIS</span><span class="sxs-lookup"><span data-stu-id="bf97d-329">ASP.NET Core Module with an IIS Shared Configuration</span></span>

<span data-ttu-id="bf97d-330">Il programma di installazione del modulo ASP.NET Core viene eseguito con i privilegi dell'account **TrustedInstaller**.</span><span class="sxs-lookup"><span data-stu-id="bf97d-330">The ASP.NET Core Module installer runs with the privileges of the **TrustedInstaller** account.</span></span> <span data-ttu-id="bf97d-331">Poiché l'account di sistema locale non dispone dell'autorizzazione di modifica per il percorso di condivisione utilizzato dalla configurazione condivisa di IIS, il programma di installazione genera un errore di accesso negato quando si tenta di configurare le impostazioni del modulo nel `applicationHost.config` file nella condivisione.</span><span class="sxs-lookup"><span data-stu-id="bf97d-331">Because the local system account doesn't have modify permission for the share path used by the IIS Shared Configuration, the installer throws an access denied error when attempting to configure the module settings in the `applicationHost.config` file on the share.</span></span>

<span data-ttu-id="bf97d-332">Quando si usa una configurazione condivisa di IIS nello stesso computer dell'installazione di IIS, eseguire il programma di installazione del bundle di hosting ASP.NET Core con il parametro `OPT_NO_SHARED_CONFIG_CHECK` impostato su `1`:</span><span class="sxs-lookup"><span data-stu-id="bf97d-332">When using an IIS Shared Configuration on the same machine as the IIS installation, run the ASP.NET Core Hosting Bundle installer with the `OPT_NO_SHARED_CONFIG_CHECK` parameter set to `1`:</span></span>

```console
dotnet-hosting-{VERSION}.exe OPT_NO_SHARED_CONFIG_CHECK=1
```

<span data-ttu-id="bf97d-333">Quando il percorso della configurazione condivisa non è nello stesso computer dell'installazione di IIS, seguire questa procedura:</span><span class="sxs-lookup"><span data-stu-id="bf97d-333">When the path to the shared configuration isn't on the same machine as the IIS installation, follow these steps:</span></span>

1. <span data-ttu-id="bf97d-334">Disabilitare la configurazione condivisa di IIS.</span><span class="sxs-lookup"><span data-stu-id="bf97d-334">Disable the IIS Shared Configuration.</span></span>
1. <span data-ttu-id="bf97d-335">Eseguire il programma di installazione.</span><span class="sxs-lookup"><span data-stu-id="bf97d-335">Run the installer.</span></span>
1. <span data-ttu-id="bf97d-336">Esportare il file aggiornato nella `applicationHost.config` condivisione.</span><span class="sxs-lookup"><span data-stu-id="bf97d-336">Export the updated `applicationHost.config` file to the share.</span></span>
1. <span data-ttu-id="bf97d-337">Abilitare di nuovo la configurazione condivisa di IIS.</span><span class="sxs-lookup"><span data-stu-id="bf97d-337">Re-enable the IIS Shared Configuration.</span></span>

## <a name="module-version-and-hosting-bundle-installer-logs"></a><span data-ttu-id="bf97d-338">Versione del modulo e log del programma di installazione del bundle di hosting</span><span class="sxs-lookup"><span data-stu-id="bf97d-338">Module version and Hosting Bundle installer logs</span></span>

<span data-ttu-id="bf97d-339">Per determinare la versione del modulo ASP.NET Core installato:</span><span class="sxs-lookup"><span data-stu-id="bf97d-339">To determine the version of the installed ASP.NET Core Module:</span></span>

1. <span data-ttu-id="bf97d-340">Nel sistema di hosting passare a `%windir%\System32\inetsrv` .</span><span class="sxs-lookup"><span data-stu-id="bf97d-340">On the hosting system, navigate to `%windir%\System32\inetsrv`.</span></span>
1. <span data-ttu-id="bf97d-341">Individuare il `aspnetcore.dll` file.</span><span class="sxs-lookup"><span data-stu-id="bf97d-341">Locate the `aspnetcore.dll` file.</span></span>
1. <span data-ttu-id="bf97d-342">Fare clic con il pulsante destro del mouse sul file e scegliere **Proprietà** dal menu di scelta rapida.</span><span class="sxs-lookup"><span data-stu-id="bf97d-342">Right-click the file and select **Properties** from the contextual menu.</span></span>
1. <span data-ttu-id="bf97d-343">Selezionare la scheda **Dettagli** . La versione del **file** e la **versione del prodotto** rappresentano la versione installata del modulo.</span><span class="sxs-lookup"><span data-stu-id="bf97d-343">Select the **Details** tab. The **File version** and **Product version** represent the installed version of the module.</span></span>

<span data-ttu-id="bf97d-344">I log del programma di installazione del bundle di hosting per il modulo sono disponibili in `C:\Users\%UserName%\AppData\Local\Temp` .</span><span class="sxs-lookup"><span data-stu-id="bf97d-344">The Hosting Bundle installer logs for the module are found at `C:\Users\%UserName%\AppData\Local\Temp`.</span></span> <span data-ttu-id="bf97d-345">Il file è denominato `dd_DotNetCoreWinSvrHosting__{TIMESTAMP}_000_AspNetCoreModule_x64.log`.</span><span class="sxs-lookup"><span data-stu-id="bf97d-345">The file is named `dd_DotNetCoreWinSvrHosting__{TIMESTAMP}_000_AspNetCoreModule_x64.log`.</span></span>

## <a name="module-schema-and-configuration-file-locations"></a><span data-ttu-id="bf97d-346">Percorsi dei file di modulo, schema e configurazione</span><span class="sxs-lookup"><span data-stu-id="bf97d-346">Module, schema, and configuration file locations</span></span>

### <a name="module"></a><span data-ttu-id="bf97d-347">Modulo</span><span class="sxs-lookup"><span data-stu-id="bf97d-347">Module</span></span>

<span data-ttu-id="bf97d-348">**IIS (x86/amd64):**</span><span class="sxs-lookup"><span data-stu-id="bf97d-348">**IIS (x86/amd64):**</span></span>

* `%windir%\System32\inetsrv\aspnetcore.dll`

* `%windir%\SysWOW64\inetsrv\aspnetcore.dll`

* `%ProgramFiles%\IIS\Asp.Net Core Module\V2\aspnetcorev2.dll`

* `%ProgramFiles(x86)%\IIS\Asp.Net Core Module\V2\aspnetcorev2.dll`

<span data-ttu-id="bf97d-349">**IIS Express (x86/amd64):**</span><span class="sxs-lookup"><span data-stu-id="bf97d-349">**IIS Express (x86/amd64):**</span></span>

* `%ProgramFiles%\IIS Express\aspnetcore.dll`

* `%ProgramFiles(x86)%\IIS Express\aspnetcore.dll`

* `%ProgramFiles%\IIS Express\Asp.Net Core Module\V2\aspnetcorev2.dll`

* `%ProgramFiles(x86)%\IIS Express\Asp.Net Core Module\V2\aspnetcorev2.dll`

### <a name="schema"></a><span data-ttu-id="bf97d-350">SCHEMA</span><span class="sxs-lookup"><span data-stu-id="bf97d-350">Schema</span></span>

<span data-ttu-id="bf97d-351">**IIS**</span><span class="sxs-lookup"><span data-stu-id="bf97d-351">**IIS**</span></span>

* `%windir%\System32\inetsrv\config\schema\aspnetcore_schema.xml`

* `%windir%\System32\inetsrv\config\schema\aspnetcore_schema_v2.xml`

<span data-ttu-id="bf97d-352">**IIS Express**</span><span class="sxs-lookup"><span data-stu-id="bf97d-352">**IIS Express**</span></span>

* `%ProgramFiles%\IIS Express\config\schema\aspnetcore_schema.xml`

* `%ProgramFiles%\IIS Express\config\schema\aspnetcore_schema_v2.xml`

### <a name="configuration"></a><span data-ttu-id="bf97d-353">Configurazione</span><span class="sxs-lookup"><span data-stu-id="bf97d-353">Configuration</span></span>

<span data-ttu-id="bf97d-354">**IIS**</span><span class="sxs-lookup"><span data-stu-id="bf97d-354">**IIS**</span></span>

* `%windir%\System32\inetsrv\config\applicationHost.config`

<span data-ttu-id="bf97d-355">**IIS Express**</span><span class="sxs-lookup"><span data-stu-id="bf97d-355">**IIS Express**</span></span>

* <span data-ttu-id="bf97d-356">Visual Studio: `{APPLICATION ROOT}\.vs\config\applicationHost.config`</span><span class="sxs-lookup"><span data-stu-id="bf97d-356">Visual Studio: `{APPLICATION ROOT}\.vs\config\applicationHost.config`</span></span>

* <span data-ttu-id="bf97d-357">*iisexpress.exe* CLI `%USERPROFILE%\Documents\IISExpress\config\applicationhost.config`</span><span class="sxs-lookup"><span data-stu-id="bf97d-357">*iisexpress.exe* CLI: `%USERPROFILE%\Documents\IISExpress\config\applicationhost.config`</span></span>

<span data-ttu-id="bf97d-358">È possibile trovare i file eseguendo una ricerca `aspnetcore` nel `applicationHost.config` file.</span><span class="sxs-lookup"><span data-stu-id="bf97d-358">The files can be found by searching for `aspnetcore` in the `applicationHost.config` file.</span></span>

::: moniker-end

::: moniker range="= aspnetcore-2.2"

<span data-ttu-id="bf97d-359">Il modulo ASP.NET Core è un modulo IIS nativo collegato alla pipeline di IIS per eseguire le operazioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="bf97d-359">The ASP.NET Core Module is a native IIS module that plugs into the IIS pipeline to either:</span></span>

* <span data-ttu-id="bf97d-360">Ospitare un app ASP.NET Core all'interno del processo di lavoro IIS (`w3wp.exe`), denominato [modello di hosting in-process](#in-process-hosting-model).</span><span class="sxs-lookup"><span data-stu-id="bf97d-360">Host an ASP.NET Core app inside of the IIS worker process (`w3wp.exe`), called the [in-process hosting model](#in-process-hosting-model).</span></span>
* <span data-ttu-id="bf97d-361">Inoltrare le richieste Web all'app back-end ASP.NET Core che esegue il [server Kestrel](xref:fundamentals/servers/kestrel), denominato [modello di hosting out-of-process](#out-of-process-hosting-model).</span><span class="sxs-lookup"><span data-stu-id="bf97d-361">Forward web requests to a backend ASP.NET Core app running the [Kestrel server](xref:fundamentals/servers/kestrel), called the [out-of-process hosting model](#out-of-process-hosting-model).</span></span>

<span data-ttu-id="bf97d-362">Versioni supportate di Windows:</span><span class="sxs-lookup"><span data-stu-id="bf97d-362">Supported Windows versions:</span></span>

* <span data-ttu-id="bf97d-363">Windows 7 o versione successiva</span><span class="sxs-lookup"><span data-stu-id="bf97d-363">Windows 7 or later</span></span>
* <span data-ttu-id="bf97d-364">Windows Server 2008 R2 o versione successiva</span><span class="sxs-lookup"><span data-stu-id="bf97d-364">Windows Server 2008 R2 or later</span></span>

<span data-ttu-id="bf97d-365">In caso di hosting in-process, il modulo usa un'implementazione di server in-process per IIS detta server HTTP di IIS (`IISHttpServer`).</span><span class="sxs-lookup"><span data-stu-id="bf97d-365">When hosting in-process, the module uses an in-process server implementation for IIS, called IIS HTTP Server (`IISHttpServer`).</span></span>

<span data-ttu-id="bf97d-366">In caso di hosting out-of-process, il modulo funziona solo con Kestrel.</span><span class="sxs-lookup"><span data-stu-id="bf97d-366">When hosting out-of-process, the module only works with Kestrel.</span></span> <span data-ttu-id="bf97d-367">Il modulo non funziona con [HTTP.sys](xref:fundamentals/servers/httpsys).</span><span class="sxs-lookup"><span data-stu-id="bf97d-367">The module doesn't function with [HTTP.sys](xref:fundamentals/servers/httpsys).</span></span>

## <a name="hosting-models"></a><span data-ttu-id="bf97d-368">Modelli di hosting</span><span class="sxs-lookup"><span data-stu-id="bf97d-368">Hosting models</span></span>

### <a name="in-process-hosting-model"></a><span data-ttu-id="bf97d-369">Modello di hosting in-process</span><span class="sxs-lookup"><span data-stu-id="bf97d-369">In-process hosting model</span></span>

<span data-ttu-id="bf97d-370">Per configurare un'app per l'hosting in-process, aggiungere la proprietà `<AspNetCoreHostingModel>` al file di progetto dell'app usando il valore `InProcess` (l'hosting out-of-process viene impostato con `OutOfProcess`):</span><span class="sxs-lookup"><span data-stu-id="bf97d-370">To configure an app for in-process hosting, add the `<AspNetCoreHostingModel>` property to the app's project file with a value of `InProcess` (out-of-process hosting is set with `OutOfProcess`):</span></span>

```xml
<PropertyGroup>
  <AspNetCoreHostingModel>InProcess</AspNetCoreHostingModel>
</PropertyGroup>
```

<span data-ttu-id="bf97d-371">Il modello di hosting In-Process non è supportato per le app ASP.NET Core destinate a .NET Framework.</span><span class="sxs-lookup"><span data-stu-id="bf97d-371">The in-process hosting model isn't supported for ASP.NET Core apps that target the .NET Framework.</span></span>

<span data-ttu-id="bf97d-372">Il valore di non `<AspNetCoreHostingModel>` fa distinzione tra maiuscole e minuscole, pertanto `inprocess` e `outofprocess` sono valori validi.</span><span class="sxs-lookup"><span data-stu-id="bf97d-372">The value of `<AspNetCoreHostingModel>` is case insensitive, so `inprocess` and `outofprocess` are valid values.</span></span>

<span data-ttu-id="bf97d-373">Se la proprietà `<AspNetCoreHostingModel>` non è presente nel file, il valore predefinito è `OutOfProcess`.</span><span class="sxs-lookup"><span data-stu-id="bf97d-373">If the `<AspNetCoreHostingModel>` property isn't present in the file, the default value is `OutOfProcess`.</span></span>

<span data-ttu-id="bf97d-374">In caso di hosting in-process, vengono applicate le caratteristiche seguenti:</span><span class="sxs-lookup"><span data-stu-id="bf97d-374">The following characteristics apply when hosting in-process:</span></span>

* <span data-ttu-id="bf97d-375">Viene usato un server HTTP di IIS (`IISHttpServer`) al posto del server [Kestrel](xref:fundamentals/servers/kestrel).</span><span class="sxs-lookup"><span data-stu-id="bf97d-375">IIS HTTP Server (`IISHttpServer`) is used instead of [Kestrel](xref:fundamentals/servers/kestrel) server.</span></span> <span data-ttu-id="bf97d-376">Per in-process, [CreateDefaultBuilder](xref:fundamentals/host/web-host#set-up-a-host) chiama <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderIISExtensions.UseIIS*> per:</span><span class="sxs-lookup"><span data-stu-id="bf97d-376">For in-process, [CreateDefaultBuilder](xref:fundamentals/host/web-host#set-up-a-host) calls <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderIISExtensions.UseIIS*> to:</span></span>

  * <span data-ttu-id="bf97d-377">Registrare `IISHttpServer`.</span><span class="sxs-lookup"><span data-stu-id="bf97d-377">Register the `IISHttpServer`.</span></span>
  * <span data-ttu-id="bf97d-378">Configurare la porta e il percorso di base su cui il server deve eseguire l'ascolto in caso di esecuzione dietro il modulo ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="bf97d-378">Configure the port and base path the server should listen on when running behind the ASP.NET Core Module.</span></span>
  * <span data-ttu-id="bf97d-379">Configurare l'host per l'acquisizione degli errori di avvio.</span><span class="sxs-lookup"><span data-stu-id="bf97d-379">Configure the host to capture startup errors.</span></span>

* <span data-ttu-id="bf97d-380">L'[attributo requestTimeout ](#attributes-of-the-aspnetcore-element) non viene applicato all'hosting in-process.</span><span class="sxs-lookup"><span data-stu-id="bf97d-380">The [requestTimeout attribute](#attributes-of-the-aspnetcore-element) doesn't apply to in-process hosting.</span></span>

* <span data-ttu-id="bf97d-381">Non è supportata la condivisione di un pool di app tra le app.</span><span class="sxs-lookup"><span data-stu-id="bf97d-381">Sharing an app pool among apps isn't supported.</span></span> <span data-ttu-id="bf97d-382">Usare un pool di app per ogni app.</span><span class="sxs-lookup"><span data-stu-id="bf97d-382">Use one app pool per app.</span></span>

* <span data-ttu-id="bf97d-383">Quando si usa la [Distribuzione Web ](/iis/publish/using-web-deploy/introduction-to-web-deploy) o si inserisce manualmente un [file app_offline.htm nella distribuzione](xref:host-and-deploy/iis/index#locked-deployment-files), l'app potrebbe non arrestarsi immediatamente se una connessione è aperta.</span><span class="sxs-lookup"><span data-stu-id="bf97d-383">When using [Web Deploy](/iis/publish/using-web-deploy/introduction-to-web-deploy) or manually placing an [app_offline.htm file in the deployment](xref:host-and-deploy/iis/index#locked-deployment-files), the app might not shut down immediately if there's an open connection.</span></span> <span data-ttu-id="bf97d-384">Ad esempio, una connessione WebSocket può ritardare l'arresto dell'app.</span><span class="sxs-lookup"><span data-stu-id="bf97d-384">For example, a websocket connection may delay app shut down.</span></span>

* <span data-ttu-id="bf97d-385">L'architettura, vale a dire il numero di bit dell'app, e il runtime installato (x64 o x86) devono corrispondere all'architettura del pool di app.</span><span class="sxs-lookup"><span data-stu-id="bf97d-385">The architecture (bitness) of the app and installed runtime (x64 or x86) must match the architecture of the app pool.</span></span>

* <span data-ttu-id="bf97d-386">Le disconnessioni del client vengono rilevate.</span><span class="sxs-lookup"><span data-stu-id="bf97d-386">Client disconnects are detected.</span></span> <span data-ttu-id="bf97d-387">Il token di annullamento [HttpContext.RequestAborted](xref:Microsoft.AspNetCore.Http.HttpContext.RequestAborted*) viene cancellato quando il client si disconnette.</span><span class="sxs-lookup"><span data-stu-id="bf97d-387">The [HttpContext.RequestAborted](xref:Microsoft.AspNetCore.Http.HttpContext.RequestAborted*) cancellation token is cancelled when the client disconnects.</span></span>

* <span data-ttu-id="bf97d-388">In ASP.NET Core 2.2.1 o versioni precedenti, <xref:System.IO.Directory.GetCurrentDirectory*> restituisce la directory di lavoro del processo avviato da IIS invece che la directory dell'app (ad esempio, *C:\Windows\System32\inetsrv* per *w3wp.exe*).</span><span class="sxs-lookup"><span data-stu-id="bf97d-388">In ASP.NET Core 2.2.1 or earlier, <xref:System.IO.Directory.GetCurrentDirectory*> returns the worker directory of the process started by IIS rather than the app's directory (for example, *C:\Windows\System32\inetsrv* for *w3wp.exe*).</span></span>

  <span data-ttu-id="bf97d-389">Per vedere il codice di esempio che imposta la directory corrente dell'app, vedere la [classe CurrentDirectoryHelpers](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/host-and-deploy/aspnet-core-module/samples_snapshot/2.x/CurrentDirectoryHelpers.cs).</span><span class="sxs-lookup"><span data-stu-id="bf97d-389">For sample code that sets the app's current directory, see the [CurrentDirectoryHelpers class](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/host-and-deploy/aspnet-core-module/samples_snapshot/2.x/CurrentDirectoryHelpers.cs).</span></span> <span data-ttu-id="bf97d-390">Chiamare il metodo `SetCurrentDirectory` .</span><span class="sxs-lookup"><span data-stu-id="bf97d-390">Call the `SetCurrentDirectory` method.</span></span> <span data-ttu-id="bf97d-391">Le chiamate successive a <xref:System.IO.Directory.GetCurrentDirectory*> specificano la directory dell'app.</span><span class="sxs-lookup"><span data-stu-id="bf97d-391">Subsequent calls to <xref:System.IO.Directory.GetCurrentDirectory*> provide the app's directory.</span></span>

* <span data-ttu-id="bf97d-392">In caso di hosting in-process, <xref:Microsoft.AspNetCore.Authentication.AuthenticationService.AuthenticateAsync*> non viene chiamato internamente per inizializzare un utente.</span><span class="sxs-lookup"><span data-stu-id="bf97d-392">When hosting in-process, <xref:Microsoft.AspNetCore.Authentication.AuthenticationService.AuthenticateAsync*> isn't called internally to initialize a user.</span></span> <span data-ttu-id="bf97d-393">Pertanto, un'implementazione di <xref:Microsoft.AspNetCore.Authentication.IClaimsTransformation> usate per trasformare le attestazioni dopo ogni autenticazione non viene attivata per impostazione predefinita.</span><span class="sxs-lookup"><span data-stu-id="bf97d-393">Therefore, an <xref:Microsoft.AspNetCore.Authentication.IClaimsTransformation> implementation used to transform claims after every authentication isn't activated by default.</span></span> <span data-ttu-id="bf97d-394">Quando si trasformano le attestazioni con un'implementazione di <xref:Microsoft.AspNetCore.Authentication.IClaimsTransformation>, chiamare <xref:Microsoft.Extensions.DependencyInjection.AuthenticationServiceCollectionExtensions.AddAuthentication*> per aggiungere i servizi di autenticazione:</span><span class="sxs-lookup"><span data-stu-id="bf97d-394">When transforming claims with an <xref:Microsoft.AspNetCore.Authentication.IClaimsTransformation> implementation, call <xref:Microsoft.Extensions.DependencyInjection.AuthenticationServiceCollectionExtensions.AddAuthentication*> to add authentication services:</span></span>

  ```csharp
  public void ConfigureServices(IServiceCollection services)
  {
      services.AddTransient<IClaimsTransformation, ClaimsTransformer>();
      services.AddAuthentication(IISServerDefaults.AuthenticationScheme);
  }

  public void Configure(IApplicationBuilder app)
  {
      app.UseAuthentication();
  }
  ```

### <a name="out-of-process-hosting-model"></a><span data-ttu-id="bf97d-395">Modello di hosting out-of-process</span><span class="sxs-lookup"><span data-stu-id="bf97d-395">Out-of-process hosting model</span></span>

<span data-ttu-id="bf97d-396">Per configurare un'app per l'hosting out-of-process, usare uno dei due approcci seguenti nel file di progetto:</span><span class="sxs-lookup"><span data-stu-id="bf97d-396">To configure an app for out-of-process hosting, use either of the following approaches in the project file:</span></span>

* <span data-ttu-id="bf97d-397">Non specificare la proprietà `<AspNetCoreHostingModel>`.</span><span class="sxs-lookup"><span data-stu-id="bf97d-397">Don't specify the `<AspNetCoreHostingModel>` property.</span></span> <span data-ttu-id="bf97d-398">Se la proprietà `<AspNetCoreHostingModel>` non è presente nel file, il valore predefinito è `OutOfProcess`.</span><span class="sxs-lookup"><span data-stu-id="bf97d-398">If the `<AspNetCoreHostingModel>` property isn't present in the file, the default value is `OutOfProcess`.</span></span>
* <span data-ttu-id="bf97d-399">Impostare il valore della proprietà `<AspNetCoreHostingModel>` su `OutOfProcess` (l'hosting in-process è impostato con `InProcess`):</span><span class="sxs-lookup"><span data-stu-id="bf97d-399">Set the value of the `<AspNetCoreHostingModel>` property to `OutOfProcess` (in-process hosting is set with `InProcess`):</span></span>

```xml
<PropertyGroup>
  <AspNetCoreHostingModel>OutOfProcess</AspNetCoreHostingModel>
</PropertyGroup>
```

<span data-ttu-id="bf97d-400">Il valore non fa distinzione tra maiuscole e minuscole, pertanto `inprocess` e `outofprocess` sono valori validi.</span><span class="sxs-lookup"><span data-stu-id="bf97d-400">The value is case insensitive, so `inprocess` and `outofprocess` are valid values.</span></span>

<span data-ttu-id="bf97d-401">Viene usato il server [Kestrel](xref:fundamentals/servers/kestrel) al posto di un server HTTP di IIS (`IISHttpServer`).</span><span class="sxs-lookup"><span data-stu-id="bf97d-401">[Kestrel](xref:fundamentals/servers/kestrel) server is used instead of IIS HTTP Server (`IISHttpServer`).</span></span>

<span data-ttu-id="bf97d-402">Per out-of-process, [CreateDefaultBuilder](xref:fundamentals/host/web-host#set-up-a-host) chiama <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderIISExtensions.UseIISIntegration*> per:</span><span class="sxs-lookup"><span data-stu-id="bf97d-402">For out-of-process, [CreateDefaultBuilder](xref:fundamentals/host/web-host#set-up-a-host) calls <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderIISExtensions.UseIISIntegration*> to:</span></span>

* <span data-ttu-id="bf97d-403">Configurare la porta e il percorso di base su cui il server deve eseguire l'ascolto in caso di esecuzione dietro il modulo ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="bf97d-403">Configure the port and base path the server should listen on when running behind the ASP.NET Core Module.</span></span>
* <span data-ttu-id="bf97d-404">Configurare l'host per l'acquisizione degli errori di avvio.</span><span class="sxs-lookup"><span data-stu-id="bf97d-404">Configure the host to capture startup errors.</span></span>

### <a name="hosting-model-changes"></a><span data-ttu-id="bf97d-405">Modifiche del modello di hosting</span><span class="sxs-lookup"><span data-stu-id="bf97d-405">Hosting model changes</span></span>

<span data-ttu-id="bf97d-406">Se l'impostazione `hostingModel` viene modificata nel file *web.config* (descritto nella sezione [Configurazione con web.config](#configuration-with-webconfig)), il modulo ricicla il processo di lavoro per IIS.</span><span class="sxs-lookup"><span data-stu-id="bf97d-406">If the `hostingModel` setting is changed in the *web.config* file (explained in the [Configuration with web.config](#configuration-with-webconfig) section), the module recycles the worker process for IIS.</span></span>

<span data-ttu-id="bf97d-407">Per IIS Express, il modulo non ricicla il processo di lavoro. Attiva invece un arresto normale del processo di IIS Express corrente.</span><span class="sxs-lookup"><span data-stu-id="bf97d-407">For IIS Express, the module doesn't recycle the worker process but instead triggers a graceful shutdown of the current IIS Express process.</span></span> <span data-ttu-id="bf97d-408">La richiesta successiva all'app attiva un nuovo processo di IIS Express.</span><span class="sxs-lookup"><span data-stu-id="bf97d-408">The next request to the app spawns a new IIS Express process.</span></span>

### <a name="process-name"></a><span data-ttu-id="bf97d-409">Nome del processo</span><span class="sxs-lookup"><span data-stu-id="bf97d-409">Process name</span></span>

<span data-ttu-id="bf97d-410">`Process.GetCurrentProcess().ProcessName` dichiara `w3wp`/`iisexpress` (In-Process) o `dotnet` (out-of-process).</span><span class="sxs-lookup"><span data-stu-id="bf97d-410">`Process.GetCurrentProcess().ProcessName` reports `w3wp`/`iisexpress` (in-process) or `dotnet` (out-of-process).</span></span>

<span data-ttu-id="bf97d-411">Molti moduli nativi, ad esempio l'autenticazione di Windows, rimangono attivi.</span><span class="sxs-lookup"><span data-stu-id="bf97d-411">Many native modules, such as Windows Authentication, remain active.</span></span> <span data-ttu-id="bf97d-412">Per altre informazioni sui moduli IIS attivi con il modulo ASP.NET Core, vedere <xref:host-and-deploy/iis/modules>.</span><span class="sxs-lookup"><span data-stu-id="bf97d-412">To learn more about IIS modules active with the ASP.NET Core Module, see <xref:host-and-deploy/iis/modules>.</span></span>

<span data-ttu-id="bf97d-413">Il modulo ASP.NET Core può anche:</span><span class="sxs-lookup"><span data-stu-id="bf97d-413">The ASP.NET Core Module can also:</span></span>

* <span data-ttu-id="bf97d-414">Impostare variabili di ambiente per il processo di lavoro.</span><span class="sxs-lookup"><span data-stu-id="bf97d-414">Set environment variables for the worker process.</span></span>
* <span data-ttu-id="bf97d-415">Registrare output stdout in una risorsa di archiviazione di file per la risoluzione di problemi di avvio.</span><span class="sxs-lookup"><span data-stu-id="bf97d-415">Log stdout output to file storage for troubleshooting startup issues.</span></span>
* <span data-ttu-id="bf97d-416">Inoltrare token di autenticazione di Windows.</span><span class="sxs-lookup"><span data-stu-id="bf97d-416">Forward Windows authentication tokens.</span></span>

## <a name="how-to-install-and-use-the-aspnet-core-module"></a><span data-ttu-id="bf97d-417">Come installare e usare il modulo ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="bf97d-417">How to install and use the ASP.NET Core Module</span></span>

<span data-ttu-id="bf97d-418">Per istruzioni su come installare e usare il modulo ASP.NET Core, vedere [Installare il bundle di hosting .NET Core](xref:host-and-deploy/iis/index#install-the-net-core-hosting-bundle).</span><span class="sxs-lookup"><span data-stu-id="bf97d-418">For instructions on how to install the ASP.NET Core Module, see [Install the .NET Core Hosting Bundle](xref:host-and-deploy/iis/index#install-the-net-core-hosting-bundle).</span></span>

## <a name="configuration-with-webconfig"></a><span data-ttu-id="bf97d-419">Configurazione con web.config</span><span class="sxs-lookup"><span data-stu-id="bf97d-419">Configuration with web.config</span></span>

<span data-ttu-id="bf97d-420">Il modulo ASP.NET Core viene configurato tramite la sezione `aspNetCore` del nodo `system.webServer` nel file *web.config* del sito.</span><span class="sxs-lookup"><span data-stu-id="bf97d-420">The ASP.NET Core Module is configured with the `aspNetCore` section of the `system.webServer` node in the site's *web.config* file.</span></span>

<span data-ttu-id="bf97d-421">Il file *web.config* seguente viene pubblicato per una [distribuzione dipendente dal framework](/dotnet/articles/core/deploying/#framework-dependent-deployments-fdd) e configura il modulo ASP.NET Core per gestire le richieste del sito:</span><span class="sxs-lookup"><span data-stu-id="bf97d-421">The following *web.config* file is published for a [framework-dependent deployment](/dotnet/articles/core/deploying/#framework-dependent-deployments-fdd) and configures the ASP.NET Core Module to handle site requests:</span></span>

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <location path="." inheritInChildApplications="false">
    <system.webServer>
      <handlers>
        <add name="aspNetCore" path="*" verb="*" modules="AspNetCoreModuleV2" resourceType="Unspecified" />
      </handlers>
      <aspNetCore processPath="dotnet"
                  arguments=".\MyApp.dll"
                  stdoutLogEnabled="false"
                  stdoutLogFile=".\logs\stdout"
                  hostingModel="inprocess" />
    </system.webServer>
  </location>
</configuration>
```

<span data-ttu-id="bf97d-422">Il file *web.config* seguente viene pubblicato per una [distribuzione autonoma](/dotnet/articles/core/deploying/#self-contained-deployments-scd):</span><span class="sxs-lookup"><span data-stu-id="bf97d-422">The following *web.config* is published for a [self-contained deployment](/dotnet/articles/core/deploying/#self-contained-deployments-scd):</span></span>

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <location path="." inheritInChildApplications="false">
    <system.webServer>
      <handlers>
        <add name="aspNetCore" path="*" verb="*" modules="AspNetCoreModuleV2" resourceType="Unspecified" />
      </handlers>
      <aspNetCore processPath=".\MyApp.exe"
                  stdoutLogEnabled="false"
                  stdoutLogFile=".\logs\stdout"
                  hostingModel="inprocess" />
    </system.webServer>
  </location>
</configuration>
```

<span data-ttu-id="bf97d-423">La <xref:System.Configuration.SectionInformation.InheritInChildApplications*> proprietà è impostata su `false` per indicare che le impostazioni specificate all'interno dell' [\<location>](/iis/manage/managing-your-configuration-settings/understanding-iis-configuration-delegation#the-concept-of-location) elemento non vengono ereditate da app che si trovano in una sottodirectory dell'app.</span><span class="sxs-lookup"><span data-stu-id="bf97d-423">The <xref:System.Configuration.SectionInformation.InheritInChildApplications*> property is set to `false` to indicate that the settings specified within the [\<location>](/iis/manage/managing-your-configuration-settings/understanding-iis-configuration-delegation#the-concept-of-location) element aren't inherited by apps that reside in a subdirectory of the app.</span></span>

<span data-ttu-id="bf97d-424">Quando un'app viene distribuita in [Servizio app di Azure](https://azure.microsoft.com/services/app-service/), il percorso `stdoutLogFile` è impostato su `\\?\%home%\LogFiles\stdout`.</span><span class="sxs-lookup"><span data-stu-id="bf97d-424">When an app is deployed to [Azure App Service](https://azure.microsoft.com/services/app-service/), the `stdoutLogFile` path is set to `\\?\%home%\LogFiles\stdout`.</span></span> <span data-ttu-id="bf97d-425">Il percorso salva i log stdout nella cartella *LogFiles*, ovvero una posizione creata automaticamente dal servizio.</span><span class="sxs-lookup"><span data-stu-id="bf97d-425">The path saves stdout logs to the *LogFiles* folder, which is a location automatically created by the service.</span></span>

<span data-ttu-id="bf97d-426">Per informazioni sulla configurazione delle applicazioni secondarie IIS, vedere <xref:host-and-deploy/iis/index#sub-applications>.</span><span class="sxs-lookup"><span data-stu-id="bf97d-426">For information on IIS sub-application configuration, see <xref:host-and-deploy/iis/index#sub-applications>.</span></span>

### <a name="attributes-of-the-aspnetcore-element"></a><span data-ttu-id="bf97d-427">Attributi dell'elemento aspNetCore</span><span class="sxs-lookup"><span data-stu-id="bf97d-427">Attributes of the aspNetCore element</span></span>

| <span data-ttu-id="bf97d-428">Attributo</span><span class="sxs-lookup"><span data-stu-id="bf97d-428">Attribute</span></span> | <span data-ttu-id="bf97d-429">Descrizione</span><span class="sxs-lookup"><span data-stu-id="bf97d-429">Description</span></span> | <span data-ttu-id="bf97d-430">Predefinito</span><span class="sxs-lookup"><span data-stu-id="bf97d-430">Default</span></span> |
| --------- | ----------- | :-----: |
| `arguments` | <p><span data-ttu-id="bf97d-431">Attributo stringa facoltativo.</span><span class="sxs-lookup"><span data-stu-id="bf97d-431">Optional string attribute.</span></span></p><p><span data-ttu-id="bf97d-432">Argomenti per il file eseguibile specificato in `processPath` .</span><span class="sxs-lookup"><span data-stu-id="bf97d-432">Arguments to the executable specified in `processPath`.</span></span></p> | |
| `disableStartUpErrorPage` | <p><span data-ttu-id="bf97d-433">Attributo booleano facoltativo.</span><span class="sxs-lookup"><span data-stu-id="bf97d-433">Optional Boolean attribute.</span></span></p><p><span data-ttu-id="bf97d-434">Se true, la pagina **502.5 - Errore del processo** non viene visualizzata e la tabella codici di stato 502 configurata in *web.config* ha la precedenza.</span><span class="sxs-lookup"><span data-stu-id="bf97d-434">If true, the **502.5 - Process Failure** page is suppressed, and the 502 status code page configured in the *web.config* takes precedence.</span></span></p> | `false` |
| `forwardWindowsAuthToken` | <p><span data-ttu-id="bf97d-435">Attributo booleano facoltativo.</span><span class="sxs-lookup"><span data-stu-id="bf97d-435">Optional Boolean attribute.</span></span></p><p><span data-ttu-id="bf97d-436">Se true, il token viene inoltrato al processo figlio in ascolto su %ASPNETCORE_PORT% come un'intestazione 'MS-ASPNETCORE-WINAUTHTOKEN' per ogni richiesta.</span><span class="sxs-lookup"><span data-stu-id="bf97d-436">If true, the token is forwarded to the child process listening on %ASPNETCORE_PORT% as a header 'MS-ASPNETCORE-WINAUTHTOKEN' per request.</span></span> <span data-ttu-id="bf97d-437">È responsabilità del processo chiamare CloseHandle su questo token per ogni richiesta.</span><span class="sxs-lookup"><span data-stu-id="bf97d-437">It's the responsibility of that process to call CloseHandle on this token per request.</span></span></p> | `true` |
| `hostingModel` | <p><span data-ttu-id="bf97d-438">Attributo stringa facoltativo.</span><span class="sxs-lookup"><span data-stu-id="bf97d-438">Optional string attribute.</span></span></p><p><span data-ttu-id="bf97d-439">Specifica il modello di hosting come in-process ( `InProcess` / `inprocess` ) o out-of-process ( `OutOfProcess` / `outofprocess` ).</span><span class="sxs-lookup"><span data-stu-id="bf97d-439">Specifies the hosting model as in-process (`InProcess`/`inprocess`) or out-of-process (`OutOfProcess`/`outofprocess`).</span></span></p> | `OutOfProcess`<br>`outofprocess` |
| `processesPerApplication` | <p><span data-ttu-id="bf97d-440">Attributo Integer facoltativo.</span><span class="sxs-lookup"><span data-stu-id="bf97d-440">Optional integer attribute.</span></span></p><p><span data-ttu-id="bf97d-441">Specifica il numero di istanze del processo specificato nell' `processPath` impostazione che è possibile creare per ogni app.</span><span class="sxs-lookup"><span data-stu-id="bf97d-441">Specifies the number of instances of the process specified in the `processPath` setting that can be spun up per app.</span></span></p><p><span data-ttu-id="bf97d-442">&dagger;Per l'hosting in-process, il valore è limitato a `1`.</span><span class="sxs-lookup"><span data-stu-id="bf97d-442">&dagger;For in-process hosting, the value is limited to `1`.</span></span></p><p><span data-ttu-id="bf97d-443">L'impostazione di `processesPerApplication` è sconsigliata.</span><span class="sxs-lookup"><span data-stu-id="bf97d-443">Setting `processesPerApplication` is discouraged.</span></span> <span data-ttu-id="bf97d-444">Questo attributo sarà rimosso nelle versioni future.</span><span class="sxs-lookup"><span data-stu-id="bf97d-444">This attribute will be removed in a future release.</span></span></p> | <span data-ttu-id="bf97d-445">Valore predefinito: `1`</span><span class="sxs-lookup"><span data-stu-id="bf97d-445">Default: `1`</span></span><br><span data-ttu-id="bf97d-446">Min: `1`</span><span class="sxs-lookup"><span data-stu-id="bf97d-446">Min: `1`</span></span><br><span data-ttu-id="bf97d-447">Max: `100`&dagger;</span><span class="sxs-lookup"><span data-stu-id="bf97d-447">Max: `100`&dagger;</span></span> |
| `processPath` | <p><span data-ttu-id="bf97d-448">Attributo stringa obbligatorio.</span><span class="sxs-lookup"><span data-stu-id="bf97d-448">Required string attribute.</span></span></p><p><span data-ttu-id="bf97d-449">Percorso del file eseguibile che avvia un processo in ascolto delle richieste HTTP.</span><span class="sxs-lookup"><span data-stu-id="bf97d-449">Path to the executable that launches a process listening for HTTP requests.</span></span> <span data-ttu-id="bf97d-450">I percorsi relativi sono supportati.</span><span class="sxs-lookup"><span data-stu-id="bf97d-450">Relative paths are supported.</span></span> <span data-ttu-id="bf97d-451">Se il percorso inizia con `.`, viene considerato relativo alla radice del sito.</span><span class="sxs-lookup"><span data-stu-id="bf97d-451">If the path begins with `.`, the path is considered to be relative to the site root.</span></span></p> | |
| `rapidFailsPerMinute` | <p><span data-ttu-id="bf97d-452">Attributo Integer facoltativo.</span><span class="sxs-lookup"><span data-stu-id="bf97d-452">Optional integer attribute.</span></span></p><p><span data-ttu-id="bf97d-453">Specifica il numero di volte in cui il processo specificato in `processPath` può arrestarsi in modo anomalo al minuto.</span><span class="sxs-lookup"><span data-stu-id="bf97d-453">Specifies the number of times the process specified in `processPath` is allowed to crash per minute.</span></span> <span data-ttu-id="bf97d-454">Se questo limite viene superato, il modulo smette di avviare il processo per la parte restante del minuto.</span><span class="sxs-lookup"><span data-stu-id="bf97d-454">If this limit is exceeded, the module stops launching the process for the remainder of the minute.</span></span></p><p><span data-ttu-id="bf97d-455">Non supportato con l'hosting in-process.</span><span class="sxs-lookup"><span data-stu-id="bf97d-455">Not supported with in-process hosting.</span></span></p> | <span data-ttu-id="bf97d-456">Valore predefinito: `10`</span><span class="sxs-lookup"><span data-stu-id="bf97d-456">Default: `10`</span></span><br><span data-ttu-id="bf97d-457">Min: `0`</span><span class="sxs-lookup"><span data-stu-id="bf97d-457">Min: `0`</span></span><br><span data-ttu-id="bf97d-458">Max: `100`</span><span class="sxs-lookup"><span data-stu-id="bf97d-458">Max: `100`</span></span> |
| `requestTimeout` | <p><span data-ttu-id="bf97d-459">Attributo Timespan facoltativo.</span><span class="sxs-lookup"><span data-stu-id="bf97d-459">Optional timespan attribute.</span></span></p><p><span data-ttu-id="bf97d-460">Specifica la durata per cui il modulo ASP.NET Core attende una risposta dal processo in ascolto su %ASPNETCORE_PORT%.</span><span class="sxs-lookup"><span data-stu-id="bf97d-460">Specifies the duration for which the ASP.NET Core Module waits for a response from the process listening on %ASPNETCORE_PORT%.</span></span></p><p><span data-ttu-id="bf97d-461">Nelle versioni del modulo ASP.NET Core fornito con ASP.NET Core 2.1 o versioni successive, `requestTimeout` viene specificato in ore, minuti e secondi.</span><span class="sxs-lookup"><span data-stu-id="bf97d-461">In versions of the ASP.NET Core Module that shipped with the release of ASP.NET Core 2.1 or later, the `requestTimeout` is specified in hours, minutes, and seconds.</span></span></p><p><span data-ttu-id="bf97d-462">Non è applicabile all'hosting in-process.</span><span class="sxs-lookup"><span data-stu-id="bf97d-462">Doesn't apply to in-process hosting.</span></span> <span data-ttu-id="bf97d-463">Per l'hosting in-process, il modulo resta in attesa che l'app elabori la richiesta.</span><span class="sxs-lookup"><span data-stu-id="bf97d-463">For in-process hosting, the module waits for the app to process the request.</span></span></p><p><span data-ttu-id="bf97d-464">I valori validi per i segmenti della stringa relativi a minuti e secondi sono compresi nell'intervallo tra 0 e 59.</span><span class="sxs-lookup"><span data-stu-id="bf97d-464">Valid values for minutes and seconds segments of the string are in the range 0-59.</span></span> <span data-ttu-id="bf97d-465">Se si usa **60** come valore per i minuti o i secondi, viene generato un errore *500 - Errore interno del server*.</span><span class="sxs-lookup"><span data-stu-id="bf97d-465">Use of **60** in the value for minutes or seconds results in a *500 - Internal Server Error*.</span></span></p> | <span data-ttu-id="bf97d-466">Valore predefinito: `00:02:00`</span><span class="sxs-lookup"><span data-stu-id="bf97d-466">Default: `00:02:00`</span></span><br><span data-ttu-id="bf97d-467">Min: `00:00:00`</span><span class="sxs-lookup"><span data-stu-id="bf97d-467">Min: `00:00:00`</span></span><br><span data-ttu-id="bf97d-468">Max: `360:00:00`</span><span class="sxs-lookup"><span data-stu-id="bf97d-468">Max: `360:00:00`</span></span> |
| `shutdownTimeLimit` | <p><span data-ttu-id="bf97d-469">Attributo Integer facoltativo.</span><span class="sxs-lookup"><span data-stu-id="bf97d-469">Optional integer attribute.</span></span></p><p><span data-ttu-id="bf97d-470">Durata in secondi durante la quale il modulo attende che l'eseguibile venga arrestato normalmente quando `app_offline.htm` viene rilevato il file.</span><span class="sxs-lookup"><span data-stu-id="bf97d-470">Duration in seconds that the module waits for the executable to gracefully shutdown when the `app_offline.htm` file is detected.</span></span></p> | <span data-ttu-id="bf97d-471">Valore predefinito: `10`</span><span class="sxs-lookup"><span data-stu-id="bf97d-471">Default: `10`</span></span><br><span data-ttu-id="bf97d-472">Min: `0`</span><span class="sxs-lookup"><span data-stu-id="bf97d-472">Min: `0`</span></span><br><span data-ttu-id="bf97d-473">Max: `600`</span><span class="sxs-lookup"><span data-stu-id="bf97d-473">Max: `600`</span></span> |
| `startupTimeLimit` | <p><span data-ttu-id="bf97d-474">Attributo Integer facoltativo.</span><span class="sxs-lookup"><span data-stu-id="bf97d-474">Optional integer attribute.</span></span></p><p><span data-ttu-id="bf97d-475">Durata in secondi per cui il modulo attende l'avvio di un processo in ascolto sulla porta da parte del file eseguibile.</span><span class="sxs-lookup"><span data-stu-id="bf97d-475">Duration in seconds that the module waits for the executable to start a process listening on the port.</span></span> <span data-ttu-id="bf97d-476">Se questo limite di tempo viene superato, il modulo termina il processo.</span><span class="sxs-lookup"><span data-stu-id="bf97d-476">If this time limit is exceeded, the module kills the process.</span></span></p><p><span data-ttu-id="bf97d-477">Quando si esegue l'hosting *in-process*: il processo **non** viene riavviato **e non usa** l' `rapidFailsPerMinute` impostazione.</span><span class="sxs-lookup"><span data-stu-id="bf97d-477">When hosting *in-process*: The process is **not** restarted and does **not** use the `rapidFailsPerMinute` setting.</span></span></p><p><span data-ttu-id="bf97d-478">Quando si esegue l'hosting *out-of-process*: il modulo tenta di riavviare il processo quando riceve una nuova richiesta e continua a provare a riavviare il processo per le successive richieste in ingresso, a meno che l'app non riesca ad avviare il `rapidFailsPerMinute` numero di volte nell'ultimo minuto in sequenza.</span><span class="sxs-lookup"><span data-stu-id="bf97d-478">When hosting *out-of-process*: The module attempts to relaunch the process when it receives a new request and continues to attempt to restart the process on subsequent incoming requests unless the app fails to start `rapidFailsPerMinute` number of times in the last rolling minute.</span></span></p><p><span data-ttu-id="bf97d-479">Un valore pari a 0 (zero) **non** è considerato un timeout infinito.</span><span class="sxs-lookup"><span data-stu-id="bf97d-479">A value of 0 (zero) is **not** considered an infinite timeout.</span></span></p> | <span data-ttu-id="bf97d-480">Valore predefinito: `120`</span><span class="sxs-lookup"><span data-stu-id="bf97d-480">Default: `120`</span></span><br><span data-ttu-id="bf97d-481">Min: `0`</span><span class="sxs-lookup"><span data-stu-id="bf97d-481">Min: `0`</span></span><br><span data-ttu-id="bf97d-482">Max: `3600`</span><span class="sxs-lookup"><span data-stu-id="bf97d-482">Max: `3600`</span></span> |
| `stdoutLogEnabled` | <p><span data-ttu-id="bf97d-483">Attributo booleano facoltativo.</span><span class="sxs-lookup"><span data-stu-id="bf97d-483">Optional Boolean attribute.</span></span></p><p><span data-ttu-id="bf97d-484">Se true, **stdout** e **stderr** per il processo specificato in `processPath` vengono reindirizzati al file specificato in **stdoutLogFile**.</span><span class="sxs-lookup"><span data-stu-id="bf97d-484">If true, **stdout** and **stderr** for the process specified in `processPath` are redirected to the file specified in **stdoutLogFile**.</span></span></p> | `false` |
| `stdoutLogFile` | <p><span data-ttu-id="bf97d-485">Attributo stringa facoltativo.</span><span class="sxs-lookup"><span data-stu-id="bf97d-485">Optional string attribute.</span></span></p><p><span data-ttu-id="bf97d-486">Specifica il percorso del file relativo o assoluto per `stdout` cui `stderr` vengono registrati e dal processo specificato in `processPath` .</span><span class="sxs-lookup"><span data-stu-id="bf97d-486">Specifies the relative or absolute file path for which `stdout` and `stderr` from the process specified in `processPath` are logged.</span></span> <span data-ttu-id="bf97d-487">I percorsi relativi sono relativi alla radice del sito.</span><span class="sxs-lookup"><span data-stu-id="bf97d-487">Relative paths are relative to the root of the site.</span></span> <span data-ttu-id="bf97d-488">Qualsiasi percorso che inizia con `.` è relativo al sito radice e tutti gli altri percorsi vengono trattati come percorsi assoluti.</span><span class="sxs-lookup"><span data-stu-id="bf97d-488">Any path starting with `.` are relative to the site root and all other paths are treated as absolute paths.</span></span> <span data-ttu-id="bf97d-489">Le eventuali cartelle specificate nel percorso vengono create dal modulo quando viene creato il file di log.</span><span class="sxs-lookup"><span data-stu-id="bf97d-489">Any folders provided in the path are created by the module when the log file is created.</span></span> <span data-ttu-id="bf97d-490">L'utilizzo di delimitatori di sottolineatura, timestamp, ID processo ed estensione di file ( `.log` ) viene aggiunto all'ultimo segmento del `stdoutLogFile` percorso.</span><span class="sxs-lookup"><span data-stu-id="bf97d-490">Using underscore delimiters, a timestamp, process ID, and file extension (`.log`) are added to the last segment of the `stdoutLogFile` path.</span></span> <span data-ttu-id="bf97d-491">Se `.\logs\stdout` viene fornito come valore, un esempio di log stdout viene salvato come `stdout_20180205194132_1934.log` nella `logs` cartella quando viene salvato il 2/5/2018 alle 19:41:32 con l'ID di processo 1934.</span><span class="sxs-lookup"><span data-stu-id="bf97d-491">If `.\logs\stdout` is supplied as a value, an example stdout log is saved as `stdout_20180205194132_1934.log` in the `logs` folder when saved on 2/5/2018 at 19:41:32 with a process ID of 1934.</span></span></p> | `aspnetcore-stdout` |

### <a name="setting-environment-variables"></a><span data-ttu-id="bf97d-492">Impostazioni delle variabili di ambiente</span><span class="sxs-lookup"><span data-stu-id="bf97d-492">Setting environment variables</span></span>

<span data-ttu-id="bf97d-493">È possibile specificare le variabili di ambiente per il processo nell'attributo `processPath`.</span><span class="sxs-lookup"><span data-stu-id="bf97d-493">Environment variables can be specified for the process in the `processPath` attribute.</span></span> <span data-ttu-id="bf97d-494">Specificare una variabile di ambiente con l'elemento figlio `<environmentVariable>` di un elemento della raccolta `<environmentVariables>`.</span><span class="sxs-lookup"><span data-stu-id="bf97d-494">Specify an environment variable with the `<environmentVariable>` child element of an `<environmentVariables>` collection element.</span></span> <span data-ttu-id="bf97d-495">Le variabili di ambiente impostate in questa sezione hanno la precedenza sulle variabili di ambiente di sistema.</span><span class="sxs-lookup"><span data-stu-id="bf97d-495">Environment variables set in this section take precedence over system environment variables.</span></span>

<span data-ttu-id="bf97d-496">Nell'esempio seguente vengono impostate due variabili di ambiente.</span><span class="sxs-lookup"><span data-stu-id="bf97d-496">The following example sets two environment variables.</span></span> <span data-ttu-id="bf97d-497">`ASPNETCORE_ENVIRONMENT` configura l'ambiente dell'app su `Development`.</span><span class="sxs-lookup"><span data-stu-id="bf97d-497">`ASPNETCORE_ENVIRONMENT` configures the app's environment to `Development`.</span></span> <span data-ttu-id="bf97d-498">Uno sviluppatore può impostare temporaneamente questo valore nel `web.config` file per forzare il caricamento della [pagina delle eccezioni](xref:fundamentals/error-handling) per gli sviluppatori durante il debug di un'eccezione dell'app.</span><span class="sxs-lookup"><span data-stu-id="bf97d-498">A developer may temporarily set this value in the `web.config` file in order to force the [Developer Exception Page](xref:fundamentals/error-handling) to load when debugging an app exception.</span></span> <span data-ttu-id="bf97d-499">`CONFIG_DIR` è un esempio di variabile di ambiente definita dall'utente, in cui lo sviluppatore ha scritto il codice che legge il valore all'avvio in modo da formare un percorso per il caricamento del file di configurazione dell'app.</span><span class="sxs-lookup"><span data-stu-id="bf97d-499">`CONFIG_DIR` is an example of a user-defined environment variable, where the developer has written code that reads the value on startup to form a path for loading the app's configuration file.</span></span>

```xml
<aspNetCore processPath="dotnet"
      arguments=".\MyApp.dll"
      stdoutLogEnabled="false"
      stdoutLogFile=".\logs\stdout"
      hostingModel="inprocess">
  <environmentVariables>
    <environmentVariable name="ASPNETCORE_ENVIRONMENT" value="Development" />
    <environmentVariable name="CONFIG_DIR" value="f:\application_config" />
  </environmentVariables>
</aspNetCore>
```

> [!NOTE]
> <span data-ttu-id="bf97d-500">Un'alternativa all'impostazione dell'ambiente direttamente in `web.config` consiste nell'includere la `<EnvironmentName>` proprietà nel [profilo di pubblicazione (con estensione pubxml)](xref:host-and-deploy/visual-studio-publish-profiles) o nel file di progetto.</span><span class="sxs-lookup"><span data-stu-id="bf97d-500">An alternative to setting the environment directly in `web.config` is to include the `<EnvironmentName>` property in the [publish profile (.pubxml)](xref:host-and-deploy/visual-studio-publish-profiles) or project file.</span></span> <span data-ttu-id="bf97d-501">Questo approccio imposta l'ambiente in `web.config` quando viene pubblicato il progetto:</span><span class="sxs-lookup"><span data-stu-id="bf97d-501">This approach sets the environment in `web.config` when the project is published:</span></span>
>
> ```xml
> <PropertyGroup>
>   <EnvironmentName>Development</EnvironmentName>
> </PropertyGroup>
> ```

> [!WARNING]
> <span data-ttu-id="bf97d-502">Impostare la variabile di ambiente `ASPNETCORE_ENVIRONMENT` su `Development` solo in server di gestione temporanea e test che non sono accessibili da reti non attendibili, ad esempio Internet.</span><span class="sxs-lookup"><span data-stu-id="bf97d-502">Only set the `ASPNETCORE_ENVIRONMENT` environment variable to `Development` on staging and testing servers that aren't accessible to untrusted networks, such as the Internet.</span></span>

## <a name="app_offlinehtm"></a><span data-ttu-id="bf97d-503">app_offline.htm</span><span class="sxs-lookup"><span data-stu-id="bf97d-503">app_offline.htm</span></span>

<span data-ttu-id="bf97d-504">Se viene rilevato un file con il nome `app_offline.htm` nella directory radice di un'app, il modulo ASP.NET Core tenta di arrestare normalmente l'app e di arrestare l'elaborazione delle richieste in ingresso.</span><span class="sxs-lookup"><span data-stu-id="bf97d-504">If a file with the name `app_offline.htm` is detected in the root directory of an app, the ASP.NET Core Module attempts to gracefully shutdown the app and stop processing incoming requests.</span></span> <span data-ttu-id="bf97d-505">Se l'app è ancora in esecuzione dopo il numero di secondi definito in `shutdownTimeLimit`, il modulo ASP.NET Core termina il processo in esecuzione.</span><span class="sxs-lookup"><span data-stu-id="bf97d-505">If the app is still running after the number of seconds defined in `shutdownTimeLimit`, the ASP.NET Core Module kills the running process.</span></span>

<span data-ttu-id="bf97d-506">Mentre il `app_offline.htm` file è presente, il modulo ASP.NET Core risponde alle richieste inviando di nuovo il contenuto del `app_offline.htm` file.</span><span class="sxs-lookup"><span data-stu-id="bf97d-506">While the `app_offline.htm` file is present, the ASP.NET Core Module responds to requests by sending back the contents of the `app_offline.htm` file.</span></span> <span data-ttu-id="bf97d-507">Quando il `app_offline.htm` file viene rimosso, la richiesta successiva avvia l'app.</span><span class="sxs-lookup"><span data-stu-id="bf97d-507">When the `app_offline.htm` file is removed, the next request starts the app.</span></span>

<span data-ttu-id="bf97d-508">Quando si usa il modello di hosting out-of-process, l'app potrebbe non arrestarsi immediatamente se una connessione è aperta.</span><span class="sxs-lookup"><span data-stu-id="bf97d-508">When using the out-of-process hosting model, the app might not shut down immediately if there's an open connection.</span></span> <span data-ttu-id="bf97d-509">Ad esempio, una connessione WebSocket può ritardare l'arresto dell'app.</span><span class="sxs-lookup"><span data-stu-id="bf97d-509">For example, a websocket connection may delay app shut down.</span></span>

## <a name="start-up-error-page"></a><span data-ttu-id="bf97d-510">Pagina di errore di avvio</span><span class="sxs-lookup"><span data-stu-id="bf97d-510">Start-up error page</span></span>

<span data-ttu-id="bf97d-511">Sia l'hosting In-Process che quello out-of-process producono pagine di errore personalizzate quando non riescono ad avviare l'app.</span><span class="sxs-lookup"><span data-stu-id="bf97d-511">Both in-process and out-of-process hosting produce custom error pages when they fail to start the app.</span></span>

<span data-ttu-id="bf97d-512">Se il modulo ASP.NET Core non riesce a trovare il gestore richieste In-Process o out-of-process, viene visualizzata una tabella codici di stato *500.0 - Errore di caricamento del gestore In-Process/out-of-process*.</span><span class="sxs-lookup"><span data-stu-id="bf97d-512">If the ASP.NET Core Module fails to find either the in-process or out-of-process request handler, a *500.0 - In-Process/Out-Of-Process Handler Load Failure* status code page appears.</span></span>

<span data-ttu-id="bf97d-513">Per l'hosting In-Process, se il modulo ASP.NET Core non riesce ad avviare l'app, viene visualizzata una tabella codici di stato *500.30 - Errore di avvio*.</span><span class="sxs-lookup"><span data-stu-id="bf97d-513">For in-process hosting if the ASP.NET Core Module fails to start the app, a *500.30 - Start Failure* status code page appears.</span></span>

<span data-ttu-id="bf97d-514">Per l'hosting out-of-process, se il modulo ASP.NET Core non riesce ad avviare il processo di back-end o il processo di back-end viene avviato ma non riesce a restare in ascolto sulla porta configurata, verrà visualizzata la tabella codici di stato *502.5 - Errore del processo*.</span><span class="sxs-lookup"><span data-stu-id="bf97d-514">For out-of-process hosting if the ASP.NET Core Module fails to launch the backend process or the backend process starts but fails to listen on the configured port, a *502.5 - Process Failure* status code page appears.</span></span>

<span data-ttu-id="bf97d-515">Per non visualizzare questa pagina e tornare alla tabella codici di stato 5xx predefinita di IIS, usare l'attributo `disableStartUpErrorPage`.</span><span class="sxs-lookup"><span data-stu-id="bf97d-515">To suppress this page and revert to the default IIS 5xx status code page, use the `disableStartUpErrorPage` attribute.</span></span> <span data-ttu-id="bf97d-516">Per altre informazioni sulla configurazione di messaggi di errore personalizzati, vedere [Errori HTTP \<httpErrors>](/iis/configuration/system.webServer/httpErrors/).</span><span class="sxs-lookup"><span data-stu-id="bf97d-516">For more information on configuring custom error messages, see [HTTP Errors \<httpErrors>](/iis/configuration/system.webServer/httpErrors/).</span></span>

## <a name="log-creation-and-redirection"></a><span data-ttu-id="bf97d-517">Creazione e reindirizzamento dei log</span><span class="sxs-lookup"><span data-stu-id="bf97d-517">Log creation and redirection</span></span>

<span data-ttu-id="bf97d-518">Il modulo ASP.NET Core reindirizza su disco l'output della console stdout e stderr se sono impostati gli attributi `stdoutLogEnabled` e `stdoutLogFile` dell'elemento `aspNetCore`.</span><span class="sxs-lookup"><span data-stu-id="bf97d-518">The ASP.NET Core Module redirects stdout and stderr console output to disk if the `stdoutLogEnabled` and `stdoutLogFile` attributes of the `aspNetCore` element are set.</span></span> <span data-ttu-id="bf97d-519">Le eventuali le cartelle nel percorso `stdoutLogFile` vengono create dal modulo quando viene creato il file di log.</span><span class="sxs-lookup"><span data-stu-id="bf97d-519">Any folders in the `stdoutLogFile` path are created by the module when the log file is created.</span></span> <span data-ttu-id="bf97d-520">Il pool di applicazioni deve avere accesso in scrittura al percorso in cui vengono scritti i log (usare `IIS AppPool\{APP POOL NAME}` per fornire l'autorizzazione di scrittura, dove il segnaposto `{APP POOL NAME}` è il nome del pool di applicazioni).</span><span class="sxs-lookup"><span data-stu-id="bf97d-520">The app pool must have write access to the location where the logs are written (use `IIS AppPool\{APP POOL NAME}` to provide write permission, where the placeholder `{APP POOL NAME}` is the app pool name).</span></span>

<span data-ttu-id="bf97d-521">I log non vengono ruotati, a meno che non si verifichi il riciclo/riavvio del processo.</span><span class="sxs-lookup"><span data-stu-id="bf97d-521">Logs aren't rotated, unless process recycling/restart occurs.</span></span> <span data-ttu-id="bf97d-522">È responsabilità del provider di servizi di hosting limitare lo spazio su disco usato dai log.</span><span class="sxs-lookup"><span data-stu-id="bf97d-522">It's the responsibility of the hoster to limit the disk space the logs consume.</span></span>

<span data-ttu-id="bf97d-523">L'uso del log stdout è consigliato solo per la risoluzione dei problemi di avvio dell'app durante l'hosting in IIS o quando si usa il [supporto in fase di sviluppo per IIS con Visual Studio](xref:host-and-deploy/iis/development-time-iis-support), non durante il debug in locale e l'esecuzione dell'app con IIS Express.</span><span class="sxs-lookup"><span data-stu-id="bf97d-523">Using the stdout log is only recommended for troubleshooting app startup issues when hosting on IIS or when using [development-time support for IIS with Visual Studio](xref:host-and-deploy/iis/development-time-iis-support), not while debugging locally and running the app with IIS Express.</span></span>

<span data-ttu-id="bf97d-524">Non usare il log stdout per scopi di registrazione generale delle app.</span><span class="sxs-lookup"><span data-stu-id="bf97d-524">Don't use the stdout log for general app logging purposes.</span></span> <span data-ttu-id="bf97d-525">Per la registrazione di routine in un'app ASP.NET Core, usare una libreria di registrazione che limita le dimensioni dei file di log e ne esegue la rotazione.</span><span class="sxs-lookup"><span data-stu-id="bf97d-525">For routine logging in an ASP.NET Core app, use a logging library that limits log file size and rotates logs.</span></span> <span data-ttu-id="bf97d-526">Per altre informazioni, vedere [Provider di registrazione di terze parti](xref:fundamentals/logging/index#third-party-logging-providers).</span><span class="sxs-lookup"><span data-stu-id="bf97d-526">For more information, see [third-party logging providers](xref:fundamentals/logging/index#third-party-logging-providers).</span></span>

<span data-ttu-id="bf97d-527">Un timestamp e l'estensione del file vengono aggiunti automaticamente al momento della creazione del file di log.</span><span class="sxs-lookup"><span data-stu-id="bf97d-527">A timestamp and file extension are added automatically when the log file is created.</span></span> <span data-ttu-id="bf97d-528">Il nome del file di log è costituito dall'aggiunta del timestamp, dell'ID del processo e dell'estensione di file ( `.log` ) all'ultimo segmento del `stdoutLogFile` percorso, in genere `stdout` delimitato da caratteri di sottolineatura.</span><span class="sxs-lookup"><span data-stu-id="bf97d-528">The log file name is composed by appending the timestamp, process ID, and file extension (`.log`) to the last segment of the `stdoutLogFile` path (typically `stdout`) delimited by underscores.</span></span> <span data-ttu-id="bf97d-529">Se il `stdoutLogFile` percorso termina con `stdout` , il nome file di un log per un'app con PID 1934 creato in 2/5/2018 a 19:42:32 è `stdout_20180205194132_1934.log` .</span><span class="sxs-lookup"><span data-stu-id="bf97d-529">If the `stdoutLogFile` path ends with `stdout`, a log for an app with a PID of 1934 created on 2/5/2018 at 19:42:32 has the file name `stdout_20180205194132_1934.log`.</span></span>

<span data-ttu-id="bf97d-530">Se `stdoutLogEnabled` è false, gli errori che si verificano all'avvio dell'app vengono acquisiti ed emessi nel log eventi fino a 30 KB.</span><span class="sxs-lookup"><span data-stu-id="bf97d-530">If `stdoutLogEnabled` is false, errors that occur on app startup are captured and emitted to the event log up to 30 KB.</span></span> <span data-ttu-id="bf97d-531">Dopo l'avvio, tutti i log aggiuntivi vengono rimossi.</span><span class="sxs-lookup"><span data-stu-id="bf97d-531">After startup, all additional logs are discarded.</span></span>

<span data-ttu-id="bf97d-532">Nell'elemento di esempio seguente viene `aspNetCore` configurata la registrazione di stdout nel percorso relativo `.\log\` .</span><span class="sxs-lookup"><span data-stu-id="bf97d-532">The following sample `aspNetCore` element configures stdout logging at the relative path `.\log\`.</span></span> <span data-ttu-id="bf97d-533">Verificare che l'identità utente del pool di applicazioni disponga delle autorizzazioni necessarie per scrivere nel percorso specificato.</span><span class="sxs-lookup"><span data-stu-id="bf97d-533">Confirm that the app pool user identity has permission to write to the path provided.</span></span>

```xml
<aspNetCore processPath="dotnet"
    arguments=".\MyApp.dll"
    stdoutLogEnabled="true"
    stdoutLogFile=".\logs\stdout"
    hostingModel="inprocess">
</aspNetCore>
```

<span data-ttu-id="bf97d-534">Quando si pubblica un'app per la distribuzione di app Azure Service, il valore viene impostato da Web SDK `stdoutLogFile` `\\?\%home%\LogFiles\stdout` .</span><span class="sxs-lookup"><span data-stu-id="bf97d-534">When publishing an app for Azure App Service deployment, the Web SDK sets the `stdoutLogFile` value to `\\?\%home%\LogFiles\stdout`.</span></span> <span data-ttu-id="bf97d-535">La `%home` variabile di ambiente è predefinita per le app ospitate dal servizio app Azure.</span><span class="sxs-lookup"><span data-stu-id="bf97d-535">The `%home` environment variable is predefined for apps hosted by Azure App Service.</span></span>

<span data-ttu-id="bf97d-536">Per ulteriori informazioni sui formati di percorso, vedere [formati di percorso dei file nei sistemi Windows](/dotnet/standard/io/file-path-formats).</span><span class="sxs-lookup"><span data-stu-id="bf97d-536">For more information on path formats, see [File path formats on Windows systems](/dotnet/standard/io/file-path-formats).</span></span>

## <a name="enhanced-diagnostic-logs"></a><span data-ttu-id="bf97d-537">Log di diagnostica avanzati</span><span class="sxs-lookup"><span data-stu-id="bf97d-537">Enhanced diagnostic logs</span></span>

<span data-ttu-id="bf97d-538">Il modulo ASP.NET Core può essere configurato per restituire log di diagnostica avanzata.</span><span class="sxs-lookup"><span data-stu-id="bf97d-538">The ASP.NET Core Module is configurable to provide enhanced diagnostics logs.</span></span> <span data-ttu-id="bf97d-539">Aggiungere l' `<handlerSettings>` elemento all' `<aspNetCore>` elemento in `web.config` .</span><span class="sxs-lookup"><span data-stu-id="bf97d-539">Add the `<handlerSettings>` element to the `<aspNetCore>` element in `web.config`.</span></span> <span data-ttu-id="bf97d-540">L'impostazione di `debugLevel` su `TRACE` restituisce una maggior fedeltà dei dati diagnostici:</span><span class="sxs-lookup"><span data-stu-id="bf97d-540">Setting the `debugLevel` to `TRACE` exposes a higher fidelity of diagnostic information:</span></span>

```xml
<aspNetCore processPath="dotnet"
    arguments=".\MyApp.dll"
    stdoutLogEnabled="false"
    stdoutLogFile="\\?\%home%\LogFiles\stdout"
    hostingModel="inprocess">
  <handlerSettings>
    <handlerSetting name="debugFile" value=".\logs\aspnetcore-debug.log" />
    <handlerSetting name="debugLevel" value="FILE,TRACE" />
  </handlerSettings>
</aspNetCore>
```

<span data-ttu-id="bf97d-541">Le cartelle nel percorso specificato per il `<handlerSetting>` valore ( `logs` nell'esempio precedente) non vengono create automaticamente dal modulo e dovrebbero essere già presenti nella distribuzione.</span><span class="sxs-lookup"><span data-stu-id="bf97d-541">Folders in the path provided to the `<handlerSetting>` value (`logs` in the preceding example) aren't created by the module automatically and should pre-exist in the deployment.</span></span> <span data-ttu-id="bf97d-542">Il pool di applicazioni deve avere accesso in scrittura al percorso in cui vengono scritti i log (usare `IIS AppPool\{APP POOL NAME}` per fornire l'autorizzazione di scrittura, dove il segnaposto `{APP POOL NAME}` è il nome del pool di applicazioni).</span><span class="sxs-lookup"><span data-stu-id="bf97d-542">The app pool must have write access to the location where the logs are written (use `IIS AppPool\{APP POOL NAME}` to provide write permission, where the placeholder `{APP POOL NAME}` is the app pool name).</span></span>

<span data-ttu-id="bf97d-543">I valori di livello debug (`debugLevel`) possono includere sia il livello che il percorso.</span><span class="sxs-lookup"><span data-stu-id="bf97d-543">Debug level (`debugLevel`) values can include both the level and the location.</span></span>

<span data-ttu-id="bf97d-544">Livelli (in ordine dal meno dettagliato al più dettagliato):</span><span class="sxs-lookup"><span data-stu-id="bf97d-544">Levels (in order from least to most verbose):</span></span>

* <span data-ttu-id="bf97d-545">ERRORE</span><span class="sxs-lookup"><span data-stu-id="bf97d-545">ERROR</span></span>
* <span data-ttu-id="bf97d-546">WARNING</span><span class="sxs-lookup"><span data-stu-id="bf97d-546">WARNING</span></span>
* <span data-ttu-id="bf97d-547">INFO</span><span class="sxs-lookup"><span data-stu-id="bf97d-547">INFO</span></span>
* <span data-ttu-id="bf97d-548">TRACE</span><span class="sxs-lookup"><span data-stu-id="bf97d-548">TRACE</span></span>

<span data-ttu-id="bf97d-549">Posizioni (sono consentite più posizioni):</span><span class="sxs-lookup"><span data-stu-id="bf97d-549">Locations (multiple locations are permitted):</span></span>

* <span data-ttu-id="bf97d-550">CONSOLE</span><span class="sxs-lookup"><span data-stu-id="bf97d-550">CONSOLE</span></span>
* <span data-ttu-id="bf97d-551">EVENTLOG</span><span class="sxs-lookup"><span data-stu-id="bf97d-551">EVENTLOG</span></span>
* <span data-ttu-id="bf97d-552">FILE</span><span class="sxs-lookup"><span data-stu-id="bf97d-552">FILE</span></span>

<span data-ttu-id="bf97d-553">Le impostazioni del gestore possono essere specificate anche tramite le variabili di ambiente:</span><span class="sxs-lookup"><span data-stu-id="bf97d-553">The handler settings can also be provided via environment variables:</span></span>

* <span data-ttu-id="bf97d-554">`ASPNETCORE_MODULE_DEBUG_FILE`: Percorso del file di log di debug.</span><span class="sxs-lookup"><span data-stu-id="bf97d-554">`ASPNETCORE_MODULE_DEBUG_FILE`: Path to the debug log file.</span></span> <span data-ttu-id="bf97d-555">(Impostazione predefinita: `aspnetcore-debug.log` )</span><span class="sxs-lookup"><span data-stu-id="bf97d-555">(Default: `aspnetcore-debug.log`)</span></span>
* <span data-ttu-id="bf97d-556">`ASPNETCORE_MODULE_DEBUG`: Impostazione del livello di debug.</span><span class="sxs-lookup"><span data-stu-id="bf97d-556">`ASPNETCORE_MODULE_DEBUG`: Debug level setting.</span></span>

> [!WARNING]
> <span data-ttu-id="bf97d-557">**Non** lasciare la registrazione del debug abilitata nella distribuzione per un tempo superiore a quello necessario alla risoluzione di problema.</span><span class="sxs-lookup"><span data-stu-id="bf97d-557">Do **not** leave debug logging enabled in the deployment for longer than required to troubleshoot an issue.</span></span> <span data-ttu-id="bf97d-558">Le dimensioni del log non sono limitate.</span><span class="sxs-lookup"><span data-stu-id="bf97d-558">The size of the log isn't limited.</span></span> <span data-ttu-id="bf97d-559">Se si lascia abilitato il log di debug, lo spazio disponibile su disco può esaurirsi e il server o il servizio app può registrare un arresto anomalo.</span><span class="sxs-lookup"><span data-stu-id="bf97d-559">Leaving the debug log enabled can exhaust the available disk space and crash the server or app service.</span></span>

<span data-ttu-id="bf97d-560">Per un esempio dell'elemento nel file, vedere la pagina relativa alla [configurazione con web.config](#configuration-with-webconfig) `aspNetCore` `web.config` .</span><span class="sxs-lookup"><span data-stu-id="bf97d-560">See [Configuration with web.config](#configuration-with-webconfig) for an example of the `aspNetCore` element in the `web.config` file.</span></span>

## <a name="proxy-configuration-uses-http-protocol-and-a-pairing-token"></a><span data-ttu-id="bf97d-561">La configurazione del proxy usa il protocollo HTTP e un token di associazione</span><span class="sxs-lookup"><span data-stu-id="bf97d-561">Proxy configuration uses HTTP protocol and a pairing token</span></span>

<span data-ttu-id="bf97d-562">*Si applica solo all'hosting out-of-process.*</span><span class="sxs-lookup"><span data-stu-id="bf97d-562">*Only applies to out-of-process hosting.*</span></span>

<span data-ttu-id="bf97d-563">Il proxy creato tra il modulo ASP.NET Core e Kestrel usa il protocollo HTTP.</span><span class="sxs-lookup"><span data-stu-id="bf97d-563">The proxy created between the ASP.NET Core Module and Kestrel uses the HTTP protocol.</span></span> <span data-ttu-id="bf97d-564">Non vi è alcun rischio di intercettazione del traffico tra il modulo e Kestrel da una posizione esterna al server.</span><span class="sxs-lookup"><span data-stu-id="bf97d-564">There's no risk of eavesdropping the traffic between the module and Kestrel from a location off of the server.</span></span>

<span data-ttu-id="bf97d-565">Viene usato un token di associazione per garantire che le richieste ricevute da Kestrel siano state trasmesse tramite proxy da IIS e non provengano da un'altra origine.</span><span class="sxs-lookup"><span data-stu-id="bf97d-565">A pairing token is used to guarantee that the requests received by Kestrel were proxied by IIS and didn't come from some other source.</span></span> <span data-ttu-id="bf97d-566">Il token di associazione viene creato e impostato in una variabile di ambiente (`ASPNETCORE_TOKEN`) dal modulo.</span><span class="sxs-lookup"><span data-stu-id="bf97d-566">The pairing token is created and set into an environment variable (`ASPNETCORE_TOKEN`) by the module.</span></span> <span data-ttu-id="bf97d-567">Il token di associazione viene impostato anche in un'intestazione (`MS-ASPNETCORE-TOKEN`) in tutte le richieste trasmesse tramite proxy.</span><span class="sxs-lookup"><span data-stu-id="bf97d-567">The pairing token is also set into a header (`MS-ASPNETCORE-TOKEN`) on every proxied request.</span></span> <span data-ttu-id="bf97d-568">Il middleware di IIS controlla ogni richiesta che riceve in modo da confermare che il valore dell'intestazione del token di associazione corrisponda al valore della variabile di ambiente.</span><span class="sxs-lookup"><span data-stu-id="bf97d-568">IIS Middleware checks each request it receives to confirm that the pairing token header value matches the environment variable value.</span></span> <span data-ttu-id="bf97d-569">Se i valori del token non corrispondono, la richiesta viene registrata e rifiutata.</span><span class="sxs-lookup"><span data-stu-id="bf97d-569">If the token values are mismatched, the request is logged and rejected.</span></span> <span data-ttu-id="bf97d-570">La variabile di ambiente del token di associazione e il traffico tra il modulo e Kestrel non sono accessibili da una posizione esterna al server.</span><span class="sxs-lookup"><span data-stu-id="bf97d-570">The pairing token environment variable and the traffic between the module and Kestrel aren't accessible from a location off of the server.</span></span> <span data-ttu-id="bf97d-571">Senza conoscere il valore del token di associazione, un utente malintenzionato non può inviare richieste che ignorano il controllo nel middleware di IIS.</span><span class="sxs-lookup"><span data-stu-id="bf97d-571">Without knowing the pairing token value, an attacker can't submit requests that bypass the check in the IIS Middleware.</span></span>

## <a name="aspnet-core-module-with-an-iis-shared-configuration"></a><span data-ttu-id="bf97d-572">Modulo ASP.NET Core con configurazione condivisa di IIS</span><span class="sxs-lookup"><span data-stu-id="bf97d-572">ASP.NET Core Module with an IIS Shared Configuration</span></span>

<span data-ttu-id="bf97d-573">Il programma di installazione del modulo ASP.NET Core viene eseguito con i privilegi dell' `TrustedInstaller` account.</span><span class="sxs-lookup"><span data-stu-id="bf97d-573">The ASP.NET Core Module installer runs with the privileges of the `TrustedInstaller` account.</span></span> <span data-ttu-id="bf97d-574">Poiché l'account di sistema locale non dispone dell'autorizzazione di modifica per il percorso di condivisione utilizzato dalla configurazione condivisa di IIS, il programma di installazione genera un errore di accesso negato quando si tenta di configurare le impostazioni del modulo nel `applicationHost.config` file nella condivisione.</span><span class="sxs-lookup"><span data-stu-id="bf97d-574">Because the local system account doesn't have modify permission for the share path used by the IIS Shared Configuration, the installer throws an access denied error when attempting to configure the module settings in the `applicationHost.config` file on the share.</span></span>

<span data-ttu-id="bf97d-575">Quando si usa una configurazione condivisa di IIS nello stesso computer dell'installazione di IIS, eseguire il programma di installazione del bundle di hosting ASP.NET Core con il parametro `OPT_NO_SHARED_CONFIG_CHECK` impostato su `1`:</span><span class="sxs-lookup"><span data-stu-id="bf97d-575">When using an IIS Shared Configuration on the same machine as the IIS installation, run the ASP.NET Core Hosting Bundle installer with the `OPT_NO_SHARED_CONFIG_CHECK` parameter set to `1`:</span></span>

```console
dotnet-hosting-{VERSION}.exe OPT_NO_SHARED_CONFIG_CHECK=1
```

<span data-ttu-id="bf97d-576">Quando il percorso della configurazione condivisa non è nello stesso computer dell'installazione di IIS, seguire questa procedura:</span><span class="sxs-lookup"><span data-stu-id="bf97d-576">When the path to the shared configuration isn't on the same machine as the IIS installation, follow these steps:</span></span>

1. <span data-ttu-id="bf97d-577">Disabilitare la configurazione condivisa di IIS.</span><span class="sxs-lookup"><span data-stu-id="bf97d-577">Disable the IIS Shared Configuration.</span></span>
1. <span data-ttu-id="bf97d-578">Eseguire il programma di installazione.</span><span class="sxs-lookup"><span data-stu-id="bf97d-578">Run the installer.</span></span>
1. <span data-ttu-id="bf97d-579">Esportare il file aggiornato nella `applicationHost.config` condivisione.</span><span class="sxs-lookup"><span data-stu-id="bf97d-579">Export the updated `applicationHost.config` file to the share.</span></span>
1. <span data-ttu-id="bf97d-580">Abilitare di nuovo la configurazione condivisa di IIS.</span><span class="sxs-lookup"><span data-stu-id="bf97d-580">Re-enable the IIS Shared Configuration.</span></span>

## <a name="module-version-and-hosting-bundle-installer-logs"></a><span data-ttu-id="bf97d-581">Versione del modulo e log del programma di installazione del bundle di hosting</span><span class="sxs-lookup"><span data-stu-id="bf97d-581">Module version and Hosting Bundle installer logs</span></span>

<span data-ttu-id="bf97d-582">Per determinare la versione del modulo ASP.NET Core installato:</span><span class="sxs-lookup"><span data-stu-id="bf97d-582">To determine the version of the installed ASP.NET Core Module:</span></span>

1. <span data-ttu-id="bf97d-583">Nel sistema di hosting passare a `%windir%\System32\inetsrv` .</span><span class="sxs-lookup"><span data-stu-id="bf97d-583">On the hosting system, navigate to `%windir%\System32\inetsrv`.</span></span>
1. <span data-ttu-id="bf97d-584">Individuare il `aspnetcore.dll` file.</span><span class="sxs-lookup"><span data-stu-id="bf97d-584">Locate the `aspnetcore.dll` file.</span></span>
1. <span data-ttu-id="bf97d-585">Fare clic con il pulsante destro del mouse sul file e scegliere **Proprietà** dal menu di scelta rapida.</span><span class="sxs-lookup"><span data-stu-id="bf97d-585">Right-click the file and select **Properties** from the contextual menu.</span></span>
1. <span data-ttu-id="bf97d-586">Selezionare la scheda **Dettagli** . La versione del **file** e la **versione del prodotto** rappresentano la versione installata del modulo.</span><span class="sxs-lookup"><span data-stu-id="bf97d-586">Select the **Details** tab. The **File version** and **Product version** represent the installed version of the module.</span></span>

<span data-ttu-id="bf97d-587">I log del programma di installazione del bundle di hosting per il modulo sono disponibili in `C:\\Users\\%UserName%\\AppData\\Local\\Temp` .</span><span class="sxs-lookup"><span data-stu-id="bf97d-587">The Hosting Bundle installer logs for the module are found at `C:\\Users\\%UserName%\\AppData\\Local\\Temp`.</span></span> <span data-ttu-id="bf97d-588">Il nome del file è `dd_DotNetCoreWinSvrHosting__\{TIMESTAMP}_000_AspNetCoreModule_x64.log` , dove il segnaposto `{TIMESTAMP}` è il timestamp.</span><span class="sxs-lookup"><span data-stu-id="bf97d-588">The file is named `dd_DotNetCoreWinSvrHosting__\{TIMESTAMP}_000_AspNetCoreModule_x64.log`, where the placeholder `{TIMESTAMP}` is the timestamp.</span></span>

## <a name="module-schema-and-configuration-file-locations"></a><span data-ttu-id="bf97d-589">Percorsi dei file di modulo, schema e configurazione</span><span class="sxs-lookup"><span data-stu-id="bf97d-589">Module, schema, and configuration file locations</span></span>

### <a name="module"></a><span data-ttu-id="bf97d-590">Modulo</span><span class="sxs-lookup"><span data-stu-id="bf97d-590">Module</span></span>

<span data-ttu-id="bf97d-591">**IIS (x86/amd64)**:</span><span class="sxs-lookup"><span data-stu-id="bf97d-591">**IIS (x86/amd64)**:</span></span>

* `%windir%\System32\inetsrv\aspnetcore.dll`

* `%windir%\SysWOW64\inetsrv\aspnetcore.dll`

* `%ProgramFiles%\IIS\Asp.Net Core Module\V2\aspnetcorev2.dll`

* `%ProgramFiles(x86)%\IIS\Asp.Net Core Module\V2\aspnetcorev2.dll`

<span data-ttu-id="bf97d-592">**IIS Express (x86/amd64)**:</span><span class="sxs-lookup"><span data-stu-id="bf97d-592">**IIS Express (x86/amd64)**:</span></span>

* `%ProgramFiles%\IIS Express\aspnetcore.dll`

* `%ProgramFiles(x86)%\IIS Express\aspnetcore.dll`

* `%ProgramFiles%\IIS Express\Asp.Net Core Module\V2\aspnetcorev2.dll`

* `%ProgramFiles(x86)%\IIS Express\Asp.Net Core Module\V2\aspnetcorev2.dll`

### <a name="schema"></a><span data-ttu-id="bf97d-593">SCHEMA</span><span class="sxs-lookup"><span data-stu-id="bf97d-593">Schema</span></span>

<span data-ttu-id="bf97d-594">**IIS**</span><span class="sxs-lookup"><span data-stu-id="bf97d-594">**IIS**</span></span>

* `%windir%\System32\inetsrv\config\schema\aspnetcore_schema.xml`

* `%windir%\System32\inetsrv\config\schema\aspnetcore_schema_v2.xml`

<span data-ttu-id="bf97d-595">**IIS Express**</span><span class="sxs-lookup"><span data-stu-id="bf97d-595">**IIS Express**</span></span>

* `%ProgramFiles%\IIS Express\config\schema\aspnetcore_schema.xml`

* `%ProgramFiles%\IIS Express\config\schema\aspnetcore_schema_v2.xml`

### <a name="configuration"></a><span data-ttu-id="bf97d-596">Configurazione</span><span class="sxs-lookup"><span data-stu-id="bf97d-596">Configuration</span></span>

<span data-ttu-id="bf97d-597">**IIS**</span><span class="sxs-lookup"><span data-stu-id="bf97d-597">**IIS**</span></span>

* `%windir%\System32\inetsrv\config\applicationHost.config`

<span data-ttu-id="bf97d-598">**IIS Express**</span><span class="sxs-lookup"><span data-stu-id="bf97d-598">**IIS Express**</span></span>

* <span data-ttu-id="bf97d-599">Visual Studio: `{APPLICATION ROOT}\.vs\config\applicationHost.config`</span><span class="sxs-lookup"><span data-stu-id="bf97d-599">Visual Studio: `{APPLICATION ROOT}\.vs\config\applicationHost.config`</span></span>

* <span data-ttu-id="bf97d-600">*iisexpress.exe* CLI `%USERPROFILE%\Documents\IISExpress\config\applicationhost.config`</span><span class="sxs-lookup"><span data-stu-id="bf97d-600">*iisexpress.exe* CLI: `%USERPROFILE%\Documents\IISExpress\config\applicationhost.config`</span></span>

<span data-ttu-id="bf97d-601">È possibile trovare i file eseguendo una ricerca `aspnetcore` nel `applicationHost.config` file.</span><span class="sxs-lookup"><span data-stu-id="bf97d-601">The files can be found by searching for `aspnetcore` in the `applicationHost.config` file.</span></span>

::: moniker-end

::: moniker range="< aspnetcore-2.2"

<span data-ttu-id="bf97d-602">Il modulo ASP.NET Core è un modulo IIS nativo collegato alla pipeline di IIS che inoltra le richieste Web alle app back-end ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="bf97d-602">The ASP.NET Core Module is a native IIS module that plugs into the IIS pipeline to forward web requests to backend ASP.NET Core apps.</span></span>

<span data-ttu-id="bf97d-603">Versioni supportate di Windows:</span><span class="sxs-lookup"><span data-stu-id="bf97d-603">Supported Windows versions:</span></span>

* <span data-ttu-id="bf97d-604">Windows 7 o versione successiva</span><span class="sxs-lookup"><span data-stu-id="bf97d-604">Windows 7 or later</span></span>
* <span data-ttu-id="bf97d-605">Windows Server 2008 R2 o versione successiva</span><span class="sxs-lookup"><span data-stu-id="bf97d-605">Windows Server 2008 R2 or later</span></span>

<span data-ttu-id="bf97d-606">Il modulo funziona solo con Kestrel.</span><span class="sxs-lookup"><span data-stu-id="bf97d-606">The module only works with Kestrel.</span></span> <span data-ttu-id="bf97d-607">Il modulo non è compatibile con [HTTP.sys](xref:fundamentals/servers/httpsys).</span><span class="sxs-lookup"><span data-stu-id="bf97d-607">The module is incompatible with [HTTP.sys](xref:fundamentals/servers/httpsys).</span></span>

<span data-ttu-id="bf97d-608">Poiché le app ASP.NET Core vengono eseguite in un processo distinto dal processo di lavoro IIS, il modulo esegue anche la gestione dei processi.</span><span class="sxs-lookup"><span data-stu-id="bf97d-608">Because ASP.NET Core apps run in a process separate from the IIS worker process, the module also handles process management.</span></span> <span data-ttu-id="bf97d-609">Il modulo avvia il processo per l'app ASP.NET Core quando arriva la prima richiesta e riavvia l'app se si verifica un arresto anomalo di questa.</span><span class="sxs-lookup"><span data-stu-id="bf97d-609">The module starts the process for the ASP.NET Core app when the first request arrives and restarts the app if it crashes.</span></span> <span data-ttu-id="bf97d-610">Si tratta essenzialmente dello stesso comportamento delle app ASP.NET 4.x eseguite In-Process in IIS e gestite dal [servizio Attivazione processo Windows (WAS, Windows Activation Service)](/iis/manage/provisioning-and-managing-iis/features-of-the-windows-process-activation-service-was).</span><span class="sxs-lookup"><span data-stu-id="bf97d-610">This is essentially the same behavior as seen with ASP.NET 4.x apps that run in-process in IIS that are managed by the [Windows Process Activation Service (WAS)](/iis/manage/provisioning-and-managing-iis/features-of-the-windows-process-activation-service-was).</span></span>

<span data-ttu-id="bf97d-611">Il diagramma seguente illustra la relazione tra IIS, il modulo ASP.NET Core e un'app:</span><span class="sxs-lookup"><span data-stu-id="bf97d-611">The following diagram illustrates the relationship between IIS, the ASP.NET Core Module, and an app:</span></span>

![Modulo ASP.NET Core](iis/index/_static/ancm-outofprocess.png)

<span data-ttu-id="bf97d-613">Le richieste arrivano dal Web al driver HTTP.sys in modalità kernel.</span><span class="sxs-lookup"><span data-stu-id="bf97d-613">Requests arrive from the web to the kernel-mode HTTP.sys driver.</span></span> <span data-ttu-id="bf97d-614">Il driver instrada le richieste a IIS sulla porta configurata per il sito Web, in genere 80 (HTTP) o 443 (HTTPS).</span><span class="sxs-lookup"><span data-stu-id="bf97d-614">The driver routes the requests to IIS on the website's configured port, usually 80 (HTTP) or 443 (HTTPS).</span></span> <span data-ttu-id="bf97d-615">Il modulo inoltra le richieste a Kestrel su una porta casuale per l'app non corrispondente alla porta 80 o 443.</span><span class="sxs-lookup"><span data-stu-id="bf97d-615">The module forwards the requests to Kestrel on a random port for the app, which isn't port 80 or 443.</span></span>

<span data-ttu-id="bf97d-616">Il modulo specifica la porta tramite una variabile di ambiente all'avvio e il [middleware di integrazione di IIS](xref:host-and-deploy/iis/index#enable-the-iisintegration-components) configura il server per l'ascolto `http://localhost:{port}` .</span><span class="sxs-lookup"><span data-stu-id="bf97d-616">The module specifies the port via an environment variable at startup, and the [IIS Integration Middleware](xref:host-and-deploy/iis/index#enable-the-iisintegration-components) configures the server to listen on `http://localhost:{port}`.</span></span> <span data-ttu-id="bf97d-617">Vengono eseguiti controlli aggiuntivi e le richieste che non provengono dal modulo vengono rifiutate.</span><span class="sxs-lookup"><span data-stu-id="bf97d-617">Additional checks are performed, and requests that don't originate from the module are rejected.</span></span> <span data-ttu-id="bf97d-618">Il modulo non supporta l'inoltro HTTPS, pertanto le richieste vengono inoltrate tramite HTTP anche se sono state ricevute da IIS tramite HTTPS.</span><span class="sxs-lookup"><span data-stu-id="bf97d-618">The module doesn't support HTTPS forwarding, so requests are forwarded over HTTP even if received by IIS over HTTPS.</span></span>

<span data-ttu-id="bf97d-619">Dopo che Kestrel ha prelevato la richiesta dal modulo, viene eseguito il push della richiesta nella pipeline middleware ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="bf97d-619">After Kestrel picks up the request from the module, the request is pushed into the ASP.NET Core middleware pipeline.</span></span> <span data-ttu-id="bf97d-620">La pipeline middleware gestisce la richiesta e la passa come istanza di `HttpContext` alla logica dell'app.</span><span class="sxs-lookup"><span data-stu-id="bf97d-620">The middleware pipeline handles the request and passes it on as an `HttpContext` instance to the app's logic.</span></span> <span data-ttu-id="bf97d-621">Il middleware aggiunto dall'integrazione di IIS aggiorna lo schema, l'IP remoto e il percorso di base all'account per l'inoltro della richiesta a Kestrel.</span><span class="sxs-lookup"><span data-stu-id="bf97d-621">Middleware added by IIS Integration updates the scheme, remote IP, and pathbase to account for forwarding the request to Kestrel.</span></span> <span data-ttu-id="bf97d-622">La risposta dell'app viene quindi passata a IIS, che ne esegue di nuovo il push al client HTTP che ha avviato la richiesta.</span><span class="sxs-lookup"><span data-stu-id="bf97d-622">The app's response is passed back to IIS, which pushes it back out to the HTTP client that initiated the request.</span></span>

<span data-ttu-id="bf97d-623">Molti moduli nativi, ad esempio l'autenticazione di Windows, rimangono attivi.</span><span class="sxs-lookup"><span data-stu-id="bf97d-623">Many native modules, such as Windows Authentication, remain active.</span></span> <span data-ttu-id="bf97d-624">Per altre informazioni sui moduli IIS attivi con il modulo ASP.NET Core, vedere <xref:host-and-deploy/iis/modules>.</span><span class="sxs-lookup"><span data-stu-id="bf97d-624">To learn more about IIS modules active with the ASP.NET Core Module, see <xref:host-and-deploy/iis/modules>.</span></span>

<span data-ttu-id="bf97d-625">Il modulo ASP.NET Core può anche:</span><span class="sxs-lookup"><span data-stu-id="bf97d-625">The ASP.NET Core Module can also:</span></span>

* <span data-ttu-id="bf97d-626">Impostare variabili di ambiente per il processo di lavoro.</span><span class="sxs-lookup"><span data-stu-id="bf97d-626">Set environment variables for the worker process.</span></span>
* <span data-ttu-id="bf97d-627">Registrare output stdout in una risorsa di archiviazione di file per la risoluzione di problemi di avvio.</span><span class="sxs-lookup"><span data-stu-id="bf97d-627">Log stdout output to file storage for troubleshooting startup issues.</span></span>
* <span data-ttu-id="bf97d-628">Inoltrare token di autenticazione di Windows.</span><span class="sxs-lookup"><span data-stu-id="bf97d-628">Forward Windows authentication tokens.</span></span>

## <a name="how-to-install-and-use-the-aspnet-core-module"></a><span data-ttu-id="bf97d-629">Come installare e usare il modulo ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="bf97d-629">How to install and use the ASP.NET Core Module</span></span>

<span data-ttu-id="bf97d-630">Per istruzioni su come installare e usare il modulo ASP.NET Core, vedere [Installare il bundle di hosting .NET Core](xref:host-and-deploy/iis/index#install-the-net-core-hosting-bundle).</span><span class="sxs-lookup"><span data-stu-id="bf97d-630">For instructions on how to install the ASP.NET Core Module, see [Install the .NET Core Hosting Bundle](xref:host-and-deploy/iis/index#install-the-net-core-hosting-bundle).</span></span>

## <a name="configuration-with-webconfig"></a><span data-ttu-id="bf97d-631">Configurazione con web.config</span><span class="sxs-lookup"><span data-stu-id="bf97d-631">Configuration with web.config</span></span>

<span data-ttu-id="bf97d-632">Il modulo ASP.NET Core viene configurato tramite la sezione `aspNetCore` del nodo `system.webServer` nel file *web.config* del sito.</span><span class="sxs-lookup"><span data-stu-id="bf97d-632">The ASP.NET Core Module is configured with the `aspNetCore` section of the `system.webServer` node in the site's *web.config* file.</span></span>

<span data-ttu-id="bf97d-633">Il file *web.config* seguente viene pubblicato per una [distribuzione dipendente dal framework](/dotnet/articles/core/deploying/#framework-dependent-deployments-fdd) e configura il modulo ASP.NET Core per gestire le richieste del sito:</span><span class="sxs-lookup"><span data-stu-id="bf97d-633">The following *web.config* file is published for a [framework-dependent deployment](/dotnet/articles/core/deploying/#framework-dependent-deployments-fdd) and configures the ASP.NET Core Module to handle site requests:</span></span>

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <system.webServer>
    <handlers>
      <add name="aspNetCore" path="*" verb="*" modules="AspNetCoreModule" resourceType="Unspecified" />
    </handlers>
    <aspNetCore processPath="dotnet"
                arguments=".\MyApp.dll"
                stdoutLogEnabled="false"
                stdoutLogFile=".\logs\stdout" />
  </system.webServer>
</configuration>
```

<span data-ttu-id="bf97d-634">Il file *web.config* seguente viene pubblicato per una [distribuzione autonoma](/dotnet/articles/core/deploying/#self-contained-deployments-scd):</span><span class="sxs-lookup"><span data-stu-id="bf97d-634">The following *web.config* is published for a [self-contained deployment](/dotnet/articles/core/deploying/#self-contained-deployments-scd):</span></span>

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <system.webServer>
    <handlers>
      <add name="aspNetCore" path="*" verb="*" modules="AspNetCoreModule" resourceType="Unspecified" />
    </handlers>
    <aspNetCore processPath=".\MyApp.exe"
                stdoutLogEnabled="false"
                stdoutLogFile=".\logs\stdout" />
  </system.webServer>
</configuration>
```

<span data-ttu-id="bf97d-635">Quando un'app viene distribuita in [Servizio app di Azure](https://azure.microsoft.com/services/app-service/), il percorso `stdoutLogFile` è impostato su `\\?\%home%\LogFiles\stdout`.</span><span class="sxs-lookup"><span data-stu-id="bf97d-635">When an app is deployed to [Azure App Service](https://azure.microsoft.com/services/app-service/), the `stdoutLogFile` path is set to `\\?\%home%\LogFiles\stdout`.</span></span> <span data-ttu-id="bf97d-636">Il percorso salva i log stdout nella cartella *LogFiles*, ovvero una posizione creata automaticamente dal servizio.</span><span class="sxs-lookup"><span data-stu-id="bf97d-636">The path saves stdout logs to the *LogFiles* folder, which is a location automatically created by the service.</span></span>

<span data-ttu-id="bf97d-637">Per informazioni sulla configurazione delle applicazioni secondarie IIS, vedere <xref:host-and-deploy/iis/index#sub-applications>.</span><span class="sxs-lookup"><span data-stu-id="bf97d-637">For information on IIS sub-application configuration, see <xref:host-and-deploy/iis/index#sub-applications>.</span></span>

### <a name="attributes-of-the-aspnetcore-element"></a><span data-ttu-id="bf97d-638">Attributi dell'elemento aspNetCore</span><span class="sxs-lookup"><span data-stu-id="bf97d-638">Attributes of the aspNetCore element</span></span>

| <span data-ttu-id="bf97d-639">Attributo</span><span class="sxs-lookup"><span data-stu-id="bf97d-639">Attribute</span></span> | <span data-ttu-id="bf97d-640">Descrizione</span><span class="sxs-lookup"><span data-stu-id="bf97d-640">Description</span></span> | <span data-ttu-id="bf97d-641">Predefinito</span><span class="sxs-lookup"><span data-stu-id="bf97d-641">Default</span></span> |
| --------- | ----------- | :-----: |
| `arguments` | <p><span data-ttu-id="bf97d-642">Attributo stringa facoltativo.</span><span class="sxs-lookup"><span data-stu-id="bf97d-642">Optional string attribute.</span></span></p><p><span data-ttu-id="bf97d-643">Argomenti per l'eseguibile specificato in **processPath**.</span><span class="sxs-lookup"><span data-stu-id="bf97d-643">Arguments to the executable specified in **processPath**.</span></span></p>| |
| `disableStartUpErrorPage` | <p><span data-ttu-id="bf97d-644">Attributo booleano facoltativo.</span><span class="sxs-lookup"><span data-stu-id="bf97d-644">Optional Boolean attribute.</span></span></p><p><span data-ttu-id="bf97d-645">Se true, la pagina **502.5 - Errore del processo** non viene visualizzata e la tabella codici di stato 502 configurata in *web.config* ha la precedenza.</span><span class="sxs-lookup"><span data-stu-id="bf97d-645">If true, the **502.5 - Process Failure** page is suppressed, and the 502 status code page configured in the *web.config* takes precedence.</span></span></p> | `false` |
| `forwardWindowsAuthToken` | <p><span data-ttu-id="bf97d-646">Attributo booleano facoltativo.</span><span class="sxs-lookup"><span data-stu-id="bf97d-646">Optional Boolean attribute.</span></span></p><p><span data-ttu-id="bf97d-647">Se true, il token viene inoltrato al processo figlio in ascolto su %ASPNETCORE_PORT% come un'intestazione 'MS-ASPNETCORE-WINAUTHTOKEN' per ogni richiesta.</span><span class="sxs-lookup"><span data-stu-id="bf97d-647">If true, the token is forwarded to the child process listening on %ASPNETCORE_PORT% as a header 'MS-ASPNETCORE-WINAUTHTOKEN' per request.</span></span> <span data-ttu-id="bf97d-648">È responsabilità del processo chiamare CloseHandle su questo token per ogni richiesta.</span><span class="sxs-lookup"><span data-stu-id="bf97d-648">It's the responsibility of that process to call CloseHandle on this token per request.</span></span></p> | `true` |
| `processesPerApplication` | <p><span data-ttu-id="bf97d-649">Attributo Integer facoltativo.</span><span class="sxs-lookup"><span data-stu-id="bf97d-649">Optional integer attribute.</span></span></p><p><span data-ttu-id="bf97d-650">Specifica il numero di istanze del processo specificato nell'impostazione **processPath** che può essere riattivato per ogni app.</span><span class="sxs-lookup"><span data-stu-id="bf97d-650">Specifies the number of instances of the process specified in the **processPath** setting that can be spun up per app.</span></span></p><p><span data-ttu-id="bf97d-651">L'impostazione di `processesPerApplication` è sconsigliata.</span><span class="sxs-lookup"><span data-stu-id="bf97d-651">Setting `processesPerApplication` is discouraged.</span></span> <span data-ttu-id="bf97d-652">Questo attributo sarà rimosso nelle versioni future.</span><span class="sxs-lookup"><span data-stu-id="bf97d-652">This attribute will be removed in a future release.</span></span></p> | <span data-ttu-id="bf97d-653">Valore predefinito: `1`</span><span class="sxs-lookup"><span data-stu-id="bf97d-653">Default: `1`</span></span><br><span data-ttu-id="bf97d-654">Min: `1`</span><span class="sxs-lookup"><span data-stu-id="bf97d-654">Min: `1`</span></span><br><span data-ttu-id="bf97d-655">Max: `100`</span><span class="sxs-lookup"><span data-stu-id="bf97d-655">Max: `100`</span></span> |
| `processPath` | <p><span data-ttu-id="bf97d-656">Attributo stringa obbligatorio.</span><span class="sxs-lookup"><span data-stu-id="bf97d-656">Required string attribute.</span></span></p><p><span data-ttu-id="bf97d-657">Percorso del file eseguibile che avvia un processo in ascolto delle richieste HTTP.</span><span class="sxs-lookup"><span data-stu-id="bf97d-657">Path to the executable that launches a process listening for HTTP requests.</span></span> <span data-ttu-id="bf97d-658">I percorsi relativi sono supportati.</span><span class="sxs-lookup"><span data-stu-id="bf97d-658">Relative paths are supported.</span></span> <span data-ttu-id="bf97d-659">Se il percorso inizia con `.`, viene considerato relativo alla radice del sito.</span><span class="sxs-lookup"><span data-stu-id="bf97d-659">If the path begins with `.`, the path is considered to be relative to the site root.</span></span></p> | |
| `rapidFailsPerMinute` | <p><span data-ttu-id="bf97d-660">Attributo Integer facoltativo.</span><span class="sxs-lookup"><span data-stu-id="bf97d-660">Optional integer attribute.</span></span></p><p><span data-ttu-id="bf97d-661">Specifica il numero di arresti anomali al minuto per il processo specificato in **processPath**.</span><span class="sxs-lookup"><span data-stu-id="bf97d-661">Specifies the number of times the process specified in **processPath** is allowed to crash per minute.</span></span> <span data-ttu-id="bf97d-662">Se questo limite viene superato, il modulo smette di avviare il processo per la parte restante del minuto.</span><span class="sxs-lookup"><span data-stu-id="bf97d-662">If this limit is exceeded, the module stops launching the process for the remainder of the minute.</span></span></p> | <span data-ttu-id="bf97d-663">Valore predefinito: `10`</span><span class="sxs-lookup"><span data-stu-id="bf97d-663">Default: `10`</span></span><br><span data-ttu-id="bf97d-664">Min: `0`</span><span class="sxs-lookup"><span data-stu-id="bf97d-664">Min: `0`</span></span><br><span data-ttu-id="bf97d-665">Max: `100`</span><span class="sxs-lookup"><span data-stu-id="bf97d-665">Max: `100`</span></span> |
| `requestTimeout` | <p><span data-ttu-id="bf97d-666">Attributo Timespan facoltativo.</span><span class="sxs-lookup"><span data-stu-id="bf97d-666">Optional timespan attribute.</span></span></p><p><span data-ttu-id="bf97d-667">Specifica la durata per cui il modulo ASP.NET Core attende una risposta dal processo in ascolto su %ASPNETCORE_PORT%.</span><span class="sxs-lookup"><span data-stu-id="bf97d-667">Specifies the duration for which the ASP.NET Core Module waits for a response from the process listening on %ASPNETCORE_PORT%.</span></span></p><p><span data-ttu-id="bf97d-668">Nelle versioni del modulo ASP.NET Core fornito con ASP.NET Core 2.1 o versioni successive, `requestTimeout` viene specificato in ore, minuti e secondi.</span><span class="sxs-lookup"><span data-stu-id="bf97d-668">In versions of the ASP.NET Core Module that shipped with the release of ASP.NET Core 2.1 or later, the `requestTimeout` is specified in hours, minutes, and seconds.</span></span></p> | <span data-ttu-id="bf97d-669">Valore predefinito: `00:02:00`</span><span class="sxs-lookup"><span data-stu-id="bf97d-669">Default: `00:02:00`</span></span><br><span data-ttu-id="bf97d-670">Min: `00:00:00`</span><span class="sxs-lookup"><span data-stu-id="bf97d-670">Min: `00:00:00`</span></span><br><span data-ttu-id="bf97d-671">Max: `360:00:00`</span><span class="sxs-lookup"><span data-stu-id="bf97d-671">Max: `360:00:00`</span></span> |
| `shutdownTimeLimit` | <p><span data-ttu-id="bf97d-672">Attributo Integer facoltativo.</span><span class="sxs-lookup"><span data-stu-id="bf97d-672">Optional integer attribute.</span></span></p><p><span data-ttu-id="bf97d-673">Durata in secondi per cui il modulo attende che il file eseguibile venga arrestato normalmente quando viene rilevato il file *app_offline.htm*.</span><span class="sxs-lookup"><span data-stu-id="bf97d-673">Duration in seconds that the module waits for the executable to gracefully shutdown when the *app_offline.htm* file is detected.</span></span></p> | <span data-ttu-id="bf97d-674">Valore predefinito: `10`</span><span class="sxs-lookup"><span data-stu-id="bf97d-674">Default: `10`</span></span><br><span data-ttu-id="bf97d-675">Min: `0`</span><span class="sxs-lookup"><span data-stu-id="bf97d-675">Min: `0`</span></span><br><span data-ttu-id="bf97d-676">Max: `600`</span><span class="sxs-lookup"><span data-stu-id="bf97d-676">Max: `600`</span></span> |
| `startupTimeLimit` | <p><span data-ttu-id="bf97d-677">Attributo Integer facoltativo.</span><span class="sxs-lookup"><span data-stu-id="bf97d-677">Optional integer attribute.</span></span></p><p><span data-ttu-id="bf97d-678">Durata in secondi per cui il modulo attende l'avvio di un processo in ascolto sulla porta da parte del file eseguibile.</span><span class="sxs-lookup"><span data-stu-id="bf97d-678">Duration in seconds that the module waits for the executable to start a process listening on the port.</span></span> <span data-ttu-id="bf97d-679">Se questo limite di tempo viene superato, il modulo termina il processo.</span><span class="sxs-lookup"><span data-stu-id="bf97d-679">If this time limit is exceeded, the module kills the process.</span></span> <span data-ttu-id="bf97d-680">Il modulo tenta di avviare nuovamente il processo quando riceve una nuova richiesta e continua a tentare di riavviare il processo alle successive richieste in ingresso, a meno che non risulti impossibile avviare l'app un numero di volte pari a **rapidFailsPerMinute** nell'ultimo minuto continuo.</span><span class="sxs-lookup"><span data-stu-id="bf97d-680">The module attempts to relaunch the process when it receives a new request and continues to attempt to restart the process on subsequent incoming requests unless the app fails to start **rapidFailsPerMinute** number of times in the last rolling minute.</span></span></p><p><span data-ttu-id="bf97d-681">Un valore pari a 0 (zero) **non** è considerato un timeout infinito.</span><span class="sxs-lookup"><span data-stu-id="bf97d-681">A value of 0 (zero) is **not** considered an infinite timeout.</span></span></p> | <span data-ttu-id="bf97d-682">Valore predefinito: `120`</span><span class="sxs-lookup"><span data-stu-id="bf97d-682">Default: `120`</span></span><br><span data-ttu-id="bf97d-683">Min: `0`</span><span class="sxs-lookup"><span data-stu-id="bf97d-683">Min: `0`</span></span><br><span data-ttu-id="bf97d-684">Max: `3600`</span><span class="sxs-lookup"><span data-stu-id="bf97d-684">Max: `3600`</span></span> |
| `stdoutLogEnabled` | <p><span data-ttu-id="bf97d-685">Attributo booleano facoltativo.</span><span class="sxs-lookup"><span data-stu-id="bf97d-685">Optional Boolean attribute.</span></span></p><p><span data-ttu-id="bf97d-686">Se true, **stdout** e **stderr** per il processo specificato in **processPath** vengono reindirizzati al file specificato in **stdoutLogFile**.</span><span class="sxs-lookup"><span data-stu-id="bf97d-686">If true, **stdout** and **stderr** for the process specified in **processPath** are redirected to the file specified in **stdoutLogFile**.</span></span></p> | `false` |
| `stdoutLogFile` | <p><span data-ttu-id="bf97d-687">Attributo stringa facoltativo.</span><span class="sxs-lookup"><span data-stu-id="bf97d-687">Optional string attribute.</span></span></p><p><span data-ttu-id="bf97d-688">Specifica il percorso relativo o assoluto per cui vengono registrati **stdout** e **stderr** dal processo specificato in **processPath**.</span><span class="sxs-lookup"><span data-stu-id="bf97d-688">Specifies the relative or absolute file path for which **stdout** and **stderr** from the process specified in **processPath** are logged.</span></span> <span data-ttu-id="bf97d-689">I percorsi relativi sono relativi alla radice del sito.</span><span class="sxs-lookup"><span data-stu-id="bf97d-689">Relative paths are relative to the root of the site.</span></span> <span data-ttu-id="bf97d-690">Qualsiasi percorso che inizia con `.` è relativo al sito radice e tutti gli altri percorsi vengono trattati come percorsi assoluti.</span><span class="sxs-lookup"><span data-stu-id="bf97d-690">Any path starting with `.` are relative to the site root and all other paths are treated as absolute paths.</span></span> <span data-ttu-id="bf97d-691">Le eventuali cartelle specificate nel percorso devono essere già esistenti affinché il modulo possa creare il file di log.</span><span class="sxs-lookup"><span data-stu-id="bf97d-691">Any folders provided in the path must exist in order for the module to create the log file.</span></span> <span data-ttu-id="bf97d-692">Usando il carattere di sottolineatura come delimitatore, il timestamp, l'ID processo e l'estensione del file (*.log*) vengono aggiunti all'ultimo segmento del percorso **stdoutLogFile**.</span><span class="sxs-lookup"><span data-stu-id="bf97d-692">Using underscore delimiters, a timestamp, process ID, and file extension (*.log*) are added to the last segment of the **stdoutLogFile** path.</span></span> <span data-ttu-id="bf97d-693">Se si specifica `.\logs\stdout` come valore, un log stdout di esempio salvato il 5/2/2018 alle 19:41:32 con un ID processo 1934 viene salvato come *stdout_20180205194132_1934.log* nella cartella *logs*.</span><span class="sxs-lookup"><span data-stu-id="bf97d-693">If `.\logs\stdout` is supplied as a value, an example stdout log is saved as *stdout_20180205194132_1934.log* in the *logs* folder when saved on 2/5/2018 at 19:41:32 with a process ID of 1934.</span></span></p> | `aspnetcore-stdout` |

### <a name="setting-environment-variables"></a><span data-ttu-id="bf97d-694">Impostazioni delle variabili di ambiente</span><span class="sxs-lookup"><span data-stu-id="bf97d-694">Setting environment variables</span></span>

<span data-ttu-id="bf97d-695">È possibile specificare le variabili di ambiente per il processo nell'attributo `processPath`.</span><span class="sxs-lookup"><span data-stu-id="bf97d-695">Environment variables can be specified for the process in the `processPath` attribute.</span></span> <span data-ttu-id="bf97d-696">Specificare una variabile di ambiente con l'elemento figlio `<environmentVariable>` di un elemento della raccolta `<environmentVariables>`.</span><span class="sxs-lookup"><span data-stu-id="bf97d-696">Specify an environment variable with the `<environmentVariable>` child element of an `<environmentVariables>` collection element.</span></span>

> [!WARNING]
> <span data-ttu-id="bf97d-697">Le variabili di ambiente impostate in questa sezione sono in conflitto con le variabili di ambiente di sistema impostate con lo stesso nome.</span><span class="sxs-lookup"><span data-stu-id="bf97d-697">Environment variables set in this section conflict with system environment variables set with the same name.</span></span> <span data-ttu-id="bf97d-698">Se una variabile di ambiente è impostata sia nel file *web.config* sia a livello di sistema in Windows, il valore del file *web.config* viene aggiunto al valore della variabile di ambiente di sistema, ad esempio `ASPNETCORE_ENVIRONMENT: Development;Development`, e ciò impedisce l'avvio dell'app.</span><span class="sxs-lookup"><span data-stu-id="bf97d-698">If an environment variable is set in both the *web.config* file and at the system level in Windows, the value from the *web.config* file becomes appended to the system environment variable value (for example, `ASPNETCORE_ENVIRONMENT: Development;Development`), which prevents the app from starting.</span></span>

<span data-ttu-id="bf97d-699">Nell'esempio seguente vengono impostate due variabili di ambiente.</span><span class="sxs-lookup"><span data-stu-id="bf97d-699">The following example sets two environment variables.</span></span> <span data-ttu-id="bf97d-700">`ASPNETCORE_ENVIRONMENT` configura l'ambiente dell'app su `Development`.</span><span class="sxs-lookup"><span data-stu-id="bf97d-700">`ASPNETCORE_ENVIRONMENT` configures the app's environment to `Development`.</span></span> <span data-ttu-id="bf97d-701">Uno sviluppatore può impostare temporaneamente questo valore nel file *web.config* per forzare il caricamento della [pagina delle eccezioni per gli sviluppatori](xref:fundamentals/error-handling) durante il debug di un'eccezione dell'app.</span><span class="sxs-lookup"><span data-stu-id="bf97d-701">A developer may temporarily set this value in the *web.config* file in order to force the [Developer Exception Page](xref:fundamentals/error-handling) to load when debugging an app exception.</span></span> <span data-ttu-id="bf97d-702">`CONFIG_DIR` è un esempio di variabile di ambiente definita dall'utente, in cui lo sviluppatore ha scritto il codice che legge il valore all'avvio in modo da formare un percorso per il caricamento del file di configurazione dell'app.</span><span class="sxs-lookup"><span data-stu-id="bf97d-702">`CONFIG_DIR` is an example of a user-defined environment variable, where the developer has written code that reads the value on startup to form a path for loading the app's configuration file.</span></span>

```xml
<aspNetCore processPath="dotnet"
      arguments=".\MyApp.dll"
      stdoutLogEnabled="false"
      stdoutLogFile="\\?\%home%\LogFiles\stdout">
  <environmentVariables>
    <environmentVariable name="ASPNETCORE_ENVIRONMENT" value="Development" />
    <environmentVariable name="CONFIG_DIR" value="f:\application_config" />
  </environmentVariables>
</aspNetCore>
```

> [!WARNING]
> <span data-ttu-id="bf97d-703">Impostare la variabile di ambiente `ASPNETCORE_ENVIRONMENT` su `Development` solo in server di gestione temporanea e test che non sono accessibili da reti non attendibili, ad esempio Internet.</span><span class="sxs-lookup"><span data-stu-id="bf97d-703">Only set the `ASPNETCORE_ENVIRONMENT` environment variable to `Development` on staging and testing servers that aren't accessible to untrusted networks, such as the Internet.</span></span>

## <a name="app_offlinehtm"></a><span data-ttu-id="bf97d-704">app_offline.htm</span><span class="sxs-lookup"><span data-stu-id="bf97d-704">app_offline.htm</span></span>

<span data-ttu-id="bf97d-705">Se un file denominato *app_offline.htm* viene rilevato nella directory radice di un'app, il modulo ASP.NET Core tenta di arrestare normalmente l'app e arresta l'elaborazione delle richieste in ingresso.</span><span class="sxs-lookup"><span data-stu-id="bf97d-705">If a file with the name *app_offline.htm* is detected in the root directory of an app, the ASP.NET Core Module attempts to gracefully shutdown the app and stop processing incoming requests.</span></span> <span data-ttu-id="bf97d-706">Se l'app è ancora in esecuzione dopo il numero di secondi definito in `shutdownTimeLimit`, il modulo ASP.NET Core termina il processo in esecuzione.</span><span class="sxs-lookup"><span data-stu-id="bf97d-706">If the app is still running after the number of seconds defined in `shutdownTimeLimit`, the ASP.NET Core Module kills the running process.</span></span>

<span data-ttu-id="bf97d-707">Mentre il file *app_offline.htm* è presente, il modulo ASP.NET Core risponde alle richieste restituendo il contenuto del file *app_offline.htm*.</span><span class="sxs-lookup"><span data-stu-id="bf97d-707">While the *app_offline.htm* file is present, the ASP.NET Core Module responds to requests by sending back the contents of the *app_offline.htm* file.</span></span> <span data-ttu-id="bf97d-708">Quando il file *app_offline.htm* viene rimosso, la richiesta successiva avvia l'app.</span><span class="sxs-lookup"><span data-stu-id="bf97d-708">When the *app_offline.htm* file is removed, the next request starts the app.</span></span>

## <a name="start-up-error-page"></a><span data-ttu-id="bf97d-709">Pagina di errore di avvio</span><span class="sxs-lookup"><span data-stu-id="bf97d-709">Start-up error page</span></span>

<span data-ttu-id="bf97d-710">Se il modulo ASP.NET Core non riesce ad avviare il processo di back-end o il processo di back-end viene avviato ma non riesce a restare in ascolto sulla porta configurata, verrà visualizzata la tabella codici di stato *502.5 - Errore del processo*.</span><span class="sxs-lookup"><span data-stu-id="bf97d-710">If the ASP.NET Core Module fails to launch the backend process or the backend process starts but fails to listen on the configured port, a *502.5 - Process Failure* status code page appears.</span></span> <span data-ttu-id="bf97d-711">Per non visualizzare questa pagina e tornare alla tabella codici di stato 502 predefinita di IIS, usare l'attributo `disableStartUpErrorPage`.</span><span class="sxs-lookup"><span data-stu-id="bf97d-711">To suppress this page and revert to the default IIS 502 status code page, use the `disableStartUpErrorPage` attribute.</span></span> <span data-ttu-id="bf97d-712">Per altre informazioni sulla configurazione di messaggi di errore personalizzati, vedere [Errori HTTP \<httpErrors>](/iis/configuration/system.webServer/httpErrors/).</span><span class="sxs-lookup"><span data-stu-id="bf97d-712">For more information on configuring custom error messages, see [HTTP Errors \<httpErrors>](/iis/configuration/system.webServer/httpErrors/).</span></span>

## <a name="log-creation-and-redirection"></a><span data-ttu-id="bf97d-713">Creazione e reindirizzamento dei log</span><span class="sxs-lookup"><span data-stu-id="bf97d-713">Log creation and redirection</span></span>

<span data-ttu-id="bf97d-714">Il modulo ASP.NET Core reindirizza su disco l'output della console stdout e stderr se sono impostati gli attributi `stdoutLogEnabled` e `stdoutLogFile` dell'elemento `aspNetCore`.</span><span class="sxs-lookup"><span data-stu-id="bf97d-714">The ASP.NET Core Module redirects stdout and stderr console output to disk if the `stdoutLogEnabled` and `stdoutLogFile` attributes of the `aspNetCore` element are set.</span></span> <span data-ttu-id="bf97d-715">Le eventuali le cartelle nel percorso `stdoutLogFile` vengono create dal modulo quando viene creato il file di log.</span><span class="sxs-lookup"><span data-stu-id="bf97d-715">Any folders in the `stdoutLogFile` path are created by the module when the log file is created.</span></span> <span data-ttu-id="bf97d-716">Il pool di app deve avere accesso in scrittura alla posizione in cui vengono scritti i log (usare `IIS AppPool\<app_pool_name>` per specificare l'autorizzazione di scrittura).</span><span class="sxs-lookup"><span data-stu-id="bf97d-716">The app pool must have write access to the location where the logs are written (use `IIS AppPool\<app_pool_name>` to provide write permission).</span></span>

<span data-ttu-id="bf97d-717">I log non vengono ruotati, a meno che non si verifichi il riciclo/riavvio del processo.</span><span class="sxs-lookup"><span data-stu-id="bf97d-717">Logs aren't rotated, unless process recycling/restart occurs.</span></span> <span data-ttu-id="bf97d-718">È responsabilità del provider di servizi di hosting limitare lo spazio su disco usato dai log.</span><span class="sxs-lookup"><span data-stu-id="bf97d-718">It's the responsibility of the hoster to limit the disk space the logs consume.</span></span>

<span data-ttu-id="bf97d-719">L'uso del log stdout è consigliato solo per la risoluzione dei problemi di avvio dell'app durante l'hosting in IIS o quando si usa il [supporto in fase di sviluppo per IIS con Visual Studio](xref:host-and-deploy/iis/development-time-iis-support), non durante il debug in locale e l'esecuzione dell'app con IIS Express.</span><span class="sxs-lookup"><span data-stu-id="bf97d-719">Using the stdout log is only recommended for troubleshooting app startup issues when hosting on IIS or when using [development-time support for IIS with Visual Studio](xref:host-and-deploy/iis/development-time-iis-support), not while debugging locally and running the app with IIS Express.</span></span>

<span data-ttu-id="bf97d-720">Non usare il log stdout per scopi di registrazione generale delle app.</span><span class="sxs-lookup"><span data-stu-id="bf97d-720">Don't use the stdout log for general app logging purposes.</span></span> <span data-ttu-id="bf97d-721">Per la registrazione di routine in un'app ASP.NET Core, usare una libreria di registrazione che limita le dimensioni dei file di log e ne esegue la rotazione.</span><span class="sxs-lookup"><span data-stu-id="bf97d-721">For routine logging in an ASP.NET Core app, use a logging library that limits log file size and rotates logs.</span></span> <span data-ttu-id="bf97d-722">Per altre informazioni, vedere [Provider di registrazione di terze parti](xref:fundamentals/logging/index#third-party-logging-providers).</span><span class="sxs-lookup"><span data-stu-id="bf97d-722">For more information, see [third-party logging providers](xref:fundamentals/logging/index#third-party-logging-providers).</span></span>

<span data-ttu-id="bf97d-723">Un timestamp e l'estensione del file vengono aggiunti automaticamente al momento della creazione del file di log.</span><span class="sxs-lookup"><span data-stu-id="bf97d-723">A timestamp and file extension are added automatically when the log file is created.</span></span> <span data-ttu-id="bf97d-724">Il nome del file di log è composto aggiungendo il timestamp, l'ID processo e l'estensione del file (*.log*) all'ultimo segmento del percorso `stdoutLogFile` (in genere *stdout*), con caratteri di sottolineatura come delimitatori.</span><span class="sxs-lookup"><span data-stu-id="bf97d-724">The log file name is composed by appending the timestamp, process ID, and file extension (*.log*) to the last segment of the `stdoutLogFile` path (typically *stdout*) delimited by underscores.</span></span> <span data-ttu-id="bf97d-725">Se il percorso `stdoutLogFile` termina con *stdout*, un log per un'app con un PID 1934 creata il 5/2/2018 alle 19:42:32 sarà denominato *stdout_20180205194132_1934.log*.</span><span class="sxs-lookup"><span data-stu-id="bf97d-725">If the `stdoutLogFile` path ends with *stdout*, a log for an app with a PID of 1934 created on 2/5/2018 at 19:42:32 has the file name *stdout_20180205194132_1934.log*.</span></span>

<span data-ttu-id="bf97d-726">Nell'elemento di esempio seguente viene `aspNetCore` configurata la registrazione di stdout nel percorso relativo `.\log\` .</span><span class="sxs-lookup"><span data-stu-id="bf97d-726">The following sample `aspNetCore` element configures stdout logging at the relative path `.\log\`.</span></span> <span data-ttu-id="bf97d-727">Verificare che l'identità dell'utente AppPool disponga dell'autorizzazione di scrittura per il percorso specificato.</span><span class="sxs-lookup"><span data-stu-id="bf97d-727">Confirm that the AppPool user identity has permission to write to the path provided.</span></span>

```xml
<aspNetCore processPath="dotnet"
    arguments=".\MyApp.dll"
    stdoutLogEnabled="true"
    stdoutLogFile=".\logs\stdout">
</aspNetCore>
```

<span data-ttu-id="bf97d-728">Quando si pubblica un'app per la distribuzione di app Azure Service, il valore viene impostato da Web SDK `stdoutLogFile` `\\?\%home%\LogFiles\stdout` .</span><span class="sxs-lookup"><span data-stu-id="bf97d-728">When publishing an app for Azure App Service deployment, the Web SDK sets the `stdoutLogFile` value to `\\?\%home%\LogFiles\stdout`.</span></span> <span data-ttu-id="bf97d-729">La `%home` variabile di ambiente è predefinita per le app ospitate dal servizio app Azure.</span><span class="sxs-lookup"><span data-stu-id="bf97d-729">The `%home` environment variable is predefined for apps hosted by Azure App Service.</span></span>

<span data-ttu-id="bf97d-730">Per creare regole di filtro di registrazione, vedere le sezioni [configurazione](xref:fundamentals/logging/index#log-filtering) e [filtro dei log](xref:fundamentals/logging/index#log-filtering) della documentazione relativa alla registrazione del ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="bf97d-730">To create logging filter rules, see the [Configuration](xref:fundamentals/logging/index#log-filtering) and [Log filtering](xref:fundamentals/logging/index#log-filtering) sections of the ASP.NET Core logging documentation.</span></span>

<span data-ttu-id="bf97d-731">Per ulteriori informazioni sui formati di percorso, vedere [formati di percorso dei file nei sistemi Windows](/dotnet/standard/io/file-path-formats).</span><span class="sxs-lookup"><span data-stu-id="bf97d-731">For more information on path formats, see [File path formats on Windows systems](/dotnet/standard/io/file-path-formats).</span></span>

## <a name="proxy-configuration-uses-http-protocol-and-a-pairing-token"></a><span data-ttu-id="bf97d-732">La configurazione del proxy usa il protocollo HTTP e un token di associazione</span><span class="sxs-lookup"><span data-stu-id="bf97d-732">Proxy configuration uses HTTP protocol and a pairing token</span></span>

<span data-ttu-id="bf97d-733">Il proxy creato tra il modulo ASP.NET Core e Kestrel usa il protocollo HTTP.</span><span class="sxs-lookup"><span data-stu-id="bf97d-733">The proxy created between the ASP.NET Core Module and Kestrel uses the HTTP protocol.</span></span> <span data-ttu-id="bf97d-734">Non vi è alcun rischio di intercettazione del traffico tra il modulo e Kestrel da una posizione esterna al server.</span><span class="sxs-lookup"><span data-stu-id="bf97d-734">There's no risk of eavesdropping the traffic between the module and Kestrel from a location off of the server.</span></span>

<span data-ttu-id="bf97d-735">Viene usato un token di associazione per garantire che le richieste ricevute da Kestrel siano state trasmesse tramite proxy da IIS e non provengano da un'altra origine.</span><span class="sxs-lookup"><span data-stu-id="bf97d-735">A pairing token is used to guarantee that the requests received by Kestrel were proxied by IIS and didn't come from some other source.</span></span> <span data-ttu-id="bf97d-736">Il token di associazione viene creato e impostato in una variabile di ambiente (`ASPNETCORE_TOKEN`) dal modulo.</span><span class="sxs-lookup"><span data-stu-id="bf97d-736">The pairing token is created and set into an environment variable (`ASPNETCORE_TOKEN`) by the module.</span></span> <span data-ttu-id="bf97d-737">Il token di associazione viene impostato anche in un'intestazione (`MS-ASPNETCORE-TOKEN`) in tutte le richieste trasmesse tramite proxy.</span><span class="sxs-lookup"><span data-stu-id="bf97d-737">The pairing token is also set into a header (`MS-ASPNETCORE-TOKEN`) on every proxied request.</span></span> <span data-ttu-id="bf97d-738">Il middleware di IIS controlla ogni richiesta che riceve in modo da confermare che il valore dell'intestazione del token di associazione corrisponda al valore della variabile di ambiente.</span><span class="sxs-lookup"><span data-stu-id="bf97d-738">IIS Middleware checks each request it receives to confirm that the pairing token header value matches the environment variable value.</span></span> <span data-ttu-id="bf97d-739">Se i valori del token non corrispondono, la richiesta viene registrata e rifiutata.</span><span class="sxs-lookup"><span data-stu-id="bf97d-739">If the token values are mismatched, the request is logged and rejected.</span></span> <span data-ttu-id="bf97d-740">La variabile di ambiente del token di associazione e il traffico tra il modulo e Kestrel non sono accessibili da una posizione esterna al server.</span><span class="sxs-lookup"><span data-stu-id="bf97d-740">The pairing token environment variable and the traffic between the module and Kestrel aren't accessible from a location off of the server.</span></span> <span data-ttu-id="bf97d-741">Senza conoscere il valore del token di associazione, un utente malintenzionato non può inviare richieste che ignorano il controllo nel middleware di IIS.</span><span class="sxs-lookup"><span data-stu-id="bf97d-741">Without knowing the pairing token value, an attacker can't submit requests that bypass the check in the IIS Middleware.</span></span>

## <a name="aspnet-core-module-with-an-iis-shared-configuration"></a><span data-ttu-id="bf97d-742">Modulo ASP.NET Core con configurazione condivisa di IIS</span><span class="sxs-lookup"><span data-stu-id="bf97d-742">ASP.NET Core Module with an IIS Shared Configuration</span></span>

<span data-ttu-id="bf97d-743">Il programma di installazione del modulo ASP.NET Core viene eseguito con i privilegi dell'account **TrustedInstaller**.</span><span class="sxs-lookup"><span data-stu-id="bf97d-743">The ASP.NET Core Module installer runs with the privileges of the **TrustedInstaller** account.</span></span> <span data-ttu-id="bf97d-744">Poiché l'account di sistema locale non dispone dell'autorizzazione di modifica per il percorso della condivisione usato dalla configurazione condivisa di IIS, il programma di installazione genera un errore di accesso negato durante il tentativo di configurare le impostazioni del modulo nel file *applicationHost.config* nella condivisione.</span><span class="sxs-lookup"><span data-stu-id="bf97d-744">Because the local system account doesn't have modify permission for the share path used by the IIS Shared Configuration, the installer throws an access denied error when attempting to configure the module settings in the *applicationHost.config* file on the share.</span></span>

<span data-ttu-id="bf97d-745">Quando si usa una configurazione condivisa di IIS, attenersi ai passaggi riportati di seguito:</span><span class="sxs-lookup"><span data-stu-id="bf97d-745">When using an IIS Shared Configuration, follow these steps:</span></span>

1. <span data-ttu-id="bf97d-746">Disabilitare la configurazione condivisa di IIS.</span><span class="sxs-lookup"><span data-stu-id="bf97d-746">Disable the IIS Shared Configuration.</span></span>
1. <span data-ttu-id="bf97d-747">Eseguire il programma di installazione.</span><span class="sxs-lookup"><span data-stu-id="bf97d-747">Run the installer.</span></span>
1. <span data-ttu-id="bf97d-748">Esportare il file *applicationHost.config* aggiornato nella condivisione.</span><span class="sxs-lookup"><span data-stu-id="bf97d-748">Export the updated *applicationHost.config* file to the share.</span></span>
1. <span data-ttu-id="bf97d-749">Abilitare di nuovo la configurazione condivisa di IIS.</span><span class="sxs-lookup"><span data-stu-id="bf97d-749">Re-enable the IIS Shared Configuration.</span></span>

## <a name="module-version-and-hosting-bundle-installer-logs"></a><span data-ttu-id="bf97d-750">Versione del modulo e log del programma di installazione del bundle di hosting</span><span class="sxs-lookup"><span data-stu-id="bf97d-750">Module version and Hosting Bundle installer logs</span></span>

<span data-ttu-id="bf97d-751">Per determinare la versione del modulo ASP.NET Core installato:</span><span class="sxs-lookup"><span data-stu-id="bf97d-751">To determine the version of the installed ASP.NET Core Module:</span></span>

1. <span data-ttu-id="bf97d-752">Nel sistema host passare a *%windir%\System32\inetsrv*.</span><span class="sxs-lookup"><span data-stu-id="bf97d-752">On the hosting system, navigate to *%windir%\System32\inetsrv*.</span></span>
1. <span data-ttu-id="bf97d-753">Individuare il file *aspnetcore.dll*.</span><span class="sxs-lookup"><span data-stu-id="bf97d-753">Locate the *aspnetcore.dll* file.</span></span>
1. <span data-ttu-id="bf97d-754">Fare clic con il pulsante destro del mouse sul file e scegliere **Proprietà** dal menu di scelta rapida.</span><span class="sxs-lookup"><span data-stu-id="bf97d-754">Right-click the file and select **Properties** from the contextual menu.</span></span>
1. <span data-ttu-id="bf97d-755">Selezionare la scheda **Dettagli** . La versione del **file** e la **versione del prodotto** rappresentano la versione installata del modulo.</span><span class="sxs-lookup"><span data-stu-id="bf97d-755">Select the **Details** tab. The **File version** and **Product version** represent the installed version of the module.</span></span>

<span data-ttu-id="bf97d-756">I log del programma di installazione del bundle di hosting per il modulo sono disponibili in *C: \\ utenti \\ % username% \\ AppData \\ Local \\ Temp*. Il file è denominato *dd_DotNetCoreWinSvrHosting__ \<timestamp> _000_AspNetCoreModule_x64. log*.</span><span class="sxs-lookup"><span data-stu-id="bf97d-756">The Hosting Bundle installer logs for the module are found at *C:\\Users\\%UserName%\\AppData\\Local\\Temp*. The file is named *dd_DotNetCoreWinSvrHosting__\<timestamp>_000_AspNetCoreModule_x64.log*.</span></span>

## <a name="module-schema-and-configuration-file-locations"></a><span data-ttu-id="bf97d-757">Percorsi dei file di modulo, schema e configurazione</span><span class="sxs-lookup"><span data-stu-id="bf97d-757">Module, schema, and configuration file locations</span></span>

### <a name="module"></a><span data-ttu-id="bf97d-758">Modulo</span><span class="sxs-lookup"><span data-stu-id="bf97d-758">Module</span></span>

<span data-ttu-id="bf97d-759">**IIS (x86/amd64):**</span><span class="sxs-lookup"><span data-stu-id="bf97d-759">**IIS (x86/amd64):**</span></span>

* <span data-ttu-id="bf97d-760">%windir%\System32\inetsrv\aspnetcore.dll</span><span class="sxs-lookup"><span data-stu-id="bf97d-760">%windir%\System32\inetsrv\aspnetcore.dll</span></span>

* <span data-ttu-id="bf97d-761">%windir%\SysWOW64\inetsrv\aspnetcore.dll</span><span class="sxs-lookup"><span data-stu-id="bf97d-761">%windir%\SysWOW64\inetsrv\aspnetcore.dll</span></span>

<span data-ttu-id="bf97d-762">**IIS Express (x86/amd64):**</span><span class="sxs-lookup"><span data-stu-id="bf97d-762">**IIS Express (x86/amd64):**</span></span>

* <span data-ttu-id="bf97d-763">%ProgramFiles%\IIS Express\aspnetcore.dll</span><span class="sxs-lookup"><span data-stu-id="bf97d-763">%ProgramFiles%\IIS Express\aspnetcore.dll</span></span>

* <span data-ttu-id="bf97d-764">%ProgramFiles(x86)%\IIS Express\aspnetcore.dll</span><span class="sxs-lookup"><span data-stu-id="bf97d-764">%ProgramFiles(x86)%\IIS Express\aspnetcore.dll</span></span>

### <a name="schema"></a><span data-ttu-id="bf97d-765">SCHEMA</span><span class="sxs-lookup"><span data-stu-id="bf97d-765">Schema</span></span>

<span data-ttu-id="bf97d-766">**IIS**</span><span class="sxs-lookup"><span data-stu-id="bf97d-766">**IIS**</span></span>

* <span data-ttu-id="bf97d-767">%windir%\System32\inetsrv\config\schema\aspnetcore_schema.xml</span><span class="sxs-lookup"><span data-stu-id="bf97d-767">%windir%\System32\inetsrv\config\schema\aspnetcore_schema.xml</span></span>

<span data-ttu-id="bf97d-768">**IIS Express**</span><span class="sxs-lookup"><span data-stu-id="bf97d-768">**IIS Express**</span></span>

* <span data-ttu-id="bf97d-769">%ProgramFiles%\IIS Express\config\schema\aspnetcore_schema.xml</span><span class="sxs-lookup"><span data-stu-id="bf97d-769">%ProgramFiles%\IIS Express\config\schema\aspnetcore_schema.xml</span></span>

### <a name="configuration"></a><span data-ttu-id="bf97d-770">Configurazione</span><span class="sxs-lookup"><span data-stu-id="bf97d-770">Configuration</span></span>

<span data-ttu-id="bf97d-771">**IIS**</span><span class="sxs-lookup"><span data-stu-id="bf97d-771">**IIS**</span></span>

* <span data-ttu-id="bf97d-772">%windir%\System32\inetsrv\config\applicationHost.config</span><span class="sxs-lookup"><span data-stu-id="bf97d-772">%windir%\System32\inetsrv\config\applicationHost.config</span></span>

<span data-ttu-id="bf97d-773">**IIS Express**</span><span class="sxs-lookup"><span data-stu-id="bf97d-773">**IIS Express**</span></span>

* <span data-ttu-id="bf97d-774">Visual Studio: {APPLICATION ROOT}\\.vs\config\applicationHost.config</span><span class="sxs-lookup"><span data-stu-id="bf97d-774">Visual Studio: {APPLICATION ROOT}\\.vs\config\applicationHost.config</span></span>

* <span data-ttu-id="bf97d-775">Interfaccia della riga di comando di *iisexpress.exe*: %USERPROFILE%\Documents\IISExpress\config\applicationhost.config</span><span class="sxs-lookup"><span data-stu-id="bf97d-775">*iisexpress.exe* CLI: %USERPROFILE%\Documents\IISExpress\config\applicationhost.config</span></span>

<span data-ttu-id="bf97d-776">È possibile trovare i file eseguendo una ricerca di *aspnetcore* nel file *applicationHost.config*.</span><span class="sxs-lookup"><span data-stu-id="bf97d-776">The files can be found by searching for *aspnetcore* in the *applicationHost.config* file.</span></span>

::: moniker-end

## <a name="additional-resources"></a><span data-ttu-id="bf97d-777">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="bf97d-777">Additional resources</span></span>

* <xref:host-and-deploy/iis/index>
* <xref:host-and-deploy/azure-apps/index>
* <span data-ttu-id="bf97d-778">[Origine riferimento modulo ASP.NET Core [ramo predefinito (Master)]](https://github.com/dotnet/aspnetcore/tree/main/src/Servers/IIS/AspNetCoreModuleV2): usare l'elenco a discesa **Branch** per selezionare una versione specifica (ad esempio, `release/3.1` ).</span><span class="sxs-lookup"><span data-stu-id="bf97d-778">[ASP.NET Core Module reference source [default branch (master)]](https://github.com/dotnet/aspnetcore/tree/main/src/Servers/IIS/AspNetCoreModuleV2): Use the **Branch** drop down list to select a specific release (for example, `release/3.1`).</span></span>
* <xref:host-and-deploy/iis/modules>
