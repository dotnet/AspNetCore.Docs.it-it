---
title: Middleware Riscrittura URL in ASP.NET Core
author: rick-anderson
description: Informazioni sulla riscrittura e il reindirizzamento di URL con il middleware Riscrittura URL nelle applicazioni ASP.NET Core.
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 08/16/2019
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
uid: fundamentals/url-rewriting
ms.openlocfilehash: 55993f951bfe910fb179fdb25b8d04f81d4ee2c9
ms.sourcegitcommit: 83524f739dd25fbfa95ee34e95342afb383b49fe
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 01/29/2021
ms.locfileid: "99057460"
---
# <a name="url-rewriting-middleware-in-aspnet-core"></a>Middleware Riscrittura URL in ASP.NET Core

Di [Mikael Mengistu](https://github.com/mikaelm12)

::: moniker range=">= aspnetcore-3.0"

Questo documento presenta la riscrittura URL con istruzioni per l'uso del middleware Riscrittura URL nelle applicazioni ASP.NET Core.

La riscrittura URL è l'azione di modificare gli URL di richiesta in base a una o più regole predefinite. Il processo di riscrittura URL crea un'astrazione tra i percorsi delle risorse e i relativi indirizzi, in modo che i percorsi e gli indirizzi non risultino strettamente collegati. La riscrittura URL risulta utile in diversi scenari per:

* Spostare o sostituire in modo temporaneo o permanente risorse server mantenendo localizzatori stabili di queste risorse.
* Suddividere l'elaborazione delle richieste tra app diverse o tra aree diverse di un'unica app.
* Rimuovere, aggiungere o riorganizzare segmenti URL nelle richieste in ingresso.
* Ottimizzare gli URL pubblici per l'ottimizzazione motore di ricerca (SEO, Search Engine Optimization).
* Consentire l'uso di URL pubblici descrittivi in modo che i visitatori possano prevedere il contenuto restituito dalla richiesta di una risorsa.
* Reindirizzare le richieste non protette a endpoint protetti.
* Evitare l'hotlinking, ovvero l'uso da parte di un sito esterno di una risorsa statica ospitata presente in un altro sito tramite il collegamento di questa al proprio contenuto.

> [!NOTE]
> La riscrittura URL può ridurre le prestazioni di un'app. Ove possibile, limitare il numero e la complessità delle regole.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/url-rewriting/samples/) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="url-redirect-and-url-rewrite"></a>Reindirizzamento URL e riscrittura URL

La differenza tra i termini *reindirizzamento URL* e *riscrittura URL* è minima, ma ha implicazioni importanti nell'offerta di risorse ai clienti. Il middleware Riscrittura URL di ASP.NET Core è in grado di svolgere entrambe le funzioni.

Un *reindirizzamento URL* implica un'operazione lato client, in cui viene richiesto al client di accedere a una risorsa a un indirizzo diverso da quello richiesto originariamente dal client. Questa operazione richiede un round trip al server. L'URL di reindirizzamento restituito al client viene visualizzato nella barra degli indirizzi del browser quando il client effettua una nuova richiesta per la risorsa.

In caso di *reindirizzamento* di `/resource` a `/different-resource`, il server risponde che il client deve ottenere la risorsa presso `/different-resource` con un codice di stato che indica se il reindirizzamento è temporaneo o permanente.

![Un endpoint di servizio WebAPI è stato modificato temporaneamente dalla versione 1 (v1) alla versione 2 (v2) nel server. Un client esegue una richiesta al servizio nel percorso della versione 1, ovvero /v1/api. Il server restituisce una risposta 302 (Trovato) con il nuovo percorso temporaneo del servizio versione 2, ovvero /v2/api. Il client esegue una seconda richiesta al servizio all'URL di reindirizzamento. Il server risponde con un codice di stato 200 (OK).](url-rewriting/_static/url_redirect.png)

Quando si reindirizzano le richieste a un URL diverso, indicare se il reindirizzamento è temporaneo o permanente specificando il codice di stato con la risposta:

* Il codice di stato *301 - Spostato permanentemente* viene usato se la risorsa ha un nuovo URL definitivo e si vuole indicare al client che tale URL dovrà essere usato per tutte le richieste future della risorsa. *Quando riceve un codice di stato 301, il client può memorizzare la risposta nella cache e riusarla.*

* Il codice di stato *302 - Trovato* viene usato quando il reindirizzamento è temporaneo o comunque soggetto a modifiche. Il codice di stato 302 indica che il client non dovrà archiviare e riusare l'URL di reindirizzamento in futuro.

Per altre informazioni sui codici di stato, vedere [RFC 2616: Status Code Definitions](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html) (RFC 2616: Definizioni dei codici di stato).

La *riscrittura URL* è un'operazione lato server che rende disponibile una risorsa da un indirizzo diverso da quello richiesto dal client. La riscrittura URL non richiede un round trip al server. L'URL riscritto non viene restituito al client e non viene visualizzato nella barra degli indirizzi del browser.

Se `/resource` viene *riscritto* in `/different-resource`, il server recupera *internamente* la risorsa in `/different-resource`.

Anche se il client fosse in grado di recuperare la risorsa nell'URL riscritto, non viene informato dell'esistenza della risorsa nell'URL riscritto quando effettua la richiesta e riceve la risposta.

![Un endpoint servizio WebAPI è stato modificato dalla versione 1 (v1) alla versione 2 (v2) sul server. Un client esegue una richiesta al servizio nel percorso della versione 1, ovvero /v1/api. L'URL della richiesta viene riscritto per accedere al servizio nel percorso della versione 2, ovvero /v2/api. Il servizio risponde al client con un codice di stato 200 (OK).](url-rewriting/_static/url_rewrite.png)

## <a name="url-rewriting-sample-app"></a>App di esempio per la riscrittura URL

È possibile esplorare le funzionalità del middleware Riscrittura URL con l'[app di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/url-rewriting/samples/). L'app applica regole di reindirizzamento e riscrittura e visualizza l'URL reindirizzato o riscritto per diversi scenari.

## <a name="when-to-use-url-rewriting-middleware"></a>Quando usare il middleware Riscrittura URL

Usare il middleware Riscrittura URL quando non è possibile usare gli approcci seguenti:

* [URL Rewrite Module for IIS in Windows Server](https://www.iis.net/downloads/microsoft/url-rewrite)
* [Modulo Apache mod_rewrite in Apache Server](https://httpd.apache.org/docs/2.4/rewrite/)
* [Riscrittura URL in Nginx](https://www.nginx.com/blog/creating-nginx-rewrite-rules/)

Usare il middleware anche se l'app è ospitata in un [server HTTP.sys](xref:fundamentals/servers/httpsys) (in precedenza denominato WebListener).

I motivi principali per usare tecnologie di riscrittura URL basate su server in IIS, Apache e Nginx sono:

* Il middleware non supporta tutte le funzionalità di questi moduli.

  Alcune funzionalità dei moduli server non funzionano con i progetti ASP.NET Core, ad esempio i vincoli `IsFile` e `IsDirectory` del modulo IIS Rewrite. In questi scenari è necessario usare il middleware.
* Le prestazioni del middleware probabilmente non corrispondono a quelle dei moduli.

  Il benchmarking rappresenta l'unico modo per sapere con certezza quale approccio comporta la maggior riduzione delle prestazioni o se tale riduzione è trascurabile.

## <a name="package"></a>Pacchetto

Il middleware di riscrittura URL è fornito dal pacchetto [Microsoft.AspNetCore.Rewrite](https://www.nuget.org/packages/Microsoft.AspNetCore.Rewrite), implicitamente incluso nelle app ASP.NET Core.

## <a name="extension-and-options"></a>Estensioni e opzioni

È possibile definire regole di riscrittura e reindirizzamento URL creando un'istanza della classe [RewriteOptions](xref:Microsoft.AspNetCore.Rewrite.RewriteOptions) con metodi di estensione per le singole regole di riscrittura. Concatenare più regole nell'ordine in cui si vuole che vengano elaborate. Le opzioni `RewriteOptions` vengono passate al middleware Riscrittura URL quando questo viene aggiunto alla pipeline della richiesta con <xref:Microsoft.AspNetCore.Builder.RewriteBuilderExtensions.UseRewriter*>:

[!code-csharp[](url-rewriting/samples/3.x/SampleApp/Startup.cs?name=snippet1)]

### <a name="redirect-non-www-to-www"></a>Reindirizzare non-www su www

Sono disponibili tre opzioni che consentono all'app di reindirizzare richieste non-`www` su `www`:

* <xref:Microsoft.AspNetCore.Rewrite.RewriteOptionsExtensions.AddRedirectToWwwPermanent*>: Reindirizza in modo permanente la richiesta al `www` sottodominio se la richiesta è non `www` . Reindirizza con un codice di stato [Status308PermanentRedirect](xref:Microsoft.AspNetCore.Http.StatusCodes.Status308PermanentRedirect).

* <xref:Microsoft.AspNetCore.Rewrite.RewriteOptionsExtensions.AddRedirectToWww*>: Reindirizza la richiesta al `www` sottodominio se la richiesta in ingresso è non `www` . Reindirizza con un codice di stato [Status307TemporaryRedirect](xref:Microsoft.AspNetCore.Http.StatusCodes.Status307TemporaryRedirect). Un overload consente di specificare il codice di stato per la risposta. Usare un campo della classe <xref:Microsoft.AspNetCore.Http.StatusCodes> per un'assegnazione di codice di stato.

### <a name="url-redirect"></a>Reindirizzamento URL

Per reindirizzare le richieste, usare <xref:Microsoft.AspNetCore.Rewrite.RewriteOptionsExtensions.AddRedirect*>. Il primo parametro contiene l'espressione regolare per la corrispondenza in base al percorso dell'URL in ingresso. Il secondo parametro è la stringa di sostituzione. Il terzo parametro, se presente, specifica il codice di stato. Se il codice di stato non è specificato, assume come valore predefinito *302 - Trovato*, che indica che la risorsa è stata temporaneamente spostata o sostituita.

[!code-csharp[](url-rewriting/samples/3.x/SampleApp/Startup.cs?name=snippet1&highlight=9)]

In un browser con gli strumenti di sviluppo abilitati, creare una richiesta all'app di esempio con il percorso `/redirect-rule/1234/5678`. L'espressione regolare definisce la corrispondenza con il percorso di richiesta in `redirect-rule/(.*)` e il percorso viene sostituito con `/redirected/1234/5678`. L'URL di reindirizzamento viene restituito al client con il codice di stato *302 - Trovato*. Il browser invia una nuova richiesta all'URL di reindirizzamento, che viene visualizzata nella barra degli indirizzi del browser. Poiché nessuna regola nell'app di esempio corrisponde all'URL di reindirizzamento:

* La seconda richiesta riceve una risposta *200 - OK* dall'app.
* Il corpo della risposta visualizza l'URL di reindirizzamento.

Quando un URL viene *reindirizzato*, viene eseguito un round trip al server.

> [!WARNING]
> Prestare attenzione quando si definiscono regole di reindirizzamento. Le regole di reindirizzamento vengono valutate in ogni richiesta all'app, anche dopo un reindirizzamento. È facile creare inavvertitamente un *ciclo di reindirizzamento infinito*.

Richiesta originale: `/redirect-rule/1234/5678`

![Finestra del browser con gli strumenti di sviluppo per il rilevamento di richieste e risposte](url-rewriting/_static/add_redirect.png)

La parte dell'espressione racchiusa tra parentesi è denominata *gruppo Capture*. Il punto (`.`) dell'espressione significa *corrispondenza con qualsiasi carattere*. L'asterisco (`*`) indica *corrispondenza con il carattere precedente per zero o più volte*. Di conseguenza gli ultimi due i segmenti di percorso dell'URL `1234/5678` vengono acquisiti tramite il gruppo Capture `(.*)`. Qualsiasi valore visualizzato nell'URL della richiesta dopo `redirect-rule/` viene acquisito da questo gruppo Capture.

Nella stringa di sostituzione i gruppi acquisiti vengono inseriti nella stringa con il simbolo di dollaro (`$`) seguito dal numero di sequenza dell'acquisizione. Il primo valore del gruppo Capture viene ottenuto con `$1`, il secondo con `$2` e la procedura continua in sequenza per i gruppi Capture presenti nell'espressione regolare. Nell'espressione regolare della regola di reindirizzamento dell'app di esempio è presente un solo gruppo acquisito, pertanto nella stringa di sostituzione è presente un solo gruppo inserito, ovvero `$1`. Quando viene applicata la regola, l'URL diventa `/redirected/1234/5678`.

### <a name="url-redirect-to-a-secure-endpoint"></a>Reindirizzamento URL a un endpoint protetto

Usare <xref:Microsoft.AspNetCore.Rewrite.RewriteOptionsExtensions.AddRedirectToHttps*> per reindirizzare le richieste HTTP allo stesso host e allo stesso percorso tramite il protocollo HTTPS. Se il codice di stato non viene specificato, il middleware usa come valore predefinito *302 - Trovato*. Se non viene specificata la porta:

* Il middleware usa come valore predefinito `null`.
* Lo schema viene modificato in `https` (protocollo HTTPS) e il client accede alla risorsa attraverso la porta 443.

L'esempio seguente illustra come impostare il codice di stato su *301 - Spostato permanentemente* e come passare alla porta 5001.

```csharp
public void Configure(IApplicationBuilder app)
{
    var options = new RewriteOptions()
        .AddRedirectToHttps(301, 5001);

    app.UseRewriter(options);
}
```

Usare <xref:Microsoft.AspNetCore.Rewrite.RewriteOptionsExtensions.AddRedirectToHttpsPermanent*> per reindirizzare le richieste non protette allo stesso host e allo stesso percorso con il protocollo protetto HTTPS attraverso la porta 443. Il middleware imposta il codice di stato su *301 - Spostato permanentemente*.

```csharp
public void Configure(IApplicationBuilder app)
{
    var options = new RewriteOptions()
        .AddRedirectToHttpsPermanent();

    app.UseRewriter(options);
}
```

> [!NOTE]
> Per il reindirizzamento a un endpoint protetto quando non sono richieste regole di reindirizzamento aggiuntive, è consigliabile usare il middleware di reindirizzamento HTTPS. Per altre informazioni, vedere l'argomento [Imporre HTTPS](xref:security/enforcing-ssl#require-https).

L'app di esempio indica come usare `AddRedirectToHttps` o `AddRedirectToHttpsPermanent`. Aggiungere il metodo di estensione a `RewriteOptions`. Eseguire una richiesta non sicura all'app su qualsiasi URL. Ignorare l'avviso di sicurezza del browser indicante che il certificato autofirmato non è attendibile oppure creare un'eccezione per rendere affidabile il certificato.

Richiesta originale con `AddRedirectToHttps(301, 5001)`: `http://localhost:5000/secure`

![Finestra del browser con gli strumenti di sviluppo per il rilevamento di richieste e risposte](url-rewriting/_static/add_redirect_to_https.png)

Richiesta originale con `AddRedirectToHttpsPermanent`: `http://localhost:5000/secure`

![Finestra del browser con gli strumenti di sviluppo per il rilevamento di richieste e risposte](url-rewriting/_static/add_redirect_to_https_permanent.png)

### <a name="url-rewrite"></a>Riscrittura URL

Usare <xref:Microsoft.AspNetCore.Rewrite.RewriteOptionsExtensions.AddRewrite*> per creare una regola di riscrittura URL. Il primo parametro contiene l'espressione regolare per la corrispondenza in base al percorso dell'URL in ingresso. Il secondo parametro è la stringa di sostituzione. Il terzo parametro, `skipRemainingRules: {true|false}`, indica al middleware se ignorare le regole di riscrittura aggiuntive quando viene applicata la regola corrente.

[!code-csharp[](url-rewriting/samples/3.x/SampleApp/Startup.cs?name=snippet1&highlight=10-11)]

Richiesta originale: `/rewrite-rule/1234/5678`

![Finestra del browser con gli strumenti di sviluppo per il rilevamento di richiesta e risposta](url-rewriting/_static/add_rewrite.png)

L'accento circonflesso (`^`) all'inizio dell'espressione indica che la corrispondenza inizia all'inizio del percorso dell'URL.

Nell'esempio precedente della regola di reindirizzamento `redirect-rule/(.*)`, non è presente l'accento circonflesso (`^`) all'inizio dell'espressione regolare. È quindi possibile ottenere una corrispondenza anteponendo qualsiasi carattere a `redirect-rule/`.

| Percorso                               | Corrispondenza |
| ---------------------------------- | :---: |
| `/redirect-rule/1234/5678`         | Sì   |
| `/my-cool-redirect-rule/1234/5678` | Sì   |
| `/anotherredirect-rule/1234/5678`  | Sì   |

La regola di riscrittura, `^rewrite-rule/(\d+)/(\d+)`, rileva la corrispondenza dei percorsi solo se iniziano con `rewrite-rule/`. Nella tabella seguente, si noti la differenza nella corrispondenza.

| Percorso                              | Corrispondenza |
| --------------------------------- | :---: |
| `/rewrite-rule/1234/5678`         | Sì   |
| `/my-cool-rewrite-rule/1234/5678` | No    |
| `/anotherrewrite-rule/1234/5678`  | No    |

Dopo la parte `^rewrite-rule/` dell'espressione sono presenti due gruppi Capture, `(\d+)/(\d+)`. `\d` indica *corrispondenza con una cifra (numero)*. Il segno più (`+`) indica *corrispondenza con uno o più dei caratteri precedenti*. Di conseguenza l'URL deve contenere un numero seguito da una barra seguita a sua volta da un altro numero. Questi gruppi Capture vengono inseriti nell'URL riscritto come `$1` e `$2`. La stringa di sostituzione della regola di riscrittura inserisce i gruppi acquisiti nella stringa di query. Il percorso richiesto `/rewrite-rule/1234/5678` viene riscritto per ottenere la risorsa in `/rewritten?var1=1234&var2=5678`. Se nella richiesta originale è presente una stringa di query, la stringa viene mantenuta quando l'URL viene riscritto.

Non viene eseguito alcun round trip al server per ottenere la risorsa. Se la risorsa esiste, viene recuperata e restituita al client con il codice di stato *200 - OK*. Poiché il client non è reindirizzato, l'URL nella barra degli indirizzi del browser non cambia. I client non sono in grado di rilevare che si è verificata un'operazione di riscrittura URL nel server.

> [!NOTE]
> Se possibile, usare sempre `skipRemainingRules: true`, perché la corrispondenza tra le regole è onerosa dal punto di vista del calcolo e aumenta il tempo di risposta dell'app. Per velocizzare il tempo di risposta dell'app:
>
> * Ordinare le regole di riscrittura dalla regola che origina più corrispondenze a quella che origina meno corrispondenze.
> * Quando viene rilevata una corrispondenza e non sono necessarie altre operazioni di elaborazione delle regole, ignorare l'elaborazione delle regole rimanenti.

### <a name="apache-mod_rewrite"></a>Modulo Apache mod_rewrite

Applicare le regole del modulo Apache mod_rewrite con <xref:Microsoft.AspNetCore.Rewrite.ApacheModRewriteOptionsExtensions.AddApacheModRewrite*>. Assicurarsi che il file delle regole venga distribuito con l'app. Per altre informazioni ed esempi di regole mod_rewrite, vedere [Modulo Apache mod_rewrite](https://httpd.apache.org/docs/2.4/rewrite/).

<xref:System.IO.StreamReader>Viene usato per leggere le regole del file di regole *ApacheModRewrite.txt* :

[!code-csharp[](url-rewriting/samples/3.x/SampleApp/Startup.cs?name=snippet1&highlight=3-4,12)]

L'app di esempio reindirizza le richieste da `/apache-mod-rules-redirect/(.\*)` a `/redirected?id=$1`. Il codice di stato della risposta è *302 - Trovato*.

[!code[](url-rewriting/samples/3.x/SampleApp/ApacheModRewrite.txt)]

Richiesta originale: `/apache-mod-rules-redirect/1234`

![Finestra del browser con gli strumenti di sviluppo per il rilevamento di richieste e risposte](url-rewriting/_static/add_apache_mod_redirect.png)

Il middleware supporta le seguenti variabili server del modulo Apache mod_rewrite:

* CONN_REMOTE_ADDR
* HTTP_ACCEPT
* HTTP_CONNECTION
* HTTP_COOKIE
* HTTP_FORWARDED
* HTTP_HOST
* HTTP_REFERER
* HTTP_USER_AGENT
* HTTPS
* IPV6
* QUERY_STRING
* REMOTE_ADDR
* REMOTE_PORT
* REQUEST_FILENAME
* REQUEST_METHOD
* REQUEST_SCHEME
* REQUEST_URI
* SCRIPT_FILENAME
* SERVER_ADDR
* SERVER_PORT
* SERVER_PROTOCOL
* TIME
* TIME_DAY
* TIME_HOUR
* TIME_MIN
* TIME_MON
* TIME_SEC
* TIME_WDAY
* TIME_YEAR

### <a name="iis-url-rewrite-module-rules"></a>Regole di IIS URL Rewrite Module

Per usare lo stesso set di regole applicabile a IIS URL Rewrite Module, usare <xref:Microsoft.AspNetCore.Rewrite.IISUrlRewriteOptionsExtensions.AddIISUrlRewrite*>. Assicurarsi che il file delle regole venga distribuito con l'app. Non indirizzare il middleware all'uso del file *web.config* dell'app quando è in esecuzione in Windows Server IIS. Con IIS queste regole devono essere archiviate al di fuori del file *web.config* dell'app, per evitare conflitti con il modulo IIS Rewrite. Per altre informazioni ed esempi di regole di IIS URL Rewrite Module, vedere [Using URL Rewrite Module 2.0](/iis/extensions/url-rewrite-module/using-url-rewrite-module-20) (Uso di URL Rewrite Module 2.0) e [URL Rewrite Module Configuration Reference](/iis/extensions/url-rewrite-module/url-rewrite-module-configuration-reference) (Guida di riferimento per la configurazione di URL Rewrite Module).

<xref:System.IO.StreamReader>Viene usato per leggere le regole del file di regole *IISUrlRewrite.xml* :

[!code-csharp[](url-rewriting/samples/3.x/SampleApp/Startup.cs?name=snippet1&highlight=5-6,13)]

L'app di esempio riscrive le richieste da `/iis-rules-rewrite/(.*)` a `/rewritten?id=$1`. La risposta viene inviata al client con il codice di stato *200 - OK*.

[!code-xml[](url-rewriting/samples/3.x/SampleApp/IISUrlRewrite.xml)]

Richiesta originale: `/iis-rules-rewrite/1234`

![Finestra del browser con gli strumenti di sviluppo per il rilevamento di richiesta e risposta](url-rewriting/_static/add_iis_url_rewrite.png)

Se è presente un modulo IIS Rewrite attivo con regole di livello server configurate che possono avere effetti indesiderati nell'app, è possibile disabilitare il modulo IIS Rewrite per un'app. Per altre informazioni, vedere [Disabling IIS modules](xref:host-and-deploy/iis/modules#disabling-iis-modules) (Disabilitazione di moduli IIS).

#### <a name="unsupported-features"></a>Funzionalità non supportate

Il middleware non supporta le seguenti funzionalità di IIS URL Rewrite Module:

* Regole in uscita
* Variabili server personalizzate
* Caratteri jolly
* LogRewrittenUrl

#### <a name="supported-server-variables"></a>Variabili server supportate

Il middleware supporta le seguenti variabili server di IIS URL Rewrite Module:

* CONTENT_LENGTH
* CONTENT_TYPE
* HTTP_ACCEPT
* HTTP_CONNECTION
* HTTP_COOKIE
* HTTP_HOST
* HTTP_REFERER
* HTTP_URL
* HTTP_USER_AGENT
* HTTPS
* LOCAL_ADDR
* QUERY_STRING
* REMOTE_ADDR
* REMOTE_PORT
* REQUEST_FILENAME
* REQUEST_URI

> [!NOTE]
> È anche possibile ottenere un <xref:Microsoft.Extensions.FileProviders.IFileProvider> tramite un <xref:Microsoft.Extensions.FileProviders.PhysicalFileProvider>. Questo approccio può garantire maggior flessibilità per la posizione dei file delle regole di riscrittura. Assicurarsi che i file di regole di riscrittura vengano distribuiti nel server in corrispondenza del percorso specificato.
>
> ```csharp
> PhysicalFileProvider fileProvider = new PhysicalFileProvider(Directory.GetCurrentDirectory());
> ```

### <a name="method-based-rule"></a>Regola basata su metodo

Usare <xref:Microsoft.AspNetCore.Rewrite.RewriteOptionsExtensions.Add*> per implementare logica della regola personalizzata in un metodo. `Add` espone <xref:Microsoft.AspNetCore.Rewrite.RewriteContext>, che rende disponibile <xref:Microsoft.AspNetCore.Http.HttpContext> per l'uso nel metodo. [RewriteContext.Result](xref:Microsoft.AspNetCore.Rewrite.RewriteContext.Result*) determina la modalità di gestione dell'elaborazione pipeline aggiuntiva. Impostare il valore su uno dei campi <xref:Microsoft.AspNetCore.Rewrite.RuleResult> descritti nella tabella seguente.

| Riscrivi risultato contesto               | Azione                                                           |
| ------------------------------------ | ---------------------------------------------------------------- |
| `RuleResult.ContinueRules` (impostazione predefinita) | Continuare ad applicare le regole.                                         |
| `RuleResult.EndResponse`             | Interrompere l'applicazione delle regole e inviare la risposta.                       |
| `RuleResult.SkipRemainingRules`      | Interrompere l'applicazione delle regole e inviare il contesto al middleware successivo. |

[!code-csharp[](url-rewriting/samples/3.x/SampleApp/Startup.cs?name=snippet1&highlight=14)]

L'app di esempio visualizza un metodo che reindirizza le richieste per i percorsi che terminano con *.xml*. Se viene inviata una richiesta per `/file.xml`, la richiesta viene reindirizzata a `/xmlfiles/file.xml`. Il codice di stato viene impostato su *301 - Spostato permanentemente*. Quando il browser invia una nuova richiesta per */xmlfiles/file.xml*, il middleware dei file statici rende disponibile il file al client dalla cartella *wwwroot/xmlfiles*. Per un reindirizzamento, impostare in modo esplicito il codice di stato della risposta. In caso contrario, viene restituito il codice di stato *200 - OK* e il reindirizzamento non viene eseguito nel client.

*RewriteRules.cs*:

[!code-csharp[](url-rewriting/samples/3.x/SampleApp/RewriteRules.cs?name=snippet_RedirectXmlFileRequests&highlight=14-18)]

Questo approccio consente anche di riscrivere le richieste. L'app di esempio illustra la riscrittura del percorso di una richiesta di file di testo per rendere disponibile il file di testo *file. txt* dalla cartella *wwwroot*. Il middleware dei file statici rende disponibile il file in base al percorso di richiesta aggiornato:

[!code-csharp[](url-rewriting/samples/3.x/SampleApp/Startup.cs?name=snippet1&highlight=15,22)]

*RewriteRules.cs*:

[!code-csharp[](url-rewriting/samples/3.x/SampleApp/RewriteRules.cs?name=snippet_RewriteTextFileRequests&highlight=7-8)]

### <a name="irule-based-rule"></a>Regola basata su IRule

Usare <xref:Microsoft.AspNetCore.Rewrite.RewriteOptionsExtensions.Add*> per usare la logica della regola in una classe che implementa l'interfaccia <xref:Microsoft.AspNetCore.Rewrite.IRule>. `IRule` garantisce maggiore flessibilità rispetto all'approccio con una regola basata su un metodo. La classe di implementazione può includere un costruttore, che consente di passare parametri per il metodo <xref:Microsoft.AspNetCore.Rewrite.IRule.ApplyRule*>.

[!code-csharp[](url-rewriting/samples/3.x/SampleApp/Startup.cs?name=snippet1&highlight=16-17)]

I valori dei parametri nell'app di esempio per `extension` e `newPath` vengono verificati e devono soddisfare diverse condizioni. `extension` deve contenere un valore e tale valore deve essere *.png*, *.jpg* o *.gif*. Se `newPath` non è valido, viene generato un evento <xref:System.ArgumentException>. Se viene inviata una richiesta per *image.png*, la richiesta viene reindirizzata a `/png-images/image.png`. Se viene inviata una richiesta per *image.jpg*, la richiesta viene reindirizzata a `/jpg-images/image.jpg`. Il codice di stato viene impostato su *301 - Spostato permanentemente* e l'elemento `context.Result` viene impostato per l'interruzione dell'elaborazione delle regole e l'invio della risposta.

[!code-csharp[](url-rewriting/samples/3.x/SampleApp/RewriteRules.cs?name=snippet_RedirectImageRequests)]

Richiesta originale: `/image.png`

![Finestra del browser con gli strumenti di sviluppo per il rilevamento di richieste e risposte per image.png](url-rewriting/_static/add_redirect_png_requests.png)

Richiesta originale: `/image.jpg`

![Finestra del browser con gli strumenti di sviluppo per il rilevamento di richieste e risposte per image.jpg](url-rewriting/_static/add_redirect_jpg_requests.png)

## <a name="regex-examples"></a>Esempi di espressioni regolari

| Obiettivo | Esempio di stringa espressione regolare<br>e corrispondenza | Esempio di stringa di sostituzione<br>e output |
| ---- | ------------------------------- | -------------------------------------- |
| Riscrivere il percorso nella stringa di query | `^path/(.*)/(.*)`<br>`/path/abc/123` | `path?var1=$1&var2=$2`<br>`/path?var1=abc&var2=123` |
| Rimuovere la barra finale | `(.*)/$`<br>`/path/` | `$1`<br>`/path` |
| Applicare la barra finale | `(.*[^/])$`<br>`/path` | `$1/`<br>`/path/` |
| Evitare la riscrittura di richieste specifiche | `^(.*)(?<!\.axd)$` o `^(?!.*\.axd$)(.*)$`<br>Sì: `/resource.htm`<br>No: `/resource.axd` | `rewritten/$1`<br>`/rewritten/resource.htm`<br>`/resource.axd` |
| Ridisporre i segmenti di URL | `path/(.*)/(.*)/(.*)`<br>`path/1/2/3` | `path/$3/$2/$1`<br>`path/3/2/1` |
| Sostituire un segmento di URL | `^(.*)/segment2/(.*)`<br>`/segment1/segment2/segment3` | `$1/replaced/$2`<br>`/segment1/replaced/segment3` |

::: moniker-end

::: moniker range="< aspnetcore-3.0"

Questo documento presenta la riscrittura URL con istruzioni per l'uso del middleware Riscrittura URL nelle applicazioni ASP.NET Core.

La riscrittura URL è l'azione di modificare gli URL di richiesta in base a una o più regole predefinite. Il processo di riscrittura URL crea un'astrazione tra i percorsi delle risorse e i relativi indirizzi, in modo che i percorsi e gli indirizzi non risultino strettamente collegati. La riscrittura URL risulta utile in diversi scenari per:

* Spostare o sostituire in modo temporaneo o permanente risorse server mantenendo localizzatori stabili di queste risorse.
* Suddividere l'elaborazione delle richieste tra app diverse o tra aree diverse di un'unica app.
* Rimuovere, aggiungere o riorganizzare segmenti URL nelle richieste in ingresso.
* Ottimizzare gli URL pubblici per l'ottimizzazione motore di ricerca (SEO, Search Engine Optimization).
* Consentire l'uso di URL pubblici descrittivi in modo che i visitatori possano prevedere il contenuto restituito dalla richiesta di una risorsa.
* Reindirizzare le richieste non protette a endpoint protetti.
* Evitare l'hotlinking, ovvero l'uso da parte di un sito esterno di una risorsa statica ospitata presente in un altro sito tramite il collegamento di questa al proprio contenuto.

> [!NOTE]
> La riscrittura URL può ridurre le prestazioni di un'app. Ove possibile, limitare il numero e la complessità delle regole.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/url-rewriting/samples/) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="url-redirect-and-url-rewrite"></a>Reindirizzamento URL e riscrittura URL

La differenza tra i termini *reindirizzamento URL* e *riscrittura URL* è minima, ma ha implicazioni importanti nell'offerta di risorse ai clienti. Il middleware Riscrittura URL di ASP.NET Core è in grado di svolgere entrambe le funzioni.

Un *reindirizzamento URL* implica un'operazione lato client, in cui viene richiesto al client di accedere a una risorsa a un indirizzo diverso da quello richiesto originariamente dal client. Questa operazione richiede un round trip al server. L'URL di reindirizzamento restituito al client viene visualizzato nella barra degli indirizzi del browser quando il client effettua una nuova richiesta per la risorsa.

In caso di *reindirizzamento* di `/resource` a `/different-resource`, il server risponde che il client deve ottenere la risorsa presso `/different-resource` con un codice di stato che indica se il reindirizzamento è temporaneo o permanente.

![Un endpoint di servizio WebAPI è stato modificato temporaneamente dalla versione 1 (v1) alla versione 2 (v2) nel server. Un client esegue una richiesta al servizio nel percorso della versione 1, ovvero /v1/api. Il server restituisce una risposta 302 (Trovato) con il nuovo percorso temporaneo del servizio versione 2, ovvero /v2/api. Il client esegue una seconda richiesta al servizio all'URL di reindirizzamento. Il server risponde con un codice di stato 200 (OK).](url-rewriting/_static/url_redirect.png)

Quando si reindirizzano le richieste a un URL diverso, indicare se il reindirizzamento è temporaneo o permanente specificando il codice di stato con la risposta:

* Il codice di stato *301 - Spostato permanentemente* viene usato se la risorsa ha un nuovo URL definitivo e si vuole indicare al client che tale URL dovrà essere usato per tutte le richieste future della risorsa. *Quando riceve un codice di stato 301, il client può memorizzare la risposta nella cache e riusarla.*

* Il codice di stato *302 - Trovato* viene usato quando il reindirizzamento è temporaneo o comunque soggetto a modifiche. Il codice di stato 302 indica che il client non dovrà archiviare e riusare l'URL di reindirizzamento in futuro.

Per altre informazioni sui codici di stato, vedere [RFC 2616: Status Code Definitions](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html) (RFC 2616: Definizioni dei codici di stato).

La *riscrittura URL* è un'operazione lato server che rende disponibile una risorsa da un indirizzo diverso da quello richiesto dal client. La riscrittura URL non richiede un round trip al server. L'URL riscritto non viene restituito al client e non viene visualizzato nella barra degli indirizzi del browser.

Se `/resource` viene *riscritto* in `/different-resource`, il server recupera *internamente* la risorsa in `/different-resource`.

Anche se il client fosse in grado di recuperare la risorsa nell'URL riscritto, non viene informato dell'esistenza della risorsa nell'URL riscritto quando effettua la richiesta e riceve la risposta.

![Un endpoint servizio WebAPI è stato modificato dalla versione 1 (v1) alla versione 2 (v2) sul server. Un client esegue una richiesta al servizio nel percorso della versione 1, ovvero /v1/api. L'URL della richiesta viene riscritto per accedere al servizio nel percorso della versione 2, ovvero /v2/api. Il servizio risponde al client con un codice di stato 200 (OK).](url-rewriting/_static/url_rewrite.png)

## <a name="url-rewriting-sample-app"></a>App di esempio per la riscrittura URL

È possibile esplorare le funzionalità del middleware Riscrittura URL con l'[app di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/url-rewriting/samples/). L'app applica regole di reindirizzamento e riscrittura e visualizza l'URL reindirizzato o riscritto per diversi scenari.

## <a name="when-to-use-url-rewriting-middleware"></a>Quando usare il middleware Riscrittura URL

Usare il middleware Riscrittura URL quando non è possibile usare gli approcci seguenti:

* [URL Rewrite Module for IIS in Windows Server](https://www.iis.net/downloads/microsoft/url-rewrite)
* [Modulo Apache mod_rewrite in Apache Server](https://httpd.apache.org/docs/2.4/rewrite/)
* [Riscrittura URL in Nginx](https://www.nginx.com/blog/creating-nginx-rewrite-rules/)

Usare il middleware anche se l'app è ospitata in un [server HTTP.sys](xref:fundamentals/servers/httpsys) (in precedenza denominato WebListener).

I motivi principali per usare tecnologie di riscrittura URL basate su server in IIS, Apache e Nginx sono:

* Il middleware non supporta tutte le funzionalità di questi moduli.

  Alcune funzionalità dei moduli server non funzionano con i progetti ASP.NET Core, ad esempio i vincoli `IsFile` e `IsDirectory` del modulo IIS Rewrite. In questi scenari è necessario usare il middleware.
* Le prestazioni del middleware probabilmente non corrispondono a quelle dei moduli.

  Il benchmarking rappresenta l'unico modo per sapere con certezza quale approccio comporta la maggior riduzione delle prestazioni o se tale riduzione è trascurabile.

## <a name="package"></a>Pacchetto

Per includere il middleware nel progetto, aggiungere un riferimento al pacchetto al [metapacchetto Microsoft.AspNetCore.App](xref:fundamentals/metapackage-app) nel file di progetto, che contiene il pacchetto [Microsoft.AspNetCore.Rewrite](https://www.nuget.org/packages/Microsoft.AspNetCore.Rewrite).

Se non si usa il metapacchetto `Microsoft.AspNetCore.App`, aggiungere un riferimento al progetto al pacchetto `Microsoft.AspNetCore.Rewrite`.

## <a name="extension-and-options"></a>Estensioni e opzioni

È possibile definire regole di riscrittura e reindirizzamento URL creando un'istanza della classe [RewriteOptions](xref:Microsoft.AspNetCore.Rewrite.RewriteOptions) con metodi di estensione per le singole regole di riscrittura. Concatenare più regole nell'ordine in cui si vuole che vengano elaborate. Le opzioni `RewriteOptions` vengono passate al middleware Riscrittura URL quando questo viene aggiunto alla pipeline della richiesta con <xref:Microsoft.AspNetCore.Builder.RewriteBuilderExtensions.UseRewriter*>:

[!code-csharp[](url-rewriting/samples/2.x/SampleApp/Startup.cs?name=snippet1)]

### <a name="redirect-non-www-to-www"></a>Reindirizzare non-www su www

Sono disponibili tre opzioni che consentono all'app di reindirizzare richieste non-`www` su `www`:

* <xref:Microsoft.AspNetCore.Rewrite.RewriteOptionsExtensions.AddRedirectToWwwPermanent*>: Reindirizza in modo permanente la richiesta al `www` sottodominio se la richiesta è non `www` . Reindirizza con un codice di stato [Status308PermanentRedirect](xref:Microsoft.AspNetCore.Http.StatusCodes.Status308PermanentRedirect).

* <xref:Microsoft.AspNetCore.Rewrite.RewriteOptionsExtensions.AddRedirectToWww*>: Reindirizza la richiesta al `www` sottodominio se la richiesta in ingresso è non `www` . Reindirizza con un codice di stato [Status307TemporaryRedirect](xref:Microsoft.AspNetCore.Http.StatusCodes.Status307TemporaryRedirect). Un overload consente di specificare il codice di stato per la risposta. Usare un campo della classe <xref:Microsoft.AspNetCore.Http.StatusCodes> per un'assegnazione di codice di stato.

### <a name="url-redirect"></a>Reindirizzamento URL

Per reindirizzare le richieste, usare <xref:Microsoft.AspNetCore.Rewrite.RewriteOptionsExtensions.AddRedirect*>. Il primo parametro contiene l'espressione regolare per la corrispondenza in base al percorso dell'URL in ingresso. Il secondo parametro è la stringa di sostituzione. Il terzo parametro, se presente, specifica il codice di stato. Se il codice di stato non è specificato, assume come valore predefinito *302 - Trovato*, che indica che la risorsa è stata temporaneamente spostata o sostituita.

[!code-csharp[](url-rewriting/samples/2.x/SampleApp/Startup.cs?name=snippet1&highlight=9)]

In un browser con gli strumenti di sviluppo abilitati, creare una richiesta all'app di esempio con il percorso `/redirect-rule/1234/5678`. L'espressione regolare definisce la corrispondenza con il percorso di richiesta in `redirect-rule/(.*)` e il percorso viene sostituito con `/redirected/1234/5678`. L'URL di reindirizzamento viene restituito al client con il codice di stato *302 - Trovato*. Il browser invia una nuova richiesta all'URL di reindirizzamento, che viene visualizzata nella barra degli indirizzi del browser. Poiché nessuna regola nell'app di esempio corrisponde all'URL di reindirizzamento:

* La seconda richiesta riceve una risposta *200 - OK* dall'app.
* Il corpo della risposta visualizza l'URL di reindirizzamento.

Quando un URL viene *reindirizzato*, viene eseguito un round trip al server.

> [!WARNING]
> Prestare attenzione quando si definiscono regole di reindirizzamento. Le regole di reindirizzamento vengono valutate in ogni richiesta all'app, anche dopo un reindirizzamento. È facile creare inavvertitamente un *ciclo di reindirizzamento infinito*.

Richiesta originale: `/redirect-rule/1234/5678`

![Finestra del browser con gli strumenti di sviluppo per il rilevamento di richieste e risposte](url-rewriting/_static/add_redirect.png)

La parte dell'espressione racchiusa tra parentesi è denominata *gruppo Capture*. Il punto (`.`) dell'espressione significa *corrispondenza con qualsiasi carattere*. L'asterisco (`*`) indica *corrispondenza con il carattere precedente per zero o più volte*. Di conseguenza gli ultimi due i segmenti di percorso dell'URL `1234/5678` vengono acquisiti tramite il gruppo Capture `(.*)`. Qualsiasi valore visualizzato nell'URL della richiesta dopo `redirect-rule/` viene acquisito da questo gruppo Capture.

Nella stringa di sostituzione i gruppi acquisiti vengono inseriti nella stringa con il simbolo di dollaro (`$`) seguito dal numero di sequenza dell'acquisizione. Il primo valore del gruppo Capture viene ottenuto con `$1`, il secondo con `$2` e la procedura continua in sequenza per i gruppi Capture presenti nell'espressione regolare. Nell'espressione regolare della regola di reindirizzamento dell'app di esempio è presente un solo gruppo acquisito, pertanto nella stringa di sostituzione è presente un solo gruppo inserito, ovvero `$1`. Quando viene applicata la regola, l'URL diventa `/redirected/1234/5678`.

### <a name="url-redirect-to-a-secure-endpoint"></a>Reindirizzamento URL a un endpoint protetto

Usare <xref:Microsoft.AspNetCore.Rewrite.RewriteOptionsExtensions.AddRedirectToHttps*> per reindirizzare le richieste HTTP allo stesso host e allo stesso percorso tramite il protocollo HTTPS. Se il codice di stato non viene specificato, il middleware usa come valore predefinito *302 - Trovato*. Se non viene specificata la porta:

* Il middleware usa come valore predefinito `null`.
* Lo schema viene modificato in `https` (protocollo HTTPS) e il client accede alla risorsa attraverso la porta 443.

L'esempio seguente illustra come impostare il codice di stato su *301 - Spostato permanentemente* e come passare alla porta 5001.

```csharp
public void Configure(IApplicationBuilder app)
{
    var options = new RewriteOptions()
        .AddRedirectToHttps(301, 5001);

    app.UseRewriter(options);
}
```

Usare <xref:Microsoft.AspNetCore.Rewrite.RewriteOptionsExtensions.AddRedirectToHttpsPermanent*> per reindirizzare le richieste non protette allo stesso host e allo stesso percorso con il protocollo protetto HTTPS attraverso la porta 443. Il middleware imposta il codice di stato su *301 - Spostato permanentemente*.

```csharp
public void Configure(IApplicationBuilder app)
{
    var options = new RewriteOptions()
        .AddRedirectToHttpsPermanent();

    app.UseRewriter(options);
}
```

> [!NOTE]
> Per il reindirizzamento a un endpoint protetto quando non sono richieste regole di reindirizzamento aggiuntive, è consigliabile usare il middleware di reindirizzamento HTTPS. Per altre informazioni, vedere l'argomento [Imporre HTTPS](xref:security/enforcing-ssl#require-https).

L'app di esempio indica come usare `AddRedirectToHttps` o `AddRedirectToHttpsPermanent`. Aggiungere il metodo di estensione a `RewriteOptions`. Eseguire una richiesta non sicura all'app su qualsiasi URL. Ignorare l'avviso di sicurezza del browser indicante che il certificato autofirmato non è attendibile oppure creare un'eccezione per rendere affidabile il certificato.

Richiesta originale con `AddRedirectToHttps(301, 5001)`: `http://localhost:5000/secure`

![Finestra del browser con gli strumenti di sviluppo per il rilevamento di richieste e risposte](url-rewriting/_static/add_redirect_to_https.png)

Richiesta originale con `AddRedirectToHttpsPermanent`: `http://localhost:5000/secure`

![Finestra del browser con gli strumenti di sviluppo per il rilevamento di richieste e risposte](url-rewriting/_static/add_redirect_to_https_permanent.png)

### <a name="url-rewrite"></a>Riscrittura URL

Usare <xref:Microsoft.AspNetCore.Rewrite.RewriteOptionsExtensions.AddRewrite*> per creare una regola di riscrittura URL. Il primo parametro contiene l'espressione regolare per la corrispondenza in base al percorso dell'URL in ingresso. Il secondo parametro è la stringa di sostituzione. Il terzo parametro, `skipRemainingRules: {true|false}`, indica al middleware se ignorare le regole di riscrittura aggiuntive quando viene applicata la regola corrente.

[!code-csharp[](url-rewriting/samples/2.x/SampleApp/Startup.cs?name=snippet1&highlight=10-11)]

Richiesta originale: `/rewrite-rule/1234/5678`

![Finestra del browser con gli strumenti di sviluppo per il rilevamento di richiesta e risposta](url-rewriting/_static/add_rewrite.png)

L'accento circonflesso (`^`) all'inizio dell'espressione indica che la corrispondenza inizia all'inizio del percorso dell'URL.

Nell'esempio precedente della regola di reindirizzamento `redirect-rule/(.*)`, non è presente l'accento circonflesso (`^`) all'inizio dell'espressione regolare. È quindi possibile ottenere una corrispondenza anteponendo qualsiasi carattere a `redirect-rule/`.

| Percorso                               | Corrispondenza |
| ---------------------------------- | :---: |
| `/redirect-rule/1234/5678`         | Sì   |
| `/my-cool-redirect-rule/1234/5678` | Sì   |
| `/anotherredirect-rule/1234/5678`  | Sì   |

La regola di riscrittura, `^rewrite-rule/(\d+)/(\d+)`, rileva la corrispondenza dei percorsi solo se iniziano con `rewrite-rule/`. Nella tabella seguente, si noti la differenza nella corrispondenza.

| Percorso                              | Corrispondenza |
| --------------------------------- | :---: |
| `/rewrite-rule/1234/5678`         | Sì   |
| `/my-cool-rewrite-rule/1234/5678` | No    |
| `/anotherrewrite-rule/1234/5678`  | No    |

Dopo la parte `^rewrite-rule/` dell'espressione sono presenti due gruppi Capture, `(\d+)/(\d+)`. `\d` indica *corrispondenza con una cifra (numero)*. Il segno più (`+`) indica *corrispondenza con uno o più dei caratteri precedenti*. Di conseguenza l'URL deve contenere un numero seguito da una barra seguita a sua volta da un altro numero. Questi gruppi Capture vengono inseriti nell'URL riscritto come `$1` e `$2`. La stringa di sostituzione della regola di riscrittura inserisce i gruppi acquisiti nella stringa di query. Il percorso richiesto `/rewrite-rule/1234/5678` viene riscritto per ottenere la risorsa in `/rewritten?var1=1234&var2=5678`. Se nella richiesta originale è presente una stringa di query, la stringa viene mantenuta quando l'URL viene riscritto.

Non viene eseguito alcun round trip al server per ottenere la risorsa. Se la risorsa esiste, viene recuperata e restituita al client con il codice di stato *200 - OK*. Poiché il client non è reindirizzato, l'URL nella barra degli indirizzi del browser non cambia. I client non sono in grado di rilevare che si è verificata un'operazione di riscrittura URL nel server.

> [!NOTE]
> Se possibile, usare sempre `skipRemainingRules: true`, perché la corrispondenza tra le regole è onerosa dal punto di vista del calcolo e aumenta il tempo di risposta dell'app. Per velocizzare il tempo di risposta dell'app:
>
> * Ordinare le regole di riscrittura dalla regola che origina più corrispondenze a quella che origina meno corrispondenze.
> * Quando viene rilevata una corrispondenza e non sono necessarie altre operazioni di elaborazione delle regole, ignorare l'elaborazione delle regole rimanenti.

### <a name="apache-mod_rewrite"></a>Modulo Apache mod_rewrite

Applicare le regole del modulo Apache mod_rewrite con <xref:Microsoft.AspNetCore.Rewrite.ApacheModRewriteOptionsExtensions.AddApacheModRewrite*>. Assicurarsi che il file delle regole venga distribuito con l'app. Per altre informazioni ed esempi di regole mod_rewrite, vedere [Modulo Apache mod_rewrite](https://httpd.apache.org/docs/2.4/rewrite/).

<xref:System.IO.StreamReader>Viene usato per leggere le regole del file di regole *ApacheModRewrite.txt* :

[!code-csharp[](url-rewriting/samples/2.x/SampleApp/Startup.cs?name=snippet1&highlight=3-4,12)]

L'app di esempio reindirizza le richieste da `/apache-mod-rules-redirect/(.\*)` a `/redirected?id=$1`. Il codice di stato della risposta è *302 - Trovato*.

[!code[](url-rewriting/samples/2.x/SampleApp/ApacheModRewrite.txt)]

Richiesta originale: `/apache-mod-rules-redirect/1234`

![Finestra del browser con gli strumenti di sviluppo per il rilevamento di richieste e risposte](url-rewriting/_static/add_apache_mod_redirect.png)

Il middleware supporta le seguenti variabili server del modulo Apache mod_rewrite:

* CONN_REMOTE_ADDR
* HTTP_ACCEPT
* HTTP_CONNECTION
* HTTP_COOKIE
* HTTP_FORWARDED
* HTTP_HOST
* HTTP_REFERER
* HTTP_USER_AGENT
* HTTPS
* IPV6
* QUERY_STRING
* REMOTE_ADDR
* REMOTE_PORT
* REQUEST_FILENAME
* REQUEST_METHOD
* REQUEST_SCHEME
* REQUEST_URI
* SCRIPT_FILENAME
* SERVER_ADDR
* SERVER_PORT
* SERVER_PROTOCOL
* TIME
* TIME_DAY
* TIME_HOUR
* TIME_MIN
* TIME_MON
* TIME_SEC
* TIME_WDAY
* TIME_YEAR

### <a name="iis-url-rewrite-module-rules"></a>Regole di IIS URL Rewrite Module

Per usare lo stesso set di regole applicabile a IIS URL Rewrite Module, usare <xref:Microsoft.AspNetCore.Rewrite.IISUrlRewriteOptionsExtensions.AddIISUrlRewrite*>. Assicurarsi che il file delle regole venga distribuito con l'app. Non indirizzare il middleware all'uso del file *web.config* dell'app quando è in esecuzione in Windows Server IIS. Con IIS queste regole devono essere archiviate al di fuori del file *web.config* dell'app, per evitare conflitti con il modulo IIS Rewrite. Per altre informazioni ed esempi di regole di IIS URL Rewrite Module, vedere [Using URL Rewrite Module 2.0](/iis/extensions/url-rewrite-module/using-url-rewrite-module-20) (Uso di URL Rewrite Module 2.0) e [URL Rewrite Module Configuration Reference](/iis/extensions/url-rewrite-module/url-rewrite-module-configuration-reference) (Guida di riferimento per la configurazione di URL Rewrite Module).

<xref:System.IO.StreamReader>Viene usato per leggere le regole del file di regole *IISUrlRewrite.xml* :

[!code-csharp[](url-rewriting/samples/2.x/SampleApp/Startup.cs?name=snippet1&highlight=5-6,13)]

L'app di esempio riscrive le richieste da `/iis-rules-rewrite/(.*)` a `/rewritten?id=$1`. La risposta viene inviata al client con il codice di stato *200 - OK*.

[!code-xml[](url-rewriting/samples/2.x/SampleApp/IISUrlRewrite.xml)]

Richiesta originale: `/iis-rules-rewrite/1234`

![Finestra del browser con gli strumenti di sviluppo per il rilevamento di richiesta e risposta](url-rewriting/_static/add_iis_url_rewrite.png)

Se è presente un modulo IIS Rewrite attivo con regole di livello server configurate che possono avere effetti indesiderati nell'app, è possibile disabilitare il modulo IIS Rewrite per un'app. Per altre informazioni, vedere [Disabling IIS modules](xref:host-and-deploy/iis/modules#disabling-iis-modules) (Disabilitazione di moduli IIS).

#### <a name="unsupported-features"></a>Funzionalità non supportate

Il middleware rilasciato con ASP.NET Core 2.x non supporta le seguenti funzionalità di IIS URL Rewrite Module:

* Regole in uscita
* Variabili server personalizzate
* Caratteri jolly
* LogRewrittenUrl

#### <a name="supported-server-variables"></a>Variabili server supportate

Il middleware supporta le seguenti variabili server di IIS URL Rewrite Module:

* CONTENT_LENGTH
* CONTENT_TYPE
* HTTP_ACCEPT
* HTTP_CONNECTION
* HTTP_COOKIE
* HTTP_HOST
* HTTP_REFERER
* HTTP_URL
* HTTP_USER_AGENT
* HTTPS
* LOCAL_ADDR
* QUERY_STRING
* REMOTE_ADDR
* REMOTE_PORT
* REQUEST_FILENAME
* REQUEST_URI

> [!NOTE]
> È anche possibile ottenere un <xref:Microsoft.Extensions.FileProviders.IFileProvider> tramite un <xref:Microsoft.Extensions.FileProviders.PhysicalFileProvider>. Questo approccio può garantire maggior flessibilità per la posizione dei file delle regole di riscrittura. Assicurarsi che i file di regole di riscrittura vengano distribuiti nel server in corrispondenza del percorso specificato.
>
> ```csharp
> PhysicalFileProvider fileProvider = new PhysicalFileProvider(Directory.GetCurrentDirectory());
> ```

### <a name="method-based-rule"></a>Regola basata su metodo

Usare <xref:Microsoft.AspNetCore.Rewrite.RewriteOptionsExtensions.Add*> per implementare logica della regola personalizzata in un metodo. `Add` espone <xref:Microsoft.AspNetCore.Rewrite.RewriteContext>, che rende disponibile <xref:Microsoft.AspNetCore.Http.HttpContext> per l'uso nel metodo. [RewriteContext.Result](xref:Microsoft.AspNetCore.Rewrite.RewriteContext.Result*) determina la modalità di gestione dell'elaborazione pipeline aggiuntiva. Impostare il valore su uno dei campi <xref:Microsoft.AspNetCore.Rewrite.RuleResult> descritti nella tabella seguente.

| Riscrivi risultato contesto               | Azione                                                           |
| ------------------------------------ | ---------------------------------------------------------------- |
| `RuleResult.ContinueRules` (impostazione predefinita) | Continuare ad applicare le regole.                                         |
| `RuleResult.EndResponse`             | Interrompere l'applicazione delle regole e inviare la risposta.                       |
| `RuleResult.SkipRemainingRules`      | Interrompere l'applicazione delle regole e inviare il contesto al middleware successivo. |

[!code-csharp[](url-rewriting/samples/2.x/SampleApp/Startup.cs?name=snippet1&highlight=14)]

L'app di esempio visualizza un metodo che reindirizza le richieste per i percorsi che terminano con *.xml*. Se viene inviata una richiesta per `/file.xml`, la richiesta viene reindirizzata a `/xmlfiles/file.xml`. Il codice di stato viene impostato su *301 - Spostato permanentemente*. Quando il browser invia una nuova richiesta per */xmlfiles/file.xml*, il middleware dei file statici rende disponibile il file al client dalla cartella *wwwroot/xmlfiles*. Per un reindirizzamento, impostare in modo esplicito il codice di stato della risposta. In caso contrario, viene restituito il codice di stato *200 - OK* e il reindirizzamento non viene eseguito nel client.

*RewriteRules.cs*:

[!code-csharp[](url-rewriting/samples/2.x/SampleApp/RewriteRules.cs?name=snippet_RedirectXmlFileRequests&highlight=14-18)]

Questo approccio consente anche di riscrivere le richieste. L'app di esempio illustra la riscrittura del percorso di una richiesta di file di testo per rendere disponibile il file di testo *file. txt* dalla cartella *wwwroot*. Il middleware dei file statici rende disponibile il file in base al percorso di richiesta aggiornato:

[!code-csharp[](url-rewriting/samples/2.x/SampleApp/Startup.cs?name=snippet1&highlight=15,22)]

*RewriteRules.cs*:

[!code-csharp[](url-rewriting/samples/2.x/SampleApp/RewriteRules.cs?name=snippet_RewriteTextFileRequests&highlight=7-8)]

### <a name="irule-based-rule"></a>Regola basata su IRule

Usare <xref:Microsoft.AspNetCore.Rewrite.RewriteOptionsExtensions.Add*> per usare la logica della regola in una classe che implementa l'interfaccia <xref:Microsoft.AspNetCore.Rewrite.IRule>. `IRule` garantisce maggiore flessibilità rispetto all'approccio con una regola basata su un metodo. La classe di implementazione può includere un costruttore, che consente di passare parametri per il metodo <xref:Microsoft.AspNetCore.Rewrite.IRule.ApplyRule*>.

[!code-csharp[](url-rewriting/samples/2.x/SampleApp/Startup.cs?name=snippet1&highlight=16-17)]

I valori dei parametri nell'app di esempio per `extension` e `newPath` vengono verificati e devono soddisfare diverse condizioni. `extension` deve contenere un valore e tale valore deve essere *.png*, *.jpg* o *.gif*. Se `newPath` non è valido, viene generato un evento <xref:System.ArgumentException>. Se viene inviata una richiesta per *image.png*, la richiesta viene reindirizzata a `/png-images/image.png`. Se viene inviata una richiesta per *image.jpg*, la richiesta viene reindirizzata a `/jpg-images/image.jpg`. Il codice di stato viene impostato su *301 - Spostato permanentemente* e l'elemento `context.Result` viene impostato per l'interruzione dell'elaborazione delle regole e l'invio della risposta.

[!code-csharp[](url-rewriting/samples/2.x/SampleApp/RewriteRules.cs?name=snippet_RedirectImageRequests)]

Richiesta originale: `/image.png`

![Finestra del browser con gli strumenti di sviluppo per il rilevamento di richieste e risposte per image.png](url-rewriting/_static/add_redirect_png_requests.png)

Richiesta originale: `/image.jpg`

![Finestra del browser con gli strumenti di sviluppo per il rilevamento di richieste e risposte per image.jpg](url-rewriting/_static/add_redirect_jpg_requests.png)

## <a name="regex-examples"></a>Esempi di espressioni regolari

| Obiettivo | Esempio di stringa espressione regolare<br>e corrispondenza | Esempio di stringa di sostituzione<br>e output |
| ---- | ------------------------------- | -------------------------------------- |
| Riscrivere il percorso nella stringa di query | `^path/(.*)/(.*)`<br>`/path/abc/123` | `path?var1=$1&var2=$2`<br>`/path?var1=abc&var2=123` |
| Rimuovere la barra finale | `(.*)/$`<br>`/path/` | `$1`<br>`/path` |
| Applicare la barra finale | `(.*[^/])$`<br>`/path` | `$1/`<br>`/path/` |
| Evitare la riscrittura di richieste specifiche | `^(.*)(?<!\.axd)$` o `^(?!.*\.axd$)(.*)$`<br>Sì: `/resource.htm`<br>No: `/resource.axd` | `rewritten/$1`<br>`/rewritten/resource.htm`<br>`/resource.axd` |
| Ridisporre i segmenti di URL | `path/(.*)/(.*)/(.*)`<br>`path/1/2/3` | `path/$3/$2/$1`<br>`path/3/2/1` |
| Sostituire un segmento di URL | `^(.*)/segment2/(.*)`<br>`/segment1/segment2/segment3` | `$1/replaced/$2`<br>`/segment1/replaced/segment3` |

::: moniker-end

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:fundamentals/startup>
* <xref:fundamentals/middleware/index>
* [Espressioni regolari in .NET](/dotnet/articles/standard/base-types/regular-expressions)
* [Linguaggio di espressioni regolari-riferimento rapido](/dotnet/articles/standard/base-types/quick-ref)
* [Modulo Apache mod_rewrite](https://httpd.apache.org/docs/2.4/rewrite/)
* [Using Url Rewrite Module 2.0 (for IIS)](/iis/extensions/url-rewrite-module/using-url-rewrite-module-20) (Uso di URL Rewrite Module 2.0 per IIS)
* [URL Rewrite Module Configuration Reference](/iis/extensions/url-rewrite-module/url-rewrite-module-configuration-reference) (Guida di riferimento per la configurazione di URL Rewrite Module)
* [Forum di IIS URL Rewrite Module](https://forums.iis.net/1152.aspx)
* [Keep a simple URL structure](https://support.google.com/webmasters/answer/76329?hl=en) (Mantenere una struttura URL semplice)
* [10 URL Rewriting Tips and Tricks](https://ruslany.net/2009/04/10-url-rewriting-tips-and-tricks/) (10 suggerimenti e consigli per la riscrittura URL)
* [To slash or not to slash](https://webmasters.googleblog.com/2010/04/to-slash-or-not-to-slash.html) (Uso del carattere barra)
