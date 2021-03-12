---
title: Sessione in ASP.NET Core
author: rick-anderson
description: Individuare gli approcci per mantenere la sessione tra le richieste.
ms.author: riande
ms.custom: mvc
ms.date: 03/06/2020
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
uid: fundamentals/app-state
ms.openlocfilehash: ab09867a1b1aa73e423babbe295036e292fc733f
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102586579"
---
# <a name="session-and-state-management-in-aspnet-core"></a>Gestione di sessioni e stato in ASP.NET Core

::: moniker range=">= aspnetcore-3.0"

Di [Rick Anderson](https://twitter.com/RickAndMSFT), [Kirk Larkin](https://twitter.com/serpent5)e [Diana lotti](https://github.com/DianaLaRose)

HTTP è un protocollo senza stato. Per impostazione predefinita, le richieste HTTP sono messaggi indipendenti che non mantengono i valori utente. Questo articolo descrive diversi approcci per mantenere i dati degli utenti tra le richieste.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/app-state/samples) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="state-management"></a>Gestione dello stato

Lo stato può essere archiviato usando diversi approcci. Ogni approccio è descritto di seguito in questo argomento.

| Approccio con risorsa di archiviazione | Meccanismo di archiviazione |
| ---------------- | ----------------- |
| [Cookies](#cookies) | HTTP cookie s. Può includere dati archiviati usando il codice dell'app sul lato server. |
| [Stato sessione](#session-state) | cookieCodice dell'app http s e lato server |
| [TempData](#tempdata) | cookieStato http s o sessione |
| [Stringhe di query](#query-strings) | Stringhe di query HTTP |
| [Campi nascosti](#hidden-fields) | Campi dei form HTTP |
| [HttpContext.Items](#httpcontextitems) | Codice app lato server |
| [Cache](#cache) | Codice app lato server |

## <a name="cookies"></a>Cookies

CookieArchivia i dati tra le richieste. Poiché cookie le istanze di vengono inviate con ogni richiesta, la loro dimensione deve essere mantenuta al minimo. Idealmente, solo un identificatore deve essere archiviato in un cookie con i dati archiviati dall'app. La maggior parte dei browser limita le cookie dimensioni a 4096 byte. cookiePer ogni dominio sono disponibili solo un numero limitato di s.

Poiché cookie le s sono soggette a manomissioni, devono essere convalidate dall'app. Cookiei possono essere eliminati dagli utenti e scadono nei client. Tuttavia, i cookie sono in genere la forma più durevole di persistenza dei dati nel client.

Cookiei vengono spesso usati per la personalizzazione, in cui il contenuto è personalizzato per un utente noto. L'utente viene solo identificato e non autenticato nella maggior parte dei casi. cookiePuò archiviare il nome dell'utente, il nome dell'account o l'ID utente univoco, ad esempio un GUID. Il cookie può essere usato per accedere alle impostazioni personalizzate dell'utente, ad esempio il colore di sfondo del sito Web preferito.

Vedere le [norme generali sulla protezione dei dati (GDPR) dell'Unione europea](https://ec.europa.eu/info/law/law-topic/data-protection) quando si rilasciano cookie e si gestiscono i problemi di privacy. Per altre informazioni, vedere il [supporto per il Regolamento generale sulla protezione dei dati in ASP.NET Core](xref:security/gdpr).

## <a name="session-state"></a>Stato sessione

Lo stato della sessione è uno scenario di ASP.NET Core per l'archiviazione dei dati utente mentre l'utente visualizza un'app Web. Lo stato della sessione usa un archivio gestito dall'app per rendere persistenti i dati tra le richieste provenienti da un client. I dati della sessione sono supportati da una cache e considerati dati temporanei. Il sito deve continuare a funzionare senza i dati della sessione. I dati critici dell'applicazione devono essere archiviati nel database utente e memorizzati nella cache della sessione solo al fine di ottimizzare le prestazioni.

La sessione non è supportata nelle [SignalR](xref:signalr/index) app perché un [ SignalR Hub](xref:signalr/hubs) può essere eseguito indipendentemente da un contesto http. Ad esempio, ciò può verificarsi quando una lunga richiesta di polling viene mantenuta aperta da un hub oltre la durata del contesto HTTP della richiesta.

ASP.NET Core gestisce lo stato della sessione fornendo un oggetto cookie al client contenente un ID di sessione. cookieID sessione:

* Viene inviato all'app con ogni richiesta.
* Viene usato dall'app per recuperare i dati della sessione.

Lo stato della sessione presenta i comportamenti seguenti:

* La sessione cookie è specifica del browser. Le sessioni non sono condivise tra più browser.
* cookieLa sessione s viene eliminata al termine della sessione del browser.
* Se cookie per una sessione scaduta viene ricevuto un oggetto, viene creata una nuova sessione che utilizza la stessa sessione cookie .
* Le sessioni vuote non vengono mantenute. La sessione deve avere almeno un valore impostato per salvare in modo permanente la sessione tra le richieste. Se una sessione non viene conservata, viene generato un nuovo ID sessione per ogni nuova richiesta.
* L'app conserva una sessione per un periodo di tempo limitato dopo l'ultima richiesta. L'app imposta il timeout della sessione o usa il valore predefinito, pari a 20 minuti. Lo stato della sessione è ideale per l'archiviazione dei dati utente:
  * Questo è specifico di una particolare sessione.
  * Dove i dati non richiedono l'archiviazione permanente tra le sessioni.
* I dati della sessione vengono eliminati quando <xref:Microsoft.AspNetCore.Http.ISession.Clear%2A?displayProperty=nameWithType> viene chiamata l'implementazione o quando la sessione scade.
* Non esiste un meccanismo predefinito per informare il codice dell'app che un browser client è stato chiuso o quando la sessione cookie è stata eliminata o scaduta nel client.
* cookiePer impostazione predefinita, lo stato della sessione non è contrassegnato come essenziale. Lo stato della sessione non è funzionante a meno che non sia consentito il rilevamento da parte del visitatore del sito. Per altre informazioni, vedere <xref:security/gdpr#tempdata-provider-and-session-state-cookies-arent-essential>.

> [!WARNING]
> Evitare di archiviare dati sensibili nello stato della sessione. L'utente potrebbe non chiudere il browser e deselezionare la sessione cookie . Alcuni browser gestiscono sessioni valide cookie tra le finestre del browser. Una sessione potrebbe non essere limitata a un singolo utente. L'utente successivo potrebbe continuare a esplorare l'app con la stessa sessione cookie .

Il provider di cache in memoria archivia i dati della sessione nella memoria del server in cui si trova l'app. In uno scenario di server farm:

* Usare *sessioni permanenti* per associare ogni sessione a un'istanza specifica dell'app in un singolo server. Il [Servizio app di Azure](https://azure.microsoft.com/services/app-service/) usa il modulo [Application Request Routing (ARR)](/iis/extensions/planning-for-arr/using-the-application-request-routing-module) per applicare le sessioni permanenti per impostazione predefinita. Tuttavia le sessioni permanenti possono compromettere la scalabilità e complicare gli aggiornamenti delle app Web. Un approccio migliore è usare la cache distribuita Redis o SQL Server, che non richiede sessioni permanenti. Per altre informazioni, vedere <xref:performance/caching/distributed>.
* La sessione cookie è crittografata tramite <xref:Microsoft.AspNetCore.DataProtection.IDataProtector> . La protezione dei dati deve essere configurata correttamente per leggere la sessione cookie s in ogni computer. Per altre informazioni, vedere <xref:security/data-protection/introduction> e [Provider di archiviazione chiavi](xref:security/data-protection/implementation/key-storage-providers).

### <a name="configure-session-state"></a>Configurare lo stato della sessione

Il pacchetto [Microsoft. AspNetCore. Session](https://www.nuget.org/packages/Microsoft.AspNetCore.Session/) :

* È incluso in modo implicito dal Framework.
* Fornisce il middleware per la gestione dello stato della sessione.

Per abilitare il middleware della sessione, `Startup` deve contenere:

* Qualsiasi cache di <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> memoria. L'implementazione `IDistributedCache` viene usata come archivio di backup per la sessione. Per altre informazioni, vedere <xref:performance/caching/distributed>.
* Una chiamata a <xref:Microsoft.Extensions.DependencyInjection.SessionServiceCollectionExtensions.AddSession%2A> in `ConfigureServices` .
* Una chiamata a <xref:Microsoft.AspNetCore.Builder.SessionMiddlewareExtensions.UseSession%2A> in `Configure` .

Il codice seguente indica come configurare il provider della sessione in memoria con un'implementazione in memoria predefinita di `IDistributedCache`:

[!code-csharp[](app-state/samples/3.x/SessionSample/Startup4.cs?name=snippet1&highlight=12-19,45)]

Il codice precedente imposta un breve timeout per semplificare i test.

L'ordine del middleware è importante.  Chiamare `UseSession` after `UseRouting` e before `UseEndpoints` . Vedere l' [ordine del middleware](xref:fundamentals/middleware/index#order).

[HttpContext. Session](xref:Microsoft.AspNetCore.Http.HttpContext.Session) è disponibile dopo che è stato configurato lo stato della sessione.

Non è possibile accedere a `HttpContext.Session` prima di chiamare `UseSession`.

Non è possibile creare una nuova sessione con una nuova sessione cookie dopo che l'app ha iniziato a scrivere nel flusso di risposta. L'eccezione viene registrata nel log del server Web e non è visualizzata nel browser.

### <a name="load-session-state-asynchronously"></a>Caricare lo stato della sessione in modo asincrono

Il provider di sessione predefinito in ASP.NET Core carica i record di sessione dall'archivio di backup sottostante in <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> modo asincrono solo se il <xref:Microsoft.AspNetCore.Http.ISession.LoadAsync%2A?displayProperty=nameWithType> metodo viene chiamato in modo esplicito prima dei <xref:Microsoft.AspNetCore.Http.ISession.TryGetValue%2A> <xref:Microsoft.AspNetCore.Http.ISession.Set%2A> metodi, o <xref:Microsoft.AspNetCore.Http.ISession.Remove%2A> . Se `LoadAsync` non viene chiamato per primo, il record di sessione sottostante viene caricato in modo sincrono e ciò può compromettere le prestazioni su larga scala.

Per consentire alle app di applicare questo modello, eseguire il wrapping delle <xref:Microsoft.AspNetCore.Session.DistributedSessionStore> <xref:Microsoft.AspNetCore.Session.DistributedSession> implementazioni e con le versioni che generano un'eccezione se il `LoadAsync` metodo non viene chiamato prima `TryGetValue` di, `Set` o `Remove` . Registrare le versioni con wrapping nel contenitore di servizi.

### <a name="session-options"></a>Opzioni di sessione

Per eseguire l'override delle impostazioni predefinite della sessione, usare <xref:Microsoft.AspNetCore.Builder.SessionOptions> .

| Opzione | Descrizione |
| ------ | ----------- |
| <xref:Microsoft.AspNetCore.Builder.SessionOptions.Cookie> | Determina le impostazioni utilizzate per creare l'oggetto cookie . <xref:Microsoft.AspNetCore.Http.CookieBuilder.Name> il valore predefinito è <xref:Microsoft.AspNetCore.Session.SessionDefaults.CookieName?displayProperty=nameWithType> ( `.AspNetCore.Session` ). <xref:Microsoft.AspNetCore.Http.CookieBuilder.Path> il valore predefinito è <xref:Microsoft.AspNetCore.Session.SessionDefaults.CookiePath?displayProperty=nameWithType> ( `/` ). <xref:Microsoft.AspNetCore.Http.CookieBuilder.SameSite> il valore predefinito è <xref:Microsoft.AspNetCore.Http.SameSiteMode.Lax?displayProperty=nameWithType> ( `1` ). <xref:Microsoft.AspNetCore.Http.CookieBuilder.HttpOnly> il valore predefinito è `true` . <xref:Microsoft.AspNetCore.Http.CookieBuilder.IsEssential> il valore predefinito è `false` . |
| <xref:Microsoft.AspNetCore.Builder.SessionOptions.IdleTimeout> | Il valore `IdleTimeout` indica quanto tempo la sessione può rimanere inattiva prima che il relativo contenuto venga abbandonato. Ogni accesso alla sessione reimposta il timeout. Questa impostazione si applica solo al contenuto della sessione, non a cookie . Il valore predefinito è 20 minuti. |
| <xref:Microsoft.AspNetCore.Builder.SessionOptions.IOTimeout> | L'intervallo di tempo massimo consentito per caricare una sessione dall'archivio o per eseguirne il commit nell'archivio. Questa impostazione può essere applicabile solo alle operazioni asincrone. Questo timeout può essere disabilitato utilizzando <xref:System.Threading.Timeout.InfiniteTimeSpan> . Il valore predefinito è 1 minuto. |

La sessione utilizza un cookie per rilevare e identificare le richieste da un singolo browser. Per impostazione predefinita, questo cookie nome è denominato `.AspNetCore.Session` e usa un percorso di `/` . Poiché il cookie valore predefinito non specifica un dominio, non viene reso disponibile per lo script sul lato client nella pagina (poiché il <xref:Microsoft.AspNetCore.Http.CookieBuilder.HttpOnly> valore predefinito è `true` ).

Per eseguire l'override delle cookie impostazioni predefinite della sessione, usare <xref:Microsoft.AspNetCore.Builder.SessionOptions> :

[!code-csharp[](app-state/samples/3.x/SessionSample/Startup2.cs?name=snippet1&highlight=5-10)]

L'app usa la <xref:Microsoft.AspNetCore.Builder.SessionOptions.IdleTimeout> proprietà per determinare per quanto tempo una sessione può rimanere inattiva prima che il relativo contenuto nella cache del server venga abbandonato. Questa proprietà è indipendente dalla cookie scadenza. Ogni richiesta che passa attraverso il [middleware di sessione](xref:Microsoft.AspNetCore.Session.SessionMiddleware) reimposta il timeout.

Lo stato della sessione è *non di blocco*. Se due richieste tentano simultaneamente di modificare il contenuto di una sessione, l'ultima richiesta sostituisce la prima. `Session` viene implementata come *sessione coerente*, ovvero tutto il contenuto viene archiviato insieme. Quando due richieste tentano di modificare diversi valori della sessione, l'ultima richiesta può sostituire le modifiche apportate alla sessione dalla prima.

### <a name="set-and-get-session-values"></a>Impostare e ottenere i valori della sessione

È possibile accedere allo stato della sessione da una classe Razor pages <xref:Microsoft.AspNetCore.Mvc.RazorPages.PageModel> o una <xref:Microsoft.AspNetCore.Mvc.Controller> classe MVC con <xref:Microsoft.AspNetCore.Http.HttpContext.Session?displayProperty=nameWithType> . Questa proprietà è un' <xref:Microsoft.AspNetCore.Http.ISession> implementazione di.

L'implementazione `ISession` offre diversi metodi di estensione per impostare e recuperare i valori interi e stringa. I metodi di estensione si trovano nello <xref:Microsoft.AspNetCore.Http> spazio dei nomi.

Metodi di estensione `ISession`:

* [Get(ISession, String)](xref:Microsoft.AspNetCore.Http.SessionExtensions.Get%2A)
* [GetInt32(ISession, String)](xref:Microsoft.AspNetCore.Http.SessionExtensions.GetInt32%2A)
* [GetString(ISession, String)](xref:Microsoft.AspNetCore.Http.SessionExtensions.GetString%2A)
* [SetInt32(ISession, String, Int32)](xref:Microsoft.AspNetCore.Http.SessionExtensions.SetInt32%2A)
* [SetString(ISession, String, String)](xref:Microsoft.AspNetCore.Http.SessionExtensions.SetString%2A)

L'esempio seguente recupera il valore della sessione per la `IndexModel.SessionKeyName` chiave ( `_Name` nell'app di esempio) in una Razor pagina di pagine:

```csharp
@page
@using Microsoft.AspNetCore.Http
@model IndexModel

...

Name: @HttpContext.Session.GetString(IndexModel.SessionKeyName)
```

L'esempio seguente illustra come impostare e ottenere un intero e una stringa:

[!code-csharp[](app-state/samples/3.x/SessionSample/Pages/Index.cshtml.cs?name=snippet1&highlight=18-19,22-23)]

Tutti i dati della sessione devono essere serializzati per abilitare uno scenario di cache distribuita, anche quando si usa la cache in memoria. I serializzatori stringa e integer sono forniti dai metodi di estensione di <xref:Microsoft.AspNetCore.Http.ISession> . I tipi complessi devono essere serializzati dall'utente usando un altro meccanismo, ad esempio JSON.

Usare il codice di esempio seguente per serializzare gli oggetti:

[!code-csharp[](app-state/samples/3.x/SessionSample/Extensions/SessionExtensions.cs?name=snippet1)]

Nell'esempio seguente viene illustrato come impostare e ottenere un oggetto serializzabile con la `SessionExtensions` classe:

[!code-csharp[](app-state/samples/3.x/SessionSample/Pages/Index.cshtml.cs?name=snippet2)]

## <a name="tempdata"></a>TempData

ASP.NET Core espone le Razor pagine [TempData](xref:Microsoft.AspNetCore.Mvc.RazorPages.PageModel.TempData) o controller <xref:Microsoft.AspNetCore.Mvc.Controller.TempData> . Questa proprietà archivia i dati finché non viene letta in un'altra richiesta. È possibile usare i metodi [Keep (String)](xref:Microsoft.AspNetCore.Mvc.ViewFeatures.ITempDataDictionary.Keep*) e [Peek (String)](xref:Microsoft.AspNetCore.Mvc.ViewFeatures.ITempDataDictionary.Peek*) per esaminare i dati senza eliminare alla fine della richiesta. [Mantenere](xref:Microsoft.AspNetCore.Mvc.ViewFeatures.ITempDataDictionary.Keep*) i contrassegni per la conservazione di tutti gli elementi nel dizionario. `TempData` è:

* Utile per il reindirizzamento quando i dati sono necessari per più di una singola richiesta.
* Implementata dai `TempData` provider utilizzando cookie lo stato della sessione o.

## <a name="tempdata-samples"></a>Esempi di TempData

Si consideri la seguente pagina che consente di creare un cliente:

[!code-csharp[](app-state/3.0samples/RazorPagesContacts/Pages/Customers/Create.cshtml.cs?name=snippet&highlight=15-16,30)]

Viene visualizzata la pagina seguente `TempData["Message"]` :

[!code-cshtml[](app-state/3.0samples/RazorPagesContacts/Pages/Customers/IndexPeek.cshtml?range=1-14)]

Nel markup precedente, alla fine della richiesta, `TempData["Message"]` **non** viene eliminato perché `Peek` viene usato. Con l'aggiornamento della pagina viene visualizzato il contenuto di `TempData["Message"]` .

Il markup seguente è simile al codice precedente, ma usa `Keep` per conservare i dati alla fine della richiesta:

[!code-cshtml[](app-state/3.0samples/RazorPagesContacts/Pages/Customers/IndexKeep.cshtml?range=1-14)]

L'esplorazione tra le pagine *IndexPeek* e *IndexKeep* non verrà eliminata `TempData["Message"]` .

Il codice seguente visualizza `TempData["Message"]` , ma alla fine della richiesta, `TempData["Message"]` viene eliminato:

[!code-cshtml[](app-state/3.0samples/RazorPagesContacts/Pages/Customers/Index.cshtml?range=1-14)]

### <a name="tempdata-providers"></a>Provider TempData

Il cookie provider TempData basato su viene usato per impostazione predefinita per archiviare TempData in cookie s.

I cookie dati vengono crittografati <xref:Microsoft.AspNetCore.DataProtection.IDataProtector> con, codificati con <xref:Microsoft.AspNetCore.WebUtilities.Base64UrlTextEncoder> , quindi suddivisi in blocchi. La cookie dimensione massima è inferiore a [4096 byte](http://www.faqs.org/rfcs/rfc2965.html) a causa della crittografia e della suddivisione in blocchi. I cookie dati non vengono compressi perché la compressione di dati crittografati può causare problemi di sicurezza, ad esempio gli attacchi di [reato](https://wikipedia.org/wiki/CRIME_(security_exploit)) e [violazione](https://wikipedia.org/wiki/BREACH_(security_exploit)) . Per ulteriori informazioni sul cookie provider TempData basato su, vedere <xref:Microsoft.AspNetCore.Mvc.ViewFeatures.CookieTempDataProvider> .

### <a name="choose-a-tempdata-provider"></a>Scegliere un provider TempData

La scelta di un provider TempData implica diverse considerazioni, tra cui:

* L'app utilizza già lo stato sessione? In tal caso, l'uso del provider TempData dello stato della sessione non comporta costi aggiuntivi per l'app oltre le dimensioni dei dati.
* L'app usa TempData solo sporadicamente per quantità di dati relativamente ridotte, fino a 500 byte? In tal caso, il cookie provider TempData aggiunge un costo ridotto a ogni richiesta che contiene TempData. Se no, il provider TempData con stato sessione può essere utile per evitare sequenze di andata e ritorno di grandi quantità di dati in ogni richiesta fino a quando il contenuto TempData non viene consumato.
* L'app viene eseguita in un server farm in più server? In tal caso, non è necessaria alcuna configurazione aggiuntiva per usare il cookie provider TempData al di fuori della protezione dei dati (vedere <xref:security/data-protection/introduction> e [provider di archiviazione chiavi](xref:security/data-protection/implementation/key-storage-providers)).

La maggior parte dei client Web, ad esempio i Web browser, applica limiti alla dimensione massima di ogni cookie e al numero totale di cookie . Quando si usa il cookie provider TempData, verificare che l'app non superi [questi limiti](http://www.faqs.org/rfcs/rfc2965.html). Considerare la dimensione totale dei dati. Conto per un aumento delle cookie dimensioni a causa della crittografia e della suddivisione in blocchi.

### <a name="configure-the-tempdata-provider"></a>Configurare il provider TempData

Il cookie provider TempData basato su è abilitato per impostazione predefinita.

Per abilitare il provider TempData basato sulla sessione, usare il <xref:Microsoft.Extensions.DependencyInjection.MvcViewFeaturesMvcBuilderExtensions.AddSessionStateTempDataProvider%2A> metodo di estensione. È necessaria una sola chiamata a `AddSessionStateTempDataProvider` :

[!code-csharp[](app-state/samples/3.x/SessionSample/Startup3.cs?name=snippet1&highlight=4,6,8,30)]

## <a name="query-strings"></a>Stringhe di query

È possibile passare una quantità limitata di dati da una richiesta a un'altra aggiungendo i dati alla stringa di query della nuova richiesta. Questo è utile per l'acquisizione dello stato con una modalità persistente, che consente la condivisione dei collegamenti con stato incorporato tramite posta elettronica o social network. Poiché le stringhe di query dell'URL sono pubbliche, non usare mai le stringhe di query per i dati sensibili.

Oltre alla condivisione non intenzionale, i dati nelle stringhe di query possono esporre l'app a attacchi di [richiesta intersito falsa (CSRF)](https://www.owasp.org/index.php/Cross-Site_Request_Forgery_(CSRF)) . Qualsiasi stato della sessione mantenuto deve proteggersi dagli attacchi CSRF. Per altre informazioni, vedere <xref:security/anti-request-forgery>.

## <a name="hidden-fields"></a>Campi nascosti

I dati possono essere salvati in campi modulo nascosti e pubblicati di nuovo nella richiesta successiva. Questo si verifica spesso nei moduli a più pagine. Poiché il client potenzialmente può alterare i dati, l'app deve sempre ripetere la convalida dei dati archiviati nei campi nascosti.

## <a name="httpcontextitems"></a>HttpContext.Items

La <xref:Microsoft.AspNetCore.Http.HttpContext.Items?displayProperty=nameWithType> raccolta viene usata per archiviare i dati durante l'elaborazione di una singola richiesta. Il contenuto della raccolta viene rimosso al termine dell'elaborazione della richiesta. La raccolta `Items` spesso viene usata per consentire ai componenti o al middleware di comunicare quando operano in momenti diversi durante una richiesta e non è disponibile un metodo diretto per passare i parametri.

Nell'esempio seguente il [middleware](xref:fundamentals/middleware/index) aggiunge `isVerified` alla `Items` raccolta:

[!code-csharp[](app-state/samples/3.x/SessionSample/Startup.cs?name=snippet1)]

Per il middleware usato solo in una singola app, le `string` chiavi fisse sono accettabili. Il middleware condiviso tra le app deve usare chiavi di oggetti univoche per evitare conflitti di chiave. L'esempio seguente illustra come usare una chiave oggetto univoca definita in una classe middleware:

[!code-csharp[](app-state/samples/3.x/SessionSample/Middleware/HttpContextItemsMiddleware.cs?name=snippet1&highlight=4,13)]

Altri elementi di codice possono accedere al valore archiviato in `HttpContext.Items` usando la chiave esposta dalla classe middleware:

[!code-csharp[](app-state/samples/3.x/SessionSample/Pages/Index.cshtml.cs?name=snippet3)]

Questo approccio ha anche il vantaggio di eliminare l'uso di stringhe chiave nel codice.

## <a name="cache"></a>Cache

La memorizzazione nella cache è un modo efficiente per archiviare e recuperare dati. L'app può controllare la durata degli elementi della cache. Per altre informazioni, vedere <xref:performance/caching/response>.

I dati memorizzati nella cache non sono associati a una richiesta, un utente o una sessione specifici. **Non memorizzare nella cache i dati specifici dell'utente che possono essere recuperati da altre richieste utente.**

Per memorizzare nella cache i dati a livello di applicazione, vedere <xref:performance/caching/memory> .

## <a name="common-errors"></a>Errori comuni

* "Unable to resolve service for type 'Microsoft.Extensions.Caching.Distributed.IDistributedCache' while attempting to activate 'Microsoft.AspNetCore.Session.DistributedSessionStore'." (Risoluzione servizio non riuscita per il tipo 'Microsoft.Extensions.Caching.Distributed.IDistributedCache' durante il tentativo di attivazione di 'Microsoft.AspNetCore.Session.DistributedSessionStore').

  Questo problema si verifica in genere quando si verifica un errore durante la configurazione di almeno un' `IDistributedCache` implementazione. Per altre informazioni, vedere <xref:performance/caching/distributed> e <xref:performance/caching/memory>.

Se il middleware della sessione non è in grado di salvare in modo permanente una sessione:

* Il middleware registra l'eccezione e la richiesta continua normalmente.
* Ciò provoca un comportamento imprevedibile.

Il middleware della sessione può non riuscire a salvare in modo permanente una sessione se l'archivio di backup non è disponibile. Ad esempio, un utente archivia un carrello acquisti nella sessione. L'utente aggiunge un articolo al carrello ma il commit ha esito negativo. L'app non riconosce l'errore e segnala all'utente che l'articolo è stato aggiunto al carrello anche se non è vero.

L'approccio consigliato per verificare la presenza di errori consiste nel chiamare `await feature.Session.CommitAsync` quando l'app esegue la scrittura nella sessione. <xref:Microsoft.AspNetCore.Http.ISession.CommitAsync*> genera un'eccezione se l'archivio di backup non è disponibile. Se `CommitAsync` ha esito negativo, l'app è in grado di elaborare l'eccezione. <xref:Microsoft.AspNetCore.Http.ISession.LoadAsync*> genera un'eccezione quando l'archivio dati non è disponibile.
  
## <a name="signalr-and-session-state"></a>SignalR e stato sessione

SignalR le app non devono usare lo stato della sessione per archiviare le informazioni. SignalR le app possono archiviare lo stato per connessione in `Context.Items` nell'hub. <!-- https://github.com/aspnet/SignalR/issues/2139 -->

## <a name="additional-resources"></a>Risorse aggiuntive

<xref:host-and-deploy/web-farm>
::: moniker-end

::: moniker range="< aspnetcore-3.0"

Di [Rick Anderson](https://twitter.com/RickAndMSFT), [Steve Smith](https://ardalis.com/), [Diana LaRose](https://github.com/DianaLaRose) e [Luke Latham](https://github.com/guardrex)

HTTP è un protocollo senza stato. Senza eseguire ulteriori passaggi, le richieste HTTP sono messaggi indipendenti che non mantengono i valori dell'utente o lo stato delle app. In questo articolo vengono descritti diversi approcci per mantenere lo stato di app e dati utente tra le richieste.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/app-state/samples) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="state-management"></a>Gestione dello stato

Lo stato può essere archiviato usando diversi approcci. Ogni approccio è descritto di seguito in questo argomento.

| Approccio con risorsa di archiviazione | Meccanismo di archiviazione |
| ---------------- | ----------------- |
| [Cookies](#cookies) | HTTP cookie s (può includere dati archiviati con il codice dell'app sul lato server) |
| [Stato sessione](#session-state) | cookieCodice dell'app http s e lato server |
| [TempData](#tempdata) | cookieStato http s o sessione |
| [Stringhe di query](#query-strings) | Stringhe di query HTTP |
| [Campi nascosti](#hidden-fields) | Campi dei form HTTP |
| [HttpContext.Items](#httpcontextitems) | Codice app lato server |
| [Cache](#cache) | Codice app lato server |
| [Inserimento delle dipendenze](#dependency-injection) | Codice app lato server |

## <a name="cookies"></a>Cookies

CookieArchivia i dati tra le richieste. Poiché cookie le istanze di vengono inviate con ogni richiesta, la loro dimensione deve essere mantenuta al minimo. Idealmente, solo un identificatore deve essere archiviato in un cookie con i dati archiviati dall'app. La maggior parte dei browser limita le cookie dimensioni a 4096 byte. cookiePer ogni dominio sono disponibili solo un numero limitato di s.

Poiché cookie le s sono soggette a manomissioni, devono essere convalidate dall'app. Cookiei possono essere eliminati dagli utenti e scadono nei client. Tuttavia, i cookie sono in genere la forma più durevole di persistenza dei dati nel client.

Cookiei vengono spesso usati per la personalizzazione, in cui il contenuto è personalizzato per un utente noto. L'utente viene solo identificato e non autenticato nella maggior parte dei casi. cookiePuò archiviare il nome dell'utente, il nome dell'account o l'ID utente univoco, ad esempio un GUID. È quindi possibile usare cookie per accedere alle impostazioni personalizzate dell'utente, ad esempio il colore di sfondo del sito Web preferito.

Tenere in considerazione le [normative sulla protezione dei dati (GDPR) dell'Unione europea](https://ec.europa.eu/info/law/law-topic/data-protection) quando si rilasciano cookie e si gestiscono i problemi di privacy. Per altre informazioni, vedere il [supporto per il Regolamento generale sulla protezione dei dati in ASP.NET Core](xref:security/gdpr).

## <a name="session-state"></a>Stato sessione

Lo stato della sessione è uno scenario di ASP.NET Core per l'archiviazione dei dati utente mentre l'utente visualizza un'app Web. Lo stato della sessione usa un archivio gestito dall'app per rendere persistenti i dati tra le richieste provenienti da un client. I dati della sessione sono supportati da una cache e considerati dati temporanei: il sito deve continuare a funzionare senza i dati della sessione. I dati critici dell'applicazione devono essere archiviati nel database utente e memorizzati nella cache della sessione solo al fine di ottimizzare le prestazioni.

> [!NOTE]
> La sessione non è supportata nelle [SignalR](xref:signalr/index) app perché un [ SignalR Hub](xref:signalr/hubs) può essere eseguito indipendentemente da un contesto http. Ad esempio, ciò può verificarsi quando una lunga richiesta di polling viene mantenuta aperta da un hub oltre la durata del contesto HTTP della richiesta.

ASP.NET Core gestisce lo stato della sessione fornendo un cookie al client che contiene un ID di sessione, che viene inviato all'app a ogni richiesta. L'app usa l'ID sessione per recuperare i dati della sessione.

Lo stato della sessione presenta i comportamenti seguenti:

* Poiché la sessione cookie è specifica del browser, le sessioni non vengono condivise tra più browser.
* cookieLa sessione s viene eliminata al termine della sessione del browser.
* Se cookie per una sessione scaduta viene ricevuto un oggetto, viene creata una nuova sessione che utilizza la stessa sessione cookie .
* Le sessioni vuote non vengono conservate: nella sessione deve essere impostato almeno un valore perché la sessione diventi permanente tra le richieste. Se una sessione non viene conservata, viene generato un nuovo ID sessione per ogni nuova richiesta.
* L'app conserva una sessione per un periodo di tempo limitato dopo l'ultima richiesta. L'app imposta il timeout della sessione o usa il valore predefinito, pari a 20 minuti. Lo stato sessione è ideale per archiviare i dati utente specifici per una determinata sessione, ma nel caso in cui i dati non richiedano un'archiviazione permanente tra le sessioni.
* I dati della sessione vengono eliminati quando <xref:Microsoft.AspNetCore.Http.ISession.Clear%2A?displayProperty=nameWithType> viene chiamata l'implementazione o quando la sessione scade.
* Non esiste un meccanismo predefinito per informare il codice dell'app che un browser client è stato chiuso o quando la sessione cookie è stata eliminata o scaduta nel client.
* I modelli ASP.NET Core MVC e Razor pages includono il supporto per regolamento generale sulla protezione dei dati (GDPR). Per impostazione predefinita, lo stato della sessione non cookie è contrassegnato come Essential, quindi lo stato della sessione non è funzionante, a meno che non sia consentito il rilevamento del sito. Per altre informazioni, vedere <xref:security/gdpr#tempdata-provider-and-session-state-cookies-arent-essential>.

> [!WARNING]
> Evitare di archiviare dati sensibili nello stato della sessione. L'utente potrebbe non chiudere il browser e deselezionare la sessione cookie . Alcuni browser gestiscono sessioni valide cookie tra le finestre del browser. Una sessione potrebbe non essere limitata a un singolo utente &mdash; . l'utente successivo potrebbe continuare a esplorare l'app con la stessa sessione cookie .

Il provider di cache in memoria archivia i dati della sessione nella memoria del server in cui si trova l'app. In uno scenario di server farm:

* Usare *sessioni permanenti* per associare ogni sessione a un'istanza specifica dell'app in un singolo server. Il [Servizio app di Azure](https://azure.microsoft.com/services/app-service/) usa il modulo [Application Request Routing (ARR)](/iis/extensions/planning-for-arr/using-the-application-request-routing-module) per applicare le sessioni permanenti per impostazione predefinita. Tuttavia le sessioni permanenti possono compromettere la scalabilità e complicare gli aggiornamenti delle app Web. Un approccio migliore è usare la cache distribuita Redis o SQL Server, che non richiede sessioni permanenti. Per altre informazioni, vedere <xref:performance/caching/distributed>.
* La sessione cookie è crittografata tramite <xref:Microsoft.AspNetCore.DataProtection.IDataProtector> . La protezione dei dati deve essere configurata correttamente per leggere la sessione cookie s in ogni computer. Per altre informazioni, vedere <xref:security/data-protection/introduction> e [Provider di archiviazione chiavi](xref:security/data-protection/implementation/key-storage-providers).

### <a name="configure-session-state"></a>Configurare lo stato della sessione

Con il pacchetto [Microsoft.AspNetCore.Session](https://www.nuget.org/packages/Microsoft.AspNetCore.Session/), incluso nel [metapacchetto Microsoft.AspNetCore.App](xref:fundamentals/metapackage-app), è disponibile il middleware per la gestione dello stato della sessione. Per abilitare il middleware della sessione, `Startup` deve contenere:

* Qualsiasi cache di <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> memoria. L'implementazione `IDistributedCache` viene usata come archivio di backup per la sessione. Per altre informazioni, vedere <xref:performance/caching/distributed>.
* Una chiamata a <xref:Microsoft.Extensions.DependencyInjection.SessionServiceCollectionExtensions.AddSession%2A> in `ConfigureServices` .
* Una chiamata a <xref:Microsoft.AspNetCore.Builder.SessionMiddlewareExtensions.UseSession%2A> in `Configure` .

Il codice seguente indica come configurare il provider della sessione in memoria con un'implementazione in memoria predefinita di `IDistributedCache`:

[!code-csharp[](app-state/samples/2.x/SessionSample/Startup.cs?name=snippet1&highlight=5-14,34)]

L'ordine del middleware è importante. Nell'esempio precedente si verifica un'eccezione `InvalidOperationException` se `UseSession` viene chiamata dopo `UseMvc`. Per altre informazioni, vedere la sezione relativa all'[ordine del middleware](xref:fundamentals/middleware/index#order).

<xref:Microsoft.AspNetCore.Http.HttpContext.Session?displayProperty=nameWithType> è disponibile dopo la configurazione dello stato della sessione.

Non è possibile accedere a `HttpContext.Session` prima di chiamare `UseSession`.

Non è possibile creare una nuova sessione con una nuova sessione cookie dopo che l'app ha iniziato a scrivere nel flusso di risposta. L'eccezione viene registrata nel log del server Web e non è visualizzata nel browser.

### <a name="load-session-state-asynchronously"></a>Caricare lo stato della sessione in modo asincrono

Il provider di sessione predefinito in ASP.NET Core carica i record di sessione dall'archivio di backup sottostante in <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> modo asincrono solo se il <xref:Microsoft.AspNetCore.Http.ISession.LoadAsync%2A?displayProperty=nameWithType> metodo viene chiamato in modo esplicito prima dei <xref:Microsoft.AspNetCore.Http.ISession.TryGetValue%2A> <xref:Microsoft.AspNetCore.Http.ISession.Set%2A> metodi, o <xref:Microsoft.AspNetCore.Http.ISession.Remove%2A> . Se `LoadAsync` non viene chiamato per primo, il record di sessione sottostante viene caricato in modo sincrono e ciò può compromettere le prestazioni su larga scala.

Per consentire alle app di applicare questo modello, eseguire il wrapping delle <xref:Microsoft.AspNetCore.Session.DistributedSessionStore> <xref:Microsoft.AspNetCore.Session.DistributedSession> implementazioni e con le versioni che generano un'eccezione se il `LoadAsync` metodo non viene chiamato prima `TryGetValue` di, `Set` o `Remove` . Registrare le versioni con wrapping nel contenitore di servizi.

### <a name="session-options"></a>Opzioni di sessione

Per eseguire l'override delle impostazioni predefinite della sessione, usare <xref:Microsoft.AspNetCore.Builder.SessionOptions> .

| Opzione | Descrizione |
| ------ | ----------- |
| <xref:Microsoft.AspNetCore.Builder.SessionOptions.Cookie> | Determina le impostazioni utilizzate per creare l'oggetto cookie . <xref:Microsoft.AspNetCore.Http.CookieBuilder.Name> il valore predefinito è <xref:Microsoft.AspNetCore.Session.SessionDefaults.CookieName?displayProperty=nameWithType> ( `.AspNetCore.Session` ). <xref:Microsoft.AspNetCore.Http.CookieBuilder.Path> il valore predefinito è <xref:Microsoft.AspNetCore.Session.SessionDefaults.CookiePath?displayProperty=nameWithType> ( `/` ). <xref:Microsoft.AspNetCore.Http.CookieBuilder.SameSite> il valore predefinito è <xref:Microsoft.AspNetCore.Http.SameSiteMode.Lax?displayProperty=nameWithType> ( `1` ). <xref:Microsoft.AspNetCore.Http.CookieBuilder.HttpOnly> il valore predefinito è `true` . <xref:Microsoft.AspNetCore.Http.CookieBuilder.IsEssential> il valore predefinito è `false` . |
| <xref:Microsoft.AspNetCore.Builder.SessionOptions.IdleTimeout> | Il valore `IdleTimeout` indica quanto tempo la sessione può rimanere inattiva prima che il relativo contenuto venga abbandonato. Ogni accesso alla sessione reimposta il timeout. Questa impostazione si applica solo al contenuto della sessione, non a cookie . Il valore predefinito è 20 minuti. |
| <xref:Microsoft.AspNetCore.Builder.SessionOptions.IOTimeout> | L'intervallo di tempo massimo consentito per caricare una sessione dall'archivio o per eseguirne il commit nell'archivio. Questa impostazione può essere applicabile solo alle operazioni asincrone. Questo timeout può essere disabilitato utilizzando <xref:System.Threading.Timeout.InfiniteTimeSpan> . Il valore predefinito è 1 minuto. |

La sessione utilizza un cookie per rilevare e identificare le richieste da un singolo browser. Per impostazione predefinita, questo cookie nome è denominato `.AspNetCore.Session` e usa un percorso di `/` . Poiché il cookie valore predefinito non specifica un dominio, non viene reso disponibile per lo script sul lato client nella pagina (poiché il <xref:Microsoft.AspNetCore.Http.CookieBuilder.HttpOnly> valore predefinito è `true` ).

Per eseguire l'override delle cookie impostazioni predefinite della sessione, usare `SessionOptions` :

[!code-csharp[](app-state/samples_snapshot/2.x/SessionSample/Startup.cs?name=snippet1&highlight=14-19)]

L'app usa la <xref:Microsoft.AspNetCore.Builder.SessionOptions.IdleTimeout> proprietà per determinare per quanto tempo una sessione può rimanere inattiva prima che il relativo contenuto nella cache del server venga abbandonato. Questa proprietà è indipendente dalla cookie scadenza. Ogni richiesta che passa attraverso il [middleware di sessione](xref:Microsoft.AspNetCore.Session.SessionMiddleware) reimposta il timeout.

Lo stato della sessione è *non di blocco*. Se due richieste tentano simultaneamente di modificare il contenuto di una sessione, l'ultima richiesta sostituisce la prima. `Session` viene implementata come *sessione coerente*, ovvero tutto il contenuto viene archiviato insieme. Quando due richieste tentano di modificare diversi valori della sessione, l'ultima richiesta può sostituire le modifiche apportate alla sessione dalla prima.

### <a name="set-and-get-session-values"></a>Impostare e ottenere i valori della sessione

È possibile accedere allo stato della sessione da una classe Razor pages <xref:Microsoft.AspNetCore.Mvc.RazorPages.PageModel> o una <xref:Microsoft.AspNetCore.Mvc.Controller> classe MVC con <xref:Microsoft.AspNetCore.Http.HttpContext.Session?displayProperty=nameWithType> . Questa proprietà è un' <xref:Microsoft.AspNetCore.Http.ISession> implementazione di.

L'implementazione `ISession` offre diversi metodi di estensione per impostare e recuperare i valori interi e stringa. I metodi di estensione si trovano nello <xref:Microsoft.AspNetCore.Http> spazio dei nomi (aggiungere un' `using Microsoft.AspNetCore.Http;` istruzione per accedere ai metodi di estensione) quando il progetto fa riferimento al pacchetto [Microsoft. AspNetCore. http. Extensions](https://www.nuget.org/packages/Microsoft.AspNetCore.Http.Extensions/) . Entrambi i pacchetti sono inclusi nel [metapacchetto Microsoft.AspNetCore.App](xref:fundamentals/metapackage-app).

Metodi di estensione `ISession`:

* [Get(ISession, String)](xref:Microsoft.AspNetCore.Http.SessionExtensions.Get%2A)
* [GetInt32(ISession, String)](xref:Microsoft.AspNetCore.Http.SessionExtensions.GetInt32%2A)
* [GetString(ISession, String)](xref:Microsoft.AspNetCore.Http.SessionExtensions.GetString%2A)
* [SetInt32(ISession, String, Int32)](xref:Microsoft.AspNetCore.Http.SessionExtensions.SetInt32%2A)
* [SetString(ISession, String, String)](xref:Microsoft.AspNetCore.Http.SessionExtensions.SetString%2A)

L'esempio seguente recupera il valore della sessione per la `IndexModel.SessionKeyName` chiave ( `_Name` nell'app di esempio) in una Razor pagina di pagine:

```csharp
@page
@using Microsoft.AspNetCore.Http
@model IndexModel

...

Name: @HttpContext.Session.GetString(IndexModel.SessionKeyName)
```

L'esempio seguente illustra come impostare e ottenere un intero e una stringa:

[!code-csharp[](app-state/samples/2.x/SessionSample/Pages/Index.cshtml.cs?name=snippet1&highlight=18-19,22-23)]

Tutti i dati della sessione devono essere serializzati per abilitare uno scenario di cache distribuita, anche quando si usa la cache in memoria. I serializzatori stringa e integer sono forniti dai metodi di estensione di <xref:Microsoft.AspNetCore.Http.ISession> ). I tipi complessi devono essere serializzati dall'utente usando un altro meccanismo, ad esempio JSON.

Aggiungere i seguenti metodi di estensione per impostare e ottenere oggetti serializzabili:

[!code-csharp[](app-state/samples/2.x/SessionSample/Extensions/SessionExtensions.cs?name=snippet1)]

L'esempio seguente illustra come impostare e ottenere un oggetto serializzabile con i metodi di estensione:

[!code-csharp[](app-state/samples/2.x/SessionSample/Pages/Index.cshtml.cs?name=snippet2)]

## <a name="tempdata"></a>TempData

ASP.NET Core espone le Razor pagine [TempData](xref:Microsoft.AspNetCore.Mvc.RazorPages.PageModel.TempData) o controller <xref:Microsoft.AspNetCore.Mvc.Controller.TempData> . Questa proprietà archivia i dati finché non viene letta in un'altra richiesta. È possibile utilizzare i metodi [Keep (String)](xref:Microsoft.AspNetCore.Mvc.ViewFeatures.ITempDataDictionary.Keep*) e [Peek (String)](xref:Microsoft.AspNetCore.Mvc.ViewFeatures.ITempDataDictionary.Peek*) per esaminare i dati senza eliminarli alla fine della richiesta. [Keep ()](xref:Microsoft.AspNetCore.Mvc.ViewFeatures.ITempDataDictionary.Keep*) contrassegna tutti gli elementi nel dizionario per la memorizzazione. `TempData` è particolarmente utile per il reindirizzamento quando i dati sono necessari per più di una singola richiesta. `TempData` viene implementato dai `TempData` provider che usano cookie o lo stato della sessione.

## <a name="tempdata-samples"></a>Esempi di TempData

Si consideri la seguente pagina che consente di creare un cliente:

[!code-csharp[](app-state/3.0samples/RazorPagesContacts/Pages/Customers/Create.cshtml.cs?name=snippet&highlight=15-16,30)]

Viene visualizzata la pagina seguente `TempData["Message"]` :

[!code-cshtml[](app-state/3.0samples/RazorPagesContacts/Pages/Customers/IndexPeek.cshtml?range=1-14)]

Nel markup precedente, alla fine della richiesta, `TempData["Message"]` **non** viene eliminato perché `Peek` viene usato. Verrà visualizzato l'aggiornamento della pagina `TempData["Message"]` .

Il markup seguente è simile al codice precedente, ma usa `Keep` per conservare i dati alla fine della richiesta:

[!code-cshtml[](app-state/3.0samples/RazorPagesContacts/Pages/Customers/IndexKeep.cshtml?range=1-14)]

L'esplorazione tra le pagine *IndexPeek* e *IndexKeep* non verrà eliminata `TempData["Message"]` .

Il codice seguente visualizza `TempData["Message"]` , ma alla fine della richiesta, `TempData["Message"]` viene eliminato:

[!code-cshtml[](app-state/3.0samples/RazorPagesContacts/Pages/Customers/Index.cshtml?range=1-14)]

### <a name="tempdata-providers"></a>Provider TempData

Il cookie provider TempData basato su viene usato per impostazione predefinita per archiviare TempData in cookie s.

I cookie dati vengono crittografati <xref:Microsoft.AspNetCore.DataProtection.IDataProtector> con, codificati con <xref:Microsoft.AspNetCore.WebUtilities.Base64UrlTextEncoder> , quindi suddivisi in blocchi. Poiché cookie è suddiviso in blocchi, il cookie limite di dimensioni singole trovato in ASP.NET Core 1. x non è applicabile. I cookie dati non vengono compressi perché la compressione di dati crittografati può causare problemi di sicurezza, ad esempio gli attacchi di [reato](https://wikipedia.org/wiki/CRIME_(security_exploit)) e [violazione](https://wikipedia.org/wiki/BREACH_(security_exploit)) . Per ulteriori informazioni sul cookie provider TempData basato su, vedere <xref:Microsoft.AspNetCore.Mvc.ViewFeatures.CookieTempDataProvider> .

### <a name="choose-a-tempdata-provider"></a>Scegliere un provider TempData

La scelta di un provider TempData implica diverse considerazioni, tra cui:

1. L'app utilizza già lo stato sessione? Se sì, l'uso del provider TempData con stato sessione non comporta un carico aggiuntivo per l'app (indipendentemente dalla dimensione dei dati).
2. L'app usa TempData solo quando necessario e per quantità di dati relativamente piccole (fino a 500 byte)? In tal caso, il cookie provider TempData aggiunge un costo ridotto a ogni richiesta che contiene TempData. Se no, il provider TempData con stato sessione può essere utile per evitare sequenze di andata e ritorno di grandi quantità di dati in ogni richiesta fino a quando il contenuto TempData non viene consumato.
3. L'app viene eseguita in un server farm in più server? In tal caso, non è necessaria alcuna configurazione aggiuntiva per usare il cookie provider TempData al di fuori della protezione dei dati (vedere <xref:security/data-protection/introduction> e [provider di archiviazione chiavi](xref:security/data-protection/implementation/key-storage-providers)).

> [!NOTE]
> La maggior parte dei client Web, ad esempio i Web browser, applica limiti per le dimensioni massime di ogni cookie , il numero totale di cookie s o entrambi. Quando si usa il cookie provider TempData, verificare che l'app non superi questi limiti. Considerare la dimensione totale dei dati. Conto per un aumento delle cookie dimensioni a causa della crittografia e della suddivisione in blocchi.

### <a name="configure-the-tempdata-provider"></a>Configurare il provider TempData

Il cookie provider TempData basato su è abilitato per impostazione predefinita.

Per abilitare il provider TempData basato sulla sessione, usare il <xref:Microsoft.Extensions.DependencyInjection.MvcViewFeaturesMvcBuilderExtensions.AddSessionStateTempDataProvider%2A> metodo di estensione:

[!code-csharp[](app-state/samples_snapshot_2/2.x/SessionSample/Startup.cs?name=snippet1&highlight=11,13,32)]

L'ordine del middleware è importante. Nell'esempio precedente si verifica un'eccezione `InvalidOperationException` se `UseSession` viene chiamata dopo `UseMvc`. Per altre informazioni, vedere la sezione relativa all'[ordine del middleware](xref:fundamentals/middleware/index#order).

> [!IMPORTANT]
> Se la destinazione è .NET Framework e si usa il provider TempData basato sulla sessione, aggiungere il pacchetto [Microsoft.AspNetCore.Session](https://www.nuget.org/packages/Microsoft.AspNetCore.Session/) al progetto.

## <a name="query-strings"></a>Stringhe di query

È possibile passare una quantità limitata di dati da una richiesta a un'altra aggiungendo i dati alla stringa di query della nuova richiesta. Questo è utile per l'acquisizione dello stato con una modalità persistente, che consente la condivisione dei collegamenti con stato incorporato tramite posta elettronica o social network. Poiché le stringhe di query dell'URL sono pubbliche, non usare mai le stringhe di query per i dati sensibili.

OItre alle condivisioni involontarie, i dati presenti nelle stringhe di query possono essere oggetto di attacchi di [falsificazione della richiesta tra siti (CSRF, Cross-Site Request Forgery)](https://www.owasp.org/index.php/Cross-Site_Request_Forgery_(CSRF)), che ingannano gli utenti autenticati invitandoli a visitare siti pericolosi. Gli utenti malintenzionati possono quindi sottrarre dati utente dall'app o eseguire azioni dannose per conto dell'utente. Qualsiasi stato dell'app o della sessione mantenuto deve garantire la protezione dagli attacchi CSRF. Per altre informazioni, vedere <xref:security/anti-request-forgery>.

## <a name="hidden-fields"></a>Campi nascosti

I dati possono essere salvati in campi modulo nascosti e pubblicati di nuovo nella richiesta successiva. Questo si verifica spesso nei moduli a più pagine. Poiché il client potenzialmente può alterare i dati, l'app deve sempre ripetere la convalida dei dati archiviati nei campi nascosti.

## <a name="httpcontextitems"></a>HttpContext.Items

La <xref:Microsoft.AspNetCore.Http.HttpContext.Items?displayProperty=nameWithType> raccolta viene usata per archiviare i dati durante l'elaborazione di una singola richiesta. Il contenuto della raccolta viene rimosso al termine dell'elaborazione della richiesta. La raccolta `Items` spesso viene usata per consentire ai componenti o al middleware di comunicare quando operano in momenti diversi durante una richiesta e non è disponibile un metodo diretto per passare i parametri.

Nell'esempio seguente [middleware](xref:fundamentals/middleware/index) aggiunge `isVerified` alla raccolta `Items`.

```csharp
app.Use(async (context, next) =>
{
    // perform some verification
    context.Items["isVerified"] = true;
    await next.Invoke();
});
```

In punto successivo della pipeline un altro middleware può accedere al valore di `isVerified`:

```csharp
app.Run(async (context) =>
{
    await context.Response.WriteAsync($"Verified: {context.Items["isVerified"]}");
});
```

Per il middleware che viene usato da un'unica app sono accettabili le chiavi `string`. Il middleware condiviso tra le istanze dell'app deve usare chiavi oggetto univoche per evitare conflitti di chiavi. L'esempio seguente illustra come usare una chiave oggetto univoca definita in una classe middleware:

[!code-csharp[](app-state/samples/2.x/SessionSample/Middleware/HttpContextItemsMiddleware.cs?name=snippet1&highlight=4,13)]

Altri elementi di codice possono accedere al valore archiviato in `HttpContext.Items` usando la chiave esposta dalla classe middleware:

[!code-csharp[](app-state/samples/2.x/SessionSample/Pages/Index.cshtml.cs?name=snippet3)]

Questo approccio ha anche il vantaggio di eliminare l'uso di stringhe chiave nel codice.

## <a name="cache"></a>Cache

La memorizzazione nella cache è un modo efficiente per archiviare e recuperare dati. L'app può controllare la durata degli elementi della cache.

I dati memorizzati nella cache non sono associati a una richiesta, un utente o una sessione specifici. **Prestare attenzione a non memorizzare nella cache i dati specifici dell'utente che possono essere recuperati dalle richieste di altri utenti.**

Per altre informazioni, vedere <xref:performance/caching/response>.

## <a name="dependency-injection"></a>Inserimento di dipendenze

Usare [Dependency Injection](xref:fundamentals/dependency-injection) (Inserimento di dipendenze) per rendere i dati disponibili a tutti gli utenti:

1. Definire un servizio che contiene i dati. Ad esempio, una classe denominata `MyAppData` è definita come segue:

    ```csharp
    public class MyAppData
    {
        // Declare properties and methods
    }
    ```

2. Aggiungere la classe del servizio a `Startup.ConfigureServices`:

    ```csharp
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddSingleton<MyAppData>();
    }
    ```

3. Usare la classe del servizio dati:

    ```csharp
    public class IndexModel : PageModel
    {
        public IndexModel(MyAppData myService)
        {
            // Do something with the service
            //    Examples: Read data, store in a field or property
        }
    }
    ```

## <a name="common-errors"></a>Errori comuni

* "Unable to resolve service for type 'Microsoft.Extensions.Caching.Distributed.IDistributedCache' while attempting to activate 'Microsoft.AspNetCore.Session.DistributedSessionStore'." (Risoluzione servizio non riuscita per il tipo 'Microsoft.Extensions.Caching.Distributed.IDistributedCache' durante il tentativo di attivazione di 'Microsoft.AspNetCore.Session.DistributedSessionStore').

  Questo errore dipende in genere dal fatto che almeno un'implementazione `IDistributedCache` non è stata configurata. Per altre informazioni, vedere <xref:performance/caching/distributed> e <xref:performance/caching/memory>.

* Se il middleware di sessione non riesce a rendere permanente una sessione (ad esempio se l'archivio di backup non è disponibile), registra l'eccezione e la richiesta continua normalmente. Ciò provoca un comportamento imprevedibile.

  Ad esempio, un utente archivia un carrello acquisti nella sessione. L'utente aggiunge un articolo al carrello ma il commit ha esito negativo. L'app non riconosce l'errore e segnala all'utente che l'articolo è stato aggiunto al carrello anche se non è vero.

  L'approccio consigliato per verificare la presenza di errori è chiamare `await feature.Session.CommitAsync();` dal codice dell'app quando l'app termina di scrivere nella sessione. `CommitAsync` genera un'eccezione se l'archivio di backup non è disponibile. Se `CommitAsync` ha esito negativo, l'app è in grado di elaborare l'eccezione. `LoadAsync` viene generata nelle stesse condizioni in cui l'archivio dati non è disponibile.
  
## <a name="signalr-and-session-state"></a>SignalR e stato sessione

SignalR le app non devono usare lo stato della sessione per archiviare le informazioni. SignalR le app possono archiviare lo stato per connessione in `Context.Items` nell'hub. <!-- https://github.com/aspnet/SignalR/issues/2139 -->

## <a name="additional-resources"></a>Risorse aggiuntive

<xref:host-and-deploy/web-farm>
::: moniker-end
