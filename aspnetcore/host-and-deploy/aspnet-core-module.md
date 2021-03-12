---
title: Modulo ASP.NET Core
author: rick-anderson
description: Informazioni sul modulo ASP.NET Core per l'hosting di app ASP.NET Core con IIS.
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 01/13/2020
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
uid: host-and-deploy/aspnet-core-module
ms.openlocfilehash: 8574bc06b7c27fbb8caf834754188dba67bcb4e3
ms.sourcegitcommit: acfe51c35497a204f75c2a61125c9408c04493e6
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102605697"
---
# <a name="aspnet-core-module"></a>Modulo ASP.NET Core

Di [Tom Dykstra](https://github.com/tdykstra), [Rick Strahl](https://github.com/RickStrahl), [Chris Ross](https://github.com/Tratcher), [Rick Anderson](https://twitter.com/RickAndMSFT), [Shirhatti](https://twitter.com/sshirhatti)e [Justin Kotalik](https://github.com/jkotalik)

::: moniker range=">= aspnetcore-5.0"

Il modulo ASP.NET Core è un modulo IIS nativo che si connette alla pipeline IIS, consentendo a ASP.NET Core applicazioni di utilizzare IIS. Eseguire ASP.NET Core app con IIS in uno dei seguenti casi: 

* Hosting di un'app ASP.NET Core all'interno del processo di lavoro IIS ( `w3wp.exe` ), denominato [modello di hosting in-process](xref:host-and-deploy/iis/in-process-hosting).
* L'invio di richieste Web a un'app ASP.NET Core back-end che esegue il server gheppio, denominato [modello di hosting out-of-process](xref:host-and-deploy/iis/out-of-process-hosting).

Esistono compromessi tra i modelli di hosting. Per impostazione predefinita, il modello di hosting in-process viene usato a causa di prestazioni e diagnostica migliori.

## <a name="install-aspnet-core-module"></a>Installare ASP.NET Core modulo

Scaricare il programma di installazione mediante il collegamento seguente:

[Programma di installazione del bundle di hosting .NET Core corrente (download diretto)](https://dotnet.microsoft.com/permalink/dotnetcore-current-windows-runtime-bundle-installer)

Per altre informazioni dettagliate su come installare il modulo ASP.NET Core o installare versioni diverse del modulo, vedere [Install the .NET Core hosting bundle](xref:host-and-deploy/iis/hosting-bundle).

Per un'esercitazione sulla pubblicazione di un'app ASP.NET Core in un server IIS, vedere <xref:tutorials/publish-to-iis>.

::: moniker-end

::: moniker range=">= aspnetcore-3.0 < aspnetcore-5.0"

Il modulo ASP.NET Core è un modulo IIS nativo collegato alla pipeline di IIS per eseguire le operazioni seguenti:

* Ospitare un app ASP.NET Core all'interno del processo di lavoro IIS (`w3wp.exe`), denominato [modello di hosting in-process](#in-process-hosting-model).
* Inoltrare le richieste Web all'app back-end ASP.NET Core che esegue il [server Kestrel](xref:fundamentals/servers/kestrel), denominato [modello di hosting out-of-process](#out-of-process-hosting-model).

Versioni supportate di Windows:

* Windows 7 o versione successiva
* Windows Server 2012 R2 o versione successiva

In caso di hosting in-process, il modulo usa un'implementazione di server in-process per IIS detta server HTTP di IIS (`IISHttpServer`).

In caso di hosting out-of-process, il modulo funziona solo con Kestrel. Il modulo non funziona con [HTTP.sys](xref:fundamentals/servers/httpsys).

## <a name="hosting-models"></a>Modelli di hosting

### <a name="in-process-hosting-model"></a>Modello di hosting in-process

ASP.NET Core app predefinite per il modello di hosting in-process.

In caso di hosting in-process, vengono applicate le caratteristiche seguenti:

* Viene usato un server HTTP di IIS (`IISHttpServer`) al posto del server [Kestrel](xref:fundamentals/servers/kestrel). Per in-process, [CreateDefaultBuilder](xref:fundamentals/host/generic-host#default-builder-settings) chiama <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderIISExtensions.UseIIS*> per:

  * Registrare `IISHttpServer`.
  * Configurare la porta e il percorso di base su cui il server deve eseguire l'ascolto in caso di esecuzione dietro il modulo ASP.NET Core.
  * Configurare l'host per l'acquisizione degli errori di avvio.

* L'[attributo requestTimeout ](#attributes-of-the-aspnetcore-element) non viene applicato all'hosting in-process.

* Non è supportata la condivisione di un pool di app tra le app. Usare un pool di app per ogni app.

* Quando si usa [distribuzione Web](/iis/publish/using-web-deploy/introduction-to-web-deploy) o si posiziona manualmente un [ `app_offline.htm` file nella distribuzione](xref:host-and-deploy/iis/index#locked-deployment-files), l'app potrebbe non arrestarsi immediatamente se è presente una connessione aperta. Ad esempio, una connessione WebSocket potrebbe ritardare l'arresto dell'app.

* L'architettura, vale a dire il numero di bit dell'app, e il runtime installato (x64 o x86) devono corrispondere all'architettura del pool di app.

* Le disconnessioni del client vengono rilevate. Il [`HttpContext.RequestAborted`](xref:Microsoft.AspNetCore.Http.HttpContext.RequestAborted*) token di annullamento viene annullato quando il client si disconnette.

* In ASP.NET Core 2.2.1 o versioni precedenti, <xref:System.IO.Directory.GetCurrentDirectory*> restituisce la directory di lavoro del processo avviato da IIS anziché la directory dell'app, ad esempio `C:\Windows\System32\inetsrv` per `w3wp.exe` .

  Per il codice di esempio che imposta la directory corrente dell'app, vedere la [ `CurrentDirectoryHelpers` classe](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/host-and-deploy/aspnet-core-module/samples_snapshot/3.x/CurrentDirectoryHelpers.cs). Chiamare il metodo `SetCurrentDirectory` . Le chiamate successive a <xref:System.IO.Directory.GetCurrentDirectory*> specificano la directory dell'app.

* In caso di hosting in-process, <xref:Microsoft.AspNetCore.Authentication.AuthenticationService.AuthenticateAsync*> non viene chiamato internamente per inizializzare un utente. Pertanto, un'implementazione di <xref:Microsoft.AspNetCore.Authentication.IClaimsTransformation> usate per trasformare le attestazioni dopo ogni autenticazione non viene attivata per impostazione predefinita. Quando si trasformano le attestazioni con un'implementazione di <xref:Microsoft.AspNetCore.Authentication.IClaimsTransformation>, chiamare <xref:Microsoft.Extensions.DependencyInjection.AuthenticationServiceCollectionExtensions.AddAuthentication*> per aggiungere i servizi di autenticazione:

  ```csharp
  public void ConfigureServices(IServiceCollection services)
  {
      services.AddTransient<IClaimsTransformation, ClaimsTransformer>();
      services.AddAuthentication(IISServerDefaults.AuthenticationScheme);
  }

  public void Configure(IApplicationBuilder app)
  {
      app.UseAuthentication();
  }
  ```
  
  * Le [distribuzioni di pacchetti Web (file singolo)](/aspnet/web-forms/overview/deployment/web-deployment-in-the-enterprise/deploying-web-packages) non sono supportate.

### <a name="out-of-process-hosting-model"></a>Modello di hosting out-of-process

Per configurare un'app per l'hosting out-of-process, impostare il valore della `<AspNetCoreHostingModel>` proprietà su `OutOfProcess` nel file di progetto ( `.csproj` ):

```xml
<PropertyGroup>
  <AspNetCoreHostingModel>OutOfProcess</AspNetCoreHostingModel>
</PropertyGroup>
```

L'hosting in-process è impostato su `InProcess` , che corrisponde al valore predefinito.

Il valore di non `<AspNetCoreHostingModel>` fa distinzione tra maiuscole e minuscole, pertanto `inprocess` e `outofprocess` sono valori validi.

Viene usato il server [Kestrel](xref:fundamentals/servers/kestrel) al posto di un server HTTP di IIS (`IISHttpServer`).

Per out-of-process, le [`CreateDefaultBuilder`](xref:fundamentals/host/generic-host#default-builder-settings) chiamate <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderIISExtensions.UseIISIntegration*> a:

* Configurare la porta e il percorso di base su cui il server deve eseguire l'ascolto in caso di esecuzione dietro il modulo ASP.NET Core.
* Configurare l'host per l'acquisizione degli errori di avvio.

### <a name="hosting-model-changes"></a>Modifiche del modello di hosting

Se l' `hostingModel` impostazione viene modificata nel `web.config` file (illustrata nella sezione [configurazione con `web.config` ](#configuration-with-webconfig) ), il modulo ricicla il processo di lavoro per IIS.

Per IIS Express, il modulo non ricicla il processo di lavoro. Attiva invece un arresto normale del processo di IIS Express corrente. La richiesta successiva all'app attiva un nuovo processo di IIS Express.

### <a name="process-name"></a>Nome del processo

`Process.GetCurrentProcess().ProcessName` dichiara `w3wp`/`iisexpress` (In-Process) o `dotnet` (out-of-process).

Molti moduli nativi, ad esempio l'autenticazione di Windows, rimangono attivi. Per altre informazioni sui moduli IIS attivi con il modulo ASP.NET Core, vedere <xref:host-and-deploy/iis/modules>.

Il modulo ASP.NET Core può anche:

* Impostare variabili di ambiente per il processo di lavoro.
* Registrare output stdout in una risorsa di archiviazione di file per la risoluzione di problemi di avvio.
* Inoltrare token di autenticazione di Windows.

## <a name="how-to-install-and-use-the-aspnet-core-module"></a>Come installare e usare il modulo ASP.NET Core

Per istruzioni su come installare e usare il modulo ASP.NET Core, vedere [Installare il bundle di hosting .NET Core](xref:host-and-deploy/iis/index#install-the-net-core-hosting-bundle).

## <a name="configuration-with-webconfig"></a>Configurazione con web.config

Il modulo ASP.NET Core viene configurato tramite la sezione `aspNetCore` del nodo `system.webServer` nel file *web.config* del sito.

Il `web.config` file seguente viene pubblicato per una [distribuzione dipendente dal Framework](/dotnet/articles/core/deploying/#framework-dependent-deployments-fdd) e configura il modulo ASP.NET Core per gestire le richieste del sito:

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <location path="." inheritInChildApplications="false">
    <system.webServer>
      <handlers>
        <add name="aspNetCore" path="*" verb="*" modules="AspNetCoreModuleV2" resourceType="Unspecified" />
      </handlers>
      <aspNetCore processPath="dotnet"
                  arguments=".\MyApp.dll"
                  stdoutLogEnabled="false"
                  stdoutLogFile=".\logs\stdout"
                  hostingModel="inprocess" />
    </system.webServer>
  </location>
</configuration>
```

Il file *web.config* seguente viene pubblicato per una [distribuzione autonoma](/dotnet/articles/core/deploying/#self-contained-deployments-scd):

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <location path="." inheritInChildApplications="false">
    <system.webServer>
      <handlers>
        <add name="aspNetCore" path="*" verb="*" modules="AspNetCoreModuleV2" resourceType="Unspecified" />
      </handlers>
      <aspNetCore processPath=".\MyApp.exe"
                  stdoutLogEnabled="false"
                  stdoutLogFile=".\logs\stdout"
                  hostingModel="inprocess" />
    </system.webServer>
  </location>
</configuration>
```

La <xref:System.Configuration.SectionInformation.InheritInChildApplications*> proprietà è impostata su `false` per indicare che le impostazioni specificate all'interno dell' [`<location>`](/iis/manage/managing-your-configuration-settings/understanding-iis-configuration-delegation#the-concept-of-location) elemento non vengono ereditate da app che si trovano in una sottodirectory dell'app.

Quando un'app viene distribuita in [Servizio app di Azure](https://azure.microsoft.com/services/app-service/), il percorso `stdoutLogFile` è impostato su `\\?\%home%\LogFiles\stdout`. Il percorso Salva i log stdout nella `LogFiles` cartella, che è un percorso creato automaticamente dal servizio.

Per informazioni sulla configurazione delle applicazioni secondarie IIS, vedere <xref:host-and-deploy/iis/index#sub-applications>.

### <a name="attributes-of-the-aspnetcore-element"></a>Attributi dell'elemento aspNetCore

| Attributo | Descrizione | Predefinito |
| --------- | ----------- | :-----: |
| `arguments` | <p>Attributo stringa facoltativo.</p><p>Argomenti per l'eseguibile specificato in **processPath**.</p> | |
| `disableStartUpErrorPage` | <p>Attributo booleano facoltativo.</p><p>Se true, la pagina **502.5 - Errore del processo** non viene visualizzata e la tabella codici di stato 502 configurata in *web.config* ha la precedenza.</p> | `false` |
| `forwardWindowsAuthToken` | <p>Attributo booleano facoltativo.</p><p>Se true, il token viene inviato al processo figlio in ascolto `%ASPNETCORE_PORT%` come intestazione `'MS-ASPNETCORE-WINAUTHTOKEN'` per ogni richiesta. È responsabilità del processo chiamare CloseHandle su questo token per ogni richiesta.</p> | `true` |
| `hostingModel` | <p>Attributo stringa facoltativo.</p><p>Specifica il modello di hosting come in-process ( `InProcess` / `inprocess` ) o out-of-process ( `OutOfProcess` / `outofprocess` ).</p> | `InProcess`<br>`inprocess` |
| `processesPerApplication` | <p>Attributo Integer facoltativo.</p><p>Specifica il numero di istanze del processo specificato nell'impostazione **processPath** che può essere riattivato per ogni app.</p><p>&dagger;Per l'hosting in-process, il valore è limitato a `1`.</p><p>L'impostazione di `processesPerApplication` è sconsigliata. Questo attributo sarà rimosso nelle versioni future.</p> | Valore predefinito: `1`<br>Min: `1`<br>Max: `100`&dagger; |
| `processPath` | <p>Attributo stringa obbligatorio.</p><p>Percorso del file eseguibile che avvia un processo in ascolto delle richieste HTTP. I percorsi relativi sono supportati. Se il percorso inizia con `.`, viene considerato relativo alla radice del sito.</p> | |
| `rapidFailsPerMinute` | <p>Attributo Integer facoltativo.</p><p>Specifica il numero di arresti anomali al minuto per il processo specificato in **processPath**. Se questo limite viene superato, il modulo smette di avviare il processo per la parte restante del minuto.</p><p>Non supportato con l'hosting in-process.</p> | Valore predefinito: `10`<br>Min: `0`<br>Max: `100` |
| `requestTimeout` | <p>Attributo Timespan facoltativo.</p><p>Specifica la durata per cui il modulo ASP.NET Core attende una risposta dal processo in ascolto su %ASPNETCORE_PORT%.</p><p>Nelle versioni del modulo ASP.NET Core fornito con ASP.NET Core 2.1 o versioni successive, `requestTimeout` viene specificato in ore, minuti e secondi.</p><p>Non è applicabile all'hosting in-process. Per l'hosting in-process, il modulo resta in attesa che l'app elabori la richiesta.</p><p>I valori validi per i segmenti della stringa relativi a minuti e secondi sono compresi nell'intervallo tra 0 e 59. Se si usa **60** come valore per i minuti o i secondi, viene generato un errore *500 - Errore interno del server*.</p> | Valore predefinito: `00:02:00`<br>Min: `00:00:00`<br>Max: `360:00:00` |
| `shutdownTimeLimit` | <p>Attributo Integer facoltativo.</p><p>Durata in secondi per cui il modulo attende che il file eseguibile venga arrestato normalmente quando viene rilevato il file *app_offline.htm*.</p> | Valore predefinito: `10`<br>Min: `0`<br>Max: `600` |
| `startupTimeLimit` | <p>Attributo Integer facoltativo.</p><p>Durata in secondi per cui il modulo attende l'avvio di un processo in ascolto sulla porta da parte del file eseguibile. Se questo limite di tempo viene superato, il modulo termina il processo.</p><p>Quando si esegue l'hosting *in-process*: il processo **non** viene riavviato **e non usa** l'impostazione **rapidFailsPerMinute** .</p><p>Quando si ospita *out-of-process*: il modulo tenta di riavviare il processo quando riceve una nuova richiesta e continua a provare a riavviare il processo per le successive richieste in ingresso, a meno che l'app non riesca ad avviare **rapidFailsPerMinute** numero di volte nell'ultimo minuto in sequenza.</p><p>Un valore pari a 0 (zero) **non** è considerato un timeout infinito.</p> | Valore predefinito: `120`<br>Min: `0`<br>Max: `3600` |
| `stdoutLogEnabled` | <p>Attributo booleano facoltativo.</p><p>Se true, **stdout** e **stderr** per il processo specificato in **processPath** vengono reindirizzati al file specificato in **stdoutLogFile**.</p> | `false` |
| `stdoutLogFile` | <p>Attributo stringa facoltativo.</p><p>Specifica il percorso relativo o assoluto per cui vengono registrati **stdout** e **stderr** dal processo specificato in **processPath**. I percorsi relativi sono relativi alla radice del sito. Qualsiasi percorso che inizia con `.` è relativo al sito radice e tutti gli altri percorsi vengono trattati come percorsi assoluti. Le eventuali cartelle specificate nel percorso vengono create dal modulo quando viene creato il file di log. L'utilizzo di delimitatori di sottolineatura, timestamp, ID processo ed estensione di file ( `.log` ) viene aggiunto all'ultimo segmento del percorso **stdoutLogFile** . Se `.\logs\stdout` viene fornito come valore, un esempio di log stdout viene salvato come `stdout_20180205194132_1934.log` nella `logs` cartella quando viene salvato il 2/5/2018 alle 19:41:32 con l'ID di processo 1934.</p> | `aspnetcore-stdout` |

### <a name="set-environment-variables"></a>Impostare le variabili di ambiente

È possibile specificare le variabili di ambiente per il processo nell'attributo `processPath`. Specificare una variabile di ambiente con l'elemento figlio `<environmentVariable>` di un elemento della raccolta `<environmentVariables>`. Le variabili di ambiente impostate in questa sezione hanno la precedenza sulle variabili di ambiente di sistema.

Nell'esempio seguente vengono impostate due variabili di ambiente in `web.config` . `ASPNETCORE_ENVIRONMENT` configura l'ambiente dell'app su `Development`. Uno sviluppatore può impostare temporaneamente questo valore nel `web.config` file per forzare il caricamento della [pagina delle eccezioni](xref:fundamentals/error-handling) per gli sviluppatori durante il debug di un'eccezione dell'app. `CONFIG_DIR` è un esempio di variabile di ambiente definita dall'utente, in cui lo sviluppatore ha scritto il codice che legge il valore all'avvio in modo da formare un percorso per il caricamento del file di configurazione dell'app.

```xml
<aspNetCore processPath="dotnet"
      arguments=".\MyApp.dll"
      stdoutLogEnabled="false"
      stdoutLogFile=".\logs\stdout"
      hostingModel="inprocess">
  <environmentVariables>
    <environmentVariable name="ASPNETCORE_ENVIRONMENT" value="Development" />
    <environmentVariable name="CONFIG_DIR" value="f:\application_config" />
  </environmentVariables>
</aspNetCore>
```

> [!NOTE]
> Un'alternativa all'impostazione dell'ambiente direttamente in `web.config` consiste nell'includere la `<EnvironmentName>` proprietà nel file di progetto o nel [profilo di pubblicazione `.pubxml` ](xref:host-and-deploy/visual-studio-publish-profiles) . Questo approccio imposta l'ambiente in `web.config` quando viene pubblicato il progetto:
>
> ```xml
> <PropertyGroup>
>   <EnvironmentName>Development</EnvironmentName>
> </PropertyGroup>
> ```

> [!WARNING]
> Impostare la variabile di ambiente `ASPNETCORE_ENVIRONMENT` su `Development` solo in server di gestione temporanea e test che non sono accessibili da reti non attendibili, ad esempio Internet.

## `app_offline.htm`

Se viene rilevato un file con il nome `app_offline.htm` nella directory radice di un'app, il modulo ASP.NET Core tenta di arrestare normalmente l'app e di arrestare l'elaborazione delle richieste in ingresso. Se l'app è ancora in esecuzione dopo il numero di secondi definito in `shutdownTimeLimit`, il modulo ASP.NET Core termina il processo in esecuzione.

Mentre il `app_offline.htm` file è presente, il modulo ASP.NET Core risponde alle richieste inviando di nuovo il contenuto del `app_offline.htm` file. Quando il `app_offline.htm` file viene rimosso, la richiesta successiva avvia l'app.

Quando si usa il modello di hosting out-of-process, l'app potrebbe non arrestarsi immediatamente se una connessione è aperta. Ad esempio, una connessione WebSocket potrebbe ritardare l'arresto dell'app.

## <a name="start-up-error-page"></a>Pagina di errore di avvio

Sia l'hosting In-Process che quello out-of-process producono pagine di errore personalizzate quando non riescono ad avviare l'app.

Se il modulo ASP.NET Core non riesce a trovare il gestore richieste In-Process o out-of-process, viene visualizzata una tabella codici di stato *500.0 - Errore di caricamento del gestore In-Process/out-of-process*.

Per l'hosting In-Process, se il modulo ASP.NET Core non riesce ad avviare l'app, viene visualizzata una tabella codici di stato *500.30 - Errore di avvio*.

Per l'hosting out-of-process, se il modulo ASP.NET Core non riesce ad avviare il processo di back-end o il processo di back-end viene avviato ma non riesce a restare in ascolto sulla porta configurata, verrà visualizzata la tabella codici di stato *502.5 - Errore del processo*.

Per non visualizzare questa pagina e tornare alla tabella codici di stato 5xx predefinita di IIS, usare l'attributo `disableStartUpErrorPage`. Per altre informazioni sulla configurazione di messaggi di errore personalizzati, vedere [Errori HTTP `<httpErrors>`](/iis/configuration/system.webServer/httpErrors/).

## <a name="log-creation-and-redirection"></a>Creazione e reindirizzamento dei log

Il modulo ASP.NET Core reindirizza su disco l'output della console stdout e stderr se sono impostati gli attributi `stdoutLogEnabled` e `stdoutLogFile` dell'elemento `aspNetCore`. Le eventuali le cartelle nel percorso `stdoutLogFile` vengono create dal modulo quando viene creato il file di log. Il pool di app deve avere accesso in scrittura alla posizione in cui vengono scritti i log (usare `IIS AppPool\<app_pool_name>` per specificare l'autorizzazione di scrittura).

I log non vengono ruotati, a meno che non si verifichi il riciclo/riavvio del processo. È responsabilità del provider di servizi di hosting limitare lo spazio su disco usato dai log.

L'uso del log stdout è consigliato solo per la risoluzione dei problemi di avvio dell'app durante l'hosting in IIS o quando si usa il [supporto in fase di sviluppo per IIS con Visual Studio](xref:host-and-deploy/iis/development-time-iis-support), non durante il debug in locale e l'esecuzione dell'app con IIS Express.

Non usare il log stdout per scopi di registrazione generale delle app. Per la registrazione di routine in un'app ASP.NET Core, usare una libreria di registrazione che limita le dimensioni dei file di log e ne esegue la rotazione. Per altre informazioni, vedere [Provider di registrazione di terze parti](xref:fundamentals/logging/index#third-party-logging-providers).

Un timestamp e l'estensione del file vengono aggiunti automaticamente al momento della creazione del file di log. Il nome del file di log è costituito dall'aggiunta del timestamp, dell'ID del processo e dell'estensione di file ( `.log` ) all'ultimo segmento del `stdoutLogFile` percorso, in genere `stdout` delimitato da caratteri di sottolineatura. Se il `stdoutLogFile` percorso termina con `stdout` , il nome file di un log per un'app con PID 1934 creato in 2/5/2018 a 19:42:32 è `stdout_20180205194132_1934.log` .

Se `stdoutLogEnabled` è false, gli errori che si verificano all'avvio dell'app vengono acquisiti ed emessi nel log eventi fino a 30 KB. Dopo l'avvio, tutti i log aggiuntivi vengono rimossi.

Nell'elemento di esempio seguente viene `aspNetCore` configurata la registrazione di stdout nel percorso relativo `.\log\` . Verificare che l'identità dell'utente AppPool disponga dell'autorizzazione di scrittura per il percorso specificato.

```xml
<aspNetCore processPath="dotnet"
    arguments=".\MyApp.dll"
    stdoutLogEnabled="true"
    stdoutLogFile=".\logs\stdout"
    hostingModel="inprocess">
</aspNetCore>
```

Quando si pubblica un'app per la distribuzione di app Azure Service, il valore viene impostato da Web SDK `stdoutLogFile` `\\?\%home%\LogFiles\stdout` . La `%home` variabile di ambiente è predefinita per le app ospitate dal servizio app Azure.

Per creare regole di filtro di registrazione, vedere le sezioni [configurazione](xref:fundamentals/logging/index#log-filtering) e [filtro dei log](xref:fundamentals/logging/index#log-filtering) della documentazione relativa alla registrazione del ASP.NET Core.

Per ulteriori informazioni sui formati di percorso, vedere [formati di percorso dei file nei sistemi Windows](/dotnet/standard/io/file-path-formats).

## <a name="enhanced-diagnostic-logs"></a>Log di diagnostica avanzati

Il modulo ASP.NET Core può essere configurato per restituire log di diagnostica avanzata. Aggiungere l' `<handlerSettings>` elemento all' `<aspNetCore>` elemento in `web.config` . L'impostazione di `debugLevel` su `TRACE` restituisce una maggior fedeltà dei dati diagnostici:

```xml
<aspNetCore processPath="dotnet"
    arguments=".\MyApp.dll"
    stdoutLogEnabled="false"
    stdoutLogFile="\\?\%home%\LogFiles\stdout"
    hostingModel="inprocess">
  <handlerSettings>
    <handlerSetting name="debugFile" value=".\logs\aspnetcore-debug.log" />
    <handlerSetting name="debugLevel" value="FILE,TRACE" />
  </handlerSettings>
</aspNetCore>
```

Tutte le cartelle nel percorso ( `logs` nell'esempio precedente) vengono create dal modulo quando viene creato il file di log. Il pool di applicazioni deve avere accesso in scrittura al percorso in cui vengono scritti i log (usare `IIS AppPool\{APP POOL NAME}` , dove il segnaposto `{APP POOL NAME}` è il nome del pool di applicazioni, per fornire l'autorizzazione di scrittura).

I valori di livello debug (`debugLevel`) possono includere sia il livello che il percorso.

Livelli (in ordine dal meno dettagliato al più dettagliato):

* ERRORE
* WARNING
* INFO
* TRACE

Posizioni (sono consentite più posizioni):

* CONSOLE
* EVENTLOG
* FILE

Le impostazioni del gestore possono essere specificate anche tramite le variabili di ambiente:

* `ASPNETCORE_MODULE_DEBUG_FILE`: Percorso del file di log di debug. (Impostazione predefinita: `aspnetcore-debug.log` )
* `ASPNETCORE_MODULE_DEBUG`: Impostazione del livello di debug.

> [!WARNING]
> **Non** lasciare la registrazione del debug abilitata nella distribuzione per un tempo superiore a quello necessario alla risoluzione di problema. Le dimensioni del log non sono limitate. Se si lascia abilitato il log di debug, lo spazio disponibile su disco può esaurirsi e il server o il servizio app può registrare un arresto anomalo.

Per un esempio dell'elemento nel file, vedere la pagina relativa alla [configurazione con web.config](#configuration-with-webconfig) `aspNetCore` `web.config` .

## <a name="modify-the-stack-size"></a>Modificare le dimensioni dello stack

*Si applica solo quando si usa il modello di hosting in-process.*

Configurare la dimensione dello stack gestito usando l' `stackSize` impostazione in byte in `web.config` . Le dimensioni predefinite sono pari a 1.048.576 byte (1 MB).

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

Il programma di installazione del modulo ASP.NET Core viene eseguito con i privilegi dell'account **TrustedInstaller**. Poiché l'account di sistema locale non dispone dell'autorizzazione di modifica per il percorso di condivisione utilizzato dalla configurazione condivisa di IIS, il programma di installazione genera un errore di accesso negato quando si tenta di configurare le impostazioni del modulo nel `applicationHost.config` file nella condivisione.

Quando si usa una configurazione condivisa di IIS nello stesso computer dell'installazione di IIS, eseguire il programma di installazione del bundle di hosting ASP.NET Core con il parametro `OPT_NO_SHARED_CONFIG_CHECK` impostato su `1`:

```console
dotnet-hosting-{VERSION}.exe OPT_NO_SHARED_CONFIG_CHECK=1
```

Quando il percorso della configurazione condivisa non è nello stesso computer dell'installazione di IIS, seguire questa procedura:

1. Disabilitare la configurazione condivisa di IIS.
1. Eseguire il programma di installazione.
1. Esportare il file aggiornato nella `applicationHost.config` condivisione.
1. Abilitare di nuovo la configurazione condivisa di IIS.

## <a name="module-version-and-hosting-bundle-installer-logs"></a>Versione del modulo e log del programma di installazione del bundle di hosting

Per determinare la versione del modulo ASP.NET Core installato:

1. Nel sistema di hosting passare a `%windir%\System32\inetsrv` .
1. Individuare il `aspnetcore.dll` file.
1. Fare clic con il pulsante destro del mouse sul file e scegliere **Proprietà** dal menu di scelta rapida.
1. Selezionare la scheda **Dettagli** . La versione del **file** e la **versione del prodotto** rappresentano la versione installata del modulo.

I log del programma di installazione del bundle di hosting per il modulo sono disponibili in `C:\Users\%UserName%\AppData\Local\Temp` . Il file è denominato `dd_DotNetCoreWinSvrHosting__{TIMESTAMP}_000_AspNetCoreModule_x64.log`.

## <a name="module-schema-and-configuration-file-locations"></a>Percorsi dei file di modulo, schema e configurazione

### <a name="module"></a>Modulo

**IIS (x86/amd64):**

* `%windir%\System32\inetsrv\aspnetcore.dll`

* `%windir%\SysWOW64\inetsrv\aspnetcore.dll`

* `%ProgramFiles%\IIS\Asp.Net Core Module\V2\aspnetcorev2.dll`

* `%ProgramFiles(x86)%\IIS\Asp.Net Core Module\V2\aspnetcorev2.dll`

**IIS Express (x86/amd64):**

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

::: moniker-end

::: moniker range="= aspnetcore-2.2"

Il modulo ASP.NET Core è un modulo IIS nativo collegato alla pipeline di IIS per eseguire le operazioni seguenti:

* Ospitare un app ASP.NET Core all'interno del processo di lavoro IIS (`w3wp.exe`), denominato [modello di hosting in-process](#in-process-hosting-model).
* Inoltrare le richieste Web all'app back-end ASP.NET Core che esegue il [server Kestrel](xref:fundamentals/servers/kestrel), denominato [modello di hosting out-of-process](#out-of-process-hosting-model).

Versioni supportate di Windows:

* Windows 7 o versione successiva
* Windows Server 2008 R2 o versione successiva

In caso di hosting in-process, il modulo usa un'implementazione di server in-process per IIS detta server HTTP di IIS (`IISHttpServer`).

In caso di hosting out-of-process, il modulo funziona solo con Kestrel. Il modulo non funziona con [HTTP.sys](xref:fundamentals/servers/httpsys).

## <a name="hosting-models"></a>Modelli di hosting

### <a name="in-process-hosting-model"></a>Modello di hosting in-process

Per configurare un'app per l'hosting in-process, aggiungere la proprietà `<AspNetCoreHostingModel>` al file di progetto dell'app usando il valore `InProcess` (l'hosting out-of-process viene impostato con `OutOfProcess`):

```xml
<PropertyGroup>
  <AspNetCoreHostingModel>InProcess</AspNetCoreHostingModel>
</PropertyGroup>
```

Il modello di hosting In-Process non è supportato per le app ASP.NET Core destinate a .NET Framework.

Il valore di non `<AspNetCoreHostingModel>` fa distinzione tra maiuscole e minuscole, pertanto `inprocess` e `outofprocess` sono valori validi.

Se la proprietà `<AspNetCoreHostingModel>` non è presente nel file, il valore predefinito è `OutOfProcess`.

In caso di hosting in-process, vengono applicate le caratteristiche seguenti:

* Viene usato un server HTTP di IIS (`IISHttpServer`) al posto del server [Kestrel](xref:fundamentals/servers/kestrel). Per in-process, [CreateDefaultBuilder](xref:fundamentals/host/web-host#set-up-a-host) chiama <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderIISExtensions.UseIIS*> per:

  * Registrare `IISHttpServer`.
  * Configurare la porta e il percorso di base su cui il server deve eseguire l'ascolto in caso di esecuzione dietro il modulo ASP.NET Core.
  * Configurare l'host per l'acquisizione degli errori di avvio.

* L'[attributo requestTimeout ](#attributes-of-the-aspnetcore-element) non viene applicato all'hosting in-process.

* Non è supportata la condivisione di un pool di app tra le app. Usare un pool di app per ogni app.

* Quando si usa la [Distribuzione Web ](/iis/publish/using-web-deploy/introduction-to-web-deploy) o si inserisce manualmente un [file app_offline.htm nella distribuzione](xref:host-and-deploy/iis/index#locked-deployment-files), l'app potrebbe non arrestarsi immediatamente se una connessione è aperta. Ad esempio, una connessione WebSocket può ritardare l'arresto dell'app.

* L'architettura, vale a dire il numero di bit dell'app, e il runtime installato (x64 o x86) devono corrispondere all'architettura del pool di app.

* Le disconnessioni del client vengono rilevate. Il token di annullamento [HttpContext.RequestAborted](xref:Microsoft.AspNetCore.Http.HttpContext.RequestAborted*) viene cancellato quando il client si disconnette.

* In ASP.NET Core 2.2.1 o versioni precedenti, <xref:System.IO.Directory.GetCurrentDirectory*> restituisce la directory di lavoro del processo avviato da IIS invece che la directory dell'app (ad esempio, *C:\Windows\System32\inetsrv* per *w3wp.exe*).

  Per vedere il codice di esempio che imposta la directory corrente dell'app, vedere la [classe CurrentDirectoryHelpers](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/host-and-deploy/aspnet-core-module/samples_snapshot/2.x/CurrentDirectoryHelpers.cs). Chiamare il metodo `SetCurrentDirectory` . Le chiamate successive a <xref:System.IO.Directory.GetCurrentDirectory*> specificano la directory dell'app.

* In caso di hosting in-process, <xref:Microsoft.AspNetCore.Authentication.AuthenticationService.AuthenticateAsync*> non viene chiamato internamente per inizializzare un utente. Pertanto, un'implementazione di <xref:Microsoft.AspNetCore.Authentication.IClaimsTransformation> usate per trasformare le attestazioni dopo ogni autenticazione non viene attivata per impostazione predefinita. Quando si trasformano le attestazioni con un'implementazione di <xref:Microsoft.AspNetCore.Authentication.IClaimsTransformation>, chiamare <xref:Microsoft.Extensions.DependencyInjection.AuthenticationServiceCollectionExtensions.AddAuthentication*> per aggiungere i servizi di autenticazione:

  ```csharp
  public void ConfigureServices(IServiceCollection services)
  {
      services.AddTransient<IClaimsTransformation, ClaimsTransformer>();
      services.AddAuthentication(IISServerDefaults.AuthenticationScheme);
  }

  public void Configure(IApplicationBuilder app)
  {
      app.UseAuthentication();
  }
  ```

### <a name="out-of-process-hosting-model"></a>Modello di hosting out-of-process

Per configurare un'app per l'hosting out-of-process, usare uno dei due approcci seguenti nel file di progetto:

* Non specificare la proprietà `<AspNetCoreHostingModel>`. Se la proprietà `<AspNetCoreHostingModel>` non è presente nel file, il valore predefinito è `OutOfProcess`.
* Impostare il valore della proprietà `<AspNetCoreHostingModel>` su `OutOfProcess` (l'hosting in-process è impostato con `InProcess`):

```xml
<PropertyGroup>
  <AspNetCoreHostingModel>OutOfProcess</AspNetCoreHostingModel>
</PropertyGroup>
```

Il valore non fa distinzione tra maiuscole e minuscole, pertanto `inprocess` e `outofprocess` sono valori validi.

Viene usato il server [Kestrel](xref:fundamentals/servers/kestrel) al posto di un server HTTP di IIS (`IISHttpServer`).

Per out-of-process, [CreateDefaultBuilder](xref:fundamentals/host/web-host#set-up-a-host) chiama <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderIISExtensions.UseIISIntegration*> per:

* Configurare la porta e il percorso di base su cui il server deve eseguire l'ascolto in caso di esecuzione dietro il modulo ASP.NET Core.
* Configurare l'host per l'acquisizione degli errori di avvio.

### <a name="hosting-model-changes"></a>Modifiche del modello di hosting

Se l'impostazione `hostingModel` viene modificata nel file *web.config* (descritto nella sezione [Configurazione con web.config](#configuration-with-webconfig)), il modulo ricicla il processo di lavoro per IIS.

Per IIS Express, il modulo non ricicla il processo di lavoro. Attiva invece un arresto normale del processo di IIS Express corrente. La richiesta successiva all'app attiva un nuovo processo di IIS Express.

### <a name="process-name"></a>Nome del processo

`Process.GetCurrentProcess().ProcessName` dichiara `w3wp`/`iisexpress` (In-Process) o `dotnet` (out-of-process).

Molti moduli nativi, ad esempio l'autenticazione di Windows, rimangono attivi. Per altre informazioni sui moduli IIS attivi con il modulo ASP.NET Core, vedere <xref:host-and-deploy/iis/modules>.

Il modulo ASP.NET Core può anche:

* Impostare variabili di ambiente per il processo di lavoro.
* Registrare output stdout in una risorsa di archiviazione di file per la risoluzione di problemi di avvio.
* Inoltrare token di autenticazione di Windows.

## <a name="how-to-install-and-use-the-aspnet-core-module"></a>Come installare e usare il modulo ASP.NET Core

Per istruzioni su come installare e usare il modulo ASP.NET Core, vedere [Installare il bundle di hosting .NET Core](xref:host-and-deploy/iis/index#install-the-net-core-hosting-bundle).

## <a name="configuration-with-webconfig"></a>Configurazione con web.config

Il modulo ASP.NET Core viene configurato tramite la sezione `aspNetCore` del nodo `system.webServer` nel file *web.config* del sito.

Il file *web.config* seguente viene pubblicato per una [distribuzione dipendente dal framework](/dotnet/articles/core/deploying/#framework-dependent-deployments-fdd) e configura il modulo ASP.NET Core per gestire le richieste del sito:

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <location path="." inheritInChildApplications="false">
    <system.webServer>
      <handlers>
        <add name="aspNetCore" path="*" verb="*" modules="AspNetCoreModuleV2" resourceType="Unspecified" />
      </handlers>
      <aspNetCore processPath="dotnet"
                  arguments=".\MyApp.dll"
                  stdoutLogEnabled="false"
                  stdoutLogFile=".\logs\stdout"
                  hostingModel="inprocess" />
    </system.webServer>
  </location>
</configuration>
```

Il file *web.config* seguente viene pubblicato per una [distribuzione autonoma](/dotnet/articles/core/deploying/#self-contained-deployments-scd):

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <location path="." inheritInChildApplications="false">
    <system.webServer>
      <handlers>
        <add name="aspNetCore" path="*" verb="*" modules="AspNetCoreModuleV2" resourceType="Unspecified" />
      </handlers>
      <aspNetCore processPath=".\MyApp.exe"
                  stdoutLogEnabled="false"
                  stdoutLogFile=".\logs\stdout"
                  hostingModel="inprocess" />
    </system.webServer>
  </location>
</configuration>
```

La <xref:System.Configuration.SectionInformation.InheritInChildApplications*> proprietà è impostata su `false` per indicare che le impostazioni specificate all'interno dell' [\<location>](/iis/manage/managing-your-configuration-settings/understanding-iis-configuration-delegation#the-concept-of-location) elemento non vengono ereditate da app che si trovano in una sottodirectory dell'app.

Quando un'app viene distribuita in [Servizio app di Azure](https://azure.microsoft.com/services/app-service/), il percorso `stdoutLogFile` è impostato su `\\?\%home%\LogFiles\stdout`. Il percorso salva i log stdout nella cartella *LogFiles*, ovvero una posizione creata automaticamente dal servizio.

Per informazioni sulla configurazione delle applicazioni secondarie IIS, vedere <xref:host-and-deploy/iis/index#sub-applications>.

### <a name="attributes-of-the-aspnetcore-element"></a>Attributi dell'elemento aspNetCore

| Attributo | Descrizione | Predefinito |
| --------- | ----------- | :-----: |
| `arguments` | <p>Attributo stringa facoltativo.</p><p>Argomenti per il file eseguibile specificato in `processPath` .</p> | |
| `disableStartUpErrorPage` | <p>Attributo booleano facoltativo.</p><p>Se true, la pagina **502.5 - Errore del processo** non viene visualizzata e la tabella codici di stato 502 configurata in *web.config* ha la precedenza.</p> | `false` |
| `forwardWindowsAuthToken` | <p>Attributo booleano facoltativo.</p><p>Se true, il token viene inoltrato al processo figlio in ascolto su %ASPNETCORE_PORT% come un'intestazione 'MS-ASPNETCORE-WINAUTHTOKEN' per ogni richiesta. È responsabilità del processo chiamare CloseHandle su questo token per ogni richiesta.</p> | `true` |
| `hostingModel` | <p>Attributo stringa facoltativo.</p><p>Specifica il modello di hosting come in-process ( `InProcess` / `inprocess` ) o out-of-process ( `OutOfProcess` / `outofprocess` ).</p> | `OutOfProcess`<br>`outofprocess` |
| `processesPerApplication` | <p>Attributo Integer facoltativo.</p><p>Specifica il numero di istanze del processo specificato nell' `processPath` impostazione che è possibile creare per ogni app.</p><p>&dagger;Per l'hosting in-process, il valore è limitato a `1`.</p><p>L'impostazione di `processesPerApplication` è sconsigliata. Questo attributo sarà rimosso nelle versioni future.</p> | Valore predefinito: `1`<br>Min: `1`<br>Max: `100`&dagger; |
| `processPath` | <p>Attributo stringa obbligatorio.</p><p>Percorso del file eseguibile che avvia un processo in ascolto delle richieste HTTP. I percorsi relativi sono supportati. Se il percorso inizia con `.`, viene considerato relativo alla radice del sito.</p> | |
| `rapidFailsPerMinute` | <p>Attributo Integer facoltativo.</p><p>Specifica il numero di volte in cui il processo specificato in `processPath` può arrestarsi in modo anomalo al minuto. Se questo limite viene superato, il modulo smette di avviare il processo per la parte restante del minuto.</p><p>Non supportato con l'hosting in-process.</p> | Valore predefinito: `10`<br>Min: `0`<br>Max: `100` |
| `requestTimeout` | <p>Attributo Timespan facoltativo.</p><p>Specifica la durata per cui il modulo ASP.NET Core attende una risposta dal processo in ascolto su %ASPNETCORE_PORT%.</p><p>Nelle versioni del modulo ASP.NET Core fornito con ASP.NET Core 2.1 o versioni successive, `requestTimeout` viene specificato in ore, minuti e secondi.</p><p>Non è applicabile all'hosting in-process. Per l'hosting in-process, il modulo resta in attesa che l'app elabori la richiesta.</p><p>I valori validi per i segmenti della stringa relativi a minuti e secondi sono compresi nell'intervallo tra 0 e 59. Se si usa **60** come valore per i minuti o i secondi, viene generato un errore *500 - Errore interno del server*.</p> | Valore predefinito: `00:02:00`<br>Min: `00:00:00`<br>Max: `360:00:00` |
| `shutdownTimeLimit` | <p>Attributo Integer facoltativo.</p><p>Durata in secondi durante la quale il modulo attende che l'eseguibile venga arrestato normalmente quando `app_offline.htm` viene rilevato il file.</p> | Valore predefinito: `10`<br>Min: `0`<br>Max: `600` |
| `startupTimeLimit` | <p>Attributo Integer facoltativo.</p><p>Durata in secondi per cui il modulo attende l'avvio di un processo in ascolto sulla porta da parte del file eseguibile. Se questo limite di tempo viene superato, il modulo termina il processo.</p><p>Quando si esegue l'hosting *in-process*: il processo **non** viene riavviato **e non usa** l' `rapidFailsPerMinute` impostazione.</p><p>Quando si esegue l'hosting *out-of-process*: il modulo tenta di riavviare il processo quando riceve una nuova richiesta e continua a provare a riavviare il processo per le successive richieste in ingresso, a meno che l'app non riesca ad avviare il `rapidFailsPerMinute` numero di volte nell'ultimo minuto in sequenza.</p><p>Un valore pari a 0 (zero) **non** è considerato un timeout infinito.</p> | Valore predefinito: `120`<br>Min: `0`<br>Max: `3600` |
| `stdoutLogEnabled` | <p>Attributo booleano facoltativo.</p><p>Se true, **stdout** e **stderr** per il processo specificato in `processPath` vengono reindirizzati al file specificato in **stdoutLogFile**.</p> | `false` |
| `stdoutLogFile` | <p>Attributo stringa facoltativo.</p><p>Specifica il percorso del file relativo o assoluto per `stdout` cui `stderr` vengono registrati e dal processo specificato in `processPath` . I percorsi relativi sono relativi alla radice del sito. Qualsiasi percorso che inizia con `.` è relativo al sito radice e tutti gli altri percorsi vengono trattati come percorsi assoluti. Le eventuali cartelle specificate nel percorso vengono create dal modulo quando viene creato il file di log. L'utilizzo di delimitatori di sottolineatura, timestamp, ID processo ed estensione di file ( `.log` ) viene aggiunto all'ultimo segmento del `stdoutLogFile` percorso. Se `.\logs\stdout` viene fornito come valore, un esempio di log stdout viene salvato come `stdout_20180205194132_1934.log` nella `logs` cartella quando viene salvato il 2/5/2018 alle 19:41:32 con l'ID di processo 1934.</p> | `aspnetcore-stdout` |

### <a name="setting-environment-variables"></a>Impostazioni delle variabili di ambiente

È possibile specificare le variabili di ambiente per il processo nell'attributo `processPath`. Specificare una variabile di ambiente con l'elemento figlio `<environmentVariable>` di un elemento della raccolta `<environmentVariables>`. Le variabili di ambiente impostate in questa sezione hanno la precedenza sulle variabili di ambiente di sistema.

Nell'esempio seguente vengono impostate due variabili di ambiente. `ASPNETCORE_ENVIRONMENT` configura l'ambiente dell'app su `Development`. Uno sviluppatore può impostare temporaneamente questo valore nel `web.config` file per forzare il caricamento della [pagina delle eccezioni](xref:fundamentals/error-handling) per gli sviluppatori durante il debug di un'eccezione dell'app. `CONFIG_DIR` è un esempio di variabile di ambiente definita dall'utente, in cui lo sviluppatore ha scritto il codice che legge il valore all'avvio in modo da formare un percorso per il caricamento del file di configurazione dell'app.

```xml
<aspNetCore processPath="dotnet"
      arguments=".\MyApp.dll"
      stdoutLogEnabled="false"
      stdoutLogFile=".\logs\stdout"
      hostingModel="inprocess">
  <environmentVariables>
    <environmentVariable name="ASPNETCORE_ENVIRONMENT" value="Development" />
    <environmentVariable name="CONFIG_DIR" value="f:\application_config" />
  </environmentVariables>
</aspNetCore>
```

> [!NOTE]
> Un'alternativa all'impostazione dell'ambiente direttamente in `web.config` consiste nell'includere la `<EnvironmentName>` proprietà nel [profilo di pubblicazione (con estensione pubxml)](xref:host-and-deploy/visual-studio-publish-profiles) o nel file di progetto. Questo approccio imposta l'ambiente in `web.config` quando viene pubblicato il progetto:
>
> ```xml
> <PropertyGroup>
>   <EnvironmentName>Development</EnvironmentName>
> </PropertyGroup>
> ```

> [!WARNING]
> Impostare la variabile di ambiente `ASPNETCORE_ENVIRONMENT` su `Development` solo in server di gestione temporanea e test che non sono accessibili da reti non attendibili, ad esempio Internet.

## <a name="app_offlinehtm"></a>app_offline.htm

Se viene rilevato un file con il nome `app_offline.htm` nella directory radice di un'app, il modulo ASP.NET Core tenta di arrestare normalmente l'app e di arrestare l'elaborazione delle richieste in ingresso. Se l'app è ancora in esecuzione dopo il numero di secondi definito in `shutdownTimeLimit`, il modulo ASP.NET Core termina il processo in esecuzione.

Mentre il `app_offline.htm` file è presente, il modulo ASP.NET Core risponde alle richieste inviando di nuovo il contenuto del `app_offline.htm` file. Quando il `app_offline.htm` file viene rimosso, la richiesta successiva avvia l'app.

Quando si usa il modello di hosting out-of-process, l'app potrebbe non arrestarsi immediatamente se una connessione è aperta. Ad esempio, una connessione WebSocket può ritardare l'arresto dell'app.

## <a name="start-up-error-page"></a>Pagina di errore di avvio

Sia l'hosting In-Process che quello out-of-process producono pagine di errore personalizzate quando non riescono ad avviare l'app.

Se il modulo ASP.NET Core non riesce a trovare il gestore richieste In-Process o out-of-process, viene visualizzata una tabella codici di stato *500.0 - Errore di caricamento del gestore In-Process/out-of-process*.

Per l'hosting In-Process, se il modulo ASP.NET Core non riesce ad avviare l'app, viene visualizzata una tabella codici di stato *500.30 - Errore di avvio*.

Per l'hosting out-of-process, se il modulo ASP.NET Core non riesce ad avviare il processo di back-end o il processo di back-end viene avviato ma non riesce a restare in ascolto sulla porta configurata, verrà visualizzata la tabella codici di stato *502.5 - Errore del processo*.

Per non visualizzare questa pagina e tornare alla tabella codici di stato 5xx predefinita di IIS, usare l'attributo `disableStartUpErrorPage`. Per altre informazioni sulla configurazione di messaggi di errore personalizzati, vedere [Errori HTTP \<httpErrors>](/iis/configuration/system.webServer/httpErrors/).

## <a name="log-creation-and-redirection"></a>Creazione e reindirizzamento dei log

Il modulo ASP.NET Core reindirizza su disco l'output della console stdout e stderr se sono impostati gli attributi `stdoutLogEnabled` e `stdoutLogFile` dell'elemento `aspNetCore`. Le eventuali le cartelle nel percorso `stdoutLogFile` vengono create dal modulo quando viene creato il file di log. Il pool di applicazioni deve avere accesso in scrittura al percorso in cui vengono scritti i log (usare `IIS AppPool\{APP POOL NAME}` per fornire l'autorizzazione di scrittura, dove il segnaposto `{APP POOL NAME}` è il nome del pool di applicazioni).

I log non vengono ruotati, a meno che non si verifichi il riciclo/riavvio del processo. È responsabilità del provider di servizi di hosting limitare lo spazio su disco usato dai log.

L'uso del log stdout è consigliato solo per la risoluzione dei problemi di avvio dell'app durante l'hosting in IIS o quando si usa il [supporto in fase di sviluppo per IIS con Visual Studio](xref:host-and-deploy/iis/development-time-iis-support), non durante il debug in locale e l'esecuzione dell'app con IIS Express.

Non usare il log stdout per scopi di registrazione generale delle app. Per la registrazione di routine in un'app ASP.NET Core, usare una libreria di registrazione che limita le dimensioni dei file di log e ne esegue la rotazione. Per altre informazioni, vedere [Provider di registrazione di terze parti](xref:fundamentals/logging/index#third-party-logging-providers).

Un timestamp e l'estensione del file vengono aggiunti automaticamente al momento della creazione del file di log. Il nome del file di log è costituito dall'aggiunta del timestamp, dell'ID del processo e dell'estensione di file ( `.log` ) all'ultimo segmento del `stdoutLogFile` percorso, in genere `stdout` delimitato da caratteri di sottolineatura. Se il `stdoutLogFile` percorso termina con `stdout` , il nome file di un log per un'app con PID 1934 creato in 2/5/2018 a 19:42:32 è `stdout_20180205194132_1934.log` .

Se `stdoutLogEnabled` è false, gli errori che si verificano all'avvio dell'app vengono acquisiti ed emessi nel log eventi fino a 30 KB. Dopo l'avvio, tutti i log aggiuntivi vengono rimossi.

Nell'elemento di esempio seguente viene `aspNetCore` configurata la registrazione di stdout nel percorso relativo `.\log\` . Verificare che l'identità utente del pool di applicazioni disponga delle autorizzazioni necessarie per scrivere nel percorso specificato.

```xml
<aspNetCore processPath="dotnet"
    arguments=".\MyApp.dll"
    stdoutLogEnabled="true"
    stdoutLogFile=".\logs\stdout"
    hostingModel="inprocess">
</aspNetCore>
```

Quando si pubblica un'app per la distribuzione di app Azure Service, il valore viene impostato da Web SDK `stdoutLogFile` `\\?\%home%\LogFiles\stdout` . La `%home` variabile di ambiente è predefinita per le app ospitate dal servizio app Azure.

Per ulteriori informazioni sui formati di percorso, vedere [formati di percorso dei file nei sistemi Windows](/dotnet/standard/io/file-path-formats).

## <a name="enhanced-diagnostic-logs"></a>Log di diagnostica avanzati

Il modulo ASP.NET Core può essere configurato per restituire log di diagnostica avanzata. Aggiungere l' `<handlerSettings>` elemento all' `<aspNetCore>` elemento in `web.config` . L'impostazione di `debugLevel` su `TRACE` restituisce una maggior fedeltà dei dati diagnostici:

```xml
<aspNetCore processPath="dotnet"
    arguments=".\MyApp.dll"
    stdoutLogEnabled="false"
    stdoutLogFile="\\?\%home%\LogFiles\stdout"
    hostingModel="inprocess">
  <handlerSettings>
    <handlerSetting name="debugFile" value=".\logs\aspnetcore-debug.log" />
    <handlerSetting name="debugLevel" value="FILE,TRACE" />
  </handlerSettings>
</aspNetCore>
```

Le cartelle nel percorso specificato per il `<handlerSetting>` valore ( `logs` nell'esempio precedente) non vengono create automaticamente dal modulo e dovrebbero essere già presenti nella distribuzione. Il pool di applicazioni deve avere accesso in scrittura al percorso in cui vengono scritti i log (usare `IIS AppPool\{APP POOL NAME}` per fornire l'autorizzazione di scrittura, dove il segnaposto `{APP POOL NAME}` è il nome del pool di applicazioni).

I valori di livello debug (`debugLevel`) possono includere sia il livello che il percorso.

Livelli (in ordine dal meno dettagliato al più dettagliato):

* ERRORE
* WARNING
* INFO
* TRACE

Posizioni (sono consentite più posizioni):

* CONSOLE
* EVENTLOG
* FILE

Le impostazioni del gestore possono essere specificate anche tramite le variabili di ambiente:

* `ASPNETCORE_MODULE_DEBUG_FILE`: Percorso del file di log di debug. (Impostazione predefinita: `aspnetcore-debug.log` )
* `ASPNETCORE_MODULE_DEBUG`: Impostazione del livello di debug.

> [!WARNING]
> **Non** lasciare la registrazione del debug abilitata nella distribuzione per un tempo superiore a quello necessario alla risoluzione di problema. Le dimensioni del log non sono limitate. Se si lascia abilitato il log di debug, lo spazio disponibile su disco può esaurirsi e il server o il servizio app può registrare un arresto anomalo.

Per un esempio dell'elemento nel file, vedere la pagina relativa alla [configurazione con web.config](#configuration-with-webconfig) `aspNetCore` `web.config` .

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
1. Esportare il file aggiornato nella `applicationHost.config` condivisione.
1. Abilitare di nuovo la configurazione condivisa di IIS.

## <a name="module-version-and-hosting-bundle-installer-logs"></a>Versione del modulo e log del programma di installazione del bundle di hosting

Per determinare la versione del modulo ASP.NET Core installato:

1. Nel sistema di hosting passare a `%windir%\System32\inetsrv` .
1. Individuare il `aspnetcore.dll` file.
1. Fare clic con il pulsante destro del mouse sul file e scegliere **Proprietà** dal menu di scelta rapida.
1. Selezionare la scheda **Dettagli** . La versione del **file** e la **versione del prodotto** rappresentano la versione installata del modulo.

I log del programma di installazione del bundle di hosting per il modulo sono disponibili in `C:\\Users\\%UserName%\\AppData\\Local\\Temp` . Il nome del file è `dd_DotNetCoreWinSvrHosting__\{TIMESTAMP}_000_AspNetCoreModule_x64.log` , dove il segnaposto `{TIMESTAMP}` è il timestamp.

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

::: moniker-end

::: moniker range="< aspnetcore-2.2"

Il modulo ASP.NET Core è un modulo IIS nativo collegato alla pipeline di IIS che inoltra le richieste Web alle app back-end ASP.NET Core.

Versioni supportate di Windows:

* Windows 7 o versione successiva
* Windows Server 2008 R2 o versione successiva

Il modulo funziona solo con Kestrel. Il modulo non è compatibile con [HTTP.sys](xref:fundamentals/servers/httpsys).

Poiché le app ASP.NET Core vengono eseguite in un processo distinto dal processo di lavoro IIS, il modulo esegue anche la gestione dei processi. Il modulo avvia il processo per l'app ASP.NET Core quando arriva la prima richiesta e riavvia l'app se si verifica un arresto anomalo di questa. Si tratta essenzialmente dello stesso comportamento delle app ASP.NET 4.x eseguite In-Process in IIS e gestite dal [servizio Attivazione processo Windows (WAS, Windows Activation Service)](/iis/manage/provisioning-and-managing-iis/features-of-the-windows-process-activation-service-was).

Il diagramma seguente illustra la relazione tra IIS, il modulo ASP.NET Core e un'app:

![Modulo ASP.NET Core](iis/index/_static/ancm-outofprocess.png)

Le richieste arrivano dal Web al driver HTTP.sys in modalità kernel. Il driver instrada le richieste a IIS sulla porta configurata per il sito Web, in genere 80 (HTTP) o 443 (HTTPS). Il modulo inoltra le richieste a Kestrel su una porta casuale per l'app non corrispondente alla porta 80 o 443.

Il modulo specifica la porta tramite una variabile di ambiente all'avvio e il [middleware di integrazione di IIS](xref:host-and-deploy/iis/index#enable-the-iisintegration-components) configura il server per l'ascolto `http://localhost:{port}` . Vengono eseguiti controlli aggiuntivi e le richieste che non provengono dal modulo vengono rifiutate. Il modulo non supporta l'inoltro HTTPS, pertanto le richieste vengono inoltrate tramite HTTP anche se sono state ricevute da IIS tramite HTTPS.

Dopo che Kestrel ha prelevato la richiesta dal modulo, viene eseguito il push della richiesta nella pipeline middleware ASP.NET Core. La pipeline middleware gestisce la richiesta e la passa come istanza di `HttpContext` alla logica dell'app. Il middleware aggiunto dall'integrazione di IIS aggiorna lo schema, l'IP remoto e il percorso di base all'account per l'inoltro della richiesta a Kestrel. La risposta dell'app viene quindi passata a IIS, che ne esegue di nuovo il push al client HTTP che ha avviato la richiesta.

Molti moduli nativi, ad esempio l'autenticazione di Windows, rimangono attivi. Per altre informazioni sui moduli IIS attivi con il modulo ASP.NET Core, vedere <xref:host-and-deploy/iis/modules>.

Il modulo ASP.NET Core può anche:

* Impostare variabili di ambiente per il processo di lavoro.
* Registrare output stdout in una risorsa di archiviazione di file per la risoluzione di problemi di avvio.
* Inoltrare token di autenticazione di Windows.

## <a name="how-to-install-and-use-the-aspnet-core-module"></a>Come installare e usare il modulo ASP.NET Core

Per istruzioni su come installare e usare il modulo ASP.NET Core, vedere [Installare il bundle di hosting .NET Core](xref:host-and-deploy/iis/index#install-the-net-core-hosting-bundle).

## <a name="configuration-with-webconfig"></a>Configurazione con web.config

Il modulo ASP.NET Core viene configurato tramite la sezione `aspNetCore` del nodo `system.webServer` nel file *web.config* del sito.

Il file *web.config* seguente viene pubblicato per una [distribuzione dipendente dal framework](/dotnet/articles/core/deploying/#framework-dependent-deployments-fdd) e configura il modulo ASP.NET Core per gestire le richieste del sito:

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <system.webServer>
    <handlers>
      <add name="aspNetCore" path="*" verb="*" modules="AspNetCoreModule" resourceType="Unspecified" />
    </handlers>
    <aspNetCore processPath="dotnet"
                arguments=".\MyApp.dll"
                stdoutLogEnabled="false"
                stdoutLogFile=".\logs\stdout" />
  </system.webServer>
</configuration>
```

Il file *web.config* seguente viene pubblicato per una [distribuzione autonoma](/dotnet/articles/core/deploying/#self-contained-deployments-scd):

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <system.webServer>
    <handlers>
      <add name="aspNetCore" path="*" verb="*" modules="AspNetCoreModule" resourceType="Unspecified" />
    </handlers>
    <aspNetCore processPath=".\MyApp.exe"
                stdoutLogEnabled="false"
                stdoutLogFile=".\logs\stdout" />
  </system.webServer>
</configuration>
```

Quando un'app viene distribuita in [Servizio app di Azure](https://azure.microsoft.com/services/app-service/), il percorso `stdoutLogFile` è impostato su `\\?\%home%\LogFiles\stdout`. Il percorso salva i log stdout nella cartella *LogFiles*, ovvero una posizione creata automaticamente dal servizio.

Per informazioni sulla configurazione delle applicazioni secondarie IIS, vedere <xref:host-and-deploy/iis/index#sub-applications>.

### <a name="attributes-of-the-aspnetcore-element"></a>Attributi dell'elemento aspNetCore

| Attributo | Descrizione | Predefinito |
| --------- | ----------- | :-----: |
| `arguments` | <p>Attributo stringa facoltativo.</p><p>Argomenti per l'eseguibile specificato in **processPath**.</p>| |
| `disableStartUpErrorPage` | <p>Attributo booleano facoltativo.</p><p>Se true, la pagina **502.5 - Errore del processo** non viene visualizzata e la tabella codici di stato 502 configurata in *web.config* ha la precedenza.</p> | `false` |
| `forwardWindowsAuthToken` | <p>Attributo booleano facoltativo.</p><p>Se true, il token viene inoltrato al processo figlio in ascolto su %ASPNETCORE_PORT% come un'intestazione 'MS-ASPNETCORE-WINAUTHTOKEN' per ogni richiesta. È responsabilità del processo chiamare CloseHandle su questo token per ogni richiesta.</p> | `true` |
| `processesPerApplication` | <p>Attributo Integer facoltativo.</p><p>Specifica il numero di istanze del processo specificato nell'impostazione **processPath** che può essere riattivato per ogni app.</p><p>L'impostazione di `processesPerApplication` è sconsigliata. Questo attributo sarà rimosso nelle versioni future.</p> | Valore predefinito: `1`<br>Min: `1`<br>Max: `100` |
| `processPath` | <p>Attributo stringa obbligatorio.</p><p>Percorso del file eseguibile che avvia un processo in ascolto delle richieste HTTP. I percorsi relativi sono supportati. Se il percorso inizia con `.`, viene considerato relativo alla radice del sito.</p> | |
| `rapidFailsPerMinute` | <p>Attributo Integer facoltativo.</p><p>Specifica il numero di arresti anomali al minuto per il processo specificato in **processPath**. Se questo limite viene superato, il modulo smette di avviare il processo per la parte restante del minuto.</p> | Valore predefinito: `10`<br>Min: `0`<br>Max: `100` |
| `requestTimeout` | <p>Attributo Timespan facoltativo.</p><p>Specifica la durata per cui il modulo ASP.NET Core attende una risposta dal processo in ascolto su %ASPNETCORE_PORT%.</p><p>Nelle versioni del modulo ASP.NET Core fornito con ASP.NET Core 2.1 o versioni successive, `requestTimeout` viene specificato in ore, minuti e secondi.</p> | Valore predefinito: `00:02:00`<br>Min: `00:00:00`<br>Max: `360:00:00` |
| `shutdownTimeLimit` | <p>Attributo Integer facoltativo.</p><p>Durata in secondi per cui il modulo attende che il file eseguibile venga arrestato normalmente quando viene rilevato il file *app_offline.htm*.</p> | Valore predefinito: `10`<br>Min: `0`<br>Max: `600` |
| `startupTimeLimit` | <p>Attributo Integer facoltativo.</p><p>Durata in secondi per cui il modulo attende l'avvio di un processo in ascolto sulla porta da parte del file eseguibile. Se questo limite di tempo viene superato, il modulo termina il processo. Il modulo tenta di avviare nuovamente il processo quando riceve una nuova richiesta e continua a tentare di riavviare il processo alle successive richieste in ingresso, a meno che non risulti impossibile avviare l'app un numero di volte pari a **rapidFailsPerMinute** nell'ultimo minuto continuo.</p><p>Un valore pari a 0 (zero) **non** è considerato un timeout infinito.</p> | Valore predefinito: `120`<br>Min: `0`<br>Max: `3600` |
| `stdoutLogEnabled` | <p>Attributo booleano facoltativo.</p><p>Se true, **stdout** e **stderr** per il processo specificato in **processPath** vengono reindirizzati al file specificato in **stdoutLogFile**.</p> | `false` |
| `stdoutLogFile` | <p>Attributo stringa facoltativo.</p><p>Specifica il percorso relativo o assoluto per cui vengono registrati **stdout** e **stderr** dal processo specificato in **processPath**. I percorsi relativi sono relativi alla radice del sito. Qualsiasi percorso che inizia con `.` è relativo al sito radice e tutti gli altri percorsi vengono trattati come percorsi assoluti. Le eventuali cartelle specificate nel percorso devono essere già esistenti affinché il modulo possa creare il file di log. Usando il carattere di sottolineatura come delimitatore, il timestamp, l'ID processo e l'estensione del file (*.log*) vengono aggiunti all'ultimo segmento del percorso **stdoutLogFile**. Se si specifica `.\logs\stdout` come valore, un log stdout di esempio salvato il 5/2/2018 alle 19:41:32 con un ID processo 1934 viene salvato come *stdout_20180205194132_1934.log* nella cartella *logs*.</p> | `aspnetcore-stdout` |

### <a name="setting-environment-variables"></a>Impostazioni delle variabili di ambiente

È possibile specificare le variabili di ambiente per il processo nell'attributo `processPath`. Specificare una variabile di ambiente con l'elemento figlio `<environmentVariable>` di un elemento della raccolta `<environmentVariables>`.

> [!WARNING]
> Le variabili di ambiente impostate in questa sezione sono in conflitto con le variabili di ambiente di sistema impostate con lo stesso nome. Se una variabile di ambiente è impostata sia nel file *web.config* sia a livello di sistema in Windows, il valore del file *web.config* viene aggiunto al valore della variabile di ambiente di sistema, ad esempio `ASPNETCORE_ENVIRONMENT: Development;Development`, e ciò impedisce l'avvio dell'app.

Nell'esempio seguente vengono impostate due variabili di ambiente. `ASPNETCORE_ENVIRONMENT` configura l'ambiente dell'app su `Development`. Uno sviluppatore può impostare temporaneamente questo valore nel file *web.config* per forzare il caricamento della [pagina delle eccezioni per gli sviluppatori](xref:fundamentals/error-handling) durante il debug di un'eccezione dell'app. `CONFIG_DIR` è un esempio di variabile di ambiente definita dall'utente, in cui lo sviluppatore ha scritto il codice che legge il valore all'avvio in modo da formare un percorso per il caricamento del file di configurazione dell'app.

```xml
<aspNetCore processPath="dotnet"
      arguments=".\MyApp.dll"
      stdoutLogEnabled="false"
      stdoutLogFile="\\?\%home%\LogFiles\stdout">
  <environmentVariables>
    <environmentVariable name="ASPNETCORE_ENVIRONMENT" value="Development" />
    <environmentVariable name="CONFIG_DIR" value="f:\application_config" />
  </environmentVariables>
</aspNetCore>
```

> [!WARNING]
> Impostare la variabile di ambiente `ASPNETCORE_ENVIRONMENT` su `Development` solo in server di gestione temporanea e test che non sono accessibili da reti non attendibili, ad esempio Internet.

## <a name="app_offlinehtm"></a>app_offline.htm

Se un file denominato *app_offline.htm* viene rilevato nella directory radice di un'app, il modulo ASP.NET Core tenta di arrestare normalmente l'app e arresta l'elaborazione delle richieste in ingresso. Se l'app è ancora in esecuzione dopo il numero di secondi definito in `shutdownTimeLimit`, il modulo ASP.NET Core termina il processo in esecuzione.

Mentre il file *app_offline.htm* è presente, il modulo ASP.NET Core risponde alle richieste restituendo il contenuto del file *app_offline.htm*. Quando il file *app_offline.htm* viene rimosso, la richiesta successiva avvia l'app.

## <a name="start-up-error-page"></a>Pagina di errore di avvio

Se il modulo ASP.NET Core non riesce ad avviare il processo di back-end o il processo di back-end viene avviato ma non riesce a restare in ascolto sulla porta configurata, verrà visualizzata la tabella codici di stato *502.5 - Errore del processo*. Per non visualizzare questa pagina e tornare alla tabella codici di stato 502 predefinita di IIS, usare l'attributo `disableStartUpErrorPage`. Per altre informazioni sulla configurazione di messaggi di errore personalizzati, vedere [Errori HTTP \<httpErrors>](/iis/configuration/system.webServer/httpErrors/).

## <a name="log-creation-and-redirection"></a>Creazione e reindirizzamento dei log

Il modulo ASP.NET Core reindirizza su disco l'output della console stdout e stderr se sono impostati gli attributi `stdoutLogEnabled` e `stdoutLogFile` dell'elemento `aspNetCore`. Le eventuali le cartelle nel percorso `stdoutLogFile` vengono create dal modulo quando viene creato il file di log. Il pool di app deve avere accesso in scrittura alla posizione in cui vengono scritti i log (usare `IIS AppPool\<app_pool_name>` per specificare l'autorizzazione di scrittura).

I log non vengono ruotati, a meno che non si verifichi il riciclo/riavvio del processo. È responsabilità del provider di servizi di hosting limitare lo spazio su disco usato dai log.

L'uso del log stdout è consigliato solo per la risoluzione dei problemi di avvio dell'app durante l'hosting in IIS o quando si usa il [supporto in fase di sviluppo per IIS con Visual Studio](xref:host-and-deploy/iis/development-time-iis-support), non durante il debug in locale e l'esecuzione dell'app con IIS Express.

Non usare il log stdout per scopi di registrazione generale delle app. Per la registrazione di routine in un'app ASP.NET Core, usare una libreria di registrazione che limita le dimensioni dei file di log e ne esegue la rotazione. Per altre informazioni, vedere [Provider di registrazione di terze parti](xref:fundamentals/logging/index#third-party-logging-providers).

Un timestamp e l'estensione del file vengono aggiunti automaticamente al momento della creazione del file di log. Il nome del file di log è composto aggiungendo il timestamp, l'ID processo e l'estensione del file (*.log*) all'ultimo segmento del percorso `stdoutLogFile` (in genere *stdout*), con caratteri di sottolineatura come delimitatori. Se il percorso `stdoutLogFile` termina con *stdout*, un log per un'app con un PID 1934 creata il 5/2/2018 alle 19:42:32 sarà denominato *stdout_20180205194132_1934.log*.

Nell'elemento di esempio seguente viene `aspNetCore` configurata la registrazione di stdout nel percorso relativo `.\log\` . Verificare che l'identità dell'utente AppPool disponga dell'autorizzazione di scrittura per il percorso specificato.

```xml
<aspNetCore processPath="dotnet"
    arguments=".\MyApp.dll"
    stdoutLogEnabled="true"
    stdoutLogFile=".\logs\stdout">
</aspNetCore>
```

Quando si pubblica un'app per la distribuzione di app Azure Service, il valore viene impostato da Web SDK `stdoutLogFile` `\\?\%home%\LogFiles\stdout` . La `%home` variabile di ambiente è predefinita per le app ospitate dal servizio app Azure.

Per creare regole di filtro di registrazione, vedere le sezioni [configurazione](xref:fundamentals/logging/index#log-filtering) e [filtro dei log](xref:fundamentals/logging/index#log-filtering) della documentazione relativa alla registrazione del ASP.NET Core.

Per ulteriori informazioni sui formati di percorso, vedere [formati di percorso dei file nei sistemi Windows](/dotnet/standard/io/file-path-formats).

## <a name="proxy-configuration-uses-http-protocol-and-a-pairing-token"></a>La configurazione del proxy usa il protocollo HTTP e un token di associazione

Il proxy creato tra il modulo ASP.NET Core e Kestrel usa il protocollo HTTP. Non vi è alcun rischio di intercettazione del traffico tra il modulo e Kestrel da una posizione esterna al server.

Viene usato un token di associazione per garantire che le richieste ricevute da Kestrel siano state trasmesse tramite proxy da IIS e non provengano da un'altra origine. Il token di associazione viene creato e impostato in una variabile di ambiente (`ASPNETCORE_TOKEN`) dal modulo. Il token di associazione viene impostato anche in un'intestazione (`MS-ASPNETCORE-TOKEN`) in tutte le richieste trasmesse tramite proxy. Il middleware di IIS controlla ogni richiesta che riceve in modo da confermare che il valore dell'intestazione del token di associazione corrisponda al valore della variabile di ambiente. Se i valori del token non corrispondono, la richiesta viene registrata e rifiutata. La variabile di ambiente del token di associazione e il traffico tra il modulo e Kestrel non sono accessibili da una posizione esterna al server. Senza conoscere il valore del token di associazione, un utente malintenzionato non può inviare richieste che ignorano il controllo nel middleware di IIS.

## <a name="aspnet-core-module-with-an-iis-shared-configuration"></a>Modulo ASP.NET Core con configurazione condivisa di IIS

Il programma di installazione del modulo ASP.NET Core viene eseguito con i privilegi dell'account **TrustedInstaller**. Poiché l'account di sistema locale non dispone dell'autorizzazione di modifica per il percorso della condivisione usato dalla configurazione condivisa di IIS, il programma di installazione genera un errore di accesso negato durante il tentativo di configurare le impostazioni del modulo nel file *applicationHost.config* nella condivisione.

Quando si usa una configurazione condivisa di IIS, attenersi ai passaggi riportati di seguito:

1. Disabilitare la configurazione condivisa di IIS.
1. Eseguire il programma di installazione.
1. Esportare il file *applicationHost.config* aggiornato nella condivisione.
1. Abilitare di nuovo la configurazione condivisa di IIS.

## <a name="module-version-and-hosting-bundle-installer-logs"></a>Versione del modulo e log del programma di installazione del bundle di hosting

Per determinare la versione del modulo ASP.NET Core installato:

1. Nel sistema host passare a *%windir%\System32\inetsrv*.
1. Individuare il file *aspnetcore.dll*.
1. Fare clic con il pulsante destro del mouse sul file e scegliere **Proprietà** dal menu di scelta rapida.
1. Selezionare la scheda **Dettagli** . La versione del **file** e la **versione del prodotto** rappresentano la versione installata del modulo.

I log del programma di installazione del bundle di hosting per il modulo sono disponibili in *C: \\ utenti \\ % username% \\ AppData \\ Local \\ Temp*. Il file è denominato *dd_DotNetCoreWinSvrHosting__ \<timestamp> _000_AspNetCoreModule_x64. log*.

## <a name="module-schema-and-configuration-file-locations"></a>Percorsi dei file di modulo, schema e configurazione

### <a name="module"></a>Modulo

**IIS (x86/amd64):**

* %windir%\System32\inetsrv\aspnetcore.dll

* %windir%\SysWOW64\inetsrv\aspnetcore.dll

**IIS Express (x86/amd64):**

* %ProgramFiles%\IIS Express\aspnetcore.dll

* %ProgramFiles(x86)%\IIS Express\aspnetcore.dll

### <a name="schema"></a>SCHEMA

**IIS**

* %windir%\System32\inetsrv\config\schema\aspnetcore_schema.xml

**IIS Express**

* %ProgramFiles%\IIS Express\config\schema\aspnetcore_schema.xml

### <a name="configuration"></a>Configurazione

**IIS**

* %windir%\System32\inetsrv\config\applicationHost.config

**IIS Express**

* Visual Studio: {APPLICATION ROOT}\\.vs\config\applicationHost.config

* Interfaccia della riga di comando di *iisexpress.exe*: %USERPROFILE%\Documents\IISExpress\config\applicationhost.config

È possibile trovare i file eseguendo una ricerca di *aspnetcore* nel file *applicationHost.config*.

::: moniker-end

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:host-and-deploy/iis/index>
* <xref:host-and-deploy/azure-apps/index>
* [Origine riferimento modulo ASP.NET Core [ramo predefinito (Main)]](https://github.com/dotnet/aspnetcore/tree/main/src/Servers/IIS/AspNetCoreModuleV2): usare l'elenco a discesa **Branch** per selezionare una versione specifica (ad esempio, `release/3.1` ).
* <xref:host-and-deploy/iis/modules>
