---
title: BlazorLayout ASP.NET Core
author: guardrex
description: Informazioni su come creare componenti di layout riutilizzabili per le Blazor app.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 06/23/2020
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
uid: blazor/layouts
ms.openlocfilehash: d1f3e2028ca120b5901aca0b24802d872ae52597
ms.sourcegitcommit: 1166b0ff3828418559510c661e8240e5c5717bb7
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/12/2021
ms.locfileid: "100279735"
---
# <a name="aspnet-core-blazor-layouts"></a>BlazorLayout ASP.NET Core

Alcuni elementi dell'app, ad esempio i menu, i messaggi di copyright e i logo aziendali, fanno in genere parte del layout generale dell'app e vengono usati da ogni componente nell'app. La copia del codice di questi elementi in tutti i componenti di un'app non è un approccio efficace. Ogni volta che uno degli elementi richiede un aggiornamento, è necessario aggiornare ogni componente. Tale duplicazione è difficile da gestire e può causare contenuti incoerenti nel tempo. I *layout* risolvono questo problema.

Tecnicamente, un layout è semplicemente un altro componente. Un layout è definito in un Razor modello o nel codice C# e può usare [Data Binding](xref:blazor/components/data-binding), l' [inserimento delle dipendenze](xref:blazor/fundamentals/dependency-injection)e altri scenari di componenti. I layout si applicano solo ai componenti instradabili con Razor [`@page`](xref:mvc/views/razor#page) direttive.

Per convertire un componente in un layout:

* Ereditare il componente da <xref:Microsoft.AspNetCore.Components.LayoutComponentBase> . <xref:Microsoft.AspNetCore.Components.LayoutComponentBase>Definisce una <xref:Microsoft.AspNetCore.Components.LayoutComponentBase.Body> proprietà per il contenuto di cui è stato eseguito il rendering all'interno del layout.
* Utilizzare la Razor sintassi `@Body` per specificare il percorso nel markup di layout in cui viene eseguito il rendering del contenuto.

Nell'esempio di codice riportato di seguito viene illustrato il Razor modello di un componente di layout `MainLayout.razor` . Il layout eredita <xref:Microsoft.AspNetCore.Components.LayoutComponentBase> e imposta l'oggetto `@Body` tra la barra di spostamento e il piè di pagina:

[!code-razor[](layouts/sample_snapshot/3.x/MainLayout.razor)]

## <a name="mainlayout-component"></a>Componente `MainLayout`

In un'app basata su uno dei Blazor modelli di progetto, il `MainLayout` componente ( `MainLayout.razor` ) si trova nella cartella dell'app `Shared` :

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](./common/samples/5.x/BlazorWebAssemblySample/Shared/MainLayout.razor)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](./common/samples/3.x/BlazorWebAssemblySample/Shared/MainLayout.razor)]

::: moniker-end

## <a name="default-layout"></a>Layout predefinito

Specificare il layout dell'app predefinito nel <xref:Microsoft.AspNetCore.Components.Routing.Router> componente nel file dell'app `App.razor` . Il <xref:Microsoft.AspNetCore.Components.Routing.Router> componente seguente, fornito dai Blazor modelli predefiniti, imposta il layout predefinito sul `MainLayout` componente:

[!code-razor[](layouts/sample_snapshot/3.x/App1.razor?highlight=3)]

[!INCLUDE[](~/blazor/includes/prefer-exact-matches.md)]

Per fornire un layout predefinito per il <xref:Microsoft.AspNetCore.Components.Routing.Router.NotFound> contenuto, specificare un oggetto <xref:Microsoft.AspNetCore.Components.LayoutView> per il <xref:Microsoft.AspNetCore.Components.Routing.Router.NotFound> contenuto:

[!code-razor[](layouts/sample_snapshot/3.x/App2.razor?highlight=6-9)]

[!INCLUDE[](~/blazor/includes/prefer-exact-matches.md)]

Per ulteriori informazioni sul <xref:Microsoft.AspNetCore.Components.Routing.Router> componente, vedere <xref:blazor/fundamentals/routing> .

La specifica del layout come layout predefinito nel router è una procedura utile perché può essere sottoposta a override in base al singolo componente o alla cartella. Preferisci usare il router per impostare il layout predefinito dell'app perché è la tecnica più generale.

## <a name="specify-a-layout-in-a-component"></a>Specificare un layout in un componente

Utilizzare la [`@layout`](xref:mvc/views/razor#layout) Razor direttiva per applicare un layout a un componente instradabile Razor che dispone anche di una [`@page`](xref:mvc/views/razor#page) direttiva. Il compilatore esegue la conversione `@layout` in un oggetto <xref:Microsoft.AspNetCore.Components.LayoutAttribute> , che viene applicato alla classe Component.

Il contenuto del componente seguente `MasterList` viene inserito nella `MasterLayout` posizione di `@Body` :

[!code-razor[](layouts/sample_snapshot/3.x/MasterList.razor?highlight=1)]

La specifica del layout direttamente in un componente sostituisce un set di *layout predefinito* nel router o una `@layout` direttiva importata da `_Imports.razor` .

## <a name="centralized-layout-selection"></a>Selezione layout centralizzata

Ogni cartella di un'app può contenere facoltativamente un file modello denominato `_Imports.razor` . Il compilatore include le direttive specificate nel file Imports in tutti i Razor modelli nella stessa cartella e in modo ricorsivo in tutte le relative sottocartelle. Pertanto, un `_Imports.razor` file contenente `@layout MyCoolLayout` garantisce che tutti i componenti di una cartella usino `MyCoolLayout` . Non è necessario aggiungere ripetutamente `@layout MyCoolLayout` a tutti i file nella `.razor` cartella e nelle sottocartelle. `@using` le direttive vengono applicate anche ai componenti nello stesso modo.

Le `_Imports.razor` importazioni di file seguenti:

* `MyCoolLayout`.
* Tutti i Razor componenti nella stessa cartella e in tutte le sottocartelle.
* Spazio dei nomi `BlazorApp1.Data`.
 
[!code-razor[](layouts/sample_snapshot/3.x/_Imports.razor)]

Il `_Imports.razor` file è simile al [file _ViewImports. cshtml per le Razor visualizzazioni e le pagine,](xref:mvc/views/layout#importing-shared-directives) ma viene applicato in modo specifico ai Razor file del componente.

La specifica di un layout in `_Imports.razor` sostituisce un layout specificato come *layout predefinito* del router.

> [!WARNING]
> **Non** aggiungere una Razor `@layout` direttiva al `_Imports.razor` file radice, il che comporta un ciclo infinito di layout nell'app. Per controllare il layout predefinito dell'app, specificare il layout nel `Router` componente. Per ulteriori informazioni, vedere la sezione [layout predefinito](#default-layout) .

> [!NOTE]
> La [`@layout`](xref:mvc/views/razor#layout) Razor direttiva applica solo un layout ai componenti instradabili Razor con le [`@page`](xref:mvc/views/razor#page) direttive.

## <a name="nested-layouts"></a>Layout annidati

Le app possono essere costituite da layout annidati. Un componente può fare riferimento a un layout che a sua volta fa riferimento a un altro layout. Ad esempio, i layout di annidamento vengono usati per creare una struttura di menu a più livelli.

Nell'esempio seguente viene illustrato come utilizzare layout annidati. Il `EpisodesComponent.razor` file è il componente da visualizzare. Il componente fa riferimento a `MasterListLayout` :

[!code-razor[](layouts/sample_snapshot/3.x/EpisodesComponent.razor?highlight=1)]

Il `MasterListLayout.razor` file fornisce il `MasterListLayout` . Il layout fa riferimento a un altro layout, `MasterLayout` , in cui viene eseguito il rendering. `EpisodesComponent` viene sottoposto a rendering quando `@Body` viene visualizzato:

[!code-razor[](layouts/sample_snapshot/3.x/MasterListLayout.razor?highlight=1,9)]

Infine, `MasterLayout` in `MasterLayout.razor` contiene gli elementi di layout di primo livello, ad esempio l'intestazione, il menu principale e il piè di pagina. `MasterListLayout` con `EpisodesComponent` viene eseguito il rendering di quando viene `@Body` visualizzato:

[!code-razor[](layouts/sample_snapshot/3.x/MasterLayout.razor?highlight=6)]

## <a name="share-a-razor-pages-layout-with-integrated-components"></a>Condividere un Razor layout di pagine con componenti integrati

Quando i componenti instradabili sono integrati in un' Razor app pagine, il layout condiviso dell'app può essere usato con i componenti. Per altre informazioni, vedere <xref:blazor/components/prerendering-and-integration>.

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:mvc/views/layout>
