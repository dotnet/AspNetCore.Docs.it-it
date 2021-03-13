---
title: Eseguire la migrazione da ASP.NET ad ASP.NET Core
author: isaac2004
description: Indicazioni sulla migrazione di app ASP.NET MVC o Web API esistenti ad ASP.NET Core.web
ms.author: scaddie
ms.date: 10/18/2019
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
uid: migration/proper-to-2x/index
ms.openlocfilehash: 7961890becc8f4513e0750f28341c9d4cf94e7ad
ms.sourcegitcommit: 07e7ee573fe4e12be93249a385db745d714ff6ae
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/12/2021
ms.locfileid: "103413334"
---
# <a name="migrate-from-aspnet-to-aspnet-core"></a><span data-ttu-id="60b8d-103">Eseguire la migrazione da ASP.NET ad ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="60b8d-103">Migrate from ASP.NET to ASP.NET Core</span></span>

<span data-ttu-id="60b8d-104">Di [Isaac Levin](https://isaaclevin.com)</span><span class="sxs-lookup"><span data-stu-id="60b8d-104">By [Isaac Levin](https://isaaclevin.com)</span></span>

<span data-ttu-id="60b8d-105">Questo articolo offre una guida di riferimento per la migrazione delle app ASP.NET ad ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="60b8d-105">This article serves as a reference guide for migrating ASP.NET apps to ASP.NET Core.</span></span> <span data-ttu-id="60b8d-106">Per una guida completa al porting, vedere l'eBook [che porta le app ASP.NET esistenti in .NET Core](https://aka.ms/aspnet-porting-ebook) .</span><span class="sxs-lookup"><span data-stu-id="60b8d-106">See the ebook [Porting existing ASP.NET apps to .NET Core](https://aka.ms/aspnet-porting-ebook) for a comprehensive porting guide.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="60b8d-107">Prerequisiti</span><span class="sxs-lookup"><span data-stu-id="60b8d-107">Prerequisites</span></span>

[<span data-ttu-id="60b8d-108">.NET Core SDK 2,2 o versione successiva</span><span class="sxs-lookup"><span data-stu-id="60b8d-108">.NET Core SDK 2.2 or later</span></span>](https://dotnet.microsoft.com/download)

## <a name="target-frameworks"></a><span data-ttu-id="60b8d-109">Framework di destinazione</span><span class="sxs-lookup"><span data-stu-id="60b8d-109">Target frameworks</span></span>

<span data-ttu-id="60b8d-110">I progetti di ASP.NET Core offrono agli sviluppatori la flessibilità necessaria per scegliere .NET Core, .NET Framework o entrambi come destinazione.</span><span class="sxs-lookup"><span data-stu-id="60b8d-110">ASP.NET Core projects offer developers the flexibility of targeting .NET Core, .NET Framework, or both.</span></span> <span data-ttu-id="60b8d-111">Vedere [Scelta di .NET Core o .NET Framework per le app server](/dotnet/standard/choosing-core-framework-server) per determinare quale framework di destinazione è più appropriato.</span><span class="sxs-lookup"><span data-stu-id="60b8d-111">See [Choosing between .NET Core and .NET Framework for server apps](/dotnet/standard/choosing-core-framework-server) to determine which target framework is most appropriate.</span></span>

<span data-ttu-id="60b8d-112">Quando la destinazione è .NET Framework, i progetti devono fare riferimento a singoli pacchetti NuGet.</span><span class="sxs-lookup"><span data-stu-id="60b8d-112">When targeting .NET Framework, projects need to reference individual NuGet packages.</span></span>

<span data-ttu-id="60b8d-113">La scelta di .NET Core come destinazione consente di eliminare numerosi riferimenti espliciti ai pacchetti, grazie al [metapacchetto](xref:fundamentals/metapackage-app) di ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="60b8d-113">Targeting .NET Core allows you to eliminate numerous explicit package references, thanks to the ASP.NET Core [metapackage](xref:fundamentals/metapackage-app).</span></span> <span data-ttu-id="60b8d-114">Installare il metapacchetto `Microsoft.AspNetCore.App` nel progetto:</span><span class="sxs-lookup"><span data-stu-id="60b8d-114">Install the `Microsoft.AspNetCore.App` metapackage in your project:</span></span>

```xml
<ItemGroup>
   <PackageReference Include="Microsoft.AspNetCore.App" />
</ItemGroup>
```

<span data-ttu-id="60b8d-115">Quando si usa il metapacchetto, con l'app non viene distribuito alcun pacchetto a cui si fa riferimento nel metapacchetto.</span><span class="sxs-lookup"><span data-stu-id="60b8d-115">When the metapackage is used, no packages referenced in the metapackage are deployed with the app.</span></span> <span data-ttu-id="60b8d-116">L'archivio di runtime di .NET Core include questi asset, che vengono precompilati per migliorare le prestazioni.</span><span class="sxs-lookup"><span data-stu-id="60b8d-116">The .NET Core Runtime Store includes these assets, and they're precompiled to improve performance.</span></span> <span data-ttu-id="60b8d-117">Per altri dettagli, vedere [Metapacchetto Microsoft.AspNetCore.All per ASP.NET Core](xref:fundamentals/metapackage-app).</span><span class="sxs-lookup"><span data-stu-id="60b8d-117">See [Microsoft.AspNetCore.App metapackage for ASP.NET Core](xref:fundamentals/metapackage-app) for more detail.</span></span>

## <a name="project-structure-differences"></a><span data-ttu-id="60b8d-118">Differenze di struttura del progetto</span><span class="sxs-lookup"><span data-stu-id="60b8d-118">Project structure differences</span></span>

<span data-ttu-id="60b8d-119">Il formato di file *CSPROJ* è stato semplificato in ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="60b8d-119">The *.csproj* file format has been simplified in ASP.NET Core.</span></span> <span data-ttu-id="60b8d-120">Alcune modifiche importanti includono:</span><span class="sxs-lookup"><span data-stu-id="60b8d-120">Some notable changes include:</span></span>

- <span data-ttu-id="60b8d-121">L'inclusione esplicita dei file non è necessaria affinché i file vengano considerati parte del progetto.</span><span class="sxs-lookup"><span data-stu-id="60b8d-121">Explicit inclusion of files isn't necessary for them to be considered part of the project.</span></span> <span data-ttu-id="60b8d-122">In questo modo si riduce il rischio di conflitti di merge XML quando si lavora con team di grandi dimensioni.</span><span class="sxs-lookup"><span data-stu-id="60b8d-122">This reduces the risk of XML merge conflicts when working on large teams.</span></span>
- <span data-ttu-id="60b8d-123">Non sono presenti riferimenti basati su GUID ad altri progetti e questo migliora la leggibilità dei file.</span><span class="sxs-lookup"><span data-stu-id="60b8d-123">There are no GUID-based references to other projects, which improves file readability.</span></span>
- <span data-ttu-id="60b8d-124">Il file può essere modificato senza scaricarlo in Visual Studio:</span><span class="sxs-lookup"><span data-stu-id="60b8d-124">The file can be edited without unloading it in Visual Studio:</span></span>

    ![Modificare l'opzione CSPROJ del menu di scelta rapida in Visual Studio 2017](_static/EditProjectVs2017.png)

## <a name="globalasax-file-replacement"></a><span data-ttu-id="60b8d-126">Sostituzione di file Global.asax</span><span class="sxs-lookup"><span data-stu-id="60b8d-126">Global.asax file replacement</span></span>

<span data-ttu-id="60b8d-127">In ASP.NET Core è stato introdotto un nuovo meccanismo per l'avvio automatico delle app.</span><span class="sxs-lookup"><span data-stu-id="60b8d-127">ASP.NET Core introduced a new mechanism for bootstrapping an app.</span></span> <span data-ttu-id="60b8d-128">Il punto di ingresso per le applicazioni ASP.NET è il file *Global.asax*.</span><span class="sxs-lookup"><span data-stu-id="60b8d-128">The entry point for ASP.NET applications is the *Global.asax* file.</span></span> <span data-ttu-id="60b8d-129">Attività quali la configurazione della route e le registrazioni di area e filtro vengono gestite nel file *Global.asax*.</span><span class="sxs-lookup"><span data-stu-id="60b8d-129">Tasks such as route configuration and filter and area registrations are handled in the *Global.asax* file.</span></span>

[!code-csharp[](samples/globalasax-sample.cs)]

<span data-ttu-id="60b8d-130">Con questo approccio l'applicazione e il server a cui viene distribuita vengono accoppiati in un modo che interferisce con l'implementazione.</span><span class="sxs-lookup"><span data-stu-id="60b8d-130">This approach couples the application and the server to which it's deployed in a way that interferes with the implementation.</span></span> <span data-ttu-id="60b8d-131">Al fine di disaccoppiare gli elementi, è stata introdotta la funzionalità [OWIN](https://owin.org/) che offre un modo più semplice di usare più framework insieme.</span><span class="sxs-lookup"><span data-stu-id="60b8d-131">In an effort to decouple, [OWIN](https://owin.org/) was introduced to provide a cleaner way to use multiple frameworks together.</span></span> <span data-ttu-id="60b8d-132">OWIN offre una pipeline per aggiungere solo i moduli necessari.</span><span class="sxs-lookup"><span data-stu-id="60b8d-132">OWIN provides a pipeline to add only the modules needed.</span></span> <span data-ttu-id="60b8d-133">L'ambiente host accetta una funzione di [avvio](xref:fundamentals/startup) per configurare i servizi e la pipeline delle richieste dell'applicazione.</span><span class="sxs-lookup"><span data-stu-id="60b8d-133">The hosting environment takes a [Startup](xref:fundamentals/startup) function to configure services and the app's request pipeline.</span></span> <span data-ttu-id="60b8d-134">`Startup` registra un set di middleware con l'applicazione.</span><span class="sxs-lookup"><span data-stu-id="60b8d-134">`Startup` registers a set of middleware with the application.</span></span> <span data-ttu-id="60b8d-135">Per ogni richiesta, l'applicazione chiama ognuno dei componenti middleware con il puntatore iniziale di un elenco collegato a un set esistente di gestori.</span><span class="sxs-lookup"><span data-stu-id="60b8d-135">For each request, the application calls each of the middleware components with the head pointer of a linked list to an existing set of handlers.</span></span> <span data-ttu-id="60b8d-136">Ogni componente middleware può aggiungere uno o più gestori alla pipeline di gestione delle richieste.</span><span class="sxs-lookup"><span data-stu-id="60b8d-136">Each middleware component can add one or more handlers to the request handling pipeline.</span></span> <span data-ttu-id="60b8d-137">Questa operazione viene eseguita restituendo un riferimento al gestore che rappresenta il nuovo inizio dell'elenco.</span><span class="sxs-lookup"><span data-stu-id="60b8d-137">This is accomplished by returning a reference to the handler that's the new head of the list.</span></span> <span data-ttu-id="60b8d-138">Ogni gestore è responsabile della memorizzazione e della chiamata del gestore successivo nell'elenco.</span><span class="sxs-lookup"><span data-stu-id="60b8d-138">Each handler is responsible for remembering and invoking the next handler in the list.</span></span> <span data-ttu-id="60b8d-139">Con ASP.NET Core, il punto di ingresso a un'applicazione è `Startup` e non esiste più una dipendenza da *Global.asax*.</span><span class="sxs-lookup"><span data-stu-id="60b8d-139">With ASP.NET Core, the entry point to an application is `Startup`, and you no longer have a dependency on *Global.asax*.</span></span> <span data-ttu-id="60b8d-140">Quando si usa OWIN con .NET Framework, usare una pipeline simile alla seguente:</span><span class="sxs-lookup"><span data-stu-id="60b8d-140">When using OWIN with .NET Framework, use something like the following as a pipeline:</span></span>

[!code-csharp[](samples/webapi-owin.cs)]

<span data-ttu-id="60b8d-141">Ciò consente di configurare le route predefinite e impostare come predefinito XmlSerialization per Json.</span><span class="sxs-lookup"><span data-stu-id="60b8d-141">This configures your default routes, and defaults to XmlSerialization over Json.</span></span> <span data-ttu-id="60b8d-142">Aggiungere altro middleware a questa pipeline in base alle esigenze, ad esempio caricamento di servizi, impostazioni di configurazione, file statici e così via.</span><span class="sxs-lookup"><span data-stu-id="60b8d-142">Add other Middleware to this pipeline as needed (loading services, configuration settings, static files, etc.).</span></span>

<span data-ttu-id="60b8d-143">ASP.NET Core usa un approccio simile, ma non si basa su OWIN per gestire la voce.</span><span class="sxs-lookup"><span data-stu-id="60b8d-143">ASP.NET Core uses a similar approach, but doesn't rely on OWIN to handle the entry.</span></span> <span data-ttu-id="60b8d-144">L'operazione viene invece eseguita usando il metodo *Program.cs* `Main` (simile alle applicazioni console) e `Startup` viene caricato da tale posizione.</span><span class="sxs-lookup"><span data-stu-id="60b8d-144">Instead, that's done through the *Program.cs* `Main` method (similar to console applications) and `Startup` is loaded through there.</span></span>

[!code-csharp[](samples/program.cs)]

<span data-ttu-id="60b8d-145">`Startup` deve includere un metodo `Configure`.</span><span class="sxs-lookup"><span data-stu-id="60b8d-145">`Startup` must include a `Configure` method.</span></span> <span data-ttu-id="60b8d-146">In `Configure` aggiungere il middleware necessario alla pipeline.</span><span class="sxs-lookup"><span data-stu-id="60b8d-146">In `Configure`, add the necessary middleware to the pipeline.</span></span> <span data-ttu-id="60b8d-147">Nell'esempio seguente, estratto dal modello di sito Web predefinito, i metodi di estensione configurano la pipeline con il supporto per:</span><span class="sxs-lookup"><span data-stu-id="60b8d-147">In the following example (from the default web site template), extension methods configure the pipeline with support for:</span></span>

- <span data-ttu-id="60b8d-148">Pagine di errore</span><span class="sxs-lookup"><span data-stu-id="60b8d-148">Error pages</span></span>
- <span data-ttu-id="60b8d-149">Protocollo HTTP Strict Transport Security (HSTS)</span><span class="sxs-lookup"><span data-stu-id="60b8d-149">HTTP Strict Transport Security</span></span>
- <span data-ttu-id="60b8d-150">Reindirizzamento HTTP a HTTPS</span><span class="sxs-lookup"><span data-stu-id="60b8d-150">HTTP redirection to HTTPS</span></span>
- <span data-ttu-id="60b8d-151">ASP.NET Core MVC</span><span class="sxs-lookup"><span data-stu-id="60b8d-151">ASP.NET Core MVC</span></span>

[!code-csharp[](samples/startup.cs)]

<span data-ttu-id="60b8d-152">L'host e applicazione sono stati disaccoppiati e questo offre la possibilità di passare a una piattaforma diversa in futuro.</span><span class="sxs-lookup"><span data-stu-id="60b8d-152">The host and application have been decoupled, which provides the flexibility of moving to a different platform in the future.</span></span>

> [!NOTE]
> <span data-ttu-id="60b8d-153">Per un riferimento più completo relativo all'avvio e al middleware di ASP.NET Core, vedere [Startup in ASP.NET Core](xref:fundamentals/startup) (Operazioni iniziali in ASP.NET Core)</span><span class="sxs-lookup"><span data-stu-id="60b8d-153">For a more in-depth reference to ASP.NET Core Startup and Middleware, see [Startup in ASP.NET Core](xref:fundamentals/startup)</span></span>

## <a name="store-configurations"></a><span data-ttu-id="60b8d-154">Configurazioni di archiviazione</span><span class="sxs-lookup"><span data-stu-id="60b8d-154">Store configurations</span></span>

<span data-ttu-id="60b8d-155">ASP.NET supporta le impostazioni di archiviazione.</span><span class="sxs-lookup"><span data-stu-id="60b8d-155">ASP.NET supports storing settings.</span></span> <span data-ttu-id="60b8d-156">Tali impostazioni vengono usate, ad esempio, per supportare l'ambiente in cui vengono distribuite le applicazioni.</span><span class="sxs-lookup"><span data-stu-id="60b8d-156">These setting are used, for example, to support the environment to which the applications were deployed.</span></span> <span data-ttu-id="60b8d-157">Una prassi comune era archiviare tutte le coppie chiave-valore personalizzate nella sezione `<appSettings>` del file *Web.config*:</span><span class="sxs-lookup"><span data-stu-id="60b8d-157">A common practice was to store all custom key-value pairs in the `<appSettings>` section of the *Web.config* file:</span></span>

[!code-xml[](samples/webconfig-sample.xml)]

<span data-ttu-id="60b8d-158">Le applicazioni leggono queste impostazioni usando la raccolta `ConfigurationManager.AppSettings` nello spazio dei nomi `System.Configuration`:</span><span class="sxs-lookup"><span data-stu-id="60b8d-158">Applications read these settings using the `ConfigurationManager.AppSettings` collection in the `System.Configuration` namespace:</span></span>

[!code-csharp[](samples/read-webconfig.cs)]

<span data-ttu-id="60b8d-159">ASP.NET Core è in grado di archiviare i dati di configurazione per l'applicazione in tutti i file e di caricarli come parte dell'avvio automatico del middleware.</span><span class="sxs-lookup"><span data-stu-id="60b8d-159">ASP.NET Core can store configuration data for the application in any file and load them as part of middleware bootstrapping.</span></span> <span data-ttu-id="60b8d-160">Il file predefinito usato nei modelli di progetto è il seguente *appsettings.json* :</span><span class="sxs-lookup"><span data-stu-id="60b8d-160">The default file used in the project templates is *appsettings.json*:</span></span>

[!code-json[](samples/appsettings-sample.json)]

<span data-ttu-id="60b8d-161">Il caricamento del file in un'istanza di `IConfiguration` all'interno dell'applicazione viene eseguito in *Startup.cs*:</span><span class="sxs-lookup"><span data-stu-id="60b8d-161">Loading this file into an instance of `IConfiguration` inside your application is done in *Startup.cs*:</span></span>

[!code-csharp[](samples/startup-builder.cs)]

<span data-ttu-id="60b8d-162">L'app legge da `Configuration` per ottenere le impostazioni:</span><span class="sxs-lookup"><span data-stu-id="60b8d-162">The app reads from `Configuration` to get the settings:</span></span>

[!code-csharp[](samples/read-appsettings.cs)]

<span data-ttu-id="60b8d-163">Sono disponibili estensioni di questo approccio che aumentano l'efficacia del processo, ad esempio l'uso dell'[inserimento delle dipendenze](xref:fundamentals/dependency-injection) per caricare un servizio con questi valori.</span><span class="sxs-lookup"><span data-stu-id="60b8d-163">There are extensions to this approach to make the process more robust, such as using [Dependency Injection](xref:fundamentals/dependency-injection) (DI) to load a service with these values.</span></span> <span data-ttu-id="60b8d-164">L'approccio con inserimento delle dipendenze offre un set fortemente tipizzato di oggetti di configurazione.</span><span class="sxs-lookup"><span data-stu-id="60b8d-164">The DI approach provides a strongly-typed set of configuration objects.</span></span>

```csharp
// Assume AppConfiguration is a class representing a strongly-typed version of AppConfiguration section
services.Configure<AppConfiguration>(Configuration.GetSection("AppConfiguration"));
```

> [!NOTE]
> <span data-ttu-id="60b8d-165">Per informazioni più dettagliate sulla configurazione di ASP.NET Core, vedere [Configurazione in ASP.NET Core](xref:fundamentals/configuration/index).</span><span class="sxs-lookup"><span data-stu-id="60b8d-165">For a more in-depth reference to ASP.NET Core configuration, see [Configuration in ASP.NET Core](xref:fundamentals/configuration/index).</span></span>

## <a name="native-dependency-injection"></a><span data-ttu-id="60b8d-166">Inserimento delle dipendenze nativo</span><span class="sxs-lookup"><span data-stu-id="60b8d-166">Native dependency injection</span></span>

<span data-ttu-id="60b8d-167">Un obiettivo importante nella compilazione di applicazioni scalabili di grandi dimensioni è l'accoppiamento libero di componenti e servizi.</span><span class="sxs-lookup"><span data-stu-id="60b8d-167">An important goal when building large, scalable applications is the loose coupling of components and services.</span></span> <span data-ttu-id="60b8d-168">L'[inserimento delle dipendenze](xref:fundamentals/dependency-injection) è una tecnica comune che consente di raggiungerlo ed è un componente nativo di ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="60b8d-168">[Dependency Injection](xref:fundamentals/dependency-injection) is a popular technique for achieving this, and it's a native component of ASP.NET Core.</span></span>

<span data-ttu-id="60b8d-169">Nelle app ASP.NET gli sviluppatori si affidano a una libreria di terze parti per implementare l'inserimento delle dipendenze.</span><span class="sxs-lookup"><span data-stu-id="60b8d-169">In ASP.NET apps, developers rely on a third-party library to implement Dependency Injection.</span></span> <span data-ttu-id="60b8d-170">Una di queste librerie è [Unity](https://github.com/unitycontainer/unity) e fa parte di Modelli e procedure Microsoft.</span><span class="sxs-lookup"><span data-stu-id="60b8d-170">One such library is [Unity](https://github.com/unitycontainer/unity), provided by Microsoft Patterns & Practices.</span></span>

<span data-ttu-id="60b8d-171">Un esempio di configurazione dell'inserimento delle dipendenze con Unity è l'implementazione di `IDependencyResolver` che esegue il wrapping di un oggetto `UnityContainer`:</span><span class="sxs-lookup"><span data-stu-id="60b8d-171">An example of setting up Dependency Injection with Unity is implementing `IDependencyResolver` that wraps a `UnityContainer`:</span></span>

[!code-csharp[](samples/sample8.cs)]

<span data-ttu-id="60b8d-172">Creare un'istanza di `UnityContainer`, registrare il servizio e impostare il sistema di risoluzione delle dipendenze di `HttpConfiguration` sulla nuova istanza di `UnityResolver` per il contenitore:</span><span class="sxs-lookup"><span data-stu-id="60b8d-172">Create an instance of your `UnityContainer`, register your service, and set the dependency resolver of `HttpConfiguration` to the new instance of `UnityResolver` for your container:</span></span>

[!code-csharp[](samples/sample9.cs)]

<span data-ttu-id="60b8d-173">Inserire `IProductRepository` dove necessario:</span><span class="sxs-lookup"><span data-stu-id="60b8d-173">Inject `IProductRepository` where needed:</span></span>

[!code-csharp[](samples/sample5.cs)]

<span data-ttu-id="60b8d-174">Poiché l'inserimento delle dipendenze fa parte di ASP.NET Core, è possibile aggiungere il servizio nel metodo `ConfigureServices` di *Startup.cs*:</span><span class="sxs-lookup"><span data-stu-id="60b8d-174">Because Dependency Injection is part of ASP.NET Core, you can add your service in the `ConfigureServices` method of *Startup.cs*:</span></span>

[!code-csharp[](samples/configure-services.cs)]

<span data-ttu-id="60b8d-175">Il repository può essere inserito in qualsiasi posizione, analogamente a Unity.</span><span class="sxs-lookup"><span data-stu-id="60b8d-175">The repository can be injected anywhere, as was true with Unity.</span></span>

> [!NOTE]
> <span data-ttu-id="60b8d-176">Per altre informazioni sull'inserimento delle dipendenze, vedere [Inserimento di dipendenze](xref:fundamentals/dependency-injection).</span><span class="sxs-lookup"><span data-stu-id="60b8d-176">For more information on dependency injection, see [Dependency injection](xref:fundamentals/dependency-injection).</span></span>

## <a name="serve-static-files"></a><span data-ttu-id="60b8d-177">Usare i file statici</span><span class="sxs-lookup"><span data-stu-id="60b8d-177">Serve static files</span></span>

<span data-ttu-id="60b8d-178">Una parte importante dello sviluppo Web è la possibilità di distribuire asset statici sul lato client.</span><span class="sxs-lookup"><span data-stu-id="60b8d-178">An important part of web development is the ability to serve static, client-side assets.</span></span> <span data-ttu-id="60b8d-179">Gli esempi più comuni di file statici sono HTML, CSS, Javascript e immagini.</span><span class="sxs-lookup"><span data-stu-id="60b8d-179">The most common examples of static files are HTML, CSS, Javascript, and images.</span></span> <span data-ttu-id="60b8d-180">Questi file devono essere salvati nella posizione di pubblicazione dell'app (o della rete CDN) con riferimenti che ne consentano il caricamento da parte di una richiesta.</span><span class="sxs-lookup"><span data-stu-id="60b8d-180">These files need to be saved in the published location of the app (or CDN) and referenced so they can be loaded by a request.</span></span> <span data-ttu-id="60b8d-181">Questo processo è stato modificato in ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="60b8d-181">This process has changed in ASP.NET Core.</span></span>

<span data-ttu-id="60b8d-182">In ASP.NET i file statici vengono archiviati in directory diverse e viene fatto riferimento ai file nelle viste.</span><span class="sxs-lookup"><span data-stu-id="60b8d-182">In ASP.NET, static files are stored in various directories and referenced in the views.</span></span>

<span data-ttu-id="60b8d-183">In ASP.NET Core i file statici vengono archiviati nella radice Web (*&lt; &gt; /wwwroot radice del contenuto*), a meno che non sia configurato diversamente.</span><span class="sxs-lookup"><span data-stu-id="60b8d-183">In ASP.NET Core, static files are stored in the "web root" (*&lt;content root&gt;/wwwroot*), unless configured otherwise.</span></span> <span data-ttu-id="60b8d-184">I file vengono caricati nella pipeline delle richieste chiamando il metodo di estensione `UseStaticFiles` da `Startup.Configure`:</span><span class="sxs-lookup"><span data-stu-id="60b8d-184">The files are loaded into the request pipeline by invoking the `UseStaticFiles` extension method from `Startup.Configure`:</span></span>

[!code-csharp[](../../fundamentals/static-files/samples/1.x/StaticFilesSample/StartupStaticFiles.cs?highlight=3&name=snippet_ConfigureMethod)]

> [!NOTE]
> <span data-ttu-id="60b8d-185">Se la destinazione è .NET Framework, installare il pacchetto NuGet `Microsoft.AspNetCore.StaticFiles`.</span><span class="sxs-lookup"><span data-stu-id="60b8d-185">If targeting .NET Framework, install the NuGet package `Microsoft.AspNetCore.StaticFiles`.</span></span>

<span data-ttu-id="60b8d-186">Ad esempio, un asset immagine nella cartella *wwwroot/images* è accessibile al browser in corrispondenza di una posizione come `http://<app>/images/<imageFileName>`.</span><span class="sxs-lookup"><span data-stu-id="60b8d-186">For example, an image asset in the *wwwroot/images* folder is accessible to the browser at a location such as `http://<app>/images/<imageFileName>`.</span></span>

> [!NOTE]
> <span data-ttu-id="60b8d-187">Per informazioni più dettagliate sulla gestione dei file statici in ASP.NET Core, vedere [File statici](xref:fundamentals/static-files).</span><span class="sxs-lookup"><span data-stu-id="60b8d-187">For a more in-depth reference to serving static files in ASP.NET Core, see [Static files](xref:fundamentals/static-files).</span></span>

## <a name="multi-value-cookies"></a><span data-ttu-id="60b8d-188">S multivalore cookie</span><span class="sxs-lookup"><span data-stu-id="60b8d-188">Multi-value cookies</span></span>

<span data-ttu-id="60b8d-189">Gli oggetti [multivalore cookie ](xref:System.Web.HttpCookie.Values) non sono supportati in ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="60b8d-189">[Multi-value cookies](xref:System.Web.HttpCookie.Values) aren't supported in ASP.NET Core.</span></span> <span data-ttu-id="60b8d-190">Crearne uno cookie per ogni valore.</span><span class="sxs-lookup"><span data-stu-id="60b8d-190">Create one cookie per value.</span></span>

## <a name="authentication-cookies-are-not-compressed-in-aspnet-core"></a><span data-ttu-id="60b8d-191">L'autenticazione cookie non è compressa in ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="60b8d-191">Authentication cookies are not compressed in ASP.NET Core</span></span>

[!INCLUDE[](~/includes/cookies-not-compressed.md)]

## <a name="partial-app-migration"></a><span data-ttu-id="60b8d-192">Migrazione parziale delle app</span><span class="sxs-lookup"><span data-stu-id="60b8d-192">Partial app migration</span></span>

<span data-ttu-id="60b8d-193">Un approccio alla migrazione parziale delle app consiste nel creare un'applicazione secondaria IIS e spostare solo determinate route da ASP.NET 4. x a ASP.NET Core mantenendo la struttura dell'URL dell'app.</span><span class="sxs-lookup"><span data-stu-id="60b8d-193">One approach to partial app migration is to create an IIS sub-application and only move certain routes from ASP.NET 4.x to ASP.NET Core while preserving the URL structure the app.</span></span> <span data-ttu-id="60b8d-194">Si consideri, ad esempio, la struttura dell'URL dell'app dal file *applicationHost.config* :</span><span class="sxs-lookup"><span data-stu-id="60b8d-194">For example, consider the URL structure of the app from the *applicationHost.config* file:</span></span>

```xml
<sites>
    <site name="Default Web Site" id="1" serverAutoStart="true">
        <application path="/">
            <virtualDirectory path="/" physicalPath="D:\sites\MainSite\" />
        </application>
        <application path="/api" applicationPool="DefaultAppPool">
            <virtualDirectory path="/" physicalPath="D:\sites\netcoreapi" />
        </application>
        <bindings>
            <binding protocol="http" bindingInformation="*:80:" />
            <binding protocol="https" bindingInformation="*:443:" sslFlags="0" />
        </bindings>
    </site>
    ...
</sites>
```

<span data-ttu-id="60b8d-195">Struttura di directory:</span><span class="sxs-lookup"><span data-stu-id="60b8d-195">Directory structure:</span></span>

```
.
├── MainSite
│   ├── ...
│   └── Web.config
└── NetCoreApi
    ├── ...
    └── web.config
```

## <a name="bind-and-input-formatters"></a><span data-ttu-id="60b8d-196">[BIND] e formattatori di input</span><span class="sxs-lookup"><span data-stu-id="60b8d-196">[BIND] and Input Formatters</span></span>

<span data-ttu-id="60b8d-197">[Nelle versioni precedenti di ASP.NET](/aspnet/mvc/overview/getting-started/introduction/examining-the-edit-methods-and-edit-view) è stato utilizzato l' `[Bind]` attributo per la protezione da attacchi overposting.</span><span class="sxs-lookup"><span data-stu-id="60b8d-197">[Previous versions of ASP.NET](/aspnet/mvc/overview/getting-started/introduction/examining-the-edit-methods-and-edit-view) used the `[Bind]` attribute to protect against overposting attacks.</span></span> <span data-ttu-id="60b8d-198">I [formattatori di input](xref:mvc/models/model-binding#input-formatters) funzionano in modo diverso in ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="60b8d-198">[Input formatters](xref:mvc/models/model-binding#input-formatters) work differently in ASP.NET Core.</span></span> <span data-ttu-id="60b8d-199">L' `[Bind]` attributo non è più progettato per impedire l'overposting quando viene usato con formattatori di input per analizzare JSON o XML.</span><span class="sxs-lookup"><span data-stu-id="60b8d-199">The `[Bind]` attribute is no longer designed to prevent overposting when used with input formatters to parse JSON or XML.</span></span> <span data-ttu-id="60b8d-200">Questi attributi influiscono sull'associazione di modelli quando l'origine dei dati è costituita da dati inviati con il `x-www-form-urlencoded` tipo di contenuto.</span><span class="sxs-lookup"><span data-stu-id="60b8d-200">These attributes affect model binding when the source of data is form data posted with the `x-www-form-urlencoded` content type.</span></span>

<span data-ttu-id="60b8d-201">Per le app che pubblicano informazioni JSON nei controller e usano formattatori di input JSON per analizzare i dati, è consigliabile sostituire l' `[Bind]` attributo con un modello di visualizzazione che corrisponda alle proprietà definite dall' `[Bind]` attributo.</span><span class="sxs-lookup"><span data-stu-id="60b8d-201">For apps that post JSON information to controllers and use JSON Input Formatters to parse the data, we recommend replacing the `[Bind]` attribute with a view model that matches the properties defined by the `[Bind]` attribute.</span></span>

## <a name="additional-resources"></a><span data-ttu-id="60b8d-202">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="60b8d-202">Additional resources</span></span>

- [<span data-ttu-id="60b8d-203">Portabilità in .NET Core - Librerie</span><span class="sxs-lookup"><span data-stu-id="60b8d-203">Porting Libraries to .NET Core</span></span>](/dotnet/core/porting/libraries)
