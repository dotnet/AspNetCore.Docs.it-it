---
title: Metodi di filtro per le Razor pagine in ASP.NET Core
author: Rick-Anderson
description: Informazioni su come creare metodi di filtro per le Razor pagine in ASP.NET Core.
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.date: 2/18/2020
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
uid: razor-pages/filter
ms.openlocfilehash: 178e6348d2d50dae34feea6a0ed261de01037136
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102586137"
---
# <a name="filter-methods-for-razor-pages-in-aspnet-core"></a>Metodi di filtro per le Razor pagine in ASP.NET Core

::: moniker range=">= aspnetcore-3.0"

Autore: [Rick Anderson](https://twitter.com/RickAndMSFT)

Razor I filtri di pagina [IPageFilter](/dotnet/api/microsoft.aspnetcore.mvc.filters.ipagefilter?view=aspnetcore-2.0) e [IAsyncPageFilter](/dotnet/api/microsoft.aspnetcore.mvc.filters.iasyncpagefilter?view=aspnetcore-2.0) consentono Razor alle pagine di eseguire codice prima e dopo l'esecuzione di un Razor gestore di pagina. Razor I filtri di pagina sono simili a [ASP.NET Core filtri azione MVC](xref:mvc/controllers/filters#action-filters), ma non possono essere applicati ai singoli metodi del gestore di pagina.

Razor Filtri di pagina:

* Eseguono il codice dopo aver selezionato un metodo del gestore, ma prima che si verifichi l'associazione di modelli.
* Eseguono il codice prima che il metodo del gestore venga eseguito, al termine dell'associazione di modelli.
* Eseguono il codice dopo l'esecuzione del metodo del gestore.
* Possono essere implementati in una pagina o a livello globale.
* Non possono essere applicati a metodi del gestore pagina specifici.
* Le dipendenze del costruttore possono essere popolate dall' [inserimento di dipendenze](xref:fundamentals/dependency-injection) . Per ulteriori informazioni, vedere [ServiceFilterAttribute](../mvc/controllers/filters.md#servicefilterattribute) e [TypeFilterAttribute](../mvc/controllers/filters.md#typefilterattribute).

Mentre i costruttori di pagina e il middleware abilitano l'esecuzione di codice personalizzato prima dell'esecuzione di un metodo del gestore, solo Razor i filtri di pagina consentono l'accesso a <xref:Microsoft.AspNetCore.Mvc.RazorPages.PageModel.HttpContext> e alla pagina. Il middleware ha accesso a `HttpContext` , ma non al "contesto di pagina". I filtri hanno un <xref:Microsoft.AspNetCore.Mvc.Filters.FilterContext> parametro derivato, che fornisce l'accesso a `HttpContext` . Di seguito è riportato un esempio per un filtro di pagina: [implementare un attributo di filtro](#ifa) che aggiunge un'intestazione alla risposta, operazione che non può essere eseguita con i costruttori o il middleware. L'accesso al contesto della pagina, che include l'accesso alle istanze della pagina e del relativo modello, è disponibile solo quando si eseguono filtri, gestori o il corpo di una Razor pagina.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/razor-pages/filter/3.1sample) ([procedura per il download](xref:index#how-to-download-a-sample))

Razor I filtri di pagina forniscono i metodi seguenti, che possono essere applicati a livello globale o a livello di pagina:

* Metodi sincroni:

  * [OnPageHandlerSelected](/dotnet/api/microsoft.aspnetcore.mvc.filters.ipagefilter.onpagehandlerselected?view=aspnetcore-2.0): metodo chiamato dopo la selezione di un metodo del gestore, ma prima che si verifichi l'associazione di modelli.
  * [OnPageHandlerExecuting](/dotnet/api/microsoft.aspnetcore.mvc.filters.ipagefilter.onpagehandlerexecuting?view=aspnetcore-2.0): metodo chiamato prima che il metodo del gestore venga eseguito, al termine dell'associazione di modelli.
  * [OnPageHandlerExecuted](/dotnet/api/microsoft.aspnetcore.mvc.filters.ipagefilter.onpagehandlerexecuted?view=aspnetcore-2.0): metodo chiamato dopo che il metodo del gestore è stato eseguito e prima del risultato dell'azione.

* Metodi asincroni:

  * [OnPageHandlerSelectionAsync](/dotnet/api/microsoft.aspnetcore.mvc.filters.iasyncpagefilter.onpagehandlerselectionasync?view=aspnetcore-2.0): metodo chiamato in modo asincrono dopo la selezione del metodo del gestore, ma prima che si verifichi l'associazione di modelli.
  * [OnPageHandlerExecutionAsync](/dotnet/api/microsoft.aspnetcore.mvc.filters.iasyncpagefilter.onpagehandlerexecutionasync?view=aspnetcore-2.0): metodo chiamato in modo asincrono prima che venga richiamato il metodo del gestore, al termine dell'associazione di modelli.

Implementare la versione sincrona **oppure** la versione asincrona di un'interfaccia di filtro, **non** entrambe. Il framework controlla per prima cosa se il filtro implementa l'interfaccia asincrona e, in tal caso, la chiama. In caso contrario, chiama i metodi dell'interfaccia sincrona. Se entrambe le interfacce sono implementate, verranno chiamati solo i metodi asincroni. La stessa regola vale per gli override nelle pagine. Implementare la versione sincrona o la versione asincrona dell'override, non entrambe.

## <a name="implement-razor-page-filters-globally"></a>Implementare i Razor filtri di pagina a livello globale

Il codice seguente implementa `IAsyncPageFilter`:

[!code-csharp[Main](filter/3.1sample/PageFilter/Filters/SampleAsyncPageFilter.cs?name=snippet1)]

Nel codice precedente, `ProcessUserAgent.Write` è il codice fornito dall'utente che funziona con la stringa agente utente.

Il codice seguente abilita `SampleAsyncPageFilter` nella classe `Startup`:

[!code-csharp[Main](filter/3.1sample/PageFilter/Startup.cs?name=snippet2)]

Il codice seguente chiama <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.PageConventionCollection.AddFolderApplicationModelConvention*> per applicare l'oggetto `SampleAsyncPageFilter` solo alle pagine in */Movies*:

[!code-csharp[Main](filter/3.1sample/PageFilter/Startup2.cs?name=snippet2)]

Il codice seguente implementa il filtro `IPageFilter` sincrono:

[!code-csharp[Main](filter/3.1sample/PageFilter/Filters/SamplePageFilter.cs?name=snippet1)]

Il codice seguente abilita `SamplePageFilter`:

[!code-csharp[Main](filter/3.1sample/PageFilter/StartupSync.cs?name=snippet2)]

## <a name="implement-razor-page-filters-by-overriding-filter-methods"></a>Implementare i Razor filtri di pagina eseguendo l'override dei metodi di filtro

Il codice seguente esegue l'override dei Razor filtri di pagina asincroni:

[!code-csharp[Main](filter/3.1sample/PageFilter/Pages/Index.cshtml.cs?name=snippet)]

<a name="ifa"></a>

## <a name="implement-a-filter-attribute"></a>Implementare un attributo di filtro

Il filtro di filtro basato su attributi predefinito <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncResultFilter.OnResultExecutionAsync*> può essere sottoclassato. Il filtro seguente aggiunge un'intestazione alla risposta:

[!code-csharp[Main](filter/3.1sample/PageFilter/Filters/AddHeaderAttribute.cs)]

Il codice seguente applica l'attributo `AddHeader`:

[!code-csharp[Main](filter/3.1sample/PageFilter/Pages/Movies/Test.cshtml.cs)]

Usare uno strumento come gli strumenti di sviluppo del browser per esaminare le intestazioni. In **intestazioni risposta** `author: Rick` viene visualizzato.

Vedere [Override dell'ordine predefinito](xref:mvc/controllers/filters#overriding-the-default-order) per istruzioni su come eseguire l'override dell'ordine.

Vedere [Annullamento e blocco](xref:mvc/controllers/filters#cancellation-and-short-circuiting) per istruzioni su come bloccare la pipeline filtro da un filtro.

<a name="auth"></a>

## <a name="authorize-filter-attribute"></a>Autorizzare l'attributo di filtro

L'attributo [Authorize](/dotnet/api/microsoft.aspnetcore.authorization.authorizeattribute?view=aspnetcore-2.0) può essere applicato a `PageModel`:

[!code-csharp[Main](filter/sample/PageFilter/Pages/ModelWithAuthFilter.cshtml.cs?highlight=7)]

::: moniker-end

::: moniker range="< aspnetcore-3.0"

Autore: [Rick Anderson](https://twitter.com/RickAndMSFT)

Razor I filtri di pagina [IPageFilter](/dotnet/api/microsoft.aspnetcore.mvc.filters.ipagefilter?view=aspnetcore-2.0) e [IAsyncPageFilter](/dotnet/api/microsoft.aspnetcore.mvc.filters.iasyncpagefilter?view=aspnetcore-2.0) consentono Razor alle pagine di eseguire codice prima e dopo l'esecuzione di un Razor gestore di pagina. Razor I filtri di pagina sono simili a [ASP.NET Core filtri azione MVC](xref:mvc/controllers/filters#action-filters), ma non possono essere applicati ai singoli metodi del gestore di pagina.

Razor Filtri di pagina:

* Eseguono il codice dopo aver selezionato un metodo del gestore, ma prima che si verifichi l'associazione di modelli.
* Eseguono il codice prima che il metodo del gestore venga eseguito, al termine dell'associazione di modelli.
* Eseguono il codice dopo l'esecuzione del metodo del gestore.
* Possono essere implementati in una pagina o a livello globale.
* Non possono essere applicati a metodi del gestore pagina specifici.

Il codice può essere eseguito prima dell'esecuzione di un metodo del gestore usando il costruttore o il middleware della pagina, ma solo i Razor filtri di pagina hanno accesso a [HttpContext](/dotnet/api/microsoft.aspnetcore.mvc.razorpages.pagemodel.httpcontext?view=aspnetcore-2.0#Microsoft_AspNetCore_Mvc_RazorPages_PageModel_HttpContext). I filtri contengono un parametro derivato da [FilterContext](/dotnet/api/microsoft.aspnetcore.mvc.filters.filtercontext?view=aspnetcore-2.0), che fornisce accesso a `HttpContext`. Ad esempio, nell'esempio [Implementare un attributo di filtro](#ifa) viene aggiunta un'intestazione alla risposta, operazione impossibile con costruttori o middleware.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/razor-pages/filter/sample/PageFilter) ([procedura per il download](xref:index#how-to-download-a-sample))

Razor I filtri di pagina forniscono i metodi seguenti, che possono essere applicati a livello globale o a livello di pagina:

* Metodi sincroni:

  * [OnPageHandlerSelected](/dotnet/api/microsoft.aspnetcore.mvc.filters.ipagefilter.onpagehandlerselected?view=aspnetcore-2.0): metodo chiamato dopo la selezione di un metodo del gestore, ma prima che si verifichi l'associazione di modelli.
  * [OnPageHandlerExecuting](/dotnet/api/microsoft.aspnetcore.mvc.filters.ipagefilter.onpagehandlerexecuting?view=aspnetcore-2.0): metodo chiamato prima che il metodo del gestore venga eseguito, al termine dell'associazione di modelli.
  * [OnPageHandlerExecuted](/dotnet/api/microsoft.aspnetcore.mvc.filters.ipagefilter.onpagehandlerexecuted?view=aspnetcore-2.0): metodo chiamato dopo che il metodo del gestore è stato eseguito e prima del risultato dell'azione.

* Metodi asincroni:

  * [OnPageHandlerSelectionAsync](/dotnet/api/microsoft.aspnetcore.mvc.filters.iasyncpagefilter.onpagehandlerselectionasync?view=aspnetcore-2.0): metodo chiamato in modo asincrono dopo la selezione del metodo del gestore, ma prima che si verifichi l'associazione di modelli.
  * [OnPageHandlerExecutionAsync](/dotnet/api/microsoft.aspnetcore.mvc.filters.iasyncpagefilter.onpagehandlerexecutionasync?view=aspnetcore-2.0): metodo chiamato in modo asincrono prima che venga richiamato il metodo del gestore, al termine dell'associazione di modelli.

> [!NOTE]
> Implementare la versione sincrona **oppure** la versione asincrona di un'interfaccia di filtro, non entrambe. Il framework controlla per prima cosa se il filtro implementa l'interfaccia asincrona e, in tal caso, la chiama. In caso contrario, chiama i metodi dell'interfaccia sincrona. Se entrambe le interfacce sono implementate, verranno chiamati solo i metodi asincroni. La stessa regola vale per gli override nelle pagine. Implementare la versione sincrona o la versione asincrona dell'override, non entrambe.

## <a name="implement-razor-page-filters-globally"></a>Implementare i Razor filtri di pagina a livello globale

Il codice seguente implementa `IAsyncPageFilter`:

[!code-csharp[Main](filter/sample/PageFilter/Filters/SampleAsyncPageFilter.cs?name=snippet1)]

Nel codice precedente, [ILogger](/dotnet/api/microsoft.extensions.logging.ilogger?view=aspnetcore-2.0) non è obbligatorio. Nell'esempio viene utilizzato per fornire informazioni di traccia per l'applicazione.

Il codice seguente abilita `SampleAsyncPageFilter` nella classe `Startup`:

[!code-csharp[Main](filter/sample/PageFilter/Startup.cs?name=snippet2&highlight=11)]

Il codice seguente mostra la classe `Startup` completa:

[!code-csharp[Main](filter/sample/PageFilter/Startup.cs?name=snippet1)]

Il codice seguente chiama `AddFolderApplicationModelConvention` per applicare `SampleAsyncPageFilter` solo alle pagine in */subFolder*:

[!code-csharp[Main](filter/sample/PageFilter/Startup2.cs?name=snippet2)]

Il codice seguente implementa il filtro `IPageFilter` sincrono:

[!code-csharp[Main](filter/sample/PageFilter/Filters/SamplePageFilter.cs?name=snippet1)]

Il codice seguente abilita `SamplePageFilter`:

[!code-csharp[Main](filter/sample/PageFilter/StartupSync.cs?name=snippet2&highlight=11)]

## <a name="implement-razor-page-filters-by-overriding-filter-methods"></a>Implementare i Razor filtri di pagina eseguendo l'override dei metodi di filtro

Il codice seguente esegue l'override dei Razor filtri di pagina sincroni:

[!code-csharp[Main](filter/sample/PageFilter/Pages/Index.cshtml.cs)]

<a name="ifa"></a>

## <a name="implement-a-filter-attribute"></a>Implementare un attributo di filtro

Il filtro basato su attributi predefinito [OnResultExecutionAsync](/dotnet/api/microsoft.aspnetcore.mvc.filters.iasyncresultfilter.onresultexecutionasync?view=aspnetcore-2.0#Microsoft_AspNetCore_Mvc_Filters_IAsyncResultFilter_OnResultExecutionAsync_Microsoft_AspNetCore_Mvc_Filters_ResultExecutingContext_Microsoft_AspNetCore_Mvc_Filters_ResultExecutionDelegate_) può essere sottoclassato. Il filtro seguente aggiunge un'intestazione alla risposta:

[!code-csharp[Main](filter/sample/PageFilter/Filters/AddHeaderAttribute.cs)]

Il codice seguente applica l'attributo `AddHeader`:

[!code-csharp[Main](filter/sample/PageFilter/Pages/Contact.cshtml.cs?name=snippet1)]

Vedere [Override dell'ordine predefinito](xref:mvc/controllers/filters#overriding-the-default-order) per istruzioni su come eseguire l'override dell'ordine.

Vedere [Annullamento e blocco](xref:mvc/controllers/filters#cancellation-and-short-circuiting) per istruzioni su come bloccare la pipeline filtro da un filtro. 

<a name="auth"></a>

## <a name="authorize-filter-attribute"></a>Autorizzare l'attributo di filtro

L'attributo [Authorize](/dotnet/api/microsoft.aspnetcore.authorization.authorizeattribute?view=aspnetcore-2.0) può essere applicato a `PageModel`:

[!code-csharp[Main](filter/sample/PageFilter/Pages/ModelWithAuthFilter.cshtml.cs?highlight=7)]

::: moniker-end