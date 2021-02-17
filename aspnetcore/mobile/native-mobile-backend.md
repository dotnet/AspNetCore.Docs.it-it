---
title: Creare servizi back-end per app native per dispositivi mobili con ASP.NET Core
author: rick-anderson
description: Informazioni su come creare servizi back-end mediante ASP.NET Core MVC per il supporto di app per dispositivi mobili native.
ms.author: riande
ms.date: 2/12/2021
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
uid: mobile/native-mobile-backend
ms.openlocfilehash: e496b7811cc534b6f0f6dfdb857f6e462b38049e
ms.sourcegitcommit: f77a7467651bab61b24261da9dc5c1dd75fc1fa9
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/17/2021
ms.locfileid: "100564024"
---
# <a name="create-backend-services-for-native-mobile-apps-with-aspnet-core"></a><span data-ttu-id="20c82-103">Creare servizi back-end per app native per dispositivi mobili con ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="20c82-103">Create backend services for native mobile apps with ASP.NET Core</span></span>

<span data-ttu-id="20c82-104">Di [James Montemagno su](https://twitter.com/JamesMontemagno)</span><span class="sxs-lookup"><span data-stu-id="20c82-104">By [James Montemagno](https://twitter.com/JamesMontemagno)</span></span>

<span data-ttu-id="20c82-105">Le app per dispositivi mobili possono comunicare con i servizi back-end di ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="20c82-105">Mobile apps can communicate with ASP.NET Core backend services.</span></span> <span data-ttu-id="20c82-106">Per istruzioni sulla connessione a servizi Web locali da simulatori iOS ed emulatori Android, vedere [Connettersi a servizi Web locali da simulatori iOS ed emulatori Android](/xamarin/cross-platform/deploy-test/connect-to-local-web-services).</span><span class="sxs-lookup"><span data-stu-id="20c82-106">For instructions on connecting local web services from iOS simulators and Android emulators, see [Connect to Local Web Services from iOS Simulators and Android Emulators](/xamarin/cross-platform/deploy-test/connect-to-local-web-services).</span></span>

[<span data-ttu-id="20c82-107">Visualizzare o scaricare codice di esempio di servizi back-end</span><span class="sxs-lookup"><span data-stu-id="20c82-107">View or download sample backend services code</span></span>](https://github.com/xamarin/xamarin-forms-samples/tree/master/WebServices/TodoREST)

## <a name="the-sample-native-mobile-app"></a><span data-ttu-id="20c82-108">App per dispositivi mobili nativa di esempio</span><span class="sxs-lookup"><span data-stu-id="20c82-108">The Sample Native Mobile App</span></span>

<span data-ttu-id="20c82-109">Questa esercitazione illustra come creare servizi back-end usando ASP.NET Core per supportare le app per dispositivi mobili native.</span><span class="sxs-lookup"><span data-stu-id="20c82-109">This tutorial demonstrates how to create backend services using ASP.NET Core to support native mobile apps.</span></span> <span data-ttu-id="20c82-110">Usa l' [app Novell. Forms Todorer](/xamarin/xamarin-forms/data-cloud/consuming/rest) come client nativo, che include client nativi distinti per Android, iOS e Windows.</span><span class="sxs-lookup"><span data-stu-id="20c82-110">It uses the [Xamarin.Forms TodoRest app](/xamarin/xamarin-forms/data-cloud/consuming/rest) as its native client, which includes separate native clients for Android, iOS, and Windows.</span></span> <span data-ttu-id="20c82-111">È possibile eseguire l'esercitazione collegata per creare l'app nativa (e installare gli strumenti Xamarin gratuiti necessari), nonché scaricare la soluzione di esempio di Xamarin.</span><span class="sxs-lookup"><span data-stu-id="20c82-111">You can follow the linked tutorial to create the native app (and install the necessary free Xamarin tools), as well as download the Xamarin sample solution.</span></span> <span data-ttu-id="20c82-112">L'esempio Novell include un progetto di servizi API Web ASP.NET Core, che viene sostituito dall'app ASP.NET Core di questo articolo, senza alcuna modifica richiesta dal client.</span><span class="sxs-lookup"><span data-stu-id="20c82-112">The Xamarin sample includes an ASP.NET Core Web API services project, which this article's ASP.NET Core app replaces (with no changes required by the client).</span></span>

![Applicazione ToDoRest in esecuzione su uno smartphone Android](native-mobile-backend/_static/todo-android.png)

### <a name="features"></a><span data-ttu-id="20c82-114">Funzionalità</span><span class="sxs-lookup"><span data-stu-id="20c82-114">Features</span></span>

<span data-ttu-id="20c82-115">L' [app Todorer](https://github.com/xamarin/xamarin-forms-samples/tree/master/WebServices/TodoREST) supporta l'inserimento, l'aggiunta, l'eliminazione e l'aggiornamento di To-Do elementi.</span><span class="sxs-lookup"><span data-stu-id="20c82-115">The [TodoREST app](https://github.com/xamarin/xamarin-forms-samples/tree/master/WebServices/TodoREST) supports listing, adding, deleting, and updating To-Do items.</span></span> <span data-ttu-id="20c82-116">Ogni elemento include un ID, un nome, delle note e una proprietà che indica se è stato eseguito.</span><span class="sxs-lookup"><span data-stu-id="20c82-116">Each item has an ID, a Name, Notes, and a property indicating whether it's been Done yet.</span></span>

<span data-ttu-id="20c82-117">La visualizzazione principale degli elementi, che appare nell'immagine in alto, contiene un elenco con il nome di ogni elemento. Un segno di spunta indica se l'elemento è stato eseguito.</span><span class="sxs-lookup"><span data-stu-id="20c82-117">The main view of the items, as shown above, lists each item's name and indicates if it's done with a checkmark.</span></span>

<span data-ttu-id="20c82-118">Se si tocca l'icona `+` viene visualizzata una finestra di dialogo per l'aggiunta di elementi:</span><span class="sxs-lookup"><span data-stu-id="20c82-118">Tapping the `+` icon opens an add item dialog:</span></span>

![Finestra di dialogo per l'aggiunta di elementi](native-mobile-backend/_static/todo-android-new-item.png)

<span data-ttu-id="20c82-120">Se si tocca un elemento sulla schermata dell'elenco principale viene visualizzata una finestra di dialogo di modifica in cui è possibile modificare le opzioni Name (Nome), Notes (Note) e Done (Fatto) dell'elemento oppure eliminare l'elemento stesso:</span><span class="sxs-lookup"><span data-stu-id="20c82-120">Tapping an item on the main list screen opens up an edit dialog where the item's Name, Notes, and Done settings can be modified, or the item can be deleted:</span></span>

![Finestra di dialogo di modifica dell'elemento](native-mobile-backend/_static/todo-android-edit-item.png)

<span data-ttu-id="20c82-122">Per testarlo in base all'app ASP.NET Core creata nella sezione successiva in esecuzione nel computer, aggiornare la costante dell'app [`RestUrl`](https://github.com/xamarin/xamarin-forms-samples/blob/master/WebServices/TodoREST/TodoREST/Constants.cs#L13) .</span><span class="sxs-lookup"><span data-stu-id="20c82-122">To test it out yourself against the ASP.NET Core app created in the next section running on your computer, update the app's [`RestUrl`](https://github.com/xamarin/xamarin-forms-samples/blob/master/WebServices/TodoREST/TodoREST/Constants.cs#L13) constant.</span></span>

<span data-ttu-id="20c82-123">Gli emulatori Android non vengono eseguiti nel computer locale e usano un IP di loopback (10.0.2.2) per comunicare con il computer locale.</span><span class="sxs-lookup"><span data-stu-id="20c82-123">Android emulators do not run on the local machine and use a loopback IP (10.0.2.2) to communicate with the local machine.</span></span> <span data-ttu-id="20c82-124">Usare [Novell. Essentials DeviceInfo](/xamarin/essentials/device-information/) per rilevare le operazioni eseguite dal sistema per usare l'URL corretto.</span><span class="sxs-lookup"><span data-stu-id="20c82-124">Leverage [Xamarin.Essentials DeviceInfo](/xamarin/essentials/device-information/) to detect what operating the system is running to use the correct URL.</span></span>

<span data-ttu-id="20c82-125">Passare al [`TodoREST`](https://github.com/xamarin/xamarin-forms-samples/tree/master/WebServices/TodoREST/TodoREST) progetto e aprire il [`Constants.cs`](https://github.com/xamarin/xamarin-forms-samples/blob/master/WebServices/TodoREST/TodoREST/Constants.cs) file.</span><span class="sxs-lookup"><span data-stu-id="20c82-125">Navigate to the [`TodoREST`](https://github.com/xamarin/xamarin-forms-samples/tree/master/WebServices/TodoREST/TodoREST) project and open the [`Constants.cs`](https://github.com/xamarin/xamarin-forms-samples/blob/master/WebServices/TodoREST/TodoREST/Constants.cs) file.</span></span> <span data-ttu-id="20c82-126">Il file *Constants.cs* contiene la configurazione seguente.</span><span class="sxs-lookup"><span data-stu-id="20c82-126">The *Constants.cs* file contains the following configuration.</span></span>

:::code language="csharp" source="~/../xamarin-forms-samples/WebServices/TodoREST/TodoREST/Constants.cs" highlight="13":::

<span data-ttu-id="20c82-127">Facoltativamente, è possibile distribuire il servizio Web in un servizio cloud, ad esempio Azure, e aggiornare il `RestUrl` .</span><span class="sxs-lookup"><span data-stu-id="20c82-127">You can optionally deploy the web service to a cloud service such as Azure and update the `RestUrl`.</span></span>

## <a name="creating-the-aspnet-core-project"></a><span data-ttu-id="20c82-128">Creazione del progetto ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="20c82-128">Creating the ASP.NET Core Project</span></span>

<span data-ttu-id="20c82-129">Creare una nuova applicazione Web ASP.NET Core in Visual Studio.</span><span class="sxs-lookup"><span data-stu-id="20c82-129">Create a new ASP.NET Core Web Application in Visual Studio.</span></span> <span data-ttu-id="20c82-130">Scegliere il modello API Web.</span><span class="sxs-lookup"><span data-stu-id="20c82-130">Choose the Web API template.</span></span> <span data-ttu-id="20c82-131">Denominare il progetto *TodoAPI*.</span><span class="sxs-lookup"><span data-stu-id="20c82-131">Name the project *TodoAPI*.</span></span>

![Finestra di dialogo Nuova applicazione Web ASP.NET Core con il modello di progetto API Web selezionato](native-mobile-backend/_static/web-api-template.png)

<span data-ttu-id="20c82-133">L'app deve rispondere a tutte le richieste effettuate alla porta 5000, incluso il traffico http Clear-Text per il client mobile.</span><span class="sxs-lookup"><span data-stu-id="20c82-133">The app should respond to all requests made to port 5000 including clear-text http traffic for our mobile client.</span></span> <span data-ttu-id="20c82-134">Aggiornare *Startup.cs* <xref:Microsoft.AspNetCore.Builder.HttpsPolicyBuilderExtensions.UseHttpsRedirection%2A> , quindi non viene eseguito in fase di sviluppo:</span><span class="sxs-lookup"><span data-stu-id="20c82-134">Update *Startup.cs* so <xref:Microsoft.AspNetCore.Builder.HttpsPolicyBuilderExtensions.UseHttpsRedirection%2A> doesn't run in development:</span></span>

:::code language="csharp" source="~/../xamarin-forms-samples/WebServices/TodoREST/TodoAPI/TodoAPI/Startup.cs" id="snippet" highlight="7-11":::

> [!NOTE]
> <span data-ttu-id="20c82-135">Eseguire l'app direttamente, anziché dietro IIS Express.</span><span class="sxs-lookup"><span data-stu-id="20c82-135">Run the app directly, rather than behind IIS Express.</span></span> <span data-ttu-id="20c82-136">Per impostazione predefinita, IIS Express ignora le richieste non locali.</span><span class="sxs-lookup"><span data-stu-id="20c82-136">IIS Express ignores non-local requests by default.</span></span> <span data-ttu-id="20c82-137">Eseguire [DotNet Run](/dotnet/core/tools/dotnet-run) da un prompt dei comandi o scegliere il profilo nome app dall'elenco a discesa destinazione di debug nella barra degli strumenti di Visual Studio.</span><span class="sxs-lookup"><span data-stu-id="20c82-137">Run [dotnet run](/dotnet/core/tools/dotnet-run) from a command prompt, or choose the app name profile from the Debug Target dropdown in the Visual Studio toolbar.</span></span>

<span data-ttu-id="20c82-138">Aggiungere una classe modello che rappresenti gli elementi attività.</span><span class="sxs-lookup"><span data-stu-id="20c82-138">Add a model class to represent To-Do items.</span></span> <span data-ttu-id="20c82-139">Contrassegnare i campi obbligatori con l' `[Required]` attributo:</span><span class="sxs-lookup"><span data-stu-id="20c82-139">Mark required fields with the `[Required]` attribute:</span></span>

:::code language="csharp" source="~/../xamarin-forms-samples/WebServices/TodoREST/TodoAPI/TodoAPI/Models/TodoItem.cs":::

<span data-ttu-id="20c82-140">I metodi dell'API richiedono un approccio per l'uso dei dati.</span><span class="sxs-lookup"><span data-stu-id="20c82-140">The API methods require some way to work with data.</span></span> <span data-ttu-id="20c82-141">Usare la stessa interfaccia `ITodoRepository` usata nell'esempio originale di Xamarin:</span><span class="sxs-lookup"><span data-stu-id="20c82-141">Use the same `ITodoRepository` interface the original Xamarin sample uses:</span></span>

:::code language="csharp" source="~/../xamarin-forms-samples/WebServices/TodoREST/TodoAPI/TodoAPI/Interfaces/ITodoRepository.cs":::

<span data-ttu-id="20c82-142">Per questo esempio, l'implementazione usa solo una raccolta privata di elementi:</span><span class="sxs-lookup"><span data-stu-id="20c82-142">For this sample, the implementation just uses a private collection of items:</span></span>

:::code language="csharp" source="~/../xamarin-forms-samples/WebServices/TodoREST/TodoAPI/TodoAPI/Services/TodoRepository.cs":::

<span data-ttu-id="20c82-143">Configurare l'implementazione in *Startup.cs*:</span><span class="sxs-lookup"><span data-stu-id="20c82-143">Configure the implementation in *Startup.cs*:</span></span>

:::code language="csharp" source="~/../xamarin-forms-samples/WebServices/TodoREST/TodoAPI/TodoAPI/Startup.cs" id="snippet2" highlight="3":::

## <a name="creating-the-controller"></a><span data-ttu-id="20c82-144">Creazione del controller</span><span class="sxs-lookup"><span data-stu-id="20c82-144">Creating the Controller</span></span>

<span data-ttu-id="20c82-145">Aggiungere un nuovo controller al progetto [TodoItemsController](https://github.com/xamarin/xamarin-forms-samples/tree/master/WebServices/TodoREST/TodoAPI/TodoAPI/Controllers/TodoItemsController.cs).</span><span class="sxs-lookup"><span data-stu-id="20c82-145">Add a new controller to the project, [TodoItemsController](https://github.com/xamarin/xamarin-forms-samples/tree/master/WebServices/TodoREST/TodoAPI/TodoAPI/Controllers/TodoItemsController.cs).</span></span> <span data-ttu-id="20c82-146">Deve ereditare da <xref:Microsoft.AspNetCore.Mvc.ControllerBase> .</span><span class="sxs-lookup"><span data-stu-id="20c82-146">It should inherit from <xref:Microsoft.AspNetCore.Mvc.ControllerBase>.</span></span> <span data-ttu-id="20c82-147">Aggiungere un attributo `Route` per indicare che il controller gestirà le richieste inoltrate ai percorsi che iniziano con `api/todoitems`.</span><span class="sxs-lookup"><span data-stu-id="20c82-147">Add a `Route` attribute to indicate that the controller will handle requests made to paths starting with `api/todoitems`.</span></span> <span data-ttu-id="20c82-148">Il token `[controller]` nella route viene sostituito dal nome del controller (omettendo il suffisso `Controller`) ed è particolarmente utile per le route globali.</span><span class="sxs-lookup"><span data-stu-id="20c82-148">The `[controller]` token in the route is replaced by the name of the controller (omitting the `Controller` suffix), and is especially helpful for global routes.</span></span> <span data-ttu-id="20c82-149">Altre informazioni sul [routing](../fundamentals/routing.md).</span><span class="sxs-lookup"><span data-stu-id="20c82-149">Learn more about [routing](../fundamentals/routing.md).</span></span>

<span data-ttu-id="20c82-150">Per funzionare, il controller richiede un `ITodoRepository`. Richiedere un'istanza di questo tipo tramite il costruttore del controller.</span><span class="sxs-lookup"><span data-stu-id="20c82-150">The controller requires an `ITodoRepository` to function; request an instance of this type through the controller's constructor.</span></span> <span data-ttu-id="20c82-151">In fase di esecuzione, questa istanza viene resa disponibile tramite il supporto dell'[inserimento di dipendenze](../fundamentals/dependency-injection.md) specificato dal framework.</span><span class="sxs-lookup"><span data-stu-id="20c82-151">At runtime, this instance will be provided using the framework's support for [dependency injection](../fundamentals/dependency-injection.md).</span></span>

:::code language="csharp" source="~/../xamarin-forms-samples/WebServices/TodoREST/TodoAPI/TodoAPI/Controllers/TodoItemsController.cs" id="snippetDI":::

<span data-ttu-id="20c82-152">Questa API supporta quattro verbi HTTP diversi per eseguire operazioni CRUD (Create, Read, Update, Delete - Creazione, Lettura, Aggiornamento, Eliminazione) nell'origine dati.</span><span class="sxs-lookup"><span data-stu-id="20c82-152">This API supports four different HTTP verbs to perform CRUD (Create, Read, Update, Delete) operations on the data source.</span></span> <span data-ttu-id="20c82-153">L'operazione più semplice tra queste è Read, che corrisponde a una richiesta HTTP GET.</span><span class="sxs-lookup"><span data-stu-id="20c82-153">The simplest of these is the Read operation, which corresponds to an HTTP GET request.</span></span>

### <a name="reading-items"></a><span data-ttu-id="20c82-154">Lettura degli elementi</span><span class="sxs-lookup"><span data-stu-id="20c82-154">Reading Items</span></span>

<span data-ttu-id="20c82-155">La richiesta di un elenco di elementi viene eseguita con una richiesta GET al metodo `List`.</span><span class="sxs-lookup"><span data-stu-id="20c82-155">Requesting a list of items is done with a GET request to the `List` method.</span></span> <span data-ttu-id="20c82-156">L'attributo `[HttpGet]` del metodo `List` indica che questa azione deve gestire solo le richieste GET.</span><span class="sxs-lookup"><span data-stu-id="20c82-156">The `[HttpGet]` attribute on the `List` method indicates that this action should only handle GET requests.</span></span> <span data-ttu-id="20c82-157">La route di questa azione è la route specificata nel controller.</span><span class="sxs-lookup"><span data-stu-id="20c82-157">The route for this action is the route specified on the controller.</span></span> <span data-ttu-id="20c82-158">Non è necessario usare il nome dell'azione come parte della route.</span><span class="sxs-lookup"><span data-stu-id="20c82-158">You don't necessarily need to use the action name as part of the route.</span></span> <span data-ttu-id="20c82-159">È sufficiente garantire che ogni azione disponga di una route univoca e non ambigua.</span><span class="sxs-lookup"><span data-stu-id="20c82-159">You just need to ensure each action has a unique and unambiguous route.</span></span> <span data-ttu-id="20c82-160">Gli attributi di routing possono essere applicati sia a livello di controller che di metodo per definire route specifiche.</span><span class="sxs-lookup"><span data-stu-id="20c82-160">Routing attributes can be applied at both the controller and method levels to build up specific routes.</span></span>

:::code language="csharp" source="~/../xamarin-forms-samples/WebServices/TodoREST/TodoAPI/TodoAPI/Controllers/TodoItemsController.cs" id="snippet":::

<span data-ttu-id="20c82-161">Il `List` metodo restituisce un codice di risposta 200 OK e tutti gli elementi todo, serializzati come JSON.</span><span class="sxs-lookup"><span data-stu-id="20c82-161">The `List` method returns a 200 OK response code and all of the Todo items, serialized as JSON.</span></span>

<span data-ttu-id="20c82-162">È possibile eseguire il test del nuovo metodo API usando un'ampia gamma di strumenti, ad esempio [Postman](https://www.getpostman.com/docs/), visualizzato di seguito:</span><span class="sxs-lookup"><span data-stu-id="20c82-162">You can test your new API method using a variety of tools, such as [Postman](https://www.getpostman.com/docs/), shown here:</span></span>

![Console di Postman con una richiesta GET per elementi di attività (todoitems) e il corpo della risposta con JSON per tre elementi restituiti](native-mobile-backend/_static/postman-get.png)

### <a name="creating-items"></a><span data-ttu-id="20c82-164">Creazione di elementi</span><span class="sxs-lookup"><span data-stu-id="20c82-164">Creating Items</span></span>

<span data-ttu-id="20c82-165">Per convenzione, la creazione di nuovi elementi di dati viene associata al verbo HTTP POST.</span><span class="sxs-lookup"><span data-stu-id="20c82-165">By convention, creating new data items is mapped to the HTTP POST verb.</span></span> <span data-ttu-id="20c82-166">Al `Create` metodo è `[HttpPost]` applicato un attributo e viene accettata un' `TodoItem` istanza.</span><span class="sxs-lookup"><span data-stu-id="20c82-166">The `Create` method has an `[HttpPost]` attribute applied to it and accepts a `TodoItem` instance.</span></span> <span data-ttu-id="20c82-167">Poiché l' `item` argomento viene passato nel corpo del post, questo parametro specifica l' `[FromBody]` attributo.</span><span class="sxs-lookup"><span data-stu-id="20c82-167">Since the `item` argument is passed in the body of the POST, this parameter specifies the `[FromBody]` attribute.</span></span>

<span data-ttu-id="20c82-168">All'interno del metodo, l'elemento viene verificato per validità ed esistenza precedente nell'archivio dati. Se non si verificano problemi, l'elemento viene aggiunto tramite il repository.</span><span class="sxs-lookup"><span data-stu-id="20c82-168">Inside the method, the item is checked for validity and prior existence in the data store, and if no issues occur, it's added using the repository.</span></span> <span data-ttu-id="20c82-169">La verifica `ModelState.IsValid` esegue la [convalida del modello](../mvc/models/validation.md) e deve essere eseguita in ogni metodo dell'API che accetta input utente.</span><span class="sxs-lookup"><span data-stu-id="20c82-169">Checking `ModelState.IsValid` performs [model validation](../mvc/models/validation.md), and should be done in every API method that accepts user input.</span></span>

:::code language="csharp" source="~/../xamarin-forms-samples/WebServices/TodoREST/TodoAPI/TodoAPI/Controllers/TodoItemsController.cs" id="snippetCreate":::

<span data-ttu-id="20c82-170">Nell'esempio viene utilizzato un oggetto `enum` contenente i codici di errore passati al client mobile:</span><span class="sxs-lookup"><span data-stu-id="20c82-170">The sample uses an `enum` containing error codes that are passed to the mobile client:</span></span>

:::code language="csharp" source="~/../xamarin-forms-samples/WebServices/TodoREST/TodoAPI/TodoAPI/Controllers/TodoItemsController.cs" id="snippetErrorCode":::

<span data-ttu-id="20c82-171">Eseguire il test dell'aggiunta di nuovi elementi con Postman scegliendo il verbo POST che rende disponibile il nuovo oggetto in formato JSON nel corpo della richiesta.</span><span class="sxs-lookup"><span data-stu-id="20c82-171">Test adding new items using Postman by choosing the POST verb providing the new object in JSON format in the Body of the request.</span></span> <span data-ttu-id="20c82-172">È anche necessario aggiungere un'intestazione di richiesta specificando come `Content-Type` la voce `application/json`.</span><span class="sxs-lookup"><span data-stu-id="20c82-172">You should also add a request header specifying a `Content-Type` of `application/json`.</span></span>

![Console di Postman che visualizzano un POST e una risposta](native-mobile-backend/_static/postman-post.png)

<span data-ttu-id="20c82-174">Il metodo restituisce l'elemento appena creato nella risposta.</span><span class="sxs-lookup"><span data-stu-id="20c82-174">The method returns the newly created item in the response.</span></span>

### <a name="updating-items"></a><span data-ttu-id="20c82-175">Aggiornamento degli elementi</span><span class="sxs-lookup"><span data-stu-id="20c82-175">Updating Items</span></span>

<span data-ttu-id="20c82-176">La modifica dei record viene eseguita mediante richieste PUT HTTP.</span><span class="sxs-lookup"><span data-stu-id="20c82-176">Modifying records is done using HTTP PUT requests.</span></span> <span data-ttu-id="20c82-177">Fatta eccezione per questa modifica, il metodo `Edit` è quasi identico a `Create`.</span><span class="sxs-lookup"><span data-stu-id="20c82-177">Other than this change, the `Edit` method is almost identical to `Create`.</span></span> <span data-ttu-id="20c82-178">Si noti che se il record non viene trovato, l'azione `Edit` restituisce una risposta `NotFound` (404).</span><span class="sxs-lookup"><span data-stu-id="20c82-178">Note that if the record isn't found, the `Edit` action will return a `NotFound` (404) response.</span></span>

:::code language="csharp" source="~/../xamarin-forms-samples/WebServices/TodoREST/TodoAPI/TodoAPI/Controllers/TodoItemsController.cs" id="snippetEdit":::

<span data-ttu-id="20c82-179">Per eseguire test con Postman, impostare il verbo su PUT.</span><span class="sxs-lookup"><span data-stu-id="20c82-179">To test with Postman, change the verb to PUT.</span></span> <span data-ttu-id="20c82-180">Specificare i dati dell'oggetto aggiornati nel Corpo della richiesta.</span><span class="sxs-lookup"><span data-stu-id="20c82-180">Specify the updated object data in the Body of the request.</span></span>

![Console di Postman con un elemento PUT e una risposta](native-mobile-backend/_static/postman-put.png)

<span data-ttu-id="20c82-182">Se va a buon fine, questo metodo restituisce una risposta `NoContent` (204) per coerenza con l'API preesistente.</span><span class="sxs-lookup"><span data-stu-id="20c82-182">This method returns a `NoContent` (204) response when successful, for consistency with the pre-existing API.</span></span>

### <a name="deleting-items"></a><span data-ttu-id="20c82-183">Eliminazione di elementi</span><span class="sxs-lookup"><span data-stu-id="20c82-183">Deleting Items</span></span>

<span data-ttu-id="20c82-184">L'eliminazione di record si ottiene inviando richieste DELETE al servizio e passando l'ID dell'elemento da eliminare.</span><span class="sxs-lookup"><span data-stu-id="20c82-184">Deleting records is accomplished by making DELETE requests to the service, and passing the ID of the item to be deleted.</span></span> <span data-ttu-id="20c82-185">Come con gli aggiornamenti, le richieste relative a elementi che non esistono ricevono risposte `NotFound`.</span><span class="sxs-lookup"><span data-stu-id="20c82-185">As with updates, requests for items that don't exist will receive `NotFound` responses.</span></span> <span data-ttu-id="20c82-186">Una richiesta con esito positivo riceve una risposta `NoContent` (204).</span><span class="sxs-lookup"><span data-stu-id="20c82-186">Otherwise, a successful request will get a `NoContent` (204) response.</span></span>

:::code language="csharp" source="~/../xamarin-forms-samples/WebServices/TodoREST/TodoAPI/TodoAPI/Controllers/TodoItemsController.cs" id="snippetDelete":::

<span data-ttu-id="20c82-187">Si noti che durante il test della funzionalità di eliminazione non è necessario alcun elemento nella sezione Corpo della richiesta.</span><span class="sxs-lookup"><span data-stu-id="20c82-187">Note that when testing the delete functionality, nothing is required in the Body of the request.</span></span>

![Console di Postman con un'operazione DELETE e una risposta](native-mobile-backend/_static/postman-delete.png)

## <a name="prevent-over-posting"></a><span data-ttu-id="20c82-189">Impedisci overposting</span><span class="sxs-lookup"><span data-stu-id="20c82-189">Prevent over-posting</span></span>

<span data-ttu-id="20c82-190">Attualmente l'app di esempio espone l'intero `TodoItem` oggetto.</span><span class="sxs-lookup"><span data-stu-id="20c82-190">Currently the sample app exposes the entire `TodoItem` object.</span></span> <span data-ttu-id="20c82-191">Le app di produzione limitano in genere i dati di input e restituiti usando un subset del modello.</span><span class="sxs-lookup"><span data-stu-id="20c82-191">Production apps typically limit the data that's input and returned using a subset of the model.</span></span> <span data-ttu-id="20c82-192">Esistono diversi motivi alla base di questo e la sicurezza è una delle principali.</span><span class="sxs-lookup"><span data-stu-id="20c82-192">There are multiple reasons behind this and security is a major one.</span></span> <span data-ttu-id="20c82-193">Il subset di un modello è in genere indicato come oggetto Trasferimento dati (DTO), modello di input o modello di visualizzazione.</span><span class="sxs-lookup"><span data-stu-id="20c82-193">The subset of a model is usually referred to as a Data Transfer Object (DTO), input model, or view model.</span></span> <span data-ttu-id="20c82-194">**Dto** viene usato in questo articolo.</span><span class="sxs-lookup"><span data-stu-id="20c82-194">**DTO** is used in this article.</span></span>

<span data-ttu-id="20c82-195">Un DTO può essere usato per:</span><span class="sxs-lookup"><span data-stu-id="20c82-195">A DTO may be used to:</span></span>

* <span data-ttu-id="20c82-196">Impedisci l'overposting.</span><span class="sxs-lookup"><span data-stu-id="20c82-196">Prevent over-posting.</span></span>
* <span data-ttu-id="20c82-197">Nascondere le proprietà che i client non dovrebbero visualizzare.</span><span class="sxs-lookup"><span data-stu-id="20c82-197">Hide properties that clients are not supposed to view.</span></span>
* <span data-ttu-id="20c82-198">Omettere alcune proprietà per ridurre le dimensioni del payload.</span><span class="sxs-lookup"><span data-stu-id="20c82-198">Omit some properties in order to reduce payload size.</span></span>
* <span data-ttu-id="20c82-199">Rendere flat gli oggetti grafici che contengono oggetti annidati.</span><span class="sxs-lookup"><span data-stu-id="20c82-199">Flatten object graphs that contain nested objects.</span></span> <span data-ttu-id="20c82-200">I grafici a oggetti flat possono essere più pratici per i client.</span><span class="sxs-lookup"><span data-stu-id="20c82-200">Flattened object graphs can be more convenient for clients.</span></span>

<span data-ttu-id="20c82-201">Per illustrare l'approccio DTO, vedere [impedire l'overposting](xref:tutorials/first-web-api#prevent-over-posting)</span><span class="sxs-lookup"><span data-stu-id="20c82-201">To demonstrate the DTO approach, see [Prevent over-posting](xref:tutorials/first-web-api#prevent-over-posting)</span></span>

## <a name="common-web-api-conventions"></a><span data-ttu-id="20c82-202">Convenzioni comuni dell'API Web</span><span class="sxs-lookup"><span data-stu-id="20c82-202">Common Web API Conventions</span></span>

<span data-ttu-id="20c82-203">Durante lo sviluppo dei servizi back-end per l'app è necessario creare un set di convenzioni o criteri coerenti per la gestione dei problemi di montaggio incrociato.</span><span class="sxs-lookup"><span data-stu-id="20c82-203">As you develop the backend services for your app, you will want to come up with a consistent set of conventions or policies for handling cross-cutting concerns.</span></span> <span data-ttu-id="20c82-204">Ad esempio nel servizio precedente le richieste di record specifici che non sono stati trovati ricevono una risposta `NotFound` anziché una risposta `BadRequest`.</span><span class="sxs-lookup"><span data-stu-id="20c82-204">For example, in the service shown above, requests for specific records that weren't found received a `NotFound` response, rather than a `BadRequest` response.</span></span> <span data-ttu-id="20c82-205">Analogamente, i comandi inoltrati a questo servizio che trasmettono tipi associati al modello verificano sempre `ModelState.IsValid` e restituiscono `BadRequest` per i tipi di modello non validi.</span><span class="sxs-lookup"><span data-stu-id="20c82-205">Similarly, commands made to this service that passed in model bound types always checked `ModelState.IsValid` and returned a `BadRequest` for invalid model types.</span></span>

<span data-ttu-id="20c82-206">Dopo aver identificato criteri comuni per le API è in genere possibile incapsulare tali criteri in un [filtro](../mvc/controllers/filters.md).</span><span class="sxs-lookup"><span data-stu-id="20c82-206">Once you've identified a common policy for your APIs, you can usually encapsulate it in a [filter](../mvc/controllers/filters.md).</span></span> <span data-ttu-id="20c82-207">Altre informazioni su [come incapsulare criteri comuni per le API nelle applicazioni ASP.NET Core MVC](/archive/msdn-magazine/2016/august/asp-net-core-real-world-asp-net-core-mvc-filters).</span><span class="sxs-lookup"><span data-stu-id="20c82-207">Learn more about [how to encapsulate common API policies in ASP.NET Core MVC applications](/archive/msdn-magazine/2016/august/asp-net-core-real-world-asp-net-core-mvc-filters).</span></span>

## <a name="additional-resources"></a><span data-ttu-id="20c82-208">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="20c82-208">Additional resources</span></span>

- [<span data-ttu-id="20c82-209">Novell. Forms: autenticazione del servizio Web</span><span class="sxs-lookup"><span data-stu-id="20c82-209">Xamarin.Forms: Web Service Authentication</span></span>](/xamarin/xamarin-forms/data-cloud/authentication/)
- [<span data-ttu-id="20c82-210">Novell. Forms: utilizzo di un servizio Web RESTful</span><span class="sxs-lookup"><span data-stu-id="20c82-210">Xamarin.Forms: Consume a RESTful Web Service</span></span>](/xamarin/xamarin-forms/data-cloud/web-services/rest)
- [<span data-ttu-id="20c82-211">Microsoft Learn: utilizzo di servizi Web REST nelle app Novell</span><span class="sxs-lookup"><span data-stu-id="20c82-211">Microsoft Learn: Consume REST web services in Xamarin Apps</span></span>](/learn/modules/consume-rest-services/)
- [<span data-ttu-id="20c82-212">Microsoft Learn: creare un'API Web con ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="20c82-212">Microsoft Learn: Create a web API with ASP.NET Core</span></span>](/learn/modules/build-web-api-aspnet-core/)
