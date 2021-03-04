---
title: BlazorVirtualizzazione componenti ASP.NET Core
author: guardrex
description: Informazioni su come usare la virtualizzazione di componenti nelle Blazor app ASP.NET Core.
monikerRange: '>= aspnetcore-5.0'
ms.author: riande
ms.custom: mvc
ms.date: 02/26/2021
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
uid: blazor/components/virtualization
ms.openlocfilehash: c81732c29b262e9134a4ff7dab077a4f31db96af
ms.sourcegitcommit: a1db01b4d3bd8c57d7a9c94ce122a6db68002d66
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/04/2021
ms.locfileid: "102109819"
---
# <a name="aspnet-core-blazor-component-virtualization"></a><span data-ttu-id="ca9ca-103">BlazorVirtualizzazione componenti ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="ca9ca-103">ASP.NET Core Blazor component virtualization</span></span>

<span data-ttu-id="ca9ca-104">Migliorare le prestazioni percepite del rendering dei componenti utilizzando il Blazor supporto di virtualizzazione incorporato del Framework con il [ `Virtualize` componente](xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601).</span><span class="sxs-lookup"><span data-stu-id="ca9ca-104">Improve the perceived performance of component rendering using the Blazor framework's built-in virtualization support with the [`Virtualize` component](xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601).</span></span> <span data-ttu-id="ca9ca-105">La virtualizzazione è una tecnica che consente di limitare il rendering dell'interfaccia utente solo alle parti attualmente visibili.</span><span class="sxs-lookup"><span data-stu-id="ca9ca-105">Virtualization is a technique for limiting UI rendering to just the parts that are currently visible.</span></span> <span data-ttu-id="ca9ca-106">La virtualizzazione, ad esempio, è utile quando l'applicazione deve eseguire il rendering di un lungo elenco di elementi ed è necessario che sia visibile solo un subset di elementi in un determinato momento.</span><span class="sxs-lookup"><span data-stu-id="ca9ca-106">For example, virtualization is helpful when the app must render a long list of items and only a subset of items is required to be visible at any given time.</span></span>

<span data-ttu-id="ca9ca-107">Usare il componente nei casi seguenti `Virtualize` :</span><span class="sxs-lookup"><span data-stu-id="ca9ca-107">Use the `Virtualize` component when:</span></span>

* <span data-ttu-id="ca9ca-108">Rendering di un set di elementi di dati in un ciclo.</span><span class="sxs-lookup"><span data-stu-id="ca9ca-108">Rendering a set of data items in a loop.</span></span>
* <span data-ttu-id="ca9ca-109">La maggior parte degli elementi non è visibile a causa dello scorrimento.</span><span class="sxs-lookup"><span data-stu-id="ca9ca-109">Most of the items aren't visible due to scrolling.</span></span>
* <span data-ttu-id="ca9ca-110">Gli elementi di cui è stato eseguito il rendering hanno le stesse dimensioni.</span><span class="sxs-lookup"><span data-stu-id="ca9ca-110">The rendered items are the same size.</span></span>

<span data-ttu-id="ca9ca-111">Quando l'utente scorre fino a un punto arbitrario nell' `Virtualize` elenco di elementi del componente, il componente calcola gli elementi visibili da visualizzare.</span><span class="sxs-lookup"><span data-stu-id="ca9ca-111">When the user scrolls to an arbitrary point in the `Virtualize` component's list of items, the component calculates the visible items to show.</span></span> <span data-ttu-id="ca9ca-112">Non viene eseguito il rendering degli elementi non visibili.</span><span class="sxs-lookup"><span data-stu-id="ca9ca-112">Unseen items aren't rendered.</span></span>

<span data-ttu-id="ca9ca-113">Senza virtualizzazione, un elenco tipico potrebbe usare un [`foreach`](/dotnet/csharp/language-reference/keywords/foreach-in) ciclo C# per eseguire il rendering di ogni elemento in un elenco.</span><span class="sxs-lookup"><span data-stu-id="ca9ca-113">Without virtualization, a typical list might use a C# [`foreach`](/dotnet/csharp/language-reference/keywords/foreach-in) loop to render each item in a list.</span></span> <span data-ttu-id="ca9ca-114">Nell'esempio seguente:</span><span class="sxs-lookup"><span data-stu-id="ca9ca-114">In the following example:</span></span>

* <span data-ttu-id="ca9ca-115">`allFlights` è una raccolta di voli aereo.</span><span class="sxs-lookup"><span data-stu-id="ca9ca-115">`allFlights` is a collection of airplane flights.</span></span>
* <span data-ttu-id="ca9ca-116">Il `FlightSummary` componente Visualizza i dettagli relativi a ogni volo.</span><span class="sxs-lookup"><span data-stu-id="ca9ca-116">The `FlightSummary` component displays details about each flight.</span></span>
* <span data-ttu-id="ca9ca-117">L' [ `@key` attributo della direttiva](xref:blazor/components/index#use-key-to-control-the-preservation-of-elements-and-components) conserva la relazione di ogni `FlightSummary` componente con il relativo volo di cui è stato eseguito il rendering da parte del volo `FlightId` .</span><span class="sxs-lookup"><span data-stu-id="ca9ca-117">The [`@key` directive attribute](xref:blazor/components/index#use-key-to-control-the-preservation-of-elements-and-components) preserves the relationship of each `FlightSummary` component to its rendered flight by the flight's `FlightId`.</span></span>

```razor
<div style="height:500px;overflow-y:scroll">
    @foreach (var flight in allFlights)
    {
        <FlightSummary @key="flight.FlightId" Details="@flight.Summary" />
    }
</div>
```

<span data-ttu-id="ca9ca-118">Se la raccolta contiene migliaia di voli, il rendering dei voli richiede molto tempo e gli utenti riscontrano un ritardo dell'interfaccia utente evidente.</span><span class="sxs-lookup"><span data-stu-id="ca9ca-118">If the collection contains thousands of flights, rendering the flights takes a long time and users experience a noticeable UI lag.</span></span> <span data-ttu-id="ca9ca-119">La maggior parte dei voli non viene sottoposta a rendering perché non rientrano nell'altezza dell' `<div>` elemento.</span><span class="sxs-lookup"><span data-stu-id="ca9ca-119">Most of the flights aren't rendered because they fall outside of the height of the `<div>` element.</span></span>

<span data-ttu-id="ca9ca-120">Anziché eseguire il rendering dell'intero elenco dei voli contemporaneamente, sostituire il [`foreach`](/dotnet/csharp/language-reference/keywords/foreach-in) ciclo nell'esempio precedente con il `Virtualize` componente:</span><span class="sxs-lookup"><span data-stu-id="ca9ca-120">Instead of rendering the entire list of flights at once, replace the [`foreach`](/dotnet/csharp/language-reference/keywords/foreach-in) loop in the preceding example with the `Virtualize` component:</span></span>

* <span data-ttu-id="ca9ca-121">Specificare `allFlights` come origine dell'elemento fisso su <xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.Items%2A?displayProperty=nameWithType> .</span><span class="sxs-lookup"><span data-stu-id="ca9ca-121">Specify `allFlights` as a fixed item source to <xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.Items%2A?displayProperty=nameWithType>.</span></span> <span data-ttu-id="ca9ca-122">Il componente Visualizza solo i voli attualmente visibili `Virtualize` .</span><span class="sxs-lookup"><span data-stu-id="ca9ca-122">Only the currently visible flights are rendered by the `Virtualize` component.</span></span>
* <span data-ttu-id="ca9ca-123">Specificare un contesto per ogni volo con il `Context` parametro.</span><span class="sxs-lookup"><span data-stu-id="ca9ca-123">Specify a context for each flight with the `Context` parameter.</span></span> <span data-ttu-id="ca9ca-124">Nell'esempio seguente `flight` viene usato come contesto, che fornisce l'accesso ai membri di ogni volo.</span><span class="sxs-lookup"><span data-stu-id="ca9ca-124">In the following example, `flight` is used as the context, which provides access to each flight's members.</span></span>

```razor
<div style="height:500px;overflow-y:scroll">
    <Virtualize Items="@allFlights" Context="flight">
        <FlightSummary @key="flight.FlightId" Details="@flight.Summary" />
    </Virtualize>
</div>
```

<span data-ttu-id="ca9ca-125">Se il contesto non è specificato con il `Context` parametro, utilizzare il valore di `context` nel modello di contenuto dell'elemento per accedere ai membri di ogni volo:</span><span class="sxs-lookup"><span data-stu-id="ca9ca-125">If a context isn't specified with the `Context` parameter, use the value of `context` in the item content template to access each flight's members:</span></span>

```razor
<div style="height:500px;overflow-y:scroll">
    <Virtualize Items="@allFlights">
        <FlightSummary @key="context.FlightId" Details="@context.Summary" />
    </Virtualize>
</div>
```

<span data-ttu-id="ca9ca-126">Il `Virtualize` componente:</span><span class="sxs-lookup"><span data-stu-id="ca9ca-126">The `Virtualize` component:</span></span>

* <span data-ttu-id="ca9ca-127">Calcola il numero di elementi di cui eseguire il rendering in base all'altezza del contenitore e alle dimensioni degli elementi di cui è stato eseguito il rendering.</span><span class="sxs-lookup"><span data-stu-id="ca9ca-127">Calculates the number of items to render based on the height of the container and the size of the rendered items.</span></span>
* <span data-ttu-id="ca9ca-128">Ricalcola ed esegue nuovamente il rendering degli elementi quando l'utente scorre.</span><span class="sxs-lookup"><span data-stu-id="ca9ca-128">Recalculates and rerenders the items as the user scrolls.</span></span>
* <span data-ttu-id="ca9ca-129">Recupera solo la sezione di record da un'API esterna che corrisponde all'area visibile corrente, anziché scaricare tutti i dati dalla raccolta.</span><span class="sxs-lookup"><span data-stu-id="ca9ca-129">Only fetches the slice of records from an external API that correspond to the current visible region, instead of downloading all of the data from the collection.</span></span>

<span data-ttu-id="ca9ca-130">Il contenuto dell'elemento per il `Virtualize` componente può includere:</span><span class="sxs-lookup"><span data-stu-id="ca9ca-130">The item content for the `Virtualize` component can include:</span></span>

* <span data-ttu-id="ca9ca-131">HTML semplice e Razor codice, come illustrato nell'esempio precedente.</span><span class="sxs-lookup"><span data-stu-id="ca9ca-131">Plain HTML and Razor code, as the preceding example shows.</span></span>
* <span data-ttu-id="ca9ca-132">Uno o più Razor componenti.</span><span class="sxs-lookup"><span data-stu-id="ca9ca-132">One or more Razor components.</span></span>
* <span data-ttu-id="ca9ca-133">Combinazione di componenti HTML/ Razor e Razor .</span><span class="sxs-lookup"><span data-stu-id="ca9ca-133">A mix of HTML/Razor and Razor components.</span></span>

## <a name="item-provider-delegate"></a><span data-ttu-id="ca9ca-134">Delegato del provider di elementi</span><span class="sxs-lookup"><span data-stu-id="ca9ca-134">Item provider delegate</span></span>

<span data-ttu-id="ca9ca-135">Se non si desidera caricare tutti gli elementi in memoria, è possibile specificare un metodo delegato del provider di elementi per il parametro del componente <xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.ItemsProvider%2A?displayProperty=nameWithType> che recupera in modo asincrono gli elementi richiesti su richiesta.</span><span class="sxs-lookup"><span data-stu-id="ca9ca-135">If you don't want to load all of the items into memory, you can specify an items provider delegate method to the component's <xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.ItemsProvider%2A?displayProperty=nameWithType> parameter that asynchronously retrieves the requested items on demand.</span></span> <span data-ttu-id="ca9ca-136">Nell'esempio seguente il `LoadEmployees` metodo fornisce gli elementi al `Virtualize` componente:</span><span class="sxs-lookup"><span data-stu-id="ca9ca-136">In the following example, the `LoadEmployees` method provides the items to the `Virtualize` component:</span></span>

```razor
<Virtualize Context="employee" ItemsProvider="@LoadEmployees">
    <p>
        @employee.FirstName @employee.LastName has the 
        job title of @employee.JobTitle.
    </p>
</Virtualize>
```

<span data-ttu-id="ca9ca-137">Il provider Items riceve un oggetto <xref:Microsoft.AspNetCore.Components.Web.Virtualization.ItemsProviderRequest> che specifica il numero di elementi richiesto a partire da un indice iniziale specifico.</span><span class="sxs-lookup"><span data-stu-id="ca9ca-137">The items provider receives an <xref:Microsoft.AspNetCore.Components.Web.Virtualization.ItemsProviderRequest>, which specifies the required number of items starting at a specific start index.</span></span> <span data-ttu-id="ca9ca-138">Il provider Items recupera quindi gli elementi richiesti da un database o da un altro servizio e li restituisce come <xref:Microsoft.AspNetCore.Components.Web.Virtualization.ItemsProviderResult%601> insieme al numero totale di elementi.</span><span class="sxs-lookup"><span data-stu-id="ca9ca-138">The items provider then retrieves the requested items from a database or other service and returns them as an <xref:Microsoft.AspNetCore.Components.Web.Virtualization.ItemsProviderResult%601> along with a count of the total items.</span></span> <span data-ttu-id="ca9ca-139">Il provider di elementi può scegliere di recuperare gli elementi con ogni richiesta o di memorizzarli nella cache in modo che siano immediatamente disponibili.</span><span class="sxs-lookup"><span data-stu-id="ca9ca-139">The items provider can choose to retrieve the items with each request or cache them so that they're readily available.</span></span>

<span data-ttu-id="ca9ca-140">Un `Virtualize` componente può accettare solo **un'origine elemento** dai relativi parametri, quindi non tentare di usare contemporaneamente un provider di elementi e assegnare una raccolta a `Items` .</span><span class="sxs-lookup"><span data-stu-id="ca9ca-140">A `Virtualize` component can only accept **one item source** from its parameters, so don't attempt to simultaneously use an items provider and assign a collection to `Items`.</span></span> <span data-ttu-id="ca9ca-141">Se entrambi sono assegnati, <xref:System.InvalidOperationException> viene generata un'eccezione quando i parametri del componente vengono impostati in fase di esecuzione.</span><span class="sxs-lookup"><span data-stu-id="ca9ca-141">If both are assigned, an <xref:System.InvalidOperationException> is thrown when the component's parameters are set at runtime.</span></span>

<span data-ttu-id="ca9ca-142">Nell' `LoadEmployees` esempio di metodo seguente vengono caricati i dipendenti da un `EmployeeService` (non visualizzato):</span><span class="sxs-lookup"><span data-stu-id="ca9ca-142">The following `LoadEmployees` method example loads employees from an `EmployeeService` (not shown):</span></span>

```csharp
private async ValueTask<ItemsProviderResult<Employee>> LoadEmployees(
    ItemsProviderRequest request)
{
    var numEmployees = Math.Min(request.Count, totalEmployees - request.StartIndex);
    var employees = await EmployeesService.GetEmployeesAsync(request.StartIndex, 
        numEmployees, request.CancellationToken);

    return new ItemsProviderResult<Employee>(employees, totalEmployees);
}
```

<span data-ttu-id="ca9ca-143"><xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.RefreshDataAsync%2A?displayProperty=nameWithType> indica al componente di ririchiedere i dati dal relativo oggetto <xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.ItemsProvider%2A> .</span><span class="sxs-lookup"><span data-stu-id="ca9ca-143"><xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.RefreshDataAsync%2A?displayProperty=nameWithType> instructs the component to rerequest data from its <xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.ItemsProvider%2A>.</span></span> <span data-ttu-id="ca9ca-144">Questa operazione è utile quando i dati esterni cambiano.</span><span class="sxs-lookup"><span data-stu-id="ca9ca-144">This is useful when external data changes.</span></span> <span data-ttu-id="ca9ca-145">Non è necessario chiamare quando si <xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.RefreshDataAsync%2A> Usa <xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.Items%2A> .</span><span class="sxs-lookup"><span data-stu-id="ca9ca-145">There's no need to call <xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.RefreshDataAsync%2A> when using <xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.Items%2A>.</span></span>

## <a name="placeholder"></a><span data-ttu-id="ca9ca-146">Segnaposto</span><span class="sxs-lookup"><span data-stu-id="ca9ca-146">Placeholder</span></span>

<span data-ttu-id="ca9ca-147">Poiché la richiesta di elementi da un'origine dati remota potrebbe richiedere del tempo, è possibile eseguire il rendering di un segnaposto con contenuto dell'elemento:</span><span class="sxs-lookup"><span data-stu-id="ca9ca-147">Because requesting items from a remote data source might take some time, you have the option to render a placeholder with item content:</span></span>

* <span data-ttu-id="ca9ca-148">Usare <xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.Placeholder%2A> ( `<Placeholder>...</Placeholder>` ) per visualizzare il contenuto fino a quando i dati dell'elemento non sono disponibili.</span><span class="sxs-lookup"><span data-stu-id="ca9ca-148">Use a <xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.Placeholder%2A> (`<Placeholder>...</Placeholder>`) to display content until the item data is available.</span></span>
* <span data-ttu-id="ca9ca-149">Utilizzare <xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.ItemContent%2A?displayProperty=nameWithType> per impostare il modello di elemento per l'elenco.</span><span class="sxs-lookup"><span data-stu-id="ca9ca-149">Use <xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.ItemContent%2A?displayProperty=nameWithType> to set the item template for the list.</span></span>

```razor
<Virtualize Context="employee" ItemsProvider="@LoadEmployees">
    <ItemContent>
        <p>
            @employee.FirstName @employee.LastName has the 
            job title of @employee.JobTitle.
        </p>
    </ItemContent>
    <Placeholder>
        <p>
            Loading&hellip;
        </p>
    </Placeholder>
</Virtualize>
```

## <a name="item-size"></a><span data-ttu-id="ca9ca-150">Dimensioni dell'elemento</span><span class="sxs-lookup"><span data-stu-id="ca9ca-150">Item size</span></span>

<span data-ttu-id="ca9ca-151">L'altezza di ogni elemento in pixel può essere impostata con <xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.ItemSize%2A?displayProperty=nameWithType> (impostazione predefinita: 50).</span><span class="sxs-lookup"><span data-stu-id="ca9ca-151">The height of each item in pixels can be set with <xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.ItemSize%2A?displayProperty=nameWithType> (default: 50).</span></span> <span data-ttu-id="ca9ca-152">Nell'esempio seguente viene modificata l'altezza di ogni elemento dal valore predefinito di 50 pixel a 25 pixel:</span><span class="sxs-lookup"><span data-stu-id="ca9ca-152">The following example changes the height of each item from the default of 50 pixels to 25 pixels:</span></span>

```razor
<Virtualize Context="employee" Items="@employees" ItemSize="25">
    ...
</Virtualize>
```

<span data-ttu-id="ca9ca-153">Per impostazione predefinita, il `Virtualize` componente misura le dimensioni di rendering (altezza) dei singoli elementi *dopo che* è stato eseguito il rendering iniziale.</span><span class="sxs-lookup"><span data-stu-id="ca9ca-153">By default, the `Virtualize` component measures the rendering size (height) of individual items *after* the initial render occurs.</span></span> <span data-ttu-id="ca9ca-154">Usare <xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.ItemSize%2A> per fornire in anticipo le dimensioni esatte degli elementi per supportare le prestazioni di rendering iniziali accurate e per garantire la corretta posizione di scorrimento per i ricaricamenti della pagina.</span><span class="sxs-lookup"><span data-stu-id="ca9ca-154">Use <xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.ItemSize%2A> to provide an exact item size in advance to assist with accurate initial render performance and to ensure the correct scroll position for page reloads.</span></span> <span data-ttu-id="ca9ca-155">Se per impostazione predefinita <xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.ItemSize%2A> viene eseguito il rendering di alcuni elementi al di fuori della visualizzazione attualmente visibile, viene attivato un secondo nuovo rendering.</span><span class="sxs-lookup"><span data-stu-id="ca9ca-155">If the default <xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.ItemSize%2A> causes some items to render outside of the currently visible view, a second re-render is triggered.</span></span> <span data-ttu-id="ca9ca-156">Per mantenere correttamente la posizione di scorrimento del browser in un elenco virtualizzato, il rendering iniziale deve essere corretto.</span><span class="sxs-lookup"><span data-stu-id="ca9ca-156">To correctly maintain the browser's scroll position in a virtualized list, the initial render must be correct.</span></span> <span data-ttu-id="ca9ca-157">In caso contrario, gli utenti potrebbero visualizzare gli elementi non corretti.</span><span class="sxs-lookup"><span data-stu-id="ca9ca-157">If not, users might view the wrong items.</span></span>

## <a name="overscan-count"></a><span data-ttu-id="ca9ca-158">Conteggio overscan</span><span class="sxs-lookup"><span data-stu-id="ca9ca-158">Overscan count</span></span>

<span data-ttu-id="ca9ca-159"><xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.OverscanCount%2A?displayProperty=nameWithType> determina il numero di elementi aggiuntivi che vengono sottoposti a rendering prima e dopo l'area visibile.</span><span class="sxs-lookup"><span data-stu-id="ca9ca-159"><xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.OverscanCount%2A?displayProperty=nameWithType> determines how many additional items are rendered before and after the visible region.</span></span> <span data-ttu-id="ca9ca-160">Questa impostazione consente di ridurre la frequenza di rendering durante lo scorrimento.</span><span class="sxs-lookup"><span data-stu-id="ca9ca-160">This setting helps to reduce the frequency of rendering during scrolling.</span></span> <span data-ttu-id="ca9ca-161">Tuttavia, i valori più elevati generano più elementi sottoposti a rendering nella pagina (impostazione predefinita: 3).</span><span class="sxs-lookup"><span data-stu-id="ca9ca-161">However, higher values result in more elements rendered in the page (default: 3).</span></span> <span data-ttu-id="ca9ca-162">Nell'esempio seguente viene modificato il conteggio overscan dal valore predefinito di tre elementi a quattro elementi:</span><span class="sxs-lookup"><span data-stu-id="ca9ca-162">The following example changes the overscan count from the default of three items to four items:</span></span>

```razor
<Virtualize Context="employee" Items="@employees" OverscanCount="4">
    ...
</Virtualize>
```

## <a name="state-changes"></a><span data-ttu-id="ca9ca-163">Modifiche stato</span><span class="sxs-lookup"><span data-stu-id="ca9ca-163">State changes</span></span>

<span data-ttu-id="ca9ca-164">Quando si apportano modifiche agli elementi di cui è stato eseguito il rendering dal `Virtualize` componente, chiamare <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> per forzare nuovamente la valutazione e il rendering del componente.</span><span class="sxs-lookup"><span data-stu-id="ca9ca-164">When making changes to items rendered by the `Virtualize` component, call <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> to force re-evaluation and rerendering of the component.</span></span> <span data-ttu-id="ca9ca-165">Per altre informazioni, vedere <xref:blazor/components/rendering>.</span><span class="sxs-lookup"><span data-stu-id="ca9ca-165">For more information, see <xref:blazor/components/rendering>.</span></span>
