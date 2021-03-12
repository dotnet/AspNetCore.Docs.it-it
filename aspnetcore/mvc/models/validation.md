---
title: Convalida del modello in ASP.NET Core MVC
author: rick-anderson
description: Informazioni sulla convalida dei modelli in ASP.NET Core MVC e Razor pagine.
ms.author: riande
ms.custom: mvc
ms.date: 12/15/2019
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
uid: mvc/models/validation
ms.openlocfilehash: 21f2a65bd93c08f16de988381e648768debde438
ms.sourcegitcommit: acfe51c35497a204f75c2a61125c9408c04493e6
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102605594"
---
# <a name="model-validation-in-aspnet-core-mvc-and-razor-pages"></a><span data-ttu-id="a6fe0-103">Convalida di modelli in ASP.NET Core MVC e Razor pagine</span><span class="sxs-lookup"><span data-stu-id="a6fe0-103">Model validation in ASP.NET Core MVC and Razor Pages</span></span>

::: moniker range=">= aspnetcore-3.0"

<span data-ttu-id="a6fe0-104">Di [Kirk Larkin](https://github.com/serpent5)</span><span class="sxs-lookup"><span data-stu-id="a6fe0-104">By [Kirk Larkin](https://github.com/serpent5)</span></span>

<span data-ttu-id="a6fe0-105">Questo articolo illustra come convalidare l'input dell'utente in un'app ASP.NET Core MVC o Razor pages.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-105">This article explains how to validate user input in an ASP.NET Core MVC or Razor Pages app.</span></span>

<span data-ttu-id="a6fe0-106">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/mvc/models/validation/samples) ([procedura per il download](xref:index#how-to-download-a-sample)).</span><span class="sxs-lookup"><span data-stu-id="a6fe0-106">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/mvc/models/validation/samples) ([how to download](xref:index#how-to-download-a-sample)).</span></span>

## <a name="model-state"></a><span data-ttu-id="a6fe0-107">Stato del modello</span><span class="sxs-lookup"><span data-stu-id="a6fe0-107">Model state</span></span>

<span data-ttu-id="a6fe0-108">Lo stato del modello rappresenta gli errori che provengono da due sottosistemi: associazione di modelli e convalida del modello.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-108">Model state represents errors that come from two subsystems: model binding and model validation.</span></span> <span data-ttu-id="a6fe0-109">Gli errori originati dall' [associazione di modelli](model-binding.md) sono in genere errori di conversione dei dati.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-109">Errors that originate from [model binding](model-binding.md) are generally data conversion errors.</span></span> <span data-ttu-id="a6fe0-110">Ad esempio, una "x" viene immessa in un campo integer.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-110">For example, an "x" is entered in an integer field.</span></span> <span data-ttu-id="a6fe0-111">La convalida del modello avviene dopo l'associazione di modelli e segnala gli errori in cui i dati non sono conformi alle regole business</span><span class="sxs-lookup"><span data-stu-id="a6fe0-111">Model validation occurs after model binding and reports errors where data doesn't conform to business rules.</span></span> <span data-ttu-id="a6fe0-112">Ad esempio, un valore 0 viene immesso in un campo che prevede una classificazione compresa tra 1 e 5.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-112">For example, a 0 is entered in a field that expects a rating between 1 and 5.</span></span>

<span data-ttu-id="a6fe0-113">L'associazione di modelli e la convalida del modello si verificano prima dell'esecuzione di un'azione del controller o di un Razor metodo del gestore di pagine.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-113">Both model binding and model validation occur before the execution of a controller action or a Razor Pages handler method.</span></span> <span data-ttu-id="a6fe0-114">Per le app Web, è responsabilità dell'app esaminare `ModelState.IsValid` e rispondere nel modo appropriato.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-114">For web apps, it's the app's responsibility to inspect `ModelState.IsValid` and react appropriately.</span></span> <span data-ttu-id="a6fe0-115">Le app Web in genere visualizzare di nuovo la pagina con un messaggio di errore:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-115">Web apps typically redisplay the page with an error message:</span></span>

[!code-csharp[](validation/samples/3.x/ValidationSample/Pages/Movies/Create.cshtml.cs?name=snippet_OnPostAsync&highlight=3-6)]

<span data-ttu-id="a6fe0-116">I controller API Web non devono controllare `ModelState.IsValid` se hanno l'attributo `[ApiController]`.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-116">Web API controllers don't have to check `ModelState.IsValid` if they have the `[ApiController]` attribute.</span></span> <span data-ttu-id="a6fe0-117">In tal caso, viene restituita una risposta HTTP 400 automatica contenente i dettagli dell'errore quando lo stato del modello non è valido.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-117">In that case, an automatic HTTP 400 response containing error details is returned when model state is invalid.</span></span> <span data-ttu-id="a6fe0-118">Per altre informazioni, vedere [Risposte HTTP 400 automatiche](xref:web-api/index#automatic-http-400-responses).</span><span class="sxs-lookup"><span data-stu-id="a6fe0-118">For more information, see [Automatic HTTP 400 responses](xref:web-api/index#automatic-http-400-responses).</span></span>

## <a name="rerun-validation"></a><span data-ttu-id="a6fe0-119">Rieseguire la convalida</span><span class="sxs-lookup"><span data-stu-id="a6fe0-119">Rerun validation</span></span>

<span data-ttu-id="a6fe0-120">La convalida è automatica, ma potrebbe essere necessario ripeterla manualmente.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-120">Validation is automatic, but you might want to repeat it manually.</span></span> <span data-ttu-id="a6fe0-121">Ad esempio, è possibile che si calcoli un valore per una proprietà e che si voglia rieseguire la convalida dopo aver impostato la proprietà sul valore calcolato.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-121">For example, you might compute a value for a property and want to rerun validation after setting the property to the computed value.</span></span> <span data-ttu-id="a6fe0-122">Per rieseguire la convalida, chiamare il metodo `TryValidateModel` come illustrato di seguito:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-122">To rerun validation, call the `TryValidateModel` method, as shown here:</span></span>

[!code-csharp[](validation/samples/3.x/ValidationSample/Pages/Movies/Create.cshtml.cs?name=snippet_TryValidate&highlight=3-6)]

## <a name="validation-attributes"></a><span data-ttu-id="a6fe0-123">Attributi di convalida</span><span class="sxs-lookup"><span data-stu-id="a6fe0-123">Validation attributes</span></span>

<span data-ttu-id="a6fe0-124">Gli attributi di convalida consentono di specificare le regole di convalida per le proprietà del modello.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-124">Validation attributes let you specify validation rules for model properties.</span></span> <span data-ttu-id="a6fe0-125">Nell'esempio seguente dell'app di esempio viene illustrata una classe di modello annotata con attributi di convalida.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-125">The following example from the sample app shows a model class that is annotated with validation attributes.</span></span> <span data-ttu-id="a6fe0-126">`[ClassicMovie]` è un attributo di convalida personalizzato, mentre gli altri sono attributi predefiniti.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-126">The `[ClassicMovie]` attribute is a custom validation attribute and the others are built-in.</span></span> <span data-ttu-id="a6fe0-127">Non viene visualizzato `[ClassicMovieWithClientValidator]` .</span><span class="sxs-lookup"><span data-stu-id="a6fe0-127">Not shown is `[ClassicMovieWithClientValidator]`.</span></span> <span data-ttu-id="a6fe0-128">`[ClassicMovieWithClientValidator]` Mostra un metodo alternativo per implementare un attributo personalizzato.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-128">`[ClassicMovieWithClientValidator]` shows an alternative way to implement a custom attribute.</span></span>

[!code-csharp[](validation/samples/3.x/ValidationSample/Models/Movie.cs?name=snippet_Class)]

## <a name="built-in-attributes"></a><span data-ttu-id="a6fe0-129">Attributi predefiniti</span><span class="sxs-lookup"><span data-stu-id="a6fe0-129">Built-in attributes</span></span>

<span data-ttu-id="a6fe0-130">Di seguito sono elencati alcuni degli attributi di convalida predefiniti:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-130">Here are some of the built-in validation attributes:</span></span>

* <span data-ttu-id="a6fe0-131">`[CreditCard]`: Convalida che la proprietà abbia un formato di carta di credito.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-131">`[CreditCard]`: Validates that the property has a credit card format.</span></span> <span data-ttu-id="a6fe0-132">Richiede [metodi aggiuntivi di convalida jQuery](https://cdnjs.cloudflare.com/ajax/libs/jquery-validate/1.19.1/additional-methods.min.js).</span><span class="sxs-lookup"><span data-stu-id="a6fe0-132">Requires [jQuery Validation Additional Methods](https://cdnjs.cloudflare.com/ajax/libs/jquery-validate/1.19.1/additional-methods.min.js).</span></span>
* <span data-ttu-id="a6fe0-133">`[Compare]`: Verifica che due proprietà di un modello corrispondano.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-133">`[Compare]`: Validates that two properties in a model match.</span></span>
* <span data-ttu-id="a6fe0-134">`[EmailAddress]`: Convalida che la proprietà abbia un formato di posta elettronica.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-134">`[EmailAddress]`: Validates that the property has an email format.</span></span>
* <span data-ttu-id="a6fe0-135">`[Phone]`: Convalida che la proprietà abbia un formato di numero di telefono.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-135">`[Phone]`: Validates that the property has a telephone number format.</span></span>
* <span data-ttu-id="a6fe0-136">`[Range]`: Convalida che il valore della proprietà rientra in un intervallo specificato.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-136">`[Range]`: Validates that the property value falls within a specified range.</span></span>
* <span data-ttu-id="a6fe0-137">`[RegularExpression]`: Verifica che il valore della proprietà corrisponda a un'espressione regolare specificata.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-137">`[RegularExpression]`: Validates that the property value matches a specified regular expression.</span></span>
* <span data-ttu-id="a6fe0-138">`[Required]`: Verifica che il campo non sia null.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-138">`[Required]`: Validates that the field is not null.</span></span> <span data-ttu-id="a6fe0-139">Per informazioni dettagliate sul comportamento di questo attributo, vedere [ `[Required]` attributo](#required-attribute) .</span><span class="sxs-lookup"><span data-stu-id="a6fe0-139">See [`[Required]` attribute](#required-attribute) for details about this attribute's behavior.</span></span>
* <span data-ttu-id="a6fe0-140">`[StringLength]`: Verifica che un valore della proprietà stringa non superi il limite di lunghezza specificato.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-140">`[StringLength]`: Validates that a string property value doesn't exceed a specified length limit.</span></span>
* <span data-ttu-id="a6fe0-141">`[Url]`: Convalida che la proprietà abbia un formato URL.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-141">`[Url]`: Validates that the property has a URL format.</span></span>
* <span data-ttu-id="a6fe0-142">`[Remote]`: Convalida l'input sul client chiamando un metodo di azione sul server.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-142">`[Remote]`: Validates input on the client by calling an action method on the server.</span></span> <span data-ttu-id="a6fe0-143">Per informazioni dettagliate sul comportamento di questo attributo, vedere [ `[Remote]` attributo](#remote-attribute) .</span><span class="sxs-lookup"><span data-stu-id="a6fe0-143">See [`[Remote]` attribute](#remote-attribute) for details about this attribute's behavior.</span></span>

<span data-ttu-id="a6fe0-144">Nello spazio dei nomi [System.ComponentModel.DataAnnotations](xref:System.ComponentModel.DataAnnotations) è possibile trovare un elenco completo degli attributi di convalida.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-144">A complete list of validation attributes can be found in the [System.ComponentModel.DataAnnotations](xref:System.ComponentModel.DataAnnotations) namespace.</span></span>

### <a name="error-messages"></a><span data-ttu-id="a6fe0-145">messaggi di errore</span><span class="sxs-lookup"><span data-stu-id="a6fe0-145">Error messages</span></span>

<span data-ttu-id="a6fe0-146">Gli attributi di convalida consentono di specificare il messaggio di errore da visualizzare in caso di input non valido.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-146">Validation attributes let you specify the error message to be displayed for invalid input.</span></span> <span data-ttu-id="a6fe0-147">Ad esempio:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-147">For example:</span></span>

```csharp
[StringLength(8, ErrorMessage = "Name length can't be more than 8.")]
```

<span data-ttu-id="a6fe0-148">Internamente gli attributi chiamano `String.Format` con un segnaposto per il nome campo e talvolta con segnaposto aggiuntivi.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-148">Internally, the attributes call `String.Format` with a placeholder for the field name and sometimes additional placeholders.</span></span> <span data-ttu-id="a6fe0-149">Ad esempio:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-149">For example:</span></span>

```csharp
[StringLength(8, ErrorMessage = "{0} length must be between {2} and {1}.", MinimumLength = 6)]
```

<span data-ttu-id="a6fe0-150">Quando applicato a una proprietà `Name`, il messaggio di errore creato con il codice precedente sarà "La lunghezza del nome utente deve essere compresa tra 6 e 8".</span><span class="sxs-lookup"><span data-stu-id="a6fe0-150">When applied to a `Name` property, the error message created by the preceding code would be "Name length must be between 6 and 8.".</span></span>

<span data-ttu-id="a6fe0-151">Per scoprire i parametri passati a `String.Format` per un messaggio di errore dell'attributo specifico, vedere il [codice sorgente DataAnnotations](https://github.com/dotnet/runtime/tree/main/src/libraries/System.ComponentModel.Annotations/src/System/ComponentModel/DataAnnotations).</span><span class="sxs-lookup"><span data-stu-id="a6fe0-151">To find out which parameters are passed to `String.Format` for a particular attribute's error message, see the [DataAnnotations source code](https://github.com/dotnet/runtime/tree/main/src/libraries/System.ComponentModel.Annotations/src/System/ComponentModel/DataAnnotations).</span></span>

## <a name="non-nullable-reference-types-and-required-attribute"></a><span data-ttu-id="a6fe0-152">Tipi di riferimento non nullable e attributo [Required]</span><span class="sxs-lookup"><span data-stu-id="a6fe0-152">Non-nullable reference types and [Required] attribute</span></span>

<span data-ttu-id="a6fe0-153">Il sistema di convalida considera i parametri non nullable o le proprietà associata come se disponesse di un `[Required]` attributo.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-153">The validation system treats non-nullable parameters or bound properties as if they had a `[Required]` attribute.</span></span> <span data-ttu-id="a6fe0-154">Se si [abilitano i `Nullable` contesti](/dotnet/csharp/nullable-references#nullable-contexts), MVC avvia in modo implicito la convalida di proprietà o parametri non nullable come se fossero stati attribuiti con l' `[Required]` attributo.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-154">By [enabling `Nullable` contexts](/dotnet/csharp/nullable-references#nullable-contexts), MVC implicitly starts validating non-nullable properties or parameters as if they had been attributed with the `[Required]` attribute.</span></span> <span data-ttu-id="a6fe0-155">Osservare il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-155">Consider the following code:</span></span>

```csharp
public class Person
{
    public string Name { get; set; }
}
```

<span data-ttu-id="a6fe0-156">Se l'app è stata compilata con `<Nullable>enable</Nullable>` , un valore mancante per `Name` in JSON o post del modulo genera un errore di convalida.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-156">If the app was built with `<Nullable>enable</Nullable>`, a missing value for `Name` in a JSON or form post results in a validation error.</span></span> <span data-ttu-id="a6fe0-157">Usare un tipo di riferimento Nullable per consentire la specifica di valori null o mancanti per la `Name` proprietà:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-157">Use a nullable reference type to allow null or missing values to be specified for the `Name` property:</span></span>

```csharp
public class Person
{
    public string? Name { get; set; }
}
```

<span data-ttu-id="a6fe0-158">.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-158">.</span></span> <span data-ttu-id="a6fe0-159">Questo comportamento può essere disabilitato configurando <xref:Microsoft.AspNetCore.Mvc.MvcOptions.SuppressImplicitRequiredAttributeForNonNullableReferenceTypes> in `Startup.ConfigureServices` :</span><span class="sxs-lookup"><span data-stu-id="a6fe0-159">This behavior can be disabled by configuring <xref:Microsoft.AspNetCore.Mvc.MvcOptions.SuppressImplicitRequiredAttributeForNonNullableReferenceTypes> in `Startup.ConfigureServices`:</span></span>

```csharp
services.AddControllers(options => options.SuppressImplicitRequiredAttributeForNonNullableReferenceTypes = true);
```

### <a name="required-validation-on-the-server"></a><span data-ttu-id="a6fe0-160">Convalida dell'attributo [Required] nel server</span><span class="sxs-lookup"><span data-stu-id="a6fe0-160">[Required] validation on the server</span></span>

<span data-ttu-id="a6fe0-161">Nel server un valore obbligatorio viene considerato mancante se la proprietà è Null.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-161">On the server, a required value is considered missing if the property is null.</span></span> <span data-ttu-id="a6fe0-162">Un campo che non ammette i valori null è sempre valido e il `[Required]` messaggio di errore dell'attributo non viene mai visualizzato.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-162">A non-nullable field is always valid, and the `[Required]` attribute's error message is never displayed.</span></span>

<span data-ttu-id="a6fe0-163">Può però succedere che l'associazione di modelli per una proprietà non nullable abbia esito negativo, generando un messaggio di errore, ad esempio `The value '' is invalid`.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-163">However, model binding for a non-nullable property may fail, resulting in an error message such as `The value '' is invalid`.</span></span> <span data-ttu-id="a6fe0-164">Per specificare un messaggio di errore personalizzato per la convalida lato server di tipi non nullable, sono disponibili le opzioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-164">To specify a custom error message for server-side validation of non-nullable types, you have the following options:</span></span>

* <span data-ttu-id="a6fe0-165">Il campo deve ammettere i valori Null, ad esempio `decimal?` anziché `decimal`.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-165">Make the field nullable (for example, `decimal?` instead of `decimal`).</span></span> <span data-ttu-id="a6fe0-166">[Nullable \<T> ](/dotnet/csharp/programming-guide/nullable-types/) i tipi di valore vengono trattati come tipi nullable standard.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-166">[Nullable\<T>](/dotnet/csharp/programming-guide/nullable-types/) value types are treated like standard nullable types.</span></span>
* <span data-ttu-id="a6fe0-167">Specificare il messaggio di errore predefinito che l'associazione di modelli deve usare, come illustrato nell'esempio seguente:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-167">Specify the default error message to be used by model binding, as shown in the following example:</span></span>

  [!code-csharp[](validation/samples/3.x/ValidationSample/Startup.cs?name=snippet_Configuration&highlight=5-6)]

  <span data-ttu-id="a6fe0-168">Per altre informazioni sugli errori dell'associazione di modelli che possono essere impostati come messaggi predefiniti, vedere <xref:Microsoft.AspNetCore.Mvc.ModelBinding.Metadata.DefaultModelBindingMessageProvider#methods>.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-168">For more information about model binding errors that you can set default messages for, see <xref:Microsoft.AspNetCore.Mvc.ModelBinding.Metadata.DefaultModelBindingMessageProvider#methods>.</span></span>

### <a name="required-validation-on-the-client"></a><span data-ttu-id="a6fe0-169">Convalida dell'attributo [Required] nel client</span><span class="sxs-lookup"><span data-stu-id="a6fe0-169">[Required] validation on the client</span></span>

<span data-ttu-id="a6fe0-170">Le stringhe e i tipi non nullable sono gestiti in modo diverso nel client rispetto a come vengono gestiti nel server.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-170">Non-nullable types and strings are handled differently on the client compared to the server.</span></span> <span data-ttu-id="a6fe0-171">Nel client:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-171">On the client:</span></span>

* <span data-ttu-id="a6fe0-172">Un valore viene considerato presente solo viene immesso un input per tale valore.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-172">A value is considered present only if input is entered for it.</span></span> <span data-ttu-id="a6fe0-173">La convalida lato client gestisce quindi i tipi non nullable come i tipi nullable.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-173">Therefore, client-side validation handles non-nullable types the same as nullable types.</span></span>
* <span data-ttu-id="a6fe0-174">Lo spazio vuoto in un campo stringa viene considerato un input valido per il metodo [required](https://jqueryvalidation.org/required-method/) della convalida di jQuery.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-174">Whitespace in a string field is considered valid input by the jQuery Validation [required](https://jqueryvalidation.org/required-method/) method.</span></span> <span data-ttu-id="a6fe0-175">La convalida lato server considera un campo stringa obbligatorio non valido solo se viene immesso uno spazio vuoto.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-175">Server-side validation considers a required string field invalid if only whitespace is entered.</span></span>

<span data-ttu-id="a6fe0-176">Come indicato in precedenza, i tipi non nullable vengono trattati come se avessero un attributo `[Required]`.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-176">As noted earlier, non-nullable types are treated as though they had a `[Required]` attribute.</span></span> <span data-ttu-id="a6fe0-177">Quindi viene eseguita la convalida lato client anche se non si applica l'attributo `[Required]`.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-177">That means you get client-side validation even if you don't apply the `[Required]` attribute.</span></span> <span data-ttu-id="a6fe0-178">Se invece non si usa l'attributo, viene visualizzato un messaggio di errore predefinito.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-178">But if you don't use the attribute, you get a default error message.</span></span> <span data-ttu-id="a6fe0-179">Per specificare un messaggio di errore personalizzato, usare l'attributo.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-179">To specify a custom error message, use the attribute.</span></span>

## <a name="remote-attribute"></a><span data-ttu-id="a6fe0-180">Attributo [Remote]</span><span class="sxs-lookup"><span data-stu-id="a6fe0-180">[Remote] attribute</span></span>

<span data-ttu-id="a6fe0-181">L'attributo `[Remote]` implementa la convalida lato client che richiede la chiamata a un metodo nel server per determinare se l'input del campo è valido.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-181">The `[Remote]` attribute implements client-side validation that requires calling a method on the server to determine whether field input is valid.</span></span> <span data-ttu-id="a6fe0-182">Ad esempio, l'app potrebbe dover verificare se un nome utente è già in uso.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-182">For example, the app may need to verify whether a user name is already in use.</span></span>

<span data-ttu-id="a6fe0-183">Per implementare la convalida remota:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-183">To implement remote validation:</span></span>

1. <span data-ttu-id="a6fe0-184">Creare un metodo di azione per JavaScript da chiamare.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-184">Create an action method for JavaScript to call.</span></span>  <span data-ttu-id="a6fe0-185">Il metodo [remoto](https://jqueryvalidation.org/remote-method/) di convalida jQuery prevede una risposta JSON:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-185">The jQuery Validation [remote](https://jqueryvalidation.org/remote-method/) method expects a JSON response:</span></span>

   * <span data-ttu-id="a6fe0-186">`true` indica che i dati di input sono validi.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-186">`true` means the input data is valid.</span></span>
   * <span data-ttu-id="a6fe0-187">`false`, `undefined` o `null` indica che l'input non è valido.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-187">`false`, `undefined`, or `null` means the input is invalid.</span></span> <span data-ttu-id="a6fe0-188">Visualizzare il messaggio di errore predefinito.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-188">Display the default error message.</span></span>
   * <span data-ttu-id="a6fe0-189">Qualsiasi altra stringa indica che l'input non è valido.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-189">Any other string means the input is invalid.</span></span> <span data-ttu-id="a6fe0-190">Visualizzare la stringa come messaggio di errore personalizzato.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-190">Display the string as a custom error message.</span></span>

   <span data-ttu-id="a6fe0-191">Di seguito è riportato un esempio di un metodo di azione che restituisce un messaggio di errore personalizzato:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-191">Here's an example of an action method that returns a custom error message:</span></span>

   [!code-csharp[](validation/samples/3.x/ValidationSample/Controllers/UsersController.cs?name=snippet_VerifyEmail)]

1. <span data-ttu-id="a6fe0-192">Nella classe del modello annotare la proprietà con un attributo `[Remote]` che punta al metodo di azione di convalida, come illustrato nell'esempio seguente:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-192">In the model class, annotate the property with a `[Remote]` attribute that points to the validation action method, as shown in the following example:</span></span>

   [!code-csharp[](validation/samples/3.x/ValidationSample/Models/User.cs?name=snippet_Email)]
 
   <span data-ttu-id="a6fe0-193">L'attributo `[Remote]` si trova nello spazio dei nomi `Microsoft.AspNetCore.Mvc`.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-193">The `[Remote]` attribute is in the `Microsoft.AspNetCore.Mvc` namespace.</span></span>
   
### <a name="additional-fields"></a><span data-ttu-id="a6fe0-194">Altri campi</span><span class="sxs-lookup"><span data-stu-id="a6fe0-194">Additional fields</span></span>

<span data-ttu-id="a6fe0-195">La proprietà `[Remote]` dell'attributo `AdditionalFields` consente di convalidare combinazioni di campi rispetto ai dati nel server.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-195">The `AdditionalFields` property of the `[Remote]` attribute lets you validate combinations of fields against data on the server.</span></span> <span data-ttu-id="a6fe0-196">Ad esempio, se il modello `User` avesse le proprietà `FirstName` e `LastName`, potrebbe essere necessario controllare che non siano già esistenti utenti con la stessa coppia di nomi.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-196">For example, if the `User` model had `FirstName` and `LastName` properties, you might want to verify that no existing users already have that pair of names.</span></span> <span data-ttu-id="a6fe0-197">Nell'esempio riportato di seguito viene illustrato come usare `AdditionalFields`:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-197">The following example shows how to use `AdditionalFields`:</span></span>

[!code-csharp[](validation/samples/3.x/ValidationSample/Models/User.cs?name=snippet_Name&highlight=1,5)]

<span data-ttu-id="a6fe0-198">`AdditionalFields` potrebbe essere impostato in modo esplicito sulle stringhe "FirstName" e "LastName", ma l'uso dell'operatore [NameOf](/dotnet/csharp/language-reference/keywords/nameof) semplifica il refactoring in un secondo momento.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-198">`AdditionalFields` could be set explicitly to the strings "FirstName" and "LastName", but using the [nameof](/dotnet/csharp/language-reference/keywords/nameof) operator simplifies later refactoring.</span></span> <span data-ttu-id="a6fe0-199">Il metodo di azione per questa convalida deve accettare `firstName` gli `lastName` argomenti e:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-199">The action method for this validation must accept both `firstName` and `lastName` arguments:</span></span>

[!code-csharp[](validation/samples/3.x/ValidationSample/Controllers/UsersController.cs?name=snippet_VerifyName)]

<span data-ttu-id="a6fe0-200">Quando si immette un nome o un cognome, JavaScript esegue una chiamata remota per verificare se tale coppia di nomi è in uso.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-200">When the user enters a first or last name, JavaScript makes a remote call to see if that pair of names has been taken.</span></span>

<span data-ttu-id="a6fe0-201">Per convalidare due o più campi aggiuntivi, specificarli sotto forma di elenco delimitato da virgole.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-201">To validate two or more additional fields, provide them as a comma-delimited list.</span></span> <span data-ttu-id="a6fe0-202">Ad esempio, per aggiungere una proprietà `MiddleName` al modello, impostare l'attributo `[Remote]` come illustrato nell'esempio seguente:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-202">For example, to add a `MiddleName` property to the model, set the `[Remote]` attribute as shown in the following example:</span></span>

```csharp
[Remote(action: "VerifyName", controller: "Users", AdditionalFields = nameof(FirstName) + "," + nameof(LastName))]
public string MiddleName { get; set; }
```

<span data-ttu-id="a6fe0-203">`AdditionalFields`, come tutti gli argomenti dell'attributo, deve essere un'espressione costante.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-203">`AdditionalFields`, like all attribute arguments, must be a constant expression.</span></span> <span data-ttu-id="a6fe0-204">Non usare quindi una [stringa interpolata](/dotnet/csharp/language-reference/keywords/interpolated-strings) oppure chiamare <xref:System.String.Join*> per inizializzare `AdditionalFields`.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-204">Therefore, don't use an [interpolated string](/dotnet/csharp/language-reference/keywords/interpolated-strings) or call <xref:System.String.Join*> to initialize `AdditionalFields`.</span></span>

## <a name="alternatives-to-built-in-attributes"></a><span data-ttu-id="a6fe0-205">Alternative agli attributi predefiniti</span><span class="sxs-lookup"><span data-stu-id="a6fe0-205">Alternatives to built-in attributes</span></span>

<span data-ttu-id="a6fe0-206">Se si vuole usare un tipo di convalida non definita da attributi predefiniti, è possibile eseguire le operazioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-206">If you need validation not provided by built-in attributes, you can:</span></span>

* <span data-ttu-id="a6fe0-207">[Creare attributi personalizzati](#custom-attributes).</span><span class="sxs-lookup"><span data-stu-id="a6fe0-207">[Create custom attributes](#custom-attributes).</span></span>
* <span data-ttu-id="a6fe0-208">[Implementare IValidatableObject](#ivalidatableobject).</span><span class="sxs-lookup"><span data-stu-id="a6fe0-208">[Implement IValidatableObject](#ivalidatableobject).</span></span>

## <a name="custom-attributes"></a><span data-ttu-id="a6fe0-209">Attributi personalizzati</span><span class="sxs-lookup"><span data-stu-id="a6fe0-209">Custom attributes</span></span>

<span data-ttu-id="a6fe0-210">Per gli scenari non gestiti dagli attributi di convalida predefiniti, è possibile creare attributi di convalida personalizzati.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-210">For scenarios that the built-in validation attributes don't handle, you can create custom validation attributes.</span></span> <span data-ttu-id="a6fe0-211">Creare una classe che eredita da <xref:System.ComponentModel.DataAnnotations.ValidationAttribute> ed eseguire l'override del metodo <xref:System.ComponentModel.DataAnnotations.ValidationAttribute.IsValid*>.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-211">Create a class that inherits from <xref:System.ComponentModel.DataAnnotations.ValidationAttribute>, and override the <xref:System.ComponentModel.DataAnnotations.ValidationAttribute.IsValid*> method.</span></span>

<span data-ttu-id="a6fe0-212">Il metodo `IsValid` accetta un oggetto denominato *value*, ovvero l'input da convalidare.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-212">The `IsValid` method accepts an object named *value*, which is the input to be validated.</span></span> <span data-ttu-id="a6fe0-213">Un overload accetta anche un oggetto `ValidationContext`, che contiene informazioni aggiuntive, ad esempio l'istanza del modello creato dall'associazione di modelli.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-213">An overload also accepts a `ValidationContext` object, which provides additional information, such as the model instance created by model binding.</span></span>

<span data-ttu-id="a6fe0-214">Nell'esempio seguente si convalida che la data di uscita di un film di genere *Classic* non sia successiva a un anno specifico.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-214">The following example validates that the release date for a movie in the *Classic* genre isn't later than a specified year.</span></span> <span data-ttu-id="a6fe0-215">`[ClassicMovie]`Attributo:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-215">The `[ClassicMovie]` attribute:</span></span>

* <span data-ttu-id="a6fe0-216">Viene eseguito solo sul server.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-216">Is only run on the server.</span></span>
* <span data-ttu-id="a6fe0-217">Per i filmati classici, convalida la data di rilascio:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-217">For Classic movies, validates the release date:</span></span>

[!code-csharp[](validation/samples/3.x/ValidationSample/Validation/ClassicMovieAttribute.cs?name=snippet_Class)]

<span data-ttu-id="a6fe0-218">La variabile `movie` nell'esempio precedente rappresenta un oggetto `Movie` contenente i dati dell'invio del modulo.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-218">The `movie` variable in the preceding example represents a `Movie` object that contains the data from the form submission.</span></span> <span data-ttu-id="a6fe0-219">Se la convalida ha esito negativo, viene restituito un codice `ValidationResult` con un messaggio di errore.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-219">When validation fails, a `ValidationResult` with an error message is returned.</span></span>

## <a name="ivalidatableobject"></a><span data-ttu-id="a6fe0-220">IValidatableObject</span><span class="sxs-lookup"><span data-stu-id="a6fe0-220">IValidatableObject</span></span>

<span data-ttu-id="a6fe0-221">L'esempio precedente usa solo tipi `Movie`.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-221">The preceding example works only with `Movie` types.</span></span> <span data-ttu-id="a6fe0-222">Un'altra opzione per la convalida a livello di classe consiste nell'implementare `IValidatableObject` nella classe del modello, come illustrato nell'esempio seguente:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-222">Another option for class-level validation is to implement `IValidatableObject` in the model class, as shown in the following example:</span></span>

[!code-csharp[](validation/samples/3.x/ValidationSample/Models/ValidatableMovie.cs?name=snippet_Class&highlight=1,26-34)]

## <a name="top-level-node-validation"></a><span data-ttu-id="a6fe0-223">Convalida del nodo di primo livello</span><span class="sxs-lookup"><span data-stu-id="a6fe0-223">Top-level node validation</span></span>

<span data-ttu-id="a6fe0-224">I nodi di primo livello includono:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-224">Top-level nodes include:</span></span>

* <span data-ttu-id="a6fe0-225">Parametri di azione</span><span class="sxs-lookup"><span data-stu-id="a6fe0-225">Action parameters</span></span>
* <span data-ttu-id="a6fe0-226">Proprietà del controller</span><span class="sxs-lookup"><span data-stu-id="a6fe0-226">Controller properties</span></span>
* <span data-ttu-id="a6fe0-227">Parametri del gestore di pagina</span><span class="sxs-lookup"><span data-stu-id="a6fe0-227">Page handler parameters</span></span>
* <span data-ttu-id="a6fe0-228">Proprietà del modello di pagina</span><span class="sxs-lookup"><span data-stu-id="a6fe0-228">Page model properties</span></span>

<span data-ttu-id="a6fe0-229">Vengono convalidati i nodi di primo livello associati al modello, oltre a convalidare le proprietà del modello.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-229">Model-bound top-level nodes are validated in addition to validating model properties.</span></span> <span data-ttu-id="a6fe0-230">Nell'esempio seguente dell'app di esempio il metodo `VerifyPhone` usa <xref:System.ComponentModel.DataAnnotations.RegularExpressionAttribute> per convalidare il parametro di azione `phone`:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-230">In the following example from the sample app, the `VerifyPhone` method uses the <xref:System.ComponentModel.DataAnnotations.RegularExpressionAttribute> to validate the `phone` action parameter:</span></span>

[!code-csharp[](validation/samples/3.x/ValidationSample/Controllers/UsersController.cs?name=snippet_VerifyPhone)]

<span data-ttu-id="a6fe0-231">I nodi di primo livello possono usare <xref:Microsoft.AspNetCore.Mvc.ModelBinding.BindRequiredAttribute> con gli attributi di convalida.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-231">Top-level nodes can use <xref:Microsoft.AspNetCore.Mvc.ModelBinding.BindRequiredAttribute> with validation attributes.</span></span> <span data-ttu-id="a6fe0-232">Nell'esempio seguente dall'app di esempio, il metodo `CheckAge` specifica che il parametro `age` deve essere associato dalla stringa di query quando viene inviato il modulo:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-232">In the following example from the sample app, the `CheckAge` method specifies that the `age` parameter must be bound from the query string when the form is submitted:</span></span>

[!code-csharp[](validation/samples/3.x/ValidationSample/Controllers/UsersController.cs?name=snippet_CheckAgeSignature)]

<span data-ttu-id="a6fe0-233">La pagina Check Age (*CheckAge.cshtml*), include due moduli.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-233">In the Check Age page (*CheckAge.cshtml*), there are two forms.</span></span> <span data-ttu-id="a6fe0-234">Il primo form invia un `Age` valore di `99` come parametro della stringa di query: `https://localhost:5001/Users/CheckAge?Age=99` .</span><span class="sxs-lookup"><span data-stu-id="a6fe0-234">The first form submits an `Age` value of `99` as a query string parameter: `https://localhost:5001/Users/CheckAge?Age=99`.</span></span>

<span data-ttu-id="a6fe0-235">Quando viene inviato un parametro correttamente formattato `age` dalla stringa di query, il modulo viene convalidato.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-235">When a properly formatted `age` parameter from the query string is submitted, the form validates.</span></span>

<span data-ttu-id="a6fe0-236">Il secondo modulo nella pagina Check Age invia il valore `Age` nel corpo della richiesta e la convalida ha esito negativo.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-236">The second form on the Check Age page submits the `Age` value in the body of the request, and validation fails.</span></span> <span data-ttu-id="a6fe0-237">L'associazione non riesce perché il parametro `age` deve provenire da una stringa di query.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-237">Binding fails because the `age` parameter must come from a query string.</span></span>

## <a name="maximum-errors"></a><span data-ttu-id="a6fe0-238">Numero massimo di errori</span><span class="sxs-lookup"><span data-stu-id="a6fe0-238">Maximum errors</span></span>

<span data-ttu-id="a6fe0-239">La convalida si interrompe quando viene raggiunto il numero di errori (200 per impostazione predefinita).</span><span class="sxs-lookup"><span data-stu-id="a6fe0-239">Validation stops when the maximum number of errors is reached (200 by default).</span></span> <span data-ttu-id="a6fe0-240">È possibile configurare questo numero con il codice seguente in `Startup.ConfigureServices`:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-240">You can configure this number with the following code in `Startup.ConfigureServices`:</span></span>

[!code-csharp[](validation/samples/3.x/ValidationSample/Startup.cs?name=snippet_Configuration&highlight=4)]

## <a name="maximum-recursion"></a><span data-ttu-id="a6fe0-241">Numero massimo di ricorsioni</span><span class="sxs-lookup"><span data-stu-id="a6fe0-241">Maximum recursion</span></span>

<span data-ttu-id="a6fe0-242"><xref:Microsoft.AspNetCore.Mvc.ModelBinding.Validation.ValidationVisitor> attraversare l'oggetto grafico del modello che deve essere convalidato.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-242"><xref:Microsoft.AspNetCore.Mvc.ModelBinding.Validation.ValidationVisitor> traverses the object graph of the model being validated.</span></span> <span data-ttu-id="a6fe0-243">Per i modelli che sono profondi o sono infinitamente ricorsivi, la convalida può causare un overflow dello stack.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-243">For models that are deep or are infinitely recursive, validation may result in stack overflow.</span></span> <span data-ttu-id="a6fe0-244">[MvcOptions.MaxValidationDepth](xref:Microsoft.AspNetCore.Mvc.MvcOptions.MaxValidationDepth) consente di arrestare tempestivamente la convalida se la ricorsione del visitatore supera la profondità configurata.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-244">[MvcOptions.MaxValidationDepth](xref:Microsoft.AspNetCore.Mvc.MvcOptions.MaxValidationDepth) provides a way to stop validation early if the visitor recursion exceeds a configured depth.</span></span> <span data-ttu-id="a6fe0-245">Il valore predefinito di `MvcOptions.MaxValidationDepth` è 32.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-245">The default value of `MvcOptions.MaxValidationDepth` is 32.</span></span>

## <a name="automatic-short-circuit"></a><span data-ttu-id="a6fe0-246">Corto circuito automatico</span><span class="sxs-lookup"><span data-stu-id="a6fe0-246">Automatic short-circuit</span></span>

<span data-ttu-id="a6fe0-247">La convalida viene ignorata automaticamente (corto circuito) se l'oggetto grafico non richiede la convalida.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-247">Validation is automatically short-circuited (skipped) if the model graph doesn't require validation.</span></span> <span data-ttu-id="a6fe0-248">Gli oggetti per cui viene ignorata la convalida sono le raccolte di primitive (ad esempio `byte[]`, `string[]`, `Dictionary<string, string>`) e gli oggetti grafici complessi che non hanno validator.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-248">Objects that the runtime skips validation for include collections of primitives (such as `byte[]`, `string[]`, `Dictionary<string, string>`) and complex object graphs that don't have any validators.</span></span>

## <a name="disable-validation"></a><span data-ttu-id="a6fe0-249">Disabilitare la convalida</span><span class="sxs-lookup"><span data-stu-id="a6fe0-249">Disable validation</span></span>

<span data-ttu-id="a6fe0-250">Per disabilitare la convalida:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-250">To disable validation:</span></span>

1. <span data-ttu-id="a6fe0-251">Creare un'implementazione di `IObjectModelValidator` che non contrassegna i campi come non validi.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-251">Create an implementation of `IObjectModelValidator` that doesn't mark any fields as invalid.</span></span>

   [!code-csharp[](validation/samples/3.x/ValidationSample/Validation/NullObjectModelValidator.cs?name=snippet_Class)]

1. <span data-ttu-id="a6fe0-252">Aggiungere il codice seguente a `Startup.ConfigureServices` per sostituire l'impostazione `IObjectModelValidator` predefinita nel contenitore di inserimento delle dipendenze.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-252">Add the following code to `Startup.ConfigureServices` to replace the default `IObjectModelValidator` implementation in the dependency injection container.</span></span>

   [!code-csharp[](validation/samples/3.x/ValidationSample/Startup.cs?name=snippet_DisableValidation)]

<span data-ttu-id="a6fe0-253">Potrebbero essere visualizzati errori di stato del modello generati dall'associazione di modelli.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-253">You might still see model state errors that originate from model binding.</span></span>

## <a name="client-side-validation"></a><span data-ttu-id="a6fe0-254">Convalida lato client</span><span class="sxs-lookup"><span data-stu-id="a6fe0-254">Client-side validation</span></span>

<span data-ttu-id="a6fe0-255">La convalida lato client non consente l'invio del modulo finché non è ritenuto valido.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-255">Client-side validation prevents submission until the form is valid.</span></span> <span data-ttu-id="a6fe0-256">Tramite il pulsante Invia viene eseguito JavaScript, che invia il modulo oppure visualizza i messaggi di errore.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-256">The Submit button runs JavaScript that either submits the form or displays error messages.</span></span>

<span data-ttu-id="a6fe0-257">La convalida lato client evitare un inutile round trip al server in caso di errori di input in un modulo.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-257">Client-side validation avoids an unnecessary round trip to the server when there are input errors on a form.</span></span> <span data-ttu-id="a6fe0-258">I riferimenti dello script seguenti in *_Layout.cshtml* e *_ValidationScriptsPartial.cshtml* supportano la convalida lato client:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-258">The following script references in *_Layout.cshtml* and *_ValidationScriptsPartial.cshtml* support client-side validation:</span></span>

[!code-cshtml[](validation/samples/3.x/ValidationSample/Views/Shared/_Layout.cshtml?name=snippet_Scripts)]

[!code-cshtml[](validation/samples/3.x/ValidationSample/Views/Shared/_ValidationScriptsPartial.cshtml?name=snippet_Scripts)]

<span data-ttu-id="a6fe0-259">Lo script di [convalida di jQuery non intrusivo](https://github.com/aspnet/jquery-validation-unobtrusive) è una libreria front-end Microsoft personalizzata basata sul famoso plug-in di [convalida jQuery](https://jqueryvalidation.org/) .</span><span class="sxs-lookup"><span data-stu-id="a6fe0-259">The [jQuery Unobtrusive Validation](https://github.com/aspnet/jquery-validation-unobtrusive) script is a custom Microsoft front-end library that builds on the popular [jQuery Validation](https://jqueryvalidation.org/) plugin.</span></span> <span data-ttu-id="a6fe0-260">Senza Query Unobtrusive Validation, sarebbe necessario scrivere il codice della stessa logica di convalida in due posizioni, vale a dire negli attributi di convalida lato server nelle proprietà del modello e nuovamente negli script lato client.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-260">Without jQuery Unobtrusive Validation, you would have to code the same validation logic in two places: once in the server-side validation attributes on model properties, and then again in client-side scripts.</span></span> <span data-ttu-id="a6fe0-261">Invece, gli [helper tag](xref:mvc/views/tag-helpers/intro)e agli [helper HTML](xref:mvc/views/overview) usano gli attributi di convalida e i metadati di tipo delle proprietà del modello per eseguire il rendering degli attributi `data-` HTML 5 negli elementi del modulo che devono essere convalidati.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-261">Instead, [Tag Helpers](xref:mvc/views/tag-helpers/intro) and [HTML helpers](xref:mvc/views/overview) use the validation attributes and type metadata from model properties to render HTML 5 `data-` attributes for the form elements that need validation.</span></span> <span data-ttu-id="a6fe0-262">jQuery uninvaded Validation analizza gli `data-` attributi e passa la logica alla convalida jQuery, "copiando in modo efficace la logica di convalida lato server nel client.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-262">jQuery Unobtrusive Validation parses the `data-` attributes and passes the logic to jQuery Validation, effectively "copying" the server-side validation logic to the client.</span></span> <span data-ttu-id="a6fe0-263">È possibile visualizzare gli errori di convalida nel client tramite gli helper tag, come illustrato di seguito:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-263">You can display validation errors on the client using tag helpers as shown here:</span></span>

[!code-cshtml[](validation/samples/3.x/ValidationSample/Pages/Movies/Create.cshtml?name=snippet_ReleaseDate&highlight=3-4)]

<span data-ttu-id="a6fe0-264">Gli helper tag precedenti eseguono il rendering del codice HTML seguente:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-264">The preceding tag helpers render the following HTML:</span></span>

```html
<div class="form-group">
    <label class="control-label" for="Movie_ReleaseDate">Release Date</label>
    <input class="form-control" type="date" data-val="true"
        data-val-required="The Release Date field is required."
        id="Movie_ReleaseDate" name="Movie.ReleaseDate" value="">
    <span class="text-danger field-validation-valid"
        data-valmsg-for="Movie.ReleaseDate" data-valmsg-replace="true"></span>
</div>
```

<span data-ttu-id="a6fe0-265">Si noti che gli attributi `data-` nell'output HTML corrispondono agli attributi di convalida per la proprietà `Movie.ReleaseDate`.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-265">Notice that the `data-` attributes in the HTML output correspond to the validation attributes for the `Movie.ReleaseDate` property.</span></span> <span data-ttu-id="a6fe0-266">L'attributo `data-val-required` contiene un messaggio di errore che viene visualizzato se l'utente non compila il campo relativo alla data di uscita.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-266">The `data-val-required` attribute contains an error message to display if the user doesn't fill in the release date field.</span></span> <span data-ttu-id="a6fe0-267">jQuery uninvaded Validation passa questo valore al metodo di convalida jQuery [Required ()](https://jqueryvalidation.org/required-method/) , che quindi Visualizza il messaggio nell'elemento associato **\<span>** .</span><span class="sxs-lookup"><span data-stu-id="a6fe0-267">jQuery Unobtrusive Validation passes this value to the jQuery Validation [required()](https://jqueryvalidation.org/required-method/) method, which then displays that message in the accompanying **\<span>** element.</span></span>

<span data-ttu-id="a6fe0-268">La convalida del tipo di dati si basa sul tipo .NET di una proprietà, a meno che sia sostituito dall'attributo `[DataType]`.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-268">Data type validation is based on the .NET type of a property, unless that is overridden by a `[DataType]` attribute.</span></span> <span data-ttu-id="a6fe0-269">I browser generano messaggi di errore predefiniti propri che il pacchetto jQuery Validation Unobtrusive Validation può comunque sostituire.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-269">Browsers have their own default error messages, but the jQuery Validation Unobtrusive Validation package can override those messages.</span></span> <span data-ttu-id="a6fe0-270">Gli attributi `[DataType]` e le sottoclassi, ad esempio `[EmailAddress]`, consentono di specificare il messaggio di errore.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-270">`[DataType]` attributes and subclasses such as `[EmailAddress]` let you specify the error message.</span></span>

## <a name="unobtrusive-validation"></a><span data-ttu-id="a6fe0-271">Convalida non intrusiva</span><span class="sxs-lookup"><span data-stu-id="a6fe0-271">Unobtrusive validation</span></span>

<span data-ttu-id="a6fe0-272">Per informazioni sulla convalida non intrusiva, vedere [questo problema di GitHub](https://github.com/dotnet/AspNetCore.Docs/issues/1111).</span><span class="sxs-lookup"><span data-stu-id="a6fe0-272">For information on unobtrusive validation, see [this GitHub issue](https://github.com/dotnet/AspNetCore.Docs/issues/1111).</span></span>

### <a name="add-validation-to-dynamic-forms"></a><span data-ttu-id="a6fe0-273">Aggiungere la convalida a moduli dinamici</span><span class="sxs-lookup"><span data-stu-id="a6fe0-273">Add Validation to Dynamic Forms</span></span>

<span data-ttu-id="a6fe0-274">la convalida non intrusiva jQuery passa la logica di convalida e i parametri alla convalida jQuery quando viene caricata la prima pagina.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-274">jQuery Unobtrusive Validation passes validation logic and parameters to jQuery Validation when the page first loads.</span></span> <span data-ttu-id="a6fe0-275">La convalida non viene quindi eseguita automaticamente nei moduli generati in modo dinamico.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-275">Therefore, validation doesn't work automatically on dynamically generated forms.</span></span> <span data-ttu-id="a6fe0-276">Per abilitare la convalida, chiedere a jQuery Unobtrusive Validation di analizzare il modulo dinamico immediatamente dopo essere stato creato.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-276">To enable validation, tell jQuery Unobtrusive Validation to parse the dynamic form immediately after you create it.</span></span> <span data-ttu-id="a6fe0-277">Il codice riportato di seguito configura ad esempio la convalida lato client in un modulo che è stato aggiunto tramite AJAX.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-277">For example, the following code sets up client-side validation on a form added via AJAX.</span></span>

```javascript
$.get({
    url: "https://url/that/returns/a/form",
    dataType: "html",
    error: function(jqXHR, textStatus, errorThrown) {
        alert(textStatus + ": Couldn't add form. " + errorThrown);
    },
    success: function(newFormHTML) {
        var container = document.getElementById("form-container");
        container.insertAdjacentHTML("beforeend", newFormHTML);
        var forms = container.getElementsByTagName("form");
        var newForm = forms[forms.length - 1];
        $.validator.unobtrusive.parse(newForm);
    }
})
```

<span data-ttu-id="a6fe0-278">Il metodo `$.validator.unobtrusive.parse()` accetta un selettore jQuery per ogni argomento.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-278">The `$.validator.unobtrusive.parse()` method accepts a jQuery selector for its one argument.</span></span> <span data-ttu-id="a6fe0-279">Questo metodo indica a jQuery Unobtrusive Validation di analizzare gli attributi `data-` dei moduli all'interno di tale tipo di selettore.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-279">This method tells jQuery Unobtrusive Validation to parse the `data-` attributes of forms within that selector.</span></span> <span data-ttu-id="a6fe0-280">I valori di questi attributi vengono quindi passati al plug-in di convalida jQuery.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-280">The values of those attributes are then passed to the jQuery Validation plugin.</span></span>

### <a name="add-validation-to-dynamic-controls"></a><span data-ttu-id="a6fe0-281">Aggiungere la convalida a controlli dinamici</span><span class="sxs-lookup"><span data-stu-id="a6fe0-281">Add Validation to Dynamic Controls</span></span>

<span data-ttu-id="a6fe0-282">Il metodo `$.validator.unobtrusive.parse()` funziona su un intero modulo e non sui singoli controlli generati dinamicamente, ad esempio `<input>` e `<select/>`.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-282">The `$.validator.unobtrusive.parse()` method works on an entire form, not on individual dynamically generated controls, such as `<input>` and `<select/>`.</span></span> <span data-ttu-id="a6fe0-283">Per eseguire il reparse del modulo, rimuovere i dati di convalida aggiunti quando il modulo è stato precedentemente analizzato, come illustrato nell'esempio seguente:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-283">To reparse the form, remove the validation data that was added when the form was parsed earlier, as shown in the following example:</span></span>

```javascript
$.get({
    url: "https://url/that/returns/a/control",
    dataType: "html",
    error: function(jqXHR, textStatus, errorThrown) {
        alert(textStatus + ": Couldn't add control. " + errorThrown);
    },
    success: function(newInputHTML) {
        var form = document.getElementById("my-form");
        form.insertAdjacentHTML("beforeend", newInputHTML);
        $(form).removeData("validator")    // Added by jQuery Validation
               .removeData("unobtrusiveValidation");   // Added by jQuery Unobtrusive Validation
        $.validator.unobtrusive.parse(form);
    }
})
```

## <a name="custom-client-side-validation"></a><span data-ttu-id="a6fe0-284">Convalida lato client personalizzata</span><span class="sxs-lookup"><span data-stu-id="a6fe0-284">Custom client-side validation</span></span>

<span data-ttu-id="a6fe0-285">La convalida lato client personalizzata viene eseguita generando `data-` attributi HTML che funzionano con un adapter di convalida jQuery personalizzato.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-285">Custom client-side validation is done by generating `data-` HTML attributes that work with a custom jQuery Validation adapter.</span></span> <span data-ttu-id="a6fe0-286">Il codice dell'adapter di esempio seguente è stato scritto per gli attributi `[ClassicMovie]` e `[ClassicMovieWithClientValidator]` presentati precedentemente nell'articolo:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-286">The following sample adapter code was written for the `[ClassicMovie]` and `[ClassicMovieWithClientValidator]` attributes that were introduced earlier in this article:</span></span>

[!code-javascript[](validation/samples/3.x/ValidationSample/wwwroot/js/classicMovieValidator.js)]

<span data-ttu-id="a6fe0-287">Per informazioni su come scrivere gli adapter, vedere la [documentazione relativa alla convalida jQuery](https://jqueryvalidation.org/documentation/).</span><span class="sxs-lookup"><span data-stu-id="a6fe0-287">For information about how to write adapters, see the [jQuery Validation documentation](https://jqueryvalidation.org/documentation/).</span></span>

<span data-ttu-id="a6fe0-288">L'uso di un adapter per un determinato campo viene attivato dagli attributi `data-`, i quali eseguono le operazioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-288">The use of an adapter for a given field is triggered by `data-` attributes that:</span></span>

* <span data-ttu-id="a6fe0-289">Contrassegnano il campo come campo da sottoporre a convalida (`data-val="true"`).</span><span class="sxs-lookup"><span data-stu-id="a6fe0-289">Flag the field as being subject to validation (`data-val="true"`).</span></span>
* <span data-ttu-id="a6fe0-290">Identificano un nome della regola di convalida e un testo del messaggio di errore, ad esempio `data-val-rulename="Error message."`.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-290">Identify a validation rule name and error message text (for example, `data-val-rulename="Error message."`).</span></span>
* <span data-ttu-id="a6fe0-291">Specificare i parametri aggiuntivi necessari al validator, ad esempio `data-val-rulename-param1="value"`.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-291">Provide any additional parameters the validator needs (for example, `data-val-rulename-param1="value"`).</span></span>

<span data-ttu-id="a6fe0-292">L'esempio seguente contiene gli attributi `data-` per l'attributo `ClassicMovie` dell'app di esempio:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-292">The following example shows the `data-` attributes for the sample app's `ClassicMovie` attribute:</span></span>

```html
<input class="form-control" type="date"
    data-val="true"
    data-val-classicmovie="Classic movies must have a release year no later than 1960."
    data-val-classicmovie-year="1960"
    data-val-required="The Release Date field is required."
    id="Movie_ReleaseDate" name="Movie.ReleaseDate" value="">
```

<span data-ttu-id="a6fe0-293">Come già accennato in precedenza, gli [helper tag](xref:mvc/views/tag-helpers/intro) e gli [helper HTML](xref:mvc/views/overview) usano le informazioni degli attributi di convalida per eseguire il rendering degli attributi `data-`.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-293">As noted earlier, [Tag Helpers](xref:mvc/views/tag-helpers/intro) and [HTML helpers](xref:mvc/views/overview) use information from validation attributes to render `data-` attributes.</span></span> <span data-ttu-id="a6fe0-294">Per scrivere il codice che crea attributi HTML `data-` personalizzati, sono disponibili due opzioni:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-294">There are two options for writing code that results in the creation of custom `data-` HTML attributes:</span></span>

* <span data-ttu-id="a6fe0-295">Creare una classe che deriva da `AttributeAdapterBase<TAttribute>` e una classe che implementa `IValidationAttributeAdapterProvider` e registrare l'attributo e il relativo adapter nell'inserimento delle dipendenze.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-295">Create a class that derives from `AttributeAdapterBase<TAttribute>` and a class that implements `IValidationAttributeAdapterProvider`, and register your attribute and its adapter in DI.</span></span> <span data-ttu-id="a6fe0-296">Questo metodo segue il [principio di singola responsabilità](https://wikipedia.org/wiki/Single_responsibility_principle) in quanto il codice di convalida relativo a server e client si trova in classi separate.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-296">This method follows the [single responsibility principal](https://wikipedia.org/wiki/Single_responsibility_principle) in that server-related and client-related validation code is in separate classes.</span></span> <span data-ttu-id="a6fe0-297">L'adapter offre anche un altro vantaggio. Essendo registrato nell'inserimento delle dipendenze, può usare gli altri servizi disponibili nell'inserimento delle dipendenze se necessario.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-297">The adapter also has the advantage that since it is registered in DI, other services in DI are available to it if needed.</span></span>
* <span data-ttu-id="a6fe0-298">Implementare `IClientModelValidator` nella classe `ValidationAttribute`.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-298">Implement `IClientModelValidator` in your `ValidationAttribute` class.</span></span> <span data-ttu-id="a6fe0-299">Questo metodo potrebbe essere appropriato se l'attributo non esegue la convalida lato server e non richiede altri servizi dell'inserimento delle dipendenze.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-299">This method might be appropriate if the attribute doesn't do any server-side validation and doesn't need any services from DI.</span></span>

### <a name="attributeadapter-for-client-side-validation"></a><span data-ttu-id="a6fe0-300">AttributeAdapter per la convalida lato client</span><span class="sxs-lookup"><span data-stu-id="a6fe0-300">AttributeAdapter for client-side validation</span></span>

<span data-ttu-id="a6fe0-301">Questo metodo di rendering degli attributi `data-` in HTML viene usato dall'attributo `ClassicMovie` nell'app di esempio.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-301">This method of rendering `data-` attributes in HTML is used by the `ClassicMovie` attribute in the sample app.</span></span> <span data-ttu-id="a6fe0-302">Per aggiungere la convalida lato client usando questo metodo:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-302">To add client validation by using this method:</span></span>

1. <span data-ttu-id="a6fe0-303">Creare una classe di adapter dell'attributo per l'attributo di convalida personalizzata.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-303">Create an attribute adapter class for the custom validation attribute.</span></span> <span data-ttu-id="a6fe0-304">Derivare la classe da <xref:Microsoft.AspNetCore.Mvc.DataAnnotations.AttributeAdapterBase%601>.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-304">Derive the class from <xref:Microsoft.AspNetCore.Mvc.DataAnnotations.AttributeAdapterBase%601>.</span></span> <span data-ttu-id="a6fe0-305">Creare un metodo `AddValidation` che aggiunge gli attributi `data-` all'output sottoposto a rendering, come illustrato in questo esempio:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-305">Create an `AddValidation` method that adds `data-` attributes to the rendered output, as shown in this example:</span></span>

   [!code-csharp[](validation/samples/3.x/ValidationSample/Validation/ClassicMovieAttributeAdapter.cs?name=snippet_Class)]

1. <span data-ttu-id="a6fe0-306">Creare una classe di provider dell'adapter che implementa <xref:Microsoft.AspNetCore.Mvc.DataAnnotations.IValidationAttributeAdapterProvider>.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-306">Create an adapter provider class that implements <xref:Microsoft.AspNetCore.Mvc.DataAnnotations.IValidationAttributeAdapterProvider>.</span></span> <span data-ttu-id="a6fe0-307">Nel metodo `GetAttributeAdapter` passare l'attributo personalizzato al costruttore dell'adapter, come illustrato in questo esempio:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-307">In the `GetAttributeAdapter` method pass in the custom attribute to the adapter's constructor, as shown in this example:</span></span>

   [!code-csharp[](validation/samples/3.x/ValidationSample/Validation/CustomValidationAttributeAdapterProvider.cs?name=snippet_Class)]

1. <span data-ttu-id="a6fe0-308">Registrare il provider dell'adapter per l'inserimento delle dipendenze in `Startup.ConfigureServices`:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-308">Register the adapter provider for DI in `Startup.ConfigureServices`:</span></span>

   [!code-csharp[](validation/samples/3.x/ValidationSample/Startup.cs?name=snippet_Configuration&highlight=9-10)]

### <a name="iclientmodelvalidator-for-client-side-validation"></a><span data-ttu-id="a6fe0-309">IClientModelValidator per la convalida lato client</span><span class="sxs-lookup"><span data-stu-id="a6fe0-309">IClientModelValidator for client-side validation</span></span>

<span data-ttu-id="a6fe0-310">Questo metodo di rendering degli attributi `data-` in HTML viene usato dall'attributo `ClassicMovieWithClientValidator` nell'app di esempio.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-310">This method of rendering `data-` attributes in HTML is used by the `ClassicMovieWithClientValidator` attribute in the sample app.</span></span> <span data-ttu-id="a6fe0-311">Per aggiungere la convalida lato client usando questo metodo:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-311">To add client validation by using this method:</span></span>

* <span data-ttu-id="a6fe0-312">Nell'attributo di convalida personalizzato implementare l'interfaccia `IClientModelValidator` e creare un metodo `AddValidation`.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-312">In the custom validation attribute, implement the `IClientModelValidator` interface and create an `AddValidation` method.</span></span> <span data-ttu-id="a6fe0-313">Nel metodo `AddValidation` aggiungere gli attributi `data-` per la convalida, come illustrato nell'esempio seguente:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-313">In the `AddValidation` method, add `data-` attributes for validation, as shown in the following example:</span></span>

  [!code-csharp[](validation/samples/3.x/ValidationSample/Validation/ClassicMovieWithClientValidatorAttribute.cs?name=snippet_Class)]

## <a name="disable-client-side-validation"></a><span data-ttu-id="a6fe0-314">Disabilitare la convalida lato client</span><span class="sxs-lookup"><span data-stu-id="a6fe0-314">Disable client-side validation</span></span>

<span data-ttu-id="a6fe0-315">Il codice seguente disabilita la convalida client nelle Razor pagine:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-315">The following code disables client validation in Razor Pages:</span></span>

[!code-csharp[](validation/samples/3.x/ValidationSample/Startup.cs?name=snippet_DisableClientValidation&highlight=2-5)]

<span data-ttu-id="a6fe0-316">Altre opzioni per disabilitare la convalida sul lato client:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-316">Other options to disable client-side validation:</span></span>

* <span data-ttu-id="a6fe0-317">Impostare come commento il riferimento a `_ValidationScriptsPartial` in tutti i file con *estensione cshtml* .</span><span class="sxs-lookup"><span data-stu-id="a6fe0-317">Comment out the reference to `_ValidationScriptsPartial` in all the *.cshtml* files.</span></span>
* <span data-ttu-id="a6fe0-318">Rimuovere il contenuto del file *Pages\Shared \_ ValidationScriptsPartial. cshtml* .</span><span class="sxs-lookup"><span data-stu-id="a6fe0-318">Remove the contents of the *Pages\Shared\_ValidationScriptsPartial.cshtml* file.</span></span>

<span data-ttu-id="a6fe0-319">L'approccio precedente non impedisce la convalida lato client della ASP.NET Core Identity Razor libreria di classi.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-319">The preceding approach won't prevent client side validation of ASP.NET Core Identity Razor Class Library.</span></span> <span data-ttu-id="a6fe0-320">Per altre informazioni, vedere <xref:security/authentication/scaffold-identity>.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-320">For more information, see <xref:security/authentication/scaffold-identity>.</span></span>

## <a name="additional-resources"></a><span data-ttu-id="a6fe0-321">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="a6fe0-321">Additional resources</span></span>

* [<span data-ttu-id="a6fe0-322">System.ComponentModel.DataAnnotations namespace</span><span class="sxs-lookup"><span data-stu-id="a6fe0-322">System.ComponentModel.DataAnnotations namespace</span></span>](xref:System.ComponentModel.DataAnnotations)
* [<span data-ttu-id="a6fe0-323">Associazione di modelli</span><span class="sxs-lookup"><span data-stu-id="a6fe0-323">Model Binding</span></span>](model-binding.md)

::: moniker-end

::: moniker range="< aspnetcore-3.0"

<span data-ttu-id="a6fe0-324">Questo articolo illustra come convalidare l'input dell'utente in un'app ASP.NET Core MVC o Razor pages.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-324">This article explains how to validate user input in an ASP.NET Core MVC or Razor Pages app.</span></span>

<span data-ttu-id="a6fe0-325">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/mvc/models/validation/sample) ([procedura per il download](xref:index#how-to-download-a-sample)).</span><span class="sxs-lookup"><span data-stu-id="a6fe0-325">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/mvc/models/validation/sample) ([how to download](xref:index#how-to-download-a-sample)).</span></span>

## <a name="model-state"></a><span data-ttu-id="a6fe0-326">Stato del modello</span><span class="sxs-lookup"><span data-stu-id="a6fe0-326">Model state</span></span>

<span data-ttu-id="a6fe0-327">Lo stato del modello rappresenta gli errori che provengono da due sottosistemi: associazione di modelli e convalida del modello.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-327">Model state represents errors that come from two subsystems: model binding and model validation.</span></span> <span data-ttu-id="a6fe0-328">Gli errori provenienti dall'[associazione di modelli](model-binding.md) sono in genere errori di conversione dei dati, ad esempio l'immissione di una "x" in un campo in cui è previsto un intero.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-328">Errors that originate from [model binding](model-binding.md) are generally data conversion errors (for example, an "x" is entered in a field that expects an integer).</span></span> <span data-ttu-id="a6fe0-329">La convalida del modello è un processo successivo all'associazione di modelli e segnala gli errori in caso di dati non conformi alle regole di business, ad esempio l'immissione del valore 0 in un campo in cui è previsto un valore compreso tra 1 e 5.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-329">Model validation occurs after model binding and reports errors where the data doesn't conform to business rules (for example, a 0 is entered in a field that expects a rating between 1 and 5).</span></span>

<span data-ttu-id="a6fe0-330">Sia l'associazione di modelli che la convalida si verificano prima dell'esecuzione di un'azione del controller o di un Razor metodo del gestore di pagine.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-330">Both model binding and validation occur before the execution of a controller action or a Razor Pages handler method.</span></span> <span data-ttu-id="a6fe0-331">Per le app Web, è responsabilità dell'app esaminare `ModelState.IsValid` e rispondere nel modo appropriato.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-331">For web apps, it's the app's responsibility to inspect `ModelState.IsValid` and react appropriately.</span></span> <span data-ttu-id="a6fe0-332">Le app Web in genere visualizzare di nuovo la pagina con un messaggio di errore:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-332">Web apps typically redisplay the page with an error message:</span></span>

[!code-csharp[](validation/samples_snapshot/2.x/Create.cshtml.cs?name=snippet&highlight=3-6)]

<span data-ttu-id="a6fe0-333">I controller API Web non devono controllare `ModelState.IsValid` se hanno l'attributo `[ApiController]`.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-333">Web API controllers don't have to check `ModelState.IsValid` if they have the `[ApiController]` attribute.</span></span> <span data-ttu-id="a6fe0-334">In tal caso, viene restituita una risposta HTTP 400 automatica contenente i dettagli dell'errore quando lo stato del modello non è valido.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-334">In that case, an automatic HTTP 400 response containing error details is returned when model state is invalid.</span></span> <span data-ttu-id="a6fe0-335">Per altre informazioni, vedere [Risposte HTTP 400 automatiche](xref:web-api/index#automatic-http-400-responses).</span><span class="sxs-lookup"><span data-stu-id="a6fe0-335">For more information, see [Automatic HTTP 400 responses](xref:web-api/index#automatic-http-400-responses).</span></span>

## <a name="rerun-validation"></a><span data-ttu-id="a6fe0-336">Rieseguire la convalida</span><span class="sxs-lookup"><span data-stu-id="a6fe0-336">Rerun validation</span></span>

<span data-ttu-id="a6fe0-337">La convalida è automatica, ma potrebbe essere necessario ripeterla manualmente.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-337">Validation is automatic, but you might want to repeat it manually.</span></span> <span data-ttu-id="a6fe0-338">Ad esempio, è possibile che si calcoli un valore per una proprietà e che si voglia rieseguire la convalida dopo aver impostato la proprietà sul valore calcolato.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-338">For example, you might compute a value for a property and want to rerun validation after setting the property to the computed value.</span></span> <span data-ttu-id="a6fe0-339">Per rieseguire la convalida, chiamare il metodo `TryValidateModel` come illustrato di seguito:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-339">To rerun validation, call the `TryValidateModel` method, as shown here:</span></span>

[!code-csharp[](validation/samples/2.x/ValidationSample/Controllers/MoviesController.cs?name=snippet_TryValidateModel&highlight=11)]

## <a name="validation-attributes"></a><span data-ttu-id="a6fe0-340">Attributi di convalida</span><span class="sxs-lookup"><span data-stu-id="a6fe0-340">Validation attributes</span></span>

<span data-ttu-id="a6fe0-341">Gli attributi di convalida consentono di specificare le regole di convalida per le proprietà del modello.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-341">Validation attributes let you specify validation rules for model properties.</span></span> <span data-ttu-id="a6fe0-342">Nell'esempio seguente dell'[app di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/mvc/models/validation/sample) viene illustrata una classe di modello annotata con attributi di convalida.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-342">The following example from [the sample app](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/mvc/models/validation/sample) shows a model class that is annotated with validation attributes.</span></span> <span data-ttu-id="a6fe0-343">`[ClassicMovie]` è un attributo di convalida personalizzato, mentre gli altri sono attributi predefiniti.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-343">The `[ClassicMovie]` attribute is a custom validation attribute and the others are built-in.</span></span> <span data-ttu-id="a6fe0-344">Non viene mostrato `[ClassicMovie2]` , che mostra un metodo alternativo per implementare un attributo personalizzato.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-344">Not shown is `[ClassicMovie2]`, which shows an alternative way to implement a custom attribute.</span></span>

[!code-csharp[](validation/samples/2.x/ValidationSample/Models/Movie.cs?name=snippet_ModelClass)]

## <a name="built-in-attributes"></a><span data-ttu-id="a6fe0-345">Attributi predefiniti</span><span class="sxs-lookup"><span data-stu-id="a6fe0-345">Built-in attributes</span></span>

<span data-ttu-id="a6fe0-346">Gli attributi di convalida predefiniti includono:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-346">Built-in validation attributes include:</span></span>

* <span data-ttu-id="a6fe0-347">`[CreditCard]`: Convalida che la proprietà abbia un formato di carta di credito.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-347">`[CreditCard]`: Validates that the property has a credit card format.</span></span>
* <span data-ttu-id="a6fe0-348">`[Compare]`: Verifica che due proprietà di un modello corrispondano.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-348">`[Compare]`: Validates that two properties in a model match.</span></span> <span data-ttu-id="a6fe0-349">Ad esempio, il file *Register.cshtml.cs* USA `[Compare]` per convalidare le due password immesse corrispondenti.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-349">For example, the *Register.cshtml.cs* file uses `[Compare]` to validate the two entered passwords match.</span></span> <span data-ttu-id="a6fe0-350">[Impalcatura Identity ](xref:security/authentication/scaffold-identity) per visualizzare il codice del registro.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-350">[Scaffold Identity](xref:security/authentication/scaffold-identity) to see the Register code.</span></span>
* <span data-ttu-id="a6fe0-351">`[EmailAddress]`: Convalida che la proprietà abbia un formato di posta elettronica.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-351">`[EmailAddress]`: Validates that the property has an email format.</span></span>
* <span data-ttu-id="a6fe0-352">`[Phone]`: Convalida che la proprietà abbia un formato di numero di telefono.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-352">`[Phone]`: Validates that the property has a telephone number format.</span></span>
* <span data-ttu-id="a6fe0-353">`[Range]`: Convalida che il valore della proprietà rientra in un intervallo specificato.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-353">`[Range]`: Validates that the property value falls within a specified range.</span></span>
* <span data-ttu-id="a6fe0-354">`[RegularExpression]`: Verifica che il valore della proprietà corrisponda a un'espressione regolare specificata.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-354">`[RegularExpression]`: Validates that the property value matches a specified regular expression.</span></span>
* <span data-ttu-id="a6fe0-355">`[Required]`: Verifica che il campo non sia null.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-355">`[Required]`: Validates that the field is not null.</span></span> <span data-ttu-id="a6fe0-356">Per informazioni dettagliate sul comportamento di questo attributo, vedere [ `[Required]` attributo](#required-attribute) .</span><span class="sxs-lookup"><span data-stu-id="a6fe0-356">See [`[Required]` attribute](#required-attribute) for details about this attribute's behavior.</span></span>
* <span data-ttu-id="a6fe0-357">`[StringLength]`: Verifica che un valore della proprietà stringa non superi il limite di lunghezza specificato.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-357">`[StringLength]`: Validates that a string property value doesn't exceed a specified length limit.</span></span>
* <span data-ttu-id="a6fe0-358">`[Url]`: Convalida che la proprietà abbia un formato URL.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-358">`[Url]`: Validates that the property has a URL format.</span></span>
* <span data-ttu-id="a6fe0-359">`[Remote]`: Convalida l'input sul client chiamando un metodo di azione sul server.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-359">`[Remote]`: Validates input on the client by calling an action method on the server.</span></span> <span data-ttu-id="a6fe0-360">Per informazioni dettagliate sul comportamento di questo attributo, vedere [ `[Remote]` attributo](#remote-attribute) .</span><span class="sxs-lookup"><span data-stu-id="a6fe0-360">See [`[Remote]` attribute](#remote-attribute) for details about this attribute's behavior.</span></span>

<span data-ttu-id="a6fe0-361">Quando si usa l' `[RegularExpression]` attributo con la convalida sul lato client, l'espressione regolare viene eseguita in JavaScript sul client.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-361">When using the `[RegularExpression]` attribute with client-side validation, the regex is executed in JavaScript on the client.</span></span> <span data-ttu-id="a6fe0-362">Ciò significa che verrà utilizzato il comportamento di corrispondenza [ECMAScript](/dotnet/standard/base-types/regular-expression-options#ecmascript-matching-behavior) .</span><span class="sxs-lookup"><span data-stu-id="a6fe0-362">This means [ECMAScript](/dotnet/standard/base-types/regular-expression-options#ecmascript-matching-behavior) matching behavior will be used.</span></span> <span data-ttu-id="a6fe0-363">Per altre informazioni, vedere [questo problema di GitHub](https://github.com/dotnet/corefx/issues/42487).</span><span class="sxs-lookup"><span data-stu-id="a6fe0-363">For more information, see [this GitHub issue](https://github.com/dotnet/corefx/issues/42487).</span></span>

<span data-ttu-id="a6fe0-364">Nello spazio dei nomi [System.ComponentModel.DataAnnotations](xref:System.ComponentModel.DataAnnotations) è possibile trovare un elenco completo degli attributi di convalida.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-364">A complete list of validation attributes can be found in the [System.ComponentModel.DataAnnotations](xref:System.ComponentModel.DataAnnotations) namespace.</span></span>

### <a name="error-messages"></a><span data-ttu-id="a6fe0-365">messaggi di errore</span><span class="sxs-lookup"><span data-stu-id="a6fe0-365">Error messages</span></span>

<span data-ttu-id="a6fe0-366">Gli attributi di convalida consentono di specificare il messaggio di errore da visualizzare in caso di input non valido.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-366">Validation attributes let you specify the error message to be displayed for invalid input.</span></span> <span data-ttu-id="a6fe0-367">Ad esempio:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-367">For example:</span></span>

```csharp
[StringLength(8, ErrorMessage = "Name length can't be more than 8.")]
```

<span data-ttu-id="a6fe0-368">Internamente gli attributi chiamano `String.Format` con un segnaposto per il nome campo e talvolta con segnaposto aggiuntivi.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-368">Internally, the attributes call `String.Format` with a placeholder for the field name and sometimes additional placeholders.</span></span> <span data-ttu-id="a6fe0-369">Ad esempio:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-369">For example:</span></span>

```csharp
[StringLength(8, ErrorMessage = "{0} length must be between {2} and {1}.", MinimumLength = 6)]
```

<span data-ttu-id="a6fe0-370">Quando applicato a una proprietà `Name`, il messaggio di errore creato con il codice precedente sarà "La lunghezza del nome utente deve essere compresa tra 6 e 8".</span><span class="sxs-lookup"><span data-stu-id="a6fe0-370">When applied to a `Name` property, the error message created by the preceding code would be "Name length must be between 6 and 8.".</span></span>

<span data-ttu-id="a6fe0-371">Per scoprire i parametri passati a `String.Format` per un messaggio di errore dell'attributo specifico, vedere il [codice sorgente DataAnnotations](https://github.com/dotnet/corefx/tree/master/src/System.ComponentModel.Annotations/src/System/ComponentModel/DataAnnotations).</span><span class="sxs-lookup"><span data-stu-id="a6fe0-371">To find out which parameters are passed to `String.Format` for a particular attribute's error message, see the [DataAnnotations source code](https://github.com/dotnet/corefx/tree/master/src/System.ComponentModel.Annotations/src/System/ComponentModel/DataAnnotations).</span></span>

## <a name="required-attribute"></a><span data-ttu-id="a6fe0-372">Attributo [Required]</span><span class="sxs-lookup"><span data-stu-id="a6fe0-372">[Required] attribute</span></span>

<span data-ttu-id="a6fe0-373">Per impostazione predefinita, il sistema di convalida considera i parametri non nullable o le proprietà come se avessero un attributo `[Required]`.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-373">By default, the validation system treats non-nullable parameters or properties as if they had a `[Required]` attribute.</span></span> <span data-ttu-id="a6fe0-374">I [tipi di valore](/dotnet/csharp/language-reference/keywords/value-types)`decimal` e `int` sono parametri non nullable.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-374">[Value types](/dotnet/csharp/language-reference/keywords/value-types) such as `decimal` and `int` are non-nullable.</span></span>

### <a name="required-validation-on-the-server"></a><span data-ttu-id="a6fe0-375">Convalida dell'attributo [Required] nel server</span><span class="sxs-lookup"><span data-stu-id="a6fe0-375">[Required] validation on the server</span></span>

<span data-ttu-id="a6fe0-376">Nel server un valore obbligatorio viene considerato mancante se la proprietà è Null.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-376">On the server, a required value is considered missing if the property is null.</span></span> <span data-ttu-id="a6fe0-377">Un campo non nullable è sempre valido e il messaggio di errore dell'attributo [Required] non viene mai visualizzato.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-377">A non-nullable field is always valid, and the [Required] attribute's error message is never displayed.</span></span>

<span data-ttu-id="a6fe0-378">Può però succedere che l'associazione di modelli per una proprietà non nullable abbia esito negativo, generando un messaggio di errore, ad esempio `The value '' is invalid`.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-378">However, model binding for a non-nullable property may fail, resulting in an error message such as `The value '' is invalid`.</span></span> <span data-ttu-id="a6fe0-379">Per specificare un messaggio di errore personalizzato per la convalida lato server di tipi non nullable, sono disponibili le opzioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-379">To specify a custom error message for server-side validation of non-nullable types, you have the following options:</span></span>

* <span data-ttu-id="a6fe0-380">Il campo deve ammettere i valori Null, ad esempio `decimal?` anziché `decimal`.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-380">Make the field nullable (for example, `decimal?` instead of `decimal`).</span></span> <span data-ttu-id="a6fe0-381">[Nullable \<T> ](/dotnet/csharp/programming-guide/nullable-types/) i tipi di valore vengono trattati come tipi nullable standard.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-381">[Nullable\<T>](/dotnet/csharp/programming-guide/nullable-types/) value types are treated like standard nullable types.</span></span>
* <span data-ttu-id="a6fe0-382">Specificare il messaggio di errore predefinito che l'associazione di modelli deve usare, come illustrato nell'esempio seguente:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-382">Specify the default error message to be used by model binding, as shown in the following example:</span></span>

  [!code-csharp[](validation/samples/2.x/ValidationSample/Startup.cs?name=snippet_MaxModelValidationErrors&highlight=4-5)]

  <span data-ttu-id="a6fe0-383">Per altre informazioni sugli errori dell'associazione di modelli che possono essere impostati come messaggi predefiniti, vedere <xref:Microsoft.AspNetCore.Mvc.ModelBinding.Metadata.DefaultModelBindingMessageProvider#methods>.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-383">For more information about model binding errors that you can set default messages for, see <xref:Microsoft.AspNetCore.Mvc.ModelBinding.Metadata.DefaultModelBindingMessageProvider#methods>.</span></span>

### <a name="required-validation-on-the-client"></a><span data-ttu-id="a6fe0-384">Convalida dell'attributo [Required] nel client</span><span class="sxs-lookup"><span data-stu-id="a6fe0-384">[Required] validation on the client</span></span>

<span data-ttu-id="a6fe0-385">Le stringhe e i tipi non nullable sono gestiti in modo diverso nel client rispetto a come vengono gestiti nel server.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-385">Non-nullable types and strings are handled differently on the client compared to the server.</span></span> <span data-ttu-id="a6fe0-386">Nel client:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-386">On the client:</span></span>

* <span data-ttu-id="a6fe0-387">Un valore viene considerato presente solo viene immesso un input per tale valore.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-387">A value is considered present only if input is entered for it.</span></span> <span data-ttu-id="a6fe0-388">La convalida lato client gestisce quindi i tipi non nullable come i tipi nullable.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-388">Therefore, client-side validation handles non-nullable types the same as nullable types.</span></span>
* <span data-ttu-id="a6fe0-389">Lo spazio vuoto in un campo stringa viene considerato un input valido per il metodo [required](https://jqueryvalidation.org/required-method/) della convalida di jQuery.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-389">Whitespace in a string field is considered valid input by the jQuery Validation [required](https://jqueryvalidation.org/required-method/) method.</span></span> <span data-ttu-id="a6fe0-390">La convalida lato server considera un campo stringa obbligatorio non valido solo se viene immesso uno spazio vuoto.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-390">Server-side validation considers a required string field invalid if only whitespace is entered.</span></span>

<span data-ttu-id="a6fe0-391">Come indicato in precedenza, i tipi non nullable vengono trattati come se avessero un attributo `[Required]`.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-391">As noted earlier, non-nullable types are treated as though they had a `[Required]` attribute.</span></span> <span data-ttu-id="a6fe0-392">Quindi viene eseguita la convalida lato client anche se non si applica l'attributo `[Required]`.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-392">That means you get client-side validation even if you don't apply the `[Required]` attribute.</span></span> <span data-ttu-id="a6fe0-393">Se invece non si usa l'attributo, viene visualizzato un messaggio di errore predefinito.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-393">But if you don't use the attribute, you get a default error message.</span></span> <span data-ttu-id="a6fe0-394">Per specificare un messaggio di errore personalizzato, usare l'attributo.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-394">To specify a custom error message, use the attribute.</span></span>

## <a name="remote-attribute"></a><span data-ttu-id="a6fe0-395">Attributo [Remote]</span><span class="sxs-lookup"><span data-stu-id="a6fe0-395">[Remote] attribute</span></span>

<span data-ttu-id="a6fe0-396">L'attributo `[Remote]` implementa la convalida lato client che richiede la chiamata a un metodo nel server per determinare se l'input del campo è valido.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-396">The `[Remote]` attribute implements client-side validation that requires calling a method on the server to determine whether field input is valid.</span></span> <span data-ttu-id="a6fe0-397">Ad esempio, l'app potrebbe dover verificare se un nome utente è già in uso.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-397">For example, the app may need to verify whether a user name is already in use.</span></span>

<span data-ttu-id="a6fe0-398">Per implementare la convalida remota:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-398">To implement remote validation:</span></span>

1. <span data-ttu-id="a6fe0-399">Creare un metodo di azione per JavaScript da chiamare.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-399">Create an action method for JavaScript to call.</span></span>  <span data-ttu-id="a6fe0-400">Il metodo [remote](https://jqueryvalidation.org/remote-method/) jQuery Validate prevede una risposta JSON:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-400">The jQuery Validate [remote](https://jqueryvalidation.org/remote-method/) method expects a JSON response:</span></span>

   * <span data-ttu-id="a6fe0-401">`"true"` indica che i dati di input sono validi.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-401">`"true"` means the input data is valid.</span></span>
   * <span data-ttu-id="a6fe0-402">`"false"`, `undefined` o `null` indica che l'input non è valido.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-402">`"false"`, `undefined`, or `null` means the input is invalid.</span></span>  <span data-ttu-id="a6fe0-403">Visualizzare il messaggio di errore predefinito.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-403">Display the default error message.</span></span>
   * <span data-ttu-id="a6fe0-404">Qualsiasi altra stringa indica che l'input non è valido.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-404">Any other string means the input is invalid.</span></span> <span data-ttu-id="a6fe0-405">Visualizzare la stringa come messaggio di errore personalizzato.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-405">Display the string as a custom error message.</span></span>

   <span data-ttu-id="a6fe0-406">Di seguito è riportato un esempio di un metodo di azione che restituisce un messaggio di errore personalizzato:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-406">Here's an example of an action method that returns a custom error message:</span></span>

   [!code-csharp[](validation/samples/2.x/ValidationSample/Controllers/UsersController.cs?name=snippet_VerifyEmail)]

1. <span data-ttu-id="a6fe0-407">Nella classe del modello annotare la proprietà con un attributo `[Remote]` che punta al metodo di azione di convalida, come illustrato nell'esempio seguente:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-407">In the model class, annotate the property with a `[Remote]` attribute that points to the validation action method, as shown in the following example:</span></span>

   [!code-csharp[](validation/samples/2.x/ValidationSample/Models/User.cs?name=snippet_UserEmailProperty)]
 
   <span data-ttu-id="a6fe0-408">L'attributo `[Remote]` si trova nello spazio dei nomi `Microsoft.AspNetCore.Mvc`.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-408">The `[Remote]` attribute is in the `Microsoft.AspNetCore.Mvc` namespace.</span></span> <span data-ttu-id="a6fe0-409">Installare il pacchetto NuGet [Microsoft.AspNetCore.Mvc.ViewFeatures](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.ViewFeatures) se non si usa il metapacchetto `Microsoft.AspNetCore.App` o `Microsoft.AspNetCore.All`.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-409">Install the [Microsoft.AspNetCore.Mvc.ViewFeatures](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.ViewFeatures) NuGet package if you're not using the `Microsoft.AspNetCore.App` or `Microsoft.AspNetCore.All` metapackage.</span></span>
   
### <a name="additional-fields"></a><span data-ttu-id="a6fe0-410">Altri campi</span><span class="sxs-lookup"><span data-stu-id="a6fe0-410">Additional fields</span></span>

<span data-ttu-id="a6fe0-411">La proprietà `[Remote]` dell'attributo `AdditionalFields` consente di convalidare combinazioni di campi rispetto ai dati nel server.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-411">The `AdditionalFields` property of the `[Remote]` attribute lets you validate combinations of fields against data on the server.</span></span> <span data-ttu-id="a6fe0-412">Ad esempio, se il modello `User` avesse le proprietà `FirstName` e `LastName`, potrebbe essere necessario controllare che non siano già esistenti utenti con la stessa coppia di nomi.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-412">For example, if the `User` model had `FirstName` and `LastName` properties, you might want to verify that no existing users already have that pair of names.</span></span> <span data-ttu-id="a6fe0-413">Nell'esempio riportato di seguito viene illustrato come usare `AdditionalFields`:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-413">The following example shows how to use `AdditionalFields`:</span></span>

[!code-csharp[](validation/samples/2.x/ValidationSample/Models/User.cs?name=snippet_UserNameProperties)]

<span data-ttu-id="a6fe0-414">`AdditionalFields` potrebbe essere impostato in modo esplicito sulle stringhe `"FirstName"` e `"LastName"` , ma l'uso dell'operatore [NameOf](/dotnet/csharp/language-reference/keywords/nameof) semplifica il refactoring in un secondo momento.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-414">`AdditionalFields` could be set explicitly to the strings `"FirstName"` and `"LastName"`, but using the [nameof](/dotnet/csharp/language-reference/keywords/nameof) operator simplifies later refactoring.</span></span> <span data-ttu-id="a6fe0-415">Il metodo di azione per la convalida deve accettare gli argomenti sia per il nome e sia per il cognome:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-415">The action method for this validation must accept both first name and last name arguments:</span></span>

[!code-csharp[](validation/samples/2.x/ValidationSample/Controllers/UsersController.cs?name=snippet_VerifyName)]

<span data-ttu-id="a6fe0-416">Quando si immette un nome o un cognome, JavaScript esegue una chiamata remota per verificare se tale coppia di nomi è in uso.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-416">When the user enters a first or last name, JavaScript makes a remote call to see if that pair of names has been taken.</span></span>

<span data-ttu-id="a6fe0-417">Per convalidare due o più campi aggiuntivi, specificarli sotto forma di elenco delimitato da virgole.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-417">To validate two or more additional fields, provide them as a comma-delimited list.</span></span> <span data-ttu-id="a6fe0-418">Ad esempio, per aggiungere una proprietà `MiddleName` al modello, impostare l'attributo `[Remote]` come illustrato nell'esempio seguente:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-418">For example, to add a `MiddleName` property to the model, set the `[Remote]` attribute as shown in the following example:</span></span>

```csharp
[Remote(action: "VerifyName", controller: "Users", AdditionalFields = nameof(FirstName) + "," + nameof(LastName))]
public string MiddleName { get; set; }
```

<span data-ttu-id="a6fe0-419">`AdditionalFields`, come tutti gli argomenti dell'attributo, deve essere un'espressione costante.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-419">`AdditionalFields`, like all attribute arguments, must be a constant expression.</span></span> <span data-ttu-id="a6fe0-420">Non usare quindi una [stringa interpolata](/dotnet/csharp/language-reference/keywords/interpolated-strings) oppure chiamare <xref:System.String.Join*> per inizializzare `AdditionalFields`.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-420">Therefore, don't use an [interpolated string](/dotnet/csharp/language-reference/keywords/interpolated-strings) or call <xref:System.String.Join*> to initialize `AdditionalFields`.</span></span>

## <a name="alternatives-to-built-in-attributes"></a><span data-ttu-id="a6fe0-421">Alternative agli attributi predefiniti</span><span class="sxs-lookup"><span data-stu-id="a6fe0-421">Alternatives to built-in attributes</span></span>

<span data-ttu-id="a6fe0-422">Se si vuole usare un tipo di convalida non definita da attributi predefiniti, è possibile eseguire le operazioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-422">If you need validation not provided by built-in attributes, you can:</span></span>

* <span data-ttu-id="a6fe0-423">[Creare attributi personalizzati](#custom-attributes).</span><span class="sxs-lookup"><span data-stu-id="a6fe0-423">[Create custom attributes](#custom-attributes).</span></span>
* <span data-ttu-id="a6fe0-424">[Implementare IValidatableObject](#ivalidatableobject).</span><span class="sxs-lookup"><span data-stu-id="a6fe0-424">[Implement IValidatableObject](#ivalidatableobject).</span></span>

## <a name="custom-attributes"></a><span data-ttu-id="a6fe0-425">Attributi personalizzati</span><span class="sxs-lookup"><span data-stu-id="a6fe0-425">Custom attributes</span></span>

<span data-ttu-id="a6fe0-426">Per gli scenari non gestiti dagli attributi di convalida predefiniti, è possibile creare attributi di convalida personalizzati.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-426">For scenarios that the built-in validation attributes don't handle, you can create custom validation attributes.</span></span> <span data-ttu-id="a6fe0-427">Creare una classe che eredita da <xref:System.ComponentModel.DataAnnotations.ValidationAttribute> ed eseguire l'override del metodo <xref:System.ComponentModel.DataAnnotations.ValidationAttribute.IsValid*>.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-427">Create a class that inherits from <xref:System.ComponentModel.DataAnnotations.ValidationAttribute>, and override the <xref:System.ComponentModel.DataAnnotations.ValidationAttribute.IsValid*> method.</span></span>

<span data-ttu-id="a6fe0-428">Il metodo `IsValid` accetta un oggetto denominato *value*, ovvero l'input da convalidare.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-428">The `IsValid` method accepts an object named *value*, which is the input to be validated.</span></span> <span data-ttu-id="a6fe0-429">Un overload accetta anche un oggetto `ValidationContext`, che contiene informazioni aggiuntive, ad esempio l'istanza del modello creato dall'associazione di modelli.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-429">An overload also accepts a `ValidationContext` object, which provides additional information, such as the model instance created by model binding.</span></span>

<span data-ttu-id="a6fe0-430">Nell'esempio seguente si convalida che la data di uscita di un film di genere *Classic* non sia successiva a un anno specifico.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-430">The following example validates that the release date for a movie in the *Classic* genre isn't later than a specified year.</span></span> <span data-ttu-id="a6fe0-431">L'attributo `[ClassicMovie2]` prima controlla il genere, poi prosegue solo se il genere è *Classic*.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-431">The `[ClassicMovie2]` attribute checks the genre first and continues only if it's *Classic*.</span></span> <span data-ttu-id="a6fe0-432">Per i film identificati come classici, controlla la data di uscita per verificare che non sia successiva al limite passato al costruttore dell'attributo.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-432">For movies identified as classics, it checks the release date to make sure it's not later than the limit passed to the attribute constructor.)</span></span>

[!code-csharp[](validation/samples/2.x/ValidationSample/Attributes/ClassicMovieAttribute.cs?name=snippet_ClassicMovieAttribute)]

<span data-ttu-id="a6fe0-433">La variabile `movie` nell'esempio precedente rappresenta un oggetto `Movie` contenente i dati dell'invio del modulo.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-433">The `movie` variable in the preceding example represents a `Movie` object that contains the data from the form submission.</span></span> <span data-ttu-id="a6fe0-434">Il metodo `IsValid` controlla la data e il genere.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-434">The `IsValid` method checks the date and genre.</span></span> <span data-ttu-id="a6fe0-435">Se la convalida ha esito positivo, `IsValid` restituisce un codice `ValidationResult.Success`.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-435">Upon successful validation, `IsValid` returns a `ValidationResult.Success` code.</span></span> <span data-ttu-id="a6fe0-436">Se la convalida ha esito negativo, viene restituito un codice `ValidationResult` con un messaggio di errore.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-436">When validation fails, a `ValidationResult` with an error message is returned.</span></span>

## <a name="ivalidatableobject"></a><span data-ttu-id="a6fe0-437">IValidatableObject</span><span class="sxs-lookup"><span data-stu-id="a6fe0-437">IValidatableObject</span></span>

<span data-ttu-id="a6fe0-438">L'esempio precedente usa solo tipi `Movie`.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-438">The preceding example works only with `Movie` types.</span></span> <span data-ttu-id="a6fe0-439">Un'altra opzione per la convalida a livello di classe consiste nell'implementare `IValidatableObject` nella classe del modello, come illustrato nell'esempio seguente:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-439">Another option for class-level validation is to implement `IValidatableObject` in the model class, as shown in the following example:</span></span>

[!code-csharp[](validation/samples/2.x/ValidationSample/Models/MovieIValidatable.cs?name=snippet&highlight=1,26-34)]

## <a name="top-level-node-validation"></a><span data-ttu-id="a6fe0-440">Convalida del nodo di primo livello</span><span class="sxs-lookup"><span data-stu-id="a6fe0-440">Top-level node validation</span></span>

<span data-ttu-id="a6fe0-441">I nodi di primo livello includono:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-441">Top-level nodes include:</span></span>

* <span data-ttu-id="a6fe0-442">Parametri di azione</span><span class="sxs-lookup"><span data-stu-id="a6fe0-442">Action parameters</span></span>
* <span data-ttu-id="a6fe0-443">Proprietà del controller</span><span class="sxs-lookup"><span data-stu-id="a6fe0-443">Controller properties</span></span>
* <span data-ttu-id="a6fe0-444">Parametri del gestore di pagina</span><span class="sxs-lookup"><span data-stu-id="a6fe0-444">Page handler parameters</span></span>
* <span data-ttu-id="a6fe0-445">Proprietà del modello di pagina</span><span class="sxs-lookup"><span data-stu-id="a6fe0-445">Page model properties</span></span>

<span data-ttu-id="a6fe0-446">Vengono convalidati i nodi di primo livello associati al modello, oltre a convalidare le proprietà del modello.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-446">Model-bound top-level nodes are validated in addition to validating model properties.</span></span> <span data-ttu-id="a6fe0-447">Nell'esempio seguente dell'app di esempio il metodo `VerifyPhone` usa <xref:System.ComponentModel.DataAnnotations.RegularExpressionAttribute> per convalidare il parametro di azione `phone`:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-447">In the following example from the sample app, the `VerifyPhone` method uses the <xref:System.ComponentModel.DataAnnotations.RegularExpressionAttribute> to validate the `phone` action parameter:</span></span>

[!code-csharp[](validation/samples/2.x/ValidationSample/Controllers/UsersController.cs?name=snippet_VerifyPhone)]

<span data-ttu-id="a6fe0-448">I nodi di primo livello possono usare <xref:Microsoft.AspNetCore.Mvc.ModelBinding.BindRequiredAttribute> con gli attributi di convalida.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-448">Top-level nodes can use <xref:Microsoft.AspNetCore.Mvc.ModelBinding.BindRequiredAttribute> with validation attributes.</span></span> <span data-ttu-id="a6fe0-449">Nell'esempio seguente dall'app di esempio, il metodo `CheckAge` specifica che il parametro `age` deve essere associato dalla stringa di query quando viene inviato il modulo:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-449">In the following example from the sample app, the `CheckAge` method specifies that the `age` parameter must be bound from the query string when the form is submitted:</span></span>

[!code-csharp[](validation/samples/2.x/ValidationSample/Controllers/UsersController.cs?name=snippet_CheckAge)]

<span data-ttu-id="a6fe0-450">La pagina Check Age (*CheckAge.cshtml*), include due moduli.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-450">In the Check Age page (*CheckAge.cshtml*), there are two forms.</span></span> <span data-ttu-id="a6fe0-451">Il primo modulo invia un valore `Age``99` come stringa di query: `https://localhost:5001/Users/CheckAge?Age=99`.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-451">The first form submits an `Age` value of `99` as a query string: `https://localhost:5001/Users/CheckAge?Age=99`.</span></span>

<span data-ttu-id="a6fe0-452">Quando viene inviato un parametro correttamente formattato `age` dalla stringa di query, il modulo viene convalidato.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-452">When a properly formatted `age` parameter from the query string is submitted, the form validates.</span></span>

<span data-ttu-id="a6fe0-453">Il secondo modulo nella pagina Check Age invia il valore `Age` nel corpo della richiesta e la convalida ha esito negativo.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-453">The second form on the Check Age page submits the `Age` value in the body of the request, and validation fails.</span></span> <span data-ttu-id="a6fe0-454">L'associazione non riesce perché il parametro `age` deve provenire da una stringa di query.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-454">Binding fails because the `age` parameter must come from a query string.</span></span>

<span data-ttu-id="a6fe0-455">Quando viene eseguita con `CompatibilityVersion.Version_2_1` o versione successiva, la convalida del nodo di primo livello è abilitata per impostazione predefinita.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-455">When running with `CompatibilityVersion.Version_2_1` or later, top-level node validation is enabled by default.</span></span> <span data-ttu-id="a6fe0-456">In caso contrario, la convalida del nodo di primo livello è disabilitata.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-456">Otherwise, top-level node validation is disabled.</span></span> <span data-ttu-id="a6fe0-457">L'opzione predefinita può essere sovrascritta impostando la proprietà <xref:Microsoft.AspNetCore.Mvc.MvcOptions.AllowValidatingTopLevelNodes*> in (`Startup.ConfigureServices`), come illustrato di seguito:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-457">The default option can be overridden by setting the <xref:Microsoft.AspNetCore.Mvc.MvcOptions.AllowValidatingTopLevelNodes*> property in (`Startup.ConfigureServices`), as shown here:</span></span>

[!code-csharp[](validation/samples_snapshot/2.x/Startup.cs?name=snippet_AddMvc&highlight=4)]

## <a name="maximum-errors"></a><span data-ttu-id="a6fe0-458">Numero massimo di errori</span><span class="sxs-lookup"><span data-stu-id="a6fe0-458">Maximum errors</span></span>

<span data-ttu-id="a6fe0-459">La convalida si interrompe quando viene raggiunto il numero di errori (200 per impostazione predefinita).</span><span class="sxs-lookup"><span data-stu-id="a6fe0-459">Validation stops when the maximum number of errors is reached (200 by default).</span></span> <span data-ttu-id="a6fe0-460">È possibile configurare questo numero con il codice seguente in `Startup.ConfigureServices`:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-460">You can configure this number with the following code in `Startup.ConfigureServices`:</span></span>

[!code-csharp[](validation/samples/2.x/ValidationSample/Startup.cs?name=snippet_MaxModelValidationErrors&highlight=3)]

## <a name="maximum-recursion"></a><span data-ttu-id="a6fe0-461">Numero massimo di ricorsioni</span><span class="sxs-lookup"><span data-stu-id="a6fe0-461">Maximum recursion</span></span>

<span data-ttu-id="a6fe0-462"><xref:Microsoft.AspNetCore.Mvc.ModelBinding.Validation.ValidationVisitor> attraversare l'oggetto grafico del modello che deve essere convalidato.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-462"><xref:Microsoft.AspNetCore.Mvc.ModelBinding.Validation.ValidationVisitor> traverses the object graph of the model being validated.</span></span> <span data-ttu-id="a6fe0-463">Per i modelli molti profondi o ricorsivi all'infinito, la convalida può generare un overflow dello stack.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-463">For models that are very deep or are infinitely recursive, validation may result in stack overflow.</span></span> <span data-ttu-id="a6fe0-464">[MvcOptions.MaxValidationDepth](xref:Microsoft.AspNetCore.Mvc.MvcOptions.MaxValidationDepth) consente di arrestare tempestivamente la convalida se la ricorsione del visitatore supera la profondità configurata.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-464">[MvcOptions.MaxValidationDepth](xref:Microsoft.AspNetCore.Mvc.MvcOptions.MaxValidationDepth) provides a way to stop validation early if the visitor recursion exceeds a configured depth.</span></span> <span data-ttu-id="a6fe0-465">Il valore predefinito di `MvcOptions.MaxValidationDepth` è 32 quando viene eseguito con `CompatibilityVersion.Version_2_2` o versione successiva.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-465">The default value of `MvcOptions.MaxValidationDepth` is 32 when running with `CompatibilityVersion.Version_2_2` or later.</span></span> <span data-ttu-id="a6fe0-466">Per le versioni precedenti, il valore è Null, vale a dire che non esistono limiti di profondità.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-466">For earlier versions, the value is null, which means no depth constraint.</span></span>

## <a name="automatic-short-circuit"></a><span data-ttu-id="a6fe0-467">Corto circuito automatico</span><span class="sxs-lookup"><span data-stu-id="a6fe0-467">Automatic short-circuit</span></span>

<span data-ttu-id="a6fe0-468">La convalida viene ignorata automaticamente (corto circuito) se l'oggetto grafico non richiede la convalida.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-468">Validation is automatically short-circuited (skipped) if the model graph doesn't require validation.</span></span> <span data-ttu-id="a6fe0-469">Gli oggetti per cui viene ignorata la convalida sono le raccolte di primitive (ad esempio `byte[]`, `string[]`, `Dictionary<string, string>`) e gli oggetti grafici complessi che non hanno validator.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-469">Objects that the runtime skips validation for include collections of primitives (such as `byte[]`, `string[]`, `Dictionary<string, string>`) and complex object graphs that don't have any validators.</span></span>

## <a name="disable-validation"></a><span data-ttu-id="a6fe0-470">Disabilitare la convalida</span><span class="sxs-lookup"><span data-stu-id="a6fe0-470">Disable validation</span></span>

<span data-ttu-id="a6fe0-471">Per disabilitare la convalida:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-471">To disable validation:</span></span>

1. <span data-ttu-id="a6fe0-472">Creare un'implementazione di `IObjectModelValidator` che non contrassegna i campi come non validi.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-472">Create an implementation of `IObjectModelValidator` that doesn't mark any fields as invalid.</span></span>

   [!code-csharp[](validation/samples/2.x/ValidationSample/Attributes/NullObjectModelValidator.cs?name=snippet_DisableValidation)]

1. <span data-ttu-id="a6fe0-473">Aggiungere il codice seguente a `Startup.ConfigureServices` per sostituire l'impostazione `IObjectModelValidator` predefinita nel contenitore di inserimento delle dipendenze.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-473">Add the following code to `Startup.ConfigureServices` to replace the default `IObjectModelValidator` implementation in the dependency injection container.</span></span>

   [!code-csharp[](validation/samples/2.x/ValidationSample/Startup.cs?name=snippet_DisableValidation)]

<span data-ttu-id="a6fe0-474">Potrebbero essere visualizzati errori di stato del modello generati dall'associazione di modelli.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-474">You might still see model state errors that originate from model binding.</span></span>

## <a name="client-side-validation"></a><span data-ttu-id="a6fe0-475">Convalida lato client</span><span class="sxs-lookup"><span data-stu-id="a6fe0-475">Client-side validation</span></span>

<span data-ttu-id="a6fe0-476">La convalida lato client non consente l'invio del modulo finché non è ritenuto valido.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-476">Client-side validation prevents submission until the form is valid.</span></span> <span data-ttu-id="a6fe0-477">Tramite il pulsante Invia viene eseguito JavaScript, che invia il modulo oppure visualizza i messaggi di errore.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-477">The Submit button runs JavaScript that either submits the form or displays error messages.</span></span>

<span data-ttu-id="a6fe0-478">La convalida lato client evitare un inutile round trip al server in caso di errori di input in un modulo.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-478">Client-side validation avoids an unnecessary round trip to the server when there are input errors on a form.</span></span> <span data-ttu-id="a6fe0-479">I riferimenti dello script seguenti in *_Layout.cshtml* e *_ValidationScriptsPartial.cshtml* supportano la convalida lato client:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-479">The following script references in *_Layout.cshtml* and *_ValidationScriptsPartial.cshtml* support client-side validation:</span></span>

[!code-cshtml[](validation/samples/2.x/ValidationSample/Views/Shared/_Layout.cshtml?name=snippet_ScriptTag)]

[!code-cshtml[](validation/samples/2.x/ValidationSample/Views/Shared/_ValidationScriptsPartial.cshtml?name=snippet_ScriptTags)]

<span data-ttu-id="a6fe0-480">Lo script [jQuery Unobtrusive Validation](https://github.com/aspnet/jquery-validation-unobtrusive) è una libreria front-end Microsoft personalizzata che si basa sul noto plug-in [jQuery Validate](https://jqueryvalidation.org/).</span><span class="sxs-lookup"><span data-stu-id="a6fe0-480">The [jQuery Unobtrusive Validation](https://github.com/aspnet/jquery-validation-unobtrusive) script is a custom Microsoft front-end library that builds on the popular [jQuery Validate](https://jqueryvalidation.org/) plugin.</span></span> <span data-ttu-id="a6fe0-481">Senza Query Unobtrusive Validation, sarebbe necessario scrivere il codice della stessa logica di convalida in due posizioni, vale a dire negli attributi di convalida lato server nelle proprietà del modello e nuovamente negli script lato client.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-481">Without jQuery Unobtrusive Validation, you would have to code the same validation logic in two places: once in the server-side validation attributes on model properties, and then again in client-side scripts.</span></span> <span data-ttu-id="a6fe0-482">Invece, gli [helper tag](xref:mvc/views/tag-helpers/intro)e agli [helper HTML](xref:mvc/views/overview) usano gli attributi di convalida e i metadati di tipo delle proprietà del modello per eseguire il rendering degli attributi `data-` HTML 5 negli elementi del modulo che devono essere convalidati.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-482">Instead, [Tag Helpers](xref:mvc/views/tag-helpers/intro) and [HTML helpers](xref:mvc/views/overview) use the validation attributes and type metadata from model properties to render HTML 5 `data-` attributes for the form elements that need validation.</span></span> <span data-ttu-id="a6fe0-483">jQuery Unobtrusive Validation analizza gli attributi `data-` e passa la logica a jQuery Validate, "copiando" in modo efficace la logica di convalida lato server nel client.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-483">jQuery Unobtrusive Validation parses the `data-` attributes and passes the logic to jQuery Validate, effectively "copying" the server-side validation logic to the client.</span></span> <span data-ttu-id="a6fe0-484">È possibile visualizzare gli errori di convalida nel client tramite gli helper tag, come illustrato di seguito:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-484">You can display validation errors on the client using tag helpers as shown here:</span></span>

[!code-cshtml[](validation/samples/2.x/ValidationSample/Views/Movies/Create.cshtml?name=snippet_ReleaseDate&highlight=4-5)]

<span data-ttu-id="a6fe0-485">Gli helper tag precedenti eseguono il rendering del codice HTML seguente.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-485">The preceding tag helpers render the following HTML.</span></span>

```html
<form action="/Movies/Create" method="post">
    <div class="form-horizontal">
        <h4>Movie</h4>
        <div class="text-danger"></div>
        <div class="form-group">
            <label class="col-md-2 control-label" for="ReleaseDate">ReleaseDate</label>
            <div class="col-md-10">
                <input class="form-control" type="datetime"
                data-val="true" data-val-required="The ReleaseDate field is required."
                id="ReleaseDate" name="ReleaseDate" value="">
                <span class="text-danger field-validation-valid"
                data-valmsg-for="ReleaseDate" data-valmsg-replace="true"></span>
            </div>
        </div>
    </div>
</form>
```

<span data-ttu-id="a6fe0-486">Si noti che gli attributi `data-` nell'output HTML corrispondono agli attributi di convalida per la proprietà `ReleaseDate`.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-486">Notice that the `data-` attributes in the HTML output correspond to the validation attributes for the `ReleaseDate` property.</span></span> <span data-ttu-id="a6fe0-487">L'attributo `data-val-required` contiene un messaggio di errore che viene visualizzato se l'utente non compila il campo relativo alla data di uscita.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-487">The `data-val-required` attribute contains an error message to display if the user doesn't fill in the release date field.</span></span> <span data-ttu-id="a6fe0-488">jQuery uninvaded Validation passa questo valore al metodo jQuery Validate [Required ()](https://jqueryvalidation.org/required-method/) , che quindi Visualizza il messaggio nell' **\<span>** elemento associato.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-488">jQuery Unobtrusive Validation passes this value to the jQuery Validate [required()](https://jqueryvalidation.org/required-method/) method, which then displays that message in the accompanying **\<span>** element.</span></span>

<span data-ttu-id="a6fe0-489">La convalida del tipo di dati si basa sul tipo .NET di una proprietà, a meno che sia sostituito dall'attributo `[DataType]`.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-489">Data type validation is based on the .NET type of a property, unless that is overridden by a `[DataType]` attribute.</span></span> <span data-ttu-id="a6fe0-490">I browser generano messaggi di errore predefiniti propri che il pacchetto jQuery Validation Unobtrusive Validation può comunque sostituire.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-490">Browsers have their own default error messages, but the jQuery Validation Unobtrusive Validation package can override those messages.</span></span> <span data-ttu-id="a6fe0-491">Gli attributi `[DataType]` e le sottoclassi, ad esempio `[EmailAddress]`, consentono di specificare il messaggio di errore.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-491">`[DataType]` attributes and subclasses such as `[EmailAddress]` let you specify the error message.</span></span>

### <a name="add-validation-to-dynamic-forms"></a><span data-ttu-id="a6fe0-492">Aggiungere la convalida a moduli dinamici</span><span class="sxs-lookup"><span data-stu-id="a6fe0-492">Add Validation to Dynamic Forms</span></span>

<span data-ttu-id="a6fe0-493">Quando la pagina viene caricata, jQuery Unobtrusive Validation passa la logica e i parametri di convalida a jQuery Validate.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-493">jQuery Unobtrusive Validation passes validation logic and parameters to jQuery Validate when the page first loads.</span></span> <span data-ttu-id="a6fe0-494">La convalida non viene quindi eseguita automaticamente nei moduli generati in modo dinamico.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-494">Therefore, validation doesn't work automatically on dynamically generated forms.</span></span> <span data-ttu-id="a6fe0-495">Per abilitare la convalida, chiedere a jQuery Unobtrusive Validation di analizzare il modulo dinamico immediatamente dopo essere stato creato.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-495">To enable validation, tell jQuery Unobtrusive Validation to parse the dynamic form immediately after you create it.</span></span> <span data-ttu-id="a6fe0-496">Il codice riportato di seguito configura ad esempio la convalida lato client in un modulo che è stato aggiunto tramite AJAX.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-496">For example, the following code sets up client-side validation on a form added via AJAX.</span></span>

```javascript
$.get({
    url: "https://url/that/returns/a/form",
    dataType: "html",
    error: function(jqXHR, textStatus, errorThrown) {
        alert(textStatus + ": Couldn't add form. " + errorThrown);
    },
    success: function(newFormHTML) {
        var container = document.getElementById("form-container");
        container.insertAdjacentHTML("beforeend", newFormHTML);
        var forms = container.getElementsByTagName("form");
        var newForm = forms[forms.length - 1];
        $.validator.unobtrusive.parse(newForm);
    }
})
```

<span data-ttu-id="a6fe0-497">Il metodo `$.validator.unobtrusive.parse()` accetta un selettore jQuery per ogni argomento.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-497">The `$.validator.unobtrusive.parse()` method accepts a jQuery selector for its one argument.</span></span> <span data-ttu-id="a6fe0-498">Questo metodo indica a jQuery Unobtrusive Validation di analizzare gli attributi `data-` dei moduli all'interno di tale tipo di selettore.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-498">This method tells jQuery Unobtrusive Validation to parse the `data-` attributes of forms within that selector.</span></span> <span data-ttu-id="a6fe0-499">I valori di tali attributi vengono quindi passati al plug-in jQuery Validate.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-499">The values of those attributes are then passed to the jQuery Validate plugin.</span></span>

### <a name="add-validation-to-dynamic-controls"></a><span data-ttu-id="a6fe0-500">Aggiungere la convalida a controlli dinamici</span><span class="sxs-lookup"><span data-stu-id="a6fe0-500">Add Validation to Dynamic Controls</span></span>

<span data-ttu-id="a6fe0-501">Il metodo `$.validator.unobtrusive.parse()` funziona su un intero modulo e non sui singoli controlli generati dinamicamente, ad esempio `<input>` e `<select/>`.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-501">The `$.validator.unobtrusive.parse()` method works on an entire form, not on individual dynamically generated controls, such as `<input>` and `<select/>`.</span></span> <span data-ttu-id="a6fe0-502">Per eseguire il reparse del modulo, rimuovere i dati di convalida aggiunti quando il modulo è stato precedentemente analizzato, come illustrato nell'esempio seguente:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-502">To reparse the form, remove the validation data that was added when the form was parsed earlier, as shown in the following example:</span></span>

```javascript
$.get({
    url: "https://url/that/returns/a/control",
    dataType: "html",
    error: function(jqXHR, textStatus, errorThrown) {
        alert(textStatus + ": Couldn't add control. " + errorThrown);
    },
    success: function(newInputHTML) {
        var form = document.getElementById("my-form");
        form.insertAdjacentHTML("beforeend", newInputHTML);
        $(form).removeData("validator")    // Added by jQuery Validate
               .removeData("unobtrusiveValidation");   // Added by jQuery Unobtrusive Validation
        $.validator.unobtrusive.parse(form);
    }
})
```

## <a name="custom-client-side-validation"></a><span data-ttu-id="a6fe0-503">Convalida lato client personalizzata</span><span class="sxs-lookup"><span data-stu-id="a6fe0-503">Custom client-side validation</span></span>

<span data-ttu-id="a6fe0-504">La convalida lato client personalizzata viene eseguita generando attributi `data-` HTML che vengono usati con un adapter jQuery Validate personalizzato.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-504">Custom client-side validation is done by generating `data-` HTML attributes that work with a custom jQuery Validate adapter.</span></span> <span data-ttu-id="a6fe0-505">Il codice dell'adapter di esempio seguente è stato scritto per gli attributi `ClassicMovie` e `ClassicMovie2` presentati precedentemente nell'articolo:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-505">The following sample adapter code was written for the `ClassicMovie` and `ClassicMovie2` attributes that were introduced earlier in this article:</span></span>

[!code-javascript[](validation/samples/2.x/ValidationSample/wwwroot/js/classicMovieValidator.js?name=snippet_UnobtrusiveValidation)]

<span data-ttu-id="a6fe0-506">Per informazioni sulla scrittura degli adapter, vedere la [documentazione di jQuery Validate](https://jqueryvalidation.org/documentation/).</span><span class="sxs-lookup"><span data-stu-id="a6fe0-506">For information about how to write adapters, see the [jQuery Validate documentation](https://jqueryvalidation.org/documentation/).</span></span>

<span data-ttu-id="a6fe0-507">L'uso di un adapter per un determinato campo viene attivato dagli attributi `data-`, i quali eseguono le operazioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-507">The use of an adapter for a given field is triggered by `data-` attributes that:</span></span>

* <span data-ttu-id="a6fe0-508">Contrassegnano il campo come campo da sottoporre a convalida (`data-val="true"`).</span><span class="sxs-lookup"><span data-stu-id="a6fe0-508">Flag the field as being subject to validation (`data-val="true"`).</span></span>
* <span data-ttu-id="a6fe0-509">Identificano un nome della regola di convalida e un testo del messaggio di errore, ad esempio `data-val-rulename="Error message."`.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-509">Identify a validation rule name and error message text (for example, `data-val-rulename="Error message."`).</span></span>
* <span data-ttu-id="a6fe0-510">Specificare i parametri aggiuntivi necessari al validator, ad esempio `data-val-rulename-parm1="value"`.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-510">Provide any additional parameters the validator needs (for example, `data-val-rulename-parm1="value"`).</span></span>

<span data-ttu-id="a6fe0-511">L'esempio seguente contiene gli attributi `data-` per l'attributo `ClassicMovie` dell'app di esempio:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-511">The following example shows the `data-` attributes for the sample app's `ClassicMovie` attribute:</span></span>

```html
<input class="form-control" type="datetime"
    data-val="true"
    data-val-classicmovie1="Classic movies must have a release year earlier than 1960."
    data-val-classicmovie1-year="1960"
    data-val-required="The ReleaseDate field is required."
    id="ReleaseDate" name="ReleaseDate" value="">
```

<span data-ttu-id="a6fe0-512">Come già accennato in precedenza, gli [helper tag](xref:mvc/views/tag-helpers/intro) e gli [helper HTML](xref:mvc/views/overview) usano le informazioni degli attributi di convalida per eseguire il rendering degli attributi `data-`.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-512">As noted earlier, [Tag Helpers](xref:mvc/views/tag-helpers/intro) and [HTML helpers](xref:mvc/views/overview) use information from validation attributes to render `data-` attributes.</span></span> <span data-ttu-id="a6fe0-513">Per scrivere il codice che crea attributi HTML `data-` personalizzati, sono disponibili due opzioni:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-513">There are two options for writing code that results in the creation of custom `data-` HTML attributes:</span></span>

* <span data-ttu-id="a6fe0-514">Creare una classe che deriva da `AttributeAdapterBase<TAttribute>` e una classe che implementa `IValidationAttributeAdapterProvider` e registrare l'attributo e il relativo adapter nell'inserimento delle dipendenze.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-514">Create a class that derives from `AttributeAdapterBase<TAttribute>` and a class that implements `IValidationAttributeAdapterProvider`, and register your attribute and its adapter in DI.</span></span> <span data-ttu-id="a6fe0-515">Questo metodo segue il [principio di singola responsabilità](https://wikipedia.org/wiki/Single_responsibility_principle) in quanto il codice di convalida relativo a server e client si trova in classi separate.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-515">This method follows the [single responsibility principal](https://wikipedia.org/wiki/Single_responsibility_principle) in that server-related and client-related validation code is in separate classes.</span></span> <span data-ttu-id="a6fe0-516">L'adapter offre anche un altro vantaggio. Essendo registrato nell'inserimento delle dipendenze, può usare gli altri servizi disponibili nell'inserimento delle dipendenze se necessario.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-516">The adapter also has the advantage that since it is registered in DI, other services in DI are available to it if needed.</span></span>
* <span data-ttu-id="a6fe0-517">Implementare `IClientModelValidator` nella classe `ValidationAttribute`.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-517">Implement `IClientModelValidator` in your `ValidationAttribute` class.</span></span> <span data-ttu-id="a6fe0-518">Questo metodo potrebbe essere appropriato se l'attributo non esegue la convalida lato server e non richiede altri servizi dell'inserimento delle dipendenze.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-518">This method might be appropriate if the attribute doesn't do any server-side validation and doesn't need any services from DI.</span></span>

### <a name="attributeadapter-for-client-side-validation"></a><span data-ttu-id="a6fe0-519">AttributeAdapter per la convalida lato client</span><span class="sxs-lookup"><span data-stu-id="a6fe0-519">AttributeAdapter for client-side validation</span></span>

<span data-ttu-id="a6fe0-520">Questo metodo di rendering degli attributi `data-` in HTML viene usato dall'attributo `ClassicMovie` nell'app di esempio.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-520">This method of rendering `data-` attributes in HTML is used by the `ClassicMovie` attribute in the sample app.</span></span> <span data-ttu-id="a6fe0-521">Per aggiungere la convalida lato client usando questo metodo:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-521">To add client validation by using this method:</span></span>

1. <span data-ttu-id="a6fe0-522">Creare una classe di adapter dell'attributo per l'attributo di convalida personalizzata.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-522">Create an attribute adapter class for the custom validation attribute.</span></span> <span data-ttu-id="a6fe0-523">Derivare la classe da <xref:Microsoft.AspNetCore.Mvc.DataAnnotations.AttributeAdapterBase%601>.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-523">Derive the class from <xref:Microsoft.AspNetCore.Mvc.DataAnnotations.AttributeAdapterBase%601>.</span></span> <span data-ttu-id="a6fe0-524">Creare un metodo `AddValidation` che aggiunge gli attributi `data-` all'output sottoposto a rendering, come illustrato in questo esempio:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-524">Create an `AddValidation` method that adds `data-` attributes to the rendered output, as shown in this example:</span></span>

   [!code-csharp[](validation/samples/2.x/ValidationSample/Attributes/ClassicMovieAttributeAdapter.cs?name=snippet_ClassicMovieAttributeAdapter)]

1. <span data-ttu-id="a6fe0-525">Creare una classe di provider dell'adapter che implementa <xref:Microsoft.AspNetCore.Mvc.DataAnnotations.IValidationAttributeAdapterProvider>.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-525">Create an adapter provider class that implements <xref:Microsoft.AspNetCore.Mvc.DataAnnotations.IValidationAttributeAdapterProvider>.</span></span> <span data-ttu-id="a6fe0-526">Nel metodo `GetAttributeAdapter` passare l'attributo personalizzato al costruttore dell'adapter, come illustrato in questo esempio:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-526">In the `GetAttributeAdapter` method pass in the custom attribute to the adapter's constructor, as shown in this example:</span></span>

   [!code-csharp[](validation/samples/2.x/ValidationSample/Attributes/CustomValidationAttributeAdapterProvider.cs?name=snippet_CustomValidationAttributeAdapterProvider)]

1. <span data-ttu-id="a6fe0-527">Registrare il provider dell'adapter per l'inserimento delle dipendenze in `Startup.ConfigureServices`:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-527">Register the adapter provider for DI in `Startup.ConfigureServices`:</span></span>

   [!code-csharp[](validation/samples/2.x/ValidationSample/Startup.cs?name=snippet_MaxModelValidationErrors&highlight=8-10)]

### <a name="iclientmodelvalidator-for-client-side-validation"></a><span data-ttu-id="a6fe0-528">IClientModelValidator per la convalida lato client</span><span class="sxs-lookup"><span data-stu-id="a6fe0-528">IClientModelValidator for client-side validation</span></span>

<span data-ttu-id="a6fe0-529">Questo metodo di rendering degli attributi `data-` in HTML viene usato dall'attributo `ClassicMovie2` nell'app di esempio.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-529">This method of rendering `data-` attributes in HTML is used by the `ClassicMovie2` attribute in the sample app.</span></span> <span data-ttu-id="a6fe0-530">Per aggiungere la convalida lato client usando questo metodo:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-530">To add client validation by using this method:</span></span>

* <span data-ttu-id="a6fe0-531">Nell'attributo di convalida personalizzato implementare l'interfaccia `IClientModelValidator` e creare un metodo `AddValidation`.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-531">In the custom validation attribute, implement the `IClientModelValidator` interface and create an `AddValidation` method.</span></span> <span data-ttu-id="a6fe0-532">Nel metodo `AddValidation` aggiungere gli attributi `data-` per la convalida, come illustrato nell'esempio seguente:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-532">In the `AddValidation` method, add `data-` attributes for validation, as shown in the following example:</span></span>

  [!code-csharp[](validation/samples/2.x/ValidationSample/Attributes/ClassicMovie2Attribute.cs?name=snippet_ClassicMovie2Attribute)]

## <a name="disable-client-side-validation"></a><span data-ttu-id="a6fe0-533">Disabilitare la convalida lato client</span><span class="sxs-lookup"><span data-stu-id="a6fe0-533">Disable client-side validation</span></span>

<span data-ttu-id="a6fe0-534">Il codice seguente disabilita la convalida lato client nelle visualizzazioni MVC:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-534">The following code disables client validation in MVC views:</span></span>

[!code-csharp[](validation/samples_snapshot/2.x/Startup2.cs?name=snippet_DisableClientValidation)]

<span data-ttu-id="a6fe0-535">E nelle Razor pagine:</span><span class="sxs-lookup"><span data-stu-id="a6fe0-535">And in Razor Pages:</span></span>

[!code-csharp[](validation/samples_snapshot/2.x/Startup3.cs?name=snippet_DisableClientValidation)]

<span data-ttu-id="a6fe0-536">Un'altra opzione per disabilitare la convalida lato client consiste nell'impostare come commento il riferimento a `_ValidationScriptsPartial` nel file *.cshtml*.</span><span class="sxs-lookup"><span data-stu-id="a6fe0-536">Another option for disabling client validation is to comment out the reference to `_ValidationScriptsPartial` in your *.cshtml* file.</span></span>

## <a name="additional-resources"></a><span data-ttu-id="a6fe0-537">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="a6fe0-537">Additional resources</span></span>

* [<span data-ttu-id="a6fe0-538">System.ComponentModel.DataAnnotations namespace</span><span class="sxs-lookup"><span data-stu-id="a6fe0-538">System.ComponentModel.DataAnnotations namespace</span></span>](xref:System.ComponentModel.DataAnnotations)
* [<span data-ttu-id="a6fe0-539">Associazione di modelli</span><span class="sxs-lookup"><span data-stu-id="a6fe0-539">Model Binding</span></span>](model-binding.md)

::: moniker-end
