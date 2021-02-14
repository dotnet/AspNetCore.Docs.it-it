---
title: BlazorRendering componente ASP.NET Core
author: guardrex
description: Informazioni sul Razor rendering dei componenti nelle Blazor app ASP.NET Core, inclusa la chiamata a StateHasChanged.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 01/13/2021
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
uid: blazor/components/rendering
ms.openlocfilehash: e1222981d4af3f4e233cdc0c57bb96a71972af15
ms.sourcegitcommit: 1166b0ff3828418559510c661e8240e5c5717bb7
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/12/2021
ms.locfileid: "100280042"
---
# <a name="aspnet-core-blazor-component-rendering"></a><span data-ttu-id="b1ab2-103">BlazorRendering componente ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="b1ab2-103">ASP.NET Core Blazor component rendering</span></span>

<span data-ttu-id="b1ab2-104">I componenti *devono* essere sottoposte a rendering quando vengono aggiunti per la prima volta alla gerarchia dei componenti tramite il componente padre.</span><span class="sxs-lookup"><span data-stu-id="b1ab2-104">Components *must* render when they're first added to the component hierarchy by their parent component.</span></span> <span data-ttu-id="b1ab2-105">Si tratta dell'unica volta in cui è necessario eseguire il rendering rigoroso di un componente.</span><span class="sxs-lookup"><span data-stu-id="b1ab2-105">This is the only time that a component strictly must render.</span></span>

<span data-ttu-id="b1ab2-106">I componenti *possono* scegliere di eseguire il rendering in qualsiasi altro momento in base alla logica e alle convenzioni.</span><span class="sxs-lookup"><span data-stu-id="b1ab2-106">Components *may* choose to render at any other time according to their own logic and conventions.</span></span>

## <a name="conventions-for-componentbase"></a><span data-ttu-id="b1ab2-107">Convenzioni per `ComponentBase`</span><span class="sxs-lookup"><span data-stu-id="b1ab2-107">Conventions for `ComponentBase`</span></span>

<span data-ttu-id="b1ab2-108">Per impostazione predefinita, Razor Components ( `.razor` ) eredita dalla <xref:Microsoft.AspNetCore.Components.ComponentBase> classe base, che contiene la logica per attivare il rendering nei momenti seguenti:</span><span class="sxs-lookup"><span data-stu-id="b1ab2-108">By default, Razor components (`.razor`) inherit from the <xref:Microsoft.AspNetCore.Components.ComponentBase> base class, which contains logic to trigger rerendering at the following times:</span></span>

* <span data-ttu-id="b1ab2-109">Dopo l'applicazione di un set di parametri aggiornato da un componente padre.</span><span class="sxs-lookup"><span data-stu-id="b1ab2-109">After applying an updated set of parameters from a parent component.</span></span>
* <span data-ttu-id="b1ab2-110">Dopo aver applicato un valore aggiornato per un parametro di propagazione.</span><span class="sxs-lookup"><span data-stu-id="b1ab2-110">After applying an updated value for a cascading parameter.</span></span>
* <span data-ttu-id="b1ab2-111">Dopo la notifica di un evento e il richiamo di uno dei relativi gestori eventi.</span><span class="sxs-lookup"><span data-stu-id="b1ab2-111">After notification of an event and invoking one of its own event handlers.</span></span>
* <span data-ttu-id="b1ab2-112">Dopo una chiamata al relativo <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> metodo.</span><span class="sxs-lookup"><span data-stu-id="b1ab2-112">After a call to its own <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> method.</span></span>

<span data-ttu-id="b1ab2-113">Componenti ereditati da <xref:Microsoft.AspNetCore.Components.ComponentBase> Skip rerenderings a causa di aggiornamenti dei parametri in presenza di una delle condizioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="b1ab2-113">Components inherited from <xref:Microsoft.AspNetCore.Components.ComponentBase> skip rerenders due to parameter updates if either of the following are true:</span></span>

* <span data-ttu-id="b1ab2-114">Tutti i valori dei parametri sono di tipi primitivi non modificabili noti (ad esempio,,, `int` `string` `DateTime` ) e non sono stati modificati dopo l'impostazione del set di parametri precedente.</span><span class="sxs-lookup"><span data-stu-id="b1ab2-114">All of the parameter values are of known immutable primitive types (for example, `int`, `string`, `DateTime`) and haven't changed since the previous set of parameters were set.</span></span>
* <span data-ttu-id="b1ab2-115">Il metodo del componente <xref:Microsoft.AspNetCore.Components.ComponentBase.ShouldRender%2A> restituisce `false` .</span><span class="sxs-lookup"><span data-stu-id="b1ab2-115">The component's <xref:Microsoft.AspNetCore.Components.ComponentBase.ShouldRender%2A> method returns `false`.</span></span>

<span data-ttu-id="b1ab2-116">Per altre informazioni su <xref:Microsoft.AspNetCore.Components.ComponentBase.ShouldRender%2A>, vedere <xref:blazor/webassembly-performance-best-practices#use-of-shouldrender>.</span><span class="sxs-lookup"><span data-stu-id="b1ab2-116">For more information on <xref:Microsoft.AspNetCore.Components.ComponentBase.ShouldRender%2A>, see <xref:blazor/webassembly-performance-best-practices#use-of-shouldrender>.</span></span>

## <a name="control-the-rendering-flow"></a><span data-ttu-id="b1ab2-117">Controllare il flusso di rendering</span><span class="sxs-lookup"><span data-stu-id="b1ab2-117">Control the rendering flow</span></span>

<span data-ttu-id="b1ab2-118">Nella maggior parte dei casi, <xref:Microsoft.AspNetCore.Components.ComponentBase> le convenzioni generano il subset corretto di rirendering dei componenti dopo che si è verificato un evento.</span><span class="sxs-lookup"><span data-stu-id="b1ab2-118">In most cases, <xref:Microsoft.AspNetCore.Components.ComponentBase> conventions result in the correct subset of component rerenders after an event occurs.</span></span> <span data-ttu-id="b1ab2-119">Gli sviluppatori non sono in genere necessari per fornire la logica manuale per indicare al Framework i componenti di cui eseguire il rendering e quando eseguirne il rendering.</span><span class="sxs-lookup"><span data-stu-id="b1ab2-119">Developers aren't usually required to provide manual logic to tell the framework which components to rerender and when to rerender them.</span></span> <span data-ttu-id="b1ab2-120">L'effetto complessivo delle convenzioni del Framework è che il componente che riceve un evento esegue il rendering di se stesso, che attiva in modo ricorsivo il rendering dei componenti discendenti i cui valori di parametro possono essere stati modificati.</span><span class="sxs-lookup"><span data-stu-id="b1ab2-120">The overall effect of the framework's conventions is that the component receiving an event rerenders itself, which recursively triggers rerendering of descendant components whose parameter values may have changed.</span></span>

<span data-ttu-id="b1ab2-121">Per ulteriori informazioni sulle conseguenze sulle prestazioni delle convenzioni del Framework e su come ottimizzare la gerarchia dei componenti di un'app, vedere <xref:blazor/webassembly-performance-best-practices#optimize-rendering-speed> .</span><span class="sxs-lookup"><span data-stu-id="b1ab2-121">For more information on the performance implications of the framework's conventions and how to optimize an app's component hierarchy, see <xref:blazor/webassembly-performance-best-practices#optimize-rendering-speed>.</span></span>

## <a name="when-to-call-statehaschanged"></a><span data-ttu-id="b1ab2-122">Quando chiamare `StateHasChanged`</span><span class="sxs-lookup"><span data-stu-id="b1ab2-122">When to call `StateHasChanged`</span></span>

<span data-ttu-id="b1ab2-123">Il <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A?displayProperty=nameWithType> metodo consente di attivare un rendering in qualsiasi momento.</span><span class="sxs-lookup"><span data-stu-id="b1ab2-123">The <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A?displayProperty=nameWithType> method allows you to trigger a render at any time.</span></span> <span data-ttu-id="b1ab2-124">Tuttavia, prestare attenzione a non chiamare <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> inutilmente, che è un errore comune, perché impone costi di rendering superflui.</span><span class="sxs-lookup"><span data-stu-id="b1ab2-124">However, be careful not to call <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> unnecessarily, which is a common mistake, because it imposes unnecessary rendering costs.</span></span>

<span data-ttu-id="b1ab2-125">*Non* è necessario chiamare <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> quando:</span><span class="sxs-lookup"><span data-stu-id="b1ab2-125">You should *not* need to call <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> when:</span></span>

* <span data-ttu-id="b1ab2-126">Gestione ordinaria degli eventi, in modo sincrono o asincrono, perché <xref:Microsoft.AspNetCore.Components.ComponentBase> attiva un rendering per la maggior parte dei gestori di eventi di routine.</span><span class="sxs-lookup"><span data-stu-id="b1ab2-126">Routinely handling events, whether synchronously or asynchronously, since <xref:Microsoft.AspNetCore.Components.ComponentBase> triggers a render for most routine event handlers.</span></span>
* <span data-ttu-id="b1ab2-127">Implementazione della logica del ciclo di vita tipica, ad esempio [`OnInitialized`](xref:blazor/components/lifecycle#component-initialization-methods) o [`OnParametersSetAsync`](xref:blazor/components/lifecycle#after-parameters-are-set) , se synchonrously o asincrono, poiché <xref:Microsoft.AspNetCore.Components.ComponentBase> attiva un rendering per eventi del ciclo di vita tipici.</span><span class="sxs-lookup"><span data-stu-id="b1ab2-127">Implementing typical lifecycle logic, such as [`OnInitialized`](xref:blazor/components/lifecycle#component-initialization-methods) or [`OnParametersSetAsync`](xref:blazor/components/lifecycle#after-parameters-are-set), whether synchonrously or asynchronously, since <xref:Microsoft.AspNetCore.Components.ComponentBase> triggers a render for typical lifecycle events.</span></span>

<span data-ttu-id="b1ab2-128">Tuttavia, potrebbe essere utile chiamare <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> nei casi descritti nelle sezioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="b1ab2-128">However, it might make sense to call <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> in the cases described by the following sections:</span></span>

* [<span data-ttu-id="b1ab2-129">Un gestore asincrono prevede più fasi asincrone</span><span class="sxs-lookup"><span data-stu-id="b1ab2-129">An asynchronous handler involves multiple asynchronous phases</span></span>](#an-asynchronous-handler-involves-multiple-asynchronous-phases)
* [<span data-ttu-id="b1ab2-130">Ricezione di una chiamata da un elemento esterno al Blazor sistema di gestione degli eventi e di rendering</span><span class="sxs-lookup"><span data-stu-id="b1ab2-130">Receiving a call from something external to the Blazor rendering and event handling system</span></span>](#receiving-a-call-from-something-external-to-the-blazor-rendering-and-event-handling-system)
* [<span data-ttu-id="b1ab2-131">Per eseguire il rendering di un componente all'esterno del sottoalbero di cui è stato eseguito il rendering da un particolare evento</span><span class="sxs-lookup"><span data-stu-id="b1ab2-131">To render component outside the subtree that is rerendered by a particular event</span></span>](#to-render-a-component-outside-the-subtree-thats-rerendered-by-a-particular-event)

### <a name="an-asynchronous-handler-involves-multiple-asynchronous-phases"></a><span data-ttu-id="b1ab2-132">Un gestore asincrono prevede più fasi asincrone</span><span class="sxs-lookup"><span data-stu-id="b1ab2-132">An asynchronous handler involves multiple asynchronous phases</span></span>

<span data-ttu-id="b1ab2-133">Si consideri il `Counter` componente seguente, che aggiorna il conteggio quattro volte a ogni clic.</span><span class="sxs-lookup"><span data-stu-id="b1ab2-133">Consider the following `Counter` component, which updates the count four times on each click.</span></span>

<span data-ttu-id="b1ab2-134">`Pages/Counter.razor`:</span><span class="sxs-lookup"><span data-stu-id="b1ab2-134">`Pages/Counter.razor`:</span></span>

```razor
@page "/counter"

<p>Current count: @currentCount</p>

<button class="btn btn-primary" @onclick="IncrementCount">Click me</button>

@code {
    private int currentCount = 0;

    private async Task IncrementCount()
    {
        currentCount++;
        // Renders here automatically

        await Task.Delay(1000);
        currentCount++;
        StateHasChanged();

        await Task.Delay(1000);
        currentCount++;
        StateHasChanged();

        await Task.Delay(1000);
        currentCount++;
        // Renders here automatically
    }
}
```

<span data-ttu-id="b1ab2-135">A causa del modo in cui le attività vengono definite in .NET, un ricevitore di un <xref:System.Threading.Tasks.Task> può osservare solo il completamento finale, non gli Stati asincroni intermedi.</span><span class="sxs-lookup"><span data-stu-id="b1ab2-135">Due to the way that tasks are defined in .NET, a receiver of a <xref:System.Threading.Tasks.Task> can only observe its final completion, not intermediate asynchronous states.</span></span> <span data-ttu-id="b1ab2-136">Pertanto, <xref:Microsoft.AspNetCore.Components.ComponentBase> può attivare il rirendering solo quando <xref:System.Threading.Tasks.Task> viene restituito per la prima volta e quando viene <xref:System.Threading.Tasks.Task> completato.</span><span class="sxs-lookup"><span data-stu-id="b1ab2-136">Therefore, <xref:Microsoft.AspNetCore.Components.ComponentBase> can only trigger rerendering when the <xref:System.Threading.Tasks.Task> is first returned and when the <xref:System.Threading.Tasks.Task> finally completes.</span></span> <span data-ttu-id="b1ab2-137">Non è in grado di eseguire il rendering in altri punti intermedi.</span><span class="sxs-lookup"><span data-stu-id="b1ab2-137">It can't know to rerender at other intermediate points.</span></span> <span data-ttu-id="b1ab2-138">Se si desidera eseguire il rendering in punti intermedi, utilizzare <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> .</span><span class="sxs-lookup"><span data-stu-id="b1ab2-138">If you want to rerender at intermediate points, use <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A>.</span></span>

### <a name="receiving-a-call-from-something-external-to-the-blazor-rendering-and-event-handling-system"></a><span data-ttu-id="b1ab2-139">Ricezione di una chiamata da un elemento esterno al Blazor sistema di gestione degli eventi e di rendering</span><span class="sxs-lookup"><span data-stu-id="b1ab2-139">Receiving a call from something external to the Blazor rendering and event handling system</span></span>

<span data-ttu-id="b1ab2-140"><xref:Microsoft.AspNetCore.Components.ComponentBase> conosce solo i propri metodi del ciclo di vita e Blazor gli eventi generati da.</span><span class="sxs-lookup"><span data-stu-id="b1ab2-140"><xref:Microsoft.AspNetCore.Components.ComponentBase> only knows about its own lifecycle methods and Blazor-triggered events.</span></span> <span data-ttu-id="b1ab2-141"><xref:Microsoft.AspNetCore.Components.ComponentBase> non conosce altri eventi che possono verificarsi nel codice.</span><span class="sxs-lookup"><span data-stu-id="b1ab2-141"><xref:Microsoft.AspNetCore.Components.ComponentBase> doesn't know about other events that may occur in your code.</span></span> <span data-ttu-id="b1ab2-142">Ad esempio, tutti gli eventi C# generati da un archivio dati personalizzato sono sconosciuti a Blazor .</span><span class="sxs-lookup"><span data-stu-id="b1ab2-142">For example, any C# events raised by a custom data store are unknown to Blazor.</span></span> <span data-ttu-id="b1ab2-143">Per consentire a tali eventi di attivare il rendering per visualizzare i valori aggiornati nell'interfaccia utente, usare <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> .</span><span class="sxs-lookup"><span data-stu-id="b1ab2-143">In order for such events to trigger rerendering to display updated values in the UI, use <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A>.</span></span>

<span data-ttu-id="b1ab2-144">In un altro caso d'uso, si consideri il `Counter` componente seguente che usa <xref:System.Timers.Timer?displayProperty=fullName> per aggiornare il conteggio a intervalli regolari e chiama <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> per aggiornare l'interfaccia utente.</span><span class="sxs-lookup"><span data-stu-id="b1ab2-144">In another use case, consider the following `Counter` component that uses <xref:System.Timers.Timer?displayProperty=fullName> to update the count at a regular interval and calls <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> to update the UI.</span></span>

<span data-ttu-id="b1ab2-145">`Pages/CounterWithTimerDisposal.razor`:</span><span class="sxs-lookup"><span data-stu-id="b1ab2-145">`Pages/CounterWithTimerDisposal.razor`:</span></span>

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

<span data-ttu-id="b1ab2-146">Nell'esempio precedente:</span><span class="sxs-lookup"><span data-stu-id="b1ab2-146">In the preceding example:</span></span>

* <span data-ttu-id="b1ab2-147">`OnTimerCallback` deve chiamare <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> perché Blazor non è in grado di riconoscere le modifiche apportate a `currentCount` nel callback.</span><span class="sxs-lookup"><span data-stu-id="b1ab2-147">`OnTimerCallback` must call <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> because Blazor isn't aware of the changes to `currentCount` in the callback.</span></span> <span data-ttu-id="b1ab2-148">`OnTimerCallback` viene eseguito al di fuori di qualsiasi Blazor flusso di rendering o di notifica degli eventi gestito.</span><span class="sxs-lookup"><span data-stu-id="b1ab2-148">`OnTimerCallback` runs outside of any Blazor-managed rendering flow or event notification.</span></span>
* <span data-ttu-id="b1ab2-149">Il componente implementa <xref:System.IDisposable> , dove <xref:System.Timers.Timer> viene eliminato quando il Framework chiama il `Dispose` metodo.</span><span class="sxs-lookup"><span data-stu-id="b1ab2-149">The component implements <xref:System.IDisposable>, where the <xref:System.Timers.Timer> is disposed when the framework calls the `Dispose` method.</span></span> <span data-ttu-id="b1ab2-150">Per altre informazioni, vedere <xref:blazor/components/lifecycle#component-disposal-with-idisposable>.</span><span class="sxs-lookup"><span data-stu-id="b1ab2-150">For more information, see <xref:blazor/components/lifecycle#component-disposal-with-idisposable>.</span></span>

<span data-ttu-id="b1ab2-151">Analogamente, poiché il callback viene richiamato all'esterno Blazor del contesto di sincronizzazione, è necessario eseguire il wrapping della logica in <xref:Microsoft.AspNetCore.Components.ComponentBase.InvokeAsync%2A?displayProperty=nameWithType> per spostarlo nel contesto di sincronizzazione del renderer.</span><span class="sxs-lookup"><span data-stu-id="b1ab2-151">Similarly, because the callback is invoked outside Blazor's synchronization context, it's necessary to wrap the logic in <xref:Microsoft.AspNetCore.Components.ComponentBase.InvokeAsync%2A?displayProperty=nameWithType> to move it onto the renderer's synchronization context.</span></span> <span data-ttu-id="b1ab2-152"><xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> può essere chiamato solo dal contesto di sincronizzazione del renderer e, in caso contrario, genera un'eccezione.</span><span class="sxs-lookup"><span data-stu-id="b1ab2-152"><xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> can only be called from the renderer's synchronization context and throws an exception otherwise.</span></span> <span data-ttu-id="b1ab2-153">Equivale al marshalling al thread dell'interfaccia utente in altri Framework dell'interfaccia utente.</span><span class="sxs-lookup"><span data-stu-id="b1ab2-153">This is equivalent to marshalling to the UI thread in other UI frameworks.</span></span>

### <a name="to-render-a-component-outside-the-subtree-thats-rerendered-by-a-particular-event"></a><span data-ttu-id="b1ab2-154">Per eseguire il rendering di un componente all'esterno del sottoalbero di cui è stato eseguito il rendering da un determinato evento</span><span class="sxs-lookup"><span data-stu-id="b1ab2-154">To render a component outside the subtree that's rerendered by a particular event</span></span>

<span data-ttu-id="b1ab2-155">L'interfaccia utente può comportare l'invio di un evento a un componente, la modifica di uno stato e la necessità di eseguire nuovamente il rendering di un componente completamente diverso che non è un discendente di quello che riceve l'evento.</span><span class="sxs-lookup"><span data-stu-id="b1ab2-155">Your UI might involve dispatching an event to one component, changing some state, and needing to rerender a completely different component that isn't a descendant of the one receiving the event.</span></span>

<span data-ttu-id="b1ab2-156">Un modo per gestire questo scenario consiste nel disporre di una classe di *gestione dello stato* , ad esempio un servizio di inserimento delle dipendenze, inserita in più componenti.</span><span class="sxs-lookup"><span data-stu-id="b1ab2-156">One way to deal with this scenario is to have a *state management* class, perhaps as a DI service, injected into multiple components.</span></span> <span data-ttu-id="b1ab2-157">Quando un componente chiama un metodo sul gestore di stato, il gestore di stato può generare un evento C# che viene quindi ricevuto da un componente indipendente.</span><span class="sxs-lookup"><span data-stu-id="b1ab2-157">When one component calls a method on the state manager, the state manager can raise a C# event that's then received by an independent component.</span></span>

<span data-ttu-id="b1ab2-158">Poiché questi eventi C# si trovano all'esterno della Blazor pipeline di rendering, chiamare <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> su altri componenti di cui si vuole eseguire il rendering in risposta agli eventi del gestore dello stato.</span><span class="sxs-lookup"><span data-stu-id="b1ab2-158">Since these C# events are outside the Blazor rendering pipeline, call <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> on other components you wish to render in response to the state manager's events.</span></span>

<span data-ttu-id="b1ab2-159">Questo è simile al caso precedente con <xref:System.Timers.Timer?displayProperty=fullName> nella sezione [precedente](#receiving-a-call-from-something-external-to-the-blazor-rendering-and-event-handling-system) .</span><span class="sxs-lookup"><span data-stu-id="b1ab2-159">This is similar to the earlier case with <xref:System.Timers.Timer?displayProperty=fullName> in the [previous section](#receiving-a-call-from-something-external-to-the-blazor-rendering-and-event-handling-system) section.</span></span> <span data-ttu-id="b1ab2-160">Poiché lo stack di chiamate di esecuzione rimane in genere nel contesto di sincronizzazione del renderer, <xref:Microsoft.AspNetCore.Components.ComponentBase.InvokeAsync%2A> non è in genere necessario.</span><span class="sxs-lookup"><span data-stu-id="b1ab2-160">Since the execution call stack typically remains on the renderer's synchronization context, <xref:Microsoft.AspNetCore.Components.ComponentBase.InvokeAsync%2A> isn't normally required.</span></span> <span data-ttu-id="b1ab2-161"><xref:Microsoft.AspNetCore.Components.ComponentBase.InvokeAsync%2A> è obbligatorio solo se la logica evita il contesto di sincronizzazione, ad esempio chiamando <xref:System.Threading.Tasks.Task.ContinueWith%2A> su un oggetto <xref:System.Threading.Tasks.Task> o in attesa di una <xref:System.Threading.Tasks.Task> con [`ConfigureAwait(false)`](xref:System.Threading.Tasks.Task.ConfigureAwait%2A) .</span><span class="sxs-lookup"><span data-stu-id="b1ab2-161"><xref:Microsoft.AspNetCore.Components.ComponentBase.InvokeAsync%2A> is only required if the logic escapes the synchronization context, such as calling <xref:System.Threading.Tasks.Task.ContinueWith%2A> on a <xref:System.Threading.Tasks.Task> or awaiting a <xref:System.Threading.Tasks.Task> with [`ConfigureAwait(false)`](xref:System.Threading.Tasks.Task.ConfigureAwait%2A).</span></span>
