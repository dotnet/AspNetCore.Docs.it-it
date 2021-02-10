---
title: BlazorInserimento di dipendenze ASP.NET Core
author: guardrex
description: Informazioni su come le Blazor app possono inserire i servizi nei componenti.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 12/19/2020
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
uid: blazor/fundamentals/dependency-injection
zone_pivot_groups: blazor-hosting-models
ms.openlocfilehash: 30edffedf1faf96ed54d5380762c8558e478966c
ms.sourcegitcommit: 04ad9cd26fcaa8bd11e261d3661f375f5f343cdc
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/10/2021
ms.locfileid: "100106921"
---
# <a name="aspnet-core-blazor-dependency-injection"></a>BlazorInserimento di dipendenze ASP.NET Core

Di [Rainer Stropek](https://www.timecockpit.com) e [Mike entusiasmanti](https://github.com/mjrousos)

L' [inserimento di dipendenze](xref:fundamentals/dependency-injection) è una tecnica per accedere ai servizi configurati in una posizione centrale:

* I servizi registrati dal Framework possono essere inseriti direttamente in componenti di Blazor app.
* Blazor le app definiscono e registrano servizi personalizzati e le rendono disponibili nell'app tramite DI.

## <a name="default-services"></a>Servizi predefiniti

I servizi illustrati nella tabella seguente sono comunemente usati nelle Blazor app.

| Servizio | Durata | Descrizione |
| ------- | -------- | ----------- |
| <xref:System.Net.Http.HttpClient> | Con ambito | <p>Fornisce metodi per l'invio di richieste HTTP e la ricezione di risposte HTTP da una risorsa identificata da un URI.</p><p>L'istanza di <xref:System.Net.Http.HttpClient> in un' Blazor WebAssembly applicazione utilizza il browser per gestire il traffico HTTP in background.</p><p>Blazor Server per impostazione predefinita, le app non includono un <xref:System.Net.Http.HttpClient> configurato come servizio. Fornire un <xref:System.Net.Http.HttpClient> a un' Blazor Server app.</p><p>Per altre informazioni, vedere <xref:blazor/call-web-api>.</p><p>Un <xref:System.Net.Http.HttpClient> viene registrato come servizio con ambito, non come singleton. Per ulteriori informazioni, vedere la sezione [durata del servizio](#service-lifetime) .</p> |
| <xref:Microsoft.JSInterop.IJSRuntime> | <p>**Blazor WebAssembly**: Singleton</p><p>**Blazor Server**: Con ambito</p> | Rappresenta un'istanza di un runtime JavaScript in cui vengono inviate le chiamate a JavaScript. Per altre informazioni, vedere <xref:blazor/call-javascript-from-dotnet>. |
| <xref:Microsoft.AspNetCore.Components.NavigationManager> | <p>**Blazor WebAssembly**: Singleton</p><p>**Blazor Server**: Con ambito</p> | Contiene gli helper per lavorare con gli URI e lo stato di navigazione. Per ulteriori informazioni, vedere [URI e Helper dello stato di navigazione](xref:blazor/fundamentals/routing#uri-and-navigation-state-helpers). |

Un provider di servizi personalizzato non fornisce automaticamente i servizi predefiniti elencati nella tabella. Se si usa un provider di servizi personalizzato e si richiede uno dei servizi indicati nella tabella, aggiungere i servizi necessari al nuovo provider di servizi.

## <a name="add-services-to-an-app"></a>Aggiungere servizi a un'app

::: zone pivot="webassembly"

Configurare i servizi per la raccolta di servizi dell'app nel `Program.Main` metodo di `Program.cs` . Nell'esempio seguente l' `MyDependency` implementazione è registrata per `IMyDependency` :

[!code-csharp[](dependency-injection/samples_snapshot/Program1.cs?highlight=7)]

Al termine della compilazione dell'host, i servizi sono disponibili nell'ambito radice prima DI eseguire il rendering dei componenti. Questa operazione può essere utile per eseguire la logica di inizializzazione prima del rendering del contenuto:

[!code-csharp[](dependency-injection/samples_snapshot/Program2.cs?highlight=7,12-13)]

L'host fornisce un'istanza di configurazione centrale per l'app. Basandosi sull'esempio precedente, l'URL del servizio meteo viene passato da un'origine di configurazione predefinita (ad esempio, `appsettings.json` ) a `InitializeWeatherAsync` :

[!code-csharp[](dependency-injection/samples_snapshot/Program3.cs?highlight=13-14)]

::: zone-end

::: zone pivot="server"

Dopo aver creato una nuova app, esaminare il `Startup.ConfigureServices` metodo in `Startup.cs` :

```csharp
using Microsoft.Extensions.DependencyInjection;

...

public void ConfigureServices(IServiceCollection services)
{
    ...
}
```

Al <xref:Microsoft.Extensions.Hosting.IHostBuilder.ConfigureServices%2A> metodo viene passato un oggetto <xref:Microsoft.Extensions.DependencyInjection.IServiceCollection> , che è un elenco di oggetti del [descrittore del servizio](xref:Microsoft.Extensions.DependencyInjection.ServiceDescriptor) . I servizi vengono aggiunti nel `ConfigureServices` Metodo fornendo descrittori del servizio alla raccolta di servizi. Nell'esempio seguente viene illustrato il concetto con l' `IDataAccess` interfaccia e la relativa implementazione concreta `DataAccess` :

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddSingleton<IDataAccess, DataAccess>();
}
```

::: zone-end

### <a name="service-lifetime"></a>Durata del servizio

I servizi possono essere configurati con le durate mostrate nella tabella seguente.

| Durata | Descrizione |
| -------- | ----------- |
| <xref:Microsoft.Extensions.DependencyInjection.ServiceDescriptor.Scoped%2A> | <p>Blazor WebAssembly Attualmente le app non hanno un concetto di ambiti di. `Scoped`-i servizi registrati si comportano come `Singleton` servizi.</p><p>Il Blazor Server modello di hosting supporta la `Scoped` durata tra le richieste HTTP, ma non tra SignalR i messaggi di connessione/circuito tra i componenti caricati nel client. Le Razor pagine o la parte MVC dell'app considera i servizi con ambito normalmente e ricrea i servizi in *ogni richiesta http* quando si naviga tra pagine o viste oppure da una pagina o da una vista a un componente. I servizi con ambito non vengono ricostruiti quando ci si sposta tra i componenti nel client, dove la comunicazione con il server avviene tramite la SignalR connessione del circuito dell'utente, non tramite richieste HTTP. Negli scenari dei componenti seguenti nel client, i servizi con ambito vengono ricostruiti perché viene creato un nuovo circuito per l'utente:</p><ul><li>L'utente chiude la finestra del browser. L'utente apre una nuova finestra e torna all'app.</li><li>L'utente chiude l'ultima scheda dell'app in una finestra del browser. L'utente apre una nuova scheda e torna all'app.</li><li>L'utente seleziona il pulsante ricarica/Aggiorna del browser.</li></ul><p>Per ulteriori informazioni sul mantenimento dello stato utente tra i servizi con ambito nelle Blazor Server app, vedere <xref:blazor/hosting-models?pivots=server> .</p> |
| <xref:Microsoft.Extensions.DependencyInjection.ServiceDescriptor.Singleton%2A> | La creazione di una *singola istanza* del servizio. Tutti i componenti che richiedono un `Singleton` servizio ricevono un'istanza dello stesso servizio. |
| <xref:Microsoft.Extensions.DependencyInjection.ServiceDescriptor.Transient%2A> | Ogni volta che un componente ottiene un'istanza di un `Transient` servizio dal contenitore del servizio, riceve una *nuova istanza* del servizio. |

Il sistema DI è basato sul sistema DI ASP.NET Core. Per altre informazioni, vedere <xref:fundamentals/dependency-injection>.

## <a name="request-a-service-in-a-component"></a>Richiedere un servizio in un componente

Una volta aggiunti i servizi alla raccolta di servizi, inserire i servizi nei componenti usando la [`@inject`](xref:mvc/views/razor#inject) Razor direttiva, che ha due parametri:

* Tipo: tipo di servizio da inserire.
* Property: nome della proprietà che riceve il servizio app inserito. La proprietà non richiede la creazione manuale. Il compilatore crea la proprietà.

Per altre informazioni, vedere <xref:mvc/views/dependency-injection>.

Usare più [`@inject`](xref:mvc/views/razor#inject) istruzioni per inserire servizi diversi.

Nell'esempio seguente viene illustrato come utilizzare [`@inject`](xref:mvc/views/razor#inject) . Il servizio che implementa `Services.IDataAccess` viene inserito nella proprietà del componente `DataRepository` . Si noti come il codice usa solo l' `IDataAccess` astrazione:

[!code-razor[](dependency-injection/samples_snapshot/CustomerList.razor?highlight=2-3,20)]

Internamente, la proprietà generata ( `DataRepository` ) utilizza l' [ `[Inject]` attributo](xref:Microsoft.AspNetCore.Components.InjectAttribute). In genere, questo attributo non viene utilizzato direttamente. Se è necessaria una classe base per i componenti e le proprietà inserite sono necessarie anche per la classe base, aggiungere manualmente l' [ `[Inject]` attributo](xref:Microsoft.AspNetCore.Components.InjectAttribute):

```csharp
using Microsoft.AspNetCore.Components;

public class ComponentBase : IComponent
{
    [Inject]
    protected IDataAccess DataRepository { get; set; }

    ...
}
```

Nei componenti derivati dalla classe di base, la [`@inject`](xref:mvc/views/razor#inject) direttiva non è obbligatoria. La <xref:Microsoft.AspNetCore.Components.InjectAttribute> classe della classe base è sufficiente:

```razor
@page "/demo"
@inherits ComponentBase

<h1>Demo Component</h1>
```

## <a name="use-di-in-services"></a>Usare l'inserimento DI dipendenze nei servizi

Servizi complessi potrebbe richiedere servizi aggiuntivi. Nell'esempio seguente `DataAccess` richiede il <xref:System.Net.Http.HttpClient> servizio predefinito. [`@inject`](xref:mvc/views/razor#inject)(o l' [ `[Inject]` attributo](xref:Microsoft.AspNetCore.Components.InjectAttribute)) non è disponibile per l'uso nei servizi. È necessario usare invece l' *inserimento del costruttore* . I servizi necessari vengono aggiunti aggiungendo parametri al costruttore del servizio. Quando si crea il servizio, vengono riconosciuti i servizi richiesti nel costruttore e forniti DI conseguenza. Nell'esempio seguente, il costruttore riceve una <xref:System.Net.Http.HttpClient> via di. <xref:System.Net.Http.HttpClient> è un servizio predefinito.

```csharp
using System.Net.Http;

public class DataAccess : IDataAccess
{
    public DataAccess(HttpClient http)
    {
        ...
    }
}
```

Prerequisiti per l'inserimento del costruttore:

* È necessario che esista un costruttore i cui argomenti possono essere tutti soddisfatti da DI. Sono consentiti parametri aggiuntivi non analizzati da DI se specificano i valori predefiniti.
* Il costruttore applicabile deve essere `public` .
* È necessario che esista un costruttore applicabile. In caso di ambiguità, viene generata un'eccezione.

## <a name="utility-base-component-classes-to-manage-a-di-scope"></a>Classi di componenti di base dell'utilità per gestire un ambito DI

Nelle app ASP.NET Core, i servizi con ambito hanno in genere come ambito la richiesta corrente. Al termine della richiesta, tutti i servizi con ambito o temporaneo vengono eliminati dal sistema DI. Nelle Blazor Server app, l'ambito della richiesta dura per la durata della connessione client, che può comportare la permanenza di servizi temporanei e con ambito più a lungo del previsto. Nelle Blazor WebAssembly app i servizi registrati con una durata con ambito vengono considerati come singleton, quindi vivono più a lungo rispetto ai servizi con ambito nelle app ASP.NET Core tipiche.

> [!NOTE]
> Per rilevare servizi temporanei monouso in un'app, vedere la sezione [rilevare gli eliminabili temporanei](#detect-transient-disposables) .

Un approccio che limita la durata di un servizio nelle Blazor app è l'uso del <xref:Microsoft.AspNetCore.Components.OwningComponentBase> tipo. <xref:Microsoft.AspNetCore.Components.OwningComponentBase> è un tipo astratto derivato da <xref:Microsoft.AspNetCore.Components.ComponentBase> che consente di creare un ambito di che corrisponde alla durata del componente. Con questo ambito, è possibile usare i servizi DI i con una durata con ambito e fare in modo che siano attivi fino a quando il componente. Quando il componente viene eliminato definitivamente, vengono eliminati anche i servizi del provider di servizi con ambito. Questa operazione può essere utile per i servizi che:

* È necessario riutilizzarlo all'interno di un componente, perché la durata temporanea non è appropriata.
* Non devono essere condivise tra i componenti, perché la durata singleton non è appropriata.

Sono disponibili due versioni del <xref:Microsoft.AspNetCore.Components.OwningComponentBase> tipo:

* <xref:Microsoft.AspNetCore.Components.OwningComponentBase> è un elemento figlio astratto e disposable del <xref:Microsoft.AspNetCore.Components.ComponentBase> tipo con una <xref:Microsoft.AspNetCore.Components.OwningComponentBase.ScopedServices> proprietà protetta di tipo <xref:System.IServiceProvider> . Questo provider può essere utilizzato per risolvere i servizi che hanno come ambito la durata del componente.

  I servizi DI inserimento nel componente usando [`@inject`](xref:mvc/views/razor#inject) o l' [ `[Inject]` attributo](xref:Microsoft.AspNetCore.Components.InjectAttribute) non vengono creati nell'ambito del componente. Per utilizzare l'ambito del componente, i servizi devono essere risolti utilizzando <xref:Microsoft.Extensions.DependencyInjection.ServiceProviderServiceExtensions.GetRequiredService%2A> o <xref:System.IServiceProvider.GetService%2A> . Tutti i servizi risolti utilizzando il <xref:Microsoft.AspNetCore.Components.OwningComponentBase.ScopedServices> provider hanno le dipendenze fornite dallo stesso ambito.

  [!code-razor[](dependency-injection/samples_snapshot/Preferences.razor?highlight=3,20-21)]

* <xref:Microsoft.AspNetCore.Components.OwningComponentBase%601> deriva da <xref:Microsoft.AspNetCore.Components.OwningComponentBase> e aggiunge una <xref:Microsoft.AspNetCore.Components.OwningComponentBase%601.Service%2A> proprietà che restituisce un'istanza di `T` dal provider dell'ambito di. Questo tipo è un modo pratico per accedere ai servizi con ambito senza usare un'istanza di <xref:System.IServiceProvider> quando è presente un servizio primario richiesto dall'app dal contenitore di inserimento delle dipendenze usando l'ambito del componente. La <xref:Microsoft.AspNetCore.Components.OwningComponentBase.ScopedServices> proprietà è disponibile, in modo che l'app possa ottenere i servizi di altri tipi, se necessario.

  [!code-razor[](dependency-injection/samples_snapshot/Users.razor?highlight=3,5,8)]

## <a name="use-of-an-entity-framework-core-ef-core-dbcontext-from-di"></a>Uso di un Entity Framework Core (EF Core) DbContext da DI

Per altre informazioni, vedere <xref:blazor/blazor-server-ef-core>.

## <a name="detect-transient-disposables"></a>Rilevare eliminabili temporanei

Gli esempi seguenti illustrano come rilevare servizi temporanei monouso in un'app che deve usare <xref:Microsoft.AspNetCore.Components.OwningComponentBase> . Per ulteriori informazioni, vedere la sezione relativa [alle classi dei componenti di base dell'utilità per gestire una](#utility-base-component-classes-to-manage-a-di-scope) sezione dell'ambito.

::: zone pivot="webassembly"

`DetectIncorrectUsagesOfTransientDisposables.cs`:

[!code-csharp[](dependency-injection/samples_snapshot/3.x/transient-disposables/DetectIncorrectUsagesOfTransientDisposables-wasm.cs)]

Nell' `TransientDisposable` esempio seguente viene rilevato ( `Program.cs` ):

::: moniker range=">= aspnetcore-5.0"

[!code-csharp[](dependency-injection/samples_snapshot/5.x/transient-disposables/DetectIncorrectUsagesOfTransientDisposables-wasm-program.cs?highlight=6,9,17,22-25)]

::: moniker-end 

::: moniker range="< aspnetcore-5.0"

[!code-csharp[](dependency-injection/samples_snapshot/3.x/transient-disposables/DetectIncorrectUsagesOfTransientDisposables-wasm-program.cs?highlight=6,9,17,22-25)]

::: moniker-end

::: zone-end

::: zone pivot="server"

`DetectIncorrectUsagesOfTransientDisposables.cs`:

[!code-csharp[](dependency-injection/samples_snapshot/3.x/transient-disposables/DetectIncorrectUsagesOfTransientDisposables-server.cs)]

Aggiungere lo spazio dei nomi per <xref:Microsoft.Extensions.DependencyInjection?displayProperty=fullName> a `Program.cs` :

```csharp
using Microsoft.Extensions.DependencyInjection;
```

In `Program.CreateHostBuilder` di `Program.cs` :

[!code-csharp[](dependency-injection/samples_snapshot/3.x/transient-disposables/DetectIncorrectUsagesOfTransientDisposables-server-program.cs?highlight=3)]

Nell' `TransientDependency` esempio seguente viene rilevato ( `Startup.cs` ):

[!code-csharp[](dependency-injection/samples_snapshot/3.x/transient-disposables/DetectIncorrectUsagesOfTransientDisposables-server-startup.cs?highlight=6-8,11-32)]

::: zone-end

L'app può registrare gli eliminabili temporanei senza generare un'eccezione. Tuttavia, se si tenta di risolvere i risultati monouso temporanei in un oggetto <xref:System.InvalidOperationException> , come illustrato nell'esempio riportato di seguito.

`Pages/TransientDisposable.razor`:

```razor
@page "/transient-disposable"
@inject TransientDisposable TransientDisposable

<h1>Transient Disposable Detection</h1>
```

Passare al `TransientDisposable` componente in `/transient-disposable` e <xref:System.InvalidOperationException> viene generata un'eccezione quando il Framework tenta di costruire un'istanza di `TransientDisposable` :

> System. InvalidOperationException: si sta tentando di risolvere TransientDisposable di servizio Disposable temporaneo nell'ambito errato. Usare una \<T> classe di base del componente ' OwningComponentBase ' per il servizio ' t'che si sta tentando di risolvere.

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:fundamentals/dependency-injection>
* [`IDisposable` linee guida per le istanze temporanee e condivise](xref:fundamentals/dependency-injection#idisposable-guidance-for-transient-and-shared-instances)
* <xref:mvc/views/dependency-injection>
