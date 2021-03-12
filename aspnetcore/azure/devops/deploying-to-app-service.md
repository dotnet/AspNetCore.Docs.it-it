---
title: Distribuire un'app nel servizio app-DevOps con ASP.NET Core e Azure
author: CamSoper
description: Distribuire un'app ASP.NET Core al servizio app Azure, il primo passaggio per DevOps con ASP.NET Core e Azure.
ms.author: casoper
ms.custom: devx-track-csharp, mvc, seodec18, devx-track-azurecli
ms.date: 10/24/2018
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
uid: azure/devops/deploy-to-app-service
ms.openlocfilehash: 6d82611eb2176126652d0d6c8db3779cde70dab5
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102586241"
---
# <a name="deploy-an-app-to-app-service"></a>Distribuire un'app nel servizio app

[App Azure servizio](/azure/app-service/) è la piattaforma di hosting Web di Azure. La distribuzione di un'app Web in app Azure servizio può essere eseguita manualmente o tramite un processo automatizzato. Questa sezione della guida illustra i metodi di distribuzione che possono essere attivati manualmente o tramite script tramite la riga di comando o attivati manualmente con Visual Studio.

In questa sezione verranno eseguite le attività seguenti:

* Scaricare e compilare l'app di esempio.
* Creare un'app Web del servizio app Azure usando il Azure Cloud Shell.
* Distribuire l'app di esempio in Azure usando git.
* Distribuire una modifica all'app usando Visual Studio.
* Aggiungere uno slot di staging all'app Web.
* Distribuire un aggiornamento nello slot di gestione temporanea.
* Scambiare gli slot di staging e di produzione.

## <a name="download-and-test-the-app"></a>Scaricare e testare l'app

L'app usata in questa guida è un'app ASP.NET Core predefinita, un [lettore di feed semplice](https://github.com/Azure-Samples/simple-feed-reader/). Si tratta di un' Razor app di pagine che usa l' `Microsoft.SyndicationFeed.ReaderWriter` API per recuperare un feed RSS/Atom e visualizzare le voci di notizie in un elenco.

È possibile esaminare il codice, ma è importante comprendere che non c'è niente di speciale sull'app. Si tratta semplicemente di una semplice app ASP.NET Core a scopo illustrativo.

Da una shell dei comandi scaricare il codice, compilare il progetto ed eseguirlo come indicato di seguito.

> *Nota: gli utenti Linux/macOS devono apportare le modifiche appropriate per i percorsi, ad esempio usando la barra ( `/` ) anziché la barra rovesciata ( `\` ).*

1. Clonare il codice in una cartella nel computer locale.

    ```console
    git clone https://github.com/Azure-Samples/simple-feed-reader/
    ```

2. Modificare la cartella di lavoro nella cartella *Simple-Feed-Reader* creata.

    ```console
    cd .\simple-feed-reader\SimpleFeedReader
    ```

3. Ripristinare i pacchetti e compilare la soluzione.

    ```dotnetcli
    dotnet build
    ```

4. Eseguire l'app.

    ```dotnetcli
    dotnet run
    ```

    ![Il comando DotNet Run ha avuto esito positivo](./media/deploying-to-app-service/dotnet-run.png)

5. Aprire un browser e passare a `http://localhost:5000`. L'app consente di digitare o incollare un URL del feed di diffusione e visualizzare un elenco di elementi di notizie.

     ![L'app che Visualizza il contenuto di un feed RSS](./media/deploying-to-app-service/app-in-browser.png)

6. Una volta soddisfatta la corretta esecuzione dell'app, chiuderla premendo **CTRL** + **C** nella shell dei comandi.

## <a name="create-the-azure-app-service-web-app"></a>Creare l'app Web del servizio app Azure

Per distribuire l'app, è necessario creare un' [app Web](/azure/app-service/app-service-web-overview)del servizio app. Dopo la creazione dell'app Web, verrà distribuita nel computer locale usando git.

1. Accedere al [Azure cloud Shell](https://shell.azure.com/bash). Nota: quando si accede per la prima volta, Cloud Shell richiede di creare un account di archiviazione per i file di configurazione. Accettare le impostazioni predefinite o specificare un nome univoco.

2. Usare il Cloud Shell per i passaggi seguenti.

    a. Dichiarare una variabile per archiviare il nome dell'app Web. Il nome deve essere univoco per essere usato nell'URL predefinito. L'uso della `$RANDOM` funzione bash per costruire il nome garantisce l'univocità e restituisce il formato `webappname99999` .

    ```console
    webappname=mywebapp$RANDOM
    ```

    b. Creare un gruppo di risorse. I gruppi di risorse forniscono un mezzo per aggregare le risorse di Azure da gestire come gruppo.

    ```azurecli
    az group create --location centralus --name AzureTutorial
    ```

    Il `az` comando richiama l'interfaccia della riga di comando di [Azure](/cli/azure/). L'interfaccia della riga di comando può essere eseguita localmente, ma viene usata nel Cloud Shell consente di risparmiare tempo e configurazione.

    c. Creare un piano di servizio app nel livello S1. Un piano di servizio app è un raggruppamento di app Web che condividono lo stesso piano tariffario. Il livello S1 non è gratuito, ma è necessario per la funzionalità degli slot di staging.

    ```azurecli
    az appservice plan create --name $webappname --resource-group AzureTutorial --sku S1
    ```

    d. Creare la risorsa dell'app Web usando il piano di servizio app nello stesso gruppo di risorse.

    ```azurecli
    az webapp create --name $webappname --resource-group AzureTutorial --plan $webappname
    ```

    e. Impostare le credenziali per la distribuzione. Queste credenziali di distribuzione si applicano a tutte le app Web nella sottoscrizione. Non usare caratteri speciali nel nome utente.

    ```azurecli
    az webapp deployment user set --user-name REPLACE_WITH_USER_NAME --password REPLACE_WITH_PASSWORD
    ```

    f. Configurare l'app Web per accettare le distribuzioni da git locale e visualizzare l' *URL di distribuzione git*. **Si noti questo URL per il riferimento in un secondo momento**.

    ```azurecli
    echo Git deployment URL: $(az webapp deployment source config-local-git --name $webappname --resource-group AzureTutorial --query url --output tsv)
    ```

    g. Visualizzare l' *URL dell'app Web*. Passare a questo URL per visualizzare l'app Web vuota. **Si noti questo URL per il riferimento in un secondo momento**.

    ```console
    echo Web app URL: http://$webappname.azurewebsites.net
    ```

3. Utilizzando una shell dei comandi nel computer locale, passare alla cartella del progetto dell'app Web (ad esempio, `.\simple-feed-reader\SimpleFeedReader` ). Eseguire i comandi seguenti per impostare git per eseguire il push all'URL di distribuzione:

    a. Aggiungere l'URL remoto al repository locale.

    ```console
    git remote add azure-prod GIT_DEPLOYMENT_URL
    ```

    b. Eseguire il push del ramo predefinito locale (*Main*) al ramo predefinito (*Main*) di *Azure-prod* remote.

    ```console
    git push azure-prod main
    ```

    Verranno richieste le credenziali di distribuzione create in precedenza. Osservare l'output nella shell dei comandi. Azure compila l'app ASP.NET Core in modalità remota.

4. In un browser passare all'URL dell' *app Web* e notare che l'app è stata compilata e distribuita. È possibile eseguire il commit di altre modifiche nel repository git locale con `git commit` . Queste modifiche vengono inserite in Azure con il `git push` comando precedente.

## <a name="deployment-with-visual-studio"></a>Distribuzione con Visual Studio

> *Nota: questa sezione si applica solo a Windows. Gli utenti di Linux e macOS devono apportare le modifiche descritte nel passaggio 2 seguente. Salvare il file ed eseguire il commit della modifica nel repository locale con `git commit` . Infine, eseguire il push della modifica con `git push` , come nella prima sezione.*

L'app è già stata distribuita dalla shell dei comandi. Usare gli strumenti integrati di Visual Studio per distribuire un aggiornamento nell'app. In background, Visual Studio esegue la stessa operazione degli strumenti da riga di comando, ma all'interno dell'interfaccia utente familiare di Visual Studio.

1. Aprire *SimpleFeedReader. sln* in Visual Studio.
2. In Esplora soluzioni aprire *Pages\Index.cshtml*. Cambiare `<h2>Simple Feed Reader</h2>` in `<h2>Simple Feed Reader - V2</h2>`.
3. Premere **CTRL** + **MAIUSC** + **B** per compilare l'app.
4. In Esplora soluzioni fare clic con il pulsante destro del mouse sul progetto e scegliere **pubblica**.

    ![Screenshot che mostra il pulsante destro del mouse, pubblica](./media/deploying-to-app-service/publish.png)
5. Visual Studio può creare una nuova risorsa del servizio app, ma questo aggiornamento verrà pubblicato sulla distribuzione esistente. Nella finestra di dialogo selezionare **una destinazione di pubblicazione** selezionare **servizio app** dall'elenco a sinistra e quindi selezionare **Seleziona esistente**. Fare clic su **Pubblica**.
6. Nella finestra di dialogo **servizio app** confermare che l'account Microsoft o dell'organizzazione usato per creare la sottoscrizione di Azure sia visualizzato in alto a destra. In caso contrario, fare clic sull'elenco a discesa e aggiungerlo.
7. Verificare che sia selezionata la **sottoscrizione** di Azure corretta. Per **Visualizza**, selezionare **gruppo di risorse**. Espandere il gruppo di risorse **AzureTutorial** , quindi selezionare l'app Web esistente. Fare clic su **OK**.

    ![Screenshot che mostra la finestra di dialogo Pubblica servizio app](./media/deploying-to-app-service/publish-dialog.png)

Visual Studio compila e distribuisce l'app in Azure. Passare all'URL dell'app Web. Verificare che la `<h2>` modifica dell'elemento sia attiva.

![L'app con il titolo modificato](./media/deploying-to-app-service/app-v2.png)

## <a name="deployment-slots"></a>Slot di distribuzione

Gli slot di distribuzione supportano la gestione temporanea delle modifiche senza compromettere l'applicazione in esecuzione nell'ambiente di produzione. Una volta che la versione di gestione temporanea dell'app è stata convalidata da un team di garanzia di qualità, è possibile scambiare gli slot di produzione e di staging. In questo modo, l'app in staging viene promossa alla produzione. I passaggi seguenti consentono di creare uno slot di staging, distribuire alcune modifiche e scambiare lo slot di gestione temporanea con la produzione dopo la verifica.

1. Accedere al [Azure cloud Shell](https://shell.azure.com/bash), se non è già stato effettuato l'accesso.
2. Creare lo slot di staging.

    a. Creare uno slot di distribuzione con il nome *staging*.

    ```azurecli
    az webapp deployment slot create --name $webappname --resource-group AzureTutorial --slot staging
    ```

    b. Configurare lo slot di staging per usare la distribuzione da git locale e ottenere l'URL di distribuzione di **gestione temporanea** . **Si noti questo URL per il riferimento in un secondo momento**.

    ```azurecli
    echo Git deployment URL for staging: $(az webapp deployment source config-local-git --name $webappname --resource-group AzureTutorial --slot staging --query url --output tsv)
    ```

    c. Visualizzare l'URL dello slot di gestione temporanea. Passare all'URL per visualizzare lo slot di gestione temporanea vuoto. **Si noti questo URL per il riferimento in un secondo momento**.

    ```console
    echo Staging web app URL: http://$webappname-staging.azurewebsites.net
    ```

3. In un editor di testo o in Visual Studio modificare di nuovo *pages/index. cshtml* in modo che l' `<h2>` elemento legga `<h2>Simple Feed Reader - V3</h2>` e salvi il file.

4. Eseguire il commit del file nel repository git locale, usando la pagina **modifiche** nella scheda *Team Explorer* di Visual Studio oppure immettendo quanto segue usando la shell dei comandi del computer locale:

    ```console
    git commit -a -m "upgraded to V3"
    ```

5. Utilizzando la shell dei comandi del computer locale, aggiungere l'URL di distribuzione di gestione temporanea come Git remoto e inserire le modifiche di cui è stato eseguito il commit:

    a. Aggiungere l'URL remoto per la gestione temporanea al repository git locale.

    ```console
    git remote add azure-staging <Git_staging_deployment_URL>
    ```

    b. Eseguire il push del ramo predefinito locale (*Main*) al ramo predefinito (*Main*) del ramo remoto di *gestione temporanea di Azure* .

    ```console
    git push azure-staging main
    ```

    Attendere mentre Azure compila e distribuisce l'app.

6. Per verificare che V3 sia stato distribuito nello slot di staging, aprire due finestre del browser. In una finestra passare all'URL dell'app Web originale. Nell'altra finestra passare all'URL dell'app Web di staging. L'URL di produzione serve V2 dell'app. L'URL di gestione temporanea serve V3 dell'app.

    ![Screenshot che confronta le finestre del browser](./media/deploying-to-app-service/ready-to-swap.png)

7. Nel Cloud Shell, scambiare lo slot di gestione temporanea verificato/scaldato nell'ambiente di produzione.

    ```azurecli
    az webapp deployment slot swap --name $webappname --resource-group AzureTutorial --slot staging
    ```

8. Verificare che lo scambio si sia verificato aggiornando le due finestre del browser.

    ![Confronto tra le finestre del browser dopo lo scambio](./media/deploying-to-app-service/swapped.png)

## <a name="summary"></a>Riepilogo

In questa sezione sono state completate le attività seguenti:

* Scaricata e compilata l'app di esempio.
* È stata creata un'app Web del servizio app Azure usando il Azure Cloud Shell.
* L'app di esempio è stata distribuita in Azure usando git.
* È stata distribuita una modifica all'app con Visual Studio.
* È stato aggiunto uno slot di staging all'app Web.
* È stato distribuito un aggiornamento nello slot di gestione temporanea.
* Scambiati gli slot di gestione temporanea e di produzione.

Nella sezione successiva si apprenderà come creare una pipeline DevOps con Azure Pipelines.

## <a name="additional-reading"></a>Altre letture

* [Panoramica di App Web](/azure/app-service/app-service-web-overview)
* [Creare un'app Web .NET Core e database SQL nel Servizio app di Azure](/azure/app-service/app-service-web-tutorial-dotnetcore-sqldb)
* [Configurazione delle credenziali per la distribuzione del Servizio app di Azure](/azure/app-service/app-service-deployment-credentials)
* [Configurare gli ambienti di gestione temporanea nel servizio app di Azure](/azure/app-service/web-sites-staged-publishing)
