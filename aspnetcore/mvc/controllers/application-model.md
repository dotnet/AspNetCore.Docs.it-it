---
title: Usare il modello applicativo in ASP.NET Core
author: ardalis
description: Informazioni su come leggere e modificare il modello applicativo per modificare il comportamento di elementi MVC in ASP.NET Core.
ms.author: riande
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
uid: mvc/controllers/application-model
ms.openlocfilehash: cf16536284ee9c88913257607df837ad6e50ea2c
ms.sourcegitcommit: 7e394a8527c9818caebb940f692ae4fcf2f1b277
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 01/31/2021
ms.locfileid: "99217375"
---
# <a name="work-with-the-application-model-in-aspnet-core"></a>Usare il modello applicativo in ASP.NET Core

Di [Steve Smith](https://ardalis.com/)

ASP.NET Core MVC definisce un *modello applicativo* che rappresenta i componenti di un'app MVC. È possibile leggere e modificare questo modello per modificare il comportamento degli elementi MVC. Per impostazione predefinita MVC segue alcune convenzioni per determinare le classi che vengono considerate come controller, i metodi di tali classi che sono azioni e il comportamento dei parametri e del routing. È possibile personalizzare questo comportamento in base alle esigenze dell'app, creando convenzioni personalizzate e applicandole a livello globale o come attributi.

## <a name="models-and-providers"></a>Modelli e provider

Il modello di applicazione MVC ASP.NET Core include interfacce astratte e classi di implementazione concrete che descrivono un'applicazione MVC. Questo modello è il risultato del rilevamento da parte di MVC di controller, azioni, parametri di azione, route e filtri dell'app in base alle convenzioni predefinite. Mediante il modello applicativo è possibile modificare l'app in modo che segua convenzioni diverse dal comportamento predefinito di MVC. I parametri, i nomi, le route e i filtri vengono usati come dati di configurazione per le azioni e controller.

Il modello applicativo ASP.NET Core MVC presenta la struttura seguente:

* Modello applicativo (ApplicationModel)
  * Controller (ControllerModel)
    * Azioni (ActionModel)
      * Parametri (ParameterModel)

Ogni livello del modello ha accesso a una raccolta `Properties` comune. I livelli inferiori possono accedere e sovrascrivere i valori di proprietà impostati dai livelli superiori nella gerarchia. Le proprietà vengono rese persistenti in `ActionDescriptor.Properties` quando vengono create le azioni. In seguito, quando viene gestita una richiesta, le proprietà aggiunte o modificate da una convenzione sono accessibili tramite `ActionContext.ActionDescriptor.Properties`. L'uso delle proprietà è un metodo ideale per configurare filtri, strumenti di associazione modelli e altri elementi per ogni singola azione.

> [!NOTE]
> La raccolta `ActionDescriptor.Properties` non è thread-safe (per le operazioni di scrittura) una volta terminato l'avvio dell'app. Le convenzioni sono il metodo migliore per aggiungere dati in modo sicuro a questa raccolta.

### <a name="iapplicationmodelprovider"></a>IApplicationModelProvider

ASP.NET Core MVC carica il modello applicativo mediante un criterio provider definito dall'interfaccia [IApplicationModelProvider](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.iapplicationmodelprovider). La presente sezione illustra alcuni dettagli di implementazione interna associati al funzionamento di questo provider. Questo è un argomento avanzato: la maggior parte delle applicazioni usa il modello applicativo mediante le convenzioni.

Le implementazioni dell'interfaccia `IApplicationModelProvider` eseguono il wrapping l'una sull'altra e ogni implementazione chiama `OnProvidersExecuting` in ordine crescente in base alla proprietà `Order` corrispondente. Il metodo `OnProvidersExecuted` viene quindi chiamato in ordine inverso. Il framework definisce diversi provider:

First (`Order=-1000`):

* [`DefaultApplicationModelProvider`](/dotnet/api/microsoft.aspnetcore.mvc.internal.defaultapplicationmodelprovider)

Then (`Order=-990`):

* [`AuthorizationApplicationModelProvider`](/dotnet/api/microsoft.aspnetcore.mvc.internal.authorizationapplicationmodelprovider)
* [`CorsApplicationModelProvider`](/dotnet/api/microsoft.aspnetcore.mvc.cors.internal.corsapplicationmodelprovider)

> [!NOTE]
> L'ordine di chiamata di due provider con lo stesso valore di `Order` è indefinito e non va considerato affidabile.

> [!NOTE]
> `IApplicationModelProvider` è un concetto avanzato che gli autori di framework potranno estendere. In generale le app devono usare le convenzioni e i framework devono usare i provider. La differenza principale è il fatto che i provider vengono sempre eseguiti prima delle convenzioni.

`DefaultApplicationModelProvider` definisce molti comportamenti predefiniti usati da ASP.NET Core MVC. Le responsabilità includono:

* Aggiunta di filtri globali al contesto
* Aggiunta di controller al contesto
* Aggiunta di metodi del controller pubblici come azioni
* Aggiunta di parametri del metodo di azione al contesto
* Applicazione di route e altri attributi

Alcuni comportamenti predefiniti vengono implementati da `DefaultApplicationModelProvider`. Questo provider è responsabile della costruzione di [`ControllerModel`](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.controllermodel) , che a sua volta fa riferimento a istanze di, [`ActionModel`](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.actionmodel) [`PropertyModel`](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.propertymodel) e [`ParameterModel`](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.parametermodel) . La classe `DefaultApplicationModelProvider` è un dettaglio di implementazione interno del framework che verrà modificato in futuro. 

`AuthorizationApplicationModelProvider` è responsabile dell'applicazione del comportamento associato agli attributi `AuthorizeFilter` e `AllowAnonymousFilter`. [Altre informazioni su questi attributi](xref:security/authorization/simple).

`CorsApplicationModelProvider` implementa il comportamento associato a `IEnableCorsAttribute` e `IDisableCorsAttribute` e anche a `DisableCorsAuthorizationFilter`. [Altre informazioni su CORS](xref:security/cors).

## <a name="conventions"></a>Convenzioni

Il modello applicativo definisce astrazioni di convenzioni che offrono un metodo di personalizzazione del comportamento dei modelli più semplice rispetto all'override dell'intero modello o provider. Queste astrazioni sono il metodo consigliato per modificare il comportamento dell'app. Le convenzioni offrono una modalità di scrittura del codice che applica in modo dinamico le personalizzazioni. Sebbene i [filtri](xref:mvc/controllers/filters) offrano un mezzo per modificare il comportamento del Framework, le personalizzazioni consentono di controllare il funzionamento dell'intera applicazione.

Sono disponibili le convenzioni seguenti:

* [`IApplicationModelConvention`](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.iapplicationmodelconvention)
* [`IControllerModelConvention`](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.icontrollermodelconvention)
* [`IActionModelConvention`](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.iactionmodelconvention)
* [`IParameterModelConvention`](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.iparametermodelconvention)

Le convenzioni vengono applicate aggiungendole alle opzioni MVC o implementando `Attribute` e applicando i controlli a controller, azioni o parametri di azione (simile a [`Filters`](xref:mvc/controllers/filters) ). A differenza dei filtri, le convenzioni vengono eseguite solo durante l'avvio dell'app e non come parte di ogni singola richiesta.

### <a name="sample-modifying-the-applicationmodel"></a>Esempio: Modifica di ApplicationModel

La convenzione seguente viene usata per aggiungere una proprietà al modello applicativo. 

[!code-csharp[](./application-model/sample/src/AppModelSample/Conventions/ApplicationDescription.cs)]

Le convenzioni del modello applicativo vengono applicate come opzioni quando MVC viene aggiunta a `ConfigureServices` in `Startup`.

[!code-csharp[](./application-model/sample/src/AppModelSample/Startup.cs?name=ConfigureServices&highlight=5)]

Le proprietà sono accessibili dalla raccolta di proprietà `ActionDescriptor` nelle azioni del controller:

[!code-csharp[](./application-model/sample/src/AppModelSample/Controllers/AppModelController.cs?name=AppModelController)]

### <a name="sample-modifying-the-controllermodel-description"></a>Esempio: Modifica della descrizione ControllerModel

Come nell'esempio precedente, è anche possibile modificare il modello di controller in modo che includa proprietà personalizzate. Queste sostituiranno le proprietà esistenti con lo stesso nome specificate nel modello applicativo. L'attributo di convenzione seguente aggiunge una descrizione a livello del controller:

[!code-csharp[](./application-model/sample/src/AppModelSample/Conventions/ControllerDescriptionAttribute.cs)]

Questa convenzione viene applicata come attributo in un controller.

[!code-csharp[](./application-model/sample/src/AppModelSample/Controllers/DescriptionAttributesController.cs?name=ControllerDescription&highlight=1)]

L'accesso alla proprietà "description" avviene come negli esempi precedenti.

### <a name="sample-modifying-the-actionmodel-description"></a>Esempio: Modifica della descrizione ActionModel

È possibile applicare una convenzione di attributo distinta a singole azioni, eseguendo l'override del comportamento già applicato a livello dell'applicazione o del controller.

[!code-csharp[](./application-model/sample/src/AppModelSample/Conventions/ActionDescriptionAttribute.cs)]

L'applicazione di questa convenzione a un'azione nel controller dell'esempio precedente visualizza come viene eseguito l'override della convenzione a livello di controller:

[!code-csharp[](./application-model/sample/src/AppModelSample/Controllers/DescriptionAttributesController.cs?name=DescriptionAttributesController&highlight=9)]

### <a name="sample-modifying-the-parametermodel"></a>Esempio: Modifica di ParameterModel

È possibile applicare la convenzione seguente a parametri di azione per modificarne l'elemento `BindingInfo`. La convenzione seguente richiede che il parametro sia un parametro di route. Le altri origini di associazione possibili (ad esempio i valori stringa di query) vengono ignorati.

[!code-csharp[](./application-model/sample/src/AppModelSample/Conventions/MustBeInRouteParameterModelConvention.cs)]

L'attributo può essere applicato a qualsiasi parametro di azione:

[!code-csharp[](./application-model/sample/src/AppModelSample/Controllers/ParameterModelController.cs?name=ParameterModelController&highlight=5)]

### <a name="sample-modifying-the-actionmodel-name"></a>Esempio: Modifica del nome di ActionModel

La convenzione seguente modifica `ActionModel` per aggiornare l'elemento *name* dell'azione alla quale viene applicata. Il nuovo nome viene passato all'attributo come parametro. Il nuovo nome viene usato in base al routing, pertanto ha effetto sulla route usata per raggiungere questo metodo di azione.

[!code-csharp[](./application-model/sample/src/AppModelSample/Conventions/CustomActionNameAttribute.cs)]

Questo attributo viene applicato a un metodo di azione in `HomeController`:

[!code-csharp[](./application-model/sample/src/AppModelSample/Controllers/HomeController.cs?name=ActionModelConvention&highlight=2)]

Anche se il nome del metodo è `SomeName` l'attributo esegue l'override della convenzione MVC che prevede l'uso del nome del metodo e sostituisce il nome dell'azione con `MyCoolAction`. Pertanto la route usata per raggiungere questa azione è `/Home/MyCoolAction`.

> [!NOTE]
> Questo esempio ottiene in pratica lo stesso risultato dell'uso dell'attributo [ActionName](/dotnet/api/microsoft.aspnetcore.mvc.actionnameattribute) incorporato.

### <a name="sample-custom-routing-convention"></a>Esempio: Convenzione per il routing personalizzato

È possibile usare una convenzione `IApplicationModelConvention` per personalizzare il funzionamento del routing. Ad esempio la convenzione seguente incorpora gli spazi dei nomi dei controller nelle rispettive route, sostituendo `.` nello spazio dei nomi con `/` nella route:

[!code-csharp[](./application-model/sample/src/AppModelSample/Conventions/NamespaceRoutingConvention.cs)]

La convenzione viene aggiunta come opzione in Startup.

[!code-csharp[](./application-model/sample/src/AppModelSample/Startup.cs?name=ConfigureServices&highlight=6)]

> [!TIP]
> È possibile aggiungere convenzioni al [middleware](xref:fundamentals/middleware/index) accedendo a `MvcOptions` mediante `services.Configure<MvcOptions>(c => c.Conventions.Add(YOURCONVENTION));`.

In questo esempio la convenzione viene applicata alle route che non usano il routing attributo in cui il nome del controller contiene "Namespace". Il controller seguente illustra questa convenzione:

[!code-csharp[](./application-model/sample/src/AppModelSample/Controllers/NamespaceRoutingController.cs?highlight=7-8)]

## <a name="application-model-usage-in-webapicompatshim"></a>Uso del modello applicativo in WebApiCompatShim

ASP.NET Core MVC usa un set di convenzioni diverso rispetto all'API Web ASP.NET 2. Con le convenzioni personalizzate è possibile modificare il comportamento di un'app ASP.NET Core MVC per renderlo coerente con quello di un'app API Web. Microsoft offre [WebApiCompatShim](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.WebApiCompatShim/) per questo scopo specifico.

> [!NOTE]
> Altre informazioni sulla [migrazione dall'API Web ASP.NET](xref:migration/webapi).

Per usare lo shim Web API Compatibility è necessario aggiungere il pacchetto al progetto e quindi aggiungere le convenzioni a MVC chiamando `AddWebApiConventions` in `Startup`:

```csharp
services.AddMvc().AddWebApiConventions();
```

Le convenzioni offerte dallo shim vengono applicate solo alle parti dell'app a cui sono applicati determinati attributi. I quattro attributi seguenti vengono usati per determinare i controller le cui convenzioni verranno modificate dalle convenzioni dello shim:

* [UseWebApiActionConventionsAttribute](/dotnet/api/microsoft.aspnetcore.mvc.webapicompatshim.usewebapiactionconventionsattribute)
* [UseWebApiOverloadingAttribute](/dotnet/api/microsoft.aspnetcore.mvc.webapicompatshim.usewebapioverloadingattribute)
* [UseWebApiParameterConventionsAttribute](/dotnet/api/microsoft.aspnetcore.mvc.webapicompatshim.usewebapiparameterconventionsattribute)
* [UseWebApiRoutesAttribute](/dotnet/api/microsoft.aspnetcore.mvc.webapicompatshim.usewebapiroutesattribute)

### <a name="action-conventions"></a>Convenzioni per le azioni

`UseWebApiActionConventionsAttribute` viene usato per eseguire il mapping del metodo HTTP alle azioni in base al nome (ad esempio `Get` esegue il mapping a `HttpGet`). Si applica solo alle azioni che non usano il routing degli attributi.

### <a name="overloading"></a>Overload

`UseWebApiOverloadingAttribute` viene usato per applicare la convenzione `WebApiOverloadingApplicationModelConvention`. Questa convenzione aggiunge un elemento `OverloadActionConstraint` al processo di selezione azioni, che limita le azioni candidate a quelle per cui la richiesta soddisfa tutti i parametri non facoltativi.

### <a name="parameter-conventions"></a>Convenzioni dei parametri

`UseWebApiParameterConventionsAttribute` viene usato per applicare la convenzione di azione `WebApiParameterConventionsApplicationModelConvention`. Questa convenzione specifica che i tipi semplici usati come parametri di azione sono associati dall'URI per impostazione predefinita, mentre i tipi complessi sono associati dal corpo della richiesta.

### <a name="routes"></a>Route

`UseWebApiRoutesAttribute` controlla se la convenzione del controller `WebApiApplicationModelConvention` viene applicata. Quando è abilitata, questa convenzione viene usata per aggiungere alla route il supporto per le [aree](xref:mvc/controllers/areas).

Oltre a un set di convenzioni, il pacchetto di compatibilità include una classe base `System.Web.Http.ApiController` che sostituisce quella offerta dall'API Web. In questo modo i controller creati per l'API Web che ereditano da `ApiController` dell'API possono funzionare come sono stati progettati, anche quando vengono eseguiti in ASP.NET Core MVC. Tutti gli `UseWebApi*` attributi elencati in precedenza vengono applicati alla classe controller di base. `ApiController` espone le proprietà, i metodi e i tipi di risultati compatibili con quelli disponibili nell'API Web.

## <a name="using-apiexplorer-to-document-your-app"></a>Uso di ApiExplorer per documentare l'app

Il modello applicativo espone una [`ApiExplorer`](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.apiexplorermodel) proprietà a ogni livello che può essere usata per attraversare la struttura dell'app. Questa funzionalità può essere usata per [generare pagine della Guida per le API Web usando strumenti come Swagger](xref:tutorials/web-api-help-pages-using-swagger). La proprietà `ApiExplorer` espone una proprietà `IsVisible` che può essere impostata per specificare quali parti del modello dell'app devono essere esposte. È possibile configurare questa impostazione usando una convenzione:

[!code-csharp[](./application-model/sample/src/AppModelSample/Conventions/EnableApiExplorerApplicationConvention.cs)]

Usando questo approccio (e se necessario convenzioni aggiuntive), è possibile attivare o disattivare la visibilità delle API a qualsiasi livello all'interno dell'app. 
