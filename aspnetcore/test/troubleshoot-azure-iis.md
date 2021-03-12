---
title: Risolvere i problemi relativi a ASP.NET Core in app Azure servizio e IIS
author: rick-anderson
description: Informazioni su come diagnosticare i problemi relativi alle distribuzioni del servizio app Azure e Internet Information Services (IIS) delle app di ASP.NET Core.
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 02/07/2020
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
uid: test/troubleshoot-azure-iis
ms.openlocfilehash: 3f6b74ee9621529943d9f685d803837e7df10cfc
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102589569"
---
# <a name="troubleshoot-aspnet-core-on-azure-app-service-and-iis"></a>Risolvere i problemi relativi a ASP.NET Core in app Azure servizio e IIS

Di [Justin Kotalik](https://github.com/jkotalik)

::: moniker range=">= aspnetcore-3.0"

Questo articolo fornisce informazioni sugli errori comuni di avvio delle app e istruzioni su come diagnosticare gli errori quando un'app viene distribuita in app Azure servizio o IIS:

[Errori di avvio dell'app](#app-startup-errors)  
Vengono illustrati gli scenari di codice di stato HTTP di avvio comuni.

[Risolvere i problemi relativi al servizio app Azure](#troubleshoot-on-azure-app-service)  
Fornisce consigli per la risoluzione dei problemi per le app distribuite nel servizio app Azure.

[Risolvere i problemi in IIS](#troubleshoot-on-iis)  
Fornisce consigli per la risoluzione dei problemi per le app distribuite in IIS o in esecuzione in IIS Express localmente. Le linee guida sono valide per le distribuzioni di Windows Server e Windows desktop.

[Cancella cache di pacchetti](#clear-package-caches)  
Spiega cosa fare quando i pacchetti incoerenti interrompono un'app durante l'esecuzione di aggiornamenti principali o la modifica delle versioni del pacchetto.

[Risorse aggiuntive](#additional-resources)  
Elenca ulteriori argomenti sulla risoluzione dei problemi.

## <a name="app-startup-errors"></a>Errori di avvio dell'app

In Visual Studio, per impostazione predefinita, un progetto ASP.NET Core viene [IIS Express](/iis/extensions/introduction-to-iis-express/iis-express-overview) host durante il debug. Un errore *502,5-Process* o un *errore di avvio 500,30* che si verifica quando il debug locale può essere diagnosticato utilizzando i consigli in questo argomento.

### <a name="40314-forbidden"></a>403,14 non consentito

Non è possibile avviare l'app. Viene registrato l'errore seguente:

```
The Web server is configured to not list the contents of this directory.
```

L'errore è in genere causato da una distribuzione interruppe sul sistema host, che include uno degli scenari seguenti:

* L'app viene distribuita nella cartella errata del sistema di hosting.
* Il processo di distribuzione non è riuscito a spostare tutti i file e le cartelle dell'app nella cartella di distribuzione nel sistema di hosting.
* Il file di *web.config* non è presente nella distribuzione oppure il contenuto del file di *web.config* non è valido.

Eseguire la procedura seguente:

1. Eliminare tutti i file e le cartelle dalla cartella di distribuzione nel sistema di hosting.
1. Ridistribuire il contenuto della cartella di *pubblicazione* dell'app nel sistema host usando il normale metodo di distribuzione, ad esempio Visual Studio, PowerShell o la distribuzione manuale:
   * Verificare che il file *web.config* sia presente nella distribuzione e che il relativo contenuto sia corretto.
   * Quando si esegue l'hosting in app Azure servizio, verificare che l'app venga distribuita nella `D:\home\site\wwwroot` cartella.
   * Quando l'app è ospitata da IIS, verificare che l'app venga distribuita nel **percorso fisico** IIS visualizzato nelle impostazioni di **base** di **Gestione IIS**.
1. Verificare che tutti i file e le cartelle dell'app vengano distribuiti confrontando la distribuzione nel sistema di hosting con il contenuto della cartella di *pubblicazione* del progetto.

Per ulteriori informazioni sul layout di un'app ASP.NET Core pubblicata, vedere <xref:host-and-deploy/directory-structure> . Per ulteriori informazioni sul file di *web.config* , vedere <xref:host-and-deploy/aspnet-core-module#configuration-with-webconfig> .

### <a name="500-internal-server-error"></a>500 - Errore interno del server

L'app viene avviata, ma un errore impedisce al server di soddisfare la richiesta.

Questo errore si verifica all'interno del codice dell'app durante l'avvio o durante la creazione di una risposta. La risposta potrebbe non avere alcun contenuto o essere visualizzata nel browser come un codice *500 Errore interno del server*. Il log eventi dell'applicazione in genere indica che l'app è stata avviata normalmente. Dal punto di vista del server, questo è corretto. L'app è stata effettivamente avviata, ma non può generare una risposta valida. Per risolvere il problema, eseguire l'app dal prompt dei comandi nel server o abilitare il log stdout del modulo ASP.NET Core.

### <a name="5000-in-process-handler-load-failure"></a>500.0 Errore di caricamento del gestore in-process

Il processo di lavoro ha esito negativo. L'app non viene avviata.

Si è verificato un errore sconosciuto durante il caricamento dei componenti del [modulo ASP.NET Core](xref:host-and-deploy/aspnet-core-module) . eseguendo una delle operazioni seguenti:

* Contattare il [supporto tecnico Microsoft](https://support.microsoft.com/oas/default.aspx?prid=15832). Selezionare **Strumenti di sviluppo** e quindi **ASP.NET Core**.
* Porre una domanda in Stack Overflow.
* Segnalare il problema nel [repository GitHub](https://github.com/dotnet/AspNetCore) di ASP.NET Core.

### <a name="50030-in-process-startup-failure"></a>500.30 Errore di avvio in-process

Il processo di lavoro ha esito negativo. L'app non viene avviata.

Il [modulo ASP.NET Core](xref:host-and-deploy/aspnet-core-module) tenta di avviare .NET Core CLR in-process, ma non è possibile avviarlo. In genere è possibile determinare la causa di un errore di avvio del processo dalle voci nel log eventi dell'applicazione e nel log stdout del modulo ASP.NET Core.

Condizioni di errore comuni:

* L'app non è configurata correttamente perché la destinazione è una versione di ASP.NET Core Framework condiviso che non è presente. Controllare le versioni del framework condiviso ASP.NET Core installate nel computer di destinazione.
* Utilizzando Azure Key Vault, non sono disponibili autorizzazioni per l'Key Vault. Verificare i criteri di accesso nel Key Vault di destinazione per verificare che siano state concesse le autorizzazioni corrette.

### <a name="50031-ancm-failed-to-find-native-dependencies"></a>500.31 ANCM Failed to Find Native Dependencies (500.31 Il modulo ASP.NET Core non è riuscito a trovare le dipendenze native)

Il processo di lavoro ha esito negativo. L'app non viene avviata.

Il [modulo ASP.NET Core](xref:host-and-deploy/aspnet-core-module) tenta di avviare il runtime di .NET Core in-process, ma non è possibile avviarlo. La causa più comune di questo errore di avvio è la mancata installazione del runtime di `Microsoft.NETCore.App` o di `Microsoft.AspNetCore.App`. Se l'app viene distribuita per ASP.NET Core 3.0 e tale versione non esiste nel computer, si verifica questo errore. Ecco un esempio di messaggio di errore:

```
The specified framework 'Microsoft.NETCore.App', version '3.0.0' was not found.
  - The following frameworks were found:
      2.2.1 at [C:\Program Files\dotnet\x64\shared\Microsoft.NETCore.App]
      3.0.0-preview5-27626-15 at [C:\Program Files\dotnet\x64\shared\Microsoft.NETCore.App]
      3.0.0-preview6-27713-13 at [C:\Program Files\dotnet\x64\shared\Microsoft.NETCore.App]
      3.0.0-preview6-27714-15 at [C:\Program Files\dotnet\x64\shared\Microsoft.NETCore.App]
      3.0.0-preview6-27723-08 at [C:\Program Files\dotnet\x64\shared\Microsoft.NETCore.App]
```

Il messaggio di errore elenca tutte le versioni di .NET Core installate e la versione richiesta dall'app. Per correggere l'errore, eseguire una delle operazioni seguenti:

* Installare la versione appropriata di .NET Core nel computer.
* Modificare l'app in modo che usi una versione di .NET Core presente nel computer.
* Pubblicare l'app come [distribuzione autonoma](/dotnet/core/deploying/#self-contained-deployments-scd).

Durante l'esecuzione in fase di sviluppo (con la variabile di ambiente `ASPNETCORE_ENVIRONMENT` impostata su `Development`), l'errore specifico viene scritto nella risposta HTTP. La causa di un errore di avvio del processo è disponibile anche nel log eventi dell'applicazione.

### <a name="50032-ancm-failed-to-load-dll"></a>500.32 ANCM Failed to Load dll (500.32 Il modulo ASP.NET Core non è riuscito a caricare la DLL)

Il processo di lavoro ha esito negativo. L'app non viene avviata.

La causa più comune di questo errore è la pubblicazione dell'app per processori con architettura non compatibile. Se il processo di lavoro è in esecuzione come app a 32 bit e l'app è stata pubblicata per un'architettura a 64 bit, si verifica questo errore.

Per correggere l'errore, eseguire una delle operazioni seguenti:

* Ripubblicare l'app per processori con la stessa architettura del processo di lavoro.
* Pubblicare l'app come [distribuzione dipendente dal framework](/dotnet/core/deploying/#framework-dependent-executables-fde).

### <a name="50033-ancm-request-handler-load-failure"></a>500.33 ANCM Request Handler Load Failure (500.33 Errore di caricamento del gestore della richiesta del modulo ASP.Net Core)

Il processo di lavoro ha esito negativo. L'app non viene avviata.

L'app non fa riferimento al framework `Microsoft.AspNetCore.App`. Solo le app destinate al `Microsoft.AspNetCore.App` Framework possono essere ospitate dal [modulo ASP.NET Core](xref:host-and-deploy/aspnet-core-module).

Per correggere questo errore, verificare che l'app sia destinata al framework `Microsoft.AspNetCore.App`. Controllare il framework di destinazione dell'app nel file `.runtimeconfig.json`.

### <a name="50034-ancm-mixed-hosting-models-not-supported"></a>500.34 ANCM Mixed Hosting Models Not Supported (500.34 Modelli di hosting misto non supportati nel modulo ASP.NET Core)

Il processo di lavoro non è in grado di eseguire un'app In-Process e un'app Out-of-process nello stesso processo.

Per correggere questo errore, eseguire le app in pool di applicazioni IIS separati.

### <a name="50035-ancm-multiple-in-process-applications-in-same-process"></a>500.35 ANCM Multiple In-Process Applications in same Process (500.35 Modulo ASP.NET Core: più applicazioni In-Process nello stesso processo)

Il processo di lavoro non può eseguire più app in-process nello stesso processo.

Per correggere questo errore, eseguire le app in pool di applicazioni IIS separati.

### <a name="50036-ancm-out-of-process-handler-load-failure"></a>500.36 ANCM Out-Of-Process Handler Load Failure (500.36 Modulo ASP.NET Core: Errore di caricamento del gestore Out-of-process)

Il gestore delle richieste Out-of-process, *aspnetcorev2_outofprocess.dll*, non è accanto al file *aspnetcorev2.dll*. Indica un'installazione danneggiata del [modulo ASP.NET Core](xref:host-and-deploy/aspnet-core-module).

Per correggere questo errore, riparare l'installazione del [bundle di hosting di .NET Core](xref:host-and-deploy/iis/index#install-the-net-core-hosting-bundle) (per IIS) o di Visual Studio (per IIS Express).

### <a name="50037-ancm-failed-to-start-within-startup-time-limit"></a>500.37 ANCM Failed to Start Within Startup Time Limit (500.37 Avvio del modulo ASP.NET Core non riuscito entro il limite di tempo stabilito)

MODULO ASP.NET Core non è stato avviato entro il limite di tempo di avvio specificato. Per impostazione predefinita, il timeout è di 120 secondi.

Questo errore può verificarsi quando si avvia un numero elevato di app nello stesso computer. Controllare i picchi di utilizzo della CPU e della memoria nel server durante l'avvio. Può essere necessario scaglionare il processo di avvio di più app.

### <a name="50038-ancm-application-dll-not-found"></a>500,38 DLL applicazione modulo ASP.NET Core non trovata

MODULO ASP.NET Core non è riuscito a individuare la DLL dell'applicazione, che deve essere accanto all'eseguibile.

Questo errore si verifica quando si ospita un'app in pacchetto come file [eseguibile di un singolo file](/dotnet/core/whats-new/dotnet-core-3-0#single-file-executables) usando il modello di hosting in-process. Il modello in-process richiede che modulo ASP.NET Core carichi l'app .NET Core nel processo IIS esistente. Questo scenario non è supportato dal modello di distribuzione a file singolo. Per correggere l'errore, usare **uno** degli approcci seguenti nel file di progetto dell'app:

1. Disabilitare la pubblicazione su file singolo impostando la `PublishSingleFile` Proprietà MSBuild su `false` .
1. Passare al modello di hosting out-of-process impostando la `AspNetCoreHostingModel` Proprietà MSBuild su `OutOfProcess` .

### <a name="5025-process-failure"></a>502.5 Process Failure (Errore del processo)

Il processo di lavoro ha esito negativo. L'app non viene avviata.

Il [modulo ASP.NET Core](xref:host-and-deploy/aspnet-core-module) tenta di avviare il processo di lavoro, ma l'avvio non riesce. In genere è possibile determinare la causa di un errore di avvio del processo dalle voci nel log eventi dell'applicazione e nel log stdout del modulo ASP.NET Core.

Una condizione di errore comune è dovuta alla configurazione non corretta dell'app perché come destinazione viene specificata una versione del framework condiviso ASP.NET Core, che non è presente. Controllare le versioni del framework condiviso ASP.NET Core installate nel computer di destinazione. Il *framework condiviso* è il set di assembly (file *DLL*) che vengono installati nel computer e cui fa riferimento un metapacchetto, ad esempio `Microsoft.AspNetCore.App`. Il riferimento del metapacchetto può specificare una versione minima richiesta. Per altre informazioni, vedere [The shared framework](https://natemcmaster.com/blog/2018/08/29/netcore-primitives-2/) (Il framework condiviso).

La pagina di errore *502.5 Errore del processo* viene restituita quando il processo di lavoro ha esito negativo a causa di un errore di configurazione dell'hosting o dell'app:

### <a name="failed-to-start-application-errorcode-0x800700c1"></a>Non è stato possibile avviare l'aggiornamento dell'applicazione. Errore: 0x800700c1

```
EventID: 1010
Source: IIS AspNetCore Module V2
Failed to start application '/LM/W3SVC/6/ROOT/', ErrorCode '0x800700c1'.
```

L'app non è stata avviata perché non è stato possibile caricarne l'assembly (*.dll*).

Questo errore si verifica quando è presente una mancata corrispondenza del numero di bit tra l'app pubblicata e il processo w3wp/iisexpress.

Verificare che l'impostazione del pool di app a 32 bit sia corretta:

1. Selezionare il pool di app in **Pool di applicazioni** di Gestione IIS.
1. Selezionare **Impostazioni avanzate** in **Modifica pool di applicazioni** nel pannello **Azioni**.
1. Impostare **Attiva applicazioni a 32 bit**:
   * Se si sta sviluppando un'app a 32 bit (x86), impostare il valore su `True`.
   * Se si sta sviluppando un'app a 64 bit (x64), impostare il valore su `False`.

Verificare che non esista un conflitto tra una `<Platform>` proprietà di MSBuild nel file di progetto e il bit pubblicato dell'app.

### <a name="connection-reset"></a>Connection reset (Connessione ripristinata)

Se si verifica un errore dopo l'invio delle intestazioni, è troppo tardi perché il server possa inviare un codice **500 Errore interno del server** quando si verifica un errore. Questo spesso accade quando si verifica un errore durante la serializzazione di oggetti complessi per una risposta. Questo tipo di errore viene visualizzato come un errore di *reimpostazione della connessione* nel client. La [registrazione delle applicazioni](xref:fundamentals/logging/index) può consentire di risolvere questi tipi di errori.

### <a name="default-startup-limits"></a>Limiti di avvio predefiniti

Il [modulo ASP.NET Core](xref:host-and-deploy/aspnet-core-module) è configurato con un valore predefinito di *startupTimeLimit* di 120 secondi. Quando si mantiene il valore predefinito, l'avvio di un'app potrebbe richiedere fino a due minuti prima che il modulo registri un errore del processo. Per informazioni sulla configurazione del modulo, vedere [Attributi dell'elemento aspNetCore](xref:host-and-deploy/aspnet-core-module#attributes-of-the-aspnetcore-element).

## <a name="troubleshoot-on-azure-app-service"></a>Risolvere i problemi relativi al servizio app Azure

[!INCLUDE [Azure App Service Preview Notice](~/includes/azure-apps-preview-notice.md)]

### <a name="application-event-log-azure-app-service"></a>Registro eventi applicazioni (servizio app Azure)

Per accedere al log eventi dell'applicazione, usare il pannello **Diagnostica e risolvi i problemi** nel portale di Azure:

1. Nel portale di Azure aprire l'app in **Servizi app**.
1. Selezionare **Diagnostica e risoluzione dei problemi**.
1. Selezionare l'intestazione **Strumenti di diagnostica**.
1. In **Strumenti di supporto** selezionare il pulsante **Eventi dell'applicazione**.
1. Esaminare l'errore più recente dalla voce *IIS AspNetCoreModule* o *IIS AspNetCoreModule V2* nella colonna **Origine**.

Un'alternativa all'uso del pannello **Diagnostica e risolvi i problemi** consiste nell'esaminare direttamente il file del log eventi dell'applicazione usando [Kudu](https://github.com/projectkudu/kudu/wiki):

1. Aprire **Strumenti avanzati** nell'area **Strumenti di sviluppo**. Selezionare il **pulsante &rarr; Vai** . Verrà aperta la console Kudu in una nuova scheda o finestra del browser.
1. Usando la barra di spostamento nella parte superiore della pagina, aprire **Console di debug** e selezionare **CMD**.
1. Aprire la cartella **LogFiles** .
1. Selezionare l'icona della matita accanto al file *eventlog.xml*.
1. Esaminare il log. Scorrere fino alla fine del log per visualizzare gli eventi più recenti.

### <a name="run-the-app-in-the-kudu-console"></a>Eseguire l'app nella console Kudu

Molti errori di avvio non producono informazioni utili nel log eventi dell'applicazione. È possibile eseguire l'app nella console di esecuzione remota [Kudu](https://github.com/projectkudu/kudu/wiki) per individuare l'errore:

1. Aprire **Strumenti avanzati** nell'area **Strumenti di sviluppo**. Selezionare il **pulsante &rarr; Vai** . Verrà aperta la console Kudu in una nuova scheda o finestra del browser.
1. Usando la barra di spostamento nella parte superiore della pagina, aprire **Console di debug** e selezionare **CMD**.

#### <a name="test-a-32-bit-x86-app"></a>Test di un'app a 32 bit (x86)

**Versione corrente**

1. `cd d:\home\site\wwwroot`
1. Eseguire l'app:
   * Se l'app è una [distribuzione dipendente dal framework](/dotnet/core/deploying/#framework-dependent-deployments-fdd):

     ```dotnetcli
     dotnet .\{ASSEMBLY NAME}.dll
     ```

   * Se l'app è una [distribuzione autonoma](/dotnet/core/deploying/#self-contained-deployments-scd):

     ```console
     {ASSEMBLY NAME}.exe
     ```

L'output della console dall'app, che visualizza gli eventuali errori, viene inviato tramite pipe alla console Kudu.

**Distribuzione dipendente dal framework in esecuzione in una versione di anteprima**

*Richiede l'installazione dell'estensione del sito del runtime ASP.NET Core {VERSION} (x86).*

1. `cd D:\home\SiteExtensions\AspNetCoreRuntime.{X.Y}.x32` (`{X.Y}` è la versione di runtime)
1. Eseguire l'app: `dotnet \home\site\wwwroot\{ASSEMBLY NAME}.dll`

L'output della console dall'app, che visualizza gli eventuali errori, viene inviato tramite pipe alla console Kudu.

#### <a name="test-a-64-bit-x64-app"></a>Test di un'app a 64 bit (x64)

**Versione corrente**

* Se l'app è una [distribuzione dipendente dal framework](/dotnet/core/deploying/#framework-dependent-deployments-fdd) a 64 bit (x64):
  1. `cd D:\Program Files\dotnet`
  1. Eseguire l'app: `dotnet \home\site\wwwroot\{ASSEMBLY NAME}.dll`
* Se l'app è una [distribuzione autonoma](/dotnet/core/deploying/#self-contained-deployments-scd):
  1. `cd D:\home\site\wwwroot`
  1. Eseguire l'app: `{ASSEMBLY NAME}.exe`

L'output della console dall'app, che visualizza gli eventuali errori, viene inviato tramite pipe alla console Kudu.

**Distribuzione dipendente dal framework in esecuzione in una versione di anteprima**

*Richiede l'installazione dell'estensione del sito del runtime ASP.NET Core {VERSION} (x64).*

1. `cd D:\home\SiteExtensions\AspNetCoreRuntime.{X.Y}.x64` (`{X.Y}` è la versione di runtime)
1. Eseguire l'app: `dotnet \home\site\wwwroot\{ASSEMBLY NAME}.dll`

L'output della console dall'app, che visualizza gli eventuali errori, viene inviato tramite pipe alla console Kudu.

### <a name="aspnet-core-module-stdout-log-azure-app-service"></a>Log stdout del modulo ASP.NET Core (servizio app Azure)

> [!WARNING]
> La mancata disabilitazione del log stdout può causare un errore dell'app o del server. Non esiste alcun limite per le dimensioni dei file di log o il numero di file di log che è possibile creare. Usare la registrazione stdout solo per la risoluzione dei problemi di avvio delle app.
>
> Per la registrazione generale in un'app ASP.NET Core dopo l'avvio, usare una libreria di registrazione che limita le dimensioni dei file di log e ne esegue la rotazione. Per altre informazioni, vedere [Provider di registrazione di terze parti](xref:fundamentals/logging/index#third-party-logging-providers).

Il log stdout del modulo ASP.NET Core spesso registra utili messaggi di errore non disponibili nel log eventi dell'applicazione. Per abilitare e visualizzare i log stdout:

1. Nel portale di Azure passare all'app Web.
1. Nel pannello **servizio app** immettere **Kudu** nella casella di ricerca.
1. Selezionare **strumenti avanzati** > **Vai**.
1. Selezionare  **console di Debug > cmd**.
1. Passa a *sito/wwwroot*
1. Selezionare l'icona a matita per modificare il file di *web.config* .
1. Nell' `<aspNetCore />` elemento impostare `stdoutLogEnabled="true"` e selezionare **Salva**.

Disabilitare la registrazione stdout quando la risoluzione dei problemi è stata completata impostando `stdoutLogEnabled="false"` .

Per altre informazioni, vedere <xref:host-and-deploy/aspnet-core-module#log-creation-and-redirection>.

<a name="enhanced-diagnostic-logs"></a>

### <a name="aspnet-core-module-debug-log-azure-app-service"></a>Log di debug del modulo ASP.NET Core (servizio app Azure)

Il log di debug del modulo ASP.NET Core fornisce dati di registrazione aggiuntivi e più approfonditi dal modulo ASP.NET Core. Per abilitare e visualizzare i log stdout:

1. Per abilitare il log di diagnostica avanzato, eseguire le operazioni seguenti:
   * Seguire le istruzioni in [Log di diagnostica avanzati](xref:host-and-deploy/aspnet-core-module#enhanced-diagnostic-logs) per configurare l'app per la registrazione di diagnostica avanzata. Ridistribuire l'app.
   * Aggiungere le impostazioni `<handlerSettings>` indicate in [Log di diagnostica avanzati](xref:host-and-deploy/aspnet-core-module#enhanced-diagnostic-logs) al file *web.config* dell'app distribuita, usando la console Kudu:
     1. Aprire **Strumenti avanzati** nell'area **Strumenti di sviluppo**. Selezionare il **pulsante &rarr; Vai** . Verrà aperta la console Kudu in una nuova scheda o finestra del browser.
     1. Usando la barra di spostamento nella parte superiore della pagina, aprire **Console di debug** e selezionare **CMD**.
     1. Aprire le cartelle nel **sito** del percorso  >  **wwwroot**. Modificare il file *web.config* selezionando il pulsante a forma di matita. Aggiungere la sezione `<handlerSettings>` come illustrato in [Log di diagnostica avanzati](xref:host-and-deploy/aspnet-core-module#enhanced-diagnostic-logs). Fare clic sul pulsante **Salva**.
1. Aprire **Strumenti avanzati** nell'area **Strumenti di sviluppo**. Selezionare il **pulsante &rarr; Vai** . Verrà aperta la console Kudu in una nuova scheda o finestra del browser.
1. Usando la barra di spostamento nella parte superiore della pagina, aprire **Console di debug** e selezionare **CMD**.
1. Aprire le cartelle nel **sito** del percorso  >  **wwwroot**. Se non è stato specificato un percorso per il file *aspnetcore-debug.log*, il file viene visualizzato nell'elenco. Se è stato specificato un percorso, passare al percorso del file di log.
1. Aprire il file di log con il pulsante a forma di matita accanto al nome del file.

Al termine della risoluzione dei problemi, disabilitare la registrazione di debug:

Per disabilitare il log di debug avanzato, eseguire le operazioni seguenti:

* Rimuovere `<handlerSettings>` dal file *web.config* in locale e ridistribuire l'app.
* Usare la console Kudu per modificare il file *web.config* e rimuovere la sezione `<handlerSettings>`. Salvare il file.

Per altre informazioni, vedere <xref:host-and-deploy/aspnet-core-module#enhanced-diagnostic-logs>.

> [!WARNING]
> La mancata disabilitazione del log di debug può causare un errore dell'app o del server. Non è previsto alcun limite per le dimensioni del file di log. Usare solo la registrazione di debug per la risoluzione dei problemi di avvio delle app.
>
> Per la registrazione generale in un'app ASP.NET Core dopo l'avvio, usare una libreria di registrazione che limita le dimensioni dei file di log e ne esegue la rotazione. Per altre informazioni, vedere [Provider di registrazione di terze parti](xref:fundamentals/logging/index#third-party-logging-providers).

### <a name="slow-or-hanging-app-azure-app-service"></a>App lenta o sospesa (servizio app Azure)

Quando un'app risponde lentamente o si blocca durante una richiesta, vedere gli articoli seguenti:

* [Risoluzione dei problemi di rallentamento delle prestazioni delle app Web nel Servizio app di Azure](/azure/app-service/app-service-web-troubleshoot-performance-degradation)
* [Use Crash Diagnoser Site Extension to Capture Dump for Intermittent Exception issues or performance issues on Azure Web App](https://blogs.msdn.microsoft.com/asiatech/2015/12/28/use-crash-diagnoser-site-extension-to-capture-dump-for-intermittent-exception-issues-or-performance-issues-on-azure-web-app/) (Usare l'estensione del sito Crash Diagnoser per acquisire il dump per i problemi di eccezioni intermittenti o i problemi di prestazioni nell'app Web di Azure)

### <a name="monitoring-blades"></a>Pannelli di monitoraggio

I pannelli di monitoraggio forniscono un'esperienza di risoluzione dei problemi alternativa ai metodi descritti in precedenza in questo argomento. È possibile usare questi pannelli per diagnosticare gli errori della serie 500.

Verificare che le estensioni di ASP.NET Core siano installate. Se le estensioni non sono installate, installarle manualmente:

1. Nella sezione del pannello **STRUMENTI DI SVILUPPO** selezionare il pannello **Estensioni**.
1. Nell'elenco dovrebbe essere visualizzato **ASP.NET Core Extensions** (Estensioni ASP.NET Core).
1. Se le estensioni non sono installate, selezionare il pulsante **Aggiungi**.
1. Scegliere **ASP.NET Core Extensions** (Estensioni ASP.NET Core) dall'elenco.
1. Selezionare **OK** per accettare le condizioni legali.
1. Selezionare **OK** nel pannello **Aggiungi estensione**.
1. Un messaggio popup informativo indica quando le estensioni sono state installate correttamente.

Se la registrazione stdout non è abilitata, attenersi ai passaggi riportati di seguito:

1. Nel portale di Azure selezionare il pannello **Strumenti avanzati** nell'area **STRUMENTI DI SVILUPPO**. Selezionare il **pulsante &rarr; Vai** . Verrà aperta la console Kudu in una nuova scheda o finestra del browser.
1. Usando la barra di spostamento nella parte superiore della pagina, aprire **Console di debug** e selezionare **CMD**.
1. Aprire le cartelle nel percorso **site** > **wwwroot** e scorrere verso il basso fino a visualizzare il file *web.config* in fondo all'elenco.
1. Fare clic sull'icona della matita accanto al file *web.config*.
1. Impostare **stdoutLogEnabled** su `true` e cambiare il percorso di **stdoutLogFile** in: `\\?\%home%\LogFiles\stdout`.
1. Selezionare **Salva** per salvare il file *web.config* aggiornato.

Passare all'attivazione della registrazione diagnostica:

1. Nel portale di Azure selezionare il pannello **Log di diagnostica**.
1. Selezionare l'interruttore **Attivato** per **Registrazione applicazioni (file system)** e **Messaggi di errore dettagliati**. Selezionare il pulsante **Salva** nella parte superiore del pannello.
1. Per includere la traccia delle richieste non riuscite, anche nota come registrazione FREB (Failed Request Event Buffering), selezionare l'interruttore **Attivato** per **Traccia delle richieste non riuscite**.
1. Selezionare il pannello **Flusso di registrazione**, immediatamente sotto il pannello **Log di diagnostica** nel portale.
1. Effettuare una richiesta all'app.
1. Nei dati del flusso di registrazione viene indicata la causa dell'errore.

Al termine della risoluzione dei problemi, assicurarsi di disabilitare la registrazione stdout.

Per visualizzare i log di traccia delle richieste non riuscite (log FREB):

1. Passare al pannello **Diagnostica e risolvi i problemi** nel portale di Azure.
1. Selezionare **Failed Request Tracing Logs** (Log di traccia delle richieste non riuscite) nell'area **STRUMENTI DI SUPPORTO** della barra laterale.

Per altre informazioni, vedere la [sezione relativa alla traccia delle richieste non riuscite nell'argomento Abilitare la registrazione diagnostica per le app Web nel servizio app di Azure](/azure/app-service/web-sites-enable-diagnostic-log#failed-request-traces) e [Domande frequenti sulle prestazioni delle applicazioni in App Web di Azure: Come si abilita la traccia delle richieste non riuscite?](/azure/app-service/app-service-web-availability-performance-application-issues-faq#how-do-i-turn-on-failed-request-tracing).

Per altre informazioni, vedere [Abilitare la registrazione diagnostica per le app Web nel servizio app di Azure](/azure/app-service/web-sites-enable-diagnostic-log).

> [!WARNING]
> La mancata disabilitazione del log stdout può causare un errore dell'app o del server. Non esiste alcun limite per le dimensioni dei file di log o il numero di file di log che è possibile creare.
>
> Per la registrazione di routine in un'app ASP.NET Core, usare una libreria di registrazione che limita le dimensioni dei file di log e ne esegue la rotazione. Per altre informazioni, vedere [Provider di registrazione di terze parti](xref:fundamentals/logging/index#third-party-logging-providers).

## <a name="troubleshoot-on-iis"></a>Risolvere i problemi in IIS

### <a name="application-event-log-iis"></a>Registro eventi applicazioni (IIS)

Accedere al log eventi dell'applicazione:

1. Aprire il menu Start, cercare *Visualizzatore eventi* e selezionare l'app **Visualizzatore eventi** .
1. In **Visualizzatore eventi** aprire il nodo **Registri di Windows**.
1. Selezionare **Applicazione** per aprire il log eventi dell'applicazione.
1. Cercare gli errori associati all'app in cui si è verificato il problema. Gli errori presentano un valore *Modulo AspNetCore IIS* o *Modulo AspNetCore IIS Express* nella colonna *Origine*.

### <a name="run-the-app-at-a-command-prompt"></a>Eseguire l'app da un prompt dei comandi

Molti errori di avvio non producono informazioni utili nel log eventi dell'applicazione. È possibile individuare la causa di alcuni errori eseguendo l'app da un prompt dei comandi nel sistema host.

#### <a name="framework-dependent-deployment"></a>Distribuzione dipendente dal framework

Se l'app è una [distribuzione dipendente dal framework](/dotnet/core/deploying/#framework-dependent-deployments-fdd):

1. Da un prompt dei comandi passare alla cartella di distribuzione e avviare l'app eseguendo l'assembly dell'app con *dotnet.exe*. Nel comando seguente sostituire il nome dell'assembly dell'app per \<assembly_name> : `dotnet .\<assembly_name>.dll` .
1. L'output della console per l'app, in cui sono indicati gli eventuali errori, verrà scritto nella finestra della console.
1. Se gli errori si verificano quando si effettua una richiesta all'app, effettuare una richiesta all'host e alla porta su cui è in ascolto Kestrel. Usando l'host e la porta predefiniti, effettuare una richiesta a `http://localhost:5000/`. Se l'app risponde normalmente nell'indirizzo endpoint di Kestrel, è più probabile che il problema sia associato alla configurazione dell'host e che non sia interno all'app.

#### <a name="self-contained-deployment"></a>Distribuzione autonoma

Se l'app è una [distribuzione autonoma](/dotnet/core/deploying/#self-contained-deployments-scd):

1. Da un prompt dei comandi passare alla cartella di distribuzione e avviare il file eseguibile dell'app. Nel comando seguente sostituire il nome dell'assembly dell'app per \<assembly_name> : `<assembly_name>.exe` .
1. L'output della console per l'app, in cui sono indicati gli eventuali errori, verrà scritto nella finestra della console.
1. Se gli errori si verificano quando si effettua una richiesta all'app, effettuare una richiesta all'host e alla porta su cui è in ascolto Kestrel. Usando l'host e la porta predefiniti, effettuare una richiesta a `http://localhost:5000/`. Se l'app risponde normalmente nell'indirizzo endpoint di Kestrel, è più probabile che il problema sia associato alla configurazione dell'host e che non sia interno all'app.

### <a name="aspnet-core-module-stdout-log-iis"></a>Log stdout del modulo ASP.NET Core (IIS)

Per abilitare e visualizzare i log stdout:

1. Passare alla cartella di distribuzione del sito nel sistema host.
1. Se la cartella *logs* non è presente, creare la cartella. Per istruzioni su come impostare MSBuild per la creazione automatica della cartella *logs* nella distribuzione, vedere l'argomento [Struttura della directory](xref:host-and-deploy/directory-structure).
1. Modificare il file *web.config*. Impostare **stdoutLogEnabled** su `true` e modificare il percorso di **stdoutLogFile** in modo da fare riferimento alla cartella *logs*, ad esempio `.\logs\stdout`. `stdout` nel percorso è il prefisso del nome del file di log. Un timestamp, l'ID del processo e l'estensione del file vengono aggiunti automaticamente al momento della creazione del log. Usando `stdout` come prefisso del nome del file, un tipico file di log è denominato *stdout_20180205184032_5412.log*.
1. Assicurarsi che l'identità del pool di applicazioni disponga delle autorizzazioni di scrittura per la cartella *logs*.
1. Salvare il file *web.config* aggiornato.
1. Effettuare una richiesta all'app.
1. Passare alla cartella *logs*. Trovare e aprire il log stdout più recente.
1. Esaminare il log per verificare se sono presenti errori.

Al termine della risoluzione dei problemi, disabilitare la registrazione stdout:

1. Modificare il file *web.config*.
1. Impostare **stdoutLogEnabled** su `false`.
1. Salvare il file.

Per altre informazioni, vedere <xref:host-and-deploy/aspnet-core-module#log-creation-and-redirection>.

> [!WARNING]
> La mancata disabilitazione del log stdout può causare un errore dell'app o del server. Non esiste alcun limite per le dimensioni dei file di log o il numero di file di log che è possibile creare.
>
> Per la registrazione di routine in un'app ASP.NET Core, usare una libreria di registrazione che limita le dimensioni dei file di log e ne esegue la rotazione. Per altre informazioni, vedere [Provider di registrazione di terze parti](xref:fundamentals/logging/index#third-party-logging-providers).

### <a name="aspnet-core-module-debug-log-iis"></a>Log di debug del modulo ASP.NET Core (IIS)

Aggiungere le impostazioni del gestore seguenti al file di *web.config* dell'app per abilitare ASP.NET Core log di debug del modulo:

```xml
<aspNetCore ...>
  <handlerSettings>
    <handlerSetting name="debugLevel" value="file" />
    <handlerSetting name="debugFile" value="c:\temp\ancm.log" />
  </handlerSettings>
</aspNetCore>
```

Verificare che il percorso specificato per il log esista e che l'identità del pool di applicazioni abbia le autorizzazioni di scrittura nel percorso.

Per altre informazioni, vedere <xref:host-and-deploy/aspnet-core-module#enhanced-diagnostic-logs>.

### <a name="enable-the-developer-exception-page"></a>Abilitare la pagina delle eccezioni per gli sviluppatori

La variabile di ambiente `ASPNETCORE_ENVIRONMENT` [ può essere aggiunta a web.config](xref:host-and-deploy/aspnet-core-module#setting-environment-variables) per eseguire l'app nell'ambiente di sviluppo. Purché l'ambiente non sia sottoposto a override durante l'avvio dell'app tramite `UseEnvironment` nel generatore di host, l'impostazione della variabile di ambiente consente di visualizzare la [pagina delle eccezioni per gli sviluppatori](xref:fundamentals/error-handling) quando viene eseguita l'app.

```xml
<aspNetCore processPath="dotnet"
      arguments=".\MyApp.dll"
      stdoutLogEnabled="false"
      stdoutLogFile=".\logs\stdout"
      hostingModel="InProcess">
  <environmentVariables>
    <environmentVariable name="ASPNETCORE_ENVIRONMENT" value="Development" />
  </environmentVariables>
</aspNetCore>
```

L'impostazione della variabile di ambiente per `ASPNETCORE_ENVIRONMENT` è consigliata solo per l'uso in server di gestione temporanea e test non esposti a Internet. Al termine della risoluzione dei problemi, rimuovere la variabile di ambiente dal file *web.config*. Per informazioni sull'impostazione delle variabili di ambiente in *web.config*, vedere [elemento figlio environmentVariables di aspNetCore](xref:host-and-deploy/aspnet-core-module#setting-environment-variables).

### <a name="obtain-data-from-an-app"></a>Ottenere dati da un'app

Se un'app è in grado di rispondere alle richieste, è possibile ottenere dati sulle richieste, le connessioni e altri dati dall'app tramite middleware inline di terminale. Per altre informazioni e codice di esempio, vedere <xref:test/troubleshoot#obtain-data-from-an-app>.

### <a name="slow-or-hanging-app-iis"></a>App lenta o sospesa (IIS)

Un *dump di arresto anomalo* del sistema è uno snapshot della memoria del sistema e può contribuire a determinare la provocazione di un arresto anomalo dell'app, dell'avvio o dell'applicazione lenta.

#### <a name="app-crashes-or-encounters-an-exception"></a>Arresto anomalo o eccezione di un'app

Ottenere e analizzare un dump da [Segnalazione errori Windows](/windows/desktop/wer/windows-error-reporting):

1. Creare una cartella per i file dump di arresto anomalo del sistema in `c:\dumps`. Il pool di app deve avere accesso in scrittura alla cartella.
1. Eseguire lo [script di PowerShell EnableDumps](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/test/troubleshoot-azure-iis/scripts/EnableDumps.ps1):
   * Se l'app usa il [modello di hosting in-process](xref:host-and-deploy/iis/index#in-process-hosting-model), eseguire lo script per *w3wp.exe*:

     ```console
     .\EnableDumps w3wp.exe c:\dumps
     ```

   * Se l'app usa il [modello di hosting out-of-process](xref:host-and-deploy/iis/index#out-of-process-hosting-model), eseguire lo script per *dotnet.exe*:

     ```console
     .\EnableDumps dotnet.exe c:\dumps
     ```

1. Eseguire l'app nelle condizioni che causano l'arresto anomalo.
1. Dopo che si è verificato l'arresto anomalo, eseguire lo [script di PowerShell DisableDumps](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/test/troubleshoot-azure-iis/scripts/DisableDumps.ps1):
   * Se l'app usa il [modello di hosting in-process](xref:host-and-deploy/iis/index#in-process-hosting-model), eseguire lo script per *w3wp.exe*:

     ```console
     .\DisableDumps w3wp.exe
     ```

   * Se l'app usa il [modello di hosting out-of-process](xref:host-and-deploy/iis/index#out-of-process-hosting-model), eseguire lo script per *dotnet.exe*:

     ```console
     .\DisableDumps dotnet.exe
     ```

Dopo l'arresto anomalo di un'app e la raccolta dei dump, l'app può terminare normalmente. Lo script di PowerShell configura Segnalazione errori Windows per raccogliere fino a cinque dump per ogni app.

> [!WARNING]
> I dump di arresto anomalo del sistema potrebbero richiedere una grande quantità di spazio su disco (fino a diversi gigabyte ognuno).

#### <a name="app-hangs-fails-during-startup-or-runs-normally"></a>L'app si blocca, si verifica un errore durante l'avvio o viene eseguita normalmente

Quando un'app si *blocca* (smette di rispondere ma non si arresta in modo anomalo), si verifica un errore durante l'avvio o viene eseguita normalmente, vedere [file di dump in modalità utente: scegliere lo strumento migliore](/windows-hardware/drivers/debugger/user-mode-dump-files#choosing-the-best-tool) per selezionare uno strumento appropriato per produrre il dump.

#### <a name="analyze-the-dump"></a>Analizzare il dump

È possibile analizzare un dump usando diversi approcci. Per altre informazioni, vedere [Analyzing a User-Mode Dump File](/windows-hardware/drivers/debugger/analyzing-a-user-mode-dump-file) (Analisi di un file dump in modalità utente).

## <a name="clear-package-caches"></a>Cancella cache di pacchetti

Un'app funzionante potrebbe non riuscire immediatamente dopo l'aggiornamento del .NET Core SDK nel computer di sviluppo o la modifica delle versioni del pacchetto all'interno dell'app. In alcuni casi i pacchetti incoerenti possono interrompere un'app quando si eseguono aggiornamenti principali. La maggior parte di questi problemi può essere risolta attenendosi alle istruzioni seguenti:

1. Eliminare le cartelle *bin* e *obj*.
1. Cancellare le cache dei pacchetti eseguendo [le impostazioni locali di DotNet NuGet All--Clear](/dotnet/core/tools/dotnet-nuget-locals) da una shell dei comandi.

   La cancellazione delle cache dei pacchetti può essere eseguita anche con lo strumento [nuget.exe](https://www.nuget.org/downloads) e l'esecuzione del comando `nuget locals all -clear` . *nuget.exe* non è un'installazione inclusa con il sistema operativo desktop Windows e deve essere ottenuta separatamente dal [sito Web NuGet](https://www.nuget.org/downloads).

1. Ripristinare e ricompilare il progetto.
1. Eliminare tutti i file nella cartella di distribuzione nel server prima di ridistribuire l'app.

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:test/troubleshoot>
* <xref:host-and-deploy/azure-iis-errors-reference>
* <xref:fundamentals/error-handling>
* <xref:host-and-deploy/aspnet-core-module>

### <a name="azure-documentation"></a>Documentazione di Azure

* [Application Insights per ASP.NET Core](/azure/application-insights/app-insights-asp-net-core)
* [Sezione app Web per il debug remoto di risoluzione dei problemi di un'app Web nel servizio app Azure con Visual Studio](/azure/app-service/web-sites-dotnet-troubleshoot-visual-studio#remotedebug)
* [Panoramica della diagnostica del servizio app di Azure](/azure/app-service/app-service-diagnostics)
* [Procedura: Eseguire il monitoraggio delle app nel servizio app di Azure](/azure/app-service/web-sites-monitor)
* [Risoluzione dei problemi di un'app Web nel servizio app di Azure tramite Visual Studio](/azure/app-service/web-sites-dotnet-troubleshoot-visual-studio)
* [Risolvere gli errori HTTP "502 - Gateway non valido" e "503 - Servizio non disponibile" nelle App Web di Azure](/azure/app-service/app-service-web-troubleshoot-http-502-http-503)
* [Risoluzione dei problemi di rallentamento delle prestazioni delle app Web nel Servizio app di Azure](/azure/app-service/app-service-web-troubleshoot-performance-degradation)
* [Domande frequenti sulle prestazioni delle applicazioni in App Web di Azure](/azure/app-service/app-service-web-availability-performance-application-issues-faq)
* [Azure Web App sandbox (App Service runtime execution limitations)](https://github.com/projectkudu/kudu/wiki/Azure-Web-App-sandbox) (Sandbox per app Web di Azure - Limitazioni di esecuzione di runtime di Servizio app di Azure)
* [Azure Friday: Azure App Service Diagnostic and Troubleshooting Experience](https://channel9.msdn.com/Shows/Azure-Friday/Azure-App-Service-Diagnostic-and-Troubleshooting-Experience) (Azure Friday: diagnostica e risoluzione dei problemi del servizio app di Azure) (video di 12 minuti)

### <a name="visual-studio-documentation"></a>Documentazione di Visual Studio

* [ASP.NET Core di debug remoto in IIS in Azure in Visual Studio 2017](/visualstudio/debugger/remote-debugging-azure)
* [ASP.NET Core di debug remoto in un computer IIS remoto in Visual Studio 2017](/visualstudio/debugger/remote-debugging-aspnet-on-a-remote-iis-computer)
* [Informazioni sul debug con Visual Studio](/visualstudio/debugger/getting-started-with-the-debugger)

### <a name="visual-studio-code-documentation"></a>Documentazione di Visual Studio Code

* [Debug con Visual Studio Code](https://code.visualstudio.com/docs/editor/debugging)

::: moniker-end

::: moniker range="= aspnetcore-2.2"

Questo articolo fornisce informazioni sugli errori comuni di avvio delle app e istruzioni su come diagnosticare gli errori quando un'app viene distribuita in app Azure servizio o IIS:

[Errori di avvio dell'app](#app-startup-errors)  
Vengono illustrati gli scenari di codice di stato HTTP di avvio comuni.

[Risolvere i problemi relativi al servizio app Azure](#troubleshoot-on-azure-app-service)  
Fornisce consigli per la risoluzione dei problemi per le app distribuite nel servizio app Azure.

[Risolvere i problemi in IIS](#troubleshoot-on-iis)  
Fornisce consigli per la risoluzione dei problemi per le app distribuite in IIS o in esecuzione in IIS Express localmente. Le linee guida sono valide per le distribuzioni di Windows Server e Windows desktop.

[Cancella cache di pacchetti](#clear-package-caches)  
Spiega cosa fare quando i pacchetti incoerenti interrompono un'app durante l'esecuzione di aggiornamenti principali o la modifica delle versioni del pacchetto.

[Risorse aggiuntive](#additional-resources)  
Elenca ulteriori argomenti sulla risoluzione dei problemi.

## <a name="app-startup-errors"></a>Errori di avvio dell'app

In Visual Studio, per impostazione predefinita, un progetto ASP.NET Core viene [IIS Express](/iis/extensions/introduction-to-iis-express/iis-express-overview) host durante il debug. Un errore *502,5-Process* o un *errore di avvio 500,30* che si verifica quando il debug locale può essere diagnosticato utilizzando i consigli in questo argomento.

### <a name="40314-forbidden"></a>403,14 non consentito

Non è possibile avviare l'app. Viene registrato l'errore seguente:

```
The Web server is configured to not list the contents of this directory.
```

L'errore è in genere causato da una distribuzione interruppe sul sistema host, che include uno degli scenari seguenti:

* L'app viene distribuita nella cartella errata del sistema di hosting.
* Il processo di distribuzione non è riuscito a spostare tutti i file e le cartelle dell'app nella cartella di distribuzione nel sistema di hosting.
* Il file di *web.config* non è presente nella distribuzione oppure il contenuto del file di *web.config* non è valido.

Eseguire la procedura seguente:

1. Eliminare tutti i file e le cartelle dalla cartella di distribuzione nel sistema di hosting.
1. Ridistribuire il contenuto della cartella di *pubblicazione* dell'app nel sistema host usando il normale metodo di distribuzione, ad esempio Visual Studio, PowerShell o la distribuzione manuale:
   * Verificare che il file *web.config* sia presente nella distribuzione e che il relativo contenuto sia corretto.
   * Quando si esegue l'hosting in app Azure servizio, verificare che l'app venga distribuita nella `D:\home\site\wwwroot` cartella.
   * Quando l'app è ospitata da IIS, verificare che l'app venga distribuita nel **percorso fisico** IIS visualizzato nelle impostazioni di **base** di **Gestione IIS**.
1. Verificare che tutti i file e le cartelle dell'app vengano distribuiti confrontando la distribuzione nel sistema di hosting con il contenuto della cartella di *pubblicazione* del progetto.

Per ulteriori informazioni sul layout di un'app ASP.NET Core pubblicata, vedere <xref:host-and-deploy/directory-structure> . Per ulteriori informazioni sul file di *web.config* , vedere <xref:host-and-deploy/aspnet-core-module#configuration-with-webconfig> .

### <a name="500-internal-server-error"></a>500 - Errore interno del server

L'app viene avviata, ma un errore impedisce al server di soddisfare la richiesta.

Questo errore si verifica all'interno del codice dell'app durante l'avvio o durante la creazione di una risposta. La risposta potrebbe non avere alcun contenuto o essere visualizzata nel browser come un codice *500 Errore interno del server*. Il log eventi dell'applicazione in genere indica che l'app è stata avviata normalmente. Dal punto di vista del server, questo è corretto. L'app è stata effettivamente avviata, ma non può generare una risposta valida. Per risolvere il problema, eseguire l'app dal prompt dei comandi nel server o abilitare il log stdout del modulo ASP.NET Core.

### <a name="5000-in-process-handler-load-failure"></a>500.0 Errore di caricamento del gestore in-process

Il processo di lavoro ha esito negativo. L'app non viene avviata.

Il [modulo ASP.NET Core](xref:host-and-deploy/aspnet-core-module) non riesce a trovare il CLR .NET Core e a trovare il gestore di richieste in-process (*aspnetcorev2_inprocess.dll*). Controllare che:

* L'app specifichi come destinazione il pacchetto NuGet [Microsoft.AspNetCore.Server.IIS](https://www.nuget.org/packages/Microsoft.AspNetCore.Server.IIS) o il [metapacchetto Microsoft.AspNetCore.App](xref:fundamentals/metapackage-app).
* Le versione del framework condiviso ASP.NET Core specificata come destinazione dall'app sia installata nel computer di destinazione.

### <a name="5000-out-of-process-handler-load-failure"></a>500.0 Errore di caricamento del gestore out-of-process

Il processo di lavoro ha esito negativo. L'app non viene avviata.

Il [modulo ASP.NET Core](xref:host-and-deploy/aspnet-core-module) non riesce a trovare il gestore della richiesta di hosting out-of-process. Verificare che *aspnetcorev2_outofprocess.dll* sia presente in una sottocartella accanto ad *aspnetcorev2.dll*.

### <a name="5025-process-failure"></a>502.5 Process Failure (Errore del processo)

Il processo di lavoro ha esito negativo. L'app non viene avviata.

Il [modulo ASP.NET Core](xref:host-and-deploy/aspnet-core-module) tenta di avviare il processo di lavoro, ma l'avvio non riesce. In genere è possibile determinare la causa di un errore di avvio del processo dalle voci nel log eventi dell'applicazione e nel log stdout del modulo ASP.NET Core.

Una condizione di errore comune è dovuta alla configurazione non corretta dell'app perché come destinazione viene specificata una versione del framework condiviso ASP.NET Core, che non è presente. Controllare le versioni del framework condiviso ASP.NET Core installate nel computer di destinazione. Il *framework condiviso* è il set di assembly (file *DLL*) che vengono installati nel computer e cui fa riferimento un metapacchetto, ad esempio `Microsoft.AspNetCore.App`. Il riferimento del metapacchetto può specificare una versione minima richiesta. Per altre informazioni, vedere [The shared framework](https://natemcmaster.com/blog/2018/08/29/netcore-primitives-2/) (Il framework condiviso).

La pagina di errore *502.5 Errore del processo* viene restituita quando il processo di lavoro ha esito negativo a causa di un errore di configurazione dell'hosting o dell'app:

### <a name="failed-to-start-application-errorcode-0x800700c1"></a>Non è stato possibile avviare l'aggiornamento dell'applicazione. Errore: 0x800700c1

```
EventID: 1010
Source: IIS AspNetCore Module V2
Failed to start application '/LM/W3SVC/6/ROOT/', ErrorCode '0x800700c1'.
```

L'app non è stata avviata perché non è stato possibile caricarne l'assembly (*.dll*).

Questo errore si verifica quando è presente una mancata corrispondenza del numero di bit tra l'app pubblicata e il processo w3wp/iisexpress.

Verificare che l'impostazione del pool di app a 32 bit sia corretta:

1. Selezionare il pool di app in **Pool di applicazioni** di Gestione IIS.
1. Selezionare **Impostazioni avanzate** in **Modifica pool di applicazioni** nel pannello **Azioni**.
1. Impostare **Attiva applicazioni a 32 bit**:
   * Se si sta sviluppando un'app a 32 bit (x86), impostare il valore su `True`.
   * Se si sta sviluppando un'app a 64 bit (x64), impostare il valore su `False`.

Verificare che non esista un conflitto tra una `<Platform>` proprietà di MSBuild nel file di progetto e il bit pubblicato dell'app.

### <a name="connection-reset"></a>Connection reset (Connessione ripristinata)

Se si verifica un errore dopo l'invio delle intestazioni, è troppo tardi perché il server possa inviare un codice **500 Errore interno del server** quando si verifica un errore. Questo spesso accade quando si verifica un errore durante la serializzazione di oggetti complessi per una risposta. Questo tipo di errore viene visualizzato come un errore di *reimpostazione della connessione* nel client. La [registrazione delle applicazioni](xref:fundamentals/logging/index) può consentire di risolvere questi tipi di errori.

### <a name="default-startup-limits"></a>Limiti di avvio predefiniti

Il [modulo ASP.NET Core](xref:host-and-deploy/aspnet-core-module) è configurato con un valore predefinito di *startupTimeLimit* di 120 secondi. Quando si mantiene il valore predefinito, l'avvio di un'app potrebbe richiedere fino a due minuti prima che il modulo registri un errore del processo. Per informazioni sulla configurazione del modulo, vedere [Attributi dell'elemento aspNetCore](xref:host-and-deploy/aspnet-core-module#attributes-of-the-aspnetcore-element).

## <a name="troubleshoot-on-azure-app-service"></a>Risolvere i problemi relativi al servizio app Azure

[!INCLUDE [Azure App Service Preview Notice](~/includes/azure-apps-preview-notice.md)]

### <a name="application-event-log-azure-app-service"></a>Registro eventi applicazioni (servizio app Azure)

Per accedere al log eventi dell'applicazione, usare il pannello **Diagnostica e risolvi i problemi** nel portale di Azure:

1. Nel portale di Azure aprire l'app in **Servizi app**.
1. Selezionare **Diagnostica e risoluzione dei problemi**.
1. Selezionare l'intestazione **Strumenti di diagnostica**.
1. In **Strumenti di supporto** selezionare il pulsante **Eventi dell'applicazione**.
1. Esaminare l'errore più recente dalla voce *IIS AspNetCoreModule* o *IIS AspNetCoreModule V2* nella colonna **Origine**.

Un'alternativa all'uso del pannello **Diagnostica e risolvi i problemi** consiste nell'esaminare direttamente il file del log eventi dell'applicazione usando [Kudu](https://github.com/projectkudu/kudu/wiki):

1. Aprire **Strumenti avanzati** nell'area **Strumenti di sviluppo**. Selezionare il **pulsante &rarr; Vai** . Verrà aperta la console Kudu in una nuova scheda o finestra del browser.
1. Usando la barra di spostamento nella parte superiore della pagina, aprire **Console di debug** e selezionare **CMD**.
1. Aprire la cartella **LogFiles** .
1. Selezionare l'icona della matita accanto al file *eventlog.xml*.
1. Esaminare il log. Scorrere fino alla fine del log per visualizzare gli eventi più recenti.

### <a name="run-the-app-in-the-kudu-console"></a>Eseguire l'app nella console Kudu

Molti errori di avvio non producono informazioni utili nel log eventi dell'applicazione. È possibile eseguire l'app nella console di esecuzione remota [Kudu](https://github.com/projectkudu/kudu/wiki) per individuare l'errore:

1. Aprire **Strumenti avanzati** nell'area **Strumenti di sviluppo**. Selezionare il **pulsante &rarr; Vai** . Verrà aperta la console Kudu in una nuova scheda o finestra del browser.
1. Usando la barra di spostamento nella parte superiore della pagina, aprire **Console di debug** e selezionare **CMD**.

#### <a name="test-a-32-bit-x86-app"></a>Test di un'app a 32 bit (x86)

**Versione corrente**

1. `cd d:\home\site\wwwroot`
1. Eseguire l'app:
   * Se l'app è una [distribuzione dipendente dal framework](/dotnet/core/deploying/#framework-dependent-deployments-fdd):

     ```dotnetcli
     dotnet .\{ASSEMBLY NAME}.dll
     ```

   * Se l'app è una [distribuzione autonoma](/dotnet/core/deploying/#self-contained-deployments-scd):

     ```console
     {ASSEMBLY NAME}.exe
     ```

L'output della console dall'app, che visualizza gli eventuali errori, viene inviato tramite pipe alla console Kudu.

**Distribuzione dipendente dal framework in esecuzione in una versione di anteprima**

*Richiede l'installazione dell'estensione del sito del runtime ASP.NET Core {VERSION} (x86).*

1. `cd D:\home\SiteExtensions\AspNetCoreRuntime.{X.Y}.x32` (`{X.Y}` è la versione di runtime)
1. Eseguire l'app: `dotnet \home\site\wwwroot\{ASSEMBLY NAME}.dll`

L'output della console dall'app, che visualizza gli eventuali errori, viene inviato tramite pipe alla console Kudu.

#### <a name="test-a-64-bit-x64-app"></a>Test di un'app a 64 bit (x64)

**Versione corrente**

* Se l'app è una [distribuzione dipendente dal framework](/dotnet/core/deploying/#framework-dependent-deployments-fdd) a 64 bit (x64):
  1. `cd D:\Program Files\dotnet`
  1. Eseguire l'app: `dotnet \home\site\wwwroot\{ASSEMBLY NAME}.dll`
* Se l'app è una [distribuzione autonoma](/dotnet/core/deploying/#self-contained-deployments-scd):
  1. `cd D:\home\site\wwwroot`
  1. Eseguire l'app: `{ASSEMBLY NAME}.exe`

L'output della console dall'app, che visualizza gli eventuali errori, viene inviato tramite pipe alla console Kudu.

**Distribuzione dipendente dal framework in esecuzione in una versione di anteprima**

*Richiede l'installazione dell'estensione del sito del runtime ASP.NET Core {VERSION} (x64).*

1. `cd D:\home\SiteExtensions\AspNetCoreRuntime.{X.Y}.x64` (`{X.Y}` è la versione di runtime)
1. Eseguire l'app: `dotnet \home\site\wwwroot\{ASSEMBLY NAME}.dll`

L'output della console dall'app, che visualizza gli eventuali errori, viene inviato tramite pipe alla console Kudu.

### <a name="aspnet-core-module-stdout-log-azure-app-service"></a>Log stdout del modulo ASP.NET Core (servizio app Azure)

Il log stdout del modulo ASP.NET Core spesso registra utili messaggi di errore non disponibili nel log eventi dell'applicazione. Per abilitare e visualizzare i log stdout:

1. Passare al pannello **Diagnostica e risolvi i problemi** nel portale di Azure.
1. In **SELECT PROBLEM CATEGORY** (SELEZIONARE LA CATEGORIA DEL PROBLEMA) selezionare il pulsante **Web App Down** (App Web inattiva).
1. In **Suggested Solutions (Soluzioni consigliate)** > **Enable Stdout Log Redirection** (Abilita il reindirizzamento del log Stdout) selezionare il pulsante **Open Kudu Console to edit Web.Config** (Apri la console Kudu per modificare Web.Config).
1. Nella **console diagnostica** Kudu aprire le cartelle nel percorso **site** > **wwwroot**. Scorrere verso il basso fino a visualizzare il file *web.config* in fondo all'elenco.
1. Fare clic sull'icona della matita accanto al file *web.config*.
1. Impostare **stdoutLogEnabled** su `true` e cambiare il percorso di **stdoutLogFile** in: `\\?\%home%\LogFiles\stdout`.
1. Selezionare **Salva** per salvare il file *web.config* aggiornato.
1. Effettuare una richiesta all'app.
1. Tornare al portale di Azure. Selezionare il pannello **Strumenti avanzati** nell'area **STRUMENTI DI SVILUPPO**. Selezionare il **pulsante &rarr; Vai** . Verrà aperta la console Kudu in una nuova scheda o finestra del browser.
1. Usando la barra di spostamento nella parte superiore della pagina, aprire **Console di debug** e selezionare **CMD**.
1. Selezionare la cartella **LogFiles**.
1. Controllare la colonna **Data modifica** e selezionare l'icona della matita per modificare il log stdout con la data di modifica più recente.
1. Quando si apre il file di log, verrà visualizzato l'errore.

Al termine della risoluzione dei problemi, disabilitare la registrazione stdout:

1. Nella **console diagnostica** Kudu tornare al percorso **site** > **wwwroot** per visualizzare il file *web.config*. Aprire nuovamente il file **web.config** selezionando l'icona della matita.
1. Impostare **stdoutLogEnabled** su `false`.
1. Selezionare **Salva** per salvare il file.

Per altre informazioni, vedere <xref:host-and-deploy/aspnet-core-module#log-creation-and-redirection>.

> [!WARNING]
> La mancata disabilitazione del log stdout può causare un errore dell'app o del server. Non esiste alcun limite per le dimensioni dei file di log o il numero di file di log che è possibile creare. Usare la registrazione stdout solo per la risoluzione dei problemi di avvio delle app.
>
> Per la registrazione generale in un'app ASP.NET Core dopo l'avvio, usare una libreria di registrazione che limita le dimensioni dei file di log e ne esegue la rotazione. Per altre informazioni, vedere [Provider di registrazione di terze parti](xref:fundamentals/logging/index#third-party-logging-providers).

### <a name="aspnet-core-module-debug-log-azure-app-service"></a>Log di debug del modulo ASP.NET Core (servizio app Azure)

Il log di debug del modulo ASP.NET Core fornisce dati di registrazione aggiuntivi e più approfonditi dal modulo ASP.NET Core. Per abilitare e visualizzare i log stdout:

1. Per abilitare il log di diagnostica avanzato, eseguire le operazioni seguenti:
   * Seguire le istruzioni in [Log di diagnostica avanzati](xref:host-and-deploy/aspnet-core-module#enhanced-diagnostic-logs) per configurare l'app per la registrazione di diagnostica avanzata. Ridistribuire l'app.
   * Aggiungere le impostazioni `<handlerSettings>` indicate in [Log di diagnostica avanzati](xref:host-and-deploy/aspnet-core-module#enhanced-diagnostic-logs) al file *web.config* dell'app distribuita, usando la console Kudu:
     1. Aprire **Strumenti avanzati** nell'area **Strumenti di sviluppo**. Selezionare il **pulsante &rarr; Vai** . Verrà aperta la console Kudu in una nuova scheda o finestra del browser.
     1. Usando la barra di spostamento nella parte superiore della pagina, aprire **Console di debug** e selezionare **CMD**.
     1. Aprire le cartelle nel **sito** del percorso  >  **wwwroot**. Modificare il file *web.config* selezionando il pulsante a forma di matita. Aggiungere la sezione `<handlerSettings>` come illustrato in [Log di diagnostica avanzati](xref:host-and-deploy/aspnet-core-module#enhanced-diagnostic-logs). Fare clic sul pulsante **Salva**.
1. Aprire **Strumenti avanzati** nell'area **Strumenti di sviluppo**. Selezionare il **pulsante &rarr; Vai** . Verrà aperta la console Kudu in una nuova scheda o finestra del browser.
1. Usando la barra di spostamento nella parte superiore della pagina, aprire **Console di debug** e selezionare **CMD**.
1. Aprire le cartelle nel **sito** del percorso  >  **wwwroot**. Se non è stato specificato un percorso per il file *aspnetcore-debug.log*, il file viene visualizzato nell'elenco. Se è stato specificato un percorso, passare al percorso del file di log.
1. Aprire il file di log con il pulsante a forma di matita accanto al nome del file.

Al termine della risoluzione dei problemi, disabilitare la registrazione di debug:

Per disabilitare il log di debug avanzato, eseguire le operazioni seguenti:

* Rimuovere `<handlerSettings>` dal file *web.config* in locale e ridistribuire l'app.
* Usare la console Kudu per modificare il file *web.config* e rimuovere la sezione `<handlerSettings>`. Salvare il file.

Per altre informazioni, vedere <xref:host-and-deploy/aspnet-core-module#enhanced-diagnostic-logs>.

> [!WARNING]
> La mancata disabilitazione del log di debug può causare un errore dell'app o del server. Non è previsto alcun limite per le dimensioni del file di log. Usare solo la registrazione di debug per la risoluzione dei problemi di avvio delle app.
>
> Per la registrazione generale in un'app ASP.NET Core dopo l'avvio, usare una libreria di registrazione che limita le dimensioni dei file di log e ne esegue la rotazione. Per altre informazioni, vedere [Provider di registrazione di terze parti](xref:fundamentals/logging/index#third-party-logging-providers).

### <a name="slow-or-hanging-app-azure-app-service"></a>App lenta o sospesa (servizio app Azure)

Quando un'app risponde lentamente o si blocca durante una richiesta, vedere gli articoli seguenti:

* [Risoluzione dei problemi di rallentamento delle prestazioni delle app Web nel Servizio app di Azure](/azure/app-service/app-service-web-troubleshoot-performance-degradation)
* [Use Crash Diagnoser Site Extension to Capture Dump for Intermittent Exception issues or performance issues on Azure Web App](https://blogs.msdn.microsoft.com/asiatech/2015/12/28/use-crash-diagnoser-site-extension-to-capture-dump-for-intermittent-exception-issues-or-performance-issues-on-azure-web-app/) (Usare l'estensione del sito Crash Diagnoser per acquisire il dump per i problemi di eccezioni intermittenti o i problemi di prestazioni nell'app Web di Azure)

### <a name="monitoring-blades"></a>Pannelli di monitoraggio

I pannelli di monitoraggio forniscono un'esperienza di risoluzione dei problemi alternativa ai metodi descritti in precedenza in questo argomento. È possibile usare questi pannelli per diagnosticare gli errori della serie 500.

Verificare che le estensioni di ASP.NET Core siano installate. Se le estensioni non sono installate, installarle manualmente:

1. Nella sezione del pannello **STRUMENTI DI SVILUPPO** selezionare il pannello **Estensioni**.
1. Nell'elenco dovrebbe essere visualizzato **ASP.NET Core Extensions** (Estensioni ASP.NET Core).
1. Se le estensioni non sono installate, selezionare il pulsante **Aggiungi**.
1. Scegliere **ASP.NET Core Extensions** (Estensioni ASP.NET Core) dall'elenco.
1. Selezionare **OK** per accettare le condizioni legali.
1. Selezionare **OK** nel pannello **Aggiungi estensione**.
1. Un messaggio popup informativo indica quando le estensioni sono state installate correttamente.

Se la registrazione stdout non è abilitata, attenersi ai passaggi riportati di seguito:

1. Nel portale di Azure selezionare il pannello **Strumenti avanzati** nell'area **STRUMENTI DI SVILUPPO**. Selezionare il **pulsante &rarr; Vai** . Verrà aperta la console Kudu in una nuova scheda o finestra del browser.
1. Usando la barra di spostamento nella parte superiore della pagina, aprire **Console di debug** e selezionare **CMD**.
1. Aprire le cartelle nel percorso **site** > **wwwroot** e scorrere verso il basso fino a visualizzare il file *web.config* in fondo all'elenco.
1. Fare clic sull'icona della matita accanto al file *web.config*.
1. Impostare **stdoutLogEnabled** su `true` e cambiare il percorso di **stdoutLogFile** in: `\\?\%home%\LogFiles\stdout`.
1. Selezionare **Salva** per salvare il file *web.config* aggiornato.

Passare all'attivazione della registrazione diagnostica:

1. Nel portale di Azure selezionare il pannello **Log di diagnostica**.
1. Selezionare l'interruttore **Attivato** per **Registrazione applicazioni (file system)** e **Messaggi di errore dettagliati**. Selezionare il pulsante **Salva** nella parte superiore del pannello.
1. Per includere la traccia delle richieste non riuscite, anche nota come registrazione FREB (Failed Request Event Buffering), selezionare l'interruttore **Attivato** per **Traccia delle richieste non riuscite**.
1. Selezionare il pannello **Flusso di registrazione**, immediatamente sotto il pannello **Log di diagnostica** nel portale.
1. Effettuare una richiesta all'app.
1. Nei dati del flusso di registrazione viene indicata la causa dell'errore.

Al termine della risoluzione dei problemi, assicurarsi di disabilitare la registrazione stdout.

Per visualizzare i log di traccia delle richieste non riuscite (log FREB):

1. Passare al pannello **Diagnostica e risolvi i problemi** nel portale di Azure.
1. Selezionare **Failed Request Tracing Logs** (Log di traccia delle richieste non riuscite) nell'area **STRUMENTI DI SUPPORTO** della barra laterale.

Per altre informazioni, vedere la [sezione relativa alla traccia delle richieste non riuscite nell'argomento Abilitare la registrazione diagnostica per le app Web nel servizio app di Azure](/azure/app-service/web-sites-enable-diagnostic-log#failed-request-traces) e [Domande frequenti sulle prestazioni delle applicazioni in App Web di Azure: Come si abilita la traccia delle richieste non riuscite?](/azure/app-service/app-service-web-availability-performance-application-issues-faq#how-do-i-turn-on-failed-request-tracing).

Per altre informazioni, vedere [Abilitare la registrazione diagnostica per le app Web nel servizio app di Azure](/azure/app-service/web-sites-enable-diagnostic-log).

> [!WARNING]
> La mancata disabilitazione del log stdout può causare un errore dell'app o del server. Non esiste alcun limite per le dimensioni dei file di log o il numero di file di log che è possibile creare.
>
> Per la registrazione di routine in un'app ASP.NET Core, usare una libreria di registrazione che limita le dimensioni dei file di log e ne esegue la rotazione. Per altre informazioni, vedere [Provider di registrazione di terze parti](xref:fundamentals/logging/index#third-party-logging-providers).

## <a name="troubleshoot-on-iis"></a>Risolvere i problemi in IIS

### <a name="application-event-log-iis"></a>Registro eventi applicazioni (IIS)

Accedere al log eventi dell'applicazione:

1. Aprire il menu Start, cercare *Visualizzatore eventi* e selezionare l'app **Visualizzatore eventi** .
1. In **Visualizzatore eventi** aprire il nodo **Registri di Windows**.
1. Selezionare **Applicazione** per aprire il log eventi dell'applicazione.
1. Cercare gli errori associati all'app in cui si è verificato il problema. Gli errori presentano un valore *Modulo AspNetCore IIS* o *Modulo AspNetCore IIS Express* nella colonna *Origine*.

### <a name="run-the-app-at-a-command-prompt"></a>Eseguire l'app da un prompt dei comandi

Molti errori di avvio non producono informazioni utili nel log eventi dell'applicazione. È possibile individuare la causa di alcuni errori eseguendo l'app da un prompt dei comandi nel sistema host.

#### <a name="framework-dependent-deployment"></a>Distribuzione dipendente dal framework

Se l'app è una [distribuzione dipendente dal framework](/dotnet/core/deploying/#framework-dependent-deployments-fdd):

1. Da un prompt dei comandi passare alla cartella di distribuzione e avviare l'app eseguendo l'assembly dell'app con *dotnet.exe*. Nel comando seguente sostituire il nome dell'assembly dell'app per \<assembly_name> : `dotnet .\<assembly_name>.dll` .
1. L'output della console per l'app, in cui sono indicati gli eventuali errori, verrà scritto nella finestra della console.
1. Se gli errori si verificano quando si effettua una richiesta all'app, effettuare una richiesta all'host e alla porta su cui è in ascolto Kestrel. Usando l'host e la porta predefiniti, effettuare una richiesta a `http://localhost:5000/`. Se l'app risponde normalmente nell'indirizzo endpoint di Kestrel, è più probabile che il problema sia associato alla configurazione dell'host e che non sia interno all'app.

#### <a name="self-contained-deployment"></a>Distribuzione autonoma

Se l'app è una [distribuzione autonoma](/dotnet/core/deploying/#self-contained-deployments-scd):

1. Da un prompt dei comandi passare alla cartella di distribuzione e avviare il file eseguibile dell'app. Nel comando seguente sostituire il nome dell'assembly dell'app per \<assembly_name> : `<assembly_name>.exe` .
1. L'output della console per l'app, in cui sono indicati gli eventuali errori, verrà scritto nella finestra della console.
1. Se gli errori si verificano quando si effettua una richiesta all'app, effettuare una richiesta all'host e alla porta su cui è in ascolto Kestrel. Usando l'host e la porta predefiniti, effettuare una richiesta a `http://localhost:5000/`. Se l'app risponde normalmente nell'indirizzo endpoint di Kestrel, è più probabile che il problema sia associato alla configurazione dell'host e che non sia interno all'app.

### <a name="aspnet-core-module-stdout-log-iis"></a>Log stdout del modulo ASP.NET Core (IIS)

Per abilitare e visualizzare i log stdout:

1. Passare alla cartella di distribuzione del sito nel sistema host.
1. Se la cartella *logs* non è presente, creare la cartella. Per istruzioni su come impostare MSBuild per la creazione automatica della cartella *logs* nella distribuzione, vedere l'argomento [Struttura della directory](xref:host-and-deploy/directory-structure).
1. Modificare il file *web.config*. Impostare **stdoutLogEnabled** su `true` e modificare il percorso di **stdoutLogFile** in modo da fare riferimento alla cartella *logs*, ad esempio `.\logs\stdout`. `stdout` nel percorso è il prefisso del nome del file di log. Un timestamp, l'ID del processo e l'estensione del file vengono aggiunti automaticamente al momento della creazione del log. Usando `stdout` come prefisso del nome del file, un tipico file di log è denominato *stdout_20180205184032_5412.log*.
1. Assicurarsi che l'identità del pool di applicazioni disponga delle autorizzazioni di scrittura per la cartella *logs*.
1. Salvare il file *web.config* aggiornato.
1. Effettuare una richiesta all'app.
1. Passare alla cartella *logs*. Trovare e aprire il log stdout più recente.
1. Esaminare il log per verificare se sono presenti errori.

Al termine della risoluzione dei problemi, disabilitare la registrazione stdout:

1. Modificare il file *web.config*.
1. Impostare **stdoutLogEnabled** su `false`.
1. Salvare il file.

Per altre informazioni, vedere <xref:host-and-deploy/aspnet-core-module#log-creation-and-redirection>.

> [!WARNING]
> La mancata disabilitazione del log stdout può causare un errore dell'app o del server. Non esiste alcun limite per le dimensioni dei file di log o il numero di file di log che è possibile creare.
>
> Per la registrazione di routine in un'app ASP.NET Core, usare una libreria di registrazione che limita le dimensioni dei file di log e ne esegue la rotazione. Per altre informazioni, vedere [Provider di registrazione di terze parti](xref:fundamentals/logging/index#third-party-logging-providers).

### <a name="aspnet-core-module-debug-log-iis"></a>Log di debug del modulo ASP.NET Core (IIS)

Aggiungere le impostazioni del gestore seguenti al file di *web.config* dell'app per abilitare ASP.NET Core log di debug del modulo:

```xml
<aspNetCore ...>
  <handlerSettings>
    <handlerSetting name="debugLevel" value="file" />
    <handlerSetting name="debugFile" value="c:\temp\ancm.log" />
  </handlerSettings>
</aspNetCore>
```

Verificare che il percorso specificato per il log esista e che l'identità del pool di applicazioni abbia le autorizzazioni di scrittura nel percorso.

Per altre informazioni, vedere <xref:host-and-deploy/aspnet-core-module#enhanced-diagnostic-logs>.

### <a name="enable-the-developer-exception-page"></a>Abilitare la pagina delle eccezioni per gli sviluppatori

La variabile di ambiente `ASPNETCORE_ENVIRONMENT` [ può essere aggiunta a web.config](xref:host-and-deploy/aspnet-core-module#setting-environment-variables) per eseguire l'app nell'ambiente di sviluppo. Purché l'ambiente non sia sottoposto a override durante l'avvio dell'app tramite `UseEnvironment` nel generatore di host, l'impostazione della variabile di ambiente consente di visualizzare la [pagina delle eccezioni per gli sviluppatori](xref:fundamentals/error-handling) quando viene eseguita l'app.

```xml
<aspNetCore processPath="dotnet"
      arguments=".\MyApp.dll"
      stdoutLogEnabled="false"
      stdoutLogFile=".\logs\stdout"
      hostingModel="InProcess">
  <environmentVariables>
    <environmentVariable name="ASPNETCORE_ENVIRONMENT" value="Development" />
  </environmentVariables>
</aspNetCore>
```

L'impostazione della variabile di ambiente per `ASPNETCORE_ENVIRONMENT` è consigliata solo per l'uso in server di gestione temporanea e test non esposti a Internet. Al termine della risoluzione dei problemi, rimuovere la variabile di ambiente dal file *web.config*. Per informazioni sull'impostazione delle variabili di ambiente in *web.config*, vedere [elemento figlio environmentVariables di aspNetCore](xref:host-and-deploy/aspnet-core-module#setting-environment-variables).

### <a name="obtain-data-from-an-app"></a>Ottenere dati da un'app

Se un'app è in grado di rispondere alle richieste, è possibile ottenere dati sulle richieste, le connessioni e altri dati dall'app tramite middleware inline di terminale. Per altre informazioni e codice di esempio, vedere <xref:test/troubleshoot#obtain-data-from-an-app>.

### <a name="slow-or-hanging-app-iis"></a>App lenta o sospesa (IIS)

Un *dump di arresto anomalo* del sistema è uno snapshot della memoria del sistema e può contribuire a determinare la provocazione di un arresto anomalo dell'app, dell'avvio o dell'applicazione lenta.

#### <a name="app-crashes-or-encounters-an-exception"></a>Arresto anomalo o eccezione di un'app

Ottenere e analizzare un dump da [Segnalazione errori Windows](/windows/desktop/wer/windows-error-reporting):

1. Creare una cartella per i file dump di arresto anomalo del sistema in `c:\dumps`. Il pool di app deve avere accesso in scrittura alla cartella.
1. Eseguire lo [script di PowerShell EnableDumps](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/test/troubleshoot-azure-iis/scripts/EnableDumps.ps1):
   * Se l'app usa il [modello di hosting in-process](xref:host-and-deploy/iis/index#in-process-hosting-model), eseguire lo script per *w3wp.exe*:

     ```console
     .\EnableDumps w3wp.exe c:\dumps
     ```

   * Se l'app usa il [modello di hosting out-of-process](xref:host-and-deploy/iis/index#out-of-process-hosting-model), eseguire lo script per *dotnet.exe*:

     ```console
     .\EnableDumps dotnet.exe c:\dumps
     ```

1. Eseguire l'app nelle condizioni che causano l'arresto anomalo.
1. Dopo che si è verificato l'arresto anomalo, eseguire lo [script di PowerShell DisableDumps](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/test/troubleshoot-azure-iis/scripts/DisableDumps.ps1):
   * Se l'app usa il [modello di hosting in-process](xref:host-and-deploy/iis/index#in-process-hosting-model), eseguire lo script per *w3wp.exe*:

     ```console
     .\DisableDumps w3wp.exe
     ```

   * Se l'app usa il [modello di hosting out-of-process](xref:host-and-deploy/iis/index#out-of-process-hosting-model), eseguire lo script per *dotnet.exe*:

     ```console
     .\DisableDumps dotnet.exe
     ```

Dopo l'arresto anomalo di un'app e la raccolta dei dump, l'app può terminare normalmente. Lo script di PowerShell configura Segnalazione errori Windows per raccogliere fino a cinque dump per ogni app.

> [!WARNING]
> I dump di arresto anomalo del sistema potrebbero richiedere una grande quantità di spazio su disco (fino a diversi gigabyte ognuno).

#### <a name="app-hangs-fails-during-startup-or-runs-normally"></a>L'app si blocca, si verifica un errore durante l'avvio o viene eseguita normalmente

Quando un'app si *blocca* (smette di rispondere ma non si arresta in modo anomalo), si verifica un errore durante l'avvio o viene eseguita normalmente, vedere [file di dump in modalità utente: scegliere lo strumento migliore](/windows-hardware/drivers/debugger/user-mode-dump-files#choosing-the-best-tool) per selezionare uno strumento appropriato per produrre il dump.

#### <a name="analyze-the-dump"></a>Analizzare il dump

È possibile analizzare un dump usando diversi approcci. Per altre informazioni, vedere [Analyzing a User-Mode Dump File](/windows-hardware/drivers/debugger/analyzing-a-user-mode-dump-file) (Analisi di un file dump in modalità utente).

## <a name="clear-package-caches"></a>Cancella cache di pacchetti

Un'app funzionante potrebbe non riuscire immediatamente dopo l'aggiornamento del .NET Core SDK nel computer di sviluppo o la modifica delle versioni del pacchetto all'interno dell'app. In alcuni casi i pacchetti incoerenti possono interrompere un'app quando si eseguono aggiornamenti principali. La maggior parte di questi problemi può essere risolta attenendosi alle istruzioni seguenti:

1. Eliminare le cartelle *bin* e *obj*.
1. Cancellare le cache dei pacchetti eseguendo [le impostazioni locali di DotNet NuGet All--Clear](/dotnet/core/tools/dotnet-nuget-locals) da una shell dei comandi.

   La cancellazione delle cache dei pacchetti può essere eseguita anche con lo strumento [nuget.exe](https://www.nuget.org/downloads) e l'esecuzione del comando `nuget locals all -clear` . *nuget.exe* non è un'installazione inclusa con il sistema operativo desktop Windows e deve essere ottenuta separatamente dal [sito Web NuGet](https://www.nuget.org/downloads).

1. Ripristinare e ricompilare il progetto.
1. Eliminare tutti i file nella cartella di distribuzione nel server prima di ridistribuire l'app.

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:test/troubleshoot>
* <xref:host-and-deploy/azure-iis-errors-reference>
* <xref:fundamentals/error-handling>
* <xref:host-and-deploy/aspnet-core-module>

### <a name="azure-documentation"></a>Documentazione di Azure

* [Application Insights per ASP.NET Core](/azure/application-insights/app-insights-asp-net-core)
* [Sezione app Web per il debug remoto di risoluzione dei problemi di un'app Web nel servizio app Azure con Visual Studio](/azure/app-service/web-sites-dotnet-troubleshoot-visual-studio#remotedebug)
* [Panoramica della diagnostica del servizio app di Azure](/azure/app-service/app-service-diagnostics)
* [Procedura: Eseguire il monitoraggio delle app nel servizio app di Azure](/azure/app-service/web-sites-monitor)
* [Risoluzione dei problemi di un'app Web nel servizio app di Azure tramite Visual Studio](/azure/app-service/web-sites-dotnet-troubleshoot-visual-studio)
* [Risolvere gli errori HTTP "502 - Gateway non valido" e "503 - Servizio non disponibile" nelle App Web di Azure](/azure/app-service/app-service-web-troubleshoot-http-502-http-503)
* [Risoluzione dei problemi di rallentamento delle prestazioni delle app Web nel Servizio app di Azure](/azure/app-service/app-service-web-troubleshoot-performance-degradation)
* [Domande frequenti sulle prestazioni delle applicazioni in App Web di Azure](/azure/app-service/app-service-web-availability-performance-application-issues-faq)
* [Azure Web App sandbox (App Service runtime execution limitations)](https://github.com/projectkudu/kudu/wiki/Azure-Web-App-sandbox) (Sandbox per app Web di Azure - Limitazioni di esecuzione di runtime di Servizio app di Azure)
* [Azure Friday: Azure App Service Diagnostic and Troubleshooting Experience](https://channel9.msdn.com/Shows/Azure-Friday/Azure-App-Service-Diagnostic-and-Troubleshooting-Experience) (Azure Friday: diagnostica e risoluzione dei problemi del servizio app di Azure) (video di 12 minuti)

### <a name="visual-studio-documentation"></a>Documentazione di Visual Studio

* [ASP.NET Core di debug remoto in IIS in Azure in Visual Studio 2017](/visualstudio/debugger/remote-debugging-azure)
* [ASP.NET Core di debug remoto in un computer IIS remoto in Visual Studio 2017](/visualstudio/debugger/remote-debugging-aspnet-on-a-remote-iis-computer)
* [Informazioni sul debug con Visual Studio](/visualstudio/debugger/getting-started-with-the-debugger)

### <a name="visual-studio-code-documentation"></a>Documentazione di Visual Studio Code

* [Debug con Visual Studio Code](https://code.visualstudio.com/docs/editor/debugging)

::: moniker-end

::: moniker range="< aspnetcore-2.2"

Questo articolo fornisce informazioni sugli errori comuni di avvio delle app e istruzioni su come diagnosticare gli errori quando un'app viene distribuita in app Azure servizio o IIS:

[Errori di avvio dell'app](#app-startup-errors)  
Vengono illustrati gli scenari di codice di stato HTTP di avvio comuni.

[Risolvere i problemi relativi al servizio app Azure](#troubleshoot-on-azure-app-service)  
Fornisce consigli per la risoluzione dei problemi per le app distribuite nel servizio app Azure.

[Risolvere i problemi in IIS](#troubleshoot-on-iis)  
Fornisce consigli per la risoluzione dei problemi per le app distribuite in IIS o in esecuzione in IIS Express localmente. Le linee guida sono valide per le distribuzioni di Windows Server e Windows desktop.

[Cancella cache di pacchetti](#clear-package-caches)  
Spiega cosa fare quando i pacchetti incoerenti interrompono un'app durante l'esecuzione di aggiornamenti principali o la modifica delle versioni del pacchetto.

[Risorse aggiuntive](#additional-resources)  
Elenca ulteriori argomenti sulla risoluzione dei problemi.

## <a name="app-startup-errors"></a>Errori di avvio dell'app

In Visual Studio, per impostazione predefinita, un progetto ASP.NET Core viene [IIS Express](/iis/extensions/introduction-to-iis-express/iis-express-overview) host durante il debug. Un *errore del processo 502,5* che si verifica quando è possibile diagnosticare localmente il debug usando i consigli in questo argomento.

### <a name="40314-forbidden"></a>403,14 non consentito

Non è possibile avviare l'app. Viene registrato l'errore seguente:

```
The Web server is configured to not list the contents of this directory.
```

L'errore è in genere causato da una distribuzione interruppe sul sistema host, che include uno degli scenari seguenti:

* L'app viene distribuita nella cartella errata del sistema di hosting.
* Il processo di distribuzione non è riuscito a spostare tutti i file e le cartelle dell'app nella cartella di distribuzione nel sistema di hosting.
* Il file di *web.config* non è presente nella distribuzione oppure il contenuto del file di *web.config* non è valido.

Eseguire la procedura seguente:

1. Eliminare tutti i file e le cartelle dalla cartella di distribuzione nel sistema di hosting.
1. Ridistribuire il contenuto della cartella di *pubblicazione* dell'app nel sistema host usando il normale metodo di distribuzione, ad esempio Visual Studio, PowerShell o la distribuzione manuale:
   * Verificare che il file *web.config* sia presente nella distribuzione e che il relativo contenuto sia corretto.
   * Quando si esegue l'hosting in app Azure servizio, verificare che l'app venga distribuita nella `D:\home\site\wwwroot` cartella.
   * Quando l'app è ospitata da IIS, verificare che l'app venga distribuita nel **percorso fisico** IIS visualizzato nelle impostazioni di **base** di **Gestione IIS**.
1. Verificare che tutti i file e le cartelle dell'app vengano distribuiti confrontando la distribuzione nel sistema di hosting con il contenuto della cartella di *pubblicazione* del progetto.

Per ulteriori informazioni sul layout di un'app ASP.NET Core pubblicata, vedere <xref:host-and-deploy/directory-structure> . Per ulteriori informazioni sul file di *web.config* , vedere <xref:host-and-deploy/aspnet-core-module#configuration-with-webconfig> .

### <a name="500-internal-server-error"></a>500 - Errore interno del server

L'app viene avviata, ma un errore impedisce al server di soddisfare la richiesta.

Questo errore si verifica all'interno del codice dell'app durante l'avvio o durante la creazione di una risposta. La risposta potrebbe non avere alcun contenuto o essere visualizzata nel browser come un codice *500 Errore interno del server*. Il log eventi dell'applicazione in genere indica che l'app è stata avviata normalmente. Dal punto di vista del server, questo è corretto. L'app è stata effettivamente avviata, ma non può generare una risposta valida. Per risolvere il problema, eseguire l'app dal prompt dei comandi nel server o abilitare il log stdout del modulo ASP.NET Core.

### <a name="5025-process-failure"></a>502.5 Process Failure (Errore del processo)

Il processo di lavoro ha esito negativo. L'app non viene avviata.

Il [modulo ASP.NET Core](xref:host-and-deploy/aspnet-core-module) tenta di avviare il processo di lavoro, ma l'avvio non riesce. In genere è possibile determinare la causa di un errore di avvio del processo dalle voci nel log eventi dell'applicazione e nel log stdout del modulo ASP.NET Core.

Una condizione di errore comune è dovuta alla configurazione non corretta dell'app perché come destinazione viene specificata una versione del framework condiviso ASP.NET Core, che non è presente. Controllare le versioni del framework condiviso ASP.NET Core installate nel computer di destinazione. Il *framework condiviso* è il set di assembly (file *DLL*) che vengono installati nel computer e cui fa riferimento un metapacchetto, ad esempio `Microsoft.AspNetCore.App`. Il riferimento del metapacchetto può specificare una versione minima richiesta. Per altre informazioni, vedere [The shared framework](https://natemcmaster.com/blog/2018/08/29/netcore-primitives-2/) (Il framework condiviso).

La pagina di errore *502.5 Errore del processo* viene restituita quando il processo di lavoro ha esito negativo a causa di un errore di configurazione dell'hosting o dell'app:

### <a name="failed-to-start-application-errorcode-0x800700c1"></a>Non è stato possibile avviare l'aggiornamento dell'applicazione. Errore: 0x800700c1

```
EventID: 1010
Source: IIS AspNetCore Module V2
Failed to start application '/LM/W3SVC/6/ROOT/', ErrorCode '0x800700c1'.
```

L'app non è stata avviata perché non è stato possibile caricarne l'assembly (*.dll*).

Questo errore si verifica quando è presente una mancata corrispondenza del numero di bit tra l'app pubblicata e il processo w3wp/iisexpress.

Verificare che l'impostazione del pool di app a 32 bit sia corretta:

1. Selezionare il pool di app in **Pool di applicazioni** di Gestione IIS.
1. Selezionare **Impostazioni avanzate** in **Modifica pool di applicazioni** nel pannello **Azioni**.
1. Impostare **Attiva applicazioni a 32 bit**:
   * Se si sta sviluppando un'app a 32 bit (x86), impostare il valore su `True`.
   * Se si sta sviluppando un'app a 64 bit (x64), impostare il valore su `False`.

Verificare che non esista un conflitto tra una `<Platform>` proprietà di MSBuild nel file di progetto e il bit pubblicato dell'app.

### <a name="connection-reset"></a>Connection reset (Connessione ripristinata)

Se si verifica un errore dopo l'invio delle intestazioni, è troppo tardi perché il server possa inviare un codice **500 Errore interno del server** quando si verifica un errore. Questo spesso accade quando si verifica un errore durante la serializzazione di oggetti complessi per una risposta. Questo tipo di errore viene visualizzato come un errore di *reimpostazione della connessione* nel client. La [registrazione delle applicazioni](xref:fundamentals/logging/index) può consentire di risolvere questi tipi di errori.

### <a name="default-startup-limits"></a>Limiti di avvio predefiniti

Il [modulo ASP.NET Core](xref:host-and-deploy/aspnet-core-module) è configurato con un valore predefinito di *startupTimeLimit* di 120 secondi. Quando si mantiene il valore predefinito, l'avvio di un'app potrebbe richiedere fino a due minuti prima che il modulo registri un errore del processo. Per informazioni sulla configurazione del modulo, vedere [Attributi dell'elemento aspNetCore](xref:host-and-deploy/aspnet-core-module#attributes-of-the-aspnetcore-element).

## <a name="troubleshoot-on-azure-app-service"></a>Risolvere i problemi relativi al servizio app Azure

[!INCLUDE [Azure App Service Preview Notice](~/includes/azure-apps-preview-notice.md)]

### <a name="application-event-log-azure-app-service"></a>Registro eventi applicazioni (servizio app Azure)

Per accedere al log eventi dell'applicazione, usare il pannello **Diagnostica e risolvi i problemi** nel portale di Azure:

1. Nel portale di Azure aprire l'app in **Servizi app**.
1. Selezionare **Diagnostica e risoluzione dei problemi**.
1. Selezionare l'intestazione **Strumenti di diagnostica**.
1. In **Strumenti di supporto** selezionare il pulsante **Eventi dell'applicazione**.
1. Esaminare l'errore più recente dalla voce *IIS AspNetCoreModule* o *IIS AspNetCoreModule V2* nella colonna **Origine**.

Un'alternativa all'uso del pannello **Diagnostica e risolvi i problemi** consiste nell'esaminare direttamente il file del log eventi dell'applicazione usando [Kudu](https://github.com/projectkudu/kudu/wiki):

1. Aprire **Strumenti avanzati** nell'area **Strumenti di sviluppo**. Selezionare il **pulsante &rarr; Vai** . Verrà aperta la console Kudu in una nuova scheda o finestra del browser.
1. Usando la barra di spostamento nella parte superiore della pagina, aprire **Console di debug** e selezionare **CMD**.
1. Aprire la cartella **LogFiles** .
1. Selezionare l'icona della matita accanto al file *eventlog.xml*.
1. Esaminare il log. Scorrere fino alla fine del log per visualizzare gli eventi più recenti.

### <a name="run-the-app-in-the-kudu-console"></a>Eseguire l'app nella console Kudu

Molti errori di avvio non producono informazioni utili nel log eventi dell'applicazione. È possibile eseguire l'app nella console di esecuzione remota [Kudu](https://github.com/projectkudu/kudu/wiki) per individuare l'errore:

1. Aprire **Strumenti avanzati** nell'area **Strumenti di sviluppo**. Selezionare il **pulsante &rarr; Vai** . Verrà aperta la console Kudu in una nuova scheda o finestra del browser.
1. Usando la barra di spostamento nella parte superiore della pagina, aprire **Console di debug** e selezionare **CMD**.

#### <a name="test-a-32-bit-x86-app"></a>Test di un'app a 32 bit (x86)

**Versione corrente**

1. `cd d:\home\site\wwwroot`
1. Eseguire l'app:
   * Se l'app è una [distribuzione dipendente dal framework](/dotnet/core/deploying/#framework-dependent-deployments-fdd):

     ```dotnetcli
     dotnet .\{ASSEMBLY NAME}.dll
     ```

   * Se l'app è una [distribuzione autonoma](/dotnet/core/deploying/#self-contained-deployments-scd):

     ```console
     {ASSEMBLY NAME}.exe
     ```

L'output della console dall'app, che visualizza gli eventuali errori, viene inviato tramite pipe alla console Kudu.

**Distribuzione dipendente dal framework in esecuzione in una versione di anteprima**

*Richiede l'installazione dell'estensione del sito del runtime ASP.NET Core {VERSION} (x86).*

1. `cd D:\home\SiteExtensions\AspNetCoreRuntime.{X.Y}.x32` (`{X.Y}` è la versione di runtime)
1. Eseguire l'app: `dotnet \home\site\wwwroot\{ASSEMBLY NAME}.dll`

L'output della console dall'app, che visualizza gli eventuali errori, viene inviato tramite pipe alla console Kudu.

#### <a name="test-a-64-bit-x64-app"></a>Test di un'app a 64 bit (x64)

**Versione corrente**

* Se l'app è una [distribuzione dipendente dal framework](/dotnet/core/deploying/#framework-dependent-deployments-fdd) a 64 bit (x64):
  1. `cd D:\Program Files\dotnet`
  1. Eseguire l'app: `dotnet \home\site\wwwroot\{ASSEMBLY NAME}.dll`
* Se l'app è una [distribuzione autonoma](/dotnet/core/deploying/#self-contained-deployments-scd):
  1. `cd D:\home\site\wwwroot`
  1. Eseguire l'app: `{ASSEMBLY NAME}.exe`

L'output della console dall'app, che visualizza gli eventuali errori, viene inviato tramite pipe alla console Kudu.

**Distribuzione dipendente dal framework in esecuzione in una versione di anteprima**

*Richiede l'installazione dell'estensione del sito del runtime ASP.NET Core {VERSION} (x64).*

1. `cd D:\home\SiteExtensions\AspNetCoreRuntime.{X.Y}.x64` (`{X.Y}` è la versione di runtime)
1. Eseguire l'app: `dotnet \home\site\wwwroot\{ASSEMBLY NAME}.dll`

L'output della console dall'app, che visualizza gli eventuali errori, viene inviato tramite pipe alla console Kudu.

### <a name="aspnet-core-module-stdout-log-azure-app-service"></a>Log stdout del modulo ASP.NET Core (servizio app Azure)

Il log stdout del modulo ASP.NET Core spesso registra utili messaggi di errore non disponibili nel log eventi dell'applicazione. Per abilitare e visualizzare i log stdout:

1. Passare al pannello **Diagnostica e risolvi i problemi** nel portale di Azure.
1. In **SELECT PROBLEM CATEGORY** (SELEZIONARE LA CATEGORIA DEL PROBLEMA) selezionare il pulsante **Web App Down** (App Web inattiva).
1. In **Suggested Solutions (Soluzioni consigliate)** > **Enable Stdout Log Redirection** (Abilita il reindirizzamento del log Stdout) selezionare il pulsante **Open Kudu Console to edit Web.Config** (Apri la console Kudu per modificare Web.Config).
1. Nella **console diagnostica** Kudu aprire le cartelle nel percorso **site** > **wwwroot**. Scorrere verso il basso fino a visualizzare il file *web.config* in fondo all'elenco.
1. Fare clic sull'icona della matita accanto al file *web.config*.
1. Impostare **stdoutLogEnabled** su `true` e cambiare il percorso di **stdoutLogFile** in: `\\?\%home%\LogFiles\stdout`.
1. Selezionare **Salva** per salvare il file *web.config* aggiornato.
1. Effettuare una richiesta all'app.
1. Tornare al portale di Azure. Selezionare il pannello **Strumenti avanzati** nell'area **STRUMENTI DI SVILUPPO**. Selezionare il **pulsante &rarr; Vai** . Verrà aperta la console Kudu in una nuova scheda o finestra del browser.
1. Usando la barra di spostamento nella parte superiore della pagina, aprire **Console di debug** e selezionare **CMD**.
1. Selezionare la cartella **LogFiles**.
1. Controllare la colonna **Data modifica** e selezionare l'icona della matita per modificare il log stdout con la data di modifica più recente.
1. Quando si apre il file di log, verrà visualizzato l'errore.

Al termine della risoluzione dei problemi, disabilitare la registrazione stdout:

1. Nella **console diagnostica** Kudu tornare al percorso **site** > **wwwroot** per visualizzare il file *web.config*. Aprire nuovamente il file **web.config** selezionando l'icona della matita.
1. Impostare **stdoutLogEnabled** su `false`.
1. Selezionare **Salva** per salvare il file.

Per altre informazioni, vedere <xref:host-and-deploy/aspnet-core-module#log-creation-and-redirection>.

> [!WARNING]
> La mancata disabilitazione del log stdout può causare un errore dell'app o del server. Non esiste alcun limite per le dimensioni dei file di log o il numero di file di log che è possibile creare. Usare la registrazione stdout solo per la risoluzione dei problemi di avvio delle app.
>
> Per la registrazione generale in un'app ASP.NET Core dopo l'avvio, usare una libreria di registrazione che limita le dimensioni dei file di log e ne esegue la rotazione. Per altre informazioni, vedere [Provider di registrazione di terze parti](xref:fundamentals/logging/index#third-party-logging-providers).

### <a name="slow-or-hanging-app-azure-app-service"></a>App lenta o sospesa (servizio app Azure)

Quando un'app risponde lentamente o si blocca durante una richiesta, vedere gli articoli seguenti:

* [Risoluzione dei problemi di rallentamento delle prestazioni delle app Web nel Servizio app di Azure](/azure/app-service/app-service-web-troubleshoot-performance-degradation)
* [Use Crash Diagnoser Site Extension to Capture Dump for Intermittent Exception issues or performance issues on Azure Web App](https://blogs.msdn.microsoft.com/asiatech/2015/12/28/use-crash-diagnoser-site-extension-to-capture-dump-for-intermittent-exception-issues-or-performance-issues-on-azure-web-app/) (Usare l'estensione del sito Crash Diagnoser per acquisire il dump per i problemi di eccezioni intermittenti o i problemi di prestazioni nell'app Web di Azure)

### <a name="monitoring-blades"></a>Pannelli di monitoraggio

I pannelli di monitoraggio forniscono un'esperienza di risoluzione dei problemi alternativa ai metodi descritti in precedenza in questo argomento. È possibile usare questi pannelli per diagnosticare gli errori della serie 500.

Verificare che le estensioni di ASP.NET Core siano installate. Se le estensioni non sono installate, installarle manualmente:

1. Nella sezione del pannello **STRUMENTI DI SVILUPPO** selezionare il pannello **Estensioni**.
1. Nell'elenco dovrebbe essere visualizzato **ASP.NET Core Extensions** (Estensioni ASP.NET Core).
1. Se le estensioni non sono installate, selezionare il pulsante **Aggiungi**.
1. Scegliere **ASP.NET Core Extensions** (Estensioni ASP.NET Core) dall'elenco.
1. Selezionare **OK** per accettare le condizioni legali.
1. Selezionare **OK** nel pannello **Aggiungi estensione**.
1. Un messaggio popup informativo indica quando le estensioni sono state installate correttamente.

Se la registrazione stdout non è abilitata, attenersi ai passaggi riportati di seguito:

1. Nel portale di Azure selezionare il pannello **Strumenti avanzati** nell'area **STRUMENTI DI SVILUPPO**. Selezionare il **pulsante &rarr; Vai** . Verrà aperta la console Kudu in una nuova scheda o finestra del browser.
1. Usando la barra di spostamento nella parte superiore della pagina, aprire **Console di debug** e selezionare **CMD**.
1. Aprire le cartelle nel percorso **site** > **wwwroot** e scorrere verso il basso fino a visualizzare il file *web.config* in fondo all'elenco.
1. Fare clic sull'icona della matita accanto al file *web.config*.
1. Impostare **stdoutLogEnabled** su `true` e cambiare il percorso di **stdoutLogFile** in: `\\?\%home%\LogFiles\stdout`.
1. Selezionare **Salva** per salvare il file *web.config* aggiornato.

Passare all'attivazione della registrazione diagnostica:

1. Nel portale di Azure selezionare il pannello **Log di diagnostica**.
1. Selezionare l'interruttore **Attivato** per **Registrazione applicazioni (file system)** e **Messaggi di errore dettagliati**. Selezionare il pulsante **Salva** nella parte superiore del pannello.
1. Per includere la traccia delle richieste non riuscite, anche nota come registrazione FREB (Failed Request Event Buffering), selezionare l'interruttore **Attivato** per **Traccia delle richieste non riuscite**.
1. Selezionare il pannello **Flusso di registrazione**, immediatamente sotto il pannello **Log di diagnostica** nel portale.
1. Effettuare una richiesta all'app.
1. Nei dati del flusso di registrazione viene indicata la causa dell'errore.

Al termine della risoluzione dei problemi, assicurarsi di disabilitare la registrazione stdout.

Per visualizzare i log di traccia delle richieste non riuscite (log FREB):

1. Passare al pannello **Diagnostica e risolvi i problemi** nel portale di Azure.
1. Selezionare **Failed Request Tracing Logs** (Log di traccia delle richieste non riuscite) nell'area **STRUMENTI DI SUPPORTO** della barra laterale.

Per altre informazioni, vedere la [sezione relativa alla traccia delle richieste non riuscite nell'argomento Abilitare la registrazione diagnostica per le app Web nel servizio app di Azure](/azure/app-service/web-sites-enable-diagnostic-log#failed-request-traces) e [Domande frequenti sulle prestazioni delle applicazioni in App Web di Azure: Come si abilita la traccia delle richieste non riuscite?](/azure/app-service/app-service-web-availability-performance-application-issues-faq#how-do-i-turn-on-failed-request-tracing).

Per altre informazioni, vedere [Abilitare la registrazione diagnostica per le app Web nel servizio app di Azure](/azure/app-service/web-sites-enable-diagnostic-log).

> [!WARNING]
> La mancata disabilitazione del log stdout può causare un errore dell'app o del server. Non esiste alcun limite per le dimensioni dei file di log o il numero di file di log che è possibile creare.
>
> Per la registrazione di routine in un'app ASP.NET Core, usare una libreria di registrazione che limita le dimensioni dei file di log e ne esegue la rotazione. Per altre informazioni, vedere [Provider di registrazione di terze parti](xref:fundamentals/logging/index#third-party-logging-providers).

## <a name="troubleshoot-on-iis"></a>Risolvere i problemi in IIS

### <a name="application-event-log-iis"></a>Registro eventi applicazioni (IIS)

Accedere al log eventi dell'applicazione:

1. Aprire il menu Start, cercare *Visualizzatore eventi* e selezionare l'app **Visualizzatore eventi** .
1. In **Visualizzatore eventi** aprire il nodo **Registri di Windows**.
1. Selezionare **Applicazione** per aprire il log eventi dell'applicazione.
1. Cercare gli errori associati all'app in cui si è verificato il problema. Gli errori presentano un valore *Modulo AspNetCore IIS* o *Modulo AspNetCore IIS Express* nella colonna *Origine*.

### <a name="run-the-app-at-a-command-prompt"></a>Eseguire l'app da un prompt dei comandi

Molti errori di avvio non producono informazioni utili nel log eventi dell'applicazione. È possibile individuare la causa di alcuni errori eseguendo l'app da un prompt dei comandi nel sistema host.

#### <a name="framework-dependent-deployment"></a>Distribuzione dipendente dal framework

Se l'app è una [distribuzione dipendente dal framework](/dotnet/core/deploying/#framework-dependent-deployments-fdd):

1. Da un prompt dei comandi passare alla cartella di distribuzione e avviare l'app eseguendo l'assembly dell'app con *dotnet.exe*. Nel comando seguente sostituire il nome dell'assembly dell'app per \<assembly_name> : `dotnet .\<assembly_name>.dll` .
1. L'output della console per l'app, in cui sono indicati gli eventuali errori, verrà scritto nella finestra della console.
1. Se gli errori si verificano quando si effettua una richiesta all'app, effettuare una richiesta all'host e alla porta su cui è in ascolto Kestrel. Usando l'host e la porta predefiniti, effettuare una richiesta a `http://localhost:5000/`. Se l'app risponde normalmente nell'indirizzo endpoint di Kestrel, è più probabile che il problema sia associato alla configurazione dell'host e che non sia interno all'app.

#### <a name="self-contained-deployment"></a>Distribuzione autonoma

Se l'app è una [distribuzione autonoma](/dotnet/core/deploying/#self-contained-deployments-scd):

1. Da un prompt dei comandi passare alla cartella di distribuzione e avviare il file eseguibile dell'app. Nel comando seguente sostituire il nome dell'assembly dell'app per \<assembly_name> : `<assembly_name>.exe` .
1. L'output della console per l'app, in cui sono indicati gli eventuali errori, verrà scritto nella finestra della console.
1. Se gli errori si verificano quando si effettua una richiesta all'app, effettuare una richiesta all'host e alla porta su cui è in ascolto Kestrel. Usando l'host e la porta predefiniti, effettuare una richiesta a `http://localhost:5000/`. Se l'app risponde normalmente nell'indirizzo endpoint di Kestrel, è più probabile che il problema sia associato alla configurazione dell'host e che non sia interno all'app.

### <a name="aspnet-core-module-stdout-log-iis"></a>Log stdout del modulo ASP.NET Core (IIS)

Per abilitare e visualizzare i log stdout:

1. Passare alla cartella di distribuzione del sito nel sistema host.
1. Se la cartella *logs* non è presente, creare la cartella. Per istruzioni su come impostare MSBuild per la creazione automatica della cartella *logs* nella distribuzione, vedere l'argomento [Struttura della directory](xref:host-and-deploy/directory-structure).
1. Modificare il file *web.config*. Impostare **stdoutLogEnabled** su `true` e modificare il percorso di **stdoutLogFile** in modo da fare riferimento alla cartella *logs*, ad esempio `.\logs\stdout`. `stdout` nel percorso è il prefisso del nome del file di log. Un timestamp, l'ID del processo e l'estensione del file vengono aggiunti automaticamente al momento della creazione del log. Usando `stdout` come prefisso del nome del file, un tipico file di log è denominato *stdout_20180205184032_5412.log*.
1. Assicurarsi che l'identità del pool di applicazioni disponga delle autorizzazioni di scrittura per la cartella *logs*.
1. Salvare il file *web.config* aggiornato.
1. Effettuare una richiesta all'app.
1. Passare alla cartella *logs*. Trovare e aprire il log stdout più recente.
1. Esaminare il log per verificare se sono presenti errori.

Al termine della risoluzione dei problemi, disabilitare la registrazione stdout:

1. Modificare il file *web.config*.
1. Impostare **stdoutLogEnabled** su `false`.
1. Salvare il file.

Per altre informazioni, vedere <xref:host-and-deploy/aspnet-core-module#log-creation-and-redirection>.

> [!WARNING]
> La mancata disabilitazione del log stdout può causare un errore dell'app o del server. Non esiste alcun limite per le dimensioni dei file di log o il numero di file di log che è possibile creare.
>
> Per la registrazione di routine in un'app ASP.NET Core, usare una libreria di registrazione che limita le dimensioni dei file di log e ne esegue la rotazione. Per altre informazioni, vedere [Provider di registrazione di terze parti](xref:fundamentals/logging/index#third-party-logging-providers).

### <a name="enable-the-developer-exception-page"></a>Abilitare la pagina delle eccezioni per gli sviluppatori

La variabile di ambiente `ASPNETCORE_ENVIRONMENT` [ può essere aggiunta a web.config](xref:host-and-deploy/aspnet-core-module#setting-environment-variables) per eseguire l'app nell'ambiente di sviluppo. Purché l'ambiente non sia sottoposto a override durante l'avvio dell'app tramite `UseEnvironment` nel generatore di host, l'impostazione della variabile di ambiente consente di visualizzare la [pagina delle eccezioni per gli sviluppatori](xref:fundamentals/error-handling) quando viene eseguita l'app.

```xml
<aspNetCore processPath="dotnet"
      arguments=".\MyApp.dll"
      stdoutLogEnabled="false"
      stdoutLogFile=".\logs\stdout">
  <environmentVariables>
    <environmentVariable name="ASPNETCORE_ENVIRONMENT" value="Development" />
  </environmentVariables>
</aspNetCore>
```

L'impostazione della variabile di ambiente per `ASPNETCORE_ENVIRONMENT` è consigliata solo per l'uso in server di gestione temporanea e test non esposti a Internet. Al termine della risoluzione dei problemi, rimuovere la variabile di ambiente dal file *web.config*. Per informazioni sull'impostazione delle variabili di ambiente in *web.config*, vedere [elemento figlio environmentVariables di aspNetCore](xref:host-and-deploy/aspnet-core-module#setting-environment-variables).

### <a name="obtain-data-from-an-app"></a>Ottenere dati da un'app

Se un'app è in grado di rispondere alle richieste, è possibile ottenere dati sulle richieste, le connessioni e altri dati dall'app tramite middleware inline di terminale. Per altre informazioni e codice di esempio, vedere <xref:test/troubleshoot#obtain-data-from-an-app>.

### <a name="slow-or-hanging-app-iis"></a>App lenta o sospesa (IIS)

Un *dump di arresto anomalo* del sistema è uno snapshot della memoria del sistema e può contribuire a determinare la provocazione di un arresto anomalo dell'app, dell'avvio o dell'applicazione lenta.

#### <a name="app-crashes-or-encounters-an-exception"></a>Arresto anomalo o eccezione di un'app

Ottenere e analizzare un dump da [Segnalazione errori Windows](/windows/desktop/wer/windows-error-reporting):

1. Creare una cartella per i file dump di arresto anomalo del sistema in `c:\dumps`. Il pool di app deve avere accesso in scrittura alla cartella.
1. Eseguire lo [script di PowerShell EnableDumps](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/test/troubleshoot-azure-iis/scripts/EnableDumps.ps1):
   * Se l'app usa il [modello di hosting in-process](xref:host-and-deploy/iis/index#in-process-hosting-model), eseguire lo script per *w3wp.exe*:

     ```console
     .\EnableDumps w3wp.exe c:\dumps
     ```

   * Se l'app usa il [modello di hosting out-of-process](xref:host-and-deploy/iis/index#out-of-process-hosting-model), eseguire lo script per *dotnet.exe*:

     ```console
     .\EnableDumps dotnet.exe c:\dumps
     ```

1. Eseguire l'app nelle condizioni che causano l'arresto anomalo.
1. Dopo che si è verificato l'arresto anomalo, eseguire lo [script di PowerShell DisableDumps](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/test/troubleshoot-azure-iis/scripts/DisableDumps.ps1):
   * Se l'app usa il [modello di hosting in-process](xref:host-and-deploy/iis/index#in-process-hosting-model), eseguire lo script per *w3wp.exe*:

     ```console
     .\DisableDumps w3wp.exe
     ```

   * Se l'app usa il [modello di hosting out-of-process](xref:host-and-deploy/iis/index#out-of-process-hosting-model), eseguire lo script per *dotnet.exe*:

     ```console
     .\DisableDumps dotnet.exe
     ```

Dopo l'arresto anomalo di un'app e la raccolta dei dump, l'app può terminare normalmente. Lo script di PowerShell configura Segnalazione errori Windows per raccogliere fino a cinque dump per ogni app.

> [!WARNING]
> I dump di arresto anomalo del sistema potrebbero richiedere una grande quantità di spazio su disco (fino a diversi gigabyte ognuno).

#### <a name="app-hangs-fails-during-startup-or-runs-normally"></a>L'app si blocca, si verifica un errore durante l'avvio o viene eseguita normalmente

Quando un'app si *blocca* (smette di rispondere ma non si arresta in modo anomalo), si verifica un errore durante l'avvio o viene eseguita normalmente, vedere [file di dump in modalità utente: scegliere lo strumento migliore](/windows-hardware/drivers/debugger/user-mode-dump-files#choosing-the-best-tool) per selezionare uno strumento appropriato per produrre il dump.

#### <a name="analyze-the-dump"></a>Analizzare il dump

È possibile analizzare un dump usando diversi approcci. Per altre informazioni, vedere [Analyzing a User-Mode Dump File](/windows-hardware/drivers/debugger/analyzing-a-user-mode-dump-file) (Analisi di un file dump in modalità utente).

## <a name="clear-package-caches"></a>Cancella cache di pacchetti

Un'app funzionante potrebbe non riuscire immediatamente dopo l'aggiornamento del .NET Core SDK nel computer di sviluppo o la modifica delle versioni del pacchetto all'interno dell'app. In alcuni casi i pacchetti incoerenti possono interrompere un'app quando si eseguono aggiornamenti principali. La maggior parte di questi problemi può essere risolta attenendosi alle istruzioni seguenti:

1. Eliminare le cartelle *bin* e *obj*.
1. Cancellare le cache dei pacchetti eseguendo [le impostazioni locali di DotNet NuGet All--Clear](/dotnet/core/tools/dotnet-nuget-locals) da una shell dei comandi.

   La cancellazione delle cache dei pacchetti può essere eseguita anche con lo strumento [nuget.exe](https://www.nuget.org/downloads) e l'esecuzione del comando `nuget locals all -clear` . *nuget.exe* non è un'installazione inclusa con il sistema operativo desktop Windows e deve essere ottenuta separatamente dal [sito Web NuGet](https://www.nuget.org/downloads).

1. Ripristinare e ricompilare il progetto.
1. Eliminare tutti i file nella cartella di distribuzione nel server prima di ridistribuire l'app.

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:test/troubleshoot>
* <xref:host-and-deploy/azure-iis-errors-reference>
* <xref:fundamentals/error-handling>
* <xref:host-and-deploy/aspnet-core-module>

### <a name="azure-documentation"></a>Documentazione di Azure

* [Application Insights per ASP.NET Core](/azure/application-insights/app-insights-asp-net-core)
* [Sezione app Web per il debug remoto di risoluzione dei problemi di un'app Web nel servizio app Azure con Visual Studio](/azure/app-service/web-sites-dotnet-troubleshoot-visual-studio#remotedebug)
* [Panoramica della diagnostica del servizio app di Azure](/azure/app-service/app-service-diagnostics)
* [Procedura: Eseguire il monitoraggio delle app nel servizio app di Azure](/azure/app-service/web-sites-monitor)
* [Risoluzione dei problemi di un'app Web nel servizio app di Azure tramite Visual Studio](/azure/app-service/web-sites-dotnet-troubleshoot-visual-studio)
* [Risolvere gli errori HTTP "502 - Gateway non valido" e "503 - Servizio non disponibile" nelle App Web di Azure](/azure/app-service/app-service-web-troubleshoot-http-502-http-503)
* [Risoluzione dei problemi di rallentamento delle prestazioni delle app Web nel Servizio app di Azure](/azure/app-service/app-service-web-troubleshoot-performance-degradation)
* [Domande frequenti sulle prestazioni delle applicazioni in App Web di Azure](/azure/app-service/app-service-web-availability-performance-application-issues-faq)
* [Azure Web App sandbox (App Service runtime execution limitations)](https://github.com/projectkudu/kudu/wiki/Azure-Web-App-sandbox) (Sandbox per app Web di Azure - Limitazioni di esecuzione di runtime di Servizio app di Azure)
* [Azure Friday: Azure App Service Diagnostic and Troubleshooting Experience](https://channel9.msdn.com/Shows/Azure-Friday/Azure-App-Service-Diagnostic-and-Troubleshooting-Experience) (Azure Friday: diagnostica e risoluzione dei problemi del servizio app di Azure) (video di 12 minuti)

### <a name="visual-studio-documentation"></a>Documentazione di Visual Studio

* [ASP.NET Core di debug remoto in IIS in Azure in Visual Studio 2017](/visualstudio/debugger/remote-debugging-azure)
* [ASP.NET Core di debug remoto in un computer IIS remoto in Visual Studio 2017](/visualstudio/debugger/remote-debugging-aspnet-on-a-remote-iis-computer)
* [Informazioni sul debug con Visual Studio](/visualstudio/debugger/getting-started-with-the-debugger)

### <a name="visual-studio-code-documentation"></a>Documentazione di Visual Studio Code

* [Debug con Visual Studio Code](https://code.visualstudio.com/docs/editor/debugging)

::: moniker-end
