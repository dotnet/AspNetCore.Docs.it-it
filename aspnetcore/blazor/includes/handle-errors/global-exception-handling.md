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
<span data-ttu-id="a6236-101">Blazor è un framework lato client applicazione a pagina singola (SPA).</span><span class="sxs-lookup"><span data-stu-id="a6236-101">Blazor is a single-page application (SPA) client-side framework.</span></span> <span data-ttu-id="a6236-102">Il browser funge da host dell'app e quindi funge da pipeline di elaborazione per i singoli Razor componenti in base alle richieste URI per la navigazione e gli asset statici.</span><span class="sxs-lookup"><span data-stu-id="a6236-102">The browser serves as the app's host and thus acts as the processing pipeline for individual Razor components based on URI requests for navigation and static assets.</span></span> <span data-ttu-id="a6236-103">Diversamente dalle app ASP.NET Core eseguite sul server con una pipeline di elaborazione middleware, non esiste una pipeline middleware che elabora le richieste per i Razor componenti che possono essere sfruttati per la gestione degli errori globale.</span><span class="sxs-lookup"><span data-stu-id="a6236-103">Unlike ASP.NET Core apps that run on the server with a middleware processing pipeline, there is no middleware pipeline that processes requests for Razor components that can be leveraged for global error handling.</span></span> <span data-ttu-id="a6236-104">Tuttavia, un'app può usare un componente di elaborazione degli errori come valore di propagazione per elaborare gli errori in modo centralizzato.</span><span class="sxs-lookup"><span data-stu-id="a6236-104">However, an app can use an error processing component as a cascading value to process errors in a centralized way.</span></span>

<span data-ttu-id="a6236-105">Il `Error` componente seguente passa a se stesso come [`CascadingValue`](xref:blazor/components/cascading-values-and-parameters#cascadingvalue-component) ai componenti figlio.</span><span class="sxs-lookup"><span data-stu-id="a6236-105">The following `Error` component passes itself as a [`CascadingValue`](xref:blazor/components/cascading-values-and-parameters#cascadingvalue-component) to child components.</span></span> <span data-ttu-id="a6236-106">Nell'esempio seguente viene semplicemente registrato l'errore, ma i metodi del componente possono elaborare gli errori in qualsiasi modo richiesto dall'app, anche tramite l'utilizzo di più metodi di elaborazione degli errori.</span><span class="sxs-lookup"><span data-stu-id="a6236-106">The following example merely logs the error, but methods of the component can process errors in any way required by the app, including through the use of multiple error processing methods.</span></span> <span data-ttu-id="a6236-107">Un vantaggio dell'uso di un componente rispetto all'uso di un [servizio inserito](xref:blazor/fundamentals/dependency-injection) o di un'implementazione di un logger personalizzato è che un componente a cascata può eseguire il rendering del contenuto e applicare gli stili CSS quando si verifica un errore.</span><span class="sxs-lookup"><span data-stu-id="a6236-107">An advantage of using a component over using an [injected service](xref:blazor/fundamentals/dependency-injection) or a custom logger implementation is that a cascaded component can render content and apply CSS styles when an error occurs.</span></span>

<span data-ttu-id="a6236-108">`Shared/Error.razor`:</span><span class="sxs-lookup"><span data-stu-id="a6236-108">`Shared/Error.razor`:</span></span>

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

<span data-ttu-id="a6236-109">Nel componente eseguire il `App` wrapping del `Router` componente con il `Error` componente.</span><span class="sxs-lookup"><span data-stu-id="a6236-109">In the `App` component, wrap the `Router` component with the `Error` component.</span></span> <span data-ttu-id="a6236-110">In questo modo, il componente può propagarsi `Error` a qualsiasi componente dell'app in cui il `Error` componente viene ricevuto come [`CascadingParameter`](xref:blazor/components/cascading-values-and-parameters#cascadingparameter-attribute) .</span><span class="sxs-lookup"><span data-stu-id="a6236-110">This permits the `Error` component to cascade down to any component of the app where the `Error` component is received as a [`CascadingParameter`](xref:blazor/components/cascading-values-and-parameters#cascadingparameter-attribute).</span></span>

<span data-ttu-id="a6236-111">`App.razor`:</span><span class="sxs-lookup"><span data-stu-id="a6236-111">`App.razor`:</span></span>

```razor
<Error>
    <Router ...>
        ...
    </Router>
</Error>
```

<span data-ttu-id="a6236-112">Per elaborare gli errori in un componente:</span><span class="sxs-lookup"><span data-stu-id="a6236-112">To process errors in a component:</span></span>

* <span data-ttu-id="a6236-113">Designare il `Error` componente come [`CascadingParameter`](xref:blazor/components/cascading-values-and-parameters#cascadingparameter-attribute) nel [`@code`](xref:mvc/views/razor#code) blocco:</span><span class="sxs-lookup"><span data-stu-id="a6236-113">Designate the `Error` component as a [`CascadingParameter`](xref:blazor/components/cascading-values-and-parameters#cascadingparameter-attribute) in the [`@code`](xref:mvc/views/razor#code) block:</span></span>

  ```razor
  [CascadingParameter]
  public Error Error { get; set; }
  ```

* <span data-ttu-id="a6236-114">Chiamare un metodo di elaborazione degli errori in qualsiasi `catch` blocco con un tipo di eccezione appropriato.</span><span class="sxs-lookup"><span data-stu-id="a6236-114">Call an error processing method in any `catch` block with an appropriate exception type.</span></span> <span data-ttu-id="a6236-115">Il `Error` componente di esempio offre un solo `ProcessError` metodo, ma il componente di elaborazione degli errori può fornire un numero qualsiasi di metodi di elaborazione degli errori per risolvere i requisiti di elaborazione degli errori alternativi nell'intera app.</span><span class="sxs-lookup"><span data-stu-id="a6236-115">The example `Error` component only offers a single `ProcessError` method, but the error processing component can provide any number of error processing methods to address alternative error processing requirements throughout the app.</span></span>

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

<span data-ttu-id="a6236-116">Utilizzando il componente e il metodo di esempio precedenti `Error` `ProcessError` , la console degli strumenti di sviluppo del browser indica l'errore registrato intercettato:</span><span class="sxs-lookup"><span data-stu-id="a6236-116">Using the preceding example `Error` component and `ProcessError` method, the browser's developer tools console indicates the trapped, logged error:</span></span>

> <span data-ttu-id="a6236-117">Fail: Blazor Sample. Shared. Error [0] errore: ProcessError-Type: System. NullReferenceException Message: riferimento oggetto non impostato su un'istanza di un oggetto.</span><span class="sxs-lookup"><span data-stu-id="a6236-117">fail: BlazorSample.Shared.Error[0] Error:ProcessError - Type: System.NullReferenceException Message: Object reference not set to an instance of an object.</span></span>

<span data-ttu-id="a6236-118">Se il `ProcessError` Metodo partecipa direttamente al rendering, ad esempio la visualizzazione di una barra dei messaggi di errore personalizzata o la modifica degli stili CSS degli elementi sottoposti a rendering, chiamare [`StateHasChanged`](xref:blazor/components/lifecycle#state-changes) alla fine del `ProcessErrors` metodo per eseguire nuovamente il rendering dell'interfaccia utente.</span><span class="sxs-lookup"><span data-stu-id="a6236-118">If the `ProcessError` method directly participates in rendering, such as showing a custom error message bar or changing the CSS styles of the rendered elements, call [`StateHasChanged`](xref:blazor/components/lifecycle#state-changes) at the end of the `ProcessErrors` method to rerender the UI.</span></span>
