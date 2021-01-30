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
# <a name="aspnet-core-no-locblazor-data-binding"></a>ASP.NET Core Blazor Data Binding

Di [Luke Latham](https://github.com/guardrex), [Daniel Roth](https://github.com/danroth27)e [Steve Sanderson](https://github.com/SteveSandersonMS)

Razor i componenti di forniscono funzionalità di data binding tramite un attributo dell'elemento HTML denominato [`@bind`](xref:mvc/views/razor#bind) con un campo, una proprietà o un Razor valore di espressione.

Nell'esempio seguente viene associato un `<input>` elemento al `currentValue` campo e un `<input>` elemento alla `CurrentValue` proprietà:

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

Quando uno degli elementi perde lo stato attivo, viene aggiornato il campo o la proprietà associata.

La casella di testo viene aggiornata nell'interfaccia utente solo quando viene eseguito il rendering del componente, non in risposta alla modifica del valore della proprietà o del campo. Poiché i componenti eseguono il rendering autonomamente dopo l'esecuzione del codice del gestore eventi, gli aggiornamenti dei campi e delle proprietà vengono in *genere* riflessi nell'interfaccia utente immediatamente dopo l'attivazione di un gestore eventi.

L'utilizzo [`@bind`](xref:mvc/views/razor#bind) di con la `CurrentValue` Proprietà ( `<input @bind="CurrentValue" />` ) equivale essenzialmente a quanto segue:

```razor
<input value="@CurrentValue"
    @onchange="@((ChangeEventArgs __e) => CurrentValue = 
        __e.Value.ToString())" />

@code {
    private string CurrentValue { get; set; }
}
```

Quando viene eseguito il rendering del componente, l'oggetto `value` dell'elemento di input deriva dalla `CurrentValue` Proprietà. Quando l'utente digita nella casella di testo e modifica lo stato attivo dell'elemento, `onchange` viene generato l'evento e la `CurrentValue` proprietà viene impostata sul valore modificato. In realtà, la generazione del codice è più complessa rispetto a [`@bind`](xref:mvc/views/razor#bind) quando vengono gestiti i casi in cui vengono eseguite le conversioni dei tipi. In linea di principio, [`@bind`](xref:mvc/views/razor#bind) associa il valore corrente di un'espressione a un `value` attributo e gestisce le modifiche utilizzando il gestore registrato.

Associare una proprietà o un campo ad altri eventi includendo anche un `@bind:event` attributo con un `event` parametro. Nell'esempio seguente viene associata la `CurrentValue` proprietà nell' `oninput` evento:

```razor
<input @bind="CurrentValue" @bind:event="oninput" />

@code {
    private string CurrentValue { get; set; }
}
```

Diversamente da `onchange` , che viene attivato quando l'elemento perde lo stato attivo, `oninput` viene attivato quando viene modificato il valore della casella di testo.

<!-- Hold location for resolution of https://github.com/dotnet/AspNetCore.Docs/issues/19721 -->

L'associazione di attributi distingue tra maiuscole e minuscole:

* `@bind` è valido.
* `@Bind` e `@BIND` non sono validi.

## <a name="unparsable-values"></a>Valori non analizzabili

Quando un utente fornisce un valore non analizzabile a un elemento associato a un oggetto DataBound, il valore non analizzabile viene automaticamente ripristinato al valore precedente quando viene attivato l'evento bind.

Considerare lo scenario seguente:

* Un `<input>` elemento è associato a un `int` tipo con un valore iniziale di `123` :

  ```razor
  <input @bind="inputValue" />

  @code {
      private int inputValue = 123;
  }
  ```

* L'utente aggiorna il valore dell'elemento a `123.45` nella pagina e modifica lo stato attivo dell'elemento.

Nello scenario precedente, il valore dell'elemento viene ripristinato in `123` . Quando il valore `123.45` viene rifiutato a favore del valore originale di `123` , l'utente riconosce che il relativo valore non è stato accettato.

Per impostazione predefinita, l'associazione si applica all' `onchange` evento dell'elemento ( `@bind="{PROPERTY OR FIELD}"` ). Utilizzare `@bind="{PROPERTY OR FIELD}" @bind:event={EVENT}` per attivare l'associazione in un evento diverso. Per l' `oninput` evento ( `@bind:event="oninput"` ), la riversione viene eseguita dopo qualsiasi sequenza di tasti che introduce un valore non analizzabile. Quando la destinazione `oninput` è l'evento con un `int` tipo associato a, a un utente viene impedito di digitare un `.` carattere. Un `.` carattere viene immediatamente rimosso, quindi l'utente riceve il feedback immediato che sono consentiti solo numeri interi. Esistono scenari in cui il ripristino del valore dell' `oninput` evento non è ideale, ad esempio quando l'utente deve essere autorizzato a cancellare un valore non analizzabile `<input>` . Le alternative includono:

* Non usare l' `oninput` evento. Usare l' `onchange` evento predefinito (specificare solo `@bind="{PROPERTY OR FIELD}"` ), in cui un valore non valido non viene ripristinato fino a quando l'elemento non perde lo stato attivo.
* Eseguire l'associazione a un tipo nullable, ad esempio `int?` o, `string` e fornire la logica personalizzata per gestire le voci non valide.
* Utilizzare un [componente di convalida del modulo](xref:blazor/forms-validation), ad esempio <xref:Microsoft.AspNetCore.Components.Forms.InputNumber%601> o <xref:Microsoft.AspNetCore.Components.Forms.InputDate%601> . I componenti di convalida dei moduli includono il supporto predefinito per la gestione di input non validi. Per altre informazioni, vedere <xref:blazor/forms-validation>. Componenti di convalida dei moduli:
  * Consente all'utente di fornire un input non valido e di ricevere errori di convalida nell'oggetto associato <xref:Microsoft.AspNetCore.Components.Forms.EditContext> .
  * Visualizzare gli errori di convalida nell'interfaccia utente senza interferire con l'utente che immette dati Web Form aggiuntivi.

## <a name="format-strings"></a>Stringhe di formato

Il data binding funziona con <xref:System.DateTime> stringhe di formato usando `@bind:format` . Altre espressioni di formato, ad esempio i formati di valuta o numerici, non sono disponibili in questo momento.

```razor
<input @bind="startDate" @bind:format="yyyy-MM-dd" />

@code {
    private DateTime startDate = new DateTime(2020, 1, 1);
}
```

Nel codice precedente, il `<input>` valore predefinito del tipo di campo dell'elemento ( `type` ) è `text` . `@bind:format` è supportato per l'associazione dei tipi .NET seguenti:

* <xref:System.DateTime?displayProperty=fullName>
* <xref:System.DateTime?displayProperty=fullName>?
* <xref:System.DateTimeOffset?displayProperty=fullName>
* <xref:System.DateTimeOffset?displayProperty=fullName>?

L' `@bind:format` attributo specifica il formato di data da applicare all'oggetto `value` dell' `<input>` elemento. Il formato viene usato anche per analizzare il valore quando `onchange` si verifica un evento.

Non è consigliabile specificare un formato per il `date` tipo di campo perché Blazor include il supporto incorporato per formattare le date. A prescindere dalla raccomandazione, utilizzare solo il `yyyy-MM-dd` formato data per il binding per funzionare correttamente se viene fornito un formato con il `date` tipo di campo:

```razor
<input type="date" @bind="startDate" @bind:format="yyyy-MM-dd">
```

## <a name="binding-with-component-parameters"></a>Associazione con parametri dei componenti

Uno scenario comune è l'associazione di una proprietà in un componente figlio a una proprietà nell'elemento padre. Questo scenario è denominato *Binding concatenato* perché si verificano simultaneamente più livelli di associazione.

I [parametri del componente](xref:blazor/components/index#component-parameters) consentono di associare le proprietà di un componente padre con la `@bind-{PROPERTY}` sintassi.

Le associazioni concatenate non possono essere implementate con [`@bind`](xref:mvc/views/razor#bind) la sintassi nel componente figlio. Un gestore eventi e un valore devono essere specificati separatamente per supportare l'aggiornamento della proprietà nell'elemento padre dal componente figlio.

Il componente padre utilizza ancora la [`@bind`](xref:mvc/views/razor#bind) sintassi per configurare l'associazione dati con il componente figlio.

Il `Child` componente seguente ( `Shared/Child.razor` ) ha un `Year` parametro del componente e un `YearChanged` callback:

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

Il callback ( <xref:Microsoft.AspNetCore.Components.EventCallback%601> ) deve essere denominato come nome del parametro del componente seguito dal `Changed` suffisso "" ( `{PARAMETER NAME}Changed` ). Nell'esempio precedente, il callback è denominato `YearChanged` . <xref:Microsoft.AspNetCore.Components.EventCallback.InvokeAsync%2A?displayProperty=nameWithType> richiama il delegato associato all'associazione con l'argomento fornito e invia una notifica degli eventi per la proprietà modificata.

Nel componente seguente `Parent` ( `Parent.razor` ) il `year` campo è associato al `Year` parametro del componente figlio:

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

Il `Year` parametro è associabile perché contiene un evento complementare `YearChanged` che corrisponde al tipo del `Year` parametro.

Per convenzione, una proprietà può essere associata a un gestore eventi corrispondente includendo un `@bind-{PROPERTY}:event` attributo assegnato al gestore. `<Child @bind-Year="year" />` equivale alla scrittura:

```razor
<Child @bind-Year="year" @bind-Year:event="YearChanged" />
```

In un esempio più sofisticato e reale, il componente seguente `PasswordField` ( `PasswordField.razor` ):

* Imposta `<input>` il valore di un elemento su un `password` campo.
* Espone le modifiche di una `Password` proprietà a un componente padre con un oggetto [`EventCallback`](xref:blazor/components/event-handling#eventcallback) che passa il valore corrente del campo figlio `password` come argomento.
* Usa l' `onclick` evento per attivare il `ToggleShowPassword` metodo. Per altre informazioni, vedere <xref:blazor/components/event-handling>.

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

Il `PasswordField` componente viene usato in un altro componente:

```razor
@page "/Parent"

<h1>Parent Component</h1>

<PasswordField @bind-Password="password" />

@code {
    private string password;
}
```

Eseguire controlli o errori di trap nel metodo che richiama il delegato dell'associazione. Nell'esempio seguente viene fornito un feedback immediato all'utente se viene utilizzato uno spazio nel valore della password:

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

Per altre informazioni su <xref:Microsoft.AspNetCore.Components.EventCallback%601>, vedere <xref:blazor/components/event-handling#eventcallback>.

## <a name="bind-across-more-than-two-components"></a>Associazione tra più di due componenti

È possibile eseguire l'associazione tramite un numero qualsiasi di componenti annidati, ma è necessario rispettare il flusso unidirezionale dei dati:

* Le notifiche *delle modifiche scorrono verso l'alto nella gerarchia*.
* I nuovi valori dei parametri *scorrono verso il basso nella gerarchia*.

Un approccio comune e consigliato consiste nel memorizzare solo i dati sottostanti nel componente padre, per evitare confusione sullo stato che deve essere aggiornato.

I componenti seguenti illustrano i concetti precedenti:

`ParentComponent.razor`:

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

`ChildComponent.razor`:

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

`GrandchildComponent.razor`:

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

Per un approccio alternativo adatto alla condivisione dei dati in memoria tra i componenti che non sono necessariamente annidati, vedere la sezione relativa al *servizio contenitore di stato in memoria* dell' <xref:blazor/state-management> articolo.

## <a name="additional-resources"></a>Risorse aggiuntive

* [Associazione ai pulsanti di opzione in un modulo](xref:blazor/forms-validation#radio-buttons)
* [Associazione `<select>` di opzioni di elementi a valori di oggetti C# `null` in un form](xref:blazor/forms-validation#binding-select-element-options-to-c-object-null-values)
