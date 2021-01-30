---
title: Formattare i dati di risposta nell'API Web ASP.NET Core
author: rick-anderson
description: Informazioni su come formattare i dati di risposta nell'API Web ASP.NET Core.
ms.author: riande
ms.custom: H1Hack27Feb2017
ms.date: 1/28/2021
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
uid: web-api/advanced/formatting
ms.openlocfilehash: 5d228af00ee34e7f8ca60a5085872fdb93842367
ms.sourcegitcommit: 83524f739dd25fbfa95ee34e95342afb383b49fe
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 01/29/2021
ms.locfileid: "99057499"
---
# <a name="format-response-data-in-aspnet-core-web-api"></a><span data-ttu-id="291ab-103">Formattare i dati di risposta nell'API Web ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="291ab-103">Format response data in ASP.NET Core Web API</span></span>

<span data-ttu-id="291ab-104">Di [Rick Anderson](https://twitter.com/RickAndMSFT) e [Steve Smith](https://ardalis.com/)</span><span class="sxs-lookup"><span data-stu-id="291ab-104">By [Rick Anderson](https://twitter.com/RickAndMSFT) and [Steve Smith](https://ardalis.com/)</span></span>

<span data-ttu-id="291ab-105">ASP.NET Core MVC supporta la formattazione dei dati di risposta.</span><span class="sxs-lookup"><span data-stu-id="291ab-105">ASP.NET Core MVC has support for formatting response data.</span></span> <span data-ttu-id="291ab-106">I dati di risposta possono essere formattati usando formati specifici o in risposta al formato richiesto dal client.</span><span class="sxs-lookup"><span data-stu-id="291ab-106">Response data can be formatted using specific formats or in response to client requested format.</span></span>

<span data-ttu-id="291ab-107">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/web-api/advanced/formatting) ([procedura per il download](xref:index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="291ab-107">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/web-api/advanced/formatting) ([how to download](xref:index#how-to-download-a-sample))</span></span>

## <a name="format-specific-action-results"></a><span data-ttu-id="291ab-108">Risultati azione specifici del formato</span><span class="sxs-lookup"><span data-stu-id="291ab-108">Format-specific Action Results</span></span>

<span data-ttu-id="291ab-109">Alcuni tipi di risultati di azioni sono specifici di un particolare formato, ad esempio <xref:Microsoft.AspNetCore.Mvc.JsonResult> e <xref:Microsoft.AspNetCore.Mvc.ContentResult>.</span><span class="sxs-lookup"><span data-stu-id="291ab-109">Some action result types are specific to a particular format, such as <xref:Microsoft.AspNetCore.Mvc.JsonResult> and <xref:Microsoft.AspNetCore.Mvc.ContentResult>.</span></span> <span data-ttu-id="291ab-110">Le azioni possono restituire risultati formattati in un formato specifico, indipendentemente dalle preferenze dei client.</span><span class="sxs-lookup"><span data-stu-id="291ab-110">Actions can return results that are formatted in a particular format, regardless of client preferences.</span></span> <span data-ttu-id="291ab-111">Ad esempio, la restituzione di `JsonResult` restituisce dati in formato JSON.</span><span class="sxs-lookup"><span data-stu-id="291ab-111">For example, returning `JsonResult` returns JSON-formatted data.</span></span> <span data-ttu-id="291ab-112">La restituzione `ContentResult` di o di una stringa restituisce dati di stringa in formato testo normale.</span><span class="sxs-lookup"><span data-stu-id="291ab-112">Returning `ContentResult` or a string returns plain-text-formatted string data.</span></span>

<span data-ttu-id="291ab-113">Non è necessario eseguire un'azione per restituire un tipo specifico.</span><span class="sxs-lookup"><span data-stu-id="291ab-113">An action isn't required to return any specific type.</span></span> <span data-ttu-id="291ab-114">ASP.NET Core supporta qualsiasi valore restituito da un oggetto.</span><span class="sxs-lookup"><span data-stu-id="291ab-114">ASP.NET Core supports any object return value.</span></span>  <span data-ttu-id="291ab-115">I risultati di azioni che restituiscono oggetti che non sono <xref:Microsoft.AspNetCore.Mvc.IActionResult> tipi vengono serializzati usando l' <xref:Microsoft.AspNetCore.Mvc.Formatters.IOutputFormatter> implementazione appropriata.</span><span class="sxs-lookup"><span data-stu-id="291ab-115">Results from actions that return objects that are not <xref:Microsoft.AspNetCore.Mvc.IActionResult> types are serialized using the appropriate <xref:Microsoft.AspNetCore.Mvc.Formatters.IOutputFormatter> implementation.</span></span> <span data-ttu-id="291ab-116">Per altre informazioni, vedere <xref:web-api/action-return-types>.</span><span class="sxs-lookup"><span data-stu-id="291ab-116">For more information, see <xref:web-api/action-return-types>.</span></span>

<span data-ttu-id="291ab-117">Il metodo helper incorporato <xref:Microsoft.AspNetCore.Mvc.ControllerBase.Ok*> restituisce dati in formato JSON: [!code-csharp[](./formatting/sample/Controllers/AuthorsController.cs?name=snippet_get)]</span><span class="sxs-lookup"><span data-stu-id="291ab-117">The built-in helper method <xref:Microsoft.AspNetCore.Mvc.ControllerBase.Ok*> returns JSON-formatted data: [!code-csharp[](./formatting/sample/Controllers/AuthorsController.cs?name=snippet_get)]</span></span>

<span data-ttu-id="291ab-118">Il download di esempio restituisce l'elenco degli autori.</span><span class="sxs-lookup"><span data-stu-id="291ab-118">The sample download returns the list of authors.</span></span> <span data-ttu-id="291ab-119">Utilizzando gli strumenti di sviluppo del browser [F12 o l'](https://www.getpostman.com/tools) autore del codice precedente:</span><span class="sxs-lookup"><span data-stu-id="291ab-119">Using the F12 browser developer tools or [Postman](https://www.getpostman.com/tools) with the previous code:</span></span>

* <span data-ttu-id="291ab-120">Viene visualizzata l'intestazione della risposta contenente **Content-Type:** `application/json; charset=utf-8` .</span><span class="sxs-lookup"><span data-stu-id="291ab-120">The response header containing **content-type:** `application/json; charset=utf-8` is displayed.</span></span>
* <span data-ttu-id="291ab-121">Verranno visualizzate le intestazioni della richiesta.</span><span class="sxs-lookup"><span data-stu-id="291ab-121">The request headers are displayed.</span></span> <span data-ttu-id="291ab-122">Ad esempio, l' `Accept` intestazione.</span><span class="sxs-lookup"><span data-stu-id="291ab-122">For example, the `Accept` header.</span></span> <span data-ttu-id="291ab-123">L' `Accept` intestazione viene ignorata dal codice precedente.</span><span class="sxs-lookup"><span data-stu-id="291ab-123">The `Accept` header is ignored by the preceding code.</span></span>

<span data-ttu-id="291ab-124">Per restituire dati in formato testo normale, usare <xref:Microsoft.AspNetCore.Mvc.ContentResult> e l'helper <xref:Microsoft.AspNetCore.Mvc.ControllerBase.Content%2A>:</span><span class="sxs-lookup"><span data-stu-id="291ab-124">To return plain text formatted data, use <xref:Microsoft.AspNetCore.Mvc.ContentResult> and the <xref:Microsoft.AspNetCore.Mvc.ControllerBase.Content%2A> helper:</span></span>

[!code-csharp[](./formatting/sample/Controllers/AuthorsController.cs?name=snippet_about)]

<span data-ttu-id="291ab-125">Nel codice precedente, l'oggetto `Content-Type` restituito è `text/plain` .</span><span class="sxs-lookup"><span data-stu-id="291ab-125">In the preceding code, the `Content-Type` returned is `text/plain`.</span></span> <span data-ttu-id="291ab-126">La restituzione di una stringa fornisce `Content-Type` `text/plain` :</span><span class="sxs-lookup"><span data-stu-id="291ab-126">Returning a string delivers `Content-Type` of `text/plain`:</span></span>

[!code-csharp[](./formatting/sample/Controllers/AuthorsController.cs?name=snippet_string)]

<span data-ttu-id="291ab-127">Per le azioni con più tipi restituiti, restituire `IActionResult` .</span><span class="sxs-lookup"><span data-stu-id="291ab-127">For actions with multiple return types, return `IActionResult`.</span></span> <span data-ttu-id="291ab-128">Ad esempio, la restituzione di codici di stato HTTP diversi in base al risultato delle operazioni eseguite.</span><span class="sxs-lookup"><span data-stu-id="291ab-128">For example, returning different HTTP status codes based on the result of operations performed.</span></span>

## <a name="content-negotiation"></a><span data-ttu-id="291ab-129">Negoziazione del contenuto</span><span class="sxs-lookup"><span data-stu-id="291ab-129">Content negotiation</span></span>

<span data-ttu-id="291ab-130">La negoziazione del contenuto si verifica quando il client specifica un' [intestazione Accept](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html).</span><span class="sxs-lookup"><span data-stu-id="291ab-130">Content negotiation occurs when the client specifies an [Accept header](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html).</span></span> <span data-ttu-id="291ab-131">Il formato predefinito usato da ASP.NET Core è [JSON](https://json.org/).</span><span class="sxs-lookup"><span data-stu-id="291ab-131">The default format used by ASP.NET Core is [JSON](https://json.org/).</span></span> <span data-ttu-id="291ab-132">Negoziazione del contenuto:</span><span class="sxs-lookup"><span data-stu-id="291ab-132">Content negotiation is:</span></span>

* <span data-ttu-id="291ab-133">Implementato da <xref:Microsoft.AspNetCore.Mvc.ObjectResult> .</span><span class="sxs-lookup"><span data-stu-id="291ab-133">Implemented by <xref:Microsoft.AspNetCore.Mvc.ObjectResult>.</span></span>
* <span data-ttu-id="291ab-134">Incorporata nei risultati dell'azione specifici del codice di stato restituiti dai metodi helper.</span><span class="sxs-lookup"><span data-stu-id="291ab-134">Built into the status code-specific action results returned from the helper methods.</span></span> <span data-ttu-id="291ab-135">I metodi helper dei risultati delle azioni sono basati su `ObjectResult` .</span><span class="sxs-lookup"><span data-stu-id="291ab-135">The action results helper methods are based on `ObjectResult`.</span></span>

<span data-ttu-id="291ab-136">Quando viene restituito un tipo di modello, il tipo restituito è `ObjectResult` .</span><span class="sxs-lookup"><span data-stu-id="291ab-136">When a model type is returned,  the return type is `ObjectResult`.</span></span>

<span data-ttu-id="291ab-137">Il metodo di azione seguente usa i metodi helper `Ok` e `NotFound`:</span><span class="sxs-lookup"><span data-stu-id="291ab-137">The following action method uses the `Ok` and `NotFound` helper methods:</span></span>

[!code-csharp[](./formatting/sample/Controllers/AuthorsController.cs?name=snippet_search)]

<span data-ttu-id="291ab-138">Per impostazione predefinita, ASP.NET Core supporta i `application/json` `text/json` tipi di supporto, e `text/plain` .</span><span class="sxs-lookup"><span data-stu-id="291ab-138">By default, ASP.NET Core supports `application/json`, `text/json`, and `text/plain` media types.</span></span> <span data-ttu-id="291ab-139">Gli strumenti, ad esempio [Fiddler](https://www.telerik.com/fiddler) o [post](https://www.getpostman.com/tools) , possono impostare l' `Accept` intestazione della richiesta per specificare il formato restituito.</span><span class="sxs-lookup"><span data-stu-id="291ab-139">Tools such as [Fiddler](https://www.telerik.com/fiddler) or [Postman](https://www.getpostman.com/tools) can set the `Accept` request header to specify the return format.</span></span> <span data-ttu-id="291ab-140">Quando l' `Accept` intestazione contiene un tipo supportato dal server, viene restituito tale tipo.</span><span class="sxs-lookup"><span data-stu-id="291ab-140">When the `Accept` header contains a type the server supports, that type is returned.</span></span> <span data-ttu-id="291ab-141">Nella sezione successiva viene illustrato come aggiungere formattatori aggiuntivi.</span><span class="sxs-lookup"><span data-stu-id="291ab-141">The next section shows how to add additional formatters.</span></span>

<span data-ttu-id="291ab-142">Le azioni del controller possono restituire POCO (Plain Old CLR Objects).</span><span class="sxs-lookup"><span data-stu-id="291ab-142">Controller actions can return POCOs (Plain Old CLR Objects).</span></span> <span data-ttu-id="291ab-143">Quando viene restituito un oggetto POCO, il runtime crea automaticamente un `ObjectResult` oggetto che esegue il wrapping dell'oggetto.</span><span class="sxs-lookup"><span data-stu-id="291ab-143">When a POCO is returned, the runtime automatically creates an `ObjectResult` that wraps the object.</span></span> <span data-ttu-id="291ab-144">Il client ottiene l'oggetto serializzato formattato.</span><span class="sxs-lookup"><span data-stu-id="291ab-144">The client gets the formatted serialized object.</span></span> <span data-ttu-id="291ab-145">Se l'oggetto restituito è `null` , `204 No Content` viene restituita una risposta.</span><span class="sxs-lookup"><span data-stu-id="291ab-145">If the object being returned is `null`, a `204 No Content` response is returned.</span></span>

<span data-ttu-id="291ab-146">Restituzione di un tipo di oggetto:</span><span class="sxs-lookup"><span data-stu-id="291ab-146">Returning an object type:</span></span>

[!code-csharp[](./formatting/sample/Controllers/AuthorsController.cs?name=snippet_alias)]

<span data-ttu-id="291ab-147">Nel codice precedente, una richiesta di un alias autore valido restituisce una `200 OK` risposta con i dati dell'autore.</span><span class="sxs-lookup"><span data-stu-id="291ab-147">In the preceding code, a request for a valid author alias returns a `200 OK` response with the author's data.</span></span> <span data-ttu-id="291ab-148">Una richiesta di un alias non valido restituisce una `204 No Content` risposta.</span><span class="sxs-lookup"><span data-stu-id="291ab-148">A request for an invalid alias returns a `204 No Content` response.</span></span>

### <a name="the-accept-header"></a><span data-ttu-id="291ab-149">Intestazione Accept</span><span class="sxs-lookup"><span data-stu-id="291ab-149">The Accept header</span></span>

<span data-ttu-id="291ab-150">La *negoziazione* del contenuto si verifica quando `Accept` viene visualizzata un'intestazione nella richiesta.</span><span class="sxs-lookup"><span data-stu-id="291ab-150">Content *negotiation* takes place when an `Accept` header appears in the request.</span></span> <span data-ttu-id="291ab-151">Quando una richiesta contiene un'intestazione Accept, ASP.NET Core:</span><span class="sxs-lookup"><span data-stu-id="291ab-151">When a request contains an accept header, ASP.NET Core:</span></span>

* <span data-ttu-id="291ab-152">Enumera i tipi di supporto nell'intestazione Accept in ordine di preferenza.</span><span class="sxs-lookup"><span data-stu-id="291ab-152">Enumerates the media types in the accept header in preference order.</span></span>
* <span data-ttu-id="291ab-153">Tenta di trovare un formattatore in grado di generare una risposta in uno dei formati specificati.</span><span class="sxs-lookup"><span data-stu-id="291ab-153">Tries to find a formatter that can produce a response in one of the formats specified.</span></span>

<span data-ttu-id="291ab-154">Se non viene trovato alcun formattatore in grado di soddisfare la richiesta del client, ASP.NET Core:</span><span class="sxs-lookup"><span data-stu-id="291ab-154">If no formatter is found that can satisfy the client's request, ASP.NET Core:</span></span>

* <span data-ttu-id="291ab-155">Restituisce `406 Not Acceptable` se <xref:Microsoft.AspNetCore.Mvc.MvcOptions.ReturnHttpNotAcceptable?displayProperty=nameWithType> è impostato su `true` , o-</span><span class="sxs-lookup"><span data-stu-id="291ab-155">Returns `406 Not Acceptable` if <xref:Microsoft.AspNetCore.Mvc.MvcOptions.ReturnHttpNotAcceptable?displayProperty=nameWithType> is set to `true`, or -</span></span>
* <span data-ttu-id="291ab-156">Tenta di trovare il primo formattatore in grado di generare una risposta.</span><span class="sxs-lookup"><span data-stu-id="291ab-156">Tries to find the first formatter that can produce a response.</span></span>

<span data-ttu-id="291ab-157">Se non è configurato alcun formattatore per il formato richiesto, viene utilizzato il primo formattatore in grado di formattare l'oggetto.</span><span class="sxs-lookup"><span data-stu-id="291ab-157">If no formatter is configured for the requested format, the first formatter that can format the object is used.</span></span> <span data-ttu-id="291ab-158">Se `Accept` nella richiesta non è presente alcuna intestazione:</span><span class="sxs-lookup"><span data-stu-id="291ab-158">If no `Accept` header appears in the request:</span></span>

* <span data-ttu-id="291ab-159">Il primo formattatore in grado di gestire l'oggetto viene utilizzato per serializzare la risposta.</span><span class="sxs-lookup"><span data-stu-id="291ab-159">The first formatter that can handle the object is used to serialize the response.</span></span>
* <span data-ttu-id="291ab-160">Non si verifica alcuna negoziazione.</span><span class="sxs-lookup"><span data-stu-id="291ab-160">There isn't any negotiation taking place.</span></span> <span data-ttu-id="291ab-161">Il server sta determinando il formato da restituire.</span><span class="sxs-lookup"><span data-stu-id="291ab-161">The server is determining what format to return.</span></span>

<span data-ttu-id="291ab-162">Se l'intestazione Accept contiene `*/*` , l'intestazione viene ignorata, a meno che non `RespectBrowserAcceptHeader` sia impostato su true in <xref:Microsoft.AspNetCore.Mvc.MvcOptions> .</span><span class="sxs-lookup"><span data-stu-id="291ab-162">If the Accept header contains `*/*`, the Header is ignored unless `RespectBrowserAcceptHeader` is set to true on <xref:Microsoft.AspNetCore.Mvc.MvcOptions>.</span></span>

### <a name="browsers-and-content-negotiation"></a><span data-ttu-id="291ab-163">Browser e negoziazione del contenuto</span><span class="sxs-lookup"><span data-stu-id="291ab-163">Browsers and content negotiation</span></span>

<span data-ttu-id="291ab-164">A differenza dei client API tipici, i Web browser forniscono le `Accept` intestazioni.</span><span class="sxs-lookup"><span data-stu-id="291ab-164">Unlike typical API clients, web browsers supply `Accept` headers.</span></span> <span data-ttu-id="291ab-165">Web browser specificare molti formati, inclusi i caratteri jolly.</span><span class="sxs-lookup"><span data-stu-id="291ab-165">Web browser specify many formats, including wildcards.</span></span> <span data-ttu-id="291ab-166">Per impostazione predefinita, quando il Framework rileva che la richiesta viene ricevuta da un browser:</span><span class="sxs-lookup"><span data-stu-id="291ab-166">By default, when the framework detects that the request is coming from a browser:</span></span>

* <span data-ttu-id="291ab-167">L' `Accept` intestazione viene ignorata.</span><span class="sxs-lookup"><span data-stu-id="291ab-167">The `Accept` header is ignored.</span></span>
* <span data-ttu-id="291ab-168">Il contenuto viene restituito in JSON, a meno che non sia configurato diversamente.</span><span class="sxs-lookup"><span data-stu-id="291ab-168">The content is returned in JSON, unless otherwise configured.</span></span>

<span data-ttu-id="291ab-169">Ciò offre un'esperienza più coerente nei browser quando si utilizzano le API.</span><span class="sxs-lookup"><span data-stu-id="291ab-169">This provides a more consistent experience across browsers when consuming APIs.</span></span>

<span data-ttu-id="291ab-170">Per configurare un'app in modo da rispettare le intestazioni Accept del browser, impostare <xref:Microsoft.AspNetCore.Mvc.MvcOptions.RespectBrowserAcceptHeader> su `true` :</span><span class="sxs-lookup"><span data-stu-id="291ab-170">To configure an app to honor browser accept headers, set <xref:Microsoft.AspNetCore.Mvc.MvcOptions.RespectBrowserAcceptHeader> to `true`:</span></span>

::: moniker range=">= aspnetcore-3.0"
[!code-csharp[](./formatting/3.0sample/StartupRespectBrowserAcceptHeader.cs?name=snippet)]
::: moniker-end
::: moniker range="< aspnetcore-3.0"
[!code-csharp[](./formatting/sample/StartupRespectBrowserAcceptHeader.cs?name=snippet)]
::: moniker-end

### <a name="configure-formatters"></a><span data-ttu-id="291ab-171">Configura formattatori</span><span class="sxs-lookup"><span data-stu-id="291ab-171">Configure formatters</span></span>

<span data-ttu-id="291ab-172">Le app che devono supportare formati aggiuntivi possono aggiungere i pacchetti NuGet appropriati e configurare il supporto.</span><span class="sxs-lookup"><span data-stu-id="291ab-172">Apps that need to support additional formats can add the appropriate NuGet packages and configure support.</span></span> <span data-ttu-id="291ab-173">Esistono formattatori separati per input e output.</span><span class="sxs-lookup"><span data-stu-id="291ab-173">There are separate formatters for input and output.</span></span> <span data-ttu-id="291ab-174">I formattatori di input vengono utilizzati dall' [associazione di modelli](xref:mvc/models/model-binding).</span><span class="sxs-lookup"><span data-stu-id="291ab-174">Input formatters are used by [Model Binding](xref:mvc/models/model-binding).</span></span> <span data-ttu-id="291ab-175">I formattatori di output vengono utilizzati per formattare le risposte.</span><span class="sxs-lookup"><span data-stu-id="291ab-175">Output formatters are used to format responses.</span></span> <span data-ttu-id="291ab-176">Per informazioni sulla creazione di un formattatore personalizzato, vedere [formattatori personalizzati](xref:web-api/advanced/custom-formatters).</span><span class="sxs-lookup"><span data-stu-id="291ab-176">For information on creating a custom formatter, see [Custom Formatters](xref:web-api/advanced/custom-formatters).</span></span>

::: moniker range=">= aspnetcore-3.0"

### <a name="add-xml-format-support"></a><span data-ttu-id="291ab-177">Aggiungere il supporto per il formato XML</span><span class="sxs-lookup"><span data-stu-id="291ab-177">Add XML format support</span></span>

<span data-ttu-id="291ab-178">I formattatori XML implementati usando <xref:System.Xml.Serialization.XmlSerializer> vengono configurati chiamando <xref:Microsoft.Extensions.DependencyInjection.MvcXmlMvcBuilderExtensions.AddXmlSerializerFormatters*> :</span><span class="sxs-lookup"><span data-stu-id="291ab-178">XML formatters implemented using <xref:System.Xml.Serialization.XmlSerializer> are configured by calling <xref:Microsoft.Extensions.DependencyInjection.MvcXmlMvcBuilderExtensions.AddXmlSerializerFormatters*>:</span></span>

[!code-csharp[](./formatting/3.0sample/Startup.cs?name=snippet)]

<span data-ttu-id="291ab-179">Il codice precedente serializza i risultati usando `XmlSerializer` .</span><span class="sxs-lookup"><span data-stu-id="291ab-179">The preceding code serializes results using `XmlSerializer`.</span></span>

<span data-ttu-id="291ab-180">Quando si usa il codice precedente, i metodi del controller restituiscono il formato appropriato in base all'intestazione della richiesta `Accept` .</span><span class="sxs-lookup"><span data-stu-id="291ab-180">When using the preceding code, controller methods return the appropriate format based on the request's `Accept` header.</span></span>

### <a name="configure-systemtextjson-based-formatters"></a><span data-ttu-id="291ab-181">Configurare System.Text.Jssui formattatori basati</span><span class="sxs-lookup"><span data-stu-id="291ab-181">Configure System.Text.Json based formatters</span></span>

<span data-ttu-id="291ab-182">È possibile configurare le funzionalità per i `System.Text.Json` formattatori basati usando <xref:Microsoft.AspNetCore.Mvc.JsonOptions.JsonSerializerOptions?displayProperty=fullName> .</span><span class="sxs-lookup"><span data-stu-id="291ab-182">Features for the `System.Text.Json` based formatters can be configured using <xref:Microsoft.AspNetCore.Mvc.JsonOptions.JsonSerializerOptions?displayProperty=fullName>.</span></span> <span data-ttu-id="291ab-183">La formattazione predefinita è camelCase.</span><span class="sxs-lookup"><span data-stu-id="291ab-183">The default formatting is camelCase.</span></span> <span data-ttu-id="291ab-184">Il codice evidenziato seguente imposta la formattazione PascalCase:</span><span class="sxs-lookup"><span data-stu-id="291ab-184">The following highlighted code sets PascalCase formatting:</span></span>

[!code-csharp[](./formatting/5.0samples/WebAPI5PascalCase/Startup.cs?name=snippet&highlight=4-5)]

<span data-ttu-id="291ab-185">Il metodo di azione seguente chiama [ControllerBase. problem](xref:Microsoft.AspNetCore.Mvc.ControllerBase.Problem%2A) per creare una <xref:Microsoft.AspNetCore.Mvc.ProblemDetails> risposta:</span><span class="sxs-lookup"><span data-stu-id="291ab-185">The following action method calls [ControllerBase.Problem](xref:Microsoft.AspNetCore.Mvc.ControllerBase.Problem%2A) to create a <xref:Microsoft.AspNetCore.Mvc.ProblemDetails> response:</span></span>

[!code-csharp[](formatting/5.0samples/WebAPI5PascalCase/Controllers/WeatherForecastController.cs?name=snippet&highlight=4)]

<span data-ttu-id="291ab-186">Con il codice precedente:</span><span class="sxs-lookup"><span data-stu-id="291ab-186">With the preceding code:</span></span>

  * <span data-ttu-id="291ab-187">`https://localhost:5001/WeatherForecast/temperature` Restituisce PascalCase.</span><span class="sxs-lookup"><span data-stu-id="291ab-187">`https://localhost:5001/WeatherForecast/temperature` returns PascalCase.</span></span>
  * <span data-ttu-id="291ab-188">`https://localhost:5001/WeatherForecast/error` Restituisce camelCase.</span><span class="sxs-lookup"><span data-stu-id="291ab-188">`https://localhost:5001/WeatherForecast/error` returns camelCase.</span></span> <span data-ttu-id="291ab-189">La risposta di errore è sempre camelCase, anche quando l'app imposta il formato su PascalCase.</span><span class="sxs-lookup"><span data-stu-id="291ab-189">The error response is always camelCase, even when the app sets the format to PascalCase.</span></span> <span data-ttu-id="291ab-190">`ProblemDetails` segue la [RFC 7807](https://tools.ietf.org/html/rfc7807#appendix-A), che specifica il minuscolo</span><span class="sxs-lookup"><span data-stu-id="291ab-190">`ProblemDetails` follows [RFC 7807](https://tools.ietf.org/html/rfc7807#appendix-A), which specifies lower case</span></span>

<span data-ttu-id="291ab-191">Il codice seguente imposta PascalCase e aggiunge un convertitore personalizzato:</span><span class="sxs-lookup"><span data-stu-id="291ab-191">The following code sets PascalCase and adds a custom converter:</span></span>

```csharp
services.AddControllers().AddJsonOptions(options =>
{
    // Use the default property (Pascal) casing.
    options.JsonSerializerOptions.PropertyNamingPolicy = null;

    // Configure a custom converter.
    options.JsonSerializerOptions.Converters.Add(new MyCustomJsonConverter());
});
```

<span data-ttu-id="291ab-192">È possibile configurare le opzioni di serializzazione dell'output, in base alle singole azioni, usando `JsonResult` .</span><span class="sxs-lookup"><span data-stu-id="291ab-192">Output serialization options, on a per-action basis, can be configured using `JsonResult`.</span></span> <span data-ttu-id="291ab-193">Ad esempio:</span><span class="sxs-lookup"><span data-stu-id="291ab-193">For example:</span></span>

```csharp
public IActionResult Get()
{
    return Json(model, new JsonSerializerOptions
    {
        WriteIndented = true,
    });
}
```

### <a name="add-newtonsoftjson-based-json-format-support"></a><span data-ttu-id="291ab-194">Aggiungere il supporto del formato JSON basato su Newtonsoft.Json</span><span class="sxs-lookup"><span data-stu-id="291ab-194">Add Newtonsoft.Json-based JSON format support</span></span>

<span data-ttu-id="291ab-195">Prima di ASP.NET Core 3,0, i formattatori JSON usati per impostazione predefinita venivano implementati utilizzando il `Newtonsoft.Json` pacchetto.</span><span class="sxs-lookup"><span data-stu-id="291ab-195">Prior to ASP.NET Core 3.0, the default used JSON formatters implemented using the `Newtonsoft.Json` package.</span></span> <span data-ttu-id="291ab-196">In ASP.NET Core 3.0 o versione successiva, i formattatori JSON predefiniti sono basati su `System.Text.Json`.</span><span class="sxs-lookup"><span data-stu-id="291ab-196">In ASP.NET Core 3.0 or later, the default JSON formatters are based on `System.Text.Json`.</span></span> <span data-ttu-id="291ab-197">Il supporto per `Newtonsoft.Json` formattatori e funzionalità basati è disponibile installando il [`Microsoft.AspNetCore.Mvc.NewtonsoftJson`](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.NewtonsoftJson/) pacchetto NuGet e configurarlo in `Startup.ConfigureServices` .</span><span class="sxs-lookup"><span data-stu-id="291ab-197">Support for `Newtonsoft.Json` based formatters and features is available by installing the [`Microsoft.AspNetCore.Mvc.NewtonsoftJson`](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.NewtonsoftJson/) NuGet package and configuring it in `Startup.ConfigureServices`.</span></span>

[!code-csharp[](./formatting/3.0sample/StartupNewtonsoftJson.cs?name=snippet)]

<span data-ttu-id="291ab-198">Nel codice precedente, la chiamata a `AddNewtonsoftJson` Configura le seguenti funzionalità dell'API Web, MVC e Razor pages da usare `Newtonsoft.Json` :</span><span class="sxs-lookup"><span data-stu-id="291ab-198">In the preceding code, the call to `AddNewtonsoftJson` configures the following Web API, MVC, and Razor Pages features to use `Newtonsoft.Json`:</span></span>

* <span data-ttu-id="291ab-199">Formattatori di input e di output che leggono e scrivono JSON</span><span class="sxs-lookup"><span data-stu-id="291ab-199">Input and output formatters that read and write JSON</span></span>
* <xref:Microsoft.AspNetCore.Mvc.JsonResult>
* [<span data-ttu-id="291ab-200">JSON Patch</span><span class="sxs-lookup"><span data-stu-id="291ab-200">JSON Patch</span></span>](xref:web-api/jsonpatch)
* <xref:Microsoft.AspNetCore.Mvc.Rendering.IJsonHelper>
* [<span data-ttu-id="291ab-201">TempData</span><span class="sxs-lookup"><span data-stu-id="291ab-201">TempData</span></span>](xref:fundamentals/app-state#tempdata)

<span data-ttu-id="291ab-202">Alcune funzionalità potrebbero non funzionare correttamente con `System.Text.Json` formattatori basati su e richiedono un riferimento ai `Newtonsoft.Json` formattatori basati su.</span><span class="sxs-lookup"><span data-stu-id="291ab-202">Some features may not work well with `System.Text.Json`-based formatters and require a reference to the `Newtonsoft.Json`-based formatters.</span></span> <span data-ttu-id="291ab-203">Continuare a usare i `Newtonsoft.Json` formattatori basati su se l'app:</span><span class="sxs-lookup"><span data-stu-id="291ab-203">Continue using the `Newtonsoft.Json`-based formatters if the app:</span></span>

* <span data-ttu-id="291ab-204">Utilizza `Newtonsoft.Json` gli attributi.</span><span class="sxs-lookup"><span data-stu-id="291ab-204">Uses `Newtonsoft.Json` attributes.</span></span> <span data-ttu-id="291ab-205">Ad esempio, `[JsonProperty]` o `[JsonIgnore]`.</span><span class="sxs-lookup"><span data-stu-id="291ab-205">For example, `[JsonProperty]` or `[JsonIgnore]`.</span></span>
* <span data-ttu-id="291ab-206">Personalizza le impostazioni di serializzazione.</span><span class="sxs-lookup"><span data-stu-id="291ab-206">Customizes the serialization settings.</span></span>
* <span data-ttu-id="291ab-207">Si basa sulle funzionalità `Newtonsoft.Json` fornite da.</span><span class="sxs-lookup"><span data-stu-id="291ab-207">Relies on features that `Newtonsoft.Json` provides.</span></span>
* <span data-ttu-id="291ab-208">Configura `Microsoft.AspNetCore.Mvc.JsonResult.SerializerSettings`.</span><span class="sxs-lookup"><span data-stu-id="291ab-208">Configures `Microsoft.AspNetCore.Mvc.JsonResult.SerializerSettings`.</span></span> <span data-ttu-id="291ab-209">Prima di ASP.NET Core 3.0 `JsonResult.SerializerSettings` accetta un'istanza di `JsonSerializerSettings` specifica di `Newtonsoft.Json`.</span><span class="sxs-lookup"><span data-stu-id="291ab-209">Prior to ASP.NET Core 3.0, `JsonResult.SerializerSettings` accepts an instance of `JsonSerializerSettings` that is specific to `Newtonsoft.Json`.</span></span>
* <span data-ttu-id="291ab-210">Genera documentazione [OpenAPI](<xref:tutorials/web-api-help-pages-using-swagger>).</span><span class="sxs-lookup"><span data-stu-id="291ab-210">Generates [OpenAPI](<xref:tutorials/web-api-help-pages-using-swagger>) documentation.</span></span>

<span data-ttu-id="291ab-211">Le funzionalità per i `Newtonsoft.Json` formattatori basati su possono essere configurate utilizzando `Microsoft.AspNetCore.Mvc.MvcNewtonsoftJsonOptions.SerializerSettings` :</span><span class="sxs-lookup"><span data-stu-id="291ab-211">Features for the `Newtonsoft.Json`-based formatters can be configured using `Microsoft.AspNetCore.Mvc.MvcNewtonsoftJsonOptions.SerializerSettings`:</span></span>

```csharp
services.AddControllers().AddNewtonsoftJson(options =>
{
    // Use the default property (Pascal) casing
    options.SerializerSettings.ContractResolver = new DefaultContractResolver();

    // Configure a custom converter
    options.SerializerSettings.Converters.Add(new MyCustomJsonConverter());
});
```

<span data-ttu-id="291ab-212">È possibile configurare le opzioni di serializzazione dell'output, in base alle singole azioni, usando `JsonResult` .</span><span class="sxs-lookup"><span data-stu-id="291ab-212">Output serialization options, on a per-action basis, can be configured using `JsonResult`.</span></span> <span data-ttu-id="291ab-213">Ad esempio:</span><span class="sxs-lookup"><span data-stu-id="291ab-213">For example:</span></span>

```csharp
public IActionResult Get()
{
    return Json(model, new JsonSerializerSettings
    {
        Formatting = Formatting.Indented,
    });
}
```

::: moniker-end

::: moniker range="<= aspnetcore-2.2"

### <a name="add-xml-format-support"></a><span data-ttu-id="291ab-214">Aggiungere il supporto per il formato XML</span><span class="sxs-lookup"><span data-stu-id="291ab-214">Add XML format support</span></span>

<span data-ttu-id="291ab-215">La formattazione XML richiede il pacchetto NuGet [Microsoft.AspNetCore.Mvc.Formatters.Xml](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.Formatters.Xml/) .</span><span class="sxs-lookup"><span data-stu-id="291ab-215">XML formatting requires the [Microsoft.AspNetCore.Mvc.Formatters.Xml](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.Formatters.Xml/) NuGet package.</span></span>

<span data-ttu-id="291ab-216">I formattatori XML implementati usando <xref:System.Xml.Serialization.XmlSerializer> vengono configurati chiamando <xref:Microsoft.Extensions.DependencyInjection.MvcXmlMvcBuilderExtensions.AddXmlSerializerFormatters*> :</span><span class="sxs-lookup"><span data-stu-id="291ab-216">XML formatters implemented using <xref:System.Xml.Serialization.XmlSerializer> are configured by calling <xref:Microsoft.Extensions.DependencyInjection.MvcXmlMvcBuilderExtensions.AddXmlSerializerFormatters*>:</span></span>

[!code-csharp[](./formatting/sample/Startup.cs?name=snippet)]

<span data-ttu-id="291ab-217">Il codice precedente serializza i risultati usando `XmlSerializer` .</span><span class="sxs-lookup"><span data-stu-id="291ab-217">The preceding code serializes results using `XmlSerializer`.</span></span>

<span data-ttu-id="291ab-218">Quando si usa il codice precedente, i metodi del controller devono restituire il formato appropriato in base all'intestazione della richiesta `Accept` .</span><span class="sxs-lookup"><span data-stu-id="291ab-218">When using the preceding code, controller methods should return the appropriate format based on the request's `Accept` header.</span></span>

::: moniker-end

### <a name="specify-a-format"></a><span data-ttu-id="291ab-219">Specificare un formato</span><span class="sxs-lookup"><span data-stu-id="291ab-219">Specify a format</span></span>

<span data-ttu-id="291ab-220">Per limitare i formati di risposta, applicare il [`[Produces]`](xref:Microsoft.AspNetCore.Mvc.ProducesAttribute) filtro.</span><span class="sxs-lookup"><span data-stu-id="291ab-220">To restrict the response formats, apply the [`[Produces]`](xref:Microsoft.AspNetCore.Mvc.ProducesAttribute) filter.</span></span> <span data-ttu-id="291ab-221">Come la maggior parte dei [filtri](xref:mvc/controllers/filters), `[Produces]` può essere applicato all'azione, al controller o all'ambito globale:</span><span class="sxs-lookup"><span data-stu-id="291ab-221">Like most [Filters](xref:mvc/controllers/filters), `[Produces]` can be applied at the action, controller, or global scope:</span></span>

[!code-csharp[](./formatting/3.0sample/Controllers/WeatherForecastController.cs?name=snippet)]

<span data-ttu-id="291ab-222">Il [`[Produces]`](xref:Microsoft.AspNetCore.Mvc.ProducesAttribute) filtro precedente:</span><span class="sxs-lookup"><span data-stu-id="291ab-222">The preceding [`[Produces]`](xref:Microsoft.AspNetCore.Mvc.ProducesAttribute) filter:</span></span>

* <span data-ttu-id="291ab-223">Impone a tutte le azioni all'interno del controller di restituire risposte in formato JSON.</span><span class="sxs-lookup"><span data-stu-id="291ab-223">Forces all actions within the controller to return JSON-formatted responses.</span></span>
* <span data-ttu-id="291ab-224">Se sono configurati altri formattatori e il client specifica un formato diverso, viene restituito JSON.</span><span class="sxs-lookup"><span data-stu-id="291ab-224">If other formatters are configured and the client specifies a different format, JSON is returned.</span></span>

<span data-ttu-id="291ab-225">Per ulteriori informazioni, vedere [filtri](xref:mvc/controllers/filters).</span><span class="sxs-lookup"><span data-stu-id="291ab-225">For more information, see [Filters](xref:mvc/controllers/filters).</span></span>

### <a name="special-case-formatters"></a><span data-ttu-id="291ab-226">Formattatori case speciali</span><span class="sxs-lookup"><span data-stu-id="291ab-226">Special case formatters</span></span>

<span data-ttu-id="291ab-227">Alcuni casi speciali vengono implementati tramite formattatori predefiniti.</span><span class="sxs-lookup"><span data-stu-id="291ab-227">Some special cases are implemented using built-in formatters.</span></span> <span data-ttu-id="291ab-228">Per impostazione predefinita, `string` i tipi restituiti vengono formattati come *testo/normale* (*testo/HTML* se richiesto tramite l' `Accept` intestazione).</span><span class="sxs-lookup"><span data-stu-id="291ab-228">By default, `string` return types are formatted as *text/plain* (*text/html* if requested via the `Accept` header).</span></span> <span data-ttu-id="291ab-229">Questo comportamento può essere eliminato rimuovendo <xref:Microsoft.AspNetCore.Mvc.Formatters.StringOutputFormatter> .</span><span class="sxs-lookup"><span data-stu-id="291ab-229">This behavior can be deleted by removing the <xref:Microsoft.AspNetCore.Mvc.Formatters.StringOutputFormatter>.</span></span> <span data-ttu-id="291ab-230">I formattatori vengono rimossi nel `ConfigureServices` metodo.</span><span class="sxs-lookup"><span data-stu-id="291ab-230">Formatters are removed in the `ConfigureServices` method.</span></span> <span data-ttu-id="291ab-231">Le azioni con un tipo restituito dall'oggetto modello restituiscono quando viene restituito `204 No Content` `null` .</span><span class="sxs-lookup"><span data-stu-id="291ab-231">Actions that have a model object return type return `204 No Content` when returning `null`.</span></span> <span data-ttu-id="291ab-232">Questo comportamento può essere eliminato rimuovendo <xref:Microsoft.AspNetCore.Mvc.Formatters.HttpNoContentOutputFormatter> .</span><span class="sxs-lookup"><span data-stu-id="291ab-232">This behavior can be deleted by removing the <xref:Microsoft.AspNetCore.Mvc.Formatters.HttpNoContentOutputFormatter>.</span></span> <span data-ttu-id="291ab-233">Il codice seguente rimuove `StringOutputFormatter` e `HttpNoContentOutputFormatter`.</span><span class="sxs-lookup"><span data-stu-id="291ab-233">The following code removes the `StringOutputFormatter` and `HttpNoContentOutputFormatter`.</span></span>

::: moniker range=">= aspnetcore-3.0"
[!code-csharp[](./formatting/3.0sample/StartupStringOutputFormatter.cs?name=snippet)]
::: moniker-end
::: moniker range="< aspnetcore-3.0"
[!code-csharp[](./formatting/sample/StartupStringOutputFormatter.cs?name=snippet)]
::: moniker-end

<span data-ttu-id="291ab-234">Senza `StringOutputFormatter` , il formattatore JSON incorporato formatta i `string` tipi restituiti.</span><span class="sxs-lookup"><span data-stu-id="291ab-234">Without the `StringOutputFormatter`, the built-in JSON formatter formats `string` return types.</span></span> <span data-ttu-id="291ab-235">Se il formattatore JSON incorporato viene rimosso ed è disponibile un formattatore XML, il formattatore XML formatta i `string` tipi restituiti.</span><span class="sxs-lookup"><span data-stu-id="291ab-235">If the built-in JSON formatter is removed and an XML formatter is available, the XML formatter formats `string` return types.</span></span> <span data-ttu-id="291ab-236">In caso contrario, i `string` tipi restituiti restituiscono `406 Not Acceptable` .</span><span class="sxs-lookup"><span data-stu-id="291ab-236">Otherwise, `string` return types return `406 Not Acceptable`.</span></span>

<span data-ttu-id="291ab-237">Senza `HttpNoContentOutputFormatter`, gli oggetti Null vengono formattati tramite il formattatore configurato.</span><span class="sxs-lookup"><span data-stu-id="291ab-237">Without the `HttpNoContentOutputFormatter`, null objects are formatted using the configured formatter.</span></span> <span data-ttu-id="291ab-238">Ad esempio:</span><span class="sxs-lookup"><span data-stu-id="291ab-238">For example:</span></span>

* <span data-ttu-id="291ab-239">Il formattatore JSON restituisce una risposta con un corpo di `null` .</span><span class="sxs-lookup"><span data-stu-id="291ab-239">The JSON formatter returns a response with a body of `null`.</span></span>
* <span data-ttu-id="291ab-240">Il formattatore XML restituisce un elemento XML vuoto con l'attributo `xsi:nil="true"` impostato.</span><span class="sxs-lookup"><span data-stu-id="291ab-240">The XML formatter returns an empty XML element with the attribute `xsi:nil="true"` set.</span></span>

## <a name="response-format-url-mappings"></a><span data-ttu-id="291ab-241">Mapping URL formato risposta</span><span class="sxs-lookup"><span data-stu-id="291ab-241">Response format URL mappings</span></span>

<span data-ttu-id="291ab-242">I client possono richiedere un particolare formato come parte dell'URL, ad esempio:</span><span class="sxs-lookup"><span data-stu-id="291ab-242">Clients can request a particular format as part of the URL, for example:</span></span>

* <span data-ttu-id="291ab-243">Nella stringa di query o in una parte del percorso.</span><span class="sxs-lookup"><span data-stu-id="291ab-243">In the query string or part of the path.</span></span>
* <span data-ttu-id="291ab-244">Utilizzando un'estensione di file specifica del formato, ad esempio XML o JSON.</span><span class="sxs-lookup"><span data-stu-id="291ab-244">By using a format-specific file extension such as .xml or .json.</span></span>

<span data-ttu-id="291ab-245">Il mapping dal percorso della richiesta deve essere specificato nella route usata dall'API.</span><span class="sxs-lookup"><span data-stu-id="291ab-245">The mapping from request path should be specified in the route the API is using.</span></span> <span data-ttu-id="291ab-246">Ad esempio:</span><span class="sxs-lookup"><span data-stu-id="291ab-246">For example:</span></span>

[!code-csharp[](./formatting/sample/Controllers/ProductsController.cs?name=snippet)]

<span data-ttu-id="291ab-247">La route precedente consente di specificare il formato richiesto come estensione di file facoltativa.</span><span class="sxs-lookup"><span data-stu-id="291ab-247">The preceding route allows the requested format to be specified as an optional file extension.</span></span> <span data-ttu-id="291ab-248">L' [`[FormatFilter]`](xref:Microsoft.AspNetCore.Mvc.FormatFilterAttribute) attributo controlla l'esistenza del valore di formato in `RouteData` ed esegue il mapping del formato della risposta al formattatore appropriato al momento della creazione della risposta.</span><span class="sxs-lookup"><span data-stu-id="291ab-248">The [`[FormatFilter]`](xref:Microsoft.AspNetCore.Mvc.FormatFilterAttribute) attribute checks for the existence of the format value in the `RouteData` and maps the response format to the appropriate formatter when the response is created.</span></span>

|           <span data-ttu-id="291ab-249">Route</span><span class="sxs-lookup"><span data-stu-id="291ab-249">Route</span></span>        |             <span data-ttu-id="291ab-250">Formattatore</span><span class="sxs-lookup"><span data-stu-id="291ab-250">Formatter</span></span>              |
|------------------------|------------------------------------|
|   `/api/products/5`    |    <span data-ttu-id="291ab-251">Formattatore di output predefinito</span><span class="sxs-lookup"><span data-stu-id="291ab-251">The default output formatter</span></span>    |
| `/api/products/5.json` | <span data-ttu-id="291ab-252">Formattatore JSON (se configurato)</span><span class="sxs-lookup"><span data-stu-id="291ab-252">The JSON formatter (if configured)</span></span> |
| `/api/products/5.xml`  | <span data-ttu-id="291ab-253">Formattatore XML (se configurato)</span><span class="sxs-lookup"><span data-stu-id="291ab-253">The XML formatter (if configured)</span></span>  |
