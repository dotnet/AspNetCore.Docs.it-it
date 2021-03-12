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
# <a name="aspnet-core-blazor-templated-components"></a>ASP.NET Core Blazor componenti basati su modelli

I componenti basati su modelli sono componenti che accettano uno o più modelli di interfaccia utente come parametri, che possono quindi essere usati come parte della logica di rendering del componente. I componenti basati su modelli consentono di creare componenti di livello superiore più riutilizzabili rispetto ai componenti normali. Di seguito sono riportati alcuni esempi:

* Componente della tabella che consente a un utente di specificare i modelli per l'intestazione, le righe e il piè di pagina della tabella.
* Componente di elenco che consente a un utente di specificare un modello per il rendering degli elementi in un elenco.

Un componente basato su modelli viene definito specificando uno o più parametri del componente di tipo <xref:Microsoft.AspNetCore.Components.RenderFragment> o <xref:Microsoft.AspNetCore.Components.RenderFragment%601> . Un frammento di rendering rappresenta un segmento di interfaccia utente di cui eseguire il rendering. <xref:Microsoft.AspNetCore.Components.RenderFragment%601> accetta un parametro di tipo che può essere specificato quando viene richiamato il frammento di rendering.

Spesso, i componenti basati su modelli sono tipizzati in modo generico, come `TableTemplate` illustrato nel componente seguente. Il tipo generico `<T>` in questo esempio viene usato per eseguire il rendering `IReadOnlyList<T>` dei valori, che in questo caso è una serie di righe PET in un componente che visualizza una tabella di animali domestici.

`Shared/TableTemplate.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Shared/templated-components/TableTemplate.razor)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Shared/templated-components/TableTemplate.razor)]

::: moniker-end

Quando si usa un componente basato su modelli, è possibile specificare i parametri del modello usando gli elementi figlio che corrispondono ai nomi dei parametri. Nell'esempio seguente, `<TableHeader>...</TableHeader>` e `<RowTemplate>...<RowTemplate>` forniscono <xref:Microsoft.AspNetCore.Components.RenderFragment%601> i modelli per `TableHeader` e `RowTemplate` del `TableTemplate` componente.

Specificare l' `Context` attributo sull'elemento Component quando si desidera specificare il nome del parametro di contenuto per il contenuto figlio implicito (senza alcun elemento figlio di wrapping). Nell'esempio seguente l' `Context` attributo viene visualizzato nell' `TableTemplate` elemento e si applica a tutti i <xref:Microsoft.AspNetCore.Components.RenderFragment%601> parametri del modello.

`Pages/Pets.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/templated-components/Pets1.razor)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/templated-components/Pets1.razor)]

::: moniker-end

In alternativa, è possibile modificare il nome del parametro usando l' `Context` attributo nell' <xref:Microsoft.AspNetCore.Components.RenderFragment%601> elemento figlio. Nell'esempio seguente, l'oggetto `Context` è impostato su `RowTemplate` anziché su `TableTemplate` :

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/templated-components/Pets2.razor?name=snippet&highlight=6)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/templated-components/Pets2.razor?name=snippet&highlight=6)]

::: moniker-end

Gli argomenti del componente di tipo <xref:Microsoft.AspNetCore.Components.RenderFragment%601> hanno un parametro implicito denominato `context` , che può essere usato. Nell'esempio seguente `Context` non è impostato. `@context.{PROPERTY}` fornisce valori di PET al modello, dove `{PROPERTY}` è una `Pet` proprietà:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/templated-components/Pets3.razor?name=snippet&highlight=7-8)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/templated-components/Pets3.razor?name=snippet&highlight=7-8)]

::: moniker-end

Quando si usano componenti tipizzati generici, il parametro di tipo viene dedotto, se possibile. Tuttavia, è possibile specificare in modo esplicito il tipo con un attributo il cui nome corrisponde al parametro di tipo, che è `TItem` riportato nell'esempio precedente:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/templated-components/Pets4.razor?name=snippet&highlight=1)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/templated-components/Pets4.razor?name=snippet&highlight=1)]

::: moniker-end

## <a name="generic-type-constraints"></a>Vincoli di tipo generico

> [!NOTE]
> I vincoli di tipo generico saranno supportati in una versione futura. Per ulteriori informazioni, vedere [Consenti vincoli di tipo generico (DotNet/aspnetcore #8433)](https://github.com/dotnet/aspnetcore/issues/8433).

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:blazor/webassembly-performance-best-practices#define-reusable-renderfragments-in-code>
