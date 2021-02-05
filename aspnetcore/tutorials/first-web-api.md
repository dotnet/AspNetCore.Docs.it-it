---
title: "Esercitazione: Creare un'API Web con ASP.NET Core"
author: rick-anderson
description: Informazioni su come creare un'API Web con ASP.NET Core.
ms.author: riande
ms.custom: mvc, devx-track-js
ms.date: 02/04/2021
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
- Models
uid: tutorials/first-web-api
ms.openlocfilehash: bef6efab3e5e84908a36c2c70f019cdd3b3e059e
ms.sourcegitcommit: 20a41c8e40a2e69e99291e2fe18caa04c02e7109
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/05/2021
ms.locfileid: "99578406"
---
# <a name="tutorial-create-a-web-api-with-aspnet-core"></a><span data-ttu-id="53ab8-103">Esercitazione: Creare un'API Web con ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="53ab8-103">Tutorial: Create a web API with ASP.NET Core</span></span>

<span data-ttu-id="53ab8-104">Di [Rick Anderson](https://twitter.com/RickAndMSFT), [Kirk Larkin](https://twitter.com/serpent5)e [Mike Wasson](https://github.com/mikewasson)</span><span class="sxs-lookup"><span data-stu-id="53ab8-104">By [Rick Anderson](https://twitter.com/RickAndMSFT), [Kirk Larkin](https://twitter.com/serpent5), and [Mike Wasson](https://github.com/mikewasson)</span></span>

<span data-ttu-id="53ab8-105">Questa esercitazione illustra le nozioni di base della creazione di un'API Web con ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="53ab8-105">This tutorial teaches the basics of building a web API with ASP.NET Core.</span></span>

::: moniker range=">= aspnetcore-5.0"

<span data-ttu-id="53ab8-106">In questa esercitazione verranno illustrate le procedure per:</span><span class="sxs-lookup"><span data-stu-id="53ab8-106">In this tutorial, you learn how to:</span></span>

> [!div class="checklist"]
> * <span data-ttu-id="53ab8-107">Creare un progetto di API Web.</span><span class="sxs-lookup"><span data-stu-id="53ab8-107">Create a web API project.</span></span>
> * <span data-ttu-id="53ab8-108">Aggiungere una classe modello e un contesto di database.</span><span class="sxs-lookup"><span data-stu-id="53ab8-108">Add a model class and a database context.</span></span>
> * <span data-ttu-id="53ab8-109">Eseguire lo scaffolding di un controller con i metodi CRUD.</span><span class="sxs-lookup"><span data-stu-id="53ab8-109">Scaffold a controller with CRUD methods.</span></span>
> * <span data-ttu-id="53ab8-110">Configurare il routing, i percorsi URL e i valori restituiti.</span><span class="sxs-lookup"><span data-stu-id="53ab8-110">Configure routing, URL paths, and return values.</span></span>
> * <span data-ttu-id="53ab8-111">Chiamare l'API Web con Postman.</span><span class="sxs-lookup"><span data-stu-id="53ab8-111">Call the web API with Postman.</span></span>

<span data-ttu-id="53ab8-112">Al termine si avrà un'API Web che può gestire gli elementi di tipo "attività" archiviati in un database.</span><span class="sxs-lookup"><span data-stu-id="53ab8-112">At the end, you have a web API that can manage "to-do" items stored in a database.</span></span>

## <a name="overview"></a><span data-ttu-id="53ab8-113">Panoramica</span><span class="sxs-lookup"><span data-stu-id="53ab8-113">Overview</span></span>

<span data-ttu-id="53ab8-114">Questa esercitazione consente di creare l'API seguente:</span><span class="sxs-lookup"><span data-stu-id="53ab8-114">This tutorial creates the following API:</span></span>

|<span data-ttu-id="53ab8-115">API</span><span class="sxs-lookup"><span data-stu-id="53ab8-115">API</span></span> | <span data-ttu-id="53ab8-116">Descrizione</span><span class="sxs-lookup"><span data-stu-id="53ab8-116">Description</span></span> | <span data-ttu-id="53ab8-117">Testo della richiesta</span><span class="sxs-lookup"><span data-stu-id="53ab8-117">Request body</span></span> | <span data-ttu-id="53ab8-118">Corpo della risposta</span><span class="sxs-lookup"><span data-stu-id="53ab8-118">Response body</span></span> |
|--- | ---- | ---- | ---- |
|`GET /api/TodoItems` | <span data-ttu-id="53ab8-119">Ottiene tutti gli elementi attività</span><span class="sxs-lookup"><span data-stu-id="53ab8-119">Get all to-do items</span></span> | <span data-ttu-id="53ab8-120">nessuno</span><span class="sxs-lookup"><span data-stu-id="53ab8-120">None</span></span> | <span data-ttu-id="53ab8-121">Matrice di elementi attività</span><span class="sxs-lookup"><span data-stu-id="53ab8-121">Array of to-do items</span></span>|
|`GET /api/TodoItems/{id}` | <span data-ttu-id="53ab8-122">Ottiene un elemento in base all'ID</span><span class="sxs-lookup"><span data-stu-id="53ab8-122">Get an item by ID</span></span> | <span data-ttu-id="53ab8-123">nessuno</span><span class="sxs-lookup"><span data-stu-id="53ab8-123">None</span></span> | <span data-ttu-id="53ab8-124">Elemento attività</span><span class="sxs-lookup"><span data-stu-id="53ab8-124">To-do item</span></span>|
|`POST /api/TodoItems` | <span data-ttu-id="53ab8-125">Aggiunge un nuovo elemento</span><span class="sxs-lookup"><span data-stu-id="53ab8-125">Add a new item</span></span> | <span data-ttu-id="53ab8-126">Elemento attività</span><span class="sxs-lookup"><span data-stu-id="53ab8-126">To-do item</span></span> | <span data-ttu-id="53ab8-127">Elemento attività</span><span class="sxs-lookup"><span data-stu-id="53ab8-127">To-do item</span></span> |
|`PUT /api/TodoItems/{id}` | <span data-ttu-id="53ab8-128">Aggiorna un elemento esistente &nbsp;</span><span class="sxs-lookup"><span data-stu-id="53ab8-128">Update an existing item &nbsp;</span></span> | <span data-ttu-id="53ab8-129">Elemento attività</span><span class="sxs-lookup"><span data-stu-id="53ab8-129">To-do item</span></span> | <span data-ttu-id="53ab8-130">nessuno</span><span class="sxs-lookup"><span data-stu-id="53ab8-130">None</span></span> |
|<span data-ttu-id="53ab8-131">`DELETE /api/TodoItems/{id}` &nbsp; &nbsp;</span><span class="sxs-lookup"><span data-stu-id="53ab8-131">`DELETE /api/TodoItems/{id}` &nbsp; &nbsp;</span></span> | <span data-ttu-id="53ab8-132">Elimina un elemento &nbsp;&nbsp;</span><span class="sxs-lookup"><span data-stu-id="53ab8-132">Delete an item &nbsp; &nbsp;</span></span> | <span data-ttu-id="53ab8-133">nessuno</span><span class="sxs-lookup"><span data-stu-id="53ab8-133">None</span></span> | <span data-ttu-id="53ab8-134">nessuno</span><span class="sxs-lookup"><span data-stu-id="53ab8-134">None</span></span>|

<span data-ttu-id="53ab8-135">Il diagramma seguente visualizza la struttura dell'app.</span><span class="sxs-lookup"><span data-stu-id="53ab8-135">The following diagram shows the design of the app.</span></span>

![Il client è rappresentato da una casella a sinistra.](first-web-api/_static/architecture.png)

## <a name="prerequisites"></a><span data-ttu-id="53ab8-141">Prerequisiti</span><span class="sxs-lookup"><span data-stu-id="53ab8-141">Prerequisites</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="53ab8-142">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="53ab8-142">Visual Studio</span></span>](#tab/visual-studio)

[!INCLUDE[](~/includes/net-core-prereqs-vs-5.0.md)]

# <a name="visual-studio-code"></a>[<span data-ttu-id="53ab8-143">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="53ab8-143">Visual Studio Code</span></span>](#tab/visual-studio-code)

[!INCLUDE[](~/includes/net-core-prereqs-vsc-5.0.md)]

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="53ab8-144">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="53ab8-144">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

[!INCLUDE[](~/includes/net-core-prereqs-mac-5.0.md)]

---

## <a name="create-a-web-project"></a><span data-ttu-id="53ab8-145">Creare un progetto Web</span><span class="sxs-lookup"><span data-stu-id="53ab8-145">Create a web project</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="53ab8-146">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="53ab8-146">Visual Studio</span></span>](#tab/visual-studio)

* <span data-ttu-id="53ab8-147">Scegliere **nuovo** progetto dal menu **file** > .</span><span class="sxs-lookup"><span data-stu-id="53ab8-147">From the **File** menu, select **New** > **Project**.</span></span>
* <span data-ttu-id="53ab8-148">Selezionare il modello **Applicazione Web ASP.NET Core** e fare clic su **Avanti**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-148">Select the **ASP.NET Core Web Application** template and click **Next**.</span></span>
* <span data-ttu-id="53ab8-149">Assegnare al progetto il nome *TodoApi* e fare clic su **Crea**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-149">Name the project *TodoApi* and click **Create**.</span></span>
* <span data-ttu-id="53ab8-150">Nella finestra di dialogo **Crea una nuova applicazione Web ASP.NET Core** verificare che siano selezionati **.net Core** e **ASP.NET Core 5,0** .</span><span class="sxs-lookup"><span data-stu-id="53ab8-150">In the **Create a new ASP.NET Core Web Application** dialog, confirm that **.NET Core** and **ASP.NET Core 5.0** are selected.</span></span> <span data-ttu-id="53ab8-151">Selezionare il modello **API** e fare clic su **Crea**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-151">Select the **API** template and click **Create**.</span></span>

![Finestra di dialogo Nuovo progetto di Visual Studio](first-web-api/_static/5/vs.png)

# <a name="visual-studio-code"></a>[<span data-ttu-id="53ab8-153">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="53ab8-153">Visual Studio Code</span></span>](#tab/visual-studio-code)

* <span data-ttu-id="53ab8-154">Aprire il [terminale integrato](https://code.visualstudio.com/docs/editor/integrated-terminal).</span><span class="sxs-lookup"><span data-stu-id="53ab8-154">Open the [integrated terminal](https://code.visualstudio.com/docs/editor/integrated-terminal).</span></span>
* <span data-ttu-id="53ab8-155">Cambiare directory (`cd`) e passare alla cartella che conterrà la cartella del progetto.</span><span class="sxs-lookup"><span data-stu-id="53ab8-155">Change directories (`cd`) to the folder that will contain the project folder.</span></span>
* <span data-ttu-id="53ab8-156">Eseguire i comandi seguenti:</span><span class="sxs-lookup"><span data-stu-id="53ab8-156">Run the following commands:</span></span>

   ```dotnetcli
   dotnet new webapi -o TodoApi
   cd TodoApi
   dotnet add package Microsoft.EntityFrameworkCore.InMemory
   code -r ../TodoApi
   ```

* <span data-ttu-id="53ab8-157">Quando una finestra di dialogo chiede se si vuole aggiungere gli asset necessari al progetto, selezionare **Sì**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-157">When a dialog box asks if you want to add required assets to the project, select **Yes**.</span></span>

  <span data-ttu-id="53ab8-158">I comandi precedenti:</span><span class="sxs-lookup"><span data-stu-id="53ab8-158">The preceding commands:</span></span>

  * <span data-ttu-id="53ab8-159">Crea un nuovo progetto API Web e lo apre in Visual Studio Code.</span><span class="sxs-lookup"><span data-stu-id="53ab8-159">Creates a new web API project and opens it in Visual Studio Code.</span></span>
  * <span data-ttu-id="53ab8-160">Aggiunge i pacchetti NuGet necessari nella sezione successiva.</span><span class="sxs-lookup"><span data-stu-id="53ab8-160">Adds the NuGet packages which are required in the next section.</span></span>

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="53ab8-161">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="53ab8-161">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

* <span data-ttu-id="53ab8-162">Selezionare **File** > **Nuova soluzione**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-162">Select **File** > **New Solution**.</span></span>

  ![Nuova soluzione macOS](first-web-api-mac/_static/sln.png)

* <span data-ttu-id="53ab8-164">In Visual Studio per Mac precedente alla versione 8,6 Selezionare API **app .NET Core**  >    >    >  **Avanti**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-164">In Visual Studio for Mac earlier than version 8.6, select **.NET Core** > **App** > **API** > **Next**.</span></span> <span data-ttu-id="53ab8-165">Nella versione 8,6 o successive selezionare API **app Web e console**  >    >    >  **Avanti**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-165">In version 8.6 or later, select **Web and Console** > **App** > **API** > **Next**.</span></span>

  ![Selezione modello API macOS](first-web-api-mac/_static/api_template.png)

* <span data-ttu-id="53ab8-167">Nella finestra di dialogo **Configura nuova ASP.NET Core API Web** selezionare il **Framework di destinazione**.NET Core 5. x più recente.</span><span class="sxs-lookup"><span data-stu-id="53ab8-167">In the **Configure the new ASP.NET Core Web API** dialog, select the latest .NET Core 5.x **Target Framework**.</span></span> <span data-ttu-id="53ab8-168">Selezionare **Avanti**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-168">Select **Next**.</span></span>

* <span data-ttu-id="53ab8-169">Immettere *TodoApi* in **Nome progetto**, quindi selezionare **Crea**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-169">Enter *TodoApi* for the **Project Name** and then select **Create**.</span></span>

  ![Finestra di dialogo di configurazione](first-web-api-mac/_static/2.png)

[!INCLUDE[](~/includes/mac-terminal-access.md)]

<span data-ttu-id="53ab8-171">Aprire un terminale di comando nella cartella del progetto ed eseguire il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="53ab8-171">Open a command terminal in the project folder and run the following command:</span></span>

   ```dotnetcli
   dotnet add package Microsoft.EntityFrameworkCore.InMemory
   ```

---

### <a name="test-the-project"></a><span data-ttu-id="53ab8-172">Testare il progetto</span><span class="sxs-lookup"><span data-stu-id="53ab8-172">Test the project</span></span>

<span data-ttu-id="53ab8-173">Il modello di progetto crea un' `WeatherForecast` API con supporto per [spavalderia](xref:tutorials/web-api-help-pages-using-swagger).</span><span class="sxs-lookup"><span data-stu-id="53ab8-173">The project template creates a `WeatherForecast` API with support for [Swagger](xref:tutorials/web-api-help-pages-using-swagger).</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="53ab8-174">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="53ab8-174">Visual Studio</span></span>](#tab/visual-studio)

<span data-ttu-id="53ab8-175">Premere CTRL+F5 per l'esecuzione senza il debugger.</span><span class="sxs-lookup"><span data-stu-id="53ab8-175">Press Ctrl+F5 to run without the debugger.</span></span>

[!INCLUDE[](~/includes/trustCertVS.md)]

  <span data-ttu-id="53ab8-176">Visual Studio viene avviato:</span><span class="sxs-lookup"><span data-stu-id="53ab8-176">Visual Studio launches:</span></span>

* <span data-ttu-id="53ab8-177">Server Web IIS Express.</span><span class="sxs-lookup"><span data-stu-id="53ab8-177">The IIS Express web server.</span></span>
* <span data-ttu-id="53ab8-178">Il browser predefinito e passa a `https://localhost:<port>/swagger/index.html` , dove `<port>` è un numero di porta scelto in modo casuale.</span><span class="sxs-lookup"><span data-stu-id="53ab8-178">The default browser and navigates to `https://localhost:<port>/swagger/index.html`, where `<port>` is a randomly chosen port number.</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="53ab8-179">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="53ab8-179">Visual Studio Code</span></span>](#tab/visual-studio-code)

[!INCLUDE[](~/includes/trustCertVSC.md)]

<span data-ttu-id="53ab8-180">Premere CTRL+F5 per eseguire l'app.</span><span class="sxs-lookup"><span data-stu-id="53ab8-180">Press Ctrl+F5 to run the app.</span></span> <span data-ttu-id="53ab8-181">In un browser passare all'URL seguente: [https://localhost:5001/swagger](https://localhost:5001/swagger)</span><span class="sxs-lookup"><span data-stu-id="53ab8-181">In a browser, go to following URL: [https://localhost:5001/swagger](https://localhost:5001/swagger)</span></span>

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="53ab8-182">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="53ab8-182">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

<span data-ttu-id="53ab8-183">Selezionare **Esegui**  >  **Avvia debug** per avviare l'app.</span><span class="sxs-lookup"><span data-stu-id="53ab8-183">Select **Run** > **Start Debugging** to launch the app.</span></span> <span data-ttu-id="53ab8-184">Visual Studio per Mac apre un browser e naviga all'indirizzo `https://localhost:<port>`, dove `<port>` è un numero di porta selezionato a caso.</span><span class="sxs-lookup"><span data-stu-id="53ab8-184">Visual Studio for Mac launches a browser and navigates to `https://localhost:<port>`, where `<port>` is a randomly chosen port number.</span></span> <span data-ttu-id="53ab8-185">Viene restituito un errore HTTP 404 (Non trovato).</span><span class="sxs-lookup"><span data-stu-id="53ab8-185">An HTTP 404 (Not Found) error is returned.</span></span> <span data-ttu-id="53ab8-186">Accodare `/swagger` all'URL (impostare l'URL su `https://localhost:<port>/swagger`).</span><span class="sxs-lookup"><span data-stu-id="53ab8-186">Append `/swagger` to the URL (change the URL to `https://localhost:<port>/swagger`).</span></span>

---

<span data-ttu-id="53ab8-187">Viene visualizzata la pagina di spavalderia `/swagger/index.html` .</span><span class="sxs-lookup"><span data-stu-id="53ab8-187">The Swagger page `/swagger/index.html` is displayed.</span></span> <span data-ttu-id="53ab8-188">Selezionare **Get**  >  **try it out**  >  **Execute (Esegui**).</span><span class="sxs-lookup"><span data-stu-id="53ab8-188">Select **GET** > **Try it out** > **Execute**.</span></span> <span data-ttu-id="53ab8-189">Viene visualizzata la pagina:</span><span class="sxs-lookup"><span data-stu-id="53ab8-189">The page displays:</span></span>

* <span data-ttu-id="53ab8-190">Comando [curl](https://curl.haxx.se/) per testare l'API weatherforecast.</span><span class="sxs-lookup"><span data-stu-id="53ab8-190">The [Curl](https://curl.haxx.se/) command to test the WeatherForecast API.</span></span>
* <span data-ttu-id="53ab8-191">URL per testare l'API WeatherForecast.</span><span class="sxs-lookup"><span data-stu-id="53ab8-191">The URL to test the WeatherForecast API.</span></span>
* <span data-ttu-id="53ab8-192">Il codice di risposta, il corpo e le intestazioni.</span><span class="sxs-lookup"><span data-stu-id="53ab8-192">The response code, body, and headers.</span></span>
* <span data-ttu-id="53ab8-193">Casella di riepilogo a discesa con i tipi di supporto e il valore e lo schema di esempio.</span><span class="sxs-lookup"><span data-stu-id="53ab8-193">A drop down list box with media types and the example value and schema.</span></span>

<!-- Review: Do we care the IE generates several errors. It shows the data, but with  Unrecognized response type; displaying content as text.
-->
<span data-ttu-id="53ab8-194">Spavalderia viene usato per generare la documentazione e le pagine della guida utili per le API Web.</span><span class="sxs-lookup"><span data-stu-id="53ab8-194">Swagger is used to generate useful documentation and help pages for web APIs.</span></span> <span data-ttu-id="53ab8-195">Questa esercitazione è incentrata sulla creazione di un'API Web.</span><span class="sxs-lookup"><span data-stu-id="53ab8-195">This tutorial focuses on creating a web API.</span></span> <span data-ttu-id="53ab8-196">Per ulteriori informazioni su spavalderia, vedere <xref:tutorials/web-api-help-pages-using-swagger> .</span><span class="sxs-lookup"><span data-stu-id="53ab8-196">For more information on Swagger, see <xref:tutorials/web-api-help-pages-using-swagger>.</span></span>

<span data-ttu-id="53ab8-197">Copiare e incollare l' **URL della richiesta** nel browser:  `https://localhost:<port>/WeatherForecast`</span><span class="sxs-lookup"><span data-stu-id="53ab8-197">Copy and paste the **Request URL** in the browser:  `https://localhost:<port>/WeatherForecast`</span></span>

<span data-ttu-id="53ab8-198">Viene restituito un codice JSON simile al seguente:</span><span class="sxs-lookup"><span data-stu-id="53ab8-198">JSON similar to the following is returned:</span></span>

```json
[
    {
        "date": "2019-07-16T19:04:05.7257911-06:00",
        "temperatureC": 52,
        "temperatureF": 125,
        "summary": "Mild"
    },
    {
        "date": "2019-07-17T19:04:05.7258461-06:00",
        "temperatureC": 36,
        "temperatureF": 96,
        "summary": "Warm"
    },
    {
        "date": "2019-07-18T19:04:05.7258467-06:00",
        "temperatureC": 39,
        "temperatureF": 102,
        "summary": "Cool"
    },
    {
        "date": "2019-07-19T19:04:05.7258471-06:00",
        "temperatureC": 10,
        "temperatureF": 49,
        "summary": "Bracing"
    },
    {
        "date": "2019-07-20T19:04:05.7258474-06:00",
        "temperatureC": -1,
        "temperatureF": 31,
        "summary": "Chilly"
    }
]
```

### <a name="update-the-launchurl"></a><span data-ttu-id="53ab8-199">Aggiornare launchUrl</span><span class="sxs-lookup"><span data-stu-id="53ab8-199">Update the launchUrl</span></span>

<span data-ttu-id="53ab8-200">In *Properties\launchSettings.json* aggiornare `launchUrl` da `"swagger"` a `"api/TodoItems"` :</span><span class="sxs-lookup"><span data-stu-id="53ab8-200">In *Properties\launchSettings.json*, update `launchUrl` from `"swagger"` to `"api/TodoItems"`:</span></span>

```json
"launchUrl": "api/TodoItems",
```

<span data-ttu-id="53ab8-201">Poiché spavalderia è stato rimosso, il markup precedente modifica l'URL che viene avviato al metodo GET del controller aggiunto nelle sezioni riportate di seguito.</span><span class="sxs-lookup"><span data-stu-id="53ab8-201">Because Swagger has been removed, the preceding markup changes the URL that is launched to the GET method of the controller added in the following sections.</span></span>

## <a name="add-a-model-class"></a><span data-ttu-id="53ab8-202">Aggiungere una classe del modello</span><span class="sxs-lookup"><span data-stu-id="53ab8-202">Add a model class</span></span>

<span data-ttu-id="53ab8-203">Un *modello* è un set di classi che rappresentano i dati gestiti dall'app.</span><span class="sxs-lookup"><span data-stu-id="53ab8-203">A *model* is a set of classes that represent the data that the app manages.</span></span> <span data-ttu-id="53ab8-204">Il modello per questa app è una classe `TodoItem` singola.</span><span class="sxs-lookup"><span data-stu-id="53ab8-204">The model for this app is a single `TodoItem` class.</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="53ab8-205">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="53ab8-205">Visual Studio</span></span>](#tab/visual-studio)

* <span data-ttu-id="53ab8-206">In **Esplora soluzioni** fare clic con il pulsante destro del mouse sul progetto.</span><span class="sxs-lookup"><span data-stu-id="53ab8-206">In **Solution Explorer**, right-click the project.</span></span> <span data-ttu-id="53ab8-207">Selezionare **Aggiungi**  >  **nuova cartella**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-207">Select **Add** > **New Folder**.</span></span> <span data-ttu-id="53ab8-208">Assegnare un nome alla cartella *Models* .</span><span class="sxs-lookup"><span data-stu-id="53ab8-208">Name the folder *Models*.</span></span>

* <span data-ttu-id="53ab8-209">Fare clic con il pulsante destro del mouse sulla *Models* cartella e scegliere **Aggiungi**  >  **classe**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-209">Right-click the *Models* folder and select **Add** > **Class**.</span></span> <span data-ttu-id="53ab8-210">Assegnare alla classe il nome *TodoItem* e selezionare **Aggiungi**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-210">Name the class *TodoItem* and select **Add**.</span></span>

* <span data-ttu-id="53ab8-211">Sostituire il codice del modello con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="53ab8-211">Replace the template code with the following:</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="53ab8-212">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="53ab8-212">Visual Studio Code</span></span>](#tab/visual-studio-code)

* <span data-ttu-id="53ab8-213">Aggiungere una cartella denominata *Models* .</span><span class="sxs-lookup"><span data-stu-id="53ab8-213">Add a folder named *Models*.</span></span>

* <span data-ttu-id="53ab8-214">Aggiungere una `TodoItem` classe alla *Models* cartella con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="53ab8-214">Add a `TodoItem` class to the *Models* folder with the following code:</span></span>

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="53ab8-215">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="53ab8-215">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

* <span data-ttu-id="53ab8-216">Fare clic con il pulsante destro del mouse sul progetto.</span><span class="sxs-lookup"><span data-stu-id="53ab8-216">Right-click the project.</span></span> <span data-ttu-id="53ab8-217">Selezionare **Aggiungi**  >  **nuova cartella**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-217">Select **Add** > **New Folder**.</span></span> <span data-ttu-id="53ab8-218">Assegnare un nome alla cartella *Models* .</span><span class="sxs-lookup"><span data-stu-id="53ab8-218">Name the folder *Models*.</span></span>

  ![Nuova cartella](first-web-api-mac/_static/folder.png)

* <span data-ttu-id="53ab8-220">Fare clic con il pulsante destro del mouse sulla *Models* cartella e scegliere **Aggiungi** > **nuovo file** > **generale** > **vuota classe**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-220">Right-click the *Models* folder, and select **Add** > **New File** > **General** > **Empty Class**.</span></span>

* <span data-ttu-id="53ab8-221">Assegnare alla classe il nome *TodoItem* e fare clic su **Nuovo**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-221">Name the class *TodoItem*, and then click **New**.</span></span>

* <span data-ttu-id="53ab8-222">Sostituire il codice del modello con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="53ab8-222">Replace the template code with the following:</span></span>

---

  [!code-csharp[](first-web-api/samples/5.x/TodoApi/Models/TodoItem.cs?name=snippet)]

<span data-ttu-id="53ab8-223">La proprietà `Id` funziona come chiave univoca in un database relazionale.</span><span class="sxs-lookup"><span data-stu-id="53ab8-223">The `Id` property functions as the unique key in a relational database.</span></span>

<span data-ttu-id="53ab8-224">Le classi di modelli possono essere inserite in qualsiasi punto del progetto, ma la *Models* cartella viene utilizzata per convenzione.</span><span class="sxs-lookup"><span data-stu-id="53ab8-224">Model classes can go anywhere in the project, but the *Models* folder is used by convention.</span></span>

## <a name="add-a-database-context"></a><span data-ttu-id="53ab8-225">Aggiungere un contesto di database</span><span class="sxs-lookup"><span data-stu-id="53ab8-225">Add a database context</span></span>

<span data-ttu-id="53ab8-226">Il *contesto di database* è la classe principale che coordina le funzionalità di Entity Framework per un modello di dati.</span><span class="sxs-lookup"><span data-stu-id="53ab8-226">The *database context* is the main class that coordinates Entity Framework functionality for a data model.</span></span> <span data-ttu-id="53ab8-227">Questa classe viene creata mediante derivazione dalla classe <xref:Microsoft.EntityFrameworkCore.DbContext?displayProperty=fullName>.</span><span class="sxs-lookup"><span data-stu-id="53ab8-227">This class is created by deriving from the <xref:Microsoft.EntityFrameworkCore.DbContext?displayProperty=fullName> class.</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="53ab8-228">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="53ab8-228">Visual Studio</span></span>](#tab/visual-studio)

### <a name="add-nuget-packages"></a><span data-ttu-id="53ab8-229">Aggiungere pacchetti NuGet</span><span class="sxs-lookup"><span data-stu-id="53ab8-229">Add NuGet packages</span></span>

* <span data-ttu-id="53ab8-230">Dal menu **Strumenti** selezionare **Gestione pacchetti NuGet > Gestisci pacchetti NuGet per la soluzione**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-230">From the **Tools** menu, select **NuGet Package Manager > Manage NuGet Packages for Solution**.</span></span>
* <span data-ttu-id="53ab8-231">Selezionare la scheda **Sfoglia** e quindi immettere **Microsoft. EntityFrameworkCore. InMemory** nella casella di ricerca.</span><span class="sxs-lookup"><span data-stu-id="53ab8-231">Select the **Browse** tab, and then enter **Microsoft.EntityFrameworkCore.InMemory** in the search box.</span></span>
* <span data-ttu-id="53ab8-232">Selezionare **Microsoft. EntityFrameworkCore. InMemory** nel riquadro sinistro.</span><span class="sxs-lookup"><span data-stu-id="53ab8-232">Select **Microsoft.EntityFrameworkCore.InMemory** in the left pane.</span></span>
* <span data-ttu-id="53ab8-233">Selezionare la casella di controllo **Progetto** nel riquadro di destra e quindi selezionare **Installa**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-233">Select the **Project** check box in the right pane and then select **Install**.</span></span>

![Gestione pacchetti NuGet](first-web-api/_static/5/vsNuGet.png)

## <a name="add-the-todocontext-database-context"></a><span data-ttu-id="53ab8-235">Aggiungere il contesto del database TodoContext</span><span class="sxs-lookup"><span data-stu-id="53ab8-235">Add the TodoContext database context</span></span>

* <span data-ttu-id="53ab8-236">Fare clic con il pulsante destro del mouse sulla *Models* cartella e scegliere **Aggiungi**  >  **classe**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-236">Right-click the *Models* folder and select **Add** > **Class**.</span></span> <span data-ttu-id="53ab8-237">Assegnare alla classe il nome *TodoContext* e fare clic su **Aggiungi**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-237">Name the class *TodoContext* and click **Add**.</span></span>

# <a name="visual-studio-code--visual-studio-for-mac"></a>[<span data-ttu-id="53ab8-238">Visual Studio Code / Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="53ab8-238">Visual Studio Code / Visual Studio for Mac</span></span>](#tab/visual-studio-code+visual-studio-mac)

* <span data-ttu-id="53ab8-239">Aggiungere una `TodoContext` classe alla *Models* cartella.</span><span class="sxs-lookup"><span data-stu-id="53ab8-239">Add a `TodoContext` class to the *Models* folder.</span></span>

---

* <span data-ttu-id="53ab8-240">Immettere il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="53ab8-240">Enter the following code:</span></span>

  [!code-csharp[](first-web-api/samples/5.x/TodoApi/Models/TodoContext.cs)]

## <a name="register-the-database-context"></a><span data-ttu-id="53ab8-241">Registrare il contesto del database</span><span class="sxs-lookup"><span data-stu-id="53ab8-241">Register the database context</span></span>

<span data-ttu-id="53ab8-242">In ASP.NET Core i servizi come il contesto del database devono essere registrati con il [contenitore di inserimento delle dipendenze](xref:fundamentals/dependency-injection).</span><span class="sxs-lookup"><span data-stu-id="53ab8-242">In ASP.NET Core, services such as the DB context must be registered with the [dependency injection (DI)](xref:fundamentals/dependency-injection) container.</span></span> <span data-ttu-id="53ab8-243">Il contenitore rende disponibile il servizio ai controller.</span><span class="sxs-lookup"><span data-stu-id="53ab8-243">The container provides the service to controllers.</span></span>

<span data-ttu-id="53ab8-244">Aggiornare *Startup.cs* con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="53ab8-244">Update *Startup.cs* with the following code:</span></span>

[!code-csharp[](first-web-api/samples/5.x/TodoApi/Startup.cs?highlight=7-8,23-24&name=snippet_all)]

<span data-ttu-id="53ab8-245">Il codice precedente:</span><span class="sxs-lookup"><span data-stu-id="53ab8-245">The preceding code:</span></span>

* <span data-ttu-id="53ab8-246">Rimuove le chiamate di spavalderia.</span><span class="sxs-lookup"><span data-stu-id="53ab8-246">Removes the Swagger calls.</span></span>
* <span data-ttu-id="53ab8-247">Rimuove le dichiarazioni `using` inutilizzate.</span><span class="sxs-lookup"><span data-stu-id="53ab8-247">Removes unused `using` declarations.</span></span>
* <span data-ttu-id="53ab8-248">Aggiunge il contesto del database al contenitore di inserimento delle dipendenze.</span><span class="sxs-lookup"><span data-stu-id="53ab8-248">Adds the database context to the DI container.</span></span>
* <span data-ttu-id="53ab8-249">Specifica che il contesto del database userà un database in memoria.</span><span class="sxs-lookup"><span data-stu-id="53ab8-249">Specifies that the database context will use an in-memory database.</span></span>

## <a name="scaffold-a-controller"></a><span data-ttu-id="53ab8-250">Eseguire lo scaffolding di un controller</span><span class="sxs-lookup"><span data-stu-id="53ab8-250">Scaffold a controller</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="53ab8-251">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="53ab8-251">Visual Studio</span></span>](#tab/visual-studio)

* <span data-ttu-id="53ab8-252">Fare clic con il pulsante destro del mouse sulla cartella *Controllers*.</span><span class="sxs-lookup"><span data-stu-id="53ab8-252">Right-click the *Controllers* folder.</span></span>
* <span data-ttu-id="53ab8-253">Selezionare **Aggiungi** > **Nuovo elemento di scaffolding**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-253">Select **Add** > **New Scaffolded Item**.</span></span>
* <span data-ttu-id="53ab8-254">Selezionare **Controller API con azioni, che usa Entity Framework** e quindi selezionare **Aggiungi**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-254">Select **API Controller with actions, using Entity Framework**, and then select **Add**.</span></span>
* <span data-ttu-id="53ab8-255">Nella finestra di dialogo **Add API Controller with actions, using Entity Framework** (Aggiungi controller API con azioni, che usa Entity Framework):</span><span class="sxs-lookup"><span data-stu-id="53ab8-255">In the **Add API Controller with actions, using Entity Framework** dialog:</span></span>

  * <span data-ttu-id="53ab8-256">Selezionare **TodoItem (TodoApi. Models )** nella **classe del modello**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-256">Select **TodoItem (TodoApi.Models)** in the **Model class**.</span></span>
  * <span data-ttu-id="53ab8-257">Selezionare **TodoContext (TodoApi. Models )** nella **classe del contesto dati**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-257">Select **TodoContext (TodoApi.Models)** in the **Data context class**.</span></span>
  * <span data-ttu-id="53ab8-258">Selezionare **Aggiungi**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-258">Select **Add**.</span></span>

# <a name="visual-studio-code--visual-studio-for-mac"></a>[<span data-ttu-id="53ab8-259">Visual Studio Code / Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="53ab8-259">Visual Studio Code / Visual Studio for Mac</span></span>](#tab/visual-studio-code+visual-studio-mac)

<span data-ttu-id="53ab8-260">Eseguire i comandi seguenti:</span><span class="sxs-lookup"><span data-stu-id="53ab8-260">Run the following commands:</span></span>

```dotnetcli
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
dotnet add package Microsoft.EntityFrameworkCore.Design
dotnet tool install -g dotnet-aspnet-codegenerator
dotnet tool update -g dotnet-aspnet-codegenerator
dotnet aspnet-codegenerator controller -name TodoItemsController -async -api -m TodoItem -dc TodoContext -outDir Controllers
```

<span data-ttu-id="53ab8-261">I comandi precedenti:</span><span class="sxs-lookup"><span data-stu-id="53ab8-261">The preceding commands:</span></span>

* <span data-ttu-id="53ab8-262">Aggiungere i pacchetti NuGet necessari per lo scaffolding.</span><span class="sxs-lookup"><span data-stu-id="53ab8-262">Add NuGet packages required for scaffolding.</span></span>
* <span data-ttu-id="53ab8-263">Installa il motore di scaffolding (`dotnet-aspnet-codegenerator`).</span><span class="sxs-lookup"><span data-stu-id="53ab8-263">Installs the scaffolding engine (`dotnet-aspnet-codegenerator`).</span></span>
* <span data-ttu-id="53ab8-264">Esegue lo scaffolding di `TodoItemsController`.</span><span class="sxs-lookup"><span data-stu-id="53ab8-264">Scaffolds the `TodoItemsController`.</span></span>

---

<span data-ttu-id="53ab8-265">Il codice generato:</span><span class="sxs-lookup"><span data-stu-id="53ab8-265">The generated code:</span></span>

* <span data-ttu-id="53ab8-266">Contrassegna la classe con l' [`[ApiController]`](xref:Microsoft.AspNetCore.Mvc.ApiControllerAttribute) attributo.</span><span class="sxs-lookup"><span data-stu-id="53ab8-266">Marks the class with the [`[ApiController]`](xref:Microsoft.AspNetCore.Mvc.ApiControllerAttribute) attribute.</span></span> <span data-ttu-id="53ab8-267">L'attributo indica che il controller risponde alle richieste di API Web.</span><span class="sxs-lookup"><span data-stu-id="53ab8-267">This attribute indicates that the controller responds to web API requests.</span></span> <span data-ttu-id="53ab8-268">Per informazioni sui comportamenti specifici consentiti dall'attributo, vedere <xref:web-api/index>.</span><span class="sxs-lookup"><span data-stu-id="53ab8-268">For information about specific behaviors that the attribute enables, see <xref:web-api/index>.</span></span>
* <span data-ttu-id="53ab8-269">Usa l'inserimento delle dipendenze per inserire il contesto del database (`TodoContext`) nel controller.</span><span class="sxs-lookup"><span data-stu-id="53ab8-269">Uses DI to inject the database context (`TodoContext`) into the controller.</span></span> <span data-ttu-id="53ab8-270">Il contesto di database viene usato in ognuno dei metodi [CRUD](https://wikipedia.org/wiki/Create,_read,_update_and_delete) nel controller.</span><span class="sxs-lookup"><span data-stu-id="53ab8-270">The database context is used in each of the [CRUD](https://wikipedia.org/wiki/Create,_read,_update_and_delete) methods in the controller.</span></span>

<span data-ttu-id="53ab8-271">I modelli di ASP.NET Core per:</span><span class="sxs-lookup"><span data-stu-id="53ab8-271">The ASP.NET Core templates for:</span></span>

* <span data-ttu-id="53ab8-272">I controller con le visualizzazioni includono `[action]` nel modello di route.</span><span class="sxs-lookup"><span data-stu-id="53ab8-272">Controllers with views include `[action]` in the route template.</span></span>
* <span data-ttu-id="53ab8-273">I controller API non includono `[action]` nel modello di route.</span><span class="sxs-lookup"><span data-stu-id="53ab8-273">API controllers don't include `[action]` in the route template.</span></span>

<span data-ttu-id="53ab8-274">Quando il `[action]` token non è incluso nel modello di route, il nome dell' [azione](xref:mvc/controllers/routing#action) viene escluso dalla route.</span><span class="sxs-lookup"><span data-stu-id="53ab8-274">When the `[action]` token isn't in the route template, the [action](xref:mvc/controllers/routing#action) name is excluded from the route.</span></span> <span data-ttu-id="53ab8-275">Ovvero, il nome del metodo associato all'azione non viene usato nella route corrispondente.</span><span class="sxs-lookup"><span data-stu-id="53ab8-275">That is, the action's associated method name isn't used in the matching route.</span></span>

## <a name="update-the-posttodoitem-create-method"></a><span data-ttu-id="53ab8-276">Aggiornare il metodo di creazione PostTodoItem</span><span class="sxs-lookup"><span data-stu-id="53ab8-276">Update the PostTodoItem create method</span></span>

<span data-ttu-id="53ab8-277">Sostituire l'istruzione return in `PostTodoItem` per usare l'operatore [nameof](/dotnet/csharp/language-reference/operators/nameof):</span><span class="sxs-lookup"><span data-stu-id="53ab8-277">Replace the return statement in the `PostTodoItem` to use the [nameof](/dotnet/csharp/language-reference/operators/nameof) operator:</span></span>

[!code-csharp[](first-web-api/samples/5.x/TodoApi/Controllers/TodoItemsController.cs?name=snippet_Create)]

<span data-ttu-id="53ab8-278">Il codice precedente è un metodo HTTP POST, come indicato dall' [`[HttpPost]`](xref:Microsoft.AspNetCore.Mvc.HttpPostAttribute) attributo.</span><span class="sxs-lookup"><span data-stu-id="53ab8-278">The preceding code is an HTTP POST method, as indicated by the [`[HttpPost]`](xref:Microsoft.AspNetCore.Mvc.HttpPostAttribute) attribute.</span></span> <span data-ttu-id="53ab8-279">Il metodo ottiene il valore dell'elemento attività dal corpo della richiesta HTTP.</span><span class="sxs-lookup"><span data-stu-id="53ab8-279">The method gets the value of the to-do item from the body of the HTTP request.</span></span>

<span data-ttu-id="53ab8-280">Per altre informazioni, vedere [Routing con attributi Http[verb]](xref:mvc/controllers/routing#attribute-routing-with-httpverb-attributes).</span><span class="sxs-lookup"><span data-stu-id="53ab8-280">For more information, see [Attribute routing with Http[Verb] attributes](xref:mvc/controllers/routing#attribute-routing-with-httpverb-attributes).</span></span>

<span data-ttu-id="53ab8-281">Il metodo <xref:Microsoft.AspNetCore.Mvc.ControllerBase.CreatedAtAction*>:</span><span class="sxs-lookup"><span data-stu-id="53ab8-281">The <xref:Microsoft.AspNetCore.Mvc.ControllerBase.CreatedAtAction*> method:</span></span>

* <span data-ttu-id="53ab8-282">Restituisce un [codice di stato HTTP 201](https://developer.mozilla.org/docs/Web/HTTP/Status/201) in caso di esito positivo.</span><span class="sxs-lookup"><span data-stu-id="53ab8-282">Returns an [HTTP 201 status code](https://developer.mozilla.org/docs/Web/HTTP/Status/201) if successful.</span></span> <span data-ttu-id="53ab8-283">HTTP 201 è la risposta standard per un metodo HTTP POST che crea una nuova risorsa nel server.</span><span class="sxs-lookup"><span data-stu-id="53ab8-283">HTTP 201 is the standard response for an HTTP POST method that creates a new resource on the server.</span></span>
* <span data-ttu-id="53ab8-284">Aggiunge un'intestazione [Location](https://developer.mozilla.org/docs/Web/HTTP/Headers/Location) alla risposta.</span><span class="sxs-lookup"><span data-stu-id="53ab8-284">Adds a [Location](https://developer.mozilla.org/docs/Web/HTTP/Headers/Location) header to the response.</span></span> <span data-ttu-id="53ab8-285">L'intestazione `Location` specifica l'[URI](https://developer.mozilla.org/docs/Glossary/URI) dell'elemento attività appena creato.</span><span class="sxs-lookup"><span data-stu-id="53ab8-285">The `Location` header specifies the [URI](https://developer.mozilla.org/docs/Glossary/URI) of the newly created to-do item.</span></span> <span data-ttu-id="53ab8-286">Per altre informazioni, vedere [10.2.2 201 Created](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html).</span><span class="sxs-lookup"><span data-stu-id="53ab8-286">For more information, see [10.2.2 201 Created](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html).</span></span>
* <span data-ttu-id="53ab8-287">Fa riferimento all'azione `GetTodoItem` per creare l'URI dell'intestazione `Location`.</span><span class="sxs-lookup"><span data-stu-id="53ab8-287">References the `GetTodoItem` action to create the `Location` header's URI.</span></span> <span data-ttu-id="53ab8-288">La parola chiave `nameof` C# viene usata per evitare di impostare il nome dell'azione come hardcoded nella chiamata a `CreatedAtAction`.</span><span class="sxs-lookup"><span data-stu-id="53ab8-288">The C# `nameof` keyword is used to avoid hard-coding the action name in the `CreatedAtAction` call.</span></span>

### <a name="install-postman"></a><span data-ttu-id="53ab8-289">Installare Postman</span><span class="sxs-lookup"><span data-stu-id="53ab8-289">Install Postman</span></span>

<span data-ttu-id="53ab8-290">Questa esercitazione usa Postman per testare l'API Web.</span><span class="sxs-lookup"><span data-stu-id="53ab8-290">This tutorial uses Postman to test the web API.</span></span>

* <span data-ttu-id="53ab8-291">Installa il [post](https://www.getpostman.com/downloads/)</span><span class="sxs-lookup"><span data-stu-id="53ab8-291">Install [Postman](https://www.getpostman.com/downloads/)</span></span>
* <span data-ttu-id="53ab8-292">Avviare l'app Web.</span><span class="sxs-lookup"><span data-stu-id="53ab8-292">Start the web app.</span></span>
* <span data-ttu-id="53ab8-293">Avviare Postman.</span><span class="sxs-lookup"><span data-stu-id="53ab8-293">Start Postman.</span></span>
* <span data-ttu-id="53ab8-294">Disattivare **SSL certificate verification** (Verifica certificato SSL)</span><span class="sxs-lookup"><span data-stu-id="53ab8-294">Disable **SSL certificate verification**</span></span>
  * <span data-ttu-id="53ab8-295">Da **File** > **Settings** (Impostazioni) (scheda **General** (Generale)) disabilitare **SSL certificate verification** (Verifica certificato SSL).</span><span class="sxs-lookup"><span data-stu-id="53ab8-295">From **File** > **Settings** (**General** tab), disable **SSL certificate verification**.</span></span>
    > [!WARNING]
    > <span data-ttu-id="53ab8-296">Riattivare la verifica dei certificati SSL al termine del test del controller.</span><span class="sxs-lookup"><span data-stu-id="53ab8-296">Re-enable SSL certificate verification after testing the controller.</span></span>

<a name="post"></a>

### <a name="test-posttodoitem-with-postman"></a><span data-ttu-id="53ab8-297">Testare PostTodoItem con Postman</span><span class="sxs-lookup"><span data-stu-id="53ab8-297">Test PostTodoItem with Postman</span></span>

* <span data-ttu-id="53ab8-298">Creare una nuova richiesta.</span><span class="sxs-lookup"><span data-stu-id="53ab8-298">Create a new request.</span></span>
* <span data-ttu-id="53ab8-299">Impostare il metodo HTTP su `POST`.</span><span class="sxs-lookup"><span data-stu-id="53ab8-299">Set the HTTP method to `POST`.</span></span>
* <span data-ttu-id="53ab8-300">Impostare l'URI su `https://localhost:<port>/api/TodoItems` .</span><span class="sxs-lookup"><span data-stu-id="53ab8-300">Set the URI to `https://localhost:<port>/api/TodoItems`.</span></span> <span data-ttu-id="53ab8-301">Ad esempio: `https://localhost:5001/api/TodoItems`.</span><span class="sxs-lookup"><span data-stu-id="53ab8-301">For example, `https://localhost:5001/api/TodoItems`.</span></span>
* <span data-ttu-id="53ab8-302">Selezionare la scheda **Corpo**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-302">Select the **Body** tab.</span></span>
* <span data-ttu-id="53ab8-303">Selezionare il pulsante di opzione **raw** (non elaborato).</span><span class="sxs-lookup"><span data-stu-id="53ab8-303">Select the **raw** radio button.</span></span>
* <span data-ttu-id="53ab8-304">Impostare il tipo su **JSON (application/json)**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-304">Set the type to **JSON (application/json)**.</span></span>
* <span data-ttu-id="53ab8-305">Nel corpo della richiesta immettere codice JSON per un elemento attività:</span><span class="sxs-lookup"><span data-stu-id="53ab8-305">In the request body enter JSON for a to-do item:</span></span>

    ```json
    {
      "name":"walk dog",
      "isComplete":true
    }
    ```

* <span data-ttu-id="53ab8-306">Selezionare **Invia**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-306">Select **Send**.</span></span>

  ![Postman con richiesta di creazione](first-web-api/_static/3/create.png)

### <a name="test-the-location-header-uri"></a><span data-ttu-id="53ab8-308">Testare l'URI dell'intestazione della posizione</span><span class="sxs-lookup"><span data-stu-id="53ab8-308">Test the location header URI</span></span>

<span data-ttu-id="53ab8-309">L'URI dell'intestazione Location può essere testato nel browser.</span><span class="sxs-lookup"><span data-stu-id="53ab8-309">The location header URI can be tested in the browser.</span></span> <span data-ttu-id="53ab8-310">Copiare e incollare l'URI dell'intestazione location nel browser.</span><span class="sxs-lookup"><span data-stu-id="53ab8-310">Copy and paste the location header URI into the browser.</span></span>

<span data-ttu-id="53ab8-311">Per eseguire il test in un post:</span><span class="sxs-lookup"><span data-stu-id="53ab8-311">To test in Postman:</span></span>

* <span data-ttu-id="53ab8-312">Selezionare la scheda **Headers** (Intestazioni) nel riquadro **Response** (Risposta).</span><span class="sxs-lookup"><span data-stu-id="53ab8-312">Select the **Headers** tab in the **Response** pane.</span></span>
* <span data-ttu-id="53ab8-313">Copiare il valore dell'intestazione **Location** (Posizione):</span><span class="sxs-lookup"><span data-stu-id="53ab8-313">Copy the **Location** header value:</span></span>

  ![Scheda Headers (Intestazioni) della console Postman](first-web-api/_static/3/create.png)

* <span data-ttu-id="53ab8-315">Impostare il metodo HTTP su `GET`.</span><span class="sxs-lookup"><span data-stu-id="53ab8-315">Set the HTTP method to `GET`.</span></span>
* <span data-ttu-id="53ab8-316">Impostare l'URI su `https://localhost:<port>/api/TodoItems/1` .</span><span class="sxs-lookup"><span data-stu-id="53ab8-316">Set the URI to `https://localhost:<port>/api/TodoItems/1`.</span></span> <span data-ttu-id="53ab8-317">Ad esempio: `https://localhost:5001/api/TodoItems/1`.</span><span class="sxs-lookup"><span data-stu-id="53ab8-317">For example, `https://localhost:5001/api/TodoItems/1`.</span></span>
* <span data-ttu-id="53ab8-318">Selezionare **Invia**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-318">Select **Send**.</span></span>

## <a name="examine-the-get-methods"></a><span data-ttu-id="53ab8-319">Esaminare i metodi GET</span><span class="sxs-lookup"><span data-stu-id="53ab8-319">Examine the GET methods</span></span>

<span data-ttu-id="53ab8-320">Vengono implementati due endpoint GET:</span><span class="sxs-lookup"><span data-stu-id="53ab8-320">Two GET endpoints are implemented:</span></span>

* `GET /api/TodoItems`
* `GET /api/TodoItems/{id}`

<span data-ttu-id="53ab8-321">Testare l'app chiamando i due endpoint da un browser o da Postman.</span><span class="sxs-lookup"><span data-stu-id="53ab8-321">Test the app by calling the two endpoints from a browser or Postman.</span></span> <span data-ttu-id="53ab8-322">Ad esempio:</span><span class="sxs-lookup"><span data-stu-id="53ab8-322">For example:</span></span>

* `https://localhost:5001/api/TodoItems`
* `https://localhost:5001/api/TodoItems/1`

<span data-ttu-id="53ab8-323">Una risposta simile alla seguente viene generata dalla chiamata a `GetTodoItems`:</span><span class="sxs-lookup"><span data-stu-id="53ab8-323">A response similar to the following is produced by the call to `GetTodoItems`:</span></span>

```json
[
  {
    "id": 1,
    "name": "Item1",
    "isComplete": false
  }
]
```

### <a name="test-get-with-postman"></a><span data-ttu-id="53ab8-324">Testare Get con Postman</span><span class="sxs-lookup"><span data-stu-id="53ab8-324">Test Get with Postman</span></span>

* <span data-ttu-id="53ab8-325">Creare una nuova richiesta.</span><span class="sxs-lookup"><span data-stu-id="53ab8-325">Create a new request.</span></span>
* <span data-ttu-id="53ab8-326">Impostare il metodo HTTP su **GET**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-326">Set the HTTP method to **GET**.</span></span>
* <span data-ttu-id="53ab8-327">Impostare l'URI della richiesta su `https://localhost:<port>/api/TodoItems` .</span><span class="sxs-lookup"><span data-stu-id="53ab8-327">Set the request URI to `https://localhost:<port>/api/TodoItems`.</span></span> <span data-ttu-id="53ab8-328">Ad esempio: `https://localhost:5001/api/TodoItems`.</span><span class="sxs-lookup"><span data-stu-id="53ab8-328">For example, `https://localhost:5001/api/TodoItems`.</span></span>
* <span data-ttu-id="53ab8-329">Impostare **Two pane view** (Visualizzazione in due riquadri) in Postman.</span><span class="sxs-lookup"><span data-stu-id="53ab8-329">Set **Two pane view** in Postman.</span></span>
* <span data-ttu-id="53ab8-330">Selezionare **Invia**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-330">Select **Send**.</span></span>

<span data-ttu-id="53ab8-331">Questa app usa un database in memoria.</span><span class="sxs-lookup"><span data-stu-id="53ab8-331">This app uses an in-memory database.</span></span> <span data-ttu-id="53ab8-332">Se l'app viene arrestata e avviata, la richiesta GET precedente non restituirà alcun dato.</span><span class="sxs-lookup"><span data-stu-id="53ab8-332">If the app is stopped and started, the preceding GET request will not return any data.</span></span> <span data-ttu-id="53ab8-333">Se non vengono restituiti dati, eseguire [POST](#post) per pubblicare i dati nell'app.</span><span class="sxs-lookup"><span data-stu-id="53ab8-333">If no data is returned, [POST](#post) data to the app.</span></span>

## <a name="routing-and-url-paths"></a><span data-ttu-id="53ab8-334">Routing e percorsi di URL</span><span class="sxs-lookup"><span data-stu-id="53ab8-334">Routing and URL paths</span></span>

<span data-ttu-id="53ab8-335">L' [`[HttpGet]`](xref:Microsoft.AspNetCore.Mvc.HttpGetAttribute) attributo indica un metodo che risponde a una richiesta HTTP Get.</span><span class="sxs-lookup"><span data-stu-id="53ab8-335">The [`[HttpGet]`](xref:Microsoft.AspNetCore.Mvc.HttpGetAttribute) attribute denotes a method that responds to an HTTP GET request.</span></span> <span data-ttu-id="53ab8-336">Il percorso dell'URL per ogni metodo viene costruito nel modo seguente:</span><span class="sxs-lookup"><span data-stu-id="53ab8-336">The URL path for each method is constructed as follows:</span></span>

* <span data-ttu-id="53ab8-337">Iniziare con la stringa di modello nell'attributo `Route` del controller:</span><span class="sxs-lookup"><span data-stu-id="53ab8-337">Start with the template string in the controller's `Route` attribute:</span></span>

  [!code-csharp[](first-web-api/samples/5.x/TodoApi/Controllers/TodoItemsController.cs?name=TodoController&highlight=1)]

* <span data-ttu-id="53ab8-338">Sostituire `[controller]` con il nome del controller, ovvero, per convenzione, il nome della classe controller meno il suffisso "Controller".</span><span class="sxs-lookup"><span data-stu-id="53ab8-338">Replace `[controller]` with the name of the controller, which by convention is the controller class name minus the "Controller" suffix.</span></span> <span data-ttu-id="53ab8-339">In questo esempio il nome della classe controller è **TodoItems** Controller, quindi il nome del controller è "TodoItems".</span><span class="sxs-lookup"><span data-stu-id="53ab8-339">For this sample, the controller class name is **TodoItems** Controller, so the controller name is "TodoItems".</span></span> <span data-ttu-id="53ab8-340">Il [routing](xref:mvc/controllers/routing) ASP.NET Core non fa distinzione tra maiuscole e minuscole.</span><span class="sxs-lookup"><span data-stu-id="53ab8-340">ASP.NET Core [routing](xref:mvc/controllers/routing) is case insensitive.</span></span>
* <span data-ttu-id="53ab8-341">Se l'attributo `[HttpGet]` ha un modello di route, ad esempio `[HttpGet("products")]`, aggiungerlo al percorso.</span><span class="sxs-lookup"><span data-stu-id="53ab8-341">If the `[HttpGet]` attribute has a route template (for example, `[HttpGet("products")]`), append that to the path.</span></span> <span data-ttu-id="53ab8-342">In questo esempio non si usa un modello.</span><span class="sxs-lookup"><span data-stu-id="53ab8-342">This sample doesn't use a template.</span></span> <span data-ttu-id="53ab8-343">Per altre informazioni, vedere [Routing con attributi Http[verb]](xref:mvc/controllers/routing#attribute-routing-with-httpverb-attributes).</span><span class="sxs-lookup"><span data-stu-id="53ab8-343">For more information, see [Attribute routing with Http[Verb] attributes](xref:mvc/controllers/routing#attribute-routing-with-httpverb-attributes).</span></span>

<span data-ttu-id="53ab8-344">Nel metodo `GetTodoItem` seguente, `"{id}"` è una variabile segnaposto per l'identificatore univoco dell'elemento attività.</span><span class="sxs-lookup"><span data-stu-id="53ab8-344">In the following `GetTodoItem` method, `"{id}"` is a placeholder variable for the unique identifier of the to-do item.</span></span> <span data-ttu-id="53ab8-345">Quando `GetTodoItem` viene richiamato, il valore di `"{id}"` nell'URL viene fornito al metodo nel `id` parametro.</span><span class="sxs-lookup"><span data-stu-id="53ab8-345">When `GetTodoItem` is invoked, the value of `"{id}"` in the URL is provided to the method in its `id` parameter.</span></span>

[!code-csharp[](first-web-api/samples/3.0/TodoApi/Controllers/TodoItemsController.cs?name=snippet_GetByID&highlight=1-2)]

## <a name="return-values"></a><span data-ttu-id="53ab8-346">Valori restituiti</span><span class="sxs-lookup"><span data-stu-id="53ab8-346">Return values</span></span>

<span data-ttu-id="53ab8-347">Il tipo restituito dei `GetTodoItems` metodi e `GetTodoItem` è [ActionResult. \<T> ](xref:web-api/action-return-types#actionresultt-type)</span><span class="sxs-lookup"><span data-stu-id="53ab8-347">The return type of the `GetTodoItems` and `GetTodoItem` methods is [ActionResult\<T> type](xref:web-api/action-return-types#actionresultt-type).</span></span> <span data-ttu-id="53ab8-348">ASP.NET Core serializza automaticamente l'oggetto su [JSON](https://www.json.org/) e scrive il codice JSON nel corpo del messaggio di risposta.</span><span class="sxs-lookup"><span data-stu-id="53ab8-348">ASP.NET Core automatically serializes the object to [JSON](https://www.json.org/) and writes the JSON into the body of the response message.</span></span> <span data-ttu-id="53ab8-349">Il codice di risposta per questo tipo restituito è [200 OK](https://developer.mozilla.org/docs/Web/HTTP/Status/200), supponendo che non esistano eccezioni non gestite.</span><span class="sxs-lookup"><span data-stu-id="53ab8-349">The response code for this return type is [200 OK](https://developer.mozilla.org/docs/Web/HTTP/Status/200), assuming there are no unhandled exceptions.</span></span> <span data-ttu-id="53ab8-350">Le eccezioni non gestite vengono convertite in errori 5xx.</span><span class="sxs-lookup"><span data-stu-id="53ab8-350">Unhandled exceptions are translated into 5xx errors.</span></span>

<span data-ttu-id="53ab8-351">I tipi restituiti `ActionResult` possono rappresentare un ampio intervallo di codici di stato HTTP.</span><span class="sxs-lookup"><span data-stu-id="53ab8-351">`ActionResult` return types can represent a wide range of HTTP status codes.</span></span> <span data-ttu-id="53ab8-352">Ad esempio, `GetTodoItem` può restituire due valori di stato diversi:</span><span class="sxs-lookup"><span data-stu-id="53ab8-352">For example, `GetTodoItem` can return two different status values:</span></span>

* <span data-ttu-id="53ab8-353">Se nessun elemento corrisponde all'ID richiesto, il metodo restituisce un codice di errore di [stato 404](https://developer.mozilla.org/docs/Web/HTTP/Status/404) <xref:Microsoft.AspNetCore.Mvc.ControllerBase.NotFound%2A> .</span><span class="sxs-lookup"><span data-stu-id="53ab8-353">If no item matches the requested ID, the method returns a [404 status](https://developer.mozilla.org/docs/Web/HTTP/Status/404) <xref:Microsoft.AspNetCore.Mvc.ControllerBase.NotFound%2A> error code.</span></span>
* <span data-ttu-id="53ab8-354">In caso contrario, il metodo restituisce il codice 200 con un corpo della risposta JSON.</span><span class="sxs-lookup"><span data-stu-id="53ab8-354">Otherwise, the method returns 200 with a JSON response body.</span></span> <span data-ttu-id="53ab8-355">La restituzione di `item` risulta in una risposta HTTP 200.</span><span class="sxs-lookup"><span data-stu-id="53ab8-355">Returning `item` results in an HTTP 200 response.</span></span>

## <a name="the-puttodoitem-method"></a><span data-ttu-id="53ab8-356">Metodo PutTodoItem</span><span class="sxs-lookup"><span data-stu-id="53ab8-356">The PutTodoItem method</span></span>

<span data-ttu-id="53ab8-357">Esaminare il metodo `PutTodoItem`:</span><span class="sxs-lookup"><span data-stu-id="53ab8-357">Examine the `PutTodoItem` method:</span></span>

[!code-csharp[](first-web-api/samples/5.x/TodoApi/Controllers/TodoItemsController.cs?name=snippet_Update)]

<span data-ttu-id="53ab8-358">`PutTodoItem` è simile a `PostTodoItem` ma usa la richiesta HTTP PUT.</span><span class="sxs-lookup"><span data-stu-id="53ab8-358">`PutTodoItem` is similar to `PostTodoItem`, except it uses HTTP PUT.</span></span> <span data-ttu-id="53ab8-359">La risposta è [204 (No Content)](https://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html).</span><span class="sxs-lookup"><span data-stu-id="53ab8-359">The response is [204 (No Content)](https://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html).</span></span> <span data-ttu-id="53ab8-360">In base alla specifica HTTP, una richiesta PUT richiede che il client invii l'intera entità aggiornata e non solo le modifiche.</span><span class="sxs-lookup"><span data-stu-id="53ab8-360">According to the HTTP specification, a PUT request requires the client to send the entire updated entity, not just the changes.</span></span> <span data-ttu-id="53ab8-361">Per supportare gli aggiornamenti parziali, usare [HTTP PATCH](xref:Microsoft.AspNetCore.Mvc.HttpPatchAttribute).</span><span class="sxs-lookup"><span data-stu-id="53ab8-361">To support partial updates, use [HTTP PATCH](xref:Microsoft.AspNetCore.Mvc.HttpPatchAttribute).</span></span>

<span data-ttu-id="53ab8-362">Se si riceve un errore chiamando `PutTodoItem`, chiamare `GET` per verificare che il database includa un elemento.</span><span class="sxs-lookup"><span data-stu-id="53ab8-362">If you get an error calling `PutTodoItem`, call `GET` to ensure there's an item in the database.</span></span>

### <a name="test-the-puttodoitem-method"></a><span data-ttu-id="53ab8-363">Testare il metodo PutTodoItem</span><span class="sxs-lookup"><span data-stu-id="53ab8-363">Test the PutTodoItem method</span></span>

<span data-ttu-id="53ab8-364">Questo esempio usa un database in memoria che deve essere inizializzato ogni volta che l'app viene avviata.</span><span class="sxs-lookup"><span data-stu-id="53ab8-364">This sample uses an in-memory database that must be initialized each time the app is started.</span></span> <span data-ttu-id="53ab8-365">Deve esistere un elemento nel database prima di eseguire una chiamata PUT.</span><span class="sxs-lookup"><span data-stu-id="53ab8-365">There must be an item in the database before you make a PUT call.</span></span> <span data-ttu-id="53ab8-366">Chiamare GET per assicurarsi che sia presente un elemento nel database prima di effettuare una chiamata PUT.</span><span class="sxs-lookup"><span data-stu-id="53ab8-366">Call GET to ensure there's an item in the database before making a PUT call.</span></span>

<span data-ttu-id="53ab8-367">Aggiornare l'elemento to-do con ID = 1 e impostarne il nome su `"feed fish"` :</span><span class="sxs-lookup"><span data-stu-id="53ab8-367">Update the to-do item that has Id = 1 and set its name to `"feed fish"`:</span></span>

```json
  {
    "Id":1,
    "name":"feed fish",
    "isComplete":true
  }
```

<span data-ttu-id="53ab8-368">L'immagine seguente visualizza l'aggiornamento di Postman:</span><span class="sxs-lookup"><span data-stu-id="53ab8-368">The following image shows the Postman update:</span></span>

![Console Postman con risposta 204 (No Content)](first-web-api/_static/3/pmcput.png)

## <a name="the-deletetodoitem-method"></a><span data-ttu-id="53ab8-370">Metodo DeleteTodoItem</span><span class="sxs-lookup"><span data-stu-id="53ab8-370">The DeleteTodoItem method</span></span>

<span data-ttu-id="53ab8-371">Esaminare il metodo `DeleteTodoItem`:</span><span class="sxs-lookup"><span data-stu-id="53ab8-371">Examine the `DeleteTodoItem` method:</span></span>

[!code-csharp[](first-web-api/samples/5.x/TodoApi/Controllers/TodoItemsController.cs?name=snippet_Delete)]

### <a name="test-the-deletetodoitem-method"></a><span data-ttu-id="53ab8-372">Testare il metodo DeleteTodoItem</span><span class="sxs-lookup"><span data-stu-id="53ab8-372">Test the DeleteTodoItem method</span></span>

<span data-ttu-id="53ab8-373">Usare Postman per eliminare un elemento attività:</span><span class="sxs-lookup"><span data-stu-id="53ab8-373">Use Postman to delete a to-do item:</span></span>

* <span data-ttu-id="53ab8-374">Impostare il metodo su `DELETE`.</span><span class="sxs-lookup"><span data-stu-id="53ab8-374">Set the method to `DELETE`.</span></span>
* <span data-ttu-id="53ab8-375">Impostare l'URI dell'oggetto da eliminare (ad esempio `https://localhost:5001/api/TodoItems/1` ).</span><span class="sxs-lookup"><span data-stu-id="53ab8-375">Set the URI of the object to delete (for example `https://localhost:5001/api/TodoItems/1`).</span></span>
* <span data-ttu-id="53ab8-376">Selezionare **Invia**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-376">Select **Send**.</span></span>

<a name="over-post-v5"></a>

## <a name="prevent-over-posting"></a><span data-ttu-id="53ab8-377">Impedisci overposting</span><span class="sxs-lookup"><span data-stu-id="53ab8-377">Prevent over-posting</span></span>

<span data-ttu-id="53ab8-378">Attualmente l'app di esempio espone l'intero `TodoItem` oggetto.</span><span class="sxs-lookup"><span data-stu-id="53ab8-378">Currently the sample app exposes the entire `TodoItem` object.</span></span> <span data-ttu-id="53ab8-379">Le app di produzione limitano in genere i dati di input e restituiti usando un subset del modello.</span><span class="sxs-lookup"><span data-stu-id="53ab8-379">Production apps typically limit the data that's input and returned using a subset of the model.</span></span> <span data-ttu-id="53ab8-380">Esistono diversi motivi alla base di questo e la sicurezza è una delle principali.</span><span class="sxs-lookup"><span data-stu-id="53ab8-380">There are multiple reasons behind this and security is a major one.</span></span> <span data-ttu-id="53ab8-381">Il subset di un modello è in genere indicato come oggetto Trasferimento dati (DTO), modello di input o modello di visualizzazione.</span><span class="sxs-lookup"><span data-stu-id="53ab8-381">The subset of a model is usually referred to as a Data Transfer Object (DTO), input model, or view model.</span></span> <span data-ttu-id="53ab8-382">**Dto** viene usato in questo articolo.</span><span class="sxs-lookup"><span data-stu-id="53ab8-382">**DTO** is used in this article.</span></span>

<span data-ttu-id="53ab8-383">Un DTO può essere usato per:</span><span class="sxs-lookup"><span data-stu-id="53ab8-383">A DTO may be used to:</span></span>

* <span data-ttu-id="53ab8-384">Impedisci l'overposting.</span><span class="sxs-lookup"><span data-stu-id="53ab8-384">Prevent over-posting.</span></span>
* <span data-ttu-id="53ab8-385">Nascondere le proprietà che i client non dovrebbero visualizzare.</span><span class="sxs-lookup"><span data-stu-id="53ab8-385">Hide properties that clients are not supposed to view.</span></span>
* <span data-ttu-id="53ab8-386">Omettere alcune proprietà per ridurre le dimensioni del payload.</span><span class="sxs-lookup"><span data-stu-id="53ab8-386">Omit some properties in order to reduce payload size.</span></span>
* <span data-ttu-id="53ab8-387">Rendere flat gli oggetti grafici che contengono oggetti annidati.</span><span class="sxs-lookup"><span data-stu-id="53ab8-387">Flatten object graphs that contain nested objects.</span></span> <span data-ttu-id="53ab8-388">I grafici a oggetti flat possono essere più pratici per i client.</span><span class="sxs-lookup"><span data-stu-id="53ab8-388">Flattened object graphs can be more convenient for clients.</span></span>

<span data-ttu-id="53ab8-389">Per illustrare l'approccio DTO, aggiornare la `TodoItem` classe in modo da includere un campo Secret:</span><span class="sxs-lookup"><span data-stu-id="53ab8-389">To demonstrate the DTO approach, update the `TodoItem` class to include a secret field:</span></span>

[!code-csharp[](first-web-api/samples/5.x/TodoApiDTO/Models/TodoItem.cs?name=snippet&highlight=8)]

<span data-ttu-id="53ab8-390">Il campo segreto deve essere nascosto da questa app, ma un'app amministrativa può scegliere di esporla.</span><span class="sxs-lookup"><span data-stu-id="53ab8-390">The secret field needs to be hidden from this app, but an administrative app could choose to expose it.</span></span>

<span data-ttu-id="53ab8-391">Verificare che sia possibile pubblicare e ottenere il campo secret.</span><span class="sxs-lookup"><span data-stu-id="53ab8-391">Verify you can post and get the secret field.</span></span>

<span data-ttu-id="53ab8-392">Creare un modello DTO:</span><span class="sxs-lookup"><span data-stu-id="53ab8-392">Create a DTO model:</span></span>

[!code-csharp[](first-web-api/samples/5.x/TodoApiDTO/Models/TodoItemDTO.cs?name=snippet)]

<span data-ttu-id="53ab8-393">Aggiornare `TodoItemsController` da usare `TodoItemDTO` :</span><span class="sxs-lookup"><span data-stu-id="53ab8-393">Update the `TodoItemsController` to use `TodoItemDTO`:</span></span>

[!code-csharp[](first-web-api/samples/5.x/TodoApiDTO/Controllers/TodoItemsController.cs?name=snippet)]

<span data-ttu-id="53ab8-394">Verificare che non sia possibile inserire o ottenere il campo secret.</span><span class="sxs-lookup"><span data-stu-id="53ab8-394">Verify you can't post or get the secret field.</span></span>

## <a name="call-the-web-api-with-javascript"></a><span data-ttu-id="53ab8-395">Chiamare l'API Web con JavaScript</span><span class="sxs-lookup"><span data-stu-id="53ab8-395">Call the web API with JavaScript</span></span>

<span data-ttu-id="53ab8-396">Vedere [esercitazione: chiamare un'API web ASP.NET Core con JavaScript](xref:tutorials/web-api-javascript).</span><span class="sxs-lookup"><span data-stu-id="53ab8-396">See [Tutorial: Call an ASP.NET Core web API with JavaScript](xref:tutorials/web-api-javascript).</span></span>

::: moniker-end

::: moniker range=">= aspnetcore-3.0 < aspnetcore-5.0"

<span data-ttu-id="53ab8-397">In questa esercitazione verranno illustrate le procedure per:</span><span class="sxs-lookup"><span data-stu-id="53ab8-397">In this tutorial, you learn how to:</span></span>

> [!div class="checklist"]
> * <span data-ttu-id="53ab8-398">Creare un progetto di API Web.</span><span class="sxs-lookup"><span data-stu-id="53ab8-398">Create a web API project.</span></span>
> * <span data-ttu-id="53ab8-399">Aggiungere una classe modello e un contesto di database.</span><span class="sxs-lookup"><span data-stu-id="53ab8-399">Add a model class and a database context.</span></span>
> * <span data-ttu-id="53ab8-400">Eseguire lo scaffolding di un controller con i metodi CRUD.</span><span class="sxs-lookup"><span data-stu-id="53ab8-400">Scaffold a controller with CRUD methods.</span></span>
> * <span data-ttu-id="53ab8-401">Configurare il routing, i percorsi URL e i valori restituiti.</span><span class="sxs-lookup"><span data-stu-id="53ab8-401">Configure routing, URL paths, and return values.</span></span>
> * <span data-ttu-id="53ab8-402">Chiamare l'API Web con Postman.</span><span class="sxs-lookup"><span data-stu-id="53ab8-402">Call the web API with Postman.</span></span>

<span data-ttu-id="53ab8-403">Al termine si avrà un'API Web che può gestire gli elementi di tipo "attività" archiviati in un database.</span><span class="sxs-lookup"><span data-stu-id="53ab8-403">At the end, you have a web API that can manage "to-do" items stored in a database.</span></span>

## <a name="overview"></a><span data-ttu-id="53ab8-404">Panoramica</span><span class="sxs-lookup"><span data-stu-id="53ab8-404">Overview</span></span>

<span data-ttu-id="53ab8-405">Questa esercitazione consente di creare l'API seguente:</span><span class="sxs-lookup"><span data-stu-id="53ab8-405">This tutorial creates the following API:</span></span>

|<span data-ttu-id="53ab8-406">API</span><span class="sxs-lookup"><span data-stu-id="53ab8-406">API</span></span> | <span data-ttu-id="53ab8-407">Descrizione</span><span class="sxs-lookup"><span data-stu-id="53ab8-407">Description</span></span> | <span data-ttu-id="53ab8-408">Testo della richiesta</span><span class="sxs-lookup"><span data-stu-id="53ab8-408">Request body</span></span> | <span data-ttu-id="53ab8-409">Corpo della risposta</span><span class="sxs-lookup"><span data-stu-id="53ab8-409">Response body</span></span> |
|--- | ---- | ---- | ---- |
|`GET /api/TodoItems` | <span data-ttu-id="53ab8-410">Ottiene tutti gli elementi attività</span><span class="sxs-lookup"><span data-stu-id="53ab8-410">Get all to-do items</span></span> | <span data-ttu-id="53ab8-411">nessuno</span><span class="sxs-lookup"><span data-stu-id="53ab8-411">None</span></span> | <span data-ttu-id="53ab8-412">Matrice di elementi attività</span><span class="sxs-lookup"><span data-stu-id="53ab8-412">Array of to-do items</span></span>|
|`GET /api/TodoItems/{id}` | <span data-ttu-id="53ab8-413">Ottiene un elemento in base all'ID</span><span class="sxs-lookup"><span data-stu-id="53ab8-413">Get an item by ID</span></span> | <span data-ttu-id="53ab8-414">nessuno</span><span class="sxs-lookup"><span data-stu-id="53ab8-414">None</span></span> | <span data-ttu-id="53ab8-415">Elemento attività</span><span class="sxs-lookup"><span data-stu-id="53ab8-415">To-do item</span></span>|
|`POST /api/TodoItems` | <span data-ttu-id="53ab8-416">Aggiunge un nuovo elemento</span><span class="sxs-lookup"><span data-stu-id="53ab8-416">Add a new item</span></span> | <span data-ttu-id="53ab8-417">Elemento attività</span><span class="sxs-lookup"><span data-stu-id="53ab8-417">To-do item</span></span> | <span data-ttu-id="53ab8-418">Elemento attività</span><span class="sxs-lookup"><span data-stu-id="53ab8-418">To-do item</span></span> |
|`PUT /api/TodoItems/{id}` | <span data-ttu-id="53ab8-419">Aggiorna un elemento esistente &nbsp;</span><span class="sxs-lookup"><span data-stu-id="53ab8-419">Update an existing item &nbsp;</span></span> | <span data-ttu-id="53ab8-420">Elemento attività</span><span class="sxs-lookup"><span data-stu-id="53ab8-420">To-do item</span></span> | <span data-ttu-id="53ab8-421">nessuno</span><span class="sxs-lookup"><span data-stu-id="53ab8-421">None</span></span> |
|<span data-ttu-id="53ab8-422">`DELETE /api/TodoItems/{id}` &nbsp; &nbsp;</span><span class="sxs-lookup"><span data-stu-id="53ab8-422">`DELETE /api/TodoItems/{id}` &nbsp; &nbsp;</span></span> | <span data-ttu-id="53ab8-423">Elimina un elemento &nbsp;&nbsp;</span><span class="sxs-lookup"><span data-stu-id="53ab8-423">Delete an item &nbsp; &nbsp;</span></span> | <span data-ttu-id="53ab8-424">nessuno</span><span class="sxs-lookup"><span data-stu-id="53ab8-424">None</span></span> | <span data-ttu-id="53ab8-425">nessuno</span><span class="sxs-lookup"><span data-stu-id="53ab8-425">None</span></span>|

<span data-ttu-id="53ab8-426">Il diagramma seguente visualizza la struttura dell'app.</span><span class="sxs-lookup"><span data-stu-id="53ab8-426">The following diagram shows the design of the app.</span></span>

![Il client è rappresentato da una casella a sinistra.](first-web-api/_static/architecture.png)

## <a name="prerequisites"></a><span data-ttu-id="53ab8-432">Prerequisiti</span><span class="sxs-lookup"><span data-stu-id="53ab8-432">Prerequisites</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="53ab8-433">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="53ab8-433">Visual Studio</span></span>](#tab/visual-studio)

[!INCLUDE[](~/includes/net-core-prereqs-vs-3.1.md)]

# <a name="visual-studio-code"></a>[<span data-ttu-id="53ab8-434">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="53ab8-434">Visual Studio Code</span></span>](#tab/visual-studio-code)

[!INCLUDE[](~/includes/net-core-prereqs-vsc-3.1.md)]

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="53ab8-435">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="53ab8-435">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

[!INCLUDE[](~/includes/net-core-prereqs-mac-3.1.md)]

---

## <a name="create-a-web-project"></a><span data-ttu-id="53ab8-436">Creare un progetto Web</span><span class="sxs-lookup"><span data-stu-id="53ab8-436">Create a web project</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="53ab8-437">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="53ab8-437">Visual Studio</span></span>](#tab/visual-studio)

* <span data-ttu-id="53ab8-438">Scegliere **nuovo** progetto dal menu **file** > .</span><span class="sxs-lookup"><span data-stu-id="53ab8-438">From the **File** menu, select **New** > **Project**.</span></span>
* <span data-ttu-id="53ab8-439">Selezionare il modello **Applicazione Web ASP.NET Core** e fare clic su **Avanti**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-439">Select the **ASP.NET Core Web Application** template and click **Next**.</span></span>
* <span data-ttu-id="53ab8-440">Assegnare al progetto il nome *TodoApi* e fare clic su **Crea**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-440">Name the project *TodoApi* and click **Create**.</span></span>
* <span data-ttu-id="53ab8-441">Nella finestra di dialogo **Crea una nuova applicazione Web ASP.NET Core** verificare che siano selezionati **.net Core** e **ASP.NET Core 3,1** .</span><span class="sxs-lookup"><span data-stu-id="53ab8-441">In the **Create a new ASP.NET Core Web Application** dialog, confirm that **.NET Core** and **ASP.NET Core 3.1** are selected.</span></span> <span data-ttu-id="53ab8-442">Selezionare il modello **API** e fare clic su **Crea**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-442">Select the **API** template and click **Create**.</span></span>

![Finestra di dialogo Nuovo progetto di Visual Studio](first-web-api/_static/vs3.png)

# <a name="visual-studio-code"></a>[<span data-ttu-id="53ab8-444">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="53ab8-444">Visual Studio Code</span></span>](#tab/visual-studio-code)

* <span data-ttu-id="53ab8-445">Aprire il [terminale integrato](https://code.visualstudio.com/docs/editor/integrated-terminal).</span><span class="sxs-lookup"><span data-stu-id="53ab8-445">Open the [integrated terminal](https://code.visualstudio.com/docs/editor/integrated-terminal).</span></span>
* <span data-ttu-id="53ab8-446">Cambiare directory (`cd`) e passare alla cartella che conterrà la cartella del progetto.</span><span class="sxs-lookup"><span data-stu-id="53ab8-446">Change directories (`cd`) to the folder that will contain the project folder.</span></span>
* <span data-ttu-id="53ab8-447">Eseguire i comandi seguenti:</span><span class="sxs-lookup"><span data-stu-id="53ab8-447">Run the following commands:</span></span>

   ```dotnetcli
   dotnet new webapi -o TodoApi
   cd TodoApi
   dotnet add package Microsoft.EntityFrameworkCore.InMemory
   code -r ../TodoApi
   ```

* <span data-ttu-id="53ab8-448">Quando una finestra di dialogo chiede se si vuole aggiungere gli asset necessari al progetto, selezionare **Sì**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-448">When a dialog box asks if you want to add required assets to the project, select **Yes**.</span></span>

  <span data-ttu-id="53ab8-449">I comandi precedenti:</span><span class="sxs-lookup"><span data-stu-id="53ab8-449">The preceding commands:</span></span>

  * <span data-ttu-id="53ab8-450">Crea un nuovo progetto API Web e lo apre in Visual Studio Code.</span><span class="sxs-lookup"><span data-stu-id="53ab8-450">Creates a new web API project and opens it in Visual Studio Code.</span></span>
  * <span data-ttu-id="53ab8-451">Aggiunge i pacchetti NuGet necessari nella sezione successiva.</span><span class="sxs-lookup"><span data-stu-id="53ab8-451">Adds the NuGet packages which are required in the next section.</span></span>

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="53ab8-452">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="53ab8-452">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

* <span data-ttu-id="53ab8-453">Selezionare **File** > **Nuova soluzione**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-453">Select **File** > **New Solution**.</span></span>

  ![Nuova soluzione macOS](first-web-api-mac/_static/sln.png)

* <span data-ttu-id="53ab8-455">In Visual Studio per Mac precedente alla versione 8,6 Selezionare API **app .NET Core**  >    >    >  **Avanti**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-455">In Visual Studio for Mac earlier than version 8.6, select **.NET Core** > **App** > **API** > **Next**.</span></span> <span data-ttu-id="53ab8-456">Nella versione 8,6 o successive selezionare API **app Web e console**  >    >    >  **Avanti**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-456">In version 8.6 or later, select **Web and Console** > **App** > **API** > **Next**.</span></span>

  ![Selezione modello API macOS](first-web-api-mac/_static/api_template.png)

* <span data-ttu-id="53ab8-458">Nella finestra di dialogo **Configura nuova ASP.NET Core API Web** selezionare il **Framework di destinazione**.NET Core 3. x più recente.</span><span class="sxs-lookup"><span data-stu-id="53ab8-458">In the **Configure the new ASP.NET Core Web API** dialog, select the latest .NET Core 3.x **Target Framework**.</span></span> <span data-ttu-id="53ab8-459">Selezionare **Avanti**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-459">Select **Next**.</span></span>

* <span data-ttu-id="53ab8-460">Immettere *TodoApi* in **Nome progetto**, quindi selezionare **Crea**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-460">Enter *TodoApi* for the **Project Name** and then select **Create**.</span></span>

  ![Finestra di dialogo di configurazione](first-web-api-mac/_static/2.png)

[!INCLUDE[](~/includes/mac-terminal-access.md)]

<span data-ttu-id="53ab8-462">Aprire un terminale di comando nella cartella del progetto ed eseguire il comando seguente:</span><span class="sxs-lookup"><span data-stu-id="53ab8-462">Open a command terminal in the project folder and run the following command:</span></span>

   ```dotnetcli
   dotnet add package Microsoft.EntityFrameworkCore.InMemory
   ```

---

### <a name="test-the-api"></a><span data-ttu-id="53ab8-463">Testare l'API</span><span class="sxs-lookup"><span data-stu-id="53ab8-463">Test the API</span></span>

<span data-ttu-id="53ab8-464">Il modello di progetto crea un'API `WeatherForecast`.</span><span class="sxs-lookup"><span data-stu-id="53ab8-464">The project template creates a `WeatherForecast` API.</span></span> <span data-ttu-id="53ab8-465">Chiamare il metodo `Get` da un browser per eseguire il test dell'app.</span><span class="sxs-lookup"><span data-stu-id="53ab8-465">Call the `Get` method from a browser to test the app.</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="53ab8-466">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="53ab8-466">Visual Studio</span></span>](#tab/visual-studio)

<span data-ttu-id="53ab8-467">Premere CTRL+F5 per eseguire l'app.</span><span class="sxs-lookup"><span data-stu-id="53ab8-467">Press Ctrl+F5 to run the app.</span></span> <span data-ttu-id="53ab8-468">Visual Studio apre un browser e naviga all'indirizzo `https://localhost:<port>/WeatherForecast`, dove `<port>` è un numero di porta selezionato a caso.</span><span class="sxs-lookup"><span data-stu-id="53ab8-468">Visual Studio launches a browser and navigates to `https://localhost:<port>/WeatherForecast`, where `<port>` is a randomly chosen port number.</span></span>

<span data-ttu-id="53ab8-469">Se in una finestra di dialogo viene chiesto se considerare attendibile il certificato IIS Express, selezionare **Sì**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-469">If you get a dialog box that asks if you should trust the IIS Express certificate, select **Yes**.</span></span> <span data-ttu-id="53ab8-470">Nella finestra di dialogo **Avviso di sicurezza** successiva selezionare **Sì**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-470">In the **Security Warning** dialog that appears next, select **Yes**.</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="53ab8-471">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="53ab8-471">Visual Studio Code</span></span>](#tab/visual-studio-code)

<span data-ttu-id="53ab8-472">Premere CTRL+F5 per eseguire l'app.</span><span class="sxs-lookup"><span data-stu-id="53ab8-472">Press Ctrl+F5 to run the app.</span></span> <span data-ttu-id="53ab8-473">In un browser passare all'URL seguente: `https://localhost:5001/WeatherForecast`.</span><span class="sxs-lookup"><span data-stu-id="53ab8-473">In a browser, go to following URL: `https://localhost:5001/WeatherForecast`.</span></span>

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="53ab8-474">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="53ab8-474">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

<span data-ttu-id="53ab8-475">Selezionare **Esegui**  >  **Avvia debug** per avviare l'app.</span><span class="sxs-lookup"><span data-stu-id="53ab8-475">Select **Run** > **Start Debugging** to launch the app.</span></span> <span data-ttu-id="53ab8-476">Visual Studio per Mac apre un browser e naviga all'indirizzo `https://localhost:<port>`, dove `<port>` è un numero di porta selezionato a caso.</span><span class="sxs-lookup"><span data-stu-id="53ab8-476">Visual Studio for Mac launches a browser and navigates to `https://localhost:<port>`, where `<port>` is a randomly chosen port number.</span></span> <span data-ttu-id="53ab8-477">Viene restituito un errore HTTP 404 (Non trovato).</span><span class="sxs-lookup"><span data-stu-id="53ab8-477">An HTTP 404 (Not Found) error is returned.</span></span> <span data-ttu-id="53ab8-478">Accodare `/WeatherForecast` all'URL (impostare l'URL su `https://localhost:<port>/WeatherForecast`).</span><span class="sxs-lookup"><span data-stu-id="53ab8-478">Append `/WeatherForecast` to the URL (change the URL to `https://localhost:<port>/WeatherForecast`).</span></span>

---

<span data-ttu-id="53ab8-479">Viene restituito un codice JSON simile al seguente:</span><span class="sxs-lookup"><span data-stu-id="53ab8-479">JSON similar to the following is returned:</span></span>

```json
[
    {
        "date": "2019-07-16T19:04:05.7257911-06:00",
        "temperatureC": 52,
        "temperatureF": 125,
        "summary": "Mild"
    },
    {
        "date": "2019-07-17T19:04:05.7258461-06:00",
        "temperatureC": 36,
        "temperatureF": 96,
        "summary": "Warm"
    },
    {
        "date": "2019-07-18T19:04:05.7258467-06:00",
        "temperatureC": 39,
        "temperatureF": 102,
        "summary": "Cool"
    },
    {
        "date": "2019-07-19T19:04:05.7258471-06:00",
        "temperatureC": 10,
        "temperatureF": 49,
        "summary": "Bracing"
    },
    {
        "date": "2019-07-20T19:04:05.7258474-06:00",
        "temperatureC": -1,
        "temperatureF": 31,
        "summary": "Chilly"
    }
]
```

## <a name="add-a-model-class"></a><span data-ttu-id="53ab8-480">Aggiungere una classe del modello</span><span class="sxs-lookup"><span data-stu-id="53ab8-480">Add a model class</span></span>

<span data-ttu-id="53ab8-481">Un *modello* è un set di classi che rappresentano i dati gestiti dall'app.</span><span class="sxs-lookup"><span data-stu-id="53ab8-481">A *model* is a set of classes that represent the data that the app manages.</span></span> <span data-ttu-id="53ab8-482">Il modello per questa app è una classe `TodoItem` singola.</span><span class="sxs-lookup"><span data-stu-id="53ab8-482">The model for this app is a single `TodoItem` class.</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="53ab8-483">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="53ab8-483">Visual Studio</span></span>](#tab/visual-studio)

* <span data-ttu-id="53ab8-484">In **Esplora soluzioni** fare clic con il pulsante destro del mouse sul progetto.</span><span class="sxs-lookup"><span data-stu-id="53ab8-484">In **Solution Explorer**, right-click the project.</span></span> <span data-ttu-id="53ab8-485">Selezionare **Aggiungi**  >  **nuova cartella**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-485">Select **Add** > **New Folder**.</span></span> <span data-ttu-id="53ab8-486">Assegnare un nome alla cartella *Models* .</span><span class="sxs-lookup"><span data-stu-id="53ab8-486">Name the folder *Models*.</span></span>

* <span data-ttu-id="53ab8-487">Fare clic con il pulsante destro del mouse sulla *Models* cartella e scegliere **Aggiungi**  >  **classe**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-487">Right-click the *Models* folder and select **Add** > **Class**.</span></span> <span data-ttu-id="53ab8-488">Assegnare alla classe il nome *TodoItem* e selezionare **Aggiungi**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-488">Name the class *TodoItem* and select **Add**.</span></span>

* <span data-ttu-id="53ab8-489">Sostituire il codice del modello con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="53ab8-489">Replace the template code with the following code:</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="53ab8-490">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="53ab8-490">Visual Studio Code</span></span>](#tab/visual-studio-code)

* <span data-ttu-id="53ab8-491">Aggiungere una cartella denominata *Models* .</span><span class="sxs-lookup"><span data-stu-id="53ab8-491">Add a folder named *Models*.</span></span>

* <span data-ttu-id="53ab8-492">Aggiungere una `TodoItem` classe alla *Models* cartella con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="53ab8-492">Add a `TodoItem` class to the *Models* folder with the following code:</span></span>

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="53ab8-493">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="53ab8-493">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

* <span data-ttu-id="53ab8-494">Fare clic con il pulsante destro del mouse sul progetto.</span><span class="sxs-lookup"><span data-stu-id="53ab8-494">Right-click the project.</span></span> <span data-ttu-id="53ab8-495">Selezionare **Aggiungi**  >  **nuova cartella**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-495">Select **Add** > **New Folder**.</span></span> <span data-ttu-id="53ab8-496">Assegnare un nome alla cartella *Models* .</span><span class="sxs-lookup"><span data-stu-id="53ab8-496">Name the folder *Models*.</span></span>

  ![Nuova cartella](first-web-api-mac/_static/folder.png)

* <span data-ttu-id="53ab8-498">Fare clic con il pulsante destro del mouse sulla *Models* cartella e scegliere **Aggiungi** > **nuovo file** > **generale** > **vuota classe**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-498">Right-click the *Models* folder, and select **Add** > **New File** > **General** > **Empty Class**.</span></span>

* <span data-ttu-id="53ab8-499">Assegnare alla classe il nome *TodoItem* e fare clic su **Nuovo**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-499">Name the class *TodoItem*, and then click **New**.</span></span>

* <span data-ttu-id="53ab8-500">Sostituire il codice del modello con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="53ab8-500">Replace the template code with the following code:</span></span>

---

  [!code-csharp[](first-web-api/samples/3.0/TodoApi/Models/TodoItem.cs?name=snippet)]

<span data-ttu-id="53ab8-501">La proprietà `Id` funziona come chiave univoca in un database relazionale.</span><span class="sxs-lookup"><span data-stu-id="53ab8-501">The `Id` property functions as the unique key in a relational database.</span></span>

<span data-ttu-id="53ab8-502">Le classi di modelli possono essere inserite in qualsiasi punto del progetto, ma la *Models* cartella viene utilizzata per convenzione.</span><span class="sxs-lookup"><span data-stu-id="53ab8-502">Model classes can go anywhere in the project, but the *Models* folder is used by convention.</span></span>

## <a name="add-a-database-context"></a><span data-ttu-id="53ab8-503">Aggiungere un contesto di database</span><span class="sxs-lookup"><span data-stu-id="53ab8-503">Add a database context</span></span>

<span data-ttu-id="53ab8-504">Il *contesto di database* è la classe principale che coordina le funzionalità di Entity Framework per un modello di dati.</span><span class="sxs-lookup"><span data-stu-id="53ab8-504">The *database context* is the main class that coordinates Entity Framework functionality for a data model.</span></span> <span data-ttu-id="53ab8-505">Questa classe viene creata mediante derivazione dalla classe `Microsoft.EntityFrameworkCore.DbContext`.</span><span class="sxs-lookup"><span data-stu-id="53ab8-505">This class is created by deriving from the `Microsoft.EntityFrameworkCore.DbContext` class.</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="53ab8-506">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="53ab8-506">Visual Studio</span></span>](#tab/visual-studio)

### <a name="add-nuget-packages"></a><span data-ttu-id="53ab8-507">Aggiungere pacchetti NuGet</span><span class="sxs-lookup"><span data-stu-id="53ab8-507">Add NuGet packages</span></span>

* <span data-ttu-id="53ab8-508">Dal menu **Strumenti** selezionare **Gestione pacchetti NuGet > Gestisci pacchetti NuGet per la soluzione**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-508">From the **Tools** menu, select **NuGet Package Manager > Manage NuGet Packages for Solution**.</span></span>
* <span data-ttu-id="53ab8-509">Selezionare la scheda **Sfoglia** e quindi immettere **Microsoft. EntityFrameworkCore. InMemory** nella casella di ricerca.</span><span class="sxs-lookup"><span data-stu-id="53ab8-509">Select the **Browse** tab, and then enter **Microsoft.EntityFrameworkCore.InMemory** in the search box.</span></span>
* <span data-ttu-id="53ab8-510">Selezionare **Microsoft. EntityFrameworkCore. InMemory** nel riquadro sinistro.</span><span class="sxs-lookup"><span data-stu-id="53ab8-510">Select **Microsoft.EntityFrameworkCore.InMemory** in the left pane.</span></span>
* <span data-ttu-id="53ab8-511">Selezionare la casella di controllo **Progetto** nel riquadro di destra e quindi selezionare **Installa**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-511">Select the **Project** check box in the right pane and then select **Install**.</span></span>

![Gestione pacchetti NuGet](first-web-api/_static/vs3NuGet.png)

## <a name="add-the-todocontext-database-context"></a><span data-ttu-id="53ab8-513">Aggiungere il contesto del database TodoContext</span><span class="sxs-lookup"><span data-stu-id="53ab8-513">Add the TodoContext database context</span></span>

* <span data-ttu-id="53ab8-514">Fare clic con il pulsante destro del mouse sulla *Models* cartella e scegliere **Aggiungi**  >  **classe**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-514">Right-click the *Models* folder and select **Add** > **Class**.</span></span> <span data-ttu-id="53ab8-515">Assegnare alla classe il nome *TodoContext* e fare clic su **Aggiungi**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-515">Name the class *TodoContext* and click **Add**.</span></span>

# <a name="visual-studio-code--visual-studio-for-mac"></a>[<span data-ttu-id="53ab8-516">Visual Studio Code / Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="53ab8-516">Visual Studio Code / Visual Studio for Mac</span></span>](#tab/visual-studio-code+visual-studio-mac)

* <span data-ttu-id="53ab8-517">Aggiungere una `TodoContext` classe alla *Models* cartella.</span><span class="sxs-lookup"><span data-stu-id="53ab8-517">Add a `TodoContext` class to the *Models* folder.</span></span>

---

* <span data-ttu-id="53ab8-518">Immettere il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="53ab8-518">Enter the following code:</span></span>

  [!code-csharp[](first-web-api/samples/3.0/TodoApi/Models/TodoContext.cs)]

## <a name="register-the-database-context"></a><span data-ttu-id="53ab8-519">Registrare il contesto del database</span><span class="sxs-lookup"><span data-stu-id="53ab8-519">Register the database context</span></span>

<span data-ttu-id="53ab8-520">In ASP.NET Core i servizi come il contesto del database devono essere registrati con il [contenitore di inserimento delle dipendenze](xref:fundamentals/dependency-injection).</span><span class="sxs-lookup"><span data-stu-id="53ab8-520">In ASP.NET Core, services such as the DB context must be registered with the [dependency injection (DI)](xref:fundamentals/dependency-injection) container.</span></span> <span data-ttu-id="53ab8-521">Il contenitore rende disponibile il servizio ai controller.</span><span class="sxs-lookup"><span data-stu-id="53ab8-521">The container provides the service to controllers.</span></span>

<span data-ttu-id="53ab8-522">Aggiornare *Startup.cs* con il codice evidenziato seguente:</span><span class="sxs-lookup"><span data-stu-id="53ab8-522">Update *Startup.cs* with the following highlighted code:</span></span>

[!code-csharp[](first-web-api/samples/3.0/TodoApi/Startup.cs?highlight=7-8,23-24&name=snippet_all)]

<span data-ttu-id="53ab8-523">Il codice precedente:</span><span class="sxs-lookup"><span data-stu-id="53ab8-523">The preceding code:</span></span>

* <span data-ttu-id="53ab8-524">Rimuove le dichiarazioni `using` inutilizzate.</span><span class="sxs-lookup"><span data-stu-id="53ab8-524">Removes unused `using` declarations.</span></span>
* <span data-ttu-id="53ab8-525">Aggiunge il contesto del database al contenitore di inserimento delle dipendenze.</span><span class="sxs-lookup"><span data-stu-id="53ab8-525">Adds the database context to the DI container.</span></span>
* <span data-ttu-id="53ab8-526">Specifica che il contesto del database userà un database in memoria.</span><span class="sxs-lookup"><span data-stu-id="53ab8-526">Specifies that the database context will use an in-memory database.</span></span>

## <a name="scaffold-a-controller"></a><span data-ttu-id="53ab8-527">Eseguire lo scaffolding di un controller</span><span class="sxs-lookup"><span data-stu-id="53ab8-527">Scaffold a controller</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="53ab8-528">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="53ab8-528">Visual Studio</span></span>](#tab/visual-studio)

* <span data-ttu-id="53ab8-529">Fare clic con il pulsante destro del mouse sulla cartella *Controllers*.</span><span class="sxs-lookup"><span data-stu-id="53ab8-529">Right-click the *Controllers* folder.</span></span>
* <span data-ttu-id="53ab8-530">Selezionare **Aggiungi** > **Nuovo elemento di scaffolding**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-530">Select **Add** > **New Scaffolded Item**.</span></span>
* <span data-ttu-id="53ab8-531">Selezionare **Controller API con azioni, che usa Entity Framework** e quindi selezionare **Aggiungi**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-531">Select **API Controller with actions, using Entity Framework**, and then select **Add**.</span></span>
* <span data-ttu-id="53ab8-532">Nella finestra di dialogo **Add API Controller with actions, using Entity Framework** (Aggiungi controller API con azioni, che usa Entity Framework):</span><span class="sxs-lookup"><span data-stu-id="53ab8-532">In the **Add API Controller with actions, using Entity Framework** dialog:</span></span>

  * <span data-ttu-id="53ab8-533">Selezionare **TodoItem (TodoApi. Models )** nella **classe del modello**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-533">Select **TodoItem (TodoApi.Models)** in the **Model class**.</span></span>
  * <span data-ttu-id="53ab8-534">Selezionare **TodoContext (TodoApi. Models )** nella **classe del contesto dati**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-534">Select **TodoContext (TodoApi.Models)** in the **Data context class**.</span></span>
  * <span data-ttu-id="53ab8-535">Selezionare **Aggiungi**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-535">Select **Add**.</span></span>

# <a name="visual-studio-code--visual-studio-for-mac"></a>[<span data-ttu-id="53ab8-536">Visual Studio Code / Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="53ab8-536">Visual Studio Code / Visual Studio for Mac</span></span>](#tab/visual-studio-code+visual-studio-mac)

<span data-ttu-id="53ab8-537">Eseguire i comandi seguenti:</span><span class="sxs-lookup"><span data-stu-id="53ab8-537">Run the following commands:</span></span>

```dotnetcli
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
dotnet add package Microsoft.EntityFrameworkCore.Design
dotnet tool install --global dotnet-aspnet-codegenerator
dotnet tool update -g Dotnet-aspnet-codegenerator
dotnet aspnet-codegenerator controller -name TodoItemsController -async -api -m TodoItem -dc TodoContext -outDir Controllers
```

<span data-ttu-id="53ab8-538">I comandi precedenti:</span><span class="sxs-lookup"><span data-stu-id="53ab8-538">The preceding commands:</span></span>

* <span data-ttu-id="53ab8-539">Aggiungere i pacchetti NuGet necessari per lo scaffolding.</span><span class="sxs-lookup"><span data-stu-id="53ab8-539">Add NuGet packages required for scaffolding.</span></span>
* <span data-ttu-id="53ab8-540">Installa il motore di scaffolding (`dotnet-aspnet-codegenerator`).</span><span class="sxs-lookup"><span data-stu-id="53ab8-540">Installs the scaffolding engine (`dotnet-aspnet-codegenerator`).</span></span>
* <span data-ttu-id="53ab8-541">Esegue lo scaffolding di `TodoItemsController`.</span><span class="sxs-lookup"><span data-stu-id="53ab8-541">Scaffolds the `TodoItemsController`.</span></span>

---

<span data-ttu-id="53ab8-542">Il codice generato:</span><span class="sxs-lookup"><span data-stu-id="53ab8-542">The generated code:</span></span>

* <span data-ttu-id="53ab8-543">Contrassegna la classe con l' [`[ApiController]`](xref:Microsoft.AspNetCore.Mvc.ApiControllerAttribute) attributo.</span><span class="sxs-lookup"><span data-stu-id="53ab8-543">Marks the class with the [`[ApiController]`](xref:Microsoft.AspNetCore.Mvc.ApiControllerAttribute) attribute.</span></span> <span data-ttu-id="53ab8-544">L'attributo indica che il controller risponde alle richieste di API Web.</span><span class="sxs-lookup"><span data-stu-id="53ab8-544">This attribute indicates that the controller responds to web API requests.</span></span> <span data-ttu-id="53ab8-545">Per informazioni sui comportamenti specifici consentiti dall'attributo, vedere <xref:web-api/index>.</span><span class="sxs-lookup"><span data-stu-id="53ab8-545">For information about specific behaviors that the attribute enables, see <xref:web-api/index>.</span></span>
* <span data-ttu-id="53ab8-546">Usa l'inserimento delle dipendenze per inserire il contesto del database (`TodoContext`) nel controller.</span><span class="sxs-lookup"><span data-stu-id="53ab8-546">Uses DI to inject the database context (`TodoContext`) into the controller.</span></span> <span data-ttu-id="53ab8-547">Il contesto di database viene usato in ognuno dei metodi [CRUD](https://wikipedia.org/wiki/Create,_read,_update_and_delete) nel controller.</span><span class="sxs-lookup"><span data-stu-id="53ab8-547">The database context is used in each of the [CRUD](https://wikipedia.org/wiki/Create,_read,_update_and_delete) methods in the controller.</span></span>

<span data-ttu-id="53ab8-548">I modelli di ASP.NET Core per:</span><span class="sxs-lookup"><span data-stu-id="53ab8-548">The ASP.NET Core templates for:</span></span>

* <span data-ttu-id="53ab8-549">I controller con le visualizzazioni includono `[action]` nel modello di route.</span><span class="sxs-lookup"><span data-stu-id="53ab8-549">Controllers with views include `[action]` in the route template.</span></span>
* <span data-ttu-id="53ab8-550">I controller API non includono `[action]` nel modello di route.</span><span class="sxs-lookup"><span data-stu-id="53ab8-550">API controllers don't include `[action]` in the route template.</span></span>

<span data-ttu-id="53ab8-551">Quando il `[action]` token non è incluso nel modello di route, il nome dell' [azione](xref:mvc/controllers/routing#action) viene escluso dalla route.</span><span class="sxs-lookup"><span data-stu-id="53ab8-551">When the `[action]` token isn't in the route template, the [action](xref:mvc/controllers/routing#action) name is excluded from the route.</span></span> <span data-ttu-id="53ab8-552">Ovvero, il nome del metodo associato all'azione non viene usato nella route corrispondente.</span><span class="sxs-lookup"><span data-stu-id="53ab8-552">That is, the action's associated method name isn't used in the matching route.</span></span>

## <a name="examine-the-posttodoitem-create-method"></a><span data-ttu-id="53ab8-553">Esaminare il metodo di creazione PostTodoItem</span><span class="sxs-lookup"><span data-stu-id="53ab8-553">Examine the PostTodoItem create method</span></span>

<span data-ttu-id="53ab8-554">Sostituire l'istruzione return in `PostTodoItem` per usare l'operatore [nameof](/dotnet/csharp/language-reference/operators/nameof):</span><span class="sxs-lookup"><span data-stu-id="53ab8-554">Replace the return statement in the `PostTodoItem` to use the [nameof](/dotnet/csharp/language-reference/operators/nameof) operator:</span></span>

[!code-csharp[](first-web-api/samples/3.0/TodoApi/Controllers/TodoItemsController.cs?name=snippet_Create)]

<span data-ttu-id="53ab8-555">Il codice precedente è un metodo HTTP POST, come indicato dall' [`[HttpPost]`](xref:Microsoft.AspNetCore.Mvc.HttpPostAttribute) attributo.</span><span class="sxs-lookup"><span data-stu-id="53ab8-555">The preceding code is an HTTP POST method, as indicated by the [`[HttpPost]`](xref:Microsoft.AspNetCore.Mvc.HttpPostAttribute) attribute.</span></span> <span data-ttu-id="53ab8-556">Il metodo ottiene il valore dell'elemento attività dal corpo della richiesta HTTP.</span><span class="sxs-lookup"><span data-stu-id="53ab8-556">The method gets the value of the to-do item from the body of the HTTP request.</span></span>

<span data-ttu-id="53ab8-557">Per altre informazioni, vedere [Routing con attributi Http[verb]](xref:mvc/controllers/routing#attribute-routing-with-httpverb-attributes).</span><span class="sxs-lookup"><span data-stu-id="53ab8-557">For more information, see [Attribute routing with Http[Verb] attributes](xref:mvc/controllers/routing#attribute-routing-with-httpverb-attributes).</span></span>

<span data-ttu-id="53ab8-558">Il metodo <xref:Microsoft.AspNetCore.Mvc.ControllerBase.CreatedAtAction*>:</span><span class="sxs-lookup"><span data-stu-id="53ab8-558">The <xref:Microsoft.AspNetCore.Mvc.ControllerBase.CreatedAtAction*> method:</span></span>

* <span data-ttu-id="53ab8-559">Restituisce un codice di stato HTTP 201 in caso di esito positivo.</span><span class="sxs-lookup"><span data-stu-id="53ab8-559">Returns an HTTP 201 status code if successful.</span></span> <span data-ttu-id="53ab8-560">HTTP 201 è la risposta standard per un metodo HTTP POST che crea una nuova risorsa nel server.</span><span class="sxs-lookup"><span data-stu-id="53ab8-560">HTTP 201 is the standard response for an HTTP POST method that creates a new resource on the server.</span></span>
* <span data-ttu-id="53ab8-561">Aggiunge un'intestazione [Location](https://developer.mozilla.org/docs/Web/HTTP/Headers/Location) alla risposta.</span><span class="sxs-lookup"><span data-stu-id="53ab8-561">Adds a [Location](https://developer.mozilla.org/docs/Web/HTTP/Headers/Location) header to the response.</span></span> <span data-ttu-id="53ab8-562">L'intestazione `Location` specifica l'[URI](https://developer.mozilla.org/docs/Glossary/URI) dell'elemento attività appena creato.</span><span class="sxs-lookup"><span data-stu-id="53ab8-562">The `Location` header specifies the [URI](https://developer.mozilla.org/docs/Glossary/URI) of the newly created to-do item.</span></span> <span data-ttu-id="53ab8-563">Per altre informazioni, vedere [10.2.2 201 Created](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html).</span><span class="sxs-lookup"><span data-stu-id="53ab8-563">For more information, see [10.2.2 201 Created](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html).</span></span>
* <span data-ttu-id="53ab8-564">Fa riferimento all'azione `GetTodoItem` per creare l'URI dell'intestazione `Location`.</span><span class="sxs-lookup"><span data-stu-id="53ab8-564">References the `GetTodoItem` action to create the `Location` header's URI.</span></span> <span data-ttu-id="53ab8-565">La parola chiave `nameof` C# viene usata per evitare di impostare il nome dell'azione come hardcoded nella chiamata a `CreatedAtAction`.</span><span class="sxs-lookup"><span data-stu-id="53ab8-565">The C# `nameof` keyword is used to avoid hard-coding the action name in the `CreatedAtAction` call.</span></span>

### <a name="install-postman"></a><span data-ttu-id="53ab8-566">Installare Postman</span><span class="sxs-lookup"><span data-stu-id="53ab8-566">Install Postman</span></span>

<span data-ttu-id="53ab8-567">Questa esercitazione usa Postman per testare l'API Web.</span><span class="sxs-lookup"><span data-stu-id="53ab8-567">This tutorial uses Postman to test the web API.</span></span>

* <span data-ttu-id="53ab8-568">Installa il [post](https://www.getpostman.com/downloads/)</span><span class="sxs-lookup"><span data-stu-id="53ab8-568">Install [Postman](https://www.getpostman.com/downloads/)</span></span>
* <span data-ttu-id="53ab8-569">Avviare l'app Web.</span><span class="sxs-lookup"><span data-stu-id="53ab8-569">Start the web app.</span></span>
* <span data-ttu-id="53ab8-570">Avviare Postman.</span><span class="sxs-lookup"><span data-stu-id="53ab8-570">Start Postman.</span></span>
* <span data-ttu-id="53ab8-571">Disattivare **SSL certificate verification** (Verifica certificato SSL)</span><span class="sxs-lookup"><span data-stu-id="53ab8-571">Disable **SSL certificate verification**</span></span>
  * <span data-ttu-id="53ab8-572">Da **File** > **Settings** (Impostazioni) (scheda **General** (Generale)) disabilitare **SSL certificate verification** (Verifica certificato SSL).</span><span class="sxs-lookup"><span data-stu-id="53ab8-572">From **File** > **Settings** (**General** tab), disable **SSL certificate verification**.</span></span>
    > [!WARNING]
    > <span data-ttu-id="53ab8-573">Riattivare la verifica dei certificati SSL al termine del test del controller.</span><span class="sxs-lookup"><span data-stu-id="53ab8-573">Re-enable SSL certificate verification after testing the controller.</span></span>

<a name="post"></a>

### <a name="test-posttodoitem-with-postman"></a><span data-ttu-id="53ab8-574">Testare PostTodoItem con Postman</span><span class="sxs-lookup"><span data-stu-id="53ab8-574">Test PostTodoItem with Postman</span></span>

* <span data-ttu-id="53ab8-575">Creare una nuova richiesta.</span><span class="sxs-lookup"><span data-stu-id="53ab8-575">Create a new request.</span></span>
* <span data-ttu-id="53ab8-576">Impostare il metodo HTTP su `POST`.</span><span class="sxs-lookup"><span data-stu-id="53ab8-576">Set the HTTP method to `POST`.</span></span>
* <span data-ttu-id="53ab8-577">Impostare l'URI su `https://localhost:<port>/api/TodoItems` .</span><span class="sxs-lookup"><span data-stu-id="53ab8-577">Set the URI to `https://localhost:<port>/api/TodoItems`.</span></span> <span data-ttu-id="53ab8-578">Ad esempio: `https://localhost:5001/api/TodoItems`.</span><span class="sxs-lookup"><span data-stu-id="53ab8-578">For example, `https://localhost:5001/api/TodoItems`.</span></span>
* <span data-ttu-id="53ab8-579">Selezionare la scheda **Corpo**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-579">Select the **Body** tab.</span></span>
* <span data-ttu-id="53ab8-580">Selezionare il pulsante di opzione **raw** (non elaborato).</span><span class="sxs-lookup"><span data-stu-id="53ab8-580">Select the **raw** radio button.</span></span>
* <span data-ttu-id="53ab8-581">Impostare il tipo su **JSON (application/json)**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-581">Set the type to **JSON (application/json)**.</span></span>
* <span data-ttu-id="53ab8-582">Nel corpo della richiesta immettere codice JSON per un elemento attività:</span><span class="sxs-lookup"><span data-stu-id="53ab8-582">In the request body enter JSON for a to-do item:</span></span>

    ```json
    {
      "name":"walk dog",
      "isComplete":true
    }
    ```

* <span data-ttu-id="53ab8-583">Selezionare **Invia**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-583">Select **Send**.</span></span>

  ![Postman con richiesta di creazione](first-web-api/_static/3/create.png)

### <a name="test-the-location-header-uri-with-postman"></a><span data-ttu-id="53ab8-585">Testare l'URI dell'intestazione Location con post</span><span class="sxs-lookup"><span data-stu-id="53ab8-585">Test the location header URI with Postman</span></span>

* <span data-ttu-id="53ab8-586">Selezionare la scheda **Headers** (Intestazioni) nel riquadro **Response** (Risposta).</span><span class="sxs-lookup"><span data-stu-id="53ab8-586">Select the **Headers** tab in the **Response** pane.</span></span>
* <span data-ttu-id="53ab8-587">Copiare il valore dell'intestazione **Location** (Posizione):</span><span class="sxs-lookup"><span data-stu-id="53ab8-587">Copy the **Location** header value:</span></span>

  ![Scheda Headers (Intestazioni) della console Postman](first-web-api/_static/3/create.png)

* <span data-ttu-id="53ab8-589">Impostare il metodo HTTP su `GET`.</span><span class="sxs-lookup"><span data-stu-id="53ab8-589">Set the HTTP method to `GET`.</span></span>
* <span data-ttu-id="53ab8-590">Impostare l'URI su `https://localhost:<port>/api/TodoItems/1` .</span><span class="sxs-lookup"><span data-stu-id="53ab8-590">Set the URI to `https://localhost:<port>/api/TodoItems/1`.</span></span> <span data-ttu-id="53ab8-591">Ad esempio: `https://localhost:5001/api/TodoItems/1`.</span><span class="sxs-lookup"><span data-stu-id="53ab8-591">For example, `https://localhost:5001/api/TodoItems/1`.</span></span>
* <span data-ttu-id="53ab8-592">Selezionare **Invia**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-592">Select **Send**.</span></span>

## <a name="examine-the-get-methods"></a><span data-ttu-id="53ab8-593">Esaminare i metodi GET</span><span class="sxs-lookup"><span data-stu-id="53ab8-593">Examine the GET methods</span></span>

<span data-ttu-id="53ab8-594">Questi metodi implementano due metodi GET:</span><span class="sxs-lookup"><span data-stu-id="53ab8-594">These methods implement two GET endpoints:</span></span>

* `GET /api/TodoItems`
* `GET /api/TodoItems/{id}`

<span data-ttu-id="53ab8-595">Testare l'app chiamando i due endpoint da un browser o da Postman.</span><span class="sxs-lookup"><span data-stu-id="53ab8-595">Test the app by calling the two endpoints from a browser or Postman.</span></span> <span data-ttu-id="53ab8-596">Ad esempio:</span><span class="sxs-lookup"><span data-stu-id="53ab8-596">For example:</span></span>

* `https://localhost:5001/api/TodoItems`
* `https://localhost:5001/api/TodoItems/1`

<span data-ttu-id="53ab8-597">Una risposta simile alla seguente viene generata dalla chiamata a `GetTodoItems`:</span><span class="sxs-lookup"><span data-stu-id="53ab8-597">A response similar to the following is produced by the call to `GetTodoItems`:</span></span>

```json
[
  {
    "id": 1,
    "name": "Item1",
    "isComplete": false
  }
]
```

### <a name="test-get-with-postman"></a><span data-ttu-id="53ab8-598">Testare Get con Postman</span><span class="sxs-lookup"><span data-stu-id="53ab8-598">Test Get with Postman</span></span>

* <span data-ttu-id="53ab8-599">Creare una nuova richiesta.</span><span class="sxs-lookup"><span data-stu-id="53ab8-599">Create a new request.</span></span>
* <span data-ttu-id="53ab8-600">Impostare il metodo HTTP su **GET**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-600">Set the HTTP method to **GET**.</span></span>
* <span data-ttu-id="53ab8-601">Impostare l'URI della richiesta su `https://localhost:<port>/api/TodoItems` .</span><span class="sxs-lookup"><span data-stu-id="53ab8-601">Set the request URI to `https://localhost:<port>/api/TodoItems`.</span></span> <span data-ttu-id="53ab8-602">Ad esempio: `https://localhost:5001/api/TodoItems`.</span><span class="sxs-lookup"><span data-stu-id="53ab8-602">For example, `https://localhost:5001/api/TodoItems`.</span></span>
* <span data-ttu-id="53ab8-603">Impostare **Two pane view** (Visualizzazione in due riquadri) in Postman.</span><span class="sxs-lookup"><span data-stu-id="53ab8-603">Set **Two pane view** in Postman.</span></span>
* <span data-ttu-id="53ab8-604">Selezionare **Invia**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-604">Select **Send**.</span></span>

<span data-ttu-id="53ab8-605">Questa app usa un database in memoria.</span><span class="sxs-lookup"><span data-stu-id="53ab8-605">This app uses an in-memory database.</span></span> <span data-ttu-id="53ab8-606">Se l'app viene arrestata e avviata, la richiesta GET precedente non restituirà alcun dato.</span><span class="sxs-lookup"><span data-stu-id="53ab8-606">If the app is stopped and started, the preceding GET request will not return any data.</span></span> <span data-ttu-id="53ab8-607">Se non vengono restituiti dati, eseguire [POST](#post) per pubblicare i dati nell'app.</span><span class="sxs-lookup"><span data-stu-id="53ab8-607">If no data is returned, [POST](#post) data to the app.</span></span>

## <a name="routing-and-url-paths"></a><span data-ttu-id="53ab8-608">Routing e percorsi di URL</span><span class="sxs-lookup"><span data-stu-id="53ab8-608">Routing and URL paths</span></span>

<span data-ttu-id="53ab8-609">L' [`[HttpGet]`](xref:Microsoft.AspNetCore.Mvc.HttpGetAttribute) attributo indica un metodo che risponde a una richiesta HTTP Get.</span><span class="sxs-lookup"><span data-stu-id="53ab8-609">The [`[HttpGet]`](xref:Microsoft.AspNetCore.Mvc.HttpGetAttribute) attribute denotes a method that responds to an HTTP GET request.</span></span> <span data-ttu-id="53ab8-610">Il percorso dell'URL per ogni metodo viene costruito nel modo seguente:</span><span class="sxs-lookup"><span data-stu-id="53ab8-610">The URL path for each method is constructed as follows:</span></span>

* <span data-ttu-id="53ab8-611">Iniziare con la stringa di modello nell'attributo `Route` del controller:</span><span class="sxs-lookup"><span data-stu-id="53ab8-611">Start with the template string in the controller's `Route` attribute:</span></span>

  [!code-csharp[](first-web-api/samples/3.0/TodoApi/Controllers/TodoItemsController.cs?name=TodoController&highlight=1)]

* <span data-ttu-id="53ab8-612">Sostituire `[controller]` con il nome del controller, ovvero, per convenzione, il nome della classe controller meno il suffisso "Controller".</span><span class="sxs-lookup"><span data-stu-id="53ab8-612">Replace `[controller]` with the name of the controller, which by convention is the controller class name minus the "Controller" suffix.</span></span> <span data-ttu-id="53ab8-613">In questo esempio il nome della classe controller è **TodoItems** Controller, quindi il nome del controller è "TodoItems".</span><span class="sxs-lookup"><span data-stu-id="53ab8-613">For this sample, the controller class name is **TodoItems** Controller, so the controller name is "TodoItems".</span></span> <span data-ttu-id="53ab8-614">Il [routing](xref:mvc/controllers/routing) ASP.NET Core non fa distinzione tra maiuscole e minuscole.</span><span class="sxs-lookup"><span data-stu-id="53ab8-614">ASP.NET Core [routing](xref:mvc/controllers/routing) is case insensitive.</span></span>
* <span data-ttu-id="53ab8-615">Se l'attributo `[HttpGet]` ha un modello di route, ad esempio `[HttpGet("products")]`, aggiungerlo al percorso.</span><span class="sxs-lookup"><span data-stu-id="53ab8-615">If the `[HttpGet]` attribute has a route template (for example, `[HttpGet("products")]`), append that to the path.</span></span> <span data-ttu-id="53ab8-616">In questo esempio non si usa un modello.</span><span class="sxs-lookup"><span data-stu-id="53ab8-616">This sample doesn't use a template.</span></span> <span data-ttu-id="53ab8-617">Per altre informazioni, vedere [Routing con attributi Http[verb]](xref:mvc/controllers/routing#attribute-routing-with-httpverb-attributes).</span><span class="sxs-lookup"><span data-stu-id="53ab8-617">For more information, see [Attribute routing with Http[Verb] attributes](xref:mvc/controllers/routing#attribute-routing-with-httpverb-attributes).</span></span>

<span data-ttu-id="53ab8-618">Nel metodo `GetTodoItem` seguente, `"{id}"` è una variabile segnaposto per l'identificatore univoco dell'elemento attività.</span><span class="sxs-lookup"><span data-stu-id="53ab8-618">In the following `GetTodoItem` method, `"{id}"` is a placeholder variable for the unique identifier of the to-do item.</span></span> <span data-ttu-id="53ab8-619">Quando `GetTodoItem` viene richiamato, il valore di `"{id}"` nell'URL viene fornito al metodo nel `id` parametro.</span><span class="sxs-lookup"><span data-stu-id="53ab8-619">When `GetTodoItem` is invoked, the value of `"{id}"` in the URL is provided to the method in its `id` parameter.</span></span>

[!code-csharp[](first-web-api/samples/3.0/TodoApi/Controllers/TodoItemsController.cs?name=snippet_GetByID&highlight=1-2)]

## <a name="return-values"></a><span data-ttu-id="53ab8-620">Valori restituiti</span><span class="sxs-lookup"><span data-stu-id="53ab8-620">Return values</span></span> 

<span data-ttu-id="53ab8-621">Il tipo restituito dei `GetTodoItems` metodi e `GetTodoItem` è [ActionResult. \<T> ](xref:web-api/action-return-types#actionresultt-type)</span><span class="sxs-lookup"><span data-stu-id="53ab8-621">The return type of the `GetTodoItems` and `GetTodoItem` methods is [ActionResult\<T> type](xref:web-api/action-return-types#actionresultt-type).</span></span> <span data-ttu-id="53ab8-622">ASP.NET Core serializza automaticamente l'oggetto su [JSON](https://www.json.org/) e scrive il codice JSON nel corpo del messaggio di risposta.</span><span class="sxs-lookup"><span data-stu-id="53ab8-622">ASP.NET Core automatically serializes the object to [JSON](https://www.json.org/) and writes the JSON into the body of the response message.</span></span> <span data-ttu-id="53ab8-623">Il codice di risposta per questo tipo restituito è 200, presupponendo che non esistano eccezioni non gestite.</span><span class="sxs-lookup"><span data-stu-id="53ab8-623">The response code for this return type is 200, assuming there are no unhandled exceptions.</span></span> <span data-ttu-id="53ab8-624">Le eccezioni non gestite vengono convertite in errori 5xx.</span><span class="sxs-lookup"><span data-stu-id="53ab8-624">Unhandled exceptions are translated into 5xx errors.</span></span>

<span data-ttu-id="53ab8-625">I tipi restituiti `ActionResult` possono rappresentare un ampio intervallo di codici di stato HTTP.</span><span class="sxs-lookup"><span data-stu-id="53ab8-625">`ActionResult` return types can represent a wide range of HTTP status codes.</span></span> <span data-ttu-id="53ab8-626">Ad esempio, `GetTodoItem` può restituire due valori di stato diversi:</span><span class="sxs-lookup"><span data-stu-id="53ab8-626">For example, `GetTodoItem` can return two different status values:</span></span>

* <span data-ttu-id="53ab8-627">Se nessun elemento corrisponde all'ID richiesto, il metodo restituisce un <xref:Microsoft.AspNetCore.Mvc.ControllerBase.NotFound%2A> codice di errore 404.</span><span class="sxs-lookup"><span data-stu-id="53ab8-627">If no item matches the requested ID, the method returns a 404 <xref:Microsoft.AspNetCore.Mvc.ControllerBase.NotFound%2A> error code.</span></span>
* <span data-ttu-id="53ab8-628">In caso contrario, il metodo restituisce il codice 200 con un corpo della risposta JSON.</span><span class="sxs-lookup"><span data-stu-id="53ab8-628">Otherwise, the method returns 200 with a JSON response body.</span></span> <span data-ttu-id="53ab8-629">La restituzione di `item` risulta in una risposta HTTP 200.</span><span class="sxs-lookup"><span data-stu-id="53ab8-629">Returning `item` results in an HTTP 200 response.</span></span>

## <a name="the-puttodoitem-method"></a><span data-ttu-id="53ab8-630">Metodo PutTodoItem</span><span class="sxs-lookup"><span data-stu-id="53ab8-630">The PutTodoItem method</span></span>

<span data-ttu-id="53ab8-631">Esaminare il metodo `PutTodoItem`:</span><span class="sxs-lookup"><span data-stu-id="53ab8-631">Examine the `PutTodoItem` method:</span></span>

[!code-csharp[](first-web-api/samples/3.0/TodoApi/Controllers/TodoItemsController.cs?name=snippet_Update)]

<span data-ttu-id="53ab8-632">`PutTodoItem` è simile a `PostTodoItem` ma usa la richiesta HTTP PUT.</span><span class="sxs-lookup"><span data-stu-id="53ab8-632">`PutTodoItem` is similar to `PostTodoItem`, except it uses HTTP PUT.</span></span> <span data-ttu-id="53ab8-633">La risposta è [204 (No Content)](https://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html).</span><span class="sxs-lookup"><span data-stu-id="53ab8-633">The response is [204 (No Content)](https://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html).</span></span> <span data-ttu-id="53ab8-634">In base alla specifica HTTP, una richiesta PUT richiede che il client invii l'intera entità aggiornata e non solo le modifiche.</span><span class="sxs-lookup"><span data-stu-id="53ab8-634">According to the HTTP specification, a PUT request requires the client to send the entire updated entity, not just the changes.</span></span> <span data-ttu-id="53ab8-635">Per supportare gli aggiornamenti parziali, usare [HTTP PATCH](xref:Microsoft.AspNetCore.Mvc.HttpPatchAttribute).</span><span class="sxs-lookup"><span data-stu-id="53ab8-635">To support partial updates, use [HTTP PATCH](xref:Microsoft.AspNetCore.Mvc.HttpPatchAttribute).</span></span>

<span data-ttu-id="53ab8-636">Se si riceve un errore chiamando `PutTodoItem`, chiamare `GET` per verificare che il database includa un elemento.</span><span class="sxs-lookup"><span data-stu-id="53ab8-636">If you get an error calling `PutTodoItem`, call `GET` to ensure there's an item in the database.</span></span>

### <a name="test-the-puttodoitem-method"></a><span data-ttu-id="53ab8-637">Testare il metodo PutTodoItem</span><span class="sxs-lookup"><span data-stu-id="53ab8-637">Test the PutTodoItem method</span></span>

<span data-ttu-id="53ab8-638">Questo esempio usa un database in memoria che deve essere inizializzato ogni volta che l'app viene avviata.</span><span class="sxs-lookup"><span data-stu-id="53ab8-638">This sample uses an in-memory database that must be initialized each time the app is started.</span></span> <span data-ttu-id="53ab8-639">Deve esistere un elemento nel database prima di eseguire una chiamata PUT.</span><span class="sxs-lookup"><span data-stu-id="53ab8-639">There must be an item in the database before you make a PUT call.</span></span> <span data-ttu-id="53ab8-640">Chiamare GET per assicurarsi che sia presente un elemento nel database prima di effettuare una chiamata PUT.</span><span class="sxs-lookup"><span data-stu-id="53ab8-640">Call GET to ensure there's an item in the database before making a PUT call.</span></span>

<span data-ttu-id="53ab8-641">Aggiornare l'attività con ID = 1 e impostarne il nome su "feed Fish":</span><span class="sxs-lookup"><span data-stu-id="53ab8-641">Update the to-do item that has Id = 1 and set its name to "feed fish":</span></span>

```json
  {
    "id":1,
    "name":"feed fish",
    "isComplete":true
  }
```

<span data-ttu-id="53ab8-642">L'immagine seguente visualizza l'aggiornamento di Postman:</span><span class="sxs-lookup"><span data-stu-id="53ab8-642">The following image shows the Postman update:</span></span>

![Console Postman con risposta 204 (No Content)](first-web-api/_static/3/pmcput.png)

## <a name="the-deletetodoitem-method"></a><span data-ttu-id="53ab8-644">Metodo DeleteTodoItem</span><span class="sxs-lookup"><span data-stu-id="53ab8-644">The DeleteTodoItem method</span></span>

<span data-ttu-id="53ab8-645">Esaminare il metodo `DeleteTodoItem`:</span><span class="sxs-lookup"><span data-stu-id="53ab8-645">Examine the `DeleteTodoItem` method:</span></span>

[!code-csharp[](first-web-api/samples/3.0/TodoApi/Controllers/TodoItemsController.cs?name=snippet_Delete)]

### <a name="test-the-deletetodoitem-method"></a><span data-ttu-id="53ab8-646">Testare il metodo DeleteTodoItem</span><span class="sxs-lookup"><span data-stu-id="53ab8-646">Test the DeleteTodoItem method</span></span>

<span data-ttu-id="53ab8-647">Usare Postman per eliminare un elemento attività:</span><span class="sxs-lookup"><span data-stu-id="53ab8-647">Use Postman to delete a to-do item:</span></span>

* <span data-ttu-id="53ab8-648">Impostare il metodo su `DELETE`.</span><span class="sxs-lookup"><span data-stu-id="53ab8-648">Set the method to `DELETE`.</span></span>
* <span data-ttu-id="53ab8-649">Impostare l'URI dell'oggetto da eliminare (ad esempio `https://localhost:5001/api/TodoItems/1` ).</span><span class="sxs-lookup"><span data-stu-id="53ab8-649">Set the URI of the object to delete (for example `https://localhost:5001/api/TodoItems/1`).</span></span>
* <span data-ttu-id="53ab8-650">Selezionare **Invia**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-650">Select **Send**.</span></span>

<a name="over-post"></a>
<a name="over-post-v3"></a>

## <a name="prevent-over-posting"></a><span data-ttu-id="53ab8-651">Impedisci overposting</span><span class="sxs-lookup"><span data-stu-id="53ab8-651">Prevent over-posting</span></span>

<span data-ttu-id="53ab8-652">Attualmente l'app di esempio espone l'intero `TodoItem` oggetto.</span><span class="sxs-lookup"><span data-stu-id="53ab8-652">Currently the sample app exposes the entire `TodoItem` object.</span></span> <span data-ttu-id="53ab8-653">Le app di produzione limitano in genere i dati di input e restituiti usando un subset del modello.</span><span class="sxs-lookup"><span data-stu-id="53ab8-653">Production apps typically limit the data that's input and returned using a subset of the model.</span></span> <span data-ttu-id="53ab8-654">Esistono diversi motivi alla base di questo e la sicurezza è una delle principali.</span><span class="sxs-lookup"><span data-stu-id="53ab8-654">There are multiple reasons behind this and security is a major one.</span></span> <span data-ttu-id="53ab8-655">Il subset di un modello è in genere indicato come oggetto Trasferimento dati (DTO), modello di input o modello di visualizzazione.</span><span class="sxs-lookup"><span data-stu-id="53ab8-655">The subset of a model is usually referred to as a Data Transfer Object (DTO), input model, or view model.</span></span> <span data-ttu-id="53ab8-656">**Dto** viene usato in questo articolo.</span><span class="sxs-lookup"><span data-stu-id="53ab8-656">**DTO** is used in this article.</span></span>

<span data-ttu-id="53ab8-657">Un DTO può essere usato per:</span><span class="sxs-lookup"><span data-stu-id="53ab8-657">A DTO may be used to:</span></span>

* <span data-ttu-id="53ab8-658">Impedisci l'overposting.</span><span class="sxs-lookup"><span data-stu-id="53ab8-658">Prevent over-posting.</span></span>
* <span data-ttu-id="53ab8-659">Nascondere le proprietà che i client non dovrebbero visualizzare.</span><span class="sxs-lookup"><span data-stu-id="53ab8-659">Hide properties that clients are not supposed to view.</span></span>
* <span data-ttu-id="53ab8-660">Omettere alcune proprietà per ridurre le dimensioni del payload.</span><span class="sxs-lookup"><span data-stu-id="53ab8-660">Omit some properties in order to reduce payload size.</span></span>
* <span data-ttu-id="53ab8-661">Rendere flat gli oggetti grafici che contengono oggetti annidati.</span><span class="sxs-lookup"><span data-stu-id="53ab8-661">Flatten object graphs that contain nested objects.</span></span> <span data-ttu-id="53ab8-662">I grafici a oggetti flat possono essere più pratici per i client.</span><span class="sxs-lookup"><span data-stu-id="53ab8-662">Flattened object graphs can be more convenient for clients.</span></span>

<span data-ttu-id="53ab8-663">Per illustrare l'approccio DTO, aggiornare la `TodoItem` classe in modo da includere un campo Secret:</span><span class="sxs-lookup"><span data-stu-id="53ab8-663">To demonstrate the DTO approach, update the `TodoItem` class to include a secret field:</span></span>

[!code-csharp[](first-web-api/samples/3.0/TodoApiDTO/Models/TodoItem.cs?name=snippet&highlight=6)]

<span data-ttu-id="53ab8-664">Il campo segreto deve essere nascosto da questa app, ma un'app amministrativa può scegliere di esporla.</span><span class="sxs-lookup"><span data-stu-id="53ab8-664">The secret field needs to be hidden from this app, but an administrative app could choose to expose it.</span></span>

<span data-ttu-id="53ab8-665">Verificare che sia possibile pubblicare e ottenere il campo secret.</span><span class="sxs-lookup"><span data-stu-id="53ab8-665">Verify you can post and get the secret field.</span></span>

<span data-ttu-id="53ab8-666">Creare un modello DTO:</span><span class="sxs-lookup"><span data-stu-id="53ab8-666">Create a DTO model:</span></span>

[!code-csharp[](first-web-api/samples/3.0/TodoApiDTO/Models/TodoItemDTO.cs?name=snippet)]

<span data-ttu-id="53ab8-667">Aggiornare `TodoItemsController` da usare `TodoItemDTO` :</span><span class="sxs-lookup"><span data-stu-id="53ab8-667">Update the `TodoItemsController` to use `TodoItemDTO`:</span></span>

[!code-csharp[](first-web-api/samples/3.0/TodoApiDTO/Controllers/TodoItemsController.cs?name=snippet)]

<span data-ttu-id="53ab8-668">Verificare che non sia possibile inserire o ottenere il campo secret.</span><span class="sxs-lookup"><span data-stu-id="53ab8-668">Verify you can't post or get the secret field.</span></span>

## <a name="call-the-web-api-with-javascript"></a><span data-ttu-id="53ab8-669">Chiamare l'API Web con JavaScript</span><span class="sxs-lookup"><span data-stu-id="53ab8-669">Call the web API with JavaScript</span></span>

<span data-ttu-id="53ab8-670">Vedere [esercitazione: chiamare un'API web ASP.NET Core con JavaScript](xref:tutorials/web-api-javascript).</span><span class="sxs-lookup"><span data-stu-id="53ab8-670">See [Tutorial: Call an ASP.NET Core web API with JavaScript](xref:tutorials/web-api-javascript).</span></span>

::: moniker-end

::: moniker range="< aspnetcore-3.0"

<span data-ttu-id="53ab8-671">In questa esercitazione verranno illustrate le procedure per:</span><span class="sxs-lookup"><span data-stu-id="53ab8-671">In this tutorial, you learn how to:</span></span>

> [!div class="checklist"]
> * <span data-ttu-id="53ab8-672">Creare un progetto di API Web.</span><span class="sxs-lookup"><span data-stu-id="53ab8-672">Create a web API project.</span></span>
> * <span data-ttu-id="53ab8-673">Aggiungere una classe modello e un contesto di database.</span><span class="sxs-lookup"><span data-stu-id="53ab8-673">Add a model class and a database context.</span></span>
> * <span data-ttu-id="53ab8-674">Aggiungere un controller.</span><span class="sxs-lookup"><span data-stu-id="53ab8-674">Add a controller.</span></span>
> * <span data-ttu-id="53ab8-675">Aggiungere metodi CRUD.</span><span class="sxs-lookup"><span data-stu-id="53ab8-675">Add CRUD methods.</span></span>
> * <span data-ttu-id="53ab8-676">Configurare routing e percorsi URL.</span><span class="sxs-lookup"><span data-stu-id="53ab8-676">Configure routing and URL paths.</span></span>
> * <span data-ttu-id="53ab8-677">Specificare valori restituiti.</span><span class="sxs-lookup"><span data-stu-id="53ab8-677">Specify return values.</span></span>
> * <span data-ttu-id="53ab8-678">Chiamare l'API Web con Postman.</span><span class="sxs-lookup"><span data-stu-id="53ab8-678">Call the web API with Postman.</span></span>
> * <span data-ttu-id="53ab8-679">Chiamare l'API Web con JavaScript.</span><span class="sxs-lookup"><span data-stu-id="53ab8-679">Call the web API with JavaScript.</span></span>

<span data-ttu-id="53ab8-680">Al termine si dispone di un'API web che può gestire gli elementi di tipo "attività" archiviati in un database relazionale.</span><span class="sxs-lookup"><span data-stu-id="53ab8-680">At the end, you have a web API that can manage "to-do" items stored in a relational database.</span></span>

## <a name="overview-21"></a><span data-ttu-id="53ab8-681">Panoramica 2,1</span><span class="sxs-lookup"><span data-stu-id="53ab8-681">Overview 2.1</span></span>

<span data-ttu-id="53ab8-682">Questa esercitazione consente di creare l'API seguente:</span><span class="sxs-lookup"><span data-stu-id="53ab8-682">This tutorial creates the following API:</span></span>

|<span data-ttu-id="53ab8-683">API</span><span class="sxs-lookup"><span data-stu-id="53ab8-683">API</span></span> | <span data-ttu-id="53ab8-684">Descrizione</span><span class="sxs-lookup"><span data-stu-id="53ab8-684">Description</span></span> | <span data-ttu-id="53ab8-685">Testo della richiesta</span><span class="sxs-lookup"><span data-stu-id="53ab8-685">Request body</span></span> | <span data-ttu-id="53ab8-686">Corpo della risposta</span><span class="sxs-lookup"><span data-stu-id="53ab8-686">Response body</span></span> |
|--- | ---- | ---- | ---- |
|<span data-ttu-id="53ab8-687">GET /api/TodoItems</span><span class="sxs-lookup"><span data-stu-id="53ab8-687">GET /api/TodoItems</span></span> | <span data-ttu-id="53ab8-688">Ottiene tutti gli elementi attività</span><span class="sxs-lookup"><span data-stu-id="53ab8-688">Get all to-do items</span></span> | <span data-ttu-id="53ab8-689">nessuno</span><span class="sxs-lookup"><span data-stu-id="53ab8-689">None</span></span> | <span data-ttu-id="53ab8-690">Matrice di elementi attività</span><span class="sxs-lookup"><span data-stu-id="53ab8-690">Array of to-do items</span></span>|
|<span data-ttu-id="53ab8-691">GET /api/TodoItems/{id}</span><span class="sxs-lookup"><span data-stu-id="53ab8-691">GET /api/TodoItems/{id}</span></span> | <span data-ttu-id="53ab8-692">Ottiene un elemento in base all'ID</span><span class="sxs-lookup"><span data-stu-id="53ab8-692">Get an item by ID</span></span> | <span data-ttu-id="53ab8-693">nessuno</span><span class="sxs-lookup"><span data-stu-id="53ab8-693">None</span></span> | <span data-ttu-id="53ab8-694">Elemento attività</span><span class="sxs-lookup"><span data-stu-id="53ab8-694">To-do item</span></span>|
|<span data-ttu-id="53ab8-695">POST /api/TodoItems</span><span class="sxs-lookup"><span data-stu-id="53ab8-695">POST /api/TodoItems</span></span> | <span data-ttu-id="53ab8-696">Aggiunge un nuovo elemento</span><span class="sxs-lookup"><span data-stu-id="53ab8-696">Add a new item</span></span> | <span data-ttu-id="53ab8-697">Elemento attività</span><span class="sxs-lookup"><span data-stu-id="53ab8-697">To-do item</span></span> | <span data-ttu-id="53ab8-698">Elemento attività</span><span class="sxs-lookup"><span data-stu-id="53ab8-698">To-do item</span></span> |
|<span data-ttu-id="53ab8-699">PUT /api/TodoItems/{id}</span><span class="sxs-lookup"><span data-stu-id="53ab8-699">PUT /api/TodoItems/{id}</span></span> | <span data-ttu-id="53ab8-700">Aggiorna un elemento esistente &nbsp;</span><span class="sxs-lookup"><span data-stu-id="53ab8-700">Update an existing item &nbsp;</span></span> | <span data-ttu-id="53ab8-701">Elemento attività</span><span class="sxs-lookup"><span data-stu-id="53ab8-701">To-do item</span></span> | <span data-ttu-id="53ab8-702">nessuno</span><span class="sxs-lookup"><span data-stu-id="53ab8-702">None</span></span> |
|<span data-ttu-id="53ab8-703">Elimina/api/TodoItems/{id} &nbsp;&nbsp;</span><span class="sxs-lookup"><span data-stu-id="53ab8-703">DELETE /api/TodoItems/{id} &nbsp; &nbsp;</span></span> | <span data-ttu-id="53ab8-704">Elimina un elemento &nbsp;&nbsp;</span><span class="sxs-lookup"><span data-stu-id="53ab8-704">Delete an item &nbsp; &nbsp;</span></span> | <span data-ttu-id="53ab8-705">nessuno</span><span class="sxs-lookup"><span data-stu-id="53ab8-705">None</span></span> | <span data-ttu-id="53ab8-706">nessuno</span><span class="sxs-lookup"><span data-stu-id="53ab8-706">None</span></span>|

<span data-ttu-id="53ab8-707">Il diagramma seguente visualizza la struttura dell'app.</span><span class="sxs-lookup"><span data-stu-id="53ab8-707">The following diagram shows the design of the app.</span></span>

![Il client è rappresentato da una casella a sinistra.](first-web-api/_static/architecture.png)

## <a name="prerequisites-21"></a><span data-ttu-id="53ab8-713">Prerequisiti 2,1</span><span class="sxs-lookup"><span data-stu-id="53ab8-713">Prerequisites 2.1</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="53ab8-714">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="53ab8-714">Visual Studio</span></span>](#tab/visual-studio)

[!INCLUDE[](~/includes/net-core-prereqs-vs2019-2.2.md)]

# <a name="visual-studio-code"></a>[<span data-ttu-id="53ab8-715">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="53ab8-715">Visual Studio Code</span></span>](#tab/visual-studio-code)

[!INCLUDE[](~/includes/net-core-prereqs-vsc-2.2.md)]

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="53ab8-716">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="53ab8-716">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

[!INCLUDE[](~/includes/net-core-prereqs-mac-2.2.md)]

---

## <a name="create-a-web-project-21"></a><span data-ttu-id="53ab8-717">Creare un progetto Web 2,1</span><span class="sxs-lookup"><span data-stu-id="53ab8-717">Create a web project 2.1</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="53ab8-718">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="53ab8-718">Visual Studio</span></span>](#tab/visual-studio)

* <span data-ttu-id="53ab8-719">Scegliere **nuovo** progetto dal menu **file** > .</span><span class="sxs-lookup"><span data-stu-id="53ab8-719">From the **File** menu, select **New** > **Project**.</span></span>
* <span data-ttu-id="53ab8-720">Selezionare il modello **Applicazione Web ASP.NET Core** e fare clic su **Avanti**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-720">Select the **ASP.NET Core Web Application** template and click **Next**.</span></span>
* <span data-ttu-id="53ab8-721">Assegnare al progetto il nome *TodoApi* e fare clic su **Crea**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-721">Name the project *TodoApi* and click **Create**.</span></span>
* <span data-ttu-id="53ab8-722">Nella finestra di dialogo **Crea una nuova applicazione Web ASP.NET Core** verificare che siano selezionati **.NET Core** e **ASP.NET Core 2.2**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-722">In the **Create a new ASP.NET Core Web Application** dialog, confirm that **.NET Core** and **ASP.NET Core 2.2** are selected.</span></span> <span data-ttu-id="53ab8-723">Selezionare il modello **API** e fare clic su **Crea**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-723">Select the **API** template and click **Create**.</span></span> <span data-ttu-id="53ab8-724">**Non** selezionare **Abilita supporto Docker**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-724">**Don't** select **Enable Docker Support**.</span></span>

![Finestra di dialogo Nuovo progetto di Visual Studio](first-web-api/_static/vs.png)

# <a name="visual-studio-code"></a>[<span data-ttu-id="53ab8-726">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="53ab8-726">Visual Studio Code</span></span>](#tab/visual-studio-code)

* <span data-ttu-id="53ab8-727">Aprire il [terminale integrato](https://code.visualstudio.com/docs/editor/integrated-terminal).</span><span class="sxs-lookup"><span data-stu-id="53ab8-727">Open the [integrated terminal](https://code.visualstudio.com/docs/editor/integrated-terminal).</span></span>
* <span data-ttu-id="53ab8-728">Cambiare directory (`cd`) e passare alla cartella che conterrà la cartella del progetto.</span><span class="sxs-lookup"><span data-stu-id="53ab8-728">Change directories (`cd`) to the folder that will contain the project folder.</span></span>
* <span data-ttu-id="53ab8-729">Eseguire i comandi seguenti:</span><span class="sxs-lookup"><span data-stu-id="53ab8-729">Run the following commands:</span></span>

   ```dotnetcli
   dotnet new webapi -o TodoApi
   code -r TodoApi
   ```

  <span data-ttu-id="53ab8-730">Questi comandi creano un nuovo progetto API Web e aprono una nuova istanza di Visual Studio Code nella nuova cartella di progetto.</span><span class="sxs-lookup"><span data-stu-id="53ab8-730">These commands create a new web API project and open a new instance of Visual Studio Code in the new project folder.</span></span>

* <span data-ttu-id="53ab8-731">Quando una finestra di dialogo chiede se si vuole aggiungere gli asset necessari al progetto, selezionare **Sì**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-731">When a dialog box asks if you want to add required assets to the project, select **Yes**.</span></span>

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="53ab8-732">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="53ab8-732">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

* <span data-ttu-id="53ab8-733">Selezionare **File** > **Nuova soluzione**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-733">Select **File** > **New Solution**.</span></span>

  ![Nuova soluzione macOS](first-web-api-mac/_static/sln.png)

* <span data-ttu-id="53ab8-735">In Visual Studio per Mac precedente alla versione 8,6 Selezionare API **app .NET Core**  >    >    >  **Avanti**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-735">In Visual Studio for Mac earlier than version 8.6, select **.NET Core** > **App** > **API** > **Next**.</span></span> <span data-ttu-id="53ab8-736">Nella versione 8,6 o successive selezionare API **app Web e console**  >    >    >  **Avanti**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-736">In version 8.6 or later, select **Web and Console** > **App** > **API** > **Next**.</span></span>
  
* <span data-ttu-id="53ab8-737">Nella finestra di dialogo **Configura nuova ASP.NET Core API Web** selezionare il **Framework di destinazione**.NET Core 2. x più recente.</span><span class="sxs-lookup"><span data-stu-id="53ab8-737">In the **Configure the new ASP.NET Core Web API** dialog, select the latest .NET Core 2.x **Target Framework**.</span></span> <span data-ttu-id="53ab8-738">Selezionare **Avanti**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-738">Select **Next**.</span></span>

* <span data-ttu-id="53ab8-739">Immettere *TodoApi* in **Nome progetto**, quindi selezionare **Crea**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-739">Enter *TodoApi* for the **Project Name** and then select **Create**.</span></span>

  ![Finestra di dialogo di configurazione](first-web-api-mac/_static/2.png)

---

### <a name="test-the-api-21"></a><span data-ttu-id="53ab8-741">Testare l'API 2,1</span><span class="sxs-lookup"><span data-stu-id="53ab8-741">Test the API 2.1</span></span>

<span data-ttu-id="53ab8-742">Il modello di progetto crea un'API `values`.</span><span class="sxs-lookup"><span data-stu-id="53ab8-742">The project template creates a `values` API.</span></span> <span data-ttu-id="53ab8-743">Chiamare il metodo `Get` da un browser per eseguire il test dell'app.</span><span class="sxs-lookup"><span data-stu-id="53ab8-743">Call the `Get` method from a browser to test the app.</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="53ab8-744">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="53ab8-744">Visual Studio</span></span>](#tab/visual-studio)

<span data-ttu-id="53ab8-745">Premere CTRL+F5 per eseguire l'app.</span><span class="sxs-lookup"><span data-stu-id="53ab8-745">Press Ctrl+F5 to run the app.</span></span> <span data-ttu-id="53ab8-746">Visual Studio apre un browser e naviga all'indirizzo `https://localhost:<port>/api/values`, dove `<port>` è un numero di porta selezionato a caso.</span><span class="sxs-lookup"><span data-stu-id="53ab8-746">Visual Studio launches a browser and navigates to `https://localhost:<port>/api/values`, where `<port>` is a randomly chosen port number.</span></span>

<span data-ttu-id="53ab8-747">Se in una finestra di dialogo viene chiesto se considerare attendibile il certificato IIS Express, selezionare **Sì**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-747">If you get a dialog box that asks if you should trust the IIS Express certificate, select **Yes**.</span></span> <span data-ttu-id="53ab8-748">Nella finestra di dialogo **Avviso di sicurezza** successiva selezionare **Sì**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-748">In the **Security Warning** dialog that appears next, select **Yes**.</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="53ab8-749">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="53ab8-749">Visual Studio Code</span></span>](#tab/visual-studio-code)

<span data-ttu-id="53ab8-750">Premere CTRL+F5 per eseguire l'app.</span><span class="sxs-lookup"><span data-stu-id="53ab8-750">Press Ctrl+F5 to run the app.</span></span> <span data-ttu-id="53ab8-751">In un browser passare all'URL seguente: `https://localhost:5001/api/values`.</span><span class="sxs-lookup"><span data-stu-id="53ab8-751">In a browser, go to following URL: `https://localhost:5001/api/values`.</span></span>

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="53ab8-752">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="53ab8-752">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

<span data-ttu-id="53ab8-753">Selezionare **Esegui**  >  **Avvia debug** per avviare l'app.</span><span class="sxs-lookup"><span data-stu-id="53ab8-753">Select **Run** > **Start Debugging** to launch the app.</span></span> <span data-ttu-id="53ab8-754">Visual Studio per Mac apre un browser e naviga all'indirizzo `https://localhost:<port>`, dove `<port>` è un numero di porta selezionato a caso.</span><span class="sxs-lookup"><span data-stu-id="53ab8-754">Visual Studio for Mac launches a browser and navigates to `https://localhost:<port>`, where `<port>` is a randomly chosen port number.</span></span> <span data-ttu-id="53ab8-755">Viene restituito un errore HTTP 404 (Non trovato).</span><span class="sxs-lookup"><span data-stu-id="53ab8-755">An HTTP 404 (Not Found) error is returned.</span></span> <span data-ttu-id="53ab8-756">Accodare `/api/values` all'URL (impostare l'URL su `https://localhost:<port>/api/values`).</span><span class="sxs-lookup"><span data-stu-id="53ab8-756">Append `/api/values` to the URL (change the URL to `https://localhost:<port>/api/values`).</span></span>

---

<span data-ttu-id="53ab8-757">Viene restituito il codice JSON seguente:</span><span class="sxs-lookup"><span data-stu-id="53ab8-757">The following JSON is returned:</span></span>

```json
["value1","value2"]
```

## <a name="add-a-model-class-21"></a><span data-ttu-id="53ab8-758">Aggiungere una classe modello 2,1</span><span class="sxs-lookup"><span data-stu-id="53ab8-758">Add a model class 2.1</span></span>

<span data-ttu-id="53ab8-759">Un *modello* è un set di classi che rappresentano i dati gestiti dall'app.</span><span class="sxs-lookup"><span data-stu-id="53ab8-759">A *model* is a set of classes that represent the data that the app manages.</span></span> <span data-ttu-id="53ab8-760">Il modello per questa app è una classe `TodoItem` singola.</span><span class="sxs-lookup"><span data-stu-id="53ab8-760">The model for this app is a single `TodoItem` class.</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="53ab8-761">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="53ab8-761">Visual Studio</span></span>](#tab/visual-studio)

* <span data-ttu-id="53ab8-762">In **Esplora soluzioni** fare clic con il pulsante destro del mouse sul progetto.</span><span class="sxs-lookup"><span data-stu-id="53ab8-762">In **Solution Explorer**, right-click the project.</span></span> <span data-ttu-id="53ab8-763">Selezionare **Aggiungi**  >  **nuova cartella**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-763">Select **Add** > **New Folder**.</span></span> <span data-ttu-id="53ab8-764">Assegnare un nome alla cartella *Models* .</span><span class="sxs-lookup"><span data-stu-id="53ab8-764">Name the folder *Models*.</span></span>

* <span data-ttu-id="53ab8-765">Fare clic con il pulsante destro del mouse sulla *Models* cartella e scegliere **Aggiungi**  >  **classe**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-765">Right-click the *Models* folder and select **Add** > **Class**.</span></span> <span data-ttu-id="53ab8-766">Assegnare alla classe il nome *TodoItem* e selezionare **Aggiungi**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-766">Name the class *TodoItem* and select **Add**.</span></span>

* <span data-ttu-id="53ab8-767">Sostituire il codice del modello con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="53ab8-767">Replace the template code with the following code:</span></span>

# <a name="visual-studio-code"></a>[<span data-ttu-id="53ab8-768">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="53ab8-768">Visual Studio Code</span></span>](#tab/visual-studio-code)

* <span data-ttu-id="53ab8-769">Aggiungere una cartella denominata *Models* .</span><span class="sxs-lookup"><span data-stu-id="53ab8-769">Add a folder named *Models*.</span></span>

* <span data-ttu-id="53ab8-770">Aggiungere una `TodoItem` classe alla *Models* cartella con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="53ab8-770">Add a `TodoItem` class to the *Models* folder with the following code:</span></span>

# <a name="visual-studio-for-mac"></a>[<span data-ttu-id="53ab8-771">Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="53ab8-771">Visual Studio for Mac</span></span>](#tab/visual-studio-mac)

* <span data-ttu-id="53ab8-772">Fare clic con il pulsante destro del mouse sul progetto.</span><span class="sxs-lookup"><span data-stu-id="53ab8-772">Right-click the project.</span></span> <span data-ttu-id="53ab8-773">Selezionare **Aggiungi**  >  **nuova cartella**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-773">Select **Add** > **New Folder**.</span></span> <span data-ttu-id="53ab8-774">Assegnare un nome alla cartella *Models* .</span><span class="sxs-lookup"><span data-stu-id="53ab8-774">Name the folder *Models*.</span></span>

  ![Nuova cartella](first-web-api-mac/_static/folder.png)

* <span data-ttu-id="53ab8-776">Fare clic con il pulsante destro del mouse sulla *Models* cartella e scegliere **Aggiungi** > **nuovo file** > **generale** > **vuota classe**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-776">Right-click the *Models* folder, and select **Add** > **New File** > **General** > **Empty Class**.</span></span>

* <span data-ttu-id="53ab8-777">Assegnare alla classe il nome *TodoItem* e fare clic su **Nuovo**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-777">Name the class *TodoItem*, and then click **New**.</span></span>

* <span data-ttu-id="53ab8-778">Sostituire il codice del modello con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="53ab8-778">Replace the template code with the following code:</span></span>

---

  [!code-csharp[](first-web-api/samples/2.2/TodoApi/Models/TodoItem.cs)]

<span data-ttu-id="53ab8-779">La proprietà `Id` funziona come chiave univoca in un database relazionale.</span><span class="sxs-lookup"><span data-stu-id="53ab8-779">The `Id` property functions as the unique key in a relational database.</span></span>

<span data-ttu-id="53ab8-780">Le classi di modelli possono essere inserite in qualsiasi punto del progetto, ma la *Models* cartella viene utilizzata per convenzione.</span><span class="sxs-lookup"><span data-stu-id="53ab8-780">Model classes can go anywhere in the project, but the *Models* folder is used by convention.</span></span>

## <a name="add-a-database-context-21"></a><span data-ttu-id="53ab8-781">Aggiungere un contesto di database 2,1</span><span class="sxs-lookup"><span data-stu-id="53ab8-781">Add a database context 2.1</span></span>

<span data-ttu-id="53ab8-782">Il *contesto di database* è la classe principale che coordina le funzionalità di Entity Framework per un modello di dati.</span><span class="sxs-lookup"><span data-stu-id="53ab8-782">The *database context* is the main class that coordinates Entity Framework functionality for a data model.</span></span> <span data-ttu-id="53ab8-783">Questa classe viene creata mediante derivazione dalla classe `Microsoft.EntityFrameworkCore.DbContext`.</span><span class="sxs-lookup"><span data-stu-id="53ab8-783">This class is created by deriving from the `Microsoft.EntityFrameworkCore.DbContext` class.</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="53ab8-784">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="53ab8-784">Visual Studio</span></span>](#tab/visual-studio)

* <span data-ttu-id="53ab8-785">Fare clic con il pulsante destro del mouse sulla *Models* cartella e scegliere **Aggiungi**  >  **classe**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-785">Right-click the *Models* folder and select **Add** > **Class**.</span></span> <span data-ttu-id="53ab8-786">Assegnare alla classe il nome *TodoContext* e fare clic su **Aggiungi**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-786">Name the class *TodoContext* and click **Add**.</span></span>

# <a name="visual-studio-code--visual-studio-for-mac"></a>[<span data-ttu-id="53ab8-787">Visual Studio Code / Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="53ab8-787">Visual Studio Code / Visual Studio for Mac</span></span>](#tab/visual-studio-code+visual-studio-mac)

* <span data-ttu-id="53ab8-788">Aggiungere una `TodoContext` classe alla *Models* cartella.</span><span class="sxs-lookup"><span data-stu-id="53ab8-788">Add a `TodoContext` class to the *Models* folder.</span></span>

---

* <span data-ttu-id="53ab8-789">Sostituire il codice del modello con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="53ab8-789">Replace the template code with the following code:</span></span>

  [!code-csharp[](first-web-api/samples/2.2/TodoApi/Models/TodoContext.cs)]

## <a name="register-the-database-context-21"></a><span data-ttu-id="53ab8-790">Registrare il contesto del database 2,1</span><span class="sxs-lookup"><span data-stu-id="53ab8-790">Register the database context 2.1</span></span>

<span data-ttu-id="53ab8-791">In ASP.NET Core i servizi come il contesto del database devono essere registrati con il [contenitore di inserimento delle dipendenze](xref:fundamentals/dependency-injection).</span><span class="sxs-lookup"><span data-stu-id="53ab8-791">In ASP.NET Core, services such as the DB context must be registered with the [dependency injection (DI)](xref:fundamentals/dependency-injection) container.</span></span> <span data-ttu-id="53ab8-792">Il contenitore rende disponibile il servizio ai controller.</span><span class="sxs-lookup"><span data-stu-id="53ab8-792">The container provides the service to controllers.</span></span>

<span data-ttu-id="53ab8-793">Aggiornare *Startup.cs* con il codice evidenziato seguente:</span><span class="sxs-lookup"><span data-stu-id="53ab8-793">Update *Startup.cs* with the following highlighted code:</span></span>

[!code-csharp[](first-web-api/samples/2.2/TodoApi/Startup1.cs?highlight=5,8,25-26&name=snippet_all)]

<span data-ttu-id="53ab8-794">Il codice precedente:</span><span class="sxs-lookup"><span data-stu-id="53ab8-794">The preceding code:</span></span>

* <span data-ttu-id="53ab8-795">Rimuove le dichiarazioni `using` inutilizzate.</span><span class="sxs-lookup"><span data-stu-id="53ab8-795">Removes unused `using` declarations.</span></span>
* <span data-ttu-id="53ab8-796">Aggiunge il contesto del database al contenitore di inserimento delle dipendenze.</span><span class="sxs-lookup"><span data-stu-id="53ab8-796">Adds the database context to the DI container.</span></span>
* <span data-ttu-id="53ab8-797">Specifica che il contesto del database userà un database in memoria.</span><span class="sxs-lookup"><span data-stu-id="53ab8-797">Specifies that the database context will use an in-memory database.</span></span>

## <a name="add-a-controller-21"></a><span data-ttu-id="53ab8-798">Aggiungere un controller 2,1</span><span class="sxs-lookup"><span data-stu-id="53ab8-798">Add a controller 2.1</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="53ab8-799">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="53ab8-799">Visual Studio</span></span>](#tab/visual-studio)

* <span data-ttu-id="53ab8-800">Fare clic con il pulsante destro del mouse sulla cartella *Controllers*.</span><span class="sxs-lookup"><span data-stu-id="53ab8-800">Right-click the *Controllers* folder.</span></span>
* <span data-ttu-id="53ab8-801">Selezionare **Aggiungi** > **nuovo elemento**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-801">Select **Add** > **New Item**.</span></span>
* <span data-ttu-id="53ab8-802">Nella finestra di dialogo **Aggiungi nuovo elemento** selezionare il modello **API Controller Class** (Classe controller API).</span><span class="sxs-lookup"><span data-stu-id="53ab8-802">In the **Add New Item** dialog, select the **API Controller Class** template.</span></span>
* <span data-ttu-id="53ab8-803">Assegnare alla classe il nome *TodoController* e selezionare **Aggiungi**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-803">Name the class *TodoController*, and select **Add**.</span></span>

  ![Finestra di dialogo Aggiungi nuovo elemento con controller nella casella di ricerca e controller API Web selezionato](first-web-api/_static/new_controller.png)

# <a name="visual-studio-code--visual-studio-for-mac"></a>[<span data-ttu-id="53ab8-805">Visual Studio Code / Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="53ab8-805">Visual Studio Code / Visual Studio for Mac</span></span>](#tab/visual-studio-code+visual-studio-mac)

* <span data-ttu-id="53ab8-806">Nella cartella *Controllers* creare una classe denominata `TodoController`.</span><span class="sxs-lookup"><span data-stu-id="53ab8-806">In the *Controllers* folder, create a class named `TodoController`.</span></span>

---

* <span data-ttu-id="53ab8-807">Sostituire il codice del modello con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="53ab8-807">Replace the template code with the following code:</span></span>

  [!code-csharp[](first-web-api/samples/2.2/TodoApi/Controllers/TodoController2.cs?name=snippet_todo1)]

<span data-ttu-id="53ab8-808">Il codice precedente:</span><span class="sxs-lookup"><span data-stu-id="53ab8-808">The preceding code:</span></span>

* <span data-ttu-id="53ab8-809">Definisce una classe controller API senza metodi.</span><span class="sxs-lookup"><span data-stu-id="53ab8-809">Defines an API controller class without methods.</span></span>
* <span data-ttu-id="53ab8-810">Contrassegna la classe con l' [`[ApiController]`](xref:Microsoft.AspNetCore.Mvc.ApiControllerAttribute) attributo.</span><span class="sxs-lookup"><span data-stu-id="53ab8-810">Marks the class with the [`[ApiController]`](xref:Microsoft.AspNetCore.Mvc.ApiControllerAttribute) attribute.</span></span> <span data-ttu-id="53ab8-811">L'attributo indica che il controller risponde alle richieste di API Web.</span><span class="sxs-lookup"><span data-stu-id="53ab8-811">This attribute indicates that the controller responds to web API requests.</span></span> <span data-ttu-id="53ab8-812">Per informazioni sui comportamenti specifici consentiti dall'attributo, vedere <xref:web-api/index>.</span><span class="sxs-lookup"><span data-stu-id="53ab8-812">For information about specific behaviors that the attribute enables, see <xref:web-api/index>.</span></span>
* <span data-ttu-id="53ab8-813">Usa l'inserimento delle dipendenze per inserire il contesto del database (`TodoContext`) nel controller.</span><span class="sxs-lookup"><span data-stu-id="53ab8-813">Uses DI to inject the database context (`TodoContext`) into the controller.</span></span> <span data-ttu-id="53ab8-814">Il contesto di database viene usato in ognuno dei metodi [CRUD](https://wikipedia.org/wiki/Create,_read,_update_and_delete) nel controller.</span><span class="sxs-lookup"><span data-stu-id="53ab8-814">The database context is used in each of the [CRUD](https://wikipedia.org/wiki/Create,_read,_update_and_delete) methods in the controller.</span></span>
* <span data-ttu-id="53ab8-815">Aggiunge un elemento denominato `Item1` al database se il database è vuoto.</span><span class="sxs-lookup"><span data-stu-id="53ab8-815">Adds an item named `Item1` to the database if the database is empty.</span></span> <span data-ttu-id="53ab8-816">Questo codice si trova nel costruttore, quindi viene eseguito ogni volta che è presente una nuova richiesta HTTP.</span><span class="sxs-lookup"><span data-stu-id="53ab8-816">This code is in the constructor, so it runs every time there's a new HTTP request.</span></span> <span data-ttu-id="53ab8-817">Se si eliminano tutti gli elementi, il costruttore crea di nuovo `Item1` quando viene nuovamente chiamato un metodo API.</span><span class="sxs-lookup"><span data-stu-id="53ab8-817">If you delete all items, the constructor creates `Item1` again the next time an API method is called.</span></span> <span data-ttu-id="53ab8-818">Potrebbe pertanto sembrare che l'eliminazione non abbia funzionato, mentre di fatto ha funzionato.</span><span class="sxs-lookup"><span data-stu-id="53ab8-818">So it may look like the deletion didn't work when it actually did work.</span></span>

## <a name="add-get-methods-21"></a><span data-ttu-id="53ab8-819">Aggiungi metodi Get 2,1</span><span class="sxs-lookup"><span data-stu-id="53ab8-819">Add Get methods 2.1</span></span>

<span data-ttu-id="53ab8-820">Per specificare un'API che recupera elementi attività, aggiungere i metodi seguenti alla classe `TodoController`:</span><span class="sxs-lookup"><span data-stu-id="53ab8-820">To provide an API that retrieves to-do items, add the following methods to the `TodoController` class:</span></span>

[!code-csharp[](first-web-api/samples/2.2/TodoApi/Controllers/TodoController.cs?name=snippet_GetAll)]

<span data-ttu-id="53ab8-821">Questi metodi implementano due metodi GET:</span><span class="sxs-lookup"><span data-stu-id="53ab8-821">These methods implement two GET endpoints:</span></span>

* `GET /api/todo`
* `GET /api/todo/{id}`

<span data-ttu-id="53ab8-822">Arrestare l'app se è ancora in esecuzione.</span><span class="sxs-lookup"><span data-stu-id="53ab8-822">Stop the app if it's still running.</span></span> <span data-ttu-id="53ab8-823">Quindi eseguirla di nuovo per includere le modifiche più recenti.</span><span class="sxs-lookup"><span data-stu-id="53ab8-823">Then run it again to include the latest changes.</span></span>

<span data-ttu-id="53ab8-824">Testare l'app chiamando i due endpoint da un browser.</span><span class="sxs-lookup"><span data-stu-id="53ab8-824">Test the app by calling the two endpoints from a browser.</span></span> <span data-ttu-id="53ab8-825">Ad esempio:</span><span class="sxs-lookup"><span data-stu-id="53ab8-825">For example:</span></span>

* `https://localhost:<port>/api/todo`
* `https://localhost:<port>/api/todo/1`

<span data-ttu-id="53ab8-826">La chiamata a `GetTodoItems` genera la risposta HTTP seguente:</span><span class="sxs-lookup"><span data-stu-id="53ab8-826">The following HTTP response is produced by the call to `GetTodoItems`:</span></span>

```json
[
  {
    "id": 1,
    "name": "Item1",
    "isComplete": false
  }
]
```

## <a name="routing-and-url-paths-21"></a><span data-ttu-id="53ab8-827">Percorsi URL e routing 2,1</span><span class="sxs-lookup"><span data-stu-id="53ab8-827">Routing and URL paths 2.1</span></span>

<span data-ttu-id="53ab8-828">L' [`[HttpGet]`](xref:Microsoft.AspNetCore.Mvc.HttpGetAttribute) attributo indica un metodo che risponde a una richiesta HTTP Get.</span><span class="sxs-lookup"><span data-stu-id="53ab8-828">The [`[HttpGet]`](xref:Microsoft.AspNetCore.Mvc.HttpGetAttribute) attribute denotes a method that responds to an HTTP GET request.</span></span> <span data-ttu-id="53ab8-829">Il percorso dell'URL per ogni metodo viene costruito nel modo seguente:</span><span class="sxs-lookup"><span data-stu-id="53ab8-829">The URL path for each method is constructed as follows:</span></span>

* <span data-ttu-id="53ab8-830">Iniziare con la stringa di modello nell'attributo `Route` del controller:</span><span class="sxs-lookup"><span data-stu-id="53ab8-830">Start with the template string in the controller's `Route` attribute:</span></span>

  [!code-csharp[](first-web-api/samples/2.2/TodoApi/Controllers/TodoController.cs?name=TodoController&highlight=3)]

* <span data-ttu-id="53ab8-831">Sostituire `[controller]` con il nome del controller, ovvero, per convenzione, il nome della classe controller meno il suffisso "Controller".</span><span class="sxs-lookup"><span data-stu-id="53ab8-831">Replace `[controller]` with the name of the controller, which by convention is the controller class name minus the "Controller" suffix.</span></span> <span data-ttu-id="53ab8-832">In questo esempio il nome della classe controller è **Todo** Controller, pertanto il nome del controller è "todo".</span><span class="sxs-lookup"><span data-stu-id="53ab8-832">For this sample, the controller class name is **Todo** Controller, so the controller name is "todo".</span></span> <span data-ttu-id="53ab8-833">Il [routing](xref:mvc/controllers/routing) ASP.NET Core non fa distinzione tra maiuscole e minuscole.</span><span class="sxs-lookup"><span data-stu-id="53ab8-833">ASP.NET Core [routing](xref:mvc/controllers/routing) is case insensitive.</span></span>
* <span data-ttu-id="53ab8-834">Se l'attributo `[HttpGet]` ha un modello di route, ad esempio `[HttpGet("products")]`, aggiungerlo al percorso.</span><span class="sxs-lookup"><span data-stu-id="53ab8-834">If the `[HttpGet]` attribute has a route template (for example, `[HttpGet("products")]`), append that to the path.</span></span> <span data-ttu-id="53ab8-835">In questo esempio non si usa un modello.</span><span class="sxs-lookup"><span data-stu-id="53ab8-835">This sample doesn't use a template.</span></span> <span data-ttu-id="53ab8-836">Per altre informazioni, vedere [Routing con attributi Http[verb]](xref:mvc/controllers/routing#attribute-routing-with-httpverb-attributes).</span><span class="sxs-lookup"><span data-stu-id="53ab8-836">For more information, see [Attribute routing with Http[Verb] attributes](xref:mvc/controllers/routing#attribute-routing-with-httpverb-attributes).</span></span>

<span data-ttu-id="53ab8-837">Nel metodo `GetTodoItem` seguente, `"{id}"` è una variabile segnaposto per l'identificatore univoco dell'elemento attività.</span><span class="sxs-lookup"><span data-stu-id="53ab8-837">In the following `GetTodoItem` method, `"{id}"` is a placeholder variable for the unique identifier of the to-do item.</span></span> <span data-ttu-id="53ab8-838">Quando viene chiamato `GetTodoItem`, il valore di `"{id}"` viene specificato per il metodo nel rispettivo parametro `id`.</span><span class="sxs-lookup"><span data-stu-id="53ab8-838">When `GetTodoItem` is invoked, the value of `"{id}"` in the URL is provided to the method in its`id` parameter.</span></span>

[!code-csharp[](first-web-api/samples/2.2/TodoApi/Controllers/TodoController.cs?name=snippet_GetByID&highlight=1-2)]

## <a name="return-values-21"></a><span data-ttu-id="53ab8-839">Valori restituiti 2,1</span><span class="sxs-lookup"><span data-stu-id="53ab8-839">Return values 2.1</span></span>

<span data-ttu-id="53ab8-840">Il tipo restituito dei `GetTodoItems` metodi e `GetTodoItem` è [ActionResult. \<T> ](xref:web-api/action-return-types#actionresultt-type)</span><span class="sxs-lookup"><span data-stu-id="53ab8-840">The return type of the `GetTodoItems` and `GetTodoItem` methods is [ActionResult\<T> type](xref:web-api/action-return-types#actionresultt-type).</span></span> <span data-ttu-id="53ab8-841">ASP.NET Core serializza automaticamente l'oggetto su [JSON](https://www.json.org/) e scrive il codice JSON nel corpo del messaggio di risposta.</span><span class="sxs-lookup"><span data-stu-id="53ab8-841">ASP.NET Core automatically serializes the object to [JSON](https://www.json.org/) and writes the JSON into the body of the response message.</span></span> <span data-ttu-id="53ab8-842">Il codice di risposta per questo tipo restituito è 200, presupponendo che non esistano eccezioni non gestite.</span><span class="sxs-lookup"><span data-stu-id="53ab8-842">The response code for this return type is 200, assuming there are no unhandled exceptions.</span></span> <span data-ttu-id="53ab8-843">Le eccezioni non gestite vengono convertite in errori 5xx.</span><span class="sxs-lookup"><span data-stu-id="53ab8-843">Unhandled exceptions are translated into 5xx errors.</span></span>

<span data-ttu-id="53ab8-844">I tipi restituiti `ActionResult` possono rappresentare un ampio intervallo di codici di stato HTTP.</span><span class="sxs-lookup"><span data-stu-id="53ab8-844">`ActionResult` return types can represent a wide range of HTTP status codes.</span></span> <span data-ttu-id="53ab8-845">Ad esempio, `GetTodoItem` può restituire due valori di stato diversi:</span><span class="sxs-lookup"><span data-stu-id="53ab8-845">For example, `GetTodoItem` can return two different status values:</span></span>

* <span data-ttu-id="53ab8-846">Se nessun elemento corrisponde all'ID richiesto, il metodo restituisce un <xref:Microsoft.AspNetCore.Mvc.ControllerBase.NotFound%2A> codice di errore 404.</span><span class="sxs-lookup"><span data-stu-id="53ab8-846">If no item matches the requested ID, the method returns a 404 <xref:Microsoft.AspNetCore.Mvc.ControllerBase.NotFound%2A> error code.</span></span>
* <span data-ttu-id="53ab8-847">In caso contrario, il metodo restituisce il codice 200 con un corpo della risposta JSON.</span><span class="sxs-lookup"><span data-stu-id="53ab8-847">Otherwise, the method returns 200 with a JSON response body.</span></span> <span data-ttu-id="53ab8-848">La restituzione di `item` risulta in una risposta HTTP 200.</span><span class="sxs-lookup"><span data-stu-id="53ab8-848">Returning `item` results in an HTTP 200 response.</span></span>

## <a name="test-the-gettodoitems-method-21"></a><span data-ttu-id="53ab8-849">Testare il metodo GetTodoItems 2,1</span><span class="sxs-lookup"><span data-stu-id="53ab8-849">Test the GetTodoItems method 2.1</span></span>

<span data-ttu-id="53ab8-850">Questa esercitazione usa Postman per testare l'API Web.</span><span class="sxs-lookup"><span data-stu-id="53ab8-850">This tutorial uses Postman to test the web API.</span></span>

* <span data-ttu-id="53ab8-851">Installare [Postman](https://www.getpostman.com/downloads/).</span><span class="sxs-lookup"><span data-stu-id="53ab8-851">Install [Postman](https://www.getpostman.com/downloads/).</span></span>
* <span data-ttu-id="53ab8-852">Avviare l'app Web.</span><span class="sxs-lookup"><span data-stu-id="53ab8-852">Start the web app.</span></span>
* <span data-ttu-id="53ab8-853">Avviare Postman.</span><span class="sxs-lookup"><span data-stu-id="53ab8-853">Start Postman.</span></span>
* <span data-ttu-id="53ab8-854">Disabilitare la **Verifica del certificato SSL**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-854">Disable **SSL certificate verification**.</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="53ab8-855">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="53ab8-855">Visual Studio</span></span>](#tab/visual-studio)

* <span data-ttu-id="53ab8-856">Da **File** > **Settings** (Impostazioni) (scheda **General** (Generale)) disabilitare **SSL certificate verification** (Verifica certificato SSL).</span><span class="sxs-lookup"><span data-stu-id="53ab8-856">From **File** > **Settings** (**General** tab), disable **SSL certificate verification**.</span></span>

# <a name="visual-studio-code--visual-studio-for-mac"></a>[<span data-ttu-id="53ab8-857">Visual Studio Code / Visual Studio per Mac</span><span class="sxs-lookup"><span data-stu-id="53ab8-857">Visual Studio Code / Visual Studio for Mac</span></span>](#tab/visual-studio-code+visual-studio-mac)

* <span data-ttu-id="53ab8-858">Dalle preferenze del **poster**  >   (scheda **generale** ) disabilitare la **Verifica del certificato SSL**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-858">From **Postman** > **Preferences** (**General** tab), disable **SSL certificate verification**.</span></span> <span data-ttu-id="53ab8-859">In alternativa, selezionare la chiave inglese e selezionare **Settings** (Impostazioni), quindi disabilitare la verifica del certificato SSL.</span><span class="sxs-lookup"><span data-stu-id="53ab8-859">Alternatively, select the wrench and select **Settings**, then disable the SSL certificate verification.</span></span>

---
  
> [!WARNING]
> <span data-ttu-id="53ab8-860">Riattivare la verifica dei certificati SSL al termine del test del controller.</span><span class="sxs-lookup"><span data-stu-id="53ab8-860">Re-enable SSL certificate verification after testing the controller.</span></span>

* <span data-ttu-id="53ab8-861">Creare una nuova richiesta.</span><span class="sxs-lookup"><span data-stu-id="53ab8-861">Create a new request.</span></span>
  * <span data-ttu-id="53ab8-862">Impostare il metodo HTTP su **GET**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-862">Set the HTTP method to **GET**.</span></span>
  * <span data-ttu-id="53ab8-863">Impostare l'URI della richiesta su `https://localhost:<port>/api/todo` .</span><span class="sxs-lookup"><span data-stu-id="53ab8-863">Set the request URI to `https://localhost:<port>/api/todo`.</span></span> <span data-ttu-id="53ab8-864">Ad esempio: `https://localhost:5001/api/todo`.</span><span class="sxs-lookup"><span data-stu-id="53ab8-864">For example, `https://localhost:5001/api/todo`.</span></span>
* <span data-ttu-id="53ab8-865">Impostare **Two pane view** (Visualizzazione in due riquadri) in Postman.</span><span class="sxs-lookup"><span data-stu-id="53ab8-865">Set **Two pane view** in Postman.</span></span>
* <span data-ttu-id="53ab8-866">Selezionare **Invia**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-866">Select **Send**.</span></span>

![Postman con richiesta GET](first-web-api/_static/2pv.png)

## <a name="add-a-create-method-21"></a><span data-ttu-id="53ab8-868">Aggiungere un metodo Create 2,1</span><span class="sxs-lookup"><span data-stu-id="53ab8-868">Add a Create method 2.1</span></span>

<span data-ttu-id="53ab8-869">Aggiungere il metodo `PostTodoItem` seguente in *Controllers/TodoController.cs*:</span><span class="sxs-lookup"><span data-stu-id="53ab8-869">Add the following `PostTodoItem` method inside of *Controllers/TodoController.cs*:</span></span> 

[!code-csharp[](first-web-api/samples/2.2/TodoApi/Controllers/TodoController.cs?name=snippet_Create)]

<span data-ttu-id="53ab8-870">Il codice precedente è un metodo HTTP POST, come indicato dall' [`[HttpPost]`](xref:Microsoft.AspNetCore.Mvc.HttpPostAttribute) attributo.</span><span class="sxs-lookup"><span data-stu-id="53ab8-870">The preceding code is an HTTP POST method, as indicated by the [`[HttpPost]`](xref:Microsoft.AspNetCore.Mvc.HttpPostAttribute) attribute.</span></span> <span data-ttu-id="53ab8-871">Il metodo ottiene il valore dell'elemento attività dal corpo della richiesta HTTP.</span><span class="sxs-lookup"><span data-stu-id="53ab8-871">The method gets the value of the to-do item from the body of the HTTP request.</span></span>

<span data-ttu-id="53ab8-872">Il metodo `CreatedAtAction`:</span><span class="sxs-lookup"><span data-stu-id="53ab8-872">The `CreatedAtAction` method:</span></span>

* <span data-ttu-id="53ab8-873">Restituisce un codice di stato HTTP 201 in caso di esito positivo.</span><span class="sxs-lookup"><span data-stu-id="53ab8-873">Returns an HTTP 201 status code, if successful.</span></span> <span data-ttu-id="53ab8-874">HTTP 201 è la risposta standard per un metodo HTTP POST che crea una nuova risorsa nel server.</span><span class="sxs-lookup"><span data-stu-id="53ab8-874">HTTP 201 is the standard response for an HTTP POST method that creates a new resource on the server.</span></span>
* <span data-ttu-id="53ab8-875">Aggiunge un'intestazione `Location` alla risposta.</span><span class="sxs-lookup"><span data-stu-id="53ab8-875">Adds a `Location` header to the response.</span></span> <span data-ttu-id="53ab8-876">L'intestazione `Location` specifica l'URI dell'elemento attività appena creato.</span><span class="sxs-lookup"><span data-stu-id="53ab8-876">The `Location` header specifies the URI of the newly created to-do item.</span></span> <span data-ttu-id="53ab8-877">Per altre informazioni, vedere [10.2.2 201 Created](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html).</span><span class="sxs-lookup"><span data-stu-id="53ab8-877">For more information, see [10.2.2 201 Created](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html).</span></span>
* <span data-ttu-id="53ab8-878">Fa riferimento all'azione `GetTodoItem` per creare l'URI dell'intestazione `Location`.</span><span class="sxs-lookup"><span data-stu-id="53ab8-878">References the `GetTodoItem` action to create the `Location` header's URI.</span></span> <span data-ttu-id="53ab8-879">La parola chiave `nameof` C# viene usata per evitare di impostare il nome dell'azione come hardcoded nella chiamata a `CreatedAtAction`.</span><span class="sxs-lookup"><span data-stu-id="53ab8-879">The C# `nameof` keyword is used to avoid hard-coding the action name in the `CreatedAtAction` call.</span></span>

  [!code-csharp[](first-web-api/samples/2.2/TodoApi/Controllers/TodoController.cs?name=snippet_GetByID&highlight=1-2)]

### <a name="test-the-posttodoitem-method-21"></a><span data-ttu-id="53ab8-880">Testare il metodo PostTodoItem 2,1</span><span class="sxs-lookup"><span data-stu-id="53ab8-880">Test the PostTodoItem method 2.1</span></span>

* <span data-ttu-id="53ab8-881">Compilare il progetto.</span><span class="sxs-lookup"><span data-stu-id="53ab8-881">Build the project.</span></span>
* <span data-ttu-id="53ab8-882">In Postman impostare il metodo HTTP su `POST`.</span><span class="sxs-lookup"><span data-stu-id="53ab8-882">In Postman, set the HTTP method to `POST`.</span></span>
* <span data-ttu-id="53ab8-883">Impostare l'URI su `https://localhost:<port>/api/Todo` .</span><span class="sxs-lookup"><span data-stu-id="53ab8-883">Set the URI to `https://localhost:<port>/api/Todo`.</span></span> <span data-ttu-id="53ab8-884">Ad esempio: `https://localhost:5001/api/Todo`.</span><span class="sxs-lookup"><span data-stu-id="53ab8-884">For example, `https://localhost:5001/api/Todo`.</span></span>
* <span data-ttu-id="53ab8-885">Selezionare la scheda **Corpo**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-885">Select the **Body** tab.</span></span>
* <span data-ttu-id="53ab8-886">Selezionare il pulsante di opzione **raw** (non elaborato).</span><span class="sxs-lookup"><span data-stu-id="53ab8-886">Select the **raw** radio button.</span></span>
* <span data-ttu-id="53ab8-887">Impostare il tipo su **JSON (application/json)**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-887">Set the type to **JSON (application/json)**.</span></span>
* <span data-ttu-id="53ab8-888">Nel corpo della richiesta immettere codice JSON per un elemento attività:</span><span class="sxs-lookup"><span data-stu-id="53ab8-888">In the request body enter JSON for a to-do item:</span></span>

    ```json
    {
      "name":"walk dog",
      "isComplete":true
    }
    ```

* <span data-ttu-id="53ab8-889">Selezionare **Invia**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-889">Select **Send**.</span></span>

  ![Postman con richiesta di creazione](first-web-api/_static/create.png)

  <span data-ttu-id="53ab8-891">Se viene visualizzato un errore HTTP 405 - Metodo non consentito, è probabile che il progetto non sia stato compilato dopo l'aggiunta del metodo `PostTodoItem`.</span><span class="sxs-lookup"><span data-stu-id="53ab8-891">If you get a 405 Method Not Allowed error, it's probably the result of not compiling the project after adding the `PostTodoItem` method.</span></span>

### <a name="test-the-location-header-uri-21"></a><span data-ttu-id="53ab8-892">Testare l'URI dell'intestazione Location 2,1</span><span class="sxs-lookup"><span data-stu-id="53ab8-892">Test the location header URI 2.1</span></span>

* <span data-ttu-id="53ab8-893">Selezionare la scheda **Headers** (Intestazioni) nel riquadro **Response** (Risposta).</span><span class="sxs-lookup"><span data-stu-id="53ab8-893">Select the **Headers** tab in the **Response** pane.</span></span>
* <span data-ttu-id="53ab8-894">Copiare il valore dell'intestazione **Location** (Posizione):</span><span class="sxs-lookup"><span data-stu-id="53ab8-894">Copy the **Location** header value:</span></span>

  ![Scheda Headers (Intestazioni) della console Postman](first-web-api/_static/pmc2.png)

* <span data-ttu-id="53ab8-896">Impostare il metodo su GET.</span><span class="sxs-lookup"><span data-stu-id="53ab8-896">Set the method to GET.</span></span>
* <span data-ttu-id="53ab8-897">Impostare l'URI su `https://localhost:<port>/api/TodoItems/2` .</span><span class="sxs-lookup"><span data-stu-id="53ab8-897">Set the URI to `https://localhost:<port>/api/TodoItems/2`.</span></span> <span data-ttu-id="53ab8-898">Ad esempio: `https://localhost:5001/api/TodoItems/2`.</span><span class="sxs-lookup"><span data-stu-id="53ab8-898">For example, `https://localhost:5001/api/TodoItems/2`.</span></span>
* <span data-ttu-id="53ab8-899">Selezionare **Invia**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-899">Select **Send**.</span></span>

## <a name="add-a-puttodoitem-method-21"></a><span data-ttu-id="53ab8-900">Aggiungere un metodo PutTodoItem 2,1</span><span class="sxs-lookup"><span data-stu-id="53ab8-900">Add a PutTodoItem method 2.1</span></span>

<span data-ttu-id="53ab8-901">Aggiungere il metodo `PutTodoItem` seguente:</span><span class="sxs-lookup"><span data-stu-id="53ab8-901">Add the following `PutTodoItem` method:</span></span>

[!code-csharp[](first-web-api/samples/2.2/TodoApi/Controllers/TodoController.cs?name=snippet_Update)]

<span data-ttu-id="53ab8-902">`PutTodoItem` è simile a `PostTodoItem` ma usa la richiesta HTTP PUT.</span><span class="sxs-lookup"><span data-stu-id="53ab8-902">`PutTodoItem` is similar to `PostTodoItem`, except it uses HTTP PUT.</span></span> <span data-ttu-id="53ab8-903">La risposta è [204 (No Content)](https://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html).</span><span class="sxs-lookup"><span data-stu-id="53ab8-903">The response is [204 (No Content)](https://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html).</span></span> <span data-ttu-id="53ab8-904">In base alla specifica HTTP, una richiesta PUT richiede che il client invii l'intera entità aggiornata e non solo le modifiche.</span><span class="sxs-lookup"><span data-stu-id="53ab8-904">According to the HTTP specification, a PUT request requires the client to send the entire updated entity, not just the changes.</span></span> <span data-ttu-id="53ab8-905">Per supportare gli aggiornamenti parziali, usare [HTTP PATCH](xref:Microsoft.AspNetCore.Mvc.HttpPatchAttribute).</span><span class="sxs-lookup"><span data-stu-id="53ab8-905">To support partial updates, use [HTTP PATCH](xref:Microsoft.AspNetCore.Mvc.HttpPatchAttribute).</span></span>

<span data-ttu-id="53ab8-906">Se si riceve un errore chiamando `PutTodoItem`, chiamare `GET` per verificare che il database includa un elemento.</span><span class="sxs-lookup"><span data-stu-id="53ab8-906">If you get an error calling `PutTodoItem`, call `GET` to ensure there's an item in the database.</span></span>

### <a name="test-the-puttodoitem-method-21"></a><span data-ttu-id="53ab8-907">Testare il metodo PutTodoItem 2,1</span><span class="sxs-lookup"><span data-stu-id="53ab8-907">Test the PutTodoItem method 2.1</span></span>

<span data-ttu-id="53ab8-908">Questo esempio usa un database in memoria che deve essere inizializzato ogni volta che l'app viene avviata.</span><span class="sxs-lookup"><span data-stu-id="53ab8-908">This sample uses an in-memory database that must be initialized each time the app is started.</span></span> <span data-ttu-id="53ab8-909">Deve esistere un elemento nel database prima di eseguire una chiamata PUT.</span><span class="sxs-lookup"><span data-stu-id="53ab8-909">There must be an item in the database before you make a PUT call.</span></span> <span data-ttu-id="53ab8-910">Chiamare GET per assicurarsi che sia presente un elemento nel database prima di effettuare una chiamata PUT.</span><span class="sxs-lookup"><span data-stu-id="53ab8-910">Call GET to ensure there's an item in the database before making a PUT call.</span></span>

<span data-ttu-id="53ab8-911">Aggiornare l'attività con ID = 1 e impostarne il nome su "feed Fish":</span><span class="sxs-lookup"><span data-stu-id="53ab8-911">Update the to-do item that has Id = 1 and set its name to "feed fish":</span></span>

```json
  {
    "id":1,
    "name":"feed fish",
    "isComplete":true
  }
```

<span data-ttu-id="53ab8-912">L'immagine seguente visualizza l'aggiornamento di Postman:</span><span class="sxs-lookup"><span data-stu-id="53ab8-912">The following image shows the Postman update:</span></span>

![Console Postman con risposta 204 (No Content)](first-web-api/_static/pmcput.png)

## <a name="add-a-deletetodoitem-method-21"></a><span data-ttu-id="53ab8-914">Aggiungere un metodo DeleteTodoItem 2,1</span><span class="sxs-lookup"><span data-stu-id="53ab8-914">Add a DeleteTodoItem method 2.1</span></span>

<span data-ttu-id="53ab8-915">Aggiungere il metodo `DeleteTodoItem` seguente:</span><span class="sxs-lookup"><span data-stu-id="53ab8-915">Add the following `DeleteTodoItem` method:</span></span>

[!code-csharp[](first-web-api/samples/2.2/TodoApi/Controllers/TodoController.cs?name=snippet_Delete)]

<span data-ttu-id="53ab8-916">La `DeleteTodoItem` risposta è [204 (nessun contenuto)](https://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html).</span><span class="sxs-lookup"><span data-stu-id="53ab8-916">The `DeleteTodoItem` response is [204 (No Content)](https://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html).</span></span>

### <a name="test-the-deletetodoitem-method-21"></a><span data-ttu-id="53ab8-917">Testare il metodo DeleteTodoItem 2,1</span><span class="sxs-lookup"><span data-stu-id="53ab8-917">Test the DeleteTodoItem method 2.1</span></span>

<span data-ttu-id="53ab8-918">Usare Postman per eliminare un elemento attività:</span><span class="sxs-lookup"><span data-stu-id="53ab8-918">Use Postman to delete a to-do item:</span></span>

* <span data-ttu-id="53ab8-919">Impostare il metodo su `DELETE`.</span><span class="sxs-lookup"><span data-stu-id="53ab8-919">Set the method to `DELETE`.</span></span>
* <span data-ttu-id="53ab8-920">Impostare l'URI dell'oggetto da eliminare (ad esempio, `https://localhost:5001/api/todo/1` ).</span><span class="sxs-lookup"><span data-stu-id="53ab8-920">Set the URI of the object to delete (for example, `https://localhost:5001/api/todo/1`).</span></span>
* <span data-ttu-id="53ab8-921">Selezionare **Invia**.</span><span class="sxs-lookup"><span data-stu-id="53ab8-921">Select **Send**.</span></span>

<span data-ttu-id="53ab8-922">L'app di esempio consente di eliminare tutti gli elementi.</span><span class="sxs-lookup"><span data-stu-id="53ab8-922">The sample app allows you to delete all the items.</span></span> <span data-ttu-id="53ab8-923">Quando viene eliminato l'ultimo elemento, tuttavia, ne viene creato uno nuovo dal costruttore della classe modello alla successiva chiamata dell'API.</span><span class="sxs-lookup"><span data-stu-id="53ab8-923">However, when the last item is deleted, a new one is created by the model class constructor the next time the API is called.</span></span>

## <a name="call-the-web-api-with-javascript-21"></a><span data-ttu-id="53ab8-924">Chiamare l'API Web con JavaScript 2,1</span><span class="sxs-lookup"><span data-stu-id="53ab8-924">Call the web API with JavaScript 2.1</span></span>

<span data-ttu-id="53ab8-925">In questa sezione viene aggiunta una pagina HTML che usa JavaScript per chiamare l'API Web.</span><span class="sxs-lookup"><span data-stu-id="53ab8-925">In this section, an HTML page is added that uses JavaScript to call the web API.</span></span> <span data-ttu-id="53ab8-926">jQuery avvia la richiesta.</span><span class="sxs-lookup"><span data-stu-id="53ab8-926">jQuery initiates the request.</span></span> <span data-ttu-id="53ab8-927">JavaScript aggiorna la pagina con i dettagli della risposta dell'API Web.</span><span class="sxs-lookup"><span data-stu-id="53ab8-927">JavaScript updates the page with the details from the web API's response.</span></span>

<span data-ttu-id="53ab8-928">Configurare l'app per [servire file statici](xref:Microsoft.AspNetCore.Builder.StaticFileExtensions.UseStaticFiles%2A) e [abilitare il mapping del file predefinito](xref:Microsoft.AspNetCore.Builder.DefaultFilesExtensions.UseDefaultFiles%2A) aggiornando *Startup.cs* con il codice evidenziato seguente:</span><span class="sxs-lookup"><span data-stu-id="53ab8-928">Configure the app to [serve static files](xref:Microsoft.AspNetCore.Builder.StaticFileExtensions.UseStaticFiles%2A) and [enable default file mapping](xref:Microsoft.AspNetCore.Builder.DefaultFilesExtensions.UseDefaultFiles%2A) by updating *Startup.cs* with the following highlighted code:</span></span>

[!code-csharp[](first-web-api/samples/2.2/TodoApi/Startup.cs?highlight=14-15&name=snippet_configure)]

<span data-ttu-id="53ab8-929">Creare una cartella *wwwroot* nella directory del progetto.</span><span class="sxs-lookup"><span data-stu-id="53ab8-929">Create a *wwwroot* folder in the project directory.</span></span>

<span data-ttu-id="53ab8-930">Aggiungere un file HTML denominato *index.html* alla directory *wwwroot*.</span><span class="sxs-lookup"><span data-stu-id="53ab8-930">Add an HTML file named *index.html* to the *wwwroot* directory.</span></span> <span data-ttu-id="53ab8-931">Sostituire il contenuto con il markup seguente:</span><span class="sxs-lookup"><span data-stu-id="53ab8-931">Replace its contents with the following markup:</span></span>

[!code-html[](first-web-api/samples/2.2/TodoApi/wwwroot/index.html)]

<span data-ttu-id="53ab8-932">Aggiungere un file JavaScript con nome *site.js* alla directory *wwwroot*.</span><span class="sxs-lookup"><span data-stu-id="53ab8-932">Add a JavaScript file named *site.js* to the *wwwroot* directory.</span></span> <span data-ttu-id="53ab8-933">Sostituirne il contenuto con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="53ab8-933">Replace its contents with the following code:</span></span>

[!code-javascript[](first-web-api/samples/2.2/TodoApi/wwwroot/site.js?name=snippet_SiteJs)]

<span data-ttu-id="53ab8-934">Può essere necessario modificare le impostazioni di avvio del progetto ASP.NET Core per il test della pagina HTML in locale:</span><span class="sxs-lookup"><span data-stu-id="53ab8-934">A change to the ASP.NET Core project's launch settings may be required to test the HTML page locally:</span></span>

* <span data-ttu-id="53ab8-935">Aprire *Properties\launchSettings.json*.</span><span class="sxs-lookup"><span data-stu-id="53ab8-935">Open *Properties\launchSettings.json*.</span></span>
* <span data-ttu-id="53ab8-936">Rimuovere la `launchUrl` proprietà per forzare l'apertura dell'app a *index.html* &mdash; file predefinito del progetto.</span><span class="sxs-lookup"><span data-stu-id="53ab8-936">Remove the `launchUrl` property to force the app to open at *index.html*&mdash;the project's default file.</span></span>

<span data-ttu-id="53ab8-937">In questo esempio vengono chiamati tutti i metodi CRUD dell'API Web.</span><span class="sxs-lookup"><span data-stu-id="53ab8-937">This sample calls all of the CRUD methods of the web API.</span></span> <span data-ttu-id="53ab8-938">Di seguito sono incluse le spiegazioni delle chiamate all'API.</span><span class="sxs-lookup"><span data-stu-id="53ab8-938">Following are explanations of the calls to the API.</span></span>

### <a name="get-a-list-of-to-do-items-21"></a><span data-ttu-id="53ab8-939">Ottenere un elenco di elementi attività 2,1</span><span class="sxs-lookup"><span data-stu-id="53ab8-939">Get a list of to-do items 2.1</span></span>

<span data-ttu-id="53ab8-940">jQuery invia una richiesta HTTP GET all'API Web, che restituisce JSON che rappresenta una matrice di elementi attività.</span><span class="sxs-lookup"><span data-stu-id="53ab8-940">jQuery sends an HTTP GET request to the web API, which returns JSON representing an array of to-do items.</span></span> <span data-ttu-id="53ab8-941">La funzione di callback `success` viene chiamata se la richiesta ha esito positivo.</span><span class="sxs-lookup"><span data-stu-id="53ab8-941">The `success` callback function is invoked if the request succeeds.</span></span> <span data-ttu-id="53ab8-942">Nel callback il modello DOM viene aggiornato con le informazioni dell'elemento attività.</span><span class="sxs-lookup"><span data-stu-id="53ab8-942">In the callback, the DOM is updated with the to-do information.</span></span>

[!code-javascript[](first-web-api/samples/2.2/TodoApi/wwwroot/site.js?name=snippet_GetData)]

### <a name="add-a-to-do-item-21"></a><span data-ttu-id="53ab8-943">Aggiungere un'attività 2,1</span><span class="sxs-lookup"><span data-stu-id="53ab8-943">Add a to-do item 2.1</span></span>

<span data-ttu-id="53ab8-944">jQuery invia una richiesta HTTP POST con l'elemento attività nel corpo della richiesta.</span><span class="sxs-lookup"><span data-stu-id="53ab8-944">jQuery sends an HTTP POST request with the to-do item in the request body.</span></span> <span data-ttu-id="53ab8-945">Le opzioni `accepts` e `contentType` sono impostate su `application/json` per specificare il tipo di supporto ricevuto e inviato.</span><span class="sxs-lookup"><span data-stu-id="53ab8-945">The `accepts` and `contentType` options are set to `application/json` to specify the media type being received and sent.</span></span> <span data-ttu-id="53ab8-946">L'elemento attività viene convertito in JSON usando [JSON.stringify](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify).</span><span class="sxs-lookup"><span data-stu-id="53ab8-946">The to-do item is converted to JSON by using [JSON.stringify](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify).</span></span> <span data-ttu-id="53ab8-947">Quando l'API restituisce un codice di stato di esecuzione riuscita, viene chiamata la funzione `getData` per aggiornare la tabella HTML.</span><span class="sxs-lookup"><span data-stu-id="53ab8-947">When the API returns a successful status code, the `getData` function is invoked to update the HTML table.</span></span>

[!code-javascript[](first-web-api/samples/2.2/TodoApi/wwwroot/site.js?name=snippet_AddItem)]

### <a name="update-a-to-do-item-21"></a><span data-ttu-id="53ab8-948">Aggiornare un'attività 2,1</span><span class="sxs-lookup"><span data-stu-id="53ab8-948">Update a to-do item 2.1</span></span>

<span data-ttu-id="53ab8-949">L'aggiornamento di un elemento attività è simile all'aggiunta di un elemento di questo tipo.</span><span class="sxs-lookup"><span data-stu-id="53ab8-949">Updating a to-do item is similar to adding one.</span></span> <span data-ttu-id="53ab8-950">L'`url` cambia per includere l'identificatore univoco dell'elemento e `type` è `PUT`.</span><span class="sxs-lookup"><span data-stu-id="53ab8-950">The `url` changes to add the unique identifier of the item, and the `type` is `PUT`.</span></span>

[!code-javascript[](first-web-api/samples/2.2/TodoApi/wwwroot/site.js?name=snippet_AjaxPut)]

### <a name="delete-a-to-do-item-21"></a><span data-ttu-id="53ab8-951">Eliminare un'attività 2,1</span><span class="sxs-lookup"><span data-stu-id="53ab8-951">Delete a to-do item 2.1</span></span>

<span data-ttu-id="53ab8-952">È possibile eliminare un elemento attività impostando `type` su `DELETE` nella chiamata AJAX e specificando l'identificatore univoco dell'elemento nell'URL.</span><span class="sxs-lookup"><span data-stu-id="53ab8-952">Deleting a to-do item is accomplished by setting the `type` on the AJAX call to `DELETE` and specifying the item's unique identifier in the URL.</span></span>

[!code-javascript[](first-web-api/samples/2.2/TodoApi/wwwroot/site.js?name=snippet_AjaxDelete)]

::: moniker-end

<a name="auth"></a>

## <a name="add-authentication-support-to-a-web-api-21"></a><span data-ttu-id="53ab8-953">Aggiungere il supporto per l'autenticazione a un'API Web 2,1</span><span class="sxs-lookup"><span data-stu-id="53ab8-953">Add authentication support to a web API 2.1</span></span>

[!INCLUDE[](~/includes/IdentityServer4.md)]

## <a name="additional-resources-21"></a><span data-ttu-id="53ab8-954">Risorse aggiuntive 2,1</span><span class="sxs-lookup"><span data-stu-id="53ab8-954">Additional resources 2.1</span></span>

<span data-ttu-id="53ab8-955">[Visualizzare o scaricare il codice di esempio per questa esercitazione](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/tutorials/first-web-api/samples).</span><span class="sxs-lookup"><span data-stu-id="53ab8-955">[View or download sample code for this tutorial](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/tutorials/first-web-api/samples).</span></span> <span data-ttu-id="53ab8-956">Vedere [come scaricare un esempio](xref:index#how-to-download-a-sample).</span><span class="sxs-lookup"><span data-stu-id="53ab8-956">See [how to download](xref:index#how-to-download-a-sample).</span></span>

<span data-ttu-id="53ab8-957">Per altre informazioni, vedere le risorse seguenti:</span><span class="sxs-lookup"><span data-stu-id="53ab8-957">For more information, see the following resources:</span></span>

* <xref:web-api/index>
* <xref:tutorials/web-api-help-pages-using-swagger>
* <xref:data/ef-rp/intro>
* <xref:mvc/controllers/routing>
* <xref:web-api/action-return-types>
* <xref:host-and-deploy/azure-apps/index>
* <xref:host-and-deploy/index>
* [<span data-ttu-id="53ab8-958">Versione YouTube dell'esercitazione</span><span class="sxs-lookup"><span data-stu-id="53ab8-958">YouTube version of this tutorial</span></span>](https://www.youtube.com/watch?v=TTkhEyGBfAk)
