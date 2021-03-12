---
title: Attivazione del middleware basata su factory in ASP.NET Core
author: rick-anderson
description: Informazioni su come usare middleware fortemente tipizzato con un'implementazione di attivazione basata su factory in ASP.NET Core.
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 09/22/2019
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
uid: fundamentals/middleware/extensibility
ms.openlocfilehash: c4835d0d9b0cc20a1176bbb5b0aedf3c5b7d51af
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102587189"
---
# <a name="factory-based-middleware-activation-in-aspnet-core"></a>Attivazione del middleware basata su factory in ASP.NET Core

::: moniker range=">= aspnetcore-3.0"

<xref:Microsoft.AspNetCore.Http.IMiddlewareFactory>/<xref:Microsoft.AspNetCore.Http.IMiddleware> è un punto di estendibilità per l'attivazione del [middleware](xref:fundamentals/middleware/index).

I metodi di estensione <xref:Microsoft.AspNetCore.Builder.UseMiddlewareExtensions.UseMiddleware*> verificano se il tipo registrato del middleware implementa <xref:Microsoft.AspNetCore.Http.IMiddleware>. In caso affermativo, viene usata l'istanza di <xref:Microsoft.AspNetCore.Http.IMiddlewareFactory> registrata nel contenitore per risolvere l'implementazione <xref:Microsoft.AspNetCore.Http.IMiddleware>, invece di usare la logica di attivazione del middleware basata sulle convenzioni. Il middleware è registrato come un [servizio con ambito o temporaneo](xref:fundamentals/dependency-injection#service-lifetimes) nel contenitore dei servizi dell'app.

Vantaggi:

* Attivazione per ogni richiesta client (inserimento di servizi con ambito)
* Tipizzazione forte del middleware

<xref:Microsoft.AspNetCore.Http.IMiddleware> viene attivato per ogni richiesta client (connessione), in modo che i servizi con ambito possano essere inseriti nel costruttore del middleware.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/middleware/extensibility/samples) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="imiddleware"></a>IMiddleware

<xref:Microsoft.AspNetCore.Http.IMiddleware> definisce il middleware per la pipeline di richieste dell'app. Il metodo [InvokeAsync(HttpContext, RequestDelegate)](xref:Microsoft.AspNetCore.Http.IMiddleware.InvokeAsync*) gestisce le richieste e restituisce un oggetto <xref:System.Threading.Tasks.Task> che rappresenta l'esecuzione del middleware.

Middleware attivato da convenzione:

[!code-csharp[](extensibility/samples/3.x/MiddlewareExtensibilitySample/Middleware/ConventionalMiddleware.cs?name=snippet1)]

Middleware attivato da <xref:Microsoft.AspNetCore.Http.MiddlewareFactory>:

[!code-csharp[](extensibility/samples/3.x/MiddlewareExtensibilitySample/Middleware/FactoryActivatedMiddleware.cs?name=snippet1)]

Vengono create estensioni per i middleware:

[!code-csharp[](extensibility/samples/3.x/MiddlewareExtensibilitySample/Middleware/MiddlewareExtensions.cs?name=snippet1)]

Non è possibile passare gli oggetti al middleware attivato da factory con <xref:Microsoft.AspNetCore.Builder.UseMiddlewareExtensions.UseMiddleware*>:

```csharp
public static IApplicationBuilder UseFactoryActivatedMiddleware(
    this IApplicationBuilder builder, bool option)
{
    // Passing 'option' as an argument throws a NotSupportedException at runtime.
    return builder.UseMiddleware<FactoryActivatedMiddleware>(option);
}
```

Il middleware attivato da factory viene aggiunto al contenitore predefinito in `Startup.ConfigureServices`:

[!code-csharp[](extensibility/samples/3.x/MiddlewareExtensibilitySample/Startup.cs?name=snippet1&highlight=6)]

Entrambi i middleware vengono registrati nella pipeline di elaborazione della richiesta in `Startup.Configure`:

[!code-csharp[](extensibility/samples/3.x/MiddlewareExtensibilitySample/Startup.cs?name=snippet2&highlight=12-13)]

## <a name="imiddlewarefactory"></a>IMiddlewareFactory

<xref:Microsoft.AspNetCore.Http.IMiddlewareFactory> specifica i metodi per creare il middleware. L'implementazione del middleware basata su factory viene registrata nel contenitore come un servizio con ambito.

L'implementazione predefinita di <xref:Microsoft.AspNetCore.Http.IMiddlewareFactory>, <xref:Microsoft.AspNetCore.Http.MiddlewareFactory>, è presente nel pacchetto [Microsoft.AspNetCore.Http](https://www.nuget.org/packages/Microsoft.AspNetCore.Http/).

::: moniker-end

::: moniker range="< aspnetcore-3.0"

<xref:Microsoft.AspNetCore.Http.IMiddlewareFactory>/<xref:Microsoft.AspNetCore.Http.IMiddleware> è un punto di estendibilità per l'attivazione del [middleware](xref:fundamentals/middleware/index).

I metodi di estensione <xref:Microsoft.AspNetCore.Builder.UseMiddlewareExtensions.UseMiddleware*> verificano se il tipo registrato del middleware implementa <xref:Microsoft.AspNetCore.Http.IMiddleware>. In caso affermativo, viene usata l'istanza di <xref:Microsoft.AspNetCore.Http.IMiddlewareFactory> registrata nel contenitore per risolvere l'implementazione <xref:Microsoft.AspNetCore.Http.IMiddleware>, invece di usare la logica di attivazione del middleware basata sulle convenzioni. Il middleware è registrato come un [servizio con ambito o temporaneo](xref:fundamentals/dependency-injection#service-lifetimes) nel contenitore dei servizi dell'app.

Vantaggi:

* Attivazione per ogni richiesta client (inserimento di servizi con ambito)
* Tipizzazione forte del middleware

<xref:Microsoft.AspNetCore.Http.IMiddleware> viene attivato per ogni richiesta client (connessione), in modo che i servizi con ambito possano essere inseriti nel costruttore del middleware.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/middleware/extensibility/samples) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="imiddleware"></a>IMiddleware

<xref:Microsoft.AspNetCore.Http.IMiddleware> definisce il middleware per la pipeline di richieste dell'app. Il metodo [InvokeAsync(HttpContext, RequestDelegate)](xref:Microsoft.AspNetCore.Http.IMiddleware.InvokeAsync*) gestisce le richieste e restituisce un oggetto <xref:System.Threading.Tasks.Task> che rappresenta l'esecuzione del middleware.

Middleware attivato da convenzione:

[!code-csharp[](extensibility/samples/2.x/MiddlewareExtensibilitySample/Middleware/ConventionalMiddleware.cs?name=snippet1)]

Middleware attivato da <xref:Microsoft.AspNetCore.Http.MiddlewareFactory>:

[!code-csharp[](extensibility/samples/2.x/MiddlewareExtensibilitySample/Middleware/FactoryActivatedMiddleware.cs?name=snippet1)]

Vengono create estensioni per i middleware:

[!code-csharp[](extensibility/samples/2.x/MiddlewareExtensibilitySample/Middleware/MiddlewareExtensions.cs?name=snippet1)]

Non è possibile passare gli oggetti al middleware attivato da factory con <xref:Microsoft.AspNetCore.Builder.UseMiddlewareExtensions.UseMiddleware*>:

```csharp
public static IApplicationBuilder UseFactoryActivatedMiddleware(
    this IApplicationBuilder builder, bool option)
{
    // Passing 'option' as an argument throws a NotSupportedException at runtime.
    return builder.UseMiddleware<FactoryActivatedMiddleware>(option);
}
```

Il middleware attivato da factory viene aggiunto al contenitore predefinito in `Startup.ConfigureServices`:

[!code-csharp[](extensibility/samples/2.x/MiddlewareExtensibilitySample/Startup.cs?name=snippet1&highlight=6)]

Entrambi i middleware vengono registrati nella pipeline di elaborazione della richiesta in `Startup.Configure`:

[!code-csharp[](extensibility/samples/2.x/MiddlewareExtensibilitySample/Startup.cs?name=snippet2&highlight=13-14)]

## <a name="imiddlewarefactory"></a>IMiddlewareFactory

<xref:Microsoft.AspNetCore.Http.IMiddlewareFactory> specifica i metodi per creare il middleware. L'implementazione del middleware basata su factory viene registrata nel contenitore come un servizio con ambito.

L'implementazione predefinita di <xref:Microsoft.AspNetCore.Http.IMiddlewareFactory>, <xref:Microsoft.AspNetCore.Http.MiddlewareFactory>, è presente nel pacchetto [Microsoft.AspNetCore.Http](https://www.nuget.org/packages/Microsoft.AspNetCore.Http/).

::: moniker-end

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:fundamentals/middleware/index>
* <xref:fundamentals/middleware/extensibility-third-party-container>
