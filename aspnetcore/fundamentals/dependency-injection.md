---
title: Inserimento delle dipendenze in ASP.NET Core
author: rick-anderson
description: Informazioni su come ASP.NET Core implementa l'inserimento delle dipendenze e su come usare questa funzionalità.
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 7/21/2020
no-loc:
- appsettings.json
- ASP.NET Core Identity
- Blazor
- Blazor Server
- Blazor WebAssembly
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: fundamentals/dependency-injection
ms.openlocfilehash: ce6804a58c5b17b57732713acc3aeca15042da0a
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102589712"
---
# <a name="dependency-injection-in-aspnet-core"></a>Inserimento delle dipendenze in ASP.NET Core

::: moniker range=">= aspnetcore-3.0"

Di [Kirk Larkin](https://twitter.com/serpent5), [Steve Smith](https://ardalis.com/), [Scott Addie](https://scottaddie.com)e [Brandon Dahler](https://github.com/brandondahler)

ASP.NET Core supporta lo schema progettuale di software per l'inserimento di dipendenze, una tecnica per ottenere l'[IoC (Inversion of Control)](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#dependency-inversion) tra le classi e le relative dipendenze.

Per altre informazioni specifiche per l'inserimento delle dipendenze all'interno di controller MVC, vedere <xref:mvc/controllers/dependency-injection>.

Per informazioni sull'uso dell'inserimento di dipendenze in applicazioni diverse dalle app Web, vedere [inserimento delle dipendenze in .NET](/dotnet/core/extensions/dependency-injection).

Per ulteriori informazioni sull'inserimento delle dipendenze delle opzioni, vedere <xref:fundamentals/configuration/options> .

In questo argomento vengono fornite informazioni sull'inserimento delle dipendenze in ASP.NET Core. La documentazione principale sull'uso dell'inserimento delle dipendenze è contenuta nell' [inserimento delle dipendenze in .NET](/dotnet/core/extensions/dependency-injection).

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/dependency-injection/samples) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="overview-of-dependency-injection"></a>Panoramica dell'inserimento delle dipendenze

Una *dipendenza* è un oggetto da cui dipende un altro oggetto. Esaminare la `MyDependency` classe seguente con un `WriteMessage` metodo da cui dipendono altre classi:

```csharp
public class MyDependency
{
    public void WriteMessage(string message)
    {
        Console.WriteLine($"MyDependency.WriteMessage called. Message: {message}");
    }
}
```

Una classe può creare un'istanza della `MyDependency` classe per utilizzare il relativo `WriteMessage` metodo. Nell'esempio seguente, la `MyDependency` classe è una dipendenza della `IndexModel` classe:

```csharp
public class IndexModel : PageModel
{
    private readonly MyDependency _dependency = new MyDependency();

    public void OnGet()
    {
        _dependency.WriteMessage("IndexModel.OnGet created this message.");
    }
}
```

La classe crea e dipende direttamente dalla `MyDependency` classe. Le dipendenze del codice, ad esempio nell'esempio precedente, sono problematiche e devono essere evitate per i motivi seguenti:

* Per sostituire `MyDependency` con un'implementazione diversa, `IndexModel` è necessario modificare la classe.
* Se `MyDependency` dispone di dipendenze, devono anche essere configurate dalla `IndexModel` classe. In un progetto di grandi dimensioni con più classi che dipendono da `MyDependency`, il codice di configurazione diventa sparso in tutta l'app.
* È difficile eseguire unit test di questa implementazione. L'app dovrebbe usare una classe `MyDependency` fittizia o stub, ma ciò non è possibile con questo approccio.

L'inserimento delle dipendenze consente di risolvere questi problemi tramite:

* L'uso di un'interfaccia o di una classe di base per astrarre l'implementazione delle dipendenze.
* La registrazione della dipendenza in un contenitore di servizi. ASP.NET Core offre il contenitore di servizi predefinito <xref:System.IServiceProvider>. I servizi vengono in genere registrati nel metodo dell'app `Startup.ConfigureServices` .
* L'*inserimento* del servizio nel costruttore della classe in cui viene usato. Il framework si assume la responsabilità della creazione di un'istanza della dipendenza e della sua eliminazione quando non è più necessaria.

Nell' [app di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/dependency-injection/samples), l' `IMyDependency` interfaccia definisce il `WriteMessage` Metodo:

[!code-csharp[](dependency-injection/samples/3.x/DependencyInjectionSample/Interfaces/IMyDependency.cs?name=snippet1)]

Questa interfaccia viene implementata da un tipo concreto, `MyDependency`:

[!code-csharp[](dependency-injection/samples/3.x/DependencyInjectionSample/Services/MyDependency.cs?name=snippet1)]

L'app di esempio registra il `IMyDependency` servizio con il tipo concreto `MyDependency` . Il <xref:Microsoft.Extensions.DependencyInjection.ServiceCollectionServiceExtensions.AddScoped%2A> metodo registra il servizio con una durata con ambito, la durata di una singola richiesta. Le [durate dei servizi](#service-lifetimes) sono descritte più avanti in questo argomento.

[!code-csharp[](dependency-injection/samples/3.x/DependencyInjectionSample/StartupMyDependency.cs?name=snippet1)]

Nell'app di esempio, il `IMyDependency` servizio viene richiesto e usato per chiamare il `WriteMessage` Metodo:

[!code-csharp[](dependency-injection/samples/3.x/DependencyInjectionSample/Pages/Index2.cshtml.cs?name=snippet1)]

Utilizzando il modello DI, il controller:

* Non usa il tipo concreto `MyDependency` , bensì solo l' `IMyDependency` interfaccia che implementa. Questo consente di modificare facilmente l'implementazione utilizzata dal controller senza modificare il controller.
* Non crea un'istanza di `MyDependency` , viene creata dal contenitore di.

L'implementazione dell' `IMyDependency` interfaccia può essere migliorata usando l'API di registrazione incorporata:

[!code-csharp[](dependency-injection/samples/3.x/DependencyInjectionSample/Services/MyDependency.cs?name=snippet2)]

Il `ConfigureServices` metodo aggiornato registra la nuova `IMyDependency` implementazione:

[!code-csharp[](dependency-injection/samples/3.x/DependencyInjectionSample/StartupMyDependency2.cs?name=snippet1)]

`MyDependency2` dipende da <xref:Microsoft.Extensions.Logging.ILogger%601> , che richiede nel costruttore. `ILogger<TCategoryName>` è un [servizio fornito dal Framework](#framework-provided-services).

Non è insolito usare l'inserimento delle dipendenze in modo concatenato. Ogni dipendenza richiesta richiede a sua volta le proprie dipendenze. Il contenitore risolve le dipendenze nel grafico e restituisce il servizio completamente risolto. Il set di dipendenze che devono essere risolte viene generalmente chiamato *albero delle dipendenze* o *grafico dipendenze* o *grafico degli oggetti*.

Il contenitore si risolve sfruttando `ILogger<TCategoryName>` i vantaggi dei [tipi aperti (generici)](/dotnet/csharp/language-reference/language-specification/types#open-and-closed-types), eliminando la necessità di registrare tutti i tipi [costruiti (generici)](/dotnet/csharp/language-reference/language-specification/types#constructed-types).

Nella terminologia relativa all'inserimento delle dipendenze, un servizio:

* È in genere un oggetto che fornisce un servizio ad altri oggetti, ad esempio il `IMyDependency` servizio.
* Non è correlato a un servizio Web, anche se il servizio può utilizzare un servizio Web.

Il Framework fornisce un sistema di [registrazione](xref:fundamentals/logging/index) affidabile. Le `IMyDependency` implementazioni illustrate negli esempi precedenti sono state scritte per illustrare il di base di, non per implementare la registrazione. La maggior parte delle app non deve scrivere logger. Il codice seguente illustra l'uso della registrazione predefinita, che non richiede la registrazione dei servizi in `ConfigureServices` :

[!code-csharp[](dependency-injection/samples/3.x/DependencyInjectionSample/Pages/About.cshtml.cs?name=snippet)]

Con il codice precedente non è necessario aggiornare `ConfigureServices` , perché la [registrazione](xref:fundamentals/logging/index) viene fornita dal Framework.

## <a name="services-injected-into-startup"></a>Servizi inseriti all'avvio

I servizi possono essere inseriti nel `Startup` costruttore e nel `Startup.Configure` metodo.

`Startup`Quando si utilizza l'host generico () è possibile inserire nel costruttore solo i servizi seguenti <xref:Microsoft.Extensions.Hosting.IHostBuilder> :

* <xref:Microsoft.AspNetCore.Hosting.IWebHostEnvironment>
* <xref:Microsoft.Extensions.Hosting.IHostEnvironment>
* <xref:Microsoft.Extensions.Configuration.IConfiguration>

Qualsiasi servizio registrato con il contenitore DI inserimento delle dipendenze può essere inserito nel `Startup.Configure` Metodo:

```csharp
public void Configure(IApplicationBuilder app, ILogger<Startup> logger)
{
    ...
}
```

Per ulteriori informazioni, vedere <xref:fundamentals/startup> e [accedere alla configurazione all'avvio](xref:fundamentals/configuration/index#access-configuration-in-startup).

## <a name="register-groups-of-services-with-extension-methods"></a>Registrare gruppi di servizi con i metodi di estensione

Il framework ASP.NET Core usa una convenzione per la registrazione di un gruppo di servizi correlati. La convenzione prevede l'uso di un singolo `Add{GROUP_NAME}` metodo di estensione per registrare tutti i servizi richiesti da una funzionalità del Framework. Il metodo di estensione, ad esempio, <xref:Microsoft.Extensions.DependencyInjection.MvcServiceCollectionExtensions.AddControllers%2A> registra i servizi necessari per i controller MVC.

Il codice seguente viene generato dal Razor modello di pagine usando singoli account utente e Mostra come aggiungere altri servizi al contenitore usando i metodi di estensione <xref:Microsoft.Extensions.DependencyInjection.EntityFrameworkServiceCollectionExtensions.AddDbContext%2A> e <xref:Microsoft.Extensions.DependencyInjection.IdentityServiceCollectionUIExtensions.AddDefaultIdentity%2A> :

[!code-csharp[](dependency-injection/samples/3.x/DependencyInjectionSample/StartupEF.cs?name=snippet)]

[!INCLUDE[](~/includes/combine-di.md)]

## <a name="service-lifetimes"></a>Durate del servizio

Vedere la [durata del servizio](/dotnet/core/extensions/dependency-injection#service-lifetimes) nell' [inserimento delle dipendenze in .NET](/dotnet/core/extensions/dependency-injection)

Per usare i servizi con ambito nel middleware, usare uno degli approcci seguenti:

* Inserire il servizio nel middleware o nel `Invoke` `InvokeAsync` metodo. L'uso dell' [inserimento del costruttore](xref:mvc/controllers/dependency-injection#constructor-injection) genera un'eccezione in fase di esecuzione perché impone il comportamento del servizio con ambito come un singleton. Nell'esempio nella sezione relativa alle [Opzioni di durata e registrazione](#lifetime-and-registration-options) viene illustrato l' `InvokeAsync` approccio.
* Usare il [middleware basato su Factory](xref:fundamentals/middleware/extensibility). Il middleware registrato con questo approccio viene attivato per ogni richiesta client (connessione), che consente di inserire i servizi con ambito nel metodo del middleware `InvokeAsync` .

Per altre informazioni, vedere <xref:fundamentals/middleware/write#per-request-middleware-dependencies>.

## <a name="service-registration-methods"></a>Metodi di registrazione del servizio

Vedere i [metodi di registrazione del servizio](/dotnet/core/extensions/dependency-injection#service-registration-methods) in [inserimento delle dipendenze in .NET](/dotnet/core/extensions/dependency-injection)

 Quando si [simulano i tipi per il test](xref:test/integration-tests#inject-mock-services), è comune usare più implementazioni.

La registrazione di un servizio con un solo tipo di implementazione equivale alla registrazione del servizio con lo stesso tipo di implementazione e di servizio. Questo è il motivo per cui non è possibile registrare più implementazioni di un servizio usando i metodi che non accettano un tipo di servizio esplicito. Questi metodi possono registrare più *istanze* di un servizio, ma avranno tutti lo stesso tipo di *implementazione* .

Uno qualsiasi dei metodi di registrazione del servizio sopra indicati può essere utilizzato per registrare più istanze del servizio dello stesso tipo di servizio. Nell'esempio seguente `AddSingleton` viene chiamato due volte con `IMyDependency` come tipo di servizio. La seconda chiamata a `AddSingleton` sostituisce quella precedente quando viene risolta come `IMyDependency` e aggiunge a quella precedente quando più servizi vengono risolti tramite `IEnumerable<IMyDependency>` . I servizi vengono visualizzati nell'ordine in cui sono stati registrati durante la risoluzione tramite `IEnumerable<{SERVICE}>` .

```csharp
services.AddSingleton<IMyDependency, MyDependency>();
services.AddSingleton<IMyDependency, DifferentDependency>();

public class MyService
{
    public MyService(IMyDependency myDependency, 
       IEnumerable<IMyDependency> myDependencies)
    {
        Trace.Assert(myDependency is DifferentDependency);

        var dependencyArray = myDependencies.ToArray();
        Trace.Assert(dependencyArray[0] is MyDependency);
        Trace.Assert(dependencyArray[1] is DifferentDependency);
    }
}
```

## <a name="constructor-injection-behavior"></a>Comportamento dell'inserimento del costruttore

Vedere [comportamento di inserimento del costruttore](/dotnet/core/extensions/dependency-injection#constructor-injection-behavior) nell' [inserimento delle dipendenze in .NET](/dotnet/core/extensions/dependency-injection)

## <a name="entity-framework-contexts"></a>Contesti di Entity Framework

Per impostazione predefinita, i contesti di Entity Framework vengono aggiunti al contenitore del servizio usando la [durata con ambito](#service-lifetimes) poiché le operazioni del database dell'app Web sono in genere limitate all'ambito della richiesta client. Per usare una durata diversa, specificare la durata usando un <xref:Microsoft.Extensions.DependencyInjection.EntityFrameworkServiceCollectionExtensions.AddDbContext%2A> Overload. I servizi di una determinata durata non devono utilizzare un contesto di database con una durata minore della durata del servizio.

## <a name="lifetime-and-registration-options"></a>Opzioni di durata e di registrazione

Per dimostrare la differenza tra le durate dei servizi e le relative opzioni di registrazione, prendere in considerazione le interfacce seguenti che rappresentano un'attività come operazione con un identificatore `OperationId` . A seconda del modo in cui viene configurata la durata del servizio di un'operazione per le interfacce seguenti, il contenitore fornisce le stesse istanze o diverse del servizio quando richiesto da una classe:

[!code-csharp[](dependency-injection/samples/3.x/DependencyInjectionSample/Interfaces/IOperation.cs?name=snippet1)]

La `Operation` classe seguente implementa tutte le interfacce precedenti. Il `Operation` costruttore genera un GUID e archivia gli ultimi 4 caratteri nella `OperationId` proprietà:

[!code-csharp[](dependency-injection/samples/3.x/DependencyInjectionSample/Models/Operation.cs?name=snippet1)]

Il `Startup.ConfigureServices` metodo crea più registrazioni della `Operation` classe in base alle durate denominate:

[!code-csharp[](dependency-injection/samples/3.x/DependencyInjectionSample/Startup2.cs?name=snippet1)]

L'app di esempio illustra le durate degli oggetti all'interno e tra le richieste. Il `IndexModel` e il middleware richiedono ogni tipo di `IOperation` tipo e registrano `OperationId` per ogni tipo:

[!code-csharp[](dependency-injection/samples/3.x/DependencyInjectionSample/Pages/Index.cshtml.cs?name=snippet1)]

Analogamente a `IndexModel` , il middleware risolve gli stessi servizi:

[!code-csharp[](dependency-injection/samples/3.x/DependencyInjectionSample/Middleware/MyMiddleware.cs?name=snippet)]

I servizi con ambito devono essere risolti nel `InvokeAsync` Metodo:

[!code-csharp[](dependency-injection/samples/3.x/DependencyInjectionSample/Middleware/MyMiddleware.cs?name=snippet2&highlight=2)]

L'output del logger Mostra:

* Gli oggetti *temporanei* sono sempre diversi. Il `OperationId` valore temporaneo è diverso in `IndexModel` e nel middleware.
* Gli oggetti con *ambito* sono gli stessi per ogni richiesta, ma sono diversi in ogni richiesta.
* Gli oggetti *singleton* sono gli stessi per ogni richiesta.

Per ridurre l'output di registrazione, impostare "Logging: LogLevel: Microsoft: Error" nella *appsettings.Development.jssu* file:

[!code-json[](dependency-injection/samples/3.x/DependencyInjectionSample/appsettings.Development.json?highlight=7)]

## <a name="call-services-from-main"></a>Chiamare i servizi da main

Creare un <xref:Microsoft.Extensions.DependencyInjection.IServiceScope>IServiceScope con [IServiceScopeFactory.CreateScope](xref:Microsoft.Extensions.DependencyInjection.IServiceScopeFactory.CreateScope%2A) per risolvere un servizio con ambito nell'ambito di applicazione dell'app. Questo approccio è utile per l'accesso a un servizio con ambito all'avvio e per l'esecuzione di attività di inizializzazione.

Nell'esempio seguente viene illustrato come accedere al servizio con ambito `IMyDependency` e come chiamare il relativo `WriteMessage` metodo in `Program.Main` :

[!code-csharp[](dependency-injection/samples/3.x/DependencyInjectionSample/Program.cs?name=snippet)]

<a name="sv"></a>

## <a name="scope-validation"></a>Convalida dell'ambito

Vedere [comportamento di inserimento del costruttore](/dotnet/core/extensions/dependency-injection#constructor-injection-behavior) nell' [inserimento delle dipendenze in .NET](/dotnet/core/extensions/dependency-injection)

Per ulteriori informazioni, vedere [Convalida dell'ambito](xref:fundamentals/host/web-host#scope-validation).

## <a name="request-services"></a>Servizi di richiesta

I servizi disponibili all'interno di una richiesta di ASP.NET Core vengono esposti tramite la raccolta [HttpContext. RequestServices](xref:Microsoft.AspNetCore.Http.HttpContext.RequestServices) . Quando i servizi vengono richiesti dall'interno di una richiesta, i servizi e le relative dipendenze vengono risolti dalla `RequestServices` raccolta.

Il Framework crea un ambito per ogni richiesta ed `RequestServices` espone il provider di servizi con ambito. Tutti i servizi con ambito sono validi fino a quando la richiesta è attiva.

> [!NOTE]
> Preferisce richiedere le dipendenze come parametri del costruttore per la risoluzione dei servizi dalla `RequestServices` raccolta. In questo modo, le classi risultano più facili da testare.

## <a name="design-services-for-dependency-injection"></a>Progettare i servizi per l'inserimento di dipendenze

Durante la progettazione di servizi per l'inserimento di dipendenze:

* Evitare membri e classi statiche con stato. Evitare di creare lo stato globale progettando le app in modo da usare i servizi singleton.
* Evitare la creazione diretta di istanze delle classi dipendenti all'interno di servizi. La creazione diretta di istanze associa il codice a una determinata implementazione.
* Semplifica la testing dei servizi.

Se una classe dispone di molte dipendenze inserite, potrebbe essere un segno che la classe ha troppe responsabilità e viola il [principio di singola responsabilità (SRP)](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#single-responsibility). Tentare di effettuare il refactoring della classe spostando alcune delle responsabilità in nuove classi. Tenere presente che Razor le classi del modello della pagina delle pagine e le classi controller MVC devono concentrarsi sulle problematiche dell'interfaccia utente.

### <a name="disposal-of-services"></a>Eliminazione dei servizi

Il contenitore chiama <xref:System.IDisposable.Dispose%2A> per i tipi <xref:System.IDisposable> creati. I servizi risolti dal contenitore non devono mai essere eliminati dallo sviluppatore. Se un tipo o una factory viene registrata come singleton, il contenitore elimina il singleton automaticamente.

Nell'esempio seguente i servizi vengono creati dal contenitore del servizio ed eliminati automaticamente:

[!code-csharp[](dependency-injection/samples/3.x/DIsample2/DIsample2/Services/Service1.cs?name=snippet)]

[!code-csharp[](dependency-injection/samples/3.x/DIsample2/DIsample2/Startup.cs?name=snippet)]

[!code-csharp[](dependency-injection/samples/3.x/DIsample2/DIsample2/Pages/Index.cshtml.cs?name=snippet)]

La console di debug Visualizza il seguente output dopo ogni aggiornamento della pagina di indice:

```console
Service1: IndexModel.OnGet
Service2: IndexModel.OnGet
Service3: IndexModel.OnGet
Service1.Dispose
```

### <a name="services-not-created-by-the-service-container"></a>Servizi non creati dal contenitore del servizio

Osservare il codice seguente:

[!code-csharp[](dependency-injection/samples/3.x/DIsample2/DIsample2/Startup2.cs?name=snippet)]

Nel codice precedente:

* Le istanze del servizio non vengono create dal contenitore dei servizi.
* Il Framework non elimina automaticamente i servizi.
* Lo sviluppatore è responsabile dell'eliminazione dei servizi.

### <a name="idisposable-guidance-for-transient-and-shared-instances"></a>Linee guida IDisposable per istanze temporanee e condivise

Vedere le [linee guida di IDisposable per istanze temporanee e condivise](/dotnet/core/extensions/dependency-injection-guidelines#idisposable-guidance-for-transient-and-shared-instances) nell' [inserimento delle dipendenze in .NET](/dotnet/core/extensions/dependency-injection)

## <a name="default-service-container-replacement"></a>Sostituzione del contenitore di servizi predefinito

Vedere [sostituzione predefinita del contenitore di servizi](/dotnet/core/extensions/dependency-injection-guidelines#default-service-container-replacement) nell' [inserimento delle dipendenze in .NET](/dotnet/core/extensions/dependency-injection)

## <a name="recommendations"></a>Consigli

Vedere le [raccomandazioni](/dotnet/core/extensions/dependency-injection-guidelines#recommendations) nell' [inserimento delle dipendenze in .NET](/dotnet/core/extensions/dependency-injection)

* Evitare di usare il *modello di localizzatore del servizio*. Ad esempio, non richiamare <xref:System.IServiceProvider.GetService%2A> per ottenere un'istanza del servizio quando è invece possibile usare l'inserimento delle dipendenze:

  **Non corretto:**

    ![Codice errato](dependency-injection/_static/bad.png)

  **Corretto**:

  ```csharp
  public class MyClass
  {
      private readonly IOptionsMonitor<MyOptions> _optionsMonitor;

      public MyClass(IOptionsMonitor<MyOptions> optionsMonitor)
      {
          _optionsMonitor = optionsMonitor;
      }

      public void MyMethod()
      {
          var option = _optionsMonitor.CurrentValue.Option;

          ...
      }
  }
  ```
* Un'altra variazione del localizzatore del servizio da evitare è inserire una factory che risolve le dipendenze in fase di esecuzione. Queste procedure combinano le strategie di [inversione del controllo](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#dependency-inversion).
* Evitare l'accesso statico a `HttpContext` (ad esempio [IHttpContextAccessor.HttpContext](xref:Microsoft.AspNetCore.Http.IHttpContextAccessor.HttpContext)).

<a name="ASP0000"></a>
* Evitare chiamate a <xref:Microsoft.Extensions.DependencyInjection.ServiceCollectionContainerBuilderExtensions.BuildServiceProvider%2A> in `ConfigureServices` . `BuildServiceProvider`La chiamata in genere si verifica quando lo sviluppatore desidera risolvere un servizio in `ConfigureServices` . Si consideri, ad esempio, il caso in cui l'oggetto `LoginPath` viene caricato dalla configurazione. Evitare l'approccio seguente:

  ![codice errato durante la chiamata di BuildServiceProvider](~/fundamentals/dependency-injection/_static/badcodeX.png)

  Nell'immagine precedente, selezionando la linea ondulata verde in `services.BuildServiceProvider` viene visualizzato l'avviso ASP0000 seguente:

  > ASP0000 la chiamata a' BuildServiceProvider ' dal codice dell'applicazione comporta una copia aggiuntiva dei servizi singleton creati. Prendere in considerazione alternative come la dipendenza che inserisce i servizi come parametri per "Configure".

  `BuildServiceProvider`La chiamata di crea un secondo contenitore, che può creare singleton incompleti e causare riferimenti a oggetti grafici tra più contenitori.

  Un modo corretto per ottenere `LoginPath` consiste nell'usare il supporto incorporato del modello di opzioni per l'inserimento delle dipendenze:

  [!code-csharp[](dependency-injection/samples/3.x/AntiPattern3/Startup.cs?name=snippet)]

* I servizi temporanei Disposable vengono acquisiti dal contenitore per l'eliminazione. Questa operazione può comportare una perdita di memoria se risolta dal contenitore di primo livello.
* Abilitare la convalida dell'ambito per assicurarsi che l'app non disponga di singleton che acquisiscono i servizi con ambito. Per ulteriori informazioni, vedere [Convalida dell'ambito](#scope-validation).

È tuttavia possibile che in alcuni casi queste raccomandazioni debbano essere ignorate. Le eccezioni sono rari, soprattutto casi speciali all'interno del Framework stesso.

L'inserimento di dipendenze è un'*alternativa* ai modelli di accesso agli oggetti statici/globali. Se l'inserimento di dipendenze viene usato con l'accesso agli oggetti statico i vantaggi non saranno evidenti.

## <a name="recommended-patterns-for-multi-tenancy-in-di"></a>Modelli consigliati per il multi-tenant in DI

[Orchard Core](https://github.com/OrchardCMS/OrchardCore) è un Framework di applicazione per la creazione di applicazioni modulari multi-tenant in ASP.NET Core. Per ulteriori informazioni, vedere la [documentazione di Orchard Core](https://docs.orchardcore.net/en/dev/).

Vedere gli [esempi di Orchard Core](https://github.com/OrchardCMS/OrchardCore.Samples) per esempi di come creare app modulari e multi-tenant usando solo il Framework Orchard core senza le funzionalità specifiche di CMS.

## <a name="framework-provided-services"></a>Servizi forniti dal framework

Il `Startup.ConfigureServices` metodo registra i servizi usati dall'app, incluse le funzionalità della piattaforma, ad esempio Entity Framework Core e ASP.NET Core MVC. Inizialmente, l'oggetto `IServiceCollection` fornito a `ConfigureServices` dispone di servizi definiti dal Framework a seconda della [configurazione dell'host](xref:fundamentals/index#host). Per le app basate sui modelli di ASP.NET Core, il Framework registra più di 250 servizi. 

La tabella seguente elenca un piccolo esempio di questi servizi registrati dal Framework:

| Tipo di servizio                                                                                    | Durata  |
|-------------------------------------------------------------------------------------------------|-----------|
| <xref:Microsoft.AspNetCore.Hosting.Builder.IApplicationBuilderFactory?displayProperty=fullName> | Temporaneo |
| <xref:Microsoft.Extensions.Hosting.IHostApplicationLifetime>                                    | Singleton |
| <xref:Microsoft.AspNetCore.Hosting.IWebHostEnvironment>                                         | Singleton |
| <xref:Microsoft.AspNetCore.Hosting.IStartup?displayProperty=fullName>                           | Singleton |
| <xref:Microsoft.AspNetCore.Hosting.IStartupFilter?displayProperty=fullName>                     | Temporaneo |
| <xref:Microsoft.AspNetCore.Hosting.Server.IServer?displayProperty=fullName>                     | Singleton |
| <xref:Microsoft.AspNetCore.Http.IHttpContextFactory?displayProperty=fullName>                   | Temporaneo |
| <xref:Microsoft.Extensions.Logging.ILogger%601?displayProperty=fullName>                        | Singleton |
| <xref:Microsoft.Extensions.Logging.ILoggerFactory?displayProperty=fullName>                     | Singleton |
| <xref:Microsoft.Extensions.ObjectPool.ObjectPoolProvider?displayProperty=fullName>              | Singleton |
| <xref:Microsoft.Extensions.Options.IConfigureOptions%601?displayProperty=fullName>              | Temporaneo |
| <xref:Microsoft.Extensions.Options.IOptions%601?displayProperty=fullName>                       | Singleton |
| <xref:System.Diagnostics.DiagnosticSource?displayProperty=fullName>                             | Singleton |
| <xref:System.Diagnostics.DiagnosticListener?displayProperty=fullName>                           | Singleton |

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:mvc/views/dependency-injection>
* <xref:mvc/controllers/dependency-injection>
* <xref:security/authorization/dependencyinjection>
* <xref:blazor/fundamentals/dependency-injection>
* [Modelli DI conferenza NDC per lo sviluppo DI app DI](https://www.youtube.com/watch?v=x-C-CNBVTaY)
* <xref:fundamentals/startup>
* <xref:fundamentals/middleware/extensibility>
* [Quattro modi per eliminare IDisposable in ASP.NET Core](https://andrewlock.net/four-ways-to-dispose-idisposables-in-asp-net-core/)
* [Scrittura di codice pulito in ASP.NET Core con inserimento delle dipendenze (MSDN)](/archive/msdn-magazine/2016/may/asp-net-writing-clean-code-in-asp-net-core-with-dependency-injection)
* [Explicit Dependencies Principle (Principio delle dipendenze esplicite)](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#explicit-dependencies)
* [Inversion of Control Containers and the Dependency Injection Pattern (Martin Fowler) (Contenitori di inversione del controllo e modello di inserimento delle dipendenze)](https://www.martinfowler.com/articles/injection.html)
* [How to register a service with multiple interfaces in ASP.NET Core DI](https://andrewlock.net/how-to-register-a-service-with-multiple-interfaces-for-in-asp-net-core-di/) (Come registrare un servizio con più interfacce in ASP.NET Core DI)

::: moniker-end

::: moniker range="< aspnetcore-3.0"

Di [Steve Smith](https://ardalis.com/), [Scott Addie](https://scottaddie.com)e [Brandon Dahler](https://github.com/brandondahler)

ASP.NET Core supporta lo schema progettuale di software per l'inserimento di dipendenze, una tecnica per ottenere l'[IoC (Inversion of Control)](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#dependency-inversion) tra le classi e le relative dipendenze.

Per altre informazioni specifiche per l'inserimento delle dipendenze all'interno di controller MVC, vedere <xref:mvc/controllers/dependency-injection>.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/dependency-injection/samples) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="overview-of-dependency-injection"></a>Panoramica dell'inserimento delle dipendenze

Una *dipendenza* è qualsiasi oggetto richiesto da un altro oggetto. Esaminare la classe `MyDependency` seguente con un metodo `WriteMessage` da cui dipendono altre classi in un'app:

```csharp
public class MyDependency
{
    public MyDependency()
    {
    }

    public Task WriteMessage(string message)
    {
        Console.WriteLine(
            $"MyDependency.WriteMessage called. Message: {message}");

        return Task.FromResult(0);
    }
}
```

È possibile creare un'istanza della classe `MyDependency` per rendere il metodo `WriteMessage`disponibile per una classe. La classe `MyDependency` è una dipendenza della classe `IndexModel`:

```csharp
public class IndexModel : PageModel
{
    MyDependency _dependency = new MyDependency();

    public async Task OnGetAsync()
    {
        await _dependency.WriteMessage(
            "IndexModel.OnGetAsync created this message.");
    }
}
```

La classe crea e dipende direttamente dall'istanza `MyDependency`. Le dipendenze del codice (come nell'esempio precedente) sono problematiche e devono essere evitate per i motivi seguenti:

* Per sostituire `MyDependency` con un'implementazione diversa, la classe deve essere modificata.
* Se `MyDependency` ha dipendenze, devono essere configurate dalla classe. In un progetto di grandi dimensioni con più classi che dipendono da `MyDependency`, il codice di configurazione diventa sparso in tutta l'app.
* È difficile eseguire unit test di questa implementazione. L'app dovrebbe usare una classe `MyDependency` fittizia o stub, ma ciò non è possibile con questo approccio.

L'inserimento delle dipendenze consente di risolvere questi problemi tramite:

* L'uso di un'interfaccia o di una classe di base per astrarre l'implementazione delle dipendenze.
* La registrazione della dipendenza in un contenitore di servizi. ASP.NET Core offre il contenitore di servizi predefinito <xref:System.IServiceProvider>. I servizi vengono registrati nel metodo `Startup.ConfigureServices` dell'app.
* L'*inserimento* del servizio nel costruttore della classe in cui viene usato. Il framework si assume la responsabilità della creazione di un'istanza della dipendenza e della sua eliminazione quando non è più necessaria.

Nell'[app di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/dependency-injection/samples), l'interfaccia `IMyDependency` definisce un metodo che il servizio fornisce all'app:

[!code-csharp[](dependency-injection/samples/2.x/DependencyInjectionSample/Interfaces/IMyDependency.cs?name=snippet1)]

Questa interfaccia viene implementata da un tipo concreto, `MyDependency`:

[!code-csharp[](dependency-injection/samples/2.x/DependencyInjectionSample/Services/MyDependency.cs?name=snippet1)]

`MyDependency` richiede <xref:Microsoft.Extensions.Logging.ILogger`1> nel costruttore. Non è insolito usare l'inserimento delle dipendenze in modo concatenato. Ogni dipendenza richiesta richiede a sua volta le proprie dipendenze. Il contenitore risolve le dipendenze nel grafico e restituisce il servizio completamente risolto. Il set di dipendenze che devono essere risolte viene generalmente chiamato *albero delle dipendenze* o *grafico dipendenze* o *grafico degli oggetti*.

`IMyDependency` e `ILogger<TCategoryName>` devono essere registrati nel contenitore di servizi. `IMyDependency` viene registrato in `Startup.ConfigureServices`. `ILogger<TCategoryName>` viene registrato dall'infrastruttura di registrazione delle astrazioni, pertanto si tratta di un [servizio fornito dal Framework](#framework-provided-services) registrato per impostazione predefinita dal Framework.

Il contenitore risolve `ILogger<TCategoryName>` avvalendosi di [tipi aperti (generici)](/dotnet/csharp/language-reference/language-specification/types#open-and-closed-types), eliminando l'esigenza di registrare ogni singolo [tipo costruito (generico)](/dotnet/csharp/language-reference/language-specification/types#constructed-types):

```csharp
services.AddSingleton(typeof(ILogger<>), typeof(Logger<>));
```

Nell'app di esempio, il servizio `IMyDependency` viene registrato con il tipo concreto `MyDependency`. La registrazione definisce come ambito della durata di servizio la durata di una singola richiesta. Le [durate dei servizi](#service-lifetimes) sono descritte più avanti in questo argomento.

[!code-csharp[](dependency-injection/samples/2.x/DependencyInjectionSample/Startup.cs?name=snippet1&highlight=5)]

> [!NOTE]
> Ogni `services.Add{SERVICE_NAME}` metodo di estensione aggiunge e potenzialmente configura i servizi. Ad esempio, `services.AddControllersWithViews` , `services.AddRazorPages` e `services.AddControllers` aggiunge i servizi ASP.NET Core le app richiedono. È consigliabile che le app seguano questa convenzione. Inserire i metodi di estensione nello spazio dei nomi <xref:Microsoft.Extensions.DependencyInjection?displayProperty=fullName> per incapsulare i gruppi di registrazioni di servizio. Inclusa anche la parte relativa allo spazio dei nomi `Microsoft.Extensions.DependencyInjection` per i metodi di estensione di.
>
> * Consente di visualizzarli in [IntelliSense](/visualstudio/ide/using-intellisense) senza aggiungere ulteriori `using` blocchi.
> * Impedisce `using` l'utilizzo di istruzioni eccessive nella `Startup` classe in cui i metodi di estensione vengono in genere chiamati da.

Se il costruttore del servizio richiede un [tipo incorporato](/dotnet/csharp/language-reference/keywords/built-in-types-table), ad esempio `string`, è possibile inserirlo usando la [configurazione](xref:fundamentals/configuration/index) o il [modello di opzioni](xref:fundamentals/configuration/options):

```csharp
public class MyDependency : IMyDependency
{
    public MyDependency(IConfiguration config)
    {
        var myStringValue = config["MyStringKey"];

        // Use myStringValue
    }

    ...
}
```

È richiesta un'istanza del servizio tramite il costruttore di una classe in cui il servizio viene usato e assegnato a un campo privato. Il campo viene usato per accedere al servizio in base alle esigenze per l'intera classe.

Nell'app di esempio, viene richiesta l'istanza `IMyDependency` usata per chiamare il metodo `WriteMessage` del servizio:

[!code-csharp[](dependency-injection/samples/2.x/DependencyInjectionSample/Pages/Index.cshtml.cs?name=snippet1&highlight=3,6,13,29-30)]

## <a name="services-injected-into-startup"></a>Servizi inseriti all'avvio

`Startup`Quando si utilizza l'host generico () è possibile inserire nel costruttore solo i tipi di servizio seguenti <xref:Microsoft.Extensions.Hosting.IHostBuilder> :

* `IWebHostEnvironment`
* <xref:Microsoft.Extensions.Hosting.IHostEnvironment>
* <xref:Microsoft.Extensions.Configuration.IConfiguration>

I servizi possono essere inseriti in `Startup.Configure` :

```csharp
public void Configure(IApplicationBuilder app, IOptions<MyOptions> options)
{
    ...
}
```

Per altre informazioni, vedere <xref:fundamentals/startup>.

## <a name="framework-provided-services"></a>Servizi forniti dal framework

Il `Startup.ConfigureServices` metodo è responsabile della definizione dei servizi utilizzati dall'app, incluse le funzionalità della piattaforma, ad esempio Entity Framework Core e ASP.NET Core MVC. Inizialmente, l'oggetto `IServiceCollection` fornito a `ConfigureServices` dispone di servizi definiti dal Framework a seconda della [configurazione dell'host](xref:fundamentals/index#host). Non è insolito che un'app basata su un modello di ASP.NET Core disponga di centinaia di servizi registrati dal Framework. Nella tabella seguente è elencato un piccolo esempio di servizi registrati dal Framework.

| Tipo di servizio | Durata |
| ------------ | -------- |
| <xref:Microsoft.AspNetCore.Hosting.Builder.IApplicationBuilderFactory?displayProperty=fullName> | Temporaneo |
| <xref:Microsoft.AspNetCore.Hosting.IApplicationLifetime?displayProperty=fullName> | Singleton |
| <xref:Microsoft.AspNetCore.Hosting.IHostingEnvironment?displayProperty=fullName> | Singleton |
| <xref:Microsoft.AspNetCore.Hosting.IStartup?displayProperty=fullName> | Singleton |
| <xref:Microsoft.AspNetCore.Hosting.IStartupFilter?displayProperty=fullName> | Temporaneo |
| <xref:Microsoft.AspNetCore.Hosting.Server.IServer?displayProperty=fullName> | Singleton |
| <xref:Microsoft.AspNetCore.Http.IHttpContextFactory?displayProperty=fullName> | Temporaneo |
| <xref:Microsoft.Extensions.Logging.ILogger`1?displayProperty=fullName> | Singleton |
| <xref:Microsoft.Extensions.Logging.ILoggerFactory?displayProperty=fullName> | Singleton |
| <xref:Microsoft.Extensions.ObjectPool.ObjectPoolProvider?displayProperty=fullName> | Singleton |
| <xref:Microsoft.Extensions.Options.IConfigureOptions`1?displayProperty=fullName> | Temporaneo |
| <xref:Microsoft.Extensions.Options.IOptions`1?displayProperty=fullName> | Singleton |
| <xref:System.Diagnostics.DiagnosticSource?displayProperty=fullName> | Singleton |
| <xref:System.Diagnostics.DiagnosticListener?displayProperty=fullName> | Singleton |

## <a name="register-additional-services-with-extension-methods"></a>Registrare servizi aggiuntivi con i metodi di estensione

Quando è disponibile un metodo di estensione della raccolta di servizi per la registrazione di un servizio (e dei servizi dipendenti, se necessario), la convenzione consiste nell'usare un singolo metodo di estensione `Add{SERVICE_NAME}` per registrare tutti i servizi richiesti da tale servizio. Il codice seguente è un esempio di come aggiungere servizi aggiuntivi al contenitore usando i metodi di estensione [AddDbContext \<TContext> ](/dotnet/api/microsoft.extensions.dependencyinjection.entityframeworkservicecollectionextensions.adddbcontext) e <xref:Microsoft.Extensions.DependencyInjection.IdentityServiceCollectionExtensions.AddIdentityCore*> :

```csharp
public void ConfigureServices(IServiceCollection services)
{
    ...

    services.AddDbContext<ApplicationDbContext>(options =>
        options.UseSqlServer(Configuration.GetConnectionString("DefaultConnection")));

    services.AddIdentity<ApplicationUser, IdentityRole>()
        .AddEntityFrameworkStores<ApplicationDbContext>()
        .AddDefaultTokenProviders();

    ...
}
```

Per altre informazioni, vedere la classe <xref:Microsoft.Extensions.DependencyInjection.ServiceCollection> nella documentazione delle API.

## <a name="service-lifetimes"></a>Durate del servizio

Scegliere una durata appropriata per ogni servizio registrato. I servizi ASP.NET Core possono essere configurati con le impostazioni di durata seguenti:

### <a name="transient"></a>Temporaneo

I servizi con durata temporanea (<xref:Microsoft.Extensions.DependencyInjection.ServiceCollectionServiceExtensions.AddTransient*>) vengono creati ogni volta che vengono richiesti dal contenitore dei servizi. Questa impostazione di durata è consigliata per i servizi semplici senza stati.

Nelle app che elaborano le richieste, i servizi temporanei vengono eliminati alla fine della richiesta.

### <a name="scoped"></a>Con ambito

I servizi con durata con ambito (<xref:Microsoft.Extensions.DependencyInjection.ServiceCollectionServiceExtensions.AddScoped*>) vengono creati una volta per ogni richiesta client (connessione).

Nelle app che elaborano le richieste, i servizi con ambito vengono eliminati alla fine della richiesta.

> [!WARNING]
> Quando si usa un servizio con ambito in un middleware, inserire il servizio nel metodo `Invoke` o `InvokeAsync`. Non inserire tramite il [Costruttore Injection](xref:mvc/controllers/dependency-injection#constructor-injection) perché impone al servizio di comportarsi come un singleton. Per altre informazioni, vedere <xref:fundamentals/middleware/write#per-request-middleware-dependencies>.

### <a name="singleton"></a>Singleton

I servizi con durata singleton (<xref:Microsoft.Extensions.DependencyInjection.ServiceCollectionServiceExtensions.AddSingleton*>) vengono creati la prima volta che vengono richiesti (o quando si esegue `Startup.ConfigureServices` e viene specificata un'istanza con la registrazione del servizio). Tutte le richieste successive usano la stessa istanza. Se l'app richiede un comportamento singleton, è consigliabile consentire al contenitore del servizio di gestire la durata del servizio. Non implementare lo schema progettuale singleton e fornire codice utente per gestire la durata dell'oggetto nella classe.

Nelle app che elaborano le richieste, i servizi Singleton vengono eliminati quando il <xref:Microsoft.Extensions.DependencyInjection.ServiceProvider> viene eliminato alla chiusura dell'app.

> [!WARNING]
> Può essere pericoloso risolvere un servizio con ambito da un singleton. Ciò potrebbe causare uno stato non corretto per il servizio durante l'elaborazione delle richieste successive.

## <a name="service-registration-methods"></a>Metodi di registrazione del servizio

I metodi di estensione per la registrazione del servizio offrono overload che risultano utili in scenari specifici.

| Metodo | Automatico<br>object<br>eliminazione | Più elementi<br>implementazioni | Pass args |
| ------ | :-----------------------------: | :-------------------------: | :-------: |
| `Add{LIFETIME}<{SERVICE}, {IMPLEMENTATION}>()`<br>Esempio:<br>`services.AddSingleton<IMyDep, MyDep>();` | Sì | Sì | No |
| `Add{LIFETIME}<{SERVICE}>(sp => new {IMPLEMENTATION})`<br>Esempi:<br>`services.AddSingleton<IMyDep>(sp => new MyDep());`<br>`services.AddSingleton<IMyDep>(sp => new MyDep("A string!"));` | Sì | Sì | Sì |
| `Add{LIFETIME}<{IMPLEMENTATION}>()`<br>Esempio:<br>`services.AddSingleton<MyDep>();` | Sì | No | No |
| `AddSingleton<{SERVICE}>(new {IMPLEMENTATION})`<br>Esempi:<br>`services.AddSingleton<IMyDep>(new MyDep());`<br>`services.AddSingleton<IMyDep>(new MyDep("A string!"));` | No | Sì | Sì |
| `AddSingleton(new {IMPLEMENTATION})`<br>Esempi:<br>`services.AddSingleton(new MyDep());`<br>`services.AddSingleton(new MyDep("A string!"));` | No | No | Sì |

Per ulteriori informazioni sull'eliminazione dei tipi, vedere la sezione [Eliminazione dei servizi](#disposal-of-services). Uno scenario comune per più implementazioni è costituito dai [tipi di simulazioni per i test](xref:test/integration-tests#inject-mock-services).

La registrazione di un servizio con un solo tipo di implementazione equivale alla registrazione del servizio con lo stesso tipo di implementazione e di servizio. Questo è il motivo per cui non è possibile registrare più implementazioni di un servizio usando i metodi che non accettano un tipo di servizio esplicito. Questi metodi possono registrare più *istanze* di un servizio, ma avranno tutti lo stesso tipo di *implementazione* .

Uno qualsiasi dei metodi di registrazione del servizio sopra indicati può essere utilizzato per registrare più istanze del servizio dello stesso tipo di servizio. Nell'esempio seguente `AddSingleton` viene chiamato due volte con `IMyDependency` come tipo di servizio. La seconda chiamata a `AddSingleton` sostituisce quella precedente quando viene risolta come `IMyDependency` e aggiunge a quella precedente quando più servizi vengono risolti tramite `IEnumerable<IMyDependency>` . I servizi vengono visualizzati nell'ordine in cui sono stati registrati durante la risoluzione tramite `IEnumerable<{SERVICE}>` .

```csharp
services.AddSingleton<IMyDependency, MyDependency>();
services.AddSingleton<IMyDependency, DifferentDependency>();

public class MyService
{
    public MyService(IMyDependency myDependency, 
       IEnumerable<IMyDependency> myDependencies)
    {
        Trace.Assert(myDependency is DifferentDependency);

        var dependencyArray = myDependencies.ToArray();
        Trace.Assert(dependencyArray[0] is MyDependency);
        Trace.Assert(dependencyArray[1] is DifferentDependency);
    }
}
```

Il Framework fornisce anche `TryAdd{LIFETIME}` metodi di estensione, che registrano il servizio solo se non è già stata registrata un'implementazione.

Nell'esempio seguente, la chiamata a `AddSingleton` Registra `MyDependency` come implementazione per `IMyDependency` . La chiamata a `TryAddSingleton` non ha alcun effetto perché `IMyDependency` dispone già di un'implementazione registrata.

```csharp
services.AddSingleton<IMyDependency, MyDependency>();
// The following line has no effect:
services.TryAddSingleton<IMyDependency, DifferentDependency>();

public class MyService
{
    public MyService(IMyDependency myDependency, 
        IEnumerable<IMyDependency> myDependencies)
    {
        Trace.Assert(myDependency is MyDependency);
        Trace.Assert(myDependencies.Single() is MyDependency);
    }
}
```

Per altre informazioni, vedere:

* <xref:Microsoft.Extensions.DependencyInjection.Extensions.ServiceCollectionDescriptorExtensions.TryAdd*>
* <xref:Microsoft.Extensions.DependencyInjection.Extensions.ServiceCollectionDescriptorExtensions.TryAddTransient*>
* <xref:Microsoft.Extensions.DependencyInjection.Extensions.ServiceCollectionDescriptorExtensions.TryAddScoped*>
* <xref:Microsoft.Extensions.DependencyInjection.Extensions.ServiceCollectionDescriptorExtensions.TryAddSingleton*>

I metodi [TryAddEnumerable(ServiceDescriptor)](xref:Microsoft.Extensions.DependencyInjection.Extensions.ServiceCollectionDescriptorExtensions.TryAddEnumerable*) registrano il servizio solo se non esiste già un'implementazione *dello stesso tipo*. Più servizi vengono risolti tramite `IEnumerable<{SERVICE}>`. Durante la registrazione di servizi, lo sviluppatore desidera aggiungere un'istanza solo se non ne è già stata aggiunta una dello stesso tipo. In genere, questo metodo viene utilizzato dagli autori di librerie per evitare di registrare due copie di un'istanza nel contenitore.

Nell'esempio seguente, la prima riga registra `MyDep` per `IMyDep1`. La seconda riga registra `MyDep` per `IMyDep2`. La seconda riga non ha alcun effetto perché `IMyDep1` dispone già di un'implementazione registrata di `MyDep`:

```csharp
public interface IMyDep1 {}
public interface IMyDep2 {}

public class MyDep : IMyDep1, IMyDep2 {}

services.TryAddEnumerable(ServiceDescriptor.Singleton<IMyDep1, MyDep>());
services.TryAddEnumerable(ServiceDescriptor.Singleton<IMyDep2, MyDep>());
// Two registrations of MyDep for IMyDep1 is avoided by the following line:
services.TryAddEnumerable(ServiceDescriptor.Singleton<IMyDep1, MyDep>());
```

### <a name="constructor-injection-behavior"></a>Comportamento dell'inserimento del costruttore

I servizi possono essere risolti con due meccanismi:

* <xref:System.IServiceProvider>
* <xref:Microsoft.Extensions.DependencyInjection.ActivatorUtilities>: Consente la creazione di oggetti senza registrazione del servizio nel contenitore di inserimento delle dipendenze. `ActivatorUtilities` viene usato con astrazioni esposte all'utente, ad esempio helper tag, controller MVC e strumenti di associazione di modelli.

I costruttori possono accettare argomenti non forniti tramite l'inserimento di dipendenze, ma gli argomenti devono assegnare valori predefiniti.

Quando i servizi vengono risolti da `IServiceProvider` o `ActivatorUtilities` , l' [inserimento del costruttore](xref:mvc/controllers/dependency-injection#constructor-injection) richiede un costruttore *pubblico* .

Quando i servizi vengono risolti da `ActivatorUtilities` , l' [inserimento del costruttore](xref:mvc/controllers/dependency-injection#constructor-injection) richiede l'esistenza di un solo costruttore applicabile. Sebbene siano supportati gli overload dei costruttori, può essere presente un solo overload i cui argomenti possono essere tutti specificati tramite l'inserimento delle dipendenze.

## <a name="entity-framework-contexts"></a>Contesti di Entity Framework

I contesti di Entity Framework vengono in genere aggiunti al contenitore di servizi usando la [durata con ambito](#service-lifetimes) perché le operazioni di database delle app Web hanno di solito come ambito la richiesta client. L'ambito della durata predefinita è se una durata non è specificata da un overload [AddDbContext \<TContext> ](/dotnet/api/microsoft.extensions.dependencyinjection.entityframeworkservicecollectionextensions.adddbcontext) durante la registrazione del contesto del database. I servizi con una durata specificata non devono usare un contesto di database con una durata più breve rispetto al servizio.

## <a name="lifetime-and-registration-options"></a>Opzioni di durata e di registrazione

Per illustrare la differenza tra le opzioni di durata e registrazione, si considerino le interfacce seguenti che rappresentano le attività come un'operazione con un identificatore univoco, `OperationId`. A seconda del modo in cui la durata di un servizio operativo viene configurata per le interfacce seguenti, il contenitore fornisce la stessa istanza o un'istanza diversa del servizio quando richiesto da una classe:

[!code-csharp[](dependency-injection/samples/2.x/DependencyInjectionSample/Interfaces/IOperation.cs?name=snippet1)]

Le interfacce vengono implementate nella classe `Operation`. Il costruttore `Operation` genera un GUID se non ne viene fornito uno:

[!code-csharp[](dependency-injection/samples/2.x/DependencyInjectionSample/Models/Operation.cs?name=snippet1)]

Viene registrato un `OperationService` che dipende da ognuno degli altri tipi `Operation`. Quando viene richiesto `OperationService` tramite l'inserimento delle dipendenze, riceve una nuova istanza di ogni servizio o un'istanza esistente in base alla durata del servizio dipendente.

* Se i servizi temporanei vengono creati quando vengono richiesti dal contenitore, `OperationId` del servizio `IOperationTransient` è diverso da `OperationId` di `OperationService`. `OperationService` riceve una nuova istanza della classe `IOperationTransient`. La nuova istanza restituisce un `OperationId` diverso.
* Se i servizi con ambito vengono creati per ogni richiesta client, `OperationId` del servizio `IOperationScoped` corrisponde a `OperationService` all'interno di una richiesta client. In tutte le richieste client entrambi i servizi condividono un valore `OperationId` diverso.
* Se i servizi singleton e con istanza singleton vengono creati una sola volta e usati per tutte le richieste client e tutti i servizi, `OperationId` rimane costante in tutte le richieste di servizio.

[!code-csharp[](dependency-injection/samples/2.x/DependencyInjectionSample/Services/OperationService.cs?name=snippet1)]

In `Startup.ConfigureServices`, ogni tipo viene aggiunto al contenitore in base alla durata denominata:

[!code-csharp[](dependency-injection/samples/2.x/DependencyInjectionSample/Startup.cs?name=snippet1&highlight=6-9,12)]

Il servizio `IOperationSingletonInstance` usa un'istanza specifica con un ID noto di `Guid.Empty`. È chiaro quando questo tipo è in uso (il relativo GUID è composto da tutti zero).

L'app di esempio dimostra le durate degli oggetti all'interno e tra le singole richieste. L'`IndexModel` dell'app di esempio richiede ogni genere di tipo `IOperation` e `OperationService`. La pagina visualizza quindi tutti i valori `OperationId` della classe del modello di pagina e del servizio tramite assegnazioni di proprietà:

[!code-csharp[](dependency-injection/samples/2.x/DependencyInjectionSample/Pages/Index.cshtml.cs?name=snippet1&highlight=7-11,14-18,21-25)]

L'output seguente mostra i risultati delle due richieste:

**Prima richiesta:**

Operazioni del controller:

Transient: d233e165-f417-469b-a866-1cf1935d2518  
Scoped: 5d997e2d-55f5-4a64-8388-51c4e3a1ad19  
Singleton: 01271bc1-9e31-48e7-8f7c-7261b040ded9  
Instance: 00000000-0000-0000-0000-000000000000

Operazioni di `OperationService`:

Transient: c6b049eb-1318-4e31-90f1-eb2dd849ff64  
Scoped: 5d997e2d-55f5-4a64-8388-51c4e3a1ad19  
Singleton: 01271bc1-9e31-48e7-8f7c-7261b040ded9  
Instance: 00000000-0000-0000-0000-000000000000

**Seconda richiesta:**

Operazioni del controller:

Transient: b63bd538-0a37-4ff1-90ba-081c5138dda0  
Scoped: 31e820c5-4834-4d22-83fc-a60118acb9f4  
Singleton: 01271bc1-9e31-48e7-8f7c-7261b040ded9  
Instance: 00000000-0000-0000-0000-000000000000

Operazioni di `OperationService`:

Transient: c4cbacb8-36a2-436d-81c8-8c1b78808aaf  
Scoped: 31e820c5-4834-4d22-83fc-a60118acb9f4  
Singleton: 01271bc1-9e31-48e7-8f7c-7261b040ded9  
Instance: 00000000-0000-0000-0000-000000000000

Osservare i valori `OperationId` che variano all'interno della richiesta e da una richiesta e l'altra:

* Gli oggetti *temporanei* sono sempre diversi. Il valore `OperationId` temporaneo per la prima e la seconda richiesta client è diverso per entrambe le operazioni `OperationService` e in tutte le richieste client. Viene fornita una nuova istanza per ogni richiesta di servizio e richiesta client.
* Gli oggetti *con ambito* sono gli stessi all'interno di una richiesta client, ma variano nelle diverse richieste client.
* Gli oggetti *singleton* sono gli stessi per ogni oggetto e per ogni richiesta, indipendentemente dal fatto che un' `Operation` istanza venga fornita in `Startup.ConfigureServices` .

## <a name="call-services-from-main"></a>Chiamare i servizi da main

Creare un <xref:Microsoft.Extensions.DependencyInjection.IServiceScope>IServiceScope con [IServiceScopeFactory.CreateScope](xref:Microsoft.Extensions.DependencyInjection.IServiceScopeFactory.CreateScope*) per risolvere un servizio con ambito nell'ambito di applicazione dell'app. Questo approccio è utile per l'accesso a un servizio con ambito all'avvio e per l'esecuzione di attività di inizializzazione. L'esempio seguente indica come ottenere un contesto per `MyScopedService` in `Program.Main`:

```csharp
using System;
using System.Threading.Tasks;
using Microsoft.AspNetCore;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Logging;

public class Program
{
    public static async Task Main(string[] args)
    {
        var host = CreateWebHostBuilder(args).Build();

        using (var serviceScope = host.Services.CreateScope())
        {
            var services = serviceScope.ServiceProvider;

            try
            {
                var serviceContext = services.GetRequiredService<MyScopedService>();
                // Use the context here
            }
            catch (Exception ex)
            {
                var logger = services.GetRequiredService<ILogger<Program>>();
                logger.LogError(ex, "An error occurred.");
            }
        }
    
        await host.RunAsync();
    }

    public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
        WebHost.CreateDefaultBuilder(args)
            .UseStartup<Startup>();
}
```

## <a name="scope-validation"></a>Convalida dell'ambito

Quando l'app viene eseguita nell'ambiente di sviluppo, il provider di servizi predefinito esegue controlli per verificare che:

* I servizi con ambito non vengano risolti direttamente o indirettamente dal provider di servizi radice.
* I servizi con ambito non vengano inseriti direttamente o indirettamente in singleton.

Il provider di servizi radice venga creato con la chiamata di <xref:Microsoft.Extensions.DependencyInjection.ServiceCollectionContainerBuilderExtensions.BuildServiceProvider*>. La durata del provider di servizi radice corrisponde alla durata dell'app/server quando il provider viene avviato con l'app e viene eliminato alla chiusura dell'app.

I servizi con ambito vengono eliminati dal contenitore che li ha creati. Se un servizio con ambito viene creato nel contenitore radice, la durata del servizio viene di fatto convertita in singleton, perché il servizio viene eliminato solo dal contenitore radice alla chiusura dell'app/server. La convalida degli ambiti servizio rileva queste situazioni nel contesto della chiamata di `BuildServiceProvider`.

Per altre informazioni, vedere <xref:fundamentals/host/web-host#scope-validation>.   

## <a name="request-services"></a>Servizi di richiesta

I servizi disponibili all'interno di una richiesta ASP.NET Core da `HttpContext` vengono esposti tramite la raccolta [HttpContext.RequestServices](xref:Microsoft.AspNetCore.Http.HttpContext.RequestServices).

I servizi di richiesta rappresentano i servizi configurati e richiesti come parte dell'app. Quando gli oggetti specificano dipendenze, le dipendenze sono soddisfatte dai tipi individuati in `RequestServices`, non in `ApplicationServices`.

In generale, l'app non deve usare queste proprietà direttamente. Richiedere invece i tipi richiesti dalle classi tramite i costruttori di classi e consentire al framework di inserire le dipendenze. Si ottengono classi più facili da testare.

> [!NOTE]
> È consigliabile richiedere le dipendenze come parametri del costruttore per l'accesso alla raccolta `RequestServices`.

## <a name="design-services-for-dependency-injection"></a>Progettare i servizi per l'inserimento di dipendenze

Le procedure consigliate prevedono di:

* Progettare i servizi per usare l'inserimento delle dipendenze per ottenere le relative dipendenze.
* Evitare membri e classi statiche con stato. Progettare le app in modo da usare i servizi singleton, evitando così di creare uno stato globale.
* Evitare la creazione diretta di istanze delle classi dipendenti all'interno di servizi. La creazione diretta di istanze associa il codice a una determinata implementazione.
* Fare in modo che le classi siano piccole, con factoring corretto e facili da testare.

Se una classe sembra avere troppe dipendenze inserite, ciò significa in genere che la classe ha un numero eccessivo di responsabilità e sta violando il [Single Responsibility Principle (SRP)](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#single-responsibility) (principio di responsabilità singola). Tentare di effettuare il refactoring della classe spostando alcune delle responsabilità in una nuova classe. Tenere presente che Razor le classi del modello della pagina delle pagine e le classi controller MVC devono concentrarsi sulle problematiche dell'interfaccia utente. Di conseguenza, le regole business e i dettagli di implementazione di accesso ai dati devono essere inseriti in classi appropriate per queste [problematiche separate](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#separation-of-concerns).

### <a name="disposal-of-services"></a>Eliminazione dei servizi

Il contenitore chiama <xref:System.IDisposable.Dispose*> per i tipi <xref:System.IDisposable> creati. Se un'istanza viene aggiunta al contenitore dal codice utente, non viene eliminata automaticamente.

Nell'esempio seguente i servizi vengono creati dal contenitore del servizio ed eliminati automaticamente:

```csharp
public class Service1 : IDisposable {}
public class Service2 : IDisposable {}

public interface IService3 {}
public class Service3 : IService3, IDisposable {}

public void ConfigureServices(IServiceCollection services)
{
    services.AddScoped<Service1>();
    services.AddSingleton<Service2>();
    services.AddSingleton<IService3>(sp => new Service3());
}
```

Nell'esempio seguente:

* Le istanze del servizio non vengono create dal contenitore dei servizi.
* Le durate di servizio previste non sono note dal Framework.
* Il Framework non elimina automaticamente i servizi.
* Se i servizi non vengono eliminati in modo esplicito nel codice dello sviluppatore, vengono mantenuti fino a quando l'app non viene chiusa.

```csharp
public class Service1 : IDisposable {}
public class Service2 : IDisposable {}

public void ConfigureServices(IServiceCollection services)
{
    services.AddSingleton<Service1>(new Service1());
    services.AddSingleton(new Service2());
}
```

### <a name="idisposable-guidance-for-transient-and-shared-instances"></a>Linee guida IDisposable per istanze temporanee e condivise

#### <a name="transient-limited-lifetime"></a>Durata temporanea e limitata

**Scenario**

L'app richiede un' <xref:System.IDisposable> istanza con una durata temporanea per uno degli scenari seguenti:

* L'istanza viene risolta nell'ambito radice.
* L'istanza deve essere eliminata prima della fine dell'ambito.

**Soluzione**

Usare il modello factory per creare un'istanza all'esterno dell'ambito padre. In questa situazione, in genere l'app dispone di un `Create` metodo che chiama direttamente il costruttore del tipo finale. Se il tipo finale ha altre dipendenze, la factory può:

* Ricevere un oggetto <xref:System.IServiceProvider> nel relativo costruttore.
* Usare <xref:Microsoft.Extensions.DependencyInjection.ActivatorUtilities.CreateInstance%2A?displayProperty=nameWithType> per creare un'istanza dell'istanza all'esterno del contenitore, usando il contenitore per le relative dipendenze.

#### <a name="shared-instance-limited-lifetime"></a>Istanza condivisa, durata limitata

**Scenario**

L'app richiede un' <xref:System.IDisposable> istanza condivisa tra più servizi, ma <xref:System.IDisposable> deve avere una durata limitata.

**Soluzione**

Registrare l'istanza con una durata con ambito. Usare <xref:Microsoft.Extensions.DependencyInjection.IServiceScopeFactory.CreateScope%2A?displayProperty=nameWithType> per avviare e creare un nuovo oggetto <xref:Microsoft.Extensions.DependencyInjection.IServiceScope> . Usare l'ambito <xref:System.IServiceProvider> per ottenere i servizi richiesti. Eliminare l'ambito quando la durata deve terminare.

#### <a name="general-guidelines"></a>Linee guida generali

* Non registrare <xref:System.IDisposable> istanze con un ambito temporaneo. Usare invece il modello Factory.
* Non risolvere istanze temporanee o <xref:System.IDisposable> con ambito nell'ambito radice. L'unica eccezione generale si verifica quando l'app crea/ricrea ed Elimina <xref:System.IServiceProvider> , che non è un modello ideale.
* Per <xref:System.IDisposable> la ricezione di una dipendenza tramite di non è necessario che il ricevitore implementi <xref:System.IDisposable> se stesso. Il ricevitore della <xref:System.IDisposable> dipendenza non deve chiamare <xref:System.IDisposable.Dispose%2A> su tale dipendenza.
* Gli ambiti devono essere usati per controllare la durata dei servizi. Gli ambiti non sono gerarchici e non esiste alcuna connessione speciale tra gli ambiti.

## <a name="default-service-container-replacement"></a>Sostituzione del contenitore di servizi predefinito

Il contenitore dei servizi incorporato è progettato per soddisfare le esigenze del Framework e della maggior parte delle app consumer. Si consiglia di usare il contenitore predefinito a meno che non sia necessaria una funzionalità specifica che il contenitore incorporato non supporta, ad esempio:

* Inserimento di proprietà
* Inserimento in base al nome
* Contenitori figlio
* Gestione della durata personalizzata
* Supporto di `Func<T>` per l'inizializzazione differita
* Registrazione basata sulle convenzioni

Con ASP.NET Core app è possibile usare i contenitori di terze parti seguenti:

* [Autofac](https://autofac.readthedocs.io/en/latest/integration/aspnetcore.html)
* [DryIoc](https://www.nuget.org/packages/DryIoc.Microsoft.DependencyInjection)
* [Laura](https://www.nuget.org/packages/Grace.DependencyInjection.Extensions)
* [LightInject](https://github.com/seesharper/LightInject.Microsoft.DependencyInjection)
* [Lamar](https://jasperfx.github.io/lamar/)
* [Stashbox](https://github.com/z4kn4fein/stashbox-extensions-dependencyinjection)
* [Unity](https://www.nuget.org/packages/Unity.Microsoft.DependencyInjection)

### <a name="thread-safety"></a>Thread safety

Creare servizi singleton thread-safe. Se un servizio singleton ha una dipendenza in un servizio temporaneo, è necessario che anche il servizio temporaneo sia thread-safe, a seconda di come viene usato dal singleton.

Il metodo factory di un singolo servizio, ad esempio il secondo argomento di [AddSingleton \<TService> (IServiceCollection, Func \<IServiceProvider,TService> )](xref:Microsoft.Extensions.DependencyInjection.ServiceCollectionServiceExtensions.AddSingleton*), non deve essere thread-safe. Come nel caso di un costruttore di tipo (`static`), è garantito che venga chiamato una sola volta da un singolo thread.

## <a name="recommendations"></a>Consigli

* La risoluzione basata sui servizi `async/await` e `Task` non è supportata. Dato che C# non supporta i costruttori asincroni, il modello consigliato consiste nell'usare i metodi asincroni dopo avere risolto in modo sincrono il servizio.

* Evitare di archiviare i dati e la configurazione direttamente nel contenitore del servizio. Ad esempio, il carrello acquisti di un utente non dovrebbe in genere essere aggiunto al contenitore del servizio. La configurazione deve usare il [modello di opzioni](xref:fundamentals/configuration/options). Analogamente, evitare gli oggetti "contenitori di dati" che hanno la sola funzione di consentire l'accesso ad altri oggetti. È preferibile richiedere l'elemento effettivo tramite inserimento delle dipendenze.
* Evitare l'accesso statico ai servizi. Ad esempio, evitare di digitare in modo statico [IApplicationBuilder. ApplicationServices](xref:Microsoft.AspNetCore.Builder.IApplicationBuilder.ApplicationServices) per l'uso altrove.

* Evitare di usare il *modello del localizzatore di servizi*, che combina [inversione delle strategie di controllo](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#dependency-inversion) .
  * Non richiamare <xref:System.IServiceProvider.GetService*> per ottenere un'istanza del servizio quando si può usare invece di:

    **Non corretto:**

    ```csharp
    public class MyClass()
   
      public void MyMethod()
      {
          var optionsMonitor = 
              _services.GetService<IOptionsMonitor<MyOptions>>();
          var option = optionsMonitor.CurrentValue.Option;
   
          ...
      }
      ```
   
    **Corretto**:

    ```csharp
    public class MyClass
    {
        private readonly IOptionsMonitor<MyOptions> _optionsMonitor;

        public MyClass(IOptionsMonitor<MyOptions> optionsMonitor)
        {
            _optionsMonitor = optionsMonitor;
        }

        public void MyMethod()
        {
            var option = _optionsMonitor.CurrentValue.Option;

            ...
        }
    }
    ```

* Evitare di inserire una factory che risolve le dipendenze in fase di esecuzione usando <xref:System.IServiceProvider.GetService*> .
* Evitare l'accesso statico a `HttpContext` (ad esempio [IHttpContextAccessor.HttpContext](xref:Microsoft.AspNetCore.Http.IHttpContextAccessor.HttpContext)).

È tuttavia possibile che in alcuni casi queste raccomandazioni debbano essere ignorate. Le eccezioni sono rari, soprattutto casi speciali all'interno del Framework stesso.

L'inserimento di dipendenze è un'*alternativa* ai modelli di accesso agli oggetti statici/globali. Se l'inserimento di dipendenze viene usato con l'accesso agli oggetti statico i vantaggi non saranno evidenti.

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:mvc/views/dependency-injection>
* <xref:mvc/controllers/dependency-injection>
* <xref:security/authorization/dependencyinjection>
* <xref:blazor/fundamentals/dependency-injection>
* <xref:fundamentals/startup>
* <xref:fundamentals/middleware/extensibility>
* [Quattro modi per eliminare IDisposable in ASP.NET Core](https://andrewlock.net/four-ways-to-dispose-idisposables-in-asp-net-core/)
* [Scrittura di codice pulito in ASP.NET Core con inserimento delle dipendenze (MSDN)](/archive/msdn-magazine/2016/may/asp-net-writing-clean-code-in-asp-net-core-with-dependency-injection)
* [Explicit Dependencies Principle (Principio delle dipendenze esplicite)](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#explicit-dependencies)
* [Inversion of Control Containers and the Dependency Injection Pattern (Martin Fowler) (Contenitori di inversione del controllo e modello di inserimento delle dipendenze)](https://www.martinfowler.com/articles/injection.html)
* [How to register a service with multiple interfaces in ASP.NET Core DI](https://andrewlock.net/how-to-register-a-service-with-multiple-interfaces-for-in-asp-net-core-di/) (Come registrare un servizio con più interfacce in ASP.NET Core DI)

::: moniker-end
