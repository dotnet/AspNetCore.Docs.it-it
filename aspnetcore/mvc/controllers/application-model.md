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
# <a name="work-with-the-application-model-in-aspnet-core"></a><span data-ttu-id="34da4-103">Usare il modello applicativo in ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="34da4-103">Work with the application model in ASP.NET Core</span></span>

<span data-ttu-id="34da4-104">Di [Steve Smith](https://ardalis.com/)</span><span class="sxs-lookup"><span data-stu-id="34da4-104">By [Steve Smith](https://ardalis.com/)</span></span>

<span data-ttu-id="34da4-105">ASP.NET Core MVC definisce un *modello applicativo* che rappresenta i componenti di un'app MVC.</span><span class="sxs-lookup"><span data-stu-id="34da4-105">ASP.NET Core MVC defines an *application model* representing the components of an MVC app.</span></span> <span data-ttu-id="34da4-106">È possibile leggere e modificare questo modello per modificare il comportamento degli elementi MVC.</span><span class="sxs-lookup"><span data-stu-id="34da4-106">You can read and manipulate this model to modify how MVC elements behave.</span></span> <span data-ttu-id="34da4-107">Per impostazione predefinita MVC segue alcune convenzioni per determinare le classi che vengono considerate come controller, i metodi di tali classi che sono azioni e il comportamento dei parametri e del routing.</span><span class="sxs-lookup"><span data-stu-id="34da4-107">By default, MVC follows certain conventions to determine which classes are considered to be controllers, which methods on those classes are actions, and how parameters and routing behave.</span></span> <span data-ttu-id="34da4-108">È possibile personalizzare questo comportamento in base alle esigenze dell'app, creando convenzioni personalizzate e applicandole a livello globale o come attributi.</span><span class="sxs-lookup"><span data-stu-id="34da4-108">You can customize this behavior to suit your app's needs by creating your own conventions and applying them globally or as attributes.</span></span>

## <a name="models-and-providers"></a><span data-ttu-id="34da4-109">Modelli e provider</span><span class="sxs-lookup"><span data-stu-id="34da4-109">Models and Providers</span></span>

<span data-ttu-id="34da4-110">Il modello di applicazione MVC ASP.NET Core include interfacce astratte e classi di implementazione concrete che descrivono un'applicazione MVC.</span><span class="sxs-lookup"><span data-stu-id="34da4-110">The ASP.NET Core MVC application model includes both abstract interfaces and concrete implementation classes that describe an MVC application.</span></span> <span data-ttu-id="34da4-111">Questo modello è il risultato del rilevamento da parte di MVC di controller, azioni, parametri di azione, route e filtri dell'app in base alle convenzioni predefinite.</span><span class="sxs-lookup"><span data-stu-id="34da4-111">This model is the result of MVC discovering the app's controllers, actions, action parameters, routes, and filters according to default conventions.</span></span> <span data-ttu-id="34da4-112">Mediante il modello applicativo è possibile modificare l'app in modo che segua convenzioni diverse dal comportamento predefinito di MVC.</span><span class="sxs-lookup"><span data-stu-id="34da4-112">By working with the application model, you can modify your app to follow different conventions from the default MVC behavior.</span></span> <span data-ttu-id="34da4-113">I parametri, i nomi, le route e i filtri vengono usati come dati di configurazione per le azioni e controller.</span><span class="sxs-lookup"><span data-stu-id="34da4-113">The parameters, names, routes, and filters are all used as configuration data for actions and controllers.</span></span>

<span data-ttu-id="34da4-114">Il modello applicativo ASP.NET Core MVC presenta la struttura seguente:</span><span class="sxs-lookup"><span data-stu-id="34da4-114">The ASP.NET Core MVC Application Model has the following structure:</span></span>

* <span data-ttu-id="34da4-115">Modello applicativo (ApplicationModel)</span><span class="sxs-lookup"><span data-stu-id="34da4-115">ApplicationModel</span></span>
  * <span data-ttu-id="34da4-116">Controller (ControllerModel)</span><span class="sxs-lookup"><span data-stu-id="34da4-116">Controllers (ControllerModel)</span></span>
    * <span data-ttu-id="34da4-117">Azioni (ActionModel)</span><span class="sxs-lookup"><span data-stu-id="34da4-117">Actions (ActionModel)</span></span>
      * <span data-ttu-id="34da4-118">Parametri (ParameterModel)</span><span class="sxs-lookup"><span data-stu-id="34da4-118">Parameters (ParameterModel)</span></span>

<span data-ttu-id="34da4-119">Ogni livello del modello ha accesso a una raccolta `Properties` comune. I livelli inferiori possono accedere e sovrascrivere i valori di proprietà impostati dai livelli superiori nella gerarchia.</span><span class="sxs-lookup"><span data-stu-id="34da4-119">Each level of the model has access to a common `Properties` collection, and lower levels can access and overwrite property values set by higher levels in the hierarchy.</span></span> <span data-ttu-id="34da4-120">Le proprietà vengono rese persistenti in `ActionDescriptor.Properties` quando vengono create le azioni.</span><span class="sxs-lookup"><span data-stu-id="34da4-120">The properties are persisted to the `ActionDescriptor.Properties` when the actions are created.</span></span> <span data-ttu-id="34da4-121">In seguito, quando viene gestita una richiesta, le proprietà aggiunte o modificate da una convenzione sono accessibili tramite `ActionContext.ActionDescriptor.Properties`.</span><span class="sxs-lookup"><span data-stu-id="34da4-121">Then when a request is being handled, any properties a convention added or modified can be accessed through `ActionContext.ActionDescriptor.Properties`.</span></span> <span data-ttu-id="34da4-122">L'uso delle proprietà è un metodo ideale per configurare filtri, strumenti di associazione modelli e altri elementi per ogni singola azione.</span><span class="sxs-lookup"><span data-stu-id="34da4-122">Using properties is a great way to configure your filters, model binders, etc. on a per-action basis.</span></span>

> [!NOTE]
> <span data-ttu-id="34da4-123">La raccolta `ActionDescriptor.Properties` non è thread-safe (per le operazioni di scrittura) una volta terminato l'avvio dell'app.</span><span class="sxs-lookup"><span data-stu-id="34da4-123">The `ActionDescriptor.Properties` collection isn't thread safe (for writes) once app startup has finished.</span></span> <span data-ttu-id="34da4-124">Le convenzioni sono il metodo migliore per aggiungere dati in modo sicuro a questa raccolta.</span><span class="sxs-lookup"><span data-stu-id="34da4-124">Conventions are the best way to safely add data to this collection.</span></span>

### <a name="iapplicationmodelprovider"></a><span data-ttu-id="34da4-125">IApplicationModelProvider</span><span class="sxs-lookup"><span data-stu-id="34da4-125">IApplicationModelProvider</span></span>

<span data-ttu-id="34da4-126">ASP.NET Core MVC carica il modello applicativo mediante un criterio provider definito dall'interfaccia [IApplicationModelProvider](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.iapplicationmodelprovider).</span><span class="sxs-lookup"><span data-stu-id="34da4-126">ASP.NET Core MVC loads the application model using a provider pattern, defined by the [IApplicationModelProvider](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.iapplicationmodelprovider) interface.</span></span> <span data-ttu-id="34da4-127">La presente sezione illustra alcuni dettagli di implementazione interna associati al funzionamento di questo provider.</span><span class="sxs-lookup"><span data-stu-id="34da4-127">This section covers some of the internal implementation details of how this provider functions.</span></span> <span data-ttu-id="34da4-128">Questo è un argomento avanzato: la maggior parte delle applicazioni usa il modello applicativo mediante le convenzioni.</span><span class="sxs-lookup"><span data-stu-id="34da4-128">This is an advanced topic - most apps that leverage the application model should do so by working with conventions.</span></span>

<span data-ttu-id="34da4-129">Le implementazioni dell'interfaccia `IApplicationModelProvider` eseguono il wrapping l'una sull'altra e ogni implementazione chiama `OnProvidersExecuting` in ordine crescente in base alla proprietà `Order` corrispondente.</span><span class="sxs-lookup"><span data-stu-id="34da4-129">Implementations of the `IApplicationModelProvider` interface "wrap" one another, with each implementation calling `OnProvidersExecuting` in ascending order based on its `Order` property.</span></span> <span data-ttu-id="34da4-130">Il metodo `OnProvidersExecuted` viene quindi chiamato in ordine inverso.</span><span class="sxs-lookup"><span data-stu-id="34da4-130">The `OnProvidersExecuted` method is then called in reverse order.</span></span> <span data-ttu-id="34da4-131">Il framework definisce diversi provider:</span><span class="sxs-lookup"><span data-stu-id="34da4-131">The framework defines several providers:</span></span>

<span data-ttu-id="34da4-132">First (`Order=-1000`):</span><span class="sxs-lookup"><span data-stu-id="34da4-132">First (`Order=-1000`):</span></span>

* [`DefaultApplicationModelProvider`](/dotnet/api/microsoft.aspnetcore.mvc.internal.defaultapplicationmodelprovider)

<span data-ttu-id="34da4-133">Then (`Order=-990`):</span><span class="sxs-lookup"><span data-stu-id="34da4-133">Then (`Order=-990`):</span></span>

* [`AuthorizationApplicationModelProvider`](/dotnet/api/microsoft.aspnetcore.mvc.internal.authorizationapplicationmodelprovider)
* [`CorsApplicationModelProvider`](/dotnet/api/microsoft.aspnetcore.mvc.cors.internal.corsapplicationmodelprovider)

> [!NOTE]
> <span data-ttu-id="34da4-134">L'ordine di chiamata di due provider con lo stesso valore di `Order` è indefinito e non va considerato affidabile.</span><span class="sxs-lookup"><span data-stu-id="34da4-134">The order in which two providers with the same value for `Order` are called is undefined, and therefore shouldn't be relied upon.</span></span>

> [!NOTE]
> <span data-ttu-id="34da4-135">`IApplicationModelProvider` è un concetto avanzato che gli autori di framework potranno estendere.</span><span class="sxs-lookup"><span data-stu-id="34da4-135">`IApplicationModelProvider` is an advanced concept for framework authors to extend.</span></span> <span data-ttu-id="34da4-136">In generale le app devono usare le convenzioni e i framework devono usare i provider.</span><span class="sxs-lookup"><span data-stu-id="34da4-136">In general, apps should use conventions and frameworks should use providers.</span></span> <span data-ttu-id="34da4-137">La differenza principale è il fatto che i provider vengono sempre eseguiti prima delle convenzioni.</span><span class="sxs-lookup"><span data-stu-id="34da4-137">The key distinction is that providers always run before conventions.</span></span>

<span data-ttu-id="34da4-138">`DefaultApplicationModelProvider` definisce molti comportamenti predefiniti usati da ASP.NET Core MVC.</span><span class="sxs-lookup"><span data-stu-id="34da4-138">The `DefaultApplicationModelProvider` establishes many of the default behaviors used by ASP.NET Core MVC.</span></span> <span data-ttu-id="34da4-139">Le responsabilità includono:</span><span class="sxs-lookup"><span data-stu-id="34da4-139">Its responsibilities include:</span></span>

* <span data-ttu-id="34da4-140">Aggiunta di filtri globali al contesto</span><span class="sxs-lookup"><span data-stu-id="34da4-140">Adding global filters to the context</span></span>
* <span data-ttu-id="34da4-141">Aggiunta di controller al contesto</span><span class="sxs-lookup"><span data-stu-id="34da4-141">Adding controllers to the context</span></span>
* <span data-ttu-id="34da4-142">Aggiunta di metodi del controller pubblici come azioni</span><span class="sxs-lookup"><span data-stu-id="34da4-142">Adding public controller methods as actions</span></span>
* <span data-ttu-id="34da4-143">Aggiunta di parametri del metodo di azione al contesto</span><span class="sxs-lookup"><span data-stu-id="34da4-143">Adding action method parameters to the context</span></span>
* <span data-ttu-id="34da4-144">Applicazione di route e altri attributi</span><span class="sxs-lookup"><span data-stu-id="34da4-144">Applying route and other attributes</span></span>

<span data-ttu-id="34da4-145">Alcuni comportamenti predefiniti vengono implementati da `DefaultApplicationModelProvider`.</span><span class="sxs-lookup"><span data-stu-id="34da4-145">Some built-in behaviors are implemented by the `DefaultApplicationModelProvider`.</span></span> <span data-ttu-id="34da4-146">Questo provider è responsabile della costruzione di [`ControllerModel`](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.controllermodel) , che a sua volta fa riferimento a istanze di, [`ActionModel`](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.actionmodel) [`PropertyModel`](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.propertymodel) e [`ParameterModel`](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.parametermodel) .</span><span class="sxs-lookup"><span data-stu-id="34da4-146">This provider is responsible for constructing the [`ControllerModel`](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.controllermodel), which in turn references [`ActionModel`](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.actionmodel), [`PropertyModel`](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.propertymodel), and [`ParameterModel`](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.parametermodel) instances.</span></span> <span data-ttu-id="34da4-147">La classe `DefaultApplicationModelProvider` è un dettaglio di implementazione interno del framework che verrà modificato in futuro.</span><span class="sxs-lookup"><span data-stu-id="34da4-147">The `DefaultApplicationModelProvider` class is an internal framework implementation detail that can and will change in the future.</span></span> 

<span data-ttu-id="34da4-148">`AuthorizationApplicationModelProvider` è responsabile dell'applicazione del comportamento associato agli attributi `AuthorizeFilter` e `AllowAnonymousFilter`.</span><span class="sxs-lookup"><span data-stu-id="34da4-148">The `AuthorizationApplicationModelProvider` is responsible for applying the behavior associated with the `AuthorizeFilter` and `AllowAnonymousFilter` attributes.</span></span> <span data-ttu-id="34da4-149">[Altre informazioni su questi attributi](xref:security/authorization/simple).</span><span class="sxs-lookup"><span data-stu-id="34da4-149">[Learn more about these attributes](xref:security/authorization/simple).</span></span>

<span data-ttu-id="34da4-150">`CorsApplicationModelProvider` implementa il comportamento associato a `IEnableCorsAttribute` e `IDisableCorsAttribute` e anche a `DisableCorsAuthorizationFilter`.</span><span class="sxs-lookup"><span data-stu-id="34da4-150">The `CorsApplicationModelProvider` implements behavior associated with the `IEnableCorsAttribute` and `IDisableCorsAttribute`, and the `DisableCorsAuthorizationFilter`.</span></span> <span data-ttu-id="34da4-151">[Altre informazioni su CORS](xref:security/cors).</span><span class="sxs-lookup"><span data-stu-id="34da4-151">[Learn more about CORS](xref:security/cors).</span></span>

## <a name="conventions"></a><span data-ttu-id="34da4-152">Convenzioni</span><span class="sxs-lookup"><span data-stu-id="34da4-152">Conventions</span></span>

<span data-ttu-id="34da4-153">Il modello applicativo definisce astrazioni di convenzioni che offrono un metodo di personalizzazione del comportamento dei modelli più semplice rispetto all'override dell'intero modello o provider.</span><span class="sxs-lookup"><span data-stu-id="34da4-153">The application model defines convention abstractions that provide a simpler way to customize the behavior of the models than overriding the entire model or provider.</span></span> <span data-ttu-id="34da4-154">Queste astrazioni sono il metodo consigliato per modificare il comportamento dell'app.</span><span class="sxs-lookup"><span data-stu-id="34da4-154">These abstractions are the recommended way to modify your app's behavior.</span></span> <span data-ttu-id="34da4-155">Le convenzioni offrono una modalità di scrittura del codice che applica in modo dinamico le personalizzazioni.</span><span class="sxs-lookup"><span data-stu-id="34da4-155">Conventions provide a way for you to write code that will dynamically apply customizations.</span></span> <span data-ttu-id="34da4-156">Sebbene i [filtri](xref:mvc/controllers/filters) offrano un mezzo per modificare il comportamento del Framework, le personalizzazioni consentono di controllare il funzionamento dell'intera applicazione.</span><span class="sxs-lookup"><span data-stu-id="34da4-156">While [filters](xref:mvc/controllers/filters) provide a means of modifying the framework's behavior, customizations let you control how the whole app works together.</span></span>

<span data-ttu-id="34da4-157">Sono disponibili le convenzioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="34da4-157">The following conventions are available:</span></span>

* [`IApplicationModelConvention`](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.iapplicationmodelconvention)
* [`IControllerModelConvention`](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.icontrollermodelconvention)
* [`IActionModelConvention`](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.iactionmodelconvention)
* [`IParameterModelConvention`](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.iparametermodelconvention)

<span data-ttu-id="34da4-158">Le convenzioni vengono applicate aggiungendole alle opzioni MVC o implementando `Attribute` e applicando i controlli a controller, azioni o parametri di azione (simile a [`Filters`](xref:mvc/controllers/filters) ).</span><span class="sxs-lookup"><span data-stu-id="34da4-158">Conventions are applied by adding them to MVC options or by implementing `Attribute`s and applying them to controllers, actions, or action parameters (similar to [`Filters`](xref:mvc/controllers/filters)).</span></span> <span data-ttu-id="34da4-159">A differenza dei filtri, le convenzioni vengono eseguite solo durante l'avvio dell'app e non come parte di ogni singola richiesta.</span><span class="sxs-lookup"><span data-stu-id="34da4-159">Unlike filters, conventions are only executed when the app is starting, not as part of each request.</span></span>

### <a name="sample-modifying-the-applicationmodel"></a><span data-ttu-id="34da4-160">Esempio: Modifica di ApplicationModel</span><span class="sxs-lookup"><span data-stu-id="34da4-160">Sample: Modifying the ApplicationModel</span></span>

<span data-ttu-id="34da4-161">La convenzione seguente viene usata per aggiungere una proprietà al modello applicativo.</span><span class="sxs-lookup"><span data-stu-id="34da4-161">The following convention is used to add a property to the application model.</span></span> 

[!code-csharp[](./application-model/sample/src/AppModelSample/Conventions/ApplicationDescription.cs)]

<span data-ttu-id="34da4-162">Le convenzioni del modello applicativo vengono applicate come opzioni quando MVC viene aggiunta a `ConfigureServices` in `Startup`.</span><span class="sxs-lookup"><span data-stu-id="34da4-162">Application model conventions are applied as options when MVC is added in `ConfigureServices` in `Startup`.</span></span>

[!code-csharp[](./application-model/sample/src/AppModelSample/Startup.cs?name=ConfigureServices&highlight=5)]

<span data-ttu-id="34da4-163">Le proprietà sono accessibili dalla raccolta di proprietà `ActionDescriptor` nelle azioni del controller:</span><span class="sxs-lookup"><span data-stu-id="34da4-163">Properties are accessible from the `ActionDescriptor` properties collection within controller actions:</span></span>

[!code-csharp[](./application-model/sample/src/AppModelSample/Controllers/AppModelController.cs?name=AppModelController)]

### <a name="sample-modifying-the-controllermodel-description"></a><span data-ttu-id="34da4-164">Esempio: Modifica della descrizione ControllerModel</span><span class="sxs-lookup"><span data-stu-id="34da4-164">Sample: Modifying the ControllerModel Description</span></span>

<span data-ttu-id="34da4-165">Come nell'esempio precedente, è anche possibile modificare il modello di controller in modo che includa proprietà personalizzate.</span><span class="sxs-lookup"><span data-stu-id="34da4-165">As in the previous example, the controller model can also be modified to include custom properties.</span></span> <span data-ttu-id="34da4-166">Queste sostituiranno le proprietà esistenti con lo stesso nome specificate nel modello applicativo.</span><span class="sxs-lookup"><span data-stu-id="34da4-166">These will override existing properties with the same name specified in the application model.</span></span> <span data-ttu-id="34da4-167">L'attributo di convenzione seguente aggiunge una descrizione a livello del controller:</span><span class="sxs-lookup"><span data-stu-id="34da4-167">The following convention attribute adds a description at the controller level:</span></span>

[!code-csharp[](./application-model/sample/src/AppModelSample/Conventions/ControllerDescriptionAttribute.cs)]

<span data-ttu-id="34da4-168">Questa convenzione viene applicata come attributo in un controller.</span><span class="sxs-lookup"><span data-stu-id="34da4-168">This convention is applied as an attribute on a controller.</span></span>

[!code-csharp[](./application-model/sample/src/AppModelSample/Controllers/DescriptionAttributesController.cs?name=ControllerDescription&highlight=1)]

<span data-ttu-id="34da4-169">L'accesso alla proprietà "description" avviene come negli esempi precedenti.</span><span class="sxs-lookup"><span data-stu-id="34da4-169">The "description" property is accessed in the same manner as in previous examples.</span></span>

### <a name="sample-modifying-the-actionmodel-description"></a><span data-ttu-id="34da4-170">Esempio: Modifica della descrizione ActionModel</span><span class="sxs-lookup"><span data-stu-id="34da4-170">Sample: Modifying the ActionModel Description</span></span>

<span data-ttu-id="34da4-171">È possibile applicare una convenzione di attributo distinta a singole azioni, eseguendo l'override del comportamento già applicato a livello dell'applicazione o del controller.</span><span class="sxs-lookup"><span data-stu-id="34da4-171">A separate attribute convention can be applied to individual actions, overriding behavior already applied at the application or controller level.</span></span>

[!code-csharp[](./application-model/sample/src/AppModelSample/Conventions/ActionDescriptionAttribute.cs)]

<span data-ttu-id="34da4-172">L'applicazione di questa convenzione a un'azione nel controller dell'esempio precedente visualizza come viene eseguito l'override della convenzione a livello di controller:</span><span class="sxs-lookup"><span data-stu-id="34da4-172">Applying this to an action within the previous example's controller demonstrates how it overrides the controller-level convention:</span></span>

[!code-csharp[](./application-model/sample/src/AppModelSample/Controllers/DescriptionAttributesController.cs?name=DescriptionAttributesController&highlight=9)]

### <a name="sample-modifying-the-parametermodel"></a><span data-ttu-id="34da4-173">Esempio: Modifica di ParameterModel</span><span class="sxs-lookup"><span data-stu-id="34da4-173">Sample: Modifying the ParameterModel</span></span>

<span data-ttu-id="34da4-174">È possibile applicare la convenzione seguente a parametri di azione per modificarne l'elemento `BindingInfo`.</span><span class="sxs-lookup"><span data-stu-id="34da4-174">The following convention can be applied to action parameters to modify their `BindingInfo`.</span></span> <span data-ttu-id="34da4-175">La convenzione seguente richiede che il parametro sia un parametro di route. Le altri origini di associazione possibili (ad esempio i valori stringa di query) vengono ignorati.</span><span class="sxs-lookup"><span data-stu-id="34da4-175">The following convention requires that the parameter be a route parameter; other potential binding sources (such as query string values) are ignored.</span></span>

[!code-csharp[](./application-model/sample/src/AppModelSample/Conventions/MustBeInRouteParameterModelConvention.cs)]

<span data-ttu-id="34da4-176">L'attributo può essere applicato a qualsiasi parametro di azione:</span><span class="sxs-lookup"><span data-stu-id="34da4-176">The attribute may be applied to any action parameter:</span></span>

[!code-csharp[](./application-model/sample/src/AppModelSample/Controllers/ParameterModelController.cs?name=ParameterModelController&highlight=5)]

### <a name="sample-modifying-the-actionmodel-name"></a><span data-ttu-id="34da4-177">Esempio: Modifica del nome di ActionModel</span><span class="sxs-lookup"><span data-stu-id="34da4-177">Sample: Modifying the ActionModel Name</span></span>

<span data-ttu-id="34da4-178">La convenzione seguente modifica `ActionModel` per aggiornare l'elemento *name* dell'azione alla quale viene applicata.</span><span class="sxs-lookup"><span data-stu-id="34da4-178">The following convention modifies the `ActionModel` to update the *name* of the action to which it's applied.</span></span> <span data-ttu-id="34da4-179">Il nuovo nome viene passato all'attributo come parametro.</span><span class="sxs-lookup"><span data-stu-id="34da4-179">The new name is provided as a parameter to the attribute.</span></span> <span data-ttu-id="34da4-180">Il nuovo nome viene usato in base al routing, pertanto ha effetto sulla route usata per raggiungere questo metodo di azione.</span><span class="sxs-lookup"><span data-stu-id="34da4-180">This new name is used by routing, so it will affect the route used to reach this action method.</span></span>

[!code-csharp[](./application-model/sample/src/AppModelSample/Conventions/CustomActionNameAttribute.cs)]

<span data-ttu-id="34da4-181">Questo attributo viene applicato a un metodo di azione in `HomeController`:</span><span class="sxs-lookup"><span data-stu-id="34da4-181">This attribute is applied to an action method in the `HomeController`:</span></span>

[!code-csharp[](./application-model/sample/src/AppModelSample/Controllers/HomeController.cs?name=ActionModelConvention&highlight=2)]

<span data-ttu-id="34da4-182">Anche se il nome del metodo è `SomeName` l'attributo esegue l'override della convenzione MVC che prevede l'uso del nome del metodo e sostituisce il nome dell'azione con `MyCoolAction`.</span><span class="sxs-lookup"><span data-stu-id="34da4-182">Even though the method name is `SomeName`, the attribute overrides the MVC convention of using the method name and replaces the action name with `MyCoolAction`.</span></span> <span data-ttu-id="34da4-183">Pertanto la route usata per raggiungere questa azione è `/Home/MyCoolAction`.</span><span class="sxs-lookup"><span data-stu-id="34da4-183">Thus, the route used to reach this action is `/Home/MyCoolAction`.</span></span>

> [!NOTE]
> <span data-ttu-id="34da4-184">Questo esempio ottiene in pratica lo stesso risultato dell'uso dell'attributo [ActionName](/dotnet/api/microsoft.aspnetcore.mvc.actionnameattribute) incorporato.</span><span class="sxs-lookup"><span data-stu-id="34da4-184">This example is essentially the same as using the built-in [ActionName](/dotnet/api/microsoft.aspnetcore.mvc.actionnameattribute) attribute.</span></span>

### <a name="sample-custom-routing-convention"></a><span data-ttu-id="34da4-185">Esempio: Convenzione per il routing personalizzato</span><span class="sxs-lookup"><span data-stu-id="34da4-185">Sample: Custom Routing Convention</span></span>

<span data-ttu-id="34da4-186">È possibile usare una convenzione `IApplicationModelConvention` per personalizzare il funzionamento del routing.</span><span class="sxs-lookup"><span data-stu-id="34da4-186">You can use an `IApplicationModelConvention` to customize how routing works.</span></span> <span data-ttu-id="34da4-187">Ad esempio la convenzione seguente incorpora gli spazi dei nomi dei controller nelle rispettive route, sostituendo `.` nello spazio dei nomi con `/` nella route:</span><span class="sxs-lookup"><span data-stu-id="34da4-187">For example, the following convention will incorporate Controllers' namespaces into their routes, replacing `.` in the namespace with `/` in the route:</span></span>

[!code-csharp[](./application-model/sample/src/AppModelSample/Conventions/NamespaceRoutingConvention.cs)]

<span data-ttu-id="34da4-188">La convenzione viene aggiunta come opzione in Startup.</span><span class="sxs-lookup"><span data-stu-id="34da4-188">The convention is added as an option in Startup.</span></span>

[!code-csharp[](./application-model/sample/src/AppModelSample/Startup.cs?name=ConfigureServices&highlight=6)]

> [!TIP]
> <span data-ttu-id="34da4-189">È possibile aggiungere convenzioni al [middleware](xref:fundamentals/middleware/index) accedendo a `MvcOptions` mediante `services.Configure<MvcOptions>(c => c.Conventions.Add(YOURCONVENTION));`.</span><span class="sxs-lookup"><span data-stu-id="34da4-189">You can add conventions to your [middleware](xref:fundamentals/middleware/index) by accessing `MvcOptions` using `services.Configure<MvcOptions>(c => c.Conventions.Add(YOURCONVENTION));`</span></span>

<span data-ttu-id="34da4-190">In questo esempio la convenzione viene applicata alle route che non usano il routing attributo in cui il nome del controller contiene "Namespace".</span><span class="sxs-lookup"><span data-stu-id="34da4-190">This sample applies this convention to routes that are not using attribute routing where the controller has  "Namespace" in its name.</span></span> <span data-ttu-id="34da4-191">Il controller seguente illustra questa convenzione:</span><span class="sxs-lookup"><span data-stu-id="34da4-191">The following controller demonstrates this convention:</span></span>

[!code-csharp[](./application-model/sample/src/AppModelSample/Controllers/NamespaceRoutingController.cs?highlight=7-8)]

## <a name="application-model-usage-in-webapicompatshim"></a><span data-ttu-id="34da4-192">Uso del modello applicativo in WebApiCompatShim</span><span class="sxs-lookup"><span data-stu-id="34da4-192">Application Model Usage in WebApiCompatShim</span></span>

<span data-ttu-id="34da4-193">ASP.NET Core MVC usa un set di convenzioni diverso rispetto all'API Web ASP.NET 2.</span><span class="sxs-lookup"><span data-stu-id="34da4-193">ASP.NET Core MVC uses a different set of conventions from ASP.NET Web API 2.</span></span> <span data-ttu-id="34da4-194">Con le convenzioni personalizzate è possibile modificare il comportamento di un'app ASP.NET Core MVC per renderlo coerente con quello di un'app API Web.</span><span class="sxs-lookup"><span data-stu-id="34da4-194">Using custom conventions, you can modify an ASP.NET Core MVC app's behavior to be consistent with that of a Web API app.</span></span> <span data-ttu-id="34da4-195">Microsoft offre [WebApiCompatShim](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.WebApiCompatShim/) per questo scopo specifico.</span><span class="sxs-lookup"><span data-stu-id="34da4-195">Microsoft ships the [WebApiCompatShim](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.WebApiCompatShim/) specifically for this purpose.</span></span>

> [!NOTE]
> <span data-ttu-id="34da4-196">Altre informazioni sulla [migrazione dall'API Web ASP.NET](xref:migration/webapi).</span><span class="sxs-lookup"><span data-stu-id="34da4-196">Learn more about [migration from ASP.NET Web API](xref:migration/webapi).</span></span>

<span data-ttu-id="34da4-197">Per usare lo shim Web API Compatibility è necessario aggiungere il pacchetto al progetto e quindi aggiungere le convenzioni a MVC chiamando `AddWebApiConventions` in `Startup`:</span><span class="sxs-lookup"><span data-stu-id="34da4-197">To use the Web API Compatibility Shim, you need to add the package to your project and then add the conventions to MVC by calling `AddWebApiConventions` in `Startup`:</span></span>

```csharp
services.AddMvc().AddWebApiConventions();
```

<span data-ttu-id="34da4-198">Le convenzioni offerte dallo shim vengono applicate solo alle parti dell'app a cui sono applicati determinati attributi.</span><span class="sxs-lookup"><span data-stu-id="34da4-198">The conventions provided by the shim are only applied to parts of the app that have had certain attributes applied to them.</span></span> <span data-ttu-id="34da4-199">I quattro attributi seguenti vengono usati per determinare i controller le cui convenzioni verranno modificate dalle convenzioni dello shim:</span><span class="sxs-lookup"><span data-stu-id="34da4-199">The following four attributes are used to control which controllers should have their conventions modified by the shim's conventions:</span></span>

* [<span data-ttu-id="34da4-200">UseWebApiActionConventionsAttribute</span><span class="sxs-lookup"><span data-stu-id="34da4-200">UseWebApiActionConventionsAttribute</span></span>](/dotnet/api/microsoft.aspnetcore.mvc.webapicompatshim.usewebapiactionconventionsattribute)
* [<span data-ttu-id="34da4-201">UseWebApiOverloadingAttribute</span><span class="sxs-lookup"><span data-stu-id="34da4-201">UseWebApiOverloadingAttribute</span></span>](/dotnet/api/microsoft.aspnetcore.mvc.webapicompatshim.usewebapioverloadingattribute)
* [<span data-ttu-id="34da4-202">UseWebApiParameterConventionsAttribute</span><span class="sxs-lookup"><span data-stu-id="34da4-202">UseWebApiParameterConventionsAttribute</span></span>](/dotnet/api/microsoft.aspnetcore.mvc.webapicompatshim.usewebapiparameterconventionsattribute)
* [<span data-ttu-id="34da4-203">UseWebApiRoutesAttribute</span><span class="sxs-lookup"><span data-stu-id="34da4-203">UseWebApiRoutesAttribute</span></span>](/dotnet/api/microsoft.aspnetcore.mvc.webapicompatshim.usewebapiroutesattribute)

### <a name="action-conventions"></a><span data-ttu-id="34da4-204">Convenzioni per le azioni</span><span class="sxs-lookup"><span data-stu-id="34da4-204">Action Conventions</span></span>

<span data-ttu-id="34da4-205">`UseWebApiActionConventionsAttribute` viene usato per eseguire il mapping del metodo HTTP alle azioni in base al nome (ad esempio `Get` esegue il mapping a `HttpGet`).</span><span class="sxs-lookup"><span data-stu-id="34da4-205">The `UseWebApiActionConventionsAttribute` is used to map the HTTP method to actions based on their name (for instance, `Get` would map to `HttpGet`).</span></span> <span data-ttu-id="34da4-206">Si applica solo alle azioni che non usano il routing degli attributi.</span><span class="sxs-lookup"><span data-stu-id="34da4-206">It only applies to actions that don't use attribute routing.</span></span>

### <a name="overloading"></a><span data-ttu-id="34da4-207">Overload</span><span class="sxs-lookup"><span data-stu-id="34da4-207">Overloading</span></span>

<span data-ttu-id="34da4-208">`UseWebApiOverloadingAttribute` viene usato per applicare la convenzione `WebApiOverloadingApplicationModelConvention`.</span><span class="sxs-lookup"><span data-stu-id="34da4-208">The `UseWebApiOverloadingAttribute` is used to apply the `WebApiOverloadingApplicationModelConvention` convention.</span></span> <span data-ttu-id="34da4-209">Questa convenzione aggiunge un elemento `OverloadActionConstraint` al processo di selezione azioni, che limita le azioni candidate a quelle per cui la richiesta soddisfa tutti i parametri non facoltativi.</span><span class="sxs-lookup"><span data-stu-id="34da4-209">This convention adds an `OverloadActionConstraint` to the action selection process, which limits candidate actions to those for which the request satisfies all non-optional parameters.</span></span>

### <a name="parameter-conventions"></a><span data-ttu-id="34da4-210">Convenzioni dei parametri</span><span class="sxs-lookup"><span data-stu-id="34da4-210">Parameter Conventions</span></span>

<span data-ttu-id="34da4-211">`UseWebApiParameterConventionsAttribute` viene usato per applicare la convenzione di azione `WebApiParameterConventionsApplicationModelConvention`.</span><span class="sxs-lookup"><span data-stu-id="34da4-211">The `UseWebApiParameterConventionsAttribute` is used to apply the `WebApiParameterConventionsApplicationModelConvention` action convention.</span></span> <span data-ttu-id="34da4-212">Questa convenzione specifica che i tipi semplici usati come parametri di azione sono associati dall'URI per impostazione predefinita, mentre i tipi complessi sono associati dal corpo della richiesta.</span><span class="sxs-lookup"><span data-stu-id="34da4-212">This convention specifies that simple types used as action parameters are bound from the URI by default, while complex types are bound from the request body.</span></span>

### <a name="routes"></a><span data-ttu-id="34da4-213">Route</span><span class="sxs-lookup"><span data-stu-id="34da4-213">Routes</span></span>

<span data-ttu-id="34da4-214">`UseWebApiRoutesAttribute` controlla se la convenzione del controller `WebApiApplicationModelConvention` viene applicata.</span><span class="sxs-lookup"><span data-stu-id="34da4-214">The `UseWebApiRoutesAttribute` controls whether the `WebApiApplicationModelConvention` controller convention is applied.</span></span> <span data-ttu-id="34da4-215">Quando è abilitata, questa convenzione viene usata per aggiungere alla route il supporto per le [aree](xref:mvc/controllers/areas).</span><span class="sxs-lookup"><span data-stu-id="34da4-215">When enabled, this convention is used to add support for [areas](xref:mvc/controllers/areas) to the route.</span></span>

<span data-ttu-id="34da4-216">Oltre a un set di convenzioni, il pacchetto di compatibilità include una classe base `System.Web.Http.ApiController` che sostituisce quella offerta dall'API Web.</span><span class="sxs-lookup"><span data-stu-id="34da4-216">In addition to a set of conventions, the compatibility package includes a `System.Web.Http.ApiController` base class that replaces the one provided by Web API.</span></span> <span data-ttu-id="34da4-217">In questo modo i controller creati per l'API Web che ereditano da `ApiController` dell'API possono funzionare come sono stati progettati, anche quando vengono eseguiti in ASP.NET Core MVC.</span><span class="sxs-lookup"><span data-stu-id="34da4-217">This allows your controllers written for Web API and inheriting from its `ApiController` to work as they were designed, while running on ASP.NET Core MVC.</span></span> <span data-ttu-id="34da4-218">Tutti gli `UseWebApi*` attributi elencati in precedenza vengono applicati alla classe controller di base.</span><span class="sxs-lookup"><span data-stu-id="34da4-218">All of the `UseWebApi*` attributes listed earlier are applied to the base controller class.</span></span> <span data-ttu-id="34da4-219">`ApiController` espone le proprietà, i metodi e i tipi di risultati compatibili con quelli disponibili nell'API Web.</span><span class="sxs-lookup"><span data-stu-id="34da4-219">The `ApiController` exposes properties, methods, and result types that are compatible with those found in Web API.</span></span>

## <a name="using-apiexplorer-to-document-your-app"></a><span data-ttu-id="34da4-220">Uso di ApiExplorer per documentare l'app</span><span class="sxs-lookup"><span data-stu-id="34da4-220">Using ApiExplorer to Document Your App</span></span>

<span data-ttu-id="34da4-221">Il modello applicativo espone una [`ApiExplorer`](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.apiexplorermodel) proprietà a ogni livello che può essere usata per attraversare la struttura dell'app.</span><span class="sxs-lookup"><span data-stu-id="34da4-221">The application model exposes an [`ApiExplorer`](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.apiexplorermodel) property at each level that can be used to traverse the app's structure.</span></span> <span data-ttu-id="34da4-222">Questa funzionalità può essere usata per [generare pagine della Guida per le API Web usando strumenti come Swagger](xref:tutorials/web-api-help-pages-using-swagger).</span><span class="sxs-lookup"><span data-stu-id="34da4-222">This can be used to [generate help pages for your Web APIs using tools like Swagger](xref:tutorials/web-api-help-pages-using-swagger).</span></span> <span data-ttu-id="34da4-223">La proprietà `ApiExplorer` espone una proprietà `IsVisible` che può essere impostata per specificare quali parti del modello dell'app devono essere esposte.</span><span class="sxs-lookup"><span data-stu-id="34da4-223">The `ApiExplorer` property exposes an `IsVisible` property that can be set to specify which parts of your app's model should be exposed.</span></span> <span data-ttu-id="34da4-224">È possibile configurare questa impostazione usando una convenzione:</span><span class="sxs-lookup"><span data-stu-id="34da4-224">You can configure this setting using a convention:</span></span>

[!code-csharp[](./application-model/sample/src/AppModelSample/Conventions/EnableApiExplorerApplicationConvention.cs)]

<span data-ttu-id="34da4-225">Usando questo approccio (e se necessario convenzioni aggiuntive), è possibile attivare o disattivare la visibilità delle API a qualsiasi livello all'interno dell'app.</span><span class="sxs-lookup"><span data-stu-id="34da4-225">Using this approach (and additional conventions if required), you can enable or disable API visibility at any level within your app.</span></span> 
