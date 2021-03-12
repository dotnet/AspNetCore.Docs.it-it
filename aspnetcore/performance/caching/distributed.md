---
title: Caching distribuito in ASP.NET Core
author: rick-anderson
description: Informazioni su come usare una cache distribuita ASP.NET Core per migliorare le prestazioni e la scalabilità delle app, soprattutto in un ambiente cloud o server farm.
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 02/07/2020
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
uid: performance/caching/distributed
ms.openlocfilehash: a4fd179772a26ffd20fa79cef4720cd5a4746ab3
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102588614"
---
# <a name="distributed-caching-in-aspnet-core"></a><span data-ttu-id="d6d95-103">Caching distribuito in ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="d6d95-103">Distributed caching in ASP.NET Core</span></span>

<span data-ttu-id="d6d95-104">Di e di Stevie [Nasir](https://github.com/mohsinnasir) e [Steve Smith](https://ardalis.com/)</span><span class="sxs-lookup"><span data-stu-id="d6d95-104">By [Mohsin Nasir](https://github.com/mohsinnasir) and [Steve Smith](https://ardalis.com/)</span></span>

::: moniker range=">= aspnetcore-3.0"

<span data-ttu-id="d6d95-105">Una cache distribuita è una cache condivisa da più server app, in genere gestita come servizio esterno ai server app che vi accedono.</span><span class="sxs-lookup"><span data-stu-id="d6d95-105">A distributed cache is a cache shared by multiple app servers, typically maintained as an external service to the app servers that access it.</span></span> <span data-ttu-id="d6d95-106">Una cache distribuita può migliorare le prestazioni e la scalabilità di un'app ASP.NET Core, soprattutto quando l'app è ospitata da un servizio cloud o da un server farm.</span><span class="sxs-lookup"><span data-stu-id="d6d95-106">A distributed cache can improve the performance and scalability of an ASP.NET Core app, especially when the app is hosted by a cloud service or a server farm.</span></span>

<span data-ttu-id="d6d95-107">Una cache distribuita presenta diversi vantaggi rispetto ad altri scenari di memorizzazione nella cache in cui i dati memorizzati nella cache vengono archiviati nei singoli server dell'app.</span><span class="sxs-lookup"><span data-stu-id="d6d95-107">A distributed cache has several advantages over other caching scenarios where cached data is stored on individual app servers.</span></span>

<span data-ttu-id="d6d95-108">Quando vengono distribuiti i dati memorizzati nella cache, i dati:</span><span class="sxs-lookup"><span data-stu-id="d6d95-108">When cached data is distributed, the data:</span></span>

* <span data-ttu-id="d6d95-109">È *coerente* (coerente) tra le richieste a più server.</span><span class="sxs-lookup"><span data-stu-id="d6d95-109">Is *coherent* (consistent) across requests to multiple servers.</span></span>
* <span data-ttu-id="d6d95-110">Sopravvive ai riavvii del server e alle distribuzioni di app.</span><span class="sxs-lookup"><span data-stu-id="d6d95-110">Survives server restarts and app deployments.</span></span>
* <span data-ttu-id="d6d95-111">Non usa la memoria locale.</span><span class="sxs-lookup"><span data-stu-id="d6d95-111">Doesn't use local memory.</span></span>

<span data-ttu-id="d6d95-112">La configurazione della cache distribuita è specifica dell'implementazione.</span><span class="sxs-lookup"><span data-stu-id="d6d95-112">Distributed cache configuration is implementation specific.</span></span> <span data-ttu-id="d6d95-113">Questo articolo descrive come configurare le cache distribuite SQL Server e Redis.</span><span class="sxs-lookup"><span data-stu-id="d6d95-113">This article describes how to configure SQL Server and Redis distributed caches.</span></span> <span data-ttu-id="d6d95-114">Sono disponibili anche implementazioni di terze parti, ad esempio [NCache](http://www.alachisoft.com/ncache/aspnet-core-idistributedcache-ncache.html) ([NCache su GitHub](https://github.com/Alachisoft/NCache)).</span><span class="sxs-lookup"><span data-stu-id="d6d95-114">Third party implementations are also available, such as [NCache](http://www.alachisoft.com/ncache/aspnet-core-idistributedcache-ncache.html) ([NCache on GitHub](https://github.com/Alachisoft/NCache)).</span></span> <span data-ttu-id="d6d95-115">Indipendentemente dall'implementazione selezionata, l'app interagisce con la cache usando l' <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> interfaccia.</span><span class="sxs-lookup"><span data-stu-id="d6d95-115">Regardless of which implementation is selected, the app interacts with the cache using the <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> interface.</span></span>

<span data-ttu-id="d6d95-116">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/performance/caching/distributed/samples/) ([procedura per il download](xref:index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="d6d95-116">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/performance/caching/distributed/samples/) ([how to download](xref:index#how-to-download-a-sample))</span></span>

## <a name="prerequisites"></a><span data-ttu-id="d6d95-117">Prerequisiti</span><span class="sxs-lookup"><span data-stu-id="d6d95-117">Prerequisites</span></span>

<span data-ttu-id="d6d95-118">Per utilizzare una SQL Server cache distribuita, aggiungere un riferimento al pacchetto [Microsoft. Extensions. Caching. SqlServer](https://www.nuget.org/packages/Microsoft.Extensions.Caching.SqlServer) .</span><span class="sxs-lookup"><span data-stu-id="d6d95-118">To use a SQL Server distributed cache, add a package reference to the [Microsoft.Extensions.Caching.SqlServer](https://www.nuget.org/packages/Microsoft.Extensions.Caching.SqlServer) package.</span></span>

<span data-ttu-id="d6d95-119">Per usare una cache distribuita Redis, aggiungere un riferimento al pacchetto [Microsoft. Extensions. Caching. StackExchangeRedis](https://www.nuget.org/packages/Microsoft.Extensions.Caching.StackExchangeRedis) .</span><span class="sxs-lookup"><span data-stu-id="d6d95-119">To use a Redis distributed cache, add a package reference to the [Microsoft.Extensions.Caching.StackExchangeRedis](https://www.nuget.org/packages/Microsoft.Extensions.Caching.StackExchangeRedis) package.</span></span>

<span data-ttu-id="d6d95-120">Per usare la cache distribuita NCache, aggiungere un riferimento al pacchetto [NCache. Microsoft. Extensions. Caching. opensource](https://www.nuget.org/packages/NCache.Microsoft.Extensions.Caching.OpenSource) .</span><span class="sxs-lookup"><span data-stu-id="d6d95-120">To use NCache distributed cache, add a package reference to the [NCache.Microsoft.Extensions.Caching.OpenSource](https://www.nuget.org/packages/NCache.Microsoft.Extensions.Caching.OpenSource) package.</span></span>

## <a name="idistributedcache-interface"></a><span data-ttu-id="d6d95-121">Interfaccia IDistributedCache</span><span class="sxs-lookup"><span data-stu-id="d6d95-121">IDistributedCache interface</span></span>

<span data-ttu-id="d6d95-122">L' <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> interfaccia fornisce i metodi seguenti per modificare gli elementi nell'implementazione della cache distribuita:</span><span class="sxs-lookup"><span data-stu-id="d6d95-122">The <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> interface provides the following methods to manipulate items in the distributed cache implementation:</span></span>

* <span data-ttu-id="d6d95-123"><xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache.Get*>, <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache.GetAsync*> : Accetta una chiave di stringa e recupera un elemento memorizzato nella cache come `byte[]` matrice se viene trovato nella cache.</span><span class="sxs-lookup"><span data-stu-id="d6d95-123"><xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache.Get*>, <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache.GetAsync*>: Accepts a string key and retrieves a cached item as a `byte[]` array if found in the cache.</span></span>
* <span data-ttu-id="d6d95-124"><xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache.Set*>, <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache.SetAsync*> : Aggiunge un elemento (come `byte[]` matrice) alla cache usando una chiave di stringa.</span><span class="sxs-lookup"><span data-stu-id="d6d95-124"><xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache.Set*>, <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache.SetAsync*>: Adds an item (as `byte[]` array) to the cache using a string key.</span></span>
* <span data-ttu-id="d6d95-125"><xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache.Refresh*>, <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache.RefreshAsync*> : Aggiorna un elemento nella cache in base alla relativa chiave, reimpostando il timeout di scadenza variabile (se presente).</span><span class="sxs-lookup"><span data-stu-id="d6d95-125"><xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache.Refresh*>, <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache.RefreshAsync*>: Refreshes an item in the cache based on its key, resetting its sliding expiration timeout (if any).</span></span>
* <span data-ttu-id="d6d95-126"><xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache.Remove*>, <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache.RemoveAsync*> : Rimuove un elemento della cache in base alla relativa chiave di stringa.</span><span class="sxs-lookup"><span data-stu-id="d6d95-126"><xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache.Remove*>, <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache.RemoveAsync*>: Removes a cache item based on its string key.</span></span>

## <a name="establish-distributed-caching-services"></a><span data-ttu-id="d6d95-127">Stabilire servizi di memorizzazione nella cache distribuiti</span><span class="sxs-lookup"><span data-stu-id="d6d95-127">Establish distributed caching services</span></span>

<span data-ttu-id="d6d95-128">Registrare un'implementazione di <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> in `Startup.ConfigureServices` .</span><span class="sxs-lookup"><span data-stu-id="d6d95-128">Register an implementation of <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> in `Startup.ConfigureServices`.</span></span> <span data-ttu-id="d6d95-129">Le implementazioni fornite dal Framework descritte in questo argomento includono:</span><span class="sxs-lookup"><span data-stu-id="d6d95-129">Framework-provided implementations described in this topic include:</span></span>

* [<span data-ttu-id="d6d95-130">Cache di memoria distribuita</span><span class="sxs-lookup"><span data-stu-id="d6d95-130">Distributed Memory Cache</span></span>](#distributed-memory-cache)
* [<span data-ttu-id="d6d95-131">Cache SQL Server distribuita</span><span class="sxs-lookup"><span data-stu-id="d6d95-131">Distributed SQL Server cache</span></span>](#distributed-sql-server-cache)
* [<span data-ttu-id="d6d95-132">Cache Redis distribuita</span><span class="sxs-lookup"><span data-stu-id="d6d95-132">Distributed Redis cache</span></span>](#distributed-redis-cache)
* [<span data-ttu-id="d6d95-133">Cache NCache distribuita</span><span class="sxs-lookup"><span data-stu-id="d6d95-133">Distributed NCache cache</span></span>](#distributed-ncache-cache)

### <a name="distributed-memory-cache"></a><span data-ttu-id="d6d95-134">Cache di memoria distribuita</span><span class="sxs-lookup"><span data-stu-id="d6d95-134">Distributed Memory Cache</span></span>

<span data-ttu-id="d6d95-135">La cache di memoria distribuita ( <xref:Microsoft.Extensions.DependencyInjection.MemoryCacheServiceCollectionExtensions.AddDistributedMemoryCache*> ) è un'implementazione fornita dal framework di <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> che archivia gli elementi in memoria.</span><span class="sxs-lookup"><span data-stu-id="d6d95-135">The Distributed Memory Cache (<xref:Microsoft.Extensions.DependencyInjection.MemoryCacheServiceCollectionExtensions.AddDistributedMemoryCache*>) is a framework-provided implementation of <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> that stores items in memory.</span></span> <span data-ttu-id="d6d95-136">La cache di memoria distribuita non è una cache distribuita effettiva.</span><span class="sxs-lookup"><span data-stu-id="d6d95-136">The Distributed Memory Cache isn't an actual distributed cache.</span></span> <span data-ttu-id="d6d95-137">Gli elementi memorizzati nella cache vengono archiviati dall'istanza dell'app nel server in cui è in esecuzione l'app.</span><span class="sxs-lookup"><span data-stu-id="d6d95-137">Cached items are stored by the app instance on the server where the app is running.</span></span>

<span data-ttu-id="d6d95-138">La cache di memoria distribuita è un'implementazione utile:</span><span class="sxs-lookup"><span data-stu-id="d6d95-138">The Distributed Memory Cache is a useful implementation:</span></span>

* <span data-ttu-id="d6d95-139">Negli scenari di sviluppo e test.</span><span class="sxs-lookup"><span data-stu-id="d6d95-139">In development and testing scenarios.</span></span>
* <span data-ttu-id="d6d95-140">Quando un singolo server viene utilizzato nell'ambiente di produzione e l'utilizzo della memoria non costituisce un problema.</span><span class="sxs-lookup"><span data-stu-id="d6d95-140">When a single server is used in production and memory consumption isn't an issue.</span></span> <span data-ttu-id="d6d95-141">L'implementazione della cache di memoria distribuita astrae l'archiviazione dei dati memorizzati nella cache.</span><span class="sxs-lookup"><span data-stu-id="d6d95-141">Implementing the Distributed Memory Cache abstracts cached data storage.</span></span> <span data-ttu-id="d6d95-142">Consente di implementare in futuro una soluzione di memorizzazione nella cache distribuita se più nodi o tolleranza di errore diventano necessari.</span><span class="sxs-lookup"><span data-stu-id="d6d95-142">It allows for implementing a true distributed caching solution in the future if multiple nodes or fault tolerance become necessary.</span></span>

<span data-ttu-id="d6d95-143">L'app di esempio usa la cache di memoria distribuita quando l'app viene eseguita nell'ambiente di sviluppo in `Startup.ConfigureServices` :</span><span class="sxs-lookup"><span data-stu-id="d6d95-143">The sample app makes use of the Distributed Memory Cache when the app is run in the Development environment in `Startup.ConfigureServices`:</span></span>

[!code-csharp[](distributed/samples/3.x/DistCacheSample/Startup.cs?name=snippet_AddDistributedMemoryCache)]

### <a name="distributed-sql-server-cache"></a><span data-ttu-id="d6d95-144">Cache SQL Server distribuita</span><span class="sxs-lookup"><span data-stu-id="d6d95-144">Distributed SQL Server Cache</span></span>

<span data-ttu-id="d6d95-145">L'implementazione della cache SQL Server distribuita ( <xref:Microsoft.Extensions.DependencyInjection.SqlServerCachingServicesExtensions.AddDistributedSqlServerCache*> ) consente alla cache distribuita di usare un database SQL Server come archivio di backup.</span><span class="sxs-lookup"><span data-stu-id="d6d95-145">The Distributed SQL Server Cache implementation (<xref:Microsoft.Extensions.DependencyInjection.SqlServerCachingServicesExtensions.AddDistributedSqlServerCache*>) allows the distributed cache to use a SQL Server database as its backing store.</span></span> <span data-ttu-id="d6d95-146">Per creare una tabella SQL Server elemento memorizzato nella cache in un'istanza di SQL Server, è possibile utilizzare lo `sql-cache` strumento.</span><span class="sxs-lookup"><span data-stu-id="d6d95-146">To create a SQL Server cached item table in a SQL Server instance, you can use the `sql-cache` tool.</span></span> <span data-ttu-id="d6d95-147">Lo strumento crea una tabella con il nome e lo schema specificati.</span><span class="sxs-lookup"><span data-stu-id="d6d95-147">The tool creates a table with the name and schema that you specify.</span></span>

<span data-ttu-id="d6d95-148">Creare una tabella in SQL Server eseguendo il `sql-cache create` comando.</span><span class="sxs-lookup"><span data-stu-id="d6d95-148">Create a table in SQL Server by running the `sql-cache create` command.</span></span> <span data-ttu-id="d6d95-149">Fornire l'istanza di SQL Server ( `Data Source` ), il database ( `Initial Catalog` ), lo schema (ad esempio, `dbo` ) e il nome della tabella (ad esempio, `TestCache` ):</span><span class="sxs-lookup"><span data-stu-id="d6d95-149">Provide the SQL Server instance (`Data Source`), database (`Initial Catalog`), schema (for example, `dbo`), and table name (for example, `TestCache`):</span></span>

```dotnetcli
dotnet sql-cache create "Data Source=(localdb)\MSSQLLocalDB;Initial Catalog=DistCache;Integrated Security=True;" dbo TestCache
```

<span data-ttu-id="d6d95-150">Viene registrato un messaggio per indicare che lo strumento è stato completato correttamente:</span><span class="sxs-lookup"><span data-stu-id="d6d95-150">A message is logged to indicate that the tool was successful:</span></span>

```console
Table and index were created successfully.
```

<span data-ttu-id="d6d95-151">La tabella creata dallo `sql-cache` strumento presenta lo schema seguente:</span><span class="sxs-lookup"><span data-stu-id="d6d95-151">The table created by the `sql-cache` tool has the following schema:</span></span>

![Tabella cache SqlServer](distributed/_static/SqlServerCacheTable.png)

> [!NOTE]
> <span data-ttu-id="d6d95-153">Un'app deve modificare i valori della cache usando un'istanza di <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> , non un oggetto <xref:Microsoft.Extensions.Caching.SqlServer.SqlServerCache> .</span><span class="sxs-lookup"><span data-stu-id="d6d95-153">An app should manipulate cache values using an instance of <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache>, not a <xref:Microsoft.Extensions.Caching.SqlServer.SqlServerCache>.</span></span>

<span data-ttu-id="d6d95-154">L'app di esempio implementa <xref:Microsoft.Extensions.Caching.SqlServer.SqlServerCache> in un ambiente non di sviluppo in `Startup.ConfigureServices` :</span><span class="sxs-lookup"><span data-stu-id="d6d95-154">The sample app implements <xref:Microsoft.Extensions.Caching.SqlServer.SqlServerCache> in a non-Development environment in `Startup.ConfigureServices`:</span></span>

[!code-csharp[](distributed/samples/3.x/DistCacheSample/Startup.cs?name=snippet_AddDistributedSqlServerCache)]

> [!NOTE]
> <span data-ttu-id="d6d95-155">Un <xref:Microsoft.Extensions.Caching.SqlServer.SqlServerCacheOptions.ConnectionString*> (e, facoltativamente, <xref:Microsoft.Extensions.Caching.SqlServer.SqlServerCacheOptions.SchemaName*> e <xref:Microsoft.Extensions.Caching.SqlServer.SqlServerCacheOptions.TableName*> ) vengono in genere archiviati all'esterno del controllo del codice sorgente, ad esempio archiviato dal [gestore dei segreti](xref:security/app-secrets) o in *appsettings.json* / *appSettings. { ENVIRONMENT} file JSON* ).</span><span class="sxs-lookup"><span data-stu-id="d6d95-155">A <xref:Microsoft.Extensions.Caching.SqlServer.SqlServerCacheOptions.ConnectionString*> (and optionally, <xref:Microsoft.Extensions.Caching.SqlServer.SqlServerCacheOptions.SchemaName*> and <xref:Microsoft.Extensions.Caching.SqlServer.SqlServerCacheOptions.TableName*>) are typically stored outside of source control (for example, stored by the [Secret Manager](xref:security/app-secrets) or in *appsettings.json*/*appsettings.{ENVIRONMENT}.json* files).</span></span> <span data-ttu-id="d6d95-156">La stringa di connessione può contenere credenziali che devono essere mantenute fuori dai sistemi di controllo del codice sorgente.</span><span class="sxs-lookup"><span data-stu-id="d6d95-156">The connection string may contain credentials that should be kept out of source control systems.</span></span>

### <a name="distributed-redis-cache"></a><span data-ttu-id="d6d95-157">Cache Redis distribuita</span><span class="sxs-lookup"><span data-stu-id="d6d95-157">Distributed Redis Cache</span></span>

<span data-ttu-id="d6d95-158">[Redis](https://redis.io/) è un archivio dati in memoria open source, che viene spesso usato come cache distribuita.</span><span class="sxs-lookup"><span data-stu-id="d6d95-158">[Redis](https://redis.io/) is an open source in-memory data store, which is often used as a distributed cache.</span></span>  <span data-ttu-id="d6d95-159">È possibile configurare una [cache Redis di Azure](https://azure.microsoft.com/services/cache/) per un'app ASP.NET Core ospitata in Azure e usare una cache Redis di Azure per lo sviluppo locale.</span><span class="sxs-lookup"><span data-stu-id="d6d95-159">You can configure an [Azure Redis Cache](https://azure.microsoft.com/services/cache/) for an Azure-hosted ASP.NET Core app, and use an Azure Redis Cache for local development.</span></span>

<span data-ttu-id="d6d95-160">Un'app configura l'implementazione della cache usando un' <xref:Microsoft.Extensions.Caching.StackExchangeRedis.RedisCache> istanza di ( <xref:Microsoft.Extensions.DependencyInjection.StackExchangeRedisCacheServiceCollectionExtensions.AddStackExchangeRedisCache*> ).</span><span class="sxs-lookup"><span data-stu-id="d6d95-160">An app configures the cache implementation using a <xref:Microsoft.Extensions.Caching.StackExchangeRedis.RedisCache> instance (<xref:Microsoft.Extensions.DependencyInjection.StackExchangeRedisCacheServiceCollectionExtensions.AddStackExchangeRedisCache*>).</span></span>

<span data-ttu-id="d6d95-161">Per altre informazioni, vedere [Azure Cache for Redis](/azure/azure-cache-for-redis/cache-overview).</span><span class="sxs-lookup"><span data-stu-id="d6d95-161">For more information, see [Azure Cache for Redis](/azure/azure-cache-for-redis/cache-overview).</span></span>

<span data-ttu-id="d6d95-162">Vedere [questo problema di GitHub](https://github.com/dotnet/AspNetCore.Docs/issues/19542) per una discussione sugli approcci alternativi a una cache Redis locale.</span><span class="sxs-lookup"><span data-stu-id="d6d95-162">See [this GitHub issue](https://github.com/dotnet/AspNetCore.Docs/issues/19542) for a discussion on alternative approaches to a local Redis cache.</span></span>

### <a name="distributed-ncache-cache"></a><span data-ttu-id="d6d95-163">Cache NCache distribuita</span><span class="sxs-lookup"><span data-stu-id="d6d95-163">Distributed NCache Cache</span></span>

<span data-ttu-id="d6d95-164">[NCache](https://github.com/Alachisoft/NCache) è una cache distribuita in memoria open source sviluppata in modo nativo in .NET e .NET Core.</span><span class="sxs-lookup"><span data-stu-id="d6d95-164">[NCache](https://github.com/Alachisoft/NCache) is an open source in-memory distributed cache developed natively in .NET and .NET Core.</span></span> <span data-ttu-id="d6d95-165">NCache funziona sia localmente che configurato come cluster di cache distribuita per un'app ASP.NET Core in esecuzione in Azure o in altre piattaforme di hosting.</span><span class="sxs-lookup"><span data-stu-id="d6d95-165">NCache works both locally and configured as a distributed cache cluster for an ASP.NET Core app running in Azure or on other hosting platforms.</span></span>

<span data-ttu-id="d6d95-166">Per installare e configurare NCache nel computer locale, vedere la [guida introduzione per Windows (.NET e .NET Core)](https://www.alachisoft.com/resources/docs/ncache/getting-started-guide-windows/).</span><span class="sxs-lookup"><span data-stu-id="d6d95-166">To install and configure NCache on your local machine, see [Getting Started Guide for Windows (.NET and .NET Core)](https://www.alachisoft.com/resources/docs/ncache/getting-started-guide-windows/).</span></span>

<span data-ttu-id="d6d95-167">Per configurare NCache:</span><span class="sxs-lookup"><span data-stu-id="d6d95-167">To configure NCache:</span></span>

1. <span data-ttu-id="d6d95-168">Installare [NCache Open Source NuGet](https://www.nuget.org/packages/Alachisoft.NCache.OpenSource.SDK/).</span><span class="sxs-lookup"><span data-stu-id="d6d95-168">Install [NCache open source NuGet](https://www.nuget.org/packages/Alachisoft.NCache.OpenSource.SDK/).</span></span>
1. <span data-ttu-id="d6d95-169">Configurare il cluster di cache in [client. ncconf](https://www.alachisoft.com/resources/docs/ncache/admin-guide/client-config.html).</span><span class="sxs-lookup"><span data-stu-id="d6d95-169">Configure the cache cluster in [client.ncconf](https://www.alachisoft.com/resources/docs/ncache/admin-guide/client-config.html).</span></span>
1. <span data-ttu-id="d6d95-170">Aggiungere il codice seguente a `Startup.ConfigureServices`:</span><span class="sxs-lookup"><span data-stu-id="d6d95-170">Add the following code to `Startup.ConfigureServices`:</span></span>

   ```csharp
   services.AddNCacheDistributedCache(configuration =>    
   {        
       configuration.CacheName = "demoClusteredCache";
       configuration.EnableLogs = true;
       configuration.ExceptionsEnabled = true;
   });
   ```

## <a name="use-the-distributed-cache"></a><span data-ttu-id="d6d95-171">Usare la cache distribuita</span><span class="sxs-lookup"><span data-stu-id="d6d95-171">Use the distributed cache</span></span>

<span data-ttu-id="d6d95-172">Per usare l' <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> interfaccia, richiedere un'istanza di <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> da qualsiasi costruttore nell'app.</span><span class="sxs-lookup"><span data-stu-id="d6d95-172">To use the <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> interface, request an instance of <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> from any constructor in the app.</span></span> <span data-ttu-id="d6d95-173">L'istanza viene fornita da [inserimento delle dipendenze (di)](xref:fundamentals/dependency-injection).</span><span class="sxs-lookup"><span data-stu-id="d6d95-173">The instance is provided by [dependency injection (DI)](xref:fundamentals/dependency-injection).</span></span>

<span data-ttu-id="d6d95-174">Quando l'app di esempio viene avviata, <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> viene inserita in `Startup.Configure` .</span><span class="sxs-lookup"><span data-stu-id="d6d95-174">When the sample app starts, <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> is injected into `Startup.Configure`.</span></span> <span data-ttu-id="d6d95-175">L'ora corrente viene memorizzata nella cache usando <xref:Microsoft.Extensions.Hosting.IHostApplicationLifetime> (per altre informazioni, vedere [host generico: IHostApplicationLifetime](xref:fundamentals/host/generic-host#ihostapplicationlifetime)):</span><span class="sxs-lookup"><span data-stu-id="d6d95-175">The current time is cached using <xref:Microsoft.Extensions.Hosting.IHostApplicationLifetime> (for more information, see [Generic Host: IHostApplicationLifetime](xref:fundamentals/host/generic-host#ihostapplicationlifetime)):</span></span>

[!code-csharp[](distributed/samples/3.x/DistCacheSample/Startup.cs?name=snippet_Configure&highlight=10)]

<span data-ttu-id="d6d95-176">L'app di esempio inserisce <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> nell'oggetto `IndexModel` per l'uso da parte della pagina di indice.</span><span class="sxs-lookup"><span data-stu-id="d6d95-176">The sample app injects <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> into the `IndexModel` for use by the Index page.</span></span>

<span data-ttu-id="d6d95-177">Ogni volta che viene caricata la pagina di indice, viene verificata la presenza dell'ora memorizzata nella cache in `OnGetAsync` .</span><span class="sxs-lookup"><span data-stu-id="d6d95-177">Each time the Index page is loaded, the cache is checked for the cached time in `OnGetAsync`.</span></span> <span data-ttu-id="d6d95-178">Se il tempo di memorizzazione nella cache non è scaduto, viene visualizzata l'ora.</span><span class="sxs-lookup"><span data-stu-id="d6d95-178">If the cached time hasn't expired, the time is displayed.</span></span> <span data-ttu-id="d6d95-179">Se sono trascorsi 20 secondi dall'ultima volta in cui è stato eseguito l'accesso all'ora memorizzata nella cache (l'ultima volta che questa pagina è stata caricata), la pagina Visualizza l' *ora della cache scaduta*.</span><span class="sxs-lookup"><span data-stu-id="d6d95-179">If 20 seconds have elapsed since the last time the cached time was accessed (the last time this page was loaded), the page displays *Cached Time Expired*.</span></span>

<span data-ttu-id="d6d95-180">Aggiornare immediatamente l'ora memorizzata nella cache all'ora corrente selezionando il pulsante **Reimposta tempo memorizzato nella cache** .</span><span class="sxs-lookup"><span data-stu-id="d6d95-180">Immediately update the cached time to the current time by selecting the **Reset Cached Time** button.</span></span> <span data-ttu-id="d6d95-181">Il pulsante attiva il `OnPostResetCachedTime` metodo del gestore.</span><span class="sxs-lookup"><span data-stu-id="d6d95-181">The button triggers the `OnPostResetCachedTime` handler method.</span></span>

[!code-csharp[](distributed/samples/3.x/DistCacheSample/Pages/Index.cshtml.cs?name=snippet_IndexModel&highlight=7,14-20,25-29)]

> [!NOTE]
> <span data-ttu-id="d6d95-182">Non è necessario usare un singleton o una durata con ambito per <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> le istanze (almeno per le implementazioni predefinite).</span><span class="sxs-lookup"><span data-stu-id="d6d95-182">There's no need to use a Singleton or Scoped lifetime for <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> instances (at least for the built-in implementations).</span></span>
>
> <span data-ttu-id="d6d95-183">È anche possibile creare un' <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> istanza di ogni volta che è necessario invece di usare, ma la creazione di un'istanza nel codice può rendere il codice più difficile da testare e violare il [principio delle dipendenze esplicite](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#explicit-dependencies).</span><span class="sxs-lookup"><span data-stu-id="d6d95-183">You can also create an <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> instance wherever you might need one instead of using DI, but creating an instance in code can make your code harder to test and violates the [Explicit Dependencies Principle](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#explicit-dependencies).</span></span>

## <a name="recommendations"></a><span data-ttu-id="d6d95-184">Consigli</span><span class="sxs-lookup"><span data-stu-id="d6d95-184">Recommendations</span></span>

<span data-ttu-id="d6d95-185">Quando si decide quale implementazione di <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> è migliore per l'app, tenere presente quanto segue:</span><span class="sxs-lookup"><span data-stu-id="d6d95-185">When deciding which implementation of <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> is best for your app, consider the following:</span></span>

* <span data-ttu-id="d6d95-186">Infrastruttura esistente</span><span class="sxs-lookup"><span data-stu-id="d6d95-186">Existing infrastructure</span></span>
* <span data-ttu-id="d6d95-187">Requisiti per le prestazioni</span><span class="sxs-lookup"><span data-stu-id="d6d95-187">Performance requirements</span></span>
* <span data-ttu-id="d6d95-188">Costo</span><span class="sxs-lookup"><span data-stu-id="d6d95-188">Cost</span></span>
* <span data-ttu-id="d6d95-189">Esperienza del team</span><span class="sxs-lookup"><span data-stu-id="d6d95-189">Team experience</span></span>

<span data-ttu-id="d6d95-190">Le soluzioni per la memorizzazione nella cache si basano in genere sull'archiviazione in memoria per consentire un rapido recupero dei dati memorizzati nella cache, ma la memoria è una risorsa limitata ed è dispendiosa per espanderla.</span><span class="sxs-lookup"><span data-stu-id="d6d95-190">Caching solutions usually rely on in-memory storage to provide fast retrieval of cached data, but memory is a limited resource and costly to expand.</span></span> <span data-ttu-id="d6d95-191">Archiviare solo i dati utilizzati di frequente in una cache.</span><span class="sxs-lookup"><span data-stu-id="d6d95-191">Only store commonly used data in a cache.</span></span>

<span data-ttu-id="d6d95-192">In genere, una cache Redis offre una velocità effettiva più elevata e una latenza inferiore rispetto a una cache SQL Server.</span><span class="sxs-lookup"><span data-stu-id="d6d95-192">Generally, a Redis cache provides higher throughput and lower latency than a SQL Server cache.</span></span> <span data-ttu-id="d6d95-193">Tuttavia, il benchmarking è in genere necessario per determinare le caratteristiche di prestazioni delle strategie di memorizzazione nella cache.</span><span class="sxs-lookup"><span data-stu-id="d6d95-193">However, benchmarking is usually required to determine the performance characteristics of caching strategies.</span></span>

<span data-ttu-id="d6d95-194">Quando SQL Server viene usato come archivio di backup della cache distribuita, l'uso dello stesso database per la cache e l'archiviazione e il recupero dei dati ordinari dell'app possono influire negativamente sulle prestazioni di entrambi.</span><span class="sxs-lookup"><span data-stu-id="d6d95-194">When SQL Server is used as a distributed cache backing store, use of the same database for the cache and the app's ordinary data storage and retrieval can negatively impact the performance of both.</span></span> <span data-ttu-id="d6d95-195">Si consiglia di usare un'istanza di SQL Server dedicata per l'archivio di backup della cache distribuita.</span><span class="sxs-lookup"><span data-stu-id="d6d95-195">We recommend using a dedicated SQL Server instance for the distributed cache backing store.</span></span>

## <a name="additional-resources"></a><span data-ttu-id="d6d95-196">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="d6d95-196">Additional resources</span></span>

* [<span data-ttu-id="d6d95-197">Cache Redis in Azure</span><span class="sxs-lookup"><span data-stu-id="d6d95-197">Redis Cache on Azure</span></span>](/azure/azure-cache-for-redis/)
* [<span data-ttu-id="d6d95-198">Database SQL in Azure</span><span class="sxs-lookup"><span data-stu-id="d6d95-198">SQL Database on Azure</span></span>](/azure/sql-database/)
* <span data-ttu-id="d6d95-199">[ASP.NET Core provider IDistributedCache per NCache in Web farm](http://www.alachisoft.com/ncache/aspnet-core-idistributedcache-ncache.html) ([NCache su GitHub](https://github.com/Alachisoft/NCache))</span><span class="sxs-lookup"><span data-stu-id="d6d95-199">[ASP.NET Core IDistributedCache Provider for NCache in Web Farms](http://www.alachisoft.com/ncache/aspnet-core-idistributedcache-ncache.html) ([NCache on GitHub](https://github.com/Alachisoft/NCache))</span></span>
* <xref:performance/caching/memory>
* <xref:fundamentals/change-tokens>
* <xref:performance/caching/response>
* <xref:performance/caching/middleware>
* <xref:mvc/views/tag-helpers/builtin-th/cache-tag-helper>
* <xref:mvc/views/tag-helpers/builtin-th/distributed-cache-tag-helper>
* <xref:host-and-deploy/web-farm>

::: moniker-end

::: moniker range="= aspnetcore-2.2"

<span data-ttu-id="d6d95-200">Una cache distribuita è una cache condivisa da più server app, in genere gestita come servizio esterno ai server app che vi accedono.</span><span class="sxs-lookup"><span data-stu-id="d6d95-200">A distributed cache is a cache shared by multiple app servers, typically maintained as an external service to the app servers that access it.</span></span> <span data-ttu-id="d6d95-201">Una cache distribuita può migliorare le prestazioni e la scalabilità di un'app ASP.NET Core, soprattutto quando l'app è ospitata da un servizio cloud o da un server farm.</span><span class="sxs-lookup"><span data-stu-id="d6d95-201">A distributed cache can improve the performance and scalability of an ASP.NET Core app, especially when the app is hosted by a cloud service or a server farm.</span></span>

<span data-ttu-id="d6d95-202">Una cache distribuita presenta diversi vantaggi rispetto ad altri scenari di memorizzazione nella cache in cui i dati memorizzati nella cache vengono archiviati nei singoli server dell'app.</span><span class="sxs-lookup"><span data-stu-id="d6d95-202">A distributed cache has several advantages over other caching scenarios where cached data is stored on individual app servers.</span></span>

<span data-ttu-id="d6d95-203">Quando vengono distribuiti i dati memorizzati nella cache, i dati:</span><span class="sxs-lookup"><span data-stu-id="d6d95-203">When cached data is distributed, the data:</span></span>

* <span data-ttu-id="d6d95-204">È *coerente* (coerente) tra le richieste a più server.</span><span class="sxs-lookup"><span data-stu-id="d6d95-204">Is *coherent* (consistent) across requests to multiple servers.</span></span>
* <span data-ttu-id="d6d95-205">Sopravvive ai riavvii del server e alle distribuzioni di app.</span><span class="sxs-lookup"><span data-stu-id="d6d95-205">Survives server restarts and app deployments.</span></span>
* <span data-ttu-id="d6d95-206">Non usa la memoria locale.</span><span class="sxs-lookup"><span data-stu-id="d6d95-206">Doesn't use local memory.</span></span>

<span data-ttu-id="d6d95-207">La configurazione della cache distribuita è specifica dell'implementazione.</span><span class="sxs-lookup"><span data-stu-id="d6d95-207">Distributed cache configuration is implementation specific.</span></span> <span data-ttu-id="d6d95-208">Questo articolo descrive come configurare le cache distribuite SQL Server e Redis.</span><span class="sxs-lookup"><span data-stu-id="d6d95-208">This article describes how to configure SQL Server and Redis distributed caches.</span></span> <span data-ttu-id="d6d95-209">Sono disponibili anche implementazioni di terze parti, ad esempio [NCache](http://www.alachisoft.com/ncache/aspnet-core-idistributedcache-ncache.html) ([NCache su GitHub](https://github.com/Alachisoft/NCache)).</span><span class="sxs-lookup"><span data-stu-id="d6d95-209">Third party implementations are also available, such as [NCache](http://www.alachisoft.com/ncache/aspnet-core-idistributedcache-ncache.html) ([NCache on GitHub](https://github.com/Alachisoft/NCache)).</span></span> <span data-ttu-id="d6d95-210">Indipendentemente dall'implementazione selezionata, l'app interagisce con la cache usando l' <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> interfaccia.</span><span class="sxs-lookup"><span data-stu-id="d6d95-210">Regardless of which implementation is selected, the app interacts with the cache using the <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> interface.</span></span>

<span data-ttu-id="d6d95-211">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/performance/caching/distributed/samples/) ([procedura per il download](xref:index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="d6d95-211">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/performance/caching/distributed/samples/) ([how to download](xref:index#how-to-download-a-sample))</span></span>

## <a name="prerequisites"></a><span data-ttu-id="d6d95-212">Prerequisiti</span><span class="sxs-lookup"><span data-stu-id="d6d95-212">Prerequisites</span></span>

<span data-ttu-id="d6d95-213">Per usare una cache distribuita SQL Server, fare riferimento al [metapacchetto Microsoft. AspNetCore. app](xref:fundamentals/metapackage-app) o aggiungere un riferimento al pacchetto [Microsoft. Extensions. Caching. SqlServer](https://www.nuget.org/packages/Microsoft.Extensions.Caching.SqlServer) .</span><span class="sxs-lookup"><span data-stu-id="d6d95-213">To use a SQL Server distributed cache, reference the [Microsoft.AspNetCore.App metapackage](xref:fundamentals/metapackage-app) or add a package reference to the [Microsoft.Extensions.Caching.SqlServer](https://www.nuget.org/packages/Microsoft.Extensions.Caching.SqlServer) package.</span></span>

<span data-ttu-id="d6d95-214">Per usare una cache distribuita Redis, fare riferimento al [metapacchetto Microsoft. AspNetCore. app](xref:fundamentals/metapackage-app) e aggiungere un riferimento al pacchetto [Microsoft. Extensions. Caching. StackExchangeRedis](https://www.nuget.org/packages/Microsoft.Extensions.Caching.StackExchangeRedis) .</span><span class="sxs-lookup"><span data-stu-id="d6d95-214">To use a Redis distributed cache, reference the [Microsoft.AspNetCore.App metapackage](xref:fundamentals/metapackage-app) and add a package reference to the [Microsoft.Extensions.Caching.StackExchangeRedis](https://www.nuget.org/packages/Microsoft.Extensions.Caching.StackExchangeRedis) package.</span></span> <span data-ttu-id="d6d95-215">Il pacchetto Redis non è incluso nel `Microsoft.AspNetCore.App` pacchetto, quindi è necessario fare riferimento al pacchetto Redis separatamente nel file di progetto.</span><span class="sxs-lookup"><span data-stu-id="d6d95-215">The Redis package isn't included in the `Microsoft.AspNetCore.App` package, so you must reference the Redis package separately in your project file.</span></span>

<span data-ttu-id="d6d95-216">Per usare la cache distribuita NCache, fare riferimento al [metapacchetto Microsoft. AspNetCore. app](xref:fundamentals/metapackage-app) e aggiungere un riferimento al pacchetto [NCache. Microsoft. Extensions. Caching. opensource](https://www.nuget.org/packages/NCache.Microsoft.Extensions.Caching.OpenSource) .</span><span class="sxs-lookup"><span data-stu-id="d6d95-216">To use NCache distributed cache, reference the [Microsoft.AspNetCore.App metapackage](xref:fundamentals/metapackage-app) and add a package reference to the [NCache.Microsoft.Extensions.Caching.OpenSource](https://www.nuget.org/packages/NCache.Microsoft.Extensions.Caching.OpenSource) package.</span></span> <span data-ttu-id="d6d95-217">Il pacchetto NCache non è incluso nel `Microsoft.AspNetCore.App` pacchetto, quindi è necessario fare riferimento al pacchetto NCache separatamente nel file di progetto.</span><span class="sxs-lookup"><span data-stu-id="d6d95-217">The NCache package isn't included in the `Microsoft.AspNetCore.App` package, so you must reference the NCache package separately in your project file.</span></span>

## <a name="idistributedcache-interface"></a><span data-ttu-id="d6d95-218">Interfaccia IDistributedCache</span><span class="sxs-lookup"><span data-stu-id="d6d95-218">IDistributedCache interface</span></span>

<span data-ttu-id="d6d95-219">L' <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> interfaccia fornisce i metodi seguenti per modificare gli elementi nell'implementazione della cache distribuita:</span><span class="sxs-lookup"><span data-stu-id="d6d95-219">The <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> interface provides the following methods to manipulate items in the distributed cache implementation:</span></span>

* <span data-ttu-id="d6d95-220"><xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache.Get*>, <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache.GetAsync*> : Accetta una chiave di stringa e recupera un elemento memorizzato nella cache come `byte[]` matrice se viene trovato nella cache.</span><span class="sxs-lookup"><span data-stu-id="d6d95-220"><xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache.Get*>, <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache.GetAsync*>: Accepts a string key and retrieves a cached item as a `byte[]` array if found in the cache.</span></span>
* <span data-ttu-id="d6d95-221"><xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache.Set*>, <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache.SetAsync*> : Aggiunge un elemento (come `byte[]` matrice) alla cache usando una chiave di stringa.</span><span class="sxs-lookup"><span data-stu-id="d6d95-221"><xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache.Set*>, <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache.SetAsync*>: Adds an item (as `byte[]` array) to the cache using a string key.</span></span>
* <span data-ttu-id="d6d95-222"><xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache.Refresh*>, <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache.RefreshAsync*> : Aggiorna un elemento nella cache in base alla relativa chiave, reimpostando il timeout di scadenza variabile (se presente).</span><span class="sxs-lookup"><span data-stu-id="d6d95-222"><xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache.Refresh*>, <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache.RefreshAsync*>: Refreshes an item in the cache based on its key, resetting its sliding expiration timeout (if any).</span></span>
* <span data-ttu-id="d6d95-223"><xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache.Remove*>, <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache.RemoveAsync*> : Rimuove un elemento della cache in base alla relativa chiave di stringa.</span><span class="sxs-lookup"><span data-stu-id="d6d95-223"><xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache.Remove*>, <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache.RemoveAsync*>: Removes a cache item based on its string key.</span></span>

## <a name="establish-distributed-caching-services"></a><span data-ttu-id="d6d95-224">Stabilire servizi di memorizzazione nella cache distribuiti</span><span class="sxs-lookup"><span data-stu-id="d6d95-224">Establish distributed caching services</span></span>

<span data-ttu-id="d6d95-225">Registrare un'implementazione di <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> in `Startup.ConfigureServices` .</span><span class="sxs-lookup"><span data-stu-id="d6d95-225">Register an implementation of <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> in `Startup.ConfigureServices`.</span></span> <span data-ttu-id="d6d95-226">Le implementazioni fornite dal Framework descritte in questo argomento includono:</span><span class="sxs-lookup"><span data-stu-id="d6d95-226">Framework-provided implementations described in this topic include:</span></span>

* [<span data-ttu-id="d6d95-227">Cache di memoria distribuita</span><span class="sxs-lookup"><span data-stu-id="d6d95-227">Distributed Memory Cache</span></span>](#distributed-memory-cache)
* [<span data-ttu-id="d6d95-228">Cache SQL Server distribuita</span><span class="sxs-lookup"><span data-stu-id="d6d95-228">Distributed SQL Server cache</span></span>](#distributed-sql-server-cache)
* [<span data-ttu-id="d6d95-229">Cache Redis distribuita</span><span class="sxs-lookup"><span data-stu-id="d6d95-229">Distributed Redis cache</span></span>](#distributed-redis-cache)
* [<span data-ttu-id="d6d95-230">Cache NCache distribuita</span><span class="sxs-lookup"><span data-stu-id="d6d95-230">Distributed NCache cache</span></span>](#distributed-ncache-cache)

### <a name="distributed-memory-cache"></a><span data-ttu-id="d6d95-231">Cache di memoria distribuita</span><span class="sxs-lookup"><span data-stu-id="d6d95-231">Distributed Memory Cache</span></span>

<span data-ttu-id="d6d95-232">La cache di memoria distribuita ( <xref:Microsoft.Extensions.DependencyInjection.MemoryCacheServiceCollectionExtensions.AddDistributedMemoryCache*> ) è un'implementazione fornita dal framework di <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> che archivia gli elementi in memoria.</span><span class="sxs-lookup"><span data-stu-id="d6d95-232">The Distributed Memory Cache (<xref:Microsoft.Extensions.DependencyInjection.MemoryCacheServiceCollectionExtensions.AddDistributedMemoryCache*>) is a framework-provided implementation of <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> that stores items in memory.</span></span> <span data-ttu-id="d6d95-233">La cache di memoria distribuita non è una cache distribuita effettiva.</span><span class="sxs-lookup"><span data-stu-id="d6d95-233">The Distributed Memory Cache isn't an actual distributed cache.</span></span> <span data-ttu-id="d6d95-234">Gli elementi memorizzati nella cache vengono archiviati dall'istanza dell'app nel server in cui è in esecuzione l'app.</span><span class="sxs-lookup"><span data-stu-id="d6d95-234">Cached items are stored by the app instance on the server where the app is running.</span></span>

<span data-ttu-id="d6d95-235">La cache di memoria distribuita è un'implementazione utile:</span><span class="sxs-lookup"><span data-stu-id="d6d95-235">The Distributed Memory Cache is a useful implementation:</span></span>

* <span data-ttu-id="d6d95-236">Negli scenari di sviluppo e test.</span><span class="sxs-lookup"><span data-stu-id="d6d95-236">In development and testing scenarios.</span></span>
* <span data-ttu-id="d6d95-237">Quando un singolo server viene utilizzato nell'ambiente di produzione e l'utilizzo della memoria non costituisce un problema.</span><span class="sxs-lookup"><span data-stu-id="d6d95-237">When a single server is used in production and memory consumption isn't an issue.</span></span> <span data-ttu-id="d6d95-238">L'implementazione della cache di memoria distribuita astrae l'archiviazione dei dati memorizzati nella cache.</span><span class="sxs-lookup"><span data-stu-id="d6d95-238">Implementing the Distributed Memory Cache abstracts cached data storage.</span></span> <span data-ttu-id="d6d95-239">Consente di implementare in futuro una soluzione di memorizzazione nella cache distribuita se più nodi o tolleranza di errore diventano necessari.</span><span class="sxs-lookup"><span data-stu-id="d6d95-239">It allows for implementing a true distributed caching solution in the future if multiple nodes or fault tolerance become necessary.</span></span>

<span data-ttu-id="d6d95-240">L'app di esempio usa la cache di memoria distribuita quando l'app viene eseguita nell'ambiente di sviluppo in `Startup.ConfigureServices` :</span><span class="sxs-lookup"><span data-stu-id="d6d95-240">The sample app makes use of the Distributed Memory Cache when the app is run in the Development environment in `Startup.ConfigureServices`:</span></span>

[!code-csharp[](distributed/samples/2.x/DistCacheSample/Startup.cs?name=snippet_AddDistributedMemoryCache)]

### <a name="distributed-sql-server-cache"></a><span data-ttu-id="d6d95-241">Cache SQL Server distribuita</span><span class="sxs-lookup"><span data-stu-id="d6d95-241">Distributed SQL Server Cache</span></span>

<span data-ttu-id="d6d95-242">L'implementazione della cache SQL Server distribuita ( <xref:Microsoft.Extensions.DependencyInjection.SqlServerCachingServicesExtensions.AddDistributedSqlServerCache*> ) consente alla cache distribuita di usare un database SQL Server come archivio di backup.</span><span class="sxs-lookup"><span data-stu-id="d6d95-242">The Distributed SQL Server Cache implementation (<xref:Microsoft.Extensions.DependencyInjection.SqlServerCachingServicesExtensions.AddDistributedSqlServerCache*>) allows the distributed cache to use a SQL Server database as its backing store.</span></span> <span data-ttu-id="d6d95-243">Per creare una tabella SQL Server elemento memorizzato nella cache in un'istanza di SQL Server, è possibile utilizzare lo `sql-cache` strumento.</span><span class="sxs-lookup"><span data-stu-id="d6d95-243">To create a SQL Server cached item table in a SQL Server instance, you can use the `sql-cache` tool.</span></span> <span data-ttu-id="d6d95-244">Lo strumento crea una tabella con il nome e lo schema specificati.</span><span class="sxs-lookup"><span data-stu-id="d6d95-244">The tool creates a table with the name and schema that you specify.</span></span>

<span data-ttu-id="d6d95-245">Creare una tabella in SQL Server eseguendo il `sql-cache create` comando.</span><span class="sxs-lookup"><span data-stu-id="d6d95-245">Create a table in SQL Server by running the `sql-cache create` command.</span></span> <span data-ttu-id="d6d95-246">Fornire l'istanza di SQL Server ( `Data Source` ), il database ( `Initial Catalog` ), lo schema (ad esempio, `dbo` ) e il nome della tabella (ad esempio, `TestCache` ):</span><span class="sxs-lookup"><span data-stu-id="d6d95-246">Provide the SQL Server instance (`Data Source`), database (`Initial Catalog`), schema (for example, `dbo`), and table name (for example, `TestCache`):</span></span>

```dotnetcli
dotnet sql-cache create "Data Source=(localdb)\MSSQLLocalDB;Initial Catalog=DistCache;Integrated Security=True;" dbo TestCache
```

<span data-ttu-id="d6d95-247">Viene registrato un messaggio per indicare che lo strumento è stato completato correttamente:</span><span class="sxs-lookup"><span data-stu-id="d6d95-247">A message is logged to indicate that the tool was successful:</span></span>

```console
Table and index were created successfully.
```

<span data-ttu-id="d6d95-248">La tabella creata dallo `sql-cache` strumento presenta lo schema seguente:</span><span class="sxs-lookup"><span data-stu-id="d6d95-248">The table created by the `sql-cache` tool has the following schema:</span></span>

![Tabella cache SqlServer](distributed/_static/SqlServerCacheTable.png)

> [!NOTE]
> <span data-ttu-id="d6d95-250">Un'app deve modificare i valori della cache usando un'istanza di <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> , non un oggetto <xref:Microsoft.Extensions.Caching.SqlServer.SqlServerCache> .</span><span class="sxs-lookup"><span data-stu-id="d6d95-250">An app should manipulate cache values using an instance of <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache>, not a <xref:Microsoft.Extensions.Caching.SqlServer.SqlServerCache>.</span></span>

<span data-ttu-id="d6d95-251">L'app di esempio implementa <xref:Microsoft.Extensions.Caching.SqlServer.SqlServerCache> in un ambiente non di sviluppo in `Startup.ConfigureServices` :</span><span class="sxs-lookup"><span data-stu-id="d6d95-251">The sample app implements <xref:Microsoft.Extensions.Caching.SqlServer.SqlServerCache> in a non-Development environment in `Startup.ConfigureServices`:</span></span>

[!code-csharp[](distributed/samples/2.x/DistCacheSample/Startup.cs?name=snippet_AddDistributedSqlServerCache)]

> [!NOTE]
> <span data-ttu-id="d6d95-252">Un <xref:Microsoft.Extensions.Caching.SqlServer.SqlServerCacheOptions.ConnectionString*> (e, facoltativamente, <xref:Microsoft.Extensions.Caching.SqlServer.SqlServerCacheOptions.SchemaName*> e <xref:Microsoft.Extensions.Caching.SqlServer.SqlServerCacheOptions.TableName*> ) vengono in genere archiviati all'esterno del controllo del codice sorgente, ad esempio archiviato dal [gestore dei segreti](xref:security/app-secrets) o in *appsettings.json* / *appSettings. { ENVIRONMENT} file JSON* ).</span><span class="sxs-lookup"><span data-stu-id="d6d95-252">A <xref:Microsoft.Extensions.Caching.SqlServer.SqlServerCacheOptions.ConnectionString*> (and optionally, <xref:Microsoft.Extensions.Caching.SqlServer.SqlServerCacheOptions.SchemaName*> and <xref:Microsoft.Extensions.Caching.SqlServer.SqlServerCacheOptions.TableName*>) are typically stored outside of source control (for example, stored by the [Secret Manager](xref:security/app-secrets) or in *appsettings.json*/*appsettings.{ENVIRONMENT}.json* files).</span></span> <span data-ttu-id="d6d95-253">La stringa di connessione può contenere credenziali che devono essere mantenute fuori dai sistemi di controllo del codice sorgente.</span><span class="sxs-lookup"><span data-stu-id="d6d95-253">The connection string may contain credentials that should be kept out of source control systems.</span></span>

### <a name="distributed-redis-cache"></a><span data-ttu-id="d6d95-254">Cache Redis distribuita</span><span class="sxs-lookup"><span data-stu-id="d6d95-254">Distributed Redis Cache</span></span>

<span data-ttu-id="d6d95-255">[Redis](https://redis.io/) è un archivio dati in memoria open source, che viene spesso usato come cache distribuita.</span><span class="sxs-lookup"><span data-stu-id="d6d95-255">[Redis](https://redis.io/) is an open source in-memory data store, which is often used as a distributed cache.</span></span> <span data-ttu-id="d6d95-256">È possibile usare Redis localmente ed è possibile configurare una [cache Redis di Azure](https://azure.microsoft.com/services/cache/) per un'app ASP.NET Core ospitata in Azure.</span><span class="sxs-lookup"><span data-stu-id="d6d95-256">You can use Redis locally, and you can configure an [Azure Redis Cache](https://azure.microsoft.com/services/cache/) for an Azure-hosted ASP.NET Core app.</span></span>

<span data-ttu-id="d6d95-257">Un'app configura l'implementazione della cache usando un' <xref:Microsoft.Extensions.Caching.StackExchangeRedis.RedisCache> istanza ( <xref:Microsoft.Extensions.DependencyInjection.StackExchangeRedisCacheServiceCollectionExtensions.AddStackExchangeRedisCache*> ) in un ambiente non di sviluppo in `Startup.ConfigureServices` :</span><span class="sxs-lookup"><span data-stu-id="d6d95-257">An app configures the cache implementation using a <xref:Microsoft.Extensions.Caching.StackExchangeRedis.RedisCache> instance (<xref:Microsoft.Extensions.DependencyInjection.StackExchangeRedisCacheServiceCollectionExtensions.AddStackExchangeRedisCache*>) in a non-Development environment in `Startup.ConfigureServices`:</span></span>

[!code-csharp[](distributed/samples/2.x/DistCacheSample/Startup.cs?name=snippet_AddStackExchangeRedisCache)]

<span data-ttu-id="d6d95-258">Per installare Redis nel computer locale:</span><span class="sxs-lookup"><span data-stu-id="d6d95-258">To install Redis on your local machine:</span></span>

1. <span data-ttu-id="d6d95-259">Installare il [pacchetto di cioccolaty Redis](https://chocolatey.org/packages/redis-64/).</span><span class="sxs-lookup"><span data-stu-id="d6d95-259">Install the [Chocolatey Redis package](https://chocolatey.org/packages/redis-64/).</span></span>
1. <span data-ttu-id="d6d95-260">Eseguire `redis-server` da un prompt dei comandi.</span><span class="sxs-lookup"><span data-stu-id="d6d95-260">Run `redis-server` from a command prompt.</span></span>

### <a name="distributed-ncache-cache"></a><span data-ttu-id="d6d95-261">Cache NCache distribuita</span><span class="sxs-lookup"><span data-stu-id="d6d95-261">Distributed NCache Cache</span></span>

<span data-ttu-id="d6d95-262">[NCache](https://github.com/Alachisoft/NCache) è una cache distribuita in memoria open source sviluppata in modo nativo in .NET e .NET Core.</span><span class="sxs-lookup"><span data-stu-id="d6d95-262">[NCache](https://github.com/Alachisoft/NCache) is an open source in-memory distributed cache developed natively in .NET and .NET Core.</span></span> <span data-ttu-id="d6d95-263">NCache funziona sia localmente che configurato come cluster di cache distribuita per un'app ASP.NET Core in esecuzione in Azure o in altre piattaforme di hosting.</span><span class="sxs-lookup"><span data-stu-id="d6d95-263">NCache works both locally and configured as a distributed cache cluster for an ASP.NET Core app running in Azure or on other hosting platforms.</span></span>

<span data-ttu-id="d6d95-264">Per installare e configurare NCache nel computer locale, vedere la [guida introduzione per Windows (.NET e .NET Core)](https://www.alachisoft.com/resources/docs/ncache/getting-started-guide-windows/).</span><span class="sxs-lookup"><span data-stu-id="d6d95-264">To install and configure NCache on your local machine, see [Getting Started Guide for Windows (.NET and .NET Core)](https://www.alachisoft.com/resources/docs/ncache/getting-started-guide-windows/).</span></span>

<span data-ttu-id="d6d95-265">Per configurare NCache:</span><span class="sxs-lookup"><span data-stu-id="d6d95-265">To configure NCache:</span></span>

1. <span data-ttu-id="d6d95-266">Installare [NCache Open Source NuGet](https://www.nuget.org/packages/Alachisoft.NCache.OpenSource.SDK/).</span><span class="sxs-lookup"><span data-stu-id="d6d95-266">Install [NCache open source NuGet](https://www.nuget.org/packages/Alachisoft.NCache.OpenSource.SDK/).</span></span>
1. <span data-ttu-id="d6d95-267">Configurare il cluster di cache in [client. ncconf](https://www.alachisoft.com/resources/docs/ncache/admin-guide/client-config.html).</span><span class="sxs-lookup"><span data-stu-id="d6d95-267">Configure the cache cluster in [client.ncconf](https://www.alachisoft.com/resources/docs/ncache/admin-guide/client-config.html).</span></span>
1. <span data-ttu-id="d6d95-268">Aggiungere il codice seguente a `Startup.ConfigureServices`:</span><span class="sxs-lookup"><span data-stu-id="d6d95-268">Add the following code to `Startup.ConfigureServices`:</span></span>

   ```csharp
   services.AddNCacheDistributedCache(configuration =>    
   {        
       configuration.CacheName = "demoClusteredCache";
       configuration.EnableLogs = true;
       configuration.ExceptionsEnabled = true;
   });
   ```

## <a name="use-the-distributed-cache"></a><span data-ttu-id="d6d95-269">Usare la cache distribuita</span><span class="sxs-lookup"><span data-stu-id="d6d95-269">Use the distributed cache</span></span>

<span data-ttu-id="d6d95-270">Per usare l' <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> interfaccia, richiedere un'istanza di <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> da qualsiasi costruttore nell'app.</span><span class="sxs-lookup"><span data-stu-id="d6d95-270">To use the <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> interface, request an instance of <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> from any constructor in the app.</span></span> <span data-ttu-id="d6d95-271">L'istanza viene fornita da [inserimento delle dipendenze (di)](xref:fundamentals/dependency-injection).</span><span class="sxs-lookup"><span data-stu-id="d6d95-271">The instance is provided by [dependency injection (DI)](xref:fundamentals/dependency-injection).</span></span>

<span data-ttu-id="d6d95-272">Quando l'app di esempio viene avviata, <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> viene inserita in `Startup.Configure` .</span><span class="sxs-lookup"><span data-stu-id="d6d95-272">When the sample app starts, <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> is injected into `Startup.Configure`.</span></span> <span data-ttu-id="d6d95-273">L'ora corrente viene memorizzata nella cache usando <xref:Microsoft.AspNetCore.Hosting.IApplicationLifetime> (per altre informazioni, vedere [Web host: interfaccia IApplicationLifetime](xref:fundamentals/host/web-host#iapplicationlifetime-interface)):</span><span class="sxs-lookup"><span data-stu-id="d6d95-273">The current time is cached using <xref:Microsoft.AspNetCore.Hosting.IApplicationLifetime> (for more information, see [Web Host: IApplicationLifetime interface](xref:fundamentals/host/web-host#iapplicationlifetime-interface)):</span></span>

[!code-csharp[](distributed/samples/2.x/DistCacheSample/Startup.cs?name=snippet_Configure&highlight=10)]

<span data-ttu-id="d6d95-274">L'app di esempio inserisce <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> nell'oggetto `IndexModel` per l'uso da parte della pagina di indice.</span><span class="sxs-lookup"><span data-stu-id="d6d95-274">The sample app injects <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> into the `IndexModel` for use by the Index page.</span></span>

<span data-ttu-id="d6d95-275">Ogni volta che viene caricata la pagina di indice, viene verificata la presenza dell'ora memorizzata nella cache in `OnGetAsync` .</span><span class="sxs-lookup"><span data-stu-id="d6d95-275">Each time the Index page is loaded, the cache is checked for the cached time in `OnGetAsync`.</span></span> <span data-ttu-id="d6d95-276">Se il tempo di memorizzazione nella cache non è scaduto, viene visualizzata l'ora.</span><span class="sxs-lookup"><span data-stu-id="d6d95-276">If the cached time hasn't expired, the time is displayed.</span></span> <span data-ttu-id="d6d95-277">Se sono trascorsi 20 secondi dall'ultima volta in cui è stato eseguito l'accesso all'ora memorizzata nella cache (l'ultima volta che questa pagina è stata caricata), la pagina Visualizza l' *ora della cache scaduta*.</span><span class="sxs-lookup"><span data-stu-id="d6d95-277">If 20 seconds have elapsed since the last time the cached time was accessed (the last time this page was loaded), the page displays *Cached Time Expired*.</span></span>

<span data-ttu-id="d6d95-278">Aggiornare immediatamente l'ora memorizzata nella cache all'ora corrente selezionando il pulsante **Reimposta tempo memorizzato nella cache** .</span><span class="sxs-lookup"><span data-stu-id="d6d95-278">Immediately update the cached time to the current time by selecting the **Reset Cached Time** button.</span></span> <span data-ttu-id="d6d95-279">Il pulsante attiva il `OnPostResetCachedTime` metodo del gestore.</span><span class="sxs-lookup"><span data-stu-id="d6d95-279">The button triggers the `OnPostResetCachedTime` handler method.</span></span>

[!code-csharp[](distributed/samples/2.x/DistCacheSample/Pages/Index.cshtml.cs?name=snippet_IndexModel&highlight=7,14-20,25-29)]

> [!NOTE]
> <span data-ttu-id="d6d95-280">Non è necessario usare un singleton o una durata con ambito per <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> le istanze (almeno per le implementazioni predefinite).</span><span class="sxs-lookup"><span data-stu-id="d6d95-280">There's no need to use a Singleton or Scoped lifetime for <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> instances (at least for the built-in implementations).</span></span>
>
> <span data-ttu-id="d6d95-281">È anche possibile creare un' <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> istanza di ogni volta che è necessario invece di usare, ma la creazione di un'istanza nel codice può rendere il codice più difficile da testare e violare il [principio delle dipendenze esplicite](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#explicit-dependencies).</span><span class="sxs-lookup"><span data-stu-id="d6d95-281">You can also create an <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> instance wherever you might need one instead of using DI, but creating an instance in code can make your code harder to test and violates the [Explicit Dependencies Principle](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#explicit-dependencies).</span></span>

## <a name="recommendations"></a><span data-ttu-id="d6d95-282">Consigli</span><span class="sxs-lookup"><span data-stu-id="d6d95-282">Recommendations</span></span>

<span data-ttu-id="d6d95-283">Quando si decide quale implementazione di <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> è migliore per l'app, tenere presente quanto segue:</span><span class="sxs-lookup"><span data-stu-id="d6d95-283">When deciding which implementation of <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> is best for your app, consider the following:</span></span>

* <span data-ttu-id="d6d95-284">Infrastruttura esistente</span><span class="sxs-lookup"><span data-stu-id="d6d95-284">Existing infrastructure</span></span>
* <span data-ttu-id="d6d95-285">Requisiti per le prestazioni</span><span class="sxs-lookup"><span data-stu-id="d6d95-285">Performance requirements</span></span>
* <span data-ttu-id="d6d95-286">Costo</span><span class="sxs-lookup"><span data-stu-id="d6d95-286">Cost</span></span>
* <span data-ttu-id="d6d95-287">Esperienza del team</span><span class="sxs-lookup"><span data-stu-id="d6d95-287">Team experience</span></span>

<span data-ttu-id="d6d95-288">Le soluzioni per la memorizzazione nella cache si basano in genere sull'archiviazione in memoria per consentire un rapido recupero dei dati memorizzati nella cache, ma la memoria è una risorsa limitata ed è dispendiosa per espanderla.</span><span class="sxs-lookup"><span data-stu-id="d6d95-288">Caching solutions usually rely on in-memory storage to provide fast retrieval of cached data, but memory is a limited resource and costly to expand.</span></span> <span data-ttu-id="d6d95-289">Archiviare solo i dati utilizzati di frequente in una cache.</span><span class="sxs-lookup"><span data-stu-id="d6d95-289">Only store commonly used data in a cache.</span></span>

<span data-ttu-id="d6d95-290">In genere, una cache Redis offre una velocità effettiva più elevata e una latenza inferiore rispetto a una cache SQL Server.</span><span class="sxs-lookup"><span data-stu-id="d6d95-290">Generally, a Redis cache provides higher throughput and lower latency than a SQL Server cache.</span></span> <span data-ttu-id="d6d95-291">Tuttavia, il benchmarking è in genere necessario per determinare le caratteristiche di prestazioni delle strategie di memorizzazione nella cache.</span><span class="sxs-lookup"><span data-stu-id="d6d95-291">However, benchmarking is usually required to determine the performance characteristics of caching strategies.</span></span>

<span data-ttu-id="d6d95-292">Quando SQL Server viene usato come archivio di backup della cache distribuita, l'uso dello stesso database per la cache e l'archiviazione e il recupero dei dati ordinari dell'app possono influire negativamente sulle prestazioni di entrambi.</span><span class="sxs-lookup"><span data-stu-id="d6d95-292">When SQL Server is used as a distributed cache backing store, use of the same database for the cache and the app's ordinary data storage and retrieval can negatively impact the performance of both.</span></span> <span data-ttu-id="d6d95-293">Si consiglia di usare un'istanza di SQL Server dedicata per l'archivio di backup della cache distribuita.</span><span class="sxs-lookup"><span data-stu-id="d6d95-293">We recommend using a dedicated SQL Server instance for the distributed cache backing store.</span></span>

## <a name="additional-resources"></a><span data-ttu-id="d6d95-294">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="d6d95-294">Additional resources</span></span>

* [<span data-ttu-id="d6d95-295">Cache Redis in Azure</span><span class="sxs-lookup"><span data-stu-id="d6d95-295">Redis Cache on Azure</span></span>](/azure/azure-cache-for-redis/)
* [<span data-ttu-id="d6d95-296">Database SQL in Azure</span><span class="sxs-lookup"><span data-stu-id="d6d95-296">SQL Database on Azure</span></span>](/azure/sql-database/)
* <span data-ttu-id="d6d95-297">[ASP.NET Core provider IDistributedCache per NCache in Web farm](http://www.alachisoft.com/ncache/aspnet-core-idistributedcache-ncache.html) ([NCache su GitHub](https://github.com/Alachisoft/NCache))</span><span class="sxs-lookup"><span data-stu-id="d6d95-297">[ASP.NET Core IDistributedCache Provider for NCache in Web Farms](http://www.alachisoft.com/ncache/aspnet-core-idistributedcache-ncache.html) ([NCache on GitHub](https://github.com/Alachisoft/NCache))</span></span>
* <xref:performance/caching/memory>
* <xref:fundamentals/change-tokens>
* <xref:performance/caching/response>
* <xref:performance/caching/middleware>
* <xref:mvc/views/tag-helpers/builtin-th/cache-tag-helper>
* <xref:mvc/views/tag-helpers/builtin-th/distributed-cache-tag-helper>
* <xref:host-and-deploy/web-farm>

::: moniker-end

::: moniker range="< aspnetcore-2.2"

<span data-ttu-id="d6d95-298">Una cache distribuita è una cache condivisa da più server app, in genere gestita come servizio esterno ai server app che vi accedono.</span><span class="sxs-lookup"><span data-stu-id="d6d95-298">A distributed cache is a cache shared by multiple app servers, typically maintained as an external service to the app servers that access it.</span></span> <span data-ttu-id="d6d95-299">Una cache distribuita può migliorare le prestazioni e la scalabilità di un'app ASP.NET Core, soprattutto quando l'app è ospitata da un servizio cloud o da un server farm.</span><span class="sxs-lookup"><span data-stu-id="d6d95-299">A distributed cache can improve the performance and scalability of an ASP.NET Core app, especially when the app is hosted by a cloud service or a server farm.</span></span>

<span data-ttu-id="d6d95-300">Una cache distribuita presenta diversi vantaggi rispetto ad altri scenari di memorizzazione nella cache in cui i dati memorizzati nella cache vengono archiviati nei singoli server dell'app.</span><span class="sxs-lookup"><span data-stu-id="d6d95-300">A distributed cache has several advantages over other caching scenarios where cached data is stored on individual app servers.</span></span>

<span data-ttu-id="d6d95-301">Quando vengono distribuiti i dati memorizzati nella cache, i dati:</span><span class="sxs-lookup"><span data-stu-id="d6d95-301">When cached data is distributed, the data:</span></span>

* <span data-ttu-id="d6d95-302">È *coerente* (coerente) tra le richieste a più server.</span><span class="sxs-lookup"><span data-stu-id="d6d95-302">Is *coherent* (consistent) across requests to multiple servers.</span></span>
* <span data-ttu-id="d6d95-303">Sopravvive ai riavvii del server e alle distribuzioni di app.</span><span class="sxs-lookup"><span data-stu-id="d6d95-303">Survives server restarts and app deployments.</span></span>
* <span data-ttu-id="d6d95-304">Non usa la memoria locale.</span><span class="sxs-lookup"><span data-stu-id="d6d95-304">Doesn't use local memory.</span></span>

<span data-ttu-id="d6d95-305">La configurazione della cache distribuita è specifica dell'implementazione.</span><span class="sxs-lookup"><span data-stu-id="d6d95-305">Distributed cache configuration is implementation specific.</span></span> <span data-ttu-id="d6d95-306">Questo articolo descrive come configurare le cache distribuite SQL Server e Redis.</span><span class="sxs-lookup"><span data-stu-id="d6d95-306">This article describes how to configure SQL Server and Redis distributed caches.</span></span> <span data-ttu-id="d6d95-307">Sono disponibili anche implementazioni di terze parti, ad esempio [NCache](http://www.alachisoft.com/ncache/aspnet-core-idistributedcache-ncache.html) ([NCache su GitHub](https://github.com/Alachisoft/NCache)).</span><span class="sxs-lookup"><span data-stu-id="d6d95-307">Third party implementations are also available, such as [NCache](http://www.alachisoft.com/ncache/aspnet-core-idistributedcache-ncache.html) ([NCache on GitHub](https://github.com/Alachisoft/NCache)).</span></span> <span data-ttu-id="d6d95-308">Indipendentemente dall'implementazione selezionata, l'app interagisce con la cache usando l' <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> interfaccia.</span><span class="sxs-lookup"><span data-stu-id="d6d95-308">Regardless of which implementation is selected, the app interacts with the cache using the <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> interface.</span></span>

<span data-ttu-id="d6d95-309">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/performance/caching/distributed/samples/) ([procedura per il download](xref:index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="d6d95-309">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/performance/caching/distributed/samples/) ([how to download](xref:index#how-to-download-a-sample))</span></span>

## <a name="prerequisites"></a><span data-ttu-id="d6d95-310">Prerequisiti</span><span class="sxs-lookup"><span data-stu-id="d6d95-310">Prerequisites</span></span>

<span data-ttu-id="d6d95-311">Per usare una cache distribuita SQL Server, fare riferimento al [metapacchetto Microsoft. AspNetCore. app](xref:fundamentals/metapackage-app) o aggiungere un riferimento al pacchetto [Microsoft. Extensions. Caching. SqlServer](https://www.nuget.org/packages/Microsoft.Extensions.Caching.SqlServer) .</span><span class="sxs-lookup"><span data-stu-id="d6d95-311">To use a SQL Server distributed cache, reference the [Microsoft.AspNetCore.App metapackage](xref:fundamentals/metapackage-app) or add a package reference to the [Microsoft.Extensions.Caching.SqlServer](https://www.nuget.org/packages/Microsoft.Extensions.Caching.SqlServer) package.</span></span>

<span data-ttu-id="d6d95-312">Per usare una cache distribuita Redis, fare riferimento al [metapacchetto Microsoft. AspNetCore. app](xref:fundamentals/metapackage-app) e aggiungere un riferimento al pacchetto [Microsoft. Extensions. Caching. Redis](https://www.nuget.org/packages/Microsoft.Extensions.Caching.Redis) .</span><span class="sxs-lookup"><span data-stu-id="d6d95-312">To use a Redis distributed cache, reference the [Microsoft.AspNetCore.App metapackage](xref:fundamentals/metapackage-app) and add a package reference to the [Microsoft.Extensions.Caching.Redis](https://www.nuget.org/packages/Microsoft.Extensions.Caching.Redis) package.</span></span> <span data-ttu-id="d6d95-313">Il pacchetto Redis non è incluso nel `Microsoft.AspNetCore.App` pacchetto, quindi è necessario fare riferimento al pacchetto Redis separatamente nel file di progetto.</span><span class="sxs-lookup"><span data-stu-id="d6d95-313">The Redis package isn't included in the `Microsoft.AspNetCore.App` package, so you must reference the Redis package separately in your project file.</span></span>

<span data-ttu-id="d6d95-314">Per usare la cache distribuita NCache, fare riferimento al [metapacchetto Microsoft. AspNetCore. app](xref:fundamentals/metapackage-app) e aggiungere un riferimento al pacchetto [NCache. Microsoft. Extensions. Caching. opensource](https://www.nuget.org/packages/NCache.Microsoft.Extensions.Caching.OpenSource) .</span><span class="sxs-lookup"><span data-stu-id="d6d95-314">To use NCache distributed cache, reference the [Microsoft.AspNetCore.App metapackage](xref:fundamentals/metapackage-app) and add a package reference to the [NCache.Microsoft.Extensions.Caching.OpenSource](https://www.nuget.org/packages/NCache.Microsoft.Extensions.Caching.OpenSource) package.</span></span> <span data-ttu-id="d6d95-315">Il pacchetto NCache non è incluso nel `Microsoft.AspNetCore.App` pacchetto, quindi è necessario fare riferimento al pacchetto NCache separatamente nel file di progetto.</span><span class="sxs-lookup"><span data-stu-id="d6d95-315">The NCache package isn't included in the `Microsoft.AspNetCore.App` package, so you must reference the NCache package separately in your project file.</span></span>

## <a name="idistributedcache-interface"></a><span data-ttu-id="d6d95-316">Interfaccia IDistributedCache</span><span class="sxs-lookup"><span data-stu-id="d6d95-316">IDistributedCache interface</span></span>

<span data-ttu-id="d6d95-317">L' <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> interfaccia fornisce i metodi seguenti per modificare gli elementi nell'implementazione della cache distribuita:</span><span class="sxs-lookup"><span data-stu-id="d6d95-317">The <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> interface provides the following methods to manipulate items in the distributed cache implementation:</span></span>

* <span data-ttu-id="d6d95-318"><xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache.Get*>, <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache.GetAsync*> : Accetta una chiave di stringa e recupera un elemento memorizzato nella cache come `byte[]` matrice se viene trovato nella cache.</span><span class="sxs-lookup"><span data-stu-id="d6d95-318"><xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache.Get*>, <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache.GetAsync*>: Accepts a string key and retrieves a cached item as a `byte[]` array if found in the cache.</span></span>
* <span data-ttu-id="d6d95-319"><xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache.Set*>, <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache.SetAsync*> : Aggiunge un elemento (come `byte[]` matrice) alla cache usando una chiave di stringa.</span><span class="sxs-lookup"><span data-stu-id="d6d95-319"><xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache.Set*>, <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache.SetAsync*>: Adds an item (as `byte[]` array) to the cache using a string key.</span></span>
* <span data-ttu-id="d6d95-320"><xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache.Refresh*>, <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache.RefreshAsync*> : Aggiorna un elemento nella cache in base alla relativa chiave, reimpostando il timeout di scadenza variabile (se presente).</span><span class="sxs-lookup"><span data-stu-id="d6d95-320"><xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache.Refresh*>, <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache.RefreshAsync*>: Refreshes an item in the cache based on its key, resetting its sliding expiration timeout (if any).</span></span>
* <span data-ttu-id="d6d95-321"><xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache.Remove*>, <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache.RemoveAsync*> : Rimuove un elemento della cache in base alla relativa chiave di stringa.</span><span class="sxs-lookup"><span data-stu-id="d6d95-321"><xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache.Remove*>, <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache.RemoveAsync*>: Removes a cache item based on its string key.</span></span>

## <a name="establish-distributed-caching-services"></a><span data-ttu-id="d6d95-322">Stabilire servizi di memorizzazione nella cache distribuiti</span><span class="sxs-lookup"><span data-stu-id="d6d95-322">Establish distributed caching services</span></span>

<span data-ttu-id="d6d95-323">Registrare un'implementazione di <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> in `Startup.ConfigureServices` .</span><span class="sxs-lookup"><span data-stu-id="d6d95-323">Register an implementation of <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> in `Startup.ConfigureServices`.</span></span> <span data-ttu-id="d6d95-324">Le implementazioni fornite dal Framework descritte in questo argomento includono:</span><span class="sxs-lookup"><span data-stu-id="d6d95-324">Framework-provided implementations described in this topic include:</span></span>

* [<span data-ttu-id="d6d95-325">Cache di memoria distribuita</span><span class="sxs-lookup"><span data-stu-id="d6d95-325">Distributed Memory Cache</span></span>](#distributed-memory-cache)
* [<span data-ttu-id="d6d95-326">Cache SQL Server distribuita</span><span class="sxs-lookup"><span data-stu-id="d6d95-326">Distributed SQL Server cache</span></span>](#distributed-sql-server-cache)
* [<span data-ttu-id="d6d95-327">Cache Redis distribuita</span><span class="sxs-lookup"><span data-stu-id="d6d95-327">Distributed Redis cache</span></span>](#distributed-redis-cache)
* [<span data-ttu-id="d6d95-328">Cache NCache distribuita</span><span class="sxs-lookup"><span data-stu-id="d6d95-328">Distributed NCache cache</span></span>](#distributed-ncache-cache)

### <a name="distributed-memory-cache"></a><span data-ttu-id="d6d95-329">Cache di memoria distribuita</span><span class="sxs-lookup"><span data-stu-id="d6d95-329">Distributed Memory Cache</span></span>

<span data-ttu-id="d6d95-330">La cache di memoria distribuita ( <xref:Microsoft.Extensions.DependencyInjection.MemoryCacheServiceCollectionExtensions.AddDistributedMemoryCache*> ) è un'implementazione fornita dal framework di <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> che archivia gli elementi in memoria.</span><span class="sxs-lookup"><span data-stu-id="d6d95-330">The Distributed Memory Cache (<xref:Microsoft.Extensions.DependencyInjection.MemoryCacheServiceCollectionExtensions.AddDistributedMemoryCache*>) is a framework-provided implementation of <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> that stores items in memory.</span></span> <span data-ttu-id="d6d95-331">La cache di memoria distribuita non è una cache distribuita effettiva.</span><span class="sxs-lookup"><span data-stu-id="d6d95-331">The Distributed Memory Cache isn't an actual distributed cache.</span></span> <span data-ttu-id="d6d95-332">Gli elementi memorizzati nella cache vengono archiviati dall'istanza dell'app nel server in cui è in esecuzione l'app.</span><span class="sxs-lookup"><span data-stu-id="d6d95-332">Cached items are stored by the app instance on the server where the app is running.</span></span>

<span data-ttu-id="d6d95-333">La cache di memoria distribuita è un'implementazione utile:</span><span class="sxs-lookup"><span data-stu-id="d6d95-333">The Distributed Memory Cache is a useful implementation:</span></span>

* <span data-ttu-id="d6d95-334">Negli scenari di sviluppo e test.</span><span class="sxs-lookup"><span data-stu-id="d6d95-334">In development and testing scenarios.</span></span>
* <span data-ttu-id="d6d95-335">Quando un singolo server viene utilizzato nell'ambiente di produzione e l'utilizzo della memoria non costituisce un problema.</span><span class="sxs-lookup"><span data-stu-id="d6d95-335">When a single server is used in production and memory consumption isn't an issue.</span></span> <span data-ttu-id="d6d95-336">L'implementazione della cache di memoria distribuita astrae l'archiviazione dei dati memorizzati nella cache.</span><span class="sxs-lookup"><span data-stu-id="d6d95-336">Implementing the Distributed Memory Cache abstracts cached data storage.</span></span> <span data-ttu-id="d6d95-337">Consente di implementare in futuro una soluzione di memorizzazione nella cache distribuita se più nodi o tolleranza di errore diventano necessari.</span><span class="sxs-lookup"><span data-stu-id="d6d95-337">It allows for implementing a true distributed caching solution in the future if multiple nodes or fault tolerance become necessary.</span></span>

<span data-ttu-id="d6d95-338">L'app di esempio usa la cache di memoria distribuita quando l'app viene eseguita nell'ambiente di sviluppo in `Startup.ConfigureServices` :</span><span class="sxs-lookup"><span data-stu-id="d6d95-338">The sample app makes use of the Distributed Memory Cache when the app is run in the Development environment in `Startup.ConfigureServices`:</span></span>

[!code-csharp[](distributed/samples/2.x/DistCacheSample/Startup.cs?name=snippet_AddDistributedMemoryCache)]

### <a name="distributed-sql-server-cache"></a><span data-ttu-id="d6d95-339">Cache SQL Server distribuita</span><span class="sxs-lookup"><span data-stu-id="d6d95-339">Distributed SQL Server Cache</span></span>

<span data-ttu-id="d6d95-340">L'implementazione della cache SQL Server distribuita ( <xref:Microsoft.Extensions.DependencyInjection.SqlServerCachingServicesExtensions.AddDistributedSqlServerCache*> ) consente alla cache distribuita di usare un database SQL Server come archivio di backup.</span><span class="sxs-lookup"><span data-stu-id="d6d95-340">The Distributed SQL Server Cache implementation (<xref:Microsoft.Extensions.DependencyInjection.SqlServerCachingServicesExtensions.AddDistributedSqlServerCache*>) allows the distributed cache to use a SQL Server database as its backing store.</span></span> <span data-ttu-id="d6d95-341">Per creare una tabella SQL Server elemento memorizzato nella cache in un'istanza di SQL Server, è possibile utilizzare lo `sql-cache` strumento.</span><span class="sxs-lookup"><span data-stu-id="d6d95-341">To create a SQL Server cached item table in a SQL Server instance, you can use the `sql-cache` tool.</span></span> <span data-ttu-id="d6d95-342">Lo strumento crea una tabella con il nome e lo schema specificati.</span><span class="sxs-lookup"><span data-stu-id="d6d95-342">The tool creates a table with the name and schema that you specify.</span></span>

<span data-ttu-id="d6d95-343">Creare una tabella in SQL Server eseguendo il `sql-cache create` comando.</span><span class="sxs-lookup"><span data-stu-id="d6d95-343">Create a table in SQL Server by running the `sql-cache create` command.</span></span> <span data-ttu-id="d6d95-344">Fornire l'istanza di SQL Server ( `Data Source` ), il database ( `Initial Catalog` ), lo schema (ad esempio, `dbo` ) e il nome della tabella (ad esempio, `TestCache` ):</span><span class="sxs-lookup"><span data-stu-id="d6d95-344">Provide the SQL Server instance (`Data Source`), database (`Initial Catalog`), schema (for example, `dbo`), and table name (for example, `TestCache`):</span></span>

```dotnetcli
dotnet sql-cache create "Data Source=(localdb)\MSSQLLocalDB;Initial Catalog=DistCache;Integrated Security=True;" dbo TestCache
```

<span data-ttu-id="d6d95-345">Viene registrato un messaggio per indicare che lo strumento è stato completato correttamente:</span><span class="sxs-lookup"><span data-stu-id="d6d95-345">A message is logged to indicate that the tool was successful:</span></span>

```console
Table and index were created successfully.
```

<span data-ttu-id="d6d95-346">La tabella creata dallo `sql-cache` strumento presenta lo schema seguente:</span><span class="sxs-lookup"><span data-stu-id="d6d95-346">The table created by the `sql-cache` tool has the following schema:</span></span>

![Tabella cache SqlServer](distributed/_static/SqlServerCacheTable.png)

> [!NOTE]
> <span data-ttu-id="d6d95-348">Un'app deve modificare i valori della cache usando un'istanza di <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> , non un oggetto <xref:Microsoft.Extensions.Caching.SqlServer.SqlServerCache> .</span><span class="sxs-lookup"><span data-stu-id="d6d95-348">An app should manipulate cache values using an instance of <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache>, not a <xref:Microsoft.Extensions.Caching.SqlServer.SqlServerCache>.</span></span>

<span data-ttu-id="d6d95-349">L'app di esempio implementa <xref:Microsoft.Extensions.Caching.SqlServer.SqlServerCache> in un ambiente non di sviluppo in `Startup.ConfigureServices` :</span><span class="sxs-lookup"><span data-stu-id="d6d95-349">The sample app implements <xref:Microsoft.Extensions.Caching.SqlServer.SqlServerCache> in a non-Development environment in `Startup.ConfigureServices`:</span></span>

[!code-csharp[](distributed/samples/2.x/DistCacheSample/Startup.cs?name=snippet_AddDistributedSqlServerCache)]

> [!NOTE]
> <span data-ttu-id="d6d95-350">Un <xref:Microsoft.Extensions.Caching.SqlServer.SqlServerCacheOptions.ConnectionString*> (e, facoltativamente, <xref:Microsoft.Extensions.Caching.SqlServer.SqlServerCacheOptions.SchemaName*> e <xref:Microsoft.Extensions.Caching.SqlServer.SqlServerCacheOptions.TableName*> ) vengono in genere archiviati all'esterno del controllo del codice sorgente, ad esempio archiviato dal [gestore dei segreti](xref:security/app-secrets) o in *appsettings.json* / *appSettings. { ENVIRONMENT} file JSON* ).</span><span class="sxs-lookup"><span data-stu-id="d6d95-350">A <xref:Microsoft.Extensions.Caching.SqlServer.SqlServerCacheOptions.ConnectionString*> (and optionally, <xref:Microsoft.Extensions.Caching.SqlServer.SqlServerCacheOptions.SchemaName*> and <xref:Microsoft.Extensions.Caching.SqlServer.SqlServerCacheOptions.TableName*>) are typically stored outside of source control (for example, stored by the [Secret Manager](xref:security/app-secrets) or in *appsettings.json*/*appsettings.{ENVIRONMENT}.json* files).</span></span> <span data-ttu-id="d6d95-351">La stringa di connessione può contenere credenziali che devono essere mantenute fuori dai sistemi di controllo del codice sorgente.</span><span class="sxs-lookup"><span data-stu-id="d6d95-351">The connection string may contain credentials that should be kept out of source control systems.</span></span>

### <a name="distributed-redis-cache"></a><span data-ttu-id="d6d95-352">Cache Redis distribuita</span><span class="sxs-lookup"><span data-stu-id="d6d95-352">Distributed Redis Cache</span></span>

<span data-ttu-id="d6d95-353">[Redis](https://redis.io/) è un archivio dati in memoria open source, che viene spesso usato come cache distribuita.</span><span class="sxs-lookup"><span data-stu-id="d6d95-353">[Redis](https://redis.io/) is an open source in-memory data store, which is often used as a distributed cache.</span></span> <span data-ttu-id="d6d95-354">È possibile usare Redis localmente ed è possibile configurare una [cache Redis di Azure](https://azure.microsoft.com/services/cache/) per un'app ASP.NET Core ospitata in Azure.</span><span class="sxs-lookup"><span data-stu-id="d6d95-354">You can use Redis locally, and you can configure an [Azure Redis Cache](https://azure.microsoft.com/services/cache/) for an Azure-hosted ASP.NET Core app.</span></span>

<span data-ttu-id="d6d95-355">Un'app configura l'implementazione della cache usando un' <xref:Microsoft.Extensions.Caching.Redis.RedisCache> istanza di ( <xref:Microsoft.Extensions.DependencyInjection.RedisCacheServiceCollectionExtensions.AddDistributedRedisCache*> ):</span><span class="sxs-lookup"><span data-stu-id="d6d95-355">An app configures the cache implementation using a <xref:Microsoft.Extensions.Caching.Redis.RedisCache> instance (<xref:Microsoft.Extensions.DependencyInjection.RedisCacheServiceCollectionExtensions.AddDistributedRedisCache*>):</span></span>

```csharp
services.AddDistributedRedisCache(options =>
{
    options.Configuration = "localhost";
    options.InstanceName = "SampleInstance";
});
```

<span data-ttu-id="d6d95-356">Per installare Redis nel computer locale:</span><span class="sxs-lookup"><span data-stu-id="d6d95-356">To install Redis on your local machine:</span></span>

1. <span data-ttu-id="d6d95-357">Installare il [pacchetto di cioccolaty Redis](https://chocolatey.org/packages/redis-64/).</span><span class="sxs-lookup"><span data-stu-id="d6d95-357">Install the [Chocolatey Redis package](https://chocolatey.org/packages/redis-64/).</span></span>
1. <span data-ttu-id="d6d95-358">Eseguire `redis-server` da un prompt dei comandi.</span><span class="sxs-lookup"><span data-stu-id="d6d95-358">Run `redis-server` from a command prompt.</span></span>

### <a name="distributed-ncache-cache"></a><span data-ttu-id="d6d95-359">Cache NCache distribuita</span><span class="sxs-lookup"><span data-stu-id="d6d95-359">Distributed NCache Cache</span></span>

<span data-ttu-id="d6d95-360">[NCache](https://github.com/Alachisoft/NCache) è una cache distribuita in memoria open source sviluppata in modo nativo in .NET e .NET Core.</span><span class="sxs-lookup"><span data-stu-id="d6d95-360">[NCache](https://github.com/Alachisoft/NCache) is an open source in-memory distributed cache developed natively in .NET and .NET Core.</span></span> <span data-ttu-id="d6d95-361">NCache funziona sia localmente che configurato come cluster di cache distribuita per un'app ASP.NET Core in esecuzione in Azure o in altre piattaforme di hosting.</span><span class="sxs-lookup"><span data-stu-id="d6d95-361">NCache works both locally and configured as a distributed cache cluster for an ASP.NET Core app running in Azure or on other hosting platforms.</span></span>

<span data-ttu-id="d6d95-362">Per installare e configurare NCache nel computer locale, vedere la [guida introduzione per Windows (.NET e .NET Core)](https://www.alachisoft.com/resources/docs/ncache/getting-started-guide-windows/).</span><span class="sxs-lookup"><span data-stu-id="d6d95-362">To install and configure NCache on your local machine, see [Getting Started Guide for Windows (.NET and .NET Core)](https://www.alachisoft.com/resources/docs/ncache/getting-started-guide-windows/).</span></span>

<span data-ttu-id="d6d95-363">Per configurare NCache:</span><span class="sxs-lookup"><span data-stu-id="d6d95-363">To configure NCache:</span></span>

1. <span data-ttu-id="d6d95-364">Installare [NCache Open Source NuGet](https://www.nuget.org/packages/Alachisoft.NCache.OpenSource.SDK/).</span><span class="sxs-lookup"><span data-stu-id="d6d95-364">Install [NCache open source NuGet](https://www.nuget.org/packages/Alachisoft.NCache.OpenSource.SDK/).</span></span>
1. <span data-ttu-id="d6d95-365">Configurare il cluster di cache in [client. ncconf](https://www.alachisoft.com/resources/docs/ncache/admin-guide/client-config.html).</span><span class="sxs-lookup"><span data-stu-id="d6d95-365">Configure the cache cluster in [client.ncconf](https://www.alachisoft.com/resources/docs/ncache/admin-guide/client-config.html).</span></span>
1. <span data-ttu-id="d6d95-366">Aggiungere il codice seguente a `Startup.ConfigureServices`:</span><span class="sxs-lookup"><span data-stu-id="d6d95-366">Add the following code to `Startup.ConfigureServices`:</span></span>

   ```csharp
   services.AddNCacheDistributedCache(configuration =>    
   {        
       configuration.CacheName = "demoClusteredCache";
       configuration.EnableLogs = true;
       configuration.ExceptionsEnabled = true;
   });
   ```

## <a name="use-the-distributed-cache"></a><span data-ttu-id="d6d95-367">Usare la cache distribuita</span><span class="sxs-lookup"><span data-stu-id="d6d95-367">Use the distributed cache</span></span>

<span data-ttu-id="d6d95-368">Per usare l' <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> interfaccia, richiedere un'istanza di <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> da qualsiasi costruttore nell'app.</span><span class="sxs-lookup"><span data-stu-id="d6d95-368">To use the <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> interface, request an instance of <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> from any constructor in the app.</span></span> <span data-ttu-id="d6d95-369">L'istanza viene fornita da [inserimento delle dipendenze (di)](xref:fundamentals/dependency-injection).</span><span class="sxs-lookup"><span data-stu-id="d6d95-369">The instance is provided by [dependency injection (DI)](xref:fundamentals/dependency-injection).</span></span>

<span data-ttu-id="d6d95-370">Quando l'app di esempio viene avviata, <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> viene inserita in `Startup.Configure` .</span><span class="sxs-lookup"><span data-stu-id="d6d95-370">When the sample app starts, <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> is injected into `Startup.Configure`.</span></span> <span data-ttu-id="d6d95-371">L'ora corrente viene memorizzata nella cache usando <xref:Microsoft.AspNetCore.Hosting.IApplicationLifetime> (per altre informazioni, vedere [Web host: interfaccia IApplicationLifetime](xref:fundamentals/host/web-host#iapplicationlifetime-interface)):</span><span class="sxs-lookup"><span data-stu-id="d6d95-371">The current time is cached using <xref:Microsoft.AspNetCore.Hosting.IApplicationLifetime> (for more information, see [Web Host: IApplicationLifetime interface](xref:fundamentals/host/web-host#iapplicationlifetime-interface)):</span></span>

[!code-csharp[](distributed/samples/2.x/DistCacheSample/Startup.cs?name=snippet_Configure&highlight=10)]

<span data-ttu-id="d6d95-372">L'app di esempio inserisce <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> nell'oggetto `IndexModel` per l'uso da parte della pagina di indice.</span><span class="sxs-lookup"><span data-stu-id="d6d95-372">The sample app injects <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> into the `IndexModel` for use by the Index page.</span></span>

<span data-ttu-id="d6d95-373">Ogni volta che viene caricata la pagina di indice, viene verificata la presenza dell'ora memorizzata nella cache in `OnGetAsync` .</span><span class="sxs-lookup"><span data-stu-id="d6d95-373">Each time the Index page is loaded, the cache is checked for the cached time in `OnGetAsync`.</span></span> <span data-ttu-id="d6d95-374">Se il tempo di memorizzazione nella cache non è scaduto, viene visualizzata l'ora.</span><span class="sxs-lookup"><span data-stu-id="d6d95-374">If the cached time hasn't expired, the time is displayed.</span></span> <span data-ttu-id="d6d95-375">Se sono trascorsi 20 secondi dall'ultima volta in cui è stato eseguito l'accesso all'ora memorizzata nella cache (l'ultima volta che questa pagina è stata caricata), la pagina Visualizza l' *ora della cache scaduta*.</span><span class="sxs-lookup"><span data-stu-id="d6d95-375">If 20 seconds have elapsed since the last time the cached time was accessed (the last time this page was loaded), the page displays *Cached Time Expired*.</span></span>

<span data-ttu-id="d6d95-376">Aggiornare immediatamente l'ora memorizzata nella cache all'ora corrente selezionando il pulsante **Reimposta tempo memorizzato nella cache** .</span><span class="sxs-lookup"><span data-stu-id="d6d95-376">Immediately update the cached time to the current time by selecting the **Reset Cached Time** button.</span></span> <span data-ttu-id="d6d95-377">Il pulsante attiva il `OnPostResetCachedTime` metodo del gestore.</span><span class="sxs-lookup"><span data-stu-id="d6d95-377">The button triggers the `OnPostResetCachedTime` handler method.</span></span>

[!code-csharp[](distributed/samples/2.x/DistCacheSample/Pages/Index.cshtml.cs?name=snippet_IndexModel&highlight=7,14-20,25-29)]

> [!NOTE]
> <span data-ttu-id="d6d95-378">Non è necessario usare un singleton o una durata con ambito per <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> le istanze (almeno per le implementazioni predefinite).</span><span class="sxs-lookup"><span data-stu-id="d6d95-378">There's no need to use a Singleton or Scoped lifetime for <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> instances (at least for the built-in implementations).</span></span>
>
> <span data-ttu-id="d6d95-379">È anche possibile creare un' <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> istanza di ogni volta che è necessario invece di usare, ma la creazione di un'istanza nel codice può rendere il codice più difficile da testare e violare il [principio delle dipendenze esplicite](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#explicit-dependencies).</span><span class="sxs-lookup"><span data-stu-id="d6d95-379">You can also create an <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> instance wherever you might need one instead of using DI, but creating an instance in code can make your code harder to test and violates the [Explicit Dependencies Principle](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#explicit-dependencies).</span></span>

## <a name="recommendations"></a><span data-ttu-id="d6d95-380">Consigli</span><span class="sxs-lookup"><span data-stu-id="d6d95-380">Recommendations</span></span>

<span data-ttu-id="d6d95-381">Quando si decide quale implementazione di <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> è migliore per l'app, tenere presente quanto segue:</span><span class="sxs-lookup"><span data-stu-id="d6d95-381">When deciding which implementation of <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> is best for your app, consider the following:</span></span>

* <span data-ttu-id="d6d95-382">Infrastruttura esistente</span><span class="sxs-lookup"><span data-stu-id="d6d95-382">Existing infrastructure</span></span>
* <span data-ttu-id="d6d95-383">Requisiti per le prestazioni</span><span class="sxs-lookup"><span data-stu-id="d6d95-383">Performance requirements</span></span>
* <span data-ttu-id="d6d95-384">Costo</span><span class="sxs-lookup"><span data-stu-id="d6d95-384">Cost</span></span>
* <span data-ttu-id="d6d95-385">Esperienza del team</span><span class="sxs-lookup"><span data-stu-id="d6d95-385">Team experience</span></span>

<span data-ttu-id="d6d95-386">Le soluzioni per la memorizzazione nella cache si basano in genere sull'archiviazione in memoria per consentire un rapido recupero dei dati memorizzati nella cache, ma la memoria è una risorsa limitata ed è dispendiosa per espanderla.</span><span class="sxs-lookup"><span data-stu-id="d6d95-386">Caching solutions usually rely on in-memory storage to provide fast retrieval of cached data, but memory is a limited resource and costly to expand.</span></span> <span data-ttu-id="d6d95-387">Archiviare solo i dati utilizzati di frequente in una cache.</span><span class="sxs-lookup"><span data-stu-id="d6d95-387">Only store commonly used data in a cache.</span></span>

<span data-ttu-id="d6d95-388">In genere, una cache Redis offre una velocità effettiva più elevata e una latenza inferiore rispetto a una cache SQL Server.</span><span class="sxs-lookup"><span data-stu-id="d6d95-388">Generally, a Redis cache provides higher throughput and lower latency than a SQL Server cache.</span></span> <span data-ttu-id="d6d95-389">Tuttavia, il benchmarking è in genere necessario per determinare le caratteristiche di prestazioni delle strategie di memorizzazione nella cache.</span><span class="sxs-lookup"><span data-stu-id="d6d95-389">However, benchmarking is usually required to determine the performance characteristics of caching strategies.</span></span>

<span data-ttu-id="d6d95-390">Quando SQL Server viene usato come archivio di backup della cache distribuita, l'uso dello stesso database per la cache e l'archiviazione e il recupero dei dati ordinari dell'app possono influire negativamente sulle prestazioni di entrambi.</span><span class="sxs-lookup"><span data-stu-id="d6d95-390">When SQL Server is used as a distributed cache backing store, use of the same database for the cache and the app's ordinary data storage and retrieval can negatively impact the performance of both.</span></span> <span data-ttu-id="d6d95-391">Si consiglia di usare un'istanza di SQL Server dedicata per l'archivio di backup della cache distribuita.</span><span class="sxs-lookup"><span data-stu-id="d6d95-391">We recommend using a dedicated SQL Server instance for the distributed cache backing store.</span></span>

## <a name="additional-resources"></a><span data-ttu-id="d6d95-392">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="d6d95-392">Additional resources</span></span>

* [<span data-ttu-id="d6d95-393">Cache Redis in Azure</span><span class="sxs-lookup"><span data-stu-id="d6d95-393">Redis Cache on Azure</span></span>](/azure/azure-cache-for-redis/)
* [<span data-ttu-id="d6d95-394">Database SQL in Azure</span><span class="sxs-lookup"><span data-stu-id="d6d95-394">SQL Database on Azure</span></span>](/azure/sql-database/)
* <span data-ttu-id="d6d95-395">[ASP.NET Core provider IDistributedCache per NCache in Web farm](http://www.alachisoft.com/ncache/aspnet-core-idistributedcache-ncache.html) ([NCache su GitHub](https://github.com/Alachisoft/NCache))</span><span class="sxs-lookup"><span data-stu-id="d6d95-395">[ASP.NET Core IDistributedCache Provider for NCache in Web Farms](http://www.alachisoft.com/ncache/aspnet-core-idistributedcache-ncache.html) ([NCache on GitHub](https://github.com/Alachisoft/NCache))</span></span>
* <xref:performance/caching/memory>
* <xref:fundamentals/change-tokens>
* <xref:performance/caching/response>
* <xref:performance/caching/middleware>
* <xref:mvc/views/tag-helpers/builtin-th/cache-tag-helper>
* <xref:mvc/views/tag-helpers/builtin-th/distributed-cache-tag-helper>
* <xref:host-and-deploy/web-farm>

::: moniker-end
