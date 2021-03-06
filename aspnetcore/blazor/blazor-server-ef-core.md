---
title: ASP.NET Core Blazor Server con Entity Framework Core (EFCore)
author: JeremyLikness
description: Linee guida per l'uso di EF Core nelle Blazor Server app.
monikerRange: '>= aspnetcore-3.1'
ms.author: jeliknes
ms.custom: mvc
ms.date: 08/14/2020
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
uid: blazor/blazor-server-ef-core
ms.openlocfilehash: db70bafc85311f8ccbf87e123739ea99c8d8f3c5
ms.sourcegitcommit: 1436bd4d70937d6ec3140da56d96caab33c4320b
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/06/2021
ms.locfileid: "102394577"
---
# <a name="aspnet-core-blazor-server-with-entity-framework-core-efcore"></a><span data-ttu-id="dfdd9-103">ASP.NET Core Blazor Server con Entity Framework Core (EFCore)</span><span class="sxs-lookup"><span data-stu-id="dfdd9-103">ASP.NET Core Blazor Server with Entity Framework Core (EFCore)</span></span>

:::moniker range=">= aspnetcore-5.0"

<span data-ttu-id="dfdd9-104">Blazor Server è un Framework di app con stato.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-104">Blazor Server is a stateful app framework.</span></span> <span data-ttu-id="dfdd9-105">L'app mantiene una connessione continuativa al server e lo stato dell'utente viene mantenuto nella memoria del server in un *circuito*.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-105">The app maintains an ongoing connection to the server, and the user's state is held in the server's memory in a *circuit*.</span></span> <span data-ttu-id="dfdd9-106">Un esempio di stato utente è costituito dai dati contenuti nelle istanze del servizio [di inserimento delle dipendenze](xref:fundamentals/dependency-injection) che hanno come ambito il circuito.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-106">One example of user state is data held in [dependency injection (DI)](xref:fundamentals/dependency-injection) service instances that are scoped to the circuit.</span></span> <span data-ttu-id="dfdd9-107">Il modello di applicazione univoco Blazor Server fornito da richiede un approccio speciale per utilizzare Entity Framework Core.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-107">The unique application model that Blazor Server provides requires a special approach to use Entity Framework Core.</span></span>

> [!NOTE]
> <span data-ttu-id="dfdd9-108">Questo articolo illustra EF Core nelle Blazor Server app.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-108">This article addresses EF Core in Blazor Server apps.</span></span> <span data-ttu-id="dfdd9-109">Blazor WebAssembly le app vengono eseguite in una sandbox di webassembly che impedisce la maggior parte delle connessioni di database dirette.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-109">Blazor WebAssembly apps run in a WebAssembly sandbox that prevents most direct database connections.</span></span> <span data-ttu-id="dfdd9-110">L'esecuzione di EF Core in Blazor WebAssembly esula dall'ambito di questo articolo.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-110">Running EF Core in Blazor WebAssembly is beyond the scope of this article.</span></span>

<h2 id="sample-app-5x"><span data-ttu-id="dfdd9-111">App di esempio</span><span class="sxs-lookup"><span data-stu-id="dfdd9-111">Sample app</span></span></h2>

<span data-ttu-id="dfdd9-112">L'app di esempio è stata creata come riferimento per le Blazor Server app che usano EF core.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-112">The sample app was built as a reference for Blazor Server apps that use EF Core.</span></span> <span data-ttu-id="dfdd9-113">L'app di esempio include una griglia con operazioni di ordinamento e filtro, eliminazione, aggiunta e aggiornamento.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-113">The sample app includes a grid with sorting and filtering, delete, add, and update operations.</span></span> <span data-ttu-id="dfdd9-114">Nell'esempio viene illustrato l'utilizzo di EF Core per gestire la concorrenza ottimistica.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-114">The sample demonstrates use of EF Core to handle optimistic concurrency.</span></span>

<span data-ttu-id="dfdd9-115">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/blazor/common/samples/5.x/BlazorServerEFCoreSample) ([procedura per il download](xref:index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="dfdd9-115">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/blazor/common/samples/5.x/BlazorServerEFCoreSample) ([how to download](xref:index#how-to-download-a-sample))</span></span>

<span data-ttu-id="dfdd9-116">Nell'esempio viene utilizzato un database [SQLite](https://www.sqlite.org/index.html) locale in modo che possa essere utilizzato su qualsiasi piattaforma.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-116">The sample uses a local [SQLite](https://www.sqlite.org/index.html) database so that it can be used on any platform.</span></span> <span data-ttu-id="dfdd9-117">Nell'esempio viene inoltre configurata la registrazione del database per visualizzare le query SQL generate.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-117">The sample also configures database logging to show the SQL queries that are generated.</span></span> <span data-ttu-id="dfdd9-118">Questa configurazione è configurata in `appsettings.Development.json` :</span><span class="sxs-lookup"><span data-stu-id="dfdd9-118">This is configured in `appsettings.Development.json`:</span></span>

[!code-json[](~/blazor/common/samples/5.x/BlazorServerEFCoreSample/BlazorServerDbContextExample/appsettings.Development.json?highlight=8)]

<span data-ttu-id="dfdd9-119">I componenti Grid, Add e View utilizzano il modello "context-per-Operation", in cui viene creato un contesto per ogni operazione.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-119">The grid, add, and view components use the "context-per-operation" pattern, where a context is created for each operation.</span></span> <span data-ttu-id="dfdd9-120">Il componente Edit usa il modello "context-per-Component", in cui viene creato un contesto per ogni componente.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-120">The edit component uses the "context-per-component" pattern, where a context is created for each component.</span></span>

> [!NOTE]
> <span data-ttu-id="dfdd9-121">Per alcuni esempi di codice in questo argomento sono necessari gli spazi dei nomi e i servizi che non vengono visualizzati.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-121">Some of the code examples in this topic require namespaces and services that aren't shown.</span></span> <span data-ttu-id="dfdd9-122">Per esaminare il codice completamente funzionante, incluse le [`@using`](xref:mvc/views/razor#using) [`@inject`](xref:mvc/views/razor#inject) direttive e obbligatorie per gli Razor esempi, vedere l' [app di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/blazor/common/samples/5.x/BlazorServerEFCoreSample).</span><span class="sxs-lookup"><span data-stu-id="dfdd9-122">To inspect the fully working code, including the required [`@using`](xref:mvc/views/razor#using) and [`@inject`](xref:mvc/views/razor#inject) directives for Razor examples, see the [sample app](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/blazor/common/samples/5.x/BlazorServerEFCoreSample).</span></span>

<h2 id="database-access-5x"><span data-ttu-id="dfdd9-123">Accesso al database</span><span class="sxs-lookup"><span data-stu-id="dfdd9-123">Database access</span></span></h2>

<span data-ttu-id="dfdd9-124">EF Core si basa su un <xref:Microsoft.EntityFrameworkCore.DbContext> come mezzo per [configurare l'accesso al database](/ef/core/miscellaneous/configuring-dbcontext) e fungere da [*unità di lavoro*](https://martinfowler.com/eaaCatalog/unitOfWork.html).</span><span class="sxs-lookup"><span data-stu-id="dfdd9-124">EF Core relies on a <xref:Microsoft.EntityFrameworkCore.DbContext> as the means to [configure database access](/ef/core/miscellaneous/configuring-dbcontext) and act as a [*unit of work*](https://martinfowler.com/eaaCatalog/unitOfWork.html).</span></span> <span data-ttu-id="dfdd9-125">EF Core fornisce l' <xref:Microsoft.Extensions.DependencyInjection.EntityFrameworkServiceCollectionExtensions.AddDbContext%2A> estensione per le app ASP.NET Core che registra il contesto come servizio con *ambito* per impostazione predefinita.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-125">EF Core provides the <xref:Microsoft.Extensions.DependencyInjection.EntityFrameworkServiceCollectionExtensions.AddDbContext%2A> extension for ASP.NET Core apps that registers the context as a *scoped* service by default.</span></span> <span data-ttu-id="dfdd9-126">Nelle Blazor Server app, le registrazioni del servizio con ambito possono essere problematiche perché l'istanza è condivisa tra i componenti all'interno del circuito dell'utente.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-126">In Blazor Server apps, scoped service registrations can be problematic because the instance is shared across components within the user's circuit.</span></span> <span data-ttu-id="dfdd9-127"><xref:Microsoft.EntityFrameworkCore.DbContext> non è thread-safe e non è progettato per l'uso simultaneo.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-127"><xref:Microsoft.EntityFrameworkCore.DbContext> isn't thread safe and isn't designed for concurrent use.</span></span> <span data-ttu-id="dfdd9-128">Le durate esistenti non sono appropriate per i motivi seguenti:</span><span class="sxs-lookup"><span data-stu-id="dfdd9-128">The existing lifetimes are inappropriate for these reasons:</span></span>

* <span data-ttu-id="dfdd9-129">Il **singleton** condivide lo stato tra tutti gli utenti dell'app e comporta un uso simultaneo non appropriato.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-129">**Singleton** shares state across all users of the app and leads to inappropriate concurrent use.</span></span>
* <span data-ttu-id="dfdd9-130">Con **ambito** (impostazione predefinita) si presenta un problema simile tra i componenti per lo stesso utente.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-130">**Scoped** (the default) poses a similar issue between components for the same user.</span></span>
* <span data-ttu-id="dfdd9-131">I risultati **temporanei** in una nuova istanza per ogni richiesta; Tuttavia, poiché i componenti possono essere di lunga durata, questo risultato è un contesto di durata più lunga di quanto possa essere previsto.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-131">**Transient** results in a new instance per request; but as components can be long-lived, this results in a longer-lived context than may be intended.</span></span>

<span data-ttu-id="dfdd9-132">Le indicazioni seguenti sono progettate per offrire un approccio coerente all'uso di EF Core nelle Blazor Server app.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-132">The following recommendations are designed to provide a consistent approach to using EF Core in Blazor Server apps.</span></span>

* <span data-ttu-id="dfdd9-133">Per impostazione predefinita, è consigliabile usare un contesto per ogni operazione.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-133">By default, consider using one context per operation.</span></span> <span data-ttu-id="dfdd9-134">Il contesto è progettato per la creazione di un'istanza di overhead veloce e bassa:</span><span class="sxs-lookup"><span data-stu-id="dfdd9-134">The context is designed for fast, low overhead instantiation:</span></span>

  ```csharp
  using var context = new MyContext();

  return await context.MyEntities.ToListAsync();
  ```

* <span data-ttu-id="dfdd9-135">Usare un flag per evitare più operazioni simultanee:</span><span class="sxs-lookup"><span data-stu-id="dfdd9-135">Use a flag to prevent multiple concurrent operations:</span></span>

  ```csharp
  if (Loading)
  {
      return;
  }

  try
  {
      Loading = true;

      ...
  }
  finally
  {
      Loading = false;
  }
  ```

  <span data-ttu-id="dfdd9-136">Posizionare le operazioni dopo la `Loading = true;` riga nel `try` blocco.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-136">Place operations after the `Loading = true;` line in the `try` block.</span></span>

* <span data-ttu-id="dfdd9-137">Per le operazioni più longeve che sfruttano i vantaggi del [rilevamento delle modifiche](/ef/core/querying/tracking) EF core o del controllo della concorrenza, [Definire](/ef/core/saving/concurrency)l' [ambito del contesto per la durata del componente](#scope-to-the-component-lifetime-5x).</span><span class="sxs-lookup"><span data-stu-id="dfdd9-137">For longer-lived operations that take advantage of EF Core's [change tracking](/ef/core/querying/tracking) or [concurrency control](/ef/core/saving/concurrency), [scope the context to the lifetime of the component](#scope-to-the-component-lifetime-5x).</span></span>

<h3 id="new-dbcontext-instances-5x"><span data-ttu-id="dfdd9-138">Nuove istanze di DbContext</span><span class="sxs-lookup"><span data-stu-id="dfdd9-138">New DbContext instances</span></span></h3>

<span data-ttu-id="dfdd9-139">Il modo più rapido per creare una nuova <xref:Microsoft.EntityFrameworkCore.DbContext> istanza consiste nell'usare `new` per creare una nuova istanza.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-139">The fastest way to create a new <xref:Microsoft.EntityFrameworkCore.DbContext> instance is by using `new` to create a new instance.</span></span> <span data-ttu-id="dfdd9-140">Esistono tuttavia diversi scenari che possono richiedere la risoluzione di dipendenze aggiuntive.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-140">However, there are several scenarios that may require resolving additional dependencies.</span></span> <span data-ttu-id="dfdd9-141">Ad esempio, è possibile usare [`DbContextOptions`](/ef/core/miscellaneous/configuring-dbcontext#configuring-dbcontextoptions) per configurare il contesto.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-141">For example, you may wish to use [`DbContextOptions`](/ef/core/miscellaneous/configuring-dbcontext#configuring-dbcontextoptions) to configure the context.</span></span>

<span data-ttu-id="dfdd9-142">La soluzione consigliata per creare una nuova <xref:Microsoft.EntityFrameworkCore.DbContext> con dipendenze consiste nell'usare una factory.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-142">The recommended solution to create a new <xref:Microsoft.EntityFrameworkCore.DbContext> with dependencies is to use a factory.</span></span> <span data-ttu-id="dfdd9-143">EF Core 5,0 o versione successiva fornisce una factory predefinita per la creazione di nuovi contesti.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-143">EF Core 5.0 or later provides a built-in factory for creating new contexts.</span></span>

<span data-ttu-id="dfdd9-144">Nell'esempio seguente viene configurato [SQLite](https://www.sqlite.org/index.html) e viene abilitata la registrazione dei dati.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-144">The following example configures [SQLite](https://www.sqlite.org/index.html) and enables data logging.</span></span> <span data-ttu-id="dfdd9-145">Il codice usa un [metodo di estensione ( `AddDbContextFactory` )](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/blazor/common/samples/3.x/BlazorServerEFCoreSample/BlazorServerDbContextExample/Data/FactoryExtensions.cs) per configurare la factory di database per e fornire opzioni predefinite:</span><span class="sxs-lookup"><span data-stu-id="dfdd9-145">The code uses an [extension method (`AddDbContextFactory`)](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/blazor/common/samples/3.x/BlazorServerEFCoreSample/BlazorServerDbContextExample/Data/FactoryExtensions.cs) to configure the database factory for DI and provide default options:</span></span>

[!code-csharp[](~/blazor/common/samples/5.x/BlazorServerEFCoreSample/BlazorServerDbContextExample/Startup.cs?name=snippet1)]

<span data-ttu-id="dfdd9-146">La factory viene inserita in componenti e usata per creare nuove istanze.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-146">The factory is injected into components and used to create new instances.</span></span> <span data-ttu-id="dfdd9-147">Ad esempio: in `Pages/Index.razor`:</span><span class="sxs-lookup"><span data-stu-id="dfdd9-147">For example, in `Pages/Index.razor`:</span></span>

[!code-csharp[](~/blazor/common/samples/5.x/BlazorServerEFCoreSample/BlazorServerDbContextExample/Pages/Index.razor?name=snippet1)]

> [!NOTE]
> <span data-ttu-id="dfdd9-148">`Wrapper` è un [riferimento](xref:blazor/components/index#capture-references-to-components) al componente `GridWrapper` .</span><span class="sxs-lookup"><span data-stu-id="dfdd9-148">`Wrapper` is a [component reference](xref:blazor/components/index#capture-references-to-components) to the `GridWrapper` component.</span></span> <span data-ttu-id="dfdd9-149">Vedere il `Index` componente ( `Pages/Index.razor` ) nell' [app di esempio](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/blazor/common/samples/5.x/BlazorServerEFCoreSample/BlazorServerDbContextExample/Pages/Index.razor).</span><span class="sxs-lookup"><span data-stu-id="dfdd9-149">See the `Index` component (`Pages/Index.razor`) in the [sample app](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/blazor/common/samples/5.x/BlazorServerEFCoreSample/BlazorServerDbContextExample/Pages/Index.razor).</span></span>

<span data-ttu-id="dfdd9-150">È <xref:Microsoft.EntityFrameworkCore.DbContext> possibile creare nuove istanze con una factory che consente di configurare la stringa di connessione per `DbContext` , ad esempio quando si usa il [ Identity modello di ASP.NET Core](xref:security/authentication/customize_identity_model):</span><span class="sxs-lookup"><span data-stu-id="dfdd9-150">New <xref:Microsoft.EntityFrameworkCore.DbContext> instances can be created with a factory that allows you to configure the connection string per `DbContext`, such as when you use [ASP.NET Core's Identity model](xref:security/authentication/customize_identity_model):</span></span>

```csharp
services.AddDbContextFactory<ApplicationDbContext>(options =>
{
    options.UseSqlServer(Configuration.GetConnectionString("DefaultConnection"));
});

services.AddScoped<ApplicationDbContext>(p => 
    p.GetRequiredService<IDbContextFactory<ApplicationDbContext>>()
    .CreateDbContext());
```

<h3 id="scope-to-the-component-lifetime-5x"><span data-ttu-id="dfdd9-151">Ambito per la durata del componente</span><span class="sxs-lookup"><span data-stu-id="dfdd9-151">Scope to the component lifetime</span></span></h3>

<span data-ttu-id="dfdd9-152">È possibile creare un <xref:Microsoft.EntityFrameworkCore.DbContext> esistente per la durata di un componente.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-152">You may wish to create a <xref:Microsoft.EntityFrameworkCore.DbContext> that exists for the lifetime of a component.</span></span> <span data-ttu-id="dfdd9-153">In questo modo, è possibile usarlo come [unità di lavoro](https://martinfowler.com/eaaCatalog/unitOfWork.html) e sfruttare le funzionalità predefinite, ad esempio il rilevamento delle modifiche e la risoluzione della concorrenza.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-153">This allows you to use it as a [unit of work](https://martinfowler.com/eaaCatalog/unitOfWork.html) and take advantage of built-in features, such as change tracking and concurrency resolution.</span></span>
<span data-ttu-id="dfdd9-154">È possibile usare la factory per creare un contesto e tenerne traccia per la durata del componente.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-154">You can use the factory to create a context and track it for the lifetime of the component.</span></span> <span data-ttu-id="dfdd9-155">Innanzitutto, implementare <xref:System.IDisposable> e inserire la factory come illustrato in `Pages/EditContact.razor` :</span><span class="sxs-lookup"><span data-stu-id="dfdd9-155">First, implement <xref:System.IDisposable> and inject the factory as shown in `Pages/EditContact.razor`:</span></span>

```razor
@implements IDisposable
@inject IDbContextFactory<ContactContext> DbFactory
```

<span data-ttu-id="dfdd9-156">L'app di esempio garantisce che il contesto venga eliminato quando il componente viene eliminato:</span><span class="sxs-lookup"><span data-stu-id="dfdd9-156">The sample app ensures the context is disposed when the component is disposed:</span></span>

[!code-csharp[](~/blazor/common/samples/5.x/BlazorServerEFCoreSample/BlazorServerDbContextExample/Pages/EditContact.razor?name=snippet1)]

<span data-ttu-id="dfdd9-157">Infine, [`OnInitializedAsync`](xref:blazor/components/lifecycle) viene eseguito l'override di per creare un nuovo contesto.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-157">Finally, [`OnInitializedAsync`](xref:blazor/components/lifecycle) is overridden to create a new context.</span></span> <span data-ttu-id="dfdd9-158">Nell'app di esempio [`OnInitializedAsync`](xref:blazor/components/lifecycle) carica il contatto nello stesso metodo:</span><span class="sxs-lookup"><span data-stu-id="dfdd9-158">In the sample app, [`OnInitializedAsync`](xref:blazor/components/lifecycle) loads the contact in the same method:</span></span>

[!code-csharp[](~/blazor/common/samples/5.x/BlazorServerEFCoreSample/BlazorServerDbContextExample/Pages/EditContact.razor?name=snippet2)]

<h3 id="enable-sensitive-data-logging"><span data-ttu-id="dfdd9-159">Abilitare la registrazione dei dati sensibili</span><span class="sxs-lookup"><span data-stu-id="dfdd9-159">Enable sensitive data logging</span></span></h3>

<span data-ttu-id="dfdd9-160"><xref:Microsoft.EntityFrameworkCore.DbContextOptionsBuilder.EnableSensitiveDataLogging%2A> include i dati dell'applicazione nei messaggi di eccezione e nella registrazione del Framework.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-160"><xref:Microsoft.EntityFrameworkCore.DbContextOptionsBuilder.EnableSensitiveDataLogging%2A> includes application data in exception messages and framework logging.</span></span> <span data-ttu-id="dfdd9-161">I dati registrati possono includere i valori assegnati alle proprietà delle istanze di entità e i valori dei parametri per i comandi inviati al database.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-161">The logged data can include the values assigned to properties of entity instances and parameter values for commands sent to the database.</span></span> <span data-ttu-id="dfdd9-162">La registrazione dei dati con <xref:Microsoft.EntityFrameworkCore.DbContextOptionsBuilder.EnableSensitiveDataLogging%2A> costituisce un rischio per la **sicurezza**, in quanto può esporre password e altre informazioni personali quando registra le istruzioni SQL eseguite sul database.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-162">Logging data with <xref:Microsoft.EntityFrameworkCore.DbContextOptionsBuilder.EnableSensitiveDataLogging%2A> is a **security risk**, as it may expose passwords and other personally identifiable information (PII) when it logs SQL statements executed against the database.</span></span>

<span data-ttu-id="dfdd9-163">È consigliabile abilitare solo <xref:Microsoft.EntityFrameworkCore.DbContextOptionsBuilder.EnableSensitiveDataLogging%2A> per lo sviluppo e il testing:</span><span class="sxs-lookup"><span data-stu-id="dfdd9-163">We recommend only enabling <xref:Microsoft.EntityFrameworkCore.DbContextOptionsBuilder.EnableSensitiveDataLogging%2A> for development and testing:</span></span>

```csharp
#if DEBUG
    services.AddDbContextFactory<ContactContext>(opt =>
        opt.UseSqlite($"Data Source={nameof(ContactContext.ContactsDb)}.db")
        .EnableSensitiveDataLogging());
#else
    services.AddDbContextFactory<ContactContext>(opt =>
        opt.UseSqlite($"Data Source={nameof(ContactContext.ContactsDb)}.db"));
#endif
```

:::moniker-end

:::moniker range="< aspnetcore-5.0"

<span data-ttu-id="dfdd9-164">Blazor Server è un Framework di app con stato.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-164">Blazor Server is a stateful app framework.</span></span> <span data-ttu-id="dfdd9-165">L'app mantiene una connessione continuativa al server e lo stato dell'utente viene mantenuto nella memoria del server in un *circuito*.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-165">The app maintains an ongoing connection to the server, and the user's state is held in the server's memory in a *circuit*.</span></span> <span data-ttu-id="dfdd9-166">Un esempio di stato utente è costituito dai dati contenuti nelle istanze del servizio [di inserimento delle dipendenze](xref:fundamentals/dependency-injection) che hanno come ambito il circuito.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-166">One example of user state is data held in [dependency injection (DI)](xref:fundamentals/dependency-injection) service instances that are scoped to the circuit.</span></span> <span data-ttu-id="dfdd9-167">Il modello di applicazione univoco Blazor Server fornito da richiede un approccio speciale per utilizzare Entity Framework Core.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-167">The unique application model that Blazor Server provides requires a special approach to use Entity Framework Core.</span></span>

> [!NOTE]
> <span data-ttu-id="dfdd9-168">Questo articolo illustra EF Core nelle Blazor Server app.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-168">This article addresses EF Core in Blazor Server apps.</span></span> <span data-ttu-id="dfdd9-169">Blazor WebAssembly le app vengono eseguite in una sandbox di webassembly che impedisce la maggior parte delle connessioni di database dirette.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-169">Blazor WebAssembly apps run in a WebAssembly sandbox that prevents most direct database connections.</span></span> <span data-ttu-id="dfdd9-170">L'esecuzione di EF Core in Blazor WebAssembly esula dall'ambito di questo articolo.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-170">Running EF Core in Blazor WebAssembly is beyond the scope of this article.</span></span>

<h2 id="sample-app-3x"><span data-ttu-id="dfdd9-171">App di esempio</span><span class="sxs-lookup"><span data-stu-id="dfdd9-171">Sample app</span></span></h2>

<span data-ttu-id="dfdd9-172">L'app di esempio è stata creata come riferimento per le Blazor Server app che usano EF core.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-172">The sample app was built as a reference for Blazor Server apps that use EF Core.</span></span> <span data-ttu-id="dfdd9-173">L'app di esempio include una griglia con operazioni di ordinamento e filtro, eliminazione, aggiunta e aggiornamento.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-173">The sample app includes a grid with sorting and filtering, delete, add, and update operations.</span></span> <span data-ttu-id="dfdd9-174">Nell'esempio viene illustrato l'utilizzo di EF Core per gestire la concorrenza ottimistica.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-174">The sample demonstrates use of EF Core to handle optimistic concurrency.</span></span>

<span data-ttu-id="dfdd9-175">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/blazor/common/samples/3.x/BlazorServerEFCoreSample) ([procedura per il download](xref:index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="dfdd9-175">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/blazor/common/samples/3.x/BlazorServerEFCoreSample) ([how to download](xref:index#how-to-download-a-sample))</span></span>

<span data-ttu-id="dfdd9-176">Nell'esempio viene utilizzato un database [SQLite](https://www.sqlite.org/index.html) locale in modo che possa essere utilizzato su qualsiasi piattaforma.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-176">The sample uses a local [SQLite](https://www.sqlite.org/index.html) database so that it can be used on any platform.</span></span> <span data-ttu-id="dfdd9-177">Nell'esempio viene inoltre configurata la registrazione del database per visualizzare le query SQL generate.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-177">The sample also configures database logging to show the SQL queries that are generated.</span></span> <span data-ttu-id="dfdd9-178">Questa configurazione è configurata in `appsettings.Development.json` :</span><span class="sxs-lookup"><span data-stu-id="dfdd9-178">This is configured in `appsettings.Development.json`:</span></span>

[!code-json[](~/blazor/common/samples/3.x/BlazorServerEFCoreSample/BlazorServerDbContextExample/appsettings.Development.json?highlight=8)]

<span data-ttu-id="dfdd9-179">I componenti Grid, Add e View utilizzano il modello "context-per-Operation", in cui viene creato un contesto per ogni operazione.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-179">The grid, add, and view components use the "context-per-operation" pattern, where a context is created for each operation.</span></span> <span data-ttu-id="dfdd9-180">Il componente Edit usa il modello "context-per-Component", in cui viene creato un contesto per ogni componente.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-180">The edit component uses the "context-per-component" pattern, where a context is created for each component.</span></span>

> [!NOTE]
> <span data-ttu-id="dfdd9-181">Per alcuni esempi di codice in questo argomento sono necessari gli spazi dei nomi e i servizi che non vengono visualizzati.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-181">Some of the code examples in this topic require namespaces and services that aren't shown.</span></span> <span data-ttu-id="dfdd9-182">Per esaminare il codice completamente funzionante, incluse le [`@using`](xref:mvc/views/razor#using) [`@inject`](xref:mvc/views/razor#inject) direttive e obbligatorie per gli Razor esempi, vedere l' [app di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/blazor/common/samples/3.x/BlazorServerEFCoreSample).</span><span class="sxs-lookup"><span data-stu-id="dfdd9-182">To inspect the fully working code, including the required [`@using`](xref:mvc/views/razor#using) and [`@inject`](xref:mvc/views/razor#inject) directives for Razor examples, see the [sample app](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/blazor/common/samples/3.x/BlazorServerEFCoreSample).</span></span>

<h2 id="database-access-3x"><span data-ttu-id="dfdd9-183">Accesso al database</span><span class="sxs-lookup"><span data-stu-id="dfdd9-183">Database access</span></span></h2>

<span data-ttu-id="dfdd9-184">EF Core si basa su un <xref:Microsoft.EntityFrameworkCore.DbContext> come mezzo per [configurare l'accesso al database](/ef/core/miscellaneous/configuring-dbcontext) e fungere da [*unità di lavoro*](https://martinfowler.com/eaaCatalog/unitOfWork.html).</span><span class="sxs-lookup"><span data-stu-id="dfdd9-184">EF Core relies on a <xref:Microsoft.EntityFrameworkCore.DbContext> as the means to [configure database access](/ef/core/miscellaneous/configuring-dbcontext) and act as a [*unit of work*](https://martinfowler.com/eaaCatalog/unitOfWork.html).</span></span> <span data-ttu-id="dfdd9-185">EF Core fornisce l' <xref:Microsoft.Extensions.DependencyInjection.EntityFrameworkServiceCollectionExtensions.AddDbContext%2A> estensione per le app ASP.NET Core che registra il contesto come servizio con *ambito* per impostazione predefinita.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-185">EF Core provides the <xref:Microsoft.Extensions.DependencyInjection.EntityFrameworkServiceCollectionExtensions.AddDbContext%2A> extension for ASP.NET Core apps that registers the context as a *scoped* service by default.</span></span> <span data-ttu-id="dfdd9-186">Nelle Blazor Server app questo può essere problematico perché l'istanza è condivisa tra i componenti all'interno del circuito dell'utente.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-186">In Blazor Server apps, this can be problematic because the instance is shared across components within the user's circuit.</span></span> <span data-ttu-id="dfdd9-187"><xref:Microsoft.EntityFrameworkCore.DbContext> non è thread-safe e non è progettato per l'uso simultaneo.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-187"><xref:Microsoft.EntityFrameworkCore.DbContext> isn't thread safe and isn't designed for concurrent use.</span></span> <span data-ttu-id="dfdd9-188">Le durate esistenti non sono appropriate per i motivi seguenti:</span><span class="sxs-lookup"><span data-stu-id="dfdd9-188">The existing lifetimes are inappropriate for these reasons:</span></span>

* <span data-ttu-id="dfdd9-189">Il **singleton** condivide lo stato tra tutti gli utenti dell'app e comporta un uso simultaneo non appropriato.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-189">**Singleton** shares state across all users of the app and leads to inappropriate concurrent use.</span></span>
* <span data-ttu-id="dfdd9-190">Con **ambito** (impostazione predefinita) si presenta un problema simile tra i componenti per lo stesso utente.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-190">**Scoped** (the default) poses a similar issue between components for the same user.</span></span>
* <span data-ttu-id="dfdd9-191">I risultati **temporanei** in una nuova istanza per ogni richiesta; Tuttavia, poiché i componenti possono essere di lunga durata, questo risultato è un contesto di durata più lunga di quanto possa essere previsto.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-191">**Transient** results in a new instance per request; but as components can be long-lived, this results in a longer-lived context than may be intended.</span></span>

<span data-ttu-id="dfdd9-192">Le indicazioni seguenti sono progettate per offrire un approccio coerente all'uso di EF Core nelle Blazor Server app.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-192">The following recommendations are designed to provide a consistent approach to using EF Core in Blazor Server apps.</span></span>

* <span data-ttu-id="dfdd9-193">Per impostazione predefinita, è consigliabile usare un contesto per ogni operazione.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-193">By default, consider using one context per operation.</span></span> <span data-ttu-id="dfdd9-194">Il contesto è progettato per la creazione di un'istanza di overhead veloce e bassa:</span><span class="sxs-lookup"><span data-stu-id="dfdd9-194">The context is designed for fast, low overhead instantiation:</span></span>

  ```csharp
  using var context = new MyContext();

  return await context.MyEntities.ToListAsync();
  ```

* <span data-ttu-id="dfdd9-195">Usare un flag per evitare più operazioni simultanee:</span><span class="sxs-lookup"><span data-stu-id="dfdd9-195">Use a flag to prevent multiple concurrent operations:</span></span>

  ```csharp
  if (Loading)
  {
      return;
  }

  try
  {
      Loading = true;

      ...
  }
  finally
  {
      Loading = false;
  }
  ```

  <span data-ttu-id="dfdd9-196">Posizionare le operazioni dopo la `Loading = true;` riga nel `try` blocco.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-196">Place operations after the `Loading = true;` line in the `try` block.</span></span>

* <span data-ttu-id="dfdd9-197">Per le operazioni più longeve che sfruttano i vantaggi del [rilevamento delle modifiche](/ef/core/querying/tracking) EF core o del controllo della concorrenza, [Definire](/ef/core/saving/concurrency)l' [ambito del contesto per la durata del componente](#scope-to-the-component-lifetime-3x).</span><span class="sxs-lookup"><span data-stu-id="dfdd9-197">For longer-lived operations that take advantage of EF Core's [change tracking](/ef/core/querying/tracking) or [concurrency control](/ef/core/saving/concurrency), [scope the context to the lifetime of the component](#scope-to-the-component-lifetime-3x).</span></span>

<h3 id="new-dbcontext-instances-3x"><span data-ttu-id="dfdd9-198">Nuove istanze di DbContext</span><span class="sxs-lookup"><span data-stu-id="dfdd9-198">New DbContext instances</span></span></h3>

<span data-ttu-id="dfdd9-199">Il modo più rapido per creare una nuova <xref:Microsoft.EntityFrameworkCore.DbContext> istanza consiste nell'usare `new` per creare una nuova istanza.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-199">The fastest way to create a new <xref:Microsoft.EntityFrameworkCore.DbContext> instance is by using `new` to create a new instance.</span></span> <span data-ttu-id="dfdd9-200">Esistono tuttavia diversi scenari che possono richiedere la risoluzione di dipendenze aggiuntive.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-200">However, there are several scenarios that may require resolving additional dependencies.</span></span> <span data-ttu-id="dfdd9-201">Ad esempio, è possibile usare [`DbContextOptions`](/ef/core/miscellaneous/configuring-dbcontext#configuring-dbcontextoptions) per configurare il contesto.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-201">For example, you may wish to use [`DbContextOptions`](/ef/core/miscellaneous/configuring-dbcontext#configuring-dbcontextoptions) to configure the context.</span></span>

<span data-ttu-id="dfdd9-202">La soluzione consigliata per creare una nuova <xref:Microsoft.EntityFrameworkCore.DbContext> con dipendenze consiste nell'usare una factory.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-202">The recommended solution to create a new <xref:Microsoft.EntityFrameworkCore.DbContext> with dependencies is to use a factory.</span></span> <span data-ttu-id="dfdd9-203">L'app di esempio implementa la propria factory in `Data/DbContextFactory.cs` .</span><span class="sxs-lookup"><span data-stu-id="dfdd9-203">The sample app implements its own factory in `Data/DbContextFactory.cs`.</span></span>

[!code-csharp[](~/blazor/common/samples/3.x/BlazorServerEFCoreSample/BlazorServerDbContextExample/Data/DbContextFactory.cs)]

<span data-ttu-id="dfdd9-204">Nella Factory precedente:</span><span class="sxs-lookup"><span data-stu-id="dfdd9-204">In the preceding factory:</span></span>

* <span data-ttu-id="dfdd9-205"><xref:Microsoft.Extensions.DependencyInjection.ActivatorUtilities.CreateInstance%2A?displayProperty=nameWithType> soddisfa tutte le dipendenze tramite il provider di servizi.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-205"><xref:Microsoft.Extensions.DependencyInjection.ActivatorUtilities.CreateInstance%2A?displayProperty=nameWithType> satisfies any dependencies via the service provider.</span></span>
* <span data-ttu-id="dfdd9-206">`IDbContextFactory` è disponibile in EF Core ASP.NET Core 5,0 o versione successiva, quindi l'interfaccia viene [implementata nell'app di esempio per ASP.NET Core 3. x](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/blazor/common/samples/3.x/BlazorServerEFCoreSample/BlazorServerDbContextExample/Data/IDbContextFactory.cs).</span><span class="sxs-lookup"><span data-stu-id="dfdd9-206">`IDbContextFactory` is available in EF Core ASP.NET Core 5.0 or later, so the interface is [implemented in the sample app for ASP.NET Core 3.x](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/blazor/common/samples/3.x/BlazorServerEFCoreSample/BlazorServerDbContextExample/Data/IDbContextFactory.cs).</span></span>

<span data-ttu-id="dfdd9-207">Nell'esempio seguente viene configurato [SQLite](https://www.sqlite.org/index.html) e viene abilitata la registrazione dei dati.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-207">The following example configures [SQLite](https://www.sqlite.org/index.html) and enables data logging.</span></span> <span data-ttu-id="dfdd9-208">Il codice usa un metodo di estensione per configurare la factory di database per e fornire opzioni predefinite:</span><span class="sxs-lookup"><span data-stu-id="dfdd9-208">The code uses an extension method to configure the database factory for DI and provide default options:</span></span>

[!code-csharp[](~/blazor/common/samples/3.x/BlazorServerEFCoreSample/BlazorServerDbContextExample/Startup.cs?name=snippet1)]

<span data-ttu-id="dfdd9-209">La factory viene inserita in componenti e usata per creare nuove istanze.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-209">The factory is injected into components and used to create new instances.</span></span> <span data-ttu-id="dfdd9-210">Ad esempio: in `Pages/Index.razor`:</span><span class="sxs-lookup"><span data-stu-id="dfdd9-210">For example, in `Pages/Index.razor`:</span></span>

[!code-csharp[](~/blazor/common/samples/3.x/BlazorServerEFCoreSample/BlazorServerDbContextExample/Pages/Index.razor?name=snippet1)]

> [!NOTE]
> <span data-ttu-id="dfdd9-211">`Wrapper` è un [riferimento](xref:blazor/components/index#capture-references-to-components) al componente `GridWrapper` .</span><span class="sxs-lookup"><span data-stu-id="dfdd9-211">`Wrapper` is a [component reference](xref:blazor/components/index#capture-references-to-components) to the `GridWrapper` component.</span></span> <span data-ttu-id="dfdd9-212">Vedere il `Index` componente ( `Pages/Index.razor` ) nell' [app di esempio](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/blazor/common/samples/3.x/BlazorServerEFCoreSample/BlazorServerDbContextExample/Pages/Index.razor).</span><span class="sxs-lookup"><span data-stu-id="dfdd9-212">See the `Index` component (`Pages/Index.razor`) in the [sample app](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/blazor/common/samples/3.x/BlazorServerEFCoreSample/BlazorServerDbContextExample/Pages/Index.razor).</span></span>

<span data-ttu-id="dfdd9-213">È <xref:Microsoft.EntityFrameworkCore.DbContext> possibile creare nuove istanze con una factory che consente di configurare la stringa di connessione per `DbContext` , ad esempio quando si usa [modello di ASP.NET Core Identity ]) (xrif: Security/Authentication/customize_identity_model):</span><span class="sxs-lookup"><span data-stu-id="dfdd9-213">New <xref:Microsoft.EntityFrameworkCore.DbContext> instances can be created with a factory that allows you to configure the connection string per `DbContext`, such as when you use [ASP.NET Core's Identity model])(xref:security/authentication/customize_identity_model):</span></span>

```csharp
services.AddDbContextFactory<ApplicationDbContext>(options =>
{
    options.UseSqlServer(Configuration.GetConnectionString("DefaultConnection"));
});

services.AddScoped<ApplicationDbContext>(p => 
    p.GetRequiredService<IDbContextFactory<ApplicationDbContext>>()
    .CreateDbContext());
```

<h3 id="scope-to-the-component-lifetime-3x"><span data-ttu-id="dfdd9-214">Ambito per la durata del componente</span><span class="sxs-lookup"><span data-stu-id="dfdd9-214">Scope to the component lifetime</span></span></h3>

<span data-ttu-id="dfdd9-215">È possibile creare un <xref:Microsoft.EntityFrameworkCore.DbContext> esistente per la durata di un componente.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-215">You may wish to create a <xref:Microsoft.EntityFrameworkCore.DbContext> that exists for the lifetime of a component.</span></span> <span data-ttu-id="dfdd9-216">In questo modo, è possibile usarlo come [unità di lavoro](https://martinfowler.com/eaaCatalog/unitOfWork.html) e sfruttare le funzionalità predefinite, ad esempio il rilevamento delle modifiche e la risoluzione della concorrenza.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-216">This allows you to use it as a [unit of work](https://martinfowler.com/eaaCatalog/unitOfWork.html) and take advantage of built-in features, such as change tracking and concurrency resolution.</span></span>
<span data-ttu-id="dfdd9-217">È possibile usare la factory per creare un contesto e tenerne traccia per la durata del componente.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-217">You can use the factory to create a context and track it for the lifetime of the component.</span></span> <span data-ttu-id="dfdd9-218">Innanzitutto, implementare <xref:System.IDisposable> e inserire la factory come illustrato in `Pages/EditContact.razor` :</span><span class="sxs-lookup"><span data-stu-id="dfdd9-218">First, implement <xref:System.IDisposable> and inject the factory as shown in `Pages/EditContact.razor`:</span></span>

```razor
@implements IDisposable
@inject IDbContextFactory<ContactContext> DbFactory
```

<span data-ttu-id="dfdd9-219">L'app di esempio garantisce che il contesto venga eliminato quando il componente viene eliminato:</span><span class="sxs-lookup"><span data-stu-id="dfdd9-219">The sample app ensures the context is disposed when the component is disposed:</span></span>

[!code-csharp[](~/blazor/common/samples/3.x/BlazorServerEFCoreSample/BlazorServerDbContextExample/Pages/EditContact.razor?name=snippet1)]

<span data-ttu-id="dfdd9-220">Infine, [`OnInitializedAsync`](xref:blazor/components/lifecycle) viene eseguito l'override di per creare un nuovo contesto.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-220">Finally, [`OnInitializedAsync`](xref:blazor/components/lifecycle) is overridden to create a new context.</span></span> <span data-ttu-id="dfdd9-221">Nell'app di esempio [`OnInitializedAsync`](xref:blazor/components/lifecycle) carica il contatto nello stesso metodo:</span><span class="sxs-lookup"><span data-stu-id="dfdd9-221">In the sample app, [`OnInitializedAsync`](xref:blazor/components/lifecycle) loads the contact in the same method:</span></span>

[!code-csharp[](~/blazor/common/samples/3.x/BlazorServerEFCoreSample/BlazorServerDbContextExample/Pages/EditContact.razor?name=snippet2)]

<span data-ttu-id="dfdd9-222">Nell'esempio precedente:</span><span class="sxs-lookup"><span data-stu-id="dfdd9-222">In the preceding example:</span></span>

* <span data-ttu-id="dfdd9-223">Quando `Busy` è impostato su `true` , possono iniziare le operazioni asincrone.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-223">When `Busy` is set to `true`, asynchronous operations may begin.</span></span> <span data-ttu-id="dfdd9-224">Quando `Busy` viene impostato di nuovo su `false` , le operazioni asincrone devono essere completate.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-224">When `Busy` is set back to `false`, asynchronous operations should be finished.</span></span>
* <span data-ttu-id="dfdd9-225">Inserire la logica di gestione degli errori aggiuntiva in un `catch` blocco.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-225">Place additional error handling logic in a `catch` block.</span></span>

<h3 id="enable-sensitive-data-logging"><span data-ttu-id="dfdd9-226">Abilitare la registrazione dei dati sensibili</span><span class="sxs-lookup"><span data-stu-id="dfdd9-226">Enable sensitive data logging</span></span></h3>

<span data-ttu-id="dfdd9-227"><xref:Microsoft.EntityFrameworkCore.DbContextOptionsBuilder.EnableSensitiveDataLogging%2A> include i dati dell'applicazione nei messaggi di eccezione e nella registrazione del Framework.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-227"><xref:Microsoft.EntityFrameworkCore.DbContextOptionsBuilder.EnableSensitiveDataLogging%2A> includes application data in exception messages and framework logging.</span></span> <span data-ttu-id="dfdd9-228">I dati registrati possono includere i valori assegnati alle proprietà delle istanze di entità e i valori dei parametri per i comandi inviati al database.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-228">The logged data can include the values assigned to properties of entity instances and parameter values for commands sent to the database.</span></span> <span data-ttu-id="dfdd9-229">La registrazione dei dati con <xref:Microsoft.EntityFrameworkCore.DbContextOptionsBuilder.EnableSensitiveDataLogging%2A> costituisce un rischio per la **sicurezza**, in quanto può esporre password e altre informazioni personali quando registra le istruzioni SQL eseguite sul database.</span><span class="sxs-lookup"><span data-stu-id="dfdd9-229">Logging data with <xref:Microsoft.EntityFrameworkCore.DbContextOptionsBuilder.EnableSensitiveDataLogging%2A> is a **security risk**, as it may expose passwords and other personally identifiable information (PII) when it logs SQL statements executed against the database.</span></span>

<span data-ttu-id="dfdd9-230">È consigliabile abilitare solo <xref:Microsoft.EntityFrameworkCore.DbContextOptionsBuilder.EnableSensitiveDataLogging%2A> per lo sviluppo e il testing:</span><span class="sxs-lookup"><span data-stu-id="dfdd9-230">We recommend only enabling <xref:Microsoft.EntityFrameworkCore.DbContextOptionsBuilder.EnableSensitiveDataLogging%2A> for development and testing:</span></span>

```csharp
#if DEBUG
    services.AddDbContextFactory<ContactContext>(opt =>
        opt.UseSqlite($"Data Source={nameof(ContactContext.ContactsDb)}.db")
        .EnableSensitiveDataLogging());
#else
    services.AddDbContextFactory<ContactContext>(opt =>
        opt.UseSqlite($"Data Source={nameof(ContactContext.ContactsDb)}.db"));
#endif
```

:::moniker-end

## <a name="additional-resources"></a><span data-ttu-id="dfdd9-231">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="dfdd9-231">Additional resources</span></span>

* [<span data-ttu-id="dfdd9-232">Documentazione di EF Core</span><span class="sxs-lookup"><span data-stu-id="dfdd9-232">EF Core documentation</span></span>](/ef/)
