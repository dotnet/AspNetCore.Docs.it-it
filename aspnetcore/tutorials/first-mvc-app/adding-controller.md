---
title: Parte 2, aggiungere un controller a un'app MVC ASP.NET Core
author: rick-anderson
description: Parte 2 della serie di esercitazioni su ASP.NET Core MVC.
ms.author: riande
ms.date: 01/23/2021
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
uid: tutorials/first-mvc-app/adding-controller
ms.custom: contperf-fy21q3
ms.openlocfilehash: 1a13fe31859b9ed41edc746ba450f552bc07f344
ms.sourcegitcommit: a1db01b4d3bd8c57d7a9c94ce122a6db68002d66
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/04/2021
ms.locfileid: "102110209"
---
# <a name="part-2-add-a-controller-to-an-aspnet-core-mvc-app"></a><span data-ttu-id="6ef74-103">Parte 2, aggiungere un controller a un'app MVC ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="6ef74-103">Part 2, add a controller to an ASP.NET Core MVC app</span></span>

<span data-ttu-id="6ef74-104">Autore: [Rick Anderson](https://twitter.com/RickAndMSFT)</span><span class="sxs-lookup"><span data-stu-id="6ef74-104">By [Rick Anderson](https://twitter.com/RickAndMSFT)</span></span>

::: moniker range=">= aspnetcore-3.0"

<span data-ttu-id="6ef74-105">Il pattern architetturale Model-View-Controller (MVC) suddivide le app in tre componenti principali: **M** odello, **V** ista e **C** ontroller.</span><span class="sxs-lookup"><span data-stu-id="6ef74-105">The Model-View-Controller (MVC) architectural pattern separates an app into three main components: **M** odel, **V** iew, and **C** ontroller.</span></span> <span data-ttu-id="6ef74-106">Il pattern MVC consente di creare app più semplici da aggiornare e sottoporre a test rispetto alle app monolitiche tradizionali.</span><span class="sxs-lookup"><span data-stu-id="6ef74-106">The MVC pattern helps you create apps that are more testable and easier to update than traditional monolithic apps.</span></span>

<span data-ttu-id="6ef74-107">Le app basate sul pattern MVC contengono:</span><span class="sxs-lookup"><span data-stu-id="6ef74-107">MVC-based apps contain:</span></span>

* <span data-ttu-id="6ef74-108">**M** odelli: classi che rappresentano i dati dell'app.</span><span class="sxs-lookup"><span data-stu-id="6ef74-108">**M** odels: Classes that represent the data of the app.</span></span> <span data-ttu-id="6ef74-109">Le classi modello usano la logica di convalida per applicare le regole business per tali dati.</span><span class="sxs-lookup"><span data-stu-id="6ef74-109">The model classes use validation logic to enforce business rules for that data.</span></span> <span data-ttu-id="6ef74-110">In genere, gli oggetti del modello recuperano e archiviano lo stato del modello in un database.</span><span class="sxs-lookup"><span data-stu-id="6ef74-110">Typically, model objects retrieve and store model state in a database.</span></span> <span data-ttu-id="6ef74-111">In questa esercitazione, un modello `Movie` recupera i dati dei film da un database, li fornisce alla vista o li aggiorna.</span><span class="sxs-lookup"><span data-stu-id="6ef74-111">In this tutorial, a `Movie` model retrieves movie data from a database, provides it to the view or updates it.</span></span> <span data-ttu-id="6ef74-112">I dati aggiornati vengono scritti in un database.</span><span class="sxs-lookup"><span data-stu-id="6ef74-112">Updated data is written to a database.</span></span>
* <span data-ttu-id="6ef74-113">**V** iste: componenti che visualizzano l'interfaccia utente dell'app.</span><span class="sxs-lookup"><span data-stu-id="6ef74-113">**V** iews: Views are the components that display the app's user interface (UI).</span></span> <span data-ttu-id="6ef74-114">In genere questa interfaccia utente presenta i dati del modello.</span><span class="sxs-lookup"><span data-stu-id="6ef74-114">Generally, this UI displays the model data.</span></span>
* <span data-ttu-id="6ef74-115">**C** ontroller: classi che:</span><span class="sxs-lookup"><span data-stu-id="6ef74-115">**C** ontrollers: Classes that:</span></span>
  * <span data-ttu-id="6ef74-116">Gestire le richieste del browser.</span><span class="sxs-lookup"><span data-stu-id="6ef74-116">Handle browser requests.</span></span>
  * <span data-ttu-id="6ef74-117">Recuperare i dati del modello.</span><span class="sxs-lookup"><span data-stu-id="6ef74-117">Retrieve model data.</span></span>
  * <span data-ttu-id="6ef74-118">Chiamare i modelli di visualizzazione che restituiscono una risposta.</span><span class="sxs-lookup"><span data-stu-id="6ef74-118">Call view templates that return a response.</span></span>

<span data-ttu-id="6ef74-119">In un'app MVC, la vista Visualizza solo le informazioni.</span><span class="sxs-lookup"><span data-stu-id="6ef74-119">In an MVC app, the view only displays information.</span></span> <span data-ttu-id="6ef74-120">Il controller gestisce e risponde all'input e all'interazione dell'utente.</span><span class="sxs-lookup"><span data-stu-id="6ef74-120">The controller handles and responds to user input and interaction.</span></span> <span data-ttu-id="6ef74-121">Il controller, ad esempio, gestisce i segmenti URL e i valori della stringa di query e passa questi valori al modello.</span><span class="sxs-lookup"><span data-stu-id="6ef74-121">For example, the controller handles URL segments and query-string values, and passes these values to the model.</span></span> <span data-ttu-id="6ef74-122">Il modello può usare questi valori per eseguire query sul database.</span><span class="sxs-lookup"><span data-stu-id="6ef74-122">The model might use these values to query the database.</span></span> <span data-ttu-id="6ef74-123">Ad esempio:</span><span class="sxs-lookup"><span data-stu-id="6ef74-123">For example:</span></span>

* <span data-ttu-id="6ef74-124">`https://localhost:5001/Home/Privacy`: specifica il `Home` controller e l' `Privacy` azione.</span><span class="sxs-lookup"><span data-stu-id="6ef74-124">`https://localhost:5001/Home/Privacy`: specifies the `Home` controller and the `Privacy` action.</span></span>
* <span data-ttu-id="6ef74-125">`https://localhost:5001/Movies/Edit/5`: è una richiesta di modifica del film con ID = 5 utilizzando il `Movies` controller e l' `Edit` azione, descritti in dettaglio più avanti nell'esercitazione.</span><span class="sxs-lookup"><span data-stu-id="6ef74-125">`https://localhost:5001/Movies/Edit/5`: is a request to edit the movie with ID=5 using the `Movies` controller and the `Edit` action, which are detailed later in the tutorial.</span></span>

<span data-ttu-id="6ef74-126">I dati della route vengono spiegati in seguito nell'esercitazione.</span><span class="sxs-lookup"><span data-stu-id="6ef74-126">Route data is explained later in the tutorial.</span></span>

<span data-ttu-id="6ef74-127">Il modello architetturale MVC separa un'app in tre gruppi principali di componenti, ovvero modelli, visualizzazioni e controller.</span><span class="sxs-lookup"><span data-stu-id="6ef74-127">The MVC architectural pattern separates an app into three main groups of components: Models, Views, and Controllers.</span></span> <span data-ttu-id="6ef74-128">Questo modello consente di ottenere la separazione dei problemi: la logica dell'interfaccia utente appartiene alla visualizzazione.</span><span class="sxs-lookup"><span data-stu-id="6ef74-128">This pattern helps to achieve separation of concerns: The UI logic belongs in the view.</span></span> <span data-ttu-id="6ef74-129">La logica di input risiede nel controller.</span><span class="sxs-lookup"><span data-stu-id="6ef74-129">Input logic belongs in the controller.</span></span> <span data-ttu-id="6ef74-130">La logica di business risiede nel modello.</span><span class="sxs-lookup"><span data-stu-id="6ef74-130">Business logic belongs in the model.</span></span> <span data-ttu-id="6ef74-131">Questa separazione consente di gestire la complessità quando si compila un'applicazione, perché consente di lavorare su un aspetto dell'implementazione alla volta senza influisca sul codice di un altro.</span><span class="sxs-lookup"><span data-stu-id="6ef74-131">This separation helps manage complexity when building an app, because it enables work on one aspect of the implementation at a time without impacting the code of another.</span></span> <span data-ttu-id="6ef74-132">Ad esempio, è possibile usare il codice della vista senza dipendere dal codice della logica di business.</span><span class="sxs-lookup"><span data-stu-id="6ef74-132">For example, you can work on the view code without depending on the business logic code.</span></span>

<span data-ttu-id="6ef74-133">Questi concetti sono introdotti e illustrati in questa serie di esercitazioni durante la compilazione di un'app di film.</span><span class="sxs-lookup"><span data-stu-id="6ef74-133">These concepts are introduced and demonstrated in this tutorial series while building a movie app.</span></span> <span data-ttu-id="6ef74-134">Il progetto MVC contiene le cartelle per i *controller* e le *viste*.</span><span class="sxs-lookup"><span data-stu-id="6ef74-134">The MVC project contains folders for the *Controllers* and *Views*.</span></span>

## <a name="add-a-controller"></a><span data-ttu-id="6ef74-135">Aggiunta di un controller</span><span class="sxs-lookup"><span data-stu-id="6ef74-135">Add a controller</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="6ef74-136">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="6ef74-136">Visual Studio</span></span>](#tab/visual-studio)

<span data-ttu-id="6ef74-137">Nella **Esplora soluzioni** fare clic con il pulsante destro del mouse su **controller > Aggiungi > controller**.</span><span class="sxs-lookup"><span data-stu-id="6ef74-137">In the **Solution Explorer**, right-click **Controllers > Add > Controller**.</span></span>

![Esplora soluzioni, fare clic con il pulsante destro del mouse su controller > Aggiungi > controller](~/tutorials/first-mvc-app/adding-controller/_static/add_controllercopyVS19v16.9.png)

<span data-ttu-id="6ef74-139">Nella finestra di dialogo **Aggiungi impalcatura** selezionare **controller MVC-vuoto**.</span><span class="sxs-lookup"><span data-stu-id="6ef74-139">In the **Add Scaffold** dialog box, select **MVC Controller - Empty**.</span></span>

![Aggiungere il controller MVC e assegnarli un nome](~/tutorials/first-mvc-app/adding-controller/_static/acCopyVS19v16.9.png)

<span data-ttu-id="6ef74-141">Nella **finestra di dialogo Aggiungi nuovo elemento-MvcMovie** immettere **HelloWorldController.cs** e selezionare **Aggiungi**.</span><span class="sxs-lookup"><span data-stu-id="6ef74-141">In the **Add New Item - MvcMovie dialog**, enter **HelloWorldController.cs** and select **Add**.</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="6ef74-142">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="6ef74-142">Visual Studio Code</span></span>](#tab/visual-studio-code)

<span data-ttu-id="6ef74-143">Selezionare l'icona **EXPLORER** e quindi fare CTRL+clic (clic con il pulsante destro del mouse) su **Controller > Nuovo File** e assegnare al nuovo file il nome *HelloWorldController.cs*.</span><span class="sxs-lookup"><span data-stu-id="6ef74-143">Select the **EXPLORER** icon and then control-click (right-click) **Controllers > New File** and name the new file *HelloWorldController.cs*.</span></span>

![Menu di scelta rapida](~/tutorials/first-mvc-app-xplat/adding-controller/_static/new_fileVSC1.51.png)

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="6ef74-145">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="6ef74-145">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

<span data-ttu-id="6ef74-146">In **Esplora soluzioni** fare clic con il pulsante destro del mouse su **Controller > Aggiungi > Nuovo file**.</span><span class="sxs-lookup"><span data-stu-id="6ef74-146">In **Solution Explorer**, right-click **Controllers > Add > New File**.</span></span>

![Menu di scelta rapida](~/tutorials/first-mvc-app-mac/adding-controller/_static/add_controller.png)

<span data-ttu-id="6ef74-148">Selezionare **ASP.NET Core** e **classe controller**.</span><span class="sxs-lookup"><span data-stu-id="6ef74-148">Select **ASP.NET Core** and **Controller Class**.</span></span>

<span data-ttu-id="6ef74-149">Assegnare il nome **HelloWorldController** al controller.</span><span class="sxs-lookup"><span data-stu-id="6ef74-149">Name the controller **HelloWorldController**.</span></span>

![Aggiungere il controller MVC e assegnarli un nome](~/tutorials/first-mvc-app-mac/adding-controller/_static/ac.png)

---

<span data-ttu-id="6ef74-151">Sostituire il contenuto di *Controllers/HelloWorldController.cs* con quanto segue:</span><span class="sxs-lookup"><span data-stu-id="6ef74-151">Replace the contents of *Controllers/HelloWorldController.cs* with the following:</span></span>

  [!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie/Controllers/HelloWorldController.cs?name=snippet_1)]

<span data-ttu-id="6ef74-152">È possibile chiamare ogni metodo `public` in un controller come endpoint HTTP.</span><span class="sxs-lookup"><span data-stu-id="6ef74-152">Every `public` method in a controller is callable as an HTTP endpoint.</span></span> <span data-ttu-id="6ef74-153">Nell'esempio precedente entrambi i metodi restituiscono una stringa.</span><span class="sxs-lookup"><span data-stu-id="6ef74-153">In the sample above, both methods return a string.</span></span> <span data-ttu-id="6ef74-154">Notare i commenti che precedono ogni metodo.</span><span class="sxs-lookup"><span data-stu-id="6ef74-154">Note the comments preceding each method.</span></span>

<span data-ttu-id="6ef74-155">Un endpoint HTTP:</span><span class="sxs-lookup"><span data-stu-id="6ef74-155">An HTTP endpoint:</span></span>

* <span data-ttu-id="6ef74-156">È un URL di destinazione nell'applicazione Web, ad esempio `https://localhost:5001/HelloWorld` .</span><span class="sxs-lookup"><span data-stu-id="6ef74-156">Is a targetable URL in the web application, such as `https://localhost:5001/HelloWorld`.</span></span>
* <span data-ttu-id="6ef74-157">Combina</span><span class="sxs-lookup"><span data-stu-id="6ef74-157">Combines:</span></span>
  * <span data-ttu-id="6ef74-158">Protocollo utilizzato: `HTTPS` .</span><span class="sxs-lookup"><span data-stu-id="6ef74-158">The protocol used: `HTTPS`.</span></span>
  * <span data-ttu-id="6ef74-159">Il percorso di rete del server Web, inclusa la porta TCP: `localhost:5001` .</span><span class="sxs-lookup"><span data-stu-id="6ef74-159">The network location of the web server, including the TCP port: `localhost:5001`.</span></span>
  * <span data-ttu-id="6ef74-160">URI di destinazione: `HelloWorld` .</span><span class="sxs-lookup"><span data-stu-id="6ef74-160">The target URI: `HelloWorld`.</span></span>

<span data-ttu-id="6ef74-161">Il primo commento indica che si tratta di un metodo [HTTP GET](https://developer.mozilla.org/docs/Web/HTTP/Methods/GET) che viene richiamato aggiungendo `/HelloWorld/` all'URL di base.</span><span class="sxs-lookup"><span data-stu-id="6ef74-161">The first comment states this is an [HTTP GET](https://developer.mozilla.org/docs/Web/HTTP/Methods/GET) method that's invoked by appending `/HelloWorld/` to the base URL.</span></span>

<span data-ttu-id="6ef74-162">Il secondo commento specifica un metodo [HTTP GET](https://developer.mozilla.org/docs/Web/HTTP/Methods) che viene richiamato aggiungendo `/HelloWorld/Welcome/` all'URL.</span><span class="sxs-lookup"><span data-stu-id="6ef74-162">The second comment specifies an [HTTP GET](https://developer.mozilla.org/docs/Web/HTTP/Methods) method that's invoked by appending `/HelloWorld/Welcome/` to the URL.</span></span> <span data-ttu-id="6ef74-163">Più avanti nell'esercitazione, il motore di ponteggi viene usato per generare `HTTP POST` metodi che aggiornano i dati.</span><span class="sxs-lookup"><span data-stu-id="6ef74-163">Later on in the tutorial, the scaffolding engine is used to generate `HTTP POST` methods, which update data.</span></span>

<span data-ttu-id="6ef74-164">Eseguire l'app senza il debugger.</span><span class="sxs-lookup"><span data-stu-id="6ef74-164">Run the app without the debugger.</span></span>

<span data-ttu-id="6ef74-165">Aggiungere "HelloWorld" al percorso nella barra degli indirizzi.</span><span class="sxs-lookup"><span data-stu-id="6ef74-165">Append "HelloWorld" to the path in the address bar.</span></span> <span data-ttu-id="6ef74-166">Il metodo `Index` restituisce una stringa.</span><span class="sxs-lookup"><span data-stu-id="6ef74-166">The `Index` method returns a string.</span></span>

![Finestra del browser con una risposta dell'app è l'azione predefinita](~/tutorials/first-mvc-app/adding-controller/_static/hell1.png)

<span data-ttu-id="6ef74-168">MVC richiama le classi controller e i metodi di azione al loro interno, a seconda dell'URL in ingresso.</span><span class="sxs-lookup"><span data-stu-id="6ef74-168">MVC invokes controller classes, and the action methods within them, depending on the incoming URL.</span></span> <span data-ttu-id="6ef74-169">La logica predefinita di [routing degli URL](xref:mvc/controllers/routing) usata da MVC usa un formato simile al seguente per determinare il codice da richiamare:</span><span class="sxs-lookup"><span data-stu-id="6ef74-169">The default [URL routing logic](xref:mvc/controllers/routing) used by MVC, uses a format like this to determine what code to invoke:</span></span>

`/[Controller]/[ActionName]/[Parameters]`

<span data-ttu-id="6ef74-170">Il formato per il routing viene impostato nel metodo `Configure` nel file *Startup.cs*.</span><span class="sxs-lookup"><span data-stu-id="6ef74-170">The routing format is set in the `Configure` method in *Startup.cs* file.</span></span>

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie3/Startup.cs?name=snippet_1&highlight=5)]

<span data-ttu-id="6ef74-171">Quando si sceglie l'app e non si forniscono i segmenti di URL, i valori predefiniti sono il controller "Home" e il metodo "Index" specificati nella riga del modello evidenziata sopra.</span><span class="sxs-lookup"><span data-stu-id="6ef74-171">When you browse to the app and don't supply any URL segments, it defaults to the "Home" controller and the "Index" method specified in the template line highlighted above.</span></span>  <span data-ttu-id="6ef74-172">Nei segmenti di URL precedenti:</span><span class="sxs-lookup"><span data-stu-id="6ef74-172">In the preceding URL segments:</span></span>

* <span data-ttu-id="6ef74-173">Il primo segmento di URL determina la classe di controller da eseguire.</span><span class="sxs-lookup"><span data-stu-id="6ef74-173">The first URL segment determines the controller class to run.</span></span> <span data-ttu-id="6ef74-174">`localhost:5001/HelloWorld` esegue quindi il mapping alla classe di controller **HelloWorld**.</span><span class="sxs-lookup"><span data-stu-id="6ef74-174">So `localhost:5001/HelloWorld` maps to the **HelloWorld** Controller class.</span></span>
* <span data-ttu-id="6ef74-175">La seconda parte del segmento URL determina il metodo di azione per la classe.</span><span class="sxs-lookup"><span data-stu-id="6ef74-175">The second part of the URL segment determines the action method on the class.</span></span> <span data-ttu-id="6ef74-176">`localhost:5001/HelloWorld/Index`Causa pertanto l' `Index` esecuzione del metodo della `HelloWorldController` classe.</span><span class="sxs-lookup"><span data-stu-id="6ef74-176">So `localhost:5001/HelloWorld/Index` causes the `Index` method of the `HelloWorldController` class to run.</span></span> <span data-ttu-id="6ef74-177">Si noti che è necessario solo passare a `localhost:5001/HelloWorld` e il metodo `Index` viene chiamato per impostazione predefinita.</span><span class="sxs-lookup"><span data-stu-id="6ef74-177">Notice that you only had to browse to `localhost:5001/HelloWorld` and the `Index` method was called by default.</span></span> <span data-ttu-id="6ef74-178">`Index` è il metodo predefinito che verrà chiamato su un controller se il nome di un metodo non è specificato in modo esplicito.</span><span class="sxs-lookup"><span data-stu-id="6ef74-178">`Index` is the default method that will be called on a controller if a method name isn't explicitly specified.</span></span>
* <span data-ttu-id="6ef74-179">La terza parte del segmento di URL ( `id`) è relativa ai dati di route.</span><span class="sxs-lookup"><span data-stu-id="6ef74-179">The third part of the URL segment ( `id`) is for route data.</span></span> <span data-ttu-id="6ef74-180">I dati della route vengono spiegati in seguito nell'esercitazione.</span><span class="sxs-lookup"><span data-stu-id="6ef74-180">Route data is explained later in the tutorial.</span></span>

<span data-ttu-id="6ef74-181">Passare a: `https://localhost:{PORT}/HelloWorld/Welcome` .</span><span class="sxs-lookup"><span data-stu-id="6ef74-181">Browse to: `https://localhost:{PORT}/HelloWorld/Welcome`.</span></span> <span data-ttu-id="6ef74-182">Sostituire `{PORT}` con il numero di porta.</span><span class="sxs-lookup"><span data-stu-id="6ef74-182">Replace `{PORT}` with your port number.</span></span>

<span data-ttu-id="6ef74-183">Il metodo `Welcome` viene eseguito e restituisce la stringa `This is the Welcome action method...`.</span><span class="sxs-lookup"><span data-stu-id="6ef74-183">The `Welcome` method runs and returns the string `This is the Welcome action method...`.</span></span> <span data-ttu-id="6ef74-184">Per questo URL, il controller è `HelloWorld` e il metodo di azione è `Welcome`.</span><span class="sxs-lookup"><span data-stu-id="6ef74-184">For this URL, the controller is `HelloWorld` and `Welcome` is the action method.</span></span> <span data-ttu-id="6ef74-185">Non è stata ancora usata la parte `[Parameters]` dell'URL.</span><span class="sxs-lookup"><span data-stu-id="6ef74-185">You haven't used the `[Parameters]` part of the URL yet.</span></span>

![Finestra del browser con una risposta dell'applicazione, This is the Welcome action method](~/tutorials/first-mvc-app/adding-controller/_static/welcome.png)

<span data-ttu-id="6ef74-187">Modificare il codice in modo da passare le informazioni dei parametri dall'URL al controller.</span><span class="sxs-lookup"><span data-stu-id="6ef74-187">Modify the code to pass some parameter information from the URL to the controller.</span></span> <span data-ttu-id="6ef74-188">Ad esempio: `/HelloWorld/Welcome?name=Rick&numtimes=4`.</span><span class="sxs-lookup"><span data-stu-id="6ef74-188">For example, `/HelloWorld/Welcome?name=Rick&numtimes=4`.</span></span>

<span data-ttu-id="6ef74-189">Modificare il metodo `Welcome` in modo da includere due parametri, come illustrato nel codice seguente.</span><span class="sxs-lookup"><span data-stu-id="6ef74-189">Change the `Welcome` method to include two parameters as shown in the following code.</span></span>

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie/Controllers/HelloWorldController.cs?name=snippet_2)]

<span data-ttu-id="6ef74-190">Il codice precedente:</span><span class="sxs-lookup"><span data-stu-id="6ef74-190">The preceding code:</span></span>

* <span data-ttu-id="6ef74-191">Usa la funzionalità di parametro facoltativo in C# per indicare che il parametro `numTimes` viene impostato come predefinito su 1, se non viene passato alcun valore.</span><span class="sxs-lookup"><span data-stu-id="6ef74-191">Uses the C# optional-parameter feature to indicate that the `numTimes` parameter defaults to 1 if no value is passed for that parameter.</span></span>
* <span data-ttu-id="6ef74-192">USA `HtmlEncoder.Default.Encode` per proteggere l'app da input dannoso, ad esempio tramite JavaScript.</span><span class="sxs-lookup"><span data-stu-id="6ef74-192">Uses `HtmlEncoder.Default.Encode` to protect the app from malicious input, such as through JavaScript.</span></span>
* <span data-ttu-id="6ef74-193">Usa [stringhe interpolate](/dotnet/articles/csharp/language-reference/keywords/interpolated-strings) in `$"Hello {name}, NumTimes is: {numTimes}"`.</span><span class="sxs-lookup"><span data-stu-id="6ef74-193">Uses [Interpolated Strings](/dotnet/articles/csharp/language-reference/keywords/interpolated-strings) in `$"Hello {name}, NumTimes is: {numTimes}"`.</span></span>

<span data-ttu-id="6ef74-194">Eseguire l'app e passare a: `https://localhost:{PORT}/HelloWorld/Welcome?name=Rick&numtimes=4` .</span><span class="sxs-lookup"><span data-stu-id="6ef74-194">Run the app and browse to: `https://localhost:{PORT}/HelloWorld/Welcome?name=Rick&numtimes=4`.</span></span> <span data-ttu-id="6ef74-195">Sostituire `{PORT}` con il numero di porta.</span><span class="sxs-lookup"><span data-stu-id="6ef74-195">Replace `{PORT}` with your port number.</span></span>

<span data-ttu-id="6ef74-196">Provare a usare valori diversi per `name` e `numtimes` nell'URL.</span><span class="sxs-lookup"><span data-stu-id="6ef74-196">Try different values for `name` and `numtimes` in the URL.</span></span> <span data-ttu-id="6ef74-197">Il sistema di [associazione di modelli](xref:mvc/models/model-binding) MVC esegue automaticamente il mapping dei parametri denominati dalla stringa di query ai parametri nel metodo.</span><span class="sxs-lookup"><span data-stu-id="6ef74-197">The MVC [model binding](xref:mvc/models/model-binding) system automatically maps the named parameters from the query string to parameters in the method.</span></span> <span data-ttu-id="6ef74-198">Per altre informazioni, vedere [Associazione di modelli](xref:mvc/models/model-binding).</span><span class="sxs-lookup"><span data-stu-id="6ef74-198">See [Model Binding](xref:mvc/models/model-binding) for more information.</span></span>

![Finestra del browser che mostra la risposta dell'applicazione Hello Rick, NumTimes è \: 4](~/tutorials/first-mvc-app/adding-controller/_static/rick4.png)

<span data-ttu-id="6ef74-200">Nell'immagine precedente:</span><span class="sxs-lookup"><span data-stu-id="6ef74-200">In the previous image:</span></span>

* <span data-ttu-id="6ef74-201">Il segmento URL `Parameters` non viene utilizzato.</span><span class="sxs-lookup"><span data-stu-id="6ef74-201">The URL segment `Parameters` isn't used.</span></span>
* <span data-ttu-id="6ef74-202">I `name` `numTimes` parametri e vengono passati nella [stringa di query](https://wikipedia.org/wiki/Query_string).</span><span class="sxs-lookup"><span data-stu-id="6ef74-202">The `name` and `numTimes` parameters are passed in the [query string](https://wikipedia.org/wiki/Query_string).</span></span>
* <span data-ttu-id="6ef74-203">`?`(Punto interrogativo) nell'URL precedente è un separatore e la stringa di query seguente.</span><span class="sxs-lookup"><span data-stu-id="6ef74-203">The `?` (question mark) in the above URL is a separator, and the query string follows.</span></span>
* <span data-ttu-id="6ef74-204">Il `&` carattere separa le coppie campo-valore.</span><span class="sxs-lookup"><span data-stu-id="6ef74-204">The `&` character separates field-value pairs.</span></span>

<span data-ttu-id="6ef74-205">Sostituire il metodo `Welcome` con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="6ef74-205">Replace the `Welcome` method with the following code:</span></span>

  [!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie/Controllers/HelloWorldController.cs?name=snippet_3)]

<span data-ttu-id="6ef74-206">Eseguire l'app e immettere l'URL seguente: `https://localhost:{PORT}/HelloWorld/Welcome/3?name=Rick`</span><span class="sxs-lookup"><span data-stu-id="6ef74-206">Run the app and enter the following URL: `https://localhost:{PORT}/HelloWorld/Welcome/3?name=Rick`</span></span>

<span data-ttu-id="6ef74-207">Nell'URL precedente:</span><span class="sxs-lookup"><span data-stu-id="6ef74-207">In the preceding URL:</span></span>

* <span data-ttu-id="6ef74-208">Il terzo segmento di URL corrisponde al parametro di route `id` .</span><span class="sxs-lookup"><span data-stu-id="6ef74-208">The third URL segment matched the route parameter `id`.</span></span> 
* <span data-ttu-id="6ef74-209">Il metodo `Welcome` contiene un parametro `id` che corrisponde al modello di URL nel metodo `MapControllerRoute`.</span><span class="sxs-lookup"><span data-stu-id="6ef74-209">The `Welcome` method contains a parameter `id` that matched the URL template in the `MapControllerRoute` method.</span></span>
* <span data-ttu-id="6ef74-210">Il carattere finale `?` (in `id?`) indica che il parametro `id` è facoltativo.</span><span class="sxs-lookup"><span data-stu-id="6ef74-210">The trailing `?` (in `id?`) indicates the `id` parameter is optional.</span></span>

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie5/Startup.cs?name=snippet_route&highlight=5)]

<span data-ttu-id="6ef74-211">Nell'esempio precedente:</span><span class="sxs-lookup"><span data-stu-id="6ef74-211">In the preceding example:</span></span>

* <span data-ttu-id="6ef74-212">Il terzo segmento di URL corrisponde al parametro di route `id` .</span><span class="sxs-lookup"><span data-stu-id="6ef74-212">The third URL segment matched the route parameter `id`.</span></span>
* <span data-ttu-id="6ef74-213">Il metodo `Welcome` contiene un parametro `id` che corrisponde al modello di URL nel metodo `MapControllerRoute`.</span><span class="sxs-lookup"><span data-stu-id="6ef74-213">The `Welcome` method contains a parameter `id` that matched the URL template in the `MapControllerRoute` method.</span></span>
* <span data-ttu-id="6ef74-214">Il carattere finale `?` (in `id?`) indica che il parametro `id` è facoltativo.</span><span class="sxs-lookup"><span data-stu-id="6ef74-214">The trailing `?` (in `id?`) indicates the `id` parameter is optional.</span></span>

> [!div class="step-by-step"]
> <span data-ttu-id="6ef74-215">[Precedente: introduzione](start-mvc.md) 
>  Passaggio [successivo: aggiungere una vista](adding-view.md)</span><span class="sxs-lookup"><span data-stu-id="6ef74-215">[Previous: Get Started](start-mvc.md)
[Next: Add a View](adding-view.md)</span></span>

::: moniker-end

::: moniker range="< aspnetcore-3.0"

<span data-ttu-id="6ef74-216">Il pattern architetturale Model-View-Controller (MVC) suddivide le app in tre componenti principali: **M** odello, **V** ista e **C** ontroller.</span><span class="sxs-lookup"><span data-stu-id="6ef74-216">The Model-View-Controller (MVC) architectural pattern separates an app into three main components: **M** odel, **V** iew, and **C** ontroller.</span></span> <span data-ttu-id="6ef74-217">Il pattern MVC consente di creare app più semplici da aggiornare e sottoporre a test rispetto alle app monolitiche tradizionali.</span><span class="sxs-lookup"><span data-stu-id="6ef74-217">The MVC pattern helps you create apps that are more testable and easier to update than traditional monolithic apps.</span></span> <span data-ttu-id="6ef74-218">Le app basate sul pattern MVC contengono:</span><span class="sxs-lookup"><span data-stu-id="6ef74-218">MVC-based apps contain:</span></span>

* <span data-ttu-id="6ef74-219">**M** odelli: classi che rappresentano i dati dell'app.</span><span class="sxs-lookup"><span data-stu-id="6ef74-219">**M** odels: Classes that represent the data of the app.</span></span> <span data-ttu-id="6ef74-220">Le classi modello usano la logica di convalida per applicare le regole business per tali dati.</span><span class="sxs-lookup"><span data-stu-id="6ef74-220">The model classes use validation logic to enforce business rules for that data.</span></span> <span data-ttu-id="6ef74-221">In genere, gli oggetti del modello recuperano e archiviano lo stato del modello in un database.</span><span class="sxs-lookup"><span data-stu-id="6ef74-221">Typically, model objects retrieve and store model state in a database.</span></span> <span data-ttu-id="6ef74-222">In questa esercitazione, un modello `Movie` recupera i dati dei film da un database, li fornisce alla vista o li aggiorna.</span><span class="sxs-lookup"><span data-stu-id="6ef74-222">In this tutorial, a `Movie` model retrieves movie data from a database, provides it to the view or updates it.</span></span> <span data-ttu-id="6ef74-223">I dati aggiornati vengono scritti in un database.</span><span class="sxs-lookup"><span data-stu-id="6ef74-223">Updated data is written to a database.</span></span>

* <span data-ttu-id="6ef74-224">**V** iste: componenti che visualizzano l'interfaccia utente dell'app.</span><span class="sxs-lookup"><span data-stu-id="6ef74-224">**V** iews: Views are the components that display the app's user interface (UI).</span></span> <span data-ttu-id="6ef74-225">In genere questa interfaccia utente presenta i dati del modello.</span><span class="sxs-lookup"><span data-stu-id="6ef74-225">Generally, this UI displays the model data.</span></span>

* <span data-ttu-id="6ef74-226">**C** ontroller: classi che gestiscono le richieste del browser.</span><span class="sxs-lookup"><span data-stu-id="6ef74-226">**C** ontrollers: Classes that handle browser requests.</span></span> <span data-ttu-id="6ef74-227">Recuperano i dati del modello e chiamano i modelli di vista che restituiscono una risposta.</span><span class="sxs-lookup"><span data-stu-id="6ef74-227">They retrieve model data and call view templates that return a response.</span></span> <span data-ttu-id="6ef74-228">In un'applicazione MVC, la vista presenta solo le informazioni; il controller gestisce e risponde all'input dell'utente e alle azioni di interazione.</span><span class="sxs-lookup"><span data-stu-id="6ef74-228">In an MVC app, the view only displays information; the controller handles and responds to user input and interaction.</span></span> <span data-ttu-id="6ef74-229">Ad esempio, il controller gestisce i valori dei dati della route e della stringa di query e li passa al modello.</span><span class="sxs-lookup"><span data-stu-id="6ef74-229">For example, the controller handles route data and query-string values, and passes these values to the model.</span></span> <span data-ttu-id="6ef74-230">Il modello può usare questi valori per eseguire query sul database.</span><span class="sxs-lookup"><span data-stu-id="6ef74-230">The model might use these values to query the database.</span></span> <span data-ttu-id="6ef74-231">Ad esempio, `https://localhost:5001/Home/About` contiene come i dati della route `Home` (il controller) e `About` (il metodo di azione da chiamare sul controller Home).</span><span class="sxs-lookup"><span data-stu-id="6ef74-231">For example, `https://localhost:5001/Home/About` has route data of `Home` (the controller) and `About` (the action method to call on the home controller).</span></span> <span data-ttu-id="6ef74-232">`https://localhost:5001/Movies/Edit/5` è una richiesta di modificare il film con ID = 5 usando il controller di film.</span><span class="sxs-lookup"><span data-stu-id="6ef74-232">`https://localhost:5001/Movies/Edit/5` is a request to edit the movie with ID=5 using the movie controller.</span></span> <span data-ttu-id="6ef74-233">I dati della route vengono spiegati in seguito nell'esercitazione.</span><span class="sxs-lookup"><span data-stu-id="6ef74-233">Route data is explained later in the tutorial.</span></span>

<span data-ttu-id="6ef74-234">Il pattern MVC consente di creare app che separano i diversi aspetti dell'app (logica di input, logica di business e logica dell'interfaccia utente), fornendo un accoppiamento libero tra questi elementi.</span><span class="sxs-lookup"><span data-stu-id="6ef74-234">The MVC pattern helps you create apps that separate the different aspects of the app (input logic, business logic, and UI logic), while providing a loose coupling between these elements.</span></span> <span data-ttu-id="6ef74-235">Il pattern specifica il punto in cui deve risiedere ogni tipo di logica nell'app.</span><span class="sxs-lookup"><span data-stu-id="6ef74-235">The pattern specifies where each kind of logic should be located in the app.</span></span> <span data-ttu-id="6ef74-236">La logica dell'interfaccia utente risiede nella vista.</span><span class="sxs-lookup"><span data-stu-id="6ef74-236">The UI logic belongs in the view.</span></span> <span data-ttu-id="6ef74-237">La logica di input risiede nel controller.</span><span class="sxs-lookup"><span data-stu-id="6ef74-237">Input logic belongs in the controller.</span></span> <span data-ttu-id="6ef74-238">La logica di business risiede nel modello.</span><span class="sxs-lookup"><span data-stu-id="6ef74-238">Business logic belongs in the model.</span></span> <span data-ttu-id="6ef74-239">Questa separazione semplifica la complessità in fase di compilazione di un'app perché consente di lavorare su un aspetto dell'implementazione alla volta senza conseguenze sul codice di un altro aspetto.</span><span class="sxs-lookup"><span data-stu-id="6ef74-239">This separation helps you manage complexity when you build an app, because it enables you to work on one aspect of the implementation at a time without impacting the code of another.</span></span> <span data-ttu-id="6ef74-240">Ad esempio, è possibile usare il codice della vista senza dipendere dal codice della logica di business.</span><span class="sxs-lookup"><span data-stu-id="6ef74-240">For example, you can work on the view code without depending on the business logic code.</span></span>

<span data-ttu-id="6ef74-241">Questi concetti vengono illustrati in questa serie di esercitazioni e descrivono come compilare un'app per i film.</span><span class="sxs-lookup"><span data-stu-id="6ef74-241">We cover these concepts in this tutorial series and show you how to use them to build a movie app.</span></span> <span data-ttu-id="6ef74-242">Il progetto MVC contiene le cartelle per i *controller* e le *viste*.</span><span class="sxs-lookup"><span data-stu-id="6ef74-242">The MVC project contains folders for the *Controllers* and *Views*.</span></span>

## <a name="add-a-controller"></a><span data-ttu-id="6ef74-243">Aggiunta di un controller</span><span class="sxs-lookup"><span data-stu-id="6ef74-243">Add a controller</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="6ef74-244">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="6ef74-244">Visual Studio</span></span>](#tab/visual-studio)

* <span data-ttu-id="6ef74-245">In **Esplora soluzioni** fare clic con il pulsante destro del mouse su **controller > Aggiungi > controller**</span><span class="sxs-lookup"><span data-stu-id="6ef74-245">In **Solution Explorer**, right-click **Controllers > Add > Controller**</span></span>

  ![Menu di scelta rapida](~/tutorials/first-mvc-app/adding-controller/_static/add_controller.png)

* <span data-ttu-id="6ef74-247">Nella finestra di dialogo **Aggiungi scaffolding**, selezionare **Controller MVC - vuoto**</span><span class="sxs-lookup"><span data-stu-id="6ef74-247">In the **Add Scaffold** dialog box, select **MVC Controller - Empty**</span></span>

  ![Aggiungere il controller MVC e assegnarli un nome](~/tutorials/first-mvc-app/adding-controller/_static/ac.png)

* <span data-ttu-id="6ef74-249">Nella **finestra di dialogo Aggiungi Controller MVC vuoto**, immettere **HelloWorldController** e selezionare **AGGIUNGI**.</span><span class="sxs-lookup"><span data-stu-id="6ef74-249">In the **Add Empty MVC Controller dialog**, enter **HelloWorldController** and select **ADD**.</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="6ef74-250">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="6ef74-250">Visual Studio Code</span></span>](#tab/visual-studio-code)

<span data-ttu-id="6ef74-251">Selezionare l'icona **EXPLORER** e quindi fare CTRL+clic (clic con il pulsante destro del mouse) su **Controller > Nuovo File** e assegnare al nuovo file il nome *HelloWorldController.cs*.</span><span class="sxs-lookup"><span data-stu-id="6ef74-251">Select the **EXPLORER** icon and then control-click (right-click) **Controllers > New File** and name the new file *HelloWorldController.cs*.</span></span>

  ![Menu di scelta rapida](~/tutorials/first-mvc-app-xplat/adding-controller/_static/new_file.png)

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="6ef74-253">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="6ef74-253">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

<span data-ttu-id="6ef74-254">In **Esplora soluzioni** fare clic con il pulsante destro del mouse su **Controller > Aggiungi > Nuovo file**.</span><span class="sxs-lookup"><span data-stu-id="6ef74-254">In **Solution Explorer**, right-click **Controllers > Add > New File**.</span></span>

![Menu di scelta rapida](~/tutorials/first-mvc-app-mac/adding-controller/_static/add_controller.png)

<span data-ttu-id="6ef74-256">Selezionare **ASP.NET Core** e **Classe controller MVC**.</span><span class="sxs-lookup"><span data-stu-id="6ef74-256">Select **ASP.NET Core** and **MVC Controller Class**.</span></span>

<span data-ttu-id="6ef74-257">Assegnare il nome **HelloWorldController** al controller.</span><span class="sxs-lookup"><span data-stu-id="6ef74-257">Name the controller **HelloWorldController**.</span></span>

![Aggiungere il controller MVC e assegnarli un nome](~/tutorials/first-mvc-app-mac/adding-controller/_static/ac.png)

---

<span data-ttu-id="6ef74-259">Sostituire il contenuto di *Controllers/HelloWorldController.cs* con quanto segue:</span><span class="sxs-lookup"><span data-stu-id="6ef74-259">Replace the contents of *Controllers/HelloWorldController.cs* with the following:</span></span>

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie/Controllers/HelloWorldController.cs?name=snippet_1)]

<span data-ttu-id="6ef74-260">È possibile chiamare ogni metodo `public` in un controller come endpoint HTTP.</span><span class="sxs-lookup"><span data-stu-id="6ef74-260">Every `public` method in a controller is callable as an HTTP endpoint.</span></span> <span data-ttu-id="6ef74-261">Nell'esempio precedente entrambi i metodi restituiscono una stringa.</span><span class="sxs-lookup"><span data-stu-id="6ef74-261">In the sample above, both methods return a string.</span></span> <span data-ttu-id="6ef74-262">Notare i commenti che precedono ogni metodo.</span><span class="sxs-lookup"><span data-stu-id="6ef74-262">Note the comments preceding each method.</span></span>

<span data-ttu-id="6ef74-263">Un endpoint HTTP è un URL definibile come destinazione nell'applicazione Web, ad esempio `https://localhost:5001/HelloWorld`, e combina il protocollo usato (`HTTPS`), il percorso di rete del server Web, tra cui la porta TCP (`localhost:5001`) e l'URI di destinazione (`HelloWorld`).</span><span class="sxs-lookup"><span data-stu-id="6ef74-263">An HTTP endpoint is a targetable URL in the web application, such as `https://localhost:5001/HelloWorld`, and combines the protocol used: `HTTPS`, the network location of the web server (including the TCP port): `localhost:5001` and the target URI `HelloWorld`.</span></span>

<span data-ttu-id="6ef74-264">Il primo commento indica che si tratta di un metodo [HTTP GET](https://www.w3schools.com/tags/ref_httpmethods.asp) che viene richiamato aggiungendo `/HelloWorld/` all'URL di base.</span><span class="sxs-lookup"><span data-stu-id="6ef74-264">The first comment states this is an [HTTP GET](https://www.w3schools.com/tags/ref_httpmethods.asp) method that's invoked by appending `/HelloWorld/` to the base URL.</span></span> <span data-ttu-id="6ef74-265">Il secondo commento specifica un metodo [HTTP GET](https://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html) che viene richiamato aggiungendo `/HelloWorld/Welcome/` all'URL.</span><span class="sxs-lookup"><span data-stu-id="6ef74-265">The second comment specifies an [HTTP GET](https://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html) method that's invoked by appending `/HelloWorld/Welcome/` to the URL.</span></span> <span data-ttu-id="6ef74-266">In una fase successiva dell'esercitazione il motore di scaffolding viene usato per generare i metodi `HTTP POST` che aggiornano i dati.</span><span class="sxs-lookup"><span data-stu-id="6ef74-266">Later on in the tutorial the scaffolding engine is used to generate `HTTP POST` methods which update data.</span></span>

<span data-ttu-id="6ef74-267">Eseguire l'app in modalità non di debug e aggiungere "HelloWorld" al percorso nella barra degli indirizzi.</span><span class="sxs-lookup"><span data-stu-id="6ef74-267">Run the app in non-debug mode and append "HelloWorld" to the path in the address bar.</span></span> <span data-ttu-id="6ef74-268">Il metodo `Index` restituisce una stringa.</span><span class="sxs-lookup"><span data-stu-id="6ef74-268">The `Index` method returns a string.</span></span>

![Finestra del browser con una risposta dell'applicazione, This is my default action](~/tutorials/first-mvc-app/adding-controller/_static/hell1.png)

<span data-ttu-id="6ef74-270">MVC richiama le classi controller, e i metodi di azione in esse contenute, in base all'URL in ingresso.</span><span class="sxs-lookup"><span data-stu-id="6ef74-270">MVC invokes controller classes (and the action methods within them) depending on the incoming URL.</span></span> <span data-ttu-id="6ef74-271">Il valore predefinito della [logica di routing degli URL](xref:mvc/controllers/routing) usata da MVC usa un formato simile al seguente per determinare il codice di richiamare:</span><span class="sxs-lookup"><span data-stu-id="6ef74-271">The default [URL routing logic](xref:mvc/controllers/routing) used by MVC uses a format like this to determine what code to invoke:</span></span>

`/[Controller]/[ActionName]/[Parameters]`

<span data-ttu-id="6ef74-272">Il formato per il routing viene impostato nel metodo `Configure` nel file *Startup.cs*.</span><span class="sxs-lookup"><span data-stu-id="6ef74-272">The routing format is set in the `Configure` method in *Startup.cs* file.</span></span>

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie/Startup.cs?name=snippet_1&highlight=5)]

<!-- 
Add link to explain lambda.
Remove link for simplified tutorial.
-->

<span data-ttu-id="6ef74-273">Quando si sceglie l'app e non si forniscono i segmenti di URL, i valori predefiniti sono il controller "Home" e il metodo "Index" specificati nella riga del modello evidenziata sopra.</span><span class="sxs-lookup"><span data-stu-id="6ef74-273">When you browse to the app and don't supply any URL segments, it defaults to the "Home" controller and the "Index" method specified in the template line highlighted above.</span></span>

<span data-ttu-id="6ef74-274">Il primo segmento di URL determina la classe di controller da eseguire.</span><span class="sxs-lookup"><span data-stu-id="6ef74-274">The first URL segment determines the controller class to run.</span></span> <span data-ttu-id="6ef74-275">`localhost:{PORT}/HelloWorld` esegue quindi il mapping alla classe `HelloWorldController`.</span><span class="sxs-lookup"><span data-stu-id="6ef74-275">So `localhost:{PORT}/HelloWorld` maps to the `HelloWorldController` class.</span></span> <span data-ttu-id="6ef74-276">La seconda parte del segmento URL determina il metodo di azione per la classe.</span><span class="sxs-lookup"><span data-stu-id="6ef74-276">The second part of the URL segment determines the action method on the class.</span></span> <span data-ttu-id="6ef74-277">Pertanto `localhost:{PORT}/HelloWorld/Index` causa l'esecuzione del metodo `Index` della classe `HelloWorldController`.</span><span class="sxs-lookup"><span data-stu-id="6ef74-277">So `localhost:{PORT}/HelloWorld/Index` would cause the `Index` method of the `HelloWorldController` class to run.</span></span> <span data-ttu-id="6ef74-278">Si noti che è necessario solo passare a `localhost:{PORT}/HelloWorld` e il metodo `Index` viene chiamato per impostazione predefinita.</span><span class="sxs-lookup"><span data-stu-id="6ef74-278">Notice that you only had to browse to `localhost:{PORT}/HelloWorld` and the `Index` method was called by default.</span></span> <span data-ttu-id="6ef74-279">In questo modo `Index` è il metodo predefinito che verrà chiamato per un controller, se non viene specificato un nome di metodo in modo esplicito.</span><span class="sxs-lookup"><span data-stu-id="6ef74-279">This is because `Index` is the default method that will be called on a controller if a method name isn't explicitly specified.</span></span> <span data-ttu-id="6ef74-280">La terza parte del segmento di URL ( `id`) è relativa ai dati di route.</span><span class="sxs-lookup"><span data-stu-id="6ef74-280">The third part of the URL segment ( `id`) is for route data.</span></span> <span data-ttu-id="6ef74-281">I dati della route vengono spiegati in seguito nell'esercitazione.</span><span class="sxs-lookup"><span data-stu-id="6ef74-281">Route data is explained later in the tutorial.</span></span>

<span data-ttu-id="6ef74-282">Passare a `https://localhost:{PORT}/HelloWorld/Welcome`.</span><span class="sxs-lookup"><span data-stu-id="6ef74-282">Browse to `https://localhost:{PORT}/HelloWorld/Welcome`.</span></span> <span data-ttu-id="6ef74-283">Il metodo `Welcome` viene eseguito e restituisce la stringa `This is the Welcome action method...`.</span><span class="sxs-lookup"><span data-stu-id="6ef74-283">The `Welcome` method runs and returns the string `This is the Welcome action method...`.</span></span> <span data-ttu-id="6ef74-284">Per questo URL, il controller è `HelloWorld` e il metodo di azione è `Welcome`.</span><span class="sxs-lookup"><span data-stu-id="6ef74-284">For this URL, the controller is `HelloWorld` and `Welcome` is the action method.</span></span> <span data-ttu-id="6ef74-285">Non è stata ancora usata la parte `[Parameters]` dell'URL.</span><span class="sxs-lookup"><span data-stu-id="6ef74-285">You haven't used the `[Parameters]` part of the URL yet.</span></span>

![Finestra del browser con una risposta dell'applicazione, This is the Welcome action method](~/tutorials/first-mvc-app/adding-controller/_static/welcome.png)

<span data-ttu-id="6ef74-287">Modificare il codice in modo da passare le informazioni dei parametri dall'URL al controller.</span><span class="sxs-lookup"><span data-stu-id="6ef74-287">Modify the code to pass some parameter information from the URL to the controller.</span></span> <span data-ttu-id="6ef74-288">Ad esempio: `/HelloWorld/Welcome?name=Rick&numtimes=4`.</span><span class="sxs-lookup"><span data-stu-id="6ef74-288">For example, `/HelloWorld/Welcome?name=Rick&numtimes=4`.</span></span> <span data-ttu-id="6ef74-289">Modificare il metodo `Welcome` in modo da includere due parametri, come illustrato nel codice seguente.</span><span class="sxs-lookup"><span data-stu-id="6ef74-289">Change the `Welcome` method to include two parameters as shown in the following code.</span></span>

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie/Controllers/HelloWorldController.cs?name=snippet_2)]

<span data-ttu-id="6ef74-290">Il codice precedente:</span><span class="sxs-lookup"><span data-stu-id="6ef74-290">The preceding code:</span></span>

* <span data-ttu-id="6ef74-291">Usa la funzionalità di parametro facoltativo in C# per indicare che il parametro `numTimes` viene impostato come predefinito su 1, se non viene passato alcun valore.</span><span class="sxs-lookup"><span data-stu-id="6ef74-291">Uses the C# optional-parameter feature to indicate that the `numTimes` parameter defaults to 1 if no value is passed for that parameter.</span></span> <!-- remove for simplified -->
* <span data-ttu-id="6ef74-292">Usa `HtmlEncoder.Default.Encode` per proteggere le app da input dannoso, in particolare in JavaScript.</span><span class="sxs-lookup"><span data-stu-id="6ef74-292">Uses `HtmlEncoder.Default.Encode` to protect the app from malicious input (namely JavaScript).</span></span>
* <span data-ttu-id="6ef74-293">Usa [stringhe interpolate](/dotnet/articles/csharp/language-reference/keywords/interpolated-strings) in `$"Hello {name}, NumTimes is: {numTimes}"`.</span><span class="sxs-lookup"><span data-stu-id="6ef74-293">Uses [Interpolated Strings](/dotnet/articles/csharp/language-reference/keywords/interpolated-strings) in `$"Hello {name}, NumTimes is: {numTimes}"`.</span></span> <!-- remove for simplified -->

<span data-ttu-id="6ef74-294">Eseguire l'app e passare a:</span><span class="sxs-lookup"><span data-stu-id="6ef74-294">Run the app and browse to:</span></span>

   `https://localhost:{PORT}/HelloWorld/Welcome?name=Rick&numtimes=4`

<span data-ttu-id="6ef74-295">Sostituire `{PORT}` con il numero di porta. È possibile provare valori diversi per `name` e `numtimes` nell'URL.</span><span class="sxs-lookup"><span data-stu-id="6ef74-295">(Replace `{PORT}` with your port number.) You can try different values for `name` and `numtimes` in the URL.</span></span> <span data-ttu-id="6ef74-296">Il sistema di [associazione di modelli](xref:mvc/models/model-binding) MVC esegue il mapping dei parametri denominati dalla stringa di query nella barra degli indirizzi ai parametri nel metodo.</span><span class="sxs-lookup"><span data-stu-id="6ef74-296">The MVC [model binding](xref:mvc/models/model-binding) system automatically maps the named parameters from the query string in the address bar to parameters in your method.</span></span> <span data-ttu-id="6ef74-297">Per altre informazioni, vedere [Associazione di modelli](xref:mvc/models/model-binding).</span><span class="sxs-lookup"><span data-stu-id="6ef74-297">See [Model Binding](xref:mvc/models/model-binding) for more information.</span></span>

![Finestra del browser che mostra la risposta dell'applicazione Hello Rick, NumTimes è \: 4](~/tutorials/first-mvc-app/adding-controller/_static/rick4.png)

<span data-ttu-id="6ef74-299">Nell'immagine precedente non viene usato il segmento di URL ( `Parameters` ), i `name` `numTimes` parametri e vengono passati nella [stringa di query](https://wikipedia.org/wiki/Query_string).</span><span class="sxs-lookup"><span data-stu-id="6ef74-299">In the image above, the URL segment (`Parameters`) isn't used, the `name` and `numTimes` parameters are passed in the [query string](https://wikipedia.org/wiki/Query_string).</span></span> <span data-ttu-id="6ef74-300">`?`(Punto interrogativo) nell'URL precedente è un separatore e la stringa di query seguente.</span><span class="sxs-lookup"><span data-stu-id="6ef74-300">The `?` (question mark) in the above URL is a separator, and the query string follows.</span></span> <span data-ttu-id="6ef74-301">Il `&` carattere separa le coppie campo-valore.</span><span class="sxs-lookup"><span data-stu-id="6ef74-301">The `&` character separates field-value pairs.</span></span>

<span data-ttu-id="6ef74-302">Sostituire il metodo `Welcome` con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="6ef74-302">Replace the `Welcome` method with the following code:</span></span>

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie/Controllers/HelloWorldController.cs?name=snippet_3)]

<span data-ttu-id="6ef74-303">Eseguire l'app e immettere l'URL seguente: `https://localhost:{PORT}/HelloWorld/Welcome/3?name=Rick`</span><span class="sxs-lookup"><span data-stu-id="6ef74-303">Run the app and enter the following URL: `https://localhost:{PORT}/HelloWorld/Welcome/3?name=Rick`</span></span>

<span data-ttu-id="6ef74-304">Questa volta il terzo segmento di URL corrisponde al parametro di route `id`.</span><span class="sxs-lookup"><span data-stu-id="6ef74-304">This time the third URL segment matched the route parameter `id`.</span></span> <span data-ttu-id="6ef74-305">Il metodo `Welcome` contiene un parametro `id` che corrisponde al modello di URL nel metodo `MapRoute`.</span><span class="sxs-lookup"><span data-stu-id="6ef74-305">The `Welcome` method contains a parameter `id` that matched the URL template in the `MapRoute` method.</span></span> <span data-ttu-id="6ef74-306">Il carattere finale `?` (in `id?`) indica che il parametro `id` è facoltativo.</span><span class="sxs-lookup"><span data-stu-id="6ef74-306">The trailing `?` (in `id?`) indicates the `id` parameter is optional.</span></span>

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie/Startup.cs?name=snippet_1&highlight=5)]

<span data-ttu-id="6ef74-307">In questi esempi il controller rappresenta la parte "VC" di MVC, ovvero il ruolo di vista e controller.</span><span class="sxs-lookup"><span data-stu-id="6ef74-307">In these examples the controller has been doing the "VC" portion of MVC - that is, the view and controller work.</span></span> <span data-ttu-id="6ef74-308">Il controller restituisce direttamente l'HTML.</span><span class="sxs-lookup"><span data-stu-id="6ef74-308">The controller is returning HTML directly.</span></span> <span data-ttu-id="6ef74-309">In genere è preferibile che i controller non restituiscano direttamente l'HTML, dal momento che la codifica e la gestione sono molto complesse.</span><span class="sxs-lookup"><span data-stu-id="6ef74-309">Generally you don't want controllers returning HTML directly, since that becomes very cumbersome to code and maintain.</span></span> <span data-ttu-id="6ef74-310">Viene invece usato in genere un Razor file modello di vista separato per generare la risposta HTML.</span><span class="sxs-lookup"><span data-stu-id="6ef74-310">Instead you typically use a separate Razor view template file to help generate the HTML response.</span></span> <span data-ttu-id="6ef74-311">Questa operazione viene eseguita nell'esercitazione successiva.</span><span class="sxs-lookup"><span data-stu-id="6ef74-311">You do that in the next tutorial.</span></span>

> [!div class="step-by-step"]
> <span data-ttu-id="6ef74-312">[Precedente](start-mvc.md) 
>  [Avanti](adding-view.md)</span><span class="sxs-lookup"><span data-stu-id="6ef74-312">[Previous](start-mvc.md)
[Next](adding-view.md)</span></span>

::: moniker-end
