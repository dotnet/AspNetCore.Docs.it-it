---
title: Creare servizi back-end per app native per dispositivi mobili con ASP.NET Core
author: ardalis
description: Informazioni su come creare servizi back-end mediante ASP.NET Core MVC per il supporto di app per dispositivi mobili native.
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
uid: mobile/native-mobile-backend
ms.openlocfilehash: 4e86241771e884ba9079bcdf9a09eebc6acd62c8
ms.sourcegitcommit: e311cfb77f26a0a23681019bd334929d1aaeda20
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/03/2021
ms.locfileid: "99530216"
---
# <a name="create-backend-services-for-native-mobile-apps-with-aspnet-core"></a><span data-ttu-id="486fb-103">Creare servizi back-end per app native per dispositivi mobili con ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="486fb-103">Create backend services for native mobile apps with ASP.NET Core</span></span>

<span data-ttu-id="486fb-104">Di [Steve Smith](https://ardalis.com/)</span><span class="sxs-lookup"><span data-stu-id="486fb-104">By [Steve Smith](https://ardalis.com/)</span></span>

<span data-ttu-id="486fb-105">Le app per dispositivi mobili possono comunicare con i servizi back-end di ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="486fb-105">Mobile apps can communicate with ASP.NET Core backend services.</span></span> <span data-ttu-id="486fb-106">Per istruzioni sulla connessione a servizi Web locali da simulatori iOS ed emulatori Android, vedere [Connettersi a servizi Web locali da simulatori iOS ed emulatori Android](/xamarin/cross-platform/deploy-test/connect-to-local-web-services).</span><span class="sxs-lookup"><span data-stu-id="486fb-106">For instructions on connecting local web services from iOS simulators and Android emulators, see [Connect to Local Web Services from iOS Simulators and Android Emulators](/xamarin/cross-platform/deploy-test/connect-to-local-web-services).</span></span>

[<span data-ttu-id="486fb-107">Visualizzare o scaricare codice di esempio di servizi back-end</span><span class="sxs-lookup"><span data-stu-id="486fb-107">View or download sample backend services code</span></span>](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mobile/native-mobile-backend/sample)

## <a name="the-sample-native-mobile-app"></a><span data-ttu-id="486fb-108">App per dispositivi mobili nativa di esempio</span><span class="sxs-lookup"><span data-stu-id="486fb-108">The Sample Native Mobile App</span></span>

<span data-ttu-id="486fb-109">Questa esercitazione illustra come creare servizi back-end mediante ASP.NET Core MVC per il supporto di app per dispositivi mobili native.</span><span class="sxs-lookup"><span data-stu-id="486fb-109">This tutorial demonstrates how to create backend services using ASP.NET Core MVC to support native mobile apps.</span></span> <span data-ttu-id="486fb-110">Usa come client nativo l'[app ToDoRest di Xamarin.Forms](/xamarin/xamarin-forms/data-cloud/consuming/rest), che include client nativi separati per i dispositivi Android, iOS, Windows Universal e Windows Phone.</span><span class="sxs-lookup"><span data-stu-id="486fb-110">It uses the [Xamarin Forms ToDoRest app](/xamarin/xamarin-forms/data-cloud/consuming/rest) as its native client, which includes separate native clients for Android, iOS, Windows Universal, and Window Phone devices.</span></span> <span data-ttu-id="486fb-111">È possibile eseguire l'esercitazione collegata per creare l'app nativa (e installare gli strumenti Xamarin gratuiti necessari), nonché scaricare la soluzione di esempio di Xamarin.</span><span class="sxs-lookup"><span data-stu-id="486fb-111">You can follow the linked tutorial to create the native app (and install the necessary free Xamarin tools), as well as download the Xamarin sample solution.</span></span> <span data-ttu-id="486fb-112">L'esempio di Xamarin include un progetto di servizi ASP.NET Web API 2, che viene sostituito dall'app ASP.NET Core di questo articolo (senza richiedere nessuna modifica al client).</span><span class="sxs-lookup"><span data-stu-id="486fb-112">The Xamarin sample includes an ASP.NET Web API 2 services project, which this article's ASP.NET Core app replaces (with no changes required by the client).</span></span>

![Applicazione ToDoRest in esecuzione su uno smartphone Android](native-mobile-backend/_static/todo-android.png)

### <a name="features"></a><span data-ttu-id="486fb-114">Funzionalità</span><span class="sxs-lookup"><span data-stu-id="486fb-114">Features</span></span>

<span data-ttu-id="486fb-115">L'app ToDoRest supporta l'aggiunta, l'eliminazione, l'aggiornamento e l'aggiunta a elenco di elementi attività.</span><span class="sxs-lookup"><span data-stu-id="486fb-115">The ToDoRest app supports listing, adding, deleting, and updating To-Do items.</span></span> <span data-ttu-id="486fb-116">Ogni elemento include un ID, un nome, delle note e una proprietà che indica se è stato eseguito.</span><span class="sxs-lookup"><span data-stu-id="486fb-116">Each item has an ID, a Name, Notes, and a property indicating whether it's been Done yet.</span></span>

<span data-ttu-id="486fb-117">La visualizzazione principale degli elementi, che appare nell'immagine in alto, contiene un elenco con il nome di ogni elemento. Un segno di spunta indica se l'elemento è stato eseguito.</span><span class="sxs-lookup"><span data-stu-id="486fb-117">The main view of the items, as shown above, lists each item's name and indicates if it's done with a checkmark.</span></span>

<span data-ttu-id="486fb-118">Se si tocca l'icona `+` viene visualizzata una finestra di dialogo per l'aggiunta di elementi:</span><span class="sxs-lookup"><span data-stu-id="486fb-118">Tapping the `+` icon opens an add item dialog:</span></span>

![Finestra di dialogo per l'aggiunta di elementi](native-mobile-backend/_static/todo-android-new-item.png)

<span data-ttu-id="486fb-120">Se si tocca un elemento sulla schermata dell'elenco principale viene visualizzata una finestra di dialogo di modifica in cui è possibile modificare le opzioni Name (Nome), Notes (Note) e Done (Fatto) dell'elemento oppure eliminare l'elemento stesso:</span><span class="sxs-lookup"><span data-stu-id="486fb-120">Tapping an item on the main list screen opens up an edit dialog where the item's Name, Notes, and Done settings can be modified, or the item can be deleted:</span></span>

![Finestra di dialogo di modifica dell'elemento](native-mobile-backend/_static/todo-android-edit-item.png)

<span data-ttu-id="486fb-122">Questo esempio è configurato per impostazione predefinita per l'uso dei servizi back-end ospitati in developer.xamarin.com, che consentono operazioni di sola lettura.</span><span class="sxs-lookup"><span data-stu-id="486fb-122">This sample is configured by default to use backend services hosted at developer.xamarin.com, which allow read-only operations.</span></span> <span data-ttu-id="486fb-123">Per eseguire direttamente il test nel proprio computer con l'app ASP.NET Core creata nella sezione successiva, è necessario modificare la costante `RestUrl` dell'app.</span><span class="sxs-lookup"><span data-stu-id="486fb-123">To test it out yourself against the ASP.NET Core app created in the next section running on your computer, you'll need to update the app's `RestUrl` constant.</span></span> <span data-ttu-id="486fb-124">Passare al progetto `ToDoREST` e aprire il file *Constants.cs*.</span><span class="sxs-lookup"><span data-stu-id="486fb-124">Navigate to the `ToDoREST` project and open the *Constants.cs* file.</span></span> <span data-ttu-id="486fb-125">Sostituire `RestUrl` con un URL che include l'indirizzo IP del computer (non localhost o 127.0.0.1, poiché questo indirizzo viene usato dall'emulatore di dispositivo e non dal computer in uso).</span><span class="sxs-lookup"><span data-stu-id="486fb-125">Replace the `RestUrl` with a URL that includes your machine's IP address (not localhost or 127.0.0.1, since this address is used from the device emulator, not from your machine).</span></span> <span data-ttu-id="486fb-126">Includere anche il numero di porta (5000).</span><span class="sxs-lookup"><span data-stu-id="486fb-126">Include the port number as well (5000).</span></span> <span data-ttu-id="486fb-127">Per verificare che i servizi funzionino con un dispositivo, assicurarsi che l'accesso a questa porta non sia bloccato da un firewall attivo.</span><span class="sxs-lookup"><span data-stu-id="486fb-127">In order to test that your services work with a device, ensure you don't have an active firewall blocking access to this port.</span></span>

```csharp
// URL of REST service (Xamarin ReadOnly Service)
//public static string RestUrl = "http://developer.xamarin.com:8081/api/todoitems{0}";

// use your machine's IP address
public static string RestUrl = "http://192.168.1.207:5000/api/todoitems/{0}";
```

## <a name="creating-the-aspnet-core-project"></a><span data-ttu-id="486fb-128">Creazione del progetto ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="486fb-128">Creating the ASP.NET Core Project</span></span>

<span data-ttu-id="486fb-129">Creare una nuova applicazione Web ASP.NET Core in Visual Studio.</span><span class="sxs-lookup"><span data-stu-id="486fb-129">Create a new ASP.NET Core Web Application in Visual Studio.</span></span> <span data-ttu-id="486fb-130">Scegliere il modello API Web e Nessuna autenticazione.</span><span class="sxs-lookup"><span data-stu-id="486fb-130">Choose the Web API template and No Authentication.</span></span> <span data-ttu-id="486fb-131">Denominare il progetto *ToDoApi*.</span><span class="sxs-lookup"><span data-stu-id="486fb-131">Name the project *ToDoApi*.</span></span>

![Finestra di dialogo Nuova applicazione Web ASP.NET Core con il modello di progetto API Web selezionato](native-mobile-backend/_static/web-api-template.png)

<span data-ttu-id="486fb-133">L'applicazione deve rispondere a tutte le richieste effettuate sulla porta 5000.</span><span class="sxs-lookup"><span data-stu-id="486fb-133">The application should respond to all requests made to port 5000.</span></span> <span data-ttu-id="486fb-134">A tale scopo aggiornare *Program.cs* in modo che includa `.UseUrls("http://*:5000")`:</span><span class="sxs-lookup"><span data-stu-id="486fb-134">Update *Program.cs* to include `.UseUrls("http://*:5000")` to achieve this:</span></span>

[!code-csharp[](native-mobile-backend/sample/ToDoApi/src/ToDoApi/Program.cs?range=10-16&highlight=3)]

> [!NOTE]
> <span data-ttu-id="486fb-135">Assicurarsi di eseguire l'applicazione direttamente anziché dietro IIS Express, che per impostazione predefinita ignora le richieste non locali.</span><span class="sxs-lookup"><span data-stu-id="486fb-135">Make sure you run the application directly, rather than behind IIS Express, which ignores non-local requests by default.</span></span> <span data-ttu-id="486fb-136">Eseguire [dotnet run](/dotnet/core/tools/dotnet-run) al prompt dei comandi oppure scegliere il profilo del nome dell'applicazione nell'elenco a discesa Destinazione di debug sulla barra degli strumenti di Visual Studio.</span><span class="sxs-lookup"><span data-stu-id="486fb-136">Run [dotnet run](/dotnet/core/tools/dotnet-run) from a command prompt, or choose the application name profile from the Debug Target dropdown in the Visual Studio toolbar.</span></span>

<span data-ttu-id="486fb-137">Aggiungere una classe modello che rappresenti gli elementi attività.</span><span class="sxs-lookup"><span data-stu-id="486fb-137">Add a model class to represent To-Do items.</span></span> <span data-ttu-id="486fb-138">Contrassegnare i campi obbligatori con l' `[Required]` attributo:</span><span class="sxs-lookup"><span data-stu-id="486fb-138">Mark required fields with the `[Required]` attribute:</span></span>

[!code-csharp[](native-mobile-backend/sample/ToDoApi/src/ToDoApi/Models/ToDoItem.cs)]

<span data-ttu-id="486fb-139">I metodi dell'API richiedono un approccio per l'uso dei dati.</span><span class="sxs-lookup"><span data-stu-id="486fb-139">The API methods require some way to work with data.</span></span> <span data-ttu-id="486fb-140">Usare la stessa interfaccia `IToDoRepository` usata nell'esempio originale di Xamarin:</span><span class="sxs-lookup"><span data-stu-id="486fb-140">Use the same `IToDoRepository` interface the original Xamarin sample uses:</span></span>

[!code-csharp[](native-mobile-backend/sample/ToDoApi/src/ToDoApi/Interfaces/IToDoRepository.cs)]

<span data-ttu-id="486fb-141">Per questo esempio, l'implementazione usa solo una raccolta privata di elementi:</span><span class="sxs-lookup"><span data-stu-id="486fb-141">For this sample, the implementation just uses a private collection of items:</span></span>

[!code-csharp[](native-mobile-backend/sample/ToDoApi/src/ToDoApi/Services/ToDoRepository.cs)]

<span data-ttu-id="486fb-142">Configurare l'implementazione in *Startup.cs*:</span><span class="sxs-lookup"><span data-stu-id="486fb-142">Configure the implementation in *Startup.cs*:</span></span>

[!code-csharp[](native-mobile-backend/sample/ToDoApi/src/ToDoApi/Startup.cs?highlight=6&range=29-35)]

<span data-ttu-id="486fb-143">A questo punto si è pronti per creare *ToDoItemsController*.</span><span class="sxs-lookup"><span data-stu-id="486fb-143">At this point, you're ready to create the *ToDoItemsController*.</span></span>

> [!TIP]
> <span data-ttu-id="486fb-144">Per altre informazioni sulla creazione di API Web, vedere [Sviluppare la prima API Web con ASP.NET Core MVC e Visual Studio](../tutorials/first-web-api.md).</span><span class="sxs-lookup"><span data-stu-id="486fb-144">Learn more about creating web APIs in [Build your first Web API with ASP.NET Core MVC and Visual Studio](../tutorials/first-web-api.md).</span></span>

## <a name="creating-the-controller"></a><span data-ttu-id="486fb-145">Creazione del controller</span><span class="sxs-lookup"><span data-stu-id="486fb-145">Creating the Controller</span></span>

<span data-ttu-id="486fb-146">Aggiungere al progetto un nuovo controller, *ToDoItemsController*.</span><span class="sxs-lookup"><span data-stu-id="486fb-146">Add a new controller to the project, *ToDoItemsController*.</span></span> <span data-ttu-id="486fb-147">Il controller deve ereditare da Microsoft.AspNetCore.Mvc.Controller.</span><span class="sxs-lookup"><span data-stu-id="486fb-147">It should inherit from Microsoft.AspNetCore.Mvc.Controller.</span></span> <span data-ttu-id="486fb-148">Aggiungere un attributo `Route` per indicare che il controller gestirà le richieste inoltrate ai percorsi che iniziano con `api/todoitems`.</span><span class="sxs-lookup"><span data-stu-id="486fb-148">Add a `Route` attribute to indicate that the controller will handle requests made to paths starting with `api/todoitems`.</span></span> <span data-ttu-id="486fb-149">Il token `[controller]` nella route viene sostituito dal nome del controller (omettendo il suffisso `Controller`) ed è particolarmente utile per le route globali.</span><span class="sxs-lookup"><span data-stu-id="486fb-149">The `[controller]` token in the route is replaced by the name of the controller (omitting the `Controller` suffix), and is especially helpful for global routes.</span></span> <span data-ttu-id="486fb-150">Altre informazioni sul [routing](../fundamentals/routing.md).</span><span class="sxs-lookup"><span data-stu-id="486fb-150">Learn more about [routing](../fundamentals/routing.md).</span></span>

<span data-ttu-id="486fb-151">Per funzionare, il controller richiede un `IToDoRepository`. Richiedere un'istanza di questo tipo tramite il costruttore del controller.</span><span class="sxs-lookup"><span data-stu-id="486fb-151">The controller requires an `IToDoRepository` to function; request an instance of this type through the controller's constructor.</span></span> <span data-ttu-id="486fb-152">In fase di esecuzione, questa istanza viene resa disponibile tramite il supporto dell'[inserimento di dipendenze](../fundamentals/dependency-injection.md) specificato dal framework.</span><span class="sxs-lookup"><span data-stu-id="486fb-152">At runtime, this instance will be provided using the framework's support for [dependency injection](../fundamentals/dependency-injection.md).</span></span>

[!code-csharp[](native-mobile-backend/sample/ToDoApi/src/ToDoApi/Controllers/ToDoItemsController.cs?range=1-17&highlight=9,14)]

<span data-ttu-id="486fb-153">Questa API supporta quattro verbi HTTP diversi per eseguire operazioni CRUD (Create, Read, Update, Delete - Creazione, Lettura, Aggiornamento, Eliminazione) nell'origine dati.</span><span class="sxs-lookup"><span data-stu-id="486fb-153">This API supports four different HTTP verbs to perform CRUD (Create, Read, Update, Delete) operations on the data source.</span></span> <span data-ttu-id="486fb-154">L'operazione più semplice tra queste è Read, che corrisponde a una richiesta HTTP GET.</span><span class="sxs-lookup"><span data-stu-id="486fb-154">The simplest of these is the Read operation, which corresponds to an HTTP GET request.</span></span>

### <a name="reading-items"></a><span data-ttu-id="486fb-155">Lettura degli elementi</span><span class="sxs-lookup"><span data-stu-id="486fb-155">Reading Items</span></span>

<span data-ttu-id="486fb-156">La richiesta di un elenco di elementi viene eseguita con una richiesta GET al metodo `List`.</span><span class="sxs-lookup"><span data-stu-id="486fb-156">Requesting a list of items is done with a GET request to the `List` method.</span></span> <span data-ttu-id="486fb-157">L'attributo `[HttpGet]` del metodo `List` indica che questa azione deve gestire solo le richieste GET.</span><span class="sxs-lookup"><span data-stu-id="486fb-157">The `[HttpGet]` attribute on the `List` method indicates that this action should only handle GET requests.</span></span> <span data-ttu-id="486fb-158">La route di questa azione è la route specificata nel controller.</span><span class="sxs-lookup"><span data-stu-id="486fb-158">The route for this action is the route specified on the controller.</span></span> <span data-ttu-id="486fb-159">Non è necessario usare il nome dell'azione come parte della route.</span><span class="sxs-lookup"><span data-stu-id="486fb-159">You don't necessarily need to use the action name as part of the route.</span></span> <span data-ttu-id="486fb-160">È sufficiente garantire che ogni azione disponga di una route univoca e non ambigua.</span><span class="sxs-lookup"><span data-stu-id="486fb-160">You just need to ensure each action has a unique and unambiguous route.</span></span> <span data-ttu-id="486fb-161">Gli attributi di routing possono essere applicati sia a livello di controller che di metodo per definire route specifiche.</span><span class="sxs-lookup"><span data-stu-id="486fb-161">Routing attributes can be applied at both the controller and method levels to build up specific routes.</span></span>

[!code-csharp[](native-mobile-backend/sample/ToDoApi/src/ToDoApi/Controllers/ToDoItemsController.cs?range=19-23)]

<span data-ttu-id="486fb-162">Il metodo `List` restituisce un codice di risposta 200 (OK) e tutti gli elementi di attività (ToDo), serializzati con formato JSON.</span><span class="sxs-lookup"><span data-stu-id="486fb-162">The `List` method returns a 200 OK response code and all of the ToDo items, serialized as JSON.</span></span>

<span data-ttu-id="486fb-163">È possibile eseguire il test del nuovo metodo API usando un'ampia gamma di strumenti, ad esempio [Postman](https://www.getpostman.com/docs/), visualizzato di seguito:</span><span class="sxs-lookup"><span data-stu-id="486fb-163">You can test your new API method using a variety of tools, such as [Postman](https://www.getpostman.com/docs/), shown here:</span></span>

![Console di Postman con una richiesta GET per elementi di attività (todoitems) e il corpo della risposta con JSON per tre elementi restituiti](native-mobile-backend/_static/postman-get.png)

### <a name="creating-items"></a><span data-ttu-id="486fb-165">Creazione di elementi</span><span class="sxs-lookup"><span data-stu-id="486fb-165">Creating Items</span></span>

<span data-ttu-id="486fb-166">Per convenzione, la creazione di nuovi elementi di dati viene associata al verbo HTTP POST.</span><span class="sxs-lookup"><span data-stu-id="486fb-166">By convention, creating new data items is mapped to the HTTP POST verb.</span></span> <span data-ttu-id="486fb-167">Al `Create` metodo è `[HttpPost]` applicato un attributo e viene accettata un' `ToDoItem` istanza.</span><span class="sxs-lookup"><span data-stu-id="486fb-167">The `Create` method has an `[HttpPost]` attribute applied to it and accepts a `ToDoItem` instance.</span></span> <span data-ttu-id="486fb-168">Poiché l' `item` argomento viene passato nel corpo del post, questo parametro specifica l' `[FromBody]` attributo.</span><span class="sxs-lookup"><span data-stu-id="486fb-168">Since the `item` argument is passed in the body of the POST, this parameter specifies the `[FromBody]` attribute.</span></span>

<span data-ttu-id="486fb-169">All'interno del metodo, l'elemento viene verificato per validità ed esistenza precedente nell'archivio dati. Se non si verificano problemi, l'elemento viene aggiunto tramite il repository.</span><span class="sxs-lookup"><span data-stu-id="486fb-169">Inside the method, the item is checked for validity and prior existence in the data store, and if no issues occur, it's added using the repository.</span></span> <span data-ttu-id="486fb-170">La verifica `ModelState.IsValid` esegue la [convalida del modello](../mvc/models/validation.md) e deve essere eseguita in ogni metodo dell'API che accetta input utente.</span><span class="sxs-lookup"><span data-stu-id="486fb-170">Checking `ModelState.IsValid` performs [model validation](../mvc/models/validation.md), and should be done in every API method that accepts user input.</span></span>

[!code-csharp[](native-mobile-backend/sample/ToDoApi/src/ToDoApi/Controllers/ToDoItemsController.cs?range=25-46)]

<span data-ttu-id="486fb-171">L'esempio usa un'enumerazione contenente codici di errore che vengono passati al client per dispositivi mobili:</span><span class="sxs-lookup"><span data-stu-id="486fb-171">The sample uses an enum containing error codes that are passed to the mobile client:</span></span>

[!code-csharp[](native-mobile-backend/sample/ToDoApi/src/ToDoApi/Controllers/ToDoItemsController.cs?range=91-99)]

<span data-ttu-id="486fb-172">Eseguire il test dell'aggiunta di nuovi elementi con Postman scegliendo il verbo POST che rende disponibile il nuovo oggetto in formato JSON nel corpo della richiesta.</span><span class="sxs-lookup"><span data-stu-id="486fb-172">Test adding new items using Postman by choosing the POST verb providing the new object in JSON format in the Body of the request.</span></span> <span data-ttu-id="486fb-173">È anche necessario aggiungere un'intestazione di richiesta specificando come `Content-Type` la voce `application/json`.</span><span class="sxs-lookup"><span data-stu-id="486fb-173">You should also add a request header specifying a `Content-Type` of `application/json`.</span></span>

![Console di Postman che visualizzano un POST e una risposta](native-mobile-backend/_static/postman-post.png)

<span data-ttu-id="486fb-175">Il metodo restituisce l'elemento appena creato nella risposta.</span><span class="sxs-lookup"><span data-stu-id="486fb-175">The method returns the newly created item in the response.</span></span>

### <a name="updating-items"></a><span data-ttu-id="486fb-176">Aggiornamento degli elementi</span><span class="sxs-lookup"><span data-stu-id="486fb-176">Updating Items</span></span>

<span data-ttu-id="486fb-177">La modifica dei record viene eseguita mediante richieste PUT HTTP.</span><span class="sxs-lookup"><span data-stu-id="486fb-177">Modifying records is done using HTTP PUT requests.</span></span> <span data-ttu-id="486fb-178">Fatta eccezione per questa modifica, il metodo `Edit` è quasi identico a `Create`.</span><span class="sxs-lookup"><span data-stu-id="486fb-178">Other than this change, the `Edit` method is almost identical to `Create`.</span></span> <span data-ttu-id="486fb-179">Si noti che se il record non viene trovato, l'azione `Edit` restituisce una risposta `NotFound` (404).</span><span class="sxs-lookup"><span data-stu-id="486fb-179">Note that if the record isn't found, the `Edit` action will return a `NotFound` (404) response.</span></span>

[!code-csharp[](native-mobile-backend/sample/ToDoApi/src/ToDoApi/Controllers/ToDoItemsController.cs?range=48-69)]

<span data-ttu-id="486fb-180">Per eseguire test con Postman, impostare il verbo su PUT.</span><span class="sxs-lookup"><span data-stu-id="486fb-180">To test with Postman, change the verb to PUT.</span></span> <span data-ttu-id="486fb-181">Specificare i dati dell'oggetto aggiornati nel Corpo della richiesta.</span><span class="sxs-lookup"><span data-stu-id="486fb-181">Specify the updated object data in the Body of the request.</span></span>

![Console di Postman con un elemento PUT e una risposta](native-mobile-backend/_static/postman-put.png)

<span data-ttu-id="486fb-183">Se va a buon fine, questo metodo restituisce una risposta `NoContent` (204) per coerenza con l'API preesistente.</span><span class="sxs-lookup"><span data-stu-id="486fb-183">This method returns a `NoContent` (204) response when successful, for consistency with the pre-existing API.</span></span>

### <a name="deleting-items"></a><span data-ttu-id="486fb-184">Eliminazione di elementi</span><span class="sxs-lookup"><span data-stu-id="486fb-184">Deleting Items</span></span>

<span data-ttu-id="486fb-185">L'eliminazione di record si ottiene inviando richieste DELETE al servizio e passando l'ID dell'elemento da eliminare.</span><span class="sxs-lookup"><span data-stu-id="486fb-185">Deleting records is accomplished by making DELETE requests to the service, and passing the ID of the item to be deleted.</span></span> <span data-ttu-id="486fb-186">Come con gli aggiornamenti, le richieste relative a elementi che non esistono ricevono risposte `NotFound`.</span><span class="sxs-lookup"><span data-stu-id="486fb-186">As with updates, requests for items that don't exist will receive `NotFound` responses.</span></span> <span data-ttu-id="486fb-187">Una richiesta con esito positivo riceve una risposta `NoContent` (204).</span><span class="sxs-lookup"><span data-stu-id="486fb-187">Otherwise, a successful request will get a `NoContent` (204) response.</span></span>

[!code-csharp[](native-mobile-backend/sample/ToDoApi/src/ToDoApi/Controllers/ToDoItemsController.cs?range=71-88)]

<span data-ttu-id="486fb-188">Si noti che durante il test della funzionalità di eliminazione non è necessario alcun elemento nella sezione Corpo della richiesta.</span><span class="sxs-lookup"><span data-stu-id="486fb-188">Note that when testing the delete functionality, nothing is required in the Body of the request.</span></span>

![Console di Postman con un'operazione DELETE e una risposta](native-mobile-backend/_static/postman-delete.png)

## <a name="common-web-api-conventions"></a><span data-ttu-id="486fb-190">Convenzioni comuni dell'API Web</span><span class="sxs-lookup"><span data-stu-id="486fb-190">Common Web API Conventions</span></span>

<span data-ttu-id="486fb-191">Durante lo sviluppo dei servizi back-end per l'app è necessario creare un set di convenzioni o criteri coerenti per la gestione dei problemi di montaggio incrociato.</span><span class="sxs-lookup"><span data-stu-id="486fb-191">As you develop the backend services for your app, you will want to come up with a consistent set of conventions or policies for handling cross-cutting concerns.</span></span> <span data-ttu-id="486fb-192">Ad esempio nel servizio precedente le richieste di record specifici che non sono stati trovati ricevono una risposta `NotFound` anziché una risposta `BadRequest`.</span><span class="sxs-lookup"><span data-stu-id="486fb-192">For example, in the service shown above, requests for specific records that weren't found received a `NotFound` response, rather than a `BadRequest` response.</span></span> <span data-ttu-id="486fb-193">Analogamente, i comandi inoltrati a questo servizio che trasmettono tipi associati al modello verificano sempre `ModelState.IsValid` e restituiscono `BadRequest` per i tipi di modello non validi.</span><span class="sxs-lookup"><span data-stu-id="486fb-193">Similarly, commands made to this service that passed in model bound types always checked `ModelState.IsValid` and returned a `BadRequest` for invalid model types.</span></span>

<span data-ttu-id="486fb-194">Dopo aver identificato criteri comuni per le API è in genere possibile incapsulare tali criteri in un [filtro](../mvc/controllers/filters.md).</span><span class="sxs-lookup"><span data-stu-id="486fb-194">Once you've identified a common policy for your APIs, you can usually encapsulate it in a [filter](../mvc/controllers/filters.md).</span></span> <span data-ttu-id="486fb-195">Altre informazioni su [come incapsulare criteri comuni per le API nelle applicazioni ASP.NET Core MVC](/archive/msdn-magazine/2016/august/asp-net-core-real-world-asp-net-core-mvc-filters).</span><span class="sxs-lookup"><span data-stu-id="486fb-195">Learn more about [how to encapsulate common API policies in ASP.NET Core MVC applications](/archive/msdn-magazine/2016/august/asp-net-core-real-world-asp-net-core-mvc-filters).</span></span>

## <a name="additional-resources"></a><span data-ttu-id="486fb-196">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="486fb-196">Additional resources</span></span>

- [<span data-ttu-id="486fb-197">Novell. Forms: autenticazione del servizio Web</span><span class="sxs-lookup"><span data-stu-id="486fb-197">Xamarin.Forms: Web Service Authentication</span></span>](/xamarin/xamarin-forms/data-cloud/authentication/)
- [<span data-ttu-id="486fb-198">Novell. Forms: utilizzo di un servizio Web RESTful</span><span class="sxs-lookup"><span data-stu-id="486fb-198">Xamarin.Forms: Consume a RESTful Web Service</span></span>](/xamarin/xamarin-forms/data-cloud/web-services/rest)
- [<span data-ttu-id="486fb-199">Microsoft Learn: utilizzo di servizi Web REST nelle app Novell</span><span class="sxs-lookup"><span data-stu-id="486fb-199">Microsoft Learn: Consume REST web services in Xamarin Apps</span></span>](/learn/modules/consume-rest-services/)
- [<span data-ttu-id="486fb-200">Microsoft Learn: creare un'API Web con ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="486fb-200">Microsoft Learn: Create a web API with ASP.NET Core</span></span>](/learn/modules/build-web-api-aspnet-core/)
