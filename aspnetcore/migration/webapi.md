---
title: Eseguire la migrazione da API Web ASP.NET a ASP.NET Core
author: ardalis
description: Informazioni su come eseguire la migrazione di un'implementazione di API Web dall'API Web ASP.NET 4. x a ASP.NET Core MVC.
ms.author: scaddie
ms.custom: mvc
ms.date: 05/26/2020
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
uid: migration/webapi
ms.openlocfilehash: 3bd34f677271ddfc00e6e65ddf3a5e3c10eec863
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102588783"
---
# <a name="migrate-from-aspnet-web-api-to-aspnet-core"></a>Eseguire la migrazione da API Web ASP.NET a ASP.NET Core

Di [Scott Addie](https://twitter.com/scott_addie) e [Steve Smith](https://ardalis.com/)

Un'API Web ASP.NET 4. x è un servizio HTTP che raggiunge un'ampia gamma di client, inclusi browser e dispositivi mobili. ASP.NET Core combina i modelli di app per le API Web e MVC di ASP.NET 4. x in un unico modello di programmazione noto come ASP.NET Core MVC. Questo articolo illustra i passaggi necessari per eseguire la migrazione dall'API Web ASP.NET 4. x a ASP.NET Core MVC.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/migration/webapi/sample) ([procedura per il download](xref:index#how-to-download-a-sample))

::: moniker range=">= aspnetcore-3.0"

## <a name="prerequisites"></a>Prerequisiti

[!INCLUDE [prerequisites](../includes/net-core-prereqs-vs-3.1.md)]

## <a name="review-aspnet-4x-web-api-project"></a>Esaminare il progetto API Web ASP.NET 4. x

Questo articolo usa il progetto *ProductsApp* creato in [Introduzione con API Web ASP.NET 2](/aspnet/web-api/overview/getting-started-with-aspnet-web-api/tutorial-your-first-web-api). In tale progetto, un progetto API Web ASP.NET 4. x di base viene configurato come segue.

In *Global.asax.cs* viene effettuata una chiamata a `WebApiConfig.Register` :

[!code-csharp[](webapi/sample/3.x/ProductsApp/Global.asax.cs?highlight=14)]

La `WebApiConfig` classe si trova nella cartella *app_start* e dispone di un `Register` metodo statico:

[!code-csharp[](webapi/sample/3.x/ProductsApp/App_Start/WebApiConfig.cs)]

Classe precedente:

* Configura il [routing degli attributi](/aspnet/web-api/overview/web-api-routing-and-actions/attribute-routing-in-web-api-2), sebbene non venga effettivamente utilizzato.
* Configura la tabella di routing.
Il codice di esempio prevede che gli URL corrispondano al formato `/api/{controller}/{id}` , con l' `{id}` opzione facoltativa.

Le sezioni seguenti illustrano la migrazione del progetto API Web a ASP.NET Core MVC.

## <a name="create-the-destination-project"></a>Creare il progetto di destinazione

Creare una nuova soluzione vuota in Visual Studio e aggiungere il progetto API Web ASP.NET 4. x da migrare:

1. Scegliere **nuovo** progetto dal menu **file** > .
1. Selezionare il modello di **soluzione vuota** e selezionare **Avanti**.
1. Assegnare alla soluzione il nome *WebAPIMigration*. Selezionare **Crea**.
1. Aggiungere il progetto *ProductsApp* esistente alla soluzione.

Aggiungere un nuovo progetto API per la migrazione a:

1. Aggiungere un nuovo progetto di **applicazione Web ASP.NET Core** alla soluzione.
1. Nella finestra di dialogo **Configura il nuovo progetto** assegnare al progetto il nome *ProductsCore* e selezionare **Crea**.
1. Nella finestra di dialogo **Crea una nuova applicazione Web ASP.NET Core** verificare che siano selezionati **.net Core** e **ASP.NET Core 3,1** . Selezionare il modello di progetto **API** e scegliere **Crea**.
1. Rimuovere i file di esempio *weatherforecast.cs* e *Controllers/WeatherForecastController. cs* dal nuovo progetto *ProductsCore* .

La soluzione ora contiene due progetti. Le sezioni seguenti illustrano come eseguire la migrazione del contenuto del progetto *ProductsApp* al progetto *ProductsCore* .

## <a name="migrate-configuration"></a>Eseguire la migrazione della configurazione

ASP.NET Core non utilizza la cartella *app_start* o il file *Global. asax* . Il file di *web.config* viene inoltre aggiunto in fase di pubblicazione.

La classe `Startup`:

* Sostituisce *Global. asax*.
* Gestisce tutte le attività di avvio dell'app.

Per altre informazioni, vedere <xref:fundamentals/startup>.

## <a name="migrate-models-and-controllers"></a>Eseguire la migrazione di modelli e controller

Il codice seguente mostra l'oggetto `ProductsController` da aggiornare per ASP.NET Core:

[!code-csharp[](webapi/sample/3.x/ProductsApp/Controllers/ProductsController.cs)]

Aggiornare `ProductsController` per ASP.NET Core:

1. Copiare *Controllers/ProductsController. cs* e la cartella *Models* dal progetto originale a quello nuovo.
1. Modificare lo spazio dei nomi radice dei file copiati in `ProductsCore` .
1. Aggiornare l' `using ProductsApp.Models;` istruzione a `using ProductsCore.Models;` .

I componenti seguenti non esistono in ASP.NET Core:

* Classe `ApiController`
* Spazio dei nomi `System.Web.Http`
* Interfaccia `IHttpActionResult`

Apportare le modifiche seguenti:

1. Cambiare `ApiController` in <xref:Microsoft.AspNetCore.Mvc.ControllerBase>. Aggiungere `using Microsoft.AspNetCore.Mvc;` per risolvere il `ControllerBase` riferimento.
1. Eliminare `using System.Web.Http;`.
1. Modificare il `GetProduct` tipo restituito dell'azione da `IHttpActionResult` a `ActionResult<Product>` .
1. Semplificare l' `GetProduct` istruzione dell'azione `return` a quanto segue:

    ```csharp
    return product;
    ```

## <a name="configure-routing"></a>Configurare il routing

Il modello di progetto *API* ASP.NET Core include la configurazione del routing dell'endpoint nel codice generato.

Gli elementi <xref:Microsoft.AspNetCore.Builder.EndpointRoutingApplicationBuilderExtensions.UseRouting%2A> e seguenti <xref:Microsoft.AspNetCore.Builder.EndpointRoutingApplicationBuilderExtensions.UseEndpoints%2A> chiamano:

* Registrare la corrispondenza della route e l'esecuzione dell'endpoint nella pipeline [middleware](xref:fundamentals/middleware/index) .
* Sostituire il file di *app_start/webapiconfig.cs* del progetto *ProductsApp* .

[!code-csharp[](webapi/sample/3.x/ProductsCore/Startup.cs?name=snippet_Configure&highlight=10,14)]

Configurare il routing come segue:

1. Contrassegnare la `ProductsController` classe con gli attributi seguenti:

    ```csharp
    [Route("api/[controller]")]
    [ApiController]
    ```

    L' [`[Route]`](xref:Microsoft.AspNetCore.Mvc.RouteAttribute) attributo precedente configura il modello di routing degli attributi del controller. L' [`[ApiController]`](xref:Microsoft.AspNetCore.Mvc.ApiControllerAttribute) attributo rende il routing degli attributi un requisito per tutte le azioni nel controller.

    Il routing degli attributi supporta i token, ad esempio `[controller]` e `[action]` . In fase di esecuzione, ogni token viene sostituito rispettivamente dal nome del controller o dell'azione a cui è stato applicato l'attributo. I token:
    * Ridurre il numero di stringhe Magic nel progetto.
    * Verificare che le route rimangano sincronizzate con i controller e le azioni corrispondenti quando vengono applicati i refactoring di ridenominazione automatica.
1. Abilitare le richieste HTTP Get alle `ProductController` azioni:
    * Applicare l' [`[HttpGet]`](xref:Microsoft.AspNetCore.Mvc.HttpGetAttribute) attributo all' `GetAllProducts` azione.
    * Applicare l' `[HttpGet("{id}")]` attributo all' `GetProduct` azione.

Eseguire il progetto migrato e passare a `/api/products` . Viene visualizzato un elenco completo di tre prodotti. Passare a `/api/products/1`. Il primo prodotto verrà visualizzato.

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:web-api/index>
* <xref:web-api/action-return-types>
* <xref:mvc/compatibility-version>

::: moniker-end

::: moniker range="<= aspnetcore-2.2"
## <a name="prerequisites"></a>Prerequisiti

[!INCLUDE [prerequisites](../includes/net-core-prereqs-vs2019-2.2.md)]

## <a name="review-aspnet-4x-web-api-project"></a>Esaminare il progetto API Web ASP.NET 4. x

Questo articolo usa il progetto *ProductsApp* creato in [Introduzione con API Web ASP.NET 2](/aspnet/web-api/overview/getting-started-with-aspnet-web-api/tutorial-your-first-web-api). In tale progetto, un progetto API Web ASP.NET 4. x di base viene configurato come segue.

In *Global.asax.cs* viene effettuata una chiamata a `WebApiConfig.Register` :

[!code-csharp[](webapi/sample/2.x/ProductsApp/Global.asax.cs?highlight=14)]

La `WebApiConfig` classe si trova nella cartella *app_start* e dispone di un `Register` metodo statico:

[!code-csharp[](webapi/sample/2.x/ProductsApp/App_Start/WebApiConfig.cs)]

Questa classe configura il [routing degli attributi](/aspnet/web-api/overview/web-api-routing-and-actions/attribute-routing-in-web-api-2), sebbene non sia effettivamente utilizzato nel progetto. Viene inoltre configurata la tabella di routing, che viene utilizzata da API Web ASP.NET. In questo caso, l'API Web ASP.NET 4. x prevede che gli URL corrispondano al formato `/api/{controller}/{id}` , con l' `{id}` opzione facoltativo.

Le sezioni seguenti illustrano la migrazione del progetto API Web a ASP.NET Core MVC.

## <a name="create-the-destination-project"></a>Creare il progetto di destinazione

Completare i passaggi seguenti in Visual Studio:

* Passare a **file**  >  **nuovo**  >  **progetto**  >  **altri tipi di progetto**  >  **soluzioni di Visual Studio**. Selezionare **soluzione vuota** e denominare la soluzione *WebAPIMigration*. Fare clic sul pulsante **OK** .
* Aggiungere il progetto *ProductsApp* esistente alla soluzione.
* Aggiungere un nuovo progetto di **applicazione Web ASP.NET Core** alla soluzione. Selezionare **.NET Core** Target Framework dall'elenco a discesa e selezionare il modello di progetto **API** . Denominare il progetto *ProductsCore* e fare clic sul pulsante **OK** .

La soluzione ora contiene due progetti. Le sezioni seguenti illustrano come eseguire la migrazione del contenuto del progetto *ProductsApp* al progetto *ProductsCore* .

## <a name="migrate-configuration"></a>Eseguire la migrazione della configurazione

ASP.NET Core non USA:

* *App_start* cartella o il file *Global. asax*
* *web.config* file viene aggiunto in fase di pubblicazione.

La classe `Startup`:

* Sostituisce *Global. asax*.
* Gestisce tutte le attività di avvio dell'app.

Per altre informazioni, vedere <xref:fundamentals/startup>.

In ASP.NET Core MVC, il routing degli attributi è incluso per impostazione predefinita quando <xref:Microsoft.AspNetCore.Builder.MvcApplicationBuilderExtensions.UseMvc*> viene chiamato in `Startup.Configure` . La `UseMvc` chiamata seguente sostituisce il file di *app_start/webapiconfig.cs* del progetto *ProductsApp* :

[!code-csharp[](webapi/sample/2.x/ProductsCore/Startup.cs?name=snippet_Configure&highlight=13)]

## <a name="migrate-models-and-controllers"></a>Eseguire la migrazione di modelli e controller

Il codice seguente illustra l' `ProductsController` aggiornamento per ASP.NET Core: [!code-csharp[](webapi/sample/2.x/ProductsApp/Controllers/ProductsController.cs)]

Aggiornare `ProductsController` per ASP.NET Core:

1. Copiare *Controllers/ProductsController. cs* dal progetto originale a quello nuovo.
1. Copiare la cartella *Models* dal progetto originale a quella nuova.
1. Modificare lo spazio dei nomi radice dei file copiati in `ProductsCore` .
1. Aggiornare l' `using ProductsApp.Models;` istruzione a `using ProductsCore.Models;` .

I componenti seguenti non esistono in ASP.NET Core:

* Classe `ApiController`
* Spazio dei nomi `System.Web.Http`
* Interfaccia `IHttpActionResult`

Apportare le modifiche seguenti:

1. Cambiare `ApiController` in <xref:Microsoft.AspNetCore.Mvc.ControllerBase>. Aggiungere `using Microsoft.AspNetCore.Mvc;` per risolvere il `ControllerBase` riferimento.
1. Eliminare `using System.Web.Http;`.
1. Modificare il `GetProduct` tipo restituito dell'azione da `IHttpActionResult` a `ActionResult<Product>` .
1. Semplificare l' `GetProduct` istruzione dell'azione `return` a quanto segue:

    ```csharp
    return product;
    ```

## <a name="configure-routing"></a>Configurare il routing

Configurare il routing come segue:

1. Contrassegnare la `ProductsController` classe con gli attributi seguenti:

    ```csharp
    [Route("api/[controller]")]
    [ApiController]
    ```

    L' [`[Route]`](xref:Microsoft.AspNetCore.Mvc.RouteAttribute) attributo precedente configura il modello di routing degli attributi del controller. L' [`[ApiController]`](xref:Microsoft.AspNetCore.Mvc.ApiControllerAttribute) attributo rende il routing degli attributi un requisito per tutte le azioni nel controller.

    Il routing degli attributi supporta i token, ad esempio `[controller]` e `[action]` . In fase di esecuzione, ogni token viene sostituito rispettivamente dal nome del controller o dell'azione a cui è stato applicato l'attributo. I token riducono il numero di stringhe magiche nel progetto. I token assicurano inoltre che le route rimangano sincronizzate con i controller e le azioni corrispondenti quando vengono applicati i refactoring di ridenominazione automatica.
1. Impostare la modalità di compatibilità del progetto su ASP.NET Core 2,2:

    [!code-csharp[](webapi/sample/2.x/ProductsCore/Startup.cs?name=snippet_ConfigureServices&highlight=4)]

    La modifica precedente:

    * È necessario per usare l' `[ApiController]` attributo a livello di controller.
    * Consente di optare per i comportamenti potenzialmente innovatori introdotti in ASP.NET Core 2,2.
1. Abilitare le richieste HTTP Get alle `ProductController` azioni:
    * Applicare l' [`[HttpGet]`](xref:Microsoft.AspNetCore.Mvc.HttpGetAttribute) attributo all' `GetAllProducts` azione.
    * Applicare l' `[HttpGet("{id}")]` attributo all' `GetProduct` azione.

Eseguire il progetto migrato e passare a `/api/products` . Viene visualizzato un elenco completo di tre prodotti. Passare a `/api/products/1`. Il primo prodotto verrà visualizzato.

## <a name="compatibility-shim"></a>Shim di compatibilità

La libreria [Microsoft. AspNetCore. Mvc. WebApiCompatShim](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.WebApiCompatShim) fornisce uno shim di compatibilità per spostare i progetti API Web ASP.NET 4. x in ASP.NET Core. Lo shim di compatibilità estende ASP.NET Core per supportare una serie di convenzioni di ASP.NET 4. x Web API 2. L'esempio portato in precedenza in questo documento è abbastanza elementare che lo shim di compatibilità non fosse necessario. Per i progetti di grandi dimensioni, l'uso dello shim di compatibilità può essere utile per colmare temporaneamente il gap dell'API tra ASP.NET Core e ASP.NET 4. x Web API 2.

Lo shim di compatibilità dell'API Web è pensato per essere usato come misura temporanea per supportare la migrazione di progetti API Web ASP.NET 4. x di grandi dimensioni a ASP.NET Core. Nel corso del tempo, è necessario aggiornare i progetti in modo da usare ASP.NET Core modelli anziché basarsi sullo shim di compatibilità.

Le funzionalità di compatibilità incluse in `Microsoft.AspNetCore.Mvc.WebApiCompatShim` includono:

* Aggiunge un `ApiController` tipo in modo che i tipi di base dei controller non debbano essere aggiornati.
* Abilita l'associazione di modelli di tipo API Web. ASP.NET Core funzioni di associazione di modelli MVC in modo analogo a quello di ASP.NET 4. x MVC 5, per impostazione predefinita. Lo shim di compatibilità modifica l'associazione di modelli in modo da essere più simile alle convenzioni di associazione del modello ASP.NET 4. x Web API 2. Ad esempio, i tipi complessi vengono automaticamente associati dal corpo della richiesta.
* Estende l'associazione di modelli in modo che le azioni del controller possano prendere parametri di tipo `HttpRequestMessage` .
* Aggiunge formattatori di messaggi che consentono alle azioni di restituire risultati di tipo `HttpResponseMessage` .
* Aggiunge metodi di risposta aggiuntivi che possono essere usati dalle azioni API Web 2 per rispondere alle risposte:
  * `HttpResponseMessage` generatori
    * `CreateResponse<T>`
    * `CreateErrorResponse`
  * Metodi di risultato dell'azione:
    * `BadRequestErrorMessageResult`
    * `ExceptionResult`
    * `InternalServerErrorResult`
    * `InvalidModelStateResult`
    * `NegotiatedContentResult`
    * `ResponseMessageResult`
* Aggiunge un'istanza di `IContentNegotiator` al contenitore di inserimento delle dipendenze dell'app e rende disponibili i tipi correlati alla negoziazione del contenuto da [Microsoft. AspNet. WebAPI. client](https://www.nuget.org/packages/Microsoft.AspNet.WebApi.Client/). Esempi di tali tipi includono `DefaultContentNegotiator` e `MediaTypeFormatter` .

Per usare lo shim di compatibilità:

1. Installare il pacchetto NuGet [Microsoft. AspNetCore. Mvc. WebApiCompatShim](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.WebApiCompatShim) .
1. Registrare i servizi dello shim di compatibilità con il contenitore DI dell'app chiamando `services.AddMvc().AddWebApiConventions()` in `Startup.ConfigureServices` .
1. Definire route specifiche dell'API Web usando nella `MapWebApiRoute` `IRouteBuilder` chiamata dell'app `IApplicationBuilder.UseMvc` .

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:web-api/index>
* <xref:web-api/action-return-types>
* <xref:mvc/compatibility-version>
::: moniker-end
