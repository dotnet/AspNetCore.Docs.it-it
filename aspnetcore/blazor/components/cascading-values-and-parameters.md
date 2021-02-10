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
# <a name="aspnet-core-blazor-cascading-values-and-parameters"></a>BlazorParametri e valori di propagazione ASP.NET Core

Di [Luke Latham](https://github.com/guardrex) e [Daniel Roth](https://github.com/danroth27)

*I valori e i parametri* di propagazione forniscono un metodo convienent per il flusso dei dati in una gerarchia di componenti da un componente predecessore a un numero qualsiasi di componenti decendent. A differenza dei [parametri del componente](xref:blazor/components/index#component-parameters), i parametri e i valori di propagazione non richiedono l'assegnazione di attributi per ogni componente discendente in cui vengono utilizzati i dati. I parametri e i valori di propagazione consentono inoltre ai componenti di coordinarsi tra loro in una gerarchia di componenti.

## <a name="cascadingvalue-component"></a>Componente `CascadingValue`

Un componente predecessore fornisce un valore di propagazione utilizzando il Blazor componente del Framework [`CascadingValue`](xref:Microsoft.AspNetCore.Components.CascadingValue%601) , che esegue il wrapping di un sottoalbero di una gerarchia di componenti e fornisce un singolo valore a tutti i componenti all'interno del sottoalbero.

Nell'esempio seguente viene illustrato il flusso delle informazioni sui temi nella gerarchia dei componenti di un componente di layout per fornire una classe di stile CSS ai pulsanti nei componenti figlio.

La `ThemeInfo` classe C# seguente viene posizionata in una cartella denominata `UIThemeClasses` e specifica le informazioni sul tema.

> [!NOTE]
> Per gli esempi in questa sezione, lo spazio dei nomi dell'app è `BlazorSample` . Quando si sperimenta il codice nella propria app di esempio, modificare lo spazio dei nomi dell'app nello spazio dei nomi dell'app di esempio.

`UIThemeClasses/ThemeInfo.cs`:

```csharp
namespace BlazorSample.UIThemeClasses
{
    public class ThemeInfo
    {
        public string ButtonClass { get; set; }
    }
}
```

Il [componente layout](xref:blazor/layouts) seguente specifica le informazioni sul tema ( `ThemeInfo` ) come valore di propagazione per tutti i componenti che costituiscono il corpo del layout della <xref:Microsoft.AspNetCore.Components.LayoutComponentBase.Body> Proprietà. `ButtonClass` viene assegnato un valore di [`btn-success`](https://getbootstrap.com/docs/5.0/components/buttons/) , che è uno stile del pulsante bootstrap. Qualsiasi componente discendente nella gerarchia dei componenti può utilizzare la `ButtonClass` proprietà tramite il valore di propagazione `ThemeInfo` .

`Shared/MainLayout.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Shared/MainLayout.razor?highlight=2,10-14,19)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Shared/MainLayout.razor?highlight=2,9-13,17)]

::: moniker-end

## <a name="cascadingparameter-attribute"></a>Attributo `[CascadingParameter]`

Per utilizzare i valori di propagazione, i componenti discendenti dichiarano i parametri di propagazione utilizzando l' [ `[CascadingParameter]` attributo](xref:Microsoft.AspNetCore.Components.CascadingParameterAttribute). I valori a cascata vengono associati ai parametri di propagazione per **tipo**. La propagazione di più valori dello stesso tipo viene illustrata nella sezione [Cascade multiple values](#cascade-multiple-values) più avanti in questo articolo.

Il componente seguente associa il valore di propagazione `ThemeInfo` a un parametro di propagazione, utilizzando facoltativamente lo stesso nome di `ThemeInfo` . Il parametro viene usato per impostare la classe CSS per il **`Increment Counter (Themed)`** pulsante.

`Pages/ThemedCounter.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/ThemedCounter.razor?highlight=2,15-17,23-24)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Pages/ThemedCounter.razor?highlight=2,15-17,23-24)]

::: moniker-end

## <a name="cascade-multiple-values"></a>Propagazione di più valori

Per eseguire il propagazione di più valori dello stesso tipo all'interno dello stesso sottoalbero, fornire una stringa univoca <xref:Microsoft.AspNetCore.Components.CascadingValue%601.Name%2A> a ogni [`CascadingValue`](xref:Microsoft.AspNetCore.Components.CascadingValue%601) componente e ai relativi [ `[CascadingParameter]` attributi](xref:Microsoft.AspNetCore.Components.CascadingParameterAttribute)corrispondenti.

Nell'esempio seguente due componenti propagano [`CascadingValue`](xref:Microsoft.AspNetCore.Components.CascadingValue%601) istanze diverse di `CascadingType` :

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

In un componente discendente i parametri a cascata ricevono i valori a cascata dal componente predecessore da <xref:Microsoft.AspNetCore.Components.CascadingValue%601.Name%2A> :

```razor
...

@code {
    [CascadingParameter(Name = "CascadeParam1")]
    protected CascadingType ChildCascadeParameter1 { get; set; }
    
    [CascadingParameter(Name = "CascadeParam2")]
    protected CascadingType ChildCascadeParameter2 { get; set; }
}
```

## <a name="pass-data-across-a-component-hierarchy"></a>Passare dati in una gerarchia di componenti

I parametri di propagazione consentono inoltre ai componenti di passare i dati attraverso una gerarchia di componenti. Si consideri il seguente esempio di set di schede dell'interfaccia utente, in cui un componente del set di schede mantiene una serie di singole schede.

> [!NOTE]
> Per gli esempi in questa sezione, lo spazio dei nomi dell'app è `BlazorSample` . Quando si sperimenta il codice nella propria app di esempio, modificare lo spazio dei nomi nello spazio dei nomi dell'app di esempio.

Creare un' `ITab` interfaccia implementata dalle schede in una cartella denominata `UIInterfaces` .

`UIInterfaces/ITab.cs`:

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

Il `TabSet` componente seguente mantiene un set di schede. I componenti del set di schede `Tab` , che vengono creati più avanti in questa sezione, forniscono gli elementi elenco ( `<li>...</li>` ) per l'elenco ( `<ul>...</ul>` ).

`Tab`I componenti figlio non vengono passati in modo esplicito come parametri a `TabSet` . Al contrario, i `Tab` componenti figlio fanno parte del contenuto figlio di `TabSet` . Tuttavia, `TabSet` per è comunque necessario un riferimento a ogni componente, in `Tab` modo che sia possibile eseguire il rendering delle intestazioni e della scheda attiva. Per abilitare questo coordinamento senza richiedere codice aggiuntivo, il `TabSet` componente *può fornire se stesso come valore* di propagazione che viene quindi prelevato dai `Tab` componenti discendenti.

`Shared/TabSet.razor`:

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

`Tab`I componenti discendenti acquisiscono l'oggetto che contiene `TabSet` come parametro a cascata. I `Tab` componenti si aggiungono alla `TabSet` coordinata e per impostare la scheda attiva.

`Shared/Tab.razor`:

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

Il `ExampleTabSet` componente seguente usa il `TabSet` componente, che contiene tre `Tab` componenti.

`Pages/ExampleTabSet.razor`:

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
