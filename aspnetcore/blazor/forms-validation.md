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
ms.openlocfilehash: 012c8794b3d239ce93ac942000c7ec4f71d06cbf
ms.sourcegitcommit: 1166b0ff3828418559510c661e8240e5c5717bb7
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/12/2021
ms.locfileid: "100280001"
---
# <a name="aspnet-core-blazor-forms-and-validation"></a>ASP.NET Core Blazor moduli e convalida

I moduli e la convalida sono supportati nell' Blazor utilizzo delle [annotazioni dei dati](xref:mvc/models/validation).

Il `ExampleModel` tipo seguente definisce la logica di convalida usando le annotazioni dei dati:

```csharp
using System.ComponentModel.DataAnnotations;

public class ExampleModel
{
    [Required]
    [StringLength(10, ErrorMessage = "Name is too long.")]
    public string Name { get; set; }
}
```

Un modulo viene definito usando il <xref:Microsoft.AspNetCore.Components.Forms.EditForm> componente. Il modulo seguente illustra elementi, componenti e codice tipici Razor :

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

Nell'esempio precedente:

* Il modulo convalida l'input dell'utente nel `name` campo usando la convalida definita nel `ExampleModel` tipo. Il modello viene creato nel blocco del componente `@code` e viene mantenuto in un campo privato ( `exampleModel` ). Il campo viene assegnato all' `Model` attributo dell' `<EditForm>` elemento.
* Le <xref:Microsoft.AspNetCore.Components.Forms.InputText> associazioni del componente `@bind-Value` :
  * Proprietà del modello ( `exampleModel.Name` ) della <xref:Microsoft.AspNetCore.Components.Forms.InputText> proprietà del componente `Value` . Per ulteriori informazioni sull'associazione di proprietà, vedere <xref:blazor/components/data-binding#binding-with-component-parameters> .
  * Delegato dell'evento di modifica alla <xref:Microsoft.AspNetCore.Components.Forms.InputText> proprietà del componente `ValueChanged` .
* Il <xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> [componente validator](#validator-components) connette il supporto della convalida usando le annotazioni dei dati.
* Il <xref:Microsoft.AspNetCore.Components.Forms.ValidationSummary> componente riepiloga i messaggi di convalida.
* `HandleValidSubmit` viene attivato quando il form viene inviato correttamente (supera la convalida).

## <a name="built-in-forms-components"></a>Componenti incorporati dei moduli

È disponibile un set di componenti predefiniti per la ricezione e la convalida dell'input dell'utente. Gli input vengono convalidati quando vengono modificati e quando viene inviato un modulo. I componenti di input disponibili sono illustrati nella tabella seguente.

::: moniker range=">= aspnetcore-5.0"

| Componente di input | Con rendering come&hellip; |
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

| Componente di input | Con rendering come&hellip; |
| --------------- | ------------------- |
| <xref:Microsoft.AspNetCore.Components.Forms.InputCheckbox> | `<input type="checkbox">` |
| <xref:Microsoft.AspNetCore.Components.Forms.InputDate%601> | `<input type="date">` |
| <xref:Microsoft.AspNetCore.Components.Forms.InputNumber%601> | `<input type="number">` |
| <xref:Microsoft.AspNetCore.Components.Forms.InputSelect%601> | `<select>` |
| <xref:Microsoft.AspNetCore.Components.Forms.InputText> | `<input>` |
| <xref:Microsoft.AspNetCore.Components.Forms.InputTextArea> | `<textarea>` |

> [!NOTE]
> I `InputRadio` `InputRadioGroup` componenti e sono disponibili in ASP.NET Core 5,0 o versione successiva. Per ulteriori informazioni, selezionare una versione 5,0 o successiva di questo articolo.

::: moniker-end

Tutti i componenti di input, tra cui <xref:Microsoft.AspNetCore.Components.Forms.EditForm> , supportano attributi arbitrari. Qualsiasi attributo che non corrisponde a un parametro component viene aggiunto all'elemento HTML sottoposto a rendering.

I componenti di input forniscono il comportamento predefinito per la convalida quando viene modificato un campo, incluso l'aggiornamento della classe CSS del campo per riflettere lo stato del campo. Alcuni componenti includono una logica di analisi utile. Ad esempio, <xref:Microsoft.AspNetCore.Components.Forms.InputDate%601> e <xref:Microsoft.AspNetCore.Components.Forms.InputNumber%601> gestiscono correttamente i valori non analizzabili registrando valori non analizzabili come errori di convalida. I tipi che possono accettare valori null supportano anche il supporto di valori null del campo di destinazione, ad esempio `int?` .

Il `Starship` tipo seguente definisce la logica di convalida usando un set di proprietà e annotazioni di dati più ampio rispetto a quello precedente `ExampleModel` :

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

Nell'esempio precedente `Description` è facoltativo perché non sono presenti annotazioni di dati.

Il form seguente convalida l'input dell'utente utilizzando la convalida definita nel `Starship` modello:

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

<xref:Microsoft.AspNetCore.Components.Forms.EditForm>Crea un oggetto <xref:Microsoft.AspNetCore.Components.Forms.EditContext> come [valore](xref:blazor/components/cascading-values-and-parameters) di propagazione che tiene traccia dei metadati relativi al processo di modifica, inclusi i campi modificati e i messaggi di convalida correnti.

Assegnare **un** oggetto <xref:Microsoft.AspNetCore.Components.Forms.EditContext> **o** <xref:Microsoft.AspNetCore.Components.Forms.EditForm.Model?displayProperty=nameWithType> a un oggetto <xref:Microsoft.AspNetCore.Components.Forms.EditForm> . L'assegnazione di entrambi non è supportata e genera un **errore di runtime**.

<xref:Microsoft.AspNetCore.Components.Forms.EditForm>Fornisce gli eventi pratici per l'invio di form valido e non valido:

* <xref:Microsoft.AspNetCore.Components.Forms.EditForm.OnValidSubmit>
* <xref:Microsoft.AspNetCore.Components.Forms.EditForm.OnInvalidSubmit>

Usare <xref:Microsoft.AspNetCore.Components.Forms.EditForm.OnSubmit> per usare codice personalizzato per attivare la convalida e verificare i valori dei campi.

Nell'esempio seguente:

* Il `HandleSubmit` metodo viene eseguito quando **`Submit`** si seleziona il pulsante.
* Il modulo viene convalidato chiamando <xref:Microsoft.AspNetCore.Components.Forms.EditContext.Validate%2A?displayProperty=nameWithType> .
* Il codice aggiuntivo viene eseguito a seconda del risultato della convalida. Inserire la logica di business nel metodo assegnato a <xref:Microsoft.AspNetCore.Components.Forms.EditForm.OnSubmit> .

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
> L'API del Framework non esiste per cancellare i messaggi di convalida direttamente da un <xref:Microsoft.AspNetCore.Components.Forms.EditContext> . Pertanto, in genere non è consigliabile aggiungere messaggi di convalida a un nuovo <xref:Microsoft.AspNetCore.Components.Forms.ValidationMessageStore> in un modulo. Per gestire i messaggi di convalida, usare un [componente validator](#validator-components) con il [codice di convalida della logica di business](#business-logic-validation), come descritto in questo articolo.

::: moniker range=">= aspnetcore-5.0"

## <a name="display-name-support"></a>Supporto del nome visualizzato

*Questa sezione si applica a ASP.NET Core in .NET 5 Release Candidate 1 (RC1) o versioni successive.*

I seguenti componenti predefiniti supportano i nomi visualizzati con il `DisplayName` parametro:

* <xref:Microsoft.AspNetCore.Components.Forms.InputDate%601>
* <xref:Microsoft.AspNetCore.Components.Forms.InputNumber%601>
* <xref:Microsoft.AspNetCore.Components.Forms.InputSelect%601>

Nell'esempio di `InputDate` componente seguente:

* Il nome visualizzato ( `DisplayName` ) è impostato su `birthday` .
* Il componente è associato alla `BirthDate` proprietà come `DateTime` tipo.

```razor
<InputDate @bind-Value="BirthDate" DisplayName="birthday" />

@code {
    public DateTime BirthDate { get; set; }
}
```

Se l'utente non fornisce un valore di data, l'errore di convalida viene visualizzato come segue:

```
The birthday must be a date.
```

::: moniker-end

## <a name="validator-components"></a>Componenti validator

I componenti validator supportano la convalida dei form gestendo un oggetto <xref:Microsoft.AspNetCore.Components.Forms.ValidationMessageStore> per un modulo <xref:Microsoft.AspNetCore.Components.Forms.EditContext> .

Il Blazor Framework fornisce il <xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> componente per l'associazione del supporto di convalida ai moduli in base agli [attributi di convalida (annotazioni dei dati)](xref:mvc/models/validation#validation-attributes). Creare componenti validator personalizzati per elaborare i messaggi di convalida per formati diversi nella stessa pagina o nello stesso formato in diversi passaggi di elaborazione del modulo, ad esempio convalida sul lato client seguita dalla convalida lato server. L'esempio di componente validator illustrato in questa sezione, `CustomValidator` , viene usato nelle sezioni seguenti di questo articolo:

* [Convalida della logica di business](#business-logic-validation)
* [Convalida server](#server-validation)

> [!NOTE]
> In molti casi, è possibile utilizzare gli attributi di convalida delle annotazioni dei dati personalizzati anziché i componenti del validator personalizzato. Gli attributi personalizzati applicati al modello del modulo vengono attivati con l'utilizzo del <xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> componente. Se utilizzata con la convalida sul lato server, tutti gli attributi personalizzati applicati al modello devono essere eseguibili nel server. Per altre informazioni, vedere <xref:mvc/models/validation#alternatives-to-built-in-attributes>.

Creare un componente validator da <xref:Microsoft.AspNetCore.Components.ComponentBase> :

* Il modulo è <xref:Microsoft.AspNetCore.Components.Forms.EditContext> un [parametro](xref:blazor/components/cascading-values-and-parameters) di propagazione del componente.
* Quando il componente validator viene inizializzato, viene creato un nuovo oggetto <xref:Microsoft.AspNetCore.Components.Forms.ValidationMessageStore> per gestire un elenco corrente di errori di form.
* L'archivio di messaggi riceve errori quando il codice dello sviluppatore nel componente del modulo chiama il `DisplayErrors` metodo. Gli errori vengono passati al `DisplayErrors` metodo in un oggetto [`Dictionary<string, List<string>>`](xref:System.Collections.Generic.Dictionary`2) . Nel dizionario la chiave è il nome del campo del form che contiene uno o più errori. Il valore è l'elenco errori.
* I messaggi vengono cancellati quando si verifica una delle condizioni seguenti:
  * La convalida viene richiesta su <xref:Microsoft.AspNetCore.Components.Forms.EditContext> quando <xref:Microsoft.AspNetCore.Components.Forms.EditContext.OnValidationRequested> viene generato l'evento. Tutti gli errori vengono cancellati.
  * Un campo viene modificato nel form quando <xref:Microsoft.AspNetCore.Components.Forms.EditContext.OnFieldChanged> viene generato l'evento. Vengono cancellati solo gli errori per il campo.
  * Il `ClearErrors` metodo viene chiamato dal codice dello sviluppatore. Tutti gli errori vengono cancellati.

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
> Le espressioni lambda anonime sono gestori eventi registrati per <xref:Microsoft.AspNetCore.Components.Forms.EditContext.OnValidationRequested> e <xref:Microsoft.AspNetCore.Components.Forms.EditContext.OnFieldChanged> nell'esempio precedente. In questo scenario non è necessario implementare <xref:System.IDisposable> e annullare la sottoscrizione dei delegati di evento. Per altre informazioni, vedere <xref:blazor/components/lifecycle#component-disposal-with-idisposable>.

## <a name="business-logic-validation"></a>Convalida della logica di business

La convalida della logica di business può essere eseguita con un [componente validator](#validator-components) che riceve errori di form in un dizionario.

Nell'esempio seguente:

* `CustomValidator`Viene usato il componente della sezione [componenti validator](#validator-components) di questo articolo.
* Per la convalida è necessario un valore per la descrizione della nave ( `Description` ) se l'utente seleziona la `Defense` classificazione Ship ( `Classification` ).

Quando i messaggi di convalida vengono impostati nel componente, vengono aggiunti ai validator <xref:Microsoft.AspNetCore.Components.Forms.ValidationMessageStore> e mostrati in <xref:Microsoft.AspNetCore.Components.Forms.EditForm> :

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
> In alternativa all'uso di [componenti di convalida](#validator-components), è possibile usare gli attributi di convalida dell'annotazione dei dati. Gli attributi personalizzati applicati al modello del modulo vengono attivati con l'utilizzo del <xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> componente. Quando viene utilizzato con la convalida sul lato server, gli attributi devono essere eseguibili nel server. Per altre informazioni, vedere <xref:mvc/models/validation#alternatives-to-built-in-attributes>.

## <a name="server-validation"></a>Convalida server

La convalida del server può essere eseguita con un [componente validator](#validator-components)server:

* Elaborare la convalida lato client nel form con il <xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> componente.
* Quando il modulo passa la convalida lato client ( <xref:Microsoft.AspNetCore.Components.Forms.EditForm.OnValidSubmit> viene chiamato), inviare <xref:Microsoft.AspNetCore.Components.Forms.EditContext.Model?displayProperty=nameWithType> a un'API del server back-end per l'elaborazione del modulo.
* Convalida del modello di processo sul server.
* L'API server include sia la convalida delle annotazioni dei dati del Framework incorporata che la logica di convalida personalizzata fornita dallo sviluppatore. Se la convalida passa sul server, elaborare il modulo e restituire un codice di stato di esito positivo (*200-OK*). Se la convalida ha esito negativo, restituire un codice di stato di errore (*400-richiesta* non valida) e gli errori di convalida dei campi.
* Disabilitare il modulo in caso di esito positivo o visualizzare gli errori.

L'esempio seguente è basato su:

* Soluzione ospitata Blazor creata dal [ Blazor modello di progetto ospitato](xref:blazor/hosting-models#blazor-webassembly). Questo esempio può essere usato con qualsiasi soluzione ospitata protetta Blazor descritta nella pagina [ Identity relativa alla sicurezza e alla documentazione](xref:blazor/security/webassembly/index#implementation-guidance).
* Il modulo del *database Starship astronave* nell'esempio precedente della sezione relativa ai [componenti dei moduli incorporati](#built-in-forms-components) .
* BlazorComponente del Framework <xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> .
* `CustomValidator`Componente visualizzato nella sezione [componenti validator](#validator-components) .

Nell'esempio seguente, l'API del server verifica che venga fornito un valore per la descrizione della nave ( `Description` ) se l'utente seleziona la `Defense` classificazione Ship ( `Classification` ).

Inserire il `Starship` modello nel progetto della soluzione in `Shared` modo che entrambe le app client e server possano usare il modello. Poiché il modello richiede le annotazioni dei dati, aggiungere un riferimento al pacchetto per [`System.ComponentModel.Annotations`](https://www.nuget.org/packages/System.ComponentModel.Annotations) al `Shared` file di progetto del progetto:

```xml
<ItemGroup>
  <PackageReference Include="System.ComponentModel.Annotations" Version="{VERSION}" />
</ItemGroup>
```

Per determinare la versione non di anteprima più recente del pacchetto, vedere la **cronologia delle versioni** del pacchetto in [NuGet.org](https://www.nuget.org/packages/System.ComponentModel.Annotations).

Nel progetto API server aggiungere un controller per elaborare le richieste di convalida dell'astronave ( `Controllers/StarshipValidation.cs` ) e restituire i messaggi di convalida non riusciti:

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

Nell'esempio precedente, il segnaposto `{ASSEMBLY NAME}` è il nome dell'assembly dell'app (ad esempio, `BlazorSample.Server` ).

Quando si verifica un errore di convalida dell'associazione di modelli nel server, un [`ApiController`](xref:web-api/index) ( <xref:Microsoft.AspNetCore.Mvc.ApiControllerAttribute> ) restituisce normalmente una [risposta di richiesta non valida predefinita](xref:web-api/index#default-badrequest-response) con un oggetto <xref:Microsoft.AspNetCore.Mvc.ValidationProblemDetails> . La risposta contiene più dati dei soli errori di convalida, come illustrato nell'esempio seguente quando tutti i campi del modulo del *database Starship della flotta stellare* non vengono inviati e il modulo non viene convalidato:

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

Se l'API del server restituisce la risposta JSON predefinita precedente, il client può analizzare la risposta per ottenere gli elementi figlio del `errors` nodo. Tuttavia, non è consigliabile analizzare il file. L'analisi del codice JSON richiede codice aggiuntivo dopo <xref:System.Net.Http.Json.HttpContentJsonExtensions.ReadFromJsonAsync%2A> la chiamata al fine di generare un oggetto [`Dictionary<string, List<string>>`](xref:System.Collections.Generic.Dictionary`2) di errori per l'elaborazione degli errori di convalida dei moduli. Idealmente, l'API del server deve restituire solo gli errori di convalida:

```json
{
  "Identifier": ["The Identifier field is required."],
  "Classification": ["The Classification field is required."],
  "IsValidatedDesign": ["This form disallows unapproved ships."],
  "MaximumAccommodation": ["Accommodation invalid (1-100000)."]
}
```

Per modificare la risposta dell'API del server in modo che restituisca solo gli errori di convalida, modificare il delegato che viene richiamato sulle azioni annotate con <xref:Microsoft.AspNetCore.Mvc.ApiControllerAttribute> in `Startup.ConfigureServices` . Per l'endpoint API ( `/StarshipValidation` ), restituire un oggetto <xref:Microsoft.AspNetCore.Mvc.BadRequestObjectResult> con <xref:Microsoft.AspNetCore.Mvc.ModelBinding.ModelStateDictionary> . Per qualsiasi altro endpoint API, mantenere il comportamento predefinito restituendo il risultato dell'oggetto con un nuovo <xref:Microsoft.AspNetCore.Mvc.ValidationProblemDetails> :

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

Per altre informazioni, vedere <xref:web-api/handle-errors#validation-failure-error-response>.

Nel progetto client aggiungere il componente validator visualizzato nella sezione [componenti validator](#validator-components) .

Nel progetto client, il modulo *database Starship astronave* viene aggiornato in modo da visualizzare gli errori di convalida del server con la guida del `CustomValidator` componente. Quando l'API del server restituisce i messaggi di convalida, vengono aggiunti al `CustomValidator` componente <xref:Microsoft.AspNetCore.Components.Forms.ValidationMessageStore> . Gli errori sono disponibili nel formato per la <xref:Microsoft.AspNetCore.Components.Forms.EditContext> visualizzazione da parte del modulo <xref:Microsoft.AspNetCore.Components.Forms.ValidationSummary> :

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
> In alternativa ai [componenti di convalida](#validator-components), è possibile usare gli attributi di convalida delle annotazioni dei dati. Gli attributi personalizzati applicati al modello del modulo vengono attivati con l'utilizzo del <xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> componente. Quando viene utilizzato con la convalida sul lato server, gli attributi devono essere eseguibili nel server. Per altre informazioni, vedere <xref:mvc/models/validation#alternatives-to-built-in-attributes>.

> [!NOTE]
> L'approccio di convalida lato server in questa sezione è adatto a qualsiasi esempio di Blazor WebAssembly soluzione ospitata in questo set di documentazione:
>
> * [Azure Active Directory (AAD)](xref:blazor/security/webassembly/hosted-with-azure-active-directory)
> * [Azure Active Directory (AAD) B2C](xref:blazor/security/webassembly/hosted-with-azure-active-directory-b2c)
> * [Identity Server](xref:blazor/security/webassembly/hosted-with-identity-server)

## <a name="inputtext-based-on-the-input-event"></a>InputText basato sull'evento di input

Utilizzare il <xref:Microsoft.AspNetCore.Components.Forms.InputText> componente per creare un componente personalizzato che utilizza l' `input` evento anziché l' `change` evento.

Nell'esempio seguente il `CustomInputText` componente eredita il componente del Framework `InputText` e imposta l'associazione di eventi sull' `oninput` evento.

`Shared/CustomInputText.razor`:

```razor
@inherits InputText

<input @attributes="AdditionalAttributes" class="@CssClass" 
    @bind="CurrentValueAsString" @bind:event="oninput" />
```

Il `CustomInputText` componente può essere usato ovunque <xref:Microsoft.AspNetCore.Components.Forms.InputText> :

`Pages/TestForm.razor`:

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

## <a name="radio-buttons"></a>Pulsanti di opzione

::: moniker range=">= aspnetcore-5.0"

Usare `InputRadio` i componenti con il `InputRadioGroup` componente per creare un gruppo di pulsanti di opzione. Nell'esempio seguente le proprietà vengono aggiunte al `Starship` modello descritto nella sezione componenti dei [moduli incorporati](#built-in-forms-components) :

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

Aggiungere quanto segue `enums` all'app. Creare un nuovo file per conservare `enums` o aggiungere al `enums` `Starship.cs` file. Rendere `enums` accessibile al `Starship` modello e al modulo *database Starship astronave* :

```csharp
public enum Manufacturer { SpaceX, NASA, ULA, VirginGalactic, Unknown }
public enum Color { ImperialRed, SpacecruiserGreen, StarshipBlue, VoyagerOrange }
public enum Engine { Ion, Plasma, Fusion, Warp }
```

Aggiornare il modulo del *database Starship della flotta stellare* descritto nella sezione [componenti dei moduli incorporati](#built-in-forms-components) . Aggiungere i componenti da produrre:

* Un gruppo di pulsanti di opzione per il produttore della nave.
* Gruppo di pulsanti di opzione annidato per il colore e il motore della spedizione.

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
> Se `Name` viene omesso, `InputRadio` i componenti vengono raggruppati in base al predecessore più recente.

::: moniker-end

::: moniker range="< aspnetcore-5.0"

Quando si utilizzano pulsanti di opzione in un modulo, data binding viene gestito in modo diverso rispetto ad altri elementi perché i pulsanti di opzione vengono valutati come un gruppo. Il valore di ogni pulsante di opzione è fisso, ma il valore del gruppo di pulsanti di opzione è il valore del pulsante di opzione selezionato. L'esempio seguente illustra come:

* Gestire data binding per un gruppo di pulsanti di opzione.
* Supportare la convalida usando un `InputRadio` componente personalizzato.

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

Nell'esempio seguente <xref:Microsoft.AspNetCore.Components.Forms.EditForm> viene utilizzato il `InputRadio` componente precedente per ottenere e convalidare una classificazione dall'utente:

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

## <a name="binding-select-element-options-to-c-object-null-values"></a>Associazione `<select>` di opzioni di elementi a valori di oggetti C# `null`

Non esiste un modo sensato per rappresentare il valore dell'opzione di un `<select>` elemento come valore dell'oggetto C# `null` , perché:

* Gli attributi HTML non possono avere `null` valori. L'equivalente più vicino a `null` in HTML è l'assenza dell'attributo HTML dell' `value` `<option>` elemento.
* Quando si seleziona un oggetto `<option>` senza `value` attributo, il browser considera il valore come *contenuto di testo* dell' `<option>` elemento.

Il Blazor Framework non tenta di evitare il comportamento predefinito perché comporterebbe:

* Creazione di una catena di soluzioni alternative per casi speciali nel Framework.
* Modifiche di rilievo al comportamento del Framework corrente.

::: moniker range=">= aspnetcore-5.0"

L'equivalente più plausibile `null` in HTML è una *stringa vuota* `value` . Il Blazor Framework gestisce le `null` conversioni di stringa vuote per l'associazione bidirezionale a un `<select>` valore di.

::: moniker-end

::: moniker range="< aspnetcore-5.0"

Il Blazor Framework non gestisce automaticamente le `null` conversioni di stringa vuote quando si tenta l'associazione bidirezionale al `<select>` valore di un oggetto. Per ulteriori informazioni, vedere [la pagina relativa alla correzione dell'associazione `<select>` a un valore null (DotNet/aspnetcore #23221)](https://github.com/dotnet/aspnetcore/pull/23221).

::: moniker-end

## <a name="validation-support"></a>Supporto della convalida

Il <xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> componente connette il supporto di convalida usando le annotazioni dei dati a a cascata <xref:Microsoft.AspNetCore.Components.Forms.EditContext> . L'abilitazione del supporto per la convalida usando le annotazioni dei dati richiede questo movimento esplicito. Per usare un sistema di convalida diverso rispetto alle annotazioni dei dati, sostituire <xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> con un'implementazione personalizzata. L'implementazione del ASP.NET Core è disponibile per l'ispezione nell'origine riferimento: [`DataAnnotationsValidator`](https://github.com/dotnet/AspNetCore/blob/master/src/Components/Forms/src/DataAnnotationsValidator.cs) / [`AddDataAnnotationsValidation`](https://github.com/dotnet/AspNetCore/blob/master/src/Components/Forms/src/EditContextDataAnnotationsExtensions.cs) . I collegamenti precedenti a origine riferimento forniscono il codice dal ramo del repository `master` , che rappresenta lo sviluppo corrente dell'unità di prodotto per la prossima versione di ASP.NET Core. Per selezionare il ramo per una versione diversa, usare il selettore di ramo GitHub (ad esempio `release/3.1` ).

Blazor esegue due tipi di convalida:

* La *convalida dei campi* viene eseguita quando l'utente esce da un campo. Durante la convalida dei campi, il <xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> componente associa tutti i risultati della convalida segnalati al campo.
* La *convalida del modello* viene eseguita quando l'utente invia il modulo. Durante la convalida del modello, il <xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> componente tenta di determinare il campo in base al nome del membro segnalato dal risultato della convalida. I risultati della convalida che non sono associati a un singolo membro sono associati al modello anziché a un campo.

### <a name="validation-summary-and-validation-message-components"></a>Componenti del messaggio di convalida e di riepilogo della convalida

Il <xref:Microsoft.AspNetCore.Components.Forms.ValidationSummary> componente riepiloga tutti i messaggi di convalida, che è simile all' [Helper tag del riepilogo di convalida](xref:mvc/views/working-with-forms#the-validation-summary-tag-helper):

```razor
<ValidationSummary />
```

Messaggi di convalida dell'output per un modello specifico con il `Model` parametro:
  
```razor
<ValidationSummary Model="@starship" />
```

Il <xref:Microsoft.AspNetCore.Components.Forms.ValidationMessage%601> componente Visualizza i messaggi di convalida per un campo specifico, che è simile all' [Helper tag del messaggio di convalida](xref:mvc/views/working-with-forms#the-validation-message-tag-helper). Specificare il campo per la convalida con l' `For` attributo e un'espressione lambda che denomina la proprietà del modello:

```razor
<ValidationMessage For="@(() => starship.MaximumAccommodation)" />
```

I <xref:Microsoft.AspNetCore.Components.Forms.ValidationMessage%601> <xref:Microsoft.AspNetCore.Components.Forms.ValidationSummary> componenti e supportano attributi arbitrari. Qualsiasi attributo che non corrisponde a un parametro component viene aggiunto all' `<div>` elemento generato o `<ul>` .

Controllare lo stile dei messaggi di convalida nel foglio di stile dell'app ( `wwwroot/css/app.css` o `wwwroot/css/site.css` ). La `validation-message` classe predefinita imposta il colore del testo dei messaggi di convalida su rosso:

```css
.validation-message {
    color: red;
}
```

### <a name="custom-validation-attributes"></a>Attributi di convalida personalizzati

Per assicurarsi che un risultato di convalida venga associato correttamente a un campo quando si usa un [attributo di convalida personalizzato](xref:mvc/models/validation#custom-attributes), passare il contesto di convalida <xref:System.ComponentModel.DataAnnotations.ValidationContext.MemberName> durante la creazione di <xref:System.ComponentModel.DataAnnotations.ValidationResult> :

```csharp
using System;
using System.ComponentModel.DataAnnotations;

private class CustomValidator : ValidationAttribute
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
> <xref:System.ComponentModel.DataAnnotations.ValidationContext.GetService%2A?displayProperty=nameWithType> è `null`. L'inserimento di servizi per la convalida nel `IsValid` metodo non è supportato.

::: moniker range=">= aspnetcore-5.0"

## <a name="custom-validation-class-attributes"></a>Attributi della classe di convalida personalizzata

I nomi delle classi di convalida personalizzate sono utili per l'integrazione con Framework CSS, ad esempio [bootstrap](https://getbootstrap.com/). Per specificare i nomi delle classi di convalida personalizzate, creare una classe derivata da `FieldCssClassProvider` e impostare la classe nell' <xref:Microsoft.AspNetCore.Components.Forms.EditContext> istanza:

```csharp
var editContext = new EditContext(model);
editContext.SetFieldCssClassProvider(new MyFieldClassProvider());

...

private class MyFieldClassProvider : FieldCssClassProvider
{
    public override string GetFieldCssClass(EditContext editContext, 
        in FieldIdentifier fieldIdentifier)
    {
        var isValid = !editContext.GetValidationMessages(fieldIdentifier).Any();

        return isValid ? "good field" : "bad field";
    }
}
```

::: moniker-end

### <a name="blazor-data-annotations-validation-package"></a>Blazor pacchetto di convalida delle annotazioni dei dati

[`Microsoft.AspNetCore.Components.DataAnnotations.Validation`](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.DataAnnotations.Validation)È un pacchetto che colma i gap dell'esperienza di convalida usando il <xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> componente. Il pacchetto è attualmente *sperimentale*.

> [!NOTE]
> Il [`Microsoft.AspNetCore.Components.DataAnnotations.Validation`](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.DataAnnotations.Validation) pacchetto dispone di una versione più recente di *release candidate* in [NuGet.org](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.DataAnnotations.Validation). Continuare a usare il pacchetto *sperimentale* versione finale al momento. L'assembly del pacchetto potrebbe essere spostato nel Framework o nel runtime in una versione futura. Per ulteriori aggiornamenti, vedere il [repository GitHub degli annunci](https://github.com/aspnet/Announcements), il [repository GitHub DotNet/aspnetcore](https://github.com/dotnet/aspnetcore)o questa sezione dell'argomento.

::: moniker range="< aspnetcore-5.0"

### <a name="compareproperty-attribute"></a>Attributo `[CompareProperty]`

<xref:System.ComponentModel.DataAnnotations.CompareAttribute>Non funziona bene con il <xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> componente perché non associa il risultato della convalida a un membro specifico. Ciò può comportare un comportamento incoerente tra la convalida a livello di campo e quando l'intero modello viene convalidato in un invio. Il [`Microsoft.AspNetCore.Components.DataAnnotations.Validation`](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.DataAnnotations.Validation) pacchetto *sperimentale* introduce un attributo di convalida aggiuntivo, `ComparePropertyAttribute` , che aggira queste limitazioni. In un' Blazor applicazione, `[CompareProperty]` è una sostituzione diretta per l' [ `[Compare]` attributo](xref:System.ComponentModel.DataAnnotations.CompareAttribute).

::: moniker-end

### <a name="nested-models-collection-types-and-complex-types"></a>Modelli annidati, tipi di raccolta e tipi complessi

Blazor fornisce supporto per la convalida dell'input del form utilizzando le annotazioni dei dati con l'oggetto incorporato <xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> . Tuttavia, l'oggetto <xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> convalida solo le proprietà di primo livello del modello associate al form che non sono proprietà di tipo raccolta o complesso.

Per convalidare l'intero grafico di oggetti del modello associato, incluse le proprietà della raccolta e del tipo complesso, usare l'oggetto `ObjectGraphDataAnnotationsValidator` fornito dal pacchetto *sperimentale* [`Microsoft.AspNetCore.Components.DataAnnotations.Validation`](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.DataAnnotations.Validation) :

```razor
<EditForm Model="@model" OnValidSubmit="@HandleValidSubmit">
    <ObjectGraphDataAnnotationsValidator />
    ...
</EditForm>
```

Annotare le proprietà del modello con `[ValidateComplexType]` . Nelle classi di modelli seguenti la `ShipDescription` classe contiene annotazioni di dati aggiuntive da convalidare quando il modello è associato al form:

`Starship.cs`:

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

`ShipDescription.cs`:

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

### <a name="enable-the-submit-button-based-on-form-validation"></a>Abilita il pulsante Invia in base alla convalida del modulo

Per abilitare e disabilitare il pulsante Invia in base alla convalida del modulo:

* Utilizzare il modulo <xref:Microsoft.AspNetCore.Components.Forms.EditContext> per assegnare il modello quando il componente viene inizializzato.
* Convalidare il form nel callback del contesto <xref:Microsoft.AspNetCore.Components.Forms.EditContext.OnFieldChanged> per abilitare e disabilitare il pulsante Submit.
* Implementare <xref:System.IDisposable> e annullare la sottoscrizione del gestore eventi nel `Dispose` metodo. Per altre informazioni, vedere <xref:blazor/components/lifecycle#component-disposal-with-idisposable>.

> [!NOTE]
> Quando si usa un oggetto <xref:Microsoft.AspNetCore.Components.Forms.EditContext> , non assegnare anche un oggetto <xref:Microsoft.AspNetCore.Components.Forms.EditForm.Model> a <xref:Microsoft.AspNetCore.Components.Forms.EditForm> .

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

Nell'esempio precedente, impostare `formInvalid` su `false` if:

* Il modulo viene precaricato con valori predefiniti validi.
* Si desidera attivare il pulsante Invia quando il modulo viene caricato.

Un effetto collaterale dell'approccio precedente è che un <xref:Microsoft.AspNetCore.Components.Forms.ValidationSummary> componente viene popolato con campi non validi dopo che l'utente interagisce con un qualsiasi campo. Questo scenario può essere risolto in uno dei modi seguenti:

* Non usare un <xref:Microsoft.AspNetCore.Components.Forms.ValidationSummary> componente nel form.
* Rendere <xref:Microsoft.AspNetCore.Components.Forms.ValidationSummary> visibile il componente quando viene selezionato il pulsante Invia, ad esempio in un `HandleValidSubmit` metodo.

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

## <a name="troubleshoot"></a>Risolvere problemi

> InvalidOperationException: EditForm richiede un parametro di modello o un parametro EditContext, ma non entrambi.

Verificare che <xref:Microsoft.AspNetCore.Components.Forms.EditForm> disponga di un oggetto <xref:Microsoft.AspNetCore.Components.Forms.EditForm.Model> **o** <xref:Microsoft.AspNetCore.Components.Forms.EditContext> . Non usare entrambi per lo stesso formato.

Quando si assegna un <xref:Microsoft.AspNetCore.Components.Forms.EditForm.Model> al form, verificare che venga creata un'istanza del tipo di modello, come illustrato nell'esempio seguente:

```csharp
private ExampleModel exampleModel = new ExampleModel();
```

::: moniker range=">= aspnetcore-5.0"

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:blazor/file-uploads>

::: moniker-end
