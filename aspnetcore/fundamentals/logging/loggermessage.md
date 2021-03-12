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
# <a name="high-performance-logging-with-loggermessage-in-aspnet-core"></a><span data-ttu-id="fe923-103">Registrazione a prestazioni elevate con LoggerMessage in ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="fe923-103">High-performance logging with LoggerMessage in ASP.NET Core</span></span>

::: moniker range=">= aspnetcore-3.0"

<span data-ttu-id="fe923-104">Le funzionalità di <xref:Microsoft.Extensions.Logging.LoggerMessage> creano delegati inseribili nella cache che richiedono un numero minore di allocazioni di oggetti e riducono il sovraccarico di calcolo rispetto ai [metodi di estensione del logger](xref:Microsoft.Extensions.Logging.LoggerExtensions), ad esempio <xref:Microsoft.Extensions.Logging.LoggerExtensions.LogInformation*> e <xref:Microsoft.Extensions.Logging.LoggerExtensions.LogDebug*>.</span><span class="sxs-lookup"><span data-stu-id="fe923-104"><xref:Microsoft.Extensions.Logging.LoggerMessage> features create cacheable delegates that require fewer object allocations and reduced computational overhead compared to [logger extension methods](xref:Microsoft.Extensions.Logging.LoggerExtensions), such as <xref:Microsoft.Extensions.Logging.LoggerExtensions.LogInformation*> and <xref:Microsoft.Extensions.Logging.LoggerExtensions.LogDebug*>.</span></span> <span data-ttu-id="fe923-105">Per gli scenari di registrazione a prestazioni elevate, usare <xref:Microsoft.Extensions.Logging.LoggerMessage>.</span><span class="sxs-lookup"><span data-stu-id="fe923-105">For high-performance logging scenarios, use the <xref:Microsoft.Extensions.Logging.LoggerMessage> pattern.</span></span>

<span data-ttu-id="fe923-106"><xref:Microsoft.Extensions.Logging.LoggerMessage> offre i seguenti vantaggi in termini di prestazioni rispetto ai metodi di estensione del logger:</span><span class="sxs-lookup"><span data-stu-id="fe923-106"><xref:Microsoft.Extensions.Logging.LoggerMessage> provides the following performance advantages over Logger extension methods:</span></span>

* <span data-ttu-id="fe923-107">I metodi di estensione del logger richiedono una "conversione boxing" dei tipi di valori, ad esempio `int`, in `object`.</span><span class="sxs-lookup"><span data-stu-id="fe923-107">Logger extension methods require "boxing" (converting) value types, such as `int`, into `object`.</span></span> <span data-ttu-id="fe923-108"><xref:Microsoft.Extensions.Logging.LoggerMessage> evita la conversione boxing usando campi <xref:System.Action> statici e metodi di estensione con parametri fortemente tipizzati.</span><span class="sxs-lookup"><span data-stu-id="fe923-108">The <xref:Microsoft.Extensions.Logging.LoggerMessage> pattern avoids boxing by using static <xref:System.Action> fields and extension methods with strongly-typed parameters.</span></span>
* <span data-ttu-id="fe923-109">I metodi di estensione del logger devono analizzare il modello di messaggio (stringa di formato denominata) ogni volta che viene scritto un messaggio del log.</span><span class="sxs-lookup"><span data-stu-id="fe923-109">Logger extension methods must parse the message template (named format string) every time a log message is written.</span></span> <span data-ttu-id="fe923-110">Solo <xref:Microsoft.Extensions.Logging.LoggerMessage> richiede una sola analisi del modello durante la definizione del messaggio.</span><span class="sxs-lookup"><span data-stu-id="fe923-110"><xref:Microsoft.Extensions.Logging.LoggerMessage> only requires parsing a template once when the message is defined.</span></span>

<span data-ttu-id="fe923-111">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/logging/loggermessage/samples/) ([procedura per il download](xref:index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="fe923-111">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/logging/loggermessage/samples/) ([how to download](xref:index#how-to-download-a-sample))</span></span>

<span data-ttu-id="fe923-112">L'app di esempio illustra le funzionalità di <xref:Microsoft.Extensions.Logging.LoggerMessage> con un sistema di verifica delle offerte di base.</span><span class="sxs-lookup"><span data-stu-id="fe923-112">The sample app demonstrates <xref:Microsoft.Extensions.Logging.LoggerMessage> features with a basic quote tracking system.</span></span> <span data-ttu-id="fe923-113">L'app aggiunge ed elimina le offerte usando un database in memoria.</span><span class="sxs-lookup"><span data-stu-id="fe923-113">The app adds and deletes quotes using an in-memory database.</span></span> <span data-ttu-id="fe923-114">Durante l'esecuzione di queste operazioni, i messaggi del log vengono generati usando <xref:Microsoft.Extensions.Logging.LoggerMessage>.</span><span class="sxs-lookup"><span data-stu-id="fe923-114">As these operations occur, log messages are generated using the <xref:Microsoft.Extensions.Logging.LoggerMessage> pattern.</span></span>

## <a name="loggermessagedefine"></a><span data-ttu-id="fe923-115">LoggerMessage.Define</span><span class="sxs-lookup"><span data-stu-id="fe923-115">LoggerMessage.Define</span></span>

<span data-ttu-id="fe923-116">[Define(LogLevel, EventId, String)](xref:Microsoft.Extensions.Logging.LoggerMessage.Define*) crea un delegato <xref:System.Action> per la registrazione di un messaggio.</span><span class="sxs-lookup"><span data-stu-id="fe923-116">[Define(LogLevel, EventId, String)](xref:Microsoft.Extensions.Logging.LoggerMessage.Define*) creates an <xref:System.Action> delegate for logging a message.</span></span> <span data-ttu-id="fe923-117">Gli overload <xref:Microsoft.Extensions.Logging.LoggerMessage.Define*> permettono il passaggio di un massimo di sei parametri di tipo in una stringa di formato denominata (modello).</span><span class="sxs-lookup"><span data-stu-id="fe923-117"><xref:Microsoft.Extensions.Logging.LoggerMessage.Define*> overloads permit passing up to six type parameters to a named format string (template).</span></span>

<span data-ttu-id="fe923-118">La stringa specificata nel metodo <xref:Microsoft.Extensions.Logging.LoggerMessage.Define*> è un modello e non una stringa interpolata.</span><span class="sxs-lookup"><span data-stu-id="fe923-118">The string provided to the <xref:Microsoft.Extensions.Logging.LoggerMessage.Define*> method is a template and not an interpolated string.</span></span> <span data-ttu-id="fe923-119">I segnaposto vengono inseriti nell'ordine in cui sono specificati i tipi.</span><span class="sxs-lookup"><span data-stu-id="fe923-119">Placeholders are filled in the order that the types are specified.</span></span> <span data-ttu-id="fe923-120">I nomi dei segnaposto nel modello devono essere descrittivi e coerenti tra i modelli.</span><span class="sxs-lookup"><span data-stu-id="fe923-120">Placeholder names in the template should be descriptive and consistent across templates.</span></span> <span data-ttu-id="fe923-121">Vengono usati come nomi di proprietà all'interno dei dati di log strutturati.</span><span class="sxs-lookup"><span data-stu-id="fe923-121">They serve as property names within structured log data.</span></span> <span data-ttu-id="fe923-122">Per i nomi dei segnaposto è consigliabile usare la [convenzione Pascal](/dotnet/standard/design-guidelines/capitalization-conventions).</span><span class="sxs-lookup"><span data-stu-id="fe923-122">We recommend [Pascal casing](/dotnet/standard/design-guidelines/capitalization-conventions) for placeholder names.</span></span> <span data-ttu-id="fe923-123">Ad esempio, `{Count}`, `{FirstName}`.</span><span class="sxs-lookup"><span data-stu-id="fe923-123">For example, `{Count}`, `{FirstName}`.</span></span>

<span data-ttu-id="fe923-124">Ogni messaggio di log è un elemento <xref:System.Action> contenuto in un campo statico creato da [LoggerMessage.Define](xref:Microsoft.Extensions.Logging.LoggerMessage.Define*).</span><span class="sxs-lookup"><span data-stu-id="fe923-124">Each log message is an <xref:System.Action> held in a static field created by [LoggerMessage.Define](xref:Microsoft.Extensions.Logging.LoggerMessage.Define*).</span></span> <span data-ttu-id="fe923-125">Ad esempio, l'app di esempio crea un campo per descrivere un messaggio di log per una richiesta GET per la pagina di indice (*Internal/LoggerExtensions.cs*):</span><span class="sxs-lookup"><span data-stu-id="fe923-125">For example, the sample app creates a field to describe a log message for a GET request for the Index page (*Internal/LoggerExtensions.cs*):</span></span>

[!code-csharp[](loggermessage/samples/3.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet1)]

<span data-ttu-id="fe923-126">Per <xref:System.Action>, specificare:</span><span class="sxs-lookup"><span data-stu-id="fe923-126">For the <xref:System.Action>, specify:</span></span>

* <span data-ttu-id="fe923-127">Il livello del log.</span><span class="sxs-lookup"><span data-stu-id="fe923-127">The log level.</span></span>
* <span data-ttu-id="fe923-128">Un identificatore di evento univoco (<xref:Microsoft.Extensions.Logging.EventId>) con il nome del metodo di estensione statico.</span><span class="sxs-lookup"><span data-stu-id="fe923-128">A unique event identifier (<xref:Microsoft.Extensions.Logging.EventId>) with the name of the static extension method.</span></span>
* <span data-ttu-id="fe923-129">Il modello di messaggio (stringa di formato denominata).</span><span class="sxs-lookup"><span data-stu-id="fe923-129">The message template (named format string).</span></span> 

<span data-ttu-id="fe923-130">Una richiesta per la pagina di indice dell'app di esempio imposta:</span><span class="sxs-lookup"><span data-stu-id="fe923-130">A request for the Index page of the sample app sets the:</span></span>

* <span data-ttu-id="fe923-131">Il livello del log su `Information`.</span><span class="sxs-lookup"><span data-stu-id="fe923-131">Log level to `Information`.</span></span>
* <span data-ttu-id="fe923-132">L'ID evento su `1` con il nome del metodo `IndexPageRequested`.</span><span class="sxs-lookup"><span data-stu-id="fe923-132">Event id to `1` with the name of the `IndexPageRequested` method.</span></span>
* <span data-ttu-id="fe923-133">Il modello di messaggio (stringa di formato denominata) su una stringa.</span><span class="sxs-lookup"><span data-stu-id="fe923-133">Message template (named format string) to a string.</span></span>

[!code-csharp[](loggermessage/samples/3.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet5)]

<span data-ttu-id="fe923-134">Negli archivi di log strutturati è possibile che venga usato il nome evento quando viene specificato con l'ID evento per arricchire la registrazione.</span><span class="sxs-lookup"><span data-stu-id="fe923-134">Structured logging stores may use the event name when it's supplied with the event id to enrich logging.</span></span> <span data-ttu-id="fe923-135">Ad esempio, [Serilog](https://github.com/serilog/serilog-extensions-logging) usa il nome evento.</span><span class="sxs-lookup"><span data-stu-id="fe923-135">For example, [Serilog](https://github.com/serilog/serilog-extensions-logging) uses the event name.</span></span>

<span data-ttu-id="fe923-136"><xref:System.Action> viene richiamato attraverso un metodo di estensione fortemente tipizzato.</span><span class="sxs-lookup"><span data-stu-id="fe923-136">The <xref:System.Action> is invoked through a strongly-typed extension method.</span></span> <span data-ttu-id="fe923-137">Il metodo `IndexPageRequested` registra un messaggio per la richiesta GET di una pagina di indice nell'app di esempio:</span><span class="sxs-lookup"><span data-stu-id="fe923-137">The `IndexPageRequested` method logs a message for an Index page GET request in the sample app:</span></span>

[!code-csharp[](loggermessage/samples/3.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet9)]

<span data-ttu-id="fe923-138">`IndexPageRequested` viene chiamato nel logger nel metodo `OnGetAsync` in *Pages/Index.cshtml.cs*:</span><span class="sxs-lookup"><span data-stu-id="fe923-138">`IndexPageRequested` is called on the logger in the `OnGetAsync` method in *Pages/Index.cshtml.cs*:</span></span>

[!code-csharp[](loggermessage/samples/3.x/LoggerMessageSample/Pages/Index.cshtml.cs?name=snippet2&highlight=3)]

<span data-ttu-id="fe923-139">Esaminare l'output della console dell'app:</span><span class="sxs-lookup"><span data-stu-id="fe923-139">Inspect the app's console output:</span></span>

```console
info: LoggerMessageSample.Pages.IndexModel[1]
      => RequestId:0HL90M6E7PHK4:00000001 RequestPath:/ => /Index
      GET request for Index page
```

<span data-ttu-id="fe923-140">Per passare i parametri in un messaggio di log, definire un massimo di sei tipi durante la creazione del campo statico.</span><span class="sxs-lookup"><span data-stu-id="fe923-140">To pass parameters to a log message, define up to six types when creating the static field.</span></span> <span data-ttu-id="fe923-141">L'app di esempio registra una stringa quando viene aggiunta un'offerta definendo un tipo `string` per il campo <xref:System.Action>:</span><span class="sxs-lookup"><span data-stu-id="fe923-141">The sample app logs a string when adding a quote by defining a `string` type for the <xref:System.Action> field:</span></span>

[!code-csharp[](loggermessage/samples/3.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet2)]

<span data-ttu-id="fe923-142">Il modello di messaggio di log del delegato riceve i valori dei segnaposto dai tipi specificati.</span><span class="sxs-lookup"><span data-stu-id="fe923-142">The delegate's log message template receives its placeholder values from the types provided.</span></span> <span data-ttu-id="fe923-143">L'app di esempio definisce un delegato per l'aggiunta di un'offerta dove il parametro dell'offerta è un elemento `string`:</span><span class="sxs-lookup"><span data-stu-id="fe923-143">The sample app defines a delegate for adding a quote where the quote parameter is a `string`:</span></span>

[!code-csharp[](loggermessage/samples/3.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet6)]

<span data-ttu-id="fe923-144">Il metodo di estensione statico per l'aggiunta di un'offerta, `QuoteAdded`, riceve il valore dell'argomento dell'offerta e lo passa al delegato <xref:System.Action>:</span><span class="sxs-lookup"><span data-stu-id="fe923-144">The static extension method for adding a quote, `QuoteAdded`, receives the quote argument value and passes it to the <xref:System.Action> delegate:</span></span>

[!code-csharp[](loggermessage/samples/3.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet10)]

<span data-ttu-id="fe923-145">Nel modello di pagina della pagina di indice (*pages/index. cshtml. cs*) `QuoteAdded` viene chiamato il metodo per registrare il messaggio:</span><span class="sxs-lookup"><span data-stu-id="fe923-145">In the Index page's page model (*Pages/Index.cshtml.cs*), `QuoteAdded` is called to log the message:</span></span>

[!code-csharp[](loggermessage/samples/3.x/LoggerMessageSample/Pages/Index.cshtml.cs?name=snippet3&highlight=6)]

<span data-ttu-id="fe923-146">Esaminare l'output della console dell'app:</span><span class="sxs-lookup"><span data-stu-id="fe923-146">Inspect the app's console output:</span></span>

```console
info: LoggerMessageSample.Pages.IndexModel[2]
      => RequestId:0HL90M6E7PHK5:0000000A RequestPath:/ => /Index
      Quote added (Quote = 'You can avoid reality, but you cannot avoid the 
          consequences of avoiding reality. - Ayn Rand')
```

<span data-ttu-id="fe923-147">L'app di esempio implementa un modello [try-catch](/dotnet/csharp/language-reference/keywords/try-catch) per l'eliminazione delle virgolette.</span><span class="sxs-lookup"><span data-stu-id="fe923-147">The sample app implements a [try-catch](/dotnet/csharp/language-reference/keywords/try-catch) pattern for quote deletion.</span></span> <span data-ttu-id="fe923-148">Per un'operazione di eliminazione con esito positivo, viene registrato un messaggio informativo.</span><span class="sxs-lookup"><span data-stu-id="fe923-148">An informational message is logged for a successful delete operation.</span></span> <span data-ttu-id="fe923-149">Per un'operazione di eliminazione con la generazione di un'eccezione, viene registrato un messaggio di errore.</span><span class="sxs-lookup"><span data-stu-id="fe923-149">An error message is logged for a delete operation when an exception is thrown.</span></span> <span data-ttu-id="fe923-150">Il messaggio di log per un'operazione di eliminazione con esito negativo include l'analisi dello stack dell'eccezione (*Internal/LoggerExtensions.cs*):</span><span class="sxs-lookup"><span data-stu-id="fe923-150">The log message for the unsuccessful delete operation includes the exception stack trace (*Internal/LoggerExtensions.cs*):</span></span>

[!code-csharp[](loggermessage/samples/3.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet3)]

[!code-csharp[](loggermessage/samples/3.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet7)]

<span data-ttu-id="fe923-151">Si noti come l'eccezione viene passata al delegato in `QuoteDeleteFailed`:</span><span class="sxs-lookup"><span data-stu-id="fe923-151">Note how the exception is passed to the delegate in `QuoteDeleteFailed`:</span></span>

[!code-csharp[](loggermessage/samples/3.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet11)]

<span data-ttu-id="fe923-152">Nel modello di pagina per la pagina di indice l'eliminazione di un'offerta con esito positivo chiama il metodo `QuoteDeleted` nel logger.</span><span class="sxs-lookup"><span data-stu-id="fe923-152">In the page model for the Index page, a successful quote deletion calls the `QuoteDeleted` method on the logger.</span></span> <span data-ttu-id="fe923-153">Quando non viene trovata un'offerta per l'eliminazione, viene generata un'eccezione <xref:System.ArgumentNullException>.</span><span class="sxs-lookup"><span data-stu-id="fe923-153">When a quote isn't found for deletion, an <xref:System.ArgumentNullException> is thrown.</span></span> <span data-ttu-id="fe923-154">L'eccezione viene intercettata dall'istruzione [try-catch](/dotnet/csharp/language-reference/keywords/try-catch) e registrata chiamando il `QuoteDeleteFailed` metodo sul logger nel blocco [catch](/dotnet/csharp/language-reference/keywords/try-catch) (*pages/index. cshtml. cs*):</span><span class="sxs-lookup"><span data-stu-id="fe923-154">The exception is trapped by the [try-catch](/dotnet/csharp/language-reference/keywords/try-catch) statement and logged by calling the `QuoteDeleteFailed` method on the logger in the [catch](/dotnet/csharp/language-reference/keywords/try-catch) block (*Pages/Index.cshtml.cs*):</span></span>

[!code-csharp[](loggermessage/samples/3.x/LoggerMessageSample/Pages/Index.cshtml.cs?name=snippet5&highlight=9,13)]

<span data-ttu-id="fe923-155">Quando un'offerta viene eliminata, esaminare l'output della console dell'app:</span><span class="sxs-lookup"><span data-stu-id="fe923-155">When a quote is successfully deleted, inspect the app's console output:</span></span>

```console
info: LoggerMessageSample.Pages.IndexModel[4]
      => RequestId:0HL90M6E7PHK5:00000016 RequestPath:/ => /Index
      Quote deleted (Quote = 'You can avoid reality, but you cannot avoid the 
          consequences of avoiding reality. - Ayn Rand' Id = 1)
```

<span data-ttu-id="fe923-156">Quando l'eliminazione di un'offerta ha esito negativo, esaminare l'output della console dell'app.</span><span class="sxs-lookup"><span data-stu-id="fe923-156">When quote deletion fails, inspect the app's console output.</span></span> <span data-ttu-id="fe923-157">Si noti che l'eccezione è inclusa nel messaggio di log:</span><span class="sxs-lookup"><span data-stu-id="fe923-157">Note that the exception is included in the log message:</span></span>

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

## <a name="loggermessagedefinescope"></a><span data-ttu-id="fe923-158">LoggerMessage.DefineScope</span><span class="sxs-lookup"><span data-stu-id="fe923-158">LoggerMessage.DefineScope</span></span>

<span data-ttu-id="fe923-159">[DefineScope(String)](xref:Microsoft.Extensions.Logging.LoggerMessage.DefineScope*) crea un delegato <xref:System.Func%601> per definire un [ambito del log](xref:fundamentals/logging/index#log-scopes).</span><span class="sxs-lookup"><span data-stu-id="fe923-159">[DefineScope(String)](xref:Microsoft.Extensions.Logging.LoggerMessage.DefineScope*) creates a <xref:System.Func%601> delegate for defining a [log scope](xref:fundamentals/logging/index#log-scopes).</span></span> <span data-ttu-id="fe923-160">Gli overload <xref:Microsoft.Extensions.Logging.LoggerMessage.DefineScope*> permettono il passaggio di un massimo di tre parametri di tipo in una stringa di formato denominata (modello).</span><span class="sxs-lookup"><span data-stu-id="fe923-160"><xref:Microsoft.Extensions.Logging.LoggerMessage.DefineScope*> overloads permit passing up to three type parameters to a named format string (template).</span></span>

<span data-ttu-id="fe923-161">Come avviene per il metodo <xref:Microsoft.Extensions.Logging.LoggerMessage.Define*>, la stringa specificata nel metodo <xref:Microsoft.Extensions.Logging.LoggerMessage.DefineScope*> è un modello e non una stringa interpolata.</span><span class="sxs-lookup"><span data-stu-id="fe923-161">As is the case with the <xref:Microsoft.Extensions.Logging.LoggerMessage.Define*> method, the string provided to the <xref:Microsoft.Extensions.Logging.LoggerMessage.DefineScope*> method is a template and not an interpolated string.</span></span> <span data-ttu-id="fe923-162">I segnaposto vengono inseriti nell'ordine in cui sono specificati i tipi.</span><span class="sxs-lookup"><span data-stu-id="fe923-162">Placeholders are filled in the order that the types are specified.</span></span> <span data-ttu-id="fe923-163">I nomi dei segnaposto nel modello devono essere descrittivi e coerenti tra i modelli.</span><span class="sxs-lookup"><span data-stu-id="fe923-163">Placeholder names in the template should be descriptive and consistent across templates.</span></span> <span data-ttu-id="fe923-164">Vengono usati come nomi di proprietà all'interno dei dati di log strutturati.</span><span class="sxs-lookup"><span data-stu-id="fe923-164">They serve as property names within structured log data.</span></span> <span data-ttu-id="fe923-165">Per i nomi dei segnaposto è consigliabile usare la [convenzione Pascal](/dotnet/standard/design-guidelines/capitalization-conventions).</span><span class="sxs-lookup"><span data-stu-id="fe923-165">We recommend [Pascal casing](/dotnet/standard/design-guidelines/capitalization-conventions) for placeholder names.</span></span> <span data-ttu-id="fe923-166">Ad esempio, `{Count}`, `{FirstName}`.</span><span class="sxs-lookup"><span data-stu-id="fe923-166">For example, `{Count}`, `{FirstName}`.</span></span>

<span data-ttu-id="fe923-167">Definire un [ambito del log](xref:fundamentals/logging/index#log-scopes) da applicare a una serie di messaggi di log usando il metodo <xref:Microsoft.Extensions.Logging.LoggerMessage.DefineScope*>.</span><span class="sxs-lookup"><span data-stu-id="fe923-167">Define a [log scope](xref:fundamentals/logging/index#log-scopes) to apply to a series of log messages using the <xref:Microsoft.Extensions.Logging.LoggerMessage.DefineScope*> method.</span></span>

<span data-ttu-id="fe923-168">L'app di esempio include un pulsante **Clear All** (Cancella tutto) per eliminare tutte le offerte nel database.</span><span class="sxs-lookup"><span data-stu-id="fe923-168">The sample app has a **Clear All** button for deleting all of the quotes in the database.</span></span> <span data-ttu-id="fe923-169">Le offerte vengono eliminate rimuovendole una alla volta.</span><span class="sxs-lookup"><span data-stu-id="fe923-169">The quotes are deleted by removing them one at a time.</span></span> <span data-ttu-id="fe923-170">Ogni volta che viene eliminata un'offerta, viene chiamato il metodo `QuoteDeleted` nel logger.</span><span class="sxs-lookup"><span data-stu-id="fe923-170">Each time a quote is deleted, the `QuoteDeleted` method is called on the logger.</span></span> <span data-ttu-id="fe923-171">Viene aggiunto un ambito di log a questi messaggi di log.</span><span class="sxs-lookup"><span data-stu-id="fe923-171">A log scope is added to these log messages.</span></span>

<span data-ttu-id="fe923-172">Abilitare `IncludeScopes` nella sezione Logger console di *appsettings.json* :</span><span class="sxs-lookup"><span data-stu-id="fe923-172">Enable `IncludeScopes` in the console logger section of *appsettings.json*:</span></span>

[!code-json[](loggermessage/samples/3.x/LoggerMessageSample/appsettings.json?highlight=3-5)]

<span data-ttu-id="fe923-173">Per creare un ambito di log, aggiungere un campo per inserire un delegato <xref:System.Func%601> per l'ambito.</span><span class="sxs-lookup"><span data-stu-id="fe923-173">To create a log scope, add a field to hold a <xref:System.Func%601> delegate for the scope.</span></span> <span data-ttu-id="fe923-174">L'app di esempio crea un campo denominato `_allQuotesDeletedScope` (*Internal/LoggerExtensions.cs*):</span><span class="sxs-lookup"><span data-stu-id="fe923-174">The sample app creates a field called `_allQuotesDeletedScope` (*Internal/LoggerExtensions.cs*):</span></span>

[!code-csharp[](loggermessage/samples/3.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet4)]

<span data-ttu-id="fe923-175">Usare <xref:Microsoft.Extensions.Logging.LoggerMessage.DefineScope*> per creare il delegato.</span><span class="sxs-lookup"><span data-stu-id="fe923-175">Use <xref:Microsoft.Extensions.Logging.LoggerMessage.DefineScope*> to create the delegate.</span></span> <span data-ttu-id="fe923-176">È possibile specificare un massimo di tre tipi da usare come argomenti di modello quando viene richiamato il delegato.</span><span class="sxs-lookup"><span data-stu-id="fe923-176">Up to three types can be specified for use as template arguments when the delegate is invoked.</span></span> <span data-ttu-id="fe923-177">L'app di esempio usa un modello di messaggio che include il numero di offerte eliminate (un tipo `int`):</span><span class="sxs-lookup"><span data-stu-id="fe923-177">The sample app uses a message template that includes the number of deleted quotes (an `int` type):</span></span>

[!code-csharp[](loggermessage/samples/3.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet8)]

<span data-ttu-id="fe923-178">Specificare un metodo di estensione statico per il messaggio di log.</span><span class="sxs-lookup"><span data-stu-id="fe923-178">Provide a static extension method for the log message.</span></span> <span data-ttu-id="fe923-179">Includere i parametri di tipo per le proprietà denominate visualizzate nel modello di messaggio.</span><span class="sxs-lookup"><span data-stu-id="fe923-179">Include any type parameters for named properties that appear in the message template.</span></span> <span data-ttu-id="fe923-180">L'app di esempio accetta un numero `count` di offerte da eliminare e restituisce `_allQuotesDeletedScope`:</span><span class="sxs-lookup"><span data-stu-id="fe923-180">The sample app takes in a `count` of quotes to delete and returns `_allQuotesDeletedScope`:</span></span>

[!code-csharp[](loggermessage/samples/3.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet12)]

<span data-ttu-id="fe923-181">L'ambito esegue il wrapping delle chiamate di estensione di registrazione in un blocco [using](/dotnet/csharp/language-reference/keywords/using-statement):</span><span class="sxs-lookup"><span data-stu-id="fe923-181">The scope wraps the logging extension calls in a [using](/dotnet/csharp/language-reference/keywords/using-statement) block:</span></span>

[!code-csharp[](loggermessage/samples/3.x/LoggerMessageSample/Pages/Index.cshtml.cs?name=snippet4&highlight=5-6,14)]

<span data-ttu-id="fe923-182">Esaminare i messaggi di log nell'output della console dell'app.</span><span class="sxs-lookup"><span data-stu-id="fe923-182">Inspect the log messages in the app's console output.</span></span> <span data-ttu-id="fe923-183">Il risultato seguente mostra tre offerte eliminate con il messaggio di ambito di log incluso:</span><span class="sxs-lookup"><span data-stu-id="fe923-183">The following result shows three quotes deleted with the log scope message included:</span></span>

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

<span data-ttu-id="fe923-184">Le funzionalità di <xref:Microsoft.Extensions.Logging.LoggerMessage> creano delegati inseribili nella cache che richiedono un numero minore di allocazioni di oggetti e riducono il sovraccarico di calcolo rispetto ai [metodi di estensione del logger](xref:Microsoft.Extensions.Logging.LoggerExtensions), ad esempio <xref:Microsoft.Extensions.Logging.LoggerExtensions.LogInformation*> e <xref:Microsoft.Extensions.Logging.LoggerExtensions.LogDebug*>.</span><span class="sxs-lookup"><span data-stu-id="fe923-184"><xref:Microsoft.Extensions.Logging.LoggerMessage> features create cacheable delegates that require fewer object allocations and reduced computational overhead compared to [logger extension methods](xref:Microsoft.Extensions.Logging.LoggerExtensions), such as <xref:Microsoft.Extensions.Logging.LoggerExtensions.LogInformation*> and <xref:Microsoft.Extensions.Logging.LoggerExtensions.LogDebug*>.</span></span> <span data-ttu-id="fe923-185">Per gli scenari di registrazione a prestazioni elevate, usare <xref:Microsoft.Extensions.Logging.LoggerMessage>.</span><span class="sxs-lookup"><span data-stu-id="fe923-185">For high-performance logging scenarios, use the <xref:Microsoft.Extensions.Logging.LoggerMessage> pattern.</span></span>

<span data-ttu-id="fe923-186"><xref:Microsoft.Extensions.Logging.LoggerMessage> offre i seguenti vantaggi in termini di prestazioni rispetto ai metodi di estensione del logger:</span><span class="sxs-lookup"><span data-stu-id="fe923-186"><xref:Microsoft.Extensions.Logging.LoggerMessage> provides the following performance advantages over Logger extension methods:</span></span>

* <span data-ttu-id="fe923-187">I metodi di estensione del logger richiedono una "conversione boxing" dei tipi di valori, ad esempio `int`, in `object`.</span><span class="sxs-lookup"><span data-stu-id="fe923-187">Logger extension methods require "boxing" (converting) value types, such as `int`, into `object`.</span></span> <span data-ttu-id="fe923-188"><xref:Microsoft.Extensions.Logging.LoggerMessage> evita la conversione boxing usando campi <xref:System.Action> statici e metodi di estensione con parametri fortemente tipizzati.</span><span class="sxs-lookup"><span data-stu-id="fe923-188">The <xref:Microsoft.Extensions.Logging.LoggerMessage> pattern avoids boxing by using static <xref:System.Action> fields and extension methods with strongly-typed parameters.</span></span>
* <span data-ttu-id="fe923-189">I metodi di estensione del logger devono analizzare il modello di messaggio (stringa di formato denominata) ogni volta che viene scritto un messaggio del log.</span><span class="sxs-lookup"><span data-stu-id="fe923-189">Logger extension methods must parse the message template (named format string) every time a log message is written.</span></span> <span data-ttu-id="fe923-190">Solo <xref:Microsoft.Extensions.Logging.LoggerMessage> richiede una sola analisi del modello durante la definizione del messaggio.</span><span class="sxs-lookup"><span data-stu-id="fe923-190"><xref:Microsoft.Extensions.Logging.LoggerMessage> only requires parsing a template once when the message is defined.</span></span>

<span data-ttu-id="fe923-191">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/logging/loggermessage/samples/) ([procedura per il download](xref:index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="fe923-191">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/logging/loggermessage/samples/) ([how to download](xref:index#how-to-download-a-sample))</span></span>

<span data-ttu-id="fe923-192">L'app di esempio illustra le funzionalità di <xref:Microsoft.Extensions.Logging.LoggerMessage> con un sistema di verifica delle offerte di base.</span><span class="sxs-lookup"><span data-stu-id="fe923-192">The sample app demonstrates <xref:Microsoft.Extensions.Logging.LoggerMessage> features with a basic quote tracking system.</span></span> <span data-ttu-id="fe923-193">L'app aggiunge ed elimina le offerte usando un database in memoria.</span><span class="sxs-lookup"><span data-stu-id="fe923-193">The app adds and deletes quotes using an in-memory database.</span></span> <span data-ttu-id="fe923-194">Durante l'esecuzione di queste operazioni, i messaggi del log vengono generati usando <xref:Microsoft.Extensions.Logging.LoggerMessage>.</span><span class="sxs-lookup"><span data-stu-id="fe923-194">As these operations occur, log messages are generated using the <xref:Microsoft.Extensions.Logging.LoggerMessage> pattern.</span></span>

## <a name="loggermessagedefine"></a><span data-ttu-id="fe923-195">LoggerMessage.Define</span><span class="sxs-lookup"><span data-stu-id="fe923-195">LoggerMessage.Define</span></span>

<span data-ttu-id="fe923-196">[Define(LogLevel, EventId, String)](xref:Microsoft.Extensions.Logging.LoggerMessage.Define*) crea un delegato <xref:System.Action> per la registrazione di un messaggio.</span><span class="sxs-lookup"><span data-stu-id="fe923-196">[Define(LogLevel, EventId, String)](xref:Microsoft.Extensions.Logging.LoggerMessage.Define*) creates an <xref:System.Action> delegate for logging a message.</span></span> <span data-ttu-id="fe923-197">Gli overload <xref:Microsoft.Extensions.Logging.LoggerMessage.Define*> permettono il passaggio di un massimo di sei parametri di tipo in una stringa di formato denominata (modello).</span><span class="sxs-lookup"><span data-stu-id="fe923-197"><xref:Microsoft.Extensions.Logging.LoggerMessage.Define*> overloads permit passing up to six type parameters to a named format string (template).</span></span>

<span data-ttu-id="fe923-198">La stringa specificata nel metodo <xref:Microsoft.Extensions.Logging.LoggerMessage.Define*> è un modello e non una stringa interpolata.</span><span class="sxs-lookup"><span data-stu-id="fe923-198">The string provided to the <xref:Microsoft.Extensions.Logging.LoggerMessage.Define*> method is a template and not an interpolated string.</span></span> <span data-ttu-id="fe923-199">I segnaposto vengono inseriti nell'ordine in cui sono specificati i tipi.</span><span class="sxs-lookup"><span data-stu-id="fe923-199">Placeholders are filled in the order that the types are specified.</span></span> <span data-ttu-id="fe923-200">I nomi dei segnaposto nel modello devono essere descrittivi e coerenti tra i modelli.</span><span class="sxs-lookup"><span data-stu-id="fe923-200">Placeholder names in the template should be descriptive and consistent across templates.</span></span> <span data-ttu-id="fe923-201">Vengono usati come nomi di proprietà all'interno dei dati di log strutturati.</span><span class="sxs-lookup"><span data-stu-id="fe923-201">They serve as property names within structured log data.</span></span> <span data-ttu-id="fe923-202">Per i nomi dei segnaposto è consigliabile usare la [convenzione Pascal](/dotnet/standard/design-guidelines/capitalization-conventions).</span><span class="sxs-lookup"><span data-stu-id="fe923-202">We recommend [Pascal casing](/dotnet/standard/design-guidelines/capitalization-conventions) for placeholder names.</span></span> <span data-ttu-id="fe923-203">Ad esempio, `{Count}`, `{FirstName}`.</span><span class="sxs-lookup"><span data-stu-id="fe923-203">For example, `{Count}`, `{FirstName}`.</span></span>

<span data-ttu-id="fe923-204">Ogni messaggio di log è un elemento <xref:System.Action> contenuto in un campo statico creato da [LoggerMessage.Define](xref:Microsoft.Extensions.Logging.LoggerMessage.Define*).</span><span class="sxs-lookup"><span data-stu-id="fe923-204">Each log message is an <xref:System.Action> held in a static field created by [LoggerMessage.Define](xref:Microsoft.Extensions.Logging.LoggerMessage.Define*).</span></span> <span data-ttu-id="fe923-205">Ad esempio, l'app di esempio crea un campo per descrivere un messaggio di log per una richiesta GET per la pagina di indice (*Internal/LoggerExtensions.cs*):</span><span class="sxs-lookup"><span data-stu-id="fe923-205">For example, the sample app creates a field to describe a log message for a GET request for the Index page (*Internal/LoggerExtensions.cs*):</span></span>

[!code-csharp[](loggermessage/samples/2.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet1)]

<span data-ttu-id="fe923-206">Per <xref:System.Action>, specificare:</span><span class="sxs-lookup"><span data-stu-id="fe923-206">For the <xref:System.Action>, specify:</span></span>

* <span data-ttu-id="fe923-207">Il livello del log.</span><span class="sxs-lookup"><span data-stu-id="fe923-207">The log level.</span></span>
* <span data-ttu-id="fe923-208">Un identificatore di evento univoco (<xref:Microsoft.Extensions.Logging.EventId>) con il nome del metodo di estensione statico.</span><span class="sxs-lookup"><span data-stu-id="fe923-208">A unique event identifier (<xref:Microsoft.Extensions.Logging.EventId>) with the name of the static extension method.</span></span>
* <span data-ttu-id="fe923-209">Il modello di messaggio (stringa di formato denominata).</span><span class="sxs-lookup"><span data-stu-id="fe923-209">The message template (named format string).</span></span> 

<span data-ttu-id="fe923-210">Una richiesta per la pagina di indice dell'app di esempio imposta:</span><span class="sxs-lookup"><span data-stu-id="fe923-210">A request for the Index page of the sample app sets the:</span></span>

* <span data-ttu-id="fe923-211">Il livello del log su `Information`.</span><span class="sxs-lookup"><span data-stu-id="fe923-211">Log level to `Information`.</span></span>
* <span data-ttu-id="fe923-212">L'ID evento su `1` con il nome del metodo `IndexPageRequested`.</span><span class="sxs-lookup"><span data-stu-id="fe923-212">Event id to `1` with the name of the `IndexPageRequested` method.</span></span>
* <span data-ttu-id="fe923-213">Il modello di messaggio (stringa di formato denominata) su una stringa.</span><span class="sxs-lookup"><span data-stu-id="fe923-213">Message template (named format string) to a string.</span></span>

[!code-csharp[](loggermessage/samples/2.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet5)]

<span data-ttu-id="fe923-214">Negli archivi di log strutturati è possibile che venga usato il nome evento quando viene specificato con l'ID evento per arricchire la registrazione.</span><span class="sxs-lookup"><span data-stu-id="fe923-214">Structured logging stores may use the event name when it's supplied with the event id to enrich logging.</span></span> <span data-ttu-id="fe923-215">Ad esempio, [Serilog](https://github.com/serilog/serilog-extensions-logging) usa il nome evento.</span><span class="sxs-lookup"><span data-stu-id="fe923-215">For example, [Serilog](https://github.com/serilog/serilog-extensions-logging) uses the event name.</span></span>

<span data-ttu-id="fe923-216"><xref:System.Action> viene richiamato attraverso un metodo di estensione fortemente tipizzato.</span><span class="sxs-lookup"><span data-stu-id="fe923-216">The <xref:System.Action> is invoked through a strongly-typed extension method.</span></span> <span data-ttu-id="fe923-217">Il metodo `IndexPageRequested` registra un messaggio per la richiesta GET di una pagina di indice nell'app di esempio:</span><span class="sxs-lookup"><span data-stu-id="fe923-217">The `IndexPageRequested` method logs a message for an Index page GET request in the sample app:</span></span>

[!code-csharp[](loggermessage/samples/2.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet9)]

<span data-ttu-id="fe923-218">`IndexPageRequested` viene chiamato nel logger nel metodo `OnGetAsync` in *Pages/Index.cshtml.cs*:</span><span class="sxs-lookup"><span data-stu-id="fe923-218">`IndexPageRequested` is called on the logger in the `OnGetAsync` method in *Pages/Index.cshtml.cs*:</span></span>

[!code-csharp[](loggermessage/samples/2.x/LoggerMessageSample/Pages/Index.cshtml.cs?name=snippet2&highlight=3)]

<span data-ttu-id="fe923-219">Esaminare l'output della console dell'app:</span><span class="sxs-lookup"><span data-stu-id="fe923-219">Inspect the app's console output:</span></span>

```console
info: LoggerMessageSample.Pages.IndexModel[1]
      => RequestId:0HL90M6E7PHK4:00000001 RequestPath:/ => /Index
      GET request for Index page
```

<span data-ttu-id="fe923-220">Per passare i parametri in un messaggio di log, definire un massimo di sei tipi durante la creazione del campo statico.</span><span class="sxs-lookup"><span data-stu-id="fe923-220">To pass parameters to a log message, define up to six types when creating the static field.</span></span> <span data-ttu-id="fe923-221">L'app di esempio registra una stringa quando viene aggiunta un'offerta definendo un tipo `string` per il campo <xref:System.Action>:</span><span class="sxs-lookup"><span data-stu-id="fe923-221">The sample app logs a string when adding a quote by defining a `string` type for the <xref:System.Action> field:</span></span>

[!code-csharp[](loggermessage/samples/2.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet2)]

<span data-ttu-id="fe923-222">Il modello di messaggio di log del delegato riceve i valori dei segnaposto dai tipi specificati.</span><span class="sxs-lookup"><span data-stu-id="fe923-222">The delegate's log message template receives its placeholder values from the types provided.</span></span> <span data-ttu-id="fe923-223">L'app di esempio definisce un delegato per l'aggiunta di un'offerta dove il parametro dell'offerta è un elemento `string`:</span><span class="sxs-lookup"><span data-stu-id="fe923-223">The sample app defines a delegate for adding a quote where the quote parameter is a `string`:</span></span>

[!code-csharp[](loggermessage/samples/2.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet6)]

<span data-ttu-id="fe923-224">Il metodo di estensione statico per l'aggiunta di un'offerta, `QuoteAdded`, riceve il valore dell'argomento dell'offerta e lo passa al delegato <xref:System.Action>:</span><span class="sxs-lookup"><span data-stu-id="fe923-224">The static extension method for adding a quote, `QuoteAdded`, receives the quote argument value and passes it to the <xref:System.Action> delegate:</span></span>

[!code-csharp[](loggermessage/samples/2.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet10)]

<span data-ttu-id="fe923-225">Nel modello di pagina della pagina di indice (*pages/index. cshtml. cs*) `QuoteAdded` viene chiamato il metodo per registrare il messaggio:</span><span class="sxs-lookup"><span data-stu-id="fe923-225">In the Index page's page model (*Pages/Index.cshtml.cs*), `QuoteAdded` is called to log the message:</span></span>

[!code-csharp[](loggermessage/samples/2.x/LoggerMessageSample/Pages/Index.cshtml.cs?name=snippet3&highlight=6)]

<span data-ttu-id="fe923-226">Esaminare l'output della console dell'app:</span><span class="sxs-lookup"><span data-stu-id="fe923-226">Inspect the app's console output:</span></span>

```console
info: LoggerMessageSample.Pages.IndexModel[2]
      => RequestId:0HL90M6E7PHK5:0000000A RequestPath:/ => /Index
      Quote added (Quote = 'You can avoid reality, but you cannot avoid the 
          consequences of avoiding reality. - Ayn Rand')
```

<span data-ttu-id="fe923-227">L'app di esempio implementa un modello [try-catch](/dotnet/csharp/language-reference/keywords/try-catch) per l'eliminazione delle virgolette.</span><span class="sxs-lookup"><span data-stu-id="fe923-227">The sample app implements a [try-catch](/dotnet/csharp/language-reference/keywords/try-catch) pattern for quote deletion.</span></span> <span data-ttu-id="fe923-228">Per un'operazione di eliminazione con esito positivo, viene registrato un messaggio informativo.</span><span class="sxs-lookup"><span data-stu-id="fe923-228">An informational message is logged for a successful delete operation.</span></span> <span data-ttu-id="fe923-229">Per un'operazione di eliminazione con la generazione di un'eccezione, viene registrato un messaggio di errore.</span><span class="sxs-lookup"><span data-stu-id="fe923-229">An error message is logged for a delete operation when an exception is thrown.</span></span> <span data-ttu-id="fe923-230">Il messaggio di log per un'operazione di eliminazione con esito negativo include l'analisi dello stack dell'eccezione (*Internal/LoggerExtensions.cs*):</span><span class="sxs-lookup"><span data-stu-id="fe923-230">The log message for the unsuccessful delete operation includes the exception stack trace (*Internal/LoggerExtensions.cs*):</span></span>

[!code-csharp[](loggermessage/samples/2.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet3)]

[!code-csharp[](loggermessage/samples/2.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet7)]

<span data-ttu-id="fe923-231">Si noti come l'eccezione viene passata al delegato in `QuoteDeleteFailed`:</span><span class="sxs-lookup"><span data-stu-id="fe923-231">Note how the exception is passed to the delegate in `QuoteDeleteFailed`:</span></span>

[!code-csharp[](loggermessage/samples/2.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet11)]

<span data-ttu-id="fe923-232">Nel modello di pagina per la pagina di indice l'eliminazione di un'offerta con esito positivo chiama il metodo `QuoteDeleted` nel logger.</span><span class="sxs-lookup"><span data-stu-id="fe923-232">In the page model for the Index page, a successful quote deletion calls the `QuoteDeleted` method on the logger.</span></span> <span data-ttu-id="fe923-233">Quando non viene trovata un'offerta per l'eliminazione, viene generata un'eccezione <xref:System.ArgumentNullException>.</span><span class="sxs-lookup"><span data-stu-id="fe923-233">When a quote isn't found for deletion, an <xref:System.ArgumentNullException> is thrown.</span></span> <span data-ttu-id="fe923-234">L'eccezione viene intercettata dall'istruzione [try-catch](/dotnet/csharp/language-reference/keywords/try-catch) e registrata chiamando il `QuoteDeleteFailed` metodo sul logger nel blocco [catch](/dotnet/csharp/language-reference/keywords/try-catch) (*pages/index. cshtml. cs*):</span><span class="sxs-lookup"><span data-stu-id="fe923-234">The exception is trapped by the [try-catch](/dotnet/csharp/language-reference/keywords/try-catch) statement and logged by calling the `QuoteDeleteFailed` method on the logger in the [catch](/dotnet/csharp/language-reference/keywords/try-catch) block (*Pages/Index.cshtml.cs*):</span></span>

[!code-csharp[](loggermessage/samples/2.x/LoggerMessageSample/Pages/Index.cshtml.cs?name=snippet5&highlight=14,18)]

<span data-ttu-id="fe923-235">Quando un'offerta viene eliminata, esaminare l'output della console dell'app:</span><span class="sxs-lookup"><span data-stu-id="fe923-235">When a quote is successfully deleted, inspect the app's console output:</span></span>

```console
info: LoggerMessageSample.Pages.IndexModel[4]
      => RequestId:0HL90M6E7PHK5:00000016 RequestPath:/ => /Index
      Quote deleted (Quote = 'You can avoid reality, but you cannot avoid the 
          consequences of avoiding reality. - Ayn Rand' Id = 1)
```

<span data-ttu-id="fe923-236">Quando l'eliminazione di un'offerta ha esito negativo, esaminare l'output della console dell'app.</span><span class="sxs-lookup"><span data-stu-id="fe923-236">When quote deletion fails, inspect the app's console output.</span></span> <span data-ttu-id="fe923-237">Si noti che l'eccezione è inclusa nel messaggio di log:</span><span class="sxs-lookup"><span data-stu-id="fe923-237">Note that the exception is included in the log message:</span></span>

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

## <a name="loggermessagedefinescope"></a><span data-ttu-id="fe923-238">LoggerMessage.DefineScope</span><span class="sxs-lookup"><span data-stu-id="fe923-238">LoggerMessage.DefineScope</span></span>

<span data-ttu-id="fe923-239">[DefineScope(String)](xref:Microsoft.Extensions.Logging.LoggerMessage.DefineScope*) crea un delegato <xref:System.Func%601> per definire un [ambito del log](xref:fundamentals/logging/index#log-scopes).</span><span class="sxs-lookup"><span data-stu-id="fe923-239">[DefineScope(String)](xref:Microsoft.Extensions.Logging.LoggerMessage.DefineScope*) creates a <xref:System.Func%601> delegate for defining a [log scope](xref:fundamentals/logging/index#log-scopes).</span></span> <span data-ttu-id="fe923-240">Gli overload <xref:Microsoft.Extensions.Logging.LoggerMessage.DefineScope*> permettono il passaggio di un massimo di tre parametri di tipo in una stringa di formato denominata (modello).</span><span class="sxs-lookup"><span data-stu-id="fe923-240"><xref:Microsoft.Extensions.Logging.LoggerMessage.DefineScope*> overloads permit passing up to three type parameters to a named format string (template).</span></span>

<span data-ttu-id="fe923-241">Come avviene per il metodo <xref:Microsoft.Extensions.Logging.LoggerMessage.Define*>, la stringa specificata nel metodo <xref:Microsoft.Extensions.Logging.LoggerMessage.DefineScope*> è un modello e non una stringa interpolata.</span><span class="sxs-lookup"><span data-stu-id="fe923-241">As is the case with the <xref:Microsoft.Extensions.Logging.LoggerMessage.Define*> method, the string provided to the <xref:Microsoft.Extensions.Logging.LoggerMessage.DefineScope*> method is a template and not an interpolated string.</span></span> <span data-ttu-id="fe923-242">I segnaposto vengono inseriti nell'ordine in cui sono specificati i tipi.</span><span class="sxs-lookup"><span data-stu-id="fe923-242">Placeholders are filled in the order that the types are specified.</span></span> <span data-ttu-id="fe923-243">I nomi dei segnaposto nel modello devono essere descrittivi e coerenti tra i modelli.</span><span class="sxs-lookup"><span data-stu-id="fe923-243">Placeholder names in the template should be descriptive and consistent across templates.</span></span> <span data-ttu-id="fe923-244">Vengono usati come nomi di proprietà all'interno dei dati di log strutturati.</span><span class="sxs-lookup"><span data-stu-id="fe923-244">They serve as property names within structured log data.</span></span> <span data-ttu-id="fe923-245">Per i nomi dei segnaposto è consigliabile usare la [convenzione Pascal](/dotnet/standard/design-guidelines/capitalization-conventions).</span><span class="sxs-lookup"><span data-stu-id="fe923-245">We recommend [Pascal casing](/dotnet/standard/design-guidelines/capitalization-conventions) for placeholder names.</span></span> <span data-ttu-id="fe923-246">Ad esempio, `{Count}`, `{FirstName}`.</span><span class="sxs-lookup"><span data-stu-id="fe923-246">For example, `{Count}`, `{FirstName}`.</span></span>

<span data-ttu-id="fe923-247">Definire un [ambito del log](xref:fundamentals/logging/index#log-scopes) da applicare a una serie di messaggi di log usando il metodo <xref:Microsoft.Extensions.Logging.LoggerMessage.DefineScope*>.</span><span class="sxs-lookup"><span data-stu-id="fe923-247">Define a [log scope](xref:fundamentals/logging/index#log-scopes) to apply to a series of log messages using the <xref:Microsoft.Extensions.Logging.LoggerMessage.DefineScope*> method.</span></span>

<span data-ttu-id="fe923-248">L'app di esempio include un pulsante **Clear All** (Cancella tutto) per eliminare tutte le offerte nel database.</span><span class="sxs-lookup"><span data-stu-id="fe923-248">The sample app has a **Clear All** button for deleting all of the quotes in the database.</span></span> <span data-ttu-id="fe923-249">Le offerte vengono eliminate rimuovendole una alla volta.</span><span class="sxs-lookup"><span data-stu-id="fe923-249">The quotes are deleted by removing them one at a time.</span></span> <span data-ttu-id="fe923-250">Ogni volta che viene eliminata un'offerta, viene chiamato il metodo `QuoteDeleted` nel logger.</span><span class="sxs-lookup"><span data-stu-id="fe923-250">Each time a quote is deleted, the `QuoteDeleted` method is called on the logger.</span></span> <span data-ttu-id="fe923-251">Viene aggiunto un ambito di log a questi messaggi di log.</span><span class="sxs-lookup"><span data-stu-id="fe923-251">A log scope is added to these log messages.</span></span>

<span data-ttu-id="fe923-252">Abilitare `IncludeScopes` nella sezione Logger console di *appsettings.json* :</span><span class="sxs-lookup"><span data-stu-id="fe923-252">Enable `IncludeScopes` in the console logger section of *appsettings.json*:</span></span>

[!code-json[](loggermessage/samples/2.x/LoggerMessageSample/appsettings.json?highlight=3-5)]

<span data-ttu-id="fe923-253">Per creare un ambito di log, aggiungere un campo per inserire un delegato <xref:System.Func%601> per l'ambito.</span><span class="sxs-lookup"><span data-stu-id="fe923-253">To create a log scope, add a field to hold a <xref:System.Func%601> delegate for the scope.</span></span> <span data-ttu-id="fe923-254">L'app di esempio crea un campo denominato `_allQuotesDeletedScope` (*Internal/LoggerExtensions.cs*):</span><span class="sxs-lookup"><span data-stu-id="fe923-254">The sample app creates a field called `_allQuotesDeletedScope` (*Internal/LoggerExtensions.cs*):</span></span>

[!code-csharp[](loggermessage/samples/2.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet4)]

<span data-ttu-id="fe923-255">Usare <xref:Microsoft.Extensions.Logging.LoggerMessage.DefineScope*> per creare il delegato.</span><span class="sxs-lookup"><span data-stu-id="fe923-255">Use <xref:Microsoft.Extensions.Logging.LoggerMessage.DefineScope*> to create the delegate.</span></span> <span data-ttu-id="fe923-256">È possibile specificare un massimo di tre tipi da usare come argomenti di modello quando viene richiamato il delegato.</span><span class="sxs-lookup"><span data-stu-id="fe923-256">Up to three types can be specified for use as template arguments when the delegate is invoked.</span></span> <span data-ttu-id="fe923-257">L'app di esempio usa un modello di messaggio che include il numero di offerte eliminate (un tipo `int`):</span><span class="sxs-lookup"><span data-stu-id="fe923-257">The sample app uses a message template that includes the number of deleted quotes (an `int` type):</span></span>

[!code-csharp[](loggermessage/samples/2.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet8)]

<span data-ttu-id="fe923-258">Specificare un metodo di estensione statico per il messaggio di log.</span><span class="sxs-lookup"><span data-stu-id="fe923-258">Provide a static extension method for the log message.</span></span> <span data-ttu-id="fe923-259">Includere i parametri di tipo per le proprietà denominate visualizzate nel modello di messaggio.</span><span class="sxs-lookup"><span data-stu-id="fe923-259">Include any type parameters for named properties that appear in the message template.</span></span> <span data-ttu-id="fe923-260">L'app di esempio accetta un numero `count` di offerte da eliminare e restituisce `_allQuotesDeletedScope`:</span><span class="sxs-lookup"><span data-stu-id="fe923-260">The sample app takes in a `count` of quotes to delete and returns `_allQuotesDeletedScope`:</span></span>

[!code-csharp[](loggermessage/samples/2.x/LoggerMessageSample/Internal/LoggerExtensions.cs?name=snippet12)]

<span data-ttu-id="fe923-261">L'ambito esegue il wrapping delle chiamate di estensione di registrazione in un blocco [using](/dotnet/csharp/language-reference/keywords/using-statement):</span><span class="sxs-lookup"><span data-stu-id="fe923-261">The scope wraps the logging extension calls in a [using](/dotnet/csharp/language-reference/keywords/using-statement) block:</span></span>

[!code-csharp[](loggermessage/samples/2.x/LoggerMessageSample/Pages/Index.cshtml.cs?name=snippet4&highlight=5-6,14)]

<span data-ttu-id="fe923-262">Esaminare i messaggi di log nell'output della console dell'app.</span><span class="sxs-lookup"><span data-stu-id="fe923-262">Inspect the log messages in the app's console output.</span></span> <span data-ttu-id="fe923-263">Il risultato seguente mostra tre offerte eliminate con il messaggio di ambito di log incluso:</span><span class="sxs-lookup"><span data-stu-id="fe923-263">The following result shows three quotes deleted with the log scope message included:</span></span>

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

## <a name="additional-resources"></a><span data-ttu-id="fe923-264">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="fe923-264">Additional resources</span></span>

* [<span data-ttu-id="fe923-265">Logging</span><span class="sxs-lookup"><span data-stu-id="fe923-265">Logging</span></span>](xref:fundamentals/logging/index)
