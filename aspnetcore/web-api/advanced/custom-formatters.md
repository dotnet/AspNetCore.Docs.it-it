---
title: Formattatori personalizzati nell'API Web ASP.NET Core
author: rick-anderson
description: Informazioni su come creare e usare i formattatori personalizzati nelle API Web ASP.NET Core.
ms.author: riande
ms.date: 06/25/2020
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
uid: web-api/advanced/custom-formatters
ms.openlocfilehash: 91c9c6513d7c8df671e283508ecc276768d79539
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102587827"
---
# <a name="custom-formatters-in-aspnet-core-web-api"></a><span data-ttu-id="7009d-103">Formattatori personalizzati nell'API Web ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="7009d-103">Custom formatters in ASP.NET Core Web API</span></span>

<span data-ttu-id="7009d-104">Di [Kirk Larkin](https://twitter.com/serpent5) e [Tom Dykstra](https://github.com/tdykstra).</span><span class="sxs-lookup"><span data-stu-id="7009d-104">By [Kirk Larkin](https://twitter.com/serpent5) and [Tom Dykstra](https://github.com/tdykstra).</span></span>

<span data-ttu-id="7009d-105">ASP.NET Core MVC supporta lo scambio di dati in API Web mediante formattatori di input e di output.</span><span class="sxs-lookup"><span data-stu-id="7009d-105">ASP.NET Core MVC supports data exchange in Web APIs using input and output formatters.</span></span> <span data-ttu-id="7009d-106">I formattatori di input vengono utilizzati dall' [associazione di modelli](xref:mvc/models/model-binding).</span><span class="sxs-lookup"><span data-stu-id="7009d-106">Input formatters are used by [Model Binding](xref:mvc/models/model-binding).</span></span> <span data-ttu-id="7009d-107">I formattatori di output vengono utilizzati per [formattare le risposte](xref:web-api/advanced/formatting).</span><span class="sxs-lookup"><span data-stu-id="7009d-107">Output formatters are used to [format responses](xref:web-api/advanced/formatting).</span></span>

<span data-ttu-id="7009d-108">Il Framework fornisce formattatori di input e output predefiniti per JSON e XML.</span><span class="sxs-lookup"><span data-stu-id="7009d-108">The framework provides built-in input and output formatters for JSON and XML.</span></span> <span data-ttu-id="7009d-109">Fornisce un formattatore di output incorporato per il testo normale, ma non fornisce un formattatore di input per testo normale.</span><span class="sxs-lookup"><span data-stu-id="7009d-109">It provides a built-in output formatter for plain text, but doesn't provide an input formatter for plain text.</span></span>

<span data-ttu-id="7009d-110">In questo articolo viene illustrato come aggiungere supporto per altri formati creando formattatori personalizzati.</span><span class="sxs-lookup"><span data-stu-id="7009d-110">This article shows how to add support for additional formats by creating custom formatters.</span></span> <span data-ttu-id="7009d-111">Per un esempio di formattatore di input di testo normale personalizzato, vedere [TextPlainInputFormatter](https://github.com/aspnet/Entropy/blob/master/samples/Mvc.Formatters/TextPlainInputFormatter.cs) su GitHub.</span><span class="sxs-lookup"><span data-stu-id="7009d-111">For an example of a custom plain text input formatter, see [TextPlainInputFormatter](https://github.com/aspnet/Entropy/blob/master/samples/Mvc.Formatters/TextPlainInputFormatter.cs) on GitHub.</span></span>

<span data-ttu-id="7009d-112">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/web-api/advanced/custom-formatters/samples) ([procedura per il download](xref:index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="7009d-112">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/web-api/advanced/custom-formatters/samples) ([how to download](xref:index#how-to-download-a-sample))</span></span>

## <a name="when-to-use-custom-formatters"></a><span data-ttu-id="7009d-113">Quando usare i formattatori personalizzati</span><span class="sxs-lookup"><span data-stu-id="7009d-113">When to use custom formatters</span></span>

<span data-ttu-id="7009d-114">Usare un formattatore personalizzato per aggiungere il supporto per un tipo di contenuto non gestito dai formattatori predefiniti.</span><span class="sxs-lookup"><span data-stu-id="7009d-114">Use a custom formatter to add support for a content type that isn't handled by the built-in formatters.</span></span>

## <a name="overview-of-how-to-use-a-custom-formatter"></a><span data-ttu-id="7009d-115">Panoramica sull'uso di un formattatore personalizzato</span><span class="sxs-lookup"><span data-stu-id="7009d-115">Overview of how to use a custom formatter</span></span>

<span data-ttu-id="7009d-116">Per creare un formattatore personalizzato:</span><span class="sxs-lookup"><span data-stu-id="7009d-116">To create a custom formatter:</span></span>

* <span data-ttu-id="7009d-117">Per la serializzazione dei dati inviati al client, creare una classe formattatore di output.</span><span class="sxs-lookup"><span data-stu-id="7009d-117">For serializing data sent to the client, create an output formatter class.</span></span>
* <span data-ttu-id="7009d-118">Per la deserializzazione dei dati ricevuti dal client, creare una classe formatter di input.</span><span class="sxs-lookup"><span data-stu-id="7009d-118">For deserializing data received from the client, create an input formatter class.</span></span>
* <span data-ttu-id="7009d-119">Aggiungere istanze delle classi formatter alle `InputFormatters` raccolte e `OutputFormatters` in <xref:Microsoft.AspNetCore.Mvc.MvcOptions> .</span><span class="sxs-lookup"><span data-stu-id="7009d-119">Add instances of formatter classes to the `InputFormatters` and `OutputFormatters` collections in <xref:Microsoft.AspNetCore.Mvc.MvcOptions>.</span></span>

## <a name="how-to-create-a-custom-formatter-class"></a><span data-ttu-id="7009d-120">Come creare una classe formattatore personalizzato</span><span class="sxs-lookup"><span data-stu-id="7009d-120">How to create a custom formatter class</span></span>

<span data-ttu-id="7009d-121">Per creare un formattatore:</span><span class="sxs-lookup"><span data-stu-id="7009d-121">To create a formatter:</span></span>

* <span data-ttu-id="7009d-122">Derivare la classe dalla classe di base appropriata.</span><span class="sxs-lookup"><span data-stu-id="7009d-122">Derive the class from the appropriate base class.</span></span> <span data-ttu-id="7009d-123">L'app di esempio deriva da <xref:Microsoft.AspNetCore.Mvc.Formatters.TextOutputFormatter> e <xref:Microsoft.AspNetCore.Mvc.Formatters.TextInputFormatter> .</span><span class="sxs-lookup"><span data-stu-id="7009d-123">The sample app derives from <xref:Microsoft.AspNetCore.Mvc.Formatters.TextOutputFormatter> and <xref:Microsoft.AspNetCore.Mvc.Formatters.TextInputFormatter>.</span></span>
* <span data-ttu-id="7009d-124">Specificare i formati di testo multimediali validi e le codifiche nel costruttore.</span><span class="sxs-lookup"><span data-stu-id="7009d-124">Specify valid media types and encodings in the constructor.</span></span>
* <span data-ttu-id="7009d-125">Eseguire l'override dei metodi <xref:Microsoft.AspNetCore.Mvc.Formatters.InputFormatter.CanReadType%2A> e <xref:Microsoft.AspNetCore.Mvc.Formatters.OutputFormatter.CanWriteType%2A>.</span><span class="sxs-lookup"><span data-stu-id="7009d-125">Override the <xref:Microsoft.AspNetCore.Mvc.Formatters.InputFormatter.CanReadType%2A> and <xref:Microsoft.AspNetCore.Mvc.Formatters.OutputFormatter.CanWriteType%2A> methods.</span></span>
* <span data-ttu-id="7009d-126">Eseguire l'override dei metodi <xref:Microsoft.AspNetCore.Mvc.Formatters.InputFormatter.ReadRequestBodyAsync%2A> e `WriteResponseBodyAsync`.</span><span class="sxs-lookup"><span data-stu-id="7009d-126">Override the <xref:Microsoft.AspNetCore.Mvc.Formatters.InputFormatter.ReadRequestBodyAsync%2A> and `WriteResponseBodyAsync` methods.</span></span>

<span data-ttu-id="7009d-127">Nel codice seguente viene illustrata la `VcardOutputFormatter` classe dell' [esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/web-api/advanced/custom-formatters/samples):</span><span class="sxs-lookup"><span data-stu-id="7009d-127">The following code shows the `VcardOutputFormatter` class from the [sample](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/web-api/advanced/custom-formatters/samples):</span></span>

[!code-csharp[](custom-formatters/samples/3.x/CustomFormattersSample/Formatters/VcardOutputFormatter.cs?name=snippet_Class)]
  
### <a name="derive-from-the-appropriate-base-class"></a><span data-ttu-id="7009d-128">Derivare dalla classe di base appropriata</span><span class="sxs-lookup"><span data-stu-id="7009d-128">Derive from the appropriate base class</span></span>

<span data-ttu-id="7009d-129">Per i tipi di supporti di testo (ad esempio, vCard), derivare dalla <xref:Microsoft.AspNetCore.Mvc.Formatters.TextInputFormatter> <xref:Microsoft.AspNetCore.Mvc.Formatters.TextOutputFormatter> classe di base o.</span><span class="sxs-lookup"><span data-stu-id="7009d-129">For text media types (for example, vCard), derive from the <xref:Microsoft.AspNetCore.Mvc.Formatters.TextInputFormatter> or <xref:Microsoft.AspNetCore.Mvc.Formatters.TextOutputFormatter> base class.</span></span>

[!code-csharp[](custom-formatters/samples/3.x/CustomFormattersSample/Formatters/VcardOutputFormatter.cs?name=snippet_ClassDeclaration)]

<span data-ttu-id="7009d-130">Per i tipi binari, derivare <xref:Microsoft.AspNetCore.Mvc.Formatters.InputFormatter> dalla <xref:Microsoft.AspNetCore.Mvc.Formatters.OutputFormatter> classe di base o.</span><span class="sxs-lookup"><span data-stu-id="7009d-130">For binary types, derive from the <xref:Microsoft.AspNetCore.Mvc.Formatters.InputFormatter> or <xref:Microsoft.AspNetCore.Mvc.Formatters.OutputFormatter> base class.</span></span>

### <a name="specify-valid-media-types-and-encodings"></a><span data-ttu-id="7009d-131">Specificare i formati multimediali validi e le codifiche</span><span class="sxs-lookup"><span data-stu-id="7009d-131">Specify valid media types and encodings</span></span>

<span data-ttu-id="7009d-132">Nel costruttore specificare i formati multimediali validi e le codifiche aggiungendo alle raccolte `SupportedMediaTypes` e `SupportedEncodings`.</span><span class="sxs-lookup"><span data-stu-id="7009d-132">In the constructor, specify valid media types and encodings by adding to the `SupportedMediaTypes` and `SupportedEncodings` collections.</span></span>

[!code-csharp[](custom-formatters/samples/3.x/CustomFormattersSample/Formatters/VcardOutputFormatter.cs?name=snippet_ctor)]

<span data-ttu-id="7009d-133">Una classe formatter **non** può utilizzare l'inserimento del costruttore per le relative dipendenze.</span><span class="sxs-lookup"><span data-stu-id="7009d-133">A formatter class can **not** use constructor injection for its dependencies.</span></span> <span data-ttu-id="7009d-134">Ad esempio, `ILogger<VcardOutputFormatter>` non può essere aggiunto come parametro al costruttore.</span><span class="sxs-lookup"><span data-stu-id="7009d-134">For example, `ILogger<VcardOutputFormatter>` cannot be added as a parameter to the constructor.</span></span> <span data-ttu-id="7009d-135">Per accedere ai servizi, usare l'oggetto di contesto che viene passato ai metodi.</span><span class="sxs-lookup"><span data-stu-id="7009d-135">To access services, use the context object that gets passed in to the methods.</span></span> <span data-ttu-id="7009d-136">Un esempio di codice in questo articolo e nell' [esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/web-api/advanced/custom-formatters/samples) illustra come eseguire questa operazione.</span><span class="sxs-lookup"><span data-stu-id="7009d-136">A code example in this article and the [sample](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/web-api/advanced/custom-formatters/samples) show how to do this.</span></span>

### <a name="override-canreadtype-and-canwritetype"></a><span data-ttu-id="7009d-137">Eseguire l'override di CanReadType e CanWriteType</span><span class="sxs-lookup"><span data-stu-id="7009d-137">Override CanReadType and CanWriteType</span></span>

<span data-ttu-id="7009d-138">Specificare il tipo in cui eseguire la deserializzazione o serializzare eseguendo l'override dei `CanReadType` `CanWriteType` metodi o.</span><span class="sxs-lookup"><span data-stu-id="7009d-138">Specify the type to deserialize into or serialize from by overriding the `CanReadType` or `CanWriteType` methods.</span></span> <span data-ttu-id="7009d-139">Ad esempio, la creazione di testo vCard da un `Contact` tipo e viceversa.</span><span class="sxs-lookup"><span data-stu-id="7009d-139">For example, creating vCard text from a `Contact` type and vice versa.</span></span>

[!code-csharp[](custom-formatters/samples/3.x/CustomFormattersSample/Formatters/VcardOutputFormatter.cs?name=snippet_CanWriteType)]

#### <a name="the-canwriteresult-method"></a><span data-ttu-id="7009d-140">Metodo CanWriteResult</span><span class="sxs-lookup"><span data-stu-id="7009d-140">The CanWriteResult method</span></span>

<span data-ttu-id="7009d-141">In alcuni scenari `CanWriteResult` è necessario eseguire l'override di anziché `CanWriteType` .</span><span class="sxs-lookup"><span data-stu-id="7009d-141">In some scenarios, `CanWriteResult` must be overridden rather than `CanWriteType`.</span></span> <span data-ttu-id="7009d-142">Usare `CanWriteResult` se vengono soddisfatte le condizioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="7009d-142">Use `CanWriteResult` if the following conditions are true:</span></span>

* <span data-ttu-id="7009d-143">Il metodo di azione restituisce una classe di modello.</span><span class="sxs-lookup"><span data-stu-id="7009d-143">The action method returns a model class.</span></span>
* <span data-ttu-id="7009d-144">Sono disponibili classi derivate che possono essere restituite in fase di esecuzione.</span><span class="sxs-lookup"><span data-stu-id="7009d-144">There are derived classes which might be returned at runtime.</span></span>
* <span data-ttu-id="7009d-145">La classe derivata restituita dall'azione deve essere nota in fase di esecuzione.</span><span class="sxs-lookup"><span data-stu-id="7009d-145">The derived class returned by the action must be known at runtime.</span></span>

<span data-ttu-id="7009d-146">Si supponga, ad esempio, che il metodo di azione:</span><span class="sxs-lookup"><span data-stu-id="7009d-146">For example, suppose the action method:</span></span>

* <span data-ttu-id="7009d-147">La firma restituisce un `Person` tipo.</span><span class="sxs-lookup"><span data-stu-id="7009d-147">Signature returns a `Person` type.</span></span>
* <span data-ttu-id="7009d-148">Può restituire un `Student` `Instructor` tipo o che deriva da `Person` .</span><span class="sxs-lookup"><span data-stu-id="7009d-148">Can return a `Student` or `Instructor` type that derives from `Person`.</span></span> 

<span data-ttu-id="7009d-149">Affinché il formattatore gestisca solo `Student` oggetti, controllare il tipo di <xref:Microsoft.AspNetCore.Mvc.Formatters.OutputFormatterCanWriteContext.Object> nell'oggetto di contesto fornito al `CanWriteResult` metodo.</span><span class="sxs-lookup"><span data-stu-id="7009d-149">For the formatter to handle only `Student` objects, check the type of <xref:Microsoft.AspNetCore.Mvc.Formatters.OutputFormatterCanWriteContext.Object> in the context object provided to the `CanWriteResult` method.</span></span> <span data-ttu-id="7009d-150">Quando il metodo di azione restituisce `IActionResult` :</span><span class="sxs-lookup"><span data-stu-id="7009d-150">When the action method returns `IActionResult`:</span></span>

* <span data-ttu-id="7009d-151">Non è necessario usare `CanWriteResult` .</span><span class="sxs-lookup"><span data-stu-id="7009d-151">It's not necessary to use `CanWriteResult`.</span></span>
* <span data-ttu-id="7009d-152">Il `CanWriteType` metodo riceve il tipo di Runtime.</span><span class="sxs-lookup"><span data-stu-id="7009d-152">The `CanWriteType` method receives the runtime type.</span></span>

<a id="read-write"></a>

### <a name="override-readrequestbodyasync-and-writeresponsebodyasync"></a><span data-ttu-id="7009d-153">Eseguire l'override di ReadRequestBodyAsync e WriteResponseBodyAsync</span><span class="sxs-lookup"><span data-stu-id="7009d-153">Override ReadRequestBodyAsync and WriteResponseBodyAsync</span></span>

<span data-ttu-id="7009d-154">La deserializzazione o la serializzazione viene eseguita in `ReadRequestBodyAsync` o `WriteResponseBodyAsync` .</span><span class="sxs-lookup"><span data-stu-id="7009d-154">Deserialization or serialization is performed in `ReadRequestBodyAsync` or `WriteResponseBodyAsync`.</span></span> <span data-ttu-id="7009d-155">Nell'esempio seguente viene illustrato come ottenere servizi dal contenitore di inserimento delle dipendenze.</span><span class="sxs-lookup"><span data-stu-id="7009d-155">The following example shows how to get services from the dependency injection container.</span></span> <span data-ttu-id="7009d-156">Non è possibile ottenere i servizi dai parametri del costruttore.</span><span class="sxs-lookup"><span data-stu-id="7009d-156">Services can't be obtained from constructor parameters.</span></span>

[!code-csharp[](custom-formatters/samples/3.x/CustomFormattersSample/Formatters/VcardOutputFormatter.cs?name=snippet_WriteResponseBodyAsync)]

## <a name="how-to-configure-mvc-to-use-a-custom-formatter"></a><span data-ttu-id="7009d-157">Come configurare MVC per usare un formattatore personalizzato</span><span class="sxs-lookup"><span data-stu-id="7009d-157">How to configure MVC to use a custom formatter</span></span>

<span data-ttu-id="7009d-158">Per usare un formattatore personalizzato, aggiungere un'istanza della classe formattatore alla raccolta `InputFormatters` o `OutputFormatters`.</span><span class="sxs-lookup"><span data-stu-id="7009d-158">To use a custom formatter, add an instance of the formatter class to the `InputFormatters` or `OutputFormatters` collection.</span></span>

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](custom-formatters/samples/3.x/CustomFormattersSample/Startup.cs?name=snippet_ConfigureServices&highlight=5-6)]

::: moniker-end

::: moniker range="< aspnetcore-3.0"

[!code-csharp[](custom-formatters/samples/2.x/CustomFormattersSample/Startup.cs?name=mvcoptions&highlight=3-4)]

::: moniker-end

<span data-ttu-id="7009d-159">I formattatori vengono valutati nell'ordine d'inserimento.</span><span class="sxs-lookup"><span data-stu-id="7009d-159">Formatters are evaluated in the order you insert them.</span></span> <span data-ttu-id="7009d-160">Il primo ha la precedenza.</span><span class="sxs-lookup"><span data-stu-id="7009d-160">The first one takes precedence.</span></span>

## <a name="the-complete-vcardinputformatter-class"></a><span data-ttu-id="7009d-161">Classe completa `VcardInputFormatter`</span><span class="sxs-lookup"><span data-stu-id="7009d-161">The complete `VcardInputFormatter` class</span></span>

<span data-ttu-id="7009d-162">Nel codice seguente viene illustrata la `VcardInputFormatter` classe dell' [esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/web-api/advanced/custom-formatters/samples):</span><span class="sxs-lookup"><span data-stu-id="7009d-162">The following code shows the `VcardInputFormatter` class from the [sample](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/web-api/advanced/custom-formatters/samples):</span></span>

[!code-csharp[](custom-formatters/samples/3.x/CustomFormattersSample/Formatters/VcardInputFormatter.cs?name=snippet_Class)]

## <a name="test-the-app"></a><span data-ttu-id="7009d-163">Testare l'app</span><span class="sxs-lookup"><span data-stu-id="7009d-163">Test the app</span></span>

<span data-ttu-id="7009d-164">[Eseguire l'app di esempio per questo articolo](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/web-api/advanced/custom-formatters/samples), che implementa i formattatori di input e output vCard di base.</span><span class="sxs-lookup"><span data-stu-id="7009d-164">[Run the sample app for this article](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/web-api/advanced/custom-formatters/samples), which implements basic vCard input and output formatters.</span></span> <span data-ttu-id="7009d-165">L'app legge e scrive le vCard simili alle seguenti:</span><span class="sxs-lookup"><span data-stu-id="7009d-165">The app reads and writes vCards similar to the following:</span></span>

```
BEGIN:VCARD
VERSION:2.1
N:Davolio;Nancy
FN:Nancy Davolio
END:VCARD
```

<span data-ttu-id="7009d-166">Per visualizzare l'output della vCard, eseguire l'app e inviare una richiesta GET con l'intestazione Accept `text/vcard` a `https://localhost:5001/api/contacts` .</span><span class="sxs-lookup"><span data-stu-id="7009d-166">To see vCard output, run the app and send a Get request with Accept header `text/vcard` to `https://localhost:5001/api/contacts`.</span></span>

<span data-ttu-id="7009d-167">Per aggiungere un vCard alla raccolta di contatti in memoria:</span><span class="sxs-lookup"><span data-stu-id="7009d-167">To add a vCard to the in-memory collection of contacts:</span></span>

* <span data-ttu-id="7009d-168">Inviare una `Post` richiesta a `/api/contacts` con uno strumento come il post.</span><span class="sxs-lookup"><span data-stu-id="7009d-168">Send a `Post` request to `/api/contacts` with a tool like Postman.</span></span>
* <span data-ttu-id="7009d-169">Impostare l'intestazione `Content-Type` su `text/vcard`.</span><span class="sxs-lookup"><span data-stu-id="7009d-169">Set the `Content-Type` header to `text/vcard`.</span></span>
* <span data-ttu-id="7009d-170">Imposta il `vCard` testo nel corpo, formattato come l'esempio precedente.</span><span class="sxs-lookup"><span data-stu-id="7009d-170">Set `vCard` text in the body, formatted like the preceding example.</span></span>

## <a name="additional-resources"></a><span data-ttu-id="7009d-171">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="7009d-171">Additional resources</span></span>

* <xref:web-api/advanced/formatting>
* <xref:grpc/dotnet-grpc>
