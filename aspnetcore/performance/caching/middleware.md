---
title: Middleware di memorizzazione nella cache delle risposte in ASP.NET Core
author: rick-anderson
description: Informazioni su come configurare e usare il middleware di memorizzazione nella cache delle risposte in ASP.NET Core.
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 02/07/2020
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
uid: performance/caching/middleware
ms.openlocfilehash: 0f1f5dfcb9595270a9659a02141f7d1eba5c44ef
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102587697"
---
# <a name="response-caching-middleware-in-aspnet-core"></a>Middleware di memorizzazione nella cache delle risposte in ASP.NET Core

A cura di [John Luo](https://github.com/JunTaoLuo)

::: moniker range=">= aspnetcore-3.0"

Questo articolo illustra come configurare il middleware di memorizzazione nella cache delle risposte in un'app ASP.NET Core. Il middleware determina quando le risposte sono memorizzabili nella cache, archivia le risposte e fornisce risposte dalla cache. Per un'introduzione alla memorizzazione nella cache HTTP e all' [`[ResponseCache]`](xref:Microsoft.AspNetCore.Mvc.ResponseCacheAttribute) attributo, vedere [caching delle risposte](xref:performance/caching/response).

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/performance/caching/middleware/samples) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="configuration"></a>Configurazione

Il middleware di memorizzazione nella cache delle risposte è disponibile in modo implicito per le app ASP.NET Core tramite il Framework condiviso.

In `Startup.ConfigureServices` aggiungere il middleware di caching della risposta alla raccolta di servizi:

[!code-csharp[](middleware/samples/3.x/ResponseCachingMiddleware/Startup.cs?name=snippet1&highlight=3)]

Configurare l'app per l'uso del middleware con il <xref:Microsoft.AspNetCore.Builder.ResponseCachingExtensions.UseResponseCaching*> metodo di estensione, che aggiunge il middleware alla pipeline di elaborazione delle richieste in `Startup.Configure` :

[!code-csharp[](middleware/samples/3.x/ResponseCachingMiddleware/Startup.cs?name=snippet2&highlight=17)]

> [!WARNING]
> <xref:Owin.CorsExtensions.UseCors%2A> deve essere chiamato prima <xref:Microsoft.AspNetCore.Builder.ResponseCachingExtensions.UseResponseCaching%2A> quando si usa il [middleware CORS](xref:security/cors).

L'app di esempio aggiunge intestazioni per controllare la memorizzazione nella cache nelle richieste successive:

* [Cache-Control](https://tools.ietf.org/html/rfc7234#section-5.2): memorizza nella cache le risposte memorizzabili nella cache per un massimo di 10 secondi.
* [Vary](https://tools.ietf.org/html/rfc7231#section-7.1.4): configura il middleware in modo da fornire una risposta memorizzata nella cache solo se l'intestazione [Accept-Encoding](https://tools.ietf.org/html/rfc7231#section-5.3.4) delle richieste successive corrisponde a quella della richiesta originale.

[!code-csharp[](middleware/samples_snippets/3.x/AddHeaders.cs)]

Le intestazioni precedenti non vengono scritte nella risposta e vengono sottoposte a override quando un controller, un'azione o una Razor pagina:

* Dispone di un attributo [[ResponseCache]](xref:Microsoft.AspNetCore.Mvc.ResponseCacheAttribute) . Questo vale anche se una proprietà non è impostata. Se, ad esempio, si omette la proprietà [VaryByHeader](./response.md#vary) , l'intestazione corrispondente verrà rimossa dalla risposta.

Il middleware di memorizzazione nella cache delle risposte memorizza nella cache solo le risposte del server che generano un codice di stato 200 (OK). Qualsiasi altra risposta, incluse le [pagine di errore](xref:fundamentals/error-handling), viene ignorata dal middleware.

> [!WARNING]
> Le risposte contenenti contenuto per i client autenticati devono essere contrassegnate come non memorizzabili nella cache per impedire al middleware di archiviare e servire tali risposte. Per informazioni dettagliate sul modo in cui il middleware determina se una risposta è memorizzabile nella cache, vedere [condizioni per la memorizzazione nella cache](#conditions-for-caching) .

## <a name="options"></a>Opzioni

Le opzioni di memorizzazione nella cache delle risposte sono illustrate nella tabella seguente.

| Opzione | Descrizione |
| ------ | ----------- |
| <xref:Microsoft.AspNetCore.ResponseCaching.ResponseCachingOptions.MaximumBodySize> | Dimensioni maggiori memorizzabili nella cache per il corpo della risposta in byte. Il valore predefinito è `64 * 1024 * 1024` (64 MB). |
| <xref:Microsoft.AspNetCore.ResponseCaching.ResponseCachingOptions.SizeLimit> | Limite delle dimensioni per il middleware della cache di risposta in byte. Il valore predefinito è `100 * 1024 * 1024` (100 MB). |
| <xref:Microsoft.AspNetCore.ResponseCaching.ResponseCachingOptions.UseCaseSensitivePaths> | Determina se le risposte vengono memorizzate nella cache nei percorsi che fanno distinzione Il valore predefinito è `false`. |

Nell'esempio seguente il middleware viene configurato per:

* Risposte della cache con dimensioni del corpo inferiori o uguali a 1.024 byte.
* Archiviare le risposte in base ai percorsi con distinzione tra maiuscole e minuscole. Ad esempio, `/page1` e `/Page1` vengono archiviati separatamente.

```csharp
services.AddResponseCaching(options =>
{
    options.MaximumBodySize = 1024;
    options.UseCaseSensitivePaths = true;
});
```

## <a name="varybyquerykeys"></a>VaryByQueryKeys

Quando si usano i controller dell'API Web o MVC o i Razor modelli di pagina di pagine, l' [`[ResponseCache]`](xref:Microsoft.AspNetCore.Mvc.ResponseCacheAttribute) attributo specifica i parametri necessari per impostare le intestazioni appropriate per la memorizzazione nella cache delle risposte. L'unico parametro dell' `[ResponseCache]` attributo che richiede rigorosamente il middleware è <xref:Microsoft.AspNetCore.Mvc.ResponseCacheAttribute.VaryByQueryKeys> , che non corrisponde a un'intestazione HTTP effettiva. Per altre informazioni, vedere <xref:performance/caching/response#responsecache-attribute>.

Quando non si usa l' `[ResponseCache]` attributo, la memorizzazione nella cache delle risposte può essere variata con `VaryByQueryKeys` . Usare <xref:Microsoft.AspNetCore.ResponseCaching.ResponseCachingFeature> direttamente da [HttpContext. Features](xref:Microsoft.AspNetCore.Http.HttpContext.Features):

```csharp
var responseCachingFeature = context.HttpContext.Features.Get<IResponseCachingFeature>();

if (responseCachingFeature != null)
{
    responseCachingFeature.VaryByQueryKeys = new[] { "MyKey" };
}
```

L'utilizzo di un singolo valore uguale a `*` in `VaryByQueryKeys` varia la cache in base a tutti i parametri di query della richiesta.

## <a name="http-headers-used-by-response-caching-middleware"></a>Intestazioni HTTP usate dal middleware di memorizzazione nella cache delle risposte

Nella tabella seguente vengono fornite informazioni sulle intestazioni HTTP che influiscono sulla memorizzazione nella cache delle risposte.

| Intestazione | Dettagli |
| ------ | ------- |
| `Authorization` | Se l'intestazione esiste, la risposta non viene memorizzata nella cache. |
| `Cache-Control` | Il middleware considera solo le risposte di memorizzazione nella cache contrassegnate con la `public` direttiva della cache. Controllare la memorizzazione nella cache con i parametri seguenti:<ul><li>validità massima</li><li>numero massimo di&#8224; obsoleti</li><li>min-Fresh</li><li>must-revalidate</li><li>no-cache</li><li>Nessun archivio</li><li>solo-if-Cached</li><li>private</li><li>public</li><li>s-maxage</li><li>proxy-riconvalida&#8225;</li></ul>&#8224;se non viene specificato alcun limite a `max-stale` , il middleware non esegue alcuna azione.<br>&#8225;`proxy-revalidate` ha lo stesso effetto di `must-revalidate` .<br><br>Per altre informazioni, vedere [RFC 7231: direttive Cache-Control della richiesta](https://tools.ietf.org/html/rfc7234#section-5.2.1). |
| `Pragma` | Un' `Pragma: no-cache` intestazione nella richiesta produce lo stesso effetto di `Cache-Control: no-cache` . Questa intestazione viene sottoposta a override dalle direttive pertinenti nell' `Cache-Control` intestazione, se presente. Considerato per compatibilità con le versioni precedenti di HTTP/1.0. |
| `Set-Cookie` | Se l'intestazione esiste, la risposta non viene memorizzata nella cache. Qualsiasi middleware nella pipeline di elaborazione della richiesta che imposta uno o più cookie impedisce al middleware di memorizzazione nella cache della risposta di memorizzare nella cache la risposta, ad esempio il [ cookie provider TempData basato su](xref:fundamentals/app-state#tempdata).  |
| `Vary` | L' `Vary` intestazione viene utilizzata per variare la risposta memorizzata nella cache da un'altra intestazione. Ad esempio, memorizzare nella cache le risposte per codifica includendo l' `Vary: Accept-Encoding` intestazione, che memorizza nella cache le risposte per le richieste con intestazioni `Accept-Encoding: gzip` e `Accept-Encoding: text/plain` separatamente. Una risposta con un valore di intestazione `*` non viene mai archiviata. |
| `Expires` | Una risposta ritenuta obsoleta da questa intestazione non viene archiviata o recuperata a meno che non venga sottoposta a override da altre `Cache-Control` intestazioni. |
| `If-None-Match` | La risposta completa viene servita dalla cache se il valore non è `*` e la `ETag` della risposta non corrisponde ad alcuno dei valori forniti. In caso contrario, viene servita una risposta 304 (non modificata). |
| `If-Modified-Since` | Se l' `If-None-Match` intestazione non è presente, viene fornita una risposta completa dalla cache se la data di risposta memorizzata nella cache è successiva al valore specificato. In caso contrario, viene servita una risposta *304 non modificata* . |
| `Date` | Quando viene servito dalla cache, l' `Date` intestazione viene impostata dal middleware se non è stata specificata nella risposta originale. |
| `Content-Length` | Quando viene servito dalla cache, l' `Content-Length` intestazione viene impostata dal middleware se non è stata specificata nella risposta originale. |
| `Age` | L' `Age` intestazione inviata nella risposta originale viene ignorata. Il middleware calcola un nuovo valore quando serve una risposta memorizzata nella cache. |

## <a name="caching-respects-request-cache-control-directives"></a>Caching rispetta le direttive Cache-Control richieste

Il middleware rispetta le regole della [specifica HTTP 1,1 Caching](https://tools.ietf.org/html/rfc7234#section-5.2). Le regole richiedono una cache per rispettare un' `Cache-Control` intestazione valida inviata dal client. In base alla specifica, un client può effettuare richieste con un `no-cache` valore di intestazione e forzare il server a generare una nuova risposta per ogni richiesta. Attualmente, non esiste alcun controllo dello sviluppatore su questo comportamento di Caching quando si usa il middleware perché il middleware rispetta la specifica di Caching ufficiale.

Per un maggiore controllo sul comportamento della memorizzazione nella cache, esplorare altre funzionalità di memorizzazione nella cache di ASP.NET Core. Vedere gli argomenti seguenti:

* <xref:performance/caching/memory>
* <xref:performance/caching/distributed>
* <xref:mvc/views/tag-helpers/builtin-th/cache-tag-helper>
* <xref:mvc/views/tag-helpers/builtin-th/distributed-cache-tag-helper>

## <a name="troubleshooting"></a>Risoluzione dei problemi

Se il comportamento di memorizzazione nella cache non è quello previsto, verificare che le risposte siano memorizzabili nella cache e in grado di essere servite dalla cache. Esaminare le intestazioni in ingresso della richiesta e le intestazioni in uscita della risposta. Abilitare la [registrazione](xref:fundamentals/logging/index) per facilitare il debug.

Durante il test e la risoluzione dei problemi relativi al comportamento di Caching, un browser può impostare intestazioni di richiesta che influiscono sulla memorizzazione nella cache in modi indesiderati Ad esempio, un browser può impostare l' `Cache-Control` intestazione su `no-cache` o `max-age=0` durante l'aggiornamento di una pagina. Gli strumenti seguenti possono impostare in modo esplicito le intestazioni delle richieste e sono preferiti per il test della memorizzazione nella cache:

* [Fiddler](https://www.telerik.com/fiddler)
* [Postman](https://www.getpostman.com/)

### <a name="conditions-for-caching"></a>Condizioni per la memorizzazione nella cache

* La richiesta deve avere come risultato una risposta del server con un codice di stato 200 (OK).
* Il metodo di richiesta deve essere GET o HEAD.
* In `Startup.Configure` il middleware di memorizzazione nella cache delle risposte deve essere inserito prima del middleware che richiede la memorizzazione nella cache. Per altre informazioni, vedere <xref:fundamentals/middleware/index>.
* L' `Authorization` intestazione non deve essere presente.
* `Cache-Control` i parametri di intestazione devono essere validi e la risposta deve essere contrassegnata `public` e non contrassegnata `private` .
* L'intestazione `Pragma: no-cache` non deve essere presente se l' `Cache-Control` intestazione non è presente, in quanto l'intestazione `Cache-Control` sostituisce l'intestazione `Pragma` quando è presente.
* L' `Set-Cookie` intestazione non deve essere presente.
* `Vary` i parametri di intestazione devono essere validi e non uguali a `*` .
* Il `Content-Length` valore dell'intestazione (se impostato) deve corrispondere alla dimensione del corpo della risposta.
* <xref:Microsoft.AspNetCore.Http.Features.IHttpSendFileFeature>Non viene utilizzato.
* La risposta non deve essere obsoleta come specificato dall' `Expires` intestazione e dalle `max-age` `s-maxage` direttive della cache e.
* Il buffer delle risposte deve avere esito positivo. La dimensione della risposta deve essere minore di quella configurata o predefinita <xref:Microsoft.AspNetCore.ResponseCaching.ResponseCachingOptions.SizeLimit> . Le dimensioni del corpo della risposta devono essere inferiori a quelle configurate o predefinite <xref:Microsoft.AspNetCore.ResponseCaching.ResponseCachingOptions.MaximumBodySize> .
* La risposta deve essere memorizzata nella cache in base alle specifiche [RFC 7234](https://tools.ietf.org/html/rfc7234) . La direttiva, ad esempio, `no-store` non deve esistere nei campi di intestazione della richiesta o della risposta. Per informazioni dettagliate, vedere la *sezione 3: archiviazione delle risposte nelle cache* di [RFC 7234](https://tools.ietf.org/html/rfc7234) .

> [!NOTE]
> Il sistema antifalsificazione per la generazione di token protetti per evitare attacchi di richiesta intersito falsa (CSRF) imposta le `Cache-Control` `Pragma` intestazioni e su in `no-cache` modo che le risposte non vengano memorizzate nella cache. Per informazioni su come disabilitare i token antifalsificazione per gli elementi del form HTML, vedere <xref:security/anti-request-forgery#aspnet-core-antiforgery-configuration> .

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:fundamentals/startup>
* <xref:fundamentals/middleware/index>
* <xref:performance/caching/memory>
* <xref:performance/caching/distributed>
* <xref:fundamentals/change-tokens>
* <xref:performance/caching/response>
* <xref:mvc/views/tag-helpers/builtin-th/cache-tag-helper>
* <xref:mvc/views/tag-helpers/builtin-th/distributed-cache-tag-helper>

::: moniker-end

::: moniker range="< aspnetcore-3.0"

Questo articolo illustra come configurare il middleware di memorizzazione nella cache delle risposte in un'app ASP.NET Core. Il middleware determina quando le risposte sono memorizzabili nella cache, archivia le risposte e fornisce risposte dalla cache. Per un'introduzione alla memorizzazione nella cache HTTP e all' [`[ResponseCache]`](xref:Microsoft.AspNetCore.Mvc.ResponseCacheAttribute) attributo, vedere [caching delle risposte](xref:performance/caching/response).

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/performance/caching/middleware/samples) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="configuration"></a>Configurazione

Usare il [metapacchetto Microsoft. AspNetCore. app](xref:fundamentals/metapackage-app) o aggiungere un riferimento al pacchetto [Microsoft. AspNetCore. ResponseCaching](https://www.nuget.org/packages/Microsoft.AspNetCore.ResponseCaching/) .

In `Startup.ConfigureServices` aggiungere il middleware di caching della risposta alla raccolta di servizi:

[!code-csharp[](middleware/samples/2.x/ResponseCachingMiddleware/Startup.cs?name=snippet1&highlight=3)]

Configurare l'app per l'uso del middleware con il <xref:Microsoft.AspNetCore.Builder.ResponseCachingExtensions.UseResponseCaching*> metodo di estensione, che aggiunge il middleware alla pipeline di elaborazione delle richieste in `Startup.Configure` :

[!code-csharp[](middleware/samples/2.x/ResponseCachingMiddleware/Startup.cs?name=snippet2&highlight=14)]

L'app di esempio aggiunge intestazioni per controllare la memorizzazione nella cache nelle richieste successive:

* [Cache-Control](https://tools.ietf.org/html/rfc7234#section-5.2): memorizza nella cache le risposte memorizzabili nella cache per un massimo di 10 secondi.
* [Vary](https://tools.ietf.org/html/rfc7231#section-7.1.4): configura il middleware in modo da fornire una risposta memorizzata nella cache solo se l'intestazione [Accept-Encoding](https://tools.ietf.org/html/rfc7231#section-5.3.4) delle richieste successive corrisponde a quella della richiesta originale.

[!code-csharp[](middleware/samples_snippets/2.x/AddHeaders.cs)]

Le intestazioni precedenti non vengono scritte nella risposta e vengono sottoposte a override quando un controller, un'azione o una Razor pagina:

* Dispone di un attributo [[ResponseCache]](xref:Microsoft.AspNetCore.Mvc.ResponseCacheAttribute) . Questo vale anche se una proprietà non è impostata. Se, ad esempio, si omette la proprietà [VaryByHeader](./response.md#vary) , l'intestazione corrispondente verrà rimossa dalla risposta.

Il middleware di memorizzazione nella cache delle risposte memorizza nella cache solo le risposte del server che generano un codice di stato 200 (OK). Qualsiasi altra risposta, incluse le [pagine di errore](xref:fundamentals/error-handling), viene ignorata dal middleware.

> [!WARNING]
> Le risposte contenenti contenuto per i client autenticati devono essere contrassegnate come non memorizzabili nella cache per impedire al middleware di archiviare e servire tali risposte. Per informazioni dettagliate sul modo in cui il middleware determina se una risposta è memorizzabile nella cache, vedere [condizioni per la memorizzazione nella cache](#conditions-for-caching) .

## <a name="options"></a>Opzioni

Le opzioni di memorizzazione nella cache delle risposte sono illustrate nella tabella seguente.

| Opzione | Descrizione |
| ------ | ----------- |
| <xref:Microsoft.AspNetCore.ResponseCaching.ResponseCachingOptions.MaximumBodySize> | Dimensioni maggiori memorizzabili nella cache per il corpo della risposta in byte. Il valore predefinito è `64 * 1024 * 1024` (64 MB). |
| <xref:Microsoft.AspNetCore.ResponseCaching.ResponseCachingOptions.SizeLimit> | Limite delle dimensioni per il middleware della cache di risposta in byte. Il valore predefinito è `100 * 1024 * 1024` (100 MB). |
| <xref:Microsoft.AspNetCore.ResponseCaching.ResponseCachingOptions.UseCaseSensitivePaths> | Determina se le risposte vengono memorizzate nella cache nei percorsi che fanno distinzione Il valore predefinito è `false`. |

Nell'esempio seguente il middleware viene configurato per:

* Risposte della cache con dimensioni del corpo inferiori o uguali a 1.024 byte.
* Archiviare le risposte in base ai percorsi con distinzione tra maiuscole e minuscole. Ad esempio, `/page1` e `/Page1` vengono archiviati separatamente.

```csharp
services.AddResponseCaching(options =>
{
    options.MaximumBodySize = 1024;
    options.UseCaseSensitivePaths = true;
});
```

## <a name="varybyquerykeys"></a>VaryByQueryKeys

Quando si usano i controller dell'API Web o MVC o i Razor modelli di pagina di pagine, l' [`[ResponseCache]`](xref:Microsoft.AspNetCore.Mvc.ResponseCacheAttribute) attributo specifica i parametri necessari per impostare le intestazioni appropriate per la memorizzazione nella cache delle risposte. L'unico parametro dell' `[ResponseCache]` attributo che richiede rigorosamente il middleware è <xref:Microsoft.AspNetCore.Mvc.ResponseCacheAttribute.VaryByQueryKeys> , che non corrisponde a un'intestazione HTTP effettiva. Per altre informazioni, vedere <xref:performance/caching/response#responsecache-attribute>.

Quando non si usa l' `[ResponseCache]` attributo, la memorizzazione nella cache delle risposte può essere variata con `VaryByQueryKeys` . Usare <xref:Microsoft.AspNetCore.ResponseCaching.ResponseCachingFeature> direttamente da [HttpContext. Features](xref:Microsoft.AspNetCore.Http.HttpContext.Features):

```csharp
var responseCachingFeature = context.HttpContext.Features.Get<IResponseCachingFeature>();

if (responseCachingFeature != null)
{
    responseCachingFeature.VaryByQueryKeys = new[] { "MyKey" };
}
```

L'utilizzo di un singolo valore uguale a `*` in `VaryByQueryKeys` varia la cache in base a tutti i parametri di query della richiesta.

## <a name="http-headers-used-by-response-caching-middleware"></a>Intestazioni HTTP usate dal middleware di memorizzazione nella cache delle risposte

Nella tabella seguente vengono fornite informazioni sulle intestazioni HTTP che influiscono sulla memorizzazione nella cache delle risposte.

| Intestazione | Dettagli |
| ------ | ------- |
| `Authorization` | Se l'intestazione esiste, la risposta non viene memorizzata nella cache. |
| `Cache-Control` | Il middleware considera solo le risposte di memorizzazione nella cache contrassegnate con la `public` direttiva della cache. Controllare la memorizzazione nella cache con i parametri seguenti:<ul><li>validità massima</li><li>numero massimo di&#8224; obsoleti</li><li>min-Fresh</li><li>must-revalidate</li><li>no-cache</li><li>Nessun archivio</li><li>solo-if-Cached</li><li>private</li><li>public</li><li>s-maxage</li><li>proxy-riconvalida&#8225;</li></ul>&#8224;se non viene specificato alcun limite a `max-stale` , il middleware non esegue alcuna azione.<br>&#8225;`proxy-revalidate` ha lo stesso effetto di `must-revalidate` .<br><br>Per altre informazioni, vedere [RFC 7231: direttive Cache-Control della richiesta](https://tools.ietf.org/html/rfc7234#section-5.2.1). |
| `Pragma` | Un' `Pragma: no-cache` intestazione nella richiesta produce lo stesso effetto di `Cache-Control: no-cache` . Questa intestazione viene sottoposta a override dalle direttive pertinenti nell' `Cache-Control` intestazione, se presente. Considerato per compatibilità con le versioni precedenti di HTTP/1.0. |
| `Set-Cookie` | Se l'intestazione esiste, la risposta non viene memorizzata nella cache. Qualsiasi middleware nella pipeline di elaborazione della richiesta che imposta uno o più cookie impedisce al middleware di memorizzazione nella cache della risposta di memorizzare nella cache la risposta, ad esempio il [ cookie provider TempData basato su](xref:fundamentals/app-state#tempdata).  |
| `Vary` | L' `Vary` intestazione viene utilizzata per variare la risposta memorizzata nella cache da un'altra intestazione. Ad esempio, memorizzare nella cache le risposte per codifica includendo l' `Vary: Accept-Encoding` intestazione, che memorizza nella cache le risposte per le richieste con intestazioni `Accept-Encoding: gzip` e `Accept-Encoding: text/plain` separatamente. Una risposta con un valore di intestazione `*` non viene mai archiviata. |
| `Expires` | Una risposta ritenuta obsoleta da questa intestazione non viene archiviata o recuperata a meno che non venga sottoposta a override da altre `Cache-Control` intestazioni. |
| `If-None-Match` | La risposta completa viene servita dalla cache se il valore non è `*` e la `ETag` della risposta non corrisponde ad alcuno dei valori forniti. In caso contrario, viene servita una risposta 304 (non modificata). |
| `If-Modified-Since` | Se l' `If-None-Match` intestazione non è presente, viene fornita una risposta completa dalla cache se la data di risposta memorizzata nella cache è successiva al valore specificato. In caso contrario, viene servita una risposta *304 non modificata* . |
| `Date` | Quando viene servito dalla cache, l' `Date` intestazione viene impostata dal middleware se non è stata specificata nella risposta originale. |
| `Content-Length` | Quando viene servito dalla cache, l' `Content-Length` intestazione viene impostata dal middleware se non è stata specificata nella risposta originale. |
| `Age` | L' `Age` intestazione inviata nella risposta originale viene ignorata. Il middleware calcola un nuovo valore quando serve una risposta memorizzata nella cache. |

## <a name="caching-respects-request-cache-control-directives"></a>Caching rispetta le direttive Cache-Control richieste

Il middleware rispetta le regole della [specifica HTTP 1,1 Caching](https://tools.ietf.org/html/rfc7234#section-5.2). Le regole richiedono una cache per rispettare un' `Cache-Control` intestazione valida inviata dal client. In base alla specifica, un client può effettuare richieste con un `no-cache` valore di intestazione e forzare il server a generare una nuova risposta per ogni richiesta. Attualmente, non esiste alcun controllo dello sviluppatore su questo comportamento di Caching quando si usa il middleware perché il middleware rispetta la specifica di Caching ufficiale.

Per un maggiore controllo sul comportamento della memorizzazione nella cache, esplorare altre funzionalità di memorizzazione nella cache di ASP.NET Core. Vedere gli argomenti seguenti:

* <xref:performance/caching/memory>
* <xref:performance/caching/distributed>
* <xref:mvc/views/tag-helpers/builtin-th/cache-tag-helper>
* <xref:mvc/views/tag-helpers/builtin-th/distributed-cache-tag-helper>

## <a name="troubleshooting"></a>Risoluzione dei problemi

Se il comportamento di memorizzazione nella cache non è quello previsto, verificare che le risposte siano memorizzabili nella cache e in grado di essere servite dalla cache. Esaminare le intestazioni in ingresso della richiesta e le intestazioni in uscita della risposta. Abilitare la [registrazione](xref:fundamentals/logging/index) per facilitare il debug.

Durante il test e la risoluzione dei problemi relativi al comportamento di Caching, un browser può impostare intestazioni di richiesta che influiscono sulla memorizzazione nella cache in modi indesiderati Ad esempio, un browser può impostare l' `Cache-Control` intestazione su `no-cache` o `max-age=0` durante l'aggiornamento di una pagina. Gli strumenti seguenti possono impostare in modo esplicito le intestazioni delle richieste e sono preferiti per il test della memorizzazione nella cache:

* [Fiddler](https://www.telerik.com/fiddler)
* [Postman](https://www.getpostman.com/)

### <a name="conditions-for-caching"></a>Condizioni per la memorizzazione nella cache

* La richiesta deve avere come risultato una risposta del server con un codice di stato 200 (OK).
* Il metodo di richiesta deve essere GET o HEAD.
* In `Startup.Configure` il middleware di memorizzazione nella cache delle risposte deve essere inserito prima del middleware che richiede la memorizzazione nella cache. Per altre informazioni, vedere <xref:fundamentals/middleware/index>.
* L' `Authorization` intestazione non deve essere presente.
* `Cache-Control` i parametri di intestazione devono essere validi e la risposta deve essere contrassegnata `public` e non contrassegnata `private` .
* L'intestazione `Pragma: no-cache` non deve essere presente se l' `Cache-Control` intestazione non è presente, in quanto l'intestazione `Cache-Control` sostituisce l'intestazione `Pragma` quando è presente.
* L' `Set-Cookie` intestazione non deve essere presente.
* `Vary` i parametri di intestazione devono essere validi e non uguali a `*` .
* Il `Content-Length` valore dell'intestazione (se impostato) deve corrispondere alla dimensione del corpo della risposta.
* <xref:Microsoft.AspNetCore.Http.Features.IHttpSendFileFeature>Non viene utilizzato.
* La risposta non deve essere obsoleta come specificato dall' `Expires` intestazione e dalle `max-age` `s-maxage` direttive della cache e.
* Il buffer delle risposte deve avere esito positivo. La dimensione della risposta deve essere minore di quella configurata o predefinita <xref:Microsoft.AspNetCore.ResponseCaching.ResponseCachingOptions.SizeLimit> . Le dimensioni del corpo della risposta devono essere inferiori a quelle configurate o predefinite <xref:Microsoft.AspNetCore.ResponseCaching.ResponseCachingOptions.MaximumBodySize> .
* La risposta deve essere memorizzata nella cache in base alle specifiche [RFC 7234](https://tools.ietf.org/html/rfc7234) . La direttiva, ad esempio, `no-store` non deve esistere nei campi di intestazione della richiesta o della risposta. Per informazioni dettagliate, vedere la *sezione 3: archiviazione delle risposte nelle cache* di [RFC 7234](https://tools.ietf.org/html/rfc7234) .

> [!NOTE]
> Il sistema antifalsificazione per la generazione di token protetti per evitare attacchi di richiesta intersito falsa (CSRF) imposta le `Cache-Control` `Pragma` intestazioni e su in `no-cache` modo che le risposte non vengano memorizzate nella cache. Per informazioni su come disabilitare i token antifalsificazione per gli elementi del form HTML, vedere <xref:security/anti-request-forgery#aspnet-core-antiforgery-configuration> .

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:fundamentals/startup>
* <xref:fundamentals/middleware/index>
* <xref:performance/caching/memory>
* <xref:performance/caching/distributed>
* <xref:fundamentals/change-tokens>
* <xref:performance/caching/response>
* <xref:mvc/views/tag-helpers/builtin-th/cache-tag-helper>
* <xref:mvc/views/tag-helpers/builtin-th/distributed-cache-tag-helper>

::: moniker-end