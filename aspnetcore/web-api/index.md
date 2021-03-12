---
title: Creare API Web con ASP.NET Core
author: scottaddie
description: Informazioni di base sulla creazione di un'API Web in ASP.NET Core.
monikerRange: '>= aspnetcore-2.1'
ms.author: scaddie
ms.custom: mvc
ms.date: 07/20/2020
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
uid: web-api/index
ms.openlocfilehash: fb7dfa53be1fee9904b4539a5c9da0700c2aa884
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102585734"
---
# <a name="create-web-apis-with-aspnet-core"></a>Creare API Web con ASP.NET Core

Di [Scott Addie](https://github.com/scottaddie) e [Tom Dykstra](https://github.com/tdykstra)

ASP.NET Core supporta la creazione di servizi RESTful, noti anche come API Web, con C#. Per gestire le richieste, un'API Web usa i controller. I *controller* in un'API Web sono classi che derivano da `ControllerBase`. Questo articolo illustra come usare i controller per gestire le richieste API Web.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/web-api/index/samples). ([Come scaricare](xref:index#how-to-download-a-sample)).

## <a name="controllerbase-class"></a>Classe ControllerBase

Un'API Web è costituita da una o più classi controller che derivano da <xref:Microsoft.AspNetCore.Mvc.ControllerBase> . Il modello di progetto API Web fornisce un controller Starter:

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](index/samples/3.x/Controllers/WeatherForecastController.cs?name=snippet_ControllerSignature&highlight=3)]

::: moniker-end

::: moniker range="<= aspnetcore-2.2"

[!code-csharp[](index/samples/2.x/2.2/Controllers/ValuesController.cs?name=snippet_ControllerSignature&highlight=3)]

::: moniker-end

Non creare un controller API Web tramite derivazione dalla classe <xref:Microsoft.AspNetCore.Mvc.Controller>. La classe `Controller` deriva da `ControllerBase` e aggiunge il supporto per le visualizzazioni, pertanto è progettata per la gestione delle pagine Web e non per le richieste di API Web. Si è verificata un'eccezione a questa regola: se si prevede di usare lo stesso controller per le visualizzazioni e le API Web, derivarlo da `Controller` .

La classe `ControllerBase` offre molti metodi e proprietà utili per la gestione delle richieste HTTP. Ad esempio, `ControllerBase.CreatedAtAction` restituisce un codice di stato 201:

[!code-csharp[](index/samples/2.x/2.2/Controllers/PetsController.cs?name=snippet_400And201&highlight=10)]

Di seguito sono elencati alcuni esempi di metodi forniti da `ControllerBase`.

|Metodo   |Note    |
|---------|---------|
|<xref:Microsoft.AspNetCore.Mvc.ControllerBase.BadRequest%2A>| Restituisce il codice di stato 400.|
|<xref:Microsoft.AspNetCore.Mvc.ControllerBase.NotFound%2A>|Restituisce il codice di stato 404.|
|<xref:Microsoft.AspNetCore.Mvc.ControllerBase.PhysicalFile%2A>|Restituisce un file.|
|<xref:Microsoft.AspNetCore.Mvc.ControllerBase.TryUpdateModelAsync%2A>|Richiama l'[associazione di modelli](xref:mvc/models/model-binding).|
|<xref:Microsoft.AspNetCore.Mvc.ControllerBase.TryValidateModel%2A>|Richiama la [convalida dei modelli](xref:mvc/models/validation).|

Per un elenco di tutti i metodi e proprietà disponibili, vedere <xref:Microsoft.AspNetCore.Mvc.ControllerBase>.

## <a name="attributes"></a>Attributi

Lo spazio dei nomi <xref:Microsoft.AspNetCore.Mvc> include attributi che possono essere usati per configurare il comportamento del controller di API Web e i metodi di azione. Nell'esempio seguente vengono utilizzati gli attributi per specificare il verbo di azione HTTP supportato e i codici di stato HTTP noti che possono essere restituiti:

[!code-csharp[](index/samples/2.x/2.2/Controllers/PetsController.cs?name=snippet_400And201&highlight=1-3)]

Di seguito sono riportati altri esempi degli attributi disponibili.

|Attributo|Note|
|---------|-----|
|[`[Route]`](<xref:Microsoft.AspNetCore.Mvc.RouteAttribute>)      |Specifica il modello di URL per un controller o un'azione.|
|[`[Bind]`](<xref:Microsoft.AspNetCore.Mvc.BindAttribute>)        |Specifica il prefisso e le proprietà da includere per l'associazione di modelli.|
|[`[HttpGet]`](<xref:Microsoft.AspNetCore.Mvc.HttpGetAttribute>)  |Identifica un'azione che supporta il verbo di azione HTTP GET.|
|[`[Consumes]`](<xref:Microsoft.AspNetCore.Mvc.ConsumesAttribute>)|Specifica i tipi di dati accettati da un'azione.|
|[`[Produces]`](<xref:Microsoft.AspNetCore.Mvc.ProducesAttribute>)|Specifica i tipi di dati restituiti da un'azione.|

Per un elenco che include gli attributi disponibili, vedere lo spazio dei nomi <xref:Microsoft.AspNetCore.Mvc>.

## <a name="apicontroller-attribute"></a>Attributo ApiController

L' [`[ApiController]`](xref:Microsoft.AspNetCore.Mvc.ApiControllerAttribute) attributo può essere applicato a una classe controller per abilitare i seguenti comportamenti supponenti specifici dell'API:

::: moniker range=">= aspnetcore-2.2"

* [Requisiti del routing degli attributi](#attribute-routing-requirement)
* [Risposte HTTP 400 automatiche](#automatic-http-400-responses)
* [Inferenza del parametro di origine di associazione](#binding-source-parameter-inference)
* [Inferenza di richieste multipart/form-data](#multipartform-data-request-inference)
* [Dettagli del problema per i codici di stato di errore](#problem-details-for-error-status-codes)

La funzionalità *Dettagli problema per i codici di stato di errore* richiede una [versione di compatibilità](xref:mvc/compatibility-version) 2,2 o successiva. Le altre funzionalità richiedono una versione di compatibilità 2,1 o successiva.

::: moniker-end

* [Requisiti del routing degli attributi](#attribute-routing-requirement)
* [Risposte HTTP 400 automatiche](#automatic-http-400-responses)
* [Inferenza del parametro di origine di associazione](#binding-source-parameter-inference)
* [Inferenza di richieste multipart/form-data](#multipartform-data-request-inference)

Queste funzionalità richiedono la [versione di compatibilità](xref:mvc/compatibility-version) 2.1 o successiva.

### <a name="attribute-on-specific-controllers"></a>Attributo su controller specifici

L'attributo `[ApiController]` può essere applicato a controller specifici, come illustrato nell'esempio seguente dal modello di progetto:

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](index/samples/3.x/Controllers/WeatherForecastController.cs?name=snippet_ControllerSignature&highlight=2])]

::: moniker-end

::: moniker range="<= aspnetcore-2.2"

[!code-csharp[](index/samples/2.x/2.2/Controllers/ValuesController.cs?name=snippet_ControllerSignature&highlight=2)]

::: moniker-end

### <a name="attribute-on-multiple-controllers"></a>Attributo su più controller

Un approccio per usare l'attributo su più di un controller consiste nel creare una classe di controller di base personalizzata annotata con l'attributo `[ApiController]`. Nell'esempio seguente viene illustrata una classe base personalizzata e un controller che deriva da esso:

[!code-csharp[](index/samples/2.x/2.2/Controllers/MyControllerBase.cs?name=snippet_MyControllerBase)]

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](index/samples/3.x/Controllers/PetsController.cs?name=snippet_Inherit)]

::: moniker-end

::: moniker range="<= aspnetcore-2.2"

[!code-csharp[](index/samples/2.x/2.2/Controllers/PetsController.cs?name=snippet_Inherit)]

::: moniker-end

::: moniker range=">= aspnetcore-2.2"

### <a name="attribute-on-an-assembly"></a>Attributo in un assembly

Se la [versione di compatibilità](xref:mvc/compatibility-version) è impostata su 2.2 o una versione successiva, l'attributo `[ApiController]` può essere applicato a un assembly. L'uso delle annotazioni in questo modo applica il comportamento delle API Web a tutti i controller nell'assembly. Non esiste alcun modo per rifiutare esplicitamente singoli controller. Applicare l'attributo a livello di assembly alla dichiarazione dello spazio dei nomi che circonda la `Startup` classe:

```csharp
[assembly: ApiController]
namespace WebApiSample
{
    public class Startup
    {
        ...
    }
}
```

::: moniker-end

## <a name="attribute-routing-requirement"></a>Requisiti del routing degli attributi

Con l'attributo `[ApiController]` il routing degli attributi è un requisito. Ad esempio:

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](index/samples/3.x/Controllers/WeatherForecastController.cs?name=snippet_ControllerSignature&highlight=2)]

Le azioni sono inaccessibili tramite [Route convenzionali](xref:mvc/controllers/routing#conventional-routing) definite da `UseEndpoints` , <xref:Microsoft.AspNetCore.Builder.MvcApplicationBuilderExtensions.UseMvc%2A> o <xref:Microsoft.AspNetCore.Builder.MvcApplicationBuilderExtensions.UseMvcWithDefaultRoute%2A> in `Startup.Configure` .

::: moniker-end

::: moniker range="<= aspnetcore-2.2"

[!code-csharp[](index/samples/2.x/2.2/Controllers/ValuesController.cs?name=snippet_ControllerSignature&highlight=1)]

Le azioni non sono accessibili tramite le [route convenzionali](xref:mvc/controllers/routing#conventional-routing) definite da <xref:Microsoft.AspNetCore.Builder.MvcApplicationBuilderExtensions.UseMvc%2A> o <xref:Microsoft.AspNetCore.Builder.MvcApplicationBuilderExtensions.UseMvcWithDefaultRoute%2A> in `Startup.Configure`.

::: moniker-end

## <a name="automatic-http-400-responses"></a>Risposte HTTP 400 automatiche

Con l'attributo `[ApiController]` gli errori di convalida del modello attivano automaticamente una risposta HTTP 400. Di conseguenza è necessario il codice seguente in un metodo di azione:

```csharp
if (!ModelState.IsValid)
{
    return BadRequest(ModelState);
}
```

ASP.NET Core MVC usa il <xref:Microsoft.AspNetCore.Mvc.Infrastructure.ModelStateInvalidFilter> filtro azioni per eseguire il controllo precedente.

### <a name="default-badrequest-response"></a>Risposta BadRequest predefinita

Con una versione di compatibilità 2,1, il tipo di risposta predefinito per una risposta HTTP 400 è <xref:Microsoft.AspNetCore.Mvc.SerializableError> . Il corpo della richiesta seguente è un esempio del tipo serializzato:

```json
{
  "": [
    "A non-empty request body is required."
  ]
}
```

::: moniker range=">= aspnetcore-2.2"

Con una versione di compatibilità 2,2 o successiva, il tipo di risposta predefinito per una risposta HTTP 400 è <xref:Microsoft.AspNetCore.Mvc.ValidationProblemDetails> . Il corpo della richiesta seguente è un esempio del tipo serializzato:

```json
{
  "type": "https://tools.ietf.org/html/rfc7231#section-6.5.1",
  "title": "One or more validation errors occurred.",
  "status": 400,
  "traceId": "|7fb5e16a-4c8f23bbfc974667.",
  "errors": {
    "": [
      "A non-empty request body is required."
    ]
  }
}
```

`ValidationProblemDetails`Tipo:

* Fornisce un formato leggibile dal computer per specificare gli errori nelle risposte all'API Web.
* È conforme alla [specifica RFC 7807](https://tools.ietf.org/html/rfc7807).

::: moniker-end

Per rendere coerenti le risposte automatiche e personalizzate, chiamare il <xref:Microsoft.AspNetCore.Mvc.ControllerBase.ValidationProblem%2A> metodo anziché <xref:System.Web.Http.ApiController.BadRequest%2A> . `ValidationProblem` Restituisce un <xref:Microsoft.AspNetCore.Mvc.ValidationProblemDetails> oggetto e la risposta automatica.

### <a name="log-automatic-400-responses"></a>Registrare le risposte 400 automatiche

Vedere [come registrare risposte automatiche 400 sugli errori di convalida del modello (DotNet/AspNetCore.Docs # 12157)](https://github.com/dotnet/AspNetCore.Docs/issues/12157).

### <a name="disable-automatic-400-response"></a>Disabilitare la risposta automatica 400

Per disabilitare il comportamento automatico per gli errori 400, impostare la proprietà <xref:Microsoft.AspNetCore.Mvc.ApiBehaviorOptions.SuppressModelStateInvalidFilter> su `true`. Aggiungere il codice evidenziato seguente in `Startup.ConfigureServices`:

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](index/samples/3.x/Startup.cs?name=snippet_ConfigureApiBehaviorOptions&highlight=2,6)]

::: moniker-end

::: moniker range="= aspnetcore-2.2"

[!code-csharp[](index/samples/2.x/2.2/Startup.cs?name=snippet_ConfigureApiBehaviorOptions&highlight=3,7)]

::: moniker-end

::: moniker range="= aspnetcore-2.1"

[!code-csharp[](index/samples/2.x/2.1/Startup.cs?name=snippet_ConfigureApiBehaviorOptions&highlight=1,5)]

::: moniker-end

## <a name="binding-source-parameter-inference"></a>Inferenza del parametro di origine di associazione

Un attributo di origine di associazione definisce la posizione in cui viene trovato il valore del parametro di un'azione. Esistono gli attributi di origine di associazione seguente:

|Attributo|Origine di binding |
|---------|---------|
|[`[FromBody]`](xref:Microsoft.AspNetCore.Mvc.FromBodyAttribute)     | Testo della richiesta |
|[`[FromForm]`](xref:Microsoft.AspNetCore.Mvc.FromFormAttribute)     | Dati di modulo nel corpo della richiesta |
|[`[FromHeader]`](xref:Microsoft.AspNetCore.Mvc.FromHeaderAttribute) | Intestazione della richiesta |
|[`[FromQuery]`](xref:Microsoft.AspNetCore.Mvc.FromQueryAttribute)   | Parametri della stringa di query della richiesta |
|[`[FromRoute]`](xref:Microsoft.AspNetCore.Mvc.FromRouteAttribute)   | Dati della route della richiesta corrente |
|[`[FromServices]`](xref:mvc/controllers/dependency-injection#action-injection-with-fromservices) | Il servizio richiesta inserito come parametro di azione |

> [!WARNING]
> Non usare `[FromRoute]` quando i valori potrebbero contenere `%2f` (vale a dire `/`). `%2f` non sarà convertito in `/` rimuovendo i caratteri di escape. Usare `[FromQuery]` se il valore potrebbe contenere `%2f`.

Senza l'attributo `[ApiController]` o altri attributi di origine di associazione come `[FromQuery]`, il runtime di ASP.NET Core tenta di usare lo strumento di associazione di modelli a oggetti complesso. Lo strumento di associazione di modelli a oggetti complesso estrae i dati dal provider di valori in un ordine definito.

Nell'esempio seguente, l'attributo `[FromQuery]` indica che il valore del parametro `discontinuedOnly` è specificato nella stringa di query dell'URL della richiesta:

[!code-csharp[](index/samples/2.x/2.2/Controllers/ProductsController.cs?name=snippet_BindingSourceAttributes&highlight=3)]

L'attributo `[ApiController]` applica le regole di inferenza per le origini dati predefinite dei parametri di azione. Queste regole consentono di evitare di dover identificare le origini di associazione manualmente applicando attributi ai parametri di azione. Il comportamento delle regole di inferenza per le origini di associazione è il seguente:

* `[FromBody]` viene dedotto per i parametri di tipo complesso. Un'eccezione alla regola di inferenza `[FromBody]` è costituita dai tipi predefiniti complessi con un significato speciale, ad esempio <xref:Microsoft.AspNetCore.Http.IFormCollection> e <xref:System.Threading.CancellationToken>. Il codice di inferenza di origine di associazione ignora tali tipi speciali.
* `[FromForm]` viene dedotto per i parametri di azione di tipo <xref:Microsoft.AspNetCore.Http.IFormFile> e <xref:Microsoft.AspNetCore.Http.IFormFileCollection>. Non viene dedotto per i tipi semplici o definiti dall'utente.
* `[FromRoute]` viene dedotto per i nomi di parametro di azione corrispondenti a un parametro nel modello di route. Quando più di una route corrisponde a un parametro di azione, tutti i valori di route vengono considerati `[FromRoute]`.
* `[FromQuery]` viene dedotto per tutti gli altri parametri di azione.

### <a name="frombody-inference-notes"></a>Note sulla regola di inferenza FromBody

`[FromBody]` non viene dedotto per i tipi semplici, ad esempio `string` o `int`. Pertanto, l'attributo `[FromBody]` deve essere usato per i tipi semplici se è necessaria tale funzionalità.

Quando a un'azione sono associati più parametri dal corpo della richiesta, viene generata un'eccezione. Tutte le firme di metodo di azione seguenti, ad esempio, causano un'eccezione:

* `[FromBody]` dedotto per entrambi, perché si tratta di tipi complessi.

  ```csharp
  [HttpPost]
  public IActionResult Action1(Product product, Order order)
  ```

* `[FromBody]` attributo per uno, dedotto per l'altro perché è un tipo complesso.

  ```csharp
  [HttpPost]
  public IActionResult Action2(Product product, [FromBody] Order order)
  ```

* `[FromBody]` attributo per entrambi.

  ```csharp
  [HttpPost]
  public IActionResult Action3([FromBody] Product product, [FromBody] Order order)
  ```

::: moniker range="= aspnetcore-2.1"

> [!NOTE]
> In ASP.NET Core 2.1, i parametri di tipo raccolta, come elenchi e matrici, vengono dedotti in modo errato come `[FromQuery]`. È necessario usare l'attributo `[FromBody]` per questi parametri, se devono essere associati dal corpo della richiesta. Questo comportamento è stato corretto in ASP.NET Core 2.2 o versioni successive, in cui i parametri di tipo raccolta vengono dedotti come associati dal corpo per impostazione predefinita.

::: moniker-end

### <a name="disable-inference-rules"></a>Disabilitare le regole di inferenza

Per disabilitare l'inferenza delle origini di associazione, impostare <xref:Microsoft.AspNetCore.Mvc.ApiBehaviorOptions.SuppressInferBindingSourcesForParameters> su `true`. Aggiungere il codice seguente a `Startup.ConfigureServices`:

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](index/samples/3.x/Startup.cs?name=snippet_ConfigureApiBehaviorOptions&highlight=2,5)]

::: moniker-end

::: moniker range="= aspnetcore-2.2"

[!code-csharp[](index/samples/2.x/2.2/Startup.cs?name=snippet_ConfigureApiBehaviorOptions&highlight=3,6)]

::: moniker-end

::: moniker range="= aspnetcore-2.1"

[!code-csharp[](index/samples/2.x/2.1/Startup.cs?name=snippet_ConfigureApiBehaviorOptions&highlight=1,4)]

::: moniker-end

## <a name="multipartform-data-request-inference"></a>Inferenza di richieste multipart/form-data

L' `[ApiController]` attributo applica una regola di inferenza quando un parametro di azione viene annotato con l' [`[FromForm]`](xref:Microsoft.AspNetCore.Mvc.FromFormAttribute) attributo. Il `multipart/form-data` tipo di contenuto della richiesta viene dedotto.

Per disabilitare il comportamento predefinito, impostare la <xref:Microsoft.AspNetCore.Mvc.ApiBehaviorOptions.SuppressConsumesConstraintForFormFileParameters> proprietà su `true` in `Startup.ConfigureServices` :

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](index/samples/3.x/Startup.cs?name=snippet_ConfigureApiBehaviorOptions&highlight=2,4)]

::: moniker-end

::: moniker range="= aspnetcore-2.2"

[!code-csharp[](index/samples/2.x/2.2/Startup.cs?name=snippet_ConfigureApiBehaviorOptions&highlight=3,5)]

::: moniker-end

::: moniker range="= aspnetcore-2.1"

[!code-csharp[](index/samples/2.x/2.1/Startup.cs?name=snippet_ConfigureApiBehaviorOptions&highlight=1,3)]

::: moniker-end

::: moniker range=">= aspnetcore-2.2"

## <a name="problem-details-for-error-status-codes"></a>Dettagli del problema per i codici di stato di errore

Con la versione di compatibilità 2.2 o successiva, MVC consente di trasformare un risultato di errore (un risultato con codice di stato 400 o superiore) in un risultato con <xref:Microsoft.AspNetCore.Mvc.ProblemDetails>. Il tipo `ProblemDetails` si basa sulla [specifica RFC 7807](https://tools.ietf.org/html/rfc7807) per fornire dettagli sull'errore leggibili dal computer in una risposta HTTP.

Si consideri il codice seguente in un'azione del controller:

[!code-csharp[](index/samples/2.x/2.2/Controllers/PetsController.cs?name=snippet_ProblemDetailsStatusCode)]

Il `NotFound` metodo produce un codice di stato HTTP 404 con un `ProblemDetails` corpo. Ad esempio:

```json
{
  type: "https://tools.ietf.org/html/rfc7231#section-6.5.4",
  title: "Not Found",
  status: 404,
  traceId: "0HLHLV31KRN83:00000001"
}
```

### <a name="disable-problemdetails-response"></a>Disabilitare la risposta ProblemDetails

La creazione automatica di un oggetto `ProblemDetails` per i codici di stato di errore è disabilitata quando la <xref:Microsoft.AspNetCore.Mvc.ApiBehaviorOptions.SuppressMapClientErrors%2A> proprietà è impostata su `true` . Aggiungere il codice seguente a `Startup.ConfigureServices`:

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](index/samples/3.x/Startup.cs?name=snippet_ConfigureApiBehaviorOptions&highlight=2,7)]

::: moniker-end

::: moniker range="= aspnetcore-2.2"

[!code-csharp[](index/samples/2.x/2.2/Startup.cs?name=snippet_ConfigureApiBehaviorOptions&highlight=3,8)]

::: moniker-end

<a name="consumes"></a>

## <a name="define-supported-request-content-types-with-the-consumes-attribute"></a>Definire i tipi di contenuto della richiesta supportati con l'attributo [consums]

Per impostazione predefinita, un'azione supporta tutti i tipi di contenuto della richiesta disponibili. Se, ad esempio, un'app è configurata in modo da supportare i [formattatori di input](xref:mvc/models/model-binding#input-formatters)sia JSON che XML, un'azione supporta più tipi di contenuto, tra cui `application/json` e `application/xml` .

L'attributo [[consums]](<xref:Microsoft.AspNetCore.Mvc.ConsumesAttribute>) consente a un'azione di limitare i tipi di contenuto della richiesta supportati. Applicare l' `[Consumes]` attributo a un'azione o a un controller, specificando uno o più tipi di contenuto:

```csharp
[HttpPost]
[Consumes("application/xml")]
public IActionResult CreateProduct(Product product)
```

Nel codice precedente, l' `CreateProduct` azione specifica il tipo di contenuto `application/xml` . Le richieste indirizzate a questa azione devono specificare un' `Content-Type` intestazione `application/xml` . Le richieste che non specificano un' `Content-Type` intestazione di `application/xml` generano una risposta del [tipo di supporto 415 non supportato](https://developer.mozilla.org/docs/Web/HTTP/Status/415) .

L' `[Consumes]` attributo consente anche a un'azione di influenzare la selezione in base al tipo di contenuto di una richiesta in ingresso applicando un vincolo di tipo. Si consideri l'esempio seguente:

[!code-csharp[](index/samples/3.x/Controllers/ConsumesController.cs?name=snippet_Class)]

Nel codice precedente, `ConsumesController` è configurato per gestire le richieste inviate all' `https://localhost:5001/api/Consumes` URL. Entrambe le azioni del controller, `PostJson` e `PostForm` , gestiscono le richieste post con lo stesso URL. Senza l' `[Consumes]` attributo che applica un vincolo di tipo, viene generata un'eccezione di corrispondenza ambigua.

L' `[Consumes]` attributo viene applicato a entrambe le azioni. L' `PostJson` azione gestisce le richieste inviate con un' `Content-Type` intestazione `application/json` . L' `PostForm` azione gestisce le richieste inviate con un' `Content-Type` intestazione `application/x-www-form-urlencoded` . 

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:web-api/action-return-types>
* <xref:web-api/handle-errors>
* <xref:web-api/advanced/custom-formatters>
* <xref:web-api/advanced/formatting>
* <xref:tutorials/web-api-help-pages-using-swagger>
* <xref:mvc/controllers/routing>
* [Microsoft Learn: creare un'API Web con ASP.NET Core](/learn/modules/build-web-api-aspnet-core/)
