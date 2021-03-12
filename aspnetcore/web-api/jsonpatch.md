---
title: JSON Patch nell'API Web ASP.NET Core
author: rick-anderson
description: Informazioni su come gestire le richieste JSON Patch in un'API Web ASP.NET Core.
ms.author: riande
ms.custom: mvc
ms.date: 04/02/2020
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
uid: web-api/jsonpatch
ms.openlocfilehash: 4ed44a0fca9e0834a78e433cdd48cbd153c58666
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102587853"
---
# <a name="jsonpatch-in-aspnet-core-web-api"></a><span data-ttu-id="34fda-103">JSON Patch nell'API Web ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="34fda-103">JsonPatch in ASP.NET Core web API</span></span>

<span data-ttu-id="34fda-104">Di [Tom Dykstra](https://github.com/tdykstra) e [Kirk Larkin](https://github.com/serpent5)</span><span class="sxs-lookup"><span data-stu-id="34fda-104">By [Tom Dykstra](https://github.com/tdykstra) and [Kirk Larkin](https://github.com/serpent5)</span></span>

::: moniker range=">= aspnetcore-3.0"

<span data-ttu-id="34fda-105">Questo articolo descrive come gestire le richieste JSON Patch in un'API Web ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="34fda-105">This article explains how to handle JSON Patch requests in an ASP.NET Core web API.</span></span>

## <a name="package-installation"></a><span data-ttu-id="34fda-106">Installazione del pacchetto</span><span class="sxs-lookup"><span data-stu-id="34fda-106">Package installation</span></span>

<span data-ttu-id="34fda-107">Per abilitare il supporto della patch JSON nell'app, seguire questa procedura:</span><span class="sxs-lookup"><span data-stu-id="34fda-107">To enable JSON Patch support in your app, complete the following steps:</span></span>

1. <span data-ttu-id="34fda-108">Installare il [`Microsoft.AspNetCore.Mvc.NewtonsoftJson`](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.NewtonsoftJson/) pacchetto NuGet.</span><span class="sxs-lookup"><span data-stu-id="34fda-108">Install the [`Microsoft.AspNetCore.Mvc.NewtonsoftJson`](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.NewtonsoftJson/) NuGet package.</span></span>
1. <span data-ttu-id="34fda-109">Aggiornare il metodo del progetto `Startup.ConfigureServices` in modo da chiamare <xref:Microsoft.Extensions.DependencyInjection.NewtonsoftJsonMvcBuilderExtensions.AddNewtonsoftJson*> .</span><span class="sxs-lookup"><span data-stu-id="34fda-109">Update the project's `Startup.ConfigureServices` method to call <xref:Microsoft.Extensions.DependencyInjection.NewtonsoftJsonMvcBuilderExtensions.AddNewtonsoftJson*>.</span></span> <span data-ttu-id="34fda-110">Ad esempio:</span><span class="sxs-lookup"><span data-stu-id="34fda-110">For example:</span></span>

    ```csharp
    services
        .AddControllersWithViews()
        .AddNewtonsoftJson();
    ```

<span data-ttu-id="34fda-111">`AddNewtonsoftJson` è compatibile con i metodi di registrazione del servizio MVC:</span><span class="sxs-lookup"><span data-stu-id="34fda-111">`AddNewtonsoftJson` is compatible with the MVC service registration methods:</span></span>

* <xref:Microsoft.Extensions.DependencyInjection.MvcServiceCollectionExtensions.AddRazorPages*>
* <xref:Microsoft.Extensions.DependencyInjection.MvcServiceCollectionExtensions.AddControllersWithViews*>
* <xref:Microsoft.Extensions.DependencyInjection.MvcServiceCollectionExtensions.AddControllers*>

## <a name="json-patch-addnewtonsoftjson-and-systemtextjson"></a><span data-ttu-id="34fda-112">Patch JSON, AddNewtonsoftJson e System.Text.Js</span><span class="sxs-lookup"><span data-stu-id="34fda-112">JSON Patch, AddNewtonsoftJson, and System.Text.Json</span></span>

<span data-ttu-id="34fda-113">`AddNewtonsoftJson` sostituisce i `System.Text.Json` formattatori di input e output basati su usati per formattare **tutto** il contenuto JSON.</span><span class="sxs-lookup"><span data-stu-id="34fda-113">`AddNewtonsoftJson` replaces the `System.Text.Json`-based input and output formatters used for formatting **all** JSON content.</span></span> <span data-ttu-id="34fda-114">Per aggiungere il supporto per la patch JSON usando `Newtonsoft.Json` , lasciando invariati gli altri formattatori, aggiornare il metodo del progetto `Startup.ConfigureServices` come indicato di seguito:</span><span class="sxs-lookup"><span data-stu-id="34fda-114">To add support for JSON Patch using `Newtonsoft.Json`, while leaving the other formatters unchanged, update the project's `Startup.ConfigureServices` method as follows:</span></span>

[!code-csharp[](jsonpatch/samples/3.0/WebApp1/Startup.cs?name=snippet)]

<span data-ttu-id="34fda-115">Il codice precedente richiede il `Microsoft.AspNetCore.Mvc.NewtonsoftJson` pacchetto e le `using` istruzioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="34fda-115">The preceding code requires the `Microsoft.AspNetCore.Mvc.NewtonsoftJson` package and the following `using` statements:</span></span>

[!code-csharp[](jsonpatch/samples/3.0/WebApp1/Startup.cs?name=snippet1)]

## <a name="patch-http-request-method"></a><span data-ttu-id="34fda-116">Metodo di richiesta HTTP PATCH</span><span class="sxs-lookup"><span data-stu-id="34fda-116">PATCH HTTP request method</span></span>

<span data-ttu-id="34fda-117">I metodi PUT e [PATCH](https://tools.ietf.org/html/rfc5789) vengono usati per aggiornare una risorsa esistente.</span><span class="sxs-lookup"><span data-stu-id="34fda-117">The PUT and [PATCH](https://tools.ietf.org/html/rfc5789) methods are used to update an existing resource.</span></span> <span data-ttu-id="34fda-118">La differenza tra i due metodi è che PUT sostituisce l'intera risorsa, mentre PATCH specifica solo le modifiche.</span><span class="sxs-lookup"><span data-stu-id="34fda-118">The difference between them is that PUT replaces the entire resource, while PATCH specifies only the changes.</span></span>

## <a name="json-patch"></a><span data-ttu-id="34fda-119">JSON Patch</span><span class="sxs-lookup"><span data-stu-id="34fda-119">JSON Patch</span></span>

<span data-ttu-id="34fda-120">[JSON Patch](https://tools.ietf.org/html/rfc6902) è un formato per specificare gli aggiornamenti da applicare a una risorsa.</span><span class="sxs-lookup"><span data-stu-id="34fda-120">[JSON Patch](https://tools.ietf.org/html/rfc6902) is a format for specifying updates to be applied to a resource.</span></span> <span data-ttu-id="34fda-121">Un documento JSON Patch è una matrice di *operazioni*.</span><span class="sxs-lookup"><span data-stu-id="34fda-121">A JSON Patch document has an array of *operations*.</span></span> <span data-ttu-id="34fda-122">Ogni operazione identifica un tipo specifico di modifica.</span><span class="sxs-lookup"><span data-stu-id="34fda-122">Each operation identifies a particular type of change.</span></span> <span data-ttu-id="34fda-123">Esempi di tali modifiche includono l'aggiunta di un elemento di matrice o la sostituzione di un valore della proprietà.</span><span class="sxs-lookup"><span data-stu-id="34fda-123">Examples of such changes include adding an array element or replacing a property value.</span></span>

<span data-ttu-id="34fda-124">Ad esempio, i documenti JSON seguenti rappresentano una risorsa, un documento di patch JSON per la risorsa e il risultato dell'applicazione delle operazioni patch.</span><span class="sxs-lookup"><span data-stu-id="34fda-124">For example, the following JSON documents represent a resource, a JSON Patch document for the resource, and the result of applying the Patch operations.</span></span>

### <a name="resource-example"></a><span data-ttu-id="34fda-125">Esempio di risorsa</span><span class="sxs-lookup"><span data-stu-id="34fda-125">Resource example</span></span>

[!code-json[](jsonpatch/samples/2.2/JSON/customer.json)]

### <a name="json-patch-example"></a><span data-ttu-id="34fda-126">Esempio di patch JSON</span><span class="sxs-lookup"><span data-stu-id="34fda-126">JSON patch example</span></span>

[!code-json[](jsonpatch/samples/2.2/JSON/add.json)]

<span data-ttu-id="34fda-127">Nel codice JSON precedente:</span><span class="sxs-lookup"><span data-stu-id="34fda-127">In the preceding JSON:</span></span>

* <span data-ttu-id="34fda-128">La proprietà `op` indica il tipo di operazione.</span><span class="sxs-lookup"><span data-stu-id="34fda-128">The `op` property indicates the type of operation.</span></span>
* <span data-ttu-id="34fda-129">La proprietà `path` indica l'elemento da aggiornare.</span><span class="sxs-lookup"><span data-stu-id="34fda-129">The `path` property indicates the element to update.</span></span>
* <span data-ttu-id="34fda-130">La proprietà `value` fornisce il nuovo valore.</span><span class="sxs-lookup"><span data-stu-id="34fda-130">The `value` property provides the new value.</span></span>

### <a name="resource-after-patch"></a><span data-ttu-id="34fda-131">Risorsa dopo la patch</span><span class="sxs-lookup"><span data-stu-id="34fda-131">Resource after patch</span></span>

<span data-ttu-id="34fda-132">Ecco la risorsa dopo aver applicato il documento JSON Patch precedente:</span><span class="sxs-lookup"><span data-stu-id="34fda-132">Here's the resource after applying the preceding JSON Patch document:</span></span>

```json
{
  "customerName": "Barry",
  "orders": [
    {
      "orderName": "Order0",
      "orderType": null
    },
    {
      "orderName": "Order1",
      "orderType": null
    },
    {
      "orderName": "Order2",
      "orderType": null
    }
  ]
}
```

<span data-ttu-id="34fda-133">Le modifiche apportate applicando un documento Patch JSON a una risorsa sono atomiche.</span><span class="sxs-lookup"><span data-stu-id="34fda-133">The changes made by applying a JSON Patch document to a resource are atomic.</span></span> <span data-ttu-id="34fda-134">Se un'operazione nell'elenco ha esito negativo, non viene applicata alcuna operazione nell'elenco.</span><span class="sxs-lookup"><span data-stu-id="34fda-134">If any operation in the list fails, no operation in the list is applied.</span></span>

## <a name="path-syntax"></a><span data-ttu-id="34fda-135">Sintassi di path</span><span class="sxs-lookup"><span data-stu-id="34fda-135">Path syntax</span></span>

<span data-ttu-id="34fda-136">La proprietà [path](https://tools.ietf.org/html/rfc6901) di un oggetto operazione include barre tra livelli.</span><span class="sxs-lookup"><span data-stu-id="34fda-136">The [path](https://tools.ietf.org/html/rfc6901) property of an operation object has slashes between levels.</span></span> <span data-ttu-id="34fda-137">Ad esempio: `"/address/zipCode"`.</span><span class="sxs-lookup"><span data-stu-id="34fda-137">For example, `"/address/zipCode"`.</span></span>

<span data-ttu-id="34fda-138">Vengono usati indici in base zero per specificare gli elementi della matrice.</span><span class="sxs-lookup"><span data-stu-id="34fda-138">Zero-based indexes are used to specify array elements.</span></span> <span data-ttu-id="34fda-139">Il primo elemento della matrice `addresses` è in corrispondenza di `/addresses/0`.</span><span class="sxs-lookup"><span data-stu-id="34fda-139">The first element of the `addresses` array would be at `/addresses/0`.</span></span> <span data-ttu-id="34fda-140">Fino alla `add` fine di una matrice, usare un trattino ( `-` ) invece di un numero di indice: `/addresses/-` .</span><span class="sxs-lookup"><span data-stu-id="34fda-140">To `add` to the end of an array, use a hyphen (`-`) rather than an index number: `/addresses/-`.</span></span>

### <a name="operations"></a><span data-ttu-id="34fda-141">Gestione operativa</span><span class="sxs-lookup"><span data-stu-id="34fda-141">Operations</span></span>

<span data-ttu-id="34fda-142">La tabella seguente mostra le operazioni supportate in base a quanto definito nella [specifica JSON Patch](https://tools.ietf.org/html/rfc6902):</span><span class="sxs-lookup"><span data-stu-id="34fda-142">The following table shows supported operations as defined in the [JSON Patch specification](https://tools.ietf.org/html/rfc6902):</span></span>

|<span data-ttu-id="34fda-143">Operazione</span><span class="sxs-lookup"><span data-stu-id="34fda-143">Operation</span></span>  | <span data-ttu-id="34fda-144">Note</span><span class="sxs-lookup"><span data-stu-id="34fda-144">Notes</span></span> |
|-----------|--------------------------------|
| `add`     | <span data-ttu-id="34fda-145">Aggiunge una proprietà o un elemento della matrice.</span><span class="sxs-lookup"><span data-stu-id="34fda-145">Add a property or array element.</span></span> <span data-ttu-id="34fda-146">Per la proprietà esistente: impostare il valore.</span><span class="sxs-lookup"><span data-stu-id="34fda-146">For existing property: set value.</span></span>|
| `remove`  | <span data-ttu-id="34fda-147">Rimuove una proprietà o un elemento della matrice.</span><span class="sxs-lookup"><span data-stu-id="34fda-147">Remove a property or array element.</span></span> |
| `replace` | <span data-ttu-id="34fda-148">Uguale a `remove` seguita da `add` nella stessa posizione.</span><span class="sxs-lookup"><span data-stu-id="34fda-148">Same as `remove` followed by `add` at same location.</span></span> |
| `move`    | <span data-ttu-id="34fda-149">Uguale a `remove` dall'origine seguita da `add` alla destinazione usando il valore dell'origine.</span><span class="sxs-lookup"><span data-stu-id="34fda-149">Same as `remove` from source followed by `add` to destination using value from source.</span></span> |
| `copy`    | <span data-ttu-id="34fda-150">Uguale a `add` alla destinazione usando il valore dell'origine.</span><span class="sxs-lookup"><span data-stu-id="34fda-150">Same as `add` to destination using value from source.</span></span> |
| `test`    | <span data-ttu-id="34fda-151">Restituisce il codice di stato di richiesta riuscita se il valore in `path` = `value` fornito.</span><span class="sxs-lookup"><span data-stu-id="34fda-151">Return success status code if value at `path` = provided `value`.</span></span>|

## <a name="json-patch-in-aspnet-core"></a><span data-ttu-id="34fda-152">Patch JSON in ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="34fda-152">JSON Patch in ASP.NET Core</span></span>

<span data-ttu-id="34fda-153">L'implementazione ASP.NET Core di JSON Patch viene fornita nel pacchetto NuGet [Microsoft.AspNetCore.JsonPatch](https://www.nuget.org/packages/microsoft.aspnetcore.jsonpatch/).</span><span class="sxs-lookup"><span data-stu-id="34fda-153">The ASP.NET Core implementation of JSON Patch is provided in the [Microsoft.AspNetCore.JsonPatch](https://www.nuget.org/packages/microsoft.aspnetcore.jsonpatch/) NuGet package.</span></span>

## <a name="action-method-code"></a><span data-ttu-id="34fda-154">Codice del metodo di azione</span><span class="sxs-lookup"><span data-stu-id="34fda-154">Action method code</span></span>

<span data-ttu-id="34fda-155">In un controller API un metodo di azione per JSON Patch:</span><span class="sxs-lookup"><span data-stu-id="34fda-155">In an API controller, an action method for JSON Patch:</span></span>

* <span data-ttu-id="34fda-156">Viene annotato con l'attributo `HttpPatch`.</span><span class="sxs-lookup"><span data-stu-id="34fda-156">Is annotated with the `HttpPatch` attribute.</span></span>
* <span data-ttu-id="34fda-157">Accetta un oggetto `JsonPatchDocument<T>` , in genere con `[FromBody]` .</span><span class="sxs-lookup"><span data-stu-id="34fda-157">Accepts a `JsonPatchDocument<T>`, typically with `[FromBody]`.</span></span>
* <span data-ttu-id="34fda-158">Chiama `ApplyTo` nel documento di patch per applicare le modifiche.</span><span class="sxs-lookup"><span data-stu-id="34fda-158">Calls `ApplyTo` on the patch document to apply the changes.</span></span>

<span data-ttu-id="34fda-159">Ecco un esempio:</span><span class="sxs-lookup"><span data-stu-id="34fda-159">Here's an example:</span></span>

[!code-csharp[](jsonpatch/samples/2.2/Controllers/HomeController.cs?name=snippet_PatchAction&highlight=1,3,9)]

<span data-ttu-id="34fda-160">Questo codice dall'app di esempio funziona con il `Customer` modello seguente:</span><span class="sxs-lookup"><span data-stu-id="34fda-160">This code from the sample app works with the following `Customer` model:</span></span>

[!code-csharp[](jsonpatch/samples/2.2/Models/Customer.cs?name=snippet_Customer)]

[!code-csharp[](jsonpatch/samples/2.2/Models/Order.cs?name=snippet_Order)]

<span data-ttu-id="34fda-161">Il metodo di azione di esempio:</span><span class="sxs-lookup"><span data-stu-id="34fda-161">The sample action method:</span></span>

* <span data-ttu-id="34fda-162">Costruisce un oggetto `Customer`.</span><span class="sxs-lookup"><span data-stu-id="34fda-162">Constructs a `Customer`.</span></span>
* <span data-ttu-id="34fda-163">Applica la patch.</span><span class="sxs-lookup"><span data-stu-id="34fda-163">Applies the patch.</span></span>
* <span data-ttu-id="34fda-164">Restituisce il risultato nel corpo della risposta.</span><span class="sxs-lookup"><span data-stu-id="34fda-164">Returns the result in the body of the response.</span></span>

<span data-ttu-id="34fda-165">In un'app reale il codice recupererebbe i dati da un archivio, ad esempio un database, e aggiornerebbe il database dopo aver applicato la patch.</span><span class="sxs-lookup"><span data-stu-id="34fda-165">In a real app, the code would retrieve the data from a store such as a database and update the database after applying the patch.</span></span>

### <a name="model-state"></a><span data-ttu-id="34fda-166">Stato del modello</span><span class="sxs-lookup"><span data-stu-id="34fda-166">Model state</span></span>

<span data-ttu-id="34fda-167">L'esempio di metodo di azione precedente chiama un overload di `ApplyTo` che accetta lo stato del modello come uno dei propri parametri.</span><span class="sxs-lookup"><span data-stu-id="34fda-167">The preceding action method example calls an overload of `ApplyTo` that takes model state as one of its parameters.</span></span> <span data-ttu-id="34fda-168">Con questa opzione, è possibile ottenere messaggi di errore nelle risposte.</span><span class="sxs-lookup"><span data-stu-id="34fda-168">With this option, you can get error messages in responses.</span></span> <span data-ttu-id="34fda-169">L'esempio seguente mostra il corpo di una risposta 400 Richiesta non valida per un'operazione `test`:</span><span class="sxs-lookup"><span data-stu-id="34fda-169">The following example shows the body of a 400 Bad Request response for a `test` operation:</span></span>

```json
{
    "Customer": [
        "The current value 'John' at path 'customerName' is not equal to the test value 'Nancy'."
    ]
}
```

### <a name="dynamic-objects"></a><span data-ttu-id="34fda-170">Oggetti dinamici</span><span class="sxs-lookup"><span data-stu-id="34fda-170">Dynamic objects</span></span>

<span data-ttu-id="34fda-171">Nell'esempio di metodo di azione seguente viene illustrato come applicare una patch a un oggetto dinamico:</span><span class="sxs-lookup"><span data-stu-id="34fda-171">The following action method example shows how to apply a patch to a dynamic object:</span></span>

[!code-csharp[](jsonpatch/samples/2.2/Controllers/HomeController.cs?name=snippet_Dynamic)]

## <a name="the-add-operation"></a><span data-ttu-id="34fda-172">Operazione add</span><span class="sxs-lookup"><span data-stu-id="34fda-172">The add operation</span></span>

* <span data-ttu-id="34fda-173">Se `path` punta a un elemento della matrice: inserisce un nuovo elemento prima di quello specificato da `path`.</span><span class="sxs-lookup"><span data-stu-id="34fda-173">If `path` points to an array element: inserts new element before the one specified by `path`.</span></span>
* <span data-ttu-id="34fda-174">Se `path` punta a una proprietà: imposta il valore della proprietà.</span><span class="sxs-lookup"><span data-stu-id="34fda-174">If `path` points to a property: sets the property value.</span></span>
* <span data-ttu-id="34fda-175">Se `path` punta a una posizione che non esiste:</span><span class="sxs-lookup"><span data-stu-id="34fda-175">If `path` points to a nonexistent location:</span></span>
  * <span data-ttu-id="34fda-176">Se la risorsa cui applicare la patch è un oggetto dinamico: aggiunge una proprietà.</span><span class="sxs-lookup"><span data-stu-id="34fda-176">If the resource to patch is a dynamic object: adds a property.</span></span>
  * <span data-ttu-id="34fda-177">Se la risorsa cui applicare la patch è un oggetto statico: la richiesta non riesce.</span><span class="sxs-lookup"><span data-stu-id="34fda-177">If the resource to patch is a static object: the request fails.</span></span>

<span data-ttu-id="34fda-178">Il documento di patch di esempio seguente imposta il valore di `CustomerName` e aggiunge un oggetto `Order` alla fine della matrice `Orders`.</span><span class="sxs-lookup"><span data-stu-id="34fda-178">The following sample patch document sets the value of `CustomerName` and adds an `Order` object to the end of the `Orders` array.</span></span>

[!code-json[](jsonpatch/samples/2.2/JSON/add.json)]

## <a name="the-remove-operation"></a><span data-ttu-id="34fda-179">Operazione remove</span><span class="sxs-lookup"><span data-stu-id="34fda-179">The remove operation</span></span>

* <span data-ttu-id="34fda-180">Se `path` punta a un elemento della matrice: rimuove l'elemento.</span><span class="sxs-lookup"><span data-stu-id="34fda-180">If `path` points to an array element: removes the element.</span></span>
* <span data-ttu-id="34fda-181">Se `path` punta a una proprietà:</span><span class="sxs-lookup"><span data-stu-id="34fda-181">If `path` points to a property:</span></span>
  * <span data-ttu-id="34fda-182">Se la risorsa cui applicare la patch è un oggetto dinamico: rimuove la proprietà.</span><span class="sxs-lookup"><span data-stu-id="34fda-182">If resource to patch is a dynamic object: removes the property.</span></span>
  * <span data-ttu-id="34fda-183">Se la risorsa cui applicare la patch è un oggetto statico:</span><span class="sxs-lookup"><span data-stu-id="34fda-183">If resource to patch is a static object:</span></span>
    * <span data-ttu-id="34fda-184">Se la proprietà ammette valori Null: la imposta su Null.</span><span class="sxs-lookup"><span data-stu-id="34fda-184">If the property is nullable: sets it to null.</span></span>
    * <span data-ttu-id="34fda-185">Se la proprietà non ammette valori Null: la imposta su `default<T>`.</span><span class="sxs-lookup"><span data-stu-id="34fda-185">If the property is non-nullable, sets it to `default<T>`.</span></span>

<span data-ttu-id="34fda-186">Il documento di patch di esempio seguente imposta `CustomerName` su null ed Elimina `Orders[0]` :</span><span class="sxs-lookup"><span data-stu-id="34fda-186">The following sample patch document sets `CustomerName` to null and deletes `Orders[0]`:</span></span>

[!code-json[](jsonpatch/samples/2.2/JSON/remove.json)]

## <a name="the-replace-operation"></a><span data-ttu-id="34fda-187">Operazione replace</span><span class="sxs-lookup"><span data-stu-id="34fda-187">The replace operation</span></span>

<span data-ttu-id="34fda-188">Questa operazione equivale a livello funzionale all'operazione `remove` seguita da un'operazione `add`.</span><span class="sxs-lookup"><span data-stu-id="34fda-188">This operation is functionally the same as a `remove` followed by an `add`.</span></span>

<span data-ttu-id="34fda-189">Nel documento della patch di esempio seguente viene impostato il valore di `CustomerName` e viene sostituito `Orders[0]` con un nuovo `Order` oggetto:</span><span class="sxs-lookup"><span data-stu-id="34fda-189">The following sample patch document sets the value of `CustomerName` and replaces `Orders[0]`with a new `Order` object:</span></span>

[!code-json[](jsonpatch/samples/2.2/JSON/replace.json)]

## <a name="the-move-operation"></a><span data-ttu-id="34fda-190">Operazione move</span><span class="sxs-lookup"><span data-stu-id="34fda-190">The move operation</span></span>

* <span data-ttu-id="34fda-191">Se `path` punta a un elemento della matrice: copia l'elemento `from` nella posizione dell'elemento `path`, quindi esegue un'operazione `remove` sull'elemento `from`.</span><span class="sxs-lookup"><span data-stu-id="34fda-191">If `path` points to an array element: copies `from` element to location of `path` element, then runs a `remove` operation on the `from` element.</span></span>
* <span data-ttu-id="34fda-192">Se `path` punta a una proprietà: copia il valore della proprietà `from` nella proprietà `path`, quindi esegue un'operazione `remove` sulla proprietà `from`.</span><span class="sxs-lookup"><span data-stu-id="34fda-192">If `path` points to a property: copies value of `from` property to `path` property, then runs a `remove` operation on the `from` property.</span></span>
* <span data-ttu-id="34fda-193">Se `path` punta a una proprietà che non esiste:</span><span class="sxs-lookup"><span data-stu-id="34fda-193">If `path` points to a nonexistent property:</span></span>
  * <span data-ttu-id="34fda-194">Se la risorsa cui applicare la patch è un oggetto statico: la richiesta non riesce.</span><span class="sxs-lookup"><span data-stu-id="34fda-194">If the resource to patch is a static object: the request fails.</span></span>
  * <span data-ttu-id="34fda-195">Se la risorsa cui applicare la patch è un oggetto dinamico: copia la proprietà `from` nella posizione indicata da `path`, quindi esegue un'operazione `remove` sulla proprietà `from`.</span><span class="sxs-lookup"><span data-stu-id="34fda-195">If the resource to patch is a dynamic object: copies `from` property to location indicated by `path`, then runs a `remove` operation on the `from` property.</span></span>

<span data-ttu-id="34fda-196">Il documento di patch di esempio seguente:</span><span class="sxs-lookup"><span data-stu-id="34fda-196">The following sample patch document:</span></span>

* <span data-ttu-id="34fda-197">Copia il valore di `Orders[0].OrderName` in `CustomerName`.</span><span class="sxs-lookup"><span data-stu-id="34fda-197">Copies the value of `Orders[0].OrderName` to `CustomerName`.</span></span>
* <span data-ttu-id="34fda-198">Imposta `Orders[0].OrderName` su Null.</span><span class="sxs-lookup"><span data-stu-id="34fda-198">Sets `Orders[0].OrderName` to null.</span></span>
* <span data-ttu-id="34fda-199">Sposta `Orders[1]` prima di `Orders[0]`.</span><span class="sxs-lookup"><span data-stu-id="34fda-199">Moves `Orders[1]` to before `Orders[0]`.</span></span>

[!code-json[](jsonpatch/samples/2.2/JSON/move.json)]

## <a name="the-copy-operation"></a><span data-ttu-id="34fda-200">Operazione copy</span><span class="sxs-lookup"><span data-stu-id="34fda-200">The copy operation</span></span>

<span data-ttu-id="34fda-201">Questa operazione equivale a livello funzionale all'operazione `move` senza il passaggio `remove` finale.</span><span class="sxs-lookup"><span data-stu-id="34fda-201">This operation is functionally the same as a `move` operation without the final `remove` step.</span></span>

<span data-ttu-id="34fda-202">Il documento di patch di esempio seguente:</span><span class="sxs-lookup"><span data-stu-id="34fda-202">The following sample patch document:</span></span>

* <span data-ttu-id="34fda-203">Copia il valore di `Orders[0].OrderName` in `CustomerName`.</span><span class="sxs-lookup"><span data-stu-id="34fda-203">Copies the value of `Orders[0].OrderName` to `CustomerName`.</span></span>
* <span data-ttu-id="34fda-204">Inserisce una copia di `Orders[1]` prima di `Orders[0]`.</span><span class="sxs-lookup"><span data-stu-id="34fda-204">Inserts a copy of `Orders[1]` before `Orders[0]`.</span></span>

[!code-json[](jsonpatch/samples/2.2/JSON/copy.json)]

## <a name="the-test-operation"></a><span data-ttu-id="34fda-205">Operazione test</span><span class="sxs-lookup"><span data-stu-id="34fda-205">The test operation</span></span>

<span data-ttu-id="34fda-206">Se il valore nella posizione indicata da `path` è diverso dal valore fornito in `value`, la richiesta non riesce.</span><span class="sxs-lookup"><span data-stu-id="34fda-206">If the value at the location indicated by `path` is different from the value provided in `value`, the request fails.</span></span> <span data-ttu-id="34fda-207">In questo caso, l'intera richiesta PATCH non riesce anche se tutte le altre operazioni nel documento di patch potrebbero riuscire.</span><span class="sxs-lookup"><span data-stu-id="34fda-207">In that case, the whole PATCH request fails even if all other operations in the patch document would otherwise succeed.</span></span>

<span data-ttu-id="34fda-208">L'operazione `test` viene usata in genere per impedire un aggiornamento in caso di conflitto di concorrenza.</span><span class="sxs-lookup"><span data-stu-id="34fda-208">The `test` operation is commonly used to prevent an update when there's a concurrency conflict.</span></span>

<span data-ttu-id="34fda-209">Il documento di patch di esempio seguente non ha alcun effetto se il valore iniziale di `CustomerName` è "John", perché il test non riesce:</span><span class="sxs-lookup"><span data-stu-id="34fda-209">The following sample patch document has no effect if the initial value of `CustomerName` is "John", because the test fails:</span></span>

[!code-json[](jsonpatch/samples/2.2/JSON/test-fail.json)]

## <a name="get-the-code"></a><span data-ttu-id="34fda-210">Ottenere il codice</span><span class="sxs-lookup"><span data-stu-id="34fda-210">Get the code</span></span>

<span data-ttu-id="34fda-211">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/web-api/jsonpatch/samples).</span><span class="sxs-lookup"><span data-stu-id="34fda-211">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/web-api/jsonpatch/samples).</span></span> <span data-ttu-id="34fda-212">([Come scaricare](xref:index#how-to-download-a-sample)).</span><span class="sxs-lookup"><span data-stu-id="34fda-212">([How to download](xref:index#how-to-download-a-sample)).</span></span>

<span data-ttu-id="34fda-213">Per testare l'esempio, eseguire l'app e inviare richieste HTTP con le impostazioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="34fda-213">To test the sample, run the app and send HTTP requests with the following settings:</span></span>

* <span data-ttu-id="34fda-214">URL: `http://localhost:{port}/jsonpatch/jsonpatchwithmodelstate`</span><span class="sxs-lookup"><span data-stu-id="34fda-214">URL: `http://localhost:{port}/jsonpatch/jsonpatchwithmodelstate`</span></span>
* <span data-ttu-id="34fda-215">Metodo HTTP: `PATCH`</span><span class="sxs-lookup"><span data-stu-id="34fda-215">HTTP method: `PATCH`</span></span>
* <span data-ttu-id="34fda-216">Intestazione: `Content-Type: application/json-patch+json`</span><span class="sxs-lookup"><span data-stu-id="34fda-216">Header: `Content-Type: application/json-patch+json`</span></span>
* <span data-ttu-id="34fda-217">Body: copiare e incollare uno degli esempi di documenti patch JSON dalla cartella del progetto *JSON* .</span><span class="sxs-lookup"><span data-stu-id="34fda-217">Body: Copy and paste one of the JSON patch document samples from the *JSON* project folder.</span></span>

## <a name="additional-resources"></a><span data-ttu-id="34fda-218">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="34fda-218">Additional resources</span></span>

* [<span data-ttu-id="34fda-219">Specifica del metodo PATCH IETF RFC 5789</span><span class="sxs-lookup"><span data-stu-id="34fda-219">IETF RFC 5789 PATCH method specification</span></span>](https://tools.ietf.org/html/rfc5789)
* [<span data-ttu-id="34fda-220">Specifica JSON Patch IETF RFC 6902</span><span class="sxs-lookup"><span data-stu-id="34fda-220">IETF RFC 6902 JSON Patch specification</span></span>](https://tools.ietf.org/html/rfc6902)
* [<span data-ttu-id="34fda-221">Specifica del formato di percorso JSON Patch IETF RFC 6901</span><span class="sxs-lookup"><span data-stu-id="34fda-221">IETF RFC 6901 JSON Patch path format spec</span></span>](https://tools.ietf.org/html/rfc6901)
* <span data-ttu-id="34fda-222">[Documentazione di JSON Patch](https://jsonpatch.com/).</span><span class="sxs-lookup"><span data-stu-id="34fda-222">[JSON Patch documentation](https://jsonpatch.com/).</span></span> <span data-ttu-id="34fda-223">Include collegamenti a risorse per la creazione di documenti JSON Patch.</span><span class="sxs-lookup"><span data-stu-id="34fda-223">Includes links to resources for creating JSON Patch documents.</span></span>
* [<span data-ttu-id="34fda-224">Codice sorgente JSON Patch in ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="34fda-224">ASP.NET Core JSON Patch source code</span></span>](https://github.com/dotnet/AspNetCore/tree/main/src/Features/JsonPatch/src)

::: moniker-end

::: moniker range="< aspnetcore-3.0"

<span data-ttu-id="34fda-225">Questo articolo descrive come gestire le richieste JSON Patch in un'API Web ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="34fda-225">This article explains how to handle JSON Patch requests in an ASP.NET Core web API.</span></span>

## <a name="patch-http-request-method"></a><span data-ttu-id="34fda-226">Metodo di richiesta HTTP PATCH</span><span class="sxs-lookup"><span data-stu-id="34fda-226">PATCH HTTP request method</span></span>

<span data-ttu-id="34fda-227">I metodi PUT e [PATCH](https://tools.ietf.org/html/rfc5789) vengono usati per aggiornare una risorsa esistente.</span><span class="sxs-lookup"><span data-stu-id="34fda-227">The PUT and [PATCH](https://tools.ietf.org/html/rfc5789) methods are used to update an existing resource.</span></span> <span data-ttu-id="34fda-228">La differenza tra i due metodi è che PUT sostituisce l'intera risorsa, mentre PATCH specifica solo le modifiche.</span><span class="sxs-lookup"><span data-stu-id="34fda-228">The difference between them is that PUT replaces the entire resource, while PATCH specifies only the changes.</span></span>

## <a name="json-patch"></a><span data-ttu-id="34fda-229">JSON Patch</span><span class="sxs-lookup"><span data-stu-id="34fda-229">JSON Patch</span></span>

<span data-ttu-id="34fda-230">[JSON Patch](https://tools.ietf.org/html/rfc6902) è un formato per specificare gli aggiornamenti da applicare a una risorsa.</span><span class="sxs-lookup"><span data-stu-id="34fda-230">[JSON Patch](https://tools.ietf.org/html/rfc6902) is a format for specifying updates to be applied to a resource.</span></span> <span data-ttu-id="34fda-231">Un documento JSON Patch è una matrice di *operazioni*.</span><span class="sxs-lookup"><span data-stu-id="34fda-231">A JSON Patch document has an array of *operations*.</span></span> <span data-ttu-id="34fda-232">Ogni operazione identifica un determinato tipo di modifica, ad esempio l'aggiunta di un elemento della matrice o la sostituzione di un valore di proprietà.</span><span class="sxs-lookup"><span data-stu-id="34fda-232">Each operation identifies a particular type of change, such as add an array element or replace a property value.</span></span>

<span data-ttu-id="34fda-233">Ad esempio, i documenti JSON seguenti rappresentano una risorsa, un documento di patch JSON per la risorsa e il risultato dell'applicazione delle operazioni patch.</span><span class="sxs-lookup"><span data-stu-id="34fda-233">For example, the following JSON documents represent a resource, a JSON patch document for the resource, and the result of applying the patch operations.</span></span>

### <a name="resource-example"></a><span data-ttu-id="34fda-234">Esempio di risorsa</span><span class="sxs-lookup"><span data-stu-id="34fda-234">Resource example</span></span>

[!code-json[](jsonpatch/samples/2.2/JSON/customer.json)]

### <a name="json-patch-example"></a><span data-ttu-id="34fda-235">Esempio di patch JSON</span><span class="sxs-lookup"><span data-stu-id="34fda-235">JSON patch example</span></span>

[!code-json[](jsonpatch/samples/2.2/JSON/add.json)]

<span data-ttu-id="34fda-236">Nel codice JSON precedente:</span><span class="sxs-lookup"><span data-stu-id="34fda-236">In the preceding JSON:</span></span>

* <span data-ttu-id="34fda-237">La proprietà `op` indica il tipo di operazione.</span><span class="sxs-lookup"><span data-stu-id="34fda-237">The `op` property indicates the type of operation.</span></span>
* <span data-ttu-id="34fda-238">La proprietà `path` indica l'elemento da aggiornare.</span><span class="sxs-lookup"><span data-stu-id="34fda-238">The `path` property indicates the element to update.</span></span>
* <span data-ttu-id="34fda-239">La proprietà `value` fornisce il nuovo valore.</span><span class="sxs-lookup"><span data-stu-id="34fda-239">The `value` property provides the new value.</span></span>

### <a name="resource-after-patch"></a><span data-ttu-id="34fda-240">Risorsa dopo la patch</span><span class="sxs-lookup"><span data-stu-id="34fda-240">Resource after patch</span></span>

<span data-ttu-id="34fda-241">Ecco la risorsa dopo aver applicato il documento JSON Patch precedente:</span><span class="sxs-lookup"><span data-stu-id="34fda-241">Here's the resource after applying the preceding JSON Patch document:</span></span>

```json
{
  "customerName": "Barry",
  "orders": [
    {
      "orderName": "Order0",
      "orderType": null
    },
    {
      "orderName": "Order1",
      "orderType": null
    },
    {
      "orderName": "Order2",
      "orderType": null
    }
  ]
}
```

<span data-ttu-id="34fda-242">Le modifiche apportate tramite l'applicazione di un documento JSON Patch a una risorsa sono atomiche: se qualsiasi operazione nell'elenco non riesce, non viene applicata alcuna operazione contenuta nell'elenco.</span><span class="sxs-lookup"><span data-stu-id="34fda-242">The changes made by applying a JSON Patch document to a resource are atomic: if any operation in the list fails, no operation in the list is applied.</span></span>

## <a name="path-syntax"></a><span data-ttu-id="34fda-243">Sintassi di path</span><span class="sxs-lookup"><span data-stu-id="34fda-243">Path syntax</span></span>

<span data-ttu-id="34fda-244">La proprietà [path](https://tools.ietf.org/html/rfc6901) di un oggetto operazione include barre tra livelli.</span><span class="sxs-lookup"><span data-stu-id="34fda-244">The [path](https://tools.ietf.org/html/rfc6901) property of an operation object has slashes between levels.</span></span> <span data-ttu-id="34fda-245">Ad esempio: `"/address/zipCode"`.</span><span class="sxs-lookup"><span data-stu-id="34fda-245">For example, `"/address/zipCode"`.</span></span>

<span data-ttu-id="34fda-246">Vengono usati indici in base zero per specificare gli elementi della matrice.</span><span class="sxs-lookup"><span data-stu-id="34fda-246">Zero-based indexes are used to specify array elements.</span></span> <span data-ttu-id="34fda-247">Il primo elemento della matrice `addresses` è in corrispondenza di `/addresses/0`.</span><span class="sxs-lookup"><span data-stu-id="34fda-247">The first element of the `addresses` array would be at `/addresses/0`.</span></span> <span data-ttu-id="34fda-248">Per aggiungere (`add`) un elemento alla fine della matrice, usare un trattino (-) invece di un numero di indice: `/addresses/-`.</span><span class="sxs-lookup"><span data-stu-id="34fda-248">To `add` to the end of an array, use a hyphen (-) rather than an index number: `/addresses/-`.</span></span>

### <a name="operations"></a><span data-ttu-id="34fda-249">Gestione operativa</span><span class="sxs-lookup"><span data-stu-id="34fda-249">Operations</span></span>

<span data-ttu-id="34fda-250">La tabella seguente mostra le operazioni supportate in base a quanto definito nella [specifica JSON Patch](https://tools.ietf.org/html/rfc6902):</span><span class="sxs-lookup"><span data-stu-id="34fda-250">The following table shows supported operations as defined in the [JSON Patch specification](https://tools.ietf.org/html/rfc6902):</span></span>

|<span data-ttu-id="34fda-251">Operazione</span><span class="sxs-lookup"><span data-stu-id="34fda-251">Operation</span></span>  | <span data-ttu-id="34fda-252">Note</span><span class="sxs-lookup"><span data-stu-id="34fda-252">Notes</span></span> |
|-----------|--------------------------------|
| `add`     | <span data-ttu-id="34fda-253">Aggiunge una proprietà o un elemento della matrice.</span><span class="sxs-lookup"><span data-stu-id="34fda-253">Add a property or array element.</span></span> <span data-ttu-id="34fda-254">Per la proprietà esistente: impostare il valore.</span><span class="sxs-lookup"><span data-stu-id="34fda-254">For existing property: set value.</span></span>|
| `remove`  | <span data-ttu-id="34fda-255">Rimuove una proprietà o un elemento della matrice.</span><span class="sxs-lookup"><span data-stu-id="34fda-255">Remove a property or array element.</span></span> |
| `replace` | <span data-ttu-id="34fda-256">Uguale a `remove` seguita da `add` nella stessa posizione.</span><span class="sxs-lookup"><span data-stu-id="34fda-256">Same as `remove` followed by `add` at same location.</span></span> |
| `move`    | <span data-ttu-id="34fda-257">Uguale a `remove` dall'origine seguita da `add` alla destinazione usando il valore dell'origine.</span><span class="sxs-lookup"><span data-stu-id="34fda-257">Same as `remove` from source followed by `add` to destination using value from source.</span></span> |
| `copy`    | <span data-ttu-id="34fda-258">Uguale a `add` alla destinazione usando il valore dell'origine.</span><span class="sxs-lookup"><span data-stu-id="34fda-258">Same as `add` to destination using value from source.</span></span> |
| `test`    | <span data-ttu-id="34fda-259">Restituisce il codice di stato di richiesta riuscita se il valore in `path` = `value` fornito.</span><span class="sxs-lookup"><span data-stu-id="34fda-259">Return success status code if value at `path` = provided `value`.</span></span>|

## <a name="jsonpatch-in-aspnet-core"></a><span data-ttu-id="34fda-260">JSON Patch in ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="34fda-260">JsonPatch in ASP.NET Core</span></span>

<span data-ttu-id="34fda-261">L'implementazione ASP.NET Core di JSON Patch viene fornita nel pacchetto NuGet [Microsoft.AspNetCore.JsonPatch](https://www.nuget.org/packages/microsoft.aspnetcore.jsonpatch/).</span><span class="sxs-lookup"><span data-stu-id="34fda-261">The ASP.NET Core implementation of JSON Patch is provided in the [Microsoft.AspNetCore.JsonPatch](https://www.nuget.org/packages/microsoft.aspnetcore.jsonpatch/) NuGet package.</span></span> <span data-ttu-id="34fda-262">Il pacchetto è incluso nel metapacchetto [Microsoft.AspNetCore.App](xref:fundamentals/metapackage-app).</span><span class="sxs-lookup"><span data-stu-id="34fda-262">The package is included in the [Microsoft.AspnetCore.App](xref:fundamentals/metapackage-app) metapackage.</span></span>

## <a name="action-method-code"></a><span data-ttu-id="34fda-263">Codice del metodo di azione</span><span class="sxs-lookup"><span data-stu-id="34fda-263">Action method code</span></span>

<span data-ttu-id="34fda-264">In un controller API un metodo di azione per JSON Patch:</span><span class="sxs-lookup"><span data-stu-id="34fda-264">In an API controller, an action method for JSON Patch:</span></span>

* <span data-ttu-id="34fda-265">Viene annotato con l'attributo `HttpPatch`.</span><span class="sxs-lookup"><span data-stu-id="34fda-265">Is annotated with the `HttpPatch` attribute.</span></span>
* <span data-ttu-id="34fda-266">Accetta un oggetto `JsonPatchDocument<T>` , in genere con `[FromBody]` .</span><span class="sxs-lookup"><span data-stu-id="34fda-266">Accepts a `JsonPatchDocument<T>`, typically with `[FromBody]`.</span></span>
* <span data-ttu-id="34fda-267">Chiama `ApplyTo` nel documento di patch per applicare le modifiche.</span><span class="sxs-lookup"><span data-stu-id="34fda-267">Calls `ApplyTo` on the patch document to apply the changes.</span></span>

<span data-ttu-id="34fda-268">Ecco un esempio:</span><span class="sxs-lookup"><span data-stu-id="34fda-268">Here's an example:</span></span>

[!code-csharp[](jsonpatch/samples/2.2/Controllers/HomeController.cs?name=snippet_PatchAction&highlight=1,3,9)]

<span data-ttu-id="34fda-269">Questo codice dell'app di esempio funziona con il modello `Customer` seguente.</span><span class="sxs-lookup"><span data-stu-id="34fda-269">This code from the sample app works with the following `Customer` model.</span></span>

[!code-csharp[](jsonpatch/samples/2.2/Models/Customer.cs?name=snippet_Customer)]

[!code-csharp[](jsonpatch/samples/2.2/Models/Order.cs?name=snippet_Order)]

<span data-ttu-id="34fda-270">Il metodo di azione di esempio:</span><span class="sxs-lookup"><span data-stu-id="34fda-270">The sample action method:</span></span>

* <span data-ttu-id="34fda-271">Costruisce un oggetto `Customer`.</span><span class="sxs-lookup"><span data-stu-id="34fda-271">Constructs a `Customer`.</span></span>
* <span data-ttu-id="34fda-272">Applica la patch.</span><span class="sxs-lookup"><span data-stu-id="34fda-272">Applies the patch.</span></span>
* <span data-ttu-id="34fda-273">Restituisce il risultato nel corpo della risposta.</span><span class="sxs-lookup"><span data-stu-id="34fda-273">Returns the result in the body of the response.</span></span>

 <span data-ttu-id="34fda-274">In un'app reale il codice recupererebbe i dati da un archivio, ad esempio un database, e aggiornerebbe il database dopo aver applicato la patch.</span><span class="sxs-lookup"><span data-stu-id="34fda-274">In a real app, the code would retrieve the data from a store such as a database and update the database after applying the patch.</span></span>

### <a name="model-state"></a><span data-ttu-id="34fda-275">Stato del modello</span><span class="sxs-lookup"><span data-stu-id="34fda-275">Model state</span></span>

<span data-ttu-id="34fda-276">L'esempio di metodo di azione precedente chiama un overload di `ApplyTo` che accetta lo stato del modello come uno dei propri parametri.</span><span class="sxs-lookup"><span data-stu-id="34fda-276">The preceding action method example calls an overload of `ApplyTo` that takes model state as one of its parameters.</span></span> <span data-ttu-id="34fda-277">Con questa opzione, è possibile ottenere messaggi di errore nelle risposte.</span><span class="sxs-lookup"><span data-stu-id="34fda-277">With this option, you can get error messages in responses.</span></span> <span data-ttu-id="34fda-278">L'esempio seguente mostra il corpo di una risposta 400 Richiesta non valida per un'operazione `test`:</span><span class="sxs-lookup"><span data-stu-id="34fda-278">The following example shows the body of a 400 Bad Request response for a `test` operation:</span></span>

```json
{
    "Customer": [
        "The current value 'John' at path 'customerName' is not equal to the test value 'Nancy'."
    ]
}
```

### <a name="dynamic-objects"></a><span data-ttu-id="34fda-279">Oggetti dinamici</span><span class="sxs-lookup"><span data-stu-id="34fda-279">Dynamic objects</span></span>

<span data-ttu-id="34fda-280">L'esempio di metodo di azione seguente mostra come applicare una patch a un oggetto dinamico.</span><span class="sxs-lookup"><span data-stu-id="34fda-280">The following action method example shows how to apply a patch to a dynamic object.</span></span>

[!code-csharp[](jsonpatch/samples/2.2/Controllers/HomeController.cs?name=snippet_Dynamic)]

## <a name="the-add-operation"></a><span data-ttu-id="34fda-281">Operazione add</span><span class="sxs-lookup"><span data-stu-id="34fda-281">The add operation</span></span>

* <span data-ttu-id="34fda-282">Se `path` punta a un elemento della matrice: inserisce un nuovo elemento prima di quello specificato da `path`.</span><span class="sxs-lookup"><span data-stu-id="34fda-282">If `path` points to an array element: inserts new element before the one specified by `path`.</span></span>
* <span data-ttu-id="34fda-283">Se `path` punta a una proprietà: imposta il valore della proprietà.</span><span class="sxs-lookup"><span data-stu-id="34fda-283">If `path` points to a property: sets the property value.</span></span>
* <span data-ttu-id="34fda-284">Se `path` punta a una posizione che non esiste:</span><span class="sxs-lookup"><span data-stu-id="34fda-284">If `path` points to a nonexistent location:</span></span>
  * <span data-ttu-id="34fda-285">Se la risorsa cui applicare la patch è un oggetto dinamico: aggiunge una proprietà.</span><span class="sxs-lookup"><span data-stu-id="34fda-285">If the resource to patch is a dynamic object: adds a property.</span></span>
  * <span data-ttu-id="34fda-286">Se la risorsa cui applicare la patch è un oggetto statico: la richiesta non riesce.</span><span class="sxs-lookup"><span data-stu-id="34fda-286">If the resource to patch is a static object: the request fails.</span></span>

<span data-ttu-id="34fda-287">Il documento di patch di esempio seguente imposta il valore di `CustomerName` e aggiunge un oggetto `Order` alla fine della matrice `Orders`.</span><span class="sxs-lookup"><span data-stu-id="34fda-287">The following sample patch document sets the value of `CustomerName` and adds an `Order` object to the end of the `Orders` array.</span></span>

[!code-json[](jsonpatch/samples/2.2/JSON/add.json)]

## <a name="the-remove-operation"></a><span data-ttu-id="34fda-288">Operazione remove</span><span class="sxs-lookup"><span data-stu-id="34fda-288">The remove operation</span></span>

* <span data-ttu-id="34fda-289">Se `path` punta a un elemento della matrice: rimuove l'elemento.</span><span class="sxs-lookup"><span data-stu-id="34fda-289">If `path` points to an array element: removes the element.</span></span>
* <span data-ttu-id="34fda-290">Se `path` punta a una proprietà:</span><span class="sxs-lookup"><span data-stu-id="34fda-290">If `path` points to a property:</span></span>
  * <span data-ttu-id="34fda-291">Se la risorsa cui applicare la patch è un oggetto dinamico: rimuove la proprietà.</span><span class="sxs-lookup"><span data-stu-id="34fda-291">If resource to patch is a dynamic object: removes the property.</span></span>
  * <span data-ttu-id="34fda-292">Se la risorsa cui applicare la patch è un oggetto statico:</span><span class="sxs-lookup"><span data-stu-id="34fda-292">If resource to patch is a static object:</span></span>
    * <span data-ttu-id="34fda-293">Se la proprietà ammette valori Null: la imposta su Null.</span><span class="sxs-lookup"><span data-stu-id="34fda-293">If the property is nullable: sets it to null.</span></span>
    * <span data-ttu-id="34fda-294">Se la proprietà non ammette valori Null: la imposta su `default<T>`.</span><span class="sxs-lookup"><span data-stu-id="34fda-294">If the property is non-nullable, sets it to `default<T>`.</span></span>

<span data-ttu-id="34fda-295">Il documento di patch di esempio seguente imposta `CustomerName` su Null ed elimina `Orders[0]`.</span><span class="sxs-lookup"><span data-stu-id="34fda-295">The following sample patch document sets `CustomerName` to null and deletes `Orders[0]`.</span></span>

[!code-json[](jsonpatch/samples/2.2/JSON/remove.json)]

## <a name="the-replace-operation"></a><span data-ttu-id="34fda-296">Operazione replace</span><span class="sxs-lookup"><span data-stu-id="34fda-296">The replace operation</span></span>

<span data-ttu-id="34fda-297">Questa operazione equivale a livello funzionale all'operazione `remove` seguita da un'operazione `add`.</span><span class="sxs-lookup"><span data-stu-id="34fda-297">This operation is functionally the same as a `remove` followed by an `add`.</span></span>

<span data-ttu-id="34fda-298">Il documento di patch di esempio seguente imposta il valore di `CustomerName` e sostituisce `Orders[0]` con un nuovo oggetto `Order`.</span><span class="sxs-lookup"><span data-stu-id="34fda-298">The following sample patch document sets the value of `CustomerName` and replaces `Orders[0]`with a new `Order` object.</span></span>

[!code-json[](jsonpatch/samples/2.2/JSON/replace.json)]

## <a name="the-move-operation"></a><span data-ttu-id="34fda-299">Operazione move</span><span class="sxs-lookup"><span data-stu-id="34fda-299">The move operation</span></span>

* <span data-ttu-id="34fda-300">Se `path` punta a un elemento della matrice: copia l'elemento `from` nella posizione dell'elemento `path`, quindi esegue un'operazione `remove` sull'elemento `from`.</span><span class="sxs-lookup"><span data-stu-id="34fda-300">If `path` points to an array element: copies `from` element to location of `path` element, then runs a `remove` operation on the `from` element.</span></span>
* <span data-ttu-id="34fda-301">Se `path` punta a una proprietà: copia il valore della proprietà `from` nella proprietà `path`, quindi esegue un'operazione `remove` sulla proprietà `from`.</span><span class="sxs-lookup"><span data-stu-id="34fda-301">If `path` points to a property: copies value of `from` property to `path` property, then runs a `remove` operation on the `from` property.</span></span>
* <span data-ttu-id="34fda-302">Se `path` punta a una proprietà che non esiste:</span><span class="sxs-lookup"><span data-stu-id="34fda-302">If `path` points to a nonexistent property:</span></span>
  * <span data-ttu-id="34fda-303">Se la risorsa cui applicare la patch è un oggetto statico: la richiesta non riesce.</span><span class="sxs-lookup"><span data-stu-id="34fda-303">If the resource to patch is a static object: the request fails.</span></span>
  * <span data-ttu-id="34fda-304">Se la risorsa cui applicare la patch è un oggetto dinamico: copia la proprietà `from` nella posizione indicata da `path`, quindi esegue un'operazione `remove` sulla proprietà `from`.</span><span class="sxs-lookup"><span data-stu-id="34fda-304">If the resource to patch is a dynamic object: copies `from` property to location indicated by `path`, then runs a `remove` operation on the `from` property.</span></span>

<span data-ttu-id="34fda-305">Il documento di patch di esempio seguente:</span><span class="sxs-lookup"><span data-stu-id="34fda-305">The following sample patch document:</span></span>

* <span data-ttu-id="34fda-306">Copia il valore di `Orders[0].OrderName` in `CustomerName`.</span><span class="sxs-lookup"><span data-stu-id="34fda-306">Copies the value of `Orders[0].OrderName` to `CustomerName`.</span></span>
* <span data-ttu-id="34fda-307">Imposta `Orders[0].OrderName` su Null.</span><span class="sxs-lookup"><span data-stu-id="34fda-307">Sets `Orders[0].OrderName` to null.</span></span>
* <span data-ttu-id="34fda-308">Sposta `Orders[1]` prima di `Orders[0]`.</span><span class="sxs-lookup"><span data-stu-id="34fda-308">Moves `Orders[1]` to before `Orders[0]`.</span></span>

[!code-json[](jsonpatch/samples/2.2/JSON/move.json)]

## <a name="the-copy-operation"></a><span data-ttu-id="34fda-309">Operazione copy</span><span class="sxs-lookup"><span data-stu-id="34fda-309">The copy operation</span></span>

<span data-ttu-id="34fda-310">Questa operazione equivale a livello funzionale all'operazione `move` senza il passaggio `remove` finale.</span><span class="sxs-lookup"><span data-stu-id="34fda-310">This operation is functionally the same as a `move` operation without the final `remove` step.</span></span>

<span data-ttu-id="34fda-311">Il documento di patch di esempio seguente:</span><span class="sxs-lookup"><span data-stu-id="34fda-311">The following sample patch document:</span></span>

* <span data-ttu-id="34fda-312">Copia il valore di `Orders[0].OrderName` in `CustomerName`.</span><span class="sxs-lookup"><span data-stu-id="34fda-312">Copies the value of `Orders[0].OrderName` to `CustomerName`.</span></span>
* <span data-ttu-id="34fda-313">Inserisce una copia di `Orders[1]` prima di `Orders[0]`.</span><span class="sxs-lookup"><span data-stu-id="34fda-313">Inserts a copy of `Orders[1]` before `Orders[0]`.</span></span>

[!code-json[](jsonpatch/samples/2.2/JSON/copy.json)]

## <a name="the-test-operation"></a><span data-ttu-id="34fda-314">Operazione test</span><span class="sxs-lookup"><span data-stu-id="34fda-314">The test operation</span></span>

<span data-ttu-id="34fda-315">Se il valore nella posizione indicata da `path` è diverso dal valore fornito in `value`, la richiesta non riesce.</span><span class="sxs-lookup"><span data-stu-id="34fda-315">If the value at the location indicated by `path` is different from the value provided in `value`, the request fails.</span></span> <span data-ttu-id="34fda-316">In questo caso, l'intera richiesta PATCH non riesce anche se tutte le altre operazioni nel documento di patch potrebbero riuscire.</span><span class="sxs-lookup"><span data-stu-id="34fda-316">In that case, the whole PATCH request fails even if all other operations in the patch document would otherwise succeed.</span></span>

<span data-ttu-id="34fda-317">L'operazione `test` viene usata in genere per impedire un aggiornamento in caso di conflitto di concorrenza.</span><span class="sxs-lookup"><span data-stu-id="34fda-317">The `test` operation is commonly used to prevent an update when there's a concurrency conflict.</span></span>

<span data-ttu-id="34fda-318">Il documento di patch di esempio seguente non ha alcun effetto se il valore iniziale di `CustomerName` è "John", perché il test non riesce:</span><span class="sxs-lookup"><span data-stu-id="34fda-318">The following sample patch document has no effect if the initial value of `CustomerName` is "John", because the test fails:</span></span>

[!code-json[](jsonpatch/samples/2.2/JSON/test-fail.json)]

## <a name="get-the-code"></a><span data-ttu-id="34fda-319">Ottenere il codice</span><span class="sxs-lookup"><span data-stu-id="34fda-319">Get the code</span></span>

<span data-ttu-id="34fda-320">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/web-api/jsonpatch/samples/2.2).</span><span class="sxs-lookup"><span data-stu-id="34fda-320">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/web-api/jsonpatch/samples/2.2).</span></span> <span data-ttu-id="34fda-321">([Come scaricare](xref:index#how-to-download-a-sample)).</span><span class="sxs-lookup"><span data-stu-id="34fda-321">([How to download](xref:index#how-to-download-a-sample)).</span></span>

<span data-ttu-id="34fda-322">Per testare l'esempio, eseguire l'app e inviare richieste HTTP con le impostazioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="34fda-322">To test the sample, run the app and send HTTP requests with the following settings:</span></span>

* <span data-ttu-id="34fda-323">URL: `http://localhost:{port}/jsonpatch/jsonpatchwithmodelstate`</span><span class="sxs-lookup"><span data-stu-id="34fda-323">URL: `http://localhost:{port}/jsonpatch/jsonpatchwithmodelstate`</span></span>
* <span data-ttu-id="34fda-324">Metodo HTTP: `PATCH`</span><span class="sxs-lookup"><span data-stu-id="34fda-324">HTTP method: `PATCH`</span></span>
* <span data-ttu-id="34fda-325">Intestazione: `Content-Type: application/json-patch+json`</span><span class="sxs-lookup"><span data-stu-id="34fda-325">Header: `Content-Type: application/json-patch+json`</span></span>
* <span data-ttu-id="34fda-326">Body: copiare e incollare uno degli esempi di documenti patch JSON dalla cartella del progetto *JSON* .</span><span class="sxs-lookup"><span data-stu-id="34fda-326">Body: Copy and paste one of the JSON patch document samples from the *JSON* project folder.</span></span>

## <a name="additional-resources"></a><span data-ttu-id="34fda-327">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="34fda-327">Additional resources</span></span>

* [<span data-ttu-id="34fda-328">Specifica del metodo PATCH IETF RFC 5789</span><span class="sxs-lookup"><span data-stu-id="34fda-328">IETF RFC 5789 PATCH method specification</span></span>](https://tools.ietf.org/html/rfc5789)
* [<span data-ttu-id="34fda-329">Specifica JSON Patch IETF RFC 6902</span><span class="sxs-lookup"><span data-stu-id="34fda-329">IETF RFC 6902 JSON Patch specification</span></span>](https://tools.ietf.org/html/rfc6902)
* [<span data-ttu-id="34fda-330">Specifica del formato di percorso JSON Patch IETF RFC 6901</span><span class="sxs-lookup"><span data-stu-id="34fda-330">IETF RFC 6901 JSON Patch path format spec</span></span>](https://tools.ietf.org/html/rfc6901)
* <span data-ttu-id="34fda-331">[Documentazione di JSON Patch](https://jsonpatch.com/).</span><span class="sxs-lookup"><span data-stu-id="34fda-331">[JSON Patch documentation](https://jsonpatch.com/).</span></span> <span data-ttu-id="34fda-332">Include collegamenti a risorse per la creazione di documenti JSON Patch.</span><span class="sxs-lookup"><span data-stu-id="34fda-332">Includes links to resources for creating JSON Patch documents.</span></span>
* [<span data-ttu-id="34fda-333">Codice sorgente JSON Patch in ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="34fda-333">ASP.NET Core JSON Patch source code</span></span>](https://github.com/dotnet/AspNetCore/tree/main/src/Features/JsonPatch/src)

::: moniker-end
