---
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
ms.openlocfilehash: 1aa36c8d91dbd92485e85f223f2391303bebac42
ms.sourcegitcommit: a1db01b4d3bd8c57d7a9c94ce122a6db68002d66
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/04/2021
ms.locfileid: "102109709"
---
Blazor è un framework lato client applicazione a pagina singola (SPA). Il browser funge da host dell'app e quindi funge da pipeline di elaborazione per i singoli Razor componenti in base alle richieste URI per la navigazione e gli asset statici. Diversamente dalle app ASP.NET Core eseguite sul server con una pipeline di elaborazione middleware, non esiste una pipeline middleware che elabora le richieste per i Razor componenti che possono essere sfruttati per la gestione degli errori globale. Tuttavia, un'app può usare un componente di elaborazione degli errori come valore di propagazione per elaborare gli errori in modo centralizzato.

Il `Error` componente seguente passa a se stesso come [`CascadingValue`](xref:blazor/components/cascading-values-and-parameters#cascadingvalue-component) ai componenti figlio. Nell'esempio seguente viene semplicemente registrato l'errore, ma i metodi del componente possono elaborare gli errori in qualsiasi modo richiesto dall'app, anche tramite l'utilizzo di più metodi di elaborazione degli errori. Un vantaggio dell'uso di un componente rispetto all'uso di un [servizio inserito](xref:blazor/fundamentals/dependency-injection) o di un'implementazione di un logger personalizzato è che un componente a cascata può eseguire il rendering del contenuto e applicare gli stili CSS quando si verifica un errore.

`Shared/Error.razor`:

```razor
@using Microsoft.Extensions.Logging
@inject ILogger<Error> Logger

<CascadingValue Value=this>
    @ChildContent
</CascadingValue>

@code {
    [Parameter]
    public RenderFragment ChildContent { get; set; }

    public void ProcessError(Exception ex)
    {
        Logger.LogError("Error:ProcessError - Type: {Type} Message: {Message}", 
            ex.GetType(), ex.Message);
    }
}
```

Nel componente eseguire il `App` wrapping del `Router` componente con il `Error` componente. In questo modo, il componente può propagarsi `Error` a qualsiasi componente dell'app in cui il `Error` componente viene ricevuto come [`CascadingParameter`](xref:blazor/components/cascading-values-and-parameters#cascadingparameter-attribute) .

`App.razor`:

```razor
<Error>
    <Router ...>
        ...
    </Router>
</Error>
```

Per elaborare gli errori in un componente:

* Designare il `Error` componente come [`CascadingParameter`](xref:blazor/components/cascading-values-and-parameters#cascadingparameter-attribute) nel [`@code`](xref:mvc/views/razor#code) blocco:

  ```razor
  [CascadingParameter]
  public Error Error { get; set; }
  ```

* Chiamare un metodo di elaborazione degli errori in qualsiasi `catch` blocco con un tipo di eccezione appropriato. Il `Error` componente di esempio offre un solo `ProcessError` metodo, ma il componente di elaborazione degli errori può fornire un numero qualsiasi di metodi di elaborazione degli errori per risolvere i requisiti di elaborazione degli errori alternativi nell'intera app.

  ```csharp
  try
  {
      ...
  }
  catch (Exception ex)
  {
      Error.ProcessError(ex);
  }
  ```

Utilizzando il componente e il metodo di esempio precedenti `Error` `ProcessError` , la console degli strumenti di sviluppo del browser indica l'errore registrato intercettato:

> Fail: Blazor Sample. Shared. Error [0] errore: ProcessError-Type: System. NullReferenceException Message: riferimento oggetto non impostato su un'istanza di un oggetto.

Se il `ProcessError` Metodo partecipa direttamente al rendering, ad esempio la visualizzazione di una barra dei messaggi di errore personalizzata o la modifica degli stili CSS degli elementi sottoposti a rendering, chiamare [`StateHasChanged`](xref:blazor/components/lifecycle#state-changes) alla fine del `ProcessErrors` metodo per eseguire nuovamente il rendering dell'interfaccia utente.
