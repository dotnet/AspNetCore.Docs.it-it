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
ms.openlocfilehash: 27701d175c86cdf4b74a9e6332b30b4d55806650
ms.sourcegitcommit: 04ad9cd26fcaa8bd11e261d3661f375f5f343cdc
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/10/2021
ms.locfileid: "100107038"
---
# <a name="aspnet-core-blazor-component-rendering"></a>BlazorRendering componente ASP.NET Core

Di [Steve Sanderson](https://github.com/SteveSandersonMS)

I componenti *devono* essere sottoposte a rendering quando vengono aggiunti per la prima volta alla gerarchia dei componenti tramite il componente padre. Si tratta dell'unica volta in cui è necessario eseguire il rendering rigoroso di un componente.

I componenti *possono* scegliere di eseguire il rendering in qualsiasi altro momento in base alla logica e alle convenzioni.

## <a name="conventions-for-componentbase"></a>Convenzioni per `ComponentBase`

Per impostazione predefinita, Razor Components ( `.razor` ) eredita dalla <xref:Microsoft.AspNetCore.Components.ComponentBase> classe base, che contiene la logica per attivare il rendering nei momenti seguenti:

* Dopo l'applicazione di un set di parametri aggiornato da un componente padre.
* Dopo aver applicato un valore aggiornato per un parametro di propagazione.
* Dopo la notifica di un evento e il richiamo di uno dei relativi gestori eventi.
* Dopo una chiamata al relativo <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> metodo.

Componenti ereditati da <xref:Microsoft.AspNetCore.Components.ComponentBase> Skip rerenderings a causa di aggiornamenti dei parametri in presenza di una delle condizioni seguenti:

* Tutti i valori dei parametri sono di tipi primitivi non modificabili noti (ad esempio,,, `int` `string` `DateTime` ) e non sono stati modificati dopo l'impostazione del set di parametri precedente.
* Il metodo del componente <xref:Microsoft.AspNetCore.Components.ComponentBase.ShouldRender%2A> restituisce `false` .

Per altre informazioni su <xref:Microsoft.AspNetCore.Components.ComponentBase.ShouldRender%2A>, vedere <xref:blazor/webassembly-performance-best-practices#use-of-shouldrender>.

## <a name="control-the-rendering-flow"></a>Controllare il flusso di rendering

Nella maggior parte dei casi, <xref:Microsoft.AspNetCore.Components.ComponentBase> le convenzioni generano il subset corretto di rirendering dei componenti dopo che si è verificato un evento. Gli sviluppatori non sono in genere necessari per fornire la logica manuale per indicare al Framework i componenti di cui eseguire il rendering e quando eseguirne il rendering. L'effetto complessivo delle convenzioni del Framework è che il componente che riceve un evento esegue il rendering di se stesso, che attiva in modo ricorsivo il rendering dei componenti discendenti i cui valori di parametro possono essere stati modificati.

Per ulteriori informazioni sulle conseguenze sulle prestazioni delle convenzioni del Framework e su come ottimizzare la gerarchia dei componenti di un'app, vedere <xref:blazor/webassembly-performance-best-practices#optimize-rendering-speed> .

## <a name="when-to-call-statehaschanged"></a>Quando chiamare `StateHasChanged`

Il <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A?displayProperty=nameWithType> metodo consente di attivare un rendering in qualsiasi momento. Tuttavia, prestare attenzione a non chiamare <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> inutilmente, che è un errore comune, perché impone costi di rendering superflui.

*Non* è necessario chiamare <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> quando:

* Gestione ordinaria degli eventi, in modo sincrono o asincrono, perché <xref:Microsoft.AspNetCore.Components.ComponentBase> attiva un rendering per la maggior parte dei gestori di eventi di routine.
* Implementazione della logica del ciclo di vita tipica, ad esempio [`OnInitialized`](xref:blazor/components/lifecycle#component-initialization-methods) o [`OnParametersSetAsync`](xref:blazor/components/lifecycle#after-parameters-are-set) , se synchonrously o asincrono, poiché <xref:Microsoft.AspNetCore.Components.ComponentBase> attiva un rendering per eventi del ciclo di vita tipici.

Tuttavia, potrebbe essere utile chiamare <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> nei casi descritti nelle sezioni seguenti:

* [Un gestore asincrono prevede più fasi asincrone](#an-asynchronous-handler-involves-multiple-asynchronous-phases)
* [Ricezione di una chiamata da un elemento esterno al Blazor sistema di gestione degli eventi e di rendering](#receiving-a-call-from-something-external-to-the-blazor-rendering-and-event-handling-system)
* [Per eseguire il rendering di un componente all'esterno del sottoalbero di cui è stato eseguito il rendering da un particolare evento](#to-render-a-component-outside-the-subtree-thats-rerendered-by-a-particular-event)

### <a name="an-asynchronous-handler-involves-multiple-asynchronous-phases"></a>Un gestore asincrono prevede più fasi asincrone

Si consideri il `Counter` componente seguente, che aggiorna il conteggio quattro volte a ogni clic.

`Pages/Counter.razor`:

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

A causa del modo in cui le attività vengono definite in .NET, un ricevitore di un <xref:System.Threading.Tasks.Task> può osservare solo il completamento finale, non gli Stati asincroni intermedi. Pertanto, <xref:Microsoft.AspNetCore.Components.ComponentBase> può attivare il rirendering solo quando <xref:System.Threading.Tasks.Task> viene restituito per la prima volta e quando viene <xref:System.Threading.Tasks.Task> completato. Non è in grado di eseguire il rendering in altri punti intermedi. Se si desidera eseguire il rendering in punti intermedi, utilizzare <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> .

### <a name="receiving-a-call-from-something-external-to-the-blazor-rendering-and-event-handling-system"></a>Ricezione di una chiamata da un elemento esterno al Blazor sistema di gestione degli eventi e di rendering

<xref:Microsoft.AspNetCore.Components.ComponentBase> conosce solo i propri metodi del ciclo di vita e Blazor gli eventi generati da. <xref:Microsoft.AspNetCore.Components.ComponentBase> non conosce altri eventi che possono verificarsi nel codice. Ad esempio, tutti gli eventi C# generati da un archivio dati personalizzato sono sconosciuti a Blazor . Per consentire a tali eventi di attivare il rendering per visualizzare i valori aggiornati nell'interfaccia utente, usare <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> .

In un altro caso d'uso, si consideri il `Counter` componente seguente che usa <xref:System.Timers.Timer?displayProperty=fullName> per aggiornare il conteggio a intervalli regolari e chiama <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> per aggiornare l'interfaccia utente.

`Pages/CounterWithTimerDisposal.razor`:

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

Nell'esempio precedente:

* `OnTimerCallback` deve chiamare <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> perché Blazor non è in grado di riconoscere le modifiche apportate a `currentCount` nel callback. `OnTimerCallback` viene eseguito al di fuori di qualsiasi Blazor flusso di rendering o di notifica degli eventi gestito.
* Il componente implementa <xref:System.IDisposable> , dove <xref:System.Timers.Timer> viene eliminato quando il Framework chiama il `Dispose` metodo. Per altre informazioni, vedere <xref:blazor/components/lifecycle#component-disposal-with-idisposable>.

Analogamente, poiché il callback viene richiamato all'esterno Blazor del contesto di sincronizzazione, è necessario eseguire il wrapping della logica in <xref:Microsoft.AspNetCore.Components.ComponentBase.InvokeAsync%2A?displayProperty=nameWithType> per spostarlo nel contesto di sincronizzazione del renderer. <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> può essere chiamato solo dal contesto di sincronizzazione del renderer e, in caso contrario, genera un'eccezione. Equivale al marshalling al thread dell'interfaccia utente in altri Framework dell'interfaccia utente.

### <a name="to-render-a-component-outside-the-subtree-thats-rerendered-by-a-particular-event"></a>Per eseguire il rendering di un componente all'esterno del sottoalbero di cui è stato eseguito il rendering da un determinato evento

L'interfaccia utente può comportare l'invio di un evento a un componente, la modifica di uno stato e la necessità di eseguire nuovamente il rendering di un componente completamente diverso che non è un discendente di quello che riceve l'evento.

Un modo per gestire questo scenario consiste nel disporre di una classe di *gestione dello stato* , ad esempio un servizio di inserimento delle dipendenze, inserita in più componenti. Quando un componente chiama un metodo sul gestore di stato, il gestore di stato può generare un evento C# che viene quindi ricevuto da un componente indipendente.

Poiché questi eventi C# si trovano all'esterno della Blazor pipeline di rendering, chiamare <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> su altri componenti di cui si vuole eseguire il rendering in risposta agli eventi del gestore dello stato.

Questo è simile al caso precedente con <xref:System.Timers.Timer?displayProperty=fullName> nella sezione [precedente](#receiving-a-call-from-something-external-to-the-blazor-rendering-and-event-handling-system) . Poiché lo stack di chiamate di esecuzione rimane in genere nel contesto di sincronizzazione del renderer, <xref:Microsoft.AspNetCore.Components.ComponentBase.InvokeAsync%2A> non è in genere necessario. <xref:Microsoft.AspNetCore.Components.ComponentBase.InvokeAsync%2A> è obbligatorio solo se la logica evita il contesto di sincronizzazione, ad esempio chiamando <xref:System.Threading.Tasks.Task.ContinueWith%2A> su un oggetto <xref:System.Threading.Tasks.Task> o in attesa di una <xref:System.Threading.Tasks.Task> con [`ConfigureAwait(false)`](xref:System.Threading.Tasks.Task.ConfigureAwait%2A) .
