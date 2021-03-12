---
title: 'Integrazione e distribuzione continue: DevOps con ASP.NET Core e Azure'
author: CamSoper
description: Integrazione e distribuzione continue in DevOps con ASP.NET Core e Azure
ms.author: scaddie
ms.date: 10/24/2018
ms.custom: devx-track-csharp, mvc, seodec18
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
uid: azure/devops/cicd
ms.openlocfilehash: 18b2c6ce27132844402f88b2817a07e3588d81c1
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102586267"
---
# <a name="continuous-integration-and-deployment"></a>Integrazione e distribuzione continua

Nel capitolo precedente è stato creato un repository git locale per la semplice app Reader di feed. In questo capitolo si pubblicherà il codice in un repository GitHub e si creerà una pipeline Azure DevOps Services usando Azure Pipelines. La pipeline consente le compilazioni continue e le distribuzioni dell'app. Qualsiasi commit nel repository GitHub attiva una compilazione e una distribuzione nello slot di staging dell'app Web di Azure.

In questa sezione verranno completate le attività seguenti:

* Pubblicare il codice dell'app in GitHub
* Disconnettere la distribuzione git locale
* Creare un'organizzazione Azure DevOps
* Creare un progetto team in Azure DevOps Services
* Creare una definizione di compilazione
* Creare una pipeline di versione
* Eseguire il commit delle modifiche in GitHub e la distribuzione automatica in Azure
* Esaminare la pipeline Azure Pipelines

## <a name="publish-the-apps-code-to-github"></a>Pubblicare il codice dell'app in GitHub

1. Aprire una finestra del browser e passare a `https://github.com` .
1. Fare clic sull' **+** elenco a discesa nell'intestazione e selezionare **nuovo repository**:

    ![Opzione del nuovo repository GitHub](media/cicd/github-new-repo.png)

1. Selezionare l'account nell'elenco a discesa **proprietario** e immettere *Simple-Feed-Reader* nella casella di testo **nome repository** .
1. Fare clic sul pulsante **Crea repository** .
1. Aprire la shell dei comandi del computer locale. Passare alla directory in cui è archiviato il repository git *Simple-Feed-Reader* .
1. Rinominare l' *origine* esistente remote in *upstream*. Eseguire il comando seguente:

    ```console
    git remote rename origin upstream
    ```

1. Aggiungere una nuova *origine* remota che punta alla copia del repository in GitHub. Eseguire il comando seguente:

    ```console
    git remote add origin https://github.com/<GitHub_username>/simple-feed-reader/
    ```

1. Pubblicare il repository git locale nel repository GitHub appena creato. Eseguire il comando seguente:

    ```console
    git push -u origin main
    ```

1. Aprire una finestra del browser e passare a `https://github.com/<GitHub_username>/simple-feed-reader/` . Verificare che il codice venga visualizzato nel repository GitHub.

## <a name="disconnect-local-git-deployment"></a>Disconnettere la distribuzione git locale

Rimuovere la distribuzione git locale con i passaggi seguenti. Azure Pipelines (un servizio Azure DevOps) sostituisce e potenzia tale funzionalità.

1. Aprire il [portale di Azure](https://portal.azure.com/)e passare all'app Web di *staging (MyWebApp \<unique_number\> /staging)* . L'app Web può essere individuata rapidamente immettendo *staging* nella casella di ricerca del portale:

    ![termine di ricerca dell'app Web di staging](media/cicd/portal-search-box.png)

1. Fare clic su **centro distribuzione**. Verrà visualizzato un nuovo pannello. Fare clic su **Disconnetti** per rimuovere la configurazione del controllo del codice sorgente git locale aggiunta nel capitolo precedente. Confermare l'operazione di rimozione facendo clic sul pulsante **Sì** .
1. Passare alla *<mywebapp unique_number>* servizio app. Come promemoria, è possibile usare la casella di ricerca del portale per individuare rapidamente il servizio app.
1. Fare clic su **centro distribuzione**. Verrà visualizzato un nuovo pannello. Fare clic su **Disconnetti** per rimuovere la configurazione del controllo del codice sorgente git locale aggiunta nel capitolo precedente. Confermare l'operazione di rimozione facendo clic sul pulsante **Sì** .

## <a name="create-an-azure-devops-organization"></a>Creare un'organizzazione Azure DevOps

1. Aprire un browser e passare alla pagina di [creazione dell'organizzazione DevOps di Azure](https://go.microsoft.com/fwlink/?LinkId=307137).
1. Digitare un nome univoco nella casella di testo **selezionare un nome memorabile** per creare l'URL per l'accesso all'organizzazione DevOps di Azure.
1. Selezionare il pulsante di opzione **git** , perché il codice è ospitato in un repository GitHub.
1. Fare clic sul pulsante **Continue** (Continue). Dopo una breve attesa, vengono creati un account e un progetto team, denominato *MyFirstProject*.

    ![Pagina di creazione dell'organizzazione DevOps di Azure](media/cicd/vsts-account-creation.png)

1. Aprire il messaggio di posta elettronica di conferma che indica che l'organizzazione e il progetto di Azure DevOps sono pronti per l'uso. Fare clic sul pulsante **Avvia progetto** :

    ![Pulsante Avvia progetto](media/cicd/vsts-start-project.png)

1. Verrà aperto un browser a *\<account_name\> . VisualStudio.com*. Fare clic sul collegamento *MyFirstProject* per avviare la configurazione della pipeline DevOps del progetto.

## <a name="configure-the-azure-pipelines-pipeline"></a>Configurare la pipeline di Azure Pipelines

Sono disponibili tre passaggi distinti per il completamento. Il completamento dei passaggi nelle tre sezioni seguenti restituisce una pipeline DevOps operativa.

### <a name="grant-azure-devops-access-to-the-github-repository"></a>Concedere ad Azure DevOps l'accesso al repository GitHub

1. Espandere il **codice di compilazione o da una Accordion del repository esterno** . Fare clic sul pulsante **Configura compilazione** :

    ![Pulsante Configura compilazione](media/cicd/vsts-setup-build.png)

1. Selezionare l'opzione **GitHub** nella sezione **selezionare un'origine** :

    ![Selezionare un'origine-GitHub](media/cicd/vsts-select-source.png)

1. L'autorizzazione è obbligatoria prima che Azure DevOps possa accedere al repository GitHub. Immettere *<GitHub_username> connessione GitHub* nella casella di testo **nome connessione** . Ad esempio:

    ![Nome della connessione GitHub](media/cicd/vsts-repo-authz.png)

1. Se l'autenticazione a due fattori è abilitata nell'account GitHub, è necessario un token di accesso personale. In tal caso, fare clic sul collegamento **autorizza con un token di accesso personale GitHub** . Vedere le istruzioni per la creazione di un [token di accesso personale GitHub ufficiale](https://help.github.com/articles/creating-a-personal-access-token-for-the-command-line/) per la guida. È necessario solo l'ambito del *repository* delle autorizzazioni. In caso contrario, fare clic sul pulsante **autorizza tramite OAuth** .
1. Quando richiesto, accedere al proprio account GitHub. Selezionare quindi autorizza per concedere l'accesso all'organizzazione DevOps di Azure. Se l'esito è positivo, viene creato un nuovo endpoint del servizio.
1. Fare clic sul pulsante con i puntini di sospensione accanto al pulsante **repository** . Selezionare il *<GitHub_username>repository/Simple-Feed-Reader* dall'elenco. Fare clic sul pulsante **Seleziona**.
1. Selezionare il ramo predefinito (*Main*) dal **ramo predefinito per le compilazioni manuali e pianificate** . Fare clic sul pulsante **Continue** (Continue). Verrà visualizzata la pagina Selezione modello.

### <a name="create-the-build-definition"></a>Creare la definizione di compilazione

1. Nella pagina Selezione modello immettere *ASP.NET Core* nella casella di ricerca:

    ![ASP.NET Core ricerca nella pagina modello](media/cicd/vsts-template-selection.png)

1. Verranno visualizzati i risultati della ricerca del modello. Passare il puntatore del mouse sul modello di **ASP.NET Core** , quindi fare clic sul pulsante **applica** .
1. Verrà visualizzata la scheda **attività** della definizione di compilazione. Fare clic sulla scheda **trigger** .
1. Selezionare la casella **Abilita integrazione continua** . Nella sezione **filtri ramo** verificare che l'elenco a discesa **tipo** sia impostato su *Includi*. Impostare l'elenco a discesa **specifica ramo** su *principale*.

    ![Abilita impostazioni di integrazione continua](media/cicd/vsts-enable-ci.png)

    Queste impostazioni comportano l'attivazione di una compilazione quando viene effettuato il push di una modifica nel ramo predefinito (*Main*) del repository GitHub. L'integrazione continua viene testata nella sezione [commit delle modifiche in GitHub e distribuzione automatica in Azure](#commit-changes-to-github-and-automatically-deploy-to-azure) .

1. Fare clic sul pulsante **salva & coda** e selezionare l'opzione **Salva** :

    ![Pulsante per il salvataggio](media/cicd/vsts-save-build.png)

1. Viene visualizzata la finestra di dialogo modale seguente:

    ![Salva definizione di compilazione-finestra di dialogo modale](media/cicd/vsts-save-modal.png)

    Utilizzare la cartella predefinita *\\* , quindi fare clic sul pulsante **Salva** .

### <a name="create-the-release-pipeline"></a>Creare la pipeline di versione

1. Fare clic sulla scheda **versioni** del progetto team. Fare clic sul pulsante **nuova pipeline** .

    ![Scheda versioni-pulsante nuova definizione](media/cicd/vsts-new-release-definition.png)

    Viene visualizzato il riquadro di selezione del modello.

1. Nella pagina Selezione modello immettere *servizio app* nella casella di ricerca:

    ![Casella di ricerca modello pipeline di rilascio](media/cicd/vsts-release-template-search.png)

1. Verranno visualizzati i risultati della ricerca del modello. Passare il puntatore del mouse sulla **distribuzione del servizio app Azure con il modello di slot** , quindi fare clic sul pulsante **applica** . Verrà visualizzata la scheda **pipeline** della pipeline di rilascio.

    ![Scheda pipeline della pipeline di rilascio](media/cicd/vsts-release-definition-pipeline.png)

1. Fare clic sul pulsante **Aggiungi** nella casella **artefatti** . Viene visualizzato il pannello **Aggiungi elemento** :

    ![Pipeline di rilascio-Aggiungi pannello artefatto](media/cicd/vsts-release-add-artifact.png)

1. Selezionare il riquadro **Compila** dalla sezione **tipo di origine** . Questo tipo consente il collegamento della pipeline di rilascio alla definizione di compilazione.
1. Selezionare *MyFirstProject* dall'elenco a discesa **progetto** .
1. Selezionare il nome della definizione di compilazione, *MyFirstProject-ASP.NET Core-ci*, dall'elenco a discesa **origine (definizione di compilazione)** .
1. Selezionare *più recente* dall'elenco a discesa **versione predefinita** . Questa opzione Compila gli artefatti prodotti dall'ultima esecuzione della definizione di compilazione.
1. Sostituire il testo nella casella di testo **alias di origine** con *Drop*.
1. Fare clic su **Add** . La sezione **artefatti** viene aggiornata per visualizzare le modifiche.
1. Fare clic sull'icona del fulmine per abilitare le distribuzioni continue:

    ![Elementi della pipeline di rilascio-icona del fulmine](media/cicd/vsts-artifacts-lightning-bolt.png)

    Se questa opzione è abilitata, viene eseguita una distribuzione ogni volta che è disponibile una nuova compilazione.
1. A destra viene visualizzato un pannello **trigger di distribuzione continua** . Per abilitare la funzionalità, fare clic sull'interruttore. Non è necessario abilitare il **trigger della richiesta pull**.
1. Fare clic sull'elenco a discesa **Aggiungi** nella sezione **filtri del ramo di compilazione** . Scegliere l'opzione relativa al **ramo predefinito della definizione di compilazione** . Questo filtro determina l'attivazione della versione solo per una compilazione dal ramo predefinito (*Main*) del repository GitHub.
1. Fare clic sul pulsante **Salva**. Fare clic sul pulsante **OK** nella finestra di dialogo **Salva** modale risultante.
1. Fare clic sulla casella **ambiente 1** . A destra viene visualizzato un pannello **dell'ambiente** . Modificare il testo dell' *ambiente 1* nella casella di testo **nome ambiente** in *produzione*.

   ![Pipeline di rilascio-casella di testo nome ambiente](media/cicd/vsts-environment-name-textbox.png)

1. Fare clic sul collegamento **1 fase, 2 attività** nella casella **produzione** :

    ![Pipeline di versione: ambiente di produzione link.png](media/cicd/vsts-production-link.png)

    Verrà visualizzata la scheda **attività** dell'ambiente.
1. Fare clic sull'attività **Distribuisci servizio app Azure a slot** . Le impostazioni vengono visualizzate in un pannello a destra.
1. Selezionare la sottoscrizione di Azure associata al servizio app dall'elenco a discesa **sottoscrizione di Azure** . Una volta selezionata, fare clic sul pulsante **autorizza** .
1. Selezionare *app Web* dall'elenco a discesa **tipo di app** .
1. Selezionare *MyWebApp/<unique_number/>* dall'elenco a discesa **nome servizio app** .
1. Selezionare *AzureTutorial* nell'elenco a discesa **gruppo di risorse** .
1. Selezionare *staging* dall'elenco a discesa **slot** .
1. Fare clic sul pulsante **Salva**.
1. Passare il puntatore del mouse sul nome della pipeline di versione predefinita. Fare clic sull'icona della matita per modificarla. Usare *MyFirstProject-ASP.NET Core-CD* come nome.

    ![Nome della pipeline di versione](media/cicd/vsts-release-definition-name.png)

1. Fare clic sul pulsante **Salva**.

## <a name="commit-changes-to-github-and-automatically-deploy-to-azure"></a>Eseguire il commit delle modifiche in GitHub e la distribuzione automatica in Azure

1. Aprire *SimpleFeedReader. sln* in Visual Studio.
1. In Esplora soluzioni aprire *Pages\Index.cshtml*. Cambiare `<h2>Simple Feed Reader - V3</h2>` in `<h2>Simple Feed Reader - V4</h2>`.
1. Premere **CTRL** + **MAIUSC** + **B** per compilare l'app.
1. Eseguire il commit del file nel repository GitHub. Utilizzare la pagina **modifiche** nella scheda *Team Explorer* di Visual Studio o eseguire le operazioni seguenti utilizzando la shell dei comandi del computer locale:

    ```console
    git commit -a -m "upgraded to V4"
    ```

1. Eseguire il push della modifica nel ramo predefinito (*Main*) nell' *origine* remota del repository GitHub. Nel comando seguente sostituire il segnaposto `{BRANCH}` con il ramo predefinito (use `main` ):

    ```console
    git push origin {BRANCH}
    ```

    Il commit viene visualizzato nel ramo predefinito del repository GitHub (*Main*):

    ![Commit di GitHub nel ramo predefinito (Main)](media/cicd/github-commit.png)

    La compilazione viene attivata perché l'integrazione continua è abilitata nella scheda **trigger** della definizione di compilazione:

    ![Abilita integrazione continua](media/cicd/enable-ci.png)

1. Passare alla scheda in **coda** della   >  pagina **compilazioni** Azure Pipelines in Azure DevOps Services. La compilazione in coda Mostra il ramo e il commit che ha attivato la compilazione:

    ![compilazione in coda](media/cicd/build-queued.png)

1. Una volta completata la compilazione, viene eseguita una distribuzione in Azure. Passare all'app nel browser. Si noti che il testo "V4" viene visualizzato nell'intestazione:

    ![App aggiornata](media/cicd/updated-app-v4.png)

## <a name="examine-the-azure-pipelines-pipeline"></a>Esaminare la pipeline Azure Pipelines

### <a name="build-definition"></a>Definizione di compilazione

È stata creata una definizione di compilazione con il nome *MyFirstProject-ASP.NET Core-ci*. Al termine, la compilazione produce un file con *estensione zip* che include gli asset da pubblicare. La pipeline di rilascio distribuisce tali asset in Azure.

Nella scheda **attività** della definizione di compilazione sono elencati i singoli passaggi utilizzati. Sono disponibili cinque attività di compilazione.

![attività definizione di compilazione](media/cicd/build-definition-tasks.png)

1. **Ripristino** &mdash; di Esegue il `dotnet restore` comando per ripristinare i pacchetti NuGet dell'app. Il feed del pacchetto predefinito usato è nuget.org.
1. **Compilazione** &mdash; di Esegue il `dotnet build --configuration release` comando per compilare il codice dell'app. Questa `--configuration` opzione viene usata per produrre una versione ottimizzata del codice, adatto per la distribuzione in un ambiente di produzione. Modificare la variabile *BuildConfiguration* nella scheda **variabili** della definizione di compilazione se, ad esempio, è necessaria una configurazione di debug.
1. Esegui **test** &mdash; Esegue il `dotnet test --configuration release --logger trx --results-directory <local_path_on_build_agent>` comando per eseguire gli unit test dell'applicazione. Gli unit test vengono eseguiti in qualsiasi progetto C# corrispondente al `**/*Tests/*.csproj` modello glob. I risultati dei test vengono salvati in un file con *estensione TRX* nel percorso specificato dall' `--results-directory` opzione. Se i test hanno esito negativo, la compilazione non riesce e non viene distribuita.

    > [!NOTE]
    > Per verificare il funzionamento degli unit test, modificare *SimpleFeedReader. Tests\Services\NewsServiceTests.cs* per interrompere intenzionalmente uno dei test. Modificare ad esempio `Assert.True(result.Count > 0);` `Assert.False(result.Count > 0);` in nel `Returns_News_Stories_Given_Valid_Uri` metodo. Eseguire il commit e il push della modifica in GitHub. La compilazione viene attivata e ha esito negativo. Lo stato della pipeline di compilazione diventa **non riuscito**. Ripristinare la modifica, il commit e il push. La compilazione ha esito positivo.

1. **Pubblica** &mdash; Esegue il `dotnet publish --configuration release --output <local_path_on_build_agent>` comando per produrre un file con *estensione zip* con gli elementi da distribuire. L' `--output` opzione specifica il percorso di pubblicazione del file con *estensione zip* . Tale percorso viene specificato passando una [variabile predefinita](/azure/devops/pipelines/build/variables) denominata `$(build.artifactstagingdirectory)` . Tale variabile si espande in un percorso locale, ad esempio *c:\agent \_ work\1\a*, nell'agente di compilazione.
1. **Pubblica artefatto** &mdash; Pubblica il file *zip* prodotto dall'attività **Publish** . L'attività accetta il percorso del file con *estensione zip* come parametro, ovvero la variabile predefinita `$(build.artifactstagingdirectory)` . Il file *zip* viene pubblicato come cartella denominata *Drop*.

Fare clic sul collegamento **Riepilogo** definizione di compilazione per visualizzare una cronologia delle compilazioni con la definizione:

![Screenshot che mostra la cronologia delle definizioni di compilazione](media/cicd/build-definition-summary.png)

Nella pagina risultante fare clic sul collegamento corrispondente al numero di build univoco:

![Screenshot che mostra la pagina di riepilogo delle definizioni di compilazione](media/cicd/build-definition-completed.png)

Viene visualizzato un riepilogo di questa compilazione specifica. Fare clic sulla scheda **artefatti** e notare che la cartella di *ricezione* generata dalla compilazione è elencata:

![Screenshot che Mostra gli artefatti delle definizioni di compilazione-cartella di ricezione](media/cicd/build-definition-artifacts.png)

Usare i collegamenti **Scarica** ed **Esplora** per esaminare gli artefatti pubblicati.

### <a name="release-pipeline"></a>Pipeline di versione

È stata creata una pipeline di versione con il nome *MyFirstProject-ASP.NET Core-CD*:

![Screenshot che mostra la panoramica della pipeline di rilascio](media/cicd/release-definition-overview.png)

I due componenti principali della pipeline di rilascio sono gli **elementi** e gli **ambienti**. Facendo clic sulla casella nella sezione **elementi** viene riportato il pannello seguente:

![Screenshot che Mostra gli artefatti della pipeline di rilascio](media/cicd/release-definition-artifacts.png)

Il valore di **origine (definizione di compilazione)** rappresenta la definizione di compilazione a cui è collegata questa pipeline di rilascio. Il file con *estensione zip* prodotto da un'esecuzione corretta della definizione di compilazione viene fornito all'ambiente di *produzione* per la distribuzione in Azure. Fare clic sul collegamento *1 fase, 2 attività* nella casella ambiente di *produzione* per visualizzare le attività della pipeline di rilascio:

![Screenshot che illustra le attività della pipeline di rilascio](media/cicd/release-definition-tasks.png)

La pipeline di versione è costituita da due attività: *distribuisci app Azure servizio in slot* e *Gestisci app Azure scambio di slot di servizio*. Se si fa clic sulla prima attività, viene visualizzata la configurazione dell'attività seguente:

![Screenshot che mostra l'attività di distribuzione della pipeline di rilascio](media/cicd/release-definition-task1.png)

La sottoscrizione di Azure, il tipo di servizio, il nome dell'app Web, il gruppo di risorse e lo slot di distribuzione sono definiti nell'attività di distribuzione. La casella di testo **pacchetto o cartella** include il percorso del file con *estensione zip* da estrarre e distribuire nello slot di *staging* dell'app Web *MyWebApp \<unique_number\>* .

Se si fa clic sull'attività swap slot, viene visualizzata la configurazione dell'attività seguente:

![Screenshot che mostra l'attività di scambio slot della pipeline di rilascio](media/cicd/release-definition-task2.png)

Vengono forniti i dettagli relativi a sottoscrizione, gruppo di risorse, tipo di servizio, nome dell'app Web e slot di distribuzione. La casella **di controllo scambia con produzione** è selezionata. Di conseguenza, i bit distribuiti nello slot di *staging* vengono scambiati nell'ambiente di produzione.

## <a name="additional-reading"></a>Altre letture

* [Creare la prima pipeline con Azure Pipelines](/azure/devops/pipelines/get-started-yaml)
* [Compilazione e progetto .NET Core](/azure/devops/pipelines/languages/dotnet-core)
* [Distribuire un'app Web con Azure Pipelines](/azure/devops/pipelines/targets/webapp)
