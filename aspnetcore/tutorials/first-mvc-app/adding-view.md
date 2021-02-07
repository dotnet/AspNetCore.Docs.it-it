---
title: "Parte 3: aggiungere una visualizzazione a un'app MVC ASP.NET Core"
author: rick-anderson
description: Parte 3 della serie di esercitazioni su ASP.NET Core MVC.
ms.author: riande
ms.date: 01/28/2021
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
uid: tutorials/first-mvc-app/adding-view
ms.custom: contperf-fy21q3
ms.openlocfilehash: 1542e44fcc6d0ae22fb1a759ea3a3ed1d866cbc7
ms.sourcegitcommit: 19a004ff2be73876a9ef0f1ac44d0331849ad159
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/07/2021
ms.locfileid: "99804569"
---
# <a name="part-3-add-a-view-to-an-aspnet-core-mvc-app"></a><span data-ttu-id="873b4-103">Parte 3: aggiungere una visualizzazione a un'app MVC ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="873b4-103">Part 3, add a view to an ASP.NET Core MVC app</span></span>

<span data-ttu-id="873b4-104">Autore: [Rick Anderson](https://twitter.com/RickAndMSFT)</span><span class="sxs-lookup"><span data-stu-id="873b4-104">By [Rick Anderson](https://twitter.com/RickAndMSFT)</span></span>

::: moniker range=">= aspnetcore-3.0"

<span data-ttu-id="873b4-105">In questa sezione viene modificata la `HelloWorldController` classe per l'utilizzo dei [Razor](xref:mvc/views/razor) file di visualizzazione.</span><span class="sxs-lookup"><span data-stu-id="873b4-105">In this section, you modify the `HelloWorldController` class to use [Razor](xref:mvc/views/razor) view files.</span></span> <span data-ttu-id="873b4-106">Questa operazione Incapsula in modo semplice il processo di generazione di risposte HTML a un client.</span><span class="sxs-lookup"><span data-stu-id="873b4-106">This cleanly encapsulates the process of generating HTML responses to a client.</span></span>

<span data-ttu-id="873b4-107">I modelli di visualizzazione vengono creati utilizzando Razor .</span><span class="sxs-lookup"><span data-stu-id="873b4-107">View templates are created using Razor.</span></span> <span data-ttu-id="873b4-108">Razormodelli di vista basati su:</span><span class="sxs-lookup"><span data-stu-id="873b4-108">Razor-based view templates:</span></span>

* <span data-ttu-id="873b4-109">Avere un'estensione di file *. cshtml* .</span><span class="sxs-lookup"><span data-stu-id="873b4-109">Have a *.cshtml* file extension.</span></span>
* <span data-ttu-id="873b4-110">Consente di creare in modo elegante l'output HTML con C#.</span><span class="sxs-lookup"><span data-stu-id="873b4-110">Provide an elegant way to create HTML output with C#.</span></span>

<span data-ttu-id="873b4-111">Attualmente il `Index` metodo restituisce una stringa con un messaggio nella classe controller.</span><span class="sxs-lookup"><span data-stu-id="873b4-111">Currently the `Index` method returns a string with a message in the controller class.</span></span> <span data-ttu-id="873b4-112">Nella classe `HelloWorldController` sostituire il metodo `Index` con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="873b4-112">In the `HelloWorldController` class, replace the `Index` method with the following code:</span></span>

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie/Controllers/HelloWorldController.cs?name=snippet_4)]

<span data-ttu-id="873b4-113">Il codice precedente:</span><span class="sxs-lookup"><span data-stu-id="873b4-113">The preceding code:</span></span>

* <span data-ttu-id="873b4-114">Chiama il metodo del controller <xref:Microsoft.AspNetCore.Mvc.Controller.View*> .</span><span class="sxs-lookup"><span data-stu-id="873b4-114">Calls the controller's <xref:Microsoft.AspNetCore.Mvc.Controller.View*> method.</span></span>
* <span data-ttu-id="873b4-115">Usa un modello di visualizzazione per generare una risposta HTML.</span><span class="sxs-lookup"><span data-stu-id="873b4-115">Uses a view template to generate an HTML response.</span></span>

<span data-ttu-id="873b4-116">Metodi controller:</span><span class="sxs-lookup"><span data-stu-id="873b4-116">Controller methods:</span></span>

* <span data-ttu-id="873b4-117">Sono denominati *metodi di azione*.</span><span class="sxs-lookup"><span data-stu-id="873b4-117">Are referred to as *action methods*.</span></span>  <span data-ttu-id="873b4-118">Ad esempio, il `Index` metodo di azione del codice precedente.</span><span class="sxs-lookup"><span data-stu-id="873b4-118">For example, the `Index` action method in the preceding code.</span></span>
* <span data-ttu-id="873b4-119">Generalmente restituisce un oggetto <xref:Microsoft.AspNetCore.Mvc.IActionResult> o una classe derivata da <xref:Microsoft.AspNetCore.Mvc.ActionResult> , non un tipo come `string` .</span><span class="sxs-lookup"><span data-stu-id="873b4-119">Generally return an <xref:Microsoft.AspNetCore.Mvc.IActionResult> or a class derived from <xref:Microsoft.AspNetCore.Mvc.ActionResult>, not a type like `string`.</span></span>

## <a name="add-a-view"></a><span data-ttu-id="873b4-120">Aggiungere una vista</span><span class="sxs-lookup"><span data-stu-id="873b4-120">Add a view</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="873b4-121">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="873b4-121">Visual Studio</span></span>](#tab/visual-studio)

<span data-ttu-id="873b4-122">Fare clic con il pulsante destro del mouse sulla cartella *views* , quindi **aggiungere > nuova cartella** e denominare la cartella *HelloWorld*.</span><span class="sxs-lookup"><span data-stu-id="873b4-122">Right-click on the *Views* folder, and then **Add > New Folder** and name the folder *HelloWorld*.</span></span>

<span data-ttu-id="873b4-123">Fare clic con il pulsante destro del mouse sulla cartella *views/HelloWorld* , quindi **aggiungere > nuovo elemento**.</span><span class="sxs-lookup"><span data-stu-id="873b4-123">Right-click on the *Views/HelloWorld* folder, and then **Add > New Item**.</span></span>

<span data-ttu-id="873b4-124">Nella finestra di dialogo **Aggiungi nuovo elemento-MvcMovie** :</span><span class="sxs-lookup"><span data-stu-id="873b4-124">In the **Add New Item - MvcMovie** dialog:</span></span>

* <span data-ttu-id="873b4-125">Nella casella di ricerca in alto a destra immettere *view* (vista)</span><span class="sxs-lookup"><span data-stu-id="873b4-125">In the search box in the upper-right, enter *view*</span></span>
* <span data-ttu-id="873b4-126">Selezione **Razor visualizzazione**</span><span class="sxs-lookup"><span data-stu-id="873b4-126">Select **Razor View**</span></span>
* <span data-ttu-id="873b4-127">Mantenere il valore della casella **Nome**, *Index.cshtml*.</span><span class="sxs-lookup"><span data-stu-id="873b4-127">Keep the **Name** box value, *Index.cshtml*.</span></span>
* <span data-ttu-id="873b4-128">Selezionare **Aggiungi**</span><span class="sxs-lookup"><span data-stu-id="873b4-128">Select **Add**</span></span>

![Finestra di dialogo Aggiungi nuovo elemento](adding-view/_static/add_view50.png)

# <a name="visual-studio-code"></a>[<span data-ttu-id="873b4-130">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="873b4-130">Visual Studio Code</span></span>](#tab/visual-studio-code)

<span data-ttu-id="873b4-131">Aggiungere una `Index` vista per `HelloWorldController` :</span><span class="sxs-lookup"><span data-stu-id="873b4-131">Add an `Index` view for the `HelloWorldController`:</span></span>

* <span data-ttu-id="873b4-132">Aggiungere una nuova cartella denominata *Views/HelloWorld*.</span><span class="sxs-lookup"><span data-stu-id="873b4-132">Add a new folder named *Views/HelloWorld*.</span></span>
* <span data-ttu-id="873b4-133">Aggiungere un nuovo file al nome di cartella *Views/HelloWorld\*\*Index.cshtml*.</span><span class="sxs-lookup"><span data-stu-id="873b4-133">Add a new file to the *Views/HelloWorld* folder name *Index.cshtml*.</span></span>

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="873b4-134">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="873b4-134">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

<span data-ttu-id="873b4-135">Control: fare clic sulla cartella *views* , quindi **aggiungere > nuova cartella** e denominare la cartella *HelloWorld*.</span><span class="sxs-lookup"><span data-stu-id="873b4-135">Control-click on the *Views* folder, and then **Add > New Folder** and name the folder *HelloWorld*.</span></span>

<span data-ttu-id="873b4-136">Control-fare clic sulla cartella *views/HelloWorld* , quindi **aggiungere > nuovo file**.</span><span class="sxs-lookup"><span data-stu-id="873b4-136">Control-click on the *Views/HelloWorld* folder, and then **Add > New File**.</span></span>

<span data-ttu-id="873b4-137">Nel finestra di dialogo **Nuovo file**:</span><span class="sxs-lookup"><span data-stu-id="873b4-137">In the **New File** dialog:</span></span>

* <span data-ttu-id="873b4-138">Selezionare **ASP.NET Core** nel riquadro sinistro.</span><span class="sxs-lookup"><span data-stu-id="873b4-138">Select **ASP.NET Core** in the left pane.</span></span>
* <span data-ttu-id="873b4-139">Selezionare **Razor Visualizza-vuoto** nel riquadro centrale.</span><span class="sxs-lookup"><span data-stu-id="873b4-139">Select **Razor View - Empty** in the center pane.</span></span>
* <span data-ttu-id="873b4-140">Digitare *index* nella casella **nome** .</span><span class="sxs-lookup"><span data-stu-id="873b4-140">Type *Index* in the **Name** box.</span></span>
* <span data-ttu-id="873b4-141">Selezionare **Nuovo**.</span><span class="sxs-lookup"><span data-stu-id="873b4-141">Select **New**.</span></span>

  ![Finestra di dialogo Aggiungi nuovo elemento](adding-view/_static/add_view_macVSM8.9.png)

---

<span data-ttu-id="873b4-143">Sostituire il contenuto del file di visualizzazione *views/HelloWorld/index. cshtml* Razor con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="873b4-143">Replace the contents of the *Views/HelloWorld/Index.cshtml* Razor view file with the following:</span></span>

[!code-cshtml[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie22/Views/HelloWorld/Index1.cshtml?highlight=7)]

<span data-ttu-id="873b4-144">Passa a `https://localhost:{PORT}/HelloWorld` :</span><span class="sxs-lookup"><span data-stu-id="873b4-144">Navigate to `https://localhost:{PORT}/HelloWorld`:</span></span>

* <span data-ttu-id="873b4-145">Il `Index` metodo in ha `HelloWorldController` eseguito l'istruzione `return View();` , che ha specificato che il metodo deve usare un file di modello di visualizzazione per eseguire il rendering di una risposta nel browser.</span><span class="sxs-lookup"><span data-stu-id="873b4-145">The `Index` method in the `HelloWorldController` ran the statement `return View();`, which specified that the method should use a view template file to render a response to the browser.</span></span>
* <span data-ttu-id="873b4-146">Il nome del file di modello di visualizzazione non è stato specificato, quindi MVC ha utilizzato il file di visualizzazione predefinito.</span><span class="sxs-lookup"><span data-stu-id="873b4-146">A view template file name wasn't specified, so MVC defaulted to using the default view file.</span></span> <span data-ttu-id="873b4-147">Quando il nome del file di visualizzazione non è specificato, viene restituita la visualizzazione predefinita.</span><span class="sxs-lookup"><span data-stu-id="873b4-147">When the view file name isn't specified, the default view is returned.</span></span> <span data-ttu-id="873b4-148">In questo esempio, la visualizzazione predefinita ha lo stesso nome del metodo di azione `Index` .</span><span class="sxs-lookup"><span data-stu-id="873b4-148">The default view has the same name as the action method, `Index` in this example.</span></span> <span data-ttu-id="873b4-149">Viene utilizzato il modello di visualizzazione */views/HelloWorld/index.cshtml* .</span><span class="sxs-lookup"><span data-stu-id="873b4-149">The view template */Views/HelloWorld/Index.cshtml* is used.</span></span>
* <span data-ttu-id="873b4-150">La figura seguente mostra la stringa "Hello from the view template!"</span><span class="sxs-lookup"><span data-stu-id="873b4-150">The following image shows the string "Hello from our View Template!"</span></span> <span data-ttu-id="873b4-151">hardcoded nella visualizzazione:</span><span class="sxs-lookup"><span data-stu-id="873b4-151">hard-coded in the view:</span></span>

  ![Finestra del browser](~/tutorials/first-mvc-app/adding-view/_static/hell_template.png)

## <a name="change-views-and-layout-pages"></a><span data-ttu-id="873b4-153">Modificare le viste e le pagine di layout</span><span class="sxs-lookup"><span data-stu-id="873b4-153">Change views and layout pages</span></span>

<span data-ttu-id="873b4-154">Selezionare il menu collegamenti **MvcMovie**, **Home** e **privacy**.</span><span class="sxs-lookup"><span data-stu-id="873b4-154">Select the menu links **MvcMovie**, **Home**, and **Privacy**.</span></span> <span data-ttu-id="873b4-155">Ogni pagina mostra lo stesso layout di menu.</span><span class="sxs-lookup"><span data-stu-id="873b4-155">Each page shows the same menu layout.</span></span> <span data-ttu-id="873b4-156">Il layout di menu viene implementato nel file *Views/Shared/_Layout.cshtml*.</span><span class="sxs-lookup"><span data-stu-id="873b4-156">The menu layout is implemented in the *Views/Shared/_Layout.cshtml* file.</span></span>

<span data-ttu-id="873b4-157">Aprire il file *Views/Shared/_Layout.cshtml*.</span><span class="sxs-lookup"><span data-stu-id="873b4-157">Open the *Views/Shared/_Layout.cshtml* file.</span></span>

<span data-ttu-id="873b4-158">I modelli di [layout](xref:mvc/views/layout) consentono:</span><span class="sxs-lookup"><span data-stu-id="873b4-158">[Layout](xref:mvc/views/layout) templates allows:</span></span>

* <span data-ttu-id="873b4-159">Specifica del layout del contenitore HTML di un sito in un'unica posizione.</span><span class="sxs-lookup"><span data-stu-id="873b4-159">Specifying the HTML container layout of a site in one place.</span></span>
* <span data-ttu-id="873b4-160">Applicazione del layout del contenitore HTML in più pagine del sito.</span><span class="sxs-lookup"><span data-stu-id="873b4-160">Applying the HTML container layout across multiple pages in the site.</span></span>

<span data-ttu-id="873b4-161">Trovare la riga `@RenderBody()`.</span><span class="sxs-lookup"><span data-stu-id="873b4-161">Find the `@RenderBody()` line.</span></span> <span data-ttu-id="873b4-162">`RenderBody` è un segnaposto dove tutte le pagine specifiche della vista vengono presentate, *incapsulate* nella pagina di layout.</span><span class="sxs-lookup"><span data-stu-id="873b4-162">`RenderBody` is a placeholder where all the view-specific pages you create show up, *wrapped* in the layout page.</span></span> <span data-ttu-id="873b4-163">Ad esempio, se si seleziona il collegamento **Privacy**, il rendering della vista **Views/Home/Privacy.cshtml** viene eseguito all'interno del metodo `RenderBody`.</span><span class="sxs-lookup"><span data-stu-id="873b4-163">For example, if you select the **Privacy** link, the **Views/Home/Privacy.cshtml** view is rendered inside the `RenderBody` method.</span></span>

## <a name="change-the-title-footer-and-menu-link-in-the-layout-file"></a><span data-ttu-id="873b4-164">Modificare il titolo, il piè di pagina e il collegamento di menu nel file di layout</span><span class="sxs-lookup"><span data-stu-id="873b4-164">Change the title, footer, and menu link in the layout file</span></span>

<span data-ttu-id="873b4-165">Sostituire il contenuto del file *Views/Shared/_Layout. cshtml* con il markup seguente.</span><span class="sxs-lookup"><span data-stu-id="873b4-165">Replace the content of the *Views/Shared/_Layout.cshtml* file with the following markup.</span></span> <span data-ttu-id="873b4-166">Le modifiche sono evidenziate:</span><span class="sxs-lookup"><span data-stu-id="873b4-166">The changes are highlighted:</span></span>
::: moniker-end

::: moniker range=">= aspnetcore-3.0 < aspnetcore-5.0"

[!code-cshtml[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie3/Views/Shared/_Layout.cshtml?highlight=6,14,40)]

::: moniker-end

::: moniker range=">= aspnetcore-5.0"

[!code-cshtml[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie5/Views/Shared/_Layout.cshtml?highlight=6,14,40)]

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

<span data-ttu-id="873b4-167">Il markup precedente ha apportato le modifiche seguenti:</span><span class="sxs-lookup"><span data-stu-id="873b4-167">The preceding markup made the following changes:</span></span>

* <span data-ttu-id="873b4-168">Tre occorrenze di `MvcMovie` a `Movie App` .</span><span class="sxs-lookup"><span data-stu-id="873b4-168">Three occurrences of `MvcMovie` to `Movie App`.</span></span>
* <span data-ttu-id="873b4-169">L'elemento di ancoraggio `<a class="navbar-brand" asp-area="" asp-controller="Home" asp-action="Index">MvcMovie</a>` è stato sostituito con `<a class="navbar-brand" asp-controller="Movies" asp-action="Index">Movie App</a>`.</span><span class="sxs-lookup"><span data-stu-id="873b4-169">The anchor element `<a class="navbar-brand" asp-area="" asp-controller="Home" asp-action="Index">MvcMovie</a>` to `<a class="navbar-brand" asp-controller="Movies" asp-action="Index">Movie App</a>`.</span></span>

<span data-ttu-id="873b4-170">Nel markup precedente, l' `asp-area=""` [attributo dell'helper tag di ancoraggio](xref:mvc/views/tag-helpers/builtin-th/anchor-tag-helper) e il valore dell'attributo sono stati omessi perché l'app non usa le [aree](xref:mvc/controllers/areas).</span><span class="sxs-lookup"><span data-stu-id="873b4-170">In the preceding markup, the `asp-area=""` [anchor Tag Helper attribute](xref:mvc/views/tag-helpers/builtin-th/anchor-tag-helper) and attribute value was omitted because this app isn't using [Areas](xref:mvc/controllers/areas).</span></span>

<span data-ttu-id="873b4-171">**Nota**: il `Movies` controller non è stato implementato.</span><span class="sxs-lookup"><span data-stu-id="873b4-171">**Note**: The `Movies` controller hasn't been implemented.</span></span> <span data-ttu-id="873b4-172">A questo punto, il `Movie App` collegamento non funziona.</span><span class="sxs-lookup"><span data-stu-id="873b4-172">At this point, the `Movie App` link isn't functional.</span></span>

<span data-ttu-id="873b4-173">Salvare le modifiche e selezionare il collegamento per la **privacy** .</span><span class="sxs-lookup"><span data-stu-id="873b4-173">Save the changes and select the **Privacy** link.</span></span> <span data-ttu-id="873b4-174">Si noti come il titolo sulla scheda del browser visualizzi **Privacy Policy - Movie App** anziché **Privacy Policy - Mvc Movie**:</span><span class="sxs-lookup"><span data-stu-id="873b4-174">Notice how the title on the browser tab displays **Privacy Policy - Movie App** instead of **Privacy Policy - Mvc Movie**:</span></span>

![Scheda Privacy](~/tutorials/first-mvc-app/adding-view/_static/privacy50.png)

<span data-ttu-id="873b4-176">Selezionare il collegamento **Home**.</span><span class="sxs-lookup"><span data-stu-id="873b4-176">Select the **Home** link.</span></span>

<span data-ttu-id="873b4-177">Si noti che il titolo e il testo di ancoraggio visualizzano l' **app Movie**.</span><span class="sxs-lookup"><span data-stu-id="873b4-177">Notice that the title and anchor text display **Movie App**.</span></span> <span data-ttu-id="873b4-178">Le modifiche sono state apportate una sola volta nel modello di layout e tutte le pagine del sito riflettono il nuovo testo del collegamento e il nuovo titolo.</span><span class="sxs-lookup"><span data-stu-id="873b4-178">The changes were made once in the layout template and all pages on the site reflect the new link text and new title.</span></span>

<span data-ttu-id="873b4-179">Esaminare il file *Views/_ViewStart.cshtml*:</span><span class="sxs-lookup"><span data-stu-id="873b4-179">Examine the *Views/_ViewStart.cshtml* file:</span></span>

```cshtml
@{
    Layout = "_Layout";
}
```

<span data-ttu-id="873b4-180">Il file *Views/_ViewStart.cshtml* attiva il file *Views/Shared/_Layout.cshtml* per ogni vista.</span><span class="sxs-lookup"><span data-stu-id="873b4-180">The *Views/_ViewStart.cshtml* file brings in the *Views/Shared/_Layout.cshtml* file to each view.</span></span> <span data-ttu-id="873b4-181">È possibile usare la proprietà `Layout` per impostare una vista di layout differente oppure impostarla su `null` e quindi non verrà usato alcun file di layout.</span><span class="sxs-lookup"><span data-stu-id="873b4-181">The `Layout` property can be used to set a different layout view, or set it to `null` so no layout file will be used.</span></span>

<span data-ttu-id="873b4-182">Aprire il file di visualizzazione *views/HelloWorld/index. cshtml* .</span><span class="sxs-lookup"><span data-stu-id="873b4-182">Open the *Views/HelloWorld/Index.cshtml* view file.</span></span>

<span data-ttu-id="873b4-183">Modificare il titolo e l' `<h2>` elemento come evidenziato nell'esempio seguente:</span><span class="sxs-lookup"><span data-stu-id="873b4-183">Change the title and `<h2>` element as highlighted in the following:</span></span>

[!code-cshtml[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie/Views/HelloWorld/Index2.cshtml?highlight=2,5)]

<span data-ttu-id="873b4-184">Il titolo e l' `<h2>` elemento sono leggermente diversi, quindi è chiaro quale parte del codice cambia la visualizzazione.</span><span class="sxs-lookup"><span data-stu-id="873b4-184">The title and `<h2>` element are slightly different so it's clear which part of the code changes the display.</span></span>

<span data-ttu-id="873b4-185">`ViewData["Title"] = "Movie List";` nel codice precedente imposta la proprietà `Title` del dizionario `ViewData` su "Movie List".</span><span class="sxs-lookup"><span data-stu-id="873b4-185">`ViewData["Title"] = "Movie List";` in the code above sets the `Title` property of the `ViewData` dictionary to "Movie List".</span></span> <span data-ttu-id="873b4-186">La proprietà `Title` viene usata nell'elemento HTML `<title>` nella pagina di layout:</span><span class="sxs-lookup"><span data-stu-id="873b4-186">The `Title` property is used in the `<title>` HTML element in the layout page:</span></span>

```cshtml
<title>@ViewData["Title"] - Movie App</title>
```

<span data-ttu-id="873b4-187">Salvare la modifica e passare a `https://localhost:{PORT}/HelloWorld`.</span><span class="sxs-lookup"><span data-stu-id="873b4-187">Save the change and navigate to `https://localhost:{PORT}/HelloWorld`.</span></span>

<span data-ttu-id="873b4-188">Si noti che sono stati modificati i seguenti elementi:</span><span class="sxs-lookup"><span data-stu-id="873b4-188">Notice that the following have changed:</span></span>

* <span data-ttu-id="873b4-189">Titolo del browser.</span><span class="sxs-lookup"><span data-stu-id="873b4-189">Browser title.</span></span>
* <span data-ttu-id="873b4-190">Intestazione primaria.</span><span class="sxs-lookup"><span data-stu-id="873b4-190">Primary heading.</span></span>
* <span data-ttu-id="873b4-191">Intestazioni secondarie.</span><span class="sxs-lookup"><span data-stu-id="873b4-191">Secondary headings.</span></span>

<span data-ttu-id="873b4-192">Se non sono presenti modifiche nel browser, è possibile che venga memorizzato nella cache il contenuto visualizzato.</span><span class="sxs-lookup"><span data-stu-id="873b4-192">If there are no changes in the browser, it could be cached content that is being viewed.</span></span> <span data-ttu-id="873b4-193">Premere CTRL + F5 nel browser per forzare il caricamento della risposta dal server.</span><span class="sxs-lookup"><span data-stu-id="873b4-193">Press Ctrl+F5 in the browser to force the response from the server to be loaded.</span></span> <span data-ttu-id="873b4-194">Il titolo del browser viene creato con il valore `ViewData["Title"]` impostato nel modello di vista *Index.cshtml* e la stringa "- Movie App" aggiuntiva viene aggiunta nel file di layout.</span><span class="sxs-lookup"><span data-stu-id="873b4-194">The browser title is created with `ViewData["Title"]` we set in the *Index.cshtml* view template and the additional "- Movie App" added in the layout file.</span></span>

<span data-ttu-id="873b4-195">Il contenuto del modello di vista *Index.cshtml* viene unito al modello di vista *Views/Shared/_Layout.cshtml*.</span><span class="sxs-lookup"><span data-stu-id="873b4-195">The content in the *Index.cshtml* view template is merged with the *Views/Shared/_Layout.cshtml* view template.</span></span> <span data-ttu-id="873b4-196">Viene inviata una singola risposta HTML al browser.</span><span class="sxs-lookup"><span data-stu-id="873b4-196">A single HTML response is sent to the browser.</span></span> <span data-ttu-id="873b4-197">I modelli di layout rendono semplice apportare modifiche che si applicano a tutte le pagine di un'app.</span><span class="sxs-lookup"><span data-stu-id="873b4-197">Layout templates make it easy to make changes that apply across all of the pages in an app.</span></span> <span data-ttu-id="873b4-198">Per altre informazioni, vedere [layout](xref:mvc/views/layout).</span><span class="sxs-lookup"><span data-stu-id="873b4-198">To learn more, see [Layout](xref:mvc/views/layout).</span></span>

![Vista dell'elenco di film](~/tutorials/first-mvc-app/adding-view/_static/hell50.png)

<span data-ttu-id="873b4-200">Il piccolo bit di "dati", il "Hello from the view template!"</span><span class="sxs-lookup"><span data-stu-id="873b4-200">The small bit of "data", the "Hello from our View Template!"</span></span> <span data-ttu-id="873b4-201">Message, è tuttavia hardcoded.</span><span class="sxs-lookup"><span data-stu-id="873b4-201">message, is hard-coded however.</span></span> <span data-ttu-id="873b4-202">L'applicazione MVC contiene una "V" (visualizzazione), una "C" (controller), ma non ancora "M" (modello).</span><span class="sxs-lookup"><span data-stu-id="873b4-202">The MVC application has a "V" (view), a "C" (controller), but no "M" (model) yet.</span></span>

## <a name="passing-data-from-the-controller-to-the-view"></a><span data-ttu-id="873b4-203">Passaggio di dati dal controller alla vista</span><span class="sxs-lookup"><span data-stu-id="873b4-203">Passing Data from the Controller to the View</span></span>

<span data-ttu-id="873b4-204">Le azioni del controller vengono richiamate in risposta a una richiesta URL in ingresso.</span><span class="sxs-lookup"><span data-stu-id="873b4-204">Controller actions are invoked in response to an incoming URL request.</span></span> <span data-ttu-id="873b4-205">Una classe controller è il punto in cui viene scritto il codice che gestisce le richieste in ingresso del browser.</span><span class="sxs-lookup"><span data-stu-id="873b4-205">A controller class is where the code is written that handles the incoming browser requests.</span></span> <span data-ttu-id="873b4-206">Il controller recupera i dati da un'origine dati e determina il tipo di risposta da inviare al browser.</span><span class="sxs-lookup"><span data-stu-id="873b4-206">The controller retrieves data from a data source and decides what type of response to send back to the browser.</span></span> <span data-ttu-id="873b4-207">I modelli di vista possono essere usati da un controller per generare e formattare una risposta HTML al browser.</span><span class="sxs-lookup"><span data-stu-id="873b4-207">View templates can be used from a controller to generate and format an HTML response to the browser.</span></span>

<span data-ttu-id="873b4-208">Il compito dei controller è fornire i dati necessari affinché un modello di vista possa eseguire il rendering di una risposta.</span><span class="sxs-lookup"><span data-stu-id="873b4-208">Controllers are responsible for providing the data required in order for a view template to render a response.</span></span>

<span data-ttu-id="873b4-209">I modelli di visualizzazione **non** devono:</span><span class="sxs-lookup"><span data-stu-id="873b4-209">View templates should **not**:</span></span>

* <span data-ttu-id="873b4-210">Logica di business</span><span class="sxs-lookup"><span data-stu-id="873b4-210">Do business logic</span></span>
* <span data-ttu-id="873b4-211">Interagire direttamente con un database.</span><span class="sxs-lookup"><span data-stu-id="873b4-211">Interact with a database directly.</span></span>

<span data-ttu-id="873b4-212">Un modello di visualizzazione dovrebbe funzionare solo con i dati forniti dal controller.</span><span class="sxs-lookup"><span data-stu-id="873b4-212">A view template should work only with the data that's provided to it by the controller.</span></span> <span data-ttu-id="873b4-213">Il mantenimento di questa "separazione dei problemi" consente di mantenere il codice:</span><span class="sxs-lookup"><span data-stu-id="873b4-213">Maintaining this "separation of concerns" helps keep the code:</span></span>

* <span data-ttu-id="873b4-214">Pulito.</span><span class="sxs-lookup"><span data-stu-id="873b4-214">Clean.</span></span>
* <span data-ttu-id="873b4-215">Verificabile.</span><span class="sxs-lookup"><span data-stu-id="873b4-215">Testable.</span></span>
* <span data-ttu-id="873b4-216">Gestibile.</span><span class="sxs-lookup"><span data-stu-id="873b4-216">Maintainable.</span></span>

<span data-ttu-id="873b4-217">Attualmente il metodo `Welcome` nella classe `HelloWorldController` accetta un parametro `name` e `ID` e quindi genera i valori direttamente al browser.</span><span class="sxs-lookup"><span data-stu-id="873b4-217">Currently, the `Welcome` method in the `HelloWorldController` class takes a `name` and a `ID` parameter and then outputs the values directly to the browser.</span></span>

<span data-ttu-id="873b4-218">Invece di ottenere il rendering di questa risposta come stringa, cambiare il controller in modo che usi un modello di vista.</span><span class="sxs-lookup"><span data-stu-id="873b4-218">Rather than have the controller render this response as a string, change the controller to use a view template instead.</span></span> <span data-ttu-id="873b4-219">Il modello di visualizzazione genera una risposta dinamica, il che significa che i dati appropriati devono essere passati dal controller alla visualizzazione per generare la risposta.</span><span class="sxs-lookup"><span data-stu-id="873b4-219">The view template generates a dynamic response, which means that appropriate data must be passed from the controller to the view to generate the response.</span></span> <span data-ttu-id="873b4-220">A tale scopo, fare in modo che il controller inserisca i dati dinamici (parametri) necessari per il modello di visualizzazione in un `ViewData` dizionario.</span><span class="sxs-lookup"><span data-stu-id="873b4-220">Do this by having the controller put the dynamic data (parameters) that the view template needs in a `ViewData` dictionary.</span></span> <span data-ttu-id="873b4-221">Il modello di visualizzazione può quindi accedere ai dati dinamici.</span><span class="sxs-lookup"><span data-stu-id="873b4-221">The view template can then access the dynamic data.</span></span>

<span data-ttu-id="873b4-222">In *HelloWorldController.cs*, modificare il metodo `Welcome` in modo da aggiungere un valore `Message` e `NumTimes` al dizionario `ViewData`.</span><span class="sxs-lookup"><span data-stu-id="873b4-222">In *HelloWorldController.cs*, change the `Welcome` method to add a `Message` and `NumTimes` value to the `ViewData` dictionary.</span></span>

<span data-ttu-id="873b4-223">Il `ViewData` dizionario è un oggetto dinamico, ovvero è possibile usare qualsiasi tipo.</span><span class="sxs-lookup"><span data-stu-id="873b4-223">The `ViewData` dictionary is a dynamic object, which means any type can be used.</span></span> <span data-ttu-id="873b4-224">L' `ViewData` oggetto non dispone di proprietà definite fino a quando non viene aggiunto alcun elemento.</span><span class="sxs-lookup"><span data-stu-id="873b4-224">The `ViewData` object has no defined properties until something is added.</span></span> <span data-ttu-id="873b4-225">Il [sistema di associazione di modelli MVC](xref:mvc/models/model-binding) esegue automaticamente il mapping dei parametri denominati `name` e `numTimes` dalla stringa di query ai parametri nel metodo.</span><span class="sxs-lookup"><span data-stu-id="873b4-225">The [MVC model binding system](xref:mvc/models/model-binding) automatically maps the named parameters `name` and `numTimes` from the query string to parameters in the method.</span></span> <span data-ttu-id="873b4-226">Il completamento `HelloWorldController` :</span><span class="sxs-lookup"><span data-stu-id="873b4-226">The complete `HelloWorldController`:</span></span>

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie/Controllers/HelloWorldController.cs?name=snippet_5&highlight=13-19)]

<span data-ttu-id="873b4-227">L'oggetto di dizionario `ViewData` contiene i dati che verranno passati alla vista.</span><span class="sxs-lookup"><span data-stu-id="873b4-227">The `ViewData` dictionary object contains data that will be passed to the view.</span></span>

<span data-ttu-id="873b4-228">Creare un modello di vista Welcome denominato *Views/HelloWorld/Welcome.cshtml*.</span><span class="sxs-lookup"><span data-stu-id="873b4-228">Create a Welcome view template named *Views/HelloWorld/Welcome.cshtml*.</span></span>

<span data-ttu-id="873b4-229">Si creerà un ciclo nel modello di vista *Welcome.cshtml* che visualizza la stringa "Hello" `NumTimes`.</span><span class="sxs-lookup"><span data-stu-id="873b4-229">You'll create a loop in the *Welcome.cshtml* view template that displays "Hello" `NumTimes`.</span></span> <span data-ttu-id="873b4-230">Sostituire il contenuto di *Views/HelloWorld/Welcome.cshtml* con quanto segue:</span><span class="sxs-lookup"><span data-stu-id="873b4-230">Replace the contents of *Views/HelloWorld/Welcome.cshtml* with the following:</span></span>

[!code-cshtml[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie/Views/HelloWorld/Welcome.cshtml)]

<span data-ttu-id="873b4-231">Salvare le modifiche e passare all'URL seguente:</span><span class="sxs-lookup"><span data-stu-id="873b4-231">Save your changes and browse to the following URL:</span></span>

`https://localhost:{PORT}/HelloWorld/Welcome?name=Rick&numtimes=4`

<span data-ttu-id="873b4-232">I dati vengono ricavati dall'URL e passati al controller usando lo strumento di [associazione di modelli MVC](xref:mvc/models/model-binding).</span><span class="sxs-lookup"><span data-stu-id="873b4-232">Data is taken from the URL and passed to the controller using the [MVC model binder](xref:mvc/models/model-binding).</span></span> <span data-ttu-id="873b4-233">Il controller crea un pacchetto di dati in un dizionario `ViewData` e passa tale oggetto alla vista.</span><span class="sxs-lookup"><span data-stu-id="873b4-233">The controller packages the data into a `ViewData` dictionary and passes that object to the view.</span></span> <span data-ttu-id="873b4-234">La vista esegue quindi il rendering dei dati in formato HTML al browser.</span><span class="sxs-lookup"><span data-stu-id="873b4-234">The view then renders the data as HTML to the browser.</span></span>

![Vista Privacy con un'etichetta di benvenuto e la frase Hello Rick riportata quattro volte](~/tutorials/first-mvc-app/adding-view/_static/rick2_50.png)

<span data-ttu-id="873b4-236">Nell'esempio precedente, il `ViewData` dizionario è stato usato per passare i dati dal controller a una visualizzazione.</span><span class="sxs-lookup"><span data-stu-id="873b4-236">In the preceding sample, the `ViewData` dictionary was used to pass data from the controller to a view.</span></span> <span data-ttu-id="873b4-237">Più avanti nell'esercitazione si userà un modello di vista per passare i dati da un controller a una vista.</span><span class="sxs-lookup"><span data-stu-id="873b4-237">Later in the tutorial, a view model is used to pass data from a controller to a view.</span></span> <span data-ttu-id="873b4-238">L'approccio al modello di visualizzazione per il passaggio dei dati è preferibile rispetto all' `ViewData` approccio del dizionario.</span><span class="sxs-lookup"><span data-stu-id="873b4-238">The view model approach to passing data is preferred over the `ViewData` dictionary approach.</span></span>

<span data-ttu-id="873b4-239">Nella prossima esercitazione, viene creato un database di film.</span><span class="sxs-lookup"><span data-stu-id="873b4-239">In the next tutorial, a database of movies is created.</span></span>

> [!div class="step-by-step"]
> <span data-ttu-id="873b4-240">[Precedente: aggiungere un controller](adding-controller.md) 
>  Passaggio [successivo: aggiungere un modello](adding-model.md)</span><span class="sxs-lookup"><span data-stu-id="873b4-240">[Previous: Add a Controller](adding-controller.md)
[Next: Add a Model](adding-model.md)</span></span>

::: moniker-end

::: moniker range="< aspnetcore-3.0"

<span data-ttu-id="873b4-241">In questa sezione, la `HelloWorldController` classe viene modificata per usare i [Razor](xref:mvc/views/razor) file di visualizzazione per incapsulare in modo semplice il processo di generazione di risposte HTML a un client.</span><span class="sxs-lookup"><span data-stu-id="873b4-241">In this section, the `HelloWorldController` class is modified to use [Razor](xref:mvc/views/razor) view files to cleanly encapsulate the process of generating HTML responses to a client.</span></span>

<span data-ttu-id="873b4-242">Si crea un file di modello di visualizzazione utilizzando Razor .</span><span class="sxs-lookup"><span data-stu-id="873b4-242">You create a view template file using Razor.</span></span> <span data-ttu-id="873b4-243">Razori modelli di vista basati su hanno un'estensione di file *. cshtml* .</span><span class="sxs-lookup"><span data-stu-id="873b4-243">Razor-based view templates have a *.cshtml* file extension.</span></span> <span data-ttu-id="873b4-244">Consentono di creare l'output HTML in modo accurato con C#.</span><span class="sxs-lookup"><span data-stu-id="873b4-244">They provide an elegant way to create HTML output with C#.</span></span>

<span data-ttu-id="873b4-245">Attualmente il metodo `Index` restituisce una stringa con un messaggio hardcoded nella classe controller.</span><span class="sxs-lookup"><span data-stu-id="873b4-245">Currently the `Index` method returns a string with a message that's hard-coded in the controller class.</span></span> <span data-ttu-id="873b4-246">Nella classe `HelloWorldController` sostituire il metodo `Index` con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="873b4-246">In the `HelloWorldController` class, replace the `Index` method with the following code:</span></span>

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie/Controllers/HelloWorldController.cs?name=snippet_4)]

<span data-ttu-id="873b4-247">Il codice precedente chiama il metodo <xref:Microsoft.AspNetCore.Mvc.Controller.View*> del controller.</span><span class="sxs-lookup"><span data-stu-id="873b4-247">The preceding code calls the controller's <xref:Microsoft.AspNetCore.Mvc.Controller.View*> method.</span></span> <span data-ttu-id="873b4-248">Usa un modello di visualizzazione per generare una risposta HTML.</span><span class="sxs-lookup"><span data-stu-id="873b4-248">It uses a view template to generate an HTML response.</span></span> <span data-ttu-id="873b4-249">I metodi del controller, noti anche come *metodi di azione*, ad esempio il metodo `Index` descritto in precedenza, restituiscono in genere un <xref:Microsoft.AspNetCore.Mvc.IActionResult> (o una classe derivata da <xref:Microsoft.AspNetCore.Mvc.ActionResult>) e non un tipo come `string`.</span><span class="sxs-lookup"><span data-stu-id="873b4-249">Controller methods (also known as *action methods*), such as the `Index` method above, generally return an <xref:Microsoft.AspNetCore.Mvc.IActionResult> (or a class derived from <xref:Microsoft.AspNetCore.Mvc.ActionResult>), not a type like `string`.</span></span>

## <a name="add-a-view"></a><span data-ttu-id="873b4-250">Aggiungere una vista</span><span class="sxs-lookup"><span data-stu-id="873b4-250">Add a view</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="873b4-251">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="873b4-251">Visual Studio</span></span>](#tab/visual-studio)

* <span data-ttu-id="873b4-252">Fare clic con il pulsante destro del mouse sulla cartella *views* , quindi **aggiungere > nuova cartella** e denominare la cartella *HelloWorld*.</span><span class="sxs-lookup"><span data-stu-id="873b4-252">Right-click on the *Views* folder, and then **Add > New Folder** and name the folder *HelloWorld*.</span></span>

* <span data-ttu-id="873b4-253">Fare clic con il pulsante destro del mouse sulla cartella *views/HelloWorld* , quindi **aggiungere > nuovo elemento**.</span><span class="sxs-lookup"><span data-stu-id="873b4-253">Right-click on the *Views/HelloWorld* folder, and then **Add > New Item**.</span></span>

* <span data-ttu-id="873b4-254">Nella finestra di dialogo **Aggiungi nuovo elemento - MvcMovie**</span><span class="sxs-lookup"><span data-stu-id="873b4-254">In the **Add New Item - MvcMovie** dialog</span></span>

  * <span data-ttu-id="873b4-255">Nella casella di ricerca in alto a destra immettere *view* (vista)</span><span class="sxs-lookup"><span data-stu-id="873b4-255">In the search box in the upper-right, enter *view*</span></span>

  * <span data-ttu-id="873b4-256">Selezione **Razor visualizzazione**</span><span class="sxs-lookup"><span data-stu-id="873b4-256">Select **Razor View**</span></span>

  * <span data-ttu-id="873b4-257">Mantenere il valore della casella **Nome**, *Index.cshtml*.</span><span class="sxs-lookup"><span data-stu-id="873b4-257">Keep the **Name** box value, *Index.cshtml*.</span></span>

  * <span data-ttu-id="873b4-258">Selezionare **Aggiungi**</span><span class="sxs-lookup"><span data-stu-id="873b4-258">Select **Add**</span></span>

![Finestra di dialogo Aggiungi nuovo elemento](adding-view/_static/add_view.png)

# <a name="visual-studio-code"></a>[<span data-ttu-id="873b4-260">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="873b4-260">Visual Studio Code</span></span>](#tab/visual-studio-code)

<span data-ttu-id="873b4-261">Aggiungere una vista `Index` per `HelloWorldController`.</span><span class="sxs-lookup"><span data-stu-id="873b4-261">Add an `Index` view for the `HelloWorldController`.</span></span>

* <span data-ttu-id="873b4-262">Aggiungere una nuova cartella denominata *Views/HelloWorld*.</span><span class="sxs-lookup"><span data-stu-id="873b4-262">Add a new folder named *Views/HelloWorld*.</span></span>
* <span data-ttu-id="873b4-263">Aggiungere un nuovo file al nome di cartella *Views/HelloWorld\*\*Index.cshtml*.</span><span class="sxs-lookup"><span data-stu-id="873b4-263">Add a new file to the *Views/HelloWorld* folder name *Index.cshtml*.</span></span>

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="873b4-264">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="873b4-264">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

* <span data-ttu-id="873b4-265">Fare clic con il pulsante destro del mouse sulla cartella *Views*, quindi su **Aggiungi > Nuova cartella** e denominare la cartella *HelloWorld*.</span><span class="sxs-lookup"><span data-stu-id="873b4-265">Right click on the *Views* folder, and then **Add > New Folder** and name the folder *HelloWorld*.</span></span>
* <span data-ttu-id="873b4-266">Fare clic con il pulsante destro del mouse sulla cartella *Views/HelloWorld*, quindi su **Aggiungi > Nuovo file**.</span><span class="sxs-lookup"><span data-stu-id="873b4-266">Right click on the *Views/HelloWorld* folder, and then **Add > New File**.</span></span>
* <span data-ttu-id="873b4-267">Nel finestra di dialogo **Nuovo file**:</span><span class="sxs-lookup"><span data-stu-id="873b4-267">In the **New File** dialog:</span></span>

  * <span data-ttu-id="873b4-268">Selezionare **Web** nel riquadro a sinistra.</span><span class="sxs-lookup"><span data-stu-id="873b4-268">Select **Web** in the left pane.</span></span>
  * <span data-ttu-id="873b4-269">Selezionare **File HTML vuoto** nel riquadro centrale.</span><span class="sxs-lookup"><span data-stu-id="873b4-269">Select **Empty HTML file** in the center pane.</span></span>
  * <span data-ttu-id="873b4-270">Digitare *Index.cshtml* nella casella **Nome**.</span><span class="sxs-lookup"><span data-stu-id="873b4-270">Type *Index.cshtml* in the **Name** box.</span></span>
  * <span data-ttu-id="873b4-271">Selezionare **Nuovo**.</span><span class="sxs-lookup"><span data-stu-id="873b4-271">Select **New**.</span></span>

![Finestra di dialogo Aggiungi nuovo elemento](adding-view/_static/add_view_mac.png)

---

<span data-ttu-id="873b4-273">Sostituire il contenuto del file di visualizzazione *views/HelloWorld/index. cshtml* Razor con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="873b4-273">Replace the contents of the *Views/HelloWorld/Index.cshtml* Razor view file with the following:</span></span>

[!code-cshtml[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie22/Views/HelloWorld/Index1.cshtml?highlight=7)]

<span data-ttu-id="873b4-274">Accedere a `https://localhost:{PORT}/HelloWorld`.</span><span class="sxs-lookup"><span data-stu-id="873b4-274">Navigate to `https://localhost:{PORT}/HelloWorld`.</span></span> <span data-ttu-id="873b4-275">Il metodo `Index` in `HelloWorldController` non ha eseguito molte operazioni; ha eseguito l'istruzione `return View();` che ha specificato che il metodo deve usare un file di modello della vista per eseguire il rendering di una risposta al browser.</span><span class="sxs-lookup"><span data-stu-id="873b4-275">The `Index` method in the `HelloWorldController` didn't do much; it ran the statement `return View();`, which specified that the method should use a view template file to render a response to the browser.</span></span> <span data-ttu-id="873b4-276">Poiché non è stato specificato un nome di file di modello di visualizzazione, per impostazione predefinita MVC usa il file di visualizzazione predefinito.</span><span class="sxs-lookup"><span data-stu-id="873b4-276">Because a view template file name wasn't specified, MVC defaulted to using the default view file.</span></span> <span data-ttu-id="873b4-277">Il file di visualizzazione predefinito ha lo stesso nome del metodo (`Index`), quindi viene usato */Views/HelloWorld/Index.cshtml*.</span><span class="sxs-lookup"><span data-stu-id="873b4-277">The default view file has the same name as the method (`Index`), so in the */Views/HelloWorld/Index.cshtml* is used.</span></span> <span data-ttu-id="873b4-278">L'immagine seguente mostra la stringa "Hello from our View Template!"</span><span class="sxs-lookup"><span data-stu-id="873b4-278">The image below shows the string "Hello from our View Template!"</span></span> <span data-ttu-id="873b4-279">hardcoded nella vista.</span><span class="sxs-lookup"><span data-stu-id="873b4-279">hard-coded in the view.</span></span>

![Finestra del browser](~/tutorials/first-mvc-app/adding-view/_static/hell_template.png)

## <a name="change-views-and-layout-pages"></a><span data-ttu-id="873b4-281">Modificare le viste e le pagine di layout</span><span class="sxs-lookup"><span data-stu-id="873b4-281">Change views and layout pages</span></span>

<span data-ttu-id="873b4-282">Selezionare i collegamenti di menu: **MvcMovie**, **Home** e **Privacy**.</span><span class="sxs-lookup"><span data-stu-id="873b4-282">Select the menu links (**MvcMovie**, **Home**, and **Privacy**).</span></span> <span data-ttu-id="873b4-283">Ogni pagina mostra lo stesso layout di menu.</span><span class="sxs-lookup"><span data-stu-id="873b4-283">Each page shows the same menu layout.</span></span> <span data-ttu-id="873b4-284">Il layout di menu viene implementato nel file *Views/Shared/_Layout.cshtml*.</span><span class="sxs-lookup"><span data-stu-id="873b4-284">The menu layout is implemented in the *Views/Shared/_Layout.cshtml* file.</span></span> <span data-ttu-id="873b4-285">Aprire il file *Views/Shared/_Layout.cshtml*.</span><span class="sxs-lookup"><span data-stu-id="873b4-285">Open the *Views/Shared/_Layout.cshtml* file.</span></span>

<span data-ttu-id="873b4-286">I modelli di [layout](xref:mvc/views/layout) consentono di specificare il layout del contenitore HTML del sito in un'unica posizione e quindi di applicarlo in più pagine del sito.</span><span class="sxs-lookup"><span data-stu-id="873b4-286">[Layout](xref:mvc/views/layout) templates allow you to specify the HTML container layout of your site in one place and then apply it across multiple pages in your site.</span></span> <span data-ttu-id="873b4-287">Trovare la riga `@RenderBody()`.</span><span class="sxs-lookup"><span data-stu-id="873b4-287">Find the `@RenderBody()` line.</span></span> <span data-ttu-id="873b4-288">`RenderBody` è un segnaposto dove tutte le pagine specifiche della vista vengono presentate, *incapsulate* nella pagina di layout.</span><span class="sxs-lookup"><span data-stu-id="873b4-288">`RenderBody` is a placeholder where all the view-specific pages you create show up, *wrapped* in the layout page.</span></span> <span data-ttu-id="873b4-289">Ad esempio, se si seleziona il collegamento **Privacy**, il rendering della vista **Views/Home/Privacy.cshtml** viene eseguito all'interno del metodo `RenderBody`.</span><span class="sxs-lookup"><span data-stu-id="873b4-289">For example, if you select the **Privacy** link, the **Views/Home/Privacy.cshtml** view is rendered inside the `RenderBody` method.</span></span>

## <a name="change-the-title-footer-and-menu-link-in-the-layout-file"></a><span data-ttu-id="873b4-290">Modificare il titolo, il piè di pagina e il collegamento di menu nel file di layout</span><span class="sxs-lookup"><span data-stu-id="873b4-290">Change the title, footer, and menu link in the layout file</span></span>

* <span data-ttu-id="873b4-291">Negli elementi del titolo e del piè di pagina modificare `MvcMovie` in `Movie App`.</span><span class="sxs-lookup"><span data-stu-id="873b4-291">In the title and footer elements, change `MvcMovie` to `Movie App`.</span></span>
* <span data-ttu-id="873b4-292">Modificare l'elemento ancora `<a class="navbar-brand" asp-area="" asp-controller="Home" asp-action="Index">MvcMovie</a>` in `<a class="navbar-brand" asp-controller="Movies" asp-action="Index">Movie App</a>`.</span><span class="sxs-lookup"><span data-stu-id="873b4-292">Change the anchor element `<a class="navbar-brand" asp-area="" asp-controller="Home" asp-action="Index">MvcMovie</a>` to `<a class="navbar-brand" asp-controller="Movies" asp-action="Index">Movie App</a>`.</span></span>

<span data-ttu-id="873b4-293">Il markup seguente visualizza le modifiche evidenziate:</span><span class="sxs-lookup"><span data-stu-id="873b4-293">The following markup shows the highlighted changes:</span></span>

[!code-cshtml[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie22/Views/Shared/_Layout.cshtml?highlight=6,24,51)]

<span data-ttu-id="873b4-294">Nel markup precedente, l'attributo `asp-area` [Helper Tag di ancoraggio](xref:mvc/views/tag-helpers/builtin-th/anchor-tag-helper) è stato omesso perché questa app non usa le [Aree](xref:mvc/controllers/areas).</span><span class="sxs-lookup"><span data-stu-id="873b4-294">In the preceding markup, the `asp-area` [anchor Tag Helper attribute](xref:mvc/views/tag-helpers/builtin-th/anchor-tag-helper) was omitted because this app is not using [Areas](xref:mvc/controllers/areas).</span></span>

<!-- Routing has changed in 2.2, it's going to the last route.
>[!WARNING]
> We haven't implemented the `Movies` controller yet, so if you click the `Movie App` link, you get a 404 (Not found) error.
-->

<span data-ttu-id="873b4-295">**Nota**: il `Movies` controller non è stato implementato.</span><span class="sxs-lookup"><span data-stu-id="873b4-295">**Note**: The `Movies` controller has not been implemented.</span></span> <span data-ttu-id="873b4-296">A questo punto, il collegamento `Movie App` non è funzionale.</span><span class="sxs-lookup"><span data-stu-id="873b4-296">At this point, the `Movie App` link is not functional.</span></span>

<span data-ttu-id="873b4-297">Salvare le modifiche e selezionare il collegamento **Privacy**.</span><span class="sxs-lookup"><span data-stu-id="873b4-297">Save your changes and select the **Privacy** link.</span></span> <span data-ttu-id="873b4-298">Si noti come il titolo sulla scheda del browser visualizzi **Privacy Policy - Movie App** anziché **Privacy Policy - Mvc Movie**:</span><span class="sxs-lookup"><span data-stu-id="873b4-298">Notice how the title on the browser tab displays **Privacy Policy - Movie App** instead of **Privacy Policy - Mvc Movie**:</span></span>

![Scheda Privacy](~/tutorials/first-mvc-app/adding-view/_static/privacy50.png)

<span data-ttu-id="873b4-300">Toccare il collegamento **Home** e notare che anche il titolo e il testo di ancoraggio visualizzano **Movie App**.</span><span class="sxs-lookup"><span data-stu-id="873b4-300">Select the **Home** link and notice that the title and anchor text also display **Movie App**.</span></span> <span data-ttu-id="873b4-301">La modifica è stata apportata una volta nel modello di layout e tutte le pagine nel sito riflettono il nuovo testo del collegamento e il nuovo titolo.</span><span class="sxs-lookup"><span data-stu-id="873b4-301">We were able to make the change once in the layout template and have all pages on the site reflect the new link text and new title.</span></span>

<span data-ttu-id="873b4-302">Esaminare il file *Views/_ViewStart.cshtml*:</span><span class="sxs-lookup"><span data-stu-id="873b4-302">Examine the *Views/_ViewStart.cshtml* file:</span></span>

```cshtml
@{
    Layout = "_Layout";
}
```

<span data-ttu-id="873b4-303">Il file *Views/_ViewStart.cshtml* attiva il file *Views/Shared/_Layout.cshtml* per ogni vista.</span><span class="sxs-lookup"><span data-stu-id="873b4-303">The *Views/_ViewStart.cshtml* file brings in the *Views/Shared/_Layout.cshtml* file to each view.</span></span> <span data-ttu-id="873b4-304">È possibile usare la proprietà `Layout` per impostare una vista di layout differente oppure impostarla su `null` e quindi non verrà usato alcun file di layout.</span><span class="sxs-lookup"><span data-stu-id="873b4-304">The `Layout` property can be used to set a different layout view, or set it to `null` so no layout file will be used.</span></span>

<span data-ttu-id="873b4-305">Modificare il titolo e l'elemento `<h2>` del file *Views/HelloWorld/Index.cshtml* di visualizzazione:</span><span class="sxs-lookup"><span data-stu-id="873b4-305">Change the title and `<h2>` element of the *Views/HelloWorld/Index.cshtml* view file:</span></span>

[!code-cshtml[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie/Views/HelloWorld/Index2.cshtml?highlight=2,5)]

<span data-ttu-id="873b4-306">Il titolo e l'elemento `<h2>`sono leggermente diversi in modo da poter esaminare la parte specifica di codice che modifica la visualizzazione.</span><span class="sxs-lookup"><span data-stu-id="873b4-306">The title and `<h2>` element are slightly different so you can see which bit of code changes the display.</span></span>

<span data-ttu-id="873b4-307">`ViewData["Title"] = "Movie List";` nel codice precedente imposta la proprietà `Title` del dizionario `ViewData` su "Movie List".</span><span class="sxs-lookup"><span data-stu-id="873b4-307">`ViewData["Title"] = "Movie List";` in the code above sets the `Title` property of the `ViewData` dictionary to "Movie List".</span></span> <span data-ttu-id="873b4-308">La proprietà `Title` viene usata nell'elemento HTML `<title>` nella pagina di layout:</span><span class="sxs-lookup"><span data-stu-id="873b4-308">The `Title` property is used in the `<title>` HTML element in the layout page:</span></span>

```cshtml
<title>@ViewData["Title"] - Movie App</title>
```

<span data-ttu-id="873b4-309">Salvare la modifica e passare a `https://localhost:{PORT}/HelloWorld`.</span><span class="sxs-lookup"><span data-stu-id="873b4-309">Save the change and navigate to `https://localhost:{PORT}/HelloWorld`.</span></span> <span data-ttu-id="873b4-310">Si noti che il titolo del browser, l'intestazione primaria e le intestazioni secondarie sono cambiati.</span><span class="sxs-lookup"><span data-stu-id="873b4-310">Notice that the browser title, the primary heading, and the secondary headings have changed.</span></span> <span data-ttu-id="873b4-311">Se le modifiche non sono visibili nel browser, è possibile che si stia visualizzando il contenuto memorizzato nella cache.</span><span class="sxs-lookup"><span data-stu-id="873b4-311">(If you don't see changes in the browser, you might be viewing cached content.</span></span> <span data-ttu-id="873b4-312">Premere CTRL + F5 nel browser per forzare il caricamento della risposta dal server. Il titolo del browser viene creato con l' `ViewData["Title"]` impostazione nel modello di vista *index. cshtml* e l'aggiunta di "-Movie app" nel file di layout.</span><span class="sxs-lookup"><span data-stu-id="873b4-312">Press Ctrl+F5 in your browser to force the response from the server to be loaded.) The browser title is created with `ViewData["Title"]` we set in the *Index.cshtml* view template and the additional "- Movie App" added in the layout file.</span></span>

<span data-ttu-id="873b4-313">Si noti anche come il contenuto del modello di vista *Index.cshtml* sia stato unito con il modello di vista *Views/Shared/_Layout.cshtml* e sia stata inviata una singola risposta HTML al browser.</span><span class="sxs-lookup"><span data-stu-id="873b4-313">Also notice how the content in the *Index.cshtml* view template was merged with the *Views/Shared/_Layout.cshtml* view template and a single HTML response was sent to the browser.</span></span> <span data-ttu-id="873b4-314">I modelli di layout rendono molto semplice apportare modifiche che si applicano a tutte le pagine dell'applicazione.</span><span class="sxs-lookup"><span data-stu-id="873b4-314">Layout templates make it really easy to make changes that apply across all of the pages in your application.</span></span> <span data-ttu-id="873b4-315">Per altre informazioni, vedere [Layout](xref:mvc/views/layout).</span><span class="sxs-lookup"><span data-stu-id="873b4-315">To learn more see [Layout](xref:mvc/views/layout).</span></span>

![Vista dell'elenco di film](~/tutorials/first-mvc-app/adding-view/_static/hell3.png)

<span data-ttu-id="873b4-317">Il breve testo di "dati", in questo caso il messaggio "Hello from our View Template!",</span><span class="sxs-lookup"><span data-stu-id="873b4-317">Our little bit of "data" (in this case the "Hello from our View Template!"</span></span> <span data-ttu-id="873b4-318">è tuttavia hardcoded.</span><span class="sxs-lookup"><span data-stu-id="873b4-318">message) is hard-coded, though.</span></span> <span data-ttu-id="873b4-319">L'applicazione MVC ha un elemento "V" (vista) ed è stato ottenuto un elemento "C" (controller), ma non ancora un elemento "M" (modello).</span><span class="sxs-lookup"><span data-stu-id="873b4-319">The MVC application has a "V" (view) and you've got a "C" (controller), but no "M" (model) yet.</span></span>

## <a name="passing-data-from-the-controller-to-the-view"></a><span data-ttu-id="873b4-320">Passaggio di dati dal controller alla vista</span><span class="sxs-lookup"><span data-stu-id="873b4-320">Passing Data from the Controller to the View</span></span>

<span data-ttu-id="873b4-321">Le azioni del controller vengono richiamate in risposta a una richiesta URL in ingresso.</span><span class="sxs-lookup"><span data-stu-id="873b4-321">Controller actions are invoked in response to an incoming URL request.</span></span> <span data-ttu-id="873b4-322">Una classe controller è il punto in cui viene scritto il codice che gestisce le richieste in ingresso del browser.</span><span class="sxs-lookup"><span data-stu-id="873b4-322">A controller class is where the code is written that handles the incoming browser requests.</span></span> <span data-ttu-id="873b4-323">Il controller recupera i dati da un'origine dati e determina il tipo di risposta da inviare al browser.</span><span class="sxs-lookup"><span data-stu-id="873b4-323">The controller retrieves data from a data source and decides what type of response to send back to the browser.</span></span> <span data-ttu-id="873b4-324">I modelli di vista possono essere usati da un controller per generare e formattare una risposta HTML al browser.</span><span class="sxs-lookup"><span data-stu-id="873b4-324">View templates can be used from a controller to generate and format an HTML response to the browser.</span></span>

<span data-ttu-id="873b4-325">Il compito dei controller è fornire i dati necessari affinché un modello di vista possa eseguire il rendering di una risposta.</span><span class="sxs-lookup"><span data-stu-id="873b4-325">Controllers are responsible for providing the data required in order for a view template to render a response.</span></span> <span data-ttu-id="873b4-326">Procedura consigliata: i modelli di vista **non** devono eseguire la logica di business o interagire direttamente con un database.</span><span class="sxs-lookup"><span data-stu-id="873b4-326">A best practice: View templates should **not** perform business logic or interact with a database directly.</span></span> <span data-ttu-id="873b4-327">Un modello di vista deve invece usare solo i dati forniti dal controller.</span><span class="sxs-lookup"><span data-stu-id="873b4-327">Rather, a view template should work only with the data that's provided to it by the controller.</span></span> <span data-ttu-id="873b4-328">Questa "separazione delle problematiche" consente di mantenere il codice pulito e semplifica la gestione e i test.</span><span class="sxs-lookup"><span data-stu-id="873b4-328">Maintaining this "separation of concerns" helps keep the code clean, testable, and maintainable.</span></span>

<span data-ttu-id="873b4-329">Attualmente il metodo `Welcome` nella classe `HelloWorldController` accetta un parametro `name` e `ID` e quindi genera i valori direttamente al browser.</span><span class="sxs-lookup"><span data-stu-id="873b4-329">Currently, the `Welcome` method in the `HelloWorldController` class takes a `name` and a `ID` parameter and then outputs the values directly to the browser.</span></span> <span data-ttu-id="873b4-330">Invece di ottenere il rendering di questa risposta come stringa, cambiare il controller in modo che usi un modello di vista.</span><span class="sxs-lookup"><span data-stu-id="873b4-330">Rather than have the controller render this response as a string, change the controller to use a view template instead.</span></span> <span data-ttu-id="873b4-331">Il modello di vista genera una risposta dinamica, il che significa che è necessario passare i bit di dati appropriati dal controller alla vista per generare la risposta.</span><span class="sxs-lookup"><span data-stu-id="873b4-331">The view template generates a dynamic response, which means that appropriate bits of data must be passed from the controller to the view in order to generate the response.</span></span> <span data-ttu-id="873b4-332">A tale scopo, fare in modo che il controller inserisca i dati dinamici (parametri) di cui il modello di vista necessita in un dizionario `ViewData` a cui il modello di vista può accedere.</span><span class="sxs-lookup"><span data-stu-id="873b4-332">Do this by having the controller put the dynamic data (parameters) that the view template needs in a `ViewData` dictionary that the view template can then access.</span></span>

<span data-ttu-id="873b4-333">In *HelloWorldController.cs*, modificare il metodo `Welcome` in modo da aggiungere un valore `Message` e `NumTimes` al dizionario `ViewData`.</span><span class="sxs-lookup"><span data-stu-id="873b4-333">In *HelloWorldController.cs*, change the `Welcome` method to add a `Message` and `NumTimes` value to the `ViewData` dictionary.</span></span> <span data-ttu-id="873b4-334">Il dizionario `ViewData` è un oggetto dinamico, ovvero è possibile usare qualunque tipo; l'oggetto `ViewData` non dispone di proprietà definite fino a quando non si inserisce un elemento al suo interno.</span><span class="sxs-lookup"><span data-stu-id="873b4-334">The `ViewData` dictionary is a dynamic object, which means any type can be used; the `ViewData` object has no defined properties until you put something inside it.</span></span> <span data-ttu-id="873b4-335">Il sistema di [associazione di modelli MVC](xref:mvc/models/model-binding) esegue il mapping dei parametri denominati (`name` e `numTimes`) dalla stringa di query nella barra degli indirizzi ai parametri nel metodo.</span><span class="sxs-lookup"><span data-stu-id="873b4-335">The [MVC model binding system](xref:mvc/models/model-binding) automatically maps the named parameters (`name` and `numTimes`) from the query string in the address bar to parameters in your method.</span></span> <span data-ttu-id="873b4-336">Il file *HelloWorldController.cs* completo avrà un aspetto simile al seguente:</span><span class="sxs-lookup"><span data-stu-id="873b4-336">The complete *HelloWorldController.cs* file looks like this:</span></span>

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie/Controllers/HelloWorldController.cs?name=snippet_5)]

<span data-ttu-id="873b4-337">L'oggetto di dizionario `ViewData` contiene i dati che verranno passati alla vista.</span><span class="sxs-lookup"><span data-stu-id="873b4-337">The `ViewData` dictionary object contains data that will be passed to the view.</span></span>

<span data-ttu-id="873b4-338">Creare un modello di vista Welcome denominato *Views/HelloWorld/Welcome.cshtml*.</span><span class="sxs-lookup"><span data-stu-id="873b4-338">Create a Welcome view template named *Views/HelloWorld/Welcome.cshtml*.</span></span>

<span data-ttu-id="873b4-339">Si creerà un ciclo nel modello di vista *Welcome.cshtml* che visualizza la stringa "Hello" `NumTimes`.</span><span class="sxs-lookup"><span data-stu-id="873b4-339">You'll create a loop in the *Welcome.cshtml* view template that displays "Hello" `NumTimes`.</span></span> <span data-ttu-id="873b4-340">Sostituire il contenuto di *Views/HelloWorld/Welcome.cshtml* con quanto segue:</span><span class="sxs-lookup"><span data-stu-id="873b4-340">Replace the contents of *Views/HelloWorld/Welcome.cshtml* with the following:</span></span>

[!code-cshtml[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie/Views/HelloWorld/Welcome.cshtml)]

<span data-ttu-id="873b4-341">Salvare le modifiche e passare all'URL seguente:</span><span class="sxs-lookup"><span data-stu-id="873b4-341">Save your changes and browse to the following URL:</span></span>

`https://localhost:{PORT}/HelloWorld/Welcome?name=Rick&numtimes=4`

<span data-ttu-id="873b4-342">I dati vengono prelevati dall'URL e passati al controller usando lo [strumento di associazione di modelli MVC](xref:mvc/models/model-binding).</span><span class="sxs-lookup"><span data-stu-id="873b4-342">Data is taken from the URL and passed to the controller using the [MVC model binder](xref:mvc/models/model-binding) .</span></span> <span data-ttu-id="873b4-343">Il controller crea un pacchetto di dati in un dizionario `ViewData` e passa tale oggetto alla vista.</span><span class="sxs-lookup"><span data-stu-id="873b4-343">The controller packages the data into a `ViewData` dictionary and passes that object to the view.</span></span> <span data-ttu-id="873b4-344">La vista esegue quindi il rendering dei dati in formato HTML al browser.</span><span class="sxs-lookup"><span data-stu-id="873b4-344">The view then renders the data as HTML to the browser.</span></span>

![Vista Privacy con un'etichetta di benvenuto e la frase Hello Rick riportata quattro volte](~/tutorials/first-mvc-app/adding-view/_static/rick2.png)

<span data-ttu-id="873b4-346">Nell'esempio precedente è stato usato il dizionario `ViewData` per passare i dati dal controller a una vista.</span><span class="sxs-lookup"><span data-stu-id="873b4-346">In the sample above, the `ViewData` dictionary was used to pass data from the controller to a view.</span></span> <span data-ttu-id="873b4-347">Più avanti nell'esercitazione si userà un modello di vista per passare i dati da un controller a una vista.</span><span class="sxs-lookup"><span data-stu-id="873b4-347">Later in the tutorial, a view model is used to pass data from a controller to a view.</span></span> <span data-ttu-id="873b4-348">In genere l'approccio basato sul modello di vista per passare i dati è preferito rispetto all'approccio basato sul dizionario `ViewData`.</span><span class="sxs-lookup"><span data-stu-id="873b4-348">The view model approach to passing data is generally much preferred over the `ViewData` dictionary approach.</span></span> <span data-ttu-id="873b4-349">Per altre informazioni, vedere [quando usare ViewBag, ViewData o TempData](https://www.rachelappel.com/when-to-use-viewbag-viewdata-or-tempdata-in-asp-net-mvc-3-applications/) .</span><span class="sxs-lookup"><span data-stu-id="873b4-349">See [When to use ViewBag, ViewData, or TempData](https://www.rachelappel.com/when-to-use-viewbag-viewdata-or-tempdata-in-asp-net-mvc-3-applications/) for more information.</span></span>

<span data-ttu-id="873b4-350">Nella prossima esercitazione, viene creato un database di film.</span><span class="sxs-lookup"><span data-stu-id="873b4-350">In the next tutorial, a database of movies is created.</span></span>

> [!div class="step-by-step"]
> <span data-ttu-id="873b4-351">[Precedente](adding-controller.md) 
>  [Avanti](adding-model.md)</span><span class="sxs-lookup"><span data-stu-id="873b4-351">[Previous](adding-controller.md)
[Next](adding-model.md)</span></span>

::: moniker-end
