---
title: Registrazione a prestazioni elevate con LoggerMessage in ASP.NET Core
author: rick-anderson
description: Informazioni su come usare LoggerMessage per creare delegati inseribili nella cache che richiedono un numero minore di allocazioni di oggetti per scenari di registrazione a prestazioni elevate.
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 08/26/2019
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
uid: fundamentals/logging/loggermessage
ms.openlocfilehash: d6fae8d2e056c0a578b64322bb82027523904254
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102587398"
---
# <a name="high-performance-logging-with-loggermessage-in-aspnet-core"></a>Registrazione a prestazioni elevate con LoggerMessage in ASP.NET Core

::: moniker range=">= aspnetcore-3.0"

Le funzionalità di <xref:Microsoft.Extensions.Logging.LoggerMessage> creano delegati inseribili nella cache che richiedono un numero minore di allocazioni di oggetti e riducono il sovraccarico di calcolo rispetto ai [metodi di estensione del logger](xref:Microsoft.Extensions.Logging.LoggerExtensions), ad esempio <xref:Microsoft.Extensions.Logging.LoggerExtensions.LogInformation*> e <xref:Microsoft.Extensions.Logging.LoggerExtensions.LogDebug*>. Per gli scenari di registrazione a prestazioni elevate, usare <xref:Microsoft.Extensions.Logging.LoggerMessage>.

<xref:Microsoft.Extensions.Logging.LoggerMessage> offre i seguenti vantaggi in termini di prestazioni rispetto ai metodi di estensione del logger:

* I metodi di estensione del logger richiedono una "conversione boxing" dei tipi di valori, ad esempio `int`, in `object`. <xref:Microsoft.Extensions.Logging.LoggerMessage> evita la conversione boxing usando campi <xref:System.Action> statici e metodi di estensione con parametri fortemente tipizzati.
* I metodi di estensione del logger devono analizzare il modello di messaggio (stringa di formato denominata) ogni volta che viene scritto un messaggio del log. Solo <xref:Microsoft.Extensions.Logging.LoggerMessage> richiede una sola analisi del modello durante la definizione del messaggio.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/logging/loggermessage/samples/) ([procedura per il download](xref:index#how-to-download-a-sample))

L'app di esempio illustra le funzionalità di <xref:Microsoft.Extensions.Logging.LoggerMessage> con un sistema di verifica delle offerte di base. L'app aggiunge ed elimina le offerte usando un database in memoria. Durante l'esecuzione di queste operazioni, i messaggi del log vengono generati usando <xref:Microsoft.Extensions.Logging.LoggerMessage>.

## <a name="loggermessagedefine"></a>LoggerMessage.Define

[Define(LogLevel, EventId, String)](xref:Microsoft.Extensions.Logging.LoggerMessage.Define*) crea un delegato <xref:System.Action> per la registrazione di un messaggio. Gli overload <xref:Microsoft.Extensions.Logging.LoggerMessage.Define*> permettono il passaggio di un massimo di sei parametri di tipo in una stringa di formato denominata (modello).

La stringa specificata nel metodo <xref:Microsoft.Extensions.Logging.LoggerMessage.Define*> è un modello e non una stringa interpolata. I segnaposto vengono inseriti nell'ordine in cui sono specificati i tipi. I nomi dei segnaposto nel modello devono essere descrittivi e coerenti tra i modelli. Vengono usati come nomi di proprietà all'interno dei dati di log strutturati. Per i nomi dei segnaposto è consigliabile usare la [convenzione Pascal](/dotnet/standard/design-guidelines/capitalization-conventions). Ad esempio, `{Count}`, `{FirstName}`.

Ogni messaggio di log è un elemento <xref:System.Action> contenuto in un campo statico creato da [LoggerMessage.Define](xref:Microsoft.Extensions.Logging.LoggerMessage.Define*). Ad esempio, l'app di esempio crea un campo per descrivere un messaggio di log per una richiesta GET per la pagina di indice (*Internal/LoggerExtensions.cs*):

[!code-csharp[](loggermessage/samples/3.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet1)]

Per <xref:System.Action>, specificare:

* Il livello del log.
* Un identificatore di evento univoco (<xref:Microsoft.Extensions.Logging.EventId>) con il nome del metodo di estensione statico.
* Il modello di messaggio (stringa di formato denominata). 

Una richiesta per la pagina di indice dell'app di esempio imposta:

* Il livello del log su `Information`.
* L'ID evento su `1` con il nome del metodo `IndexPageRequested`.
* Il modello di messaggio (stringa di formato denominata) su una stringa.

[!code-csharp[](loggermessage/samples/3.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet5)]

Negli archivi di log strutturati è possibile che venga usato il nome evento quando viene specificato con l'ID evento per arricchire la registrazione. Ad esempio, [Serilog](https://github.com/serilog/serilog-extensions-logging) usa il nome evento.

<xref:System.Action> viene richiamato attraverso un metodo di estensione fortemente tipizzato. Il metodo `IndexPageRequested` registra un messaggio per la richiesta GET di una pagina di indice nell'app di esempio:

[!code-csharp[](loggermessage/samples/3.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet9)]

`IndexPageRequested` viene chiamato nel logger nel metodo `OnGetAsync` in *Pages/Index.cshtml.cs*:

[!code-csharp[](loggermessage/samples/3.x/LoggerMessageSample/Pages/Index.cshtml.cs?name=snippet2&highlight=3)]

Esaminare l'output della console dell'app:

```console
info: LoggerMessageSample.Pages.IndexModel[1]
      => RequestId:0HL90M6E7PHK4:00000001 RequestPath:/ => /Index
      GET request for Index page
```

Per passare i parametri in un messaggio di log, definire un massimo di sei tipi durante la creazione del campo statico. L'app di esempio registra una stringa quando viene aggiunta un'offerta definendo un tipo `string` per il campo <xref:System.Action>:

[!code-csharp[](loggermessage/samples/3.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet2)]

Il modello di messaggio di log del delegato riceve i valori dei segnaposto dai tipi specificati. L'app di esempio definisce un delegato per l'aggiunta di un'offerta dove il parametro dell'offerta è un elemento `string`:

[!code-csharp[](loggermessage/samples/3.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet6)]

Il metodo di estensione statico per l'aggiunta di un'offerta, `QuoteAdded`, riceve il valore dell'argomento dell'offerta e lo passa al delegato <xref:System.Action>:

[!code-csharp[](loggermessage/samples/3.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet10)]

Nel modello di pagina della pagina di indice (*pages/index. cshtml. cs*) `QuoteAdded` viene chiamato il metodo per registrare il messaggio:

[!code-csharp[](loggermessage/samples/3.x/LoggerMessageSample/Pages/Index.cshtml.cs?name=snippet3&highlight=6)]

Esaminare l'output della console dell'app:

```console
info: LoggerMessageSample.Pages.IndexModel[2]
      => RequestId:0HL90M6E7PHK5:0000000A RequestPath:/ => /Index
      Quote added (Quote = 'You can avoid reality, but you cannot avoid the 
          consequences of avoiding reality. - Ayn Rand')
```

L'app di esempio implementa un modello [try-catch](/dotnet/csharp/language-reference/keywords/try-catch) per l'eliminazione delle virgolette. Per un'operazione di eliminazione con esito positivo, viene registrato un messaggio informativo. Per un'operazione di eliminazione con la generazione di un'eccezione, viene registrato un messaggio di errore. Il messaggio di log per un'operazione di eliminazione con esito negativo include l'analisi dello stack dell'eccezione (*Internal/LoggerExtensions.cs*):

[!code-csharp[](loggermessage/samples/3.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet3)]

[!code-csharp[](loggermessage/samples/3.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet7)]

Si noti come l'eccezione viene passata al delegato in `QuoteDeleteFailed`:

[!code-csharp[](loggermessage/samples/3.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet11)]

Nel modello di pagina per la pagina di indice l'eliminazione di un'offerta con esito positivo chiama il metodo `QuoteDeleted` nel logger. Quando non viene trovata un'offerta per l'eliminazione, viene generata un'eccezione <xref:System.ArgumentNullException>. L'eccezione viene intercettata dall'istruzione [try-catch](/dotnet/csharp/language-reference/keywords/try-catch) e registrata chiamando il `QuoteDeleteFailed` metodo sul logger nel blocco [catch](/dotnet/csharp/language-reference/keywords/try-catch) (*pages/index. cshtml. cs*):

[!code-csharp[](loggermessage/samples/3.x/LoggerMessageSample/Pages/Index.cshtml.cs?name=snippet5&highlight=9,13)]

Quando un'offerta viene eliminata, esaminare l'output della console dell'app:

```console
info: LoggerMessageSample.Pages.IndexModel[4]
      => RequestId:0HL90M6E7PHK5:00000016 RequestPath:/ => /Index
      Quote deleted (Quote = 'You can avoid reality, but you cannot avoid the 
          consequences of avoiding reality. - Ayn Rand' Id = 1)
```

Quando l'eliminazione di un'offerta ha esito negativo, esaminare l'output della console dell'app. Si noti che l'eccezione è inclusa nel messaggio di log:

```console
LoggerMessageSample.Pages.IndexModel: Error: Quote delete failed (Id = 999)

System.NullReferenceException: Object reference not set to an instance of an object.
   at lambda_method(Closure , ValueBuffer )
   at System.Linq.Enumerable.SelectEnumerableIterator`2.MoveNext()
   at Microsoft.EntityFrameworkCore.InMemory.Query.Internal.InMemoryShapedQueryCompilingExpressionVisitor.AsyncQueryingEnumerable`1.AsyncEnumerator.MoveNextAsync()
   at Microsoft.EntityFrameworkCore.Query.ShapedQueryCompilingExpressionVisitor.SingleOrDefaultAsync[TSource](IAsyncEnumerable`1 asyncEnumerable, CancellationToken cancellationToken)
   at Microsoft.EntityFrameworkCore.Query.ShapedQueryCompilingExpressionVisitor.SingleOrDefaultAsync[TSource](IAsyncEnumerable`1 asyncEnumerable, CancellationToken cancellationToken)
   at LoggerMessageSample.Pages.IndexModel.OnPostDeleteQuoteAsync(Int32 id) in c:\Users\guard\Documents\GitHub\Docs\aspnetcore\fundamentals\logging\loggermessage\samples\3.x\LoggerMessageSample\Pages\Index.cshtml.cs:line 77
```

## <a name="loggermessagedefinescope"></a>LoggerMessage.DefineScope

[DefineScope(String)](xref:Microsoft.Extensions.Logging.LoggerMessage.DefineScope*) crea un delegato <xref:System.Func%601> per definire un [ambito del log](xref:fundamentals/logging/index#log-scopes). Gli overload <xref:Microsoft.Extensions.Logging.LoggerMessage.DefineScope*> permettono il passaggio di un massimo di tre parametri di tipo in una stringa di formato denominata (modello).

Come avviene per il metodo <xref:Microsoft.Extensions.Logging.LoggerMessage.Define*>, la stringa specificata nel metodo <xref:Microsoft.Extensions.Logging.LoggerMessage.DefineScope*> è un modello e non una stringa interpolata. I segnaposto vengono inseriti nell'ordine in cui sono specificati i tipi. I nomi dei segnaposto nel modello devono essere descrittivi e coerenti tra i modelli. Vengono usati come nomi di proprietà all'interno dei dati di log strutturati. Per i nomi dei segnaposto è consigliabile usare la [convenzione Pascal](/dotnet/standard/design-guidelines/capitalization-conventions). Ad esempio, `{Count}`, `{FirstName}`.

Definire un [ambito del log](xref:fundamentals/logging/index#log-scopes) da applicare a una serie di messaggi di log usando il metodo <xref:Microsoft.Extensions.Logging.LoggerMessage.DefineScope*>.

L'app di esempio include un pulsante **Clear All** (Cancella tutto) per eliminare tutte le offerte nel database. Le offerte vengono eliminate rimuovendole una alla volta. Ogni volta che viene eliminata un'offerta, viene chiamato il metodo `QuoteDeleted` nel logger. Viene aggiunto un ambito di log a questi messaggi di log.

Abilitare `IncludeScopes` nella sezione Logger console di *appsettings.json* :

[!code-json[](loggermessage/samples/3.x/LoggerMessageSample/appsettings.json?highlight=3-5)]

Per creare un ambito di log, aggiungere un campo per inserire un delegato <xref:System.Func%601> per l'ambito. L'app di esempio crea un campo denominato `_allQuotesDeletedScope` (*Internal/LoggerExtensions.cs*):

[!code-csharp[](loggermessage/samples/3.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet4)]

Usare <xref:Microsoft.Extensions.Logging.LoggerMessage.DefineScope*> per creare il delegato. È possibile specificare un massimo di tre tipi da usare come argomenti di modello quando viene richiamato il delegato. L'app di esempio usa un modello di messaggio che include il numero di offerte eliminate (un tipo `int`):

[!code-csharp[](loggermessage/samples/3.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet8)]

Specificare un metodo di estensione statico per il messaggio di log. Includere i parametri di tipo per le proprietà denominate visualizzate nel modello di messaggio. L'app di esempio accetta un numero `count` di offerte da eliminare e restituisce `_allQuotesDeletedScope`:

[!code-csharp[](loggermessage/samples/3.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet12)]

L'ambito esegue il wrapping delle chiamate di estensione di registrazione in un blocco [using](/dotnet/csharp/language-reference/keywords/using-statement):

[!code-csharp[](loggermessage/samples/3.x/LoggerMessageSample/Pages/Index.cshtml.cs?name=snippet4&highlight=5-6,14)]

Esaminare i messaggi di log nell'output della console dell'app. Il risultato seguente mostra tre offerte eliminate con il messaggio di ambito di log incluso:

```console
info: LoggerMessageSample.Pages.IndexModel[4]
      => RequestId:0HL90M6E7PHK5:0000002E RequestPath:/ => /Index => 
          All quotes deleted (Count = 3)
      Quote deleted (Quote = 'Quote 1' Id = 2)
info: LoggerMessageSample.Pages.IndexModel[4]
      => RequestId:0HL90M6E7PHK5:0000002E RequestPath:/ => /Index => 
          All quotes deleted (Count = 3)
      Quote deleted (Quote = 'Quote 2' Id = 3)
info: LoggerMessageSample.Pages.IndexModel[4]
      => RequestId:0HL90M6E7PHK5:0000002E RequestPath:/ => /Index => 
          All quotes deleted (Count = 3)
      Quote deleted (Quote = 'Quote 3' Id = 4)
```

::: moniker-end

::: moniker range="< aspnetcore-3.0"

Le funzionalità di <xref:Microsoft.Extensions.Logging.LoggerMessage> creano delegati inseribili nella cache che richiedono un numero minore di allocazioni di oggetti e riducono il sovraccarico di calcolo rispetto ai [metodi di estensione del logger](xref:Microsoft.Extensions.Logging.LoggerExtensions), ad esempio <xref:Microsoft.Extensions.Logging.LoggerExtensions.LogInformation*> e <xref:Microsoft.Extensions.Logging.LoggerExtensions.LogDebug*>. Per gli scenari di registrazione a prestazioni elevate, usare <xref:Microsoft.Extensions.Logging.LoggerMessage>.

<xref:Microsoft.Extensions.Logging.LoggerMessage> offre i seguenti vantaggi in termini di prestazioni rispetto ai metodi di estensione del logger:

* I metodi di estensione del logger richiedono una "conversione boxing" dei tipi di valori, ad esempio `int`, in `object`. <xref:Microsoft.Extensions.Logging.LoggerMessage> evita la conversione boxing usando campi <xref:System.Action> statici e metodi di estensione con parametri fortemente tipizzati.
* I metodi di estensione del logger devono analizzare il modello di messaggio (stringa di formato denominata) ogni volta che viene scritto un messaggio del log. Solo <xref:Microsoft.Extensions.Logging.LoggerMessage> richiede una sola analisi del modello durante la definizione del messaggio.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/logging/loggermessage/samples/) ([procedura per il download](xref:index#how-to-download-a-sample))

L'app di esempio illustra le funzionalità di <xref:Microsoft.Extensions.Logging.LoggerMessage> con un sistema di verifica delle offerte di base. L'app aggiunge ed elimina le offerte usando un database in memoria. Durante l'esecuzione di queste operazioni, i messaggi del log vengono generati usando <xref:Microsoft.Extensions.Logging.LoggerMessage>.

## <a name="loggermessagedefine"></a>LoggerMessage.Define

[Define(LogLevel, EventId, String)](xref:Microsoft.Extensions.Logging.LoggerMessage.Define*) crea un delegato <xref:System.Action> per la registrazione di un messaggio. Gli overload <xref:Microsoft.Extensions.Logging.LoggerMessage.Define*> permettono il passaggio di un massimo di sei parametri di tipo in una stringa di formato denominata (modello).

La stringa specificata nel metodo <xref:Microsoft.Extensions.Logging.LoggerMessage.Define*> è un modello e non una stringa interpolata. I segnaposto vengono inseriti nell'ordine in cui sono specificati i tipi. I nomi dei segnaposto nel modello devono essere descrittivi e coerenti tra i modelli. Vengono usati come nomi di proprietà all'interno dei dati di log strutturati. Per i nomi dei segnaposto è consigliabile usare la [convenzione Pascal](/dotnet/standard/design-guidelines/capitalization-conventions). Ad esempio, `{Count}`, `{FirstName}`.

Ogni messaggio di log è un elemento <xref:System.Action> contenuto in un campo statico creato da [LoggerMessage.Define](xref:Microsoft.Extensions.Logging.LoggerMessage.Define*). Ad esempio, l'app di esempio crea un campo per descrivere un messaggio di log per una richiesta GET per la pagina di indice (*Internal/LoggerExtensions.cs*):

[!code-csharp[](loggermessage/samples/2.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet1)]

Per <xref:System.Action>, specificare:

* Il livello del log.
* Un identificatore di evento univoco (<xref:Microsoft.Extensions.Logging.EventId>) con il nome del metodo di estensione statico.
* Il modello di messaggio (stringa di formato denominata). 

Una richiesta per la pagina di indice dell'app di esempio imposta:

* Il livello del log su `Information`.
* L'ID evento su `1` con il nome del metodo `IndexPageRequested`.
* Il modello di messaggio (stringa di formato denominata) su una stringa.

[!code-csharp[](loggermessage/samples/2.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet5)]

Negli archivi di log strutturati è possibile che venga usato il nome evento quando viene specificato con l'ID evento per arricchire la registrazione. Ad esempio, [Serilog](https://github.com/serilog/serilog-extensions-logging) usa il nome evento.

<xref:System.Action> viene richiamato attraverso un metodo di estensione fortemente tipizzato. Il metodo `IndexPageRequested` registra un messaggio per la richiesta GET di una pagina di indice nell'app di esempio:

[!code-csharp[](loggermessage/samples/2.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet9)]

`IndexPageRequested` viene chiamato nel logger nel metodo `OnGetAsync` in *Pages/Index.cshtml.cs*:

[!code-csharp[](loggermessage/samples/2.x/LoggerMessageSample/Pages/Index.cshtml.cs?name=snippet2&highlight=3)]

Esaminare l'output della console dell'app:

```console
info: LoggerMessageSample.Pages.IndexModel[1]
      => RequestId:0HL90M6E7PHK4:00000001 RequestPath:/ => /Index
      GET request for Index page
```

Per passare i parametri in un messaggio di log, definire un massimo di sei tipi durante la creazione del campo statico. L'app di esempio registra una stringa quando viene aggiunta un'offerta definendo un tipo `string` per il campo <xref:System.Action>:

[!code-csharp[](loggermessage/samples/2.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet2)]

Il modello di messaggio di log del delegato riceve i valori dei segnaposto dai tipi specificati. L'app di esempio definisce un delegato per l'aggiunta di un'offerta dove il parametro dell'offerta è un elemento `string`:

[!code-csharp[](loggermessage/samples/2.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet6)]

Il metodo di estensione statico per l'aggiunta di un'offerta, `QuoteAdded`, riceve il valore dell'argomento dell'offerta e lo passa al delegato <xref:System.Action>:

[!code-csharp[](loggermessage/samples/2.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet10)]

Nel modello di pagina della pagina di indice (*pages/index. cshtml. cs*) `QuoteAdded` viene chiamato il metodo per registrare il messaggio:

[!code-csharp[](loggermessage/samples/2.x/LoggerMessageSample/Pages/Index.cshtml.cs?name=snippet3&highlight=6)]

Esaminare l'output della console dell'app:

```console
info: LoggerMessageSample.Pages.IndexModel[2]
      => RequestId:0HL90M6E7PHK5:0000000A RequestPath:/ => /Index
      Quote added (Quote = 'You can avoid reality, but you cannot avoid the 
          consequences of avoiding reality. - Ayn Rand')
```

L'app di esempio implementa un modello [try-catch](/dotnet/csharp/language-reference/keywords/try-catch) per l'eliminazione delle virgolette. Per un'operazione di eliminazione con esito positivo, viene registrato un messaggio informativo. Per un'operazione di eliminazione con la generazione di un'eccezione, viene registrato un messaggio di errore. Il messaggio di log per un'operazione di eliminazione con esito negativo include l'analisi dello stack dell'eccezione (*Internal/LoggerExtensions.cs*):

[!code-csharp[](loggermessage/samples/2.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet3)]

[!code-csharp[](loggermessage/samples/2.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet7)]

Si noti come l'eccezione viene passata al delegato in `QuoteDeleteFailed`:

[!code-csharp[](loggermessage/samples/2.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet11)]

Nel modello di pagina per la pagina di indice l'eliminazione di un'offerta con esito positivo chiama il metodo `QuoteDeleted` nel logger. Quando non viene trovata un'offerta per l'eliminazione, viene generata un'eccezione <xref:System.ArgumentNullException>. L'eccezione viene intercettata dall'istruzione [try-catch](/dotnet/csharp/language-reference/keywords/try-catch) e registrata chiamando il `QuoteDeleteFailed` metodo sul logger nel blocco [catch](/dotnet/csharp/language-reference/keywords/try-catch) (*pages/index. cshtml. cs*):

[!code-csharp[](loggermessage/samples/2.x/LoggerMessageSample/Pages/Index.cshtml.cs?name=snippet5&highlight=14,18)]

Quando un'offerta viene eliminata, esaminare l'output della console dell'app:

```console
info: LoggerMessageSample.Pages.IndexModel[4]
      => RequestId:0HL90M6E7PHK5:00000016 RequestPath:/ => /Index
      Quote deleted (Quote = 'You can avoid reality, but you cannot avoid the 
          consequences of avoiding reality. - Ayn Rand' Id = 1)
```

Quando l'eliminazione di un'offerta ha esito negativo, esaminare l'output della console dell'app. Si noti che l'eccezione è inclusa nel messaggio di log:

```console
fail: LoggerMessageSample.Pages.IndexModel[5]
      => RequestId:0HL90M6E7PHK5:00000010 RequestPath:/ => /Index
      Quote delete failed (Id = 999)
System.ArgumentNullException: Value cannot be null.
Parameter name: entity
   at Microsoft.EntityFrameworkCore.Utilities.Check.NotNull[T]
       (T value, String parameterName)
   at Microsoft.EntityFrameworkCore.DbContext.Remove[TEntity](TEntity entity)
   at Microsoft.EntityFrameworkCore.Internal.InternalDbSet`1.Remove(TEntity entity)
   at LoggerMessageSample.Pages.IndexModel.<OnPostDeleteQuoteAsync>d__14.MoveNext() 
      in <PATH>\sample\Pages\Index.cshtml.cs:line 87
```

## <a name="loggermessagedefinescope"></a>LoggerMessage.DefineScope

[DefineScope(String)](xref:Microsoft.Extensions.Logging.LoggerMessage.DefineScope*) crea un delegato <xref:System.Func%601> per definire un [ambito del log](xref:fundamentals/logging/index#log-scopes). Gli overload <xref:Microsoft.Extensions.Logging.LoggerMessage.DefineScope*> permettono il passaggio di un massimo di tre parametri di tipo in una stringa di formato denominata (modello).

Come avviene per il metodo <xref:Microsoft.Extensions.Logging.LoggerMessage.Define*>, la stringa specificata nel metodo <xref:Microsoft.Extensions.Logging.LoggerMessage.DefineScope*> è un modello e non una stringa interpolata. I segnaposto vengono inseriti nell'ordine in cui sono specificati i tipi. I nomi dei segnaposto nel modello devono essere descrittivi e coerenti tra i modelli. Vengono usati come nomi di proprietà all'interno dei dati di log strutturati. Per i nomi dei segnaposto è consigliabile usare la [convenzione Pascal](/dotnet/standard/design-guidelines/capitalization-conventions). Ad esempio, `{Count}`, `{FirstName}`.

Definire un [ambito del log](xref:fundamentals/logging/index#log-scopes) da applicare a una serie di messaggi di log usando il metodo <xref:Microsoft.Extensions.Logging.LoggerMessage.DefineScope*>.

L'app di esempio include un pulsante **Clear All** (Cancella tutto) per eliminare tutte le offerte nel database. Le offerte vengono eliminate rimuovendole una alla volta. Ogni volta che viene eliminata un'offerta, viene chiamato il metodo `QuoteDeleted` nel logger. Viene aggiunto un ambito di log a questi messaggi di log.

Abilitare `IncludeScopes` nella sezione Logger console di *appsettings.json* :

[!code-json[](loggermessage/samples/2.x/LoggerMessageSample/appsettings.json?highlight=3-5)]

Per creare un ambito di log, aggiungere un campo per inserire un delegato <xref:System.Func%601> per l'ambito. L'app di esempio crea un campo denominato `_allQuotesDeletedScope` (*Internal/LoggerExtensions.cs*):

[!code-csharp[](loggermessage/samples/2.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet4)]

Usare <xref:Microsoft.Extensions.Logging.LoggerMessage.DefineScope*> per creare il delegato. È possibile specificare un massimo di tre tipi da usare come argomenti di modello quando viene richiamato il delegato. L'app di esempio usa un modello di messaggio che include il numero di offerte eliminate (un tipo `int`):

[!code-csharp[](loggermessage/samples/2.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet8)]

Specificare un metodo di estensione statico per il messaggio di log. Includere i parametri di tipo per le proprietà denominate visualizzate nel modello di messaggio. L'app di esempio accetta un numero `count` di offerte da eliminare e restituisce `_allQuotesDeletedScope`:

[!code-csharp[](loggermessage/samples/2.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet12)]

L'ambito esegue il wrapping delle chiamate di estensione di registrazione in un blocco [using](/dotnet/csharp/language-reference/keywords/using-statement):

[!code-csharp[](loggermessage/samples/2.x/LoggerMessageSample/Pages/Index.cshtml.cs?name=snippet4&highlight=5-6,14)]

Esaminare i messaggi di log nell'output della console dell'app. Il risultato seguente mostra tre offerte eliminate con il messaggio di ambito di log incluso:

```console
info: LoggerMessageSample.Pages.IndexModel[4]
      => RequestId:0HL90M6E7PHK5:0000002E RequestPath:/ => /Index => 
          All quotes deleted (Count = 3)
      Quote deleted (Quote = 'Quote 1' Id = 2)
info: LoggerMessageSample.Pages.IndexModel[4]
      => RequestId:0HL90M6E7PHK5:0000002E RequestPath:/ => /Index => 
          All quotes deleted (Count = 3)
      Quote deleted (Quote = 'Quote 2' Id = 3)
info: LoggerMessageSample.Pages.IndexModel[4]
      => RequestId:0HL90M6E7PHK5:0000002E RequestPath:/ => /Index => 
          All quotes deleted (Count = 3)
      Quote deleted (Quote = 'Quote 3' Id = 4)
```

::: moniker-end

## <a name="additional-resources"></a>Risorse aggiuntive

* [Logging](xref:fundamentals/logging/index)
