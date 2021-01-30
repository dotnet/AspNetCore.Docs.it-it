---
title: ASP.NET Core Blazor Data Binding
author: guardrex
description: Informazioni sulle funzionalità di data binding per i componenti e gli elementi DOM nelle Blazor app.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 10/22/2020
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
uid: blazor/components/data-binding
ms.openlocfilehash: 67a63f1b4f705a4857dea2e6d1a942d4f21469f5
ms.sourcegitcommit: 83524f739dd25fbfa95ee34e95342afb383b49fe
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 01/29/2021
ms.locfileid: "99057096"
---
# <a name="aspnet-core-no-locblazor-data-binding"></a><span data-ttu-id="f59cf-103">ASP.NET Core Blazor Data Binding</span><span class="sxs-lookup"><span data-stu-id="f59cf-103">ASP.NET Core Blazor data binding</span></span>

<span data-ttu-id="f59cf-104">Di [Luke Latham](https://github.com/guardrex), [Daniel Roth](https://github.com/danroth27)e [Steve Sanderson](https://github.com/SteveSandersonMS)</span><span class="sxs-lookup"><span data-stu-id="f59cf-104">By [Luke Latham](https://github.com/guardrex), [Daniel Roth](https://github.com/danroth27), and [Steve Sanderson](https://github.com/SteveSandersonMS)</span></span>

<span data-ttu-id="f59cf-105">Razor i componenti di forniscono funzionalità di data binding tramite un attributo dell'elemento HTML denominato [`@bind`](xref:mvc/views/razor#bind) con un campo, una proprietà o un Razor valore di espressione.</span><span class="sxs-lookup"><span data-stu-id="f59cf-105">Razor components provide data binding features via an HTML element attribute named [`@bind`](xref:mvc/views/razor#bind) with a field, property, or Razor expression value.</span></span>

<span data-ttu-id="f59cf-106">Nell'esempio seguente viene associato un `<input>` elemento al `currentValue` campo e un `<input>` elemento alla `CurrentValue` proprietà:</span><span class="sxs-lookup"><span data-stu-id="f59cf-106">The following example binds an `<input>` element to the `currentValue` field and an `<input>` element to the `CurrentValue` property:</span></span>

```razor
<p>
    <input @bind="currentValue" /> Current value: @currentValue
</p>

<p>
    <input @bind="CurrentValue" /> Current value: @CurrentValue
</p>

@code {
    private string currentValue;

    private string CurrentValue { get; set; }
}
```

<span data-ttu-id="f59cf-107">Quando uno degli elementi perde lo stato attivo, viene aggiornato il campo o la proprietà associata.</span><span class="sxs-lookup"><span data-stu-id="f59cf-107">When one of the elements loses focus, its bound field or property is updated.</span></span>

<span data-ttu-id="f59cf-108">La casella di testo viene aggiornata nell'interfaccia utente solo quando viene eseguito il rendering del componente, non in risposta alla modifica del valore della proprietà o del campo.</span><span class="sxs-lookup"><span data-stu-id="f59cf-108">The text box is updated in the UI only when the component is rendered, not in response to changing the field's or property's value.</span></span> <span data-ttu-id="f59cf-109">Poiché i componenti eseguono il rendering autonomamente dopo l'esecuzione del codice del gestore eventi, gli aggiornamenti dei campi e delle proprietà vengono in *genere* riflessi nell'interfaccia utente immediatamente dopo l'attivazione di un gestore eventi.</span><span class="sxs-lookup"><span data-stu-id="f59cf-109">Since components render themselves after event handler code executes, field and property updates are *usually* reflected in the UI immediately after an event handler is triggered.</span></span>

<span data-ttu-id="f59cf-110">L'utilizzo [`@bind`](xref:mvc/views/razor#bind) di con la `CurrentValue` Proprietà ( `<input @bind="CurrentValue" />` ) equivale essenzialmente a quanto segue:</span><span class="sxs-lookup"><span data-stu-id="f59cf-110">Using [`@bind`](xref:mvc/views/razor#bind) with the `CurrentValue` property (`<input @bind="CurrentValue" />`) is essentially equivalent to the following:</span></span>

```razor
<input value="@CurrentValue"
    @onchange="@((ChangeEventArgs __e) => CurrentValue = 
        __e.Value.ToString())" />

@code {
    private string CurrentValue { get; set; }
}
```

<span data-ttu-id="f59cf-111">Quando viene eseguito il rendering del componente, l'oggetto `value` dell'elemento di input deriva dalla `CurrentValue` Proprietà.</span><span class="sxs-lookup"><span data-stu-id="f59cf-111">When the component is rendered, the `value` of the input element comes from the `CurrentValue` property.</span></span> <span data-ttu-id="f59cf-112">Quando l'utente digita nella casella di testo e modifica lo stato attivo dell'elemento, `onchange` viene generato l'evento e la `CurrentValue` proprietà viene impostata sul valore modificato.</span><span class="sxs-lookup"><span data-stu-id="f59cf-112">When the user types in the text box and changes element focus, the `onchange` event is fired and the `CurrentValue` property is set to the changed value.</span></span> <span data-ttu-id="f59cf-113">In realtà, la generazione del codice è più complessa rispetto a [`@bind`](xref:mvc/views/razor#bind) quando vengono gestiti i casi in cui vengono eseguite le conversioni dei tipi.</span><span class="sxs-lookup"><span data-stu-id="f59cf-113">In reality, the code generation is more complex than that because [`@bind`](xref:mvc/views/razor#bind) handles cases where type conversions are performed.</span></span> <span data-ttu-id="f59cf-114">In linea di principio, [`@bind`](xref:mvc/views/razor#bind) associa il valore corrente di un'espressione a un `value` attributo e gestisce le modifiche utilizzando il gestore registrato.</span><span class="sxs-lookup"><span data-stu-id="f59cf-114">In principle, [`@bind`](xref:mvc/views/razor#bind) associates the current value of an expression with a `value` attribute and handles changes using the registered handler.</span></span>

<span data-ttu-id="f59cf-115">Associare una proprietà o un campo ad altri eventi includendo anche un `@bind:event` attributo con un `event` parametro.</span><span class="sxs-lookup"><span data-stu-id="f59cf-115">Bind a property or field on other events by also including an `@bind:event` attribute with an `event` parameter.</span></span> <span data-ttu-id="f59cf-116">Nell'esempio seguente viene associata la `CurrentValue` proprietà nell' `oninput` evento:</span><span class="sxs-lookup"><span data-stu-id="f59cf-116">The following example binds the `CurrentValue` property on the `oninput` event:</span></span>

```razor
<input @bind="CurrentValue" @bind:event="oninput" />

@code {
    private string CurrentValue { get; set; }
}
```

<span data-ttu-id="f59cf-117">Diversamente da `onchange` , che viene attivato quando l'elemento perde lo stato attivo, `oninput` viene attivato quando viene modificato il valore della casella di testo.</span><span class="sxs-lookup"><span data-stu-id="f59cf-117">Unlike `onchange`, which fires when the element loses focus, `oninput` fires when the value of the text box changes.</span></span>

<!-- Hold location for resolution of https://github.com/dotnet/AspNetCore.Docs/issues/19721 -->

<span data-ttu-id="f59cf-118">L'associazione di attributi distingue tra maiuscole e minuscole:</span><span class="sxs-lookup"><span data-stu-id="f59cf-118">Attribute binding is case sensitive:</span></span>

* <span data-ttu-id="f59cf-119">`@bind` è valido.</span><span class="sxs-lookup"><span data-stu-id="f59cf-119">`@bind` is valid.</span></span>
* <span data-ttu-id="f59cf-120">`@Bind` e `@BIND` non sono validi.</span><span class="sxs-lookup"><span data-stu-id="f59cf-120">`@Bind` and `@BIND` are invalid.</span></span>

## <a name="unparsable-values"></a><span data-ttu-id="f59cf-121">Valori non analizzabili</span><span class="sxs-lookup"><span data-stu-id="f59cf-121">Unparsable values</span></span>

<span data-ttu-id="f59cf-122">Quando un utente fornisce un valore non analizzabile a un elemento associato a un oggetto DataBound, il valore non analizzabile viene automaticamente ripristinato al valore precedente quando viene attivato l'evento bind.</span><span class="sxs-lookup"><span data-stu-id="f59cf-122">When a user provides an unparsable value to a databound element, the unparsable value is automatically reverted to its previous value when the bind event is triggered.</span></span>

<span data-ttu-id="f59cf-123">Considerare lo scenario seguente:</span><span class="sxs-lookup"><span data-stu-id="f59cf-123">Consider the following scenario:</span></span>

* <span data-ttu-id="f59cf-124">Un `<input>` elemento è associato a un `int` tipo con un valore iniziale di `123` :</span><span class="sxs-lookup"><span data-stu-id="f59cf-124">An `<input>` element is bound to an `int` type with an initial value of `123`:</span></span>

  ```razor
  <input @bind="inputValue" />

  @code {
      private int inputValue = 123;
  }
  ```

* <span data-ttu-id="f59cf-125">L'utente aggiorna il valore dell'elemento a `123.45` nella pagina e modifica lo stato attivo dell'elemento.</span><span class="sxs-lookup"><span data-stu-id="f59cf-125">The user updates the value of the element to `123.45` in the page and changes the element focus.</span></span>

<span data-ttu-id="f59cf-126">Nello scenario precedente, il valore dell'elemento viene ripristinato in `123` .</span><span class="sxs-lookup"><span data-stu-id="f59cf-126">In the preceding scenario, the element's value is reverted to `123`.</span></span> <span data-ttu-id="f59cf-127">Quando il valore `123.45` viene rifiutato a favore del valore originale di `123` , l'utente riconosce che il relativo valore non è stato accettato.</span><span class="sxs-lookup"><span data-stu-id="f59cf-127">When the value `123.45` is rejected in favor of the original value of `123`, the user understands that their value wasn't accepted.</span></span>

<span data-ttu-id="f59cf-128">Per impostazione predefinita, l'associazione si applica all' `onchange` evento dell'elemento ( `@bind="{PROPERTY OR FIELD}"` ).</span><span class="sxs-lookup"><span data-stu-id="f59cf-128">By default, binding applies to the element's `onchange` event (`@bind="{PROPERTY OR FIELD}"`).</span></span> <span data-ttu-id="f59cf-129">Utilizzare `@bind="{PROPERTY OR FIELD}" @bind:event={EVENT}` per attivare l'associazione in un evento diverso.</span><span class="sxs-lookup"><span data-stu-id="f59cf-129">Use `@bind="{PROPERTY OR FIELD}" @bind:event={EVENT}` to trigger binding on a different event.</span></span> <span data-ttu-id="f59cf-130">Per l' `oninput` evento ( `@bind:event="oninput"` ), la riversione viene eseguita dopo qualsiasi sequenza di tasti che introduce un valore non analizzabile.</span><span class="sxs-lookup"><span data-stu-id="f59cf-130">For the `oninput` event (`@bind:event="oninput"`), the reversion occurs after any keystroke that introduces an unparsable value.</span></span> <span data-ttu-id="f59cf-131">Quando la destinazione `oninput` è l'evento con un `int` tipo associato a, a un utente viene impedito di digitare un `.` carattere.</span><span class="sxs-lookup"><span data-stu-id="f59cf-131">When targeting the `oninput` event with an `int`-bound type, a user is prevented from typing a `.` character.</span></span> <span data-ttu-id="f59cf-132">Un `.` carattere viene immediatamente rimosso, quindi l'utente riceve il feedback immediato che sono consentiti solo numeri interi.</span><span class="sxs-lookup"><span data-stu-id="f59cf-132">A `.` character is immediately removed, so the user receives immediate feedback that only whole numbers are permitted.</span></span> <span data-ttu-id="f59cf-133">Esistono scenari in cui il ripristino del valore dell' `oninput` evento non è ideale, ad esempio quando l'utente deve essere autorizzato a cancellare un valore non analizzabile `<input>` .</span><span class="sxs-lookup"><span data-stu-id="f59cf-133">There are scenarios where reverting the value on the `oninput` event isn't ideal, such as when the user should be allowed to clear an unparsable `<input>` value.</span></span> <span data-ttu-id="f59cf-134">Le alternative includono:</span><span class="sxs-lookup"><span data-stu-id="f59cf-134">Alternatives include:</span></span>

* <span data-ttu-id="f59cf-135">Non usare l' `oninput` evento.</span><span class="sxs-lookup"><span data-stu-id="f59cf-135">Don't use the `oninput` event.</span></span> <span data-ttu-id="f59cf-136">Usare l' `onchange` evento predefinito (specificare solo `@bind="{PROPERTY OR FIELD}"` ), in cui un valore non valido non viene ripristinato fino a quando l'elemento non perde lo stato attivo.</span><span class="sxs-lookup"><span data-stu-id="f59cf-136">Use the default `onchange` event (only specify `@bind="{PROPERTY OR FIELD}"`), where an invalid value isn't reverted until the element loses focus.</span></span>
* <span data-ttu-id="f59cf-137">Eseguire l'associazione a un tipo nullable, ad esempio `int?` o, `string` e fornire la logica personalizzata per gestire le voci non valide.</span><span class="sxs-lookup"><span data-stu-id="f59cf-137">Bind to a nullable type, such as `int?` or `string` and provide custom logic to handle invalid entries.</span></span>
* <span data-ttu-id="f59cf-138">Utilizzare un [componente di convalida del modulo](xref:blazor/forms-validation), ad esempio <xref:Microsoft.AspNetCore.Components.Forms.InputNumber%601> o <xref:Microsoft.AspNetCore.Components.Forms.InputDate%601> .</span><span class="sxs-lookup"><span data-stu-id="f59cf-138">Use a [form validation component](xref:blazor/forms-validation), such as <xref:Microsoft.AspNetCore.Components.Forms.InputNumber%601> or <xref:Microsoft.AspNetCore.Components.Forms.InputDate%601>.</span></span> <span data-ttu-id="f59cf-139">I componenti di convalida dei moduli includono il supporto predefinito per la gestione di input non validi.</span><span class="sxs-lookup"><span data-stu-id="f59cf-139">Form validation components have built-in support to manage invalid inputs.</span></span> <span data-ttu-id="f59cf-140">Per altre informazioni, vedere <xref:blazor/forms-validation>.</span><span class="sxs-lookup"><span data-stu-id="f59cf-140">For more information, see <xref:blazor/forms-validation>.</span></span> <span data-ttu-id="f59cf-141">Componenti di convalida dei moduli:</span><span class="sxs-lookup"><span data-stu-id="f59cf-141">Form validation components:</span></span>
  * <span data-ttu-id="f59cf-142">Consente all'utente di fornire un input non valido e di ricevere errori di convalida nell'oggetto associato <xref:Microsoft.AspNetCore.Components.Forms.EditContext> .</span><span class="sxs-lookup"><span data-stu-id="f59cf-142">Permit the user to provide invalid input and receive validation errors on the associated <xref:Microsoft.AspNetCore.Components.Forms.EditContext>.</span></span>
  * <span data-ttu-id="f59cf-143">Visualizzare gli errori di convalida nell'interfaccia utente senza interferire con l'utente che immette dati Web Form aggiuntivi.</span><span class="sxs-lookup"><span data-stu-id="f59cf-143">Display validation errors in the UI without interfering with the user entering additional webform data.</span></span>

## <a name="format-strings"></a><span data-ttu-id="f59cf-144">Stringhe di formato</span><span class="sxs-lookup"><span data-stu-id="f59cf-144">Format strings</span></span>

<span data-ttu-id="f59cf-145">Il data binding funziona con <xref:System.DateTime> stringhe di formato usando `@bind:format` .</span><span class="sxs-lookup"><span data-stu-id="f59cf-145">Data binding works with <xref:System.DateTime> format strings using `@bind:format`.</span></span> <span data-ttu-id="f59cf-146">Altre espressioni di formato, ad esempio i formati di valuta o numerici, non sono disponibili in questo momento.</span><span class="sxs-lookup"><span data-stu-id="f59cf-146">Other format expressions, such as currency or number formats, aren't available at this time.</span></span>

```razor
<input @bind="startDate" @bind:format="yyyy-MM-dd" />

@code {
    private DateTime startDate = new DateTime(2020, 1, 1);
}
```

<span data-ttu-id="f59cf-147">Nel codice precedente, il `<input>` valore predefinito del tipo di campo dell'elemento ( `type` ) è `text` .</span><span class="sxs-lookup"><span data-stu-id="f59cf-147">In the preceding code, the `<input>` element's field type (`type`) defaults to `text`.</span></span> <span data-ttu-id="f59cf-148">`@bind:format` è supportato per l'associazione dei tipi .NET seguenti:</span><span class="sxs-lookup"><span data-stu-id="f59cf-148">`@bind:format` is supported for binding the following .NET types:</span></span>

* <xref:System.DateTime?displayProperty=fullName>
* <span data-ttu-id="f59cf-149"><xref:System.DateTime?displayProperty=fullName>?</span><span class="sxs-lookup"><span data-stu-id="f59cf-149"><xref:System.DateTime?displayProperty=fullName>?</span></span>
* <xref:System.DateTimeOffset?displayProperty=fullName>
* <span data-ttu-id="f59cf-150"><xref:System.DateTimeOffset?displayProperty=fullName>?</span><span class="sxs-lookup"><span data-stu-id="f59cf-150"><xref:System.DateTimeOffset?displayProperty=fullName>?</span></span>

<span data-ttu-id="f59cf-151">L' `@bind:format` attributo specifica il formato di data da applicare all'oggetto `value` dell' `<input>` elemento.</span><span class="sxs-lookup"><span data-stu-id="f59cf-151">The `@bind:format` attribute specifies the date format to apply to the `value` of the `<input>` element.</span></span> <span data-ttu-id="f59cf-152">Il formato viene usato anche per analizzare il valore quando `onchange` si verifica un evento.</span><span class="sxs-lookup"><span data-stu-id="f59cf-152">The format is also used to parse the value when an `onchange` event occurs.</span></span>

<span data-ttu-id="f59cf-153">Non è consigliabile specificare un formato per il `date` tipo di campo perché Blazor include il supporto incorporato per formattare le date.</span><span class="sxs-lookup"><span data-stu-id="f59cf-153">Specifying a format for the `date` field type isn't recommended because Blazor has built-in support to format dates.</span></span> <span data-ttu-id="f59cf-154">A prescindere dalla raccomandazione, utilizzare solo il `yyyy-MM-dd` formato data per il binding per funzionare correttamente se viene fornito un formato con il `date` tipo di campo:</span><span class="sxs-lookup"><span data-stu-id="f59cf-154">In spite of the recommendation, only use the `yyyy-MM-dd` date format for binding to function correctly if a format is supplied with the `date` field type:</span></span>

```razor
<input type="date" @bind="startDate" @bind:format="yyyy-MM-dd">
```

## <a name="binding-with-component-parameters"></a><span data-ttu-id="f59cf-155">Associazione con parametri dei componenti</span><span class="sxs-lookup"><span data-stu-id="f59cf-155">Binding with component parameters</span></span>

<span data-ttu-id="f59cf-156">Uno scenario comune è l'associazione di una proprietà in un componente figlio a una proprietà nell'elemento padre.</span><span class="sxs-lookup"><span data-stu-id="f59cf-156">A common scenario is binding a property in a child component to a property in its parent.</span></span> <span data-ttu-id="f59cf-157">Questo scenario è denominato *Binding concatenato* perché si verificano simultaneamente più livelli di associazione.</span><span class="sxs-lookup"><span data-stu-id="f59cf-157">This scenario is called a *chained bind* because multiple levels of binding occur simultaneously.</span></span>

<span data-ttu-id="f59cf-158">I [parametri del componente](xref:blazor/components/index#component-parameters) consentono di associare le proprietà di un componente padre con la `@bind-{PROPERTY}` sintassi.</span><span class="sxs-lookup"><span data-stu-id="f59cf-158">[Component parameters](xref:blazor/components/index#component-parameters) permit binding properties of a parent component with `@bind-{PROPERTY}` syntax.</span></span>

<span data-ttu-id="f59cf-159">Le associazioni concatenate non possono essere implementate con [`@bind`](xref:mvc/views/razor#bind) la sintassi nel componente figlio.</span><span class="sxs-lookup"><span data-stu-id="f59cf-159">Chained binds can't be implemented with [`@bind`](xref:mvc/views/razor#bind) syntax in the child component.</span></span> <span data-ttu-id="f59cf-160">Un gestore eventi e un valore devono essere specificati separatamente per supportare l'aggiornamento della proprietà nell'elemento padre dal componente figlio.</span><span class="sxs-lookup"><span data-stu-id="f59cf-160">An event handler and value must be specified separately to support updating the property in the parent from the child component.</span></span>

<span data-ttu-id="f59cf-161">Il componente padre utilizza ancora la [`@bind`](xref:mvc/views/razor#bind) sintassi per configurare l'associazione dati con il componente figlio.</span><span class="sxs-lookup"><span data-stu-id="f59cf-161">The parent component still leverages the [`@bind`](xref:mvc/views/razor#bind) syntax to set up the data-binding with the child component.</span></span>

<span data-ttu-id="f59cf-162">Il `Child` componente seguente ( `Shared/Child.razor` ) ha un `Year` parametro del componente e un `YearChanged` callback:</span><span class="sxs-lookup"><span data-stu-id="f59cf-162">The following `Child` component (`Shared/Child.razor`) has a `Year` component parameter and `YearChanged` callback:</span></span>

```razor
<div class="card bg-light mt-3" style="width:18rem ">
    <div class="card-body">
        <h3 class="card-title">Child Component</h3>
        <p class="card-text">Child <code>Year</code>: @Year</p>
    </div>
</div>

<button @onclick="UpdateYearFromChild">Update Year from Child</button>

@code {
    private Random r = new Random();

    [Parameter]
    public int Year { get; set; }

    [Parameter]
    public EventCallback<int> YearChanged { get; set; }

    private async Task UpdateYearFromChild()
    {
        await YearChanged.InvokeAsync(r.Next(1950, 2021));
    }
}
```

<span data-ttu-id="f59cf-163">Il callback ( <xref:Microsoft.AspNetCore.Components.EventCallback%601> ) deve essere denominato come nome del parametro del componente seguito dal `Changed` suffisso "" ( `{PARAMETER NAME}Changed` ).</span><span class="sxs-lookup"><span data-stu-id="f59cf-163">The callback (<xref:Microsoft.AspNetCore.Components.EventCallback%601>) must be named as the component parameter name followed by the "`Changed`" suffix (`{PARAMETER NAME}Changed`).</span></span> <span data-ttu-id="f59cf-164">Nell'esempio precedente, il callback è denominato `YearChanged` .</span><span class="sxs-lookup"><span data-stu-id="f59cf-164">In the preceding example, the callback is named `YearChanged`.</span></span> <span data-ttu-id="f59cf-165"><xref:Microsoft.AspNetCore.Components.EventCallback.InvokeAsync%2A?displayProperty=nameWithType> richiama il delegato associato all'associazione con l'argomento fornito e invia una notifica degli eventi per la proprietà modificata.</span><span class="sxs-lookup"><span data-stu-id="f59cf-165"><xref:Microsoft.AspNetCore.Components.EventCallback.InvokeAsync%2A?displayProperty=nameWithType> invokes the delegate associated with the binding with the provided argument and dispatches an event notification for the changed property.</span></span>

<span data-ttu-id="f59cf-166">Nel componente seguente `Parent` ( `Parent.razor` ) il `year` campo è associato al `Year` parametro del componente figlio:</span><span class="sxs-lookup"><span data-stu-id="f59cf-166">In the following `Parent` component (`Parent.razor`), the `year` field is bound to the `Year` parameter of the child component:</span></span>

```razor
@page "/Parent"

<h1>Parent Component</h1>

<p>Parent <code>year</code>: @year</p>

<button @onclick="UpdateYear">Update Parent <code>year</code></button>

<Child @bind-Year="year" />

@code {
    private Random r = new Random();
    private int year = 1979;

    private void UpdateYear()
    {
        year = r.Next(1950, 2021);
    }
}
```

<span data-ttu-id="f59cf-167">Il `Year` parametro è associabile perché contiene un evento complementare `YearChanged` che corrisponde al tipo del `Year` parametro.</span><span class="sxs-lookup"><span data-stu-id="f59cf-167">The `Year` parameter is bindable because it has a companion `YearChanged` event that matches the type of the `Year` parameter.</span></span>

<span data-ttu-id="f59cf-168">Per convenzione, una proprietà può essere associata a un gestore eventi corrispondente includendo un `@bind-{PROPERTY}:event` attributo assegnato al gestore.</span><span class="sxs-lookup"><span data-stu-id="f59cf-168">By convention, a property can be bound to a corresponding event handler by including an `@bind-{PROPERTY}:event` attribute assigned to the handler.</span></span> <span data-ttu-id="f59cf-169">`<Child @bind-Year="year" />` equivale alla scrittura:</span><span class="sxs-lookup"><span data-stu-id="f59cf-169">`<Child @bind-Year="year" />` is equivalent to writing:</span></span>

```razor
<Child @bind-Year="year" @bind-Year:event="YearChanged" />
```

<span data-ttu-id="f59cf-170">In un esempio più sofisticato e reale, il componente seguente `PasswordField` ( `PasswordField.razor` ):</span><span class="sxs-lookup"><span data-stu-id="f59cf-170">In a more sophisticated and real-world example, the following `PasswordField` component (`PasswordField.razor`):</span></span>

* <span data-ttu-id="f59cf-171">Imposta `<input>` il valore di un elemento su un `password` campo.</span><span class="sxs-lookup"><span data-stu-id="f59cf-171">Sets an `<input>` element's value to a `password` field.</span></span>
* <span data-ttu-id="f59cf-172">Espone le modifiche di una `Password` proprietà a un componente padre con un oggetto [`EventCallback`](xref:blazor/components/event-handling#eventcallback) che passa il valore corrente del campo figlio `password` come argomento.</span><span class="sxs-lookup"><span data-stu-id="f59cf-172">Exposes changes of a `Password` property to a parent component with an [`EventCallback`](xref:blazor/components/event-handling#eventcallback) that passes in the current value of the child's `password` field as its argument.</span></span>
* <span data-ttu-id="f59cf-173">Usa l' `onclick` evento per attivare il `ToggleShowPassword` metodo.</span><span class="sxs-lookup"><span data-stu-id="f59cf-173">Uses the `onclick` event to trigger the `ToggleShowPassword` method.</span></span> <span data-ttu-id="f59cf-174">Per altre informazioni, vedere <xref:blazor/components/event-handling>.</span><span class="sxs-lookup"><span data-stu-id="f59cf-174">For more information, see <xref:blazor/components/event-handling>.</span></span>

```razor
<h1>Provide your password</h1>

Password:

<input @oninput="OnPasswordChanged" 
       required 
       type="@(showPassword ? "text" : "password")" 
       value="@password" />

<button class="btn btn-primary" @onclick="ToggleShowPassword">
    Show password
</button>

@code {
    private bool showPassword;
    private string password;

    [Parameter]
    public string Password { get; set; }

    [Parameter]
    public EventCallback<string> PasswordChanged { get; set; }

    private async Task OnPasswordChanged(ChangeEventArgs e)
    {
        password = e.Value.ToString();

        await PasswordChanged.InvokeAsync(password);
    }

    private void ToggleShowPassword()
    {
        showPassword = !showPassword;
    }
}
```

<span data-ttu-id="f59cf-175">Il `PasswordField` componente viene usato in un altro componente:</span><span class="sxs-lookup"><span data-stu-id="f59cf-175">The `PasswordField` component is used in another component:</span></span>

```razor
@page "/Parent"

<h1>Parent Component</h1>

<PasswordField @bind-Password="password" />

@code {
    private string password;
}
```

<span data-ttu-id="f59cf-176">Eseguire controlli o errori di trap nel metodo che richiama il delegato dell'associazione.</span><span class="sxs-lookup"><span data-stu-id="f59cf-176">Perform checks or trap errors in the method that invokes the binding's delegate.</span></span> <span data-ttu-id="f59cf-177">Nell'esempio seguente viene fornito un feedback immediato all'utente se viene utilizzato uno spazio nel valore della password:</span><span class="sxs-lookup"><span data-stu-id="f59cf-177">The following example provides immediate feedback to the user if a space is used in the password's value:</span></span>

```razor
<h1>Child Component</h1>

Password: 

<input @oninput="OnPasswordChanged" 
       required 
       type="@(showPassword ? "text" : "password")" 
       value="@password" />

<button class="btn btn-primary" @onclick="ToggleShowPassword">
    Show password
</button>

<span class="text-danger">@validationMessage</span>

@code {
    private bool showPassword;
    private string password;
    private string validationMessage;

    [Parameter]
    public string Password { get; set; }

    [Parameter]
    public EventCallback<string> PasswordChanged { get; set; }

    private Task OnPasswordChanged(ChangeEventArgs e)
    {
        password = e.Value.ToString();

        if (password.Contains(' '))
        {
            validationMessage = "Spaces not allowed!";

            return Task.CompletedTask;
        }
        else
        {
            validationMessage = string.Empty;

            return PasswordChanged.InvokeAsync(password);
        }
    }

    private void ToggleShowPassword()
    {
        showPassword = !showPassword;
    }
}
```

<span data-ttu-id="f59cf-178">Per altre informazioni su <xref:Microsoft.AspNetCore.Components.EventCallback%601>, vedere <xref:blazor/components/event-handling#eventcallback>.</span><span class="sxs-lookup"><span data-stu-id="f59cf-178">For more information on <xref:Microsoft.AspNetCore.Components.EventCallback%601>, see <xref:blazor/components/event-handling#eventcallback>.</span></span>

## <a name="bind-across-more-than-two-components"></a><span data-ttu-id="f59cf-179">Associazione tra più di due componenti</span><span class="sxs-lookup"><span data-stu-id="f59cf-179">Bind across more than two components</span></span>

<span data-ttu-id="f59cf-180">È possibile eseguire l'associazione tramite un numero qualsiasi di componenti annidati, ma è necessario rispettare il flusso unidirezionale dei dati:</span><span class="sxs-lookup"><span data-stu-id="f59cf-180">You can bind through any number of nested components, but you must respect the one-way flow of data:</span></span>

* <span data-ttu-id="f59cf-181">Le notifiche *delle modifiche scorrono verso l'alto nella gerarchia*.</span><span class="sxs-lookup"><span data-stu-id="f59cf-181">Change notifications *flow up the hierarchy*.</span></span>
* <span data-ttu-id="f59cf-182">I nuovi valori dei parametri *scorrono verso il basso nella gerarchia*.</span><span class="sxs-lookup"><span data-stu-id="f59cf-182">New parameter values *flow down the hierarchy*.</span></span>

<span data-ttu-id="f59cf-183">Un approccio comune e consigliato consiste nel memorizzare solo i dati sottostanti nel componente padre, per evitare confusione sullo stato che deve essere aggiornato.</span><span class="sxs-lookup"><span data-stu-id="f59cf-183">A common and recommended approach is to only store the underlying data in the parent component to avoid any confusion about what state must be updated.</span></span>

<span data-ttu-id="f59cf-184">I componenti seguenti illustrano i concetti precedenti:</span><span class="sxs-lookup"><span data-stu-id="f59cf-184">The following components demonstrate the preceding concepts:</span></span>

<span data-ttu-id="f59cf-185">`ParentComponent.razor`:</span><span class="sxs-lookup"><span data-stu-id="f59cf-185">`ParentComponent.razor`:</span></span>

```razor
<h1>Parent Component</h1>

<p>Parent Message: <b>@parentMessage</b></p>

<p>
    <button @onclick="ChangeValue">Change from Parent</button>
</p>

<ChildComponent @bind-ChildMessage="parentMessage" />

@code {
    private string parentMessage = "Initial value set in Parent";

    private void ChangeValue()
    {
        parentMessage = $"Set in Parent {DateTime.Now}";
    }
}
```

<span data-ttu-id="f59cf-186">`ChildComponent.razor`:</span><span class="sxs-lookup"><span data-stu-id="f59cf-186">`ChildComponent.razor`:</span></span>

```razor
<div class="border rounded m-1 p-1">
    <h2>Child Component</h2>

    <p>Child Message: <b>@ChildMessage</b></p>

    <p>
        <button @onclick="ChangeValue">Change from Child</button>
    </p>

    <GrandchildComponent @bind-GrandchildMessage="BoundValue" />
</div>

@code {
    [Parameter]
    public string ChildMessage { get; set; }

    [Parameter]
    public EventCallback<string> ChildMessageChanged { get; set; }

    private string BoundValue
    {
        get => ChildMessage;
        set => ChildMessageChanged.InvokeAsync(value);
    }

    private async Task ChangeValue()
    {
        await ChildMessageChanged.InvokeAsync(
            $"Set in Child {DateTime.Now}");
    }
}
```

<span data-ttu-id="f59cf-187">`GrandchildComponent.razor`:</span><span class="sxs-lookup"><span data-stu-id="f59cf-187">`GrandchildComponent.razor`:</span></span>

```razor
<div class="border rounded m-1 p-1">
    <h3>Grandchild Component</h3>

    <p>Grandchild Message: <b>@GrandchildMessage</b></p>

    <p>
        <button @onclick="ChangeValue">Change from Grandchild</button>
    </p>
</div>

@code {
    [Parameter]
    public string GrandchildMessage { get; set; }

    [Parameter]
    public EventCallback<string> GrandchildMessageChanged { get; set; }

    private async Task ChangeValue()
    {
        await GrandchildMessageChanged.InvokeAsync(
            $"Set in Grandchild {DateTime.Now}");
    }
}
```

<span data-ttu-id="f59cf-188">Per un approccio alternativo adatto alla condivisione dei dati in memoria tra i componenti che non sono necessariamente annidati, vedere la sezione relativa al *servizio contenitore di stato in memoria* dell' <xref:blazor/state-management> articolo.</span><span class="sxs-lookup"><span data-stu-id="f59cf-188">For an alternative approach suited to sharing data in-memory across components that aren't necessarily nested, see the *In-memory state container service* section of the <xref:blazor/state-management> article.</span></span>

## <a name="additional-resources"></a><span data-ttu-id="f59cf-189">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="f59cf-189">Additional resources</span></span>

* [<span data-ttu-id="f59cf-190">Associazione ai pulsanti di opzione in un modulo</span><span class="sxs-lookup"><span data-stu-id="f59cf-190">Binding to radio buttons in a form</span></span>](xref:blazor/forms-validation#radio-buttons)
* [<span data-ttu-id="f59cf-191">Associazione `<select>` di opzioni di elementi a valori di oggetti C# `null` in un form</span><span class="sxs-lookup"><span data-stu-id="f59cf-191">Binding `<select>` element options to C# object `null` values in a form</span></span>](xref:blazor/forms-validation#binding-select-element-options-to-c-object-null-values)
