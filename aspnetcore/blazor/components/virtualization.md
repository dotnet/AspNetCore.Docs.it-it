---
title: BlazorVirtualizzazione componenti ASP.NET Core
author: guardrex
description: Informazioni su come usare la virtualizzazione di componenti nelle Blazor app ASP.NET Core.
monikerRange: '>= aspnetcore-5.0'
ms.author: riande
ms.custom: mvc
ms.date: 10/02/2020
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
ms.openlocfilehash: d9fc767a4b5160c616053b075ba92194bcffa275
ms.sourcegitcommit: 1166b0ff3828418559510c661e8240e5c5717bb7
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/12/2021
ms.locfileid: "100280018"
---
# <a name="aspnet-core-blazor-component-virtualization"></a><span data-ttu-id="83b67-103">BlazorVirtualizzazione componenti ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="83b67-103">ASP.NET Core Blazor component virtualization</span></span>

<span data-ttu-id="83b67-104">Migliorare le prestazioni percepite del rendering dei componenti utilizzando il Blazor supporto della virtualizzazione incorporato del Framework.</span><span class="sxs-lookup"><span data-stu-id="83b67-104">Improve the perceived performance of component rendering using the Blazor framework's built-in virtualization support.</span></span> <span data-ttu-id="83b67-105">La virtualizzazione è una tecnica che consente di limitare il rendering dell'interfaccia utente solo alle parti attualmente visibili.</span><span class="sxs-lookup"><span data-stu-id="83b67-105">Virtualization is a technique for limiting UI rendering to just the parts that are currently visible.</span></span> <span data-ttu-id="83b67-106">La virtualizzazione, ad esempio, è utile quando l'applicazione deve eseguire il rendering di un lungo elenco di elementi ed è necessario che sia visibile solo un subset di elementi in un determinato momento.</span><span class="sxs-lookup"><span data-stu-id="83b67-106">For example, virtualization is helpful when the app must render a long list of items and only a subset of items is required to be visible at any given time.</span></span> <span data-ttu-id="83b67-107">Blazorfornisce il [ `Virtualize` componente](xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601) che può essere usato per aggiungere la virtualizzazione ai componenti di un'app.</span><span class="sxs-lookup"><span data-stu-id="83b67-107">Blazor provides the [`Virtualize` component](xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601) that can be used to add virtualization to an app's components.</span></span>

<span data-ttu-id="83b67-108">Il `Virtualize` componente può essere usato nei casi seguenti:</span><span class="sxs-lookup"><span data-stu-id="83b67-108">The `Virtualize` component can be used when:</span></span>

* <span data-ttu-id="83b67-109">Rendering di un set di elementi di dati in un ciclo.</span><span class="sxs-lookup"><span data-stu-id="83b67-109">Rendering a set of data items in a loop.</span></span>
* <span data-ttu-id="83b67-110">La maggior parte degli elementi non è visibile a causa dello scorrimento.</span><span class="sxs-lookup"><span data-stu-id="83b67-110">Most of the items aren't visible due to scrolling.</span></span>
* <span data-ttu-id="83b67-111">Gli elementi di cui è stato eseguito il rendering hanno esattamente le stesse dimensioni.</span><span class="sxs-lookup"><span data-stu-id="83b67-111">The rendered items are exactly the same size.</span></span> <span data-ttu-id="83b67-112">Quando l'utente scorre fino a un punto arbitrario, il componente può calcolare gli elementi visibili da visualizzare.</span><span class="sxs-lookup"><span data-stu-id="83b67-112">When the user scrolls to an arbitrary point, the component can calculate the visible items to show.</span></span>

<span data-ttu-id="83b67-113">Senza virtualizzazione, un elenco tipico potrebbe usare un [`foreach`](/dotnet/csharp/language-reference/keywords/foreach-in) ciclo C# per eseguire il rendering di ogni elemento nell'elenco:</span><span class="sxs-lookup"><span data-stu-id="83b67-113">Without virtualization, a typical list might use a C# [`foreach`](/dotnet/csharp/language-reference/keywords/foreach-in) loop to render each item in the list:</span></span>

```razor
<div style="height:500px;overflow-y:scroll">
    @foreach (var flight in allFlights)
    {
        <FlightSummary @key="flight.FlightId" Details="@flight.Summary" />
    }
</div>
```

<span data-ttu-id="83b67-114">Se l'elenco contiene migliaia di elementi, il rendering dell'elenco potrebbe richiedere molto tempo.</span><span class="sxs-lookup"><span data-stu-id="83b67-114">If the list contains thousands of items, then rendering the list may take a long time.</span></span> <span data-ttu-id="83b67-115">L'utente può riscontrare un ritardo dell'interfaccia utente evidente.</span><span class="sxs-lookup"><span data-stu-id="83b67-115">The user may experience a noticeable UI lag.</span></span>

<span data-ttu-id="83b67-116">Anziché eseguire il rendering di ogni elemento nell'elenco in una sola volta, sostituire il [`foreach`](/dotnet/csharp/language-reference/keywords/foreach-in) ciclo con il `Virtualize` componente e specificare un'origine di elemento fisso con <xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.Items%2A?displayProperty=nameWithType> .</span><span class="sxs-lookup"><span data-stu-id="83b67-116">Instead of rendering each item in the list all at one time, replace the [`foreach`](/dotnet/csharp/language-reference/keywords/foreach-in) loop with the `Virtualize` component and specify a fixed item source with <xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.Items%2A?displayProperty=nameWithType>.</span></span> <span data-ttu-id="83b67-117">Vengono visualizzati solo gli elementi attualmente visibili:</span><span class="sxs-lookup"><span data-stu-id="83b67-117">Only the items that are currently visible are rendered:</span></span>

```razor
<div style="height:500px;overflow-y:scroll">
    <Virtualize Items="@allFlights" Context="flight">
        <FlightSummary @key="flight.FlightId" Details="@flight.Summary" />
    </Virtualize>
</div>
```

<span data-ttu-id="83b67-118">Se non si specifica un contesto per il componente con `Context` , usare il `context` valore nel modello di contenuto dell'elemento:</span><span class="sxs-lookup"><span data-stu-id="83b67-118">If not specifying a context to the component with `Context`, use the `context` value in the item content template:</span></span>

```razor
<div style="height:500px;overflow-y:scroll">
    <Virtualize Items="@allFlights">
        <FlightSummary @key="context.FlightId" Details="@context.Summary" />
    </Virtualize>
</div>
```

> [!NOTE]
> <span data-ttu-id="83b67-119">Il processo di mapping degli oggetti modello a elementi e componenti può essere controllato con l' [`@key`](xref:mvc/views/razor#key) attributo della direttiva.</span><span class="sxs-lookup"><span data-stu-id="83b67-119">The mapping process of model objects to elements and components can be controlled with the [`@key`](xref:mvc/views/razor#key) directive attribute.</span></span> <span data-ttu-id="83b67-120">`@key` fa in modo che l'algoritmo diffe garantisca la conservazione di elementi o componenti in base al valore della chiave.</span><span class="sxs-lookup"><span data-stu-id="83b67-120">`@key` causes the diffing algorithm to guarantee preservation of elements or components based on the key's value.</span></span>
>
> <span data-ttu-id="83b67-121">Per altre informazioni, vedere gli articoli seguenti:</span><span class="sxs-lookup"><span data-stu-id="83b67-121">For more information, see the following articles:</span></span>
>
> * <xref:blazor/components/index#use-key-to-control-the-preservation-of-elements-and-components>
> * [<span data-ttu-id="83b67-122">Razor informazioni di riferimento sulla sintassi per ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="83b67-122">Razor syntax reference for ASP.NET Core</span></span>](xref:mvc/views/razor#key)

<span data-ttu-id="83b67-123">Il `Virtualize` componente:</span><span class="sxs-lookup"><span data-stu-id="83b67-123">The `Virtualize` component:</span></span>

* <span data-ttu-id="83b67-124">Calcola il numero di elementi di cui eseguire il rendering in base all'altezza del contenitore e alle dimensioni degli elementi di cui è stato eseguito il rendering.</span><span class="sxs-lookup"><span data-stu-id="83b67-124">Calculates how many items to render based on the height of the container and the size of the rendered items.</span></span>
* <span data-ttu-id="83b67-125">Ricalcola ed esegue nuovamente il rendering degli elementi quando l'utente scorre.</span><span class="sxs-lookup"><span data-stu-id="83b67-125">Recalculates and rerenders items as the user scrolls.</span></span>
* <span data-ttu-id="83b67-126">Recupera solo la sezione di record da un'API esterna che corrisponde all'area visibile corrente, anziché scaricare tutti i dati dalla raccolta.</span><span class="sxs-lookup"><span data-stu-id="83b67-126">Only fetches the slice of records from an external API that correspond to the current visible region, instead of downloading all of the data from the collection.</span></span>

<span data-ttu-id="83b67-127">Il contenuto dell'elemento per il `Virtualize` componente può includere:</span><span class="sxs-lookup"><span data-stu-id="83b67-127">The item content for the `Virtualize` component can include:</span></span>

* <span data-ttu-id="83b67-128">HTML semplice e Razor codice, come illustrato nell'esempio precedente.</span><span class="sxs-lookup"><span data-stu-id="83b67-128">Plain HTML and Razor code, as the preceding example shows.</span></span>
* <span data-ttu-id="83b67-129">Uno o più Razor componenti.</span><span class="sxs-lookup"><span data-stu-id="83b67-129">One or more Razor components.</span></span>
* <span data-ttu-id="83b67-130">Combinazione di componenti HTML/ Razor e Razor .</span><span class="sxs-lookup"><span data-stu-id="83b67-130">A mix of HTML/Razor and Razor components.</span></span>

## <a name="item-provider-delegate"></a><span data-ttu-id="83b67-131">Delegato del provider di elementi</span><span class="sxs-lookup"><span data-stu-id="83b67-131">Item provider delegate</span></span>

<span data-ttu-id="83b67-132">Se non si desidera caricare tutti gli elementi in memoria, è possibile specificare un metodo delegato del provider di elementi per il parametro del componente <xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.ItemsProvider%2A?displayProperty=nameWithType> che recupera in modo asincrono gli elementi richiesti su richiesta.</span><span class="sxs-lookup"><span data-stu-id="83b67-132">If you don't want to load all of the items into memory, you can specify an items provider delegate method to the component's <xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.ItemsProvider%2A?displayProperty=nameWithType> parameter that asynchronously retrieves the requested items on demand.</span></span> <span data-ttu-id="83b67-133">Nell'esempio seguente il `LoadEmployees` metodo fornisce gli elementi al `Virtualize` componente:</span><span class="sxs-lookup"><span data-stu-id="83b67-133">In the following example, the `LoadEmployees` method provides the items to the `Virtualize` component:</span></span>

```razor
<Virtualize Context="employee" ItemsProvider="@LoadEmployees">
    <p>
        @employee.FirstName @employee.LastName has the 
        job title of @employee.JobTitle.
    </p>
</Virtualize>
```

<span data-ttu-id="83b67-134">Il provider Items riceve un oggetto <xref:Microsoft.AspNetCore.Components.Web.Virtualization.ItemsProviderRequest> che specifica il numero di elementi richiesto a partire da un indice iniziale specifico.</span><span class="sxs-lookup"><span data-stu-id="83b67-134">The items provider receives an <xref:Microsoft.AspNetCore.Components.Web.Virtualization.ItemsProviderRequest>, which specifies the required number of items starting at a specific start index.</span></span> <span data-ttu-id="83b67-135">Il provider Items recupera quindi gli elementi richiesti da un database o da un altro servizio e li restituisce come <xref:Microsoft.AspNetCore.Components.Web.Virtualization.ItemsProviderResult%601> insieme al numero totale di elementi.</span><span class="sxs-lookup"><span data-stu-id="83b67-135">The items provider then retrieves the requested items from a database or other service and returns them as an <xref:Microsoft.AspNetCore.Components.Web.Virtualization.ItemsProviderResult%601> along with a count of the total items.</span></span> <span data-ttu-id="83b67-136">Il provider di elementi può scegliere di recuperare gli elementi con ogni richiesta o di memorizzarli nella cache in modo che siano immediatamente disponibili.</span><span class="sxs-lookup"><span data-stu-id="83b67-136">The items provider can choose to retrieve the items with each request or cache them so that they're readily available.</span></span>

<span data-ttu-id="83b67-137">Un `Virtualize` componente può accettare solo **un'origine elemento** dai relativi parametri, quindi non tentare di usare contemporaneamente un provider di elementi e assegnare una raccolta a `Items` .</span><span class="sxs-lookup"><span data-stu-id="83b67-137">A `Virtualize` component can only accept **one item source** from its parameters, so don't attempt to simultaneously use an items provider and assign a collection to `Items`.</span></span> <span data-ttu-id="83b67-138">Se entrambi sono assegnati, <xref:System.InvalidOperationException> viene generata un'eccezione quando i parametri del componente vengono impostati in fase di esecuzione.</span><span class="sxs-lookup"><span data-stu-id="83b67-138">If both are assigned, an <xref:System.InvalidOperationException> is thrown when the component's parameters are set at runtime.</span></span>

<span data-ttu-id="83b67-139">Nell' `LoadEmployees` esempio di metodo seguente vengono caricati i dipendenti da un `EmployeeService` (non visualizzato):</span><span class="sxs-lookup"><span data-stu-id="83b67-139">The following `LoadEmployees` method example loads employees from an `EmployeeService` (not shown):</span></span>

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

<span data-ttu-id="83b67-140"><xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.RefreshDataAsync%2A?displayProperty=nameWithType> indica al componente di ririchiedere i dati dal relativo oggetto <xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.ItemsProvider%2A> .</span><span class="sxs-lookup"><span data-stu-id="83b67-140"><xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.RefreshDataAsync%2A?displayProperty=nameWithType> instructs the component to rerequest data from its <xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.ItemsProvider%2A>.</span></span> <span data-ttu-id="83b67-141">Questa operazione è utile quando i dati esterni cambiano.</span><span class="sxs-lookup"><span data-stu-id="83b67-141">This is useful when external data changes.</span></span> <span data-ttu-id="83b67-142">Non è necessario chiamare questo quando si usa <xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.Items%2A> .</span><span class="sxs-lookup"><span data-stu-id="83b67-142">There's no need to call this when using <xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.Items%2A>.</span></span>

## <a name="placeholder"></a><span data-ttu-id="83b67-143">Segnaposto</span><span class="sxs-lookup"><span data-stu-id="83b67-143">Placeholder</span></span>

<span data-ttu-id="83b67-144">Poiché la richiesta di elementi da un'origine dati remota potrebbe richiedere del tempo, è possibile eseguire il rendering di un segnaposto con contenuto dell'elemento:</span><span class="sxs-lookup"><span data-stu-id="83b67-144">Because requesting items from a remote data source might take some time, you have the option to render a placeholder with item content:</span></span>

* <span data-ttu-id="83b67-145">Usare <xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.Placeholder%2A> ( `<Placeholder>...</Placeholder>` ) per visualizzare il contenuto fino a quando i dati dell'elemento non sono disponibili.</span><span class="sxs-lookup"><span data-stu-id="83b67-145">Use a <xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.Placeholder%2A> (`<Placeholder>...</Placeholder>`) to display content until the item data is available.</span></span>
* <span data-ttu-id="83b67-146">Utilizzare <xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.ItemContent%2A?displayProperty=nameWithType> per impostare il modello di elemento per l'elenco.</span><span class="sxs-lookup"><span data-stu-id="83b67-146">Use <xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.ItemContent%2A?displayProperty=nameWithType> to set the item template for the list.</span></span>

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

## <a name="item-size"></a><span data-ttu-id="83b67-147">Dimensioni dell'elemento</span><span class="sxs-lookup"><span data-stu-id="83b67-147">Item size</span></span>

<span data-ttu-id="83b67-148">L'altezza di ogni elemento in pixel può essere impostata con <xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.ItemSize%2A?displayProperty=nameWithType> (impostazione predefinita: 50):</span><span class="sxs-lookup"><span data-stu-id="83b67-148">The height of each item in pixels can be set with <xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.ItemSize%2A?displayProperty=nameWithType> (default: 50):</span></span>

```razor
<Virtualize Context="employee" Items="@employees" ItemSize="25">
    ...
</Virtualize>
```

<span data-ttu-id="83b67-149">Per impostazione predefinita, il `Virtualize` componente misura le dimensioni effettive del rendering *dopo che* è stato eseguito il rendering iniziale.</span><span class="sxs-lookup"><span data-stu-id="83b67-149">By default, the `Virtualize` component measures the actual rendering size *after* the initial render occurs.</span></span> <span data-ttu-id="83b67-150">Usare <xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.ItemSize%2A> per fornire in anticipo le dimensioni esatte degli elementi per supportare le prestazioni di rendering iniziali accurate e per garantire la corretta posizione di scorrimento per i ricaricamenti della pagina.</span><span class="sxs-lookup"><span data-stu-id="83b67-150">Use <xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.ItemSize%2A> to provide an exact item size in advance to assist with accurate initial render performance and to ensure the correct scroll position for page reloads.</span></span> <span data-ttu-id="83b67-151">Se per impostazione predefinita <xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.ItemSize%2A> viene eseguito il rendering di alcuni elementi al di fuori della visualizzazione attualmente visibile, viene attivato un secondo nuovo rendering.</span><span class="sxs-lookup"><span data-stu-id="83b67-151">If the default <xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.ItemSize%2A> causes some items to render outside of the currently visible view, a second re-render is triggered.</span></span> <span data-ttu-id="83b67-152">Per mantenere correttamente la posizione di scorrimento del browser in un elenco virtualizzato, il rendering iniziale deve essere corretto.</span><span class="sxs-lookup"><span data-stu-id="83b67-152">To correctly maintain the browser's scroll position in a virtualized list, the initial render must be correct.</span></span> <span data-ttu-id="83b67-153">In caso contrario, gli utenti potrebbero visualizzare gli elementi non corretti.</span><span class="sxs-lookup"><span data-stu-id="83b67-153">If not, users might view the wrong items.</span></span> 

## <a name="overscan-count"></a><span data-ttu-id="83b67-154">Conteggio overscan</span><span class="sxs-lookup"><span data-stu-id="83b67-154">Overscan count</span></span>

<span data-ttu-id="83b67-155"><xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.OverscanCount%2A?displayProperty=nameWithType> determina il numero di elementi aggiuntivi che vengono sottoposti a rendering prima e dopo l'area visibile.</span><span class="sxs-lookup"><span data-stu-id="83b67-155"><xref:Microsoft.AspNetCore.Components.Web.Virtualization.Virtualize%601.OverscanCount%2A?displayProperty=nameWithType> determines how many additional items are rendered before and after the visible region.</span></span> <span data-ttu-id="83b67-156">Questa impostazione consente di ridurre la frequenza di rendering durante lo scorrimento.</span><span class="sxs-lookup"><span data-stu-id="83b67-156">This setting helps to reduce the frequency of rendering during scrolling.</span></span> <span data-ttu-id="83b67-157">Tuttavia, i valori più elevati comportano il rendering di più elementi nella pagina (impostazione predefinita: 3):</span><span class="sxs-lookup"><span data-stu-id="83b67-157">However, higher values result in more elements rendered in the page (default: 3):</span></span>

```razor
<Virtualize Context="employee" Items="@employees" OverscanCount="4">
    ...
</Virtualize>
```

## <a name="state-changes"></a><span data-ttu-id="83b67-158">Modifiche stato</span><span class="sxs-lookup"><span data-stu-id="83b67-158">State changes</span></span>

<span data-ttu-id="83b67-159">Quando si apportano modifiche agli elementi di cui è stato eseguito il rendering dal `Virtualize` componente, chiamare <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> per forzare nuovamente la valutazione e il rendering del componente.</span><span class="sxs-lookup"><span data-stu-id="83b67-159">When making changes to items rendered by the `Virtualize` component, call <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> to force re-evaluation and rerendering of the component.</span></span> <span data-ttu-id="83b67-160">Per altre informazioni, vedere <xref:blazor/components/rendering>.</span><span class="sxs-lookup"><span data-stu-id="83b67-160">For more information, see <xref:blazor/components/rendering>.</span></span>
