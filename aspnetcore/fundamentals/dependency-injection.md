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
# <a name="dependency-injection-in-aspnet-core"></a><span data-ttu-id="eb861-103">Inserimento delle dipendenze in ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="eb861-103">Dependency injection in ASP.NET Core</span></span>

::: moniker range=">= aspnetcore-3.0"

<span data-ttu-id="eb861-104">Di [Kirk Larkin](https://twitter.com/serpent5), [Steve Smith](https://ardalis.com/), [Scott Addie](https://scottaddie.com)e [Brandon Dahler](https://github.com/brandondahler)</span><span class="sxs-lookup"><span data-stu-id="eb861-104">By [Kirk Larkin](https://twitter.com/serpent5), [Steve Smith](https://ardalis.com/), [Scott Addie](https://scottaddie.com), and [Brandon Dahler](https://github.com/brandondahler)</span></span>

<span data-ttu-id="eb861-105">ASP.NET Core supporta lo schema progettuale di software per l'inserimento di dipendenze, una tecnica per ottenere l'[IoC (Inversion of Control)](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#dependency-inversion) tra le classi e le relative dipendenze.</span><span class="sxs-lookup"><span data-stu-id="eb861-105">ASP.NET Core supports the dependency injection (DI) software design pattern, which is a technique for achieving [Inversion of Control (IoC)](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#dependency-inversion) between classes and their dependencies.</span></span>

<span data-ttu-id="eb861-106">Per altre informazioni specifiche per l'inserimento delle dipendenze all'interno di controller MVC, vedere <xref:mvc/controllers/dependency-injection>.</span><span class="sxs-lookup"><span data-stu-id="eb861-106">For more information specific to dependency injection within MVC controllers, see <xref:mvc/controllers/dependency-injection>.</span></span>

<span data-ttu-id="eb861-107">Per informazioni sull'uso dell'inserimento di dipendenze in applicazioni diverse dalle app Web, vedere [inserimento delle dipendenze in .NET](/dotnet/core/extensions/dependency-injection).</span><span class="sxs-lookup"><span data-stu-id="eb861-107">For information on using dependency injection in applications other than web apps, see [Dependency injection in .NET](/dotnet/core/extensions/dependency-injection).</span></span>

<span data-ttu-id="eb861-108">Per ulteriori informazioni sull'inserimento delle dipendenze delle opzioni, vedere <xref:fundamentals/configuration/options> .</span><span class="sxs-lookup"><span data-stu-id="eb861-108">For more information on dependency injection of options, see <xref:fundamentals/configuration/options>.</span></span>

<span data-ttu-id="eb861-109">In questo argomento vengono fornite informazioni sull'inserimento delle dipendenze in ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="eb861-109">This topic provides information on dependency injection in ASP.NET Core.</span></span> <span data-ttu-id="eb861-110">La documentazione principale sull'uso dell'inserimento delle dipendenze è contenuta nell' [inserimento delle dipendenze in .NET](/dotnet/core/extensions/dependency-injection).</span><span class="sxs-lookup"><span data-stu-id="eb861-110">The primary documentation on using dependency injection is contained in [Dependency injection in .NET](/dotnet/core/extensions/dependency-injection).</span></span>

<span data-ttu-id="eb861-111">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/dependency-injection/samples) ([procedura per il download](xref:index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="eb861-111">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/dependency-injection/samples) ([how to download](xref:index#how-to-download-a-sample))</span></span>

## <a name="overview-of-dependency-injection"></a><span data-ttu-id="eb861-112">Panoramica dell'inserimento delle dipendenze</span><span class="sxs-lookup"><span data-stu-id="eb861-112">Overview of dependency injection</span></span>

<span data-ttu-id="eb861-113">Una *dipendenza* è un oggetto da cui dipende un altro oggetto.</span><span class="sxs-lookup"><span data-stu-id="eb861-113">A *dependency* is an object that another object depends on.</span></span> <span data-ttu-id="eb861-114">Esaminare la `MyDependency` classe seguente con un `WriteMessage` metodo da cui dipendono altre classi:</span><span class="sxs-lookup"><span data-stu-id="eb861-114">Examine the following `MyDependency` class with a `WriteMessage` method that other classes depend on:</span></span>

```csharp
public class MyDependency
{
    public void WriteMessage(string message)
    {
        Console.WriteLine($"MyDependency.WriteMessage called. Message: {message}");
    }
}
```

<span data-ttu-id="eb861-115">Una classe può creare un'istanza della `MyDependency` classe per utilizzare il relativo `WriteMessage` metodo.</span><span class="sxs-lookup"><span data-stu-id="eb861-115">A class can create an instance of the `MyDependency` class to make use of its `WriteMessage` method.</span></span> <span data-ttu-id="eb861-116">Nell'esempio seguente, la `MyDependency` classe è una dipendenza della `IndexModel` classe:</span><span class="sxs-lookup"><span data-stu-id="eb861-116">In the following example, the `MyDependency` class is a dependency of the `IndexModel` class:</span></span>

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

<span data-ttu-id="eb861-117">La classe crea e dipende direttamente dalla `MyDependency` classe.</span><span class="sxs-lookup"><span data-stu-id="eb861-117">The class creates and directly depends on the `MyDependency` class.</span></span> <span data-ttu-id="eb861-118">Le dipendenze del codice, ad esempio nell'esempio precedente, sono problematiche e devono essere evitate per i motivi seguenti:</span><span class="sxs-lookup"><span data-stu-id="eb861-118">Code dependencies, such as in the previous example, are problematic and should be avoided for the following reasons:</span></span>

* <span data-ttu-id="eb861-119">Per sostituire `MyDependency` con un'implementazione diversa, `IndexModel` è necessario modificare la classe.</span><span class="sxs-lookup"><span data-stu-id="eb861-119">To replace `MyDependency` with a different implementation, the `IndexModel` class must be modified.</span></span>
* <span data-ttu-id="eb861-120">Se `MyDependency` dispone di dipendenze, devono anche essere configurate dalla `IndexModel` classe.</span><span class="sxs-lookup"><span data-stu-id="eb861-120">If `MyDependency` has dependencies, they must also be configured by the `IndexModel` class.</span></span> <span data-ttu-id="eb861-121">In un progetto di grandi dimensioni con più classi che dipendono da `MyDependency`, il codice di configurazione diventa sparso in tutta l'app.</span><span class="sxs-lookup"><span data-stu-id="eb861-121">In a large project with multiple classes depending on `MyDependency`, the configuration code becomes scattered across the app.</span></span>
* <span data-ttu-id="eb861-122">È difficile eseguire unit test di questa implementazione.</span><span class="sxs-lookup"><span data-stu-id="eb861-122">This implementation is difficult to unit test.</span></span> <span data-ttu-id="eb861-123">L'app dovrebbe usare una classe `MyDependency` fittizia o stub, ma ciò non è possibile con questo approccio.</span><span class="sxs-lookup"><span data-stu-id="eb861-123">The app should use a mock or stub `MyDependency` class, which isn't possible with this approach.</span></span>

<span data-ttu-id="eb861-124">L'inserimento delle dipendenze consente di risolvere questi problemi tramite:</span><span class="sxs-lookup"><span data-stu-id="eb861-124">Dependency injection addresses these problems through:</span></span>

* <span data-ttu-id="eb861-125">L'uso di un'interfaccia o di una classe di base per astrarre l'implementazione delle dipendenze.</span><span class="sxs-lookup"><span data-stu-id="eb861-125">The use of an interface or base class to abstract the dependency implementation.</span></span>
* <span data-ttu-id="eb861-126">La registrazione della dipendenza in un contenitore di servizi.</span><span class="sxs-lookup"><span data-stu-id="eb861-126">Registration of the dependency in a service container.</span></span> <span data-ttu-id="eb861-127">ASP.NET Core offre il contenitore di servizi predefinito <xref:System.IServiceProvider>.</span><span class="sxs-lookup"><span data-stu-id="eb861-127">ASP.NET Core provides a built-in service container, <xref:System.IServiceProvider>.</span></span> <span data-ttu-id="eb861-128">I servizi vengono in genere registrati nel metodo dell'app `Startup.ConfigureServices` .</span><span class="sxs-lookup"><span data-stu-id="eb861-128">Services are typically registered in the app's `Startup.ConfigureServices` method.</span></span>
* <span data-ttu-id="eb861-129">L'*inserimento* del servizio nel costruttore della classe in cui viene usato.</span><span class="sxs-lookup"><span data-stu-id="eb861-129">*Injection* of the service into the constructor of the class where it's used.</span></span> <span data-ttu-id="eb861-130">Il framework si assume la responsabilità della creazione di un'istanza della dipendenza e della sua eliminazione quando non è più necessaria.</span><span class="sxs-lookup"><span data-stu-id="eb861-130">The framework takes on the responsibility of creating an instance of the dependency and disposing of it when it's no longer needed.</span></span>

<span data-ttu-id="eb861-131">Nell' [app di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/dependency-injection/samples), l' `IMyDependency` interfaccia definisce il `WriteMessage` Metodo:</span><span class="sxs-lookup"><span data-stu-id="eb861-131">In the [sample app](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/dependency-injection/samples), the `IMyDependency` interface defines the `WriteMessage` method:</span></span>

[!code-csharp[](dependency-injection/samples/3.x/DependencyInjectionSample/Interfaces/IMyDependency.cs?name=snippet1)]

<span data-ttu-id="eb861-132">Questa interfaccia viene implementata da un tipo concreto, `MyDependency`:</span><span class="sxs-lookup"><span data-stu-id="eb861-132">This interface is implemented by a concrete type, `MyDependency`:</span></span>

[!code-csharp[](dependency-injection/samples/3.x/DependencyInjectionSample/Services/MyDependency.cs?name=snippet1)]

<span data-ttu-id="eb861-133">L'app di esempio registra il `IMyDependency` servizio con il tipo concreto `MyDependency` .</span><span class="sxs-lookup"><span data-stu-id="eb861-133">The sample app registers the `IMyDependency` service with the concrete type `MyDependency`.</span></span> <span data-ttu-id="eb861-134">Il <xref:Microsoft.Extensions.DependencyInjection.ServiceCollectionServiceExtensions.AddScoped%2A> metodo registra il servizio con una durata con ambito, la durata di una singola richiesta.</span><span class="sxs-lookup"><span data-stu-id="eb861-134">The <xref:Microsoft.Extensions.DependencyInjection.ServiceCollectionServiceExtensions.AddScoped%2A> method registers the service with a scoped lifetime, the lifetime of a single request.</span></span> <span data-ttu-id="eb861-135">Le [durate dei servizi](#service-lifetimes) sono descritte più avanti in questo argomento.</span><span class="sxs-lookup"><span data-stu-id="eb861-135">[Service lifetimes](#service-lifetimes) are described later in this topic.</span></span>

[!code-csharp[](dependency-injection/samples/3.x/DependencyInjectionSample/StartupMyDependency.cs?name=snippet1)]

<span data-ttu-id="eb861-136">Nell'app di esempio, il `IMyDependency` servizio viene richiesto e usato per chiamare il `WriteMessage` Metodo:</span><span class="sxs-lookup"><span data-stu-id="eb861-136">In the sample app, the `IMyDependency` service is requested and used to call the `WriteMessage` method:</span></span>

[!code-csharp[](dependency-injection/samples/3.x/DependencyInjectionSample/Pages/Index2.cshtml.cs?name=snippet1)]

<span data-ttu-id="eb861-137">Utilizzando il modello DI, il controller:</span><span class="sxs-lookup"><span data-stu-id="eb861-137">By using the DI pattern, the controller:</span></span>

* <span data-ttu-id="eb861-138">Non usa il tipo concreto `MyDependency` , bensì solo l' `IMyDependency` interfaccia che implementa.</span><span class="sxs-lookup"><span data-stu-id="eb861-138">Doesn't use the concrete type `MyDependency`, only the `IMyDependency` interface it implements.</span></span> <span data-ttu-id="eb861-139">Questo consente di modificare facilmente l'implementazione utilizzata dal controller senza modificare il controller.</span><span class="sxs-lookup"><span data-stu-id="eb861-139">That makes it easy to change the implementation that the controller uses without modifying the controller.</span></span>
* <span data-ttu-id="eb861-140">Non crea un'istanza di `MyDependency` , viene creata dal contenitore di.</span><span class="sxs-lookup"><span data-stu-id="eb861-140">Doesn't create an instance of `MyDependency`, it's created by the DI container.</span></span>

<span data-ttu-id="eb861-141">L'implementazione dell' `IMyDependency` interfaccia può essere migliorata usando l'API di registrazione incorporata:</span><span class="sxs-lookup"><span data-stu-id="eb861-141">The implementation of the `IMyDependency` interface can be improved by using the built-in logging API:</span></span>

[!code-csharp[](dependency-injection/samples/3.x/DependencyInjectionSample/Services/MyDependency.cs?name=snippet2)]

<span data-ttu-id="eb861-142">Il `ConfigureServices` metodo aggiornato registra la nuova `IMyDependency` implementazione:</span><span class="sxs-lookup"><span data-stu-id="eb861-142">The updated `ConfigureServices` method registers the new `IMyDependency` implementation:</span></span>

[!code-csharp[](dependency-injection/samples/3.x/DependencyInjectionSample/StartupMyDependency2.cs?name=snippet1)]

<span data-ttu-id="eb861-143">`MyDependency2` dipende da <xref:Microsoft.Extensions.Logging.ILogger%601> , che richiede nel costruttore.</span><span class="sxs-lookup"><span data-stu-id="eb861-143">`MyDependency2` depends on <xref:Microsoft.Extensions.Logging.ILogger%601>, which it requests in the constructor.</span></span> <span data-ttu-id="eb861-144">`ILogger<TCategoryName>` è un [servizio fornito dal Framework](#framework-provided-services).</span><span class="sxs-lookup"><span data-stu-id="eb861-144">`ILogger<TCategoryName>` is a [framework-provided service](#framework-provided-services).</span></span>

<span data-ttu-id="eb861-145">Non è insolito usare l'inserimento delle dipendenze in modo concatenato.</span><span class="sxs-lookup"><span data-stu-id="eb861-145">It's not unusual to use dependency injection in a chained fashion.</span></span> <span data-ttu-id="eb861-146">Ogni dipendenza richiesta richiede a sua volta le proprie dipendenze.</span><span class="sxs-lookup"><span data-stu-id="eb861-146">Each requested dependency in turn requests its own dependencies.</span></span> <span data-ttu-id="eb861-147">Il contenitore risolve le dipendenze nel grafico e restituisce il servizio completamente risolto.</span><span class="sxs-lookup"><span data-stu-id="eb861-147">The container resolves the dependencies in the graph and returns the fully resolved service.</span></span> <span data-ttu-id="eb861-148">Il set di dipendenze che devono essere risolte viene generalmente chiamato *albero delle dipendenze* o *grafico dipendenze* o *grafico degli oggetti*.</span><span class="sxs-lookup"><span data-stu-id="eb861-148">The collective set of dependencies that must be resolved is typically referred to as a *dependency tree*, *dependency graph*, or *object graph*.</span></span>

<span data-ttu-id="eb861-149">Il contenitore si risolve sfruttando `ILogger<TCategoryName>` i vantaggi dei [tipi aperti (generici)](/dotnet/csharp/language-reference/language-specification/types#open-and-closed-types), eliminando la necessità di registrare tutti i tipi [costruiti (generici)](/dotnet/csharp/language-reference/language-specification/types#constructed-types).</span><span class="sxs-lookup"><span data-stu-id="eb861-149">The container resolves `ILogger<TCategoryName>` by taking advantage of [(generic) open types](/dotnet/csharp/language-reference/language-specification/types#open-and-closed-types), eliminating the need to register every [(generic) constructed type](/dotnet/csharp/language-reference/language-specification/types#constructed-types).</span></span>

<span data-ttu-id="eb861-150">Nella terminologia relativa all'inserimento delle dipendenze, un servizio:</span><span class="sxs-lookup"><span data-stu-id="eb861-150">In dependency injection terminology, a service:</span></span>

* <span data-ttu-id="eb861-151">È in genere un oggetto che fornisce un servizio ad altri oggetti, ad esempio il `IMyDependency` servizio.</span><span class="sxs-lookup"><span data-stu-id="eb861-151">Is typically an object that provides a service to other objects, such as the `IMyDependency` service.</span></span>
* <span data-ttu-id="eb861-152">Non è correlato a un servizio Web, anche se il servizio può utilizzare un servizio Web.</span><span class="sxs-lookup"><span data-stu-id="eb861-152">Is not related to a web service, although the service may use a web service.</span></span>

<span data-ttu-id="eb861-153">Il Framework fornisce un sistema di [registrazione](xref:fundamentals/logging/index) affidabile.</span><span class="sxs-lookup"><span data-stu-id="eb861-153">The framework provides a robust [logging](xref:fundamentals/logging/index) system.</span></span> <span data-ttu-id="eb861-154">Le `IMyDependency` implementazioni illustrate negli esempi precedenti sono state scritte per illustrare il di base di, non per implementare la registrazione.</span><span class="sxs-lookup"><span data-stu-id="eb861-154">The `IMyDependency` implementations shown in the preceding examples were written to demonstrate basic DI, not to implement logging.</span></span> <span data-ttu-id="eb861-155">La maggior parte delle app non deve scrivere logger.</span><span class="sxs-lookup"><span data-stu-id="eb861-155">Most apps shouldn't need to write loggers.</span></span> <span data-ttu-id="eb861-156">Il codice seguente illustra l'uso della registrazione predefinita, che non richiede la registrazione dei servizi in `ConfigureServices` :</span><span class="sxs-lookup"><span data-stu-id="eb861-156">The following code demonstrates using the default logging, which doesn't require any services to be registered in `ConfigureServices`:</span></span>

[!code-csharp[](dependency-injection/samples/3.x/DependencyInjectionSample/Pages/About.cshtml.cs?name=snippet)]

<span data-ttu-id="eb861-157">Con il codice precedente non è necessario aggiornare `ConfigureServices` , perché la [registrazione](xref:fundamentals/logging/index) viene fornita dal Framework.</span><span class="sxs-lookup"><span data-stu-id="eb861-157">Using the preceding code, there is no need to update `ConfigureServices`, because [logging](xref:fundamentals/logging/index) is provided by the framework.</span></span>

## <a name="services-injected-into-startup"></a><span data-ttu-id="eb861-158">Servizi inseriti all'avvio</span><span class="sxs-lookup"><span data-stu-id="eb861-158">Services injected into Startup</span></span>

<span data-ttu-id="eb861-159">I servizi possono essere inseriti nel `Startup` costruttore e nel `Startup.Configure` metodo.</span><span class="sxs-lookup"><span data-stu-id="eb861-159">Services can be injected into the `Startup` constructor and the `Startup.Configure` method.</span></span>

<span data-ttu-id="eb861-160">`Startup`Quando si utilizza l'host generico () è possibile inserire nel costruttore solo i servizi seguenti <xref:Microsoft.Extensions.Hosting.IHostBuilder> :</span><span class="sxs-lookup"><span data-stu-id="eb861-160">Only the following services can be injected into the `Startup` constructor when using the Generic Host (<xref:Microsoft.Extensions.Hosting.IHostBuilder>):</span></span>

* <xref:Microsoft.AspNetCore.Hosting.IWebHostEnvironment>
* <xref:Microsoft.Extensions.Hosting.IHostEnvironment>
* <xref:Microsoft.Extensions.Configuration.IConfiguration>

<span data-ttu-id="eb861-161">Qualsiasi servizio registrato con il contenitore DI inserimento delle dipendenze può essere inserito nel `Startup.Configure` Metodo:</span><span class="sxs-lookup"><span data-stu-id="eb861-161">Any service registered with the DI container can be injected into the `Startup.Configure` method:</span></span>

```csharp
public void Configure(IApplicationBuilder app, ILogger<Startup> logger)
{
    ...
}
```

<span data-ttu-id="eb861-162">Per ulteriori informazioni, vedere <xref:fundamentals/startup> e [accedere alla configurazione all'avvio](xref:fundamentals/configuration/index#access-configuration-in-startup).</span><span class="sxs-lookup"><span data-stu-id="eb861-162">For more information, see <xref:fundamentals/startup> and [Access configuration in Startup](xref:fundamentals/configuration/index#access-configuration-in-startup).</span></span>

## <a name="register-groups-of-services-with-extension-methods"></a><span data-ttu-id="eb861-163">Registrare gruppi di servizi con i metodi di estensione</span><span class="sxs-lookup"><span data-stu-id="eb861-163">Register groups of services with extension methods</span></span>

<span data-ttu-id="eb861-164">Il framework ASP.NET Core usa una convenzione per la registrazione di un gruppo di servizi correlati.</span><span class="sxs-lookup"><span data-stu-id="eb861-164">The ASP.NET Core framework uses a convention for registering a group of related services.</span></span> <span data-ttu-id="eb861-165">La convenzione prevede l'uso di un singolo `Add{GROUP_NAME}` metodo di estensione per registrare tutti i servizi richiesti da una funzionalità del Framework.</span><span class="sxs-lookup"><span data-stu-id="eb861-165">The convention is to use a single `Add{GROUP_NAME}` extension method to register all of the services required by a framework feature.</span></span> <span data-ttu-id="eb861-166">Il metodo di estensione, ad esempio, <xref:Microsoft.Extensions.DependencyInjection.MvcServiceCollectionExtensions.AddControllers%2A> registra i servizi necessari per i controller MVC.</span><span class="sxs-lookup"><span data-stu-id="eb861-166">For example, the <xref:Microsoft.Extensions.DependencyInjection.MvcServiceCollectionExtensions.AddControllers%2A> extension method registers the services required for MVC controllers.</span></span>

<span data-ttu-id="eb861-167">Il codice seguente viene generato dal Razor modello di pagine usando singoli account utente e Mostra come aggiungere altri servizi al contenitore usando i metodi di estensione <xref:Microsoft.Extensions.DependencyInjection.EntityFrameworkServiceCollectionExtensions.AddDbContext%2A> e <xref:Microsoft.Extensions.DependencyInjection.IdentityServiceCollectionUIExtensions.AddDefaultIdentity%2A> :</span><span class="sxs-lookup"><span data-stu-id="eb861-167">The following code is generated by the Razor Pages template using individual user accounts and shows how to add additional services to the container using the extension methods <xref:Microsoft.Extensions.DependencyInjection.EntityFrameworkServiceCollectionExtensions.AddDbContext%2A> and <xref:Microsoft.Extensions.DependencyInjection.IdentityServiceCollectionUIExtensions.AddDefaultIdentity%2A>:</span></span>

[!code-csharp[](dependency-injection/samples/3.x/DependencyInjectionSample/StartupEF.cs?name=snippet)]

[!INCLUDE[](~/includes/combine-di.md)]

## <a name="service-lifetimes"></a><span data-ttu-id="eb861-168">Durate del servizio</span><span class="sxs-lookup"><span data-stu-id="eb861-168">Service lifetimes</span></span>

<span data-ttu-id="eb861-169">Vedere la [durata del servizio](/dotnet/core/extensions/dependency-injection#service-lifetimes) nell' [inserimento delle dipendenze in .NET](/dotnet/core/extensions/dependency-injection)</span><span class="sxs-lookup"><span data-stu-id="eb861-169">See [Service lifetimes](/dotnet/core/extensions/dependency-injection#service-lifetimes) in [Dependency injection in .NET](/dotnet/core/extensions/dependency-injection)</span></span>

<span data-ttu-id="eb861-170">Per usare i servizi con ambito nel middleware, usare uno degli approcci seguenti:</span><span class="sxs-lookup"><span data-stu-id="eb861-170">To use scoped services in middleware, use one of the following approaches:</span></span>

* <span data-ttu-id="eb861-171">Inserire il servizio nel middleware o nel `Invoke` `InvokeAsync` metodo.</span><span class="sxs-lookup"><span data-stu-id="eb861-171">Inject the service into the middleware's `Invoke` or `InvokeAsync` method.</span></span> <span data-ttu-id="eb861-172">L'uso dell' [inserimento del costruttore](xref:mvc/controllers/dependency-injection#constructor-injection) genera un'eccezione in fase di esecuzione perché impone il comportamento del servizio con ambito come un singleton.</span><span class="sxs-lookup"><span data-stu-id="eb861-172">Using [constructor injection](xref:mvc/controllers/dependency-injection#constructor-injection) throws a runtime exception because it forces the scoped service to behave like a singleton.</span></span> <span data-ttu-id="eb861-173">Nell'esempio nella sezione relativa alle [Opzioni di durata e registrazione](#lifetime-and-registration-options) viene illustrato l' `InvokeAsync` approccio.</span><span class="sxs-lookup"><span data-stu-id="eb861-173">The sample in the [Lifetime and registration options](#lifetime-and-registration-options) section demonstrates the `InvokeAsync` approach.</span></span>
* <span data-ttu-id="eb861-174">Usare il [middleware basato su Factory](xref:fundamentals/middleware/extensibility).</span><span class="sxs-lookup"><span data-stu-id="eb861-174">Use [Factory-based middleware](xref:fundamentals/middleware/extensibility).</span></span> <span data-ttu-id="eb861-175">Il middleware registrato con questo approccio viene attivato per ogni richiesta client (connessione), che consente di inserire i servizi con ambito nel metodo del middleware `InvokeAsync` .</span><span class="sxs-lookup"><span data-stu-id="eb861-175">Middleware registered using this approach is activated per client request (connection), which allows scoped services to be injected into the middleware's `InvokeAsync` method.</span></span>

<span data-ttu-id="eb861-176">Per altre informazioni, vedere <xref:fundamentals/middleware/write#per-request-middleware-dependencies>.</span><span class="sxs-lookup"><span data-stu-id="eb861-176">For more information, see <xref:fundamentals/middleware/write#per-request-middleware-dependencies>.</span></span>

## <a name="service-registration-methods"></a><span data-ttu-id="eb861-177">Metodi di registrazione del servizio</span><span class="sxs-lookup"><span data-stu-id="eb861-177">Service registration methods</span></span>

<span data-ttu-id="eb861-178">Vedere i [metodi di registrazione del servizio](/dotnet/core/extensions/dependency-injection#service-registration-methods) in [inserimento delle dipendenze in .NET](/dotnet/core/extensions/dependency-injection)</span><span class="sxs-lookup"><span data-stu-id="eb861-178">See [Service registration methods](/dotnet/core/extensions/dependency-injection#service-registration-methods) in [Dependency injection in .NET](/dotnet/core/extensions/dependency-injection)</span></span>

 <span data-ttu-id="eb861-179">Quando si [simulano i tipi per il test](xref:test/integration-tests#inject-mock-services), è comune usare più implementazioni.</span><span class="sxs-lookup"><span data-stu-id="eb861-179">It's common to use multiple implementations when [mocking types for testing](xref:test/integration-tests#inject-mock-services).</span></span>

<span data-ttu-id="eb861-180">La registrazione di un servizio con un solo tipo di implementazione equivale alla registrazione del servizio con lo stesso tipo di implementazione e di servizio.</span><span class="sxs-lookup"><span data-stu-id="eb861-180">Registering a service with only an implementation type is equivalent to registering that service with the same implementation and service type.</span></span> <span data-ttu-id="eb861-181">Questo è il motivo per cui non è possibile registrare più implementazioni di un servizio usando i metodi che non accettano un tipo di servizio esplicito.</span><span class="sxs-lookup"><span data-stu-id="eb861-181">This is why multiple implementations of a service cannot be registered using the methods that don't take an explicit service type.</span></span> <span data-ttu-id="eb861-182">Questi metodi possono registrare più *istanze* di un servizio, ma avranno tutti lo stesso tipo di *implementazione* .</span><span class="sxs-lookup"><span data-stu-id="eb861-182">These methods can register multiple *instances* of a service, but they will all have the same *implementation* type.</span></span>

<span data-ttu-id="eb861-183">Uno qualsiasi dei metodi di registrazione del servizio sopra indicati può essere utilizzato per registrare più istanze del servizio dello stesso tipo di servizio.</span><span class="sxs-lookup"><span data-stu-id="eb861-183">Any of the above service registration methods can be used to register multiple service instances of the same service type.</span></span> <span data-ttu-id="eb861-184">Nell'esempio seguente `AddSingleton` viene chiamato due volte con `IMyDependency` come tipo di servizio.</span><span class="sxs-lookup"><span data-stu-id="eb861-184">In the following example, `AddSingleton` is called twice with `IMyDependency` as the service type.</span></span> <span data-ttu-id="eb861-185">La seconda chiamata a `AddSingleton` sostituisce quella precedente quando viene risolta come `IMyDependency` e aggiunge a quella precedente quando più servizi vengono risolti tramite `IEnumerable<IMyDependency>` .</span><span class="sxs-lookup"><span data-stu-id="eb861-185">The second call to `AddSingleton` overrides the previous one when resolved as `IMyDependency` and adds to the previous one when multiple services are resolved via `IEnumerable<IMyDependency>`.</span></span> <span data-ttu-id="eb861-186">I servizi vengono visualizzati nell'ordine in cui sono stati registrati durante la risoluzione tramite `IEnumerable<{SERVICE}>` .</span><span class="sxs-lookup"><span data-stu-id="eb861-186">Services appear in the order they were registered when resolved via `IEnumerable<{SERVICE}>`.</span></span>

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

## <a name="constructor-injection-behavior"></a><span data-ttu-id="eb861-187">Comportamento dell'inserimento del costruttore</span><span class="sxs-lookup"><span data-stu-id="eb861-187">Constructor injection behavior</span></span>

<span data-ttu-id="eb861-188">Vedere [comportamento di inserimento del costruttore](/dotnet/core/extensions/dependency-injection#constructor-injection-behavior) nell' [inserimento delle dipendenze in .NET](/dotnet/core/extensions/dependency-injection)</span><span class="sxs-lookup"><span data-stu-id="eb861-188">See [Constructor injection behavior](/dotnet/core/extensions/dependency-injection#constructor-injection-behavior) in [Dependency injection in .NET](/dotnet/core/extensions/dependency-injection)</span></span>

## <a name="entity-framework-contexts"></a><span data-ttu-id="eb861-189">Contesti di Entity Framework</span><span class="sxs-lookup"><span data-stu-id="eb861-189">Entity Framework contexts</span></span>

<span data-ttu-id="eb861-190">Per impostazione predefinita, i contesti di Entity Framework vengono aggiunti al contenitore del servizio usando la [durata con ambito](#service-lifetimes) poiché le operazioni del database dell'app Web sono in genere limitate all'ambito della richiesta client.</span><span class="sxs-lookup"><span data-stu-id="eb861-190">By default, Entity Framework contexts are added to the service container using the [scoped lifetime](#service-lifetimes) because web app database operations are normally scoped to the client request.</span></span> <span data-ttu-id="eb861-191">Per usare una durata diversa, specificare la durata usando un <xref:Microsoft.Extensions.DependencyInjection.EntityFrameworkServiceCollectionExtensions.AddDbContext%2A> Overload.</span><span class="sxs-lookup"><span data-stu-id="eb861-191">To use a different lifetime, specify the lifetime by using an <xref:Microsoft.Extensions.DependencyInjection.EntityFrameworkServiceCollectionExtensions.AddDbContext%2A> overload.</span></span> <span data-ttu-id="eb861-192">I servizi di una determinata durata non devono utilizzare un contesto di database con una durata minore della durata del servizio.</span><span class="sxs-lookup"><span data-stu-id="eb861-192">Services of a given lifetime shouldn't use a database context with a lifetime that's shorter than the service's lifetime.</span></span>

## <a name="lifetime-and-registration-options"></a><span data-ttu-id="eb861-193">Opzioni di durata e di registrazione</span><span class="sxs-lookup"><span data-stu-id="eb861-193">Lifetime and registration options</span></span>

<span data-ttu-id="eb861-194">Per dimostrare la differenza tra le durate dei servizi e le relative opzioni di registrazione, prendere in considerazione le interfacce seguenti che rappresentano un'attività come operazione con un identificatore `OperationId` .</span><span class="sxs-lookup"><span data-stu-id="eb861-194">To demonstrate the difference between service lifetimes and their registration options, consider the following interfaces that represent a task as an operation with an identifier, `OperationId`.</span></span> <span data-ttu-id="eb861-195">A seconda del modo in cui viene configurata la durata del servizio di un'operazione per le interfacce seguenti, il contenitore fornisce le stesse istanze o diverse del servizio quando richiesto da una classe:</span><span class="sxs-lookup"><span data-stu-id="eb861-195">Depending on how the lifetime of an operation's service is configured for the following interfaces, the container provides either the same or different instances of the service when requested by a class:</span></span>

[!code-csharp[](dependency-injection/samples/3.x/DependencyInjectionSample/Interfaces/IOperation.cs?name=snippet1)]

<span data-ttu-id="eb861-196">La `Operation` classe seguente implementa tutte le interfacce precedenti.</span><span class="sxs-lookup"><span data-stu-id="eb861-196">The following `Operation` class implements all of the preceding interfaces.</span></span> <span data-ttu-id="eb861-197">Il `Operation` costruttore genera un GUID e archivia gli ultimi 4 caratteri nella `OperationId` proprietà:</span><span class="sxs-lookup"><span data-stu-id="eb861-197">The `Operation` constructor generates a GUID and stores the last 4 characters in the `OperationId` property:</span></span>

[!code-csharp[](dependency-injection/samples/3.x/DependencyInjectionSample/Models/Operation.cs?name=snippet1)]

<span data-ttu-id="eb861-198">Il `Startup.ConfigureServices` metodo crea più registrazioni della `Operation` classe in base alle durate denominate:</span><span class="sxs-lookup"><span data-stu-id="eb861-198">The `Startup.ConfigureServices` method creates multiple registrations of the `Operation` class according to the named lifetimes:</span></span>

[!code-csharp[](dependency-injection/samples/3.x/DependencyInjectionSample/Startup2.cs?name=snippet1)]

<span data-ttu-id="eb861-199">L'app di esempio illustra le durate degli oggetti all'interno e tra le richieste.</span><span class="sxs-lookup"><span data-stu-id="eb861-199">The sample app demonstrates object lifetimes both within and between requests.</span></span> <span data-ttu-id="eb861-200">Il `IndexModel` e il middleware richiedono ogni tipo di `IOperation` tipo e registrano `OperationId` per ogni tipo:</span><span class="sxs-lookup"><span data-stu-id="eb861-200">The `IndexModel` and the middleware request each kind of `IOperation` type and log the `OperationId` for each:</span></span>

[!code-csharp[](dependency-injection/samples/3.x/DependencyInjectionSample/Pages/Index.cshtml.cs?name=snippet1)]

<span data-ttu-id="eb861-201">Analogamente a `IndexModel` , il middleware risolve gli stessi servizi:</span><span class="sxs-lookup"><span data-stu-id="eb861-201">Similar to the `IndexModel`, the middleware resolves the same services:</span></span>

[!code-csharp[](dependency-injection/samples/3.x/DependencyInjectionSample/Middleware/MyMiddleware.cs?name=snippet)]

<span data-ttu-id="eb861-202">I servizi con ambito devono essere risolti nel `InvokeAsync` Metodo:</span><span class="sxs-lookup"><span data-stu-id="eb861-202">Scoped services must be resolved in the `InvokeAsync` method:</span></span>

[!code-csharp[](dependency-injection/samples/3.x/DependencyInjectionSample/Middleware/MyMiddleware.cs?name=snippet2&highlight=2)]

<span data-ttu-id="eb861-203">L'output del logger Mostra:</span><span class="sxs-lookup"><span data-stu-id="eb861-203">The logger output shows:</span></span>

* <span data-ttu-id="eb861-204">Gli oggetti *temporanei* sono sempre diversi.</span><span class="sxs-lookup"><span data-stu-id="eb861-204">*Transient* objects are always different.</span></span> <span data-ttu-id="eb861-205">Il `OperationId` valore temporaneo è diverso in `IndexModel` e nel middleware.</span><span class="sxs-lookup"><span data-stu-id="eb861-205">The transient `OperationId` value is different in the `IndexModel` and in the middleware.</span></span>
* <span data-ttu-id="eb861-206">Gli oggetti con *ambito* sono gli stessi per ogni richiesta, ma sono diversi in ogni richiesta.</span><span class="sxs-lookup"><span data-stu-id="eb861-206">*Scoped* objects are the same for each request but different across each request.</span></span>
* <span data-ttu-id="eb861-207">Gli oggetti *singleton* sono gli stessi per ogni richiesta.</span><span class="sxs-lookup"><span data-stu-id="eb861-207">*Singleton* objects are the same for every request.</span></span>

<span data-ttu-id="eb861-208">Per ridurre l'output di registrazione, impostare "Logging: LogLevel: Microsoft: Error" nella *appsettings.Development.jssu* file:</span><span class="sxs-lookup"><span data-stu-id="eb861-208">To reduce the logging output, set "Logging:LogLevel:Microsoft:Error" in the *appsettings.Development.json* file:</span></span>

[!code-json[](dependency-injection/samples/3.x/DependencyInjectionSample/appsettings.Development.json?highlight=7)]

## <a name="call-services-from-main"></a><span data-ttu-id="eb861-209">Chiamare i servizi da main</span><span class="sxs-lookup"><span data-stu-id="eb861-209">Call services from main</span></span>

<span data-ttu-id="eb861-210">Creare un <xref:Microsoft.Extensions.DependencyInjection.IServiceScope>IServiceScope con [IServiceScopeFactory.CreateScope](xref:Microsoft.Extensions.DependencyInjection.IServiceScopeFactory.CreateScope%2A) per risolvere un servizio con ambito nell'ambito di applicazione dell'app.</span><span class="sxs-lookup"><span data-stu-id="eb861-210">Create an <xref:Microsoft.Extensions.DependencyInjection.IServiceScope> with [IServiceScopeFactory.CreateScope](xref:Microsoft.Extensions.DependencyInjection.IServiceScopeFactory.CreateScope%2A) to resolve a scoped service within the app's scope.</span></span> <span data-ttu-id="eb861-211">Questo approccio è utile per l'accesso a un servizio con ambito all'avvio e per l'esecuzione di attività di inizializzazione.</span><span class="sxs-lookup"><span data-stu-id="eb861-211">This approach is useful to access a scoped service at startup to run initialization tasks.</span></span>

<span data-ttu-id="eb861-212">Nell'esempio seguente viene illustrato come accedere al servizio con ambito `IMyDependency` e come chiamare il relativo `WriteMessage` metodo in `Program.Main` :</span><span class="sxs-lookup"><span data-stu-id="eb861-212">The following example shows how to access the scoped `IMyDependency` service and call its `WriteMessage` method in `Program.Main`:</span></span>

[!code-csharp[](dependency-injection/samples/3.x/DependencyInjectionSample/Program.cs?name=snippet)]

<a name="sv"></a>

## <a name="scope-validation"></a><span data-ttu-id="eb861-213">Convalida dell'ambito</span><span class="sxs-lookup"><span data-stu-id="eb861-213">Scope validation</span></span>

<span data-ttu-id="eb861-214">Vedere [comportamento di inserimento del costruttore](/dotnet/core/extensions/dependency-injection#constructor-injection-behavior) nell' [inserimento delle dipendenze in .NET](/dotnet/core/extensions/dependency-injection)</span><span class="sxs-lookup"><span data-stu-id="eb861-214">See [Constructor injection behavior](/dotnet/core/extensions/dependency-injection#constructor-injection-behavior) in [Dependency injection in .NET](/dotnet/core/extensions/dependency-injection)</span></span>

<span data-ttu-id="eb861-215">Per ulteriori informazioni, vedere [Convalida dell'ambito](xref:fundamentals/host/web-host#scope-validation).</span><span class="sxs-lookup"><span data-stu-id="eb861-215">For more information, see [Scope validation](xref:fundamentals/host/web-host#scope-validation).</span></span>

## <a name="request-services"></a><span data-ttu-id="eb861-216">Servizi di richiesta</span><span class="sxs-lookup"><span data-stu-id="eb861-216">Request Services</span></span>

<span data-ttu-id="eb861-217">I servizi disponibili all'interno di una richiesta di ASP.NET Core vengono esposti tramite la raccolta [HttpContext. RequestServices](xref:Microsoft.AspNetCore.Http.HttpContext.RequestServices) .</span><span class="sxs-lookup"><span data-stu-id="eb861-217">The services available within an ASP.NET Core request are exposed through the [HttpContext.RequestServices](xref:Microsoft.AspNetCore.Http.HttpContext.RequestServices) collection.</span></span> <span data-ttu-id="eb861-218">Quando i servizi vengono richiesti dall'interno di una richiesta, i servizi e le relative dipendenze vengono risolti dalla `RequestServices` raccolta.</span><span class="sxs-lookup"><span data-stu-id="eb861-218">When services are requested from inside of a request, the services and their dependencies are resolved from the `RequestServices` collection.</span></span>

<span data-ttu-id="eb861-219">Il Framework crea un ambito per ogni richiesta ed `RequestServices` espone il provider di servizi con ambito.</span><span class="sxs-lookup"><span data-stu-id="eb861-219">The framework creates a scope per request and `RequestServices` exposes the scoped service provider.</span></span> <span data-ttu-id="eb861-220">Tutti i servizi con ambito sono validi fino a quando la richiesta è attiva.</span><span class="sxs-lookup"><span data-stu-id="eb861-220">All scoped services are valid for as long as the request is active.</span></span>

> [!NOTE]
> <span data-ttu-id="eb861-221">Preferisce richiedere le dipendenze come parametri del costruttore per la risoluzione dei servizi dalla `RequestServices` raccolta.</span><span class="sxs-lookup"><span data-stu-id="eb861-221">Prefer requesting dependencies as constructor parameters to resolving services from the `RequestServices` collection.</span></span> <span data-ttu-id="eb861-222">In questo modo, le classi risultano più facili da testare.</span><span class="sxs-lookup"><span data-stu-id="eb861-222">This results in classes that are easier to test.</span></span>

## <a name="design-services-for-dependency-injection"></a><span data-ttu-id="eb861-223">Progettare i servizi per l'inserimento di dipendenze</span><span class="sxs-lookup"><span data-stu-id="eb861-223">Design services for dependency injection</span></span>

<span data-ttu-id="eb861-224">Durante la progettazione di servizi per l'inserimento di dipendenze:</span><span class="sxs-lookup"><span data-stu-id="eb861-224">When designing services for dependency injection:</span></span>

* <span data-ttu-id="eb861-225">Evitare membri e classi statiche con stato.</span><span class="sxs-lookup"><span data-stu-id="eb861-225">Avoid stateful, static classes and members.</span></span> <span data-ttu-id="eb861-226">Evitare di creare lo stato globale progettando le app in modo da usare i servizi singleton.</span><span class="sxs-lookup"><span data-stu-id="eb861-226">Avoid creating global state by designing apps to use singleton services instead.</span></span>
* <span data-ttu-id="eb861-227">Evitare la creazione diretta di istanze delle classi dipendenti all'interno di servizi.</span><span class="sxs-lookup"><span data-stu-id="eb861-227">Avoid direct instantiation of dependent classes within services.</span></span> <span data-ttu-id="eb861-228">La creazione diretta di istanze associa il codice a una determinata implementazione.</span><span class="sxs-lookup"><span data-stu-id="eb861-228">Direct instantiation couples the code to a particular implementation.</span></span>
* <span data-ttu-id="eb861-229">Semplifica la testing dei servizi.</span><span class="sxs-lookup"><span data-stu-id="eb861-229">Make services small, well-factored, and easily tested.</span></span>

<span data-ttu-id="eb861-230">Se una classe dispone di molte dipendenze inserite, potrebbe essere un segno che la classe ha troppe responsabilità e viola il [principio di singola responsabilità (SRP)](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#single-responsibility).</span><span class="sxs-lookup"><span data-stu-id="eb861-230">If a class has a lot of injected dependencies, it might be a sign that the class has too many responsibilities and violates the [Single Responsibility Principle (SRP)](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#single-responsibility).</span></span> <span data-ttu-id="eb861-231">Tentare di effettuare il refactoring della classe spostando alcune delle responsabilità in nuove classi.</span><span class="sxs-lookup"><span data-stu-id="eb861-231">Attempt to refactor the class by moving some of its responsibilities into new classes.</span></span> <span data-ttu-id="eb861-232">Tenere presente che Razor le classi del modello della pagina delle pagine e le classi controller MVC devono concentrarsi sulle problematiche dell'interfaccia utente.</span><span class="sxs-lookup"><span data-stu-id="eb861-232">Keep in mind that Razor Pages page model classes and MVC controller classes should focus on UI concerns.</span></span>

### <a name="disposal-of-services"></a><span data-ttu-id="eb861-233">Eliminazione dei servizi</span><span class="sxs-lookup"><span data-stu-id="eb861-233">Disposal of services</span></span>

<span data-ttu-id="eb861-234">Il contenitore chiama <xref:System.IDisposable.Dispose%2A> per i tipi <xref:System.IDisposable> creati.</span><span class="sxs-lookup"><span data-stu-id="eb861-234">The container calls <xref:System.IDisposable.Dispose%2A> for the <xref:System.IDisposable> types it creates.</span></span> <span data-ttu-id="eb861-235">I servizi risolti dal contenitore non devono mai essere eliminati dallo sviluppatore.</span><span class="sxs-lookup"><span data-stu-id="eb861-235">Services resolved from the container should never be disposed by the developer.</span></span> <span data-ttu-id="eb861-236">Se un tipo o una factory viene registrata come singleton, il contenitore elimina il singleton automaticamente.</span><span class="sxs-lookup"><span data-stu-id="eb861-236">If a type or factory is registered as a singleton, the container disposes the singleton automatically.</span></span>

<span data-ttu-id="eb861-237">Nell'esempio seguente i servizi vengono creati dal contenitore del servizio ed eliminati automaticamente:</span><span class="sxs-lookup"><span data-stu-id="eb861-237">In the following example, the services are created by the service container and disposed automatically:</span></span>

[!code-csharp[](dependency-injection/samples/3.x/DIsample2/DIsample2/Services/Service1.cs?name=snippet)]

[!code-csharp[](dependency-injection/samples/3.x/DIsample2/DIsample2/Startup.cs?name=snippet)]

[!code-csharp[](dependency-injection/samples/3.x/DIsample2/DIsample2/Pages/Index.cshtml.cs?name=snippet)]

<span data-ttu-id="eb861-238">La console di debug Visualizza il seguente output dopo ogni aggiornamento della pagina di indice:</span><span class="sxs-lookup"><span data-stu-id="eb861-238">The debug console shows the following output after each refresh of the Index page:</span></span>

```console
Service1: IndexModel.OnGet
Service2: IndexModel.OnGet
Service3: IndexModel.OnGet
Service1.Dispose
```

### <a name="services-not-created-by-the-service-container"></a><span data-ttu-id="eb861-239">Servizi non creati dal contenitore del servizio</span><span class="sxs-lookup"><span data-stu-id="eb861-239">Services not created by the service container</span></span>

<span data-ttu-id="eb861-240">Osservare il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="eb861-240">Consider the following code:</span></span>

[!code-csharp[](dependency-injection/samples/3.x/DIsample2/DIsample2/Startup2.cs?name=snippet)]

<span data-ttu-id="eb861-241">Nel codice precedente:</span><span class="sxs-lookup"><span data-stu-id="eb861-241">In the preceding code:</span></span>

* <span data-ttu-id="eb861-242">Le istanze del servizio non vengono create dal contenitore dei servizi.</span><span class="sxs-lookup"><span data-stu-id="eb861-242">The service instances aren't created by the service container.</span></span>
* <span data-ttu-id="eb861-243">Il Framework non elimina automaticamente i servizi.</span><span class="sxs-lookup"><span data-stu-id="eb861-243">The framework doesn't dispose of the services automatically.</span></span>
* <span data-ttu-id="eb861-244">Lo sviluppatore è responsabile dell'eliminazione dei servizi.</span><span class="sxs-lookup"><span data-stu-id="eb861-244">The developer is responsible for disposing the services.</span></span>

### <a name="idisposable-guidance-for-transient-and-shared-instances"></a><span data-ttu-id="eb861-245">Linee guida IDisposable per istanze temporanee e condivise</span><span class="sxs-lookup"><span data-stu-id="eb861-245">IDisposable guidance for Transient and shared instances</span></span>

<span data-ttu-id="eb861-246">Vedere le [linee guida di IDisposable per istanze temporanee e condivise](/dotnet/core/extensions/dependency-injection-guidelines#idisposable-guidance-for-transient-and-shared-instances) nell' [inserimento delle dipendenze in .NET](/dotnet/core/extensions/dependency-injection)</span><span class="sxs-lookup"><span data-stu-id="eb861-246">See [IDisposable guidance for Transient and shared instance](/dotnet/core/extensions/dependency-injection-guidelines#idisposable-guidance-for-transient-and-shared-instances) in [Dependency injection in .NET](/dotnet/core/extensions/dependency-injection)</span></span>

## <a name="default-service-container-replacement"></a><span data-ttu-id="eb861-247">Sostituzione del contenitore di servizi predefinito</span><span class="sxs-lookup"><span data-stu-id="eb861-247">Default service container replacement</span></span>

<span data-ttu-id="eb861-248">Vedere [sostituzione predefinita del contenitore di servizi](/dotnet/core/extensions/dependency-injection-guidelines#default-service-container-replacement) nell' [inserimento delle dipendenze in .NET](/dotnet/core/extensions/dependency-injection)</span><span class="sxs-lookup"><span data-stu-id="eb861-248">See [Default service container replacement](/dotnet/core/extensions/dependency-injection-guidelines#default-service-container-replacement) in [Dependency injection in .NET](/dotnet/core/extensions/dependency-injection)</span></span>

## <a name="recommendations"></a><span data-ttu-id="eb861-249">Consigli</span><span class="sxs-lookup"><span data-stu-id="eb861-249">Recommendations</span></span>

<span data-ttu-id="eb861-250">Vedere le [raccomandazioni](/dotnet/core/extensions/dependency-injection-guidelines#recommendations) nell' [inserimento delle dipendenze in .NET](/dotnet/core/extensions/dependency-injection)</span><span class="sxs-lookup"><span data-stu-id="eb861-250">See [Recommendations](/dotnet/core/extensions/dependency-injection-guidelines#recommendations) in [Dependency injection in .NET](/dotnet/core/extensions/dependency-injection)</span></span>

* <span data-ttu-id="eb861-251">Evitare di usare il *modello di localizzatore del servizio*.</span><span class="sxs-lookup"><span data-stu-id="eb861-251">Avoid using the *service locator pattern*.</span></span> <span data-ttu-id="eb861-252">Ad esempio, non richiamare <xref:System.IServiceProvider.GetService%2A> per ottenere un'istanza del servizio quando è invece possibile usare l'inserimento delle dipendenze:</span><span class="sxs-lookup"><span data-stu-id="eb861-252">For example, don't invoke <xref:System.IServiceProvider.GetService%2A> to obtain a service instance when you can use DI instead:</span></span>

  <span data-ttu-id="eb861-253">**Non corretto:**</span><span class="sxs-lookup"><span data-stu-id="eb861-253">**Incorrect:**</span></span>

    ![Codice errato](dependency-injection/_static/bad.png)

  <span data-ttu-id="eb861-255">**Corretto**:</span><span class="sxs-lookup"><span data-stu-id="eb861-255">**Correct**:</span></span>

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
* <span data-ttu-id="eb861-256">Un'altra variazione del localizzatore del servizio da evitare è inserire una factory che risolve le dipendenze in fase di esecuzione.</span><span class="sxs-lookup"><span data-stu-id="eb861-256">Another service locator variation to avoid is injecting a factory that resolves dependencies at runtime.</span></span> <span data-ttu-id="eb861-257">Queste procedure combinano le strategie di [inversione del controllo](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#dependency-inversion).</span><span class="sxs-lookup"><span data-stu-id="eb861-257">Both of these practices mix [Inversion of Control](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#dependency-inversion) strategies.</span></span>
* <span data-ttu-id="eb861-258">Evitare l'accesso statico a `HttpContext` (ad esempio [IHttpContextAccessor.HttpContext](xref:Microsoft.AspNetCore.Http.IHttpContextAccessor.HttpContext)).</span><span class="sxs-lookup"><span data-stu-id="eb861-258">Avoid static access to `HttpContext` (for example, [IHttpContextAccessor.HttpContext](xref:Microsoft.AspNetCore.Http.IHttpContextAccessor.HttpContext)).</span></span>

<a name="ASP0000"></a>
* <span data-ttu-id="eb861-259">Evitare chiamate a <xref:Microsoft.Extensions.DependencyInjection.ServiceCollectionContainerBuilderExtensions.BuildServiceProvider%2A> in `ConfigureServices` .</span><span class="sxs-lookup"><span data-stu-id="eb861-259">Avoid calls to <xref:Microsoft.Extensions.DependencyInjection.ServiceCollectionContainerBuilderExtensions.BuildServiceProvider%2A> in `ConfigureServices`.</span></span> <span data-ttu-id="eb861-260">`BuildServiceProvider`La chiamata in genere si verifica quando lo sviluppatore desidera risolvere un servizio in `ConfigureServices` .</span><span class="sxs-lookup"><span data-stu-id="eb861-260">Calling `BuildServiceProvider` typically happens when the developer wants to resolve a service in `ConfigureServices`.</span></span> <span data-ttu-id="eb861-261">Si consideri, ad esempio, il caso in cui l'oggetto `LoginPath` viene caricato dalla configurazione.</span><span class="sxs-lookup"><span data-stu-id="eb861-261">For example, consider the case where the `LoginPath` is loaded from configuration.</span></span> <span data-ttu-id="eb861-262">Evitare l'approccio seguente:</span><span class="sxs-lookup"><span data-stu-id="eb861-262">Avoid the following approach:</span></span>

  ![codice errato durante la chiamata di BuildServiceProvider](~/fundamentals/dependency-injection/_static/badcodeX.png)

  <span data-ttu-id="eb861-264">Nell'immagine precedente, selezionando la linea ondulata verde in `services.BuildServiceProvider` viene visualizzato l'avviso ASP0000 seguente:</span><span class="sxs-lookup"><span data-stu-id="eb861-264">In the preceding image, selecting the green wavy line under `services.BuildServiceProvider` shows the following ASP0000 warning:</span></span>

  > <span data-ttu-id="eb861-265">ASP0000 la chiamata a' BuildServiceProvider ' dal codice dell'applicazione comporta una copia aggiuntiva dei servizi singleton creati.</span><span class="sxs-lookup"><span data-stu-id="eb861-265">ASP0000 Calling 'BuildServiceProvider' from application code results in an additional copy of singleton services being created.</span></span> <span data-ttu-id="eb861-266">Prendere in considerazione alternative come la dipendenza che inserisce i servizi come parametri per "Configure".</span><span class="sxs-lookup"><span data-stu-id="eb861-266">Consider alternatives such as dependency injecting services as parameters to 'Configure'.</span></span>

  <span data-ttu-id="eb861-267">`BuildServiceProvider`La chiamata di crea un secondo contenitore, che può creare singleton incompleti e causare riferimenti a oggetti grafici tra più contenitori.</span><span class="sxs-lookup"><span data-stu-id="eb861-267">Calling `BuildServiceProvider` creates a second container, which can create torn singletons and cause references to object graphs across multiple containers.</span></span>

  <span data-ttu-id="eb861-268">Un modo corretto per ottenere `LoginPath` consiste nell'usare il supporto incorporato del modello di opzioni per l'inserimento delle dipendenze:</span><span class="sxs-lookup"><span data-stu-id="eb861-268">A correct way to get `LoginPath` is to use the options pattern's built-in support for DI:</span></span>

  [!code-csharp[](dependency-injection/samples/3.x/AntiPattern3/Startup.cs?name=snippet)]

* <span data-ttu-id="eb861-269">I servizi temporanei Disposable vengono acquisiti dal contenitore per l'eliminazione.</span><span class="sxs-lookup"><span data-stu-id="eb861-269">Disposable transient services are captured by the container for disposal.</span></span> <span data-ttu-id="eb861-270">Questa operazione può comportare una perdita di memoria se risolta dal contenitore di primo livello.</span><span class="sxs-lookup"><span data-stu-id="eb861-270">This can turn into a memory leak if resolved from the top level container.</span></span>
* <span data-ttu-id="eb861-271">Abilitare la convalida dell'ambito per assicurarsi che l'app non disponga di singleton che acquisiscono i servizi con ambito.</span><span class="sxs-lookup"><span data-stu-id="eb861-271">Enable scope validation to make sure the app doesn't have singletons that capture scoped services.</span></span> <span data-ttu-id="eb861-272">Per ulteriori informazioni, vedere [Convalida dell'ambito](#scope-validation).</span><span class="sxs-lookup"><span data-stu-id="eb861-272">For more information, see [Scope validation](#scope-validation).</span></span>

<span data-ttu-id="eb861-273">È tuttavia possibile che in alcuni casi queste raccomandazioni debbano essere ignorate.</span><span class="sxs-lookup"><span data-stu-id="eb861-273">Like all sets of recommendations, you may encounter situations where ignoring a recommendation is required.</span></span> <span data-ttu-id="eb861-274">Le eccezioni sono rari, soprattutto casi speciali all'interno del Framework stesso.</span><span class="sxs-lookup"><span data-stu-id="eb861-274">Exceptions are rare, mostly special cases within the framework itself.</span></span>

<span data-ttu-id="eb861-275">L'inserimento di dipendenze è un'*alternativa* ai modelli di accesso agli oggetti statici/globali.</span><span class="sxs-lookup"><span data-stu-id="eb861-275">DI is an *alternative* to static/global object access patterns.</span></span> <span data-ttu-id="eb861-276">Se l'inserimento di dipendenze viene usato con l'accesso agli oggetti statico i vantaggi non saranno evidenti.</span><span class="sxs-lookup"><span data-stu-id="eb861-276">You may not be able to realize the benefits of DI if you mix it with static object access.</span></span>

## <a name="recommended-patterns-for-multi-tenancy-in-di"></a><span data-ttu-id="eb861-277">Modelli consigliati per il multi-tenant in DI</span><span class="sxs-lookup"><span data-stu-id="eb861-277">Recommended patterns for multi-tenancy in DI</span></span>

<span data-ttu-id="eb861-278">[Orchard Core](https://github.com/OrchardCMS/OrchardCore) è un Framework di applicazione per la creazione di applicazioni modulari multi-tenant in ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="eb861-278">[Orchard Core](https://github.com/OrchardCMS/OrchardCore) is an application framework for building modular, multi-tenant applications on ASP.NET Core.</span></span> <span data-ttu-id="eb861-279">Per ulteriori informazioni, vedere la [documentazione di Orchard Core](https://docs.orchardcore.net/en/dev/).</span><span class="sxs-lookup"><span data-stu-id="eb861-279">For more information, see the [Orchard Core Documentation](https://docs.orchardcore.net/en/dev/).</span></span>

<span data-ttu-id="eb861-280">Vedere gli [esempi di Orchard Core](https://github.com/OrchardCMS/OrchardCore.Samples) per esempi di come creare app modulari e multi-tenant usando solo il Framework Orchard core senza le funzionalità specifiche di CMS.</span><span class="sxs-lookup"><span data-stu-id="eb861-280">See the [Orchard Core samples](https://github.com/OrchardCMS/OrchardCore.Samples) for examples of how to build modular and multi-tenant apps using just the Orchard Core Framework without any of its CMS-specific features.</span></span>

## <a name="framework-provided-services"></a><span data-ttu-id="eb861-281">Servizi forniti dal framework</span><span class="sxs-lookup"><span data-stu-id="eb861-281">Framework-provided services</span></span>

<span data-ttu-id="eb861-282">Il `Startup.ConfigureServices` metodo registra i servizi usati dall'app, incluse le funzionalità della piattaforma, ad esempio Entity Framework Core e ASP.NET Core MVC.</span><span class="sxs-lookup"><span data-stu-id="eb861-282">The `Startup.ConfigureServices` method registers services that the app uses, including platform features, such as Entity Framework Core and ASP.NET Core MVC.</span></span> <span data-ttu-id="eb861-283">Inizialmente, l'oggetto `IServiceCollection` fornito a `ConfigureServices` dispone di servizi definiti dal Framework a seconda della [configurazione dell'host](xref:fundamentals/index#host).</span><span class="sxs-lookup"><span data-stu-id="eb861-283">Initially, the `IServiceCollection` provided to `ConfigureServices` has services defined by the framework depending on [how the host was configured](xref:fundamentals/index#host).</span></span> <span data-ttu-id="eb861-284">Per le app basate sui modelli di ASP.NET Core, il Framework registra più di 250 servizi.</span><span class="sxs-lookup"><span data-stu-id="eb861-284">For apps based on the ASP.NET Core templates, the framework registers more than 250 services.</span></span> 

<span data-ttu-id="eb861-285">La tabella seguente elenca un piccolo esempio di questi servizi registrati dal Framework:</span><span class="sxs-lookup"><span data-stu-id="eb861-285">The following table lists a small sample of these framework-registered services:</span></span>

| <span data-ttu-id="eb861-286">Tipo di servizio</span><span class="sxs-lookup"><span data-stu-id="eb861-286">Service Type</span></span>                                                                                    | <span data-ttu-id="eb861-287">Durata</span><span class="sxs-lookup"><span data-stu-id="eb861-287">Lifetime</span></span>  |
|-------------------------------------------------------------------------------------------------|-----------|
| <xref:Microsoft.AspNetCore.Hosting.Builder.IApplicationBuilderFactory?displayProperty=fullName> | <span data-ttu-id="eb861-288">Temporaneo</span><span class="sxs-lookup"><span data-stu-id="eb861-288">Transient</span></span> |
| <xref:Microsoft.Extensions.Hosting.IHostApplicationLifetime>                                    | <span data-ttu-id="eb861-289">Singleton</span><span class="sxs-lookup"><span data-stu-id="eb861-289">Singleton</span></span> |
| <xref:Microsoft.AspNetCore.Hosting.IWebHostEnvironment>                                         | <span data-ttu-id="eb861-290">Singleton</span><span class="sxs-lookup"><span data-stu-id="eb861-290">Singleton</span></span> |
| <xref:Microsoft.AspNetCore.Hosting.IStartup?displayProperty=fullName>                           | <span data-ttu-id="eb861-291">Singleton</span><span class="sxs-lookup"><span data-stu-id="eb861-291">Singleton</span></span> |
| <xref:Microsoft.AspNetCore.Hosting.IStartupFilter?displayProperty=fullName>                     | <span data-ttu-id="eb861-292">Temporaneo</span><span class="sxs-lookup"><span data-stu-id="eb861-292">Transient</span></span> |
| <xref:Microsoft.AspNetCore.Hosting.Server.IServer?displayProperty=fullName>                     | <span data-ttu-id="eb861-293">Singleton</span><span class="sxs-lookup"><span data-stu-id="eb861-293">Singleton</span></span> |
| <xref:Microsoft.AspNetCore.Http.IHttpContextFactory?displayProperty=fullName>                   | <span data-ttu-id="eb861-294">Temporaneo</span><span class="sxs-lookup"><span data-stu-id="eb861-294">Transient</span></span> |
| <xref:Microsoft.Extensions.Logging.ILogger%601?displayProperty=fullName>                        | <span data-ttu-id="eb861-295">Singleton</span><span class="sxs-lookup"><span data-stu-id="eb861-295">Singleton</span></span> |
| <xref:Microsoft.Extensions.Logging.ILoggerFactory?displayProperty=fullName>                     | <span data-ttu-id="eb861-296">Singleton</span><span class="sxs-lookup"><span data-stu-id="eb861-296">Singleton</span></span> |
| <xref:Microsoft.Extensions.ObjectPool.ObjectPoolProvider?displayProperty=fullName>              | <span data-ttu-id="eb861-297">Singleton</span><span class="sxs-lookup"><span data-stu-id="eb861-297">Singleton</span></span> |
| <xref:Microsoft.Extensions.Options.IConfigureOptions%601?displayProperty=fullName>              | <span data-ttu-id="eb861-298">Temporaneo</span><span class="sxs-lookup"><span data-stu-id="eb861-298">Transient</span></span> |
| <xref:Microsoft.Extensions.Options.IOptions%601?displayProperty=fullName>                       | <span data-ttu-id="eb861-299">Singleton</span><span class="sxs-lookup"><span data-stu-id="eb861-299">Singleton</span></span> |
| <xref:System.Diagnostics.DiagnosticSource?displayProperty=fullName>                             | <span data-ttu-id="eb861-300">Singleton</span><span class="sxs-lookup"><span data-stu-id="eb861-300">Singleton</span></span> |
| <xref:System.Diagnostics.DiagnosticListener?displayProperty=fullName>                           | <span data-ttu-id="eb861-301">Singleton</span><span class="sxs-lookup"><span data-stu-id="eb861-301">Singleton</span></span> |

## <a name="additional-resources"></a><span data-ttu-id="eb861-302">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="eb861-302">Additional resources</span></span>

* <xref:mvc/views/dependency-injection>
* <xref:mvc/controllers/dependency-injection>
* <xref:security/authorization/dependencyinjection>
* <xref:blazor/fundamentals/dependency-injection>
* [<span data-ttu-id="eb861-303">Modelli DI conferenza NDC per lo sviluppo DI app DI</span><span class="sxs-lookup"><span data-stu-id="eb861-303">NDC Conference Patterns for DI app development</span></span>](https://www.youtube.com/watch?v=x-C-CNBVTaY)
* <xref:fundamentals/startup>
* <xref:fundamentals/middleware/extensibility>
* [<span data-ttu-id="eb861-304">Quattro modi per eliminare IDisposable in ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="eb861-304">Four ways to dispose IDisposables in ASP.NET Core</span></span>](https://andrewlock.net/four-ways-to-dispose-idisposables-in-asp-net-core/)
* [<span data-ttu-id="eb861-305">Scrittura di codice pulito in ASP.NET Core con inserimento delle dipendenze (MSDN)</span><span class="sxs-lookup"><span data-stu-id="eb861-305">Writing Clean Code in ASP.NET Core with Dependency Injection (MSDN)</span></span>](/archive/msdn-magazine/2016/may/asp-net-writing-clean-code-in-asp-net-core-with-dependency-injection)
* [<span data-ttu-id="eb861-306">Explicit Dependencies Principle (Principio delle dipendenze esplicite)</span><span class="sxs-lookup"><span data-stu-id="eb861-306">Explicit Dependencies Principle</span></span>](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#explicit-dependencies)
* [<span data-ttu-id="eb861-307">Inversion of Control Containers and the Dependency Injection Pattern (Martin Fowler) (Contenitori di inversione del controllo e modello di inserimento delle dipendenze)</span><span class="sxs-lookup"><span data-stu-id="eb861-307">Inversion of Control Containers and the Dependency Injection Pattern (Martin Fowler)</span></span>](https://www.martinfowler.com/articles/injection.html)
* <span data-ttu-id="eb861-308">[How to register a service with multiple interfaces in ASP.NET Core DI](https://andrewlock.net/how-to-register-a-service-with-multiple-interfaces-for-in-asp-net-core-di/) (Come registrare un servizio con più interfacce in ASP.NET Core DI)</span><span class="sxs-lookup"><span data-stu-id="eb861-308">[How to register a service with multiple interfaces in ASP.NET Core DI](https://andrewlock.net/how-to-register-a-service-with-multiple-interfaces-for-in-asp-net-core-di/)</span></span>

::: moniker-end

::: moniker range="< aspnetcore-3.0"

<span data-ttu-id="eb861-309">Di [Steve Smith](https://ardalis.com/), [Scott Addie](https://scottaddie.com)e [Brandon Dahler](https://github.com/brandondahler)</span><span class="sxs-lookup"><span data-stu-id="eb861-309">By [Steve Smith](https://ardalis.com/), [Scott Addie](https://scottaddie.com), and [Brandon Dahler](https://github.com/brandondahler)</span></span>

<span data-ttu-id="eb861-310">ASP.NET Core supporta lo schema progettuale di software per l'inserimento di dipendenze, una tecnica per ottenere l'[IoC (Inversion of Control)](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#dependency-inversion) tra le classi e le relative dipendenze.</span><span class="sxs-lookup"><span data-stu-id="eb861-310">ASP.NET Core supports the dependency injection (DI) software design pattern, which is a technique for achieving [Inversion of Control (IoC)](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#dependency-inversion) between classes and their dependencies.</span></span>

<span data-ttu-id="eb861-311">Per altre informazioni specifiche per l'inserimento delle dipendenze all'interno di controller MVC, vedere <xref:mvc/controllers/dependency-injection>.</span><span class="sxs-lookup"><span data-stu-id="eb861-311">For more information specific to dependency injection within MVC controllers, see <xref:mvc/controllers/dependency-injection>.</span></span>

<span data-ttu-id="eb861-312">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/dependency-injection/samples) ([procedura per il download](xref:index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="eb861-312">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/dependency-injection/samples) ([how to download](xref:index#how-to-download-a-sample))</span></span>

## <a name="overview-of-dependency-injection"></a><span data-ttu-id="eb861-313">Panoramica dell'inserimento delle dipendenze</span><span class="sxs-lookup"><span data-stu-id="eb861-313">Overview of dependency injection</span></span>

<span data-ttu-id="eb861-314">Una *dipendenza* è qualsiasi oggetto richiesto da un altro oggetto.</span><span class="sxs-lookup"><span data-stu-id="eb861-314">A *dependency* is any object that another object requires.</span></span> <span data-ttu-id="eb861-315">Esaminare la classe `MyDependency` seguente con un metodo `WriteMessage` da cui dipendono altre classi in un'app:</span><span class="sxs-lookup"><span data-stu-id="eb861-315">Examine the following `MyDependency` class with a `WriteMessage` method that other classes in an app depend upon:</span></span>

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

<span data-ttu-id="eb861-316">È possibile creare un'istanza della classe `MyDependency` per rendere il metodo `WriteMessage`disponibile per una classe.</span><span class="sxs-lookup"><span data-stu-id="eb861-316">An instance of the `MyDependency` class can be created to make the `WriteMessage` method available to a class.</span></span> <span data-ttu-id="eb861-317">La classe `MyDependency` è una dipendenza della classe `IndexModel`:</span><span class="sxs-lookup"><span data-stu-id="eb861-317">The `MyDependency` class is a dependency of the `IndexModel` class:</span></span>

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

<span data-ttu-id="eb861-318">La classe crea e dipende direttamente dall'istanza `MyDependency`.</span><span class="sxs-lookup"><span data-stu-id="eb861-318">The class creates and directly depends on the `MyDependency` instance.</span></span> <span data-ttu-id="eb861-319">Le dipendenze del codice (come nell'esempio precedente) sono problematiche e devono essere evitate per i motivi seguenti:</span><span class="sxs-lookup"><span data-stu-id="eb861-319">Code dependencies (such as the previous example) are problematic and should be avoided for the following reasons:</span></span>

* <span data-ttu-id="eb861-320">Per sostituire `MyDependency` con un'implementazione diversa, la classe deve essere modificata.</span><span class="sxs-lookup"><span data-stu-id="eb861-320">To replace `MyDependency` with a different implementation, the class must be modified.</span></span>
* <span data-ttu-id="eb861-321">Se `MyDependency` ha dipendenze, devono essere configurate dalla classe.</span><span class="sxs-lookup"><span data-stu-id="eb861-321">If `MyDependency` has dependencies, they must be configured by the class.</span></span> <span data-ttu-id="eb861-322">In un progetto di grandi dimensioni con più classi che dipendono da `MyDependency`, il codice di configurazione diventa sparso in tutta l'app.</span><span class="sxs-lookup"><span data-stu-id="eb861-322">In a large project with multiple classes depending on `MyDependency`, the configuration code becomes scattered across the app.</span></span>
* <span data-ttu-id="eb861-323">È difficile eseguire unit test di questa implementazione.</span><span class="sxs-lookup"><span data-stu-id="eb861-323">This implementation is difficult to unit test.</span></span> <span data-ttu-id="eb861-324">L'app dovrebbe usare una classe `MyDependency` fittizia o stub, ma ciò non è possibile con questo approccio.</span><span class="sxs-lookup"><span data-stu-id="eb861-324">The app should use a mock or stub `MyDependency` class, which isn't possible with this approach.</span></span>

<span data-ttu-id="eb861-325">L'inserimento delle dipendenze consente di risolvere questi problemi tramite:</span><span class="sxs-lookup"><span data-stu-id="eb861-325">Dependency injection addresses these problems through:</span></span>

* <span data-ttu-id="eb861-326">L'uso di un'interfaccia o di una classe di base per astrarre l'implementazione delle dipendenze.</span><span class="sxs-lookup"><span data-stu-id="eb861-326">The use of an interface or base class to abstract the dependency implementation.</span></span>
* <span data-ttu-id="eb861-327">La registrazione della dipendenza in un contenitore di servizi.</span><span class="sxs-lookup"><span data-stu-id="eb861-327">Registration of the dependency in a service container.</span></span> <span data-ttu-id="eb861-328">ASP.NET Core offre il contenitore di servizi predefinito <xref:System.IServiceProvider>.</span><span class="sxs-lookup"><span data-stu-id="eb861-328">ASP.NET Core provides a built-in service container, <xref:System.IServiceProvider>.</span></span> <span data-ttu-id="eb861-329">I servizi vengono registrati nel metodo `Startup.ConfigureServices` dell'app.</span><span class="sxs-lookup"><span data-stu-id="eb861-329">Services are registered in the app's `Startup.ConfigureServices` method.</span></span>
* <span data-ttu-id="eb861-330">L'*inserimento* del servizio nel costruttore della classe in cui viene usato.</span><span class="sxs-lookup"><span data-stu-id="eb861-330">*Injection* of the service into the constructor of the class where it's used.</span></span> <span data-ttu-id="eb861-331">Il framework si assume la responsabilità della creazione di un'istanza della dipendenza e della sua eliminazione quando non è più necessaria.</span><span class="sxs-lookup"><span data-stu-id="eb861-331">The framework takes on the responsibility of creating an instance of the dependency and disposing of it when it's no longer needed.</span></span>

<span data-ttu-id="eb861-332">Nell'[app di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/dependency-injection/samples), l'interfaccia `IMyDependency` definisce un metodo che il servizio fornisce all'app:</span><span class="sxs-lookup"><span data-stu-id="eb861-332">In the [sample app](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/dependency-injection/samples), the `IMyDependency` interface defines a method that the service provides to the app:</span></span>

[!code-csharp[](dependency-injection/samples/2.x/DependencyInjectionSample/Interfaces/IMyDependency.cs?name=snippet1)]

<span data-ttu-id="eb861-333">Questa interfaccia viene implementata da un tipo concreto, `MyDependency`:</span><span class="sxs-lookup"><span data-stu-id="eb861-333">This interface is implemented by a concrete type, `MyDependency`:</span></span>

[!code-csharp[](dependency-injection/samples/2.x/DependencyInjectionSample/Services/MyDependency.cs?name=snippet1)]

<span data-ttu-id="eb861-334">`MyDependency` richiede <xref:Microsoft.Extensions.Logging.ILogger`1> nel costruttore.</span><span class="sxs-lookup"><span data-stu-id="eb861-334">`MyDependency` requests an <xref:Microsoft.Extensions.Logging.ILogger`1> in its constructor.</span></span> <span data-ttu-id="eb861-335">Non è insolito usare l'inserimento delle dipendenze in modo concatenato.</span><span class="sxs-lookup"><span data-stu-id="eb861-335">It's not unusual to use dependency injection in a chained fashion.</span></span> <span data-ttu-id="eb861-336">Ogni dipendenza richiesta richiede a sua volta le proprie dipendenze.</span><span class="sxs-lookup"><span data-stu-id="eb861-336">Each requested dependency in turn requests its own dependencies.</span></span> <span data-ttu-id="eb861-337">Il contenitore risolve le dipendenze nel grafico e restituisce il servizio completamente risolto.</span><span class="sxs-lookup"><span data-stu-id="eb861-337">The container resolves the dependencies in the graph and returns the fully resolved service.</span></span> <span data-ttu-id="eb861-338">Il set di dipendenze che devono essere risolte viene generalmente chiamato *albero delle dipendenze* o *grafico dipendenze* o *grafico degli oggetti*.</span><span class="sxs-lookup"><span data-stu-id="eb861-338">The collective set of dependencies that must be resolved is typically referred to as a *dependency tree*, *dependency graph*, or *object graph*.</span></span>

<span data-ttu-id="eb861-339">`IMyDependency` e `ILogger<TCategoryName>` devono essere registrati nel contenitore di servizi.</span><span class="sxs-lookup"><span data-stu-id="eb861-339">`IMyDependency` and `ILogger<TCategoryName>` must be registered in the service container.</span></span> <span data-ttu-id="eb861-340">`IMyDependency` viene registrato in `Startup.ConfigureServices`.</span><span class="sxs-lookup"><span data-stu-id="eb861-340">`IMyDependency` is registered in `Startup.ConfigureServices`.</span></span> <span data-ttu-id="eb861-341">`ILogger<TCategoryName>` viene registrato dall'infrastruttura di registrazione delle astrazioni, pertanto si tratta di un [servizio fornito dal Framework](#framework-provided-services) registrato per impostazione predefinita dal Framework.</span><span class="sxs-lookup"><span data-stu-id="eb861-341">`ILogger<TCategoryName>` is registered by the logging abstractions infrastructure, so it's a [framework-provided service](#framework-provided-services) registered by default by the framework.</span></span>

<span data-ttu-id="eb861-342">Il contenitore risolve `ILogger<TCategoryName>` avvalendosi di [tipi aperti (generici)](/dotnet/csharp/language-reference/language-specification/types#open-and-closed-types), eliminando l'esigenza di registrare ogni singolo [tipo costruito (generico)](/dotnet/csharp/language-reference/language-specification/types#constructed-types):</span><span class="sxs-lookup"><span data-stu-id="eb861-342">The container resolves `ILogger<TCategoryName>` by taking advantage of [(generic) open types](/dotnet/csharp/language-reference/language-specification/types#open-and-closed-types), eliminating the need to register every [(generic) constructed type](/dotnet/csharp/language-reference/language-specification/types#constructed-types):</span></span>

```csharp
services.AddSingleton(typeof(ILogger<>), typeof(Logger<>));
```

<span data-ttu-id="eb861-343">Nell'app di esempio, il servizio `IMyDependency` viene registrato con il tipo concreto `MyDependency`.</span><span class="sxs-lookup"><span data-stu-id="eb861-343">In the sample app, the `IMyDependency` service is registered with the concrete type `MyDependency`.</span></span> <span data-ttu-id="eb861-344">La registrazione definisce come ambito della durata di servizio la durata di una singola richiesta.</span><span class="sxs-lookup"><span data-stu-id="eb861-344">The registration scopes the service lifetime to the lifetime of a single request.</span></span> <span data-ttu-id="eb861-345">Le [durate dei servizi](#service-lifetimes) sono descritte più avanti in questo argomento.</span><span class="sxs-lookup"><span data-stu-id="eb861-345">[Service lifetimes](#service-lifetimes) are described later in this topic.</span></span>

[!code-csharp[](dependency-injection/samples/2.x/DependencyInjectionSample/Startup.cs?name=snippet1&highlight=5)]

> [!NOTE]
> <span data-ttu-id="eb861-346">Ogni `services.Add{SERVICE_NAME}` metodo di estensione aggiunge e potenzialmente configura i servizi.</span><span class="sxs-lookup"><span data-stu-id="eb861-346">Each `services.Add{SERVICE_NAME}` extension method adds, and potentially configures, services.</span></span> <span data-ttu-id="eb861-347">Ad esempio, `services.AddControllersWithViews` , `services.AddRazorPages` e `services.AddControllers` aggiunge i servizi ASP.NET Core le app richiedono.</span><span class="sxs-lookup"><span data-stu-id="eb861-347">For example, `services.AddControllersWithViews`, `services.AddRazorPages`, and `services.AddControllers` adds the services ASP.NET Core apps require.</span></span> <span data-ttu-id="eb861-348">È consigliabile che le app seguano questa convenzione.</span><span class="sxs-lookup"><span data-stu-id="eb861-348">We recommended that apps follow this convention.</span></span> <span data-ttu-id="eb861-349">Inserire i metodi di estensione nello spazio dei nomi <xref:Microsoft.Extensions.DependencyInjection?displayProperty=fullName> per incapsulare i gruppi di registrazioni di servizio.</span><span class="sxs-lookup"><span data-stu-id="eb861-349">Place extension methods in the <xref:Microsoft.Extensions.DependencyInjection?displayProperty=fullName> namespace to encapsulate groups of service registrations.</span></span> <span data-ttu-id="eb861-350">Inclusa anche la parte relativa allo spazio dei nomi `Microsoft.Extensions.DependencyInjection` per i metodi di estensione di.</span><span class="sxs-lookup"><span data-stu-id="eb861-350">Including the namespace portion `Microsoft.Extensions.DependencyInjection` for DI extension methods also:</span></span>
>
> * <span data-ttu-id="eb861-351">Consente di visualizzarli in [IntelliSense](/visualstudio/ide/using-intellisense) senza aggiungere ulteriori `using` blocchi.</span><span class="sxs-lookup"><span data-stu-id="eb861-351">Allows them to be displayed in [IntelliSense](/visualstudio/ide/using-intellisense) without adding additional `using` blocks.</span></span>
> * <span data-ttu-id="eb861-352">Impedisce `using` l'utilizzo di istruzioni eccessive nella `Startup` classe in cui i metodi di estensione vengono in genere chiamati da.</span><span class="sxs-lookup"><span data-stu-id="eb861-352">Prevents excessive `using` statements in the `Startup` class where these extension methods are typically called from.</span></span>

<span data-ttu-id="eb861-353">Se il costruttore del servizio richiede un [tipo incorporato](/dotnet/csharp/language-reference/keywords/built-in-types-table), ad esempio `string`, è possibile inserirlo usando la [configurazione](xref:fundamentals/configuration/index) o il [modello di opzioni](xref:fundamentals/configuration/options):</span><span class="sxs-lookup"><span data-stu-id="eb861-353">If the service's constructor requires a [built in type](/dotnet/csharp/language-reference/keywords/built-in-types-table), such as a `string`, the type can be injected by using [configuration](xref:fundamentals/configuration/index) or the [options pattern](xref:fundamentals/configuration/options):</span></span>

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

<span data-ttu-id="eb861-354">È richiesta un'istanza del servizio tramite il costruttore di una classe in cui il servizio viene usato e assegnato a un campo privato.</span><span class="sxs-lookup"><span data-stu-id="eb861-354">An instance of the service is requested via the constructor of a class where the service is used and assigned to a private field.</span></span> <span data-ttu-id="eb861-355">Il campo viene usato per accedere al servizio in base alle esigenze per l'intera classe.</span><span class="sxs-lookup"><span data-stu-id="eb861-355">The field is used to access the service as necessary throughout the class.</span></span>

<span data-ttu-id="eb861-356">Nell'app di esempio, viene richiesta l'istanza `IMyDependency` usata per chiamare il metodo `WriteMessage` del servizio:</span><span class="sxs-lookup"><span data-stu-id="eb861-356">In the sample app, the `IMyDependency` instance is requested and used to call the service's `WriteMessage` method:</span></span>

[!code-csharp[](dependency-injection/samples/2.x/DependencyInjectionSample/Pages/Index.cshtml.cs?name=snippet1&highlight=3,6,13,29-30)]

## <a name="services-injected-into-startup"></a><span data-ttu-id="eb861-357">Servizi inseriti all'avvio</span><span class="sxs-lookup"><span data-stu-id="eb861-357">Services injected into Startup</span></span>

<span data-ttu-id="eb861-358">`Startup`Quando si utilizza l'host generico () è possibile inserire nel costruttore solo i tipi di servizio seguenti <xref:Microsoft.Extensions.Hosting.IHostBuilder> :</span><span class="sxs-lookup"><span data-stu-id="eb861-358">Only the following service types can be injected into the `Startup` constructor when using the Generic Host (<xref:Microsoft.Extensions.Hosting.IHostBuilder>):</span></span>

* `IWebHostEnvironment`
* <xref:Microsoft.Extensions.Hosting.IHostEnvironment>
* <xref:Microsoft.Extensions.Configuration.IConfiguration>

<span data-ttu-id="eb861-359">I servizi possono essere inseriti in `Startup.Configure` :</span><span class="sxs-lookup"><span data-stu-id="eb861-359">Services can be injected into `Startup.Configure`:</span></span>

```csharp
public void Configure(IApplicationBuilder app, IOptions<MyOptions> options)
{
    ...
}
```

<span data-ttu-id="eb861-360">Per altre informazioni, vedere <xref:fundamentals/startup>.</span><span class="sxs-lookup"><span data-stu-id="eb861-360">For more information, see <xref:fundamentals/startup>.</span></span>

## <a name="framework-provided-services"></a><span data-ttu-id="eb861-361">Servizi forniti dal framework</span><span class="sxs-lookup"><span data-stu-id="eb861-361">Framework-provided services</span></span>

<span data-ttu-id="eb861-362">Il `Startup.ConfigureServices` metodo è responsabile della definizione dei servizi utilizzati dall'app, incluse le funzionalità della piattaforma, ad esempio Entity Framework Core e ASP.NET Core MVC.</span><span class="sxs-lookup"><span data-stu-id="eb861-362">The `Startup.ConfigureServices` method is responsible for defining the services that the app uses, including platform features, such as Entity Framework Core and ASP.NET Core MVC.</span></span> <span data-ttu-id="eb861-363">Inizialmente, l'oggetto `IServiceCollection` fornito a `ConfigureServices` dispone di servizi definiti dal Framework a seconda della [configurazione dell'host](xref:fundamentals/index#host).</span><span class="sxs-lookup"><span data-stu-id="eb861-363">Initially, the `IServiceCollection` provided to `ConfigureServices` has services defined by the framework depending on [how the host was configured](xref:fundamentals/index#host).</span></span> <span data-ttu-id="eb861-364">Non è insolito che un'app basata su un modello di ASP.NET Core disponga di centinaia di servizi registrati dal Framework.</span><span class="sxs-lookup"><span data-stu-id="eb861-364">It's not uncommon for an app based on an ASP.NET Core template to have hundreds of services registered by the framework.</span></span> <span data-ttu-id="eb861-365">Nella tabella seguente è elencato un piccolo esempio di servizi registrati dal Framework.</span><span class="sxs-lookup"><span data-stu-id="eb861-365">A small sample of framework-registered services is listed in the following table.</span></span>

| <span data-ttu-id="eb861-366">Tipo di servizio</span><span class="sxs-lookup"><span data-stu-id="eb861-366">Service Type</span></span> | <span data-ttu-id="eb861-367">Durata</span><span class="sxs-lookup"><span data-stu-id="eb861-367">Lifetime</span></span> |
| ------------ | -------- |
| <xref:Microsoft.AspNetCore.Hosting.Builder.IApplicationBuilderFactory?displayProperty=fullName> | <span data-ttu-id="eb861-368">Temporaneo</span><span class="sxs-lookup"><span data-stu-id="eb861-368">Transient</span></span> |
| <xref:Microsoft.AspNetCore.Hosting.IApplicationLifetime?displayProperty=fullName> | <span data-ttu-id="eb861-369">Singleton</span><span class="sxs-lookup"><span data-stu-id="eb861-369">Singleton</span></span> |
| <xref:Microsoft.AspNetCore.Hosting.IHostingEnvironment?displayProperty=fullName> | <span data-ttu-id="eb861-370">Singleton</span><span class="sxs-lookup"><span data-stu-id="eb861-370">Singleton</span></span> |
| <xref:Microsoft.AspNetCore.Hosting.IStartup?displayProperty=fullName> | <span data-ttu-id="eb861-371">Singleton</span><span class="sxs-lookup"><span data-stu-id="eb861-371">Singleton</span></span> |
| <xref:Microsoft.AspNetCore.Hosting.IStartupFilter?displayProperty=fullName> | <span data-ttu-id="eb861-372">Temporaneo</span><span class="sxs-lookup"><span data-stu-id="eb861-372">Transient</span></span> |
| <xref:Microsoft.AspNetCore.Hosting.Server.IServer?displayProperty=fullName> | <span data-ttu-id="eb861-373">Singleton</span><span class="sxs-lookup"><span data-stu-id="eb861-373">Singleton</span></span> |
| <xref:Microsoft.AspNetCore.Http.IHttpContextFactory?displayProperty=fullName> | <span data-ttu-id="eb861-374">Temporaneo</span><span class="sxs-lookup"><span data-stu-id="eb861-374">Transient</span></span> |
| <xref:Microsoft.Extensions.Logging.ILogger`1?displayProperty=fullName> | <span data-ttu-id="eb861-375">Singleton</span><span class="sxs-lookup"><span data-stu-id="eb861-375">Singleton</span></span> |
| <xref:Microsoft.Extensions.Logging.ILoggerFactory?displayProperty=fullName> | <span data-ttu-id="eb861-376">Singleton</span><span class="sxs-lookup"><span data-stu-id="eb861-376">Singleton</span></span> |
| <xref:Microsoft.Extensions.ObjectPool.ObjectPoolProvider?displayProperty=fullName> | <span data-ttu-id="eb861-377">Singleton</span><span class="sxs-lookup"><span data-stu-id="eb861-377">Singleton</span></span> |
| <xref:Microsoft.Extensions.Options.IConfigureOptions`1?displayProperty=fullName> | <span data-ttu-id="eb861-378">Temporaneo</span><span class="sxs-lookup"><span data-stu-id="eb861-378">Transient</span></span> |
| <xref:Microsoft.Extensions.Options.IOptions`1?displayProperty=fullName> | <span data-ttu-id="eb861-379">Singleton</span><span class="sxs-lookup"><span data-stu-id="eb861-379">Singleton</span></span> |
| <xref:System.Diagnostics.DiagnosticSource?displayProperty=fullName> | <span data-ttu-id="eb861-380">Singleton</span><span class="sxs-lookup"><span data-stu-id="eb861-380">Singleton</span></span> |
| <xref:System.Diagnostics.DiagnosticListener?displayProperty=fullName> | <span data-ttu-id="eb861-381">Singleton</span><span class="sxs-lookup"><span data-stu-id="eb861-381">Singleton</span></span> |

## <a name="register-additional-services-with-extension-methods"></a><span data-ttu-id="eb861-382">Registrare servizi aggiuntivi con i metodi di estensione</span><span class="sxs-lookup"><span data-stu-id="eb861-382">Register additional services with extension methods</span></span>

<span data-ttu-id="eb861-383">Quando è disponibile un metodo di estensione della raccolta di servizi per la registrazione di un servizio (e dei servizi dipendenti, se necessario), la convenzione consiste nell'usare un singolo metodo di estensione `Add{SERVICE_NAME}` per registrare tutti i servizi richiesti da tale servizio.</span><span class="sxs-lookup"><span data-stu-id="eb861-383">When a service collection extension method is available to register a service (and its dependent services, if required), the convention is to use a single `Add{SERVICE_NAME}` extension method to register all of the services required by that service.</span></span> <span data-ttu-id="eb861-384">Il codice seguente è un esempio di come aggiungere servizi aggiuntivi al contenitore usando i metodi di estensione [AddDbContext \<TContext> ](/dotnet/api/microsoft.extensions.dependencyinjection.entityframeworkservicecollectionextensions.adddbcontext) e <xref:Microsoft.Extensions.DependencyInjection.IdentityServiceCollectionExtensions.AddIdentityCore*> :</span><span class="sxs-lookup"><span data-stu-id="eb861-384">The following code is an example of how to add additional services to the container using the extension methods [AddDbContext\<TContext>](/dotnet/api/microsoft.extensions.dependencyinjection.entityframeworkservicecollectionextensions.adddbcontext) and <xref:Microsoft.Extensions.DependencyInjection.IdentityServiceCollectionExtensions.AddIdentityCore*>:</span></span>

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

<span data-ttu-id="eb861-385">Per altre informazioni, vedere la classe <xref:Microsoft.Extensions.DependencyInjection.ServiceCollection> nella documentazione delle API.</span><span class="sxs-lookup"><span data-stu-id="eb861-385">For more information, see the <xref:Microsoft.Extensions.DependencyInjection.ServiceCollection> class in the API documentation.</span></span>

## <a name="service-lifetimes"></a><span data-ttu-id="eb861-386">Durate del servizio</span><span class="sxs-lookup"><span data-stu-id="eb861-386">Service lifetimes</span></span>

<span data-ttu-id="eb861-387">Scegliere una durata appropriata per ogni servizio registrato.</span><span class="sxs-lookup"><span data-stu-id="eb861-387">Choose an appropriate lifetime for each registered service.</span></span> <span data-ttu-id="eb861-388">I servizi ASP.NET Core possono essere configurati con le impostazioni di durata seguenti:</span><span class="sxs-lookup"><span data-stu-id="eb861-388">ASP.NET Core services can be configured with the following lifetimes:</span></span>

### <a name="transient"></a><span data-ttu-id="eb861-389">Temporaneo</span><span class="sxs-lookup"><span data-stu-id="eb861-389">Transient</span></span>

<span data-ttu-id="eb861-390">I servizi con durata temporanea (<xref:Microsoft.Extensions.DependencyInjection.ServiceCollectionServiceExtensions.AddTransient*>) vengono creati ogni volta che vengono richiesti dal contenitore dei servizi.</span><span class="sxs-lookup"><span data-stu-id="eb861-390">Transient lifetime services (<xref:Microsoft.Extensions.DependencyInjection.ServiceCollectionServiceExtensions.AddTransient*>) are created each time they're requested from the service container.</span></span> <span data-ttu-id="eb861-391">Questa impostazione di durata è consigliata per i servizi semplici senza stati.</span><span class="sxs-lookup"><span data-stu-id="eb861-391">This lifetime works best for lightweight, stateless services.</span></span>

<span data-ttu-id="eb861-392">Nelle app che elaborano le richieste, i servizi temporanei vengono eliminati alla fine della richiesta.</span><span class="sxs-lookup"><span data-stu-id="eb861-392">In apps that process requests, transient services are disposed at the end of the request.</span></span>

### <a name="scoped"></a><span data-ttu-id="eb861-393">Con ambito</span><span class="sxs-lookup"><span data-stu-id="eb861-393">Scoped</span></span>

<span data-ttu-id="eb861-394">I servizi con durata con ambito (<xref:Microsoft.Extensions.DependencyInjection.ServiceCollectionServiceExtensions.AddScoped*>) vengono creati una volta per ogni richiesta client (connessione).</span><span class="sxs-lookup"><span data-stu-id="eb861-394">Scoped lifetime services (<xref:Microsoft.Extensions.DependencyInjection.ServiceCollectionServiceExtensions.AddScoped*>) are created once per client request (connection).</span></span>

<span data-ttu-id="eb861-395">Nelle app che elaborano le richieste, i servizi con ambito vengono eliminati alla fine della richiesta.</span><span class="sxs-lookup"><span data-stu-id="eb861-395">In apps that process requests, scoped services are disposed at the end of the request.</span></span>

> [!WARNING]
> <span data-ttu-id="eb861-396">Quando si usa un servizio con ambito in un middleware, inserire il servizio nel metodo `Invoke` o `InvokeAsync`.</span><span class="sxs-lookup"><span data-stu-id="eb861-396">When using a scoped service in a middleware, inject the service into the `Invoke` or `InvokeAsync` method.</span></span> <span data-ttu-id="eb861-397">Non inserire tramite il [Costruttore Injection](xref:mvc/controllers/dependency-injection#constructor-injection) perché impone al servizio di comportarsi come un singleton.</span><span class="sxs-lookup"><span data-stu-id="eb861-397">Don't inject via [constructor injection](xref:mvc/controllers/dependency-injection#constructor-injection) because it forces the service to behave like a singleton.</span></span> <span data-ttu-id="eb861-398">Per altre informazioni, vedere <xref:fundamentals/middleware/write#per-request-middleware-dependencies>.</span><span class="sxs-lookup"><span data-stu-id="eb861-398">For more information, see <xref:fundamentals/middleware/write#per-request-middleware-dependencies>.</span></span>

### <a name="singleton"></a><span data-ttu-id="eb861-399">Singleton</span><span class="sxs-lookup"><span data-stu-id="eb861-399">Singleton</span></span>

<span data-ttu-id="eb861-400">I servizi con durata singleton (<xref:Microsoft.Extensions.DependencyInjection.ServiceCollectionServiceExtensions.AddSingleton*>) vengono creati la prima volta che vengono richiesti (o quando si esegue `Startup.ConfigureServices` e viene specificata un'istanza con la registrazione del servizio).</span><span class="sxs-lookup"><span data-stu-id="eb861-400">Singleton lifetime services (<xref:Microsoft.Extensions.DependencyInjection.ServiceCollectionServiceExtensions.AddSingleton*>) are created the first time they're requested (or when `Startup.ConfigureServices` is run and an instance is specified with the service registration).</span></span> <span data-ttu-id="eb861-401">Tutte le richieste successive usano la stessa istanza.</span><span class="sxs-lookup"><span data-stu-id="eb861-401">Every subsequent request uses the same instance.</span></span> <span data-ttu-id="eb861-402">Se l'app richiede un comportamento singleton, è consigliabile consentire al contenitore del servizio di gestire la durata del servizio.</span><span class="sxs-lookup"><span data-stu-id="eb861-402">If the app requires singleton behavior, allowing the service container to manage the service's lifetime is recommended.</span></span> <span data-ttu-id="eb861-403">Non implementare lo schema progettuale singleton e fornire codice utente per gestire la durata dell'oggetto nella classe.</span><span class="sxs-lookup"><span data-stu-id="eb861-403">Don't implement the singleton design pattern and provide user code to manage the object's lifetime in the class.</span></span>

<span data-ttu-id="eb861-404">Nelle app che elaborano le richieste, i servizi Singleton vengono eliminati quando il <xref:Microsoft.Extensions.DependencyInjection.ServiceProvider> viene eliminato alla chiusura dell'app.</span><span class="sxs-lookup"><span data-stu-id="eb861-404">In apps that process requests, singleton services are disposed when the <xref:Microsoft.Extensions.DependencyInjection.ServiceProvider> is disposed at app shutdown.</span></span>

> [!WARNING]
> <span data-ttu-id="eb861-405">Può essere pericoloso risolvere un servizio con ambito da un singleton.</span><span class="sxs-lookup"><span data-stu-id="eb861-405">It's dangerous to resolve a scoped service from a singleton.</span></span> <span data-ttu-id="eb861-406">Ciò potrebbe causare uno stato non corretto per il servizio durante l'elaborazione delle richieste successive.</span><span class="sxs-lookup"><span data-stu-id="eb861-406">It may cause the service to have incorrect state when processing subsequent requests.</span></span>

## <a name="service-registration-methods"></a><span data-ttu-id="eb861-407">Metodi di registrazione del servizio</span><span class="sxs-lookup"><span data-stu-id="eb861-407">Service registration methods</span></span>

<span data-ttu-id="eb861-408">I metodi di estensione per la registrazione del servizio offrono overload che risultano utili in scenari specifici.</span><span class="sxs-lookup"><span data-stu-id="eb861-408">Service registration extension methods offer overloads that are useful in specific scenarios.</span></span>

| <span data-ttu-id="eb861-409">Metodo</span><span class="sxs-lookup"><span data-stu-id="eb861-409">Method</span></span> | <span data-ttu-id="eb861-410">Automatico</span><span class="sxs-lookup"><span data-stu-id="eb861-410">Automatic</span></span><br><span data-ttu-id="eb861-411">object</span><span class="sxs-lookup"><span data-stu-id="eb861-411">object</span></span><br><span data-ttu-id="eb861-412">eliminazione</span><span class="sxs-lookup"><span data-stu-id="eb861-412">disposal</span></span> | <span data-ttu-id="eb861-413">Più elementi</span><span class="sxs-lookup"><span data-stu-id="eb861-413">Multiple</span></span><br><span data-ttu-id="eb861-414">implementazioni</span><span class="sxs-lookup"><span data-stu-id="eb861-414">implementations</span></span> | <span data-ttu-id="eb861-415">Pass args</span><span class="sxs-lookup"><span data-stu-id="eb861-415">Pass args</span></span> |
| ------ | :-----------------------------: | :-------------------------: | :-------: |
| `Add{LIFETIME}<{SERVICE}, {IMPLEMENTATION}>()`<br><span data-ttu-id="eb861-416">Esempio:</span><span class="sxs-lookup"><span data-stu-id="eb861-416">Example:</span></span><br>`services.AddSingleton<IMyDep, MyDep>();` | <span data-ttu-id="eb861-417">Sì</span><span class="sxs-lookup"><span data-stu-id="eb861-417">Yes</span></span> | <span data-ttu-id="eb861-418">Sì</span><span class="sxs-lookup"><span data-stu-id="eb861-418">Yes</span></span> | <span data-ttu-id="eb861-419">No</span><span class="sxs-lookup"><span data-stu-id="eb861-419">No</span></span> |
| `Add{LIFETIME}<{SERVICE}>(sp => new {IMPLEMENTATION})`<br><span data-ttu-id="eb861-420">Esempi:</span><span class="sxs-lookup"><span data-stu-id="eb861-420">Examples:</span></span><br>`services.AddSingleton<IMyDep>(sp => new MyDep());`<br>`services.AddSingleton<IMyDep>(sp => new MyDep("A string!"));` | <span data-ttu-id="eb861-421">Sì</span><span class="sxs-lookup"><span data-stu-id="eb861-421">Yes</span></span> | <span data-ttu-id="eb861-422">Sì</span><span class="sxs-lookup"><span data-stu-id="eb861-422">Yes</span></span> | <span data-ttu-id="eb861-423">Sì</span><span class="sxs-lookup"><span data-stu-id="eb861-423">Yes</span></span> |
| `Add{LIFETIME}<{IMPLEMENTATION}>()`<br><span data-ttu-id="eb861-424">Esempio:</span><span class="sxs-lookup"><span data-stu-id="eb861-424">Example:</span></span><br>`services.AddSingleton<MyDep>();` | <span data-ttu-id="eb861-425">Sì</span><span class="sxs-lookup"><span data-stu-id="eb861-425">Yes</span></span> | <span data-ttu-id="eb861-426">No</span><span class="sxs-lookup"><span data-stu-id="eb861-426">No</span></span> | <span data-ttu-id="eb861-427">No</span><span class="sxs-lookup"><span data-stu-id="eb861-427">No</span></span> |
| `AddSingleton<{SERVICE}>(new {IMPLEMENTATION})`<br><span data-ttu-id="eb861-428">Esempi:</span><span class="sxs-lookup"><span data-stu-id="eb861-428">Examples:</span></span><br>`services.AddSingleton<IMyDep>(new MyDep());`<br>`services.AddSingleton<IMyDep>(new MyDep("A string!"));` | <span data-ttu-id="eb861-429">No</span><span class="sxs-lookup"><span data-stu-id="eb861-429">No</span></span> | <span data-ttu-id="eb861-430">Sì</span><span class="sxs-lookup"><span data-stu-id="eb861-430">Yes</span></span> | <span data-ttu-id="eb861-431">Sì</span><span class="sxs-lookup"><span data-stu-id="eb861-431">Yes</span></span> |
| `AddSingleton(new {IMPLEMENTATION})`<br><span data-ttu-id="eb861-432">Esempi:</span><span class="sxs-lookup"><span data-stu-id="eb861-432">Examples:</span></span><br>`services.AddSingleton(new MyDep());`<br>`services.AddSingleton(new MyDep("A string!"));` | <span data-ttu-id="eb861-433">No</span><span class="sxs-lookup"><span data-stu-id="eb861-433">No</span></span> | <span data-ttu-id="eb861-434">No</span><span class="sxs-lookup"><span data-stu-id="eb861-434">No</span></span> | <span data-ttu-id="eb861-435">Sì</span><span class="sxs-lookup"><span data-stu-id="eb861-435">Yes</span></span> |

<span data-ttu-id="eb861-436">Per ulteriori informazioni sull'eliminazione dei tipi, vedere la sezione [Eliminazione dei servizi](#disposal-of-services).</span><span class="sxs-lookup"><span data-stu-id="eb861-436">For more information on type disposal, see the [Disposal of services](#disposal-of-services) section.</span></span> <span data-ttu-id="eb861-437">Uno scenario comune per più implementazioni è costituito dai [tipi di simulazioni per i test](xref:test/integration-tests#inject-mock-services).</span><span class="sxs-lookup"><span data-stu-id="eb861-437">A common scenario for multiple implementations is [mocking types for testing](xref:test/integration-tests#inject-mock-services).</span></span>

<span data-ttu-id="eb861-438">La registrazione di un servizio con un solo tipo di implementazione equivale alla registrazione del servizio con lo stesso tipo di implementazione e di servizio.</span><span class="sxs-lookup"><span data-stu-id="eb861-438">Registering a service with only an implementation type is equivalent to registering that service with the same implementation and service type.</span></span> <span data-ttu-id="eb861-439">Questo è il motivo per cui non è possibile registrare più implementazioni di un servizio usando i metodi che non accettano un tipo di servizio esplicito.</span><span class="sxs-lookup"><span data-stu-id="eb861-439">This is why multiple implementations of a service cannot be registered using the methods that don't take an explicit service type.</span></span> <span data-ttu-id="eb861-440">Questi metodi possono registrare più *istanze* di un servizio, ma avranno tutti lo stesso tipo di *implementazione* .</span><span class="sxs-lookup"><span data-stu-id="eb861-440">These methods can register multiple *instances* of a service, but they will all have the same *implementation* type.</span></span>

<span data-ttu-id="eb861-441">Uno qualsiasi dei metodi di registrazione del servizio sopra indicati può essere utilizzato per registrare più istanze del servizio dello stesso tipo di servizio.</span><span class="sxs-lookup"><span data-stu-id="eb861-441">Any of the above service registration methods can be used to register multiple service instances of the same service type.</span></span> <span data-ttu-id="eb861-442">Nell'esempio seguente `AddSingleton` viene chiamato due volte con `IMyDependency` come tipo di servizio.</span><span class="sxs-lookup"><span data-stu-id="eb861-442">In the following example, `AddSingleton` is called twice with `IMyDependency` as the service type.</span></span> <span data-ttu-id="eb861-443">La seconda chiamata a `AddSingleton` sostituisce quella precedente quando viene risolta come `IMyDependency` e aggiunge a quella precedente quando più servizi vengono risolti tramite `IEnumerable<IMyDependency>` .</span><span class="sxs-lookup"><span data-stu-id="eb861-443">The second call to `AddSingleton` overrides the previous one when resolved as `IMyDependency` and adds to the previous one when multiple services are resolved via `IEnumerable<IMyDependency>`.</span></span> <span data-ttu-id="eb861-444">I servizi vengono visualizzati nell'ordine in cui sono stati registrati durante la risoluzione tramite `IEnumerable<{SERVICE}>` .</span><span class="sxs-lookup"><span data-stu-id="eb861-444">Services appear in the order they were registered when resolved via `IEnumerable<{SERVICE}>`.</span></span>

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

<span data-ttu-id="eb861-445">Il Framework fornisce anche `TryAdd{LIFETIME}` metodi di estensione, che registrano il servizio solo se non è già stata registrata un'implementazione.</span><span class="sxs-lookup"><span data-stu-id="eb861-445">The framework also provides `TryAdd{LIFETIME}` extension methods, which register the service only if there isn't already an implementation registered.</span></span>

<span data-ttu-id="eb861-446">Nell'esempio seguente, la chiamata a `AddSingleton` Registra `MyDependency` come implementazione per `IMyDependency` .</span><span class="sxs-lookup"><span data-stu-id="eb861-446">In the following example, the call to `AddSingleton` registers `MyDependency` as an implementation for `IMyDependency`.</span></span> <span data-ttu-id="eb861-447">La chiamata a `TryAddSingleton` non ha alcun effetto perché `IMyDependency` dispone già di un'implementazione registrata.</span><span class="sxs-lookup"><span data-stu-id="eb861-447">The call to `TryAddSingleton` has no effect because `IMyDependency` already has a registered implementation.</span></span>

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

<span data-ttu-id="eb861-448">Per altre informazioni, vedere:</span><span class="sxs-lookup"><span data-stu-id="eb861-448">For more information, see:</span></span>

* <xref:Microsoft.Extensions.DependencyInjection.Extensions.ServiceCollectionDescriptorExtensions.TryAdd*>
* <xref:Microsoft.Extensions.DependencyInjection.Extensions.ServiceCollectionDescriptorExtensions.TryAddTransient*>
* <xref:Microsoft.Extensions.DependencyInjection.Extensions.ServiceCollectionDescriptorExtensions.TryAddScoped*>
* <xref:Microsoft.Extensions.DependencyInjection.Extensions.ServiceCollectionDescriptorExtensions.TryAddSingleton*>

<span data-ttu-id="eb861-449">I metodi [TryAddEnumerable(ServiceDescriptor)](xref:Microsoft.Extensions.DependencyInjection.Extensions.ServiceCollectionDescriptorExtensions.TryAddEnumerable*) registrano il servizio solo se non esiste già un'implementazione *dello stesso tipo*.</span><span class="sxs-lookup"><span data-stu-id="eb861-449">[TryAddEnumerable(ServiceDescriptor)](xref:Microsoft.Extensions.DependencyInjection.Extensions.ServiceCollectionDescriptorExtensions.TryAddEnumerable*) methods only register the service if there isn't already an implementation *of the same type*.</span></span> <span data-ttu-id="eb861-450">Più servizi vengono risolti tramite `IEnumerable<{SERVICE}>`.</span><span class="sxs-lookup"><span data-stu-id="eb861-450">Multiple services are resolved via `IEnumerable<{SERVICE}>`.</span></span> <span data-ttu-id="eb861-451">Durante la registrazione di servizi, lo sviluppatore desidera aggiungere un'istanza solo se non ne è già stata aggiunta una dello stesso tipo.</span><span class="sxs-lookup"><span data-stu-id="eb861-451">When registering services, the developer only wants to add an instance if one of the same type hasn't already been added.</span></span> <span data-ttu-id="eb861-452">In genere, questo metodo viene utilizzato dagli autori di librerie per evitare di registrare due copie di un'istanza nel contenitore.</span><span class="sxs-lookup"><span data-stu-id="eb861-452">Generally, this method is used by library authors to avoid registering two copies of an instance in the container.</span></span>

<span data-ttu-id="eb861-453">Nell'esempio seguente, la prima riga registra `MyDep` per `IMyDep1`.</span><span class="sxs-lookup"><span data-stu-id="eb861-453">In the following example, the first line registers `MyDep` for `IMyDep1`.</span></span> <span data-ttu-id="eb861-454">La seconda riga registra `MyDep` per `IMyDep2`.</span><span class="sxs-lookup"><span data-stu-id="eb861-454">The second line registers `MyDep` for `IMyDep2`.</span></span> <span data-ttu-id="eb861-455">La seconda riga non ha alcun effetto perché `IMyDep1` dispone già di un'implementazione registrata di `MyDep`:</span><span class="sxs-lookup"><span data-stu-id="eb861-455">The third line has no effect because `IMyDep1` already has a registered implementation of `MyDep`:</span></span>

```csharp
public interface IMyDep1 {}
public interface IMyDep2 {}

public class MyDep : IMyDep1, IMyDep2 {}

services.TryAddEnumerable(ServiceDescriptor.Singleton<IMyDep1, MyDep>());
services.TryAddEnumerable(ServiceDescriptor.Singleton<IMyDep2, MyDep>());
// Two registrations of MyDep for IMyDep1 is avoided by the following line:
services.TryAddEnumerable(ServiceDescriptor.Singleton<IMyDep1, MyDep>());
```

### <a name="constructor-injection-behavior"></a><span data-ttu-id="eb861-456">Comportamento dell'inserimento del costruttore</span><span class="sxs-lookup"><span data-stu-id="eb861-456">Constructor injection behavior</span></span>

<span data-ttu-id="eb861-457">I servizi possono essere risolti con due meccanismi:</span><span class="sxs-lookup"><span data-stu-id="eb861-457">Services can be resolved by two mechanisms:</span></span>

* <xref:System.IServiceProvider>
* <span data-ttu-id="eb861-458"><xref:Microsoft.Extensions.DependencyInjection.ActivatorUtilities>: Consente la creazione di oggetti senza registrazione del servizio nel contenitore di inserimento delle dipendenze.</span><span class="sxs-lookup"><span data-stu-id="eb861-458"><xref:Microsoft.Extensions.DependencyInjection.ActivatorUtilities>: Permits object creation without service registration in the dependency injection container.</span></span> <span data-ttu-id="eb861-459">`ActivatorUtilities` viene usato con astrazioni esposte all'utente, ad esempio helper tag, controller MVC e strumenti di associazione di modelli.</span><span class="sxs-lookup"><span data-stu-id="eb861-459">`ActivatorUtilities` is used with user-facing abstractions, such as Tag Helpers, MVC controllers, and model binders.</span></span>

<span data-ttu-id="eb861-460">I costruttori possono accettare argomenti non forniti tramite l'inserimento di dipendenze, ma gli argomenti devono assegnare valori predefiniti.</span><span class="sxs-lookup"><span data-stu-id="eb861-460">Constructors can accept arguments that aren't provided by dependency injection, but the arguments must assign default values.</span></span>

<span data-ttu-id="eb861-461">Quando i servizi vengono risolti da `IServiceProvider` o `ActivatorUtilities` , l' [inserimento del costruttore](xref:mvc/controllers/dependency-injection#constructor-injection) richiede un costruttore *pubblico* .</span><span class="sxs-lookup"><span data-stu-id="eb861-461">When services are resolved by `IServiceProvider` or `ActivatorUtilities`, [constructor injection](xref:mvc/controllers/dependency-injection#constructor-injection) requires a *public* constructor.</span></span>

<span data-ttu-id="eb861-462">Quando i servizi vengono risolti da `ActivatorUtilities` , l' [inserimento del costruttore](xref:mvc/controllers/dependency-injection#constructor-injection) richiede l'esistenza di un solo costruttore applicabile.</span><span class="sxs-lookup"><span data-stu-id="eb861-462">When services are resolved by `ActivatorUtilities`, [constructor injection](xref:mvc/controllers/dependency-injection#constructor-injection) requires that only one applicable constructor exists.</span></span> <span data-ttu-id="eb861-463">Sebbene siano supportati gli overload dei costruttori, può essere presente un solo overload i cui argomenti possono essere tutti specificati tramite l'inserimento delle dipendenze.</span><span class="sxs-lookup"><span data-stu-id="eb861-463">Constructor overloads are supported, but only one overload can exist whose arguments can all be fulfilled by dependency injection.</span></span>

## <a name="entity-framework-contexts"></a><span data-ttu-id="eb861-464">Contesti di Entity Framework</span><span class="sxs-lookup"><span data-stu-id="eb861-464">Entity Framework contexts</span></span>

<span data-ttu-id="eb861-465">I contesti di Entity Framework vengono in genere aggiunti al contenitore di servizi usando la [durata con ambito](#service-lifetimes) perché le operazioni di database delle app Web hanno di solito come ambito la richiesta client.</span><span class="sxs-lookup"><span data-stu-id="eb861-465">Entity Framework contexts are usually added to the service container using the [scoped lifetime](#service-lifetimes) because web app database operations are normally scoped to the client request.</span></span> <span data-ttu-id="eb861-466">L'ambito della durata predefinita è se una durata non è specificata da un overload [AddDbContext \<TContext> ](/dotnet/api/microsoft.extensions.dependencyinjection.entityframeworkservicecollectionextensions.adddbcontext) durante la registrazione del contesto del database.</span><span class="sxs-lookup"><span data-stu-id="eb861-466">The default lifetime is scoped if a lifetime isn't specified by an [AddDbContext\<TContext>](/dotnet/api/microsoft.extensions.dependencyinjection.entityframeworkservicecollectionextensions.adddbcontext) overload when registering the database context.</span></span> <span data-ttu-id="eb861-467">I servizi con una durata specificata non devono usare un contesto di database con una durata più breve rispetto al servizio.</span><span class="sxs-lookup"><span data-stu-id="eb861-467">Services of a given lifetime shouldn't use a database context with a shorter lifetime than the service.</span></span>

## <a name="lifetime-and-registration-options"></a><span data-ttu-id="eb861-468">Opzioni di durata e di registrazione</span><span class="sxs-lookup"><span data-stu-id="eb861-468">Lifetime and registration options</span></span>

<span data-ttu-id="eb861-469">Per illustrare la differenza tra le opzioni di durata e registrazione, si considerino le interfacce seguenti che rappresentano le attività come un'operazione con un identificatore univoco, `OperationId`.</span><span class="sxs-lookup"><span data-stu-id="eb861-469">To demonstrate the difference between the lifetime and registration options, consider the following interfaces that represent tasks as an operation with a unique identifier, `OperationId`.</span></span> <span data-ttu-id="eb861-470">A seconda del modo in cui la durata di un servizio operativo viene configurata per le interfacce seguenti, il contenitore fornisce la stessa istanza o un'istanza diversa del servizio quando richiesto da una classe:</span><span class="sxs-lookup"><span data-stu-id="eb861-470">Depending on how the lifetime of an operations service is configured for the following interfaces, the container provides either the same or a different instance of the service when requested by a class:</span></span>

[!code-csharp[](dependency-injection/samples/2.x/DependencyInjectionSample/Interfaces/IOperation.cs?name=snippet1)]

<span data-ttu-id="eb861-471">Le interfacce vengono implementate nella classe `Operation`.</span><span class="sxs-lookup"><span data-stu-id="eb861-471">The interfaces are implemented in the `Operation` class.</span></span> <span data-ttu-id="eb861-472">Il costruttore `Operation` genera un GUID se non ne viene fornito uno:</span><span class="sxs-lookup"><span data-stu-id="eb861-472">The `Operation` constructor generates a GUID if one isn't supplied:</span></span>

[!code-csharp[](dependency-injection/samples/2.x/DependencyInjectionSample/Models/Operation.cs?name=snippet1)]

<span data-ttu-id="eb861-473">Viene registrato un `OperationService` che dipende da ognuno degli altri tipi `Operation`.</span><span class="sxs-lookup"><span data-stu-id="eb861-473">An `OperationService` is registered that depends on each of the other `Operation` types.</span></span> <span data-ttu-id="eb861-474">Quando viene richiesto `OperationService` tramite l'inserimento delle dipendenze, riceve una nuova istanza di ogni servizio o un'istanza esistente in base alla durata del servizio dipendente.</span><span class="sxs-lookup"><span data-stu-id="eb861-474">When `OperationService` is requested via dependency injection, it receives either a new instance of each service or an existing instance based on the lifetime of the dependent service.</span></span>

* <span data-ttu-id="eb861-475">Se i servizi temporanei vengono creati quando vengono richiesti dal contenitore, `OperationId` del servizio `IOperationTransient` è diverso da `OperationId` di `OperationService`.</span><span class="sxs-lookup"><span data-stu-id="eb861-475">When transient services are created when requested from the container, the `OperationId` of the `IOperationTransient` service is different than the `OperationId` of the `OperationService`.</span></span> <span data-ttu-id="eb861-476">`OperationService` riceve una nuova istanza della classe `IOperationTransient`.</span><span class="sxs-lookup"><span data-stu-id="eb861-476">`OperationService` receives a new instance of the `IOperationTransient` class.</span></span> <span data-ttu-id="eb861-477">La nuova istanza restituisce un `OperationId` diverso.</span><span class="sxs-lookup"><span data-stu-id="eb861-477">The new instance yields a different `OperationId`.</span></span>
* <span data-ttu-id="eb861-478">Se i servizi con ambito vengono creati per ogni richiesta client, `OperationId` del servizio `IOperationScoped` corrisponde a `OperationService` all'interno di una richiesta client.</span><span class="sxs-lookup"><span data-stu-id="eb861-478">When scoped services are created per client request, the `OperationId` of the `IOperationScoped` service is the same as that of `OperationService` within a client request.</span></span> <span data-ttu-id="eb861-479">In tutte le richieste client entrambi i servizi condividono un valore `OperationId` diverso.</span><span class="sxs-lookup"><span data-stu-id="eb861-479">Across client requests, both services share a different `OperationId` value.</span></span>
* <span data-ttu-id="eb861-480">Se i servizi singleton e con istanza singleton vengono creati una sola volta e usati per tutte le richieste client e tutti i servizi, `OperationId` rimane costante in tutte le richieste di servizio.</span><span class="sxs-lookup"><span data-stu-id="eb861-480">When singleton and singleton-instance services are created once and used across all client requests and all services, the `OperationId` is constant across all service requests.</span></span>

[!code-csharp[](dependency-injection/samples/2.x/DependencyInjectionSample/Services/OperationService.cs?name=snippet1)]

<span data-ttu-id="eb861-481">In `Startup.ConfigureServices`, ogni tipo viene aggiunto al contenitore in base alla durata denominata:</span><span class="sxs-lookup"><span data-stu-id="eb861-481">In `Startup.ConfigureServices`, each type is added to the container according to its named lifetime:</span></span>

[!code-csharp[](dependency-injection/samples/2.x/DependencyInjectionSample/Startup.cs?name=snippet1&highlight=6-9,12)]

<span data-ttu-id="eb861-482">Il servizio `IOperationSingletonInstance` usa un'istanza specifica con un ID noto di `Guid.Empty`.</span><span class="sxs-lookup"><span data-stu-id="eb861-482">The `IOperationSingletonInstance` service is using a specific instance with a known ID of `Guid.Empty`.</span></span> <span data-ttu-id="eb861-483">È chiaro quando questo tipo è in uso (il relativo GUID è composto da tutti zero).</span><span class="sxs-lookup"><span data-stu-id="eb861-483">It's clear when this type is in use (its GUID is all zeroes).</span></span>

<span data-ttu-id="eb861-484">L'app di esempio dimostra le durate degli oggetti all'interno e tra le singole richieste.</span><span class="sxs-lookup"><span data-stu-id="eb861-484">The sample app demonstrates object lifetimes within and between individual requests.</span></span> <span data-ttu-id="eb861-485">L'`IndexModel` dell'app di esempio richiede ogni genere di tipo `IOperation` e `OperationService`.</span><span class="sxs-lookup"><span data-stu-id="eb861-485">The sample app's `IndexModel` requests each kind of `IOperation` type and the `OperationService`.</span></span> <span data-ttu-id="eb861-486">La pagina visualizza quindi tutti i valori `OperationId` della classe del modello di pagina e del servizio tramite assegnazioni di proprietà:</span><span class="sxs-lookup"><span data-stu-id="eb861-486">The page then displays all of the page model class's and service's `OperationId` values through property assignments:</span></span>

[!code-csharp[](dependency-injection/samples/2.x/DependencyInjectionSample/Pages/Index.cshtml.cs?name=snippet1&highlight=7-11,14-18,21-25)]

<span data-ttu-id="eb861-487">L'output seguente mostra i risultati delle due richieste:</span><span class="sxs-lookup"><span data-stu-id="eb861-487">Two following output shows the results of two requests:</span></span>

<span data-ttu-id="eb861-488">**Prima richiesta:**</span><span class="sxs-lookup"><span data-stu-id="eb861-488">**First request:**</span></span>

<span data-ttu-id="eb861-489">Operazioni del controller:</span><span class="sxs-lookup"><span data-stu-id="eb861-489">Controller operations:</span></span>

<span data-ttu-id="eb861-490">Transient: d233e165-f417-469b-a866-1cf1935d2518</span><span class="sxs-lookup"><span data-stu-id="eb861-490">Transient: d233e165-f417-469b-a866-1cf1935d2518</span></span>  
<span data-ttu-id="eb861-491">Scoped: 5d997e2d-55f5-4a64-8388-51c4e3a1ad19</span><span class="sxs-lookup"><span data-stu-id="eb861-491">Scoped: 5d997e2d-55f5-4a64-8388-51c4e3a1ad19</span></span>  
<span data-ttu-id="eb861-492">Singleton: 01271bc1-9e31-48e7-8f7c-7261b040ded9</span><span class="sxs-lookup"><span data-stu-id="eb861-492">Singleton: 01271bc1-9e31-48e7-8f7c-7261b040ded9</span></span>  
<span data-ttu-id="eb861-493">Instance: 00000000-0000-0000-0000-000000000000</span><span class="sxs-lookup"><span data-stu-id="eb861-493">Instance: 00000000-0000-0000-0000-000000000000</span></span>

<span data-ttu-id="eb861-494">Operazioni di `OperationService`:</span><span class="sxs-lookup"><span data-stu-id="eb861-494">`OperationService` operations:</span></span>

<span data-ttu-id="eb861-495">Transient: c6b049eb-1318-4e31-90f1-eb2dd849ff64</span><span class="sxs-lookup"><span data-stu-id="eb861-495">Transient: c6b049eb-1318-4e31-90f1-eb2dd849ff64</span></span>  
<span data-ttu-id="eb861-496">Scoped: 5d997e2d-55f5-4a64-8388-51c4e3a1ad19</span><span class="sxs-lookup"><span data-stu-id="eb861-496">Scoped: 5d997e2d-55f5-4a64-8388-51c4e3a1ad19</span></span>  
<span data-ttu-id="eb861-497">Singleton: 01271bc1-9e31-48e7-8f7c-7261b040ded9</span><span class="sxs-lookup"><span data-stu-id="eb861-497">Singleton: 01271bc1-9e31-48e7-8f7c-7261b040ded9</span></span>  
<span data-ttu-id="eb861-498">Instance: 00000000-0000-0000-0000-000000000000</span><span class="sxs-lookup"><span data-stu-id="eb861-498">Instance: 00000000-0000-0000-0000-000000000000</span></span>

<span data-ttu-id="eb861-499">**Seconda richiesta:**</span><span class="sxs-lookup"><span data-stu-id="eb861-499">**Second request:**</span></span>

<span data-ttu-id="eb861-500">Operazioni del controller:</span><span class="sxs-lookup"><span data-stu-id="eb861-500">Controller operations:</span></span>

<span data-ttu-id="eb861-501">Transient: b63bd538-0a37-4ff1-90ba-081c5138dda0</span><span class="sxs-lookup"><span data-stu-id="eb861-501">Transient: b63bd538-0a37-4ff1-90ba-081c5138dda0</span></span>  
<span data-ttu-id="eb861-502">Scoped: 31e820c5-4834-4d22-83fc-a60118acb9f4</span><span class="sxs-lookup"><span data-stu-id="eb861-502">Scoped: 31e820c5-4834-4d22-83fc-a60118acb9f4</span></span>  
<span data-ttu-id="eb861-503">Singleton: 01271bc1-9e31-48e7-8f7c-7261b040ded9</span><span class="sxs-lookup"><span data-stu-id="eb861-503">Singleton: 01271bc1-9e31-48e7-8f7c-7261b040ded9</span></span>  
<span data-ttu-id="eb861-504">Instance: 00000000-0000-0000-0000-000000000000</span><span class="sxs-lookup"><span data-stu-id="eb861-504">Instance: 00000000-0000-0000-0000-000000000000</span></span>

<span data-ttu-id="eb861-505">Operazioni di `OperationService`:</span><span class="sxs-lookup"><span data-stu-id="eb861-505">`OperationService` operations:</span></span>

<span data-ttu-id="eb861-506">Transient: c4cbacb8-36a2-436d-81c8-8c1b78808aaf</span><span class="sxs-lookup"><span data-stu-id="eb861-506">Transient: c4cbacb8-36a2-436d-81c8-8c1b78808aaf</span></span>  
<span data-ttu-id="eb861-507">Scoped: 31e820c5-4834-4d22-83fc-a60118acb9f4</span><span class="sxs-lookup"><span data-stu-id="eb861-507">Scoped: 31e820c5-4834-4d22-83fc-a60118acb9f4</span></span>  
<span data-ttu-id="eb861-508">Singleton: 01271bc1-9e31-48e7-8f7c-7261b040ded9</span><span class="sxs-lookup"><span data-stu-id="eb861-508">Singleton: 01271bc1-9e31-48e7-8f7c-7261b040ded9</span></span>  
<span data-ttu-id="eb861-509">Instance: 00000000-0000-0000-0000-000000000000</span><span class="sxs-lookup"><span data-stu-id="eb861-509">Instance: 00000000-0000-0000-0000-000000000000</span></span>

<span data-ttu-id="eb861-510">Osservare i valori `OperationId` che variano all'interno della richiesta e da una richiesta e l'altra:</span><span class="sxs-lookup"><span data-stu-id="eb861-510">Observe which of the `OperationId` values vary within a request and between requests:</span></span>

* <span data-ttu-id="eb861-511">Gli oggetti *temporanei* sono sempre diversi.</span><span class="sxs-lookup"><span data-stu-id="eb861-511">*Transient* objects are always different.</span></span> <span data-ttu-id="eb861-512">Il valore `OperationId` temporaneo per la prima e la seconda richiesta client è diverso per entrambe le operazioni `OperationService` e in tutte le richieste client.</span><span class="sxs-lookup"><span data-stu-id="eb861-512">The transient `OperationId` value for both the first and second client requests are different for both `OperationService` operations and across client requests.</span></span> <span data-ttu-id="eb861-513">Viene fornita una nuova istanza per ogni richiesta di servizio e richiesta client.</span><span class="sxs-lookup"><span data-stu-id="eb861-513">A new instance is provided to each service request and client request.</span></span>
* <span data-ttu-id="eb861-514">Gli oggetti *con ambito* sono gli stessi all'interno di una richiesta client, ma variano nelle diverse richieste client.</span><span class="sxs-lookup"><span data-stu-id="eb861-514">*Scoped* objects are the same within a client request but different across client requests.</span></span>
* <span data-ttu-id="eb861-515">Gli oggetti *singleton* sono gli stessi per ogni oggetto e per ogni richiesta, indipendentemente dal fatto che un' `Operation` istanza venga fornita in `Startup.ConfigureServices` .</span><span class="sxs-lookup"><span data-stu-id="eb861-515">*Singleton* objects are the same for every object and every request regardless of whether an `Operation` instance is provided in `Startup.ConfigureServices`.</span></span>

## <a name="call-services-from-main"></a><span data-ttu-id="eb861-516">Chiamare i servizi da main</span><span class="sxs-lookup"><span data-stu-id="eb861-516">Call services from main</span></span>

<span data-ttu-id="eb861-517">Creare un <xref:Microsoft.Extensions.DependencyInjection.IServiceScope>IServiceScope con [IServiceScopeFactory.CreateScope](xref:Microsoft.Extensions.DependencyInjection.IServiceScopeFactory.CreateScope*) per risolvere un servizio con ambito nell'ambito di applicazione dell'app.</span><span class="sxs-lookup"><span data-stu-id="eb861-517">Create an <xref:Microsoft.Extensions.DependencyInjection.IServiceScope> with [IServiceScopeFactory.CreateScope](xref:Microsoft.Extensions.DependencyInjection.IServiceScopeFactory.CreateScope*) to resolve a scoped service within the app's scope.</span></span> <span data-ttu-id="eb861-518">Questo approccio è utile per l'accesso a un servizio con ambito all'avvio e per l'esecuzione di attività di inizializzazione.</span><span class="sxs-lookup"><span data-stu-id="eb861-518">This approach is useful to access a scoped service at startup to run initialization tasks.</span></span> <span data-ttu-id="eb861-519">L'esempio seguente indica come ottenere un contesto per `MyScopedService` in `Program.Main`:</span><span class="sxs-lookup"><span data-stu-id="eb861-519">The following example shows how to obtain a context for the `MyScopedService` in `Program.Main`:</span></span>

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

## <a name="scope-validation"></a><span data-ttu-id="eb861-520">Convalida dell'ambito</span><span class="sxs-lookup"><span data-stu-id="eb861-520">Scope validation</span></span>

<span data-ttu-id="eb861-521">Quando l'app viene eseguita nell'ambiente di sviluppo, il provider di servizi predefinito esegue controlli per verificare che:</span><span class="sxs-lookup"><span data-stu-id="eb861-521">When the app is running in the Development environment, the default service provider performs checks to verify that:</span></span>

* <span data-ttu-id="eb861-522">I servizi con ambito non vengano risolti direttamente o indirettamente dal provider di servizi radice.</span><span class="sxs-lookup"><span data-stu-id="eb861-522">Scoped services aren't directly or indirectly resolved from the root service provider.</span></span>
* <span data-ttu-id="eb861-523">I servizi con ambito non vengano inseriti direttamente o indirettamente in singleton.</span><span class="sxs-lookup"><span data-stu-id="eb861-523">Scoped services aren't directly or indirectly injected into singletons.</span></span>

<span data-ttu-id="eb861-524">Il provider di servizi radice venga creato con la chiamata di <xref:Microsoft.Extensions.DependencyInjection.ServiceCollectionContainerBuilderExtensions.BuildServiceProvider*>.</span><span class="sxs-lookup"><span data-stu-id="eb861-524">The root service provider is created when <xref:Microsoft.Extensions.DependencyInjection.ServiceCollectionContainerBuilderExtensions.BuildServiceProvider*> is called.</span></span> <span data-ttu-id="eb861-525">La durata del provider di servizi radice corrisponde alla durata dell'app/server quando il provider viene avviato con l'app e viene eliminato alla chiusura dell'app.</span><span class="sxs-lookup"><span data-stu-id="eb861-525">The root service provider's lifetime corresponds to the app/server's lifetime when the provider starts with the app and is disposed when the app shuts down.</span></span>

<span data-ttu-id="eb861-526">I servizi con ambito vengono eliminati dal contenitore che li ha creati.</span><span class="sxs-lookup"><span data-stu-id="eb861-526">Scoped services are disposed by the container that created them.</span></span> <span data-ttu-id="eb861-527">Se un servizio con ambito viene creato nel contenitore radice, la durata del servizio viene di fatto convertita in singleton, perché il servizio viene eliminato solo dal contenitore radice alla chiusura dell'app/server.</span><span class="sxs-lookup"><span data-stu-id="eb861-527">If a scoped service is created in the root container, the service's lifetime is effectively promoted to singleton because it's only disposed by the root container when app/server is shut down.</span></span> <span data-ttu-id="eb861-528">La convalida degli ambiti servizio rileva queste situazioni nel contesto della chiamata di `BuildServiceProvider`.</span><span class="sxs-lookup"><span data-stu-id="eb861-528">Validating service scopes catches these situations when `BuildServiceProvider` is called.</span></span>

<span data-ttu-id="eb861-529">Per altre informazioni, vedere <xref:fundamentals/host/web-host#scope-validation>.</span><span class="sxs-lookup"><span data-stu-id="eb861-529">For more information, see <xref:fundamentals/host/web-host#scope-validation>.</span></span>   

## <a name="request-services"></a><span data-ttu-id="eb861-530">Servizi di richiesta</span><span class="sxs-lookup"><span data-stu-id="eb861-530">Request Services</span></span>

<span data-ttu-id="eb861-531">I servizi disponibili all'interno di una richiesta ASP.NET Core da `HttpContext` vengono esposti tramite la raccolta [HttpContext.RequestServices](xref:Microsoft.AspNetCore.Http.HttpContext.RequestServices).</span><span class="sxs-lookup"><span data-stu-id="eb861-531">The services available within an ASP.NET Core request from `HttpContext` are exposed through the [HttpContext.RequestServices](xref:Microsoft.AspNetCore.Http.HttpContext.RequestServices) collection.</span></span>

<span data-ttu-id="eb861-532">I servizi di richiesta rappresentano i servizi configurati e richiesti come parte dell'app.</span><span class="sxs-lookup"><span data-stu-id="eb861-532">Request Services represent the services configured and requested as part of the app.</span></span> <span data-ttu-id="eb861-533">Quando gli oggetti specificano dipendenze, le dipendenze sono soddisfatte dai tipi individuati in `RequestServices`, non in `ApplicationServices`.</span><span class="sxs-lookup"><span data-stu-id="eb861-533">When the objects specify dependencies, these are satisfied by the types found in `RequestServices`, not `ApplicationServices`.</span></span>

<span data-ttu-id="eb861-534">In generale, l'app non deve usare queste proprietà direttamente.</span><span class="sxs-lookup"><span data-stu-id="eb861-534">Generally, the app shouldn't use these properties directly.</span></span> <span data-ttu-id="eb861-535">Richiedere invece i tipi richiesti dalle classi tramite i costruttori di classi e consentire al framework di inserire le dipendenze.</span><span class="sxs-lookup"><span data-stu-id="eb861-535">Instead, request the types that classes require via class constructors and allow the framework inject the dependencies.</span></span> <span data-ttu-id="eb861-536">Si ottengono classi più facili da testare.</span><span class="sxs-lookup"><span data-stu-id="eb861-536">This yields classes that are easier to test.</span></span>

> [!NOTE]
> <span data-ttu-id="eb861-537">È consigliabile richiedere le dipendenze come parametri del costruttore per l'accesso alla raccolta `RequestServices`.</span><span class="sxs-lookup"><span data-stu-id="eb861-537">Prefer requesting dependencies as constructor parameters to accessing the `RequestServices` collection.</span></span>

## <a name="design-services-for-dependency-injection"></a><span data-ttu-id="eb861-538">Progettare i servizi per l'inserimento di dipendenze</span><span class="sxs-lookup"><span data-stu-id="eb861-538">Design services for dependency injection</span></span>

<span data-ttu-id="eb861-539">Le procedure consigliate prevedono di:</span><span class="sxs-lookup"><span data-stu-id="eb861-539">Best practices are to:</span></span>

* <span data-ttu-id="eb861-540">Progettare i servizi per usare l'inserimento delle dipendenze per ottenere le relative dipendenze.</span><span class="sxs-lookup"><span data-stu-id="eb861-540">Design services to use dependency injection to obtain their dependencies.</span></span>
* <span data-ttu-id="eb861-541">Evitare membri e classi statiche con stato.</span><span class="sxs-lookup"><span data-stu-id="eb861-541">Avoid stateful, static classes and members.</span></span> <span data-ttu-id="eb861-542">Progettare le app in modo da usare i servizi singleton, evitando così di creare uno stato globale.</span><span class="sxs-lookup"><span data-stu-id="eb861-542">Design apps to use singleton services instead, which avoid creating global state.</span></span>
* <span data-ttu-id="eb861-543">Evitare la creazione diretta di istanze delle classi dipendenti all'interno di servizi.</span><span class="sxs-lookup"><span data-stu-id="eb861-543">Avoid direct instantiation of dependent classes within services.</span></span> <span data-ttu-id="eb861-544">La creazione diretta di istanze associa il codice a una determinata implementazione.</span><span class="sxs-lookup"><span data-stu-id="eb861-544">Direct instantiation couples the code to a particular implementation.</span></span>
* <span data-ttu-id="eb861-545">Fare in modo che le classi siano piccole, con factoring corretto e facili da testare.</span><span class="sxs-lookup"><span data-stu-id="eb861-545">Make app classes small, well-factored, and easily tested.</span></span>

<span data-ttu-id="eb861-546">Se una classe sembra avere troppe dipendenze inserite, ciò significa in genere che la classe ha un numero eccessivo di responsabilità e sta violando il [Single Responsibility Principle (SRP)](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#single-responsibility) (principio di responsabilità singola).</span><span class="sxs-lookup"><span data-stu-id="eb861-546">If a class seems to have too many injected dependencies, this is generally a sign that the class has too many responsibilities and is violating the [Single Responsibility Principle (SRP)](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#single-responsibility).</span></span> <span data-ttu-id="eb861-547">Tentare di effettuare il refactoring della classe spostando alcune delle responsabilità in una nuova classe.</span><span class="sxs-lookup"><span data-stu-id="eb861-547">Attempt to refactor the class by moving some of its responsibilities into a new class.</span></span> <span data-ttu-id="eb861-548">Tenere presente che Razor le classi del modello della pagina delle pagine e le classi controller MVC devono concentrarsi sulle problematiche dell'interfaccia utente.</span><span class="sxs-lookup"><span data-stu-id="eb861-548">Keep in mind that Razor Pages page model classes and MVC controller classes should focus on UI concerns.</span></span> <span data-ttu-id="eb861-549">Di conseguenza, le regole business e i dettagli di implementazione di accesso ai dati devono essere inseriti in classi appropriate per queste [problematiche separate](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#separation-of-concerns).</span><span class="sxs-lookup"><span data-stu-id="eb861-549">Business rules and data access implementation details should be kept in classes appropriate to these [separate concerns](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#separation-of-concerns).</span></span>

### <a name="disposal-of-services"></a><span data-ttu-id="eb861-550">Eliminazione dei servizi</span><span class="sxs-lookup"><span data-stu-id="eb861-550">Disposal of services</span></span>

<span data-ttu-id="eb861-551">Il contenitore chiama <xref:System.IDisposable.Dispose*> per i tipi <xref:System.IDisposable> creati.</span><span class="sxs-lookup"><span data-stu-id="eb861-551">The container calls <xref:System.IDisposable.Dispose*> for the <xref:System.IDisposable> types it creates.</span></span> <span data-ttu-id="eb861-552">Se un'istanza viene aggiunta al contenitore dal codice utente, non viene eliminata automaticamente.</span><span class="sxs-lookup"><span data-stu-id="eb861-552">If an instance is added to the container by user code, it isn't disposed automatically.</span></span>

<span data-ttu-id="eb861-553">Nell'esempio seguente i servizi vengono creati dal contenitore del servizio ed eliminati automaticamente:</span><span class="sxs-lookup"><span data-stu-id="eb861-553">In the following example, the services are created by the service container and disposed automatically:</span></span>

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

<span data-ttu-id="eb861-554">Nell'esempio seguente:</span><span class="sxs-lookup"><span data-stu-id="eb861-554">In the following example:</span></span>

* <span data-ttu-id="eb861-555">Le istanze del servizio non vengono create dal contenitore dei servizi.</span><span class="sxs-lookup"><span data-stu-id="eb861-555">The service instances aren't created by the service container.</span></span>
* <span data-ttu-id="eb861-556">Le durate di servizio previste non sono note dal Framework.</span><span class="sxs-lookup"><span data-stu-id="eb861-556">The intended service lifetimes aren't known by the framework.</span></span>
* <span data-ttu-id="eb861-557">Il Framework non elimina automaticamente i servizi.</span><span class="sxs-lookup"><span data-stu-id="eb861-557">The framework doesn't dispose of the services automatically.</span></span>
* <span data-ttu-id="eb861-558">Se i servizi non vengono eliminati in modo esplicito nel codice dello sviluppatore, vengono mantenuti fino a quando l'app non viene chiusa.</span><span class="sxs-lookup"><span data-stu-id="eb861-558">If the services aren't explicitly disposed in developer code, they persist until the app shuts down.</span></span>

```csharp
public class Service1 : IDisposable {}
public class Service2 : IDisposable {}

public void ConfigureServices(IServiceCollection services)
{
    services.AddSingleton<Service1>(new Service1());
    services.AddSingleton(new Service2());
}
```

### <a name="idisposable-guidance-for-transient-and-shared-instances"></a><span data-ttu-id="eb861-559">Linee guida IDisposable per istanze temporanee e condivise</span><span class="sxs-lookup"><span data-stu-id="eb861-559">IDisposable guidance for Transient and shared instances</span></span>

#### <a name="transient-limited-lifetime"></a><span data-ttu-id="eb861-560">Durata temporanea e limitata</span><span class="sxs-lookup"><span data-stu-id="eb861-560">Transient, limited lifetime</span></span>

<span data-ttu-id="eb861-561">**Scenario**</span><span class="sxs-lookup"><span data-stu-id="eb861-561">**Scenario**</span></span>

<span data-ttu-id="eb861-562">L'app richiede un' <xref:System.IDisposable> istanza con una durata temporanea per uno degli scenari seguenti:</span><span class="sxs-lookup"><span data-stu-id="eb861-562">The app requires an <xref:System.IDisposable> instance with a transient lifetime for either of the following scenarios:</span></span>

* <span data-ttu-id="eb861-563">L'istanza viene risolta nell'ambito radice.</span><span class="sxs-lookup"><span data-stu-id="eb861-563">The instance is resolved in the root scope.</span></span>
* <span data-ttu-id="eb861-564">L'istanza deve essere eliminata prima della fine dell'ambito.</span><span class="sxs-lookup"><span data-stu-id="eb861-564">The instance should be disposed before the scope ends.</span></span>

<span data-ttu-id="eb861-565">**Soluzione**</span><span class="sxs-lookup"><span data-stu-id="eb861-565">**Solution**</span></span>

<span data-ttu-id="eb861-566">Usare il modello factory per creare un'istanza all'esterno dell'ambito padre.</span><span class="sxs-lookup"><span data-stu-id="eb861-566">Use the factory pattern to create an instance outside of the parent scope.</span></span> <span data-ttu-id="eb861-567">In questa situazione, in genere l'app dispone di un `Create` metodo che chiama direttamente il costruttore del tipo finale.</span><span class="sxs-lookup"><span data-stu-id="eb861-567">In this situation, the app would generally have a `Create` method that calls the final type's constructor directly.</span></span> <span data-ttu-id="eb861-568">Se il tipo finale ha altre dipendenze, la factory può:</span><span class="sxs-lookup"><span data-stu-id="eb861-568">If the final type has other dependencies, the factory can:</span></span>

* <span data-ttu-id="eb861-569">Ricevere un oggetto <xref:System.IServiceProvider> nel relativo costruttore.</span><span class="sxs-lookup"><span data-stu-id="eb861-569">Receive an <xref:System.IServiceProvider> in its constructor.</span></span>
* <span data-ttu-id="eb861-570">Usare <xref:Microsoft.Extensions.DependencyInjection.ActivatorUtilities.CreateInstance%2A?displayProperty=nameWithType> per creare un'istanza dell'istanza all'esterno del contenitore, usando il contenitore per le relative dipendenze.</span><span class="sxs-lookup"><span data-stu-id="eb861-570">Use <xref:Microsoft.Extensions.DependencyInjection.ActivatorUtilities.CreateInstance%2A?displayProperty=nameWithType> to instantiate the instance outside the container, while using the container for its dependencies.</span></span>

#### <a name="shared-instance-limited-lifetime"></a><span data-ttu-id="eb861-571">Istanza condivisa, durata limitata</span><span class="sxs-lookup"><span data-stu-id="eb861-571">Shared Instance, limited lifetime</span></span>

<span data-ttu-id="eb861-572">**Scenario**</span><span class="sxs-lookup"><span data-stu-id="eb861-572">**Scenario**</span></span>

<span data-ttu-id="eb861-573">L'app richiede un' <xref:System.IDisposable> istanza condivisa tra più servizi, ma <xref:System.IDisposable> deve avere una durata limitata.</span><span class="sxs-lookup"><span data-stu-id="eb861-573">The app requires a shared <xref:System.IDisposable> instance across multiple services, but the <xref:System.IDisposable> should have a limited lifetime.</span></span>

<span data-ttu-id="eb861-574">**Soluzione**</span><span class="sxs-lookup"><span data-stu-id="eb861-574">**Solution**</span></span>

<span data-ttu-id="eb861-575">Registrare l'istanza con una durata con ambito.</span><span class="sxs-lookup"><span data-stu-id="eb861-575">Register the instance with a Scoped lifetime.</span></span> <span data-ttu-id="eb861-576">Usare <xref:Microsoft.Extensions.DependencyInjection.IServiceScopeFactory.CreateScope%2A?displayProperty=nameWithType> per avviare e creare un nuovo oggetto <xref:Microsoft.Extensions.DependencyInjection.IServiceScope> .</span><span class="sxs-lookup"><span data-stu-id="eb861-576">Use <xref:Microsoft.Extensions.DependencyInjection.IServiceScopeFactory.CreateScope%2A?displayProperty=nameWithType> to start and create a new <xref:Microsoft.Extensions.DependencyInjection.IServiceScope>.</span></span> <span data-ttu-id="eb861-577">Usare l'ambito <xref:System.IServiceProvider> per ottenere i servizi richiesti.</span><span class="sxs-lookup"><span data-stu-id="eb861-577">Use the scope's <xref:System.IServiceProvider> to get required services.</span></span> <span data-ttu-id="eb861-578">Eliminare l'ambito quando la durata deve terminare.</span><span class="sxs-lookup"><span data-stu-id="eb861-578">Dispose the scope when the lifetime should end.</span></span>

#### <a name="general-guidelines"></a><span data-ttu-id="eb861-579">Linee guida generali</span><span class="sxs-lookup"><span data-stu-id="eb861-579">General Guidelines</span></span>

* <span data-ttu-id="eb861-580">Non registrare <xref:System.IDisposable> istanze con un ambito temporaneo.</span><span class="sxs-lookup"><span data-stu-id="eb861-580">Don't register <xref:System.IDisposable> instances with a Transient scope.</span></span> <span data-ttu-id="eb861-581">Usare invece il modello Factory.</span><span class="sxs-lookup"><span data-stu-id="eb861-581">Use the factory pattern instead.</span></span>
* <span data-ttu-id="eb861-582">Non risolvere istanze temporanee o <xref:System.IDisposable> con ambito nell'ambito radice.</span><span class="sxs-lookup"><span data-stu-id="eb861-582">Don't resolve Transient or Scoped <xref:System.IDisposable> instances in the root scope.</span></span> <span data-ttu-id="eb861-583">L'unica eccezione generale si verifica quando l'app crea/ricrea ed Elimina <xref:System.IServiceProvider> , che non è un modello ideale.</span><span class="sxs-lookup"><span data-stu-id="eb861-583">The only general exception is when the app creates/recreates and disposes the <xref:System.IServiceProvider>, which isn't an ideal pattern.</span></span>
* <span data-ttu-id="eb861-584">Per <xref:System.IDisposable> la ricezione di una dipendenza tramite di non è necessario che il ricevitore implementi <xref:System.IDisposable> se stesso.</span><span class="sxs-lookup"><span data-stu-id="eb861-584">Receiving an <xref:System.IDisposable> dependency via DI doesn't require that the receiver implement <xref:System.IDisposable> itself.</span></span> <span data-ttu-id="eb861-585">Il ricevitore della <xref:System.IDisposable> dipendenza non deve chiamare <xref:System.IDisposable.Dispose%2A> su tale dipendenza.</span><span class="sxs-lookup"><span data-stu-id="eb861-585">The receiver of the <xref:System.IDisposable> dependency shouldn't call <xref:System.IDisposable.Dispose%2A> on that dependency.</span></span>
* <span data-ttu-id="eb861-586">Gli ambiti devono essere usati per controllare la durata dei servizi.</span><span class="sxs-lookup"><span data-stu-id="eb861-586">Scopes should be used to control lifetimes of services.</span></span> <span data-ttu-id="eb861-587">Gli ambiti non sono gerarchici e non esiste alcuna connessione speciale tra gli ambiti.</span><span class="sxs-lookup"><span data-stu-id="eb861-587">Scopes aren't hierarchical, and there's no special connection among scopes.</span></span>

## <a name="default-service-container-replacement"></a><span data-ttu-id="eb861-588">Sostituzione del contenitore di servizi predefinito</span><span class="sxs-lookup"><span data-stu-id="eb861-588">Default service container replacement</span></span>

<span data-ttu-id="eb861-589">Il contenitore dei servizi incorporato è progettato per soddisfare le esigenze del Framework e della maggior parte delle app consumer.</span><span class="sxs-lookup"><span data-stu-id="eb861-589">The built-in service container is designed to serve the needs of the framework and most consumer apps.</span></span> <span data-ttu-id="eb861-590">Si consiglia di usare il contenitore predefinito a meno che non sia necessaria una funzionalità specifica che il contenitore incorporato non supporta, ad esempio:</span><span class="sxs-lookup"><span data-stu-id="eb861-590">We recommend using the built-in container unless you need a specific feature that the built-in container doesn't support, such as:</span></span>

* <span data-ttu-id="eb861-591">Inserimento di proprietà</span><span class="sxs-lookup"><span data-stu-id="eb861-591">Property injection</span></span>
* <span data-ttu-id="eb861-592">Inserimento in base al nome</span><span class="sxs-lookup"><span data-stu-id="eb861-592">Injection based on name</span></span>
* <span data-ttu-id="eb861-593">Contenitori figlio</span><span class="sxs-lookup"><span data-stu-id="eb861-593">Child containers</span></span>
* <span data-ttu-id="eb861-594">Gestione della durata personalizzata</span><span class="sxs-lookup"><span data-stu-id="eb861-594">Custom lifetime management</span></span>
* <span data-ttu-id="eb861-595">Supporto di `Func<T>` per l'inizializzazione differita</span><span class="sxs-lookup"><span data-stu-id="eb861-595">`Func<T>` support for lazy initialization</span></span>
* <span data-ttu-id="eb861-596">Registrazione basata sulle convenzioni</span><span class="sxs-lookup"><span data-stu-id="eb861-596">Convention-based registration</span></span>

<span data-ttu-id="eb861-597">Con ASP.NET Core app è possibile usare i contenitori di terze parti seguenti:</span><span class="sxs-lookup"><span data-stu-id="eb861-597">The following third-party containers can be used with ASP.NET Core apps:</span></span>

* [<span data-ttu-id="eb861-598">Autofac</span><span class="sxs-lookup"><span data-stu-id="eb861-598">Autofac</span></span>](https://autofac.readthedocs.io/en/latest/integration/aspnetcore.html)
* [<span data-ttu-id="eb861-599">DryIoc</span><span class="sxs-lookup"><span data-stu-id="eb861-599">DryIoc</span></span>](https://www.nuget.org/packages/DryIoc.Microsoft.DependencyInjection)
* [<span data-ttu-id="eb861-600">Laura</span><span class="sxs-lookup"><span data-stu-id="eb861-600">Grace</span></span>](https://www.nuget.org/packages/Grace.DependencyInjection.Extensions)
* [<span data-ttu-id="eb861-601">LightInject</span><span class="sxs-lookup"><span data-stu-id="eb861-601">LightInject</span></span>](https://github.com/seesharper/LightInject.Microsoft.DependencyInjection)
* [<span data-ttu-id="eb861-602">Lamar</span><span class="sxs-lookup"><span data-stu-id="eb861-602">Lamar</span></span>](https://jasperfx.github.io/lamar/)
* [<span data-ttu-id="eb861-603">Stashbox</span><span class="sxs-lookup"><span data-stu-id="eb861-603">Stashbox</span></span>](https://github.com/z4kn4fein/stashbox-extensions-dependencyinjection)
* [<span data-ttu-id="eb861-604">Unity</span><span class="sxs-lookup"><span data-stu-id="eb861-604">Unity</span></span>](https://www.nuget.org/packages/Unity.Microsoft.DependencyInjection)

### <a name="thread-safety"></a><span data-ttu-id="eb861-605">Thread safety</span><span class="sxs-lookup"><span data-stu-id="eb861-605">Thread safety</span></span>

<span data-ttu-id="eb861-606">Creare servizi singleton thread-safe.</span><span class="sxs-lookup"><span data-stu-id="eb861-606">Create thread-safe singleton services.</span></span> <span data-ttu-id="eb861-607">Se un servizio singleton ha una dipendenza in un servizio temporaneo, è necessario che anche il servizio temporaneo sia thread-safe, a seconda di come viene usato dal singleton.</span><span class="sxs-lookup"><span data-stu-id="eb861-607">If a singleton service has a dependency on a transient service, the transient service may also require thread safety depending how it's used by the singleton.</span></span>

<span data-ttu-id="eb861-608">Il metodo factory di un singolo servizio, ad esempio il secondo argomento di [AddSingleton \<TService> (IServiceCollection, Func \<IServiceProvider,TService> )](xref:Microsoft.Extensions.DependencyInjection.ServiceCollectionServiceExtensions.AddSingleton*), non deve essere thread-safe.</span><span class="sxs-lookup"><span data-stu-id="eb861-608">The factory method of single service, such as the second argument to [AddSingleton\<TService>(IServiceCollection, Func\<IServiceProvider,TService>)](xref:Microsoft.Extensions.DependencyInjection.ServiceCollectionServiceExtensions.AddSingleton*), doesn't need to be thread-safe.</span></span> <span data-ttu-id="eb861-609">Come nel caso di un costruttore di tipo (`static`), è garantito che venga chiamato una sola volta da un singolo thread.</span><span class="sxs-lookup"><span data-stu-id="eb861-609">Like a type (`static`) constructor, it's guaranteed to be called once by a single thread.</span></span>

## <a name="recommendations"></a><span data-ttu-id="eb861-610">Consigli</span><span class="sxs-lookup"><span data-stu-id="eb861-610">Recommendations</span></span>

* <span data-ttu-id="eb861-611">La risoluzione basata sui servizi `async/await` e `Task` non è supportata.</span><span class="sxs-lookup"><span data-stu-id="eb861-611">`async/await` and `Task` based service resolution is not supported.</span></span> <span data-ttu-id="eb861-612">Dato che C# non supporta i costruttori asincroni, il modello consigliato consiste nell'usare i metodi asincroni dopo avere risolto in modo sincrono il servizio.</span><span class="sxs-lookup"><span data-stu-id="eb861-612">C# does not support asynchronous constructors; therefore, the recommended pattern is to use asynchronous methods after synchronously resolving the service.</span></span>

* <span data-ttu-id="eb861-613">Evitare di archiviare i dati e la configurazione direttamente nel contenitore del servizio.</span><span class="sxs-lookup"><span data-stu-id="eb861-613">Avoid storing data and configuration directly in the service container.</span></span> <span data-ttu-id="eb861-614">Ad esempio, il carrello acquisti di un utente non dovrebbe in genere essere aggiunto al contenitore del servizio.</span><span class="sxs-lookup"><span data-stu-id="eb861-614">For example, a user's shopping cart shouldn't typically be added to the service container.</span></span> <span data-ttu-id="eb861-615">La configurazione deve usare il [modello di opzioni](xref:fundamentals/configuration/options).</span><span class="sxs-lookup"><span data-stu-id="eb861-615">Configuration should use the [options pattern](xref:fundamentals/configuration/options).</span></span> <span data-ttu-id="eb861-616">Analogamente, evitare gli oggetti "contenitori di dati" che hanno la sola funzione di consentire l'accesso ad altri oggetti.</span><span class="sxs-lookup"><span data-stu-id="eb861-616">Similarly, avoid "data holder" objects that only exist to allow access to some other object.</span></span> <span data-ttu-id="eb861-617">È preferibile richiedere l'elemento effettivo tramite inserimento delle dipendenze.</span><span class="sxs-lookup"><span data-stu-id="eb861-617">It's better to request the actual item via DI.</span></span>
* <span data-ttu-id="eb861-618">Evitare l'accesso statico ai servizi.</span><span class="sxs-lookup"><span data-stu-id="eb861-618">Avoid static access to services.</span></span> <span data-ttu-id="eb861-619">Ad esempio, evitare di digitare in modo statico [IApplicationBuilder. ApplicationServices](xref:Microsoft.AspNetCore.Builder.IApplicationBuilder.ApplicationServices) per l'uso altrove.</span><span class="sxs-lookup"><span data-stu-id="eb861-619">For example, avoid statically-typing [IApplicationBuilder.ApplicationServices](xref:Microsoft.AspNetCore.Builder.IApplicationBuilder.ApplicationServices) for use elsewhere.</span></span>

* <span data-ttu-id="eb861-620">Evitare di usare il *modello del localizzatore di servizi*, che combina [inversione delle strategie di controllo](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#dependency-inversion) .</span><span class="sxs-lookup"><span data-stu-id="eb861-620">Avoid using the *service locator pattern*, which mixes [Inversion of Control](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#dependency-inversion) strategies.</span></span>
  * <span data-ttu-id="eb861-621">Non richiamare <xref:System.IServiceProvider.GetService*> per ottenere un'istanza del servizio quando si può usare invece di:</span><span class="sxs-lookup"><span data-stu-id="eb861-621">Don't invoke <xref:System.IServiceProvider.GetService*> to obtain a service instance when you can use DI instead:</span></span>

    <span data-ttu-id="eb861-622">**Non corretto:**</span><span class="sxs-lookup"><span data-stu-id="eb861-622">**Incorrect:**</span></span>

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
   
    <span data-ttu-id="eb861-623">**Corretto**:</span><span class="sxs-lookup"><span data-stu-id="eb861-623">**Correct**:</span></span>

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

* <span data-ttu-id="eb861-624">Evitare di inserire una factory che risolve le dipendenze in fase di esecuzione usando <xref:System.IServiceProvider.GetService*> .</span><span class="sxs-lookup"><span data-stu-id="eb861-624">Avoid injecting a factory that resolves dependencies at runtime using <xref:System.IServiceProvider.GetService*>.</span></span>
* <span data-ttu-id="eb861-625">Evitare l'accesso statico a `HttpContext` (ad esempio [IHttpContextAccessor.HttpContext](xref:Microsoft.AspNetCore.Http.IHttpContextAccessor.HttpContext)).</span><span class="sxs-lookup"><span data-stu-id="eb861-625">Avoid static access to `HttpContext` (for example, [IHttpContextAccessor.HttpContext](xref:Microsoft.AspNetCore.Http.IHttpContextAccessor.HttpContext)).</span></span>

<span data-ttu-id="eb861-626">È tuttavia possibile che in alcuni casi queste raccomandazioni debbano essere ignorate.</span><span class="sxs-lookup"><span data-stu-id="eb861-626">Like all sets of recommendations, you may encounter situations where ignoring a recommendation is required.</span></span> <span data-ttu-id="eb861-627">Le eccezioni sono rari, soprattutto casi speciali all'interno del Framework stesso.</span><span class="sxs-lookup"><span data-stu-id="eb861-627">Exceptions are rare, mostly special cases within the framework itself.</span></span>

<span data-ttu-id="eb861-628">L'inserimento di dipendenze è un'*alternativa* ai modelli di accesso agli oggetti statici/globali.</span><span class="sxs-lookup"><span data-stu-id="eb861-628">DI is an *alternative* to static/global object access patterns.</span></span> <span data-ttu-id="eb861-629">Se l'inserimento di dipendenze viene usato con l'accesso agli oggetti statico i vantaggi non saranno evidenti.</span><span class="sxs-lookup"><span data-stu-id="eb861-629">You may not be able to realize the benefits of DI if you mix it with static object access.</span></span>

## <a name="additional-resources"></a><span data-ttu-id="eb861-630">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="eb861-630">Additional resources</span></span>

* <xref:mvc/views/dependency-injection>
* <xref:mvc/controllers/dependency-injection>
* <xref:security/authorization/dependencyinjection>
* <xref:blazor/fundamentals/dependency-injection>
* <xref:fundamentals/startup>
* <xref:fundamentals/middleware/extensibility>
* [<span data-ttu-id="eb861-631">Quattro modi per eliminare IDisposable in ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="eb861-631">Four ways to dispose IDisposables in ASP.NET Core</span></span>](https://andrewlock.net/four-ways-to-dispose-idisposables-in-asp-net-core/)
* [<span data-ttu-id="eb861-632">Scrittura di codice pulito in ASP.NET Core con inserimento delle dipendenze (MSDN)</span><span class="sxs-lookup"><span data-stu-id="eb861-632">Writing Clean Code in ASP.NET Core with Dependency Injection (MSDN)</span></span>](/archive/msdn-magazine/2016/may/asp-net-writing-clean-code-in-asp-net-core-with-dependency-injection)
* [<span data-ttu-id="eb861-633">Explicit Dependencies Principle (Principio delle dipendenze esplicite)</span><span class="sxs-lookup"><span data-stu-id="eb861-633">Explicit Dependencies Principle</span></span>](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#explicit-dependencies)
* [<span data-ttu-id="eb861-634">Inversion of Control Containers and the Dependency Injection Pattern (Martin Fowler) (Contenitori di inversione del controllo e modello di inserimento delle dipendenze)</span><span class="sxs-lookup"><span data-stu-id="eb861-634">Inversion of Control Containers and the Dependency Injection Pattern (Martin Fowler)</span></span>](https://www.martinfowler.com/articles/injection.html)
* <span data-ttu-id="eb861-635">[How to register a service with multiple interfaces in ASP.NET Core DI](https://andrewlock.net/how-to-register-a-service-with-multiple-interfaces-for-in-asp-net-core-di/) (Come registrare un servizio con più interfacce in ASP.NET Core DI)</span><span class="sxs-lookup"><span data-stu-id="eb861-635">[How to register a service with multiple interfaces in ASP.NET Core DI](https://andrewlock.net/how-to-register-a-service-with-multiple-interfaces-for-in-asp-net-core-di/)</span></span>

::: moniker-end
