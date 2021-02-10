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
ms.openlocfilehash: 03a49c827a1f70e6b721adf293857bb33475ed36
ms.sourcegitcommit: 04ad9cd26fcaa8bd11e261d3661f375f5f343cdc
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/10/2021
ms.locfileid: "100107077"
---
# <a name="aspnet-core-blazor-lifecycle"></a><span data-ttu-id="83a10-103">Ciclo di vita ASP.NET Core Blazor</span><span class="sxs-lookup"><span data-stu-id="83a10-103">ASP.NET Core Blazor lifecycle</span></span>

<span data-ttu-id="83a10-104">Il Blazor Framework include metodi del ciclo di vita sincroni e asincroni.</span><span class="sxs-lookup"><span data-stu-id="83a10-104">The Blazor framework includes synchronous and asynchronous lifecycle methods.</span></span> <span data-ttu-id="83a10-105">Eseguire l'override dei metodi Lifecycle per eseguire operazioni aggiuntive sui componenti durante l'inizializzazione e il rendering dei componenti.</span><span class="sxs-lookup"><span data-stu-id="83a10-105">Override lifecycle methods to perform additional operations on components during component initialization and rendering.</span></span>

<span data-ttu-id="83a10-106">I diagrammi seguenti illustrano il ciclo di vita Blazor .</span><span class="sxs-lookup"><span data-stu-id="83a10-106">The following diagrams illustrate the Blazor lifecycle.</span></span> <span data-ttu-id="83a10-107">I metodi del ciclo di vita sono definiti con esempi nelle sezioni seguenti di questo articolo.</span><span class="sxs-lookup"><span data-stu-id="83a10-107">Lifecycle methods are defined with examples in the following sections of this article.</span></span>

<span data-ttu-id="83a10-108">Eventi del ciclo di vita del componente:</span><span class="sxs-lookup"><span data-stu-id="83a10-108">Component lifecycle events:</span></span>

1. <span data-ttu-id="83a10-109">Se il componente esegue il rendering per la prima volta in una richiesta:</span><span class="sxs-lookup"><span data-stu-id="83a10-109">If the component is rendering for the first time on a request:</span></span>
   * <span data-ttu-id="83a10-110">Creare l'istanza del componente.</span><span class="sxs-lookup"><span data-stu-id="83a10-110">Create the component's instance.</span></span>
   * <span data-ttu-id="83a10-111">Eseguire l'inserimento della proprietà.</span><span class="sxs-lookup"><span data-stu-id="83a10-111">Perform property injection.</span></span> <span data-ttu-id="83a10-112">Eseguire [`SetParametersAsync`](#before-parameters-are-set) .</span><span class="sxs-lookup"><span data-stu-id="83a10-112">Run [`SetParametersAsync`](#before-parameters-are-set).</span></span>
   * <span data-ttu-id="83a10-113">Chiamare [`OnInitialized{Async}`](#component-initialization-methods) .</span><span class="sxs-lookup"><span data-stu-id="83a10-113">Call [`OnInitialized{Async}`](#component-initialization-methods).</span></span> <span data-ttu-id="83a10-114">Se <xref:System.Threading.Tasks.Task> viene restituito un oggetto, <xref:System.Threading.Tasks.Task> viene atteso e viene eseguito il rendering del componente.</span><span class="sxs-lookup"><span data-stu-id="83a10-114">If a <xref:System.Threading.Tasks.Task> is returned, the <xref:System.Threading.Tasks.Task> is awaited and the component is rendered.</span></span> <span data-ttu-id="83a10-115">Se <xref:System.Threading.Tasks.Task> non viene restituito alcun oggetto, viene eseguito il rendering del componente.</span><span class="sxs-lookup"><span data-stu-id="83a10-115">If a <xref:System.Threading.Tasks.Task> isn't returned, the component is rendered.</span></span>
1. <span data-ttu-id="83a10-116">Chiamare [`OnParametersSet{Async}`](#after-parameters-are-set) ed eseguire il rendering del componente.</span><span class="sxs-lookup"><span data-stu-id="83a10-116">Call [`OnParametersSet{Async}`](#after-parameters-are-set) and render the component.</span></span> <span data-ttu-id="83a10-117">Se un oggetto <xref:System.Threading.Tasks.Task> viene restituito da `OnParametersSetAsync` , <xref:System.Threading.Tasks.Task> viene atteso e quindi viene eseguito il rendering del componente.</span><span class="sxs-lookup"><span data-stu-id="83a10-117">If a <xref:System.Threading.Tasks.Task> is returned from `OnParametersSetAsync`, the <xref:System.Threading.Tasks.Task> is awaited and then the component is rerendered.</span></span>

![Eventi del ciclo di vita dei componenti di un elemento::: NO-LOC (Razor)::: Component in::: NO-LOC (Blazer):::](lifecycle/_static/lifecycle1.png)

<span data-ttu-id="83a10-119">Elaborazione evento Document Object Model (DOM):</span><span class="sxs-lookup"><span data-stu-id="83a10-119">Document Object Model (DOM) event processing:</span></span>

1. <span data-ttu-id="83a10-120">Il gestore eventi viene eseguito.</span><span class="sxs-lookup"><span data-stu-id="83a10-120">The event handler is run.</span></span>
1. <span data-ttu-id="83a10-121">Se <xref:System.Threading.Tasks.Task> viene restituito un oggetto, <xref:System.Threading.Tasks.Task> viene atteso e quindi viene eseguito il rendering del componente.</span><span class="sxs-lookup"><span data-stu-id="83a10-121">If a <xref:System.Threading.Tasks.Task> is returned, the <xref:System.Threading.Tasks.Task> is awaited and then the component is rendered.</span></span> <span data-ttu-id="83a10-122">Se <xref:System.Threading.Tasks.Task> non viene restituito alcun oggetto, viene eseguito il rendering del componente.</span><span class="sxs-lookup"><span data-stu-id="83a10-122">If a <xref:System.Threading.Tasks.Task> isn't returned, the component is rendered.</span></span>

![Elaborazione di eventi Document Object Model (DOM)](lifecycle/_static/lifecycle2.png)

<span data-ttu-id="83a10-124">Ciclo di vita `Render` :</span><span class="sxs-lookup"><span data-stu-id="83a10-124">The `Render` lifecycle:</span></span>

1. <span data-ttu-id="83a10-125">Evitare ulteriori operazioni di rendering sul componente:</span><span class="sxs-lookup"><span data-stu-id="83a10-125">Avoid further rendering operations on the component:</span></span>
   * <span data-ttu-id="83a10-126">Dopo il primo rendering.</span><span class="sxs-lookup"><span data-stu-id="83a10-126">After the first render.</span></span>
   * <span data-ttu-id="83a10-127">Quando [`ShouldRender`](#suppress-ui-refreshing) è `false` .</span><span class="sxs-lookup"><span data-stu-id="83a10-127">When [`ShouldRender`](#suppress-ui-refreshing) is `false`.</span></span>
1. <span data-ttu-id="83a10-128">Compilare la diff dell'albero di rendering (differenza) ed eseguire il rendering del componente.</span><span class="sxs-lookup"><span data-stu-id="83a10-128">Build the render tree diff (difference) and render the component.</span></span>
1. <span data-ttu-id="83a10-129">Attendere il DOM da aggiornare.</span><span class="sxs-lookup"><span data-stu-id="83a10-129">Await the DOM to update.</span></span>
1. <span data-ttu-id="83a10-130">Chiamare [`OnAfterRender{Async}`](#after-component-render) .</span><span class="sxs-lookup"><span data-stu-id="83a10-130">Call [`OnAfterRender{Async}`](#after-component-render).</span></span>

![Ciclo di vita rendering](lifecycle/_static/lifecycle3.png)

<span data-ttu-id="83a10-132">Lo sviluppatore chiama per [`StateHasChanged`](#state-changes) generare un rendering.</span><span class="sxs-lookup"><span data-stu-id="83a10-132">Developer calls to [`StateHasChanged`](#state-changes) result in a render.</span></span> <span data-ttu-id="83a10-133">Per altre informazioni, vedere <xref:blazor/components/rendering>.</span><span class="sxs-lookup"><span data-stu-id="83a10-133">For more information, see <xref:blazor/components/rendering>.</span></span>

## <a name="lifecycle-methods"></a><span data-ttu-id="83a10-134">Metodi del ciclo di vita</span><span class="sxs-lookup"><span data-stu-id="83a10-134">Lifecycle methods</span></span>

### <a name="before-parameters-are-set"></a><span data-ttu-id="83a10-135">Prima dell'impostazione dei parametri</span><span class="sxs-lookup"><span data-stu-id="83a10-135">Before parameters are set</span></span>

<span data-ttu-id="83a10-136"><xref:Microsoft.AspNetCore.Components.ComponentBase.SetParametersAsync%2A> Imposta i parametri forniti dall'elemento padre del componente nell'albero di rendering o dai parametri di route.</span><span class="sxs-lookup"><span data-stu-id="83a10-136"><xref:Microsoft.AspNetCore.Components.ComponentBase.SetParametersAsync%2A> sets parameters supplied by the component's parent in the render tree or from route parameters.</span></span> <span data-ttu-id="83a10-137">Eseguendo l'override del metodo, il codice dello sviluppatore può interagire direttamente con i <xref:Microsoft.AspNetCore.Components.ParameterView> parametri di.</span><span class="sxs-lookup"><span data-stu-id="83a10-137">By overriding the method, developer code can interact directly with the <xref:Microsoft.AspNetCore.Components.ParameterView>'s parameters.</span></span>

<span data-ttu-id="83a10-138">Nell'esempio seguente, <xref:Microsoft.AspNetCore.Components.ParameterView.TryGetValue%2A?displayProperty=nameWithType> assegna il valore del `Param` parametro a se l' `value` analisi di un parametro di route per `Param` ha esito positivo.</span><span class="sxs-lookup"><span data-stu-id="83a10-138">In the following example, <xref:Microsoft.AspNetCore.Components.ParameterView.TryGetValue%2A?displayProperty=nameWithType> assigns the `Param` parameter's value to `value` if parsing a route parameter for `Param` is successful.</span></span> <span data-ttu-id="83a10-139">Quando `value` non è `null` , il valore viene visualizzato dal componente.</span><span class="sxs-lookup"><span data-stu-id="83a10-139">When `value` isn't `null`, the value is displayed by the component.</span></span>

<span data-ttu-id="83a10-140">Sebbene la corrispondenza tra i parametri di route non riscontri la [distinzione tra maiuscole](xref:blazor/fundamentals/routing#route-parameters)e minuscole, trova <xref:Microsoft.AspNetCore.Components.ParameterView.TryGetValue%2A> solo i nomi dei parametri con distinzione maiuscole</span><span class="sxs-lookup"><span data-stu-id="83a10-140">Although [route parameter matching is case insensitive](xref:blazor/fundamentals/routing#route-parameters), <xref:Microsoft.AspNetCore.Components.ParameterView.TryGetValue%2A> only matches case sensitive parameter names in the route template.</span></span> <span data-ttu-id="83a10-141">L'esempio seguente è obbligatorio per usare `/{Param?}` , non per `/{param?}` ottenere il valore.</span><span class="sxs-lookup"><span data-stu-id="83a10-141">The following example is required to use `/{Param?}`, not `/{param?}`, in order to get the value.</span></span> <span data-ttu-id="83a10-142">Se `/{param?}` viene usato in questo scenario, <xref:Microsoft.AspNetCore.Components.ParameterView.TryGetValue%2A> restituisce `false` e `message` non è impostato su una delle due stringhe.</span><span class="sxs-lookup"><span data-stu-id="83a10-142">If `/{param?}` is used in this scenario, <xref:Microsoft.AspNetCore.Components.ParameterView.TryGetValue%2A> returns `false` and `message` isn't set to either string.</span></span>

<span data-ttu-id="83a10-143">`Pages/SetParametersAsyncExample.razor`:</span><span class="sxs-lookup"><span data-stu-id="83a10-143">`Pages/SetParametersAsyncExample.razor`:</span></span>

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

<span data-ttu-id="83a10-144"><xref:Microsoft.AspNetCore.Components.ParameterView> contiene il set di valori di parametro per il componente ogni volta che <xref:Microsoft.AspNetCore.Components.ComponentBase.SetParametersAsync%2A> viene chiamato.</span><span class="sxs-lookup"><span data-stu-id="83a10-144"><xref:Microsoft.AspNetCore.Components.ParameterView> contains the set of parameter values for the component each time <xref:Microsoft.AspNetCore.Components.ComponentBase.SetParametersAsync%2A> is called.</span></span>

<span data-ttu-id="83a10-145">L'implementazione predefinita di <xref:Microsoft.AspNetCore.Components.ComponentBase.SetParametersAsync%2A> imposta il valore di ogni proprietà con l' [`[Parameter]`](xref:Microsoft.AspNetCore.Components.ParameterAttribute) [ `[CascadingParameter]` attributo](xref:Microsoft.AspNetCore.Components.CascadingParameterAttribute) o con un valore corrispondente in <xref:Microsoft.AspNetCore.Components.ParameterView> .</span><span class="sxs-lookup"><span data-stu-id="83a10-145">The default implementation of <xref:Microsoft.AspNetCore.Components.ComponentBase.SetParametersAsync%2A> sets the value of each property with the [`[Parameter]`](xref:Microsoft.AspNetCore.Components.ParameterAttribute) or [`[CascadingParameter]` attribute](xref:Microsoft.AspNetCore.Components.CascadingParameterAttribute) that has a corresponding value in the <xref:Microsoft.AspNetCore.Components.ParameterView>.</span></span> <span data-ttu-id="83a10-146">I parametri che non hanno un valore corrispondente in <xref:Microsoft.AspNetCore.Components.ParameterView> rimangono invariati.</span><span class="sxs-lookup"><span data-stu-id="83a10-146">Parameters that don't have a corresponding value in <xref:Microsoft.AspNetCore.Components.ParameterView> are left unchanged.</span></span>

<span data-ttu-id="83a10-147">Se [`base.SetParametersAsync`](xref:Microsoft.AspNetCore.Components.ComponentBase.SetParametersAsync%2A) non viene richiamato, il codice personalizzato può interpretare il valore dei parametri in ingresso in qualsiasi modo necessario.</span><span class="sxs-lookup"><span data-stu-id="83a10-147">If [`base.SetParametersAsync`](xref:Microsoft.AspNetCore.Components.ComponentBase.SetParametersAsync%2A) isn't invoked, the custom code can interpret the incoming parameters value in any way required.</span></span> <span data-ttu-id="83a10-148">Ad esempio, non è necessario assegnare i parametri in ingresso alle proprietà della classe.</span><span class="sxs-lookup"><span data-stu-id="83a10-148">For example, there's no requirement to assign the incoming parameters to the properties on the class.</span></span>

<span data-ttu-id="83a10-149">Se sono configurati gestori di eventi, rimuoverli a disposizione.</span><span class="sxs-lookup"><span data-stu-id="83a10-149">If any event handlers are set up, unhook them on disposal.</span></span> <span data-ttu-id="83a10-150">Per ulteriori informazioni, vedere la sezione relativa all' [eliminazione dei componenti con `IDisposable` ](#component-disposal-with-idisposable) .</span><span class="sxs-lookup"><span data-stu-id="83a10-150">For more information, see the [Component disposal with `IDisposable`](#component-disposal-with-idisposable) section.</span></span>

### <a name="component-initialization-methods"></a><span data-ttu-id="83a10-151">Metodi di inizializzazione componenti</span><span class="sxs-lookup"><span data-stu-id="83a10-151">Component initialization methods</span></span>

<span data-ttu-id="83a10-152"><xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> e <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitialized%2A> vengono richiamati quando il componente viene inizializzato dopo aver ricevuto i parametri iniziali dal componente padre in <xref:Microsoft.AspNetCore.Components.ComponentBase.SetParametersAsync%2A> .</span><span class="sxs-lookup"><span data-stu-id="83a10-152"><xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> and <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitialized%2A> are invoked when the component is initialized after having received its initial parameters from its parent component in <xref:Microsoft.AspNetCore.Components.ComponentBase.SetParametersAsync%2A>.</span></span> 

<span data-ttu-id="83a10-153">Usare <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> quando il componente esegue un'operazione asincrona e deve essere aggiornato al termine dell'operazione.</span><span class="sxs-lookup"><span data-stu-id="83a10-153">Use <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> when the component performs an asynchronous operation and should refresh when the operation is completed.</span></span>

<span data-ttu-id="83a10-154">Per un'operazione sincrona, eseguire l'override di <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitialized%2A> :</span><span class="sxs-lookup"><span data-stu-id="83a10-154">For a synchronous operation, override <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitialized%2A>:</span></span>

```csharp
protected override void OnInitialized()
{
    ...
}
```

<span data-ttu-id="83a10-155">Per eseguire un'operazione asincrona, eseguire l'override <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> e utilizzare l' [`await`](/dotnet/csharp/language-reference/operators/await) operatore sull'operazione:</span><span class="sxs-lookup"><span data-stu-id="83a10-155">To perform an asynchronous operation, override <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> and use the [`await`](/dotnet/csharp/language-reference/operators/await) operator on the operation:</span></span>

```csharp
protected override async Task OnInitializedAsync()
{
    await ...
}
```

<span data-ttu-id="83a10-156">Blazor Server app che [preeseguono](xref:blazor/fundamentals/signalr#render-mode) <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> *due volte* la chiamata al contenuto:</span><span class="sxs-lookup"><span data-stu-id="83a10-156">Blazor Server apps that [prerender their content](xref:blazor/fundamentals/signalr#render-mode) call <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> *twice*:</span></span>

* <span data-ttu-id="83a10-157">Una volta quando il componente viene inizialmente sottoposto a rendering statico come parte della pagina.</span><span class="sxs-lookup"><span data-stu-id="83a10-157">Once when the component is initially rendered statically as part of the page.</span></span>
* <span data-ttu-id="83a10-158">Una seconda volta quando il browser stabilisce una connessione al server.</span><span class="sxs-lookup"><span data-stu-id="83a10-158">A second time when the browser establishes a connection back to the server.</span></span>

<span data-ttu-id="83a10-159">Per impedire l'esecuzione di codice dello sviluppatore per <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> due volte, vedere la sezione [riconnessione con stato dopo il rendering](#stateful-reconnection-after-prerendering) .</span><span class="sxs-lookup"><span data-stu-id="83a10-159">To prevent developer code in <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> from running twice, see the [Stateful reconnection after prerendering](#stateful-reconnection-after-prerendering) section.</span></span>

<span data-ttu-id="83a10-160">Mentre un' Blazor Server app è prerendering, alcune azioni, ad esempio la chiamata a JavaScript, non sono possibili perché non è stata stabilita una connessione con il browser.</span><span class="sxs-lookup"><span data-stu-id="83a10-160">While a Blazor Server app is prerendering, certain actions, such as calling into JavaScript, aren't possible because a connection with the browser hasn't been established.</span></span> <span data-ttu-id="83a10-161">I componenti potrebbero dover eseguire il rendering in modo diverso quando ne viene eseguito il rendering.</span><span class="sxs-lookup"><span data-stu-id="83a10-161">Components may need to render differently when prerendered.</span></span> <span data-ttu-id="83a10-162">Per altre informazioni, vedere la sezione [rilevare quando l'app è prerendering](#detect-when-the-app-is-prerendering) .</span><span class="sxs-lookup"><span data-stu-id="83a10-162">For more information, see the [Detect when the app is prerendering](#detect-when-the-app-is-prerendering) section.</span></span>

<span data-ttu-id="83a10-163">Se sono configurati gestori di eventi, rimuoverli a disposizione.</span><span class="sxs-lookup"><span data-stu-id="83a10-163">If any event handlers are set up, unhook them on disposal.</span></span> <span data-ttu-id="83a10-164">Per ulteriori informazioni, vedere la sezione relativa all' [eliminazione dei componenti con `IDisposable` ](#component-disposal-with-idisposable) .</span><span class="sxs-lookup"><span data-stu-id="83a10-164">For more information, see the [Component disposal with `IDisposable`](#component-disposal-with-idisposable) section.</span></span>

### <a name="after-parameters-are-set"></a><span data-ttu-id="83a10-165">Parametri after impostati</span><span class="sxs-lookup"><span data-stu-id="83a10-165">After parameters are set</span></span>

<span data-ttu-id="83a10-166"><xref:Microsoft.AspNetCore.Components.ComponentBase.OnParametersSetAsync%2A> o <xref:Microsoft.AspNetCore.Components.ComponentBase.OnParametersSet%2A> sono chiamati:</span><span class="sxs-lookup"><span data-stu-id="83a10-166"><xref:Microsoft.AspNetCore.Components.ComponentBase.OnParametersSetAsync%2A> or <xref:Microsoft.AspNetCore.Components.ComponentBase.OnParametersSet%2A> are called:</span></span>

* <span data-ttu-id="83a10-167">Dopo che il componente è stato inizializzato in <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitialized%2A> o <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> .</span><span class="sxs-lookup"><span data-stu-id="83a10-167">After the component is initialized in <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitialized%2A> or <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A>.</span></span>
* <span data-ttu-id="83a10-168">Quando il componente padre esegue nuovamente il rendering e fornisce:</span><span class="sxs-lookup"><span data-stu-id="83a10-168">When the parent component re-renders and supplies:</span></span>
  * <span data-ttu-id="83a10-169">Solo i tipi non modificabili primitivi noti di cui è stato modificato almeno un parametro.</span><span class="sxs-lookup"><span data-stu-id="83a10-169">Only known primitive immutable types of which at least one parameter has changed.</span></span>
  * <span data-ttu-id="83a10-170">Tutti i parametri tipizzati complessi.</span><span class="sxs-lookup"><span data-stu-id="83a10-170">Any complex-typed parameters.</span></span> <span data-ttu-id="83a10-171">Il Framework non è in grado di stabilire se i valori di un parametro tipizzato complesso sono stati modificati internamente, quindi considera il set di parametri come modificato.</span><span class="sxs-lookup"><span data-stu-id="83a10-171">The framework can't know whether the values of a complex-typed parameter have mutated internally, so it treats the parameter set as changed.</span></span>

```csharp
protected override async Task OnParametersSetAsync()
{
    await ...
}
```

> [!NOTE]
> <span data-ttu-id="83a10-172">Il lavoro asincrono quando si applicano parametri e valori di proprietà deve verificarsi durante l'evento del ciclo di vita <xref:Microsoft.AspNetCore.Components.ComponentBase.OnParametersSetAsync%2A> .</span><span class="sxs-lookup"><span data-stu-id="83a10-172">Asynchronous work when applying parameters and property values must occur during the <xref:Microsoft.AspNetCore.Components.ComponentBase.OnParametersSetAsync%2A> lifecycle event.</span></span>

```csharp
protected override void OnParametersSet()
{
    ...
}
```

<span data-ttu-id="83a10-173">Se sono configurati gestori di eventi, rimuoverli a disposizione.</span><span class="sxs-lookup"><span data-stu-id="83a10-173">If any event handlers are set up, unhook them on disposal.</span></span> <span data-ttu-id="83a10-174">Per ulteriori informazioni, vedere la sezione relativa all' [eliminazione dei componenti con `IDisposable` ](#component-disposal-with-idisposable) .</span><span class="sxs-lookup"><span data-stu-id="83a10-174">For more information, see the [Component disposal with `IDisposable`](#component-disposal-with-idisposable) section.</span></span>

### <a name="after-component-render"></a><span data-ttu-id="83a10-175">Rendering del componente dopo</span><span class="sxs-lookup"><span data-stu-id="83a10-175">After component render</span></span>

<span data-ttu-id="83a10-176"><xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A> e <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRender%2A> vengono chiamati dopo che un componente ha terminato il rendering.</span><span class="sxs-lookup"><span data-stu-id="83a10-176"><xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A> and <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRender%2A> are called after a component has finished rendering.</span></span> <span data-ttu-id="83a10-177">I riferimenti a elementi e componenti vengono popolati a questo punto.</span><span class="sxs-lookup"><span data-stu-id="83a10-177">Element and component references are populated at this point.</span></span> <span data-ttu-id="83a10-178">Usare questa fase per eseguire passaggi di inizializzazione aggiuntivi usando il contenuto sottoposto a rendering, ad esempio l'attivazione di librerie JavaScript di terze parti che operano sugli elementi DOM sottoposti a rendering.</span><span class="sxs-lookup"><span data-stu-id="83a10-178">Use this stage to perform additional initialization steps using the rendered content, such as activating third-party JavaScript libraries that operate on the rendered DOM elements.</span></span>

<span data-ttu-id="83a10-179">Il `firstRender` parametro per <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A> e <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRender%2A> :</span><span class="sxs-lookup"><span data-stu-id="83a10-179">The `firstRender` parameter for <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A> and <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRender%2A>:</span></span>

* <span data-ttu-id="83a10-180">Viene impostato `true` sulla prima volta che viene eseguito il rendering dell'istanza del componente.</span><span class="sxs-lookup"><span data-stu-id="83a10-180">Is set to `true` the first time that the component instance is rendered.</span></span>
* <span data-ttu-id="83a10-181">Può essere usato per garantire che il lavoro di inizializzazione venga eseguito una sola volta.</span><span class="sxs-lookup"><span data-stu-id="83a10-181">Can be used to ensure that initialization work is only performed once.</span></span>

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
> <span data-ttu-id="83a10-182">Il lavoro asincrono immediatamente dopo il rendering deve verificarsi durante l'evento del ciclo di vita <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A> .</span><span class="sxs-lookup"><span data-stu-id="83a10-182">Asynchronous work immediately after rendering must occur during the <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A> lifecycle event.</span></span>
>
> <span data-ttu-id="83a10-183">Anche se si restituisce un oggetto <xref:System.Threading.Tasks.Task> da <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A> , il Framework non pianifica un ulteriore ciclo di rendering per il componente al termine dell'attività.</span><span class="sxs-lookup"><span data-stu-id="83a10-183">Even if you return a <xref:System.Threading.Tasks.Task> from <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A>, the framework doesn't schedule a further render cycle for your component once that task completes.</span></span> <span data-ttu-id="83a10-184">In questo modo è possibile evitare un ciclo di rendering infinito.</span><span class="sxs-lookup"><span data-stu-id="83a10-184">This is to avoid an infinite render loop.</span></span> <span data-ttu-id="83a10-185">È diverso dagli altri metodi del ciclo di vita, che pianificano un ulteriore ciclo di rendering dopo il completamento dell'attività restituita.</span><span class="sxs-lookup"><span data-stu-id="83a10-185">It's different from the other lifecycle methods, which schedule a further render cycle once the returned task completes.</span></span>

```csharp
protected override void OnAfterRender(bool firstRender)
{
    if (firstRender)
    {
        ...
    }
}
```

<span data-ttu-id="83a10-186"><xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRender%2A> e <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A> *non vengono chiamati durante il processo di prerendering sul server*.</span><span class="sxs-lookup"><span data-stu-id="83a10-186"><xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRender%2A> and <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A> *aren't called during the prerendering process on the server*.</span></span> <span data-ttu-id="83a10-187">I metodi vengono chiamati quando il componente viene sottoposto a rendering interattivo dopo il completamento del rendering.</span><span class="sxs-lookup"><span data-stu-id="83a10-187">The methods are called when the component is rendered interactively after prerendering is finished.</span></span> <span data-ttu-id="83a10-188">Quando viene eseguito il rendering dell'app:</span><span class="sxs-lookup"><span data-stu-id="83a10-188">When the app prerenders:</span></span>

1. <span data-ttu-id="83a10-189">Il componente viene eseguito sul server per produrre un markup HTML statico nella risposta HTTP.</span><span class="sxs-lookup"><span data-stu-id="83a10-189">The component executes on the server to produce some static HTML markup in the HTTP response.</span></span> <span data-ttu-id="83a10-190">Durante questa fase, <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRender%2A> <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A> non viene chiamato.</span><span class="sxs-lookup"><span data-stu-id="83a10-190">During this phase, <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRender%2A> and <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A> aren't called.</span></span>
1. <span data-ttu-id="83a10-191">Quando `blazor.server.js` o `blazor.webassembly.js` si avvia nel browser, il componente viene riavviato in modalità di rendering interattiva.</span><span class="sxs-lookup"><span data-stu-id="83a10-191">When `blazor.server.js` or `blazor.webassembly.js` start up in the browser, the component is restarted in an interactive rendering mode.</span></span> <span data-ttu-id="83a10-192">Dopo che un componente viene riavviato <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRender%2A> e <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A> **viene** chiamato perché l'app non è più all'interno della fase di prerendering.</span><span class="sxs-lookup"><span data-stu-id="83a10-192">After a component is restarted, <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRender%2A> and <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A> **are** called because the app isn't inside the prerendering phase any longer.</span></span>

<span data-ttu-id="83a10-193">Se sono configurati gestori di eventi, rimuoverli a disposizione.</span><span class="sxs-lookup"><span data-stu-id="83a10-193">If any event handlers are set up, unhook them on disposal.</span></span> <span data-ttu-id="83a10-194">Per ulteriori informazioni, vedere la sezione relativa all' [eliminazione dei componenti con `IDisposable` ](#component-disposal-with-idisposable) .</span><span class="sxs-lookup"><span data-stu-id="83a10-194">For more information, see the [Component disposal with `IDisposable`](#component-disposal-with-idisposable) section.</span></span>

### <a name="suppress-ui-refreshing"></a><span data-ttu-id="83a10-195">Evita aggiornamento dell'interfaccia utente</span><span class="sxs-lookup"><span data-stu-id="83a10-195">Suppress UI refreshing</span></span>

<span data-ttu-id="83a10-196">Eseguire l'override <xref:Microsoft.AspNetCore.Components.ComponentBase.ShouldRender%2A> per impedire l'aggiornamento dell'interfaccia utente.</span><span class="sxs-lookup"><span data-stu-id="83a10-196">Override <xref:Microsoft.AspNetCore.Components.ComponentBase.ShouldRender%2A> to suppress UI refreshing.</span></span> <span data-ttu-id="83a10-197">Se l'implementazione restituisce `true` , l'interfaccia utente viene aggiornata:</span><span class="sxs-lookup"><span data-stu-id="83a10-197">If the implementation returns `true`, the UI is refreshed:</span></span>

```csharp
protected override bool ShouldRender()
{
    var renderUI = true;

    return renderUI;
}
```

<span data-ttu-id="83a10-198"><xref:Microsoft.AspNetCore.Components.ComponentBase.ShouldRender%2A> viene chiamato ogni volta che il componente viene sottoposto a rendering.</span><span class="sxs-lookup"><span data-stu-id="83a10-198"><xref:Microsoft.AspNetCore.Components.ComponentBase.ShouldRender%2A> is called each time the component is rendered.</span></span>

<span data-ttu-id="83a10-199">Anche se <xref:Microsoft.AspNetCore.Components.ComponentBase.ShouldRender%2A> viene sottoposto a override, il componente viene sempre sottoposto a rendering iniziale.</span><span class="sxs-lookup"><span data-stu-id="83a10-199">Even if <xref:Microsoft.AspNetCore.Components.ComponentBase.ShouldRender%2A> is overridden, the component is always initially rendered.</span></span>

<span data-ttu-id="83a10-200">Per altre informazioni, vedere <xref:blazor/webassembly-performance-best-practices#avoid-unnecessary-rendering-of-component-subtrees>.</span><span class="sxs-lookup"><span data-stu-id="83a10-200">For more information, see <xref:blazor/webassembly-performance-best-practices#avoid-unnecessary-rendering-of-component-subtrees>.</span></span>

## <a name="state-changes"></a><span data-ttu-id="83a10-201">Modifiche stato</span><span class="sxs-lookup"><span data-stu-id="83a10-201">State changes</span></span>

<span data-ttu-id="83a10-202"><xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> notifica al componente che lo stato è stato modificato.</span><span class="sxs-lookup"><span data-stu-id="83a10-202"><xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> notifies the component that its state has changed.</span></span> <span data-ttu-id="83a10-203">Se applicabile, <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> la chiamata a comporta il rendering del componente.</span><span class="sxs-lookup"><span data-stu-id="83a10-203">When applicable, calling <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> causes the component to be rerendered.</span></span>

<span data-ttu-id="83a10-204"><xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> viene chiamato automaticamente per i <xref:Microsoft.AspNetCore.Components.EventCallback> metodi.</span><span class="sxs-lookup"><span data-stu-id="83a10-204"><xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> is called automatically for <xref:Microsoft.AspNetCore.Components.EventCallback> methods.</span></span> <span data-ttu-id="83a10-205">Per altre informazioni, vedere <xref:blazor/components/event-handling#eventcallback>.</span><span class="sxs-lookup"><span data-stu-id="83a10-205">For more information, see <xref:blazor/components/event-handling#eventcallback>.</span></span>

<span data-ttu-id="83a10-206">Per altre informazioni, vedere <xref:blazor/components/rendering>.</span><span class="sxs-lookup"><span data-stu-id="83a10-206">For more information, see <xref:blazor/components/rendering>.</span></span>

## <a name="handle-incomplete-async-actions-at-render"></a><span data-ttu-id="83a10-207">Gestisci azioni asincrone incomplete durante il rendering</span><span class="sxs-lookup"><span data-stu-id="83a10-207">Handle incomplete async actions at render</span></span>

<span data-ttu-id="83a10-208">Le azioni asincrone eseguite negli eventi del ciclo di vita potrebbero non essere state completate prima del rendering del componente.</span><span class="sxs-lookup"><span data-stu-id="83a10-208">Asynchronous actions performed in lifecycle events might not have completed before the component is rendered.</span></span> <span data-ttu-id="83a10-209">Gli oggetti possono essere `null` o compilati in modo non completo con i dati mentre è in esecuzione il metodo del ciclo di vita.</span><span class="sxs-lookup"><span data-stu-id="83a10-209">Objects might be `null` or incompletely populated with data while the lifecycle method is executing.</span></span> <span data-ttu-id="83a10-210">Fornire la logica di rendering per confermare che gli oggetti vengono inizializzati.</span><span class="sxs-lookup"><span data-stu-id="83a10-210">Provide rendering logic to confirm that objects are initialized.</span></span> <span data-ttu-id="83a10-211">Esegue il rendering degli elementi dell'interfaccia utente segnaposto (ad esempio, un messaggio di caricamento) mentre gli oggetti sono `null` .</span><span class="sxs-lookup"><span data-stu-id="83a10-211">Render placeholder UI elements (for example, a loading message) while objects are `null`.</span></span>

<span data-ttu-id="83a10-212">Nel `FetchData` componente dei Blazor modelli <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> viene eseguito l'override di per ricevere i dati di previsione () in modo asincrono `forecasts` .</span><span class="sxs-lookup"><span data-stu-id="83a10-212">In the `FetchData` component of the Blazor templates, <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> is overridden to asynchronously receive forecast data (`forecasts`).</span></span> <span data-ttu-id="83a10-213">Quando `forecasts` è `null` , all'utente viene visualizzato un messaggio di caricamento.</span><span class="sxs-lookup"><span data-stu-id="83a10-213">When `forecasts` is `null`, a loading message is displayed to the user.</span></span> <span data-ttu-id="83a10-214">Quando l'oggetto `Task` restituito da <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> viene completato, viene eseguito il rendering del componente con lo stato aggiornato.</span><span class="sxs-lookup"><span data-stu-id="83a10-214">After the `Task` returned by <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> completes, the component is rerendered with the updated state.</span></span>

<span data-ttu-id="83a10-215">`Pages/FetchData.razor` nel Blazor Server modello:</span><span class="sxs-lookup"><span data-stu-id="83a10-215">`Pages/FetchData.razor` in the Blazor Server template:</span></span>

[!code-razor[](lifecycle/samples_snapshot/FetchData.razor?highlight=9,21,25)]

## <a name="handle-errors"></a><span data-ttu-id="83a10-216">Gestire gli errori</span><span class="sxs-lookup"><span data-stu-id="83a10-216">Handle errors</span></span>

<span data-ttu-id="83a10-217">Per informazioni sulla gestione degli errori durante l'esecuzione del metodo del ciclo di vita, vedere <xref:blazor/fundamentals/handle-errors#lifecycle-methods> .</span><span class="sxs-lookup"><span data-stu-id="83a10-217">For information on handling errors during lifecycle method execution, see <xref:blazor/fundamentals/handle-errors#lifecycle-methods>.</span></span>

## <a name="stateful-reconnection-after-prerendering"></a><span data-ttu-id="83a10-218">Riconnessione con stato dopo il rendering preliminare</span><span class="sxs-lookup"><span data-stu-id="83a10-218">Stateful reconnection after prerendering</span></span>

<span data-ttu-id="83a10-219">In un' Blazor Server app quando <xref:Microsoft.AspNetCore.Mvc.TagHelpers.ComponentTagHelper.RenderMode> è <xref:Microsoft.AspNetCore.Mvc.Rendering.RenderMode.ServerPrerendered> , il componente viene inizialmente sottoposto a rendering statico come parte della pagina.</span><span class="sxs-lookup"><span data-stu-id="83a10-219">In a Blazor Server app when <xref:Microsoft.AspNetCore.Mvc.TagHelpers.ComponentTagHelper.RenderMode> is <xref:Microsoft.AspNetCore.Mvc.Rendering.RenderMode.ServerPrerendered>, the component is initially rendered statically as part of the page.</span></span> <span data-ttu-id="83a10-220">Quando il browser stabilisce una connessione al server, viene *nuovamente* eseguito il rendering del componente e il componente è ora interattivo.</span><span class="sxs-lookup"><span data-stu-id="83a10-220">Once the browser establishes a connection back to the server, the component is rendered *again*, and the component is now interactive.</span></span> <span data-ttu-id="83a10-221">Se [`OnInitialized{Async}`](#component-initialization-methods) è presente il metodo del ciclo di vita per l'inizializzazione del componente, il metodo viene eseguito *due volte*:</span><span class="sxs-lookup"><span data-stu-id="83a10-221">If the [`OnInitialized{Async}`](#component-initialization-methods) lifecycle method for initializing the component is present, the method is executed *twice*:</span></span>

* <span data-ttu-id="83a10-222">Quando il componente viene preeseguito in modo statico.</span><span class="sxs-lookup"><span data-stu-id="83a10-222">When the component is prerendered statically.</span></span>
* <span data-ttu-id="83a10-223">Una volta stabilita la connessione al server.</span><span class="sxs-lookup"><span data-stu-id="83a10-223">After the server connection has been established.</span></span>

<span data-ttu-id="83a10-224">Ciò può comportare una modifica evidente nei dati visualizzati nell'interfaccia utente quando il componente viene sottoposto a rendering.</span><span class="sxs-lookup"><span data-stu-id="83a10-224">This can result in a noticeable change in the data displayed in the UI when the component is finally rendered.</span></span>

<span data-ttu-id="83a10-225">Per evitare lo scenario di doppio rendering in un' Blazor Server app:</span><span class="sxs-lookup"><span data-stu-id="83a10-225">To avoid the double-rendering scenario in a Blazor Server app:</span></span>

* <span data-ttu-id="83a10-226">Passare un identificatore che può essere usato per memorizzare nella cache lo stato durante il prerendering e recuperare lo stato dopo il riavvio dell'app.</span><span class="sxs-lookup"><span data-stu-id="83a10-226">Pass in an identifier that can be used to cache the state during prerendering and to retrieve the state after the app restarts.</span></span>
* <span data-ttu-id="83a10-227">Utilizzare l'identificatore durante il prerendering per salvare lo stato del componente.</span><span class="sxs-lookup"><span data-stu-id="83a10-227">Use the identifier during prerendering to save component state.</span></span>
* <span data-ttu-id="83a10-228">Utilizzare l'identificatore dopo il prerendering per recuperare lo stato memorizzato nella cache.</span><span class="sxs-lookup"><span data-stu-id="83a10-228">Use the identifier after prerendering to retrieve the cached state.</span></span>

<span data-ttu-id="83a10-229">Il codice seguente illustra un aggiornamento `WeatherForecastService` in un'app basata su modello Blazor Server che evita il doppio rendering:</span><span class="sxs-lookup"><span data-stu-id="83a10-229">The following code demonstrates an updated `WeatherForecastService` in a template-based Blazor Server app that avoids the double rendering:</span></span>

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

<span data-ttu-id="83a10-230">Per ulteriori informazioni su <xref:Microsoft.AspNetCore.Mvc.TagHelpers.ComponentTagHelper.RenderMode> , vedere <xref:blazor/fundamentals/signalr#render-mode> .</span><span class="sxs-lookup"><span data-stu-id="83a10-230">For more information on the <xref:Microsoft.AspNetCore.Mvc.TagHelpers.ComponentTagHelper.RenderMode>, see <xref:blazor/fundamentals/signalr#render-mode>.</span></span>

## <a name="detect-when-the-app-is-prerendering"></a><span data-ttu-id="83a10-231">Rilevare il momento in cui viene eseguito il prerendering dell'app</span><span class="sxs-lookup"><span data-stu-id="83a10-231">Detect when the app is prerendering</span></span>

[!INCLUDE[](~/blazor/includes/prerendering.md)]

## <a name="component-disposal-with-idisposable"></a><span data-ttu-id="83a10-232">Eliminazione di componenti con `IDisposable`</span><span class="sxs-lookup"><span data-stu-id="83a10-232">Component disposal with `IDisposable`</span></span>

<span data-ttu-id="83a10-233">Se un componente implementa <xref:System.IDisposable> , il Framework chiama il [metodo di eliminazione](/dotnet/standard/garbage-collection/implementing-dispose) quando il componente viene rimosso dall'interfaccia utente, in cui è possibile rilasciare le risorse non gestite.</span><span class="sxs-lookup"><span data-stu-id="83a10-233">If a component implements <xref:System.IDisposable>, the framework calls the [disposal method](/dotnet/standard/garbage-collection/implementing-dispose) when the component is removed from the UI, where unmanaged resources can be released.</span></span> <span data-ttu-id="83a10-234">L'eliminazione può essere eseguita in qualsiasi momento, incluso durante l' [inizializzazione dei componenti](#component-initialization-methods).</span><span class="sxs-lookup"><span data-stu-id="83a10-234">Disposal can occur at any time, including during [component initialization](#component-initialization-methods).</span></span> <span data-ttu-id="83a10-235">Il componente seguente implementa <xref:System.IDisposable> con la [`@implements`](xref:mvc/views/razor#implements) Razor direttiva:</span><span class="sxs-lookup"><span data-stu-id="83a10-235">The following component implements <xref:System.IDisposable> with the [`@implements`](xref:mvc/views/razor#implements) Razor directive:</span></span>

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

<span data-ttu-id="83a10-236">Se un oggetto richiede l'eliminazione, è possibile usare un'espressione lambda per eliminare l'oggetto quando <xref:System.IDisposable.Dispose%2A?displayProperty=nameWithType> viene chiamato:</span><span class="sxs-lookup"><span data-stu-id="83a10-236">If an object requires disposal, a lambda can be used to dispose of the object when <xref:System.IDisposable.Dispose%2A?displayProperty=nameWithType> is called:</span></span>

<span data-ttu-id="83a10-237">`Pages/CounterWithTimerDisposal.razor`:</span><span class="sxs-lookup"><span data-stu-id="83a10-237">`Pages/CounterWithTimerDisposal.razor`:</span></span>

```razor
@page "/counter-with-timer-disposal"
@using System.Timers
@implements IDisposable

<h1>Counter with <code>Timer</code> disposal</h1>

<p>Current count: @currentCount</p>

@code {
    private int currentCount = 0;
    private Timer timer = new Timer(1000);

    protected override void OnInitialized()
    {
        timer.Elapsed += (sender, eventArgs) => OnTimerCallback();
        timer.Start();
    }

    private void OnTimerCallback()
    {
        _ = InvokeAsync(() =>
        {
            currentCount++;
            StateHasChanged();
        });
    }

    public void IDisposable.Dispose() => timer.Dispose();
}
```

<span data-ttu-id="83a10-238">L'esempio precedente viene visualizzato in <xref:blazor/components/rendering#receiving-a-call-from-something-external-to-the-blazor-rendering-and-event-handling-system> .</span><span class="sxs-lookup"><span data-stu-id="83a10-238">The preceding example appears in <xref:blazor/components/rendering#receiving-a-call-from-something-external-to-the-blazor-rendering-and-event-handling-system>.</span></span>

<span data-ttu-id="83a10-239">Per le attività di eliminazione asincrona, usare `DisposeAsync` anziché <xref:System.IDisposable.Dispose> :</span><span class="sxs-lookup"><span data-stu-id="83a10-239">For asynchronous disposal tasks, use `DisposeAsync` instead of <xref:System.IDisposable.Dispose>:</span></span>

```csharp
public async ValueTask DisposeAsync()
{
    ...
}
```

> [!NOTE]
> <span data-ttu-id="83a10-240">La chiamata <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> di in `Dispose` non è supportata.</span><span class="sxs-lookup"><span data-stu-id="83a10-240">Calling <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> in `Dispose` isn't supported.</span></span> <span data-ttu-id="83a10-241"><xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> potrebbe essere richiamato come parte del propagazione del renderer, quindi la richiesta di aggiornamenti dell'interfaccia utente in quel momento non è supportata.</span><span class="sxs-lookup"><span data-stu-id="83a10-241"><xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> might be invoked as part of tearing down the renderer, so requesting UI updates at that point isn't supported.</span></span>

<span data-ttu-id="83a10-242">Annulla la sottoscrizione di gestori eventi da eventi .NET.</span><span class="sxs-lookup"><span data-stu-id="83a10-242">Unsubscribe event handlers from .NET events.</span></span> <span data-ttu-id="83a10-243">Gli esempi di [ Blazor form](xref:blazor/forms-validation) seguenti illustrano come annullare la sottoscrizione di un gestore eventi nel `Dispose` Metodo:</span><span class="sxs-lookup"><span data-stu-id="83a10-243">The following [Blazor form](xref:blazor/forms-validation) examples show how to unsubscribe an event handler in the `Dispose` method:</span></span>

* <span data-ttu-id="83a10-244">Approccio basato su campo privato e lambda</span><span class="sxs-lookup"><span data-stu-id="83a10-244">Private field and lambda approach</span></span>

  [!code-razor[](lifecycle/samples_snapshot/event-handler-disposal-1.razor?highlight=23,28)]

* <span data-ttu-id="83a10-245">Approccio metodo privato</span><span class="sxs-lookup"><span data-stu-id="83a10-245">Private method approach</span></span>

  [!code-razor[](lifecycle/samples_snapshot/event-handler-disposal-2.razor?highlight=16,26)]

<span data-ttu-id="83a10-246">Quando vengono utilizzate funzioni, metodi o espressioni [anonime](/dotnet/csharp/programming-guide/statements-expressions-operators/anonymous-functions), non è necessario implementare <xref:System.IDisposable> e annullare la sottoscrizione di delegati.</span><span class="sxs-lookup"><span data-stu-id="83a10-246">When [anonymous functions](/dotnet/csharp/programming-guide/statements-expressions-operators/anonymous-functions), methods or expressions, are used, it isn't necessary to implement <xref:System.IDisposable> and unsubscribe delegates.</span></span> <span data-ttu-id="83a10-247">Tuttavia, la mancata sottoscrizione di un delegato è un problema **quando l'oggetto che espone l'evento sopravvive alla durata del componente che registra il delegato**.</span><span class="sxs-lookup"><span data-stu-id="83a10-247">However, failing to unsubscribe a delegate is a problem **when the object exposing the event outlives the lifetime of the component registering the delegate**.</span></span> <span data-ttu-id="83a10-248">In tal caso, si verifica una perdita di memoria perché il delegato registrato mantiene attivo l'oggetto originale.</span><span class="sxs-lookup"><span data-stu-id="83a10-248">When this occurs, a memory leak results because the registered delegate keeps the original object alive.</span></span> <span data-ttu-id="83a10-249">Pertanto, utilizzare solo gli approcci seguenti quando si è certi che il delegato dell'evento viene eliminato rapidamente.</span><span class="sxs-lookup"><span data-stu-id="83a10-249">Therefore, only use the following approaches when you know that the event delegate disposes quickly.</span></span> <span data-ttu-id="83a10-250">In caso di dubbi sulla durata degli oggetti che richiedono l'eliminazione, sottoscrivere un metodo delegato ed eliminare correttamente il delegato come illustrato negli esempi precedenti.</span><span class="sxs-lookup"><span data-stu-id="83a10-250">When in doubt about the lifetime of objects that require disposal, subscribe a delegate method and properly dispose the delegate as the preceding examples show.</span></span>

* <span data-ttu-id="83a10-251">Approccio al metodo lambda Anonimo (eliminazione esplicita non necessaria)</span><span class="sxs-lookup"><span data-stu-id="83a10-251">Anonymous lambda method approach (explicit disposal not required)</span></span>

  ```csharp
  private void HandleFieldChanged(object sender, FieldChangedEventArgs e)
  {
      formInvalid = !editContext.Validate();
      StateHasChanged();
  }

  protected override void OnInitialized()
  {
      editContext = new EditContext(starship);
      editContext.OnFieldChanged += (s, e) => HandleFieldChanged((editContext)s, e);
  }
  ```

* <span data-ttu-id="83a10-252">Approccio per le espressioni lambda anonime (eliminazione esplicita non necessaria)</span><span class="sxs-lookup"><span data-stu-id="83a10-252">Anonymous lambda expression approach (explicit disposal not required)</span></span>

  ```csharp
  private ValidationMessageStore messageStore;

  [CascadingParameter]
  private EditContext CurrentEditContext { get; set; }

  protected override void OnInitialized()
  {
      ...

      messageStore = new ValidationMessageStore(CurrentEditContext);

      CurrentEditContext.OnValidationRequested += (s, e) => messageStore.Clear();
      CurrentEditContext.OnFieldChanged += (s, e) => 
          messageStore.Clear(e.FieldIdentifier);
  }
  ```

  <span data-ttu-id="83a10-253">Nell'esempio completo del codice precedente con espressioni lambda anonime viene visualizzato <xref:blazor/forms-validation#validator-components> .</span><span class="sxs-lookup"><span data-stu-id="83a10-253">The full example of the preceding code with anonymous lambda expressions appears in <xref:blazor/forms-validation#validator-components>.</span></span>

<span data-ttu-id="83a10-254">Per ulteriori informazioni, vedere la pagina relativa alla [pulizia di risorse non gestite](/dotnet/standard/garbage-collection/unmanaged) e agli argomenti che lo seguono sull'implementazione dei `Dispose` `DisposeAsync` metodi e.</span><span class="sxs-lookup"><span data-stu-id="83a10-254">For more information, see [Cleaning up unmanaged resources](/dotnet/standard/garbage-collection/unmanaged) and the topics that follow it on implementing the `Dispose` and `DisposeAsync` methods.</span></span>

## <a name="cancelable-background-work"></a><span data-ttu-id="83a10-255">Lavoro in background annullabile</span><span class="sxs-lookup"><span data-stu-id="83a10-255">Cancelable background work</span></span>

<span data-ttu-id="83a10-256">I componenti eseguono spesso attività in background con esecuzione prolungata, ad esempio la creazione di chiamate di rete ( <xref:System.Net.Http.HttpClient> ) e l'interazione con i database.</span><span class="sxs-lookup"><span data-stu-id="83a10-256">Components often perform long-running background work, such as making network calls (<xref:System.Net.Http.HttpClient>) and interacting with databases.</span></span> <span data-ttu-id="83a10-257">È consigliabile arrestare il lavoro in background per conservare le risorse di sistema in diverse situazioni.</span><span class="sxs-lookup"><span data-stu-id="83a10-257">It's desirable to stop the background work to conserve system resources in several situations.</span></span> <span data-ttu-id="83a10-258">Ad esempio, le operazioni asincrone in background non vengono arrestate automaticamente quando un utente si sposta da un componente.</span><span class="sxs-lookup"><span data-stu-id="83a10-258">For example, background asynchronous operations don't automatically stop when a user navigates away from a component.</span></span>

<span data-ttu-id="83a10-259">Altri motivi per cui gli elementi di lavoro in background potrebbero richiedere l'annullamento includono:</span><span class="sxs-lookup"><span data-stu-id="83a10-259">Other reasons why background work items might require cancellation include:</span></span>

* <span data-ttu-id="83a10-260">Un'attività in background in esecuzione è stata avviata con i parametri di elaborazione o i dati di input difettosi.</span><span class="sxs-lookup"><span data-stu-id="83a10-260">An executing background task was started with faulty input data or processing parameters.</span></span>
* <span data-ttu-id="83a10-261">Il set corrente di elementi di lavoro in background in esecuzione deve essere sostituito con un nuovo set di elementi di lavoro.</span><span class="sxs-lookup"><span data-stu-id="83a10-261">The current set of executing background work items must be replaced with a new set of work items.</span></span>
* <span data-ttu-id="83a10-262">È necessario modificare la priorità delle attività attualmente in esecuzione.</span><span class="sxs-lookup"><span data-stu-id="83a10-262">The priority of currently executing tasks must be changed.</span></span>
* <span data-ttu-id="83a10-263">L'app deve essere arrestata per poterla ridistribuire nel server.</span><span class="sxs-lookup"><span data-stu-id="83a10-263">The app has to be shut down in order to redeploy it to the server.</span></span>
* <span data-ttu-id="83a10-264">Le risorse del server diventano limitate, rendendo necessaria la ripianificazione degli elementi di lavoro in background.</span><span class="sxs-lookup"><span data-stu-id="83a10-264">Server resources become limited, necessitating the rescheduling of background work items.</span></span>

<span data-ttu-id="83a10-265">Per implementare un modello di lavoro in background annullabile in un componente:</span><span class="sxs-lookup"><span data-stu-id="83a10-265">To implement a cancelable background work pattern in a component:</span></span>

* <span data-ttu-id="83a10-266">Utilizzare <xref:System.Threading.CancellationTokenSource> e <xref:System.Threading.CancellationToken> .</span><span class="sxs-lookup"><span data-stu-id="83a10-266">Use a <xref:System.Threading.CancellationTokenSource> and <xref:System.Threading.CancellationToken>.</span></span>
* <span data-ttu-id="83a10-267">In caso di [eliminazione del componente](#component-disposal-with-idisposable) e in qualsiasi momento si desideri annullare l'annullamento del token, chiamare [`CancellationTokenSource.Cancel`](xref:System.Threading.CancellationTokenSource.Cancel%2A) per segnalare che le attività in background devono essere annullate.</span><span class="sxs-lookup"><span data-stu-id="83a10-267">On [disposal of the component](#component-disposal-with-idisposable) and at any point cancellation is desired by manually cancelling the token, call [`CancellationTokenSource.Cancel`](xref:System.Threading.CancellationTokenSource.Cancel%2A) to signal that the background work should be cancelled.</span></span>
* <span data-ttu-id="83a10-268">Dopo che la chiamata asincrona restituisce, chiamare <xref:System.Threading.CancellationToken.ThrowIfCancellationRequested%2A> sul token.</span><span class="sxs-lookup"><span data-stu-id="83a10-268">After the asynchronous call returns, call <xref:System.Threading.CancellationToken.ThrowIfCancellationRequested%2A> on the token.</span></span>

<span data-ttu-id="83a10-269">Nell'esempio seguente:</span><span class="sxs-lookup"><span data-stu-id="83a10-269">In the following example:</span></span>

* <span data-ttu-id="83a10-270">`await Task.Delay(5000, cts.Token);` rappresenta il lavoro in background asincrono con esecuzione prolungata.</span><span class="sxs-lookup"><span data-stu-id="83a10-270">`await Task.Delay(5000, cts.Token);` represents long-running asynchronous background work.</span></span>
* <span data-ttu-id="83a10-271">`BackgroundResourceMethod` rappresenta un metodo in background con esecuzione prolungata che non deve essere avviato se l'oggetto `Resource` viene eliminato prima della chiamata al metodo.</span><span class="sxs-lookup"><span data-stu-id="83a10-271">`BackgroundResourceMethod` represents a long-running background method that shouldn't start if the `Resource` is disposed before the method is called.</span></span>

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

## <a name="blazor-server-reconnection-events"></a><span data-ttu-id="83a10-272">Blazor Server eventi di riconnessione</span><span class="sxs-lookup"><span data-stu-id="83a10-272">Blazor Server reconnection events</span></span>

<span data-ttu-id="83a10-273">Gli eventi del ciclo di vita dei componenti trattati in questo articolo operano separatamente rispetto ai [ Blazor Server gestori di eventi di riconnessione](xref:blazor/fundamentals/signalr#reflect-the-connection-state-in-the-ui).</span><span class="sxs-lookup"><span data-stu-id="83a10-273">The component lifecycle events covered in this article operate separately from [Blazor Server's reconnection event handlers](xref:blazor/fundamentals/signalr#reflect-the-connection-state-in-the-ui).</span></span> <span data-ttu-id="83a10-274">Quando un' Blazor Server app perde la SignalR connessione al client, vengono interrotti solo gli aggiornamenti dell'interfaccia utente.</span><span class="sxs-lookup"><span data-stu-id="83a10-274">When a Blazor Server app loses its SignalR connection to the client, only UI updates are interrupted.</span></span> <span data-ttu-id="83a10-275">Gli aggiornamenti dell'interfaccia utente vengono ripresi quando viene ristabilita la connessione.</span><span class="sxs-lookup"><span data-stu-id="83a10-275">UI updates are resumed when the connection is re-established.</span></span> <span data-ttu-id="83a10-276">Per ulteriori informazioni sulla configurazione e sugli eventi del gestore del circuito, vedere <xref:blazor/fundamentals/signalr> .</span><span class="sxs-lookup"><span data-stu-id="83a10-276">For more information on circuit handler events and configuration, see <xref:blazor/fundamentals/signalr>.</span></span>
