---
title: Rilevare le modifiche apportate con i token di modifica in ASP.NET Core
author: rick-anderson
description: Informazioni su come usare i token di modifica per rilevare le modifiche.
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
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
uid: fundamentals/change-tokens
ms.openlocfilehash: df2be1b89ad9681ff70dd71cb3026786f59c8b2a
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102589387"
---
# <a name="detect-changes-with-change-tokens-in-aspnet-core"></a><span data-ttu-id="2acd3-103">Rilevare le modifiche apportate con i token di modifica in ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="2acd3-103">Detect changes with change tokens in ASP.NET Core</span></span>

::: moniker range=">= aspnetcore-3.0"

<span data-ttu-id="2acd3-104">Un *token di modifica* è un blocco predefinito di uso generico e di basso livello che viene usato per il rilevamento delle modifiche di stato.</span><span class="sxs-lookup"><span data-stu-id="2acd3-104">A *change token* is a general-purpose, low-level building block used to track state changes.</span></span>

<span data-ttu-id="2acd3-105">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/change-tokens/samples/) ([procedura per il download](xref:index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="2acd3-105">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/change-tokens/samples/) ([how to download](xref:index#how-to-download-a-sample))</span></span>

## <a name="ichangetoken-interface"></a><span data-ttu-id="2acd3-106">Interfaccia IChangeToken</span><span class="sxs-lookup"><span data-stu-id="2acd3-106">IChangeToken interface</span></span>

<span data-ttu-id="2acd3-107"><xref:Microsoft.Extensions.Primitives.IChangeToken> propaga le notifiche relative a una modifica apportata.</span><span class="sxs-lookup"><span data-stu-id="2acd3-107"><xref:Microsoft.Extensions.Primitives.IChangeToken> propagates notifications that a change has occurred.</span></span> <span data-ttu-id="2acd3-108">`IChangeToken` risiede nello spazio dei nomi <xref:Microsoft.Extensions.Primitives?displayProperty=fullName>.</span><span class="sxs-lookup"><span data-stu-id="2acd3-108">`IChangeToken` resides in the <xref:Microsoft.Extensions.Primitives?displayProperty=fullName> namespace.</span></span> <span data-ttu-id="2acd3-109">Il pacchetto NuGet [Microsoft. Extensions. Primitives](https://www.nuget.org/packages/Microsoft.Extensions.Primitives/) viene fornito in modo implicito alle app ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="2acd3-109">The [Microsoft.Extensions.Primitives](https://www.nuget.org/packages/Microsoft.Extensions.Primitives/) NuGet package is implicitly provided to the ASP.NET Core apps.</span></span>

<span data-ttu-id="2acd3-110">`IChangeToken` ha due proprietà:</span><span class="sxs-lookup"><span data-stu-id="2acd3-110">`IChangeToken` has two properties:</span></span>

* <span data-ttu-id="2acd3-111"><xref:Microsoft.Extensions.Primitives.IChangeToken.ActiveChangeCallbacks> indica se il token genera callback in modo proattivo.</span><span class="sxs-lookup"><span data-stu-id="2acd3-111"><xref:Microsoft.Extensions.Primitives.IChangeToken.ActiveChangeCallbacks> indicate if the token proactively raises callbacks.</span></span> <span data-ttu-id="2acd3-112">Se `ActiveChangedCallbacks` è impostato su `false`, non viene mai chiamato alcun callback e l'app deve eseguire il polling di `HasChanged` per ottenere le modifiche.</span><span class="sxs-lookup"><span data-stu-id="2acd3-112">If `ActiveChangedCallbacks` is set to `false`, a callback is never called, and the app must poll `HasChanged` for changes.</span></span> <span data-ttu-id="2acd3-113">È anche possibile che un token non venga mai annullato se non vengono apportate modifiche o se il listener di modifica sottostante viene eliminato o disabilitato.</span><span class="sxs-lookup"><span data-stu-id="2acd3-113">It's also possible for a token to never be cancelled if no changes occur or the underlying change listener is disposed or disabled.</span></span>
* <span data-ttu-id="2acd3-114"><xref:Microsoft.Extensions.Primitives.IChangeToken.HasChanged> riceve un valore che indica se è stata apportata una modifica.</span><span class="sxs-lookup"><span data-stu-id="2acd3-114"><xref:Microsoft.Extensions.Primitives.IChangeToken.HasChanged> receives a value that indicates if a change has occurred.</span></span>

<span data-ttu-id="2acd3-115">L' `IChangeToken` interfaccia include il metodo [RegisterChangeCallback (Action \<Object> , Object)](xref:Microsoft.Extensions.Primitives.IChangeToken.RegisterChangeCallback*) , che registra un callback che viene richiamato quando il token viene modificato.</span><span class="sxs-lookup"><span data-stu-id="2acd3-115">The `IChangeToken` interface includes the [RegisterChangeCallback(Action\<Object>, Object)](xref:Microsoft.Extensions.Primitives.IChangeToken.RegisterChangeCallback*) method, which registers a callback that's invoked when the token has changed.</span></span> <span data-ttu-id="2acd3-116">Prima che il callback venga richiamato è necessario impostare `HasChanged`.</span><span class="sxs-lookup"><span data-stu-id="2acd3-116">`HasChanged` must be set before the callback is invoked.</span></span>

## <a name="changetoken-class"></a><span data-ttu-id="2acd3-117">Classe ChangeToken</span><span class="sxs-lookup"><span data-stu-id="2acd3-117">ChangeToken class</span></span>

<span data-ttu-id="2acd3-118"><xref:Microsoft.Extensions.Primitives.ChangeToken> è una classe statica usata per propagare le notifiche relative a una modifica che è stata apportata.</span><span class="sxs-lookup"><span data-stu-id="2acd3-118"><xref:Microsoft.Extensions.Primitives.ChangeToken> is a static class used to propagate notifications that a change has occurred.</span></span> <span data-ttu-id="2acd3-119">`ChangeToken` risiede nello spazio dei nomi <xref:Microsoft.Extensions.Primitives?displayProperty=fullName>.</span><span class="sxs-lookup"><span data-stu-id="2acd3-119">`ChangeToken` resides in the <xref:Microsoft.Extensions.Primitives?displayProperty=fullName> namespace.</span></span> <span data-ttu-id="2acd3-120">Il pacchetto NuGet [Microsoft. Extensions. Primitives](https://www.nuget.org/packages/Microsoft.Extensions.Primitives/) viene fornito in modo implicito alle app ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="2acd3-120">The [Microsoft.Extensions.Primitives](https://www.nuget.org/packages/Microsoft.Extensions.Primitives/) NuGet package is implicitly provided to the ASP.NET Core apps.</span></span>

<span data-ttu-id="2acd3-121">Il metodo [token. OnChange (Func \<IChangeToken> , Action)](xref:Microsoft.Extensions.Primitives.ChangeToken.OnChange*) registra un oggetto `Action` da chiamare ogni volta che il token cambia:</span><span class="sxs-lookup"><span data-stu-id="2acd3-121">The [ChangeToken.OnChange(Func\<IChangeToken>, Action)](xref:Microsoft.Extensions.Primitives.ChangeToken.OnChange*) method registers an `Action` to call whenever the token changes:</span></span>

* <span data-ttu-id="2acd3-122">`Func<IChangeToken>` produce il token.</span><span class="sxs-lookup"><span data-stu-id="2acd3-122">`Func<IChangeToken>` produces the token.</span></span>
* <span data-ttu-id="2acd3-123">`Action` viene chiamato alla modifica del token.</span><span class="sxs-lookup"><span data-stu-id="2acd3-123">`Action` is called when the token changes.</span></span>

<span data-ttu-id="2acd3-124">L'overload di [token. OnChange \<TState> (Func \<IChangeToken> , Action \<TState> , tstate)](xref:Microsoft.Extensions.Primitives.ChangeToken.OnChange*) accetta un `TState` parametro aggiuntivo passato al consumer del token `Action` .</span><span class="sxs-lookup"><span data-stu-id="2acd3-124">The [ChangeToken.OnChange\<TState>(Func\<IChangeToken>, Action\<TState>, TState)](xref:Microsoft.Extensions.Primitives.ChangeToken.OnChange*) overload takes an additional `TState` parameter that's passed into the token consumer `Action`.</span></span>

<span data-ttu-id="2acd3-125">`OnChange` restituisce un oggetto <xref:System.IDisposable>.</span><span class="sxs-lookup"><span data-stu-id="2acd3-125">`OnChange` returns an <xref:System.IDisposable>.</span></span> <span data-ttu-id="2acd3-126">La chiamata a <xref:System.IDisposable.Dispose*> interrompe l'ascolto di altre modifiche da parte del token e rilascia le risorse del token.</span><span class="sxs-lookup"><span data-stu-id="2acd3-126">Calling <xref:System.IDisposable.Dispose*> stops the token from listening for further changes and releases the token's resources.</span></span>

## <a name="example-uses-of-change-tokens-in-aspnet-core"></a><span data-ttu-id="2acd3-127">Esempi di uso di token di modifica in ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="2acd3-127">Example uses of change tokens in ASP.NET Core</span></span>

<span data-ttu-id="2acd3-128">I token di modifica vengono usati nelle aree principali di ASP.NET Core per monitorare le modifiche apportate agli oggetti:</span><span class="sxs-lookup"><span data-stu-id="2acd3-128">Change tokens are used in prominent areas of ASP.NET Core to monitor for changes to objects:</span></span>

* <span data-ttu-id="2acd3-129">Per monitorare le modifiche ai file, il metodo <xref:Microsoft.Extensions.FileProviders.IFileProvider.Watch*> di <xref:Microsoft.Extensions.FileProviders.IFileProvider> crea un elemento `IChangeToken` per le cartelle o i file specificati da controllare.</span><span class="sxs-lookup"><span data-stu-id="2acd3-129">For monitoring changes to files, <xref:Microsoft.Extensions.FileProviders.IFileProvider>'s <xref:Microsoft.Extensions.FileProviders.IFileProvider.Watch*> method creates an `IChangeToken` for the specified files or folder to watch.</span></span>
* <span data-ttu-id="2acd3-130">È possibile aggiungere token `IChangeToken` alle voci della cache per attivare le eliminazioni dalla cache alla modifica.</span><span class="sxs-lookup"><span data-stu-id="2acd3-130">`IChangeToken` tokens can be added to cache entries to trigger cache evictions on change.</span></span>
* <span data-ttu-id="2acd3-131">Per le modifiche di `TOptions`, l'implementazione <xref:Microsoft.Extensions.Options.OptionsMonitor`1> predefinita di <xref:Microsoft.Extensions.Options.IOptionsMonitor`1> ha un overload che accetta una o più istanze di <xref:Microsoft.Extensions.Options.IOptionsChangeTokenSource`1>.</span><span class="sxs-lookup"><span data-stu-id="2acd3-131">For `TOptions` changes, the default <xref:Microsoft.Extensions.Options.OptionsMonitor`1> implementation of <xref:Microsoft.Extensions.Options.IOptionsMonitor`1> has an overload that accepts one or more <xref:Microsoft.Extensions.Options.IOptionsChangeTokenSource`1> instances.</span></span> <span data-ttu-id="2acd3-132">Ogni istanza restituisce un elemento `IChangeToken` per registrare un callback di notifica delle modifiche per il rilevamento delle modifiche apportate alle opzioni.</span><span class="sxs-lookup"><span data-stu-id="2acd3-132">Each instance returns an `IChangeToken` to register a change notification callback for tracking options changes.</span></span>

## <a name="monitor-for-configuration-changes"></a><span data-ttu-id="2acd3-133">Monitorare le modifiche di configurazione</span><span class="sxs-lookup"><span data-stu-id="2acd3-133">Monitor for configuration changes</span></span>

<span data-ttu-id="2acd3-134">Per impostazione predefinita, i modelli di ASP.NET Core usano [file di configurazione JSON](xref:fundamentals/configuration/index#json-configuration-provider) ( *appsettings.json* , *appsettings.Development.js* e *appsettings.Production.json*) per caricare le impostazioni di configurazione dell'app.</span><span class="sxs-lookup"><span data-stu-id="2acd3-134">By default, ASP.NET Core templates use [JSON configuration files](xref:fundamentals/configuration/index#json-configuration-provider) (*appsettings.json*, *appsettings.Development.json*, and *appsettings.Production.json*) to load app configuration settings.</span></span>

<span data-ttu-id="2acd3-135">Questi file vengono configurati usando il metodo di estensione [AddJsonFile(IConfigurationBuilder, String, Boolean, Boolean)](xref:Microsoft.Extensions.Configuration.JsonConfigurationExtensions.AddJsonFile*) per <xref:Microsoft.Extensions.Configuration.ConfigurationBuilder> che accetta un parametro `reloadOnChange`.</span><span class="sxs-lookup"><span data-stu-id="2acd3-135">These files are configured using the [AddJsonFile(IConfigurationBuilder, String, Boolean, Boolean)](xref:Microsoft.Extensions.Configuration.JsonConfigurationExtensions.AddJsonFile*) extension method on <xref:Microsoft.Extensions.Configuration.ConfigurationBuilder> that accepts a `reloadOnChange` parameter.</span></span> <span data-ttu-id="2acd3-136">`reloadOnChange` indica se la configurazione deve essere ricaricata alla modifica del file.</span><span class="sxs-lookup"><span data-stu-id="2acd3-136">`reloadOnChange` indicates if configuration should be reloaded on file changes.</span></span> <span data-ttu-id="2acd3-137">Questa impostazione viene visualizzata nel metodo pratico <xref:Microsoft.Extensions.Hosting.Host><xref:Microsoft.Extensions.Hosting.Host.CreateDefaultBuilder*>:</span><span class="sxs-lookup"><span data-stu-id="2acd3-137">This setting appears in the <xref:Microsoft.Extensions.Hosting.Host> convenience method <xref:Microsoft.Extensions.Hosting.Host.CreateDefaultBuilder*>:</span></span>

```csharp
config.AddJsonFile("appsettings.json", optional: true, reloadOnChange: true)
      .AddJsonFile($"appsettings.{env.EnvironmentName}.json", optional: true, 
          reloadOnChange: true);
```

<span data-ttu-id="2acd3-138">La configurazione basata su file è rappresentata da <xref:Microsoft.Extensions.Configuration.FileConfigurationSource>.</span><span class="sxs-lookup"><span data-stu-id="2acd3-138">File-based configuration is represented by <xref:Microsoft.Extensions.Configuration.FileConfigurationSource>.</span></span> <span data-ttu-id="2acd3-139">`FileConfigurationSource` usa <xref:Microsoft.Extensions.FileProviders.IFileProvider> per monitorare i file.</span><span class="sxs-lookup"><span data-stu-id="2acd3-139">`FileConfigurationSource` uses <xref:Microsoft.Extensions.FileProviders.IFileProvider> to monitor files.</span></span>

<span data-ttu-id="2acd3-140">Per impostazione predefinita, `IFileMonitor` viene offerto da una classe <xref:Microsoft.Extensions.FileProviders.PhysicalFileProvider> che usa <xref:System.IO.FileSystemWatcher> per monitorare le modifiche apportate al file di configurazione.</span><span class="sxs-lookup"><span data-stu-id="2acd3-140">By default, the `IFileMonitor` is provided by a <xref:Microsoft.Extensions.FileProviders.PhysicalFileProvider>, which uses <xref:System.IO.FileSystemWatcher> to monitor for configuration file changes.</span></span>

<span data-ttu-id="2acd3-141">L'app di esempio illustra due implementazioni per il monitoraggio delle modifiche alla configurazione.</span><span class="sxs-lookup"><span data-stu-id="2acd3-141">The sample app demonstrates two implementations for monitoring configuration changes.</span></span> <span data-ttu-id="2acd3-142">In caso di modifica di uno qualsiasi dei file appsettings&mdash;, entrambe le implementazioni di monitoraggio dei file eseguono codice personalizzato e l'app di esempio scrive un messaggio nella console.</span><span class="sxs-lookup"><span data-stu-id="2acd3-142">If any of the *appsettings* files change, both of the file monitoring implementations execute custom code&mdash;the sample app writes a message to the console.</span></span>

<span data-ttu-id="2acd3-143">L'elemento `FileSystemWatcher` di un file di configurazione può attivare più callback del token per una singola modifica del file di configurazione.</span><span class="sxs-lookup"><span data-stu-id="2acd3-143">A configuration file's `FileSystemWatcher` can trigger multiple token callbacks for a single configuration file change.</span></span> <span data-ttu-id="2acd3-144">Per assicurarsi che il codice personalizzato venga eseguito solo una volta quando vengono attivati più callback del token, l'implementazione dell'esempio controlla gli hash dei file.</span><span class="sxs-lookup"><span data-stu-id="2acd3-144">To ensure that the custom code is only run once when multiple token callbacks are triggered, the sample's implementation checks file hashes.</span></span> <span data-ttu-id="2acd3-145">L'esempio usa l'hash di file SHA1.</span><span class="sxs-lookup"><span data-stu-id="2acd3-145">The sample uses SHA1 file hashing.</span></span> <span data-ttu-id="2acd3-146">Viene implementato un nuovo tentativo con un'interruzione temporanea esponenziale.</span><span class="sxs-lookup"><span data-stu-id="2acd3-146">A retry is implemented with an exponential back-off.</span></span> <span data-ttu-id="2acd3-147">Il nuovo tentativo è presente perché potrebbe verificarsi un blocco di file che impedisce temporaneamente l'elaborazione di un nuovo hash in un file.</span><span class="sxs-lookup"><span data-stu-id="2acd3-147">The retry is present because file locking may occur that temporarily prevents computing a new hash on a file.</span></span>

<span data-ttu-id="2acd3-148">*Utilities/Utilities.cs*:</span><span class="sxs-lookup"><span data-stu-id="2acd3-148">*Utilities/Utilities.cs*:</span></span>

[!code-csharp[](change-tokens/samples/3.x/SampleApp/Utilities/Utilities.cs?name=snippet1)]

### <a name="simple-startup-change-token"></a><span data-ttu-id="2acd3-149">Semplice token di modifica dell'avvio</span><span class="sxs-lookup"><span data-stu-id="2acd3-149">Simple startup change token</span></span>

<span data-ttu-id="2acd3-150">Registrare un callback dell'elemento `Action` consumer del token per le notifiche di modifica al token di ricaricamento della configurazione.</span><span class="sxs-lookup"><span data-stu-id="2acd3-150">Register a token consumer `Action` callback for change notifications to the configuration reload token.</span></span>

<span data-ttu-id="2acd3-151">In `Startup.Configure`:</span><span class="sxs-lookup"><span data-stu-id="2acd3-151">In `Startup.Configure`:</span></span>

[!code-csharp[](change-tokens/samples/3.x/SampleApp/Startup.cs?name=snippet2)]

<span data-ttu-id="2acd3-152">`config.GetReloadToken()` fornisce il token.</span><span class="sxs-lookup"><span data-stu-id="2acd3-152">`config.GetReloadToken()` provides the token.</span></span> <span data-ttu-id="2acd3-153">Il callback è il metodo `InvokeChanged`:</span><span class="sxs-lookup"><span data-stu-id="2acd3-153">The callback is the `InvokeChanged` method:</span></span>

[!code-csharp[](change-tokens/samples/3.x/SampleApp/Startup.cs?name=snippet3)]

<span data-ttu-id="2acd3-154">Il valore `state` del callback viene usato per passare `IWebHostEnvironment`, che è utile per specificare il file di configurazione *appsettings* corretto da monitorare (ad esempio, *appsettings.Development.json* nell'ambiente di sviluppo).</span><span class="sxs-lookup"><span data-stu-id="2acd3-154">The `state` of the callback is used to pass in the `IWebHostEnvironment`, which is useful for specifying the correct *appsettings* configuration file to monitor (for example, *appsettings.Development.json* when in the Development environment).</span></span> <span data-ttu-id="2acd3-155">Gli hash dei file vengono usati per impedire che l'istruzione `WriteConsole` venga eseguita più volte a causa di più callback del token quando il file di configurazione è stato modificato una sola volta.</span><span class="sxs-lookup"><span data-stu-id="2acd3-155">File hashes are used to prevent the `WriteConsole` statement from running multiple times due to multiple token callbacks when the configuration file has only changed once.</span></span>

<span data-ttu-id="2acd3-156">Questo sistema rimane in esecuzione finché l'app è in esecuzione e non può essere disabilitato dall'utente.</span><span class="sxs-lookup"><span data-stu-id="2acd3-156">This system runs as long as the app is running and can't be disabled by the user.</span></span>

### <a name="monitor-configuration-changes-as-a-service"></a><span data-ttu-id="2acd3-157">Monitorare le modifiche di configurazione come servizio</span><span class="sxs-lookup"><span data-stu-id="2acd3-157">Monitor configuration changes as a service</span></span>

<span data-ttu-id="2acd3-158">L'esempio implementa:</span><span class="sxs-lookup"><span data-stu-id="2acd3-158">The sample implements:</span></span>

* <span data-ttu-id="2acd3-159">Monitoraggio di base del token di avvio.</span><span class="sxs-lookup"><span data-stu-id="2acd3-159">Basic startup token monitoring.</span></span>
* <span data-ttu-id="2acd3-160">Monitoraggio come servizio.</span><span class="sxs-lookup"><span data-stu-id="2acd3-160">Monitoring as a service.</span></span>
* <span data-ttu-id="2acd3-161">Un meccanismo per abilitare e disabilitare il monitoraggio.</span><span class="sxs-lookup"><span data-stu-id="2acd3-161">A mechanism to enable and disable monitoring.</span></span>

<span data-ttu-id="2acd3-162">L'esempio stabilisce un'interfaccia `IConfigurationMonitor`.</span><span class="sxs-lookup"><span data-stu-id="2acd3-162">The sample establishes an `IConfigurationMonitor` interface.</span></span>

<span data-ttu-id="2acd3-163">*Extensions/ConfigurationMonitor.cs*:</span><span class="sxs-lookup"><span data-stu-id="2acd3-163">*Extensions/ConfigurationMonitor.cs*:</span></span>

[!code-csharp[](change-tokens/samples/3.x/SampleApp/Extensions/ConfigurationMonitor.cs?name=snippet1)]

<span data-ttu-id="2acd3-164">Il costruttore della classe implementata, `ConfigurationMonitor`, registra un callback per le notifiche di modifica:</span><span class="sxs-lookup"><span data-stu-id="2acd3-164">The constructor of the implemented class, `ConfigurationMonitor`, registers a callback for change notifications:</span></span>

[!code-csharp[](change-tokens/samples/3.x/SampleApp/Extensions/ConfigurationMonitor.cs?name=snippet2)]

<span data-ttu-id="2acd3-165">`config.GetReloadToken()` fornisce il token.</span><span class="sxs-lookup"><span data-stu-id="2acd3-165">`config.GetReloadToken()` supplies the token.</span></span> <span data-ttu-id="2acd3-166">`InvokeChanged` è il metodo di callback.</span><span class="sxs-lookup"><span data-stu-id="2acd3-166">`InvokeChanged` is the callback method.</span></span> <span data-ttu-id="2acd3-167">Il valore `state` in questa istanza è un riferimento all'istanza `IConfigurationMonitor` usata per accedere allo stato di monitoraggio.</span><span class="sxs-lookup"><span data-stu-id="2acd3-167">The `state` in this instance is a reference to the `IConfigurationMonitor` instance that's used to access the monitoring state.</span></span> <span data-ttu-id="2acd3-168">Vengono usate due proprietà:</span><span class="sxs-lookup"><span data-stu-id="2acd3-168">Two properties are used:</span></span>

* <span data-ttu-id="2acd3-169">`MonitoringEnabled`: Indica se il callback deve eseguire il codice personalizzato.</span><span class="sxs-lookup"><span data-stu-id="2acd3-169">`MonitoringEnabled`: Indicates if the callback should run its custom code.</span></span>
* <span data-ttu-id="2acd3-170">`CurrentState`: Descrive lo stato di monitoraggio corrente per l'uso nell'interfaccia utente.</span><span class="sxs-lookup"><span data-stu-id="2acd3-170">`CurrentState`: Describes the current monitoring state for use in the UI.</span></span>

<span data-ttu-id="2acd3-171">Il metodo `InvokeChanged` è simile all'approccio precedente, ad eccezione del fatto che:</span><span class="sxs-lookup"><span data-stu-id="2acd3-171">The `InvokeChanged` method is similar to the earlier approach, except that it:</span></span>

* <span data-ttu-id="2acd3-172">Non esegue il proprio codice a meno che `MonitoringEnabled` non sia impostato su `true`.</span><span class="sxs-lookup"><span data-stu-id="2acd3-172">Doesn't run its code unless `MonitoringEnabled` is `true`.</span></span>
* <span data-ttu-id="2acd3-173">Restituisce l'elemento `state` corrente nell'output `WriteConsole`.</span><span class="sxs-lookup"><span data-stu-id="2acd3-173">Outputs the current `state` in its `WriteConsole` output.</span></span>

[!code-csharp[](change-tokens/samples/3.x/SampleApp/Extensions/ConfigurationMonitor.cs?name=snippet3)]

<span data-ttu-id="2acd3-174">Un'istanza di `ConfigurationMonitor` viene registrata come servizio in `Startup.ConfigureServices`:</span><span class="sxs-lookup"><span data-stu-id="2acd3-174">An instance `ConfigurationMonitor` is registered as a service in `Startup.ConfigureServices`:</span></span>

[!code-csharp[](change-tokens/samples/3.x/SampleApp/Startup.cs?name=snippet1)]

<span data-ttu-id="2acd3-175">La pagina di indice offre all'utente il controllo sul monitoraggio della configurazione.</span><span class="sxs-lookup"><span data-stu-id="2acd3-175">The Index page offers the user control over configuration monitoring.</span></span> <span data-ttu-id="2acd3-176">L'istanza di `IConfigurationMonitor` viene inserita in `IndexModel`.</span><span class="sxs-lookup"><span data-stu-id="2acd3-176">The instance of `IConfigurationMonitor` is injected into the `IndexModel`.</span></span>

<span data-ttu-id="2acd3-177">*Pages/Index.cshtml.cs*:</span><span class="sxs-lookup"><span data-stu-id="2acd3-177">*Pages/Index.cshtml.cs*:</span></span>

[!code-csharp[](change-tokens/samples/3.x/SampleApp/Pages/Index.cshtml.cs?name=snippet1)]

<span data-ttu-id="2acd3-178">Il monitoraggio di configurazione (`_monitor`) viene usato per abilitare o disabilitare il monitoraggio e impostare lo stato corrente per commenti e suggerimenti dell'interfaccia utente:</span><span class="sxs-lookup"><span data-stu-id="2acd3-178">The configuration monitor (`_monitor`) is used to enable or disable monitoring and set the current state for UI feedback:</span></span>

[!code-csharp[](change-tokens/samples/3.x/SampleApp/Pages/Index.cshtml.cs?name=snippet2)]

<span data-ttu-id="2acd3-179">Quando viene attivato `OnPostStartMonitoring`, il monitoraggio è abilitato e lo stato corrente viene cancellato.</span><span class="sxs-lookup"><span data-stu-id="2acd3-179">When `OnPostStartMonitoring` is triggered, monitoring is enabled, and the current state is cleared.</span></span> <span data-ttu-id="2acd3-180">Quando viene attivato `OnPostStopMonitoring`, il monitoraggio è disabilitato e lo stato viene impostato in modo da indicare che il monitoraggio non viene eseguito.</span><span class="sxs-lookup"><span data-stu-id="2acd3-180">When `OnPostStopMonitoring` is triggered, monitoring is disabled, and the state is set to reflect that monitoring isn't occurring.</span></span>

<span data-ttu-id="2acd3-181">I pulsanti nell'interfaccia utente abilitano e disabilitano il monitoraggio.</span><span class="sxs-lookup"><span data-stu-id="2acd3-181">Buttons in the UI enable and disable monitoring.</span></span>

<span data-ttu-id="2acd3-182">*Pages/index. cshtml*:</span><span class="sxs-lookup"><span data-stu-id="2acd3-182">*Pages/Index.cshtml*:</span></span>

[!code-cshtml[](change-tokens/samples/3.x/SampleApp/Pages/Index.cshtml?name=snippet_Buttons)]

## <a name="monitor-cached-file-changes"></a><span data-ttu-id="2acd3-183">Monitorare le modifiche al file nella cache</span><span class="sxs-lookup"><span data-stu-id="2acd3-183">Monitor cached file changes</span></span>

<span data-ttu-id="2acd3-184">È possibile memorizzare il contenuto del file nella cache in memoria usando <xref:Microsoft.Extensions.Caching.Memory.IMemoryCache>.</span><span class="sxs-lookup"><span data-stu-id="2acd3-184">File content can be cached in-memory using <xref:Microsoft.Extensions.Caching.Memory.IMemoryCache>.</span></span> <span data-ttu-id="2acd3-185">La memorizzazione nella cache in memoria è descritta nell'argomento [Cache in memoria](xref:performance/caching/memory).</span><span class="sxs-lookup"><span data-stu-id="2acd3-185">In-memory caching is described in the [Cache in-memory](xref:performance/caching/memory) topic.</span></span> <span data-ttu-id="2acd3-186">Senza eseguire passaggi aggiuntivi, ad esempio l'implementazione descritta di seguito, la cache restituirà dati *non aggiornati* (obsoleti) se i dati di origine vengono modificati.</span><span class="sxs-lookup"><span data-stu-id="2acd3-186">Without taking additional steps, such as the implementation described below, *stale* (outdated) data is returned from a cache if the source data changes.</span></span>

<span data-ttu-id="2acd3-187">Ad esempio, se durante il rinnovo di un periodo di [scadenza variabile](xref:Microsoft.Extensions.Caching.Memory.MemoryCacheEntryOptions.SlidingExpiration) non si tiene conto dello stato di un file di origine memorizzato nella cache, i dati relativi ai file nella cache risulteranno non aggiornati.</span><span class="sxs-lookup"><span data-stu-id="2acd3-187">For example, not taking into account the status of a cached source file when renewing a [sliding expiration](xref:Microsoft.Extensions.Caching.Memory.MemoryCacheEntryOptions.SlidingExpiration) period leads to stale cached file data.</span></span> <span data-ttu-id="2acd3-188">Ogni richiesta di dati rinnoverà il periodo di scadenza variabile, ma il file non verrà mai ricaricato nella cache.</span><span class="sxs-lookup"><span data-stu-id="2acd3-188">Each request for the data renews the sliding expiration period, but the file is never reloaded into the cache.</span></span> <span data-ttu-id="2acd3-189">Le funzionalità dell'app che usano il contenuto del file memorizzato nella cache sono soggette alla possibile ricezione di contenuto non aggiornato.</span><span class="sxs-lookup"><span data-stu-id="2acd3-189">Any app features that use the file's cached content are subject to possibly receiving stale content.</span></span>

<span data-ttu-id="2acd3-190">L'uso dei token di modifica in uno scenario di memorizzazione di file nella cache consente di evitare la presenza di contenuto dei file non aggiornato nella cache.</span><span class="sxs-lookup"><span data-stu-id="2acd3-190">Using change tokens in a file caching scenario prevents the presence of stale file content in the cache.</span></span> <span data-ttu-id="2acd3-191">L'app di esempio illustra un'implementazione dell'approccio.</span><span class="sxs-lookup"><span data-stu-id="2acd3-191">The sample app demonstrates an implementation of the approach.</span></span>

<span data-ttu-id="2acd3-192">Nell'esempio viene usato `GetFileContent` per:</span><span class="sxs-lookup"><span data-stu-id="2acd3-192">The sample uses `GetFileContent` to:</span></span>

* <span data-ttu-id="2acd3-193">Restituire il contenuto del file.</span><span class="sxs-lookup"><span data-stu-id="2acd3-193">Return file content.</span></span>
* <span data-ttu-id="2acd3-194">Implementare un algoritmo di nuovi tentativi con interruzione temporanea esponenziale per i casi in cui un blocco di file impedisca temporaneamente la lettura di un file.</span><span class="sxs-lookup"><span data-stu-id="2acd3-194">Implement a retry algorithm with exponential back-off to cover cases where a file lock temporarily prevents reading a file.</span></span>

<span data-ttu-id="2acd3-195">*Utilities/Utilities.cs*:</span><span class="sxs-lookup"><span data-stu-id="2acd3-195">*Utilities/Utilities.cs*:</span></span>

[!code-csharp[](change-tokens/samples/3.x/SampleApp/Utilities/Utilities.cs?name=snippet2)]

<span data-ttu-id="2acd3-196">Viene creato un elemento `FileService` per gestire le ricerche nel file memorizzato nella cache.</span><span class="sxs-lookup"><span data-stu-id="2acd3-196">A `FileService` is created to handle cached file lookups.</span></span> <span data-ttu-id="2acd3-197">La chiamata del servizio al metodo `GetFileContent` prova a ottenere il contenuto del file dalla cache in memoria e a restituirlo al chiamante (*Services/FileService.cs*).</span><span class="sxs-lookup"><span data-stu-id="2acd3-197">The `GetFileContent` method call of the service attempts to obtain file content from the in-memory cache and return it to the caller (*Services/FileService.cs*).</span></span>

<span data-ttu-id="2acd3-198">Se non è possibile reperire il contenuto memorizzato nella cache usando la chiave della cache, verranno intraprese le azioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="2acd3-198">If cached content isn't found using the cache key, the following actions are taken:</span></span>

1. <span data-ttu-id="2acd3-199">Il contenuto del file viene ottenuto usando `GetFileContent`.</span><span class="sxs-lookup"><span data-stu-id="2acd3-199">The file content is obtained using `GetFileContent`.</span></span>
1. <span data-ttu-id="2acd3-200">Un token di modifica viene ottenuto dal provider di file con [IFileProviders.Watch](xref:Microsoft.Extensions.FileProviders.IFileProvider.Watch*).</span><span class="sxs-lookup"><span data-stu-id="2acd3-200">A change token is obtained from the file provider with [IFileProviders.Watch](xref:Microsoft.Extensions.FileProviders.IFileProvider.Watch*).</span></span> <span data-ttu-id="2acd3-201">Il callback del token viene attivato alla modifica del file.</span><span class="sxs-lookup"><span data-stu-id="2acd3-201">The token's callback is triggered when the file is modified.</span></span>
1. <span data-ttu-id="2acd3-202">Il contenuto del file viene memorizzato nella cache con un periodo di [scadenza variabile](xref:Microsoft.Extensions.Caching.Memory.MemoryCacheEntryOptions.SlidingExpiration).</span><span class="sxs-lookup"><span data-stu-id="2acd3-202">The file content is cached with a [sliding expiration](xref:Microsoft.Extensions.Caching.Memory.MemoryCacheEntryOptions.SlidingExpiration) period.</span></span> <span data-ttu-id="2acd3-203">Al token di modifica viene associato [MemoryCacheEntryExtensions.AddExpirationToken](xref:Microsoft.Extensions.Caching.Memory.MemoryCacheEntryExtensions.AddExpirationToken*) per eliminare la voce della cache se il file viene modificato mentre è memorizzato nella cache.</span><span class="sxs-lookup"><span data-stu-id="2acd3-203">The change token is attached with [MemoryCacheEntryExtensions.AddExpirationToken](xref:Microsoft.Extensions.Caching.Memory.MemoryCacheEntryExtensions.AddExpirationToken*) to evict the cache entry if the file changes while it's cached.</span></span>

<span data-ttu-id="2acd3-204">Nell'esempio seguente i file vengono archiviati nella [radice del contenuto](xref:fundamentals/index#content-root)dell'app.</span><span class="sxs-lookup"><span data-stu-id="2acd3-204">In the following example, files are stored in the app's [content root](xref:fundamentals/index#content-root).</span></span> <span data-ttu-id="2acd3-205">`IWebHostEnvironment.ContentRootFileProvider` viene usato per ottenere un oggetto che <xref:Microsoft.Extensions.FileProviders.IFileProvider> punta all'oggetto dell'app `IWebHostEnvironment.ContentRootPath` .</span><span class="sxs-lookup"><span data-stu-id="2acd3-205">`IWebHostEnvironment.ContentRootFileProvider` is used to obtain an <xref:Microsoft.Extensions.FileProviders.IFileProvider> pointing at the app's `IWebHostEnvironment.ContentRootPath`.</span></span> <span data-ttu-id="2acd3-206">`filePath` viene ottenuto con [IFileInfo.PhysicalPath](xref:Microsoft.Extensions.FileProviders.IFileInfo.PhysicalPath).</span><span class="sxs-lookup"><span data-stu-id="2acd3-206">The `filePath` is obtained with [IFileInfo.PhysicalPath](xref:Microsoft.Extensions.FileProviders.IFileInfo.PhysicalPath).</span></span>

[!code-csharp[](change-tokens/samples/3.x/SampleApp/Services/FileService.cs?name=snippet1)]

<span data-ttu-id="2acd3-207">`FileService` è registrato nel contenitore di servizi insieme al servizio di memorizzazione nella cache.</span><span class="sxs-lookup"><span data-stu-id="2acd3-207">The `FileService` is registered in the service container along with the memory caching service.</span></span>

<span data-ttu-id="2acd3-208">In `Startup.ConfigureServices`:</span><span class="sxs-lookup"><span data-stu-id="2acd3-208">In `Startup.ConfigureServices`:</span></span>

[!code-csharp[](change-tokens/samples/3.x/SampleApp/Startup.cs?name=snippet4)]

<span data-ttu-id="2acd3-209">Il modello di pagina carica il contenuto del file usando il servizio.</span><span class="sxs-lookup"><span data-stu-id="2acd3-209">The page model loads the file's content using the service.</span></span>

<span data-ttu-id="2acd3-210">Nel metodo `OnGet` della pagina Index (*Pages/Index.cshtml.cs*):</span><span class="sxs-lookup"><span data-stu-id="2acd3-210">In the Index page's `OnGet` method (*Pages/Index.cshtml.cs*):</span></span>

[!code-csharp[](change-tokens/samples/3.x/SampleApp/Pages/Index.cshtml.cs?name=snippet3)]

## <a name="compositechangetoken-class"></a><span data-ttu-id="2acd3-211">Classe CompositeChangeToken</span><span class="sxs-lookup"><span data-stu-id="2acd3-211">CompositeChangeToken class</span></span>

<span data-ttu-id="2acd3-212">Per rappresentare una o più istanze di `IChangeToken` in un singolo oggetto, usare la classe <xref:Microsoft.Extensions.Primitives.CompositeChangeToken>.</span><span class="sxs-lookup"><span data-stu-id="2acd3-212">For representing one or more `IChangeToken` instances in a single object, use the <xref:Microsoft.Extensions.Primitives.CompositeChangeToken> class.</span></span>

```csharp
var firstCancellationTokenSource = new CancellationTokenSource();
var secondCancellationTokenSource = new CancellationTokenSource();

var firstCancellationToken = firstCancellationTokenSource.Token;
var secondCancellationToken = secondCancellationTokenSource.Token;

var firstCancellationChangeToken = new CancellationChangeToken(firstCancellationToken);
var secondCancellationChangeToken = new CancellationChangeToken(secondCancellationToken);

var compositeChangeToken = 
    new CompositeChangeToken(
        new List<IChangeToken> 
        {
            firstCancellationChangeToken, 
            secondCancellationChangeToken
        });
```

<span data-ttu-id="2acd3-213">`HasChanged` nel token composito indica `true` se l'elemento `HasChanged` di qualsiasi token rappresentato è `true`.</span><span class="sxs-lookup"><span data-stu-id="2acd3-213">`HasChanged` on the composite token reports `true` if any represented token `HasChanged` is `true`.</span></span> <span data-ttu-id="2acd3-214">`ActiveChangeCallbacks` nel token composito indica `true` se l'elemento `ActiveChangeCallbacks` di qualsiasi token rappresentato è `true`.</span><span class="sxs-lookup"><span data-stu-id="2acd3-214">`ActiveChangeCallbacks` on the composite token reports `true` if any represented token `ActiveChangeCallbacks` is `true`.</span></span> <span data-ttu-id="2acd3-215">Se si verificano più eventi di modifica simultanei, il callback della modifica composita viene richiamato una volta.</span><span class="sxs-lookup"><span data-stu-id="2acd3-215">If multiple concurrent change events occur, the composite change callback is invoked one time.</span></span>

::: moniker-end

::: moniker range="< aspnetcore-3.0"

<span data-ttu-id="2acd3-216">Un *token di modifica* è un blocco predefinito di uso generico e di basso livello che viene usato per il rilevamento delle modifiche di stato.</span><span class="sxs-lookup"><span data-stu-id="2acd3-216">A *change token* is a general-purpose, low-level building block used to track state changes.</span></span>

<span data-ttu-id="2acd3-217">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/change-tokens/samples/) ([procedura per il download](xref:index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="2acd3-217">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/change-tokens/samples/) ([how to download](xref:index#how-to-download-a-sample))</span></span>

## <a name="ichangetoken-interface"></a><span data-ttu-id="2acd3-218">Interfaccia IChangeToken</span><span class="sxs-lookup"><span data-stu-id="2acd3-218">IChangeToken interface</span></span>

<span data-ttu-id="2acd3-219"><xref:Microsoft.Extensions.Primitives.IChangeToken> propaga le notifiche relative a una modifica apportata.</span><span class="sxs-lookup"><span data-stu-id="2acd3-219"><xref:Microsoft.Extensions.Primitives.IChangeToken> propagates notifications that a change has occurred.</span></span> <span data-ttu-id="2acd3-220">`IChangeToken` risiede nello spazio dei nomi <xref:Microsoft.Extensions.Primitives?displayProperty=fullName>.</span><span class="sxs-lookup"><span data-stu-id="2acd3-220">`IChangeToken` resides in the <xref:Microsoft.Extensions.Primitives?displayProperty=fullName> namespace.</span></span> <span data-ttu-id="2acd3-221">Per le app che non usano il [metapacchetto Microsoft.AspNetCore.App](xref:fundamentals/metapackage-app), creare un riferimento al pacchetto per il pacchetto NuGet [Microsoft.Extensions.Primitives](https://www.nuget.org/packages/Microsoft.Extensions.Primitives/).</span><span class="sxs-lookup"><span data-stu-id="2acd3-221">For apps that don't use the [Microsoft.AspNetCore.App metapackage](xref:fundamentals/metapackage-app), create a package reference for the [Microsoft.Extensions.Primitives](https://www.nuget.org/packages/Microsoft.Extensions.Primitives/) NuGet package.</span></span>

<span data-ttu-id="2acd3-222">`IChangeToken` ha due proprietà:</span><span class="sxs-lookup"><span data-stu-id="2acd3-222">`IChangeToken` has two properties:</span></span>

* <span data-ttu-id="2acd3-223"><xref:Microsoft.Extensions.Primitives.IChangeToken.ActiveChangeCallbacks> indica se il token genera callback in modo proattivo.</span><span class="sxs-lookup"><span data-stu-id="2acd3-223"><xref:Microsoft.Extensions.Primitives.IChangeToken.ActiveChangeCallbacks> indicate if the token proactively raises callbacks.</span></span> <span data-ttu-id="2acd3-224">Se `ActiveChangedCallbacks` è impostato su `false`, non viene mai chiamato alcun callback e l'app deve eseguire il polling di `HasChanged` per ottenere le modifiche.</span><span class="sxs-lookup"><span data-stu-id="2acd3-224">If `ActiveChangedCallbacks` is set to `false`, a callback is never called, and the app must poll `HasChanged` for changes.</span></span> <span data-ttu-id="2acd3-225">È anche possibile che un token non venga mai annullato se non vengono apportate modifiche o se il listener di modifica sottostante viene eliminato o disabilitato.</span><span class="sxs-lookup"><span data-stu-id="2acd3-225">It's also possible for a token to never be cancelled if no changes occur or the underlying change listener is disposed or disabled.</span></span>
* <span data-ttu-id="2acd3-226"><xref:Microsoft.Extensions.Primitives.IChangeToken.HasChanged> riceve un valore che indica se è stata apportata una modifica.</span><span class="sxs-lookup"><span data-stu-id="2acd3-226"><xref:Microsoft.Extensions.Primitives.IChangeToken.HasChanged> receives a value that indicates if a change has occurred.</span></span>

<span data-ttu-id="2acd3-227">L' `IChangeToken` interfaccia include il metodo [RegisterChangeCallback (Action \<Object> , Object)](xref:Microsoft.Extensions.Primitives.IChangeToken.RegisterChangeCallback*) , che registra un callback che viene richiamato quando il token viene modificato.</span><span class="sxs-lookup"><span data-stu-id="2acd3-227">The `IChangeToken` interface includes the [RegisterChangeCallback(Action\<Object>, Object)](xref:Microsoft.Extensions.Primitives.IChangeToken.RegisterChangeCallback*) method, which registers a callback that's invoked when the token has changed.</span></span> <span data-ttu-id="2acd3-228">Prima che il callback venga richiamato è necessario impostare `HasChanged`.</span><span class="sxs-lookup"><span data-stu-id="2acd3-228">`HasChanged` must be set before the callback is invoked.</span></span>

## <a name="changetoken-class"></a><span data-ttu-id="2acd3-229">Classe ChangeToken</span><span class="sxs-lookup"><span data-stu-id="2acd3-229">ChangeToken class</span></span>

<span data-ttu-id="2acd3-230"><xref:Microsoft.Extensions.Primitives.ChangeToken> è una classe statica usata per propagare le notifiche relative a una modifica che è stata apportata.</span><span class="sxs-lookup"><span data-stu-id="2acd3-230"><xref:Microsoft.Extensions.Primitives.ChangeToken> is a static class used to propagate notifications that a change has occurred.</span></span> <span data-ttu-id="2acd3-231">`ChangeToken` risiede nello spazio dei nomi <xref:Microsoft.Extensions.Primitives?displayProperty=fullName>.</span><span class="sxs-lookup"><span data-stu-id="2acd3-231">`ChangeToken` resides in the <xref:Microsoft.Extensions.Primitives?displayProperty=fullName> namespace.</span></span> <span data-ttu-id="2acd3-232">Per le app che non usano il [metapacchetto Microsoft.AspNetCore.App](xref:fundamentals/metapackage-app), creare un riferimento al pacchetto per il pacchetto NuGet [Microsoft.Extensions.Primitives](https://www.nuget.org/packages/Microsoft.Extensions.Primitives/).</span><span class="sxs-lookup"><span data-stu-id="2acd3-232">For apps that don't use the [Microsoft.AspNetCore.App metapackage](xref:fundamentals/metapackage-app), create a package reference for the [Microsoft.Extensions.Primitives](https://www.nuget.org/packages/Microsoft.Extensions.Primitives/) NuGet package.</span></span>

<span data-ttu-id="2acd3-233">Il metodo [token. OnChange (Func \<IChangeToken> , Action)](xref:Microsoft.Extensions.Primitives.ChangeToken.OnChange*) registra un oggetto `Action` da chiamare ogni volta che il token cambia:</span><span class="sxs-lookup"><span data-stu-id="2acd3-233">The [ChangeToken.OnChange(Func\<IChangeToken>, Action)](xref:Microsoft.Extensions.Primitives.ChangeToken.OnChange*) method registers an `Action` to call whenever the token changes:</span></span>

* <span data-ttu-id="2acd3-234">`Func<IChangeToken>` produce il token.</span><span class="sxs-lookup"><span data-stu-id="2acd3-234">`Func<IChangeToken>` produces the token.</span></span>
* <span data-ttu-id="2acd3-235">`Action` viene chiamato alla modifica del token.</span><span class="sxs-lookup"><span data-stu-id="2acd3-235">`Action` is called when the token changes.</span></span>

<span data-ttu-id="2acd3-236">L'overload di [token. OnChange \<TState> (Func \<IChangeToken> , Action \<TState> , tstate)](xref:Microsoft.Extensions.Primitives.ChangeToken.OnChange*) accetta un `TState` parametro aggiuntivo passato al consumer del token `Action` .</span><span class="sxs-lookup"><span data-stu-id="2acd3-236">The [ChangeToken.OnChange\<TState>(Func\<IChangeToken>, Action\<TState>, TState)](xref:Microsoft.Extensions.Primitives.ChangeToken.OnChange*) overload takes an additional `TState` parameter that's passed into the token consumer `Action`.</span></span>

<span data-ttu-id="2acd3-237">`OnChange` restituisce un oggetto <xref:System.IDisposable>.</span><span class="sxs-lookup"><span data-stu-id="2acd3-237">`OnChange` returns an <xref:System.IDisposable>.</span></span> <span data-ttu-id="2acd3-238">La chiamata a <xref:System.IDisposable.Dispose*> interrompe l'ascolto di altre modifiche da parte del token e rilascia le risorse del token.</span><span class="sxs-lookup"><span data-stu-id="2acd3-238">Calling <xref:System.IDisposable.Dispose*> stops the token from listening for further changes and releases the token's resources.</span></span>

## <a name="example-uses-of-change-tokens-in-aspnet-core"></a><span data-ttu-id="2acd3-239">Esempi di uso di token di modifica in ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="2acd3-239">Example uses of change tokens in ASP.NET Core</span></span>

<span data-ttu-id="2acd3-240">I token di modifica vengono usati nelle aree principali di ASP.NET Core per monitorare le modifiche apportate agli oggetti:</span><span class="sxs-lookup"><span data-stu-id="2acd3-240">Change tokens are used in prominent areas of ASP.NET Core to monitor for changes to objects:</span></span>

* <span data-ttu-id="2acd3-241">Per monitorare le modifiche ai file, il metodo <xref:Microsoft.Extensions.FileProviders.IFileProvider.Watch*> di <xref:Microsoft.Extensions.FileProviders.IFileProvider> crea un elemento `IChangeToken` per le cartelle o i file specificati da controllare.</span><span class="sxs-lookup"><span data-stu-id="2acd3-241">For monitoring changes to files, <xref:Microsoft.Extensions.FileProviders.IFileProvider>'s <xref:Microsoft.Extensions.FileProviders.IFileProvider.Watch*> method creates an `IChangeToken` for the specified files or folder to watch.</span></span>
* <span data-ttu-id="2acd3-242">È possibile aggiungere token `IChangeToken` alle voci della cache per attivare le eliminazioni dalla cache alla modifica.</span><span class="sxs-lookup"><span data-stu-id="2acd3-242">`IChangeToken` tokens can be added to cache entries to trigger cache evictions on change.</span></span>
* <span data-ttu-id="2acd3-243">Per le modifiche di `TOptions`, l'implementazione <xref:Microsoft.Extensions.Options.OptionsMonitor`1> predefinita di <xref:Microsoft.Extensions.Options.IOptionsMonitor`1> ha un overload che accetta una o più istanze di <xref:Microsoft.Extensions.Options.IOptionsChangeTokenSource`1>.</span><span class="sxs-lookup"><span data-stu-id="2acd3-243">For `TOptions` changes, the default <xref:Microsoft.Extensions.Options.OptionsMonitor`1> implementation of <xref:Microsoft.Extensions.Options.IOptionsMonitor`1> has an overload that accepts one or more <xref:Microsoft.Extensions.Options.IOptionsChangeTokenSource`1> instances.</span></span> <span data-ttu-id="2acd3-244">Ogni istanza restituisce un elemento `IChangeToken` per registrare un callback di notifica delle modifiche per il rilevamento delle modifiche apportate alle opzioni.</span><span class="sxs-lookup"><span data-stu-id="2acd3-244">Each instance returns an `IChangeToken` to register a change notification callback for tracking options changes.</span></span>

## <a name="monitor-for-configuration-changes"></a><span data-ttu-id="2acd3-245">Monitorare le modifiche di configurazione</span><span class="sxs-lookup"><span data-stu-id="2acd3-245">Monitor for configuration changes</span></span>

<span data-ttu-id="2acd3-246">Per impostazione predefinita, i modelli di ASP.NET Core usano [file di configurazione JSON](xref:fundamentals/configuration/index#json-configuration-provider) ( *appsettings.json* , *appsettings.Development.js* e *appsettings.Production.json*) per caricare le impostazioni di configurazione dell'app.</span><span class="sxs-lookup"><span data-stu-id="2acd3-246">By default, ASP.NET Core templates use [JSON configuration files](xref:fundamentals/configuration/index#json-configuration-provider) (*appsettings.json*, *appsettings.Development.json*, and *appsettings.Production.json*) to load app configuration settings.</span></span>

<span data-ttu-id="2acd3-247">Questi file vengono configurati usando il metodo di estensione [AddJsonFile(IConfigurationBuilder, String, Boolean, Boolean)](xref:Microsoft.Extensions.Configuration.JsonConfigurationExtensions.AddJsonFile*) per <xref:Microsoft.Extensions.Configuration.ConfigurationBuilder> che accetta un parametro `reloadOnChange`.</span><span class="sxs-lookup"><span data-stu-id="2acd3-247">These files are configured using the [AddJsonFile(IConfigurationBuilder, String, Boolean, Boolean)](xref:Microsoft.Extensions.Configuration.JsonConfigurationExtensions.AddJsonFile*) extension method on <xref:Microsoft.Extensions.Configuration.ConfigurationBuilder> that accepts a `reloadOnChange` parameter.</span></span> <span data-ttu-id="2acd3-248">`reloadOnChange` indica se la configurazione deve essere ricaricata alla modifica del file.</span><span class="sxs-lookup"><span data-stu-id="2acd3-248">`reloadOnChange` indicates if configuration should be reloaded on file changes.</span></span> <span data-ttu-id="2acd3-249">Questa impostazione viene visualizzata nel metodo pratico <xref:Microsoft.AspNetCore.WebHost><xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder*>:</span><span class="sxs-lookup"><span data-stu-id="2acd3-249">This setting appears in the <xref:Microsoft.AspNetCore.WebHost> convenience method <xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder*>:</span></span>

```csharp
config.AddJsonFile("appsettings.json", optional: true, reloadOnChange: true)
      .AddJsonFile($"appsettings.{env.EnvironmentName}.json", optional: true, 
          reloadOnChange: true);
```

<span data-ttu-id="2acd3-250">La configurazione basata su file è rappresentata da <xref:Microsoft.Extensions.Configuration.FileConfigurationSource>.</span><span class="sxs-lookup"><span data-stu-id="2acd3-250">File-based configuration is represented by <xref:Microsoft.Extensions.Configuration.FileConfigurationSource>.</span></span> <span data-ttu-id="2acd3-251">`FileConfigurationSource` usa <xref:Microsoft.Extensions.FileProviders.IFileProvider> per monitorare i file.</span><span class="sxs-lookup"><span data-stu-id="2acd3-251">`FileConfigurationSource` uses <xref:Microsoft.Extensions.FileProviders.IFileProvider> to monitor files.</span></span>

<span data-ttu-id="2acd3-252">Per impostazione predefinita, `IFileMonitor` viene offerto da una classe <xref:Microsoft.Extensions.FileProviders.PhysicalFileProvider> che usa <xref:System.IO.FileSystemWatcher> per monitorare le modifiche apportate al file di configurazione.</span><span class="sxs-lookup"><span data-stu-id="2acd3-252">By default, the `IFileMonitor` is provided by a <xref:Microsoft.Extensions.FileProviders.PhysicalFileProvider>, which uses <xref:System.IO.FileSystemWatcher> to monitor for configuration file changes.</span></span>

<span data-ttu-id="2acd3-253">L'app di esempio illustra due implementazioni per il monitoraggio delle modifiche alla configurazione.</span><span class="sxs-lookup"><span data-stu-id="2acd3-253">The sample app demonstrates two implementations for monitoring configuration changes.</span></span> <span data-ttu-id="2acd3-254">In caso di modifica di uno qualsiasi dei file appsettings&mdash;, entrambe le implementazioni di monitoraggio dei file eseguono codice personalizzato e l'app di esempio scrive un messaggio nella console.</span><span class="sxs-lookup"><span data-stu-id="2acd3-254">If any of the *appsettings* files change, both of the file monitoring implementations execute custom code&mdash;the sample app writes a message to the console.</span></span>

<span data-ttu-id="2acd3-255">L'elemento `FileSystemWatcher` di un file di configurazione può attivare più callback del token per una singola modifica del file di configurazione.</span><span class="sxs-lookup"><span data-stu-id="2acd3-255">A configuration file's `FileSystemWatcher` can trigger multiple token callbacks for a single configuration file change.</span></span> <span data-ttu-id="2acd3-256">Per assicurarsi che il codice personalizzato venga eseguito solo una volta quando vengono attivati più callback del token, l'implementazione dell'esempio controlla gli hash dei file.</span><span class="sxs-lookup"><span data-stu-id="2acd3-256">To ensure that the custom code is only run once when multiple token callbacks are triggered, the sample's implementation checks file hashes.</span></span> <span data-ttu-id="2acd3-257">L'esempio usa l'hash di file SHA1.</span><span class="sxs-lookup"><span data-stu-id="2acd3-257">The sample uses SHA1 file hashing.</span></span> <span data-ttu-id="2acd3-258">Viene implementato un nuovo tentativo con un'interruzione temporanea esponenziale.</span><span class="sxs-lookup"><span data-stu-id="2acd3-258">A retry is implemented with an exponential back-off.</span></span> <span data-ttu-id="2acd3-259">Il nuovo tentativo è presente perché potrebbe verificarsi un blocco di file che impedisce temporaneamente l'elaborazione di un nuovo hash in un file.</span><span class="sxs-lookup"><span data-stu-id="2acd3-259">The retry is present because file locking may occur that temporarily prevents computing a new hash on a file.</span></span>

<span data-ttu-id="2acd3-260">*Utilities/Utilities.cs*:</span><span class="sxs-lookup"><span data-stu-id="2acd3-260">*Utilities/Utilities.cs*:</span></span>

[!code-csharp[](change-tokens/samples/2.x/SampleApp/Utilities/Utilities.cs?name=snippet1)]

### <a name="simple-startup-change-token"></a><span data-ttu-id="2acd3-261">Semplice token di modifica dell'avvio</span><span class="sxs-lookup"><span data-stu-id="2acd3-261">Simple startup change token</span></span>

<span data-ttu-id="2acd3-262">Registrare un callback dell'elemento `Action` consumer del token per le notifiche di modifica al token di ricaricamento della configurazione.</span><span class="sxs-lookup"><span data-stu-id="2acd3-262">Register a token consumer `Action` callback for change notifications to the configuration reload token.</span></span>

<span data-ttu-id="2acd3-263">In `Startup.Configure`:</span><span class="sxs-lookup"><span data-stu-id="2acd3-263">In `Startup.Configure`:</span></span>

[!code-csharp[](change-tokens/samples/2.x/SampleApp/Startup.cs?name=snippet2)]

<span data-ttu-id="2acd3-264">`config.GetReloadToken()` fornisce il token.</span><span class="sxs-lookup"><span data-stu-id="2acd3-264">`config.GetReloadToken()` provides the token.</span></span> <span data-ttu-id="2acd3-265">Il callback è il metodo `InvokeChanged`:</span><span class="sxs-lookup"><span data-stu-id="2acd3-265">The callback is the `InvokeChanged` method:</span></span>

[!code-csharp[](change-tokens/samples/2.x/SampleApp/Startup.cs?name=snippet3)]

<span data-ttu-id="2acd3-266">Il valore `state` del callback viene usato per passare `IHostingEnvironment`, che è utile per specificare il file di configurazione *appsettings* corretto da monitorare (ad esempio, *appsettings.Development.json* nell'ambiente di sviluppo).</span><span class="sxs-lookup"><span data-stu-id="2acd3-266">The `state` of the callback is used to pass in the `IHostingEnvironment`, which is useful for specifying the correct *appsettings* configuration file to monitor (for example, *appsettings.Development.json* when in the Development environment).</span></span> <span data-ttu-id="2acd3-267">Gli hash dei file vengono usati per impedire che l'istruzione `WriteConsole` venga eseguita più volte a causa di più callback del token quando il file di configurazione è stato modificato una sola volta.</span><span class="sxs-lookup"><span data-stu-id="2acd3-267">File hashes are used to prevent the `WriteConsole` statement from running multiple times due to multiple token callbacks when the configuration file has only changed once.</span></span>

<span data-ttu-id="2acd3-268">Questo sistema rimane in esecuzione finché l'app è in esecuzione e non può essere disabilitato dall'utente.</span><span class="sxs-lookup"><span data-stu-id="2acd3-268">This system runs as long as the app is running and can't be disabled by the user.</span></span>

### <a name="monitor-configuration-changes-as-a-service"></a><span data-ttu-id="2acd3-269">Monitorare le modifiche di configurazione come servizio</span><span class="sxs-lookup"><span data-stu-id="2acd3-269">Monitor configuration changes as a service</span></span>

<span data-ttu-id="2acd3-270">L'esempio implementa:</span><span class="sxs-lookup"><span data-stu-id="2acd3-270">The sample implements:</span></span>

* <span data-ttu-id="2acd3-271">Monitoraggio di base del token di avvio.</span><span class="sxs-lookup"><span data-stu-id="2acd3-271">Basic startup token monitoring.</span></span>
* <span data-ttu-id="2acd3-272">Monitoraggio come servizio.</span><span class="sxs-lookup"><span data-stu-id="2acd3-272">Monitoring as a service.</span></span>
* <span data-ttu-id="2acd3-273">Un meccanismo per abilitare e disabilitare il monitoraggio.</span><span class="sxs-lookup"><span data-stu-id="2acd3-273">A mechanism to enable and disable monitoring.</span></span>

<span data-ttu-id="2acd3-274">L'esempio stabilisce un'interfaccia `IConfigurationMonitor`.</span><span class="sxs-lookup"><span data-stu-id="2acd3-274">The sample establishes an `IConfigurationMonitor` interface.</span></span>

<span data-ttu-id="2acd3-275">*Extensions/ConfigurationMonitor.cs*:</span><span class="sxs-lookup"><span data-stu-id="2acd3-275">*Extensions/ConfigurationMonitor.cs*:</span></span>

[!code-csharp[](change-tokens/samples/2.x/SampleApp/Extensions/ConfigurationMonitor.cs?name=snippet1)]

<span data-ttu-id="2acd3-276">Il costruttore della classe implementata, `ConfigurationMonitor`, registra un callback per le notifiche di modifica:</span><span class="sxs-lookup"><span data-stu-id="2acd3-276">The constructor of the implemented class, `ConfigurationMonitor`, registers a callback for change notifications:</span></span>

[!code-csharp[](change-tokens/samples/2.x/SampleApp/Extensions/ConfigurationMonitor.cs?name=snippet2)]

<span data-ttu-id="2acd3-277">`config.GetReloadToken()` fornisce il token.</span><span class="sxs-lookup"><span data-stu-id="2acd3-277">`config.GetReloadToken()` supplies the token.</span></span> <span data-ttu-id="2acd3-278">`InvokeChanged` è il metodo di callback.</span><span class="sxs-lookup"><span data-stu-id="2acd3-278">`InvokeChanged` is the callback method.</span></span> <span data-ttu-id="2acd3-279">Il valore `state` in questa istanza è un riferimento all'istanza `IConfigurationMonitor` usata per accedere allo stato di monitoraggio.</span><span class="sxs-lookup"><span data-stu-id="2acd3-279">The `state` in this instance is a reference to the `IConfigurationMonitor` instance that's used to access the monitoring state.</span></span> <span data-ttu-id="2acd3-280">Vengono usate due proprietà:</span><span class="sxs-lookup"><span data-stu-id="2acd3-280">Two properties are used:</span></span>

* <span data-ttu-id="2acd3-281">`MonitoringEnabled`: Indica se il callback deve eseguire il codice personalizzato.</span><span class="sxs-lookup"><span data-stu-id="2acd3-281">`MonitoringEnabled`: Indicates if the callback should run its custom code.</span></span>
* <span data-ttu-id="2acd3-282">`CurrentState`: Descrive lo stato di monitoraggio corrente per l'uso nell'interfaccia utente.</span><span class="sxs-lookup"><span data-stu-id="2acd3-282">`CurrentState`: Describes the current monitoring state for use in the UI.</span></span>

<span data-ttu-id="2acd3-283">Il metodo `InvokeChanged` è simile all'approccio precedente, ad eccezione del fatto che:</span><span class="sxs-lookup"><span data-stu-id="2acd3-283">The `InvokeChanged` method is similar to the earlier approach, except that it:</span></span>

* <span data-ttu-id="2acd3-284">Non esegue il proprio codice a meno che `MonitoringEnabled` non sia impostato su `true`.</span><span class="sxs-lookup"><span data-stu-id="2acd3-284">Doesn't run its code unless `MonitoringEnabled` is `true`.</span></span>
* <span data-ttu-id="2acd3-285">Restituisce l'elemento `state` corrente nell'output `WriteConsole`.</span><span class="sxs-lookup"><span data-stu-id="2acd3-285">Outputs the current `state` in its `WriteConsole` output.</span></span>

[!code-csharp[](change-tokens/samples/2.x/SampleApp/Extensions/ConfigurationMonitor.cs?name=snippet3)]

<span data-ttu-id="2acd3-286">Un'istanza di `ConfigurationMonitor` viene registrata come servizio in `Startup.ConfigureServices`:</span><span class="sxs-lookup"><span data-stu-id="2acd3-286">An instance `ConfigurationMonitor` is registered as a service in `Startup.ConfigureServices`:</span></span>

[!code-csharp[](change-tokens/samples/2.x/SampleApp/Startup.cs?name=snippet1)]

<span data-ttu-id="2acd3-287">La pagina di indice offre all'utente il controllo sul monitoraggio della configurazione.</span><span class="sxs-lookup"><span data-stu-id="2acd3-287">The Index page offers the user control over configuration monitoring.</span></span> <span data-ttu-id="2acd3-288">L'istanza di `IConfigurationMonitor` viene inserita in `IndexModel`.</span><span class="sxs-lookup"><span data-stu-id="2acd3-288">The instance of `IConfigurationMonitor` is injected into the `IndexModel`.</span></span>

<span data-ttu-id="2acd3-289">*Pages/Index.cshtml.cs*:</span><span class="sxs-lookup"><span data-stu-id="2acd3-289">*Pages/Index.cshtml.cs*:</span></span>

[!code-csharp[](change-tokens/samples/2.x/SampleApp/Pages/Index.cshtml.cs?name=snippet1)]

<span data-ttu-id="2acd3-290">Il monitoraggio di configurazione (`_monitor`) viene usato per abilitare o disabilitare il monitoraggio e impostare lo stato corrente per commenti e suggerimenti dell'interfaccia utente:</span><span class="sxs-lookup"><span data-stu-id="2acd3-290">The configuration monitor (`_monitor`) is used to enable or disable monitoring and set the current state for UI feedback:</span></span>

[!code-csharp[](change-tokens/samples/2.x/SampleApp/Pages/Index.cshtml.cs?name=snippet2)]

<span data-ttu-id="2acd3-291">Quando viene attivato `OnPostStartMonitoring`, il monitoraggio è abilitato e lo stato corrente viene cancellato.</span><span class="sxs-lookup"><span data-stu-id="2acd3-291">When `OnPostStartMonitoring` is triggered, monitoring is enabled, and the current state is cleared.</span></span> <span data-ttu-id="2acd3-292">Quando viene attivato `OnPostStopMonitoring`, il monitoraggio è disabilitato e lo stato viene impostato in modo da indicare che il monitoraggio non viene eseguito.</span><span class="sxs-lookup"><span data-stu-id="2acd3-292">When `OnPostStopMonitoring` is triggered, monitoring is disabled, and the state is set to reflect that monitoring isn't occurring.</span></span>

<span data-ttu-id="2acd3-293">I pulsanti nell'interfaccia utente abilitano e disabilitano il monitoraggio.</span><span class="sxs-lookup"><span data-stu-id="2acd3-293">Buttons in the UI enable and disable monitoring.</span></span>

<span data-ttu-id="2acd3-294">*Pages/index. cshtml*:</span><span class="sxs-lookup"><span data-stu-id="2acd3-294">*Pages/Index.cshtml*:</span></span>

[!code-cshtml[](change-tokens/samples/2.x/SampleApp/Pages/Index.cshtml?name=snippet_Buttons)]

## <a name="monitor-cached-file-changes"></a><span data-ttu-id="2acd3-295">Monitorare le modifiche al file nella cache</span><span class="sxs-lookup"><span data-stu-id="2acd3-295">Monitor cached file changes</span></span>

<span data-ttu-id="2acd3-296">È possibile memorizzare il contenuto del file nella cache in memoria usando <xref:Microsoft.Extensions.Caching.Memory.IMemoryCache>.</span><span class="sxs-lookup"><span data-stu-id="2acd3-296">File content can be cached in-memory using <xref:Microsoft.Extensions.Caching.Memory.IMemoryCache>.</span></span> <span data-ttu-id="2acd3-297">La memorizzazione nella cache in memoria è descritta nell'argomento [Cache in memoria](xref:performance/caching/memory).</span><span class="sxs-lookup"><span data-stu-id="2acd3-297">In-memory caching is described in the [Cache in-memory](xref:performance/caching/memory) topic.</span></span> <span data-ttu-id="2acd3-298">Senza eseguire passaggi aggiuntivi, ad esempio l'implementazione descritta di seguito, la cache restituirà dati *non aggiornati* (obsoleti) se i dati di origine vengono modificati.</span><span class="sxs-lookup"><span data-stu-id="2acd3-298">Without taking additional steps, such as the implementation described below, *stale* (outdated) data is returned from a cache if the source data changes.</span></span>

<span data-ttu-id="2acd3-299">Ad esempio, se durante il rinnovo di un periodo di [scadenza variabile](xref:Microsoft.Extensions.Caching.Memory.MemoryCacheEntryOptions.SlidingExpiration) non si tiene conto dello stato di un file di origine memorizzato nella cache, i dati relativi ai file nella cache risulteranno non aggiornati.</span><span class="sxs-lookup"><span data-stu-id="2acd3-299">For example, not taking into account the status of a cached source file when renewing a [sliding expiration](xref:Microsoft.Extensions.Caching.Memory.MemoryCacheEntryOptions.SlidingExpiration) period leads to stale cached file data.</span></span> <span data-ttu-id="2acd3-300">Ogni richiesta di dati rinnoverà il periodo di scadenza variabile, ma il file non verrà mai ricaricato nella cache.</span><span class="sxs-lookup"><span data-stu-id="2acd3-300">Each request for the data renews the sliding expiration period, but the file is never reloaded into the cache.</span></span> <span data-ttu-id="2acd3-301">Le funzionalità dell'app che usano il contenuto del file memorizzato nella cache sono soggette alla possibile ricezione di contenuto non aggiornato.</span><span class="sxs-lookup"><span data-stu-id="2acd3-301">Any app features that use the file's cached content are subject to possibly receiving stale content.</span></span>

<span data-ttu-id="2acd3-302">L'uso dei token di modifica in uno scenario di memorizzazione di file nella cache consente di evitare la presenza di contenuto dei file non aggiornato nella cache.</span><span class="sxs-lookup"><span data-stu-id="2acd3-302">Using change tokens in a file caching scenario prevents the presence of stale file content in the cache.</span></span> <span data-ttu-id="2acd3-303">L'app di esempio illustra un'implementazione dell'approccio.</span><span class="sxs-lookup"><span data-stu-id="2acd3-303">The sample app demonstrates an implementation of the approach.</span></span>

<span data-ttu-id="2acd3-304">Nell'esempio viene usato `GetFileContent` per:</span><span class="sxs-lookup"><span data-stu-id="2acd3-304">The sample uses `GetFileContent` to:</span></span>

* <span data-ttu-id="2acd3-305">Restituire il contenuto del file.</span><span class="sxs-lookup"><span data-stu-id="2acd3-305">Return file content.</span></span>
* <span data-ttu-id="2acd3-306">Implementare un algoritmo di nuovi tentativi con interruzione temporanea esponenziale per i casi in cui un blocco di file impedisca temporaneamente la lettura di un file.</span><span class="sxs-lookup"><span data-stu-id="2acd3-306">Implement a retry algorithm with exponential back-off to cover cases where a file lock temporarily prevents reading a file.</span></span>

<span data-ttu-id="2acd3-307">*Utilities/Utilities.cs*:</span><span class="sxs-lookup"><span data-stu-id="2acd3-307">*Utilities/Utilities.cs*:</span></span>

[!code-csharp[](change-tokens/samples/2.x/SampleApp/Utilities/Utilities.cs?name=snippet2)]

<span data-ttu-id="2acd3-308">Viene creato un elemento `FileService` per gestire le ricerche nel file memorizzato nella cache.</span><span class="sxs-lookup"><span data-stu-id="2acd3-308">A `FileService` is created to handle cached file lookups.</span></span> <span data-ttu-id="2acd3-309">La chiamata del servizio al metodo `GetFileContent` prova a ottenere il contenuto del file dalla cache in memoria e a restituirlo al chiamante (*Services/FileService.cs*).</span><span class="sxs-lookup"><span data-stu-id="2acd3-309">The `GetFileContent` method call of the service attempts to obtain file content from the in-memory cache and return it to the caller (*Services/FileService.cs*).</span></span>

<span data-ttu-id="2acd3-310">Se non è possibile reperire il contenuto memorizzato nella cache usando la chiave della cache, verranno intraprese le azioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="2acd3-310">If cached content isn't found using the cache key, the following actions are taken:</span></span>

1. <span data-ttu-id="2acd3-311">Il contenuto del file viene ottenuto usando `GetFileContent`.</span><span class="sxs-lookup"><span data-stu-id="2acd3-311">The file content is obtained using `GetFileContent`.</span></span>
1. <span data-ttu-id="2acd3-312">Un token di modifica viene ottenuto dal provider di file con [IFileProviders.Watch](xref:Microsoft.Extensions.FileProviders.IFileProvider.Watch*).</span><span class="sxs-lookup"><span data-stu-id="2acd3-312">A change token is obtained from the file provider with [IFileProviders.Watch](xref:Microsoft.Extensions.FileProviders.IFileProvider.Watch*).</span></span> <span data-ttu-id="2acd3-313">Il callback del token viene attivato alla modifica del file.</span><span class="sxs-lookup"><span data-stu-id="2acd3-313">The token's callback is triggered when the file is modified.</span></span>
1. <span data-ttu-id="2acd3-314">Il contenuto del file viene memorizzato nella cache con un periodo di [scadenza variabile](xref:Microsoft.Extensions.Caching.Memory.MemoryCacheEntryOptions.SlidingExpiration).</span><span class="sxs-lookup"><span data-stu-id="2acd3-314">The file content is cached with a [sliding expiration](xref:Microsoft.Extensions.Caching.Memory.MemoryCacheEntryOptions.SlidingExpiration) period.</span></span> <span data-ttu-id="2acd3-315">Al token di modifica viene associato [MemoryCacheEntryExtensions.AddExpirationToken](xref:Microsoft.Extensions.Caching.Memory.MemoryCacheEntryExtensions.AddExpirationToken*) per eliminare la voce della cache se il file viene modificato mentre è memorizzato nella cache.</span><span class="sxs-lookup"><span data-stu-id="2acd3-315">The change token is attached with [MemoryCacheEntryExtensions.AddExpirationToken](xref:Microsoft.Extensions.Caching.Memory.MemoryCacheEntryExtensions.AddExpirationToken*) to evict the cache entry if the file changes while it's cached.</span></span>

<span data-ttu-id="2acd3-316">Nell'esempio seguente i file vengono archiviati nella [radice del contenuto](xref:fundamentals/index#content-root)dell'app.</span><span class="sxs-lookup"><span data-stu-id="2acd3-316">In the following example, files are stored in the app's [content root](xref:fundamentals/index#content-root).</span></span> <span data-ttu-id="2acd3-317">[IHostingEnvironment.ContentRootFileProvider](xref:Microsoft.AspNetCore.Hosting.IHostingEnvironment.ContentRootFileProvider) viene usato per ottenere un <xref:Microsoft.Extensions.FileProviders.IFileProvider> che punta al <xref:Microsoft.AspNetCore.Hosting.IHostingEnvironment.ContentRootPath> dell'app.</span><span class="sxs-lookup"><span data-stu-id="2acd3-317">[IHostingEnvironment.ContentRootFileProvider](xref:Microsoft.AspNetCore.Hosting.IHostingEnvironment.ContentRootFileProvider) is used to obtain an <xref:Microsoft.Extensions.FileProviders.IFileProvider> pointing at the app's <xref:Microsoft.AspNetCore.Hosting.IHostingEnvironment.ContentRootPath>.</span></span> <span data-ttu-id="2acd3-318">`filePath` viene ottenuto con [IFileInfo.PhysicalPath](xref:Microsoft.Extensions.FileProviders.IFileInfo.PhysicalPath).</span><span class="sxs-lookup"><span data-stu-id="2acd3-318">The `filePath` is obtained with [IFileInfo.PhysicalPath](xref:Microsoft.Extensions.FileProviders.IFileInfo.PhysicalPath).</span></span>

[!code-csharp[](change-tokens/samples/2.x/SampleApp/Services/FileService.cs?name=snippet1)]

<span data-ttu-id="2acd3-319">`FileService` è registrato nel contenitore di servizi insieme al servizio di memorizzazione nella cache.</span><span class="sxs-lookup"><span data-stu-id="2acd3-319">The `FileService` is registered in the service container along with the memory caching service.</span></span>

<span data-ttu-id="2acd3-320">In `Startup.ConfigureServices`:</span><span class="sxs-lookup"><span data-stu-id="2acd3-320">In `Startup.ConfigureServices`:</span></span>

[!code-csharp[](change-tokens/samples/2.x/SampleApp/Startup.cs?name=snippet4)]

<span data-ttu-id="2acd3-321">Il modello di pagina carica il contenuto del file usando il servizio.</span><span class="sxs-lookup"><span data-stu-id="2acd3-321">The page model loads the file's content using the service.</span></span>

<span data-ttu-id="2acd3-322">Nel metodo `OnGet` della pagina Index (*Pages/Index.cshtml.cs*):</span><span class="sxs-lookup"><span data-stu-id="2acd3-322">In the Index page's `OnGet` method (*Pages/Index.cshtml.cs*):</span></span>

[!code-csharp[](change-tokens/samples/2.x/SampleApp/Pages/Index.cshtml.cs?name=snippet3)]

## <a name="compositechangetoken-class"></a><span data-ttu-id="2acd3-323">Classe CompositeChangeToken</span><span class="sxs-lookup"><span data-stu-id="2acd3-323">CompositeChangeToken class</span></span>

<span data-ttu-id="2acd3-324">Per rappresentare una o più istanze di `IChangeToken` in un singolo oggetto, usare la classe <xref:Microsoft.Extensions.Primitives.CompositeChangeToken>.</span><span class="sxs-lookup"><span data-stu-id="2acd3-324">For representing one or more `IChangeToken` instances in a single object, use the <xref:Microsoft.Extensions.Primitives.CompositeChangeToken> class.</span></span>

```csharp
var firstCancellationTokenSource = new CancellationTokenSource();
var secondCancellationTokenSource = new CancellationTokenSource();

var firstCancellationToken = firstCancellationTokenSource.Token;
var secondCancellationToken = secondCancellationTokenSource.Token;

var firstCancellationChangeToken = new CancellationChangeToken(firstCancellationToken);
var secondCancellationChangeToken = new CancellationChangeToken(secondCancellationToken);

var compositeChangeToken = 
    new CompositeChangeToken(
        new List<IChangeToken> 
        {
            firstCancellationChangeToken, 
            secondCancellationChangeToken
        });
```

<span data-ttu-id="2acd3-325">`HasChanged` nel token composito indica `true` se l'elemento `HasChanged` di qualsiasi token rappresentato è `true`.</span><span class="sxs-lookup"><span data-stu-id="2acd3-325">`HasChanged` on the composite token reports `true` if any represented token `HasChanged` is `true`.</span></span> <span data-ttu-id="2acd3-326">`ActiveChangeCallbacks` nel token composito indica `true` se l'elemento `ActiveChangeCallbacks` di qualsiasi token rappresentato è `true`.</span><span class="sxs-lookup"><span data-stu-id="2acd3-326">`ActiveChangeCallbacks` on the composite token reports `true` if any represented token `ActiveChangeCallbacks` is `true`.</span></span> <span data-ttu-id="2acd3-327">Se si verificano più eventi di modifica simultanei, il callback della modifica composita viene richiamato una volta.</span><span class="sxs-lookup"><span data-stu-id="2acd3-327">If multiple concurrent change events occur, the composite change callback is invoked one time.</span></span>

::: moniker-end

## <a name="additional-resources"></a><span data-ttu-id="2acd3-328">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="2acd3-328">Additional resources</span></span>

* <xref:performance/caching/memory>
* <xref:performance/caching/distributed>
* <xref:performance/caching/response>
* <xref:performance/caching/middleware>
* <xref:mvc/views/tag-helpers/builtin-th/cache-tag-helper>
* <xref:mvc/views/tag-helpers/builtin-th/distributed-cache-tag-helper>
