---
title: Blazor WebAssemblyProcedure consigliate per le prestazioni ASP.NET Core
author: pranavkm
description: Suggerimenti per migliorare le prestazioni in ASP.NET Core Blazor WebAssembly app ed evitare problemi comuni relativi alle prestazioni.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc, devx-track-js
ms.date: 10/09/2020
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
uid: blazor/webassembly-performance-best-practices
ms.openlocfilehash: 64eb69ba597b1e48489733275ac3bed96c6c3241
ms.sourcegitcommit: ef8d8c79993a6608bf597ad036edcf30b231843f
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/09/2021
ms.locfileid: "99975235"
---
# <a name="aspnet-core-blazor-webassembly-performance-best-practices"></a><span data-ttu-id="b0f58-103">Blazor WebAssemblyProcedure consigliate per le prestazioni ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="b0f58-103">ASP.NET Core Blazor WebAssembly performance best practices</span></span>

<span data-ttu-id="b0f58-104">Di [](https://github.com/pranavkm) [Manuel e Steve Sanderson](https://github.com/SteveSandersonMS)</span><span class="sxs-lookup"><span data-stu-id="b0f58-104">By [Pranav Krishnamoorthy](https://github.com/pranavkm) and [Steve Sanderson](https://github.com/SteveSandersonMS)</span></span>

<span data-ttu-id="b0f58-105">Blazor WebAssembly viene accuratamente progettato e ottimizzato per garantire prestazioni elevate in scenari di interfaccia utente dell'applicazione più realistici.</span><span class="sxs-lookup"><span data-stu-id="b0f58-105">Blazor WebAssembly is carefully designed and optimized to enable high performance in most realistic application UI scenarios.</span></span> <span data-ttu-id="b0f58-106">Tuttavia, la produzione dei migliori risultati dipende dagli sviluppatori che usano i modelli e le funzionalità corrette.</span><span class="sxs-lookup"><span data-stu-id="b0f58-106">However, producing the best results depends on developers using the right patterns and features.</span></span> <span data-ttu-id="b0f58-107">Prendere in considerazione gli aspetti seguenti:</span><span class="sxs-lookup"><span data-stu-id="b0f58-107">Consider the following aspects:</span></span>

* <span data-ttu-id="b0f58-108">**Velocità effettiva di runtime**: il codice .NET viene eseguito in un interprete nel runtime di webassembly, quindi la velocità effettiva della CPU è limitata.</span><span class="sxs-lookup"><span data-stu-id="b0f58-108">**Runtime throughput**: The .NET code runs on an interpreter within the WebAssembly runtime, so CPU throughput is limited.</span></span> <span data-ttu-id="b0f58-109">In scenari complessi, l'app trae vantaggio dall' [ottimizzazione della velocità di rendering](#optimize-rendering-speed).</span><span class="sxs-lookup"><span data-stu-id="b0f58-109">In demanding scenarios, the app benefits from [optimizing rendering speed](#optimize-rendering-speed).</span></span>
* <span data-ttu-id="b0f58-110">**Tempo di avvio**: l'app trasferisce un Runtime .NET al browser, quindi è importante usare le funzionalità che [riducono al minimo le dimensioni del download dell'applicazione](#minimize-app-download-size).</span><span class="sxs-lookup"><span data-stu-id="b0f58-110">**Startup time**: The app transfers a .NET runtime to the browser, so it's important to use features that [minimize the application download size](#minimize-app-download-size).</span></span>

## <a name="optimize-rendering-speed"></a><span data-ttu-id="b0f58-111">Ottimizzare la velocità di rendering</span><span class="sxs-lookup"><span data-stu-id="b0f58-111">Optimize rendering speed</span></span>

<span data-ttu-id="b0f58-112">Le sezioni seguenti forniscono consigli per ridurre il carico di lavoro di rendering e migliorare la velocità di risposta dell'interfaccia utente.</span><span class="sxs-lookup"><span data-stu-id="b0f58-112">The following sections provide recommendations to minimize rendering workload and improve UI responsiveness.</span></span> <span data-ttu-id="b0f58-113">Seguendo questo Consiglio, è possibile eseguire facilmente un *miglioramento di dieci volte o superiore* nella velocità di rendering dell'interfaccia utente.</span><span class="sxs-lookup"><span data-stu-id="b0f58-113">Following this advice could easily make a *ten-fold or higher improvement* in UI rendering speeds.</span></span>

### <a name="avoid-unnecessary-rendering-of-component-subtrees"></a><span data-ttu-id="b0f58-114">Evitare il rendering superfluo di sottoalberi dei componenti</span><span class="sxs-lookup"><span data-stu-id="b0f58-114">Avoid unnecessary rendering of component subtrees</span></span>

<span data-ttu-id="b0f58-115">In fase di esecuzione, i componenti esistono come una gerarchia.</span><span class="sxs-lookup"><span data-stu-id="b0f58-115">At runtime, components exist as a hierarchy.</span></span> <span data-ttu-id="b0f58-116">Un componente radice contiene componenti figlio.</span><span class="sxs-lookup"><span data-stu-id="b0f58-116">A root component has child components.</span></span> <span data-ttu-id="b0f58-117">A sua volta, i figli della radice hanno i propri componenti figlio e così via.</span><span class="sxs-lookup"><span data-stu-id="b0f58-117">In turn, the root's children have their own child components, and so on.</span></span> <span data-ttu-id="b0f58-118">Quando si verifica un evento, ad esempio un utente che seleziona un pulsante, questo è il modo Blazor in cui decide i componenti di cui eseguire il rendering:</span><span class="sxs-lookup"><span data-stu-id="b0f58-118">When an event occurs, such as a user selecting a button, this is how Blazor decides which components to rerender:</span></span>

1. <span data-ttu-id="b0f58-119">L'evento stesso viene inviato a qualsiasi componente di cui è stato eseguito il rendering del gestore dell'evento.</span><span class="sxs-lookup"><span data-stu-id="b0f58-119">The event itself is dispatched to whichever component rendered the event's handler.</span></span> <span data-ttu-id="b0f58-120">Dopo l'esecuzione del gestore dell'evento, viene eseguito il rendering del componente.</span><span class="sxs-lookup"><span data-stu-id="b0f58-120">After executing the event handler, that component is rerendered.</span></span>
1. <span data-ttu-id="b0f58-121">Ogni volta che viene eseguito il rendering di un componente, fornisce una nuova copia dei valori di parametro a ognuno dei relativi componenti figlio.</span><span class="sxs-lookup"><span data-stu-id="b0f58-121">Whenever any component is rerendered, it supplies a new copy of the parameter values to each of its child components.</span></span>
1. <span data-ttu-id="b0f58-122">Quando si riceve un nuovo set di valori di parametro, ogni componente sceglie se eseguire nuovamente il rendering.</span><span class="sxs-lookup"><span data-stu-id="b0f58-122">When receiving a new set of parameter values, each component chooses whether to rerender.</span></span> <span data-ttu-id="b0f58-123">Per impostazione predefinita, i componenti eseguono il rendering se è possibile che i valori dei parametri siano stati modificati, ad esempio se sono oggetti modificabili.</span><span class="sxs-lookup"><span data-stu-id="b0f58-123">By default, components rerender if the parameter values may have changed (for example, if they are mutable objects).</span></span>

<span data-ttu-id="b0f58-124">Gli ultimi due passaggi di questa sequenza continuano in modo ricorsivo la gerarchia dei componenti.</span><span class="sxs-lookup"><span data-stu-id="b0f58-124">The last two steps of this sequence continue recursively down the component hierarchy.</span></span> <span data-ttu-id="b0f58-125">In molti casi, viene eseguito il rendering dell'intero sottoalbero.</span><span class="sxs-lookup"><span data-stu-id="b0f58-125">In many cases, the entire subtree is rerendered.</span></span> <span data-ttu-id="b0f58-126">Questo significa che gli eventi destinati a componenti di alto livello possono causare costosi processi di rirendering perché è necessario eseguire il rendering di tutti gli elementi al di sotto di tale punto.</span><span class="sxs-lookup"><span data-stu-id="b0f58-126">This means that events targeting high-level components can cause expensive rerendering processes because everything below that point must be rerendered.</span></span>

<span data-ttu-id="b0f58-127">Se si desidera interrompere il processo e impedire il rendering della ricorsione in un sottoalbero particolare, è possibile eseguire una delle operazioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="b0f58-127">If you want to interrupt this process and prevent rendering recursion into a particular subtree, then you can either:</span></span>

* <span data-ttu-id="b0f58-128">Verificare che tutti i parametri di un determinato componente siano tipi non modificabili primitivi (ad esempio,,,, `string` `int` `bool` `DateTime` e altri).</span><span class="sxs-lookup"><span data-stu-id="b0f58-128">Ensure that all parameters to a certain component are of primitive immutable types (for example, `string`, `int`, `bool`, `DateTime`, and others).</span></span> <span data-ttu-id="b0f58-129">La logica predefinita per il rilevamento delle modifiche ignora automaticamente il rendering se nessuno di questi valori di parametro è stato modificato.</span><span class="sxs-lookup"><span data-stu-id="b0f58-129">The built-in logic for detecting changes automatically skips rerendering if none of these parameter values have changed.</span></span> <span data-ttu-id="b0f58-130">Se si esegue il rendering di un componente figlio con `<Customer CustomerId="@item.CustomerId" />` , dove `CustomerId` è un `int` valore, non viene nuovamente eseguito il rendering tranne quando viene `item.CustomerId` modificato.</span><span class="sxs-lookup"><span data-stu-id="b0f58-130">If you render a child component with `<Customer CustomerId="@item.CustomerId" />`, where `CustomerId` is an `int` value, then it isn't rerendered except when `item.CustomerId` changes.</span></span>
* <span data-ttu-id="b0f58-131">Se è necessario accettare valori di parametro non primitivi, ad esempio tipi di modelli personalizzati, callback di eventi o <xref:Microsoft.AspNetCore.Components.RenderFragment> valori, è possibile eseguire l'override <xref:Microsoft.AspNetCore.Components.ComponentBase.ShouldRender%2A> di per controllare la decisione relativa all'eventuale rendering, descritto nella sezione [utilizzo `ShouldRender` di](#use-of-shouldrender) .</span><span class="sxs-lookup"><span data-stu-id="b0f58-131">If you need to accept nonprimitive parameter values, such as custom model types, event callbacks, or <xref:Microsoft.AspNetCore.Components.RenderFragment> values, then you can override <xref:Microsoft.AspNetCore.Components.ComponentBase.ShouldRender%2A> to control the decision about whether to render, which is described in the [Use of `ShouldRender`](#use-of-shouldrender) section.</span></span>

<span data-ttu-id="b0f58-132">Ignorando il rendering di interi sottoalberi, potrebbe essere possibile rimuovere la maggior parte del costo di rendering quando si verifica un evento.</span><span class="sxs-lookup"><span data-stu-id="b0f58-132">By skipping rerendering of whole subtrees, you may be able to remove the vast majority of the rendering cost when an event occurs.</span></span>

<span data-ttu-id="b0f58-133">È possibile scomporre i componenti figlio in modo specifico, in modo che sia possibile ignorare il rendering della parte dell'interfaccia utente.</span><span class="sxs-lookup"><span data-stu-id="b0f58-133">You may wish to factor out child components specifically so that you can skip rerendering that part of the UI.</span></span> <span data-ttu-id="b0f58-134">Si tratta di un modo valido per ridurre il costo di rendering di un componente padre.</span><span class="sxs-lookup"><span data-stu-id="b0f58-134">This is a valid way to reduce the rendering cost of a parent component.</span></span>

#### <a name="use-of-shouldrender"></a><span data-ttu-id="b0f58-135">Utilizzo di `ShouldRender`</span><span class="sxs-lookup"><span data-stu-id="b0f58-135">Use of `ShouldRender`</span></span>

<span data-ttu-id="b0f58-136">Se la creazione di un componente solo interfaccia utente che non cambia mai dopo il rendering iniziale (indipendentemente dai valori dei parametri), configurare <xref:Microsoft.AspNetCore.Components.ComponentBase.ShouldRender%2A> per restituire `false` :</span><span class="sxs-lookup"><span data-stu-id="b0f58-136">If authoring a UI-only component that never changes after the initial render (regardless of any parameter values), configure <xref:Microsoft.AspNetCore.Components.ComponentBase.ShouldRender%2A> to return `false`:</span></span>

```razor
@code {
    protected override bool ShouldRender() => false;
}
```

<span data-ttu-id="b0f58-137">Se il componente richiede solo il rirendering quando i valori dei parametri cambiano in modo particolare, è possibile usare campi privati per tenere traccia delle informazioni necessarie per rilevare le modifiche.</span><span class="sxs-lookup"><span data-stu-id="b0f58-137">If the component only requires rerendering when its parameter values mutate in particular ways, then you can use private fields to track the necessary information to detect changes.</span></span> <span data-ttu-id="b0f58-138">Nell'esempio seguente, `shouldRender` si basa sul controllo di qualsiasi tipo di modifica o mutazione che dovrebbe richiedere un rirendering.</span><span class="sxs-lookup"><span data-stu-id="b0f58-138">In the following example, `shouldRender` is based on checking for any kind of change or mutation that should prompt a rerender.</span></span> <span data-ttu-id="b0f58-139">`prevOutboundFlightId` e `prevInboundFlightId` tenere traccia delle informazioni per il prossimo aggiornamento potenziale:</span><span class="sxs-lookup"><span data-stu-id="b0f58-139">`prevOutboundFlightId` and `prevInboundFlightId` track information for the next potential update:</span></span>

```razor
@code {
    [Parameter]
    public FlightInfo OutboundFlight { get; set; }
    
    [Parameter]
    public FlightInfo InboundFlight { get; set; }

    private int prevOutboundFlightId;
    private int prevInboundFlightId;
    private bool shouldRender;

    protected override void OnParametersSet()
    {
        shouldRender = OutboundFlight.FlightId != prevOutboundFlightId
            || InboundFlight.FlightId != prevInboundFlightId;

        prevOutboundFlightId = OutboundFlight.FlightId;
        prevInboundFlightId = InboundFlight.FlightId;
    }

    protected override bool ShouldRender() => shouldRender;

    // Note that 
}
```

<span data-ttu-id="b0f58-140">Nel codice precedente, un gestore eventi può anche impostare `shouldRender` su in `true` modo che il componente venga nuovamente eseguito dopo l'evento.</span><span class="sxs-lookup"><span data-stu-id="b0f58-140">In the preceding code, an event handler may also set `shouldRender` to `true` so that the component is rerendered after the event.</span></span>

<span data-ttu-id="b0f58-141">Per la maggior parte dei componenti, questo livello di controllo manuale non è necessario.</span><span class="sxs-lookup"><span data-stu-id="b0f58-141">For most components, this level of manual control isn't necessary.</span></span> <span data-ttu-id="b0f58-142">È consigliabile ignorare il rendering di sottoalberi se tali sottoalberi sono particolarmente costosi per il rendering e causano un ritardo dell'interfaccia utente.</span><span class="sxs-lookup"><span data-stu-id="b0f58-142">You should only be concerned about skipping rendering subtrees if those subtrees are particularly expensive to render and are causing UI lag.</span></span>

<span data-ttu-id="b0f58-143">Per altre informazioni, vedere <xref:blazor/components/lifecycle>.</span><span class="sxs-lookup"><span data-stu-id="b0f58-143">For more information, see <xref:blazor/components/lifecycle>.</span></span>

::: moniker range=">= aspnetcore-5.0"

### <a name="virtualization"></a><span data-ttu-id="b0f58-144">Virtualizzazione</span><span class="sxs-lookup"><span data-stu-id="b0f58-144">Virtualization</span></span>

<span data-ttu-id="b0f58-145">Quando si esegue il rendering di grandi quantità di interfaccia utente all'interno di un ciclo, ad esempio un elenco o una griglia con migliaia di voci, la quantità totale di operazioni di rendering può causare un ritardo nel rendering dell'interfaccia utente e quindi un'esperienza utente insufficiente.</span><span class="sxs-lookup"><span data-stu-id="b0f58-145">When rendering large amounts of UI within a loop, for example a list or grid with thousands of entries, the sheer quantity of rendering operations can lead to a lag in UI rendering and thus a poor user experience.</span></span> <span data-ttu-id="b0f58-146">Dato che l'utente può visualizzare solo un numero ridotto di elementi in una sola volta senza scorrere, sembra superfluo dedicare molto tempo al rendering degli elementi che attualmente non sono visibili.</span><span class="sxs-lookup"><span data-stu-id="b0f58-146">Given that the user can only see a small number of elements at once without scrolling, it seems wasteful to spend so much time rendering elements that aren't currently visible.</span></span>

<span data-ttu-id="b0f58-147">Per risolvere questo problema, Blazor fornisce il `Virtualize` componente che crea l'aspetto e i comportamenti di scorrimento di un elenco di grandi dimensioni arbitrario, ma esegue solo il rendering degli elementi dell'elenco che si trovano all'interno del viewport di scorrimento corrente.</span><span class="sxs-lookup"><span data-stu-id="b0f58-147">To address this, Blazor provides the `Virtualize` component that creates the appearance and scroll behaviors of an arbitrarily-large list but only renders the list items that are within the current scroll viewport.</span></span> <span data-ttu-id="b0f58-148">Questo significa, ad esempio, che l'app può avere un elenco con 100.000 voci, ma paga solo il costo di rendering di 20 elementi visibili in un momento qualsiasi.</span><span class="sxs-lookup"><span data-stu-id="b0f58-148">For example, this means that the app can have a list with 100,000 entries but only pay the rendering cost of 20 items that are visible at any one time.</span></span> <span data-ttu-id="b0f58-149">L'uso del `Virtualize` componente può aumentare le prestazioni dell'interfaccia utente in base agli ordini di grandezza.</span><span class="sxs-lookup"><span data-stu-id="b0f58-149">Use of the `Virtualize` component can scale up UI performance by orders of magnitude.</span></span>

<span data-ttu-id="b0f58-150">Per altre informazioni, vedere <xref:blazor/components/virtualization>.</span><span class="sxs-lookup"><span data-stu-id="b0f58-150">For more information, see <xref:blazor/components/virtualization>.</span></span>

::: moniker-end

### <a name="create-lightweight-optimized-components"></a><span data-ttu-id="b0f58-151">Creazione di componenti semplici e ottimizzati</span><span class="sxs-lookup"><span data-stu-id="b0f58-151">Create lightweight, optimized components</span></span>

<span data-ttu-id="b0f58-152">La maggior parte dei Blazor componenti non richiede attività di ottimizzazione aggressive.</span><span class="sxs-lookup"><span data-stu-id="b0f58-152">Most Blazor components don't require aggressive optimization efforts.</span></span> <span data-ttu-id="b0f58-153">Questo è dovuto al fatto che la maggior parte dei componenti non si ripete spesso nell'interfaccia utente e non viene eseguito il rendering a frequenza elevata.</span><span class="sxs-lookup"><span data-stu-id="b0f58-153">This is because most components don't often repeat in the UI and don't rerender at high frequency.</span></span> <span data-ttu-id="b0f58-154">Ad esempio, `@page` componenti e componenti che rappresentano parti dell'interfaccia utente di alto livello come finestre di dialogo o moduli, molto probabilmente vengono visualizzati solo uno alla volta e ne viene eseguito il rendering solo in risposta a un movimento dell'utente.</span><span class="sxs-lookup"><span data-stu-id="b0f58-154">For example, `@page` components and components representing high-level UI pieces such as dialogs or forms, most likely appear only one at a time and only rerender in response to a user gesture.</span></span> <span data-ttu-id="b0f58-155">Questi componenti non creano un carico di lavoro di rendering elevato, quindi è possibile usare liberamente qualsiasi combinazione di funzionalità del Framework che si vuole senza preoccuparsi delle prestazioni di rendering.</span><span class="sxs-lookup"><span data-stu-id="b0f58-155">These components don't create a high rendering workload, so you can freely use any combination of framework features you want without worrying much about rendering performance.</span></span>

<span data-ttu-id="b0f58-156">Tuttavia, esistono anche scenari comuni in cui si compilano componenti che devono essere ripetuti su larga scala.</span><span class="sxs-lookup"><span data-stu-id="b0f58-156">However, there are also common scenarios where you build components that need to be repeated at scale.</span></span> <span data-ttu-id="b0f58-157">Ad esempio:</span><span class="sxs-lookup"><span data-stu-id="b0f58-157">For example:</span></span>

* <span data-ttu-id="b0f58-158">I moduli annidati di grandi dimensioni possono avere centinaia di input singoli, etichette e altri elementi.</span><span class="sxs-lookup"><span data-stu-id="b0f58-158">Large nested forms may have hundreds of individual inputs, labels, and other elements.</span></span>
* <span data-ttu-id="b0f58-159">Le griglie possono contenere migliaia di celle.</span><span class="sxs-lookup"><span data-stu-id="b0f58-159">Grids may have thousands of cells.</span></span>
* <span data-ttu-id="b0f58-160">I grafici a dispersione possono contenere milioni di punti dati.</span><span class="sxs-lookup"><span data-stu-id="b0f58-160">Scatter plots may have millions of data points.</span></span>

<span data-ttu-id="b0f58-161">Se si modellano le singole unità come istanze di componenti separate, ne saranno presenti così tante che le prestazioni di rendering diventano critiche.</span><span class="sxs-lookup"><span data-stu-id="b0f58-161">If modelling each unit as separate component instances, there will be so many of them that their rendering performance does become critical.</span></span> <span data-ttu-id="b0f58-162">In questa sezione vengono fornite indicazioni su come rendere tali componenti leggeri, in modo che l'interfaccia utente rimanga veloce e reattiva.</span><span class="sxs-lookup"><span data-stu-id="b0f58-162">This section provides advice on making such components lightweight so that the UI remains fast and responsive.</span></span>

#### <a name="avoid-thousands-of-component-instances"></a><span data-ttu-id="b0f58-163">Evitare migliaia di istanze di componenti</span><span class="sxs-lookup"><span data-stu-id="b0f58-163">Avoid thousands of component instances</span></span>

<span data-ttu-id="b0f58-164">Ogni componente è un'isola separata che può essere sottoposta a rendering indipendentemente dagli elementi padre e figlio.</span><span class="sxs-lookup"><span data-stu-id="b0f58-164">Each component is a separate island that can render independently of its parents and children.</span></span> <span data-ttu-id="b0f58-165">Scegliendo come suddividere l'interfaccia utente in una gerarchia di componenti, si sta assumendo il controllo della granularità del rendering dell'interfaccia utente.</span><span class="sxs-lookup"><span data-stu-id="b0f58-165">By choosing how to split up the UI into a hierarchy of components, you are taking control over the granularity of UI rendering.</span></span> <span data-ttu-id="b0f58-166">Questo può essere positivo o negativo per le prestazioni.</span><span class="sxs-lookup"><span data-stu-id="b0f58-166">This can be either good or bad for performance.</span></span>

* <span data-ttu-id="b0f58-167">Suddividendo l'interfaccia utente in più componenti, è possibile avere parti più piccole del rendering dell'interfaccia utente quando si verificano gli eventi.</span><span class="sxs-lookup"><span data-stu-id="b0f58-167">By splitting the UI into more components, you can have smaller portions of the UI rerender when events occur.</span></span> <span data-ttu-id="b0f58-168">Ad esempio, quando un utente fa clic su un pulsante in una riga di tabella, è possibile che venga eseguito solo il rirendering di una singola riga anziché l'intera pagina o tabella.</span><span class="sxs-lookup"><span data-stu-id="b0f58-168">For example when a user clicks a button in a table row, you may be able to have only that single row rerender instead of the whole page or table.</span></span>
* <span data-ttu-id="b0f58-169">Tuttavia, ogni componente aggiuntivo comporta un sovraccarico di memoria e CPU aggiuntivo per gestire lo stato indipendente e il ciclo di vita di rendering.</span><span class="sxs-lookup"><span data-stu-id="b0f58-169">However, each extra component involves some extra memory and CPU overhead to deal with its independent state and rendering lifecycle.</span></span>

<span data-ttu-id="b0f58-170">Quando si ottimizzano le prestazioni di Blazor WebAssembly in .NET 5, è stato misurato un sovraccarico di rendering di circa 0,06 ms per ogni istanza del componente.</span><span class="sxs-lookup"><span data-stu-id="b0f58-170">When tuning the performance of Blazor WebAssembly on .NET 5, we measured a rendering overhead of around 0.06 ms per component instance.</span></span> <span data-ttu-id="b0f58-171">Si basa su un componente semplice che accetta tre parametri in esecuzione su un portatile tipico.</span><span class="sxs-lookup"><span data-stu-id="b0f58-171">This is based on a simple component that accepts three parameters running on a typical laptop.</span></span> <span data-ttu-id="b0f58-172">Internamente, l'overhead è in gran parte dovuto al recupero dello stato per componente dai dizionari e al passaggio e alla ricezione di parametri.</span><span class="sxs-lookup"><span data-stu-id="b0f58-172">Internally, the overhead is largely due to retrieving per-component state from dictionaries and passing and receiving parameters.</span></span> <span data-ttu-id="b0f58-173">Per moltiplicazione, è possibile notare che l'aggiunta di 2.000 istanze di componenti aggiuntivi aggiungerà 0,12 secondi al tempo di rendering e l'interfaccia utente inizierà a rallentare gli utenti.</span><span class="sxs-lookup"><span data-stu-id="b0f58-173">By multiplication, you can see that adding 2,000 extra component instances would add 0.12 seconds to the rendering time and the UI would begin feeling slow to users.</span></span>

<span data-ttu-id="b0f58-174">È possibile rendere i componenti più leggeri, in modo da poterli avere più, ma spesso la tecnica più potente non deve avere così tanti componenti.</span><span class="sxs-lookup"><span data-stu-id="b0f58-174">It's possible to make components more lightweight so that you can have more of them, but often the more powerful technique is not to have so many components.</span></span> <span data-ttu-id="b0f58-175">Le sezioni seguenti descrivono due approcci.</span><span class="sxs-lookup"><span data-stu-id="b0f58-175">The following sections describe two approaches.</span></span>

##### <a name="inline-child-components-into-their-parents"></a><span data-ttu-id="b0f58-176">Componenti figlio inline nei relativi elementi padre</span><span class="sxs-lookup"><span data-stu-id="b0f58-176">Inline child components into their parents</span></span>

<span data-ttu-id="b0f58-177">Si consideri il componente seguente che esegue il rendering di una sequenza di componenti figlio:</span><span class="sxs-lookup"><span data-stu-id="b0f58-177">Consider the following component that renders a sequence of child components:</span></span>

```razor
<div class="chat">
    @foreach (var message in messages)
    {
        <ChatMessageDisplay Message="@message" />
    }
</div>
```

<span data-ttu-id="b0f58-178">Per il codice di esempio precedente, il `<ChatMessageDisplay>` componente viene definito in un file `ChatMessageDisplay.razor` contenente:</span><span class="sxs-lookup"><span data-stu-id="b0f58-178">For the preceding example code, the `<ChatMessageDisplay>` component is defined in a file `ChatMessageDisplay.razor` containing:</span></span>

```razor
<div class="chat-message">
    <span class="author">@Message.Author</span>
    <span class="text">@Message.Text</span>
</div>

@code {
    [Parameter]
    public ChatMessage Message { get; set; }
}
```

<span data-ttu-id="b0f58-179">L'esempio precedente funziona correttamente e viene eseguito correttamente a condizione che migliaia di messaggi non vengano visualizzati contemporaneamente.</span><span class="sxs-lookup"><span data-stu-id="b0f58-179">The preceding example works fine and performs well as long as thousands of messages aren't shown at once.</span></span> <span data-ttu-id="b0f58-180">Per mostrare migliaia di messaggi contemporaneamente, provare a *non* scomporre il `ChatMessageDisplay` componente separato.</span><span class="sxs-lookup"><span data-stu-id="b0f58-180">To show thousands of messages at once, consider *not* factoring out the separate `ChatMessageDisplay` component.</span></span> <span data-ttu-id="b0f58-181">Inline invece il rendering direttamente nell'elemento padre:</span><span class="sxs-lookup"><span data-stu-id="b0f58-181">Instead, inline the rendering directly into the parent:</span></span>

```razor
<div class="chat">
    @foreach (var message in messages)
    {
        <div class="chat-message">
            <span class="author">@message.Author</span>
            <span class="text">@message.Text</span>
        </div>
    }
</div>
```

<span data-ttu-id="b0f58-182">In questo modo si evita l'overhead per componente per il rendering di un numero così elevato di componenti figlio, a scapito del fatto che non è possibile eseguire nuovamente il rendering di ognuno di essi in modo indipendente.</span><span class="sxs-lookup"><span data-stu-id="b0f58-182">This avoids the per-component overhead of rendering so many child components at the cost of not being able to rerender each of them independently.</span></span>

##### <a name="define-reusable-renderfragments-in-code"></a><span data-ttu-id="b0f58-183">Definire riutilizzabile `RenderFragments` nel codice</span><span class="sxs-lookup"><span data-stu-id="b0f58-183">Define reusable `RenderFragments` in code</span></span>

<span data-ttu-id="b0f58-184">È possibile scomporre i componenti figlio esclusivamente come un modo per riutilizzare la logica di rendering.</span><span class="sxs-lookup"><span data-stu-id="b0f58-184">You may be factoring out child components purely as a way of reusing rendering logic.</span></span> <span data-ttu-id="b0f58-185">In tal caso, è comunque possibile riutilizzare la logica di rendering senza dichiarare i componenti effettivi.</span><span class="sxs-lookup"><span data-stu-id="b0f58-185">If that's the case, it's still possible to reuse rendering logic without declaring actual components.</span></span> <span data-ttu-id="b0f58-186">Nel blocco di un componente è `@code` possibile definire un <xref:Microsoft.AspNetCore.Components.RenderFragment> che genera l'interfaccia utente e può essere chiamato da qualsiasi posizione:</span><span class="sxs-lookup"><span data-stu-id="b0f58-186">In any component's `@code` block, you can define a <xref:Microsoft.AspNetCore.Components.RenderFragment> that emits UI and can be called from anywhere:</span></span>

```razor
<h1>Hello, world!</h1>

@RenderWelcomeInfo

@code {
    private RenderFragment RenderWelcomeInfo = __builder =>
    {
        <div>
            <p>Welcome to your new app!</p>

            <SurveyPrompt Title="How is Blazor working for you?" />
        </div>
    };
}
```

<span data-ttu-id="b0f58-187">Come demonstated nell'esempio precedente, i componenti possono emettere markup dal codice all'interno del `@code` blocco e al di fuori di esso.</span><span class="sxs-lookup"><span data-stu-id="b0f58-187">As demonstated in the preceding example, components can emit markup from code within their `@code` block and outside it.</span></span> <span data-ttu-id="b0f58-188">Questo approccio definisce un <xref:Microsoft.AspNetCore.Components.RenderFragment> delegato di cui è possibile eseguire il rendering all'interno dell'output di rendering normale del componente, facoltativamente in più posizioni.</span><span class="sxs-lookup"><span data-stu-id="b0f58-188">This approach defines a <xref:Microsoft.AspNetCore.Components.RenderFragment> delegate that you can render inside the component's normal render output, optionally in multiple places.</span></span> <span data-ttu-id="b0f58-189">È necessario che il delegato accetti un parametro denominato `__builder` di tipo <xref:Microsoft.AspNetCore.Components.Rendering.RenderTreeBuilder> in modo che il Razor compilatore possa produrre istruzioni per il rendering.</span><span class="sxs-lookup"><span data-stu-id="b0f58-189">It's necessary for the delegate to accept a parameter called `__builder` of type <xref:Microsoft.AspNetCore.Components.Rendering.RenderTreeBuilder> so that the Razor compiler can produce rendering instructions for it.</span></span>

<span data-ttu-id="b0f58-190">Se si desidera rendere questo oggetto riutilizzabile in più componenti, è consigliabile dichiararlo come `public static` membro:</span><span class="sxs-lookup"><span data-stu-id="b0f58-190">If you want to make this reusable across multiple components, consider declaring it as a `public static` member:</span></span>

```razor
public static RenderFragment SayHello = __builder =>
{
    <h1>Hello!</h1>
};
```

<span data-ttu-id="b0f58-191">Questo ora può essere richiamato da un componente non correlato.</span><span class="sxs-lookup"><span data-stu-id="b0f58-191">This could now be invoked from an unrelated component.</span></span> <span data-ttu-id="b0f58-192">Questa tecnica è utile per la creazione di librerie di frammenti di markup riutilizzabili che eseguono il rendering senza alcun sovraccarico per componente.</span><span class="sxs-lookup"><span data-stu-id="b0f58-192">This technique is useful for building libraries of reusable markup snippets that render without any per-component overhead.</span></span>

<span data-ttu-id="b0f58-193"><xref:Microsoft.AspNetCore.Components.RenderFragment> i delegati possono anche accettare parametri.</span><span class="sxs-lookup"><span data-stu-id="b0f58-193"><xref:Microsoft.AspNetCore.Components.RenderFragment> delegates can also accept parameters.</span></span> <span data-ttu-id="b0f58-194">Per creare l'equivalente del `ChatMessageDisplay` componente dall'esempio precedente:</span><span class="sxs-lookup"><span data-stu-id="b0f58-194">To create the equivalent of the `ChatMessageDisplay` component from the earlier example:</span></span>

```razor
<div class="chat">
    @foreach (var message in messages)
    {
        @ChatMessageDisplay(message)
    }
</div>

@code {
    private RenderFragment<ChatMessage> ChatMessageDisplay = message => __builder =>
    {
        <div class="chat-message">
            <span class="author">@message.Author</span>
            <span class="text">@message.Text</span>
        </div>
    };
}
```

<span data-ttu-id="b0f58-195">Questo approccio offre il vantaggio di riutilizzare la logica di rendering senza overhead per componente.</span><span class="sxs-lookup"><span data-stu-id="b0f58-195">This approach provides the benefit of reusing rendering logic without per-component overhead.</span></span> <span data-ttu-id="b0f58-196">Tuttavia, non è in grado di aggiornare il sottoalbero dell'interfaccia utente in modo indipendente, né di ignorare il rendering del sottoalbero dell'interfaccia utente quando viene eseguito il rendering dell'elemento padre, dal momento che non esiste alcun limite per i componenti.</span><span class="sxs-lookup"><span data-stu-id="b0f58-196">However, it doesn't have the benefit of being able to refresh its subtree of the UI independently, nor does it have the ability to skip rendering that subtree of the UI when its parent renders, since there's no component boundary.</span></span>

<span data-ttu-id="b0f58-197">Per un campo, un metodo o una proprietà non statica a cui non è possibile fare riferimento da un inizializzatore di campo, ad esempio `TitleTemplate` nell'esempio seguente, usare una proprietà anziché un campo per <xref:Microsoft.AspNetCore.Components.RenderFragment> :</span><span class="sxs-lookup"><span data-stu-id="b0f58-197">For a non-static field, method, or property that can't be referenced by a field initializer, such as `TitleTemplate` in the following example, use a property instead of a field for the <xref:Microsoft.AspNetCore.Components.RenderFragment>:</span></span>

```csharp
protected RenderFragment DisplayTitle => __builder =>
{
    <div>
        @TitleTemplate
    </div>   
};
```

#### <a name="dont-receive-too-many-parameters"></a><span data-ttu-id="b0f58-198">Non ricevere troppi parametri</span><span class="sxs-lookup"><span data-stu-id="b0f58-198">Don't receive too many parameters</span></span>

<span data-ttu-id="b0f58-199">Se un componente si ripete molto spesso, ad esempio centinaia o migliaia di volte, tenere presente che l'overhead associato al passaggio e alla ricezione di ogni parametro viene compilato.</span><span class="sxs-lookup"><span data-stu-id="b0f58-199">If a component repeats extremely often, for example hundreds or thousands of times, then bear in mind that the overhead of passing and receiving each parameter builds up.</span></span>

<span data-ttu-id="b0f58-200">È raro che troppi parametri limitino notevolmente le prestazioni, ma può essere un fattore.</span><span class="sxs-lookup"><span data-stu-id="b0f58-200">It's rare that too many parameters severely restricts performance, but it can be a factor.</span></span> <span data-ttu-id="b0f58-201">Per un `<TableCell>` componente che esegue il rendering di 1.000 volte all'interno di una griglia, ogni parametro aggiuntivo passato a esso potrebbe aggiungere circa 15 ms al costo totale di rendering.</span><span class="sxs-lookup"><span data-stu-id="b0f58-201">For a `<TableCell>` component that renders 1,000 times within a grid, each extra parameter passed to it could add around 15 ms to the total rendering cost.</span></span> <span data-ttu-id="b0f58-202">Se ogni cella accettava 10 parametri, il passaggio del parametro richiede circa 150 ms per ogni componente e, di conseguenza, 150.000 ms (150 secondi) e, di conseguenza, un'interfaccia utente ritardata.</span><span class="sxs-lookup"><span data-stu-id="b0f58-202">If each cell accepted 10 parameters, parameter passing takes around 150 ms per component render and  thus perhaps 150,000 ms (150 seconds) and on its own cause a laggy UI.</span></span>

<span data-ttu-id="b0f58-203">Per ridurre questo carico, è possibile raggruppare più parametri tramite classi personalizzate.</span><span class="sxs-lookup"><span data-stu-id="b0f58-203">To reduce this load, you could bundle together multiple parameters via custom classes.</span></span> <span data-ttu-id="b0f58-204">È ad esempio possibile che un `<TableCell>` componente accetti:</span><span class="sxs-lookup"><span data-stu-id="b0f58-204">For example, a `<TableCell>` component might accept:</span></span>

```razor
@typeparam TItem

...

@code {
    [Parameter]
    public TItem Data { get; set; }
    
    [Parameter]
    public GridOptions Options { get; set; }
}
```

<span data-ttu-id="b0f58-205">Nell'esempio precedente, `Data` è diverso per ogni cella, ma `Options` è comune in tutti gli elementi.</span><span class="sxs-lookup"><span data-stu-id="b0f58-205">In the preceding example, `Data` is different for every cell, but `Options` is common across all of them.</span></span> <span data-ttu-id="b0f58-206">Naturalmente, potrebbe trattarsi di un miglioramento per non avere un `<TableCell>` componente e inline la relativa logica nel componente padre.</span><span class="sxs-lookup"><span data-stu-id="b0f58-206">Of course, it might be an improvement not to have a `<TableCell>` component and instead inline its logic into the parent component.</span></span>

#### <a name="ensure-cascading-parameters-are-fixed"></a><span data-ttu-id="b0f58-207">Verificare che i parametri a catena siano corretti</span><span class="sxs-lookup"><span data-stu-id="b0f58-207">Ensure cascading parameters are fixed</span></span>

<span data-ttu-id="b0f58-208">Il `<CascadingValue>` componente ha un parametro facoltativo denominato `IsFixed` .</span><span class="sxs-lookup"><span data-stu-id="b0f58-208">The `<CascadingValue>` component has an optional parameter called `IsFixed`.</span></span>

* <span data-ttu-id="b0f58-209">Se il `IsFixed` valore è `false` (impostazione predefinita), ogni destinatario del valore a cascata imposta una sottoscrizione per la ricezione delle notifiche di modifica.</span><span class="sxs-lookup"><span data-stu-id="b0f58-209">If the `IsFixed` value is `false` (the default), then every recipient of the cascaded value sets up a subscription to receive change notifications.</span></span> <span data-ttu-id="b0f58-210">In questo caso, ognuno `[CascadingParameter]` è **sostanzialmente più costoso** di un normale `[Parameter]` a causa del rilevamento della sottoscrizione.</span><span class="sxs-lookup"><span data-stu-id="b0f58-210">In this case, each `[CascadingParameter]` is **substantially more expensive** than a regular `[Parameter]` due to the subscription tracking.</span></span>
* <span data-ttu-id="b0f58-211">Se il `IsFixed` valore è `true` (ad esempio, `<CascadingValue Value="@someValue" IsFixed="true">` ), destinatari riceverà il valore iniziale ma *non* configurarà alcuna sottoscrizione per la ricezione degli aggiornamenti.</span><span class="sxs-lookup"><span data-stu-id="b0f58-211">If the `IsFixed` value is `true` (for example, `<CascadingValue Value="@someValue" IsFixed="true">`), then receipients receive the initial value but do *not* set up any subscription to receive updates.</span></span> <span data-ttu-id="b0f58-212">In questo caso, ogni `[CascadingParameter]` è leggero e **non è più costoso** di un normale `[Parameter]` .</span><span class="sxs-lookup"><span data-stu-id="b0f58-212">In this case, each `[CascadingParameter]` is lightweight and **no more expensive** than a regular `[Parameter]`.</span></span>

<span data-ttu-id="b0f58-213">Quindi, laddove possibile, è consigliabile usare `IsFixed="true"` i valori a cascata.</span><span class="sxs-lookup"><span data-stu-id="b0f58-213">So wherever possible, you should use `IsFixed="true"` on cascaded values.</span></span> <span data-ttu-id="b0f58-214">Questa operazione può essere eseguita ogni volta che il valore fornito non cambia nel tempo.</span><span class="sxs-lookup"><span data-stu-id="b0f58-214">You can do this whenever the value being supplied doesn't change over time.</span></span> <span data-ttu-id="b0f58-215">Nel modello comune in cui un componente passa `this` come valore a catena, è necessario usare `IsFixed="true"` :</span><span class="sxs-lookup"><span data-stu-id="b0f58-215">In the common pattern where a component passes `this` as a cascaded value, you should use `IsFixed="true"`:</span></span>

```razor
<CascadingValue Value="this" IsFixed="true">
    <SomeOtherComponents>
</CascadingValue>
```

<span data-ttu-id="b0f58-216">Questo fa una grande differenza se è presente un numero elevato di altri componenti che ricevono il valore a cascata.</span><span class="sxs-lookup"><span data-stu-id="b0f58-216">This makes a huge difference if there are a large number of other components that receive the cascaded value.</span></span> <span data-ttu-id="b0f58-217">Per altre informazioni, vedere <xref:blazor/components/cascading-values-and-parameters>.</span><span class="sxs-lookup"><span data-stu-id="b0f58-217">For more information, see <xref:blazor/components/cascading-values-and-parameters>.</span></span>

#### <a name="avoid-attribute-splatting-with-captureunmatchedvalues"></a><span data-ttu-id="b0f58-218">Evitare l'attributo splatting con `CaptureUnmatchedValues`</span><span class="sxs-lookup"><span data-stu-id="b0f58-218">Avoid attribute splatting with `CaptureUnmatchedValues`</span></span>

<span data-ttu-id="b0f58-219">I componenti possono scegliere di ricevere i valori di parametro "senza corrispondenza" utilizzando il <xref:Microsoft.AspNetCore.Components.ParameterAttribute.CaptureUnmatchedValues> flag:</span><span class="sxs-lookup"><span data-stu-id="b0f58-219">Components can elect to receive "unmatched" parameter values using the <xref:Microsoft.AspNetCore.Components.ParameterAttribute.CaptureUnmatchedValues> flag:</span></span>

```razor
<div @attributes="OtherAttributes">...</div>

@code {
    [Parameter(CaptureUnmatchedValues = true)]
    public IDictionary<string, object> OtherAttributes { get; set; }
}
```

<span data-ttu-id="b0f58-220">Questo approccio consente di passare gli attributi aggiuntivi arbitrari all'elemento.</span><span class="sxs-lookup"><span data-stu-id="b0f58-220">This approach allows passing through arbitrary additional attributes to the element.</span></span> <span data-ttu-id="b0f58-221">Tuttavia, è anche piuttosto costoso perché il renderer deve:</span><span class="sxs-lookup"><span data-stu-id="b0f58-221">However, it is also quite expensive because the renderer must:</span></span>

* <span data-ttu-id="b0f58-222">Corrisponde a tutti i parametri forniti rispetto al set di parametri noti per compilare un dizionario.</span><span class="sxs-lookup"><span data-stu-id="b0f58-222">Match all of the supplied parameters against the set of known parameters to build a dictionary.</span></span>
* <span data-ttu-id="b0f58-223">Tenere traccia della sovrascrittura di più copie dello stesso attributo.</span><span class="sxs-lookup"><span data-stu-id="b0f58-223">Keep track of how multiple copies of the same attribute overwrite each other.</span></span>

<span data-ttu-id="b0f58-224"><xref:Microsoft.AspNetCore.Components.ParameterAttribute.CaptureUnmatchedValues>È possibile utilizzare su componenti non critici per le prestazioni, ad esempio quelli che non vengono ripetuti spesso.</span><span class="sxs-lookup"><span data-stu-id="b0f58-224">Feel free to use <xref:Microsoft.AspNetCore.Components.ParameterAttribute.CaptureUnmatchedValues> on non-performance-critical components, such as ones that are not repeated frequently.</span></span> <span data-ttu-id="b0f58-225">Tuttavia, per i componenti che eseguono il rendering su larga scala, ad esempio ogni elemento in un elenco di grandi dimensioni o celle di una griglia, provare a evitare l'attributo splatting.</span><span class="sxs-lookup"><span data-stu-id="b0f58-225">However for components that render at scale, such as each items in a large list or cells in a grid, try to avoid attribute splatting.</span></span>

<span data-ttu-id="b0f58-226">Per altre informazioni, vedere <xref:blazor/components/index#attribute-splatting-and-arbitrary-parameters>.</span><span class="sxs-lookup"><span data-stu-id="b0f58-226">For more information, see <xref:blazor/components/index#attribute-splatting-and-arbitrary-parameters>.</span></span>

#### <a name="implement-setparametersasync-manually"></a><span data-ttu-id="b0f58-227">Implementare `SetParametersAsync` manualmente</span><span class="sxs-lookup"><span data-stu-id="b0f58-227">Implement `SetParametersAsync` manually</span></span>

<span data-ttu-id="b0f58-228">Uno degli aspetti principali del sovraccarico di rendering per componente consiste nel scrivere i valori dei parametri in ingresso nelle `[Parameter]` Proprietà.</span><span class="sxs-lookup"><span data-stu-id="b0f58-228">One of the main aspects of the per-component rendering overhead is writing incoming parameter values to the `[Parameter]` properties.</span></span> <span data-ttu-id="b0f58-229">Per eseguire questa operazione, il renderer deve utilizzare la reflection.</span><span class="sxs-lookup"><span data-stu-id="b0f58-229">The renderer has to use reflection to do this.</span></span> <span data-ttu-id="b0f58-230">Anche se si tratta di un po' ottimizzato, l'assenza del supporto JIT nel runtime di webassembly impone limiti.</span><span class="sxs-lookup"><span data-stu-id="b0f58-230">Even though this is somewhat optimized, the absence of JIT support on the WebAssembly runtime imposes limits.</span></span>

<span data-ttu-id="b0f58-231">In alcuni casi estremi, può essere utile evitare la reflection e implementare manualmente la logica di impostazione dei parametri.</span><span class="sxs-lookup"><span data-stu-id="b0f58-231">In some extreme cases, you may wish to avoid the reflection and implement your own parameter setting logic manually.</span></span> <span data-ttu-id="b0f58-232">Questo può essere applicabile nei casi seguenti:</span><span class="sxs-lookup"><span data-stu-id="b0f58-232">This may be applicable when:</span></span>

* <span data-ttu-id="b0f58-233">Si dispone di un componente che esegue il rendering molto spesso (ad esempio, sono presenti centinaia o migliaia di copie dell'interfaccia utente).</span><span class="sxs-lookup"><span data-stu-id="b0f58-233">You have a component that renders extremely often (for example, there are hundreds or thousands of copies of it in the UI).</span></span>
* <span data-ttu-id="b0f58-234">Accetta molti parametri.</span><span class="sxs-lookup"><span data-stu-id="b0f58-234">It accepts many parameters.</span></span>
* <span data-ttu-id="b0f58-235">Si noterà che l'overhead della ricezione dei parametri ha un impatto osservabile sulla velocità di risposta dell'interfaccia utente.</span><span class="sxs-lookup"><span data-stu-id="b0f58-235">You find that the overhead of receiving parameters has an observable impact on UI responsiveness.</span></span>

<span data-ttu-id="b0f58-236">In questi casi, è possibile eseguire l'override del metodo virtuale del componente <xref:Microsoft.AspNetCore.Components.ComponentBase.SetParametersAsync%2A> e implementare la logica specifica del componente.</span><span class="sxs-lookup"><span data-stu-id="b0f58-236">In these cases, you can override the component's virtual <xref:Microsoft.AspNetCore.Components.ComponentBase.SetParametersAsync%2A> method and implement your own component-specific logic.</span></span> <span data-ttu-id="b0f58-237">Nell'esempio seguente vengono deliberatamente evitate le ricerche nel dizionario:</span><span class="sxs-lookup"><span data-stu-id="b0f58-237">The following example deliberately avoids any dictionary lookups:</span></span>

```razor
@code {
    [Parameter]
    public int MessageId { get; set; }

    [Parameter]
    public string Text { get; set; }

    [Parameter]
    public EventCallback<string> TextChanged { get; set; }

    [Parameter]
    public Theme CurrentTheme { get; set; }

    public override Task SetParametersAsync(ParameterView parameters)
    {
        foreach (var parameter in parameters)
        {
            switch (parameter.Name)
            {
                case nameof(MessageId):
                    MessageId = (int)parameter.Value;
                    break;
                case nameof(Text):
                    Text = (string)parameter.Value;
                    break;
                case nameof(TextChanged):
                    TextChanged = (EventCallback<string>)parameter.Value;
                    break;
                case nameof(CurrentTheme):
                    CurrentTheme = (Theme)parameter.Value;
                    break;
                default:
                    throw new ArgumentException($"Unknown parameter: {parameter.Name}");
            }
        }

        return base.SetParametersAsync(ParameterView.Empty);
    }
}
```

<span data-ttu-id="b0f58-238">Nel codice precedente, la restituzione della classe base <xref:Microsoft.AspNetCore.Components.ComponentBase.SetParametersAsync%2A> esegue i normali metodi del ciclo di vita senza assegnare nuovamente parametri.</span><span class="sxs-lookup"><span data-stu-id="b0f58-238">In the preceding code, returning the base class <xref:Microsoft.AspNetCore.Components.ComponentBase.SetParametersAsync%2A> runs the normal lifecycle methods without assigning parameters again.</span></span>

<span data-ttu-id="b0f58-239">Come si può vedere nel codice precedente, l'override <xref:Microsoft.AspNetCore.Components.ComponentBase.SetParametersAsync%2A> e la fornitura di logica personalizzata sono complicate e laboriose, pertanto non è consigliabile usare questo approccio in generale.</span><span class="sxs-lookup"><span data-stu-id="b0f58-239">As you can see in the preceding code, overriding <xref:Microsoft.AspNetCore.Components.ComponentBase.SetParametersAsync%2A> and supplying custom logic is complicated and laborious, so we don't recommend this approach in general.</span></span> <span data-ttu-id="b0f58-240">In casi estremi, può migliorare le prestazioni di rendering del 20-25%, ma è opportuno considerare questo approccio solo negli scenari elencati in precedenza.</span><span class="sxs-lookup"><span data-stu-id="b0f58-240">In extreme cases, it can improve rendering performance by 20-25%, but you should only consider this approach in the scenarios listed earlier.</span></span>

### <a name="dont-trigger-events-too-rapidly"></a><span data-ttu-id="b0f58-241">Non attivare eventi troppo rapidamente</span><span class="sxs-lookup"><span data-stu-id="b0f58-241">Don't trigger events too rapidly</span></span>

<span data-ttu-id="b0f58-242">Alcuni eventi del browser vengono attivati molto spesso, ad esempio `onmousemove` e `onscroll` , che possono essere attivati decine o centinaia di volte al secondo.</span><span class="sxs-lookup"><span data-stu-id="b0f58-242">Some browser events fire extremely frequently, for example `onmousemove` and `onscroll`, which can fire tens or hundreds of times per second.</span></span> <span data-ttu-id="b0f58-243">Nella maggior parte dei casi, non è necessario eseguire gli aggiornamenti dell'interfaccia utente in modo frequente.</span><span class="sxs-lookup"><span data-stu-id="b0f58-243">In most cases, you don't need to perform UI updates this frequently.</span></span> <span data-ttu-id="b0f58-244">Se si tenta di eseguire questa operazione, si potrebbe danneggiare la velocità di risposta dell'interfaccia utente o utilizzare un tempo di CPU eccessivo.</span><span class="sxs-lookup"><span data-stu-id="b0f58-244">If you try to do so, you may harm UI responsiveness or consume excessive CPU time.</span></span>

<span data-ttu-id="b0f58-245">Anziché usare eventi nativi `@onmousemove` o `@onscroll` , può essere preferibile usare l'interoperabilità js per registrare un callback che viene attivato meno di frequente.</span><span class="sxs-lookup"><span data-stu-id="b0f58-245">Rather than using native `@onmousemove` or `@onscroll` events, you may prefer to use JS interop to register a callback that fires less frequently.</span></span> <span data-ttu-id="b0f58-246">Ad esempio, il componente seguente ( `MyComponent.razor` ) Visualizza la posizione del mouse, ma solo gli aggiornamenti al massimo ogni 500 ms:</span><span class="sxs-lookup"><span data-stu-id="b0f58-246">For example, the following component (`MyComponent.razor`) displays the position of the mouse but only updates at most once every 500 ms:</span></span>

```razor
@inject IJSRuntime JS
@implements IDisposable

<h1>@message</h1>

<div @ref="myMouseMoveElement" style="border:1px dashed red;height:200px;">
    Move mouse here
</div>

@code {
    ElementReference myMouseMoveElement;
    DotNetObjectReference<MyComponent> selfReference;
    private string message = "Move the mouse in the box";

    [JSInvokable]
    public void HandleMouseMove(int x, int y)
    {
        message = $"Mouse move at {x}, {y}";
        StateHasChanged();
    }

    protected override async Task OnAfterRenderAsync(bool firstRender)
    {
        if (firstRender)
        {
            selfReference = DotNetObjectReference.Create(this);
            var minInterval = 500; // Only notify every 500 ms
            await JS.InvokeVoidAsync("onThrottledMouseMove", 
                myMouseMoveElement, selfReference, minInterval);
        }
    }

    public void Dispose() => selfReference?.Dispose();
}
```

<span data-ttu-id="b0f58-247">Il codice JavaScript corrispondente, che può essere inserito nella `index.html` pagina o caricato come modulo ES6, registra il listener di eventi DOM effettivo.</span><span class="sxs-lookup"><span data-stu-id="b0f58-247">The corresponding JavaScript code, which can be placed in the `index.html` page or loaded as an ES6 module, registers the actual DOM event listener.</span></span> <span data-ttu-id="b0f58-248">In questo esempio, il listener di eventi usa la [ `throttle` funzione di Lodash](https://lodash.com/docs/4.17.15#throttle) per limitare la frequenza delle chiamate:</span><span class="sxs-lookup"><span data-stu-id="b0f58-248">In this example, the event listener uses [Lodash's `throttle` function](https://lodash.com/docs/4.17.15#throttle) to limit the rate of invocations:</span></span>

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/lodash.js/4.17.20/lodash.min.js"></script>
<script>
  function onThrottledMouseMove(elem, component, interval) {
    elem.addEventListener('mousemove', _.throttle(e => {
      component.invokeMethodAsync('HandleMouseMove', e.offsetX, e.offsetY);
    }, interval));
  }
</script>
```

<span data-ttu-id="b0f58-249">Questa tecnica può essere ancora più importante per Blazor Server , perché ogni chiamata di evento comporta il recapito di un messaggio tramite la rete.</span><span class="sxs-lookup"><span data-stu-id="b0f58-249">This technique can be even more important for Blazor Server, since each event invocation involves delivering a message over the network.</span></span> <span data-ttu-id="b0f58-250">È utile per Blazor WebAssembly perché può ridurre notevolmente la quantità di lavoro di rendering.</span><span class="sxs-lookup"><span data-stu-id="b0f58-250">It's valuable for Blazor WebAssembly because it can greatly reduce the amount of rendering work.</span></span>

## <a name="optimize-javascript-interop-speed"></a><span data-ttu-id="b0f58-251">Ottimizza la velocità di interoperabilità JavaScript</span><span class="sxs-lookup"><span data-stu-id="b0f58-251">Optimize JavaScript interop speed</span></span>

<span data-ttu-id="b0f58-252">Le chiamate tra .NET e JavaScript comportano un sovraccarico aggiuntivo perché:</span><span class="sxs-lookup"><span data-stu-id="b0f58-252">Calls between .NET and JavaScript involve some additional overhead because:</span></span>

* <span data-ttu-id="b0f58-253">Per impostazione predefinita, le chiamate sono asincrone.</span><span class="sxs-lookup"><span data-stu-id="b0f58-253">By default, calls are asynchronous.</span></span>
* <span data-ttu-id="b0f58-254">Per impostazione predefinita, i parametri e i valori restituiti sono serializzati in JSON.</span><span class="sxs-lookup"><span data-stu-id="b0f58-254">By default, parameters and return values are JSON-serialized.</span></span> <span data-ttu-id="b0f58-255">Questo consente di fornire un meccanismo di conversione di facile comprensione tra i tipi .NET e JavaScript.</span><span class="sxs-lookup"><span data-stu-id="b0f58-255">This is to provide an easy-to-understand conversion mechanism between .NET and JavaScript types.</span></span>

<span data-ttu-id="b0f58-256">Inoltre Blazor Server , queste chiamate vengono passate attraverso la rete.</span><span class="sxs-lookup"><span data-stu-id="b0f58-256">Additionally on Blazor Server, these calls are passed across the network.</span></span>

### <a name="avoid-excessively-fine-grained-calls"></a><span data-ttu-id="b0f58-257">Evitare chiamate con granularità eccessiva</span><span class="sxs-lookup"><span data-stu-id="b0f58-257">Avoid excessively fine-grained calls</span></span>

<span data-ttu-id="b0f58-258">Poiché ogni chiamata implica un sovraccarico, può essere utile ridurre il numero di chiamate.</span><span class="sxs-lookup"><span data-stu-id="b0f58-258">Since each call involves some overhead, it can be valuable to reduce the number of calls.</span></span> <span data-ttu-id="b0f58-259">Si consideri il codice seguente, in cui viene archiviata una raccolta di elementi nell'archivio del browser `localStorage` :</span><span class="sxs-lookup"><span data-stu-id="b0f58-259">Consider the following code, which stores a collection of items in the browser's `localStorage` store:</span></span>

```csharp
private async Task StoreAllInLocalStorage(IEnumerable<TodoItem> items)
{
    foreach (var item in items)
    {
        await JS.InvokeVoidAsync("localStorage.setItem", item.Id, 
            JsonSerializer.Serialize(item));
    }
}
```

<span data-ttu-id="b0f58-260">L'esempio precedente esegue una chiamata di interoperabilità JS separata per ogni elemento.</span><span class="sxs-lookup"><span data-stu-id="b0f58-260">The preceding example makes a separate JS interop call for each item.</span></span> <span data-ttu-id="b0f58-261">Al contrario, l'approccio seguente riduce l'interoperabilità JS a una singola chiamata:</span><span class="sxs-lookup"><span data-stu-id="b0f58-261">Instead, the following approach reduces the JS interop to a single call:</span></span>

```csharp
private async Task StoreAllInLocalStorage(IEnumerable<TodoItem> items)
{
    await JS.InvokeVoidAsync("storeAllInLocalStorage", items);
}
```

<span data-ttu-id="b0f58-262">La funzione JavaScript corrispondente definita nel modo seguente:</span><span class="sxs-lookup"><span data-stu-id="b0f58-262">The corresponding JavaScript function defined as follows:</span></span>

```javascript
function storeAllInLocalStorage(items) {
  items.forEach(item => {
    localStorage.setItem(item.id, JSON.stringify(item));
  });
}
```

<span data-ttu-id="b0f58-263">Per Blazor WebAssembly , questo è in genere importante solo se si sta effettuando un numero elevato di chiamate di interoperabilità js.</span><span class="sxs-lookup"><span data-stu-id="b0f58-263">For Blazor WebAssembly, this usually only matters if you're making a large number of JS interop calls.</span></span>

### <a name="consider-making-synchronous-calls"></a><span data-ttu-id="b0f58-264">Prendere in considerazione l'esecuzione di chiamate sincrone</span><span class="sxs-lookup"><span data-stu-id="b0f58-264">Consider making synchronous calls</span></span>

<span data-ttu-id="b0f58-265">Per impostazione predefinita, le chiamate di interoperabilità JavaScript sono asincrone, indipendentemente dal fatto che il codice chiamato sia sincrono o asincrono.</span><span class="sxs-lookup"><span data-stu-id="b0f58-265">JavaScript interop calls are asynchronous by default, regardless of whether the code being called is synchronous or asynchronous.</span></span> <span data-ttu-id="b0f58-266">Ciò consente di garantire che i componenti siano compatibili con Blazor WebAssembly e Blazor Server .</span><span class="sxs-lookup"><span data-stu-id="b0f58-266">This is to ensure components are compatible with both Blazor WebAssembly and Blazor Server.</span></span> <span data-ttu-id="b0f58-267">In Blazor Server , tutte le chiamate di interoperabilità JavaScript devono essere asincrone perché vengono inviate tramite una connessione di rete.</span><span class="sxs-lookup"><span data-stu-id="b0f58-267">On Blazor Server, all JavaScript interop calls must be asynchronous because they are sent over a network connection.</span></span>

<span data-ttu-id="b0f58-268">Se si è certi che l'app venga eseguita solo in Blazor WebAssembly , è possibile scegliere di eseguire chiamate di interoperabilità JavaScript sincrone.</span><span class="sxs-lookup"><span data-stu-id="b0f58-268">If you know for certain that your app only ever runs on Blazor WebAssembly, you can choose to make synchronous JavaScript interop calls.</span></span> <span data-ttu-id="b0f58-269">Questa operazione ha un sovraccarico leggermente inferiore rispetto all'esecuzione di chiamate asincrone e può comportare un minor numero di cicli di rendering perché non esiste alcuno stato intermedio durante l'attesa dei risultati.</span><span class="sxs-lookup"><span data-stu-id="b0f58-269">This has slightly less overhead than making asynchronous calls and can result in fewer render cycles because there is no intermediate state while awaiting results.</span></span>

<span data-ttu-id="b0f58-270">Per eseguire una chiamata sincrona da .NET a JavaScript, eseguire il cast <xref:Microsoft.JSInterop.IJSRuntime> a <xref:Microsoft.JSInterop.IJSInProcessRuntime> :</span><span class="sxs-lookup"><span data-stu-id="b0f58-270">To make a synchronous call from .NET to JavaScript, cast <xref:Microsoft.JSInterop.IJSRuntime> to <xref:Microsoft.JSInterop.IJSInProcessRuntime>:</span></span>

```razor
@inject IJSRuntime JS

...

@code {
    protected override void HandleSomeEvent()
    {
        var jsInProcess = (IJSInProcessRuntime)JS;
        var value = jsInProcess.Invoke<string>("javascriptFunctionIdentifier");
    }
}
```

::: moniker range=">= aspnetcore-5.0"

<span data-ttu-id="b0f58-271">Quando si utilizza `IJSObjectReference` , è possibile effettuare una chiamata sincrona eseguendo il cast a `IJSInProcessObjectReference` .</span><span class="sxs-lookup"><span data-stu-id="b0f58-271">When working with `IJSObjectReference`, you can make a synchronous call by casting to `IJSInProcessObjectReference`.</span></span>

::: moniker-end

<span data-ttu-id="b0f58-272">Per eseguire una chiamata sincrona da JavaScript a .NET, usare `DotNet.invokeMethod` anziché `DotNet.invokeMethodAsync` .</span><span class="sxs-lookup"><span data-stu-id="b0f58-272">To make a synchronous call from JavaScript to .NET, use `DotNet.invokeMethod` instead of `DotNet.invokeMethodAsync`.</span></span>

<span data-ttu-id="b0f58-273">Le chiamate sincrone funzionano se:</span><span class="sxs-lookup"><span data-stu-id="b0f58-273">Synchronous calls work if:</span></span>

* <span data-ttu-id="b0f58-274">L'app è in esecuzione in Blazor WebAssembly , non Blazor Server .</span><span class="sxs-lookup"><span data-stu-id="b0f58-274">The app is running on Blazor WebAssembly, not Blazor Server.</span></span>
* <span data-ttu-id="b0f58-275">La funzione chiamata restituisce un valore in modo sincrono (non è un `async` metodo e non restituisce .NET <xref:System.Threading.Tasks.Task> o JavaScript `Promise` ).</span><span class="sxs-lookup"><span data-stu-id="b0f58-275">The called function returns a value synchronously (it isn't an `async` method and doesn't return a .NET <xref:System.Threading.Tasks.Task> or JavaScript `Promise`).</span></span>

<span data-ttu-id="b0f58-276">Per altre informazioni, vedere <xref:blazor/call-javascript-from-dotnet>.</span><span class="sxs-lookup"><span data-stu-id="b0f58-276">For more information, see <xref:blazor/call-javascript-from-dotnet>.</span></span>

::: moniker range=">= aspnetcore-5.0"
 
### <a name="consider-making-unmarshalled-calls"></a><span data-ttu-id="b0f58-277">Prendere in considerazione l'esecuzione di chiamate non Marshall</span><span class="sxs-lookup"><span data-stu-id="b0f58-277">Consider making unmarshalled calls</span></span>

<span data-ttu-id="b0f58-278">Quando si esegue in Blazor WebAssembly , è possibile effettuare chiamate non Marshall da .NET a JavaScript.</span><span class="sxs-lookup"><span data-stu-id="b0f58-278">When running on Blazor WebAssembly, it's possible to make unmarshalled calls from .NET to JavaScript.</span></span> <span data-ttu-id="b0f58-279">Si tratta di chiamate sincrone che non eseguono la serializzazione JSON di argomenti o valori restituiti.</span><span class="sxs-lookup"><span data-stu-id="b0f58-279">These are synchronous calls that don't perform JSON serialization of arguments or return values.</span></span> <span data-ttu-id="b0f58-280">Tutti gli aspetti della gestione della memoria e delle traduzioni tra le rappresentazioni .NET e JavaScript vengono lasciati allo sviluppatore.</span><span class="sxs-lookup"><span data-stu-id="b0f58-280">All aspects of memory management and translations between .NET and JavaScript representations are left up to the developer.</span></span>

> [!WARNING]
> <span data-ttu-id="b0f58-281">Quando `IJSUnmarshalledRuntime` si usa ha il minor sovraccarico degli approcci di interoperabilità js, le API JavaScript necessarie per interagire con queste API non sono attualmente documentate e sono soggette a modifiche di rilievo nelle versioni future.</span><span class="sxs-lookup"><span data-stu-id="b0f58-281">While using `IJSUnmarshalledRuntime` has the least overhead of the JS interop approaches, the JavaScript APIs required to interact with these APIs are currently undocumented and subject to breaking changes in future releases.</span></span>

```javascript
function jsInteropCall() {
    return BINDING.js_to_mono_obj("Hello world");
}
```

```razor
@inject IJSRuntime JS

@code {
    protected override void OnInitialized()
    {
        var unmarshalledJs = (IJSUnmarshalledRuntime)JS;
        var value = unmarshalledJs.InvokeUnmarshalled<string>("jsInteropCall");
    }
}
```

::: moniker-end

## <a name="minimize-app-download-size"></a><span data-ttu-id="b0f58-282">Ridurre le dimensioni del download dell'app</span><span class="sxs-lookup"><span data-stu-id="b0f58-282">Minimize app download size</span></span>

::: moniker range=">= aspnetcore-5.0"

### <a name="intermediate-language-il-trimming"></a><span data-ttu-id="b0f58-283">Troncamento del linguaggio intermedio (IL)</span><span class="sxs-lookup"><span data-stu-id="b0f58-283">Intermediate Language (IL) trimming</span></span>

<span data-ttu-id="b0f58-284">Il trimming degli assembly inutilizzati da un' Blazor WebAssembly app consente di ridurre le dimensioni dell'app rimuovendo il codice inutilizzato nei file binari dell'app.</span><span class="sxs-lookup"><span data-stu-id="b0f58-284">Trimming unused assemblies from a Blazor WebAssembly app reduces the app's size by removing unused code in the app's binaries.</span></span> <span data-ttu-id="b0f58-285">Per altre informazioni, vedere <xref:blazor/host-and-deploy/configure-trimmer>.</span><span class="sxs-lookup"><span data-stu-id="b0f58-285">For more information, see <xref:blazor/host-and-deploy/configure-trimmer>.</span></span>

::: moniker-end

::: moniker range="< aspnetcore-5.0"

### <a name="intermediate-language-il-linking"></a><span data-ttu-id="b0f58-286">Collegamento del linguaggio intermedio (IL)</span><span class="sxs-lookup"><span data-stu-id="b0f58-286">Intermediate Language (IL) linking</span></span>

<span data-ttu-id="b0f58-287">Il [collegamento di un' Blazor WebAssembly app](xref:blazor/host-and-deploy/configure-linker) consente di ridurre le dimensioni dell'app, ritagliando il codice inutilizzato nei file binari dell'app.</span><span class="sxs-lookup"><span data-stu-id="b0f58-287">[Linking a Blazor WebAssembly app](xref:blazor/host-and-deploy/configure-linker) reduces the app's size by trimming unused code in the app's binaries.</span></span> <span data-ttu-id="b0f58-288">Per impostazione predefinita, il linker linguaggio intermedio (IL) viene abilitato solo quando si compila nella `Release` configurazione.</span><span class="sxs-lookup"><span data-stu-id="b0f58-288">By default, the Intermediate Language (IL) Linker is only enabled when building in `Release` configuration.</span></span> <span data-ttu-id="b0f58-289">Per trarre vantaggio da questo problema, pubblicare l'app per la distribuzione usando il [`dotnet publish`](/dotnet/core/tools/dotnet-publish) comando con l'opzione [-c |--Configuration](/dotnet/core/tools/dotnet-publish#options) impostata su `Release` :</span><span class="sxs-lookup"><span data-stu-id="b0f58-289">To benefit from this, publish the app for deployment using the [`dotnet publish`](/dotnet/core/tools/dotnet-publish) command with the [-c|--configuration](/dotnet/core/tools/dotnet-publish#options) option set to `Release`:</span></span>

```dotnetcli
dotnet publish -c Release
```

::: moniker-end

### <a name="use-systemtextjson"></a><span data-ttu-id="b0f58-290">USA System.Text.Js</span><span class="sxs-lookup"><span data-stu-id="b0f58-290">Use System.Text.Json</span></span>

<span data-ttu-id="b0f58-291">Blazorl'implementazione dell'interoperabilità JS di è basata su <xref:System.Text.Json> , ovvero una libreria di serializzazione JSON a prestazioni elevate con allocazione di memoria insufficiente.</span><span class="sxs-lookup"><span data-stu-id="b0f58-291">Blazor's JS interop implementation relies on <xref:System.Text.Json>, which is a high-performance JSON serialization library with low memory allocation.</span></span> <span data-ttu-id="b0f58-292">L'uso <xref:System.Text.Json> di non comporta dimensioni aggiuntive del payload dell'app rispetto all'aggiunta di una o più librerie JSON alternative.</span><span class="sxs-lookup"><span data-stu-id="b0f58-292">Using <xref:System.Text.Json> doesn't result in additional app payload size over adding one or more alternate JSON libraries.</span></span>

<span data-ttu-id="b0f58-293">Per informazioni aggiuntive sulla migrazione, vedere [come eseguire la migrazione da `Newtonsoft.Json` a `System.Text.Json` ](/dotnet/standard/serialization/system-text-json-migrate-from-newtonsoft-how-to).</span><span class="sxs-lookup"><span data-stu-id="b0f58-293">For migration guidance, see [How to migrate from `Newtonsoft.Json` to `System.Text.Json`](/dotnet/standard/serialization/system-text-json-migrate-from-newtonsoft-how-to).</span></span>

### <a name="lazy-load-assemblies"></a><span data-ttu-id="b0f58-294">Assembly di caricamento lazy</span><span class="sxs-lookup"><span data-stu-id="b0f58-294">Lazy load assemblies</span></span>

<span data-ttu-id="b0f58-295">Caricare gli assembly in fase di esecuzione quando gli assembly sono necessari per una route.</span><span class="sxs-lookup"><span data-stu-id="b0f58-295">Load assemblies at runtime when the assemblies are required by a route.</span></span> <span data-ttu-id="b0f58-296">Per altre informazioni, vedere <xref:blazor/webassembly-lazy-load-assemblies>.</span><span class="sxs-lookup"><span data-stu-id="b0f58-296">For more information, see <xref:blazor/webassembly-lazy-load-assemblies>.</span></span>

### <a name="compression"></a><span data-ttu-id="b0f58-297">Compressione</span><span class="sxs-lookup"><span data-stu-id="b0f58-297">Compression</span></span>

<span data-ttu-id="b0f58-298">Quando Blazor WebAssembly viene pubblicata un'app, l'output viene compresso in modo statico durante la pubblicazione per ridurre le dimensioni dell'app e rimuovere l'overhead per la compressione in fase di esecuzione.</span><span class="sxs-lookup"><span data-stu-id="b0f58-298">When a Blazor WebAssembly app is published, the output is statically compressed during publish to reduce the app's size and remove the overhead for runtime compression.</span></span> <span data-ttu-id="b0f58-299">Blazor si basa sul server per eseguire il contenuto negotation e gestire file compressi in modo statico.</span><span class="sxs-lookup"><span data-stu-id="b0f58-299">Blazor relies on the server to perform content negotation and serve statically-compressed files.</span></span>

<span data-ttu-id="b0f58-300">Dopo che un'app è stata distribuita, verificare che l'app servi i file compressi.</span><span class="sxs-lookup"><span data-stu-id="b0f58-300">After an app is deployed, verify that the app serves compressed files.</span></span> <span data-ttu-id="b0f58-301">Controllare la scheda rete nel Strumenti di sviluppo del browser e verificare che i file siano serviti con `Content-Encoding: br` o `Content-Encoding: gz` .</span><span class="sxs-lookup"><span data-stu-id="b0f58-301">Inspect the Network tab in a browser's Developer Tools and verify that the files are served with `Content-Encoding: br` or `Content-Encoding: gz`.</span></span> <span data-ttu-id="b0f58-302">Se l'host non serve file compressi, seguire le istruzioni riportate in <xref:blazor/host-and-deploy/webassembly#compression> .</span><span class="sxs-lookup"><span data-stu-id="b0f58-302">If the host isn't serving compressed files, follow the instructions in <xref:blazor/host-and-deploy/webassembly#compression>.</span></span>

### <a name="disable-unused-features"></a><span data-ttu-id="b0f58-303">Disabilitare le funzionalità inutilizzate</span><span class="sxs-lookup"><span data-stu-id="b0f58-303">Disable unused features</span></span>

<span data-ttu-id="b0f58-304">Blazor WebAssemblyil runtime di include le seguenti funzionalità .NET che possono essere disabilitate se l'app non le richiede per una dimensione di payload inferiore:</span><span class="sxs-lookup"><span data-stu-id="b0f58-304">Blazor WebAssembly's runtime includes the following .NET features that can be disabled if the app doesn't require them for a smaller payload size:</span></span>

* <span data-ttu-id="b0f58-305">Un file di dati è incluso per rendere corrette le informazioni sul fuso orario.</span><span class="sxs-lookup"><span data-stu-id="b0f58-305">A data file is included to make timezone information correct.</span></span> <span data-ttu-id="b0f58-306">Se l'app non richiede questa funzionalità, provare a disabilitarla impostando la `BlazorEnableTimeZoneSupport` Proprietà MSBuild nel file di progetto dell'app su `false` :</span><span class="sxs-lookup"><span data-stu-id="b0f58-306">If the app doesn't require this feature, consider disabling it by setting the `BlazorEnableTimeZoneSupport` MSBuild property in the app's project file to `false`:</span></span>

  ```xml
  <PropertyGroup>
    <BlazorEnableTimeZoneSupport>false</BlazorEnableTimeZoneSupport>
  </PropertyGroup>
  ```

::: moniker range=">= aspnetcore-5.0"

* <span data-ttu-id="b0f58-307">Per impostazione predefinita, include Blazor WebAssembly le risorse di globalizzazione necessarie per visualizzare i valori, ad esempio le date e la valuta, nelle impostazioni cultura dell'utente.</span><span class="sxs-lookup"><span data-stu-id="b0f58-307">By default, Blazor WebAssembly carries globalization resources required to display values, such as dates and currency, in the user's culture.</span></span> <span data-ttu-id="b0f58-308">Se l'app non richiede la localizzazione, è possibile [configurare l'app in modo che supporti la lingua inglese](xref:blazor/globalization-localization), che è basata sulle `en-US` impostazioni cultura:</span><span class="sxs-lookup"><span data-stu-id="b0f58-308">If the app doesn't require localization, you may [configure the app to support the invariant culture](xref:blazor/globalization-localization), which is based on the `en-US` culture:</span></span>

  ```xml
  <PropertyGroup>
    <InvariantGlobalization>true</InvariantGlobalization>
  </PropertyGroup>
  ```

::: moniker-end

::: moniker range="< aspnetcore-5.0"

* <span data-ttu-id="b0f58-309">Le informazioni sulle regole di confronto sono incluse per fare in modo che le API <xref:System.StringComparison.InvariantCultureIgnoreCase?displayProperty=nameWithType> funzionino correttamente.</span><span class="sxs-lookup"><span data-stu-id="b0f58-309">Collation information is included to make APIs such as <xref:System.StringComparison.InvariantCultureIgnoreCase?displayProperty=nameWithType> work correctly.</span></span> <span data-ttu-id="b0f58-310">Se si è certi che l'app non richiede i dati delle regole di confronto, provare a disabilitarla impostando la `BlazorWebAssemblyPreserveCollationData` Proprietà MSBuild nel file di progetto dell'app su `false` :</span><span class="sxs-lookup"><span data-stu-id="b0f58-310">If you're certain that the app doesn't require the collation data, consider disabling it by setting the `BlazorWebAssemblyPreserveCollationData` MSBuild property in the app's project file to `false`:</span></span>

  ```xml
  <PropertyGroup>
    <BlazorWebAssemblyPreserveCollationData>false</BlazorWebAssemblyPreserveCollationData>
  </PropertyGroup>
  ```

::: moniker-end
