## <a name="troubleshoot"></a>Risolvere problemi

::: moniker range=">= aspnetcore-5.0"

### <a name="common-errors"></a>Errori comuni

* Client non autorizzato per AAD

  > Info: autorizzazione Microsoft. AspNetCore. Authorization. DefaultAuthorizationService [2] non riuscita. Questi requisiti non sono soddisfatti: DenyAnonymousAuthorizationRequirement: richiede un utente autenticato.

  Errore di callback di accesso da AAD:

  * Errore: `unauthorized_client`
  * Descrizione `AADB2C90058: The provided application is not configured to allow public clients.`

  Per risolvere l'errore:

  1. Nel portale di Azure accedere al manifesto dell' [app](/azure/active-directory/develop/reference-app-manifest).
  1. Impostare l' [ `allowPublicClient` attributo](/azure/active-directory/develop/reference-app-manifest#allowpublicclient-attribute) su `null` o su `true` .

::: moniker-end

### <a name="cookies-and-site-data"></a>Cookie e dati del sito

I cookie e i dati del sito possono essere mantenuti tra gli aggiornamenti delle app e interferiscono con i test e la risoluzione Quando si apportano modifiche al codice dell'app, si cancellano le modifiche dell'account utente con il provider o le modifiche alla configurazione dell'app provider:

* Cookie di accesso utente
* Cookie dell'app
* Dati del sito memorizzati nella cache e archiviati

Un approccio che impedisce ai cookie e ai dati del sito di interferire con i test e la risoluzione dei problemi consiste nel:

* Configurare un browser
  * Utilizzare un browser per i test che è possibile configurare per eliminare tutti i cookie e i dati del sito ogni volta che il browser viene chiuso.
  * Verificare che il browser sia chiuso manualmente o dall'IDE per qualsiasi modifica all'app, all'utente di test o alla configurazione del provider.
* Usare un comando personalizzato per aprire un browser in modalità privata o in incognito in Visual Studio:
  * Aprire la finestra di dialogo **Sfoglia con** dal pulsante di **esecuzione** di Visual Studio.
  * Fare clic sul pulsante **Aggiungi**.
  * Consente di specificare il percorso del browser nel campo del **programma** . I percorsi eseguibili seguenti sono i percorsi di installazione tipici di Windows 10. Se il browser è installato in un percorso diverso o non si usa Windows 10, specificare il percorso del file eseguibile del browser.
    * Microsoft Edge: `C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe`
    * Google Chrome: `C:\Program Files (x86)\Google\Chrome\Application\chrome.exe`
    * Mozilla Firefox: `C:\Program Files\Mozilla Firefox\firefox.exe`
  * Nel campo **arguments (argomenti** ) specificare l'opzione della riga di comando utilizzata dal browser per aprire in modalità privata o in incognito. Alcuni browser richiedono l'URL dell'app.
    * Microsoft Edge: usare `-inprivate` .
    * Google Chrome: usare `--incognito --new-window {URL}` , dove il segnaposto `{URL}` è l'URL da aprire, ad esempio `https://localhost:5001` .
    * Mozilla Firefox: usare `-private -url {URL}` , dove il segnaposto `{URL}` è l'URL da aprire, ad esempio `https://localhost:5001` .
  * Specificare un nome nel campo **nome descrittivo** . Ad esempio: `Firefox Auth Testing`.
  * Selezionare il pulsante **OK** .
  * Per evitare di dover selezionare il profilo del browser per ogni iterazione del test con un'app, impostare il profilo come predefinito con il pulsante **Imposta come predefinito** .
  * Verificare che il browser sia chiuso dall'IDE per qualsiasi modifica all'app, all'utente di test o alla configurazione del provider.

### <a name="run-the-server-app"></a>Eseguire l'app Server

Durante il test e la risoluzione dei problemi di un'app Blazer ospitata, assicurarsi di eseguire l'app dal **`Server`** progetto. Ad esempio, in Visual Studio verificare che il progetto server sia evidenziato in **Esplora soluzioni** prima di avviare l'app con uno degli approcci seguenti:

* Selezionare il pulsante **Run** (Esegui).
* Usare **debug**  >  **Avvia debug** dal menu.
* Premere <kbd>F5</kbd>.

### <a name="inspect-the-content-of-a-json-web-token-jwt"></a>Esaminare il contenuto di un token JSON Web (JWT)

Per decodificare un token JSON Web (JWT), usare lo strumento [JWT.ms](https://jwt.ms/) di Microsoft. I valori nell'interfaccia utente non lasciano mai il browser.
