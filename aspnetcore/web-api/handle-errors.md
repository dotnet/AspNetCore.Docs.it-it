---
title: Gestire gli errori nelle API Web di ASP.NET Core
author: pranavkm
description: Informazioni sulla gestione degli errori con ASP.NET Core le API Web.
monikerRange: '>= aspnetcore-2.1'
ms.author: prkrishn
ms.custom: mvc
ms.date: 1/11/2021
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
uid: web-api/handle-errors
ms.openlocfilehash: f9cc38f444bd3db826595b7de64db05792915a23
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102585721"
---
# <a name="handle-errors-in-aspnet-core-web-apis"></a>Gestire gli errori nelle API Web di ASP.NET Core

Questo articolo descrive come gestire e personalizzare la gestione degli errori con ASP.NET Core le API Web.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/web-api/handle-errors/samples) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="developer-exception-page"></a>Pagina delle eccezioni per gli sviluppatori

La [pagina di eccezione Developer](xref:fundamentals/error-handling) è uno strumento utile per ottenere analisi dettagliate dello stack per gli errori del server. USA <xref:Microsoft.AspNetCore.Diagnostics.DeveloperExceptionPageMiddleware> per acquisire eccezioni sincrone e asincrone dalla pipeline HTTP e per generare risposte di errore. Per illustrare, prendere in considerazione la seguente azione del controller:

[!code-csharp[](handle-errors/samples/3.x/Controllers/WeatherForecastController.cs?name=snippet_GetByCity)]

Eseguire il `curl` comando seguente per testare l'azione precedente:

```bash
curl -i https://localhost:5001/weatherforecast/chicago
```

::: moniker range=">= aspnetcore-3.0"

In ASP.NET Core 3,0 e versioni successive, la pagina delle eccezioni per sviluppatori Visualizza una risposta di testo normale se il client non richiede l'output in formato HTML. Compare l'output seguente:

```console
HTTP/1.1 500 Internal Server Error
Transfer-Encoding: chunked
Content-Type: text/plain
Server: Microsoft-IIS/10.0
X-Powered-By: ASP.NET
Date: Fri, 27 Sep 2019 16:13:16 GMT

System.ArgumentException: We don't offer a weather forecast for chicago. (Parameter 'city')
   at WebApiSample.Controllers.WeatherForecastController.Get(String city) in C:\working_folder\aspnet\AspNetCore.Docs\aspnetcore\web-api\handle-errors\samples\3.x\Controllers\WeatherForecastController.cs:line 34
   at lambda_method(Closure , Object , Object[] )
   at Microsoft.Extensions.Internal.ObjectMethodExecutor.Execute(Object target, Object[] parameters)
   at Microsoft.AspNetCore.Mvc.Infrastructure.ActionMethodExecutor.SyncObjectResultExecutor.Execute(IActionResultTypeMapper mapper, ObjectMethodExecutor executor, Object controller, Object[] arguments)
   at Microsoft.AspNetCore.Mvc.Infrastructure.ControllerActionInvoker.<InvokeActionMethodAsync>g__Logged|12_1(ControllerActionInvoker invoker)
   at Microsoft.AspNetCore.Mvc.Infrastructure.ControllerActionInvoker.<InvokeNextActionFilterAsync>g__Awaited|10_0(ControllerActionInvoker invoker, Task lastTask, State next, Scope scope, Object state, Boolean isCompleted)
   at Microsoft.AspNetCore.Mvc.Infrastructure.ControllerActionInvoker.Rethrow(ActionExecutedContextSealed context)
   at Microsoft.AspNetCore.Mvc.Infrastructure.ControllerActionInvoker.Next(State& next, Scope& scope, Object& state, Boolean& isCompleted)
   at Microsoft.AspNetCore.Mvc.Infrastructure.ControllerActionInvoker.InvokeInnerFilterAsync()
--- End of stack trace from previous location where exception was thrown ---
   at Microsoft.AspNetCore.Mvc.Infrastructure.ResourceInvoker.<InvokeFilterPipelineAsync>g__Awaited|19_0(ResourceInvoker invoker, Task lastTask, State next, Scope scope, Object state, Boolean isCompleted)
   at Microsoft.AspNetCore.Mvc.Infrastructure.ResourceInvoker.<InvokeAsync>g__Logged|17_1(ResourceInvoker invoker)
   at Microsoft.AspNetCore.Routing.EndpointMiddleware.<Invoke>g__AwaitRequestTask|6_0(Endpoint endpoint, Task requestTask, ILogger logger)
   at Microsoft.AspNetCore.Authorization.AuthorizationMiddleware.Invoke(HttpContext context)
   at Microsoft.AspNetCore.Diagnostics.DeveloperExceptionPageMiddleware.Invoke(HttpContext context)

HEADERS
=======
Accept: */*
Host: localhost:44312
User-Agent: curl/7.55.1
```

Per visualizzare invece una risposta in formato HTML, impostare l' `Accept` intestazione della richiesta HTTP sul `text/html` tipo di supporto. Ad esempio:

```bash
curl -i -H "Accept: text/html" https://localhost:5001/weatherforecast/chicago
```

Si consideri il seguente estratto della risposta HTTP:

::: moniker-end

::: moniker range="<= aspnetcore-2.2"

In ASP.NET Core 2,2 e versioni precedenti, nella pagina eccezione Developer viene visualizzata una risposta in formato HTML. Si consideri, ad esempio, il seguente estratto della risposta HTTP:

::: moniker-end

```console
HTTP/1.1 500 Internal Server Error
Transfer-Encoding: chunked
Content-Type: text/html; charset=utf-8
Server: Microsoft-IIS/10.0
X-Powered-By: ASP.NET
Date: Fri, 27 Sep 2019 16:55:37 GMT

<!DOCTYPE html>
<html lang="en" xmlns="http://www.w3.org/1999/xhtml">
    <head>
        <meta charset="utf-8" />
        <title>Internal Server Error</title>
        <style>
            body {
    font-family: 'Segoe UI', Tahoma, Arial, Helvetica, sans-serif;
    font-size: .813em;
    color: #222;
    background-color: #fff;
}
```

::: moniker range=">= aspnetcore-3.0"

La risposta in formato HTML risulta utile quando si esegue il test tramite strumenti come il post. L'acquisizione di schermate seguente mostra sia il testo normale che le risposte in formato HTML in post:

![Test della pagina di eccezione per gli sviluppatori nel post](handle-errors/_static/developer-exception-page-postman.gif)

::: moniker-end

> [!WARNING]
> Abilitare la pagina delle eccezioni **per gli sviluppatori solo quando l'app è in esecuzione nell'ambiente di sviluppo**. Non condividere informazioni dettagliate sulle eccezioni pubblicamente quando l'app viene eseguita nell'ambiente di produzione. Per altre informazioni sulla configurazione di ambienti, vedere <xref:fundamentals/environments>.
>
> Non contrassegnare il metodo di azione del gestore errori con attributi di metodo HTTP, ad esempio `HttpGet` . I verbi espliciti impediscono che alcune richieste raggiungano il metodo di azione. Consentire l'accesso anonimo al metodo se gli utenti non autenticati dovrebbero visualizzare l'errore.

## <a name="exception-handler"></a>Gestore di eccezioni

Negli ambienti non di sviluppo, è possibile usare il [middleware di gestione delle eccezioni](xref:fundamentals/error-handling) per produrre un payload degli errori:

1. In `Startup.Configure` richiamare <xref:Microsoft.AspNetCore.Builder.ExceptionHandlerExtensions.UseExceptionHandler%2A> per usare il middleware:

    ::: moniker range=">= aspnetcore-3.0"

    [!code-csharp[](handle-errors/samples/3.x/Startup.cs?name=snippet_UseExceptionHandler&highlight=9)]

    ::: moniker-end

    ::: moniker range="<= aspnetcore-2.2"

    [!code-csharp[](handle-errors/samples/2.x/2.2/Startup.cs?name=snippet_UseExceptionHandler&highlight=9)]

    ::: moniker-end

1. Configurare un'azione del controller per rispondere alla `/error` Route:

    ::: moniker range=">= aspnetcore-3.0"

    [!code-csharp[](handle-errors/samples/3.x/Controllers/ErrorController.cs?name=snippet_ErrorController)]

    ::: moniker-end

    ::: moniker range="<= aspnetcore-2.2"

    [!code-csharp[](handle-errors/samples/2.x/2.2/Controllers/ErrorController.cs?name=snippet_ErrorController)]

    ::: moniker-end

L' `Error` azione precedente invia al client un payload conforme a [RFC 7807](https://tools.ietf.org/html/rfc7807).

Il middleware di gestione delle eccezioni può inoltre fornire un output più dettagliato del contenuto negoziato nell'ambiente di sviluppo locale. Usare la procedura seguente per produrre un formato di payload coerente negli ambienti di sviluppo e di produzione:

1. In `Startup.Configure` registrare le istanze del middleware di gestione delle eccezioni specifiche dell'ambiente:

    ::: moniker range=">= aspnetcore-3.0"

    ```csharp
    public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
    {
        if (env.IsDevelopment())
        {
            app.UseExceptionHandler("/error-local-development");
        }
        else
        {
            app.UseExceptionHandler("/error");
        }
    }
    ```

    ::: moniker-end

    ::: moniker range="<= aspnetcore-2.2"

    ```csharp
    public void Configure(IApplicationBuilder app, IHostingEnvironment env)
    {
        if (env.IsDevelopment())
        {
            app.UseExceptionHandler("/error-local-development");
        }
        else
        {
            app.UseExceptionHandler("/error");
        }
    }
    ```

    ::: moniker-end

    Nel codice precedente il middleware è registrato con:

    * Route di `/error-local-development` nell'ambiente di sviluppo.
    * Una route di `/error` in ambienti che non sono in fase di sviluppo.
    
1. Applicare il routing degli attributi alle azioni del controller:

    ::: moniker range=">= aspnetcore-3.0"

    [!code-csharp[](handle-errors/samples/3.x/Controllers/ErrorController.cs?name=snippet_ErrorControllerEnvironmentSpecific)]

    ::: moniker-end

    ::: moniker range="<= aspnetcore-2.2"

    [!code-csharp[](handle-errors/samples/2.x/2.2/Controllers/ErrorController.cs?name=snippet_ErrorControllerEnvironmentSpecific)]

    ::: moniker-end

    Il codice precedente chiama [ControllerBase. problem](xref:Microsoft.AspNetCore.Mvc.ControllerBase.Problem%2A) per creare una <xref:Microsoft.AspNetCore.Mvc.ProblemDetails> risposta.

## <a name="use-exceptions-to-modify-the-response"></a>Usare le eccezioni per modificare la risposta

Il contenuto della risposta può essere modificato dall'esterno del controller. Nell'API Web ASP.NET 4. x, un modo per eseguire questa operazione consiste nell'usare il <xref:System.Web.Http.HttpResponseException> tipo. ASP.NET Core non include un tipo equivalente. `HttpResponseException`È possibile aggiungere il supporto per con i passaggi seguenti:

1. Creare un tipo di eccezione noto denominato `HttpResponseException` :

    [!code-csharp[](handle-errors/samples/3.x/Exceptions/HttpResponseException.cs?name=snippet_HttpResponseException)]

1. Creare un filtro azione denominato `HttpResponseExceptionFilter` :

    [!code-csharp[](handle-errors/samples/3.x/Filters/HttpResponseExceptionFilter.cs?name=snippet_HttpResponseExceptionFilter)]

    Il filtro precedente specifica un `Order` valore integer massimo meno 10. In questo modo è possibile eseguire altri filtri alla fine della pipeline.

1. In `Startup.ConfigureServices` aggiungere il filtro azioni alla raccolta filters:

    ::: moniker range=">= aspnetcore-3.0"

    [!code-csharp[](handle-errors/samples/3.x/Startup.cs?name=snippet_AddExceptionFilter)]

    ::: moniker-end

    ::: moniker range="= aspnetcore-2.2"
    
    [!code-csharp[](handle-errors/samples/2.x/2.2/Startup.cs?name=snippet_AddExceptionFilter)]

    ::: moniker-end

    ::: moniker range="= aspnetcore-2.1"

    [!code-csharp[](handle-errors/samples/2.x/2.1/Startup.cs?name=snippet_AddExceptionFilter)]

    ::: moniker-end

## <a name="validation-failure-error-response"></a>Risposta errore di convalida non riuscita

Per i controller API Web, MVC risponde con un <xref:Microsoft.AspNetCore.Mvc.ValidationProblemDetails> tipo di risposta quando la convalida del modello ha esito negativo. MVC usa i risultati di <xref:Microsoft.AspNetCore.Mvc.ApiBehaviorOptions.InvalidModelStateResponseFactory> per costruire la risposta di errore per un errore di convalida. Nell'esempio seguente viene usata la factory per modificare il tipo di risposta predefinito <xref:Microsoft.AspNetCore.Mvc.SerializableError> in `Startup.ConfigureServices` :

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](handle-errors/samples/3.x/Startup.cs?name=snippet_DisableProblemDetailsInvalidModelStateResponseFactory&highlight=4-13)]

::: moniker-end

::: moniker range="= aspnetcore-2.2"

[!code-csharp[](handle-errors/samples/2.x/2.2/Startup.cs?name=snippet_DisableProblemDetailsInvalidModelStateResponseFactory&highlight=5-14)]

::: moniker-end

::: moniker range="= aspnetcore-2.1"

[!code-csharp[](handle-errors/samples/2.x/2.1/Startup.cs?name=snippet_DisableProblemDetailsInvalidModelStateResponseFactory&highlight=6-15)]

::: moniker-end

## <a name="client-error-response"></a>Risposta errore client

Un *risultato di errore* viene definito come risultato con un codice di stato HTTP 400 o superiore. Per i controller API Web, MVC trasforma un risultato di errore in un risultato con <xref:Microsoft.AspNetCore.Mvc.ProblemDetails> .

::: moniker range="= aspnetcore-2.1"

> [!IMPORTANT]
> ASP.NET Core 2,1 genera una risposta dettagliata al problema che è quasi conforme a RFC 7807. Se è importante la conformità al 100%, aggiornare il progetto a ASP.NET Core 2,2 o versione successiva.

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

La risposta di errore può essere configurata in uno dei modi seguenti:

1. [Implementare ProblemDetailsFactory](#implement-problemdetailsfactory)
1. [Usare ApiBehaviorOptions. ClientErrorMapping](#use-apibehavioroptionsclienterrormapping)

### <a name="implement-problemdetailsfactory"></a>Implementare `ProblemDetailsFactory`

MVC utilizza <xref:Microsoft.AspNetCore.Mvc.Infrastructure.ProblemDetailsFactory?displayProperty=fullName> per produrre tutte le istanze di <xref:Microsoft.AspNetCore.Mvc.ProblemDetails> e <xref:Microsoft.AspNetCore.Mvc.ValidationProblemDetails> . Sono incluse le risposte degli errori del client, le risposte degli errori di convalida e i <xref:Microsoft.AspNetCore.Mvc.ControllerBase.Problem%2A?displayProperty=nameWithType> <xref:Microsoft.AspNetCore.Mvc.ControllerBase.ValidationProblem%2A?displayProperty=nameWithType> metodi helper e.

Per personalizzare la risposta dettagliata al problema, registrare un'implementazione personalizzata di <xref:Microsoft.AspNetCore.Mvc.Infrastructure.ProblemDetailsFactory> in `Startup.ConfigureServices` :

```csharp
public void ConfigureServices(IServiceCollection serviceCollection)
{
    services.AddControllers();
    services.AddTransient<ProblemDetailsFactory, CustomProblemDetailsFactory>();
}
```

::: moniker-end

::: moniker range="= aspnetcore-2.2"

La risposta di errore può essere configurata come descritto nella sezione [use ApiBehaviorOptions. ClientErrorMapping](#use-apibehavioroptionsclienterrormapping) .

::: moniker-end

::: moniker range=">= aspnetcore-2.2"

### <a name="use-apibehavioroptionsclienterrormapping"></a>Usare ApiBehaviorOptions. ClientErrorMapping

Usare la proprietà <xref:Microsoft.AspNetCore.Mvc.ApiBehaviorOptions.ClientErrorMapping%2A> per configurare il contenuto della risposta `ProblemDetails`. Ad esempio, il codice seguente in `Startup.ConfigureServices` Aggiorna la `type` proprietà per le risposte 404:

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](index/samples/3.x/Startup.cs?name=snippet_ConfigureApiBehaviorOptions&highlight=8-9)]

::: moniker-end

::: moniker range="= aspnetcore-2.2"

[!code-csharp[](index/samples/2.x/2.2/Startup.cs?name=snippet_ConfigureApiBehaviorOptions&highlight=9-10)]

::: moniker-end

## <a name="custom-middleware-to-handle-exceptions"></a>Middleware personalizzato per la gestione delle eccezioni

Le impostazioni predefinite nel middleware di gestione delle eccezioni funzionano correttamente per la maggior parte delle app. Per le app che richiedono una gestione delle eccezioni specializzata, provare a [personalizzare il middleware di gestione delle eccezioni](xref:fundamentals/error-handling#exception-handler-lambda).
