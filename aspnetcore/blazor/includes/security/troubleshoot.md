---
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
ms.openlocfilehash: f58da78475d65cbb70b0b177e1b0443535e97d55
ms.sourcegitcommit: 1436bd4d70937d6ec3140da56d96caab33c4320b
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/06/2021
ms.locfileid: "102402300"
---
## <a name="troubleshoot"></a>Risolvere problemi

### <a name="common-errors"></a>Errori comuni

* Configurazione errata dell'app o del Identity provider (IP)

  Gli errori più comuni sono causati da una configurazione non corretta. Di seguito sono riportati alcuni esempi:
  
  * A seconda dei requisiti dello scenario, un'autorità mancante o errata, un'istanza, un ID tenant, un dominio tenant, un ID client o un URI di reindirizzamento impedisce a un'app di autenticare i client.
  * Un ambito del token di accesso non corretto impedisce ai client di accedere agli endpoint dell'API Web del server.
  * Le autorizzazioni dell'API server errate o mancanti impediscono ai client di accedere agli endpoint dell'API Web del server.
  * Esecuzione dell'app in una porta diversa da quella configurata nell'URI di reindirizzamento della Identity registrazione dell'app del provider.
  
  Le sezioni di configurazione delle linee guida di questo articolo mostrano esempi della configurazione corretta. Controllare attentamente ogni sezione dell'articolo ricerca di app e configurazione errata dell'IP.
  
  Se la configurazione appare corretta:
  
  * Analizzare i registri applicazioni.
  * Esaminare il traffico di rete tra l'app client e l'app IP o server con gli strumenti di sviluppo del browser. Spesso un messaggio di errore esatto o un messaggio con un indizio sulla causa del problema vengono restituiti al client dall'app IP o server dopo aver effettuato una richiesta. Le linee guida per gli strumenti di sviluppo sono disponibili negli articoli seguenti:

    * [Google Chrome](https://developers.google.com/web/tools/chrome-devtools/network) (documentazione di Google)
    * [Microsoft Edge](/microsoft-edge/devtools-guide-chromium/network/)
    * [Mozilla Firefox](https://developer.mozilla.org/docs/Tools/Network_Monitor) (documentazione di Mozilla)

  * Decodifica il contenuto di un token JSON Web (JWT) usato per l'autenticazione di un client o l'accesso a un'API Web Server, a seconda della posizione in cui si è verificato il problema. Per altre informazioni, vedere [esaminare il contenuto di un token JSON Web (JWT)](#inspect-the-content-of-a-json-web-token-jwt).
  
  Il team di documentazione risponde al feedback dei documenti e ai bug negli articoli (si apre un problema dalla sezione feedback della **pagina** ), ma non è in grado di fornire supporto tecnico. Sono disponibili diversi forum di supporto pubblico per semplificare la risoluzione dei problemi di un'app. È consigliabile attenersi alle linee guida seguenti:
  
  * [Stack Overflow (Tag: `blazor` )](https://stackoverflow.com/questions/tagged/blazor)
  * [Team di ASP.NET Core Slack](http://tattoocoder.com/aspnet-slack-sign-up/)
  * [Blazor Gitter](https://gitter.im/aspnet/Blazor)
  
  Per i report sui bug di Framework riproducibili non riservati, non sensibili e non riservati, [aprire un problema con l'unità di prodotto ASP.NET Core](https://github.com/dotnet/aspnetcore/issues). Non aprire un problema con l'unità di prodotto fino a quando non si è approfondita la causa di un problema e non è possibile risolverlo autonomamente e con l'aiuto della community su un forum di supporto pubblico. L'unità prodotto non è in grado di risolvere i problemi relativi alle singole app interrotte a causa di semplici errori di configurazione o casi di utilizzo che coinvolgono servizi di terze parti. Se un report è sensibile o riservato in natura o descrive un potenziale difetto di sicurezza nel prodotto che gli utenti malintenzionati possono sfruttare, vedere [segnalazione di problemi di sicurezza e bug (repository GitHub DotNet/aspnetcore)](https://github.com/dotnet/aspnetcore/blob/main/CONTRIBUTING.md#reporting-security-issues-and-bugs).

::: moniker range=">= aspnetcore-5.0"

* Client non autorizzato per AAD

  > Info: autorizzazione Microsoft. AspNetCore. Authorization. DefaultAuthorizationService [2] non riuscita. Questi requisiti non sono soddisfatti: DenyAnonymousAuthorizationRequirement: richiede un utente autenticato.

  Errore di callback di accesso da AAD:

  * Errore: `unauthorized_client`
  * Descrizione `AADB2C90058: The provided application is not configured to allow public clients.`

  Per risolvere l'errore:

  1. Nel portale di Azure accedere al manifesto dell' [app](/azure/active-directory/develop/reference-app-manifest).
  1. Impostare l' [ `allowPublicClient` attributo](/azure/active-directory/develop/reference-app-manifest#allowpublicclient-attribute) su `null` o su `true` .

::: moniker-end

### <a name="cookies-and-site-data"></a>Cookies e dati del sito

Cookiee i dati del sito possono essere mantenuti tra gli aggiornamenti delle app e interferire con i test e la risoluzione dei problemi Quando si apportano modifiche al codice dell'app, si cancellano le modifiche dell'account utente con il provider o le modifiche alla configurazione dell'app provider:

* Accesso utente cookie
* App cookie s
* Dati del sito memorizzati nella cache e archiviati

Un approccio per impedire che i cookie dati del sito e di quelli persistenti interferiscano con i test e la risoluzione dei problemi sono:

* Configurare un browser
  * Utilizzare un browser per i test che è possibile configurare per eliminare tutti i cookie dati di e del sito ogni volta che il browser viene chiuso.
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

### <a name="app-upgrades"></a>Aggiornamenti di app

Un'app funzionante potrebbe non riuscire immediatamente dopo l'aggiornamento del .NET Core SDK nel computer di sviluppo o la modifica delle versioni del pacchetto all'interno dell'app. In alcuni casi i pacchetti incoerenti possono interrompere un'app quando si eseguono aggiornamenti principali. La maggior parte di questi problemi può essere risolta attenendosi alle istruzioni seguenti:

1. Cancellare le cache dei pacchetti NuGet del sistema locale eseguendo [`dotnet nuget locals all --clear`](/dotnet/core/tools/dotnet-nuget-locals) da una shell dei comandi.
1. Eliminare le `bin` cartelle e del progetto `obj` .
1. Ripristinare e ricompilare il progetto.
1. Eliminare tutti i file nella cartella di distribuzione nel server prima di ridistribuire l'app.

> [!NOTE]
> L'uso di versioni dei pacchetti incompatibili con il Framework di destinazione dell'app non è supportato. Per informazioni su un pacchetto, usare la [raccolta NuGet](https://www.nuget.org) o [FuGet Package Explorer](https://www.fuget.org).

### <a name="run-the-server-app"></a>Eseguire l'app Server

Durante il test e la risoluzione dei problemi di una soluzione ospitata Blazor , assicurarsi di eseguire l'app dal **`Server`** progetto. Ad esempio, in Visual Studio verificare che il progetto server sia evidenziato in **Esplora soluzioni** prima di avviare l'app con uno degli approcci seguenti:

* Selezionare il pulsante **Run** (Esegui).
* Usare **debug**  >  **Avvia debug** dal menu.
* Premere <kbd>F5</kbd>.

### <a name="inspect-the-content-of-a-json-web-token-jwt"></a>Esaminare il contenuto di un token JSON Web (JWT)

Per decodificare un token JSON Web (JWT), usare lo strumento [JWT.ms](https://jwt.ms/) di Microsoft. I valori nell'interfaccia utente non lasciano mai il browser.

Esempio di JWT codificato (abbreviato per la visualizzazione):

> eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImtpZCI6Ilg1ZVhrNHh5b2pORnVtMWtsMll0djhkbE5QNC1j ... bQdHBHGcQQRbW7Wmo6SWYG4V_bU55Ug_PW4pLPr20tTS8Ct7_uwy9DWrzCMzpD-EiwT5IjXwlGX3IXVjHIlX50IVIydBoPQtadvT7saKo1G5Jmutgq41o-dmz6-yBMKV2_nXA25Q

Esempio di JWT decodificato dallo strumento per un'app che esegue l'autenticazione con AAD B2C di Azure:

```json
{
  "typ": "JWT",
  "alg": "RS256",
  "kid": "X5eXk4xyojNFum1kl2Ytv8dlNP4-c57dO6QGTVBwaNk"
}.{
  "exp": 1610059429,
  "nbf": 1610055829,
  "ver": "1.0",
  "iss": "https://mysiteb2c.b2clogin.com/5cc15ea8-a296-4aa3-97e4-226dcc9ad298/v2.0/",
  "sub": "5ee963fb-24d6-4d72-a1b6-889c6e2c7438",
  "aud": "70bde375-fce3-4b82-984a-b247d823a03f",
  "nonce": "b2641f54-8dc4-42ca-97ea-7f12ff4af871",
  "iat": 1610055829,
  "auth_time": 1610055822,
  "idp": "idp.com",
  "tfp": "B2C_1_signupsignin"
}.[Signature]
```
