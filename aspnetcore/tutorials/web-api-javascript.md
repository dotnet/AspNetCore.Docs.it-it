---
title: "Esercitazione: Chiamare un'API Web ASP.NET Core con JavaScript"
author: rick-anderson
description: Informazioni su come chiamare un'API Web ASP.NET Core con JavaScript.
ms.author: riande
ms.custom: mvc
ms.date: 08/27/2019
uid: tutorials/web-api-javascript
ms.openlocfilehash: 0070816149d64fc1d71d453eb0f135050c78597a
ms.sourcegitcommit: de17150e5ec7507d7114dde0e5dbc2e45a66ef53
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 08/28/2019
ms.locfileid: "70116640"
---
# <a name="tutorial-call-an-aspnet-core-web-api-with-javascript"></a><span data-ttu-id="ab550-103">Esercitazione: Chiamare un'API Web ASP.NET Core con JavaScript</span><span class="sxs-lookup"><span data-stu-id="ab550-103">Tutorial: Call an ASP.NET Core web API with JavaScript</span></span>

<span data-ttu-id="ab550-104">Di [Rick Anderson](https://twitter.com/RickAndMSFT)</span><span class="sxs-lookup"><span data-stu-id="ab550-104">By [Rick Anderson](https://twitter.com/RickAndMSFT)</span></span>

<span data-ttu-id="ab550-105">Questa esercitazione illustra come chiamare un'API Web ASP.NET Core con JavaScript usando l'[API Fetch](https://developer.mozilla.org/docs/Web/API/Fetch_API).</span><span class="sxs-lookup"><span data-stu-id="ab550-105">This tutorial shows how to call an ASP.NET Core web API with JavaScript, using the [Fetch API](https://developer.mozilla.org/docs/Web/API/Fetch_API).</span></span>

::: moniker range="< aspnetcore-3.0"

<span data-ttu-id="ab550-106">Per ASP.NET Core 2.2, vedere la versione 2.2 di [Chiamare l'API Web con JavaScript](xref:tutorials/first-web-api#call-the-web-api-with-javascript).</span><span class="sxs-lookup"><span data-stu-id="ab550-106">For ASP.NET Core 2.2, see the 2.2 version of [Call the web API with JavaScript](xref:tutorials/first-web-api#call-the-web-api-with-javascript).</span></span>

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

## <a name="prerequisites"></a><span data-ttu-id="ab550-107">Prerequisiti</span><span class="sxs-lookup"><span data-stu-id="ab550-107">Prerequisites</span></span>

* <span data-ttu-id="ab550-108">Completare [Esercitazione: Creare un'API Web](xref:tutorials/first-web-api)</span><span class="sxs-lookup"><span data-stu-id="ab550-108">Complete [Tutorial: Create a web API](xref:tutorials/first-web-api)</span></span>
* <span data-ttu-id="ab550-109">Familiarità con CSS, HTML e JavaScript</span><span class="sxs-lookup"><span data-stu-id="ab550-109">Familiarity with CSS, HTML, and JavaScript</span></span>

## <a name="call-the-web-api-with-javascript"></a><span data-ttu-id="ab550-110">Chiamare l'API Web con JavaScript</span><span class="sxs-lookup"><span data-stu-id="ab550-110">Call the web API with JavaScript</span></span>

<span data-ttu-id="ab550-111">In questa sezione verrà aggiunta una pagina HTML contenente i moduli per la creazione e la gestione di elementi attività.</span><span class="sxs-lookup"><span data-stu-id="ab550-111">In this section, you'll add an HTML page containing forms for creating and managing to-do items.</span></span> <span data-ttu-id="ab550-112">Agli elementi della pagina vengono associati gestori eventi.</span><span class="sxs-lookup"><span data-stu-id="ab550-112">Event handlers are attached to elements on the page.</span></span> <span data-ttu-id="ab550-113">I gestori eventi generano richieste HTTP ai metodi di azione dell'API Web.</span><span class="sxs-lookup"><span data-stu-id="ab550-113">The event handlers result in HTTP requests to the web API's action methods.</span></span> <span data-ttu-id="ab550-114">La funzione `fetch` dell'API Fetch avvia ogni richiesta HTTP.</span><span class="sxs-lookup"><span data-stu-id="ab550-114">The Fetch API's `fetch` function initiates each HTTP request.</span></span>

<span data-ttu-id="ab550-115">La funzione `fetch` restituisce un oggetto `Promise` che contiene una risposta HTTP rappresentata come oggetto `Response`.</span><span class="sxs-lookup"><span data-stu-id="ab550-115">The `fetch` function returns a `Promise` object, which contains an HTTP response represented as a `Response` object.</span></span> <span data-ttu-id="ab550-116">Un modello comune consiste nell'estrarre il corpo della risposta JSON richiamando la funzione `json` per l'oggetto `Response`.</span><span class="sxs-lookup"><span data-stu-id="ab550-116">A common pattern is to extract the JSON response body by invoking the `json` function on the `Response` object.</span></span> <span data-ttu-id="ab550-117">JavaScript aggiorna la pagina con i dettagli della risposta dell'API Web.</span><span class="sxs-lookup"><span data-stu-id="ab550-117">JavaScript updates the page with the details from the web API's response.</span></span>

<span data-ttu-id="ab550-118">La chiamata `fetch` più semplice accetta un solo parametro che rappresenta la route.</span><span class="sxs-lookup"><span data-stu-id="ab550-118">The simplest `fetch` call accepts a single parameter representing the route.</span></span> <span data-ttu-id="ab550-119">Un secondo parametro, noto come oggetto `init`, è facoltativo.</span><span class="sxs-lookup"><span data-stu-id="ab550-119">A second parameter, known as the `init` object, is optional.</span></span> <span data-ttu-id="ab550-120">`init` viene usato per configurare la richiesta HTTP.</span><span class="sxs-lookup"><span data-stu-id="ab550-120">`init` is used to configure the HTTP request.</span></span>

1. <span data-ttu-id="ab550-121">Configurare l'app in modo che [gestisca i file statici](/dotnet/api/microsoft.aspnetcore.builder.staticfileextensions.usestaticfiles#Microsoft_AspNetCore_Builder_StaticFileExtensions_UseStaticFiles_Microsoft_AspNetCore_Builder_IApplicationBuilder_) e [consenta il mapping di file predefinito](/dotnet/api/microsoft.aspnetcore.builder.defaultfilesextensions.usedefaultfiles#Microsoft_AspNetCore_Builder_DefaultFilesExtensions_UseDefaultFiles_Microsoft_AspNetCore_Builder_IApplicationBuilder_).</span><span class="sxs-lookup"><span data-stu-id="ab550-121">Configure the app to [serve static files](/dotnet/api/microsoft.aspnetcore.builder.staticfileextensions.usestaticfiles#Microsoft_AspNetCore_Builder_StaticFileExtensions_UseStaticFiles_Microsoft_AspNetCore_Builder_IApplicationBuilder_) and [enable default file mapping](/dotnet/api/microsoft.aspnetcore.builder.defaultfilesextensions.usedefaultfiles#Microsoft_AspNetCore_Builder_DefaultFilesExtensions_UseDefaultFiles_Microsoft_AspNetCore_Builder_IApplicationBuilder_).</span></span> <span data-ttu-id="ab550-122">Il codice evidenziato seguente è necessario nel metodo `Configure` di *Startup.cs*:</span><span class="sxs-lookup"><span data-stu-id="ab550-122">The following highlighted code is needed in the `Configure` method of *Startup.cs*:</span></span>

    [!code-csharp[](first-web-api/samples/3.0/TodoApi/StartupJavaScript.cs?highlight=8-9&name=snippet_configure)]

1. <span data-ttu-id="ab550-123">Creare una directory *wwwroot* nella radice del progetto.</span><span class="sxs-lookup"><span data-stu-id="ab550-123">Create a *wwwroot* directory in the project root.</span></span>

1. <span data-ttu-id="ab550-124">Aggiungere un file HTML denominato *index.html* alla directory *wwwroot*.</span><span class="sxs-lookup"><span data-stu-id="ab550-124">Add an HTML file named *index.html* to the *wwwroot* directory.</span></span> <span data-ttu-id="ab550-125">Sostituire il contenuto con il markup seguente:</span><span class="sxs-lookup"><span data-stu-id="ab550-125">Replace its contents with the following markup:</span></span>

    [!code-html[](first-web-api/samples/3.0/TodoApi/wwwroot/index.html)]

1. <span data-ttu-id="ab550-126">Aggiungere un file JavaScript con nome *site.js* alla directory *wwwroot*.</span><span class="sxs-lookup"><span data-stu-id="ab550-126">Add a JavaScript file named *site.js* to the *wwwroot* directory.</span></span> <span data-ttu-id="ab550-127">Sostituire il contenuto con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="ab550-127">Replace its contents with the following code:</span></span>

    [!code-javascript[](first-web-api/samples/3.0/TodoApi/wwwroot/js/site.js?name=snippet_SiteJs)]

<span data-ttu-id="ab550-128">Può essere necessario modificare le impostazioni di avvio del progetto ASP.NET Core per il test della pagina HTML in locale:</span><span class="sxs-lookup"><span data-stu-id="ab550-128">A change to the ASP.NET Core project's launch settings may be required to test the HTML page locally:</span></span>

1. <span data-ttu-id="ab550-129">Aprire *Properties\launchSettings.json*.</span><span class="sxs-lookup"><span data-stu-id="ab550-129">Open *Properties\launchSettings.json*.</span></span>
1. <span data-ttu-id="ab550-130">Rimuovere la proprietà `launchUrl` per forzare l'app ad aprire *index.html*, il file predefinito del progetto.</span><span class="sxs-lookup"><span data-stu-id="ab550-130">Remove the `launchUrl` property to force the app to open at *index.html*&mdash;the project's default file.</span></span>

<span data-ttu-id="ab550-131">In questo esempio vengono chiamati tutti i metodi CRUD dell'API Web.</span><span class="sxs-lookup"><span data-stu-id="ab550-131">This sample calls all of the CRUD methods of the web API.</span></span> <span data-ttu-id="ab550-132">Di seguito sono disponibili spiegazioni per le richieste dell'API Web.</span><span class="sxs-lookup"><span data-stu-id="ab550-132">Following are explanations of the web API requests.</span></span>

### <a name="get-a-list-of-to-do-items"></a><span data-ttu-id="ab550-133">Ottenere un elenco di elementi attività</span><span class="sxs-lookup"><span data-stu-id="ab550-133">Get a list of to-do items</span></span>

<span data-ttu-id="ab550-134">Nel codice seguente viene inviata una richiesta HTTP GET alla route *api/TodoItems*:</span><span class="sxs-lookup"><span data-stu-id="ab550-134">In the following code, an HTTP GET request is sent to the *api/TodoItems* route:</span></span>

[!code-javascript[](first-web-api/samples/3.0/TodoApi/wwwroot/js/site.js?name=snippet_GetItems)]

<span data-ttu-id="ab550-135">Quando l'API Web restituisce un codice di stato di esecuzione riuscita, viene chiamata la funzione `_displayItems`.</span><span class="sxs-lookup"><span data-stu-id="ab550-135">When the web API returns a successful status code, the `_displayItems` function is invoked.</span></span> <span data-ttu-id="ab550-136">Ogni elemento attività nel parametro matrice accettato da `_displayItems` viene aggiunto a una tabella con i pulsanti **Edit** (Modifica) e **Delete** (Elimina).</span><span class="sxs-lookup"><span data-stu-id="ab550-136">Each to-do item in the array parameter accepted by `_displayItems` is added to a table with **Edit** and **Delete** buttons.</span></span> <span data-ttu-id="ab550-137">Se la richiesta dell'API Web ha esito negativo, viene registrato un errore nella console del browser.</span><span class="sxs-lookup"><span data-stu-id="ab550-137">If the web API request fails, an error is logged to the browser's console.</span></span>

### <a name="add-a-to-do-item"></a><span data-ttu-id="ab550-138">Aggiungere un elemento attività</span><span class="sxs-lookup"><span data-stu-id="ab550-138">Add a to-do item</span></span>

<span data-ttu-id="ab550-139">Nel codice seguente:</span><span class="sxs-lookup"><span data-stu-id="ab550-139">In the following code:</span></span>

* <span data-ttu-id="ab550-140">Viene dichiarata una variabile `item` per costruire una rappresentazione letterale dell'oggetto dell'elemento attività.</span><span class="sxs-lookup"><span data-stu-id="ab550-140">An `item` variable is declared to construct an object literal representation of the to-do item.</span></span>
* <span data-ttu-id="ab550-141">Viene configurata una richiesta Fetch con le opzioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="ab550-141">A Fetch request is configured with the following options:</span></span>
    * <span data-ttu-id="ab550-142">`method`&mdash;specifica il verbo di azione HTTP POST.</span><span class="sxs-lookup"><span data-stu-id="ab550-142">`method`&mdash;specifies the POST HTTP action verb.</span></span>
    * <span data-ttu-id="ab550-143">`body`&mdash;specifica la rappresentazione JSON del corpo della richiesta.</span><span class="sxs-lookup"><span data-stu-id="ab550-143">`body`&mdash;specifies the JSON representation of the request body.</span></span> <span data-ttu-id="ab550-144">Il codice JSON viene prodotto passando il valore letterale dell'oggetto archiviato in `item` alla funzione [JSON.stringify](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify).</span><span class="sxs-lookup"><span data-stu-id="ab550-144">The JSON is produced by passing the object literal stored in `item` to the [JSON.stringify](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify) function.</span></span>
    * <span data-ttu-id="ab550-145">`headers`&mdash;specifica le intestazioni della richiesta HTTP `Accept` e `Content-Type`.</span><span class="sxs-lookup"><span data-stu-id="ab550-145">`headers`&mdash;specifies the `Accept` and `Content-Type` HTTP request headers.</span></span> <span data-ttu-id="ab550-146">Entrambe le intestazioni vengono impostate su `application/json` per specificare rispettivamente il tipo di supporto ricevuto e inviato.</span><span class="sxs-lookup"><span data-stu-id="ab550-146">Both headers are set to `application/json` to specify the media type being received and sent, respectively.</span></span>
* <span data-ttu-id="ab550-147">Una richiesta HTTP POST viene inviata alla route *api/TodoItems*.</span><span class="sxs-lookup"><span data-stu-id="ab550-147">An HTTP POST request is sent to the *api/TodoItems* route.</span></span>

[!code-javascript[](first-web-api/samples/3.0/TodoApi/wwwroot/js/site.js?name=snippet_AddItem)]

<span data-ttu-id="ab550-148">Quando l'API Web restituisce un codice di stato di esecuzione riuscita, viene richiamata la funzione `getItems` per aggiornare la tabella HTML.</span><span class="sxs-lookup"><span data-stu-id="ab550-148">When the web API returns a successful status code, the `getItems` function is invoked to update the HTML table.</span></span> <span data-ttu-id="ab550-149">Se la richiesta dell'API Web ha esito negativo, viene registrato un errore nella console del browser.</span><span class="sxs-lookup"><span data-stu-id="ab550-149">If the web API request fails, an error is logged to the browser's console.</span></span>

### <a name="update-a-to-do-item"></a><span data-ttu-id="ab550-150">Aggiornare un elemento attività</span><span class="sxs-lookup"><span data-stu-id="ab550-150">Update a to-do item</span></span>

<span data-ttu-id="ab550-151">L'aggiornamento di un elemento attività è simile all'aggiunta di un elemento. Esistono tuttavia due differenze significative:</span><span class="sxs-lookup"><span data-stu-id="ab550-151">Updating a to-do item is similar to adding one; however, there are two significant differences:</span></span>

* <span data-ttu-id="ab550-152">Alla route viene aggiunto come suffisso l'identificatore univoco dell'elemento da aggiornare.</span><span class="sxs-lookup"><span data-stu-id="ab550-152">The route is suffixed with the unique identifier of the item to update.</span></span> <span data-ttu-id="ab550-153">Ad esempio, *api/TodoItems/1*.</span><span class="sxs-lookup"><span data-stu-id="ab550-153">For example, *api/TodoItems/1*.</span></span>
* <span data-ttu-id="ab550-154">Il verbo di azione HTTP è PUT, come indicato dall'opzione `method`.</span><span class="sxs-lookup"><span data-stu-id="ab550-154">The HTTP action verb is PUT, as indicated by the `method` option.</span></span>

[!code-javascript[](first-web-api/samples/3.0/TodoApi/wwwroot/js/site.js?name=snippet_UpdateItem)]

### <a name="delete-a-to-do-item"></a><span data-ttu-id="ab550-155">Eliminare un elemento attività</span><span class="sxs-lookup"><span data-stu-id="ab550-155">Delete a to-do item</span></span>

<span data-ttu-id="ab550-156">Per eliminare un elemento attività, impostare l'opzione `method` della richiesta su `DELETE` e specificare l'identificatore univoco dell'elemento nell'URL.</span><span class="sxs-lookup"><span data-stu-id="ab550-156">To delete a to-do item, set the request's `method` option to `DELETE` and specify the item's unique identifier in the URL.</span></span>

[!code-javascript[](first-web-api/samples/3.0/TodoApi/wwwroot/js/site.js?name=snippet_DeleteItem)]

<span data-ttu-id="ab550-157">Passare all'esercitazione successiva per apprendere come generare pagine della Guida dell'API Web:</span><span class="sxs-lookup"><span data-stu-id="ab550-157">Advance to the next tutorial to learn how to generate web API help pages:</span></span>

> [!div class="nextstepaction"]
> <xref:tutorials/get-started-with-swashbuckle>

::: moniker-end