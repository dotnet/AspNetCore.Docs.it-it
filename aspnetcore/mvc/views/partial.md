---
title: Visualizzazioni parziali in ASP.NET Core
author: ardalis
description: Informazioni su come usare le visualizzazioni parziali per suddividere file di markup di grandi dimensioni e ridurre la duplicazione del markup comune nelle pagine Web nelle app ASP.NET Core.
ms.author: riande
ms.custom: mvc
ms.date: 06/12/2019
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
uid: mvc/views/partial
ms.openlocfilehash: 0a8e4a4fdecd657840c6c02424ffffa64d4ab473
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102586878"
---
# <a name="partial-views-in-aspnet-core"></a>Visualizzazioni parziali in ASP.NET Core

Di [Steve Smith](https://ardalis.com/), [Maher JENDOUBI](https://twitter.com/maherjend), [Rick Anderson](https://twitter.com/RickAndMSFT) e [Scott Sauber](https://twitter.com/scottsauber)

Una visualizzazione parziale è un [Razor](xref:mvc/views/razor) file di markup (con *estensione cshtml*) senza una [`@page`](xref:mvc/views/razor#page) direttiva che esegue il rendering dell'output HTML *all'interno* dell'output sottoposto a rendering di un altro file di markup.

::: moniker range=">= aspnetcore-2.1"

Il termine *visualizzazione parziale* viene usato per lo sviluppo di un'app MVC, in cui i file di markup sono denominati *visualizzazioni* o un' Razor app pagine, in cui i file di markup vengono chiamati *pagine*. Questo argomento si riferisce in modo generico alle pagine di visualizzazioni e pagine MVC Razor come *file di markup*.

::: moniker-end

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/mvc/views/partial/sample) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="when-to-use-partial-views"></a>Quando usare le visualizzazioni parziali

Le visualizzazioni parziali sono un modo efficace di:

* Suddividere file di markup di grandi dimensioni in componenti più piccoli.

  In un file di markup complesso e di grandi dimensioni, composto da diverse parti logiche, gestire ognuna di queste parti isolate in una visualizzazione parziale offre dei vantaggi. Il codice del file di markup è infatti gestibile in quanto il markup contiene solo la struttura della pagina complessiva e riferimenti alle visualizzazioni parziali.
* Ridurre la duplicazione del contenuto di markup comune nei file di markup.

  Quando nei vari file di markup vengono usati gli stessi elementi di markup, una visualizzazione parziale rimuove la duplicazione del contenuto di markup in un file di visualizzazione parziale. Quando il markup viene modificato nella visualizzazione parziale, aggiorna l'output sottoposto a rendering dei file di markup che usano la visualizzazione parziale.

Le visualizzazioni parziali non devono essere usate per gestire gli elementi di layout comuni. Gli elementi comuni del layout devono essere specificati nei file [_Layout.cshtml](xref:mvc/views/layout).

Non usare una visualizzazione parziale in cui per il rendering del markup è necessaria una logica di rendering complessa o l'esecuzione di codice. Invece di una visualizzazione parziale, usare un [componente di visualizzazione](xref:mvc/views/view-components).

## <a name="declare-partial-views"></a>Dichiarare le visualizzazioni parziali

::: moniker range=">= aspnetcore-2.0"

Una visualizzazione parziale è un file di markup con *estensione cshtml* senza una [`@page`](xref:mvc/views/razor#page) direttiva gestita nella cartella *views* (MVC) o nella cartella *pages* ( Razor pagine).

In ASP.NET Core MVC, l'elemento <xref:Microsoft.AspNetCore.Mvc.ViewResult> di un controller è in grado di restituire una visualizzazione o una visualizzazione parziale. Nelle Razor pagine, un <xref:Microsoft.AspNetCore.Mvc.RazorPages.PageModel> può restituire una visualizzazione parziale rappresentata come un <xref:Microsoft.AspNetCore.Mvc.PartialViewResult> oggetto. Il riferimento e il rendering delle visualizzazioni parziali sono descritti nella sezione [Riferimento a una visualizzazione parziale](#reference-a-partial-view).

A differenza del rendering di pagine o visualizzazioni MVC, una visualizzazione parziale non esegue *_ViewStart.cshtml*. Per altre informazioni su *_ViewStart.cshtml*, vedere <xref:mvc/views/layout>.

I nomi dei file di visualizzazione parziale iniziano spesso con un carattere di sottolineatura (`_`). Questa convenzione di denominazione non è obbligatoria, ma è utile per differenziare visivamente le visualizzazioni parziali dalle visualizzazioni e dalle pagine.

::: moniker-end

::: moniker range="< aspnetcore-2.0"

Una visualizzazione parziale è un file di markup con estensione *cshtml* che si trova nella cartella *Views*.

L'elemento <xref:Microsoft.AspNetCore.Mvc.ViewResult> di un controller è in grado di restituire una visualizzazione o una visualizzazione parziale. Il riferimento e il rendering delle visualizzazioni parziali sono descritti nella sezione [Riferimento a una visualizzazione parziale](#reference-a-partial-view).

A differenza del rendering di visualizzazioni MVC, una visualizzazione parziale non esegue *_ViewStart.cshtml*. Per altre informazioni su *_ViewStart.cshtml*, vedere <xref:mvc/views/layout>.

I nomi dei file di visualizzazione parziale iniziano spesso con un carattere di sottolineatura (`_`). Questa convenzione di denominazione non è obbligatoria, ma è utile per differenziare visivamente le visualizzazioni parziali dalle visualizzazioni.

::: moniker-end

## <a name="reference-a-partial-view"></a>Riferimento a una visualizzazione parziale

::: moniker range=">= aspnetcore-2.0"

### <a name="use-a-partial-view-in-a-razor-pages-pagemodel"></a>Usare una visualizzazione parziale in una Razor pagina PageModel

In ASP.NET Core 2,0 o 2,1, il metodo del gestore seguente esegue il rendering della visualizzazione parziale *\_ AuthorPartialRP. cshtml* nella risposta:

```csharp
public IActionResult OnGetPartial() =>
    new PartialViewResult
    {
        ViewName = "_AuthorPartialRP",
        ViewData = ViewData,
    };
```

::: moniker-end

::: moniker range=">= aspnetcore-2.2"

In ASP.NET Core 2.2 o versioni successive, un metodo del gestore può chiamare in alternativa il metodo <xref:Microsoft.AspNetCore.Mvc.RazorPages.PageBase.Partial*> per produrre un oggetto `PartialViewResult`:

[!code-csharp[](partial/sample/PartialViewsSample/Pages/DiscoveryRP.cshtml.cs?name=snippet_OnGetPartial)]

::: moniker-end

### <a name="use-a-partial-view-in-a-markup-file"></a>Usare una visualizzazione parziale in un file di markup

::: moniker range=">= aspnetcore-2.1"

All'interno di un file di markup è possibile fare riferimento a una visualizzazione parziale in diversi modi. È consigliabile usare uno dei metodi asincroni seguenti nelle app:

* [Helper tag Partial](#partial-tag-helper)
* [Helper HTML asincrono](#asynchronous-html-helper)

::: moniker-end

::: moniker range="< aspnetcore-2.1"

All'interno di un file di markup è possibile fare riferimento a una visualizzazione parziale in due modi:

* [Helper HTML asincrono](#asynchronous-html-helper)
* [Helper HTML sincrono](#synchronous-html-helper)

È consigliabile usare l'[helper HTML asincrono](#asynchronous-html-helper) nelle app.

::: moniker-end

::: moniker range=">= aspnetcore-2.1"

### <a name="partial-tag-helper"></a>Helper tag Partial

L' [Helper per tag parziale](xref:mvc/views/tag-helpers/builtin-th/partial-tag-helper) richiede ASP.NET Core 2,1 o versione successiva.

Questo helper esegue il rendering del contenuto in modo asincrono e usa una sintassi HTML:

```cshtml
<partial name="_PartialName" />
```

Quando è presente un'estensione di file, l'helper tag fa riferimento a una visualizzazione parziale che deve essere contenuta nella stessa cartella in cui si trova il file di markup che chiama la visualizzazione parziale:

```cshtml
<partial name="_PartialName.cshtml" />
```

L'esempio seguente fa riferimento a una visualizzazione parziale dalla radice dell'app. I percorsi che iniziano con una tilde e una barra (`~/`) o con una barra (`/`) fanno riferimento alla radice dell'app:

**Razor Pagine**

```cshtml
<partial name="~/Pages/Folder/_PartialName.cshtml" />
<partial name="/Pages/Folder/_PartialName.cshtml" />
```

**MVC**

```cshtml
<partial name="~/Views/Folder/_PartialName.cshtml" />
<partial name="/Views/Folder/_PartialName.cshtml" />
```

L'esempio seguente fa riferimento a una visualizzazione parziale con un percorso relativo:

```cshtml
<partial name="../Account/_PartialName.cshtml" />
```

Per altre informazioni, vedere <xref:mvc/views/tag-helpers/builtin-th/partial-tag-helper>.

::: moniker-end

### <a name="asynchronous-html-helper"></a>Helper HTML asincrono

Quando si usa un helper HTML, la procedura consigliata consiste nell'usare <xref:Microsoft.AspNetCore.Mvc.Rendering.HtmlHelperPartialExtensions.PartialAsync*>. `PartialAsync` restituisce un tipo <xref:Microsoft.AspNetCore.Html.IHtmlContent> di cui è stato eseguito il wrapping in un <xref:System.Threading.Tasks.Task%601>. Il riferimento al metodo avviene facendo precedere la chiamata attesa da un carattere `@`:

```cshtml
@await Html.PartialAsync("_PartialName")
```

Quando è presente l'estensione di file, l'helper HTML fa riferimento a una visualizzazione parziale che deve essere contenuta nella stessa cartella in cui si trova il file di markup che chiama la visualizzazione parziale:

```cshtml
@await Html.PartialAsync("_PartialName.cshtml")
```

L'esempio seguente fa riferimento a una visualizzazione parziale dalla radice dell'app. I percorsi che iniziano con una tilde e una barra (`~/`) o con una barra (`/`) fanno riferimento alla radice dell'app:

::: moniker range=">= aspnetcore-2.1"

**Razor Pagine**

```cshtml
@await Html.PartialAsync("~/Pages/Folder/_PartialName.cshtml")
@await Html.PartialAsync("/Pages/Folder/_PartialName.cshtml")
```

**MVC**

::: moniker-end

```cshtml
@await Html.PartialAsync("~/Views/Folder/_PartialName.cshtml")
@await Html.PartialAsync("/Views/Folder/_PartialName.cshtml")
```

L'esempio seguente fa riferimento a una visualizzazione parziale con un percorso relativo:

```cshtml
@await Html.PartialAsync("../Account/_LoginPartial.cshtml")
```

In alternativa è possibile eseguire il rendering di una visualizzazione parziale con <xref:Microsoft.AspNetCore.Mvc.Rendering.HtmlHelperPartialExtensions.RenderPartialAsync*>. Questo metodo non restituisce un <xref:Microsoft.AspNetCore.Html.IHtmlContent>. Trasmette l'output sottoposto a rendering direttamente alla risposta. Poiché il metodo non restituisce un risultato, deve essere chiamato all'interno di un Razor blocco di codice:

[!code-cshtml[](partial/sample/PartialViewsSample/Views/Home/Discovery.cshtml?name=snippet_RenderPartialAsync)]

Poiché `RenderPartialAsync` trasmette il contenuto sottoposto a rendering, assicura prestazioni migliori in alcuni scenari. In situazioni in cui le prestazioni sono importanti, eseguire il benchmark della pagina usando entrambi gli approcci e usare quindi l'approccio che genera una risposta più rapidamente.

### <a name="synchronous-html-helper"></a>Helper HTML sincrono

<xref:Microsoft.AspNetCore.Mvc.Rendering.HtmlHelperPartialExtensions.Partial*> e <xref:Microsoft.AspNetCore.Mvc.Rendering.HtmlHelperPartialExtensions.RenderPartial*> sono gli equivalenti sincroni rispettivamente di `PartialAsync` e `RenderPartialAsync`. L'uso degli equivalenti sincroni non è consigliabile perché in alcuni scenari causano un deadlock. I metodi sincroni verranno rimossi in una versione futura.

> [!IMPORTANT]
> Se è necessario eseguire codice, usare un [componente di visualizzazione](xref:mvc/views/view-components) invece di una visualizzazione parziale.

::: moniker range=">= aspnetcore-2.1"

La chiamata di `Partial` o `RenderPartial` genera un avviso di Visual Studio Analyzer. Ad esempio, la presenza di `Partial` produce il messaggio di avviso seguente:

> L'uso di IHtmlHelper.Partial potrebbe determinare deadlock dell'applicazione. È consigliabile usare l'helper tag &lt;Partial&gt; o IHtmlHelper.PartialAsync.

Sostituire le chiamate a `@Html.Partial` con `@await Html.PartialAsync` o l' [Helper Tag parziale](xref:mvc/views/tag-helpers/builtin-th/partial-tag-helper). Per altre informazioni sulla migrazione dell'helper tag Partial, vedere [Eseguire la migrazione da un helper HTML](xref:mvc/views/tag-helpers/builtin-th/partial-tag-helper#migrate-from-an-html-helper).

::: moniker-end

## <a name="partial-view-discovery"></a>Individuazione delle visualizzazioni parziali

Quando si fa riferimento a una visualizzazione parziale per nome senza un'estensione di file, viene eseguita una ricerca nelle posizioni seguenti, nell'ordine indicato:

::: moniker range=">= aspnetcore-2.1"

**Razor Pagine**

1. Cartella della pagina attualmente in esecuzione
1. Grafico delle directory sopra la cartella della pagina
1. `/Shared`
1. `/Pages/Shared`
1. `/Views/Shared`

**MVC**

::: moniker-end

::: moniker range=">= aspnetcore-2.0"

1. `/Areas/<Area-Name>/Views/<Controller-Name>`
1. `/Areas/<Area-Name>/Views/Shared`
1. `/Views/Shared`
1. `/Pages/Shared`

::: moniker-end

::: moniker range="< aspnetcore-2.0"

1. `/Areas/<Area-Name>/Views/<Controller-Name>`
1. `/Areas/<Area-Name>/Views/Shared`
1. `/Views/Shared`

::: moniker-end

Le convenzioni seguenti si applicano all'individuazione delle visualizzazioni parziali:

* Sono consentite visualizzazioni parziali diverse con lo stesso nome purché si trovino in cartelle diverse.
* Quando si fa riferimento a una visualizzazione parziale per nome senza un'estensione di file e la visualizzazione parziale è presente sia nella cartella del chiamante sia nella cartella *Shared*, viene usata la visualizzazione parziale nella cartella del chiamante. Se la visualizzazione parziale non è presente nella cartella del chiamante, viene usata la visualizzazione parziale nella cartella *Shared*. Le visualizzazioni parziali nella cartella *Shared* sono dette *visualizzazioni parziali condivise* o *visualizzazioni parziali predefinite*.
* Le visualizzazioni parziali possono essere *concatenate* &mdash; una visualizzazione parziale può chiamare un'altra visualizzazione parziale se un riferimento circolare non è formato dalle chiamate. I percorsi relativi sono sempre relativi al file corrente, non alla radice o al padre del file.

> [!NOTE]
> Un oggetto [Razor](xref:mvc/views/razor) `section` definito in una visualizzazione parziale è invisibile ai file di markup padre. L'oggetto `section` è visibile solo per la visualizzazione parziale in cui è definito.

## <a name="access-data-from-partial-views"></a>Accesso ai dati da visualizzazioni parziali

Quando viene creata un'istanza di una visualizzazione parziale, riceve una *copia* del dizionario `ViewData` del padre. Gli aggiornamenti apportati ai dati all'interno della visualizzazione parziale non vengono mantenuti per la visualizzazione padre. Le modifiche a `ViewData` in una visualizzazione parziale vengono perse quando viene restituita la visualizzazione parziale.

L'esempio seguente mostra come passare un'istanza di [ViewDataDictionary](/dotnet/api/microsoft.aspnetcore.mvc.viewfeatures.viewdatadictionary) a una visualizzazione parziale:

```cshtml
@await Html.PartialAsync("_PartialName", customViewData)
```

È possibile trasmettere un modello in una visualizzazione parziale. Il modello può essere un oggetto personalizzato. È possibile passare un modello con `PartialAsync` (esegue il rendering di un blocco di contenuto al chiamante) o con `RenderPartialAsync` (trasmette il contenuto all'output):

```cshtml
@await Html.PartialAsync("_PartialName", model)
```

::: moniker range=">= aspnetcore-2.1"

**Razor Pagine**

Il markup seguente nella pagina di esempio è tratto dalla pagina *Pages/ArticlesRP/ReadRP.cshtml*. La pagina contiene due visualizzazioni parziali. La seconda visualizzazione parziale viene trasmessa a un modello e `ViewData` viene trasmesso alla visualizzazione parziale. L'overload del costruttore `ViewDataDictionary` viene usato per passare un nuovo dizionario `ViewData` mantenendo il dizionario `ViewData` esistente.

[!code-cshtml[](partial/sample/PartialViewsSample/Pages/ArticlesRP/ReadRP.cshtml?name=snippet_ReadPartialViewRP&highlight=5,15-20)]

*Pages/Shared/_AuthorPartialRP.cshtml* è la prima visualizzazione parziale a cui fa riferimento il file di markup *ReadRP.cshtml*:

[!code-cshtml[](partial/sample/PartialViewsSample/Pages/Shared/_AuthorPartialRP.cshtml)]

*Pages/ArticlesRP/_ArticleSectionRP.cshtml* è la seconda visualizzazione parziale a cui fa riferimento il file di markup *ReadRP.cshtml*:

[!code-cshtml[](partial/sample/PartialViewsSample/Pages/ArticlesRP/_ArticleSectionRP.cshtml)]

**MVC**

::: moniker-end

Il markup seguente nell'app di esempio mostra la visualizzazione *Views/Articles/ReadRP.cshtml*. La visualizzazione contiene due visualizzazioni parziali. La seconda visualizzazione parziale viene trasmessa a un modello e `ViewData` viene trasmesso alla visualizzazione parziale. L'overload del costruttore `ViewDataDictionary` viene usato per passare un nuovo dizionario `ViewData` mantenendo il dizionario `ViewData` esistente.

[!code-cshtml[](partial/sample/PartialViewsSample/Views/Articles/Read.cshtml?name=snippet_ReadPartialView&highlight=5,15-20)]

*Views/Shared/_AuthorPartial. cshtml* è la prima visualizzazione parziale a cui fa riferimento il file di markup *Read. cshtml* :

[!code-cshtml[](partial/sample/PartialViewsSample/Views/Shared/_AuthorPartial.cshtml)]

*Views/Articles/_ArticleSection.cshtml* è la seconda visualizzazione parziale a cui fa riferimento il file di markup *ReadRP.cshtml*:

[!code-cshtml[](partial/sample/PartialViewsSample/Views/Articles/_ArticleSection.cshtml)]

In fase di esecuzione, le visualizzazioni parziali vengono sottoposte a rendering nell'output sottoposto a rendering del file di markup, di cui viene a sua volta eseguito il rendering all'interno dell'elemento *_Layout.cshtml* condiviso. La prima visualizzazione parziale esegue il rendering del nome dell'autore e della data di pubblicazione dell'articolo:

> Abraham Lincoln
>
> Questa visualizzazione parziale dal &lt;percorso del file di visualizzazione parziale condivisa&gt;.
> 11/19/1863 12:00:00 AM

La seconda visualizzazione parziale esegue il rendering delle sezioni dell'articolo:

> Section One Index: 0
>
> Four score and seven years ago ...
>
> Section Two Index: 1
>
> Now we are engaged in a great civil war, testing ...
>
> Section Three Index: 2
>
> But, in a larger sense, we can not dedicate ...

## <a name="additional-resources"></a>Risorse aggiuntive

::: moniker range=">= aspnetcore-2.1"

* [Razor informazioni di riferimento sulla sintassi per ASP.NET Core](xref:mvc/views/razor)
* <xref:mvc/views/tag-helpers/intro>
* <xref:mvc/views/tag-helpers/builtin-th/partial-tag-helper>
* <xref:mvc/views/view-components>
* <xref:mvc/controllers/areas>

::: moniker-end

::: moniker range="< aspnetcore-2.1"

* [Razor informazioni di riferimento sulla sintassi per ASP.NET Core](xref:mvc/views/razor)
* <xref:mvc/views/view-components>
* <xref:mvc/controllers/areas>

::: moniker-end
