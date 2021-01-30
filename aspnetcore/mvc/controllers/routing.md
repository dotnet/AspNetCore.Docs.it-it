---
title: Routing ad azioni del controller in ASP.NET Core
author: rick-anderson
description: Informazioni su come ASP.NET Core MVC usa middleware di routing per verificare la corrispondenza degli URL delle richieste in ingresso ed eseguirne il mapping alle azioni.
ms.author: riande
ms.date: 3/25/2020
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
uid: mvc/controllers/routing
ms.openlocfilehash: 04ece9628265135efd0dd92d29c8b14fc897329e
ms.sourcegitcommit: 83524f739dd25fbfa95ee34e95342afb383b49fe
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 01/29/2021
ms.locfileid: "99057356"
---
# <a name="routing-to-controller-actions-in-aspnet-core"></a>Routing ad azioni del controller in ASP.NET Core

Di [Ryan Nowak](https://github.com/rynowak), [Kirk Larkin](https://twitter.com/serpent5)e [Rick Anderson](https://twitter.com/RickAndMSFT)

::: moniker range=">= aspnetcore-3.0"

I controller di ASP.NET Core usano il [middleware](xref:fundamentals/middleware/index) di routing per trovare le corrispondenze con gli URL delle richieste in ingresso ed eseguirne il mapping alle [azioni](#action).  Modelli di route:

* Sono definiti nel codice di avvio o negli attributi.
* Descrizione della corrispondenza dei percorsi URL con le [azioni](#action).
* Vengono usati per generare URL per i collegamenti. I collegamenti generati vengono in genere restituiti nelle risposte.

Le azioni sono indirizzate [convenzionalmente](#cr) o [indirizzate agli attributi](#ar). L'inserimento di una route sul controller o sull' [azione](#action) lo rende instradato con attributi. Per altre informazioni, vedere [Routing misto](#routing-mixed-ref-label).

Questo documento:

* Illustra le interazioni tra MVC e il routing:
  * Il modo in cui le app MVC tipiche usano le funzionalità di routing.
  * Vengono illustrati entrambi:
    * [Routing convenzionale](#cr) usato in genere con i controller e le visualizzazioni.
    * *Routing degli attributi* usato con le API REST. Se si è interessati principalmente al routing per le API REST, passare alla sezione [routing degli attributi per le API REST](#ar) .
  * Vedere [routing](xref:fundamentals/routing) per i dettagli del routing avanzato.
* Si riferisce al sistema di routing predefinito aggiunto in ASP.NET Core 3,0, denominato endpoint routing. È possibile usare i controller con la versione precedente di routing per motivi di compatibilità. Per istruzioni, vedere la [Guida alla migrazione 2.2-3.0](xref:migration/22-to-30) . Per materiale di riferimento sul sistema di routing legacy, vedere la [versione 2,2 di questo documento](xref:mvc/controllers/routing?view=aspnetcore-2.2) .

<a name="cr"></a>

## <a name="set-up-conventional-route"></a>Configurare la route convenzionale

`Startup.Configure` Quando si usa il [routing convenzionale](#crd), in genere il codice è simile al seguente:

[!code-csharp[](routing/samples/3.x/main/StartupDefaultMVC.cs?name=snippet)]

All'interno della chiamata a <xref:Microsoft.AspNetCore.Builder.EndpointRoutingApplicationBuilderExtensions.UseEndpoints%2A> , <xref:Microsoft.AspNetCore.Builder.ControllerEndpointRouteBuilderExtensions.MapControllerRoute%2A> viene usato per creare una singola route. La route singola è denominata `default` Route. La maggior parte delle app con controller e visualizzazioni usa un modello di route simile a quello della `default` Route. Le API REST devono usare il [routing degli attributi](#ar).

Il modello di route `"{controller=Home}/{action=Index}/{id?}"` :

* Corrisponde a un percorso URL come `/Products/Details/5`
* Estrae i valori della route `{ controller = Products, action = Details, id = 5 }` suddivisione in token il percorso. L'estrazione dei valori di route comporta una corrispondenza se l'app dispone di un controller denominato `ProductsController` e di un' `Details` azione:

  [!code-csharp[](routing/samples/3.x/main/Controllers/ProductsController.cs?name=snippetA)]

  [!INCLUDE[](~/includes/MyDisplayRouteInfo.md)]

* `/Products/Details/5` il modello associa il valore di `id = 5` per impostare il `id` parametro su `5` . Per ulteriori informazioni, vedere [associazione di modelli](xref:mvc/models/model-binding) .
* `{controller=Home}` definisce `Home` come valore predefinito `controller` .
* `{action=Index}` definisce `Index` come valore predefinito `action` .
*  Il `?` carattere in `{id?}` definisce `id` come facoltativo.
  * I parametri di route predefiniti e facoltativi non devono necessariamente essere presenti nel percorso URL per trovare una corrispondenza. Vedere [Riferimento per i modelli di route](xref:fundamentals/routing#route-template-reference) per una descrizione dettagliata della sintassi del modello di route.
* Corrisponde al percorso dell'URL `/` .
* Produce i valori della route `{ controller = Home, action = Index }` .

I valori per `controller` e `action` usano i valori predefiniti. `id` non produce un valore poiché non esiste un segmento corrispondente nel percorso URL. `/` corrisponde solo se esiste un' `HomeController` azione e `Index` :

```csharp
public class HomeController : Controller
{
  public IActionResult Index() { ... }
}
```

Utilizzando la definizione del controller e il modello di route precedenti, l' `HomeController.Index` azione viene eseguita per i seguenti percorsi URL:

* `/Home/Index/17`
* `/Home/Index`
* `/Home`
* `/`

Il percorso URL `/` Usa i controller e l'azione predefiniti del modello `Home` di route `Index` . Il percorso URL `/Home` Usa l'azione predefinita del modello di route `Index` .

Il metodo pratico <xref:Microsoft.AspNetCore.Builder.ControllerEndpointRouteBuilderExtensions.MapDefaultControllerRoute%2A>:

```csharp
endpoints.MapDefaultControllerRoute();
```

Sostituisce

```csharp
endpoints.MapControllerRoute("default", "{controller=Home}/{action=Index}/{id?}");
```

> [!IMPORTANT]
> Il routing viene configurato usando <xref:Microsoft.AspNetCore.Builder.EndpointRoutingApplicationBuilderExtensions.UseRouting%2A> il <xref:Microsoft.AspNetCore.Builder.EndpointRoutingApplicationBuilderExtensions.UseEndpoints%2A> middleware e. Per usare i controller:
>
> * Chiamare <xref:Microsoft.AspNetCore.Builder.ControllerEndpointRouteBuilderExtensions.MapControllers%2A> all'interno per eseguire il mapping tra i `UseEndpoints` controller [indirizzati degli attributi](#ar) .
> * Chiamare <xref:Microsoft.AspNetCore.Builder.ControllerEndpointRouteBuilderExtensions.MapControllerRoute%2A> o <xref:Microsoft.AspNetCore.Builder.ControllerEndpointRouteBuilderExtensions.MapAreaControllerRoute%2A> per eseguire il mapping tra controller [indirizzati convenzionalmente](#cr) e controller [indirizzati degli attributi](#ar) .

<a name="routing-conventional-ref-label"></a>
<a name="crd"></a>

## <a name="conventional-routing"></a>Routing convenzionale

Il routing convenzionale viene usato con i controller e le visualizzazioni. La route predefinita (`default`):

[!code-csharp[](routing/samples/3.x/main/StartupDefaultMVC.cs?name=snippet2)]

Il precedente è un esempio di *Route convenzionale*. Viene chiamato *routing convenzionale* perché stabilisce una *convenzione* per i percorsi URL:

* Il primo segmento di percorso, `{controller=Home}` , esegue il mapping al nome del controller.
* Il secondo segmento, `{action=Index}` , esegue il mapping al nome dell' [azione](#action) .
* Il terzo segmento `{id?}` viene usato per un oggetto facoltativo `id` . `?`In lo `{id?}` rende facoltativo. `id` viene utilizzato per eseguire il mapping a un'entità del modello.

Utilizzando questa `default` Route, il percorso dell'URL:

* `/Products/List` esegue il mapping all' `ProductsController.List` azione.
* `/Blog/Article/17` esegue il mapping a `BlogController.Article` e in genere il modello associa il `id` parametro a 17.

Questo mapping:

* Si basa **solo** sui nomi del controller e dell' [azione](#action) .
* Non è basato su spazi dei nomi, percorsi di file di origine o parametri del metodo.

L'uso del routing convenzionale con la route predefinita consente di creare l'app senza dover ottenere un nuovo modello di URL per ogni azione. Per un'app con azioni di stile [CRUD](https://wikipedia.org/wiki/Create,_read,_update_and_delete) , con coerenza per gli URL tra i controller:

* Consente di semplificare il codice.
* Rende l'interfaccia utente più prevedibile.

> [!WARNING]
> `id`Nel codice precedente viene definito come facoltativo dal modello di route. Le azioni possono essere eseguite senza l'ID facoltativo fornito come parte dell'URL. In genere, quando `id` viene omesso dall'URL:
>
> * `id` viene impostato su `0` dall'associazione di modelli.
> * Non è stata trovata alcuna entità nel database corrispondente `id == 0` .
>
> Il [routing degli attributi](#ar) offre un controllo con granularità fine per rendere l'ID necessario per alcune azioni e non per altri. Per convenzione, la documentazione include parametri facoltativi come `id` quando è probabile che vengano visualizzati nell'utilizzo corretto.

La maggior parte delle app dovrebbe scegliere uno schema di routing semplice e descrittivo in modo che gli URL siano leggibili e significativi. La route convenzionale predefinita `{controller=Home}/{action=Index}/{id?}`:

* Supporta uno schema di routing semplice e descrittivo.
* È un punto iniziale utile per le app basate su interfaccia utente.
* È l'unico modello di route necessario per molte app dell'interfaccia utente Web. Per le app di interfaccia utente Web di dimensioni maggiori, un'altra route che usa le [aree](#areas) è spesso sufficiente.

<xref:Microsoft.AspNetCore.Builder.ControllerEndpointRouteBuilderExtensions.MapControllerRoute%2A> e <xref:Microsoft.AspNetCore.Builder.MvcAreaRouteBuilderExtensions.MapAreaRoute%2A> :

* Assegnare automaticamente un valore di **ordine** ai rispettivi endpoint in base all'ordine in cui vengono richiamati.

Routing degli endpoint in ASP.NET Core 3,0 e versioni successive:

* Non ha un concetto di route.
* Non fornisce garanzie di ordinamento per l'esecuzione dell'estensibilità, tutti gli endpoint vengono elaborati contemporaneamente.

Abilitare la [registrazione](xref:fundamentals/logging/index) per verificare in che modo le implementazioni del routing predefinite, ad esempio <xref:Microsoft.AspNetCore.Routing.Route>, corrispondono alle richieste.

Il [routing degli attributi](#ar) è illustrato più avanti in questo documento.

<a name="mr"></a>

### <a name="multiple-conventional-routes"></a>Più route convenzionali

È possibile aggiungere più [Route convenzionali](#cr) all'interno `UseEndpoints` di aggiungendo altre chiamate a <xref:Microsoft.AspNetCore.Builder.ControllerEndpointRouteBuilderExtensions.MapControllerRoute%2A> e <xref:Microsoft.AspNetCore.Builder.ControllerEndpointRouteBuilderExtensions.MapAreaControllerRoute%2A> . In questo modo è possibile definire più convenzioni o aggiungere route convenzionali dedicate a un' [azione](#action)specifica, ad esempio:

[!code-csharp[](routing/samples/3.x/main/Startup.cs?name=snippet_1)]

<a name="dcr"></a>

La `blog` Route nel codice precedente è una **Route convenzionale dedicata**. Viene chiamato Route convenzionale dedicata perché:

* Usa il [routing convenzionale](#cr).
* È dedicata a un' [azione](#action)specifica.

Poiché `controller` e `action` non vengono visualizzati nel modello di route `"blog/{*article}"` come parametri:

* Possono avere solo i valori predefiniti `{ controller = "Blog", action = "Article" }` .
* Questa route è sempre associata all'azione `BlogController.Article` .

`/Blog`, `/Blog/Article` e `/Blog/{any-string}` sono gli unici percorsi URL che corrispondono alla route del Blog.

L'esempio precedente:

* `blog` la route ha una priorità più elevata per le corrispondenze rispetto alla `default` Route perché viene aggiunta per prima.
* È un esempio di routing di tipo [Slug](https://developer.mozilla.org/docs/Glossary/Slug) in cui è tipico avere un nome di articolo come parte dell'URL.

> [!WARNING]
> In ASP.NET Core 3,0 e versioni successive, il routing non:
> * Definire un concetto denominato *Route*. `UseRouting` aggiunge la corrispondenza della route alla pipeline middleware. Il `UseRouting` middleware esamina il set di endpoint definiti nell'app e seleziona la corrispondenza migliore per l'endpoint in base alla richiesta.
> * Fornire garanzie sull'ordine di esecuzione dell'estensibilità <xref:Microsoft.AspNetCore.Routing.IRouteConstraint> , ad esempio o <xref:Microsoft.AspNetCore.Mvc.ActionConstraints.IActionConstraint> .
>
>Vedere [routing](xref:fundamentals/routing) per il materiale di riferimento sul routing.

<a name="cro"></a>

### <a name="conventional-routing-order"></a>Ordine di routing convenzionale

Il routing convenzionale corrisponde solo a una combinazione di azione e controller definiti dall'app. Questa operazione è progettata per semplificare i casi in cui le route convenzionali si sovrappongono.
Se si aggiungono route usando <xref:Microsoft.AspNetCore.Builder.ControllerEndpointRouteBuilderExtensions.MapControllerRoute%2A> , <xref:Microsoft.AspNetCore.Builder.ControllerEndpointRouteBuilderExtensions.MapDefaultControllerRoute%2A> e si <xref:Microsoft.AspNetCore.Builder.ControllerEndpointRouteBuilderExtensions.MapAreaControllerRoute%2A> assegna automaticamente un valore di ordine ai rispettivi endpoint in base all'ordine in cui vengono richiamati. Corrisponde a una route visualizzata in precedenza con una priorità più alta. Il routing convenzionale dipende dall'ordinamento. In generale, le route con aree devono essere posizionate in precedenza perché sono più specifiche delle route senza un'area. Le route [convenzionali dedicate](#dcr) con parametri di route catch-all come `{*article}` possono rendere troppo [greedy](xref:fundamentals/routing#greedy)una route, vale a dire che corrisponde agli URL per i quali si intende trovare una corrispondenza con altre route. Inserire le route greedy in un secondo momento nella tabella di route per evitare corrispondenze greedy.

[!INCLUDE[](~/includes/catchall.md)]

<a name="best"></a>

### <a name="resolving-ambiguous-actions"></a>Risoluzione di azioni ambigue

Quando due endpoint corrispondono tramite il routing, il routing deve eseguire una delle operazioni seguenti:

* Scegliere il candidato migliore.
* Generazione di un'eccezione.

Ad esempio:

[!code-csharp[](routing/samples/3.x/main/Controllers/ProductsController.cs?name=snippet9)]

Il controller precedente definisce due azioni che corrispondono a:

* Percorso URL `/Products33/Edit/17`
* Dati di route `{ controller = Products33, action = Edit, id = 17 }` .

Si tratta di un modello tipico per i controller MVC:

* `Edit(int)` Visualizza un modulo per la modifica di un prodotto.
* `Edit(int, Product)` elabora il form inviato.

Per risolvere la route corretta:

* `Edit(int, Product)` viene selezionato quando la richiesta è un HTTP `POST` .
* `Edit(int)` viene selezionato quando il [verbo http](#verb) è qualsiasi altro elemento. `Edit(int)` viene in genere chiamato tramite `GET` .

L'oggetto <xref:Microsoft.AspNetCore.Mvc.HttpPostAttribute> , `[HttpPost]` , viene fornito al routing in modo che sia possibile scegliere in base al metodo HTTP della richiesta. `HttpPostAttribute`Consente `Edit(int, Product)` di ottenere una corrispondenza migliore rispetto a `Edit(int)` .

È importante comprendere il ruolo degli attributi, ad esempio `HttpPostAttribute` . Attributi simili vengono definiti per altri [verbi HTTP](#verb). Nel [routing convenzionale](#cr), è comune per le azioni usare lo stesso nome di azione quando fanno parte di un flusso di lavoro di invio del modulo di visualizzazione. Vedere ad esempio [esaminare i due metodi di azione di modifica](xref:tutorials/first-mvc-app/controller-methods-views#get-post).

Se il routing non è in grado di scegliere un candidato migliore, <xref:System.Reflection.AmbiguousMatchException> viene generata un'eccezione, che elenca più endpoint corrispondenti.

<a name="routing-route-name-ref-label"></a>

### <a name="conventional-route-names"></a>Nomi di route convenzionali

Le stringhe  `"blog"` e `"default"` negli esempi seguenti sono nomi di route convenzionali:

[!code-csharp[](routing/samples/3.x/main/Startup.cs?name=snippet_1)]

I nomi di route assegnano alla route un nome logico. La route denominata può essere usata per la generazione di URL. L'uso di una route denominata semplifica la creazione di URL quando l'ordinamento delle route può rendere complessa la generazione di URL. I nomi delle route devono essere univoci a livello di applicazione.

Nomi di route:

* Non hanno alcun effetto sulla corrispondenza degli URL o sulla gestione delle richieste.
* Vengono usati solo per la generazione di URL.

Il concetto di nome della route è rappresentato nel routing come [IEndpointNameMetadata](xref:Microsoft.AspNetCore.Routing.IEndpointNameMetadata). Il **nome della route** e il nome dell' **endpoint**:

* Sono intercambiabili.
* Quello usato nella documentazione e nel codice dipende dall'API descritta.

<a name="attribute-routing-ref-label"></a>
<a name="ar"></a>

## <a name="attribute-routing-for-rest-apis"></a>Routing degli attributi per le API REST

Le API REST devono usare il routing degli attributi per modellare la funzionalità dell'app come un set di risorse in cui le operazioni sono rappresentate da [verbi HTTP](#verb).

Il routing con attributi usa un set di attributi per eseguire il mapping delle azioni direttamente ai modelli di route. Il `StartUp.Configure` codice seguente è tipico per un'API REST e viene usato nell'esempio seguente:

[!code-csharp[](routing/samples/3.x/main/StartupAPI.cs?name=snippet)]

Nel codice precedente, <xref:Microsoft.AspNetCore.Builder.ControllerEndpointRouteBuilderExtensions.MapControllers%2A> viene chiamato all'interno `UseEndpoints` di per eseguire il mapping dei controller indirizzati degli attributi.

Nell'esempio seguente:

* `Configure`Viene utilizzato il metodo precedente.
* `HomeController` corrisponde a un set di URL simile a quello che la route convenzionale predefinita `{controller=Home}/{action=Index}/{id?}` corrisponde.

[!code-csharp[](routing/samples/3.x/main/Controllers/HomeController.cs?name=snippet2)]

L' `HomeController.Index` azione viene eseguita per uno dei percorsi URL `/` ,, `/Home` `/Home/Index` o `/Home/Index/3` .

In questo esempio viene evidenziata una differenza di programmazione principale tra routing degli attributi e [routing convenzionale](#cr). Il routing degli attributi richiede un maggior numero di input per specificare una route. La route predefinita convenzionale gestisce le route in maniera più concisa. Tuttavia, il routing degli attributi consente e richiede un controllo preciso dei modelli di route applicabili a ciascuna [azione](#action).

Con il routing degli attributi, i nomi del controller e dell'azione non sono parte dell'azione corrispondente, a meno che non venga utilizzata la [sostituzione del token](#routing-token-replacement-templates-ref-label) . L'esempio seguente corrisponde agli stessi URL dell'esempio precedente:

[!code-csharp[](routing/samples/3.x/main/Controllers/MyDemoController.cs?name=snippet)]

Il codice seguente usa la sostituzione dei token per `action` e `controller` :

[!code-csharp[](routing/samples/3.x/main/Controllers/HomeController.cs?name=snippet22)]

Il codice seguente si applica `[Route("[controller]/[action]")]` al controller:

[!code-csharp[](routing/samples/3.x/main/Controllers/HomeController.cs?name=snippet24)]

Nel codice precedente, i `Index` modelli di metodo devono anteporre `/` o `~/` ai modelli di route. I modelli di route applicati a un'azione che iniziano con `/` o `~/` non vengono combinati con i modelli di route applicati al controller.

Vedere [precedenza del modello di route](xref:fundamentals/routing#rtp) per informazioni sulla selezione del modello di route.

## <a name="reserved-routing-names"></a>Nomi riservati di routing

Quando si usano i controller o le pagine, le parole chiave seguenti sono nomi di parametro di route riservati Razor :

* `action`
* `area`
* `controller`
* `handler`
* `page`

`page`L'uso di come parametro di route con routing degli attributi è un errore comune. Questa operazione comporta un comportamento incoerente e confuso con la generazione di URL.

[!code-csharp[](routing/samples/3.x/main/Controllers/MyDemo2Controller.cs?name=snippet)]

I nomi di parametro speciali vengono usati dalla generazione dell'URL per determinare se un'operazione di generazione URL fa riferimento a una Razor pagina o a un controller.

<a name="verb"></a>

## <a name="http-verb-templates"></a>Modelli di verbi HTTP

ASP.NET Core dispone dei seguenti modelli di verbi HTTP:

* [[HttpGet]](xref:Microsoft.AspNetCore.Mvc.HttpGetAttribute)
* [HttpPost](xref:Microsoft.AspNetCore.Mvc.HttpPostAttribute)
* [HttpPut](xref:Microsoft.AspNetCore.Mvc.HttpPutAttribute)
* [HttpDelete](xref:Microsoft.AspNetCore.Mvc.HttpDeleteAttribute)
* [[HttpHead]](xref:Microsoft.AspNetCore.Mvc.HttpHeadAttribute)
* [[HttpPatch]](xref:Microsoft.AspNetCore.Mvc.HttpPatchAttribute)

<a name="rt"></a>

### <a name="route-templates"></a>Modelli di route

ASP.NET Core include i modelli di route seguenti:

* Tutti i [modelli di verbi HTTP](#verb) sono modelli di route.
* [Route](xref:Microsoft.AspNetCore.Mvc.RouteAttribute)

<a name="arx"></a>

### <a name="attribute-routing-with-http-verb-attributes"></a>Routing degli attributi con attributi di verbi http

Si consideri il controller seguente:

[!code-csharp[](routing/samples/3.x/main/Controllers/Test2Controller.cs?name=snippet)]

Nel codice precedente:

* Ogni azione contiene l' `[HttpGet]` attributo, che vincola la corrispondenza solo alle richieste HTTP Get.
* L' `GetProduct` azione include il `"{id}"` modello, pertanto `id` viene aggiunto al `"api/[controller]"` modello sul controller. Il modello di metodi è `"api/[controller]/"{id}""` . Pertanto questa azione corrisponde solo alle richieste Get per il form,, e `/api/test2/xyz` `/api/test2/123` `/api/test2/{any string}` così via.
  [!code-csharp[](routing/samples/3.x/main/Controllers/Test2Controller.cs?name=snippet2)]
* L' `GetIntProduct` azione contiene il `"int/{id:int}")` modello. La `:int` parte del modello vincola i `id` valori di route alle stringhe che possono essere convertite in un numero intero. Una richiesta GET a `/api/test2/int/abc` :
  * Non corrisponde a questa azione.
  * Restituisce un errore [404 non trovato](https://developer.mozilla.org/docs/Web/HTTP/Status/404) .
    [!code-csharp[](routing/samples/3.x/main/Controllers/Test2Controller.cs?name=snippet3)]
* L' `GetInt2Product` azione contiene `{id}` nel modello, ma non vincola i `id` valori che possono essere convertiti in un numero intero. Una richiesta GET a `/api/test2/int2/abc` :
  * Corrisponde a questa route.
  * L'associazione di modelli non riesce a eseguire `abc` la conversione in un numero intero. Il `id` parametro del metodo è Integer.
  * Restituisce una [richiesta non valida 400](https://developer.mozilla.org/docs/Web/HTTP/Status/400) perché l'associazione di modelli non è riuscita a eseguire la conversione `abc` in un numero intero.
      [!code-csharp[](routing/samples/3.x/main/Controllers/Test2Controller.cs?name=snippet4)]

Il routing degli attributi può utilizzare <xref:Microsoft.AspNetCore.Mvc.Routing.HttpMethodAttribute> attributi come <xref:Microsoft.AspNetCore.Mvc.HttpPostAttribute> , <xref:Microsoft.AspNetCore.Mvc.HttpPutAttribute> e <xref:Microsoft.AspNetCore.Mvc.HttpDeleteAttribute> . Tutti gli attributi del [verbo http](#verb) accettano un modello di route. Nell'esempio seguente vengono illustrate due azioni che corrispondono allo stesso modello di route:

[!code-csharp[](routing/samples/3.x/main/Controllers/MyProductsController.cs?name=snippet1)]

Utilizzando il percorso URL `/products3` :

* L' `MyProductsController.ListProducts` azione viene eseguita quando il [verbo http](#verb) è `GET` .
* L' `MyProductsController.CreateProduct` azione viene eseguita quando il [verbo http](#verb) è `POST` .

Quando si compila un'API REST, è raro che sia necessario usare `[Route(...)]` su un metodo di azione perché l'azione accetta tutti i metodi HTTP. È preferibile usare l' [attributo verbo http](#verb) più specifico per essere precisi sul supporto dell'API. I client delle API REST devono sapere quali percorsi e verbi HTTP devono essere mappati a operazioni logiche specifiche.

Le API REST devono usare il routing degli attributi per modellare la funzionalità dell'app come un set di risorse in cui le operazioni sono rappresentate da verbi HTTP. Ciò significa che molte operazioni, ad esempio GET e POST nella stessa risorsa logica, utilizzano lo stesso URL. Il routing degli attributi offre un livello di controllo necessario per progettare con attenzione il layout dell'endpoint pubblico di un'API.

Poiché una route con attributi si applica a un'azione specifica, è facile fare in modo che i parametri siano richiesti come parte della definizione del modello di route. Nell'esempio seguente `id` è necessario come parte del percorso URL:

[!code-csharp[](routing/samples/3.x/main/Controllers/ProductsApiController.cs?name=snippet2)]

`Products2ApiController.GetProduct(int)`Azione:

* Viene eseguito con un percorso URL come `/products2/3`
* Non viene eseguito con il percorso URL `/products2` .

L'attributo [[consums]](<xref:Microsoft.AspNetCore.Mvc.ConsumesAttribute>) consente a un'azione di limitare i tipi di contenuto della richiesta supportati. Per altre informazioni, vedere [definire i tipi di contenuto della richiesta supportati con l'attributo consums](xref:web-api/index#consumes).

 Vedere [Routing](xref:fundamentals/routing) per una descrizione completa dei modelli di route e delle opzioni correlate.

Per altre informazioni su `[ApiController]` , vedere [attributo ApiController](xref:web-api/index##apicontroller-attribute).

## <a name="route-name"></a>Nome route

Nell’esempio di codice riportato di seguito viene definito un nome di route di `Products_List`:

[!code-csharp[](routing/samples/3.x/main/Controllers/ProductsApiController.cs?name=snippet2)]

I nomi di route possono essere usati per generare un URL in base a un percorso specifico. Nomi di route:

* Non hanno alcun effetto sul comportamento di corrispondenza dell'URL del routing.
* Vengono usati solo per la generazione di URL.

I nomi delle route devono essere univoci a livello di applicazione.

Confrontare il codice precedente con la route predefinita convenzionale, che definisce il `id` parametro come facoltativo ( `{id?}` ). La possibilità di specificare con precisione le API offre vantaggi, ad esempio consentendo `/products` e `/products/5` di essere inviati a diverse azioni.

<a name="routing-combining-ref-label"></a>

## <a name="combining-attribute-routes"></a>Combinazione di route per attributi

Per rendere il routing con attributi meno ripetitivo, gli attributi di route del controller vengono combinati con gli attributi di route delle singole azioni. I modelli di route definiti per il controller vengono anteposti ai modelli di route delle azioni. Inserendo un attributo di route nel controller, **tutte** le azioni presenti nel controller useranno il routing con attributi.

[!code-csharp[](routing/samples/3.x/main/Controllers/ProductsApiController.cs?name=snippet)]

Nell'esempio precedente:

* Il percorso URL `/products` può corrispondere `ProductsApi.ListProducts`
* Il percorso URL `/products/5` può corrispondere `ProductsApi.GetProduct(int)` .

Entrambe le azioni corrispondono solo a HTTP `GET` perché sono contrassegnate con l' `[HttpGet]` attributo.

I modelli di route applicati a un'azione che iniziano con `/` o `~/` non vengono combinati con i modelli di route applicati al controller. Nell'esempio seguente viene individuata la corrispondenza di un set di percorsi URL simile a quello della route predefinita.

[!code-csharp[](routing/samples/3.x/main/Controllers/HomeController.cs?name=snippet)]

Nella tabella seguente vengono illustrati gli `[Route]` attributi del codice precedente:

| Attributo               | Combina con `[Route("Home")]` | Definisce il modello di route |
| ----------------- | ------------ | --------- |
| `[Route("")]` | Sì | `"Home"` |
| `[Route("Index")]` | Sì | `"Home/Index"` |
| `[Route("/")]` | **No** | `""` |
| `[Route("About")]` | Sì | `"Home/About"` |

<a name="routing-ordering-ref-label"></a>
<a name="oar"></a>

### <a name="attribute-route-order"></a>Ordine Route attributi

Il routing compila un albero e corrisponde a tutti gli endpoint contemporaneamente:

* Le voci della route si comportano come se fossero posizionate in un ordinamento ideale.
* Le route più specifiche hanno la possibilità di essere eseguite prima delle route più generali.

Ad esempio, una route di attributo come `blog/search/{topic}` è più specifica di una route di attributo come `blog/{*article}` . `blog/search/{topic}`Per impostazione predefinita, la route ha una priorità più elevata perché è più specifica. Utilizzando il [routing convenzionale](#cr), lo sviluppatore è responsabile dell'inserimento delle route nell'ordine desiderato.

Le route degli attributi possono configurare un ordine utilizzando la <xref:Microsoft.AspNetCore.Mvc.RouteAttribute.Order> Proprietà. Tutti gli [attributi di route](xref:Microsoft.AspNetCore.Mvc.RouteAttribute) forniti dal Framework includono `Order` . Le route vengono elaborate in base a un ordinamento crescente della proprietà `Order`. L'ordine predefinito è `0`. L'impostazione di una route tramite `Order = -1` viene eseguita prima delle route che non impostano un ordine. L'impostazione di una route tramite `Order = 1` viene eseguita dopo l'ordinamento predefinito della route.

**Evitare** la dipendenza da `Order` . Se lo spazio URL di un'app richiede che i valori di ordine espliciti vengano indirizzati correttamente, è probabile che anche i client siano confusi. In generale, il routing degli attributi seleziona la route corretta con l'URL corrispondente. Se l'ordine predefinito usato per la generazione di URL non funziona, l'uso di un nome di route come override è in genere più semplice dell'applicazione della `Order` Proprietà.

Considerare i seguenti due controller che definiscono entrambi la corrispondenza della route `/home` :

[!code-csharp[](routing/samples/3.x/main/Controllers/HomeController.cs?name=snippet2)]

[!code-csharp[](routing/samples/3.x/main/Controllers/MyDemoController.cs?name=snippet)]

La richiesta `/home` con il codice precedente genera un'eccezione simile alla seguente:

```text
AmbiguousMatchException: The request matched multiple endpoints. Matches:

 WebMvcRouting.Controllers.HomeController.Index
 WebMvcRouting.Controllers.MyDemoController.MyIndex
```

`Order`L'aggiunta a uno degli attributi di route risolve l'ambiguità:

[!code-csharp[](routing/samples/3.x/main/Controllers/MyDemo3Controller.cs?name=snippet3& highlight=2)]

Con il codice precedente, `/home` esegue l' `HomeController.Index` endpoint. Per ottenere l'oggetto `MyDemoController.MyIndex` , richiedere `/home/MyIndex` . **Nota**:

* Il codice precedente è un esempio o una progettazione di routing insufficiente. È stato usato per illustrare la `Order` Proprietà.
* La `Order` Proprietà risolve solo l'ambiguità e non è possibile trovare una corrispondenza per il modello. È preferibile rimuovere il `[Route("Home")]` modello.

Vedere la pagina relativa alle [ Razor convenzioni di route e app: ordine](xref:razor-pages/razor-pages-conventions#route-order) di route per informazioni sull'ordine di route con Razor pagine.

In alcuni casi, viene restituito un errore HTTP 500 con route ambigue. Usare la [registrazione](xref:fundamentals/logging/index) per verificare quali endpoint hanno causato `AmbiguousMatchException` .

<a name="routing-token-replacement-templates-ref-label"></a>

## <a name="token-replacement-in-route-templates-controller-action-area"></a>Sostituzione di token nei modelli di route [controller], [azione], [area]

Per praticità, le route degli attributi supportano la *sostituzione dei token* mediante l'inclusione di un token tra parentesi quadre ( `[` , `]` ). I token `[action]` , `[area]` e `[controller]` vengono sostituiti con i valori del nome dell'azione, del nome dell'area e del nome del controller dall'azione in cui è definita la route:

[!code-csharp[](routing/samples/3.x/main/Controllers/ProductsController.cs?name=snippet)]

Nel codice precedente:

  [!code-csharp[](routing/samples/3.x/main/Controllers/ProductsController.cs?name=snippet10)]

  * Corrisponde `/Products0/List`

  [!code-csharp[](routing/samples/3.x/main/Controllers/ProductsController.cs?name=snippet11)]

  * Corrisponde `/Products0/Edit/{id}`

La sostituzione dei token avviene come ultimo passaggio della creazione delle route con attributi. L'esempio precedente si comporta come il codice seguente:

[!code-csharp[](routing/samples/3.x/main/Controllers/ProductsController.cs?name=snippet20)]

[!INCLUDE[](~/includes/MTcomments.md)]

Le route con attributi possono anche essere combinate con l'ereditarietà. Questa funzionalità è potente in combinazione con la sostituzione dei token. La sostituzione dei token si applica anche ai nomi di route definiti dalle route con attributi.
`[Route("[controller]/[action]", Name="[controller]_[action]")]`genera un nome di route univoco per ogni azione:

[!code-csharp[](routing/samples/3.x/main/Controllers/ProductsController.cs?name=snippet5)]

La sostituzione dei token si applica anche ai nomi di route definiti dalle route con attributi.
`[Route("[controller]/[action]", Name="[controller]_[action]")]`
 genera un nome di route univoco per ogni azione.

Per verificare la corrispondenza del delimitatore letterale della sostituzione di token `[` o `]`, eseguirne l'escape ripetendo il carattere (`[[` o `]]`).

<a name="routing-token-replacement-transformers-ref-label"></a>

### <a name="use-a-parameter-transformer-to-customize-token-replacement"></a>Usare un trasformatore di parametri per personalizzare la sostituzione dei token

La sostituzione dei token può essere personalizzata usando un trasformatore di parametri. Un trasformatore di parametri implementa <xref:Microsoft.AspNetCore.Routing.IOutboundParameterTransformer> e trasforma il valore dei parametri. Un trasformatore di parametro personalizzato, ad esempio, `SlugifyParameterTransformer` modifica il `SubscriptionManagement` valore di route in `subscription-management` :

[!code-csharp[](routing/samples/3.x/main/StartupSlugifyParamTransformer.cs?name=snippet2)]

<xref:Microsoft.AspNetCore.Mvc.ApplicationModels.RouteTokenTransformerConvention> è una convenzione del modello di applicazione che:

* Applica un trasformatore di parametri a tutte le route di attributi in un'applicazione.
* Personalizza i valori dei token delle route di attributi quando vengono sostituiti.

[!code-csharp[](routing/samples/3.x/main/Controllers/SubscriptionManagementController.cs?name=snippet)]

Il `ListAll` metodo precedente corrisponde a `/subscription-management/list-all` .

La convenzione `RouteTokenTransformerConvention` è registrata come un'opzione in `ConfigureServices`.

[!code-csharp[](routing/samples/3.x/main/StartupSlugifyParamTransformer.cs?name=snippet)]

Per la definizione di Slug, vedere la [documentazione Web MDN in slug](https://developer.mozilla.org/docs/Glossary/Slug) .

[!INCLUDE[](~/includes/regex.md)]
<a name="routing-multiple-routes-ref-label"></a>

### <a name="multiple-attribute-routes"></a>Route di più attributi

Il routing con attributi supporta la definizione di più route che raggiungono la stessa azione. L'uso più comune è simulare il comportamento della route convenzionale predefinita come illustrato nell'esempio seguente:

[!code-csharp[](routing/samples/3.x/main/Controllers/ProductsController.cs?name=snippet6x)]

L'inserimento di più attributi di route sul controller significa che ognuno di essi si combina con ognuno degli attributi di route nei metodi di azione:

[!code-csharp[](routing/samples/3.x/main/Controllers/ProductsController.cs?name=snippet6)]

Tutti i vincoli di route [http verb](#verb) implementano `IActionConstraint` .

Quando più attributi di route che implementano <xref:Microsoft.AspNetCore.Mvc.ActionConstraints.IActionConstraint> sono posizionati in un'azione:

* Ogni vincolo di azione viene combinato con il modello di route applicato al controller.

[!code-csharp[](routing/samples/3.x/main/Controllers/ProductsController.cs?name=snippet7)]

L'uso di più route sulle azioni potrebbe sembrare utile e potente, ma è preferibile tenere lo spazio URL dell'app di base e ben definito. Usare più route sulle azioni **solo** se necessario, ad esempio per supportare i client esistenti.

<a name="routing-attr-options"></a>

### <a name="specifying-attribute-route-optional-parameters-default-values-and-constraints"></a>Definizione di parametri facoltativi, valori predefiniti e vincoli della route con attributi

Le route con attributi supportano la stessa sintassi inline delle route convenzionali per specificare i parametri facoltativi, i valori predefiniti e i vincoli.

[!code-csharp[](routing/samples/3.x/main/Controllers/ProductsController.cs?name=snippet8&highlight=3)]

Nel codice precedente, `[HttpPost("product14/{id:int}")]` applica un vincolo di route. L' `Products14Controller.ShowProduct` azione corrisponde solo ai percorsi URL, ad esempio `/product14/3` . La parte del modello `{id:int}` di route vincola tale segmento ai soli numeri interi.

Vedere [Riferimento per i modelli di route](xref:fundamentals/routing#route-template-reference) per una descrizione dettagliata della sintassi del modello di route.

<a name="routing-cust-rt-attr-irt-ref-label"></a>

### <a name="custom-route-attributes-using-iroutetemplateprovider"></a>Attributi di Route personalizzati con IRouteTemplateProvider

Tutti gli [attributi di route](#rt) implementano <xref:Microsoft.AspNetCore.Mvc.Routing.IRouteTemplateProvider> . Il runtime di ASP.NET Core:

* Cerca gli attributi nelle classi controller e nei metodi di azione all'avvio dell'app.
* USA gli attributi che implementano `IRouteTemplateProvider` per compilare il set iniziale di route.

Implementare `IRouteTemplateProvider` per definire gli attributi di Route personalizzati. Ogni `IRouteTemplateProvider` consente di definire una singola route con un modello di route, un ordinamento e un nome personalizzati:

[!code-csharp[](routing/samples/3.x/main/Controllers/MyTestApiController.cs?name=snippet&highlight=1-10)]

Il `Get` metodo precedente restituisce `Order = 2, Template = api/MyTestApi` .

<a name="routing-app-model-ref-label"></a>

### <a name="use-application-model-to-customize-attribute-routes"></a>Usare il modello di applicazione per personalizzare le route degli attributi

Modello di applicazione:

* È un modello a oggetti creato all'avvio.
* Contiene tutti i metadati usati da ASP.NET Core per indirizzare ed eseguire le azioni in un'app.

Il modello di applicazione include tutti i dati raccolti dagli attributi di route. I dati degli attributi della route vengono forniti dall' `IRouteTemplateProvider` implementazione di. Convenzioni

* È possibile scrivere per modificare il modello di applicazione per personalizzare il comportamento del routing.
* Vengono letti all'avvio dell'app.

Questa sezione illustra un esempio di base di personalizzazione del routing con il modello di applicazione. Il codice seguente rende le route approssimativamente allineate con la struttura di cartelle del progetto.

[!code-csharp[](routing/samples/3.x/nsrc/NamespaceRoutingConvention.cs?name=snippet)]

Il codice seguente impedisce `namespace` che la Convenzione venga applicata ai controller indirizzati con l'attributo:

[!code-csharp[](routing/samples/3.x/nsrc/NamespaceRoutingConvention.cs?name=snippet2)]

Ad esempio, il controller seguente non usa `NamespaceRoutingConvention` :

[!code-csharp[](routing/samples/3.x/nsrc/Controllers/ManagersController.cs?name=snippet&highlight=1)]

Il metodo `NamespaceRoutingConvention.Apply`:

* Non esegue alcuna operazione se il controller è instradato all'attributo.
* Imposta il modello di controller basato su `namespace` , con la base `namespace` rimossa.

Il `NamespaceRoutingConvention` può essere applicato in `Startup.ConfigureServices` :

[!code-csharp[](routing/samples/3.x/nsrc/Startup.cs?name=snippet&highlight=1,14-18)]

Si consideri, ad esempio, il controller seguente:

[!code-csharp[](routing/samples/3.x/nsrc/Controllers/UsersController.cs)]

Nel codice precedente:

* La base `namespace` è `My.Application` .
* Il nome completo del controller precedente è `My.Application.Admin.Controllers.UsersController` .
* `NamespaceRoutingConvention`Imposta il modello dei controller su `Admin/Controllers/Users/[action]/{id?` .

Il `NamespaceRoutingConvention` può essere applicato anche come attributo in un controller:

[!code-csharp[](routing/samples/3.x/nsrc/Controllers/TestController.cs?name=snippet&highlight=1)]

<a name="routing-mixed-ref-label"></a>

## <a name="mixed-routing-attribute-routing-vs-conventional-routing"></a>Routing misto: routing con attributi e routing convenzionale

ASP.NET Core app possono combinare l'uso del routing convenzionale e del routing degli attributi. In genere le route convenzionali vengono usate per i controller che gestiscono le pagine HTML per i browser e le route con attributi per i controller che gestiscono le API REST.

Le azioni vengono indirizzate in modo convenzionale o con attributi. Se una route viene inserita nel controller o nell'azione, viene indirizzata con attributi. Le azioni che definiscono le route con attributi non possono essere raggiunte usando le route convenzionali e viceversa. **Qualsiasi** attributo di route nel controller esegue il routing di **tutte** le azioni nell'attributo controller.

Il routing degli attributi e il routing convenzionale utilizzano lo stesso motore di routing.

<a name="routing-url-gen-ref-label"></a>
<a name="ambient"></a>

## <a name="url-generation-and-ambient-values"></a>Generazione di URL e valori di ambiente

Le app possono usare le funzionalità di generazione URL di routing per generare collegamenti URL alle azioni. La generazione di URL Elimina gli URL hardcoded, rendendo il codice più affidabile e gestibile. Questa sezione è incentrata sulle funzionalità di generazione degli URL fornite da MVC e copre solo le nozioni di base sul funzionamento della generazione di URL. Vedere [Routing](xref:fundamentals/routing) per una descrizione dettagliata della generazione di URL.

L' <xref:Microsoft.AspNetCore.Mvc.IUrlHelper> interfaccia è l'elemento sottostante dell'infrastruttura tra MVC e il routing per la generazione di URL. Un'istanza di `IUrlHelper` è disponibile tramite la `Url` Proprietà in controller, visualizzazioni e componenti di visualizzazione.

Nell'esempio seguente l' `IUrlHelper` interfaccia viene utilizzata tramite la `Controller.Url` proprietà per generare un URL a un'altra azione.

[!code-csharp[](routing/samples/3.x/main/Controllers/UrlGenerationController.cs?name=snippet_1)]

Se l'app usa la route convenzionale predefinita, il valore della `url` variabile è la stringa del percorso dell'URL `/UrlGeneration/Destination` . Questo percorso URL viene creato tramite il routing combinando:

* Valori della route dalla richiesta corrente, denominati **valori di ambiente**.
* I valori passati a `Url.Action` e sostituiscono tali valori nel modello di route:

``` text
ambient values: { controller = "UrlGeneration", action = "Source" }
values passed to Url.Action: { controller = "UrlGeneration", action = "Destination" }
route template: {controller}/{action}/{id?}

result: /UrlGeneration/Destination
```

Il valore di ogni parametro di route del modello di route viene sostituito attraverso la corrispondenza dei nomi con i valori e i valori di ambiente. Un parametro di route che non ha un valore può:

* Utilizzare un valore predefinito se ne è presente uno.
* Viene ignorato se è facoltativo. Ad esempio, `id` dal modello di route `{controller}/{action}/{id?}` .

La generazione di URL non riesce se un parametro di route obbligatorio non ha un valore corrispondente. Se la generazione di URL non riesce per una route, viene tentata la route successiva finché non vengono tentate tutte le route o viene trovata una corrispondenza.

Nell'esempio precedente di viene `Url.Action` presupposto il [routing convenzionale](#cr). La generazione di URL funziona in modo analogo al [routing degli attributi](#ar), sebbene i concetti siano diversi. Con routing convenzionale:

* I valori di route vengono usati per espandere un modello.
* I valori di route per `controller` e `action` vengono in genere visualizzati in tale modello. Questa operazione funziona perché gli URL corrispondenti al routing rispettano una convenzione.

Nell'esempio seguente viene usato il routing degli attributi:

[!code-csharp[](routing/samples/3.x/main/Controllers/UrlGenerationAttrController.cs?name=snippet_1)]

L' `Source` azione nel codice precedente genera `custom/url/to/destination` .

<xref:Microsoft.AspNetCore.Routing.LinkGenerator> è stato aggiunto in ASP.NET Core 3,0 come alternativa a `IUrlHelper` . `LinkGenerator` offre funzionalità simili ma più flessibili. Ogni metodo in `IUrlHelper` dispone anche di una famiglia di metodi corrispondente `LinkGenerator` .

### <a name="generating-urls-by-action-name"></a>Generazione di URL in base al nome dell'azione

[URL. Action](xref:Microsoft.AspNetCore.Mvc.IUrlHelper.Action*), [LinkGenerator. GetPathByAction](xref:Microsoft.AspNetCore.Routing.ControllerLinkGeneratorExtensions.GetPathByAction*)e tutti gli overload correlati sono progettati per generare l'endpoint di destinazione specificando il nome del controller e il nome dell'azione.

Quando `Url.Action` si usa, i valori di route correnti per `controller` e `action` vengono forniti dal runtime:

* Il valore di `controller` e `action` fa parte dei [valori di ambiente](#ambient) e. Il metodo `Url.Action` Usa sempre i valori correnti di `action` e `controller` e genera un percorso URL che indirizza all'azione corrente.

Il routing tenta di usare i valori nei valori di ambiente per inserire le informazioni non fornite durante la generazione di un URL. Si consideri una route come `{a}/{b}/{c}/{d}` con i valori di ambiente `{ a = Alice, b = Bob, c = Carol, d = David }` :

* Il routing ha informazioni sufficienti per generare un URL senza valori aggiuntivi.
* Il routing ha informazioni sufficienti perché tutti i parametri di route hanno un valore.

Se il valore `{ d = Donovan }` viene aggiunto:

* Il valore `{ d = David }` viene ignorato.
* Il percorso dell'URL generato è `Alice/Bob/Carol/Donovan` .

**Avviso**: i percorsi URL sono gerarchici. Nell'esempio precedente, se il valore `{ c = Cheryl }` viene aggiunto:

* Entrambi i valori `{ c = Carol, d = David }` vengono ignorati.
* Non esiste più un valore per `d` e la generazione dell'URL ha esito negativo.
* `c` `d` Per generare un URL, è necessario specificare i valori desiderati di e.  

È possibile che si verifichi questo problema con la route predefinita `{controller}/{action}/{id?}` . Questo problema è raro in pratica perché `Url.Action` specifica sempre in modo esplicito un `controller` `action` valore e.

Diversi overload di [URL. Action](xref:Microsoft.AspNetCore.Mvc.IUrlHelper.Action*) accettano un oggetto dei valori di route per fornire valori per i parametri di route diversi da `controller` e `action` . L'oggetto valori di route viene spesso usato con `id` . Ad esempio: `Url.Action("Buy", "Products", new { id = 17 })`. Oggetto valori di route:

* Per convenzione è in genere un oggetto di tipo anonimo.
* Può essere un oggetto `IDictionary<>` o un oggetto [poco](https://wikipedia.org/wiki/Plain_old_CLR_object)).

I valori di route aggiuntivi che non corrispondono a parametri di route vengono inseriti nella stringa di query.

[!code-csharp[](routing/samples/3.x/main/Controllers/TestController.cs?name=snippet)]

Il codice precedente genera `/Products/Buy/17?color=red` .

Il codice seguente genera un URL assoluto:

[!code-csharp[](routing/samples/3.x/main/Controllers/TestController.cs?name=snippet2)]

Per creare un URL assoluto, usare uno dei seguenti elementi:

* Overload che accetta un oggetto `protocol` . Ad esempio, il codice precedente.
* [LinkGenerator. GetUriByAction](xref:Microsoft.AspNetCore.Routing.ControllerLinkGeneratorExtensions.GetUriByAction*), che genera URI assoluti per impostazione predefinita.

<a name="routing-gen-urls-route-ref-label"></a>

### <a name="generate-urls-by-route"></a>Genera URL per Route

Nel codice precedente è stata illustrata la generazione di un URL passando il nome del controller e dell'azione. `IUrlHelper` fornisce anche la famiglia di metodi [URL. RouteUrl](xref:Microsoft.AspNetCore.Mvc.IUrlHelper.RouteUrl*) . Questi metodi sono simili a [URL. Action](xref:Microsoft.AspNetCore.Mvc.IUrlHelper.Action*), ma non copiano i valori correnti di `action` e `controller` nei valori di route. L'utilizzo più comune di `Url.RouteUrl` :

* Specifica un nome di route per generare l'URL.
* In genere non specifica un nome di controller o di azione.

[!code-csharp[](routing/samples/3.x/main/Controllers/UrlGeneration2Controller.cs?name=snippet_1)]

Il Razor file seguente genera un collegamento HTML a `Destination_Route` :

[!code-cshtml[](routing/samples/3.x/main/Views/Shared/MyLink.cshtml)]

<a name="routing-gen-urls-html-ref-label"></a>

### <a name="generate-urls-in-html-and-no-locrazor"></a>Genera URL in HTML e Razor

<xref:Microsoft.AspNetCore.Mvc.Rendering.IHtmlHelper> fornisce i <xref:Microsoft.AspNetCore.Mvc.ViewFeatures.HtmlHelper> metodi [HTML. BeginForm](xref:Microsoft.AspNetCore.Mvc.Rendering.IHtmlHelper.BeginForm*) e [HTML. ActionLink](xref:Microsoft.AspNetCore.Mvc.Rendering.IHtmlHelper.ActionLink*) per generare `<form>` `<a>` rispettivamente gli elementi e. Questi metodi usano il metodo [URL. Action](xref:Microsoft.AspNetCore.Mvc.IUrlHelper.Action*) per generare un URL e accettano argomenti simili. Gli oggetti `Url.RouteUrl` complementi di `HtmlHelper` sono `Html.BeginRouteForm` e `Html.RouteLink` e hanno una funzionalità simile.

Gli helper tag generano gli URL attraverso l'helper tag `form` e l'helper tag `<a>`. Entrambi usano `IUrlHelper` per la propria implementazione. Per ulteriori informazioni, vedere [Helper tag nei moduli](xref:mvc/views/working-with-forms) .

All'interno delle visualizzazioni `IUrlHelper` è reso disponibile dalla proprietà `Url` per tutte le generazioni di URL ad hoc che non rientrano nelle situazioni descritte in precedenza.

<a name="routing-gen-urls-action-ref-label"></a>

### <a name="url-generation-in-action-results"></a>Generazione dell'URL nei risultati dell'azione

Negli esempi precedenti è stato illustrato l'utilizzo `IUrlHelper` di in un controller. L'utilizzo più comune in un controller consiste nel generare un URL come parte del risultato di un'azione.

Le classi di base <xref:Microsoft.AspNetCore.Mvc.ControllerBase> e <xref:Microsoft.AspNetCore.Mvc.Controller> offrono metodi pratici per i risultati delle azioni che fanno riferimento a un'altra azione. Un utilizzo tipico prevede il reindirizzamento dopo l'accettazione dell'input utente:

[!code-csharp[](routing/samples/3.x/main/Controllers/CustomerController.cs?name=snippet)]

I metodi factory dei risultati delle azioni come <xref:Microsoft.AspNetCore.Mvc.ControllerBase.RedirectToAction%2A> e <xref:Microsoft.AspNetCore.Mvc.ControllerBase.CreatedAtAction%2A> seguono un modello simile ai metodi in `IUrlHelper` .

<a name="routing-dedicated-ref-label"></a>

### <a name="special-case-for-dedicated-conventional-routes"></a>Caso speciale per le route convenzionali dedicate

Il [routing convenzionale](#cr) può usare un tipo speciale di definizione di route denominata [Route convenzionale dedicata](#dcr). Nell'esempio seguente la route denominata `blog` è una route convenzionale dedicata:

[!code-csharp[](routing/samples/3.x/main/Startup.cs?name=snippet_1)]

Usando le definizioni di route precedenti, `Url.Action("Index", "Home")` genera il percorso URL `/` usando la `default` Route, ma perché? Si potrebbe pensare che i valori di route `{ controller = Home, action = Index }` siano sufficienti per generare un URL usando `blog` e che il risultato sia `/blog?action=Index&controller=Home`.

Le [Route convenzionali dedicate](#dcr) si basano su un comportamento speciale dei valori predefiniti che non hanno un parametro di route corrispondente che impedisce che la route sia troppo [greedy](xref:fundamentals/routing#greedy) con la generazione di URL. In questo caso i valori predefiniti sono `{ controller = Blog, action = Article }` e né `controller` né `action` vengono visualizzati come parametri di route. Quando il routing esegue la generazione di URL, i valori specificati devono corrispondere ai valori predefiniti. La generazione di URL con ha `blog` esito negativo perché i valori `{ controller = Home, action = Index }` non corrispondono `{ controller = Blog, action = Article }` . Il routing quindi esegue il fallback per provare `default`, che ha esito positivo.

<a name="routing-areas-ref-label"></a>

## <a name="areas"></a>Aree

Le [aree](xref:mvc/controllers/areas) sono una funzionalità di MVC utilizzata per organizzare le funzionalità correlate in un gruppo come separato:

* Spazio dei nomi di routing per le azioni del controller.
* Struttura di cartelle per le visualizzazioni.

L'uso delle aree consente a un'app di avere più controller con lo stesso nome, purché abbiano aree diverse. Usando le aree si crea una gerarchia per il routing aggiungendo un altro parametro di route, `area` a `controller` e `action`. Questa sezione illustra il modo in cui il routing interagisce con le aree. Per informazioni dettagliate sull'uso delle aree con le visualizzazioni, vedere [aree](xref:mvc/controllers/areas) .

Nell'esempio seguente viene configurato MVC per l'utilizzo della route convenzionale predefinita e di una `area` Route per un oggetto `area` denominato `Blog` :

[!code-csharp[](routing/samples/3.x/AreasRouting/Startup.cs?name=snippet1)]

Nel codice precedente, <xref:Microsoft.AspNetCore.Builder.ControllerEndpointRouteBuilderExtensions.MapAreaControllerRoute%2A> viene chiamato per creare l'oggetto `"blog_route"` . Il secondo parametro, `"Blog"` , è il nome dell'area.

Quando si corrisponde a un percorso URL come `/Manage/Users/AddUser` , la `"blog_route"` Route genera i valori della route `{ area = Blog, controller = Users, action = AddUser }` . Il `area` valore di route viene prodotto da un valore predefinito per `area` . La route creata da `MapAreaControllerRoute` equivale a quanto segue:

[!code-csharp[](routing/samples/3.x/AreasRouting/Startup2.cs?name=snippet2)]

`MapAreaControllerRoute` crea una route che usa sia un valore predefinito che un vincolo per `area` usando il nome di area specificato, in questo caso `Blog`. Il valore predefinito assicura che la route generi sempre `{ area = Blog, ... }`, il vincolo richiede il valore `{ area = Blog, ... }` per la generazione di URL.

Il routing convenzionale dipende dall'ordinamento. In generale, le route con aree devono essere posizionate in precedenza perché sono più specifiche delle route senza un'area.

Utilizzando l'esempio precedente, i valori della route `{ area = Blog, controller = Users, action = AddUser }` corrispondono all'azione seguente:

[!code-csharp[](routing/samples/3.x/AreasRouting/Areas/Blog/Controllers/UsersController.cs)]

L'attributo [[area]](xref:Microsoft.AspNetCore.Mvc.AreaAttribute) è quello che denota un controller come parte di un'area. Questo controller si trova nell' `Blog` area. I controller senza `[Area]` attributo non sono membri di nessuna area e non corrispondono  quando il `area` valore di route viene fornito dal routing. Nell'esempio seguente solo il primo controller indicato può corrispondere ai valori di route `{ area = Blog, controller = Users, action = AddUser }`.

[!code-csharp[](routing/samples/3.x/AreasRouting/Areas/Blog/Controllers/UsersController.cs)]

[!code-csharp[](routing/samples/3.x/AreasRouting/Areas/Zebra/Controllers/UsersController.cs)]

[!code-csharp[](routing/samples/3.x/AreasRouting/Controllers/UsersController.cs)]

Lo spazio dei nomi di ogni controller viene visualizzato qui per completezza. Se il controller precedente usa lo stesso spazio dei nomi, verrà generato un errore del compilatore. Gli spazi dei nomi delle classi non hanno effetto sul routing di MVC.

I primi due controller sono membri di aree e corrispondono solo quando viene specificato il relativo nome di area dal valore di route `area`. Il terzo controller non è un membro di un'area e può corrispondere solo quando non vengono specificati valori per `area` dal routing.

<a name="aa"></a>

In termini di corrispondenza con *nessun valore*, l'assenza del valore `area` è come se il valore per `area` fosse Null o la stringa vuota.

Quando si esegue un'azione all'interno di un'area, il valore di route per `area` è disponibile come [valore di ambiente](#ambient) per il routing da usare per la generazione di URL. Ciò significa che per impostazione predefinita le aree funzionano *temporaneamente* per la generazione di URL, come illustrato nell'esempio seguente.

[!code-csharp[](routing/samples/3.x/AreasRouting/Startup3.cs?name=snippet3)]

[!code-csharp[](routing/samples/3.x/AreasRouting/Areas/Duck/Controllers/UsersController.cs)]

Il codice seguente genera un URL per `/Zebra/Users/AddUser` :

[!code-csharp[](routing/samples/3.x/AreasRouting/Controllers/HomeController.cs?name=snippet)]

<a name="action"></a>

## <a name="action-definition"></a>Definizione di azione

I metodi pubblici in un controller, ad eccezione di quelli con l'attributo [NonAction](xref:Microsoft.AspNetCore.Mvc.NonActionAttribute) , sono azioni.

## <a name="sample-code"></a>Codice di esempio

* [!INCLUDE[](~/includes/MyDisplayRouteInfo.md)]
* [Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/controllers/routing/samples/3.x) ([procedura per il download](xref:index#how-to-download-a-sample))

[!INCLUDE[](~/includes/dbg-route.md)]

::: moniker-end

::: moniker range="< aspnetcore-3.0"

ASP.NET Core MVC usa [middleware](xref:fundamentals/middleware/index) di routing per verificare la corrispondenza degli URL delle richieste in ingresso ed eseguirne il mapping alle azioni. Le route sono definite nel codice di avvio o negli attributi. Le route descrivono il modo in cui i percorsi URL devono corrispondere alle azioni. Vengono usate anche per generare gli URL (per i collegamenti) inviati nelle risposte.

Le azioni vengono indirizzate in modo convenzionale o con attributi. Se una route viene inserita nel controller o nell'azione, viene indirizzata con attributi. Per altre informazioni, vedere [Routing misto](#routing-mixed-ref-label).

Questo documento illustra le interazioni tra MVC e il routing e il modo in cui le tipiche app MVC usano le funzionalità di routing. Vedere [Routing](xref:fundamentals/routing) per informazioni dettagliate sul routing avanzato.

## <a name="setting-up-routing-middleware"></a>Impostazione del middleware di routing

Nel metodo *Configure* può apparire codice simile al seguente:

```csharp
app.UseMvc(routes =>
{
   routes.MapRoute("default", "{controller=Home}/{action=Index}/{id?}");
});
```

Nella chiamata a `UseMvc`, si usa `MapRoute` per creare una singola route, a cui si farà riferimento come alla route predefinita (`default`). La maggior parte delle app MVC userà una route con un modello simile alla route `default`.

Il modello di route `"{controller=Home}/{action=Index}/{id?}"` può corrispondere a un percorso URL come `/Products/Details/5` ed estrae i valori di route `{ controller = Products, action = Details, id = 5 }` suddividendo il percorso in token. MVC tenterà di individuare un controller denominato `ProductsController` e di eseguire l'azione `Details`:

```csharp
public class ProductsController : Controller
{
   public IActionResult Details(int id) { ... }
}
```

Si noti che in questo esempio l'associazione di modelli usa il valore di `id = 5` per impostare il parametro `id` su `5` quando si richiama l'azione. Per maggiori dettagli, vedere [Associazione di modelli](../models/model-binding.md).

Usando la route `default`:

```csharp
routes.MapRoute("default", "{controller=Home}/{action=Index}/{id?}");
```

Il modello di route:

* `{controller=Home}` definisce `Home` come oggetto `controller` predefinito

* `{action=Index}` definisce `Index` come oggetto `action` predefinito

* `{id?}` definisce `id` come facoltativo

I parametri di route predefiniti e facoltativi non devono necessariamente essere presenti nel percorso URL per trovare una corrispondenza. Vedere [Riferimento per i modelli di route](xref:fundamentals/routing#route-template-reference) per una descrizione dettagliata della sintassi del modello di route.

`"{controller=Home}/{action=Index}/{id?}"` può corrispondere al percorso URL `/` e restituirà i valori di route `{ controller = Home, action = Index }`. I valori per `controller` e `action` usano i valori predefiniti, `id` non genera un valore perché non esiste un segmento corrispondente nel percorso URL. MVC usa questi valori di route per selezionare l'azione `HomeController` e `Index`:

```csharp
public class HomeController : Controller
{
  public IActionResult Index() { ... }
}
```

Usando questa definizione del controller e il modello di route, l'azione `HomeController.Index` viene eseguita per tutti i percorsi URL seguenti:

* `/Home/Index/17`

* `/Home/Index`

* `/Home`

* `/`

Il metodo pratico `UseMvcWithDefaultRoute`:

```csharp
app.UseMvcWithDefaultRoute();
```

Può essere usato per sostituire:

```csharp
app.UseMvc(routes =>
{
   routes.MapRoute("default", "{controller=Home}/{action=Index}/{id?}");
});
```

`UseMvc` e `UseMvcWithDefaultRoute` aggiungono un'istanza di `RouterMiddleware` alla pipeline middleware. MVC non interagisce direttamente con il middleware e usa il routing per gestire le richieste. MVC è connesso alle route attraverso un'istanza di `MvcRouteHandler`. Il codice all'interno di `UseMvc` è simile al seguente:

```csharp
var routes = new RouteBuilder(app);

// Add connection to MVC, will be hooked up by calls to MapRoute.
routes.DefaultHandler = new MvcRouteHandler(...);

// Execute callback to register routes.
// routes.MapRoute("default", "{controller=Home}/{action=Index}/{id?}");

// Create route collection and add the middleware.
app.UseRouter(routes.Build());
```

`UseMvc` non definisce direttamente le route, aggiunge un segnaposto alla raccolta di route per la route `attribute`. L'overload `UseMvc(Action<IRouteBuilder>)` consente di aggiungere le proprie route e supporta anche il routing con attributi.  `UseMvc` e tutte le relative varianti aggiungono un segnaposto per l'attributo Route: il routing degli attributi è sempre disponibile indipendentemente dalla configurazione `UseMvc` . `UseMvcWithDefaultRoute` definisce una route predefinita e supporta il routing con attributi. La sezione [Routing con attributi](#attribute-routing-ref-label) include maggiori dettagli sul routing con attributi.

<a name="routing-conventional-ref-label"></a>

## <a name="conventional-routing"></a>Routing convenzionale

La route predefinita (`default`):

```csharp
routes.MapRoute("default", "{controller=Home}/{action=Index}/{id?}");
```

Il codice precedente è un esempio di routing convenzionale. Questo stile è denominato routing convenzionale perché stabilisce una *convenzione* per i percorsi URL:

* Il primo segmento di percorso viene mappato al nome del controller.
* Il secondo esegue il mapping al nome dell'azione.
* Il terzo segmento viene usato per un oggetto facoltativo `id` . `id` esegue il mapping a un'entità del modello.

Usando la route `default`, il percorso URL `/Products/List` esegue il mapping all'azione `ProductsController.List` e `/Blog/Article/17` lo esegue a `BlogController.Article`. Questo mapping si basa **solo** sui nomi dell'azione e del controller e non su spazi dei nomi, percorsi dei file di origine o parametri del metodo.

> [!TIP]
> L'uso del routing convenzionale con la route predefinita consente di compilare rapidamente l'applicazione senza dover creare un nuovo modello di URL per ogni azione che si definisce. Per un'applicazione con azioni di stile CRUD, la coerenza degli URL tra i controller può semplificare il codice e rendere l'interfaccia utente più prevedibile.

> [!WARNING]
> L'oggetto `id` è definito come facoltativo nel modello di route, ovvero le azioni possono essere eseguite senza l'ID specificato come parte dell'URL. In genere, se si omette `id` dall'URL, il valore viene impostato su `0` dall'associazione del modello e di conseguenza non viene rilevata alcuna entità nel database corrispondente a `id == 0`. Il routing con attributi offre un controllo con granularità fine che consente di rendere l'ID obbligatorio per alcune azioni e non per altre. Per convenzione la documentazione include i parametri facoltativi come `id` quando è probabile che appaiano nell'uso corretto.

## <a name="multiple-routes"></a>Route multiple

È possibile aggiungere più route all'interno di `UseMvc` aggiungendo altre chiamate a `MapRoute`. Questa operazione consente di definire più convenzioni o di aggiungere route convenzionali dedicate a un'azione specifica, ad esempio:

```csharp
app.UseMvc(routes =>
{
   routes.MapRoute("blog", "blog/{*article}",
            defaults: new { controller = "Blog", action = "Article" });
   routes.MapRoute("default", "{controller=Home}/{action=Index}/{id?}");
});
```

La route `blog` è una *route convenzionale dedicata*, vale a dire che usa il sistema di routing convenzionale, ma è dedicata a un'azione specifica. Poiché `controller` e `action` non appaiono nel modello di route come parametri, possono avere solo i valori predefiniti e quindi questa route eseguirà sempre il mapping all'azione `BlogController.Article`.

Le route sono ordinate nella raccolta di route e verranno elaborate nell'ordine in cui vengono aggiunte. Quindi in questo esempio la route `blog` viene tentata prima della route `default`.

> [!NOTE]
> Le *Route convenzionali dedicate* usano spesso parametri di route **catch-all** come `{*article}` per acquisire la parte rimanente del percorso URL. In questo modo una route può diventare troppo "greedy", ovvero corrispondere agli URL che dovrebbero corrispondere ad altre route. Posizionare le route "greedy" più avanti nella tabella delle route per risolvere il problema.

### <a name="fallback"></a>Fallback

Come parte dell'elaborazione della richiesta, MVC verifica che i valori di route possano essere usati per trovare un controller e un'azione nell'applicazione. Se i valori di route non corrispondano a un'azione, la route non viene considerata una corrispondenza e viene tentata la route successiva. Questa operazione si chiama *fallback* e viene eseguita allo scopo di semplificare i casi in cui le route convenzionali si sovrappongono.

### <a name="disambiguating-actions"></a>Rimozione dell'ambiguità delle azioni

Quando due azioni corrispondono in base al routing, MVC deve rimuovere l'ambiguità per scegliere il candidato migliore, altrimenti genera un'eccezione. Ad esempio:

```csharp
public class ProductsController : Controller
{
   public IActionResult Edit(int id) { ... }

   [HttpPost]
   public IActionResult Edit(int id, Product product) { ... }
}
```

Questo controller definisce due azioni corrispondenti al percorso URL `/Products/Edit/17` e indirizzano i dati `{ controller = Products, action = Edit, id = 17 }`. Si tratta di un modello tipico per i controller MVC in cui `Edit(int)` visualizza un modulo per modificare un prodotto e `Edit(int, Product)` elabora il modulo inviato. Per poter eseguire l'operazione, MVC deve scegliere `Edit(int, Product)` se la richiesta è un `POST` HTTP e `Edit(int)` quando il verbo HTTP è un altro elemento.

`HttpPostAttribute` (`[HttpPost]`) è un'implementazione di `IActionConstraint` che consente di selezionare l'azione solo quando il verbo HTTP è `POST`. La presenza di un oggetto `IActionConstraint` fa sì che `Edit(int, Product)` sia una corrispondenza migliore di `Edit(int)`, quindi `Edit(int, Product)` viene tentata per prima.

È necessario scrivere implementazioni personalizzate di `IActionConstraint` solo in scenari specializzati, ma è importante comprendere il ruolo degli attributi, ad esempio `HttpPostAttribute`. Attributi simili vengono definiti per altri verbi HTTP. Nel routing convenzionale è normale che le azioni usino lo stesso nome di azione quando fanno parte di un flusso di lavoro `show form -> submit form`. La comodità offerta da questo modello sarà più evidente dopo aver esaminato la sezione relativa a [IActionConstraint](#understanding-iactionconstraint).

Se vi sono più route corrispondenti e MVC non è in grado di trovare la route migliore, viene generata un'eccezione `AmbiguousActionException`.

<a name="routing-route-name-ref-label"></a>

### <a name="route-names"></a>Nomi di route

Le stringhe `"blog"` e `"default"` degli esempi seguenti sono nomi di route:

```csharp
app.UseMvc(routes =>
{
   routes.MapRoute("blog", "blog/{*article}",
               defaults: new { controller = "Blog", action = "Article" });
   routes.MapRoute("default", "{controller=Home}/{action=Index}/{id?}");
});
```

I nomi di route consentono di assegnare a una route un nome logico, in modo che la route denominata possa essere usata per la generazione di URL. Questo semplifica notevolmente la creazione degli URL quando l'ordinamento delle route può rendere complessa la generazione di URL. I nomi delle route devono essere univoci a livello di applicazione.

I nomi di route non influiscono sulla corrispondenza degli URL o sulla gestione delle richieste, vengono usati solo per la generazione di URL. L'articolo sul [routing](xref:fundamentals/routing) contiene informazioni dettagliate sulla generazione di URL, inclusa la generazione negli helper specifici di MVC.

<a name="attribute-routing-ref-label"></a>

## <a name="attribute-routing"></a>Routing con attributi

Il routing con attributi usa un set di attributi per eseguire il mapping delle azioni direttamente ai modelli di route. Nell'esempio seguente viene usato l'oggetto `app.UseMvc();` nel metodo `Configure` e non viene passata alcuna route. `HomeController` corrisponde a un set di URL simile a quello a cui corrisponderebbe la route predefinita `{controller=Home}/{action=Index}/{id?}`:

```csharp
public class HomeController : Controller
{
   [Route("")]
   [Route("Home")]
   [Route("Home/Index")]
   public IActionResult Index()
   {
      return View();
   }
   [Route("Home/About")]
   public IActionResult About()
   {
      return View();
   }
   [Route("Home/Contact")]
   public IActionResult Contact()
   {
      return View();
   }
}
```

L'azione `HomeController.Index()` verrà eseguita per tutti i percorsi URL, `/`, `/Home` o `/Home/Index`.

> [!NOTE]
> In questo esempio viene evidenziata un'importante differenza a livello di programmazione tra il routing con attributi e il routing convenzionale. Il routing con attributi richiede più input per specificare una route, la route convenzionale predefinita gestisce le route in modo più conciso. Tuttavia, il routing con attributi consente, e richiede, un controllo preciso dei modelli route da applicare a ogni azione.

Con il routing con attributi i nomi del controller e delle azioni **non** hanno alcun ruolo nella selezione dell'azione. In questo esempio viene verificata la corrispondenza degli stessi URL dell'esempio precedente.

```csharp
public class MyDemoController : Controller
{
   [Route("")]
   [Route("Home")]
   [Route("Home/Index")]
   public IActionResult MyIndex()
   {
      return View("Index");
   }
   [Route("Home/About")]
   public IActionResult MyAbout()
   {
      return View("About");
   }
   [Route("Home/Contact")]
   public IActionResult MyContact()
   {
      return View("Contact");
   }
}
```

> [!NOTE]
> I modelli di route precedenti non definiscono i parametri di route per `action`, `area` e `controller`. In realtà, questi parametri di route non sono consentiti nelle route con attributi. Poiché il modello di route è già associato a un'azione, non avrebbe senso analizzare il nome dell'azione dall'URL.

## <a name="attribute-routing-with-httpverb-attributes"></a>Routing con attributi Http[Verb]

Il routing con attributi può usare anche gli attributi `Http[Verb]`, ad esempio `HttpPostAttribute`. Tutti questi attributi possono accettare un modello di route. Questo esempio illustra due azioni che corrispondono allo stesso modello di route:

```csharp
[HttpGet("/products")]
public IActionResult ListProducts()
{
   // ...
}

[HttpPost("/products")]
public IActionResult CreateProduct(...)
{
   // ...
}
```

Per un percorso URL come `/products` l'azione `ProductsApi.ListProducts` verrà eseguita quando il verbo HTTP è `GET` e `ProductsApi.CreateProduct` verrà eseguita quando il verbo HTTP è `POST`. Nel routing con attributi l'URL viene prima confrontato con il set di modelli di route definiti dagli attributi della route. Quando un modello di route corrisponde, vengono applicati i vincoli `IActionConstraint` per determinare quali azioni possono essere eseguite.

> [!TIP]
> Quando si compila un'API REST, è raro che si voglia usare `[Route(...)]` su un metodo di azione perché l'azione accetta tutti i metodi HTTP. Si consiglia di usare l'oggetto `Http*Verb*Attributes`, più specifico, per essere precisi circa gli elementi supportati dall'API. I client delle API REST devono sapere quali percorsi e verbi HTTP devono essere mappati a operazioni logiche specifiche.

Poiché una route con attributi si applica a un'azione specifica, è facile fare in modo che i parametri siano richiesti come parte della definizione del modello di route. In questo esempio `id` è richiesto come parte del percorso URL.

```csharp
public class ProductsApiController : Controller
{
   [HttpGet("/products/{id}", Name = "Products_List")]
   public IActionResult GetProduct(int id) { ... }
}
```

L'azione `ProductsApi.GetProduct(int)` verrà eseguita per un percorso URL come `/products/3`, ma non per un percorso URL come `/products`. Vedere [Routing](xref:fundamentals/routing) per una descrizione completa dei modelli di route e delle opzioni correlate.

## <a name="route-name"></a>Nome di route

Nell’esempio di codice riportato di seguito viene definito un *nome di route* di `Products_List`:

```csharp
public class ProductsApiController : Controller
{
   [HttpGet("/products/{id}", Name = "Products_List")]
   public IActionResult GetProduct(int id) { ... }
}
```

I nomi di route possono essere usati per generare un URL in base a un percorso specifico. I nomi di route non influiscono sul comportamento del routing riguardo alla corrispondenza degli URL e vengono usati solo per la generazione di URL. I nomi delle route devono essere univoci a livello di applicazione.

> [!NOTE]
> Confrontare questa opzione con la *route predefinita* convenzionale, che definisce il parametro `id` come facoltativo (`{id?}`). La possibilità di specificare in modo preciso le API presenta dei vantaggi, ad esempio consente di inviare `/products` e `/products/5` ad azioni  differenti.

<a name="routing-combining-ref-label"></a>

### <a name="combining-routes"></a>Combinazione di route

Per rendere il routing con attributi meno ripetitivo, gli attributi di route del controller vengono combinati con gli attributi di route delle singole azioni. I modelli di route definiti per il controller vengono anteposti ai modelli di route delle azioni. Inserendo un attributo di route nel controller, **tutte** le azioni presenti nel controller useranno il routing con attributi.

```csharp
[Route("products")]
public class ProductsApiController : Controller
{
   [HttpGet]
   public IActionResult ListProducts() { ... }

   [HttpGet("{id}")]
   public ActionResult GetProduct(int id) { ... }
}
```

In questo esempio il percorso URL `/products` può corrispondere a `ProductsApi.ListProducts` e il percorso URL `/products/5` può corrispondere a `ProductsApi.GetProduct(int)`. Entrambe le azioni corrispondono solo a HTTP `GET` perché sono contrassegnate con `HttpGetAttribute` .

I modelli di route applicati a un'azione che iniziano con `/` o `~/` non vengono combinati con i modelli di route applicati al controller. Questo esempio illustra la corrispondenza di un set di percorsi URL simile alla *route predefinita*.

```csharp
[Route("Home")]
public class HomeController : Controller
{
    [Route("")]      // Combines to define the route template "Home"
    [Route("Index")] // Combines to define the route template "Home/Index"
    [Route("/")]     // Doesn't combine, defines the route template ""
    public IActionResult Index()
    {
        ViewData["Message"] = "Home index";
        var url = Url.Action("Index", "Home");
        ViewData["Message"] = "Home index" + "var url = Url.Action; =  " + url;
        return View();
    }

    [Route("About")] // Combines to define the route template "Home/About"
    public IActionResult About()
    {
        return View();
    }   
}
```

<a name="routing-ordering-ref-label"></a>

### <a name="ordering-attribute-routes"></a>Ordinamento delle route con attributi

Diversamente dalle route convenzionali, che vengono eseguite in un ordine definito, il routing degli attributi compila un albero e trova la corrispondenza di tutte le route simultaneamente. Si comporta come se le voci delle route seguissero un ordine ideale in cui le route più specifiche vengono probabilmente eseguite prima delle route più generali.

Ad esempio, una route come `blog/search/{topic}` è più specifica di una route come `blog/{*article}`. In termini logici la route `blog/search/{topic}` viene eseguita per prima, per impostazione predefinita, perché questo è l'ordine più plausibile. Usando il routing convenzionale, lo sviluppatore ordina le route in base alle proprie esigenze.

Nelle route con attributi si può configurare un ordine usando la proprietà `Order` di tutti gli attributi di route specificati dal framework. Le route vengono elaborate in base a un ordinamento crescente della proprietà `Order`. L'ordine predefinito è `0`. L'impostazione di una route usando `Order = -1` viene eseguita prima delle route per cui non è impostato un ordine. L'impostazione di una route usando `Order = 1` viene eseguito dopo l'ordinamento predefinito delle route.

> [!TIP]
> Evitare la dipendenza da `Order`. Se lo spazio URL richiede i valori in un ordine esplicito per indirizzare correttamente i dati, si può probabilmente creare confusione anche per i client. In genere il routing con attributi seleziona la route corretta con la corrispondenza di URL. Se l'ordine predefinito usato per la generazione di URL non funziona, usare il nome della route come override è in genere più semplice che applicare la proprietà `Order`.

Razor Il routing di pagine e il routing del controller MVC condividono un'implementazione. Le informazioni sull'ordine di route negli Razor argomenti pagine sono disponibili nelle [ Razor pagine Route e app Conventions: Order Route](xref:razor-pages/razor-pages-conventions#route-order).

<a name="routing-token-replacement-templates-ref-label"></a>

## <a name="token-replacement-in-route-templates-controller-action-area"></a>Sostituzione dei token nei modelli di route ([controller], [action], [area])

Per praticità, le route degli attributi supportano la *sostituzione dei token* mediante l'inclusione di un token tra parentesi quadre ( `[` , `]` ). I token `[action]`, `[area]` e `[controller]` vengono sostituiti con i valori di nome dell'azione, nome dell'area e nome del controller dall'azione in cui è definita la route. Nell'esempio seguente le azioni corrispondono ai percorsi URL come descritto nei commenti:

[!code-csharp[](routing/samples/2.x/main/Controllers/ProductsController.cs?range=7-11,13-17,20-22)]

La sostituzione dei token avviene come ultimo passaggio della creazione delle route con attributi. L'esempio precedente si comporterà come il codice seguente:

[!code-csharp[](routing/samples/2.x/main/Controllers/ProductsController2.cs?range=7-11,13-17,20-22)]

Le route con attributi possono anche essere combinate con l'ereditarietà. Ciò risulta particolarmente efficace se combinato con la sostituzione dei token.

```csharp
[Route("api/[controller]")]
public abstract class MyBaseController : Controller { ... }

public class ProductsController : MyBaseController
{
   [HttpGet] // Matches '/api/Products'
   public IActionResult List() { ... }

   [HttpPut("{id}")] // Matches '/api/Products/{id}'
   public IActionResult Edit(int id) { ... }
}
```

La sostituzione dei token si applica anche ai nomi di route definiti dalle route con attributi. `[Route("[controller]/[action]", Name="[controller]_[action]")]` genera un nome di route univoco per ogni azione.

Per verificare la corrispondenza del delimitatore letterale della sostituzione di token `[` o `]`, eseguirne l'escape ripetendo il carattere (`[[` o `]]`).

::: moniker-end

::: moniker range="= aspnetcore-2.2"

<a name="routing-token-replacement-transformers-ref-label"></a>

### <a name="use-a-parameter-transformer-to-customize-token-replacement"></a>Usare un trasformatore di parametri per personalizzare la sostituzione dei token

La sostituzione dei token può essere personalizzata usando un trasformatore di parametri. Un trasformatore di parametri implementa `IOutboundParameterTransformer` e trasforma il valore dei parametri. Ad esempio, un trasformatore di parametri `SlugifyParameterTransformer` personalizzato cambia il valore di route `SubscriptionManagement` in `subscription-management`.

`RouteTokenTransformerConvention` è una convenzione del modello di applicazione che:

* Applica un trasformatore di parametri a tutte le route di attributi in un'applicazione.
* Personalizza i valori dei token delle route di attributi quando vengono sostituiti.

```csharp
public class SubscriptionManagementController : Controller
{
    [HttpGet("[controller]/[action]")] // Matches '/subscription-management/list-all'
    public IActionResult ListAll() { ... }
}
```

La convenzione `RouteTokenTransformerConvention` è registrata come un'opzione in `ConfigureServices`.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc(options =>
    {
        options.Conventions.Add(new RouteTokenTransformerConvention(
                                     new SlugifyParameterTransformer()));
    });
}

public class SlugifyParameterTransformer : IOutboundParameterTransformer
{
    public string TransformOutbound(object value)
    {
        if (value == null) { return null; }

        // Slugify value
        return Regex.Replace(value.ToString(), "([a-z])([A-Z])", "$1-$2").ToLower();
    }
}
```

::: moniker-end


::: moniker range="< aspnetcore-3.0"
<a name="routing-multiple-routes-ref-label"></a>

### <a name="multiple-routes"></a>Route multiple

Il routing con attributi supporta la definizione di più route che raggiungono la stessa azione. L'uso più comune è simulare il comportamento della *route convenzionale predefinita* come illustrato nell'esempio seguente:

```csharp
[Route("[controller]")]
public class ProductsController : Controller
{
   [Route("")]     // Matches 'Products'
   [Route("Index")] // Matches 'Products/Index'
   public IActionResult Index()
}
```

Inserire più attributi di route nel controller significa che verranno tutti combinati con tutti gli attributi di route nei metodi dell'azione.

```csharp
[Route("Store")]
[Route("[controller]")]
public class ProductsController : Controller
{
   [HttpPost("Buy")]     // Matches 'Products/Buy' and 'Store/Buy'
   [HttpPost("Checkout")] // Matches 'Products/Checkout' and 'Store/Checkout'
   public IActionResult Buy()
}
```

Quando più attributi di route (che implementano `IActionConstraint`) sono inseriti in un'azione, ogni vincolo dell'azione viene combinato con il modello di route dall'attributo che lo definisce.

```csharp
[Route("api/[controller]")]
public class ProductsController : Controller
{
   [HttpPut("Buy")]      // Matches PUT 'api/Products/Buy'
   [HttpPost("Checkout")] // Matches POST 'api/Products/Checkout'
   public IActionResult Buy()
}
```

> [!TIP]
> Benché l'uso di più route per le azioni possa sembrare efficace, è preferibile mantenere lo spazio URL dell'applicazione semplice e ben definito. Usare più route per le azioni solo se necessario, ad esempio per supportare i client esistenti.

<a name="routing-attr-options"></a>

### <a name="specifying-attribute-route-optional-parameters-default-values-and-constraints"></a>Definizione di parametri facoltativi, valori predefiniti e vincoli della route con attributi

Le route con attributi supportano la stessa sintassi inline delle route convenzionali per specificare i parametri facoltativi, i valori predefiniti e i vincoli.

```csharp
[HttpPost("product/{id:int}")]
public IActionResult ShowProduct(int id)
{
   // ...
}
```

Vedere [Riferimento per i modelli di route](xref:fundamentals/routing#route-template-reference) per una descrizione dettagliata della sintassi del modello di route.

<a name="routing-cust-rt-attr-irt-ref-label"></a>

### <a name="custom-route-attributes-using-iroutetemplateprovider"></a>Attributi di route personalizzati che usano `IRouteTemplateProvider`

Tutti gli attributi di route disponibili nel framework (`[Route(...)]`, `[HttpGet(...)]` e così via) implementano l'interfaccia `IRouteTemplateProvider`. MVC cerca gli attributi per le classi del controller e i metodi delle azioni all'avvio dell'app e usa quelli che implementano `IRouteTemplateProvider` per compilare il set iniziale di route.

È possibile implementare `IRouteTemplateProvider` per definire i propri attributi di route. Ogni `IRouteTemplateProvider` consente di definire una singola route con un modello di route, un ordinamento e un nome personalizzati:

```csharp
public class MyApiControllerAttribute : Attribute, IRouteTemplateProvider
{
   public string Template => "api/[controller]";

   public int? Order { get; set; }

   public string Name { get; set; }
}
```

L'attributo dell'esempio precedente imposta automaticamente `Template` su `"api/[controller]"` quando si applica `[MyApiController]`.

<a name="routing-app-model-ref-label"></a>

### <a name="using-application-model-to-customize-attribute-routes"></a>Uso del modello applicativo per personalizzare le route con attributi

Il *modello applicativo* è un modello a oggetti creato all'avvio con tutti i metadati usati da MVC per indirizzare ed eseguire le azioni. Il *modello applicativo* include tutti i dati raccolti da attributi di route (attraverso `IRouteTemplateProvider`). È possibile scrivere *convenzioni* per modificare il modello applicativo in fase di avvio per personalizzare il comportamento del routing. Questa sezione illustra un semplice esempio di personalizzazione del routing con il modello applicativo.

[!code-csharp[](routing/samples/2.x/main/NamespaceRoutingConvention.cs)]

<a name="routing-mixed-ref-label"></a>

## <a name="mixed-routing-attribute-routing-vs-conventional-routing"></a>Routing misto: routing con attributi e routing convenzionale

Le applicazioni MVC sono in grado di usare una combinazione di routing convenzionale e routing con attributi. In genere le route convenzionali vengono usate per i controller che gestiscono le pagine HTML per i browser e le route con attributi per i controller che gestiscono le API REST.

Le azioni vengono indirizzate in modo convenzionale o con attributi. Se una route viene inserita nel controller o nell'azione, viene indirizzata con attributi. Le azioni che definiscono le route con attributi non possono essere raggiunte usando le route convenzionali e viceversa. **Qualsiasi** attributo di route del controller rende tutte le azioni presenti nel controller indirizzate con attributi.

> [!NOTE]
> Ciò che distingue i due tipi di sistema di routing è il processo applicato dopo che si è verificata la corrispondenza di un URL con un modello di route. Nel routing convenzionale i valori di route della corrispondenza vengono usati per scegliere l'azione e il controller da una tabella di ricerca di tutte le azioni indirizzate in modo convenzionale. Nel routing con attributi ogni modello è già associato a un'azione e non è necessaria alcuna ulteriore ricerca.

## <a name="complex-segments"></a>Segmenti complessi

I segmenti complessi (ad esempio, `[Route("/dog{token}cat")]`), vengono elaborati individuando corrispondenze per i valori letterali da destra a sinistra in modalità non-greedy. Vedere [il codice sorgente](https://github.com/aspnet/Routing/blob/9cea167cfac36cf034dbb780e3f783114ef94780/src/Microsoft.AspNetCore.Routing/Patterns/RoutePatternMatcher.cs#L296) per una descrizione. Per altre informazioni, vedere [questo problema](https://github.com/dotnet/AspNetCore.Docs/issues/8197).

<a name="routing-url-gen-ref-label"></a>

## <a name="url-generation"></a>Generazione di URL

Le applicazioni MVC sono in grado di usare le funzionalità di generazione di URL del routing per generare i collegamenti URL alle azioni. La generazione di URL evita la codifica degli URL e consente di rendere il codice più affidabile e gestibile. Questa sezione illustra le funzionalità di generazione di URL disponibili in MVC e tratta solo le nozioni di base sul funzionamento della generazione di URL. Vedere [Routing](xref:fundamentals/routing) per una descrizione dettagliata della generazione di URL.

L'interfaccia `IUrlHelper` è l'elemento sottostante dell'infrastruttura tra MVC e il routing per la generazione di URL. È possibile trovare un'istanza di `IUrlHelper` disponibile usando la proprietà `Url` in controller, visualizzazioni e componenti di visualizzazione.

In questo esempio l'interfaccia `IUrlHelper` viene usata attraverso la proprietà `Controller.Url` per generare un URL a un'altra azione.

[!code-csharp[](routing/samples/2.x/main/Controllers/UrlGenerationController.cs?name=snippet_1)]

Se l'applicazione usa la route convenzionale predefinita, il valore della variabile `url` sarà la stringa del percorso URL `/UrlGeneration/Destination`. Il percorso URL viene creato dal routing combinando i valori di route della richiesta corrente (valori di ambiente) con i valori passati a `Url.Action` e sostituendo tali valori nel modello di route:

```
ambient values: { controller = "UrlGeneration", action = "Source" }
values passed to Url.Action: { controller = "UrlGeneration", action = "Destination" }
route template: {controller}/{action}/{id?}

result: /UrlGeneration/Destination
```

Il valore di ogni parametro di route del modello di route viene sostituito attraverso la corrispondenza dei nomi con i valori e i valori di ambiente. Per un parametro di route senza un valore si può usare un valore predefinito, se ne esiste uno, oppure ignorare il parametro se è facoltativo (come nel caso di `id` in questo esempio). La generazione di URL avrà esito negativo se un parametro di route obbligatorio non ha un valore corrispondente. Se la generazione di URL non riesce per una route, viene tentata la route successiva finché non vengono tentate tutte le route o viene trovata una corrispondenza.

Nell'esempio di `Url.Action` precedente si presuppone che il routing sia convenzionale, ma la generazione di URL funziona in modo analogo con il routing con attributi, sebbene i concetti siano differenti. Con il routing convenzionale i valori di route vengono usati per espandere un modello e i valori di route per `controller` e `action` in genere appaiono in quel modello. Questo procedimento funziona perché gli URL corrispondenti in base al routing rispettano una *convenzione*. Nel routing con attributi i valori di route per `controller` e `action` non possono essere visualizzati nel modello ma vengono usati per individuare il modello da usare.

Questo esempio usa il routing con attributi:

[!code-csharp[](routing/samples/2.x/main/StartupUseMvc.cs?name=snippet_1)]

[!code-csharp[](routing/samples/2.x/main/Controllers/UrlGenerationControllerAttr.cs?name=snippet_1)]

MVC compila una tabella di ricerca di tutte le azioni indirizzate con attributi e stabilisce la corrispondenza dei valori `controller` e `action` per selezionare il modello di route da usare per la generazione di URL. Nell'esempio precedente viene generato `custom/url/to/destination`.

### <a name="generating-urls-by-action-name"></a>Generazione di URL in base al nome dell'azione

`Url.Action` (`IUrlHelper` . `Action`) e tutti gli overload correlati sono basati sull'idea che si voglia definire l'oggetto a cui ci si sta collegando specificando un nome di controller e un nome di azione.

> [!NOTE]
> Quando si usa `Url.Action`, i valori di route correnti per `controller` e `action` vengono specificati automaticamente: i valori di `controller` e `action` fanno parte sia dei *valori di ambiente* **sia dei** *valori*. Il metodo `Url.Action`, usa sempre i valori correnti di `action` e `controller` e genererà un percorso URL che indirizza all'azione corrente.

Il routing tenta di usare i valori nei valori di ambiente per ottenere le informazioni non specificate durante la generazione di un URL. Usando una route come `{a}/{b}/{c}/{d}` e i valori di ambiente `{ a = Alice, b = Bob, c = Carol, d = David }`, il routing ha informazioni sufficienti per generare un URL senza valori aggiuntivi, poiché tutti i parametri di route hanno un valore. Se è stato aggiunto il valore `{ d = Donovan }`, il valore `{ d = David }` viene ignorato e il percorso URL generato è `Alice/Bob/Carol/Donovan`.

> [!WARNING]
> I percorsi URL sono gerarchici. Nell'esempio precedente, se è stato aggiunto il valore `{ c = Cheryl }`, entrambi i valori `{ c = Carol, d = David }` vengono ignorati. In questo caso non è più disponibile un valore per `d` e la generazione dell'URL avrà esito negativo. Sarà necessario specificare un valore per `c` e `d`.  In apparenza questo problema si può risolvere con la route predefinita (`{controller}/{action}/{id?}`), ma in realtà questo comportamento si verifica raramente perché `Url.Action` specifica sempre in modo esplicito un valore `controller` e `action`.

Anche gli overload di `Url.Action` di maggiore durata accettano anche un oggetto *valori di route* aggiuntivo per specificare i valori per i parametri di route diversi da `controller` e `action`. In genere viene usato con `id` come `Url.Action("Buy", "Products", new { id = 17 })`. Per convenzione l'oggetto *valori di route* è in genere un oggetto di tipo anonimo, ma può anche essere un `IDictionary<>` o un *normale oggetto .NET*. I valori di route aggiuntivi che non corrispondono a parametri di route vengono inseriti nella stringa di query.

[!code-csharp[](routing/samples/2.x/main/Controllers/TestController.cs)]

> [!TIP]
> Per creare un URL assoluto, usare un overload che accetta un `protocol`: `Url.Action("Buy", "Products", new { id = 17 }, protocol: Request.Scheme)`

<a name="routing-gen-urls-route-ref-label"></a>

### <a name="generating-urls-by-route"></a>Generazione di URL in base alla route

Il codice precedente illustra come generare un URL passando il nome del controller e dell'azione. `IUrlHelper` specifica anche la famiglia di metodi `Url.RouteUrl`. Questi metodi sono simili a `Url.Action`, ma non copiano i valori correnti di `action` e `controller` nei valori di route. L'uso più comune consiste nello specificare un nome di route per usare un percorso specifico per generare l'URL, in genere *senza* specificare un nome di controller o azione.

[!code-csharp[](routing/samples/2.x/main/Controllers/UrlGenerationControllerRouting.cs?name=snippet_1)]

<a name="routing-gen-urls-html-ref-label"></a>

### <a name="generating-urls-in-html"></a>Generazione di URL in HTML

`IHtmlHelper` specifica i metodi `HtmlHelper``Html.BeginForm` e `Html.ActionLink` per generare rispettivamente gli elementi `<form>` e `<a>`. Questi metodi usano il metodo `Url.Action` per generare un URL e accettano argomenti simili. Gli oggetti `Url.RouteUrl` complementi di `HtmlHelper` sono `Html.BeginRouteForm` e `Html.RouteLink` e hanno una funzionalità simile.

Gli helper tag generano gli URL attraverso l'helper tag `form` e l'helper tag `<a>`. Entrambi usano `IUrlHelper` per la propria implementazione. Per altre informazioni, vedere [Uso dei moduli](../views/working-with-forms.md).

All'interno delle visualizzazioni `IUrlHelper` è reso disponibile dalla proprietà `Url` per tutte le generazioni di URL ad hoc che non rientrano nelle situazioni descritte in precedenza.

<a name="routing-gen-urls-action-ref-label"></a>

### <a name="generating-urls-in-action-results"></a>Generazione di URL nei risultati delle azioni

Negli esempi precedenti è stato usato `IUrlHelper` in un controller, ma l'uso più comune in un controller è generare un URL come parte del risultato di un'azione.

Le classi di base `ControllerBase` e `Controller` offrono metodi pratici per i risultati delle azioni che fanno riferimento a un'altra azione. Un uso tipico è eseguire il reindirizzamento dopo aver accettato l'input dell'utente.

```csharp
public IActionResult Edit(int id, Customer customer)
{
    if (ModelState.IsValid)
    {
        // Update DB with new details.
        return RedirectToAction("Index");
    }
    return View(customer);
}
```

I metodi factory dei risultati delle azioni seguono un modello simile ai metodi per `IUrlHelper`.

<a name="routing-dedicated-ref-label"></a>

### <a name="special-case-for-dedicated-conventional-routes"></a>Caso speciale per le route convenzionali dedicate

Nel routing convenzionale è possibile usare un tipo speciale di definizione di route denominata *route convenzionale dedicata*. Nell'esempio seguente la route denominata `blog` è una route convenzionale dedicata.

```csharp
app.UseMvc(routes =>
{
    routes.MapRoute("blog", "blog/{*article}",
        defaults: new { controller = "Blog", action = "Article" });
    routes.MapRoute("default", "{controller=Home}/{action=Index}/{id?}");
});
```

Usando queste definizioni di route, `Url.Action("Index", "Home")` genera il percorso URL `/` con la route `default` per un motivo ben preciso. Si potrebbe pensare che i valori di route `{ controller = Home, action = Index }` siano sufficienti per generare un URL usando `blog` e che il risultato sia `/blog?action=Index&controller=Home`.

Le route convenzionali dedicate si basano su un comportamento speciale dei valori predefiniti per i quali non esiste un parametro di route corrispondente che impedisce alla route di essere troppo "greedy" con la generazione degli URL. In questo caso i valori predefiniti sono `{ controller = Blog, action = Article }` e né `controller` né `action` vengono visualizzati come parametri di route. Quando il routing esegue la generazione di URL, i valori specificati devono corrispondere ai valori predefiniti. La generazione di URL che usa `blog` avrà esito negativo perché i valori `{ controller = Home, action = Index }` non corrispondono a `{ controller = Blog, action = Article }`. Il routing quindi esegue il fallback per provare `default`, che ha esito positivo.

<a name="routing-areas-ref-label"></a>

## <a name="areas"></a>Aree

Le [aree](areas.md) sono una funzionalità di MVC che consente di organizzare le funzioni correlate in un gruppo come spazio dei nomi di routing separato (per le azioni del controller) e struttura di cartelle (per le visualizzazioni). L'uso delle aree consente a un'applicazione di usare più controller con lo stesso nome, purché abbiano *aree* diverse. Usando le aree si crea una gerarchia per il routing aggiungendo un altro parametro di route, `area` a `controller` e `action`. Questa sezione illustra il modo in cui il routing interagisce con le aree. Vedere [Aree](areas.md) per informazioni dettagliate sull'uso delle aree con le visualizzazioni.

Nell'esempio seguente MVC viene configurato in modo da usare la route convenzionale predefinita e una *route di area* per un'area denominata `Blog`:

[!code-csharp[](routing/samples/3.x/AreasRouting/Startup.cs?name=snippet1)]

Nella corrispondenza di un percorso URL come `/Manage/Users/AddUser` la prima route produrrà i valori di route `{ area = Blog, controller = Users, action = AddUser }`. Il valore di route `area` è generato da un valore predefinito per `area`, infatti la route creata da `MapAreaRoute` equivale alla seguente:

[!code-csharp[](routing/samples/3.x/AreasRouting/Startup.cs?name=snippet2)]

`MapAreaRoute` crea una route che usa sia un valore predefinito che un vincolo per `area` usando il nome di area specificato, in questo caso `Blog`. Il valore predefinito assicura che la route generi sempre `{ area = Blog, ... }`, il vincolo richiede il valore `{ area = Blog, ... }` per la generazione di URL.

> [!TIP]
> Il routing convenzionale dipende dall'ordinamento. In generale, le route con aree devono essere posizionate prima delle altre nella tabella di route poiché sono più specifiche rispetto alle route senza un'area.

Se si usa l'esempio precedente, i valori di route corrispondono all'azione seguente:

[!code-csharp[](routing/samples/3.x/AreasRouting/Areas/Blog/Controllers/UsersController.cs)]

`AreaAttribute` è ciò che denota un controller come parte di un'area, questo controller in particolare si trova nell'area `Blog`. I controller senza un attributo `[Area]` non sono membri di alcuna area e **non** corrispondono quando il valore di route `area` viene specificato dal routing. Nell'esempio seguente solo il primo controller indicato può corrispondere ai valori di route `{ area = Blog, controller = Users, action = AddUser }`.

[!code-csharp[](routing/samples/3.x/AreasRouting/Areas/Blog/Controllers/UsersController.cs)]

[!code-csharp[](routing/samples/3.x/AreasRouting/Areas/Zebra/Controllers/UsersController.cs)]

[!code-csharp[](routing/samples/3.x/AreasRouting/Controllers/UsersController.cs)]

> [!NOTE]
> Lo spazio dei nomi di ogni controller viene indicato qui per motivi di completezza, per evitare conflitti di denominazione nei controller e la generazione di errori del compilatore. Gli spazi dei nomi delle classi non hanno effetto sul routing di MVC.

I primi due controller sono membri di aree e corrispondono solo quando viene specificato il relativo nome di area dal valore di route `area`. Il terzo controller non è un membro di un'area e può corrispondere solo quando non vengono specificati valori per `area` dal routing.

> [!NOTE]
> In termini di corrispondenza con *nessun valore*, l'assenza del valore `area` è come se il valore per `area` fosse Null o la stringa vuota.

Quando si esegue un'azione all'interno di un'area, il valore di route per `area` sarà disponibile come *valore di ambiente* per il routing da usare per la generazione di URL. Ciò significa che per impostazione predefinita le aree funzionano *temporaneamente* per la generazione di URL, come illustrato nell'esempio seguente.
[!code-csharp[](routing/samples/3.x/AreasRouting/Startup.cs?name=snippet3)]

[!code-csharp[](routing/samples/3.x/AreasRouting/Areas/Duck/Controllers/UsersController.cs)]

<a name="iactionconstraint-ref-label"></a>

## <a name="understanding-iactionconstraint"></a>Informazioni su IActionConstraint

> [!NOTE]
> Questa sezione offre un'analisi approfondita degli elementi interni del framework e del modo in cui MVC sceglie un'azione da eseguire. Un'applicazione tipica non richiede un oggetto `IActionConstraint` personalizzato

Probabilmente l'utente ha già usato `IActionConstraint` anche se non ha familiarità con l'interfaccia. L'attributo `[HttpGet]` e gli attributi `[Http-VERB]` dello stesso tipo implementano `IActionConstraint` per limitare l'esecuzione di un metodo di azione.

```csharp
public class ProductsController : Controller
{
    [HttpGet]
    public IActionResult Edit() { }

    public IActionResult Edit(...) { }
}
```

Se si usa la route convenzionale predefinita, il percorso URL `/Products/Edit` produce i valori `{ controller = Products, action = Edit }`, che corrispondono a **entrambe** le azioni illustrate qui. Nella terminologia di `IActionConstraint` si dice che entrambe le azioni sono considerate candidati poiché corrispondono entrambe ai dati della route.

Quando viene eseguito `HttpGetAttribute` indica che *Edit()* è una corrispondenza per *GET* e non è una corrispondenza per qualsiasi altro verbo HTTP. L'azione `Edit(...)` non ha tutti i vincoli definiti e quindi corrisponde a qualsiasi verbo HTTP. Presupponendo l'uso di un oggetto `POST`, corrisponde solo `Edit(...)`. Per un oggetto `GET` possono invece corrispondere entrambe le azioni, tuttavia un'azione con `IActionConstraint` viene sempre considerata *migliore* rispetto a un'azione senza tale oggetto. Quindi, dal momento che `Edit()` ha `[HttpGet]` è considerata più specifica e verrà selezionata se entrambe le azioni possono corrispondere.

Concettualmente, `IActionConstraint` è una forma di *overload*, ma anziché sostituire i metodi con lo stesso nome, supporta l'overload tra le azioni che corrispondono allo stesso URL. Anche il routing con attributi usa `IActionConstraint` e può accadere che azioni di controller diversi vengano entrambe considerate candidati.

<a name="iactionconstraint-impl-ref-label"></a>

### <a name="implementing-iactionconstraint"></a>Implementazione di IActionConstraint

Il modo più semplice di implementare un oggetto `IActionConstraint` è creare una classe derivata da `System.Attribute` e inserirla nelle azioni e nel controller. MVC rileverà automaticamente tutti gli oggetti `IActionConstraint` applicati come attributi. È possibile usare il modello di applicazione per applicare i vincoli e questo è probabilmente l'approccio più flessibile, poiché consente di metaprogrammare le modalità di applicazione.

Nell'esempio seguente un vincolo sceglie un'azione in base a un *codice paese* dei dati della route. Vedere l'[esempio completo su GitHub](https://github.com/aspnet/Entropy/blob/master/samples/Mvc.ActionConstraintSample.Web/CountrySpecificAttribute.cs).

```csharp
public class CountrySpecificAttribute : Attribute, IActionConstraint
{
    private readonly string _countryCode;

    public CountrySpecificAttribute(string countryCode)
    {
        _countryCode = countryCode;
    }

    public int Order
    {
        get
        {
            return 0;
        }
    }

    public bool Accept(ActionConstraintContext context)
    {
        return string.Equals(
            context.RouteContext.RouteData.Values["country"].ToString(),
            _countryCode,
            StringComparison.OrdinalIgnoreCase);
    }
}
```

L'utente è responsabile dell'implementazione del metodo `Accept` e della scelta di un "ordine" per il vincolo da eseguire. In questo caso il metodo `Accept` restituisce `true` per indicare che l'azione è una corrispondenza quando il valore di route `country` corrisponde. Questo comportamento è diverso da quello di `RouteValueAttribute` poiché viene consentito il fallback a un'azione senza attributi. L'esempio indica che se si definisce un'azione `en-US`, per il codice paese, ad esempio `fr-FR`, viene eseguito il fallback a un controller più generico a cui non è applicato `[CountrySpecific(...)]`.

La proprietà `Order` decide a quale *fase* appartiene il vincolo. I vincoli relativi alle azioni vengono eseguiti in gruppi basati su `Order`. Ad esempio, tutti gli attributi del metodo HTTP specificati dal framework usano lo stesso valore `Order` in modo da essere eseguiti nella stessa fase. È possibile usare un numero illimitato di fasi per implementare i criteri necessari.

> [!TIP]
> Quando si sceglie un valore per `Order` considerare se il vincolo dovrà essere applicato o meno prima dei metodi HTTP. I numeri più bassi vengono eseguiti per primi.

::: moniker-end
