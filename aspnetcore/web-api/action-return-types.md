---
title: Tipi restituiti dall'azione del controller nell'API Web ASP.NET Core
author: scottaddie
description: Informazioni sull'uso dei diversi tipi restituiti del metodo di azione del controller in un'API Web ASP.NET Core.
ms.author: scaddie
ms.custom: mvc
ms.date: 02/03/2020
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
uid: web-api/action-return-types
ms.openlocfilehash: b377025a843f82a57fd0aac4961ddcedc67ff3b9
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102589725"
---
# <a name="controller-action-return-types-in-aspnet-core-web-api"></a><span data-ttu-id="c6380-103">Tipi restituiti dall'azione del controller nell'API Web ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="c6380-103">Controller action return types in ASP.NET Core web API</span></span>

<span data-ttu-id="c6380-104">Di [Scott Addie](https://github.com/scottaddie)</span><span class="sxs-lookup"><span data-stu-id="c6380-104">By [Scott Addie](https://github.com/scottaddie)</span></span>

<span data-ttu-id="c6380-105">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/web-api/action-return-types/samples) ([procedura per il download](xref:index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="c6380-105">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/web-api/action-return-types/samples) ([how to download](xref:index#how-to-download-a-sample))</span></span>

<span data-ttu-id="c6380-106">ASP.NET Core offre le opzioni seguenti per i tipi restituiti dell'azione del controller API Web:</span><span class="sxs-lookup"><span data-stu-id="c6380-106">ASP.NET Core offers the following options for web API controller action return types:</span></span>

::: moniker range=">= aspnetcore-2.1"

* [<span data-ttu-id="c6380-107">Tipo specifico</span><span class="sxs-lookup"><span data-stu-id="c6380-107">Specific type</span></span>](#specific-type)
* [<span data-ttu-id="c6380-108">IActionResult</span><span class="sxs-lookup"><span data-stu-id="c6380-108">IActionResult</span></span>](#iactionresult-type)
* [<span data-ttu-id="c6380-109">ActionResult\<T></span><span class="sxs-lookup"><span data-stu-id="c6380-109">ActionResult\<T></span></span>](#actionresultt-type)

::: moniker-end

::: moniker range="<= aspnetcore-2.0"

* [<span data-ttu-id="c6380-110">Tipo specifico</span><span class="sxs-lookup"><span data-stu-id="c6380-110">Specific type</span></span>](#specific-type)
* [<span data-ttu-id="c6380-111">IActionResult</span><span class="sxs-lookup"><span data-stu-id="c6380-111">IActionResult</span></span>](#iactionresult-type)

::: moniker-end

<span data-ttu-id="c6380-112">Questo documento spiega quando è più appropriato utilizzare ogni tipo restituito.</span><span class="sxs-lookup"><span data-stu-id="c6380-112">This document explains when it's most appropriate to use each return type.</span></span>

## <a name="specific-type"></a><span data-ttu-id="c6380-113">Tipo specifico</span><span class="sxs-lookup"><span data-stu-id="c6380-113">Specific type</span></span>

<span data-ttu-id="c6380-114">L'azione più semplice restituisce un tipo di dati primitivo o complesso, ad esempio, `string` o un tipo di oggetto personalizzato.</span><span class="sxs-lookup"><span data-stu-id="c6380-114">The simplest action returns a primitive or complex data type (for example, `string` or a custom object type).</span></span> <span data-ttu-id="c6380-115">Si consideri l'azione seguente, che restituisce una raccolta di oggetti `Product` personalizzati:</span><span class="sxs-lookup"><span data-stu-id="c6380-115">Consider the following action, which returns a collection of custom `Product` objects:</span></span>

[!code-csharp[](../web-api/action-return-types/samples/2x/WebApiSample.Api.21/Controllers/ProductsController.cs?name=snippet_Get)]

<span data-ttu-id="c6380-116">In assenza di condizioni note da cui proteggersi durante l'esecuzione dell'azione, la restituzione di un tipo specifico potrebbe essere sufficiente.</span><span class="sxs-lookup"><span data-stu-id="c6380-116">Without known conditions to safeguard against during action execution, returning a specific type could suffice.</span></span> <span data-ttu-id="c6380-117">L'azione precedente non accetta parametri, quindi non è necessario eseguire la convalida dei vincoli dei parametri.</span><span class="sxs-lookup"><span data-stu-id="c6380-117">The preceding action accepts no parameters, so parameter constraints validation isn't needed.</span></span>

<span data-ttu-id="c6380-118">Quando sono possibili più tipi restituiti, è comune combinare un <xref:Microsoft.AspNetCore.Mvc.ActionResult> tipo restituito con il tipo restituito primitivo o complesso.</span><span class="sxs-lookup"><span data-stu-id="c6380-118">When multiple return types are possible, it's common to mix an <xref:Microsoft.AspNetCore.Mvc.ActionResult> return type with the primitive or complex return type.</span></span> <span data-ttu-id="c6380-119">[IActionResult](#iactionresult-type) o [ActionResult \<T> ](#actionresultt-type) sono necessari per gestire questo tipo di azione.</span><span class="sxs-lookup"><span data-stu-id="c6380-119">Either [IActionResult](#iactionresult-type) or [ActionResult\<T>](#actionresultt-type) are necessary to accommodate this type of action.</span></span> <span data-ttu-id="c6380-120">In questo documento vengono forniti diversi esempi di più tipi restituiti.</span><span class="sxs-lookup"><span data-stu-id="c6380-120">Several samples of multiple return types are provided in this document.</span></span>

### <a name="return-ienumerablet-or-iasyncenumerablet"></a><span data-ttu-id="c6380-121">Return IEnumerable \<T> o IAsyncEnumerable\<T></span><span class="sxs-lookup"><span data-stu-id="c6380-121">Return IEnumerable\<T> or IAsyncEnumerable\<T></span></span>

<span data-ttu-id="c6380-122">In ASP.NET Core 2,2 e versioni precedenti, <xref:System.Collections.Generic.IEnumerable%601> la restituzione da un'azione comporta l'iterazione di una raccolta sincrona da parte del serializzatore.</span><span class="sxs-lookup"><span data-stu-id="c6380-122">In ASP.NET Core 2.2 and earlier, returning <xref:System.Collections.Generic.IEnumerable%601> from an action results in synchronous collection iteration by the serializer.</span></span> <span data-ttu-id="c6380-123">Il risultato è il blocco delle chiamate e un potenziale per l'inedia del pool di thread.</span><span class="sxs-lookup"><span data-stu-id="c6380-123">The result is the blocking of calls and a potential for thread pool starvation.</span></span> <span data-ttu-id="c6380-124">Per illustrare, si supponga che Entity Framework (EF) Core venga usato per le esigenze di accesso ai dati dell'API Web.</span><span class="sxs-lookup"><span data-stu-id="c6380-124">To illustrate, imagine that Entity Framework (EF) Core is being used for the web API's data access needs.</span></span> <span data-ttu-id="c6380-125">Il tipo restituito dell'azione seguente viene enumerato in modo sincrono durante la serializzazione:</span><span class="sxs-lookup"><span data-stu-id="c6380-125">The following action's return type is synchronously enumerated during serialization:</span></span>

```csharp
public IEnumerable<Product> GetOnSaleProducts() =>
    _context.Products.Where(p => p.IsOnSale);
```

<span data-ttu-id="c6380-126">Per evitare l'enumerazione sincrona e le attese di blocco sul database in ASP.NET Core 2,2 e versioni precedenti, richiamare `ToListAsync` :</span><span class="sxs-lookup"><span data-stu-id="c6380-126">To avoid synchronous enumeration and blocking waits on the database in ASP.NET Core 2.2 and earlier, invoke `ToListAsync`:</span></span>

```csharp
public async Task<IEnumerable<Product>> GetOnSaleProducts() =>
    await _context.Products.Where(p => p.IsOnSale).ToListAsync();
```

<span data-ttu-id="c6380-127">In ASP.NET Core 3,0 e versioni successive, tornando `IAsyncEnumerable<T>` da un'azione:</span><span class="sxs-lookup"><span data-stu-id="c6380-127">In ASP.NET Core 3.0 and later, returning `IAsyncEnumerable<T>` from an action:</span></span>

* <span data-ttu-id="c6380-128">Non produce più un'iterazione sincrona.</span><span class="sxs-lookup"><span data-stu-id="c6380-128">No longer results in synchronous iteration.</span></span>
* <span data-ttu-id="c6380-129">Diventa efficiente come la restituzione <xref:System.Collections.Generic.IEnumerable%601> .</span><span class="sxs-lookup"><span data-stu-id="c6380-129">Becomes as efficient as returning <xref:System.Collections.Generic.IEnumerable%601>.</span></span>

<span data-ttu-id="c6380-130">ASP.NET Core 3,0 e versioni successive memorizza nel buffer il risultato dell'azione seguente prima di fornirlo al serializzatore:</span><span class="sxs-lookup"><span data-stu-id="c6380-130">ASP.NET Core 3.0 and later buffers the result of the following action before providing it to the serializer:</span></span>

```csharp
public IEnumerable<Product> GetOnSaleProducts() =>
    _context.Products.Where(p => p.IsOnSale);
```

<span data-ttu-id="c6380-131">Provare a dichiarare il tipo restituito della firma dell'azione come `IAsyncEnumerable<T>` per garantire l'iterazione asincrona.</span><span class="sxs-lookup"><span data-stu-id="c6380-131">Consider declaring the action signature's return type as `IAsyncEnumerable<T>` to guarantee the asynchronous iteration.</span></span> <span data-ttu-id="c6380-132">In definitiva, la modalità di iterazione è basata sul tipo concreto sottostante restituito.</span><span class="sxs-lookup"><span data-stu-id="c6380-132">Ultimately, the iteration mode is based on the underlying concrete type being returned.</span></span> <span data-ttu-id="c6380-133">MVC esegue automaticamente il buffering di qualsiasi tipo concreto che implementa `IAsyncEnumerable<T>` .</span><span class="sxs-lookup"><span data-stu-id="c6380-133">MVC automatically buffers any concrete type that implements `IAsyncEnumerable<T>`.</span></span>

<span data-ttu-id="c6380-134">Si consideri l'azione seguente, che restituisce i record di prodotto con prezzo di vendita come `IEnumerable<Product>` :</span><span class="sxs-lookup"><span data-stu-id="c6380-134">Consider the following action, which returns sale-priced product records as `IEnumerable<Product>`:</span></span>

[!code-csharp[](../web-api/action-return-types/samples/3x/WebApiSample.Api.31/Controllers/ProductsController.cs?name=snippet_GetOnSaleProducts)]

<span data-ttu-id="c6380-135">L' `IAsyncEnumerable<Product>` equivalente dell'azione precedente è:</span><span class="sxs-lookup"><span data-stu-id="c6380-135">The `IAsyncEnumerable<Product>` equivalent of the preceding action is:</span></span>

[!code-csharp[](../web-api/action-return-types/samples/3x/WebApiSample.Api.31/Controllers/ProductsController.cs?name=snippet_GetOnSaleProductsAsync)]

<span data-ttu-id="c6380-136">Entrambe le azioni precedenti sono non bloccanti a partire da ASP.NET Core 3,0.</span><span class="sxs-lookup"><span data-stu-id="c6380-136">Both of the preceding actions are non-blocking as of ASP.NET Core 3.0.</span></span>

## <a name="iactionresult-type"></a><span data-ttu-id="c6380-137">Tipo IActionResult</span><span class="sxs-lookup"><span data-stu-id="c6380-137">IActionResult type</span></span>

<span data-ttu-id="c6380-138">Il <xref:Microsoft.AspNetCore.Mvc.IActionResult> tipo restituito è appropriato quando più `ActionResult` tipi restituiti sono possibili in un'azione.</span><span class="sxs-lookup"><span data-stu-id="c6380-138">The <xref:Microsoft.AspNetCore.Mvc.IActionResult> return type is appropriate when multiple `ActionResult` return types are possible in an action.</span></span> <span data-ttu-id="c6380-139">I tipi `ActionResult` rappresentano vari codici di stato HTTP.</span><span class="sxs-lookup"><span data-stu-id="c6380-139">The `ActionResult` types represent various HTTP status codes.</span></span> <span data-ttu-id="c6380-140">Qualsiasi classe non astratta che deriva da viene `ActionResult` qualificata come tipo restituito valido.</span><span class="sxs-lookup"><span data-stu-id="c6380-140">Any non-abstract class deriving from `ActionResult` qualifies as a valid return type.</span></span> <span data-ttu-id="c6380-141">Alcuni tipi restituiti comuni in questa categoria sono <xref:Microsoft.AspNetCore.Mvc.BadRequestResult> (400), <xref:Microsoft.AspNetCore.Mvc.NotFoundResult> (404) e <xref:Microsoft.AspNetCore.Mvc.OkObjectResult> (200).</span><span class="sxs-lookup"><span data-stu-id="c6380-141">Some common return types in this category are <xref:Microsoft.AspNetCore.Mvc.BadRequestResult> (400), <xref:Microsoft.AspNetCore.Mvc.NotFoundResult> (404), and <xref:Microsoft.AspNetCore.Mvc.OkObjectResult> (200).</span></span> <span data-ttu-id="c6380-142">In alternativa, è possibile utilizzare i metodi pratici della <xref:Microsoft.AspNetCore.Mvc.ControllerBase> classe per restituire i `ActionResult` tipi da un'azione.</span><span class="sxs-lookup"><span data-stu-id="c6380-142">Alternatively, convenience methods in the <xref:Microsoft.AspNetCore.Mvc.ControllerBase> class can be used to return `ActionResult` types from an action.</span></span> <span data-ttu-id="c6380-143">Ad esempio, `return BadRequest();` è una forma abbreviata di `return new BadRequestResult();` .</span><span class="sxs-lookup"><span data-stu-id="c6380-143">For example, `return BadRequest();` is a shorthand form of `return new BadRequestResult();`.</span></span>

<span data-ttu-id="c6380-144">Poiché in questo tipo di azione sono presenti più tipi e percorsi restituiti, è necessario utilizzare l'attributo in modo liberato [`[ProducesResponseType]`](xref:Microsoft.AspNetCore.Mvc.ProducesResponseTypeAttribute) .</span><span class="sxs-lookup"><span data-stu-id="c6380-144">Because there are multiple return types and paths in this type of action, liberal use of the [`[ProducesResponseType]`](xref:Microsoft.AspNetCore.Mvc.ProducesResponseTypeAttribute) attribute is necessary.</span></span> <span data-ttu-id="c6380-145">Questo attributo produce dettagli di risposta più descrittivi per le pagine della Guida dell'API Web generate da strumenti come [spavalderia](xref:tutorials/web-api-help-pages-using-swagger).</span><span class="sxs-lookup"><span data-stu-id="c6380-145">This attribute produces more descriptive response details for web API help pages generated by tools like [Swagger](xref:tutorials/web-api-help-pages-using-swagger).</span></span> <span data-ttu-id="c6380-146">`[ProducesResponseType]` indica i tipi noti e i codici di stato HTTP che l'azione deve restituire.</span><span class="sxs-lookup"><span data-stu-id="c6380-146">`[ProducesResponseType]` indicates the known types and HTTP status codes to be returned by the action.</span></span>

### <a name="synchronous-action"></a><span data-ttu-id="c6380-147">Azione sincrona</span><span class="sxs-lookup"><span data-stu-id="c6380-147">Synchronous action</span></span>

<span data-ttu-id="c6380-148">Si consideri la seguente azione sincrona che prevede due possibili tipi restituiti:</span><span class="sxs-lookup"><span data-stu-id="c6380-148">Consider the following synchronous action in which there are two possible return types:</span></span>

::: moniker range=">= aspnetcore-2.1"

[!code-csharp[](../web-api/action-return-types/samples/2x/WebApiSample.Api.21/Controllers/ProductsController.cs?name=snippet_GetByIdIActionResult&highlight=8,11)]

::: moniker-end

::: moniker range="<= aspnetcore-2.0"

[!code-csharp[](../web-api/action-return-types/samples/2x/WebApiSample.Api.Pre21/Controllers/ProductsController.cs?name=snippet_GetById&highlight=8,11)]

::: moniker-end

<span data-ttu-id="c6380-149">Nell'azione precedente:</span><span class="sxs-lookup"><span data-stu-id="c6380-149">In the preceding action:</span></span>

* <span data-ttu-id="c6380-150">Viene restituito un codice di stato 404 quando il prodotto rappresentato da `id` non esiste nell'archivio dati sottostante.</span><span class="sxs-lookup"><span data-stu-id="c6380-150">A 404 status code is returned when the product represented by `id` doesn't exist in the underlying data store.</span></span> <span data-ttu-id="c6380-151">Il <xref:Microsoft.AspNetCore.Mvc.ControllerBase.NotFound*> Metodo convenience viene richiamato come sintassi abbreviata per `return new NotFoundResult();` .</span><span class="sxs-lookup"><span data-stu-id="c6380-151">The <xref:Microsoft.AspNetCore.Mvc.ControllerBase.NotFound*> convenience method is invoked as shorthand for `return new NotFoundResult();`.</span></span>
* <span data-ttu-id="c6380-152">Quando il prodotto esiste, viene restituito un codice di stato 200 con l' `Product` oggetto.</span><span class="sxs-lookup"><span data-stu-id="c6380-152">A 200 status code is returned with the `Product` object when the product does exist.</span></span> <span data-ttu-id="c6380-153">Il <xref:Microsoft.AspNetCore.Mvc.ControllerBase.Ok*> Metodo convenience viene richiamato come sintassi abbreviata per `return new OkObjectResult(product);` .</span><span class="sxs-lookup"><span data-stu-id="c6380-153">The <xref:Microsoft.AspNetCore.Mvc.ControllerBase.Ok*> convenience method is invoked as shorthand for `return new OkObjectResult(product);`.</span></span>

### <a name="asynchronous-action"></a><span data-ttu-id="c6380-154">Azione asincrona</span><span class="sxs-lookup"><span data-stu-id="c6380-154">Asynchronous action</span></span>

<span data-ttu-id="c6380-155">Si consideri la seguente azione asincrona che prevede due possibili tipi restituiti:</span><span class="sxs-lookup"><span data-stu-id="c6380-155">Consider the following asynchronous action in which there are two possible return types:</span></span>

::: moniker range=">= aspnetcore-2.1"

[!code-csharp[](../web-api/action-return-types/samples/2x/WebApiSample.Api.21/Controllers/ProductsController.cs?name=snippet_CreateAsyncIActionResult&highlight=9,14)]

::: moniker-end

::: moniker range="<= aspnetcore-2.0"

[!code-csharp[](../web-api/action-return-types/samples/2x/WebApiSample.Api.Pre21/Controllers/ProductsController.cs?name=snippet_CreateAsync&highlight=9,14)]

::: moniker-end

<span data-ttu-id="c6380-156">Nell'azione precedente:</span><span class="sxs-lookup"><span data-stu-id="c6380-156">In the preceding action:</span></span>

* <span data-ttu-id="c6380-157">Quando la descrizione del prodotto contiene "XYZ widget", viene restituito un codice di stato 400.</span><span class="sxs-lookup"><span data-stu-id="c6380-157">A 400 status code is returned when the product description contains "XYZ Widget".</span></span> <span data-ttu-id="c6380-158">Il <xref:Microsoft.AspNetCore.Mvc.ControllerBase.BadRequest*> Metodo convenience viene richiamato come sintassi abbreviata per `return new BadRequestResult();` .</span><span class="sxs-lookup"><span data-stu-id="c6380-158">The <xref:Microsoft.AspNetCore.Mvc.ControllerBase.BadRequest*> convenience method is invoked as shorthand for `return new BadRequestResult();`.</span></span>
* <span data-ttu-id="c6380-159">Un codice di stato 201 viene generato dal <xref:Microsoft.AspNetCore.Mvc.ControllerBase.CreatedAtAction*> metodo di praticità quando viene creato un prodotto.</span><span class="sxs-lookup"><span data-stu-id="c6380-159">A 201 status code is generated by the <xref:Microsoft.AspNetCore.Mvc.ControllerBase.CreatedAtAction*> convenience method when a product is created.</span></span> <span data-ttu-id="c6380-160">Un'alternativa alla chiamata di `CreatedAtAction` è `return new CreatedAtActionResult(nameof(GetById), "Products", new { id = product.Id }, product);` .</span><span class="sxs-lookup"><span data-stu-id="c6380-160">An alternative to calling `CreatedAtAction` is `return new CreatedAtActionResult(nameof(GetById), "Products", new { id = product.Id }, product);`.</span></span> <span data-ttu-id="c6380-161">In questo percorso di codice, l' `Product` oggetto viene fornito nel corpo della risposta.</span><span class="sxs-lookup"><span data-stu-id="c6380-161">In this code path, the `Product` object is provided in the response body.</span></span> <span data-ttu-id="c6380-162">`Location`Viene fornita un'intestazione di risposta contenente l'URL del prodotto appena creato.</span><span class="sxs-lookup"><span data-stu-id="c6380-162">A `Location` response header containing the newly created product's URL is provided.</span></span>

<span data-ttu-id="c6380-163">Ad esempio, il modello seguente indica che le richieste devono includere le proprietà `Name` e `Description`.</span><span class="sxs-lookup"><span data-stu-id="c6380-163">For example, the following model indicates that requests must include the `Name` and `Description` properties.</span></span> <span data-ttu-id="c6380-164">La mancata fornitura `Name` e `Description` nella richiesta causa la mancata convalida del modello.</span><span class="sxs-lookup"><span data-stu-id="c6380-164">Failure to provide `Name` and `Description` in the request causes model validation to fail.</span></span>

[!code-csharp[](../web-api/action-return-types/samples/2x/WebApiSample.DataAccess/Models/Product.cs?name=snippet_ProductClass&highlight=5-6,8-9)]

::: moniker range=">= aspnetcore-2.1"

<span data-ttu-id="c6380-165">Se [`[ApiController]`](xref:Microsoft.AspNetCore.Mvc.ApiControllerAttribute) viene applicato l'attributo in ASP.NET Core 2,1 o versioni successive, gli errori di convalida del modello generano un codice di stato 400.</span><span class="sxs-lookup"><span data-stu-id="c6380-165">If the [`[ApiController]`](xref:Microsoft.AspNetCore.Mvc.ApiControllerAttribute) attribute in ASP.NET Core 2.1 or later is applied, model validation errors result in a 400 status code.</span></span> <span data-ttu-id="c6380-166">Per altre informazioni, vedere [Risposte HTTP 400 automatiche](xref:web-api/index#automatic-http-400-responses).</span><span class="sxs-lookup"><span data-stu-id="c6380-166">For more information, see [Automatic HTTP 400 responses](xref:web-api/index#automatic-http-400-responses).</span></span>

## <a name="actionresultt-type"></a><span data-ttu-id="c6380-167">\<T>Tipo ActionResult</span><span class="sxs-lookup"><span data-stu-id="c6380-167">ActionResult\<T> type</span></span>

<span data-ttu-id="c6380-168">ASP.NET Core 2,1 ha introdotto il tipo restituito [ActionResult \<T> ](xref:Microsoft.AspNetCore.Mvc.ActionResult`1) per le azioni del controller API Web.</span><span class="sxs-lookup"><span data-stu-id="c6380-168">ASP.NET Core 2.1 introduced the [ActionResult\<T>](xref:Microsoft.AspNetCore.Mvc.ActionResult`1) return type for web API controller actions.</span></span> <span data-ttu-id="c6380-169">Consente di restituire un tipo che deriva da <xref:Microsoft.AspNetCore.Mvc.ActionResult> o restituisce un [tipo specifico](#specific-type).</span><span class="sxs-lookup"><span data-stu-id="c6380-169">It enables you to return a type deriving from <xref:Microsoft.AspNetCore.Mvc.ActionResult> or return a [specific type](#specific-type).</span></span> <span data-ttu-id="c6380-170">`ActionResult<T>` offre i vantaggi seguenti rispetto al [tipo IActionResult](#iactionresult-type):</span><span class="sxs-lookup"><span data-stu-id="c6380-170">`ActionResult<T>` offers the following benefits over the [IActionResult type](#iactionresult-type):</span></span>

* <span data-ttu-id="c6380-171">La [`[ProducesResponseType]`](xref:Microsoft.AspNetCore.Mvc.ProducesResponseTypeAttribute) proprietà dell'attributo `Type` può essere esclusa.</span><span class="sxs-lookup"><span data-stu-id="c6380-171">The [`[ProducesResponseType]`](xref:Microsoft.AspNetCore.Mvc.ProducesResponseTypeAttribute) attribute's `Type` property can be excluded.</span></span> <span data-ttu-id="c6380-172">Ad esempio, `[ProducesResponseType(200, Type = typeof(Product))]` viene semplificato in `[ProducesResponseType(200)]`.</span><span class="sxs-lookup"><span data-stu-id="c6380-172">For example, `[ProducesResponseType(200, Type = typeof(Product))]` is simplified to `[ProducesResponseType(200)]`.</span></span> <span data-ttu-id="c6380-173">Il tipo restituito previsto dell'azione viene invece dedotto da `T` in `ActionResult<T>`.</span><span class="sxs-lookup"><span data-stu-id="c6380-173">The action's expected return type is instead inferred from the `T` in `ActionResult<T>`.</span></span>
* <span data-ttu-id="c6380-174">[Operatori di cast impliciti](/dotnet/csharp/language-reference/keywords/implicit) supportano la conversione di `T` e `ActionResult` in `ActionResult<T>`.</span><span class="sxs-lookup"><span data-stu-id="c6380-174">[Implicit cast operators](/dotnet/csharp/language-reference/keywords/implicit) support the conversion of both `T` and `ActionResult` to `ActionResult<T>`.</span></span> <span data-ttu-id="c6380-175">`T` converte in <xref:Microsoft.AspNetCore.Mvc.ObjectResult> , il che significa che `return new ObjectResult(T);` è semplificato in `return T;` .</span><span class="sxs-lookup"><span data-stu-id="c6380-175">`T` converts to <xref:Microsoft.AspNetCore.Mvc.ObjectResult>, which means `return new ObjectResult(T);` is simplified to `return T;`.</span></span>

<span data-ttu-id="c6380-176">C# non supporta operatori di cast impliciti sulle interfacce.</span><span class="sxs-lookup"><span data-stu-id="c6380-176">C# doesn't support implicit cast operators on interfaces.</span></span> <span data-ttu-id="c6380-177">Di conseguenza, per la conversione dell'interfaccia in un tipo concreto è necessario usare `ActionResult<T>`.</span><span class="sxs-lookup"><span data-stu-id="c6380-177">Consequently, conversion of the interface to a concrete type is necessary to use `ActionResult<T>`.</span></span> <span data-ttu-id="c6380-178">Ad esempio, usare `IEnumerable` nell'esempio seguente non funziona:</span><span class="sxs-lookup"><span data-stu-id="c6380-178">For example, use of `IEnumerable` in the following example doesn't work:</span></span>

```csharp
[HttpGet]
public ActionResult<IEnumerable<Product>> Get() =>
    _repository.GetProducts();
```

<span data-ttu-id="c6380-179">Una delle opzioni disponibili per correggere il codice precedente consiste nel restituire `_repository.GetProducts().ToList();`.</span><span class="sxs-lookup"><span data-stu-id="c6380-179">One option to fix the preceding code is to return `_repository.GetProducts().ToList();`.</span></span>

<span data-ttu-id="c6380-180">La maggior parte delle azioni presenta un tipo restituito specifico.</span><span class="sxs-lookup"><span data-stu-id="c6380-180">Most actions have a specific return type.</span></span> <span data-ttu-id="c6380-181">Durante l'esecuzione di un'azione possono verificarsi condizioni impreviste, nel qual caso il tipo specifico non viene restituito.</span><span class="sxs-lookup"><span data-stu-id="c6380-181">Unexpected conditions can occur during action execution, in which case the specific type isn't returned.</span></span> <span data-ttu-id="c6380-182">Ad esempio, il parametro di input di un'azione potrebbe non superare la convalida del modello.</span><span class="sxs-lookup"><span data-stu-id="c6380-182">For example, an action's input parameter may fail model validation.</span></span> <span data-ttu-id="c6380-183">In tal caso, è consueto che venga restituito il tipo `ActionResult` appropriato anziché il tipo specifico.</span><span class="sxs-lookup"><span data-stu-id="c6380-183">In such a case, it's common to return the appropriate `ActionResult` type instead of the specific type.</span></span>

### <a name="synchronous-action"></a><span data-ttu-id="c6380-184">Azione sincrona</span><span class="sxs-lookup"><span data-stu-id="c6380-184">Synchronous action</span></span>

<span data-ttu-id="c6380-185">Si consideri un'azione sincrona che prevede due possibili tipi restituiti:</span><span class="sxs-lookup"><span data-stu-id="c6380-185">Consider a synchronous action in which there are two possible return types:</span></span>

[!code-csharp[](../web-api/action-return-types/samples/2x/WebApiSample.Api.21/Controllers/ProductsController.cs?name=snippet_GetByIdActionResult&highlight=8,11)]

<span data-ttu-id="c6380-186">Nell'azione precedente:</span><span class="sxs-lookup"><span data-stu-id="c6380-186">In the preceding action:</span></span>

* <span data-ttu-id="c6380-187">Quando il prodotto non esiste nel database, viene restituito un codice di stato 404.</span><span class="sxs-lookup"><span data-stu-id="c6380-187">A 404 status code is returned when the product doesn't exist in the database.</span></span>
* <span data-ttu-id="c6380-188">Quando il prodotto esiste, viene restituito un codice di stato 200 con l' `Product` oggetto corrispondente.</span><span class="sxs-lookup"><span data-stu-id="c6380-188">A 200 status code is returned with the corresponding `Product` object when the product does exist.</span></span> <span data-ttu-id="c6380-189">Prima di ASP.NET Core 2,1, la `return product;` riga doveva essere `return Ok(product);` .</span><span class="sxs-lookup"><span data-stu-id="c6380-189">Before ASP.NET Core 2.1, the `return product;` line had to be `return Ok(product);`.</span></span>

### <a name="asynchronous-action"></a><span data-ttu-id="c6380-190">Azione asincrona</span><span class="sxs-lookup"><span data-stu-id="c6380-190">Asynchronous action</span></span>

<span data-ttu-id="c6380-191">Si consideri un'azione asincrona che prevede due possibili tipi restituiti:</span><span class="sxs-lookup"><span data-stu-id="c6380-191">Consider an asynchronous action in which there are two possible return types:</span></span>

[!code-csharp[](../web-api/action-return-types/samples/2x/WebApiSample.Api.21/Controllers/ProductsController.cs?name=snippet_CreateAsyncActionResult&highlight=9,14)]

<span data-ttu-id="c6380-192">Nell'azione precedente:</span><span class="sxs-lookup"><span data-stu-id="c6380-192">In the preceding action:</span></span>

* <span data-ttu-id="c6380-193"><xref:Microsoft.AspNetCore.Mvc.ControllerBase.BadRequest*>Il runtime di ASP.NET Core restituisce un codice di stato 400 () quando:</span><span class="sxs-lookup"><span data-stu-id="c6380-193">A 400 status code (<xref:Microsoft.AspNetCore.Mvc.ControllerBase.BadRequest*>) is returned by the ASP.NET Core runtime when:</span></span>
  * <span data-ttu-id="c6380-194">L' [`[ApiController]`](xref:Microsoft.AspNetCore.Mvc.ApiControllerAttribute) attributo è stato applicato e la convalida del modello ha esito negativo.</span><span class="sxs-lookup"><span data-stu-id="c6380-194">The [`[ApiController]`](xref:Microsoft.AspNetCore.Mvc.ApiControllerAttribute) attribute has been applied and model validation fails.</span></span>
  * <span data-ttu-id="c6380-195">La descrizione del prodotto contiene "XYZ Widget".</span><span class="sxs-lookup"><span data-stu-id="c6380-195">The product description contains "XYZ Widget".</span></span>
* <span data-ttu-id="c6380-196">Un codice di stato 201 viene generato dal <xref:Microsoft.AspNetCore.Mvc.ControllerBase.CreatedAtAction*> metodo durante la creazione di un prodotto.</span><span class="sxs-lookup"><span data-stu-id="c6380-196">A 201 status code is generated by the <xref:Microsoft.AspNetCore.Mvc.ControllerBase.CreatedAtAction*> method when a product is created.</span></span> <span data-ttu-id="c6380-197">In questo percorso di codice, l' `Product` oggetto viene fornito nel corpo della risposta.</span><span class="sxs-lookup"><span data-stu-id="c6380-197">In this code path, the `Product` object is provided in the response body.</span></span> <span data-ttu-id="c6380-198">`Location`Viene fornita un'intestazione di risposta contenente l'URL del prodotto appena creato.</span><span class="sxs-lookup"><span data-stu-id="c6380-198">A `Location` response header containing the newly created product's URL is provided.</span></span>

::: moniker-end

## <a name="additional-resources"></a><span data-ttu-id="c6380-199">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="c6380-199">Additional resources</span></span>

* <xref:mvc/controllers/actions>
* <xref:mvc/models/validation>
* <xref:tutorials/web-api-help-pages-using-swagger>
