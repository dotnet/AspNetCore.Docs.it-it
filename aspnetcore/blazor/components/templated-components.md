---
title: ASP.NET Core Blazor componenti basati su modelli
author: guardrex
description: Informazioni su come i componenti basati su modelli possono accettare uno o più modelli di interfaccia utente come parametri, che possono quindi essere usati come parte della logica di rendering del componente.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 03/04/2021
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
uid: blazor/components/templated-components
ms.openlocfilehash: 6c94218f3808baca18f23a53688bafdd6354e760
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102589478"
---
# <a name="aspnet-core-blazor-templated-components"></a><span data-ttu-id="5da89-103">ASP.NET Core Blazor componenti basati su modelli</span><span class="sxs-lookup"><span data-stu-id="5da89-103">ASP.NET Core Blazor templated components</span></span>

<span data-ttu-id="5da89-104">I componenti basati su modelli sono componenti che accettano uno o più modelli di interfaccia utente come parametri, che possono quindi essere usati come parte della logica di rendering del componente.</span><span class="sxs-lookup"><span data-stu-id="5da89-104">Templated components are components that accept one or more UI templates as parameters, which can then be used as part of the component's rendering logic.</span></span> <span data-ttu-id="5da89-105">I componenti basati su modelli consentono di creare componenti di livello superiore più riutilizzabili rispetto ai componenti normali.</span><span class="sxs-lookup"><span data-stu-id="5da89-105">Templated components allow you to author higher-level components that are more reusable than regular components.</span></span> <span data-ttu-id="5da89-106">Di seguito sono riportati alcuni esempi:</span><span class="sxs-lookup"><span data-stu-id="5da89-106">A couple of examples include:</span></span>

* <span data-ttu-id="5da89-107">Componente della tabella che consente a un utente di specificare i modelli per l'intestazione, le righe e il piè di pagina della tabella.</span><span class="sxs-lookup"><span data-stu-id="5da89-107">A table component that allows a user to specify templates for the table's header, rows, and footer.</span></span>
* <span data-ttu-id="5da89-108">Componente di elenco che consente a un utente di specificare un modello per il rendering degli elementi in un elenco.</span><span class="sxs-lookup"><span data-stu-id="5da89-108">A list component that allows a user to specify a template for rendering items in a list.</span></span>

<span data-ttu-id="5da89-109">Un componente basato su modelli viene definito specificando uno o più parametri del componente di tipo <xref:Microsoft.AspNetCore.Components.RenderFragment> o <xref:Microsoft.AspNetCore.Components.RenderFragment%601> .</span><span class="sxs-lookup"><span data-stu-id="5da89-109">A templated component is defined by specifying one or more component parameters of type <xref:Microsoft.AspNetCore.Components.RenderFragment> or <xref:Microsoft.AspNetCore.Components.RenderFragment%601>.</span></span> <span data-ttu-id="5da89-110">Un frammento di rendering rappresenta un segmento di interfaccia utente di cui eseguire il rendering.</span><span class="sxs-lookup"><span data-stu-id="5da89-110">A render fragment represents a segment of UI to render.</span></span> <span data-ttu-id="5da89-111"><xref:Microsoft.AspNetCore.Components.RenderFragment%601> accetta un parametro di tipo che può essere specificato quando viene richiamato il frammento di rendering.</span><span class="sxs-lookup"><span data-stu-id="5da89-111"><xref:Microsoft.AspNetCore.Components.RenderFragment%601> takes a type parameter that can be specified when the render fragment is invoked.</span></span>

<span data-ttu-id="5da89-112">Spesso, i componenti basati su modelli sono tipizzati in modo generico, come `TableTemplate` illustrato nel componente seguente.</span><span class="sxs-lookup"><span data-stu-id="5da89-112">Often, templated components are generically typed, as the following `TableTemplate` component demonstrates.</span></span> <span data-ttu-id="5da89-113">Il tipo generico `<T>` in questo esempio viene usato per eseguire il rendering `IReadOnlyList<T>` dei valori, che in questo caso è una serie di righe PET in un componente che visualizza una tabella di animali domestici.</span><span class="sxs-lookup"><span data-stu-id="5da89-113">The generic type `<T>` in this example is used to render `IReadOnlyList<T>` values, which in this case is a series of pet rows in a component that displays a table of pets.</span></span>

<span data-ttu-id="5da89-114">`Shared/TableTemplate.razor`:</span><span class="sxs-lookup"><span data-stu-id="5da89-114">`Shared/TableTemplate.razor`:</span></span>

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Shared/templated-components/TableTemplate.razor)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Shared/templated-components/TableTemplate.razor)]

::: moniker-end

<span data-ttu-id="5da89-115">Quando si usa un componente basato su modelli, è possibile specificare i parametri del modello usando gli elementi figlio che corrispondono ai nomi dei parametri.</span><span class="sxs-lookup"><span data-stu-id="5da89-115">When using a templated component, the template parameters can be specified using child elements that match the names of the parameters.</span></span> <span data-ttu-id="5da89-116">Nell'esempio seguente, `<TableHeader>...</TableHeader>` e `<RowTemplate>...<RowTemplate>` forniscono <xref:Microsoft.AspNetCore.Components.RenderFragment%601> i modelli per `TableHeader` e `RowTemplate` del `TableTemplate` componente.</span><span class="sxs-lookup"><span data-stu-id="5da89-116">In the following example, `<TableHeader>...</TableHeader>` and `<RowTemplate>...<RowTemplate>` supply <xref:Microsoft.AspNetCore.Components.RenderFragment%601> templates for `TableHeader` and `RowTemplate` of the `TableTemplate` component.</span></span>

<span data-ttu-id="5da89-117">Specificare l' `Context` attributo sull'elemento Component quando si desidera specificare il nome del parametro di contenuto per il contenuto figlio implicito (senza alcun elemento figlio di wrapping).</span><span class="sxs-lookup"><span data-stu-id="5da89-117">Specify the `Context` attribute on the component element when you want to specify the content parameter name for implicit child content (without any wrapping child element).</span></span> <span data-ttu-id="5da89-118">Nell'esempio seguente l' `Context` attributo viene visualizzato nell' `TableTemplate` elemento e si applica a tutti i <xref:Microsoft.AspNetCore.Components.RenderFragment%601> parametri del modello.</span><span class="sxs-lookup"><span data-stu-id="5da89-118">In the following example, the `Context` attribute appears on the `TableTemplate` element and applies to all <xref:Microsoft.AspNetCore.Components.RenderFragment%601> template parameters.</span></span>

<span data-ttu-id="5da89-119">`Pages/Pets.razor`:</span><span class="sxs-lookup"><span data-stu-id="5da89-119">`Pages/Pets.razor`:</span></span>

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/templated-components/Pets1.razor)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/templated-components/Pets1.razor)]

::: moniker-end

<span data-ttu-id="5da89-120">In alternativa, è possibile modificare il nome del parametro usando l' `Context` attributo nell' <xref:Microsoft.AspNetCore.Components.RenderFragment%601> elemento figlio.</span><span class="sxs-lookup"><span data-stu-id="5da89-120">Alternatively, you can change the parameter name using the `Context` attribute on the <xref:Microsoft.AspNetCore.Components.RenderFragment%601> child element.</span></span> <span data-ttu-id="5da89-121">Nell'esempio seguente, l'oggetto `Context` è impostato su `RowTemplate` anziché su `TableTemplate` :</span><span class="sxs-lookup"><span data-stu-id="5da89-121">In the following example, the `Context` is set on `RowTemplate` rather than `TableTemplate`:</span></span>

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/templated-components/Pets2.razor?name=snippet&highlight=6)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/templated-components/Pets2.razor?name=snippet&highlight=6)]

::: moniker-end

<span data-ttu-id="5da89-122">Gli argomenti del componente di tipo <xref:Microsoft.AspNetCore.Components.RenderFragment%601> hanno un parametro implicito denominato `context` , che può essere usato.</span><span class="sxs-lookup"><span data-stu-id="5da89-122">Component arguments of type <xref:Microsoft.AspNetCore.Components.RenderFragment%601> have an implicit parameter named `context`, which can be used.</span></span> <span data-ttu-id="5da89-123">Nell'esempio seguente `Context` non è impostato.</span><span class="sxs-lookup"><span data-stu-id="5da89-123">In the following example, `Context` isn't set.</span></span> <span data-ttu-id="5da89-124">`@context.{PROPERTY}` fornisce valori di PET al modello, dove `{PROPERTY}` è una `Pet` proprietà:</span><span class="sxs-lookup"><span data-stu-id="5da89-124">`@context.{PROPERTY}` supplies pet values to the template, where `{PROPERTY}` is a `Pet` property:</span></span>

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/templated-components/Pets3.razor?name=snippet&highlight=7-8)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/templated-components/Pets3.razor?name=snippet&highlight=7-8)]

::: moniker-end

<span data-ttu-id="5da89-125">Quando si usano componenti tipizzati generici, il parametro di tipo viene dedotto, se possibile.</span><span class="sxs-lookup"><span data-stu-id="5da89-125">When using generic-typed components, the type parameter is inferred if possible.</span></span> <span data-ttu-id="5da89-126">Tuttavia, è possibile specificare in modo esplicito il tipo con un attributo il cui nome corrisponde al parametro di tipo, che è `TItem` riportato nell'esempio precedente:</span><span class="sxs-lookup"><span data-stu-id="5da89-126">However, you can explicitly specify the type with an attribute that has a name matching the type parameter, which is `TItem` in the preceding example:</span></span>

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/templated-components/Pets4.razor?name=snippet&highlight=1)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/templated-components/Pets4.razor?name=snippet&highlight=1)]

::: moniker-end

## <a name="generic-type-constraints"></a><span data-ttu-id="5da89-127">Vincoli di tipo generico</span><span class="sxs-lookup"><span data-stu-id="5da89-127">Generic type constraints</span></span>

> [!NOTE]
> <span data-ttu-id="5da89-128">I vincoli di tipo generico saranno supportati in una versione futura.</span><span class="sxs-lookup"><span data-stu-id="5da89-128">Generic type constraints will be supported in a future release.</span></span> <span data-ttu-id="5da89-129">Per ulteriori informazioni, vedere [Consenti vincoli di tipo generico (DotNet/aspnetcore #8433)](https://github.com/dotnet/aspnetcore/issues/8433).</span><span class="sxs-lookup"><span data-stu-id="5da89-129">For more information, see [Allow generic type constraints (dotnet/aspnetcore #8433)](https://github.com/dotnet/aspnetcore/issues/8433).</span></span>

## <a name="additional-resources"></a><span data-ttu-id="5da89-130">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="5da89-130">Additional resources</span></span>

* <xref:blazor/webassembly-performance-best-practices#define-reusable-renderfragments-in-code>
