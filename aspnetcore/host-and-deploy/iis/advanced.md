---
title: Configurazione avanzata
author: rick-anderson
description: Configurazione avanzata con il modulo ASP.NET Core e il Internet Information Services (IIS).
monikerRange: '>= aspnetcore-5.0'
ms.author: riande
ms.custom: mvc
ms.date: 5/7/2020
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
uid: host-and-deploy/iis/advanced
ms.openlocfilehash: de3abb7b5f89e3b84da38a8eabb183f9551cc1f9
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102588919"
---
# <a name="advanced-configuration-of-the-aspnet-core-module-and-iis"></a>Configurazione avanzata del modulo ASP.NET Core e di IIS

Questo articolo illustra le opzioni di configurazione avanzate e gli scenari per il modulo ASP.NET Core e IIS.

## <a name="modify-the-stack-size"></a>Modificare le dimensioni dello stack

*Si applica solo quando si usa il modello di hosting in-process.*

Configurare la dimensione dello stack gestito usando l' `stackSize` impostazione in byte nel `web.config` file. Le dimensioni predefinite sono pari a 1.048.576 byte (1 MB). Nell'esempio seguente le dimensioni dello stack vengono modificate in 2 MB (2.097.152 byte):

```xml
<aspNetCore processPath="dotnet"
    arguments=".\MyApp.dll"
    stdoutLogEnabled="false"
    stdoutLogFile="\\?\%home%\LogFiles\stdout"
    hostingModel="inprocess">
  <handlerSettings>
    <handlerSetting name="stackSize" value="2097152" />
  </handlerSettings>
</aspNetCore>
```

## <a name="proxy-configuration-uses-http-protocol-and-a-pairing-token"></a>La configurazione del proxy usa il protocollo HTTP e un token di associazione

*Si applica solo all'hosting out-of-process.*

Il proxy creato tra il modulo ASP.NET Core e Kestrel usa il protocollo HTTP. Non vi è alcun rischio di intercettazione del traffico tra il modulo e Kestrel da una posizione esterna al server.

Viene usato un token di associazione per garantire che le richieste ricevute da Kestrel siano state trasmesse tramite proxy da IIS e non provengano da un'altra origine. Il token di associazione viene creato e impostato in una variabile di ambiente (`ASPNETCORE_TOKEN`) dal modulo. Il token di associazione viene impostato anche in un'intestazione (`MS-ASPNETCORE-TOKEN`) in tutte le richieste trasmesse tramite proxy. Il middleware di IIS controlla ogni richiesta che riceve in modo da confermare che il valore dell'intestazione del token di associazione corrisponda al valore della variabile di ambiente. Se i valori del token non corrispondono, la richiesta viene registrata e rifiutata. La variabile di ambiente del token di associazione e il traffico tra il modulo e Kestrel non sono accessibili da una posizione esterna al server. Senza conoscere il valore del token di associazione, un utente malintenzionato non può inviare richieste che ignorano il controllo nel middleware di IIS.

## <a name="aspnet-core-module-with-an-iis-shared-configuration"></a>Modulo ASP.NET Core con configurazione condivisa di IIS

Il programma di installazione del modulo ASP.NET Core viene eseguito con i privilegi dell' `TrustedInstaller` account. Poiché l'account di sistema locale non dispone dell'autorizzazione di modifica per il percorso di condivisione utilizzato dalla configurazione condivisa di IIS, il programma di installazione genera un errore di accesso negato quando si tenta di configurare le impostazioni del modulo nel `applicationHost.config` file nella condivisione.

Quando si usa una configurazione condivisa di IIS nello stesso computer dell'installazione di IIS, eseguire il programma di installazione del bundle di hosting ASP.NET Core con il parametro `OPT_NO_SHARED_CONFIG_CHECK` impostato su `1`:

```console
dotnet-hosting-{VERSION}.exe OPT_NO_SHARED_CONFIG_CHECK=1
```

Quando il percorso della configurazione condivisa non è nello stesso computer dell'installazione di IIS, seguire questa procedura:

1. Disabilitare la configurazione condivisa di IIS.
1. Eseguire il programma di installazione.
1. Esportare il file aggiornato nella `applicationHost.config` condivisione file.
1. Abilitare di nuovo la configurazione condivisa di IIS.

## <a name="data-protection"></a>Protezione dei dati

Lo [stack di protezione dei dati di ASP.NET Core](xref:security/data-protection/introduction) viene usato da diversi [middleware](xref:fundamentals/middleware/index) di ASP.NET Core, inclusi quelli usati nell'autenticazione. Anche se le DPAPI (Data Protection API) non vengono chiamate dal codice dell'utente, è consigliabile configurare la protezione dati per la creazione di un [archivio di chiavi](xref:security/data-protection/implementation/key-management) crittografiche permanente con uno script di distribuzione o nel codice dell'utente. Se non si configura la protezione dei dati, le chiavi vengono mantenute in memoria ed eliminate al riavvio dell'app.

Se l'anello della chiave di protezione dei dati viene archiviato in memoria quando l'app viene riavviata:

* Tutti i cookie token di autenticazione basati su vengono invalidati. 
* Gli utenti devono ripetere l'accesso alla richiesta successiva. 
* Tutti i dati protetti con il gruppo di chiavi non possono più essere decrittografati. Questo può includere [i token CSRF](xref:security/anti-request-forgery#aspnet-core-antiforgery-configuration) e [ASP.NET Core TempData MVC cookie ](xref:fundamentals/app-state#tempdata).

Per configurare la protezione dati in IIS in modo da rendere permanente il gruppo di chiavi, usare **uno** degli approcci seguenti:

* **Creare chiavi del registro di sistema per la protezione dei dati**

  Le chiavi di protezione dei dati usate dalle app ASP.NET Core vengono archiviate nel registro di sistema esterno alle app. Per salvare in modo permanente le chiavi per una determinata app, creare le chiavi del registro di sistema per il pool di app.

  Per le installazioni IIS autonome non web farm è possibile usare lo [script PowerShell Data Protection Provision-AutoGenKeys.ps1 (Provisioning di protezione dati-AutoGenKeys.ps1)](https://github.com/dotnet/AspNetCore/blob/main/src/DataProtection/Provision-AutoGenKeys.ps1) per ogni pool di app usato con un'app ASP.NET Core. Questo script crea una chiave del registro di sistema nel registro di sistema HKLM accessibile solo all'account del processo di lavoro del pool di applicazioni dell'app. Le chiavi vengono crittografate a riposo tramite DPAPI con una chiave a livello del computer.

  In Web farm scenari, un'app può essere configurata in modo da usare un percorso UNC per archiviare il relativo anello chiave di protezione dati. Per impostazione predefinita, le chiavi non sono crittografate. Assicurarsi che le autorizzazioni per i file per la condivisione di rete siano limitate all'account di Windows con cui viene eseguita l'app. È possibile usare un certificato X509 per la protezione delle chiavi inattive. Si consideri un meccanismo per consentire agli utenti di caricare i certificati. Posizionare i certificati nell'archivio certificati attendibili dell'utente e assicurarsi che siano disponibili in tutti i computer in cui viene eseguita l'app dell'utente. Per altre informazioni, vedere <xref:security/data-protection/configuration/overview>.

* **Configurare il pool di applicazioni IIS per caricare il profilo utente**

  Questa impostazione si trova nella sezione **Modello del processo** in **Impostazioni avanzate** per il pool di app. Impostare **Caricamento del profilo utente** su `True`. Con l'impostazione `True` le chiavi vengono archiviate nella directory del profilo utente e protette tramite DPAPI con una chiave specifica per l'account utente. Le chiavi vengono salvate in modo permanente nella `%LOCALAPPDATA%/ASP.NET/DataProtection-Keys` cartella.

  È necessario abilitare anche l' [ `setProfileEnvironment` attributo](/iis/configuration/system.applicationhost/applicationpools/add/processmodel#configuration) del pool di app. Il valore predefinito di `setProfileEnvironment` è `true`. In alcuni scenari (ad esempio, per il sistema operativo Windows), `setProfileEnvironment` è impostato su `false`. Se le chiavi non vengono archiviate nella directory del profilo utente come previsto:

  1. Passa alla cartella `%windir%/system32/inetsrv/config`.
  1. Aprire il file `applicationHost.config`.
  1. Individuare l'elemento `<system.applicationHost><applicationPools><applicationPoolDefaults><processModel>`.
  1. Verificare che l'attributo `setProfileEnvironment` non sia presente, condizione che corrisponde all'impostazione predefinita `true`, o impostare in modo esplicito il valore dell'attributo su `true`.

* **Usare il file system come archivio del gruppo di chiavi**

  Modificare il codice dell'app per [usare il file system come archivio del gruppo di chiavi](xref:security/data-protection/configuration/overview). Usare un certificato X509 per proteggere il gruppo di chiavi e verificare che sia un certificato attendibile. Se il certificato è autofirmato, inserirlo nell'archivio radice attendibile.

  Quando si usa IIS in una web farm:

  * Usare una condivisione di file a cui possono accedere tutti i computer.
  * Distribuire un certificato X509 in ogni computer. Configurare la [protezione dei dati nel codice](xref:security/data-protection/configuration/overview).

* **Impostare criteri a livello di computer per la protezione dei dati**

  Il sistema di protezione dei dati ha un supporto limitato per l'impostazione di [criteri a livello di computer](xref:security/data-protection/configuration/machine-wide-policy) predefiniti per tutte le app che utilizzano le API di protezione dei dati. Per altre informazioni, vedere <xref:security/data-protection/introduction>.

## <a name="iis-configuration"></a>Configurazione di IIS

**Sistemi operativi Windows Server**

Abilitare il ruolo del server **Server Web (IIS)** e stabilire i servizi di ruolo.

1. Usare la procedura guidata **Aggiungi ruoli e funzionalità** accessibile tramite il menu **Gestisci** o il collegamento in **Server Manager**. Nel passaggio **Ruoli del server** selezionare la casella per **Server Web (IIS)**.

   ![Il ruolo Server Web IIS viene selezionato nel passaggio dei ruoli del server Selezionare.](index/_static/server-roles-ws2016.png)

1. Dopo il passaggio **Funzionalità**, viene caricato il passaggio **Servizi ruolo** per Server Web (IIS). Selezionare i servizi ruolo IIS desiderati o accettare i servizi ruolo predefiniti specificati.

   ![I servizi ruolo predefiniti vengono selezionati nel passaggio Selezionare i servizi ruolo.](index/_static/role-services-ws2016.png)

   **Autenticazione di Windows (facoltativa)**  
   Per abilitare l'autenticazione di Windows, espandere i nodi seguenti: sicurezza del **server Web**  >  . Selezionare la funzionalità **Autenticazione di Windows**. Per ulteriori informazioni, vedere [ `<windowsAuthentication>` autenticazione di Windows](/iis/configuration/system.webServer/security/authentication/windowsAuthentication/) e [configurare l'autenticazione di Windows](xref:security/authentication/windowsauth).

   **WebSockets (facoltativo)**  
   WebSockets è supportato con ASP.NET Core 1.1 o versioni successive. Per abilitare i WebSocket, espandere i nodi seguenti: sviluppo di applicazioni **server Web**  >  . Selezionare la funzionalità **Protocollo WebSocket**. Per altre informazioni, vedere [Oggetti WebSocket](xref:fundamentals/websockets).

1. Procedere con il passaggio **Conferma** per installare il ruolo del server web e i servizi. Dopo l'installazione del ruolo **server Web (IIS)** non è necessario riavviare il server o IIS.

**Sistemi operativi desktop di Windows**

Abilitare **Console di gestione IIS** e **Servizi Web**.

1. Passare a **Pannello di controllo**  >  **programmi** programmi  >  **e funzionalità**  >  **attiva o disattiva funzionalità Windows** (lato sinistro dello schermo).

1. Aprire il nodo **Internet Information Services**. Aprire il nodo **Strumenti di gestione Web**.

1. Selezionare la casella per **Console di gestione IIS**.

1. Selezionare la casella per **Servizi World Wide Web**.

1. Accettare le funzionalità predefinite per **Servizi World Wide Web** o personalizzare le funzionalità IIS.

   **Autenticazione di Windows (facoltativa)**  
   Per abilitare l'autenticazione di Windows, espandere i nodi seguenti: **World Wide Web sicurezza dei servizi**  >  . Selezionare la funzionalità **Autenticazione di Windows**. Per ulteriori informazioni, vedere [ `<windowsAuthentication>` autenticazione di Windows](/iis/configuration/system.webServer/security/authentication/windowsAuthentication/) e [configurare l'autenticazione di Windows](xref:security/authentication/windowsauth).

   **WebSockets (facoltativo)**  
   WebSockets è supportato con ASP.NET Core 1.1 o versioni successive. Per abilitare i WebSocket, espandere i nodi seguenti: funzionalità di sviluppo di applicazioni di **World Wide Web Services**  >  . Selezionare la funzionalità **Protocollo WebSocket**. Per altre informazioni, vedere [Oggetti WebSocket](xref:fundamentals/websockets).

1. Se l'installazione di IIS richiede un riavvio, riavviare il sistema.

![La console di gestione IIS e servizi World Wide Web sono selezionati nelle funzionalità di Windows.](index/_static/windows-features-win10.png)

## <a name="virtual-directories"></a>Directory virtuali

Le [directory virtuali IIS](/iis/get-started/planning-your-iis-architecture/understanding-sites-applications-and-virtual-directories-on-iis#virtual-directories) non sono supportate con le app ASP.NET Core. Un'app può essere ospitata come [applicazione secondaria](#sub-applications).

## <a name="sub-applications"></a>Applicazioni secondarie

Un'app ASP.NET Core può essere ospitata come [applicazione secondaria di IIS (app secondaria)](/iis/get-started/planning-your-iis-architecture/understanding-sites-applications-and-virtual-directories-on-iis#applications). Il percorso dell'app secondaria diventa parte dell'URL dell'app radice.

I collegamenti ad asset statici all'interno dell'app secondaria devono usare la notazione con tilde-barra (`~/`). La notazione con tilde-barra attiva un [helper tag](xref:mvc/views/tag-helpers/intro) per anteporre la base del percorso dell'app secondaria al collegamento relativo sottoposto a rendering. Per un'app secondaria in `/subapp_path`, il rendering di un'immagine collegata con `src="~/image.png"` viene eseguito come `src="/subapp_path/image.png"`. Il middleware dei file statici dell'app radice non elabora la richiesta di file statici. La richiesta viene elaborata dal middleware dei file statici dell'app secondaria.

Se l'attributo `src` di un asset statico viene impostato su un percorso assoluto (ad esempio, `src="/image.png"`), il rendering del collegamento viene eseguito senza la base del percorso dell'app secondaria. Il middleware dei file statici dell'app radice prova a servire l'asset dalla [radice Web](xref:fundamentals/index#web-root) dell'app radice con conseguente risposta *404 - Non trovato*, a meno che l'asset statico non sia disponibile dal'app radice.

Per ospitare un'app ASP.NET Core come app secondaria in un'altra app ASP.NET Core:

1. Definire un pool di app per l'app secondaria. Impostare **Versione .NET CLR** su **Nessun codice gestito** perché Core Common Language Runtime (CoreCLR) for .NET Core viene avviato per ospitare l'app nel processo di lavoro e non CLR desktop (.NET CLR).

1. Aggiungere il sito radice in Gestione IIS con l'applicazione secondaria in una cartella all'interno del sito principale.

1. Fare clic con il pulsante destro del mouse sulla cartella dell'applicazione secondaria in Gestione IIS e scegliere **Converti in applicazione**.

1. Nella finestra di dialogo **Aggiungi applicazione** usare il pulsante **Seleziona** per **Pool di applicazioni** per assegnare il pool di app creato per l'app secondaria. Selezionare **OK**.

L'assegnazione di un pool di app separato all'app secondaria è un requisito quando si usa il modello di hosting in-process.

Per ulteriori informazioni sul modello di hosting in-process e sulla configurazione del modulo ASP.NET Core, vedere <xref:host-and-deploy/aspnet-core-module> .

## <a name="application-pools"></a>Pool di applicazioni

L'isolamento dei pool di app è determinato dal modello di hosting:

* Hosting in-process: le app sono necessarie per l'esecuzione in pool di applicazioni separati.
* Hosting out-of-process: è consigliabile isolare le app le une dalle altre eseguendo ogni app nel proprio pool di applicazioni.

Nella finestra di dialogo **Aggiungi sito Web** è selezionato per impostazione predefinita un singolo pool di app per ogni app. Quando si specifica un valore in **Nome del sito**, il testo viene automaticamente trasferito alla casella di testo **Pool di applicazioni**. Quando si aggiunge il sito viene creato un nuovo pool di applicazioni con il nome del sito.

## <a name="application-pool-identity"></a>Pool di applicazioni Identity

Un account di identità del pool di app consente di eseguire un'app in un account univoco, senza dover creare e gestire domini o account locali. In IIS 8.0 o versioni successive il processo di lavoro amministrazione IIS (WAS) crea un account virtuale con il nome del nuovo pool di applicazioni ed esegue i processi di lavoro del pool di applicazioni inclusi nell'account per impostazione predefinita. Nella console di gestione IIS in **Impostazioni avanzate** per il pool di applicazioni, verificare che **Identity** sia impostato su Usa `ApplicationPoolIdentity` :

![Finestra di dialogo Impostazioni avanzate del pool di applicazione](index/_static/apppool-identity.png)

Il processo di gestione IIS crea un identificatore sicuro con il nome del pool di app nel sistema di sicurezza di Windows. Le risorse possono essere protette mediante questa identità, che tuttavia non è un account utente reale e non viene visualizzata nella console di gestione utenti di Windows.

Se il processo di lavoro IIS richiede l'accesso con privilegi elevati all'app, modificare l'elenco di controllo di accesso (ACL) della directory contenente l'app:

1. Aprire Esplora risorse, quindi spostarsi nella directory.

1. Fare clic con il pulsante destro del mouse sulla directory e scegliere **Proprietà**.

1. Nella scheda **Sicurezza** selezionare il pulsante **Modifica** e quindi il pulsante **Aggiungi**.

1. Fare clic sul pulsante **Percorsi** e verificare che il sistema sia selezionato.

1. Immettere il `IIS AppPool\{APP POOL NAME}` formato, dove il segnaposto `{APP POOL NAME}` è il nome del pool di applicazioni, in **immettere i nomi degli oggetti da selezionare** . Fare clic sul pulsante **Controlla nomi**. Per *DefaultAppPool* controllare i nomi usando `IIS AppPool\DefaultAppPool` . Quando si seleziona il pulsante **Controlla nomi** , il valore `DefaultAppPool` è indicato nell'area dei nomi degli oggetti. Non è possibile immettere il nome del pool di applicazioni direttamente nell'area dei nomi degli oggetti. Usare il `IIS AppPool\{APP POOL NAME}` formato, dove il segnaposto `{APP POOL NAME}` è il nome del pool di applicazioni, quando si verifica il nome dell'oggetto.

   ![Finestra di dialogo Seleziona utenti o gruppi per la cartella dell'app: il nome del pool di applicazioni "DefaultAppPool" viene aggiunto in coda a "IIS AppPool\" nell'area dei nomi degli oggetti prima di selezionare "Controlla nomi".](index/_static/select-users-or-groups-1.png)

1. Selezionare **OK**.

   ![Finestra di dialogo Seleziona utenti o gruppi per la cartella dell'app: dopo aver selezionato "Controlla nomi", il nome dell'oggetto "DefaultAppPool" viene visualizzato nell'area dei nomi degli oggetti.](index/_static/select-users-or-groups-2.png)

1. Le autorizzazioni di lettura ed esecuzione devono essere concesse per impostazione predefinita. Fornire autorizzazioni aggiuntive in base alle necessità.

L'accesso può essere concesso anche dal prompt dei comandi usando lo strumento **ICACLS**. Usando *DefaultAppPool* come esempio, viene eseguito il comando seguente:

```console
ICACLS C:\sites\MyWebApp /grant "IIS AppPool\DefaultAppPool":F
```

Per altre informazioni, vedere l'argomento [icacls](/windows-server/administration/windows-commands/icacls).

## <a name="http2-support"></a>Supporto HTTP/2

[HTTP/2](https://httpwg.org/specs/rfc7540.html) è supportato con ASP.NET Core negli scenari di distribuzione IIS seguenti:

* In-Process
  * Windows Server 2016/Windows 10 o versioni successive; IIS 10 o versioni successive
  * Connessione TLS 1.2 o successiva
* Out-of-process
  * Windows Server 2016/Windows 10 o versioni successive; IIS 10 o versioni successive
  * Le connessioni server perimetrali pubbliche usano HTTP/2, mentre la connessione con proxy inverso al [server Kestrel](xref:fundamentals/servers/kestrel) usa HTTP/1.1.
  * Connessione TLS 1.2 o successiva

Per una distribuzione in-process quando viene stabilita una connessione HTTP/2, i [`HttpRequest.Protocol`](xref:Microsoft.AspNetCore.Http.HttpRequest.Protocol*) report `HTTP/2` . Per una distribuzione out-of-process quando viene stabilita una connessione HTTP/2, i [`HttpRequest.Protocol`](xref:Microsoft.AspNetCore.Http.HttpRequest.Protocol*) report `HTTP/1.1` .

Per altre informazioni sui modelli di hosting in-process e out-of-process, vedere <xref:host-and-deploy/aspnet-core-module>.

HTTP/2 è abilitato per impostazione predefinita. Se non viene stabilita una connessione HTTP/2, la connessione esegue il fallback a HTTP/1.1. Per altre informazioni sulla configurazione di HTTP/2 con le distribuzioni IIS, vedere [HTTP/2 on IIS](/iis/get-started/whats-new-in-iis-10/http2-on-iis) (HTTP/2 in IIS).

## <a name="cors-preflight-requests"></a>Richieste preliminari CORS

*Questa sezione si applica solo alle app ASP.NET Core destinate a .NET Framework.*

Per un'app ASP.NET Core destinata a .NET Framework, le richieste OPTIONS non vengono passate all'app per impostazione predefinita in IIS. Per informazioni su come configurare i gestori IIS dell'app in `web.config` per passare le richieste di opzioni, vedere [abilitare le richieste tra le origini in API Web ASP.NET 2:](/aspnet/web-api/overview/security/enabling-cross-origin-requests-in-web-api#how-cors-works)funzionamento di CORS.

## <a name="application-initialization-module-and-idle-timeout"></a>Modulo di inizializzazione dell'applicazione e timeout di inattività

Per l'hosting in IIS dal modulo ASP.NET Core versione 2:

* [Modulo di inizializzazione dell'applicazione](#application-initialization-module): l'app è ospitata [in-process](xref:host-and-deploy/iis/in-process-hosting) o [out-of-process](xref:host-and-deploy/iis/out-of-process-hosting) può essere configurata per l'avvio automatico in un processo di lavoro o il riavvio del server.
* [Timeout di inattività](#idle-timeout): è possibile configurare l'app ospitata [in-process in](xref:host-and-deploy/iis/in-process-hosting) modo che non si timeout durante i periodi di inattività.

### <a name="application-initialization-module"></a>Modulo Inizializzazione applicazione

*Si applica alle app ospitate in-process e out-of-process.*

[Inizializzazione applicazione di IIS](/iis/get-started/whats-new-in-iis-8/iis-80-application-initialization) è una funzionalità di IIS che invia una richiesta HTTP all'app quando il pool di app viene avviato o riciclato. La richiesta attiva l'avvio dell'app. Per impostazione predefinita, IIS invia una richiesta all'URL radice dell'app (`/`) per inizializzare l'app (vedere le [risorse aggiuntive](#application-initialization-module-and-idle-timeout-additional-resources) per altri dettagli sulla configurazione).

Verificare che la funzionalità del ruolo Inizializzazione applicazione di IIS sia abilitata:

Nei sistemi desktop Windows 7 o versioni successive quando si usa IIS in locale:

1. Passare a **Pannello di controllo**  >  **programmi** programmi  >  **e funzionalità**  >  **attiva o disattiva funzionalità Windows** (lato sinistro dello schermo).
1. Aprire **Internet Information Services**  >  funzionalità di sviluppo di applicazioni **World Wide Web Services**  >  .
1. Selezionare la casella di controllo **Inizializzazione applicazione**.

In Windows Server 2008 R2 o versioni successive:

1. Aprire l'**Aggiunta guidata ruoli e funzionalità**.
1. Nel pannello **Selezione servizi ruolo** aprire il nodo **Sviluppo applicazioni**.
1. Selezionare la casella di controllo **Inizializzazione applicazione**.

Per abilitare il modulo Inizializzazione applicazione per il sito, usare uno degli approcci seguenti:

* In Gestione IIS:

  1. Selezionare **Pool di applicazioni** nel pannello **Connessioni**.
  1. Fare clic con il pulsante destro del mouse sul pool di app dell'app nell'elenco e scegliere **Impostazioni avanzate**.
  1. La **modalità di avvio** predefinita è `OnDemand` . Impostare la **modalità di avvio** su `AlwaysRunning` . Selezionare **OK**.
  1. Aprire il nodo **Siti** nel pannello **Connessioni**.
  1. Fare clic con il pulsante destro del mouse sull'app e scegliere **Gestisci**  >  **Impostazioni avanzate** sito Web.
  1. L'impostazione predefinita di **precaricamento abilitato** è `False` . Impostare **preload abilitato** su `True` . Selezionare **OK**.

* Usando `web.config` , aggiungere l' `<applicationInitialization>` elemento con `doAppInitAfterRestart` impostato su `true` agli `<system.webServer>` elementi nel file dell'app `web.config` :

  ```xml
  <?xml version="1.0" encoding="utf-8"?>
  <configuration>
    <location path="." inheritInChildApplications="false">
      <system.webServer>
        <applicationInitialization doAppInitAfterRestart="true" />
      </system.webServer>
    </location>
  </configuration>
  ```

### <a name="idle-timeout"></a>Timeout di inattività

*Si applica solo alle app ospitate in-process.*

Per evitare l'inattività dell'app, impostare il timeout di inattività del pool di app tramite Gestione IIS:

1. Selezionare **Pool di applicazioni** nel pannello **Connessioni**.
1. Fare clic con il pulsante destro del mouse sul pool di app dell'app nell'elenco e scegliere **Impostazioni avanzate**.
1. Il **timeout di inattività predefinito (minuti)** è `20` minuti. Imposta il **timeout di inattività (minuti)** su `0` (zero). Selezionare **OK**.
1. Riciclare il processo di lavoro.

Per evitare il timeout delle app ospitate [out-of-process](xref:host-and-deploy/iis/out-of-process-hosting), usare uno degli approcci seguenti:

* Effettuare il ping dell'app da un servizio esterno per mantenerla in esecuzione.
* Se l'app ospita solo servizi in background, evitare l'hosting IIS e usare un [servizio Windows per ospitare l'app ASP.NET Core](xref:host-and-deploy/windows-service).

### <a name="application-initialization-module-and-idle-timeout-additional-resources"></a>Risorse aggiuntive per il modulo Inizializzazione applicazione e il timeout di inattività

* [IIS 8.0 Application Initialization](/iis/get-started/whats-new-in-iis-8/iis-80-application-initialization) (Inizializzazione delle applicazioni in IIS 8.0)
* [Inizializzazione `<applicationInitialization>` dell'applicazione ](/iis/configuration/system.webserver/applicationinitialization/).
* [Impostazioni del modello di processo per un `<processModel>` pool di applicazioni ](/iis/configuration/system.applicationhost/applicationpools/add/processmodel).

## <a name="module-schema-and-configuration-file-locations"></a>Percorsi dei file di modulo, schema e configurazione

### <a name="module"></a>Modulo

**IIS (x86/amd64)**:

* `%windir%\System32\inetsrv\aspnetcore.dll`

* `%windir%\SysWOW64\inetsrv\aspnetcore.dll`

* `%ProgramFiles%\IIS\Asp.Net Core Module\V2\aspnetcorev2.dll`

* `%ProgramFiles(x86)%\IIS\Asp.Net Core Module\V2\aspnetcorev2.dll`

**IIS Express (x86/amd64)**:

* `%ProgramFiles%\IIS Express\aspnetcore.dll`

* `%ProgramFiles(x86)%\IIS Express\aspnetcore.dll`

* `%ProgramFiles%\IIS Express\Asp.Net Core Module\V2\aspnetcorev2.dll`

* `%ProgramFiles(x86)%\IIS Express\Asp.Net Core Module\V2\aspnetcorev2.dll`

### <a name="schema"></a>SCHEMA

**IIS**

* `%windir%\System32\inetsrv\config\schema\aspnetcore_schema.xml`

* `%windir%\System32\inetsrv\config\schema\aspnetcore_schema_v2.xml`

**IIS Express**

* `%ProgramFiles%\IIS Express\config\schema\aspnetcore_schema.xml`

* `%ProgramFiles%\IIS Express\config\schema\aspnetcore_schema_v2.xml`

### <a name="configuration"></a>Configurazione

**IIS**

* `%windir%\System32\inetsrv\config\applicationHost.config`

**IIS Express**

* Visual Studio: `{APPLICATION ROOT}\.vs\config\applicationHost.config`

* *iisexpress.exe* CLI `%USERPROFILE%\Documents\IISExpress\config\applicationhost.config`

È possibile trovare i file eseguendo una ricerca `aspnetcore` nel `applicationHost.config` file.
