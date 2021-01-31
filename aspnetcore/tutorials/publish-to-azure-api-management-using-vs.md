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
# <a name="publish-an-aspnet-core-web-api-to-azure-api-management-with-visual-studio"></a>Pubblicare un'API Web di ASP.NET Core in gestione API di Azure con Visual Studio

Di [Matt Soucoup](https://twitter.com/codemillmatt)

## <a name="set-up"></a>Configurazione

- Aprire un [account Azure gratuito](https://azure.microsoft.com/free/dotnet/) se non è già disponibile un account.
- Se non è già stato fatto, [creare una nuova istanza di gestione API di Azure](/azure/api-management/get-started-create-service-instance) .
- [Creare un progetto di app per le API Web](xref:tutorials/first-web-api#create-a-web-project).

## <a name="configure-the-app"></a>Configurare l'app

L'aggiunta di definizioni di spavalderia all'API Web ASP.NET Core consente a gestione API di Azure di leggere le definizioni delle API dell'app. Completare questi passaggi.

### <a name="add-swagger"></a>Aggiungi spavalderia

1. Aggiungere il pacchetto NuGet [Swashbuckle. AspNetCore](https://www.nuget.org/packages/Swashbuckle.AspNetCore) al progetto API Web ASP.NET Core:

    ![Schermata per configurare la finestra di dialogo NuGet](publish-to-azure-api-management-using-vs/_static/configure_nuget.png)

1. Aggiungere la riga seguente al metodo `Startup.ConfigureServices`:
    
    ```csharp
    services.AddSwaggerGen();
    ```
    
1. Aggiungere la riga seguente al metodo `Startup.Configure`:

    ```csharp
    app.UseSwagger();
    ```

### <a name="change-the-api-routing"></a>Modificare il routing dell'API

Successivamente, si modificherà la struttura dell'URL necessaria per accedere all' `Get` azione di `WeatherForecastController` . Completare i passaggi seguenti:

1. Aprire il file *WeatherForecastController.cs* .
1. Eliminare l' `[Route("[controller]")]` attributo a livello di classe. La definizione della classe sarà simile alla seguente:

    ```csharp
    [ApiController]
    public class WeatherForecastController : ControllerBase
    ```

1. Aggiungere un `[Route("/")]` attributo all' `Get()` azione. La definizione della funzione sarà simile alla seguente:

    ```csharp
    [HttpGet]
    [Route("/")]
    public IEnumerable<WeatherForecast> Get()
    ```

## <a name="publish-the-web-api-to-azure-app-service"></a>Pubblicare l'API Web nel servizio app Azure

Completare i passaggi seguenti per pubblicare l'API Web di ASP.NET Core in gestione API di Azure:

1. Pubblicare l'app per le API nel servizio app Azure.
1. Aggiungere un'API vuota all'istanza del servizio gestione API di Azure.
1. Pubblicare l'app per le API Web ASP.NET Core nell'istanza del servizio gestione API di Azure.

### <a name="publish-the-api-app-to-azure-app-service"></a>Pubblicare l'app per le API nel servizio app Azure

Completare i passaggi seguenti per pubblicare l'API Web di ASP.NET Core in gestione API di Azure:

1. In **Esplora soluzioni** fare clic con il pulsante destro del mouse sul progetto e scegliere **pubblica**:

    ![Menu di scelta rapida con il collegamento per la pubblicazione evidenziato](publish-to-azure-api-management-using-vs/_static/publish_menu.png)

1. Nella finestra di dialogo **pubblica** selezionare **Azure** e fare clic sul pulsante **Avanti** :

    ![Finestra di dialogo Pubblica](publish-to-azure-api-management-using-vs/_static/publish_dialog.png)

1. Selezionare **servizio app Azure (Windows)** e fare clic sul pulsante **Avanti** :

    ![Finestra di dialogo pubblica: selezionare il servizio app](publish-to-azure-api-management-using-vs/_static/publish_dialog_app_svc.png)

1. Selezionare **Crea un nuovo servizio app Azure**.

    ![Finestra di dialogo pubblica: selezionare l'istanza del servizio di Azure](publish-to-azure-api-management-using-vs/_static/publish_dialog_create_new_app_svc.png)

    Viene visualizzata la finestra di dialogo **Crea servizio app**. I campi di immissione **Nome dell'app**, **Gruppo di risorse** e **Piano di servizio app** vengono popolati automaticamente. È possibile mantenere questi nomi o modificarli.
1. Selezionare il pulsante **Crea**.

    ![Finestra di dialogo Crea servizio app](publish-to-azure-api-management-using-vs/_static/publish_dialog_app_svc_attributes.png)

Al termine della creazione, la finestra di dialogo viene chiusa automaticamente e la finestra di dialogo di **pubblicazione** viene nuovamente messa a fuoco. L'istanza creata viene selezionata automaticamente.

![Finestra di dialogo pubblica: selezionare l'istanza del servizio app](publish-to-azure-api-management-using-vs/_static/publish_dialog_app_svc_created.png)

A questo punto, è necessario aggiungere un'API al servizio gestione API di Azure. Lasciare aperto Visual Studio quando si completano le attività seguenti.

### <a name="add-an-api-to-azure-api-management"></a>Aggiungere un'API a gestione API di Azure

1. Aprire l'istanza del servizio gestione API creata in precedenza nel portale di Azure. Selezionare il pannello **API** :

  ![Pannello API selezionato dall'istanza del servizio gestione API](publish-to-azure-api-management-using-vs/_static/portal_api_overview.png)

1. Selezionare i 3 puntini accanto a **echo API** , quindi scegliere **Elimina** dal menu a comparsa per rimuoverlo.

  ![Eliminazione dell'API Echo dall'istanza del servizio gestione API](publish-to-azure-api-management-using-vs/_static/portal_delete_echo.png)

1. Nel pannello **Aggiungi nuova API** selezionare il riquadro **API vuota** :

  ![Schermata che mostra il riquadro API vuota evidenziato](publish-to-azure-api-management-using-vs/_static/portal_api_create_blank.png)

1. Immettere i valori seguenti nella finestra di dialogo **Crea un'API vuota** visualizzata:    

    - **Nome visualizzato**: *weatherforecasts*
    - **Nome**: *weatherforecasts*
    - **Suffisso URL API**: *V1*
    - Lasciare vuoto il campo **URL servizio Web** .

1. Selezionare il pulsante **Crea**.

    ![Screenshot della finestra di dialogo Crea un'API vuota completata](publish-to-azure-api-management-using-vs/_static/portal_api_blank_complete.png)

Viene creata l'API vuota.

![Screenshot di un'API vuota nel portale di gestione API](publish-to-azure-api-management-using-vs/_static/portal_api_blank_created_empty.png)

### <a name="publish-the-aspnet-core-web-api-to-azure-api-management"></a>Pubblicare l'API Web di ASP.NET Core in gestione API di Azure

1. Tornare a Visual Studio. La finestra di dialogo di **pubblicazione** deve essere ancora aperta da dove è stato interrotto.
1. Selezionare il servizio app Azure appena pubblicato in modo che sia evidenziato.
1. Selezionare il pulsante **Avanti**.

    ![Screenshot della finestra di dialogo di pubblicazione con il servizio app selezionato](publish-to-azure-api-management-using-vs/_static/publish_dialog_app_svc_created.png)

1. La finestra di dialogo ora Mostra il servizio gestione API di Azure creato in precedenza. Espanderla e la cartella *API* per visualizzare l'API vuota creata.
1. Selezionare il nome dell'API vuota e selezionare il pulsante **fine** .

    ![Screenshot dell'API vuota di gestione API di Azure appena creata selezionata nella finestra di dialogo pubblica](publish-to-azure-api-management-using-vs/_static/publish_dialog_api_selected.png)

1. La finestra di dialogo viene chiusa e viene visualizzata una schermata di riepilogo con le informazioni sulla pubblicazione. Fare clic sul pulsante **Pubblica**.

    ![Screenshot di Visual Studio con il profilo di pubblicazione visualizzato](publish-to-azure-api-management-using-vs/_static/vs_publish_profile.png)

    L'API Web pubblicherà sia app Azure servizio che gestione API di Azure. Viene visualizzata una nuova finestra del browser che mostra l'API in esecuzione nel servizio app Azure. È possibile chiudere la finestra.

1. Tornare all'istanza di gestione API di Azure nel portale di Azure.
1. Aggiornare la finestra del browser.
1. Selezionare l'API vuota creata nei passaggi precedenti. Ora è popolato ed è possibile esplorare le informazioni.

    ![Screenshot dell'API popolata in gestione API di Azure](publish-to-azure-api-management-using-vs/_static/deployed_to_azure_api_mgmt.png)

### <a name="configure-the-published-api-name"></a>Configurare il nome dell'API pubblicata

Si noti che il nome dell'API è diverso da quello specificato. L'API pubblicata è denominata *WeatherAPI*; Tuttavia, è stato denominato *weatherforecasts* al momento della creazione. Per correggere il nome, completare i passaggi seguenti:

![Screenshot dell'API popolata con nomi diversi evidenziati](publish-to-azure-api-management-using-vs/_static/deployed_to_azure_api_mgmt_wrong_name.png)

1. Eliminare la riga seguente dal `Startup.ConfigureServices` Metodo:
    
    ```csharp
    services.AddSwaggerGen();
    ```

1. Aggiungere al metodo `Startup.ConfigureServices` il codice seguente:
    
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

1. Aprire il profilo di pubblicazione appena creato. È possibile trovarla da **Esplora soluzioni** nella cartella *Properties/PublishProfiles* .

    ![Screenshot che mostra il percorso del file del profilo di pubblicazione evidenziato](publish-to-azure-api-management-using-vs/_static/vs_publish_profile_highlighted.png)

1. Modificare il `<OpenAPIDocumentName>` valore dell'elemento da `v1` a `WeatherForecasts` .

    ![screenshot delle modifiche necessarie per il profilo di pubblicazione](publish-to-azure-api-management-using-vs/_static/vs_publish_profile_changes.png)

1. Ripubblicare l'API Web di ASP.NET Core e aprire l'istanza di gestione API di Azure nel portale di Azure.
1. Aggiornare la pagina nel browser. Si noterà che il nome dell'API è ora corretto.

    ![Screenshot dell'API completata nel portale](publish-to-azure-api-management-using-vs/_static/portal_finish.png)

### <a name="verify-the-web-api-is-working"></a>Verificare che l'API Web funzioni

È possibile testare l'API Web ASP.NET Core distribuita in gestione API di Azure dalla portale di Azure con i passaggi seguenti:

1. Aprire la scheda **Test**.
1. Selezionare **/** o l'operazione **Get** .
1. Selezionare **Invia**.

    ![Screenshot del portale prima del test](publish-to-azure-api-management-using-vs/_static/portal_pre_test.png)

Una risposta con esito positivo avrà un aspetto simile al seguente:

![Screenshot di una risposta corretta da gestione API](publish-to-azure-api-management-using-vs/_static/portal_successful_response.png)

## <a name="clean-up"></a>Eseguire la pulizia

Al termine del test dell'app, passare al [portale di Azure](https://portal.azure.com/) ed eliminare l'app.

1. Selezionare **Gruppi di risorse** e in seguito il gruppo di risorse che è stato creato.

    ![Portale di Azure: Gruppi di risorse nel menu laterale](publish-to-azure-api-management-using-vs/_static/portalrg.png)

1. Nella pagina **Gruppi di risorse** selezionare **Elimina**.

    ![Portale di Azure: pagina Gruppi di risorse](publish-to-azure-api-management-using-vs/_static/rgd.png)

1. Immettere il nome del gruppo di risorse e selezionare **Elimina**. A questo punto l'app e tutte le altre risorse create in questa esercitazione vengono eliminate da Azure.

## <a name="next-steps"></a>Passaggi successivi

<xref:host-and-deploy/azure-apps/azure-continuous-deployment>

## <a name="additional-resources"></a>Risorse aggiuntive

- [Gestione API di Azure](/azure/api-management/api-management-key-concepts)
- [Servizio app di Azure](/azure/app-service/app-service-web-overview)
