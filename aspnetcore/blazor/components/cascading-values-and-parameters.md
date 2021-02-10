---
title: BlazorParametri e valori di propagazione ASP.NET Core
author: guardrex
description: Informazioni su come eseguire il flusso dei dati da un componente predecessore ai componenti discendenti.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 02/02/2021
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
uid: blazor/components/cascading-values-and-parameters
ms.openlocfilehash: 9b667ff83bf6dd9b400805eff403c8c3f5c7b82a
ms.sourcegitcommit: 04ad9cd26fcaa8bd11e261d3661f375f5f343cdc
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/10/2021
ms.locfileid: "100107090"
---
# <a name="aspnet-core-blazor-cascading-values-and-parameters"></a><span data-ttu-id="a2e10-103">BlazorParametri e valori di propagazione ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="a2e10-103">ASP.NET Core Blazor cascading values and parameters</span></span>

<span data-ttu-id="a2e10-104">Di [Luke Latham](https://github.com/guardrex) e [Daniel Roth](https://github.com/danroth27)</span><span class="sxs-lookup"><span data-stu-id="a2e10-104">By [Luke Latham](https://github.com/guardrex) and [Daniel Roth](https://github.com/danroth27)</span></span>

<span data-ttu-id="a2e10-105">*I valori e i parametri* di propagazione forniscono un metodo convienent per il flusso dei dati in una gerarchia di componenti da un componente predecessore a un numero qualsiasi di componenti decendent.</span><span class="sxs-lookup"><span data-stu-id="a2e10-105">*Cascading values and parameters* provide a convienent way to flow data down a component hierarchy from an ancestor component to any number of decendent components.</span></span> <span data-ttu-id="a2e10-106">A differenza dei [parametri del componente](xref:blazor/components/index#component-parameters), i parametri e i valori di propagazione non richiedono l'assegnazione di attributi per ogni componente discendente in cui vengono utilizzati i dati.</span><span class="sxs-lookup"><span data-stu-id="a2e10-106">Unlike [Component parameters](xref:blazor/components/index#component-parameters), cascading values and parameters don't require an attribute assignment for each descendent component where the data is consumed.</span></span> <span data-ttu-id="a2e10-107">I parametri e i valori di propagazione consentono inoltre ai componenti di coordinarsi tra loro in una gerarchia di componenti.</span><span class="sxs-lookup"><span data-stu-id="a2e10-107">Cascading values and parameters also allow components to coordinate with each other across a component hierarchy.</span></span>

## <a name="cascadingvalue-component"></a><span data-ttu-id="a2e10-108">Componente `CascadingValue`</span><span class="sxs-lookup"><span data-stu-id="a2e10-108">`CascadingValue` component</span></span>

<span data-ttu-id="a2e10-109">Un componente predecessore fornisce un valore di propagazione utilizzando il Blazor componente del Framework [`CascadingValue`](xref:Microsoft.AspNetCore.Components.CascadingValue%601) , che esegue il wrapping di un sottoalbero di una gerarchia di componenti e fornisce un singolo valore a tutti i componenti all'interno del sottoalbero.</span><span class="sxs-lookup"><span data-stu-id="a2e10-109">An ancestor component provides a cascading value using the Blazor framework's [`CascadingValue`](xref:Microsoft.AspNetCore.Components.CascadingValue%601) component, which wraps a subtree of a component hierarchy and supplies a single value to all of the components within its subtree.</span></span>

<span data-ttu-id="a2e10-110">Nell'esempio seguente viene illustrato il flusso delle informazioni sui temi nella gerarchia dei componenti di un componente di layout per fornire una classe di stile CSS ai pulsanti nei componenti figlio.</span><span class="sxs-lookup"><span data-stu-id="a2e10-110">The following example demonstrates the flow of theme information down the component hierarchy of a layout component to provide a CSS style class to buttons in child components.</span></span>

<span data-ttu-id="a2e10-111">La `ThemeInfo` classe C# seguente viene posizionata in una cartella denominata `UIThemeClasses` e specifica le informazioni sul tema.</span><span class="sxs-lookup"><span data-stu-id="a2e10-111">The following `ThemeInfo` C# class is placed in a folder named `UIThemeClasses` and specifies the theme information.</span></span>

> [!NOTE]
> <span data-ttu-id="a2e10-112">Per gli esempi in questa sezione, lo spazio dei nomi dell'app è `BlazorSample` .</span><span class="sxs-lookup"><span data-stu-id="a2e10-112">For the examples in this section, the app's namespace is `BlazorSample`.</span></span> <span data-ttu-id="a2e10-113">Quando si sperimenta il codice nella propria app di esempio, modificare lo spazio dei nomi dell'app nello spazio dei nomi dell'app di esempio.</span><span class="sxs-lookup"><span data-stu-id="a2e10-113">When experimenting with the code in your own sample app, change the app's namespace to your sample app's namespace.</span></span>

<span data-ttu-id="a2e10-114">`UIThemeClasses/ThemeInfo.cs`:</span><span class="sxs-lookup"><span data-stu-id="a2e10-114">`UIThemeClasses/ThemeInfo.cs`:</span></span>

```csharp
namespace BlazorSample.UIThemeClasses
{
    public class ThemeInfo
    {
        public string ButtonClass { get; set; }
    }
}
```

<span data-ttu-id="a2e10-115">Il [componente layout](xref:blazor/layouts) seguente specifica le informazioni sul tema ( `ThemeInfo` ) come valore di propagazione per tutti i componenti che costituiscono il corpo del layout della <xref:Microsoft.AspNetCore.Components.LayoutComponentBase.Body> Proprietà.</span><span class="sxs-lookup"><span data-stu-id="a2e10-115">The following [layout component](xref:blazor/layouts) specifies theme information (`ThemeInfo`) as a cascading value for all components that make up the layout body of the <xref:Microsoft.AspNetCore.Components.LayoutComponentBase.Body> property.</span></span> <span data-ttu-id="a2e10-116">`ButtonClass` viene assegnato un valore di [`btn-success`](https://getbootstrap.com/docs/5.0/components/buttons/) , che è uno stile del pulsante bootstrap.</span><span class="sxs-lookup"><span data-stu-id="a2e10-116">`ButtonClass` is assigned a value of [`btn-success`](https://getbootstrap.com/docs/5.0/components/buttons/), which is a Bootstrap button style.</span></span> <span data-ttu-id="a2e10-117">Qualsiasi componente discendente nella gerarchia dei componenti può utilizzare la `ButtonClass` proprietà tramite il valore di propagazione `ThemeInfo` .</span><span class="sxs-lookup"><span data-stu-id="a2e10-117">Any descendent component in the component hierarchy can use the `ButtonClass` property through the `ThemeInfo` cascading value.</span></span>

<span data-ttu-id="a2e10-118">`Shared/MainLayout.razor`:</span><span class="sxs-lookup"><span data-stu-id="a2e10-118">`Shared/MainLayout.razor`:</span></span>

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Shared/MainLayout.razor?highlight=2,10-14,19)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Shared/MainLayout.razor?highlight=2,9-13,17)]

::: moniker-end

## <a name="cascadingparameter-attribute"></a><span data-ttu-id="a2e10-119">Attributo `[CascadingParameter]`</span><span class="sxs-lookup"><span data-stu-id="a2e10-119">`[CascadingParameter]` attribute</span></span>

<span data-ttu-id="a2e10-120">Per utilizzare i valori di propagazione, i componenti discendenti dichiarano i parametri di propagazione utilizzando l' [ `[CascadingParameter]` attributo](xref:Microsoft.AspNetCore.Components.CascadingParameterAttribute).</span><span class="sxs-lookup"><span data-stu-id="a2e10-120">To make use of cascading values, descendent components declare cascading parameters using the [`[CascadingParameter]` attribute](xref:Microsoft.AspNetCore.Components.CascadingParameterAttribute).</span></span> <span data-ttu-id="a2e10-121">I valori a cascata vengono associati ai parametri di propagazione per **tipo**.</span><span class="sxs-lookup"><span data-stu-id="a2e10-121">Cascading values are bound to cascading parameters **by type**.</span></span> <span data-ttu-id="a2e10-122">La propagazione di più valori dello stesso tipo viene illustrata nella sezione [Cascade multiple values](#cascade-multiple-values) più avanti in questo articolo.</span><span class="sxs-lookup"><span data-stu-id="a2e10-122">Cascading multiple values of the same type is covered in the [Cascade multiple values](#cascade-multiple-values) section later in this article.</span></span>

<span data-ttu-id="a2e10-123">Il componente seguente associa il valore di propagazione `ThemeInfo` a un parametro di propagazione, utilizzando facoltativamente lo stesso nome di `ThemeInfo` .</span><span class="sxs-lookup"><span data-stu-id="a2e10-123">The following component binds the `ThemeInfo` cascading value to a cascading parameter, optionally using the same name of `ThemeInfo`.</span></span> <span data-ttu-id="a2e10-124">Il parametro viene usato per impostare la classe CSS per il **`Increment Counter (Themed)`** pulsante.</span><span class="sxs-lookup"><span data-stu-id="a2e10-124">The parameter is used to set the CSS class for the **`Increment Counter (Themed)`** button.</span></span>

<span data-ttu-id="a2e10-125">`Pages/ThemedCounter.razor`:</span><span class="sxs-lookup"><span data-stu-id="a2e10-125">`Pages/ThemedCounter.razor`:</span></span>

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/ThemedCounter.razor?highlight=2,15-17,23-24)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Pages/ThemedCounter.razor?highlight=2,15-17,23-24)]

::: moniker-end

## <a name="cascade-multiple-values"></a><span data-ttu-id="a2e10-126">Propagazione di più valori</span><span class="sxs-lookup"><span data-stu-id="a2e10-126">Cascade multiple values</span></span>

<span data-ttu-id="a2e10-127">Per eseguire il propagazione di più valori dello stesso tipo all'interno dello stesso sottoalbero, fornire una stringa univoca <xref:Microsoft.AspNetCore.Components.CascadingValue%601.Name%2A> a ogni [`CascadingValue`](xref:Microsoft.AspNetCore.Components.CascadingValue%601) componente e ai relativi [ `[CascadingParameter]` attributi](xref:Microsoft.AspNetCore.Components.CascadingParameterAttribute)corrispondenti.</span><span class="sxs-lookup"><span data-stu-id="a2e10-127">To cascade multiple values of the same type within the same subtree, provide a unique <xref:Microsoft.AspNetCore.Components.CascadingValue%601.Name%2A> string to each [`CascadingValue`](xref:Microsoft.AspNetCore.Components.CascadingValue%601) component and their corresponding [`[CascadingParameter]` attributes](xref:Microsoft.AspNetCore.Components.CascadingParameterAttribute).</span></span>

<span data-ttu-id="a2e10-128">Nell'esempio seguente due componenti propagano [`CascadingValue`](xref:Microsoft.AspNetCore.Components.CascadingValue%601) istanze diverse di `CascadingType` :</span><span class="sxs-lookup"><span data-stu-id="a2e10-128">In the following example, two [`CascadingValue`](xref:Microsoft.AspNetCore.Components.CascadingValue%601) components cascade different instances of `CascadingType`:</span></span>

```razor
<CascadingValue Value="@parentCascadeParameter1" Name="CascadeParam1">
    <CascadingValue Value="@ParentCascadeParameter2" Name="CascadeParam2">
        ...
    </CascadingValue>
</CascadingValue>

@code {
    private CascadingType parentCascadeParameter1;

    [Parameter]
    public CascadingType ParentCascadeParameter2 { get; set; }

    ...
}
```

<span data-ttu-id="a2e10-129">In un componente discendente i parametri a cascata ricevono i valori a cascata dal componente predecessore da <xref:Microsoft.AspNetCore.Components.CascadingValue%601.Name%2A> :</span><span class="sxs-lookup"><span data-stu-id="a2e10-129">In a descendant component, the cascaded parameters receive their cascaded values from the ancestor component by <xref:Microsoft.AspNetCore.Components.CascadingValue%601.Name%2A>:</span></span>

```razor
...

@code {
    [CascadingParameter(Name = "CascadeParam1")]
    protected CascadingType ChildCascadeParameter1 { get; set; }
    
    [CascadingParameter(Name = "CascadeParam2")]
    protected CascadingType ChildCascadeParameter2 { get; set; }
}
```

## <a name="pass-data-across-a-component-hierarchy"></a><span data-ttu-id="a2e10-130">Passare dati in una gerarchia di componenti</span><span class="sxs-lookup"><span data-stu-id="a2e10-130">Pass data across a component hierarchy</span></span>

<span data-ttu-id="a2e10-131">I parametri di propagazione consentono inoltre ai componenti di passare i dati attraverso una gerarchia di componenti.</span><span class="sxs-lookup"><span data-stu-id="a2e10-131">Cascading parameters also enable components to pass data across a component hierarchy.</span></span> <span data-ttu-id="a2e10-132">Si consideri il seguente esempio di set di schede dell'interfaccia utente, in cui un componente del set di schede mantiene una serie di singole schede.</span><span class="sxs-lookup"><span data-stu-id="a2e10-132">Consider the following UI tab set example, where a tab set component maintains a series of individual tabs.</span></span>

> [!NOTE]
> <span data-ttu-id="a2e10-133">Per gli esempi in questa sezione, lo spazio dei nomi dell'app è `BlazorSample` .</span><span class="sxs-lookup"><span data-stu-id="a2e10-133">For the examples in this section, the app's namespace is `BlazorSample`.</span></span> <span data-ttu-id="a2e10-134">Quando si sperimenta il codice nella propria app di esempio, modificare lo spazio dei nomi nello spazio dei nomi dell'app di esempio.</span><span class="sxs-lookup"><span data-stu-id="a2e10-134">When experimenting with the code in your own sample app, change the namespace to your sample app's namespace.</span></span>

<span data-ttu-id="a2e10-135">Creare un' `ITab` interfaccia implementata dalle schede in una cartella denominata `UIInterfaces` .</span><span class="sxs-lookup"><span data-stu-id="a2e10-135">Create an `ITab` interface that tabs implement in a folder named `UIInterfaces`.</span></span>

<span data-ttu-id="a2e10-136">`UIInterfaces/ITab.cs`:</span><span class="sxs-lookup"><span data-stu-id="a2e10-136">`UIInterfaces/ITab.cs`:</span></span>

```csharp
using Microsoft.AspNetCore.Components;

namespace BlazorSample.UIInterfaces
{
    public interface ITab
    {
        RenderFragment ChildContent { get; }
    }
}
```

<span data-ttu-id="a2e10-137">Il `TabSet` componente seguente mantiene un set di schede.</span><span class="sxs-lookup"><span data-stu-id="a2e10-137">The following `TabSet` component maintains a set of tabs.</span></span> <span data-ttu-id="a2e10-138">I componenti del set di schede `Tab` , che vengono creati più avanti in questa sezione, forniscono gli elementi elenco ( `<li>...</li>` ) per l'elenco ( `<ul>...</ul>` ).</span><span class="sxs-lookup"><span data-stu-id="a2e10-138">The tab set's `Tab` components, which are created later in this section, supply the list items (`<li>...</li>`) for the list (`<ul>...</ul>`).</span></span>

<span data-ttu-id="a2e10-139">`Tab`I componenti figlio non vengono passati in modo esplicito come parametri a `TabSet` .</span><span class="sxs-lookup"><span data-stu-id="a2e10-139">Child `Tab` components aren't explicitly passed as parameters to the `TabSet`.</span></span> <span data-ttu-id="a2e10-140">Al contrario, i `Tab` componenti figlio fanno parte del contenuto figlio di `TabSet` .</span><span class="sxs-lookup"><span data-stu-id="a2e10-140">Instead, the child `Tab` components are part of the child content of the `TabSet`.</span></span> <span data-ttu-id="a2e10-141">Tuttavia, `TabSet` per è comunque necessario un riferimento a ogni componente, in `Tab` modo che sia possibile eseguire il rendering delle intestazioni e della scheda attiva. Per abilitare questo coordinamento senza richiedere codice aggiuntivo, il `TabSet` componente *può fornire se stesso come valore* di propagazione che viene quindi prelevato dai `Tab` componenti discendenti.</span><span class="sxs-lookup"><span data-stu-id="a2e10-141">However, the `TabSet` still needs a reference each `Tab` component so that it can render the headers and the active tab. To enable this coordination without requiring additional code, the `TabSet` component *can provide itself as a cascading value* that is then picked up by the descendent `Tab` components.</span></span>

<span data-ttu-id="a2e10-142">`Shared/TabSet.razor`:</span><span class="sxs-lookup"><span data-stu-id="a2e10-142">`Shared/TabSet.razor`:</span></span>

```razor
@using BlazorSample.UIInterfaces

<!-- Display the tab headers -->

<CascadingValue Value=this>
    <ul class="nav nav-tabs">
        @ChildContent
    </ul>
</CascadingValue>

<!-- Display body for only the active tab -->

<div class="nav-tabs-body p-4">
    @ActiveTab?.ChildContent
</div>

@code {
    [Parameter]
    public RenderFragment ChildContent { get; set; }

    public ITab ActiveTab { get; private set; }

    public void AddTab(ITab tab)
    {
        if (ActiveTab == null)
        {
            SetActiveTab(tab);
        }
    }

    public void SetActiveTab(ITab tab)
    {
        if (ActiveTab != tab)
        {
            ActiveTab = tab;
            StateHasChanged();
        }
    }
}
```

<span data-ttu-id="a2e10-143">`Tab`I componenti discendenti acquisiscono l'oggetto che contiene `TabSet` come parametro a cascata.</span><span class="sxs-lookup"><span data-stu-id="a2e10-143">Descendent `Tab` components capture the containing `TabSet` as a cascading parameter.</span></span> <span data-ttu-id="a2e10-144">I `Tab` componenti si aggiungono alla `TabSet` coordinata e per impostare la scheda attiva.</span><span class="sxs-lookup"><span data-stu-id="a2e10-144">The `Tab` components add themselves to the `TabSet` and coordinate to set the active tab.</span></span>

<span data-ttu-id="a2e10-145">`Shared/Tab.razor`:</span><span class="sxs-lookup"><span data-stu-id="a2e10-145">`Shared/Tab.razor`:</span></span>

```razor
@using BlazorSample.UIInterfaces
@implements ITab

<li>
    <a @onclick="ActivateTab" class="nav-link @TitleCssClass" role="button">
        @Title
    </a>
</li>

@code {
    [CascadingParameter]
    public TabSet ContainerTabSet { get; set; }

    [Parameter]
    public string Title { get; set; }

    [Parameter]
    public RenderFragment ChildContent { get; set; }

    private string TitleCssClass => 
        ContainerTabSet.ActiveTab == this ? "active" : null;

    protected override void OnInitialized()
    {
        ContainerTabSet.AddTab(this);
    }

    private void ActivateTab()
    {
        ContainerTabSet.SetActiveTab(this);
    }
}
```

<span data-ttu-id="a2e10-146">Il `ExampleTabSet` componente seguente usa il `TabSet` componente, che contiene tre `Tab` componenti.</span><span class="sxs-lookup"><span data-stu-id="a2e10-146">The following `ExampleTabSet` component uses the `TabSet` component, which contains three `Tab` components.</span></span>

<span data-ttu-id="a2e10-147">`Pages/ExampleTabSet.razor`:</span><span class="sxs-lookup"><span data-stu-id="a2e10-147">`Pages/ExampleTabSet.razor`:</span></span>

```razor
@page "/example-tab-set"

<TabSet>
    <Tab Title="First tab">
        <h4>Greetings from the first tab!</h4>

        <label>
            <input type="checkbox" @bind="showThirdTab" />
            Toggle third tab
        </label>
    </Tab>

    <Tab Title="Second tab">
        <h4>Hello from the second tab!</h4>
    </Tab>

    @if (showThirdTab)
    {
        <Tab Title="Third tab">
            <h4>Welcome to the disappearing third tab!</h4>
            <p>Toggle this tab from the first tab.</p>
        </Tab>
    }
</TabSet>

@code {
    private bool showThirdTab;
}
```
