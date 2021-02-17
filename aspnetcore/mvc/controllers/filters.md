---
title: Filtri in ASP.NET Core
author: Rick-Anderson
description: Informazioni sul funzionamento dei filtri e su come usarli in ASP.NET Core.
ms.author: riande
ms.custom: mvc
ms.date: 02/04/2020
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
uid: mvc/controllers/filters
ms.openlocfilehash: 416fc292d82cf841b2134e23a8e494e3e8d945ca
ms.sourcegitcommit: f77a7467651bab61b24261da9dc5c1dd75fc1fa9
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/17/2021
ms.locfileid: "100563983"
---
# <a name="filters-in-aspnet-core"></a>Filtri in ASP.NET Core

::: moniker range=">= aspnetcore-3.0"

Di [Kirk Larkin](https://github.com/serpent5), [Rick Anderson](https://twitter.com/RickAndMSFT), [Tom Dykstra](https://github.com/tdykstra/) e [Steve Smith](https://ardalis.com/)

I *filtri* in ASP.NET Core consentono l'esecuzione del codice prima o dopo fasi specifiche della pipeline di elaborazione della richiesta.

I filtri predefiniti gestiscono attività, ad esempio:

* Autorizzazione (impedire l'accesso alle risorse per cui un utente non è autorizzato).
* Memorizzazione nella cache delle risposte (blocco della pipeline delle richieste per restituire una risposta memorizzata nella cache).

I filtri personalizzati possono essere creati per gestire problemi relativi a più settori. Esempi di problematiche trasversali includono la gestione degli errori, la memorizzazione nella cache, la configurazione, l'autorizzazione e la registrazione.  I filtri evitano la duplicazione del codice. Ad esempio, un filtro eccezioni per la gestione degli errori potrebbe consolidare la gestione degli errori.

Questo documento si applica a Razor pagine, controller API e controller con visualizzazioni. I filtri non funzionano direttamente con i [ Razor componenti](xref:blazor/components/index). Un filtro può influire indirettamente su un componente solo quando:

* Il componente è incorporato in una pagina o in una vista.
* La pagina o il controller/Visualizzazione usa il filtro.

[Visualizzare o scaricare un esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/controllers/filters/3.1sample) ([procedura per il download](xref:index#how-to-download-a-sample)).

## <a name="how-filters-work"></a>Funzionamento dei filtri

I filtri vengono eseguiti all'interno della *pipeline di chiamata di azioni ASP.NET Core*, talvolta definita *pipeline di filtro*. La pipeline di filtro viene eseguita dopo che ASP.NET Core ha selezionato l'azione da eseguire.

![La richiesta viene elaborata tramite altri middleware, middleware di routing, selezione dell'azione e pipeline di chiamata dell'azione. L'elaborazione della richiesta torna alla selezione azione, al middleware di routing e a diversi altri middleware prima di diventare la risposta che viene inviata al client.](filters/_static/filter-pipeline-1.png)

### <a name="filter-types"></a>Tipi di filtro

Ogni tipo di filtro viene eseguito in una fase diversa della pipeline di filtro:

* I [filtri di autorizzazione](#authorization-filters) vengono eseguiti per primi e consentono di determinare se l'utente è autorizzato per la richiesta. I filtri di autorizzazione consentono di cortocircuitare la pipeline se la richiesta non è autorizzata.

* [Filtri risorse](#resource-filters):

  * Vengono eseguiti dopo l'autorizzazione.  
  * <xref:Microsoft.AspNetCore.Mvc.Filters.IResourceFilter.OnResourceExecuting*> esegue il codice prima del resto della pipeline filtro. Ad esempio, `OnResourceExecuting` esegue il codice prima dell'associazione di modelli.
  * <xref:Microsoft.AspNetCore.Mvc.Filters.IResourceFilter.OnResourceExecuted*> esegue il codice una volta completato il resto della pipeline.

* [Filtri azione](#action-filters):

  * Eseguire il codice immediatamente prima e dopo la chiamata di un metodo di azione.
  * Consente di modificare gli argomenti passati in un'azione.
  * Può modificare il risultato restituito dall'azione.
  * **Non** sono supportate nelle Razor pagine.

* I [filtri eccezioni](#exception-filters) applicano i criteri globali alle eccezioni non gestite che si verificano prima della scrittura del corpo della risposta.

* I [filtri risultati](#result-filters) eseguono il codice immediatamente prima e dopo l'esecuzione dei risultati dell'azione. Vengono eseguiti solo quando il metodo di azione è stato eseguito correttamente. Sono utili per la logica che deve racchiudere l'esecuzione del formattatore o di una vista.

Il diagramma seguente illustra come interagiscono i tipi di filtro nella pipeline di filtro.

![La richiesta passa attraverso i filtri autorizzazione, i filtri risorse, l'associazione di modelli, i filtri azione, l'esecuzione dell'azione e la conversione del risultato dell'azione, i filtri eccezioni, i filtri risultato e l'esecuzione del risultato. All'uscita della pipeline, la richiesta viene elaborata solo dai filtri risultato e dai filtri risorse prima di diventare una risposta inviata al client.](filters/_static/filter-pipeline-2.png)

## <a name="implementation"></a>Implementazione

I filtri supportano sia implementazioni sincrone che asincrone tramite definizioni di interfaccia diverse.

I filtri sincroni eseguono codice prima e dopo la fase della pipeline. La chiamata di <xref:Microsoft.AspNetCore.Mvc.Controller.OnActionExecuting*>, ad esempio, avviene prima della chiamata del metodo di azione. La chiamata di <xref:Microsoft.AspNetCore.Mvc.Controller.OnActionExecuted*> avviene dopo l'esecuzione del metodo di azione.

[!code-csharp[](./filters/3.1sample/FiltersSample/Filters/MySampleActionFilter.cs?name=snippet_ActionFilter)]

Nel codice precedente, il [debug](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/mvc/controllers/filters/3.1sample/FiltersSample/Helper/MyDebug.cs) è una funzione di utilità nell' [esempio di download](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/mvc/controllers/filters/3.1sample/FiltersSample/Helper/MyDebug.cs).

I filtri asincroni definiscono un `On-Stage-ExecutionAsync` metodo. Ad esempio, <xref:Microsoft.AspNetCore.Mvc.Controller.OnActionExecutionAsync*>:

[!code-csharp[](./filters/3.1sample/FiltersSample/Filters/SampleAsyncActionFilter.cs?name=snippet)]

Nel codice precedente, `SampleAsyncActionFilter` ha un <xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutionDelegate> ( `next` ) che esegue il metodo di azione.

### <a name="multiple-filter-stages"></a>Fasi di filtro multiple

È possibile implementare interfacce per più fasi di filtro in una singola classe. Ad esempio, la <xref:Microsoft.AspNetCore.Mvc.Filters.ActionFilterAttribute> classe implementa:

* Sincrona: <xref:Microsoft.AspNetCore.Mvc.Filters.IActionFilter> e  <xref:Microsoft.AspNetCore.Mvc.Filters.IResultFilter>
* Asincrono: <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncActionFilter> e <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncResultFilter>
* <xref:Microsoft.AspNetCore.Mvc.Filters.IOrderedFilter>

Implementare la versione sincrona **oppure** la versione asincrona di un'interfaccia di filtro, **non** entrambe. Il runtime controlla per prima cosa se il filtro implementa l'interfaccia asincrona e, in tal caso, la chiama. In caso contrario, chiama i metodi dell'interfaccia sincrona. Se in una classe sono implementate entrambe le interfacce, sincrona e asincrona, viene chiamato solo il metodo asincrono. Quando si usano classi astratte come <xref:Microsoft.AspNetCore.Mvc.Filters.ActionFilterAttribute> , eseguire l'override solo dei metodi sincroni o dei metodi asincroni per ogni tipo di filtro.

### <a name="built-in-filter-attributes"></a>Attributi filtro predefiniti

ASP.NET Core include filtri predefiniti basati su attributi che è possibile impostare come sottoclasse e personalizzare. Il filtro dei risultati seguente, ad esempio, aggiunge un'intestazione alla risposta:

<a name="add-header-attribute"></a>

[!code-csharp[](./filters/3.1sample/FiltersSample/Filters/AddHeaderAttribute.cs?name=snippet)]

Gli attributi consentono ai filtri di accettare argomenti, come illustrato nell'esempio precedente. Applicare `AddHeaderAttribute` a un controller o a un metodo di azione e specificare il nome e il valore dell'intestazione HTTP:

[!code-csharp[](./filters/3.1sample/FiltersSample/Controllers/SampleController.cs?name=snippet_AddHeader&highlight=1)]

Usare uno strumento come gli [strumenti di sviluppo del browser](https://developer.mozilla.org/docs/Learn/Common_questions/What_are_browser_developer_tools) per esaminare le intestazioni. In **intestazioni risposta** `author: Rick Anderson` viene visualizzato.

Il codice seguente implementa un oggetto `ActionFilterAttribute` che:

* Legge il titolo e il nome dal sistema di configurazione. A differenza dell'esempio precedente, il codice seguente non richiede l'aggiunta di parametri di filtro al codice.
* Aggiunge il titolo e il nome all'intestazione della risposta.

[!code-csharp[](./filters/3.1sample/FiltersSample/Filters/MyActionFilterAttribute.cs?name=snippet)]

Le opzioni di configurazione vengono fornite dal [sistema di configurazione](xref:fundamentals/configuration/index) usando il [modello di opzioni](xref:fundamentals/configuration/options). Ad esempio, dal *appsettings.json* file:

[!code-json[](filters/3.1sample/FiltersSample/appsettings.json)]

Nel `StartUp.ConfigureServices` eseguire le operazioni seguenti:

* La `PositionOptions` classe viene aggiunta al contenitore del servizio con l' `"Position"` area di configurazione.
* `MyActionFilterAttribute`Viene aggiunto al contenitore del servizio.

[!code-csharp[](./filters/3.1sample/FiltersSample/StartupAF.cs?name=snippet)]

Il codice seguente illustra la `PositionOptions` classe:

[!code-csharp[](filters/3.1sample/FiltersSample/Helper/PositionOptions.cs?name=snippet)]

Il codice seguente applica al `MyActionFilterAttribute` `Index2` Metodo:

[!code-csharp[](./filters/3.1sample/FiltersSample/Controllers/SampleController.cs?name=snippet2&highlight=9)]

In **intestazioni di risposta**, `author: Rick Anderson` e `Editor: Joe Smith` viene visualizzato quando `Sample/Index2` viene chiamato l'endpoint.

Il codice seguente applica `MyActionFilterAttribute` e alla `AddHeaderAttribute` Razor pagina:

[!code-csharp[](filters/3.1sample/FiltersSample/Pages/Movies/Index.cshtml.cs?name=snippet)]

Non è possibile applicare i filtri ai Razor metodi del gestore di pagina. Possono essere applicati al Razor modello di pagina o a livello globale.

Molte delle interfacce del filtro presentano attributi corrispondenti che possono essere usati come classi di base per implementazioni personalizzate.

Attributi dei filtri:

* <xref:Microsoft.AspNetCore.Mvc.Filters.ActionFilterAttribute>
* <xref:Microsoft.AspNetCore.Mvc.Filters.ExceptionFilterAttribute>
* <xref:Microsoft.AspNetCore.Mvc.Filters.ResultFilterAttribute>
* <xref:Microsoft.AspNetCore.Mvc.FormatFilterAttribute>
* <xref:Microsoft.AspNetCore.Mvc.ServiceFilterAttribute>
* <xref:Microsoft.AspNetCore.Mvc.TypeFilterAttribute>

## <a name="filter-scopes-and-order-of-execution"></a>Ambiti dei filtri e ordine di esecuzione

È possibile aggiungere un filtro alla pipeline in uno dei tre *ambiti*:

* Uso di un attributo in un'azione del controller. Gli attributi di filtro non possono essere applicati ai Razor metodi del gestore di pagine.
* Uso di un attributo in un controller o in una Razor pagina.
* A livello globale per tutti i controller, le azioni e le Razor pagine, come illustrato nel codice seguente:

[!code-csharp[](./filters/3.1sample/FiltersSample/StartupOrder.cs?name=snippet)]

### <a name="default-order-of-execution"></a>Ordine di esecuzione predefinito

Quando sono presenti più filtri per una particolare fase della pipeline, l'ambito determina l'ordine di esecuzione predefinito dei filtri.  I filtri globali racchiudono i filtri di classe, che a loro volta racchiudono i filtri dei metodi.

Come risultato dell'annidamento dei filtri, il codice *after* dei filtri viene eseguito in ordine inverso rispetto al codice *before*. Sequenza di filtro:

* Codice *before* dei filtri globali.
  * Codice *prima* dei filtri di pagina e controller Razor .
    * Codice *before* dei filtri del metodo di azione.
    * Codice *after* dei filtri del metodo di azione.
  * Codice *dopo* i filtri del controller e della Razor pagina.
* Codice *after* dei filtri globali.
  
L'esempio seguente illustra l'ordine in cui i metodi dei filtri vengono chiamati per i filtri di azione sincroni.

| Sequenza | Ambito del filtro | Metodo del filtro |
|:--------:|:------------:|:-------------:|
| 1 | Globale | `OnActionExecuting` |
| 2 | Controller o Razor pagina| `OnActionExecuting` |
| 3 | Metodo | `OnActionExecuting` |
| 4 | Metodo | `OnActionExecuted` |
| 5 | Controller o Razor pagina | `OnActionExecuted` |
| 6 | Globale | `OnActionExecuted` |

### <a name="controller-level-filters"></a>Filtri a livello di controller

Ogni controller che eredita dalla classe di base <xref:Microsoft.AspNetCore.Mvc.Controller> include i metodi [Controller.OnActionExecuting](xref:Microsoft.AspNetCore.Mvc.Controller.OnActionExecuting*), [Controller.OnActionExecutionAsync](xref:Microsoft.AspNetCore.Mvc.Controller.OnActionExecutionAsync*) e [Controller.OnActionExecuted](xref:Microsoft.AspNetCore.Mvc.Controller.OnActionExecuted*)
`OnActionExecuted`. Questi metodi:

* Eseguono il wrapping dei filtri che vengono eseguiti per una determinata azione.
* La chiamata di `OnActionExecuting` avviene prima di qualsiasi filtro dell'azione.
* La chiamata di `OnActionExecuted` avviene dopo tutti i filtri dell'azione.
* La chiamata di `OnActionExecutionAsync` avviene prima di qualsiasi filtro dell'azione. Il codice del filtro dopo `next` viene eseguito dopo il metodo di azione.

Ad esempio, l'applicazione di `MySampleActionFilter` nell'esempio scaricato avviene a livello globale all'avvio.

`TestController`:

* Applica `SampleActionFilterAttribute` ( `[SampleActionFilter]` ) all' `FilterTest2` azione.
* Esegue l'override di `OnActionExecuting` e `OnActionExecuted`.

[!code-csharp[](./filters/3.1sample/FiltersSample/Controllers/TestController.cs?name=snippet)]

[!INCLUDE[](~/includes/MyDisplayRouteInfo.md)]

<!-- test via  webBuilder.UseStartup<Startup>(); -->

Passando a `https://localhost:5001/Test/FilterTest2`, viene eseguito il codice seguente:

* `TestController.OnActionExecuting`
  * `MySampleActionFilter.OnActionExecuting`
    * `SampleActionFilterAttribute.OnActionExecuting`
      * `TestController.FilterTest2`
    * `SampleActionFilterAttribute.OnActionExecuted`
  * `MySampleActionFilter.OnActionExecuted`
* `TestController.OnActionExecuted`

Filtri a livello di controller impostare la proprietà [Order](https://github.com/dotnet/AspNetCore/blob/master/src/Mvc/Mvc.Core/src/Filters/ControllerActionFilter.cs#L15-L17) su `int.MinValue` . I filtri a livello di controller **non** possono essere impostati per l'esecuzione dopo i filtri applicati ai metodi. L'ordine è illustrato nella sezione successiva.

Per le Razor pagine, vedere [implementare Razor filtri di pagina eseguendo l'override dei metodi di filtro](xref:razor-pages/filter#implement-razor-page-filters-by-overriding-filter-methods).

### <a name="overriding-the-default-order"></a>Override dell'ordine predefinito

È possibile eseguire l'override della sequenza di esecuzione predefinita implementando <xref:Microsoft.AspNetCore.Mvc.Filters.IOrderedFilter>. `IOrderedFilter` espone la proprietà <xref:Microsoft.AspNetCore.Mvc.Filters.IOrderedFilter.Order>, che ha la precedenza sull'ambito per determinare l'ordine di esecuzione. Un filtro con un valore di `Order` inferiore:

* Esegue il codice *before* prima di quello di un filtro con un valore di `Order` più alto.
* Esegue il codice *after* dopo quello di un filtro con un valore di `Order` più alto.

La `Order` proprietà viene impostata con un parametro del costruttore:

[!code-csharp[](./filters/3.1sample/FiltersSample/Controllers/Test3Controller.cs?name=snippet)]

Considerare i due filtri azione nel controller seguente:

[!code-csharp[](./filters/3.1sample/FiltersSample/Controllers/Test2Controller.cs?name=snippet)]

In è stato aggiunto un filtro globale `StartUp.ConfigureServices` :

[!code-csharp[](./filters/3.1sample/FiltersSample/StartupOrder.cs?name=snippet)]

I 3 filtri vengono eseguiti nell'ordine seguente:

* `Test2Controller.OnActionExecuting`
  * `MySampleActionFilter.OnActionExecuting`
    * `MyAction2FilterAttribute.OnActionExecuting`
      * `Test2Controller.FilterTest2`
    * `MyAction2FilterAttribute.OnResultExecuting`
  * `MySampleActionFilter.OnActionExecuted`
* `Test2Controller.OnActionExecuted`

La proprietà `Order` ha la precedenza sull'ambito nel determinare l'ordine di esecuzione dei filtri. I filtri vengono ordinati prima in base all'ordine, poi viene usato l'ambito per interrompere i collegamenti. Tutti i filtri predefiniti implementano `IOrderedFilter` e impostano il valore predefinito di `Order` su 0. Come indicato in precedenza, i filtri a livello di controller impostano la proprietà [Order](https://github.com/dotnet/AspNetCore/blob/master/src/Mvc/Mvc.Core/src/Filters/ControllerActionFilter.cs#L15-L17) su `int.MinValue` per i filtri incorporati, l'ambito determina l'ordine a meno che non `Order` sia impostato su un valore diverso da zero.

Nel codice precedente, `MySampleActionFilter` ha ambito globale, quindi viene eseguito prima di `MyAction2FilterAttribute` , che ha ambito del controller. Per `MyAction2FilterAttribute` eseguire prima l'esecuzione, impostare l'ordine su `int.MinValue` :

[!code-csharp[](./filters/3.1sample/FiltersSample/Controllers/Test2Controller.cs?name=snippet2)]

Per eseguire prima l'esecuzione del filtro globale `MySampleActionFilter` , impostare `Order` su `int.MinValue` :

[!code-csharp[](./filters/3.1sample/FiltersSample/StartupOrder2.cs?name=snippet&highlight=6)]

## <a name="cancellation-and-short-circuiting"></a>Annullamento e corto circuito

È possibile causare il corto circuito della pipeline di filtro impostando la proprietà <xref:Microsoft.AspNetCore.Mvc.Filters.ResourceExecutingContext.Result> sul parametro <xref:Microsoft.AspNetCore.Mvc.Filters.ResourceExecutingContext> fornito al metodo di filtro. Ad esempio, il filtro di risorse seguente impedisce l'esecuzione del resto della pipeline:

<a name="short-circuiting-resource-filter"></a>

[!code-csharp[](./filters/3.1sample/FiltersSample/Filters/ShortCircuitingResourceFilterAttribute.cs?name=snippet)]

Nel codice seguente sia il filtro `ShortCircuitingResourceFilter` che il filtro `AddHeader` hanno come destinazione il metodo di azione `SomeResource`. `ShortCircuitingResourceFilter`:

* Viene eseguito per primo perché è un filtro risorsa e `AddHeader` è un filtro azione.
* Blocca il resto della pipeline.

Pertanto il filtro `AddHeader` non viene mai eseguito per l'azione `SomeResource`. Questo comportamento sarebbe lo stesso se entrambi i filtri venissero applicati a livello di metodo di azione, a condizione che il filtro `ShortCircuitingResourceFilter` venga eseguito prima. `ShortCircuitingResourceFilter` viene eseguito per primo per il tipo di filtro o per l'uso esplicito della proprietà `Order`.

[!code-csharp[](./filters/3.1sample/FiltersSample/Controllers/SampleController.cs?name=snippet3&highlight=1,15)]

## <a name="dependency-injection"></a>Inserimento di dipendenze

È possibile aggiungere filtri per tipo o per istanza. Se viene aggiunta un'istanza, tale istanza viene usata per ogni richiesta. Se viene aggiunto un tipo, il filtro è attivato dal tipo. Un filtro attivato dal tipo comporta:

* La creazione di un'istanza per ogni richiesta.
* Il popolamento di tutte le dipendenze del costruttore tramite [inserimento delle dipendenze](xref:fundamentals/dependency-injection).

I filtri implementati come attributi e aggiunti direttamente alle classi controller o ai metodi di azione non possono avere dipendenze costruttore specificate dall'[inserimento di dipendenze](xref:fundamentals/dependency-injection). Le dipendenze del costruttore non possono essere fornite tramite inserimento delle dipendenze in quanto:

* I parametri del costruttore devono essere forniti agli attributi nella posizione in cui vengono applicati. 
* Questa è una limitazione del funzionamento degli attributi.

I filtri seguenti supportano le dipendenze del costruttore fornite dall'inserimento delle dipendenze:

* <xref:Microsoft.AspNetCore.Mvc.ServiceFilterAttribute>
* <xref:Microsoft.AspNetCore.Mvc.TypeFilterAttribute>
* <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory> implementato nell'attributo.

I filtri precedenti possono essere applicati a un controller o a un metodo di azione:

I logger sono disponibili tramite l'inserimento delle dipendenze. Evitare tuttavia di creare e usare filtri esclusivamente per scopi di registrazione. La funzionalità di [registrazione del framework predefinita](xref:fundamentals/logging/index) in genere fornisce il necessario per la registrazione. La registrazione aggiunta ai filtri:

* Deve concentrarsi su problemi del dominio di business o comportamenti specifici del filtro.
* **Non** deve registrare azioni o altri eventi del framework. Le azioni di log dei filtri incorporati e gli eventi del Framework.

### <a name="servicefilterattribute"></a>ServiceFilterAttribute

I tipi di implementazione del filtro di servizi sono registrati in `ConfigureServices`. <xref:Microsoft.AspNetCore.Mvc.ServiceFilterAttribute> recupera un'istanza del filtro dall'inserimento di dipendenze.

Il codice seguente illustra `AddHeaderResultServiceFilter`:

[!code-csharp[](./filters/3.1sample/FiltersSample/Filters/LoggingAddHeaderFilter.cs?name=snippet_ResultFilter)]

Nel codice seguente l'oggetto `AddHeaderResultServiceFilter` viene aggiunto al contenitore di inserimento delle dipendenze:

[!code-csharp[](./filters/3.1sample/FiltersSample/Startup.cs?name=snippet&highlight=4)]

Nel codice seguente l'attributo `ServiceFilter` recupera un'istanza del filtro `AddHeaderResultServiceFilter` dall'inserimento delle dipendenze:

[!code-csharp[](./filters/3.1sample/FiltersSample/Controllers/HomeController.cs?name=snippet_ServiceFilter&highlight=1)]

Quando si usa l'impostazione `ServiceFilterAttribute`, [ServiceFilterAttribute.IsReusable](xref:Microsoft.AspNetCore.Mvc.ServiceFilterAttribute.IsReusable):

* Indica che l'istanza del filtro *può* essere riutilizzata all'esterno dell'ambito della richiesta in cui è stata creata. Il runtime di ASP.NET Core non garantisce:

  * Che venga creata una singola istanza del filtro.
  * Che il filtro non verrà richiesto di nuovo dal contenitore di inserimento delle dipendenze in un momento successivo.

* Non si deve usare con un filtro che dipende da servizi con una durata diversa da singleton.

 <xref:Microsoft.AspNetCore.Mvc.ServiceFilterAttribute> implementa <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory>. `IFilterFactory` espone il metodo <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory.CreateInstance*> per creare un'istanza <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterMetadata>. `CreateInstance` carica il tipo specificato dall'inserimento delle dipendenze.

### <a name="typefilterattribute"></a>TypeFilterAttribute

<xref:Microsoft.AspNetCore.Mvc.TypeFilterAttribute> è simile a <xref:Microsoft.AspNetCore.Mvc.ServiceFilterAttribute>, ma il relativo tipo non viene risolto direttamente dal contenitore dell'inserimento di dipendenze. Viene creata un'istanza del tipo tramite <xref:Microsoft.Extensions.DependencyInjection.ObjectFactory?displayProperty=fullName>.

Poiché i tipi `TypeFilterAttribute` non vengono risolti direttamente dal contenitore di inserimento delle dipendenze:

* Non è necessario che i tipi a cui viene fatto riferimento tramite `TypeFilterAttribute` siano registrati nel contenitore di inserimento delle dipendenze.  Le loro dipendenze vengono soddisfatte dal contenitore di inserimento delle dipendenze.
* In via facoltativa, `TypeFilterAttribute` può anche accettare gli argomenti del costruttore per il tipo.

Quando si usa l'impostazione `TypeFilterAttribute`, [TypeFilterAttribute.IsReusable](xref:Microsoft.AspNetCore.Mvc.TypeFilterAttribute.IsReusable):
* Indica che l'istanza del filtro *può* essere riutilizzata all'esterno dell'ambito della richiesta in cui è stata creata. Il runtime di ASP.NET Core non fornisce alcuna garanzia che venga creata una singola istanza del filtro.

* Non si deve usare con un filtro che dipende da servizi con una durata diversa da singleton.

L'esempio seguente illustra come passare argomenti a un tipo usando `TypeFilterAttribute`:

[!code-csharp[](filters/3.1sample/FiltersSample/Controllers/HomeController.cs?name=snippet_TypeFilter&highlight=1,2)]

<!-- 
https://localhost:5001/home/hi?name=joe
VS debug window shows 
FiltersSample.Filters.LogConstantFilter:Information: Method 'Hi' called
-->

## <a name="authorization-filters"></a>Filtri autorizzazione

I filtri di autorizzazione:

* Sono i primi filtri a essere eseguiti nella pipeline di filtro.
* Controllano l'accesso ai metodi di azione.
* Dispongono di un metodo precedente, ma non di un metodo successivo.

I filtri di autorizzazione personalizzati richiedono un framework di autorizzazione personalizzato. È preferibile configurare criteri di autorizzazione o scrivere criteri di autorizzazione personalizzati piuttosto che scrivere un filtro personalizzato. Il filtro di autorizzazione predefinito:

* Chiama il sistema di autorizzazione.
* Non autorizza le richieste.

**Non** generare eccezioni nei filtri di autorizzazione:

* L'eccezione non verrà gestita.
* I filtri di eccezione non gestiranno l'eccezione.

È consigliabile emettere una richiesta di verifica in caso di eccezione in un filtro di autorizzazione.

Altre informazioni sull'[autorizzazione](xref:security/authorization/introduction).

## <a name="resource-filters"></a>Filtri risorse

I filtri di risorse:

* Implementano l'interfaccia <xref:Microsoft.AspNetCore.Mvc.Filters.IResourceFilter> o <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncResourceFilter>.
* L'esecuzione racchiude la maggior parte della pipeline di filtro.
* Solo i [filtri di autorizzazione](#authorization-filters) vengono eseguiti prima dei filtri di risorse.

I filtri di risorse sono utili per causare il corto circuito della maggior parte della pipeline. Un filtro di memorizzazione nella cache può, ad esempio, evitare il resto della pipeline in caso di riscontro nella cache.

Esempi di filtri di risorse:

* Il [filtro di risorse di corto circuito](#short-circuiting-resource-filter) illustrato in precedenza.
* [DisableFormValueModelBindingAttribute](https://github.com/aspnet/Entropy/blob/master/samples/Mvc.FileUpload/Filters/DisableFormValueModelBindingAttribute.cs):

  * Impedisce all'associazione di modelli di accedere ai dati del modulo.
  * Viene usato quando si caricano file di grandi dimensioni per impedire la lettura dei dati del modulo in memoria.

## <a name="action-filters"></a>Filtri azione

I filtri azione **non** si applicano alle Razor pagine. Razor Pages supporta <xref:Microsoft.AspNetCore.Mvc.Filters.IPageFilter> e <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncPageFilter> . Per ulteriori informazioni, vedere [metodi di filtro per le Razor pagine](xref:razor-pages/filter).

I filtri di azione:

* Implementano l'interfaccia <xref:Microsoft.AspNetCore.Mvc.Filters.IActionFilter> o <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncActionFilter>.
* La loro esecuzione circonda l'esecuzione dei metodi di azione.

Il codice seguente mostra un filtro di azione di esempio:

[!code-csharp[](./filters/3.1sample/FiltersSample/Filters/MySampleActionFilter.cs?name=snippet_ActionFilter)]

La classe <xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutingContext> specifica le proprietà seguenti:

* <xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutingContext.ActionArguments> : consente di leggere gli input in un metodo di azione.
* <xref:Microsoft.AspNetCore.Mvc.Controller>: consente di modificare l'istanza del controller.
* <xref:System.Web.Mvc.ActionExecutingContext.Result>: l'impostazione di `Result` causa il corto circuito del metodo di azione e dei filtri di azione successivi.

La generazione di un'eccezione in un metodo di azione:

* Impedisce l'esecuzione dei filtri successivi.
* A differenza dell'impostazione di `Result`, è considerata un errore anziché un risultato positivo.

La classe <xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutedContext> specifica `Controller` e `Result` oltre alle proprietà seguenti:

* <xref:System.Web.Mvc.ActionExecutedContext.Canceled>: true se un altro file ha causato il corto circuito dell'azione.
* <xref:System.Web.Mvc.ActionExecutedContext.Exception>: non Null se l'azione o un filtro di azione eseguito in precedenza ha generato un'eccezione. Se si imposta questa proprietà su Null:

  * L'eccezione viene gestita in modo efficace.
  * L'oggetto `Result` viene eseguito come se fosse stato restituito dal metodo di azione.

Per un oggetto `IAsyncActionFilter`, una chiamata a <xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutionDelegate>:

* Esegue qualsiasi filtro azione successivo e il metodo di azione.
* Restituisce `ActionExecutedContext`.

Per causare il corto circuito, assegnare <xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutingContext.Result?displayProperty=fullName> a un'istanza del risultato e non chiamare `next` (`ActionExecutionDelegate`).

Il framework fornisce un oggetto <xref:Microsoft.AspNetCore.Mvc.Filters.ActionFilterAttribute> astratto che è possibile impostare come sottoclasse.

Il filtro di azione `OnActionExecuting` può essere usato per:

* Convalidare lo stato del modello.
* Restituire un errore se lo stato non è valido.

[!code-csharp[](./filters/3.1sample/FiltersSample/Filters/ValidateModelAttribute.cs?name=snippet)]

> [!NOTE]
> I controller annotati con l' `[ApiController]` attributo convalidano automaticamente lo stato del modello e restituiscono una risposta 400. Per altre informazioni, vedere [Risposte HTTP 400 automatiche](xref:web-api/index#automatic-http-400-responses).

Il metodo `OnActionExecuted` viene eseguito dopo il metodo di azione:

* È possibile visualizzare e modificare i risultati dell'azione tramite la proprietà <xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutedContext.Result>.
* L'impostazione di <xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutedContext.Canceled> è true se un altro filtro ha causato il corto circuito dell'esecuzione dell'azione.
* L'impostazione di <xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutedContext.Exception> è un valore non Null se l'azione o un filtro di azione successivo ha generato un'eccezione. Impostazione di `Exception` su null:

  * Gestisce un'eccezione in modo efficace.
  * L'oggetto `ActionExecutedContext.Result` viene eseguito come se fosse stato restituito normalmente dal metodo di azione.

[!code-csharp[](./filters/3.1sample/FiltersSample/Filters/ValidateModelAttribute.cs?name=snippet2&higlight=12-99)]

## <a name="exception-filters"></a>Filtri eccezioni

Filtri eccezioni:

* Implementano <xref:Microsoft.AspNetCore.Mvc.Filters.IExceptionFilter> o <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncExceptionFilter>.
* Possono essere usati per implementare criteri di gestione degli errori comuni.

Il filtro di eccezione di esempio seguente usa una visualizzazione degli errori personalizzata per mostrare i dettagli sulle eccezioni che si verificano quando l'app è in fase di sviluppo:

[!code-csharp[](./filters/3.1sample/FiltersSample/Filters/CustomExceptionFilter.cs?name=snippet_ExceptionFilter&highlight=16-19)]

Il codice seguente verifica il filtro eccezioni:

[!code-csharp[](filters/3.1sample/FiltersSample/Controllers/FailingController.cs?name=snippet)]

Filtri eccezioni:

* Non dispone di eventi precedenti o successivi.
* Implementano <xref:Microsoft.AspNetCore.Mvc.Filters.IExceptionFilter.OnException*> o <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncExceptionFilter.OnExceptionAsync*>.
* Gestire le eccezioni non gestite che si verificano nella Razor creazione della pagina o del controller, dell' [associazione di modelli](xref:mvc/models/model-binding), dei filtri azione o dei metodi di azione.
* **Non** rilevano le eccezioni che si verificano nei filtri di risorse, nei filtri dei risultati o nell'esecuzione dei risultati MVC.

Per gestire un'eccezione, impostare la proprietà <xref:System.Web.Mvc.ExceptionContext.ExceptionHandled> su `true` o scrivere una risposta. In questo modo si arresta la propagazione dell'eccezione. Un filtro di eccezione non può modificare un'eccezione in una "operazione riuscita". Questa operazione può essere eseguita solo tramite un filtro di azione.

Filtri eccezioni:

* Sono ideali per intercettare le eccezioni che si verificano nelle azioni.
* Non sono flessibili quanto il middleware di gestione degli errori.

Scegliere il middleware per la gestione delle eccezioni. Usare i filtri di eccezione solo quando la gestione degli errori *varia* in base al metodo di azione chiamato. Un'app può ad esempio avere metodi di azione sia per gli endpoint API che per le visualizzazioni/HTML. Gli endpoint dell'API possono restituire informazioni sull'errore come JSON, mentre le azioni basate sulla visualizzazione possono restituire una pagina di errore in formato HTML.

## <a name="result-filters"></a>Filtri risultato

I filtri dei risultati:

* Implementare un'interfaccia:
  * <xref:Microsoft.AspNetCore.Mvc.Filters.IResultFilter> o <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncResultFilter>
  * <xref:Microsoft.AspNetCore.Mvc.Filters.IAlwaysRunResultFilter> o <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncAlwaysRunResultFilter>
* La loro esecuzione circonda l'esecuzione dei risultati dell'azione.

### <a name="iresultfilter-and-iasyncresultfilter"></a>IResultFilter e IAsyncResultFilter

Il codice seguente illustra un filtro dei risultati che aggiunge un'intestazione HTTP:

[!code-csharp[](./filters/3.1sample/FiltersSample/Filters/LoggingAddHeaderFilter.cs?name=snippet_ResultFilter)]

Il tipo di risultato eseguito dipende dall'azione. Un'azione che restituisce una visualizzazione include tutte le operazioni di elaborazione Razor come parte dell' <xref:Microsoft.AspNetCore.Mvc.ViewResult> oggetto in esecuzione. Un metodo API può eseguire la serializzazione in quanto parte dell'esecuzione del risultato. Altre informazioni sui [risultati dell'azione](xref:mvc/controllers/actions).

I filtri dei risultati vengono eseguiti solo quando un filtro azione o azione produce un risultato di azione. I filtri dei risultati non vengono eseguiti nei casi seguenti:

* Un filtro di autorizzazione o un filtro risorse cortocircui la pipeline.
* Un filtro di eccezione non gestisca un'eccezione producendo un risultato dell'azione.

Il metodo <xref:Microsoft.AspNetCore.Mvc.Filters.IResultFilter.OnResultExecuting*?displayProperty=fullName> può causare il corto circuito dell'esecuzione del risultato dell'azione e dei filtri dei risultati successivi se si imposta <xref:Microsoft.AspNetCore.Mvc.Filters.ResultExecutingContext.Cancel?displayProperty=fullName> su `true`. In caso di corto circuito, scrivere nell'oggetto risposta per evitare di generare una risposta vuota. Generazione di un'eccezione in `IResultFilter.OnResultExecuting` :

* Impedisce l'esecuzione del risultato dell'azione e dei filtri successivi.
* Viene considerato un errore anziché un risultato positivo.

Quando <xref:Microsoft.AspNetCore.Mvc.Filters.IResultFilter.OnResultExecuted*?displayProperty=fullName> viene eseguito il metodo, è probabile che la risposta sia già stata inviata al client. Se la risposta è già stata inviata al client, non è possibile modificarla.

L'impostazione di `ResultExecutedContext.Canceled` è `true` se un altro filtro ha causato il corto circuito dell'esecuzione del risultato dell'azione.

L'impostazione di `ResultExecutedContext.Exception` è un valore non Null se il risultato dell'azione o un filtro dei risultati successivo ha generato un'eccezione. `Exception`L'impostazione di su null consente di gestire in modo efficace un'eccezione e impedisce che l'eccezione venga generata nuovamente in un secondo momento nella pipeline. Non c'è alcun modo affidabile per scrivere i dati in una risposta quando si gestisce un'eccezione in un filtro dei risultati. Se le intestazioni sono state scaricate nel client quando il risultato di un'azione genera un'eccezione, non c'è alcun meccanismo affidabile per inviare un codice di errore.

Per un oggetto <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncResultFilter>, una chiamata a `await next` in <xref:Microsoft.AspNetCore.Mvc.Filters.ResultExecutionDelegate> esegue tutti i filtri dei risultati successivi e il risultato dell'azione. Per causare il corto circuito, impostare [ResultExecutingContext.Cancel](xref:Microsoft.AspNetCore.Mvc.Filters.ResultExecutingContext.Cancel) su `true` e non chiamare `ResultExecutionDelegate`:

[!code-csharp[](./filters/3.1sample/FiltersSample/Filters/MyAsyncResponseFilter.cs?name=snippet)]

Il framework fornisce un oggetto `ResultFilterAttribute` astratto che è possibile impostare come sottoclasse. La classe [AddHeaderAttribute](#add-header-attribute) illustrata in precedenza è un esempio di un attributo di filtro dei risultati.

### <a name="ialwaysrunresultfilter-and-iasyncalwaysrunresultfilter"></a>IAlwaysRunResultFilter e IAsyncAlwaysRunResultFilter

Le interfacce <xref:Microsoft.AspNetCore.Mvc.Filters.IAlwaysRunResultFilter> e <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncAlwaysRunResultFilter> dichiarano un'implementazione di <xref:Microsoft.AspNetCore.Mvc.Filters.IResultFilter> eseguita per tutti i risultati dell'azione. Sono inclusi i risultati dell'azione prodotti da:

* Filtri di autorizzazione e filtri delle risorse che interessano il cortocircuito.
* Filtri eccezioni.

Ad esempio, il filtro seguente viene eseguito sempre e imposta un risultato dell'azione (<xref:Microsoft.AspNetCore.Mvc.ObjectResult>) con un codice di stato *422 Entità non elaborabile* quando la negoziazione del contenuto ha esito negativo:

[!code-csharp[](./filters/3.1sample/FiltersSample/Filters/UnprocessableResultFilter.cs?name=snippet)]

## <a name="ifilterfactory"></a>IFilterFactory

<xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory> implementa <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterMetadata>. Pertanto, un'istanza di `IFilterFactory` può essere usata come un'istanza di `IFilterMetadata` in un punto qualsiasi della pipeline filtro. Quando il runtime si prepara per richiamare il filtro, cerca di eseguirne il cast a un oggetto `IFilterFactory`. Se l'esecuzione del cast ha esito positivo, viene chiamato il metodo <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory.CreateInstance*> per creare l'istanza di `IFilterMetadata` richiamata. In questo modo viene specificata una struttura flessibile, poiché non è necessario impostare in modo esplicito la pipeline filtro all'avvio dell'app.

`IFilterFactory.IsReusable`:

* È un suggerimento della factory che l'istanza di filtro creata dalla factory può essere riutilizzata all'esterno dell'ambito della richiesta in cui è stata creata.
* ***Non*** deve essere usato con un filtro che dipende da servizi con una durata diversa da singleton.

Il runtime di ASP.NET Core non garantisce:

* Che venga creata una singola istanza del filtro.
* Che il filtro non verrà richiesto di nuovo dal contenitore di inserimento delle dipendenze in un momento successivo.

> [!WARNING] 
> Configurare <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory.IsReusable?displayProperty=nameWithType> per restituire solo `true` se l'origine dei filtri non è ambigua, i filtri sono senza stato e i filtri sono sicuri per l'uso in più richieste HTTP. Ad esempio, non restituire i filtri da DI DI cui è stata eseguita la registrazione come ambito o temporaneo se `IFilterFactory.IsReusable` restituisce `true` .

Un altro approccio alla creazione di filtri consiste nell'implementare `IFilterFactory` usando implementazioni dell'attributo personalizzate:

[!code-csharp[](./filters/3.1sample/FiltersSample/Filters/AddHeaderWithFactoryAttribute.cs?name=snippet_IFilterFactory&highlight=1,4,5,6,7)]

Il filtro viene applicato nel codice seguente:

[!code-csharp[](./filters/3.1sample/FiltersSample/Controllers/SampleController.cs?name=snippet3&highlight=21)]

Testare il codice precedente eseguendo l' [esempio di download](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/controllers/filters/3.1sample):

* Richiamare gli strumenti di sviluppo F12.
* Accedere a `https://localhost:5001/Sample/HeaderWithFactory`.

Gli strumenti di sviluppo F12 visualizzano le intestazioni di risposta seguenti aggiunte dal codice di esempio:

* **Autore:**`Rick Anderson`
* **globaladdheader:** `Result filter added to MvcOptions.Filters`
* **interno:**`My header`

Il codice precedente crea l'intestazione di risposta **internal:** `My header`.

### <a name="ifilterfactory-implemented-on-an-attribute"></a>Implementazione di IFilterFactory in un attributo

<!-- Review 
This section needs to be rewritten.
What's a non-named attribute?
-->

I filtri che implementano `IFilterFactory` sono utili per i filtri che:

* Non richiedono il passaggio di parametri.
* Hanno dipendenze del costruttore che devono essere soddisfatte dall'inserimento delle dipendenze.

<xref:Microsoft.AspNetCore.Mvc.TypeFilterAttribute> implementa <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory>. `IFilterFactory` espone il metodo <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory.CreateInstance*> per creare un'istanza <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterMetadata>. `CreateInstance` carica il tipo specificato dal contenitore di inserimento delle dipendenze.

[!code-csharp[](./filters/3.1sample/FiltersSample/Filters/SampleActionFilterAttribute.cs?name=snippet_TypeFilterAttribute&highlight=1,3,7)]

Il codice seguente illustra tre approcci all'applicazione di `[SampleActionFilter]`:

[!code-csharp[](./filters/3.1sample/FiltersSample/Controllers/HomeController.cs?name=snippet&highlight=1)]

Nel codice precedente la decorazione del metodo con `[SampleActionFilter]` è l'approccio da preferire per applicare `SampleActionFilter`.

## <a name="using-middleware-in-the-filter-pipeline"></a>Uso di middleware nella pipeline filtro

I filtri risorse funzionano come [middleware](xref:fundamentals/middleware/index) in quanto racchiudono l'esecuzione di tutto ciò che viene dopo nella pipeline. Tuttavia, i filtri differiscono dal middleware in quanto fanno parte del runtime, il che significa che hanno accesso a contesto e costrutti.

Per usare il middleware come filtro, creare un tipo con un metodo `Configure` che specifica il middleware da inserire nella pipeline di filtro. L'esempio seguente usa il middleware di localizzazione per stabilire le impostazioni cultura correnti per una richiesta:

[!code-csharp[](./filters/3.1sample/FiltersSample/Filters/LocalizationPipeline.cs?name=snippet_MiddlewareFilter&highlight=3,22)]

Usare <xref:Microsoft.AspNetCore.Mvc.MiddlewareFilterAttribute> per eseguire il middleware:

[!code-csharp[](./filters/3.1sample/FiltersSample/Controllers/HomeController.cs?name=snippet_MiddlewareFilter&highlight=2)]

I filtri middleware vengono eseguiti nella stessa fase della pipeline filtro come filtri risorse, prima dell'associazione di modelli e dopo il resto della pipeline.

## <a name="next-actions"></a>Azioni successive

* Vedere [metodi di filtro per le Razor pagine](xref:razor-pages/filter).
* Per sperimentare i filtri, [scaricare, testare e modificare l'esempio di GitHub](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/controllers/filters/3.1sample).

::: moniker-end

::: moniker range="< aspnetcore-3.0"

Di [Kirk Larkin](https://github.com/serpent5), [Rick Anderson](https://twitter.com/RickAndMSFT), [Tom Dykstra](https://github.com/tdykstra/) e [Steve Smith](https://ardalis.com/)

I *filtri* in ASP.NET Core consentono l'esecuzione del codice prima o dopo fasi specifiche della pipeline di elaborazione della richiesta.

I filtri predefiniti gestiscono attività, ad esempio:

* Autorizzazione (impedire l'accesso alle risorse per cui un utente non è autorizzato).
* Memorizzazione nella cache delle risposte (blocco della pipeline delle richieste per restituire una risposta memorizzata nella cache).

I filtri personalizzati possono essere creati per gestire problemi relativi a più settori. Esempi di problematiche trasversali includono la gestione degli errori, la memorizzazione nella cache, la configurazione, l'autorizzazione e la registrazione.  I filtri evitano la duplicazione del codice. Ad esempio, un filtro eccezioni per la gestione degli errori potrebbe consolidare la gestione degli errori.

Questo documento si applica a Razor pagine, controller API e controller con visualizzazioni.

[Visualizzare o scaricare un esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/controllers/filters/sample) ([procedura per il download](xref:index#how-to-download-a-sample)).

## <a name="how-filters-work"></a>Funzionamento dei filtri

I filtri vengono eseguiti all'interno della *pipeline di chiamata di azioni ASP.NET Core*, talvolta definita *pipeline di filtro*.  La pipeline di filtro viene eseguita dopo che ASP.NET Core ha selezionato l'azione da eseguire.

![La richiesta viene elaborata tramite altro middleware, il middleware di routing, la selezione dell'azione e la pipeline di chiamata di azioni ASP.NET Core. L'elaborazione della richiesta torna alla selezione azione, al middleware di routing e a diversi altri middleware prima di diventare la risposta che viene inviata al client.](filters/_static/filter-pipeline-1.png)

### <a name="filter-types"></a>Tipi di filtro

Ogni tipo di filtro viene eseguito in una fase diversa della pipeline di filtro:

* I [filtri di autorizzazione](#authorization-filters) vengono eseguiti per primi e consentono di determinare se l'utente è autorizzato per la richiesta. I filtri di autorizzazione causano il corto circuito della pipeline se la richiesta non è autorizzata.

* [Filtri risorse](#resource-filters):

  * Vengono eseguiti dopo l'autorizzazione.  
  * <xref:Microsoft.AspNetCore.Mvc.Filters.IResourceFilter.OnResourceExecuting*> può eseguire codice prima del resto della pipeline di filtro. Ad esempio, `OnResourceExecuting` può eseguire codice prima dell'associazione di modelli.
  * <xref:Microsoft.AspNetCore.Mvc.Filters.IResourceFilter.OnResourceExecuted*> può eseguire codice dopo che il resto della pipeline è stato completato.

* I [filtri azione](#action-filters) possono eseguire codice immediatamente prima e dopo la chiamata di un metodo di azione singolo. Possono essere usati per modificare gli argomenti passati a un'azione e il risultato restituito dall'azione. I filtri azione **non** sono supportati nelle Razor pagine.

* I [filtri eccezioni](#exception-filters) vengono usati per applicare i criteri globali a eccezioni non gestite che si verificano prima che qualsiasi elemento sia stato scritto nel corpo della risposta.

* I [filtri risultato](#result-filters) possono eseguire codice immediatamente prima e dopo l'esecuzione di risultati di azioni singole. Vengono eseguiti solo quando il metodo di azione è stato eseguito correttamente. Sono utili per la logica che deve racchiudere l'esecuzione del formattatore o di una vista.

Il diagramma seguente illustra come interagiscono i tipi di filtro nella pipeline di filtro.

![La richiesta passa attraverso i filtri autorizzazione, i filtri risorse, l'associazione di modelli, i filtri azione, l'esecuzione dell'azione e la conversione del risultato dell'azione, i filtri eccezioni, i filtri risultato e l'esecuzione del risultato. All'uscita della pipeline, la richiesta viene elaborata solo dai filtri risultato e dai filtri risorse prima di diventare una risposta inviata al client.](filters/_static/filter-pipeline-2.png)

## <a name="implementation"></a>Implementazione

I filtri supportano sia implementazioni sincrone che asincrone tramite definizioni di interfaccia diverse.

I filtri sincroni possono eseguire codice prima (`On-Stage-Executing`) e dopo (`On-Stage-Executed`) la relativa fase della pipeline. La chiamata di `OnActionExecuting`, ad esempio, avviene prima della chiamata del metodo di azione. La chiamata di `OnActionExecuted` avviene dopo l'esecuzione del metodo di azione.

[!code-csharp[](./filters/sample/FiltersSample/Filters/MySampleActionFilter.cs?name=snippet_ActionFilter)]

I filtri asincroni definiscono un metodo `On-Stage-ExecutionAsync`:

[!code-csharp[](./filters/sample/FiltersSample/Filters/SampleAsyncActionFilter.cs?name=snippet)]

Nel codice precedente `SampleAsyncActionFilter` ha un oggetto <xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutionDelegate> (`next`) che esegue il metodo di azione.  Ognuno dei metodi `On-Stage-ExecutionAsync` accetta un oggetto `FilterType-ExecutionDelegate` che esegue la fase della pipeline del filtro.

### <a name="multiple-filter-stages"></a>Fasi di filtro multiple

È possibile implementare interfacce per più fasi di filtro in una singola classe. Ad esempio, la classe <xref:Microsoft.AspNetCore.Mvc.Filters.ActionFilterAttribute> implementa `IActionFilter`, `IResultFilter` e i relativi equivalenti asincroni.

Implementare la versione sincrona **oppure** la versione asincrona di un'interfaccia di filtro, **non** entrambe. Il runtime controlla per prima cosa se il filtro implementa l'interfaccia asincrona e, in tal caso, la chiama. In caso contrario, chiama i metodi dell'interfaccia sincrona. Se in una classe sono implementate entrambe le interfacce, sincrona e asincrona, viene chiamato solo il metodo asincrono. Quando si usano classi astratte come <xref:Microsoft.AspNetCore.Mvc.Filters.ActionFilterAttribute> eseguire l'override solo dei metodi sincroni o del metodo asincrono per ogni tipo di filtro.

### <a name="built-in-filter-attributes"></a>Attributi filtro predefiniti

ASP.NET Core include filtri predefiniti basati su attributi che è possibile impostare come sottoclasse e personalizzare. Il filtro dei risultati seguente, ad esempio, aggiunge un'intestazione alla risposta:

<a name="add-header-attribute"></a>

[!code-csharp[](./filters/sample/FiltersSample/Filters/AddHeaderAttribute.cs?name=snippet)]

Gli attributi consentono ai filtri di accettare argomenti, come illustrato nell'esempio precedente. Applicare `AddHeaderAttribute` a un controller o a un metodo di azione e specificare il nome e il valore dell'intestazione HTTP:

[!code-csharp[](./filters/sample/FiltersSample/Controllers/SampleController.cs?name=snippet_AddHeader&highlight=1)]

<!-- `https://localhost:5001/Sample` -->

Molte delle interfacce del filtro presentano attributi corrispondenti che possono essere usati come classi di base per implementazioni personalizzate.

Attributi dei filtri:

* <xref:Microsoft.AspNetCore.Mvc.Filters.ActionFilterAttribute>
* <xref:Microsoft.AspNetCore.Mvc.Filters.ExceptionFilterAttribute>
* <xref:Microsoft.AspNetCore.Mvc.Filters.ResultFilterAttribute>
* <xref:Microsoft.AspNetCore.Mvc.FormatFilterAttribute>
* <xref:Microsoft.AspNetCore.Mvc.ServiceFilterAttribute>
* <xref:Microsoft.AspNetCore.Mvc.TypeFilterAttribute>

## <a name="filter-scopes-and-order-of-execution"></a>Ambiti dei filtri e ordine di esecuzione

È possibile aggiungere un filtro alla pipeline in uno dei tre *ambiti*:

* Usando un attributo di un'azione.
* Usando un attributo di un controller.
* A livello globale per tutti i controller e le azioni come illustrato nel codice seguente:

[!code-csharp[](./filters/sample/FiltersSample/StartupGF.cs?name=snippet_ConfigureServices)]

Il codice precedente aggiunge tre filtri a livello globale tramite la raccolta [MvcOptions.Filters](xref:Microsoft.AspNetCore.Mvc.MvcOptions.Filters).

### <a name="default-order-of-execution"></a>Ordine di esecuzione predefinito

Quando sono presenti più filtri *dello stesso tipo*, scope determina l'ordine predefinito di esecuzione del filtro.  Filtri globali circondano i filtri di classe. Filtri di classe filtri di metodo Racchiudi.

Come risultato dell'annidamento dei filtri, il codice *after* dei filtri viene eseguito in ordine inverso rispetto al codice *before*. Sequenza di filtro:

* Codice *before* dei filtri globali.
  * Codice *before* dei filtri del controller.
    * Codice *before* dei filtri del metodo di azione.
    * Codice *after* dei filtri del metodo di azione.
  * Codice *after* dei filtri del controller.
* Codice *after* dei filtri globali.
  
L'esempio seguente illustra l'ordine in cui i metodi dei filtri vengono chiamati per i filtri di azione sincroni.

| Sequenza | Ambito del filtro | Metodo del filtro |
|:--------:|:------------:|:-------------:|
| 1 | Globale | `OnActionExecuting` |
| 2 | Controller | `OnActionExecuting` |
| 3 | Metodo | `OnActionExecuting` |
| 4 | Metodo | `OnActionExecuted` |
| 5 | Controller | `OnActionExecuted` |
| 6 | Globale | `OnActionExecuted` |

Questa sequenza mostra che:

* Il filtro del metodo è annidato all'interno del filtro del controller.
* Il filtro del controller è annidato all'interno del filtro globale.

### <a name="controller-and-razor-page-level-filters"></a>Filtri a Razor livello di pagina e controller

Ogni controller che eredita dalla classe di base <xref:Microsoft.AspNetCore.Mvc.Controller> include i metodi [Controller.OnActionExecuting](xref:Microsoft.AspNetCore.Mvc.Controller.OnActionExecuting*), [Controller.OnActionExecutionAsync](xref:Microsoft.AspNetCore.Mvc.Controller.OnActionExecutionAsync*) e [Controller.OnActionExecuted](xref:Microsoft.AspNetCore.Mvc.Controller.OnActionExecuted*)
`OnActionExecuted`. Questi metodi:

* Eseguono il wrapping dei filtri che vengono eseguiti per una determinata azione.
* La chiamata di `OnActionExecuting` avviene prima di qualsiasi filtro dell'azione.
* La chiamata di `OnActionExecuted` avviene dopo tutti i filtri dell'azione.
* La chiamata di `OnActionExecutionAsync` avviene prima di qualsiasi filtro dell'azione. Il codice del filtro dopo `next` viene eseguito dopo il metodo di azione.

Ad esempio, l'applicazione di `MySampleActionFilter` nell'esempio scaricato avviene a livello globale all'avvio.

`TestController`:

* Applica `SampleActionFilterAttribute` ( `[SampleActionFilter]` ) all' `FilterTest2` azione.
* Esegue l'override di `OnActionExecuting` e `OnActionExecuted`.

[!code-csharp[](./filters/sample/FiltersSample/Controllers/TestController.cs?name=snippet)]

Passando a `https://localhost:5001/Test/FilterTest2`, viene eseguito il codice seguente:

* `TestController.OnActionExecuting`
  * `MySampleActionFilter.OnActionExecuting`
    * `SampleActionFilterAttribute.OnActionExecuting`
      * `TestController.FilterTest2`
    * `SampleActionFilterAttribute.OnActionExecuted`
  * `MySampleActionFilter.OnActionExecuted`
* `TestController.OnActionExecuted`

Per le Razor pagine, vedere [implementare Razor filtri di pagina eseguendo l'override dei metodi di filtro](xref:razor-pages/filter#implement-razor-page-filters-by-overriding-filter-methods).

### <a name="overriding-the-default-order"></a>Override dell'ordine predefinito

È possibile eseguire l'override della sequenza di esecuzione predefinita implementando <xref:Microsoft.AspNetCore.Mvc.Filters.IOrderedFilter>. `IOrderedFilter` espone la proprietà <xref:Microsoft.AspNetCore.Mvc.Filters.IOrderedFilter.Order>, che ha la precedenza sull'ambito per determinare l'ordine di esecuzione. Un filtro con un valore di `Order` inferiore:

* Esegue il codice *before* prima di quello di un filtro con un valore di `Order` più alto.
* Esegue il codice *after* dopo quello di un filtro con un valore di `Order` più alto.

La proprietà `Order` può essere impostata con un parametro del costruttore:

```csharp
[MyFilter(Name = "Controller Level Attribute", Order=1)]
```

Prendere in considerazione gli stessi tre filtri di azione illustrati nell'esempio precedente. Se la proprietà `Order` del controller e dei filtri globali è impostata, rispettivamente, su 1 e su 2, l'ordine di esecuzione viene invertito.

| Sequenza | Ambito del filtro | Proprietà`Order` | Metodo del filtro |
|:--------:|:------------:|:-----------------:|:-------------:|
| 1 | Metodo | 0 | `OnActionExecuting` |
| 2 | Controller | 1  | `OnActionExecuting` |
| 3 | Globale | 2  | `OnActionExecuting` |
| 4 | Globale | 2  | `OnActionExecuted` |
| 5 | Controller | 1  | `OnActionExecuted` |
| 6 | Metodo | 0  | `OnActionExecuted` |

La proprietà `Order` ha la precedenza sull'ambito nel determinare l'ordine di esecuzione dei filtri. I filtri vengono ordinati prima in base all'ordine, poi viene usato l'ambito per interrompere i collegamenti. Tutti i filtri predefiniti implementano `IOrderedFilter` e impostano il valore predefinito di `Order` su 0. Per i filtri predefiniti, l'ambito determina l'ordine, a meno che non si imposti `Order` su un valore diverso da zero.

## <a name="cancellation-and-short-circuiting"></a>Annullamento e corto circuito

È possibile causare il corto circuito della pipeline di filtro impostando la proprietà <xref:Microsoft.AspNetCore.Mvc.Filters.ResourceExecutingContext.Result> sul parametro <xref:Microsoft.AspNetCore.Mvc.Filters.ResourceExecutingContext> fornito al metodo di filtro. Ad esempio, il filtro di risorse seguente impedisce l'esecuzione del resto della pipeline:

<a name="short-circuiting-resource-filter"></a>

[!code-csharp[](./filters/sample/FiltersSample/Filters/ShortCircuitingResourceFilterAttribute.cs?name=snippet)]

Nel codice seguente sia il filtro `ShortCircuitingResourceFilter` che il filtro `AddHeader` hanno come destinazione il metodo di azione `SomeResource`. `ShortCircuitingResourceFilter`:

* Viene eseguito per primo perché è un filtro risorsa e `AddHeader` è un filtro azione.
* Blocca il resto della pipeline.

Pertanto il filtro `AddHeader` non viene mai eseguito per l'azione `SomeResource`. Questo comportamento sarebbe lo stesso se entrambi i filtri venissero applicati a livello di metodo di azione, a condizione che il filtro `ShortCircuitingResourceFilter` venga eseguito prima. `ShortCircuitingResourceFilter` viene eseguito per primo per il tipo di filtro o per l'uso esplicito della proprietà `Order`.

[!code-csharp[](./filters/sample/FiltersSample/Controllers/SampleController.cs?name=snippet_AddHeader&highlight=1,9)]

## <a name="dependency-injection"></a>Inserimento di dipendenze

È possibile aggiungere filtri per tipo o per istanza. Se viene aggiunta un'istanza, tale istanza viene usata per ogni richiesta. Se viene aggiunto un tipo, il filtro è attivato dal tipo. Un filtro attivato dal tipo comporta:

* La creazione di un'istanza per ogni richiesta.
* Il popolamento di tutte le dipendenze del costruttore tramite [inserimento delle dipendenze](xref:fundamentals/dependency-injection).

I filtri implementati come attributi e aggiunti direttamente alle classi controller o ai metodi di azione non possono avere dipendenze costruttore specificate dall'[inserimento di dipendenze](xref:fundamentals/dependency-injection). Le dipendenze del costruttore non possono essere fornite tramite inserimento delle dipendenze in quanto:

* I parametri del costruttore devono essere forniti agli attributi nella posizione in cui vengono applicati. 
* Questa è una limitazione del funzionamento degli attributi.

I filtri seguenti supportano le dipendenze del costruttore fornite dall'inserimento delle dipendenze:

* <xref:Microsoft.AspNetCore.Mvc.ServiceFilterAttribute>
* <xref:Microsoft.AspNetCore.Mvc.TypeFilterAttribute>
* <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory> implementato nell'attributo.

I filtri precedenti possono essere applicati a un controller o a un metodo di azione:

I logger sono disponibili tramite l'inserimento delle dipendenze. Evitare tuttavia di creare e usare filtri esclusivamente per scopi di registrazione. La funzionalità di [registrazione del framework predefinita](xref:fundamentals/logging/index) in genere fornisce il necessario per la registrazione. La registrazione aggiunta ai filtri:

* Deve concentrarsi su problemi del dominio di business o comportamenti specifici del filtro.
* **Non** deve registrare azioni o altri eventi del framework. I filtri predefiniti registrano azioni ed eventi del framework.

### <a name="servicefilterattribute"></a>ServiceFilterAttribute

I tipi di implementazione del filtro di servizi sono registrati in `ConfigureServices`. <xref:Microsoft.AspNetCore.Mvc.ServiceFilterAttribute> recupera un'istanza del filtro dall'inserimento di dipendenze.

Il codice seguente illustra `AddHeaderResultServiceFilter`:

[!code-csharp[](./filters/sample/FiltersSample/Filters/LoggingAddHeaderFilter.cs?name=snippet_ResultFilter)]

Nel codice seguente l'oggetto `AddHeaderResultServiceFilter` viene aggiunto al contenitore di inserimento delle dipendenze:

[!code-csharp[](./filters/sample/FiltersSample/Startup.cs?name=snippet_ConfigureServices&highlight=4)]

Nel codice seguente l'attributo `ServiceFilter` recupera un'istanza del filtro `AddHeaderResultServiceFilter` dall'inserimento delle dipendenze:

[!code-csharp[](./filters/sample/FiltersSample/Controllers/HomeController.cs?name=snippet_ServiceFilter&highlight=1)]

Quando si usa l'impostazione `ServiceFilterAttribute`, [ServiceFilterAttribute.IsReusable](xref:Microsoft.AspNetCore.Mvc.ServiceFilterAttribute.IsReusable):

* Indica che l'istanza del filtro *può* essere riutilizzata all'esterno dell'ambito della richiesta in cui è stata creata. Il runtime di ASP.NET Core non garantisce:

  * Che venga creata una singola istanza del filtro.
  * Che il filtro non verrà richiesto di nuovo dal contenitore di inserimento delle dipendenze in un momento successivo.

* Non si deve usare con un filtro che dipende da servizi con una durata diversa da singleton.

 <xref:Microsoft.AspNetCore.Mvc.ServiceFilterAttribute> implementa <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory>. `IFilterFactory` espone il metodo <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory.CreateInstance*> per creare un'istanza <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterMetadata>. `CreateInstance` carica il tipo specificato dall'inserimento delle dipendenze.

### <a name="typefilterattribute"></a>TypeFilterAttribute

<xref:Microsoft.AspNetCore.Mvc.TypeFilterAttribute> è simile a <xref:Microsoft.AspNetCore.Mvc.ServiceFilterAttribute>, ma il relativo tipo non viene risolto direttamente dal contenitore dell'inserimento di dipendenze. Viene creata un'istanza del tipo tramite <xref:Microsoft.Extensions.DependencyInjection.ObjectFactory?displayProperty=fullName>.

Poiché i tipi `TypeFilterAttribute` non vengono risolti direttamente dal contenitore di inserimento delle dipendenze:

* Non è necessario che i tipi a cui viene fatto riferimento tramite `TypeFilterAttribute` siano registrati nel contenitore di inserimento delle dipendenze.  Le loro dipendenze vengono soddisfatte dal contenitore di inserimento delle dipendenze.
* In via facoltativa, `TypeFilterAttribute` può anche accettare gli argomenti del costruttore per il tipo.

Quando si usa l'impostazione `TypeFilterAttribute`, [TypeFilterAttribute.IsReusable](xref:Microsoft.AspNetCore.Mvc.TypeFilterAttribute.IsReusable):
* Indica che l'istanza del filtro *può* essere riutilizzata all'esterno dell'ambito della richiesta in cui è stata creata. Il runtime di ASP.NET Core non fornisce alcuna garanzia che venga creata una singola istanza del filtro.

* Non si deve usare con un filtro che dipende da servizi con una durata diversa da singleton.

L'esempio seguente illustra come passare argomenti a un tipo usando `TypeFilterAttribute`:

[!code-csharp[](../../mvc/controllers/filters/sample/FiltersSample/Controllers/HomeController.cs?name=snippet_TypeFilter&highlight=1,2)]
[!code-csharp[](../../mvc/controllers/filters/sample/FiltersSample/Filters/LogConstantFilter.cs?name=snippet_TypeFilter_Implementation&highlight=6)]

<!-- 
https://localhost:5001/home/hi?name=joe
VS debug window shows 
FiltersSample.Filters.LogConstantFilter:Information: Method 'Hi' called
-->

## <a name="authorization-filters"></a>Filtri autorizzazione

I filtri di autorizzazione:

* Sono i primi filtri a essere eseguiti nella pipeline di filtro.
* Controllano l'accesso ai metodi di azione.
* Dispongono di un metodo precedente, ma non di un metodo successivo.

I filtri di autorizzazione personalizzati richiedono un framework di autorizzazione personalizzato. È preferibile configurare criteri di autorizzazione o scrivere criteri di autorizzazione personalizzati piuttosto che scrivere un filtro personalizzato. Il filtro di autorizzazione predefinito:

* Chiama il sistema di autorizzazione.
* Non autorizza le richieste.

**Non** generare eccezioni nei filtri di autorizzazione:

* L'eccezione non verrà gestita.
* I filtri di eccezione non gestiranno l'eccezione.

È consigliabile emettere una richiesta di verifica in caso di eccezione in un filtro di autorizzazione.

Altre informazioni sull'[autorizzazione](xref:security/authorization/introduction).

## <a name="resource-filters"></a>Filtri risorse

I filtri di risorse:

* Implementano l'interfaccia <xref:Microsoft.AspNetCore.Mvc.Filters.IResourceFilter> o <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncResourceFilter>.
* L'esecuzione racchiude la maggior parte della pipeline di filtro.
* Solo i [filtri di autorizzazione](#authorization-filters) vengono eseguiti prima dei filtri di risorse.

I filtri di risorse sono utili per causare il corto circuito della maggior parte della pipeline. Un filtro di memorizzazione nella cache può, ad esempio, evitare il resto della pipeline in caso di riscontro nella cache.

Esempi di filtri di risorse:

* Il [filtro di risorse di corto circuito](#short-circuiting-resource-filter) illustrato in precedenza.
* [DisableFormValueModelBindingAttribute](https://github.com/aspnet/Entropy/blob/rel/2.0.0-preview2/samples/Mvc.FileUpload/Filters/DisableFormValueModelBindingAttribute.cs):

  * Impedisce all'associazione di modelli di accedere ai dati del modulo.
  * Viene usato quando si caricano file di grandi dimensioni per impedire la lettura dei dati del modulo in memoria.

## <a name="action-filters"></a>Filtri azione

> [!IMPORTANT]
> I filtri azione **non** si applicano alle Razor pagine. Razor Pages supporta <xref:Microsoft.AspNetCore.Mvc.Filters.IPageFilter> e <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncPageFilter> . Per ulteriori informazioni, vedere [metodi di filtro per le Razor pagine](xref:razor-pages/filter).

I filtri di azione:

* Implementano l'interfaccia <xref:Microsoft.AspNetCore.Mvc.Filters.IActionFilter> o <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncActionFilter>.
* La loro esecuzione circonda l'esecuzione dei metodi di azione.

Il codice seguente mostra un filtro di azione di esempio:

[!code-csharp[](./filters/sample/FiltersSample/Filters/MySampleActionFilter.cs?name=snippet_ActionFilter)]

La classe <xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutingContext> specifica le proprietà seguenti:

* <xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutingContext.ActionArguments>: consente di leggere gli input per un metodo di azione.
* <xref:Microsoft.AspNetCore.Mvc.Controller>: consente di modificare l'istanza del controller.
* <xref:System.Web.Mvc.ActionExecutingContext.Result>: l'impostazione di `Result` causa il corto circuito del metodo di azione e dei filtri di azione successivi.

La generazione di un'eccezione in un metodo di azione:

* Impedisce l'esecuzione dei filtri successivi.
* A differenza dell'impostazione di `Result`, è considerata un errore anziché un risultato positivo.

La classe <xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutedContext> specifica `Controller` e `Result` oltre alle proprietà seguenti:

* <xref:System.Web.Mvc.ActionExecutedContext.Canceled>: true se un altro file ha causato il corto circuito dell'azione.
* <xref:System.Web.Mvc.ActionExecutedContext.Exception>: non Null se l'azione o un filtro di azione eseguito in precedenza ha generato un'eccezione. Se si imposta questa proprietà su Null:

  * L'eccezione viene gestita in modo efficace.
  * L'oggetto `Result` viene eseguito come se fosse stato restituito dal metodo di azione.

Per un oggetto `IAsyncActionFilter`, una chiamata a <xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutionDelegate>:

* Esegue qualsiasi filtro azione successivo e il metodo di azione.
* Restituisce `ActionExecutedContext`.

Per causare il corto circuito, assegnare <xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutingContext.Result?displayProperty=fullName> a un'istanza del risultato e non chiamare `next` (`ActionExecutionDelegate`).

Il framework fornisce un oggetto <xref:Microsoft.AspNetCore.Mvc.Filters.ActionFilterAttribute> astratto che è possibile impostare come sottoclasse.

Il filtro di azione `OnActionExecuting` può essere usato per:

* Convalidare lo stato del modello.
* Restituire un errore se lo stato non è valido.

[!code-csharp[](./filters/sample/FiltersSample/Filters/ValidateModelAttribute.cs?name=snippet)]

Il metodo `OnActionExecuted` viene eseguito dopo il metodo di azione:

* È possibile visualizzare e modificare i risultati dell'azione tramite la proprietà <xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutedContext.Result>.
* L'impostazione di <xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutedContext.Canceled> è true se un altro filtro ha causato il corto circuito dell'esecuzione dell'azione.
* L'impostazione di <xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutedContext.Exception> è un valore non Null se l'azione o un filtro di azione successivo ha generato un'eccezione. Impostazione di `Exception` su null:

  * Gestisce un'eccezione in modo efficace.
  * L'oggetto `ActionExecutedContext.Result` viene eseguito come se fosse stato restituito normalmente dal metodo di azione.

[!code-csharp[](./filters/sample/FiltersSample/Filters/ValidateModelAttribute.cs?name=snippet2&higlight=12-99)]

## <a name="exception-filters"></a>Filtri eccezioni

Filtri eccezioni:

* Implementano <xref:Microsoft.AspNetCore.Mvc.Filters.IExceptionFilter> o <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncExceptionFilter>. 
* Possono essere usati per implementare criteri di gestione degli errori comuni.

Il filtro di eccezione di esempio seguente usa una visualizzazione degli errori personalizzata per mostrare i dettagli sulle eccezioni che si verificano quando l'app è in fase di sviluppo:

[!code-csharp[](./filters/sample/FiltersSample/Filters/CustomExceptionFilter.cs?name=snippet_ExceptionFilter&highlight=16-19)]

Filtri eccezioni:

* Non dispone di eventi precedenti o successivi.
* Implementano <xref:Microsoft.AspNetCore.Mvc.Filters.IExceptionFilter.OnException*> o <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncExceptionFilter.OnExceptionAsync*>.
* Gestire le eccezioni non gestite che si verificano nella Razor creazione della pagina o del controller, dell' [associazione di modelli](xref:mvc/models/model-binding), dei filtri azione o dei metodi di azione.
* **Non** rilevano le eccezioni che si verificano nei filtri di risorse, nei filtri dei risultati o nell'esecuzione dei risultati MVC.

Per gestire un'eccezione, impostare la proprietà <xref:System.Web.Mvc.ExceptionContext.ExceptionHandled> su `true` o scrivere una risposta. In questo modo si arresta la propagazione dell'eccezione. Un filtro di eccezione non può modificare un'eccezione in una "operazione riuscita". Questa operazione può essere eseguita solo tramite un filtro di azione.

Filtri eccezioni:

* Sono ideali per intercettare le eccezioni che si verificano nelle azioni.
* Non sono flessibili quanto il middleware di gestione degli errori.

Scegliere il middleware per la gestione delle eccezioni. Usare i filtri di eccezione solo quando la gestione degli errori *varia* in base al metodo di azione chiamato. Un'app può ad esempio avere metodi di azione sia per gli endpoint API che per le visualizzazioni/HTML. Gli endpoint dell'API possono restituire informazioni sull'errore come JSON, mentre le azioni basate sulla visualizzazione possono restituire una pagina di errore in formato HTML.

## <a name="result-filters"></a>Filtri risultato

I filtri dei risultati:

* Implementare un'interfaccia:
  * <xref:Microsoft.AspNetCore.Mvc.Filters.IResultFilter> o <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncResultFilter>
  * <xref:Microsoft.AspNetCore.Mvc.Filters.IAlwaysRunResultFilter> o <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncAlwaysRunResultFilter>
* La loro esecuzione circonda l'esecuzione dei risultati dell'azione.

### <a name="iresultfilter-and-iasyncresultfilter"></a>IResultFilter e IAsyncResultFilter

Il codice seguente illustra un filtro dei risultati che aggiunge un'intestazione HTTP:

[!code-csharp[](./filters/sample/FiltersSample/Filters/LoggingAddHeaderFilter.cs?name=snippet_ResultFilter)]

Il tipo di risultato eseguito dipende dall'azione. Un'azione che restituisce una visualizzazione include tutte le elaborazioni Razor come parte dell'elemento <xref:Microsoft.AspNetCore.Mvc.ViewResult> eseguito. Un metodo API può eseguire la serializzazione in quanto parte dell'esecuzione del risultato. Altre informazioni sui [risultati dell'azione](xref:mvc/controllers/actions).

I filtri dei risultati vengono eseguiti solo quando un filtro azione o azione produce un risultato di azione. I filtri dei risultati non vengono eseguiti nei casi seguenti:

* Un filtro di autorizzazione o un filtro risorse cortocircui la pipeline.
* Un filtro di eccezione non gestisca un'eccezione producendo un risultato dell'azione.

Il metodo <xref:Microsoft.AspNetCore.Mvc.Filters.IResultFilter.OnResultExecuting*?displayProperty=fullName> può causare il corto circuito dell'esecuzione del risultato dell'azione e dei filtri dei risultati successivi se si imposta <xref:Microsoft.AspNetCore.Mvc.Filters.ResultExecutingContext.Cancel?displayProperty=fullName> su `true`. In caso di corto circuito, scrivere nell'oggetto risposta per evitare di generare una risposta vuota. La generazione di un'eccezione in `IResultFilter.OnResultExecuting`:

* Impedisce l'esecuzione del risultato dell'azione e dei filtri successivi.
* È considerata un errore anziché un risultato positivo.

Quando <xref:Microsoft.AspNetCore.Mvc.Filters.IResultFilter.OnResultExecuted*?displayProperty=fullName> viene eseguito il metodo, è probabile che la risposta sia già stata inviata al client. Se la risposta è già stata inviata al client, non è possibile modificarla ulteriormente.

L'impostazione di `ResultExecutedContext.Canceled` è `true` se un altro filtro ha causato il corto circuito dell'esecuzione del risultato dell'azione.

L'impostazione di `ResultExecutedContext.Exception` è un valore non Null se il risultato dell'azione o un filtro dei risultati successivo ha generato un'eccezione. Se si imposta `Exception` su Null l'eccezione viene gestita in modo efficace e si evita che venga generata nuovamente da ASP.NET Core in un punto successivo della pipeline. Non c'è alcun modo affidabile per scrivere i dati in una risposta quando si gestisce un'eccezione in un filtro dei risultati. Se le intestazioni sono state scaricate nel client quando il risultato di un'azione genera un'eccezione, non c'è alcun meccanismo affidabile per inviare un codice di errore.

Per un oggetto <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncResultFilter>, una chiamata a `await next` in <xref:Microsoft.AspNetCore.Mvc.Filters.ResultExecutionDelegate> esegue tutti i filtri dei risultati successivi e il risultato dell'azione. Per causare il corto circuito, impostare [ResultExecutingContext.Cancel](xref:Microsoft.AspNetCore.Mvc.Filters.ResultExecutingContext.Cancel) su `true` e non chiamare `ResultExecutionDelegate`:

[!code-csharp[](./filters/sample/FiltersSample/Filters/MyAsyncResponseFilter.cs?name=snippet)]

Il framework fornisce un oggetto `ResultFilterAttribute` astratto che è possibile impostare come sottoclasse. La classe [AddHeaderAttribute](#add-header-attribute) illustrata in precedenza è un esempio di un attributo di filtro dei risultati.

### <a name="ialwaysrunresultfilter-and-iasyncalwaysrunresultfilter"></a>IAlwaysRunResultFilter e IAsyncAlwaysRunResultFilter

Le interfacce <xref:Microsoft.AspNetCore.Mvc.Filters.IAlwaysRunResultFilter> e <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncAlwaysRunResultFilter> dichiarano un'implementazione di <xref:Microsoft.AspNetCore.Mvc.Filters.IResultFilter> eseguita per tutti i risultati dell'azione. Sono inclusi i risultati dell'azione prodotti da:

* Filtri di autorizzazione e filtri delle risorse che interessano il cortocircuito.
* Filtri eccezioni.

Ad esempio, il filtro seguente viene eseguito sempre e imposta un risultato dell'azione (<xref:Microsoft.AspNetCore.Mvc.ObjectResult>) con un codice di stato *422 Entità non elaborabile* quando la negoziazione del contenuto ha esito negativo:

[!code-csharp[](./filters/sample/FiltersSample/Filters/UnprocessableResultFilter.cs?name=snippet)]

## <a name="ifilterfactory"></a>IFilterFactory

<xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory> implementa <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterMetadata>. Pertanto, un'istanza di `IFilterFactory` può essere usata come un'istanza di `IFilterMetadata` in un punto qualsiasi della pipeline filtro. Quando il runtime si prepara per richiamare il filtro, cerca di eseguirne il cast a un oggetto `IFilterFactory`. Se l'esecuzione del cast ha esito positivo, viene chiamato il metodo <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory.CreateInstance*> per creare l'istanza di `IFilterMetadata` richiamata. In questo modo viene specificata una struttura flessibile, poiché non è necessario impostare in modo esplicito la pipeline filtro all'avvio dell'app.

Un altro approccio alla creazione di filtri consiste nell'implementare `IFilterFactory` usando implementazioni dell'attributo personalizzate:

[!code-csharp[](./filters/sample/FiltersSample/Filters/AddHeaderWithFactoryAttribute.cs?name=snippet_IFilterFactory&highlight=1,4,5,6,7)]

Il codice precedente può essere testato eseguendo l'[esempio scaricato](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/controllers/filters/sample):

* Richiamare gli strumenti di sviluppo F12.
* Accedere a `https://localhost:5001/Sample/HeaderWithFactory`.

Gli strumenti di sviluppo F12 visualizzano le intestazioni di risposta seguenti aggiunte dal codice di esempio:

* **Autore:**`Joe Smith`
* **globaladdheader:** `Result filter added to MvcOptions.Filters`
* **interno:**`My header`

Il codice precedente crea l'intestazione di risposta **internal:** `My header`.

### <a name="ifilterfactory-implemented-on-an-attribute"></a>Implementazione di IFilterFactory in un attributo

<!-- Review 
This section needs to be rewritten.
What's a non-named attribute?
-->

I filtri che implementano `IFilterFactory` sono utili per i filtri che:

* Non richiedono il passaggio di parametri.
* Hanno dipendenze del costruttore che devono essere soddisfatte dall'inserimento delle dipendenze.

<xref:Microsoft.AspNetCore.Mvc.TypeFilterAttribute> implementa <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory>. `IFilterFactory` espone il metodo <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory.CreateInstance*> per creare un'istanza <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterMetadata>. `CreateInstance` carica il tipo specificato dal contenitore di inserimento delle dipendenze.

[!code-csharp[](./filters/sample/FiltersSample/Filters/SampleActionFilterAttribute.cs?name=snippet_TypeFilterAttribute&highlight=1,3,7)]

Il codice seguente illustra tre approcci all'applicazione di `[SampleActionFilter]`:

[!code-csharp[](./filters/sample/FiltersSample/Controllers/HomeController.cs?name=snippet&highlight=1)]

Nel codice precedente la decorazione del metodo con `[SampleActionFilter]` è l'approccio da preferire per applicare `SampleActionFilter`.

## <a name="using-middleware-in-the-filter-pipeline"></a>Uso di middleware nella pipeline filtro

I filtri risorse funzionano come [middleware](xref:fundamentals/middleware/index) in quanto racchiudono l'esecuzione di tutto ciò che viene dopo nella pipeline. I filtri sono diversi dal middleware in quanto fanno parte del runtime di ASP.NET Core, quindi hanno accesso al contesto e ai costrutti di ASP.NET Core.

Per usare il middleware come filtro, creare un tipo con un metodo `Configure` che specifica il middleware da inserire nella pipeline di filtro. L'esempio seguente usa il middleware di localizzazione per stabilire le impostazioni cultura correnti per una richiesta:

[!code-csharp[](./filters/sample/FiltersSample/Filters/LocalizationPipeline.cs?name=snippet_MiddlewareFilter&highlight=3,22)]

Usare <xref:Microsoft.AspNetCore.Mvc.MiddlewareFilterAttribute> per eseguire il middleware:

[!code-csharp[](./filters/sample/FiltersSample/Controllers/HomeController.cs?name=snippet_MiddlewareFilter&highlight=2)]

I filtri middleware vengono eseguiti nella stessa fase della pipeline filtro come filtri risorse, prima dell'associazione di modelli e dopo il resto della pipeline.

## <a name="next-actions"></a>Azioni successive

* Vedere [metodi di filtro per le Razor pagine](xref:razor-pages/filter).
* Per sperimentare i filtri, [scaricare, testare e modificare l'esempio di GitHub](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/controllers/filters/sample).

::: moniker-end
