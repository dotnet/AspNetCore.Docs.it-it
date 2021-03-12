---
title: Ospitare ASP.NET Core in un servizio Windows
author: rick-anderson
description: Informazioni su come ospitare un'app ASP.NET Core in un servizio Windows.
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
uid: host-and-deploy/windows-service
ms.openlocfilehash: e5c7dd0e52f1246d3ac6ad9622573db4c276654b
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102588815"
---
# <a name="host-aspnet-core-in-a-windows-service"></a>Ospitare ASP.NET Core in un servizio Windows

::: moniker range=">= aspnetcore-3.0"

È possibile ospitare un'app ASP.NET Core in Windows come [servizio Windows](/dotnet/framework/windows-services/introduction-to-windows-service-applications) senza usare IIS. Quando è ospitata come servizio di Windows, l'app viene avviata automaticamente dopo il riavvio del server.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/host-and-deploy/windows-service/samples) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="prerequisites"></a>Prerequisiti

* [ASP.NET Core SDK 2.1 o versione successiva](https://dotnet.microsoft.com/download)
* [PowerShell 6,2 o versione successiva](https://github.com/PowerShell/PowerShell)

## <a name="worker-service-template"></a>Modello di servizio di ruolo di lavoro

Il modello di servizio di ruolo di lavoro di ASP.NET Core rappresenta un punto di partenza per la scrittura di app di servizi a esecuzione prolungata. Per usare il modello come base per un'app di servizio Windows:

1. Creare un'app di servizio di ruolo di lavoro dal modello .NET Core.
1. Seguire le indicazioni nella sezione [Configurazione dell'app](#app-configuration) per aggiornare l'app di servizio di ruolo di lavoro affinché venga eseguita come servizio Windows.

[!INCLUDE[](~/includes/worker-template-instructions.md)]

## <a name="app-configuration"></a>Configurazione delle app

L'app richiede un riferimento al pacchetto per [Microsoft. Extensions. Hosting. WindowsServices](https://www.nuget.org/packages/Microsoft.Extensions.Hosting.WindowsServices).

`IHostBuilder.UseWindowsService` viene chiamato durante la compilazione dell'host. Se l'app è in esecuzione come servizio di Windows, il metodo:

* Imposta la durata dell'host su `WindowsServiceLifetime`.
* Imposta la [radice del contenuto](xref:fundamentals/index#content-root) su [AppContext. BaseDirectory](xref:System.AppContext.BaseDirectory). Per altre informazioni, vedere la sezione [Directory corrente e radice del contenuto](#current-directory-and-content-root).
* Abilita la registrazione nel registro eventi:
  * Il nome dell'applicazione viene usato come nome di origine predefinito.
  * Il livello di registrazione predefinito è *avviso* o superiore per un'app basata su un modello di ASP.NET Core che chiama `CreateDefaultBuilder` per compilare l'host.
  * Eseguire l'override del livello di registrazione predefinito con la `Logging:EventLog:LogLevel:Default` chiave in *appsettings.json* / *appSettings. { Environment}. JSON* o un altro provider di configurazione.
  * Solo gli amministratori possono creare nuove origini eventi. Quando non è possibile creare un'origine evento usando il nome dell'applicazione, viene registrato un avviso nell'origine *Applicazione* e i log eventi vengono disabilitati.

In `CreateHostBuilder` di *Program.cs*:

```csharp
Host.CreateDefaultBuilder(args)
    .UseWindowsService()
    ...
```

Questo argomento è accompagnato dalle app di esempio seguenti:

* Esempio di servizio di lavoro in background: un esempio di app non Web basato sul [modello di servizio](#worker-service-template) del ruolo di lavoro che usa i [servizi ospitati](xref:fundamentals/host/hosted-services) per le attività in background.
* Esempio di servizio app Web: un Razor esempio di app Web di pagine che viene eseguito come servizio Windows con [servizi ospitati](xref:fundamentals/host/hosted-services) per le attività in background.

Per informazioni aggiuntive su MVC, vedere gli articoli in <xref:mvc/overview> e <xref:migration/22-to-30> .

## <a name="deployment-type"></a>Tipo di distribuzione

Per informazioni e consigli sugli scenari di distribuzione, vedere [Distribuzione di applicazioni .NET Core](/dotnet/core/deploying/).

### <a name="sdk"></a>SDK

Per un servizio basato su app Web che usa le Razor pagine o i framework MVC, specificare Web SDK nel file di progetto:

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
```

Se il servizio esegue solo attività in background, ad esempio [servizi ospitati](xref:fundamentals/host/hosted-services), specificare l'SDK del ruolo di lavoro nel file di progetto:

```xml
<Project Sdk="Microsoft.NET.Sdk.Worker">
```

### <a name="framework-dependent-deployment-fdd"></a>Distribuzione dipendente dal framework

La distribuzione dipendente dal framework si basa sulla presenza di una versione condivisa a livello di sistema di .NET Core nel sistema di destinazione. Quando lo scenario di distribuzione dipendente dal framework viene implementato in base alle indicazioni di questo articolo, l'SDK genera un file eseguibile (con estensione *exe*) detto *eseguibile dipendente dal framework*.

Se si usa l' [SDK Web](#sdk), un file di *web.config* , che in genere viene prodotto durante la pubblicazione di un'app ASP.NET Core, non è necessario per un'app dei servizi Windows. Per disabilitare la creazione del file *web.config*, aggiungere la proprietà `<IsTransformWebConfigDisabled>` impostata su `true`.

```xml
<PropertyGroup>
  <TargetFramework>netcoreapp3.0</TargetFramework>
  <IsTransformWebConfigDisabled>true</IsTransformWebConfigDisabled>
</PropertyGroup>
```

### <a name="self-contained-deployment-scd"></a>Distribuzione autonoma

Una distribuzione autonoma non si basa sulla presenza di un framework condiviso nel sistema host. Il runtime e le dipendenze dell'app vengono distribuiti con l'app.

Un [identificatore di runtime (RID)](/dotnet/core/rid-catalog) di Windows viene incluso nel `<PropertyGroup>` che contiene il framework di destinazione:

```xml
<RuntimeIdentifier>win7-x64</RuntimeIdentifier>
```

Per eseguire la pubblicazione per più identificatori di runtime:

* Specificare gli identificatori di runtime in un elenco delimitato da punto e virgola.
* Usare il nome della proprietà [\<RuntimeIdentifiers>](/dotnet/core/tools/csproj#runtimeidentifiers) (plurale).

Per altre informazioni, vedere il [Catalogo RID di .NET Core](/dotnet/core/rid-catalog).

## <a name="service-user-account"></a>Account utente del servizio

Per creare un account utente per un servizio, usare il cmdlet [New-LocalUser](/powershell/module/microsoft.powershell.localaccounts/new-localuser) da una shell dei comandi di PowerShell 6 amministrativa.

In Aggiornamento di Windows 10 (ottobre 2018) (versione 1809/build 10.0.17763) o versioni successive:

```powershell
New-LocalUser -Name {SERVICE NAME}
```

In un sistema operativo Windows precedente ad Aggiornamento di Windows 10 (ottobre 2018) (versione 1809/build 10.0.17763):

```console
powershell -Command "New-LocalUser -Name {SERVICE NAME}"
```

Fornire una [password complessa](/windows/security/threat-protection/security-policy-settings/password-must-meet-complexity-requirements) quando richiesto.

Se non viene specificato il parametro `-AccountExpires` per il cmdlet [New-LocalUser](/powershell/module/microsoft.powershell.localaccounts/new-localuser) con un valore <xref:System.DateTime> di scadenza, l'account non scade.

Per altre informazioni, vedere [Microsoft.PowerShell.LocalAccounts](/powershell/module/microsoft.powershell.localaccounts/) e [Service User Accounts](/windows/desktop/services/service-user-accounts) (Account utente di servizio).

Un approccio alternativo per la gestione degli utenti quando si usa Active Directory consiste nell'usare account del servizio gestito. Per altre informazioni, vedere [Panoramica degli account del servizio gestito del gruppo](/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview).

## <a name="log-on-as-a-service-rights"></a>Diritti Accesso come servizio

Per stabilire i diritti *Accesso come servizio* per un account utente di servizio:

1. Aprire l'editor Criteri di sicurezza locali eseguendo *secpol.msc*.
1. Espandere il nodo **Criteri locali** e selezionare **Assegnazione diritti utente**.
1. Aprire il criterio **Accesso come servizio**.
1. Selezionare **Aggiungi utente o gruppo**.
1. Specificare il nome oggetto (account utente) in uno dei modi seguenti:
   1. Digitare l'account utente (`{DOMAIN OR COMPUTER NAME\USER}`) nel campo del nome oggetto e scegliere **OK** per aggiungere l'utente al criterio.
   1. Fare clic su **Advanced** (Avanzate). Selezionare **Trova**. Selezionare l'account utente dall'elenco. Selezionare **OK**. Scegliere di nuovo **OK** per aggiungere l'utente al criterio.
1. Scegliere **OK** o **Applica** per accettare le modifiche.

## <a name="create-and-manage-the-windows-service"></a>Creare e gestire il servizio di Windows

### <a name="create-a-service"></a>Creare un servizio

Usare i comandi di PowerShell per registrare un servizio. Da una shell dei comandi di PowerShell 6 amministrativa eseguire i comandi seguenti:

```powershell
$acl = Get-Acl "{EXE PATH}"
$aclRuleArgs = "{DOMAIN OR COMPUTER NAME\USER}", "Read,Write,ReadAndExecute", "ContainerInherit,ObjectInherit", "None", "Allow"
$accessRule = New-Object System.Security.AccessControl.FileSystemAccessRule($aclRuleArgs)
$acl.SetAccessRule($accessRule)
$acl | Set-Acl "{EXE PATH}"

New-Service -Name {SERVICE NAME} -BinaryPathName "{EXE FILE PATH}" -Credential "{DOMAIN OR COMPUTER NAME\USER}" -Description "{DESCRIPTION}" -DisplayName "{DISPLAY NAME}" -StartupType Automatic
```

* `{EXE PATH}`: Percorso della cartella dell'app nell'host (ad esempio, `d:\myservice` ). Non includere l'eseguibile dell'app nel percorso. Non è necessario aggiungere una barra finale.
* `{DOMAIN OR COMPUTER NAME\USER}`: Account utente del servizio (ad esempio, `Contoso\ServiceUser` ).
* `{SERVICE NAME}`: Nome del servizio (ad esempio, `MyService` ).
* `{EXE FILE PATH}`: Percorso eseguibile dell'app (ad esempio, `d:\myservice\myservice.exe` ). Includere il nome del file eseguibile con l'estensione.
* `{DESCRIPTION}`: Descrizione del servizio (ad esempio, `My sample service` ).
* `{DISPLAY NAME}`: Nome visualizzato del servizio (ad esempio, `My Service` ).

### <a name="start-a-service"></a>Avviare un servizio

Per avviare un servizio, usare il comando di PowerShell 6 seguente:

```powershell
Start-Service -Name {SERVICE NAME}
```

L'avvio del servizio richiede alcuni secondi.

### <a name="determine-a-services-status"></a>Determinare lo stato di un servizio

Per verificare lo stato di un servizio, usare il comando di PowerShell 6 seguente:

```powershell
Get-Service -Name {SERVICE NAME}
```

Lo stato viene indicato con uno dei valori seguenti:

* `Starting`
* `Running`
* `Stopping`
* `Stopped`

### <a name="stop-a-service"></a>Arrestare un servizio

Per arrestare un servizio, usare il comando di PowerShell 6 seguente:

```powershell
Stop-Service -Name {SERVICE NAME}
```

### <a name="remove-a-service"></a>Rimuovere un servizio

Dopo il breve lasso di tempo necessario per arrestare un servizio, rimuovere il servizio con il comando di PowerShell 6 seguente:

```powershell
Remove-Service -Name {SERVICE NAME}
```

## <a name="proxy-server-and-load-balancer-scenarios"></a>Scenari con server proxy e servizi di bilanciamento del carico

I servizi che interagiscono con le richieste da Internet o da una rete aziendale e si trovano dietro un proxy o un servizio di bilanciamento del carico potrebbero richiedere una configurazione aggiuntiva. Per altre informazioni, vedere <xref:host-and-deploy/proxy-load-balancer>.

## <a name="configure-endpoints"></a>Configurare gli endpoint

Per impostazione predefinita, ASP.NET Core è associato a `http://localhost:5000`. Configurare l'URL e la porta impostando la `ASPNETCORE_URLS` variabile di ambiente.

Per ulteriori approcci alla configurazione di porte e URL, vedere l'articolo relativo al server pertinente:

::: moniker-end

::: moniker range=">= aspnetcore-5.0"

* <xref:fundamentals/servers/kestrel/endpoints>

::: moniker-end

::: moniker range="< aspnetcore-5.0 >= aspnetcore-3.0"

* <xref:fundamentals/servers/kestrel#endpoint-configuration>

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

* <xref:fundamentals/servers/httpsys#configure-windows-server>

Le linee guida precedenti riguardano il supporto per gli endpoint HTTPS. Ad esempio, configurare l'app per HTTPS quando si usa l'autenticazione con un servizio Windows.

> [!NOTE]
> L'uso del certificato di sviluppo ASP.NET Core HTTPS per proteggere un endpoint del servizio non è supportato.

## <a name="current-directory-and-content-root"></a>Directory corrente e radice del contenuto

La directory di lavoro corrente restituita chiamando <xref:System.IO.Directory.GetCurrentDirectory%2A> per un servizio Windows è la cartella *C:\\WINDOWS\\system32*. La cartella *system32* non è un percorso appropriato per archiviare i file di un servizio, ad esempio i file di impostazioni. Usare uno degli approcci seguenti per gestire e accedere agli asset e ai file di impostazioni di un servizio.

### <a name="use-contentrootpath-or-contentrootfileprovider"></a>Usare ContentRootPath o ContentRootFileProvider

Usare [IHostEnvironment.ContentRootPath](xref:Microsoft.Extensions.Hosting.IHostEnvironment.ContentRootPath) o <xref:Microsoft.Extensions.Hosting.IHostEnvironment.ContentRootFileProvider> per individuare le risorse di un'app.

Quando l'app viene eseguita come servizio, <xref:Microsoft.Extensions.Hosting.WindowsServiceLifetimeHostBuilderExtensions.UseWindowsService%2A> imposta <xref:Microsoft.Extensions.Hosting.IHostEnvironment.ContentRootPath> su [AppContext. BaseDirectory](xref:System.AppContext.BaseDirectory).

I file di impostazioni predefinite dell'app *appsettings.json* e *appSettings. { Environment}. JSON*, viene caricato dalla radice del contenuto dell'app chiamando [CreateDefaultBuilder durante la costruzione dell'host](xref:fundamentals/host/generic-host#set-up-a-host).

Per gli altri file di impostazioni caricati dal codice Developer in <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureAppConfiguration%2A> , non è necessario chiamare <xref:Microsoft.Extensions.Configuration.FileConfigurationExtensions.SetBasePath%2A> . Nell'esempio seguente, il *custom_settings.js* nel file è presente nella radice del contenuto dell'app e viene caricato senza impostare esplicitamente un percorso di base:

[!code-csharp[](windows-service/samples_snapshot/CustomSettingsExample.cs?highlight=13)]

Non tentare di usare <xref:System.IO.Directory.GetCurrentDirectory%2A> per ottenere un percorso di risorsa perché un'app di servizio Windows restituisce la cartella *C: \\ Windows \\ system32* come directory corrente.

### <a name="store-a-services-files-in-a-suitable-location-on-disk"></a>Archiviare i file di un servizio in un percorso appropriato nel disco

Specificare un percorso assoluto con <xref:Microsoft.Extensions.Configuration.FileConfigurationExtensions.SetBasePath%2A> quando si usa un <xref:Microsoft.Extensions.Configuration.IConfigurationBuilder> per la cartella contenente i file.

## <a name="troubleshoot"></a>Risolvere problemi

Per risolvere i problemi relativi a un'app di servizio Windows, vedere <xref:test/troubleshoot> .

### <a name="common-errors"></a>Errori comuni

* È in uso una versione precedente o provvisoria di PowerShell.
* Il servizio registrato non usa l'output **pubblicato** dell'app dal comando [DotNet Publish](/dotnet/core/tools/dotnet-publish) . L'output del comando [DotNet Build](/dotnet/core/tools/dotnet-build) non è supportato per la distribuzione di app. Gli asset pubblicati si trovano in una delle cartelle seguenti, a seconda del tipo di distribuzione:
  * *bin/Release/{Target Framework}/Publish* (FDD)
  * *bin/Release/{Target Framework}/{Runtime Identifier}/Publish* (SCD)
* Il servizio non è nello stato in esecuzione.
* I percorsi delle risorse utilizzate dall'app, ad esempio i certificati, non sono corretti. Il percorso di base di un servizio Windows è *c: \\ Windows \\ system32*.
* L'utente non dispone *di diritti di accesso come servizio* .
* La password dell'utente è scaduta o non è stata passata correttamente durante l'esecuzione del `New-Service` comando di PowerShell.
* L'app richiede l'autenticazione ASP.NET Core ma non è configurata per le connessioni protette (HTTPS).
* La porta dell'URL della richiesta non è corretta o non è configurata correttamente nell'app.

### <a name="system-and-application-event-logs"></a>Log eventi di sistema e dell'applicazione

Accedere ai registri eventi di sistema e dell'applicazione:

1. Aprire il menu Start, cercare *Visualizzatore eventi* e selezionare l'app **Visualizzatore eventi** .
1. In **Visualizzatore eventi** aprire il nodo **Registri di Windows**.
1. Selezionare **sistema** per aprire il registro eventi di sistema. Selezionare **Applicazione** per aprire il log eventi dell'applicazione.
1. Cercare gli errori associati all'app in cui si è verificato il problema.

### <a name="run-the-app-at-a-command-prompt"></a>Eseguire l'app da un prompt dei comandi

Molti errori di avvio non producono informazioni utili nei log eventi. È possibile individuare la causa di alcuni errori eseguendo l'app da un prompt dei comandi nel sistema host. Per registrare dettagli aggiuntivi dall'app, abbassare il [livello di registrazione](xref:fundamentals/logging/index#log-level) o eseguire l'app nell' [ambiente di sviluppo](xref:fundamentals/environments).

### <a name="clear-package-caches"></a>Cancella cache di pacchetti

Un'app funzionante potrebbe non riuscire immediatamente dopo l'aggiornamento del .NET Core SDK nel computer di sviluppo o la modifica delle versioni del pacchetto all'interno dell'app. In alcuni casi i pacchetti incoerenti possono interrompere un'app quando si eseguono aggiornamenti principali. La maggior parte di questi problemi può essere risolta attenendosi alle istruzioni seguenti:

1. Eliminare le cartelle *bin* e *obj*.
1. Cancellare le cache dei pacchetti eseguendo [le impostazioni locali di DotNet NuGet All--Clear](/dotnet/core/tools/dotnet-nuget-locals) da una shell dei comandi.

   La cancellazione delle cache dei pacchetti può essere eseguita anche con lo strumento [nuget.exe](https://www.nuget.org/downloads) e l'esecuzione del comando `nuget locals all -clear` . *nuget.exe* non è un'installazione inclusa con il sistema operativo desktop Windows e deve essere ottenuta separatamente dal [sito Web NuGet](https://www.nuget.org/downloads).

1. Ripristinare e ricompilare il progetto.
1. Eliminare tutti i file nella cartella di distribuzione nel server prima di ridistribuire l'app.

### <a name="slow-or-hanging-app"></a>App lenta o bloccata

Un *dump di arresto anomalo* del sistema è uno snapshot della memoria del sistema e può contribuire a determinare la provocazione di un arresto anomalo dell'app, dell'avvio o dell'applicazione lenta.

#### <a name="app-crashes-or-encounters-an-exception"></a>Arresto anomalo o eccezione di un'app

Ottenere e analizzare un dump da [Segnalazione errori Windows](/windows/desktop/wer/windows-error-reporting):

1. Creare una cartella per i file dump di arresto anomalo del sistema in `c:\dumps`.
1. Eseguire lo [script di PowerShell EnableDumps](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/host-and-deploy/windows-service/samples/scripts/EnableDumps.ps1) con il nome dell'eseguibile dell'applicazione:

   ```powershell
   .\EnableDumps {APPLICATION EXE} c:\dumps
   ```

1. Eseguire l'app nelle condizioni che causano l'arresto anomalo.
1. Dopo che si è verificato l'arresto anomalo, eseguire lo [script di PowerShell DisableDumps](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/host-and-deploy/windows-service/samples/scripts/DisableDumps.ps1):

   ```powershell
   .\DisableDumps {APPLICATION EXE}
   ```

Dopo l'arresto anomalo di un'app e la raccolta dei dump, l'app può terminare normalmente. Lo script di PowerShell configura Segnalazione errori Windows per raccogliere fino a cinque dump per ogni app.

> [!WARNING]
> I dump di arresto anomalo del sistema potrebbero richiedere una grande quantità di spazio su disco (fino a diversi gigabyte ognuno).

#### <a name="app-hangs-fails-during-startup-or-runs-normally"></a>L'app si blocca, si verifica un errore durante l'avvio o viene eseguita normalmente

Quando un'app si *blocca* (smette di rispondere ma non si arresta in modo anomalo), si verifica un errore durante l'avvio o viene eseguita normalmente, vedere [file di dump in modalità utente: scegliere lo strumento migliore](/windows-hardware/drivers/debugger/user-mode-dump-files#choosing-the-best-tool) per selezionare uno strumento appropriato per produrre il dump.

#### <a name="analyze-the-dump"></a>Analizzare il dump

È possibile analizzare un dump usando diversi approcci. Per altre informazioni, vedere [Analyzing a User-Mode Dump File](/windows-hardware/drivers/debugger/analyzing-a-user-mode-dump-file) (Analisi di un file dump in modalità utente).

## <a name="additional-resources"></a>Risorse aggiuntive

::: moniker-end

::: moniker range=">= aspnetcore-5.0"
* [Configurazione dell'endpoint Kestrel](xref:fundamentals/servers/kestrel/endpoints) (include la configurazione HTTPS e il supporto SNI)
::: moniker-end
::: moniker range="< aspnetcore-5.0 >= aspnetcore-3.0"
* [Configurazione dell'endpoint Kestrel](xref:fundamentals/servers/kestrel#endpoint-configuration) (include la configurazione HTTPS e il supporto SNI)
::: moniker-end

::: moniker range=">= aspnetcore-3.0"
* <xref:fundamentals/host/generic-host>
* <xref:test/troubleshoot>

::: moniker-end

::: moniker range="= aspnetcore-2.2"

È possibile ospitare un'app ASP.NET Core in Windows come [servizio Windows](/dotnet/framework/windows-services/introduction-to-windows-service-applications) senza usare IIS. Quando è ospitata come servizio di Windows, l'app viene avviata automaticamente dopo il riavvio del server.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/host-and-deploy/windows-service/samples) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="prerequisites"></a>Prerequisiti

* [ASP.NET Core SDK 2.1 o versione successiva](https://dotnet.microsoft.com/download)
* [PowerShell 6,2 o versione successiva](https://github.com/PowerShell/PowerShell)

## <a name="app-configuration"></a>Configurazione delle app

L'app richiede i riferimenti ai pacchetti [Microsoft.AspNetCore.Hosting.WindowsServices](https://www.nuget.org/packages/Microsoft.AspNetCore.Hosting.WindowsServices) e [Microsoft.Extensions.Logging.EventLog](https://www.nuget.org/packages/Microsoft.Extensions.Logging.EventLog).

Per eseguire test e debug durante l'esecuzione all'esterno di un servizio, aggiungere il codice per determinare se l'app è in esecuzione come servizio o è un'app console. Controllare se il debugger è collegato o se è presente un'opzione `--console`. Se una delle due condizioni è vera (l'app non è eseguita come servizio), chiamare <xref:Microsoft.AspNetCore.Hosting.WebHostExtensions.Run*>. Se le condizioni sono false (l'app è eseguita come servizio):

* Chiamare <xref:System.IO.Directory.SetCurrentDirectory*> e usare un percorso per la posizione di pubblicazione dell'app. Non chiamare <xref:System.IO.Directory.GetCurrentDirectory*> per ottenere il percorso perché un'app servizio Windows restituisce la cartella *C:\\WINDOWS\\system32* quando viene chiamato <xref:System.IO.Directory.GetCurrentDirectory*>. Per altre informazioni, vedere la sezione [Directory corrente e radice del contenuto](#current-directory-and-content-root). Questo passaggio viene eseguito prima che l'app venga configurata in `CreateWebHostBuilder`.
* Chiamare <xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostWindowsServiceExtensions.RunAsService*> per eseguire l'app come servizio.

Dato che il [provider di configurazione della riga di comando](xref:fundamentals/configuration/index#command-line-configuration-provider) richiede coppie nome-valore per gli argomenti della riga di comando, l'opzione `--console` viene rimossa dagli argomenti prima che <xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder*> riceva gli argomenti.

Per scrivere nel registro eventi di Windows, aggiungere il provider EventLog a <xref:Microsoft.AspNetCore.Hosting.WebHostBuilder.ConfigureLogging*>. Impostare il livello di registrazione con la chiave `Logging:LogLevel:Default` nel file *appsettings.Production.json*.

Nel seguente esempio dell'app di esempio, viene chiamato `RunAsCustomService` invece di <xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostWindowsServiceExtensions.RunAsService*> per gestire gli eventi di durata all'interno dell'app. Per altre informazioni, vedere la sezione [Gestire gli eventi di avvio e arresto](#handle-starting-and-stopping-events).

[!code-csharp[](windows-service/samples/2.x/AspNetCoreService/Program.cs?name=snippet_Program)]

## <a name="deployment-type"></a>Tipo di distribuzione

Per informazioni e consigli sugli scenari di distribuzione, vedere [Distribuzione di applicazioni .NET Core](/dotnet/core/deploying/).

### <a name="sdk"></a>SDK

Per un servizio basato su app Web che usa le Razor pagine o i framework MVC, specificare Web SDK nel file di progetto:

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
```

Se il servizio esegue solo attività in background, ad esempio [servizi ospitati](xref:fundamentals/host/hosted-services), specificare l'SDK del ruolo di lavoro nel file di progetto:

```xml
<Project Sdk="Microsoft.NET.Sdk.Worker">
```

### <a name="framework-dependent-deployment-fdd"></a>Distribuzione dipendente dal framework

La distribuzione dipendente dal framework si basa sulla presenza di una versione condivisa a livello di sistema di .NET Core nel sistema di destinazione. Quando lo scenario di distribuzione dipendente dal framework viene implementato in base alle indicazioni di questo articolo, l'SDK genera un file eseguibile (con estensione *exe*) detto *eseguibile dipendente dal framework*.

L'identificatore di Windows [Runtime (RID)](/dotnet/core/rid-catalog) ( [\<RuntimeIdentifier>](/dotnet/core/tools/csproj#runtimeidentifier) ) contiene il Framework di destinazione. Nell'esempio seguente il RID è impostato su `win7-x64`. La proprietà `<SelfContained>` è impostata su `false`. Queste proprietà indicano all'SDK di generare un file eseguibile (con estensione *exe*) per Windows e un'app che dipende dal framework .NET Core condiviso.

Un file *web.config*, che viene normalmente generato quando si pubblica un'app ASP.NET Core, non è necessario per un'app di servizi Windows. Per disabilitare la creazione del file *web.config*, aggiungere la proprietà `<IsTransformWebConfigDisabled>` impostata su `true`.

```xml
<PropertyGroup>
  <TargetFramework>netcoreapp2.2</TargetFramework>
  <RuntimeIdentifier>win7-x64</RuntimeIdentifier>
  <SelfContained>false</SelfContained>
  <IsTransformWebConfigDisabled>true</IsTransformWebConfigDisabled>
</PropertyGroup>
```

### <a name="self-contained-deployment-scd"></a>Distribuzione autonoma

Una distribuzione autonoma non si basa sulla presenza di un framework condiviso nel sistema host. Il runtime e le dipendenze dell'app vengono distribuiti con l'app.

Un [identificatore di runtime (RID)](/dotnet/core/rid-catalog) di Windows viene incluso nel `<PropertyGroup>` che contiene il framework di destinazione:

```xml
<RuntimeIdentifier>win7-x64</RuntimeIdentifier>
```

Per eseguire la pubblicazione per più identificatori di runtime:

* Specificare gli identificatori di runtime in un elenco delimitato da punto e virgola.
* Usare il nome della proprietà [\<RuntimeIdentifiers>](/dotnet/core/tools/csproj#runtimeidentifiers) (plurale).

Per altre informazioni, vedere il [Catalogo RID di .NET Core](/dotnet/core/rid-catalog).

Una proprietà `<SelfContained>` è impostata su `true`:

```xml
<SelfContained>true</SelfContained>
```

## <a name="service-user-account"></a>Account utente del servizio

Per creare un account utente per un servizio, usare il cmdlet [New-LocalUser](/powershell/module/microsoft.powershell.localaccounts/new-localuser) da una shell dei comandi di PowerShell 6 amministrativa.

In Aggiornamento di Windows 10 (ottobre 2018) (versione 1809/build 10.0.17763) o versioni successive:

```powershell
New-LocalUser -Name {SERVICE NAME}
```

In un sistema operativo Windows precedente ad Aggiornamento di Windows 10 (ottobre 2018) (versione 1809/build 10.0.17763):

```console
powershell -Command "New-LocalUser -Name {SERVICE NAME}"
```

Fornire una [password complessa](/windows/security/threat-protection/security-policy-settings/password-must-meet-complexity-requirements) quando richiesto.

Se non viene specificato il parametro `-AccountExpires` per il cmdlet [New-LocalUser](/powershell/module/microsoft.powershell.localaccounts/new-localuser) con un valore <xref:System.DateTime> di scadenza, l'account non scade.

Per altre informazioni, vedere [Microsoft.PowerShell.LocalAccounts](/powershell/module/microsoft.powershell.localaccounts/) e [Service User Accounts](/windows/desktop/services/service-user-accounts) (Account utente di servizio).

Un approccio alternativo per la gestione degli utenti quando si usa Active Directory consiste nell'usare account del servizio gestito. Per altre informazioni, vedere [Panoramica degli account del servizio gestito del gruppo](/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview).

## <a name="log-on-as-a-service-rights"></a>Diritti Accesso come servizio

Per stabilire i diritti *Accesso come servizio* per un account utente di servizio:

1. Aprire l'editor Criteri di sicurezza locali eseguendo *secpol.msc*.
1. Espandere il nodo **Criteri locali** e selezionare **Assegnazione diritti utente**.
1. Aprire il criterio **Accesso come servizio**.
1. Selezionare **Aggiungi utente o gruppo**.
1. Specificare il nome oggetto (account utente) in uno dei modi seguenti:
   1. Digitare l'account utente (`{DOMAIN OR COMPUTER NAME\USER}`) nel campo del nome oggetto e scegliere **OK** per aggiungere l'utente al criterio.
   1. Fare clic su **Advanced** (Avanzate). Selezionare **Trova**. Selezionare l'account utente dall'elenco. Selezionare **OK**. Scegliere di nuovo **OK** per aggiungere l'utente al criterio.
1. Scegliere **OK** o **Applica** per accettare le modifiche.

## <a name="create-and-manage-the-windows-service"></a>Creare e gestire il servizio di Windows

### <a name="create-a-service"></a>Creare un servizio

Usare i comandi di PowerShell per registrare un servizio. Da una shell dei comandi di PowerShell 6 amministrativa eseguire i comandi seguenti:

```powershell
$acl = Get-Acl "{EXE PATH}"
$aclRuleArgs = {DOMAIN OR COMPUTER NAME\USER}, "Read,Write,ReadAndExecute", "ContainerInherit,ObjectInherit", "None", "Allow"
$accessRule = New-Object System.Security.AccessControl.FileSystemAccessRule($aclRuleArgs)
$acl.SetAccessRule($accessRule)
$acl | Set-Acl "{EXE PATH}"

New-Service -Name {SERVICE NAME} -BinaryPathName {EXE FILE PATH} -Credential {DOMAIN OR COMPUTER NAME\USER} -Description "{DESCRIPTION}" -DisplayName "{DISPLAY NAME}" -StartupType Automatic
```

* `{EXE PATH}`: Percorso della cartella dell'app nell'host (ad esempio, `d:\myservice` ). Non includere l'eseguibile dell'app nel percorso. Non è necessario aggiungere una barra finale.
* `{DOMAIN OR COMPUTER NAME\USER}`: Account utente del servizio (ad esempio, `Contoso\ServiceUser` ).
* `{SERVICE NAME}`: Nome del servizio (ad esempio, `MyService` ).
* `{EXE FILE PATH}`: Percorso eseguibile dell'app (ad esempio, `d:\myservice\myservice.exe` ). Includere il nome del file eseguibile con l'estensione.
* `{DESCRIPTION}`: Descrizione del servizio (ad esempio, `My sample service` ).
* `{DISPLAY NAME}`: Nome visualizzato del servizio (ad esempio, `My Service` ).

### <a name="start-a-service"></a>Avviare un servizio

Per avviare un servizio, usare il comando di PowerShell 6 seguente:

```powershell
Start-Service -Name {SERVICE NAME}
```

L'avvio del servizio richiede alcuni secondi.

### <a name="determine-a-services-status"></a>Determinare lo stato di un servizio

Per verificare lo stato di un servizio, usare il comando di PowerShell 6 seguente:

```powershell
Get-Service -Name {SERVICE NAME}
```

Lo stato viene indicato con uno dei valori seguenti:

* `Starting`
* `Running`
* `Stopping`
* `Stopped`

### <a name="stop-a-service"></a>Arrestare un servizio

Per arrestare un servizio, usare il comando di PowerShell 6 seguente:

```powershell
Stop-Service -Name {SERVICE NAME}
```

### <a name="remove-a-service"></a>Rimuovere un servizio

Dopo il breve lasso di tempo necessario per arrestare un servizio, rimuovere il servizio con il comando di PowerShell 6 seguente:

```powershell
Remove-Service -Name {SERVICE NAME}
```

## <a name="handle-starting-and-stopping-events"></a>Gestire gli eventi di avvio e arresto

Per gestire gli eventi <xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostService.OnStarting*>, <xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostService.OnStarted*> e <xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostService.OnStopping*>:

1. Creare una classe che deriva da <xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostService> con i metodi `OnStarting`, `OnStarted` e `OnStopping`:

   [!code-csharp[](windows-service/samples/2.x/AspNetCoreService/CustomWebHostService.cs?name=snippet_CustomWebHostService)]

2. Creare un metodo di estensione per <xref:Microsoft.AspNetCore.Hosting.IWebHost> che passa `CustomWebHostService` a <xref:System.ServiceProcess.ServiceBase.Run*>:

   [!code-csharp[](windows-service/samples/2.x/AspNetCoreService/WebHostServiceExtensions.cs?name=ExtensionsClass)]

3. In `Program.Main` chiamare il metodo di estensione `RunAsCustomService` invece di <xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostWindowsServiceExtensions.RunAsService*>:

   ```csharp
   host.RunAsCustomService();
   ```

   Per visualizzare il percorso di <xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostWindowsServiceExtensions.RunAsService*> in `Program.Main`, fare riferimento all'esempio di codice illustrato nella sezione [Tipo di distribuzione](#deployment-type).

## <a name="proxy-server-and-load-balancer-scenarios"></a>Scenari con server proxy e servizi di bilanciamento del carico

I servizi che interagiscono con le richieste da Internet o da una rete aziendale e si trovano dietro un proxy o un servizio di bilanciamento del carico potrebbero richiedere una configurazione aggiuntiva. Per altre informazioni, vedere <xref:host-and-deploy/proxy-load-balancer>.

## <a name="configure-endpoints"></a>Configurare gli endpoint

Per impostazione predefinita, ASP.NET Core è associato a `http://localhost:5000`. Configurare l'URL e la porta impostando la `ASPNETCORE_URLS` variabile di ambiente.

Per ulteriori approcci alla configurazione di porte e URL, vedere l'articolo relativo al server pertinente:

* <xref:fundamentals/servers/kestrel#endpoint-configuration>
* <xref:fundamentals/servers/httpsys#configure-windows-server>

Le linee guida precedenti riguardano il supporto per gli endpoint HTTPS. Ad esempio, configurare l'app per HTTPS quando si usa l'autenticazione con un servizio Windows.

> [!NOTE]
> L'uso del certificato di sviluppo ASP.NET Core HTTPS per proteggere un endpoint del servizio non è supportato.

## <a name="current-directory-and-content-root"></a>Directory corrente e radice del contenuto

La directory di lavoro corrente restituita chiamando <xref:System.IO.Directory.GetCurrentDirectory*> per un servizio Windows è la cartella *C:\\WINDOWS\\system32*. La cartella *system32* non è un percorso appropriato per archiviare i file di un servizio, ad esempio i file di impostazioni. Usare uno degli approcci seguenti per gestire e accedere agli asset e ai file di impostazioni di un servizio.

### <a name="set-the-content-root-path-to-the-apps-folder"></a>Impostare il percorso radice del contenuto sulla cartella dell'app

<xref:Microsoft.Extensions.Hosting.IHostingEnvironment.ContentRootPath*> è lo stesso percorso fornito all'argomento `binPath` durante la creazione di un servizio. Anziché chiamare `GetCurrentDirectory` per creare percorsi per i file di impostazioni, chiamare <xref:System.IO.Directory.SetCurrentDirectory*> con il percorso della radice del [contenuto](xref:fundamentals/index#content-root)dell'app.

In `Program.Main`, determinare il percorso della cartella dell'eseguibile del servizio e usare il percorso per stabilire la radice del contenuto dell'app:

```csharp
var pathToExe = Process.GetCurrentProcess().MainModule.FileName;
var pathToContentRoot = Path.GetDirectoryName(pathToExe);
Directory.SetCurrentDirectory(pathToContentRoot);

CreateWebHostBuilder(args)
    .Build()
    .RunAsService();
```

### <a name="store-a-services-files-in-a-suitable-location-on-disk"></a>Archiviare i file di un servizio in un percorso appropriato nel disco

Specificare un percorso assoluto con <xref:Microsoft.Extensions.Configuration.FileConfigurationExtensions.SetBasePath*> quando si usa un <xref:Microsoft.Extensions.Configuration.IConfigurationBuilder> per la cartella contenente i file.

## <a name="troubleshoot"></a>Risolvere problemi

Per risolvere i problemi relativi a un'app di servizio Windows, vedere <xref:test/troubleshoot> .

### <a name="common-errors"></a>Errori comuni

* È in uso una versione precedente o provvisoria di PowerShell.
* Il servizio registrato non usa l'output **pubblicato** dell'app dal comando [DotNet Publish](/dotnet/core/tools/dotnet-publish) . L'output del comando [DotNet Build](/dotnet/core/tools/dotnet-build) non è supportato per la distribuzione di app. Gli asset pubblicati si trovano in una delle cartelle seguenti, a seconda del tipo di distribuzione:
  * *bin/Release/{Target Framework}/Publish* (FDD)
  * *bin/Release/{Target Framework}/{Runtime Identifier}/Publish* (SCD)
* Il servizio non è nello stato in esecuzione.
* I percorsi delle risorse utilizzate dall'app, ad esempio i certificati, non sono corretti. Il percorso di base di un servizio Windows è *c: \\ Windows \\ system32*.
* L'utente non dispone *di diritti di accesso come servizio* .
* La password dell'utente è scaduta o non è stata passata correttamente durante l'esecuzione del `New-Service` comando di PowerShell.
* L'app richiede l'autenticazione ASP.NET Core ma non è configurata per le connessioni protette (HTTPS).
* La porta dell'URL della richiesta non è corretta o non è configurata correttamente nell'app.

### <a name="system-and-application-event-logs"></a>Log eventi di sistema e dell'applicazione

Accedere ai registri eventi di sistema e dell'applicazione:

1. Aprire il menu Start, cercare *Visualizzatore eventi* e selezionare l'app **Visualizzatore eventi** .
1. In **Visualizzatore eventi** aprire il nodo **Registri di Windows**.
1. Selezionare **sistema** per aprire il registro eventi di sistema. Selezionare **Applicazione** per aprire il log eventi dell'applicazione.
1. Cercare gli errori associati all'app in cui si è verificato il problema.

### <a name="run-the-app-at-a-command-prompt"></a>Eseguire l'app da un prompt dei comandi

Molti errori di avvio non producono informazioni utili nei log eventi. È possibile individuare la causa di alcuni errori eseguendo l'app da un prompt dei comandi nel sistema host. Per registrare dettagli aggiuntivi dall'app, abbassare il [livello di registrazione](xref:fundamentals/logging/index#log-level) o eseguire l'app nell' [ambiente di sviluppo](xref:fundamentals/environments).

### <a name="clear-package-caches"></a>Cancella cache di pacchetti

Un'app funzionante potrebbe non riuscire immediatamente dopo l'aggiornamento del .NET Core SDK nel computer di sviluppo o la modifica delle versioni del pacchetto all'interno dell'app. In alcuni casi i pacchetti incoerenti possono interrompere un'app quando si eseguono aggiornamenti principali. La maggior parte di questi problemi può essere risolta attenendosi alle istruzioni seguenti:

1. Eliminare le cartelle *bin* e *obj*.
1. Cancellare le cache dei pacchetti eseguendo [le impostazioni locali di DotNet NuGet All--Clear](/dotnet/core/tools/dotnet-nuget-locals) da una shell dei comandi.

   La cancellazione delle cache dei pacchetti può essere eseguita anche con lo strumento [nuget.exe](https://www.nuget.org/downloads) e l'esecuzione del comando `nuget locals all -clear` . *nuget.exe* non è un'installazione inclusa con il sistema operativo desktop Windows e deve essere ottenuta separatamente dal [sito Web NuGet](https://www.nuget.org/downloads).

1. Ripristinare e ricompilare il progetto.
1. Eliminare tutti i file nella cartella di distribuzione nel server prima di ridistribuire l'app.

### <a name="slow-or-hanging-app"></a>App lenta o bloccata

Un *dump di arresto anomalo* del sistema è uno snapshot della memoria del sistema e può contribuire a determinare la provocazione di un arresto anomalo dell'app, dell'avvio o dell'applicazione lenta.

#### <a name="app-crashes-or-encounters-an-exception"></a>Arresto anomalo o eccezione di un'app

Ottenere e analizzare un dump da [Segnalazione errori Windows](/windows/desktop/wer/windows-error-reporting):

1. Creare una cartella per i file dump di arresto anomalo del sistema in `c:\dumps`.
1. Eseguire lo [script di PowerShell EnableDumps](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/host-and-deploy/windows-service/scripts/EnableDumps.ps1) con il nome dell'eseguibile dell'applicazione:

   ```console
   .\EnableDumps {APPLICATION EXE} c:\dumps
   ```

1. Eseguire l'app nelle condizioni che causano l'arresto anomalo.
1. Dopo che si è verificato l'arresto anomalo, eseguire lo [script di PowerShell DisableDumps](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/host-and-deploy/windows-service/scripts/DisableDumps.ps1):

   ```console
   .\DisableDumps {APPLICATION EXE}
   ```

Dopo l'arresto anomalo di un'app e la raccolta dei dump, l'app può terminare normalmente. Lo script di PowerShell configura Segnalazione errori Windows per raccogliere fino a cinque dump per ogni app.

> [!WARNING]
> I dump di arresto anomalo del sistema potrebbero richiedere una grande quantità di spazio su disco (fino a diversi gigabyte ognuno).

#### <a name="app-hangs-fails-during-startup-or-runs-normally"></a>L'app si blocca, si verifica un errore durante l'avvio o viene eseguita normalmente

Quando un'app si *blocca* (smette di rispondere ma non si arresta in modo anomalo), si verifica un errore durante l'avvio o viene eseguita normalmente, vedere [file di dump in modalità utente: scegliere lo strumento migliore](/windows-hardware/drivers/debugger/user-mode-dump-files#choosing-the-best-tool) per selezionare uno strumento appropriato per produrre il dump.

#### <a name="analyze-the-dump"></a>Analizzare il dump

È possibile analizzare un dump usando diversi approcci. Per altre informazioni, vedere [Analyzing a User-Mode Dump File](/windows-hardware/drivers/debugger/analyzing-a-user-mode-dump-file) (Analisi di un file dump in modalità utente).

## <a name="additional-resources"></a>Risorse aggiuntive

* [Configurazione dell'endpoint Kestrel](xref:fundamentals/servers/kestrel#endpoint-configuration) (include la configurazione HTTPS e il supporto SNI)
* <xref:fundamentals/host/web-host>
* <xref:test/troubleshoot>

::: moniker-end

::: moniker range="< aspnetcore-2.2"

È possibile ospitare un'app ASP.NET Core in Windows come [servizio Windows](/dotnet/framework/windows-services/introduction-to-windows-service-applications) senza usare IIS. Quando è ospitata come servizio di Windows, l'app viene avviata automaticamente dopo il riavvio del server.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/host-and-deploy/windows-service/samples) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="prerequisites"></a>Prerequisiti

* [ASP.NET Core SDK 2.1 o versione successiva](https://dotnet.microsoft.com/download)
* [PowerShell 6,2 o versione successiva](https://github.com/PowerShell/PowerShell)

## <a name="app-configuration"></a>Configurazione delle app

L'app richiede i riferimenti ai pacchetti [Microsoft.AspNetCore.Hosting.WindowsServices](https://www.nuget.org/packages/Microsoft.AspNetCore.Hosting.WindowsServices) e [Microsoft.Extensions.Logging.EventLog](https://www.nuget.org/packages/Microsoft.Extensions.Logging.EventLog).

Per eseguire test e debug durante l'esecuzione all'esterno di un servizio, aggiungere il codice per determinare se l'app è in esecuzione come servizio o è un'app console. Controllare se il debugger è collegato o se è presente un'opzione `--console`. Se una delle due condizioni è vera (l'app non è eseguita come servizio), chiamare <xref:Microsoft.AspNetCore.Hosting.WebHostExtensions.Run*>. Se le condizioni sono false (l'app è eseguita come servizio):

* Chiamare <xref:System.IO.Directory.SetCurrentDirectory*> e usare un percorso per la posizione di pubblicazione dell'app. Non chiamare <xref:System.IO.Directory.GetCurrentDirectory*> per ottenere il percorso perché un'app servizio Windows restituisce la cartella *C:\\WINDOWS\\system32* quando viene chiamato <xref:System.IO.Directory.GetCurrentDirectory*>. Per altre informazioni, vedere la sezione [Directory corrente e radice del contenuto](#current-directory-and-content-root). Questo passaggio viene eseguito prima che l'app venga configurata in `CreateWebHostBuilder`.
* Chiamare <xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostWindowsServiceExtensions.RunAsService*> per eseguire l'app come servizio.

Dato che il [provider di configurazione della riga di comando](xref:fundamentals/configuration/index#command-line-configuration-provider) richiede coppie nome-valore per gli argomenti della riga di comando, l'opzione `--console` viene rimossa dagli argomenti prima che <xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder*> riceva gli argomenti.

Per scrivere nel registro eventi di Windows, aggiungere il provider EventLog a <xref:Microsoft.AspNetCore.Hosting.WebHostBuilder.ConfigureLogging*>. Impostare il livello di registrazione con la chiave `Logging:LogLevel:Default` nel file *appsettings.Production.json*.

Nel seguente esempio dell'app di esempio, viene chiamato `RunAsCustomService` invece di <xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostWindowsServiceExtensions.RunAsService*> per gestire gli eventi di durata all'interno dell'app. Per altre informazioni, vedere la sezione [Gestire gli eventi di avvio e arresto](#handle-starting-and-stopping-events).

[!code-csharp[](windows-service/samples/2.x/AspNetCoreService/Program.cs?name=snippet_Program)]

## <a name="deployment-type"></a>Tipo di distribuzione

Per informazioni e consigli sugli scenari di distribuzione, vedere [Distribuzione di applicazioni .NET Core](/dotnet/core/deploying/).

### <a name="sdk"></a>SDK

Per un servizio basato su app Web che usa le Razor pagine o i framework MVC, specificare Web SDK nel file di progetto:

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
```

Se il servizio esegue solo attività in background, ad esempio [servizi ospitati](xref:fundamentals/host/hosted-services), specificare l'SDK del ruolo di lavoro nel file di progetto:

```xml
<Project Sdk="Microsoft.NET.Sdk.Worker">
```

### <a name="framework-dependent-deployment-fdd"></a>Distribuzione dipendente dal framework

La distribuzione dipendente dal framework si basa sulla presenza di una versione condivisa a livello di sistema di .NET Core nel sistema di destinazione. Quando lo scenario di distribuzione dipendente dal framework viene implementato in base alle indicazioni di questo articolo, l'SDK genera un file eseguibile (con estensione *exe*) detto *eseguibile dipendente dal framework*.

L'identificatore di Windows [Runtime (RID)](/dotnet/core/rid-catalog) ( [\<RuntimeIdentifier>](/dotnet/core/tools/csproj#runtimeidentifier) ) contiene il Framework di destinazione. Nell'esempio seguente il RID è impostato su `win7-x64`. La proprietà `<SelfContained>` è impostata su `false`. Queste proprietà indicano all'SDK di generare un file eseguibile (con estensione *exe*) per Windows e un'app che dipende dal framework .NET Core condiviso.

La proprietà `<UseAppHost>` è impostata su `true`. Questa proprietà fornisce il servizio con un percorso di attivazione (un file eseguibile, *.exe*) per una distribuzione dipendente dal framework.

Un file *web.config*, che viene normalmente generato quando si pubblica un'app ASP.NET Core, non è necessario per un'app di servizi Windows. Per disabilitare la creazione del file *web.config*, aggiungere la proprietà `<IsTransformWebConfigDisabled>` impostata su `true`.

```xml
<PropertyGroup>
  <TargetFramework>netcoreapp2.2</TargetFramework>
  <RuntimeIdentifier>win7-x64</RuntimeIdentifier>
  <UseAppHost>true</UseAppHost>
  <SelfContained>false</SelfContained>
  <IsTransformWebConfigDisabled>true</IsTransformWebConfigDisabled>
</PropertyGroup>
```

### <a name="self-contained-deployment-scd"></a>Distribuzione autonoma

Una distribuzione autonoma non si basa sulla presenza di un framework condiviso nel sistema host. Il runtime e le dipendenze dell'app vengono distribuiti con l'app.

Un [identificatore di runtime (RID)](/dotnet/core/rid-catalog) di Windows viene incluso nel `<PropertyGroup>` che contiene il framework di destinazione:

```xml
<RuntimeIdentifier>win7-x64</RuntimeIdentifier>
```

Per eseguire la pubblicazione per più identificatori di runtime:

* Specificare gli identificatori di runtime in un elenco delimitato da punto e virgola.
* Usare il nome della proprietà [\<RuntimeIdentifiers>](/dotnet/core/tools/csproj#runtimeidentifiers) (plurale).

Per altre informazioni, vedere il [Catalogo RID di .NET Core](/dotnet/core/rid-catalog).

Una proprietà `<SelfContained>` è impostata su `true`:

```xml
<SelfContained>true</SelfContained>
```

## <a name="service-user-account"></a>Account utente del servizio

Per creare un account utente per un servizio, usare il cmdlet [New-LocalUser](/powershell/module/microsoft.powershell.localaccounts/new-localuser) da una shell dei comandi di PowerShell 6 amministrativa.

In Aggiornamento di Windows 10 (ottobre 2018) (versione 1809/build 10.0.17763) o versioni successive:

```powershell
New-LocalUser -Name {SERVICE NAME}
```

In un sistema operativo Windows precedente ad Aggiornamento di Windows 10 (ottobre 2018) (versione 1809/build 10.0.17763):

```console
powershell -Command "New-LocalUser -Name {SERVICE NAME}"
```

Fornire una [password complessa](/windows/security/threat-protection/security-policy-settings/password-must-meet-complexity-requirements) quando richiesto.

Se non viene specificato il parametro `-AccountExpires` per il cmdlet [New-LocalUser](/powershell/module/microsoft.powershell.localaccounts/new-localuser) con un valore <xref:System.DateTime> di scadenza, l'account non scade.

Per altre informazioni, vedere [Microsoft.PowerShell.LocalAccounts](/powershell/module/microsoft.powershell.localaccounts/) e [Service User Accounts](/windows/desktop/services/service-user-accounts) (Account utente di servizio).

Un approccio alternativo per la gestione degli utenti quando si usa Active Directory consiste nell'usare account del servizio gestito. Per altre informazioni, vedere [Panoramica degli account del servizio gestito del gruppo](/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview).

## <a name="log-on-as-a-service-rights"></a>Diritti Accesso come servizio

Per stabilire i diritti *Accesso come servizio* per un account utente di servizio:

1. Aprire l'editor Criteri di sicurezza locali eseguendo *secpol.msc*.
1. Espandere il nodo **Criteri locali** e selezionare **Assegnazione diritti utente**.
1. Aprire il criterio **Accesso come servizio**.
1. Selezionare **Aggiungi utente o gruppo**.
1. Specificare il nome oggetto (account utente) in uno dei modi seguenti:
   1. Digitare l'account utente (`{DOMAIN OR COMPUTER NAME\USER}`) nel campo del nome oggetto e scegliere **OK** per aggiungere l'utente al criterio.
   1. Fare clic su **Advanced** (Avanzate). Selezionare **Trova**. Selezionare l'account utente dall'elenco. Selezionare **OK**. Scegliere di nuovo **OK** per aggiungere l'utente al criterio.
1. Scegliere **OK** o **Applica** per accettare le modifiche.

## <a name="create-and-manage-the-windows-service"></a>Creare e gestire il servizio di Windows

### <a name="create-a-service"></a>Creare un servizio

Usare i comandi di PowerShell per registrare un servizio. Da una shell dei comandi di PowerShell 6 amministrativa eseguire i comandi seguenti:

```powershell
$acl = Get-Acl "{EXE PATH}"
$aclRuleArgs = {DOMAIN OR COMPUTER NAME\USER}, "Read,Write,ReadAndExecute", "ContainerInherit,ObjectInherit", "None", "Allow"
$accessRule = New-Object System.Security.AccessControl.FileSystemAccessRule($aclRuleArgs)
$acl.SetAccessRule($accessRule)
$acl | Set-Acl "{EXE PATH}"

New-Service -Name {SERVICE NAME} -BinaryPathName {EXE FILE PATH} -Credential {DOMAIN OR COMPUTER NAME\USER} -Description "{DESCRIPTION}" -DisplayName "{DISPLAY NAME}" -StartupType Automatic
```

* `{EXE PATH}`: Percorso della cartella dell'app nell'host (ad esempio, `d:\myservice` ). Non includere l'eseguibile dell'app nel percorso. Non è necessario aggiungere una barra finale.
* `{DOMAIN OR COMPUTER NAME\USER}`: Account utente del servizio (ad esempio, `Contoso\ServiceUser` ).
* `{SERVICE NAME}`: Nome del servizio (ad esempio, `MyService` ).
* `{EXE FILE PATH}`: Percorso eseguibile dell'app (ad esempio, `d:\myservice\myservice.exe` ). Includere il nome del file eseguibile con l'estensione.
* `{DESCRIPTION}`: Descrizione del servizio (ad esempio, `My sample service` ).
* `{DISPLAY NAME}`: Nome visualizzato del servizio (ad esempio, `My Service` ).

### <a name="start-a-service"></a>Avviare un servizio

Per avviare un servizio, usare il comando di PowerShell 6 seguente:

```powershell
Start-Service -Name {SERVICE NAME}
```

L'avvio del servizio richiede alcuni secondi.

### <a name="determine-a-services-status"></a>Determinare lo stato di un servizio

Per verificare lo stato di un servizio, usare il comando di PowerShell 6 seguente:

```powershell
Get-Service -Name {SERVICE NAME}
```

Lo stato viene indicato con uno dei valori seguenti:

* `Starting`
* `Running`
* `Stopping`
* `Stopped`

### <a name="stop-a-service"></a>Arrestare un servizio

Per arrestare un servizio, usare il comando di PowerShell 6 seguente:

```powershell
Stop-Service -Name {SERVICE NAME}
```

### <a name="remove-a-service"></a>Rimuovere un servizio

Dopo il breve lasso di tempo necessario per arrestare un servizio, rimuovere il servizio con il comando di PowerShell 6 seguente:

```powershell
Remove-Service -Name {SERVICE NAME}
```

## <a name="handle-starting-and-stopping-events"></a>Gestire gli eventi di avvio e arresto

Per gestire gli eventi <xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostService.OnStarting*>, <xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostService.OnStarted*> e <xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostService.OnStopping*>:

1. Creare una classe che deriva da <xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostService> con i metodi `OnStarting`, `OnStarted` e `OnStopping`:

   [!code-csharp[](windows-service/samples/2.x/AspNetCoreService/CustomWebHostService.cs?name=snippet_CustomWebHostService)]

2. Creare un metodo di estensione per <xref:Microsoft.AspNetCore.Hosting.IWebHost> che passa `CustomWebHostService` a <xref:System.ServiceProcess.ServiceBase.Run*>:

   [!code-csharp[](windows-service/samples/2.x/AspNetCoreService/WebHostServiceExtensions.cs?name=ExtensionsClass)]

3. In `Program.Main` chiamare il metodo di estensione `RunAsCustomService` invece di <xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostWindowsServiceExtensions.RunAsService*>:

   ```csharp
   host.RunAsCustomService();
   ```

   Per visualizzare il percorso di <xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostWindowsServiceExtensions.RunAsService*> in `Program.Main`, fare riferimento all'esempio di codice illustrato nella sezione [Tipo di distribuzione](#deployment-type).

## <a name="proxy-server-and-load-balancer-scenarios"></a>Scenari con server proxy e servizi di bilanciamento del carico

I servizi che interagiscono con le richieste da Internet o da una rete aziendale e si trovano dietro un proxy o un servizio di bilanciamento del carico potrebbero richiedere una configurazione aggiuntiva. Per altre informazioni, vedere <xref:host-and-deploy/proxy-load-balancer>.

## <a name="configure-endpoints"></a>Configurare gli endpoint

Per impostazione predefinita, ASP.NET Core è associato a `http://localhost:5000`. Configurare l'URL e la porta impostando la `ASPNETCORE_URLS` variabile di ambiente.

Per ulteriori approcci alla configurazione di porte e URL, vedere l'articolo relativo al server pertinente:

* <xref:fundamentals/servers/kestrel#endpoint-configuration>
* <xref:fundamentals/servers/httpsys#configure-windows-server>

Le linee guida precedenti riguardano il supporto per gli endpoint HTTPS. Ad esempio, configurare l'app per HTTPS quando si usa l'autenticazione con un servizio Windows.

> [!NOTE]
> L'uso del certificato di sviluppo ASP.NET Core HTTPS per proteggere un endpoint del servizio non è supportato.

## <a name="current-directory-and-content-root"></a>Directory corrente e radice del contenuto

La directory di lavoro corrente restituita chiamando <xref:System.IO.Directory.GetCurrentDirectory*> per un servizio Windows è la cartella *C:\\WINDOWS\\system32*. La cartella *system32* non è un percorso appropriato per archiviare i file di un servizio, ad esempio i file di impostazioni. Usare uno degli approcci seguenti per gestire e accedere agli asset e ai file di impostazioni di un servizio.

### <a name="set-the-content-root-path-to-the-apps-folder"></a>Impostare il percorso radice del contenuto sulla cartella dell'app

<xref:Microsoft.Extensions.Hosting.IHostingEnvironment.ContentRootPath*> è lo stesso percorso fornito all'argomento `binPath` durante la creazione di un servizio. Anziché chiamare `GetCurrentDirectory` per creare percorsi per i file di impostazioni, chiamare <xref:System.IO.Directory.SetCurrentDirectory*> con il percorso della radice del [contenuto](xref:fundamentals/index#content-root)dell'app.

In `Program.Main`, determinare il percorso della cartella dell'eseguibile del servizio e usare il percorso per stabilire la radice del contenuto dell'app:

```csharp
var pathToExe = Process.GetCurrentProcess().MainModule.FileName;
var pathToContentRoot = Path.GetDirectoryName(pathToExe);
Directory.SetCurrentDirectory(pathToContentRoot);

CreateWebHostBuilder(args)
    .Build()
    .RunAsService();
```

### <a name="store-a-services-files-in-a-suitable-location-on-disk"></a>Archiviare i file di un servizio in un percorso appropriato nel disco

Specificare un percorso assoluto con <xref:Microsoft.Extensions.Configuration.FileConfigurationExtensions.SetBasePath*> quando si usa un <xref:Microsoft.Extensions.Configuration.IConfigurationBuilder> per la cartella contenente i file.

## <a name="troubleshoot"></a>Risolvere problemi

Per risolvere i problemi relativi a un'app di servizio Windows, vedere <xref:test/troubleshoot> .

### <a name="common-errors"></a>Errori comuni

* È in uso una versione precedente o provvisoria di PowerShell.
* Il servizio registrato non usa l'output **pubblicato** dell'app dal comando [DotNet Publish](/dotnet/core/tools/dotnet-publish) . L'output del comando [DotNet Build](/dotnet/core/tools/dotnet-build) non è supportato per la distribuzione di app. Gli asset pubblicati si trovano in una delle cartelle seguenti, a seconda del tipo di distribuzione:
  * *bin/Release/{Target Framework}/Publish* (FDD)
  * *bin/Release/{Target Framework}/{Runtime Identifier}/Publish* (SCD)
* Il servizio non è nello stato in esecuzione.
* I percorsi delle risorse utilizzate dall'app, ad esempio i certificati, non sono corretti. Il percorso di base di un servizio Windows è *c: \\ Windows \\ system32*.
* L'utente non dispone *di diritti di accesso come servizio* .
* La password dell'utente è scaduta o non è stata passata correttamente durante l'esecuzione del `New-Service` comando di PowerShell.
* L'app richiede l'autenticazione ASP.NET Core ma non è configurata per le connessioni protette (HTTPS).
* La porta dell'URL della richiesta non è corretta o non è configurata correttamente nell'app.

### <a name="system-and-application-event-logs"></a>Log eventi di sistema e dell'applicazione

Accedere ai registri eventi di sistema e dell'applicazione:

1. Aprire il menu Start, cercare *Visualizzatore eventi* e selezionare l'app **Visualizzatore eventi** .
1. In **Visualizzatore eventi** aprire il nodo **Registri di Windows**.
1. Selezionare **sistema** per aprire il registro eventi di sistema. Selezionare **Applicazione** per aprire il log eventi dell'applicazione.
1. Cercare gli errori associati all'app in cui si è verificato il problema.

### <a name="run-the-app-at-a-command-prompt"></a>Eseguire l'app da un prompt dei comandi

Molti errori di avvio non producono informazioni utili nei log eventi. È possibile individuare la causa di alcuni errori eseguendo l'app da un prompt dei comandi nel sistema host. Per registrare dettagli aggiuntivi dall'app, abbassare il [livello di registrazione](xref:fundamentals/logging/index#log-level) o eseguire l'app nell' [ambiente di sviluppo](xref:fundamentals/environments).

### <a name="clear-package-caches"></a>Cancella cache di pacchetti

Un'app funzionante potrebbe non riuscire immediatamente dopo l'aggiornamento del .NET Core SDK nel computer di sviluppo o la modifica delle versioni del pacchetto all'interno dell'app. In alcuni casi i pacchetti incoerenti possono interrompere un'app quando si eseguono aggiornamenti principali. La maggior parte di questi problemi può essere risolta attenendosi alle istruzioni seguenti:

1. Eliminare le cartelle *bin* e *obj*.
1. Cancellare le cache dei pacchetti eseguendo [le impostazioni locali di DotNet NuGet All--Clear](/dotnet/core/tools/dotnet-nuget-locals) da una shell dei comandi.

   La cancellazione delle cache dei pacchetti può essere eseguita anche con lo strumento [nuget.exe](https://www.nuget.org/downloads) e l'esecuzione del comando `nuget locals all -clear` . *nuget.exe* non è un'installazione inclusa con il sistema operativo desktop Windows e deve essere ottenuta separatamente dal [sito Web NuGet](https://www.nuget.org/downloads).

1. Ripristinare e ricompilare il progetto.
1. Eliminare tutti i file nella cartella di distribuzione nel server prima di ridistribuire l'app.

### <a name="slow-or-hanging-app"></a>App lenta o bloccata

Un *dump di arresto anomalo* del sistema è uno snapshot della memoria del sistema e può contribuire a determinare la provocazione di un arresto anomalo dell'app, dell'avvio o dell'applicazione lenta.

#### <a name="app-crashes-or-encounters-an-exception"></a>Arresto anomalo o eccezione di un'app

Ottenere e analizzare un dump da [Segnalazione errori Windows](/windows/desktop/wer/windows-error-reporting):

1. Creare una cartella per i file dump di arresto anomalo del sistema in `c:\dumps`.
1. Eseguire lo [script di PowerShell EnableDumps](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/host-and-deploy/windows-service/scripts/EnableDumps.ps1) con il nome dell'eseguibile dell'applicazione:

   ```console
   .\EnableDumps {APPLICATION EXE} c:\dumps
   ```

1. Eseguire l'app nelle condizioni che causano l'arresto anomalo.
1. Dopo che si è verificato l'arresto anomalo, eseguire lo [script di PowerShell DisableDumps](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/host-and-deploy/windows-service/scripts/DisableDumps.ps1):

   ```console
   .\DisableDumps {APPLICATION EXE}
   ```

Dopo l'arresto anomalo di un'app e la raccolta dei dump, l'app può terminare normalmente. Lo script di PowerShell configura Segnalazione errori Windows per raccogliere fino a cinque dump per ogni app.

> [!WARNING]
> I dump di arresto anomalo del sistema potrebbero richiedere una grande quantità di spazio su disco (fino a diversi gigabyte ognuno).

#### <a name="app-hangs-fails-during-startup-or-runs-normally"></a>L'app si blocca, si verifica un errore durante l'avvio o viene eseguita normalmente

Quando un'app si *blocca* (smette di rispondere ma non si arresta in modo anomalo), si verifica un errore durante l'avvio o viene eseguita normalmente, vedere [file di dump in modalità utente: scegliere lo strumento migliore](/windows-hardware/drivers/debugger/user-mode-dump-files#choosing-the-best-tool) per selezionare uno strumento appropriato per produrre il dump.

#### <a name="analyze-the-dump"></a>Analizzare il dump

È possibile analizzare un dump usando diversi approcci. Per altre informazioni, vedere [Analyzing a User-Mode Dump File](/windows-hardware/drivers/debugger/analyzing-a-user-mode-dump-file) (Analisi di un file dump in modalità utente).

## <a name="additional-resources"></a>Risorse aggiuntive

* [Configurazione dell'endpoint Kestrel](xref:fundamentals/servers/kestrel#endpoint-configuration) (include la configurazione HTTPS e il supporto SNI)
* <xref:fundamentals/host/web-host>
* <xref:test/troubleshoot>

::: moniker-end
