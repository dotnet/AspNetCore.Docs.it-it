---
title: ASP.NET Core e Entity Framework 6
author: rick-anderson
description: Entity Framework 6,3 e versioni successive funzionano con ASP.NET Core 3,1 e versioni successive.
ms.author: riande
ms.custom: mvc
ms.date: 7/14/2020
no-loc:
- appsettings.json
- ASP.NET Core Identity
- cookie
- Cookie
- Blazor
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: data/entity-framework-6
ms.openlocfilehash: 44211ac7fa2acc7a7a9471ef362cff02f94fa2b6
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102588269"
---
# <a name="aspnet-core-and-entity-framework-6"></a><span data-ttu-id="304fa-103">ASP.NET Core e Entity Framework 6</span><span class="sxs-lookup"><span data-stu-id="304fa-103">ASP.NET Core and Entity Framework 6</span></span>
::: moniker range=">= aspnetcore-3.0"

<span data-ttu-id="304fa-104">Di [Patrick Goode](https://github.com/attrib75)</span><span class="sxs-lookup"><span data-stu-id="304fa-104">By [Patrick Goode](https://github.com/attrib75)</span></span>

## <a name="using-entity-framework-6-with-aspnet-core"></a><span data-ttu-id="304fa-105">Uso di Entity Framework 6 con ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="304fa-105">Using Entity Framework 6 with ASP.NET Core</span></span>

<span data-ttu-id="304fa-106">Per lo sviluppo è necessario usare [Entity Framework Core](/ef/) .</span><span class="sxs-lookup"><span data-stu-id="304fa-106">[Entity Framework Core](/ef/) should be used for new development.</span></span> <span data-ttu-id="304fa-107">L' [esempio di download](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/data/entity-framework-6/3.xsample) USA [Entity Framework 6 (EF6)](/ef/ef6), che può essere usato per eseguire la migrazione di app in uscita da ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="304fa-107">The [download sample](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/data/entity-framework-6/3.xsample) uses [Entity Framework 6 (EF6)](/ef/ef6), which can be used to migrate exiting apps to ASP.NET Core.</span></span>

## <a name="additional-resources"></a><span data-ttu-id="304fa-108">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="304fa-108">Additional resources</span></span>

* [<span data-ttu-id="304fa-109">Entity Framework - configurazione basata su codice</span><span class="sxs-lookup"><span data-stu-id="304fa-109">Entity Framework - Code-Based Configuration</span></span>](/ef/ef6/fundamentals/configuring/code-based)

::: moniker-end

::: moniker range="< aspnetcore-3.0"

<span data-ttu-id="304fa-110">Di [Paweł Grudzień](https://github.com/pgrudzien12), [Damien Pontifex](https://github.com/DamienPontifex) e [Tom Dykstra](https://github.com/tdykstra)</span><span class="sxs-lookup"><span data-stu-id="304fa-110">By [Paweł Grudzień](https://github.com/pgrudzien12), [Damien Pontifex](https://github.com/DamienPontifex), and [Tom Dykstra](https://github.com/tdykstra)</span></span>

<span data-ttu-id="304fa-111">In questo articolo viene illustrato come usare Entity Framework 6 in un'applicazione ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="304fa-111">This article shows how to use Entity Framework 6 in an ASP.NET Core application.</span></span>    

## <a name="overview"></a><span data-ttu-id="304fa-112">Panoramica</span><span class="sxs-lookup"><span data-stu-id="304fa-112">Overview</span></span> 

<span data-ttu-id="304fa-113">Per l'uso di Entity Framework 6, il progetto deve eseguire la compilazione con .NET Framework, poiché Entity Framework 6 non supporta .NET Core.</span><span class="sxs-lookup"><span data-stu-id="304fa-113">To use Entity Framework 6, your project has to compile against .NET Framework, as Entity Framework 6 doesn't support .NET Core.</span></span> <span data-ttu-id="304fa-114">Se sono richieste funzionalità multipiattaforma, è necessario eseguire l'aggiornamento a [Entity Framework Core](/ef/).</span><span class="sxs-lookup"><span data-stu-id="304fa-114">If you need cross-platform features you will need to upgrade to [Entity Framework Core](/ef/).</span></span>  

<span data-ttu-id="304fa-115">Il modo consigliato per usare Entity Framework 6 in un'applicazione ASP.NET Core consiste nell'inserire il contesto EF6 e le classi del modello in un progetto di libreria di classi destinato .NET Framework.</span><span class="sxs-lookup"><span data-stu-id="304fa-115">The recommended way to use Entity Framework 6 in an ASP.NET Core application is to put the EF6 context and model classes in a class library project that targets .NET Framework.</span></span> <span data-ttu-id="304fa-116">Aggiungere un riferimento alla libreria di classi dal progetto ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="304fa-116">Add a reference to the class library from the ASP.NET Core project.</span></span> <span data-ttu-id="304fa-117">Vedere l'esempio [soluzione di Visual Studio con progetti EF6 e ASP.NET Core](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/data/entity-framework-6/sample/).</span><span class="sxs-lookup"><span data-stu-id="304fa-117">See the sample [Visual Studio solution with EF6 and ASP.NET Core projects](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/data/entity-framework-6/sample/).</span></span>    

<span data-ttu-id="304fa-118">Non è possibile inserire un contesto EF6 in un progetto ASP.NET poiché i progetti .NET Core non supportano tutte le funzionalità che i comandi EF6, come *Enable-Migrations*, richiedono.</span><span class="sxs-lookup"><span data-stu-id="304fa-118">You can't put an EF6 context in an ASP.NET Core project because .NET Core projects don't support all of the functionality that EF6 commands such as *Enable-Migrations* require.</span></span>    

<span data-ttu-id="304fa-119">Indipendentemente dal tipo di progetto in cui si trova il contesto EF6, solo gli strumenti da riga di comando EF6 funzionano con un contesto EF6.</span><span class="sxs-lookup"><span data-stu-id="304fa-119">Regardless of project type in which you locate your EF6 context, only EF6 command-line tools work with an EF6 context.</span></span> <span data-ttu-id="304fa-120">Ad esempio, `Scaffold-DbContext` è disponibile solo in Entity Framework Core.</span><span class="sxs-lookup"><span data-stu-id="304fa-120">For example, `Scaffold-DbContext` is only available in Entity Framework Core.</span></span> <span data-ttu-id="304fa-121">Se è necessario eseguire reverse engineering di un database in un modello EF6, vedere <https://docs.microsoft.com/ef/ef6/modeling/code-first/workflows/existing-database> .</span><span class="sxs-lookup"><span data-stu-id="304fa-121">If you need to do reverse engineering of a database into an EF6 model, see <https://docs.microsoft.com/ef/ef6/modeling/code-first/workflows/existing-database>.</span></span>    

## <a name="reference-full-framework-and-ef6-in-the-aspnet-core-project"></a><span data-ttu-id="304fa-122">Framework completo di riferimento e EF6 nel progetto ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="304fa-122">Reference full framework and EF6 in the ASP.NET Core project</span></span> 

<span data-ttu-id="304fa-123">Il progetto ASP.NET Core deve avere come destinazione .NET Framework e fare riferimento a EF6.</span><span class="sxs-lookup"><span data-stu-id="304fa-123">Your ASP.NET Core project needs to target .NET Framework and reference EF6.</span></span> <span data-ttu-id="304fa-124">Ad esempio, il file *csproj* del progetto ASP.NET Core avrà un aspetto simile a quanto segue (sono visualizzate solo le parti pertinenti del file).</span><span class="sxs-lookup"><span data-stu-id="304fa-124">For example, the *.csproj* file of your ASP.NET Core project will look similar to the following example (only relevant parts of the file are shown).</span></span>    

[!code-xml[](entity-framework-6/sample/MVCCore/MVCCore.csproj?range=3-9&highlight=2)]   

<span data-ttu-id="304fa-125">Quando si crea un nuovo progetto, usare il modello **Applicazione Web di ASP.NET Core (.NET Framework)**.</span><span class="sxs-lookup"><span data-stu-id="304fa-125">When creating a new project, use the **ASP.NET Core Web Application (.NET Framework)** template.</span></span>    

## <a name="handle-connection-strings"></a><span data-ttu-id="304fa-126">Gestire le stringhe di connessione</span><span class="sxs-lookup"><span data-stu-id="304fa-126">Handle connection strings</span></span>    

<span data-ttu-id="304fa-127">Gli strumenti da riga di comando EF6 che vengono usati nel progetto della libreria di classi EF6 richiedono un costruttore predefinito, così da poter creare un'istanza del contesto.</span><span class="sxs-lookup"><span data-stu-id="304fa-127">The EF6 command-line tools that you'll use in the EF6 class library project require a default constructor so they can instantiate the context.</span></span> <span data-ttu-id="304fa-128">Tuttavia è opportuno specificare la stringa di connessione da usare nel progetto ASP.NET Core. In questo caso il costruttore di contesto deve avere un parametro che consente di passare alla stringa di connessione.</span><span class="sxs-lookup"><span data-stu-id="304fa-128">But you'll probably want to specify the connection string to use in the ASP.NET Core project, in which case your context constructor must have a parameter that lets you pass in the connection string.</span></span> <span data-ttu-id="304fa-129">Ecco un esempio.</span><span class="sxs-lookup"><span data-stu-id="304fa-129">Here's an example.</span></span>   

[!code-csharp[](entity-framework-6/sample/EF6/SchoolContext.cs?name=snippet_Constructor)]   

<span data-ttu-id="304fa-130">Poiché il contesto EF6 non ha un costruttore senza parametri, il progetto EF6 deve fornire un'implementazione di <https://docs.microsoft.com/dotnet/api/system.data.entity.infrastructure.idbcontextfactory-1?view=entity-framework-6.2.0> .</span><span class="sxs-lookup"><span data-stu-id="304fa-130">Since your EF6 context doesn't have a parameterless constructor, your EF6 project has to provide an implementation of <https://docs.microsoft.com/dotnet/api/system.data.entity.infrastructure.idbcontextfactory-1?view=entity-framework-6.2.0>.</span></span> <span data-ttu-id="304fa-131">Gli strumenti da riga di comando EF6 trovano e usano detta implementazione, così da poter creare un'istanza del contesto.</span><span class="sxs-lookup"><span data-stu-id="304fa-131">The EF6 command-line tools will find and use that implementation so they can instantiate the context.</span></span> <span data-ttu-id="304fa-132">Ecco un esempio.</span><span class="sxs-lookup"><span data-stu-id="304fa-132">Here's an example.</span></span>   

[!code-csharp[](entity-framework-6/sample/EF6/SchoolContextFactory.cs?name=snippet_IDbContextFactory)]  

<span data-ttu-id="304fa-133">In questo codice di esempio, l'implementazione `IDbContextFactory` passa in una stringa di connessione hardcoded.</span><span class="sxs-lookup"><span data-stu-id="304fa-133">In this sample code, the `IDbContextFactory` implementation passes in a hard-coded connection string.</span></span> <span data-ttu-id="304fa-134">Questa è la stringa di connessione che gli strumenti da riga di comando useranno.</span><span class="sxs-lookup"><span data-stu-id="304fa-134">This is the connection string that the command-line tools will use.</span></span> <span data-ttu-id="304fa-135">È opportuno implementare una strategia per garantire che la libreria di classi usi la stessa stringa di connessione usata dall'applicazione chiamante.</span><span class="sxs-lookup"><span data-stu-id="304fa-135">You'll want to implement a strategy to ensure that the class library uses the same connection string that the calling application uses.</span></span> <span data-ttu-id="304fa-136">Ad esempio, è possibile ottenere il valore dalla variabile di un ambiente in entrambi i progetti.</span><span class="sxs-lookup"><span data-stu-id="304fa-136">For example, you could get the value from an environment variable in both projects.</span></span>   

## <a name="set-up-dependency-injection-in-the-aspnet-core-project"></a><span data-ttu-id="304fa-137">Impostare l'inserimento di dipendenze nel progetto ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="304fa-137">Set up dependency injection in the ASP.NET Core project</span></span>  

<span data-ttu-id="304fa-138">Nel file *Startup.cs* del progetto Core, impostare il contesto di EF6 per l'inserimento di dipendenze in `ConfigureServices`.</span><span class="sxs-lookup"><span data-stu-id="304fa-138">In the Core project's *Startup.cs* file, set up the EF6 context for dependency injection (DI) in `ConfigureServices`.</span></span> <span data-ttu-id="304fa-139">Gli oggetti di contesto di Entity Framework devono essere limitati a una durata per richiesta.</span><span class="sxs-lookup"><span data-stu-id="304fa-139">EF context objects should be scoped for a per-request lifetime.</span></span>   

[!code-csharp[](entity-framework-6/sample/MVCCore/Startup.cs?name=snippet_ConfigureServices&highlight=5)]   

<span data-ttu-id="304fa-140">È quindi possibile ottenere un'istanza del contesto nei controller usando l'inserimento di dipendenze.</span><span class="sxs-lookup"><span data-stu-id="304fa-140">You can then get an instance of the context in your controllers by using DI.</span></span> <span data-ttu-id="304fa-141">Il codice è simile a quanto si scriverebbe per un contesto EF Core:</span><span class="sxs-lookup"><span data-stu-id="304fa-141">The code is similar to what you'd write for an EF Core context:</span></span>    

[!code-csharp[](entity-framework-6/sample/MVCCore/Controllers/StudentsController.cs?name=snippet_ContextInController)]  

## <a name="sample-application"></a><span data-ttu-id="304fa-142">Applicazione di esempio</span><span class="sxs-lookup"><span data-stu-id="304fa-142">Sample application</span></span>   

<span data-ttu-id="304fa-143">Per un'applicazione di esempio funzionante, vedere la [soluzione di Visual Studio di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/data/entity-framework-6/sample/) che accompagna questo articolo.</span><span class="sxs-lookup"><span data-stu-id="304fa-143">For a working sample application, see the [sample Visual Studio solution](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/data/entity-framework-6/sample/) that accompanies this article.</span></span>    

<span data-ttu-id="304fa-144">Questo esempio può essere creato da zero tramite i passaggi seguenti in Visual Studio:</span><span class="sxs-lookup"><span data-stu-id="304fa-144">This sample can be created from scratch by the following steps in Visual Studio:</span></span>    

* <span data-ttu-id="304fa-145">Creare una soluzione.</span><span class="sxs-lookup"><span data-stu-id="304fa-145">Create a solution.</span></span>    

* <span data-ttu-id="304fa-146">**Aggiungi** > **Nuovo progetto** > **Web** > **Applicazione Web ASP.NET Core**</span><span class="sxs-lookup"><span data-stu-id="304fa-146">**Add** > **New Project** > **Web** > **ASP.NET Core Web Application**</span></span>    
  * <span data-ttu-id="304fa-147">Nella finestra di dialogo di selezione del modello di progetto selezionare API e .NET Framework nell'elenco a discesa</span><span class="sxs-lookup"><span data-stu-id="304fa-147">In project template selection dialog, select API and .NET Framework in dropdown</span></span> 

* <span data-ttu-id="304fa-148">**Aggiungi** > **Nuovo progetto** > **Windows Desktop** > **Libreria di classi (.NET Framework)**</span><span class="sxs-lookup"><span data-stu-id="304fa-148">**Add** > **New Project** > **Windows Desktop** > **Class Library (.NET Framework)**</span></span>  

* <span data-ttu-id="304fa-149">Nella **Console di Gestione pacchetti** eseguire il comando per entrambi i progetti`Install-Package Entityframework`.</span><span class="sxs-lookup"><span data-stu-id="304fa-149">In **Package Manager Console** (PMC) for both projects, run the command `Install-Package Entityframework`.</span></span>    

* <span data-ttu-id="304fa-150">Nel progetto di libreria di classi, creare le classi di modelli di dati e una classe di contesto e un'implementazione di `IDbContextFactory`.</span><span class="sxs-lookup"><span data-stu-id="304fa-150">In the class library project, create data model classes and a context class, and an implementation of `IDbContextFactory`.</span></span>    

* <span data-ttu-id="304fa-151">Nella Console di Gestione pacchetti del progetto di libreria di classi, eseguire i comandi `Enable-Migrations` e `Add-Migration Initial`.</span><span class="sxs-lookup"><span data-stu-id="304fa-151">In PMC for the class library project, run the commands `Enable-Migrations` and `Add-Migration Initial`.</span></span> <span data-ttu-id="304fa-152">Se il progetto ASP.NET Core è stato impostato come progetto di avvio, aggiungere `-StartupProjectName EF6` a questi comandi.</span><span class="sxs-lookup"><span data-stu-id="304fa-152">If you have set the ASP.NET Core project as the startup project, add `-StartupProjectName EF6` to these commands.</span></span> 

* <span data-ttu-id="304fa-153">Nel progetto Core aggiungere un riferimento al progetto di libreria di classi.</span><span class="sxs-lookup"><span data-stu-id="304fa-153">In the Core project, add a project reference to the class library project.</span></span>    

* <span data-ttu-id="304fa-154">Nel progetto Core, in *Startup.cs*, registrare il contesto per l'inserimento delle dipendenze.</span><span class="sxs-lookup"><span data-stu-id="304fa-154">In the Core project, in *Startup.cs*, register the context for DI.</span></span>    

* <span data-ttu-id="304fa-155">Nel progetto principale, in *appsettings.json* , aggiungere la stringa di connessione.</span><span class="sxs-lookup"><span data-stu-id="304fa-155">In the Core project, in *appsettings.json*, add the connection string.</span></span>  

* <span data-ttu-id="304fa-156">Nel progetto Core, aggiungere un controller e le visualizzazioni per verificare che è possibile leggere e scrivere i dati.</span><span class="sxs-lookup"><span data-stu-id="304fa-156">In the Core project, add a controller and view(s) to verify that you can read and write data.</span></span> <span data-ttu-id="304fa-157">Si noti che lo scaffolding di ASP.NET MVC di base non funzionerà con il contesto EF6 a cui viene fatto riferimento dalla libreria di classi.</span><span class="sxs-lookup"><span data-stu-id="304fa-157">(Note that ASP.NET Core MVC scaffolding won't work with the EF6 context referenced from the class library.)</span></span>

::: moniker-end
