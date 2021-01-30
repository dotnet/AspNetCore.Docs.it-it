---
title: Panoramica di ASP.NET MVC
author: ardalis
description: Informazioni sul framework avanzato di ASP.NET Core MVC per la creazione di app Web e API tramite lo schema progettuale MVC (Model-View-Controller).
ms.author: riande
ms.date: 02/12/2020
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
uid: mvc/overview
ms.openlocfilehash: 642a7b26a3ba0a79a35437a5c19ee0be63a367af
ms.sourcegitcommit: 83524f739dd25fbfa95ee34e95342afb383b49fe
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 01/29/2021
ms.locfileid: "99057317"
---
# <a name="overview-of-aspnet-core-mvc"></a><span data-ttu-id="1eb44-103">Panoramica di ASP.NET MVC</span><span class="sxs-lookup"><span data-stu-id="1eb44-103">Overview of ASP.NET Core MVC</span></span>

<span data-ttu-id="1eb44-104">Di [Steve Smith](https://ardalis.com/)</span><span class="sxs-lookup"><span data-stu-id="1eb44-104">By [Steve Smith](https://ardalis.com/)</span></span>

<span data-ttu-id="1eb44-105">ASP.NET Core MVC è un framework avanzato per la creazione di app Web e API tramite lo schema progettuale MVC (Model-View-Controller).</span><span class="sxs-lookup"><span data-stu-id="1eb44-105">ASP.NET Core MVC is a rich framework for building web apps and APIs using the Model-View-Controller design pattern.</span></span>

## <a name="what-is-the-mvc-pattern"></a><span data-ttu-id="1eb44-106">Cos'è lo schema MVC?</span><span class="sxs-lookup"><span data-stu-id="1eb44-106">What is the MVC pattern?</span></span>

<span data-ttu-id="1eb44-107">Nello schema architetturale MVC (Model-View-Controller) l'applicazione viene suddivisa in tre gruppi principali di componenti: modelli, visualizzazioni e controller.</span><span class="sxs-lookup"><span data-stu-id="1eb44-107">The Model-View-Controller (MVC) architectural pattern separates an application into three main groups of components: Models, Views, and Controllers.</span></span> <span data-ttu-id="1eb44-108">Questo schema consente di realizzare la [separazione delle competenze](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#separation-of-concerns).</span><span class="sxs-lookup"><span data-stu-id="1eb44-108">This pattern helps to achieve [separation of concerns](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#separation-of-concerns).</span></span> <span data-ttu-id="1eb44-109">Grazie all'uso di questo schema, le richieste dell'utente vengono indirizzate a un controller responsabile di interagire con il modello per eseguire le azioni dell'utente e/o recuperare i risultati delle query.</span><span class="sxs-lookup"><span data-stu-id="1eb44-109">Using this pattern, user requests are routed to a Controller which is responsible for working with the Model to perform user actions and/or retrieve results of queries.</span></span> <span data-ttu-id="1eb44-110">Il controller sceglie la visualizzazione da mostrare all'utente e le fornisce i dati del modello necessari.</span><span class="sxs-lookup"><span data-stu-id="1eb44-110">The Controller chooses the View to display to the user, and provides it with any Model data it requires.</span></span>

<span data-ttu-id="1eb44-111">Il diagramma seguente illustra i tre componenti principali e quali fanno riferimento agli altri:</span><span class="sxs-lookup"><span data-stu-id="1eb44-111">The following diagram shows the three main components and which ones reference the others:</span></span>

![Schema MVC](overview/_static/mvc.png)

<span data-ttu-id="1eb44-113">Questa definizione delle responsabilità consente di ridimensionare l'applicazione in termini di complessità, perché è più facile scrivere codice, eseguire il debug e testare qualcosa (modello, visualizzazione o controller) che presenta un unico processo.</span><span class="sxs-lookup"><span data-stu-id="1eb44-113">This delineation of responsibilities helps you scale the application in terms of complexity because it's easier to code, debug, and test something (model, view, or controller) that has a single job.</span></span> <span data-ttu-id="1eb44-114">È più difficile aggiornare, testare ed eseguire il debug di codice con dipendenze distribuite in due o più di queste tre aree.</span><span class="sxs-lookup"><span data-stu-id="1eb44-114">It's more difficult to update, test, and debug code that has dependencies spread across two or more of these three areas.</span></span> <span data-ttu-id="1eb44-115">La logica dell'interfaccia utente, ad esempio, tende a cambiare più frequentemente rispetto alla logica di business.</span><span class="sxs-lookup"><span data-stu-id="1eb44-115">For example, user interface logic tends to change more frequently than business logic.</span></span> <span data-ttu-id="1eb44-116">Se il codice di presentazione e la logica di business sono combinati in un singolo oggetto, l'oggetto che contiene la logica di business deve essere modificato ogni volta che l'interfaccia utente viene modificata.</span><span class="sxs-lookup"><span data-stu-id="1eb44-116">If presentation code and business logic are combined in a single object, an object containing business logic must be modified every time the user interface is changed.</span></span> <span data-ttu-id="1eb44-117">Ciò spesso introduce errori e rende necessaria la ripetizione dei test della logica di business dopo ogni minima modifica dell'interfaccia utente.</span><span class="sxs-lookup"><span data-stu-id="1eb44-117">This often introduces errors and requires the retesting of business logic after every minimal user interface change.</span></span>

> [!NOTE]
> <span data-ttu-id="1eb44-118">La visualizzazione e il controller dipendono dal modello.</span><span class="sxs-lookup"><span data-stu-id="1eb44-118">Both the view and the controller depend on the model.</span></span> <span data-ttu-id="1eb44-119">Il modello, tuttavia, non dipende né dalla visualizzazione né dal controller.</span><span class="sxs-lookup"><span data-stu-id="1eb44-119">However, the model depends on neither the view nor the controller.</span></span> <span data-ttu-id="1eb44-120">Questo è uno dei principali vantaggi della separazione.</span><span class="sxs-lookup"><span data-stu-id="1eb44-120">This is one of the key benefits of the separation.</span></span> <span data-ttu-id="1eb44-121">La separazione consente di compilare e testare il modello in modo indipendente dalla presentazione visiva.</span><span class="sxs-lookup"><span data-stu-id="1eb44-121">This separation allows the model to be built and tested independent of the visual presentation.</span></span>

### <a name="model-responsibilities"></a><span data-ttu-id="1eb44-122">Responsabilità del modello</span><span class="sxs-lookup"><span data-stu-id="1eb44-122">Model Responsibilities</span></span>

<span data-ttu-id="1eb44-123">In un'applicazione MVC il modello rappresenta lo stato dell'applicazione e la logica di business o le operazioni che essa deve eseguire.</span><span class="sxs-lookup"><span data-stu-id="1eb44-123">The Model in an MVC application represents the state of the application and any business logic or operations that should be performed by it.</span></span> <span data-ttu-id="1eb44-124">La logica di business deve essere incapsulata nel modello insieme alla logica di implementazione per rendere persistente lo stato dell'applicazione.</span><span class="sxs-lookup"><span data-stu-id="1eb44-124">Business logic should be encapsulated in the model, along with any implementation logic for persisting the state of the application.</span></span> <span data-ttu-id="1eb44-125">Le visualizzazioni fortemente tipizzate usano in genere tipi ViewModel progettati per contenere i dati da visualizzare in una particolare visualizzazione.</span><span class="sxs-lookup"><span data-stu-id="1eb44-125">Strongly-typed views typically use ViewModel types designed to contain the data to display on that view.</span></span> <span data-ttu-id="1eb44-126">Il controller crea e popola queste istanze di ViewModel dal modello.</span><span class="sxs-lookup"><span data-stu-id="1eb44-126">The controller creates and populates these ViewModel instances from the model.</span></span>

### <a name="view-responsibilities"></a><span data-ttu-id="1eb44-127">Responsabilità della visualizzazione</span><span class="sxs-lookup"><span data-stu-id="1eb44-127">View Responsibilities</span></span>

<span data-ttu-id="1eb44-128">Le visualizzazioni sono responsabili della presentazione del contenuto tramite l'interfaccia utente.</span><span class="sxs-lookup"><span data-stu-id="1eb44-128">Views are responsible for presenting content through the user interface.</span></span> <span data-ttu-id="1eb44-129">Usano il [ Razor motore di visualizzazione](#razor-view-engine) per incorporare il codice .NET nel markup HTML.</span><span class="sxs-lookup"><span data-stu-id="1eb44-129">They use the [Razor view engine](#razor-view-engine) to embed .NET code in HTML markup.</span></span> <span data-ttu-id="1eb44-130">Nelle visualizzazioni la quantità di logica deve essere minima e la logica in esse contenuta deve essere relativa alla presentazione del contenuto.</span><span class="sxs-lookup"><span data-stu-id="1eb44-130">There should be minimal logic within views, and any logic in them should relate to presenting content.</span></span> <span data-ttu-id="1eb44-131">Se è necessario eseguire una grande quantità di logica nei file di visualizzazione per visualizzare dati da un modello complesso, valutare l'uso di un [componente di visualizzazione](views/view-components.md), un ViewModel o un modello di visualizzazione per semplificare la visualizzazione.</span><span class="sxs-lookup"><span data-stu-id="1eb44-131">If you find the need to perform a great deal of logic in view files in order to display data from a complex model, consider using a [View Component](views/view-components.md), ViewModel, or view template to simplify the view.</span></span>

### <a name="controller-responsibilities"></a><span data-ttu-id="1eb44-132">Responsabilità del controller</span><span class="sxs-lookup"><span data-stu-id="1eb44-132">Controller Responsibilities</span></span>

<span data-ttu-id="1eb44-133">I controller sono i componenti che gestiscono l'interazione dell'utente, interagiscono con il modello e selezionano in definitiva la visualizzazione di cui verrà eseguito il rendering.</span><span class="sxs-lookup"><span data-stu-id="1eb44-133">Controllers are the components that handle user interaction, work with the model, and ultimately select a view to render.</span></span> <span data-ttu-id="1eb44-134">In un'applicazione MVC la visualizzazione presenta solo le informazioni, mentre il controller gestisce e risponde all'input e all'interazione dell'utente.</span><span class="sxs-lookup"><span data-stu-id="1eb44-134">In an MVC application, the view only displays information; the controller handles and responds to user input and interaction.</span></span> <span data-ttu-id="1eb44-135">Nello schema MVC il controller è il punto di ingresso iniziale ed è responsabile della selezione dei tipi di modello con cui interagire e della visualizzazione di cui eseguire il rendering. Come suggerito dal nome, questo componente controlla il modo in cui l'app risponde a una determinata richiesta.</span><span class="sxs-lookup"><span data-stu-id="1eb44-135">In the MVC pattern, the controller is the initial entry point, and is responsible for selecting which model types to work with and which view to render (hence its name - it controls how the app responds to a given request).</span></span>

> [!NOTE]
> <span data-ttu-id="1eb44-136">È consigliabile non sovraccaricare eccessivamente i controller con troppe responsabilità.</span><span class="sxs-lookup"><span data-stu-id="1eb44-136">Controllers shouldn't be overly complicated by too many responsibilities.</span></span> <span data-ttu-id="1eb44-137">Per evitare che la logica del controller diventi troppo complessa, escludere la logica di business dal controller e includerla nel modello di dominio.</span><span class="sxs-lookup"><span data-stu-id="1eb44-137">To keep controller logic from becoming overly complex, push business logic out of the controller and into the domain model.</span></span>

>[!TIP]
> <span data-ttu-id="1eb44-138">Se si ritiene che le azioni del controller eseguano frequentemente gli stessi tipi di azioni, spostare queste azioni comuni nei [filtri](#filters).</span><span class="sxs-lookup"><span data-stu-id="1eb44-138">If you find that your controller actions frequently perform the same kinds of actions, move these common actions into [filters](#filters).</span></span>

## <a name="what-is-aspnet-core-mvc"></a><span data-ttu-id="1eb44-139">Cos'è ASP.NET Core MVC</span><span class="sxs-lookup"><span data-stu-id="1eb44-139">What is ASP.NET Core MVC</span></span>

<span data-ttu-id="1eb44-140">Il framework ASP.NET Core MVC è un framework di presentazione leggero, open source e altamente testabile ottimizzato per l'uso con ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="1eb44-140">The ASP.NET Core MVC framework is a lightweight, open source, highly testable presentation framework optimized for use with ASP.NET Core.</span></span>

<span data-ttu-id="1eb44-141">ASP.NET Core MVC offre un sistema basato su schemi per la creazione di siti Web dinamici che consente una netta separazione delle competenze.</span><span class="sxs-lookup"><span data-stu-id="1eb44-141">ASP.NET Core MVC provides a patterns-based way to build dynamic websites that enables a clean separation of concerns.</span></span> <span data-ttu-id="1eb44-142">Offre il controllo completo sul markup, supporta lo sviluppo basato su test e usa gli standard Web più recenti.</span><span class="sxs-lookup"><span data-stu-id="1eb44-142">It gives you full control over markup, supports TDD-friendly development and uses the latest web standards.</span></span>

## <a name="features"></a><span data-ttu-id="1eb44-143">Funzionalità</span><span class="sxs-lookup"><span data-stu-id="1eb44-143">Features</span></span>

<span data-ttu-id="1eb44-144">ASP.NET Core MVC include:</span><span class="sxs-lookup"><span data-stu-id="1eb44-144">ASP.NET Core MVC includes the following:</span></span>

* [<span data-ttu-id="1eb44-145">Routing</span><span class="sxs-lookup"><span data-stu-id="1eb44-145">Routing</span></span>](#routing)
* [<span data-ttu-id="1eb44-146">Associazione di modelli</span><span class="sxs-lookup"><span data-stu-id="1eb44-146">Model binding</span></span>](#model-binding)
* [<span data-ttu-id="1eb44-147">Convalida modello</span><span class="sxs-lookup"><span data-stu-id="1eb44-147">Model validation</span></span>](#model-validation)
* [<span data-ttu-id="1eb44-148">Inserimento delle dipendenze</span><span class="sxs-lookup"><span data-stu-id="1eb44-148">Dependency injection</span></span>](../fundamentals/dependency-injection.md)
* [<span data-ttu-id="1eb44-149">Filtri</span><span class="sxs-lookup"><span data-stu-id="1eb44-149">Filters</span></span>](#filters)
* [<span data-ttu-id="1eb44-150">Aree</span><span class="sxs-lookup"><span data-stu-id="1eb44-150">Areas</span></span>](#areas)
* [<span data-ttu-id="1eb44-151">API Web</span><span class="sxs-lookup"><span data-stu-id="1eb44-151">Web APIs</span></span>](#web-apis)
* [<span data-ttu-id="1eb44-152">Testabilità</span><span class="sxs-lookup"><span data-stu-id="1eb44-152">Testability</span></span>](#testability)
* [<span data-ttu-id="1eb44-153">Razor motore di visualizzazione</span><span class="sxs-lookup"><span data-stu-id="1eb44-153">Razor view engine</span></span>](#razor-view-engine)
* [<span data-ttu-id="1eb44-154">Visualizzazioni fortemente tipizzate</span><span class="sxs-lookup"><span data-stu-id="1eb44-154">Strongly typed views</span></span>](#strongly-typed-views)
* [<span data-ttu-id="1eb44-155">Helper per i tag</span><span class="sxs-lookup"><span data-stu-id="1eb44-155">Tag Helpers</span></span>](#tag-helpers)
* [<span data-ttu-id="1eb44-156">Visualizza componenti</span><span class="sxs-lookup"><span data-stu-id="1eb44-156">View Components</span></span>](#view-components)

### <a name="routing"></a><span data-ttu-id="1eb44-157">Routing</span><span class="sxs-lookup"><span data-stu-id="1eb44-157">Routing</span></span>

<span data-ttu-id="1eb44-158">ASP.NET Core MVC si basa sul [routing di ASP.NET Core](../fundamentals/routing.md), un potente componente per il mapping di URL che consente di compilare applicazioni con URL comprensibili che supportano la ricerca.</span><span class="sxs-lookup"><span data-stu-id="1eb44-158">ASP.NET Core MVC is built on top of [ASP.NET Core's routing](../fundamentals/routing.md), a powerful URL-mapping component that lets you build applications that have comprehensible and searchable URLs.</span></span> <span data-ttu-id="1eb44-159">Ciò consente di definire criteri di denominazione dell'URL dell'applicazione che funzionano perfettamente per l'ottimizzazione dei motori di ricerca (SEO) e la generazione di collegamenti, indipendentemente da come sono organizzati i file nel server Web.</span><span class="sxs-lookup"><span data-stu-id="1eb44-159">This enables you to define your application's URL naming patterns that work well for search engine optimization (SEO) and for link generation, without regard for how the files on your web server are organized.</span></span> <span data-ttu-id="1eb44-160">È possibile definire le route usando una pratica sintassi del modello di route che supporta i vincoli di valore delle route, i valori predefiniti e quelli facoltativi.</span><span class="sxs-lookup"><span data-stu-id="1eb44-160">You can define your routes using a convenient route template syntax that supports route value constraints, defaults and optional values.</span></span>

<span data-ttu-id="1eb44-161">Il *routing basato su convenzioni* consente di definire a livello globale i formati URL accettati dall'applicazione e il modo in cui ognuno di questi formati viene mappato a un metodo di azione specifico su un determinato controller.</span><span class="sxs-lookup"><span data-stu-id="1eb44-161">*Convention-based routing* enables you to globally define the URL formats that your application accepts and how each of those formats maps to a specific action method on a given controller.</span></span> <span data-ttu-id="1eb44-162">Alla ricezione di una richiesta in ingresso, il motore di routing analizza l'URL e lo confronta con uno dei formati di URL definiti, quindi chiama il metodo di azione del controller associato.</span><span class="sxs-lookup"><span data-stu-id="1eb44-162">When an incoming request is received, the routing engine parses the URL and matches it to one of the defined URL formats, and then calls the associated controller's action method.</span></span>

```csharp
routes.MapRoute(name: "Default", template: "{controller=Home}/{action=Index}/{id?}");
```

<span data-ttu-id="1eb44-163">Il *routing di attributi* consente di specificare informazioni di routing assegnando ai controller e alle azioni attributi che definiscono le route dell'applicazione.</span><span class="sxs-lookup"><span data-stu-id="1eb44-163">*Attribute routing* enables you to specify routing information by decorating your controllers and actions with attributes that define your application's routes.</span></span> <span data-ttu-id="1eb44-164">Ciò significa che le definizioni delle route vengono posizionate accanto al controller e all'azione a cui sono associate.</span><span class="sxs-lookup"><span data-stu-id="1eb44-164">This means that your route definitions are placed next to the controller and action with which they're associated.</span></span>

```csharp
[Route("api/[controller]")]
public class ProductsController : Controller
{
    [HttpGet("{id}")]
    public IActionResult GetProduct(int id)
    {
      ...
    }
}
```

### <a name="model-binding"></a><span data-ttu-id="1eb44-165">Associazione di modelli</span><span class="sxs-lookup"><span data-stu-id="1eb44-165">Model binding</span></span>

<span data-ttu-id="1eb44-166">L'[associazione di modelli](models/model-binding.md) di ASP.NET Core MVC converte i dati delle richieste client, ad esempio valori del modulo, dati della route, parametri della stringa di query, intestazioni HTTP, in oggetti che il controller è in grado di gestire.</span><span class="sxs-lookup"><span data-stu-id="1eb44-166">ASP.NET Core MVC [model binding](models/model-binding.md) converts client request data  (form values, route data, query string parameters, HTTP headers) into objects that the controller can handle.</span></span> <span data-ttu-id="1eb44-167">Di conseguenza, non è necessario che la logica del controller risolva i dati della richiesta in ingresso poiché li avrà semplicemente come parametri associati ai propri metodi di azione.</span><span class="sxs-lookup"><span data-stu-id="1eb44-167">As a result, your controller logic doesn't have to do the work of figuring out the incoming request data; it simply has the data as parameters to its action methods.</span></span>

```csharp
public async Task<IActionResult> Login(LoginViewModel model, string returnUrl = null) { ... }
```

### <a name="model-validation"></a><span data-ttu-id="1eb44-168">Convalida modello</span><span class="sxs-lookup"><span data-stu-id="1eb44-168">Model validation</span></span>

<span data-ttu-id="1eb44-169">ASP.NET Core MVC supporta la [convalida](models/validation.md) assegnando all'oggetto modello attributi di convalida di annotazione dei dati.</span><span class="sxs-lookup"><span data-stu-id="1eb44-169">ASP.NET Core MVC supports [validation](models/validation.md) by decorating your model object with data annotation validation attributes.</span></span> <span data-ttu-id="1eb44-170">Gli attributi di convalida vengono controllati sul lato client prima che i valori siano inviati al server, nonché nel server prima che sia chiamata l'azione del controller.</span><span class="sxs-lookup"><span data-stu-id="1eb44-170">The validation attributes are checked on the client side before values are posted to the server, as well as on the server before the controller action is called.</span></span>

```csharp
using System.ComponentModel.DataAnnotations;
public class LoginViewModel
{
    [Required]
    [EmailAddress]
    public string Email { get; set; }

    [Required]
    [DataType(DataType.Password)]
    public string Password { get; set; }

    [Display(Name = "Remember me?")]
    public bool RememberMe { get; set; }
}
```

<span data-ttu-id="1eb44-171">Un'azione del controller:</span><span class="sxs-lookup"><span data-stu-id="1eb44-171">A controller action:</span></span>

```csharp
public async Task<IActionResult> Login(LoginViewModel model, string returnUrl = null)
{
    if (ModelState.IsValid)
    {
      // work with the model
    }
    // At this point, something failed, redisplay form
    return View(model);
}
```

<span data-ttu-id="1eb44-172">Il framework gestisce la convalida dei dati della richiesta nel client e nel server.</span><span class="sxs-lookup"><span data-stu-id="1eb44-172">The framework handles validating request data both on the client and on the server.</span></span> <span data-ttu-id="1eb44-173">La logica di convalida specificata nei tipi di modello viene aggiunta alle visualizzazioni sottoposte a rendering come annotazioni discrete e viene applicata al browser con [jQuery Validation](https://jqueryvalidation.org/).</span><span class="sxs-lookup"><span data-stu-id="1eb44-173">Validation logic specified on model types is added to the rendered views as unobtrusive annotations and is enforced in the browser with [jQuery Validation](https://jqueryvalidation.org/).</span></span>

### <a name="dependency-injection"></a><span data-ttu-id="1eb44-174">Inserimento di dipendenze</span><span class="sxs-lookup"><span data-stu-id="1eb44-174">Dependency injection</span></span>

<span data-ttu-id="1eb44-175">ASP.NET Core include il supporto predefinito per l'[inserimento di dipendenze](../fundamentals/dependency-injection.md).</span><span class="sxs-lookup"><span data-stu-id="1eb44-175">ASP.NET Core has built-in support for [dependency injection (DI)](../fundamentals/dependency-injection.md).</span></span> <span data-ttu-id="1eb44-176">In ASP.NET Core MVC i [controller](controllers/dependency-injection.md) possono richiedere i servizi necessari attraverso i propri costruttori. Ciò consente loro di seguire il [principio delle dipendenze esplicite](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#explicit-dependencies).</span><span class="sxs-lookup"><span data-stu-id="1eb44-176">In ASP.NET Core MVC, [controllers](controllers/dependency-injection.md) can request needed services through their constructors, allowing them to follow the [Explicit Dependencies Principle](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#explicit-dependencies).</span></span>

<span data-ttu-id="1eb44-177">L'app può inoltre usare l'[inserimento di dipendenze nei file di visualizzazione](views/dependency-injection.md) tramite la direttiva `@inject`:</span><span class="sxs-lookup"><span data-stu-id="1eb44-177">Your app can also use [dependency injection in view files](views/dependency-injection.md), using the `@inject` directive:</span></span>

```cshtml
@inject SomeService ServiceName

<!DOCTYPE html>
<html lang="en">
<head>
    <title>@ServiceName.GetTitle</title>
</head>
<body>
    <h1>@ServiceName.GetTitle</h1>
</body>
</html>
```

### <a name="filters"></a><span data-ttu-id="1eb44-178">Filtri</span><span class="sxs-lookup"><span data-stu-id="1eb44-178">Filters</span></span>

<span data-ttu-id="1eb44-179">I [filtri](controllers/filters.md) consentono agli sviluppatori di incapsulare questioni trasversali quali la gestione delle eccezioni o l'autorizzazione.</span><span class="sxs-lookup"><span data-stu-id="1eb44-179">[Filters](controllers/filters.md) help developers encapsulate cross-cutting concerns, like exception handling or authorization.</span></span> <span data-ttu-id="1eb44-180">I filtri consentono l'esecuzione di logica pre-elaborazione e post-elaborazione personalizzata per i metodi di azione e possono essere configurati per l'esecuzione in determinate fasi della pipeline di esecuzione per una richiesta specifica.</span><span class="sxs-lookup"><span data-stu-id="1eb44-180">Filters enable running custom pre- and post-processing logic for action methods, and can be configured to run at certain points within the execution pipeline for a given request.</span></span> <span data-ttu-id="1eb44-181">È possibile applicare i filtri ai controller o alle azioni come attributi o eseguirli a livello globale.</span><span class="sxs-lookup"><span data-stu-id="1eb44-181">Filters can be applied to controllers or actions as attributes (or can be run globally).</span></span> <span data-ttu-id="1eb44-182">Il framework include diversi filtri, ad esempio, `Authorize`.</span><span class="sxs-lookup"><span data-stu-id="1eb44-182">Several filters (such as `Authorize`) are included in the framework.</span></span> <span data-ttu-id="1eb44-183">`[Authorize]` è l'attributo usato per creare filtri di autorizzazione MVC.</span><span class="sxs-lookup"><span data-stu-id="1eb44-183">`[Authorize]` is the attribute that is used to create MVC authorization filters.</span></span>

```csharp
[Authorize]
public class AccountController : Controller
```

### <a name="areas"></a><span data-ttu-id="1eb44-184">Aree</span><span class="sxs-lookup"><span data-stu-id="1eb44-184">Areas</span></span>

<span data-ttu-id="1eb44-185">Le [aree](controllers/areas.md) consentono di partizionare un'app Web ASP.NET Core MVC di grandi dimensioni in raggruppamenti funzionali più piccoli.</span><span class="sxs-lookup"><span data-stu-id="1eb44-185">[Areas](controllers/areas.md) provide a way to partition a large ASP.NET Core MVC Web app into smaller functional groupings.</span></span> <span data-ttu-id="1eb44-186">Un'area è una struttura MVC all'interno di un'applicazione.</span><span class="sxs-lookup"><span data-stu-id="1eb44-186">An area is an MVC structure inside an application.</span></span> <span data-ttu-id="1eb44-187">In un progetto MVC i componenti logici come modello, controller e visualizzazione si trovano in cartelle diverse e MVC usa le convenzioni di denominazione per creare la relazione tra questi componenti.</span><span class="sxs-lookup"><span data-stu-id="1eb44-187">In an MVC project, logical components like Model, Controller, and View are kept in different folders, and MVC uses naming conventions to create the relationship between these components.</span></span> <span data-ttu-id="1eb44-188">Per un'app di grandi dimensioni può risultare utile suddividere l'app in aree di funzionalità di alto livello distinte.</span><span class="sxs-lookup"><span data-stu-id="1eb44-188">For a large app, it may be advantageous to partition the app into separate high level areas of functionality.</span></span> <span data-ttu-id="1eb44-189">Ad esempio, un'app di e-commerce con più business unit, ad esempio checkout, fatturazione e ricerca e così via. Ognuna di queste unità dispone di visualizzazioni, controller e modelli di componenti logici.</span><span class="sxs-lookup"><span data-stu-id="1eb44-189">For instance, an e-commerce app with multiple business units, such as checkout, billing, and search etc. Each of these units have their own logical component views, controllers, and models.</span></span>

### <a name="web-apis"></a><span data-ttu-id="1eb44-190">API Web</span><span class="sxs-lookup"><span data-stu-id="1eb44-190">Web APIs</span></span>

<span data-ttu-id="1eb44-191">Oltre a essere una piattaforma ideale per la creazione di siti Web, ASP.NET Core MVC include un ottimo supporto per la creazione di API Web.</span><span class="sxs-lookup"><span data-stu-id="1eb44-191">In addition to being a great platform for building web sites, ASP.NET Core MVC has great support for building Web APIs.</span></span> <span data-ttu-id="1eb44-192">È possibile creare servizi destinati a un'ampia gamma di client, tra cui browser e dispositivi mobili.</span><span class="sxs-lookup"><span data-stu-id="1eb44-192">You can build services that reach a broad range of clients including browsers and mobile devices.</span></span>

<span data-ttu-id="1eb44-193">Il framework include il supporto per la negoziazione del contenuto HTTP con il supporto predefinito per la [formattazione dei dati](xref:web-api/advanced/formatting) come JSON o XML.</span><span class="sxs-lookup"><span data-stu-id="1eb44-193">The framework includes support for HTTP content-negotiation with built-in support to [format data](xref:web-api/advanced/formatting) as JSON or XML.</span></span> <span data-ttu-id="1eb44-194">È possibile scrivere [formattatori personalizzati](xref:web-api/advanced/custom-formatters) per aggiungere il supporto per i propri formati.</span><span class="sxs-lookup"><span data-stu-id="1eb44-194">Write [custom formatters](xref:web-api/advanced/custom-formatters) to add support for your own formats.</span></span>

<span data-ttu-id="1eb44-195">Usare la generazione di collegamenti per abilitare il supporto per l'ipermedia.</span><span class="sxs-lookup"><span data-stu-id="1eb44-195">Use link generation to enable support for hypermedia.</span></span> <span data-ttu-id="1eb44-196">È possibile abilitare facilmente il supporto per la [Condivisione di risorse tra le origini (CORS)](https://www.w3.org/TR/cors/) in modo da poter condividere le proprie API Web tra più applicazioni Web.</span><span class="sxs-lookup"><span data-stu-id="1eb44-196">Easily enable support for [cross-origin resource sharing (CORS)](https://www.w3.org/TR/cors/) so that your Web APIs can be shared across multiple Web applications.</span></span>

### <a name="testability"></a><span data-ttu-id="1eb44-197">Testabilità</span><span class="sxs-lookup"><span data-stu-id="1eb44-197">Testability</span></span>

<span data-ttu-id="1eb44-198">L'uso delle interfacce e dell'inserimento di dipendenze rende il framework adatto al testing unità. Il framework include inoltre funzionalità come TestHost e il provider InMemory per Entity Framework grazie alle quali i [test di integrazione](xref:test/integration-tests) risultano semplici e veloci.</span><span class="sxs-lookup"><span data-stu-id="1eb44-198">The framework's use of interfaces and dependency injection make it well-suited to unit testing, and the framework includes features (like a TestHost and InMemory provider for Entity Framework) that make [integration tests](xref:test/integration-tests) quick and easy as well.</span></span> <span data-ttu-id="1eb44-199">Per altre informazioni, vedere [Test della logica dei controller](controllers/testing.md).</span><span class="sxs-lookup"><span data-stu-id="1eb44-199">Learn more about [how to test controller logic](controllers/testing.md).</span></span>

### <a name="no-locrazor-view-engine"></a><span data-ttu-id="1eb44-200">Razor motore di visualizzazione</span><span class="sxs-lookup"><span data-stu-id="1eb44-200">Razor view engine</span></span>

<span data-ttu-id="1eb44-201">[ASP.NET Core visualizzazioni MVC](views/overview.md) usano il [ Razor motore di visualizzazione](views/razor.md) per eseguire il rendering delle visualizzazioni.</span><span class="sxs-lookup"><span data-stu-id="1eb44-201">[ASP.NET Core MVC views](views/overview.md) use the [Razor view engine](views/razor.md) to render views.</span></span> <span data-ttu-id="1eb44-202">Razor è un linguaggio di markup modello compatto, espressivo e fluido per la definizione di visualizzazioni con codice C# incorporato.</span><span class="sxs-lookup"><span data-stu-id="1eb44-202">Razor is a compact, expressive and fluid template markup language for defining views using embedded C# code.</span></span> <span data-ttu-id="1eb44-203">Razor viene utilizzato per generare dinamicamente contenuto Web nel server.</span><span class="sxs-lookup"><span data-stu-id="1eb44-203">Razor is used to dynamically generate web content on the server.</span></span> <span data-ttu-id="1eb44-204">È possibile combinare correttamente il codice server con il contenuto e il codice sul lato client.</span><span class="sxs-lookup"><span data-stu-id="1eb44-204">You can cleanly mix server code with client side content and code.</span></span>

```cshtml
<ul>
    @for (int i = 0; i < 5; i++) {
        <li>List item @i</li>
    }
</ul>
```

<span data-ttu-id="1eb44-205">Utilizzando il Razor motore di visualizzazione è possibile definire [layout](views/layout.md), [visualizzazioni parziali](views/partial.md) e sezioni sostituibili.</span><span class="sxs-lookup"><span data-stu-id="1eb44-205">Using the Razor view engine you can define [layouts](views/layout.md), [partial views](views/partial.md) and replaceable sections.</span></span>

### <a name="strongly-typed-views"></a><span data-ttu-id="1eb44-206">Visualizzazioni fortemente tipizzate</span><span class="sxs-lookup"><span data-stu-id="1eb44-206">Strongly typed views</span></span>

<span data-ttu-id="1eb44-207">Razor le visualizzazioni in MVC possono essere fortemente tipizzate in base al modello.</span><span class="sxs-lookup"><span data-stu-id="1eb44-207">Razor views in MVC can be strongly typed based on your model.</span></span> <span data-ttu-id="1eb44-208">I controller sono in grado di passare un modello fortemente tipizzato alle visualizzazioni abilitando per queste ultime il controllo del tipo e il supporto IntelliSense.</span><span class="sxs-lookup"><span data-stu-id="1eb44-208">Controllers can pass a strongly typed model to views enabling your views to have type checking and IntelliSense support.</span></span>

<span data-ttu-id="1eb44-209">La visualizzazione seguente, ad esempio, esegue il rendering di un modello di tipo `IEnumerable<Product>`:</span><span class="sxs-lookup"><span data-stu-id="1eb44-209">For example, the following view renders a model of type `IEnumerable<Product>`:</span></span>

```cshtml
@model IEnumerable<Product>
<ul>
    @foreach (Product p in Model)
    {
        <li>@p.Name</li>
    }
</ul>
```

### <a name="tag-helpers"></a><span data-ttu-id="1eb44-210">Helper tag</span><span class="sxs-lookup"><span data-stu-id="1eb44-210">Tag Helpers</span></span>

<span data-ttu-id="1eb44-211">Gli [Helper Tag](views/tag-helpers/intro.md) consentono al codice lato server di partecipare alla creazione e al rendering di elementi HTML nei Razor file.</span><span class="sxs-lookup"><span data-stu-id="1eb44-211">[Tag Helpers](views/tag-helpers/intro.md) enable server side code to participate in creating and rendering HTML elements in Razor files.</span></span> <span data-ttu-id="1eb44-212">È possibile usare gli helper tag per definire tag personalizzati, ad esempio `<environment>` o per modificare il comportamento di tag esistenti, ad esempio `<label>`.</span><span class="sxs-lookup"><span data-stu-id="1eb44-212">You can use tag helpers to define custom tags (for example, `<environment>`) or to modify the behavior of existing tags (for example, `<label>`).</span></span> <span data-ttu-id="1eb44-213">Gli helper tag vengono associati a elementi specifici in base al nome dell'elemento e ai relativi attributi.</span><span class="sxs-lookup"><span data-stu-id="1eb44-213">Tag Helpers bind to specific elements based on the element name and its attributes.</span></span> <span data-ttu-id="1eb44-214">Offrono i vantaggi del rendering lato server mantenendo al tempo stesso un'esperienza di modifica HTML.</span><span class="sxs-lookup"><span data-stu-id="1eb44-214">They provide the benefits of server-side rendering while still preserving an HTML editing experience.</span></span>

<span data-ttu-id="1eb44-215">Esistono molti helper tag predefiniti per le attività comuni, ad esempio la creazione di moduli e collegamenti, il caricamento di asset e così via, e altri ancora sono disponibili nei repository GitHub pubblici e come pacchetti NuGet.</span><span class="sxs-lookup"><span data-stu-id="1eb44-215">There are many built-in Tag Helpers for common tasks - such as creating forms, links, loading assets and more - and even more available in public GitHub repositories and as NuGet packages.</span></span> <span data-ttu-id="1eb44-216">Gli helper tag vengono creati in C# e hanno come destinazione gli elementi HTML in base al nome di elemento, nome di attributo o tag padre.</span><span class="sxs-lookup"><span data-stu-id="1eb44-216">Tag Helpers are authored in C#, and they target HTML elements based on element name, attribute name, or parent tag.</span></span> <span data-ttu-id="1eb44-217">L'helper tag LinkTagHelper predefinito, ad esempio, può essere usato per creare un collegamento all'azione `Login` di `AccountsController`:</span><span class="sxs-lookup"><span data-stu-id="1eb44-217">For example, the built-in LinkTagHelper can be used to create a link to the `Login` action of the `AccountsController`:</span></span>

```cshtml
<p>
    Thank you for confirming your email.
    Please <a asp-controller="Account" asp-action="Login">Click here to Log in</a>.
</p>
```

<span data-ttu-id="1eb44-218">`EnvironmentTagHelper` può essere usato per includere diversi script nelle visualizzazioni (ad esempio, non elaborate o minimizzate) in base all'ambiente di runtime, ad esempio ambiente sviluppo, di gestione temporanea o di produzione:</span><span class="sxs-lookup"><span data-stu-id="1eb44-218">The `EnvironmentTagHelper` can be used to include different scripts in your views (for example, raw or minified) based on the runtime environment, such as Development, Staging, or Production:</span></span>

```cshtml
<environment names="Development">
    <script src="~/lib/jquery/dist/jquery.js"></script>
</environment>
<environment names="Staging,Production">
    <script src="https://ajax.aspnetcdn.com/ajax/jquery/jquery-2.1.4.min.js"
            asp-fallback-src="~/lib/jquery/dist/jquery.min.js"
            asp-fallback-test="window.jQuery">
    </script>
</environment>
```

<span data-ttu-id="1eb44-219">Gli helper tag forniscono un'esperienza di sviluppo intuitiva e un ambiente IntelliSense avanzato per la creazione di HTML e Razor markup.</span><span class="sxs-lookup"><span data-stu-id="1eb44-219">Tag Helpers provide an HTML-friendly development experience and a rich IntelliSense environment for creating HTML and Razor markup.</span></span> <span data-ttu-id="1eb44-220">La maggior parte degli helper tag predefiniti hanno come destinazione elementi HTML esistenti e forniscono attributi sul lato server per l'elemento.</span><span class="sxs-lookup"><span data-stu-id="1eb44-220">Most of the built-in Tag Helpers target existing HTML elements and provide server-side attributes for the element.</span></span>

### <a name="view-components"></a><span data-ttu-id="1eb44-221">Componenti di visualizzazione</span><span class="sxs-lookup"><span data-stu-id="1eb44-221">View Components</span></span>

<span data-ttu-id="1eb44-222">I [componenti di visualizzazione](views/view-components.md) consentono di creare pacchetti di logica di rendering e di riusarla nell'applicazione.</span><span class="sxs-lookup"><span data-stu-id="1eb44-222">[View Components](views/view-components.md) allow you to package rendering logic and reuse it throughout the application.</span></span> <span data-ttu-id="1eb44-223">Sono simili alle [visualizzazioni parziali](views/partial.md), ma con la logica associata.</span><span class="sxs-lookup"><span data-stu-id="1eb44-223">They're similar to [partial views](views/partial.md), but with associated logic.</span></span>

## <a name="compatibility-version"></a><span data-ttu-id="1eb44-224">Versione di compatibilità</span><span class="sxs-lookup"><span data-stu-id="1eb44-224">Compatibility version</span></span>

<span data-ttu-id="1eb44-225">Il metodo <xref:Microsoft.Extensions.DependencyInjection.MvcCoreMvcBuilderExtensions.SetCompatibilityVersion*> consente a un'app di acconsentire o rifiutare esplicitamente modifiche potenzialmente importanti del comportamento introdotte in ASP.NET Core MVC 2.1 o versioni successive.</span><span class="sxs-lookup"><span data-stu-id="1eb44-225">The <xref:Microsoft.Extensions.DependencyInjection.MvcCoreMvcBuilderExtensions.SetCompatibilityVersion*> method allows an app to opt-in or opt-out of potentially breaking behavior changes introduced in ASP.NET Core MVC 2.1 or later.</span></span>

<span data-ttu-id="1eb44-226">Per altre informazioni, vedere <xref:mvc/compatibility-version>.</span><span class="sxs-lookup"><span data-stu-id="1eb44-226">For more information, see <xref:mvc/compatibility-version>.</span></span>

## <a name="additional-resources"></a><span data-ttu-id="1eb44-227">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="1eb44-227">Additional resources</span></span>

* <span data-ttu-id="1eb44-228">[Tested. AspNetCore. Mvc-libreria di test Fluent per ASP.NET Core MVC](https://github.com/ivaylokenov/MyTested.AspNetCore.Mvc): libreria di unit test fortemente tipizzata, che fornisce un'interfaccia Fluent per il test di app per le API Web e MVC.</span><span class="sxs-lookup"><span data-stu-id="1eb44-228">[MyTested.AspNetCore.Mvc - Fluent Testing Library for ASP.NET Core MVC](https://github.com/ivaylokenov/MyTested.AspNetCore.Mvc): Strongly-typed unit testing library, providing a fluent interface for testing MVC and web API apps.</span></span> <span data-ttu-id="1eb44-229">(*Non mantenuto o supportato da Microsoft).*</span><span class="sxs-lookup"><span data-stu-id="1eb44-229">(*Not maintained or supported by Microsoft.*)</span></span>
* <xref:blazor/components/prerendering-and-integration>
