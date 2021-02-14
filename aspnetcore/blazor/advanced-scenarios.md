---
title: BlazorScenari ASP.NET Core avanzati
author: guardrex
description: Informazioni sugli scenari avanzati in Blazor , tra cui come incorporare la logica RenderTreeBuilder manuale in un'app.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 02/18/2020
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
uid: blazor/advanced-scenarios
ms.openlocfilehash: ba2bf91f3318225383ec9d164c34be9124aa311b
ms.sourcegitcommit: 1166b0ff3828418559510c661e8240e5c5717bb7
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/12/2021
ms.locfileid: "100280844"
---
# <a name="aspnet-core-blazor-advanced-scenarios"></a><span data-ttu-id="2e34d-103">BlazorScenari ASP.NET Core avanzati</span><span class="sxs-lookup"><span data-stu-id="2e34d-103">ASP.NET Core Blazor advanced scenarios</span></span>

## <a name="blazor-server-circuit-handler"></a><span data-ttu-id="2e34d-104">Blazor Server gestore circuito</span><span class="sxs-lookup"><span data-stu-id="2e34d-104">Blazor Server circuit handler</span></span>

<span data-ttu-id="2e34d-105">Blazor Server consente al codice di definire un *gestore di circuito*, che consente l'esecuzione di codice in base alle modifiche apportate allo stato del circuito di un utente.</span><span class="sxs-lookup"><span data-stu-id="2e34d-105">Blazor Server allows code to define a *circuit handler*, which allows running code on changes to the state of a user's circuit.</span></span> <span data-ttu-id="2e34d-106">Un gestore di circuito viene implementato tramite la derivazione da `CircuitHandler` e la registrazione della classe nel contenitore del servizio dell'app.</span><span class="sxs-lookup"><span data-stu-id="2e34d-106">A circuit handler is implemented by deriving from `CircuitHandler` and registering the class in the app's service container.</span></span> <span data-ttu-id="2e34d-107">L'esempio seguente di un gestore di circuito tiene traccia delle SignalR connessioni aperte:</span><span class="sxs-lookup"><span data-stu-id="2e34d-107">The following example of a circuit handler tracks open SignalR connections:</span></span>

```csharp
using System.Collections.Generic;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Components.Server.Circuits;

public class TrackingCircuitHandler : CircuitHandler
{
    private HashSet<Circuit> circuits = new HashSet<Circuit>();

    public override Task OnConnectionUpAsync(Circuit circuit, 
        CancellationToken cancellationToken)
    {
        circuits.Add(circuit);

        return Task.CompletedTask;
    }

    public override Task OnConnectionDownAsync(Circuit circuit, 
        CancellationToken cancellationToken)
    {
        circuits.Remove(circuit);

        return Task.CompletedTask;
    }

    public int ConnectedCircuits => circuits.Count;
}
```

<span data-ttu-id="2e34d-108">I gestori del circuito vengono registrati usando l'inserimento DI dipendenze.</span><span class="sxs-lookup"><span data-stu-id="2e34d-108">Circuit handlers are registered using DI.</span></span> <span data-ttu-id="2e34d-109">Le istanze con ambito vengono create per ogni istanza di un circuito.</span><span class="sxs-lookup"><span data-stu-id="2e34d-109">Scoped instances are created per instance of a circuit.</span></span> <span data-ttu-id="2e34d-110">Utilizzando nell' `TrackingCircuitHandler` esempio precedente, viene creato un servizio singleton perché è necessario tenere traccia dello stato di tutti i circuiti:</span><span class="sxs-lookup"><span data-stu-id="2e34d-110">Using the `TrackingCircuitHandler` in the preceding example, a singleton service is created because the state of all circuits must be tracked:</span></span>

```csharp
public void ConfigureServices(IServiceCollection services)
{
    ...
    services.AddSingleton<CircuitHandler, TrackingCircuitHandler>();
}
```

<span data-ttu-id="2e34d-111">Se i metodi di un gestore di circuito personalizzato generano un'eccezione non gestita, l'eccezione è fatale per il Blazor Server circuito.</span><span class="sxs-lookup"><span data-stu-id="2e34d-111">If a custom circuit handler's methods throw an unhandled exception, the exception is fatal to the Blazor Server circuit.</span></span> <span data-ttu-id="2e34d-112">Per tollerare le eccezioni nel codice di un gestore o i metodi chiamati, eseguire il wrapping del codice in una o più [`try-catch`](/dotnet/csharp/language-reference/keywords/try-catch) istruzioni con gestione e registrazione degli errori.</span><span class="sxs-lookup"><span data-stu-id="2e34d-112">To tolerate exceptions in a handler's code or called methods, wrap the code in one or more [`try-catch`](/dotnet/csharp/language-reference/keywords/try-catch) statements with error handling and logging.</span></span>

<span data-ttu-id="2e34d-113">Quando un circuito termina perché un utente si è disconnesso e il Framework pulisce lo stato del circuito, il Framework Elimina l'ambito di.</span><span class="sxs-lookup"><span data-stu-id="2e34d-113">When a circuit ends because a user has disconnected and the framework is cleaning up the circuit state, the framework disposes of the circuit's DI scope.</span></span> <span data-ttu-id="2e34d-114">Con l'eliminazione dell'ambito vengono eliminati tutti i servizi con ambito DI circuito che implementano <xref:System.IDisposable?displayProperty=fullName> .</span><span class="sxs-lookup"><span data-stu-id="2e34d-114">Disposing the scope disposes any circuit-scoped DI services that implement <xref:System.IDisposable?displayProperty=fullName>.</span></span> <span data-ttu-id="2e34d-115">Se un servizio DI INSERIMENTO DI dipendenze genera un'eccezione non gestita durante l'eliminazione, il Framework registra l'eccezione.</span><span class="sxs-lookup"><span data-stu-id="2e34d-115">If any DI service throws an unhandled exception during disposal, the framework logs the exception.</span></span>

## <a name="manual-rendertreebuilder-logic"></a><span data-ttu-id="2e34d-116">Logica RenderTreeBuilder manuale</span><span class="sxs-lookup"><span data-stu-id="2e34d-116">Manual RenderTreeBuilder logic</span></span>

<span data-ttu-id="2e34d-117"><xref:Microsoft.AspNetCore.Components.Rendering.RenderTreeBuilder> fornisce metodi per la modifica di componenti ed elementi, inclusa la compilazione manuale di componenti nel codice C#.</span><span class="sxs-lookup"><span data-stu-id="2e34d-117"><xref:Microsoft.AspNetCore.Components.Rendering.RenderTreeBuilder> provides methods for manipulating components and elements, including building components manually in C# code.</span></span>

> [!NOTE]
> <span data-ttu-id="2e34d-118">L'utilizzo di <xref:Microsoft.AspNetCore.Components.Rendering.RenderTreeBuilder> per la creazione di componenti è uno scenario avanzato.</span><span class="sxs-lookup"><span data-stu-id="2e34d-118">Use of <xref:Microsoft.AspNetCore.Components.Rendering.RenderTreeBuilder> to create components is an advanced scenario.</span></span> <span data-ttu-id="2e34d-119">Un componente con formato non valido, ad esempio un tag di markup non chiuso, può causare un comportamento indefinito.</span><span class="sxs-lookup"><span data-stu-id="2e34d-119">A malformed component (for example, an unclosed markup tag) can result in undefined behavior.</span></span>

<span data-ttu-id="2e34d-120">Si consideri il `PetDetails` componente seguente, che può essere incorporato manualmente in un altro componente:</span><span class="sxs-lookup"><span data-stu-id="2e34d-120">Consider the following `PetDetails` component, which can be manually built into another component:</span></span>

```razor
<h2>Pet Details Component</h2>

<p>@PetDetailsQuote</p>

@code
{
    [Parameter]
    public string PetDetailsQuote { get; set; }
}
```

<span data-ttu-id="2e34d-121">Nell'esempio seguente il ciclo nel `CreateComponent` metodo genera tre `PetDetails` componenti.</span><span class="sxs-lookup"><span data-stu-id="2e34d-121">In the following example, the loop in the `CreateComponent` method generates three `PetDetails` components.</span></span> <span data-ttu-id="2e34d-122">Nei <xref:Microsoft.AspNetCore.Components.Rendering.RenderTreeBuilder> metodi con un numero di sequenza, i numeri di sequenza sono numeri di riga del codice sorgente.</span><span class="sxs-lookup"><span data-stu-id="2e34d-122">In <xref:Microsoft.AspNetCore.Components.Rendering.RenderTreeBuilder> methods with a sequence number, sequence numbers are source code line numbers.</span></span> <span data-ttu-id="2e34d-123">L' Blazor algoritmo Difference si basa sui numeri di sequenza corrispondenti a righe di codice distinte, non sulle chiamate di chiamata distinti.</span><span class="sxs-lookup"><span data-stu-id="2e34d-123">The Blazor difference algorithm relies on the sequence numbers corresponding to distinct lines of code, not distinct call invocations.</span></span> <span data-ttu-id="2e34d-124">Quando si crea un componente con <xref:Microsoft.AspNetCore.Components.Rendering.RenderTreeBuilder> i metodi, impostare come hardcoded gli argomenti per i numeri di sequenza.</span><span class="sxs-lookup"><span data-stu-id="2e34d-124">When creating a component with <xref:Microsoft.AspNetCore.Components.Rendering.RenderTreeBuilder> methods, hardcode the arguments for sequence numbers.</span></span> <span data-ttu-id="2e34d-125">**L'utilizzo di un calcolo o di un contatore per generare il numero di sequenza può causare un calo delle prestazioni.**</span><span class="sxs-lookup"><span data-stu-id="2e34d-125">**Using a calculation or counter to generate the sequence number can lead to poor performance.**</span></span> <span data-ttu-id="2e34d-126">Per ulteriori informazioni, vedere la sezione [numeri di sequenza correlati ai numeri di riga del codice e non all'ordine di esecuzione](#sequence-numbers-relate-to-code-line-numbers-and-not-execution-order) .</span><span class="sxs-lookup"><span data-stu-id="2e34d-126">For more information, see the [Sequence numbers relate to code line numbers and not execution order](#sequence-numbers-relate-to-code-line-numbers-and-not-execution-order) section.</span></span>

<span data-ttu-id="2e34d-127">`BuiltContent` componente</span><span class="sxs-lookup"><span data-stu-id="2e34d-127">`BuiltContent` component:</span></span>

```razor
@page "/BuiltContent"

<h1>Build a component</h1>

@CustomRender

<button type="button" @onclick="RenderComponent">
    Create three Pet Details components
</button>

@code {
    private RenderFragment CustomRender { get; set; }
    
    private RenderFragment CreateComponent() => builder =>
    {
        for (var i = 0; i < 3; i++) 
        {
            builder.OpenComponent(0, typeof(PetDetails));
            builder.AddAttribute(1, "PetDetailsQuote", "Someone's best friend!");
            builder.CloseComponent();
        }
    };    
    
    private void RenderComponent()
    {
        CustomRender = CreateComponent();
    }
}
```

> [!WARNING]
> <span data-ttu-id="2e34d-128">I tipi in <xref:Microsoft.AspNetCore.Components.RenderTree> consentono l'elaborazione dei *risultati* delle operazioni di rendering.</span><span class="sxs-lookup"><span data-stu-id="2e34d-128">The types in <xref:Microsoft.AspNetCore.Components.RenderTree> allow processing of the *results* of rendering operations.</span></span> <span data-ttu-id="2e34d-129">Questi sono i dettagli interni dell' Blazor implementazione del Framework.</span><span class="sxs-lookup"><span data-stu-id="2e34d-129">These are internal details of the Blazor framework implementation.</span></span> <span data-ttu-id="2e34d-130">Questi tipi devono essere considerati *instabili* e soggetti a modifiche nelle versioni future.</span><span class="sxs-lookup"><span data-stu-id="2e34d-130">These types should be considered *unstable* and subject to change in future releases.</span></span>

### <a name="sequence-numbers-relate-to-code-line-numbers-and-not-execution-order"></a><span data-ttu-id="2e34d-131">I numeri di sequenza sono correlati ai numeri di riga del codice e non all'ordine di esecuzione</span><span class="sxs-lookup"><span data-stu-id="2e34d-131">Sequence numbers relate to code line numbers and not execution order</span></span>

<span data-ttu-id="2e34d-132">Razor i file dei componenti ( `.razor` ) vengono sempre compilati.</span><span class="sxs-lookup"><span data-stu-id="2e34d-132">Razor component files (`.razor`) are always compiled.</span></span> <span data-ttu-id="2e34d-133">La compilazione è un potenziale vantaggio rispetto all'interpretazione del codice perché il passaggio di compilazione può essere usato per inserire informazioni che migliorano le prestazioni dell'app in fase di esecuzione.</span><span class="sxs-lookup"><span data-stu-id="2e34d-133">Compilation is a potential advantage over interpreting code because the compile step can be used to inject information that improves app performance at runtime.</span></span>

<span data-ttu-id="2e34d-134">Un esempio fondamentale di questi miglioramenti riguarda i *numeri di sequenza*.</span><span class="sxs-lookup"><span data-stu-id="2e34d-134">A key example of these improvements involves *sequence numbers*.</span></span> <span data-ttu-id="2e34d-135">I numeri di sequenza indicano al runtime quali output provengono da righe di codice distinte e ordinate.</span><span class="sxs-lookup"><span data-stu-id="2e34d-135">Sequence numbers indicate to the runtime which outputs came from which distinct and ordered lines of code.</span></span> <span data-ttu-id="2e34d-136">Il runtime usa queste informazioni per generare differenze di albero efficienti nel tempo lineare, che è molto più veloce rispetto a quanto normalmente è possibile per un algoritmo diff della struttura ad albero generale.</span><span class="sxs-lookup"><span data-stu-id="2e34d-136">The runtime uses this information to generate efficient tree diffs in linear time, which is far faster than is normally possible for a general tree diff algorithm.</span></span>

<span data-ttu-id="2e34d-137">Si consideri il seguente Razor file Component ( `.razor` ):</span><span class="sxs-lookup"><span data-stu-id="2e34d-137">Consider the following Razor component (`.razor`) file:</span></span>

```razor
@if (someFlag)
{
    <text>First</text>
}

Second
```

<span data-ttu-id="2e34d-138">Il codice precedente viene compilato in un modo simile al seguente:</span><span class="sxs-lookup"><span data-stu-id="2e34d-138">The preceding code compiles to something like the following:</span></span>

```csharp
if (someFlag)
{
    builder.AddContent(0, "First");
}

builder.AddContent(1, "Second");
```

<span data-ttu-id="2e34d-139">Quando il codice viene eseguito per la prima volta, se `someFlag` è `true` , il generatore riceve:</span><span class="sxs-lookup"><span data-stu-id="2e34d-139">When the code executes for the first time, if `someFlag` is `true`, the builder receives:</span></span>

| <span data-ttu-id="2e34d-140">Sequenza</span><span class="sxs-lookup"><span data-stu-id="2e34d-140">Sequence</span></span> | <span data-ttu-id="2e34d-141">Tipo</span><span class="sxs-lookup"><span data-stu-id="2e34d-141">Type</span></span>      | <span data-ttu-id="2e34d-142">Data</span><span class="sxs-lookup"><span data-stu-id="2e34d-142">Data</span></span>   |
| :------: | --------- | :----: |
| <span data-ttu-id="2e34d-143">0</span><span class="sxs-lookup"><span data-stu-id="2e34d-143">0</span></span>        | <span data-ttu-id="2e34d-144">Nodo testo</span><span class="sxs-lookup"><span data-stu-id="2e34d-144">Text node</span></span> | <span data-ttu-id="2e34d-145">First (Primo)</span><span class="sxs-lookup"><span data-stu-id="2e34d-145">First</span></span>  |
| <span data-ttu-id="2e34d-146">1</span><span class="sxs-lookup"><span data-stu-id="2e34d-146">1</span></span>        | <span data-ttu-id="2e34d-147">Nodo testo</span><span class="sxs-lookup"><span data-stu-id="2e34d-147">Text node</span></span> | <span data-ttu-id="2e34d-148">Second</span><span class="sxs-lookup"><span data-stu-id="2e34d-148">Second</span></span> |

<span data-ttu-id="2e34d-149">Si supponga che `someFlag` diventi `false` e che venga eseguito nuovamente il rendering del markup.</span><span class="sxs-lookup"><span data-stu-id="2e34d-149">Imagine that `someFlag` becomes `false`, and the markup is rendered again.</span></span> <span data-ttu-id="2e34d-150">Questa volta, il generatore riceve:</span><span class="sxs-lookup"><span data-stu-id="2e34d-150">This time, the builder receives:</span></span>

| <span data-ttu-id="2e34d-151">Sequenza</span><span class="sxs-lookup"><span data-stu-id="2e34d-151">Sequence</span></span> | <span data-ttu-id="2e34d-152">Tipo</span><span class="sxs-lookup"><span data-stu-id="2e34d-152">Type</span></span>       | <span data-ttu-id="2e34d-153">Data</span><span class="sxs-lookup"><span data-stu-id="2e34d-153">Data</span></span>   |
| :------: | ---------- | :----: |
| <span data-ttu-id="2e34d-154">1</span><span class="sxs-lookup"><span data-stu-id="2e34d-154">1</span></span>        | <span data-ttu-id="2e34d-155">Nodo testo</span><span class="sxs-lookup"><span data-stu-id="2e34d-155">Text node</span></span>  | <span data-ttu-id="2e34d-156">Second</span><span class="sxs-lookup"><span data-stu-id="2e34d-156">Second</span></span> |

<span data-ttu-id="2e34d-157">Quando il runtime esegue una diff, rileva che l'elemento in sequenza `0` è stato rimosso, quindi genera lo *script di modifica* semplice seguente:</span><span class="sxs-lookup"><span data-stu-id="2e34d-157">When the runtime performs a diff, it sees that the item at sequence `0` was removed, so it generates the following trivial *edit script*:</span></span>

* <span data-ttu-id="2e34d-158">Rimuovere il primo nodo di testo.</span><span class="sxs-lookup"><span data-stu-id="2e34d-158">Remove the first text node.</span></span>

### <a name="the-problem-with-generating-sequence-numbers-programmatically"></a><span data-ttu-id="2e34d-159">Il problema della generazione di numeri di sequenza a livello di codice</span><span class="sxs-lookup"><span data-stu-id="2e34d-159">The problem with generating sequence numbers programmatically</span></span>

<span data-ttu-id="2e34d-160">Si supponga invece che sia stata scritta la logica del generatore di albero di rendering seguente:</span><span class="sxs-lookup"><span data-stu-id="2e34d-160">Imagine instead that you wrote the following render tree builder logic:</span></span>

```csharp
var seq = 0;

if (someFlag)
{
    builder.AddContent(seq++, "First");
}

builder.AddContent(seq++, "Second");
```

<span data-ttu-id="2e34d-161">A questo punto, il primo output è:</span><span class="sxs-lookup"><span data-stu-id="2e34d-161">Now, the first output is:</span></span>

| <span data-ttu-id="2e34d-162">Sequenza</span><span class="sxs-lookup"><span data-stu-id="2e34d-162">Sequence</span></span> | <span data-ttu-id="2e34d-163">Tipo</span><span class="sxs-lookup"><span data-stu-id="2e34d-163">Type</span></span>      | <span data-ttu-id="2e34d-164">Data</span><span class="sxs-lookup"><span data-stu-id="2e34d-164">Data</span></span>   |
| :------: | --------- | :----: |
| <span data-ttu-id="2e34d-165">0</span><span class="sxs-lookup"><span data-stu-id="2e34d-165">0</span></span>        | <span data-ttu-id="2e34d-166">Nodo testo</span><span class="sxs-lookup"><span data-stu-id="2e34d-166">Text node</span></span> | <span data-ttu-id="2e34d-167">First (Primo)</span><span class="sxs-lookup"><span data-stu-id="2e34d-167">First</span></span>  |
| <span data-ttu-id="2e34d-168">1</span><span class="sxs-lookup"><span data-stu-id="2e34d-168">1</span></span>        | <span data-ttu-id="2e34d-169">Nodo testo</span><span class="sxs-lookup"><span data-stu-id="2e34d-169">Text node</span></span> | <span data-ttu-id="2e34d-170">Second</span><span class="sxs-lookup"><span data-stu-id="2e34d-170">Second</span></span> |

<span data-ttu-id="2e34d-171">Questo risultato è identico al caso precedente, pertanto non esistono problemi negativi.</span><span class="sxs-lookup"><span data-stu-id="2e34d-171">This outcome is identical to the prior case, so no negative issues exist.</span></span> <span data-ttu-id="2e34d-172">`someFlag` si trova `false` nel secondo rendering e l'output è:</span><span class="sxs-lookup"><span data-stu-id="2e34d-172">`someFlag` is `false` on the second rendering, and the output is:</span></span>

| <span data-ttu-id="2e34d-173">Sequenza</span><span class="sxs-lookup"><span data-stu-id="2e34d-173">Sequence</span></span> | <span data-ttu-id="2e34d-174">Tipo</span><span class="sxs-lookup"><span data-stu-id="2e34d-174">Type</span></span>      | <span data-ttu-id="2e34d-175">Data</span><span class="sxs-lookup"><span data-stu-id="2e34d-175">Data</span></span>   |
| :------: | --------- | ------ |
| <span data-ttu-id="2e34d-176">0</span><span class="sxs-lookup"><span data-stu-id="2e34d-176">0</span></span>        | <span data-ttu-id="2e34d-177">Nodo testo</span><span class="sxs-lookup"><span data-stu-id="2e34d-177">Text node</span></span> | <span data-ttu-id="2e34d-178">Second</span><span class="sxs-lookup"><span data-stu-id="2e34d-178">Second</span></span> |

<span data-ttu-id="2e34d-179">Questa volta, l'algoritmo Diff rileva che si sono verificate *due* modifiche e l'algoritmo genera lo script di modifica seguente:</span><span class="sxs-lookup"><span data-stu-id="2e34d-179">This time, the diff algorithm sees that *two* changes have occurred, and the algorithm generates the following edit script:</span></span>

* <span data-ttu-id="2e34d-180">Modificare il valore del primo nodo di testo in `Second` .</span><span class="sxs-lookup"><span data-stu-id="2e34d-180">Change the value of the first text node to `Second`.</span></span>
* <span data-ttu-id="2e34d-181">Rimuovere il secondo nodo di testo.</span><span class="sxs-lookup"><span data-stu-id="2e34d-181">Remove the second text node.</span></span>

<span data-ttu-id="2e34d-182">La generazione dei numeri di sequenza ha perso tutte le informazioni utili sul punto in cui i `if/else` rami e i cicli erano presenti nel codice originale.</span><span class="sxs-lookup"><span data-stu-id="2e34d-182">Generating the sequence numbers has lost all the useful information about where the `if/else` branches and loops were present in the original code.</span></span> <span data-ttu-id="2e34d-183">In questo modo si ottiene una differenza **due volte più a lungo** .</span><span class="sxs-lookup"><span data-stu-id="2e34d-183">This results in a diff **twice as long** as before.</span></span>

<span data-ttu-id="2e34d-184">Questo è un esempio semplice.</span><span class="sxs-lookup"><span data-stu-id="2e34d-184">This is a trivial example.</span></span> <span data-ttu-id="2e34d-185">Nei casi più realistici con strutture complesse e profondamente annidate e soprattutto con i cicli, il costo delle prestazioni è in genere più elevato.</span><span class="sxs-lookup"><span data-stu-id="2e34d-185">In more realistic cases with complex and deeply nested structures, and especially with loops, the performance cost is usually higher.</span></span> <span data-ttu-id="2e34d-186">Invece di identificare immediatamente i blocchi o i rami del ciclo che sono stati inseriti o rimossi, l'algoritmo Diff deve ripresentarsi in modo approfondito negli alberi di rendering.</span><span class="sxs-lookup"><span data-stu-id="2e34d-186">Instead of immediately identifying which loop blocks or branches have been inserted or removed, the diff algorithm has to recurse deeply into the render trees.</span></span> <span data-ttu-id="2e34d-187">Ciò comporta in genere la necessità di compilare script di modifica più lunghi perché l'algoritmo Diff viene informato in modo non più approfondito sulle relazioni tra le vecchie e le nuove strutture.</span><span class="sxs-lookup"><span data-stu-id="2e34d-187">This usually results in having to build longer edit scripts because the diff algorithm is misinformed about how the old and new structures relate to each other.</span></span>

### <a name="guidance-and-conclusions"></a><span data-ttu-id="2e34d-188">Linee guida e conclusioni</span><span class="sxs-lookup"><span data-stu-id="2e34d-188">Guidance and conclusions</span></span>

* <span data-ttu-id="2e34d-189">Le prestazioni dell'app soffrono se i numeri di sequenza vengono generati dinamicamente.</span><span class="sxs-lookup"><span data-stu-id="2e34d-189">App performance suffers if sequence numbers are generated dynamically.</span></span>
* <span data-ttu-id="2e34d-190">Il Framework non è in grado di creare automaticamente i propri numeri di sequenza in fase di esecuzione perché le informazioni necessarie non esistono a meno che non vengano acquisite in fase di compilazione.</span><span class="sxs-lookup"><span data-stu-id="2e34d-190">The framework can't create its own sequence numbers automatically at runtime because the necessary information doesn't exist unless it's captured at compile time.</span></span>
* <span data-ttu-id="2e34d-191">Non scrivere blocchi lunghi di logica implementata manualmente <xref:Microsoft.AspNetCore.Components.Rendering.RenderTreeBuilder> .</span><span class="sxs-lookup"><span data-stu-id="2e34d-191">Don't write long blocks of manually-implemented <xref:Microsoft.AspNetCore.Components.Rendering.RenderTreeBuilder> logic.</span></span> <span data-ttu-id="2e34d-192">Preferire i `.razor` file e consentire al compilatore di gestire i numeri di sequenza.</span><span class="sxs-lookup"><span data-stu-id="2e34d-192">Prefer `.razor` files and allow the compiler to deal with the sequence numbers.</span></span> <span data-ttu-id="2e34d-193">Se non si è in grado di evitare <xref:Microsoft.AspNetCore.Components.Rendering.RenderTreeBuilder> la logica manuale, suddividere i blocchi di codice lunghi in parti più piccole racchiuse tra le <xref:Microsoft.AspNetCore.Components.Rendering.RenderTreeBuilder.OpenRegion%2A> / <xref:Microsoft.AspNetCore.Components.Rendering.RenderTreeBuilder.CloseRegion%2A> chiamate.</span><span class="sxs-lookup"><span data-stu-id="2e34d-193">If you're unable to avoid manual <xref:Microsoft.AspNetCore.Components.Rendering.RenderTreeBuilder> logic, split long blocks of code into smaller pieces wrapped in <xref:Microsoft.AspNetCore.Components.Rendering.RenderTreeBuilder.OpenRegion%2A>/<xref:Microsoft.AspNetCore.Components.Rendering.RenderTreeBuilder.CloseRegion%2A> calls.</span></span> <span data-ttu-id="2e34d-194">Ogni area ha il proprio spazio separato dei numeri di sequenza, quindi è possibile riavviare da zero (o qualsiasi altro numero arbitrario) all'interno di ogni area.</span><span class="sxs-lookup"><span data-stu-id="2e34d-194">Each region has its own separate space of sequence numbers, so you can restart from zero (or any other arbitrary number) inside each region.</span></span>
* <span data-ttu-id="2e34d-195">Se i numeri di sequenza sono hardcoded, l'algoritmo Diff richiede solo che i numeri di sequenza aumentino nel valore.</span><span class="sxs-lookup"><span data-stu-id="2e34d-195">If sequence numbers are hardcoded, the diff algorithm only requires that sequence numbers increase in value.</span></span> <span data-ttu-id="2e34d-196">Il valore iniziale e i gap sono irrilevanti.</span><span class="sxs-lookup"><span data-stu-id="2e34d-196">The initial value and gaps are irrelevant.</span></span> <span data-ttu-id="2e34d-197">Una delle opzioni legittime consiste nell'usare il numero di riga del codice come numero di sequenza oppure iniziare da zero e aumentare di uno o di centinaia (o qualsiasi intervallo preferito).</span><span class="sxs-lookup"><span data-stu-id="2e34d-197">One legitimate option is to use the code line number as the sequence number, or start from zero and increase by ones or hundreds (or any preferred interval).</span></span> 
* <span data-ttu-id="2e34d-198">Blazor USA i numeri di sequenza, mentre altri Framework dell'interfaccia utente con differenze tra gli alberi non li usano.</span><span class="sxs-lookup"><span data-stu-id="2e34d-198">Blazor uses sequence numbers, while other tree-diffing UI frameworks don't use them.</span></span> <span data-ttu-id="2e34d-199">La diffing è molto più veloce quando si usano i numeri Blazor di sequenza e presenta il vantaggio di un passaggio di compilazione che gestisce automaticamente i numeri di sequenza per gli sviluppatori che creano `.razor` file.</span><span class="sxs-lookup"><span data-stu-id="2e34d-199">Diffing is far faster when sequence numbers are used, and Blazor has the advantage of a compile step that deals with sequence numbers automatically for developers authoring `.razor` files.</span></span>
