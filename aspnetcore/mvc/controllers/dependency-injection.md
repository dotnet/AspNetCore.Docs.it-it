---
title: Inserimento di dipendenze in controller in ASP.NET Core
author: ardalis
description: Informazioni su come i controller ASP.NET Core MVC richiedono le proprie dipendenze in modo esplicito tramite i rispettivi costruttori, usando l'inserimento delle dipendenze in ASP.NET Core.
ms.author: riande
ms.date: 02/24/2019
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
uid: mvc/controllers/dependency-injection
ms.openlocfilehash: f3654e008733c57b4cd4cd34a52b747af2258be1
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102589075"
---
# <a name="dependency-injection-into-controllers-in-aspnet-core"></a>Inserimento di dipendenze in controller in ASP.NET Core

::: moniker range=">= aspnetcore-3.0"

Di [Shadi Namrouti](https://github.com/shadinamrouti), [Rick Anderson](https://twitter.com/RickAndMSFT) e [Steve Smith](https://github.com/ardalis)

I controller ASP.NET Core MVC richiedono le proprie dipendenze in modo esplicito tramite costruttori. ASP.NET Core include il supporto predefinito per l'[inserimento di dipendenze](xref:fundamentals/dependency-injection). L'inserimento delle dipendenze rende più facile testare e gestire le app.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/mvc/controllers/dependency-injection/sample) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="constructor-injection"></a>Inserimento di costruttori

I servizi vengono aggiunti come parametri del costruttore e il runtime risolve il servizio dal contenitore di servizi. In genere i servizi vengono definiti tramite interfacce. Si consideri ad esempio un'app che richiede l'ora corrente. L'interfaccia seguente espone il servizio `IDateTime`:

[!code-csharp[](dependency-injection/3.1sample/ControllerDI/Interfaces/IDateTime.cs?name=snippet)]

Il codice seguente implementa l'interfaccia `IDateTime`:

[!code-csharp[](dependency-injection/3.1sample/ControllerDI/Services/SystemDateTime.cs?name=snippet)]

Aggiungere il servizio al contenitore di servizi:

[!code-csharp[](dependency-injection/3.1sample/ControllerDI/Startup1.cs?name=snippet&highlight=3)]

Per altre informazioni su <xref:Microsoft.Extensions.DependencyInjection.ServiceCollectionServiceExtensions.AddSingleton*>, vedere le [durate dei servizi di inserimento delle dipendenze](xref:fundamentals/dependency-injection#service-lifetimes).

Il codice seguente visualizza un messaggio di saluto all'utente in base all'orario:

[!code-csharp[](dependency-injection/3.1sample/ControllerDI/Controllers/HomeController.cs?name=snippet)]

Eseguire l'app e verrà visualizzato un messaggio in base all'orario.

## <a name="action-injection-with-fromservices"></a>Inserimento di azioni con FromServices

<xref:Microsoft.AspNetCore.Mvc.FromServicesAttribute> consente di inserire un servizio direttamente in un metodo di azione senza usare l'inserimento del costruttore:

[!code-csharp[](dependency-injection/3.1sample/ControllerDI/Controllers/HomeController.cs?name=snippet2)]

## <a name="access-settings-from-a-controller"></a>Accedere alle impostazioni da un controller

L'accesso alle impostazioni di un'app o alle impostazioni di configurazione da un controller è un tipo di azione comune. Il *modello di opzioni* descritto in <xref:fundamentals/configuration/options> rappresenta l'approccio consigliato per gestire le impostazioni. In linea generale, non inserire direttamente <xref:Microsoft.Extensions.Configuration.IConfiguration> in un controller.

Creare una classe che rappresenta le opzioni. Ad esempio:

[!code-csharp[](dependency-injection/3.1sample/ControllerDI/Models/SampleWebSettings.cs?name=snippet)]

Aggiungere la classe di configurazione alla raccolta di servizi:

[!code-csharp[](dependency-injection/3.1sample/ControllerDI/Startup.cs?highlight=4&name=snippet1)]

Configurare l'app per leggere le impostazioni da un file in formato JSON:

[!code-csharp[](dependency-injection/3.1sample/ControllerDI/Program.cs?name=snippet&range=10-15)]

Il codice seguente richiede le impostazioni `IOptions<SampleWebSettings>` dal contenitore dei servizi e le usa nel metodo `Index`:

[!code-csharp[](dependency-injection/3.1sample/ControllerDI/Controllers/SettingsController.cs?name=snippet)]

## <a name="additional-resources"></a>Risorse aggiuntive

* Vedere <xref:mvc/controllers/testing> per informazioni su come semplificare il test del codice richiedendo in modo esplicito le dipendenze nei controller.

* [Sostituire il contenitore di inserimento delle dipendenze predefinito con un'implementazione di terze parti](xref:fundamentals/dependency-injection#default-service-container-replacement).

::: moniker-end

::: moniker range="< aspnetcore-3.0"

Di [Shadi Namrouti](https://github.com/shadinamrouti), [Rick Anderson](https://twitter.com/RickAndMSFT) e [Steve Smith](https://github.com/ardalis)

I controller ASP.NET Core MVC richiedono le proprie dipendenze in modo esplicito tramite costruttori. ASP.NET Core include il supporto predefinito per l'[inserimento di dipendenze](xref:fundamentals/dependency-injection). L'inserimento delle dipendenze rende più facile testare e gestire le app.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/mvc/controllers/dependency-injection/sample) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="constructor-injection"></a>Inserimento di costruttori

I servizi vengono aggiunti come parametri del costruttore e il runtime risolve il servizio dal contenitore di servizi. In genere i servizi vengono definiti tramite interfacce. Si consideri ad esempio un'app che richiede l'ora corrente. L'interfaccia seguente espone il servizio `IDateTime`:

[!code-csharp[](dependency-injection/sample/ControllerDI/Interfaces/IDateTime.cs?name=snippet)]

Il codice seguente implementa l'interfaccia `IDateTime`:

[!code-csharp[](dependency-injection/sample/ControllerDI/Services/SystemDateTime.cs?name=snippet)]

Aggiungere il servizio al contenitore di servizi:

[!code-csharp[](dependency-injection/sample/ControllerDI/Startup1.cs?name=snippet&highlight=3)]

Per altre informazioni su <xref:Microsoft.Extensions.DependencyInjection.ServiceCollectionServiceExtensions.AddSingleton*>, vedere le [durate dei servizi di inserimento delle dipendenze](xref:fundamentals/dependency-injection#service-lifetimes).

Il codice seguente visualizza un messaggio di saluto all'utente in base all'orario:

[!code-csharp[](dependency-injection/sample/ControllerDI/Controllers/HomeController.cs?name=snippet)]

Eseguire l'app e verrà visualizzato un messaggio in base all'orario.

## <a name="action-injection-with-fromservices"></a>Inserimento di azioni con FromServices

<xref:Microsoft.AspNetCore.Mvc.FromServicesAttribute> consente di inserire un servizio direttamente in un metodo di azione senza usare l'inserimento del costruttore:

[!code-csharp[](dependency-injection/sample/ControllerDI/Controllers/HomeController.cs?name=snippet2)]

## <a name="access-settings-from-a-controller"></a>Accedere alle impostazioni da un controller

L'accesso alle impostazioni di un'app o alle impostazioni di configurazione da un controller è un tipo di azione comune. Il *modello di opzioni* descritto in <xref:fundamentals/configuration/options> rappresenta l'approccio consigliato per gestire le impostazioni. In linea generale, non inserire direttamente <xref:Microsoft.Extensions.Configuration.IConfiguration> in un controller.

Creare una classe che rappresenta le opzioni. Ad esempio:

[!code-csharp[](dependency-injection/sample/ControllerDI/Models/SampleWebSettings.cs?name=snippet)]

Aggiungere la classe di configurazione alla raccolta di servizi:

[!code-csharp[](dependency-injection/sample/ControllerDI/Startup.cs?highlight=4&name=snippet1)]

Configurare l'app per leggere le impostazioni da un file in formato JSON:

[!code-csharp[](dependency-injection/sample/ControllerDI/Program.cs?name=snippet&range=10-15)]

Il codice seguente richiede le impostazioni `IOptions<SampleWebSettings>` dal contenitore dei servizi e le usa nel metodo `Index`:

[!code-csharp[](dependency-injection/sample/ControllerDI/Controllers/SettingsController.cs?name=snippet)]

## <a name="additional-resources"></a>Risorse aggiuntive

* Vedere <xref:mvc/controllers/testing> per informazioni su come semplificare il test del codice richiedendo in modo esplicito le dipendenze nei controller.

* [Sostituire il contenitore di inserimento delle dipendenze predefinito con un'implementazione di terze parti](xref:fundamentals/dependency-injection#default-service-container-replacement).

::: moniker-end