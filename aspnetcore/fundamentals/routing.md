---
title: Routing in ASP.NET Core
author: rick-anderson
description: Scopri in che modo ASP.NET Core routing è responsabile della corrispondenza tra le richieste HTTP e l'invio a endpoint eseguibili.
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 4/1/2020
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
uid: fundamentals/routing
ms.openlocfilehash: 0ce89d2dee3fb2054655c003daddfda2ffa52696
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102587282"
---
# <a name="routing-in-aspnet-core"></a>Routing in ASP.NET Core

Di [Ryan Nowak](https://github.com/rynowak), [Kirk Larkin](https://twitter.com/serpent5)e [Rick Anderson](https://twitter.com/RickAndMSFT)

::: moniker range=">= aspnetcore-3.0"

Il routing è responsabile della corrispondenza delle richieste HTTP in ingresso e dell'invio di tali richieste agli endpoint eseguibili dell'app. Gli [endpoint](#endpoint) sono le unità dell'app del codice di gestione delle richieste eseguibile. Gli endpoint vengono definiti nell'app e configurati all'avvio dell'app. Il processo di corrispondenza dell'endpoint può estrarre valori dall'URL della richiesta e fornire tali valori per l'elaborazione della richiesta. Usando le informazioni sull'endpoint dall'app, il routing è anche in grado di generare URL con mapping agli endpoint.

Le app possono configurare il routing tramite:

- Controllers
- Razor Pagine
- SignalR
- Servizi gRPC
- [Middleware](xref:fundamentals/middleware/index) abilitato per gli endpoint, ad esempio i [controlli di integrità](xref:host-and-deploy/health-checks).
- Delegati e espressioni lambda registrate con il routing.

In questo documento vengono illustrati i dettagli di basso livello del routing ASP.NET Core. Per informazioni sulla configurazione del routing:

* Per i controller, vedere <xref:mvc/controllers/routing> .
* Per le Razor convenzioni di pagine, vedere <xref:razor-pages/razor-pages-conventions> .

Il sistema di routing degli endpoint descritto in questo documento si applica a ASP.NET Core 3,0 e versioni successive. Per informazioni sul sistema di routing precedente basato su <xref:Microsoft.AspNetCore.Routing.IRouter> , selezionare la versione ASP.NET Core 2,1 usando uno degli approcci seguenti:

* Selettore di versione per una versione precedente.
* Selezionare [ASP.NET Core 2,1 routing](?view=aspnetcore-2.1).

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/routing/samples/3.x) ([procedura per il download](xref:index#how-to-download-a-sample))

Gli esempi di download per questo documento sono abilitati da una `Startup` classe specifica. Per eseguire un esempio specifico, modificare *Program.cs* per chiamare la `Startup` classe desiderata.

## <a name="routing-basics"></a>Nozioni fondamentali sul routing

Tutti i modelli di ASP.NET Core includono il routing nel codice generato. Il routing viene registrato nella pipeline [middleware](xref:fundamentals/middleware/index) in `Startup.Configure` .

Il codice seguente illustra un esempio di routing di base:

[!code-csharp[](routing/samples/3.x/RoutingSample/Startup.cs?name=snippet&highlight=8,10)]

Il routing utilizza una coppia di middleware, registrata da <xref:Microsoft.AspNetCore.Builder.EndpointRoutingApplicationBuilderExtensions.UseRouting*> e <xref:Microsoft.AspNetCore.Builder.EndpointRoutingApplicationBuilderExtensions.UseEndpoints*> :

* `UseRouting` aggiunge la corrispondenza della route alla pipeline middleware. Questo middleware esamina il set di endpoint definiti nell'app e seleziona la [migliore corrispondenza](#urlm) in base alla richiesta.
* `UseEndpoints` aggiunge l'esecuzione dell'endpoint alla pipeline middleware. Esegue il delegato associato all'endpoint selezionato.

L'esempio precedente include una singola *Route per l'endpoint di codice* usando il metodo [MapGet](xref:Microsoft.AspNetCore.Builder.EndpointRouteBuilderExtensions.MapGet*) :

* Quando `GET` viene inviata una richiesta HTTP all'URL radice `/` :
  * Il delegato della richiesta visualizzato viene eseguito.
  * `Hello World!` viene scritto nella risposta HTTP. Per impostazione predefinita, l'URL radice `/` è `https://localhost:5001/` .
* Se il metodo di richiesta non è `GET` o se l'URL radice non è `/` , non viene restituita alcuna corrispondenza e viene restituito un http 404.

### <a name="endpoint"></a>Endpoint

<a name="endpoint"></a>

Il `MapGet` metodo viene utilizzato per definire un **endpoint**. Un endpoint è un elemento che può essere:

* Selezionato, associando l'URL e il metodo HTTP.
* Eseguita eseguendo il delegato.

Gli endpoint che possono essere confrontati ed eseguiti dall'app sono configurati in `UseEndpoints` . Ad esempio, <xref:Microsoft.AspNetCore.Builder.EndpointRouteBuilderExtensions.MapGet*> , <xref:Microsoft.AspNetCore.Builder.EndpointRouteBuilderExtensions.MapPost*> e [metodi simili](xref:Microsoft.AspNetCore.Builder.EndpointRouteBuilderExtensions) connettono i delegati di richiesta al sistema di routing.
Per connettere ASP.NET Core funzionalità del Framework al sistema di routing, è possibile utilizzare metodi aggiuntivi:
- [Mappare Razor pagine per le Razor pagine](xref:Microsoft.AspNetCore.Builder.RazorPagesEndpointRouteBuilderExtensions.MapRazorPages*)
- [MapControllers per i controller](xref:Microsoft.AspNetCore.Builder.ControllerEndpointRouteBuilderExtensions.MapControllers*)
- [MapHub \<THub> per SignalR](xref:Microsoft.AspNetCore.SignalR.HubRouteBuilder.MapHub*) 
- [MapGrpcService \<TService> per gRPC](xref:grpc/aspnetcore)

Nell'esempio seguente viene illustrato il routing con un modello di route più sofisticato:

[!code-csharp[](routing/samples/3.x/RoutingSample/RouteTemplateStartup.cs?name=snippet)]

La stringa `/hello/{name:alpha}` è un **modello di route**. Viene usato per configurare la corrispondenza con l'endpoint. In questo caso, il modello corrisponde a:

* Un URL come `/hello/Ryan`
* Qualsiasi percorso URL che inizia con `/hello/` seguito da una sequenza di caratteri alfabetici.  `:alpha` applica un vincolo di route che corrisponde solo ai caratteri alfabetici. I [vincoli di route](#route-constraint-reference) vengono illustrati più avanti in questo documento.

Il secondo segmento del percorso URL, `{name:alpha}` :

* Viene associato al `name` parametro.
* Viene acquisito e archiviato in [HttpRequest. routeValues](xref:Microsoft.AspNetCore.Http.HttpRequest.RouteValues*).

Il sistema di routing degli endpoint descritto in questo documento è nuovo a partire da ASP.NET Core 3,0. Tuttavia, tutte le versioni di ASP.NET Core supportano lo stesso set di funzionalità del modello di route e vincoli di route.

Nell'esempio seguente viene illustrato il routing con i [controlli di integrità](xref:host-and-deploy/health-checks) e l'autorizzazione:

[!code-csharp[](routing/samples/3.x/RoutingSample/AuthorizationStartup.cs?name=snippet)]

[!INCLUDE[request localized comments](~/includes/code-comments-loc.md)]

Nell'esempio precedente viene illustrato come:

* Il middleware di autorizzazione può essere usato con il routing.
* Gli endpoint possono essere usati per configurare il comportamento di autorizzazione.

La <xref:Microsoft.AspNetCore.Builder.HealthCheckEndpointRouteBuilderExtensions.MapHealthChecks*> chiamata aggiunge un endpoint di controllo integrità. Il concatenamento <xref:Microsoft.AspNetCore.Builder.AuthorizationEndpointConventionBuilderExtensions.RequireAuthorization*> a questa chiamata collega un criterio di autorizzazione all'endpoint.

Chiamare <xref:Microsoft.AspNetCore.Builder.AuthAppBuilderExtensions.UseAuthentication*> e <xref:Microsoft.AspNetCore.Builder.AuthorizationAppBuilderExtensions.UseAuthorization*> aggiungere il middleware di autenticazione e autorizzazione. Questi middleware sono posizionati tra <xref:Microsoft.AspNetCore.Builder.EndpointRoutingApplicationBuilderExtensions.UseRouting*> e in `UseEndpoints` modo da poter:

* Consente di visualizzare l'endpoint selezionato da `UseRouting` .
* Applicare un criterio di autorizzazione prima <xref:Microsoft.AspNetCore.Builder.EndpointRoutingApplicationBuilderExtensions.UseEndpoints*> dell'invio all'endpoint.

<a name="metadata"></a>

### <a name="endpoint-metadata"></a>Metadati dell'endpoint

Nell'esempio precedente sono presenti due endpoint, ma solo l'endpoint di controllo integrità dispone di un criterio di autorizzazione collegato. Se la richiesta corrisponde all'endpoint di controllo integrità, `/healthz` viene eseguito un controllo di autorizzazione. Ciò dimostra che gli endpoint possono avere dati aggiuntivi collegati. Questi dati aggiuntivi sono detti **metadati** dell'endpoint:

* I metadati possono essere elaborati dal middleware compatibile con il routing.
* I metadati possono essere di qualsiasi tipo .NET.

## <a name="routing-concepts"></a>Concetti di routing

Il sistema di routing si basa sulla pipeline middleware aggiungendo il concetto di **endpoint** potente. Gli endpoint rappresentano unità della funzionalità dell'app che sono diverse l'una dall'altra in termini di routing, autorizzazione e numero di sistemi ASP.NET Core.

<a name="endpoint"></a>

### <a name="aspnet-core-endpoint-definition"></a>Definizione di endpoint ASP.NET Core

Un endpoint ASP.NET Core è:

* Eseguibile: dispone di un <xref:Microsoft.AspNetCore.Http.Endpoint.RequestDelegate> .
* Extensible: include una raccolta di [metadati](xref:Microsoft.AspNetCore.Http.Endpoint.Metadata*) .
* Selectable: dispone facoltativamente [delle informazioni di routing](xref:Microsoft.AspNetCore.Routing.RouteEndpoint.RoutePattern*).
* Enumerable: la raccolta di endpoint può essere elencata recuperando <xref:Microsoft.AspNetCore.Routing.EndpointDataSource> da [di](xref:fundamentals/dependency-injection).

Il codice seguente illustra come recuperare e controllare l'endpoint che corrisponde alla richiesta corrente:

[!code-csharp[](routing/samples/3.x/RoutingSample/EndpointInspectorStartup.cs?name=snippet)]

L'endpoint, se selezionato, può essere recuperato da `HttpContext` . Le proprietà possono essere controllate. Gli oggetti endpoint non sono modificabili e non possono essere modificati dopo la creazione. Il tipo più comune di endpoint è <xref:Microsoft.AspNetCore.Routing.RouteEndpoint> . `RouteEndpoint` include informazioni che ne consentono la selezione da parte del sistema di routing.

Nel codice precedente, [app. Usare](xref:Microsoft.AspNetCore.Builder.UseExtensions.Use*) configura un [middleware](xref:fundamentals/middleware/index)inline.

<a name="mt"></a>

Il codice seguente mostra che, a seconda della posizione `app.Use` in cui viene chiamato nella pipeline, potrebbe non essere presente un endpoint:

[!code-csharp[](routing/samples/3.x/RoutingSample/MiddlewareFlowStartup.cs?name=snippet)]

Questo esempio precedente aggiunge `Console.WriteLine` istruzioni che visualizzano se è stato selezionato un endpoint. Per maggiore chiarezza, l'esempio assegna un nome visualizzato all'endpoint fornito `/` .

L'esecuzione di questo codice con un URL di `/` Visualizza:

```txt
1. Endpoint: (null)
2. Endpoint: Hello
3. Endpoint: Hello
```

L'esecuzione di questo codice con qualsiasi altro URL consente di visualizzare:

```txt
1. Endpoint: (null)
2. Endpoint: (null)
4. Endpoint: (null)
```

Questo output dimostra che:

* L'endpoint è sempre null prima della `UseRouting` chiamata a.
* Se viene trovata una corrispondenza, l'endpoint è diverso da null tra `UseRouting` e <xref:Microsoft.AspNetCore.Builder.EndpointRoutingApplicationBuilderExtensions.UseEndpoints*> .
* Il `UseEndpoints` middleware è **terminale** quando viene trovata una corrispondenza. Il [middleware terminale](#tm) è definito più avanti in questo documento.
* Il middleware dopo l' `UseEndpoints` esecuzione viene eseguito solo quando non viene trovata alcuna corrispondenza.

Il `UseRouting` middleware usa il metodo [seendpoint](xref:Microsoft.AspNetCore.Http.EndpointHttpContextExtensions.SetEndpoint*) per aggiungere l'endpoint al contesto corrente. È possibile sostituire il `UseRouting` middleware con la logica personalizzata e ottenere comunque i vantaggi derivanti dall'uso degli endpoint. Gli endpoint sono una primitiva di basso livello, ad esempio middleware, e non sono associati all'implementazione del routing. Per la maggior parte delle app non è necessario sostituire `UseRouting` con la logica personalizzata.

Il `UseEndpoints` middleware è progettato per essere usato in combinazione con il `UseRouting` middleware. La logica di base per l'esecuzione di un endpoint non è complessa. Usare <xref:Microsoft.AspNetCore.Http.EndpointHttpContextExtensions.GetEndpoint*> per recuperare l'endpoint e quindi richiamare la relativa <xref:Microsoft.AspNetCore.Http.Endpoint.RequestDelegate> Proprietà.

Il codice seguente illustra come il middleware può influenzare o rispondere al routing:

[!code-csharp[](routing/samples/3.x/RoutingSample/IntegratedMiddlewareStartup.cs?name=snippet)]

Nell'esempio precedente vengono illustrati due concetti importanti:

* Il middleware può essere eseguito prima `UseRouting` di per modificare i dati su cui opera il routing.
    * In genere il middleware visualizzato prima che il routing modifichi una proprietà della richiesta, ad esempio <xref:Microsoft.AspNetCore.Builder.RewriteBuilderExtensions.UseRewriter*> , <xref:Microsoft.AspNetCore.Builder.HttpMethodOverrideExtensions.UseHttpMethodOverride*> o <xref:Microsoft.AspNetCore.Builder.UsePathBaseExtensions.UsePathBase*> .
* Il middleware può essere eseguito tra `UseRouting` e <xref:Microsoft.AspNetCore.Builder.EndpointRoutingApplicationBuilderExtensions.UseEndpoints*> per elaborare i risultati del routing prima dell'esecuzione dell'endpoint.
    * Middleware che esegue tra `UseRouting` e `UseEndpoints` :
      * In genere controlla i metadati per comprendere gli endpoint.
      * Spesso prende decisioni di sicurezza, come fatto da `UseAuthorization` e `UseCors` .
    * La combinazione di middleware e metadati consente la configurazione dei criteri per endpoint.

Il codice precedente mostra un esempio di middleware personalizzato che supporta i criteri per endpoint. Il middleware scrive un *log di controllo* di accesso ai dati sensibili nella console. Il middleware può essere configurato per *controllare* un endpoint con i `AuditPolicyAttribute` metadati. In questo esempio viene illustrato un modello *di consenso esplicito in* cui vengono controllati solo gli endpoint contrassegnati come sensibili. È possibile definire questa logica in senso inverso, ad esempio controllando tutti gli elementi che non sono contrassegnati come sicuri. Il sistema di metadati dell'endpoint è flessibile. Questa logica può essere progettata nel modo più adatto al caso d'uso.

Il codice di esempio precedente ha lo scopo di illustrare i concetti di base degli endpoint. **L'esempio non è destinato all'uso in produzione**. Una versione più completa di un middleware di *log di controllo* :

* Accedere a un file o a un database.
* Includere i dettagli, ad esempio l'utente, l'indirizzo IP, il nome dell'endpoint sensibile e altro ancora.

I metadati dei criteri `AuditPolicyAttribute` di controllo sono definiti come un `Attribute` per un uso più semplice con i Framework basati su classi quali i controller e SignalR . Quando si usa la *Route per il codice*:

* I metadati sono collegati a un'API del generatore.
* I Framework basati su classi includono tutti gli attributi nel metodo e nella classe corrispondenti durante la creazione degli endpoint.

Le procedure consigliate per i tipi di metadati sono definirle come interfacce o attributi. Interfacce e attributi consentono il riutilizzo del codice. Il sistema dei metadati è flessibile e non impone alcuna restrizione.

<a name="tm"></a>

### <a name="comparing-a-terminal-middleware-and-routing"></a>Confronto tra un middleware terminale e il routing

L'esempio di codice seguente contrasta l'uso del middleware con l'uso del routing:

[!code-csharp[](routing/samples/3.x/RoutingSample/TerminalMiddlewareStartup.cs?name=snippet)]

Lo stile del middleware visualizzato con `Approach 1:` è **middleware terminale**. Viene chiamato middleware terminale perché esegue un'operazione di corrispondenza:

* L'operazione di corrispondenza nell'esempio precedente è `Path == "/"` per il middleware e `Path == "/Movie"` per il routing.
* Quando una corrispondenza ha esito positivo, vengono eseguite alcune funzionalità e viene restituito, anziché richiamare il `next` middleware.

Si chiama middleware terminale perché termina la ricerca, esegue alcune funzionalità e quindi restituisce.

Confronto tra un middleware terminale e il routing:
* Entrambi gli approcci consentono di terminare la pipeline di elaborazione:
    * Il middleware termina la pipeline restituendo anziché richiamare `next` .
    * Gli endpoint sono sempre terminali.
* Il middleware terminale consente di posizionare il middleware in una posizione arbitraria nella pipeline:
    * Gli endpoint vengono eseguiti in corrispondenza della posizione di <xref:Microsoft.AspNetCore.Builder.EndpointRoutingApplicationBuilderExtensions.UseEndpoints*> .
* Il middleware terminale consente il codice arbitrario per determinare quando il middleware corrisponde a:
    * Il codice personalizzato per la corrispondenza delle route può essere dettagliato e difficile da scrivere correttamente.
    * Il routing fornisce soluzioni semplici per le app tipiche. La maggior parte delle app non richiede codice personalizzato per la corrispondenza delle route.
* Interfaccia degli endpoint con middleware, ad esempio `UseAuthorization` e `UseCors` .
    * L'uso di un middleware terminale con `UseAuthorization` o `UseCors` richiede l'interazione manuale con il sistema di autorizzazione.

Un [endpoint](#endpoint) definisce entrambi:

* Delegato per l'elaborazione delle richieste.
* Raccolta di metadati arbitrari. I metadati vengono usati per implementare le problematiche trasversali in base ai criteri e alla configurazione collegati a ogni endpoint.

Il middleware terminale può essere uno strumento efficace, ma può richiedere:

* Una quantità significativa di codice e test.
* Integrazione manuale con altri sistemi per raggiungere il livello di flessibilità desiderato.

Prendere in considerazione l'integrazione con routing prima di scrivere un middleware terminale.

Il middleware terminale esistente che si integra con [Map](xref:fundamentals/middleware/index#branch-the-middleware-pipeline) o <xref:Microsoft.AspNetCore.Builder.MapWhenExtensions.MapWhen*> può in genere essere trasformato in un endpoint in grado di riconoscere il routing. [MapHealthChecks](https://github.com/dotnet/AspNetCore/blob/main/src/Middleware/HealthChecks/src/Builder/HealthCheckEndpointRouteBuilderExtensions.cs#L16) illustra il modello per il router-Ware:
* Scrivere un metodo di estensione in <xref:Microsoft.AspNetCore.Routing.IEndpointRouteBuilder> .
* Creare una pipeline middleware annidata usando <xref:Microsoft.AspNetCore.Routing.IEndpointRouteBuilder.CreateApplicationBuilder*> .
* Alleghi il middleware alla nuova pipeline. In questo caso, <xref:Microsoft.AspNetCore.Builder.HealthCheckApplicationBuilderExtensions.UseHealthChecks*>.
* <xref:Microsoft.AspNetCore.Builder.IApplicationBuilder.Build*> pipeline middleware in un oggetto <xref:Microsoft.AspNetCore.Http.RequestDelegate> .
* Chiamare `Map` e fornire la nuova pipeline middleware.
* Restituisce l'oggetto generatore fornito da `Map` dal metodo di estensione.

Il codice seguente illustra l'uso di [MapHealthChecks](xref:host-and-deploy/health-checks):

[!code-csharp[](routing/samples/3.x/RoutingSample/AuthorizationStartup.cs?name=snippet)]

Nell'esempio precedente viene illustrato il motivo per cui la restituzione dell'oggetto generatore è importante. La restituzione dell'oggetto generatore consente allo sviluppatore di app di configurare criteri come l'autorizzazione per l'endpoint. In questo esempio il middleware dei controlli di integrità non ha alcuna integrazione diretta con il sistema di autorizzazione.

Il sistema dei metadati è stato creato in risposta ai problemi riscontrati dagli autori di estendibilità tramite il middleware terminale. È problematico per ogni middleware implementare una propria integrazione con il sistema di autorizzazione.

<a name="urlm"></a>

### <a name="url-matching"></a>Corrispondenza URL

* È il processo mediante il quale il routing corrisponde a una richiesta in ingresso a un [endpoint](#endpoint).
* Si basa sui dati nel percorso e nelle intestazioni URL.
* Può essere esteso per considerare tutti i dati nella richiesta.

Quando viene eseguito un middleware di routing, imposta i `Endpoint` valori di route e su una [funzionalità di richiesta](xref:fundamentals/request-features) nell'oggetto <xref:Microsoft.AspNetCore.Http.HttpContext> dalla richiesta corrente:

* La chiamata a [HttpContext. GetEndPoint](<xref:Microsoft.AspNetCore.Http.EndpointHttpContextExtensions.GetEndpoint*>) Ottiene l'endpoint.
* `HttpRequest.RouteValues` ottiene la raccolta dei valori di route.

Il [middleware](xref:fundamentals/middleware/index) eseguito dopo il middleware di routing può ispezionare l'endpoint ed eseguire un'azione. Un middleware di autorizzazione, ad esempio, può interrogare la raccolta di metadati dell'endpoint per i criteri di autorizzazione. Dopo l'esecuzione di tutto il middleware nella pipeline di elaborazione della richiesta, viene richiamato il delegato dell'endpoint selezionato.

Il sistema di routing nell'endpoint di routing è responsabile di tutte le decisioni relative all'invio. Poiché il middleware applica criteri basati sull'endpoint selezionato, è importante:

* Qualsiasi decisione che può influire sull'invio o sull'applicazione di criteri di sicurezza viene effettuata all'interno del sistema di routing.

> [!WARNING]
> Per la compatibilità con le versioni precedenti, quando Razor viene eseguito un delegato dell'endpoint di un controller o di una pagina, le proprietà di [RouteContext. RouteData](xref:Microsoft.AspNetCore.Routing.RouteContext.RouteData) vengono impostate sui valori appropriati in base all'elaborazione della richiesta eseguita fino a questo momento.
>
> Il `RouteContext` tipo verrà contrassegnato come obsoleto in una versione futura:
>
> * Eseguire la migrazione `RouteData.Values` a `HttpRequest.RouteValues` .
> * Eseguire `RouteData.DataTokens` la migrazione per recuperare [IDataTokensMetadata](xref:Microsoft.AspNetCore.Routing.IDataTokensMetadata) dai metadati dell'endpoint.

La corrispondenza degli URL opera in un set di fasi configurabile. In ogni fase l'output è un set di corrispondenze. Il set di corrispondenze può essere ulteriormente ridotto dalla fase successiva. L'implementazione del routing non garantisce un ordine di elaborazione per gli endpoint corrispondenti. **Tutte le** corrispondenze possibili vengono elaborate in una sola volta. Le fasi corrispondenti dell'URL si verificano nell'ordine seguente. ASP.NET Core:

1. Elabora il percorso URL rispetto al set di endpoint e ai relativi modelli di route, raccogliendo **tutte** le corrispondenze.
1. Accetta l'elenco precedente e rimuove le corrispondenze che non riescono con i vincoli di route applicati.
1. Accetta l'elenco precedente e rimuove le corrispondenze che non superano il set di istanze di [MatcherPolicy](xref:Microsoft.AspNetCore.Routing.MatcherPolicy) .
1. USA [EndpointSelector](xref:Microsoft.AspNetCore.Routing.Matching.EndpointSelector) per prendere una decisione finale dall'elenco precedente.

Per l'elenco degli endpoint viene assegnata la priorità in base a:

* [RouteEndpoint. Order](xref:Microsoft.AspNetCore.Routing.RouteEndpoint.Order*)
* [Precedenza del modello di route](#rtp)

Tutti gli endpoint corrispondenti vengono elaborati in ogni fase fino a quando non <xref:Microsoft.AspNetCore.Routing.Matching.EndpointSelector> viene raggiunto il. `EndpointSelector`È la fase finale. Viene scelto l'endpoint con la priorità più alta dalle corrispondenze come la migliore corrispondenza. Se sono presenti altre corrispondenze con la stessa priorità della corrispondenza migliore, viene generata un'eccezione di corrispondenza ambigua.

La precedenza della route viene calcolata in base a un modello di route **più specifico** a cui viene assegnata una priorità più alta. Si considerino, ad esempio, i modelli `/hello` e `/{message}` :

* Entrambe corrispondono al percorso URL `/hello` .
* `/hello`  è più specifico e pertanto con una priorità più elevata.

In generale, la precedenza della route consente di scegliere la migliore corrispondenza per i tipi di schemi URL usati in pratica. Utilizzare <xref:Microsoft.AspNetCore.Routing.RouteEndpoint.Order> solo quando necessario per evitare ambiguità.

A causa dei tipi di estendibilità offerti dal routing, non è possibile che il sistema di routing calcoli in anticipo le route ambigue. Si consideri, ad esempio, i modelli di route `/{message:alpha}` e `/{message:int}` :

* Il `alpha` vincolo corrisponde solo ai caratteri alfabetici.
* Il `int` vincolo corrisponde solo ai numeri.
* Questi modelli hanno la stessa precedenza di route, ma non esiste un singolo URL corrispondente.
* Se il sistema di routing ha segnalato un errore di ambiguità all'avvio, blocca il caso d'uso valido.

> [!WARNING]
>
> L'ordine delle operazioni all'interno di <xref:Microsoft.AspNetCore.Builder.EndpointRoutingApplicationBuilderExtensions.UseEndpoints*> non influisce sul comportamento del routing, con un'eccezione. <xref:Microsoft.AspNetCore.Builder.ControllerEndpointRouteBuilderExtensions.MapControllerRoute*> e <xref:Microsoft.AspNetCore.Builder.MvcAreaRouteBuilderExtensions.MapAreaRoute*> assegna automaticamente un valore di ordine ai rispettivi endpoint in base all'ordine in cui vengono richiamati. Questo simula il comportamento a lungo termine dei controller senza che il sistema di routing fornisca le stesse garanzie delle implementazioni di routing precedenti.
>
> Nell'implementazione legacy del routing, è possibile implementare l'estendibilità del routing che dipende dall'ordine in cui vengono elaborate le route. Routing degli endpoint in ASP.NET Core 3,0 e versioni successive:
> 
> * Non ha un concetto di route.
> * Non fornisce garanzie di ordinamento. Tutti gli endpoint vengono elaborati contemporaneamente.

<a name="rtp"></a>

### <a name="route-template-precedence-and-endpoint-selection-order"></a>Ordine di precedenza e selezione degli endpoint del modello di route

La [precedenza del modello di route](https://github.com/dotnet/aspnetcore/blob/main/src/Http/Routing/src/Template/RoutePrecedence.cs#L16) è un sistema che assegna a ogni modello di route un valore in base a quanto è specifico. Precedenza modello di route:

* Evita la necessità di modificare l'ordine degli endpoint in casi comuni.
* Tenta di trovare la corrispondenza con le aspettative comuni del comportamento di routing.

Si considerino, ad esempio, i modelli `/Products/List` e `/Products/{id}` . Sarebbe ragionevole presupporre che `/Products/List` sia una corrispondenza migliore rispetto `/Products/{id}` al percorso URL `/Products/List` . Questa operazione funziona perché il segmento letterale `/List` viene considerato con una precedenza migliore rispetto al segmento di parametro `/{id}` .

I dettagli del funzionamento della precedenza sono associati alla modalità di definizione dei modelli di route:

* I modelli con più segmenti sono considerati più specifici.
* Un segmento con testo letterale viene considerato più specifico rispetto a un segmento di parametro.
* Un segmento di parametro con un vincolo è considerato più specifico di uno senza.
* Un segmento complesso è considerato specifico come un segmento di parametro con un vincolo.
* I parametri catch-all sono i meno specifici. Per informazioni importanti sulle route catch-all, vedere **catch-all** nel [riferimento del modello di route](#rtr) .

Per un riferimento di valori esatti, vedere il [codice sorgente su GitHub](https://github.com/dotnet/aspnetcore/blob/main/src/Http/Routing/src/Template/RoutePrecedence.cs#L189) .

<a name="lg"></a>

### <a name="url-generation-concepts"></a>Concetti relativi alla generazione di URL

Generazione URL:

* È il processo mediante il quale il routing può creare un percorso URL basato su un set di valori di route.
* Consente una separazione logica tra gli endpoint e gli URL che vi accedono.

Il routing degli endpoint include l' <xref:Microsoft.AspNetCore.Routing.LinkGenerator> API. `LinkGenerator` è un servizio singleton disponibile in [di](xref:fundamentals/dependency-injection). L' `LinkGenerator` API può essere usata al di fuori del contesto di una richiesta in esecuzione. [MVC. IUrlHelper](xref:Microsoft.AspNetCore.Mvc.IUrlHelper) e gli scenari basati su <xref:Microsoft.AspNetCore.Mvc.IUrlHelper> , ad esempio gli [Helper Tag](xref:mvc/views/tag-helpers/intro), gli helper HTML e [i risultati dell'azione](xref:mvc/controllers/actions), usano l' `LinkGenerator` API internamente per fornire funzionalità di generazione dei collegamenti.

Il generatore di collegamenti si basa sui concetti di **indirizzo** e di **schemi di indirizzi**. Lo schema di indirizzi consente di determinare gli endpoint che devono essere considerati per la generazione dei collegamenti. Ad esempio, gli scenari relativi al nome della route e ai valori di route molti utenti hanno familiarità con i controller e le Razor pagine vengono implementati come uno schema di indirizzi.

Il generatore di collegamenti può collegarsi a controller e Razor pagine tramite i seguenti metodi di estensione:

* <xref:Microsoft.AspNetCore.Routing.ControllerLinkGeneratorExtensions.GetPathByAction*>
* <xref:Microsoft.AspNetCore.Routing.ControllerLinkGeneratorExtensions.GetUriByAction*>
* <xref:Microsoft.AspNetCore.Routing.PageLinkGeneratorExtensions.GetPathByPage*>
* <xref:Microsoft.AspNetCore.Routing.PageLinkGeneratorExtensions.GetUriByPage*>

Gli overload di questi metodi accettano argomenti che includono `HttpContext` . Questi metodi sono equivalenti dal punto di vista funzionale a [URL. Action](xref:System.Web.Mvc.UrlHelper.Action*) e [URL. Page](xref:Microsoft.AspNetCore.Mvc.UrlHelperExtensions.Page*), ma offrono flessibilità e opzioni aggiuntive.

I `GetPath*` metodi sono molto simili a `Url.Action` e `Url.Page` , perché generano un URI contenente un percorso assoluto. I metodi `GetUri*` generano sempre un URI assoluto contenente uno schema e un host. I metodi che accettano `HttpContext` generano un URI nel contesto della richiesta in esecuzione. I valori della route di [ambiente](#ambient) , il percorso di base URL, lo schema e l'host dalla richiesta in esecuzione vengono usati a meno che non vengano sottoposti a override.

<xref:Microsoft.AspNetCore.Routing.LinkGenerator> viene chiamato con un indirizzo. La generazione di un URI viene eseguita in due passaggi:

1. Un indirizzo viene associato a un elenco di endpoint che corrispondono all'indirizzo.
1. <xref:Microsoft.AspNetCore.Routing.RouteEndpoint.RoutePattern> di ogni endpoint viene valutato fino a quando non viene individuato un formato di route che corrisponde ai valori specificati. L'output risultante viene unito alle altre parti dell'URI specificate nel generatore di collegamenti e restituito.

I metodi forniti da <xref:Microsoft.AspNetCore.Routing.LinkGenerator> supportano le funzionalità di generazione di collegamenti standard per tutti i tipi di indirizzi. Il modo più pratico per usare il generatore di collegamenti è tramite i metodi di estensione che eseguono operazioni per un tipo di indirizzo specifico:

| Metodo di estensione | Descrizione |
| ---------------- | ----------- |
| <xref:Microsoft.AspNetCore.Routing.LinkGenerator.GetPathByAddress*> | Genera un URI con un percorso assoluto in base ai valori specificati. |
| <xref:Microsoft.AspNetCore.Routing.LinkGenerator.GetUriByAddress*> | Genera un URI assoluto in base ai valori specificati.             |

> [!WARNING]
> Prestare attenzione alle implicazioni seguenti della chiamata ai metodi <xref:Microsoft.AspNetCore.Routing.LinkGenerator>:
>
> * Usare i metodi di estensione `GetUri*` con cautela in una configurazione di app che non convalida l'intestazione `Host` delle richieste in ingresso. Se l' `Host` intestazione delle richieste in ingresso non viene convalidata, l'input della richiesta non attendibile può essere restituito al client negli URI in una vista o in una pagina. È consigliabile che in tutte le app di produzione il server sia configurato per la convalida dell'intestazione `Host` rispetto ai valori validi noti.
>
> * Usare <xref:Microsoft.AspNetCore.Routing.LinkGenerator> con cautela nel middleware in associazione a `Map` o `MapWhen`. `Map*` modifica il percorso di base della richiesta in esecuzione, che ha effetto sull'output della generazione di collegamenti. Tutte le API <xref:Microsoft.AspNetCore.Routing.LinkGenerator> consentono di specificare un percorso di base. Specificare un percorso di base vuoto per annullare l' `Map*` effetto sulla generazione del collegamento.

### <a name="middleware-example"></a>Esempio di middleware

Nell'esempio seguente, un middleware usa l' <xref:Microsoft.AspNetCore.Routing.LinkGenerator> API per creare un collegamento a un metodo di azione che elenca i prodotti dell'archivio. Usando il generatore di collegamenti inserendolo in una classe e chiamando `GenerateLink` è disponibile per qualsiasi classe in un'app:

[!code-csharp[](routing/samples/3.x/RoutingSample/Middleware/ProductsLinkMiddleware.cs?name=snippet)]

<a name="rtr"></a>

## <a name="route-template-reference"></a>Riferimento per il modello di route

I token all'interno `{}` di definiscono i parametri di route associati se la route viene confrontata. È possibile definire più di un parametro di route in un segmento di route, ma i parametri di route devono essere separati da un valore letterale. Ad esempio, `{controller=Home}{action=Index}` non è una route valida perché non è presente un valore letterale tra `{controller}` e `{action}`.  I parametri di route devono avere un nome e possono avere attributi aggiuntivi specificati.

Il testo letterale diverso dai parametri di route, ad esempio `{id}`, e il separatore di percorso `/` devono corrispondere al testo nell'URL. La corrispondenza del testo non fa distinzione tra maiuscole e minuscole e si basa sulla rappresentazione decodificata del percorso dell'URL. Per trovare la corrispondenza con un delimitatore di parametro di route letterale `{` o, eseguire l' `}` escape del delimitatore ripetendo il carattere. Ad esempio `{{` o `}}` .

Asterisco `*` o doppio asterisco `**` :

* Può essere usato come prefisso per un parametro di route per l'associazione al resto dell'URI.
* Sono detti parametri **catch-all** . Ad esempio, `blog/{**slug}`:
  * Corrisponde a qualsiasi URI che inizia con `/blog` e ha un valore che lo segue.
  * Il valore seguente `/blog` viene assegnato al valore di route [Slug](https://developer.mozilla.org/docs/Glossary/Slug) .

[!INCLUDE[](~/includes/catchall.md)]

I parametri catch-all possono anche corrispondere alla stringa vuota.

Il parametro catch-all esegue l'escape dei caratteri appropriati quando la route viene usata per generare un URL, inclusi i caratteri separatori di percorso `/` . Ad esempio, la route `foo/{*path}` con i valori di route `{ path = "my/path" }` genera `foo/my%2Fpath`. Si noti la barra di escape. Per eseguire il round trip dei caratteri di separatore di percorso, usare il prefisso del parametro di route `**`. La route `foo/{**path}` con `{ path = "my/path" }` genera `foo/my/path`.

I modelli di URL che tentano di acquisire un nome file con un'estensione facoltativa hanno considerazioni aggiuntive. Considerare ad esempio il modello `files/{filename}.{ext?}`. Se esistono i valori per `filename` e `ext`, vengono popolati entrambi i valori. Se `filename` nell'URL è presente solo un valore per, la route corrisponde perché l'oggetto finale `.` è facoltativo. Gli URL seguenti corrispondono a questa route:

* `/files/myFile.txt`
* `/files/myFile`

I parametri di route possono avere **valori predefiniti**, definiti specificando il valore predefinito dopo il nome del parametro, separato da un segno di uguale (`=`). Ad esempio, `{controller=Home}` definisce `Home` come valore predefinito per `controller`. Il valore predefinito viene usato se nell'URL non è presente alcun valore per il parametro. I parametri di route vengono resi facoltativi aggiungendo un punto interrogativo ( `?` ) alla fine del nome del parametro. Ad esempio: `id?`. La differenza tra i valori facoltativi e i parametri di route predefiniti è la seguente:

* Un parametro di route con un valore predefinito produce sempre un valore.
* Un parametro facoltativo ha un valore solo quando un valore viene fornito dall'URL della richiesta.

I parametri di route possono presentare dei vincoli che devono corrispondere al valore di route associato dall'URL. `:`L'aggiunta di un nome di vincolo e dopo il nome del parametro di route specifica un vincolo inline in un parametro di route. Se il vincolo richiede argomenti, vengono racchiusi tra parentesi `(...)` dopo il nome del vincolo. È possibile specificare più *vincoli inline* aggiungendo un altro `:` nome di vincolo e.

Il nome del vincolo e gli argomenti vengono passati al servizio <xref:Microsoft.AspNetCore.Routing.IInlineConstraintResolver> per creare un'istanza di <xref:Microsoft.AspNetCore.Routing.IRouteConstraint> da usare nell'elaborazione dell'URL. Il modello di route `blog/{article:minlength(10)}` specifica ad esempio un vincolo `minlength` con l'argomento `10`. Per altre informazioni sui vincoli di route e per un elenco dei vincoli specificati dal framework, vedere la sezione [Riferimento per i vincoli di route](#route-constraint-reference).

I parametri di route possono avere anche trasformatori di parametro. I trasformatori di parametro trasformano il valore di un parametro durante la generazione di collegamenti, azioni e pagine corrispondenti agli URL. Analogamente ai vincoli, i trasformatori di parametro possono essere aggiunti inline a un parametro di route mediante l'aggiunta di un nome del trasformatore `:` e dopo il nome del parametro di route. Ad esempio, il modello di route `blog/{article:slugify}` specifica un trasformatore `slugify`. Per altre informazioni sui trasformatori di parametro, vedere la sezione [Riferimento ai trasformatori di parametro](#parameter-transformer-reference).

La tabella seguente illustra i modelli di route di esempio e il relativo comportamento:

| Modello di route                           | URI corrispondente di esempio    | L'URI della richiesta&hellip;                                                    |
| ---------------------------------------- | ----------------------- | -------------------------------------------------------------------------- |
| `hello`                                  | `/hello`                | Verifica la corrispondenza solo del singolo percorso `/hello`.                                     |
| `{Page=Home}`                            | `/`                     | Verifica la corrispondenza e imposta `Page` su `Home`.                                         |
| `{Page=Home}`                            | `/Contact`              | Verifica la corrispondenza e imposta `Page` su `Contact`.                                      |
| `{controller}/{action}/{id?}`            | `/Products/List`        | Esegue il mapping al controller `Products` e all'azione `List`.                       |
| `{controller}/{action}/{id?}`            | `/Products/Details/123` | Esegue il mapping al `Products` controller e all'  `Details` azione con `id` impostato su 123. |
| `{controller=Home}/{action=Index}/{id?}` | `/`                     | Esegue il mapping al `Home` controller e al `Index` metodo. `id` viene ignorato.        |
| `{controller=Home}/{action=Index}/{id?}` | `/Products`         | Esegue il mapping al `Products` controller e al `Index` metodo. `id` viene ignorato.        |

L'uso di un modello è in genere l'approccio più semplice al routing. I vincoli e le impostazioni predefinite possono essere specificati anche all'esterno del modello di route.

### <a name="complex-segments"></a>Segmenti complessi

I segmenti complessi vengono elaborati mediante la corrispondenza di delimitatori letterali da destra a sinistra in modo [non greedy](#greedy) . Ad esempio, `[Route("/a{b}c{d}")]` è un segmento complesso.
I segmenti complessi funzionano in un modo specifico che devono essere riconosciuti per utilizzarli correttamente. Nell'esempio riportato in questa sezione viene illustrato il motivo per cui i segmenti complessi funzionano correttamente solo quando il testo del delimitatore non viene visualizzato all'interno dei valori del parametro. Per i casi più complessi, è necessario utilizzare un' [espressione regolare](/dotnet/standard/base-types/regular-expressions) e quindi estrarre manualmente i valori.

[!INCLUDE[](~/includes/regex.md)]

Questo è un riepilogo dei passaggi eseguiti dal routing con il modello `/a{b}c{d}` e il percorso URL `/abcd` . `|`Viene usato per visualizzare il funzionamento dell'algoritmo:

* Il primo valore letterale, da destra a sinistra, è `c` . Quindi `/abcd` , la ricerca viene eseguita da destra e trova `/ab|c|d` .
* Tutto a destra ( `d` ) viene ora associato al parametro di route `{d}` .
* Il valore letterale successivo, da destra a sinistra, è `a` . Quindi `/ab|c|d` , viene eseguita una ricerca a partire dal punto in cui è stato interrotto, quindi `a` viene trovato `/|a|b|c|d` .
* Il valore a destra ( `b` ) viene ora associato al parametro di route `{b}` .
* Non esiste alcun testo rimanente e nessun modello di route rimanente, quindi si tratta di una corrispondenza.

Di seguito è riportato un esempio di un caso negativo che usa lo stesso modello `/a{b}c{d}` e il percorso URL `/aabcd` . `|`Viene utilizzato per facilitare la visualizzazione del funzionamento dell'algoritmo. Questo caso non corrisponde a una corrispondenza, descritta dallo stesso algoritmo:
* Il primo valore letterale, da destra a sinistra, è `c` . Quindi `/aabcd` , la ricerca viene eseguita da destra e trova `/aab|c|d` .
* Tutto a destra ( `d` ) viene ora associato al parametro di route `{d}` .
* Il valore letterale successivo, da destra a sinistra, è `a` . Quindi `/aab|c|d` , viene eseguita una ricerca a partire dal punto in cui è stato interrotto, quindi `a` viene trovato `/a|a|b|c|d` .
* Il valore a destra ( `b` ) viene ora associato al parametro di route `{b}` .
* A questo punto è presente un testo rimanente `a` , ma l'algoritmo ha esaurito il modello di route da analizzare, quindi non si tratta di una corrispondenza.

Poiché l'algoritmo di corrispondenza [non è greedy](#greedy):

* Corrisponde alla minima quantità di testo possibile in ogni passaggio.
* Ogni caso in cui il valore del delimitatore viene visualizzato all'interno dei valori del parametro non corrisponde.

Le espressioni regolari forniscono un maggiore controllo sul comportamento corrispondente.

<a name="greedy"></a>

La corrispondenza greedy, conosciuta anche come [Corrispondenza Lazy](https://wikipedia.org/wiki/Regular_expression#Lazy_matching), corrisponde alla stringa più grande possibile. Non greedy corrisponde alla stringa più piccola possibile.

## <a name="route-constraint-reference"></a>Riferimento per i vincoli di route

I vincoli di route vengono eseguiti quando si verifica una corrispondenza nell'URL in ingresso e il percorso URL viene suddiviso in valori di route in formato token. I vincoli di route controllano in genere il valore di route associato tramite il modello di route e fanno una decisione vera o falsa sul fatto che il valore sia accettabile. Alcuni vincoli di route usano i dati all'esterno del valore di route per stabilire se la richiesta può essere instradata. Ad esempio, <xref:Microsoft.AspNetCore.Routing.Constraints.HttpMethodRouteConstraint> può accettare o rifiutare una richiesta in base al relativo verbo HTTP. I vincoli vengono usati nelle richieste del routing e nella generazione di collegamenti.

> [!WARNING]
> Non usare i vincoli per la convalida dell'input. Se vengono usati vincoli per la convalida dell'input, l'input non valido restituisce una `404` risposta non trovata. L'input non valido deve produrre una richiesta non valida `400` con un messaggio di errore appropriato. I vincoli di route vengono usati per evitare ambiguità tra route simili, non per convalidare gli input per una route specifica.

La tabella seguente illustra i vincoli di route di esempio e il comportamento previsto:

| vincolo | Esempio | Esempi di corrispondenza | Note |
| ---------- | ------- | --------------- | ----- |
| `int` | `{id:int}` | `123456789`, `-123456789` | Corrisponde a qualsiasi numero intero |
| `bool` | `{active:bool}` | `true`, `FALSE` | Corrisponde a `true` o `false` . Non fa distinzione tra maiuscole e minuscole. |
| `datetime` | `{dob:datetime}` | `2016-12-31`, `2016-12-31 7:32pm` | Corrisponde a un `DateTime` valore valido della lingua inglese. Vedere l'avviso precedente. |
| `decimal` | `{price:decimal}` | `49.99`, `-1,000.01` | Corrisponde a un `decimal` valore valido della lingua inglese. Vedere l'avviso precedente.|
| `double` | `{weight:double}` | `1.234`, `-1,001.01e8` | Corrisponde a un `double` valore valido della lingua inglese. Vedere l'avviso precedente.|
| `float` | `{weight:float}` | `1.234`, `-1,001.01e8` | Corrisponde a un `float` valore valido della lingua inglese. Vedere l'avviso precedente.|
| `guid` | `{id:guid}` | `CD2C1638-1638-72D5-1638-DEADBEEF1638` | Corrisponde a un valore `Guid` valido |
| `long` | `{ticks:long}` | `123456789`, `-123456789` | Corrisponde a un valore `long` valido |
| `minlength(value)` | `{username:minlength(4)}` | `Rick` | La stringa deve contenere almeno 4 caratteri |
| `maxlength(value)` | `{filename:maxlength(8)}` | `MyFile` | La stringa non deve contenere più di 8 caratteri |
| `length(length)` | `{filename:length(12)}` | `somefile.txt` | La stringa deve contenere esattamente 12 caratteri |
| `length(min,max)` | `{filename:length(8,16)}` | `somefile.txt` | La stringa deve contenere almeno 8 e non più di 16 caratteri |
| `min(value)` | `{age:min(18)}` | `19` | Il valore intero deve essere almeno 18 |
| `max(value)` | `{age:max(120)}` | `91` | Il valore intero non deve essere superiore a 120 |
| `range(min,max)` | `{age:range(18,120)}` | `91` | Il valore intero deve essere almeno 18 ma non più di 120 |
| `alpha` | `{name:alpha}` | `Rick` | La stringa deve essere composta da uno o più caratteri alfabetici `a` - `z` e senza distinzione tra maiuscole e minuscole. |
| `regex(expression)` | `{ssn:regex(^\\d{{3}}-\\d{{2}}-\\d{{4}}$)}` | `123-45-6789` | La stringa deve corrispondere all'espressione regolare. Vedere suggerimenti sulla definizione di un'espressione regolare. |
| `required` | `{name:required}` | `Rick` | Usato per imporre che un valore diverso da un parametro sia presente durante la generazione dell'URL |

[!INCLUDE[](~/includes/regex.md)]

È possibile applicare più vincoli delimitati da due punti a un singolo parametro. Ad esempio il vincolo seguente limita un parametro a un valore intero maggiore o uguale a 1:

```csharp
[Route("users/{id:int:min(1)}")]
public User GetUserById(int id) { }
```

> [!WARNING]
> I vincoli di route che verificano l'URL e vengono convertiti in un tipo CLR utilizzano sempre la lingua inglese. Ad esempio, la conversione al tipo CLR `int` o `DateTime` . Questi vincoli presuppongono che l'URL non sia localizzabile. I vincoli di route specificati dal framework non modificano i valori archiviati nei valori di route. Tutti i valori di route analizzati dall'URL vengono archiviati come stringhe. Ad esempio, il vincolo `float` prova a convertire il valore di route in un valore float, ma il valore convertito viene usato solo per verificare che può essere convertito in un valore float.

### <a name="regular-expressions-in-constraints"></a>Espressioni regolari nei vincoli

[!INCLUDE[](~/includes/regex.md)]

Le espressioni regolari possono essere specificate come vincoli inline usando il `regex(...)` vincolo di route. I metodi della <xref:Microsoft.AspNetCore.Builder.ControllerEndpointRouteBuilderExtensions.MapControllerRoute*> famiglia accettano anche un valore letterale di oggetti Constraints. Se viene usato questo formato, i valori stringa vengono interpretati come espressioni regolari.

Il codice seguente usa un vincolo Regex inline:

[!code-csharp[](routing/samples/3.x/RoutingSample/StartupRegex.cs?name=snippet)]

Il codice seguente usa un valore letterale dell'oggetto per specificare un vincolo Regex:

[!code-csharp[](routing/samples/3.x/RoutingSample/StartupRegex2.cs?name=snippet)]

Il framework di ASP.NET Core aggiunge `RegexOptions.IgnoreCase | RegexOptions.Compiled | RegexOptions.CultureInvariant` al costruttore di espressioni regolari. Per una descrizione di questi membri, vedere <xref:System.Text.RegularExpressions.RegexOptions>.

Le espressioni regolari usano delimitatori e token simili a quelli usati dal routing e dal linguaggio C#. Per i token di espressione è necessario aggiungere caratteri di escape. Per utilizzare l'espressione regolare `^\d{3}-\d{2}-\d{4}$` in un vincolo inline, utilizzare uno dei seguenti elementi:

* Sostituire `\` i caratteri specificati nella stringa come `\\` caratteri nel file di origine C# per poter eseguire l'escape del `\` carattere di escape della stringa.
* [Valori letterali stringa Verbatim](/dotnet/csharp/language-reference/keywords/string).

Per eseguire l'escape dei caratteri di delimitazione del parametro di routing,,, `{` `}` `[` `]` , raddoppiare i caratteri nell'espressione, ad esempio `{{` `}}` `[[` `]]` ,,,. La tabella seguente illustra un'espressione regolare e la relativa versione di escape:

| Espressione regolare‏    | Espressione regolare con escape     |
| --------------------- | ------------------------------ |
| `^\d{3}-\d{2}-\d{4}$` | `^\\d{{3}}-\\d{{2}}-\\d{{4}}$` |
| `^[a-z]{2}$`          | `^[[a-z]]{{2}}$`               |

Le espressioni regolari utilizzate nel routing spesso iniziano con il `^` carattere e corrispondono alla posizione iniziale della stringa. Le espressioni terminano spesso con il `$` carattere e corrispondono alla fine della stringa. I `^` `$` caratteri e assicurano che l'espressione regolare corrisponda all'intero valore del parametro di route. Senza i `^` `$` caratteri e, l'espressione regolare trova la corrispondenza con qualsiasi sottostringa all'interno della stringa, che è spesso indesiderata. La tabella seguente fornisce esempi e spiega perché corrispondono o non riescono a corrispondere:

| Expression   | string    | Corrispondenza | Commento               |
| ------------ | --------- | :---: |  -------------------- |
| `[a-z]{2}`   | hello     | Sì   | Corrispondenze di sottostringhe     |
| `[a-z]{2}`   | 123abc456 | Sì   | Corrispondenze di sottostringhe     |
| `[a-z]{2}`   | mz        | Sì   | Corrisponde all'espressione    |
| `[a-z]{2}`   | MZ        | Sì   | Senza distinzione maiuscole/minuscole    |
| `^[a-z]{2}$` | hello     | No    | Vedere `^` e `$` sopra |
| `^[a-z]{2}$` | 123abc456 | No    | Vedere `^` e `$` sopra |

Per altre informazioni sulla sintassi delle espressioni regolari, vedere [Espressioni regolari di .NET Framework](/dotnet/standard/base-types/regular-expression-language-quick-reference).

Per limitare un parametro a un set noto di valori possibili, usare un'espressione regolare. Ad esempio, `{action:regex(^(list|get|create)$)}` verifica la corrispondenza del valore di route `action` solo con `list`, `get` o `create`. Se viene passata nel dizionario di vincoli, la stringa `^(list|get|create)$` è equivalente. Anche i vincoli passati nel dizionario dei vincoli che non corrispondono a uno dei vincoli noti vengono considerati espressioni regolari. I vincoli passati in un modello che non corrispondono a uno dei vincoli noti non vengono considerati come espressioni regolari.

### <a name="custom-route-constraints"></a>Vincoli di Route personalizzati

È possibile creare vincoli di Route personalizzati implementando l' <xref:Microsoft.AspNetCore.Routing.IRouteConstraint> interfaccia. L' `IRouteConstraint` interfaccia contiene <xref:System.Web.Routing.IRouteConstraint.Match*> , che restituisce `true` se il vincolo è soddisfatto e `false` in caso contrario.

I vincoli di Route personalizzati sono raramente necessari. Prima di implementare un vincolo di route personalizzato, prendere in considerazione le alternative, ad esempio l'associazione di modelli.

Nella cartella ASP.NET Core [Constraints](https://github.com/dotnet/aspnetcore/tree/main/src/Http/Routing/src/Constraints) sono disponibili ottimi esempi di creazione di vincoli. Ad esempio, [GuidRouteConstraint](https://github.com/dotnet/aspnetcore/blob/main/src/Http/Routing/src/Constraints/GuidRouteConstraint.cs#L18).

Per usare un oggetto personalizzato `IRouteConstraint` , il tipo di vincolo di route deve essere registrato con l'app <xref:Microsoft.AspNetCore.Routing.RouteOptions.ConstraintMap> nel contenitore del servizio. `ConstraintMap` è un dizionario che esegue il mapping delle chiavi dei vincoli di route alle implementazioni di `IRouteConstraint` che convalidano tali vincoli. La proprietà `ConstraintMap` di un'app può essere aggiornata in `Startup.ConfigureServices` come parte di una chiamata [services.AddRouting](xref:Microsoft.Extensions.DependencyInjection.RoutingServiceCollectionExtensions.AddRouting*) o configurando <xref:Microsoft.AspNetCore.Routing.RouteOptions> direttamente con `services.Configure<RouteOptions>`. Ad esempio:

[!code-csharp[](routing/samples/3.x/RoutingSample/StartupConstraint.cs?name=snippet)]

Il vincolo precedente viene applicato nel codice seguente:

[!code-csharp[](routing/samples/3.x/RoutingSample/Controllers/TestController.cs?name=snippet&highlight=6,13)]

[!INCLUDE[](~/includes/MyDisplayRouteInfo.md)]

L'implementazione di `MyCustomConstraint` impedisce `0` che venga applicato a un parametro di route:

[!code-csharp[](routing/samples/3.x/RoutingSample/StartupConstraint.cs?name=snippet2)]

[!INCLUDE[](~/includes/regex.md)]

Il codice precedente:

* Impedisce `0` il `{id}` segmento della route.
* Viene illustrato come fornire un esempio di base per l'implementazione di un vincolo personalizzato. Non deve essere usato in un'app di produzione.

Il codice seguente rappresenta un approccio migliore per impedire l'elaborazione di un oggetto `id` che contiene un oggetto `0` :

[!code-csharp[](routing/samples/3.x/RoutingSample/Controllers/TestController.cs?name=snippet2)]

Il codice precedente presenta i vantaggi seguenti rispetto all' `MyCustomConstraint` approccio:

* Non richiede un vincolo personalizzato.
* Restituisce un errore più descrittivo quando il parametro di route include `0` .

## <a name="parameter-transformer-reference"></a>Riferimento ai trasformatori di parametro

I trasformatori di parametro:

* Eseguire quando si genera un collegamento utilizzando <xref:Microsoft.AspNetCore.Routing.LinkGenerator> .
* Implementare <xref:Microsoft.AspNetCore.Routing.IOutboundParameterTransformer?displayProperty=fullName>.
* Vengono configurati tramite <xref:Microsoft.AspNetCore.Routing.RouteOptions.ConstraintMap>.
* Acquisire il valore della route del parametro e trasformarlo in un nuovo valore stringa.
* Il valore trasformato viene usato nel collegamento generato.

Ad esempio, un trasformatore di parametro `slugify` personalizzato nel modello di route `blog\{article:slugify}` con `Url.Action(new { article = "MyTestArticle" })` genera `blog\my-test-article`.

Si consideri l' `IOutboundParameterTransformer` implementazione seguente:

[!code-csharp[](routing/samples/3.x/RoutingSample/StartupConstraint2.cs?name=snippet2)]

Per usare un trasformatore di parametro in un modello di route, configurarlo usando <xref:Microsoft.AspNetCore.Routing.RouteOptions.ConstraintMap> in `Startup.ConfigureServices` :

[!code-csharp[](routing/samples/3.x/RoutingSample/StartupConstraint2.cs?name=snippet)]

Il framework ASP.NET Core usa i trasformatori di parametro per trasformare l'URI in cui viene risolto un endpoint. I trasformatori dei parametri, ad esempio, trasformano i valori di route utilizzati per la corrispondenza di `area` , `controller` , `action` e `page` .

```csharp
routes.MapControllerRoute(
    name: "default",
    template: "{controller:slugify=Home}/{action:slugify=Index}/{id?}");
```

Con il modello di route precedente, l'azione `SubscriptionManagementController.GetAll` viene confrontata con l'URI `/subscription-management/get-all` . Un trasformatore di parametro non modifica i valori della route usati per generare un collegamento. Ad esempio, `Url.Action("GetAll", "SubscriptionManagement")` restituisce `/subscription-management/get-all`.

ASP.NET Core fornisce le convenzioni API per l'uso di trasformatori di parametri con route generate:

* La <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.RouteTokenTransformerConvention?displayProperty=fullName> convenzione MVC applica un trasformatore di parametro specificato a tutte le route di attributi nell'app. Il trasformatore di parametro trasforma i token di route di attributi man mano che vengono sostituiti. Per altre informazioni vedere [Use a parameter transformer to customize token replacement](xref:mvc/controllers/routing#use-a-parameter-transformer-to-customize-token-replacement) (Usare un trasformatore di parametro per personalizzare la sostituzione di token).
* Razor Pages usa la <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.PageRouteTransformerConvention> convenzione API. Questa convenzione applica un trasformatore di parametro specificato a tutte le pagine individuate automaticamente Razor . Il trasformatore di parametri trasforma la cartella e i segmenti dei nomi di file delle Razor Route di pagine. Per altre informazioni, vedere [Use a parameter transformer to customize page routes](xref:razor-pages/razor-pages-conventions#use-a-parameter-transformer-to-customize-page-routes) (Usare un trasformatore di parametro per personalizzare route di pagine).

<a name="ugr"></a>

## <a name="url-generation-reference"></a>Riferimento per la generazione di URL

Questa sezione contiene un riferimento per l'algoritmo implementato dalla generazione di URL. In pratica, gli esempi più complessi di generazione di URL utilizzano controller o Razor pagine. Per ulteriori informazioni, vedere  [routing nei controller](xref:mvc/controllers/routing) .

Il processo di generazione dell'URL inizia con una chiamata a [LinkGenerator. GetPathByAddress](xref:Microsoft.AspNetCore.Routing.LinkGenerator.GetPathByAddress*) o a un metodo simile. Il metodo viene fornito con un indirizzo, un set di valori di route e, facoltativamente, informazioni sulla richiesta corrente da `HttpContext` .

Il primo passaggio consiste nell'usare l'indirizzo per risolvere un set di endpoint candidati usando un oggetto [`IEndpointAddressScheme<TAddress>`](xref:Microsoft.AspNetCore.Routing.IEndpointAddressScheme`1) che corrisponde al tipo dell'indirizzo.

Una volta che il set di candidati viene trovato dallo schema degli indirizzi, gli endpoint vengono ordinati ed elaborati in modo iterativo fino a quando un'operazione di generazione URL non riesce. La generazione di URL **non** controlla le ambiguità, il primo risultato restituito è il risultato finale.

### <a name="troubleshooting-url-generation-with-logging"></a>Risoluzione dei problemi di generazione URL con registrazione

Il primo passaggio nella risoluzione dei problemi di generazione degli URL consiste nell'impostazione del livello di registrazione di `Microsoft.AspNetCore.Routing` su `TRACE` . `LinkGenerator` registra molti dettagli sull'elaborazione che possono essere utili per la risoluzione dei problemi.

Per informazioni dettagliate sulla generazione di URL, vedere riferimento per la [generazione di URL](#ugr) .

### <a name="addresses"></a>Indirizzi

Gli indirizzi rappresentano il concetto di generazione di URL usato per associare una chiamata al generatore di collegamenti a un set di endpoint candidati.

Gli indirizzi sono un concetto estendibile che include due implementazioni per impostazione predefinita:

* Utilizzando il *nome dell'endpoint* ( `string` ) come indirizzo:
    * Fornisce funzionalità simili al nome della route MVC.
    * Usa il <xref:Microsoft.AspNetCore.Routing.IEndpointNameMetadata> tipo di metadati.
    * Risolve la stringa fornita in base ai metadati di tutti gli endpoint registrati.
    * Genera un'eccezione all'avvio se più endpoint utilizzano lo stesso nome.
    * Consigliato per uso generico all'esterno dei controller e delle Razor pagine.
* Utilizzando *i valori di route* ( <xref:Microsoft.AspNetCore.Routing.RouteValuesAddress> ) come indirizzo:
    * Fornisce funzionalità simili a controller e Razor pagine di generazione URL legacy.
    * Molto complesso da estendere ed eseguire il debug.
    * Fornisce l'implementazione usata da `IUrlHelper` , helper tag, helper HTML, risultati dell'azione e così via.

Il ruolo dello schema di indirizzi è quello di fare in modo che l'associazione tra l'indirizzo e gli endpoint corrispondenti venga stabilita da criteri arbitrari:

* Lo schema nome endpoint esegue una ricerca dizionario di base.
* Lo schema dei valori di route ha un subset migliore complesso di algoritmo set.

<a name="ambient"></a>

### <a name="ambient-values-and-explicit-values"></a>Valori di ambiente e valori espliciti

Dalla richiesta corrente, il routing accede ai valori della route della richiesta corrente `HttpContext.Request.RouteValues` . I valori associati alla richiesta corrente sono detti **valori di ambiente**. Per maggiore chiarezza, la documentazione fa riferimento ai valori di route passati ai metodi come **valori espliciti**.

Nell'esempio seguente vengono illustrati i valori di ambiente e i valori espliciti. Fornisce valori di ambiente dalla richiesta corrente e valori espliciti: `{ id = 17, }` :

[!code-csharp[](routing/samples/3.x/RoutingSample/Controllers/WidgetController.cs?name=snippet)]

Il codice precedente:

* Restituisce `/Widget/Index/17`.
* Ottiene <xref:Microsoft.AspNetCore.Routing.LinkGenerator> tramite [di](xref:fundamentals/dependency-injection).

Il codice seguente non fornisce valori di ambiente e valori espliciti: `{ controller = "Home", action = "Subscribe", id = 17, }` :

[!code-csharp[](routing/samples/3.x/RoutingSample/Controllers/WidgetController.cs?name=snippet2)]

Il metodo precedente restituisce `/Home/Subscribe/17`

Il codice seguente nell'oggetto `WidgetController` restituisce `/Widget/Subscribe/17` :

[!code-csharp[](routing/samples/3.x/RoutingSample/Controllers/WidgetController.cs?name=snippet3)]

Il codice seguente fornisce al controller i valori di ambiente nella richiesta corrente e i valori espliciti: `{ action = "Edit", id = 17, }` :

[!code-csharp[](routing/samples/3.x/RoutingSample/Controllers/GadgetController.cs?name=snippet)]

Nel codice precedente:

* `/Gadget/Edit/17` viene restituito.
* <xref:Microsoft.AspNetCore.Mvc.ControllerBase.Url> Ottiene l'oggetto <xref:Microsoft.AspNetCore.Mvc.IUrlHelper> .
* <xref:Microsoft.AspNetCore.Mvc.UrlHelperExtensions.Action*>   
genera un URL con un percorso assoluto per un metodo di azione. L'URL contiene il `action` nome e i `route` valori specificati.

Il codice seguente fornisce valori di ambiente dalla richiesta corrente e valori espliciti: `{ page = "./Edit, id = 17, }` :

[!code-csharp[](routing/samples/3.x/RoutingSample/Pages/Index.cshtml.cs?name=snippet)]

Il codice precedente imposta `url` su  `/Edit/17` quando la pagina di modifica Razor contiene la direttiva della pagina seguente:

 `@page "{id:int}"`

Se la pagina di modifica non contiene il `"{id:int}"` modello di route, `url` è `/Edit?id=17` .

Il comportamento di MVC <xref:Microsoft.AspNetCore.Mvc.IUrlHelper> aggiunge un livello di complessità oltre alle regole descritte di seguito:

* `IUrlHelper` fornisce sempre i valori di route della richiesta corrente come valori di ambiente.
* [IUrlHelper. Action](xref:Microsoft.AspNetCore.Mvc.UrlHelperExtensions.Action*) copia sempre i `action` `controller` valori di route e correnti come valori espliciti a meno che non vengano sostituiti dallo sviluppatore.
* [IUrlHelper. Page](xref:Microsoft.AspNetCore.Mvc.UrlHelperExtensions.Page*) copia sempre il `page` valore di route corrente come valore esplicito a meno che non venga sottoposto a override. <!--by the user-->
* `IUrlHelper.Page` esegue sempre l'override del `handler` valore di route corrente con `null` come valori espliciti a meno che non venga sottoposto a override.

Spesso gli utenti sono sorpresi dai dettagli comportamentali dei valori di ambiente, perché MVC non sembra rispettare le proprie regole. Per motivi cronologici e di compatibilità, determinati valori di route, ad esempio `action` ,, `controller` `page` e `handler` hanno il proprio comportamento speciale.

La funzionalità equivalente fornita da `LinkGenerator.GetPathByAction` e `LinkGenerator.GetPathByPage` Duplica le anomalie di `IUrlHelper` per la compatibilità.

### <a name="url-generation-process"></a>Processo di generazione URL

Una volta individuato il set di endpoint candidati, l'algoritmo di generazione dell'URL:

* Elabora gli endpoint in modo iterativo.
* Restituisce il primo risultato riuscito.

Il primo passaggio di questo processo è denominato **invalidamento del valore di route**.  L'invalidamento del valore di route è il processo mediante il quale il routing decide i valori di route dei valori di ambiente da utilizzare e che devono essere ignorati. Ogni valore di ambiente viene considerato e combinato con i valori espliciti o ignorati.

Il modo migliore per considerare il ruolo dei valori di ambiente è che tentano di salvare gli sviluppatori di applicazioni che digitano, in alcuni casi comuni. Tradizionalmente, gli scenari in cui i valori di ambiente sono utili sono correlati a MVC:

* Quando si esegue il collegamento a un'altra azione nello stesso controller, non è necessario specificare il nome del controller.
* Quando si esegue il collegamento a un altro controller nella stessa area, non è necessario specificare il nome dell'area.
* Quando si esegue il collegamento allo stesso metodo di azione, non è necessario specificare i valori di route.
* Quando si esegue il collegamento a un'altra parte dell'app, non si vuole trasferire i valori di route che non hanno significato in questa parte dell'app.

Le chiamate a `LinkGenerator` o `IUrlHelper` che restituiscono `null` sono in genere dovute alla mancata comprensione dell'invalidamento del valore della route. Risolvere i problemi di invalidamento del valore della route specificando in modo esplicito più valori di route per vedere se il problema viene risolto.

L'invalidamento del valore di route si basa sul presupposto che lo schema dell'URL dell'app sia gerarchico, con una gerarchia formata da sinistra verso destra. Si consideri il modello di route del controller `{controller}/{action}/{id?}` di base per ottenere un senso intuitivo di come funziona in pratica. Una **modifica** a un valore **invalida** tutti i valori di route visualizzati a destra. Ciò riflette il presupposto della gerarchia. Se l'app ha un valore di ambiente per `id` e l'operazione specifica un valore diverso per `controller` :

* `id` non verrà riutilizzato perché `{controller}` è a sinistra di `{id?}` .

Di seguito sono riportati alcuni esempi che illustrano questo principio:

* Se i valori espliciti contengono un valore per `id` , il valore di ambiente per `id` viene ignorato. I valori di ambiente per `controller` e `action` possono essere utilizzati.
* Se i valori espliciti contengono un valore per `action` , qualsiasi valore di ambiente per `action` viene ignorato. I valori di ambiente per `controller` possono essere utilizzati. Se il valore esplicito per `action` è diverso dal valore di ambiente per `action` , il `id` valore non verrà utilizzato.  Se il valore esplicito per `action` è uguale al valore di ambiente di `action` , è `id` possibile utilizzare il valore.
* Se i valori espliciti contengono un valore per `controller` , qualsiasi valore di ambiente per `controller` viene ignorato. Se il valore esplicito per `controller` è diverso dal valore di ambiente per `controller` , `action` i `id` valori e non verranno utilizzati. Se il valore esplicito per `controller` è uguale al valore di ambiente di `controller` , è `action` `id` possibile utilizzare i valori e.

Questo processo è ulteriormente complicato dall'esistenza delle route degli attributi e delle route convenzionali dedicate. Route convenzionali del controller come `{controller}/{action}/{id?}` specificare una gerarchia usando i parametri di route. Per route [convenzionali dedicate](xref:mvc/controllers/routing#dcr) e [Route di attributi](xref:mvc/controllers/routing#ar) a controller e Razor pagine:

* Esiste una gerarchia di valori di route.
* Non vengono visualizzati nel modello.

In questi casi, la generazione di URL definisce il concetto di **valori richiesti** . Per gli endpoint creati da controller e Razor pagine sono specificati i valori obbligatori che consentono il funzionamento dell'invalidamento del valore di route.

Algoritmo di invalidamento del valore di route in dettaglio:

* I nomi dei valori obbligatori vengono combinati con i parametri di route, quindi elaborati da sinistra a destra.
* Per ogni parametro, vengono confrontati il valore di ambiente e il valore esplicito:
    * Se il valore di ambiente e il valore esplicito sono uguali, il processo continua.
    * Se il valore di ambiente è presente e il valore esplicito non è, il valore di ambiente viene usato durante la generazione dell'URL.
    * Se il valore di ambiente non è presente e il valore esplicito è, rifiutare il valore di ambiente e tutti i valori di ambiente successivi.
    * Se il valore di ambiente e il valore esplicito sono presenti e i due valori sono diversi, rifiutare il valore di ambiente e tutti i valori di ambiente successivi.

A questo punto, l'operazione di generazione URL è pronta per valutare i vincoli di route. Il set di valori accettati viene combinato con i valori predefiniti del parametro, fornito ai vincoli. Se tutti i vincoli passano, l'operazione continua.

Successivamente, è possibile usare i **valori accettati** per espandere il modello di route. Il modello di route viene elaborato:

* Da sinistra a destra.
* Ogni parametro ha il valore accettato sostituito.
* Con i casi speciali seguenti:
  * Se nei valori accettati manca un valore e il parametro ha un valore predefinito, viene usato il valore predefinito.
  * Se nei valori accettati manca un valore e il parametro è facoltativo, l'elaborazione continua.
  * Se un parametro di route a destra di un parametro facoltativo mancante ha un valore, l'operazione ha esito negativo.
  * <!-- review default-valued parameters optional parameters --> I parametri con valori predefiniti contigui e i parametri facoltativi vengono compressi laddove possibile.

I valori specificati in modo esplicito che non corrispondono a un segmento della route vengono aggiunti alla stringa di query. La tabella seguente illustra il risultato ottenuto quando si usa il modello di route `{controller}/{action}/{id?}`.

| Valori di ambiente                     | Valori espliciti                        | Risultato                  |
| ---------------------------------- | -------------------------------------- | ----------------------- |
| controller = "Home"                | action = "About"                       | `/Home/About`           |
| controller = "Home"                | controller = "Order", action = "About" | `/Order/About`          |
| controller = "Home", color = "Red" | action = "About"                       | `/Home/About`           |
| controller = "Home"                | action = "About", color = "Red"        | `/Home/About?color=Red` |

### <a name="problems-with-route-value-invalidation"></a>Problemi di invalidamento del valore di route

A partire da ASP.NET Core 3,0, alcuni schemi di generazione URL usati nelle versioni precedenti ASP.NET Core non funzionano correttamente con la generazione di URL. Il team ASP.NET Core prevede di aggiungere funzionalità per soddisfare queste esigenze in una versione futura. Per ora, la soluzione migliore consiste nell'usare il routing legacy.

Il codice seguente illustra un esempio di uno schema di generazione URL non supportato dal routing.

[!code-csharp[](routing/samples/3.x/RoutingSample/StartupUnsupported.cs?name=snippet)]

Nel codice precedente, il `culture` parametro di route viene usato per la localizzazione. Il desiderio è che il `culture` parametro sia sempre accettato come valore di ambiente. Tuttavia, il `culture` parametro non viene accettato come valore di ambiente a causa del modo in cui i valori richiesti funzionano:

* Nel `"default"` modello di route, il `culture` parametro di route è a sinistra di `controller` , quindi le modifiche a `controller` non invalidano `culture` .
* Nel `"blog"` modello di route, il `culture` parametro di route viene considerato a destra di `controller` , che viene visualizzato nei valori richiesti.

## <a name="configuring-endpoint-metadata"></a>Configurazione dei metadati dell'endpoint

I collegamenti seguenti forniscono informazioni sulla configurazione dei metadati dell'endpoint:

* [Abilitare cors con routing degli endpoint](xref:security/cors#enable-cors-with-endpoint-routing)
* [Esempio di IAuthorizationPolicyProvider](https://github.com/dotnet/AspNetCore/tree/release/3.1/src/Security/samples/CustomPolicyProvider) con un `[MinimumAgeAuthorize]` attributo personalizzato
* [Testare l'autenticazione con l'attributo [autorizzate]](xref:security/authentication/identity#test-identity)
* <xref:Microsoft.AspNetCore.Builder.AuthorizationEndpointConventionBuilderExtensions.RequireAuthorization*>
* [Selezione dello schema con l'attributo [autorizzate]](xref:security/authorization/limitingidentitybyscheme#selecting-the-scheme-with-the-authorize-attribute)
* [Applicare i criteri usando l'attributo [autorizzate]](xref:security/authorization/policies#apply-policies-to-mvc-controllers)
* <xref:security/authorization/roles>

<a name="hostmatch"></a>

## <a name="host-matching-in-routes-with-requirehost"></a>Corrispondenza host nelle route con RequireHost

<xref:Microsoft.AspNetCore.Builder.RoutingEndpointConventionBuilderExtensions.RequireHost*> applica un vincolo alla route che richiede l'host specificato. Il `RequireHost` parametro o [[host]](xref:Microsoft.AspNetCore.Routing.HostAttribute) può essere:

* Host: `www.domain.com` , corrisponde `www.domain.com` a qualsiasi porta.
* Host con carattere jolly: `*.domain.com` , `www.domain.com` corrisponde `subdomain.domain.com` a, `www.subdomain.domain.com` o su qualsiasi porta.
* Port: `*:5000` , corrisponde alla porta 5000 con qualsiasi host.
* Host e porta: `www.domain.com:5000` oppure `*.domain.com:5000` corrisponde a host e porta.

È possibile specificare più parametri usando `RequireHost` o `[Host]` . Il vincolo corrisponde a host validi per uno qualsiasi dei parametri. Ad esempio, `[Host("domain.com", "*.domain.com")]` corrisponde a `domain.com` , `www.domain.com` e `subdomain.domain.com` .

Il codice seguente usa `RequireHost` per richiedere l'host specificato nella route:

[!code-csharp[](routing/samples/3.x/RoutingSample/StartupRequireHost.cs?name=snippet)]

Il codice seguente usa l' `[Host]` attributo sul controller per richiedere uno degli host specificati:

[!code-csharp[](routing/samples/3.x/RoutingSample/Controllers/ProductController.cs?name=snippet)]

Quando l' `[Host]` attributo viene applicato sia al controller che al metodo di azione:

* Viene utilizzato l'attributo sull'azione.
* L'attributo controller viene ignorato.

## <a name="performance-guidance-for-routing"></a>Linee guida sulle prestazioni per il routing

La maggior parte del routing è stata aggiornata in ASP.NET Core 3,0 per migliorare le prestazioni.

Quando un'applicazione presenta problemi di prestazioni, il routing è spesso sospetto come il problema. Il motivo per cui il routing è sospetto è che i Framework come i controller e Razor le pagine segnalino la quantità di tempo impiegato all'interno del Framework nei messaggi di registrazione. Quando esiste una differenza significativa tra il tempo segnalato dai controller e il tempo totale della richiesta:

* Gli sviluppatori eliminano il codice dell'app come origine del problema.
* Si presuppone che il routing sia la ragione.

Il routing è sottoposto a test delle prestazioni con migliaia di endpoint. È improbabile che un'app tipica riscontri un problema di prestazioni semplicemente troppo grande. La causa principale più comune di rallentamento delle prestazioni del routing è in genere un middleware personalizzato.

Nell'esempio di codice seguente viene illustrata una tecnica di base per limitare l'origine del ritardo:

[!code-csharp[](routing/samples/3.x/RoutingSample/StartupDelay.cs?name=snippet)]

Al routing temporale:

* Interleave ogni middleware con una copia del middleware di temporizzazione illustrato nel codice precedente.
* Aggiungere un identificatore univoco per correlare i dati di intervallo con il codice.

Si tratta di un metodo di base per limitare il ritardo quando è significativo, ad esempio più di `10ms` .  La sottrazione `Time 2` da `Time 1` segnala il tempo impiegato all'interno del `UseRouting` middleware.

Il codice seguente usa un approccio più compatto al codice temporale precedente:

[!code-csharp[](routing/samples/3.x/RoutingSample/StartupSW.cs?name=snippetSW)]

[!code-csharp[](routing/samples/3.x/RoutingSample/StartupSW.cs?name=snippet)]

### <a name="potentially-expensive-routing-features"></a>Funzionalità di routing potenzialmente costose

Nell'elenco seguente vengono fornite informazioni dettagliate sulle funzionalità di routing relativamente dispendiose rispetto ai modelli di route di base:

* Espressioni regolari: è possibile scrivere espressioni regolari complesse o con tempi di esecuzione prolungati con una piccola quantità di input.

* Segmenti complessi ( `{x}-{y}-{z}` ): 
  * Sono molto più costosi dell'analisi di un segmento di percorso URL normale.
  * Comporta l'allocazione di molte altre sottostringhe.
  * La logica del segmento complesso non è stata aggiornata nell'aggiornamento delle prestazioni di routing ASP.NET Core 3,0.

* Accesso sincrono ai dati: molte app complesse hanno accesso al database come parte del routing. ASP.NET Core 2,2 e il routing precedente potrebbero non fornire i punti di estendibilità corretti per supportare il routing di accesso al database. Ad esempio, <xref:Microsoft.AspNetCore.Routing.IRouteConstraint> e <xref:Microsoft.AspNetCore.Mvc.ActionConstraints.IActionConstraint> sono sincroni. I punti di estendibilità, ad esempio <xref:Microsoft.AspNetCore.Routing.MatcherPolicy> e, <xref:Microsoft.AspNetCore.Routing.EndpointSelectorContext> sono asincroni.

## <a name="guidance-for-library-authors"></a>Linee guida per gli autori di librerie

Questa sezione contiene indicazioni per gli autori di librerie che si basano sul routing. Questi dettagli sono destinati a garantire che gli sviluppatori di app abbiano un'esperienza ottimale usando le librerie e i Framework che estendono il routing.

### <a name="define-endpoints"></a>Definire gli endpoint

Per creare un Framework che usa il routing per la corrispondenza degli URL, iniziare definendo un'esperienza utente basata su <xref:Microsoft.AspNetCore.Builder.EndpointRoutingApplicationBuilderExtensions.UseEndpoints*> .

**Eseguire** la compilazione sopra <xref:Microsoft.AspNetCore.Routing.IEndpointRouteBuilder> . Ciò consente agli utenti di comporre il Framework con altre ASP.NET Core funzionalità senza confusione. Ogni modello di ASP.NET Core include il routing. Si supponga che il routing sia presente e familiare per gli utenti.

```csharp
app.UseEndpoints(endpoints =>
{
    // Your framework
    endpoints.MapMyFramework(...);

    endpoints.MapHealthChecks("/healthz");
});
```

**Restituisce un** tipo concreto sealed da una chiamata a `MapMyFramework(...)` che implementa <xref:Microsoft.AspNetCore.Builder.IEndpointConventionBuilder> . La maggior parte dei metodi del Framework `Map...` seguono questo modello. `IEndpointConventionBuilder`Interfaccia:

* Consente la composizione dei metadati.
* È destinato a un'ampia gamma di metodi di estensione.

La dichiarazione di un tipo personalizzato consente di aggiungere funzionalità specifiche del Framework al generatore. È possibile eseguire il wrapping di un generatore dichiarato dal Framework e inviare le chiamate.

```csharp
app.UseEndpoints(endpoints =>
{
    // Your framework
    endpoints.MapMyFramework(...).RequireAuthorization()
                                 .WithMyFrameworkFeature(awesome: true);

    endpoints.MapHealthChecks("/healthz");
});
```

**Prendere in considerazione** la scrittura <xref:Microsoft.AspNetCore.Routing.EndpointDataSource> . `EndpointDataSource` è la primitiva di basso livello per la dichiarazione e l'aggiornamento di una raccolta di endpoint. `EndpointDataSource` è un'API potente utilizzata dai controller e dalle Razor pagine.

I test di routing hanno un [esempio di base](https://github.com/dotnet/AspNetCore/blob/main/src/Http/Routing/test/testassets/RoutingSandbox/Framework/FrameworkEndpointDataSource.cs#L17) di un'origine dati che non esegue l'aggiornamento.

**Non** tentare di registrare un per `EndpointDataSource` impostazione predefinita. Richiedere agli utenti di registrare il Framework in <xref:Microsoft.AspNetCore.Builder.EndpointRoutingApplicationBuilderExtensions.UseEndpoints*> . La filosofia del routing è che nulla è incluso per impostazione predefinita ed `UseEndpoints` è la posizione in cui registrare gli endpoint.

### <a name="creating-routing-integrated-middleware"></a>Creazione del middleware integrato per il routing

**Prendere in considerazione** la definizione di tipi di metadati come interfaccia.

 È possibile utilizzare i tipi di metadati come attributo per classi e metodi.

[!code-csharp[](routing/samples/3.x/RoutingSample/ICoolMetadata.cs?name=snippet2)]

I Framework come controller e Razor pagine supportano l'applicazione di attributi di metadati ai tipi e ai metodi. Se si dichiarano i tipi di metadati:

* Renderli accessibili come [attributi](/dotnet/csharp/programming-guide/concepts/attributes/).
* La maggior parte degli utenti ha familiarità con l'applicazione di attributi.

La dichiarazione di un tipo di metadati come interfaccia comporta l'aggiunta di un altro livello di flessibilità:

* Le interfacce sono componibili.
* Gli sviluppatori possono dichiarare tipi propri che combinano più criteri.

È **possibile eseguire l'** override dei metadati, come illustrato nell'esempio seguente:

[!code-csharp[](routing/samples/3.x/RoutingSample/ICoolMetadata.cs?name=snippet)]

Il modo migliore per seguire queste linee guida consiste nell'evitare di definire i **metadati del marcatore**:

* Non solo cercare la presenza di un tipo di metadati.
* Definire una proprietà nei metadati e controllare la proprietà.

La raccolta dei metadati è ordinata e supporta l'override per priorità. Nel caso di controller, i metadati del metodo di azione sono più specifici.

**Rendere il** middleware utile con e senza routing.

```csharp
app.UseRouting();

app.UseAuthorization(new AuthorizationPolicy() { ... });

app.UseEndpoints(endpoints =>
{
    // Your framework
    endpoints.MapMyFramework(...).RequireAuthorization();
});
```

Come esempio di questa linea guida, prendere in considerazione il `UseAuthorization` middleware. Il middleware di autorizzazione consente di passare un criterio di fallback. <!-- shown where?  (shown here) --> I criteri di fallback, se specificati, si applicano a entrambi:

* Endpoint senza criteri specificati.
* Richieste che non corrispondono a un endpoint.

In questo modo il middleware di autorizzazione risulta utile al di fuori del contesto di routing. Il middleware di autorizzazione può essere usato per la programmazione middleware tradizionale.

[!INCLUDE[](~/includes/dbg-route.md)]

::: moniker-end

::: moniker range="= aspnetcore-2.2"

Il routing è responsabile del mapping degli URI di richiesta agli endpoint e dell'invio di richieste in ingresso a tali endpoint. Le route sono definite nell'app e vengono configurate all'avvio dell'app. Una route può facoltativamente estrarre valori dall'URL contenuto nella richiesta e questi valori possono quindi essere usati per l'elaborazione della richiesta. Usando le informazioni sulla route dall'app, il routing è anche in grado di generare URL con mapping agli endpoint.

Per usare gli scenari di routing più recenti in ASP.NET Core 2.2, specificare la [versione di compatibilità](xref:mvc/compatibility-version) nella registrazione dei servizi MVC in `Startup.ConfigureServices`:

```csharp
services.AddMvc()
    .SetCompatibilityVersion(CompatibilityVersion.Version_2_2);
```

L'opzione <xref:Microsoft.AspNetCore.Mvc.MvcOptions.EnableEndpointRouting> determina se il routing deve usare internamente la logica basata sugli endpoint o la logica basata su <xref:Microsoft.AspNetCore.Routing.IRouter> di ASP.NET Core 2.1 o versioni precedenti. Quando la versione di compatibilità è impostata su 2.2 o versioni successive, il valore predefinito è `true`. Impostare il valore su `false` per usare la logica di routing precedente:

```csharp
// Use the routing logic of ASP.NET Core 2.1 or earlier:
services.AddMvc(options => options.EnableEndpointRouting = false)
    .SetCompatibilityVersion(CompatibilityVersion.Version_2_2);
```

Per altre informazioni sul routing basato su <xref:Microsoft.AspNetCore.Routing.IRouter>, vedere la [versione per ASP.NET Core 2.1 di questo argomento](?view=aspnetcore-2.1).

> [!IMPORTANT]
> Questo documento descrive il routing di basso livello di ASP.NET Core. Per informazioni sul routing di ASP.NET Core MVC, vedere <xref:mvc/controllers/routing>. Per informazioni sulle convenzioni di routing nelle Razor pagine, vedere <xref:razor-pages/razor-pages-conventions> .

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/routing/samples) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="routing-basics"></a>Nozioni fondamentali sul routing

La maggior parte delle app dovrebbe scegliere uno schema di routing semplice e descrittivo in modo che gli URL siano leggibili e significativi. La route convenzionale predefinita `{controller=Home}/{action=Index}/{id?}`:

* Supporta uno schema di routing semplice e descrittivo.
* È un punto iniziale utile per le app basate su interfaccia utente.

Gli sviluppatori aggiungono in genere Route concise aggiuntive a aree con traffico elevato di un'app in situazioni specializzate usando il [routing degli attributi](xref:mvc/controllers/routing#attribute-routing) o route convenzionali dedicate. Esempi di situazioni specializzate sono gli endpoint di Blog e ecommerce.

Le API Web devono usare il routing degli attributi per modellare le funzionalità dell'app come set di risorse in cui le operazioni sono rappresentate da verbi HTTP. Ciò significa che molte operazioni, ad esempio GET e POST, sulla stessa risorsa logica utilizzano lo stesso URL. Il routing degli attributi offre un livello di controllo necessario per progettare con attenzione il layout dell'endpoint pubblico di un'API.

Razor Le app di pagine usano il routing convenzionale predefinito per gestire le risorse denominate nella cartella *pagine* di un'app. Sono disponibili convenzioni aggiuntive che consentono di personalizzare il Razor comportamento di routing delle pagine. Per altre informazioni, vedere <xref:razor-pages/index> e <xref:razor-pages/razor-pages-conventions>.

Il supporto della generazione di URL consente di sviluppare l'app senza URL hardcoded per collegare l'app. Il supporto consente di iniziare con una configurazione di routing di base e di modificare le route dopo aver determinato il layout delle risorse dell'app.

Il routing usa gli *endpoint* ( `Endpoint` ) per rappresentare gli endpoint logici in un'app.

Un endpoint definisce un delegato per l'elaborazione delle richieste e una raccolta di metadati arbitrari. I metadati vengono usati per implementare gli elementi trasversali in base ai criteri e la configurazione associata a ogni endpoint.

Il sistema di routing ha le caratteristiche seguenti:

* La sintassi del modello di route viene usata per definire le route con parametri di route in formato token.
* La configurazione degli endpoint basata su convenzioni e attributi è consentita.
* <xref:Microsoft.AspNetCore.Routing.IRouteConstraint> consente di determinare se un parametro URL contiene un valore valido per il vincolo di un endpoint specifico.
* I modelli di app, ad esempio MVC/ Razor pages, registrano tutti i relativi endpoint, che hanno un'implementazione prevedibile degli scenari di routing.
* L'implementazione del routing determina le decisioni relative al routing nel punto desiderato della pipeline del middleware.
* Il middleware successivo a un middleware di routing può esaminare il risultato della decisione dell'endpoint del middleware di routing per un URI di richiesta specifico.
* È possibile enumerare tutti gli endpoint nell'app in un punto qualsiasi della pipeline del middleware.
* Un'app può usare il routing per generare URL, ad esempio per il reindirizzamento o i collegamenti, in base alle informazioni degli endpoint. In questo modo si evita di impostare gli URL come hardcoded ottimizzando la manutenibilità.
* La generazione degli URL è basata sugli indirizzi, che supportano l'estensibilità arbitraria:

  * L'API del generatore di collegamenti (<xref:Microsoft.AspNetCore.Routing.LinkGenerator>) può essere risolta ovunque tramite l'[inserimento delle dipendenze](xref:fundamentals/dependency-injection) per generare gli URL.
  * Se l'API del generatore di collegamenti non è disponibile tramite l'inserimento delle dipendenze, <xref:Microsoft.AspNetCore.Mvc.IUrlHelper> offre metodi per la generazione degli URL.

> [!NOTE]
> Con il rilascio del routing degli endpoint in ASP.NET Core 2,2, il collegamento dell'endpoint è limitato alle Razor azioni e alle pagine MVC/pagine. Le espansioni delle funzionalità di collegamento degli endpoint sono previste per le versioni future.

Il routing è connesso alla pipeline [middleware](xref:fundamentals/middleware/index) per mezzo della classe <xref:Microsoft.AspNetCore.Builder.RouterMiddleware>. [ASP.NET Core MVC](xref:mvc/overview) aggiunge il routing alla pipeline middleware come parte della configurazione e gestisce il routing nelle app MVC e Razor pages. Per informazioni sull'uso del routing come componente autonomo, vedere la sezione [Usare il middleware di routing](#use-routing-middleware).

### <a name="url-matching"></a>Corrispondenza URL

La corrispondenza dell'URL è il processo con cui il routing invia una richiesta in ingresso a un *endpoint*. Questo processo si basa sui dati presenti nel percorso URL, ma può essere esteso in modo da prendere in considerazione tutti i dati della richiesta. La possibilità di inviare le richieste a gestori separati è fondamentale per ridurre le dimensioni e la complessità di un'app.

Il sistema di routing nell'endpoint di routing è responsabile di tutte le decisioni relative all'invio. Poiché il middleware applica i criteri in base all'endpoint selezionato, è importante che qualsiasi decisione che può influire sull'invio o l'applicazione dei criteri di sicurezza venga effettuata all'interno del sistema di routing.

Quando viene eseguito il delegato dell'endpoint, le proprietà di [RouteContext.RouteData](xref:Microsoft.AspNetCore.Routing.RouteContext.RouteData) vengono impostate sui valori appropriati in base all'elaborazione della richiesta eseguita fino a quel punto.

[RouteData.Values](xref:Microsoft.AspNetCore.Routing.RouteData.Values*) è un dizionario di *valori di route* prodotti dalla route. Questi valori in genere sono determinati dalla suddivisione in token dell'URL e possono essere usati per accettare l'input dell'utente o per prendere ulteriori decisioni riguardo all'invio all'interno dell'app.

[RouteData.DataTokens](xref:Microsoft.AspNetCore.Routing.RouteData.DataTokens*) è un elenco proprietà dei dati aggiuntivi correlati alla route corrispondente. Gli elementi <xref:Microsoft.AspNetCore.Routing.RouteData.DataTokens*> vengono forniti per supportare l'associazione dei dati sullo stato con ogni route in modo che l'app sia in grado di prendere decisioni in base alla route corrispondente. Questi valori sono definiti dallo sviluppatore e **non** influiscono sul comportamento del routing in alcun modo. Inoltre, i valori accantonati in [RouteData.DataTokens](xref:Microsoft.AspNetCore.Routing.RouteData.DataTokens*) possono essere di qualsiasi tipo, a differenza dei valori [RouteData.Values](xref:Microsoft.AspNetCore.Routing.RouteData.Values) che devono essere convertibili in e da stringhe.

[RouteData.Routers](xref:Microsoft.AspNetCore.Routing.RouteData.Routers) è un elenco delle route che hanno preso parte alla corrispondenza corretta della richiesta. Le route possono essere annidate l'una nell'altra. La proprietà <xref:Microsoft.AspNetCore.Routing.RouteData.Routers> rispecchia il percorso tramite l'albero logico di route che hanno prodotto una corrispondenza. In genere il primo elemento in <xref:Microsoft.AspNetCore.Routing.RouteData.Routers> è la raccolta di route e deve essere usato per la generazione di URL. L'ultimo elemento in <xref:Microsoft.AspNetCore.Routing.RouteData.Routers> è il gestore di route corrispondente.

<a name="lg"></a>

### <a name="url-generation-with-linkgenerator"></a>Generazione di URL con LinkGenerator

La generazione di URL è il processo con cui il routing crea un percorso URL basato su un set di valori di route. Questo consente di avere una separazione logica tra gli endpoint e gli URL che vi accedono.

Il routing di endpoint include l'API del generatore di collegamenti (<xref:Microsoft.AspNetCore.Routing.LinkGenerator>). <xref:Microsoft.AspNetCore.Routing.LinkGenerator> è un servizio singleton che può essere recuperato da [di](xref:fundamentals/dependency-injection). L'API può essere usata all'esterno del contesto di una richiesta in esecuzione. <xref:Microsoft.AspNetCore.Mvc.IUrlHelper> e gli scenari di MVC basati su <xref:Microsoft.AspNetCore.Mvc.IUrlHelper>, ad esempio [helper tag](xref:mvc/views/tag-helpers/intro), helper HTML e [risultati delle azioni](xref:mvc/controllers/actions), usano il generatore di collegamenti per offrire le funzionalità di generazione di collegamenti.

Il generatore di collegamenti si basa sui concetti di *indirizzo* e di *schemi di indirizzi*. Lo schema di indirizzi consente di determinare gli endpoint che devono essere considerati per la generazione dei collegamenti. Ad esempio, gli scenari relativi al nome della route e ai valori di route molti utenti hanno familiarità con MVC/ Razor pages sono implementati come uno schema di indirizzi.

Il generatore di collegamenti può essere collegato a Razor pagine e azioni MVC/pagine tramite i metodi di estensione seguenti:

* <xref:Microsoft.AspNetCore.Routing.ControllerLinkGeneratorExtensions.GetPathByAction*>
* <xref:Microsoft.AspNetCore.Routing.ControllerLinkGeneratorExtensions.GetUriByAction*>
* <xref:Microsoft.AspNetCore.Routing.PageLinkGeneratorExtensions.GetPathByPage*>
* <xref:Microsoft.AspNetCore.Routing.PageLinkGeneratorExtensions.GetUriByPage*>

Un overload di questi metodi accetta gli argomenti che includono `HttpContext`. Questi metodi sono funzionalmente equivalenti a `Url.Action` e `Url.Page` ma offrono una maggiore flessibilità e altre opzioni.

I metodi `GetPath*` sono più simili a `Url.Action` e `Url.Page` poiché generano un URI che contiene un percorso assoluto. I metodi `GetUri*` generano sempre un URI assoluto contenente uno schema e un host. I metodi che accettano `HttpContext` generano un URI nel contesto della richiesta in esecuzione. Se non vengono sovrascritti, vengono usati i valori di route di ambiente, il percorso base dell'URL, lo schema e l'host dalla richiesta in esecuzione.

<xref:Microsoft.AspNetCore.Routing.LinkGenerator> viene chiamato con un indirizzo. La generazione di un URI viene eseguita in due passaggi:

1. Un indirizzo viene associato a un elenco di endpoint che corrispondono all'indirizzo.
1. `RoutePattern` di ogni endpoint viene valutato fino a quando non viene individuato un formato di route che corrisponde ai valori specificati. L'output risultante viene unito alle altre parti dell'URI specificate nel generatore di collegamenti e restituito.

I metodi forniti da <xref:Microsoft.AspNetCore.Routing.LinkGenerator> supportano le funzionalità di generazione di collegamenti standard per tutti i tipi di indirizzi. È consigliabile usare il generatore di collegamenti tramite i metodi di estensione che eseguono le operazioni per un tipo di indirizzo specifico.

| Metodo di estensione   | Descrizione                                                         |
| ------------------ | ------------------------------------------------------------------- |
| <xref:Microsoft.AspNetCore.Routing.LinkGenerator.GetPathByAddress*> | Genera un URI con un percorso assoluto in base ai valori specificati. |
| <xref:Microsoft.AspNetCore.Routing.LinkGenerator.GetUriByAddress*> | Genera un URI assoluto in base ai valori specificati.             |

> [!WARNING]
> Prestare attenzione alle implicazioni seguenti della chiamata ai metodi <xref:Microsoft.AspNetCore.Routing.LinkGenerator>:
>
> * Usare i metodi di estensione `GetUri*` con cautela in una configurazione di app che non convalida l'intestazione `Host` delle richieste in ingresso. Se l'intestazione `Host` delle richieste in ingresso non viene convalidata, l'input delle richieste non attendibili può essere inviato nuovamente al client negli URI di una visualizzazione o pagina. È consigliabile che in tutte le app di produzione il server sia configurato per la convalida dell'intestazione `Host` rispetto ai valori validi noti.
>
> * Usare <xref:Microsoft.AspNetCore.Routing.LinkGenerator> con cautela nel middleware in associazione a `Map` o `MapWhen`. `Map*` modifica il percorso di base della richiesta in esecuzione, che ha effetto sull'output della generazione di collegamenti. Tutte le API <xref:Microsoft.AspNetCore.Routing.LinkGenerator> consentono di specificare un percorso di base. Specificare sempre un percorso di base vuoto per annullare l'effetto di `Map*` sulla generazione di collegamenti.

## <a name="differences-from-earlier-versions-of-routing"></a>Differenze rispetto alle versioni precedenti del routing

Tra il routing di endpoint in ASP.NET Core 2.2 o versioni successive e le versioni precedenti del routing in ASP.NET Core esistono alcune differenze:

* Il sistema di routing di endpoint non supporta l'estensibilità basata su <xref:Microsoft.AspNetCore.Routing.IRouter>, inclusa l'eredità da <xref:Microsoft.AspNetCore.Routing.Route>.

* Il routing di endpoint non supporta [WebApiCompatShim](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.WebApiCompatShim). Usare la [versione di compatibilità](xref:mvc/compatibility-version) 2,1 ( `.SetCompatibilityVersion(CompatibilityVersion.Version_2_1)` ) per continuare a usare lo shim di compatibilità.

* Il routing di endpoint ha un comportamento diverso per l'utilizzo delle maiuscole e delle minuscole degli URI generati quando si usano le route convenzionali.

  Esaminare il modello di route predefinito seguente:

  ```csharp
  app.UseMvc(routes =>
  {
      routes.MapRoute("default", "{controller=Home}/{action=Index}/{id?}");
  });
  ```

  Si supponga di generare un collegamento a un'azione usando la route seguente:

  ```csharp
  var link = Url.Action("ReadPost", "blog", new { id = 17, });
  ```

  Con il routing basato su <xref:Microsoft.AspNetCore.Routing.IRouter>, questo codice genera un URI di `/blog/ReadPost/17`, che rispetta le maiuscole e le minuscole del valore di route specificato. Il routing di endpoint in ASP.NET Core 2.2 o versioni successive produce `/Blog/ReadPost/17` ("Blog" viene convertita in lettere maiuscole). Il routing di endpoint offre l'interfaccia `IOutboundParameterTransformer` che può essere usata per personalizzare questo comportamento globalmente o per applicare convenzioni diverse per il mapping degli URL.

  Per altre informazioni, vedere la sezione [Riferimento ai trasformatori di parametro](#parameter-transformer-reference).

* La generazione di collegamenti utilizzata da MVC/ Razor pagine con route convenzionali si comporta in modo diverso quando si tenta di eseguire il collegamento a un controller, un'azione o una pagina che non esiste.

  Esaminare il modello di route predefinito seguente:

  ```csharp
  app.UseMvc(routes =>
  {
      routes.MapRoute("default", "{controller=Home}/{action=Index}/{id?}");
  });
  ```

  Si supponga di generare un collegamento a un'azione usando il modello predefinito con il codice seguente:

  ```csharp
  var link = Url.Action("ReadPost", "Blog", new { id = 17, });
  ```

  Con il routing basato su `IRouter`, il risultato è sempre `/Blog/ReadPost/17`, anche se `BlogController` non è presente o non ha un metodo di azione `ReadPost`. Come previsto, il routing di endpoint in ASP.NET Core 2.2 o versioni successive produce `/Blog/ReadPost/17` se è presente il metodo di azione. *Tuttavia, il routing di endpoint produce una stringa vuota se l'azione non è presente.* Concettualmente, il routing di endpoint non presume che l'endpoint sia presente se l'azione non è presente.

* L'*algoritmo di invalidamento dei valori di ambiente* della generazione di collegamenti ha un comportamento diverso quando viene usato con il routing di endpoint.

  L'*invalidamento dei valori di ambiente* è l'algoritmo che decide quali valori di route della richiesta in esecuzione (valori di ambiente) possono essere usati nelle operazioni di generazione di collegamenti. Il routing convenzionale ha sempre invalidato i valori di route aggiuntivi durante il collegamento a un'azione diversa. Prima del rilascio di ASP.NET Core 2.2, il routing degli attributi non aveva questo comportamento. Nelle versioni precedenti di ASP.NET Core i collegamenti a un'altra azione con gli stessi nomi di parametro di route causavano errori nella generazione dei collegamenti. In ASP.NET Core 2.2 o versioni successive entrambi i tipi di routing invalidano i valori in caso di collegamento a un'altra azione.

  Esaminare l'esempio seguente in ASP.NET Core 2.1 o versioni precedenti. In caso di collegamento a un'altra azione o a un'altra pagina, i valori di route possono essere riutilizzati in modi indesiderati.

  In */Pages/Store/Product.cshtml*:

  ```cshtml
  @page "{id}"
  @Url.Page("/Login")
  ```

  In */Pages/Login.cshtml*:

  ```cshtml
  @page "{id?}"
  ```

  Se l'URI è `/Store/Product/18` in ASP.NET Core 2.1 o versioni precedenti, il collegamento generato nella pagina Store/Info da `@Url.Page("/Login")` è `/Login/18`. Il valore `id` di 18 viene riutilizzato, anche se la destinazione del collegamento è una parte diversa dell'app. Il valore di route `id` nel contesto della pagina `/Login` è probabilmente un valore di ID utente, non un valore di ID prodotto del punto vendita.

  Nel routing di endpoint con ASP.NET Core 2.2 o versioni successive, il risultato è `/Login`. I valori di ambiente non vengono riutilizzati quando la destinazione collegata è costituita da un'azione o una pagina diversa.

* Sintassi dei parametri di route di round trip: le barre non vengono codificate quando viene usata una sintassi del parametro catch-all con doppio asterisco (`**`).

  Durante la generazione di collegamenti, il sistema di routing codifica il valore acquisito in un parametro catch-all con doppio asterisco (`**`) (ad esempio, `{**myparametername}`) escludendo le barre. Il parametro catch-all con doppio asterisco è supportato con il routing basato su `IRouter` in ASP.NET Core 2.2 o versioni successive.

  La sintassi del parametro catch-all con asterisco singolo nelle versioni precedenti di ASP.NET Core (`{*myparametername}`) resta supportata e le barre vengono codificate.

  | Route              | Collegamento generato con<br>`Url.Action(new { category = "admin/products" })`&hellip; |
  | ------------------ | --------------------------------------------------------------------- |
  | `/search/{*page}`  | `/search/admin%2Fproducts` (la barra viene codificata)             |
  | `/search/{**page}` | `/search/admin/products`                                              |

### <a name="middleware-example"></a>Esempio di middleware

Nell'esempio seguente un middleware usa l'API <xref:Microsoft.AspNetCore.Routing.LinkGenerator> per creare il collegamento a un metodo di azione che elenca i prodotti del punto vendita. L'uso del generatore di collegamenti tramite l'inserimento in una classe e la chiamata a `GenerateLink` è possibile in tutte le classi di un'app.

```csharp
using Microsoft.AspNetCore.Routing;

public class ProductsLinkMiddleware
{
    private readonly LinkGenerator _linkGenerator;

    public ProductsLinkMiddleware(RequestDelegate next, LinkGenerator linkGenerator)
    {
        _linkGenerator = linkGenerator;
    }

    public async Task InvokeAsync(HttpContext httpContext)
    {
        var url = _linkGenerator.GetPathByAction("ListProducts", "Store");

        httpContext.Response.ContentType = "text/plain";

        await httpContext.Response.WriteAsync($"Go to {url} to see our products.");
    }
}
```

### <a name="create-routes"></a>Creare route

La maggior parte delle app crea le route chiamando <xref:Microsoft.AspNetCore.Builder.MapRouteRouteBuilderExtensions.MapRoute*> o uno dei metodi di estensione simili definiti per <xref:Microsoft.AspNetCore.Routing.IRouteBuilder>. Tutti i metodi di estensione <xref:Microsoft.AspNetCore.Routing.IRouteBuilder> creano un'istanza di <xref:Microsoft.AspNetCore.Routing.Route> e la aggiungono alla raccolta di route.

<xref:Microsoft.AspNetCore.Builder.MapRouteRouteBuilderExtensions.MapRoute*> non accetta alcun parametro del gestore di route. <xref:Microsoft.AspNetCore.Builder.MapRouteRouteBuilderExtensions.MapRoute*> aggiunge solo le route che sono gestite da <xref:Microsoft.AspNetCore.Routing.RouteBuilder.DefaultHandler*>. Per altre informazioni sul routing in MVC, vedere <xref:mvc/controllers/routing>.

L'esempio di codice seguente si riferisce a una chiamata <xref:Microsoft.AspNetCore.Builder.MapRouteRouteBuilderExtensions.MapRoute*> usata da una tipica definizione di route di ASP.NET Core MVC:

```csharp
routes.MapRoute(
    name: "default",
    template: "{controller=Home}/{action=Index}/{id?}");
```

Questo modello confronta un percorso URL ed estrae i valori di route. Ad esempio, il percorso `/Products/Details/17` genera i valori di route seguenti: `{ controller = Products, action = Details, id = 17 }`.

I valori di route vengono determinati suddividendo il percorso URL in segmenti e confrontando ogni segmento con il nome del *parametro di route* nel modello di route. I parametri di route sono denominati. Parametri definiti racchiudendo il nome del parametro tra parentesi graffe `{ ... }`.

Il modello precedente può anche confrontare il percorso URL `/` e generare i valori `{ controller = Home, action = Index }`. Ciò accade perché i parametri di route `{controller}` e `{action}` hanno valori predefiniti e il parametro di route `id` è facoltativo. Un segno di uguale (`=`) seguito da un valore dopo il nome del parametro di route definisce un valore predefinito per il parametro. Un punto interrogativo (`?`) dopo il nome del parametro di route definisce un parametro facoltativo.

I parametri di route con un valore predefinito producono *sempre* un valore di route quando la route corrisponde. I parametri facoltativi non producono un valore di route se non è presente un segmento di percorso URL corrispondente. Vedere la sezione [Riferimento per i modelli di route](#route-template-reference) per una descrizione completa degli scenari e della sintassi del modello di route.

Nell'esempio seguente la definizione del parametro di route `{id:int}` definisce un [vincolo di route](#route-constraint-reference) per il parametro di route `id`:

```csharp
routes.MapRoute(
    name: "default",
    template: "{controller=Home}/{action=Index}/{id:int}");
```

Questo modello confronta un percorso URL come `/Products/Details/17`, ma non `/Products/Details/Apples`. I vincoli di route implementano <xref:Microsoft.AspNetCore.Routing.IRouteConstraint> ed esaminano i valori di route per verificarli. In questo esempio il valore di route `id` deve poter essere convertito in un numero intero. Vedere [Riferimento per i vincoli di route](#route-constraint-reference) per una descrizione dei vincoli di route specificati dal framework.

Altri overload di <xref:Microsoft.AspNetCore.Builder.MapRouteRouteBuilderExtensions.MapRoute*> accettano i valori per `constraints`, `dataTokens` e `defaults`. L'uso tipico di questi parametri è passare un oggetto tipizzato in modo anonimo, in cui i nomi di proprietà di tipo anonimo corrispondono ai nomi dei parametri di route.

Gli esempi <xref:Microsoft.AspNetCore.Builder.MapRouteRouteBuilderExtensions.MapRoute*> seguenti creano route equivalenti:

```csharp
routes.MapRoute(
    name: "default_route",
    template: "{controller}/{action}/{id?}",
    defaults: new { controller = "Home", action = "Index" });

routes.MapRoute(
    name: "default_route",
    template: "{controller=Home}/{action=Index}/{id?}");
```

> [!TIP]
> La sintassi inline per definire i vincoli e i valori predefiniti può essere utile per le route semplici. Esistono tuttavia scenari, come i token di dati, che non sono supportati dalla sintassi inline.

L'esempio seguente illustra alcuni scenari aggiuntivi:

```csharp
routes.MapRoute(
    name: "blog",
    template: "Blog/{**article}",
    defaults: new { controller = "Blog", action = "ReadArticle" });
```

Il modello precedente confronta un percorso URL come `/Blog/All-About-Routing/Introduction` ed estrae i valori `{ controller = Blog, action = ReadArticle, article = All-About-Routing/Introduction }`. I valori di route predefiniti per `controller` e `action` vengono generati dalla route anche se sono non presenti parametri di route corrispondenti nel modello. I valori predefiniti possono essere specificati nel modello di route. Il parametro di route `article` è definito come *catch-all* in base alla presenza di un doppio asterisco (`**`) prima del nome del parametro di route. I parametri di route catch-all acquisiscono il resto del percorso URL e possono anche corrispondere alla stringa vuota.

L'esempio seguente aggiunge i vincoli di route e i token di dati:

```csharp
routes.MapRoute(
    name: "us_english_products",
    template: "en-US/Products/{id}",
    defaults: new { controller = "Products", action = "Details" },
    constraints: new { id = new IntRouteConstraint() },
    dataTokens: new { locale = "en-US" });
```

Il modello precedente confronta un percorso URL come `/en-US/Products/5` ed estrae i valori `{ controller = Products, action = Details, id = 5 }` e i token di dati `{ locale = en-US }`.

![Token della finestra Variabili locali](routing/_static/tokens.png)

### <a name="route-class-url-generation"></a>Generazione di URL della classe di route

La classe <xref:Microsoft.AspNetCore.Routing.Route> è inoltre in grado di eseguire la generazione di URL combinando un set di valori di route con il relativo modello di route. Questo è logicamente il processo inverso di corrispondenza del percorso URL.

> [!TIP]
> Per comprendere meglio la generazione di URL, si pensi a quale URL si vuole generare e al modo in cui un modello di route deve corrispondere a tale URL. Quali valori devono essere prodotti? Questo equivale approssimativamente al funzionamento della generazione di URL nella classe <xref:Microsoft.AspNetCore.Routing.Route>.

L'esempio seguente usa una route predefinita di ASP.NET Core MVC generale:

```csharp
routes.MapRoute(
    name: "default",
    template: "{controller=Home}/{action=Index}/{id?}");
```

Con i valori di route `{ controller = Products, action = List }`, viene generato l'URL `/Products/List`. I valori di route vengono sostituiti dai parametri di route corrispondenti per formare il percorso URL. Poiché `id` è un parametro di route facoltativo, l'URL è stato generato senza un valore per `id`.

Con i valori di route `{ controller = Home, action = Index }`, viene generato l'URL `/`. Poiché i valori di route specificati corrispondono ai valori predefiniti, i segmenti corrispondenti ai valori predefiniti possono essere omessi.

Per entrambi gli URL generati viene eseguito il round trip con la definizione di route seguente (`/Home/Index` e `/`) e vengono prodotti gli stessi valori di route usati per generare l'URL.

> [!NOTE]
> Un'app che usa ASP.NET Core MVC deve usare <xref:Microsoft.AspNetCore.Mvc.Routing.UrlHelper> per generare gli URL invece di eseguire la chiamata direttamente nel routing.

Per altre informazioni sulla generazione di URL, vedere la sezione [Riferimento per la generazione di URL](#url-generation-reference).

## <a name="use-routing-middleware"></a>Usare il middleware di routing

Fare riferimento al [metapacchetto Microsoft.AspNetCore.App](xref:fundamentals/metapackage-app) nel file di progetto dell'app.

Aggiungere il routing al contenitore del servizio in `Startup.ConfigureServices`:

[!code-csharp[](routing/samples/2.x/RoutingSample/Startup.cs?name=snippet_ConfigureServices&highlight=3)]

Le route devono essere configurate nel metodo `Startup.Configure`. L'app di esempio usa le API seguenti:

* <xref:Microsoft.AspNetCore.Routing.RouteBuilder>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapGet*>: Corrisponde solo alle richieste HTTP GET.
* <xref:Microsoft.AspNetCore.Builder.RoutingBuilderExtensions.UseRouter*>

[!code-csharp[](routing/samples/2.x/RoutingSample/Startup.cs?name=snippet_RouteHandler)]

Nella tabella seguente sono elencate le risposte con gli URI specificati.

| URI                    | Risposta                                          |
| ---------------------- | ------------------------------------------------- |
| `/package/create/3`    | Hello! Route values: [operation, create], [id, 3] |
| `/package/track/-3`    | Hello! Route values: [operation, track], [id, -3] |
| `/package/track/-3/`   | Hello! Route values: [operation, track], [id, -3] |
| `/package/track/`      | La richiesta non viene eseguita, nessuna corrispondenza.              |
| `GET /hello/Joe`       | Hi, Joe!                                          |
| `POST /hello/Joe`      | La richiesta non viene eseguita, corrispondenza solo con HTTP GET. |
| `GET /hello/Joe/Smith` | La richiesta non viene eseguita, nessuna corrispondenza.              |

Il framework offre un set di metodi di estensione per la creazione di route (<xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions>):

* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapDelete*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapGet*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapMiddlewareDelete*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapMiddlewareGet*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapMiddlewarePost*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapMiddlewarePut*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapMiddlewareRoute*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapMiddlewareVerb*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapPost*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapPut*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapRoute*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapVerb*>

I metodi `Map[Verb]` usano i vincoli per limitare la route al verbo HTTP nel nome del metodo. Vedere ad esempio <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapGet*> e <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapVerb*>.

## <a name="route-template-reference"></a>Riferimento per il modello di route

I token all'interno di parentesi graffe (`{ ... }`) definiscono i *parametri di route* che vengono associati se esiste una corrispondenza per la route. È possibile definire più parametri di route in un segmento di route, ma devono essere separati da un valore letterale. Ad esempio, `{controller=Home}{action=Index}` non è una route valida perché non è presente un valore letterale tra `{controller}` e `{action}`. Questi parametri di route devono avere un nome e possono avere attributi aggiuntivi.

Il testo letterale diverso dai parametri di route, ad esempio `{id}`, e il separatore di percorso `/` devono corrispondere al testo nell'URL. La corrispondenza del testo non fa distinzione tra maiuscole e minuscole e si basa sulla rappresentazione decodificata del percorso degli URL. Per verificare la corrispondenza di un delimitatore letterale dei parametri di route (`{` o `}`), eseguire l'escape del delimitatore ripetendo il carattere (`{{` o `}}`).

I modelli di URL che tentano di acquisire un nome file con un'estensione facoltativa hanno considerazioni aggiuntive. Considerare ad esempio il modello `files/{filename}.{ext?}`. Se esistono i valori per `filename` e `ext`, vengono popolati entrambi i valori. Se esiste solo un valore per `filename` nell'URL, la route corrisponde perché il punto finale (`.`) è facoltativo. Gli URL seguenti corrispondono a questa route:

* `/files/myFile.txt`
* `/files/myFile`

È possibile usare un asterisco (`*`) o un doppio asterisco (`**`) come prefisso per un parametro di route da associare alla parte rimanente dell'URI. Si tratta di parametri chiamati parametri *catch-all*. Ad esempio, `blog/{**slug}` corrisponde a qualsiasi URI che inizia con `/blog` seguito da qualsiasi valore, che viene assegnato al valore di route `slug`. I parametri catch-all possono anche corrispondere alla stringa vuota.

Il parametro catch-all esegue l'escape dei caratteri appropriati, incluso il separatore di percorso (`/`), quando la route viene usata per generare un URL. Ad esempio, la route `foo/{*path}` con i valori di route `{ path = "my/path" }` genera `foo/my%2Fpath`. Si noti la barra di escape. Per eseguire il round trip dei caratteri di separatore di percorso, usare il prefisso del parametro di route `**`. La route `foo/{**path}` con `{ path = "my/path" }` genera `foo/my/path`.

I parametri di route possono avere *valori predefiniti*, definiti specificando il valore predefinito dopo il nome del parametro, separato da un segno di uguale (`=`). Ad esempio, `{controller=Home}` definisce `Home` come valore predefinito per `controller`. Il valore predefinito viene usato se nell'URL non è presente alcun valore per il parametro. I parametri di route vengono resi facoltativi aggiungendo un punto interrogativo (`?`) alla fine del nome del parametro, come in `id?`. La differenza tra parametri facoltativi e parametri di route predefiniti è che un parametro di route con un valore predefinito produce sempre un valore, mentre un parametro facoltativo ha un valore solo se ne viene specificato uno dall'URL della richiesta.

I parametri di route possono presentare dei vincoli che devono corrispondere al valore di route associato dall'URL. Aggiungendo due punti (`:`) e il nome del vincolo dopo il nome del parametro di route, si specifica un *vincolo inline* per un parametro di route. Se il vincolo richiede argomenti, vengono racchiusi tra parentesi (`(...)`) dopo il nome del vincolo. È possibile specificare più vincoli inline aggiungendo di nuovo i due punti (`:`) e il nome del vincolo.

Il nome del vincolo e gli argomenti vengono passati al servizio <xref:Microsoft.AspNetCore.Routing.IInlineConstraintResolver> per creare un'istanza di <xref:Microsoft.AspNetCore.Routing.IRouteConstraint> da usare nell'elaborazione dell'URL. Il modello di route `blog/{article:minlength(10)}` specifica ad esempio un vincolo `minlength` con l'argomento `10`. Per altre informazioni sui vincoli di route e per un elenco dei vincoli specificati dal framework, vedere la sezione [Riferimento per i vincoli di route](#route-constraint-reference).

I parametri di route possono avere anche trasformatori di parametro che trasformano un valore di parametro durante la generazione dei collegamenti e l'abbinamento delle azioni e delle pagine agli URI. Come i vincoli, i trasformatori di parametro possono essere aggiunti inline a un parametro di route inserendo due punti (`:`) e il nome del trasformatore dopo il nome del parametro di route. Ad esempio, il modello di route `blog/{article:slugify}` specifica un trasformatore `slugify`. Per altre informazioni sui trasformatori di parametro, vedere la sezione [Riferimento ai trasformatori di parametro](#parameter-transformer-reference).

La tabella seguente illustra i modelli di route di esempio e il relativo comportamento.

| Modello di route                           | URI corrispondente di esempio    | L'URI della richiesta&hellip;                                                    |
| ---------------------------------------- | ----------------------- | -------------------------------------------------------------------------- |
| `hello`                                  | `/hello`                | Verifica la corrispondenza solo del singolo percorso `/hello`.                                     |
| `{Page=Home}`                            | `/`                     | Verifica la corrispondenza e imposta `Page` su `Home`.                                         |
| `{Page=Home}`                            | `/Contact`              | Verifica la corrispondenza e imposta `Page` su `Contact`.                                      |
| `{controller}/{action}/{id?}`            | `/Products/List`        | Esegue il mapping al controller `Products` e all'azione `List`.                       |
| `{controller}/{action}/{id?}`            | `/Products/Details/123` | Esegue il mapping al controller `Products` e all'azione `Details` (`id` impostato su 123). |
| `{controller=Home}/{action=Index}/{id?}` | `/`                     | Esegue il mapping al controller `Home` e al metodo `Index` (`id` viene ignorato).        |

L'uso di un modello è in genere l'approccio più semplice al routing. I vincoli e le impostazioni predefinite possono essere specificati anche all'esterno del modello di route.

> [!TIP]
> Abilitare la [registrazione](xref:fundamentals/logging/index) per verificare in che modo le implementazioni del routing predefinite, ad esempio <xref:Microsoft.AspNetCore.Routing.Route>, corrispondono alle richieste.

## <a name="reserved-routing-names"></a>Nomi riservati di routing

Le parole chiave seguenti sono nomi riservati e non possono essere usate come nomi o parametri di route:

* `action`
* `area`
* `controller`
* `handler`
* `page`

## <a name="route-constraint-reference"></a>Riferimento per i vincoli di route

I vincoli di route vengono eseguiti quando si verifica una corrispondenza nell'URL in ingresso e il percorso URL viene suddiviso in valori di route in formato token. I vincoli di route in genere controllano il valore di route associato usando il modello di route e stabiliscono con una decisione Sì/No se il valore è accettabile o meno. Alcuni vincoli di route usano i dati all'esterno del valore di route per stabilire se la richiesta può essere instradata. Ad esempio, <xref:Microsoft.AspNetCore.Routing.Constraints.HttpMethodRouteConstraint> può accettare o rifiutare una richiesta in base al relativo verbo HTTP. I vincoli vengono usati nelle richieste del routing e nella generazione di collegamenti.

> [!WARNING]
> Non usare i vincoli per la **convalida dell'input**. Se vengono usati vincoli per la **convalida dell'input**, un input non valido causa la visualizzazione di un errore di tipo *404 (Non trovato)* invece di un errore di tipo *400 - Richiesta non valida* con un messaggio di errore appropriato. I vincoli di route vengono usati per evitare **ambiguità** tra route simili, non per convalidare gli input per una route specifica.

La tabella seguente illustra i vincoli di route di esempio e il relativo comportamento.

| Vincolo | Esempio | Corrispondenze di esempio | Note |
|------------|---------|-----------------|-------|
| `int` | `{id:int}` | `123456789`, `-123456789` | Corrisponde a qualsiasi numero intero.|
| `bool` | `{active:bool}` | `true`, `FALSE` | Corrisponde a `true` o `false` . Senza distinzione tra maiuscole e minuscole.|
| `datetime` | `{dob:datetime}` | `2016-12-31`, `2016-12-31 7:32pm` | Corrisponde a un `DateTime` valore valido della lingua inglese. Vedere l'avviso precedente.|
| `decimal` | `{price:decimal}` | `49.99`, `-1,000.01` | Corrisponde a un `decimal` valore valido della lingua inglese. Vedere l'avviso precedente.|
| `double` | `{weight:double}` | `1.234`, `-1,001.01e8` | Corrisponde a un `double` valore valido della lingua inglese. Vedere l'avviso precedente.|
| `float` | `{weight:float}` | `1.234`, `-1,001.01e8` | Corrisponde a un `float` valore valido della lingua inglese. Vedere l'avviso precedente.|
| `guid` | `{id:guid}` | `CD2C1638-1638-72D5-1638-DEADBEEF1638`, `{CD2C1638-1638-72D5-1638-DEADBEEF1638}` | Corrisponde a un `Guid` valore valido.|
| `long` | `{ticks:long}` | `123456789`, `-123456789` | Corrisponde a un `long` valore valido.|
| `minlength(value)` | `{username:minlength(4)}` | `Rick` | La stringa deve contenere almeno 4 caratteri.|
| `maxlength(value)` | `{filename:maxlength(8)}` | `MyFile` | La stringa ha un massimo di 8 caratteri.|
| `length(length)` | `{filename:length(12)}` | `somefile.txt` | La lunghezza della stringa deve essere uguale a 12 caratteri.|
| `length(min,max)` | `{filename:length(8,16)}` | `somefile.txt` | La stringa deve essere almeno 8 e può contenere un massimo di 16 caratteri.|
| `min(value)` | `{age:min(18)}` | `19` | Il valore intero deve essere almeno 18.|
| `max(value)` | `{age:max(120)}` | `91` | Valore intero massimo di 120.|
| `range(min,max)` | `{age:range(18,120)}` | `91` | Il valore intero deve essere almeno 18 e un massimo di 120.|
| `alpha` | `{name:alpha}` | `Rick` | La stringa deve essere costituita da uno o più caratteri alfabetici `a` - `z` . Senza distinzione tra maiuscole e minuscole.|
| `regex(expression)` | `{ssn:regex(^\\d{{3}}-\\d{{2}}-\\d{{4}}$)}` | `123-45-6789` | La stringa deve corrispondere all'espressione regolare. Vedere suggerimenti sulla definizione di un'espressione regolare.|
| `required` | `{name:required}` | `Rick` | Utilizzato per imporre che un valore non di parametro sia presente durante la generazione dell'URL.|

Più vincoli delimitati da punti possono essere applicati a un singolo parametro. Ad esempio il vincolo seguente limita un parametro a un valore intero maggiore o uguale a 1:

```csharp
[Route("users/{id:int:min(1)}")]
public User GetUserById(int id) { }
```

> [!WARNING]
> I vincoli di route che verificano l'URL e vengono convertiti in un tipo CLR, ad esempio `int` o `DateTime`, usano sempre le impostazioni cultura inglese non dipendenti da paese/area geografica, presupponendo che l'URL sia non localizzabile. I vincoli di route specificati dal framework non modificano i valori archiviati nei valori di route. Tutti i valori di route analizzati dall'URL vengono archiviati come stringhe. Ad esempio, il vincolo `float` prova a convertire il valore di route in un valore float, ma il valore convertito viene usato solo per verificare che può essere convertito in un valore float.

## <a name="regular-expressions"></a>Espressioni regolari

Il framework di ASP.NET Core aggiunge `RegexOptions.IgnoreCase | RegexOptions.Compiled | RegexOptions.CultureInvariant` al costruttore di espressioni regolari. Per una descrizione di questi membri, vedere <xref:System.Text.RegularExpressions.RegexOptions>.

Le espressioni regolari usano delimitatori e token simili a quelli usati dal routing e dal linguaggio C#. Per i token di espressione è necessario aggiungere caratteri di escape. Per utilizzare l'espressione regolare `^\d{3}-\d{2}-\d{4}$` nel routing:

* L'espressione deve contenere i caratteri della barra rovesciata singola `\` specificati nella stringa come doppi caratteri di barra rovesciata `\\` nel codice sorgente.
* Per `\\` eseguire l'escape del `\` carattere di escape della stringa, è necessario l'espressione regolare.
* L'espressione regolare non richiede `\\` quando si usano [valori letterali stringa Verbatim](/dotnet/csharp/language-reference/keywords/string).

Per eseguire l'escape dei caratteri di delimitazione del parametro di routing,,, `{` `}` `[` `]` , raddoppiare i caratteri nell'espressione `{{` , `}` `[[` `]]` ,,. La tabella seguente illustra un'espressione regolare e la versione di escape:

| Espressione regolare    | Espressione regolare con caratteri di escape     |
| --------------------- | ------------------------------ |
| `^\d{3}-\d{2}-\d{4}$` | `^\\d{{3}}-\\d{{2}}-\\d{{4}}$` |
| `^[a-z]{2}$`          | `^[[a-z]]{{2}}$`               |

Le espressioni regolari utilizzate nel routing spesso iniziano con il `^` carattere del punto di inserimento e corrispondono alla posizione iniziale della stringa. Le espressioni spesso terminano con il `$` carattere di segno di dollaro e la fine della corrispondenza della stringa. I caratteri `^` e `$` consentono di verificare che l'espressione regolare corrisponda all'intero valore del parametro di route. Senza i caratteri `^` e `$` l'espressione regolare corrisponde a qualsiasi sottostringa all'interno della stringa e spesso questo non è il risultato desiderato. La tabella seguente include alcuni esempi e descrive perché si verifica o non si verifica la corrispondenza.

| Expression   | string    | Corrispondenza | Commento               |
| ------------ | --------- | :---: |  -------------------- |
| `[a-z]{2}`   | hello     | Sì   | Corrispondenze di sottostringhe     |
| `[a-z]{2}`   | 123abc456 | Sì   | Corrispondenze di sottostringhe     |
| `[a-z]{2}`   | mz        | Sì   | Corrisponde all'espressione    |
| `[a-z]{2}`   | MZ        | Sì   | Senza distinzione maiuscole/minuscole    |
| `^[a-z]{2}$` | hello     | No    | Vedere `^` e `$` sopra |
| `^[a-z]{2}$` | 123abc456 | No    | Vedere `^` e `$` sopra |

Per altre informazioni sulla sintassi delle espressioni regolari, vedere [Espressioni regolari di .NET Framework](/dotnet/standard/base-types/regular-expression-language-quick-reference).

Per limitare un parametro a un set noto di valori possibili, usare un'espressione regolare. Ad esempio, `{action:regex(^(list|get|create)$)}` verifica la corrispondenza del valore di route `action` solo con `list`, `get` o `create`. Se viene passata nel dizionario di vincoli, la stringa `^(list|get|create)$` è equivalente. Anche i vincoli passati nel dizionario di vincoli (non inline all'interno di un modello) che non corrispondono a uno dei vincoli noti vengono considerati espressioni regolari.

## <a name="custom-route-constraints"></a>Vincoli di Route personalizzati

Oltre ai vincoli di route predefiniti, è possibile creare vincoli di route personalizzati implementando l'interfaccia <xref:Microsoft.AspNetCore.Routing.IRouteConstraint>. L'interfaccia <xref:Microsoft.AspNetCore.Routing.IRouteConstraint> contiene un singolo metodo, `Match`, che restituisce `true` se il vincolo viene soddisfatto e `false` in caso contrario.

Per usare un'interfaccia <xref:Microsoft.AspNetCore.Routing.IRouteConstraint> personalizzata, il tipo di vincolo di route deve essere registrato con la proprietà <xref:Microsoft.AspNetCore.Routing.RouteOptions.ConstraintMap> dell'app nel contenitore di servizi dell'app. <xref:Microsoft.AspNetCore.Routing.RouteOptions.ConstraintMap> è un dizionario che esegue il mapping delle chiavi dei vincoli di route alle implementazioni di <xref:Microsoft.AspNetCore.Routing.IRouteConstraint> che convalidano tali vincoli. La proprietà <xref:Microsoft.AspNetCore.Routing.RouteOptions.ConstraintMap> di un'app può essere aggiornata in `Startup.ConfigureServices` come parte di una chiamata [services.AddRouting](xref:Microsoft.Extensions.DependencyInjection.RoutingServiceCollectionExtensions.AddRouting*) o configurando <xref:Microsoft.AspNetCore.Routing.RouteOptions> direttamente con `services.Configure<RouteOptions>`. Ad esempio:

```csharp
services.AddRouting(options =>
{
    options.ConstraintMap.Add("customName", typeof(MyCustomConstraint));
});
```

Il vincolo può quindi essere applicato alle route nel modo consueto, usando il nome specificato al momento della registrazione del tipo di vincolo. Ad esempio:

```csharp
[HttpGet("{id:customName}")]
public ActionResult<string> Get(string id)
```

## <a name="parameter-transformer-reference"></a>Riferimento ai trasformatori di parametro

I trasformatori di parametro:

* Vengono eseguiti quando si genera un collegamento per <xref:Microsoft.AspNetCore.Routing.Route>.
* Implementare `Microsoft.AspNetCore.Routing.IOutboundParameterTransformer`.
* Vengono configurati tramite <xref:Microsoft.AspNetCore.Routing.RouteOptions.ConstraintMap>.
* Acquisire il valore della route del parametro e trasformarlo in un nuovo valore stringa.
* Il valore trasformato viene usato nel collegamento generato.

Ad esempio, un trasformatore di parametro `slugify` personalizzato nel modello di route `blog\{article:slugify}` con `Url.Action(new { article = "MyTestArticle" })` genera `blog\my-test-article`.

Per usare un trasformatore di parametro in un modello di route, configurarlo prima usando <xref:Microsoft.AspNetCore.Routing.RouteOptions.ConstraintMap> in `Startup.ConfigureServices`:

```csharp
services.AddRouting(options =>
{
    // Replace the type and the name used to refer to it with your own
    // IOutboundParameterTransformer implementation
    options.ConstraintMap["slugify"] = typeof(SlugifyParameterTransformer);
});
```

I trasformatori di parametro sono usati dai framework per trasformare l'URI in cui viene risolto un endpoint. Ad esempio, ASP.NET Core MVC usa i trasformatori di parametro per trasformare il valore di route usato per abbinare `area`, `controller`, `action` e `page`.

```csharp
routes.MapRoute(
    name: "default",
    template: "{controller:slugify=Home}/{action:slugify=Index}/{id?}");
```

Con la route precedente, l'azione `SubscriptionManagementController.GetAll` viene abbinata all'URI `/subscription-management/get-all`. Un trasformatore di parametro non modifica i valori della route usati per generare un collegamento. Ad esempio, `Url.Action("GetAll", "SubscriptionManagement")` restituisce `/subscription-management/get-all`.

ASP.NET Core offre convenzioni API per l'uso di trasformatori di parametro con le route generate:

* ASP.NET Core MVC include la convenzione API `Microsoft.AspNetCore.Mvc.ApplicationModels.RouteTokenTransformerConvention`. Questa convenzione applica un trasformatore di parametro specifico a tutte le route di attributi nell'app. Il trasformatore di parametro trasforma i token di route di attributi man mano che vengono sostituiti. Per altre informazioni vedere [Use a parameter transformer to customize token replacement](xref:mvc/controllers/routing#use-a-parameter-transformer-to-customize-token-replacement) (Usare un trasformatore di parametro per personalizzare la sostituzione di token).
* Razor Le pagine hanno la `Microsoft.AspNetCore.Mvc.ApplicationModels.PageRouteTransformerConvention` convenzione API. Questa convenzione applica un trasformatore di parametro specificato a tutte le pagine individuate automaticamente Razor . Il trasformatore di parametri trasforma la cartella e i segmenti dei nomi di file delle Razor Route di pagine. Per altre informazioni, vedere [Use a parameter transformer to customize page routes](xref:razor-pages/razor-pages-conventions#use-a-parameter-transformer-to-customize-page-routes) (Usare un trasformatore di parametro per personalizzare route di pagine).

## <a name="url-generation-reference"></a>Riferimento per la generazione di URL

L'esempio seguente mostra come generare un collegamento a una route usando un dizionario di valori di route e un elemento <xref:Microsoft.AspNetCore.Routing.RouteCollection>.

[!code-csharp[](routing/samples/2.x/RoutingSample/Startup.cs?name=snippet_Dictionary)]

L'elemento <xref:Microsoft.AspNetCore.Routing.VirtualPathData.VirtualPath> generato alla fine dell'esempio precedente è `/package/create/123`. Il dizionario fornisce i valori di route `operation` e `id` del modello "Track Package Route", `package/{operation}/{id}`. Per informazioni dettagliate, vedere il codice di esempio nella sezione [Usare il middleware di routing](#use-routing-middleware) oppure l'[app di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/routing/samples).

Il secondo parametro per il costruttore <xref:Microsoft.AspNetCore.Routing.VirtualPathContext> è una raccolta di *valori di ambiente*. I valori di ambiente sono utili poiché limitano il numero di valori che uno sviluppatore deve specificare all'interno del contesto di una richiesta. I valori di route correnti della richiesta corrente sono considerati valori di ambiente per la generazione del collegamento. Nell'azione `About` di un'app ASP.NET Core MVC di `HomeController`, non è necessario specificare il valore di route del controller per collegarsi all'azione `Index`. Viene usato il valore di ambiente `Home`.

I valori di ambiente che non corrispondono a un parametro vengono ignorati. I valori di ambiente vengono ignorati anche quando un valore specificato in modo esplicito esegue l'override del valore di ambiente. La corrispondenza si verifica da sinistra a destra nell'URL.

I valori specificati in modo esplicito ma che non corrispondono a un segmento della route vengono aggiunti alla stringa di query. La tabella seguente illustra il risultato ottenuto quando si usa il modello di route `{controller}/{action}/{id?}`.

| Valori di ambiente                     | Valori espliciti                        | Risultato                  |
| ---------------------------------- | -------------------------------------- | ----------------------- |
| controller = "Home"                | action = "About"                       | `/Home/About`           |
| controller = "Home"                | controller = "Order", action = "About" | `/Order/About`          |
| controller = "Home", color = "Red" | action = "About"                       | `/Home/About`           |
| controller = "Home"                | action = "About", color = "Red"        | `/Home/About?color=Red` |

Se una route ha un valore predefinito che non corrisponde a un parametro e tale valore viene specificato in modo esplicito, deve corrispondere al valore predefinito:

```csharp
routes.MapRoute("blog_route", "blog/{*slug}",
    defaults: new { controller = "Blog", action = "ReadPost" });
```

La generazione dei collegamenti genera un collegamento per questa route quando vengono specificati i valori corrispondenti per `controller` e `action`.

## <a name="complex-segments"></a>Segmenti complessi

I segmenti complessi (ad esempio, `[Route("/x{token}y")]`), vengono elaborati individuando corrispondenze per i valori letterali da destra a sinistra in modalità non-greedy. Vedere [questo codice](https://github.com/dotnet/aspnetcore/blob/v2.2.13/src/Http/Routing/src/Patterns/RoutePatternMatcher.cs#L293) per una spiegazione dettagliata di come vengono confrontati i segmenti complessi. L'[esempio di codice](https://github.com/dotnet/aspnetcore/blob/v2.2.13/src/Http/Routing/src/Patterns/RoutePatternMatcher.cs#L293) non viene usato da ASP.NET Core, ma offre una spiegazione esauriente dei segmenti complessi.
<!-- While that code is no longer used by ASP.NET Core for complex segment matching, it provides a good match to the current algorithm. The [current code](https://github.com/dotnet/AspNetCore/blob/91514c9af7e0f4c44029b51f05a01c6fe4c96e4c/src/Http/Routing/src/Matching/DfaMatcherBuilder.cs#L227-L244) is too abstracted from matching to be useful for understanding complex segment matching.
-->

::: moniker-end

::: moniker range="< aspnetcore-2.2"

Il routing è responsabile del mapping degli URI delle richieste ai gestori di route e dell'invio delle richieste in ingresso. Le route sono definite nell'app e vengono configurate all'avvio dell'app. Una route può facoltativamente estrarre valori dall'URL contenuto nella richiesta e questi valori possono quindi essere usati per l'elaborazione della richiesta. Con l'uso di route configurate dall'app, il routing è in grado di generare gli URL che eseguono il mapping ai gestori di route.

Per usare gli scenari di routing più recenti in ASP.NET Core 2.1, specificare la [versione di compatibilità](xref:mvc/compatibility-version) nella registrazione dei servizi MVC in `Startup.ConfigureServices`:

```csharp
services.AddMvc()
    .SetCompatibilityVersion(CompatibilityVersion.Version_2_1);
```

> [!IMPORTANT]
> Questo documento descrive il routing di basso livello di ASP.NET Core. Per informazioni sul routing di ASP.NET Core MVC, vedere <xref:mvc/controllers/routing>. Per informazioni sulle convenzioni di routing nelle Razor pagine, vedere <xref:razor-pages/razor-pages-conventions> .

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/routing/samples) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="routing-basics"></a>Nozioni fondamentali sul routing

La maggior parte delle app dovrebbe scegliere uno schema di routing semplice e descrittivo in modo che gli URL siano leggibili e significativi. La route convenzionale predefinita `{controller=Home}/{action=Index}/{id?}`:

* Supporta uno schema di routing semplice e descrittivo.
* È un punto iniziale utile per le app basate su interfaccia utente.

Gli sviluppatori solitamente aggiungono ulteriori route brevi nelle aree a traffico elevato dell'app in situazioni specifiche (ad esempio, endpoint di blog e e-commerce) usando il [routing degli attributi](xref:mvc/controllers/routing#attribute-routing) o route convenzionali dedicate.

Le API Web devono usare il routing degli attributi per modellare le funzionalità dell'app come set di risorse in cui le operazioni sono rappresentate da verbi HTTP. Questo significa che molte operazioni (ad esempio, GET, POST) sulla stessa risorsa logica useranno lo stesso URL. Il routing degli attributi offre un livello di controllo necessario per progettare con attenzione il layout dell'endpoint pubblico di un'API.

Razor Le app di pagine usano il routing convenzionale predefinito per gestire le risorse denominate nella cartella *pagine* di un'app. Sono disponibili convenzioni aggiuntive che consentono di personalizzare il Razor comportamento di routing delle pagine. Per altre informazioni, vedere <xref:razor-pages/index> e <xref:razor-pages/razor-pages-conventions>.

Il supporto della generazione di URL consente di sviluppare l'app senza URL hardcoded per collegare l'app. Il supporto consente di iniziare con una configurazione di routing di base e di modificare le route dopo aver determinato il layout delle risorse dell'app.

Il routing usa le implementazioni di route di <xref:Microsoft.AspNetCore.Routing.IRouter> a:

* Eseguire il mapping di richieste in ingresso ai *gestori di route*.
* Generare gli URL usati nelle risposte.

Per impostazione predefinita, un'app ha una sola raccolta di route. Quando arriva una richiesta, le route della raccolta vengono elaborate nell'ordine in cui si trovano nella raccolta. Il framework tenta di associare l'URL di una richiesta in ingresso a una route della raccolta chiamando il metodo <xref:Microsoft.AspNetCore.Routing.IRouter.RouteAsync*> in ogni route della raccolta. Una risposta può usare il routing per generare gli URL, ad esempio per il reindirizzamento o i collegamenti, in base alle informazioni delle route. In questo modo si evita di impostare gli URL come hardcoded ottimizzando la manutenibilità.

Il sistema di routing ha le caratteristiche seguenti:

* La sintassi del modello di route viene usata per definire le route con parametri di route in formato token.
* La configurazione degli endpoint basata su convenzioni e attributi è consentita.
* <xref:Microsoft.AspNetCore.Routing.IRouteConstraint> consente di determinare se un parametro URL contiene un valore valido per il vincolo di un endpoint specifico.
* I modelli di app, ad esempio MVC/ Razor pages, registrano tutte le route, che hanno un'implementazione prevedibile degli scenari di routing.
* Una risposta può usare il routing per generare gli URL, ad esempio per il reindirizzamento o i collegamenti, in base alle informazioni delle route. In questo modo si evita di impostare gli URL come hardcoded ottimizzando la manutenibilità.
* La generazione degli URL è basata sulle route, che supportano l'estensibilità arbitraria. <xref:Microsoft.AspNetCore.Mvc.IUrlHelper> offre metodi per generare gli URL.
<!-- fix [middleware](xref:fundamentals/middleware/index) -->
Il routing è connesso alla pipeline [middleware](xref:fundamentals/middleware/index) per mezzo della classe <xref:Microsoft.AspNetCore.Builder.RouterMiddleware>. [ASP.NET Core MVC](xref:mvc/overview) aggiunge il routing alla pipeline middleware come parte della configurazione e gestisce il routing nelle app MVC e Razor pages. Per informazioni sull'uso del routing come componente autonomo, vedere la sezione [Usare il middleware di routing](#use-routing-middleware).

### <a name="url-matching"></a>Corrispondenza URL

La corrispondenza dell'URL è il processo con cui il routing invia una richiesta in ingresso a un *gestore*. Questo processo si basa sui dati presenti nel percorso URL, ma può essere esteso in modo da prendere in considerazione tutti i dati della richiesta. La possibilità di inviare le richieste a gestori separati è fondamentale per ridurre le dimensioni e la complessità di un'app.

Le richieste in ingresso immettono <xref:Microsoft.AspNetCore.Builder.RouterMiddleware>, che chiama il metodo <xref:Microsoft.AspNetCore.Routing.IRouter.RouteAsync*> per ogni route della sequenza. L'istanza di <xref:Microsoft.AspNetCore.Routing.IRouter> sceglie se *gestire* la richiesta impostando [RouteContext.Handler](xref:Microsoft.AspNetCore.Routing.RouteContext.Handler*) su un valore <xref:Microsoft.AspNetCore.Http.RequestDelegate> non Null. Se una route imposta un gestore per la richiesta, l'elaborazione della route si interrompe e viene richiamato il gestore per elaborare la richiesta. Se non viene individuato alcun gestore di route per elaborare la richiesta, il middleware passa la richiesta al middleware successivo nella pipeline delle richieste.

L'input principale per <xref:Microsoft.AspNetCore.Routing.IRouter.RouteAsync*> è l'elemento [RouteContext.HttpContext](xref:Microsoft.AspNetCore.Routing.RouteContext.HttpContext*) associato alla richiesta corrente. [RouteContext.Handler](xref:Microsoft.AspNetCore.Routing.RouteContext.Handler) e [RouteContext.RouteData](xref:Microsoft.AspNetCore.Routing.RouteContext.RouteData*) sono gli output che vengono impostati quando una route corrisponde.

Una corrispondenza che chiama <xref:Microsoft.AspNetCore.Routing.IRouter.RouteAsync*> imposta anche le proprietà di [RouteContext.RouteData](xref:Microsoft.AspNetCore.Routing.RouteContext.RouteData) sui valori appropriati in base all'elaborazione della richiesta eseguita fino a quel punto.

[RouteData.Values](xref:Microsoft.AspNetCore.Routing.RouteData.Values*) è un dizionario di *valori di route* prodotti dalla route. Questi valori in genere sono determinati dalla suddivisione in token dell'URL e possono essere usati per accettare l'input dell'utente o per prendere ulteriori decisioni riguardo all'invio all'interno dell'app.

[RouteData.DataTokens](xref:Microsoft.AspNetCore.Routing.RouteData.DataTokens*) è un elenco proprietà dei dati aggiuntivi correlati alla route corrispondente. Gli elementi <xref:Microsoft.AspNetCore.Routing.RouteData.DataTokens*> vengono forniti per supportare l'associazione dei dati sullo stato con ogni route in modo che l'app sia in grado di prendere decisioni in base alla route corrispondente. Questi valori sono definiti dallo sviluppatore e **non** influiscono sul comportamento del routing in alcun modo. Inoltre, i valori accantonati in [RouteData.DataTokens](xref:Microsoft.AspNetCore.Routing.RouteData.DataTokens*) possono essere di qualsiasi tipo, a differenza dei valori [RouteData.Values](xref:Microsoft.AspNetCore.Routing.RouteData.Values) che devono essere convertibili in e da stringhe.

[RouteData.Routers](xref:Microsoft.AspNetCore.Routing.RouteData.Routers) è un elenco delle route che hanno preso parte alla corrispondenza corretta della richiesta. Le route possono essere annidate l'una nell'altra. La proprietà <xref:Microsoft.AspNetCore.Routing.RouteData.Routers> rispecchia il percorso tramite l'albero logico di route che hanno prodotto una corrispondenza. In genere il primo elemento in <xref:Microsoft.AspNetCore.Routing.RouteData.Routers> è la raccolta di route e deve essere usato per la generazione di URL. L'ultimo elemento in <xref:Microsoft.AspNetCore.Routing.RouteData.Routers> è il gestore di route corrispondente.

<a name="lg"></a>

### <a name="url-generation"></a>Generazione di URL

La generazione di URL è il processo con cui il routing crea un percorso URL basato su un set di valori di route. Questo consente di avere una separazione logica tra i gestori di route e gli URL che vi accedono.

La generazione di URL segue un processo iterativo simile, ma inizia con la chiamata del codice dell'utente o del framework nel metodo <xref:Microsoft.AspNetCore.Routing.IRouter.GetVirtualPath*> della raccolta di route. Per ogni *route* viene chiamato in sequenza il metodo <xref:Microsoft.AspNetCore.Routing.IRouter.GetVirtualPath*> finché non viene restituito un valore <xref:Microsoft.AspNetCore.Routing.VirtualPathData> non Null.

Gli input primari per <xref:Microsoft.AspNetCore.Routing.IRouter.GetVirtualPath*> sono:

* [VirtualPathContext.HttpContext](xref:Microsoft.AspNetCore.Routing.VirtualPathContext.HttpContext)
* [VirtualPathContext.Values](xref:Microsoft.AspNetCore.Routing.VirtualPathContext.Values)
* [VirtualPathContext.AmbientValues](xref:Microsoft.AspNetCore.Routing.VirtualPathContext.AmbientValues)

Le route usano principalmente i valori di route specificati da <xref:Microsoft.AspNetCore.Routing.VirtualPathContext.Values> e <xref:Microsoft.AspNetCore.Routing.VirtualPathContext.AmbientValues> per decidere se è possibile generare un URL e quali valori includere. Gli elementi <xref:Microsoft.AspNetCore.Routing.VirtualPathContext.AmbientValues> sono il set di valori di route prodotti dalla corrispondenza della richiesta corrente. Al contrario, gli elementi <xref:Microsoft.AspNetCore.Routing.VirtualPathContext.Values> sono i valori di route che specificano in che modo viene generato l'URL per l'operazione corrente. <xref:Microsoft.AspNetCore.Routing.VirtualPathContext.HttpContext> viene specificato nel caso in cui una route ottenga servizi o dati aggiuntivi associati al contesto corrente.

> [!TIP]
> Considerare [VirtualPathContext.Values](xref:Microsoft.AspNetCore.Routing.VirtualPathContext.Values*) come un set di override per [VirtualPathContext.AmbientValues](xref:Microsoft.AspNetCore.Routing.VirtualPathContext.AmbientValues*). La generazione di URL prova a usare nuovamente i valori di route della richiesta corrente per generare URL per i collegamenti che usano la stessa route o gli stessi valori di route.

L'output di <xref:Microsoft.AspNetCore.Routing.IRouter.GetVirtualPath*> è un oggetto <xref:Microsoft.AspNetCore.Routing.VirtualPathData>. <xref:Microsoft.AspNetCore.Routing.VirtualPathData> è un elemento parallelo di <xref:Microsoft.AspNetCore.Routing.RouteData>. <xref:Microsoft.AspNetCore.Routing.VirtualPathData> contiene l'elemento <xref:Microsoft.AspNetCore.Routing.VirtualPathData.VirtualPath> per l'URL di output e alcune proprietà aggiuntive che devono essere impostate dalla route.

La proprietà [VirtualPathData.VirtualPath](xref:Microsoft.AspNetCore.Routing.VirtualPathData.VirtualPath*) contiene il *percorso virtuale* prodotto dalla route. In base alle esigenze specifiche, può essere necessario elaborare ulteriormente il percorso. Se si vuole eseguire il rendering in HTML dell'URL generato, anteporre il percorso di base dell'app.

[VirtualPathData.Router](xref:Microsoft.AspNetCore.Routing.VirtualPathData.Router*) è un riferimento alla route che ha generato correttamente l'URL.

[VirtualPathData.DataTokens](xref:Microsoft.AspNetCore.Routing.VirtualPathData.DataTokens*) è un dizionario di dati aggiuntivi correlati alla route che ha generato l'URL. È l'elemento parallelo di [RouteData.DataTokens](xref:Microsoft.AspNetCore.Routing.RouteData.DataTokens*).

### <a name="create-routes"></a>Creare route

Il routing specifica la classe <xref:Microsoft.AspNetCore.Routing.Route> come implementazione standard di <xref:Microsoft.AspNetCore.Routing.IRouter>. <xref:Microsoft.AspNetCore.Routing.Route> usa la sintassi del *modello di route* per definire i criteri in base ai quali confrontare il percorso URL quando si chiama <xref:Microsoft.AspNetCore.Routing.IRouter.RouteAsync*>. <xref:Microsoft.AspNetCore.Routing.Route> usa lo stesso modello di route per generare un URL quando si chiama <xref:Microsoft.AspNetCore.Routing.IRouter.GetVirtualPath*>.

La maggior parte delle app crea le route chiamando <xref:Microsoft.AspNetCore.Builder.MapRouteRouteBuilderExtensions.MapRoute*> o uno dei metodi di estensione simili definiti per <xref:Microsoft.AspNetCore.Routing.IRouteBuilder>. Tutti i metodi di estensione <xref:Microsoft.AspNetCore.Routing.IRouteBuilder> creano un'istanza di <xref:Microsoft.AspNetCore.Routing.Route> e la aggiungono alla raccolta di route.

<xref:Microsoft.AspNetCore.Builder.MapRouteRouteBuilderExtensions.MapRoute*> non accetta alcun parametro del gestore di route. <xref:Microsoft.AspNetCore.Builder.MapRouteRouteBuilderExtensions.MapRoute*> aggiunge solo le route che sono gestite da <xref:Microsoft.AspNetCore.Routing.RouteBuilder.DefaultHandler*>. Il gestore predefinito è `IRouter` e il gestore potrebbe non gestire la richiesta. Ad esempio, ASP.NET Core MVC viene in genere configurato come un gestore predefinito che gestisce solo le richieste che corrispondono a un'azione e un controller disponibili. Per altre informazioni sul routing in MVC, vedere <xref:mvc/controllers/routing>.

L'esempio di codice seguente si riferisce a una chiamata <xref:Microsoft.AspNetCore.Builder.MapRouteRouteBuilderExtensions.MapRoute*> usata da una tipica definizione di route di ASP.NET Core MVC:

```csharp
routes.MapRoute(
    name: "default",
    template: "{controller=Home}/{action=Index}/{id?}");
```

Questo modello confronta un percorso URL ed estrae i valori di route. Ad esempio, il percorso `/Products/Details/17` genera i valori di route seguenti: `{ controller = Products, action = Details, id = 17 }`.

I valori di route vengono determinati suddividendo il percorso URL in segmenti e confrontando ogni segmento con il nome del *parametro di route* nel modello di route. I parametri di route sono denominati. Parametri definiti racchiudendo il nome del parametro tra parentesi graffe `{ ... }`.

Il modello precedente può anche confrontare il percorso URL `/` e generare i valori `{ controller = Home, action = Index }`. Ciò accade perché i parametri di route `{controller}` e `{action}` hanno valori predefiniti e il parametro di route `id` è facoltativo. Un segno di uguale (`=`) seguito da un valore dopo il nome del parametro di route definisce un valore predefinito per il parametro. Un punto interrogativo (`?`) dopo il nome del parametro di route definisce un parametro facoltativo.

I parametri di route con un valore predefinito producono *sempre* un valore di route quando la route corrisponde. I parametri facoltativi non producono un valore di route se non è presente un segmento di percorso URL corrispondente. Vedere la sezione [Riferimento per i modelli di route](#route-template-reference) per una descrizione completa degli scenari e della sintassi del modello di route.

Nell'esempio seguente la definizione del parametro di route `{id:int}` definisce un [vincolo di route](#route-constraint-reference) per il parametro di route `id`:

```csharp
routes.MapRoute(
    name: "default",
    template: "{controller=Home}/{action=Index}/{id:int}");
```

Questo modello confronta un percorso URL come `/Products/Details/17`, ma non `/Products/Details/Apples`. I vincoli di route implementano <xref:Microsoft.AspNetCore.Routing.IRouteConstraint> ed esaminano i valori di route per verificarli. In questo esempio il valore di route `id` deve poter essere convertito in un numero intero. Vedere [Riferimento per i vincoli di route](#route-constraint-reference) per una descrizione dei vincoli di route specificati dal framework.

Altri overload di <xref:Microsoft.AspNetCore.Builder.MapRouteRouteBuilderExtensions.MapRoute*> accettano i valori per `constraints`, `dataTokens` e `defaults`. L'uso tipico di questi parametri è passare un oggetto tipizzato in modo anonimo, in cui i nomi di proprietà di tipo anonimo corrispondono ai nomi dei parametri di route.

Gli esempi <xref:Microsoft.AspNetCore.Builder.MapRouteRouteBuilderExtensions.MapRoute*> seguenti creano route equivalenti:

```csharp
routes.MapRoute(
    name: "default_route",
    template: "{controller}/{action}/{id?}",
    defaults: new { controller = "Home", action = "Index" });

routes.MapRoute(
    name: "default_route",
    template: "{controller=Home}/{action=Index}/{id?}");
```

> [!TIP]
> La sintassi inline per definire i vincoli e i valori predefiniti può essere utile per le route semplici. Esistono tuttavia scenari, come i token di dati, che non sono supportati dalla sintassi inline.

L'esempio seguente illustra alcuni scenari aggiuntivi:

```csharp
routes.MapRoute(
    name: "blog",
    template: "Blog/{*article}",
    defaults: new { controller = "Blog", action = "ReadArticle" });
```

Il modello precedente confronta un percorso URL come `/Blog/All-About-Routing/Introduction` ed estrae i valori `{ controller = Blog, action = ReadArticle, article = All-About-Routing/Introduction }`. I valori di route predefiniti per `controller` e `action` vengono generati dalla route anche se sono non presenti parametri di route corrispondenti nel modello. I valori predefiniti possono essere specificati nel modello di route. Il parametro di route `article` è definito come *catch-all* in base alla presenza di un asterisco (`*`) prima del nome del parametro di route. I parametri di route catch-all acquisiscono il resto del percorso URL e possono anche corrispondere alla stringa vuota.

L'esempio seguente aggiunge i vincoli di route e i token di dati:

```csharp
routes.MapRoute(
    name: "us_english_products",
    template: "en-US/Products/{id}",
    defaults: new { controller = "Products", action = "Details" },
    constraints: new { id = new IntRouteConstraint() },
    dataTokens: new { locale = "en-US" });
```

Il modello precedente confronta un percorso URL come `/en-US/Products/5` ed estrae i valori `{ controller = Products, action = Details, id = 5 }` e i token di dati `{ locale = en-US }`.

![Token della finestra Variabili locali](routing/_static/tokens.png)

### <a name="route-class-url-generation"></a>Generazione di URL della classe di route

La classe <xref:Microsoft.AspNetCore.Routing.Route> è inoltre in grado di eseguire la generazione di URL combinando un set di valori di route con il relativo modello di route. Questo è logicamente il processo inverso di corrispondenza del percorso URL.

> [!TIP]
> Per comprendere meglio la generazione di URL, si pensi a quale URL si vuole generare e al modo in cui un modello di route deve corrispondere a tale URL. Quali valori devono essere prodotti? Questo equivale approssimativamente al funzionamento della generazione di URL nella classe <xref:Microsoft.AspNetCore.Routing.Route>.

L'esempio seguente usa una route predefinita di ASP.NET Core MVC generale:

```csharp
routes.MapRoute(
    name: "default",
    template: "{controller=Home}/{action=Index}/{id?}");
```

Con i valori di route `{ controller = Products, action = List }`, viene generato l'URL `/Products/List`. I valori di route vengono sostituiti dai parametri di route corrispondenti per formare il percorso URL. Poiché `id` è un parametro di route facoltativo, l'URL è stato generato senza un valore per `id`.

Con i valori di route `{ controller = Home, action = Index }`, viene generato l'URL `/`. Poiché i valori di route specificati corrispondono ai valori predefiniti, i segmenti corrispondenti ai valori predefiniti possono essere omessi.

Per entrambi gli URL generati viene eseguito il round trip con la definizione di route seguente (`/Home/Index` e `/`) e vengono prodotti gli stessi valori di route usati per generare l'URL.

> [!NOTE]
> Un'app che usa ASP.NET Core MVC deve usare <xref:Microsoft.AspNetCore.Mvc.Routing.UrlHelper> per generare gli URL invece di eseguire la chiamata direttamente nel routing.

Per altre informazioni sulla generazione di URL, vedere la sezione [Riferimento per la generazione di URL](#url-generation-reference).

## <a name="use-routing-middleware"></a>USA middleware di routing

Fare riferimento al [metapacchetto Microsoft.AspNetCore.App](xref:fundamentals/metapackage-app) nel file di progetto dell'app.

Aggiungere il routing al contenitore del servizio in `Startup.ConfigureServices`:

[!code-csharp[](routing/samples/2.x/RoutingSample/Startup.cs?name=snippet_ConfigureServices&highlight=3)]

Le route devono essere configurate nel metodo `Startup.Configure`. L'app di esempio usa le API seguenti:

* <xref:Microsoft.AspNetCore.Routing.RouteBuilder>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapGet*>: Corrisponde solo alle richieste HTTP GET.
* <xref:Microsoft.AspNetCore.Builder.RoutingBuilderExtensions.UseRouter*>

[!code-csharp[](routing/samples/2.x/RoutingSample/Startup.cs?name=snippet_RouteHandler)]

Nella tabella seguente sono elencate le risposte con gli URI specificati.

| URI                    | Risposta                                          |
| ---------------------- | ------------------------------------------------- |
| `/package/create/3`    | Hello! Route values: [operation, create], [id, 3] |
| `/package/track/-3`    | Hello! Route values: [operation, track], [id, -3] |
| `/package/track/-3/`   | Hello! Route values: [operation, track], [id, -3] |
| `/package/track/`      | La richiesta non viene eseguita, nessuna corrispondenza.              |
| `GET /hello/Joe`       | Hi, Joe!                                          |
| `POST /hello/Joe`      | La richiesta non viene eseguita, corrispondenza solo con HTTP GET. |
| `GET /hello/Joe/Smith` | La richiesta non viene eseguita, nessuna corrispondenza.              |

Se si sta configurando una singola route, chiamare <xref:Microsoft.AspNetCore.Builder.RoutingBuilderExtensions.UseRouter*> passando un'istanza di `IRouter`. Non è necessario usare <xref:Microsoft.AspNetCore.Routing.RouteBuilder>.

Il framework offre un set di metodi di estensione per la creazione di route (<xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions>):

* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapDelete*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapGet*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapMiddlewareDelete*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapMiddlewareGet*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapMiddlewarePost*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapMiddlewarePut*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapMiddlewareRoute*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapMiddlewareVerb*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapPost*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapPut*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapRoute*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapVerb*>

Alcuni dei metodi elencati, ad esempio <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapGet*>, richiedono <xref:Microsoft.AspNetCore.Http.RequestDelegate>. <xref:Microsoft.AspNetCore.Http.RequestDelegate> viene usato come *gestore di route* quando la route corrisponde. Altri metodi di questa famiglia consentono di configurare una pipeline middleware che verrà usata come gestore di route. Se il metodo `Map*` non accetta un gestore, ad esempio <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapRoute*>, il metodo usa <xref:Microsoft.AspNetCore.Routing.RouteBuilder.DefaultHandler*>.

I metodi `Map[Verb]` usano i vincoli per limitare la route al verbo HTTP nel nome del metodo. Vedere ad esempio <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapGet*> e <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapVerb*>.

## <a name="route-template-reference"></a>Riferimento per il modello di route

I token all'interno di parentesi graffe (`{ ... }`) definiscono i *parametri di route* che vengono associati se esiste una corrispondenza per la route. È possibile definire più parametri di route in un segmento di route, ma devono essere separati da un valore letterale. Ad esempio, `{controller=Home}{action=Index}` non è una route valida perché non è presente un valore letterale tra `{controller}` e `{action}`. Questi parametri di route devono avere un nome e possono avere attributi aggiuntivi.

Il testo letterale diverso dai parametri di route, ad esempio `{id}`, e il separatore di percorso `/` devono corrispondere al testo nell'URL. La corrispondenza del testo non fa distinzione tra maiuscole e minuscole e si basa sulla rappresentazione decodificata del percorso degli URL. Per verificare la corrispondenza di un delimitatore letterale dei parametri di route (`{` o `}`), eseguire l'escape del delimitatore ripetendo il carattere (`{{` o `}}`).

I modelli di URL che tentano di acquisire un nome file con un'estensione facoltativa hanno considerazioni aggiuntive. Considerare ad esempio il modello `files/{filename}.{ext?}`. Se esistono i valori per `filename` e `ext`, vengono popolati entrambi i valori. Se esiste solo un valore per `filename` nell'URL, la route corrisponde perché il punto finale (`.`) è facoltativo. Gli URL seguenti corrispondono a questa route:

* `/files/myFile.txt`
* `/files/myFile`

È possibile usare l'asterisco (`*`) come prefisso per un parametro di route da associare alla parte rimanente dell'URI, ovvero un parametro *catch-all*. Ad esempio, `blog/{*slug}` corrisponde a qualsiasi URI che inizia con `/blog` seguito da qualsiasi valore, che viene assegnato al valore di route `slug`. I parametri catch-all possono anche corrispondere alla stringa vuota.

Il parametro catch-all esegue l'escape dei caratteri appropriati, incluso il separatore di percorso (`/`), quando la route viene usata per generare un URL. Ad esempio, la route `foo/{*path}` con i valori di route `{ path = "my/path" }` genera `foo/my%2Fpath`. Si noti la barra di escape.

I parametri di route possono avere *valori predefiniti*, definiti specificando il valore predefinito dopo il nome del parametro, separato da un segno di uguale (`=`). Ad esempio, `{controller=Home}` definisce `Home` come valore predefinito per `controller`. Il valore predefinito viene usato se nell'URL non è presente alcun valore per il parametro. I parametri di route vengono resi facoltativi aggiungendo un punto interrogativo (`?`) alla fine del nome del parametro, come in `id?`. La differenza tra parametri facoltativi e parametri di route predefiniti è che un parametro di route con un valore predefinito produce sempre un valore, mentre un parametro facoltativo ha un valore solo se ne viene specificato uno dall'URL della richiesta.

I parametri di route possono presentare dei vincoli che devono corrispondere al valore di route associato dall'URL. Aggiungendo due punti (`:`) e il nome del vincolo dopo il nome del parametro di route, si specifica un *vincolo inline* per un parametro di route. Se il vincolo richiede argomenti, vengono racchiusi tra parentesi (`(...)`) dopo il nome del vincolo. È possibile specificare più vincoli inline aggiungendo di nuovo i due punti (`:`) e il nome del vincolo.

Il nome del vincolo e gli argomenti vengono passati al servizio <xref:Microsoft.AspNetCore.Routing.IInlineConstraintResolver> per creare un'istanza di <xref:Microsoft.AspNetCore.Routing.IRouteConstraint> da usare nell'elaborazione dell'URL. Il modello di route `blog/{article:minlength(10)}` specifica ad esempio un vincolo `minlength` con l'argomento `10`. Per altre informazioni sui vincoli di route e per un elenco dei vincoli specificati dal framework, vedere la sezione [Riferimento per i vincoli di route](#route-constraint-reference).

La tabella seguente illustra i modelli di route di esempio e il relativo comportamento.

| Modello di route                           | URI corrispondente di esempio    | L'URI della richiesta&hellip;                                                    |
| ---------------------------------------- | ----------------------- | -------------------------------------------------------------------------- |
| `hello`                                  | `/hello`                | Verifica la corrispondenza solo del singolo percorso `/hello`.                                     |
| `{Page=Home}`                            | `/`                     | Verifica la corrispondenza e imposta `Page` su `Home`.                                         |
| `{Page=Home}`                            | `/Contact`              | Verifica la corrispondenza e imposta `Page` su `Contact`.                                      |
| `{controller}/{action}/{id?}`            | `/Products/List`        | Esegue il mapping al controller `Products` e all'azione `List`.                       |
| `{controller}/{action}/{id?}`            | `/Products/Details/123` | Esegue il mapping al controller `Products` e all'azione `Details` (`id` impostato su 123). |
| `{controller=Home}/{action=Index}/{id?}` | `/`                     | Esegue il mapping al controller `Home` e al metodo `Index` (`id` viene ignorato).        |

L'uso di un modello è in genere l'approccio più semplice al routing. I vincoli e le impostazioni predefinite possono essere specificati anche all'esterno del modello di route.

> [!TIP]
> Abilitare la [registrazione](xref:fundamentals/logging/index) per verificare in che modo le implementazioni del routing predefinite, ad esempio <xref:Microsoft.AspNetCore.Routing.Route>, corrispondono alle richieste.

## <a name="route-constraint-reference"></a>Riferimento per i vincoli di route

I vincoli di route vengono eseguiti quando si verifica una corrispondenza nell'URL in ingresso e il percorso URL viene suddiviso in valori di route in formato token. I vincoli di route in genere controllano il valore di route associato usando il modello di route e stabiliscono con una decisione Sì/No se il valore è accettabile o meno. Alcuni vincoli di route usano i dati all'esterno del valore di route per stabilire se la richiesta può essere instradata. Ad esempio, <xref:Microsoft.AspNetCore.Routing.Constraints.HttpMethodRouteConstraint> può accettare o rifiutare una richiesta in base al relativo verbo HTTP. I vincoli vengono usati nelle richieste del routing e nella generazione di collegamenti.

> [!WARNING]
> Non usare i vincoli per la **convalida dell'input**. Se vengono usati vincoli per la **convalida dell'input**, un input non valido causa la visualizzazione di un errore di tipo *404 (Non trovato)* invece di un errore di tipo *400 - Richiesta non valida* con un messaggio di errore appropriato. I vincoli di route vengono usati per evitare **ambiguità** tra route simili, non per convalidare gli input per una route specifica.

La tabella seguente illustra i vincoli di route di esempio e il relativo comportamento.

| vincolo | Esempio | Esempi di corrispondenza | Note |
| ---------- | ------- | --------------- | ----- |
| `int` | `{id:int}` | `123456789`, `-123456789` | Corrisponde a qualsiasi numero intero |
| `bool` | `{active:bool}` | `true`, `FALSE` | Corrisponde a `true` o `false` (senza distinzione tra maiuscole e minuscole) |
| `datetime` | `{dob:datetime}` | `2016-12-31`, `2016-12-31 7:32pm` | Corrisponde a un `DateTime` valore valido della lingua inglese. Vedere l'avviso precedente.|
| `decimal` | `{price:decimal}` | `49.99`, `-1,000.01` | Corrisponde a un `decimal` valore valido della lingua inglese. Vedere l'avviso precedente.|
| `double` | `{weight:double}` | `1.234`, `-1,001.01e8` | Corrisponde a un `double` valore valido della lingua inglese. Vedere l'avviso precedente.|
| `float` | `{weight:float}` | `1.234`, `-1,001.01e8` | Corrisponde a un `float` valore valido della lingua inglese. Vedere l'avviso precedente.|
| `guid` | `{id:guid}` | `CD2C1638-1638-72D5-1638-DEADBEEF1638`, `{CD2C1638-1638-72D5-1638-DEADBEEF1638}` | Corrisponde a un valore `Guid` valido |
| `long` | `{ticks:long}` | `123456789`, `-123456789` | Corrisponde a un valore `long` valido |
| `minlength(value)` | `{username:minlength(4)}` | `Rick` | La stringa deve contenere almeno 4 caratteri |
| `maxlength(value)` | `{filename:maxlength(8)}` | `Richard` | La stringa non deve contenere più di 8 caratteri |
| `length(length)` | `{filename:length(12)}` | `somefile.txt` | La stringa deve contenere esattamente 12 caratteri |
| `length(min,max)` | `{filename:length(8,16)}` | `somefile.txt` | La stringa deve contenere almeno 8 e non più di 16 caratteri |
| `min(value)` | `{age:min(18)}` | `19` | Il valore intero deve essere almeno 18 |
| `max(value)` | `{age:max(120)}` | `91` | Il valore intero non deve essere superiore a 120 |
| `range(min,max)` | `{age:range(18,120)}` | `91` | Il valore intero deve essere almeno 18 ma non più di 120 |
| `alpha` | `{name:alpha}` | `Rick` | La stringa deve essere costituita da uno o più caratteri alfabetici (`a`-`z`, senza distinzione tra maiuscole e minuscole) |
| `regex(expression)` | `{ssn:regex(^\\d{{3}}-\\d{{2}}-\\d{{4}}$)}` | `123-45-6789` | La stringa deve corrispondere all'espressione regolare (vedere i suggerimenti per la definizione di un'espressione regolare) |
| `required` | `{name:required}` | `Rick` | Usato per imporre che un valore diverso da un parametro sia presente durante la generazione dell'URL |

Più vincoli delimitati da punti possono essere applicati a un singolo parametro. Ad esempio il vincolo seguente limita un parametro a un valore intero maggiore o uguale a 1:

```csharp
[Route("users/{id:int:min(1)}")]
public User GetUserById(int id) { }
```

> [!WARNING]
> I vincoli di route che verificano l'URL e vengono convertiti in un tipo CLR, ad esempio `int` o `DateTime`, usano sempre le impostazioni cultura inglese non dipendenti da paese/area geografica, presupponendo che l'URL sia non localizzabile. I vincoli di route specificati dal framework non modificano i valori archiviati nei valori di route. Tutti i valori di route analizzati dall'URL vengono archiviati come stringhe. Ad esempio, il vincolo `float` prova a convertire il valore di route in un valore float, ma il valore convertito viene usato solo per verificare che può essere convertito in un valore float.

## <a name="regular-expressions"></a>Espressioni regolari

Il framework di ASP.NET Core aggiunge `RegexOptions.IgnoreCase | RegexOptions.Compiled | RegexOptions.CultureInvariant` al costruttore di espressioni regolari. Per una descrizione di questi membri, vedere <xref:System.Text.RegularExpressions.RegexOptions>.

Le espressioni regolari usano delimitatori e token simili a quelli usati dal routing e dal linguaggio C#. Per i token di espressione è necessario aggiungere caratteri di escape. Per usare l'espressione regolare `^\d{3}-\d{2}-\d{4}$` nel routing, l'espressione deve includere i caratteri `\` (barra rovesciata singola) specificati nella stringa come caratteri `\\` (barra rovesciata doppia) nel file di origine C# per eseguire l'escape del carattere di escape della stringa `\`, a meno che non si usino [valori letterali di stringa verbatim](/dotnet/csharp/language-reference/keywords/string). Per eseguire l'escape dii caratteri delimitatori dei parametri di routing (`{`, `}`, `[`, `]`), raddoppiare i caratteri nell'espressione (`{{`, `}`, `[[`, `]]`). La tabella seguente mostra un'espressione regolare e la versione dopo l'escape.

| Espressione regolare    | Espressione regolare con caratteri di escape     |
| --------------------- | ------------------------------ |
| `^\d{3}-\d{2}-\d{4}$` | `^\\d{{3}}-\\d{{2}}-\\d{{4}}$` |
| `^[a-z]{2}$`          | `^[[a-z]]{{2}}$`               |

Le espressioni regolari usate nel routing spesso iniziano con l'accento circonflesso (`^`) e corrispondono alla posizione iniziale della stringa. Le espressioni spesso terminano con il segno di dollaro (`$`) e corrispondono alla fine della stringa. I caratteri `^` e `$` consentono di verificare che l'espressione regolare corrisponda all'intero valore del parametro di route. Senza i caratteri `^` e `$` l'espressione regolare corrisponde a qualsiasi sottostringa all'interno della stringa e spesso questo non è il risultato desiderato. La tabella seguente include alcuni esempi e descrive perché si verifica o non si verifica la corrispondenza.

| Expression   | string    | Corrispondenza | Commento               |
| ------------ | --------- | :---: |  -------------------- |
| `[a-z]{2}`   | hello     | Sì   | Corrispondenze di sottostringhe     |
| `[a-z]{2}`   | 123abc456 | Sì   | Corrispondenze di sottostringhe     |
| `[a-z]{2}`   | mz        | Sì   | Corrisponde all'espressione    |
| `[a-z]{2}`   | MZ        | Sì   | Senza distinzione maiuscole/minuscole    |
| `^[a-z]{2}$` | hello     | No    | Vedere `^` e `$` sopra |
| `^[a-z]{2}$` | 123abc456 | No    | Vedere `^` e `$` sopra |

Per altre informazioni sulla sintassi delle espressioni regolari, vedere [Espressioni regolari di .NET Framework](/dotnet/standard/base-types/regular-expression-language-quick-reference).

Per limitare un parametro a un set noto di valori possibili, usare un'espressione regolare. Ad esempio, `{action:regex(^(list|get|create)$)}` verifica la corrispondenza del valore di route `action` solo con `list`, `get` o `create`. Se viene passata nel dizionario di vincoli, la stringa `^(list|get|create)$` è equivalente. Anche i vincoli passati nel dizionario di vincoli (non inline all'interno di un modello) che non corrispondono a uno dei vincoli noti vengono considerati espressioni regolari.

## <a name="custom-route-constraints"></a>Vincoli di route personalizzati

Oltre ai vincoli di route predefiniti, è possibile creare vincoli di route personalizzati implementando l'interfaccia <xref:Microsoft.AspNetCore.Routing.IRouteConstraint>. L'interfaccia <xref:Microsoft.AspNetCore.Routing.IRouteConstraint> contiene un singolo metodo, `Match`, che restituisce `true` se il vincolo viene soddisfatto e `false` in caso contrario.

Per usare un'interfaccia <xref:Microsoft.AspNetCore.Routing.IRouteConstraint> personalizzata, il tipo di vincolo di route deve essere registrato con la proprietà <xref:Microsoft.AspNetCore.Routing.RouteOptions.ConstraintMap> dell'app nel contenitore di servizi dell'app. <xref:Microsoft.AspNetCore.Routing.RouteOptions.ConstraintMap> è un dizionario che esegue il mapping delle chiavi dei vincoli di route alle implementazioni di <xref:Microsoft.AspNetCore.Routing.IRouteConstraint> che convalidano tali vincoli. La proprietà <xref:Microsoft.AspNetCore.Routing.RouteOptions.ConstraintMap> di un'app può essere aggiornata in `Startup.ConfigureServices` come parte di una chiamata [services.AddRouting](xref:Microsoft.Extensions.DependencyInjection.RoutingServiceCollectionExtensions.AddRouting*) o configurando <xref:Microsoft.AspNetCore.Routing.RouteOptions> direttamente con `services.Configure<RouteOptions>`. Ad esempio:

```csharp
services.AddRouting(options =>
{
    options.ConstraintMap.Add("customName", typeof(MyCustomConstraint));
});
```

Il vincolo può quindi essere applicato alle route nel modo consueto, usando il nome specificato al momento della registrazione del tipo di vincolo. Ad esempio:

```csharp
[HttpGet("{id:customName}")]
public ActionResult<string> Get(string id)
```

## <a name="url-generation-reference"></a>Riferimento per la generazione di URL

L'esempio seguente mostra come generare un collegamento a una route usando un dizionario di valori di route e un elemento <xref:Microsoft.AspNetCore.Routing.RouteCollection>.

[!code-csharp[](routing/samples/2.x/RoutingSample/Startup.cs?name=snippet_Dictionary)]

L'elemento <xref:Microsoft.AspNetCore.Routing.VirtualPathData.VirtualPath> generato alla fine dell'esempio precedente è `/package/create/123`. Il dizionario fornisce i valori di route `operation` e `id` del modello "Track Package Route", `package/{operation}/{id}`. Per informazioni dettagliate, vedere il codice di esempio nella sezione [Usare il middleware di routing](#use-routing-middleware) oppure l'[app di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/routing/samples).

Il secondo parametro per il costruttore <xref:Microsoft.AspNetCore.Routing.VirtualPathContext> è una raccolta di *valori di ambiente*. I valori di ambiente sono utili poiché limitano il numero di valori che uno sviluppatore deve specificare all'interno del contesto di una richiesta. I valori di route correnti della richiesta corrente sono considerati valori di ambiente per la generazione del collegamento. Nell'azione `About` di un'app ASP.NET Core MVC di `HomeController`, non è necessario specificare il valore di route del controller per collegarsi all'azione `Index`. Viene usato il valore di ambiente `Home`.

I valori di ambiente che non corrispondono a un parametro vengono ignorati. I valori di ambiente vengono ignorati anche quando un valore specificato in modo esplicito esegue l'override del valore di ambiente. La corrispondenza si verifica da sinistra a destra nell'URL.

I valori specificati in modo esplicito ma che non corrispondono a un segmento della route vengono aggiunti alla stringa di query. La tabella seguente illustra il risultato ottenuto quando si usa il modello di route `{controller}/{action}/{id?}`.

| Valori di ambiente                     | Valori espliciti                        | Risultato                  |
| ---------------------------------- | -------------------------------------- | ----------------------- |
| controller = "Home"                | action = "About"                       | `/Home/About`           |
| controller = "Home"                | controller = "Order", action = "About" | `/Order/About`          |
| controller = "Home", color = "Red" | action = "About"                       | `/Home/About`           |
| controller = "Home"                | action = "About", color = "Red"        | `/Home/About?color=Red` |

Se una route ha un valore predefinito che non corrisponde a un parametro e tale valore viene specificato in modo esplicito, deve corrispondere al valore predefinito:

```csharp
routes.MapRoute("blog_route", "blog/{*slug}",
    defaults: new { controller = "Blog", action = "ReadPost" });
```

La generazione dei collegamenti genera un collegamento per questa route quando vengono specificati i valori corrispondenti per `controller` e `action`.

## <a name="complex-segments"></a>Segmenti complessi

I segmenti complessi (ad esempio, `[Route("/x{token}y")]`), vengono elaborati individuando corrispondenze per i valori letterali da destra a sinistra in modalità non-greedy. Vedere [questo codice](https://github.com/dotnet/aspnetcore/blob/v2.2.13/src/Http/Routing/src/Patterns/RoutePatternMatcher.cs#L293) per una spiegazione dettagliata di come vengono confrontati i segmenti complessi. L'[esempio di codice](https://github.com/dotnet/aspnetcore/blob/v2.2.13/src/Http/Routing/src/Patterns/RoutePatternMatcher.cs#L293) non viene usato da ASP.NET Core, ma offre una spiegazione esauriente dei segmenti complessi.

::: moniker-end
