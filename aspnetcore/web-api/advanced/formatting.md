---
title: Formattare i dati di risposta nell'API Web ASP.NET Core
author: rick-anderson
description: Informazioni su come formattare i dati di risposta nell'API Web ASP.NET Core.
ms.author: riande
ms.custom: H1Hack27Feb2017
ms.date: 1/28/2021
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
uid: web-api/advanced/formatting
ms.openlocfilehash: e3e176072032494f94f120efabb95d6c127fb5e6
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102587814"
---
# <a name="format-response-data-in-aspnet-core-web-api"></a>Formattare i dati di risposta nell'API Web ASP.NET Core

Di [Rick Anderson](https://twitter.com/RickAndMSFT) e [Steve Smith](https://ardalis.com/)

ASP.NET Core MVC supporta la formattazione dei dati di risposta. I dati di risposta possono essere formattati usando formati specifici o in risposta al formato richiesto dal client.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/web-api/advanced/formatting) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="format-specific-action-results"></a>Risultati azione specifici del formato

Alcuni tipi di risultati di azioni sono specifici di un particolare formato, ad esempio <xref:Microsoft.AspNetCore.Mvc.JsonResult> e <xref:Microsoft.AspNetCore.Mvc.ContentResult>. Le azioni possono restituire risultati formattati in un formato specifico, indipendentemente dalle preferenze dei client. Ad esempio, la restituzione di `JsonResult` restituisce dati in formato JSON. La restituzione `ContentResult` di o di una stringa restituisce dati di stringa in formato testo normale.

Non è necessario eseguire un'azione per restituire un tipo specifico. ASP.NET Core supporta qualsiasi valore restituito da un oggetto.  I risultati di azioni che restituiscono oggetti che non sono <xref:Microsoft.AspNetCore.Mvc.IActionResult> tipi vengono serializzati usando l' <xref:Microsoft.AspNetCore.Mvc.Formatters.IOutputFormatter> implementazione appropriata. Per altre informazioni, vedere <xref:web-api/action-return-types>.

Il metodo helper incorporato <xref:Microsoft.AspNetCore.Mvc.ControllerBase.Ok*> restituisce dati in formato JSON: [!code-csharp[](./formatting/sample/Controllers/AuthorsController.cs?name=snippet_get)]

Il download di esempio restituisce l'elenco degli autori. Utilizzando gli strumenti di sviluppo del browser [F12 o l'](https://www.getpostman.com/tools) autore del codice precedente:

* Viene visualizzata l'intestazione della risposta contenente **Content-Type:** `application/json; charset=utf-8` .
* Verranno visualizzate le intestazioni della richiesta. Ad esempio, l' `Accept` intestazione. L' `Accept` intestazione viene ignorata dal codice precedente.

Per restituire dati in formato testo normale, usare <xref:Microsoft.AspNetCore.Mvc.ContentResult> e l'helper <xref:Microsoft.AspNetCore.Mvc.ControllerBase.Content%2A>:

[!code-csharp[](./formatting/sample/Controllers/AuthorsController.cs?name=snippet_about)]

Nel codice precedente, l'oggetto `Content-Type` restituito è `text/plain` . La restituzione di una stringa fornisce `Content-Type` `text/plain` :

[!code-csharp[](./formatting/sample/Controllers/AuthorsController.cs?name=snippet_string)]

Per le azioni con più tipi restituiti, restituire `IActionResult` . Ad esempio, la restituzione di codici di stato HTTP diversi in base al risultato delle operazioni eseguite.

## <a name="content-negotiation"></a>Negoziazione del contenuto

La negoziazione del contenuto si verifica quando il client specifica un' [intestazione Accept](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html). Il formato predefinito usato da ASP.NET Core è [JSON](https://json.org/). Negoziazione del contenuto:

* Implementato da <xref:Microsoft.AspNetCore.Mvc.ObjectResult> .
* Incorporata nei risultati dell'azione specifici del codice di stato restituiti dai metodi helper. I metodi helper dei risultati delle azioni sono basati su `ObjectResult` .

Quando viene restituito un tipo di modello, il tipo restituito è `ObjectResult` .

Il metodo di azione seguente usa i metodi helper `Ok` e `NotFound`:

[!code-csharp[](./formatting/sample/Controllers/AuthorsController.cs?name=snippet_search)]

Per impostazione predefinita, ASP.NET Core supporta i `application/json` `text/json` tipi di supporto, e `text/plain` . Gli strumenti, ad esempio [Fiddler](https://www.telerik.com/fiddler) o [post](https://www.getpostman.com/tools) , possono impostare l' `Accept` intestazione della richiesta per specificare il formato restituito. Quando l' `Accept` intestazione contiene un tipo supportato dal server, viene restituito tale tipo. Nella sezione successiva viene illustrato come aggiungere formattatori aggiuntivi.

Le azioni del controller possono restituire POCO (Plain Old CLR Objects). Quando viene restituito un oggetto POCO, il runtime crea automaticamente un `ObjectResult` oggetto che esegue il wrapping dell'oggetto. Il client ottiene l'oggetto serializzato formattato. Se l'oggetto restituito è `null` , `204 No Content` viene restituita una risposta.

Restituzione di un tipo di oggetto:

[!code-csharp[](./formatting/sample/Controllers/AuthorsController.cs?name=snippet_alias)]

Nel codice precedente, una richiesta di un alias autore valido restituisce una `200 OK` risposta con i dati dell'autore. Una richiesta di un alias non valido restituisce una `204 No Content` risposta.

### <a name="the-accept-header"></a>Intestazione Accept

La *negoziazione* del contenuto si verifica quando `Accept` viene visualizzata un'intestazione nella richiesta. Quando una richiesta contiene un'intestazione Accept, ASP.NET Core:

* Enumera i tipi di supporto nell'intestazione Accept in ordine di preferenza.
* Tenta di trovare un formattatore in grado di generare una risposta in uno dei formati specificati.

Se non viene trovato alcun formattatore in grado di soddisfare la richiesta del client, ASP.NET Core:

* Restituisce `406 Not Acceptable` se <xref:Microsoft.AspNetCore.Mvc.MvcOptions.ReturnHttpNotAcceptable?displayProperty=nameWithType> è impostato su `true` , o-
* Tenta di trovare il primo formattatore in grado di generare una risposta.

Se non è configurato alcun formattatore per il formato richiesto, viene utilizzato il primo formattatore in grado di formattare l'oggetto. Se `Accept` nella richiesta non è presente alcuna intestazione:

* Il primo formattatore in grado di gestire l'oggetto viene utilizzato per serializzare la risposta.
* Non si verifica alcuna negoziazione. Il server sta determinando il formato da restituire.

Se l'intestazione Accept contiene `*/*` , l'intestazione viene ignorata, a meno che non `RespectBrowserAcceptHeader` sia impostato su true in <xref:Microsoft.AspNetCore.Mvc.MvcOptions> .

### <a name="browsers-and-content-negotiation"></a>Browser e negoziazione del contenuto

A differenza dei client API tipici, i Web browser forniscono le `Accept` intestazioni. Web browser specificare molti formati, inclusi i caratteri jolly. Per impostazione predefinita, quando il Framework rileva che la richiesta viene ricevuta da un browser:

* L' `Accept` intestazione viene ignorata.
* Il contenuto viene restituito in JSON, a meno che non sia configurato diversamente.

Ciò offre un'esperienza più coerente nei browser quando si utilizzano le API.

Per configurare un'app in modo da rispettare le intestazioni Accept del browser, impostare <xref:Microsoft.AspNetCore.Mvc.MvcOptions.RespectBrowserAcceptHeader> su `true` :

::: moniker range=">= aspnetcore-3.0"
[!code-csharp[](./formatting/3.0sample/StartupRespectBrowserAcceptHeader.cs?name=snippet)]
::: moniker-end
::: moniker range="< aspnetcore-3.0"
[!code-csharp[](./formatting/sample/StartupRespectBrowserAcceptHeader.cs?name=snippet)]
::: moniker-end

### <a name="configure-formatters"></a>Configura formattatori

Le app che devono supportare formati aggiuntivi possono aggiungere i pacchetti NuGet appropriati e configurare il supporto. Esistono formattatori separati per input e output. I formattatori di input vengono utilizzati dall' [associazione di modelli](xref:mvc/models/model-binding). I formattatori di output vengono utilizzati per formattare le risposte. Per informazioni sulla creazione di un formattatore personalizzato, vedere [formattatori personalizzati](xref:web-api/advanced/custom-formatters).

::: moniker range=">= aspnetcore-3.0"

### <a name="add-xml-format-support"></a>Aggiungere il supporto per il formato XML

I formattatori XML implementati usando <xref:System.Xml.Serialization.XmlSerializer> vengono configurati chiamando <xref:Microsoft.Extensions.DependencyInjection.MvcXmlMvcBuilderExtensions.AddXmlSerializerFormatters*> :

[!code-csharp[](./formatting/3.0sample/Startup.cs?name=snippet)]

Il codice precedente serializza i risultati usando `XmlSerializer` .

Quando si usa il codice precedente, i metodi del controller restituiscono il formato appropriato in base all'intestazione della richiesta `Accept` .

### <a name="configure-systemtextjson-based-formatters"></a>Configurare `System.Text.Json` formattatori basati

È possibile configurare le funzionalità per i `System.Text.Json` formattatori basati usando <xref:Microsoft.AspNetCore.Mvc.JsonOptions.JsonSerializerOptions?displayProperty=fullName> . La formattazione predefinita è camelCase. Il codice evidenziato seguente imposta la formattazione PascalCase:

[!code-csharp[](./formatting/5.0samples/WebAPI5PascalCase/Startup.cs?name=snippet&highlight=4-5)]

Il metodo di azione seguente chiama [ControllerBase. problem](xref:Microsoft.AspNetCore.Mvc.ControllerBase.Problem%2A) per creare una <xref:Microsoft.AspNetCore.Mvc.ProblemDetails> risposta:

[!code-csharp[](formatting/5.0samples/WebAPI5PascalCase/Controllers/WeatherForecastController.cs?name=snippet&highlight=4)]

Con il codice precedente:

  * `https://localhost:5001/WeatherForecast/temperature` Restituisce PascalCase.
  * `https://localhost:5001/WeatherForecast/error` Restituisce camelCase. La risposta di errore è sempre camelCase, anche quando l'app imposta il formato su PascalCase. `ProblemDetails` segue la [RFC 7807](https://tools.ietf.org/html/rfc7807#appendix-A), che specifica il minuscolo

Il codice seguente imposta PascalCase e aggiunge un convertitore personalizzato:

```csharp
services.AddControllers().AddJsonOptions(options =>
{
    // Use the default property (Pascal) casing.
    options.JsonSerializerOptions.PropertyNamingPolicy = null;

    // Configure a custom converter.
    options.JsonSerializerOptions.Converters.Add(new MyCustomJsonConverter());
});
```

È possibile configurare le opzioni di serializzazione dell'output, in base alle singole azioni, usando `JsonResult` . Ad esempio:

```csharp
public IActionResult Get()
{
    return Json(model, new JsonSerializerOptions
    {
        WriteIndented = true,
    });
}
```

### <a name="add-newtonsoftjson-based-json-format-support"></a>Aggiungere il supporto del formato JSON basato su Newtonsoft.Json

Prima di ASP.NET Core 3,0, i formattatori JSON usati per impostazione predefinita venivano implementati utilizzando il `Newtonsoft.Json` pacchetto. In ASP.NET Core 3.0 o versione successiva, i formattatori JSON predefiniti sono basati su `System.Text.Json`. Il supporto per `Newtonsoft.Json` formattatori e funzionalità basati è disponibile installando il [`Microsoft.AspNetCore.Mvc.NewtonsoftJson`](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.NewtonsoftJson/) pacchetto NuGet e configurarlo in `Startup.ConfigureServices` .

[!code-csharp[](./formatting/3.0sample/StartupNewtonsoftJson.cs?name=snippet)]

Nel codice precedente, la chiamata a `AddNewtonsoftJson` Configura le seguenti funzionalità dell'API Web, MVC e Razor pages da usare `Newtonsoft.Json` :

* Formattatori di input e di output che leggono e scrivono JSON
* <xref:Microsoft.AspNetCore.Mvc.JsonResult>
* [JSON Patch](xref:web-api/jsonpatch)
* <xref:Microsoft.AspNetCore.Mvc.Rendering.IJsonHelper>
* [TempData](xref:fundamentals/app-state#tempdata)

Alcune funzionalità potrebbero non funzionare correttamente con `System.Text.Json` formattatori basati su e richiedono un riferimento ai `Newtonsoft.Json` formattatori basati su. Continuare a usare i `Newtonsoft.Json` formattatori basati su se l'app:

* Utilizza `Newtonsoft.Json` gli attributi. Ad esempio, `[JsonProperty]` o `[JsonIgnore]`.
* Personalizza le impostazioni di serializzazione.
* Si basa sulle funzionalità `Newtonsoft.Json` fornite da.
* Configura `Microsoft.AspNetCore.Mvc.JsonResult.SerializerSettings`. Prima di ASP.NET Core 3.0 `JsonResult.SerializerSettings` accetta un'istanza di `JsonSerializerSettings` specifica di `Newtonsoft.Json`.
* Genera documentazione [OpenAPI](<xref:tutorials/web-api-help-pages-using-swagger>).

Le funzionalità per i `Newtonsoft.Json` formattatori basati su possono essere configurate utilizzando `Microsoft.AspNetCore.Mvc.MvcNewtonsoftJsonOptions.SerializerSettings` :

```csharp
services.AddControllers().AddNewtonsoftJson(options =>
{
    // Use the default property (Pascal) casing
    options.SerializerSettings.ContractResolver = new DefaultContractResolver();

    // Configure a custom converter
    options.SerializerSettings.Converters.Add(new MyCustomJsonConverter());
});
```

È possibile configurare le opzioni di serializzazione dell'output, in base alle singole azioni, usando `JsonResult` . Ad esempio:

```csharp
public IActionResult Get()
{
    return Json(model, new JsonSerializerSettings
    {
        Formatting = Formatting.Indented,
    });
}
```

::: moniker-end

::: moniker range="<= aspnetcore-2.2"

### <a name="add-xml-format-support"></a>Aggiungere il supporto per il formato XML

La formattazione XML richiede il pacchetto NuGet [Microsoft.AspNetCore.Mvc.Formatters.Xml](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.Formatters.Xml/) .

I formattatori XML implementati usando <xref:System.Xml.Serialization.XmlSerializer> vengono configurati chiamando <xref:Microsoft.Extensions.DependencyInjection.MvcXmlMvcBuilderExtensions.AddXmlSerializerFormatters*> :

[!code-csharp[](./formatting/sample/Startup.cs?name=snippet)]

Il codice precedente serializza i risultati usando `XmlSerializer` .

Quando si usa il codice precedente, i metodi del controller devono restituire il formato appropriato in base all'intestazione della richiesta `Accept` .

::: moniker-end

### <a name="specify-a-format"></a>Specificare un formato

Per limitare i formati di risposta, applicare il [`[Produces]`](xref:Microsoft.AspNetCore.Mvc.ProducesAttribute) filtro. Come la maggior parte dei [filtri](xref:mvc/controllers/filters), `[Produces]` può essere applicato all'azione, al controller o all'ambito globale:

[!code-csharp[](./formatting/3.0sample/Controllers/WeatherForecastController.cs?name=snippet)]

Il [`[Produces]`](xref:Microsoft.AspNetCore.Mvc.ProducesAttribute) filtro precedente:

* Impone a tutte le azioni all'interno del controller di restituire risposte in formato JSON.
* Se sono configurati altri formattatori e il client specifica un formato diverso, viene restituito JSON.

Per ulteriori informazioni, vedere [filtri](xref:mvc/controllers/filters).

### <a name="special-case-formatters"></a>Formattatori case speciali

Alcuni casi speciali vengono implementati tramite formattatori predefiniti. Per impostazione predefinita, `string` i tipi restituiti vengono formattati come *testo/normale* (*testo/HTML* se richiesto tramite l' `Accept` intestazione). Questo comportamento può essere eliminato rimuovendo <xref:Microsoft.AspNetCore.Mvc.Formatters.StringOutputFormatter> . I formattatori vengono rimossi nel `ConfigureServices` metodo. Le azioni con un tipo restituito dall'oggetto modello restituiscono quando viene restituito `204 No Content` `null` . Questo comportamento può essere eliminato rimuovendo <xref:Microsoft.AspNetCore.Mvc.Formatters.HttpNoContentOutputFormatter> . Il codice seguente rimuove `StringOutputFormatter` e `HttpNoContentOutputFormatter`.

::: moniker range=">= aspnetcore-3.0"
[!code-csharp[](./formatting/3.0sample/StartupStringOutputFormatter.cs?name=snippet)]
::: moniker-end
::: moniker range="< aspnetcore-3.0"
[!code-csharp[](./formatting/sample/StartupStringOutputFormatter.cs?name=snippet)]
::: moniker-end

Senza `StringOutputFormatter` , il formattatore JSON incorporato formatta i `string` tipi restituiti. Se il formattatore JSON incorporato viene rimosso ed è disponibile un formattatore XML, il formattatore XML formatta i `string` tipi restituiti. In caso contrario, i `string` tipi restituiti restituiscono `406 Not Acceptable` .

Senza `HttpNoContentOutputFormatter`, gli oggetti Null vengono formattati tramite il formattatore configurato. Ad esempio:

* Il formattatore JSON restituisce una risposta con un corpo di `null` .
* Il formattatore XML restituisce un elemento XML vuoto con l'attributo `xsi:nil="true"` impostato.

## <a name="response-format-url-mappings"></a>Mapping URL formato risposta

I client possono richiedere un particolare formato come parte dell'URL, ad esempio:

* Nella stringa di query o in una parte del percorso.
* Utilizzando un'estensione di file specifica del formato, ad esempio XML o JSON.

Il mapping dal percorso della richiesta deve essere specificato nella route usata dall'API. Ad esempio:

[!code-csharp[](./formatting/sample/Controllers/ProductsController.cs?name=snippet)]

La route precedente consente di specificare il formato richiesto come estensione di file facoltativa. L' [`[FormatFilter]`](xref:Microsoft.AspNetCore.Mvc.FormatFilterAttribute) attributo controlla l'esistenza del valore di formato in `RouteData` ed esegue il mapping del formato della risposta al formattatore appropriato al momento della creazione della risposta.

|           Route        |             Formattatore              |
|------------------------|------------------------------------|
|   `/api/products/5`    |    Formattatore di output predefinito    |
| `/api/products/5.json` | Formattatore JSON (se configurato) |
| `/api/products/5.xml`  | Formattatore XML (se configurato)  |
