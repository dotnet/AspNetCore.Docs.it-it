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
# <a name="factory-based-middleware-activation-in-aspnet-core"></a><span data-ttu-id="7ad7f-103">Attivazione del middleware basata su factory in ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="7ad7f-103">Factory-based middleware activation in ASP.NET Core</span></span>

::: moniker range=">= aspnetcore-3.0"

<span data-ttu-id="7ad7f-104"><xref:Microsoft.AspNetCore.Http.IMiddlewareFactory>/<xref:Microsoft.AspNetCore.Http.IMiddleware> è un punto di estendibilità per l'attivazione del [middleware](xref:fundamentals/middleware/index).</span><span class="sxs-lookup"><span data-stu-id="7ad7f-104"><xref:Microsoft.AspNetCore.Http.IMiddlewareFactory>/<xref:Microsoft.AspNetCore.Http.IMiddleware> is an extensibility point for [middleware](xref:fundamentals/middleware/index) activation.</span></span>

<span data-ttu-id="7ad7f-105">I metodi di estensione <xref:Microsoft.AspNetCore.Builder.UseMiddlewareExtensions.UseMiddleware*> verificano se il tipo registrato del middleware implementa <xref:Microsoft.AspNetCore.Http.IMiddleware>.</span><span class="sxs-lookup"><span data-stu-id="7ad7f-105"><xref:Microsoft.AspNetCore.Builder.UseMiddlewareExtensions.UseMiddleware*> extension methods check if a middleware's registered type implements <xref:Microsoft.AspNetCore.Http.IMiddleware>.</span></span> <span data-ttu-id="7ad7f-106">In caso affermativo, viene usata l'istanza di <xref:Microsoft.AspNetCore.Http.IMiddlewareFactory> registrata nel contenitore per risolvere l'implementazione <xref:Microsoft.AspNetCore.Http.IMiddleware>, invece di usare la logica di attivazione del middleware basata sulle convenzioni.</span><span class="sxs-lookup"><span data-stu-id="7ad7f-106">If it does, the <xref:Microsoft.AspNetCore.Http.IMiddlewareFactory> instance registered in the container is used to resolve the <xref:Microsoft.AspNetCore.Http.IMiddleware> implementation instead of using the convention-based middleware activation logic.</span></span> <span data-ttu-id="7ad7f-107">Il middleware è registrato come un [servizio con ambito o temporaneo](xref:fundamentals/dependency-injection#service-lifetimes) nel contenitore dei servizi dell'app.</span><span class="sxs-lookup"><span data-stu-id="7ad7f-107">The middleware is registered as a [scoped or transient service](xref:fundamentals/dependency-injection#service-lifetimes) in the app's service container.</span></span>

<span data-ttu-id="7ad7f-108">Vantaggi:</span><span class="sxs-lookup"><span data-stu-id="7ad7f-108">Benefits:</span></span>

* <span data-ttu-id="7ad7f-109">Attivazione per ogni richiesta client (inserimento di servizi con ambito)</span><span class="sxs-lookup"><span data-stu-id="7ad7f-109">Activation per client request (injection of scoped services)</span></span>
* <span data-ttu-id="7ad7f-110">Tipizzazione forte del middleware</span><span class="sxs-lookup"><span data-stu-id="7ad7f-110">Strong typing of middleware</span></span>

<span data-ttu-id="7ad7f-111"><xref:Microsoft.AspNetCore.Http.IMiddleware> viene attivato per ogni richiesta client (connessione), in modo che i servizi con ambito possano essere inseriti nel costruttore del middleware.</span><span class="sxs-lookup"><span data-stu-id="7ad7f-111"><xref:Microsoft.AspNetCore.Http.IMiddleware> is activated per client request (connection), so scoped services can be injected into the middleware's constructor.</span></span>

<span data-ttu-id="7ad7f-112">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/middleware/extensibility/samples) ([procedura per il download](xref:index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="7ad7f-112">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/middleware/extensibility/samples) ([how to download](xref:index#how-to-download-a-sample))</span></span>

## <a name="imiddleware"></a><span data-ttu-id="7ad7f-113">IMiddleware</span><span class="sxs-lookup"><span data-stu-id="7ad7f-113">IMiddleware</span></span>

<span data-ttu-id="7ad7f-114"><xref:Microsoft.AspNetCore.Http.IMiddleware> definisce il middleware per la pipeline di richieste dell'app.</span><span class="sxs-lookup"><span data-stu-id="7ad7f-114"><xref:Microsoft.AspNetCore.Http.IMiddleware> defines middleware for the app's request pipeline.</span></span> <span data-ttu-id="7ad7f-115">Il metodo [InvokeAsync(HttpContext, RequestDelegate)](xref:Microsoft.AspNetCore.Http.IMiddleware.InvokeAsync*) gestisce le richieste e restituisce un oggetto <xref:System.Threading.Tasks.Task> che rappresenta l'esecuzione del middleware.</span><span class="sxs-lookup"><span data-stu-id="7ad7f-115">The [InvokeAsync(HttpContext, RequestDelegate)](xref:Microsoft.AspNetCore.Http.IMiddleware.InvokeAsync*) method handles requests and returns a <xref:System.Threading.Tasks.Task> that represents the execution of the middleware.</span></span>

<span data-ttu-id="7ad7f-116">Middleware attivato da convenzione:</span><span class="sxs-lookup"><span data-stu-id="7ad7f-116">Middleware activated by convention:</span></span>

[!code-csharp[](extensibility/samples/3.x/MiddlewareExtensibilitySample/Middleware/ConventionalMiddleware.cs?name=snippet1)]

<span data-ttu-id="7ad7f-117">Middleware attivato da <xref:Microsoft.AspNetCore.Http.MiddlewareFactory>:</span><span class="sxs-lookup"><span data-stu-id="7ad7f-117">Middleware activated by <xref:Microsoft.AspNetCore.Http.MiddlewareFactory>:</span></span>

[!code-csharp[](extensibility/samples/3.x/MiddlewareExtensibilitySample/Middleware/FactoryActivatedMiddleware.cs?name=snippet1)]

<span data-ttu-id="7ad7f-118">Vengono create estensioni per i middleware:</span><span class="sxs-lookup"><span data-stu-id="7ad7f-118">Extensions are created for the middlewares:</span></span>

[!code-csharp[](extensibility/samples/3.x/MiddlewareExtensibilitySample/Middleware/MiddlewareExtensions.cs?name=snippet1)]

<span data-ttu-id="7ad7f-119">Non è possibile passare gli oggetti al middleware attivato da factory con <xref:Microsoft.AspNetCore.Builder.UseMiddlewareExtensions.UseMiddleware*>:</span><span class="sxs-lookup"><span data-stu-id="7ad7f-119">It isn't possible to pass objects to the factory-activated middleware with <xref:Microsoft.AspNetCore.Builder.UseMiddlewareExtensions.UseMiddleware*>:</span></span>

```csharp
public static IApplicationBuilder UseFactoryActivatedMiddleware(
    this IApplicationBuilder builder, bool option)
{
    // Passing 'option' as an argument throws a NotSupportedException at runtime.
    return builder.UseMiddleware<FactoryActivatedMiddleware>(option);
}
```

<span data-ttu-id="7ad7f-120">Il middleware attivato da factory viene aggiunto al contenitore predefinito in `Startup.ConfigureServices`:</span><span class="sxs-lookup"><span data-stu-id="7ad7f-120">The factory-activated middleware is added to the built-in container in `Startup.ConfigureServices`:</span></span>

[!code-csharp[](extensibility/samples/3.x/MiddlewareExtensibilitySample/Startup.cs?name=snippet1&highlight=6)]

<span data-ttu-id="7ad7f-121">Entrambi i middleware vengono registrati nella pipeline di elaborazione della richiesta in `Startup.Configure`:</span><span class="sxs-lookup"><span data-stu-id="7ad7f-121">Both middlewares are registered in the request processing pipeline in `Startup.Configure`:</span></span>

[!code-csharp[](extensibility/samples/3.x/MiddlewareExtensibilitySample/Startup.cs?name=snippet2&highlight=12-13)]

## <a name="imiddlewarefactory"></a><span data-ttu-id="7ad7f-122">IMiddlewareFactory</span><span class="sxs-lookup"><span data-stu-id="7ad7f-122">IMiddlewareFactory</span></span>

<span data-ttu-id="7ad7f-123"><xref:Microsoft.AspNetCore.Http.IMiddlewareFactory> specifica i metodi per creare il middleware.</span><span class="sxs-lookup"><span data-stu-id="7ad7f-123"><xref:Microsoft.AspNetCore.Http.IMiddlewareFactory> provides methods to create middleware.</span></span> <span data-ttu-id="7ad7f-124">L'implementazione del middleware basata su factory viene registrata nel contenitore come un servizio con ambito.</span><span class="sxs-lookup"><span data-stu-id="7ad7f-124">The middleware factory implementation is registered in the container as a scoped service.</span></span>

<span data-ttu-id="7ad7f-125">L'implementazione predefinita di <xref:Microsoft.AspNetCore.Http.IMiddlewareFactory>, <xref:Microsoft.AspNetCore.Http.MiddlewareFactory>, è presente nel pacchetto [Microsoft.AspNetCore.Http](https://www.nuget.org/packages/Microsoft.AspNetCore.Http/).</span><span class="sxs-lookup"><span data-stu-id="7ad7f-125">The default <xref:Microsoft.AspNetCore.Http.IMiddlewareFactory> implementation, <xref:Microsoft.AspNetCore.Http.MiddlewareFactory>, is found in the [Microsoft.AspNetCore.Http](https://www.nuget.org/packages/Microsoft.AspNetCore.Http/) package.</span></span>

::: moniker-end

::: moniker range="< aspnetcore-3.0"

<span data-ttu-id="7ad7f-126"><xref:Microsoft.AspNetCore.Http.IMiddlewareFactory>/<xref:Microsoft.AspNetCore.Http.IMiddleware> è un punto di estendibilità per l'attivazione del [middleware](xref:fundamentals/middleware/index).</span><span class="sxs-lookup"><span data-stu-id="7ad7f-126"><xref:Microsoft.AspNetCore.Http.IMiddlewareFactory>/<xref:Microsoft.AspNetCore.Http.IMiddleware> is an extensibility point for [middleware](xref:fundamentals/middleware/index) activation.</span></span>

<span data-ttu-id="7ad7f-127">I metodi di estensione <xref:Microsoft.AspNetCore.Builder.UseMiddlewareExtensions.UseMiddleware*> verificano se il tipo registrato del middleware implementa <xref:Microsoft.AspNetCore.Http.IMiddleware>.</span><span class="sxs-lookup"><span data-stu-id="7ad7f-127"><xref:Microsoft.AspNetCore.Builder.UseMiddlewareExtensions.UseMiddleware*> extension methods check if a middleware's registered type implements <xref:Microsoft.AspNetCore.Http.IMiddleware>.</span></span> <span data-ttu-id="7ad7f-128">In caso affermativo, viene usata l'istanza di <xref:Microsoft.AspNetCore.Http.IMiddlewareFactory> registrata nel contenitore per risolvere l'implementazione <xref:Microsoft.AspNetCore.Http.IMiddleware>, invece di usare la logica di attivazione del middleware basata sulle convenzioni.</span><span class="sxs-lookup"><span data-stu-id="7ad7f-128">If it does, the <xref:Microsoft.AspNetCore.Http.IMiddlewareFactory> instance registered in the container is used to resolve the <xref:Microsoft.AspNetCore.Http.IMiddleware> implementation instead of using the convention-based middleware activation logic.</span></span> <span data-ttu-id="7ad7f-129">Il middleware è registrato come un [servizio con ambito o temporaneo](xref:fundamentals/dependency-injection#service-lifetimes) nel contenitore dei servizi dell'app.</span><span class="sxs-lookup"><span data-stu-id="7ad7f-129">The middleware is registered as a [scoped or transient service](xref:fundamentals/dependency-injection#service-lifetimes) in the app's service container.</span></span>

<span data-ttu-id="7ad7f-130">Vantaggi:</span><span class="sxs-lookup"><span data-stu-id="7ad7f-130">Benefits:</span></span>

* <span data-ttu-id="7ad7f-131">Attivazione per ogni richiesta client (inserimento di servizi con ambito)</span><span class="sxs-lookup"><span data-stu-id="7ad7f-131">Activation per client request (injection of scoped services)</span></span>
* <span data-ttu-id="7ad7f-132">Tipizzazione forte del middleware</span><span class="sxs-lookup"><span data-stu-id="7ad7f-132">Strong typing of middleware</span></span>

<span data-ttu-id="7ad7f-133"><xref:Microsoft.AspNetCore.Http.IMiddleware> viene attivato per ogni richiesta client (connessione), in modo che i servizi con ambito possano essere inseriti nel costruttore del middleware.</span><span class="sxs-lookup"><span data-stu-id="7ad7f-133"><xref:Microsoft.AspNetCore.Http.IMiddleware> is activated per client request (connection), so scoped services can be injected into the middleware's constructor.</span></span>

<span data-ttu-id="7ad7f-134">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/middleware/extensibility/samples) ([procedura per il download](xref:index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="7ad7f-134">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/middleware/extensibility/samples) ([how to download](xref:index#how-to-download-a-sample))</span></span>

## <a name="imiddleware"></a><span data-ttu-id="7ad7f-135">IMiddleware</span><span class="sxs-lookup"><span data-stu-id="7ad7f-135">IMiddleware</span></span>

<span data-ttu-id="7ad7f-136"><xref:Microsoft.AspNetCore.Http.IMiddleware> definisce il middleware per la pipeline di richieste dell'app.</span><span class="sxs-lookup"><span data-stu-id="7ad7f-136"><xref:Microsoft.AspNetCore.Http.IMiddleware> defines middleware for the app's request pipeline.</span></span> <span data-ttu-id="7ad7f-137">Il metodo [InvokeAsync(HttpContext, RequestDelegate)](xref:Microsoft.AspNetCore.Http.IMiddleware.InvokeAsync*) gestisce le richieste e restituisce un oggetto <xref:System.Threading.Tasks.Task> che rappresenta l'esecuzione del middleware.</span><span class="sxs-lookup"><span data-stu-id="7ad7f-137">The [InvokeAsync(HttpContext, RequestDelegate)](xref:Microsoft.AspNetCore.Http.IMiddleware.InvokeAsync*) method handles requests and returns a <xref:System.Threading.Tasks.Task> that represents the execution of the middleware.</span></span>

<span data-ttu-id="7ad7f-138">Middleware attivato da convenzione:</span><span class="sxs-lookup"><span data-stu-id="7ad7f-138">Middleware activated by convention:</span></span>

[!code-csharp[](extensibility/samples/2.x/MiddlewareExtensibilitySample/Middleware/ConventionalMiddleware.cs?name=snippet1)]

<span data-ttu-id="7ad7f-139">Middleware attivato da <xref:Microsoft.AspNetCore.Http.MiddlewareFactory>:</span><span class="sxs-lookup"><span data-stu-id="7ad7f-139">Middleware activated by <xref:Microsoft.AspNetCore.Http.MiddlewareFactory>:</span></span>

[!code-csharp[](extensibility/samples/2.x/MiddlewareExtensibilitySample/Middleware/FactoryActivatedMiddleware.cs?name=snippet1)]

<span data-ttu-id="7ad7f-140">Vengono create estensioni per i middleware:</span><span class="sxs-lookup"><span data-stu-id="7ad7f-140">Extensions are created for the middlewares:</span></span>

[!code-csharp[](extensibility/samples/2.x/MiddlewareExtensibilitySample/Middleware/MiddlewareExtensions.cs?name=snippet1)]

<span data-ttu-id="7ad7f-141">Non è possibile passare gli oggetti al middleware attivato da factory con <xref:Microsoft.AspNetCore.Builder.UseMiddlewareExtensions.UseMiddleware*>:</span><span class="sxs-lookup"><span data-stu-id="7ad7f-141">It isn't possible to pass objects to the factory-activated middleware with <xref:Microsoft.AspNetCore.Builder.UseMiddlewareExtensions.UseMiddleware*>:</span></span>

```csharp
public static IApplicationBuilder UseFactoryActivatedMiddleware(
    this IApplicationBuilder builder, bool option)
{
    // Passing 'option' as an argument throws a NotSupportedException at runtime.
    return builder.UseMiddleware<FactoryActivatedMiddleware>(option);
}
```

<span data-ttu-id="7ad7f-142">Il middleware attivato da factory viene aggiunto al contenitore predefinito in `Startup.ConfigureServices`:</span><span class="sxs-lookup"><span data-stu-id="7ad7f-142">The factory-activated middleware is added to the built-in container in `Startup.ConfigureServices`:</span></span>

[!code-csharp[](extensibility/samples/2.x/MiddlewareExtensibilitySample/Startup.cs?name=snippet1&highlight=6)]

<span data-ttu-id="7ad7f-143">Entrambi i middleware vengono registrati nella pipeline di elaborazione della richiesta in `Startup.Configure`:</span><span class="sxs-lookup"><span data-stu-id="7ad7f-143">Both middlewares are registered in the request processing pipeline in `Startup.Configure`:</span></span>

[!code-csharp[](extensibility/samples/2.x/MiddlewareExtensibilitySample/Startup.cs?name=snippet2&highlight=13-14)]

## <a name="imiddlewarefactory"></a><span data-ttu-id="7ad7f-144">IMiddlewareFactory</span><span class="sxs-lookup"><span data-stu-id="7ad7f-144">IMiddlewareFactory</span></span>

<span data-ttu-id="7ad7f-145"><xref:Microsoft.AspNetCore.Http.IMiddlewareFactory> specifica i metodi per creare il middleware.</span><span class="sxs-lookup"><span data-stu-id="7ad7f-145"><xref:Microsoft.AspNetCore.Http.IMiddlewareFactory> provides methods to create middleware.</span></span> <span data-ttu-id="7ad7f-146">L'implementazione del middleware basata su factory viene registrata nel contenitore come un servizio con ambito.</span><span class="sxs-lookup"><span data-stu-id="7ad7f-146">The middleware factory implementation is registered in the container as a scoped service.</span></span>

<span data-ttu-id="7ad7f-147">L'implementazione predefinita di <xref:Microsoft.AspNetCore.Http.IMiddlewareFactory>, <xref:Microsoft.AspNetCore.Http.MiddlewareFactory>, è presente nel pacchetto [Microsoft.AspNetCore.Http](https://www.nuget.org/packages/Microsoft.AspNetCore.Http/).</span><span class="sxs-lookup"><span data-stu-id="7ad7f-147">The default <xref:Microsoft.AspNetCore.Http.IMiddlewareFactory> implementation, <xref:Microsoft.AspNetCore.Http.MiddlewareFactory>, is found in the [Microsoft.AspNetCore.Http](https://www.nuget.org/packages/Microsoft.AspNetCore.Http/) package.</span></span>

::: moniker-end

## <a name="additional-resources"></a><span data-ttu-id="7ad7f-148">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="7ad7f-148">Additional resources</span></span>

* <xref:fundamentals/middleware/index>
* <xref:fundamentals/middleware/extensibility-third-party-container>
