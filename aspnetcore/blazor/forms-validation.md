---
title: ASP.NET Core Blazor moduli e convalida
author: guardrex
description: Informazioni su come usare i moduli e gli scenari di convalida di campi in Blazor .
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 09/17/2020
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
uid: blazor/forms-validation
ms.openlocfilehash: eb72810a5b65232aa778daa556a9b2d406807e87
ms.sourcegitcommit: 1436bd4d70937d6ec3140da56d96caab33c4320b
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/06/2021
ms.locfileid: "102395136"
---
# <a name="aspnet-core-blazor-forms-and-validation"></a><span data-ttu-id="a2aed-103">ASP.NET Core Blazor moduli e convalida</span><span class="sxs-lookup"><span data-stu-id="a2aed-103">ASP.NET Core Blazor forms and validation</span></span>

<span data-ttu-id="a2aed-104">I moduli e la convalida sono supportati nell' Blazor utilizzo delle [annotazioni dei dati](xref:mvc/models/validation).</span><span class="sxs-lookup"><span data-stu-id="a2aed-104">Forms and validation are supported in Blazor using [data annotations](xref:mvc/models/validation).</span></span>

<span data-ttu-id="a2aed-105">Il `ExampleModel` tipo seguente definisce la logica di convalida usando le annotazioni dei dati:</span><span class="sxs-lookup"><span data-stu-id="a2aed-105">The following `ExampleModel` type defines validation logic using data annotations:</span></span>

```csharp
using System.ComponentModel.DataAnnotations;

public class ExampleModel
{
    [Required]
    [StringLength(10, ErrorMessage = "Name is too long.")]
    public string Name { get; set; }
}
```

<span data-ttu-id="a2aed-106">Un modulo viene definito usando il <xref:Microsoft.AspNetCore.Components.Forms.EditForm> componente.</span><span class="sxs-lookup"><span data-stu-id="a2aed-106">A form is defined using the <xref:Microsoft.AspNetCore.Components.Forms.EditForm> component.</span></span> <span data-ttu-id="a2aed-107">Il modulo seguente illustra elementi, componenti e codice tipici Razor :</span><span class="sxs-lookup"><span data-stu-id="a2aed-107">The following form demonstrates typical elements, components, and Razor code:</span></span>

```razor
<EditForm Model="@exampleModel" OnValidSubmit="@HandleValidSubmit">
    <DataAnnotationsValidator />
    <ValidationSummary />

    <InputText id="name" @bind-Value="exampleModel.Name" />

    <button type="submit">Submit</button>
</EditForm>

@code {
    private ExampleModel exampleModel = new ExampleModel();

    private void HandleValidSubmit()
    {
        ...
    }
}
```

<span data-ttu-id="a2aed-108">Nell'esempio precedente:</span><span class="sxs-lookup"><span data-stu-id="a2aed-108">In the preceding example:</span></span>

* <span data-ttu-id="a2aed-109">Il modulo convalida l'input dell'utente nel `name` campo usando la convalida definita nel `ExampleModel` tipo.</span><span class="sxs-lookup"><span data-stu-id="a2aed-109">The form validates user input in the `name` field using the validation defined in the `ExampleModel` type.</span></span> <span data-ttu-id="a2aed-110">Il modello viene creato nel blocco del componente `@code` e viene mantenuto in un campo privato ( `exampleModel` ).</span><span class="sxs-lookup"><span data-stu-id="a2aed-110">The model is created in the component's `@code` block and held in a private field (`exampleModel`).</span></span> <span data-ttu-id="a2aed-111">Il campo viene assegnato all' `Model` attributo dell' `<EditForm>` elemento.</span><span class="sxs-lookup"><span data-stu-id="a2aed-111">The field is assigned to the `Model` attribute of the `<EditForm>` element.</span></span>
* <span data-ttu-id="a2aed-112">Le <xref:Microsoft.AspNetCore.Components.Forms.InputText> associazioni del componente `@bind-Value` :</span><span class="sxs-lookup"><span data-stu-id="a2aed-112">The <xref:Microsoft.AspNetCore.Components.Forms.InputText> component's `@bind-Value` binds:</span></span>
  * <span data-ttu-id="a2aed-113">Proprietà del modello ( `exampleModel.Name` ) della <xref:Microsoft.AspNetCore.Components.Forms.InputText> proprietà del componente `Value` .</span><span class="sxs-lookup"><span data-stu-id="a2aed-113">The model property (`exampleModel.Name`) to the <xref:Microsoft.AspNetCore.Components.Forms.InputText> component's `Value` property.</span></span> <span data-ttu-id="a2aed-114">Per ulteriori informazioni sull'associazione di proprietà, vedere <xref:blazor/components/data-binding#binding-with-component-parameters> .</span><span class="sxs-lookup"><span data-stu-id="a2aed-114">For more information on property binding, see <xref:blazor/components/data-binding#binding-with-component-parameters>.</span></span>
  * <span data-ttu-id="a2aed-115">Delegato dell'evento di modifica alla <xref:Microsoft.AspNetCore.Components.Forms.InputText> proprietà del componente `ValueChanged` .</span><span class="sxs-lookup"><span data-stu-id="a2aed-115">A change event delegate to the <xref:Microsoft.AspNetCore.Components.Forms.InputText> component's `ValueChanged` property.</span></span>
* <span data-ttu-id="a2aed-116">Il <xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> [componente validator](#validator-components) connette il supporto della convalida usando le annotazioni dei dati.</span><span class="sxs-lookup"><span data-stu-id="a2aed-116">The <xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> [validator component](#validator-components) attaches validation support using data annotations.</span></span>
* <span data-ttu-id="a2aed-117">Il <xref:Microsoft.AspNetCore.Components.Forms.ValidationSummary> componente riepiloga i messaggi di convalida.</span><span class="sxs-lookup"><span data-stu-id="a2aed-117">The <xref:Microsoft.AspNetCore.Components.Forms.ValidationSummary> component summarizes validation messages.</span></span>
* <span data-ttu-id="a2aed-118">`HandleValidSubmit` viene attivato quando il form viene inviato correttamente (supera la convalida).</span><span class="sxs-lookup"><span data-stu-id="a2aed-118">`HandleValidSubmit` is triggered when the form successfully submits (passes validation).</span></span>

## <a name="built-in-forms-components"></a><span data-ttu-id="a2aed-119">Componenti incorporati dei moduli</span><span class="sxs-lookup"><span data-stu-id="a2aed-119">Built-in forms components</span></span>

<span data-ttu-id="a2aed-120">È disponibile un set di componenti predefiniti per la ricezione e la convalida dell'input dell'utente.</span><span class="sxs-lookup"><span data-stu-id="a2aed-120">A set of built-in components are available to receive and validate user input.</span></span> <span data-ttu-id="a2aed-121">Gli input vengono convalidati quando vengono modificati e quando viene inviato un modulo.</span><span class="sxs-lookup"><span data-stu-id="a2aed-121">Inputs are validated when they're changed and when a form is submitted.</span></span> <span data-ttu-id="a2aed-122">I componenti di input disponibili sono illustrati nella tabella seguente.</span><span class="sxs-lookup"><span data-stu-id="a2aed-122">Available input components are shown in the following table.</span></span>

::: moniker range=">= aspnetcore-5.0"

| <span data-ttu-id="a2aed-123">Componente di input</span><span class="sxs-lookup"><span data-stu-id="a2aed-123">Input component</span></span> | <span data-ttu-id="a2aed-124">Con rendering come&hellip;</span><span class="sxs-lookup"><span data-stu-id="a2aed-124">Rendered as&hellip;</span></span> |
| --------------- | ------------------- |
| <xref:Microsoft.AspNetCore.Components.Forms.InputCheckbox> | `<input type="checkbox">` |
| <xref:Microsoft.AspNetCore.Components.Forms.InputDate%601> | `<input type="date">` |
| [`InputFile`](xref:blazor/file-uploads) | `<input type="file">` |
| <xref:Microsoft.AspNetCore.Components.Forms.InputNumber%601> | `<input type="number">` |
| [`InputRadio`](#radio-buttons) | `<input type="radio">` |
| [`InputRadioGroup`](#radio-buttons) | `<input type="radio">` |
| <xref:Microsoft.AspNetCore.Components.Forms.InputSelect%601> | `<select>` |
| <xref:Microsoft.AspNetCore.Components.Forms.InputText> | `<input>` |
| <xref:Microsoft.AspNetCore.Components.Forms.InputTextArea> | `<textarea>` |

::: moniker-end

::: moniker range="< aspnetcore-5.0"

| <span data-ttu-id="a2aed-125">Componente di input</span><span class="sxs-lookup"><span data-stu-id="a2aed-125">Input component</span></span> | <span data-ttu-id="a2aed-126">Con rendering come&hellip;</span><span class="sxs-lookup"><span data-stu-id="a2aed-126">Rendered as&hellip;</span></span> |
| --------------- | ------------------- |
| <xref:Microsoft.AspNetCore.Components.Forms.InputCheckbox> | `<input type="checkbox">` |
| <xref:Microsoft.AspNetCore.Components.Forms.InputDate%601> | `<input type="date">` |
| <xref:Microsoft.AspNetCore.Components.Forms.InputNumber%601> | `<input type="number">` |
| <xref:Microsoft.AspNetCore.Components.Forms.InputSelect%601> | `<select>` |
| <xref:Microsoft.AspNetCore.Components.Forms.InputText> | `<input>` |
| <xref:Microsoft.AspNetCore.Components.Forms.InputTextArea> | `<textarea>` |

> [!NOTE]
> <span data-ttu-id="a2aed-127">I `InputRadio` `InputRadioGroup` componenti e sono disponibili in ASP.NET Core 5,0 o versione successiva.</span><span class="sxs-lookup"><span data-stu-id="a2aed-127">The `InputRadio` and `InputRadioGroup` components are available in ASP.NET Core 5.0 or later.</span></span> <span data-ttu-id="a2aed-128">Per ulteriori informazioni, selezionare una versione 5,0 o successiva di questo articolo.</span><span class="sxs-lookup"><span data-stu-id="a2aed-128">For more information, select a 5.0 or later version of this article.</span></span>

::: moniker-end

<span data-ttu-id="a2aed-129">Tutti i componenti di input, tra cui <xref:Microsoft.AspNetCore.Components.Forms.EditForm> , supportano attributi arbitrari.</span><span class="sxs-lookup"><span data-stu-id="a2aed-129">All of the input components, including <xref:Microsoft.AspNetCore.Components.Forms.EditForm>, support arbitrary attributes.</span></span> <span data-ttu-id="a2aed-130">Qualsiasi attributo che non corrisponde a un parametro component viene aggiunto all'elemento HTML sottoposto a rendering.</span><span class="sxs-lookup"><span data-stu-id="a2aed-130">Any attribute that doesn't match a component parameter is added to the rendered HTML element.</span></span>

<span data-ttu-id="a2aed-131">I componenti di input forniscono il comportamento predefinito per la convalida quando viene modificato un campo, incluso l'aggiornamento della classe CSS del campo per riflettere lo stato del campo.</span><span class="sxs-lookup"><span data-stu-id="a2aed-131">Input components provide default behavior for validating when a field is changed, including updating the field CSS class to reflect the field state.</span></span> <span data-ttu-id="a2aed-132">Alcuni componenti includono una logica di analisi utile.</span><span class="sxs-lookup"><span data-stu-id="a2aed-132">Some components include useful parsing logic.</span></span> <span data-ttu-id="a2aed-133">Ad esempio, <xref:Microsoft.AspNetCore.Components.Forms.InputDate%601> e <xref:Microsoft.AspNetCore.Components.Forms.InputNumber%601> gestiscono correttamente i valori non analizzabili registrando valori non analizzabili come errori di convalida.</span><span class="sxs-lookup"><span data-stu-id="a2aed-133">For example, <xref:Microsoft.AspNetCore.Components.Forms.InputDate%601> and <xref:Microsoft.AspNetCore.Components.Forms.InputNumber%601> handle unparseable values gracefully by registering unparseable values as validation errors.</span></span> <span data-ttu-id="a2aed-134">I tipi che possono accettare valori null supportano anche il supporto di valori null del campo di destinazione, ad esempio `int?` .</span><span class="sxs-lookup"><span data-stu-id="a2aed-134">Types that can accept null values also support nullability of the target field (for example, `int?`).</span></span>

<span data-ttu-id="a2aed-135">Il `Starship` tipo seguente definisce la logica di convalida usando un set di proprietà e annotazioni di dati più ampio rispetto a quello precedente `ExampleModel` :</span><span class="sxs-lookup"><span data-stu-id="a2aed-135">The following `Starship` type defines validation logic using a larger set of properties and data annotations than the earlier `ExampleModel`:</span></span>

```csharp
using System;
using System.ComponentModel.DataAnnotations;

public class Starship
{
    [Required]
    [StringLength(16, ErrorMessage = "Identifier too long (16 character limit).")]
    public string Identifier { get; set; }

    public string Description { get; set; }

    [Required]
    public string Classification { get; set; }

    [Range(1, 100000, ErrorMessage = "Accommodation invalid (1-100000).")]
    public int MaximumAccommodation { get; set; }

    [Required]
    [Range(typeof(bool), "true", "true", 
        ErrorMessage = "This form disallows unapproved ships.")]
    public bool IsValidatedDesign { get; set; }

    [Required]
    public DateTime ProductionDate { get; set; }
}
```

<span data-ttu-id="a2aed-136">Nell'esempio precedente `Description` è facoltativo perché non sono presenti annotazioni di dati.</span><span class="sxs-lookup"><span data-stu-id="a2aed-136">In the preceding example, `Description` is optional because no data annotations are present.</span></span>

<span data-ttu-id="a2aed-137">Il form seguente convalida l'input dell'utente utilizzando la convalida definita nel `Starship` modello:</span><span class="sxs-lookup"><span data-stu-id="a2aed-137">The following form validates user input using the validation defined in the `Starship` model:</span></span>

```razor
@page "/FormsValidation"

<h1>Starfleet Starship Database</h1>

<h2>New Ship Entry Form</h2>

<EditForm Model="@starship" OnValidSubmit="@HandleValidSubmit">
    <DataAnnotationsValidator />
    <ValidationSummary />

    <p>
        <label>
            Identifier:
            <InputText @bind-Value="starship.Identifier" />
        </label>
    </p>
    <p>
        <label>
            Description (optional):
            <InputTextArea @bind-Value="starship.Description" />
        </label>
    </p>
    <p>
        <label>
            Primary Classification:
            <InputSelect @bind-Value="starship.Classification">
                <option value="">Select classification ...</option>
                <option value="Exploration">Exploration</option>
                <option value="Diplomacy">Diplomacy</option>
                <option value="Defense">Defense</option>
            </InputSelect>
        </label>
    </p>
    <p>
        <label>
            Maximum Accommodation:
            <InputNumber @bind-Value="starship.MaximumAccommodation" />
        </label>
    </p>
    <p>
        <label>
            Engineering Approval:
            <InputCheckbox @bind-Value="starship.IsValidatedDesign" />
        </label>
    </p>
    <p>
        <label>
            Production Date:
            <InputDate @bind-Value="starship.ProductionDate" />
        </label>
    </p>

    <button type="submit">Submit</button>

    <p>
        <a href="http://www.startrek.com/">Star Trek</a>, 
        &copy;1966-2019 CBS Studios, Inc. and 
        <a href="https://www.paramount.com">Paramount Pictures</a>
    </p>
</EditForm>

@code {
    private Starship starship = new Starship() { ProductionDate = DateTime.UtcNow };

    private void HandleValidSubmit()
    {
        ...
    }
}
```

<span data-ttu-id="a2aed-138"><xref:Microsoft.AspNetCore.Components.Forms.EditForm>Crea un oggetto <xref:Microsoft.AspNetCore.Components.Forms.EditContext> come [valore](xref:blazor/components/cascading-values-and-parameters) di propagazione che tiene traccia dei metadati relativi al processo di modifica, inclusi i campi modificati e i messaggi di convalida correnti.</span><span class="sxs-lookup"><span data-stu-id="a2aed-138">The <xref:Microsoft.AspNetCore.Components.Forms.EditForm> creates an <xref:Microsoft.AspNetCore.Components.Forms.EditContext> as a [cascading value](xref:blazor/components/cascading-values-and-parameters) that tracks metadata about the edit process, including which fields have been modified and the current validation messages.</span></span>

<span data-ttu-id="a2aed-139">Assegnare **un** oggetto <xref:Microsoft.AspNetCore.Components.Forms.EditContext> **o** <xref:Microsoft.AspNetCore.Components.Forms.EditForm.Model?displayProperty=nameWithType> a un oggetto <xref:Microsoft.AspNetCore.Components.Forms.EditForm> .</span><span class="sxs-lookup"><span data-stu-id="a2aed-139">Assign **either** an <xref:Microsoft.AspNetCore.Components.Forms.EditContext> **or** an <xref:Microsoft.AspNetCore.Components.Forms.EditForm.Model?displayProperty=nameWithType> to an <xref:Microsoft.AspNetCore.Components.Forms.EditForm>.</span></span> <span data-ttu-id="a2aed-140">L'assegnazione di entrambi non è supportata e genera un **errore di runtime**.</span><span class="sxs-lookup"><span data-stu-id="a2aed-140">Assignment of both isn't supported and generates a **runtime error**.</span></span>

<span data-ttu-id="a2aed-141"><xref:Microsoft.AspNetCore.Components.Forms.EditForm>Fornisce gli eventi pratici per l'invio di form valido e non valido:</span><span class="sxs-lookup"><span data-stu-id="a2aed-141">The <xref:Microsoft.AspNetCore.Components.Forms.EditForm> provides convenient events for valid and invalid form submission:</span></span>

* <xref:Microsoft.AspNetCore.Components.Forms.EditForm.OnValidSubmit>
* <xref:Microsoft.AspNetCore.Components.Forms.EditForm.OnInvalidSubmit>

<span data-ttu-id="a2aed-142">Usare <xref:Microsoft.AspNetCore.Components.Forms.EditForm.OnSubmit> per usare codice personalizzato per attivare la convalida e verificare i valori dei campi.</span><span class="sxs-lookup"><span data-stu-id="a2aed-142">Use <xref:Microsoft.AspNetCore.Components.Forms.EditForm.OnSubmit> to use custom code to trigger validation and check field values.</span></span>

<span data-ttu-id="a2aed-143">Nell'esempio seguente:</span><span class="sxs-lookup"><span data-stu-id="a2aed-143">In the following example:</span></span>

* <span data-ttu-id="a2aed-144">Il `HandleSubmit` metodo viene eseguito quando **`Submit`** si seleziona il pulsante.</span><span class="sxs-lookup"><span data-stu-id="a2aed-144">The `HandleSubmit` method executes when the **`Submit`** button is selected.</span></span>
* <span data-ttu-id="a2aed-145">Il modulo viene convalidato chiamando <xref:Microsoft.AspNetCore.Components.Forms.EditContext.Validate%2A?displayProperty=nameWithType> .</span><span class="sxs-lookup"><span data-stu-id="a2aed-145">The form is validated by calling <xref:Microsoft.AspNetCore.Components.Forms.EditContext.Validate%2A?displayProperty=nameWithType>.</span></span>
* <span data-ttu-id="a2aed-146">Il codice aggiuntivo viene eseguito a seconda del risultato della convalida.</span><span class="sxs-lookup"><span data-stu-id="a2aed-146">Additional code is executed depending on the validation result.</span></span> <span data-ttu-id="a2aed-147">Inserire la logica di business nel metodo assegnato a <xref:Microsoft.AspNetCore.Components.Forms.EditForm.OnSubmit> .</span><span class="sxs-lookup"><span data-stu-id="a2aed-147">Place business logic in the method assigned to <xref:Microsoft.AspNetCore.Components.Forms.EditForm.OnSubmit>.</span></span>

```razor
<EditForm EditContext="@editContext" OnSubmit="@HandleSubmit">

    ...

    <button type="submit">Submit</button>
</EditForm>

@code {
    private Starship starship = new Starship() { ProductionDate = DateTime.UtcNow };
    private EditContext editContext;

    protected override void OnInitialized()
    {
        editContext = new EditContext(starship);
    }

    private async Task HandleSubmit()
    {
        var isValid = editContext.Validate();

        if (isValid)
        {
            ...
        }
        else
        {
            ...
        }
    }
}
```

> [!NOTE]
> <span data-ttu-id="a2aed-148">L'API del Framework non esiste per cancellare i messaggi di convalida direttamente da un <xref:Microsoft.AspNetCore.Components.Forms.EditContext> .</span><span class="sxs-lookup"><span data-stu-id="a2aed-148">Framework API doesn't exist to clear validation messages directly from an <xref:Microsoft.AspNetCore.Components.Forms.EditContext>.</span></span> <span data-ttu-id="a2aed-149">Pertanto, in genere non è consigliabile aggiungere messaggi di convalida a un nuovo <xref:Microsoft.AspNetCore.Components.Forms.ValidationMessageStore> in un modulo.</span><span class="sxs-lookup"><span data-stu-id="a2aed-149">Therefore, we don't generally recommend adding validation messages to a new <xref:Microsoft.AspNetCore.Components.Forms.ValidationMessageStore> in a form.</span></span> <span data-ttu-id="a2aed-150">Per gestire i messaggi di convalida, usare un [componente validator](#validator-components) con il [codice di convalida della logica di business](#business-logic-validation), come descritto in questo articolo.</span><span class="sxs-lookup"><span data-stu-id="a2aed-150">To manage validation messages, use a [validator component](#validator-components) with your [business logic validation code](#business-logic-validation), as described in this article.</span></span>

::: moniker range=">= aspnetcore-5.0"

## <a name="display-name-support"></a><span data-ttu-id="a2aed-151">Supporto del nome visualizzato</span><span class="sxs-lookup"><span data-stu-id="a2aed-151">Display name support</span></span>

<span data-ttu-id="a2aed-152">*Questa sezione si applica a ASP.NET Core in .NET 5 Release Candidate 1 (RC1) o versioni successive.*</span><span class="sxs-lookup"><span data-stu-id="a2aed-152">*This section applies to ASP.NET Core in .NET 5 Release Candidate 1 (RC1) or later.*</span></span>

<span data-ttu-id="a2aed-153">I seguenti componenti predefiniti supportano i nomi visualizzati con il `DisplayName` parametro:</span><span class="sxs-lookup"><span data-stu-id="a2aed-153">The following built-in components support display names with the `DisplayName` parameter:</span></span>

* <xref:Microsoft.AspNetCore.Components.Forms.InputDate%601>
* <xref:Microsoft.AspNetCore.Components.Forms.InputNumber%601>
* <xref:Microsoft.AspNetCore.Components.Forms.InputSelect%601>

<span data-ttu-id="a2aed-154">Nell'esempio di `InputDate` componente seguente:</span><span class="sxs-lookup"><span data-stu-id="a2aed-154">In the following `InputDate` component example:</span></span>

* <span data-ttu-id="a2aed-155">Il nome visualizzato ( `DisplayName` ) è impostato su `birthday` .</span><span class="sxs-lookup"><span data-stu-id="a2aed-155">The display name (`DisplayName`) is set to `birthday`.</span></span>
* <span data-ttu-id="a2aed-156">Il componente è associato alla `BirthDate` proprietà come `DateTime` tipo.</span><span class="sxs-lookup"><span data-stu-id="a2aed-156">The component is bound to the `BirthDate` property as a `DateTime` type.</span></span>

```razor
<InputDate @bind-Value="BirthDate" DisplayName="birthday" />

@code {
    public DateTime BirthDate { get; set; }
}
```

<span data-ttu-id="a2aed-157">Se l'utente non fornisce un valore di data, l'errore di convalida viene visualizzato come segue:</span><span class="sxs-lookup"><span data-stu-id="a2aed-157">If the user doesn't provide a date value, the validation error appears as:</span></span>

```
The birthday must be a date.
```

::: moniker-end

## <a name="validator-components"></a><span data-ttu-id="a2aed-158">Componenti validator</span><span class="sxs-lookup"><span data-stu-id="a2aed-158">Validator components</span></span>

<span data-ttu-id="a2aed-159">I componenti validator supportano la convalida dei form gestendo un oggetto <xref:Microsoft.AspNetCore.Components.Forms.ValidationMessageStore> per un modulo <xref:Microsoft.AspNetCore.Components.Forms.EditContext> .</span><span class="sxs-lookup"><span data-stu-id="a2aed-159">Validator components support form validation by managing a <xref:Microsoft.AspNetCore.Components.Forms.ValidationMessageStore> for a form's <xref:Microsoft.AspNetCore.Components.Forms.EditContext>.</span></span>

<span data-ttu-id="a2aed-160">Il Blazor Framework fornisce il <xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> componente per l'associazione del supporto di convalida ai moduli in base agli [attributi di convalida (annotazioni dei dati)](xref:mvc/models/validation#validation-attributes).</span><span class="sxs-lookup"><span data-stu-id="a2aed-160">The Blazor framework provides the <xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> component to attach validation support to forms based on [validation attributes (data annotations)](xref:mvc/models/validation#validation-attributes).</span></span> <span data-ttu-id="a2aed-161">Creare componenti validator personalizzati per elaborare i messaggi di convalida per formati diversi nella stessa pagina o nello stesso formato in diversi passaggi di elaborazione del modulo, ad esempio convalida sul lato client seguita dalla convalida lato server.</span><span class="sxs-lookup"><span data-stu-id="a2aed-161">Create custom validator components to process validation messages for different forms on the same page or the same form at different steps of form processing, for example client-side validation followed by server-side validation.</span></span> <span data-ttu-id="a2aed-162">L'esempio di componente validator illustrato in questa sezione, `CustomValidator` , viene usato nelle sezioni seguenti di questo articolo:</span><span class="sxs-lookup"><span data-stu-id="a2aed-162">The validator component example shown in this section, `CustomValidator`, is used in the following sections of this article:</span></span>

* [<span data-ttu-id="a2aed-163">Convalida della logica di business</span><span class="sxs-lookup"><span data-stu-id="a2aed-163">Business logic validation</span></span>](#business-logic-validation)
* [<span data-ttu-id="a2aed-164">Convalida server</span><span class="sxs-lookup"><span data-stu-id="a2aed-164">Server validation</span></span>](#server-validation)

> [!NOTE]
> <span data-ttu-id="a2aed-165">In molti casi, è possibile utilizzare gli attributi di convalida delle annotazioni dei dati personalizzati anziché i componenti del validator personalizzato.</span><span class="sxs-lookup"><span data-stu-id="a2aed-165">Custom data annotation validation attributes can be used instead of custom validator components in many cases.</span></span> <span data-ttu-id="a2aed-166">Gli attributi personalizzati applicati al modello del modulo vengono attivati con l'utilizzo del <xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> componente.</span><span class="sxs-lookup"><span data-stu-id="a2aed-166">Custom attributes applied to the form's model activate with the use of the <xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> component.</span></span> <span data-ttu-id="a2aed-167">Se utilizzata con la convalida sul lato server, tutti gli attributi personalizzati applicati al modello devono essere eseguibili nel server.</span><span class="sxs-lookup"><span data-stu-id="a2aed-167">When used with server-side validation, any custom attributes applied to the model must be executable on the server.</span></span> <span data-ttu-id="a2aed-168">Per altre informazioni, vedere <xref:mvc/models/validation#alternatives-to-built-in-attributes>.</span><span class="sxs-lookup"><span data-stu-id="a2aed-168">For more information, see <xref:mvc/models/validation#alternatives-to-built-in-attributes>.</span></span>

<span data-ttu-id="a2aed-169">Creare un componente validator da <xref:Microsoft.AspNetCore.Components.ComponentBase> :</span><span class="sxs-lookup"><span data-stu-id="a2aed-169">Create a validator component from <xref:Microsoft.AspNetCore.Components.ComponentBase>:</span></span>

* <span data-ttu-id="a2aed-170">Il modulo è <xref:Microsoft.AspNetCore.Components.Forms.EditContext> un [parametro](xref:blazor/components/cascading-values-and-parameters) di propagazione del componente.</span><span class="sxs-lookup"><span data-stu-id="a2aed-170">The form's <xref:Microsoft.AspNetCore.Components.Forms.EditContext> is a [cascading parameter](xref:blazor/components/cascading-values-and-parameters) of the component.</span></span>
* <span data-ttu-id="a2aed-171">Quando il componente validator viene inizializzato, viene creato un nuovo oggetto <xref:Microsoft.AspNetCore.Components.Forms.ValidationMessageStore> per gestire un elenco corrente di errori di form.</span><span class="sxs-lookup"><span data-stu-id="a2aed-171">When the validator component is initialized, a new <xref:Microsoft.AspNetCore.Components.Forms.ValidationMessageStore> is created to maintain a current list of form errors.</span></span>
* <span data-ttu-id="a2aed-172">L'archivio di messaggi riceve errori quando il codice dello sviluppatore nel componente del modulo chiama il `DisplayErrors` metodo.</span><span class="sxs-lookup"><span data-stu-id="a2aed-172">The message store receives errors when developer code in the form's component calls the `DisplayErrors` method.</span></span> <span data-ttu-id="a2aed-173">Gli errori vengono passati al `DisplayErrors` metodo in un oggetto [`Dictionary<string, List<string>>`](xref:System.Collections.Generic.Dictionary`2) .</span><span class="sxs-lookup"><span data-stu-id="a2aed-173">The errors are passed to the `DisplayErrors` method in a [`Dictionary<string, List<string>>`](xref:System.Collections.Generic.Dictionary`2).</span></span> <span data-ttu-id="a2aed-174">Nel dizionario la chiave è il nome del campo del form che contiene uno o più errori.</span><span class="sxs-lookup"><span data-stu-id="a2aed-174">In the dictionary, the key is the name of the form field that has one or more errors.</span></span> <span data-ttu-id="a2aed-175">Il valore è l'elenco errori.</span><span class="sxs-lookup"><span data-stu-id="a2aed-175">The value is the error list.</span></span>
* <span data-ttu-id="a2aed-176">I messaggi vengono cancellati quando si verifica una delle condizioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="a2aed-176">Messages are cleared when any of the following have occurred:</span></span>
  * <span data-ttu-id="a2aed-177">La convalida viene richiesta su <xref:Microsoft.AspNetCore.Components.Forms.EditContext> quando <xref:Microsoft.AspNetCore.Components.Forms.EditContext.OnValidationRequested> viene generato l'evento.</span><span class="sxs-lookup"><span data-stu-id="a2aed-177">Validation is requested on the <xref:Microsoft.AspNetCore.Components.Forms.EditContext> when the <xref:Microsoft.AspNetCore.Components.Forms.EditContext.OnValidationRequested> event is raised.</span></span> <span data-ttu-id="a2aed-178">Tutti gli errori vengono cancellati.</span><span class="sxs-lookup"><span data-stu-id="a2aed-178">All of the errors are cleared.</span></span>
  * <span data-ttu-id="a2aed-179">Un campo viene modificato nel form quando <xref:Microsoft.AspNetCore.Components.Forms.EditContext.OnFieldChanged> viene generato l'evento.</span><span class="sxs-lookup"><span data-stu-id="a2aed-179">A field changes in the form when the <xref:Microsoft.AspNetCore.Components.Forms.EditContext.OnFieldChanged> event is raised.</span></span> <span data-ttu-id="a2aed-180">Vengono cancellati solo gli errori per il campo.</span><span class="sxs-lookup"><span data-stu-id="a2aed-180">Only the errors for the field are cleared.</span></span>
  * <span data-ttu-id="a2aed-181">Il `ClearErrors` metodo viene chiamato dal codice dello sviluppatore.</span><span class="sxs-lookup"><span data-stu-id="a2aed-181">The `ClearErrors` method is called by developer code.</span></span> <span data-ttu-id="a2aed-182">Tutti gli errori vengono cancellati.</span><span class="sxs-lookup"><span data-stu-id="a2aed-182">All of the errors are cleared.</span></span>

```csharp
using System;
using System.Collections.Generic;
using Microsoft.AspNetCore.Components;
using Microsoft.AspNetCore.Components.Forms;

public class CustomValidator : ComponentBase
{
    private ValidationMessageStore messageStore;

    [CascadingParameter]
    private EditContext CurrentEditContext { get; set; }

    protected override void OnInitialized()
    {
        if (CurrentEditContext == null)
        {
            throw new InvalidOperationException(
                $"{nameof(CustomValidator)} requires a cascading " +
                $"parameter of type {nameof(EditContext)}. " +
                $"For example, you can use {nameof(CustomValidator)} " +
                $"inside an {nameof(EditForm)}.");
        }

        messageStore = new ValidationMessageStore(CurrentEditContext);

        CurrentEditContext.OnValidationRequested += (s, e) => 
            messageStore.Clear();
        CurrentEditContext.OnFieldChanged += (s, e) => 
            messageStore.Clear(e.FieldIdentifier);
    }

    public void DisplayErrors(Dictionary<string, List<string>> errors)
    {
        foreach (var err in errors)
        {
            messageStore.Add(CurrentEditContext.Field(err.Key), err.Value);
        }

        CurrentEditContext.NotifyValidationStateChanged();
    }

    public void ClearErrors()
    {
        messageStore.Clear();
        CurrentEditContext.NotifyValidationStateChanged();
    }
}
```

> [!NOTE]
> <span data-ttu-id="a2aed-183">Le espressioni lambda anonime sono gestori eventi registrati per <xref:Microsoft.AspNetCore.Components.Forms.EditContext.OnValidationRequested> e <xref:Microsoft.AspNetCore.Components.Forms.EditContext.OnFieldChanged> nell'esempio precedente.</span><span class="sxs-lookup"><span data-stu-id="a2aed-183">Anonymous lambda expressions are registered event handlers for <xref:Microsoft.AspNetCore.Components.Forms.EditContext.OnValidationRequested> and <xref:Microsoft.AspNetCore.Components.Forms.EditContext.OnFieldChanged> in the preceding example.</span></span> <span data-ttu-id="a2aed-184">In questo scenario non è necessario implementare <xref:System.IDisposable> e annullare la sottoscrizione dei delegati di evento.</span><span class="sxs-lookup"><span data-stu-id="a2aed-184">It isn't necessary to implement <xref:System.IDisposable> and unsubscribe the event delegates in this scenario.</span></span> <span data-ttu-id="a2aed-185">Per altre informazioni, vedere <xref:blazor/components/lifecycle#component-disposal-with-idisposable>.</span><span class="sxs-lookup"><span data-stu-id="a2aed-185">For more information, see <xref:blazor/components/lifecycle#component-disposal-with-idisposable>.</span></span>

## <a name="business-logic-validation"></a><span data-ttu-id="a2aed-186">Convalida della logica di business</span><span class="sxs-lookup"><span data-stu-id="a2aed-186">Business logic validation</span></span>

<span data-ttu-id="a2aed-187">La convalida della logica di business può essere eseguita con un [componente validator](#validator-components) che riceve errori di form in un dizionario.</span><span class="sxs-lookup"><span data-stu-id="a2aed-187">Business logic validation can be accomplished with a [validator component](#validator-components) that receives form errors in a dictionary.</span></span>

<span data-ttu-id="a2aed-188">Nell'esempio seguente:</span><span class="sxs-lookup"><span data-stu-id="a2aed-188">In the following example:</span></span>

* <span data-ttu-id="a2aed-189">`CustomValidator`Viene usato il componente della sezione [componenti validator](#validator-components) di questo articolo.</span><span class="sxs-lookup"><span data-stu-id="a2aed-189">The `CustomValidator` component from the [Validator components](#validator-components) section of this article is used.</span></span>
* <span data-ttu-id="a2aed-190">Per la convalida è necessario un valore per la descrizione della nave ( `Description` ) se l'utente seleziona la `Defense` classificazione Ship ( `Classification` ).</span><span class="sxs-lookup"><span data-stu-id="a2aed-190">The validation requires a value for the ship's description (`Description`) if the user selects the `Defense` ship classification (`Classification`).</span></span>

<span data-ttu-id="a2aed-191">Quando i messaggi di convalida vengono impostati nel componente, vengono aggiunti ai validator <xref:Microsoft.AspNetCore.Components.Forms.ValidationMessageStore> e mostrati in <xref:Microsoft.AspNetCore.Components.Forms.EditForm> :</span><span class="sxs-lookup"><span data-stu-id="a2aed-191">When validation messages are set in the component, they're added to the validator's <xref:Microsoft.AspNetCore.Components.Forms.ValidationMessageStore> and shown in the <xref:Microsoft.AspNetCore.Components.Forms.EditForm>:</span></span>

```razor
@page "/FormsValidation"

<h1>Starfleet Starship Database</h1>

<h2>New Ship Entry Form</h2>

<EditForm Model="@starship" OnValidSubmit="@HandleValidSubmit">
    <DataAnnotationsValidator />
    <CustomValidator @ref="customValidator" />
    <ValidationSummary />

    ...

</EditForm>

@code {
    private CustomValidator customValidator;
    private Starship starship = new Starship() { ProductionDate = DateTime.UtcNow };

    private void HandleValidSubmit()
    {
        customValidator.ClearErrors();

        var errors = new Dictionary<string, List<string>>();

        if (starship.Classification == "Defense" &&
                string.IsNullOrEmpty(starship.Description))
        {
            errors.Add(nameof(starship.Description),
                new List<string>() { "For a 'Defense' ship classification, " +
                "'Description' is required." });
        }

        if (errors.Count() > 0)
        {
            customValidator.DisplayErrors(errors);
        }
        else
        {
            // Process the form
        }
    }
}
```

> [!NOTE]
> <span data-ttu-id="a2aed-192">In alternativa all'uso di [componenti di convalida](#validator-components), è possibile usare gli attributi di convalida dell'annotazione dei dati.</span><span class="sxs-lookup"><span data-stu-id="a2aed-192">As an alternative to using [validation components](#validator-components), data annotation validation attributes can be used.</span></span> <span data-ttu-id="a2aed-193">Gli attributi personalizzati applicati al modello del modulo vengono attivati con l'utilizzo del <xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> componente.</span><span class="sxs-lookup"><span data-stu-id="a2aed-193">Custom attributes applied to the form's model activate with the use of the <xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> component.</span></span> <span data-ttu-id="a2aed-194">Quando viene utilizzato con la convalida sul lato server, gli attributi devono essere eseguibili nel server.</span><span class="sxs-lookup"><span data-stu-id="a2aed-194">When used with server-side validation, the attributes must be executable on the server.</span></span> <span data-ttu-id="a2aed-195">Per altre informazioni, vedere <xref:mvc/models/validation#alternatives-to-built-in-attributes>.</span><span class="sxs-lookup"><span data-stu-id="a2aed-195">For more information, see <xref:mvc/models/validation#alternatives-to-built-in-attributes>.</span></span>

## <a name="server-validation"></a><span data-ttu-id="a2aed-196">Convalida server</span><span class="sxs-lookup"><span data-stu-id="a2aed-196">Server validation</span></span>

<span data-ttu-id="a2aed-197">La convalida del server può essere eseguita con un [componente validator](#validator-components)server:</span><span class="sxs-lookup"><span data-stu-id="a2aed-197">Server validation can be accomplished with a server [validator component](#validator-components):</span></span>

* <span data-ttu-id="a2aed-198">Elaborare la convalida lato client nel form con il <xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> componente.</span><span class="sxs-lookup"><span data-stu-id="a2aed-198">Process client-side validation in the form with the <xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> component.</span></span>
* <span data-ttu-id="a2aed-199">Quando il modulo passa la convalida lato client ( <xref:Microsoft.AspNetCore.Components.Forms.EditForm.OnValidSubmit> viene chiamato), inviare <xref:Microsoft.AspNetCore.Components.Forms.EditContext.Model?displayProperty=nameWithType> a un'API del server back-end per l'elaborazione del modulo.</span><span class="sxs-lookup"><span data-stu-id="a2aed-199">When the form passes client-side validation (<xref:Microsoft.AspNetCore.Components.Forms.EditForm.OnValidSubmit> is called), send the <xref:Microsoft.AspNetCore.Components.Forms.EditContext.Model?displayProperty=nameWithType> to a backend server API for form processing.</span></span>
* <span data-ttu-id="a2aed-200">Convalida del modello di processo sul server.</span><span class="sxs-lookup"><span data-stu-id="a2aed-200">Process model validation on the server.</span></span>
* <span data-ttu-id="a2aed-201">L'API server include sia la convalida delle annotazioni dei dati del Framework incorporata che la logica di convalida personalizzata fornita dallo sviluppatore.</span><span class="sxs-lookup"><span data-stu-id="a2aed-201">The server API includes both the built-in framework data annotations validation and custom validation logic supplied by the developer.</span></span> <span data-ttu-id="a2aed-202">Se la convalida passa sul server, elaborare il modulo e restituire un codice di stato di esito positivo (*200-OK*).</span><span class="sxs-lookup"><span data-stu-id="a2aed-202">If validation passes on the server, process the form and send back a success status code (*200 - OK*).</span></span> <span data-ttu-id="a2aed-203">Se la convalida ha esito negativo, restituire un codice di stato di errore (*400-richiesta* non valida) e gli errori di convalida dei campi.</span><span class="sxs-lookup"><span data-stu-id="a2aed-203">If validation fails, return a failure status code (*400 - Bad Request*) and the field validation errors.</span></span>
* <span data-ttu-id="a2aed-204">Disabilitare il modulo in caso di esito positivo o visualizzare gli errori.</span><span class="sxs-lookup"><span data-stu-id="a2aed-204">Either disable the form on success or display the errors.</span></span>

<span data-ttu-id="a2aed-205">L'esempio seguente è basato su:</span><span class="sxs-lookup"><span data-stu-id="a2aed-205">The following example is based on:</span></span>

* <span data-ttu-id="a2aed-206">Una soluzione ospitata Blazor WebAssembly creata dal [ Blazor WebAssembly modello di progetto](xref:blazor/project-structure).</span><span class="sxs-lookup"><span data-stu-id="a2aed-206">A hosted Blazor WebAssembly solution created from the [Blazor WebAssembly project template](xref:blazor/project-structure).</span></span> <span data-ttu-id="a2aed-207">Questo esempio può essere usato con qualsiasi soluzione ospitata protetta Blazor descritta nella pagina [ Identity relativa alla sicurezza e alla documentazione](xref:blazor/security/webassembly/index#implementation-guidance).</span><span class="sxs-lookup"><span data-stu-id="a2aed-207">The example can be used with any of the secure hosted Blazor solutions described in the [Security and Identity documentation](xref:blazor/security/webassembly/index#implementation-guidance).</span></span>
* <span data-ttu-id="a2aed-208">Il modulo del *database Starship astronave* nell'esempio precedente della sezione relativa ai [componenti dei moduli incorporati](#built-in-forms-components) .</span><span class="sxs-lookup"><span data-stu-id="a2aed-208">The *Starfleet Starship Database* form example in the preceding [Built-in forms components](#built-in-forms-components) section.</span></span>
* <span data-ttu-id="a2aed-209">BlazorComponente del Framework <xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> .</span><span class="sxs-lookup"><span data-stu-id="a2aed-209">The Blazor framework's <xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> component.</span></span>
* <span data-ttu-id="a2aed-210">`CustomValidator`Componente visualizzato nella sezione [componenti validator](#validator-components) .</span><span class="sxs-lookup"><span data-stu-id="a2aed-210">The `CustomValidator` component shown in the [Validator components](#validator-components) section.</span></span>

<span data-ttu-id="a2aed-211">Nell'esempio seguente, l'API del server verifica che venga fornito un valore per la descrizione della nave ( `Description` ) se l'utente seleziona la `Defense` classificazione Ship ( `Classification` ).</span><span class="sxs-lookup"><span data-stu-id="a2aed-211">In the following example, the server API validates that a value is provided for the ship's description (`Description`) if the user selects the `Defense` ship classification (`Classification`).</span></span>

<span data-ttu-id="a2aed-212">Inserire il `Starship` modello nel progetto della soluzione in `Shared` modo che entrambe le app client e server possano usare il modello.</span><span class="sxs-lookup"><span data-stu-id="a2aed-212">Place the `Starship` model into the solution's `Shared` project so that both the client and server apps can use the model.</span></span> <span data-ttu-id="a2aed-213">Poiché il modello richiede le annotazioni dei dati, aggiungere un riferimento al pacchetto per [`System.ComponentModel.Annotations`](https://www.nuget.org/packages/System.ComponentModel.Annotations) al `Shared` file di progetto del progetto:</span><span class="sxs-lookup"><span data-stu-id="a2aed-213">Since the model requires data annotations, add a package reference for [`System.ComponentModel.Annotations`](https://www.nuget.org/packages/System.ComponentModel.Annotations) to the `Shared` project's project file:</span></span>

```xml
<ItemGroup>
  <PackageReference Include="System.ComponentModel.Annotations" Version="{VERSION}" />
</ItemGroup>
```

<span data-ttu-id="a2aed-214">Per determinare la versione non di anteprima più recente del pacchetto, vedere la **cronologia delle versioni** del pacchetto in [NuGet.org](https://www.nuget.org/packages/System.ComponentModel.Annotations).</span><span class="sxs-lookup"><span data-stu-id="a2aed-214">To determine the latest non-preview version of the package, see the package **Version History** at [NuGet.org](https://www.nuget.org/packages/System.ComponentModel.Annotations).</span></span>

<span data-ttu-id="a2aed-215">Nel progetto API server aggiungere un controller per elaborare le richieste di convalida dell'astronave ( `Controllers/StarshipValidation.cs` ) e restituire i messaggi di convalida non riusciti:</span><span class="sxs-lookup"><span data-stu-id="a2aed-215">In the server API project, add a controller to process starship validation requests (`Controllers/StarshipValidation.cs`) and return failed validation messages:</span></span>

```csharp
using System;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Extensions.Logging;
using BlazorSample.Shared;

namespace {ASSEMBLY NAME}.Controllers
{
    [Authorize]
    [ApiController]
    [Route("[controller]")]
    public class StarshipValidationController : ControllerBase
    {
        private readonly ILogger<StarshipValidationController> logger;

        public StarshipValidationController(
            ILogger<StarshipValidationController> logger)
        {
            this.logger = logger;
        }

        [HttpPost]
        public async Task<IActionResult> Post(Starship starship)
        {
            try
            {
                if (starship.Classification == "Defense" && 
                    string.IsNullOrEmpty(starship.Description))
                {
                    ModelState.AddModelError(nameof(starship.Description),
                        "For a 'Defense' ship " +
                        "classification, 'Description' is required.");
                }
                else
                {
                    // Process the form asynchronously
                    // async ...

                    return Ok(ModelState);
                }
            }
            catch (Exception ex)
            {
                logger.LogError("Validation Error: {Message}", ex.Message);
            }

            return BadRequest(ModelState);
        }
    }
}
```

<span data-ttu-id="a2aed-216">Nell'esempio precedente, il segnaposto `{ASSEMBLY NAME}` è il nome dell'assembly dell'app (ad esempio, `BlazorSample.Server` ).</span><span class="sxs-lookup"><span data-stu-id="a2aed-216">In the preceding example, the placeholder `{ASSEMBLY NAME}` is the app's assembly name (for example, `BlazorSample.Server`).</span></span>

<span data-ttu-id="a2aed-217">Quando si verifica un errore di convalida dell'associazione di modelli nel server, un [`ApiController`](xref:web-api/index) ( <xref:Microsoft.AspNetCore.Mvc.ApiControllerAttribute> ) restituisce normalmente una [risposta di richiesta non valida predefinita](xref:web-api/index#default-badrequest-response) con un oggetto <xref:Microsoft.AspNetCore.Mvc.ValidationProblemDetails> .</span><span class="sxs-lookup"><span data-stu-id="a2aed-217">When a model binding validation error occurs on the server, an [`ApiController`](xref:web-api/index) (<xref:Microsoft.AspNetCore.Mvc.ApiControllerAttribute>) normally returns a [default bad request response](xref:web-api/index#default-badrequest-response) with a <xref:Microsoft.AspNetCore.Mvc.ValidationProblemDetails>.</span></span> <span data-ttu-id="a2aed-218">La risposta contiene più dati dei soli errori di convalida, come illustrato nell'esempio seguente quando tutti i campi del modulo del *database Starship della flotta stellare* non vengono inviati e il modulo non viene convalidato:</span><span class="sxs-lookup"><span data-stu-id="a2aed-218">The response contains more data than just the validation errors, as shown in the following example when all of the fields of the *Starfleet Starship Database* form aren't submitted and the form fails validation:</span></span>

```json
{
  "title": "One or more validation errors occurred.",
  "status": 400,
  "errors": {
    "Identifier": ["The Identifier field is required."],
    "Classification": ["The Classification field is required."],
    "IsValidatedDesign": ["This form disallows unapproved ships."],
    "MaximumAccommodation": ["Accommodation invalid (1-100000)."]
  }
}
```

<span data-ttu-id="a2aed-219">Se l'API del server restituisce la risposta JSON predefinita precedente, il client può analizzare la risposta per ottenere gli elementi figlio del `errors` nodo.</span><span class="sxs-lookup"><span data-stu-id="a2aed-219">If the server API returns the preceding default JSON response, it's possible for the client to parse the response to obtain the children of the `errors` node.</span></span> <span data-ttu-id="a2aed-220">Tuttavia, non è consigliabile analizzare il file.</span><span class="sxs-lookup"><span data-stu-id="a2aed-220">However, it's inconvenient to parse the file.</span></span> <span data-ttu-id="a2aed-221">L'analisi del codice JSON richiede codice aggiuntivo dopo <xref:System.Net.Http.Json.HttpContentJsonExtensions.ReadFromJsonAsync%2A> la chiamata al fine di generare un oggetto [`Dictionary<string, List<string>>`](xref:System.Collections.Generic.Dictionary`2) di errori per l'elaborazione degli errori di convalida dei moduli.</span><span class="sxs-lookup"><span data-stu-id="a2aed-221">Parsing the JSON requires additional code after calling <xref:System.Net.Http.Json.HttpContentJsonExtensions.ReadFromJsonAsync%2A> in order to produce a [`Dictionary<string, List<string>>`](xref:System.Collections.Generic.Dictionary`2) of errors for forms validation error processing.</span></span> <span data-ttu-id="a2aed-222">Idealmente, l'API del server deve restituire solo gli errori di convalida:</span><span class="sxs-lookup"><span data-stu-id="a2aed-222">Ideally, the server API should only return the validation errors:</span></span>

```json
{
  "Identifier": ["The Identifier field is required."],
  "Classification": ["The Classification field is required."],
  "IsValidatedDesign": ["This form disallows unapproved ships."],
  "MaximumAccommodation": ["Accommodation invalid (1-100000)."]
}
```

<span data-ttu-id="a2aed-223">Per modificare la risposta dell'API del server in modo che restituisca solo gli errori di convalida, modificare il delegato che viene richiamato sulle azioni annotate con <xref:Microsoft.AspNetCore.Mvc.ApiControllerAttribute> in `Startup.ConfigureServices` .</span><span class="sxs-lookup"><span data-stu-id="a2aed-223">To modify the server API's response to make it only return the validation errors, change the delegate that's invoked on actions that are annotated with <xref:Microsoft.AspNetCore.Mvc.ApiControllerAttribute> in `Startup.ConfigureServices`.</span></span> <span data-ttu-id="a2aed-224">Per l'endpoint API ( `/StarshipValidation` ), restituire un oggetto <xref:Microsoft.AspNetCore.Mvc.BadRequestObjectResult> con <xref:Microsoft.AspNetCore.Mvc.ModelBinding.ModelStateDictionary> .</span><span class="sxs-lookup"><span data-stu-id="a2aed-224">For the API endpoint (`/StarshipValidation`), return a <xref:Microsoft.AspNetCore.Mvc.BadRequestObjectResult> with the <xref:Microsoft.AspNetCore.Mvc.ModelBinding.ModelStateDictionary>.</span></span> <span data-ttu-id="a2aed-225">Per qualsiasi altro endpoint API, mantenere il comportamento predefinito restituendo il risultato dell'oggetto con un nuovo <xref:Microsoft.AspNetCore.Mvc.ValidationProblemDetails> :</span><span class="sxs-lookup"><span data-stu-id="a2aed-225">For any other API endpoints, preserve the default behavior by returning the object result with a new <xref:Microsoft.AspNetCore.Mvc.ValidationProblemDetails>:</span></span>

```csharp
using Microsoft.AspNetCore.Mvc;

...

services.AddControllersWithViews()
    .ConfigureApiBehaviorOptions(options =>
    {
        options.InvalidModelStateResponseFactory = context =>
        {
            if (context.HttpContext.Request.Path == "/StarshipValidation")
            {
                return new BadRequestObjectResult(context.ModelState);
            }
            else
            {
                return new BadRequestObjectResult(
                    new ValidationProblemDetails(context.ModelState));
            }
        };
    });
```

<span data-ttu-id="a2aed-226">Per altre informazioni, vedere <xref:web-api/handle-errors#validation-failure-error-response>.</span><span class="sxs-lookup"><span data-stu-id="a2aed-226">For more information, see <xref:web-api/handle-errors#validation-failure-error-response>.</span></span>

<span data-ttu-id="a2aed-227">Nel progetto client aggiungere il componente validator visualizzato nella sezione [componenti validator](#validator-components) .</span><span class="sxs-lookup"><span data-stu-id="a2aed-227">In the client project, add the validator component shown in the [Validator components](#validator-components) section.</span></span>

<span data-ttu-id="a2aed-228">Nel progetto client, il modulo *database Starship astronave* viene aggiornato in modo da visualizzare gli errori di convalida del server con la guida del `CustomValidator` componente.</span><span class="sxs-lookup"><span data-stu-id="a2aed-228">In the client project, the *Starfleet Starship Database* form is updated to show server validation errors with help of the `CustomValidator` component.</span></span> <span data-ttu-id="a2aed-229">Quando l'API del server restituisce i messaggi di convalida, vengono aggiunti al `CustomValidator` componente <xref:Microsoft.AspNetCore.Components.Forms.ValidationMessageStore> .</span><span class="sxs-lookup"><span data-stu-id="a2aed-229">When the server API returns validation messages, they're added to the `CustomValidator` component's <xref:Microsoft.AspNetCore.Components.Forms.ValidationMessageStore>.</span></span> <span data-ttu-id="a2aed-230">Gli errori sono disponibili nel formato per la <xref:Microsoft.AspNetCore.Components.Forms.EditContext> visualizzazione da parte del modulo <xref:Microsoft.AspNetCore.Components.Forms.ValidationSummary> :</span><span class="sxs-lookup"><span data-stu-id="a2aed-230">The errors are available in the form's <xref:Microsoft.AspNetCore.Components.Forms.EditContext> for display by the form's <xref:Microsoft.AspNetCore.Components.Forms.ValidationSummary>:</span></span>

```razor
@page "/FormValidation"
@using System.Net
@using System.Net.Http.Json
@using Microsoft.AspNetCore.Authorization
@using Microsoft.AspNetCore.Components.WebAssembly.Authentication
@using Microsoft.Extensions.Logging
@using BlazorSample.Shared
@attribute [Authorize]
@inject HttpClient Http
@inject ILogger<FormValidation> Logger

<h1>Starfleet Starship Database</h1>

<h2>New Ship Entry Form</h2>

<EditForm Model="@starship" OnValidSubmit="@HandleValidSubmit">
    <DataAnnotationsValidator />
    <CustomValidator @ref="customValidator" />
    <ValidationSummary />

    <p>
        <label>
            Identifier:
            <InputText @bind-Value="starship.Identifier" disabled="@disabled" />
        </label>
    </p>
    <p>
        <label>
            Description (optional):
            <InputTextArea @bind-Value="starship.Description" 
                disabled="@disabled" />
        </label>
    </p>
    <p>
        <label>
            Primary Classification:
            <InputSelect @bind-Value="starship.Classification" disabled="@disabled">
                <option value="">Select classification ...</option>
                <option value="Exploration">Exploration</option>
                <option value="Diplomacy">Diplomacy</option>
                <option value="Defense">Defense</option>
            </InputSelect>
        </label>
    </p>
    <p>
        <label>
            Maximum Accommodation:
            <InputNumber @bind-Value="starship.MaximumAccommodation" 
                disabled="@disabled" />
        </label>
    </p>
    <p>
        <label>
            Engineering Approval:
            <InputCheckbox @bind-Value="starship.IsValidatedDesign" 
                disabled="@disabled" />
        </label>
    </p>
    <p>
        <label>
            Production Date:
            <InputDate @bind-Value="starship.ProductionDate" disabled="@disabled" />
        </label>
    </p>

    <button type="submit" disabled="@disabled">Submit</button>

    <p style="@messageStyles">
        @message
    </p>

    <p>
        <a href="http://www.startrek.com/">Star Trek</a>,
        &copy;1966-2019 CBS Studios, Inc. and
        <a href="https://www.paramount.com">Paramount Pictures</a>
    </p>
</EditForm>

@code {
    private bool disabled;
    private string message;
    private string messageStyles = "visibility:hidden";
    private CustomValidator customValidator;
    private Starship starship = new Starship() { ProductionDate = DateTime.UtcNow };

    private async Task HandleValidSubmit(EditContext editContext)
    {
        customValidator.ClearErrors();

        try
        {
            var response = await Http.PostAsJsonAsync<Starship>(
                "StarshipValidation", (Starship)editContext.Model);

            var errors = await response.Content
                .ReadFromJsonAsync<Dictionary<string, List<string>>>();

            if (response.StatusCode == HttpStatusCode.BadRequest && 
                errors.Count() > 0)
            {
                customValidator.DisplayErrors(errors);
            }
            else if (!response.IsSuccessStatusCode)
            {
                throw new HttpRequestException(
                    $"Validation failed. Status Code: {response.StatusCode}");
            }
            else
            {
                disabled = true;
                messageStyles = "color:green";
                message = "The form has been processed.";
            }
        }
        catch (AccessTokenNotAvailableException ex)
        {
            ex.Redirect();
        }
        catch (Exception ex)
        {
            Logger.LogError("Form processing error: {Message}", ex.Message);
            disabled = true;
            messageStyles = "color:red";
            message = "There was an error processing the form.";
        }
    }
}
```

> [!NOTE]
> <span data-ttu-id="a2aed-231">In alternativa ai [componenti di convalida](#validator-components), è possibile usare gli attributi di convalida delle annotazioni dei dati.</span><span class="sxs-lookup"><span data-stu-id="a2aed-231">As an alternative to [validation components](#validator-components), data annotation validation attributes can be used.</span></span> <span data-ttu-id="a2aed-232">Gli attributi personalizzati applicati al modello del modulo vengono attivati con l'utilizzo del <xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> componente.</span><span class="sxs-lookup"><span data-stu-id="a2aed-232">Custom attributes applied to the form's model activate with the use of the <xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> component.</span></span> <span data-ttu-id="a2aed-233">Quando viene utilizzato con la convalida sul lato server, gli attributi devono essere eseguibili nel server.</span><span class="sxs-lookup"><span data-stu-id="a2aed-233">When used with server-side validation, the attributes must be executable on the server.</span></span> <span data-ttu-id="a2aed-234">Per altre informazioni, vedere <xref:mvc/models/validation#alternatives-to-built-in-attributes>.</span><span class="sxs-lookup"><span data-stu-id="a2aed-234">For more information, see <xref:mvc/models/validation#alternatives-to-built-in-attributes>.</span></span>

> [!NOTE]
> <span data-ttu-id="a2aed-235">L'approccio di convalida lato server in questa sezione è adatto a qualsiasi esempio di Blazor WebAssembly soluzione ospitata in questo set di documentazione:</span><span class="sxs-lookup"><span data-stu-id="a2aed-235">The server-side validation approach in this section is suitable for any of the Blazor WebAssembly hosted solution examples in this documentation set:</span></span>
>
> * [<span data-ttu-id="a2aed-236">Azure Active Directory (AAD)</span><span class="sxs-lookup"><span data-stu-id="a2aed-236">Azure Active Directory (AAD)</span></span>](xref:blazor/security/webassembly/hosted-with-azure-active-directory)
> * [<span data-ttu-id="a2aed-237">Azure Active Directory (AAD) B2C</span><span class="sxs-lookup"><span data-stu-id="a2aed-237">Azure Active Directory (AAD) B2C</span></span>](xref:blazor/security/webassembly/hosted-with-azure-active-directory-b2c)
> * [<span data-ttu-id="a2aed-238">Identity Server</span><span class="sxs-lookup"><span data-stu-id="a2aed-238">Identity Server</span></span>](xref:blazor/security/webassembly/hosted-with-identity-server)

## <a name="inputtext-based-on-the-input-event"></a><span data-ttu-id="a2aed-239">InputText basato sull'evento di input</span><span class="sxs-lookup"><span data-stu-id="a2aed-239">InputText based on the input event</span></span>

<span data-ttu-id="a2aed-240">Utilizzare il <xref:Microsoft.AspNetCore.Components.Forms.InputText> componente per creare un componente personalizzato che utilizza l' `input` evento anziché l' `change` evento.</span><span class="sxs-lookup"><span data-stu-id="a2aed-240">Use the <xref:Microsoft.AspNetCore.Components.Forms.InputText> component to create a custom component that uses the `input` event instead of the `change` event.</span></span>

<span data-ttu-id="a2aed-241">Nell'esempio seguente il `CustomInputText` componente eredita il componente del Framework `InputText` e imposta l'associazione di eventi sull' `oninput` evento.</span><span class="sxs-lookup"><span data-stu-id="a2aed-241">In the following example, the `CustomInputText` component inherits the framework's `InputText` component and sets event binding to the `oninput` event.</span></span>

<span data-ttu-id="a2aed-242">`Shared/CustomInputText.razor`:</span><span class="sxs-lookup"><span data-stu-id="a2aed-242">`Shared/CustomInputText.razor`:</span></span>

```razor
@inherits InputText

<input @attributes="AdditionalAttributes" class="@CssClass" 
    @bind="CurrentValueAsString" @bind:event="oninput" />
```

<span data-ttu-id="a2aed-243">Il `CustomInputText` componente può essere usato ovunque <xref:Microsoft.AspNetCore.Components.Forms.InputText> :</span><span class="sxs-lookup"><span data-stu-id="a2aed-243">The `CustomInputText` component can be used anywhere <xref:Microsoft.AspNetCore.Components.Forms.InputText> is used:</span></span>

<span data-ttu-id="a2aed-244">`Pages/TestForm.razor`:</span><span class="sxs-lookup"><span data-stu-id="a2aed-244">`Pages/TestForm.razor`:</span></span>

```razor
@page "/testform"
@using System.ComponentModel.DataAnnotations;

<EditForm Model="@exampleModel" OnValidSubmit="@HandleValidSubmit">
    <DataAnnotationsValidator />
    <ValidationSummary />

    <CustomInputText @bind-Value="exampleModel.Name" />

    <button type="submit">Submit</button>
</EditForm>

<p>
    CurrentValue: @exampleModel.Name
</p>

@code {
    private ExampleModel exampleModel = new ExampleModel();

    private void HandleValidSubmit()
    {
        ...
    }

    public class ExampleModel
    {
        [Required]
        [StringLength(10, ErrorMessage = "Name is too long.")]
        public string Name { get; set; }
    }
}
```

## <a name="radio-buttons"></a><span data-ttu-id="a2aed-245">Pulsanti di opzione</span><span class="sxs-lookup"><span data-stu-id="a2aed-245">Radio buttons</span></span>

::: moniker range=">= aspnetcore-5.0"

<span data-ttu-id="a2aed-246">Usare `InputRadio` i componenti con il `InputRadioGroup` componente per creare un gruppo di pulsanti di opzione.</span><span class="sxs-lookup"><span data-stu-id="a2aed-246">Use `InputRadio` components with the `InputRadioGroup` component to create a radio button group.</span></span> <span data-ttu-id="a2aed-247">Nell'esempio seguente le proprietà vengono aggiunte al `Starship` modello descritto nella sezione componenti dei [moduli incorporati](#built-in-forms-components) :</span><span class="sxs-lookup"><span data-stu-id="a2aed-247">In the following example, properties are added to the `Starship` model described in the [Built-in forms components](#built-in-forms-components) section:</span></span>

```csharp
[Required]
[Range(typeof(Manufacturer), nameof(Manufacturer.SpaceX), 
    nameof(Manufacturer.VirginGalactic), ErrorMessage = "Pick a manufacturer.")]
public Manufacturer Manufacturer { get; set; } = Manufacturer.Unknown;

[Required, EnumDataType(typeof(Color))]
public Color? Color { get; set; } = null;

[Required, EnumDataType(typeof(Engine))]
public Engine? Engine { get; set; } = null;
```

<span data-ttu-id="a2aed-248">Aggiungere quanto segue `enums` all'app.</span><span class="sxs-lookup"><span data-stu-id="a2aed-248">Add the following `enums` to the app.</span></span> <span data-ttu-id="a2aed-249">Creare un nuovo file per conservare `enums` o aggiungere al `enums` `Starship.cs` file.</span><span class="sxs-lookup"><span data-stu-id="a2aed-249">Create a new file to hold the `enums` or add the `enums` to the `Starship.cs` file.</span></span> <span data-ttu-id="a2aed-250">Rendere `enums` accessibile al `Starship` modello e al modulo *database Starship astronave* :</span><span class="sxs-lookup"><span data-stu-id="a2aed-250">Make the `enums` accessible to the `Starship` model and the *Starfleet Starship Database* form:</span></span>

```csharp
public enum Manufacturer { SpaceX, NASA, ULA, VirginGalactic, Unknown }
public enum Color { ImperialRed, SpacecruiserGreen, StarshipBlue, VoyagerOrange }
public enum Engine { Ion, Plasma, Fusion, Warp }
```

<span data-ttu-id="a2aed-251">Aggiornare il modulo del *database Starship della flotta stellare* descritto nella sezione [componenti dei moduli incorporati](#built-in-forms-components) .</span><span class="sxs-lookup"><span data-stu-id="a2aed-251">Update the *Starfleet Starship Database* form described in the [Built-in forms components](#built-in-forms-components) section.</span></span> <span data-ttu-id="a2aed-252">Aggiungere i componenti da produrre:</span><span class="sxs-lookup"><span data-stu-id="a2aed-252">Add the components to produce:</span></span>

* <span data-ttu-id="a2aed-253">Un gruppo di pulsanti di opzione per il produttore della nave.</span><span class="sxs-lookup"><span data-stu-id="a2aed-253">A radio button group for the ship manufacturer.</span></span>
* <span data-ttu-id="a2aed-254">Gruppo di pulsanti di opzione annidato per il colore e il motore della spedizione.</span><span class="sxs-lookup"><span data-stu-id="a2aed-254">A nested radio button group for ship color and engine.</span></span>

```razor
<p>
    <InputRadioGroup @bind-Value="starship.Manufacturer">
        Manufacturer:
        <br>
        @foreach (var manufacturer in (Manufacturer[])Enum
            .GetValues(typeof(Manufacturer)))
        {
            <InputRadio Value="manufacturer" />
            @manufacturer
            <br>
        }
    </InputRadioGroup>
</p>

<p>
    Pick one color and one engine:
    <InputRadioGroup Name="engine" @bind-Value="starship.Engine">
        <InputRadioGroup Name="color" @bind-Value="starship.Color">
            <InputRadio Name="color" Value="Color.ImperialRed" />Imperial Red<br>
            <InputRadio Name="engine" Value="Engine.Ion" />Ion<br>
            <InputRadio Name="color" Value="Color.SpacecruiserGreen" />
                Spacecruiser Green<br>
            <InputRadio Name="engine" Value="Engine.Plasma" />Plasma<br>
            <InputRadio Name="color" Value="Color.StarshipBlue" />Starship Blue<br>
            <InputRadio Name="engine" Value="Engine.Fusion" />Fusion<br>
            <InputRadio Name="color" Value="Color.VoyagerOrange" />
                Voyager Orange<br>
            <InputRadio Name="engine" Value="Engine.Warp" />Warp<br>
        </InputRadioGroup>
    </InputRadioGroup>
</p>
```

> [!NOTE]
> <span data-ttu-id="a2aed-255">Se `Name` viene omesso, `InputRadio` i componenti vengono raggruppati in base al predecessore più recente.</span><span class="sxs-lookup"><span data-stu-id="a2aed-255">If `Name` is omitted, `InputRadio` components are grouped by their most recent ancestor.</span></span>

::: moniker-end

::: moniker range="< aspnetcore-5.0"

<span data-ttu-id="a2aed-256">Quando si utilizzano pulsanti di opzione in un modulo, data binding viene gestito in modo diverso rispetto ad altri elementi perché i pulsanti di opzione vengono valutati come un gruppo.</span><span class="sxs-lookup"><span data-stu-id="a2aed-256">When working with radio buttons in a form, data binding is handled differently than other elements because radio buttons are evaluated as a group.</span></span> <span data-ttu-id="a2aed-257">Il valore di ogni pulsante di opzione è fisso, ma il valore del gruppo di pulsanti di opzione è il valore del pulsante di opzione selezionato.</span><span class="sxs-lookup"><span data-stu-id="a2aed-257">The value of each radio button is fixed, but the value of the radio button group is the value of the selected radio button.</span></span> <span data-ttu-id="a2aed-258">L'esempio seguente illustra come:</span><span class="sxs-lookup"><span data-stu-id="a2aed-258">The following example shows how to:</span></span>

* <span data-ttu-id="a2aed-259">Gestire data binding per un gruppo di pulsanti di opzione.</span><span class="sxs-lookup"><span data-stu-id="a2aed-259">Handle data binding for a radio button group.</span></span>
* <span data-ttu-id="a2aed-260">Supportare la convalida usando un `InputRadio` componente personalizzato.</span><span class="sxs-lookup"><span data-stu-id="a2aed-260">Support validation using a custom `InputRadio` component.</span></span>

```razor
@using System.Globalization
@typeparam TValue
@inherits InputBase<TValue>

<input @attributes="AdditionalAttributes" type="radio" value="@SelectedValue" 
       checked="@(SelectedValue.Equals(Value))" @onchange="OnChange" />

@code {
    [Parameter]
    public TValue SelectedValue { get; set; }

    private void OnChange(ChangeEventArgs args)
    {
        CurrentValueAsString = args.Value.ToString();
    }

    protected override bool TryParseValueFromString(string value, 
        out TValue result, out string errorMessage)
    {
        var success = BindConverter.TryConvertTo<TValue>(
            value, CultureInfo.CurrentCulture, out var parsedValue);
        if (success)
        {
            result = parsedValue;
            errorMessage = null;

            return true;
        }
        else
        {
            result = default;
            errorMessage = $"{FieldIdentifier.FieldName} field isn't valid.";

            return false;
        }
    }
}
```

<span data-ttu-id="a2aed-261">Nell'esempio seguente <xref:Microsoft.AspNetCore.Components.Forms.EditForm> viene utilizzato il `InputRadio` componente precedente per ottenere e convalidare una classificazione dall'utente:</span><span class="sxs-lookup"><span data-stu-id="a2aed-261">The following <xref:Microsoft.AspNetCore.Components.Forms.EditForm> uses the preceding `InputRadio` component to obtain and validate a rating from the user:</span></span>

```razor
@page "/RadioButtonExample"
@using System.ComponentModel.DataAnnotations

<h1>Radio Button Group Test</h1>

<EditForm Model="@model" OnValidSubmit="@HandleValidSubmit">
    <DataAnnotationsValidator />
    <ValidationSummary />

    @for (int i = 1; i <= 5; i++)
    {
        <label>
            <InputRadio name="rate" SelectedValue="i" @bind-Value="model.Rating" />
            @i
        </label>
    }

    <button type="submit">Submit</button>
</EditForm>

<p>You chose: @model.Rating</p>

@code {
    private Model model = new Model();

    private void HandleValidSubmit()
    {
        ...
    }

    public class Model
    {
        [Range(1, 5)]
        public int Rating { get; set; }
    }
}
```

::: moniker-end

## <a name="binding-select-element-options-to-c-object-null-values"></a><span data-ttu-id="a2aed-262">Associazione `<select>` di opzioni di elementi a valori di oggetti C# `null`</span><span class="sxs-lookup"><span data-stu-id="a2aed-262">Binding `<select>` element options to C# object `null` values</span></span>

<span data-ttu-id="a2aed-263">Non esiste un modo sensato per rappresentare il valore dell'opzione di un `<select>` elemento come valore dell'oggetto C# `null` , perché:</span><span class="sxs-lookup"><span data-stu-id="a2aed-263">There's no sensible way to represent a `<select>` element option value as a C# object `null` value, because:</span></span>

* <span data-ttu-id="a2aed-264">Gli attributi HTML non possono avere `null` valori.</span><span class="sxs-lookup"><span data-stu-id="a2aed-264">HTML attributes can't have `null` values.</span></span> <span data-ttu-id="a2aed-265">L'equivalente più vicino a `null` in HTML è l'assenza dell'attributo HTML dell' `value` `<option>` elemento.</span><span class="sxs-lookup"><span data-stu-id="a2aed-265">The closest equivalent to `null` in HTML is absence of the HTML `value` attribute from the `<option>` element.</span></span>
* <span data-ttu-id="a2aed-266">Quando si seleziona un oggetto `<option>` senza `value` attributo, il browser considera il valore come *contenuto di testo* dell' `<option>` elemento.</span><span class="sxs-lookup"><span data-stu-id="a2aed-266">When selecting an `<option>` with no `value` attribute, the browser treats the value as the *text content* of that `<option>`'s element.</span></span>

<span data-ttu-id="a2aed-267">Il Blazor Framework non tenta di evitare il comportamento predefinito perché comporterebbe:</span><span class="sxs-lookup"><span data-stu-id="a2aed-267">The Blazor framework doesn't attempt to suppress the default behavior because it would involve:</span></span>

* <span data-ttu-id="a2aed-268">Creazione di una catena di soluzioni alternative per casi speciali nel Framework.</span><span class="sxs-lookup"><span data-stu-id="a2aed-268">Creating a chain of special-case workarounds in the framework.</span></span>
* <span data-ttu-id="a2aed-269">Modifiche di rilievo al comportamento del Framework corrente.</span><span class="sxs-lookup"><span data-stu-id="a2aed-269">Breaking changes to current framework behavior.</span></span>

::: moniker range=">= aspnetcore-5.0"

<span data-ttu-id="a2aed-270">L'equivalente più plausibile `null` in HTML è una *stringa vuota* `value` .</span><span class="sxs-lookup"><span data-stu-id="a2aed-270">The most plausible `null` equivalent in HTML is an *empty string* `value`.</span></span> <span data-ttu-id="a2aed-271">Il Blazor Framework gestisce le `null` conversioni di stringa vuote per l'associazione bidirezionale a un `<select>` valore di.</span><span class="sxs-lookup"><span data-stu-id="a2aed-271">The Blazor framework handles `null` to empty string conversions for two-way binding to a `<select>`'s value.</span></span>

::: moniker-end

::: moniker range="< aspnetcore-5.0"

<span data-ttu-id="a2aed-272">Il Blazor Framework non gestisce automaticamente le `null` conversioni di stringa vuote quando si tenta l'associazione bidirezionale al `<select>` valore di un oggetto.</span><span class="sxs-lookup"><span data-stu-id="a2aed-272">The Blazor framework doesn't automatically handle `null` to empty string conversions when attempting two-way binding to a `<select>`'s value.</span></span> <span data-ttu-id="a2aed-273">Per ulteriori informazioni, vedere [la pagina relativa alla correzione dell'associazione `<select>` a un valore null (DotNet/aspnetcore #23221)](https://github.com/dotnet/aspnetcore/pull/23221).</span><span class="sxs-lookup"><span data-stu-id="a2aed-273">For more information, see [Fix binding `<select>` to a null value (dotnet/aspnetcore #23221)](https://github.com/dotnet/aspnetcore/pull/23221).</span></span>

::: moniker-end

## <a name="validation-support"></a><span data-ttu-id="a2aed-274">Supporto della convalida</span><span class="sxs-lookup"><span data-stu-id="a2aed-274">Validation support</span></span>

<span data-ttu-id="a2aed-275">Il <xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> componente connette il supporto di convalida usando le annotazioni dei dati a a cascata <xref:Microsoft.AspNetCore.Components.Forms.EditContext> .</span><span class="sxs-lookup"><span data-stu-id="a2aed-275">The <xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> component attaches validation support using data annotations to the cascaded <xref:Microsoft.AspNetCore.Components.Forms.EditContext>.</span></span> <span data-ttu-id="a2aed-276">L'abilitazione del supporto per la convalida usando le annotazioni dei dati richiede questo movimento esplicito.</span><span class="sxs-lookup"><span data-stu-id="a2aed-276">Enabling support for validation using data annotations requires this explicit gesture.</span></span> <span data-ttu-id="a2aed-277">Per usare un sistema di convalida diverso rispetto alle annotazioni dei dati, sostituire <xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> con un'implementazione personalizzata.</span><span class="sxs-lookup"><span data-stu-id="a2aed-277">To use a different validation system than data annotations, replace the <xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> with a custom implementation.</span></span> <span data-ttu-id="a2aed-278">Le implementazioni ASP.NET Core sono disponibili per l'ispezione nell'origine riferimento: [`DataAnnotationsValidator`](https://github.com/dotnet/AspNetCore/blob/main/src/Components/Forms/src/DataAnnotationsValidator.cs) / [`AddDataAnnotationsValidation`](https://github.com/dotnet/AspNetCore/blob/main/src/Components/Forms/src/EditContextDataAnnotationsExtensions.cs) .</span><span class="sxs-lookup"><span data-stu-id="a2aed-278">The ASP.NET Core implementations are available for inspection in the reference source: [`DataAnnotationsValidator`](https://github.com/dotnet/AspNetCore/blob/main/src/Components/Forms/src/DataAnnotationsValidator.cs)/[`AddDataAnnotationsValidation`](https://github.com/dotnet/AspNetCore/blob/main/src/Components/Forms/src/EditContextDataAnnotationsExtensions.cs).</span></span>

[!INCLUDE[](~/blazor/includes/aspnetcore-repo-ref-source-links.md)]

<span data-ttu-id="a2aed-279">Blazor esegue due tipi di convalida:</span><span class="sxs-lookup"><span data-stu-id="a2aed-279">Blazor performs two types of validation:</span></span>

* <span data-ttu-id="a2aed-280">La *convalida dei campi* viene eseguita quando l'utente esce da un campo.</span><span class="sxs-lookup"><span data-stu-id="a2aed-280">*Field validation* is performed when the user tabs out of a field.</span></span> <span data-ttu-id="a2aed-281">Durante la convalida dei campi, il <xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> componente associa tutti i risultati della convalida segnalati al campo.</span><span class="sxs-lookup"><span data-stu-id="a2aed-281">During field validation, the <xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> component associates all reported validation results with the field.</span></span>
* <span data-ttu-id="a2aed-282">La *convalida del modello* viene eseguita quando l'utente invia il modulo.</span><span class="sxs-lookup"><span data-stu-id="a2aed-282">*Model validation* is performed when the user submits the form.</span></span> <span data-ttu-id="a2aed-283">Durante la convalida del modello, il <xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> componente tenta di determinare il campo in base al nome del membro segnalato dal risultato della convalida.</span><span class="sxs-lookup"><span data-stu-id="a2aed-283">During model validation, the <xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> component attempts to determine the field based on the member name that the validation result reports.</span></span> <span data-ttu-id="a2aed-284">I risultati della convalida che non sono associati a un singolo membro sono associati al modello anziché a un campo.</span><span class="sxs-lookup"><span data-stu-id="a2aed-284">Validation results that aren't associated with an individual member are associated with the model rather than a field.</span></span>

### <a name="validation-summary-and-validation-message-components"></a><span data-ttu-id="a2aed-285">Componenti del messaggio di convalida e di riepilogo della convalida</span><span class="sxs-lookup"><span data-stu-id="a2aed-285">Validation Summary and Validation Message components</span></span>

<span data-ttu-id="a2aed-286">Il <xref:Microsoft.AspNetCore.Components.Forms.ValidationSummary> componente riepiloga tutti i messaggi di convalida, che è simile all' [Helper tag del riepilogo di convalida](xref:mvc/views/working-with-forms#the-validation-summary-tag-helper):</span><span class="sxs-lookup"><span data-stu-id="a2aed-286">The <xref:Microsoft.AspNetCore.Components.Forms.ValidationSummary> component summarizes all validation messages, which is similar to the [Validation Summary Tag Helper](xref:mvc/views/working-with-forms#the-validation-summary-tag-helper):</span></span>

```razor
<ValidationSummary />
```

<span data-ttu-id="a2aed-287">Messaggi di convalida dell'output per un modello specifico con il `Model` parametro:</span><span class="sxs-lookup"><span data-stu-id="a2aed-287">Output validation messages for a specific model with the `Model` parameter:</span></span>
  
```razor
<ValidationSummary Model="@starship" />
```

<span data-ttu-id="a2aed-288">Il <xref:Microsoft.AspNetCore.Components.Forms.ValidationMessage%601> componente Visualizza i messaggi di convalida per un campo specifico, che è simile all' [Helper tag del messaggio di convalida](xref:mvc/views/working-with-forms#the-validation-message-tag-helper).</span><span class="sxs-lookup"><span data-stu-id="a2aed-288">The <xref:Microsoft.AspNetCore.Components.Forms.ValidationMessage%601> component displays validation messages for a specific field, which is similar to the [Validation Message Tag Helper](xref:mvc/views/working-with-forms#the-validation-message-tag-helper).</span></span> <span data-ttu-id="a2aed-289">Specificare il campo per la convalida con l' `For` attributo e un'espressione lambda che denomina la proprietà del modello:</span><span class="sxs-lookup"><span data-stu-id="a2aed-289">Specify the field for validation with the `For` attribute and a lambda expression naming the model property:</span></span>

```razor
<ValidationMessage For="@(() => starship.MaximumAccommodation)" />
```

<span data-ttu-id="a2aed-290">I <xref:Microsoft.AspNetCore.Components.Forms.ValidationMessage%601> <xref:Microsoft.AspNetCore.Components.Forms.ValidationSummary> componenti e supportano attributi arbitrari.</span><span class="sxs-lookup"><span data-stu-id="a2aed-290">The <xref:Microsoft.AspNetCore.Components.Forms.ValidationMessage%601> and <xref:Microsoft.AspNetCore.Components.Forms.ValidationSummary> components support arbitrary attributes.</span></span> <span data-ttu-id="a2aed-291">Qualsiasi attributo che non corrisponde a un parametro component viene aggiunto all' `<div>` elemento generato o `<ul>` .</span><span class="sxs-lookup"><span data-stu-id="a2aed-291">Any attribute that doesn't match a component parameter is added to the generated `<div>` or `<ul>` element.</span></span>

<span data-ttu-id="a2aed-292">Controllare lo stile dei messaggi di convalida nel foglio di stile dell'app ( `wwwroot/css/app.css` o `wwwroot/css/site.css` ).</span><span class="sxs-lookup"><span data-stu-id="a2aed-292">Control the style of validation messages in the app's stylesheet (`wwwroot/css/app.css` or `wwwroot/css/site.css`).</span></span> <span data-ttu-id="a2aed-293">La `validation-message` classe predefinita imposta il colore del testo dei messaggi di convalida su rosso:</span><span class="sxs-lookup"><span data-stu-id="a2aed-293">The default `validation-message` class sets the text color of validation messages to red:</span></span>

```css
.validation-message {
    color: red;
}
```

### <a name="custom-validation-attributes"></a><span data-ttu-id="a2aed-294">Attributi di convalida personalizzati</span><span class="sxs-lookup"><span data-stu-id="a2aed-294">Custom validation attributes</span></span>

<span data-ttu-id="a2aed-295">Per assicurarsi che un risultato di convalida venga associato correttamente a un campo quando si usa un [attributo di convalida personalizzato](xref:mvc/models/validation#custom-attributes), passare il contesto di convalida <xref:System.ComponentModel.DataAnnotations.ValidationContext.MemberName> durante la creazione di <xref:System.ComponentModel.DataAnnotations.ValidationResult> :</span><span class="sxs-lookup"><span data-stu-id="a2aed-295">To ensure that a validation result is correctly associated with a field when using a [custom validation attribute](xref:mvc/models/validation#custom-attributes), pass the validation context's <xref:System.ComponentModel.DataAnnotations.ValidationContext.MemberName> when creating the <xref:System.ComponentModel.DataAnnotations.ValidationResult>:</span></span>

```csharp
using System;
using System.ComponentModel.DataAnnotations;

public class CustomValidator : ValidationAttribute
{
    protected override ValidationResult IsValid(object value, 
        ValidationContext validationContext)
    {
        ...

        return new ValidationResult("Validation message to user.",
            new[] { validationContext.MemberName });
    }
}
```

> [!NOTE]
> <span data-ttu-id="a2aed-296"><xref:System.ComponentModel.DataAnnotations.ValidationContext.GetService%2A?displayProperty=nameWithType> è `null`.</span><span class="sxs-lookup"><span data-stu-id="a2aed-296"><xref:System.ComponentModel.DataAnnotations.ValidationContext.GetService%2A?displayProperty=nameWithType> is `null`.</span></span> <span data-ttu-id="a2aed-297">L'inserimento di servizi per la convalida nel `IsValid` metodo non è supportato.</span><span class="sxs-lookup"><span data-stu-id="a2aed-297">Injecting services for validation in the `IsValid` method isn't supported.</span></span>

::: moniker range=">= aspnetcore-5.0"

## <a name="custom-validation-class-attributes"></a><span data-ttu-id="a2aed-298">Attributi della classe di convalida personalizzata</span><span class="sxs-lookup"><span data-stu-id="a2aed-298">Custom validation class attributes</span></span>

<span data-ttu-id="a2aed-299">I nomi delle classi di convalida personalizzate sono utili per l'integrazione con Framework CSS, ad esempio [bootstrap](https://getbootstrap.com/).</span><span class="sxs-lookup"><span data-stu-id="a2aed-299">Custom validation class names are useful when integrating with CSS frameworks, such as [Bootstrap](https://getbootstrap.com/).</span></span>

<span data-ttu-id="a2aed-300">Per specificare i nomi delle classi di convalida personalizzate:</span><span class="sxs-lookup"><span data-stu-id="a2aed-300">To specify custom validation class names:</span></span>

* <span data-ttu-id="a2aed-301">Fornire stili CSS per la convalida personalizzata.</span><span class="sxs-lookup"><span data-stu-id="a2aed-301">Provide CSS styles for custom validation.</span></span> <span data-ttu-id="a2aed-302">Nell'esempio seguente vengono specificati gli stili validi e non validi:</span><span class="sxs-lookup"><span data-stu-id="a2aed-302">In the following example, valid and invalid styles are specified:</span></span>

```css
.validField {
    border-color: lawngreen;
}

.invalidField {
    background-color: tomato;
}
```

* <span data-ttu-id="a2aed-303">Creare una classe derivata da `FieldCssClassProvider` che controlla i messaggi di convalida del campo e applica lo stile valido o non valido appropriato:</span><span class="sxs-lookup"><span data-stu-id="a2aed-303">Create a class derived from `FieldCssClassProvider` that checks for field validation messages and applies the appropriate valid or invalid style:</span></span>

```csharp
using System.Linq;
using Microsoft.AspNetCore.Components.Forms;

public class MyFieldClassProvider : FieldCssClassProvider
{
    public override string GetFieldCssClass(EditContext editContext, 
        in FieldIdentifier fieldIdentifier)
    {
        var isValid = !editContext.GetValidationMessages(fieldIdentifier).Any();

        return isValid ? "validField" : "invalidField";
    }
}
```

* <span data-ttu-id="a2aed-304">Impostare la classe nell'istanza del form <xref:Microsoft.AspNetCore.Components.Forms.EditContext> :</span><span class="sxs-lookup"><span data-stu-id="a2aed-304">Set the class on the form's <xref:Microsoft.AspNetCore.Components.Forms.EditContext> instance:</span></span>

```razor
...

<EditForm EditContext="@editContext" OnValidSubmit="@HandleValidSubmit">
    ...
</EditForm>

...

@code {
    private EditContext editContext;
    private Model model = new Model();

    protected override void OnInitialized()
    {
        editContext = new EditContext(model);
        editContext.SetFieldCssClassProvider(new MyFieldClassProvider());
    }

    private void HandleValidSubmit()
    {
        ...
    }
}
```

<span data-ttu-id="a2aed-305">Nell'esempio precedente viene verificata la validità di tutti i campi del modulo e viene applicato uno stile a ogni campo.</span><span class="sxs-lookup"><span data-stu-id="a2aed-305">The preceding example checks the validity of all form fields and applies a style to each field.</span></span> <span data-ttu-id="a2aed-306">Se il modulo deve applicare stili personalizzati solo a un subset dei campi, applicare gli `MyFieldClassProvider` stili in modo condizionale.</span><span class="sxs-lookup"><span data-stu-id="a2aed-306">If the form should only apply custom styles to a subset of the fields, make `MyFieldClassProvider` apply styles conditionally.</span></span> <span data-ttu-id="a2aed-307">Nell'esempio seguente viene applicato solo uno stile al `Identifier` campo:</span><span class="sxs-lookup"><span data-stu-id="a2aed-307">The following example only applies a style to the `Identifier` field:</span></span>

```csharp
public class MyFieldClassProvider : FieldCssClassProvider
{
    public override string GetFieldCssClass(EditContext editContext,
        in FieldIdentifier fieldIdentifier)
    {
        if (fieldIdentifier.FieldName == "Identifier")
        {
            var isValid = !editContext.GetValidationMessages(fieldIdentifier).Any();

            return isValid ? "validField" : "invalidField";
        }

        return string.Empty;
    }
}
```

::: moniker-end

### <a name="blazor-data-annotations-validation-package"></a><span data-ttu-id="a2aed-308">Blazor pacchetto di convalida delle annotazioni dei dati</span><span class="sxs-lookup"><span data-stu-id="a2aed-308">Blazor data annotations validation package</span></span>

<span data-ttu-id="a2aed-309">[`Microsoft.AspNetCore.Components.DataAnnotations.Validation`](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.DataAnnotations.Validation)È un pacchetto che colma i gap dell'esperienza di convalida usando il <xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> componente.</span><span class="sxs-lookup"><span data-stu-id="a2aed-309">The [`Microsoft.AspNetCore.Components.DataAnnotations.Validation`](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.DataAnnotations.Validation) is a package that fills validation experience gaps using the <xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> component.</span></span> <span data-ttu-id="a2aed-310">Il pacchetto è attualmente *sperimentale*.</span><span class="sxs-lookup"><span data-stu-id="a2aed-310">The package is currently *experimental*.</span></span>

> [!NOTE]
> <span data-ttu-id="a2aed-311">Il [`Microsoft.AspNetCore.Components.DataAnnotations.Validation`](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.DataAnnotations.Validation) pacchetto dispone di una versione più recente di *release candidate* in [NuGet.org](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.DataAnnotations.Validation). Continuare a usare il pacchetto *sperimentale* versione finale al momento.</span><span class="sxs-lookup"><span data-stu-id="a2aed-311">The [`Microsoft.AspNetCore.Components.DataAnnotations.Validation`](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.DataAnnotations.Validation) package has a latest version of *release candidate* at [Nuget.org](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.DataAnnotations.Validation). Continue to use the *experimental* release candidate package at this time.</span></span> <span data-ttu-id="a2aed-312">L'assembly del pacchetto potrebbe essere spostato nel Framework o nel runtime in una versione futura.</span><span class="sxs-lookup"><span data-stu-id="a2aed-312">The package's assembly might be moved to either the framework or the runtime in a future release.</span></span> <span data-ttu-id="a2aed-313">Per ulteriori aggiornamenti, vedere il [repository GitHub degli annunci](https://github.com/aspnet/Announcements), il [repository GitHub DotNet/aspnetcore](https://github.com/dotnet/aspnetcore)o questa sezione dell'argomento.</span><span class="sxs-lookup"><span data-stu-id="a2aed-313">Watch the [Announcements GitHub repository](https://github.com/aspnet/Announcements), the [dotnet/aspnetcore GitHub repository](https://github.com/dotnet/aspnetcore), or this topic section for further updates.</span></span>

::: moniker range="< aspnetcore-5.0"

### <a name="compareproperty-attribute"></a><span data-ttu-id="a2aed-314">Attributo `[CompareProperty]`</span><span class="sxs-lookup"><span data-stu-id="a2aed-314">`[CompareProperty]` attribute</span></span>

<span data-ttu-id="a2aed-315"><xref:System.ComponentModel.DataAnnotations.CompareAttribute>Non funziona bene con il <xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> componente perché non associa il risultato della convalida a un membro specifico.</span><span class="sxs-lookup"><span data-stu-id="a2aed-315">The <xref:System.ComponentModel.DataAnnotations.CompareAttribute> doesn't work well with the <xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> component because it doesn't associate the validation result with a specific member.</span></span> <span data-ttu-id="a2aed-316">Ciò può comportare un comportamento incoerente tra la convalida a livello di campo e quando l'intero modello viene convalidato in un invio.</span><span class="sxs-lookup"><span data-stu-id="a2aed-316">This can result in inconsistent behavior between field-level validation and when the entire model is validated on a submit.</span></span> <span data-ttu-id="a2aed-317">Il [`Microsoft.AspNetCore.Components.DataAnnotations.Validation`](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.DataAnnotations.Validation) pacchetto *sperimentale* introduce un attributo di convalida aggiuntivo, `ComparePropertyAttribute` , che aggira queste limitazioni.</span><span class="sxs-lookup"><span data-stu-id="a2aed-317">The [`Microsoft.AspNetCore.Components.DataAnnotations.Validation`](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.DataAnnotations.Validation) *experimental* package introduces an additional validation attribute, `ComparePropertyAttribute`, that works around these limitations.</span></span> <span data-ttu-id="a2aed-318">In un' Blazor applicazione, `[CompareProperty]` è una sostituzione diretta per l' [ `[Compare]` attributo](xref:System.ComponentModel.DataAnnotations.CompareAttribute).</span><span class="sxs-lookup"><span data-stu-id="a2aed-318">In a Blazor app, `[CompareProperty]` is a direct replacement for the [`[Compare]` attribute](xref:System.ComponentModel.DataAnnotations.CompareAttribute).</span></span>

::: moniker-end

### <a name="nested-models-collection-types-and-complex-types"></a><span data-ttu-id="a2aed-319">Modelli annidati, tipi di raccolta e tipi complessi</span><span class="sxs-lookup"><span data-stu-id="a2aed-319">Nested models, collection types, and complex types</span></span>

<span data-ttu-id="a2aed-320">Blazor fornisce supporto per la convalida dell'input del form utilizzando le annotazioni dei dati con l'oggetto incorporato <xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> .</span><span class="sxs-lookup"><span data-stu-id="a2aed-320">Blazor provides support for validating form input using data annotations with the built-in <xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator>.</span></span> <span data-ttu-id="a2aed-321">Tuttavia, l'oggetto <xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> convalida solo le proprietà di primo livello del modello associate al form che non sono proprietà di tipo raccolta o complesso.</span><span class="sxs-lookup"><span data-stu-id="a2aed-321">However, the <xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> only validates top-level properties of the model bound to the form that aren't collection- or complex-type properties.</span></span>

<span data-ttu-id="a2aed-322">Per convalidare l'intero grafico di oggetti del modello associato, incluse le proprietà della raccolta e del tipo complesso, usare l'oggetto `ObjectGraphDataAnnotationsValidator` fornito dal pacchetto *sperimentale* [`Microsoft.AspNetCore.Components.DataAnnotations.Validation`](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.DataAnnotations.Validation) :</span><span class="sxs-lookup"><span data-stu-id="a2aed-322">To validate the bound model's entire object graph, including collection- and complex-type properties, use the `ObjectGraphDataAnnotationsValidator` provided by the *experimental* [`Microsoft.AspNetCore.Components.DataAnnotations.Validation`](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.DataAnnotations.Validation) package:</span></span>

```razor
<EditForm Model="@model" OnValidSubmit="@HandleValidSubmit">
    <ObjectGraphDataAnnotationsValidator />
    ...
</EditForm>
```

<span data-ttu-id="a2aed-323">Annotare le proprietà del modello con `[ValidateComplexType]` .</span><span class="sxs-lookup"><span data-stu-id="a2aed-323">Annotate model properties with `[ValidateComplexType]`.</span></span> <span data-ttu-id="a2aed-324">Nelle classi di modelli seguenti la `ShipDescription` classe contiene annotazioni di dati aggiuntive da convalidare quando il modello è associato al form:</span><span class="sxs-lookup"><span data-stu-id="a2aed-324">In the following model classes, the `ShipDescription` class contains additional data annotations to validate when the model is bound to the form:</span></span>

<span data-ttu-id="a2aed-325">`Starship.cs`:</span><span class="sxs-lookup"><span data-stu-id="a2aed-325">`Starship.cs`:</span></span>

```csharp
using System;
using System.ComponentModel.DataAnnotations;

public class Starship
{
    ...

    [ValidateComplexType]
    public ShipDescription ShipDescription { get; set; } = 
        new ShipDescription();

    ...
}
```

<span data-ttu-id="a2aed-326">`ShipDescription.cs`:</span><span class="sxs-lookup"><span data-stu-id="a2aed-326">`ShipDescription.cs`:</span></span>

```csharp
using System;
using System.ComponentModel.DataAnnotations;

public class ShipDescription
{
    [Required]
    [StringLength(40, ErrorMessage = "Description too long (40 char).")]
    public string ShortDescription { get; set; }

    [Required]
    [StringLength(240, ErrorMessage = "Description too long (240 char).")]
    public string LongDescription { get; set; }
}
```

### <a name="enable-the-submit-button-based-on-form-validation"></a><span data-ttu-id="a2aed-327">Abilita il pulsante Invia in base alla convalida del modulo</span><span class="sxs-lookup"><span data-stu-id="a2aed-327">Enable the submit button based on form validation</span></span>

<span data-ttu-id="a2aed-328">Per abilitare e disabilitare il pulsante Invia in base alla convalida del modulo:</span><span class="sxs-lookup"><span data-stu-id="a2aed-328">To enable and disable the submit button based on form validation:</span></span>

* <span data-ttu-id="a2aed-329">Utilizzare il modulo <xref:Microsoft.AspNetCore.Components.Forms.EditContext> per assegnare il modello quando il componente viene inizializzato.</span><span class="sxs-lookup"><span data-stu-id="a2aed-329">Use the form's <xref:Microsoft.AspNetCore.Components.Forms.EditContext> to assign the model when the component is initialized.</span></span>
* <span data-ttu-id="a2aed-330">Convalidare il form nel callback del contesto <xref:Microsoft.AspNetCore.Components.Forms.EditContext.OnFieldChanged> per abilitare e disabilitare il pulsante Submit.</span><span class="sxs-lookup"><span data-stu-id="a2aed-330">Validate the form in the context's <xref:Microsoft.AspNetCore.Components.Forms.EditContext.OnFieldChanged> callback to enable and disable the submit button.</span></span>
* <span data-ttu-id="a2aed-331">Implementare <xref:System.IDisposable> e annullare la sottoscrizione del gestore eventi nel `Dispose` metodo.</span><span class="sxs-lookup"><span data-stu-id="a2aed-331">Implement <xref:System.IDisposable> and unsubscribe the event handler in the `Dispose` method.</span></span> <span data-ttu-id="a2aed-332">Per altre informazioni, vedere <xref:blazor/components/lifecycle#component-disposal-with-idisposable>.</span><span class="sxs-lookup"><span data-stu-id="a2aed-332">For more information, see <xref:blazor/components/lifecycle#component-disposal-with-idisposable>.</span></span>

> [!NOTE]
> <span data-ttu-id="a2aed-333">Quando si usa un oggetto <xref:Microsoft.AspNetCore.Components.Forms.EditContext> , non assegnare anche un oggetto <xref:Microsoft.AspNetCore.Components.Forms.EditForm.Model> a <xref:Microsoft.AspNetCore.Components.Forms.EditForm> .</span><span class="sxs-lookup"><span data-stu-id="a2aed-333">When using an <xref:Microsoft.AspNetCore.Components.Forms.EditContext>, don't also assign a <xref:Microsoft.AspNetCore.Components.Forms.EditForm.Model> to the <xref:Microsoft.AspNetCore.Components.Forms.EditForm>.</span></span>

```razor
@implements IDisposable

<EditForm EditContext="@editContext">
    <DataAnnotationsValidator />
    <ValidationSummary />

    ...

    <button type="submit" disabled="@formInvalid">Submit</button>
</EditForm>

@code {
    private Starship starship = new Starship() { ProductionDate = DateTime.UtcNow };
    private bool formInvalid = true;
    private EditContext editContext;

    protected override void OnInitialized()
    {
        editContext = new EditContext(starship);
        editContext.OnFieldChanged += HandleFieldChanged;
    }

    private void HandleFieldChanged(object sender, FieldChangedEventArgs e)
    {
        formInvalid = !editContext.Validate();
        StateHasChanged();
    }

    public void Dispose()
    {
        editContext.OnFieldChanged -= HandleFieldChanged;
    }
}
```

<span data-ttu-id="a2aed-334">Nell'esempio precedente, impostare `formInvalid` su `false` if:</span><span class="sxs-lookup"><span data-stu-id="a2aed-334">In the preceding example, set `formInvalid` to `false` if:</span></span>

* <span data-ttu-id="a2aed-335">Il modulo viene precaricato con valori predefiniti validi.</span><span class="sxs-lookup"><span data-stu-id="a2aed-335">The form is preloaded with valid default values.</span></span>
* <span data-ttu-id="a2aed-336">Si desidera attivare il pulsante Invia quando il modulo viene caricato.</span><span class="sxs-lookup"><span data-stu-id="a2aed-336">You want the submit button enabled when the form loads.</span></span>

<span data-ttu-id="a2aed-337">Un effetto collaterale dell'approccio precedente è che un <xref:Microsoft.AspNetCore.Components.Forms.ValidationSummary> componente viene popolato con campi non validi dopo che l'utente interagisce con un qualsiasi campo.</span><span class="sxs-lookup"><span data-stu-id="a2aed-337">A side effect of the preceding approach is that a <xref:Microsoft.AspNetCore.Components.Forms.ValidationSummary> component is populated with invalid fields after the user interacts with any one field.</span></span> <span data-ttu-id="a2aed-338">Questo scenario può essere risolto in uno dei modi seguenti:</span><span class="sxs-lookup"><span data-stu-id="a2aed-338">This scenario can be addressed in either of the following ways:</span></span>

* <span data-ttu-id="a2aed-339">Non usare un <xref:Microsoft.AspNetCore.Components.Forms.ValidationSummary> componente nel form.</span><span class="sxs-lookup"><span data-stu-id="a2aed-339">Don't use a <xref:Microsoft.AspNetCore.Components.Forms.ValidationSummary> component on the form.</span></span>
* <span data-ttu-id="a2aed-340">Rendere <xref:Microsoft.AspNetCore.Components.Forms.ValidationSummary> visibile il componente quando viene selezionato il pulsante Invia, ad esempio in un `HandleValidSubmit` metodo.</span><span class="sxs-lookup"><span data-stu-id="a2aed-340">Make the <xref:Microsoft.AspNetCore.Components.Forms.ValidationSummary> component visible when the submit button is selected (for example, in a `HandleValidSubmit` method).</span></span>

```razor
<EditForm EditContext="@editContext" OnValidSubmit="@HandleValidSubmit">
    <DataAnnotationsValidator />
    <ValidationSummary style="@displaySummary" />

    ...

    <button type="submit" disabled="@formInvalid">Submit</button>
</EditForm>

@code {
    private string displaySummary = "display:none";

    ...

    private void HandleValidSubmit()
    {
        displaySummary = "display:block";
    }
}
```

## <a name="troubleshoot"></a><span data-ttu-id="a2aed-341">Risolvere problemi</span><span class="sxs-lookup"><span data-stu-id="a2aed-341">Troubleshoot</span></span>

> <span data-ttu-id="a2aed-342">InvalidOperationException: EditForm richiede un parametro di modello o un parametro EditContext, ma non entrambi.</span><span class="sxs-lookup"><span data-stu-id="a2aed-342">InvalidOperationException: EditForm requires a Model parameter, or an EditContext parameter, but not both.</span></span>

<span data-ttu-id="a2aed-343">Verificare che <xref:Microsoft.AspNetCore.Components.Forms.EditForm> disponga di un oggetto <xref:Microsoft.AspNetCore.Components.Forms.EditForm.Model> **o** <xref:Microsoft.AspNetCore.Components.Forms.EditContext> .</span><span class="sxs-lookup"><span data-stu-id="a2aed-343">Confirm that the <xref:Microsoft.AspNetCore.Components.Forms.EditForm> has a <xref:Microsoft.AspNetCore.Components.Forms.EditForm.Model> **or** <xref:Microsoft.AspNetCore.Components.Forms.EditContext>.</span></span> <span data-ttu-id="a2aed-344">Non usare entrambi per lo stesso formato.</span><span class="sxs-lookup"><span data-stu-id="a2aed-344">Don't use both for the same form.</span></span>

<span data-ttu-id="a2aed-345">Quando si assegna un <xref:Microsoft.AspNetCore.Components.Forms.EditForm.Model> al form, verificare che venga creata un'istanza del tipo di modello, come illustrato nell'esempio seguente:</span><span class="sxs-lookup"><span data-stu-id="a2aed-345">When assigning a <xref:Microsoft.AspNetCore.Components.Forms.EditForm.Model> to the form, confirm that the model type is instantiated, as the following example shows:</span></span>

```csharp
private ExampleModel exampleModel = new ExampleModel();
```

::: moniker range=">= aspnetcore-5.0"

## <a name="additional-resources"></a><span data-ttu-id="a2aed-346">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="a2aed-346">Additional resources</span></span>

* <xref:blazor/file-uploads>

::: moniker-end
