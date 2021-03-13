---
title: Abilitare le richieste tra le origini (CORS) in ASP.NET Core
author: rick-anderson
description: Informazioni su come CORS come standard per consentire o rifiutare le richieste tra origini in un'app ASP.NET Core.
ms.author: riande
ms.custom: mvc
ms.date: 04/17/2020
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
uid: security/cors
ms.openlocfilehash: b057e5e08b8a4d0f9bcd68f92102cad309655acc
ms.sourcegitcommit: 07e7ee573fe4e12be93249a385db745d714ff6ae
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/12/2021
ms.locfileid: "103413509"
---
# <a name="enable-cross-origin-requests-cors-in-aspnet-core"></a>Abilitare le richieste tra le origini (CORS) in ASP.NET Core

::: moniker range=">= aspnetcore-3.0"

Di [Rick Anderson](https://twitter.com/RickAndMSFT) e [Kirk Larkin](https://twitter.com/serpent5)

Questo articolo illustra come abilitare CORS in un'app ASP.NET Core.

La sicurezza del browser impedisce a una pagina Web di eseguire richieste a un dominio diverso da quello che ha gestito la pagina Web. Questa restrizione è detta *criterio della stessa origine*. Il criterio della stessa origine impedisce a un sito dannoso di leggere dati sensibili da un altro sito. In alcuni casi potrebbe essere necessario consentire ad altri siti di effettuare richieste tra le origini per l'app. Per altre informazioni, vedere l' [articolo relativo a Mozilla CORS](https://developer.mozilla.org/docs/Web/HTTP/CORS).

[Condivisione risorse tra le origini](https://www.w3.org/TR/cors/) (CORS):

* È uno standard W3C che consente a un server di ridurre i criteri della stessa origine.
* **Non** è una funzionalità di sicurezza, CORS rilassa la sicurezza. Un'API non è più sicura consentendo CORS. Per ulteriori informazioni, vedere [funzionamento di CORS](#how-cors).
* Consente a un server di consentire in modo esplicito alcune richieste tra origini rifiutando altre.
* È più sicuro e flessibile rispetto alle tecniche precedenti, ad esempio [JSONP](/dotnet/framework/wcf/samples/jsonp).

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/cors/3.1sample/Cors/WebAPI) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="same-origin"></a>Stessa origine

Due URL hanno la stessa origine se hanno schemi, host e porte identici ([RFC 6454](https://tools.ietf.org/html/rfc6454)).

Questi due URL hanno la stessa origine:

* `https://example.com/foo.html`
* `https://example.com/bar.html`

Questi URL hanno origini diverse da quelle dei due URL precedenti:

* `https://example.net`: Dominio diverso
* `https://www.example.com/foo.html`: Sottodominio diverso
* `http://example.com/foo.html`: Schema diverso
* `https://example.com:9000/foo.html`: Porta diversa

## <a name="enable-cors"></a>Abilitare CORS

Sono disponibili tre modi per abilitare CORS:

* Nel middleware usando un criterio [denominato](#np) o un [criterio predefinito](#dp).
* Uso del [routing degli endpoint](#ecors).
* Con l'attributo [[EnableCors]](#attr) .

L'utilizzo dell'attributo [[EnableCors]](#attr) con un criterio denominato fornisce il controllo più raffinato per limitare gli endpoint che supportano CORS.

> [!WARNING]
> <xref:Microsoft.AspNetCore.Builder.CorsMiddlewareExtensions.UseCors%2A> deve essere chiamato nell'ordine corretto. Per ulteriori informazioni, vedere l' [ordine del middleware](xref:fundamentals/middleware/index#middleware-order). Ad esempio, `UseCors` deve essere chiamato prima di <xref:Microsoft.AspNetCore.Builder.ResponseCachingExtensions.UseResponseCaching%2A> quando si usa `UseResponseCaching` .

Ogni approccio è descritto in dettaglio nelle sezioni seguenti.

<a name="np"></a>

## <a name="cors-with-named-policy-and-middleware"></a>CORS con criteri e middleware denominati

Il middleware CORS gestisce le richieste tra le origini. Il codice seguente applica un criterio CORS a tutti gli endpoint dell'app con le origini specificate:

[!code-csharp[](cors/3.1sample/Cors/WebAPI/Startup.cs?name=snippet&highlight=3,9,32)]

Il codice precedente:

* Imposta il nome dei criteri su `_myAllowSpecificOrigins` . Il nome del criterio è arbitrario.
* Chiama il <xref:Microsoft.AspNetCore.Builder.CorsMiddlewareExtensions.UseCors*> metodo di estensione e specifica il  `_myAllowSpecificOrigins` criterio CORS. `UseCors` aggiunge il middleware CORS. La chiamata a `UseCors` deve essere posizionata dopo `UseRouting` , ma prima di `UseAuthorization` . Per ulteriori informazioni, vedere l' [ordine del middleware](xref:fundamentals/middleware/index#middleware-order).
* Chiama <xref:Microsoft.Extensions.DependencyInjection.CorsServiceCollectionExtensions.AddCors*> con un' [espressione lambda](/dotnet/csharp/programming-guide/statements-expressions-operators/lambda-expressions). L'espressione lambda accetta un <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder> oggetto. Le [Opzioni di configurazione](#cors-policy-options), ad esempio `WithOrigins` , sono descritte più avanti in questo articolo.
* Abilita il `_myAllowSpecificOrigins` criterio CORS per tutti gli endpoint del controller. Vedere [endpoint routing](#ecors) per applicare un criterio CORS a endpoint specifici.
* Quando si usa il [middleware di memorizzazione nella cache delle risposte](xref:performance/caching/middleware), chiamare <xref:Microsoft.AspNetCore.Builder.CorsMiddlewareExtensions.UseCors%2A> prima <xref:Microsoft.AspNetCore.Builder.ResponseCachingExtensions.UseResponseCaching%2A> .

Con il routing dell'endpoint è **necessario** configurare il middleware CORS per l'esecuzione tra le chiamate a `UseRouting` e `UseEndpoints` .

Vedere [test CORS](#testc) per istruzioni sul test di codice simile al codice precedente.

La <xref:Microsoft.Extensions.DependencyInjection.MvcCorsMvcCoreBuilderExtensions.AddCors*> chiamata al metodo aggiunge i servizi CORS al contenitore del servizio dell'app:

[!code-csharp[](cors/3.1sample/Cors/WebAPI/Startup.cs?name=snippet2)]

Per ulteriori informazioni, vedere [Opzioni dei criteri CORS](#cpo) in questo documento.

I <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder> metodi possono essere concatenati, come illustrato nel codice seguente:

[!code-csharp[](cors/3.1sample/Cors/WebAPI/Startup2.cs?name=snippet)]

Nota: l'URL specificato **non** deve contenere una barra finale ( `/` ). Se l'URL termina con `/` , il confronto restituisce `false` e non viene restituita alcuna intestazione.

<a name="dp"></a>

### <a name="cors-with-default-policy-and-middleware"></a>CORS con criteri e middleware predefiniti

Il codice evidenziato seguente Abilita i criteri predefiniti di CORS:

[!code-csharp[](cors/3.1sample/Cors/WebAPI/StartupDefaultPolicy.cs?name=snippet2&highlight=7,29)]

Il codice precedente applica i criteri predefiniti di CORS a tutti gli endpoint del controller.

<a name="ecors"></a>

## <a name="enable-cors-with-endpoint-routing"></a>Abilitare cors con routing degli endpoint

L'abilitazione di CORS in base all'endpoint `RequireCors` attualmente in **uso non** supporta [le richieste preliminari automatiche](#apf). Per ulteriori informazioni, vedere [questo problema di GitHub](https://github.com/dotnet/aspnetcore/issues/20709) e [testare cors con endpoint routing e [HttpOptions]](#tcer).

Con il routing degli endpoint, è possibile abilitare CORS in base all'endpoint usando il <xref:Microsoft.AspNetCore.Builder.CorsEndpointConventionBuilderExtensions.RequireCors*> set di metodi di estensione:

[!code-csharp[](cors/3.1sample/Cors/WebAPI/StartupEndPt.cs?name=snippet2&highlight=3,7-15,32,40,43)]

Nel codice precedente:

* `app.UseCors` Abilita il middleware CORS. Poiché un criterio predefinito non è stato configurato, `app.UseCors()` da solo non Abilita CORS.
* Gli `/echo` endpoint del controller e consentono richieste tra le origini usando i criteri specificati.
* Gli `/echo2` Razor endpoint delle pagine e non  consentono richieste tra le origini perché non è stato specificato alcun criterio predefinito.

L'attributo [[DisableCors]](#dc) non **Disabilita CORS**  che è stato abilitato dal routing di endpoint con `RequireCors` .

Per istruzioni sul test del codice simile a quello precedente, vedere [test CORS with endpoint routing e [HttpOptions]](#tcer) .

<a name="attr"></a>

## <a name="enable-cors-with-attributes"></a>Abilita CORS con attributi

L'abilitazione di CORS con l'attributo [[EnableCors]](xref:Microsoft.AspNetCore.Cors.EnableCorsAttribute) e l'applicazione di un criterio denominato solo agli endpoint che richiedono CORS fornisce il controllo migliore.

L'attributo [[EnableCors]](xref:Microsoft.AspNetCore.Cors.EnableCorsAttribute) rappresenta un'alternativa all'applicazione di CORS a livello globale. L' `[EnableCors]` attributo Abilita CORS per gli endpoint selezionati, anziché tutti gli endpoint:

* `[EnableCors]` Specifica i criteri predefiniti.
* `[EnableCors("{Policy String}")]` Specifica un criterio denominato.

L' `[EnableCors]` attributo può essere applicato a:

* Razor Pagina `PageModel`
* Controller
* Metodo di azione del controller

È possibile applicare criteri diversi ai controller, ai modelli di pagina o ai metodi di azione con l' `[EnableCors]` attributo. Quando l' `[EnableCors]` attributo viene applicato a un controller, a un modello di pagina o a un metodo di azione e CORS è abilitato nel middleware, vengono applicati **entrambi** i criteri. **È consigliabile evitare di combinare i criteri. Usare l'** `[EnableCors]` **attributo o il middleware, non entrambi nella stessa app.**

Il codice seguente applica un criterio diverso a ogni metodo:

[!code-csharp[](cors/3.1sample/Cors/WebAPI/Controllers/WidgetController.cs?name=snippet&highlight=6,14)]

Il codice seguente crea due criteri CORS:

[!code-csharp[](cors/3.1sample/Cors/WebAPI/Startup3.cs?name=snippet&highlight=12-28,44)]

Per il controllo più raffinato della limitazione delle richieste CORS:

* Usare `[EnableCors("MyPolicy")]` con un criterio denominato.
* Non definire un criterio predefinito.
* Non usare il [routing degli endpoint](#ecors).

Il codice nella sezione successiva soddisfa l'elenco precedente.

Vedere [test CORS](#testc) per istruzioni sul test di codice simile al codice precedente.

<a name="dc"></a>

### <a name="disable-cors"></a>Disabilitare CORS

L'attributo [[DisableCors]](xref:Microsoft.AspNetCore.Cors.DisableCorsAttribute) non **Disabilita CORS**  che è stato abilitato dal routing dell' [endpoint](#ecors).

Il codice seguente definisce i criteri CORS `"MyPolicy"` :

[!code-csharp[](cors/3.1sample/Cors/WebAPI/StartupTestMyPolicy.cs?name=snippet)]

Il codice seguente Disabilita CORS per l' `GetValues2` azione:

[!code-csharp[](cors/3.1sample/Cors/WebAPI/Controllers/ValuesController.cs?name=snippet&highlight=1,23)]

Il codice precedente:

* Non Abilita CORS con [routing endpoint](#ecors).
* Non definisce un [criterio CORS predefinito](#dp).
* USA [[EnableCors ("criteri")]](#attr) per abilitare il `"MyPolicy"` criterio CORS per il controller.
* Disabilita CORS per il `GetValues2` metodo.

Vedere [test CORS](#testc) per istruzioni sul test del codice precedente.

<a name="cpo"></a>

## <a name="cors-policy-options"></a>Opzioni dei criteri di CORS

In questa sezione vengono descritte le varie opzioni che è possibile impostare in un criterio CORS:

* [Imposta le origini consentite](#set-the-allowed-origins)
* [Impostare i metodi HTTP consentiti](#set-the-allowed-http-methods)
* [Impostare le intestazioni della richiesta consentita](#set-the-allowed-request-headers)
* [Impostare le intestazioni di risposta esposte](#set-the-exposed-response-headers)
* [Credenziali nelle richieste tra le origini](#credentials-in-cross-origin-requests)
* [Imposta la data di scadenza preliminare](#set-the-preflight-expiration-time)

<xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsOptions.AddPolicy*> viene chiamato in `Startup.ConfigureServices` . Per alcune opzioni, può essere utile leggere prima la sezione [come funziona CORS](#how-cors) .

## <a name="set-the-allowed-origins"></a>Imposta le origini consentite

<xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.AllowAnyOrigin*>: Consente le richieste CORS da tutte le origini con qualsiasi schema ( `http` o `https` ). `AllowAnyOrigin` non è sicuro perché *qualsiasi sito Web* può effettuare richieste tra origini per l'app.

> [!NOTE]
> La definizione di `AllowAnyOrigin` e `AllowCredentials` è una configurazione non sicura e può comportare la falsificazione della richiesta tra siti. Il servizio CORS restituisce una risposta CORS non valida quando un'app è configurata con entrambi i metodi.

`AllowAnyOrigin` influiscono sulle richieste preliminari e sull' `Access-Control-Allow-Origin` intestazione. Per ulteriori informazioni, vedere la sezione [preflight requests](#preflight-requests) .

<xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.SetIsOriginAllowedToAllowWildcardSubdomains*>: Imposta la <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicy.IsOriginAllowed*> proprietà dei criteri in modo che sia una funzione che consente alle origini di corrispondere a un dominio con caratteri jolly configurato durante la valutazione se l'origine è consentita.

[!code-csharp[](cors/3.1sample/Cors/WebAPI/StartupAllowSubdomain.cs?name=snippet)]

### <a name="set-the-allowed-http-methods"></a>Impostare i metodi HTTP consentiti

<xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.AllowAnyMethod*>:

* Consente qualsiasi metodo HTTP:
* Influiscono sulle richieste preliminari e sull' `Access-Control-Allow-Methods` intestazione. Per ulteriori informazioni, vedere la sezione [preflight requests](#preflight-requests) .

### <a name="set-the-allowed-request-headers"></a>Impostare le intestazioni della richiesta consentita

Per consentire l'invio di intestazioni specifiche in una richiesta CORS, denominate [intestazioni richiesta autore](https://xhr.spec.whatwg.org/#request), chiamare <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.WithHeaders*> e specificare le intestazioni consentite:

[!code-csharp[](cors/3.1sample/Cors/WebAPI/StartupAllowSubdomain.cs?name=snippet2)]

Per consentire tutte le [intestazioni di richiesta di autore](https://www.w3.org/TR/cors/#author-request-headers), chiamare <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.AllowAnyHeader*> :

[!code-csharp[](cors/3.1sample/Cors/WebAPI/StartupAllowSubdomain.cs?name=snippet3)]

`AllowAnyHeader` influiscono sulle richieste preliminari e sull'intestazione [Access-Control-Request-Headers](https://developer.mozilla.org/docs/Web/HTTP/Headers/Access-Control-Request-Method) . Per ulteriori informazioni, vedere la sezione [preflight requests](#preflight-requests) .

Un criterio middleware CORS corrisponde a intestazioni specifiche specificate da `WithHeaders` è possibile solo quando le intestazioni inviate `Access-Control-Request-Headers` corrispondono esattamente alle intestazioni indicate in `WithHeaders` .

Si consideri, ad esempio, un'app configurata come segue:

[!code-csharp[](cors/3.1sample/Cors/WebAPI/StartupAllowSubdomain.cs?name=snippet4)]

Il middleware CORS rifiuta una richiesta preliminare con la seguente intestazione di richiesta perché `Content-Language` ([HeaderNames. ContentLanguage](xref:Microsoft.Net.Http.Headers.HeaderNames.ContentLanguage)) non è elencato in `WithHeaders` :

```
Access-Control-Request-Headers: Cache-Control, Content-Language
```

L'app restituisce una risposta *200 OK* , ma non invia le intestazioni CORS. Pertanto, il browser non tenta di eseguire la richiesta tra origini.

### <a name="set-the-exposed-response-headers"></a>Impostare le intestazioni di risposta esposte

Per impostazione predefinita, il browser non espone tutte le intestazioni di risposta all'app. Per altre informazioni, vedere la pagina relativa alla [condivisione delle risorse tra le origini W3C (terminologia): intestazione della risposta semplice](https://www.w3.org/TR/cors/#simple-response-header).

Le intestazioni di risposta disponibili per impostazione predefinita sono:

* `Cache-Control`
* `Content-Language`
* `Content-Type`
* `Expires`
* `Last-Modified`
* `Pragma`

La specifica CORS chiama queste intestazioni di *risposta semplici*. Per rendere disponibili altre intestazioni per l'app, chiamare <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.WithExposedHeaders*> :

[!code-csharp[](cors/3.1sample/Cors/WebAPI/StartupAllowSubdomain.cs?name=snippet5)]
### <a name="credentials-in-cross-origin-requests"></a>Credenziali nelle richieste tra le origini

Le credenziali richiedono una gestione speciale in una richiesta CORS. Per impostazione predefinita, il browser non invia le credenziali con una richiesta tra le origini. Le credenziali includono cookie s e gli schemi di autenticazione HTTP. Per inviare le credenziali con una richiesta tra le origini, il client deve impostare `XMLHttpRequest.withCredentials` su `true` .

Utilizzando `XMLHttpRequest` direttamente:

```javascript
var xhr = new XMLHttpRequest();
xhr.open('get', 'https://www.example.com/api/test');
xhr.withCredentials = true;
```

Uso di jQuery:

```javascript
$.ajax({
  type: 'get',
  url: 'https://www.example.com/api/test',
  xhrFields: {
    withCredentials: true
  }
});
```

Uso dell' [API fetch](https://developer.mozilla.org/docs/Web/API/Fetch_API):

```javascript
fetch('https://www.example.com/api/test', {
    credentials: 'include'
});
```

Il server deve consentire le credenziali. Per consentire le credenziali tra le origini, chiamare <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.AllowCredentials*> :

[!code-csharp[](cors/3.1sample/Cors/WebAPI/StartupAllowSubdomain.cs?name=snippet6)]

La risposta HTTP include un' `Access-Control-Allow-Credentials` intestazione che indica al browser che il server consente le credenziali per una richiesta tra le origini.

Se il browser invia le credenziali, ma la risposta non include un' `Access-Control-Allow-Credentials` intestazione valida, il browser non espone la risposta all'app e la richiesta tra le origini ha esito negativo.

Consentire le credenziali tra le origini costituisce un rischio per la sicurezza. Un sito Web in un altro dominio può inviare le credenziali dell'utente che ha eseguito l'accesso all'app per conto dell'utente senza la conoscenza dell'utente. <!-- TODO Review: When using `AllowCredentials`, all CORS enabled domains must be trusted.
I don't like "all CORS enabled domains must be trusted", because it implies that if you're not using  `AllowCredentials`, domains don't need to be trusted. -->

La specifica CORS indica inoltre che l'impostazione delle origini su `"*"` (tutte le origini) non è valida se l' `Access-Control-Allow-Credentials` intestazione è presente.

<a name="pref"></a>

## <a name="preflight-requests"></a>Richieste preliminari

Per alcune richieste CORS, il browser invia una richiesta di [Opzioni](https://developer.mozilla.org/docs/Web/HTTP/Methods/OPTIONS) aggiuntive prima di effettuare la richiesta effettiva. Questa richiesta è detta [richiesta preliminare](https://developer.mozilla.org/docs/Glossary/Preflight_request). Il browser può ignorare la richiesta preliminare se si verificano **tutte** le condizioni seguenti:

* Il metodo di richiesta è GET, HEAD o POST.
* L'app non imposta intestazioni di richiesta diverse da `Accept` , `Accept-Language` , `Content-Language` , `Content-Type` o `Last-Event-ID` .
* L' `Content-Type` intestazione, se impostata, presenta uno dei valori seguenti:
  * `application/x-www-form-urlencoded`
  * `multipart/form-data`
  * `text/plain`

La regola sulle intestazioni di richiesta impostate per la richiesta del client si applica alle intestazioni impostate dall'app chiamando `setRequestHeader` sull' `XMLHttpRequest` oggetto. La specifica CORS chiama queste intestazioni di [richiesta autore](https://www.w3.org/TR/cors/#author-request-headers)intestazioni. La regola non si applica alle intestazioni che il browser può impostare, ad esempio `User-Agent` , `Host` o `Content-Length` .

Di seguito è riportato un esempio di risposta simile alla richiesta preliminare eseguita dal pulsante **[Put test]** nella sezione [test CORS](#testc) di questo documento.

```
General:
Request URL: https://cors3.azurewebsites.net/api/values/5
Request Method: OPTIONS
Status Code: 204 No Content

Response Headers:
Access-Control-Allow-Methods: PUT,DELETE,GET
Access-Control-Allow-Origin: https://cors1.azurewebsites.net
Server: Microsoft-IIS/10.0
Set-Cookie: ARRAffinity=8f8...8;Path=/;HttpOnly;Domain=cors1.azurewebsites.net
Vary: Origin

Request Headers:
Accept: */*
Accept-Encoding: gzip, deflate, br
Accept-Language: en-US,en;q=0.9
Access-Control-Request-Method: PUT
Connection: keep-alive
Host: cors3.azurewebsites.net
Origin: https://cors1.azurewebsites.net
Referer: https://cors1.azurewebsites.net/
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: cross-site
User-Agent: Mozilla/5.0
```

La richiesta preliminare usa il metodo delle [Opzioni http](https://developer.mozilla.org/docs/Web/HTTP/Methods/OPTIONS) . Può includere le intestazioni seguenti:

* [Access-Control-request-method](https://developer.mozilla.org/docs/Web/HTTP/Headers/Access-Control-Request-Method): il metodo HTTP che verrà usato per la richiesta effettiva.
* [Access-Control-Request-Headers](https://developer.mozilla.org/docs/Web/HTTP/Headers/Access-Control-Allow-Headers): elenco di intestazioni di richiesta impostate dall'app sulla richiesta effettiva. Come indicato in precedenza, non include le intestazioni impostate dal browser, ad esempio `User-Agent` .
* [Access-Control-Allow-Methods](https://developer.mozilla.org/docs/Web/HTTP/Headers/Access-Control-Allow-Methods)

Se la richiesta preliminare viene negata, l'app restituisce una `200 OK` risposta ma non imposta le intestazioni CORS. Pertanto, il browser non tenta di eseguire la richiesta tra origini. Per un esempio di richiesta preliminare negata, vedere la sezione [test CORS](#testc) di questo documento.

Usando gli strumenti F12, l'app console Visualizza un errore simile a uno dei seguenti, a seconda del browser:

* Firefox: richiesta tra le origini bloccata: gli stessi criteri di origine non consentono la lettura della risorsa remota in `https://cors1.azurewebsites.net/api/TodoItems1/MyDelete2/5` . (Motivo: richiesta CORS non riuscita). [Altre informazioni](https://developer.mozilla.org/docs/Web/HTTP/CORS/Errors/CORSDidNotSucceed)
* Basata su cromo: l'accesso al recupero in ' https://cors1.azurewebsites.net/api/TodoItems1/MyDelete2/5 ' dall'origine ' https://cors3.azurewebsites.net ' è stato bloccato dal criterio CORS: la risposta alla richiesta preliminare non supera il controllo di accesso: non è presente alcuna intestazione ' Access-Control-Allow-Origin ' nella risorsa richiesta. Se si ritiene soddisfacente una risposta opaca, impostare la modalità della richiesta 'no-cors' per recuperare la risorsa con CORS disabilitato.

Per consentire intestazioni specifiche, chiamare <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.WithHeaders*> :

[!code-csharp[](cors/3.1sample/Cors/WebAPI/StartupAllowSubdomain.cs?name=snippet2)]

Per consentire tutte le [intestazioni di richiesta di autore](https://www.w3.org/TR/cors/#author-request-headers), chiamare <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.AllowAnyHeader*> :

[!code-csharp[](cors/3.1sample/Cors/WebAPI/StartupAllowSubdomain.cs?name=snippet3)]

I browser non sono coerenti con la modalità di impostazione `Access-Control-Request-Headers` . Se uno:

* Le intestazioni sono impostate su un valore diverso da `"*"`
* <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicy.AllowAnyHeader*> viene chiamato: includere almeno `Accept` , `Content-Type` e `Origin` , più eventuali intestazioni personalizzate che si desidera supportare.

<a name="apf"></a>

### <a name="automatic-preflight-request-code"></a>Codice richiesta preliminare automatica

Quando viene applicato il criterio CORS:

* A livello globale, chiamando il `app.UseCors` `Startup.Configure` .
* Utilizzando l' `[EnableCors]` attributo.

ASP.NET Core risponde alla richiesta di opzioni preliminari.

L'abilitazione di CORS in base all'endpoint `RequireCors` attualmente in **uso non** supporta le richieste preliminari automatiche.

Questo comportamento viene illustrato nella sezione [test CORS](#testc) di questo documento.

<a name="pro"></a>

### <a name="httpoptions-attribute-for-preflight-requests"></a>Attributo [HttpOptions] per le richieste preliminari

Quando CORS è abilitato con i criteri appropriati, ASP.NET Core risponde in genere alle richieste preliminari CORS automaticamente. In alcuni scenari questo potrebbe non essere il caso. Ad esempio, usando [cors con il routing degli endpoint](#ecors).

Il codice seguente usa l'attributo [[HttpOptions]](xref:Microsoft.AspNetCore.Mvc.HttpOptionsAttribute) per creare endpoint per le richieste di opzioni:

[!code-csharp[](cors/3.1sample/Cors/WebAPI/Controllers/TodoItems2Controller.cs?name=snippet&highlight=5-17)]

Per istruzioni sul test del codice precedente [, vedere test CORS with endpoint routing e [HttpOptions]](#tcer) .

### <a name="set-the-preflight-expiration-time"></a>Imposta la data di scadenza preliminare

L' `Access-Control-Max-Age` intestazione specifica per quanto tempo la risposta alla richiesta preliminare può essere memorizzata nella cache. Per impostare questa intestazione, chiamare <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.SetPreflightMaxAge*> :

[!code-csharp[](cors/3.1sample/Cors/WebAPI/StartupAllowSubdomain.cs?name=snippet7)]
<a name="how-cors"></a>

## <a name="how-cors-works"></a>Funzionamento di CORS

In questa sezione viene descritto cosa accade in una richiesta [CORS](https://developer.mozilla.org/docs/Web/HTTP/CORS) al livello dei messaggi HTTP.

* CORS **non** è una funzionalità di sicurezza. CORS è uno standard W3C che consente a un server di ridurre i criteri della stessa origine.
  * Ad esempio, un attore malintenzionato potrebbe usare [lo scripting tra siti (XSS)](xref:security/cross-site-scripting) per il sito ed eseguire una richiesta tra siti al sito abilitato per CORS per rubare le informazioni.
* Un'API non è più sicura consentendo CORS.
  * Il client (browser) deve applicare CORS. Il server esegue la richiesta e restituisce la risposta, ovvero il client che restituisce un errore e blocca la risposta. Uno degli strumenti seguenti, ad esempio, visualizzerà la risposta del server:
    * [Fiddler](https://www.telerik.com/fiddler)
    * [Postman](https://www.getpostman.com/)
    * [HttpClient .NET](/dotnet/csharp/tutorials/console-webapiclient)
    * Un Web browser immettendo l'URL nella barra degli indirizzi.
* Si tratta di un modo per consentire a un server di consentire ai browser di eseguire una richiesta dell' [API di recupero](https://developer.mozilla.org/docs/Web/API/Fetch_API) o di [XHR](https://developer.mozilla.org/docs/Web/API/XMLHttpRequest) tra le origini, che altrimenti sarebbe proibita.
  * I browser senza CORS non possono eseguire richieste tra le origini. Prima di CORS, [JSONP](https://www.w3schools.com/js/js_json_jsonp.asp) è stato usato per aggirare questa restrizione. JSONP non usa XHR, usa il `<script>` tag per ricevere la risposta. Gli script possono essere caricati tra le origini.

La [specifica CORS](https://www.w3.org/TR/cors/) ha introdotto diverse nuove intestazioni HTTP che consentono richieste tra le origini. Se un browser supporta CORS, queste intestazioni vengono impostate automaticamente per le richieste tra le origini. Il codice JavaScript personalizzato non è necessario per abilitare CORS.

[Pulsante Put test](https://cors3.azurewebsites.net/test) nell' [esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/cors/3.1sample/Cors/WebAPI) distribuito

Di seguito è riportato un esempio di richiesta tra più origini dal pulsante test [valori](https://cors3.azurewebsites.net/) a `https://cors1.azurewebsites.net/api/values` . `Origin`Intestazione:

* Fornisce il dominio del sito che sta effettuando la richiesta.
* È obbligatorio e deve essere diverso dall'host.

**Intestazioni generali**

```
Request URL: https://cors1.azurewebsites.net/api/values
Request Method: GET
Status Code: 200 OK
```

**Intestazioni della risposta**

```
Content-Encoding: gzip
Content-Type: text/plain; charset=utf-8
Server: Microsoft-IIS/10.0
Set-Cookie: ARRAffinity=8f...;Path=/;HttpOnly;Domain=cors1.azurewebsites.net
Transfer-Encoding: chunked
Vary: Accept-Encoding
X-Powered-By: ASP.NET
```

**Intestazioni delle richieste**

```
Accept: */*
Accept-Encoding: gzip, deflate, br
Accept-Language: en-US,en;q=0.9
Connection: keep-alive
Host: cors1.azurewebsites.net
Origin: https://cors3.azurewebsites.net
Referer: https://cors3.azurewebsites.net/
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: cross-site
User-Agent: Mozilla/5.0 ...
```

Nelle `OPTIONS` richieste, il server imposta l'intestazione delle **intestazioni di risposta** `Access-Control-Allow-Origin: {allowed origin}` nella risposta. Ad esempio, la richiesta del pulsante [Delete [EnableCors]](https://cors1.azurewebsites.net/test?number=2) di [esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/cors/3.1sample/Cors/WebAPI)distribuito `OPTIONS` contiene le intestazioni seguenti:

**Intestazioni generali**

```
Request URL: https://cors3.azurewebsites.net/api/TodoItems2/MyDelete2/5
Request Method: OPTIONS
Status Code: 204 No Content
```

**Intestazioni della risposta**

```
Access-Control-Allow-Headers: Content-Type,x-custom-header
Access-Control-Allow-Methods: PUT,DELETE,GET,OPTIONS
Access-Control-Allow-Origin: https://cors1.azurewebsites.net
Server: Microsoft-IIS/10.0
Set-Cookie: ARRAffinity=8f...;Path=/;HttpOnly;Domain=cors3.azurewebsites.net
Vary: Origin
X-Powered-By: ASP.NET
```

**Intestazioni delle richieste**

```
Accept: */*
Accept-Encoding: gzip, deflate, br
Accept-Language: en-US,en;q=0.9
Access-Control-Request-Headers: content-type
Access-Control-Request-Method: DELETE
Connection: keep-alive
Host: cors3.azurewebsites.net
Origin: https://cors1.azurewebsites.net
Referer: https://cors1.azurewebsites.net/test?number=2
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: cross-site
User-Agent: Mozilla/5.0
```

Nelle intestazioni di **risposta** precedenti, il server imposta l'intestazione [Access-Control-Allow-Origin](https://developer.mozilla.org/docs/Web/HTTP/Headers/Access-Control-Allow-Origin) nella risposta. Il `https://cors1.azurewebsites.net` valore di questa intestazione corrisponde all' `Origin` intestazione della richiesta.

Se <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.AllowAnyOrigin*> viene chiamato, `Access-Control-Allow-Origin: *` viene restituito il valore del carattere jolly. `AllowAnyOrigin` consente qualsiasi origine.

Se la risposta non include l' `Access-Control-Allow-Origin` intestazione, la richiesta tra le origini ha esito negativo. In particolare, il browser non consente la richiesta. Anche se il server restituisce una risposta con esito positivo, il browser non rende disponibile la risposta all'app client.

<a name="options"></a>

### <a name="display-options-requests"></a>Richieste di opzioni di visualizzazione

Per impostazione predefinita, i browser Chrome e Edge non visualizzano le richieste OPTIONS nella scheda rete degli strumenti F12. Per visualizzare le richieste di opzioni in questi browser:

* `chrome://flags/#out-of-blink-cors` o `edge://flags/#out-of-blink-cors`
* disabilitare il flag.
* Riavvia.

Per impostazione predefinita, Firefox Visualizza le richieste OPTIONS.

## <a name="cors-in-iis"></a>CORS in IIS

Quando si esegue la distribuzione in IIS, CORS deve essere eseguito prima dell'autenticazione di Windows se il server non è configurato per consentire l'accesso anonimo. Per supportare questo scenario, è necessario installare e configurare il [modulo cors di IIS](https://www.iis.net/downloads/microsoft/iis-cors-module) per l'app.

<a name="testc"></a>

## <a name="test-cors"></a>Testare CORS

Il [download di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/cors/3.1sample/Cors/WebAPI) contiene codice per testare CORS. Vedere [come scaricare un esempio](xref:index#how-to-download-a-sample). L'esempio è un progetto API con Razor pagine aggiunte:

[!code-csharp[](cors/3.1sample/Cors/WebAPI/StartupTest2.cs?name=snippet2)]

  > [!WARNING]
  > `WithOrigins("https://localhost:<port>");` deve essere usato solo per il test di un'app di esempio simile al [codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/live/aspnetcore/security/cors/3.1sample/Cors)per il download.

Di seguito sono riportati `ValuesController` gli endpoint per i test:

[!code-csharp[](cors/3.1sample/Cors/WebAPI/Controllers/ValuesController.cs?name=snippet)]

[MyDisplayRouteInfo](https://github.com/Rick-Anderson/RouteInfo/blob/master/Microsoft.Docs.Samples.RouteInfo/ControllerContextExtensions.cs) viene fornito dal pacchetto NuGet [Rick.Docs. Samples. RouteInfo](https://www.nuget.org/packages/Rick.Docs.Samples.RouteInfo) e visualizza le informazioni sulla route.

Testare il codice di esempio precedente usando uno degli approcci seguenti:

* Usare l'app di esempio distribuita all'indirizzo [https://cors3.azurewebsites.net/](https://cors3.azurewebsites.net/) . Non è necessario scaricare l'esempio.
* Eseguire l'esempio con `dotnet run` utilizzando l'URL predefinito `https://localhost:5001` .
* Eseguire l'esempio da Visual Studio con la porta impostata su 44398 per un URL di `https://localhost:44398` .

Uso di un browser con gli strumenti F12:

* Selezionare il pulsante **valori** ed esaminare le intestazioni nella scheda **rete** .
* Selezionare il pulsante **put test (Put test** ). Per istruzioni sulla visualizzazione della richiesta OPTIONS, vedere [visualizzare le richieste di opzioni](#options) . Il **test put** crea due richieste, una richiesta preliminare Options e la richiesta PUT.
* Selezionare il **`GetValues2 [DisableCors]`** pulsante per attivare una richiesta CORS non riuscita. Come indicato nel documento, la risposta restituisce 200 esito positivo, ma la richiesta CORS non viene eseguita. Selezionare la scheda **console** per visualizzare l'errore CORS. A seconda del browser, viene visualizzato un errore simile al seguente:

     L'accesso al recupero `'https://cors1.azurewebsites.net/api/values/GetValues2'` dall'origine `'https://cors3.azurewebsites.net'` è stato bloccato dal criterio CORS: non è presente alcuna intestazione ' Access-Control-Allow-Origin ' nella risorsa richiesta. Se si ritiene soddisfacente una risposta opaca, impostare la modalità della richiesta 'no-cors' per recuperare la risorsa con CORS disabilitato.
     
Gli endpoint abilitati per CORS possono essere testati con uno strumento, ad esempio [curl](https://curl.haxx.se/), [Fiddler](https://www.telerik.com/fiddler)o [postazione](https://www.getpostman.com/). Quando si utilizza uno strumento, l'origine della richiesta specificata dall' `Origin` intestazione deve essere diversa dall'host che riceve la richiesta. Se la richiesta non è *tra origini* basate sul valore dell' `Origin` intestazione:

* Non è necessario che il middleware CORS elabori la richiesta.
* Le intestazioni CORS non vengono restituite nella risposta.

Il comando seguente usa `curl` per emettere una richiesta OPTIONS con informazioni:

```bash
curl -X OPTIONS https://cors3.azurewebsites.net/api/TodoItems2/5 -i
```

<!--
curl come with Git. Add to path variable
C:\Program Files\Git\mingw64\bin\
-->

<a name="tcer"></a>

### <a name="test-cors-with-endpoint-routing-and-httpoptions"></a>Test di CORS con routing endpoint e [HttpOptions]

L'abilitazione di CORS in base all'endpoint `RequireCors` attualmente in **uso non** supporta [le richieste preliminari automatiche](#apf). Si consideri il codice seguente che usa il [routing dell'endpoint per abilitare CORS](#ecors):

[!code-csharp[](cors/3.1sample/Cors/WebAPI/StartupEndPointBugTest.cs?name=snippet2)]

Di seguito sono riportati gli `TodoItems1Controller` endpoint per i test:

[!code-csharp[](cors/3.1sample/Cors/WebAPI/Controllers/TodoItems1Controller.cs?name=snippet2)]

Testare il codice precedente dalla [pagina di test](https://cors1.azurewebsites.net/test?number=1) dell' [esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/cors/3.1sample/Cors/WebAPI)distribuito.

I pulsanti **Delete [EnableCors]** e **Get [EnableCors]** sono riusciti perché gli endpoint hanno `[EnableCors]` e rispondono alle richieste preliminari. Gli altri endpoint hanno esito negativo. Il pulsante **Get** ha esito negativo perché il [codice JavaScript](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/cors/3.1sample/Cors/WebAPI/wwwroot/js/MyJS.js) Invia:

```javascript
 headers: {
      "Content-Type": "x-custom-header"
 },
```

Di seguito `TodoItems2Controller` vengono forniti endpoint simili, ma è incluso codice esplicito per rispondere alle richieste di opzioni:

[!code-csharp[](cors/3.1sample/Cors/WebAPI/Controllers/TodoItems2Controller.cs?name=snippet2)]

Testare il codice precedente dalla [pagina di test](https://cors1.azurewebsites.net/test?number=2) dell'esempio distribuito. Nell'elenco a discesa **controller** selezionare **preliminare** e quindi **impostare controller**. Tutte le chiamate CORS agli `TodoItems2Controller` endpoint hanno esito positivo.

## <a name="additional-resources"></a>Risorse aggiuntive

* [Condivisione di risorse tra le origini (CORS)](https://developer.mozilla.org/docs/Web/HTTP/CORS)
* [Introduzione al modulo CORS di IIS](https://blogs.iis.net/iisteam/getting-started-with-the-iis-cors-module)

::: moniker-end

::: moniker range="< aspnetcore-3.0"

Autore: [Rick Anderson](https://twitter.com/RickAndMSFT)

Questo articolo illustra come abilitare CORS in un'app ASP.NET Core.

La sicurezza del browser impedisce a una pagina Web di eseguire richieste a un dominio diverso da quello che ha gestito la pagina Web. Questa restrizione è detta *criterio della stessa origine*. Il criterio della stessa origine impedisce a un sito dannoso di leggere dati sensibili da un altro sito. In alcuni casi potrebbe essere necessario consentire ad altri siti di effettuare richieste tra le origini all'app. Per altre informazioni, vedere l' [articolo relativo a Mozilla CORS](https://developer.mozilla.org/docs/Web/HTTP/CORS).

[Condivisione risorse tra le origini](https://www.w3.org/TR/cors/) (CORS):

* È uno standard W3C che consente a un server di ridurre i criteri della stessa origine.
* **Non** è una funzionalità di sicurezza, CORS rilassa la sicurezza. Un'API non è più sicura consentendo CORS. Per ulteriori informazioni, vedere [funzionamento di CORS](#how-cors).
* Consente a un server di consentire in modo esplicito alcune richieste tra origini rifiutando altre.
* È più sicuro e flessibile rispetto alle tecniche precedenti, ad esempio [JSONP](/dotnet/framework/wcf/samples/jsonp).

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/cors/sample) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="same-origin"></a>Stessa origine

Due URL hanno la stessa origine se hanno schemi, host e porte identici ([RFC 6454](https://tools.ietf.org/html/rfc6454)).

Questi due URL hanno la stessa origine:

* `https://example.com/foo.html`
* `https://example.com/bar.html`

Questi URL hanno origini diverse da quelle dei due URL precedenti:

* `https://example.net`: Dominio diverso
* `https://www.example.com/foo.html`: Sottodominio diverso
* `http://example.com/foo.html`: Schema diverso
* `https://example.com:9000/foo.html`: Porta diversa

Internet Explorer non considera la porta durante il confronto delle origini.

## <a name="cors-with-named-policy-and-middleware"></a>CORS con criteri e middleware denominati

Il middleware CORS gestisce le richieste tra le origini. Il codice seguente Abilita CORS per l'intera app con l'origine specificata:

[!code-csharp[](cors/sample/Cors/WebAPI/Startup.cs?name=snippet&highlight=8,14-23,38)]

Il codice precedente:

* Imposta il nome del criterio su " \_ myAllowSpecificOrigins". Il nome del criterio è arbitrario.
* Chiama il <xref:Microsoft.AspNetCore.Builder.CorsMiddlewareExtensions.UseCors*> metodo di estensione, che Abilita CORS.
* Chiama <xref:Microsoft.Extensions.DependencyInjection.CorsServiceCollectionExtensions.AddCors*> con un' [espressione lambda](/dotnet/csharp/programming-guide/statements-expressions-operators/lambda-expressions). L'espressione lambda accetta un <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder> oggetto. Le [Opzioni di configurazione](#cors-policy-options), ad esempio `WithOrigins` , sono descritte più avanti in questo articolo.

La <xref:Microsoft.Extensions.DependencyInjection.MvcCorsMvcCoreBuilderExtensions.AddCors*> chiamata al metodo aggiunge i servizi CORS al contenitore del servizio dell'app:

[!code-csharp[](cors/sample/Cors/WebAPI/Startup.cs?name=snippet2)]

Per ulteriori informazioni, vedere [Opzioni dei criteri CORS](#cpo) in questo documento.

Il <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder> metodo può concatenare i metodi, come illustrato nel codice seguente:

[!code-csharp[](cors/sample/Cors/WebAPI/Startup2.cs?name=snippet2)]

Nota: l'URL **non** deve contenere una barra finale ( `/` ). Se l'URL termina con `/` , il confronto restituisce `false` e non viene restituita alcuna intestazione.

Il codice seguente applica i criteri di CORS a tutti gli endpoint di app tramite il middleware CORS:
```csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
    }
    else
    {
        app.UseHsts();
    }

    app.UseCors();

    app.UseHttpsRedirection();
    app.UseMvc();
}
```
Nota: `UseCors` è necessario chiamare prima `UseMvc` .

Vedere [abilitare CORS in Razor pagine, controller e metodi di azione](#ecors) per applicare i criteri di CORS a livello di pagina/controller/azione.

Vedere [test CORS](#test) per istruzioni sul test di codice simile al codice precedente.

## <a name="enable-cors-with-attributes"></a>Abilita CORS con attributi

L'attributo [ &lbrack; EnableCors &rbrack; ](xref:Microsoft.AspNetCore.Cors.EnableCorsAttribute) fornisce un'alternativa all'applicazione di CORS a livello globale. L' `[EnableCors]` attributo Abilita CORS per i punti finali selezionati, anziché tutti i punti finali.

Usare `[EnableCors]` per specificare i criteri predefiniti e `[EnableCors("{Policy String}")]` per specificare un criterio.

L' `[EnableCors]` attributo può essere applicato a:

* Razor Pagina `PageModel`
* Controller
* Metodo di azione del controller

È possibile applicare criteri diversi a controller/pagina-modello/azione con l'  `[EnableCors]` attributo. Quando l' `[EnableCors]` attributo viene applicato a un metodo di azione/modello di pagina, mentre CORS è abilitato nel middleware, vengono applicati **entrambi** i criteri. Si consiglia di **non** combinare i criteri. Usare l' `[EnableCors]` attributo o il middleware, **non entrambi**. Quando `[EnableCors]` si usa, **non** definire criteri predefiniti.

Il codice seguente applica un criterio diverso a ogni metodo:

[!code-csharp[](cors/sample/Cors/WebAPI/Controllers/WidgetController.cs?name=snippet&highlight=6,14)]

Il codice seguente crea un criterio predefinito CORS e un criterio denominato `"AnotherPolicy"` :

[!code-csharp[](cors/sample/Cors/WebAPI/StartupMultiPolicy.cs?name=snippet&highlight=12-28)]

### <a name="disable-cors"></a>Disabilitare CORS

L'attributo [ &lbrack; DISABLECORS &rbrack; ](xref:Microsoft.AspNetCore.Cors.DisableCorsAttribute) Disabilita CORS per il controller, il modello di pagina o l'azione.

<a name="cpo"></a>

## <a name="cors-policy-options"></a>Opzioni dei criteri di CORS

In questa sezione vengono descritte le varie opzioni che è possibile impostare in un criterio CORS:

* [Imposta le origini consentite](#set-the-allowed-origins)
* [Impostare i metodi HTTP consentiti](#set-the-allowed-http-methods)
* [Impostare le intestazioni della richiesta consentita](#set-the-allowed-request-headers)
* [Impostare le intestazioni di risposta esposte](#set-the-exposed-response-headers)
* [Credenziali nelle richieste tra le origini](#credentials-in-cross-origin-requests)
* [Imposta la data di scadenza preliminare](#set-the-preflight-expiration-time)

<xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsOptions.AddPolicy*> viene chiamato in `Startup.ConfigureServices` . Per alcune opzioni, può essere utile leggere prima la sezione [come funziona CORS](#how-cors) .

## <a name="set-the-allowed-origins"></a>Imposta le origini consentite

<xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.AllowAnyOrigin*>: Consente le richieste CORS da tutte le origini con qualsiasi schema ( `http` o `https` ). `AllowAnyOrigin` non è sicuro perché *qualsiasi sito Web* può effettuare richieste tra origini per l'app.

> [!NOTE]
> La definizione di `AllowAnyOrigin` e `AllowCredentials` è una configurazione non sicura e può comportare la falsificazione della richiesta tra siti. Per un'app sicura, specificare un elenco esatto di origini se il client deve autorizzare se stesso ad accedere alle risorse del server.

`AllowAnyOrigin` influiscono sulle richieste preliminari e sull' `Access-Control-Allow-Origin` intestazione. Per ulteriori informazioni, vedere la sezione [preflight requests](#preflight-requests) .

<xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.SetIsOriginAllowedToAllowWildcardSubdomains*>: Imposta la <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicy.IsOriginAllowed*> proprietà dei criteri in modo che sia una funzione che consente alle origini di corrispondere a un dominio con caratteri jolly configurato durante la valutazione se l'origine è consentita.

[!code-csharp[](cors/sample/CorsExample4/Startup.cs?range=100-105&highlight=4-5)]

### <a name="set-the-allowed-http-methods"></a>Impostare i metodi HTTP consentiti

<xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.AllowAnyMethod*>:

* Consente qualsiasi metodo HTTP:
* Influiscono sulle richieste preliminari e sull' `Access-Control-Allow-Methods` intestazione. Per ulteriori informazioni, vedere la sezione [preflight requests](#preflight-requests) .

### <a name="set-the-allowed-request-headers"></a>Impostare le intestazioni della richiesta consentita

Per consentire l'invio di intestazioni specifiche in una richiesta CORS, denominate *intestazioni richiesta autore*, chiamare <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.WithHeaders*> e specificare le intestazioni consentite:

[!code-csharp[](cors/sample/CorsExample4/Startup.cs?range=55-60&highlight=5)]

Per consentire tutte le intestazioni di richiesta di autore, chiamare <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.AllowAnyHeader*> :

[!code-csharp[](cors/sample/CorsExample4/Startup.cs?range=64-69&highlight=5)]

Questa impostazione influiscono sulle richieste preliminari e sull' `Access-Control-Request-Headers` intestazione. Per ulteriori informazioni, vedere la sezione [preflight requests](#preflight-requests) .

Il middleware CORS consente sempre l'invio di quattro intestazioni in, `Access-Control-Request-Headers` indipendentemente dai valori configurati in CorsPolicy. Headers. Questo elenco di intestazioni include:

* `Accept`
* `Accept-Language`
* `Content-Language`
* `Origin`

Si consideri, ad esempio, un'app configurata come segue:

```csharp
app.UseCors(policy => policy.WithHeaders(HeaderNames.CacheControl));
```

Il middleware CORS risponde correttamente a una richiesta preliminare con la seguente intestazione di richiesta perché `Content-Language` è sempre consentita:

```
Access-Control-Request-Headers: Cache-Control, Content-Language
```

### <a name="set-the-exposed-response-headers"></a>Impostare le intestazioni di risposta esposte

Per impostazione predefinita, il browser non espone tutte le intestazioni di risposta all'app. Per altre informazioni, vedere la pagina relativa alla [condivisione delle risorse tra le origini W3C (terminologia): intestazione della risposta semplice](https://www.w3.org/TR/cors/#simple-response-header).

Le intestazioni di risposta disponibili per impostazione predefinita sono:

* `Cache-Control`
* `Content-Language`
* `Content-Type`
* `Expires`
* `Last-Modified`
* `Pragma`

La specifica CORS chiama queste intestazioni di *risposta semplici*. Per rendere disponibili altre intestazioni per l'app, chiamare <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.WithExposedHeaders*> :

[!code-csharp[](cors/sample/CorsExample4/Startup.cs?range=73-78&highlight=5)]

### <a name="credentials-in-cross-origin-requests"></a>Credenziali nelle richieste tra le origini

Le credenziali richiedono una gestione speciale in una richiesta CORS. Per impostazione predefinita, il browser non invia le credenziali con una richiesta tra le origini. Le credenziali includono cookie s e gli schemi di autenticazione HTTP. Per inviare le credenziali con una richiesta tra le origini, il client deve impostare `XMLHttpRequest.withCredentials` su `true` .

Utilizzando `XMLHttpRequest` direttamente:

```javascript
var xhr = new XMLHttpRequest();
xhr.open('get', 'https://www.example.com/api/test');
xhr.withCredentials = true;
```

Uso di jQuery:

```javascript
$.ajax({
  type: 'get',
  url: 'https://www.example.com/api/test',
  xhrFields: {
    withCredentials: true
  }
});
```

Uso dell' [API fetch](https://developer.mozilla.org/docs/Web/API/Fetch_API):

```javascript
fetch('https://www.example.com/api/test', {
    credentials: 'include'
});
```

Il server deve consentire le credenziali. Per consentire le credenziali tra le origini, chiamare <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.AllowCredentials*> :

[!code-csharp[](cors/sample/CorsExample4/Startup.cs?range=82-87&highlight=5)]

La risposta HTTP include un' `Access-Control-Allow-Credentials` intestazione che indica al browser che il server consente le credenziali per una richiesta tra le origini.

Se il browser invia le credenziali, ma la risposta non include un' `Access-Control-Allow-Credentials` intestazione valida, il browser non espone la risposta all'app e la richiesta tra le origini ha esito negativo.

Consentire le credenziali tra le origini costituisce un rischio per la sicurezza. Un sito Web in un altro dominio può inviare le credenziali dell'utente che ha eseguito l'accesso all'app per conto dell'utente senza la conoscenza dell'utente. <!-- TODO Review: When using `AllowCredentials`, all CORS enabled domains must be trusted.
I don't like "all CORS enabled domains must be trusted", because it implies that if you're not using  `AllowCredentials`, domains don't need to be trusted. -->

La specifica CORS indica inoltre che l'impostazione delle origini su `"*"` (tutte le origini) non è valida se l' `Access-Control-Allow-Credentials` intestazione è presente.

### <a name="preflight-requests"></a>Richieste preliminari

Per alcune richieste CORS, il browser invia una richiesta aggiuntiva prima di effettuare la richiesta effettiva. Questa richiesta è detta *richiesta preliminare*. Il browser può ignorare la richiesta preliminare se vengono soddisfatte le condizioni seguenti:

* Il metodo di richiesta è GET, HEAD o POST.
* L'app non imposta intestazioni di richiesta diverse da `Accept` , `Accept-Language` , `Content-Language` , `Content-Type` o `Last-Event-ID` .
* L' `Content-Type` intestazione, se impostata, presenta uno dei valori seguenti:
  * `application/x-www-form-urlencoded`
  * `multipart/form-data`
  * `text/plain`

La regola sulle intestazioni di richiesta impostate per la richiesta del client si applica alle intestazioni impostate dall'app chiamando `setRequestHeader` sull' `XMLHttpRequest` oggetto. La specifica CORS chiama queste intestazioni di *richiesta autore* intestazioni. La regola non si applica alle intestazioni che il browser può impostare, ad esempio `User-Agent` , `Host` o `Content-Length` .

Di seguito è riportato un esempio di una richiesta preliminare:

```
OPTIONS https://myservice.azurewebsites.net/api/test HTTP/1.1
Accept: */*
Origin: https://myclient.azurewebsites.net
Access-Control-Request-Method: PUT
Access-Control-Request-Headers: accept, x-my-custom-header
Accept-Encoding: gzip, deflate
User-Agent: Mozilla/5.0 (compatible; MSIE 10.0; Windows NT 6.2; WOW64; Trident/6.0)
Host: myservice.azurewebsites.net
Content-Length: 0
```

La richiesta pre-flight usa il metodo delle opzioni HTTP. Sono incluse due intestazioni speciali:

* `Access-Control-Request-Method`: Metodo HTTP che verrà usato per la richiesta effettiva.
* `Access-Control-Request-Headers`: Elenco di intestazioni di richiesta impostate dall'app sulla richiesta effettiva. Come indicato in precedenza, non include le intestazioni impostate dal browser, ad esempio `User-Agent` .

<!-- I think this needs to be removed, was put here accidently -->

Quando CORS è abilitato con i criteri appropriati, ASP.NET Core in genere risponde automaticamente alle richieste preliminari di CORS. Vedere l' [attributo [HttpOptions] per le richieste preliminari](#pro).

Una richiesta preliminare CORS può includere un' `Access-Control-Request-Headers` intestazione, che indica al server le intestazioni inviate con la richiesta effettiva.

Per consentire intestazioni specifiche, chiamare <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.WithHeaders*> :

[!code-csharp[](cors/sample/CorsExample4/Startup.cs?range=55-60&highlight=5)]

Per consentire tutte le intestazioni di richiesta di autore, chiamare <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.AllowAnyHeader*> :

[!code-csharp[](cors/sample/CorsExample4/Startup.cs?range=64-69&highlight=5)]

I browser non sono completamente coerenti nel modo in cui vengono impostati `Access-Control-Request-Headers` . Se si impostano le intestazioni su un valore diverso da `"*"` (o si utilizza <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicy.AllowAnyHeader*> ), è necessario includere almeno `Accept` , `Content-Type` e `Origin` , più eventuali intestazioni personalizzate che si desidera supportare.

Di seguito è riportato un esempio di risposta alla richiesta preliminare (presupponendo che il server consenta la richiesta):

```
HTTP/1.1 200 OK
Cache-Control: no-cache
Pragma: no-cache
Content-Length: 0
Access-Control-Allow-Origin: https://myclient.azurewebsites.net
Access-Control-Allow-Headers: x-my-custom-header
Access-Control-Allow-Methods: PUT
Date: Wed, 20 May 2015 06:33:22 GMT
```

La risposta include un' `Access-Control-Allow-Methods` intestazione che elenca i metodi consentiti e, facoltativamente `Access-Control-Allow-Headers` , un'intestazione, che elenca le intestazioni consentite. Se la richiesta preliminare ha esito positivo, il browser invia la richiesta effettiva.

Se la richiesta preliminare viene negata, l'app restituisce una risposta *200 OK* , ma non invia le intestazioni CORS. Pertanto, il browser non tenta di eseguire la richiesta tra origini.

### <a name="set-the-preflight-expiration-time"></a>Imposta la data di scadenza preliminare

L' `Access-Control-Max-Age` intestazione specifica per quanto tempo la risposta alla richiesta preliminare può essere memorizzata nella cache. Per impostare questa intestazione, chiamare <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.SetPreflightMaxAge*> :

[!code-csharp[](cors/sample/CorsExample4/Startup.cs?range=91-96&highlight=5)]

<a name="how-cors"></a>

## <a name="how-cors-works"></a>Funzionamento di CORS

In questa sezione viene descritto cosa accade in una richiesta [CORS](https://developer.mozilla.org/docs/Web/HTTP/CORS) al livello dei messaggi HTTP.

* CORS **non** è una funzionalità di sicurezza. CORS è uno standard W3C che consente a un server di ridurre i criteri della stessa origine.
  * Ad esempio, un attore malintenzionato potrebbe usare [Impedisci lo scripting tra siti (XSS)](xref:security/cross-site-scripting) per il sito ed eseguire una richiesta tra siti al sito abilitato per CORS per rubare le informazioni.
* L'API non è più sicura consentendo CORS.
  * Il client (browser) deve applicare CORS. Il server esegue la richiesta e restituisce la risposta, ovvero il client che restituisce un errore e blocca la risposta. Uno degli strumenti seguenti, ad esempio, visualizzerà la risposta del server:
    * [Fiddler](https://www.telerik.com/fiddler)
    * [Postman](https://www.getpostman.com/)
    * [HttpClient .NET](/dotnet/csharp/tutorials/console-webapiclient)
    * Un Web browser immettendo l'URL nella barra degli indirizzi.
* Si tratta di un modo per consentire a un server di consentire ai browser di eseguire una richiesta dell' [API di recupero](https://developer.mozilla.org/docs/Web/API/Fetch_API) o di [XHR](https://developer.mozilla.org/docs/Web/API/XMLHttpRequest) tra le origini, che altrimenti sarebbe proibita.
  * I browser (senza CORS) non possono eseguire richieste tra le origini. Prima di CORS, [JSONP](https://www.w3schools.com/js/js_json_jsonp.asp) è stato usato per aggirare questa restrizione. JSONP non usa XHR, usa il `<script>` tag per ricevere la risposta. Gli script possono essere caricati tra le origini.

La [specifica CORS](https://www.w3.org/TR/cors/) ha introdotto diverse nuove intestazioni HTTP che consentono richieste tra le origini. Se un browser supporta CORS, queste intestazioni vengono impostate automaticamente per le richieste tra le origini. Il codice JavaScript personalizzato non è necessario per abilitare CORS.

Di seguito è riportato un esempio di richiesta tra le origini. L' `Origin` intestazione fornisce il dominio del sito che sta effettuando la richiesta. L' `Origin` intestazione è obbligatoria e deve essere diversa dall'host.

```
GET https://myservice.azurewebsites.net/api/test HTTP/1.1
Referer: https://myclient.azurewebsites.net/
Accept: */*
Accept-Language: en-US
Origin: https://myclient.azurewebsites.net
Accept-Encoding: gzip, deflate
User-Agent: Mozilla/5.0 (compatible; MSIE 10.0; Windows NT 6.2; WOW64; Trident/6.0)
Host: myservice.azurewebsites.net
```

Se il server consente la richiesta, imposta l' `Access-Control-Allow-Origin` intestazione nella risposta. Il valore di questa intestazione corrisponde all' `Origin` intestazione della richiesta o è il valore del carattere jolly `"*"` , ovvero qualsiasi origine è consentita:

```
HTTP/1.1 200 OK
Cache-Control: no-cache
Pragma: no-cache
Content-Type: text/plain; charset=utf-8
Access-Control-Allow-Origin: https://myclient.azurewebsites.net
Date: Wed, 20 May 2015 06:27:30 GMT
Content-Length: 12

Test message
```

Se la risposta non include l' `Access-Control-Allow-Origin` intestazione, la richiesta tra le origini ha esito negativo. In particolare, il browser non consente la richiesta. Anche se il server restituisce una risposta con esito positivo, il browser non rende disponibile la risposta all'app client.

<a name="test"></a>

## <a name="test-cors"></a>Testare CORS

Per testare CORS:

1. [Creare un progetto API](xref:tutorials/first-web-api). In alternativa, è possibile [scaricare l'esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/cors/sample/Cors).
1. Abilitare CORS usando uno degli approcci descritti in questo documento. Ad esempio:

  [!code-csharp[](cors/sample/Cors/WebAPI/StartupTest.cs?name=snippet2&highlight=13-18)]

  > [!WARNING]
  > `WithOrigins("https://localhost:<port>");` deve essere usato solo per il test di un'app di esempio simile al [codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/live/aspnetcore/security/cors/sample/Cors)per il download.

1. Creare un progetto di app Web ( Razor pagine o MVC). Nell'esempio vengono utilizzate le Razor pagine. È possibile creare l'app Web nella stessa soluzione del progetto API.
1. Aggiungere il codice evidenziato seguente al file *index. cshtml* :

  [!code-cshtml[](cors/sample/Cors/ClientApp/Pages/Index2.cshtml?highlight=7-99)]

1. Nel codice precedente sostituire `url: 'https://<web app>.azurewebsites.net/api/values/1',` con l'URL dell'app distribuita.
1. Distribuire il progetto API. Ad esempio, [eseguire la distribuzione in Azure](xref:host-and-deploy/azure-apps/index).
1. Eseguire le Razor pagine o l'app MVC dal desktop e fare clic sul pulsante **test** . Usare gli strumenti F12 per esaminare i messaggi di errore.
1. Rimuovere l'origine localhost da `WithOrigins` e distribuire l'app. In alternativa, eseguire l'app client con una porta diversa. Ad esempio, eseguire da Visual Studio.
1. Eseguire il test con l'app client. Gli errori CORS restituiscono un errore, ma il messaggio di errore non è disponibile per JavaScript. Usare la scheda console negli strumenti F12 per visualizzare l'errore. A seconda del browser, viene ricevuto un errore (nella console strumenti F12) simile al seguente:

   * Uso di Microsoft Edge:

     **SEC7120: [CORS] l'origine `https://localhost:44375` non è stata trovata `https://localhost:44375` nell'intestazione della risposta Access-Control-Allow-Origin per la risorsa tra le origini in `https://webapi.azurewebsites.net/api/values/1`**

   * Uso di Chrome:

     **L'accesso a XMLHttpRequest `https://webapi.azurewebsites.net/api/values/1` da Origin `https://localhost:44375` è stato bloccato dal criterio CORS: non è presente alcuna intestazione ' Access-Control-Allow-Origin ' nella risorsa richiesta.**
     
Gli endpoint abilitati per CORS possono essere testati con uno strumento, ad esempio [Fiddler](https://www.telerik.com/fiddler) o [postazione](https://www.getpostman.com/). Quando si utilizza uno strumento, l'origine della richiesta specificata dall' `Origin` intestazione deve essere diversa dall'host che riceve la richiesta. Se la richiesta non è *tra origini* basate sul valore dell' `Origin` intestazione:

* Non è necessario che il middleware CORS elabori la richiesta.
* Le intestazioni CORS non vengono restituite nella risposta.

## <a name="cors-in-iis"></a>CORS in IIS

Quando si esegue la distribuzione in IIS, CORS deve essere eseguito prima dell'autenticazione di Windows se il server non è configurato per consentire l'accesso anonimo. Per supportare questo scenario, è necessario installare e configurare il [modulo cors di IIS](https://www.iis.net/downloads/microsoft/iis-cors-module) per l'app.

## <a name="additional-resources"></a>Risorse aggiuntive

* [Condivisione di risorse tra le origini (CORS)](https://developer.mozilla.org/docs/Web/HTTP/CORS)
* [Introduzione al modulo CORS di IIS](https://blogs.iis.net/iisteam/getting-started-with-the-iis-cors-module)

::: moniker-end
