---
title: Memorizzare nella cache in memoria ASP.NET Core
author: rick-anderson
description: Informazioni su come memorizzare i dati nella cache in memoria in ASP.NET Core.
ms.author: riande
ms.custom: mvc
ms.date: 02/02/2020
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
uid: performance/caching/memory
ms.openlocfilehash: 19e8dc0ae4d5f8fd28d03d5be87c0b1bbf32d940
ms.sourcegitcommit: 04ad9cd26fcaa8bd11e261d3661f375f5f343cdc
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/10/2021
ms.locfileid: "100107220"
---
# <a name="cache-in-memory-in-aspnet-core"></a>Memorizzare nella cache in memoria ASP.NET Core

::: moniker range=">= aspnetcore-3.0"

Di [Rick Anderson](https://twitter.com/RickAndMSFT), [John Luo](https://github.com/JunTaoLuo)e [Steve Smith](https://ardalis.com/)

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/performance/caching/memory/3.0sample) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="caching-basics"></a>Nozioni fondamentali sulla memorizzazione nella cache

La memorizzazione nella cache può migliorare significativamente le prestazioni e la scalabilità di un'app riducendo il lavoro necessario per generare il contenuto. La memorizzazione nella cache funziona meglio con i dati che cambiano raramente **ed** è costoso da generare. La memorizzazione nella cache crea una copia dei dati che possono essere restituiti molto più velocemente rispetto all'origine. Le app devono essere scritte e testate in modo da non dipendere **mai** dai dati memorizzati nella cache.

ASP.NET Core supporta diverse cache. La cache più semplice si basa su [IMemoryCache](/dotnet/api/microsoft.extensions.caching.memory.imemorycache). `IMemoryCache` rappresenta una cache archiviata nella memoria del server Web. Le app in esecuzione in un server farm (più server) devono garantire che le sessioni siano permanenti quando si usa la cache in memoria. Le sessioni permanenti garantiscono che le richieste successive provenienti da un client vadano tutte allo stesso server. Ad esempio, le app Web di Azure usano [Application Request Routing](https://www.iis.net/learn/extensions/planning-for-arr) (arr) per indirizzare tutte le richieste successive allo stesso server.

Le sessioni non permanenti in una Web farm richiedono una [cache distribuita](distributed.md) per evitare problemi di coerenza della cache. Per alcune app, una cache distribuita può supportare una maggiore scalabilità orizzontale rispetto a una cache in memoria. L'uso di una cache distribuita trasferisce la memoria cache a un processo esterno.

La cache in memoria può archiviare qualsiasi oggetto. L'interfaccia della cache distribuita è limitata a `byte[]` . Gli elementi memorizzati nella cache in memoria e nella cache distribuita vengono archiviati come coppie chiave-valore.

## <a name="systemruntimecachingmemorycache"></a>System. Runtime. Caching/MemoryCache

<xref:System.Runtime.Caching>/<xref:System.Runtime.Caching.MemoryCache> ([Pacchetto NuGet](https://www.nuget.org/packages/System.Runtime.Caching/)) può essere usato con:

* .NET Standard 2,0 o versione successiva.
* Qualsiasi [implementazione di .NET](/dotnet/standard/net-standard#net-implementation-support) destinata a .NET standard 2,0 o versione successiva. Ad esempio, ASP.NET Core 2,0 o versione successiva.
* .NET Framework 4.5 o versioni successive.

È consigliabile usare [Microsoft. Extensions. Caching. memory](https://www.nuget.org/packages/Microsoft.Extensions.Caching.Memory/) / `IMemoryCache` (descritto in questo articolo) `System.Runtime.Caching` / `MemoryCache` perché è più integrato in ASP.NET Core. Ad esempio, `IMemoryCache` funziona in modo nativo con ASP.NET Core [inserimento delle dipendenze](xref:fundamentals/dependency-injection).

Usare `System.Runtime.Caching` / `MemoryCache` come Bridge di compatibilità quando si porta il codice da ASP.NET 4. x a ASP.NET Core.

## <a name="cache-guidelines"></a>Linee guida per la cache

* Il codice deve sempre avere un'opzione di fallback per recuperare i dati e **non** dipendere da un valore memorizzato nella cache disponibile.
* La cache usa una risorsa scarsa, ovvero la memoria. Limita aumento dimensioni cache:
  * Non **usare input** esterno come chiavi di cache.
  * Usare le scadenze per limitare la crescita della cache.
  * [Per limitare le dimensioni della cache, usare le dimensioni, le dimensioni e il valore di SizeLimit](#use-setsize-size-and-sizelimit-to-limit-cache-size). Il runtime di ASP.NET Core **non limita le** dimensioni della cache in base all'utilizzo eccessivo della memoria. Spetta allo sviluppatore limitare le dimensioni della cache.

## <a name="use-imemorycache"></a>Usare IMemoryCache

> [!WARNING]
> L'uso di una cache *Shared* Memory dall' [inserimento delle dipendenze](xref:fundamentals/dependency-injection) e della chiamata `SetSize` `Size` di, o `SizeLimit` per limitare le dimensioni della cache può causare un errore dell'app. Quando si imposta un limite di dimensioni in una cache, è necessario che tutte le voci specifichino una dimensione al momento dell'aggiunta. Questo può causare problemi perché gli sviluppatori potrebbero non avere il controllo completo su ciò che usa la cache condivisa. Ad esempio, Entity Framework Core utilizza la cache condivisa e non specifica una dimensione. Se un'app imposta un limite per le dimensioni della cache e USA EF Core, l'app genera un'eccezione `InvalidOperationException` .
> Quando `SetSize` si usa, `Size` o `SizeLimit` per limitare la cache, creare un singleton della cache per la memorizzazione nella cache. Per altre informazioni e per un esempio, vedere [use sesize, Size e SizeLimit per limitare le dimensioni della cache](#use-setsize-size-and-sizelimit-to-limit-cache-size).
> Una cache condivisa è condivisa da altri Framework o librerie. Ad esempio, EF Core utilizza la cache condivisa e non specifica una dimensione. 

La memorizzazione nella cache in memoria è un *servizio* a cui viene fatto riferimento da un'app che usa l' [inserimento delle dipendenze](xref:fundamentals/dependency-injection). Richiedere l' `IMemoryCache` istanza nel costruttore:

[!code-csharp[](memory/3.0sample/WebCacheSample/Controllers/HomeController.cs?name=snippet_ctor)]

Il codice seguente usa [TryGetValue](/dotnet/api/microsoft.extensions.caching.memory.imemorycache.trygetvalue?view=aspnetcore-2.0#Microsoft_Extensions_Caching_Memory_IMemoryCache_TryGetValue_System_Object_System_Object__) per verificare se un'ora è nella cache. Se l'ora non viene memorizzata nella cache, viene creata una nuova voce che viene aggiunta alla cache con [set](/dotnet/api/microsoft.extensions.caching.memory.cacheextensions.set?view=aspnetcore-2.0#Microsoft_Extensions_Caching_Memory_CacheExtensions_Set__1_Microsoft_Extensions_Caching_Memory_IMemoryCache_System_Object___0_Microsoft_Extensions_Caching_Memory_MemoryCacheEntryOptions_). La `CacheKeys` classe fa parte dell'esempio di download.

[!code-csharp[](memory/3.0sample/WebCacheSample/CacheKeys.cs)]

[!code-csharp[](memory/3.0sample/WebCacheSample/Controllers/HomeController.cs?name=snippet1)]

Vengono visualizzati l'ora corrente e l'ora memorizzata nella cache:

[!code-cshtml[](memory/3.0sample/WebCacheSample/Views/Home/Cache.cshtml)]

Il codice seguente usa il metodo di estensione [set](/dotnet/api/microsoft.extensions.caching.memory.cacheextensions.set#Microsoft_Extensions_Caching_Memory_CacheExtensions_Set__1_Microsoft_Extensions_Caching_Memory_IMemoryCache_System_Object___0_System_TimeSpan_) per memorizzare i dati nella cache per un'ora relativa senza creare l' `MemoryCacheEntryOptions` oggetto.
[!code-csharp[](memory/3.0sample/WebCacheSample/Controllers/HomeController.cs?name=snippet_set)]

Il valore memorizzato nella `DateTime` cache rimane nella cache mentre sono presenti richieste entro il periodo di timeout.

Il codice seguente usa [GetOrCreate](/dotnet/api/microsoft.extensions.caching.memory.cacheextensions.getorcreate#Microsoft_Extensions_Caching_Memory_CacheExtensions_GetOrCreate__1_Microsoft_Extensions_Caching_Memory_IMemoryCache_System_Object_System_Func_Microsoft_Extensions_Caching_Memory_ICacheEntry___0__) e [GetOrCreateAsync](/dotnet/api/microsoft.extensions.caching.memory.cacheextensions.getorcreateasync#Microsoft_Extensions_Caching_Memory_CacheExtensions_GetOrCreateAsync__1_Microsoft_Extensions_Caching_Memory_IMemoryCache_System_Object_System_Func_Microsoft_Extensions_Caching_Memory_ICacheEntry_System_Threading_Tasks_Task___0___) per memorizzare i dati nella cache.

[!code-csharp[](memory/3.0sample/WebCacheSample/Controllers/HomeController.cs?name=snippet2&highlight=3-7,14-19)]

Il codice seguente chiama [Get](/dotnet/api/microsoft.extensions.caching.memory.cacheextensions.get#Microsoft_Extensions_Caching_Memory_CacheExtensions_Get__1_Microsoft_Extensions_Caching_Memory_IMemoryCache_System_Object_) per recuperare l'ora memorizzata nella cache:

[!code-csharp[](memory/3.0sample/WebCacheSample/Controllers/HomeController.cs?name=snippet_gct)]

Il codice seguente ottiene o crea un elemento memorizzato nella cache con scadenza assoluta:

[!code-csharp[](memory/3.0sample/WebCacheSample/Controllers/HomeController.cs?name=snippet99)]

Un set di elementi memorizzato nella cache con una scadenza variabile è a rischio di diventare obsoleto. Se si accede con maggiore frequenza rispetto all'intervallo di scadenza variabile, l'elemento non scadrà mai. Combinare una scadenza variabile con una scadenza assoluta per garantire che l'elemento scada al termine del periodo di scadenza assoluto. La scadenza assoluta imposta un limite superiore per quanto tempo l'elemento può essere memorizzato nella cache, consentendo comunque la scadenza dell'elemento in un momento precedente se non è richiesto entro l'intervallo di scadenza variabile. Quando vengono specificate sia la scadenza assoluta che quella variabile, le scadenze sono logicamente ORed. Se l'intervallo di scadenza variabile *o* l'ora di scadenza assoluta passano, l'elemento viene rimosso dalla cache.

Il codice seguente ottiene o crea un elemento memorizzato nella cache con una scadenza scorrevole *e* assoluta:

[!code-csharp[](memory/3.0sample/WebCacheSample/Controllers/HomeController.cs?name=snippet9)]

Il codice precedente garantisce che i dati non verranno memorizzati nella cache più a lungo del tempo assoluto.

<xref:Microsoft.Extensions.Caching.Memory.CacheExtensions.GetOrCreate*>, <xref:Microsoft.Extensions.Caching.Memory.CacheExtensions.GetOrCreateAsync*> e <xref:Microsoft.Extensions.Caching.Memory.CacheExtensions.Get*> sono metodi di estensione nella <xref:Microsoft.Extensions.Caching.Memory.CacheExtensions> classe. Questi metodi estendono le funzionalità di <xref:Microsoft.Extensions.Caching.Memory.IMemoryCache> .

## <a name="memorycacheentryoptions"></a>MemoryCacheEntryOptions

L'esempio seguente:

* Imposta una data di scadenza variabile. Le richieste che accedono a questo elemento memorizzato nella cache reimpostano il clock di scadenza variabile.
* Imposta la priorità della cache su [CacheItemPriority. NeverRemove](xref:Microsoft.Extensions.Caching.Memory.CacheItemPriority.NeverRemove).
* Imposta un [PostEvictionDelegate](/dotnet/api/microsoft.extensions.caching.memory.postevictiondelegate) che verrà chiamato dopo la rimozione della voce dalla cache. Il callback viene eseguito su un thread diverso dal codice che rimuove l'elemento dalla cache.

[!code-csharp[](memory/3.0sample/WebCacheSample/Controllers/HomeController.cs?name=snippet_et&highlight=14-21)]

## <a name="use-setsize-size-and-sizelimit-to-limit-cache-size"></a>Usare le dimensioni di ridimensionamento, dimensioni e SizeLimit per limitare le dimensioni della cache

Un' `MemoryCache` istanza può facoltativamente specificare e applicare un limite di dimensioni. Il limite delle dimensioni della cache non dispone di un'unità di misura definita perché la cache non ha alcun meccanismo per misurare la dimensione delle voci. Se viene impostato il limite delle dimensioni della cache, è necessario che tutte le voci specifichino la dimensione. Il runtime di ASP.NET Core non limita le dimensioni della cache in base all'utilizzo eccessivo della memoria. Spetta allo sviluppatore limitare le dimensioni della cache. Le dimensioni specificate sono in unità scelte dallo sviluppatore.

Ad esempio:

* Se l'app Web è stata utilizzata principalmente per la memorizzazione nella cache delle stringhe, le dimensioni della voce della cache potrebbero essere la lunghezza della stringa.
* L'app può specificare le dimensioni di tutte le voci come 1 e il limite di dimensioni è il numero di voci.

Se <xref:Microsoft.Extensions.Caching.Memory.MemoryCacheOptions.SizeLimit> non è impostato, la cache cresce senza alcun limite. Il runtime di ASP.NET Core non elimina la cache quando la memoria di sistema è insufficiente. Le app devono essere progettate per:

* Limitare la crescita della cache.
* Chiamare <xref:Microsoft.Extensions.Caching.Memory.MemoryCache.Compact*> o <xref:Microsoft.Extensions.Caching.Memory.MemoryCache.Remove*> quando la memoria disponibile è limitata:

Il codice seguente consente di creare una dimensione fissa unificata <xref:Microsoft.Extensions.Caching.Memory.MemoryCache> accessibile dall' [inserimento delle dipendenze](xref:fundamentals/dependency-injection):

[!code-csharp[](memory/sample/RPcache/Services/MyMemoryCache.cs?name=snippet)]

`SizeLimit` non dispone di unità. Le voci memorizzate nella cache devono specificare le dimensioni in qualsiasi unità ritenute più appropriate se è stato impostato il limite delle dimensioni della cache. Tutti gli utenti di un'istanza della cache devono usare lo stesso sistema di unità. Una voce non verrà memorizzata nella cache se la somma delle dimensioni della voce memorizzata nella cache supera il valore specificato da `SizeLimit` . Se non è impostato alcun limite per le dimensioni della cache, le dimensioni della cache impostate per la voce verranno ignorate.

Il codice seguente esegue la registrazione `MyMemoryCache` con il contenitore di [inserimento delle dipendenze](xref:fundamentals/dependency-injection) .

[!code-csharp[](memory/3.0sample/RPcache/Startup.cs?name=snippet)]

`MyMemoryCache` viene creato come cache di memoria indipendente per i componenti che sono consapevoli di questa dimensione limitata della cache e sanno come impostare la dimensione della voce della cache in modo appropriato.

Il codice seguente usa `MyMemoryCache` :

[!code-csharp[](memory/3.0sample/RPcache/Pages/SetSize.cshtml.cs?name=snippet)]

La dimensione della voce della cache può essere impostata da <xref:Microsoft.Extensions.Caching.Memory.MemoryCacheEntryOptions.Size> o dai <xref:Microsoft.Extensions.Caching.Memory.MemoryCacheEntryExtensions.SetSize*> metodi di estensione:

[!code-csharp[](memory/3.0sample/RPcache/Pages/SetSize.cshtml.cs?name=snippet2&highlight=9,10,14,15)]

### <a name="memorycachecompact"></a>MemoryCache. Compact

`MemoryCache.Compact` tenta di rimuovere la percentuale specificata della cache nell'ordine seguente:

* Tutti gli elementi scaduti.
* Elementi per priorità. Gli elementi con priorità più bassa vengono rimossi per primi.
* Oggetti utilizzati meno di recente.
* Elementi con la prima scadenza assoluta.
* Elementi con la scadenza variabile meno recente.

Gli elementi aggiunti con priorità <xref:Microsoft.Extensions.Caching.Memory.CacheItemPriority.NeverRemove> non vengono mai rimossi. Il codice seguente rimuove un elemento della cache e chiama `Compact` :

[!code-csharp[](memory/3.0sample/RPcache/Pages/TestCache.cshtml.cs?name=snippet3)]

Per altre informazioni, vedere [Compact Source su GitHub](https://github.com/dotnet/extensions/blob/v3.0.0-preview8.19405.4/src/Caching/Memory/src/MemoryCache.cs#L382-L393) .

## <a name="cache-dependencies"></a>Dipendenze della cache

Nell'esempio seguente viene illustrato come impostare come scaduti una voce della cache in caso di scadenza di una voce dipendente. <xref:Microsoft.Extensions.Primitives.CancellationChangeToken>Viene aggiunto un oggetto all'elemento memorizzato nella cache. Quando `Cancel` viene chiamato su `CancellationTokenSource` , entrambe le voci della cache vengono eliminate.

[!code-csharp[](memory/3.0sample/WebCacheSample/Controllers/HomeController.cs?name=snippet_ed)]

L'uso di un oggetto <xref:System.Threading.CancellationTokenSource> consente di rimuovere più voci della cache come gruppo. Con il `using` modello nel codice precedente, le voci della cache create all'interno del `using` blocco erediteranno i trigger e le impostazioni di scadenza.

## <a name="additional-notes"></a>Note aggiuntive

* La scadenza non viene eseguita in background. Non sono presenti timer che analizzano attivamente la cache per gli elementi scaduti. Qualsiasi attività nella cache ( `Get` , `Set` , `Remove` ) può attivare un'analisi in background per gli elementi scaduti. Un timer in `CancellationTokenSource` ( <xref:System.Threading.CancellationTokenSource.CancelAfter*> ) rimuove anche la voce e attiva un'analisi per gli elementi scaduti. Nell'esempio seguente viene utilizzato [CancellationTokenSource (TimeSpan)](/dotnet/api/system.threading.cancellationtokensource.-ctor) per il token registrato. Quando questo token viene attivato, rimuove immediatamente la voce e genera i callback di rimozione:

[!code-csharp[](memory/3.0sample/WebCacheSample/Controllers/HomeController.cs?name=snippet_ae)]

* Quando si usa un callback per ripopolare un elemento della cache:

  * Più richieste possono trovare il valore della chiave memorizzato nella cache perché il callback non è stato completato.
  * Questo può comportare la ricompilazione dell'elemento memorizzato nella cache da parte di più thread.

* Quando viene utilizzata una voce della cache per crearne un'altra, l'elemento figlio copia i token di scadenza della voce padre e le impostazioni di scadenza basate sul tempo. L'elemento figlio non è scaduto dalla rimozione manuale o dall'aggiornamento della voce padre.

* Utilizzare <xref:Microsoft.Extensions.Caching.Memory.ICacheEntry.PostEvictionCallbacks> per impostare i callback che verranno generati dopo la rimozione della voce della cache dalla cache.
* Per la maggior parte delle app `IMemoryCache` è abilitato. Ad esempio, chiamando `AddMvc` , `AddControllersWithViews` , `AddRazorPages` , `AddMvcCore().AddRazorViewEngine` e molti altri `Add{Service}` metodi in `ConfigureServices` , Abilita `IMemoryCache` . Per le app che non chiamano uno dei metodi precedenti `Add{Service}` , potrebbe essere necessario chiamare <xref:Microsoft.Extensions.DependencyInjection.MemoryCacheServiceCollectionExtensions.AddMemoryCache*> in `ConfigureServices` .

## <a name="background-cache-update"></a>Aggiornamento della cache in background

Usare un [servizio in background](xref:fundamentals/host/hosted-services) , ad esempio <xref:Microsoft.Extensions.Hosting.IHostedService> per aggiornare la cache. Il servizio in background può ricalcolare le voci e quindi assegnarle alla cache solo quando sono pronte.

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:performance/caching/distributed>
* <xref:fundamentals/change-tokens>
* <xref:performance/caching/response>
* <xref:performance/caching/middleware>
* <xref:mvc/views/tag-helpers/builtin-th/cache-tag-helper>
* <xref:mvc/views/tag-helpers/builtin-th/distributed-cache-tag-helper>

::: moniker-end

::: moniker range="< aspnetcore-3.0"

<!-- This is the 2.1 version -->
Di [Rick Anderson](https://twitter.com/RickAndMSFT), [John Luo](https://github.com/JunTaoLuo)e [Steve Smith](https://ardalis.com/)

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/performance/caching/memory/sample) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="caching-basics"></a>Nozioni fondamentali sulla memorizzazione nella cache

La memorizzazione nella cache può migliorare significativamente le prestazioni e la scalabilità di un'app riducendo il lavoro necessario per generare il contenuto. La memorizzazione nella cache funziona meglio con i dati che cambiano raramente. La memorizzazione nella cache crea una copia dei dati che possono essere restituiti molto più velocemente rispetto a quelli dell'origine originale. Il codice deve essere scritto e testato in modo da non dipendere **mai** dai dati memorizzati nella cache.

ASP.NET Core supporta diverse cache. La cache più semplice si basa su [IMemoryCache](/dotnet/api/microsoft.extensions.caching.memory.imemorycache), che rappresenta una cache archiviata nella memoria del server Web. Le app eseguite in un server farm (più server) devono garantire che le sessioni siano permanenti quando si usa la cache in memoria. Le sessioni permanenti garantiscono che le richieste successive provenienti da un client vadano tutte allo stesso server. Ad esempio, le app Web di Azure usano [Application Request Routing](https://www.iis.net/learn/extensions/planning-for-arr) (arr) per indirizzare tutte le richieste da un agente utente allo stesso server.

Le sessioni non permanenti in una Web farm richiedono una [cache distribuita](distributed.md) per evitare problemi di coerenza della cache. Per alcune app, una cache distribuita può supportare una maggiore scalabilità orizzontale rispetto a una cache in memoria. L'uso di una cache distribuita trasferisce la memoria cache a un processo esterno.

La cache in memoria può archiviare qualsiasi oggetto. L'interfaccia della cache distribuita è limitata a `byte[]` . Gli elementi memorizzati nella cache in memoria e nella cache distribuita vengono archiviati come coppie chiave-valore.

## <a name="systemruntimecachingmemorycache"></a>System. Runtime. Caching/MemoryCache

<xref:System.Runtime.Caching>/<xref:System.Runtime.Caching.MemoryCache> ([Pacchetto NuGet](https://www.nuget.org/packages/System.Runtime.Caching/)) può essere usato con:

* .NET Standard 2,0 o versione successiva.
* Qualsiasi [implementazione di .NET](/dotnet/standard/net-standard#net-implementation-support) destinata a .NET standard 2,0 o versione successiva. Ad esempio, ASP.NET Core 2,0 o versione successiva.
* .NET Framework 4.5 o versioni successive.

È consigliabile usare [Microsoft. Extensions. Caching. memory](https://www.nuget.org/packages/Microsoft.Extensions.Caching.Memory/) / `IMemoryCache` (descritto in questo articolo) `System.Runtime.Caching` / `MemoryCache` perché è più integrato in ASP.NET Core. Ad esempio, `IMemoryCache` funziona in modo nativo con ASP.NET Core [inserimento delle dipendenze](xref:fundamentals/dependency-injection).

Usare `System.Runtime.Caching` / `MemoryCache` come Bridge di compatibilità quando si porta il codice da ASP.NET 4. x a ASP.NET Core.

## <a name="cache-guidelines"></a>Linee guida per la cache

* Il codice deve sempre avere un'opzione di fallback per recuperare i dati e **non** dipendere da un valore memorizzato nella cache disponibile.
* La cache usa una risorsa scarsa, ovvero la memoria. Limita aumento dimensioni cache:
  * Non **usare input** esterno come chiavi di cache.
  * Usare le scadenze per limitare la crescita della cache.
  * [Per limitare le dimensioni della cache, usare le dimensioni, le dimensioni e il valore di SizeLimit](#use-setsize-size-and-sizelimit-to-limit-cache-size). Il runtime di ASP.NET Core non limita le dimensioni della cache in base all'utilizzo eccessivo della memoria. Spetta allo sviluppatore limitare le dimensioni della cache.

## <a name="using-imemorycache"></a>Uso di IMemoryCache

> [!WARNING]
> L'uso di una cache *Shared* Memory dall' [inserimento delle dipendenze](xref:fundamentals/dependency-injection) e della chiamata `SetSize` `Size` di, o `SizeLimit` per limitare le dimensioni della cache può causare un errore dell'app. Quando si imposta un limite di dimensioni in una cache, è necessario che tutte le voci specifichino una dimensione al momento dell'aggiunta. Questo può causare problemi perché gli sviluppatori potrebbero non avere il controllo completo su ciò che usa la cache condivisa. Ad esempio, Entity Framework Core utilizza la cache condivisa e non specifica una dimensione. Se un'app imposta un limite per le dimensioni della cache e USA EF Core, l'app genera un'eccezione `InvalidOperationException` .
> Quando `SetSize` si usa, `Size` o `SizeLimit` per limitare la cache, creare un singleton della cache per la memorizzazione nella cache. Per altre informazioni e per un esempio, vedere [use sesize, Size e SizeLimit per limitare le dimensioni della cache](#use-setsize-size-and-sizelimit-to-limit-cache-size).

La memorizzazione nella cache in memoria è un *servizio* a cui viene fatto riferimento dall'app mediante l' [inserimento di dipendenze](../../fundamentals/dependency-injection.md). Chiama `AddMemoryCache` in `ConfigureServices` :

[!code-csharp[](memory/sample/WebCache/Startup.cs?highlight=9)]

Richiedere l' `IMemoryCache` istanza nel costruttore:

[!code-csharp[](memory/sample/WebCache/Controllers/HomeController.cs?name=snippet_ctor)]

`IMemoryCache` richiede il pacchetto NuGet [Microsoft. Extensions. Caching. memory](https://www.nuget.org/packages/Microsoft.Extensions.Caching.Memory/), disponibile nel [metapacchetto Microsoft. AspNetCore. app](xref:fundamentals/metapackage-app).

Il codice seguente usa [TryGetValue](/dotnet/api/microsoft.extensions.caching.memory.imemorycache.trygetvalue?view=aspnetcore-2.0#Microsoft_Extensions_Caching_Memory_IMemoryCache_TryGetValue_System_Object_System_Object__) per verificare se un'ora è nella cache. Se l'ora non viene memorizzata nella cache, viene creata una nuova voce che viene aggiunta alla cache con [set](/dotnet/api/microsoft.extensions.caching.memory.cacheextensions.set?view=aspnetcore-2.0#Microsoft_Extensions_Caching_Memory_CacheExtensions_Set__1_Microsoft_Extensions_Caching_Memory_IMemoryCache_System_Object___0_Microsoft_Extensions_Caching_Memory_MemoryCacheEntryOptions_).

[!code-csharp[](memory/sample/WebCache/CacheKeys.cs)]

[!code-csharp[](memory/sample/WebCache/Controllers/HomeController.cs?name=snippet1)]

Vengono visualizzati l'ora corrente e l'ora memorizzata nella cache:

[!code-cshtml[](memory/sample/WebCache/Views/Home/Cache.cshtml)]

Il valore memorizzato nella `DateTime` cache rimane nella cache mentre sono presenti richieste entro il periodo di timeout. La figura seguente mostra l'ora corrente e un tempo precedente recuperato dalla cache:

![Visualizzazione indice con due ore diverse visualizzate](memory/_static/time.png)

Il codice seguente usa [GetOrCreate](/dotnet/api/microsoft.extensions.caching.memory.cacheextensions.getorcreate#Microsoft_Extensions_Caching_Memory_CacheExtensions_GetOrCreate__1_Microsoft_Extensions_Caching_Memory_IMemoryCache_System_Object_System_Func_Microsoft_Extensions_Caching_Memory_ICacheEntry___0__) e [GetOrCreateAsync](/dotnet/api/microsoft.extensions.caching.memory.cacheextensions.getorcreateasync#Microsoft_Extensions_Caching_Memory_CacheExtensions_GetOrCreateAsync__1_Microsoft_Extensions_Caching_Memory_IMemoryCache_System_Object_System_Func_Microsoft_Extensions_Caching_Memory_ICacheEntry_System_Threading_Tasks_Task___0___) per memorizzare i dati nella cache.

[!code-csharp[](memory/sample/WebCache/Controllers/HomeController.cs?name=snippet2&highlight=3-7,14-19)]

Il codice seguente chiama [Get](/dotnet/api/microsoft.extensions.caching.memory.cacheextensions.get#Microsoft_Extensions_Caching_Memory_CacheExtensions_Get__1_Microsoft_Extensions_Caching_Memory_IMemoryCache_System_Object_) per recuperare l'ora memorizzata nella cache:

[!code-csharp[](memory/sample/WebCache/Controllers/HomeController.cs?name=snippet_gct)]

<xref:Microsoft.Extensions.Caching.Memory.CacheExtensions.GetOrCreate*> , <xref:Microsoft.Extensions.Caching.Memory.CacheExtensions.GetOrCreateAsync*> e [Get](/dotnet/api/microsoft.extensions.caching.memory.cacheextensions.get#Microsoft_Extensions_Caching_Memory_CacheExtensions_Get__1_Microsoft_Extensions_Caching_Memory_IMemoryCache_System_Object_) sono metodi di estensione parte della classe [CacheExtensions](/dotnet/api/microsoft.extensions.caching.memory.cacheextensions) che estende la funzionalità di <xref:Microsoft.Extensions.Caching.Memory.IMemoryCache> . Vedere [Metodi IMemoryCache](/dotnet/api/microsoft.extensions.caching.memory.imemorycache) e [Metodi CacheExtensions](/dotnet/api/microsoft.extensions.caching.memory.cacheextensions) per una descrizione di altri metodi della cache.

## <a name="memorycacheentryoptions"></a>MemoryCacheEntryOptions

L'esempio seguente:

* Imposta una data di scadenza variabile. Le richieste che accedono a questo elemento memorizzato nella cache reimpostano il clock di scadenza variabile.
* Imposta la priorità della cache su `CacheItemPriority.NeverRemove` .
* Imposta un [PostEvictionDelegate](/dotnet/api/microsoft.extensions.caching.memory.postevictiondelegate) che verrà chiamato dopo la rimozione della voce dalla cache. Il callback viene eseguito su un thread diverso dal codice che rimuove l'elemento dalla cache.

[!code-csharp[](memory/sample/WebCache/Controllers/HomeController.cs?name=snippet_et&highlight=14-21)]

## <a name="use-setsize-size-and-sizelimit-to-limit-cache-size"></a>Usare le dimensioni di ridimensionamento, dimensioni e SizeLimit per limitare le dimensioni della cache

Un' `MemoryCache` istanza può facoltativamente specificare e applicare un limite di dimensioni. Il limite delle dimensioni della cache non dispone di un'unità di misura definita perché la cache non ha alcun meccanismo per misurare la dimensione delle voci. Se viene impostato il limite delle dimensioni della cache, è necessario che tutte le voci specifichino la dimensione. Il runtime di ASP.NET Core non limita le dimensioni della cache in base all'utilizzo eccessivo della memoria. Spetta allo sviluppatore limitare le dimensioni della cache. Le dimensioni specificate sono in unità scelte dallo sviluppatore.

Ad esempio:

* Se l'app Web è stata utilizzata principalmente per la memorizzazione nella cache delle stringhe, le dimensioni della voce della cache potrebbero essere la lunghezza della stringa.
* L'app può specificare le dimensioni di tutte le voci come 1 e il limite di dimensioni è il numero di voci.

Se <xref:Microsoft.Extensions.Caching.Memory.MemoryCacheOptions.SizeLimit> non è impostato, la cache cresce senza alcun limite. Il runtime di ASP.NET Core non elimina la cache quando la memoria di sistema è insufficiente. Le app sono molto progettate per:

* Limitare la crescita della cache.
* Chiamare <xref:Microsoft.Extensions.Caching.Memory.MemoryCache.Compact*> o <xref:Microsoft.Extensions.Caching.Memory.MemoryCache.Remove*> quando la memoria disponibile è limitata:

Il codice seguente consente di creare una dimensione fissa unificata <xref:Microsoft.Extensions.Caching.Memory.MemoryCache> accessibile dall' [inserimento delle dipendenze](xref:fundamentals/dependency-injection):

[!code-csharp[](memory/sample/RPcache/Services/MyMemoryCache.cs?name=snippet)]

`SizeLimit` non dispone di unità. Le voci memorizzate nella cache devono specificare le dimensioni in qualsiasi unità ritenute più appropriate se è stato impostato il limite delle dimensioni della cache. Tutti gli utenti di un'istanza della cache devono usare lo stesso sistema di unità. Una voce non verrà memorizzata nella cache se la somma delle dimensioni della voce memorizzata nella cache supera il valore specificato da `SizeLimit` . Se non è impostato alcun limite per le dimensioni della cache, le dimensioni della cache impostate per la voce verranno ignorate.

Il codice seguente esegue la registrazione `MyMemoryCache` con il contenitore di [inserimento delle dipendenze](xref:fundamentals/dependency-injection) .

[!code-csharp[](memory/sample/RPcache/Startup.cs?name=snippet&highlight=5)]

`MyMemoryCache` viene creato come cache di memoria indipendente per i componenti che sono consapevoli di questa dimensione limitata della cache e sanno come impostare la dimensione della voce della cache in modo appropriato.

Il codice seguente usa `MyMemoryCache` :

[!code-csharp[](memory/sample/RPcache/Pages/About.cshtml.cs?name=snippet)]

La dimensione della voce della cache può essere impostata in base alle [dimensioni](/dotnet/api/microsoft.extensions.caching.memory.memorycacheentryoptions.size?view=aspnetcore-2.1#Microsoft_Extensions_Caching_Memory_MemoryCacheEntryOptions_Size) o al metodo di estensione [sesize](/dotnet/api/microsoft.extensions.caching.memory.memorycacheentryextensions.setsize?view=aspnetcore-2.0#Microsoft_Extensions_Caching_Memory_MemoryCacheEntryExtensions_SetSize_Microsoft_Extensions_Caching_Memory_MemoryCacheEntryOptions_System_Int64_) :

[!code-csharp[](memory/sample/RPcache/Pages/About.cshtml.cs?name=snippet2&highlight=9,10,14,15)]

### <a name="memorycachecompact"></a>MemoryCache. Compact

`MemoryCache.Compact` tenta di rimuovere la percentuale specificata della cache nell'ordine seguente:

* Tutti gli elementi scaduti.
* Elementi per priorità. Gli elementi con priorità più bassa vengono rimossi per primi.
* Oggetti utilizzati meno di recente.
* Elementi con la prima scadenza assoluta.
* Elementi con la scadenza variabile meno recente.

Gli elementi aggiunti con priorità <xref:Microsoft.Extensions.Caching.Memory.CacheItemPriority.NeverRemove> non vengono mai rimossi.

[!code-csharp[](memory/3.0sample/RPcache/Pages/TestCache.cshtml.cs?name=snippet3)]

Per altre informazioni, vedere [Compact Source su GitHub](https://github.com/dotnet/extensions/blob/v3.0.0-preview8.19405.4/src/Caching/Memory/src/MemoryCache.cs#L382-L393) .

## <a name="cache-dependencies"></a>Dipendenze della cache

Nell'esempio seguente viene illustrato come impostare come scaduti una voce della cache in caso di scadenza di una voce dipendente. <xref:Microsoft.Extensions.Primitives.CancellationChangeToken>Viene aggiunto un oggetto all'elemento memorizzato nella cache. Quando `Cancel` viene chiamato su `CancellationTokenSource` , entrambe le voci della cache vengono eliminate.

[!code-csharp[](memory/sample/WebCache/Controllers/HomeController.cs?name=snippet_ed)]

L'uso di un oggetto `CancellationTokenSource` consente di rimuovere più voci della cache come gruppo. Con il `using` modello nel codice precedente, le voci della cache create all'interno del `using` blocco erediteranno i trigger e le impostazioni di scadenza.

## <a name="additional-notes"></a>Note aggiuntive

* Quando si usa un callback per ripopolare un elemento della cache:

  * Più richieste possono trovare il valore della chiave memorizzato nella cache perché il callback non è stato completato.
  * Questo può comportare la ricompilazione dell'elemento memorizzato nella cache da parte di più thread.

* Quando viene utilizzata una voce della cache per crearne un'altra, l'elemento figlio copia i token di scadenza della voce padre e le impostazioni di scadenza basate sul tempo. L'elemento figlio non è scaduto dalla rimozione manuale o dall'aggiornamento della voce padre.

* Utilizzare [PostEvictionCallbacks](/dotnet/api/microsoft.extensions.caching.memory.icacheentry.postevictioncallbacks#Microsoft_Extensions_Caching_Memory_ICacheEntry_PostEvictionCallbacks) per impostare i callback che verranno generati dopo la rimozione della voce della cache dalla cache.

## <a name="background-cache-update"></a>Aggiornamento della cache in background

Usare un [servizio in background](xref:fundamentals/host/hosted-services) , ad esempio <xref:Microsoft.Extensions.Hosting.IHostedService> per aggiornare la cache. Il servizio in background può ricalcolare le voci e quindi assegnarle alla cache solo quando sono pronte.

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:performance/caching/distributed>
* <xref:fundamentals/change-tokens>
* <xref:performance/caching/response>
* <xref:performance/caching/middleware>
* <xref:mvc/views/tag-helpers/builtin-th/cache-tag-helper>
* <xref:mvc/views/tag-helpers/builtin-th/distributed-cache-tag-helper>

::: moniker-end
