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
ms.openlocfilehash: e65983fb1a440057283111ea5a79a79b765607b7
ms.sourcegitcommit: 610936e4d3507f7f3d467ed7859ab9354ec158ba
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 01/25/2021
ms.locfileid: "98751679"
---
# <a name="handle-errors-in-aspnet-core"></a><span data-ttu-id="ab1e7-103">Gestire gli errori in ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="ab1e7-103">Handle errors in ASP.NET Core</span></span>

::: moniker range=">= aspnetcore-5.0"

<span data-ttu-id="ab1e7-104">Di [Kirk Larkin](https://twitter.com/serpent5), [Tom Dykstra](https://github.com/tdykstra/)e [Steve Smith](https://ardalis.com/)</span><span class="sxs-lookup"><span data-stu-id="ab1e7-104">By [Kirk Larkin](https://twitter.com/serpent5), [Tom Dykstra](https://github.com/tdykstra/), and [Steve Smith](https://ardalis.com/)</span></span>

<span data-ttu-id="ab1e7-105">Questo articolo illustra gli approcci comuni per la gestione degli errori nelle app Web ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-105">This article covers common approaches to handling errors in ASP.NET Core web apps.</span></span> <span data-ttu-id="ab1e7-106">Vedere <xref:web-api/handle-errors> per le API Web.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-106">See <xref:web-api/handle-errors> for web APIs.</span></span>

<span data-ttu-id="ab1e7-107">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/error-handling/samples).</span><span class="sxs-lookup"><span data-stu-id="ab1e7-107">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/error-handling/samples).</span></span> <span data-ttu-id="ab1e7-108">([Come scaricare](xref:index#how-to-download-a-sample)). La scheda rete di strumenti di sviluppo del browser F12 è utile quando si esegue il test dell'applicazione di esempio.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-108">([How to download](xref:index#how-to-download-a-sample).) The network tab on the F12 browser developer tools is useful when testing the sample app.</span></span>

## <a name="developer-exception-page"></a><span data-ttu-id="ab1e7-109">Pagina delle eccezioni per gli sviluppatori</span><span class="sxs-lookup"><span data-stu-id="ab1e7-109">Developer Exception Page</span></span>

<span data-ttu-id="ab1e7-110">In *Pagina delle eccezioni per gli sviluppatori* sono disponibili informazioni dettagliate sulle eccezioni delle richieste.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-110">The *Developer Exception Page* displays detailed information about request exceptions.</span></span> <span data-ttu-id="ab1e7-111">I modelli di ASP.NET Core generano il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="ab1e7-111">The ASP.NET Core templates generate the following code:</span></span>

[!code-csharp[](error-handling/samples/5.x/ErrorHandlingSample/Startup.cs?name=snippet&highlight=3-6)]

<span data-ttu-id="ab1e7-112">Il codice evidenziato precedente Abilita la pagina delle eccezioni per gli sviluppatori quando l'app è in esecuzione nell' [ambiente di sviluppo](xref:fundamentals/environments).</span><span class="sxs-lookup"><span data-stu-id="ab1e7-112">The preceding highlighted code enables the developer exception page when the app is running in the [Development environment](xref:fundamentals/environments).</span></span>

<span data-ttu-id="ab1e7-113">I modelli si posizionano all' <xref:Microsoft.AspNetCore.Builder.DeveloperExceptionPageExtensions.UseDeveloperExceptionPage%2A> inizio della pipeline del middleware, in modo da poter intercettare le eccezioni generate nel middleware che segue.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-113">The templates place <xref:Microsoft.AspNetCore.Builder.DeveloperExceptionPageExtensions.UseDeveloperExceptionPage%2A> early in the middleware pipeline so that it can catch exceptions thrown in middleware that follows.</span></span>

<span data-ttu-id="ab1e7-114">Il codice precedente Abilita la pagina di eccezione Developer \***solo** quando l'app è in esecuzione nell'ambiente di sviluppo.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-114">The preceding code enables the Developer Exception Page \***only** _ when the app runs in the Development environment.</span></span> <span data-ttu-id="ab1e7-115">Le informazioni dettagliate sulle eccezioni non devono essere visualizzate pubblicamente quando l'app viene eseguita nell'ambiente di produzione.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-115">Detailed exception information should not be displayed publicly when the app runs in the Production environment.</span></span> <span data-ttu-id="ab1e7-116">Per altre informazioni sulla configurazione di ambienti, vedere <xref:fundamentals/environments>.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-116">For more information on configuring environments, see <xref:fundamentals/environments>.</span></span>

<span data-ttu-id="ab1e7-117">La pagina delle eccezioni per sviluppatori include le informazioni seguenti sull'eccezione e la richiesta:</span><span class="sxs-lookup"><span data-stu-id="ab1e7-117">The Developer Exception Page includes the following information about the exception and the request:</span></span>

<span data-ttu-id="ab1e7-118">_ Traccia dello stack</span><span class="sxs-lookup"><span data-stu-id="ab1e7-118">_ Stack trace</span></span>
* <span data-ttu-id="ab1e7-119">Parametri della stringa di query se presenti</span><span class="sxs-lookup"><span data-stu-id="ab1e7-119">Query string parameters if any</span></span>
* <span data-ttu-id="ab1e7-120">Cookies se presente</span><span class="sxs-lookup"><span data-stu-id="ab1e7-120">Cookies if any</span></span>
* <span data-ttu-id="ab1e7-121">Intestazioni</span><span class="sxs-lookup"><span data-stu-id="ab1e7-121">Headers</span></span>

## <a name="exception-handler-page"></a><span data-ttu-id="ab1e7-122">Pagina del gestore di eccezioni</span><span class="sxs-lookup"><span data-stu-id="ab1e7-122">Exception handler page</span></span>

<span data-ttu-id="ab1e7-123">Per configurare una pagina di gestione degli errori personalizzata per l' [ambiente di produzione](xref:fundamentals/environments), chiamare <xref:Microsoft.AspNetCore.Builder.ExceptionHandlerExtensions.UseExceptionHandler%2A> .</span><span class="sxs-lookup"><span data-stu-id="ab1e7-123">To configure a custom error handling page for the [Production environment](xref:fundamentals/environments), call <xref:Microsoft.AspNetCore.Builder.ExceptionHandlerExtensions.UseExceptionHandler%2A>.</span></span> <span data-ttu-id="ab1e7-124">Questo middleware di gestione delle eccezioni:</span><span class="sxs-lookup"><span data-stu-id="ab1e7-124">This exception handling middleware:</span></span>

* <span data-ttu-id="ab1e7-125">Intercetta e registra le eccezioni.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-125">Catches and logs exceptions.</span></span>
* <span data-ttu-id="ab1e7-126">Esegue nuovamente la richiesta in una pipeline alternativa usando il percorso indicato.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-126">Re-executes the request in an alternate pipeline using the path indicated.</span></span> <span data-ttu-id="ab1e7-127">La richiesta non viene eseguita nuovamente se la risposta è stata avviata.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-127">The request isn't re-executed if the response has started.</span></span> <span data-ttu-id="ab1e7-128">Il codice generato dal modello esegue nuovamente la richiesta utilizzando il `/Error` percorso.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-128">The template generated code re-executes the request using the `/Error` path.</span></span>

<span data-ttu-id="ab1e7-129">Nell'esempio seguente, <xref:Microsoft.AspNetCore.Builder.ExceptionHandlerExtensions.UseExceptionHandler%2A> aggiunge il middleware di gestione delle eccezioni in ambienti non di sviluppo:</span><span class="sxs-lookup"><span data-stu-id="ab1e7-129">In the following example, <xref:Microsoft.AspNetCore.Builder.ExceptionHandlerExtensions.UseExceptionHandler%2A> adds the exception handling middleware in non-Development environments:</span></span>

[!code-csharp[](error-handling/samples/2.x/ErrorHandlingSample/Startup.cs?name=snippet_DevPageAndHandlerPage&highlight=5-9)]

<span data-ttu-id="ab1e7-130">Il Razor modello di app pagine fornisce una pagina di errore (*cshtml*) e una <xref:Microsoft.AspNetCore.Mvc.RazorPages.PageModel> classe ( `ErrorModel` ) nella cartella *pagine* .</span><span class="sxs-lookup"><span data-stu-id="ab1e7-130">The Razor Pages app template provides an Error page (*.cshtml*) and <xref:Microsoft.AspNetCore.Mvc.RazorPages.PageModel> class (`ErrorModel`) in the *Pages* folder.</span></span> <span data-ttu-id="ab1e7-131">Per un'app MVC, il modello di progetto include un `Error` metodo di azione e una visualizzazione degli errori per il controller Home.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-131">For an MVC app, the project template includes an `Error` action method and an Error view for the Home controller.</span></span>

<span data-ttu-id="ab1e7-132">Il middleware di gestione delle eccezioni esegue nuovamente la richiesta usando il metodo HTTP *originale* .</span><span class="sxs-lookup"><span data-stu-id="ab1e7-132">The exception handling middleware re-executes the request using the *original* HTTP method.</span></span> <span data-ttu-id="ab1e7-133">Se un endpoint del gestore errori è limitato a un set specifico di metodi HTTP, viene eseguito solo per i metodi HTTP.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-133">If an error handler endpoint is restricted to a specific set of HTTP methods, it runs only for those HTTP methods.</span></span> <span data-ttu-id="ab1e7-134">Ad esempio, un'azione del controller MVC che usa l' `[HttpGet]` attributo viene eseguita solo per le richieste Get.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-134">For example, an MVC controller action that uses the `[HttpGet]` attribute runs only for GET requests.</span></span> <span data-ttu-id="ab1e7-135">Per assicurarsi che *tutte* le richieste raggiungano la pagina di gestione degli errori personalizzata, non limitarle a un set specifico di metodi HTTP.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-135">To ensure that *all* requests reach the custom error handling page, don't restrict them to a specific set of HTTP methods.</span></span>

<span data-ttu-id="ab1e7-136">Per gestire le eccezioni in modo diverso in base al metodo HTTP originale:</span><span class="sxs-lookup"><span data-stu-id="ab1e7-136">To handle exceptions differently based on the original HTTP method:</span></span>

* <span data-ttu-id="ab1e7-137">Per le Razor pagine, creare più metodi del gestore.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-137">For Razor Pages, create multiple handler methods.</span></span> <span data-ttu-id="ab1e7-138">Utilizzare, ad esempio, `OnGet` per gestire le eccezioni Get e utilizzare `OnPost` per gestire le eccezioni post.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-138">For example, use `OnGet` to handle GET exceptions and use `OnPost` to handle POST exceptions.</span></span>
* <span data-ttu-id="ab1e7-139">Per MVC, applicare attributi di verbi HTTP a più azioni.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-139">For MVC, apply HTTP verb attributes to multiple actions.</span></span> <span data-ttu-id="ab1e7-140">Utilizzare, ad esempio, `[HttpGet]` per gestire le eccezioni Get e utilizzare `[HttpPost]` per gestire le eccezioni post.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-140">For example, use `[HttpGet]` to handle GET exceptions and use `[HttpPost]` to handle POST exceptions.</span></span>

<span data-ttu-id="ab1e7-141">Per consentire agli utenti non autenticati di visualizzare la pagina di gestione degli errori personalizzata, assicurarsi che supporti l'accesso anonimo.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-141">To allow unauthenticated users to view the custom error handling page, ensure that it supports anonymous access.</span></span>

### <a name="access-the-exception"></a><span data-ttu-id="ab1e7-142">Accedere all'eccezione</span><span class="sxs-lookup"><span data-stu-id="ab1e7-142">Access the exception</span></span>

<span data-ttu-id="ab1e7-143">Utilizzare <xref:Microsoft.AspNetCore.Diagnostics.IExceptionHandlerPathFeature> per accedere all'eccezione e al percorso della richiesta originale in un gestore degli errori.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-143">Use <xref:Microsoft.AspNetCore.Diagnostics.IExceptionHandlerPathFeature> to access the exception and the original request path in an error handler.</span></span> <span data-ttu-id="ab1e7-144">Il codice seguente aggiunge `ExceptionMessage` alle pagine predefinite */Error. cshtml. cs* generate dai modelli di ASP.NET Core:</span><span class="sxs-lookup"><span data-stu-id="ab1e7-144">The following code adds `ExceptionMessage` to the default *Pages/Error.cshtml.cs* generated by the ASP.NET Core templates:</span></span>

[!code-csharp[](error-handling/samples/5.x/ErrorHandlingSample/Pages/Error.cshtml.cs?name=snippet)]

> [!WARNING]
> <span data-ttu-id="ab1e7-145">**Non** fornire informazioni sugli errori sensibili ai client.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-145">Do **not** serve sensitive error information to clients.</span></span> <span data-ttu-id="ab1e7-146">Fornire informazioni sugli errori rappresenta un rischio di sicurezza.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-146">Serving errors is a security risk.</span></span>

<span data-ttu-id="ab1e7-147">Per testare l'eccezione nell' [app di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/error-handling/samples/5.x):</span><span class="sxs-lookup"><span data-stu-id="ab1e7-147">To test the exception in the [sample app](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/error-handling/samples/5.x):</span></span>

* <span data-ttu-id="ab1e7-148">Impostare l'ambiente sull'ambiente di produzione.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-148">Set the environment to production.</span></span>
* <span data-ttu-id="ab1e7-149">Rimuovere i commenti da `webBuilder.UseStartup<Startup>();` in *Program.cs*.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-149">Remove the comments from `webBuilder.UseStartup<Startup>();` in *Program.cs*.</span></span>
* <span data-ttu-id="ab1e7-150">Selezionare **attiva un'eccezione** nel Home page.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-150">Select **Trigger an exception** on the home page.</span></span>

## <a name="exception-handler-lambda"></a><span data-ttu-id="ab1e7-151">Espressione lambda per il gestore di eccezioni</span><span class="sxs-lookup"><span data-stu-id="ab1e7-151">Exception handler lambda</span></span>

<span data-ttu-id="ab1e7-152">Un'alternativa a una [pagina di gestione delle eccezioni personalizzata](#exception-handler-page) consiste nel fornire un'espressione lambda a <xref:Microsoft.AspNetCore.Builder.ExceptionHandlerExtensions.UseExceptionHandler%2A>.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-152">An alternative to a [custom exception handler page](#exception-handler-page) is to provide a lambda to <xref:Microsoft.AspNetCore.Builder.ExceptionHandlerExtensions.UseExceptionHandler%2A>.</span></span> <span data-ttu-id="ab1e7-153">L'uso di un'espressione lambda consente l'accesso all'errore prima di restituire la risposta.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-153">Using a lambda allows access to the error before returning the response.</span></span>

<span data-ttu-id="ab1e7-154">Il codice seguente usa un'espressione lambda per la gestione delle eccezioni:</span><span class="sxs-lookup"><span data-stu-id="ab1e7-154">The following code uses a lambda for exception handling:</span></span>

[!code-csharp[](error-handling/samples/5.x/ErrorHandlingSample/StartupLambda.cs?name=snippet)]

<!-- 
In the preceding code, `await context.Response.WriteAsync(new string(' ', 512));` is added so the Internet Explorer browser displays the error message rather than an IE error message. For more information, see [this GitHub issue](https://github.com/dotnet/AspNetCore.Docs/issues/16144).
-->

> [!WARNING]
> <span data-ttu-id="ab1e7-155">**Non** fornire informazioni sugli errori sensibili da <xref:Microsoft.AspNetCore.Diagnostics.IExceptionHandlerFeature> o <xref:Microsoft.AspNetCore.Diagnostics.IExceptionHandlerPathFeature> ai client.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-155">Do **not** serve sensitive error information from <xref:Microsoft.AspNetCore.Diagnostics.IExceptionHandlerFeature> or <xref:Microsoft.AspNetCore.Diagnostics.IExceptionHandlerPathFeature> to clients.</span></span> <span data-ttu-id="ab1e7-156">Fornire informazioni sugli errori rappresenta un rischio di sicurezza.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-156">Serving errors is a security risk.</span></span>

<span data-ttu-id="ab1e7-157">Per testare l'espressione lambda di gestione delle eccezioni nell' [app di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/error-handling/samples/5.x):</span><span class="sxs-lookup"><span data-stu-id="ab1e7-157">To test the exception handling lambda in the [sample app](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/error-handling/samples/5.x):</span></span>

* <span data-ttu-id="ab1e7-158">Impostare l'ambiente sull'ambiente di produzione.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-158">Set the environment to production.</span></span>
* <span data-ttu-id="ab1e7-159">Rimuovere i commenti da `webBuilder.UseStartup<StartupLambda>();` in *Program.cs*.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-159">Remove the comments from `webBuilder.UseStartup<StartupLambda>();` in *Program.cs*.</span></span>
* <span data-ttu-id="ab1e7-160">Selezionare **attiva un'eccezione** nel Home page.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-160">Select **Trigger an exception** on the home page.</span></span>

## <a name="usestatuscodepages"></a><span data-ttu-id="ab1e7-161">UseStatusCodePages</span><span class="sxs-lookup"><span data-stu-id="ab1e7-161">UseStatusCodePages</span></span>

<span data-ttu-id="ab1e7-162">Per impostazione predefinita, un'app ASP.NET Core non fornisce una tabella codici di stato per i codici di stato di errore HTTP, ad esempio *404-non trovato*.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-162">By default, an ASP.NET Core app doesn't provide a status code page for HTTP error status codes, such as *404 - Not Found*.</span></span> <span data-ttu-id="ab1e7-163">Quando l'app rileva un codice di stato di errore HTTP 400-599 che non ha un corpo, restituisce il codice di stato e un corpo della risposta vuoto.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-163">When the app encounters an HTTP 400-599 error status code that doesn't have a body, it returns the status code and an empty response body.</span></span> <span data-ttu-id="ab1e7-164">Per specificare le tabelle codici di stato, usare il middleware tabelle codici di stato.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-164">To provide status code pages, use the status code pages middleware.</span></span> <span data-ttu-id="ab1e7-165">Per abilitare i gestori di solo testo predefiniti per i codici di stato di errore comuni, chiamare <xref:Microsoft.AspNetCore.Builder.StatusCodePagesExtensions.UseStatusCodePages%2A> nel metodo `Startup.Configure`:</span><span class="sxs-lookup"><span data-stu-id="ab1e7-165">To enable default text-only handlers for common error status codes, call <xref:Microsoft.AspNetCore.Builder.StatusCodePagesExtensions.UseStatusCodePages%2A> in the `Startup.Configure` method:</span></span>

[!code-csharp[](error-handling/samples/5.x/ErrorHandlingSample/StartupUseStatusCodePages.cs?name=snippet&highlight=13)]

<span data-ttu-id="ab1e7-166">Chiamare `UseStatusCodePages` prima del middleware di gestione delle richieste.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-166">Call `UseStatusCodePages` before request handling middleware.</span></span> <span data-ttu-id="ab1e7-167">Ad esempio, chiamare `UseStatusCodePages` prima del middleware del file statico e del middleware degli endpoint.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-167">For example, call `UseStatusCodePages` before the Static File Middleware and the Endpoints Middleware.</span></span>

<span data-ttu-id="ab1e7-168">Quando `UseStatusCodePages` non viene utilizzato, la navigazione a un URL senza un endpoint restituisce un messaggio di errore dipendente dal browser che indica che l'endpoint non è stato trovato.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-168">When `UseStatusCodePages` isn't used, navigating to a URL without an endpoint returns a browser dependent error message indicating the endpoint can't be found.</span></span> <span data-ttu-id="ab1e7-169">Ad esempio, passando a `Home/Privacy2` .</span><span class="sxs-lookup"><span data-stu-id="ab1e7-169">For example, navigating to `Home/Privacy2`.</span></span> <span data-ttu-id="ab1e7-170">Quando `UseStatusCodePages` viene chiamato il metodo, il browser restituisce:</span><span class="sxs-lookup"><span data-stu-id="ab1e7-170">When `UseStatusCodePages` is called, the browser returns:</span></span>

```html
Status Code: 404; Not Found
```

<span data-ttu-id="ab1e7-171">`UseStatusCodePages` non viene in genere utilizzato nell'ambiente di produzione perché restituisce un messaggio che non è utile per gli utenti.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-171">`UseStatusCodePages` isn't typically used in production because it returns a message that isn't useful to users.</span></span>

<span data-ttu-id="ab1e7-172">Per eseguire `UseStatusCodePages` il test nell' [app di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/error-handling/samples/5.x):</span><span class="sxs-lookup"><span data-stu-id="ab1e7-172">To test `UseStatusCodePages` in the [sample app](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/error-handling/samples/5.x):</span></span>

* <span data-ttu-id="ab1e7-173">Impostare l'ambiente sull'ambiente di produzione.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-173">Set the environment to production.</span></span>
* <span data-ttu-id="ab1e7-174">Rimuovere i commenti da `webBuilder.UseStartup<StartupUseStatusCodePages>();` in *Program.cs*.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-174">Remove the comments from `webBuilder.UseStartup<StartupUseStatusCodePages>();` in *Program.cs*.</span></span>
* <span data-ttu-id="ab1e7-175">Selezionare i collegamenti nel home page home page.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-175">Select the links on the home page on the home page.</span></span>

> [!NOTE]
> <span data-ttu-id="ab1e7-176">Il middleware delle tabelle codici di stato **non** rileva le eccezioni.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-176">The status code pages middleware does **not** catch exceptions.</span></span> <span data-ttu-id="ab1e7-177">Per fornire una pagina di gestione degli errori personalizzata, utilizzare la [pagina gestore eccezioni](#exception-handler-page).</span><span class="sxs-lookup"><span data-stu-id="ab1e7-177">To provide a custom error handling page, use the [exception handler page](#exception-handler-page).</span></span>

### <a name="usestatuscodepages-with-format-string"></a><span data-ttu-id="ab1e7-178">UseStatusCodePages con stringa di formato</span><span class="sxs-lookup"><span data-stu-id="ab1e7-178">UseStatusCodePages with format string</span></span>

<span data-ttu-id="ab1e7-179">Per personalizzare il tipo di contenuto della risposta e il testo, usare l'overload di <xref:Microsoft.AspNetCore.Builder.StatusCodePagesExtensions.UseStatusCodePages%2A> che accetta un tipo di contenuto e una stringa di formato:</span><span class="sxs-lookup"><span data-stu-id="ab1e7-179">To customize the response content type and text, use the overload of <xref:Microsoft.AspNetCore.Builder.StatusCodePagesExtensions.UseStatusCodePages%2A> that takes a content type and format string:</span></span>

[!code-csharp[](error-handling/samples/5.x/ErrorHandlingSample/StartupFormat.cs?name=snippet&highlight=13-14)]

<span data-ttu-id="ab1e7-180">Nel codice precedente, `{0}` è un segnaposto per il codice di errore.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-180">In the preceding code, `{0}` is a placeholder for the error code.</span></span>

<span data-ttu-id="ab1e7-181">`UseStatusCodePages` con una stringa di formato non viene in genere usata nell'ambiente di produzione perché restituisce un messaggio che non è utile per gli utenti.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-181">`UseStatusCodePages` with a format string isn't typically used in production because it returns a message that isn't useful to users.</span></span>

<span data-ttu-id="ab1e7-182">Per eseguire `UseStatusCodePages` il test nell' [app di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/error-handling/samples/5.x), rimuovere i commenti da `webBuilder.UseStartup<StartupFormat>();` in *Program.cs*.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-182">To test `UseStatusCodePages` in the [sample app](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/error-handling/samples/5.x), remove the comments from `webBuilder.UseStartup<StartupFormat>();` in *Program.cs*.</span></span>

### <a name="usestatuscodepages-with-lambda"></a><span data-ttu-id="ab1e7-183">UseStatusCodePages con espressione lambda</span><span class="sxs-lookup"><span data-stu-id="ab1e7-183">UseStatusCodePages with lambda</span></span>

<span data-ttu-id="ab1e7-184">Per specificare la gestione degli errori personalizzata e il codice per la scrittura delle risposte, usare l'overload di <xref:Microsoft.AspNetCore.Builder.StatusCodePagesExtensions.UseStatusCodePages%2A> che accetta un'espressione lambda:</span><span class="sxs-lookup"><span data-stu-id="ab1e7-184">To specify custom error-handling and response-writing code, use the overload of <xref:Microsoft.AspNetCore.Builder.StatusCodePagesExtensions.UseStatusCodePages%2A> that takes a lambda expression:</span></span>

[!code-csharp[](error-handling/samples/5.x/ErrorHandlingSample/StartupStatusLambda.cs?name=snippet&highlight=13-20)]

<span data-ttu-id="ab1e7-185">`UseStatusCodePages` con un'espressione lambda non viene in genere usata nell'ambiente di produzione perché restituisce un messaggio che non è utile per gli utenti.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-185">`UseStatusCodePages` with a lambda isn't typically used in production because it returns a message that isn't useful to users.</span></span>

<span data-ttu-id="ab1e7-186">Per eseguire `UseStatusCodePages` il test nell' [app di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/error-handling/samples/5.x), rimuovere i commenti da `webBuilder.UseStartup<StartupStatusLambda>();` in *Program.cs*.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-186">To test `UseStatusCodePages` in the [sample app](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/error-handling/samples/5.x), remove the comments from `webBuilder.UseStartup<StartupStatusLambda>();` in *Program.cs*.</span></span>

### <a name="usestatuscodepageswithredirects"></a><span data-ttu-id="ab1e7-187">UseStatusCodePagesWithRedirects</span><span class="sxs-lookup"><span data-stu-id="ab1e7-187">UseStatusCodePagesWithRedirects</span></span>

<span data-ttu-id="ab1e7-188">Metodo di estensione <xref:Microsoft.AspNetCore.Builder.StatusCodePagesExtensions.UseStatusCodePagesWithRedirects%2A>:</span><span class="sxs-lookup"><span data-stu-id="ab1e7-188">The <xref:Microsoft.AspNetCore.Builder.StatusCodePagesExtensions.UseStatusCodePagesWithRedirects%2A> extension method:</span></span>

* <span data-ttu-id="ab1e7-189">Invia un codice di stato [302 - Trovato](https://developer.mozilla.org/docs/Web/HTTP/Status/302) al client.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-189">Sends a [302 - Found](https://developer.mozilla.org/docs/Web/HTTP/Status/302) status code to the client.</span></span>
* <span data-ttu-id="ab1e7-190">Reindirizza il client all'endpoint di gestione degli errori fornito nel modello di URL.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-190">Redirects the client to the error handling endpoint provided in the URL template.</span></span> <span data-ttu-id="ab1e7-191">Nell'endpoint di gestione degli errori vengono in genere visualizzate informazioni sull'errore e viene restituito il protocollo HTTP 200.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-191">The error handling endpoint typically displays error information and returns HTTP 200.</span></span>

[!code-csharp[](error-handling/samples/5.x/ErrorHandlingSample/StartupSCredirect.cs?name=snippet&highlight=13)]

<span data-ttu-id="ab1e7-192">Il modello di URL può includere un `{0}` segnaposto per il codice di stato, come illustrato nel codice precedente.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-192">The URL template can include a `{0}` placeholder for the status code, as shown in the preceding code.</span></span> <span data-ttu-id="ab1e7-193">Se il modello di URL inizia con `~` (tilde), `~` viene sostituito dall'app `PathBase` .</span><span class="sxs-lookup"><span data-stu-id="ab1e7-193">If the URL template starts with `~` (tilde), the `~` is replaced by the app's `PathBase`.</span></span> <span data-ttu-id="ab1e7-194">Quando si specifica un endpoint nell'app, creare una visualizzazione o una Razor pagina MVC per l'endpoint.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-194">When specifying an endpoint in the app, create an MVC view or Razor page for the endpoint.</span></span> <span data-ttu-id="ab1e7-195">Per un Razor esempio di pagine, vedere [pages/MyStatusCode. cshtml](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/error-handling/samples/5.x/ErrorHandlingSample/Pages) nell' [app di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/error-handling/samples/5.x).</span><span class="sxs-lookup"><span data-stu-id="ab1e7-195">For a Razor Pages example, see [Pages/MyStatusCode.cshtml](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/error-handling/samples/5.x/ErrorHandlingSample/Pages) in the [sample app](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/error-handling/samples/5.x).</span></span>

<span data-ttu-id="ab1e7-196">Questo metodo viene usato comunemente quando l'app:</span><span class="sxs-lookup"><span data-stu-id="ab1e7-196">This method is commonly used when the app:</span></span>

* <span data-ttu-id="ab1e7-197">Deve reindirizzare il client a un endpoint diverso, in genere nei casi in cui un'altra app elabora l'errore.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-197">Should redirect the client to a different endpoint, usually in cases where a different app processes the error.</span></span> <span data-ttu-id="ab1e7-198">Per le app Web, la barra degli indirizzi del browser del client riflette l'endpoint reindirizzato.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-198">For web apps, the client's browser address bar reflects the redirected endpoint.</span></span>
* <span data-ttu-id="ab1e7-199">Non deve conservare e restituire il codice di stato originale con la risposta di reindirizzamento iniziale.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-199">Shouldn't preserve and return the original status code with the initial redirect response.</span></span>

<span data-ttu-id="ab1e7-200">Per eseguire `UseStatusCodePages` il test nell' [app di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/error-handling/samples/5.x), rimuovere i commenti da `webBuilder.UseStartup<StartupSCredirect>();` in *Program.cs*.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-200">To test `UseStatusCodePages` in the [sample app](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/error-handling/samples/5.x), remove the comments from `webBuilder.UseStartup<StartupSCredirect>();` in *Program.cs*.</span></span>

### <a name="usestatuscodepageswithreexecute"></a><span data-ttu-id="ab1e7-201">UseStatusCodePagesWithReExecute</span><span class="sxs-lookup"><span data-stu-id="ab1e7-201">UseStatusCodePagesWithReExecute</span></span>

<span data-ttu-id="ab1e7-202">Metodo di estensione <xref:Microsoft.AspNetCore.Builder.StatusCodePagesExtensions.UseStatusCodePagesWithReExecute%2A>:</span><span class="sxs-lookup"><span data-stu-id="ab1e7-202">The <xref:Microsoft.AspNetCore.Builder.StatusCodePagesExtensions.UseStatusCodePagesWithReExecute%2A> extension method:</span></span>

* <span data-ttu-id="ab1e7-203">Restituisce il codice di stato originale al client.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-203">Returns the original status code to the client.</span></span>
* <span data-ttu-id="ab1e7-204">Genera il corpo della risposta eseguendo nuovamente la pipeline delle richieste con un percorso alternativo.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-204">Generates the response body by re-executing the request pipeline using an alternate path.</span></span>

[!code-csharp[](error-handling/samples/5.x/ErrorHandlingSample/StartupSCreX.cs?name=snippet&highlight=13)]

<span data-ttu-id="ab1e7-205">Se viene specificato un endpoint all'interno dell'app, creare una visualizzazione o una Razor pagina MVC per l'endpoint.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-205">If an endpoint within the app is specified, create an MVC view or Razor page for the endpoint.</span></span> <span data-ttu-id="ab1e7-206">Assicurarsi `UseStatusCodePagesWithReExecute` che venga inserito prima `UseRouting` di in modo che sia possibile reindirizzare la richiesta alla pagina stato.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-206">Ensure `UseStatusCodePagesWithReExecute` is placed before `UseRouting` so the request can be rerouted to the status page.</span></span> <span data-ttu-id="ab1e7-207">Per un Razor esempio di pagine, vedere [pages/MyStatusCode2. cshtml](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/error-handling/samples/5.x/ErrorHandlingSample/Pages) nell' [app di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/error-handling/samples/5.x).</span><span class="sxs-lookup"><span data-stu-id="ab1e7-207">For a Razor Pages example, see [Pages/MyStatusCode2.cshtml](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/error-handling/samples/5.x/ErrorHandlingSample/Pages) in the [sample app](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/error-handling/samples/5.x).</span></span>

<span data-ttu-id="ab1e7-208">Questo metodo viene usato comunemente quando l'app deve:</span><span class="sxs-lookup"><span data-stu-id="ab1e7-208">This method is commonly used when the app should:</span></span>

* <span data-ttu-id="ab1e7-209">Elaborare la richiesta senza il reindirizzamento a un endpoint diverso.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-209">Process the request without redirecting to a different endpoint.</span></span> <span data-ttu-id="ab1e7-210">Per le app Web, la barra degli indirizzi del browser del client riflette l'endpoint richiesto in origine.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-210">For web apps, the client's browser address bar reflects the originally requested endpoint.</span></span>
* <span data-ttu-id="ab1e7-211">Conservare e restituire il codice di stato originale con la risposta.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-211">Preserve and return the original status code with the response.</span></span>

<span data-ttu-id="ab1e7-212">I modelli URL e stringa di query possono includere un segnaposto `{0}` per il codice di stato.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-212">The URL and query string templates may include a placeholder `{0}` for the status code.</span></span> <span data-ttu-id="ab1e7-213">Il modello di URL deve iniziare con `/` .</span><span class="sxs-lookup"><span data-stu-id="ab1e7-213">The URL template must start with `/`.</span></span>

<!-- Review: removing this. The sample code doesn't use @page "{code?}"
If you want that, it should be @page "{code:int?}"
but that's not required. Original text follows:

When using a placeholder in the path, confirm that the endpoint can process the path segment. For example, a Razor Page for errors should accept the optional path segment value with the `@page` directive:

```cshtml
@page "{code?}"
```
-->

<span data-ttu-id="ab1e7-214">L'endpoint che elabora l'errore può ottenere l'URL originale che ha generato l'errore, come illustrato nell'esempio seguente:</span><span class="sxs-lookup"><span data-stu-id="ab1e7-214">The endpoint that processes the error can get the original URL that generated the error, as shown in the following example:</span></span>

[!code-csharp[](error-handling/samples/5.x/ErrorHandlingSample/Pages/MyStatusCode2.cshtml.cs?name=snippet)]

<span data-ttu-id="ab1e7-215">Per un Razor esempio di pagine, vedere [pages/MyStatusCode2. cshtml](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/error-handling/samples/5.x/ErrorHandlingSample/Pages) nell' [app di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/error-handling/samples/5.x).</span><span class="sxs-lookup"><span data-stu-id="ab1e7-215">For a Razor Pages example, see [Pages/MyStatusCode2.cshtml](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/error-handling/samples/5.x/ErrorHandlingSample/Pages) in the [sample app](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/error-handling/samples/5.x).</span></span>

<span data-ttu-id="ab1e7-216">Per eseguire `UseStatusCodePages` il test nell' [app di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/error-handling/samples/5.x), rimuovere i commenti da `webBuilder.UseStartup<StartupSCreX>();` in *Program.cs*.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-216">To test `UseStatusCodePages` in the [sample app](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/error-handling/samples/5.x), remove the comments from `webBuilder.UseStartup<StartupSCreX>();` in *Program.cs*.</span></span>

## <a name="disable-status-code-pages"></a><span data-ttu-id="ab1e7-217">Disabilitare le tabelle codici di stato</span><span class="sxs-lookup"><span data-stu-id="ab1e7-217">Disable status code pages</span></span>

<span data-ttu-id="ab1e7-218">Per disabilitare le tabelle codici di stato per un controller MVC o un metodo di azione, usare l'attributo [[SkipStatusCodePages]](xref:Microsoft.AspNetCore.Mvc.SkipStatusCodePagesAttribute).</span><span class="sxs-lookup"><span data-stu-id="ab1e7-218">To disable status code pages for an MVC controller or action method, use the [[SkipStatusCodePages]](xref:Microsoft.AspNetCore.Mvc.SkipStatusCodePagesAttribute) attribute.</span></span>

<span data-ttu-id="ab1e7-219">Per disabilitare le tabelle codici di stato per richieste specifiche in un Razor metodo del gestore di pagine o in un controller MVC, usare <xref:Microsoft.AspNetCore.Diagnostics.IStatusCodePagesFeature> :</span><span class="sxs-lookup"><span data-stu-id="ab1e7-219">To disable status code pages for specific requests in a Razor Pages handler method or in an MVC controller, use <xref:Microsoft.AspNetCore.Diagnostics.IStatusCodePagesFeature>:</span></span>

[!code-csharp[](error-handling/samples/5.x/ErrorHandlingSample/Pages/Privacy.cshtml.cs?name=snippet)]

## <a name="exception-handling-code"></a><span data-ttu-id="ab1e7-220">Codice di gestione delle eccezioni</span><span class="sxs-lookup"><span data-stu-id="ab1e7-220">Exception-handling code</span></span>

<span data-ttu-id="ab1e7-221">Anche il codice nelle pagine di gestione delle eccezioni può generare eccezioni.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-221">Code in exception handling pages can also throw exceptions.</span></span> <span data-ttu-id="ab1e7-222">Le pagine di errore di produzione devono essere testate accuratamente e prestare particolare attenzione per evitare di generare eccezioni proprie.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-222">Production error pages should be tested thoroughly and take extra care to avoid throwing exceptions of their own.</span></span>
<!-- Review: original, which is not realistic 
 > It's often a good idea for production error pages to consist of purely static content.

 comments: - after you catch the exception, you need code to log the details and perhaps dynamically create a string with an error message. 
-->

### <a name="response-headers"></a><span data-ttu-id="ab1e7-223">Intestazioni di risposta</span><span class="sxs-lookup"><span data-stu-id="ab1e7-223">Response headers</span></span>

<span data-ttu-id="ab1e7-224">Dopo l'invio delle intestazioni per una risposta:</span><span class="sxs-lookup"><span data-stu-id="ab1e7-224">Once the headers for a response are sent:</span></span>

* <span data-ttu-id="ab1e7-225">L'app non può modificare il codice di stato della risposta.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-225">The app can't change the response's status code.</span></span>
* <span data-ttu-id="ab1e7-226">Non è possibile eseguire pagine o gestori delle eccezioni.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-226">Any exception pages or handlers can't run.</span></span> <span data-ttu-id="ab1e7-227">La risposta deve essere completata o la connessione deve essere interrotta.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-227">The response must be completed or the connection aborted.</span></span>

## <a name="server-exception-handling"></a><span data-ttu-id="ab1e7-228">Gestione delle eccezioni del server</span><span class="sxs-lookup"><span data-stu-id="ab1e7-228">Server exception handling</span></span>

<span data-ttu-id="ab1e7-229">Oltre alla logica di gestione delle eccezioni in un'app, l' [implementazione del server http](xref:fundamentals/servers/index) può gestire alcune eccezioni.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-229">In addition to the exception handling logic in an app, the [HTTP server implementation](xref:fundamentals/servers/index) can handle some exceptions.</span></span> <span data-ttu-id="ab1e7-230">Se il server rileva un'eccezione prima dell'invio delle intestazioni di risposta, il server invia una `500 - Internal Server Error` risposta senza il corpo della risposta.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-230">If the server catches an exception before response headers are sent, the server sends a `500 - Internal Server Error` response without a response body.</span></span> <span data-ttu-id="ab1e7-231">Se il server rileva un'eccezione dopo l'invio delle intestazioni di risposta, il server chiude la connessione.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-231">If the server catches an exception after response headers are sent, the server closes the connection.</span></span> <span data-ttu-id="ab1e7-232">Le richieste che non vengono gestite dall'app vengono gestite dal server.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-232">Requests that aren't handled by the app are handled by the server.</span></span> <span data-ttu-id="ab1e7-233">Qualsiasi eccezione che si verifica quando il server sta gestendo la richiesta viene gestita dalla gestione delle eccezioni del server.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-233">Any exception that occurs when the server is handling the request is handled by the server's exception handling.</span></span> <span data-ttu-id="ab1e7-234">Eventuali pagine di errore personalizzate dell'app, middleware di gestione delle eccezioni e filtri non hanno effetto su questo comportamento.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-234">The app's custom error pages, exception handling middleware, and filters don't affect this behavior.</span></span>

## <a name="startup-exception-handling"></a><span data-ttu-id="ab1e7-235">Gestione delle eccezioni durante l'avvio</span><span class="sxs-lookup"><span data-stu-id="ab1e7-235">Startup exception handling</span></span>

<span data-ttu-id="ab1e7-236">Solo il livello di hosting può gestire le eccezioni che si verificano durante l'avvio dell'app.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-236">Only the hosting layer can handle exceptions that take place during app startup.</span></span> <span data-ttu-id="ab1e7-237">L'host può essere configurato per [acquisire gli errori di avvio](xref:fundamentals/host/web-host#capture-startup-errors) e [acquisire errori dettagliati](xref:fundamentals/host/web-host#detailed-errors).</span><span class="sxs-lookup"><span data-stu-id="ab1e7-237">The host can be configured to [capture startup errors](xref:fundamentals/host/web-host#capture-startup-errors) and [capture detailed errors](xref:fundamentals/host/web-host#detailed-errors).</span></span>

<span data-ttu-id="ab1e7-238">Il livello di hosting può visualizzare una pagina di errore per un errore di avvio acquisito solo se l'errore si verifica dopo l'associazione indirizzo host/porta.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-238">The hosting layer can show an error page for a captured startup error only if the error occurs after host address/port binding.</span></span> <span data-ttu-id="ab1e7-239">Se si verifica un errore di associazione:</span><span class="sxs-lookup"><span data-stu-id="ab1e7-239">If binding fails:</span></span>

* <span data-ttu-id="ab1e7-240">Il livello di hosting registra un'eccezione critica.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-240">The hosting layer logs a critical exception.</span></span>
* <span data-ttu-id="ab1e7-241">Si verifica un arresto anomalo del processo di dotnet.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-241">The dotnet process crashes.</span></span>
* <span data-ttu-id="ab1e7-242">Non viene visualizzata alcuna pagina di errore quando il server HTTP è [Kestrel](xref:fundamentals/servers/kestrel).</span><span class="sxs-lookup"><span data-stu-id="ab1e7-242">No error page is displayed when the HTTP server is [Kestrel](xref:fundamentals/servers/kestrel).</span></span>

<span data-ttu-id="ab1e7-243">Se durante l'esecuzione in [IIS](/iis) (o servizio app di Azure) o in [IIS Express](/iis/extensions/introduction-to-iis-express/iis-express-overview) il processo non può essere avviato, viene restituito l'errore *502.5 Errore del processo* dal [modulo ASP.NET Core](xref:host-and-deploy/aspnet-core-module).</span><span class="sxs-lookup"><span data-stu-id="ab1e7-243">When running on [IIS](/iis) (or Azure App Service) or [IIS Express](/iis/extensions/introduction-to-iis-express/iis-express-overview), a *502.5 - Process Failure* is returned by the [ASP.NET Core Module](xref:host-and-deploy/aspnet-core-module) if the process can't start.</span></span> <span data-ttu-id="ab1e7-244">Per altre informazioni, vedere <xref:test/troubleshoot-azure-iis>.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-244">For more information, see <xref:test/troubleshoot-azure-iis>.</span></span>

## <a name="database-error-page"></a><span data-ttu-id="ab1e7-245">Pagina di errore di database</span><span class="sxs-lookup"><span data-stu-id="ab1e7-245">Database error page</span></span>

<span data-ttu-id="ab1e7-246">Il filtro eccezioni pagina sviluppatore database `AddDatabaseDeveloperPageExceptionFilter` acquisisce le eccezioni correlate al database che possono essere risolte utilizzando Entity Framework Core migrazioni.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-246">The Database developer page exception filter `AddDatabaseDeveloperPageExceptionFilter` captures database-related exceptions that can be resolved by using Entity Framework Core migrations.</span></span> <span data-ttu-id="ab1e7-247">Quando si verificano queste eccezioni, viene generata una risposta HTML con i dettagli delle azioni possibili per risolvere il problema.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-247">When these exceptions occur, an HTML response is generated with details of possible actions to resolve the issue.</span></span> <span data-ttu-id="ab1e7-248">Questa pagina è abilitata solo nell'ambiente di sviluppo.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-248">This page is enabled only in the Development environment.</span></span> <span data-ttu-id="ab1e7-249">Il codice seguente è stato generato dal ASP.NET Core Razor modelli di pagine quando sono stati specificati singoli account utente:</span><span class="sxs-lookup"><span data-stu-id="ab1e7-249">The following code was generated by the ASP.NET Core Razor Pages templates when individual user accounts were specified:</span></span>

[!code-csharp[](error-handling/samples/5.x/StartupDBexFilter.cs?name=snippet&highlight=6)]

## <a name="exception-filters"></a><span data-ttu-id="ab1e7-250">Filtri eccezioni</span><span class="sxs-lookup"><span data-stu-id="ab1e7-250">Exception filters</span></span>

<span data-ttu-id="ab1e7-251">Nelle app MVC i filtri delle eccezioni possono essere configurati a livello globale o per ogni controller o azione singolarmente.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-251">In MVC apps, exception filters can be configured globally or on a per-controller or per-action basis.</span></span> <span data-ttu-id="ab1e7-252">Nelle Razor app Pages possono essere configurate a livello globale o per modello di pagina.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-252">In Razor Pages apps, they can be configured globally or per page model.</span></span> <span data-ttu-id="ab1e7-253">Questi filtri gestiscono le eventuali eccezioni non gestite che si verificano durante l'esecuzione di un'azione del controller o di un altro filtro.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-253">These filters handle any unhandled exceptions that occur during the execution of a controller action or another filter.</span></span> <span data-ttu-id="ab1e7-254">Per altre informazioni, vedere <xref:mvc/controllers/filters#exception-filters>.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-254">For more information, see <xref:mvc/controllers/filters#exception-filters>.</span></span>

<span data-ttu-id="ab1e7-255">I filtri eccezioni sono utili per intercettare le eccezioni che si verificano all'interno di azioni MVC, ma non sono flessibili come il [middleware di gestione delle eccezioni](https://github.com/dotnet/aspnetcore/blob/master/src/Middleware/Diagnostics/src/ExceptionHandler/ExceptionHandlerMiddleware.cs)incorporato, `UseExceptionHandler` .</span><span class="sxs-lookup"><span data-stu-id="ab1e7-255">Exception filters are useful for trapping exceptions that occur within MVC actions, but they're not as flexible as the built-in [exception handling middleware](https://github.com/dotnet/aspnetcore/blob/master/src/Middleware/Diagnostics/src/ExceptionHandler/ExceptionHandlerMiddleware.cs), `UseExceptionHandler`.</span></span> <span data-ttu-id="ab1e7-256">È consigliabile usare `UseExceptionHandler` , a meno che non sia necessario eseguire la gestione degli errori in modo diverso in base all'azione MVC scelta.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-256">We recommend using `UseExceptionHandler`, unless you need to perform error handling differently based on which MVC action is chosen.</span></span>

[!code-csharp[](error-handling/samples/5.x/ErrorHandlingSample/Startup.cs?name=snippet&highlight=9)]

## <a name="model-state-errors"></a><span data-ttu-id="ab1e7-257">Errori di stato del modello</span><span class="sxs-lookup"><span data-stu-id="ab1e7-257">Model state errors</span></span>

<span data-ttu-id="ab1e7-258">Per informazioni su come gestire gli errori dello stato del modello, vedere [Associazione di modelli](xref:mvc/models/model-binding) e [Convalida del modello](xref:mvc/models/validation).</span><span class="sxs-lookup"><span data-stu-id="ab1e7-258">For information about how to handle model state errors, see [Model binding](xref:mvc/models/model-binding) and [Model validation](xref:mvc/models/validation).</span></span>

## <a name="additional-resources"></a><span data-ttu-id="ab1e7-259">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="ab1e7-259">Additional resources</span></span>

* <xref:test/troubleshoot-azure-iis>
* <xref:host-and-deploy/azure-iis-errors-reference>

::: moniker-end

::: moniker range="< aspnetcore-5.0"

<span data-ttu-id="ab1e7-260">Di  [Tom Dykstra](https://github.com/tdykstra/)e [Steve Smith](https://ardalis.com/)</span><span class="sxs-lookup"><span data-stu-id="ab1e7-260">By  [Tom Dykstra](https://github.com/tdykstra/), and [Steve Smith](https://ardalis.com/)</span></span>

<span data-ttu-id="ab1e7-261">Questo articolo illustra gli approcci comuni per la gestione degli errori nelle app Web ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-261">This article covers common approaches to handling errors in ASP.NET Core web apps.</span></span> <span data-ttu-id="ab1e7-262">Vedere <xref:web-api/handle-errors> per le API Web.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-262">See <xref:web-api/handle-errors> for web APIs.</span></span>

<span data-ttu-id="ab1e7-263">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/error-handling/samples).</span><span class="sxs-lookup"><span data-stu-id="ab1e7-263">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/error-handling/samples).</span></span> <span data-ttu-id="ab1e7-264">([Come scaricare un esempio](xref:index#how-to-download-a-sample).)</span><span class="sxs-lookup"><span data-stu-id="ab1e7-264">([How to download](xref:index#how-to-download-a-sample).)</span></span>

## <a name="developer-exception-page"></a><span data-ttu-id="ab1e7-265">Pagina delle eccezioni per gli sviluppatori</span><span class="sxs-lookup"><span data-stu-id="ab1e7-265">Developer Exception Page</span></span>

<span data-ttu-id="ab1e7-266">In *Pagina delle eccezioni per gli sviluppatori* sono disponibili informazioni dettagliate sulle eccezioni delle richieste.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-266">The *Developer Exception Page* displays detailed information about request exceptions.</span></span> <span data-ttu-id="ab1e7-267">I modelli di ASP.NET Core generano il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="ab1e7-267">The ASP.NET Core templates generate the following code:</span></span>

[!code-csharp[](error-handling/samples/2.x/ErrorHandlingSample/Startup.cs?name=snippet_DevPageAndHandlerPage&highlight=1-4)]

<span data-ttu-id="ab1e7-268">Il codice precedente Abilita la pagina delle eccezioni per gli sviluppatori quando l'app è in esecuzione nell' [ambiente di sviluppo](xref:fundamentals/environments).</span><span class="sxs-lookup"><span data-stu-id="ab1e7-268">The preceding code enables the developer exception page when the app is running in the [Development environment](xref:fundamentals/environments).</span></span>

<span data-ttu-id="ab1e7-269">I modelli si posizionano <xref:Microsoft.AspNetCore.Builder.DeveloperExceptionPageExtensions.UseDeveloperExceptionPage%2A> prima di qualsiasi middleware, in modo che le eccezioni vengano rilevate nel middleware che segue.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-269">The templates place <xref:Microsoft.AspNetCore.Builder.DeveloperExceptionPageExtensions.UseDeveloperExceptionPage%2A> before any middleware so exceptions are caught in the middleware that follows.</span></span>

<span data-ttu-id="ab1e7-270">Il codice precedente Abilita la pagina delle eccezioni **per gli sviluppatori solo quando l'app è in esecuzione nell'ambiente di sviluppo**.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-270">The preceding code enables the Developer Exception Page **only when the app is running in the Development environment**.</span></span> <span data-ttu-id="ab1e7-271">Le informazioni dettagliate sulle eccezioni non devono essere visualizzate pubblicamente quando l'app viene eseguita nell'ambiente di produzione.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-271">Detailed exception information should not be displayed publicly when the app runs in production.</span></span> <span data-ttu-id="ab1e7-272">Per altre informazioni sulla configurazione di ambienti, vedere <xref:fundamentals/environments>.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-272">For more information on configuring environments, see <xref:fundamentals/environments>.</span></span>

<span data-ttu-id="ab1e7-273">La pagina delle eccezioni per sviluppatori include le informazioni seguenti sull'eccezione e la richiesta:</span><span class="sxs-lookup"><span data-stu-id="ab1e7-273">The Developer Exception Page includes the following information about the exception and the request:</span></span>

* <span data-ttu-id="ab1e7-274">Analisi dello stack</span><span class="sxs-lookup"><span data-stu-id="ab1e7-274">Stack trace</span></span>
* <span data-ttu-id="ab1e7-275">Parametri della stringa di query se presenti</span><span class="sxs-lookup"><span data-stu-id="ab1e7-275">Query string parameters if any</span></span>
* <span data-ttu-id="ab1e7-276">Cookies se presente</span><span class="sxs-lookup"><span data-stu-id="ab1e7-276">Cookies if any</span></span>
* <span data-ttu-id="ab1e7-277">Intestazioni</span><span class="sxs-lookup"><span data-stu-id="ab1e7-277">Headers</span></span>

## <a name="exception-handler-page"></a><span data-ttu-id="ab1e7-278">Pagina del gestore di eccezioni</span><span class="sxs-lookup"><span data-stu-id="ab1e7-278">Exception handler page</span></span>

<span data-ttu-id="ab1e7-279">Per configurare una pagina di gestione degli errori personalizzata per l'ambiente di produzione, usare il middleware di gestione delle eccezioni.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-279">To configure a custom error handling page for the Production environment, use the Exception Handling Middleware.</span></span> <span data-ttu-id="ab1e7-280">Il middleware:</span><span class="sxs-lookup"><span data-stu-id="ab1e7-280">The middleware:</span></span>

* <span data-ttu-id="ab1e7-281">Intercetta e registra le eccezioni.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-281">Catches and logs exceptions.</span></span>
* <span data-ttu-id="ab1e7-282">Esegue nuovamente la richiesta in una pipeline alternativa per la pagina o il controller indicati.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-282">Re-executes the request in an alternate pipeline for the page or controller indicated.</span></span> <span data-ttu-id="ab1e7-283">La richiesta non viene eseguita nuovamente se la risposta è stata avviata.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-283">The request isn't re-executed if the response has started.</span></span> <span data-ttu-id="ab1e7-284">Il codice generato dal modello esegue nuovamente la richiesta a `/Error` .</span><span class="sxs-lookup"><span data-stu-id="ab1e7-284">The template generated code re-executes the request to `/Error`.</span></span>

<span data-ttu-id="ab1e7-285">Nell'esempio seguente <xref:Microsoft.AspNetCore.Builder.ExceptionHandlerExtensions.UseExceptionHandler%2A> aggiunge il middleware di gestione delle eccezioni in ambienti non di sviluppo:</span><span class="sxs-lookup"><span data-stu-id="ab1e7-285">In the following example, <xref:Microsoft.AspNetCore.Builder.ExceptionHandlerExtensions.UseExceptionHandler%2A> adds the Exception Handling Middleware in non-Development environments:</span></span>

[!code-csharp[](error-handling/samples/2.x/ErrorHandlingSample/Startup.cs?name=snippet_DevPageAndHandlerPage&highlight=5-9)]

<span data-ttu-id="ab1e7-286">Il Razor modello di app pagine fornisce una pagina di errore (*cshtml*) e una <xref:Microsoft.AspNetCore.Mvc.RazorPages.PageModel> classe ( `ErrorModel` ) nella cartella *pagine* .</span><span class="sxs-lookup"><span data-stu-id="ab1e7-286">The Razor Pages app template provides an Error page (*.cshtml*) and <xref:Microsoft.AspNetCore.Mvc.RazorPages.PageModel> class (`ErrorModel`) in the *Pages* folder.</span></span> <span data-ttu-id="ab1e7-287">Per un'app MVC, il modello di progetto include un metodo di azione di errore e una visualizzazione degli errori nel controller Home.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-287">For an MVC app, the project template includes an Error action method and an Error view in the Home controller.</span></span>

<span data-ttu-id="ab1e7-288">Non contrassegnare il metodo di azione del gestore errori con attributi di metodo HTTP, ad esempio `HttpGet` .</span><span class="sxs-lookup"><span data-stu-id="ab1e7-288">Don't mark the error handler action method with HTTP method attributes, such as `HttpGet`.</span></span> <span data-ttu-id="ab1e7-289">I verbi espliciti impediscono ad alcune richieste di raggiungere il metodo.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-289">Explicit verbs prevent some requests from reaching the method.</span></span> <span data-ttu-id="ab1e7-290">Consentire l'accesso anonimo al metodo se gli utenti non autenticati dovrebbero visualizzare la visualizzazione degli errori.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-290">Allow anonymous access to the method if unauthenticated users should see the error view.</span></span>

### <a name="access-the-exception"></a><span data-ttu-id="ab1e7-291">Accedere all'eccezione</span><span class="sxs-lookup"><span data-stu-id="ab1e7-291">Access the exception</span></span>

<span data-ttu-id="ab1e7-292">Usare <xref:Microsoft.AspNetCore.Diagnostics.IExceptionHandlerPathFeature> per accedere all'eccezione e al percorso della richiesta originale in un controller o in una pagina del gestore degli errori:</span><span class="sxs-lookup"><span data-stu-id="ab1e7-292">Use <xref:Microsoft.AspNetCore.Diagnostics.IExceptionHandlerPathFeature> to access the exception and the original request path in an error handler controller or page:</span></span>

[!code-csharp[](error-handling/samples/2.x/ErrorHandlingSample/Pages/MyFolder/Error.cshtml.cs?name=snippet_ExceptionHandlerPathFeature&3,7)]

> [!WARNING]
> <span data-ttu-id="ab1e7-293">**Non** fornire informazioni sugli errori sensibili ai client.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-293">Do **not** serve sensitive error information to clients.</span></span> <span data-ttu-id="ab1e7-294">Fornire informazioni sugli errori rappresenta un rischio di sicurezza.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-294">Serving errors is a security risk.</span></span>

<span data-ttu-id="ab1e7-295">Per attivare la pagina di gestione delle eccezioni precedente, impostare l'ambiente sulle produzioni e forzare un'eccezione.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-295">To trigger the preceding exception handling page, set the environment to productions and force an exception.</span></span>

## <a name="exception-handler-lambda"></a><span data-ttu-id="ab1e7-296">Espressione lambda per il gestore di eccezioni</span><span class="sxs-lookup"><span data-stu-id="ab1e7-296">Exception handler lambda</span></span>

<span data-ttu-id="ab1e7-297">Un'alternativa a una [pagina di gestione delle eccezioni personalizzata](#exception-handler-page) consiste nel fornire un'espressione lambda a <xref:Microsoft.AspNetCore.Builder.ExceptionHandlerExtensions.UseExceptionHandler%2A>.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-297">An alternative to a [custom exception handler page](#exception-handler-page) is to provide a lambda to <xref:Microsoft.AspNetCore.Builder.ExceptionHandlerExtensions.UseExceptionHandler%2A>.</span></span> <span data-ttu-id="ab1e7-298">L'uso di un'espressione lambda consente l'accesso all'errore prima di restituire la risposta.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-298">Using a lambda allows access to the error before returning the response.</span></span>

<span data-ttu-id="ab1e7-299">Di seguito è riportato un esempio dell'uso di un'espressione lambda per la gestione delle eccezioni:</span><span class="sxs-lookup"><span data-stu-id="ab1e7-299">Here's an example of using a lambda for exception handling:</span></span>

[!code-csharp[](error-handling/samples/2.x/ErrorHandlingSample/Startup.cs?name=snippet_HandlerPageLambda)]

<span data-ttu-id="ab1e7-300">Nel codice precedente `await context.Response.WriteAsync(new string(' ', 512));` viene aggiunto, in modo che il browser Internet Explorer visualizzi il messaggio di errore anziché un messaggio di errore di IE.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-300">In the preceding code, `await context.Response.WriteAsync(new string(' ', 512));` is added so the Internet Explorer browser displays the error message rather than an IE error message.</span></span> <span data-ttu-id="ab1e7-301">Per altre informazioni, vedere [questo problema di GitHub](https://github.com/dotnet/AspNetCore.Docs/issues/16144).</span><span class="sxs-lookup"><span data-stu-id="ab1e7-301">For more information, see [this GitHub issue](https://github.com/dotnet/AspNetCore.Docs/issues/16144).</span></span>

> [!WARNING]
> <span data-ttu-id="ab1e7-302">**Non** fornire informazioni sugli errori sensibili da <xref:Microsoft.AspNetCore.Diagnostics.IExceptionHandlerFeature> o <xref:Microsoft.AspNetCore.Diagnostics.IExceptionHandlerPathFeature> ai client.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-302">Do **not** serve sensitive error information from <xref:Microsoft.AspNetCore.Diagnostics.IExceptionHandlerFeature> or <xref:Microsoft.AspNetCore.Diagnostics.IExceptionHandlerPathFeature> to clients.</span></span> <span data-ttu-id="ab1e7-303">Fornire informazioni sugli errori rappresenta un rischio di sicurezza.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-303">Serving errors is a security risk.</span></span>

<span data-ttu-id="ab1e7-304">Per visualizzare il risultato dell'espressione lambda di gestione delle eccezioni nell'[app di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/error-handling/samples), usare le direttive del preprocessore `ProdEnvironment` e `ErrorHandlerLambda` e selezionare **Trigger an exception** (Attiva un'eccezione) nella home page.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-304">To see the result of the exception handling lambda in the [sample app](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/error-handling/samples), use the `ProdEnvironment` and `ErrorHandlerLambda` preprocessor directives, and select **Trigger an exception** on the home page.</span></span>

## <a name="usestatuscodepages"></a><span data-ttu-id="ab1e7-305">UseStatusCodePages</span><span class="sxs-lookup"><span data-stu-id="ab1e7-305">UseStatusCodePages</span></span>

<span data-ttu-id="ab1e7-306">Per impostazione predefinita, un'app ASP.NET Core non offre una tabella codici di stato per i codici di stato HTTP, ad esempio *404 - Non trovato*.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-306">By default, an ASP.NET Core app doesn't provide a status code page for HTTP status codes, such as *404 - Not Found*.</span></span> <span data-ttu-id="ab1e7-307">L'app restituisce un codice di stato e un corpo della risposta vuoto.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-307">The app returns a status code and an empty response body.</span></span> <span data-ttu-id="ab1e7-308">Per specificare le tabelle codici di stato, usare il middleware delle tabelle codici di stato.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-308">To provide status code pages, use Status Code Pages middleware.</span></span>

<span data-ttu-id="ab1e7-309">Il middleware viene reso disponibile dal pacchetto [Microsoft. AspNetCore. Diagnostics](https://www.nuget.org/packages/Microsoft.AspNetCore.Diagnostics/) .</span><span class="sxs-lookup"><span data-stu-id="ab1e7-309">The middleware is made available by the [Microsoft.AspNetCore.Diagnostics](https://www.nuget.org/packages/Microsoft.AspNetCore.Diagnostics/) package.</span></span>

<span data-ttu-id="ab1e7-310">Per abilitare i gestori di solo testo predefiniti per i codici di stato di errore comuni, chiamare <xref:Microsoft.AspNetCore.Builder.StatusCodePagesExtensions.UseStatusCodePages%2A> nel metodo `Startup.Configure`:</span><span class="sxs-lookup"><span data-stu-id="ab1e7-310">To enable default text-only handlers for common error status codes, call <xref:Microsoft.AspNetCore.Builder.StatusCodePagesExtensions.UseStatusCodePages%2A> in the `Startup.Configure` method:</span></span>

[!code-csharp[](error-handling/samples/2.x/ErrorHandlingSample/Startup.cs?name=snippet_StatusCodePages)]

<span data-ttu-id="ab1e7-311">Chiamare `UseStatusCodePages` prima del middleware di gestione delle richieste (ad esempio, middleware dei file statici e middleware MVC).</span><span class="sxs-lookup"><span data-stu-id="ab1e7-311">Call `UseStatusCodePages` before request handling middleware (for example, Static File Middleware and MVC Middleware).</span></span>

<span data-ttu-id="ab1e7-312">Quando `UseStatusCodePages` non viene utilizzato, la navigazione a un URL senza un endpoint restituisce un messaggio di errore dipendente dal browser che indica che l'endpoint non è stato trovato.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-312">When `UseStatusCodePages` isn't used, navigating to a URL without an endpoint returns a browser dependent error message indicating the endpoint can't be found.</span></span> <span data-ttu-id="ab1e7-313">Ad esempio, passando a `Home/Privacy2` .</span><span class="sxs-lookup"><span data-stu-id="ab1e7-313">For example, navigating to `Home/Privacy2`.</span></span> <span data-ttu-id="ab1e7-314">Quando `UseStatusCodePages` viene chiamato il metodo, il browser restituisce:</span><span class="sxs-lookup"><span data-stu-id="ab1e7-314">When `UseStatusCodePages` is called, the browser returns:</span></span>

```
Status Code: 404; Not Found
```

## <a name="usestatuscodepages-with-format-string"></a><span data-ttu-id="ab1e7-315">UseStatusCodePages con stringa di formato</span><span class="sxs-lookup"><span data-stu-id="ab1e7-315">UseStatusCodePages with format string</span></span>

<span data-ttu-id="ab1e7-316">Per personalizzare il tipo di contenuto della risposta e il testo, usare l'overload di <xref:Microsoft.AspNetCore.Builder.StatusCodePagesExtensions.UseStatusCodePages%2A> che accetta un tipo di contenuto e una stringa di formato:</span><span class="sxs-lookup"><span data-stu-id="ab1e7-316">To customize the response content type and text, use the overload of <xref:Microsoft.AspNetCore.Builder.StatusCodePagesExtensions.UseStatusCodePages%2A> that takes a content type and format string:</span></span>

[!code-csharp[](error-handling/samples/2.x/ErrorHandlingSample/Startup.cs?name=snippet_StatusCodePagesFormatString)]

## <a name="usestatuscodepages-with-lambda"></a><span data-ttu-id="ab1e7-317">UseStatusCodePages con espressione lambda</span><span class="sxs-lookup"><span data-stu-id="ab1e7-317">UseStatusCodePages with lambda</span></span>

<span data-ttu-id="ab1e7-318">Per specificare la gestione degli errori personalizzata e il codice per la scrittura delle risposte, usare l'overload di <xref:Microsoft.AspNetCore.Builder.StatusCodePagesExtensions.UseStatusCodePages%2A> che accetta un'espressione lambda:</span><span class="sxs-lookup"><span data-stu-id="ab1e7-318">To specify custom error-handling and response-writing code, use the overload of <xref:Microsoft.AspNetCore.Builder.StatusCodePagesExtensions.UseStatusCodePages%2A> that takes a lambda expression:</span></span>

[!code-csharp[](error-handling/samples/2.x/ErrorHandlingSample/Startup.cs?name=snippet_StatusCodePagesLambda)]

## <a name="usestatuscodepageswithredirects"></a><span data-ttu-id="ab1e7-319">UseStatusCodePagesWithRedirects</span><span class="sxs-lookup"><span data-stu-id="ab1e7-319">UseStatusCodePagesWithRedirects</span></span>

<span data-ttu-id="ab1e7-320">Metodo di estensione <xref:Microsoft.AspNetCore.Builder.StatusCodePagesExtensions.UseStatusCodePagesWithRedirects%2A>:</span><span class="sxs-lookup"><span data-stu-id="ab1e7-320">The <xref:Microsoft.AspNetCore.Builder.StatusCodePagesExtensions.UseStatusCodePagesWithRedirects%2A> extension method:</span></span>

* <span data-ttu-id="ab1e7-321">Invia un codice di stato *302 - Trovato* al client.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-321">Sends a *302 - Found* status code to the client.</span></span>
* <span data-ttu-id="ab1e7-322">Reindirizza il client al percorso specificato nel modello di URL.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-322">Redirects the client to the location provided in the URL template.</span></span>

[!code-csharp[](error-handling/samples/2.x/ErrorHandlingSample/Startup.cs?name=snippet_StatusCodePagesWithRedirect)]

<span data-ttu-id="ab1e7-323">Il modello di URL può includere un segnaposto `{0}` per il codice di stato, come illustrato nell'esempio.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-323">The URL template can include a `{0}` placeholder for the status code, as shown in the example.</span></span> <span data-ttu-id="ab1e7-324">Se il modello di URL inizia con `~` (tilde), `~` viene sostituito dall'app `PathBase` .</span><span class="sxs-lookup"><span data-stu-id="ab1e7-324">If the URL template starts with `~` (tilde), the `~` is replaced by the app's `PathBase`.</span></span> <span data-ttu-id="ab1e7-325">Se si punta a un endpoint all'interno dell'app, creare una visualizzazione o una Razor pagina MVC per l'endpoint.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-325">If you point to an endpoint within the app, create an MVC view or Razor page for the endpoint.</span></span> <span data-ttu-id="ab1e7-326">Per un Razor esempio di pagine, vedere *pages/StatusCode. cshtml* nell' [app di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/error-handling/samples).</span><span class="sxs-lookup"><span data-stu-id="ab1e7-326">For a Razor Pages example, see *Pages/StatusCode.cshtml* in the [sample app](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/error-handling/samples).</span></span>

<span data-ttu-id="ab1e7-327">Questo metodo viene usato comunemente quando l'app:</span><span class="sxs-lookup"><span data-stu-id="ab1e7-327">This method is commonly used when the app:</span></span>

* <span data-ttu-id="ab1e7-328">Deve reindirizzare il client a un endpoint diverso, in genere nei casi in cui un'altra app elabora l'errore.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-328">Should redirect the client to a different endpoint, usually in cases where a different app processes the error.</span></span> <span data-ttu-id="ab1e7-329">Per le app Web, la barra degli indirizzi del browser del client riflette l'endpoint reindirizzato.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-329">For web apps, the client's browser address bar reflects the redirected endpoint.</span></span>
* <span data-ttu-id="ab1e7-330">Non deve conservare e restituire il codice di stato originale con la risposta di reindirizzamento iniziale.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-330">Shouldn't preserve and return the original status code with the initial redirect response.</span></span>

## <a name="usestatuscodepageswithreexecute"></a><span data-ttu-id="ab1e7-331">UseStatusCodePagesWithReExecute</span><span class="sxs-lookup"><span data-stu-id="ab1e7-331">UseStatusCodePagesWithReExecute</span></span>

<span data-ttu-id="ab1e7-332">Metodo di estensione <xref:Microsoft.AspNetCore.Builder.StatusCodePagesExtensions.UseStatusCodePagesWithReExecute%2A>:</span><span class="sxs-lookup"><span data-stu-id="ab1e7-332">The <xref:Microsoft.AspNetCore.Builder.StatusCodePagesExtensions.UseStatusCodePagesWithReExecute%2A> extension method:</span></span>

* <span data-ttu-id="ab1e7-333">Restituisce il codice di stato originale al client.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-333">Returns the original status code to the client.</span></span>
* <span data-ttu-id="ab1e7-334">Genera il corpo della risposta eseguendo nuovamente la pipeline delle richieste con un percorso alternativo.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-334">Generates the response body by re-executing the request pipeline using an alternate path.</span></span>

[!code-csharp[](error-handling/samples/2.x/ErrorHandlingSample/Startup.cs?name=snippet_StatusCodePagesWithReExecute)]

<span data-ttu-id="ab1e7-335">Se si punta a un endpoint all'interno dell'app, creare una visualizzazione o una Razor pagina MVC per l'endpoint.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-335">If you point to an endpoint within the app, create an MVC view or Razor page for the endpoint.</span></span> <span data-ttu-id="ab1e7-336">Assicurarsi `UseStatusCodePagesWithReExecute` che venga inserito prima `UseRouting` di in modo che sia possibile reindirizzare la richiesta alla pagina stato.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-336">Ensure `UseStatusCodePagesWithReExecute` is placed before `UseRouting` so the request can be rerouted to the status page.</span></span> <span data-ttu-id="ab1e7-337">Per un Razor esempio di pagine, vedere *pages/StatusCode. cshtml* nell' [app di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/error-handling/samples).</span><span class="sxs-lookup"><span data-stu-id="ab1e7-337">For a Razor Pages example, see *Pages/StatusCode.cshtml* in the [sample app](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/error-handling/samples).</span></span>

<span data-ttu-id="ab1e7-338">Questo metodo viene usato comunemente quando l'app deve:</span><span class="sxs-lookup"><span data-stu-id="ab1e7-338">This method is commonly used when the app should:</span></span>

* <span data-ttu-id="ab1e7-339">Elaborare la richiesta senza il reindirizzamento a un endpoint diverso.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-339">Process the request without redirecting to a different endpoint.</span></span> <span data-ttu-id="ab1e7-340">Per le app Web, la barra degli indirizzi del browser del client riflette l'endpoint richiesto in origine.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-340">For web apps, the client's browser address bar reflects the originally requested endpoint.</span></span>
* <span data-ttu-id="ab1e7-341">Conservare e restituire il codice di stato originale con la risposta.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-341">Preserve and return the original status code with the response.</span></span>

<span data-ttu-id="ab1e7-342">I modelli di URL e di stringa di query potrebbero includere un segnaposto (`{0}`) per il codice di stato.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-342">The URL and query string templates may include a placeholder (`{0}`) for the status code.</span></span> <span data-ttu-id="ab1e7-343">Il modello di URL deve iniziare con una barra (`/`).</span><span class="sxs-lookup"><span data-stu-id="ab1e7-343">The URL template must start with a slash (`/`).</span></span> <span data-ttu-id="ab1e7-344">Quando si usa un segnaposto nel percorso, verificare che l'endpoint (pagina o controller) possa elaborare il segmento del percorso.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-344">When using a placeholder in the path, confirm that the endpoint (page or controller) can process the path segment.</span></span> <span data-ttu-id="ab1e7-345">Ad esempio, una Razor pagina per gli errori deve accettare il valore del segmento di percorso facoltativo con la `@page` direttiva:</span><span class="sxs-lookup"><span data-stu-id="ab1e7-345">For example, a Razor Page for errors should accept the optional path segment value with the `@page` directive:</span></span>

```cshtml
@page "{code?}"
```

<span data-ttu-id="ab1e7-346">L'endpoint che elabora l'errore può ottenere l'URL originale che ha generato l'errore, come illustrato nell'esempio seguente:</span><span class="sxs-lookup"><span data-stu-id="ab1e7-346">The endpoint that processes the error can get the original URL that generated the error, as shown in the following example:</span></span>

[!code-csharp[](error-handling/samples/2.x/ErrorHandlingSample/Pages/StatusCode.cshtml.cs?name=snippet_StatusCodeReExecute)]

## <a name="disable-status-code-pages"></a><span data-ttu-id="ab1e7-347">Disabilitare le tabelle codici di stato</span><span class="sxs-lookup"><span data-stu-id="ab1e7-347">Disable status code pages</span></span>

<span data-ttu-id="ab1e7-348">Per disabilitare le tabelle codici di stato per un controller MVC o un metodo di azione, usare l' [`[SkipStatusCodePages]`](xref:Microsoft.AspNetCore.Mvc.SkipStatusCodePagesAttribute) attributo.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-348">To disable status code pages for an MVC controller or action method, use the [`[SkipStatusCodePages]`](xref:Microsoft.AspNetCore.Mvc.SkipStatusCodePagesAttribute) attribute.</span></span>

<span data-ttu-id="ab1e7-349">Per disabilitare le tabelle codici di stato per richieste specifiche in un Razor metodo del gestore di pagine o in un controller MVC, usare <xref:Microsoft.AspNetCore.Diagnostics.IStatusCodePagesFeature> :</span><span class="sxs-lookup"><span data-stu-id="ab1e7-349">To disable status code pages for specific requests in a Razor Pages handler method or in an MVC controller, use <xref:Microsoft.AspNetCore.Diagnostics.IStatusCodePagesFeature>:</span></span>

```csharp
var statusCodePagesFeature = HttpContext.Features.Get<IStatusCodePagesFeature>();

if (statusCodePagesFeature != null)
{
    statusCodePagesFeature.Enabled = false;
}
```

## <a name="exception-handling-code"></a><span data-ttu-id="ab1e7-350">Codice di gestione delle eccezioni</span><span class="sxs-lookup"><span data-stu-id="ab1e7-350">Exception-handling code</span></span>

<span data-ttu-id="ab1e7-351">Il codice in pagine di gestione delle eccezioni può generare eccezioni.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-351">Code in exception handling pages can throw exceptions.</span></span> <span data-ttu-id="ab1e7-352">È spesso consigliabile che le pagine di errore di produzione contengano esclusivamente contenuto statico.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-352">It's often a good idea for production error pages to consist of purely static content.</span></span>

### <a name="response-headers"></a><span data-ttu-id="ab1e7-353">Intestazioni di risposta</span><span class="sxs-lookup"><span data-stu-id="ab1e7-353">Response headers</span></span>

<span data-ttu-id="ab1e7-354">Dopo l'invio delle intestazioni per una risposta:</span><span class="sxs-lookup"><span data-stu-id="ab1e7-354">Once the headers for a response are sent:</span></span>

* <span data-ttu-id="ab1e7-355">L'app non può modificare il codice di stato della risposta.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-355">The app can't change the response's status code.</span></span>
* <span data-ttu-id="ab1e7-356">Non è possibile eseguire pagine o gestori delle eccezioni.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-356">Any exception pages or handlers can't run.</span></span> <span data-ttu-id="ab1e7-357">La risposta deve essere completata o la connessione deve essere interrotta.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-357">The response must be completed or the connection aborted.</span></span>

## <a name="server-exception-handling"></a><span data-ttu-id="ab1e7-358">Gestione delle eccezioni del server</span><span class="sxs-lookup"><span data-stu-id="ab1e7-358">Server exception handling</span></span>

<span data-ttu-id="ab1e7-359">Oltre alla logica di gestione delle eccezioni nell'app, l'[implementazione del server HTTP](xref:fundamentals/servers/index) può gestire alcune eccezioni.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-359">In addition to the exception handling logic in your app, the [HTTP server implementation](xref:fundamentals/servers/index) can handle some exceptions.</span></span> <span data-ttu-id="ab1e7-360">Se rileva un'eccezione prima dell'invio delle intestazioni di risposta, il server invia una risposta *500 - Errore interno del server* senza corpo.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-360">If the server catches an exception before response headers are sent, the server sends a *500 - Internal Server Error* response without a response body.</span></span> <span data-ttu-id="ab1e7-361">Se il server rileva un'eccezione dopo l'invio delle intestazioni di risposta, il server chiude la connessione.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-361">If the server catches an exception after response headers are sent, the server closes the connection.</span></span> <span data-ttu-id="ab1e7-362">Le richieste che non sono gestite dall'app vengono gestite dal server.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-362">Requests that aren't handled by your app are handled by the server.</span></span> <span data-ttu-id="ab1e7-363">Qualsiasi eccezione che si verifica quando il server sta gestendo la richiesta viene gestita dalla gestione delle eccezioni del server.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-363">Any exception that occurs when the server is handling the request is handled by the server's exception handling.</span></span> <span data-ttu-id="ab1e7-364">Eventuali pagine di errore personalizzate dell'app, middleware di gestione delle eccezioni e filtri non hanno effetto su questo comportamento.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-364">The app's custom error pages, exception handling middleware, and filters don't affect this behavior.</span></span>

## <a name="startup-exception-handling"></a><span data-ttu-id="ab1e7-365">Gestione delle eccezioni durante l'avvio</span><span class="sxs-lookup"><span data-stu-id="ab1e7-365">Startup exception handling</span></span>

<span data-ttu-id="ab1e7-366">Solo il livello di hosting può gestire le eccezioni che si verificano durante l'avvio dell'app.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-366">Only the hosting layer can handle exceptions that take place during app startup.</span></span> <span data-ttu-id="ab1e7-367">L'host può essere configurato per [acquisire gli errori di avvio](xref:fundamentals/host/web-host#capture-startup-errors) e [acquisire errori dettagliati](xref:fundamentals/host/web-host#detailed-errors).</span><span class="sxs-lookup"><span data-stu-id="ab1e7-367">The host can be configured to [capture startup errors](xref:fundamentals/host/web-host#capture-startup-errors) and [capture detailed errors](xref:fundamentals/host/web-host#detailed-errors).</span></span>

<span data-ttu-id="ab1e7-368">Il livello di hosting può visualizzare una pagina di errore per un errore di avvio acquisito solo se l'errore si verifica dopo l'associazione indirizzo host/porta.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-368">The hosting layer can show an error page for a captured startup error only if the error occurs after host address/port binding.</span></span> <span data-ttu-id="ab1e7-369">Se si verifica un errore di associazione:</span><span class="sxs-lookup"><span data-stu-id="ab1e7-369">If binding fails:</span></span>

* <span data-ttu-id="ab1e7-370">Il livello di hosting registra un'eccezione critica.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-370">The hosting layer logs a critical exception.</span></span>
* <span data-ttu-id="ab1e7-371">Si verifica un arresto anomalo del processo di dotnet.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-371">The dotnet process crashes.</span></span>
* <span data-ttu-id="ab1e7-372">Non viene visualizzata alcuna pagina di errore quando il server HTTP è [Kestrel](xref:fundamentals/servers/kestrel).</span><span class="sxs-lookup"><span data-stu-id="ab1e7-372">No error page is displayed when the HTTP server is [Kestrel](xref:fundamentals/servers/kestrel).</span></span>

<span data-ttu-id="ab1e7-373">Se durante l'esecuzione in [IIS](/iis) (o servizio app di Azure) o in [IIS Express](/iis/extensions/introduction-to-iis-express/iis-express-overview) il processo non può essere avviato, viene restituito l'errore *502.5 Errore del processo* dal [modulo ASP.NET Core](xref:host-and-deploy/aspnet-core-module).</span><span class="sxs-lookup"><span data-stu-id="ab1e7-373">When running on [IIS](/iis) (or Azure App Service) or [IIS Express](/iis/extensions/introduction-to-iis-express/iis-express-overview), a *502.5 - Process Failure* is returned by the [ASP.NET Core Module](xref:host-and-deploy/aspnet-core-module) if the process can't start.</span></span> <span data-ttu-id="ab1e7-374">Per altre informazioni, vedere <xref:test/troubleshoot-azure-iis>.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-374">For more information, see <xref:test/troubleshoot-azure-iis>.</span></span>

## <a name="database-error-page"></a><span data-ttu-id="ab1e7-375">Pagina di errore di database</span><span class="sxs-lookup"><span data-stu-id="ab1e7-375">Database error page</span></span>

<span data-ttu-id="ab1e7-376">Il middleware della pagina di errore del database acquisisce le eccezioni correlate al database che possono essere risolte tramite Entity Framework migrazioni.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-376">Database Error Page Middleware captures database-related exceptions that can be resolved by using Entity Framework migrations.</span></span> <span data-ttu-id="ab1e7-377">Quando si verificano queste eccezioni, viene generata una risposta HTML con i dettagli delle azioni possibili per risolvere il problema.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-377">When these exceptions occur, an HTML response with details of possible actions to resolve the issue is generated.</span></span> <span data-ttu-id="ab1e7-378">Questa pagina deve essere abilitata solo nell'ambiente di sviluppo.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-378">This page should be enabled only in the Development environment.</span></span> <span data-ttu-id="ab1e7-379">Abilitare la pagina aggiungendo codice `Startup.Configure`:</span><span class="sxs-lookup"><span data-stu-id="ab1e7-379">Enable the page by adding code to `Startup.Configure`:</span></span>

```csharp
if (env.IsDevelopment())
{
    app.UseDatabaseErrorPage();
}
```

<span data-ttu-id="ab1e7-380"><xref:Microsoft.AspNetCore.Builder.DatabaseErrorPageExtensions.UseDatabaseErrorPage%2A> richiede il pacchetto NuGet [Microsoft. AspNetCore. Diagnostics. EntityFrameworkCore](https://www.nuget.org/packages/Microsoft.AspNetCore.Diagnostics.EntityFrameworkCore/) .</span><span class="sxs-lookup"><span data-stu-id="ab1e7-380"><xref:Microsoft.AspNetCore.Builder.DatabaseErrorPageExtensions.UseDatabaseErrorPage%2A> requires the [Microsoft.AspNetCore.Diagnostics.EntityFrameworkCore](https://www.nuget.org/packages/Microsoft.AspNetCore.Diagnostics.EntityFrameworkCore/) NuGet package.</span></span>

<!-- FUTURE UPDATE: On the next topic overhaul/release update, add API crosslink to this section for xref:Microsoft.AspNetCore.Builder.DatabaseErrorPageExtensions.UseDatabaseErrorPage* when available via the API docs. -->

## <a name="exception-filters"></a><span data-ttu-id="ab1e7-381">Filtri eccezioni</span><span class="sxs-lookup"><span data-stu-id="ab1e7-381">Exception filters</span></span>

<span data-ttu-id="ab1e7-382">Nelle app MVC i filtri delle eccezioni possono essere configurati a livello globale o per ogni controller o azione singolarmente.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-382">In MVC apps, exception filters can be configured globally or on a per-controller or per-action basis.</span></span> <span data-ttu-id="ab1e7-383">Nelle Razor app Pages possono essere configurate a livello globale o per modello di pagina.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-383">In Razor Pages apps, they can be configured globally or per page model.</span></span> <span data-ttu-id="ab1e7-384">Questi filtri gestiscono tutte le eccezioni non gestite che si verificano durante l'esecuzione di un'azione del controller o di un altro filtro</span><span class="sxs-lookup"><span data-stu-id="ab1e7-384">These filters handle any unhandled exception that occurs during the execution of a controller action or another filter.</span></span> <span data-ttu-id="ab1e7-385">Per altre informazioni, vedere <xref:mvc/controllers/filters#exception-filters>.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-385">For more information, see <xref:mvc/controllers/filters#exception-filters>.</span></span>

> [!TIP]
> <span data-ttu-id="ab1e7-386">I filtri delle eccezioni sono utili per intercettare le eccezioni che si verificano all'interno di azioni MVC, ma non sono flessibili come il middleware di gestione degli errori.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-386">Exception filters are useful for trapping exceptions that occur within MVC actions, but they're not as flexible as the Exception Handling Middleware.</span></span> <span data-ttu-id="ab1e7-387">È consigliabile usare il middleware.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-387">We recommend using the middleware.</span></span> <span data-ttu-id="ab1e7-388">Usare i filtri solo quando è necessario eseguire la gestione degli errori in modo diverso in base all'azione MVC scelta.</span><span class="sxs-lookup"><span data-stu-id="ab1e7-388">Use filters only where you need to perform error handling differently based on which MVC action is chosen.</span></span>

## <a name="model-state-errors"></a><span data-ttu-id="ab1e7-389">Errori di stato del modello</span><span class="sxs-lookup"><span data-stu-id="ab1e7-389">Model state errors</span></span>

<span data-ttu-id="ab1e7-390">Per informazioni su come gestire gli errori dello stato del modello, vedere [Associazione di modelli](xref:mvc/models/model-binding) e [Convalida del modello](xref:mvc/models/validation).</span><span class="sxs-lookup"><span data-stu-id="ab1e7-390">For information about how to handle model state errors, see [Model binding](xref:mvc/models/model-binding) and [Model validation](xref:mvc/models/validation).</span></span>

## <a name="additional-resources"></a><span data-ttu-id="ab1e7-391">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="ab1e7-391">Additional resources</span></span>

* <xref:test/troubleshoot-azure-iis>
* <xref:host-and-deploy/azure-iis-errors-reference>

::: moniker-end
