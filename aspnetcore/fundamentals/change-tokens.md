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
# <a name="detect-changes-with-change-tokens-in-aspnet-core"></a>Rilevare le modifiche apportate con i token di modifica in ASP.NET Core

::: moniker range=">= aspnetcore-3.0"

Un *token di modifica* è un blocco predefinito di uso generico e di basso livello che viene usato per il rilevamento delle modifiche di stato.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/change-tokens/samples/) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="ichangetoken-interface"></a>Interfaccia IChangeToken

<xref:Microsoft.Extensions.Primitives.IChangeToken> propaga le notifiche relative a una modifica apportata. `IChangeToken` risiede nello spazio dei nomi <xref:Microsoft.Extensions.Primitives?displayProperty=fullName>. Il pacchetto NuGet [Microsoft. Extensions. Primitives](https://www.nuget.org/packages/Microsoft.Extensions.Primitives/) viene fornito in modo implicito alle app ASP.NET Core.

`IChangeToken` ha due proprietà:

* <xref:Microsoft.Extensions.Primitives.IChangeToken.ActiveChangeCallbacks> indica se il token genera callback in modo proattivo. Se `ActiveChangedCallbacks` è impostato su `false`, non viene mai chiamato alcun callback e l'app deve eseguire il polling di `HasChanged` per ottenere le modifiche. È anche possibile che un token non venga mai annullato se non vengono apportate modifiche o se il listener di modifica sottostante viene eliminato o disabilitato.
* <xref:Microsoft.Extensions.Primitives.IChangeToken.HasChanged> riceve un valore che indica se è stata apportata una modifica.

L' `IChangeToken` interfaccia include il metodo [RegisterChangeCallback (Action \<Object> , Object)](xref:Microsoft.Extensions.Primitives.IChangeToken.RegisterChangeCallback*) , che registra un callback che viene richiamato quando il token viene modificato. Prima che il callback venga richiamato è necessario impostare `HasChanged`.

## <a name="changetoken-class"></a>Classe ChangeToken

<xref:Microsoft.Extensions.Primitives.ChangeToken> è una classe statica usata per propagare le notifiche relative a una modifica che è stata apportata. `ChangeToken` risiede nello spazio dei nomi <xref:Microsoft.Extensions.Primitives?displayProperty=fullName>. Il pacchetto NuGet [Microsoft. Extensions. Primitives](https://www.nuget.org/packages/Microsoft.Extensions.Primitives/) viene fornito in modo implicito alle app ASP.NET Core.

Il metodo [token. OnChange (Func \<IChangeToken> , Action)](xref:Microsoft.Extensions.Primitives.ChangeToken.OnChange*) registra un oggetto `Action` da chiamare ogni volta che il token cambia:

* `Func<IChangeToken>` produce il token.
* `Action` viene chiamato alla modifica del token.

L'overload di [token. OnChange \<TState> (Func \<IChangeToken> , Action \<TState> , tstate)](xref:Microsoft.Extensions.Primitives.ChangeToken.OnChange*) accetta un `TState` parametro aggiuntivo passato al consumer del token `Action` .

`OnChange` restituisce un oggetto <xref:System.IDisposable>. La chiamata a <xref:System.IDisposable.Dispose*> interrompe l'ascolto di altre modifiche da parte del token e rilascia le risorse del token.

## <a name="example-uses-of-change-tokens-in-aspnet-core"></a>Esempi di uso di token di modifica in ASP.NET Core

I token di modifica vengono usati nelle aree principali di ASP.NET Core per monitorare le modifiche apportate agli oggetti:

* Per monitorare le modifiche ai file, il metodo <xref:Microsoft.Extensions.FileProviders.IFileProvider.Watch*> di <xref:Microsoft.Extensions.FileProviders.IFileProvider> crea un elemento `IChangeToken` per le cartelle o i file specificati da controllare.
* È possibile aggiungere token `IChangeToken` alle voci della cache per attivare le eliminazioni dalla cache alla modifica.
* Per le modifiche di `TOptions`, l'implementazione <xref:Microsoft.Extensions.Options.OptionsMonitor`1> predefinita di <xref:Microsoft.Extensions.Options.IOptionsMonitor`1> ha un overload che accetta una o più istanze di <xref:Microsoft.Extensions.Options.IOptionsChangeTokenSource`1>. Ogni istanza restituisce un elemento `IChangeToken` per registrare un callback di notifica delle modifiche per il rilevamento delle modifiche apportate alle opzioni.

## <a name="monitor-for-configuration-changes"></a>Monitorare le modifiche di configurazione

Per impostazione predefinita, i modelli di ASP.NET Core usano [file di configurazione JSON](xref:fundamentals/configuration/index#json-configuration-provider) ( *appsettings.json* , *appsettings.Development.js* e *appsettings.Production.json*) per caricare le impostazioni di configurazione dell'app.

Questi file vengono configurati usando il metodo di estensione [AddJsonFile(IConfigurationBuilder, String, Boolean, Boolean)](xref:Microsoft.Extensions.Configuration.JsonConfigurationExtensions.AddJsonFile*) per <xref:Microsoft.Extensions.Configuration.ConfigurationBuilder> che accetta un parametro `reloadOnChange`. `reloadOnChange` indica se la configurazione deve essere ricaricata alla modifica del file. Questa impostazione viene visualizzata nel metodo pratico <xref:Microsoft.Extensions.Hosting.Host><xref:Microsoft.Extensions.Hosting.Host.CreateDefaultBuilder*>:

```csharp
config.AddJsonFile("appsettings.json", optional: true, reloadOnChange: true)
      .AddJsonFile($"appsettings.{env.EnvironmentName}.json", optional: true, 
          reloadOnChange: true);
```

La configurazione basata su file è rappresentata da <xref:Microsoft.Extensions.Configuration.FileConfigurationSource>. `FileConfigurationSource` usa <xref:Microsoft.Extensions.FileProviders.IFileProvider> per monitorare i file.

Per impostazione predefinita, `IFileMonitor` viene offerto da una classe <xref:Microsoft.Extensions.FileProviders.PhysicalFileProvider> che usa <xref:System.IO.FileSystemWatcher> per monitorare le modifiche apportate al file di configurazione.

L'app di esempio illustra due implementazioni per il monitoraggio delle modifiche alla configurazione. In caso di modifica di uno qualsiasi dei file appsettings&mdash;, entrambe le implementazioni di monitoraggio dei file eseguono codice personalizzato e l'app di esempio scrive un messaggio nella console.

L'elemento `FileSystemWatcher` di un file di configurazione può attivare più callback del token per una singola modifica del file di configurazione. Per assicurarsi che il codice personalizzato venga eseguito solo una volta quando vengono attivati più callback del token, l'implementazione dell'esempio controlla gli hash dei file. L'esempio usa l'hash di file SHA1. Viene implementato un nuovo tentativo con un'interruzione temporanea esponenziale. Il nuovo tentativo è presente perché potrebbe verificarsi un blocco di file che impedisce temporaneamente l'elaborazione di un nuovo hash in un file.

*Utilities/Utilities.cs*:

[!code-csharp[](change-tokens/samples/3.x/SampleApp/Utilities/Utilities.cs?name=snippet1)]

### <a name="simple-startup-change-token"></a>Semplice token di modifica dell'avvio

Registrare un callback dell'elemento `Action` consumer del token per le notifiche di modifica al token di ricaricamento della configurazione.

In `Startup.Configure`:

[!code-csharp[](change-tokens/samples/3.x/SampleApp/Startup.cs?name=snippet2)]

`config.GetReloadToken()` fornisce il token. Il callback è il metodo `InvokeChanged`:

[!code-csharp[](change-tokens/samples/3.x/SampleApp/Startup.cs?name=snippet3)]

Il valore `state` del callback viene usato per passare `IWebHostEnvironment`, che è utile per specificare il file di configurazione *appsettings* corretto da monitorare (ad esempio, *appsettings.Development.json* nell'ambiente di sviluppo). Gli hash dei file vengono usati per impedire che l'istruzione `WriteConsole` venga eseguita più volte a causa di più callback del token quando il file di configurazione è stato modificato una sola volta.

Questo sistema rimane in esecuzione finché l'app è in esecuzione e non può essere disabilitato dall'utente.

### <a name="monitor-configuration-changes-as-a-service"></a>Monitorare le modifiche di configurazione come servizio

L'esempio implementa:

* Monitoraggio di base del token di avvio.
* Monitoraggio come servizio.
* Un meccanismo per abilitare e disabilitare il monitoraggio.

L'esempio stabilisce un'interfaccia `IConfigurationMonitor`.

*Extensions/ConfigurationMonitor.cs*:

[!code-csharp[](change-tokens/samples/3.x/SampleApp/Extensions/ConfigurationMonitor.cs?name=snippet1)]

Il costruttore della classe implementata, `ConfigurationMonitor`, registra un callback per le notifiche di modifica:

[!code-csharp[](change-tokens/samples/3.x/SampleApp/Extensions/ConfigurationMonitor.cs?name=snippet2)]

`config.GetReloadToken()` fornisce il token. `InvokeChanged` è il metodo di callback. Il valore `state` in questa istanza è un riferimento all'istanza `IConfigurationMonitor` usata per accedere allo stato di monitoraggio. Vengono usate due proprietà:

* `MonitoringEnabled`: Indica se il callback deve eseguire il codice personalizzato.
* `CurrentState`: Descrive lo stato di monitoraggio corrente per l'uso nell'interfaccia utente.

Il metodo `InvokeChanged` è simile all'approccio precedente, ad eccezione del fatto che:

* Non esegue il proprio codice a meno che `MonitoringEnabled` non sia impostato su `true`.
* Restituisce l'elemento `state` corrente nell'output `WriteConsole`.

[!code-csharp[](change-tokens/samples/3.x/SampleApp/Extensions/ConfigurationMonitor.cs?name=snippet3)]

Un'istanza di `ConfigurationMonitor` viene registrata come servizio in `Startup.ConfigureServices`:

[!code-csharp[](change-tokens/samples/3.x/SampleApp/Startup.cs?name=snippet1)]

La pagina di indice offre all'utente il controllo sul monitoraggio della configurazione. L'istanza di `IConfigurationMonitor` viene inserita in `IndexModel`.

*Pages/Index.cshtml.cs*:

[!code-csharp[](change-tokens/samples/3.x/SampleApp/Pages/Index.cshtml.cs?name=snippet1)]

Il monitoraggio di configurazione (`_monitor`) viene usato per abilitare o disabilitare il monitoraggio e impostare lo stato corrente per commenti e suggerimenti dell'interfaccia utente:

[!code-csharp[](change-tokens/samples/3.x/SampleApp/Pages/Index.cshtml.cs?name=snippet2)]

Quando viene attivato `OnPostStartMonitoring`, il monitoraggio è abilitato e lo stato corrente viene cancellato. Quando viene attivato `OnPostStopMonitoring`, il monitoraggio è disabilitato e lo stato viene impostato in modo da indicare che il monitoraggio non viene eseguito.

I pulsanti nell'interfaccia utente abilitano e disabilitano il monitoraggio.

*Pages/index. cshtml*:

[!code-cshtml[](change-tokens/samples/3.x/SampleApp/Pages/Index.cshtml?name=snippet_Buttons)]

## <a name="monitor-cached-file-changes"></a>Monitorare le modifiche al file nella cache

È possibile memorizzare il contenuto del file nella cache in memoria usando <xref:Microsoft.Extensions.Caching.Memory.IMemoryCache>. La memorizzazione nella cache in memoria è descritta nell'argomento [Cache in memoria](xref:performance/caching/memory). Senza eseguire passaggi aggiuntivi, ad esempio l'implementazione descritta di seguito, la cache restituirà dati *non aggiornati* (obsoleti) se i dati di origine vengono modificati.

Ad esempio, se durante il rinnovo di un periodo di [scadenza variabile](xref:Microsoft.Extensions.Caching.Memory.MemoryCacheEntryOptions.SlidingExpiration) non si tiene conto dello stato di un file di origine memorizzato nella cache, i dati relativi ai file nella cache risulteranno non aggiornati. Ogni richiesta di dati rinnoverà il periodo di scadenza variabile, ma il file non verrà mai ricaricato nella cache. Le funzionalità dell'app che usano il contenuto del file memorizzato nella cache sono soggette alla possibile ricezione di contenuto non aggiornato.

L'uso dei token di modifica in uno scenario di memorizzazione di file nella cache consente di evitare la presenza di contenuto dei file non aggiornato nella cache. L'app di esempio illustra un'implementazione dell'approccio.

Nell'esempio viene usato `GetFileContent` per:

* Restituire il contenuto del file.
* Implementare un algoritmo di nuovi tentativi con interruzione temporanea esponenziale per i casi in cui un blocco di file impedisca temporaneamente la lettura di un file.

*Utilities/Utilities.cs*:

[!code-csharp[](change-tokens/samples/3.x/SampleApp/Utilities/Utilities.cs?name=snippet2)]

Viene creato un elemento `FileService` per gestire le ricerche nel file memorizzato nella cache. La chiamata del servizio al metodo `GetFileContent` prova a ottenere il contenuto del file dalla cache in memoria e a restituirlo al chiamante (*Services/FileService.cs*).

Se non è possibile reperire il contenuto memorizzato nella cache usando la chiave della cache, verranno intraprese le azioni seguenti:

1. Il contenuto del file viene ottenuto usando `GetFileContent`.
1. Un token di modifica viene ottenuto dal provider di file con [IFileProviders.Watch](xref:Microsoft.Extensions.FileProviders.IFileProvider.Watch*). Il callback del token viene attivato alla modifica del file.
1. Il contenuto del file viene memorizzato nella cache con un periodo di [scadenza variabile](xref:Microsoft.Extensions.Caching.Memory.MemoryCacheEntryOptions.SlidingExpiration). Al token di modifica viene associato [MemoryCacheEntryExtensions.AddExpirationToken](xref:Microsoft.Extensions.Caching.Memory.MemoryCacheEntryExtensions.AddExpirationToken*) per eliminare la voce della cache se il file viene modificato mentre è memorizzato nella cache.

Nell'esempio seguente i file vengono archiviati nella [radice del contenuto](xref:fundamentals/index#content-root)dell'app. `IWebHostEnvironment.ContentRootFileProvider` viene usato per ottenere un oggetto che <xref:Microsoft.Extensions.FileProviders.IFileProvider> punta all'oggetto dell'app `IWebHostEnvironment.ContentRootPath` . `filePath` viene ottenuto con [IFileInfo.PhysicalPath](xref:Microsoft.Extensions.FileProviders.IFileInfo.PhysicalPath).

[!code-csharp[](change-tokens/samples/3.x/SampleApp/Services/FileService.cs?name=snippet1)]

`FileService` è registrato nel contenitore di servizi insieme al servizio di memorizzazione nella cache.

In `Startup.ConfigureServices`:

[!code-csharp[](change-tokens/samples/3.x/SampleApp/Startup.cs?name=snippet4)]

Il modello di pagina carica il contenuto del file usando il servizio.

Nel metodo `OnGet` della pagina Index (*Pages/Index.cshtml.cs*):

[!code-csharp[](change-tokens/samples/3.x/SampleApp/Pages/Index.cshtml.cs?name=snippet3)]

## <a name="compositechangetoken-class"></a>Classe CompositeChangeToken

Per rappresentare una o più istanze di `IChangeToken` in un singolo oggetto, usare la classe <xref:Microsoft.Extensions.Primitives.CompositeChangeToken>.

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

`HasChanged` nel token composito indica `true` se l'elemento `HasChanged` di qualsiasi token rappresentato è `true`. `ActiveChangeCallbacks` nel token composito indica `true` se l'elemento `ActiveChangeCallbacks` di qualsiasi token rappresentato è `true`. Se si verificano più eventi di modifica simultanei, il callback della modifica composita viene richiamato una volta.

::: moniker-end

::: moniker range="< aspnetcore-3.0"

Un *token di modifica* è un blocco predefinito di uso generico e di basso livello che viene usato per il rilevamento delle modifiche di stato.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/change-tokens/samples/) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="ichangetoken-interface"></a>Interfaccia IChangeToken

<xref:Microsoft.Extensions.Primitives.IChangeToken> propaga le notifiche relative a una modifica apportata. `IChangeToken` risiede nello spazio dei nomi <xref:Microsoft.Extensions.Primitives?displayProperty=fullName>. Per le app che non usano il [metapacchetto Microsoft.AspNetCore.App](xref:fundamentals/metapackage-app), creare un riferimento al pacchetto per il pacchetto NuGet [Microsoft.Extensions.Primitives](https://www.nuget.org/packages/Microsoft.Extensions.Primitives/).

`IChangeToken` ha due proprietà:

* <xref:Microsoft.Extensions.Primitives.IChangeToken.ActiveChangeCallbacks> indica se il token genera callback in modo proattivo. Se `ActiveChangedCallbacks` è impostato su `false`, non viene mai chiamato alcun callback e l'app deve eseguire il polling di `HasChanged` per ottenere le modifiche. È anche possibile che un token non venga mai annullato se non vengono apportate modifiche o se il listener di modifica sottostante viene eliminato o disabilitato.
* <xref:Microsoft.Extensions.Primitives.IChangeToken.HasChanged> riceve un valore che indica se è stata apportata una modifica.

L' `IChangeToken` interfaccia include il metodo [RegisterChangeCallback (Action \<Object> , Object)](xref:Microsoft.Extensions.Primitives.IChangeToken.RegisterChangeCallback*) , che registra un callback che viene richiamato quando il token viene modificato. Prima che il callback venga richiamato è necessario impostare `HasChanged`.

## <a name="changetoken-class"></a>Classe ChangeToken

<xref:Microsoft.Extensions.Primitives.ChangeToken> è una classe statica usata per propagare le notifiche relative a una modifica che è stata apportata. `ChangeToken` risiede nello spazio dei nomi <xref:Microsoft.Extensions.Primitives?displayProperty=fullName>. Per le app che non usano il [metapacchetto Microsoft.AspNetCore.App](xref:fundamentals/metapackage-app), creare un riferimento al pacchetto per il pacchetto NuGet [Microsoft.Extensions.Primitives](https://www.nuget.org/packages/Microsoft.Extensions.Primitives/).

Il metodo [token. OnChange (Func \<IChangeToken> , Action)](xref:Microsoft.Extensions.Primitives.ChangeToken.OnChange*) registra un oggetto `Action` da chiamare ogni volta che il token cambia:

* `Func<IChangeToken>` produce il token.
* `Action` viene chiamato alla modifica del token.

L'overload di [token. OnChange \<TState> (Func \<IChangeToken> , Action \<TState> , tstate)](xref:Microsoft.Extensions.Primitives.ChangeToken.OnChange*) accetta un `TState` parametro aggiuntivo passato al consumer del token `Action` .

`OnChange` restituisce un oggetto <xref:System.IDisposable>. La chiamata a <xref:System.IDisposable.Dispose*> interrompe l'ascolto di altre modifiche da parte del token e rilascia le risorse del token.

## <a name="example-uses-of-change-tokens-in-aspnet-core"></a>Esempi di uso di token di modifica in ASP.NET Core

I token di modifica vengono usati nelle aree principali di ASP.NET Core per monitorare le modifiche apportate agli oggetti:

* Per monitorare le modifiche ai file, il metodo <xref:Microsoft.Extensions.FileProviders.IFileProvider.Watch*> di <xref:Microsoft.Extensions.FileProviders.IFileProvider> crea un elemento `IChangeToken` per le cartelle o i file specificati da controllare.
* È possibile aggiungere token `IChangeToken` alle voci della cache per attivare le eliminazioni dalla cache alla modifica.
* Per le modifiche di `TOptions`, l'implementazione <xref:Microsoft.Extensions.Options.OptionsMonitor`1> predefinita di <xref:Microsoft.Extensions.Options.IOptionsMonitor`1> ha un overload che accetta una o più istanze di <xref:Microsoft.Extensions.Options.IOptionsChangeTokenSource`1>. Ogni istanza restituisce un elemento `IChangeToken` per registrare un callback di notifica delle modifiche per il rilevamento delle modifiche apportate alle opzioni.

## <a name="monitor-for-configuration-changes"></a>Monitorare le modifiche di configurazione

Per impostazione predefinita, i modelli di ASP.NET Core usano [file di configurazione JSON](xref:fundamentals/configuration/index#json-configuration-provider) ( *appsettings.json* , *appsettings.Development.js* e *appsettings.Production.json*) per caricare le impostazioni di configurazione dell'app.

Questi file vengono configurati usando il metodo di estensione [AddJsonFile(IConfigurationBuilder, String, Boolean, Boolean)](xref:Microsoft.Extensions.Configuration.JsonConfigurationExtensions.AddJsonFile*) per <xref:Microsoft.Extensions.Configuration.ConfigurationBuilder> che accetta un parametro `reloadOnChange`. `reloadOnChange` indica se la configurazione deve essere ricaricata alla modifica del file. Questa impostazione viene visualizzata nel metodo pratico <xref:Microsoft.AspNetCore.WebHost><xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder*>:

```csharp
config.AddJsonFile("appsettings.json", optional: true, reloadOnChange: true)
      .AddJsonFile($"appsettings.{env.EnvironmentName}.json", optional: true, 
          reloadOnChange: true);
```

La configurazione basata su file è rappresentata da <xref:Microsoft.Extensions.Configuration.FileConfigurationSource>. `FileConfigurationSource` usa <xref:Microsoft.Extensions.FileProviders.IFileProvider> per monitorare i file.

Per impostazione predefinita, `IFileMonitor` viene offerto da una classe <xref:Microsoft.Extensions.FileProviders.PhysicalFileProvider> che usa <xref:System.IO.FileSystemWatcher> per monitorare le modifiche apportate al file di configurazione.

L'app di esempio illustra due implementazioni per il monitoraggio delle modifiche alla configurazione. In caso di modifica di uno qualsiasi dei file appsettings&mdash;, entrambe le implementazioni di monitoraggio dei file eseguono codice personalizzato e l'app di esempio scrive un messaggio nella console.

L'elemento `FileSystemWatcher` di un file di configurazione può attivare più callback del token per una singola modifica del file di configurazione. Per assicurarsi che il codice personalizzato venga eseguito solo una volta quando vengono attivati più callback del token, l'implementazione dell'esempio controlla gli hash dei file. L'esempio usa l'hash di file SHA1. Viene implementato un nuovo tentativo con un'interruzione temporanea esponenziale. Il nuovo tentativo è presente perché potrebbe verificarsi un blocco di file che impedisce temporaneamente l'elaborazione di un nuovo hash in un file.

*Utilities/Utilities.cs*:

[!code-csharp[](change-tokens/samples/2.x/SampleApp/Utilities/Utilities.cs?name=snippet1)]

### <a name="simple-startup-change-token"></a>Semplice token di modifica dell'avvio

Registrare un callback dell'elemento `Action` consumer del token per le notifiche di modifica al token di ricaricamento della configurazione.

In `Startup.Configure`:

[!code-csharp[](change-tokens/samples/2.x/SampleApp/Startup.cs?name=snippet2)]

`config.GetReloadToken()` fornisce il token. Il callback è il metodo `InvokeChanged`:

[!code-csharp[](change-tokens/samples/2.x/SampleApp/Startup.cs?name=snippet3)]

Il valore `state` del callback viene usato per passare `IHostingEnvironment`, che è utile per specificare il file di configurazione *appsettings* corretto da monitorare (ad esempio, *appsettings.Development.json* nell'ambiente di sviluppo). Gli hash dei file vengono usati per impedire che l'istruzione `WriteConsole` venga eseguita più volte a causa di più callback del token quando il file di configurazione è stato modificato una sola volta.

Questo sistema rimane in esecuzione finché l'app è in esecuzione e non può essere disabilitato dall'utente.

### <a name="monitor-configuration-changes-as-a-service"></a>Monitorare le modifiche di configurazione come servizio

L'esempio implementa:

* Monitoraggio di base del token di avvio.
* Monitoraggio come servizio.
* Un meccanismo per abilitare e disabilitare il monitoraggio.

L'esempio stabilisce un'interfaccia `IConfigurationMonitor`.

*Extensions/ConfigurationMonitor.cs*:

[!code-csharp[](change-tokens/samples/2.x/SampleApp/Extensions/ConfigurationMonitor.cs?name=snippet1)]

Il costruttore della classe implementata, `ConfigurationMonitor`, registra un callback per le notifiche di modifica:

[!code-csharp[](change-tokens/samples/2.x/SampleApp/Extensions/ConfigurationMonitor.cs?name=snippet2)]

`config.GetReloadToken()` fornisce il token. `InvokeChanged` è il metodo di callback. Il valore `state` in questa istanza è un riferimento all'istanza `IConfigurationMonitor` usata per accedere allo stato di monitoraggio. Vengono usate due proprietà:

* `MonitoringEnabled`: Indica se il callback deve eseguire il codice personalizzato.
* `CurrentState`: Descrive lo stato di monitoraggio corrente per l'uso nell'interfaccia utente.

Il metodo `InvokeChanged` è simile all'approccio precedente, ad eccezione del fatto che:

* Non esegue il proprio codice a meno che `MonitoringEnabled` non sia impostato su `true`.
* Restituisce l'elemento `state` corrente nell'output `WriteConsole`.

[!code-csharp[](change-tokens/samples/2.x/SampleApp/Extensions/ConfigurationMonitor.cs?name=snippet3)]

Un'istanza di `ConfigurationMonitor` viene registrata come servizio in `Startup.ConfigureServices`:

[!code-csharp[](change-tokens/samples/2.x/SampleApp/Startup.cs?name=snippet1)]

La pagina di indice offre all'utente il controllo sul monitoraggio della configurazione. L'istanza di `IConfigurationMonitor` viene inserita in `IndexModel`.

*Pages/Index.cshtml.cs*:

[!code-csharp[](change-tokens/samples/2.x/SampleApp/Pages/Index.cshtml.cs?name=snippet1)]

Il monitoraggio di configurazione (`_monitor`) viene usato per abilitare o disabilitare il monitoraggio e impostare lo stato corrente per commenti e suggerimenti dell'interfaccia utente:

[!code-csharp[](change-tokens/samples/2.x/SampleApp/Pages/Index.cshtml.cs?name=snippet2)]

Quando viene attivato `OnPostStartMonitoring`, il monitoraggio è abilitato e lo stato corrente viene cancellato. Quando viene attivato `OnPostStopMonitoring`, il monitoraggio è disabilitato e lo stato viene impostato in modo da indicare che il monitoraggio non viene eseguito.

I pulsanti nell'interfaccia utente abilitano e disabilitano il monitoraggio.

*Pages/index. cshtml*:

[!code-cshtml[](change-tokens/samples/2.x/SampleApp/Pages/Index.cshtml?name=snippet_Buttons)]

## <a name="monitor-cached-file-changes"></a>Monitorare le modifiche al file nella cache

È possibile memorizzare il contenuto del file nella cache in memoria usando <xref:Microsoft.Extensions.Caching.Memory.IMemoryCache>. La memorizzazione nella cache in memoria è descritta nell'argomento [Cache in memoria](xref:performance/caching/memory). Senza eseguire passaggi aggiuntivi, ad esempio l'implementazione descritta di seguito, la cache restituirà dati *non aggiornati* (obsoleti) se i dati di origine vengono modificati.

Ad esempio, se durante il rinnovo di un periodo di [scadenza variabile](xref:Microsoft.Extensions.Caching.Memory.MemoryCacheEntryOptions.SlidingExpiration) non si tiene conto dello stato di un file di origine memorizzato nella cache, i dati relativi ai file nella cache risulteranno non aggiornati. Ogni richiesta di dati rinnoverà il periodo di scadenza variabile, ma il file non verrà mai ricaricato nella cache. Le funzionalità dell'app che usano il contenuto del file memorizzato nella cache sono soggette alla possibile ricezione di contenuto non aggiornato.

L'uso dei token di modifica in uno scenario di memorizzazione di file nella cache consente di evitare la presenza di contenuto dei file non aggiornato nella cache. L'app di esempio illustra un'implementazione dell'approccio.

Nell'esempio viene usato `GetFileContent` per:

* Restituire il contenuto del file.
* Implementare un algoritmo di nuovi tentativi con interruzione temporanea esponenziale per i casi in cui un blocco di file impedisca temporaneamente la lettura di un file.

*Utilities/Utilities.cs*:

[!code-csharp[](change-tokens/samples/2.x/SampleApp/Utilities/Utilities.cs?name=snippet2)]

Viene creato un elemento `FileService` per gestire le ricerche nel file memorizzato nella cache. La chiamata del servizio al metodo `GetFileContent` prova a ottenere il contenuto del file dalla cache in memoria e a restituirlo al chiamante (*Services/FileService.cs*).

Se non è possibile reperire il contenuto memorizzato nella cache usando la chiave della cache, verranno intraprese le azioni seguenti:

1. Il contenuto del file viene ottenuto usando `GetFileContent`.
1. Un token di modifica viene ottenuto dal provider di file con [IFileProviders.Watch](xref:Microsoft.Extensions.FileProviders.IFileProvider.Watch*). Il callback del token viene attivato alla modifica del file.
1. Il contenuto del file viene memorizzato nella cache con un periodo di [scadenza variabile](xref:Microsoft.Extensions.Caching.Memory.MemoryCacheEntryOptions.SlidingExpiration). Al token di modifica viene associato [MemoryCacheEntryExtensions.AddExpirationToken](xref:Microsoft.Extensions.Caching.Memory.MemoryCacheEntryExtensions.AddExpirationToken*) per eliminare la voce della cache se il file viene modificato mentre è memorizzato nella cache.

Nell'esempio seguente i file vengono archiviati nella [radice del contenuto](xref:fundamentals/index#content-root)dell'app. [IHostingEnvironment.ContentRootFileProvider](xref:Microsoft.AspNetCore.Hosting.IHostingEnvironment.ContentRootFileProvider) viene usato per ottenere un <xref:Microsoft.Extensions.FileProviders.IFileProvider> che punta al <xref:Microsoft.AspNetCore.Hosting.IHostingEnvironment.ContentRootPath> dell'app. `filePath` viene ottenuto con [IFileInfo.PhysicalPath](xref:Microsoft.Extensions.FileProviders.IFileInfo.PhysicalPath).

[!code-csharp[](change-tokens/samples/2.x/SampleApp/Services/FileService.cs?name=snippet1)]

`FileService` è registrato nel contenitore di servizi insieme al servizio di memorizzazione nella cache.

In `Startup.ConfigureServices`:

[!code-csharp[](change-tokens/samples/2.x/SampleApp/Startup.cs?name=snippet4)]

Il modello di pagina carica il contenuto del file usando il servizio.

Nel metodo `OnGet` della pagina Index (*Pages/Index.cshtml.cs*):

[!code-csharp[](change-tokens/samples/2.x/SampleApp/Pages/Index.cshtml.cs?name=snippet3)]

## <a name="compositechangetoken-class"></a>Classe CompositeChangeToken

Per rappresentare una o più istanze di `IChangeToken` in un singolo oggetto, usare la classe <xref:Microsoft.Extensions.Primitives.CompositeChangeToken>.

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

`HasChanged` nel token composito indica `true` se l'elemento `HasChanged` di qualsiasi token rappresentato è `true`. `ActiveChangeCallbacks` nel token composito indica `true` se l'elemento `ActiveChangeCallbacks` di qualsiasi token rappresentato è `true`. Se si verificano più eventi di modifica simultanei, il callback della modifica composita viene richiamato una volta.

::: moniker-end

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:performance/caching/memory>
* <xref:performance/caching/distributed>
* <xref:performance/caching/response>
* <xref:performance/caching/middleware>
* <xref:mvc/views/tag-helpers/builtin-th/cache-tag-helper>
* <xref:mvc/views/tag-helpers/builtin-th/distributed-cache-tag-helper>
