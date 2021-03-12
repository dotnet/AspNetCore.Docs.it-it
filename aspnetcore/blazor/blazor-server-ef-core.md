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
ms.openlocfilehash: 53d276db996304852d69566584e43d47aa73f921
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102586553"
---
# <a name="aspnet-core-blazor-server-with-entity-framework-core-efcore"></a>ASP.NET Core Blazor Server con Entity Framework Core (EFCore)

:::moniker range=">= aspnetcore-5.0"

Blazor Server è un Framework di app con stato. L'app mantiene una connessione continuativa al server e lo stato dell'utente viene mantenuto nella memoria del server in un *circuito*. Un esempio di stato utente è costituito dai dati contenuti nelle istanze del servizio [di inserimento delle dipendenze](xref:fundamentals/dependency-injection) che hanno come ambito il circuito. Il modello di applicazione univoco Blazor Server fornito da richiede un approccio speciale per utilizzare Entity Framework Core.

> [!NOTE]
> Questo articolo illustra EF Core nelle Blazor Server app. Blazor WebAssembly le app vengono eseguite in una sandbox di webassembly che impedisce la maggior parte delle connessioni di database dirette. L'esecuzione di EF Core in Blazor WebAssembly esula dall'ambito di questo articolo.

<h2 id="sample-app-5x">App di esempio</h2>

L'app di esempio è stata creata come riferimento per le Blazor Server app che usano EF core. L'app di esempio include una griglia con operazioni di ordinamento e filtro, eliminazione, aggiunta e aggiornamento. Nell'esempio viene illustrato l'utilizzo di EF Core per gestire la concorrenza ottimistica.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/blazor/common/samples/5.x/BlazorServerEFCoreSample) ([procedura per il download](xref:index#how-to-download-a-sample))

Nell'esempio viene utilizzato un database [SQLite](https://www.sqlite.org/index.html) locale in modo che possa essere utilizzato su qualsiasi piattaforma. Nell'esempio viene inoltre configurata la registrazione del database per visualizzare le query SQL generate. Questa configurazione è configurata in `appsettings.Development.json` :

[!code-json[](~/blazor/common/samples/5.x/BlazorServerEFCoreSample/BlazorServerDbContextExample/appsettings.Development.json?highlight=8)]

I componenti Grid, Add e View utilizzano il modello "context-per-Operation", in cui viene creato un contesto per ogni operazione. Il componente Edit usa il modello "context-per-Component", in cui viene creato un contesto per ogni componente.

> [!NOTE]
> Per alcuni esempi di codice in questo argomento sono necessari gli spazi dei nomi e i servizi che non vengono visualizzati. Per esaminare il codice completamente funzionante, incluse le [`@using`](xref:mvc/views/razor#using) [`@inject`](xref:mvc/views/razor#inject) direttive e obbligatorie per gli Razor esempi, vedere l' [app di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/blazor/common/samples/5.x/BlazorServerEFCoreSample).

<h2 id="database-access-5x">Accesso al database</h2>

EF Core si basa su un <xref:Microsoft.EntityFrameworkCore.DbContext> come mezzo per [configurare l'accesso al database](/ef/core/miscellaneous/configuring-dbcontext) e fungere da [*unità di lavoro*](https://martinfowler.com/eaaCatalog/unitOfWork.html). EF Core fornisce l' <xref:Microsoft.Extensions.DependencyInjection.EntityFrameworkServiceCollectionExtensions.AddDbContext%2A> estensione per le app ASP.NET Core che registra il contesto come servizio con *ambito* per impostazione predefinita. Nelle Blazor Server app, le registrazioni del servizio con ambito possono essere problematiche perché l'istanza è condivisa tra i componenti all'interno del circuito dell'utente. <xref:Microsoft.EntityFrameworkCore.DbContext> non è thread-safe e non è progettato per l'uso simultaneo. Le durate esistenti non sono appropriate per i motivi seguenti:

* Il **singleton** condivide lo stato tra tutti gli utenti dell'app e comporta un uso simultaneo non appropriato.
* Con **ambito** (impostazione predefinita) si presenta un problema simile tra i componenti per lo stesso utente.
* I risultati **temporanei** in una nuova istanza per ogni richiesta; Tuttavia, poiché i componenti possono essere di lunga durata, questo risultato è un contesto di durata più lunga di quanto possa essere previsto.

Le indicazioni seguenti sono progettate per offrire un approccio coerente all'uso di EF Core nelle Blazor Server app.

* Per impostazione predefinita, è consigliabile usare un contesto per ogni operazione. Il contesto è progettato per la creazione di un'istanza di overhead veloce e bassa:

  ```csharp
  using var context = new MyContext();

  return await context.MyEntities.ToListAsync();
  ```

* Usare un flag per evitare più operazioni simultanee:

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

  Posizionare le operazioni dopo la `Loading = true;` riga nel `try` blocco.

* Per le operazioni più longeve che sfruttano i vantaggi del [rilevamento delle modifiche](/ef/core/querying/tracking) EF core o del controllo della concorrenza, [Definire](/ef/core/saving/concurrency)l' [ambito del contesto per la durata del componente](#scope-to-the-component-lifetime-5x).

<h3 id="new-dbcontext-instances-5x">Nuove istanze di DbContext</h3>

Il modo più rapido per creare una nuova <xref:Microsoft.EntityFrameworkCore.DbContext> istanza consiste nell'usare `new` per creare una nuova istanza. Esistono tuttavia diversi scenari che possono richiedere la risoluzione di dipendenze aggiuntive. Ad esempio, è possibile usare [`DbContextOptions`](/ef/core/miscellaneous/configuring-dbcontext#configuring-dbcontextoptions) per configurare il contesto.

La soluzione consigliata per creare una nuova <xref:Microsoft.EntityFrameworkCore.DbContext> con dipendenze consiste nell'usare una factory. EF Core 5,0 o versione successiva fornisce una factory predefinita per la creazione di nuovi contesti.

Nell'esempio seguente viene configurato [SQLite](https://www.sqlite.org/index.html) e viene abilitata la registrazione dei dati. Il codice usa un [metodo di estensione ( `AddDbContextFactory` )](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/blazor/common/samples/3.x/BlazorServerEFCoreSample/BlazorServerDbContextExample/Data/FactoryExtensions.cs) per configurare la factory di database per e fornire opzioni predefinite:

[!code-csharp[](~/blazor/common/samples/5.x/BlazorServerEFCoreSample/BlazorServerDbContextExample/Startup.cs?name=snippet1)]

La factory viene inserita in componenti e usata per creare nuove istanze. Ad esempio: in `Pages/Index.razor`:

[!code-csharp[](~/blazor/common/samples/5.x/BlazorServerEFCoreSample/BlazorServerDbContextExample/Pages/Index.razor?name=snippet1)]

> [!NOTE]
> `Wrapper` è un [riferimento](xref:blazor/components/index#capture-references-to-components) al componente `GridWrapper` . Vedere il `Index` componente ( `Pages/Index.razor` ) nell' [app di esempio](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/blazor/common/samples/5.x/BlazorServerEFCoreSample/BlazorServerDbContextExample/Pages/Index.razor).

È <xref:Microsoft.EntityFrameworkCore.DbContext> possibile creare nuove istanze con una factory che consente di configurare la stringa di connessione per `DbContext` , ad esempio quando si usa il [ Identity modello di ASP.NET Core](xref:security/authentication/customize_identity_model):

```csharp
services.AddDbContextFactory<ApplicationDbContext>(options =>
{
    options.UseSqlServer(Configuration.GetConnectionString("DefaultConnection"));
});

services.AddScoped<ApplicationDbContext>(p => 
    p.GetRequiredService<IDbContextFactory<ApplicationDbContext>>()
    .CreateDbContext());
```

<h3 id="scope-to-the-component-lifetime-5x">Ambito per la durata del componente</h3>

È possibile creare un <xref:Microsoft.EntityFrameworkCore.DbContext> esistente per la durata di un componente. In questo modo, è possibile usarlo come [unità di lavoro](https://martinfowler.com/eaaCatalog/unitOfWork.html) e sfruttare le funzionalità predefinite, ad esempio il rilevamento delle modifiche e la risoluzione della concorrenza.
È possibile usare la factory per creare un contesto e tenerne traccia per la durata del componente. Innanzitutto, implementare <xref:System.IDisposable> e inserire la factory come illustrato in `Pages/EditContact.razor` :

```razor
@implements IDisposable
@inject IDbContextFactory<ContactContext> DbFactory
```

L'app di esempio garantisce che il contesto venga eliminato quando il componente viene eliminato:

[!code-csharp[](~/blazor/common/samples/5.x/BlazorServerEFCoreSample/BlazorServerDbContextExample/Pages/EditContact.razor?name=snippet1)]

Infine, [`OnInitializedAsync`](xref:blazor/components/lifecycle) viene eseguito l'override di per creare un nuovo contesto. Nell'app di esempio [`OnInitializedAsync`](xref:blazor/components/lifecycle) carica il contatto nello stesso metodo:

[!code-csharp[](~/blazor/common/samples/5.x/BlazorServerEFCoreSample/BlazorServerDbContextExample/Pages/EditContact.razor?name=snippet2)]

<h3 id="enable-sensitive-data-logging">Abilitare la registrazione dei dati sensibili</h3>

<xref:Microsoft.EntityFrameworkCore.DbContextOptionsBuilder.EnableSensitiveDataLogging%2A> include i dati dell'applicazione nei messaggi di eccezione e nella registrazione del Framework. I dati registrati possono includere i valori assegnati alle proprietà delle istanze di entità e i valori dei parametri per i comandi inviati al database. La registrazione dei dati con <xref:Microsoft.EntityFrameworkCore.DbContextOptionsBuilder.EnableSensitiveDataLogging%2A> costituisce un rischio per la **sicurezza**, in quanto può esporre password e altre informazioni personali quando registra le istruzioni SQL eseguite sul database.

È consigliabile abilitare solo <xref:Microsoft.EntityFrameworkCore.DbContextOptionsBuilder.EnableSensitiveDataLogging%2A> per lo sviluppo e il testing:

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

Blazor Server è un Framework di app con stato. L'app mantiene una connessione continuativa al server e lo stato dell'utente viene mantenuto nella memoria del server in un *circuito*. Un esempio di stato utente è costituito dai dati contenuti nelle istanze del servizio [di inserimento delle dipendenze](xref:fundamentals/dependency-injection) che hanno come ambito il circuito. Il modello di applicazione univoco Blazor Server fornito da richiede un approccio speciale per utilizzare Entity Framework Core.

> [!NOTE]
> Questo articolo illustra EF Core nelle Blazor Server app. Blazor WebAssembly le app vengono eseguite in una sandbox di webassembly che impedisce la maggior parte delle connessioni di database dirette. L'esecuzione di EF Core in Blazor WebAssembly esula dall'ambito di questo articolo.

<h2 id="sample-app-3x">App di esempio</h2>

L'app di esempio è stata creata come riferimento per le Blazor Server app che usano EF core. L'app di esempio include una griglia con operazioni di ordinamento e filtro, eliminazione, aggiunta e aggiornamento. Nell'esempio viene illustrato l'utilizzo di EF Core per gestire la concorrenza ottimistica.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/blazor/common/samples/3.x/BlazorServerEFCoreSample) ([procedura per il download](xref:index#how-to-download-a-sample))

Nell'esempio viene utilizzato un database [SQLite](https://www.sqlite.org/index.html) locale in modo che possa essere utilizzato su qualsiasi piattaforma. Nell'esempio viene inoltre configurata la registrazione del database per visualizzare le query SQL generate. Questa configurazione è configurata in `appsettings.Development.json` :

[!code-json[](~/blazor/common/samples/3.x/BlazorServerEFCoreSample/BlazorServerDbContextExample/appsettings.Development.json?highlight=8)]

I componenti Grid, Add e View utilizzano il modello "context-per-Operation", in cui viene creato un contesto per ogni operazione. Il componente Edit usa il modello "context-per-Component", in cui viene creato un contesto per ogni componente.

> [!NOTE]
> Per alcuni esempi di codice in questo argomento sono necessari gli spazi dei nomi e i servizi che non vengono visualizzati. Per esaminare il codice completamente funzionante, incluse le [`@using`](xref:mvc/views/razor#using) [`@inject`](xref:mvc/views/razor#inject) direttive e obbligatorie per gli Razor esempi, vedere l' [app di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/blazor/common/samples/3.x/BlazorServerEFCoreSample).

<h2 id="database-access-3x">Accesso al database</h2>

EF Core si basa su un <xref:Microsoft.EntityFrameworkCore.DbContext> come mezzo per [configurare l'accesso al database](/ef/core/miscellaneous/configuring-dbcontext) e fungere da [*unità di lavoro*](https://martinfowler.com/eaaCatalog/unitOfWork.html). EF Core fornisce l' <xref:Microsoft.Extensions.DependencyInjection.EntityFrameworkServiceCollectionExtensions.AddDbContext%2A> estensione per le app ASP.NET Core che registra il contesto come servizio con *ambito* per impostazione predefinita. Nelle Blazor Server app questo può essere problematico perché l'istanza è condivisa tra i componenti all'interno del circuito dell'utente. <xref:Microsoft.EntityFrameworkCore.DbContext> non è thread-safe e non è progettato per l'uso simultaneo. Le durate esistenti non sono appropriate per i motivi seguenti:

* Il **singleton** condivide lo stato tra tutti gli utenti dell'app e comporta un uso simultaneo non appropriato.
* Con **ambito** (impostazione predefinita) si presenta un problema simile tra i componenti per lo stesso utente.
* I risultati **temporanei** in una nuova istanza per ogni richiesta; Tuttavia, poiché i componenti possono essere di lunga durata, questo risultato è un contesto di durata più lunga di quanto possa essere previsto.

Le indicazioni seguenti sono progettate per offrire un approccio coerente all'uso di EF Core nelle Blazor Server app.

* Per impostazione predefinita, è consigliabile usare un contesto per ogni operazione. Il contesto è progettato per la creazione di un'istanza di overhead veloce e bassa:

  ```csharp
  using var context = new MyContext();

  return await context.MyEntities.ToListAsync();
  ```

* Usare un flag per evitare più operazioni simultanee:

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

  Posizionare le operazioni dopo la `Loading = true;` riga nel `try` blocco.

* Per le operazioni più longeve che sfruttano i vantaggi del [rilevamento delle modifiche](/ef/core/querying/tracking) EF core o del controllo della concorrenza, [Definire](/ef/core/saving/concurrency)l' [ambito del contesto per la durata del componente](#scope-to-the-component-lifetime-3x).

<h3 id="new-dbcontext-instances-3x">Nuove istanze di DbContext</h3>

Il modo più rapido per creare una nuova <xref:Microsoft.EntityFrameworkCore.DbContext> istanza consiste nell'usare `new` per creare una nuova istanza. Esistono tuttavia diversi scenari che possono richiedere la risoluzione di dipendenze aggiuntive. Ad esempio, è possibile usare [`DbContextOptions`](/ef/core/miscellaneous/configuring-dbcontext#configuring-dbcontextoptions) per configurare il contesto.

La soluzione consigliata per creare una nuova <xref:Microsoft.EntityFrameworkCore.DbContext> con dipendenze consiste nell'usare una factory. L'app di esempio implementa la propria factory in `Data/DbContextFactory.cs` .

[!code-csharp[](~/blazor/common/samples/3.x/BlazorServerEFCoreSample/BlazorServerDbContextExample/Data/DbContextFactory.cs)]

Nella Factory precedente:

* <xref:Microsoft.Extensions.DependencyInjection.ActivatorUtilities.CreateInstance%2A?displayProperty=nameWithType> soddisfa tutte le dipendenze tramite il provider di servizi.
* `IDbContextFactory` è disponibile in EF Core ASP.NET Core 5,0 o versione successiva, quindi l'interfaccia viene [implementata nell'app di esempio per ASP.NET Core 3. x](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/blazor/common/samples/3.x/BlazorServerEFCoreSample/BlazorServerDbContextExample/Data/IDbContextFactory.cs).

Nell'esempio seguente viene configurato [SQLite](https://www.sqlite.org/index.html) e viene abilitata la registrazione dei dati. Il codice usa un metodo di estensione per configurare la factory di database per e fornire opzioni predefinite:

[!code-csharp[](~/blazor/common/samples/3.x/BlazorServerEFCoreSample/BlazorServerDbContextExample/Startup.cs?name=snippet1)]

La factory viene inserita in componenti e usata per creare nuove istanze. Ad esempio: in `Pages/Index.razor`:

[!code-csharp[](~/blazor/common/samples/3.x/BlazorServerEFCoreSample/BlazorServerDbContextExample/Pages/Index.razor?name=snippet1)]

> [!NOTE]
> `Wrapper` è un [riferimento](xref:blazor/components/index#capture-references-to-components) al componente `GridWrapper` . Vedere il `Index` componente ( `Pages/Index.razor` ) nell' [app di esempio](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/blazor/common/samples/3.x/BlazorServerEFCoreSample/BlazorServerDbContextExample/Pages/Index.razor).

È <xref:Microsoft.EntityFrameworkCore.DbContext> possibile creare nuove istanze con una factory che consente di configurare la stringa di connessione per `DbContext` , ad esempio quando si usa [modello di ASP.NET Core Identity ]) (xrif: Security/Authentication/customize_identity_model):

```csharp
services.AddDbContextFactory<ApplicationDbContext>(options =>
{
    options.UseSqlServer(Configuration.GetConnectionString("DefaultConnection"));
});

services.AddScoped<ApplicationDbContext>(p => 
    p.GetRequiredService<IDbContextFactory<ApplicationDbContext>>()
    .CreateDbContext());
```

<h3 id="scope-to-the-component-lifetime-3x">Ambito per la durata del componente</h3>

È possibile creare un <xref:Microsoft.EntityFrameworkCore.DbContext> esistente per la durata di un componente. In questo modo, è possibile usarlo come [unità di lavoro](https://martinfowler.com/eaaCatalog/unitOfWork.html) e sfruttare le funzionalità predefinite, ad esempio il rilevamento delle modifiche e la risoluzione della concorrenza.
È possibile usare la factory per creare un contesto e tenerne traccia per la durata del componente. Innanzitutto, implementare <xref:System.IDisposable> e inserire la factory come illustrato in `Pages/EditContact.razor` :

```razor
@implements IDisposable
@inject IDbContextFactory<ContactContext> DbFactory
```

L'app di esempio garantisce che il contesto venga eliminato quando il componente viene eliminato:

[!code-csharp[](~/blazor/common/samples/3.x/BlazorServerEFCoreSample/BlazorServerDbContextExample/Pages/EditContact.razor?name=snippet1)]

Infine, [`OnInitializedAsync`](xref:blazor/components/lifecycle) viene eseguito l'override di per creare un nuovo contesto. Nell'app di esempio [`OnInitializedAsync`](xref:blazor/components/lifecycle) carica il contatto nello stesso metodo:

[!code-csharp[](~/blazor/common/samples/3.x/BlazorServerEFCoreSample/BlazorServerDbContextExample/Pages/EditContact.razor?name=snippet2)]

Nell'esempio precedente:

* Quando `Busy` è impostato su `true` , possono iniziare le operazioni asincrone. Quando `Busy` viene impostato di nuovo su `false` , le operazioni asincrone devono essere completate.
* Inserire la logica di gestione degli errori aggiuntiva in un `catch` blocco.

<h3 id="enable-sensitive-data-logging">Abilitare la registrazione dei dati sensibili</h3>

<xref:Microsoft.EntityFrameworkCore.DbContextOptionsBuilder.EnableSensitiveDataLogging%2A> include i dati dell'applicazione nei messaggi di eccezione e nella registrazione del Framework. I dati registrati possono includere i valori assegnati alle proprietà delle istanze di entità e i valori dei parametri per i comandi inviati al database. La registrazione dei dati con <xref:Microsoft.EntityFrameworkCore.DbContextOptionsBuilder.EnableSensitiveDataLogging%2A> costituisce un rischio per la **sicurezza**, in quanto può esporre password e altre informazioni personali quando registra le istruzioni SQL eseguite sul database.

È consigliabile abilitare solo <xref:Microsoft.EntityFrameworkCore.DbContextOptionsBuilder.EnableSensitiveDataLogging%2A> per lo sviluppo e il testing:

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

## <a name="additional-resources"></a>Risorse aggiuntive

* [Documentazione di EF Core](/ef/)
