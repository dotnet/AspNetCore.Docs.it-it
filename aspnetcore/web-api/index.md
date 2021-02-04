---
title: Creare API Web con ASP.NET Core
author: scottaddie
description: Informazioni di base sulla creazione di un'API Web in ASP.NET Core.
monikerRange: '>= aspnetcore-2.1'
ms.author: scaddie
ms.custom: mvc
ms.date: 07/20/2020
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
uid: web-api/index
ms.openlocfilehash: 03debcb211a8cb8f0ebd6d5e67fbc8de7b574e27
ms.sourcegitcommit: c1839f2992b003c92cd958244a2e0771ae928786
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/04/2021
ms.locfileid: "99551658"
---
# <a name="create-web-apis-with-aspnet-core"></a><span data-ttu-id="9fe42-103">Creare API Web con ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="9fe42-103">Create web APIs with ASP.NET Core</span></span>

<span data-ttu-id="9fe42-104">Di [Scott Addie](https://github.com/scottaddie) e [Tom Dykstra](https://github.com/tdykstra)</span><span class="sxs-lookup"><span data-stu-id="9fe42-104">By [Scott Addie](https://github.com/scottaddie) and [Tom Dykstra](https://github.com/tdykstra)</span></span>

<span data-ttu-id="9fe42-105">ASP.NET Core supporta la creazione di servizi RESTful, noti anche come API Web, con C#.</span><span class="sxs-lookup"><span data-stu-id="9fe42-105">ASP.NET Core supports creating RESTful services, also known as web APIs, using C#.</span></span> <span data-ttu-id="9fe42-106">Per gestire le richieste, un'API Web usa i controller.</span><span class="sxs-lookup"><span data-stu-id="9fe42-106">To handle requests, a web API uses controllers.</span></span> <span data-ttu-id="9fe42-107">I *controller* in un'API Web sono classi che derivano da `ControllerBase`.</span><span class="sxs-lookup"><span data-stu-id="9fe42-107">*Controllers* in a web API are classes that derive from `ControllerBase`.</span></span> <span data-ttu-id="9fe42-108">Questo articolo illustra come usare i controller per gestire le richieste API Web.</span><span class="sxs-lookup"><span data-stu-id="9fe42-108">This article shows how to use controllers for handling web API requests.</span></span>

<span data-ttu-id="9fe42-109">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/web-api/index/samples).</span><span class="sxs-lookup"><span data-stu-id="9fe42-109">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/web-api/index/samples).</span></span> <span data-ttu-id="9fe42-110">([Come scaricare](xref:index#how-to-download-a-sample)).</span><span class="sxs-lookup"><span data-stu-id="9fe42-110">([How to download](xref:index#how-to-download-a-sample)).</span></span>

## <a name="controllerbase-class"></a><span data-ttu-id="9fe42-111">Classe ControllerBase</span><span class="sxs-lookup"><span data-stu-id="9fe42-111">ControllerBase class</span></span>

<span data-ttu-id="9fe42-112">Un'API Web è costituita da una o più classi controller che derivano da <xref:Microsoft.AspNetCore.Mvc.ControllerBase> .</span><span class="sxs-lookup"><span data-stu-id="9fe42-112">A web API consists of one or more controller classes that derive from <xref:Microsoft.AspNetCore.Mvc.ControllerBase>.</span></span> <span data-ttu-id="9fe42-113">Il modello di progetto API Web fornisce un controller Starter:</span><span class="sxs-lookup"><span data-stu-id="9fe42-113">The web API project template provides a starter controller:</span></span>

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](index/samples/3.x/Controllers/WeatherForecastController.cs?name=snippet_ControllerSignature&highlight=3)]

::: moniker-end

::: moniker range="<= aspnetcore-2.2"

[!code-csharp[](index/samples/2.x/2.2/Controllers/ValuesController.cs?name=snippet_ControllerSignature&highlight=3)]

::: moniker-end

<span data-ttu-id="9fe42-114">Non creare un controller API Web tramite derivazione dalla classe <xref:Microsoft.AspNetCore.Mvc.Controller>.</span><span class="sxs-lookup"><span data-stu-id="9fe42-114">Don't create a web API controller by deriving from the <xref:Microsoft.AspNetCore.Mvc.Controller> class.</span></span> <span data-ttu-id="9fe42-115">La classe `Controller` deriva da `ControllerBase` e aggiunge il supporto per le visualizzazioni, pertanto è progettata per la gestione delle pagine Web e non per le richieste di API Web.</span><span class="sxs-lookup"><span data-stu-id="9fe42-115">`Controller` derives from `ControllerBase` and adds support for views, so it's for handling web pages, not web API requests.</span></span> <span data-ttu-id="9fe42-116">Si è verificata un'eccezione a questa regola: se si prevede di usare lo stesso controller per le visualizzazioni e le API Web, derivarlo da `Controller` .</span><span class="sxs-lookup"><span data-stu-id="9fe42-116">There's an exception to this rule: if you plan to use the same controller for both views and web APIs, derive it from `Controller`.</span></span>

<span data-ttu-id="9fe42-117">La classe `ControllerBase` offre molti metodi e proprietà utili per la gestione delle richieste HTTP.</span><span class="sxs-lookup"><span data-stu-id="9fe42-117">The `ControllerBase` class provides many properties and methods that are useful for handling HTTP requests.</span></span> <span data-ttu-id="9fe42-118">Ad esempio, `ControllerBase.CreatedAtAction` restituisce un codice di stato 201:</span><span class="sxs-lookup"><span data-stu-id="9fe42-118">For example, `ControllerBase.CreatedAtAction` returns a 201 status code:</span></span>

[!code-csharp[](index/samples/2.x/2.2/Controllers/PetsController.cs?name=snippet_400And201&highlight=10)]

<span data-ttu-id="9fe42-119">Di seguito sono elencati alcuni esempi di metodi forniti da `ControllerBase`.</span><span class="sxs-lookup"><span data-stu-id="9fe42-119">Here are some more examples of methods that `ControllerBase` provides.</span></span>

|<span data-ttu-id="9fe42-120">Metodo</span><span class="sxs-lookup"><span data-stu-id="9fe42-120">Method</span></span>   |<span data-ttu-id="9fe42-121">Note</span><span class="sxs-lookup"><span data-stu-id="9fe42-121">Notes</span></span>    |
|---------|---------|
|<xref:Microsoft.AspNetCore.Mvc.ControllerBase.BadRequest%2A>| <span data-ttu-id="9fe42-122">Restituisce il codice di stato 400.</span><span class="sxs-lookup"><span data-stu-id="9fe42-122">Returns 400 status code.</span></span>|
|<xref:Microsoft.AspNetCore.Mvc.ControllerBase.NotFound%2A>|<span data-ttu-id="9fe42-123">Restituisce il codice di stato 404.</span><span class="sxs-lookup"><span data-stu-id="9fe42-123">Returns 404 status code.</span></span>|
|<xref:Microsoft.AspNetCore.Mvc.ControllerBase.PhysicalFile%2A>|<span data-ttu-id="9fe42-124">Restituisce un file.</span><span class="sxs-lookup"><span data-stu-id="9fe42-124">Returns a file.</span></span>|
|<xref:Microsoft.AspNetCore.Mvc.ControllerBase.TryUpdateModelAsync%2A>|<span data-ttu-id="9fe42-125">Richiama l'[associazione di modelli](xref:mvc/models/model-binding).</span><span class="sxs-lookup"><span data-stu-id="9fe42-125">Invokes [model binding](xref:mvc/models/model-binding).</span></span>|
|<xref:Microsoft.AspNetCore.Mvc.ControllerBase.TryValidateModel%2A>|<span data-ttu-id="9fe42-126">Richiama la [convalida dei modelli](xref:mvc/models/validation).</span><span class="sxs-lookup"><span data-stu-id="9fe42-126">Invokes [model validation](xref:mvc/models/validation).</span></span>|

<span data-ttu-id="9fe42-127">Per un elenco di tutti i metodi e proprietà disponibili, vedere <xref:Microsoft.AspNetCore.Mvc.ControllerBase>.</span><span class="sxs-lookup"><span data-stu-id="9fe42-127">For a list of all available methods and properties, see <xref:Microsoft.AspNetCore.Mvc.ControllerBase>.</span></span>

## <a name="attributes"></a><span data-ttu-id="9fe42-128">Attributi</span><span class="sxs-lookup"><span data-stu-id="9fe42-128">Attributes</span></span>

<span data-ttu-id="9fe42-129">Lo spazio dei nomi <xref:Microsoft.AspNetCore.Mvc> include attributi che possono essere usati per configurare il comportamento del controller di API Web e i metodi di azione.</span><span class="sxs-lookup"><span data-stu-id="9fe42-129">The <xref:Microsoft.AspNetCore.Mvc> namespace provides attributes that can be used to configure the behavior of web API controllers and action methods.</span></span> <span data-ttu-id="9fe42-130">Nell'esempio seguente vengono utilizzati gli attributi per specificare il verbo di azione HTTP supportato e i codici di stato HTTP noti che possono essere restituiti:</span><span class="sxs-lookup"><span data-stu-id="9fe42-130">The following example uses attributes to specify the supported HTTP action verb and any known HTTP status codes that could be returned:</span></span>

[!code-csharp[](index/samples/2.x/2.2/Controllers/PetsController.cs?name=snippet_400And201&highlight=1-3)]

<span data-ttu-id="9fe42-131">Di seguito sono riportati altri esempi degli attributi disponibili.</span><span class="sxs-lookup"><span data-stu-id="9fe42-131">Here are some more examples of attributes that are available.</span></span>

|<span data-ttu-id="9fe42-132">Attributo</span><span class="sxs-lookup"><span data-stu-id="9fe42-132">Attribute</span></span>|<span data-ttu-id="9fe42-133">Note</span><span class="sxs-lookup"><span data-stu-id="9fe42-133">Notes</span></span>|
|---------|-----|
|[`[Route]`](<xref:Microsoft.AspNetCore.Mvc.RouteAttribute>)      |<span data-ttu-id="9fe42-134">Specifica il modello di URL per un controller o un'azione.</span><span class="sxs-lookup"><span data-stu-id="9fe42-134">Specifies URL pattern for a controller or action.</span></span>|
|[`[Bind]`](<xref:Microsoft.AspNetCore.Mvc.BindAttribute>)        |<span data-ttu-id="9fe42-135">Specifica il prefisso e le proprietà da includere per l'associazione di modelli.</span><span class="sxs-lookup"><span data-stu-id="9fe42-135">Specifies prefix and properties to include for model binding.</span></span>|
|[`[HttpGet]`](<xref:Microsoft.AspNetCore.Mvc.HttpGetAttribute>)  |<span data-ttu-id="9fe42-136">Identifica un'azione che supporta il verbo di azione HTTP GET.</span><span class="sxs-lookup"><span data-stu-id="9fe42-136">Identifies an action that supports the HTTP GET action verb.</span></span>|
|[`[Consumes]`](<xref:Microsoft.AspNetCore.Mvc.ConsumesAttribute>)|<span data-ttu-id="9fe42-137">Specifica i tipi di dati accettati da un'azione.</span><span class="sxs-lookup"><span data-stu-id="9fe42-137">Specifies data types that an action accepts.</span></span>|
|[`[Produces]`](<xref:Microsoft.AspNetCore.Mvc.ProducesAttribute>)|<span data-ttu-id="9fe42-138">Specifica i tipi di dati restituiti da un'azione.</span><span class="sxs-lookup"><span data-stu-id="9fe42-138">Specifies data types that an action returns.</span></span>|

<span data-ttu-id="9fe42-139">Per un elenco che include gli attributi disponibili, vedere lo spazio dei nomi <xref:Microsoft.AspNetCore.Mvc>.</span><span class="sxs-lookup"><span data-stu-id="9fe42-139">For a list that includes the available attributes, see the <xref:Microsoft.AspNetCore.Mvc> namespace.</span></span>

## <a name="apicontroller-attribute"></a><span data-ttu-id="9fe42-140">Attributo ApiController</span><span class="sxs-lookup"><span data-stu-id="9fe42-140">ApiController attribute</span></span>

<span data-ttu-id="9fe42-141">L' [`[ApiController]`](xref:Microsoft.AspNetCore.Mvc.ApiControllerAttribute) attributo può essere applicato a una classe controller per abilitare i seguenti comportamenti supponenti specifici dell'API:</span><span class="sxs-lookup"><span data-stu-id="9fe42-141">The [`[ApiController]`](xref:Microsoft.AspNetCore.Mvc.ApiControllerAttribute) attribute can be applied to a controller class to enable the following opinionated, API-specific behaviors:</span></span>

::: moniker range=">= aspnetcore-2.2"

* [<span data-ttu-id="9fe42-142">Requisiti del routing degli attributi</span><span class="sxs-lookup"><span data-stu-id="9fe42-142">Attribute routing requirement</span></span>](#attribute-routing-requirement)
* [<span data-ttu-id="9fe42-143">Risposte HTTP 400 automatiche</span><span class="sxs-lookup"><span data-stu-id="9fe42-143">Automatic HTTP 400 responses</span></span>](#automatic-http-400-responses)
* [<span data-ttu-id="9fe42-144">Inferenza del parametro di origine di associazione</span><span class="sxs-lookup"><span data-stu-id="9fe42-144">Binding source parameter inference</span></span>](#binding-source-parameter-inference)
* [<span data-ttu-id="9fe42-145">Inferenza di richieste multipart/form-data</span><span class="sxs-lookup"><span data-stu-id="9fe42-145">Multipart/form-data request inference</span></span>](#multipartform-data-request-inference)
* [<span data-ttu-id="9fe42-146">Dettagli del problema per i codici di stato di errore</span><span class="sxs-lookup"><span data-stu-id="9fe42-146">Problem details for error status codes</span></span>](#problem-details-for-error-status-codes)

<span data-ttu-id="9fe42-147">La funzionalità *Dettagli problema per i codici di stato di errore* richiede una [versione di compatibilità](xref:mvc/compatibility-version) 2,2 o successiva.</span><span class="sxs-lookup"><span data-stu-id="9fe42-147">The *Problem details for error status codes* feature requires a [compatibility version](xref:mvc/compatibility-version) of 2.2 or later.</span></span> <span data-ttu-id="9fe42-148">Le altre funzionalità richiedono una versione di compatibilità 2,1 o successiva.</span><span class="sxs-lookup"><span data-stu-id="9fe42-148">The other features require a compatibility version of 2.1 or later.</span></span>

::: moniker-end

* [<span data-ttu-id="9fe42-149">Requisiti del routing degli attributi</span><span class="sxs-lookup"><span data-stu-id="9fe42-149">Attribute routing requirement</span></span>](#attribute-routing-requirement)
* [<span data-ttu-id="9fe42-150">Risposte HTTP 400 automatiche</span><span class="sxs-lookup"><span data-stu-id="9fe42-150">Automatic HTTP 400 responses</span></span>](#automatic-http-400-responses)
* [<span data-ttu-id="9fe42-151">Inferenza del parametro di origine di associazione</span><span class="sxs-lookup"><span data-stu-id="9fe42-151">Binding source parameter inference</span></span>](#binding-source-parameter-inference)
* [<span data-ttu-id="9fe42-152">Inferenza di richieste multipart/form-data</span><span class="sxs-lookup"><span data-stu-id="9fe42-152">Multipart/form-data request inference</span></span>](#multipartform-data-request-inference)

<span data-ttu-id="9fe42-153">Queste funzionalità richiedono la [versione di compatibilità](xref:mvc/compatibility-version) 2.1 o successiva.</span><span class="sxs-lookup"><span data-stu-id="9fe42-153">These features require a [compatibility version](xref:mvc/compatibility-version) of 2.1 or later.</span></span>

### <a name="attribute-on-specific-controllers"></a><span data-ttu-id="9fe42-154">Attributo su controller specifici</span><span class="sxs-lookup"><span data-stu-id="9fe42-154">Attribute on specific controllers</span></span>

<span data-ttu-id="9fe42-155">L'attributo `[ApiController]` può essere applicato a controller specifici, come illustrato nell'esempio seguente dal modello di progetto:</span><span class="sxs-lookup"><span data-stu-id="9fe42-155">The `[ApiController]` attribute can be applied to specific controllers, as in the following example from the project template:</span></span>

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](index/samples/3.x/Controllers/WeatherForecastController.cs?name=snippet_ControllerSignature&highlight=2])]

::: moniker-end

::: moniker range="<= aspnetcore-2.2"

[!code-csharp[](index/samples/2.x/2.2/Controllers/ValuesController.cs?name=snippet_ControllerSignature&highlight=2)]

::: moniker-end

### <a name="attribute-on-multiple-controllers"></a><span data-ttu-id="9fe42-156">Attributo su più controller</span><span class="sxs-lookup"><span data-stu-id="9fe42-156">Attribute on multiple controllers</span></span>

<span data-ttu-id="9fe42-157">Un approccio per usare l'attributo su più di un controller consiste nel creare una classe di controller di base personalizzata annotata con l'attributo `[ApiController]`.</span><span class="sxs-lookup"><span data-stu-id="9fe42-157">One approach to using the attribute on more than one controller is to create a custom base controller class annotated with the `[ApiController]` attribute.</span></span> <span data-ttu-id="9fe42-158">Nell'esempio seguente viene illustrata una classe base personalizzata e un controller che deriva da esso:</span><span class="sxs-lookup"><span data-stu-id="9fe42-158">The following example shows a custom base class and a controller that derives from it:</span></span>

[!code-csharp[](index/samples/2.x/2.2/Controllers/MyControllerBase.cs?name=snippet_MyControllerBase)]

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](index/samples/3.x/Controllers/PetsController.cs?name=snippet_Inherit)]

::: moniker-end

::: moniker range="<= aspnetcore-2.2"

[!code-csharp[](index/samples/2.x/2.2/Controllers/PetsController.cs?name=snippet_Inherit)]

::: moniker-end

::: moniker range=">= aspnetcore-2.2"

### <a name="attribute-on-an-assembly"></a><span data-ttu-id="9fe42-159">Attributo in un assembly</span><span class="sxs-lookup"><span data-stu-id="9fe42-159">Attribute on an assembly</span></span>

<span data-ttu-id="9fe42-160">Se la [versione di compatibilità](xref:mvc/compatibility-version) è impostata su 2.2 o una versione successiva, l'attributo `[ApiController]` può essere applicato a un assembly.</span><span class="sxs-lookup"><span data-stu-id="9fe42-160">If [compatibility version](xref:mvc/compatibility-version) is set to 2.2 or later, the `[ApiController]` attribute can be applied to an assembly.</span></span> <span data-ttu-id="9fe42-161">L'uso delle annotazioni in questo modo applica il comportamento delle API Web a tutti i controller nell'assembly.</span><span class="sxs-lookup"><span data-stu-id="9fe42-161">Annotation in this manner applies web API behavior to all controllers in the assembly.</span></span> <span data-ttu-id="9fe42-162">Non esiste alcun modo per rifiutare esplicitamente singoli controller.</span><span class="sxs-lookup"><span data-stu-id="9fe42-162">There's no way to opt out for individual controllers.</span></span> <span data-ttu-id="9fe42-163">Applicare l'attributo a livello di assembly alla dichiarazione dello spazio dei nomi che circonda la `Startup` classe:</span><span class="sxs-lookup"><span data-stu-id="9fe42-163">Apply the assembly-level attribute to the namespace declaration surrounding the `Startup` class:</span></span>

```csharp
[assembly: ApiController]
namespace WebApiSample
{
    public class Startup
    {
        ...
    }
}
```

::: moniker-end

## <a name="attribute-routing-requirement"></a><span data-ttu-id="9fe42-164">Requisiti del routing degli attributi</span><span class="sxs-lookup"><span data-stu-id="9fe42-164">Attribute routing requirement</span></span>

<span data-ttu-id="9fe42-165">Con l'attributo `[ApiController]` il routing degli attributi è un requisito.</span><span class="sxs-lookup"><span data-stu-id="9fe42-165">The `[ApiController]` attribute makes attribute routing a requirement.</span></span> <span data-ttu-id="9fe42-166">Ad esempio:</span><span class="sxs-lookup"><span data-stu-id="9fe42-166">For example:</span></span>

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](index/samples/3.x/Controllers/WeatherForecastController.cs?name=snippet_ControllerSignature&highlight=2)]

<span data-ttu-id="9fe42-167">Le azioni sono inaccessibili tramite [Route convenzionali](xref:mvc/controllers/routing#conventional-routing) definite da `UseEndpoints` , <xref:Microsoft.AspNetCore.Builder.MvcApplicationBuilderExtensions.UseMvc%2A> o <xref:Microsoft.AspNetCore.Builder.MvcApplicationBuilderExtensions.UseMvcWithDefaultRoute%2A> in `Startup.Configure` .</span><span class="sxs-lookup"><span data-stu-id="9fe42-167">Actions are inaccessible via [conventional routes](xref:mvc/controllers/routing#conventional-routing) defined by `UseEndpoints`, <xref:Microsoft.AspNetCore.Builder.MvcApplicationBuilderExtensions.UseMvc%2A>, or <xref:Microsoft.AspNetCore.Builder.MvcApplicationBuilderExtensions.UseMvcWithDefaultRoute%2A> in `Startup.Configure`.</span></span>

::: moniker-end

::: moniker range="<= aspnetcore-2.2"

[!code-csharp[](index/samples/2.x/2.2/Controllers/ValuesController.cs?name=snippet_ControllerSignature&highlight=1)]

<span data-ttu-id="9fe42-168">Le azioni non sono accessibili tramite le [route convenzionali](xref:mvc/controllers/routing#conventional-routing) definite da <xref:Microsoft.AspNetCore.Builder.MvcApplicationBuilderExtensions.UseMvc%2A> o <xref:Microsoft.AspNetCore.Builder.MvcApplicationBuilderExtensions.UseMvcWithDefaultRoute%2A> in `Startup.Configure`.</span><span class="sxs-lookup"><span data-stu-id="9fe42-168">Actions are inaccessible via [conventional routes](xref:mvc/controllers/routing#conventional-routing) defined by <xref:Microsoft.AspNetCore.Builder.MvcApplicationBuilderExtensions.UseMvc%2A> or <xref:Microsoft.AspNetCore.Builder.MvcApplicationBuilderExtensions.UseMvcWithDefaultRoute%2A> in `Startup.Configure`.</span></span>

::: moniker-end

## <a name="automatic-http-400-responses"></a><span data-ttu-id="9fe42-169">Risposte HTTP 400 automatiche</span><span class="sxs-lookup"><span data-stu-id="9fe42-169">Automatic HTTP 400 responses</span></span>

<span data-ttu-id="9fe42-170">Con l'attributo `[ApiController]` gli errori di convalida del modello attivano automaticamente una risposta HTTP 400.</span><span class="sxs-lookup"><span data-stu-id="9fe42-170">The `[ApiController]` attribute makes model validation errors automatically trigger an HTTP 400 response.</span></span> <span data-ttu-id="9fe42-171">Di conseguenza è necessario il codice seguente in un metodo di azione:</span><span class="sxs-lookup"><span data-stu-id="9fe42-171">Consequently, the following code is unnecessary in an action method:</span></span>

```csharp
if (!ModelState.IsValid)
{
    return BadRequest(ModelState);
}
```

<span data-ttu-id="9fe42-172">ASP.NET Core MVC usa il <xref:Microsoft.AspNetCore.Mvc.Infrastructure.ModelStateInvalidFilter> filtro azioni per eseguire il controllo precedente.</span><span class="sxs-lookup"><span data-stu-id="9fe42-172">ASP.NET Core MVC uses the <xref:Microsoft.AspNetCore.Mvc.Infrastructure.ModelStateInvalidFilter> action filter to do the preceding check.</span></span>

### <a name="default-badrequest-response"></a><span data-ttu-id="9fe42-173">Risposta BadRequest predefinita</span><span class="sxs-lookup"><span data-stu-id="9fe42-173">Default BadRequest response</span></span>

<span data-ttu-id="9fe42-174">Con una versione di compatibilità 2,1, il tipo di risposta predefinito per una risposta HTTP 400 è <xref:Microsoft.AspNetCore.Mvc.SerializableError> .</span><span class="sxs-lookup"><span data-stu-id="9fe42-174">With a compatibility version of 2.1, the default response type for an HTTP 400 response is <xref:Microsoft.AspNetCore.Mvc.SerializableError>.</span></span> <span data-ttu-id="9fe42-175">Il corpo della richiesta seguente è un esempio del tipo serializzato:</span><span class="sxs-lookup"><span data-stu-id="9fe42-175">The following request body is an example of the serialized type:</span></span>

```json
{
  "": [
    "A non-empty request body is required."
  ]
}
```

::: moniker range=">= aspnetcore-2.2"

<span data-ttu-id="9fe42-176">Con una versione di compatibilità 2,2 o successiva, il tipo di risposta predefinito per una risposta HTTP 400 è <xref:Microsoft.AspNetCore.Mvc.ValidationProblemDetails> .</span><span class="sxs-lookup"><span data-stu-id="9fe42-176">With a compatibility version of 2.2 or later, the default response type for an HTTP 400 response is <xref:Microsoft.AspNetCore.Mvc.ValidationProblemDetails>.</span></span> <span data-ttu-id="9fe42-177">Il corpo della richiesta seguente è un esempio del tipo serializzato:</span><span class="sxs-lookup"><span data-stu-id="9fe42-177">The following request body is an example of the serialized type:</span></span>

```json
{
  "type": "https://tools.ietf.org/html/rfc7231#section-6.5.1",
  "title": "One or more validation errors occurred.",
  "status": 400,
  "traceId": "|7fb5e16a-4c8f23bbfc974667.",
  "errors": {
    "": [
      "A non-empty request body is required."
    ]
  }
}
```

<span data-ttu-id="9fe42-178">`ValidationProblemDetails`Tipo:</span><span class="sxs-lookup"><span data-stu-id="9fe42-178">The `ValidationProblemDetails` type:</span></span>

* <span data-ttu-id="9fe42-179">Fornisce un formato leggibile dal computer per specificare gli errori nelle risposte all'API Web.</span><span class="sxs-lookup"><span data-stu-id="9fe42-179">Provides a machine-readable format for specifying errors in web API responses.</span></span>
* <span data-ttu-id="9fe42-180">È conforme alla [specifica RFC 7807](https://tools.ietf.org/html/rfc7807).</span><span class="sxs-lookup"><span data-stu-id="9fe42-180">Complies with the [RFC 7807 specification](https://tools.ietf.org/html/rfc7807).</span></span>

::: moniker-end

<span data-ttu-id="9fe42-181">Per rendere coerenti le risposte automatiche e personalizzate, chiamare il <xref:Microsoft.AspNetCore.Mvc.ControllerBase.ValidationProblem%2A> metodo anziché <xref:System.Web.Http.ApiController.BadRequest%2A> .</span><span class="sxs-lookup"><span data-stu-id="9fe42-181">To make automatic and custom responses consistent, call the <xref:Microsoft.AspNetCore.Mvc.ControllerBase.ValidationProblem%2A> method instead of <xref:System.Web.Http.ApiController.BadRequest%2A>.</span></span> <span data-ttu-id="9fe42-182">`ValidationProblem` Restituisce un <xref:Microsoft.AspNetCore.Mvc.ValidationProblemDetails> oggetto e la risposta automatica.</span><span class="sxs-lookup"><span data-stu-id="9fe42-182">`ValidationProblem` returns a <xref:Microsoft.AspNetCore.Mvc.ValidationProblemDetails> object as well as the automatic response.</span></span>

### <a name="log-automatic-400-responses"></a><span data-ttu-id="9fe42-183">Registrare le risposte 400 automatiche</span><span class="sxs-lookup"><span data-stu-id="9fe42-183">Log automatic 400 responses</span></span>

<span data-ttu-id="9fe42-184">Vedere [come registrare risposte automatiche 400 sugli errori di convalida del modello (DotNet/AspNetCore.Docs # 12157)](https://github.com/dotnet/AspNetCore.Docs/issues/12157).</span><span class="sxs-lookup"><span data-stu-id="9fe42-184">See [How to log automatic 400 responses on model validation errors (dotnet/AspNetCore.Docs#12157)](https://github.com/dotnet/AspNetCore.Docs/issues/12157).</span></span>

### <a name="disable-automatic-400-response"></a><span data-ttu-id="9fe42-185">Disabilitare la risposta automatica 400</span><span class="sxs-lookup"><span data-stu-id="9fe42-185">Disable automatic 400 response</span></span>

<span data-ttu-id="9fe42-186">Per disabilitare il comportamento automatico per gli errori 400, impostare la proprietà <xref:Microsoft.AspNetCore.Mvc.ApiBehaviorOptions.SuppressModelStateInvalidFilter> su `true`.</span><span class="sxs-lookup"><span data-stu-id="9fe42-186">To disable the automatic 400 behavior, set the <xref:Microsoft.AspNetCore.Mvc.ApiBehaviorOptions.SuppressModelStateInvalidFilter> property to `true`.</span></span> <span data-ttu-id="9fe42-187">Aggiungere il codice evidenziato seguente in `Startup.ConfigureServices`:</span><span class="sxs-lookup"><span data-stu-id="9fe42-187">Add the following highlighted code in `Startup.ConfigureServices`:</span></span>

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](index/samples/3.x/Startup.cs?name=snippet_ConfigureApiBehaviorOptions&highlight=2,6)]

::: moniker-end

::: moniker range="= aspnetcore-2.2"

[!code-csharp[](index/samples/2.x/2.2/Startup.cs?name=snippet_ConfigureApiBehaviorOptions&highlight=3,7)]

::: moniker-end

::: moniker range="= aspnetcore-2.1"

[!code-csharp[](index/samples/2.x/2.1/Startup.cs?name=snippet_ConfigureApiBehaviorOptions&highlight=1,5)]

::: moniker-end

## <a name="binding-source-parameter-inference"></a><span data-ttu-id="9fe42-188">Inferenza del parametro di origine di associazione</span><span class="sxs-lookup"><span data-stu-id="9fe42-188">Binding source parameter inference</span></span>

<span data-ttu-id="9fe42-189">Un attributo di origine di associazione definisce la posizione in cui viene trovato il valore del parametro di un'azione.</span><span class="sxs-lookup"><span data-stu-id="9fe42-189">A binding source attribute defines the location at which an action parameter's value is found.</span></span> <span data-ttu-id="9fe42-190">Esistono gli attributi di origine di associazione seguente:</span><span class="sxs-lookup"><span data-stu-id="9fe42-190">The following binding source attributes exist:</span></span>

|<span data-ttu-id="9fe42-191">Attributo</span><span class="sxs-lookup"><span data-stu-id="9fe42-191">Attribute</span></span>|<span data-ttu-id="9fe42-192">Origine di binding</span><span class="sxs-lookup"><span data-stu-id="9fe42-192">Binding source</span></span> |
|---------|---------|
|[`[FromBody]`](xref:Microsoft.AspNetCore.Mvc.FromBodyAttribute)     | <span data-ttu-id="9fe42-193">Testo della richiesta</span><span class="sxs-lookup"><span data-stu-id="9fe42-193">Request body</span></span> |
|[`[FromForm]`](xref:Microsoft.AspNetCore.Mvc.FromFormAttribute)     | <span data-ttu-id="9fe42-194">Dati di modulo nel corpo della richiesta</span><span class="sxs-lookup"><span data-stu-id="9fe42-194">Form data in the request body</span></span> |
|[`[FromHeader]`](xref:Microsoft.AspNetCore.Mvc.FromHeaderAttribute) | <span data-ttu-id="9fe42-195">Intestazione della richiesta</span><span class="sxs-lookup"><span data-stu-id="9fe42-195">Request header</span></span> |
|[`[FromQuery]`](xref:Microsoft.AspNetCore.Mvc.FromQueryAttribute)   | <span data-ttu-id="9fe42-196">Parametri della stringa di query della richiesta</span><span class="sxs-lookup"><span data-stu-id="9fe42-196">Request query string parameter</span></span> |
|[`[FromRoute]`](xref:Microsoft.AspNetCore.Mvc.FromRouteAttribute)   | <span data-ttu-id="9fe42-197">Dati della route della richiesta corrente</span><span class="sxs-lookup"><span data-stu-id="9fe42-197">Route data from the current request</span></span> |
|[`[FromServices]`](xref:mvc/controllers/dependency-injection#action-injection-with-fromservices) | <span data-ttu-id="9fe42-198">Il servizio richiesta inserito come parametro di azione</span><span class="sxs-lookup"><span data-stu-id="9fe42-198">The request service injected as an action parameter</span></span> |

> [!WARNING]
> <span data-ttu-id="9fe42-199">Non usare `[FromRoute]` quando i valori potrebbero contenere `%2f` (vale a dire `/`).</span><span class="sxs-lookup"><span data-stu-id="9fe42-199">Don't use `[FromRoute]` when values might contain `%2f` (that is `/`).</span></span> <span data-ttu-id="9fe42-200">`%2f` non sarà convertito in `/` rimuovendo i caratteri di escape.</span><span class="sxs-lookup"><span data-stu-id="9fe42-200">`%2f` won't be unescaped to `/`.</span></span> <span data-ttu-id="9fe42-201">Usare `[FromQuery]` se il valore potrebbe contenere `%2f`.</span><span class="sxs-lookup"><span data-stu-id="9fe42-201">Use `[FromQuery]` if the value might contain `%2f`.</span></span>

<span data-ttu-id="9fe42-202">Senza l'attributo `[ApiController]` o altri attributi di origine di associazione come `[FromQuery]`, il runtime di ASP.NET Core tenta di usare lo strumento di associazione di modelli a oggetti complesso.</span><span class="sxs-lookup"><span data-stu-id="9fe42-202">Without the `[ApiController]` attribute or binding source attributes like `[FromQuery]`, the ASP.NET Core runtime attempts to use the complex object model binder.</span></span> <span data-ttu-id="9fe42-203">Lo strumento di associazione di modelli a oggetti complesso estrae i dati dal provider di valori in un ordine definito.</span><span class="sxs-lookup"><span data-stu-id="9fe42-203">The complex object model binder pulls data from value providers in a defined order.</span></span>

<span data-ttu-id="9fe42-204">Nell'esempio seguente, l'attributo `[FromQuery]` indica che il valore del parametro `discontinuedOnly` è specificato nella stringa di query dell'URL della richiesta:</span><span class="sxs-lookup"><span data-stu-id="9fe42-204">In the following example, the `[FromQuery]` attribute indicates that the `discontinuedOnly` parameter value is provided in the request URL's query string:</span></span>

[!code-csharp[](index/samples/2.x/2.2/Controllers/ProductsController.cs?name=snippet_BindingSourceAttributes&highlight=3)]

<span data-ttu-id="9fe42-205">L'attributo `[ApiController]` applica le regole di inferenza per le origini dati predefinite dei parametri di azione.</span><span class="sxs-lookup"><span data-stu-id="9fe42-205">The `[ApiController]` attribute applies inference rules for the default data sources of action parameters.</span></span> <span data-ttu-id="9fe42-206">Queste regole consentono di evitare di dover identificare le origini di associazione manualmente applicando attributi ai parametri di azione.</span><span class="sxs-lookup"><span data-stu-id="9fe42-206">These rules save you from having to identify binding sources manually by applying attributes to the action parameters.</span></span> <span data-ttu-id="9fe42-207">Il comportamento delle regole di inferenza per le origini di associazione è il seguente:</span><span class="sxs-lookup"><span data-stu-id="9fe42-207">The binding source inference rules behave as follows:</span></span>

* <span data-ttu-id="9fe42-208">`[FromBody]` viene dedotto per i parametri di tipo complesso.</span><span class="sxs-lookup"><span data-stu-id="9fe42-208">`[FromBody]` is inferred for complex type parameters.</span></span> <span data-ttu-id="9fe42-209">Un'eccezione alla regola di inferenza `[FromBody]` è costituita dai tipi predefiniti complessi con un significato speciale, ad esempio <xref:Microsoft.AspNetCore.Http.IFormCollection> e <xref:System.Threading.CancellationToken>.</span><span class="sxs-lookup"><span data-stu-id="9fe42-209">An exception to the `[FromBody]` inference rule is any complex, built-in type with a special meaning, such as <xref:Microsoft.AspNetCore.Http.IFormCollection> and <xref:System.Threading.CancellationToken>.</span></span> <span data-ttu-id="9fe42-210">Il codice di inferenza di origine di associazione ignora tali tipi speciali.</span><span class="sxs-lookup"><span data-stu-id="9fe42-210">The binding source inference code ignores those special types.</span></span>
* <span data-ttu-id="9fe42-211">`[FromForm]` viene dedotto per i parametri di azione di tipo <xref:Microsoft.AspNetCore.Http.IFormFile> e <xref:Microsoft.AspNetCore.Http.IFormFileCollection>.</span><span class="sxs-lookup"><span data-stu-id="9fe42-211">`[FromForm]` is inferred for action parameters of type <xref:Microsoft.AspNetCore.Http.IFormFile> and <xref:Microsoft.AspNetCore.Http.IFormFileCollection>.</span></span> <span data-ttu-id="9fe42-212">Non viene dedotto per i tipi semplici o definiti dall'utente.</span><span class="sxs-lookup"><span data-stu-id="9fe42-212">It's not inferred for any simple or user-defined types.</span></span>
* <span data-ttu-id="9fe42-213">`[FromRoute]` viene dedotto per i nomi di parametro di azione corrispondenti a un parametro nel modello di route.</span><span class="sxs-lookup"><span data-stu-id="9fe42-213">`[FromRoute]` is inferred for any action parameter name matching a parameter in the route template.</span></span> <span data-ttu-id="9fe42-214">Quando più di una route corrisponde a un parametro di azione, tutti i valori di route vengono considerati `[FromRoute]`.</span><span class="sxs-lookup"><span data-stu-id="9fe42-214">When more than one route matches an action parameter, any route value is considered `[FromRoute]`.</span></span>
* <span data-ttu-id="9fe42-215">`[FromQuery]` viene dedotto per tutti gli altri parametri di azione.</span><span class="sxs-lookup"><span data-stu-id="9fe42-215">`[FromQuery]` is inferred for any other action parameters.</span></span>

### <a name="frombody-inference-notes"></a><span data-ttu-id="9fe42-216">Note sulla regola di inferenza FromBody</span><span class="sxs-lookup"><span data-stu-id="9fe42-216">FromBody inference notes</span></span>

<span data-ttu-id="9fe42-217">`[FromBody]` non viene dedotto per i tipi semplici, ad esempio `string` o `int`.</span><span class="sxs-lookup"><span data-stu-id="9fe42-217">`[FromBody]` isn't inferred for simple types such as `string` or `int`.</span></span> <span data-ttu-id="9fe42-218">Pertanto, l'attributo `[FromBody]` deve essere usato per i tipi semplici se è necessaria tale funzionalità.</span><span class="sxs-lookup"><span data-stu-id="9fe42-218">Therefore, the `[FromBody]` attribute should be used for simple types when that functionality is needed.</span></span>

<span data-ttu-id="9fe42-219">Quando a un'azione sono associati più parametri dal corpo della richiesta, viene generata un'eccezione.</span><span class="sxs-lookup"><span data-stu-id="9fe42-219">When an action has more than one parameter bound from the request body, an exception is thrown.</span></span> <span data-ttu-id="9fe42-220">Tutte le firme di metodo di azione seguenti, ad esempio, causano un'eccezione:</span><span class="sxs-lookup"><span data-stu-id="9fe42-220">For example, all of the following action method signatures cause an exception:</span></span>

* <span data-ttu-id="9fe42-221">`[FromBody]` dedotto per entrambi, perché si tratta di tipi complessi.</span><span class="sxs-lookup"><span data-stu-id="9fe42-221">`[FromBody]` inferred on both because they're complex types.</span></span>

  ```csharp
  [HttpPost]
  public IActionResult Action1(Product product, Order order)
  ```

* <span data-ttu-id="9fe42-222">`[FromBody]` attributo per uno, dedotto per l'altro perché è un tipo complesso.</span><span class="sxs-lookup"><span data-stu-id="9fe42-222">`[FromBody]` attribute on one, inferred on the other because it's a complex type.</span></span>

  ```csharp
  [HttpPost]
  public IActionResult Action2(Product product, [FromBody] Order order)
  ```

* <span data-ttu-id="9fe42-223">`[FromBody]` attributo per entrambi.</span><span class="sxs-lookup"><span data-stu-id="9fe42-223">`[FromBody]` attribute on both.</span></span>

  ```csharp
  [HttpPost]
  public IActionResult Action3([FromBody] Product product, [FromBody] Order order)
  ```

::: moniker range="= aspnetcore-2.1"

> [!NOTE]
> <span data-ttu-id="9fe42-224">In ASP.NET Core 2.1, i parametri di tipo raccolta, come elenchi e matrici, vengono dedotti in modo errato come `[FromQuery]`.</span><span class="sxs-lookup"><span data-stu-id="9fe42-224">In ASP.NET Core 2.1, collection type parameters such as lists and arrays are incorrectly inferred as `[FromQuery]`.</span></span> <span data-ttu-id="9fe42-225">È necessario usare l'attributo `[FromBody]` per questi parametri, se devono essere associati dal corpo della richiesta.</span><span class="sxs-lookup"><span data-stu-id="9fe42-225">The `[FromBody]` attribute should be used for these parameters if they are to be bound from the request body.</span></span> <span data-ttu-id="9fe42-226">Questo comportamento è stato corretto in ASP.NET Core 2.2 o versioni successive, in cui i parametri di tipo raccolta vengono dedotti come associati dal corpo per impostazione predefinita.</span><span class="sxs-lookup"><span data-stu-id="9fe42-226">This behavior is corrected in ASP.NET Core 2.2 or later, where collection type parameters are inferred to be bound from the body by default.</span></span>

::: moniker-end

### <a name="disable-inference-rules"></a><span data-ttu-id="9fe42-227">Disabilitare le regole di inferenza</span><span class="sxs-lookup"><span data-stu-id="9fe42-227">Disable inference rules</span></span>

<span data-ttu-id="9fe42-228">Per disabilitare l'inferenza delle origini di associazione, impostare <xref:Microsoft.AspNetCore.Mvc.ApiBehaviorOptions.SuppressInferBindingSourcesForParameters> su `true`.</span><span class="sxs-lookup"><span data-stu-id="9fe42-228">To disable binding source inference, set <xref:Microsoft.AspNetCore.Mvc.ApiBehaviorOptions.SuppressInferBindingSourcesForParameters> to `true`.</span></span> <span data-ttu-id="9fe42-229">Aggiungere il codice seguente a `Startup.ConfigureServices`:</span><span class="sxs-lookup"><span data-stu-id="9fe42-229">Add the following code in `Startup.ConfigureServices`:</span></span>

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](index/samples/3.x/Startup.cs?name=snippet_ConfigureApiBehaviorOptions&highlight=2,5)]

::: moniker-end

::: moniker range="= aspnetcore-2.2"

[!code-csharp[](index/samples/2.x/2.2/Startup.cs?name=snippet_ConfigureApiBehaviorOptions&highlight=3,6)]

::: moniker-end

::: moniker range="= aspnetcore-2.1"

[!code-csharp[](index/samples/2.x/2.1/Startup.cs?name=snippet_ConfigureApiBehaviorOptions&highlight=1,4)]

::: moniker-end

## <a name="multipartform-data-request-inference"></a><span data-ttu-id="9fe42-230">Inferenza di richieste multipart/form-data</span><span class="sxs-lookup"><span data-stu-id="9fe42-230">Multipart/form-data request inference</span></span>

<span data-ttu-id="9fe42-231">L' `[ApiController]` attributo applica una regola di inferenza quando un parametro di azione viene annotato con l' [`[FromForm]`](xref:Microsoft.AspNetCore.Mvc.FromFormAttribute) attributo.</span><span class="sxs-lookup"><span data-stu-id="9fe42-231">The `[ApiController]` attribute applies an inference rule when an action parameter is annotated with the [`[FromForm]`](xref:Microsoft.AspNetCore.Mvc.FromFormAttribute) attribute.</span></span> <span data-ttu-id="9fe42-232">Il `multipart/form-data` tipo di contenuto della richiesta viene dedotto.</span><span class="sxs-lookup"><span data-stu-id="9fe42-232">The `multipart/form-data` request content type is inferred.</span></span>

<span data-ttu-id="9fe42-233">Per disabilitare il comportamento predefinito, impostare la <xref:Microsoft.AspNetCore.Mvc.ApiBehaviorOptions.SuppressConsumesConstraintForFormFileParameters> proprietà su `true` in `Startup.ConfigureServices` :</span><span class="sxs-lookup"><span data-stu-id="9fe42-233">To disable the default behavior, set the <xref:Microsoft.AspNetCore.Mvc.ApiBehaviorOptions.SuppressConsumesConstraintForFormFileParameters> property to `true` in `Startup.ConfigureServices`:</span></span>

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](index/samples/3.x/Startup.cs?name=snippet_ConfigureApiBehaviorOptions&highlight=2,4)]

::: moniker-end

::: moniker range="= aspnetcore-2.2"

[!code-csharp[](index/samples/2.x/2.2/Startup.cs?name=snippet_ConfigureApiBehaviorOptions&highlight=3,5)]

::: moniker-end

::: moniker range="= aspnetcore-2.1"

[!code-csharp[](index/samples/2.x/2.1/Startup.cs?name=snippet_ConfigureApiBehaviorOptions&highlight=1,3)]

::: moniker-end

::: moniker range=">= aspnetcore-2.2"

## <a name="problem-details-for-error-status-codes"></a><span data-ttu-id="9fe42-234">Dettagli del problema per i codici di stato di errore</span><span class="sxs-lookup"><span data-stu-id="9fe42-234">Problem details for error status codes</span></span>

<span data-ttu-id="9fe42-235">Con la versione di compatibilità 2.2 o successiva, MVC consente di trasformare un risultato di errore (un risultato con codice di stato 400 o superiore) in un risultato con <xref:Microsoft.AspNetCore.Mvc.ProblemDetails>.</span><span class="sxs-lookup"><span data-stu-id="9fe42-235">When the compatibility version is 2.2 or later, MVC transforms an error result (a result with status code 400 or higher) to a result with <xref:Microsoft.AspNetCore.Mvc.ProblemDetails>.</span></span> <span data-ttu-id="9fe42-236">Il tipo `ProblemDetails` si basa sulla [specifica RFC 7807](https://tools.ietf.org/html/rfc7807) per fornire dettagli sull'errore leggibili dal computer in una risposta HTTP.</span><span class="sxs-lookup"><span data-stu-id="9fe42-236">The `ProblemDetails` type is based on the [RFC 7807 specification](https://tools.ietf.org/html/rfc7807) for providing machine-readable error details in an HTTP response.</span></span>

<span data-ttu-id="9fe42-237">Si consideri il codice seguente in un'azione del controller:</span><span class="sxs-lookup"><span data-stu-id="9fe42-237">Consider the following code in a controller action:</span></span>

[!code-csharp[](index/samples/2.x/2.2/Controllers/PetsController.cs?name=snippet_ProblemDetailsStatusCode)]

<span data-ttu-id="9fe42-238">Il `NotFound` metodo produce un codice di stato HTTP 404 con un `ProblemDetails` corpo.</span><span class="sxs-lookup"><span data-stu-id="9fe42-238">The `NotFound` method produces an HTTP 404 status code with a `ProblemDetails` body.</span></span> <span data-ttu-id="9fe42-239">Ad esempio:</span><span class="sxs-lookup"><span data-stu-id="9fe42-239">For example:</span></span>

```json
{
  type: "https://tools.ietf.org/html/rfc7231#section-6.5.4",
  title: "Not Found",
  status: 404,
  traceId: "0HLHLV31KRN83:00000001"
}
```

### <a name="disable-problemdetails-response"></a><span data-ttu-id="9fe42-240">Disabilitare la risposta ProblemDetails</span><span class="sxs-lookup"><span data-stu-id="9fe42-240">Disable ProblemDetails response</span></span>

<span data-ttu-id="9fe42-241">La creazione automatica di un oggetto `ProblemDetails` per i codici di stato di errore è disabilitata quando la <xref:Microsoft.AspNetCore.Mvc.ApiBehaviorOptions.SuppressMapClientErrors%2A> proprietà è impostata su `true` .</span><span class="sxs-lookup"><span data-stu-id="9fe42-241">The automatic creation of a `ProblemDetails` for error status codes is disabled when the <xref:Microsoft.AspNetCore.Mvc.ApiBehaviorOptions.SuppressMapClientErrors%2A> property is set to `true`.</span></span> <span data-ttu-id="9fe42-242">Aggiungere il codice seguente a `Startup.ConfigureServices`:</span><span class="sxs-lookup"><span data-stu-id="9fe42-242">Add the following code in `Startup.ConfigureServices`:</span></span>

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](index/samples/3.x/Startup.cs?name=snippet_ConfigureApiBehaviorOptions&highlight=2,7)]

::: moniker-end

::: moniker range="= aspnetcore-2.2"

[!code-csharp[](index/samples/2.x/2.2/Startup.cs?name=snippet_ConfigureApiBehaviorOptions&highlight=3,8)]

::: moniker-end

<a name="consumes"></a>

## <a name="define-supported-request-content-types-with-the-consumes-attribute"></a><span data-ttu-id="9fe42-243">Definire i tipi di contenuto della richiesta supportati con l'attributo [consums]</span><span class="sxs-lookup"><span data-stu-id="9fe42-243">Define supported request content types with the [Consumes] attribute</span></span>

<span data-ttu-id="9fe42-244">Per impostazione predefinita, un'azione supporta tutti i tipi di contenuto della richiesta disponibili.</span><span class="sxs-lookup"><span data-stu-id="9fe42-244">By default, an action supports all available request content types.</span></span> <span data-ttu-id="9fe42-245">Se, ad esempio, un'app è configurata in modo da supportare i [formattatori di input](xref:mvc/models/model-binding#input-formatters)sia JSON che XML, un'azione supporta più tipi di contenuto, tra cui `application/json` e `application/xml` .</span><span class="sxs-lookup"><span data-stu-id="9fe42-245">For example, if an app is configured to support both JSON and XML [input formatters](xref:mvc/models/model-binding#input-formatters), an action supports multiple content types, including `application/json` and `application/xml`.</span></span>

<span data-ttu-id="9fe42-246">L'attributo [[consums]](<xref:Microsoft.AspNetCore.Mvc.ConsumesAttribute>) consente a un'azione di limitare i tipi di contenuto della richiesta supportati.</span><span class="sxs-lookup"><span data-stu-id="9fe42-246">The [[Consumes]](<xref:Microsoft.AspNetCore.Mvc.ConsumesAttribute>) attribute allows an action to limit the supported request content types.</span></span> <span data-ttu-id="9fe42-247">Applicare l' `[Consumes]` attributo a un'azione o a un controller, specificando uno o più tipi di contenuto:</span><span class="sxs-lookup"><span data-stu-id="9fe42-247">Apply the `[Consumes]` attribute to an action or controller, specifying one or more content types:</span></span>

```csharp
[HttpPost]
[Consumes("application/xml")]
public IActionResult CreateProduct(Product product)
```

<span data-ttu-id="9fe42-248">Nel codice precedente, l' `CreateProduct` azione specifica il tipo di contenuto `application/xml` .</span><span class="sxs-lookup"><span data-stu-id="9fe42-248">In the preceding code, the `CreateProduct` action specifies the content type `application/xml`.</span></span> <span data-ttu-id="9fe42-249">Le richieste indirizzate a questa azione devono specificare un' `Content-Type` intestazione `application/xml` .</span><span class="sxs-lookup"><span data-stu-id="9fe42-249">Requests routed to this action must specify a `Content-Type` header of `application/xml`.</span></span> <span data-ttu-id="9fe42-250">Le richieste che non specificano un' `Content-Type` intestazione di `application/xml` generano una risposta del [tipo di supporto 415 non supportato](https://developer.mozilla.org/docs/Web/HTTP/Status/415) .</span><span class="sxs-lookup"><span data-stu-id="9fe42-250">Requests that don't specify a `Content-Type` header of `application/xml` result in a [415 Unsupported Media Type](https://developer.mozilla.org/docs/Web/HTTP/Status/415) response.</span></span>

<span data-ttu-id="9fe42-251">L' `[Consumes]` attributo consente anche a un'azione di influenzare la selezione in base al tipo di contenuto di una richiesta in ingresso applicando un vincolo di tipo.</span><span class="sxs-lookup"><span data-stu-id="9fe42-251">The `[Consumes]` attribute also allows an action to influence its selection based on an incoming request's content type by applying a type constraint.</span></span> <span data-ttu-id="9fe42-252">Si consideri l'esempio seguente:</span><span class="sxs-lookup"><span data-stu-id="9fe42-252">Consider the following example:</span></span>

[!code-csharp[](index/samples/3.x/Controllers/ConsumesController.cs?name=snippet_Class)]

<span data-ttu-id="9fe42-253">Nel codice precedente, `ConsumesController` è configurato per gestire le richieste inviate all' `https://localhost:5001/api/Consumes` URL.</span><span class="sxs-lookup"><span data-stu-id="9fe42-253">In the preceding code, `ConsumesController` is configured to handle requests sent to the `https://localhost:5001/api/Consumes` URL.</span></span> <span data-ttu-id="9fe42-254">Entrambe le azioni del controller, `PostJson` e `PostForm` , gestiscono le richieste post con lo stesso URL.</span><span class="sxs-lookup"><span data-stu-id="9fe42-254">Both of the controller's actions, `PostJson` and `PostForm`, handle POST requests with the same URL.</span></span> <span data-ttu-id="9fe42-255">Senza l' `[Consumes]` attributo che applica un vincolo di tipo, viene generata un'eccezione di corrispondenza ambigua.</span><span class="sxs-lookup"><span data-stu-id="9fe42-255">Without the `[Consumes]` attribute applying a type constraint, an ambiguous match exception is thrown.</span></span>

<span data-ttu-id="9fe42-256">L' `[Consumes]` attributo viene applicato a entrambe le azioni.</span><span class="sxs-lookup"><span data-stu-id="9fe42-256">The `[Consumes]` attribute is applied to both actions.</span></span> <span data-ttu-id="9fe42-257">L' `PostJson` azione gestisce le richieste inviate con un' `Content-Type` intestazione `application/json` .</span><span class="sxs-lookup"><span data-stu-id="9fe42-257">The `PostJson` action handles requests sent with a `Content-Type` header of `application/json`.</span></span> <span data-ttu-id="9fe42-258">L' `PostForm` azione gestisce le richieste inviate con un' `Content-Type` intestazione `application/x-www-form-urlencoded` .</span><span class="sxs-lookup"><span data-stu-id="9fe42-258">The `PostForm` action handles requests sent with a `Content-Type` header of `application/x-www-form-urlencoded`.</span></span> 

## <a name="additional-resources"></a><span data-ttu-id="9fe42-259">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="9fe42-259">Additional resources</span></span>

* <xref:web-api/action-return-types>
* <xref:web-api/handle-errors>
* <xref:web-api/advanced/custom-formatters>
* <xref:web-api/advanced/formatting>
* <xref:tutorials/web-api-help-pages-using-swagger>
* <xref:mvc/controllers/routing>
* [<span data-ttu-id="9fe42-260">Microsoft Learn: creare un'API Web con ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="9fe42-260">Microsoft Learn: Create a web API with ASP.NET Core</span></span>](/learn/modules/build-web-api-aspnet-core/)
