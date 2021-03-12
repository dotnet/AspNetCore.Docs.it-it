---
title: Helper tag nei moduli in ASP.NET Core
author: rick-anderson
description: Descrive gli helper tag predefiniti usati con i moduli.
ms.author: riande
ms.custom: mvc
ms.date: 12/05/2019
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
uid: mvc/views/working-with-forms
ms.openlocfilehash: 51e5f2f74493e7f4c18273c8589ed0424a1f2cac
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102585981"
---
# <a name="tag-helpers-in-forms-in-aspnet-core"></a>Helper tag nei moduli in ASP.NET Core

Di [Rick Anderson](https://twitter.com/RickAndMSFT), [N. Taylor Mullen](https://github.com/NTaylorMullen), [Dave Paquette](https://twitter.com/Dave_Paquette) e [Jerrie Pelser](https://github.com/jerriep)

Questo documento illustra l'uso di helper tag Form e gli elementi HTML comunemente usati all'interno di questi. L'elemento [Form](https://www.w3.org/TR/html401/interact/forms.html) del linguaggio HTML rappresenta il meccanismo principale usato dalle app Web per eseguire il postback di dati nel server. La maggior parte di questo documento descrive gli [helper tag](tag-helpers/intro.md) e spiega come questi consentono di creare moduli HTML solidi in modo produttivo. Prima di leggere questo documento, è consigliabile leggere [Introduzione agli helper tag](tag-helpers/intro.md).

In molti casi, gli helper HTML offrono un approccio alternativo a un helper tag specifico, ma è importante tenere presente che gli helper tag non sostituiscono gli helper HTML e che non esiste un helper tag per ogni helper HTML. Se un'alternativa sotto forma di helper HTML esiste, viene citata.

<a name="my-asp-route-param-ref-label"></a>

## <a name="the-form-tag-helper"></a>Helper tag Form

Helper tag [form](https://www.w3.org/TR/html401/interact/forms.html) :

* Genera il [\<FORM>](https://www.w3.org/TR/html401/interact/forms.html) `action` valore dell'attributo HTML per un'azione del controller MVC o una route denominata

* Genera un [token di verifica della richiesta](/aspnet/mvc/overview/security/xsrfcsrf-prevention-in-aspnet-mvc-and-web-pages) nascosto per impedire richieste intersito false, in caso di uso con l'attributo `[ValidateAntiForgeryToken]` nel metodo azione HTTP Post

* Fornisce l'attributo `asp-route-<Parameter Name>`, dove `<Parameter Name>` viene aggiunto ai valori della route. I parametri `routeValues` in `Html.BeginForm` e `Html.BeginRouteForm` forniscono una funzionalità simile.

* Ha come helper HTML alternativi `Html.BeginForm` e `Html.BeginRouteForm`

Esempio:

[!code-cshtml[](working-with-forms/sample/final/Views/Demo/RegisterFormOnly.cshtml)]

L'helper tag Form precedente genera il codice HTML seguente:

```html
<form method="post" action="/Demo/Register">
    <!-- Input and Submit elements -->
    <input name="__RequestVerificationToken" type="hidden" value="<removed for brevity>">
</form>
```

Il runtime MVC genera il valore dell'attributo `action` dagli attributi `asp-controller` e `asp-action` dell'helper tag Form. L'helper tag Form genera anche un [token di verifica della richiesta](/aspnet/mvc/overview/security/xsrfcsrf-prevention-in-aspnet-mvc-and-web-pages) nascosto per impedire richieste intersito false, in caso di uso con l'attributo `[ValidateAntiForgeryToken]` nel metodo azione HTTP Post. La protezione di un Form HTML puro da richieste intersito false è difficile, e l'helper tag Form offre questo servizio.

### <a name="using-a-named-route"></a>Uso di una route denominata

L'attributo `asp-route` degli helper tag può anche generare un markup per l'attributo `action` HTML. Un'app con una [route](../../fundamentals/routing.md) denominata `register` può usare il markup seguente per la pagina di registrazione:

[!code-cshtml[](../../mvc/views/working-with-forms/sample/final/Views/Demo/RegisterRoute.cshtml)]

Molte delle visualizzazioni nella cartella *Views/Account* (generata quando si crea una nuova app Web con *account utente singoli*) contengono l'attributo [asp-route-returnurl](xref:mvc/views/working-with-forms):

```cshtml
<form asp-controller="Account" asp-action="Login"
     asp-route-returnurl="@ViewData["ReturnUrl"]"
     method="post" class="form-horizontal" role="form">
```

>[!NOTE]
>Con i modelli predefiniti, `returnUrl` viene popolato automaticamente solo quando si tenta di accedere a una risorsa autorizzata senza aver effettuato l'autenticazione o l'autorizzazione. Se l'utente tenta un accesso non autorizzato, il middleware di sicurezza lo reindirizza alla pagina di accesso con `returnUrl` impostato.

## <a name="the-form-action-tag-helper"></a>Helper tag per l'azione modulo

L'helper tag per l'azione modulo genera l'attributo `formaction` per l'elemento `<button ...>` o il tag `<input type="image" ...>` generato. L'attributo `formaction` controlla se un modulo invia i dati. Viene associato a [\<input>](https://www.w3.org/wiki/HTML/Elements/input) elementi di tipo `image` e di [\<button>](https://www.w3.org/wiki/HTML/Elements/button) elementi. L'helper Tag azione modulo consente l'utilizzo di diversi attributi [AnchorTagHelper](xref:mvc/views/tag-helpers/builtin-th/anchor-tag-helper) `asp-` per controllare il `formaction` collegamento generato per l'elemento corrispondente.

Attributi di [AnchorTagHelper](xref:mvc/views/tag-helpers/builtin-th/anchor-tag-helper) supportati per controllare il valore di `formaction`:

|Attributo|Descrizione|
|---|---|
|[asp-controller](xref:mvc/views/tag-helpers/builtin-th/anchor-tag-helper#asp-controller)|Nome del controller.|
|[asp-action](xref:mvc/views/tag-helpers/builtin-th/anchor-tag-helper#asp-action)|Nome del metodo di azione.|
|[asp-area](xref:mvc/views/tag-helpers/builtin-th/anchor-tag-helper#asp-area)|Nome dell'area.|
|[asp-page](xref:mvc/views/tag-helpers/builtin-th/anchor-tag-helper#asp-page)|Nome della Razor pagina.|
|[asp-page-handler](xref:mvc/views/tag-helpers/builtin-th/anchor-tag-helper#asp-page-handler)|Nome del gestore di Razor pagina.|
|[asp-route](xref:mvc/views/tag-helpers/builtin-th/anchor-tag-helper#asp-route)|Nome della route.|
|[asp-route-{value}](xref:mvc/views/tag-helpers/builtin-th/anchor-tag-helper#asp-route-value)|Singolo valore di route URL. Ad esempio: `asp-route-id="1234"`.|
|[asp-all-route-data](xref:mvc/views/tag-helpers/builtin-th/anchor-tag-helper#asp-all-route-data)|Tutti i valori di route.|
|[asp-fragment](xref:mvc/views/tag-helpers/builtin-th/anchor-tag-helper#asp-fragment)|Frammento di URL.|

### <a name="submit-to-controller-example"></a>Esempio di invio al controller

Il markup seguente invia il modulo all'azione `Index` di `HomeController` quando viene selezionato l'input o un pulsante:

```cshtml
<form method="post">
    <button asp-controller="Home" asp-action="Index">Click Me</button>
    <input type="image" src="..." alt="Or Click Me" asp-controller="Home" 
                                asp-action="Index">
</form>
```

Il markup precedente genera il codice HTML seguente:

```html
<form method="post">
    <button formaction="/Home">Click Me</button>
    <input type="image" src="..." alt="Or Click Me" formaction="/Home">
</form>
```

### <a name="submit-to-page-example"></a>Esempio di invio a una pagina

Il markup seguente invia il modulo alla `About` Razor pagina:

```cshtml
<form method="post">
    <button asp-page="About">Click Me</button>
    <input type="image" src="..." alt="Or Click Me" asp-page="About">
</form>
```

Il markup precedente genera il codice HTML seguente:

```html
<form method="post">
    <button formaction="/About">Click Me</button>
    <input type="image" src="..." alt="Or Click Me" formaction="/About">
</form>
```

### <a name="submit-to-route-example"></a>Esempio di invio a una route

Prendere in considerazione l'endpoint `/Home/Test`:

```csharp
public class HomeController : Controller
{
    [Route("/Home/Test", Name = "Custom")]
    public string Test()
    {
        return "This is the test page";
    }
}
```

Il markup seguente invia il modulo all'endpoint `/Home/Test`.

```cshtml
<form method="post">
    <button asp-route="Custom">Click Me</button>
    <input type="image" src="..." alt="Or Click Me" asp-route="Custom">
</form>
```

Il markup precedente genera il codice HTML seguente:

```html
<form method="post">
    <button formaction="/Home/Test">Click Me</button>
    <input type="image" src="..." alt="Or Click Me" formaction="/Home/Test">
</form>
```

## <a name="the-input-tag-helper"></a>Helper tag Input

L'helper tag di input associa un elemento HTML [\<input>](https://www.w3.org/wiki/HTML/Elements/input) a un'espressione di modello nella visualizzazione Razor.

Sintassi:

```cshtml
<input asp-for="<Expression Name>">
```

L'helper tag Input:

* Genera gli attributi HTML `id` e `name` per il nome dell'espressione specificata nell'attributo `asp-for`. `asp-for="Property1.Property2"` equivale a: `m => m.Property1.Property2`. Il nome dell'espressione viene usato come valore dell'attributo `asp-for`. Per altre informazioni, vedere la sezione [Nomi delle espressioni](#expression-names).

* Imposta il valore dell'attributo HTML `type` in base agli attributi relativi al tipo di modello e all'[annotazione dei dati](/dotnet/api/microsoft.aspnetcore.mvc.dataannotations.iattributeadapter) applicati alla proprietà del modello

* Non sovrascrive il valore dell'attributo HTML `type` se è già stato specificato

* Genera attributi di convalida [HTML5](https://developer.mozilla.org/docs/Web/Guide/HTML/HTML5) da attributi di [annotazione dei dati](/dotnet/api/microsoft.aspnetcore.mvc.dataannotations.iattributeadapter) applicati alla proprietà del modello

* Sovrappone una funzionalità di helper HTML con `Html.TextBoxFor` e `Html.EditorFor`. Per i dettagli, vedere la sezione **Alternative helper HTML per l'helper tag Input**.

* Consente una tipizzazione forte. Se il nome di proprietà viene modificato e non si aggiorna l'helper tag, si ottiene un errore simile al seguente:

  ```
  An error occurred during the compilation of a resource required to process
  this request. Please review the following specific error details and modify
  your source code appropriately.

  Type expected
   'RegisterViewModel' does not contain a definition for 'Email' and no
   extension method 'Email' accepting a first argument of type 'RegisterViewModel'
   could be found (are you missing a using directive or an assembly reference?)
  ```

L'helper tag `Input` imposta l'attributo HTML `type` in base al tipo .NET. La tabella seguente elenca alcuni tipi .NET comuni e il tipo HTML generato. Non sono elencati tutti i tipi .NET.

|Tipo .NET|Tipo di input|
|---|---|
|Bool|type="checkbox"|
|string|type="text"|
|Datetime|type=["datetime-local"](https://developer.mozilla.org/docs/Web/HTML/Element/input/datetime-local)|
|Byte|type="number"|
|Int|type="number"|
|Single, Double|type="number"|

La tabella seguente illustra alcuni attributi di [annotazioni dei dati](/dotnet/api/microsoft.aspnetcore.mvc.dataannotations.iattributeadapter) comuni di cui l'helper tag Input esegue il mapping a tipi di input specifici (non tutti gli attributi di convalida sono elencati):

|Attributo|Tipo di input|
|---|---|
|[EmailAddress]|type="email"|
|[Url]|type="url"|
|[HiddenInput]|type="hidden"|
|[Phone]|type="tel"|
|[DataType(DataType.Password)]|type="password"|
|[DataType(DataType.Date)]|type="date"|
|[DataType(DataType.Time)]|type="time"|

Esempio:

[!code-csharp[](working-with-forms/sample/final/ViewModels/RegisterViewModel.cs)]

[!code-cshtml[](working-with-forms/sample/final/Views/Demo/RegisterInput.cshtml)]

Il codice precedente genera il codice HTML seguente:

```html
  <form method="post" action="/Demo/RegisterInput">
      Email:
      <input type="email" data-val="true"
             data-val-email="The Email Address field is not a valid email address."
             data-val-required="The Email Address field is required."
             id="Email" name="Email" value=""><br>
      Password:
      <input type="password" data-val="true"
             data-val-required="The Password field is required."
             id="Password" name="Password"><br>
      <button type="submit">Register</button>
      <input name="__RequestVerificationToken" type="hidden" value="<removed for brevity>">
   </form>
```

Le annotazioni dei dati applicate alle proprietà `Email` e `Password` generano metadati per il modello. L'helper tag Input usa i metadati del modello e genere attributi [HTML5](https://developer.mozilla.org/docs/Web/Guide/HTML/HTML5) `data-val-*` (vedere [Convalida del modello](../models/validation.md)). Questi attributi descrivono i validator da collegare ai campi di input. Ciò consente una convalida HTML5 e [jQuery](https://jquery.com/) discreta. Gli attributi non intrusivi hanno il formato `data-val-rule="Error Message"` , dove rule è il nome della regola di convalida, ad esempio `data-val-required` ,, `data-val-email` `data-val-maxlength` e così via. Se un messaggio di errore viene fornito nell'attributo, viene visualizzato come valore per l' `data-val-rule` attributo. Esistono anche attributi di `data-val-ruleName-argumentName="argumentValue"` di Form che offrono dettagli aggiuntivi sulla regola, ad esempio, `data-val-maxlength-max="1024"` .

### <a name="html-helper-alternatives-to-input-tag-helper"></a>Alternative helper HTML per l'helper tag Input

`Html.TextBox`, `Html.TextBoxFor`, `Html.Editor` e `Html.EditorFor` hanno funzionalità che si sovrappongono a quelle dell'helper tag Input. L'helper tag Input imposta automaticamente l'attributo `type`, mentre `Html.TextBox` e `Html.TextBoxFor` non lo fanno. `Html.Editor` e `Html.EditorFor` gestiscono raccolte, oggetti complessi e modelli, mentre l'helper tag Input non lo fa. L'helper tag Input, `Html.EditorFor` e `Html.TextBoxFor` sono fortemente tipizzati (usano espressioni lambda), mentre `Html.TextBox` e `Html.Editor` non lo sono (usano nomi di espressioni).

### <a name="htmlattributes"></a>HtmlAttributes

Quando eseguono i modelli predefiniti, `@Html.Editor()` e `@Html.EditorFor()` usano una voce `ViewDataDictionary` speciale denominata `htmlAttributes`. È possibile aumentare questo comportamento usando i parametri `additionalViewData`. La chiave "htmlAttributes" non fa distinzione tra maiuscole e minuscole. La chiave "htmlAttributes" viene gestita in modo analogo all'oggetto `htmlAttributes` passato agli helper di input come `@Html.TextBox()`.

```cshtml
@Html.EditorFor(model => model.YourProperty, 
  new { htmlAttributes = new { @class="myCssClass", style="Width:100px" } })
```

### <a name="expression-names"></a>Nomi delle espressioni

Il valore dell'attributo `asp-for` è un `ModelExpression` e il lato destro di un'espressione lambda. Pertanto, `asp-for="Property1"` diventa `m => m.Property1` nel codice generato. Per questo motivo il prefisso `Model` non è necessario. È possibile usare il carattere "\@" per iniziare un'espressione inline, spostandolo prima di `m.`:

```cshtml
@{
  var joe = "Joe";
}

<input asp-for="@joe">
```

Genera quanto segue:

```html
<input type="text" id="joe" name="joe" value="Joe">
```

Con le proprietà delle raccolte, `asp-for="CollectionProperty[23].Member"` genera lo stesso nome di `asp-for="CollectionProperty[i].Member"` se il valore di `i` è `23`.

Quando ASP.NET Core MVC calcola il valore di `ModelExpression`, analizza diverse origini, tra cui `ModelState`. Considerare `<input type="text" asp-for="@Name">`. L'attributo `value` calcolato è il primo valore non Null da:

* Voce `ModelState` con chiave "Name".
* Risultato dell'espressione `Model.Name`.

### <a name="navigating-child-properties"></a>Esplorazione delle proprietà figlio

È anche possibile passare alle proprietà figlio tramite il percorso delle proprietà del modello di visualizzazione. Si consideri una classe modello più complessa contenente una proprietà `Address` figlio.

[!code-csharp[](../../mvc/views/working-with-forms/sample/final/ViewModels/AddressViewModel.cs?highlight=1,2,3,4&range=5-8)]

[!code-csharp[](../../mvc/views/working-with-forms/sample/final/ViewModels/RegisterAddressViewModel.cs?highlight=8&range=5-13)]

Nella visualizzazione, viene eseguita l'associazione a `Address.AddressLine1`:

[!code-cshtml[](../../mvc/views/working-with-forms/sample/final/Views/Demo/RegisterAddress.cshtml?highlight=6)]

Per `Address.AddressLine1` viene generato il codice HTML seguente:

```html
<input type="text" id="Address_AddressLine1" name="Address.AddressLine1" value="">
```

### <a name="expression-names-and-collections"></a>Nomi delle espressioni e raccolte

Esempio di modello contenente una matrice di `Colors`:

[!code-csharp[](../../mvc/views/working-with-forms/sample/final/ViewModels/Person.cs?highlight=3&range=5-10)]

Metodo di azione:

```csharp
public IActionResult Edit(int id, int colorIndex)
{
    ViewData["Index"] = colorIndex;
    return View(GetPerson(id));
}
```

Di seguito Razor viene illustrato come accedere a un `Color` elemento specifico:

[!code-cshtml[](working-with-forms/sample/final/Views/Demo/EditColor.cshtml)]

Modello *Views/Shared/EditorTemplates/String.cshtml*:

[!code-cshtml[](working-with-forms/sample/final/Views/Shared/EditorTemplates/String.cshtml)]

Esempio di uso di `List<T>`:

[!code-csharp[](working-with-forms/sample/final/ViewModels/ToDoItem.cs?range=3-8)]

Di seguito Razor viene illustrato come eseguire l'iterazione di una raccolta:

[!code-cshtml[](working-with-forms/sample/final/Views/Demo/Edit.cshtml)]

Modello *Views/Shared/EditorTemplates/ToDoItem.cshtml*:

[!code-cshtml[](working-with-forms/sample/final/Views/Shared/EditorTemplates/ToDoItem.cshtml)]

`foreach` deve essere usato se possibile, quando il valore verrà usato in un contesto `asp-for` o in un contesto equivalente `Html.DisplayFor`. In genere `for` è preferibile a `foreach` (se lo scenario lo consente) perché non è necessario allocare un enumeratore. Tuttavia, la valutazione di un indicizzatore in un'espressione LINQ può essere dispendiosa e deve essere ridotta al minimo.

&nbsp;

>[!NOTE]
>Il codice di esempio commentato precedente illustra come sostituire l'espressione lambda con l'operatore `@` per accedere a ogni `ToDoItem` nell'elenco.

## <a name="the-textarea-tag-helper"></a>Helper tag Textarea

`Textarea Tag Helper` è simile all'helper Tag Input.

* Genera gli `id` `name` attributi e e gli attributi di convalida dei dati dal modello per un [\<textarea>](https://www.w3.org/wiki/HTML/Elements/textarea) elemento.

* Consente una tipizzazione forte.

* Helper HTML alternativo: `Html.TextAreaFor`

Esempio:

[!code-csharp[](working-with-forms/sample/final/ViewModels/DescriptionViewModel.cs)]

[!code-cshtml[](../../mvc/views/working-with-forms/sample/final/Views/Demo/RegisterTextArea.cshtml?highlight=4)]

Viene generato il codice HTML seguente:

```html
<form method="post" action="/Demo/RegisterTextArea">
  <textarea data-val="true"
   data-val-maxlength="The field Description must be a string or array type with a maximum length of &#x27;1024&#x27;."
   data-val-maxlength-max="1024"
   data-val-minlength="The field Description must be a string or array type with a minimum length of &#x27;5&#x27;."
   data-val-minlength-min="5"
   id="Description" name="Description">
  </textarea>
  <button type="submit">Test</button>
  <input name="__RequestVerificationToken" type="hidden" value="<removed for brevity>">
</form>
```

## <a name="the-label-tag-helper"></a>Helper tag Label

* Genera la didascalia dell'etichetta e l' `for` attributo su un [\<label>](https://www.w3.org/wiki/HTML/Elements/label) elemento per il nome di un'espressione

* Helper HTML alternativo: `Html.LabelFor`.

Rispetto a un elemento etichetta HTML puro, `Label Tag Helper` offre i vantaggi seguenti:

* Si ottiene automaticamente il valore descrittivo dell'etichetta dall'attributo `Display`. Il nome visualizzato desiderato può cambiare nel tempo e la combinazione dell'attributo `Display` e dell'helper tag Label applica l'attributo `Display` ovunque venga usato.

* Meno markup nel codice sorgente

* Tipizzazione forte con la proprietà del modello.

Esempio:

[!code-csharp[](working-with-forms/sample/final/ViewModels/SimpleViewModel.cs)]

[!code-cshtml[](../../mvc/views/working-with-forms/sample/final/Views/Demo/RegisterLabel.cshtml?highlight=4)]

Per l'elemento `<label>` viene generato il codice HTML seguente:

```html
<label for="Email">Email Address</label>
```

L'helper tag Label genera il valore dell'attributo `for` di "Email", che è l'ID associato all'elemento `<input>`. Gli helper tag generano elementi `id` e `for` coerenti. In questo modo possono essere associati correttamente. La didascalia di questo esempio deriva dall'attributo `Display`. Se il modello non contiene un attributo `Display`, la didascalia corrisponde al nome della proprietà dell'espressione.

## <a name="the-validation-tag-helpers"></a>Helper tag di convalida

Ci sono due helper tag di convalida. `Validation Message Tag Helper`, che visualizza un messaggio di convalida per una singola proprietà del modello, e `Validation Summary Tag Helper`, che visualizza un riepilogo degli errori di convalida. `Input Tag Helper` aggiunge attributi di convalida sul lato client HTML5 agli elementi di input in base agli attributi di annotazione dei dati per le classi del modello. La convalida viene eseguita anche per il server. Quando si verifica un errore di convalida, l'helper tag di convalida visualizza questi messaggi di errore.

### <a name="the-validation-message-tag-helper"></a>Helper tag Validation Message

* Aggiunge l'attributo [HTML5](https://developer.mozilla.org/docs/Web/Guide/HTML/HTML5)  `data-valmsg-for="property"` all'elemento [span](https://developer.mozilla.org/docs/Web/HTML/Element/span), che collega i messaggi di errore di convalida nel campo di input della proprietà del modello specificato. Quando si verifica un errore di convalida sul lato client, [jQuery](https://jquery.com/) visualizza il messaggio di errore nell'elemento `<span>`.

* La convalida viene eseguita anche nel server. Nei client JavaScript può essere disabilitato e una parte della convalida può essere eseguita solo sul lato server.

* Helper HTML alternativo: `Html.ValidationMessageFor`

`Validation Message Tag Helper` viene usato con l'attributo `asp-validation-for` in un elemento HTML [span](https://developer.mozilla.org/docs/Web/HTML/Element/span).

```cshtml
<span asp-validation-for="Email"></span>
```

L'helper tag Validation Message genera il codice HTML seguente:

```html
<span class="field-validation-valid"
  data-valmsg-for="Email"
  data-valmsg-replace="true"></span>
```

In genere si usa `Validation Message Tag Helper` dopo un helper tag `Input` per la stessa proprietà. In questo modo è possibile visualizzare i messaggi di errore di convalida vicino all'input che ha causato l'errore.

> [!NOTE]
> È necessario avere una visualizzazione con i riferimenti agli script JavaScript e [jQuery](https://jquery.com/) corretti per la convalida lato client. Per altre informazioni, vedere [Convalida del modello](../models/validation.md).

Se si verifica un errore di convalida sul lato server, ad esempio quando la convalida sul lato server è personalizzata o quando la convalida sul lato client è disabilitata, MVC inserisce il messaggio di errore nel corpo dell'elemento `<span>`.

```html
<span class="field-validation-error" data-valmsg-for="Email"
            data-valmsg-replace="true">
   The Email Address field is required.
</span>
```

### <a name="the-validation-summary-tag-helper"></a>Helper tag Validation Summary

* Considera come destinazione gli elementi `<div>` con l'attributo `asp-validation-summary`

* Helper HTML alternativo: `@Html.ValidationSummary`

`Validation Summary Tag Helper` viene usato per visualizzare un riepilogo dei messaggi di convalida. Il valore dell'attributo `asp-validation-summary` può corrispondere a uno dei valori seguenti:

|asp-validation-summary|Messaggi di convalida visualizzati|
|--- |--- |
|ValidationSummary.All|Livello di modello e proprietà|
|ValidationSummary.ModelOnly|Modello|
|ValidationSummary.None|nessuno|

### <a name="sample"></a>Esempio

Nell'esempio seguente il modello di dati include `DataAnnotation` attributi, che generano messaggi di errore di convalida sull' `<input>` elemento.  Quando si verifica un errore di convalida, l'helper tag di convalida visualizza il messaggio di errore:

[!code-csharp[](working-with-forms/sample/final/ViewModels/RegisterViewModel.cs)]

[!code-cshtml[](../../mvc/views/working-with-forms/sample/final/Views/Demo/RegisterValidation.cshtml?highlight=4,6,8&range=1-10)]

Codice HTML generato (se il modello è valido):

```html
<form action="/DemoReg/Register" method="post">
  <div class="validation-summary-valid" data-valmsg-summary="true">
  <ul><li style="display:none"></li></ul></div>
  Email:  <input name="Email" id="Email" type="email" value=""
   data-val-required="The Email field is required."
   data-val-email="The Email field is not a valid email address."
   data-val="true"><br>
  <span class="field-validation-valid" data-valmsg-replace="true"
   data-valmsg-for="Email"></span><br>
  Password: <input name="Password" id="Password" type="password"
   data-val-required="The Password field is required." data-val="true"><br>
  <span class="field-validation-valid" data-valmsg-replace="true"
   data-valmsg-for="Password"></span><br>
  <button type="submit">Register</button>
  <input name="__RequestVerificationToken" type="hidden" value="<removed for brevity>">
</form>
```

## <a name="the-select-tag-helper"></a>Helper tag Select

* Genera l'elemento [select](https://www.w3.org/wiki/HTML/Elements/select) e gli elementi [option](https://www.w3.org/wiki/HTML/Elements/option) associati per le proprietà del modello.

* Ha come helper HTML alternativi `Html.DropDownListFor` e `Html.ListBoxFor`

`Select Tag Helper` `asp-for` specifica il nome della proprietà di modello per l'elemento [select](https://www.w3.org/wiki/HTML/Elements/select) e `asp-items` specifica gli elementi [option](https://www.w3.org/wiki/HTML/Elements/option).  Ad esempio:

[!code-cshtml[](working-with-forms/sample/final/Views/Home/Index.cshtml?range=4)]

Esempio:

[!code-csharp[](working-with-forms/sample/final/ViewModels/CountryViewModel.cs)]

Il metodo `Index` inizializza `CountryViewModel`, imposta il paese selezionato e lo passa alla visualizzazione `Index`.

[!code-csharp[](working-with-forms/sample/final/Controllers/HomeController.cs?range=8-13)]

Il metodo `Index` di HTTP POST visualizza la selezione:

[!code-csharp[](working-with-forms/sample/final/Controllers/HomeController.cs?range=15-27)]

Visualizzazione `Index`:

[!code-cshtml[](working-with-forms/sample/final/Views/Home/Index.cshtml?highlight=4)]

Che genera il codice HTML seguente (con "CA" selezionato):

```html
<form method="post" action="/">
     <select id="Country" name="Country">
       <option value="MX">Mexico</option>
       <option selected="selected" value="CA">Canada</option>
       <option value="US">USA</option>
     </select>
       <br /><button type="submit">Register</button>
     <input name="__RequestVerificationToken" type="hidden" value="<removed for brevity>">
   </form>
```

> [!NOTE]
> Non è consigliabile usare `ViewBag` o `ViewData` con l'helper tag Select. Un modello di visualizzazione è più solido e in genere presenta meno problemi quando si tratta di fornire metadati MVC.

Il valore dell'attributo `asp-for` rappresenta un caso speciale e non richiede un prefisso `Model`, che invece è richiesto dagli altri attributi di helper tag, ad esempio `asp-items`

[!code-cshtml[](working-with-forms/sample/final/Views/Home/Index.cshtml?range=4)]

### <a name="enum-binding"></a>Associazione di enum

Spesso è comodo usare `<select>` con una proprietà `enum` e generare gli elementi `SelectListItem` dai valori `enum`.

Esempio:

[!code-csharp[](working-with-forms/sample/final/ViewModels/CountryEnumViewModel.cs?range=3-7)]

[!code-csharp[](working-with-forms/sample/final/ViewModels/CountryEnum.cs)]

Il metodo `GetEnumSelectList` genera un oggetto `SelectList` per un enum.

[!code-cshtml[](../../mvc/views/working-with-forms/sample/final/Views/Home/IndexEnum.cshtml?highlight=5)]

È possibile contrassegnare l'elenco di enumeratori con l' `Display` attributo per ottenere un'interfaccia utente più ricca:

[!code-csharp[](working-with-forms/sample/final/ViewModels/CountryEnum.cs?highlight=5,7)]

Viene generato il codice HTML seguente:

```html
  <form method="post" action="/Home/IndexEnum">
         <select data-val="true" data-val-required="The EnumCountry field is required."
                 id="EnumCountry" name="EnumCountry">
             <option value="0">United Mexican States</option>
             <option value="1">United States of America</option>
             <option value="2">Canada</option>
             <option value="3">France</option>
             <option value="4">Germany</option>
             <option selected="selected" value="5">Spain</option>
         </select>
         <br /><button type="submit">Register</button>
         <input name="__RequestVerificationToken" type="hidden" value="<removed for brevity>">
    </form>
```

### <a name="option-group"></a>Gruppo di opzioni

L'  [\<optgroup>](https://www.w3.org/wiki/HTML/Elements/optgroup) elemento HTML viene generato quando il modello di visualizzazione contiene uno o più `SelectListGroup` oggetti.

`CountryViewModelGroup` suddivide gli elementi `SelectListItem` nei gruppi "North America" ed "Europe":

[!code-csharp[](../../mvc/views/working-with-forms/sample/final/ViewModels/CountryViewModelGroup.cs?highlight=5,6,14,20,26,32,38,44&range=6-56)]

I due gruppi sono illustrati di seguito:

![esempio di gruppi di opzioni](working-with-forms/_static/grp.png)

Codice HTML generato:

```html
 <form method="post" action="/Home/IndexGroup">
      <select id="Country" name="Country">
          <optgroup label="North America">
              <option value="MEX">Mexico</option>
              <option value="CAN">Canada</option>
              <option value="US">USA</option>
          </optgroup>
          <optgroup label="Europe">
              <option value="FR">France</option>
              <option value="ES">Spain</option>
              <option value="DE">Germany</option>
          </optgroup>
      </select>
      <br /><button type="submit">Register</button>
      <input name="__RequestVerificationToken" type="hidden" value="<removed for brevity>">
 </form>
```

### <a name="multiple-select"></a>Selezione multipla

L'helper tag Select genera automaticamente l'attributo [multiple = "multiple"](https://w3c.github.io/html-reference/select.html) se la proprietà specificata nell'attributo `asp-for` è `IEnumerable`. Dato, ad esempio, il modello seguente:

[!code-csharp[](../../mvc/views/working-with-forms/sample/final/ViewModels/CountryViewModelIEnumerable.cs?highlight=6)]

Con la visualizzazione seguente:

[!code-cshtml[](../../mvc/views/working-with-forms/sample/final/Views/Home/IndexMultiSelect.cshtml?highlight=4)]

Viene generato il codice HTML seguente:

```html
<form method="post" action="/Home/IndexMultiSelect">
    <select id="CountryCodes"
    multiple="multiple"
    name="CountryCodes"><option value="MX">Mexico</option>
<option value="CA">Canada</option>
<option value="US">USA</option>
<option value="FR">France</option>
<option value="ES">Spain</option>
<option value="DE">Germany</option>
</select>
    <br /><button type="submit">Register</button>
  <input name="__RequestVerificationToken" type="hidden" value="<removed for brevity>">
</form>
```

### <a name="no-selection"></a>Nessuna selezione

Se si usa l'opzione "not specified" in più pagine, è possibile creare un modello per evitare di ripetere lo stesso codice HTML:

[!code-cshtml[](../../mvc/views/working-with-forms/sample/final/Views/Home/IndexEmptyTemplate.cshtml?highlight=5)]

Modello *Views/Shared/EditorTemplates/CountryViewModel.cshtml*:

[!code-cshtml[](working-with-forms/sample/final/Views/Shared/EditorTemplates/CountryViewModel.cshtml)]

L'aggiunta [\<option>](https://www.w3.org/wiki/HTML/Elements/option) di elementi HTML non è limitata al case di *selezione* . La visualizzazione e il metodo di azione seguenti, ad esempio, generano codice HTML simile al codice precedente:

[!code-csharp[](working-with-forms/sample/final/Controllers/HomeController.cs?name=snippetNone)]

[!code-cshtml[](working-with-forms/sample/final/Views/Home/IndexOption.cshtml)]

L'elemento `<option>` corretto viene selezionato (con l'attributo `selected="selected"`) a seconda del valore `Country` corrente.

[!code-csharp[](working-with-forms/sample/final/Controllers/HomeController.cs?range=114-119)]

```html
 <form method="post" action="/Home/IndexEmpty">
      <select id="Country" name="Country">
          <option value="">&lt;none&gt;</option>
          <option value="MX">Mexico</option>
          <option value="CA" selected="selected">Canada</option>
          <option value="US">USA</option>
      </select>
      <br /><button type="submit">Register</button>
   <input name="__RequestVerificationToken" type="hidden" value="<removed for brevity>">
 </form>
 ```

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:mvc/views/tag-helpers/intro>
* [Elemento Form HTML](https://www.w3.org/TR/html401/interact/forms.html)
* [Token di verifica della richiesta](/aspnet/mvc/overview/security/xsrfcsrf-prevention-in-aspnet-mvc-and-web-pages)
* <xref:mvc/models/model-binding>
* <xref:mvc/models/validation>
* [Interfaccia IAttributeAdapter](/dotnet/api/Microsoft.AspNetCore.Mvc.DataAnnotations.IAttributeAdapter)
* [Frammenti di codice per questo documento](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/mvc/views/working-with-forms/sample/final)
