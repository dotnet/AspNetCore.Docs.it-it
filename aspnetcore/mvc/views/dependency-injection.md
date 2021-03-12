---
title: Inserimento di dipendenze nelle visualizzazioni in ASP.NET Core
author: ardalis
description: Informazioni sulle modalità con cui ASP.NET Core supporta l'inserimento di dipendenze nelle visualizzazioni MVC.
ms.author: riande
ms.date: 10/14/2016
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
uid: mvc/views/dependency-injection
ms.openlocfilehash: 9d4920a7b572163b048bfb06029d913cee3f9e5a
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102587253"
---
# <a name="dependency-injection-into-views-in-aspnet-core"></a>Inserimento di dipendenze nelle visualizzazioni in ASP.NET Core

Di [Steve Smith](https://ardalis.com/)

ASP.NET Core supporta l'[inserimento di dipendenze](xref:fundamentals/dependency-injection) nelle visualizzazioni. Questo può essere utile per i servizi specifici delle visualizzazioni, ad esempio per la localizzazione o per dati necessari solo per il popolamento degli elementi delle visualizzazioni. È consigliabile mantenere la [separazione delle competenze](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#separation-of-concerns) tra i controller e le visualizzazioni. La maggior parte dei dati nelle visualizzazioni devono essere passati dal controller.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/mvc/views/dependency-injection/sample) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="configuration-injection"></a>Inserimento della configurazione

*appsettings.json* i valori possono essere inseriti direttamente in una visualizzazione.

Esempio di un *appsettings.json* file:

```json
{
   "root": {
      "parent": {
         "child": "myvalue"
      }
   }
}
```

Sintassi per `@inject`: `@inject <type> <name>`

Esempio che usa `@inject`:

```csharp
@using Microsoft.Extensions.Configuration
@inject IConfiguration Configuration
@{
   string myValue = Configuration["root:parent:child"];
   ...
}
```

## <a name="service-injection"></a>Inserimento del servizio

È possibile inserire un servizio in una visualizzazione usando la direttiva `@inject`. È possibile considerare `@inject` l'aggiunta di una proprietà alla visualizzazione, con il popolamento della proprietà tramite inserimento delle dipendenze.

[!code-cshtml[](../../mvc/views/dependency-injection/sample/src/ViewInjectSample/Views/ToDo/Index.cshtml?highlight=4,5,15,16,17)]

Questa visualizzazione presenta un elenco di istanze di `ToDoItem`, nonché un riepilogo con statistiche generali. Il riepilogo è popolato dal servizio `StatisticsService` inserito. Questo servizio è registrato per l'inserimento di dipendenze in `ConfigureServices` in *Startup.cs*:

[!code-csharp[](../../mvc/views/dependency-injection/sample/src/ViewInjectSample/Startup.cs?highlight=6,7&range=15-22)]

`StatisticsService` esegue alcuni calcoli per il set di istanze di `ToDoItem`, a cui accede tramite un repository:

[!code-csharp[](../../mvc/views/dependency-injection/sample/src/ViewInjectSample/Model/Services/StatisticsService.cs?highlight=15,20,25)]

Il repository di esempio usa una raccolta in memoria. L'implementazione illustrata in precedenza, che opera su tutti i dati in memoria, non è consigliata per set di dati di grandi dimensioni a cui si accede in remoto.

L'esempio visualizza i dati del modello associato alla visualizzazione e il servizio inserito nella visualizzazione stessa:

![Visualizzazione To Do con l'elenco degli elementi totali, degli elementi completati e della priorità media, e con un elenco di attività con i relativi livelli di priorità e i valori booleani che indicano il completamento.](dependency-injection/_static/screenshot.png)

## <a name="populating-lookup-data"></a>Popolamento di dati di ricerca

L'inserimento di visualizzazioni può essere utile per popolare le opzioni negli elementi dell'interfaccia utente, ad esempio negli elenchi a discesa. Si consideri un modulo di profilo utente che include opzioni che consentono di specificare il sesso, lo stato e altre preferenze. Per il rendering di un form di questo tipo tramite un approccio MVC standard sarebbe necessario che il controller richiedesse servizi di accesso ai dati per ognuno di questi set di opzioni e quindi popolasse un modello o un `ViewBag` con ogni set di opzioni da associare.

Un approccio alternativo consiste nell'ottenere le opzioni inserendo i servizi direttamente nella visualizzazione. In questo modo la quantità di codice necessario per il controller viene ridotta al minimo, dato che la logica di costruzione di questo elemento di visualizzazione viene spostata nella visualizzazione stessa. L'azione del controller per la visualizzazione di un modulo di modifica del profilo deve solo passare l'istanza del profilo al modulo:

[!code-csharp[](../../mvc/views/dependency-injection/sample/src/ViewInjectSample/Controllers/ProfileController.cs?highlight=9,19)]

Il form HTML usato per aggiornare queste preferenze include elenchi a discesa per tre delle proprietà:

![Visualizzazione Update Profile con un modulo che consente l'immissione di nome, sesso, stato e colore preferito.](dependency-injection/_static/updateprofile.png)

Questi elenchi vengono popolati da un servizio inserito nella visualizzazione:

[!code-cshtml[](../../mvc/views/dependency-injection/sample/src/ViewInjectSample/Views/Profile/Index.cshtml?highlight=4,16,17,21,22,26,27)]

`ProfileOptionsService` è un servizio a livello di interfaccia utente progettato per fornire solo i dati necessari per il modulo:

[!code-csharp[](../../mvc/views/dependency-injection/sample/src/ViewInjectSample/Model/Services/ProfileOptionsService.cs?highlight=7,13,24)]

> [!IMPORTANT]
> Non si deve dimenticare di registrare i tipi che si richiedono tramite l'inserimento di dipendenze nel metodo `Startup.ConfigureServices`. Un tipo non registrato genera un'eccezione in fase di runtime, perché viene eseguita internamente una query al provider di servizi tramite [GetRequiredService](/dotnet/api/microsoft.extensions.dependencyinjection.serviceproviderserviceextensions.getrequiredservice).

## <a name="overriding-services"></a>Override di servizi

Oltre all'inserimento di nuovi servizi, questa tecnica può essere usata anche per eseguire l'override di servizi precedentemente inseriti in una pagina. La figura seguente mostra tutti i campi disponibili nella pagina usata nel primo esempio:

![Menu di scelta rapida di IntelliSense in un simbolo @ tipizzato con l'elenco dei campi Html, Component, StatsService e Url](dependency-injection/_static/razor-fields.png)

Come si può notare, i campi predefiniti includono `Html`, `Component`, e `Url`, nonché il servizio `StatsService` inserito. Se ad esempio si vogliono sostituire gli helper HTML predefiniti con helper personalizzati, è possibile farlo facilmente tramite `@inject`:

[!code-cshtml[](../../mvc/views/dependency-injection/sample/src/ViewInjectSample/Views/Helper/Index.cshtml?highlight=3,11)]

Se si vuole estendere i servizi esistenti, è sufficiente usare questa tecnica mentre si eredita dall'implementazione esistente o si esegue il wrapping di quest'ultima con un'implementazione personalizzata.

## <a name="see-also"></a>Vedere anche

* Blog di Simon Timms: [Getting Lookup Data Into Your View](https://blog.simontimms.com/2015/06/09/getting-lookup-data-into-you-view/) (Inserimento di dati di ricerca nei propri)
