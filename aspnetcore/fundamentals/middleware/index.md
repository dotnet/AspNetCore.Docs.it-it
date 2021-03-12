---
title: Middleware di ASP.NET Core
author: rick-anderson
description: Informazioni sul middleware di ASP.NET Core e sulla pipeline delle richieste.
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 07/15/2020
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
uid: fundamentals/middleware/index
ms.openlocfilehash: 15d011e88ab291173668a0b6dc5f46e97fdfeff0
ms.sourcegitcommit: acfe51c35497a204f75c2a61125c9408c04493e6
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102605711"
---
# <a name="aspnet-core-middleware"></a>Middleware di ASP.NET Core

::: moniker range=">= aspnetcore-3.0"

Di [Rick Anderson](https://twitter.com/RickAndMSFT) e [Steve Smith](https://ardalis.com/)

Il middleware è un software che viene assemblato in una pipeline dell'app per gestire richieste e risposte. Ogni componente:

* Sceglie se passare la richiesta al componente successivo nella pipeline.
* Può eseguire le operazioni prima e dopo il componente successivo nella pipeline.

Per compilare la pipeline delle richieste vengono usati i delegati di richiesta. I delegati di richiesta gestiscono ogni richiesta HTTP.

I delegati di richiesta vengono configurati tramite i metodi di estensione <xref:Microsoft.AspNetCore.Builder.RunExtensions.Run%2A>, <xref:Microsoft.AspNetCore.Builder.MapExtensions.Map%2A> e <xref:Microsoft.AspNetCore.Builder.UseExtensions.Use%2A>. È possibile specificare un singolo delegato di richiesta inline come metodo anonimo (chiamato middleware inline) o definirlo in una classe riutilizzabile. Le classi riutilizzabili o i metodi anonimi inline costituiscono il *middleware* o sono noti anche come *componenti middleware*. Ogni componente middleware è responsabile della chiamata del componente seguente nella pipeline o del corto circuito della pipeline. Quando un middleware esegue un corto circuito, viene denominato *middleware terminale* perché impedisce all'ulteriore middleware di elaborare la richiesta.

In <xref:migration/http-modules> sono descritte le differenze tra le pipeline delle richieste in ASP.NET Core e in ASP.NET 4.x e sono riportati ulteriori esempi di middleware.

## <a name="create-a-middleware-pipeline-with-iapplicationbuilder"></a>Creare una pipeline middleware con IApplicationBuilder

La pipeline delle richieste ASP.NET Core è costituita da una sequenza di delegati di richiesta, chiamati uno dopo l'altro. Il diagramma seguente illustra il concetto. Il thread di esecuzione seguente le frecce nere.

![Modello di elaborazione delle richieste che visualizza una richiesta in arrivo, l'elaborazione tramite tre middleware e la risposta che esce dall'app. Ogni middleware esegue la propria logica e passa la richiesta al middleware successivo in corrispondenza dell'istruzione next(). Dopo che il terzo middleware ha elaborato la richiesta, la richiesta torna ai due middleware precedenti in ordine inverso per un'ulteriore elaborazione dopo le istruzioni next() prima di uscire dall'app sotto forma di risposta al client.](index/_static/request-delegate-pipeline.png)

I delegati possono eseguire le operazioni prima del delegato successivo e dopo di esso. I delegati che gestiscono le eccezioni devono essere chiamati nella prima parte della pipeline in modo che possano individuare le eccezioni che si verificano nelle parti successive della pipeline.

L'app di ASP.NET Core più semplice imposta un delegato di richiesta singolo che gestisce tutte le richieste. In questo caso non è inclusa una pipeline di richieste effettiva. Al contrario, viene chiamata una singola funzione anonima in risposta a ogni richiesta HTTP.

[!code-csharp[](index/snapshot/Middleware/Startup.cs)]

È possibile concatenare più delegati di richiesta insieme con <xref:Microsoft.AspNetCore.Builder.UseExtensions.Use%2A>. Il parametro `next` rappresenta il delegato successivo nella pipeline. È possibile eseguire il corto circuito della pipeline *non* chiamando il parametro *next*. In genere è possibile eseguire un'azione prima e dopo il delegato successivo, come illustra l'esempio seguente:

[!code-csharp[](index/snapshot/Chain/Startup.cs?highlight=5-10)]

Quando un delegato non passa una richiesta al delegato successivo, si crea un cosiddetto *corto circuito della pipeline delle richieste*. Il corto circuito è spesso opportuno poiché evita l'esecuzione di operazioni non necessarie. Ad esempio, il [middleware dei file statici](xref:fundamentals/static-files) può operare come *middleware terminale* elaborando una richiesta di un file statico ed effettuando il corto circuito della pipeline rimanente. Il middleware aggiunto alla pipeline prima del middleware che termina l'ulteriore elaborazione elabora comunque il codice dopo le istruzioni `next.Invoke`. Vedere comunque l'avviso seguente sul tentativo di scrivere una risposta che è già stata inviata.

> [!WARNING]
> Non chiamare `next.Invoke` dopo aver inviato la risposta al client. Le modifiche apportate a <xref:Microsoft.AspNetCore.Http.HttpResponse> dopo l'avvio della risposta generano un'eccezione. Se ad esempio si [impostano intestazioni e un codice di stato, viene generata un'eccezione](xref:performance/performance-best-practices#do-not-modify-the-status-code-or-headers-after-the-response-body-has-started). Scrivere nel corpo della risposta dopo aver chiamato `next`:
>
> * Può causare una violazione del protocollo. Ad esempio, scrivere un contenuto che supera il valore `Content-Length` specificato.
> * Può danneggiare il formato del corpo. Ad esempio, scrivere un piè di pagina HTML in un file CSS.
>
> <xref:Microsoft.AspNetCore.Http.HttpResponse.HasStarted%2A> è un hint utile per indicare se le intestazioni sono state inviate o se è stato scritto contenuto nel corpo.

<xref:Microsoft.AspNetCore.Builder.RunExtensions.Run%2A> i delegati non ricevono un `next` parametro. Il primo `Run` delegato è sempre terminale e termina la pipeline. `Run` è una convenzione. Alcuni componenti del middleware possono esporre `Run[Middleware]` metodi che vengono eseguiti alla fine della pipeline:

[!code-csharp[](index/snapshot/Chain/Startup.cs?highlight=12-15)]
[!INCLUDE[about the series](~/includes/code-comments-loc.md)]

Nell'esempio precedente, il `Run` delegato scrive nella `"Hello from 2nd delegate."` risposta e quindi termina la pipeline. Se `Use` `Run` dopo il delegato viene aggiunto un altro delegato o `Run` , questo non viene chiamato.

<a name="order"></a>

## <a name="middleware-order"></a>Ordine middleware

Il diagramma seguente illustra la pipeline di elaborazione delle richieste completa per le app ASP.NET Core MVC e le Razor pagine. È possibile vedere in che modo, in un'app tipica, i middleware esistenti sono ordinati e dove vengono aggiunti middleware personalizzati. Si ha il controllo completo su come riordinare i middleware esistenti o inserire nuovi middleware personalizzati in base alle esigenze per gli scenari.

![Pipeline middleware ASP.NET Core](index/_static/middleware-pipeline.svg)

Il middleware dell' **endpoint** nel diagramma precedente esegue la pipeline di filtro per il tipo di app &mdash; MVC o le Razor pagine corrispondenti.

![Pipeline filtro ASP.NET Core](index/_static/mvc-endpoint.svg)

L'ordine in cui vengono aggiunti i componenti middleware nel metodo `Startup.Configure` definisce l'ordine in cui i componenti middleware vengono richiamati per le richieste e l'ordine inverso per la risposta. L'ordine è **fondamentale** per la sicurezza, le prestazioni e le funzionalità.

Il `Startup.Configure` metodo seguente aggiunge componenti middleware correlati alla sicurezza nell'ordine tipico consigliato:

[!code-csharp[](index/snapshot/StartupAll3.cs?name=snippet)]

Nel codice precedente:

* Il middleware che non viene aggiunto quando si crea una nuova app Web con [account utente singoli](xref:security/authentication/identity) è impostato come commento.
* Non è necessario che tutti i middleware vengano inseriti nell'ordine esatto, ma in molti. Ad esempio:
  * `UseCors`, `UseAuthentication` e devono essere inseriti `UseAuthorization` nell'ordine indicato.
  * `UseCors` attualmente deve andare prima `UseResponseCaching` a causa di [questo bug](https://github.com/dotnet/aspnetcore/issues/23218).

In alcuni scenari il middleware avrà un ordinamento diverso. Ad esempio, la memorizzazione nella cache e l'ordinamento della compressione sono specifici dello scenario ed esistono più ordinamenti validi. Ad esempio:

```csharp
app.UseResponseCaching();
app.UseResponseCompression();
```

Con il codice precedente, la CPU potrebbe essere salvata memorizzando nella cache la risposta compressa, ma è possibile che si verifichi la memorizzazione nella cache di più rappresentazioni di una risorsa usando algoritmi di compressione diversi, ad esempio gzip o brotli.

L'ordinamento seguente combina i file statici per consentire la memorizzazione nella cache dei file statici compressi:

```csharp
app.UseResponseCaching();
app.UseResponseCompression();
app.UseStaticFiles();
```

Il metodo `Startup.Configure` seguente aggiunge componenti del middleware per gli scenari di app comuni:

1. Gestione errori/eccezioni
   * Quando l'app viene eseguita nell'ambiente di sviluppo:
     * Il middleware della pagina delle eccezioni per gli sviluppatori (<xref:Microsoft.AspNetCore.Builder.DeveloperExceptionPageExtensions.UseDeveloperExceptionPage%2A>) segnala gli errori di runtime delle app.
     * Il middleware della pagina di errore del database segnala gli errori di runtime del database.
   * Quando l'app viene eseguita nell'ambiente di produzione:
     * Il middleware del gestore delle eccezioni (<xref:Microsoft.AspNetCore.Builder.ExceptionHandlerExtensions.UseExceptionHandler%2A>) intercetta le eccezioni generate nei middleware seguenti.
     * Il middleware del protocollo HSTS (HTTP Strict Transport Security) (<xref:Microsoft.AspNetCore.Builder.HstsBuilderExtensions.UseHsts%2A>) aggiunge l'intestazione `Strict-Transport-Security`.
1. Il middleware di reindirizzamento HTTPS (<xref:Microsoft.AspNetCore.Builder.HttpsPolicyBuilderExtensions.UseHttpsRedirection%2A>) reindirizza le richieste HTTP a HTTPS.
1. Il middleware dei file statici (<xref:Microsoft.AspNetCore.Builder.StaticFileExtensions.UseStaticFiles%2A>) restituisce i file statici e impedisce ulteriori elaborazioni della richiesta.
1. Cookie Il middleware criterio ( <xref:Microsoft.AspNetCore.Builder.CookiePolicyAppBuilderExtensions.UseCookiePolicy%2A> ) conforme all'app alle normative EU regolamento generale sulla protezione dei dati (GDPR).
1. Middleware di routing ( <xref:Microsoft.AspNetCore.Builder.EndpointRoutingApplicationBuilderExtensions.UseRouting%2A> ) per indirizzare le richieste.
1. Il middleware di autenticazione (<xref:Microsoft.AspNetCore.Builder.AuthAppBuilderExtensions.UseAuthentication%2A>) tenta di autenticare l'utente prima che sia autorizzato ad accedere a risorse protette.
1. Il middleware di autorizzazione ( <xref:Microsoft.AspNetCore.Builder.AuthorizationAppBuilderExtensions.UseAuthorization%2A> ) autorizza un utente ad accedere a risorse protette.
1. Il middleware di sessione (<xref:Microsoft.AspNetCore.Builder.SessionMiddlewareExtensions.UseSession%2A>) stabilisce e mantiene aggiornato lo stato sessione. Se l'app usa lo stato della sessione, chiamare il middleware della sessione dopo il middleware dei Cookie criteri e prima del middleware MVC.
1. Middleware di routing dell'endpoint ( <xref:Microsoft.AspNetCore.Builder.EndpointRoutingApplicationBuilderExtensions.UseEndpoints%2A> con <xref:Microsoft.AspNetCore.Builder.RazorPagesEndpointRouteBuilderExtensions.MapRazorPages%2A> ) per aggiungere Razor gli endpoint di pagina alla pipeline della richiesta.

<!--

FUTURE UPDATE

On the next topic overhaul/release update, add API crosslink to "Database Error Page Middleware" in Item 1 of the list ...

Microsoft.AspNetCore.Builder.DatabaseErrorPageExtensions.UseDatabaseErrorPage*

... when available via the API docs.

-->

```csharp
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
        app.UseDatabaseErrorPage();
    }
    else
    {
        app.UseExceptionHandler("/Error");
        app.UseHsts();
    }

    app.UseHttpsRedirection();
    app.UseStaticFiles();
    app.UseCookiePolicy();
    app.UseRouting();
    app.UseAuthentication();
    app.UseAuthorization();
    app.UseSession();

    app.UseEndpoints(endpoints =>
    {
        endpoints.MapRazorPages();
    });
}
```

Nell'esempio di codice precedente, ogni metodo di estensione del middleware viene esposto in <xref:Microsoft.AspNetCore.Builder.IApplicationBuilder> tramite lo spazio dei nomi <xref:Microsoft.AspNetCore.Builder?displayProperty=fullName>.

<xref:Microsoft.AspNetCore.Builder.ExceptionHandlerExtensions.UseExceptionHandler%2A> è il primo componente middleware aggiunto alla pipeline. Pertanto, il middleware del gestore di eccezioni intercetta le eccezioni che si verificano nelle chiamate successive.

Il middleware dei file statici viene chiamato nella prima parte della pipeline in moda che possa gestire le richieste ed eseguire un corto circuito senza passare attraverso i componenti rimanenti. Il middleware dei file statici **non** fornisce controlli di autorizzazione. Tutti i file serviti dal middleware dei file statici, inclusi quelli in *wwwroot*, sono disponibili pubblicamente. Vedere <xref:fundamentals/static-files> per un approccio alla protezione dei file statici.

Se la richiesta non è gestita dal middleware dei file statici, viene passata al middleware di autenticazione (<xref:Microsoft.AspNetCore.Builder.AuthAppBuilderExtensions.UseAuthentication%2A>) che esegue l'autenticazione. L'autenticazione non esegue il corto circuito di richieste non autenticate. Sebbene il middleware di autenticazione esegua l'autenticazione delle richieste, l'autorizzazione (e il rifiuto) si verifica solo dopo che MVC seleziona una Razor pagina specifica o un controller MVC e un'azione.

L'esempio seguente illustra un ordinamento del middleware nel quale le richieste dei file statici vengono gestite dal middleware dei file statici prima del middleware di compressione delle risposte. I file statici non vengono compressi con questo ordine di middleware. Le Razor risposte alle pagine possono essere compresse.

```csharp
public void Configure(IApplicationBuilder app)
{
    // Static files aren't compressed by Static File Middleware.
    app.UseStaticFiles();

    app.UseRouting();

    app.UseResponseCompression();

    app.UseEndpoints(endpoints =>
    {
        endpoints.MapRazorPages();
    });
}
```

Per le applicazioni a pagina singola (Spa), il middleware SPA <xref:Microsoft.Extensions.DependencyInjection.SpaStaticFilesExtensions.UseSpaStaticFiles%2A> in genere viene visualizzato per ultimo nella pipeline middleware. Il middleware SPA viene visualizzato per ultimo:

* Per consentire a tutti gli altri middleware di rispondere prima alle richieste corrispondenti.
* Per consentire l'esecuzione di SPAs con routing sul lato client per tutte le route non riconosciute dall'app Server.

Per ulteriori informazioni sulle Spa, vedere le guide per i modelli di progetto [React](xref:spa/react) e [angolare](xref:spa/angular) .

### <a name="forwarded-headers-middleware-order"></a>Ordine middleware intestazioni inoltri

[!INCLUDE[](~/includes/ForwardedHeaders.md)]

## <a name="branch-the-middleware-pipeline"></a>Creare un ramo della pipeline middleware

Le estensioni <xref:Microsoft.AspNetCore.Builder.MapExtensions.Map%2A> vengono usate come convenzione per la diramazione della pipeline. `Map` crea un ramo nella pipeline delle richieste in base alle corrispondenze del percorso della richiesta specificato. Se il percorso della richiesta inizia con il percorso specificato, il ramo viene eseguito.

[!code-csharp[](index/snapshot/Chain/StartupMap.cs)]

La tabella seguente visualizza le richieste e le risposte da `http://localhost:1234` usando il codice precedente.

| Richiesta             | Risposta                     |
| ------------------- | ---------------------------- |
| localhost:1234      | Hello from non-Map delegate. |
| localhost:1234/map1 | Map Test 1                   |
| localhost:1234/map2 | Map Test 2                   |
| localhost:1234/map3 | Hello from non-Map delegate. |

Quando si usa `Map`, i segmenti di percorso corrispondenti vengono rimossi da `HttpRequest.Path` e aggiunti a `HttpRequest.PathBase` per ogni richiesta.

`Map` supporta l'annidamento, ad esempio:

```csharp
app.Map("/level1", level1App => {
    level1App.Map("/level2a", level2AApp => {
        // "/level1/level2a" processing
    });
    level1App.Map("/level2b", level2BApp => {
        // "/level1/level2b" processing
    });
});
```

`Map` può anche trovare la corrispondenza di più segmenti contemporaneamente:

[!code-csharp[](index/snapshot/Chain/StartupMultiSeg.cs?highlight=13)]

<xref:Microsoft.AspNetCore.Builder.MapWhenExtensions.MapWhen%2A> crea un ramo nella pipeline delle richieste in base al risultato del predicato specificato. È possibile usare qualsiasi predicato di tipo `Func<HttpContext, bool>` per mappare le richieste a un nuovo ramo della pipeline. Nell'esempio seguente viene usato un predicato per rilevare la presenza di una variabile di stringa di query `branch`:

[!code-csharp[](index/snapshot/Chain/StartupMapWhen.cs?highlight=14-15)]

La tabella seguente visualizza le richieste e le risposte da `http://localhost:1234` usando il codice precedente:

| Richiesta                       | Risposta                     |
| ----------------------------- | ---------------------------- |
| localhost:1234                | Hello from non-Map delegate. |
| localhost: 1234/? Branch = principale | Ramo usato = Main         |

<xref:Microsoft.AspNetCore.Builder.UseWhenExtensions.UseWhen%2A> dirama anche la pipeline della richiesta in base al risultato del predicato specificato. Diversamente da `MapWhen` , questo ramo viene riunito alla pipeline principale se non esegue un cortocircuito o contiene un middleware terminale:

[!code-csharp[](index/snapshot/Chain/StartupUseWhen.cs?highlight=18-19)]

Nell'esempio precedente, risposta "Hello from Main pipeline". viene scritto per tutte le richieste. Se la richiesta include una variabile di stringa di query `branch` , il relativo valore viene registrato prima che la pipeline principale venga riaggiunta.

## <a name="built-in-middleware"></a>Middleware incorporato

ASP.NET Core include i componenti middleware seguenti. Nella colonna *Ordinamento* sono disponibili note sul posizionamento del middleware nella pipeline di elaborazione delle richieste e indicazioni sulle condizioni nelle quali il middleware può terminare l'elaborazione delle richieste. Quando un middleware esegue un corto circuito della pipeline di elaborazione delle richieste e impedisce al middleware a valle di elaborare una richiesta, viene denominato *middleware terminale*. Per altre informazioni sul corto circuito, vedere la sezione [Creare una pipeline middleware con IApplicationBuilder](#create-a-middleware-pipeline-with-iapplicationbuilder).

| Middleware | Descrizione | JSON |
| ---------- | ----------- | ----- |
| [autenticazione](xref:security/authentication/identity) | Offre il supporto dell'autenticazione. | Prima di `HttpContext.User`. Terminale per i callback OAuth. |
| [Autorizzazione](xref:Microsoft.AspNetCore.Builder.AuthorizationAppBuilderExtensions.UseAuthorization%2A) | Fornisce supporto per l'autorizzazione. | Subito dopo il middleware di autenticazione. |
| [Cookie Politica](xref:security/gdpr) | Tiene traccia del consenso degli utenti per l'archiviazione di informazioni personali e impone gli standard minimi per i cookie campi, ad esempio `secure` e `SameSite` . | Prima del middleware che rilascia cookie . Esempi: Autenticazione, Sessione, MVC (TempData). |
| [CORS](xref:security/cors) | Configura la condivisione di risorse tra le origini (CORS). | Prima dei componenti che usano CORS. `UseCors` attualmente deve andare prima `UseResponseCaching` a causa di [questo bug](https://github.com/dotnet/aspnetcore/issues/23218).|
| [Diagnostica](xref:fundamentals/error-handling) | Diversi middleware distinti che forniscono una pagina di eccezione per gli sviluppatori, la gestione delle eccezioni, le tabelle codici di stato e la pagina Web predefinita per le nuove app. | Prima dei componenti che generano errori. Terminale per le eccezioni o per la pagina Web predefinita per le nuove app. |
| [Intestazioni inoltrate](xref:host-and-deploy/proxy-load-balancer) | Inoltra le intestazioni proxy nella richiesta corrente. | Prima dei componenti che usano i campi aggiornati. Esempi: schema, host, IP client, metodo. |
| [Controllo integrità](xref:host-and-deploy/health-checks) | Controlla l'integrità di un'app ASP.NET Core e le relative dipendenze, come il controllo della disponibilità del database. | Terminale se una richiesta corrisponde a un endpoint di controllo di integrità. |
| [Propagazione dell'intestazione](xref:fundamentals/http-requests#header-propagation-middleware) | Propaga le intestazioni HTTP dalla richiesta in ingresso alle richieste del client HTTP in uscita. |
| [Override del metodo HTTP](xref:Microsoft.AspNetCore.Builder.HttpMethodOverrideExtensions) | Consente a una richiesta POST in arrivo di eseguire l'override del metodo. | Prima dei componenti che usano il metodo aggiornato. |
| [Reindirizzamento HTTPS](xref:security/enforcing-ssl#require-https) | Reindirizzare tutte le richieste HTTP a HTTPS. | Prima dei componenti che usano l'URL. |
| [Protocollo HTTP Strict Transport Security (HSTS)](xref:security/enforcing-ssl#http-strict-transport-security-protocol-hsts) | Middleware di ottimizzazione della sicurezza che aggiunge un'intestazione della risposta speciale. | Prima dell'invio delle risposte e dopo i componenti che modificano le richieste. Esempi: intestazioni inoltrate, riscrittura dell'URL. |
| [MVC](xref:mvc/overview) | Elabora le richieste con MVC/ Razor pagine. | Terminale se una richiesta corrisponde a una route. |
| [OWIN](xref:fundamentals/owin) | Interoperabilità con app, server e middleware basati su OWIN. | Terminale se la richiesta viene elaborata completamente dal middleware OWIN. |
| [Memorizzazione nella cache delle risposte](xref:performance/caching/middleware) | Offre il supporto per la memorizzazione delle risposte nella cache. | Prima dei componenti che richiedono la memorizzazione nella cache. `UseCORS` deve essere precedente a `UseResponseCaching` .|
| [Compressione della risposta](xref:performance/response-compression) | Offre il supporto per la compressione delle risposte. | Prima dei componenti che richiedono la compressione. |
| [Localizzazione della richiesta](xref:fundamentals/localization) | Offre il supporto per la localizzazione. | Prima dei componenti sensibili alla localizzazione. |
| [Routing di endpoint](xref:fundamentals/routing) | Definisce e vincola le route di richiesta. | Terminale per le route corrispondenti. |
| [SPA](xref:Microsoft.AspNetCore.Builder.SpaApplicationBuilderExtensions.UseSpa%2A) | Gestisce tutte le richieste da questo punto nella catena di middleware restituendo la pagina predefinita per l'applicazione a pagina singola (SPA) | In ritardo nella catena, in modo che altro middleware per la conservazione di file statici, azioni MVC e così via, abbia la precedenza.|
| [Sessione](xref:fundamentals/app-state) | Offre il supporto per la gestione delle sessioni utente. | Prima dei componenti che richiedono la Sessione. | 
| [File statici](xref:fundamentals/static-files) | Offre il supporto per la gestione di file statici e l'esplorazione directory. | Terminale se una richiesta corrisponde a un file. |
| [URL Rewrite](xref:fundamentals/url-rewriting) (Riscrittura URL) | Offre il supporto per la riscrittura degli URL e il reindirizzamento delle richieste. | Prima dei componenti che usano l'URL. |
| [WebSocket](xref:fundamentals/websockets) | Abilita il protocollo WebSocket. | Prima dei componenti necessari per accettare le richieste WebSocket. |

## <a name="additional-resources"></a>Risorse aggiuntive

* Il [ciclo di vita e le opzioni di registrazione](xref:fundamentals/dependency-injection#lifetime-and-registration-options) contengono un esempio completo di middleware con servizi di durata con *ambito*, *temporaneo* e *singleton* .
* <xref:fundamentals/middleware/write>
* <xref:test/middleware>
* <xref:migration/http-modules>
* <xref:fundamentals/startup>
* <xref:fundamentals/request-features>
* <xref:fundamentals/middleware/extensibility>
* <xref:fundamentals/middleware/extensibility-third-party-container>

::: moniker-end

::: moniker range="< aspnetcore-3.0"

Di [Rick Anderson](https://twitter.com/RickAndMSFT) e [Steve Smith](https://ardalis.com/)

Il middleware è un software che viene assemblato in una pipeline dell'app per gestire richieste e risposte. Ogni componente:

* Sceglie se passare la richiesta al componente successivo nella pipeline.
* Può eseguire le operazioni prima e dopo il componente successivo nella pipeline.

Per compilare la pipeline delle richieste vengono usati i delegati di richiesta. I delegati di richiesta gestiscono ogni richiesta HTTP.

I delegati di richiesta vengono configurati tramite i metodi di estensione <xref:Microsoft.AspNetCore.Builder.RunExtensions.Run%2A>, <xref:Microsoft.AspNetCore.Builder.MapExtensions.Map%2A> e <xref:Microsoft.AspNetCore.Builder.UseExtensions.Use%2A>. È possibile specificare un singolo delegato di richiesta inline come metodo anonimo (chiamato middleware inline) o definirlo in una classe riutilizzabile. Le classi riutilizzabili o i metodi anonimi inline costituiscono il *middleware* o sono noti anche come *componenti middleware*. Ogni componente middleware è responsabile della chiamata del componente seguente nella pipeline o del corto circuito della pipeline. Quando un middleware esegue un corto circuito, viene denominato *middleware terminale* perché impedisce all'ulteriore middleware di elaborare la richiesta.

In <xref:migration/http-modules> sono descritte le differenze tra le pipeline delle richieste in ASP.NET Core e in ASP.NET 4.x e sono riportati ulteriori esempi di middleware.

## <a name="create-a-middleware-pipeline-with-iapplicationbuilder"></a>Creare una pipeline middleware con IApplicationBuilder

La pipeline delle richieste ASP.NET Core è costituita da una sequenza di delegati di richiesta, chiamati uno dopo l'altro. Il diagramma seguente illustra il concetto. Il thread di esecuzione seguente le frecce nere.

![Modello di elaborazione delle richieste che visualizza una richiesta in arrivo, l'elaborazione tramite tre middleware e la risposta che esce dall'app. Ogni middleware esegue la propria logica e passa la richiesta al middleware successivo in corrispondenza dell'istruzione next(). Dopo che il terzo middleware ha elaborato la richiesta, la richiesta torna ai due middleware precedenti in ordine inverso per un'ulteriore elaborazione dopo le istruzioni next() prima di uscire dall'app sotto forma di risposta al client.](index/_static/request-delegate-pipeline.png)

I delegati possono eseguire le operazioni prima del delegato successivo e dopo di esso. I delegati che gestiscono le eccezioni devono essere chiamati nella prima parte della pipeline in modo che possano individuare le eccezioni che si verificano nelle parti successive della pipeline.

L'app di ASP.NET Core più semplice imposta un delegato di richiesta singolo che gestisce tutte le richieste. In questo caso non è inclusa una pipeline di richieste effettiva. Al contrario, viene chiamata una singola funzione anonima in risposta a ogni richiesta HTTP.

[!code-csharp[](index/snapshot/Middleware/Startup.cs)]

Il primo delegato <xref:Microsoft.AspNetCore.Builder.RunExtensions.Run%2A> termina la pipeline.

È possibile concatenare più delegati di richiesta insieme con <xref:Microsoft.AspNetCore.Builder.UseExtensions.Use%2A>. Il parametro `next` rappresenta il delegato successivo nella pipeline. È possibile eseguire il corto circuito della pipeline *non* chiamando il parametro *next*. In genere è possibile eseguire un'azione prima e dopo il delegato successivo, come illustra l'esempio seguente:

[!code-csharp[](index/snapshot/Chain/Startup.cs)]

Quando un delegato non passa una richiesta al delegato successivo, si crea un cosiddetto *corto circuito della pipeline delle richieste*. Il corto circuito è spesso opportuno poiché evita l'esecuzione di operazioni non necessarie. Ad esempio, il [middleware dei file statici](xref:fundamentals/static-files) può operare come *middleware terminale* elaborando una richiesta di un file statico ed effettuando il corto circuito della pipeline rimanente. Il middleware aggiunto alla pipeline prima del middleware che termina l'ulteriore elaborazione elabora comunque il codice dopo le istruzioni `next.Invoke`. Vedere comunque l'avviso seguente sul tentativo di scrivere una risposta che è già stata inviata.

> [!WARNING]
> Non chiamare `next.Invoke` dopo aver inviato la risposta al client. Le modifiche apportate a <xref:Microsoft.AspNetCore.Http.HttpResponse> dopo l'avvio della risposta generano un'eccezione. Ad esempio, le modifiche come l'impostazione delle intestazioni e di un codice di stato generano un'eccezione. Scrivere nel corpo della risposta dopo aver chiamato `next`:
>
> * Può causare una violazione del protocollo. Ad esempio, scrivere un contenuto che supera il valore `Content-Length` specificato.
> * Può danneggiare il formato del corpo. Ad esempio, scrivere un piè di pagina HTML in un file CSS.
>
> <xref:Microsoft.AspNetCore.Http.HttpResponse.HasStarted%2A> è un hint utile per indicare se le intestazioni sono state inviate o se è stato scritto contenuto nel corpo.

<a name="order"></a>

## <a name="middleware-order"></a>Ordine middleware

L'ordine in cui vengono aggiunti i componenti middleware nel metodo `Startup.Configure` definisce l'ordine in cui i componenti middleware vengono richiamati per le richieste e l'ordine inverso per la risposta. L'ordine è **fondamentale** per la sicurezza, le prestazioni e le funzionalità.

Il `Startup.Configure` metodo seguente aggiunge componenti middleware correlati alla sicurezza nell'ordine consigliato:

[!code-csharp[](index/snapshot/Startup22.cs?name=snippet)]

Nel codice precedente:

* Il middleware che non viene aggiunto quando si crea una nuova app Web con [account utente singoli](xref:security/authentication/identity) è impostato come commento.
* Non è necessario che tutti i middleware vengano inseriti nell'ordine esatto, ma in molti. Ad esempio, `UseCors` e `UseAuthentication` devono andare nell'ordine indicato.

Il metodo `Startup.Configure` seguente aggiunge componenti del middleware per gli scenari di app comuni:

1. Gestione errori/eccezioni
   * Quando l'app viene eseguita nell'ambiente di sviluppo:
     * Il middleware della pagina delle eccezioni per gli sviluppatori (<xref:Microsoft.AspNetCore.Builder.DeveloperExceptionPageExtensions.UseDeveloperExceptionPage%2A>) segnala gli errori di runtime delle app.
     * Il middleware della pagina degli errori del database (`Microsoft.AspNetCore.Builder.DatabaseErrorPageExtensions.UseDatabaseErrorPage`) segnala gli errori di runtime del database.
   * Quando l'app viene eseguita nell'ambiente di produzione:
     * Il middleware del gestore delle eccezioni (<xref:Microsoft.AspNetCore.Builder.ExceptionHandlerExtensions.UseExceptionHandler%2A>) intercetta le eccezioni generate nei middleware seguenti.
     * Il middleware del protocollo HSTS (HTTP Strict Transport Security) (<xref:Microsoft.AspNetCore.Builder.HstsBuilderExtensions.UseHsts%2A>) aggiunge l'intestazione `Strict-Transport-Security`.
1. Il middleware di reindirizzamento HTTPS (<xref:Microsoft.AspNetCore.Builder.HttpsPolicyBuilderExtensions.UseHttpsRedirection%2A>) reindirizza le richieste HTTP a HTTPS.
1. Il middleware dei file statici (<xref:Microsoft.AspNetCore.Builder.StaticFileExtensions.UseStaticFiles%2A>) restituisce i file statici e impedisce ulteriori elaborazioni della richiesta.
1. Cookie Il middleware criterio ( <xref:Microsoft.AspNetCore.Builder.CookiePolicyAppBuilderExtensions.UseCookiePolicy%2A> ) conforme all'app alle normative EU regolamento generale sulla protezione dei dati (GDPR).
1. Il middleware di autenticazione (<xref:Microsoft.AspNetCore.Builder.AuthAppBuilderExtensions.UseAuthentication%2A>) tenta di autenticare l'utente prima che sia autorizzato ad accedere a risorse protette.
1. Il middleware di sessione (<xref:Microsoft.AspNetCore.Builder.SessionMiddlewareExtensions.UseSession%2A>) stabilisce e mantiene aggiornato lo stato sessione. Se l'app usa lo stato della sessione, chiamare il middleware della sessione dopo il middleware dei Cookie criteri e prima del middleware MVC.
1. MVC (<xref:Microsoft.AspNetCore.Builder.MvcApplicationBuilderExtensions.UseMvc%2A>) per aggiungere MVC alla pipeline delle richieste.

```csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
        app.UseDatabaseErrorPage();
    }
    else
    {
        app.UseExceptionHandler("/Error");
        app.UseHsts();
    }

    app.UseHttpsRedirection();
    app.UseStaticFiles();
    app.UseCookiePolicy();
    app.UseAuthentication();
    app.UseSession();
    app.UseMvc();
}
```

Nell'esempio di codice precedente, ogni metodo di estensione del middleware viene esposto in <xref:Microsoft.AspNetCore.Builder.IApplicationBuilder> tramite lo spazio dei nomi <xref:Microsoft.AspNetCore.Builder?displayProperty=fullName>.

<xref:Microsoft.AspNetCore.Builder.ExceptionHandlerExtensions.UseExceptionHandler%2A> è il primo componente middleware aggiunto alla pipeline. Pertanto, il middleware del gestore di eccezioni intercetta le eccezioni che si verificano nelle chiamate successive.

Il middleware dei file statici viene chiamato nella prima parte della pipeline in moda che possa gestire le richieste ed eseguire un corto circuito senza passare attraverso i componenti rimanenti. Il middleware dei file statici **non** fornisce controlli di autorizzazione. Tutti i file serviti dal middleware dei file statici, inclusi quelli in *wwwroot*, sono disponibili pubblicamente. Vedere <xref:fundamentals/static-files> per un approccio alla protezione dei file statici.

Se la richiesta non è gestita dal middleware dei file statici, viene passata al middleware di autenticazione (<xref:Microsoft.AspNetCore.Builder.AuthAppBuilderExtensions.UseAuthentication%2A>) che esegue l'autenticazione. L'autenticazione non esegue il corto circuito di richieste non autenticate. Sebbene il middleware di autenticazione esegua l'autenticazione delle richieste, l'autorizzazione (e il rifiuto) si verifica solo dopo che MVC seleziona una Razor pagina specifica o un controller MVC e un'azione.

L'esempio seguente illustra un ordinamento del middleware nel quale le richieste dei file statici vengono gestite dal middleware dei file statici prima del middleware di compressione delle risposte. I file statici non vengono compressi con questo ordine di middleware. Le risposte MVC da <xref:Microsoft.AspNetCore.Builder.MvcApplicationBuilderExtensions.UseMvcWithDefaultRoute%2A> possono essere compresse.

```csharp
public void Configure(IApplicationBuilder app)
{
    // Static files aren't compressed by Static File Middleware.
    app.UseStaticFiles();

    app.UseResponseCompression();

    app.UseMvcWithDefaultRoute();
}
```

## <a name="use-run-and-map"></a>Use, Run e Map

Configurare la pipeline HTTP usando <xref:Microsoft.AspNetCore.Builder.UseExtensions.Use%2A>,<xref:Microsoft.AspNetCore.Builder.RunExtensions.Run%2A> e <xref:Microsoft.AspNetCore.Builder.MapExtensions.Map%2A>. Il metodo `Use` può eseguire il corto circuito della pipeline, ovvero non chiamare un delegato di richiesta `next`. `Run` è una convenzione e alcuni componenti del middleware possono esporre metodi `Run[Middleware]` che vengono eseguiti al termine della pipeline.

Le estensioni <xref:Microsoft.AspNetCore.Builder.MapExtensions.Map%2A> vengono usate come convenzione per la diramazione della pipeline. `Map` crea un ramo nella pipeline delle richieste in base alle corrispondenze del percorso della richiesta specificato. Se il percorso della richiesta inizia con il percorso specificato, il ramo viene eseguito.

[!code-csharp[](index/snapshot/Chain/StartupMap.cs)]

La tabella seguente visualizza le richieste e le risposte da `http://localhost:1234` usando il codice precedente.

| Richiesta             | Risposta                     |
| ------------------- | ---------------------------- |
| localhost:1234      | Hello from non-Map delegate. |
| localhost:1234/map1 | Map Test 1                   |
| localhost:1234/map2 | Map Test 2                   |
| localhost:1234/map3 | Hello from non-Map delegate. |

Quando si usa `Map`, i segmenti di percorso corrispondenti vengono rimossi da `HttpRequest.Path` e aggiunti a `HttpRequest.PathBase` per ogni richiesta.

<xref:Microsoft.AspNetCore.Builder.MapWhenExtensions.MapWhen%2A> crea un ramo nella pipeline delle richieste in base al risultato del predicato specificato. È possibile usare qualsiasi predicato di tipo `Func<HttpContext, bool>` per mappare le richieste a un nuovo ramo della pipeline. Nell'esempio seguente viene usato un predicato per rilevare la presenza di una variabile di stringa di query `branch`:

[!code-csharp[](index/snapshot/Chain/StartupMapWhen.cs)]

La tabella seguente visualizza le richieste e le risposte da `http://localhost:1234` usando il codice precedente.

| Richiesta                       | Risposta                     |
| ----------------------------- | ---------------------------- |
| localhost:1234                | Hello from non-Map delegate. |
| localhost: 1234/? Branch = principale | Ramo usato = Main         |

`Map` supporta l'annidamento, ad esempio:

```csharp
app.Map("/level1", level1App => {
    level1App.Map("/level2a", level2AApp => {
        // "/level1/level2a" processing
    });
    level1App.Map("/level2b", level2BApp => {
        // "/level1/level2b" processing
    });
});
```

`Map` può anche trovare la corrispondenza di più segmenti contemporaneamente:

[!code-csharp[](index/snapshot/Chain/StartupMultiSeg.cs?highlight=13)]

## <a name="built-in-middleware"></a>Middleware incorporato

ASP.NET Core include i componenti middleware seguenti. Nella colonna *Ordinamento* sono disponibili note sul posizionamento del middleware nella pipeline di elaborazione delle richieste e indicazioni sulle condizioni nelle quali il middleware può terminare l'elaborazione delle richieste. Quando un middleware esegue un corto circuito della pipeline di elaborazione delle richieste e impedisce al middleware a valle di elaborare una richiesta, viene denominato *middleware terminale*. Per altre informazioni sul corto circuito, vedere la sezione [Creare una pipeline middleware con IApplicationBuilder](#create-a-middleware-pipeline-with-iapplicationbuilder).

| Middleware | Descrizione | JSON |
| ---------- | ----------- | ----- |
| [autenticazione](xref:security/authentication/identity) | Offre il supporto dell'autenticazione. | Prima di `HttpContext.User`. Terminale per i callback OAuth. |
| [Cookie Politica](xref:security/gdpr) | Tiene traccia del consenso degli utenti per l'archiviazione di informazioni personali e impone gli standard minimi per i cookie campi, ad esempio `secure` e `SameSite` . | Prima del middleware che rilascia cookie . Esempi: Autenticazione, Sessione, MVC (TempData). |
| [CORS](xref:security/cors) | Configura la condivisione di risorse tra le origini (CORS). | Prima dei componenti che usano CORS. |
| [Diagnostica](xref:fundamentals/error-handling) | Diversi middleware distinti che forniscono una pagina di eccezione per gli sviluppatori, la gestione delle eccezioni, le tabelle codici di stato e la pagina Web predefinita per le nuove app. | Prima dei componenti che generano errori. Terminale per le eccezioni o per la pagina Web predefinita per le nuove app. |
| [Intestazioni inoltrate](xref:host-and-deploy/proxy-load-balancer) | Inoltra le intestazioni proxy nella richiesta corrente. | Prima dei componenti che usano i campi aggiornati. Esempi: schema, host, IP client, metodo. |
| [Controllo integrità](xref:host-and-deploy/health-checks) | Controlla l'integrità di un'app ASP.NET Core e le relative dipendenze, come il controllo della disponibilità del database. | Terminale se una richiesta corrisponde a un endpoint di controllo di integrità. |
| [Override del metodo HTTP](xref:Microsoft.AspNetCore.Builder.HttpMethodOverrideExtensions) | Consente a una richiesta POST in arrivo di eseguire l'override del metodo. | Prima dei componenti che usano il metodo aggiornato. |
| [Reindirizzamento HTTPS](xref:security/enforcing-ssl#require-https) | Reindirizzare tutte le richieste HTTP a HTTPS. | Prima dei componenti che usano l'URL. |
| [Protocollo HTTP Strict Transport Security (HSTS)](xref:security/enforcing-ssl#http-strict-transport-security-protocol-hsts) | Middleware di ottimizzazione della sicurezza che aggiunge un'intestazione della risposta speciale. | Prima dell'invio delle risposte e dopo i componenti che modificano le richieste. Esempi: intestazioni inoltrate, riscrittura dell'URL. |
| [MVC](xref:mvc/overview) | Elabora le richieste con MVC/ Razor pagine. | Terminale se una richiesta corrisponde a una route. |
| [OWIN](xref:fundamentals/owin) | Interoperabilità con app, server e middleware basati su OWIN. | Terminale se la richiesta viene elaborata completamente dal middleware OWIN. |
| [Memorizzazione nella cache delle risposte](xref:performance/caching/middleware) | Offre il supporto per la memorizzazione delle risposte nella cache. | Prima dei componenti che richiedono la memorizzazione nella cache. |
| [Compressione della risposta](xref:performance/response-compression) | Offre il supporto per la compressione delle risposte. | Prima dei componenti che richiedono la compressione. |
| [Localizzazione della richiesta](xref:fundamentals/localization) | Offre il supporto per la localizzazione. | Prima dei componenti sensibili alla localizzazione. |
| [Routing di endpoint](xref:fundamentals/routing) | Definisce e vincola le route di richiesta. | Terminale per le route corrispondenti. |
| [Sessione](xref:fundamentals/app-state) | Offre il supporto per la gestione delle sessioni utente. | Prima dei componenti che richiedono la Sessione. |
| [File statici](xref:fundamentals/static-files) | Offre il supporto per la gestione di file statici e l'esplorazione directory. | Terminale se una richiesta corrisponde a un file. |
| [URL Rewrite](xref:fundamentals/url-rewriting) (Riscrittura URL) | Offre il supporto per la riscrittura degli URL e il reindirizzamento delle richieste. | Prima dei componenti che usano l'URL. |
| [WebSocket](xref:fundamentals/websockets) | Abilita il protocollo WebSocket. | Prima dei componenti necessari per accettare le richieste WebSocket. |

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:fundamentals/middleware/write>
* <xref:test/middleware>
* <xref:migration/http-modules>
* <xref:fundamentals/startup>
* <xref:fundamentals/request-features>
* <xref:fundamentals/middleware/extensibility>
* <xref:fundamentals/middleware/extensibility-third-party-container>

::: moniker-end
