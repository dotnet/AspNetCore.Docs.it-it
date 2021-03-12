---
title: Inserimento di dipendenze nelle visualizzazioni in ASP.NET Core
author: ardalis
description: Informazioni sulle modalità con cui ASP.NET Core supporta l'inserimento di dipendenze nelle visualizzazioni MVC.
ms.author: riande
ms.date: 10/14/2016
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
uid: mvc/views/dependency-injection
ms.openlocfilehash: 9d4920a7b572163b048bfb06029d913cee3f9e5a
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102587253"
---
# <a name="dependency-injection-into-views-in-aspnet-core"></a><span data-ttu-id="eda06-103">Inserimento di dipendenze nelle visualizzazioni in ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="eda06-103">Dependency injection into views in ASP.NET Core</span></span>

<span data-ttu-id="eda06-104">Di [Steve Smith](https://ardalis.com/)</span><span class="sxs-lookup"><span data-stu-id="eda06-104">By [Steve Smith](https://ardalis.com/)</span></span>

<span data-ttu-id="eda06-105">ASP.NET Core supporta l'[inserimento di dipendenze](xref:fundamentals/dependency-injection) nelle visualizzazioni.</span><span class="sxs-lookup"><span data-stu-id="eda06-105">ASP.NET Core supports [dependency injection](xref:fundamentals/dependency-injection) into views.</span></span> <span data-ttu-id="eda06-106">Questo può essere utile per i servizi specifici delle visualizzazioni, ad esempio per la localizzazione o per dati necessari solo per il popolamento degli elementi delle visualizzazioni.</span><span class="sxs-lookup"><span data-stu-id="eda06-106">This can be useful for view-specific services, such as localization or data required only for populating view elements.</span></span> <span data-ttu-id="eda06-107">È consigliabile mantenere la [separazione delle competenze](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#separation-of-concerns) tra i controller e le visualizzazioni.</span><span class="sxs-lookup"><span data-stu-id="eda06-107">You should try to maintain [separation of concerns](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#separation-of-concerns) between your controllers and views.</span></span> <span data-ttu-id="eda06-108">La maggior parte dei dati nelle visualizzazioni devono essere passati dal controller.</span><span class="sxs-lookup"><span data-stu-id="eda06-108">Most of the data your views display should be passed in from the controller.</span></span>

<span data-ttu-id="eda06-109">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/mvc/views/dependency-injection/sample) ([procedura per il download](xref:index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="eda06-109">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/mvc/views/dependency-injection/sample) ([how to download](xref:index#how-to-download-a-sample))</span></span>

## <a name="configuration-injection"></a><span data-ttu-id="eda06-110">Inserimento della configurazione</span><span class="sxs-lookup"><span data-stu-id="eda06-110">Configuration injection</span></span>

<span data-ttu-id="eda06-111">*appsettings.json* i valori possono essere inseriti direttamente in una visualizzazione.</span><span class="sxs-lookup"><span data-stu-id="eda06-111">*appsettings.json* values can be injected directly into a view.</span></span>

<span data-ttu-id="eda06-112">Esempio di un *appsettings.json* file:</span><span class="sxs-lookup"><span data-stu-id="eda06-112">Example of an *appsettings.json* file:</span></span>

```json
{
   "root": {
      "parent": {
         "child": "myvalue"
      }
   }
}
```

<span data-ttu-id="eda06-113">Sintassi per `@inject`: `@inject <type> <name>`</span><span class="sxs-lookup"><span data-stu-id="eda06-113">The syntax for `@inject`: `@inject <type> <name>`</span></span>

<span data-ttu-id="eda06-114">Esempio che usa `@inject`:</span><span class="sxs-lookup"><span data-stu-id="eda06-114">An example using `@inject`:</span></span>

```csharp
@using Microsoft.Extensions.Configuration
@inject IConfiguration Configuration
@{
   string myValue = Configuration["root:parent:child"];
   ...
}
```

## <a name="service-injection"></a><span data-ttu-id="eda06-115">Inserimento del servizio</span><span class="sxs-lookup"><span data-stu-id="eda06-115">Service injection</span></span>

<span data-ttu-id="eda06-116">È possibile inserire un servizio in una visualizzazione usando la direttiva `@inject`.</span><span class="sxs-lookup"><span data-stu-id="eda06-116">A service can be injected into a view using the `@inject` directive.</span></span> <span data-ttu-id="eda06-117">È possibile considerare `@inject` l'aggiunta di una proprietà alla visualizzazione, con il popolamento della proprietà tramite inserimento delle dipendenze.</span><span class="sxs-lookup"><span data-stu-id="eda06-117">You can think of `@inject` as adding a property to the view, and populating the property using DI.</span></span>

[!code-cshtml[](../../mvc/views/dependency-injection/sample/src/ViewInjectSample/Views/ToDo/Index.cshtml?highlight=4,5,15,16,17)]

<span data-ttu-id="eda06-118">Questa visualizzazione presenta un elenco di istanze di `ToDoItem`, nonché un riepilogo con statistiche generali.</span><span class="sxs-lookup"><span data-stu-id="eda06-118">This view displays a list of `ToDoItem` instances, along with a summary showing overall statistics.</span></span> <span data-ttu-id="eda06-119">Il riepilogo è popolato dal servizio `StatisticsService` inserito.</span><span class="sxs-lookup"><span data-stu-id="eda06-119">The summary is populated from the injected `StatisticsService`.</span></span> <span data-ttu-id="eda06-120">Questo servizio è registrato per l'inserimento di dipendenze in `ConfigureServices` in *Startup.cs*:</span><span class="sxs-lookup"><span data-stu-id="eda06-120">This service is registered for dependency injection in `ConfigureServices` in *Startup.cs*:</span></span>

[!code-csharp[](../../mvc/views/dependency-injection/sample/src/ViewInjectSample/Startup.cs?highlight=6,7&range=15-22)]

<span data-ttu-id="eda06-121">`StatisticsService` esegue alcuni calcoli per il set di istanze di `ToDoItem`, a cui accede tramite un repository:</span><span class="sxs-lookup"><span data-stu-id="eda06-121">The `StatisticsService` performs some calculations on the set of `ToDoItem` instances, which it accesses via a repository:</span></span>

[!code-csharp[](../../mvc/views/dependency-injection/sample/src/ViewInjectSample/Model/Services/StatisticsService.cs?highlight=15,20,25)]

<span data-ttu-id="eda06-122">Il repository di esempio usa una raccolta in memoria.</span><span class="sxs-lookup"><span data-stu-id="eda06-122">The sample repository uses an in-memory collection.</span></span> <span data-ttu-id="eda06-123">L'implementazione illustrata in precedenza, che opera su tutti i dati in memoria, non è consigliata per set di dati di grandi dimensioni a cui si accede in remoto.</span><span class="sxs-lookup"><span data-stu-id="eda06-123">The implementation shown above (which operates on all of the data in memory) isn't recommended for large, remotely accessed data sets.</span></span>

<span data-ttu-id="eda06-124">L'esempio visualizza i dati del modello associato alla visualizzazione e il servizio inserito nella visualizzazione stessa:</span><span class="sxs-lookup"><span data-stu-id="eda06-124">The sample displays data from the model bound to the view and the service injected into the view:</span></span>

![Visualizzazione To Do con l'elenco degli elementi totali, degli elementi completati e della priorità media, e con un elenco di attività con i relativi livelli di priorità e i valori booleani che indicano il completamento.](dependency-injection/_static/screenshot.png)

## <a name="populating-lookup-data"></a><span data-ttu-id="eda06-126">Popolamento di dati di ricerca</span><span class="sxs-lookup"><span data-stu-id="eda06-126">Populating Lookup Data</span></span>

<span data-ttu-id="eda06-127">L'inserimento di visualizzazioni può essere utile per popolare le opzioni negli elementi dell'interfaccia utente, ad esempio negli elenchi a discesa.</span><span class="sxs-lookup"><span data-stu-id="eda06-127">View injection can be useful to populate options in UI elements, such as dropdown lists.</span></span> <span data-ttu-id="eda06-128">Si consideri un modulo di profilo utente che include opzioni che consentono di specificare il sesso, lo stato e altre preferenze.</span><span class="sxs-lookup"><span data-stu-id="eda06-128">Consider a user profile form that includes options for specifying gender, state, and other preferences.</span></span> <span data-ttu-id="eda06-129">Per il rendering di un form di questo tipo tramite un approccio MVC standard sarebbe necessario che il controller richiedesse servizi di accesso ai dati per ognuno di questi set di opzioni e quindi popolasse un modello o un `ViewBag` con ogni set di opzioni da associare.</span><span class="sxs-lookup"><span data-stu-id="eda06-129">Rendering such a form using a standard MVC approach would require the controller to request data access services for each of these sets of options, and then populate a model or `ViewBag` with each set of options to be bound.</span></span>

<span data-ttu-id="eda06-130">Un approccio alternativo consiste nell'ottenere le opzioni inserendo i servizi direttamente nella visualizzazione.</span><span class="sxs-lookup"><span data-stu-id="eda06-130">An alternative approach injects services directly into the view to obtain the options.</span></span> <span data-ttu-id="eda06-131">In questo modo la quantità di codice necessario per il controller viene ridotta al minimo, dato che la logica di costruzione di questo elemento di visualizzazione viene spostata nella visualizzazione stessa.</span><span class="sxs-lookup"><span data-stu-id="eda06-131">This minimizes the amount of code required by the controller, moving this view element construction logic into the view itself.</span></span> <span data-ttu-id="eda06-132">L'azione del controller per la visualizzazione di un modulo di modifica del profilo deve solo passare l'istanza del profilo al modulo:</span><span class="sxs-lookup"><span data-stu-id="eda06-132">The controller action to display a profile editing form only needs to pass the form the profile instance:</span></span>

[!code-csharp[](../../mvc/views/dependency-injection/sample/src/ViewInjectSample/Controllers/ProfileController.cs?highlight=9,19)]

<span data-ttu-id="eda06-133">Il form HTML usato per aggiornare queste preferenze include elenchi a discesa per tre delle proprietà:</span><span class="sxs-lookup"><span data-stu-id="eda06-133">The HTML form used to update these preferences includes dropdown lists for three of the properties:</span></span>

![Visualizzazione Update Profile con un modulo che consente l'immissione di nome, sesso, stato e colore preferito.](dependency-injection/_static/updateprofile.png)

<span data-ttu-id="eda06-135">Questi elenchi vengono popolati da un servizio inserito nella visualizzazione:</span><span class="sxs-lookup"><span data-stu-id="eda06-135">These lists are populated by a service that has been injected into the view:</span></span>

[!code-cshtml[](../../mvc/views/dependency-injection/sample/src/ViewInjectSample/Views/Profile/Index.cshtml?highlight=4,16,17,21,22,26,27)]

<span data-ttu-id="eda06-136">`ProfileOptionsService` è un servizio a livello di interfaccia utente progettato per fornire solo i dati necessari per il modulo:</span><span class="sxs-lookup"><span data-stu-id="eda06-136">The `ProfileOptionsService` is a UI-level service designed to provide just the data needed for this form:</span></span>

[!code-csharp[](../../mvc/views/dependency-injection/sample/src/ViewInjectSample/Model/Services/ProfileOptionsService.cs?highlight=7,13,24)]

> [!IMPORTANT]
> <span data-ttu-id="eda06-137">Non si deve dimenticare di registrare i tipi che si richiedono tramite l'inserimento di dipendenze nel metodo `Startup.ConfigureServices`.</span><span class="sxs-lookup"><span data-stu-id="eda06-137">Don't forget to register types you request through dependency injection in `Startup.ConfigureServices`.</span></span> <span data-ttu-id="eda06-138">Un tipo non registrato genera un'eccezione in fase di runtime, perché viene eseguita internamente una query al provider di servizi tramite [GetRequiredService](/dotnet/api/microsoft.extensions.dependencyinjection.serviceproviderserviceextensions.getrequiredservice).</span><span class="sxs-lookup"><span data-stu-id="eda06-138">An unregistered type throws an exception at runtime because the service provider is internally queried via [GetRequiredService](/dotnet/api/microsoft.extensions.dependencyinjection.serviceproviderserviceextensions.getrequiredservice).</span></span>

## <a name="overriding-services"></a><span data-ttu-id="eda06-139">Override di servizi</span><span class="sxs-lookup"><span data-stu-id="eda06-139">Overriding Services</span></span>

<span data-ttu-id="eda06-140">Oltre all'inserimento di nuovi servizi, questa tecnica può essere usata anche per eseguire l'override di servizi precedentemente inseriti in una pagina.</span><span class="sxs-lookup"><span data-stu-id="eda06-140">In addition to injecting new services, this technique can also be used to override previously injected services on a page.</span></span> <span data-ttu-id="eda06-141">La figura seguente mostra tutti i campi disponibili nella pagina usata nel primo esempio:</span><span class="sxs-lookup"><span data-stu-id="eda06-141">The figure below shows all of the fields available on the page used in the first example:</span></span>

![Menu di scelta rapida di IntelliSense in un simbolo @ tipizzato con l'elenco dei campi Html, Component, StatsService e Url](dependency-injection/_static/razor-fields.png)

<span data-ttu-id="eda06-143">Come si può notare, i campi predefiniti includono `Html`, `Component`, e `Url`, nonché il servizio `StatsService` inserito.</span><span class="sxs-lookup"><span data-stu-id="eda06-143">As you can see, the default fields include `Html`, `Component`, and `Url` (as well as the `StatsService` that we injected).</span></span> <span data-ttu-id="eda06-144">Se ad esempio si vogliono sostituire gli helper HTML predefiniti con helper personalizzati, è possibile farlo facilmente tramite `@inject`:</span><span class="sxs-lookup"><span data-stu-id="eda06-144">If for instance you wanted to replace the default HTML Helpers with your own, you could easily do so using `@inject`:</span></span>

[!code-cshtml[](../../mvc/views/dependency-injection/sample/src/ViewInjectSample/Views/Helper/Index.cshtml?highlight=3,11)]

<span data-ttu-id="eda06-145">Se si vuole estendere i servizi esistenti, è sufficiente usare questa tecnica mentre si eredita dall'implementazione esistente o si esegue il wrapping di quest'ultima con un'implementazione personalizzata.</span><span class="sxs-lookup"><span data-stu-id="eda06-145">If you want to extend existing services, you can simply use this technique while inheriting from or wrapping the existing implementation with your own.</span></span>

## <a name="see-also"></a><span data-ttu-id="eda06-146">Vedere anche</span><span class="sxs-lookup"><span data-stu-id="eda06-146">See Also</span></span>

* <span data-ttu-id="eda06-147">Blog di Simon Timms: [Getting Lookup Data Into Your View](https://blog.simontimms.com/2015/06/09/getting-lookup-data-into-you-view/) (Inserimento di dati di ricerca nei propri)</span><span class="sxs-lookup"><span data-stu-id="eda06-147">Simon Timms Blog: [Getting Lookup Data Into Your View](https://blog.simontimms.com/2015/06/09/getting-lookup-data-into-you-view/)</span></span>
