---
title: Eseguire la migrazione da API Web ASP.NET a ASP.NET Core
author: ardalis
description: Informazioni su come eseguire la migrazione di un'implementazione di API Web dall'API Web ASP.NET 4. x a ASP.NET Core MVC.
ms.author: scaddie
ms.custom: mvc
ms.date: 05/26/2020
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
uid: migration/webapi
ms.openlocfilehash: 3bd34f677271ddfc00e6e65ddf3a5e3c10eec863
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102588783"
---
# <a name="migrate-from-aspnet-web-api-to-aspnet-core"></a><span data-ttu-id="4014d-103">Eseguire la migrazione da API Web ASP.NET a ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="4014d-103">Migrate from ASP.NET Web API to ASP.NET Core</span></span>

<span data-ttu-id="4014d-104">Di [Scott Addie](https://twitter.com/scott_addie) e [Steve Smith](https://ardalis.com/)</span><span class="sxs-lookup"><span data-stu-id="4014d-104">By [Scott Addie](https://twitter.com/scott_addie) and [Steve Smith](https://ardalis.com/)</span></span>

<span data-ttu-id="4014d-105">Un'API Web ASP.NET 4. x è un servizio HTTP che raggiunge un'ampia gamma di client, inclusi browser e dispositivi mobili.</span><span class="sxs-lookup"><span data-stu-id="4014d-105">An ASP.NET 4.x Web API is an HTTP service that reaches a broad range of clients, including browsers and mobile devices.</span></span> <span data-ttu-id="4014d-106">ASP.NET Core combina i modelli di app per le API Web e MVC di ASP.NET 4. x in un unico modello di programmazione noto come ASP.NET Core MVC.</span><span class="sxs-lookup"><span data-stu-id="4014d-106">ASP.NET Core combines ASP.NET 4.x's MVC and Web API app models into a single programming model known as ASP.NET Core MVC.</span></span> <span data-ttu-id="4014d-107">Questo articolo illustra i passaggi necessari per eseguire la migrazione dall'API Web ASP.NET 4. x a ASP.NET Core MVC.</span><span class="sxs-lookup"><span data-stu-id="4014d-107">This article demonstrates the steps required to migrate from ASP.NET 4.x Web API to ASP.NET Core MVC.</span></span>

<span data-ttu-id="4014d-108">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/migration/webapi/sample) ([procedura per il download](xref:index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="4014d-108">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/migration/webapi/sample) ([how to download](xref:index#how-to-download-a-sample))</span></span>

::: moniker range=">= aspnetcore-3.0"

## <a name="prerequisites"></a><span data-ttu-id="4014d-109">Prerequisiti</span><span class="sxs-lookup"><span data-stu-id="4014d-109">Prerequisites</span></span>

[!INCLUDE [prerequisites](../includes/net-core-prereqs-vs-3.1.md)]

## <a name="review-aspnet-4x-web-api-project"></a><span data-ttu-id="4014d-110">Esaminare il progetto API Web ASP.NET 4. x</span><span class="sxs-lookup"><span data-stu-id="4014d-110">Review ASP.NET 4.x Web API project</span></span>

<span data-ttu-id="4014d-111">Questo articolo usa il progetto *ProductsApp* creato in [Introduzione con API Web ASP.NET 2](/aspnet/web-api/overview/getting-started-with-aspnet-web-api/tutorial-your-first-web-api).</span><span class="sxs-lookup"><span data-stu-id="4014d-111">This article uses the *ProductsApp* project created in [Getting Started with ASP.NET Web API 2](/aspnet/web-api/overview/getting-started-with-aspnet-web-api/tutorial-your-first-web-api).</span></span> <span data-ttu-id="4014d-112">In tale progetto, un progetto API Web ASP.NET 4. x di base viene configurato come segue.</span><span class="sxs-lookup"><span data-stu-id="4014d-112">In that project, a basic ASP.NET 4.x Web API project is configured as follows.</span></span>

<span data-ttu-id="4014d-113">In *Global.asax.cs* viene effettuata una chiamata a `WebApiConfig.Register` :</span><span class="sxs-lookup"><span data-stu-id="4014d-113">In *Global.asax.cs*, a call is made to `WebApiConfig.Register`:</span></span>

[!code-csharp[](webapi/sample/3.x/ProductsApp/Global.asax.cs?highlight=14)]

<span data-ttu-id="4014d-114">La `WebApiConfig` classe si trova nella cartella *app_start* e dispone di un `Register` metodo statico:</span><span class="sxs-lookup"><span data-stu-id="4014d-114">The `WebApiConfig` class is found in the *App_Start* folder and has a static `Register` method:</span></span>

[!code-csharp[](webapi/sample/3.x/ProductsApp/App_Start/WebApiConfig.cs)]

<span data-ttu-id="4014d-115">Classe precedente:</span><span class="sxs-lookup"><span data-stu-id="4014d-115">The preceding class:</span></span>

* <span data-ttu-id="4014d-116">Configura il [routing degli attributi](/aspnet/web-api/overview/web-api-routing-and-actions/attribute-routing-in-web-api-2), sebbene non venga effettivamente utilizzato.</span><span class="sxs-lookup"><span data-stu-id="4014d-116">Configures [attribute routing](/aspnet/web-api/overview/web-api-routing-and-actions/attribute-routing-in-web-api-2), although it's not actually being used.</span></span>
* <span data-ttu-id="4014d-117">Configura la tabella di routing.</span><span class="sxs-lookup"><span data-stu-id="4014d-117">Configures the routing table.</span></span>
<span data-ttu-id="4014d-118">Il codice di esempio prevede che gli URL corrispondano al formato `/api/{controller}/{id}` , con l' `{id}` opzione facoltativa.</span><span class="sxs-lookup"><span data-stu-id="4014d-118">The sample code expects URLs to match the format `/api/{controller}/{id}`, with `{id}` being optional.</span></span>

<span data-ttu-id="4014d-119">Le sezioni seguenti illustrano la migrazione del progetto API Web a ASP.NET Core MVC.</span><span class="sxs-lookup"><span data-stu-id="4014d-119">The following sections demonstrate migration of the Web API project to ASP.NET Core MVC.</span></span>

## <a name="create-the-destination-project"></a><span data-ttu-id="4014d-120">Creare il progetto di destinazione</span><span class="sxs-lookup"><span data-stu-id="4014d-120">Create the destination project</span></span>

<span data-ttu-id="4014d-121">Creare una nuova soluzione vuota in Visual Studio e aggiungere il progetto API Web ASP.NET 4. x da migrare:</span><span class="sxs-lookup"><span data-stu-id="4014d-121">Create a new blank solution in Visual Studio and add the ASP.NET 4.x Web API project to migrate:</span></span>

1. <span data-ttu-id="4014d-122">Scegliere **nuovo** progetto dal menu **file** > .</span><span class="sxs-lookup"><span data-stu-id="4014d-122">From the **File** menu, select **New** > **Project**.</span></span>
1. <span data-ttu-id="4014d-123">Selezionare il modello di **soluzione vuota** e selezionare **Avanti**.</span><span class="sxs-lookup"><span data-stu-id="4014d-123">Select the **Blank Solution** template and select **Next**.</span></span>
1. <span data-ttu-id="4014d-124">Assegnare alla soluzione il nome *WebAPIMigration*.</span><span class="sxs-lookup"><span data-stu-id="4014d-124">Name the solution *WebAPIMigration*.</span></span> <span data-ttu-id="4014d-125">Selezionare **Crea**.</span><span class="sxs-lookup"><span data-stu-id="4014d-125">Select **Create**.</span></span>
1. <span data-ttu-id="4014d-126">Aggiungere il progetto *ProductsApp* esistente alla soluzione.</span><span class="sxs-lookup"><span data-stu-id="4014d-126">Add the existing *ProductsApp* project to the solution.</span></span>

<span data-ttu-id="4014d-127">Aggiungere un nuovo progetto API per la migrazione a:</span><span class="sxs-lookup"><span data-stu-id="4014d-127">Add a new API project to migrate to:</span></span>

1. <span data-ttu-id="4014d-128">Aggiungere un nuovo progetto di **applicazione Web ASP.NET Core** alla soluzione.</span><span class="sxs-lookup"><span data-stu-id="4014d-128">Add a new **ASP.NET Core Web Application** project to the solution.</span></span>
1. <span data-ttu-id="4014d-129">Nella finestra di dialogo **Configura il nuovo progetto** assegnare al progetto il nome *ProductsCore* e selezionare **Crea**.</span><span class="sxs-lookup"><span data-stu-id="4014d-129">In the **Configure your new project** dialog, Name the project *ProductsCore*, and select **Create**.</span></span>
1. <span data-ttu-id="4014d-130">Nella finestra di dialogo **Crea una nuova applicazione Web ASP.NET Core** verificare che siano selezionati **.net Core** e **ASP.NET Core 3,1** .</span><span class="sxs-lookup"><span data-stu-id="4014d-130">In the **Create a new ASP.NET Core Web Application** dialog, confirm that **.NET Core** and **ASP.NET Core 3.1** are selected.</span></span> <span data-ttu-id="4014d-131">Selezionare il modello di progetto **API** e scegliere **Crea**.</span><span class="sxs-lookup"><span data-stu-id="4014d-131">Select the **API** project template, and select **Create**.</span></span>
1. <span data-ttu-id="4014d-132">Rimuovere i file di esempio *weatherforecast.cs* e *Controllers/WeatherForecastController. cs* dal nuovo progetto *ProductsCore* .</span><span class="sxs-lookup"><span data-stu-id="4014d-132">Remove the *WeatherForecast.cs* and *Controllers/WeatherForecastController.cs* example files from the new *ProductsCore* project.</span></span>

<span data-ttu-id="4014d-133">La soluzione ora contiene due progetti.</span><span class="sxs-lookup"><span data-stu-id="4014d-133">The solution now contains two projects.</span></span> <span data-ttu-id="4014d-134">Le sezioni seguenti illustrano come eseguire la migrazione del contenuto del progetto *ProductsApp* al progetto *ProductsCore* .</span><span class="sxs-lookup"><span data-stu-id="4014d-134">The following sections explain migrating the *ProductsApp* project's contents to the *ProductsCore* project.</span></span>

## <a name="migrate-configuration"></a><span data-ttu-id="4014d-135">Eseguire la migrazione della configurazione</span><span class="sxs-lookup"><span data-stu-id="4014d-135">Migrate configuration</span></span>

<span data-ttu-id="4014d-136">ASP.NET Core non utilizza la cartella *app_start* o il file *Global. asax* .</span><span class="sxs-lookup"><span data-stu-id="4014d-136">ASP.NET Core doesn't use the *App_Start* folder or the *Global.asax* file.</span></span> <span data-ttu-id="4014d-137">Il file di *web.config* viene inoltre aggiunto in fase di pubblicazione.</span><span class="sxs-lookup"><span data-stu-id="4014d-137">Additionally, the *web.config* file is added at publish time.</span></span>

<span data-ttu-id="4014d-138">La classe `Startup`:</span><span class="sxs-lookup"><span data-stu-id="4014d-138">The `Startup` class:</span></span>

* <span data-ttu-id="4014d-139">Sostituisce *Global. asax*.</span><span class="sxs-lookup"><span data-stu-id="4014d-139">Replaces *Global.asax*.</span></span>
* <span data-ttu-id="4014d-140">Gestisce tutte le attività di avvio dell'app.</span><span class="sxs-lookup"><span data-stu-id="4014d-140">Handles all app startup tasks.</span></span>

<span data-ttu-id="4014d-141">Per altre informazioni, vedere <xref:fundamentals/startup>.</span><span class="sxs-lookup"><span data-stu-id="4014d-141">For more information, see <xref:fundamentals/startup>.</span></span>

## <a name="migrate-models-and-controllers"></a><span data-ttu-id="4014d-142">Eseguire la migrazione di modelli e controller</span><span class="sxs-lookup"><span data-stu-id="4014d-142">Migrate models and controllers</span></span>

<span data-ttu-id="4014d-143">Il codice seguente mostra l'oggetto `ProductsController` da aggiornare per ASP.NET Core:</span><span class="sxs-lookup"><span data-stu-id="4014d-143">The following code shows the `ProductsController` to be updated for ASP.NET Core:</span></span>

[!code-csharp[](webapi/sample/3.x/ProductsApp/Controllers/ProductsController.cs)]

<span data-ttu-id="4014d-144">Aggiornare `ProductsController` per ASP.NET Core:</span><span class="sxs-lookup"><span data-stu-id="4014d-144">Update the `ProductsController` for ASP.NET Core:</span></span>

1. <span data-ttu-id="4014d-145">Copiare *Controllers/ProductsController. cs* e la cartella *Models* dal progetto originale a quello nuovo.</span><span class="sxs-lookup"><span data-stu-id="4014d-145">Copy *Controllers/ProductsController.cs* and the *Models* folder from the original project to the new one.</span></span>
1. <span data-ttu-id="4014d-146">Modificare lo spazio dei nomi radice dei file copiati in `ProductsCore` .</span><span class="sxs-lookup"><span data-stu-id="4014d-146">Change the copied files' root namespace to `ProductsCore`.</span></span>
1. <span data-ttu-id="4014d-147">Aggiornare l' `using ProductsApp.Models;` istruzione a `using ProductsCore.Models;` .</span><span class="sxs-lookup"><span data-stu-id="4014d-147">Update the `using ProductsApp.Models;` statement to `using ProductsCore.Models;`.</span></span>

<span data-ttu-id="4014d-148">I componenti seguenti non esistono in ASP.NET Core:</span><span class="sxs-lookup"><span data-stu-id="4014d-148">The following components don't exist in ASP.NET Core:</span></span>

* <span data-ttu-id="4014d-149">Classe `ApiController`</span><span class="sxs-lookup"><span data-stu-id="4014d-149">`ApiController` class</span></span>
* <span data-ttu-id="4014d-150">Spazio dei nomi `System.Web.Http`</span><span class="sxs-lookup"><span data-stu-id="4014d-150">`System.Web.Http` namespace</span></span>
* <span data-ttu-id="4014d-151">Interfaccia `IHttpActionResult`</span><span class="sxs-lookup"><span data-stu-id="4014d-151">`IHttpActionResult` interface</span></span>

<span data-ttu-id="4014d-152">Apportare le modifiche seguenti:</span><span class="sxs-lookup"><span data-stu-id="4014d-152">Make the following changes:</span></span>

1. <span data-ttu-id="4014d-153">Cambiare `ApiController` in <xref:Microsoft.AspNetCore.Mvc.ControllerBase>.</span><span class="sxs-lookup"><span data-stu-id="4014d-153">Change `ApiController` to <xref:Microsoft.AspNetCore.Mvc.ControllerBase>.</span></span> <span data-ttu-id="4014d-154">Aggiungere `using Microsoft.AspNetCore.Mvc;` per risolvere il `ControllerBase` riferimento.</span><span class="sxs-lookup"><span data-stu-id="4014d-154">Add `using Microsoft.AspNetCore.Mvc;` to resolve the `ControllerBase` reference.</span></span>
1. <span data-ttu-id="4014d-155">Eliminare `using System.Web.Http;`.</span><span class="sxs-lookup"><span data-stu-id="4014d-155">Delete `using System.Web.Http;`.</span></span>
1. <span data-ttu-id="4014d-156">Modificare il `GetProduct` tipo restituito dell'azione da `IHttpActionResult` a `ActionResult<Product>` .</span><span class="sxs-lookup"><span data-stu-id="4014d-156">Change the `GetProduct` action's return type from `IHttpActionResult` to `ActionResult<Product>`.</span></span>
1. <span data-ttu-id="4014d-157">Semplificare l' `GetProduct` istruzione dell'azione `return` a quanto segue:</span><span class="sxs-lookup"><span data-stu-id="4014d-157">Simplify the `GetProduct` action's `return` statement to the following:</span></span>

    ```csharp
    return product;
    ```

## <a name="configure-routing"></a><span data-ttu-id="4014d-158">Configurare il routing</span><span class="sxs-lookup"><span data-stu-id="4014d-158">Configure routing</span></span>

<span data-ttu-id="4014d-159">Il modello di progetto *API* ASP.NET Core include la configurazione del routing dell'endpoint nel codice generato.</span><span class="sxs-lookup"><span data-stu-id="4014d-159">The ASP.NET Core *API* project template includes endpoint routing configuration in the generated code.</span></span>

<span data-ttu-id="4014d-160">Gli elementi <xref:Microsoft.AspNetCore.Builder.EndpointRoutingApplicationBuilderExtensions.UseRouting%2A> e seguenti <xref:Microsoft.AspNetCore.Builder.EndpointRoutingApplicationBuilderExtensions.UseEndpoints%2A> chiamano:</span><span class="sxs-lookup"><span data-stu-id="4014d-160">The following <xref:Microsoft.AspNetCore.Builder.EndpointRoutingApplicationBuilderExtensions.UseRouting%2A> and <xref:Microsoft.AspNetCore.Builder.EndpointRoutingApplicationBuilderExtensions.UseEndpoints%2A> calls:</span></span>

* <span data-ttu-id="4014d-161">Registrare la corrispondenza della route e l'esecuzione dell'endpoint nella pipeline [middleware](xref:fundamentals/middleware/index) .</span><span class="sxs-lookup"><span data-stu-id="4014d-161">Register route matching and endpoint execution in the [middleware](xref:fundamentals/middleware/index) pipeline.</span></span>
* <span data-ttu-id="4014d-162">Sostituire il file di *app_start/webapiconfig.cs* del progetto *ProductsApp* .</span><span class="sxs-lookup"><span data-stu-id="4014d-162">Replace the *ProductsApp* project's *App_Start/WebApiConfig.cs* file.</span></span>

[!code-csharp[](webapi/sample/3.x/ProductsCore/Startup.cs?name=snippet_Configure&highlight=10,14)]

<span data-ttu-id="4014d-163">Configurare il routing come segue:</span><span class="sxs-lookup"><span data-stu-id="4014d-163">Configure routing as follows:</span></span>

1. <span data-ttu-id="4014d-164">Contrassegnare la `ProductsController` classe con gli attributi seguenti:</span><span class="sxs-lookup"><span data-stu-id="4014d-164">Mark the `ProductsController` class with the following attributes:</span></span>

    ```csharp
    [Route("api/[controller]")]
    [ApiController]
    ```

    <span data-ttu-id="4014d-165">L' [`[Route]`](xref:Microsoft.AspNetCore.Mvc.RouteAttribute) attributo precedente configura il modello di routing degli attributi del controller.</span><span class="sxs-lookup"><span data-stu-id="4014d-165">The preceding [`[Route]`](xref:Microsoft.AspNetCore.Mvc.RouteAttribute) attribute configures the controller's attribute routing pattern.</span></span> <span data-ttu-id="4014d-166">L' [`[ApiController]`](xref:Microsoft.AspNetCore.Mvc.ApiControllerAttribute) attributo rende il routing degli attributi un requisito per tutte le azioni nel controller.</span><span class="sxs-lookup"><span data-stu-id="4014d-166">The [`[ApiController]`](xref:Microsoft.AspNetCore.Mvc.ApiControllerAttribute) attribute makes attribute routing a requirement for all actions in this controller.</span></span>

    <span data-ttu-id="4014d-167">Il routing degli attributi supporta i token, ad esempio `[controller]` e `[action]` .</span><span class="sxs-lookup"><span data-stu-id="4014d-167">Attribute routing supports tokens, such as `[controller]` and `[action]`.</span></span> <span data-ttu-id="4014d-168">In fase di esecuzione, ogni token viene sostituito rispettivamente dal nome del controller o dell'azione a cui è stato applicato l'attributo.</span><span class="sxs-lookup"><span data-stu-id="4014d-168">At runtime, each token is replaced with the name of the controller or action, respectively, to which the attribute has been applied.</span></span> <span data-ttu-id="4014d-169">I token:</span><span class="sxs-lookup"><span data-stu-id="4014d-169">The tokens:</span></span>
    * <span data-ttu-id="4014d-170">Ridurre il numero di stringhe Magic nel progetto.</span><span class="sxs-lookup"><span data-stu-id="4014d-170">Reduce the number of magic strings in the project.</span></span>
    * <span data-ttu-id="4014d-171">Verificare che le route rimangano sincronizzate con i controller e le azioni corrispondenti quando vengono applicati i refactoring di ridenominazione automatica.</span><span class="sxs-lookup"><span data-stu-id="4014d-171">Ensure routes remain synchronized with the corresponding controllers and actions when automatic rename refactorings are applied.</span></span>
1. <span data-ttu-id="4014d-172">Abilitare le richieste HTTP Get alle `ProductController` azioni:</span><span class="sxs-lookup"><span data-stu-id="4014d-172">Enable HTTP Get requests to the `ProductController` actions:</span></span>
    * <span data-ttu-id="4014d-173">Applicare l' [`[HttpGet]`](xref:Microsoft.AspNetCore.Mvc.HttpGetAttribute) attributo all' `GetAllProducts` azione.</span><span class="sxs-lookup"><span data-stu-id="4014d-173">Apply the [`[HttpGet]`](xref:Microsoft.AspNetCore.Mvc.HttpGetAttribute) attribute to the `GetAllProducts` action.</span></span>
    * <span data-ttu-id="4014d-174">Applicare l' `[HttpGet("{id}")]` attributo all' `GetProduct` azione.</span><span class="sxs-lookup"><span data-stu-id="4014d-174">Apply the `[HttpGet("{id}")]` attribute to the `GetProduct` action.</span></span>

<span data-ttu-id="4014d-175">Eseguire il progetto migrato e passare a `/api/products` .</span><span class="sxs-lookup"><span data-stu-id="4014d-175">Run the migrated project, and browse to `/api/products`.</span></span> <span data-ttu-id="4014d-176">Viene visualizzato un elenco completo di tre prodotti.</span><span class="sxs-lookup"><span data-stu-id="4014d-176">A full list of three products appears.</span></span> <span data-ttu-id="4014d-177">Passare a `/api/products/1`.</span><span class="sxs-lookup"><span data-stu-id="4014d-177">Browse to `/api/products/1`.</span></span> <span data-ttu-id="4014d-178">Il primo prodotto verrà visualizzato.</span><span class="sxs-lookup"><span data-stu-id="4014d-178">The first product appears.</span></span>

## <a name="additional-resources"></a><span data-ttu-id="4014d-179">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="4014d-179">Additional resources</span></span>

* <xref:web-api/index>
* <xref:web-api/action-return-types>
* <xref:mvc/compatibility-version>

::: moniker-end

::: moniker range="<= aspnetcore-2.2"
## <a name="prerequisites"></a><span data-ttu-id="4014d-180">Prerequisiti</span><span class="sxs-lookup"><span data-stu-id="4014d-180">Prerequisites</span></span>

[!INCLUDE [prerequisites](../includes/net-core-prereqs-vs2019-2.2.md)]

## <a name="review-aspnet-4x-web-api-project"></a><span data-ttu-id="4014d-181">Esaminare il progetto API Web ASP.NET 4. x</span><span class="sxs-lookup"><span data-stu-id="4014d-181">Review ASP.NET 4.x Web API project</span></span>

<span data-ttu-id="4014d-182">Questo articolo usa il progetto *ProductsApp* creato in [Introduzione con API Web ASP.NET 2](/aspnet/web-api/overview/getting-started-with-aspnet-web-api/tutorial-your-first-web-api).</span><span class="sxs-lookup"><span data-stu-id="4014d-182">This article uses the *ProductsApp* project created in [Getting Started with ASP.NET Web API 2](/aspnet/web-api/overview/getting-started-with-aspnet-web-api/tutorial-your-first-web-api).</span></span> <span data-ttu-id="4014d-183">In tale progetto, un progetto API Web ASP.NET 4. x di base viene configurato come segue.</span><span class="sxs-lookup"><span data-stu-id="4014d-183">In that project, a basic ASP.NET 4.x Web API project is configured as follows.</span></span>

<span data-ttu-id="4014d-184">In *Global.asax.cs* viene effettuata una chiamata a `WebApiConfig.Register` :</span><span class="sxs-lookup"><span data-stu-id="4014d-184">In *Global.asax.cs*, a call is made to `WebApiConfig.Register`:</span></span>

[!code-csharp[](webapi/sample/2.x/ProductsApp/Global.asax.cs?highlight=14)]

<span data-ttu-id="4014d-185">La `WebApiConfig` classe si trova nella cartella *app_start* e dispone di un `Register` metodo statico:</span><span class="sxs-lookup"><span data-stu-id="4014d-185">The `WebApiConfig` class is found in the *App_Start* folder and has a static `Register` method:</span></span>

[!code-csharp[](webapi/sample/2.x/ProductsApp/App_Start/WebApiConfig.cs)]

<span data-ttu-id="4014d-186">Questa classe configura il [routing degli attributi](/aspnet/web-api/overview/web-api-routing-and-actions/attribute-routing-in-web-api-2), sebbene non sia effettivamente utilizzato nel progetto.</span><span class="sxs-lookup"><span data-stu-id="4014d-186">This class configures [attribute routing](/aspnet/web-api/overview/web-api-routing-and-actions/attribute-routing-in-web-api-2), although it's not actually being used in the project.</span></span> <span data-ttu-id="4014d-187">Viene inoltre configurata la tabella di routing, che viene utilizzata da API Web ASP.NET.</span><span class="sxs-lookup"><span data-stu-id="4014d-187">It also configures the routing table, which is used by ASP.NET Web API.</span></span> <span data-ttu-id="4014d-188">In questo caso, l'API Web ASP.NET 4. x prevede che gli URL corrispondano al formato `/api/{controller}/{id}` , con l' `{id}` opzione facoltativo.</span><span class="sxs-lookup"><span data-stu-id="4014d-188">In this case, ASP.NET 4.x Web API expects URLs to match the format `/api/{controller}/{id}`, with `{id}` being optional.</span></span>

<span data-ttu-id="4014d-189">Le sezioni seguenti illustrano la migrazione del progetto API Web a ASP.NET Core MVC.</span><span class="sxs-lookup"><span data-stu-id="4014d-189">The following sections demonstrate migration of the Web API project to ASP.NET Core MVC.</span></span>

## <a name="create-the-destination-project"></a><span data-ttu-id="4014d-190">Creare il progetto di destinazione</span><span class="sxs-lookup"><span data-stu-id="4014d-190">Create the destination project</span></span>

<span data-ttu-id="4014d-191">Completare i passaggi seguenti in Visual Studio:</span><span class="sxs-lookup"><span data-stu-id="4014d-191">Complete the following steps in Visual Studio:</span></span>

* <span data-ttu-id="4014d-192">Passare a **file**  >  **nuovo**  >  **progetto**  >  **altri tipi di progetto**  >  **soluzioni di Visual Studio**.</span><span class="sxs-lookup"><span data-stu-id="4014d-192">Go to **File** > **New** > **Project** > **Other Project Types** > **Visual Studio Solutions**.</span></span> <span data-ttu-id="4014d-193">Selezionare **soluzione vuota** e denominare la soluzione *WebAPIMigration*.</span><span class="sxs-lookup"><span data-stu-id="4014d-193">Select **Blank Solution**, and name the solution *WebAPIMigration*.</span></span> <span data-ttu-id="4014d-194">Fare clic sul pulsante **OK** .</span><span class="sxs-lookup"><span data-stu-id="4014d-194">Click the **OK** button.</span></span>
* <span data-ttu-id="4014d-195">Aggiungere il progetto *ProductsApp* esistente alla soluzione.</span><span class="sxs-lookup"><span data-stu-id="4014d-195">Add the existing *ProductsApp* project to the solution.</span></span>
* <span data-ttu-id="4014d-196">Aggiungere un nuovo progetto di **applicazione Web ASP.NET Core** alla soluzione.</span><span class="sxs-lookup"><span data-stu-id="4014d-196">Add a new **ASP.NET Core Web Application** project to the solution.</span></span> <span data-ttu-id="4014d-197">Selezionare **.NET Core** Target Framework dall'elenco a discesa e selezionare il modello di progetto **API** .</span><span class="sxs-lookup"><span data-stu-id="4014d-197">Select the **.NET Core** target framework from the drop-down, and select the **API** project template.</span></span> <span data-ttu-id="4014d-198">Denominare il progetto *ProductsCore* e fare clic sul pulsante **OK** .</span><span class="sxs-lookup"><span data-stu-id="4014d-198">Name the project *ProductsCore*, and click the **OK** button.</span></span>

<span data-ttu-id="4014d-199">La soluzione ora contiene due progetti.</span><span class="sxs-lookup"><span data-stu-id="4014d-199">The solution now contains two projects.</span></span> <span data-ttu-id="4014d-200">Le sezioni seguenti illustrano come eseguire la migrazione del contenuto del progetto *ProductsApp* al progetto *ProductsCore* .</span><span class="sxs-lookup"><span data-stu-id="4014d-200">The following sections explain migrating the *ProductsApp* project's contents to the *ProductsCore* project.</span></span>

## <a name="migrate-configuration"></a><span data-ttu-id="4014d-201">Eseguire la migrazione della configurazione</span><span class="sxs-lookup"><span data-stu-id="4014d-201">Migrate configuration</span></span>

<span data-ttu-id="4014d-202">ASP.NET Core non USA:</span><span class="sxs-lookup"><span data-stu-id="4014d-202">ASP.NET Core doesn't use:</span></span>

* <span data-ttu-id="4014d-203">*App_start* cartella o il file *Global. asax*</span><span class="sxs-lookup"><span data-stu-id="4014d-203">*App_Start* folder or the *Global.asax* file</span></span>
* <span data-ttu-id="4014d-204">*web.config* file viene aggiunto in fase di pubblicazione.</span><span class="sxs-lookup"><span data-stu-id="4014d-204">*web.config* file is added at publish time.</span></span>

<span data-ttu-id="4014d-205">La classe `Startup`:</span><span class="sxs-lookup"><span data-stu-id="4014d-205">The `Startup` class:</span></span>

* <span data-ttu-id="4014d-206">Sostituisce *Global. asax*.</span><span class="sxs-lookup"><span data-stu-id="4014d-206">Replaces *Global.asax*.</span></span>
* <span data-ttu-id="4014d-207">Gestisce tutte le attività di avvio dell'app.</span><span class="sxs-lookup"><span data-stu-id="4014d-207">Handles all app startup tasks.</span></span>

<span data-ttu-id="4014d-208">Per altre informazioni, vedere <xref:fundamentals/startup>.</span><span class="sxs-lookup"><span data-stu-id="4014d-208">For more information, see <xref:fundamentals/startup>.</span></span>

<span data-ttu-id="4014d-209">In ASP.NET Core MVC, il routing degli attributi è incluso per impostazione predefinita quando <xref:Microsoft.AspNetCore.Builder.MvcApplicationBuilderExtensions.UseMvc*> viene chiamato in `Startup.Configure` .</span><span class="sxs-lookup"><span data-stu-id="4014d-209">In ASP.NET Core MVC, attribute routing is included by default when <xref:Microsoft.AspNetCore.Builder.MvcApplicationBuilderExtensions.UseMvc*> is called in `Startup.Configure`.</span></span> <span data-ttu-id="4014d-210">La `UseMvc` chiamata seguente sostituisce il file di *app_start/webapiconfig.cs* del progetto *ProductsApp* :</span><span class="sxs-lookup"><span data-stu-id="4014d-210">The following `UseMvc` call replaces the *ProductsApp* project's *App_Start/WebApiConfig.cs* file:</span></span>

[!code-csharp[](webapi/sample/2.x/ProductsCore/Startup.cs?name=snippet_Configure&highlight=13)]

## <a name="migrate-models-and-controllers"></a><span data-ttu-id="4014d-211">Eseguire la migrazione di modelli e controller</span><span class="sxs-lookup"><span data-stu-id="4014d-211">Migrate models and controllers</span></span>

<span data-ttu-id="4014d-212">Il codice seguente illustra l' `ProductsController` aggiornamento per ASP.NET Core: [!code-csharp[](webapi/sample/2.x/ProductsApp/Controllers/ProductsController.cs)]</span><span class="sxs-lookup"><span data-stu-id="4014d-212">The following code shows the `ProductsController` update for ASP.NET Core: [!code-csharp[](webapi/sample/2.x/ProductsApp/Controllers/ProductsController.cs)]</span></span>

<span data-ttu-id="4014d-213">Aggiornare `ProductsController` per ASP.NET Core:</span><span class="sxs-lookup"><span data-stu-id="4014d-213">Update the `ProductsController` for ASP.NET Core:</span></span>

1. <span data-ttu-id="4014d-214">Copiare *Controllers/ProductsController. cs* dal progetto originale a quello nuovo.</span><span class="sxs-lookup"><span data-stu-id="4014d-214">Copy *Controllers/ProductsController.cs* from the original project to the new one.</span></span>
1. <span data-ttu-id="4014d-215">Copiare la cartella *Models* dal progetto originale a quella nuova.</span><span class="sxs-lookup"><span data-stu-id="4014d-215">Copy the *Models* folder from the original project to the new one.</span></span>
1. <span data-ttu-id="4014d-216">Modificare lo spazio dei nomi radice dei file copiati in `ProductsCore` .</span><span class="sxs-lookup"><span data-stu-id="4014d-216">Change the copied files' root namespace to `ProductsCore`.</span></span>
1. <span data-ttu-id="4014d-217">Aggiornare l' `using ProductsApp.Models;` istruzione a `using ProductsCore.Models;` .</span><span class="sxs-lookup"><span data-stu-id="4014d-217">Update the `using ProductsApp.Models;` statement to `using ProductsCore.Models;`.</span></span>

<span data-ttu-id="4014d-218">I componenti seguenti non esistono in ASP.NET Core:</span><span class="sxs-lookup"><span data-stu-id="4014d-218">The following components don't exist in ASP.NET Core:</span></span>

* <span data-ttu-id="4014d-219">Classe `ApiController`</span><span class="sxs-lookup"><span data-stu-id="4014d-219">`ApiController` class</span></span>
* <span data-ttu-id="4014d-220">Spazio dei nomi `System.Web.Http`</span><span class="sxs-lookup"><span data-stu-id="4014d-220">`System.Web.Http` namespace</span></span>
* <span data-ttu-id="4014d-221">Interfaccia `IHttpActionResult`</span><span class="sxs-lookup"><span data-stu-id="4014d-221">`IHttpActionResult` interface</span></span>

<span data-ttu-id="4014d-222">Apportare le modifiche seguenti:</span><span class="sxs-lookup"><span data-stu-id="4014d-222">Make the following changes:</span></span>

1. <span data-ttu-id="4014d-223">Cambiare `ApiController` in <xref:Microsoft.AspNetCore.Mvc.ControllerBase>.</span><span class="sxs-lookup"><span data-stu-id="4014d-223">Change `ApiController` to <xref:Microsoft.AspNetCore.Mvc.ControllerBase>.</span></span> <span data-ttu-id="4014d-224">Aggiungere `using Microsoft.AspNetCore.Mvc;` per risolvere il `ControllerBase` riferimento.</span><span class="sxs-lookup"><span data-stu-id="4014d-224">Add `using Microsoft.AspNetCore.Mvc;` to resolve the `ControllerBase` reference.</span></span>
1. <span data-ttu-id="4014d-225">Eliminare `using System.Web.Http;`.</span><span class="sxs-lookup"><span data-stu-id="4014d-225">Delete `using System.Web.Http;`.</span></span>
1. <span data-ttu-id="4014d-226">Modificare il `GetProduct` tipo restituito dell'azione da `IHttpActionResult` a `ActionResult<Product>` .</span><span class="sxs-lookup"><span data-stu-id="4014d-226">Change the `GetProduct` action's return type from `IHttpActionResult` to `ActionResult<Product>`.</span></span>
1. <span data-ttu-id="4014d-227">Semplificare l' `GetProduct` istruzione dell'azione `return` a quanto segue:</span><span class="sxs-lookup"><span data-stu-id="4014d-227">Simplify the `GetProduct` action's `return` statement to the following:</span></span>

    ```csharp
    return product;
    ```

## <a name="configure-routing"></a><span data-ttu-id="4014d-228">Configurare il routing</span><span class="sxs-lookup"><span data-stu-id="4014d-228">Configure routing</span></span>

<span data-ttu-id="4014d-229">Configurare il routing come segue:</span><span class="sxs-lookup"><span data-stu-id="4014d-229">Configure routing as follows:</span></span>

1. <span data-ttu-id="4014d-230">Contrassegnare la `ProductsController` classe con gli attributi seguenti:</span><span class="sxs-lookup"><span data-stu-id="4014d-230">Mark the `ProductsController` class with the following attributes:</span></span>

    ```csharp
    [Route("api/[controller]")]
    [ApiController]
    ```

    <span data-ttu-id="4014d-231">L' [`[Route]`](xref:Microsoft.AspNetCore.Mvc.RouteAttribute) attributo precedente configura il modello di routing degli attributi del controller.</span><span class="sxs-lookup"><span data-stu-id="4014d-231">The preceding [`[Route]`](xref:Microsoft.AspNetCore.Mvc.RouteAttribute) attribute configures the controller's attribute routing pattern.</span></span> <span data-ttu-id="4014d-232">L' [`[ApiController]`](xref:Microsoft.AspNetCore.Mvc.ApiControllerAttribute) attributo rende il routing degli attributi un requisito per tutte le azioni nel controller.</span><span class="sxs-lookup"><span data-stu-id="4014d-232">The [`[ApiController]`](xref:Microsoft.AspNetCore.Mvc.ApiControllerAttribute) attribute makes attribute routing a requirement for all actions in this controller.</span></span>

    <span data-ttu-id="4014d-233">Il routing degli attributi supporta i token, ad esempio `[controller]` e `[action]` .</span><span class="sxs-lookup"><span data-stu-id="4014d-233">Attribute routing supports tokens, such as `[controller]` and `[action]`.</span></span> <span data-ttu-id="4014d-234">In fase di esecuzione, ogni token viene sostituito rispettivamente dal nome del controller o dell'azione a cui è stato applicato l'attributo.</span><span class="sxs-lookup"><span data-stu-id="4014d-234">At runtime, each token is replaced with the name of the controller or action, respectively, to which the attribute has been applied.</span></span> <span data-ttu-id="4014d-235">I token riducono il numero di stringhe magiche nel progetto.</span><span class="sxs-lookup"><span data-stu-id="4014d-235">The tokens reduce the number of magic strings in the project.</span></span> <span data-ttu-id="4014d-236">I token assicurano inoltre che le route rimangano sincronizzate con i controller e le azioni corrispondenti quando vengono applicati i refactoring di ridenominazione automatica.</span><span class="sxs-lookup"><span data-stu-id="4014d-236">The tokens also ensure routes remain synchronized with the corresponding controllers and actions when automatic rename refactorings are applied.</span></span>
1. <span data-ttu-id="4014d-237">Impostare la modalità di compatibilità del progetto su ASP.NET Core 2,2:</span><span class="sxs-lookup"><span data-stu-id="4014d-237">Set the project's compatibility mode to ASP.NET Core 2.2:</span></span>

    [!code-csharp[](webapi/sample/2.x/ProductsCore/Startup.cs?name=snippet_ConfigureServices&highlight=4)]

    <span data-ttu-id="4014d-238">La modifica precedente:</span><span class="sxs-lookup"><span data-stu-id="4014d-238">The preceding change:</span></span>

    * <span data-ttu-id="4014d-239">È necessario per usare l' `[ApiController]` attributo a livello di controller.</span><span class="sxs-lookup"><span data-stu-id="4014d-239">Is required to use the `[ApiController]` attribute at the controller level.</span></span>
    * <span data-ttu-id="4014d-240">Consente di optare per i comportamenti potenzialmente innovatori introdotti in ASP.NET Core 2,2.</span><span class="sxs-lookup"><span data-stu-id="4014d-240">Opts in to potentially breaking behaviors introduced in ASP.NET Core 2.2.</span></span>
1. <span data-ttu-id="4014d-241">Abilitare le richieste HTTP Get alle `ProductController` azioni:</span><span class="sxs-lookup"><span data-stu-id="4014d-241">Enable HTTP Get requests to the `ProductController` actions:</span></span>
    * <span data-ttu-id="4014d-242">Applicare l' [`[HttpGet]`](xref:Microsoft.AspNetCore.Mvc.HttpGetAttribute) attributo all' `GetAllProducts` azione.</span><span class="sxs-lookup"><span data-stu-id="4014d-242">Apply the [`[HttpGet]`](xref:Microsoft.AspNetCore.Mvc.HttpGetAttribute) attribute to the `GetAllProducts` action.</span></span>
    * <span data-ttu-id="4014d-243">Applicare l' `[HttpGet("{id}")]` attributo all' `GetProduct` azione.</span><span class="sxs-lookup"><span data-stu-id="4014d-243">Apply the `[HttpGet("{id}")]` attribute to the `GetProduct` action.</span></span>

<span data-ttu-id="4014d-244">Eseguire il progetto migrato e passare a `/api/products` .</span><span class="sxs-lookup"><span data-stu-id="4014d-244">Run the migrated project, and browse to `/api/products`.</span></span> <span data-ttu-id="4014d-245">Viene visualizzato un elenco completo di tre prodotti.</span><span class="sxs-lookup"><span data-stu-id="4014d-245">A full list of three products appears.</span></span> <span data-ttu-id="4014d-246">Passare a `/api/products/1`.</span><span class="sxs-lookup"><span data-stu-id="4014d-246">Browse to `/api/products/1`.</span></span> <span data-ttu-id="4014d-247">Il primo prodotto verrà visualizzato.</span><span class="sxs-lookup"><span data-stu-id="4014d-247">The first product appears.</span></span>

## <a name="compatibility-shim"></a><span data-ttu-id="4014d-248">Shim di compatibilità</span><span class="sxs-lookup"><span data-stu-id="4014d-248">Compatibility shim</span></span>

<span data-ttu-id="4014d-249">La libreria [Microsoft. AspNetCore. Mvc. WebApiCompatShim](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.WebApiCompatShim) fornisce uno shim di compatibilità per spostare i progetti API Web ASP.NET 4. x in ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="4014d-249">The [Microsoft.AspNetCore.Mvc.WebApiCompatShim](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.WebApiCompatShim) library provides a compatibility shim to move ASP.NET 4.x Web API projects to ASP.NET Core.</span></span> <span data-ttu-id="4014d-250">Lo shim di compatibilità estende ASP.NET Core per supportare una serie di convenzioni di ASP.NET 4. x Web API 2.</span><span class="sxs-lookup"><span data-stu-id="4014d-250">The compatibility shim extends ASP.NET Core to support a number of conventions from ASP.NET 4.x Web API 2.</span></span> <span data-ttu-id="4014d-251">L'esempio portato in precedenza in questo documento è abbastanza elementare che lo shim di compatibilità non fosse necessario.</span><span class="sxs-lookup"><span data-stu-id="4014d-251">The sample ported previously in this document is basic enough that the compatibility shim was unnecessary.</span></span> <span data-ttu-id="4014d-252">Per i progetti di grandi dimensioni, l'uso dello shim di compatibilità può essere utile per colmare temporaneamente il gap dell'API tra ASP.NET Core e ASP.NET 4. x Web API 2.</span><span class="sxs-lookup"><span data-stu-id="4014d-252">For larger projects, using the compatibility shim can be useful for temporarily bridging the API gap between ASP.NET Core and ASP.NET 4.x Web API 2.</span></span>

<span data-ttu-id="4014d-253">Lo shim di compatibilità dell'API Web è pensato per essere usato come misura temporanea per supportare la migrazione di progetti API Web ASP.NET 4. x di grandi dimensioni a ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="4014d-253">The Web API compatibility shim is meant to be used as a temporary measure to support migrating large ASP.NET 4.x Web API projects to ASP.NET Core.</span></span> <span data-ttu-id="4014d-254">Nel corso del tempo, è necessario aggiornare i progetti in modo da usare ASP.NET Core modelli anziché basarsi sullo shim di compatibilità.</span><span class="sxs-lookup"><span data-stu-id="4014d-254">Over time, projects should be updated to use ASP.NET Core patterns instead of relying on the compatibility shim.</span></span>

<span data-ttu-id="4014d-255">Le funzionalità di compatibilità incluse in `Microsoft.AspNetCore.Mvc.WebApiCompatShim` includono:</span><span class="sxs-lookup"><span data-stu-id="4014d-255">Compatibility features included in `Microsoft.AspNetCore.Mvc.WebApiCompatShim` include:</span></span>

* <span data-ttu-id="4014d-256">Aggiunge un `ApiController` tipo in modo che i tipi di base dei controller non debbano essere aggiornati.</span><span class="sxs-lookup"><span data-stu-id="4014d-256">Adds an `ApiController` type so that controllers' base types don't need to be updated.</span></span>
* <span data-ttu-id="4014d-257">Abilita l'associazione di modelli di tipo API Web.</span><span class="sxs-lookup"><span data-stu-id="4014d-257">Enables Web API-style model binding.</span></span> <span data-ttu-id="4014d-258">ASP.NET Core funzioni di associazione di modelli MVC in modo analogo a quello di ASP.NET 4. x MVC 5, per impostazione predefinita.</span><span class="sxs-lookup"><span data-stu-id="4014d-258">ASP.NET Core MVC model binding functions similarly to that of ASP.NET 4.x MVC 5, by default.</span></span> <span data-ttu-id="4014d-259">Lo shim di compatibilità modifica l'associazione di modelli in modo da essere più simile alle convenzioni di associazione del modello ASP.NET 4. x Web API 2.</span><span class="sxs-lookup"><span data-stu-id="4014d-259">The compatibility shim changes model binding to be more similar to ASP.NET 4.x Web API 2 model binding conventions.</span></span> <span data-ttu-id="4014d-260">Ad esempio, i tipi complessi vengono automaticamente associati dal corpo della richiesta.</span><span class="sxs-lookup"><span data-stu-id="4014d-260">For example, complex types are automatically bound from the request body.</span></span>
* <span data-ttu-id="4014d-261">Estende l'associazione di modelli in modo che le azioni del controller possano prendere parametri di tipo `HttpRequestMessage` .</span><span class="sxs-lookup"><span data-stu-id="4014d-261">Extends model binding so that controller actions can take parameters of type `HttpRequestMessage`.</span></span>
* <span data-ttu-id="4014d-262">Aggiunge formattatori di messaggi che consentono alle azioni di restituire risultati di tipo `HttpResponseMessage` .</span><span class="sxs-lookup"><span data-stu-id="4014d-262">Adds message formatters allowing actions to return results of type `HttpResponseMessage`.</span></span>
* <span data-ttu-id="4014d-263">Aggiunge metodi di risposta aggiuntivi che possono essere usati dalle azioni API Web 2 per rispondere alle risposte:</span><span class="sxs-lookup"><span data-stu-id="4014d-263">Adds additional response methods that Web API 2 actions may have used to serve responses:</span></span>
  * <span data-ttu-id="4014d-264">`HttpResponseMessage` generatori</span><span class="sxs-lookup"><span data-stu-id="4014d-264">`HttpResponseMessage` generators:</span></span>
    * `CreateResponse<T>`
    * `CreateErrorResponse`
  * <span data-ttu-id="4014d-265">Metodi di risultato dell'azione:</span><span class="sxs-lookup"><span data-stu-id="4014d-265">Action result methods:</span></span>
    * `BadRequestErrorMessageResult`
    * `ExceptionResult`
    * `InternalServerErrorResult`
    * `InvalidModelStateResult`
    * `NegotiatedContentResult`
    * `ResponseMessageResult`
* <span data-ttu-id="4014d-266">Aggiunge un'istanza di `IContentNegotiator` al contenitore di inserimento delle dipendenze dell'app e rende disponibili i tipi correlati alla negoziazione del contenuto da [Microsoft. AspNet. WebAPI. client](https://www.nuget.org/packages/Microsoft.AspNet.WebApi.Client/).</span><span class="sxs-lookup"><span data-stu-id="4014d-266">Adds an instance of `IContentNegotiator` to the app's dependency injection (DI) container and makes available the content negotiation-related types from [Microsoft.AspNet.WebApi.Client](https://www.nuget.org/packages/Microsoft.AspNet.WebApi.Client/).</span></span> <span data-ttu-id="4014d-267">Esempi di tali tipi includono `DefaultContentNegotiator` e `MediaTypeFormatter` .</span><span class="sxs-lookup"><span data-stu-id="4014d-267">Examples of such types include `DefaultContentNegotiator` and `MediaTypeFormatter`.</span></span>

<span data-ttu-id="4014d-268">Per usare lo shim di compatibilità:</span><span class="sxs-lookup"><span data-stu-id="4014d-268">To use the compatibility shim:</span></span>

1. <span data-ttu-id="4014d-269">Installare il pacchetto NuGet [Microsoft. AspNetCore. Mvc. WebApiCompatShim](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.WebApiCompatShim) .</span><span class="sxs-lookup"><span data-stu-id="4014d-269">Install the [Microsoft.AspNetCore.Mvc.WebApiCompatShim](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.WebApiCompatShim) NuGet package.</span></span>
1. <span data-ttu-id="4014d-270">Registrare i servizi dello shim di compatibilità con il contenitore DI dell'app chiamando `services.AddMvc().AddWebApiConventions()` in `Startup.ConfigureServices` .</span><span class="sxs-lookup"><span data-stu-id="4014d-270">Register the compatibility shim's services with the app's DI container by calling `services.AddMvc().AddWebApiConventions()` in `Startup.ConfigureServices`.</span></span>
1. <span data-ttu-id="4014d-271">Definire route specifiche dell'API Web usando nella `MapWebApiRoute` `IRouteBuilder` chiamata dell'app `IApplicationBuilder.UseMvc` .</span><span class="sxs-lookup"><span data-stu-id="4014d-271">Define web API-specific routes using `MapWebApiRoute` on the `IRouteBuilder` in the app's `IApplicationBuilder.UseMvc` call.</span></span>

## <a name="additional-resources"></a><span data-ttu-id="4014d-272">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="4014d-272">Additional resources</span></span>

* <xref:web-api/index>
* <xref:web-api/action-return-types>
* <xref:mvc/compatibility-version>
::: moniker-end
