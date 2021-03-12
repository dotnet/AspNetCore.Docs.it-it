---
title: Attivazione del middleware con un contenitore di terze parti in ASP.NET Core
author: rick-anderson
description: Informazioni su come usare un middleware fortemente tipizzato con un'attivazione basata su factory e un contenitore di terze parti in ASP.NET Core.
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
uid: fundamentals/middleware/extensibility-third-party-container
ms.openlocfilehash: bbbfe0e9fba33eba9d69101e5615e931ea917bc1
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102587372"
---
# <a name="middleware-activation-with-a-third-party-container-in-aspnet-core"></a>Attivazione del middleware con un contenitore di terze parti in ASP.NET Core

::: moniker range=">= aspnetcore-3.0"

In questo articolo viene illustrato come usare <xref:Microsoft.AspNetCore.Http.IMiddlewareFactory> e <xref:Microsoft.AspNetCore.Http.IMiddleware> come punto di estendibilità per l'attivazione del [middleware](xref:fundamentals/middleware/index) con un contenitore di terze parti. Per informazioni introduttive su `IMiddlewareFactory` e `IMiddleware`, vedere <xref:fundamentals/middleware/extensibility>.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/middleware/extensibility-third-party-container/samples/) ([procedura per il download](xref:index#how-to-download-a-sample))

L'app di esempio illustra l'attivazione del middleware da parte di un'implementazione `IMiddlewareFactory`, `SimpleInjectorMiddlewareFactory`. Nell'esempio viene utilizzato il contenitore di inserimento delle dipendenze [Simple Injector](https://simpleinjector.org).

L'implementazione del middleware nell'esempio registra il valore fornito da un parametro stringa di query (`key`). Il middleware usa un contesto di database inserito, ovvero un servizio con ambito, per registrare il valore della stringa di query in un database in memoria.

> [!NOTE]
> Nell'app di esempio viene utilizzato [Simple Injector](https://github.com/simpleinjector/SimpleInjector) esclusivamente a scopi dimostrativi. L'utilizzo di Simple Injector non ne implica la sponsorizzazione. Le modalità di attivazione del middleware descritte nella documentazione di Simple Injector e i problemi in GitHub sono consigliati dai gestori di Simple Injector. Per ulteriori informazioni, vedere la [documentazione di Simple Injector](https://simpleinjector.readthedocs.io/en/latest/index.html) e il [repository GitHub su Simple Injector](https://github.com/simpleinjector/SimpleInjector).

## <a name="imiddlewarefactory"></a>IMiddlewareFactory

<xref:Microsoft.AspNetCore.Http.IMiddlewareFactory> specifica i metodi per creare il middleware.

Nell'app di esempio viene implementato il middleware basato su factory per creare un'istanza di `SimpleInjectorActivatedMiddleware`. Il middleware basato su factory usa il contenitore Simple Injector per risolvere il middleware:

[!code-csharp[](extensibility-third-party-container/samples/3.x/SampleApp/Middleware/SimpleInjectorMiddlewareFactory.cs?name=snippet1&highlight=5-8,12)]

## <a name="imiddleware"></a>IMiddleware

<xref:Microsoft.AspNetCore.Http.IMiddleware> definisce il middleware per la pipeline di richieste dell'app.

Middleware attivato da un'implementazione `IMiddlewareFactory` (*Middleware/SimpleInjectorActivatedMiddleware.cs*):

[!code-csharp[](extensibility-third-party-container/samples/3.x/SampleApp/Middleware/SimpleInjectorActivatedMiddleware.cs?name=snippet1)]

Per il middleware viene creata un'estensione (*Middleware/MiddlewareExtensions.cs*):

[!code-csharp[](extensibility-third-party-container/samples/3.x/SampleApp/Middleware/MiddlewareExtensions.cs?name=snippet1)]

`Startup.ConfigureServices` deve eseguire diverse attività:

* Impostare il contenitore Simple Injector.
* Registrare la factory e il middleware.
* Rendere disponibile il contesto di database dell'app dal contenitore Simple Injector.

[!code-csharp[](extensibility-third-party-container/samples/3.x/SampleApp/Startup.cs?name=snippet1)]

Il middleware è registrato nella pipeline di elaborazione della richiesta in `Startup.Configure`:

[!code-csharp[](extensibility-third-party-container/samples/3.x/SampleApp/Startup.cs?name=snippet2&highlight=12)]

::: moniker-end

::: moniker range="< aspnetcore-3.0"

In questo articolo viene illustrato come usare <xref:Microsoft.AspNetCore.Http.IMiddlewareFactory> e <xref:Microsoft.AspNetCore.Http.IMiddleware> come punto di estendibilità per l'attivazione del [middleware](xref:fundamentals/middleware/index) con un contenitore di terze parti. Per informazioni introduttive su `IMiddlewareFactory` e `IMiddleware`, vedere <xref:fundamentals/middleware/extensibility>.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/middleware/extensibility-third-party-container/samples/) ([procedura per il download](xref:index#how-to-download-a-sample))

L'app di esempio illustra l'attivazione del middleware da parte di un'implementazione `IMiddlewareFactory`, `SimpleInjectorMiddlewareFactory`. Nell'esempio viene utilizzato il contenitore di inserimento delle dipendenze [Simple Injector](https://simpleinjector.org).

L'implementazione del middleware nell'esempio registra il valore fornito da un parametro stringa di query (`key`). Il middleware usa un contesto di database inserito, ovvero un servizio con ambito, per registrare il valore della stringa di query in un database in memoria.

> [!NOTE]
> Nell'app di esempio viene utilizzato [Simple Injector](https://github.com/simpleinjector/SimpleInjector) esclusivamente a scopi dimostrativi. L'utilizzo di Simple Injector non ne implica la sponsorizzazione. Le modalità di attivazione del middleware descritte nella documentazione di Simple Injector e i problemi in GitHub sono consigliati dai gestori di Simple Injector. Per ulteriori informazioni, vedere la [documentazione di Simple Injector](https://simpleinjector.readthedocs.io/en/latest/index.html) e il [repository GitHub su Simple Injector](https://github.com/simpleinjector/SimpleInjector).

## <a name="imiddlewarefactory"></a>IMiddlewareFactory

<xref:Microsoft.AspNetCore.Http.IMiddlewareFactory> specifica i metodi per creare il middleware.

Nell'app di esempio viene implementato il middleware basato su factory per creare un'istanza di `SimpleInjectorActivatedMiddleware`. Il middleware basato su factory usa il contenitore Simple Injector per risolvere il middleware:

[!code-csharp[](extensibility-third-party-container/samples/2.x/SampleApp/Middleware/SimpleInjectorMiddlewareFactory.cs?name=snippet1&highlight=5-8,12)]

## <a name="imiddleware"></a>IMiddleware

<xref:Microsoft.AspNetCore.Http.IMiddleware> definisce il middleware per la pipeline di richieste dell'app.

Middleware attivato da un'implementazione `IMiddlewareFactory` (*Middleware/SimpleInjectorActivatedMiddleware.cs*):

[!code-csharp[](extensibility-third-party-container/samples/2.x/SampleApp/Middleware/SimpleInjectorActivatedMiddleware.cs?name=snippet1)]

Per il middleware viene creata un'estensione (*Middleware/MiddlewareExtensions.cs*):

[!code-csharp[](extensibility-third-party-container/samples/2.x/SampleApp/Middleware/MiddlewareExtensions.cs?name=snippet1)]

`Startup.ConfigureServices` deve eseguire diverse attività:

* Impostare il contenitore Simple Injector.
* Registrare la factory e il middleware.
* Rendere disponibile il contesto di database dell'app dal contenitore Simple Injector.

[!code-csharp[](extensibility-third-party-container/samples/2.x/SampleApp/Startup.cs?name=snippet1)]

Il middleware è registrato nella pipeline di elaborazione della richiesta in `Startup.Configure`:

[!code-csharp[](extensibility-third-party-container/samples/2.x/SampleApp/Startup.cs?name=snippet2&highlight=12)]

::: moniker-end

## <a name="additional-resources"></a>Risorse aggiuntive

* [Middleware](xref:fundamentals/middleware/index)
* [Attivazione del middleware basata sulla factory](xref:fundamentals/middleware/extensibility)
* [Repository di GitHub su Simple Injector](https://github.com/simpleinjector/SimpleInjector)
* [Documentazione di Simple Injector](https://simpleinjector.readthedocs.io/en/latest/index.html)
