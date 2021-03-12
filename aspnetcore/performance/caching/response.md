---
title: Memorizzazione nella cache delle risposte in ASP.NET Core
author: rick-anderson
description: Informazioni su come usare la memorizzazione nella cache delle risposte per ridurre i requisiti di larghezza di banda e migliorare le prestazioni delle app ASP.NET Core.
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.date: 11/04/2019
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
uid: performance/caching/response
ms.openlocfilehash: 539ddb118279adb3a53394cdb0c2e5169092ebc0
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102589231"
---
# <a name="response-caching-in-aspnet-core"></a>Memorizzazione nella cache delle risposte in ASP.NET Core

Di [John Luo](https://github.com/JunTaoLuo), [Rick Anderson](https://twitter.com/RickAndMSFT)e [Steve Smith](https://ardalis.com/)

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/performance/caching/response/samples) ([procedura per il download](xref:index#how-to-download-a-sample))

La memorizzazione nella cache delle risposte riduce il numero di richieste effettuate da un client o da un proxy a un server Web. La memorizzazione nella cache delle risposte riduce anche la quantità di lavoro eseguita dal server Web per generare una risposta. La memorizzazione nella cache delle risposte è controllata dalle intestazioni che specificano come si desidera che client, proxy e middleware memorizzino le risposte.

L' [attributo ResponseCache](#responsecache-attribute) fa parte dell'impostazione delle intestazioni di memorizzazione nella cache delle risposte. I client e i proxy intermedi devono rispettare le intestazioni per la memorizzazione nella cache delle risposte nella [specifica HTTP 1,1 Caching](https://tools.ietf.org/html/rfc7234).

Per la memorizzazione nella cache sul lato server che segue la specifica di Caching HTTP 1,1, usare [middleware di caching della risposta](xref:performance/caching/middleware). Il middleware può usare le <xref:Microsoft.AspNetCore.Mvc.ResponseCacheAttribute> proprietà per influenzare il comportamento di memorizzazione nella cache sul lato server.

## <a name="http-based-response-caching"></a>Memorizzazione nella cache delle risposte basata su HTTP

La [specifica di Caching HTTP 1,1](https://tools.ietf.org/html/rfc7234) descrive il comportamento delle cache Internet. L'intestazione HTTP primaria utilizzata per la memorizzazione nella cache è [Cache-Control](https://tools.ietf.org/html/rfc7234#section-5.2), che viene utilizzata per specificare le *direttive* della cache. Le direttive controllano il comportamento di memorizzazione nella cache come richieste da client a server e quando le risposte vengono riportate dai server ai client. Le richieste e le risposte passano attraverso i server proxy e i server proxy devono essere conformi anche alla specifica HTTP 1,1 Caching.

`Cache-Control`Le direttive comuni sono illustrate nella tabella seguente.

| Direttiva                                                       | Azione |
| --------------------------------------------------------------- | ------ |
| [public](https://tools.ietf.org/html/rfc7234#section-5.2.2.5)   | Una cache può archiviare la risposta. |
| [privata](https://tools.ietf.org/html/rfc7234#section-5.2.2.6)  | La risposta non deve essere archiviata da una cache condivisa. Una cache privata può archiviare e riutilizzare la risposta. |
| [validità massima](https://tools.ietf.org/html/rfc7234#section-5.2.1.1)  | Il client non accetta una risposta la cui età è superiore al numero di secondi specificato. Esempi: `max-age=60` (60 secondi), `max-age=2592000` (1 mese) |
| [no-cache](https://tools.ietf.org/html/rfc7234#section-5.2.1.4) | **Sulle richieste**: una cache non deve usare una risposta archiviata per soddisfare la richiesta. Il server di origine rigenera la risposta per il client e il middleware aggiorna la risposta archiviata nella cache.<br><br>**In** risposta: non è necessario usare la risposta per una richiesta successiva senza convalida nel server di origine. |
| [Nessun archivio](https://tools.ietf.org/html/rfc7234#section-5.2.1.5) | **Sulle richieste**: una cache non deve archiviare la richiesta.<br><br>**Nelle risposte**: una cache non deve archiviare alcuna parte della risposta. |

Nella tabella seguente sono illustrate le altre intestazioni della cache che svolgono un ruolo nella memorizzazione nella cache.

| Intestazione                                                     | Funzione |
| ---------------------------------------------------------- | -------- |
| [Age](https://tools.ietf.org/html/rfc7234#section-5.1)     | Stima della quantità di tempo in secondi trascorsa dalla generazione della risposta o dalla convalida corretta nel server di origine. |
| [Scadenza](https://tools.ietf.org/html/rfc7234#section-5.3) | Tempo trascorso il quale la risposta è considerata obsoleta. |
| [Pragma](https://tools.ietf.org/html/rfc7234#section-5.4)  | Esiste per la compatibilità con le versioni precedenti con le cache HTTP/1.0 per l'impostazione del `no-cache` comportamento. Se l' `Cache-Control` intestazione è presente, l' `Pragma` intestazione viene ignorata. |
| [Variare](https://tools.ietf.org/html/rfc7231#section-7.1.4)  | Specifica che una risposta memorizzata nella cache non deve essere inviata a meno che tutti i `Vary` campi di intestazione corrispondano sia nella richiesta originale della risposta memorizzata nella cache che nella nuova richiesta. |

## <a name="http-based-caching-respects-request-cache-control-directives"></a>La memorizzazione nella cache basata su HTTP rispetta le direttive Cache-Control di richiesta

La [specifica di Caching HTTP 1,1 per l'intestazione Cache-Control](https://tools.ietf.org/html/rfc7234#section-5.2) richiede una cache per rispettare un' `Cache-Control` intestazione valida inviata dal client. Un client può effettuare richieste con un `no-cache` valore di intestazione e forzare il server a generare una nuova risposta per ogni richiesta.

Per rispettare sempre `Cache-Control` le intestazioni delle richieste client, è opportuno considerare l'obiettivo della memorizzazione nella cache HTTP. Nella specifica ufficiale, la memorizzazione nella cache è finalizzata a ridurre la latenza e il sovraccarico di rete per soddisfare le richieste in una rete di client, proxy e server. Non è necessariamente un modo per controllare il carico su un server di origine.

Il comportamento di memorizzazione nella cache non è disponibile per gli sviluppatori quando si usa il [middleware di memorizzazione nella cache delle risposte](xref:performance/caching/middleware) , perché il middleware rispetta la specifica di Caching ufficiale. I [miglioramenti pianificati per il middleware](https://github.com/dotnet/AspNetCore/issues/2612) sono un'opportunità per configurare il middleware in modo da ignorare l'intestazione di una richiesta `Cache-Control` quando si decide di gestire una risposta memorizzata nella cache. I miglioramenti pianificati offrono la possibilità di controllare meglio il carico del server.

## <a name="other-caching-technology-in-aspnet-core"></a>Altra tecnologia di memorizzazione nella cache in ASP.NET Core

### <a name="in-memory-caching"></a>Memorizzazione nella cache in memoria

La memorizzazione nella cache in memoria usa la memoria del server per archiviare i dati memorizzati nella cache. Questo tipo di memorizzazione nella cache è adatto per un singolo server o più server che usano *sessioni permanenti*. Sessioni permanenti significa che le richieste provenienti da un client vengono sempre indirizzate allo stesso server per l'elaborazione.

Per altre informazioni, vedere <xref:performance/caching/memory>.

### <a name="distributed-cache"></a>Cache distribuita

Usare una cache distribuita per archiviare i dati in memoria quando l'app è ospitata in un cloud o in server farm. La cache è condivisa tra i server che elaborano le richieste. Un client può inviare una richiesta gestita da qualsiasi server del gruppo se sono disponibili dati memorizzati nella cache per il client. ASP.NET Core funziona con le cache distribuite SQL Server, [Redis](https://www.nuget.org/packages/Microsoft.Extensions.Caching.StackExchangeRedis)e [NCache](https://www.nuget.org/packages/Alachisoft.NCache.OpenSource.SDK/) .

Per altre informazioni, vedere <xref:performance/caching/distributed>.

### <a name="cache-tag-helper"></a>Helper per tag di cache

Memorizza nella cache il contenuto di una visualizzazione o Razor di una pagina MVC con l'helper tag di cache. L'helper tag di cache usa la memorizzazione nella cache in memoria per archiviare i dati.

Per altre informazioni, vedere <xref:mvc/views/tag-helpers/builtin-th/cache-tag-helper>.

### <a name="distributed-cache-tag-helper"></a>Helper tag di cache distribuita

Consente di memorizzare nella cache il contenuto di una visualizzazione o Razor di una pagina MVC negli scenari cloud distribuito o Web farm con l'helper tag di cache distribuita. L'helper tag di cache distribuita usa SQL Server, [Redis](https://www.nuget.org/packages/Microsoft.Extensions.Caching.StackExchangeRedis)o [NCache](https://www.nuget.org/packages/Alachisoft.NCache.OpenSource.SDK/) per archiviare i dati.

Per altre informazioni, vedere <xref:mvc/views/tag-helpers/builtin-th/distributed-cache-tag-helper>.

## <a name="responsecache-attribute"></a>Attributo ResponseCache

<xref:Microsoft.AspNetCore.Mvc.ResponseCacheAttribute>Specifica i parametri necessari per impostare le intestazioni appropriate nella memorizzazione nella cache delle risposte.

> [!WARNING]
> Disabilitare la memorizzazione nella cache per il contenuto che contiene informazioni per i client autenticati. La memorizzazione nella cache deve essere abilitata solo per il contenuto che non cambia in base all'identità di un utente o se un utente ha eseguito l'accesso.

<xref:Microsoft.AspNetCore.Mvc.CacheProfile.VaryByQueryKeys> varia la risposta memorizzata in base ai valori dell'elenco specificato di chiavi di query. Quando viene specificato un singolo valore `*` , il middleware varia le risposte in base a tutti i parametri della stringa di query della richiesta.

Il [middleware di memorizzazione nella cache delle risposte](xref:performance/caching/middleware) deve essere abilitato per impostare la <xref:Microsoft.AspNetCore.Mvc.CacheProfile.VaryByQueryKeys> Proprietà. In caso contrario, viene generata un'eccezione in fase di esecuzione. Non esiste un'intestazione HTTP corrispondente per la <xref:Microsoft.AspNetCore.Mvc.CacheProfile.VaryByQueryKeys> Proprietà. La proprietà è una funzionalità HTTP gestita dal middleware di memorizzazione nella cache delle risposte. Affinché il middleware possa gestire una risposta memorizzata nella cache, la stringa di query e il valore della stringa di query devono corrispondere a una richiesta precedente. Si consideri, ad esempio, la sequenza di richieste e i risultati mostrati nella tabella seguente.

| Richiesta                          | Risultato                    |
| -------------------------------- | ------------------------- |
| `http://example.com?key1=value1` | Restituito dal server. |
| `http://example.com?key1=value1` | Restituito dal middleware. |
| `http://example.com?key1=value2` | Restituito dal server. |

La prima richiesta viene restituita dal server e memorizzata nella cache nel middleware. La seconda richiesta viene restituita dal middleware perché la stringa di query corrisponde alla richiesta precedente. La terza richiesta non si trovi nella cache middleware perché il valore della stringa di query non corrisponde a una richiesta precedente.

<xref:Microsoft.AspNetCore.Mvc.ResponseCacheAttribute>Viene usato per configurare e creare (tramite <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory> ) un oggetto `Microsoft.AspNetCore.Mvc.Internal.ResponseCacheFilter` . `ResponseCacheFilter`Esegue le operazioni di aggiornamento delle intestazioni HTTP e delle funzionalità appropriate della risposta. Il filtro:

* Rimuove le intestazioni esistenti per `Vary` , `Cache-Control` e `Pragma` .
* Scrive le intestazioni appropriate in base alle proprietà impostate in <xref:Microsoft.AspNetCore.Mvc.ResponseCacheAttribute> .
* Aggiorna la funzionalità HTTP di caching delle risposte se <xref:Microsoft.AspNetCore.Mvc.CacheProfile.VaryByQueryKeys> è impostato.

### <a name="vary"></a>Variare

Questa intestazione viene scritta solo quando la <xref:Microsoft.AspNetCore.Mvc.CacheProfile.VaryByHeader> proprietà è impostata. Proprietà impostata sul valore della `Vary` Proprietà. Nell'esempio seguente viene utilizzata la <xref:Microsoft.AspNetCore.Mvc.CacheProfile.VaryByHeader> proprietà:

[!code-csharp[](response/samples/2.x/ResponseCacheSample/Pages/Cache1.cshtml.cs?name=snippet)]

Usando l'app di esempio, visualizzare le intestazioni della risposta con gli strumenti di rete del browser. Le intestazioni di risposta seguenti vengono inviate con la risposta della pagina cache1:

```
Cache-Control: public,max-age=30
Vary: User-Agent
```

### <a name="nostore-and-locationnone"></a>NoStore e location. None

<xref:Microsoft.AspNetCore.Mvc.CacheProfile.NoStore> esegue l'override della maggior parte delle altre proprietà. Quando questa proprietà è impostata su `true` , l' `Cache-Control` intestazione viene impostata su `no-store` . Se <xref:Microsoft.AspNetCore.Mvc.CacheProfile.Location> è impostato su `None` :

* `Cache-Control` è impostato su `no-store,no-cache`.
* `Pragma` è impostato su `no-cache`.

Se <xref:Microsoft.AspNetCore.Mvc.CacheProfile.NoStore> è `false` e <xref:Microsoft.AspNetCore.Mvc.CacheProfile.Location> è `None` , `Cache-Control` e `Pragma` sono impostati su `no-cache` .

<xref:Microsoft.AspNetCore.Mvc.CacheProfile.NoStore> viene in genere impostato su `true` per le pagine di errore. La pagina cache2 nell'app di esempio genera intestazioni di risposta che indicano al client di non archiviare la risposta.

[!code-csharp[](response/samples/2.x/ResponseCacheSample/Pages/Cache2.cshtml.cs?name=snippet)]

L'app di esempio restituisce la pagina cache2 con le intestazioni seguenti:

```
Cache-Control: no-store,no-cache
Pragma: no-cache
```

### <a name="location-and-duration"></a>Località e durata

Per abilitare la memorizzazione nella cache, <xref:Microsoft.AspNetCore.Mvc.CacheProfile.Duration> deve essere impostato su un valore positivo e <xref:Microsoft.AspNetCore.Mvc.CacheProfile.Location> deve essere `Any` (valore predefinito) o `Client` . Il Framework imposta l' `Cache-Control` intestazione sul valore della posizione seguito dalla `max-age` della risposta.

<xref:Microsoft.AspNetCore.Mvc.CacheProfile.Location>le opzioni di `Any` e vengono `Client` convertite in `Cache-Control` valori di intestazione `public` rispettivamente di e `private` . Come indicato nella sezione [NoStore e location. None](#nostore-and-locationnone) , l'impostazione <xref:Microsoft.AspNetCore.Mvc.CacheProfile.Location> di su `None` imposta entrambe `Cache-Control` le `Pragma` intestazioni e su `no-cache` .

`Location.Any` ( `Cache-Control` impostato su `public` ) indica che il *client o qualsiasi proxy intermedio* può memorizzare nella cache il valore, incluso il [middleware di memorizzazione nella cache delle risposte](xref:performance/caching/middleware).

`Location.Client` ( `Cache-Control` impostato su `private` ) indica che *solo il client* può memorizzare nella cache il valore. Nessuna cache intermedia deve memorizzare nella cache il valore, incluso il [middleware di memorizzazione nella cache delle risposte](xref:performance/caching/middleware).

Le intestazioni di controllo della cache forniscono semplicemente indicazioni ai client e ai proxy intermedi quando e come memorizzare nella cache le risposte. Non vi è alcuna garanzia che i client e i proxy soddisfino la [specifica HTTP 1,1 Caching](https://tools.ietf.org/html/rfc7234). Il [middleware di memorizzazione nella cache delle risposte](xref:performance/caching/middleware) segue sempre le regole di memorizzazione nella cache definite dalla specifica.

L'esempio seguente mostra il modello di pagina Cache3 dall'app di esempio e le intestazioni prodotte impostando <xref:Microsoft.AspNetCore.Mvc.CacheProfile.Duration> e lasciando il <xref:Microsoft.AspNetCore.Mvc.CacheProfile.Location> valore predefinito:

[!code-csharp[](response/samples/2.x/ResponseCacheSample/Pages/Cache3.cshtml.cs?name=snippet)]

L'app di esempio restituisce la pagina Cache3 con l'intestazione seguente:

```
Cache-Control: public,max-age=10
```

### <a name="cache-profiles"></a>Profili cache

Anziché duplicare le impostazioni della cache di risposta su molti attributi dell'azione del controller, è possibile configurare i profili della cache come opzioni durante la configurazione di MVC/ Razor pagine in `Startup.ConfigureServices` . I valori trovati in un profilo della cache a cui viene fatto riferimento vengono usati come valori predefiniti da <xref:Microsoft.AspNetCore.Mvc.ResponseCacheAttribute> e vengono sottoposti a override da qualsiasi proprietà specificata nell'attributo.

Configurare un profilo della cache. L'esempio seguente mostra un profilo della cache di 30 secondi nell'app di esempio `Startup.ConfigureServices` :

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](response/samples/3.x/Startup.cs?name=snippet1)]

::: moniker-end

::: moniker range="< aspnetcore-3.0"

[!code-csharp[](response/samples/2.x/ResponseCacheSample/Startup.cs?name=snippet1)]

::: moniker-end

Il modello di pagina Cache4 dell'app di esempio fa riferimento al `Default30` profilo della cache:

[!code-csharp[](response/samples/2.x/ResponseCacheSample/Pages/Cache4.cshtml.cs?name=snippet)]

Il <xref:Microsoft.AspNetCore.Mvc.ResponseCacheAttribute> può essere applicato a:

* Razor Pagine: gli attributi non possono essere applicati ai metodi del gestore.
* Controller MVC.
* Metodi di azione MVC: gli attributi a livello di metodo eseguono l'override delle impostazioni specificate negli attributi a livello di classe.

Intestazione risultante applicata alla risposta della pagina Cache4 dal `Default30` profilo della cache:

```
Cache-Control: public,max-age=30
```

## <a name="additional-resources"></a>Risorse aggiuntive

* [Archiviazione delle risposte nelle cache](https://tools.ietf.org/html/rfc7234#section-3)
* [Cache-Control](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.9)
* <xref:performance/caching/memory>
* <xref:performance/caching/distributed>
* <xref:fundamentals/change-tokens>
* <xref:performance/caching/middleware>
* <xref:mvc/views/tag-helpers/builtin-th/cache-tag-helper>
* <xref:mvc/views/tag-helpers/builtin-th/distributed-cache-tag-helper>
