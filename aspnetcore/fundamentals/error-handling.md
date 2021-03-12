---
title: Gestire gli errori in ASP.NET Core
author: rick-anderson
description: Informazioni su come gestire gli errori nelle app ASP.NET Core.
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 12/05/2019
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
uid: fundamentals/error-handling
ms.openlocfilehash: 1687195bc5d80d50289db51a5e8ac6113ca8b05f
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102586124"
---
# <a name="handle-errors-in-aspnet-core"></a>Gestire gli errori in ASP.NET Core

::: moniker range=">= aspnetcore-5.0"

Di [Kirk Larkin](https://twitter.com/serpent5), [Tom Dykstra](https://github.com/tdykstra/)e [Steve Smith](https://ardalis.com/)

Questo articolo illustra gli approcci comuni per la gestione degli errori nelle app Web ASP.NET Core. Vedere <xref:web-api/handle-errors> per le API Web.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/error-handling/samples). ([Come scaricare](xref:index#how-to-download-a-sample)). La scheda rete di strumenti di sviluppo del browser F12 è utile quando si esegue il test dell'applicazione di esempio.

## <a name="developer-exception-page"></a>Pagina delle eccezioni per gli sviluppatori

In *Pagina delle eccezioni per gli sviluppatori* sono disponibili informazioni dettagliate sulle eccezioni delle richieste. I modelli di ASP.NET Core generano il codice seguente:

[!code-csharp[](error-handling/samples/5.x/ErrorHandlingSample/Startup.cs?name=snippet&highlight=3-6)]

Il codice evidenziato precedente Abilita la pagina delle eccezioni per gli sviluppatori quando l'app è in esecuzione nell' [ambiente di sviluppo](xref:fundamentals/environments).

I modelli si posizionano all' <xref:Microsoft.AspNetCore.Builder.DeveloperExceptionPageExtensions.UseDeveloperExceptionPage%2A> inizio della pipeline del middleware, in modo da poter intercettare le eccezioni generate nel middleware che segue.

Il codice precedente Abilita la pagina delle eccezioni per gli sviluppatori ***solo*** quando l'app è in esecuzione nell'ambiente di sviluppo. Le informazioni dettagliate sulle eccezioni non devono essere visualizzate pubblicamente quando l'app viene eseguita nell'ambiente di produzione. Per altre informazioni sulla configurazione di ambienti, vedere <xref:fundamentals/environments>.

La pagina delle eccezioni per sviluppatori include le informazioni seguenti sull'eccezione e la richiesta:

* Analisi dello stack
* Parametri della stringa di query se presenti
* Cookies se presente
* Intestazioni

## <a name="exception-handler-page"></a>Pagina del gestore di eccezioni

Per configurare una pagina di gestione degli errori personalizzata per l' [ambiente di produzione](xref:fundamentals/environments), chiamare <xref:Microsoft.AspNetCore.Builder.ExceptionHandlerExtensions.UseExceptionHandler%2A> . Questo middleware di gestione delle eccezioni:

* Intercetta e registra le eccezioni.
* Esegue nuovamente la richiesta in una pipeline alternativa usando il percorso indicato. La richiesta non viene eseguita nuovamente se la risposta è stata avviata. Il codice generato dal modello esegue nuovamente la richiesta utilizzando il `/Error` percorso.

Nell'esempio seguente, <xref:Microsoft.AspNetCore.Builder.ExceptionHandlerExtensions.UseExceptionHandler%2A> aggiunge il middleware di gestione delle eccezioni in ambienti non di sviluppo:

[!code-csharp[](error-handling/samples/2.x/ErrorHandlingSample/Startup.cs?name=snippet_DevPageAndHandlerPage&highlight=5-9)]

Il Razor modello di app pagine fornisce una pagina di errore (*cshtml*) e una <xref:Microsoft.AspNetCore.Mvc.RazorPages.PageModel> classe ( `ErrorModel` ) nella cartella *pagine* . Per un'app MVC, il modello di progetto include un `Error` metodo di azione e una visualizzazione degli errori per il controller Home.

Il middleware di gestione delle eccezioni esegue nuovamente la richiesta usando il metodo HTTP *originale* . Se un endpoint del gestore errori è limitato a un set specifico di metodi HTTP, viene eseguito solo per i metodi HTTP. Ad esempio, un'azione del controller MVC che usa l' `[HttpGet]` attributo viene eseguita solo per le richieste Get. Per assicurarsi che *tutte* le richieste raggiungano la pagina di gestione degli errori personalizzata, non limitarle a un set specifico di metodi HTTP.

Per gestire le eccezioni in modo diverso in base al metodo HTTP originale:

* Per le Razor pagine, creare più metodi del gestore. Utilizzare, ad esempio, `OnGet` per gestire le eccezioni Get e utilizzare `OnPost` per gestire le eccezioni post.
* Per MVC, applicare attributi di verbi HTTP a più azioni. Utilizzare, ad esempio, `[HttpGet]` per gestire le eccezioni Get e utilizzare `[HttpPost]` per gestire le eccezioni post.

Per consentire agli utenti non autenticati di visualizzare la pagina di gestione degli errori personalizzata, assicurarsi che supporti l'accesso anonimo.

### <a name="access-the-exception"></a>Accedere all'eccezione

Utilizzare <xref:Microsoft.AspNetCore.Diagnostics.IExceptionHandlerPathFeature> per accedere all'eccezione e al percorso della richiesta originale in un gestore degli errori. Il codice seguente aggiunge `ExceptionMessage` alle pagine predefinite */Error. cshtml. cs* generate dai modelli di ASP.NET Core:

[!code-csharp[](error-handling/samples/5.x/ErrorHandlingSample/Pages/Error.cshtml.cs?name=snippet)]

> [!WARNING]
> **Non** fornire informazioni sugli errori sensibili ai client. Fornire informazioni sugli errori rappresenta un rischio di sicurezza.

Per testare l'eccezione nell' [app di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/error-handling/samples/5.x):

* Impostare l'ambiente sull'ambiente di produzione.
* Rimuovere i commenti da `webBuilder.UseStartup<Startup>();` in *Program.cs*.
* Selezionare **attiva un'eccezione** nel Home page.

## <a name="exception-handler-lambda"></a>Espressione lambda per il gestore di eccezioni

Un'alternativa a una [pagina di gestione delle eccezioni personalizzata](#exception-handler-page) consiste nel fornire un'espressione lambda a <xref:Microsoft.AspNetCore.Builder.ExceptionHandlerExtensions.UseExceptionHandler%2A>. L'uso di un'espressione lambda consente l'accesso all'errore prima di restituire la risposta.

Il codice seguente usa un'espressione lambda per la gestione delle eccezioni:

[!code-csharp[](error-handling/samples/5.x/ErrorHandlingSample/StartupLambda.cs?name=snippet)]

<!-- 
In the preceding code, `await context.Response.WriteAsync(new string(' ', 512));` is added so the Internet Explorer browser displays the error message rather than an IE error message. For more information, see [this GitHub issue](https://github.com/dotnet/AspNetCore.Docs/issues/16144).
-->

> [!WARNING]
> **Non** fornire informazioni sugli errori sensibili da <xref:Microsoft.AspNetCore.Diagnostics.IExceptionHandlerFeature> o <xref:Microsoft.AspNetCore.Diagnostics.IExceptionHandlerPathFeature> ai client. Fornire informazioni sugli errori rappresenta un rischio di sicurezza.

Per testare l'espressione lambda di gestione delle eccezioni nell' [app di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/error-handling/samples/5.x):

* Impostare l'ambiente sull'ambiente di produzione.
* Rimuovere i commenti da `webBuilder.UseStartup<StartupLambda>();` in *Program.cs*.
* Selezionare **attiva un'eccezione** nel Home page.

## <a name="usestatuscodepages"></a>UseStatusCodePages

Per impostazione predefinita, un'app ASP.NET Core non fornisce una tabella codici di stato per i codici di stato di errore HTTP, ad esempio *404-non trovato*. Quando l'app rileva un codice di stato di errore HTTP 400-599 che non ha un corpo, restituisce il codice di stato e un corpo della risposta vuoto. Per specificare le tabelle codici di stato, usare il middleware tabelle codici di stato. Per abilitare i gestori di solo testo predefiniti per i codici di stato di errore comuni, chiamare <xref:Microsoft.AspNetCore.Builder.StatusCodePagesExtensions.UseStatusCodePages%2A> nel metodo `Startup.Configure`:

[!code-csharp[](error-handling/samples/5.x/ErrorHandlingSample/StartupUseStatusCodePages.cs?name=snippet&highlight=13)]

Chiamare `UseStatusCodePages` prima del middleware di gestione delle richieste. Ad esempio, chiamare `UseStatusCodePages` prima del middleware del file statico e del middleware degli endpoint.

Quando `UseStatusCodePages` non viene utilizzato, la navigazione a un URL senza un endpoint restituisce un messaggio di errore dipendente dal browser che indica che l'endpoint non è stato trovato. Ad esempio, passando a `Home/Privacy2` . Quando `UseStatusCodePages` viene chiamato il metodo, il browser restituisce:

```html
Status Code: 404; Not Found
```

`UseStatusCodePages` non viene in genere utilizzato nell'ambiente di produzione perché restituisce un messaggio che non è utile per gli utenti.

Per eseguire `UseStatusCodePages` il test nell' [app di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/error-handling/samples/5.x):

* Impostare l'ambiente sull'ambiente di produzione.
* Rimuovere i commenti da `webBuilder.UseStartup<StartupUseStatusCodePages>();` in *Program.cs*.
* Selezionare i collegamenti nel home page home page.

> [!NOTE]
> Il middleware delle tabelle codici di stato **non** rileva le eccezioni. Per fornire una pagina di gestione degli errori personalizzata, utilizzare la [pagina gestore eccezioni](#exception-handler-page).

### <a name="usestatuscodepages-with-format-string"></a>UseStatusCodePages con stringa di formato

Per personalizzare il tipo di contenuto della risposta e il testo, usare l'overload di <xref:Microsoft.AspNetCore.Builder.StatusCodePagesExtensions.UseStatusCodePages%2A> che accetta un tipo di contenuto e una stringa di formato:

[!code-csharp[](error-handling/samples/5.x/ErrorHandlingSample/StartupFormat.cs?name=snippet&highlight=13-14)]

Nel codice precedente, `{0}` è un segnaposto per il codice di errore.

`UseStatusCodePages` con una stringa di formato non viene in genere usata nell'ambiente di produzione perché restituisce un messaggio che non è utile per gli utenti.

Per eseguire `UseStatusCodePages` il test nell' [app di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/error-handling/samples/5.x), rimuovere i commenti da `webBuilder.UseStartup<StartupFormat>();` in *Program.cs*.

### <a name="usestatuscodepages-with-lambda"></a>UseStatusCodePages con espressione lambda

Per specificare la gestione degli errori personalizzata e il codice per la scrittura delle risposte, usare l'overload di <xref:Microsoft.AspNetCore.Builder.StatusCodePagesExtensions.UseStatusCodePages%2A> che accetta un'espressione lambda:

[!code-csharp[](error-handling/samples/5.x/ErrorHandlingSample/StartupStatusLambda.cs?name=snippet&highlight=13-20)]

`UseStatusCodePages` con un'espressione lambda non viene in genere usata nell'ambiente di produzione perché restituisce un messaggio che non è utile per gli utenti.

Per eseguire `UseStatusCodePages` il test nell' [app di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/error-handling/samples/5.x), rimuovere i commenti da `webBuilder.UseStartup<StartupStatusLambda>();` in *Program.cs*.

### <a name="usestatuscodepageswithredirects"></a>UseStatusCodePagesWithRedirects

Metodo di estensione <xref:Microsoft.AspNetCore.Builder.StatusCodePagesExtensions.UseStatusCodePagesWithRedirects%2A>:

* Invia un codice di stato [302 - Trovato](https://developer.mozilla.org/docs/Web/HTTP/Status/302) al client.
* Reindirizza il client all'endpoint di gestione degli errori fornito nel modello di URL. Nell'endpoint di gestione degli errori vengono in genere visualizzate informazioni sull'errore e viene restituito il protocollo HTTP 200.

[!code-csharp[](error-handling/samples/5.x/ErrorHandlingSample/StartupSCredirect.cs?name=snippet&highlight=13)]

Il modello di URL può includere un `{0}` segnaposto per il codice di stato, come illustrato nel codice precedente. Se il modello di URL inizia con `~` (tilde), `~` viene sostituito dall'app `PathBase` . Quando si specifica un endpoint nell'app, creare una visualizzazione o una Razor pagina MVC per l'endpoint. Per un Razor esempio di pagine, vedere [pages/MyStatusCode. cshtml](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/error-handling/samples/5.x/ErrorHandlingSample/Pages) nell' [app di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/error-handling/samples/5.x).

Questo metodo viene usato comunemente quando l'app:

* Deve reindirizzare il client a un endpoint diverso, in genere nei casi in cui un'altra app elabora l'errore. Per le app Web, la barra degli indirizzi del browser del client riflette l'endpoint reindirizzato.
* Non deve conservare e restituire il codice di stato originale con la risposta di reindirizzamento iniziale.

Per eseguire `UseStatusCodePages` il test nell' [app di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/error-handling/samples/5.x), rimuovere i commenti da `webBuilder.UseStartup<StartupSCredirect>();` in *Program.cs*.

### <a name="usestatuscodepageswithreexecute"></a>UseStatusCodePagesWithReExecute

Metodo di estensione <xref:Microsoft.AspNetCore.Builder.StatusCodePagesExtensions.UseStatusCodePagesWithReExecute%2A>:

* Restituisce il codice di stato originale al client.
* Genera il corpo della risposta eseguendo nuovamente la pipeline delle richieste con un percorso alternativo.

[!code-csharp[](error-handling/samples/5.x/ErrorHandlingSample/StartupSCreX.cs?name=snippet&highlight=13)]

Se viene specificato un endpoint all'interno dell'app, creare una visualizzazione o una Razor pagina MVC per l'endpoint. Assicurarsi `UseStatusCodePagesWithReExecute` che venga inserito prima `UseRouting` di in modo che sia possibile reindirizzare la richiesta alla pagina stato. Per un Razor esempio di pagine, vedere [pages/MyStatusCode2. cshtml](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/error-handling/samples/5.x/ErrorHandlingSample/Pages) nell' [app di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/error-handling/samples/5.x).

Questo metodo viene usato comunemente quando l'app deve:

* Elaborare la richiesta senza il reindirizzamento a un endpoint diverso. Per le app Web, la barra degli indirizzi del browser del client riflette l'endpoint richiesto in origine.
* Conservare e restituire il codice di stato originale con la risposta.

I modelli URL e stringa di query possono includere un segnaposto `{0}` per il codice di stato. Il modello di URL deve iniziare con `/` .

<!-- Review: removing this. The sample code doesn't use @page "{code?}"
If you want that, it should be @page "{code:int?}"
but that's not required. Original text follows:

When using a placeholder in the path, confirm that the endpoint can process the path segment. For example, a Razor Page for errors should accept the optional path segment value with the `@page` directive:

```cshtml
@page "{code?}"
```
-->

L'endpoint che elabora l'errore può ottenere l'URL originale che ha generato l'errore, come illustrato nell'esempio seguente:

[!code-csharp[](error-handling/samples/5.x/ErrorHandlingSample/Pages/MyStatusCode2.cshtml.cs?name=snippet)]

Per un Razor esempio di pagine, vedere [pages/MyStatusCode2. cshtml](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/error-handling/samples/5.x/ErrorHandlingSample/Pages) nell' [app di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/error-handling/samples/5.x).

Per eseguire `UseStatusCodePages` il test nell' [app di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/error-handling/samples/5.x), rimuovere i commenti da `webBuilder.UseStartup<StartupSCreX>();` in *Program.cs*.

## <a name="disable-status-code-pages"></a>Disabilitare le tabelle codici di stato

Per disabilitare le tabelle codici di stato per un controller MVC o un metodo di azione, usare l'attributo [[SkipStatusCodePages]](xref:Microsoft.AspNetCore.Mvc.SkipStatusCodePagesAttribute).

Per disabilitare le tabelle codici di stato per richieste specifiche in un Razor metodo del gestore di pagine o in un controller MVC, usare <xref:Microsoft.AspNetCore.Diagnostics.IStatusCodePagesFeature> :

[!code-csharp[](error-handling/samples/5.x/ErrorHandlingSample/Pages/Privacy.cshtml.cs?name=snippet)]

## <a name="exception-handling-code"></a>Codice di gestione delle eccezioni

Anche il codice nelle pagine di gestione delle eccezioni può generare eccezioni. Le pagine di errore di produzione devono essere testate accuratamente e prestare particolare attenzione per evitare di generare eccezioni proprie.
<!-- Review: original, which is not realistic 
 > It's often a good idea for production error pages to consist of purely static content.

 comments: - after you catch the exception, you need code to log the details and perhaps dynamically create a string with an error message. 
-->

### <a name="response-headers"></a>Intestazioni di risposta

Dopo l'invio delle intestazioni per una risposta:

* L'app non può modificare il codice di stato della risposta.
* Non è possibile eseguire pagine o gestori delle eccezioni. La risposta deve essere completata o la connessione deve essere interrotta.

## <a name="server-exception-handling"></a>Gestione delle eccezioni del server

Oltre alla logica di gestione delle eccezioni in un'app, l' [implementazione del server http](xref:fundamentals/servers/index) può gestire alcune eccezioni. Se il server rileva un'eccezione prima dell'invio delle intestazioni di risposta, il server invia una `500 - Internal Server Error` risposta senza il corpo della risposta. Se il server rileva un'eccezione dopo l'invio delle intestazioni di risposta, il server chiude la connessione. Le richieste che non vengono gestite dall'app vengono gestite dal server. Qualsiasi eccezione che si verifica quando il server sta gestendo la richiesta viene gestita dalla gestione delle eccezioni del server. Eventuali pagine di errore personalizzate dell'app, middleware di gestione delle eccezioni e filtri non hanno effetto su questo comportamento.

## <a name="startup-exception-handling"></a>Gestione delle eccezioni durante l'avvio

Solo il livello di hosting può gestire le eccezioni che si verificano durante l'avvio dell'app. L'host può essere configurato per [acquisire gli errori di avvio](xref:fundamentals/host/web-host#capture-startup-errors) e [acquisire errori dettagliati](xref:fundamentals/host/web-host#detailed-errors).

Il livello di hosting può visualizzare una pagina di errore per un errore di avvio acquisito solo se l'errore si verifica dopo l'associazione indirizzo host/porta. Se si verifica un errore di associazione:

* Il livello di hosting registra un'eccezione critica.
* Si verifica un arresto anomalo del processo di dotnet.
* Non viene visualizzata alcuna pagina di errore quando il server HTTP è [Kestrel](xref:fundamentals/servers/kestrel).

Se durante l'esecuzione in [IIS](/iis) (o servizio app di Azure) o in [IIS Express](/iis/extensions/introduction-to-iis-express/iis-express-overview) il processo non può essere avviato, viene restituito l'errore *502.5 Errore del processo* dal [modulo ASP.NET Core](xref:host-and-deploy/aspnet-core-module). Per altre informazioni, vedere <xref:test/troubleshoot-azure-iis>.

## <a name="database-error-page"></a>Pagina di errore di database

Il filtro eccezioni pagina sviluppatore database `AddDatabaseDeveloperPageExceptionFilter` acquisisce le eccezioni correlate al database che possono essere risolte utilizzando Entity Framework Core migrazioni. Quando si verificano queste eccezioni, viene generata una risposta HTML con i dettagli delle azioni possibili per risolvere il problema. Questa pagina è abilitata solo nell'ambiente di sviluppo. Il codice seguente è stato generato dal ASP.NET Core Razor modelli di pagine quando sono stati specificati singoli account utente:

[!code-csharp[](error-handling/samples/5.x/StartupDBexFilter.cs?name=snippet&highlight=6)]

## <a name="exception-filters"></a>Filtri eccezioni

Nelle app MVC i filtri delle eccezioni possono essere configurati a livello globale o per ogni controller o azione singolarmente. Nelle Razor app Pages possono essere configurate a livello globale o per modello di pagina. Questi filtri gestiscono le eventuali eccezioni non gestite che si verificano durante l'esecuzione di un'azione del controller o di un altro filtro. Per altre informazioni, vedere <xref:mvc/controllers/filters#exception-filters>.

I filtri eccezioni sono utili per intercettare le eccezioni che si verificano all'interno di azioni MVC, ma non sono flessibili come il [middleware di gestione delle eccezioni](https://github.com/dotnet/aspnetcore/blob/main/src/Middleware/Diagnostics/src/ExceptionHandler/ExceptionHandlerMiddleware.cs)incorporato, `UseExceptionHandler` . È consigliabile usare `UseExceptionHandler` , a meno che non sia necessario eseguire la gestione degli errori in modo diverso in base all'azione MVC scelta.

[!code-csharp[](error-handling/samples/5.x/ErrorHandlingSample/Startup.cs?name=snippet&highlight=9)]

## <a name="model-state-errors"></a>Errori di stato del modello

Per informazioni su come gestire gli errori dello stato del modello, vedere [Associazione di modelli](xref:mvc/models/model-binding) e [Convalida del modello](xref:mvc/models/validation).

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:test/troubleshoot-azure-iis>
* <xref:host-and-deploy/azure-iis-errors-reference>

::: moniker-end

::: moniker range="< aspnetcore-5.0"

Di  [Tom Dykstra](https://github.com/tdykstra/)e [Steve Smith](https://ardalis.com/)

Questo articolo illustra gli approcci comuni per la gestione degli errori nelle app Web ASP.NET Core. Vedere <xref:web-api/handle-errors> per le API Web.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/error-handling/samples). ([Come scaricare un esempio](xref:index#how-to-download-a-sample).)

## <a name="developer-exception-page"></a>Pagina delle eccezioni per gli sviluppatori

In *Pagina delle eccezioni per gli sviluppatori* sono disponibili informazioni dettagliate sulle eccezioni delle richieste. I modelli di ASP.NET Core generano il codice seguente:

[!code-csharp[](error-handling/samples/2.x/ErrorHandlingSample/Startup.cs?name=snippet_DevPageAndHandlerPage&highlight=1-4)]

Il codice precedente Abilita la pagina delle eccezioni per gli sviluppatori quando l'app è in esecuzione nell' [ambiente di sviluppo](xref:fundamentals/environments).

I modelli si posizionano <xref:Microsoft.AspNetCore.Builder.DeveloperExceptionPageExtensions.UseDeveloperExceptionPage%2A> prima di qualsiasi middleware, in modo che le eccezioni vengano rilevate nel middleware che segue.

Il codice precedente Abilita la pagina delle eccezioni **per gli sviluppatori solo quando l'app è in esecuzione nell'ambiente di sviluppo**. Le informazioni dettagliate sulle eccezioni non devono essere visualizzate pubblicamente quando l'app viene eseguita nell'ambiente di produzione. Per altre informazioni sulla configurazione di ambienti, vedere <xref:fundamentals/environments>.

La pagina delle eccezioni per sviluppatori include le informazioni seguenti sull'eccezione e la richiesta:

* Analisi dello stack
* Parametri della stringa di query se presenti
* Cookies se presente
* Intestazioni

## <a name="exception-handler-page"></a>Pagina del gestore di eccezioni

Per configurare una pagina di gestione degli errori personalizzata per l'ambiente di produzione, usare il middleware di gestione delle eccezioni. Il middleware:

* Intercetta e registra le eccezioni.
* Esegue nuovamente la richiesta in una pipeline alternativa per la pagina o il controller indicati. La richiesta non viene eseguita nuovamente se la risposta è stata avviata. Il codice generato dal modello esegue nuovamente la richiesta a `/Error` .

Nell'esempio seguente <xref:Microsoft.AspNetCore.Builder.ExceptionHandlerExtensions.UseExceptionHandler%2A> aggiunge il middleware di gestione delle eccezioni in ambienti non di sviluppo:

[!code-csharp[](error-handling/samples/2.x/ErrorHandlingSample/Startup.cs?name=snippet_DevPageAndHandlerPage&highlight=5-9)]

Il Razor modello di app pagine fornisce una pagina di errore (*cshtml*) e una <xref:Microsoft.AspNetCore.Mvc.RazorPages.PageModel> classe ( `ErrorModel` ) nella cartella *pagine* . Per un'app MVC, il modello di progetto include un metodo di azione di errore e una visualizzazione degli errori nel controller Home.

Non contrassegnare il metodo di azione del gestore errori con attributi di metodo HTTP, ad esempio `HttpGet` . I verbi espliciti impediscono ad alcune richieste di raggiungere il metodo. Consentire l'accesso anonimo al metodo se gli utenti non autenticati dovrebbero visualizzare la visualizzazione degli errori.

### <a name="access-the-exception"></a>Accedere all'eccezione

Usare <xref:Microsoft.AspNetCore.Diagnostics.IExceptionHandlerPathFeature> per accedere all'eccezione e al percorso della richiesta originale in un controller o in una pagina del gestore degli errori:

[!code-csharp[](error-handling/samples/2.x/ErrorHandlingSample/Pages/MyFolder/Error.cshtml.cs?name=snippet_ExceptionHandlerPathFeature&3,7)]

> [!WARNING]
> **Non** fornire informazioni sugli errori sensibili ai client. Fornire informazioni sugli errori rappresenta un rischio di sicurezza.

Per attivare la pagina di gestione delle eccezioni precedente, impostare l'ambiente sulle produzioni e forzare un'eccezione.

## <a name="exception-handler-lambda"></a>Espressione lambda per il gestore di eccezioni

Un'alternativa a una [pagina di gestione delle eccezioni personalizzata](#exception-handler-page) consiste nel fornire un'espressione lambda a <xref:Microsoft.AspNetCore.Builder.ExceptionHandlerExtensions.UseExceptionHandler%2A>. L'uso di un'espressione lambda consente l'accesso all'errore prima di restituire la risposta.

Di seguito è riportato un esempio dell'uso di un'espressione lambda per la gestione delle eccezioni:

[!code-csharp[](error-handling/samples/2.x/ErrorHandlingSample/Startup.cs?name=snippet_HandlerPageLambda)]

Nel codice precedente `await context.Response.WriteAsync(new string(' ', 512));` viene aggiunto, in modo che il browser Internet Explorer visualizzi il messaggio di errore anziché un messaggio di errore di IE. Per altre informazioni, vedere [questo problema di GitHub](https://github.com/dotnet/AspNetCore.Docs/issues/16144).

> [!WARNING]
> **Non** fornire informazioni sugli errori sensibili da <xref:Microsoft.AspNetCore.Diagnostics.IExceptionHandlerFeature> o <xref:Microsoft.AspNetCore.Diagnostics.IExceptionHandlerPathFeature> ai client. Fornire informazioni sugli errori rappresenta un rischio di sicurezza.

Per visualizzare il risultato dell'espressione lambda di gestione delle eccezioni nell'[app di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/error-handling/samples), usare le direttive del preprocessore `ProdEnvironment` e `ErrorHandlerLambda` e selezionare **Trigger an exception** (Attiva un'eccezione) nella home page.

## <a name="usestatuscodepages"></a>UseStatusCodePages

Per impostazione predefinita, un'app ASP.NET Core non offre una tabella codici di stato per i codici di stato HTTP, ad esempio *404 - Non trovato*. L'app restituisce un codice di stato e un corpo della risposta vuoto. Per specificare le tabelle codici di stato, usare il middleware delle tabelle codici di stato.

Il middleware viene reso disponibile dal pacchetto [Microsoft. AspNetCore. Diagnostics](https://www.nuget.org/packages/Microsoft.AspNetCore.Diagnostics/) .

Per abilitare i gestori di solo testo predefiniti per i codici di stato di errore comuni, chiamare <xref:Microsoft.AspNetCore.Builder.StatusCodePagesExtensions.UseStatusCodePages%2A> nel metodo `Startup.Configure`:

[!code-csharp[](error-handling/samples/2.x/ErrorHandlingSample/Startup.cs?name=snippet_StatusCodePages)]

Chiamare `UseStatusCodePages` prima del middleware di gestione delle richieste (ad esempio, middleware dei file statici e middleware MVC).

Quando `UseStatusCodePages` non viene utilizzato, la navigazione a un URL senza un endpoint restituisce un messaggio di errore dipendente dal browser che indica che l'endpoint non è stato trovato. Ad esempio, passando a `Home/Privacy2` . Quando `UseStatusCodePages` viene chiamato il metodo, il browser restituisce:

```
Status Code: 404; Not Found
```

## <a name="usestatuscodepages-with-format-string"></a>UseStatusCodePages con stringa di formato

Per personalizzare il tipo di contenuto della risposta e il testo, usare l'overload di <xref:Microsoft.AspNetCore.Builder.StatusCodePagesExtensions.UseStatusCodePages%2A> che accetta un tipo di contenuto e una stringa di formato:

[!code-csharp[](error-handling/samples/2.x/ErrorHandlingSample/Startup.cs?name=snippet_StatusCodePagesFormatString)]

## <a name="usestatuscodepages-with-lambda"></a>UseStatusCodePages con espressione lambda

Per specificare la gestione degli errori personalizzata e il codice per la scrittura delle risposte, usare l'overload di <xref:Microsoft.AspNetCore.Builder.StatusCodePagesExtensions.UseStatusCodePages%2A> che accetta un'espressione lambda:

[!code-csharp[](error-handling/samples/2.x/ErrorHandlingSample/Startup.cs?name=snippet_StatusCodePagesLambda)]

## <a name="usestatuscodepageswithredirects"></a>UseStatusCodePagesWithRedirects

Metodo di estensione <xref:Microsoft.AspNetCore.Builder.StatusCodePagesExtensions.UseStatusCodePagesWithRedirects%2A>:

* Invia un codice di stato *302 - Trovato* al client.
* Reindirizza il client al percorso specificato nel modello di URL.

[!code-csharp[](error-handling/samples/2.x/ErrorHandlingSample/Startup.cs?name=snippet_StatusCodePagesWithRedirect)]

Il modello di URL può includere un segnaposto `{0}` per il codice di stato, come illustrato nell'esempio. Se il modello di URL inizia con `~` (tilde), `~` viene sostituito dall'app `PathBase` . Se si punta a un endpoint all'interno dell'app, creare una visualizzazione o una Razor pagina MVC per l'endpoint. Per un Razor esempio di pagine, vedere *pages/StatusCode. cshtml* nell' [app di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/error-handling/samples).

Questo metodo viene usato comunemente quando l'app:

* Deve reindirizzare il client a un endpoint diverso, in genere nei casi in cui un'altra app elabora l'errore. Per le app Web, la barra degli indirizzi del browser del client riflette l'endpoint reindirizzato.
* Non deve conservare e restituire il codice di stato originale con la risposta di reindirizzamento iniziale.

## <a name="usestatuscodepageswithreexecute"></a>UseStatusCodePagesWithReExecute

Metodo di estensione <xref:Microsoft.AspNetCore.Builder.StatusCodePagesExtensions.UseStatusCodePagesWithReExecute%2A>:

* Restituisce il codice di stato originale al client.
* Genera il corpo della risposta eseguendo nuovamente la pipeline delle richieste con un percorso alternativo.

[!code-csharp[](error-handling/samples/2.x/ErrorHandlingSample/Startup.cs?name=snippet_StatusCodePagesWithReExecute)]

Se si punta a un endpoint all'interno dell'app, creare una visualizzazione o una Razor pagina MVC per l'endpoint. Assicurarsi `UseStatusCodePagesWithReExecute` che venga inserito prima `UseRouting` di in modo che sia possibile reindirizzare la richiesta alla pagina stato. Per un Razor esempio di pagine, vedere *pages/StatusCode. cshtml* nell' [app di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/error-handling/samples).

Questo metodo viene usato comunemente quando l'app deve:

* Elaborare la richiesta senza il reindirizzamento a un endpoint diverso. Per le app Web, la barra degli indirizzi del browser del client riflette l'endpoint richiesto in origine.
* Conservare e restituire il codice di stato originale con la risposta.

I modelli di URL e di stringa di query potrebbero includere un segnaposto (`{0}`) per il codice di stato. Il modello di URL deve iniziare con una barra (`/`). Quando si usa un segnaposto nel percorso, verificare che l'endpoint (pagina o controller) possa elaborare il segmento del percorso. Ad esempio, una Razor pagina per gli errori deve accettare il valore del segmento di percorso facoltativo con la `@page` direttiva:

```cshtml
@page "{code?}"
```

L'endpoint che elabora l'errore può ottenere l'URL originale che ha generato l'errore, come illustrato nell'esempio seguente:

[!code-csharp[](error-handling/samples/2.x/ErrorHandlingSample/Pages/StatusCode.cshtml.cs?name=snippet_StatusCodeReExecute)]

Non contrassegnare il metodo di azione del gestore errori con attributi di metodo HTTP, ad esempio `HttpGet` . I verbi espliciti impediscono ad alcune richieste di raggiungere il metodo. Consentire l'accesso anonimo al metodo se gli utenti non autenticati dovrebbero visualizzare la visualizzazione degli errori.

## <a name="disable-status-code-pages"></a>Disabilitare le tabelle codici di stato

Per disabilitare le tabelle codici di stato per un controller MVC o un metodo di azione, usare l' [`[SkipStatusCodePages]`](xref:Microsoft.AspNetCore.Mvc.SkipStatusCodePagesAttribute) attributo.

Per disabilitare le tabelle codici di stato per richieste specifiche in un Razor metodo del gestore di pagine o in un controller MVC, usare <xref:Microsoft.AspNetCore.Diagnostics.IStatusCodePagesFeature> :

```csharp
var statusCodePagesFeature = HttpContext.Features.Get<IStatusCodePagesFeature>();

if (statusCodePagesFeature != null)
{
    statusCodePagesFeature.Enabled = false;
}
```

## <a name="exception-handling-code"></a>Codice di gestione delle eccezioni

Il codice in pagine di gestione delle eccezioni può generare eccezioni. È spesso consigliabile che le pagine di errore di produzione contengano esclusivamente contenuto statico.

### <a name="response-headers"></a>Intestazioni di risposta

Dopo l'invio delle intestazioni per una risposta:

* L'app non può modificare il codice di stato della risposta.
* Non è possibile eseguire pagine o gestori delle eccezioni. La risposta deve essere completata o la connessione deve essere interrotta.

## <a name="server-exception-handling"></a>Gestione delle eccezioni del server

Oltre alla logica di gestione delle eccezioni nell'app, l'[implementazione del server HTTP](xref:fundamentals/servers/index) può gestire alcune eccezioni. Se rileva un'eccezione prima dell'invio delle intestazioni di risposta, il server invia una risposta *500 - Errore interno del server* senza corpo. Se il server rileva un'eccezione dopo l'invio delle intestazioni di risposta, il server chiude la connessione. Le richieste che non sono gestite dall'app vengono gestite dal server. Qualsiasi eccezione che si verifica quando il server sta gestendo la richiesta viene gestita dalla gestione delle eccezioni del server. Eventuali pagine di errore personalizzate dell'app, middleware di gestione delle eccezioni e filtri non hanno effetto su questo comportamento.

## <a name="startup-exception-handling"></a>Gestione delle eccezioni durante l'avvio

Solo il livello di hosting può gestire le eccezioni che si verificano durante l'avvio dell'app. L'host può essere configurato per [acquisire gli errori di avvio](xref:fundamentals/host/web-host#capture-startup-errors) e [acquisire errori dettagliati](xref:fundamentals/host/web-host#detailed-errors).

Il livello di hosting può visualizzare una pagina di errore per un errore di avvio acquisito solo se l'errore si verifica dopo l'associazione indirizzo host/porta. Se si verifica un errore di associazione:

* Il livello di hosting registra un'eccezione critica.
* Si verifica un arresto anomalo del processo di dotnet.
* Non viene visualizzata alcuna pagina di errore quando il server HTTP è [Kestrel](xref:fundamentals/servers/kestrel).

Se durante l'esecuzione in [IIS](/iis) (o servizio app di Azure) o in [IIS Express](/iis/extensions/introduction-to-iis-express/iis-express-overview) il processo non può essere avviato, viene restituito l'errore *502.5 Errore del processo* dal [modulo ASP.NET Core](xref:host-and-deploy/aspnet-core-module). Per altre informazioni, vedere <xref:test/troubleshoot-azure-iis>.

## <a name="database-error-page"></a>Pagina di errore di database

Il middleware della pagina di errore del database acquisisce le eccezioni correlate al database che possono essere risolte tramite Entity Framework migrazioni. Quando si verificano queste eccezioni, viene generata una risposta HTML con i dettagli delle azioni possibili per risolvere il problema. Questa pagina deve essere abilitata solo nell'ambiente di sviluppo. Abilitare la pagina aggiungendo codice `Startup.Configure`:

```csharp
if (env.IsDevelopment())
{
    app.UseDatabaseErrorPage();
}
```

<xref:Microsoft.AspNetCore.Builder.DatabaseErrorPageExtensions.UseDatabaseErrorPage%2A> richiede il pacchetto NuGet [Microsoft. AspNetCore. Diagnostics. EntityFrameworkCore](https://www.nuget.org/packages/Microsoft.AspNetCore.Diagnostics.EntityFrameworkCore/) .

<!-- FUTURE UPDATE: On the next topic overhaul/release update, add API crosslink to this section for xref:Microsoft.AspNetCore.Builder.DatabaseErrorPageExtensions.UseDatabaseErrorPage* when available via the API docs. -->

## <a name="exception-filters"></a>Filtri eccezioni

Nelle app MVC i filtri delle eccezioni possono essere configurati a livello globale o per ogni controller o azione singolarmente. Nelle Razor app Pages possono essere configurate a livello globale o per modello di pagina. Questi filtri gestiscono tutte le eccezioni non gestite che si verificano durante l'esecuzione di un'azione del controller o di un altro filtro Per altre informazioni, vedere <xref:mvc/controllers/filters#exception-filters>.

> [!TIP]
> I filtri delle eccezioni sono utili per intercettare le eccezioni che si verificano all'interno di azioni MVC, ma non sono flessibili come il middleware di gestione degli errori. È consigliabile usare il middleware. Usare i filtri solo quando è necessario eseguire la gestione degli errori in modo diverso in base all'azione MVC scelta.

## <a name="model-state-errors"></a>Errori di stato del modello

Per informazioni su come gestire gli errori dello stato del modello, vedere [Associazione di modelli](xref:mvc/models/model-binding) e [Convalida del modello](xref:mvc/models/validation).

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:test/troubleshoot-azure-iis>
* <xref:host-and-deploy/azure-iis-errors-reference>

::: moniker-end
