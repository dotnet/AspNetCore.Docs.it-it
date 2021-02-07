---
title: Ciclo di vita ASP.NET Core Blazor
author: guardrex
description: Informazioni su come usare i Razor metodi del ciclo di vita del componente nelle Blazor app ASP.NET Core.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 11/06/2020
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
uid: blazor/components/lifecycle
ms.openlocfilehash: f19d25006009723a8e69f24af92155f65c2195fe
ms.sourcegitcommit: 19a004ff2be73876a9ef0f1ac44d0331849ad159
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/07/2021
ms.locfileid: "99804461"
---
# <a name="aspnet-core-blazor-lifecycle"></a><span data-ttu-id="680d8-103">Ciclo di vita ASP.NET Core Blazor</span><span class="sxs-lookup"><span data-stu-id="680d8-103">ASP.NET Core Blazor lifecycle</span></span>

<span data-ttu-id="680d8-104">Di [Luke Latham](https://github.com/guardrex) e [Daniel Roth](https://github.com/danroth27)</span><span class="sxs-lookup"><span data-stu-id="680d8-104">By [Luke Latham](https://github.com/guardrex) and [Daniel Roth](https://github.com/danroth27)</span></span>

<span data-ttu-id="680d8-105">Il Blazor Framework include metodi del ciclo di vita sincroni e asincroni.</span><span class="sxs-lookup"><span data-stu-id="680d8-105">The Blazor framework includes synchronous and asynchronous lifecycle methods.</span></span> <span data-ttu-id="680d8-106">Eseguire l'override dei metodi Lifecycle per eseguire operazioni aggiuntive sui componenti durante l'inizializzazione e il rendering dei componenti.</span><span class="sxs-lookup"><span data-stu-id="680d8-106">Override lifecycle methods to perform additional operations on components during component initialization and rendering.</span></span>

<span data-ttu-id="680d8-107">I diagrammi seguenti illustrano il ciclo di vita Blazor .</span><span class="sxs-lookup"><span data-stu-id="680d8-107">The following diagrams illustrate the Blazor lifecycle.</span></span> <span data-ttu-id="680d8-108">I metodi del ciclo di vita sono definiti con esempi nelle sezioni seguenti di questo articolo.</span><span class="sxs-lookup"><span data-stu-id="680d8-108">Lifecycle methods are defined with examples in the following sections of this article.</span></span>

<span data-ttu-id="680d8-109">Eventi del ciclo di vita del componente:</span><span class="sxs-lookup"><span data-stu-id="680d8-109">Component lifecycle events:</span></span>

1. <span data-ttu-id="680d8-110">Se il componente esegue il rendering per la prima volta in una richiesta:</span><span class="sxs-lookup"><span data-stu-id="680d8-110">If the component is rendering for the first time on a request:</span></span>
   * <span data-ttu-id="680d8-111">Creare l'istanza del componente.</span><span class="sxs-lookup"><span data-stu-id="680d8-111">Create the component's instance.</span></span>
   * <span data-ttu-id="680d8-112">Eseguire l'inserimento della proprietà.</span><span class="sxs-lookup"><span data-stu-id="680d8-112">Perform property injection.</span></span> <span data-ttu-id="680d8-113">Eseguire [`SetParametersAsync`](#before-parameters-are-set) .</span><span class="sxs-lookup"><span data-stu-id="680d8-113">Run [`SetParametersAsync`](#before-parameters-are-set).</span></span>
   * <span data-ttu-id="680d8-114">Chiamare [`OnInitialized{Async}`](#component-initialization-methods) .</span><span class="sxs-lookup"><span data-stu-id="680d8-114">Call [`OnInitialized{Async}`](#component-initialization-methods).</span></span> <span data-ttu-id="680d8-115">Se <xref:System.Threading.Tasks.Task> viene restituito un oggetto, <xref:System.Threading.Tasks.Task> viene atteso e viene eseguito il rendering del componente.</span><span class="sxs-lookup"><span data-stu-id="680d8-115">If a <xref:System.Threading.Tasks.Task> is returned, the <xref:System.Threading.Tasks.Task> is awaited and the component is rendered.</span></span> <span data-ttu-id="680d8-116">Se <xref:System.Threading.Tasks.Task> non viene restituito alcun oggetto, viene eseguito il rendering del componente.</span><span class="sxs-lookup"><span data-stu-id="680d8-116">If a <xref:System.Threading.Tasks.Task> isn't returned, the component is rendered.</span></span>
1. <span data-ttu-id="680d8-117">Chiamare [`OnParametersSet{Async}`](#after-parameters-are-set) ed eseguire il rendering del componente.</span><span class="sxs-lookup"><span data-stu-id="680d8-117">Call [`OnParametersSet{Async}`](#after-parameters-are-set) and render the component.</span></span> <span data-ttu-id="680d8-118">Se un oggetto <xref:System.Threading.Tasks.Task> viene restituito da `OnParametersSetAsync` , <xref:System.Threading.Tasks.Task> viene atteso e quindi viene eseguito il rendering del componente.</span><span class="sxs-lookup"><span data-stu-id="680d8-118">If a <xref:System.Threading.Tasks.Task> is returned from `OnParametersSetAsync`, the <xref:System.Threading.Tasks.Task> is awaited and then the component is rerendered.</span></span>

![Eventi del ciclo di vita dei componenti di un elemento::: NO-LOC (Razor)::: Component in::: NO-LOC (Blazer):::](lifecycle/_static/lifecycle1.png)

<span data-ttu-id="680d8-120">Elaborazione evento Document Object Model (DOM):</span><span class="sxs-lookup"><span data-stu-id="680d8-120">Document Object Model (DOM) event processing:</span></span>

1. <span data-ttu-id="680d8-121">Il gestore eventi viene eseguito.</span><span class="sxs-lookup"><span data-stu-id="680d8-121">The event handler is run.</span></span>
1. <span data-ttu-id="680d8-122">Se <xref:System.Threading.Tasks.Task> viene restituito un oggetto, <xref:System.Threading.Tasks.Task> viene atteso e quindi viene eseguito il rendering del componente.</span><span class="sxs-lookup"><span data-stu-id="680d8-122">If a <xref:System.Threading.Tasks.Task> is returned, the <xref:System.Threading.Tasks.Task> is awaited and then the component is rendered.</span></span> <span data-ttu-id="680d8-123">Se <xref:System.Threading.Tasks.Task> non viene restituito alcun oggetto, viene eseguito il rendering del componente.</span><span class="sxs-lookup"><span data-stu-id="680d8-123">If a <xref:System.Threading.Tasks.Task> isn't returned, the component is rendered.</span></span>

![Elaborazione di eventi Document Object Model (DOM)](lifecycle/_static/lifecycle2.png)

<span data-ttu-id="680d8-125">Ciclo di vita `Render` :</span><span class="sxs-lookup"><span data-stu-id="680d8-125">The `Render` lifecycle:</span></span>

1. <span data-ttu-id="680d8-126">Evitare ulteriori operazioni di rendering sul componente:</span><span class="sxs-lookup"><span data-stu-id="680d8-126">Avoid further rendering operations on the component:</span></span>
   * <span data-ttu-id="680d8-127">Dopo il primo rendering.</span><span class="sxs-lookup"><span data-stu-id="680d8-127">After the first render.</span></span>
   * <span data-ttu-id="680d8-128">Quando [`ShouldRender`](#suppress-ui-refreshing) è `false` .</span><span class="sxs-lookup"><span data-stu-id="680d8-128">When [`ShouldRender`](#suppress-ui-refreshing) is `false`.</span></span>
1. <span data-ttu-id="680d8-129">Compilare la diff dell'albero di rendering (differenza) ed eseguire il rendering del componente.</span><span class="sxs-lookup"><span data-stu-id="680d8-129">Build the render tree diff (difference) and render the component.</span></span>
1. <span data-ttu-id="680d8-130">Attendere il DOM da aggiornare.</span><span class="sxs-lookup"><span data-stu-id="680d8-130">Await the DOM to update.</span></span>
1. <span data-ttu-id="680d8-131">Chiamare [`OnAfterRender{Async}`](#after-component-render) .</span><span class="sxs-lookup"><span data-stu-id="680d8-131">Call [`OnAfterRender{Async}`](#after-component-render).</span></span>

![Ciclo di vita rendering](lifecycle/_static/lifecycle3.png)

<span data-ttu-id="680d8-133">Lo sviluppatore chiama per [`StateHasChanged`](#state-changes) generare un rendering.</span><span class="sxs-lookup"><span data-stu-id="680d8-133">Developer calls to [`StateHasChanged`](#state-changes) result in a render.</span></span> <span data-ttu-id="680d8-134">Per altre informazioni, vedere <xref:blazor/components/rendering>.</span><span class="sxs-lookup"><span data-stu-id="680d8-134">For more information, see <xref:blazor/components/rendering>.</span></span>

## <a name="lifecycle-methods"></a><span data-ttu-id="680d8-135">Metodi del ciclo di vita</span><span class="sxs-lookup"><span data-stu-id="680d8-135">Lifecycle methods</span></span>

### <a name="before-parameters-are-set"></a><span data-ttu-id="680d8-136">Prima dell'impostazione dei parametri</span><span class="sxs-lookup"><span data-stu-id="680d8-136">Before parameters are set</span></span>

<span data-ttu-id="680d8-137"><xref:Microsoft.AspNetCore.Components.ComponentBase.SetParametersAsync%2A> Imposta i parametri forniti dall'elemento padre del componente nell'albero di rendering o dai parametri di route.</span><span class="sxs-lookup"><span data-stu-id="680d8-137"><xref:Microsoft.AspNetCore.Components.ComponentBase.SetParametersAsync%2A> sets parameters supplied by the component's parent in the render tree or from route parameters.</span></span> <span data-ttu-id="680d8-138">Eseguendo l'override del metodo, il codice dello sviluppatore può interagire direttamente con i <xref:Microsoft.AspNetCore.Components.ParameterView> parametri di.</span><span class="sxs-lookup"><span data-stu-id="680d8-138">By overriding the method, developer code can interact directly with the <xref:Microsoft.AspNetCore.Components.ParameterView>'s parameters.</span></span>

<span data-ttu-id="680d8-139">Nell'esempio seguente, <xref:Microsoft.AspNetCore.Components.ParameterView.TryGetValue%2A?displayProperty=nameWithType> assegna il valore del `Param` parametro a se l' `value` analisi di un parametro di route per `Param` ha esito positivo.</span><span class="sxs-lookup"><span data-stu-id="680d8-139">In the following example, <xref:Microsoft.AspNetCore.Components.ParameterView.TryGetValue%2A?displayProperty=nameWithType> assigns the `Param` parameter's value to `value` if parsing a route parameter for `Param` is successful.</span></span> <span data-ttu-id="680d8-140">Quando `value` non è `null` , il valore viene visualizzato dal componente.</span><span class="sxs-lookup"><span data-stu-id="680d8-140">When `value` isn't `null`, the value is displayed by the component.</span></span>

<span data-ttu-id="680d8-141">Sebbene la corrispondenza tra i parametri di route non riscontri la [distinzione tra maiuscole](xref:blazor/fundamentals/routing#route-parameters)e minuscole, trova <xref:Microsoft.AspNetCore.Components.ParameterView.TryGetValue%2A> solo i nomi dei parametri con distinzione maiuscole</span><span class="sxs-lookup"><span data-stu-id="680d8-141">Although [route parameter matching is case insensitive](xref:blazor/fundamentals/routing#route-parameters), <xref:Microsoft.AspNetCore.Components.ParameterView.TryGetValue%2A> only matches case sensitive parameter names in the route template.</span></span> <span data-ttu-id="680d8-142">L'esempio seguente è obbligatorio per usare `/{Param?}` , non per `/{param?}` ottenere il valore.</span><span class="sxs-lookup"><span data-stu-id="680d8-142">The following example is required to use `/{Param?}`, not `/{param?}`, in order to get the value.</span></span> <span data-ttu-id="680d8-143">Se `/{param?}` viene usato in questo scenario, <xref:Microsoft.AspNetCore.Components.ParameterView.TryGetValue%2A> restituisce `false` e `message` non è impostato su una delle due stringhe.</span><span class="sxs-lookup"><span data-stu-id="680d8-143">If `/{param?}` is used in this scenario, <xref:Microsoft.AspNetCore.Components.ParameterView.TryGetValue%2A> returns `false` and `message` isn't set to either string.</span></span>

<span data-ttu-id="680d8-144">`Pages/SetParametersAsyncExample.razor`:</span><span class="sxs-lookup"><span data-stu-id="680d8-144">`Pages/SetParametersAsyncExample.razor`:</span></span>

```razor
@page "/setparametersasync-example/{Param?}"

<h1>SetParametersAsync Example</h1>

<p>@message</p>

@code {
    private string message;

    [Parameter]
    public string Param { get; set; }

    public override async Task SetParametersAsync(ParameterView parameters)
    {
        if (parameters.TryGetValue<string>(nameof(Param), out var value))
        {
            if (value is null)
            {
                message = "The value of 'Param' is null.";
            }
            else
            {
                message = $"The value of 'Param' is {value}.";
            }
        }

        await base.SetParametersAsync(parameters);
    }
}
```

<span data-ttu-id="680d8-145"><xref:Microsoft.AspNetCore.Components.ParameterView> contiene il set di valori di parametro per il componente ogni volta che <xref:Microsoft.AspNetCore.Components.ComponentBase.SetParametersAsync%2A> viene chiamato.</span><span class="sxs-lookup"><span data-stu-id="680d8-145"><xref:Microsoft.AspNetCore.Components.ParameterView> contains the set of parameter values for the component each time <xref:Microsoft.AspNetCore.Components.ComponentBase.SetParametersAsync%2A> is called.</span></span>

<span data-ttu-id="680d8-146">L'implementazione predefinita di <xref:Microsoft.AspNetCore.Components.ComponentBase.SetParametersAsync%2A> imposta il valore di ogni proprietà con l' [`[Parameter]`](xref:Microsoft.AspNetCore.Components.ParameterAttribute) attributo o con [`[CascadingParameter]`](xref:Microsoft.AspNetCore.Components.CascadingParameterAttribute) un valore corrispondente in <xref:Microsoft.AspNetCore.Components.ParameterView> .</span><span class="sxs-lookup"><span data-stu-id="680d8-146">The default implementation of <xref:Microsoft.AspNetCore.Components.ComponentBase.SetParametersAsync%2A> sets the value of each property with the [`[Parameter]`](xref:Microsoft.AspNetCore.Components.ParameterAttribute) or [`[CascadingParameter]`](xref:Microsoft.AspNetCore.Components.CascadingParameterAttribute) attribute that has a corresponding value in the <xref:Microsoft.AspNetCore.Components.ParameterView>.</span></span> <span data-ttu-id="680d8-147">I parametri che non hanno un valore corrispondente in <xref:Microsoft.AspNetCore.Components.ParameterView> rimangono invariati.</span><span class="sxs-lookup"><span data-stu-id="680d8-147">Parameters that don't have a corresponding value in <xref:Microsoft.AspNetCore.Components.ParameterView> are left unchanged.</span></span>

<span data-ttu-id="680d8-148">Se [`base.SetParametersAsync`](xref:Microsoft.AspNetCore.Components.ComponentBase.SetParametersAsync%2A) non viene richiamato, il codice personalizzato può interpretare il valore dei parametri in ingresso in qualsiasi modo necessario.</span><span class="sxs-lookup"><span data-stu-id="680d8-148">If [`base.SetParametersAsync`](xref:Microsoft.AspNetCore.Components.ComponentBase.SetParametersAsync%2A) isn't invoked, the custom code can interpret the incoming parameters value in any way required.</span></span> <span data-ttu-id="680d8-149">Ad esempio, non è necessario assegnare i parametri in ingresso alle proprietà della classe.</span><span class="sxs-lookup"><span data-stu-id="680d8-149">For example, there's no requirement to assign the incoming parameters to the properties on the class.</span></span>

<span data-ttu-id="680d8-150">Se sono configurati gestori di eventi, rimuoverli a disposizione.</span><span class="sxs-lookup"><span data-stu-id="680d8-150">If any event handlers are set up, unhook them on disposal.</span></span> <span data-ttu-id="680d8-151">Per ulteriori informazioni, vedere la sezione relativa all' [eliminazione dei componenti con `IDisposable` ](#component-disposal-with-idisposable) .</span><span class="sxs-lookup"><span data-stu-id="680d8-151">For more information, see the [Component disposal with `IDisposable`](#component-disposal-with-idisposable) section.</span></span>

### <a name="component-initialization-methods"></a><span data-ttu-id="680d8-152">Metodi di inizializzazione componenti</span><span class="sxs-lookup"><span data-stu-id="680d8-152">Component initialization methods</span></span>

<span data-ttu-id="680d8-153"><xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> e <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitialized%2A> vengono richiamati quando il componente viene inizializzato dopo aver ricevuto i parametri iniziali dal componente padre in <xref:Microsoft.AspNetCore.Components.ComponentBase.SetParametersAsync%2A> .</span><span class="sxs-lookup"><span data-stu-id="680d8-153"><xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> and <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitialized%2A> are invoked when the component is initialized after having received its initial parameters from its parent component in <xref:Microsoft.AspNetCore.Components.ComponentBase.SetParametersAsync%2A>.</span></span> 

<span data-ttu-id="680d8-154">Usare <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> quando il componente esegue un'operazione asincrona e deve essere aggiornato al termine dell'operazione.</span><span class="sxs-lookup"><span data-stu-id="680d8-154">Use <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> when the component performs an asynchronous operation and should refresh when the operation is completed.</span></span>

<span data-ttu-id="680d8-155">Per un'operazione sincrona, eseguire l'override di <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitialized%2A> :</span><span class="sxs-lookup"><span data-stu-id="680d8-155">For a synchronous operation, override <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitialized%2A>:</span></span>

```csharp
protected override void OnInitialized()
{
    ...
}
```

<span data-ttu-id="680d8-156">Per eseguire un'operazione asincrona, eseguire l'override <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> e utilizzare l' [`await`](/dotnet/csharp/language-reference/operators/await) operatore sull'operazione:</span><span class="sxs-lookup"><span data-stu-id="680d8-156">To perform an asynchronous operation, override <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> and use the [`await`](/dotnet/csharp/language-reference/operators/await) operator on the operation:</span></span>

```csharp
protected override async Task OnInitializedAsync()
{
    await ...
}
```

<span data-ttu-id="680d8-157">Blazor Server app che [preeseguono](xref:blazor/fundamentals/additional-scenarios#render-mode) <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> *due volte* la chiamata al contenuto:</span><span class="sxs-lookup"><span data-stu-id="680d8-157">Blazor Server apps that [prerender their content](xref:blazor/fundamentals/additional-scenarios#render-mode) call <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> *twice*:</span></span>

* <span data-ttu-id="680d8-158">Una volta quando il componente viene inizialmente sottoposto a rendering statico come parte della pagina.</span><span class="sxs-lookup"><span data-stu-id="680d8-158">Once when the component is initially rendered statically as part of the page.</span></span>
* <span data-ttu-id="680d8-159">Una seconda volta quando il browser stabilisce una connessione al server.</span><span class="sxs-lookup"><span data-stu-id="680d8-159">A second time when the browser establishes a connection back to the server.</span></span>

<span data-ttu-id="680d8-160">Per impedire l'esecuzione di codice dello sviluppatore per <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> due volte, vedere la sezione [riconnessione con stato dopo il rendering](#stateful-reconnection-after-prerendering) .</span><span class="sxs-lookup"><span data-stu-id="680d8-160">To prevent developer code in <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> from running twice, see the [Stateful reconnection after prerendering](#stateful-reconnection-after-prerendering) section.</span></span>

<span data-ttu-id="680d8-161">Mentre un' Blazor Server app è prerendering, alcune azioni, ad esempio la chiamata a JavaScript, non sono possibili perché non è stata stabilita una connessione con il browser.</span><span class="sxs-lookup"><span data-stu-id="680d8-161">While a Blazor Server app is prerendering, certain actions, such as calling into JavaScript, aren't possible because a connection with the browser hasn't been established.</span></span> <span data-ttu-id="680d8-162">I componenti potrebbero dover eseguire il rendering in modo diverso quando ne viene eseguito il rendering.</span><span class="sxs-lookup"><span data-stu-id="680d8-162">Components may need to render differently when prerendered.</span></span> <span data-ttu-id="680d8-163">Per altre informazioni, vedere la sezione [rilevare quando l'app è prerendering](#detect-when-the-app-is-prerendering) .</span><span class="sxs-lookup"><span data-stu-id="680d8-163">For more information, see the [Detect when the app is prerendering](#detect-when-the-app-is-prerendering) section.</span></span>

<span data-ttu-id="680d8-164">Se sono configurati gestori di eventi, rimuoverli a disposizione.</span><span class="sxs-lookup"><span data-stu-id="680d8-164">If any event handlers are set up, unhook them on disposal.</span></span> <span data-ttu-id="680d8-165">Per ulteriori informazioni, vedere la sezione relativa all' [eliminazione dei componenti con `IDisposable` ](#component-disposal-with-idisposable) .</span><span class="sxs-lookup"><span data-stu-id="680d8-165">For more information, see the [Component disposal with `IDisposable`](#component-disposal-with-idisposable) section.</span></span>

### <a name="after-parameters-are-set"></a><span data-ttu-id="680d8-166">Parametri after impostati</span><span class="sxs-lookup"><span data-stu-id="680d8-166">After parameters are set</span></span>

<span data-ttu-id="680d8-167"><xref:Microsoft.AspNetCore.Components.ComponentBase.OnParametersSetAsync%2A> o <xref:Microsoft.AspNetCore.Components.ComponentBase.OnParametersSet%2A> sono chiamati:</span><span class="sxs-lookup"><span data-stu-id="680d8-167"><xref:Microsoft.AspNetCore.Components.ComponentBase.OnParametersSetAsync%2A> or <xref:Microsoft.AspNetCore.Components.ComponentBase.OnParametersSet%2A> are called:</span></span>

* <span data-ttu-id="680d8-168">Dopo che il componente è stato inizializzato in <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitialized%2A> o <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> .</span><span class="sxs-lookup"><span data-stu-id="680d8-168">After the component is initialized in <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitialized%2A> or <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A>.</span></span>
* <span data-ttu-id="680d8-169">Quando il componente padre esegue nuovamente il rendering e fornisce:</span><span class="sxs-lookup"><span data-stu-id="680d8-169">When the parent component re-renders and supplies:</span></span>
  * <span data-ttu-id="680d8-170">Solo i tipi non modificabili primitivi noti di cui è stato modificato almeno un parametro.</span><span class="sxs-lookup"><span data-stu-id="680d8-170">Only known primitive immutable types of which at least one parameter has changed.</span></span>
  * <span data-ttu-id="680d8-171">Tutti i parametri tipizzati complessi.</span><span class="sxs-lookup"><span data-stu-id="680d8-171">Any complex-typed parameters.</span></span> <span data-ttu-id="680d8-172">Il Framework non è in grado di stabilire se i valori di un parametro tipizzato complesso sono stati modificati internamente, quindi considera il set di parametri come modificato.</span><span class="sxs-lookup"><span data-stu-id="680d8-172">The framework can't know whether the values of a complex-typed parameter have mutated internally, so it treats the parameter set as changed.</span></span>

```csharp
protected override async Task OnParametersSetAsync()
{
    await ...
}
```

> [!NOTE]
> <span data-ttu-id="680d8-173">Il lavoro asincrono quando si applicano parametri e valori di proprietà deve verificarsi durante l'evento del ciclo di vita <xref:Microsoft.AspNetCore.Components.ComponentBase.OnParametersSetAsync%2A> .</span><span class="sxs-lookup"><span data-stu-id="680d8-173">Asynchronous work when applying parameters and property values must occur during the <xref:Microsoft.AspNetCore.Components.ComponentBase.OnParametersSetAsync%2A> lifecycle event.</span></span>

```csharp
protected override void OnParametersSet()
{
    ...
}
```

<span data-ttu-id="680d8-174">Se sono configurati gestori di eventi, rimuoverli a disposizione.</span><span class="sxs-lookup"><span data-stu-id="680d8-174">If any event handlers are set up, unhook them on disposal.</span></span> <span data-ttu-id="680d8-175">Per ulteriori informazioni, vedere la sezione relativa all' [eliminazione dei componenti con `IDisposable` ](#component-disposal-with-idisposable) .</span><span class="sxs-lookup"><span data-stu-id="680d8-175">For more information, see the [Component disposal with `IDisposable`](#component-disposal-with-idisposable) section.</span></span>

### <a name="after-component-render"></a><span data-ttu-id="680d8-176">Rendering del componente dopo</span><span class="sxs-lookup"><span data-stu-id="680d8-176">After component render</span></span>

<span data-ttu-id="680d8-177"><xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A> e <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRender%2A> vengono chiamati dopo che un componente ha terminato il rendering.</span><span class="sxs-lookup"><span data-stu-id="680d8-177"><xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A> and <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRender%2A> are called after a component has finished rendering.</span></span> <span data-ttu-id="680d8-178">I riferimenti a elementi e componenti vengono popolati a questo punto.</span><span class="sxs-lookup"><span data-stu-id="680d8-178">Element and component references are populated at this point.</span></span> <span data-ttu-id="680d8-179">Usare questa fase per eseguire passaggi di inizializzazione aggiuntivi usando il contenuto sottoposto a rendering, ad esempio l'attivazione di librerie JavaScript di terze parti che operano sugli elementi DOM sottoposti a rendering.</span><span class="sxs-lookup"><span data-stu-id="680d8-179">Use this stage to perform additional initialization steps using the rendered content, such as activating third-party JavaScript libraries that operate on the rendered DOM elements.</span></span>

<span data-ttu-id="680d8-180">Il `firstRender` parametro per <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A> e <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRender%2A> :</span><span class="sxs-lookup"><span data-stu-id="680d8-180">The `firstRender` parameter for <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A> and <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRender%2A>:</span></span>

* <span data-ttu-id="680d8-181">Viene impostato `true` sulla prima volta che viene eseguito il rendering dell'istanza del componente.</span><span class="sxs-lookup"><span data-stu-id="680d8-181">Is set to `true` the first time that the component instance is rendered.</span></span>
* <span data-ttu-id="680d8-182">Può essere usato per garantire che il lavoro di inizializzazione venga eseguito una sola volta.</span><span class="sxs-lookup"><span data-stu-id="680d8-182">Can be used to ensure that initialization work is only performed once.</span></span>

```csharp
protected override async Task OnAfterRenderAsync(bool firstRender)
{
    if (firstRender)
    {
        await ...
    }
}
```

> [!NOTE]
> <span data-ttu-id="680d8-183">Il lavoro asincrono immediatamente dopo il rendering deve verificarsi durante l'evento del ciclo di vita <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A> .</span><span class="sxs-lookup"><span data-stu-id="680d8-183">Asynchronous work immediately after rendering must occur during the <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A> lifecycle event.</span></span>
>
> <span data-ttu-id="680d8-184">Anche se si restituisce un oggetto <xref:System.Threading.Tasks.Task> da <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A> , il Framework non pianifica un ulteriore ciclo di rendering per il componente al termine dell'attività.</span><span class="sxs-lookup"><span data-stu-id="680d8-184">Even if you return a <xref:System.Threading.Tasks.Task> from <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A>, the framework doesn't schedule a further render cycle for your component once that task completes.</span></span> <span data-ttu-id="680d8-185">In questo modo è possibile evitare un ciclo di rendering infinito.</span><span class="sxs-lookup"><span data-stu-id="680d8-185">This is to avoid an infinite render loop.</span></span> <span data-ttu-id="680d8-186">È diverso dagli altri metodi del ciclo di vita, che pianificano un ulteriore ciclo di rendering dopo il completamento dell'attività restituita.</span><span class="sxs-lookup"><span data-stu-id="680d8-186">It's different from the other lifecycle methods, which schedule a further render cycle once the returned task completes.</span></span>

```csharp
protected override void OnAfterRender(bool firstRender)
{
    if (firstRender)
    {
        ...
    }
}
```

<span data-ttu-id="680d8-187"><xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRender%2A> e <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A> *non vengono chiamati durante il processo di prerendering sul server*.</span><span class="sxs-lookup"><span data-stu-id="680d8-187"><xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRender%2A> and <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A> *aren't called during the prerendering process on the server*.</span></span> <span data-ttu-id="680d8-188">I metodi vengono chiamati quando il componente viene sottoposto a rendering interattivo dopo il completamento del rendering.</span><span class="sxs-lookup"><span data-stu-id="680d8-188">The methods are called when the component is rendered interactively after prerendering is finished.</span></span> <span data-ttu-id="680d8-189">Quando viene eseguito il rendering dell'app:</span><span class="sxs-lookup"><span data-stu-id="680d8-189">When the app prerenders:</span></span>

1. <span data-ttu-id="680d8-190">Il componente viene eseguito sul server per produrre un markup HTML statico nella risposta HTTP.</span><span class="sxs-lookup"><span data-stu-id="680d8-190">The component executes on the server to produce some static HTML markup in the HTTP response.</span></span> <span data-ttu-id="680d8-191">Durante questa fase, <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRender%2A> <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A> non viene chiamato.</span><span class="sxs-lookup"><span data-stu-id="680d8-191">During this phase, <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRender%2A> and <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A> aren't called.</span></span>
1. <span data-ttu-id="680d8-192">Quando `blazor.server.js` o `blazor.webassembly.js` si avvia nel browser, il componente viene riavviato in modalità di rendering interattiva.</span><span class="sxs-lookup"><span data-stu-id="680d8-192">When `blazor.server.js` or `blazor.webassembly.js` start up in the browser, the component is restarted in an interactive rendering mode.</span></span> <span data-ttu-id="680d8-193">Dopo che un componente viene riavviato <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRender%2A> e <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A> **viene** chiamato perché l'app non è più all'interno della fase di prerendering.</span><span class="sxs-lookup"><span data-stu-id="680d8-193">After a component is restarted, <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRender%2A> and <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A> **are** called because the app isn't inside the prerendering phase any longer.</span></span>

<span data-ttu-id="680d8-194">Se sono configurati gestori di eventi, rimuoverli a disposizione.</span><span class="sxs-lookup"><span data-stu-id="680d8-194">If any event handlers are set up, unhook them on disposal.</span></span> <span data-ttu-id="680d8-195">Per ulteriori informazioni, vedere la sezione relativa all' [eliminazione dei componenti con `IDisposable` ](#component-disposal-with-idisposable) .</span><span class="sxs-lookup"><span data-stu-id="680d8-195">For more information, see the [Component disposal with `IDisposable`](#component-disposal-with-idisposable) section.</span></span>

### <a name="suppress-ui-refreshing"></a><span data-ttu-id="680d8-196">Evita aggiornamento dell'interfaccia utente</span><span class="sxs-lookup"><span data-stu-id="680d8-196">Suppress UI refreshing</span></span>

<span data-ttu-id="680d8-197">Eseguire l'override <xref:Microsoft.AspNetCore.Components.ComponentBase.ShouldRender%2A> per impedire l'aggiornamento dell'interfaccia utente.</span><span class="sxs-lookup"><span data-stu-id="680d8-197">Override <xref:Microsoft.AspNetCore.Components.ComponentBase.ShouldRender%2A> to suppress UI refreshing.</span></span> <span data-ttu-id="680d8-198">Se l'implementazione restituisce `true` , l'interfaccia utente viene aggiornata:</span><span class="sxs-lookup"><span data-stu-id="680d8-198">If the implementation returns `true`, the UI is refreshed:</span></span>

```csharp
protected override bool ShouldRender()
{
    var renderUI = true;

    return renderUI;
}
```

<span data-ttu-id="680d8-199"><xref:Microsoft.AspNetCore.Components.ComponentBase.ShouldRender%2A> viene chiamato ogni volta che il componente viene sottoposto a rendering.</span><span class="sxs-lookup"><span data-stu-id="680d8-199"><xref:Microsoft.AspNetCore.Components.ComponentBase.ShouldRender%2A> is called each time the component is rendered.</span></span>

<span data-ttu-id="680d8-200">Anche se <xref:Microsoft.AspNetCore.Components.ComponentBase.ShouldRender%2A> viene sottoposto a override, il componente viene sempre sottoposto a rendering iniziale.</span><span class="sxs-lookup"><span data-stu-id="680d8-200">Even if <xref:Microsoft.AspNetCore.Components.ComponentBase.ShouldRender%2A> is overridden, the component is always initially rendered.</span></span>

<span data-ttu-id="680d8-201">Per altre informazioni, vedere <xref:blazor/webassembly-performance-best-practices#avoid-unnecessary-rendering-of-component-subtrees>.</span><span class="sxs-lookup"><span data-stu-id="680d8-201">For more information, see <xref:blazor/webassembly-performance-best-practices#avoid-unnecessary-rendering-of-component-subtrees>.</span></span>

## <a name="state-changes"></a><span data-ttu-id="680d8-202">Modifiche stato</span><span class="sxs-lookup"><span data-stu-id="680d8-202">State changes</span></span>

<span data-ttu-id="680d8-203"><xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> notifica al componente che lo stato è stato modificato.</span><span class="sxs-lookup"><span data-stu-id="680d8-203"><xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> notifies the component that its state has changed.</span></span> <span data-ttu-id="680d8-204">Se applicabile, <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> la chiamata a comporta il rendering del componente.</span><span class="sxs-lookup"><span data-stu-id="680d8-204">When applicable, calling <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> causes the component to be rerendered.</span></span>

<span data-ttu-id="680d8-205"><xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> viene chiamato automaticamente per i <xref:Microsoft.AspNetCore.Components.EventCallback> metodi.</span><span class="sxs-lookup"><span data-stu-id="680d8-205"><xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> is called automatically for <xref:Microsoft.AspNetCore.Components.EventCallback> methods.</span></span> <span data-ttu-id="680d8-206">Per altre informazioni, vedere <xref:blazor/components/event-handling#eventcallback>.</span><span class="sxs-lookup"><span data-stu-id="680d8-206">For more information, see <xref:blazor/components/event-handling#eventcallback>.</span></span>

<span data-ttu-id="680d8-207">Per altre informazioni, vedere <xref:blazor/components/rendering>.</span><span class="sxs-lookup"><span data-stu-id="680d8-207">For more information, see <xref:blazor/components/rendering>.</span></span>

## <a name="handle-incomplete-async-actions-at-render"></a><span data-ttu-id="680d8-208">Gestisci azioni asincrone incomplete durante il rendering</span><span class="sxs-lookup"><span data-stu-id="680d8-208">Handle incomplete async actions at render</span></span>

<span data-ttu-id="680d8-209">Le azioni asincrone eseguite negli eventi del ciclo di vita potrebbero non essere state completate prima del rendering del componente.</span><span class="sxs-lookup"><span data-stu-id="680d8-209">Asynchronous actions performed in lifecycle events might not have completed before the component is rendered.</span></span> <span data-ttu-id="680d8-210">Gli oggetti possono essere `null` o compilati in modo non completo con i dati mentre è in esecuzione il metodo del ciclo di vita.</span><span class="sxs-lookup"><span data-stu-id="680d8-210">Objects might be `null` or incompletely populated with data while the lifecycle method is executing.</span></span> <span data-ttu-id="680d8-211">Fornire la logica di rendering per confermare che gli oggetti vengono inizializzati.</span><span class="sxs-lookup"><span data-stu-id="680d8-211">Provide rendering logic to confirm that objects are initialized.</span></span> <span data-ttu-id="680d8-212">Esegue il rendering degli elementi dell'interfaccia utente segnaposto (ad esempio, un messaggio di caricamento) mentre gli oggetti sono `null` .</span><span class="sxs-lookup"><span data-stu-id="680d8-212">Render placeholder UI elements (for example, a loading message) while objects are `null`.</span></span>

<span data-ttu-id="680d8-213">Nel `FetchData` componente dei Blazor modelli <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> viene eseguito l'override di per ricevere i dati di previsione () in modo asincrono `forecasts` .</span><span class="sxs-lookup"><span data-stu-id="680d8-213">In the `FetchData` component of the Blazor templates, <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> is overridden to asynchronously receive forecast data (`forecasts`).</span></span> <span data-ttu-id="680d8-214">Quando `forecasts` è `null` , all'utente viene visualizzato un messaggio di caricamento.</span><span class="sxs-lookup"><span data-stu-id="680d8-214">When `forecasts` is `null`, a loading message is displayed to the user.</span></span> <span data-ttu-id="680d8-215">Quando l'oggetto `Task` restituito da <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> viene completato, viene eseguito il rendering del componente con lo stato aggiornato.</span><span class="sxs-lookup"><span data-stu-id="680d8-215">After the `Task` returned by <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> completes, the component is rerendered with the updated state.</span></span>

<span data-ttu-id="680d8-216">`Pages/FetchData.razor` nel Blazor Server modello:</span><span class="sxs-lookup"><span data-stu-id="680d8-216">`Pages/FetchData.razor` in the Blazor Server template:</span></span>

[!code-razor[](lifecycle/samples_snapshot/FetchData.razor?highlight=9,21,25)]

## <a name="handle-errors"></a><span data-ttu-id="680d8-217">Gestire gli errori</span><span class="sxs-lookup"><span data-stu-id="680d8-217">Handle errors</span></span>

<span data-ttu-id="680d8-218">Per informazioni sulla gestione degli errori durante l'esecuzione del metodo del ciclo di vita, vedere <xref:blazor/fundamentals/handle-errors#lifecycle-methods> .</span><span class="sxs-lookup"><span data-stu-id="680d8-218">For information on handling errors during lifecycle method execution, see <xref:blazor/fundamentals/handle-errors#lifecycle-methods>.</span></span>

## <a name="stateful-reconnection-after-prerendering"></a><span data-ttu-id="680d8-219">Riconnessione con stato dopo il rendering preliminare</span><span class="sxs-lookup"><span data-stu-id="680d8-219">Stateful reconnection after prerendering</span></span>

<span data-ttu-id="680d8-220">In un' Blazor Server app quando <xref:Microsoft.AspNetCore.Mvc.TagHelpers.ComponentTagHelper.RenderMode> è <xref:Microsoft.AspNetCore.Mvc.Rendering.RenderMode.ServerPrerendered> , il componente viene inizialmente sottoposto a rendering statico come parte della pagina.</span><span class="sxs-lookup"><span data-stu-id="680d8-220">In a Blazor Server app when <xref:Microsoft.AspNetCore.Mvc.TagHelpers.ComponentTagHelper.RenderMode> is <xref:Microsoft.AspNetCore.Mvc.Rendering.RenderMode.ServerPrerendered>, the component is initially rendered statically as part of the page.</span></span> <span data-ttu-id="680d8-221">Quando il browser stabilisce una connessione al server, viene *nuovamente* eseguito il rendering del componente e il componente è ora interattivo.</span><span class="sxs-lookup"><span data-stu-id="680d8-221">Once the browser establishes a connection back to the server, the component is rendered *again*, and the component is now interactive.</span></span> <span data-ttu-id="680d8-222">Se [`OnInitialized{Async}`](#component-initialization-methods) è presente il metodo del ciclo di vita per l'inizializzazione del componente, il metodo viene eseguito *due volte*:</span><span class="sxs-lookup"><span data-stu-id="680d8-222">If the [`OnInitialized{Async}`](#component-initialization-methods) lifecycle method for initializing the component is present, the method is executed *twice*:</span></span>

* <span data-ttu-id="680d8-223">Quando il componente viene preeseguito in modo statico.</span><span class="sxs-lookup"><span data-stu-id="680d8-223">When the component is prerendered statically.</span></span>
* <span data-ttu-id="680d8-224">Una volta stabilita la connessione al server.</span><span class="sxs-lookup"><span data-stu-id="680d8-224">After the server connection has been established.</span></span>

<span data-ttu-id="680d8-225">Ciò può comportare una modifica evidente nei dati visualizzati nell'interfaccia utente quando il componente viene sottoposto a rendering.</span><span class="sxs-lookup"><span data-stu-id="680d8-225">This can result in a noticeable change in the data displayed in the UI when the component is finally rendered.</span></span>

<span data-ttu-id="680d8-226">Per evitare lo scenario di doppio rendering in un' Blazor Server app:</span><span class="sxs-lookup"><span data-stu-id="680d8-226">To avoid the double-rendering scenario in a Blazor Server app:</span></span>

* <span data-ttu-id="680d8-227">Passare un identificatore che può essere usato per memorizzare nella cache lo stato durante il prerendering e recuperare lo stato dopo il riavvio dell'app.</span><span class="sxs-lookup"><span data-stu-id="680d8-227">Pass in an identifier that can be used to cache the state during prerendering and to retrieve the state after the app restarts.</span></span>
* <span data-ttu-id="680d8-228">Utilizzare l'identificatore durante il prerendering per salvare lo stato del componente.</span><span class="sxs-lookup"><span data-stu-id="680d8-228">Use the identifier during prerendering to save component state.</span></span>
* <span data-ttu-id="680d8-229">Utilizzare l'identificatore dopo il prerendering per recuperare lo stato memorizzato nella cache.</span><span class="sxs-lookup"><span data-stu-id="680d8-229">Use the identifier after prerendering to retrieve the cached state.</span></span>

<span data-ttu-id="680d8-230">Il codice seguente illustra un aggiornamento `WeatherForecastService` in un'app basata su modello Blazor Server che evita il doppio rendering:</span><span class="sxs-lookup"><span data-stu-id="680d8-230">The following code demonstrates an updated `WeatherForecastService` in a template-based Blazor Server app that avoids the double rendering:</span></span>

```csharp
public class WeatherForecastService
{
    private static readonly string[] summaries = new[]
    {
        "Freezing", "Bracing", "Chilly", "Cool", "Mild",
        "Warm", "Balmy", "Hot", "Sweltering", "Scorching"
    };
    
    public WeatherForecastService(IMemoryCache memoryCache)
    {
        MemoryCache = memoryCache;
    }
    
    public IMemoryCache MemoryCache { get; }

    public Task<WeatherForecast[]> GetForecastAsync(DateTime startDate)
    {
        return MemoryCache.GetOrCreateAsync(startDate, async e =>
        {
            e.SetOptions(new MemoryCacheEntryOptions
            {
                AbsoluteExpirationRelativeToNow = 
                    TimeSpan.FromSeconds(30)
            });

            var rng = new Random();

            await Task.Delay(TimeSpan.FromSeconds(10));

            return Enumerable.Range(1, 5).Select(index => new WeatherForecast
            {
                Date = startDate.AddDays(index),
                TemperatureC = rng.Next(-20, 55),
                Summary = summaries[rng.Next(summaries.Length)]
            }).ToArray();
        });
    }
}
```

<span data-ttu-id="680d8-231">Per ulteriori informazioni su <xref:Microsoft.AspNetCore.Mvc.TagHelpers.ComponentTagHelper.RenderMode> , vedere <xref:blazor/fundamentals/additional-scenarios#render-mode> .</span><span class="sxs-lookup"><span data-stu-id="680d8-231">For more information on the <xref:Microsoft.AspNetCore.Mvc.TagHelpers.ComponentTagHelper.RenderMode>, see <xref:blazor/fundamentals/additional-scenarios#render-mode>.</span></span>

## <a name="detect-when-the-app-is-prerendering"></a><span data-ttu-id="680d8-232">Rilevare il momento in cui viene eseguito il prerendering dell'app</span><span class="sxs-lookup"><span data-stu-id="680d8-232">Detect when the app is prerendering</span></span>

[!INCLUDE[](~/blazor/includes/prerendering.md)]

## <a name="component-disposal-with-idisposable"></a><span data-ttu-id="680d8-233">Eliminazione di componenti con `IDisposable`</span><span class="sxs-lookup"><span data-stu-id="680d8-233">Component disposal with `IDisposable`</span></span>

<span data-ttu-id="680d8-234">Se un componente implementa <xref:System.IDisposable> , il Framework chiama il [metodo di eliminazione](/dotnet/standard/garbage-collection/implementing-dispose) quando il componente viene rimosso dall'interfaccia utente, in cui è possibile rilasciare le risorse non gestite.</span><span class="sxs-lookup"><span data-stu-id="680d8-234">If a component implements <xref:System.IDisposable>, the framework calls the [disposal method](/dotnet/standard/garbage-collection/implementing-dispose) when the component is removed from the UI, where unmanaged resources can be released.</span></span> <span data-ttu-id="680d8-235">L'eliminazione può essere eseguita in qualsiasi momento, incluso durante l' [inizializzazione dei componenti](#component-initialization-methods).</span><span class="sxs-lookup"><span data-stu-id="680d8-235">Disposal can occur at any time, including during [component initialization](#component-initialization-methods).</span></span> <span data-ttu-id="680d8-236">Il componente seguente implementa <xref:System.IDisposable> con la [`@implements`](xref:mvc/views/razor#implements) Razor direttiva:</span><span class="sxs-lookup"><span data-stu-id="680d8-236">The following component implements <xref:System.IDisposable> with the [`@implements`](xref:mvc/views/razor#implements) Razor directive:</span></span>

```razor
@using System
@implements IDisposable

...

@code {
    public void Dispose()
    {
        ...
    }
}
```

<span data-ttu-id="680d8-237">Per le attività di eliminazione asincrona, usare `DisposeAsync` anziché `Dispose` nell'esempio precedente:</span><span class="sxs-lookup"><span data-stu-id="680d8-237">For asynchronous disposal tasks, use `DisposeAsync` instead of `Dispose` in the preceding example:</span></span>

```csharp
public async ValueTask DisposeAsync()
{
    ...
}
```

> [!NOTE]
> <span data-ttu-id="680d8-238">La chiamata <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> di in `Dispose` non è supportata.</span><span class="sxs-lookup"><span data-stu-id="680d8-238">Calling <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> in `Dispose` isn't supported.</span></span> <span data-ttu-id="680d8-239"><xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> potrebbe essere richiamato come parte del propagazione del renderer, quindi la richiesta di aggiornamenti dell'interfaccia utente in quel momento non è supportata.</span><span class="sxs-lookup"><span data-stu-id="680d8-239"><xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> might be invoked as part of tearing down the renderer, so requesting UI updates at that point isn't supported.</span></span>

<span data-ttu-id="680d8-240">Annulla la sottoscrizione di gestori eventi da eventi .NET.</span><span class="sxs-lookup"><span data-stu-id="680d8-240">Unsubscribe event handlers from .NET events.</span></span> <span data-ttu-id="680d8-241">Negli esempi di [ Blazor form](xref:blazor/forms-validation) seguenti viene illustrato come scollegare un gestore eventi nel `Dispose` Metodo:</span><span class="sxs-lookup"><span data-stu-id="680d8-241">The following [Blazor form](xref:blazor/forms-validation) examples show how to unhook an event handler in the `Dispose` method:</span></span>

* <span data-ttu-id="680d8-242">Approccio basato su campo privato e lambda</span><span class="sxs-lookup"><span data-stu-id="680d8-242">Private field and lambda approach</span></span>

  [!code-razor[](lifecycle/samples_snapshot/event-handler-disposal-1.razor?highlight=23,28)]

* <span data-ttu-id="680d8-243">Approccio metodo privato</span><span class="sxs-lookup"><span data-stu-id="680d8-243">Private method approach</span></span>

  [!code-razor[](lifecycle/samples_snapshot/event-handler-disposal-2.razor?highlight=16,26)]
  
<span data-ttu-id="680d8-244">Per ulteriori informazioni, vedere la pagina relativa alla [pulizia di risorse non gestite](/dotnet/standard/garbage-collection/unmanaged) e agli argomenti che lo seguono sull'implementazione dei `Dispose` `DisposeAsync` metodi e.</span><span class="sxs-lookup"><span data-stu-id="680d8-244">For more information, see [Cleaning up unmanaged resources](/dotnet/standard/garbage-collection/unmanaged) and the topics that follow it on implementing the `Dispose` and `DisposeAsync` methods.</span></span>

## <a name="cancelable-background-work"></a><span data-ttu-id="680d8-245">Lavoro in background annullabile</span><span class="sxs-lookup"><span data-stu-id="680d8-245">Cancelable background work</span></span>

<span data-ttu-id="680d8-246">I componenti eseguono spesso attività in background con esecuzione prolungata, ad esempio la creazione di chiamate di rete ( <xref:System.Net.Http.HttpClient> ) e l'interazione con i database.</span><span class="sxs-lookup"><span data-stu-id="680d8-246">Components often perform long-running background work, such as making network calls (<xref:System.Net.Http.HttpClient>) and interacting with databases.</span></span> <span data-ttu-id="680d8-247">È consigliabile arrestare il lavoro in background per conservare le risorse di sistema in diverse situazioni.</span><span class="sxs-lookup"><span data-stu-id="680d8-247">It's desirable to stop the background work to conserve system resources in several situations.</span></span> <span data-ttu-id="680d8-248">Ad esempio, le operazioni asincrone in background non vengono arrestate automaticamente quando un utente si sposta da un componente.</span><span class="sxs-lookup"><span data-stu-id="680d8-248">For example, background asynchronous operations don't automatically stop when a user navigates away from a component.</span></span>

<span data-ttu-id="680d8-249">Altri motivi per cui gli elementi di lavoro in background potrebbero richiedere l'annullamento includono:</span><span class="sxs-lookup"><span data-stu-id="680d8-249">Other reasons why background work items might require cancellation include:</span></span>

* <span data-ttu-id="680d8-250">Un'attività in background in esecuzione è stata avviata con i parametri di elaborazione o i dati di input difettosi.</span><span class="sxs-lookup"><span data-stu-id="680d8-250">An executing background task was started with faulty input data or processing parameters.</span></span>
* <span data-ttu-id="680d8-251">Il set corrente di elementi di lavoro in background in esecuzione deve essere sostituito con un nuovo set di elementi di lavoro.</span><span class="sxs-lookup"><span data-stu-id="680d8-251">The current set of executing background work items must be replaced with a new set of work items.</span></span>
* <span data-ttu-id="680d8-252">È necessario modificare la priorità delle attività attualmente in esecuzione.</span><span class="sxs-lookup"><span data-stu-id="680d8-252">The priority of currently executing tasks must be changed.</span></span>
* <span data-ttu-id="680d8-253">L'app deve essere arrestata per poterla ridistribuire nel server.</span><span class="sxs-lookup"><span data-stu-id="680d8-253">The app has to be shut down in order to redeploy it to the server.</span></span>
* <span data-ttu-id="680d8-254">Le risorse del server diventano limitate, rendendo necessaria la ripianificazione degli elementi di lavoro in background.</span><span class="sxs-lookup"><span data-stu-id="680d8-254">Server resources become limited, necessitating the rescheduling of background work items.</span></span>

<span data-ttu-id="680d8-255">Per implementare un modello di lavoro in background annullabile in un componente:</span><span class="sxs-lookup"><span data-stu-id="680d8-255">To implement a cancelable background work pattern in a component:</span></span>

* <span data-ttu-id="680d8-256">Utilizzare <xref:System.Threading.CancellationTokenSource> e <xref:System.Threading.CancellationToken> .</span><span class="sxs-lookup"><span data-stu-id="680d8-256">Use a <xref:System.Threading.CancellationTokenSource> and <xref:System.Threading.CancellationToken>.</span></span>
* <span data-ttu-id="680d8-257">In caso di [eliminazione del componente](#component-disposal-with-idisposable) e in qualsiasi momento si desideri annullare l'annullamento del token, chiamare [`CancellationTokenSource.Cancel`](xref:System.Threading.CancellationTokenSource.Cancel%2A) per segnalare che le attività in background devono essere annullate.</span><span class="sxs-lookup"><span data-stu-id="680d8-257">On [disposal of the component](#component-disposal-with-idisposable) and at any point cancellation is desired by manually cancelling the token, call [`CancellationTokenSource.Cancel`](xref:System.Threading.CancellationTokenSource.Cancel%2A) to signal that the background work should be cancelled.</span></span>
* <span data-ttu-id="680d8-258">Dopo che la chiamata asincrona restituisce, chiamare <xref:System.Threading.CancellationToken.ThrowIfCancellationRequested%2A> sul token.</span><span class="sxs-lookup"><span data-stu-id="680d8-258">After the asynchronous call returns, call <xref:System.Threading.CancellationToken.ThrowIfCancellationRequested%2A> on the token.</span></span>

<span data-ttu-id="680d8-259">Nell'esempio seguente:</span><span class="sxs-lookup"><span data-stu-id="680d8-259">In the following example:</span></span>

* <span data-ttu-id="680d8-260">`await Task.Delay(5000, cts.Token);` rappresenta il lavoro in background asincrono con esecuzione prolungata.</span><span class="sxs-lookup"><span data-stu-id="680d8-260">`await Task.Delay(5000, cts.Token);` represents long-running asynchronous background work.</span></span>
* <span data-ttu-id="680d8-261">`BackgroundResourceMethod` rappresenta un metodo in background con esecuzione prolungata che non deve essere avviato se l'oggetto `Resource` viene eliminato prima della chiamata al metodo.</span><span class="sxs-lookup"><span data-stu-id="680d8-261">`BackgroundResourceMethod` represents a long-running background method that shouldn't start if the `Resource` is disposed before the method is called.</span></span>

```razor
@implements IDisposable
@using System.Threading

<button @onclick="LongRunningWork">Trigger long running work</button>

@code {
    private Resource resource = new Resource();
    private CancellationTokenSource cts = new CancellationTokenSource();

    protected async Task LongRunningWork()
    {
        await Task.Delay(5000, cts.Token);

        cts.Token.ThrowIfCancellationRequested();
        resource.BackgroundResourceMethod();
    }

    public void Dispose()
    {
        cts.Cancel();
        cts.Dispose();
        resource.Dispose();
    }

    private class Resource : IDisposable
    {
        private bool disposed;

        public void BackgroundResourceMethod()
        {
            if (disposed)
            {
                throw new ObjectDisposedException(nameof(Resource));
            }
            
            ...
        }
        
        public void Dispose()
        {
            disposed = true;
        }
    }
}
```

## <a name="blazor-server-reconnection-events"></a><span data-ttu-id="680d8-262">Blazor Server eventi di riconnessione</span><span class="sxs-lookup"><span data-stu-id="680d8-262">Blazor Server reconnection events</span></span>

<span data-ttu-id="680d8-263">Gli eventi del ciclo di vita dei componenti trattati in questo articolo operano separatamente rispetto ai [ Blazor Server gestori di eventi di riconnessione](xref:blazor/fundamentals/additional-scenarios#reflect-the-connection-state-in-the-ui).</span><span class="sxs-lookup"><span data-stu-id="680d8-263">The component lifecycle events covered in this article operate separately from [Blazor Server's reconnection event handlers](xref:blazor/fundamentals/additional-scenarios#reflect-the-connection-state-in-the-ui).</span></span> <span data-ttu-id="680d8-264">Quando un' Blazor Server app perde la SignalR connessione al client, vengono interrotti solo gli aggiornamenti dell'interfaccia utente.</span><span class="sxs-lookup"><span data-stu-id="680d8-264">When a Blazor Server app loses its SignalR connection to the client, only UI updates are interrupted.</span></span> <span data-ttu-id="680d8-265">Gli aggiornamenti dell'interfaccia utente vengono ripresi quando viene ristabilita la connessione.</span><span class="sxs-lookup"><span data-stu-id="680d8-265">UI updates are resumed when the connection is re-established.</span></span> <span data-ttu-id="680d8-266">Per ulteriori informazioni sulla configurazione e sugli eventi del gestore del circuito, vedere <xref:blazor/fundamentals/additional-scenarios> .</span><span class="sxs-lookup"><span data-stu-id="680d8-266">For more information on circuit handler events and configuration, see <xref:blazor/fundamentals/additional-scenarios>.</span></span>
