---
title: Pubblicare un'API Web di ASP.NET Core in gestione API di Azure con Visual Studio
author: codemillmatt
description: Informazioni su come pubblicare un'API Web ASP.NET Core in gestione API di Azure con Visual Studio.
ms.author: masoucou
ms.custom: devx-track-csharp, mvc
ms.date: 11/22/2020
uid: tutorials/publish-to-azure-api-management-using-vs
ms.openlocfilehash: ddff54bbd146c98cf83a865910401df26e7ac4ec
ms.sourcegitcommit: 7e394a8527c9818caebb940f692ae4fcf2f1b277
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 01/31/2021
ms.locfileid: "99217583"
---
# <a name="publish-an-aspnet-core-web-api-to-azure-api-management-with-visual-studio"></a><span data-ttu-id="2d0f8-103">Pubblicare un'API Web di ASP.NET Core in gestione API di Azure con Visual Studio</span><span class="sxs-lookup"><span data-stu-id="2d0f8-103">Publish an ASP.NET Core web API to Azure API Management with Visual Studio</span></span>

<span data-ttu-id="2d0f8-104">Di [Matt Soucoup](https://twitter.com/codemillmatt)</span><span class="sxs-lookup"><span data-stu-id="2d0f8-104">By [Matt Soucoup](https://twitter.com/codemillmatt)</span></span>

## <a name="set-up"></a><span data-ttu-id="2d0f8-105">Configurazione</span><span class="sxs-lookup"><span data-stu-id="2d0f8-105">Set up</span></span>

- <span data-ttu-id="2d0f8-106">Aprire un [account Azure gratuito](https://azure.microsoft.com/free/dotnet/) se non è già disponibile un account.</span><span class="sxs-lookup"><span data-stu-id="2d0f8-106">Open a [free Azure account](https://azure.microsoft.com/free/dotnet/) if you don't have one.</span></span>
- <span data-ttu-id="2d0f8-107">Se non è già stato fatto, [creare una nuova istanza di gestione API di Azure](/azure/api-management/get-started-create-service-instance) .</span><span class="sxs-lookup"><span data-stu-id="2d0f8-107">[Create a new Azure API Management instance](/azure/api-management/get-started-create-service-instance) if you haven't already.</span></span>
- <span data-ttu-id="2d0f8-108">[Creare un progetto di app per le API Web](xref:tutorials/first-web-api#create-a-web-project).</span><span class="sxs-lookup"><span data-stu-id="2d0f8-108">[Create a web API app project](xref:tutorials/first-web-api#create-a-web-project).</span></span>

## <a name="configure-the-app"></a><span data-ttu-id="2d0f8-109">Configurare l'app</span><span class="sxs-lookup"><span data-stu-id="2d0f8-109">Configure the app</span></span>

<span data-ttu-id="2d0f8-110">L'aggiunta di definizioni di spavalderia all'API Web ASP.NET Core consente a gestione API di Azure di leggere le definizioni delle API dell'app.</span><span class="sxs-lookup"><span data-stu-id="2d0f8-110">Adding Swagger definitions to the ASP.NET Core web API allows Azure API Management to read the app's API definitions.</span></span> <span data-ttu-id="2d0f8-111">Completare questi passaggi.</span><span class="sxs-lookup"><span data-stu-id="2d0f8-111">Complete the following steps.</span></span>

### <a name="add-swagger"></a><span data-ttu-id="2d0f8-112">Aggiungi spavalderia</span><span class="sxs-lookup"><span data-stu-id="2d0f8-112">Add Swagger</span></span>

1. <span data-ttu-id="2d0f8-113">Aggiungere il pacchetto NuGet [Swashbuckle. AspNetCore](https://www.nuget.org/packages/Swashbuckle.AspNetCore) al progetto API Web ASP.NET Core:</span><span class="sxs-lookup"><span data-stu-id="2d0f8-113">Add the [Swashbuckle.AspNetCore](https://www.nuget.org/packages/Swashbuckle.AspNetCore) NuGet package to the ASP.NET Core web API project:</span></span>

    ![Schermata per configurare la finestra di dialogo NuGet](publish-to-azure-api-management-using-vs/_static/configure_nuget.png)

1. <span data-ttu-id="2d0f8-115">Aggiungere la riga seguente al metodo `Startup.ConfigureServices`:</span><span class="sxs-lookup"><span data-stu-id="2d0f8-115">Add the following line to the `Startup.ConfigureServices` method:</span></span>
    
    ```csharp
    services.AddSwaggerGen();
    ```
    
1. <span data-ttu-id="2d0f8-116">Aggiungere la riga seguente al metodo `Startup.Configure`:</span><span class="sxs-lookup"><span data-stu-id="2d0f8-116">Add the following line to the `Startup.Configure` method:</span></span>

    ```csharp
    app.UseSwagger();
    ```

### <a name="change-the-api-routing"></a><span data-ttu-id="2d0f8-117">Modificare il routing dell'API</span><span class="sxs-lookup"><span data-stu-id="2d0f8-117">Change the API routing</span></span>

<span data-ttu-id="2d0f8-118">Successivamente, si modificherà la struttura dell'URL necessaria per accedere all' `Get` azione di `WeatherForecastController` .</span><span class="sxs-lookup"><span data-stu-id="2d0f8-118">Next, you'll change the URL structure needed to access the `Get` action of the `WeatherForecastController`.</span></span> <span data-ttu-id="2d0f8-119">Completare i passaggi seguenti:</span><span class="sxs-lookup"><span data-stu-id="2d0f8-119">Complete the following steps:</span></span>

1. <span data-ttu-id="2d0f8-120">Aprire il file *WeatherForecastController.cs* .</span><span class="sxs-lookup"><span data-stu-id="2d0f8-120">Open the *WeatherForecastController.cs* file.</span></span>
1. <span data-ttu-id="2d0f8-121">Eliminare l' `[Route("[controller]")]` attributo a livello di classe.</span><span class="sxs-lookup"><span data-stu-id="2d0f8-121">Delete the `[Route("[controller]")]` class-level attribute.</span></span> <span data-ttu-id="2d0f8-122">La definizione della classe sarà simile alla seguente:</span><span class="sxs-lookup"><span data-stu-id="2d0f8-122">The class definition will look like the following:</span></span>

    ```csharp
    [ApiController]
    public class WeatherForecastController : ControllerBase
    ```

1. <span data-ttu-id="2d0f8-123">Aggiungere un `[Route("/")]` attributo all' `Get()` azione.</span><span class="sxs-lookup"><span data-stu-id="2d0f8-123">Add a `[Route("/")]` attribute to the `Get()` action.</span></span> <span data-ttu-id="2d0f8-124">La definizione della funzione sarà simile alla seguente:</span><span class="sxs-lookup"><span data-stu-id="2d0f8-124">The function definition will look like the following:</span></span>

    ```csharp
    [HttpGet]
    [Route("/")]
    public IEnumerable<WeatherForecast> Get()
    ```

## <a name="publish-the-web-api-to-azure-app-service"></a><span data-ttu-id="2d0f8-125">Pubblicare l'API Web nel servizio app Azure</span><span class="sxs-lookup"><span data-stu-id="2d0f8-125">Publish the web API to Azure App Service</span></span>

<span data-ttu-id="2d0f8-126">Completare i passaggi seguenti per pubblicare l'API Web di ASP.NET Core in gestione API di Azure:</span><span class="sxs-lookup"><span data-stu-id="2d0f8-126">Complete the following steps to publish the ASP.NET Core web API to Azure API Management:</span></span>

1. <span data-ttu-id="2d0f8-127">Pubblicare l'app per le API nel servizio app Azure.</span><span class="sxs-lookup"><span data-stu-id="2d0f8-127">Publish the API app to Azure App Service.</span></span>
1. <span data-ttu-id="2d0f8-128">Aggiungere un'API vuota all'istanza del servizio gestione API di Azure.</span><span class="sxs-lookup"><span data-stu-id="2d0f8-128">Add a blank API to the Azure API Management service instance.</span></span>
1. <span data-ttu-id="2d0f8-129">Pubblicare l'app per le API Web ASP.NET Core nell'istanza del servizio gestione API di Azure.</span><span class="sxs-lookup"><span data-stu-id="2d0f8-129">Publish the ASP.NET Core web API app to the Azure API Management service instance.</span></span>

### <a name="publish-the-api-app-to-azure-app-service"></a><span data-ttu-id="2d0f8-130">Pubblicare l'app per le API nel servizio app Azure</span><span class="sxs-lookup"><span data-stu-id="2d0f8-130">Publish the API app to Azure App Service</span></span>

<span data-ttu-id="2d0f8-131">Completare i passaggi seguenti per pubblicare l'API Web di ASP.NET Core in gestione API di Azure:</span><span class="sxs-lookup"><span data-stu-id="2d0f8-131">Complete the following steps to publish the ASP.NET Core web API to Azure API Management:</span></span>

1. <span data-ttu-id="2d0f8-132">In **Esplora soluzioni** fare clic con il pulsante destro del mouse sul progetto e scegliere **pubblica**:</span><span class="sxs-lookup"><span data-stu-id="2d0f8-132">In **Solution Explorer**, right-click the project and select **Publish**:</span></span>

    ![Menu di scelta rapida con il collegamento per la pubblicazione evidenziato](publish-to-azure-api-management-using-vs/_static/publish_menu.png)

1. <span data-ttu-id="2d0f8-134">Nella finestra di dialogo **pubblica** selezionare **Azure** e fare clic sul pulsante **Avanti** :</span><span class="sxs-lookup"><span data-stu-id="2d0f8-134">In the **Publish** dialog, select **Azure** and select the **Next** button:</span></span>

    ![Finestra di dialogo Pubblica](publish-to-azure-api-management-using-vs/_static/publish_dialog.png)

1. <span data-ttu-id="2d0f8-136">Selezionare **servizio app Azure (Windows)** e fare clic sul pulsante **Avanti** :</span><span class="sxs-lookup"><span data-stu-id="2d0f8-136">Select **Azure App Service (Windows)** and select the **Next** button:</span></span>

    ![Finestra di dialogo pubblica: selezionare il servizio app](publish-to-azure-api-management-using-vs/_static/publish_dialog_app_svc.png)

1. <span data-ttu-id="2d0f8-138">Selezionare **Crea un nuovo servizio app Azure**.</span><span class="sxs-lookup"><span data-stu-id="2d0f8-138">Select **Create a new Azure App Service**.</span></span>

    ![Finestra di dialogo pubblica: selezionare l'istanza del servizio di Azure](publish-to-azure-api-management-using-vs/_static/publish_dialog_create_new_app_svc.png)

    <span data-ttu-id="2d0f8-140">Viene visualizzata la finestra di dialogo **Crea servizio app**.</span><span class="sxs-lookup"><span data-stu-id="2d0f8-140">The **Create App Service** dialog appears.</span></span> <span data-ttu-id="2d0f8-141">I campi di immissione **Nome dell'app**, **Gruppo di risorse** e **Piano di servizio app** vengono popolati automaticamente.</span><span class="sxs-lookup"><span data-stu-id="2d0f8-141">The **App Name**, **Resource Group**, and **App Service Plan** entry fields are populated.</span></span> <span data-ttu-id="2d0f8-142">È possibile mantenere questi nomi o modificarli.</span><span class="sxs-lookup"><span data-stu-id="2d0f8-142">You can keep these names or change them.</span></span>
1. <span data-ttu-id="2d0f8-143">Selezionare il pulsante **Crea**.</span><span class="sxs-lookup"><span data-stu-id="2d0f8-143">Select the **Create** button.</span></span>

    ![Finestra di dialogo Crea servizio app](publish-to-azure-api-management-using-vs/_static/publish_dialog_app_svc_attributes.png)

<span data-ttu-id="2d0f8-145">Al termine della creazione, la finestra di dialogo viene chiusa automaticamente e la finestra di dialogo di **pubblicazione** viene nuovamente messa a fuoco.</span><span class="sxs-lookup"><span data-stu-id="2d0f8-145">After creation is completed, the dialog is automatically closed and the **Publish** dialog gets focus again.</span></span> <span data-ttu-id="2d0f8-146">L'istanza creata viene selezionata automaticamente.</span><span class="sxs-lookup"><span data-stu-id="2d0f8-146">The instance that was created is automatically selected.</span></span>

![Finestra di dialogo pubblica: selezionare l'istanza del servizio app](publish-to-azure-api-management-using-vs/_static/publish_dialog_app_svc_created.png)

<span data-ttu-id="2d0f8-148">A questo punto, è necessario aggiungere un'API al servizio gestione API di Azure.</span><span class="sxs-lookup"><span data-stu-id="2d0f8-148">At this point, you need to add an API to the Azure API Management service.</span></span> <span data-ttu-id="2d0f8-149">Lasciare aperto Visual Studio quando si completano le attività seguenti.</span><span class="sxs-lookup"><span data-stu-id="2d0f8-149">Leave Visual Studio open while you complete the following tasks.</span></span>

### <a name="add-an-api-to-azure-api-management"></a><span data-ttu-id="2d0f8-150">Aggiungere un'API a gestione API di Azure</span><span class="sxs-lookup"><span data-stu-id="2d0f8-150">Add an API to Azure API Management</span></span>

1. <span data-ttu-id="2d0f8-151">Aprire l'istanza del servizio gestione API creata in precedenza nel portale di Azure.</span><span class="sxs-lookup"><span data-stu-id="2d0f8-151">Open the API Management Service instance created previously in the Azure portal.</span></span> <span data-ttu-id="2d0f8-152">Selezionare il pannello **API** :</span><span class="sxs-lookup"><span data-stu-id="2d0f8-152">Select the **APIs** blade:</span></span>

  ![Pannello API selezionato dall'istanza del servizio gestione API](publish-to-azure-api-management-using-vs/_static/portal_api_overview.png)

1. <span data-ttu-id="2d0f8-154">Selezionare i 3 puntini accanto a **echo API** , quindi scegliere **Elimina** dal menu a comparsa per rimuoverlo.</span><span class="sxs-lookup"><span data-stu-id="2d0f8-154">Select the 3 dots next to **Echo API** and then select **Delete** from the pop-up menu to remove it.</span></span>

  ![Eliminazione dell'API Echo dall'istanza del servizio gestione API](publish-to-azure-api-management-using-vs/_static/portal_delete_echo.png)

1. <span data-ttu-id="2d0f8-156">Nel pannello **Aggiungi nuova API** selezionare il riquadro **API vuota** :</span><span class="sxs-lookup"><span data-stu-id="2d0f8-156">From the **Add a new API** panel, select the **Blank API** tile:</span></span>

  ![Schermata che mostra il riquadro API vuota evidenziato](publish-to-azure-api-management-using-vs/_static/portal_api_create_blank.png)

1. <span data-ttu-id="2d0f8-158">Immettere i valori seguenti nella finestra di dialogo **Crea un'API vuota** visualizzata:</span><span class="sxs-lookup"><span data-stu-id="2d0f8-158">Enter the following values in the **Create a blank API** dialog that appears:</span></span>    

    - <span data-ttu-id="2d0f8-159">**Nome visualizzato**: *weatherforecasts*</span><span class="sxs-lookup"><span data-stu-id="2d0f8-159">**Display Name**: *WeatherForecasts*</span></span>
    - <span data-ttu-id="2d0f8-160">**Nome**: *weatherforecasts*</span><span class="sxs-lookup"><span data-stu-id="2d0f8-160">**Name**: *weatherforecasts*</span></span>
    - <span data-ttu-id="2d0f8-161">**Suffisso URL API**: *V1*</span><span class="sxs-lookup"><span data-stu-id="2d0f8-161">**API Url suffix**: *v1*</span></span>
    - <span data-ttu-id="2d0f8-162">Lasciare vuoto il campo **URL servizio Web** .</span><span class="sxs-lookup"><span data-stu-id="2d0f8-162">Leave the **Web service URL** field empty.</span></span>

1. <span data-ttu-id="2d0f8-163">Selezionare il pulsante **Crea**.</span><span class="sxs-lookup"><span data-stu-id="2d0f8-163">Select the **Create** button.</span></span>

    ![Screenshot della finestra di dialogo Crea un'API vuota completata](publish-to-azure-api-management-using-vs/_static/portal_api_blank_complete.png)

<span data-ttu-id="2d0f8-165">Viene creata l'API vuota.</span><span class="sxs-lookup"><span data-stu-id="2d0f8-165">The blank API is created.</span></span>

![Screenshot di un'API vuota nel portale di gestione API](publish-to-azure-api-management-using-vs/_static/portal_api_blank_created_empty.png)

### <a name="publish-the-aspnet-core-web-api-to-azure-api-management"></a><span data-ttu-id="2d0f8-167">Pubblicare l'API Web di ASP.NET Core in gestione API di Azure</span><span class="sxs-lookup"><span data-stu-id="2d0f8-167">Publish the ASP.NET Core web API to Azure API Management</span></span>

1. <span data-ttu-id="2d0f8-168">Tornare a Visual Studio.</span><span class="sxs-lookup"><span data-stu-id="2d0f8-168">Switch back to Visual Studio.</span></span> <span data-ttu-id="2d0f8-169">La finestra di dialogo di **pubblicazione** deve essere ancora aperta da dove è stato interrotto.</span><span class="sxs-lookup"><span data-stu-id="2d0f8-169">The **Publish** dialog should still be open where you left off before.</span></span>
1. <span data-ttu-id="2d0f8-170">Selezionare il servizio app Azure appena pubblicato in modo che sia evidenziato.</span><span class="sxs-lookup"><span data-stu-id="2d0f8-170">Select the newly published Azure App Service so it's highlighted.</span></span>
1. <span data-ttu-id="2d0f8-171">Selezionare il pulsante **Avanti**.</span><span class="sxs-lookup"><span data-stu-id="2d0f8-171">Select the **Next** button.</span></span>

    ![Screenshot della finestra di dialogo di pubblicazione con il servizio app selezionato](publish-to-azure-api-management-using-vs/_static/publish_dialog_app_svc_created.png)

1. <span data-ttu-id="2d0f8-173">La finestra di dialogo ora Mostra il servizio gestione API di Azure creato in precedenza.</span><span class="sxs-lookup"><span data-stu-id="2d0f8-173">The dialog now shows the Azure API Management service created before.</span></span> <span data-ttu-id="2d0f8-174">Espanderla e la cartella *API* per visualizzare l'API vuota creata.</span><span class="sxs-lookup"><span data-stu-id="2d0f8-174">Expand it and the *APIs* folder to see the blank API you created.</span></span>
1. <span data-ttu-id="2d0f8-175">Selezionare il nome dell'API vuota e selezionare il pulsante **fine** .</span><span class="sxs-lookup"><span data-stu-id="2d0f8-175">Select the blank API's name and select the **Finish** button.</span></span>

    ![Screenshot dell'API vuota di gestione API di Azure appena creata selezionata nella finestra di dialogo pubblica](publish-to-azure-api-management-using-vs/_static/publish_dialog_api_selected.png)

1. <span data-ttu-id="2d0f8-177">La finestra di dialogo viene chiusa e viene visualizzata una schermata di riepilogo con le informazioni sulla pubblicazione.</span><span class="sxs-lookup"><span data-stu-id="2d0f8-177">The dialog closes and a summary screen appears with information about the publish.</span></span> <span data-ttu-id="2d0f8-178">Fare clic sul pulsante **Pubblica**.</span><span class="sxs-lookup"><span data-stu-id="2d0f8-178">Select the **Publish** button.</span></span>

    ![Screenshot di Visual Studio con il profilo di pubblicazione visualizzato](publish-to-azure-api-management-using-vs/_static/vs_publish_profile.png)

    <span data-ttu-id="2d0f8-180">L'API Web pubblicherà sia app Azure servizio che gestione API di Azure.</span><span class="sxs-lookup"><span data-stu-id="2d0f8-180">The web API will publish to both Azure App Service and Azure API Management.</span></span> <span data-ttu-id="2d0f8-181">Viene visualizzata una nuova finestra del browser che mostra l'API in esecuzione nel servizio app Azure.</span><span class="sxs-lookup"><span data-stu-id="2d0f8-181">A new browser window will appear and show the API running in Azure App Service.</span></span> <span data-ttu-id="2d0f8-182">È possibile chiudere la finestra.</span><span class="sxs-lookup"><span data-stu-id="2d0f8-182">You can close that window.</span></span>

1. <span data-ttu-id="2d0f8-183">Tornare all'istanza di gestione API di Azure nel portale di Azure.</span><span class="sxs-lookup"><span data-stu-id="2d0f8-183">Switch back to the Azure API Management instance in the Azure portal.</span></span>
1. <span data-ttu-id="2d0f8-184">Aggiornare la finestra del browser.</span><span class="sxs-lookup"><span data-stu-id="2d0f8-184">Refresh the browser window.</span></span>
1. <span data-ttu-id="2d0f8-185">Selezionare l'API vuota creata nei passaggi precedenti.</span><span class="sxs-lookup"><span data-stu-id="2d0f8-185">Select the blank API you created in the preceding steps.</span></span> <span data-ttu-id="2d0f8-186">Ora è popolato ed è possibile esplorare le informazioni.</span><span class="sxs-lookup"><span data-stu-id="2d0f8-186">It's now populated and you can explore around.</span></span>

    ![Screenshot dell'API popolata in gestione API di Azure](publish-to-azure-api-management-using-vs/_static/deployed_to_azure_api_mgmt.png)

### <a name="configure-the-published-api-name"></a><span data-ttu-id="2d0f8-188">Configurare il nome dell'API pubblicata</span><span class="sxs-lookup"><span data-stu-id="2d0f8-188">Configure the published API name</span></span>

<span data-ttu-id="2d0f8-189">Si noti che il nome dell'API è diverso da quello specificato.</span><span class="sxs-lookup"><span data-stu-id="2d0f8-189">Notice the name of the API is different than what you named it.</span></span> <span data-ttu-id="2d0f8-190">L'API pubblicata è denominata *WeatherAPI*; Tuttavia, è stato denominato *weatherforecasts* al momento della creazione.</span><span class="sxs-lookup"><span data-stu-id="2d0f8-190">The published API is named *WeatherAPI*; however, you named it *WeatherForecasts* when you created it.</span></span> <span data-ttu-id="2d0f8-191">Per correggere il nome, completare i passaggi seguenti:</span><span class="sxs-lookup"><span data-stu-id="2d0f8-191">Complete the following steps to fix the name:</span></span>

![Screenshot dell'API popolata con nomi diversi evidenziati](publish-to-azure-api-management-using-vs/_static/deployed_to_azure_api_mgmt_wrong_name.png)

1. <span data-ttu-id="2d0f8-193">Eliminare la riga seguente dal `Startup.ConfigureServices` Metodo:</span><span class="sxs-lookup"><span data-stu-id="2d0f8-193">Delete the following line from the `Startup.ConfigureServices` method:</span></span>
    
    ```csharp
    services.AddSwaggerGen();
    ```

1. <span data-ttu-id="2d0f8-194">Aggiungere al metodo `Startup.ConfigureServices` il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="2d0f8-194">Add the following code to the `Startup.ConfigureServices` method:</span></span>
    
    ```csharp
    services.AddSwaggerGen(config =>
    {
        config.SwaggerDoc("v1", new Microsoft.OpenApi.Models.OpenApiInfo
        {
            Title = "WeatherForecasts",
            Version = "v1"
        });
    });
    ```

1. <span data-ttu-id="2d0f8-195">Aprire il profilo di pubblicazione appena creato.</span><span class="sxs-lookup"><span data-stu-id="2d0f8-195">Open the newly created publish profile.</span></span> <span data-ttu-id="2d0f8-196">È possibile trovarla da **Esplora soluzioni** nella cartella *Properties/PublishProfiles* .</span><span class="sxs-lookup"><span data-stu-id="2d0f8-196">It can be found from **Solution Explorer** in the *Properties/PublishProfiles* folder.</span></span>

    ![Screenshot che mostra il percorso del file del profilo di pubblicazione evidenziato](publish-to-azure-api-management-using-vs/_static/vs_publish_profile_highlighted.png)

1. <span data-ttu-id="2d0f8-198">Modificare il `<OpenAPIDocumentName>` valore dell'elemento da `v1` a `WeatherForecasts` .</span><span class="sxs-lookup"><span data-stu-id="2d0f8-198">Change the `<OpenAPIDocumentName>` element's value from `v1` to `WeatherForecasts`.</span></span>

    ![screenshot delle modifiche necessarie per il profilo di pubblicazione](publish-to-azure-api-management-using-vs/_static/vs_publish_profile_changes.png)

1. <span data-ttu-id="2d0f8-200">Ripubblicare l'API Web di ASP.NET Core e aprire l'istanza di gestione API di Azure nel portale di Azure.</span><span class="sxs-lookup"><span data-stu-id="2d0f8-200">Republish the ASP.NET Core web API and open the Azure API Management instance in the Azure portal.</span></span>
1. <span data-ttu-id="2d0f8-201">Aggiornare la pagina nel browser.</span><span class="sxs-lookup"><span data-stu-id="2d0f8-201">Refresh the page in your browser.</span></span> <span data-ttu-id="2d0f8-202">Si noterà che il nome dell'API è ora corretto.</span><span class="sxs-lookup"><span data-stu-id="2d0f8-202">You'll see the name of the API is now correct.</span></span>

    ![Screenshot dell'API completata nel portale](publish-to-azure-api-management-using-vs/_static/portal_finish.png)

### <a name="verify-the-web-api-is-working"></a><span data-ttu-id="2d0f8-204">Verificare che l'API Web funzioni</span><span class="sxs-lookup"><span data-stu-id="2d0f8-204">Verify the web API is working</span></span>

<span data-ttu-id="2d0f8-205">È possibile testare l'API Web ASP.NET Core distribuita in gestione API di Azure dalla portale di Azure con i passaggi seguenti:</span><span class="sxs-lookup"><span data-stu-id="2d0f8-205">You can test the deployed ASP.NET Core web API in Azure API Management from the Azure portal with the following steps:</span></span>

1. <span data-ttu-id="2d0f8-206">Aprire la scheda **Test**.</span><span class="sxs-lookup"><span data-stu-id="2d0f8-206">Open the **Test** tab.</span></span>
1. <span data-ttu-id="2d0f8-207">Selezionare **/** o l'operazione **Get** .</span><span class="sxs-lookup"><span data-stu-id="2d0f8-207">Select **/** or the **Get** operation.</span></span>
1. <span data-ttu-id="2d0f8-208">Selezionare **Invia**.</span><span class="sxs-lookup"><span data-stu-id="2d0f8-208">Select **Send**.</span></span>

    ![Screenshot del portale prima del test](publish-to-azure-api-management-using-vs/_static/portal_pre_test.png)

<span data-ttu-id="2d0f8-210">Una risposta con esito positivo avrà un aspetto simile al seguente:</span><span class="sxs-lookup"><span data-stu-id="2d0f8-210">A successful response will look like the following:</span></span>

![Screenshot di una risposta corretta da gestione API](publish-to-azure-api-management-using-vs/_static/portal_successful_response.png)

## <a name="clean-up"></a><span data-ttu-id="2d0f8-212">Eseguire la pulizia</span><span class="sxs-lookup"><span data-stu-id="2d0f8-212">Clean up</span></span>

<span data-ttu-id="2d0f8-213">Al termine del test dell'app, passare al [portale di Azure](https://portal.azure.com/) ed eliminare l'app.</span><span class="sxs-lookup"><span data-stu-id="2d0f8-213">When you've finished testing the app, go to the [Azure portal](https://portal.azure.com/) and delete the app.</span></span>

1. <span data-ttu-id="2d0f8-214">Selezionare **Gruppi di risorse** e in seguito il gruppo di risorse che è stato creato.</span><span class="sxs-lookup"><span data-stu-id="2d0f8-214">Select **Resource groups**, then select the resource group you created.</span></span>

    ![Portale di Azure: Gruppi di risorse nel menu laterale](publish-to-azure-api-management-using-vs/_static/portalrg.png)

1. <span data-ttu-id="2d0f8-216">Nella pagina **Gruppi di risorse** selezionare **Elimina**.</span><span class="sxs-lookup"><span data-stu-id="2d0f8-216">In the **Resource groups** page, select **Delete**.</span></span>

    ![Portale di Azure: pagina Gruppi di risorse](publish-to-azure-api-management-using-vs/_static/rgd.png)

1. <span data-ttu-id="2d0f8-218">Immettere il nome del gruppo di risorse e selezionare **Elimina**.</span><span class="sxs-lookup"><span data-stu-id="2d0f8-218">Enter the name of the resource group and select **Delete**.</span></span> <span data-ttu-id="2d0f8-219">A questo punto l'app e tutte le altre risorse create in questa esercitazione vengono eliminate da Azure.</span><span class="sxs-lookup"><span data-stu-id="2d0f8-219">Your app and all other resources created in this tutorial are now deleted from Azure.</span></span>

## <a name="next-steps"></a><span data-ttu-id="2d0f8-220">Passaggi successivi</span><span class="sxs-lookup"><span data-stu-id="2d0f8-220">Next steps</span></span>

<xref:host-and-deploy/azure-apps/azure-continuous-deployment>

## <a name="additional-resources"></a><span data-ttu-id="2d0f8-221">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="2d0f8-221">Additional resources</span></span>

- [<span data-ttu-id="2d0f8-222">Gestione API di Azure</span><span class="sxs-lookup"><span data-stu-id="2d0f8-222">Azure API Management</span></span>](/azure/api-management/api-management-key-concepts)
- [<span data-ttu-id="2d0f8-223">Servizio app di Azure</span><span class="sxs-lookup"><span data-stu-id="2d0f8-223">Azure App Service</span></span>](/azure/app-service/app-service-web-overview)
