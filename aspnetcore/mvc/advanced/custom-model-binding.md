---
title: Associazione di modelli personalizzata in ASP.NET Core
author: ardalis
description: Informazioni su come l'associazione di modelli consente alle azioni del controller di funzionare direttamente con i tipi di modello in ASP.NET Core.
ms.author: riande
ms.date: 01/06/2020
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
uid: mvc/advanced/custom-model-binding
ms.openlocfilehash: b674d3a034124a26e2e76f2a7f0220e3f0ecea56
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102588770"
---
# <a name="custom-model-binding-in-aspnet-core"></a><span data-ttu-id="ea49c-103">Associazione di modelli personalizzata in ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="ea49c-103">Custom Model Binding in ASP.NET Core</span></span>

::: moniker range=">= aspnetcore-3.0"

<span data-ttu-id="ea49c-104">Di [Steve Smith](https://ardalis.com/) e [Kirk Larkin](https://twitter.com/serpent5)</span><span class="sxs-lookup"><span data-stu-id="ea49c-104">By [Steve Smith](https://ardalis.com/) and [Kirk Larkin](https://twitter.com/serpent5)</span></span>

<span data-ttu-id="ea49c-105">Mediante l'associazione di modelli le azioni dei controller possono operare direttamente con i tipi di modello (passati come argomenti dei metodi), anziché con le richieste HTTP.</span><span class="sxs-lookup"><span data-stu-id="ea49c-105">Model binding allows controller actions to work directly with model types (passed in as method arguments), rather than HTTP requests.</span></span> <span data-ttu-id="ea49c-106">Il mapping tra i dati delle richieste in ingresso e i modelli applicativi è gestito dagli strumenti di associazione di modelli.</span><span class="sxs-lookup"><span data-stu-id="ea49c-106">Mapping between incoming request data and application models is handled by model binders.</span></span> <span data-ttu-id="ea49c-107">Gli sviluppatori possono estendere la funzionalità di associazione di modelli predefinita implementando strumenti di associazione di modelli personalizzati (anche se in genere non è necessario creare un provider personalizzato).</span><span class="sxs-lookup"><span data-stu-id="ea49c-107">Developers can extend the built-in model binding functionality by implementing custom model binders (though typically, you don't need to write your own provider).</span></span>

<span data-ttu-id="ea49c-108">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/mvc/advanced/custom-model-binding/samples) ([procedura per il download](xref:index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="ea49c-108">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/mvc/advanced/custom-model-binding/samples) ([how to download](xref:index#how-to-download-a-sample))</span></span>

## <a name="default-model-binder-limitations"></a><span data-ttu-id="ea49c-109">Limiti degli strumenti di associazione di modelli predefiniti</span><span class="sxs-lookup"><span data-stu-id="ea49c-109">Default model binder limitations</span></span>

<span data-ttu-id="ea49c-110">Gli strumenti di associazione di modelli predefiniti supportano la maggior parte dei tipi di dati .NET Core comuni e soddisfano le esigenze della maggior parte degli sviluppatori.</span><span class="sxs-lookup"><span data-stu-id="ea49c-110">The default model binders support most of the common .NET Core data types and should meet most developers' needs.</span></span> <span data-ttu-id="ea49c-111">Tali strumenti associano direttamente l'input di testo della richiesta ai tipi di modello.</span><span class="sxs-lookup"><span data-stu-id="ea49c-111">They expect to bind text-based input from the request directly to model types.</span></span> <span data-ttu-id="ea49c-112">Può essere necessario trasformare l'input prima dell'associazione.</span><span class="sxs-lookup"><span data-stu-id="ea49c-112">You might need to transform the input prior to binding it.</span></span> <span data-ttu-id="ea49c-113">Un esempio è il caso in cui è presente una chiave che può essere usata per cercare i dati del modello.</span><span class="sxs-lookup"><span data-stu-id="ea49c-113">For example, when you have a key that can be used to look up model data.</span></span> <span data-ttu-id="ea49c-114">È possibile usare uno strumento di associazione di modelli personalizzato per il recupero di dati in base alla chiave.</span><span class="sxs-lookup"><span data-stu-id="ea49c-114">You can use a custom model binder to fetch data based on the key.</span></span>

## <a name="model-binding-review"></a><span data-ttu-id="ea49c-115">Analisi dell'associazione di modelli</span><span class="sxs-lookup"><span data-stu-id="ea49c-115">Model binding review</span></span>

<span data-ttu-id="ea49c-116">L'associazione di modelli usa definizioni specifiche per i tipi sui quali opera.</span><span class="sxs-lookup"><span data-stu-id="ea49c-116">Model binding uses specific definitions for the types it operates on.</span></span> <span data-ttu-id="ea49c-117">Un *tipo semplice* viene convertito da una stringa singola dell'input.</span><span class="sxs-lookup"><span data-stu-id="ea49c-117">A *simple type* is converted from a single string in the input.</span></span> <span data-ttu-id="ea49c-118">Un *tipo complesso* viene convertito da più valori di input.</span><span class="sxs-lookup"><span data-stu-id="ea49c-118">A *complex type* is converted from multiple input values.</span></span> <span data-ttu-id="ea49c-119">Il framework determina la differenza in base all'esistenza di un elemento `TypeConverter`.</span><span class="sxs-lookup"><span data-stu-id="ea49c-119">The framework determines the difference based on the existence of a `TypeConverter`.</span></span> <span data-ttu-id="ea49c-120">È consigliabile creare un convertitore di tipi se è presente un mapping `string` -> `SomeType` semplice che non richiede risorse esterne.</span><span class="sxs-lookup"><span data-stu-id="ea49c-120">We recommended you create a type converter if you have a simple `string` -> `SomeType` mapping that doesn't require external resources.</span></span>

<span data-ttu-id="ea49c-121">Prima di creare uno strumento di associazione di modelli personalizzato, è utile esaminare le modalità di implementazione degli strumenti di associazione di modelli esistenti.</span><span class="sxs-lookup"><span data-stu-id="ea49c-121">Before creating your own custom model binder, it's worth reviewing how existing model binders are implemented.</span></span> <span data-ttu-id="ea49c-122">Si consideri l'oggetto <xref:Microsoft.AspNetCore.Mvc.ModelBinding.Binders.ByteArrayModelBinder> che può essere usato per convertire le stringhe con codifica Base64 in matrici di byte.</span><span class="sxs-lookup"><span data-stu-id="ea49c-122">Consider the <xref:Microsoft.AspNetCore.Mvc.ModelBinding.Binders.ByteArrayModelBinder> which can be used to convert base64-encoded strings into byte arrays.</span></span> <span data-ttu-id="ea49c-123">Le matrici di byte vengono spesso archiviate come file o campi BLOB del database.</span><span class="sxs-lookup"><span data-stu-id="ea49c-123">The byte arrays are often stored as files or database BLOB fields.</span></span>

### <a name="working-with-the-bytearraymodelbinder"></a><span data-ttu-id="ea49c-124">Uso di ByteArrayModelBinder</span><span class="sxs-lookup"><span data-stu-id="ea49c-124">Working with the ByteArrayModelBinder</span></span>

<span data-ttu-id="ea49c-125">Le stringhe con codifica Base64 possono essere usate per rappresentare i dati binari.</span><span class="sxs-lookup"><span data-stu-id="ea49c-125">Base64-encoded strings can be used to represent binary data.</span></span> <span data-ttu-id="ea49c-126">Un'immagine, ad esempio, può essere codificata come stringa.</span><span class="sxs-lookup"><span data-stu-id="ea49c-126">For example, an image can be encoded as a string.</span></span> <span data-ttu-id="ea49c-127">Nell'esempio è inclusa un'immagine come stringa con codifica Base64 in [Base64String.txt](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/mvc/advanced/custom-model-binding/samples/3.x/CustomModelBindingSample/Base64String.txt).</span><span class="sxs-lookup"><span data-stu-id="ea49c-127">The sample includes an image as a base64-encoded string in [Base64String.txt](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/mvc/advanced/custom-model-binding/samples/3.x/CustomModelBindingSample/Base64String.txt).</span></span>

<span data-ttu-id="ea49c-128">ASP.NET Core MVC può accettare una stringa con codifica base64 e usare un oggetto `ByteArrayModelBinder` per convertirla in una matrice di byte.</span><span class="sxs-lookup"><span data-stu-id="ea49c-128">ASP.NET Core MVC can take a base64-encoded string and use a `ByteArrayModelBinder` to convert it into a byte array.</span></span> <span data-ttu-id="ea49c-129"><xref:Microsoft.AspNetCore.Mvc.ModelBinding.Binders.ByteArrayModelBinderProvider>Esegue il mapping degli `byte[]` argomenti a `ByteArrayModelBinder` :</span><span class="sxs-lookup"><span data-stu-id="ea49c-129">The <xref:Microsoft.AspNetCore.Mvc.ModelBinding.Binders.ByteArrayModelBinderProvider> maps `byte[]` arguments to `ByteArrayModelBinder`:</span></span>

```csharp
public IModelBinder GetBinder(ModelBinderProviderContext context)
{
    if (context == null)
    {
        throw new ArgumentNullException(nameof(context));
    }

    if (context.Metadata.ModelType == typeof(byte[]))
    {
        var loggerFactory = context.Services.GetRequiredService<ILoggerFactory>();
        return new ByteArrayModelBinder(loggerFactory);
    }

    return null;
}
```

<span data-ttu-id="ea49c-130">Quando si crea un gestore di associazione di modelli personalizzato, è possibile implementare un proprio `IModelBinderProvider` tipo o usare <xref:Microsoft.AspNetCore.Mvc.ModelBinderAttribute> .</span><span class="sxs-lookup"><span data-stu-id="ea49c-130">When creating your own custom model binder, you can implement your own `IModelBinderProvider` type, or use the <xref:Microsoft.AspNetCore.Mvc.ModelBinderAttribute>.</span></span>

<span data-ttu-id="ea49c-131">L'esempio indica come usare `ByteArrayModelBinder` per convertire una stringa con codifica base64 in un `byte[]` e salvare il risultato in un file:</span><span class="sxs-lookup"><span data-stu-id="ea49c-131">The following example shows how to use `ByteArrayModelBinder` to convert a base64-encoded string to a `byte[]` and save the result to a file:</span></span>

[!code-csharp[](custom-model-binding/samples/3.x/CustomModelBindingSample/Controllers/ImageController.cs?name=snippet_Post)]
[!INCLUDE[about the series](~/includes/code-comments-loc.md)]

<span data-ttu-id="ea49c-132">È possibile pubblicare (POST) una stringa con codifica base64 in questo metodo API usando uno strumento come [Postman](https://www.getpostman.com/):</span><span class="sxs-lookup"><span data-stu-id="ea49c-132">You can POST a base64-encoded string to this api method using a tool like [Postman](https://www.getpostman.com/):</span></span>

<span data-ttu-id="ea49c-133">![Postman](custom-model-binding/images/postman.png "Postman")</span><span class="sxs-lookup"><span data-stu-id="ea49c-133">![postman](custom-model-binding/images/postman.png "postman")</span></span>

<span data-ttu-id="ea49c-134">Se lo strumento di associazione riesce ad associare i dati della richiesta a proprietà o argomenti denominati in modo appropriato, l'associazione di modelli ha esito positivo.</span><span class="sxs-lookup"><span data-stu-id="ea49c-134">As long as the binder can bind request data to appropriately named properties or arguments, model binding will succeed.</span></span> <span data-ttu-id="ea49c-135">L'esempio seguente indica come usare `ByteArrayModelBinder` con un modello di visualizzazione:</span><span class="sxs-lookup"><span data-stu-id="ea49c-135">The following example shows how to use `ByteArrayModelBinder` with a view model:</span></span>

[!code-csharp[](custom-model-binding/samples/3.x/CustomModelBindingSample/Controllers/ImageController.cs?name=snippet_SaveProfile&highlight=2)]

## <a name="custom-model-binder-sample"></a><span data-ttu-id="ea49c-136">Esempio di strumento di associazione di modelli personalizzato</span><span class="sxs-lookup"><span data-stu-id="ea49c-136">Custom model binder sample</span></span>

<span data-ttu-id="ea49c-137">In questa sezione viene implementato uno strumento di associazione di modelli personalizzato che:</span><span class="sxs-lookup"><span data-stu-id="ea49c-137">In this section we'll implement a custom model binder that:</span></span>

- <span data-ttu-id="ea49c-138">Converte i dati della richiesta in ingresso in argomenti chiave fortemente tipizzati.</span><span class="sxs-lookup"><span data-stu-id="ea49c-138">Converts incoming request data into strongly typed key arguments.</span></span>
- <span data-ttu-id="ea49c-139">Usa Entity Framework Core per recuperare l'entità associata.</span><span class="sxs-lookup"><span data-stu-id="ea49c-139">Uses Entity Framework Core to fetch the associated entity.</span></span>
- <span data-ttu-id="ea49c-140">Passa l'entità associata come argomento al metodo di azione.</span><span class="sxs-lookup"><span data-stu-id="ea49c-140">Passes the associated entity as an argument to the action method.</span></span>

<span data-ttu-id="ea49c-141">L'esempio seguente usa l'attributo `ModelBinder` nel modello `Author`:</span><span class="sxs-lookup"><span data-stu-id="ea49c-141">The following sample uses the `ModelBinder` attribute on the `Author` model:</span></span>

[!code-csharp[](custom-model-binding/samples/3.x/CustomModelBindingSample/Data/Author.cs?highlight=6)]

<span data-ttu-id="ea49c-142">Nel codice precedente l'attributo `ModelBinder` specifica il tipo di `IModelBinder` da usare per associare i parametri di azione `Author`.</span><span class="sxs-lookup"><span data-stu-id="ea49c-142">In the preceding code, the `ModelBinder` attribute specifies the type of `IModelBinder` that should be used to bind `Author` action parameters.</span></span>

<span data-ttu-id="ea49c-143">La classe `AuthorEntityBinder` seguente associa un parametro `Author` recuperando l'entità da un'origine dati tramite Entity Framework Core e un `authorId`:</span><span class="sxs-lookup"><span data-stu-id="ea49c-143">The following `AuthorEntityBinder` class binds an `Author` parameter by fetching the entity from a data source using Entity Framework Core and an `authorId`:</span></span>

[!code-csharp[](custom-model-binding/samples/3.x/CustomModelBindingSample/Binders/AuthorEntityBinder.cs?name=snippet_Class)]

> [!NOTE]
> <span data-ttu-id="ea49c-144">La classe `AuthorEntityBinder` precedente è destinata a illustrare uno strumento di associazione di modelli personalizzato.</span><span class="sxs-lookup"><span data-stu-id="ea49c-144">The preceding `AuthorEntityBinder` class is intended to illustrate a custom model binder.</span></span> <span data-ttu-id="ea49c-145">La classe non ha lo scopo di illustrare procedure consigliate per uno scenario di ricerca.</span><span class="sxs-lookup"><span data-stu-id="ea49c-145">The class isn't intended to illustrate best practices for a lookup scenario.</span></span> <span data-ttu-id="ea49c-146">Per la ricerca, associare l'elemento `authorId` ed eseguire query nel database in un metodo di azione.</span><span class="sxs-lookup"><span data-stu-id="ea49c-146">For lookup, bind the `authorId` and query the database in an action method.</span></span> <span data-ttu-id="ea49c-147">Questo approccio separa gli errori di associazione di modelli dai casi `NotFound`.</span><span class="sxs-lookup"><span data-stu-id="ea49c-147">This approach separates model binding failures from `NotFound` cases.</span></span>

<span data-ttu-id="ea49c-148">L'esempio di codice seguente indica come usare `AuthorEntityBinder` in un metodo di azione:</span><span class="sxs-lookup"><span data-stu-id="ea49c-148">The following code shows how to use the `AuthorEntityBinder` in an action method:</span></span>

[!code-csharp[](custom-model-binding/samples/3.x/CustomModelBindingSample/Controllers/BoundAuthorsController.cs?name=snippet_Get&highlight=2)]

<span data-ttu-id="ea49c-149">L'attributo `ModelBinder` può essere usato per applicare gli elementi `AuthorEntityBinder` ai parametri che non usano convenzioni predefinite:</span><span class="sxs-lookup"><span data-stu-id="ea49c-149">The `ModelBinder` attribute can be used to apply the `AuthorEntityBinder` to parameters that don't use default conventions:</span></span>

[!code-csharp[](custom-model-binding/samples/3.x/CustomModelBindingSample/Controllers/BoundAuthorsController.cs?name=snippet_GetById&highlight=2)]

<span data-ttu-id="ea49c-150">In questo esempio, dato che il nome dell'argomento non è il valore `authorId` predefinito, viene specificato nel parametro usando l'attributo `ModelBinder`.</span><span class="sxs-lookup"><span data-stu-id="ea49c-150">In this example, since the name of the argument isn't the default `authorId`, it's specified on the parameter using the `ModelBinder` attribute.</span></span> <span data-ttu-id="ea49c-151">Il controller e il metodo di azione sono semplificati rispetto alla ricerca dell'entità nel metodo di azione.</span><span class="sxs-lookup"><span data-stu-id="ea49c-151">Both the controller and action method are simplified compared to looking up the entity in the action method.</span></span> <span data-ttu-id="ea49c-152">La logica per recuperare l'autore usando Entity Framework Core viene spostata nello strumento di associazione di modelli.</span><span class="sxs-lookup"><span data-stu-id="ea49c-152">The logic to fetch the author using Entity Framework Core is moved to the model binder.</span></span> <span data-ttu-id="ea49c-153">Ciò può rappresentare una semplificazione notevole in presenza di vari metodi associati al modello `Author`.</span><span class="sxs-lookup"><span data-stu-id="ea49c-153">This can be a considerable simplification when you have several methods that bind to the `Author` model.</span></span>

<span data-ttu-id="ea49c-154">È possibile applicare l'attributo `ModelBinder` a singole proprietà del modello (ad esempio a ViewModel) o a parametri del metodo di azione per specificare un determinato strumento di associazione di modelli o nome di modello solo per quel determinato tipo o azione.</span><span class="sxs-lookup"><span data-stu-id="ea49c-154">You can apply the `ModelBinder` attribute to individual model properties (such as on a viewmodel) or to action method parameters to specify a certain model binder or model name for just that type or action.</span></span>

### <a name="implementing-a-modelbinderprovider"></a><span data-ttu-id="ea49c-155">Implementazione di un elemento ModelBinderProvider</span><span class="sxs-lookup"><span data-stu-id="ea49c-155">Implementing a ModelBinderProvider</span></span>

<span data-ttu-id="ea49c-156">Anziché applicare un attributo, è possibile implementare `IModelBinderProvider`.</span><span class="sxs-lookup"><span data-stu-id="ea49c-156">Instead of applying an attribute, you can implement `IModelBinderProvider`.</span></span> <span data-ttu-id="ea49c-157">Gli strumenti di associazione del framework incorporati vengono implementati in questo modo.</span><span class="sxs-lookup"><span data-stu-id="ea49c-157">This is how the built-in framework binders are implemented.</span></span> <span data-ttu-id="ea49c-158">Quando si specifica il tipo sul quale opera lo strumento di associazione, si indica il tipo di argomento prodotto dallo strumento, **non** l'input accettato dallo strumento.</span><span class="sxs-lookup"><span data-stu-id="ea49c-158">When you specify the type your binder operates on, you specify the type of argument it produces, **not** the input your binder accepts.</span></span> <span data-ttu-id="ea49c-159">Il provider strumento di associazione seguente funziona con l'elemento `AuthorEntityBinder`.</span><span class="sxs-lookup"><span data-stu-id="ea49c-159">The following binder provider works with the `AuthorEntityBinder`.</span></span> <span data-ttu-id="ea49c-160">Quando viene aggiunto alla raccolta di provider di MVC, non è necessario usare l'attributo `ModelBinder` sui parametri tipizzati `Author` o `Author`.</span><span class="sxs-lookup"><span data-stu-id="ea49c-160">When it's added to MVC's collection of providers, you don't need to use the `ModelBinder` attribute on `Author` or `Author`-typed parameters.</span></span>

[!code-csharp[](custom-model-binding/samples/3.x/CustomModelBindingSample/Binders/AuthorEntityBinderProvider.cs?highlight=17-20)]

> <span data-ttu-id="ea49c-161">Nota: il codice precedente restituisce un elemento `BinderTypeModelBinder`.</span><span class="sxs-lookup"><span data-stu-id="ea49c-161">Note: The preceding code returns a `BinderTypeModelBinder`.</span></span> <span data-ttu-id="ea49c-162">`BinderTypeModelBinder` funziona come factory per gli strumenti di associazione di modelli e implementa la funzionalità DI (Dependency Injection, Inserimento di dipendenze).</span><span class="sxs-lookup"><span data-stu-id="ea49c-162">`BinderTypeModelBinder` acts as a factory for model binders and provides dependency injection (DI).</span></span> <span data-ttu-id="ea49c-163">`AuthorEntityBinder` richiede DI (Dependency Injection, Inserimento di dipendenze) per l'accesso a EF Core.</span><span class="sxs-lookup"><span data-stu-id="ea49c-163">The `AuthorEntityBinder` requires DI to access EF Core.</span></span> <span data-ttu-id="ea49c-164">Usare `BinderTypeModelBinder` se lo strumento di associazione di modelli richiede servizi da DI (Dependency Injection, Inserimento di dipendenze).</span><span class="sxs-lookup"><span data-stu-id="ea49c-164">Use `BinderTypeModelBinder` if your model binder requires services from DI.</span></span>

<span data-ttu-id="ea49c-165">Per usare un provider dello strumento di associazione di modelli personalizzato, aggiungerlo in `ConfigureServices`:</span><span class="sxs-lookup"><span data-stu-id="ea49c-165">To use a custom model binder provider, add it in `ConfigureServices`:</span></span>

[!code-csharp[](custom-model-binding/samples/3.x/CustomModelBindingSample/Startup.cs?name=snippet_ConfigureServices&highlight=5-8)]

<span data-ttu-id="ea49c-166">Durante la valutazione degli strumenti di associazione di modelli, la raccolta di provider viene esaminata in ordine.</span><span class="sxs-lookup"><span data-stu-id="ea49c-166">When evaluating model binders, the collection of providers is examined in order.</span></span> <span data-ttu-id="ea49c-167">Viene utilizzato il primo provider che restituisce uno strumento di associazione che corrisponde al modello di input.</span><span class="sxs-lookup"><span data-stu-id="ea49c-167">The first provider that returns a binder that matches the input model is used.</span></span> <span data-ttu-id="ea49c-168">L'aggiunta del provider alla fine della raccolta può pertanto comportare la chiamata di uno strumento di associazione di modelli predefinito prima che il Binder personalizzato abbia la possibilità.</span><span class="sxs-lookup"><span data-stu-id="ea49c-168">Adding your provider to the end of the collection may thus result in a built-in model binder being called before your custom binder has a chance.</span></span> <span data-ttu-id="ea49c-169">In questo esempio, il provider personalizzato viene aggiunto all'inizio della raccolta per assicurarsi che venga sempre usato per gli `Author` argomenti dell'azione.</span><span class="sxs-lookup"><span data-stu-id="ea49c-169">In this example, the custom provider is added to the beginning of the collection to ensure it's always used for `Author` action arguments.</span></span>

### <a name="polymorphic-model-binding"></a><span data-ttu-id="ea49c-170">Associazione di modelli polimorfici</span><span class="sxs-lookup"><span data-stu-id="ea49c-170">Polymorphic model binding</span></span>

<span data-ttu-id="ea49c-171">L'associazione a modelli diversi di tipi derivati è nota come associazione di modelli polimorfici.</span><span class="sxs-lookup"><span data-stu-id="ea49c-171">Binding to different models of derived types is known as polymorphic model binding.</span></span> <span data-ttu-id="ea49c-172">L'associazione di modelli personalizzati polimorfici è obbligatoria quando il valore della richiesta deve essere associato al tipo di modello derivato specifico.</span><span class="sxs-lookup"><span data-stu-id="ea49c-172">Polymorphic custom model binding is required when the request value must be bound to the specific derived model type.</span></span> <span data-ttu-id="ea49c-173">Associazione di modelli polimorfici:</span><span class="sxs-lookup"><span data-stu-id="ea49c-173">Polymorphic model binding:</span></span>

* <span data-ttu-id="ea49c-174">Non è tipico per un'API REST progettata per interagire con tutti i linguaggi.</span><span class="sxs-lookup"><span data-stu-id="ea49c-174">Isn't typical for a REST API that's designed to interoperate with all languages.</span></span>
* <span data-ttu-id="ea49c-175">Rende difficile la motivazione dei modelli associati.</span><span class="sxs-lookup"><span data-stu-id="ea49c-175">Makes it difficult to reason about the bound models.</span></span>

<span data-ttu-id="ea49c-176">Tuttavia, se un'app richiede l'associazione di modelli polimorfici, un'implementazione potrebbe avere un aspetto simile al codice seguente:</span><span class="sxs-lookup"><span data-stu-id="ea49c-176">However, if an app requires polymorphic model binding, an implementation might look like the following code:</span></span>

[!code-csharp[](custom-model-binding/samples/3.x/PolymorphicModelBindingSample/ModelBinders/PolymorphicModelBinder.cs?name=snippet)]

## <a name="recommendations-and-best-practices"></a><span data-ttu-id="ea49c-177">Suggerimenti e procedure ottimali</span><span class="sxs-lookup"><span data-stu-id="ea49c-177">Recommendations and best practices</span></span>

<span data-ttu-id="ea49c-178">Gli strumenti di associazione di modelli personalizzati:</span><span class="sxs-lookup"><span data-stu-id="ea49c-178">Custom model binders:</span></span>

- <span data-ttu-id="ea49c-179">Non devono provare a impostare codici di stato o restituire risultati (ad esempio 404 Non trovato).</span><span class="sxs-lookup"><span data-stu-id="ea49c-179">Shouldn't attempt to set status codes or return results (for example, 404 Not Found).</span></span> <span data-ttu-id="ea49c-180">Se si verifica un errore nell'associazione di modelli, l'errore deve essere gestito da un [filtro azioni](xref:mvc/controllers/filters) o da logica inclusa nel metodo di azione.</span><span class="sxs-lookup"><span data-stu-id="ea49c-180">If model binding fails, an [action filter](xref:mvc/controllers/filters) or logic within the action method itself should handle the failure.</span></span>
- <span data-ttu-id="ea49c-181">Sono particolarmente utili per eliminare codice ripetitivo e problemi di montaggio incrociato dai metodi di azione.</span><span class="sxs-lookup"><span data-stu-id="ea49c-181">Are most useful for eliminating repetitive code and cross-cutting concerns from action methods.</span></span>
- <span data-ttu-id="ea49c-182">In genere non devono essere usati per convertire una stringa in un tipo personalizzato. Un elemento <xref:System.ComponentModel.TypeConverter> rappresenta solitamente una scelta migliore.</span><span class="sxs-lookup"><span data-stu-id="ea49c-182">Typically shouldn't be used to convert a string into a custom type, a <xref:System.ComponentModel.TypeConverter> is usually a better option.</span></span>

::: moniker-end
::: moniker range="< aspnetcore-3.0"

<span data-ttu-id="ea49c-183">Di [Steve Smith](https://ardalis.com/)</span><span class="sxs-lookup"><span data-stu-id="ea49c-183">By [Steve Smith](https://ardalis.com/)</span></span>

<span data-ttu-id="ea49c-184">Mediante l'associazione di modelli le azioni dei controller possono operare direttamente con i tipi di modello (passati come argomenti dei metodi), anziché con le richieste HTTP.</span><span class="sxs-lookup"><span data-stu-id="ea49c-184">Model binding allows controller actions to work directly with model types (passed in as method arguments), rather than HTTP requests.</span></span> <span data-ttu-id="ea49c-185">Il mapping tra i dati delle richieste in ingresso e i modelli applicativi è gestito dagli strumenti di associazione di modelli.</span><span class="sxs-lookup"><span data-stu-id="ea49c-185">Mapping between incoming request data and application models is handled by model binders.</span></span> <span data-ttu-id="ea49c-186">Gli sviluppatori possono estendere la funzionalità di associazione di modelli predefinita implementando strumenti di associazione di modelli personalizzati (anche se in genere non è necessario creare un provider personalizzato).</span><span class="sxs-lookup"><span data-stu-id="ea49c-186">Developers can extend the built-in model binding functionality by implementing custom model binders (though typically, you don't need to write your own provider).</span></span>

<span data-ttu-id="ea49c-187">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/mvc/advanced/custom-model-binding/samples) ([procedura per il download](xref:index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="ea49c-187">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/mvc/advanced/custom-model-binding/samples) ([how to download](xref:index#how-to-download-a-sample))</span></span>

## <a name="default-model-binder-limitations"></a><span data-ttu-id="ea49c-188">Limiti degli strumenti di associazione di modelli predefiniti</span><span class="sxs-lookup"><span data-stu-id="ea49c-188">Default model binder limitations</span></span>

<span data-ttu-id="ea49c-189">Gli strumenti di associazione di modelli predefiniti supportano la maggior parte dei tipi di dati .NET Core comuni e soddisfano le esigenze della maggior parte degli sviluppatori.</span><span class="sxs-lookup"><span data-stu-id="ea49c-189">The default model binders support most of the common .NET Core data types and should meet most developers' needs.</span></span> <span data-ttu-id="ea49c-190">Tali strumenti associano direttamente l'input di testo della richiesta ai tipi di modello.</span><span class="sxs-lookup"><span data-stu-id="ea49c-190">They expect to bind text-based input from the request directly to model types.</span></span> <span data-ttu-id="ea49c-191">Può essere necessario trasformare l'input prima dell'associazione.</span><span class="sxs-lookup"><span data-stu-id="ea49c-191">You might need to transform the input prior to binding it.</span></span> <span data-ttu-id="ea49c-192">Un esempio è il caso in cui è presente una chiave che può essere usata per cercare i dati del modello.</span><span class="sxs-lookup"><span data-stu-id="ea49c-192">For example, when you have a key that can be used to look up model data.</span></span> <span data-ttu-id="ea49c-193">È possibile usare uno strumento di associazione di modelli personalizzato per il recupero di dati in base alla chiave.</span><span class="sxs-lookup"><span data-stu-id="ea49c-193">You can use a custom model binder to fetch data based on the key.</span></span>

## <a name="model-binding-review"></a><span data-ttu-id="ea49c-194">Analisi dell'associazione di modelli</span><span class="sxs-lookup"><span data-stu-id="ea49c-194">Model binding review</span></span>

<span data-ttu-id="ea49c-195">L'associazione di modelli usa definizioni specifiche per i tipi sui quali opera.</span><span class="sxs-lookup"><span data-stu-id="ea49c-195">Model binding uses specific definitions for the types it operates on.</span></span> <span data-ttu-id="ea49c-196">Un *tipo semplice* viene convertito da una stringa singola dell'input.</span><span class="sxs-lookup"><span data-stu-id="ea49c-196">A *simple type* is converted from a single string in the input.</span></span> <span data-ttu-id="ea49c-197">Un *tipo complesso* viene convertito da più valori di input.</span><span class="sxs-lookup"><span data-stu-id="ea49c-197">A *complex type* is converted from multiple input values.</span></span> <span data-ttu-id="ea49c-198">Il framework determina la differenza in base all'esistenza di un elemento `TypeConverter`.</span><span class="sxs-lookup"><span data-stu-id="ea49c-198">The framework determines the difference based on the existence of a `TypeConverter`.</span></span> <span data-ttu-id="ea49c-199">È consigliabile creare un convertitore di tipi se è presente un mapping `string` -> `SomeType` semplice che non richiede risorse esterne.</span><span class="sxs-lookup"><span data-stu-id="ea49c-199">We recommended you create a type converter if you have a simple `string` -> `SomeType` mapping that doesn't require external resources.</span></span>

<span data-ttu-id="ea49c-200">Prima di creare uno strumento di associazione di modelli personalizzato, è utile esaminare le modalità di implementazione degli strumenti di associazione di modelli esistenti.</span><span class="sxs-lookup"><span data-stu-id="ea49c-200">Before creating your own custom model binder, it's worth reviewing how existing model binders are implemented.</span></span> <span data-ttu-id="ea49c-201">Si consideri l'oggetto <xref:Microsoft.AspNetCore.Mvc.ModelBinding.Binders.ByteArrayModelBinder> che può essere usato per convertire le stringhe con codifica Base64 in matrici di byte.</span><span class="sxs-lookup"><span data-stu-id="ea49c-201">Consider the <xref:Microsoft.AspNetCore.Mvc.ModelBinding.Binders.ByteArrayModelBinder> which can be used to convert base64-encoded strings into byte arrays.</span></span> <span data-ttu-id="ea49c-202">Le matrici di byte vengono spesso archiviate come file o campi BLOB del database.</span><span class="sxs-lookup"><span data-stu-id="ea49c-202">The byte arrays are often stored as files or database BLOB fields.</span></span>

### <a name="working-with-the-bytearraymodelbinder"></a><span data-ttu-id="ea49c-203">Uso di ByteArrayModelBinder</span><span class="sxs-lookup"><span data-stu-id="ea49c-203">Working with the ByteArrayModelBinder</span></span>

<span data-ttu-id="ea49c-204">Le stringhe con codifica Base64 possono essere usate per rappresentare i dati binari.</span><span class="sxs-lookup"><span data-stu-id="ea49c-204">Base64-encoded strings can be used to represent binary data.</span></span> <span data-ttu-id="ea49c-205">Un'immagine, ad esempio, può essere codificata come stringa.</span><span class="sxs-lookup"><span data-stu-id="ea49c-205">For example, an image can be encoded as a string.</span></span> <span data-ttu-id="ea49c-206">Nell'esempio è inclusa un'immagine come stringa con codifica Base64 in [Base64String.txt](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/mvc/advanced/custom-model-binding/samples/2.x/CustomModelBindingSample/Base64String.txt).</span><span class="sxs-lookup"><span data-stu-id="ea49c-206">The sample includes an image as a base64-encoded string in [Base64String.txt](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/mvc/advanced/custom-model-binding/samples/2.x/CustomModelBindingSample/Base64String.txt).</span></span>

<span data-ttu-id="ea49c-207">ASP.NET Core MVC può accettare una stringa con codifica base64 e usare un oggetto `ByteArrayModelBinder` per convertirla in una matrice di byte.</span><span class="sxs-lookup"><span data-stu-id="ea49c-207">ASP.NET Core MVC can take a base64-encoded string and use a `ByteArrayModelBinder` to convert it into a byte array.</span></span> <span data-ttu-id="ea49c-208"><xref:Microsoft.AspNetCore.Mvc.ModelBinding.Binders.ByteArrayModelBinderProvider>Esegue il mapping degli `byte[]` argomenti a `ByteArrayModelBinder` :</span><span class="sxs-lookup"><span data-stu-id="ea49c-208">The <xref:Microsoft.AspNetCore.Mvc.ModelBinding.Binders.ByteArrayModelBinderProvider> maps `byte[]` arguments to `ByteArrayModelBinder`:</span></span>

```csharp
public IModelBinder GetBinder(ModelBinderProviderContext context)
{
    if (context == null)
    {
        throw new ArgumentNullException(nameof(context));
    }

    if (context.Metadata.ModelType == typeof(byte[]))
    {
        return new ByteArrayModelBinder();
    }

    return null;
}
```

<span data-ttu-id="ea49c-209">Quando si crea un gestore di associazione di modelli personalizzato, è possibile implementare un proprio `IModelBinderProvider` tipo o usare <xref:Microsoft.AspNetCore.Mvc.ModelBinderAttribute> .</span><span class="sxs-lookup"><span data-stu-id="ea49c-209">When creating your own custom model binder, you can implement your own `IModelBinderProvider` type, or use the <xref:Microsoft.AspNetCore.Mvc.ModelBinderAttribute>.</span></span>

<span data-ttu-id="ea49c-210">L'esempio indica come usare `ByteArrayModelBinder` per convertire una stringa con codifica base64 in un `byte[]` e salvare il risultato in un file:</span><span class="sxs-lookup"><span data-stu-id="ea49c-210">The following example shows how to use `ByteArrayModelBinder` to convert a base64-encoded string to a `byte[]` and save the result to a file:</span></span>

[!code-csharp[](custom-model-binding/samples/2.x/CustomModelBindingSample/Controllers/ImageController.cs?name=post1)]

<span data-ttu-id="ea49c-211">È possibile pubblicare (POST) una stringa con codifica base64 in questo metodo API usando uno strumento come [Postman](https://www.getpostman.com/):</span><span class="sxs-lookup"><span data-stu-id="ea49c-211">You can POST a base64-encoded string to this api method using a tool like [Postman](https://www.getpostman.com/):</span></span>

<span data-ttu-id="ea49c-212">![Postman](custom-model-binding/images/postman.png "Postman")</span><span class="sxs-lookup"><span data-stu-id="ea49c-212">![postman](custom-model-binding/images/postman.png "postman")</span></span>

<span data-ttu-id="ea49c-213">Se lo strumento di associazione riesce ad associare i dati della richiesta a proprietà o argomenti denominati in modo appropriato, l'associazione di modelli ha esito positivo.</span><span class="sxs-lookup"><span data-stu-id="ea49c-213">As long as the binder can bind request data to appropriately named properties or arguments, model binding will succeed.</span></span> <span data-ttu-id="ea49c-214">L'esempio seguente indica come usare `ByteArrayModelBinder` con un modello di visualizzazione:</span><span class="sxs-lookup"><span data-stu-id="ea49c-214">The following example shows how to use `ByteArrayModelBinder` with a view model:</span></span>

[!code-csharp[](custom-model-binding/samples/2.x/CustomModelBindingSample/Controllers/ImageController.cs?name=post2&highlight=2)]

## <a name="custom-model-binder-sample"></a><span data-ttu-id="ea49c-215">Esempio di strumento di associazione di modelli personalizzato</span><span class="sxs-lookup"><span data-stu-id="ea49c-215">Custom model binder sample</span></span>

<span data-ttu-id="ea49c-216">In questa sezione viene implementato uno strumento di associazione di modelli personalizzato che:</span><span class="sxs-lookup"><span data-stu-id="ea49c-216">In this section we'll implement a custom model binder that:</span></span>

- <span data-ttu-id="ea49c-217">Converte i dati della richiesta in ingresso in argomenti chiave fortemente tipizzati.</span><span class="sxs-lookup"><span data-stu-id="ea49c-217">Converts incoming request data into strongly typed key arguments.</span></span>
- <span data-ttu-id="ea49c-218">Usa Entity Framework Core per recuperare l'entità associata.</span><span class="sxs-lookup"><span data-stu-id="ea49c-218">Uses Entity Framework Core to fetch the associated entity.</span></span>
- <span data-ttu-id="ea49c-219">Passa l'entità associata come argomento al metodo di azione.</span><span class="sxs-lookup"><span data-stu-id="ea49c-219">Passes the associated entity as an argument to the action method.</span></span>

<span data-ttu-id="ea49c-220">L'esempio seguente usa l'attributo `ModelBinder` nel modello `Author`:</span><span class="sxs-lookup"><span data-stu-id="ea49c-220">The following sample uses the `ModelBinder` attribute on the `Author` model:</span></span>

[!code-csharp[](custom-model-binding/samples/2.x/CustomModelBindingSample/Data/Author.cs?highlight=6)]

<span data-ttu-id="ea49c-221">Nel codice precedente l'attributo `ModelBinder` specifica il tipo di `IModelBinder` da usare per associare i parametri di azione `Author`.</span><span class="sxs-lookup"><span data-stu-id="ea49c-221">In the preceding code, the `ModelBinder` attribute specifies the type of `IModelBinder` that should be used to bind `Author` action parameters.</span></span>

<span data-ttu-id="ea49c-222">La classe `AuthorEntityBinder` seguente associa un parametro `Author` recuperando l'entità da un'origine dati tramite Entity Framework Core e un `authorId`:</span><span class="sxs-lookup"><span data-stu-id="ea49c-222">The following `AuthorEntityBinder` class binds an `Author` parameter by fetching the entity from a data source using Entity Framework Core and an `authorId`:</span></span>

[!code-csharp[](custom-model-binding/samples/2.x/CustomModelBindingSample/Binders/AuthorEntityBinder.cs?name=demo)]

> [!NOTE]
> <span data-ttu-id="ea49c-223">La classe `AuthorEntityBinder` precedente è destinata a illustrare uno strumento di associazione di modelli personalizzato.</span><span class="sxs-lookup"><span data-stu-id="ea49c-223">The preceding `AuthorEntityBinder` class is intended to illustrate a custom model binder.</span></span> <span data-ttu-id="ea49c-224">La classe non ha lo scopo di illustrare procedure consigliate per uno scenario di ricerca.</span><span class="sxs-lookup"><span data-stu-id="ea49c-224">The class isn't intended to illustrate best practices for a lookup scenario.</span></span> <span data-ttu-id="ea49c-225">Per la ricerca, associare l'elemento `authorId` ed eseguire query nel database in un metodo di azione.</span><span class="sxs-lookup"><span data-stu-id="ea49c-225">For lookup, bind the `authorId` and query the database in an action method.</span></span> <span data-ttu-id="ea49c-226">Questo approccio separa gli errori di associazione di modelli dai casi `NotFound`.</span><span class="sxs-lookup"><span data-stu-id="ea49c-226">This approach separates model binding failures from `NotFound` cases.</span></span>

<span data-ttu-id="ea49c-227">L'esempio di codice seguente indica come usare `AuthorEntityBinder` in un metodo di azione:</span><span class="sxs-lookup"><span data-stu-id="ea49c-227">The following code shows how to use the `AuthorEntityBinder` in an action method:</span></span>

[!code-csharp[](custom-model-binding/samples/2.x/CustomModelBindingSample/Controllers/BoundAuthorsController.cs?name=demo2&highlight=2)]

<span data-ttu-id="ea49c-228">L'attributo `ModelBinder` può essere usato per applicare gli elementi `AuthorEntityBinder` ai parametri che non usano convenzioni predefinite:</span><span class="sxs-lookup"><span data-stu-id="ea49c-228">The `ModelBinder` attribute can be used to apply the `AuthorEntityBinder` to parameters that don't use default conventions:</span></span>

[!code-csharp[](custom-model-binding/samples/2.x/CustomModelBindingSample/Controllers/BoundAuthorsController.cs?name=demo1&highlight=2)]

<span data-ttu-id="ea49c-229">In questo esempio, dato che il nome dell'argomento non è il valore `authorId` predefinito, viene specificato nel parametro usando l'attributo `ModelBinder`.</span><span class="sxs-lookup"><span data-stu-id="ea49c-229">In this example, since the name of the argument isn't the default `authorId`, it's specified on the parameter using the `ModelBinder` attribute.</span></span> <span data-ttu-id="ea49c-230">Il controller e il metodo di azione sono semplificati rispetto alla ricerca dell'entità nel metodo di azione.</span><span class="sxs-lookup"><span data-stu-id="ea49c-230">Both the controller and action method are simplified compared to looking up the entity in the action method.</span></span> <span data-ttu-id="ea49c-231">La logica per recuperare l'autore usando Entity Framework Core viene spostata nello strumento di associazione di modelli.</span><span class="sxs-lookup"><span data-stu-id="ea49c-231">The logic to fetch the author using Entity Framework Core is moved to the model binder.</span></span> <span data-ttu-id="ea49c-232">Ciò può rappresentare una semplificazione notevole in presenza di vari metodi associati al modello `Author`.</span><span class="sxs-lookup"><span data-stu-id="ea49c-232">This can be a considerable simplification when you have several methods that bind to the `Author` model.</span></span>

<span data-ttu-id="ea49c-233">È possibile applicare l'attributo `ModelBinder` a singole proprietà del modello (ad esempio a ViewModel) o a parametri del metodo di azione per specificare un determinato strumento di associazione di modelli o nome di modello solo per quel determinato tipo o azione.</span><span class="sxs-lookup"><span data-stu-id="ea49c-233">You can apply the `ModelBinder` attribute to individual model properties (such as on a viewmodel) or to action method parameters to specify a certain model binder or model name for just that type or action.</span></span>

### <a name="implementing-a-modelbinderprovider"></a><span data-ttu-id="ea49c-234">Implementazione di un elemento ModelBinderProvider</span><span class="sxs-lookup"><span data-stu-id="ea49c-234">Implementing a ModelBinderProvider</span></span>

<span data-ttu-id="ea49c-235">Anziché applicare un attributo, è possibile implementare `IModelBinderProvider`.</span><span class="sxs-lookup"><span data-stu-id="ea49c-235">Instead of applying an attribute, you can implement `IModelBinderProvider`.</span></span> <span data-ttu-id="ea49c-236">Gli strumenti di associazione del framework incorporati vengono implementati in questo modo.</span><span class="sxs-lookup"><span data-stu-id="ea49c-236">This is how the built-in framework binders are implemented.</span></span> <span data-ttu-id="ea49c-237">Quando si specifica il tipo sul quale opera lo strumento di associazione, si indica il tipo di argomento prodotto dallo strumento, **non** l'input accettato dallo strumento.</span><span class="sxs-lookup"><span data-stu-id="ea49c-237">When you specify the type your binder operates on, you specify the type of argument it produces, **not** the input your binder accepts.</span></span> <span data-ttu-id="ea49c-238">Il provider strumento di associazione seguente funziona con l'elemento `AuthorEntityBinder`.</span><span class="sxs-lookup"><span data-stu-id="ea49c-238">The following binder provider works with the `AuthorEntityBinder`.</span></span> <span data-ttu-id="ea49c-239">Quando viene aggiunto alla raccolta di provider di MVC, non è necessario usare l'attributo `ModelBinder` sui parametri tipizzati `Author` o `Author`.</span><span class="sxs-lookup"><span data-stu-id="ea49c-239">When it's added to MVC's collection of providers, you don't need to use the `ModelBinder` attribute on `Author` or `Author`-typed parameters.</span></span>

[!code-csharp[](custom-model-binding/samples/2.x/CustomModelBindingSample/Binders/AuthorEntityBinderProvider.cs?highlight=17-20)]

> <span data-ttu-id="ea49c-240">Nota: il codice precedente restituisce un elemento `BinderTypeModelBinder`.</span><span class="sxs-lookup"><span data-stu-id="ea49c-240">Note: The preceding code returns a `BinderTypeModelBinder`.</span></span> <span data-ttu-id="ea49c-241">`BinderTypeModelBinder` funziona come factory per gli strumenti di associazione di modelli e implementa la funzionalità DI (Dependency Injection, Inserimento di dipendenze).</span><span class="sxs-lookup"><span data-stu-id="ea49c-241">`BinderTypeModelBinder` acts as a factory for model binders and provides dependency injection (DI).</span></span> <span data-ttu-id="ea49c-242">`AuthorEntityBinder` richiede DI (Dependency Injection, Inserimento di dipendenze) per l'accesso a EF Core.</span><span class="sxs-lookup"><span data-stu-id="ea49c-242">The `AuthorEntityBinder` requires DI to access EF Core.</span></span> <span data-ttu-id="ea49c-243">Usare `BinderTypeModelBinder` se lo strumento di associazione di modelli richiede servizi da DI (Dependency Injection, Inserimento di dipendenze).</span><span class="sxs-lookup"><span data-stu-id="ea49c-243">Use `BinderTypeModelBinder` if your model binder requires services from DI.</span></span>

<span data-ttu-id="ea49c-244">Per usare un provider dello strumento di associazione di modelli personalizzato, aggiungerlo in `ConfigureServices`:</span><span class="sxs-lookup"><span data-stu-id="ea49c-244">To use a custom model binder provider, add it in `ConfigureServices`:</span></span>

[!code-csharp[](custom-model-binding/samples/2.x/CustomModelBindingSample/Startup.cs?name=snippet_ConfigureServices&highlight=5-10)]

<span data-ttu-id="ea49c-245">Durante la valutazione degli strumenti di associazione di modelli, la raccolta di provider viene esaminata in ordine.</span><span class="sxs-lookup"><span data-stu-id="ea49c-245">When evaluating model binders, the collection of providers is examined in order.</span></span> <span data-ttu-id="ea49c-246">Viene usato il primo provider che restituisce uno strumento di associazione.</span><span class="sxs-lookup"><span data-stu-id="ea49c-246">The first provider that returns a binder is used.</span></span> <span data-ttu-id="ea49c-247">Se il provider personalizzato viene aggiunto alla fine della raccolta, è possibile che uno strumento di associazione di modelli incorporato venga chiamato prima dello strumento di associazione di modelli personalizzato.</span><span class="sxs-lookup"><span data-stu-id="ea49c-247">Adding your provider to the end of the collection may result in a built-in model binder being called before your custom binder has a chance.</span></span> <span data-ttu-id="ea49c-248">In questo esempio il provider personalizzato viene aggiunto all'inizio della raccolta, per garantire che venga usato per gli argomenti dell'azione `Author`.</span><span class="sxs-lookup"><span data-stu-id="ea49c-248">In this example, the custom provider is added to the beginning of the collection to ensure it's used for `Author` action arguments.</span></span>

### <a name="polymorphic-model-binding"></a><span data-ttu-id="ea49c-249">Associazione di modelli polimorfici</span><span class="sxs-lookup"><span data-stu-id="ea49c-249">Polymorphic model binding</span></span>

<span data-ttu-id="ea49c-250">L'associazione a modelli diversi di tipi derivati è nota come associazione di modelli polimorfici.</span><span class="sxs-lookup"><span data-stu-id="ea49c-250">Binding to different models of derived types is known as polymorphic model binding.</span></span> <span data-ttu-id="ea49c-251">L'associazione di modelli personalizzati polimorfici è obbligatoria quando il valore della richiesta deve essere associato al tipo di modello derivato specifico.</span><span class="sxs-lookup"><span data-stu-id="ea49c-251">Polymorphic custom model binding is required when the request value must be bound to the specific derived model type.</span></span> <span data-ttu-id="ea49c-252">Associazione di modelli polimorfici:</span><span class="sxs-lookup"><span data-stu-id="ea49c-252">Polymorphic model binding:</span></span>

* <span data-ttu-id="ea49c-253">Non è tipico per un'API REST progettata per interagire con tutti i linguaggi.</span><span class="sxs-lookup"><span data-stu-id="ea49c-253">Isn't typical for a REST API that's designed to interoperate with all languages.</span></span>
* <span data-ttu-id="ea49c-254">Rende difficile la motivazione dei modelli associati.</span><span class="sxs-lookup"><span data-stu-id="ea49c-254">Makes it difficult to reason about the bound models.</span></span>

<span data-ttu-id="ea49c-255">Tuttavia, se un'app richiede l'associazione di modelli polimorfici, un'implementazione potrebbe avere un aspetto simile al codice seguente:</span><span class="sxs-lookup"><span data-stu-id="ea49c-255">However, if an app requires polymorphic model binding, an implementation might look like the following code:</span></span>

[!code-csharp[](custom-model-binding/samples/3.x/PolymorphicModelBindingSample/ModelBinders/PolymorphicModelBinder.cs?name=snippet)]

## <a name="recommendations-and-best-practices"></a><span data-ttu-id="ea49c-256">Suggerimenti e procedure ottimali</span><span class="sxs-lookup"><span data-stu-id="ea49c-256">Recommendations and best practices</span></span>

<span data-ttu-id="ea49c-257">Gli strumenti di associazione di modelli personalizzati:</span><span class="sxs-lookup"><span data-stu-id="ea49c-257">Custom model binders:</span></span>

- <span data-ttu-id="ea49c-258">Non devono provare a impostare codici di stato o restituire risultati (ad esempio 404 Non trovato).</span><span class="sxs-lookup"><span data-stu-id="ea49c-258">Shouldn't attempt to set status codes or return results (for example, 404 Not Found).</span></span> <span data-ttu-id="ea49c-259">Se si verifica un errore nell'associazione di modelli, l'errore deve essere gestito da un [filtro azioni](xref:mvc/controllers/filters) o da logica inclusa nel metodo di azione.</span><span class="sxs-lookup"><span data-stu-id="ea49c-259">If model binding fails, an [action filter](xref:mvc/controllers/filters) or logic within the action method itself should handle the failure.</span></span>
- <span data-ttu-id="ea49c-260">Sono particolarmente utili per eliminare codice ripetitivo e problemi di montaggio incrociato dai metodi di azione.</span><span class="sxs-lookup"><span data-stu-id="ea49c-260">Are most useful for eliminating repetitive code and cross-cutting concerns from action methods.</span></span>
- <span data-ttu-id="ea49c-261">In genere non devono essere usati per convertire una stringa in un tipo personalizzato. Un elemento <xref:System.ComponentModel.TypeConverter> rappresenta solitamente una scelta migliore.</span><span class="sxs-lookup"><span data-stu-id="ea49c-261">Typically shouldn't be used to convert a string into a custom type, a <xref:System.ComponentModel.TypeConverter> is usually a better option.</span></span>

::: moniker-end
