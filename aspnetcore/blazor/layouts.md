---
title: BlazorLayout ASP.NET Core
author: guardrex
description: Informazioni su come creare componenti di layout riutilizzabili per le Blazor app.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 03/02/2021
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
ms.openlocfilehash: 9f3400b766a043fdf3872838bffe392eddba4049
ms.sourcegitcommit: a1db01b4d3bd8c57d7a9c94ce122a6db68002d66
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/04/2021
ms.locfileid: "102394947"
---
# <a name="aspnet-core-blazor-layouts"></a>BlazorLayout ASP.NET Core

Alcuni elementi dell'app, ad esempio i menu, i messaggi di copyright e i logo aziendali, fanno in genere parte della presentazione complessiva dell'app. L'inserimento di una copia del markup per questi elementi in tutti i componenti di un'app non è efficiente. Ogni volta che uno di questi elementi viene aggiornato, è necessario aggiornare ogni componente che usa l'elemento. Questo approccio è costoso da gestire e può causare contenuti incoerenti in caso di mancato aggiornamento. I *layout* risolvono questi problemi.

Un Blazor layout è un Razor componente che condivide il markup con i componenti che vi fanno riferimento. I layout possono usare [Data Binding](xref:blazor/components/data-binding), l' [inserimento delle dipendenze](xref:blazor/fundamentals/dependency-injection)e altre funzionalità dei componenti.

## <a name="layout-components"></a>Componenti layout

### <a name="create-a-layout-component"></a>Creazione di un componente di layout

Per creare un componente di layout:

* Creare un Razor componente definito da un Razor modello o da un codice C#. I componenti di layout basati su un Razor modello usano l' `.razor` estensione del file proprio come i Razor componenti normali. Poiché i componenti di layout sono condivisi tra i componenti di un'app, vengono in genere inseriti nella `Shared` cartella dell'app. Tuttavia, i layout possono essere posizionati in qualsiasi posizione accessibile ai componenti che lo usano. Un layout, ad esempio, può essere inserito nella stessa cartella dei componenti che lo usano.
* Ereditare il componente da <xref:Microsoft.AspNetCore.Components.LayoutComponentBase> . <xref:Microsoft.AspNetCore.Components.LayoutComponentBase>Definisce una <xref:Microsoft.AspNetCore.Components.LayoutComponentBase.Body> Proprietà ( <xref:Microsoft.AspNetCore.Components.RenderFragment> tipo) per il contenuto di cui è stato eseguito il rendering all'interno del layout.
* Utilizzare la Razor sintassi `@Body` per specificare il percorso nel markup di layout in cui viene eseguito il rendering del contenuto.

Il `DoctorWhoLayout` componente seguente mostra il Razor modello di un componente di layout. Il layout eredita <xref:Microsoft.AspNetCore.Components.LayoutComponentBase> e imposta l'oggetto `@Body` tra la barra di spostamento ( `<nav>...</nav>` ) e il piè di pagina ( `<footer>...</footer>` ).

`Shared/DoctorWhoLayout.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Shared/layouts/DoctorWhoLayout.razor?highlight=1,13)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Shared/layouts/DoctorWhoLayout.razor?highlight=1,13)]

::: moniker-end

### <a name="mainlayout-component"></a>Componente `MainLayout`

In un'app creata da un [ Blazor modello di progetto](xref:blazor/project-structure), il `MainLayout` componente è il [layout predefinito](#apply-a-default-layout-to-an-app)dell'app.

`Shared/MainLayout.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Shared/layouts/MainLayout.razor)]

la [ Blazor funzionalità di isolamento CSS applica gli](xref:blazor/components/css-isolation) stili CSS isolati al `MainLayout` componente. Per convenzione, gli stili vengono forniti dal foglio di stile associato con lo stesso nome `Shared/MainLayout.razor.css` . L'implementazione del ASP.NET Core Framework del foglio di stile è disponibile per l'ispezione nell' [origine riferimento ASP.NET Core (repository GitHub DotNet/aspnetcore)](https://github.com/dotnet/aspnetcore/blob/main/src/ProjectTemplates/Web.ProjectTemplates/content/ComponentsWebAssembly-CSharp/Client/Shared/MainLayout.razor.css).

[!INCLUDE[](~/blazor/includes/aspnetcore-repo-ref-source-links.md)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Shared/layouts/MainLayout.razor)]

::: moniker-end

## <a name="apply-a-layout"></a>Applicare un layout

### <a name="apply-a-layout-to-a-component"></a>Applicare un layout a un componente

Utilizzare la [`@layout`](xref:mvc/views/razor#layout) Razor direttiva per applicare un layout a un componente instradabile con Razor una [`@page`](xref:mvc/views/razor#page) direttiva. Il compilatore converte `@layout` in un oggetto <xref:Microsoft.AspNetCore.Components.LayoutAttribute> e applica l'attributo alla classe Component.

Il contenuto del componente seguente `Episodes` viene inserito in `DoctorWhoLayout` in corrispondenza della posizione di `@Body` .

`Pages/Episodes.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Pages/layouts/Episodes.razor?highlight=2)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Pages/layouts/Episodes.razor?highlight=2)]

::: moniker-end

Il markup HTML sottoposto a rendering seguente viene generato `DoctorWhoLayout` dal `Episodes` componente e precedente. Il markup estraneo non viene visualizzato per concentrarsi sul contenuto fornito dai due componenti interessati:

* L'intestazione del **&trade; database dell'episodio Doctor** () `<h1>...</h1>` nell'intestazione ( `<header>...</header>` ), nella barra di spostamento ( `<nav>...</nav>` ) e nell'elemento informazioni sul marchio () del piè di pagina `<div>...</div>` ( `<footer>...</footer>` ) proviene dal `DoctorWhoLayout` componente.
* L'intestazione degli **episodi** ( `<h2>...</h2>` ) e l'elenco di puntatori ( `<ul>...</ul>` ) provengono dal `Episodes` componente.

```html
<body>
    <div id="app">
        <header>
            <h1>Doctor Who&trade; Episode Database</h1>
        </header>

        <nav>
            <a href="masterlist">Master Episode List</a>
            <a href="search">Search</a>
            <a href="new">Add Episode</a>
        </nav>

        <h2>Episodes</h2>

        <ul>
            <li>...</li>
            <li>...</li>
            <li>...</li>
        </ul>

        <footer>
            Doctor Who is a registered trademark of the BBC. 
            https://www.doctorwho.tv/
        </footer>
    </div>
</body>
```

La specifica del layout direttamente in un componente sostituisce un *layout predefinito*:

* Impostata da una `@layout` direttiva importata da un `_Imports` componente ( `_Imports.razor` ), come descritto nella sezione [applicare un layout a una cartella di componenti](#apply-a-layout-to-a-folder-of-components) .
* Impostare come layout predefinito dell'app, come descritto nella sezione [applicare un layout predefinito a un'app](#apply-a-default-layout-to-an-app) più avanti in questo articolo.

### <a name="apply-a-layout-to-a-folder-of-components"></a>Applicare un layout a una cartella di componenti

Ogni cartella di un'app può contenere facoltativamente un file modello denominato `_Imports.razor` . Il compilatore include le direttive specificate nel file Imports in tutti i Razor modelli nella stessa cartella e in modo ricorsivo in tutte le relative sottocartelle. Pertanto, un `_Imports.razor` file contenente `@layout DoctorWhoLayout` garantisce che tutti i componenti di una cartella usino il `DoctorWhoLayout` componente. Non è necessario aggiungere ripetutamente `@layout DoctorWhoLayout` a tutti i Razor componenti ( `.razor` ) nella cartella e nelle sottocartelle.

`_Imports.razor`:

```razor
@layout DoctorWhoLayout
...
```

Il `_Imports.razor` file è simile al [file _ViewImports. cshtml per le Razor visualizzazioni e le pagine,](xref:mvc/views/layout#importing-shared-directives) ma viene applicato in modo specifico ai Razor file del componente.

La specifica di un layout in `_Imports.razor` sostituisce un layout specificato come [layout predefinito dell'app](#apply-a-default-layout-to-an-app)del router, descritto nella sezione seguente.

> [!WARNING]
> **Non** aggiungere una Razor `@layout` direttiva al `_Imports.razor` file radice, il che comporta un ciclo infinito di layout. Per controllare il layout predefinito dell'app, specificare il layout nel `Router` componente. Per altre informazioni, vedere la sezione [applicare un layout predefinito a un'app](#apply-a-default-layout-to-an-app) .

> [!NOTE]
> La [`@layout`](xref:mvc/views/razor#layout) Razor direttiva applica solo un layout ai componenti instradabili Razor con una [`@page`](xref:mvc/views/razor#page) direttiva.

### <a name="apply-a-default-layout-to-an-app"></a>Applicare un layout predefinito a un'app

Specificare il layout dell'app predefinito in nel `App` componente del componente <xref:Microsoft.AspNetCore.Components.Routing.Router> . L'esempio seguente di un'app basata su un [ Blazor modello di progetto](xref:blazor/project-structure) imposta il layout predefinito sul `MainLayout` componente.

`App.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/layouts/App1.razor?highlight=3)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/layouts/App1.razor?highlight=3)]

::: moniker-end

[!INCLUDE[](~/blazor/includes/prefer-exact-matches.md)]

Per ulteriori informazioni sul <xref:Microsoft.AspNetCore.Components.Routing.Router> componente, vedere <xref:blazor/fundamentals/routing> .

La specifica del layout come layout predefinito nel `Router` componente è una procedura utile perché è possibile eseguire l'override del layout per ogni singolo componente o per ogni cartella, come descritto nelle sezioni precedenti di questo articolo. È consigliabile usare il `Router` componente per impostare il layout predefinito dell'app perché è l'approccio più generale e flessibile per l'uso dei layout.

### <a name="apply-a-layout-to-arbitrary-content-layoutview-component"></a>Applicare un layout al contenuto arbitrario ( `LayoutView` componente)

Per impostare un layout per il contenuto di un modello arbitrario Razor , specificare il layout con un <xref:Microsoft.AspNetCore.Components.LayoutView> componente. È possibile usare un oggetto <xref:Microsoft.AspNetCore.Components.LayoutView> in qualsiasi Razor componente. Nell'esempio seguente viene impostato un componente di layout denominato `ErrorLayout` per il `MainLayout` modello del componente <xref:Microsoft.AspNetCore.Components.Routing.Router.NotFound> ( `<NotFound>...</NotFound>` ).

`App.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/layouts/App2.razor?name=snippet&highlight=6,9)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/layouts/App2.razor?name=snippet&highlight=6,9)]

::: moniker-end

[!INCLUDE[](~/blazor/includes/prefer-exact-matches.md)]

## <a name="nested-layouts"></a>Layout annidati

Un componente può fare riferimento a un layout che a sua volta fa riferimento a un altro layout. I layout annidati, ad esempio, vengono utilizzati per creare una struttura di menu a più livelli.

Nell'esempio seguente viene illustrato come utilizzare layout annidati. Il componente `Episodes` illustrato nella sezione [applicare un layout a un componente](#apply-a-layout-to-a-component) è il componente da visualizzare. Il componente fa riferimento al `DoctorWhoLayout` componente.

Il `DoctorWhoLayout` componente seguente è una versione modificata dell'esempio illustrato in precedenza in questo articolo. Gli elementi dell'intestazione e del piè di pagina vengono rimossi e il layout fa riferimento a un altro layout `ProductionsLayout` . Il `Episodes` rendering del componente viene `@Body` visualizzato in `DoctorWhoLayout` .

`Shared/DoctorWhoLayout.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Shared/layouts/DoctorWhoLayout2.razor?highlight=2,12)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Shared/layouts/DoctorWhoLayout2.razor?highlight=2,12)]

::: moniker-end

Il `ProductionsLayout` componente contiene gli elementi di layout di primo livello, in cui ora si trovano gli elementi Header ( `<header>...</header>` ) e footer ( `<footer>...</footer>` ). `DoctorWhoLayout`Viene eseguito il rendering dell'oggetto con il `Episodes` componente `@Body` , dove viene visualizzato.

`Shared/ProductionsLayout.razor`:

::: moniker range=">= aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/5.x/BlazorSample_WebAssembly/Shared/layouts/ProductionsLayout.razor?highlight=13)]

::: moniker-end

::: moniker range="< aspnetcore-5.0"

[!code-razor[](~/blazor/common/samples/3.x/BlazorSample_WebAssembly/Shared/layouts/ProductionsLayout.razor?highlight=13)]

::: moniker-end

Il markup HTML sottoposto a rendering seguente viene generato dal layout annidato precedente. Il markup estraneo non viene visualizzato per concentrarsi sul contenuto annidato fornito dai tre componenti interessati:

* Gli elementi Header ( `<header>...</header>` ), barra di spostamento produzione () e piè di pagina `<nav>...</nav>` ( `<footer>...</footer>` ) e il relativo contenuto provengono dal `ProductionsLayout` componente.
* Il componente è il **medico che &trade; punta al database** heading ( `<h1>...</h1>` ), la barra di spostamento degli episodi ( `<nav>...</nav>` ) e l'elemento informazioni sul marchio ( `<div>...</div>` ) `DoctorWhoLayout` .
* L'intestazione degli **episodi** ( `<h2>...</h2>` ) e l'elenco di puntatori ( `<ul>...</ul>` ) provengono dal `Episodes` componente.

```html
<body>
    <div id="app">
        <header>
            <h1>Productions</h1>
        </header>

        <nav>
            <a href="master-production-list">Master Production List</a>
            <a href="production-search">Search</a>
            <a href="new-production">Add Production</a>
        </nav>

        <h1>Doctor Who&trade; Episode Database</h1>

        <nav>
            <a href="episode-masterlist">Master Episode List</a>
            <a href="episode-search">Search</a>
            <a href="new-episode">Add Episode</a>
        </nav>

        <h2>Episodes</h2>

        <ul>
            <li>...</li>
            <li>...</li>
            <li>...</li>
        </ul>

        <div>
            Doctor Who is a registered trademark of the BBC. 
            https://www.doctorwho.tv/
        </div>

        <footer>
            Footer of Productions Layout
        </footer>
    </div>
</body>
```

## <a name="share-a-razor-pages-layout-with-integrated-components"></a>Condividere un Razor layout di pagine con componenti integrati

Quando i componenti instradabili sono integrati in un' Razor app pagine, il layout condiviso dell'app può essere usato con i componenti. Per altre informazioni, vedere <xref:blazor/components/prerendering-and-integration>.

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:mvc/views/layout>
