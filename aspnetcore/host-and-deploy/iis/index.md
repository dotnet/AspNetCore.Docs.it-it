---
title: Host ASP.NET Core in Windows con IIS
author: rick-anderson
description: Informazioni su come ospitare app ASP.NET Core in Windows Server Internet Information Services (IIS).
monikerRange: '>= aspnetcore-2.1'
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
uid: host-and-deploy/iis/index
ms.openlocfilehash: 13c4c2e2759355b10a4648b313f4dbed4b3aa482
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102588958"
---
# <a name="host-aspnet-core-on-windows-with-iis"></a>Host ASP.NET Core in Windows con IIS

::: moniker range=">= aspnetcore-5.0"

Internet Information Services (IIS) è un server Web flessibile, sicuro e gestibile per l'hosting di app Web, incluso ASP.NET Core.

## <a name="supported-platforms"></a>Piattaforme supportate

Sono supportati i sistemi operativi seguenti:

* Windows 7 o versione successiva
* Windows Server 2012 R2 o versione successiva

Sono supportate le app pubblicate per la distribuzione a 32 bit (x86) o a 64 bit (x64). Distribuire un'app a 32 bit con .NET Core SDK a 32 bit (x86), a meno che l'app:

* Non richieda lo spazio indirizzi di memoria virtuale più grande disponibile per un'app a 64 bit.
* Non richieda le dimensioni maggiori dello stack IIS.
* Non abbia dipendenze native a 64 bit.

## <a name="install-the-aspnet-core-modulehosting-bundle"></a>Installare il pacchetto di hosting/modulo ASP.NET Core

Scaricare il programma di installazione mediante il collegamento seguente:

[Programma di installazione del bundle di hosting .NET Core corrente (download diretto)](https://dotnet.microsoft.com/permalink/dotnetcore-current-windows-runtime-bundle-installer)

Per altre informazioni dettagliate su come installare il modulo ASP.NET Core o installare diverse versioni, vedere [Install the .NET Core hosting bundle](xref:host-and-deploy/iis/hosting-bundle).

## <a name="get-started"></a>Introduzione

Per informazioni introduttive sull'hosting di un sito Web in IIS, vedere la [Guida introduttiva](xref:tutorials/publish-to-iis).

Per informazioni introduttive sull'hosting di un sito Web in app Azure Services, vedere la pagina relativa [alla distribuzione in app Azure Service Guide](xref:host-and-deploy/azure-apps/index).

## <a name="deployment-resources-for-iis-administrators"></a>Risorse di distribuzione per amministratori IIS

* [Documentazione di ISS](/iis)
* [Getting Started with the IIS Manager in IIS](/iis/get-started/getting-started-with-iis/getting-started-with-the-iis-manager-in-iis-7-and-iis-8) (Introduzione a Gestione IIS in IIS)
* [Distribuzione di applicazioni .NET Core](/dotnet/core/deploying/)
* <xref:host-and-deploy/aspnet-core-module>
* <xref:host-and-deploy/directory-structure>
* <xref:host-and-deploy/iis/modules>
* <xref:test/troubleshoot-azure-iis>
* <xref:host-and-deploy/azure-iis-errors-reference>

## <a name="overlapped-recycle"></a>Riciclo sovrapposto

In generale, è consigliabile usare un modello come le [distribuzioni blu-verdi](https://www.martinfowler.com/bliki/BlueGreenDeployment.html) per le distribuzioni senza tempi di inattività. Funzionalità come la guida del riciclo sovrapposto, ma non garantiscono che sia possibile eseguire una distribuzione senza tempi di inattività. Per altre informazioni, vedere [questo problema di GitHub](https://github.com/dotnet/aspnetcore/issues/10117).

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:test/troubleshoot>
* <xref:index>
* [Il sito IIS ufficiale di Microsoft](https://www.iis.net/)
* [Libreria di contenuti tecnici di Windows Server](/windows-server/windows-server)
* [HTTP/2 on IIS](/iis/get-started/whats-new-in-iis-10/http2-on-iis) (HTTP/2 in IIS)
* <xref:host-and-deploy/iis/transform-webconfig>

::: moniker-end

::: moniker range=">= aspnetcore-3.0 < aspnetcore-5.0"

Per un'esercitazione sulla pubblicazione di un'app ASP.NET Core in un server IIS, vedere <xref:tutorials/publish-to-iis>.

[Installare il bundle di hosting .NET Core](#install-the-net-core-hosting-bundle)

## <a name="supported-operating-systems"></a>Sistemi operativi supportati

Sono supportati i sistemi operativi seguenti:

* Windows 7 o versione successiva
* Windows Server 2012 R2 o versione successiva

Il [server HTTP.sys](xref:fundamentals/servers/httpsys) (chiamato in precedenza WebListener) non funziona in una configurazione proxy inverso con IIS. È necessario usare il [server Kestrel](xref:fundamentals/servers/kestrel).

Per informazioni sull'hosting in Azure, vedere <xref:host-and-deploy/azure-apps/index>.

Per linee guida per la risoluzione dei problemi, vedere <xref:test/troubleshoot>.

## <a name="supported-platforms"></a>Piattaforme supportate

Sono supportate le app pubblicate per la distribuzione a 32 bit (x86) o a 64 bit (x64). Distribuire un'app a 32 bit con .NET Core SDK a 32 bit (x86), a meno che l'app:

* Non richieda lo spazio indirizzi di memoria virtuale più grande disponibile per un'app a 64 bit.
* Non richieda le dimensioni maggiori dello stack IIS.
* Non abbia dipendenze native a 64 bit.

Per le app pubblicate per 32 bit (x86) è necessario che sia abilitato 32 bit per i pool di applicazioni IIS. Per ulteriori informazioni, vedere la sezione [creare il sito IIS](#create-the-iis-site) .

Usare .NET Core SDK a 64 bit (x64) per pubblicare un'app a 64 bit. Un runtime a 64 bit deve essere presente nel sistema host.

## <a name="hosting-models"></a>Modelli di hosting

### <a name="in-process-hosting-model"></a>Modello di hosting in-process

Se si usa l'hosting in-process, un'app ASP.NET Core esegue lo stesso processo del processo di lavoro IIS. L'hosting in-process offre prestazioni migliori rispetto all'hosting out-of-process perché le richieste non vengono inserite in proxy sulla scheda di loopback, un'interfaccia di rete che restituisce il traffico di rete in uscita allo stesso computer. Per gestire il processo, IIS usa il [servizio Attivazione processo Windows (WAS)](/iis/manage/provisioning-and-managing-iis/features-of-the-windows-process-activation-service-was).

Il [modulo ASP.NET Core](xref:host-and-deploy/aspnet-core-module):

* Esegue l'inizializzazione dell'app.
  * Carica [CoreCLR](/dotnet/standard/glossary#coreclr).
  * Chiama `Program.Main`.
* Gestisce la durata della richiesta nativa di IIS.

Il diagramma seguente illustra la relazione tra IIS, il modulo ASP.NET Core e un'app ospitata in-process:

![Modulo ASP.NET Core nello scenario di hosting in-process](index/_static/ancm-inprocess.png)

1. Una richiesta arriva dal Web al driver HTTP.sys in modalità kernel.
1. Il driver instrada la richiesta nativa IIS sulla porta configurata per il sito Web, in genere 80 (HTTP) o 443 (HTTPS).
1. Il modulo ASP.NET Core riceve la richiesta nativa e la passa al server HTTP IIS ( `IISHttpServer` ). Il server HTTP di IIS è un'implementazione di server in-process per IIS che converte la richiesta da nativa a gestita.

Dopo l'elaborazione della richiesta da parte del server HTTP IIS:

1. La richiesta viene inviata alla pipeline del middleware ASP.NET Core.
1. La pipeline middleware gestisce la richiesta e la passa come istanza di `HttpContext` alla logica dell'app.
1. La risposta dell'app viene restituita a IIS tramite il server HTTP di IIS.
1. IIS invia la risposta al client che ha avviato la richiesta.

L'hosting in-process è il consenso esplicito per le app esistenti. I modelli Web ASP.NET Core utilizzano il modello di hosting in-process.

`CreateDefaultBuilder` aggiunge un' <xref:Microsoft.AspNetCore.Hosting.Server.IServer> istanza di chiamando il <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderIISExtensions.UseIIS%2A> metodo per avviare il [CoreCLR](/dotnet/standard/glossary#coreclr) e ospitare l'app all'interno del processo di lavoro IIS ( `w3wp.exe` o `iisexpress.exe` ). I test delle prestazioni indicano che l'hosting di un'app .NET Core in-process offre una velocità effettiva delle richieste significativamente superiore rispetto all'hosting dell'app out-of-process o all'inoltro delle richieste a [Kestrel](xref:fundamentals/servers/kestrel).

Le app pubblicate come un singolo file eseguibile non possono essere caricate dal modello di hosting in-process.

### <a name="out-of-process-hosting-model"></a>Modello di hosting out-of-process

Poiché le app ASP.NET Core vengono eseguite in un processo separato dal processo di lavoro IIS, il modulo ASP.NET Core gestisce la gestione dei processi. Il modulo avvia il processo per l'app ASP.NET Core quando arriva la prima richiesta e riavvia l'app se viene arrestata o si arresta in modo anomalo. Si tratta essenzialmente dello stesso comportamento delle app eseguite in-process e gestite dal [servizio Attivazione processo Windows](/iis/manage/provisioning-and-managing-iis/features-of-the-windows-process-activation-service-was).

Il diagramma seguente illustra la relazione tra IIS, il modulo ASP.NET Core e un'app ospitata out-of-process:

![Modulo ASP.NET Core nello scenario di hosting out-of-process](index/_static/ancm-outofprocess.png)

1. Le richieste arrivano dal Web al driver HTTP.sys in modalità kernel.
1. Il driver instrada le richieste a IIS sulla porta configurata del sito Web. La porta configurata è generalmente 80 (HTTP) o 443 (HTTPS).
1. Il modulo Invia le richieste a gheppio su una porta casuale per l'app. La porta casuale non è 80 o 443.

<!-- make this a bullet list -->
Il modulo ASP.NET Core specifica la porta tramite una variabile di ambiente all'avvio. L' <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderIISExtensions.UseIISIntegration%2A> estensione configura il server per l'ascolto `http://localhost:{PORT}` . Vengono eseguiti controlli aggiuntivi e le richieste che non provengono dal modulo vengono rifiutate. Il modulo non supporta l'invio HTTPS. Le richieste vengono inviate tramite HTTP anche se ricevute da IIS su HTTPS.

Quando il gheppio preleva la richiesta dal modulo, la richiesta viene trasmessa alla pipeline del middleware ASP.NET Core. La pipeline middleware gestisce la richiesta e la passa come istanza di `HttpContext` alla logica dell'app. Il middleware aggiunto dall'integrazione di IIS aggiorna lo schema, l'IP remoto e il percorso di base all'account per l'inoltro della richiesta a Kestrel. La risposta dell'app viene passata di nuovo a IIS, che la inoltra al client HTTP che ha avviato la richiesta.

Per indicazioni sulla configurazione del modulo ASP.NET Core, vedere <xref:host-and-deploy/aspnet-core-module>.

Per altre informazioni sull'hosting, vedere [Hosting in ASP.NET Core](xref:fundamentals/index#host).

## <a name="application-configuration"></a>Configurazione dell'applicazione

### <a name="enable-the-iisintegration-components"></a>Abilitare i componenti IISIntegration

Quando si compila un host in `CreateHostBuilder` ( `Program.cs` ), chiamare <xref:Microsoft.Extensions.Hosting.Host.CreateDefaultBuilder%2A> per abilitare l'integrazione con IIS:

```csharp
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        ...
```

Per altre informazioni su `CreateDefaultBuilder`, vedere <xref:fundamentals/host/generic-host#default-builder-settings>.

### <a name="iis-options"></a>Opzioni IIS

**Modello di hosting in-process**

Per configurare le opzioni del server IIS includere una configurazione del servizio per <xref:Microsoft.AspNetCore.Builder.IISServerOptions> in <xref:Microsoft.AspNetCore.Hosting.IStartup.ConfigureServices%2A>. L'esempio seguente disabilita AutomaticAuthentication:

```csharp
services.Configure<IISServerOptions>(options => 
{
    options.AutomaticAuthentication = false;
});
```

| Opzione                         | Predefinito | Impostazione |
| ------------------------------ | :-----: | ------- |
| `AutomaticAuthentication`      | `true`  | Se `true`, IIS Server imposta l'utente `HttpContext.User` autenticato tramite l'[autenticazione di Windows](xref:security/authentication/windowsauth). Se `false`, il server fornisce solo un'identità per `HttpContext.User` e risponde alle richieste esplicite di `AuthenticationScheme`. Per il funzionamento di `AutomaticAuthentication` l’autenticazione di Windows deve essere abilitata in IIS. Per altre informazioni, vedere [Autenticazione di Windows](xref:security/authentication/windowsauth). |
| `AuthenticationDisplayName`    | `null`  | Imposta il nome visualizzato dagli utenti nelle pagine di accesso. |
| `AllowSynchronousIO`           | `false` | Indica se l'i/O sincrono è consentito per `HttpContext.Request` e `HttpContext.Response` . |
| `MaxRequestBodySize`           | `30000000`  | Ottiene o imposta la dimensione massima del corpo della richiesta per `HttpRequest`. Notare che IIS stesso include il limite `maxAllowedContentLength`, che verrà elaborato prima del set `MaxRequestBodySize` in `IISServerOptions`. La modifica di `MaxRequestBodySize` non influisce su `maxAllowedContentLength`. Per aumentare `maxAllowedContentLength` , aggiungere una voce in `web.config` per impostare `maxAllowedContentLength` su un valore più alto. Per ulteriori informazioni, vedere [Configurazione](/iis/configuration/system.webServer/security/requestFiltering/requestLimits/#configuration). |

**Modello di hosting out-of-process**

Per configurare le opzioni di IIS includere una configurazione del servizio per <xref:Microsoft.AspNetCore.Builder.IISOptions> in <xref:Microsoft.AspNetCore.Hosting.IStartup.ConfigureServices%2A>. Nell'esempio seguente si impedisce all'app di popolare `HttpContext.Connection.ClientCertificate`:

```csharp
services.Configure<IISOptions>(options => 
{
    options.ForwardClientCertificate = false;
});
```

| Opzione                         | Predefinito | Impostazione |
| ------------------------------ | :-----: | ------- |
| `AutomaticAuthentication`      | `true`  | Se `true`, il [middleware di integrazione IIS](#enable-the-iisintegration-components) imposta `HttpContext.User` autenticato tramite l'[autenticazione di Windows](xref:security/authentication/windowsauth). Se `false`, il middleware fornisce solo un'identità per `HttpContext.User` e risponde solo alle richieste esplicite di `AuthenticationScheme`. Per il funzionamento di `AutomaticAuthentication` l’autenticazione di Windows deve essere abilitata in IIS. Per altre informazioni, vedere l'argomento [Autenticazione di Windows](xref:security/authentication/windowsauth). |
| `AuthenticationDisplayName`    | `null`  | Imposta il nome visualizzato dagli utenti nelle pagine di accesso. |
| `ForwardClientCertificate`     | `true`  | Se è `true` ed è presente l’intestazione della richiesta `MS-ASPNETCORE-CLIENTCERT`, `HttpContext.Connection.ClientCertificate` viene popolato. |

### <a name="proxy-server-and-load-balancer-scenarios"></a>Scenari con server proxy e servizi di bilanciamento del carico

Il [middleware di integrazione IIS](#enable-the-iisintegration-components) e il modulo ASP.NET Core sono configurati per l'invio di:

* Schema (HTTP/HTTPS).
* Indirizzo IP remoto da cui ha avuto origine la richiesta.

Il [middleware di integrazione di IIS](#enable-the-iisintegration-components) configura il middleware delle intestazioni con inoltri.

Potrebbero essere necessari interventi di configurazione aggiuntivi per le app ospitate dietro ulteriori server proxy e servizi di bilanciamento del carico. Per altre informazioni, vedere [Configurare ASP.NET Core per l'utilizzo di server proxy e servizi di bilanciamento del carico](xref:host-and-deploy/proxy-load-balancer).

### <a name="webconfig-file"></a>File `web.config`

Il `web.config` file configura il [modulo ASP.NET Core](xref:host-and-deploy/aspnet-core-module). La creazione, la trasformazione e la pubblicazione del `web.config` file vengono gestite da una destinazione MSBuild ( `_TransformWebConfig` ) quando viene pubblicato il progetto. Questa destinazione si trova tra le destinazioni Web SDK (`Microsoft.NET.Sdk.Web`). L'SDK è impostato all'inizio del file di progetto:

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
```

Se un `web.config` file non è presente nel progetto, il file viene creato con il corretto `processPath` e `arguments` per configurare il modulo ASP.NET Core e spostato nell' [output pubblicato](xref:host-and-deploy/directory-structure).

Se un `web.config` file è presente nel progetto, il file viene trasformato con il corretto `processPath` e `arguments` per configurare il modulo ASP.NET Core e spostato nell'output pubblicato. La trasformazione non modifica le impostazioni di configurazione di IIS incluse nel file.

Il `web.config` file può fornire impostazioni di configurazione di IIS aggiuntive che controllano i moduli IIS attivi. Per informazioni sui moduli IIS in grado di elaborare le richieste con le app di ASP.NET Core, vedere l'argomento [Moduli IIS](xref:host-and-deploy/iis/modules).

Per impedire a Web SDK di trasformare il `web.config` file, usare la `<IsTransformWebConfigDisabled>` proprietà nel file di progetto:

```xml
<PropertyGroup>
  <IsTransformWebConfigDisabled>true</IsTransformWebConfigDisabled>
</PropertyGroup>
```

Quando si disabilita il Web SDK dalla trasformazione del file, `processPath` `arguments` è necessario impostare manualmente lo sviluppatore. Per altre informazioni, vedere <xref:host-and-deploy/aspnet-core-module>.

### <a name="webconfig-file-location"></a>`web.config` percorso file

Per configurare correttamente il [modulo ASP.NET Core](xref:host-and-deploy/aspnet-core-module) , il `web.config` file deve essere presente nel percorso [radice del contenuto](xref:fundamentals/index#content-root) (in genere il percorso di base dell'app) dell'app distribuita. Corrisponde al percorso fisico del sito Web fornito a IIS. Il `web.config` file è necessario alla radice dell'app per abilitare la pubblicazione di più app usando distribuzione Web.

I file sensibili sono presenti nel percorso fisico dell'app, ad esempio `{ASSEMBLY}.runtimeconfig.json` , `{ASSEMBLY}.xml` (commenti in formato documentazione XML) e `{ASSEMBLY}.deps.json` , dove il segnaposto `{ASSEMBLY}` è il nome dell'assembly. Quando il `web.config` file è presente e il sito viene avviato normalmente, in IIS questi file riservati non vengono utilizzati se richiesti. Se il `web.config` file è mancante, denominato in modo non corretto o non è in grado di configurare il sito per l'avvio normale, è possibile che in IIS vengano utilizzati pubblicamente file riservati.

**Il `web.config` file deve essere presente nella distribuzione in qualsiasi momento, denominato correttamente e in grado di configurare il sito per l'avvio normale. Non rimuovere mai il `web.config` file da una distribuzione di produzione.**

### <a name="transform-webconfig"></a>Trasformare web.config

Se è necessario eseguire la trasformazione `web.config` alla pubblicazione, vedere <xref:host-and-deploy/iis/transform-webconfig> . Potrebbe essere necessario trasformare `web.config` in pubblicazione per impostare le variabili di ambiente in base alla configurazione, al profilo o all'ambiente.

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

1. Passare a **Pannello di controllo** > **Programmi** > **Programmi e funzionalità** > **Disattivare o attivare le funzionalità di Windows** (a sinistra dello schermo).

1. Aprire il nodo **Internet Information Services**. Aprire il nodo **Strumenti di gestione Web**.

1. Selezionare la casella per **Console di gestione IIS**.

1. Selezionare la casella per **Servizi World Wide Web**.

1. Accettare le funzionalità predefinite per **Servizi World Wide Web** o personalizzare le funzionalità IIS.

   **Autenticazione di Windows (facoltativa)**  
   Per abilitare l'autenticazione di Windows, espandere i nodi seguenti: **World Wide Web sicurezza dei servizi**  >  . Selezionare la funzionalità **Autenticazione di Windows**. Per ulteriori informazioni, vedere [ \<windowsAuthentication> autenticazione di Windows](/iis/configuration/system.webServer/security/authentication/windowsAuthentication/) e [configurare l'autenticazione di Windows](xref:security/authentication/windowsauth).

   **WebSockets (facoltativo)**  
   WebSockets è supportato con ASP.NET Core 1.1 o versioni successive. Per abilitare i WebSocket, espandere i nodi seguenti: funzionalità di sviluppo di applicazioni di **World Wide Web Services**  >  . Selezionare la funzionalità **Protocollo WebSocket**. Per altre informazioni, vedere [Oggetti WebSocket](xref:fundamentals/websockets).

1. Se l'installazione di IIS richiede un riavvio, riavviare il sistema.

![La console di gestione IIS e servizi World Wide Web sono selezionati nelle funzionalità di Windows.](index/_static/windows-features-win10.png)

## <a name="install-the-net-core-hosting-bundle"></a>Installare il bundle di hosting .NET Core

Installare il *bundle di hosting .NET Core* nel sistema di hosting. Il bundle installa il runtime di .NET Core, la libreria di .NET Core e il [modulo ASP.NET Core](xref:host-and-deploy/aspnet-core-module). Il modulo consente l'esecuzione delle app ASP.NET Core dietro IIS.

> [!IMPORTANT]
> Se il bundle di hosting viene installato prima di IIS, è necessario riparare l'installazione del bundle. Eseguire di nuovo il programma di installazione del bundle di hosting dopo l'installazione di IIS.
>
> Se il bundle di hosting viene installato dopo l'installazione della versione a 64 bit (x64) di .NET Core, uno o più SDK potrebbero risultare mancanti ([Non sono stati rilevati .NET Core SDK](xref:test/troubleshoot#no-net-core-sdks-were-detected)). Per risolvere il problema, vedere <xref:test/troubleshoot#missing-sdk-after-installing-the-net-core-hosting-bundle>.

### <a name="direct-download-current-version"></a>Download diretto (versione corrente)

Scaricare il programma di installazione mediante il collegamento seguente:

[Programma di installazione del bundle di hosting .NET Core corrente (download diretto)](https://dotnet.microsoft.com/permalink/dotnetcore-current-windows-runtime-bundle-installer)

### <a name="earlier-versions-of-the-installer"></a>Versioni precedenti del programma di installazione

Per ottenere una versione precedente del programma di installazione:

1. Passare alla pagina [download di .NET Core](https://dotnet.microsoft.com/download/dotnet-core) .
1. Selezionare la versione desiderata di .NET Core.
1. Nella colonna **Run apps - Runtime** (App di esecuzione - Runtime), trovare la riga della versione di runtime di .NET Core desiderata.
1. Scaricare il programma di installazione usando il collegamento **bundle di hosting** .

> [!WARNING]
> Alcuni programmi di installazione contengono versioni che hanno terminato il loro ciclo di vita e non sono più supportate da Microsoft. Per altre informazioni, vedere i [criteri di supporto](https://dotnet.microsoft.com/platform/support/policy/dotnet-core).

### <a name="install-the-hosting-bundle"></a>Installare il bundle di hosting

1. Eseguire il programma di installazione nel server. Quando si esegue il programma di installazione da una shell dei comandi di amministratore sono disponibili i parametri seguenti:

   * `OPT_NO_ANCM=1`: Ignorare l'installazione del modulo ASP.NET Core.
   * `OPT_NO_RUNTIME=1`: Ignorare l'installazione del runtime di .NET Core. Utilizzato quando il server ospita solo le [distribuzioni autosufficienti (SCD)](/dotnet/core/deploying/#self-contained-deployments-scd).
   * `OPT_NO_SHAREDFX=1`: Ignora l'installazione del Framework condiviso ASP.NET (runtime ASP.NET). Utilizzato quando il server ospita solo le [distribuzioni autosufficienti (SCD)](/dotnet/core/deploying/#self-contained-deployments-scd).
   * `OPT_NO_X86=1`: Ignora l'installazione di Runtime x86. Usare questo parametro se si è certi che non verrà eseguito l'hosting di app a 32 bit. Se non esiste alcuna possibilità che in futuro venga eseguito l'hosting di app sia a 32 che a 64 bit, non usare questo parametro e installare entrambi i runtime.
   * `OPT_NO_SHARED_CONFIG_CHECK=1`: Disabilitare il controllo per l'utilizzo di una configurazione condivisa di IIS quando la configurazione condivisa ( `applicationHost.config` ) si trova nello stesso computer dell'installazione di IIS. *Disponibile solo per i programmi di installazione di bundler di hosting ASP.NET Core 2.2 o versioni successive.* Per altre informazioni, vedere <xref:host-and-deploy/aspnet-core-module#aspnet-core-module-with-an-iis-shared-configuration>.
1. Riavviare il sistema o eseguire i comandi seguenti in una shell dei comandi:

   ```console
   net stop was /y
   net start w3svc
   ```
   Il riavvio di IIS rileva una modifica alla variabile di ambiente di sistema PATH apportata dal programma di installazione.

ASP.NET Core non adotta il comportamento di rollforward per le versioni di patch dei pacchetti di Framework condivisi. Dopo l'aggiornamento del Framework condiviso mediante l'installazione di un nuovo bundle di hosting, riavviare il sistema o eseguire i comandi seguenti in una shell dei comandi:

```console
net stop was /y
net start w3svc
```

> [!NOTE]
> Per informazioni sulla configurazione condivisa di IIS, vedere [Modulo di ASP.NET Core con configurazione condivisa di IIS](xref:host-and-deploy/aspnet-core-module#aspnet-core-module-with-an-iis-shared-configuration).

## <a name="install-web-deploy-when-publishing-with-visual-studio"></a>Installare la Distribuzione Web quando si pubblica con Visual Studio

Quando si distribuiscono app ai server con [Distribuzione Web](/iis/install/installing-publishing-technologies/installing-and-configuring-web-deploy-on-iis-80-or-later), installare la versione più recente di Distribuzione Web nel server. Per installare Distribuzione Web usare l'[Installazione guidata piattaforma Web (WebPI)](https://www.microsoft.com/web/downloads/platform.aspx) o ottenere direttamente un programma di installazione in [Microsoft Download Center](https://www.microsoft.com/download/details.aspx?id=43717). Il metodo preferito consiste nell'usare WebPI. WebPI offre un'installazione autonoma e una configurazione per i provider di hosting.

## <a name="create-the-iis-site"></a>Creare il sito IIS

1. Nel sistema host creare una cartella per contenere le cartelle e i file pubblicati dell'app. In un passaggio successivo, il percorso della cartella viene fornito a IIS come percorso fisico dell'app. Per altre informazioni sulla cartella di distribuzione e il layout dei file di un'app, vedere <xref:host-and-deploy/directory-structure>.

1. In Gestione IIS aprire il nodo del server nel pannello **connessioni** . Fare clic con il pulsante destro del mouse sulla cartella **Siti**. Scegliere **Aggiungi sito Web** dal menu di scelta rapida.

1. Specificare un **Nome del sito** e impostare il **Percorso fisico** per la cartella di distribuzione dell'app. Specificare la configurazione dell' **associazione** e creare il sito Web selezionando **OK**:

   ![Specificare il nome del sito, il percorso fisico e il nome Host nel passaggio Aggiungi sito Web.](index/_static/add-website-ws2016.png)

   > [!WARNING]
   > Le associazioni con caratteri jolly di livello superiore (`http://*:80/` e `http://+:80`) **non** devono essere usate, poiché possono introdurre vulnerabilità a livello di sicurezza nell'app. Questo concetto vale sia per i caratteri jolly sicuri che vulnerabili. Usare nomi host espliciti al posto di caratteri jolly. L'associazione con caratteri jolly del sottodominio (ad esempio, `*.mysub.com`) non costituisce un rischio per la sicurezza se viene controllato l'intero dominio padre (a differenza di `*.com`, che è vulnerabile). Vedere la [sezione 5.4 di RFC7230](https://tools.ietf.org/html/rfc7230#section-5.4) per altre informazioni.

1. Nel nodo del server selezionare **Pool di applicazioni**.

1. Fare clic con il pulsante destro del mouse sul pool di applicazioni del sito e scegliere **Impostazioni di base** dal menu di scelta rapida.

1. Nella finestra **Modifica pool di applicazioni** impostare **Versione .NET CLR** su **Nessun codice gestito**.

   ![Impostare Nessun codice gestito per la versione .NET CLR.](index/_static/edit-apppool-ws2016.png)

    ASP.NET Core viene eseguito in un processo separato e gestisce il runtime. ASP.NET Core non si basa sul caricamento del CLR desktop (CLR .NET). Il core Common Language Runtime (CoreCLR) per .NET Core viene avviato per ospitare l'app nel processo di lavoro. L'impostazione di **Versione .NET CLR** su **Nessun codice gestito** è facoltativa ma consigliata.

1. *ASP.NET Core 2.2 o versione successiva*:

   * Per una [distribuzione autonoma](/dotnet/core/deploying/#self-contained-deployments-scd) a 32 bit (x86) pubblicata con un SDK a 32 bit che usa il [modello di hosting in-process](#in-process-hosting-model), abilitare il Pool di applicazioni per 32 bit. In Gestione IIS passare a **pool di applicazioni** nella barra laterale **connessioni** . Selezionare il pool di applicazioni dell'app. Nella barra laterale **azioni** selezionare **Impostazioni avanzate**. Impostare **Abilita applicazioni a 32 bit** su `True` . 

   * per una [distribuzione autonoma](/dotnet/core/deploying/#self-contained-deployments-scd) a 64 bit (x64) che usa il [modello di hosting in-process](#in-process-hosting-model), disabilitare il pool di app per i processi a 32 bit (x86). In Gestione IIS passare a **pool di applicazioni** nella barra laterale **connessioni** . Selezionare il pool di applicazioni dell'app. Nella barra laterale **azioni** selezionare **Impostazioni avanzate**. Impostare **Abilita applicazioni a 32 bit** su `False` . 

1. Confermare che l'identità del modello del processo disponga delle autorizzazioni appropriate.

   Se l'identità predefinita del pool di app (**modello**  >  **Identity** di processo) viene modificata **da Identity ApplicationPool** a un'altra identità, verificare che la nuova identità disponga delle autorizzazioni necessarie per accedere alla cartella dell'app, al database e ad altre risorse richieste. Ad esempio, il pool di applicazioni richiede l'accesso in lettura e scrittura alle cartelle in cui l'app legge e scrive i file.

**Configurazione dell'autenticazione di Windows (facoltativa)**  
Per altre informazioni, vedere [Configurare l'autenticazione Windows](xref:security/authentication/windowsauth).

## <a name="deploy-the-app"></a>Distribuire l'app

Distribuire l'app nella cartella **Percorso fisico** di IIS specificata nella sezione [Creare il sito IIS](#create-the-iis-site). [Distribuzione Web](/iis/publish/using-web-deploy/introduction-to-web-deploy) è il meccanismo consigliato per la distribuzione, ma esistono diverse opzioni per lo sviluppo di app dalla cartella del progetto `publish` alla cartella di distribuzione del sistema di hosting.

### <a name="web-deploy-with-visual-studio"></a>Distribuzione web con Visual Studio

Vedere l'argomento [Visual Studio publish profiles for ASP.NET Core app deployment](xref:host-and-deploy/visual-studio-publish-profiles#publish-profiles) (Profili di pubblicazione Visual Studio per la distribuzione di app ASP.NET Core) per informazioni su come creare un profilo di pubblicazione da usare con Distribuzione Web. Se il provider di hosting fornisce un profilo di pubblicazione o il supporto per crearne uno, scaricare il profilo e importarlo usando la finestra di dialogo **Pubblica** di Visual Studio:

![Pagina della finestra di dialogo Pubblica](index/_static/pub-dialog.png)

### <a name="web-deploy-outside-of-visual-studio"></a>Distribuzione Web all'esterno di Visual Studio

È anche possibile usare [Distribuzione Web](/iis/publish/using-web-deploy/introduction-to-web-deploy) all'esterno di Visual Studio dalla riga di comando. Per altre informazioni sulla distribuzione, vedere [Strumento Distribuzione Web](/iis/publish/using-web-deploy/use-the-web-deployment-tool).

### <a name="alternatives-to-web-deploy"></a>Alternative a Distribuzione web

Usare uno dei metodi disponibili, ad esempio la copia manuale, [Xcopy](/windows-server/administration/windows-commands/xcopy), [Robocopy](/windows-server/administration/windows-commands/robocopy) o [PowerShell](/powershell/), per spostare l'app nel sistema di hosting.

Per altre informazioni sulla distribuzione di ASP.NET Core in IIS, vedere la sezione [Risorse di distribuzione per amministratori IIS](#deployment-resources-for-iis-administrators).

## <a name="browse-the-website"></a>Esplorare il sito Web

Dopo aver distribuito l'app nel sistema di hosting, effettuare una richiesta a uno degli endpoint pubblici dell'app.

Nell'esempio seguente il sito è associato al **nome host** di IIS `www.mysite.com` sulla **porta** `80`. Viene effettuata una richiesta a `http://www.mysite.com`:

![Il browser Microsoft Edge ha caricato la pagina di avvio IIS.](index/_static/browsewebsite.png)

## <a name="locked-deployment-files"></a>File di distribuzione bloccati

I file nella cartella di distribuzione sono bloccati durante l'esecuzione dell'app. I file bloccati non possono essere sovrascritti durante la distribuzione. Per rilasciare i file bloccati in una distribuzione, arrestare il pool di applicazioni usando **uno** dei metodi seguenti:

* Usare Distribuzione Web e fare riferimento a `Microsoft.NET.Sdk.Web` nel file di progetto. Un `app_offline.htm` file viene inserito alla radice della directory dell'app Web. Quando il file è presente, il modulo ASP.NET Core arresta normalmente l'app e serve il `app_offline.htm` file durante la distribuzione. Per altre informazioni, vedere la [Guida di riferimento per la configurazione del modulo ASP.NET Core](xref:host-and-deploy/aspnet-core-module#app_offlinehtm).
* Arrestare manualmente il pool di applicazioni in Gestione IIS sul server.
* Usare PowerShell per eliminare `app_offline.htm` (richiede PowerShell 5 o versione successiva):

  ```powershell
  $pathToApp = 'PATH_TO_APP'

  # Stop the AppPool
  New-Item -Path $pathToApp app_offline.htm

  # Provide script commands here to deploy the app

  # Restart the AppPool
  Remove-Item -Path $pathToApp app_offline.htm
  ```

## <a name="data-protection"></a>Protezione dei dati

Lo [stack di protezione dei dati di ASP.NET Core](xref:security/data-protection/introduction) viene usato da diversi [middleware](xref:fundamentals/middleware/index) di ASP.NET Core, inclusi quelli usati nell'autenticazione. Anche se le DPAPI (Data Protection API) non vengono chiamate dal codice dell'utente, è consigliabile configurare la protezione dati per la creazione di un [archivio di chiavi](xref:security/data-protection/implementation/key-management) crittografiche permanente con uno script di distribuzione o nel codice dell'utente. Se non si configura la protezione dei dati, le chiavi vengono mantenute in memoria ed eliminate al riavvio dell'app.

Se il gruppo di chiavi viene archiviato in memoria quando l'app viene riavviata:

* Tutti i cookie token di autenticazione basati su vengono invalidati. 
* Gli utenti devono ripetere l'accesso alla richiesta successiva. 
* Tutti i dati protetti con il gruppo di chiavi non possono più essere decrittografati. Questo può includere [i token CSRF](xref:security/anti-request-forgery#aspnet-core-antiforgery-configuration) e [ASP.NET Core TempData MVC cookie ](xref:fundamentals/app-state#tempdata).

Per configurare la protezione dati in IIS in modo da rendere permanente il gruppo di chiavi, usare **uno** degli approcci seguenti:

* **Creare chiavi del Registro di sistema di protezione dati**

  Le chiavi di protezione dati usate dalle app ASP.NET Core vengono archiviate nel Registro di sistema esterno alle app. Per mantenere le chiavi per una determinata app, creare chiavi del Registro di sistema per il pool di applicazioni.

  Per le installazioni IIS autonome non web farm è possibile usare lo [script PowerShell Data Protection Provision-AutoGenKeys.ps1 (Provisioning di protezione dati-AutoGenKeys.ps1)](https://github.com/dotnet/AspNetCore/blob/main/src/DataProtection/Provision-AutoGenKeys.ps1) per ogni pool di app usato con un'app ASP.NET Core. Questo script crea una chiave del Registro di sistema nel registro HKLM accessibile solo all'account del processo di lavoro del pool di applicazioni dell'app. Le chiavi vengono crittografate a riposo tramite DPAPI con una chiave a livello del computer.

  In scenari di web farm un'app può essere configurata in modo da usare un percorso UNC in cui archiviare il proprio gruppo di chiavi di protezione dati. Per impostazione predefinita, le chiavi di protezione dati non vengono crittografate. Assicurarsi che le autorizzazioni file per la condivisione di rete siano limitate all'account di Windows in cui viene eseguita l'app. È possibile usare un certificato X509 per la protezione delle chiavi inattive. Considerare l'implementazione di un meccanismo per consentire agli utenti di caricare i certificati: inserire i certificati nell'archivio certificati attendibili dell'utente e assicurarsi che siano disponibili in tutti i computer in cui viene eseguita l'app dell'utente. Vedere [Configurare la protezione dati di ASP.NET Core](xref:security/data-protection/configuration/overview) per altri dettagli.

* **Configurare il pool di applicazioni IIS per caricare il profilo utente**

  Questa impostazione si trova nella sezione **Modello del processo** in **Impostazioni avanzate** per il pool di app. Impostare **Caricamento del profilo utente** su `True`. Con l'impostazione `True` le chiavi vengono archiviate nella directory del profilo utente e protette tramite DPAPI con una chiave specifica per l'account utente. Le chiavi vengono salvate in modo permanente nella cartella *%LOCALAPPDATA%/ASP.NET/DataProtection-Keys*.

  Deve essere abilitato anche l'[attributo setProfileEnvironment](/iis/configuration/system.applicationhost/applicationpools/add/processmodel#configuration) del pool di app. Il valore predefinito di `setProfileEnvironment` è `true`. In alcuni scenari (ad esempio, per il sistema operativo Windows), `setProfileEnvironment` è impostato su `false`. Se le chiavi non vengono archiviate nella directory del profilo utente come previsto:

  1. Passare alla cartella *%windir%/system32/inetsrv/config*.
  1. Aprire il file *applicationHost.config*.
  1. Individuare l'elemento `<system.applicationHost><applicationPools><applicationPoolDefaults><processModel>`.
  1. Verificare che l'attributo `setProfileEnvironment` non sia presente, condizione che corrisponde all'impostazione predefinita `true`, o impostare in modo esplicito il valore dell'attributo su `true`.

* **Usare il file system come archivio del gruppo di chiavi**

  Modificare il codice dell'app per [usare il file system come archivio del gruppo di chiavi](xref:security/data-protection/configuration/overview). Usare un certificato X509 per proteggere il gruppo di chiavi e verificare che sia un certificato attendibile. Se il certificato è autofirmato, inserirlo nell'archivio radice attendibile.

  Quando si usa IIS in una web farm:

  * Usare una condivisione di file a cui possono accedere tutti i computer.
  * Distribuire un certificato X509 in ogni computer. Configurare [protezione dei dati nel codice](xref:security/data-protection/configuration/overview).

* **Impostare criteri a livello di computer per la protezione dei dati**

  Il sistema di protezione dei dati ha un supporto limitato per l'impostazione di [criteri a livello di computer](xref:security/data-protection/configuration/machine-wide-policy) predefiniti per tutte le app che usano le API di protezione dei dati. Per altre informazioni, vedere <xref:security/data-protection/introduction>.

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

## <a name="configuration-of-iis-with-webconfig"></a>Configurazione di IIS con web.config

La configurazione di IIS è influenzata dalla sezione `<system.webServer>` di *web.config* per gli scenari IIS funzionali per le app ASP.NET Core con il modulo ASP.NET Core. Ad esempio, la configurazione di IIS è funzionale per la compressione dinamica. Se IIS è configurato a livello di server per l'uso della compressione dinamica, l'elemento `<urlCompression>` nel file *web.config* dell'app può disabilitare questa impostazione per un'app ASP.NET Core.

Per altre informazioni, vedere i seguenti argomenti:

* [Riferimento alla configurazione per \<system.webServer>](/iis/configuration/system.webServer/)
* <xref:host-and-deploy/aspnet-core-module>
* <xref:host-and-deploy/iis/modules>

Per impostare le variabili di ambiente per singole app in esecuzione in pool di applicazioni isolate (supportate per IIS 10,0 o versioni successive), vedere la sezione *comandoAppCmd.exe* dell'argomento [variabili \<environmentVariables> di ambiente](/iis/configuration/system.applicationHost/applicationPools/add/environmentVariables/#appcmdexe) nella documentazione di riferimento di IIS.

## <a name="configuration-sections-of-webconfig"></a>Sezioni di configurazione di web.config

Le sezioni di configurazione di app ASP.NET 4.x in *web.config* non vengono usate dalle app ASP.NET Core per la configurazione:

* `<system.web>`
* `<appSettings>`
* `<connectionStrings>`
* `<location>`

Le applicazioni ASP.NET Core vengono configurate tramite altri provider di configurazione. Per ulteriori informazioni, vedere [configurazione](xref:fundamentals/configuration/index).

## <a name="application-pools"></a>Pool di applicazioni

L'isolamento dei pool di app è determinato dal modello di hosting:

* Hosting in-process: le app sono necessarie per l'esecuzione in pool di applicazioni separati.
* Hosting out-of-process: è consigliabile isolare le app le une dalle altre eseguendo ogni app nel proprio pool di applicazioni.

Nella finestra di dialogo **Aggiungi sito Web** è selezionato per impostazione predefinita un singolo pool di app per ogni app. Quando si specifica un valore in **Nome del sito**, il testo viene automaticamente trasferito alla casella di testo **Pool di applicazioni**. Quando si aggiunge il sito viene creato un nuovo pool di applicazioni con il nome del sito.

## <a name="application-pool-identity"></a>Pool di applicazioni Identity

Un account di identità del pool di app consente di eseguire un'app in un account univoco, senza dover creare e gestire domini o account locali. In IIS 8.0 o versioni successive il processo di lavoro amministrazione IIS (WAS) crea un account virtuale con il nome del nuovo pool di applicazioni ed esegue i processi di lavoro del pool di applicazioni inclusi nell'account per impostazione predefinita. Nella console di gestione IIS in **Impostazioni avanzate** per il pool di applicazioni, verificare che **Identity** sia impostato per utilizzare **ApplicationPool Identity**:

![Finestra di dialogo Impostazioni avanzate del pool di applicazione](index/_static/apppool-identity.png)

Il processo di gestione IIS crea un identificatore sicuro con il nome del pool di app nel sistema di sicurezza di Windows. Le risorse possono essere protette mediante questa identità, che tuttavia non è un account utente reale e non viene visualizzata nella console di gestione utenti di Windows.

Se il processo di lavoro IIS richiede l'accesso con privilegi elevati all'app, modificare l'elenco di controllo di accesso (ACL) della directory contenente l'app:

1. Aprire Esplora risorse, quindi spostarsi nella directory.

1. Fare clic con il pulsante destro del mouse sulla directory e scegliere **Proprietà**.

1. Nella scheda **Sicurezza** selezionare il pulsante **Modifica** e quindi il pulsante **Aggiungi**.

1. Fare clic sul pulsante **Percorsi** e verificare che il sistema sia selezionato.

1. Immettere `IIS AppPool\{APP POOL NAME}` , dove il segnaposto `{APP POOL NAME}` è il nome del pool di applicazioni, nell'area **immettere i nomi degli oggetti da selezionare** . Fare clic sul pulsante **Controlla nomi**. Per *DefaultAppPool* controllare i nomi usando `IIS AppPool\DefaultAppPool` . Quando si seleziona il pulsante **Controlla nomi** , il valore `DefaultAppPool` è indicato nell'area dei nomi degli oggetti. Non è possibile immettere il nome del pool di applicazioni direttamente nell'area dei nomi degli oggetti. Usare il `IIS AppPool\{APP POOL NAME}` formato, dove il segnaposto `{APP POOL NAME}` è il nome del pool di applicazioni, quando si verifica il nome dell'oggetto.

   ![Finestra di dialogo Seleziona utenti o gruppi per la cartella dell'app: il nome del pool di applicazioni "DefaultAppPool" viene aggiunto in coda a "IIS AppPool\" nell'area dei nomi degli oggetti prima di selezionare "Controlla nomi".](index/_static/select-users-or-groups-1.png)

1. Selezionare **OK**.

   ![Finestra di dialogo Seleziona utenti o gruppi per la cartella dell'app: dopo aver selezionato "Controlla nomi", il nome dell'oggetto "DefaultAppPool" viene visualizzato nell'area dei nomi degli oggetti.](index/_static/select-users-or-groups-2.png)

1. Le autorizzazioni di lettura ed esecuzione devono essere concesse per impostazione predefinita. Fornire autorizzazioni aggiuntive in base alle necessità.

L'accesso può essere concesso anche dal prompt dei comandi usando lo strumento **ICACLS**. Utilizzando `DefaultAppPool` come esempio, viene utilizzato il seguente comando:

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

* [Modulo di inizializzazione dell'applicazione](#application-initialization-module): l'app è ospitata [in-process](#in-process-hosting-model) o [out-of-process](#out-of-process-hosting-model) può essere configurata per l'avvio automatico in un processo di lavoro o il riavvio del server.
* [Timeout di inattività](#idle-timeout): le app ospitate [in-process](#in-process-hosting-model) possono essere configurate in modo da non eseguire il timeout durante i periodi di inattività.

### <a name="application-initialization-module"></a>Modulo Inizializzazione applicazione

*Si applica alle app ospitate in-process e out-of-process.*

[Inizializzazione applicazione di IIS](/iis/get-started/whats-new-in-iis-8/iis-80-application-initialization) è una funzionalità di IIS che invia una richiesta HTTP all'app quando il pool di app viene avviato o riciclato. La richiesta attiva l'avvio dell'app. Per impostazione predefinita, IIS invia una richiesta all'URL radice dell'app (`/`) per inizializzare l'app (vedere le [risorse aggiuntive](#application-initialization-module-and-idle-timeout-additional-resources) per altri dettagli sulla configurazione).

Verificare che la funzionalità del ruolo Inizializzazione applicazione di IIS sia abilitata:

Nei sistemi desktop Windows 7 o versioni successive quando si usa IIS in locale:

1. Passare a **Pannello di controllo** > **Programmi** > **Programmi e funzionalità** > **Disattivare o attivare le funzionalità di Windows** (a sinistra dello schermo).
1. Aprire **Internet Information Services** > **Servizi Web** > **Funzionalità per lo sviluppo di applicazioni**.
1. Selezionare la casella di controllo **Inizializzazione applicazione**.

In Windows Server 2008 R2 o versioni successive:

1. Aprire l'**Aggiunta guidata ruoli e funzionalità**.
1. Nel pannello **Selezione servizi ruolo** aprire il nodo **Sviluppo applicazioni**.
1. Selezionare la casella di controllo **Inizializzazione applicazione**.

Per abilitare il modulo Inizializzazione applicazione per il sito, usare uno degli approcci seguenti:

* In Gestione IIS:

  1. Selezionare **Pool di applicazioni** nel pannello **Connessioni**.
  1. Fare clic con il pulsante destro del mouse sul pool di app dell'app nell'elenco e scegliere **Impostazioni avanzate**.
  1. L'impostazione predefinita per **Modalità avvio** è **OnDemand**. Impostare **Modalità avvio** su **AlwaysRunning**. Selezionare **OK**.
  1. Aprire il nodo **Siti** nel pannello **Connessioni**.
  1. Fare clic con il pulsante destro del mouse sull'app e scegliere **Gestisci sito Web** > **Impostazioni avanzate**.
  1. L'impostazione predefinita di **Precaricamento abilitato** è **False**. Impostare **Precaricamento abilitato** su **True**. Selezionare **OK**.

* Usando `web.config` , aggiungere l' `<applicationInitialization>` elemento con `doAppInitAfterRestart` impostato su `true` agli `<system.webServer>` elementi nel file web.config' dell'app:

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
1. L'impostazione predefinita di **Timeout di inattività (minuti)** è **20** minuti. Impostare **Timeout di inattività (minuti)** su **0** (zero). Selezionare **OK**.
1. Riciclare il processo di lavoro.

Per evitare il timeout delle app ospitate [out-of-process](#out-of-process-hosting-model), usare uno degli approcci seguenti:

* Effettuare il ping dell'app da un servizio esterno per mantenerla in esecuzione.
* Se l'app ospita solo servizi in background, evitare l'hosting IIS e usare un [servizio Windows per ospitare l'app ASP.NET Core](xref:host-and-deploy/windows-service).

### <a name="application-initialization-module-and-idle-timeout-additional-resources"></a>Risorse aggiuntive per il modulo Inizializzazione applicazione e il timeout di inattività

* [IIS 8.0 Application Initialization](/iis/get-started/whats-new-in-iis-8/iis-80-application-initialization) (Inizializzazione delle applicazioni in IIS 8.0)
* [Inizializzazione \<applicationInitialization> dell'applicazione ](/iis/configuration/system.webserver/applicationinitialization/).
* [Impostazioni del modello di processo per un \<processModel> pool di applicazioni ](/iis/configuration/system.applicationhost/applicationpools/add/processmodel).

## <a name="deployment-resources-for-iis-administrators"></a>Risorse di distribuzione per amministratori IIS

* [Documentazione di ISS](/iis)
* [Getting Started with the IIS Manager in IIS](/iis/get-started/getting-started-with-iis/getting-started-with-the-iis-manager-in-iis-7-and-iis-8) (Introduzione a Gestione IIS in IIS)
* [Distribuzione di applicazioni .NET Core](/dotnet/core/deploying/)
* <xref:host-and-deploy/aspnet-core-module>
* <xref:host-and-deploy/directory-structure>
* <xref:host-and-deploy/iis/modules>
* <xref:test/troubleshoot-azure-iis>
* <xref:host-and-deploy/azure-iis-errors-reference>

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:test/troubleshoot>
* <xref:index>
* [Il sito IIS ufficiale di Microsoft](https://www.iis.net/)
* [Libreria di contenuti tecnici di Windows Server](/windows-server/windows-server)
* [HTTP/2 on IIS](/iis/get-started/whats-new-in-iis-10/http2-on-iis) (HTTP/2 in IIS)
* <xref:host-and-deploy/iis/transform-webconfig>

::: moniker-end

::: moniker range="= aspnetcore-2.2"

Per un'esercitazione sulla pubblicazione di un'app ASP.NET Core in un server IIS, vedere <xref:tutorials/publish-to-iis>.

[Installare il bundle di hosting .NET Core](#install-the-net-core-hosting-bundle)

## <a name="supported-operating-systems"></a>Sistemi operativi supportati

Sono supportati i sistemi operativi seguenti:

* Windows 7 o versione successiva
* Windows Server 2008 R2 o versione successiva

Il [server HTTP.sys](xref:fundamentals/servers/httpsys) (chiamato in precedenza WebListener) non funziona in una configurazione proxy inverso con IIS. È necessario usare il [server Kestrel](xref:fundamentals/servers/kestrel).

Per informazioni sull'hosting in Azure, vedere <xref:host-and-deploy/azure-apps/index>.

Per linee guida per la risoluzione dei problemi, vedere <xref:test/troubleshoot>.

## <a name="supported-platforms"></a>Piattaforme supportate

Sono supportate le app pubblicate per la distribuzione a 32 bit (x86) o a 64 bit (x64). Distribuire un'app a 32 bit con .NET Core SDK a 32 bit (x86), a meno che l'app:

* Non richieda lo spazio indirizzi di memoria virtuale più grande disponibile per un'app a 64 bit.
* Non richieda le dimensioni maggiori dello stack IIS.
* Non abbia dipendenze native a 64 bit.

Usare .NET Core SDK a 64 bit (x64) per pubblicare un'app a 64 bit. Un runtime a 64 bit deve essere presente nel sistema host.

## <a name="hosting-models"></a>Modelli di hosting

### <a name="in-process-hosting-model"></a>Modello di hosting in-process

Se si usa l'hosting in-process, un'app ASP.NET Core esegue lo stesso processo del processo di lavoro IIS. L'hosting in-process offre prestazioni migliori rispetto all'hosting out-of-process perché:

* Le richieste non vengono inoltrate tramite proxy sulla scheda loopback. Una scheda di loopback è un'interfaccia di rete che restituisce il traffico di rete in uscita allo stesso computer.

Per gestire il processo, IIS usa il [servizio Attivazione processo Windows (WAS)](/iis/manage/provisioning-and-managing-iis/features-of-the-windows-process-activation-service-was).

Il [modulo ASP.NET Core](xref:host-and-deploy/aspnet-core-module):

* Esegue l'inizializzazione dell'app.
  * Carica [CoreCLR](/dotnet/standard/glossary#coreclr).
  * Chiama `Program.Main`.
* Gestisce la durata della richiesta nativa di IIS.

Il modello di hosting In-Process non è supportato per le app ASP.NET Core destinate a .NET Framework.

Il diagramma seguente illustra la relazione tra IIS, il modulo ASP.NET Core e un'app ospitata in-process:

![Modulo ASP.NET Core nello scenario di hosting in-process](index/_static/ancm-inprocess.png)

Una richiesta arriva dal Web al driver HTTP.sys in modalità kernel. Il driver instrada la richiesta nativa IIS sulla porta configurata per il sito Web, in genere 80 (HTTP) o 443 (HTTPS). Il modulo ASP.NET Core riceve la richiesta nativa e la passa al server HTTP IIS ( `IISHttpServer` ). Il server HTTP di IIS è un'implementazione di server in-process per IIS che converte la richiesta da nativa a gestita.

Dopo che la richiesta è stata elaborata dal server HTTP di IIS, viene eseguito il push della richiesta nella pipeline middleware ASP.NET Core. La pipeline middleware gestisce la richiesta e la passa come istanza di `HttpContext` alla logica dell'app. La risposta dell'app viene restituita a IIS tramite il server HTTP di IIS. IIS invia la risposta al client che ha avviato la richiesta.

L'hosting in-process è una funzionalità che richiede il consenso esplicito per le app esistenti, mentre i modelli [dotnet new](/dotnet/core/tools/dotnet-new) usano per impostazione predefinita il modello di hosting in-process per tutti gli scenari IIS e IIS Express.

`CreateDefaultBuilder` aggiunge un'istanza di <xref:Microsoft.AspNetCore.Hosting.Server.IServer> chiamando il metodo <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderIISExtensions.UseIIS%2A> per avviare [CoreCLR](/dotnet/standard/glossary#coreclr) e ospitare l'app all'interno del processo di lavoro IIS (*w3wp.exe* o *iisexpress.exe*). I test delle prestazioni indicano che l'hosting di un'app .NET Core in-process offre una velocità effettiva delle richieste significativamente superiore rispetto all'hosting dell'app out-of-process o all'inoltro delle richieste al server [Kestrel](xref:fundamentals/servers/kestrel).

### <a name="out-of-process-hosting-model"></a>Modello di hosting out-of-process

Poiché le app ASP.NET Core vengono eseguite in un processo separato dal processo di lavoro IIS, il modulo ASP.NET Core gestisce la gestione dei processi. Il modulo avvia il processo per l'app ASP.NET Core quando arriva la prima richiesta e riavvia l'app se viene arrestata o si arresta in modo anomalo. Si tratta essenzialmente dello stesso comportamento delle app eseguite in-process e gestite dal [servizio Attivazione processo Windows](/iis/manage/provisioning-and-managing-iis/features-of-the-windows-process-activation-service-was).

Il diagramma seguente illustra la relazione tra IIS, il modulo ASP.NET Core e un'app ospitata out-of-process:

![Modulo ASP.NET Core nello scenario di hosting out-of-process](index/_static/ancm-outofprocess.png)

Le richieste arrivano dal Web al driver HTTP.sys in modalità kernel. Il driver instrada le richieste a IIS sulla porta configurata per il sito Web, in genere 80 (HTTP) o 443 (HTTPS). Il modulo inoltra le richieste a Kestrel su una porta casuale per l'app non corrispondente alla porta 80 o 443.

Il modulo specifica la porta tramite una variabile di ambiente all'avvio e l'estensione <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderIISExtensions.UseIISIntegration*> configura il server per l'ascolto su `http://localhost:{PORT}`. Vengono eseguiti controlli aggiuntivi e le richieste che non provengono dal modulo vengono rifiutate. Il modulo non supporta l'inoltro HTTPS, pertanto le richieste vengono inoltrate tramite HTTP anche se sono state ricevute da IIS tramite HTTPS.

Dopo che Kestrel ha prelevato la richiesta dal modulo, viene eseguito il push della richiesta nella pipeline middleware ASP.NET Core. La pipeline middleware gestisce la richiesta e la passa come istanza di `HttpContext` alla logica dell'app. Il middleware aggiunto dall'integrazione di IIS aggiorna lo schema, l'IP remoto e il percorso di base all'account per l'inoltro della richiesta a Kestrel. La risposta dell'app viene quindi passata a IIS, che ne esegue di nuovo il push al client HTTP che ha avviato la richiesta.

Per indicazioni sulla configurazione del modulo ASP.NET Core, vedere <xref:host-and-deploy/aspnet-core-module>.

Per altre informazioni sull'hosting, vedere [Hosting in ASP.NET Core](xref:fundamentals/index#host).

## <a name="application-configuration"></a>Configurazione dell'applicazione

### <a name="enable-the-iisintegration-components"></a>Abilitare i componenti IISIntegration

Quando si compila un host in `CreateWebHostBuilder` (*Program.cs*), chiamare <xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder%2A> per abilitare l'integrazione con IIS:

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        ...
```

Per altre informazioni su `CreateDefaultBuilder`, vedere <xref:fundamentals/host/web-host#set-up-a-host>.

### <a name="iis-options"></a>Opzioni IIS

**Modello di hosting in-process**

Per configurare le opzioni del server IIS includere una configurazione del servizio per <xref:Microsoft.AspNetCore.Builder.IISServerOptions> in <xref:Microsoft.AspNetCore.Hosting.IStartup.ConfigureServices%2A>. L'esempio seguente disabilita AutomaticAuthentication:

```csharp
services.Configure<IISServerOptions>(options => 
{
    options.AutomaticAuthentication = false;
});
```

| Opzione                         | Predefinito | Impostazione |
| ------------------------------ | :-----: | ------- |
| `AutomaticAuthentication`      | `true`  | Se `true`, IIS Server imposta l'utente `HttpContext.User` autenticato tramite l'[autenticazione di Windows](xref:security/authentication/windowsauth). Se `false`, il server fornisce solo un'identità per `HttpContext.User` e risponde alle richieste esplicite di `AuthenticationScheme`. Per il funzionamento di `AutomaticAuthentication` l’autenticazione di Windows deve essere abilitata in IIS. Per altre informazioni, vedere [Autenticazione di Windows](xref:security/authentication/windowsauth). |
| `AuthenticationDisplayName`    | `null`  | Imposta il nome visualizzato dagli utenti nelle pagine di accesso. |

**Modello di hosting out-of-process**

Per configurare le opzioni di IIS includere una configurazione del servizio per <xref:Microsoft.AspNetCore.Builder.IISOptions> in <xref:Microsoft.AspNetCore.Hosting.IStartup.ConfigureServices*>. Nell'esempio seguente si impedisce all'app di popolare `HttpContext.Connection.ClientCertificate`:

```csharp
services.Configure<IISOptions>(options => 
{
    options.ForwardClientCertificate = false;
});
```

| Opzione                         | Predefinito | Impostazione |
| ------------------------------ | :-----: | ------- |
| `AutomaticAuthentication`      | `true`  | Se `true`, il [middleware di integrazione IIS](#enable-the-iisintegration-components) imposta `HttpContext.User` autenticato tramite l'[autenticazione di Windows](xref:security/authentication/windowsauth). Se `false`, il middleware fornisce solo un'identità per `HttpContext.User` e risponde solo alle richieste esplicite di `AuthenticationScheme`. Per il funzionamento di `AutomaticAuthentication` l’autenticazione di Windows deve essere abilitata in IIS. Per altre informazioni, vedere l'argomento [Autenticazione di Windows](xref:security/authentication/windowsauth). |
| `AuthenticationDisplayName`    | `null`  | Imposta il nome visualizzato dagli utenti nelle pagine di accesso. |
| `ForwardClientCertificate`     | `true`  | Se è `true` ed è presente l’intestazione della richiesta `MS-ASPNETCORE-CLIENTCERT`, `HttpContext.Connection.ClientCertificate` viene popolato. |

### <a name="proxy-server-and-load-balancer-scenarios"></a>Scenari con server proxy e servizi di bilanciamento del carico

Il [middleware di integrazione IIS](#enable-the-iisintegration-components), che consente di configurare il middleware delle intestazioni inoltrate, e il modulo ASP.NET Core sono configurati per inoltrare lo schema (HTTP/HTTPS) e l'indirizzo IP remoto di origine della richiesta. Potrebbero essere necessari interventi di configurazione aggiuntivi per le app ospitate dietro ulteriori server proxy e servizi di bilanciamento del carico. Per altre informazioni, vedere [Configurare ASP.NET Core per l'utilizzo di server proxy e servizi di bilanciamento del carico](xref:host-and-deploy/proxy-load-balancer).

### <a name="webconfig-file"></a>File web.config

Il file *web.config* configura il [modulo ASP.NET Core](xref:host-and-deploy/aspnet-core-module). La creazione, la trasformazione e la pubblicazione del file *web.config* sono operazioni gestite da una destinazione MSBuild (`_TransformWebConfig`) quando viene pubblicato il progetto. Questa destinazione si trova tra le destinazioni Web SDK (`Microsoft.NET.Sdk.Web`). L'SDK è impostato all'inizio del file di progetto:

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
```

Se nel progetto non è presente un file *web.config*, il file viene creato con un valore *processPath* e *argomenti* corretti per la configurazione del modulo ASP.NET Core. Il file viene quindi spostato nell'[output pubblicato](xref:host-and-deploy/directory-structure).

Se nel progetto è presente un file *web.config*, il file viene trasformato con il valore di *processPath* e gli *argomenti* corretti per la configurazione del modulo ASP.NET Core, quindi viene spostato nell'output pubblicato. La trasformazione non modifica le impostazioni di configurazione di IIS incluse nel file.

Il file *web.config* può fornire ulteriori impostazioni di configurazione di IIS che controllano i moduli IIS attivi. Per informazioni sui moduli IIS in grado di elaborare le richieste con le app di ASP.NET Core, vedere l'argomento [Moduli IIS](xref:host-and-deploy/iis/modules).

Per impedire a Web SDK di trasformare il file di *web.config* , utilizzare la **\<IsTransformWebConfigDisabled>** proprietà nel file di progetto:

```xml
<PropertyGroup>
  <IsTransformWebConfigDisabled>true</IsTransformWebConfigDisabled>
</PropertyGroup>
```

Quando si disabilita la trasformazione del file in Web SDK, il valore di *processPath* e gli *argomenti* devono essere impostati manualmente dallo sviluppatore. Per altre informazioni, vedere <xref:host-and-deploy/aspnet-core-module>.

### <a name="webconfig-file-location"></a>Posizione del file web.config

Per configurare correttamente il [modulo ASP.NET Core](xref:host-and-deploy/aspnet-core-module) , il file di *web.config* deve essere presente nel percorso radice del [contenuto](xref:fundamentals/index#content-root) (in genere il percorso di base dell'app) dell'app distribuita. Corrisponde al percorso fisico del sito Web fornito a IIS. Il file *web.config* deve essere presente nella radice dell'app per abilitare la pubblicazione di più app mediante Distribuzione Web.

I file sensibili sono presenti nel percorso fisico dell'app, ad esempio *\<assembly>.runtimeconfig.json*, *\<assembly> XML* (commenti in formato documentazione XML) e *\<assembly>.deps.jssu*. Quando il file *web.config* è presente e il sito viene avviato normalmente, IIS non rende disponibili questi file riservati se vengono richiesti. Se il file *web.config* non è presente, ha un nome non corretto o non è in grado di configurare il sito per l'avvio normale, IIS potrebbe rendere disponibili pubblicamente i file riservati.

**Il file di *web.config* deve essere presente nella distribuzione in qualsiasi momento, denominato correttamente e in grado di configurare il sito per l'avvio normale. Non rimuovere mai il file di *web.config* da una distribuzione di produzione.**

### <a name="transform-webconfig"></a>Trasformare web.config

Se è necessario trasformare *web.config* in fase di pubblicazione (ad esempio, impostare variabili di ambiente in base a configurazione, profilo o ambiente), vedere <xref:host-and-deploy/iis/transform-webconfig>.

## <a name="iis-configuration"></a>Configurazione di IIS

**Sistemi operativi Windows Server**

Abilitare il ruolo del server **Server Web (IIS)** e stabilire i servizi di ruolo.

1. Usare la procedura guidata **Aggiungi ruoli e funzionalità** accessibile tramite il menu **Gestisci** o il collegamento in **Server Manager**. Nel passaggio **Ruoli del server** selezionare la casella per **Server Web (IIS)**.

   ![Il ruolo Server Web IIS viene selezionato nel passaggio dei ruoli del server Selezionare.](index/_static/server-roles-ws2016.png)

1. Dopo il passaggio **Funzionalità**, viene caricato il passaggio **Servizi ruolo** per Server Web (IIS). Selezionare i servizi ruolo IIS desiderati o accettare i servizi ruolo predefiniti specificati.

   ![I servizi ruolo predefiniti vengono selezionati nel passaggio Selezionare i servizi ruolo.](index/_static/role-services-ws2016.png)

   **Autenticazione di Windows (facoltativa)**  
   Per abilitare l'autenticazione di Windows, espandere i nodi seguenti: sicurezza del **server Web**  >  . Selezionare la funzionalità **Autenticazione di Windows**. Per ulteriori informazioni, vedere [ \<windowsAuthentication> autenticazione di Windows](/iis/configuration/system.webServer/security/authentication/windowsAuthentication/) e [configurare l'autenticazione di Windows](xref:security/authentication/windowsauth).

   **WebSockets (facoltativo)**  
   WebSockets è supportato con ASP.NET Core 1.1 o versioni successive. Per abilitare i WebSocket, espandere i nodi seguenti: sviluppo di applicazioni **server Web**  >  . Selezionare la funzionalità **Protocollo WebSocket**. Per altre informazioni, vedere [Oggetti WebSocket](xref:fundamentals/websockets).

1. Procedere con il passaggio **Conferma** per installare il ruolo del server web e i servizi. Dopo l'installazione del ruolo **server Web (IIS)** non è necessario riavviare il server o IIS.

**Sistemi operativi desktop di Windows**

Abilitare **Console di gestione IIS** e **Servizi Web**.

1. Passare a **Pannello di controllo** > **Programmi** > **Programmi e funzionalità** > **Disattivare o attivare le funzionalità di Windows** (a sinistra dello schermo).

1. Aprire il nodo **Internet Information Services**. Aprire il nodo **Strumenti di gestione Web**.

1. Selezionare la casella per **Console di gestione IIS**.

1. Selezionare la casella per **Servizi World Wide Web**.

1. Accettare le funzionalità predefinite per **Servizi World Wide Web** o personalizzare le funzionalità IIS.

   **Autenticazione di Windows (facoltativa)**  
   Per abilitare l'autenticazione di Windows, espandere i nodi seguenti: **World Wide Web sicurezza dei servizi**  >  . Selezionare la funzionalità **Autenticazione di Windows**. Per ulteriori informazioni, vedere [ \<windowsAuthentication> autenticazione di Windows](/iis/configuration/system.webServer/security/authentication/windowsAuthentication/) e [configurare l'autenticazione di Windows](xref:security/authentication/windowsauth).

   **WebSockets (facoltativo)**  
   WebSockets è supportato con ASP.NET Core 1.1 o versioni successive. Per abilitare i WebSocket, espandere i nodi seguenti: funzionalità di sviluppo di applicazioni di **World Wide Web Services**  >  . Selezionare la funzionalità **Protocollo WebSocket**. Per altre informazioni, vedere [Oggetti WebSocket](xref:fundamentals/websockets).

1. Se l'installazione di IIS richiede un riavvio, riavviare il sistema.

![La console di gestione IIS e servizi World Wide Web sono selezionati nelle funzionalità di Windows.](index/_static/windows-features-win10.png)

## <a name="install-the-net-core-hosting-bundle"></a>Installare il bundle di hosting .NET Core

Installare il *bundle di hosting .NET Core* nel sistema di hosting. Il bundle installa il runtime di .NET Core, la libreria di .NET Core e il [modulo ASP.NET Core](xref:host-and-deploy/aspnet-core-module). Il modulo consente l'esecuzione delle app ASP.NET Core dietro IIS.

> [!IMPORTANT]
> Se il bundle di hosting viene installato prima di IIS, è necessario riparare l'installazione del bundle. Eseguire di nuovo il programma di installazione del bundle di hosting dopo l'installazione di IIS.
>
> Se il bundle di hosting viene installato dopo l'installazione della versione a 64 bit (x64) di .NET Core, uno o più SDK potrebbero risultare mancanti ([Non sono stati rilevati .NET Core SDK](xref:test/troubleshoot#no-net-core-sdks-were-detected)). Per risolvere il problema, vedere <xref:test/troubleshoot#missing-sdk-after-installing-the-net-core-hosting-bundle>.

### <a name="download"></a>Scarica

1. Passare alla pagina [download di .NET Core](https://dotnet.microsoft.com/download/dotnet-core) .
1. Selezionare la versione desiderata di .NET Core.
1. Nella colonna **Run apps - Runtime** (App di esecuzione - Runtime), trovare la riga della versione di runtime di .NET Core desiderata.
1. Scaricare il programma di installazione usando il collegamento **bundle di hosting** .

> [!WARNING]
> Alcuni programmi di installazione contengono versioni che hanno terminato il loro ciclo di vita e non sono più supportate da Microsoft. Per altre informazioni, vedere i [criteri di supporto](https://dotnet.microsoft.com/platform/support/policy/dotnet-core).

### <a name="install-the-hosting-bundle"></a>Installare il bundle di hosting

1. Eseguire il programma di installazione nel server. Quando si esegue il programma di installazione da una shell dei comandi di amministratore sono disponibili i parametri seguenti:

   * `OPT_NO_ANCM=1`: Ignorare l'installazione del modulo ASP.NET Core.
   * `OPT_NO_RUNTIME=1`: Ignorare l'installazione del runtime di .NET Core. Utilizzato quando il server ospita solo le [distribuzioni autosufficienti (SCD)](/dotnet/core/deploying/#self-contained-deployments-scd).
   * `OPT_NO_SHAREDFX=1`: Ignora l'installazione del Framework condiviso ASP.NET (runtime ASP.NET). Utilizzato quando il server ospita solo le [distribuzioni autosufficienti (SCD)](/dotnet/core/deploying/#self-contained-deployments-scd).
   * `OPT_NO_X86=1`: Ignora l'installazione di Runtime x86. Usare questo parametro se si è certi che non verrà eseguito l'hosting di app a 32 bit. Se non esiste alcuna possibilità che in futuro venga eseguito l'hosting di app sia a 32 che a 64 bit, non usare questo parametro e installare entrambi i runtime.
   * `OPT_NO_SHARED_CONFIG_CHECK=1`: Disabilitare il controllo per l'utilizzo di una configurazione condivisa di IIS quando la configurazione condivisa (*applicationHost.config*) si trova nello stesso computer dell'installazione di IIS. *Disponibile solo per i programmi di installazione di bundler di hosting ASP.NET Core 2.2 o versioni successive.* Per altre informazioni, vedere <xref:host-and-deploy/aspnet-core-module#aspnet-core-module-with-an-iis-shared-configuration>.
1. Riavviare il sistema o eseguire i comandi seguenti in una shell dei comandi:

   ```console
   net stop was /y
   net start w3svc
   ```
   Il riavvio di IIS rileva una modifica alla variabile di ambiente di sistema PATH apportata dal programma di installazione.

Non è necessario arrestare manualmente singoli siti in IIS durante l'installazione del bundle di hosting. Le app ospitate (siti IIS) vengono riavviate quando IIS viene riavviato. Le app vengono riavviate quando ricevono la prima richiesta, incluso il [modulo di inizializzazione dell'applicazione](#application-initialization-module-and-idle-timeout).

ASP.NET Core adotta il comportamento di rollforward per le versioni di patch dei pacchetti di Framework condivisi. Quando le app ospitate da IIS vengono riavviate con IIS, le app vengono caricate con le ultime versioni di patch dei pacchetti a cui si fa riferimento quando ricevono la prima richiesta. Se IIS non viene riavviato, le app vengono riavviate e presentano un comportamento di rollforward quando i processi di lavoro vengono riciclati e ricevono la prima richiesta.

> [!NOTE]
> Per informazioni sulla configurazione condivisa di IIS, vedere [Modulo di ASP.NET Core con configurazione condivisa di IIS](xref:host-and-deploy/aspnet-core-module#aspnet-core-module-with-an-iis-shared-configuration).

## <a name="install-web-deploy-when-publishing-with-visual-studio"></a>Installare la Distribuzione Web quando si pubblica con Visual Studio

Quando si distribuiscono app ai server con [Distribuzione Web](/iis/install/installing-publishing-technologies/installing-and-configuring-web-deploy-on-iis-80-or-later), installare la versione più recente di Distribuzione Web nel server. Per installare Distribuzione Web usare l'[Installazione guidata piattaforma Web (WebPI)](https://www.microsoft.com/web/downloads/platform.aspx) o ottenere direttamente un programma di installazione in [Microsoft Download Center](https://www.microsoft.com/download/details.aspx?id=43717). Il metodo preferito consiste nell'usare WebPI. WebPI offre un'installazione autonoma e una configurazione per i provider di hosting.

## <a name="create-the-iis-site"></a>Creare il sito IIS

1. Nel sistema host creare una cartella per contenere le cartelle e i file pubblicati dell'app. In un passaggio successivo, il percorso della cartella viene fornito a IIS come percorso fisico dell'app. Per altre informazioni sulla cartella di distribuzione e il layout dei file di un'app, vedere <xref:host-and-deploy/directory-structure>.

1. In Gestione IIS aprire il nodo del server nel pannello **connessioni** . Fare clic con il pulsante destro del mouse sulla cartella **Siti**. Scegliere **Aggiungi sito Web** dal menu di scelta rapida.

1. Specificare un **Nome del sito** e impostare il **Percorso fisico** per la cartella di distribuzione dell'app. Specificare la configurazione dell' **associazione** e creare il sito Web selezionando **OK**:

   ![Specificare il nome del sito, il percorso fisico e il nome Host nel passaggio Aggiungi sito Web.](index/_static/add-website-ws2016.png)

   > [!WARNING]
   > Le associazioni con caratteri jolly di livello superiore (`http://*:80/` e `http://+:80`) **non** devono essere usate, poiché possono introdurre vulnerabilità a livello di sicurezza nell'app. Questo concetto vale sia per i caratteri jolly sicuri che vulnerabili. Usare nomi host espliciti al posto di caratteri jolly. L'associazione con caratteri jolly del sottodominio (ad esempio, `*.mysub.com`) non costituisce un rischio per la sicurezza se viene controllato l'intero dominio padre (a differenza di `*.com`, che è vulnerabile). Vedere la [sezione 5.4 di RFC7230](https://tools.ietf.org/html/rfc7230#section-5.4) per altre informazioni.

1. Nel nodo del server selezionare **Pool di applicazioni**.

1. Fare clic con il pulsante destro del mouse sul pool di applicazioni del sito e scegliere **Impostazioni di base** dal menu di scelta rapida.

1. Nella finestra **Modifica pool di applicazioni** impostare **Versione .NET CLR** su **Nessun codice gestito**.

   ![Impostare Nessun codice gestito per la versione .NET CLR.](index/_static/edit-apppool-ws2016.png)

    ASP.NET Core viene eseguito in un processo separato e gestisce il runtime. ASP.NET Core non si basa sul caricamento di Common Language Runtime (CLR di .NET) desktop. Core Common Language Runtime (CoreCLR) for .NET Core viene avviato per ospitare l'app nel processo di lavoro. L'impostazione di **Versione .NET CLR** su **Nessun codice gestito** è facoltativa ma consigliata.

1. *ASP.NET Core 2.2 o versioni successive*: per una [distribuzione autonoma](/dotnet/core/deploying/#self-contained-deployments-scd) a 64 bit (x64) che usa il [modello di hosting In-Process](#in-process-hosting-model), disabilitare il pool di app per i processi a 32 bit (x86).

   Nella barra laterale **Azioni** in Gestione IIS > **Pool di applicazioni** selezionare **Impostazioni predefinite pool di applicazioni** o **Impostazioni avanzate**. Individuare **Attiva applicazioni a 32 bit** e impostare il valore su `False`. Questa impostazione non viene applicata alle app distribuite per l'[hosting out-of-process](xref:host-and-deploy/aspnet-core-module#out-of-process-hosting-model).

1. Confermare che l'identità del modello del processo disponga delle autorizzazioni appropriate.

   Se l'identità predefinita del pool di app (**modello**  >  **Identity** di processo) viene modificata **da Identity ApplicationPool** a un'altra identità, verificare che la nuova identità disponga delle autorizzazioni necessarie per accedere alla cartella dell'app, al database e ad altre risorse richieste. Ad esempio, il pool di applicazioni richiede l'accesso in lettura e scrittura alle cartelle in cui l'app legge e scrive i file.

**Configurazione dell'autenticazione di Windows (facoltativa)**  
Per altre informazioni, vedere [Configurare l'autenticazione Windows](xref:security/authentication/windowsauth).

## <a name="deploy-the-app"></a>Distribuire l'app

Distribuire l'app nella cartella **Percorso fisico** di IIS specificata nella sezione [Creare il sito IIS](#create-the-iis-site). [Distribuzione Web](/iis/publish/using-web-deploy/introduction-to-web-deploy) è il meccanismo consigliato per la distribuzione, ma esistono diverse opzioni per spostare l'app dalla cartella di *pubblicazione* del progetto alla cartella di distribuzione del sistema di hosting.

### <a name="web-deploy-with-visual-studio"></a>Distribuzione web con Visual Studio

Vedere l'argomento [Visual Studio publish profiles for ASP.NET Core app deployment](xref:host-and-deploy/visual-studio-publish-profiles#publish-profiles) (Profili di pubblicazione Visual Studio per la distribuzione di app ASP.NET Core) per informazioni su come creare un profilo di pubblicazione da usare con Distribuzione Web. Se il provider di hosting fornisce un profilo di pubblicazione o il supporto per crearne uno, scaricare il profilo e importarlo usando la finestra di dialogo **Pubblica** di Visual Studio:

![Pagina della finestra di dialogo Pubblica](index/_static/pub-dialog.png)

### <a name="web-deploy-outside-of-visual-studio"></a>Distribuzione Web all'esterno di Visual Studio

È anche possibile usare [Distribuzione Web](/iis/publish/using-web-deploy/introduction-to-web-deploy) all'esterno di Visual Studio dalla riga di comando. Per altre informazioni sulla distribuzione, vedere [Strumento Distribuzione Web](/iis/publish/using-web-deploy/use-the-web-deployment-tool).

### <a name="alternatives-to-web-deploy"></a>Alternative a Distribuzione web

Usare uno dei metodi disponibili, ad esempio la copia manuale, [Xcopy](/windows-server/administration/windows-commands/xcopy), [Robocopy](/windows-server/administration/windows-commands/robocopy) o [PowerShell](/powershell/), per spostare l'app nel sistema di hosting.

Per altre informazioni sulla distribuzione di ASP.NET Core in IIS, vedere la sezione [Risorse di distribuzione per amministratori IIS](#deployment-resources-for-iis-administrators).

## <a name="browse-the-website"></a>Esplorare il sito Web

Dopo aver distribuito l'app nel sistema di hosting, effettuare una richiesta a uno degli endpoint pubblici dell'app.

Nell'esempio seguente il sito è associato al **nome host** di IIS `www.mysite.com` sulla **porta** `80`. Viene effettuata una richiesta a `http://www.mysite.com`:

![Il browser Microsoft Edge ha caricato la pagina di avvio IIS.](index/_static/browsewebsite.png)

## <a name="locked-deployment-files"></a>File di distribuzione bloccati

I file nella cartella di distribuzione sono bloccati durante l'esecuzione dell'app. I file bloccati non possono essere sovrascritti durante la distribuzione. Per rilasciare i file bloccati in una distribuzione, arrestare il pool di applicazioni usando **uno** dei metodi seguenti:

* Usare Distribuzione Web e fare riferimento a `Microsoft.NET.Sdk.Web` nel file di progetto. Nella radice della directory dell'app web viene posizionato un file *app_offline.htm*. Quando il file è presente, il modulo ASP.NET Core arresta normalmente l'app e rende disponibile il file *app_offline.htm* durante la distribuzione. Per altre informazioni, vedere la [Guida di riferimento per la configurazione del modulo ASP.NET Core](xref:host-and-deploy/aspnet-core-module#app_offlinehtm).
* Arrestare manualmente il pool di applicazioni in Gestione IIS sul server.
* Usare PowerShell per eliminare *app_offline.htm* (richiede PowerShell 5 o versione successiva):

  ```powershell
  $pathToApp = 'PATH_TO_APP'

  # Stop the AppPool
  New-Item -Path $pathToApp app_offline.htm

  # Provide script commands here to deploy the app

  # Restart the AppPool
  Remove-Item -Path $pathToApp app_offline.htm

  ```

## <a name="data-protection"></a>Protezione dei dati

Lo [stack di protezione dei dati di ASP.NET Core](xref:security/data-protection/introduction) viene usato da diversi [middleware](xref:fundamentals/middleware/index) di ASP.NET Core, inclusi quelli usati nell'autenticazione. Anche se le DPAPI (Data Protection API) non vengono chiamate dal codice dell'utente, è consigliabile configurare la protezione dati per la creazione di un [archivio di chiavi](xref:security/data-protection/implementation/key-management) crittografiche permanente con uno script di distribuzione o nel codice dell'utente. Se non si configura la protezione dei dati, le chiavi vengono mantenute in memoria ed eliminate al riavvio dell'app.

Se il gruppo di chiavi viene archiviato in memoria quando l'app viene riavviata:

* Tutti i cookie token di autenticazione basati su vengono invalidati. 
* Gli utenti devono ripetere l'accesso alla richiesta successiva. 
* Tutti i dati protetti con il gruppo di chiavi non possono più essere decrittografati. Questo può includere [i token CSRF](xref:security/anti-request-forgery#aspnet-core-antiforgery-configuration) e [ASP.NET Core TempData MVC cookie ](xref:fundamentals/app-state#tempdata).

Per configurare la protezione dati in IIS in modo da rendere permanente il gruppo di chiavi, usare **uno** degli approcci seguenti:

* **Creare chiavi del Registro di sistema di protezione dati**

  Le chiavi di protezione dati usate dalle app ASP.NET Core vengono archiviate nel Registro di sistema esterno alle app. Per mantenere le chiavi per una determinata app, creare chiavi del Registro di sistema per il pool di applicazioni.

  Per le installazioni IIS autonome non web farm è possibile usare lo [script PowerShell Data Protection Provision-AutoGenKeys.ps1 (Provisioning di protezione dati-AutoGenKeys.ps1)](https://github.com/dotnet/AspNetCore/blob/main/src/DataProtection/Provision-AutoGenKeys.ps1) per ogni pool di app usato con un'app ASP.NET Core. Questo script crea una chiave del Registro di sistema nel registro HKLM accessibile solo all'account del processo di lavoro del pool di applicazioni dell'app. Le chiavi vengono crittografate a riposo tramite DPAPI con una chiave a livello del computer.

  In scenari di web farm un'app può essere configurata in modo da usare un percorso UNC in cui archiviare il proprio gruppo di chiavi di protezione dati. Per impostazione predefinita, le chiavi di protezione dati non vengono crittografate. Assicurarsi che le autorizzazioni file per la condivisione di rete siano limitate all'account di Windows in cui viene eseguita l'app. È possibile usare un certificato X509 per la protezione delle chiavi inattive. Considerare l'implementazione di un meccanismo per consentire agli utenti di caricare i certificati: inserire i certificati nell'archivio certificati attendibili dell'utente e assicurarsi che siano disponibili in tutti i computer in cui viene eseguita l'app dell'utente. Vedere [Configurare la protezione dati di ASP.NET Core](xref:security/data-protection/configuration/overview) per altri dettagli.

* **Configurare il pool di applicazioni IIS per caricare il profilo utente**

  Questa impostazione si trova nella sezione **Modello del processo** in **Impostazioni avanzate** per il pool di app. Impostare **Caricamento del profilo utente** su `True`. Con l'impostazione `True` le chiavi vengono archiviate nella directory del profilo utente e protette tramite DPAPI con una chiave specifica per l'account utente. Le chiavi vengono salvate in modo permanente nella cartella *%LOCALAPPDATA%/ASP.NET/DataProtection-Keys*.

  Deve essere abilitato anche l'[attributo setProfileEnvironment](/iis/configuration/system.applicationhost/applicationpools/add/processmodel#configuration) del pool di app. Il valore predefinito di `setProfileEnvironment` è `true`. In alcuni scenari (ad esempio, per il sistema operativo Windows), `setProfileEnvironment` è impostato su `false`. Se le chiavi non vengono archiviate nella directory del profilo utente come previsto:

  1. Passare alla cartella *%windir%/system32/inetsrv/config*.
  1. Aprire il file *applicationHost.config*.
  1. Individuare l'elemento `<system.applicationHost><applicationPools><applicationPoolDefaults><processModel>`.
  1. Verificare che l'attributo `setProfileEnvironment` non sia presente, condizione che corrisponde all'impostazione predefinita `true`, o impostare in modo esplicito il valore dell'attributo su `true`.

* **Usare il file system come archivio del gruppo di chiavi**

  Modificare il codice dell'app per [usare il file system come archivio del gruppo di chiavi](xref:security/data-protection/configuration/overview). Usare un certificato X509 per proteggere il gruppo di chiavi e verificare che sia un certificato attendibile. Se il certificato è autofirmato, inserirlo nell'archivio radice attendibile.

  Quando si usa IIS in una web farm:

  * Usare una condivisione di file a cui possono accedere tutti i computer.
  * Distribuire un certificato X509 in ogni computer. Configurare [protezione dei dati nel codice](xref:security/data-protection/configuration/overview).

* **Impostare criteri a livello di computer per la protezione dei dati**

  Il sistema di protezione dei dati ha un supporto limitato per l'impostazione di [criteri a livello di computer](xref:security/data-protection/configuration/machine-wide-policy) predefiniti per tutte le app che usano le API di protezione dei dati. Per altre informazioni, vedere <xref:security/data-protection/introduction>.

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

## <a name="configuration-of-iis-with-webconfig"></a>Configurazione di IIS con web.config

La configurazione di IIS è influenzata dalla sezione `<system.webServer>` di *web.config* per gli scenari IIS funzionali per le app ASP.NET Core con il modulo ASP.NET Core. Ad esempio, la configurazione di IIS è funzionale per la compressione dinamica. Se IIS è configurato a livello di server per l'uso della compressione dinamica, l'elemento `<urlCompression>` nel file *web.config* dell'app può disabilitare questa impostazione per un'app ASP.NET Core.

Per altre informazioni, vedere i seguenti argomenti:

* [Riferimento alla configurazione per \<system.webServer>](/iis/configuration/system.webServer/)
* <xref:host-and-deploy/aspnet-core-module>
* <xref:host-and-deploy/iis/modules>

Per impostare le variabili di ambiente per singole app in esecuzione in pool di applicazioni isolate (supportate per IIS 10,0 o versioni successive), vedere la sezione *comandoAppCmd.exe* dell'argomento [variabili \<environmentVariables> di ambiente](/iis/configuration/system.applicationHost/applicationPools/add/environmentVariables/#appcmdexe) nella documentazione di riferimento di IIS.

## <a name="configuration-sections-of-webconfig"></a>Sezioni di configurazione di web.config

Le sezioni di configurazione di app ASP.NET 4.x in *web.config* non vengono usate dalle app ASP.NET Core per la configurazione:

* `<system.web>`
* `<appSettings>`
* `<connectionStrings>`
* `<location>`

Le applicazioni ASP.NET Core vengono configurate tramite altri provider di configurazione. Per ulteriori informazioni, vedere [configurazione](xref:fundamentals/configuration/index).

## <a name="application-pools"></a>Pool di applicazioni

L'isolamento dei pool di app è determinato dal modello di hosting:

* Hosting in-process: le app sono necessarie per l'esecuzione in pool di applicazioni separati.
* Hosting out-of-process: è consigliabile isolare le app le une dalle altre eseguendo ogni app nel proprio pool di applicazioni.

Nella finestra di dialogo **Aggiungi sito Web** è selezionato per impostazione predefinita un singolo pool di app per ogni app. Quando si specifica un valore in **Nome del sito**, il testo viene automaticamente trasferito alla casella di testo **Pool di applicazioni**. Quando si aggiunge il sito viene creato un nuovo pool di applicazioni con il nome del sito.

## <a name="application-pool-identity"></a>Pool di applicazioni Identity

Un account di identità del pool di app consente di eseguire un'app in un account univoco, senza dover creare e gestire domini o account locali. In IIS 8.0 o versioni successive il processo di lavoro amministrazione IIS (WAS) crea un account virtuale con il nome del nuovo pool di applicazioni ed esegue i processi di lavoro del pool di applicazioni inclusi nell'account per impostazione predefinita. Nella console di gestione IIS in **Impostazioni avanzate** per il pool di applicazioni, verificare che **Identity** sia impostato per utilizzare **ApplicationPool Identity**:

![Finestra di dialogo Impostazioni avanzate del pool di applicazione](index/_static/apppool-identity.png)

Il processo di gestione IIS crea un identificatore sicuro con il nome del pool di app nel sistema di sicurezza di Windows. Le risorse possono essere protette mediante questa identità, che tuttavia non è un account utente reale e non viene visualizzata nella console di gestione utenti di Windows.

Se il processo di lavoro IIS richiede l'accesso con privilegi elevati all'app, modificare l'elenco di controllo di accesso (ACL) della directory contenente l'app:

1. Aprire Esplora risorse, quindi spostarsi nella directory.

1. Fare clic con il pulsante destro del mouse sulla directory e scegliere **Proprietà**.

1. Nella scheda **Sicurezza** selezionare il pulsante **Modifica** e quindi il pulsante **Aggiungi**.

1. Fare clic sul pulsante **Percorsi** e verificare che il sistema sia selezionato.

1. Immettere **IIS AppPool\\<nome_pool_applicazioni>** nell'area **Immettere i nomi degli oggetti da selezionare**. Fare clic sul pulsante **Controlla nomi**. Per *DefaultAppPool* controllare i nomi usando **IIS AppPool\DefaultAppPool**. Quando il pulsante **Controlla nomi** è selezionato, nell'area dei nomi degli oggetti viene indicato un valore di **DefaultAppPool**. Non è possibile immettere il nome del pool di applicazioni direttamente nell'area dei nomi degli oggetti. Usare il formato **IIS AppPool\\<nome_pool_applicazioni>** durante il controllo dei nomi degli oggetti.

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

Per una distribuzione In-Process, se viene stabilita una connessione HTTP/2, [HttpRequest.Protocol](xref:Microsoft.AspNetCore.Http.HttpRequest.Protocol*) corrisponde a `HTTP/2`. Per una distribuzione out-of-process, se viene stabilita una connessione HTTP/2, [HttpRequest.Protocol](xref:Microsoft.AspNetCore.Http.HttpRequest.Protocol*) corrisponde a `HTTP/1.1`.

Per altre informazioni sui modelli di hosting in-process e out-of-process, vedere <xref:host-and-deploy/aspnet-core-module>.

HTTP/2 è abilitato per impostazione predefinita. Se non viene stabilita una connessione HTTP/2, la connessione esegue il fallback a HTTP/1.1. Per altre informazioni sulla configurazione di HTTP/2 con le distribuzioni IIS, vedere [HTTP/2 on IIS](/iis/get-started/whats-new-in-iis-10/http2-on-iis) (HTTP/2 in IIS).

## <a name="cors-preflight-requests"></a>Richieste preliminari CORS

*Questa sezione si applica solo alle app ASP.NET Core destinate a .NET Framework.*

Per un'app ASP.NET Core destinata a .NET Framework, le richieste OPTIONS non vengono passate all'app per impostazione predefinita in IIS. Per informazioni su come configurare i gestori IIS dell'app in *web.config* per passare le richieste di opzioni, vedere [abilitare le richieste tra le origini in API Web ASP.NET 2: funzionamento di CORS](/aspnet/web-api/overview/security/enabling-cross-origin-requests-in-web-api#how-cors-works).

## <a name="application-initialization-module-and-idle-timeout"></a>Modulo di inizializzazione dell'applicazione e timeout di inattività

Per l'hosting in IIS dal modulo ASP.NET Core versione 2:

* [Modulo di inizializzazione dell'applicazione](#application-initialization-module): l'app è ospitata [in-process](#in-process-hosting-model) o [out-of-process](#out-of-process-hosting-model) può essere configurata per l'avvio automatico in un processo di lavoro o il riavvio del server.
* [Timeout di inattività](#idle-timeout): le app ospitate [in-process](#in-process-hosting-model) possono essere configurate in modo da non eseguire il timeout durante i periodi di inattività.

### <a name="application-initialization-module"></a>Modulo Inizializzazione applicazione

*Si applica alle app ospitate in-process e out-of-process.*

[Inizializzazione applicazione di IIS](/iis/get-started/whats-new-in-iis-8/iis-80-application-initialization) è una funzionalità di IIS che invia una richiesta HTTP all'app quando il pool di app viene avviato o riciclato. La richiesta attiva l'avvio dell'app. Per impostazione predefinita, IIS invia una richiesta all'URL radice dell'app (`/`) per inizializzare l'app (vedere le [risorse aggiuntive](#application-initialization-module-and-idle-timeout-additional-resources) per altri dettagli sulla configurazione).

Verificare che la funzionalità del ruolo Inizializzazione applicazione di IIS sia abilitata:

Nei sistemi desktop Windows 7 o versioni successive quando si usa IIS in locale:

1. Passare a **Pannello di controllo** > **Programmi** > **Programmi e funzionalità** > **Disattivare o attivare le funzionalità di Windows** (a sinistra dello schermo).
1. Aprire **Internet Information Services** > **Servizi Web** > **Funzionalità per lo sviluppo di applicazioni**.
1. Selezionare la casella di controllo **Inizializzazione applicazione**.

In Windows Server 2008 R2 o versioni successive:

1. Aprire l'**Aggiunta guidata ruoli e funzionalità**.
1. Nel pannello **Selezione servizi ruolo** aprire il nodo **Sviluppo applicazioni**.
1. Selezionare la casella di controllo **Inizializzazione applicazione**.

Per abilitare il modulo Inizializzazione applicazione per il sito, usare uno degli approcci seguenti:

* In Gestione IIS:

  1. Selezionare **Pool di applicazioni** nel pannello **Connessioni**.
  1. Fare clic con il pulsante destro del mouse sul pool di app dell'app nell'elenco e scegliere **Impostazioni avanzate**.
  1. L'impostazione predefinita per **Modalità avvio** è **OnDemand**. Impostare **Modalità avvio** su **AlwaysRunning**. Selezionare **OK**.
  1. Aprire il nodo **Siti** nel pannello **Connessioni**.
  1. Fare clic con il pulsante destro del mouse sull'app e scegliere **Gestisci sito Web** > **Impostazioni avanzate**.
  1. L'impostazione predefinita di **Precaricamento abilitato** è **False**. Impostare **Precaricamento abilitato** su **True**. Selezionare **OK**.

* Usando *web.config* aggiungere l'elemento `<applicationInitialization>` con `doAppInitAfterRestart` impostato su `true` per gli elementi `<system.webServer>` nel file *web.config* dell'app:

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
1. L'impostazione predefinita di **Timeout di inattività (minuti)** è **20** minuti. Impostare **Timeout di inattività (minuti)** su **0** (zero). Selezionare **OK**.
1. Riciclare il processo di lavoro.

Per evitare il timeout delle app ospitate [out-of-process](#out-of-process-hosting-model), usare uno degli approcci seguenti:

* Effettuare il ping dell'app da un servizio esterno per mantenerla in esecuzione.
* Se l'app ospita solo servizi in background, evitare l'hosting IIS e usare un [servizio Windows per ospitare l'app ASP.NET Core](xref:host-and-deploy/windows-service).

### <a name="application-initialization-module-and-idle-timeout-additional-resources"></a>Risorse aggiuntive per il modulo Inizializzazione applicazione e il timeout di inattività

* [IIS 8.0 Application Initialization](/iis/get-started/whats-new-in-iis-8/iis-80-application-initialization) (Inizializzazione delle applicazioni in IIS 8.0)
* [Inizializzazione \<applicationInitialization> dell'applicazione ](/iis/configuration/system.webserver/applicationinitialization/).
* [Impostazioni del modello di processo per un \<processModel> pool di applicazioni ](/iis/configuration/system.applicationhost/applicationpools/add/processmodel).

## <a name="deployment-resources-for-iis-administrators"></a>Risorse di distribuzione per amministratori IIS

* [Documentazione di ISS](/iis)
* [Getting Started with the IIS Manager in IIS](/iis/get-started/getting-started-with-iis/getting-started-with-the-iis-manager-in-iis-7-and-iis-8) (Introduzione a Gestione IIS in IIS)
* [Distribuzione di applicazioni .NET Core](/dotnet/core/deploying/)
* <xref:host-and-deploy/aspnet-core-module>
* <xref:host-and-deploy/directory-structure>
* <xref:host-and-deploy/iis/modules>
* <xref:test/troubleshoot-azure-iis>
* <xref:host-and-deploy/azure-iis-errors-reference>

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:test/troubleshoot>
* <xref:index>
* [Il sito IIS ufficiale di Microsoft](https://www.iis.net/)
* [Libreria di contenuti tecnici di Windows Server](/windows-server/windows-server)
* [HTTP/2 on IIS](/iis/get-started/whats-new-in-iis-10/http2-on-iis) (HTTP/2 in IIS)
* <xref:host-and-deploy/iis/transform-webconfig>

::: moniker-end

::: moniker range="< aspnetcore-2.2"

Per un'esercitazione sulla pubblicazione di un'app ASP.NET Core in un server IIS, vedere <xref:tutorials/publish-to-iis>.

[Installare il bundle di hosting .NET Core](#install-the-net-core-hosting-bundle)

## <a name="supported-operating-systems"></a>Sistemi operativi supportati

Sono supportati i sistemi operativi seguenti:

* Windows 7 o versione successiva
* Windows Server 2008 R2 o versione successiva

Il [server HTTP.sys](xref:fundamentals/servers/httpsys) (chiamato in precedenza WebListener) non funziona in una configurazione proxy inverso con IIS. È necessario usare il [server Kestrel](xref:fundamentals/servers/kestrel).

Per informazioni sull'hosting in Azure, vedere <xref:host-and-deploy/azure-apps/index>.

Per linee guida per la risoluzione dei problemi, vedere <xref:test/troubleshoot>.

## <a name="supported-platforms"></a>Piattaforme supportate

Sono supportate le app pubblicate per la distribuzione a 32 bit (x86) o a 64 bit (x64). Distribuire un'app a 32 bit con .NET Core SDK a 32 bit (x86), a meno che l'app:

* Non richieda lo spazio indirizzi di memoria virtuale più grande disponibile per un'app a 64 bit.
* Non richieda le dimensioni maggiori dello stack IIS.
* Non abbia dipendenze native a 64 bit.

Usare .NET Core SDK a 64 bit (x64) per pubblicare un'app a 64 bit. Un runtime a 64 bit deve essere presente nel sistema host.

ASP.NET Core viene fornito con il [server Kestrel](xref:fundamentals/servers/kestrel), ovvero un server HTTP multipiattaforma predefinito.

Quando si usa [IIS](/iis/get-started/introduction-to-iis/introduction-to-iis-architecture) o [IIS Express](/iis/extensions/introduction-to-iis-express/iis-express-overview), l'app viene eseguita in un processo separato dal processo di lavoro IIS (*out-of-process*) con il [server Kestrel](xref:fundamentals/servers/index#kestrel).

Poiché le app ASP.NET Core vengono eseguite in un processo distinto dal processo di lavoro IIS, il modulo esegue la gestione dei processi. Il modulo avvia il processo per l'app ASP.NET Core quando arriva la prima richiesta e riavvia l'app se viene arrestata o si arresta in modo anomalo. Si tratta essenzialmente dello stesso comportamento delle app eseguite in-process e gestite dal [servizio Attivazione processo Windows](/iis/manage/provisioning-and-managing-iis/features-of-the-windows-process-activation-service-was).

Il diagramma seguente illustra la relazione tra IIS, il modulo ASP.NET Core e un'app ospitata out-of-process:

![Modulo ASP.NET Core](index/_static/ancm-outofprocess.png)

Le richieste arrivano dal Web al driver HTTP.sys in modalità kernel. Il driver instrada le richieste a IIS sulla porta configurata per il sito Web, in genere 80 (HTTP) o 443 (HTTPS). Il modulo inoltra le richieste a Kestrel su una porta casuale per l'app non corrispondente alla porta 80 o 443.

Il modulo specifica la porta tramite una variabile di ambiente all'avvio e il [middleware di integrazione di IIS](xref:host-and-deploy/iis/index#enable-the-iisintegration-components) configura il server per l'ascolto `http://localhost:{port}` . Vengono eseguiti controlli aggiuntivi e le richieste che non provengono dal modulo vengono rifiutate. Il modulo non supporta l'inoltro HTTPS, pertanto le richieste vengono inoltrate tramite HTTP anche se sono state ricevute da IIS tramite HTTPS.

Dopo che Kestrel ha prelevato la richiesta dal modulo, viene eseguito il push della richiesta nella pipeline middleware ASP.NET Core. La pipeline middleware gestisce la richiesta e la passa come istanza di `HttpContext` alla logica dell'app. Il middleware aggiunto dall'integrazione di IIS aggiorna lo schema, l'IP remoto e il percorso di base all'account per l'inoltro della richiesta a Kestrel. La risposta dell'app viene quindi passata a IIS, che ne esegue di nuovo il push al client HTTP che ha avviato la richiesta.

`CreateDefaultBuilder` configura il server [Kestrel](xref:fundamentals/servers/kestrel) come server Web e abilita l'integrazione di IIS configurando il percorso base e la porta per il [modulo ASP.NET Core](xref:host-and-deploy/aspnet-core-module).

Il modulo ASP.NET Core genera una porta dinamica da assegnare al processo back-end. `CreateDefaultBuilder` chiama il metodo <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderIISExtensions.UseIISIntegration%2A>. `UseIISIntegration` configura Kestrel per l'ascolto sulla porta dinamica all'indirizzo IP localhost (`127.0.0.1`). Se la porta dinamica è 1234, Kestrel è in ascolto su `127.0.0.1:1234`. Questa configurazione sostituisce le altre configurazioni URL fornite da:

* `UseUrls`
* [API Listen di Kestrel](xref:fundamentals/servers/kestrel#endpoint-configuration)
* [Configurazione](xref:fundamentals/configuration/index) (o [opzione URL della riga di comando](xref:fundamentals/host/web-host#override-configuration))

Le chiamate a `UseUrls` o all'API `Listen` di Kestrel non sono necessarie quando si usa il modulo. In caso di chiamata di `UseUrls` o `Listen`, Kestrel resta in ascolto sulla porta specificata solo quando si esegue l'app senza IIS.

Per indicazioni sulla configurazione del modulo ASP.NET Core, vedere <xref:host-and-deploy/aspnet-core-module>.

Per altre informazioni sull'hosting, vedere [Hosting in ASP.NET Core](xref:fundamentals/index#host).

## <a name="application-configuration"></a>Configurazione dell'applicazione

### <a name="enable-the-iisintegration-components"></a>Abilitare i componenti IISIntegration

Quando si compila un host in `CreateWebHostBuilder` (*Program.cs*), chiamare <xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder%2A> per abilitare l'integrazione con IIS:

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        ...
```

Per altre informazioni su `CreateDefaultBuilder`, vedere <xref:fundamentals/host/web-host#set-up-a-host>.

### <a name="iis-options"></a>Opzioni IIS

| Opzione                         | Predefinito | Impostazione |
| ------------------------------ | :-----: | ------- |
| `AutomaticAuthentication`      | `true`  | Se `true`, IIS Server imposta l'utente `HttpContext.User` autenticato tramite l'[autenticazione di Windows](xref:security/authentication/windowsauth). Se `false`, il server fornisce solo un'identità per `HttpContext.User` e risponde alle richieste esplicite di `AuthenticationScheme`. Per il funzionamento di `AutomaticAuthentication` l’autenticazione di Windows deve essere abilitata in IIS. Per altre informazioni, vedere [Autenticazione di Windows](xref:security/authentication/windowsauth). |
| `AuthenticationDisplayName`    | `null`  | Imposta il nome visualizzato dagli utenti nelle pagine di accesso. |

Per configurare le opzioni di IIS includere una configurazione del servizio per <xref:Microsoft.AspNetCore.Builder.IISOptions> in <xref:Microsoft.AspNetCore.Hosting.IStartup.ConfigureServices*>. Nell'esempio seguente si impedisce all'app di popolare `HttpContext.Connection.ClientCertificate`:

```csharp
services.Configure<IISOptions>(options => 
{
    options.ForwardClientCertificate = false;
});
```

| Opzione                         | Predefinito | Impostazione |
| ------------------------------ | :-----: | ------- |
| `AutomaticAuthentication`      | `true`  | Se `true`, il [middleware di integrazione IIS](#enable-the-iisintegration-components) imposta `HttpContext.User` autenticato tramite l'[autenticazione di Windows](xref:security/authentication/windowsauth). Se `false`, il middleware fornisce solo un'identità per `HttpContext.User` e risponde solo alle richieste esplicite di `AuthenticationScheme`. Per il funzionamento di `AutomaticAuthentication` l’autenticazione di Windows deve essere abilitata in IIS. Per altre informazioni, vedere l'argomento [Autenticazione di Windows](xref:security/authentication/windowsauth). |
| `AuthenticationDisplayName`    | `null`  | Imposta il nome visualizzato dagli utenti nelle pagine di accesso. |
| `ForwardClientCertificate`     | `true`  | Se è `true` ed è presente l’intestazione della richiesta `MS-ASPNETCORE-CLIENTCERT`, `HttpContext.Connection.ClientCertificate` viene popolato. |

### <a name="proxy-server-and-load-balancer-scenarios"></a>Scenari con server proxy e servizi di bilanciamento del carico

Il [middleware di integrazione IIS](#enable-the-iisintegration-components), che consente di configurare il middleware delle intestazioni inoltrate, e il modulo ASP.NET Core sono configurati per inoltrare lo schema (HTTP/HTTPS) e l'indirizzo IP remoto di origine della richiesta. Potrebbero essere necessari interventi di configurazione aggiuntivi per le app ospitate dietro ulteriori server proxy e servizi di bilanciamento del carico. Per altre informazioni, vedere [Configurare ASP.NET Core per l'utilizzo di server proxy e servizi di bilanciamento del carico](xref:host-and-deploy/proxy-load-balancer).

### <a name="webconfig-file"></a>File web.config

Il file *web.config* configura il [modulo ASP.NET Core](xref:host-and-deploy/aspnet-core-module). La creazione, la trasformazione e la pubblicazione del file *web.config* sono operazioni gestite da una destinazione MSBuild (`_TransformWebConfig`) quando viene pubblicato il progetto. Questa destinazione si trova tra le destinazioni Web SDK (`Microsoft.NET.Sdk.Web`). L'SDK è impostato all'inizio del file di progetto:

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
```

Se nel progetto non è presente un file *web.config*, il file viene creato con un valore *processPath* e *argomenti* corretti per la configurazione del modulo ASP.NET Core. Il file viene quindi spostato nell'[output pubblicato](xref:host-and-deploy/directory-structure).

Se nel progetto è presente un file *web.config*, il file viene trasformato con il valore di *processPath* e gli *argomenti* corretti per la configurazione del modulo ASP.NET Core, quindi viene spostato nell'output pubblicato. La trasformazione non modifica le impostazioni di configurazione di IIS incluse nel file.

Il file *web.config* può fornire ulteriori impostazioni di configurazione di IIS che controllano i moduli IIS attivi. Per informazioni sui moduli IIS in grado di elaborare le richieste con le app di ASP.NET Core, vedere l'argomento [Moduli IIS](xref:host-and-deploy/iis/modules).

Per impedire a Web SDK di trasformare il file di *web.config* , utilizzare la **\<IsTransformWebConfigDisabled>** proprietà nel file di progetto:

```xml
<PropertyGroup>
  <IsTransformWebConfigDisabled>true</IsTransformWebConfigDisabled>
</PropertyGroup>
```

Quando si disabilita la trasformazione del file in Web SDK, il valore di *processPath* e gli *argomenti* devono essere impostati manualmente dallo sviluppatore. Per altre informazioni, vedere <xref:host-and-deploy/aspnet-core-module>.

### <a name="webconfig-file-location"></a>Posizione del file web.config

Per configurare correttamente il [modulo ASP.NET Core](xref:host-and-deploy/aspnet-core-module) , il file di *web.config* deve essere presente nel percorso radice del [contenuto](xref:fundamentals/index#content-root) (in genere il percorso di base dell'app) dell'app distribuita. Corrisponde al percorso fisico del sito Web fornito a IIS. Il file *web.config* deve essere presente nella radice dell'app per abilitare la pubblicazione di più app mediante Distribuzione Web.

I file sensibili sono presenti nel percorso fisico dell'app, ad esempio *\<assembly>.runtimeconfig.json*, *\<assembly> XML* (commenti in formato documentazione XML) e *\<assembly>.deps.jssu*. Quando il file *web.config* è presente e il sito viene avviato normalmente, IIS non rende disponibili questi file riservati se vengono richiesti. Se il file *web.config* non è presente, ha un nome non corretto o non è in grado di configurare il sito per l'avvio normale, IIS potrebbe rendere disponibili pubblicamente i file riservati.

**Il file di *web.config* deve essere presente nella distribuzione in qualsiasi momento, denominato correttamente e in grado di configurare il sito per l'avvio normale. Non rimuovere mai il file di *web.config* da una distribuzione di produzione.**

### <a name="transform-webconfig"></a>Trasformare web.config

Se è necessario trasformare *web.config* in fase di pubblicazione (ad esempio, impostare variabili di ambiente in base a configurazione, profilo o ambiente), vedere <xref:host-and-deploy/iis/transform-webconfig>.

## <a name="iis-configuration"></a>Configurazione di IIS

**Sistemi operativi Windows Server**

Abilitare il ruolo del server **Server Web (IIS)** e stabilire i servizi di ruolo.

1. Usare la procedura guidata **Aggiungi ruoli e funzionalità** accessibile tramite il menu **Gestisci** o il collegamento in **Server Manager**. Nel passaggio **Ruoli del server** selezionare la casella per **Server Web (IIS)**.

   ![Il ruolo Server Web IIS viene selezionato nel passaggio dei ruoli del server Selezionare.](index/_static/server-roles-ws2016.png)

1. Dopo il passaggio **Funzionalità**, viene caricato il passaggio **Servizi ruolo** per Server Web (IIS). Selezionare i servizi ruolo IIS desiderati o accettare i servizi ruolo predefiniti specificati.

   ![I servizi ruolo predefiniti vengono selezionati nel passaggio Selezionare i servizi ruolo.](index/_static/role-services-ws2016.png)

   **Autenticazione di Windows (facoltativa)**  
   Per abilitare l'autenticazione di Windows, espandere i nodi seguenti: sicurezza del **server Web**  >  . Selezionare la funzionalità **Autenticazione di Windows**. Per ulteriori informazioni, vedere [ \<windowsAuthentication> autenticazione di Windows](/iis/configuration/system.webServer/security/authentication/windowsAuthentication/) e [configurare l'autenticazione di Windows](xref:security/authentication/windowsauth).

   **WebSockets (facoltativo)**  
   WebSockets è supportato con ASP.NET Core 1.1 o versioni successive. Per abilitare i WebSocket, espandere i nodi seguenti: sviluppo di applicazioni **server Web**  >  . Selezionare la funzionalità **Protocollo WebSocket**. Per altre informazioni, vedere [Oggetti WebSocket](xref:fundamentals/websockets).

1. Procedere con il passaggio **Conferma** per installare il ruolo del server web e i servizi. Dopo l'installazione del ruolo **server Web (IIS)** non è necessario riavviare il server o IIS.

**Sistemi operativi desktop di Windows**

Abilitare **Console di gestione IIS** e **Servizi Web**.

1. Passare a **Pannello di controllo** > **Programmi** > **Programmi e funzionalità** > **Disattivare o attivare le funzionalità di Windows** (a sinistra dello schermo).

1. Aprire il nodo **Internet Information Services**. Aprire il nodo **Strumenti di gestione Web**.

1. Selezionare la casella per **Console di gestione IIS**.

1. Selezionare la casella per **Servizi World Wide Web**.

1. Accettare le funzionalità predefinite per **Servizi World Wide Web** o personalizzare le funzionalità IIS.

   **Autenticazione di Windows (facoltativa)**  
   Per abilitare l'autenticazione di Windows, espandere i nodi seguenti: **World Wide Web sicurezza dei servizi**  >  . Selezionare la funzionalità **Autenticazione di Windows**. Per ulteriori informazioni, vedere [ \<windowsAuthentication> autenticazione di Windows](/iis/configuration/system.webServer/security/authentication/windowsAuthentication/) e [configurare l'autenticazione di Windows](xref:security/authentication/windowsauth).

   **WebSockets (facoltativo)**  
   WebSockets è supportato con ASP.NET Core 1.1 o versioni successive. Per abilitare i WebSocket, espandere i nodi seguenti: funzionalità di sviluppo di applicazioni di **World Wide Web Services**  >  . Selezionare la funzionalità **Protocollo WebSocket**. Per altre informazioni, vedere [Oggetti WebSocket](xref:fundamentals/websockets).

1. Se l'installazione di IIS richiede un riavvio, riavviare il sistema.

![La console di gestione IIS e servizi World Wide Web sono selezionati nelle funzionalità di Windows.](index/_static/windows-features-win10.png)

## <a name="install-the-net-core-hosting-bundle"></a>Installare il bundle di hosting .NET Core

Installare il *bundle di hosting .NET Core* nel sistema di hosting. Il bundle installa il runtime di .NET Core, la libreria di .NET Core e il [modulo ASP.NET Core](xref:host-and-deploy/aspnet-core-module). Il modulo consente l'esecuzione delle app ASP.NET Core dietro IIS.

> [!IMPORTANT]
> Se il bundle di hosting viene installato prima di IIS, è necessario riparare l'installazione del bundle. Eseguire di nuovo il programma di installazione del bundle di hosting dopo l'installazione di IIS.
>
> Se il bundle di hosting viene installato dopo l'installazione della versione a 64 bit (x64) di .NET Core, uno o più SDK potrebbero risultare mancanti ([Non sono stati rilevati .NET Core SDK](xref:test/troubleshoot#no-net-core-sdks-were-detected)). Per risolvere il problema, vedere <xref:test/troubleshoot#missing-sdk-after-installing-the-net-core-hosting-bundle>.

### <a name="download"></a>Scarica

1. Passare alla pagina [download di .NET Core](https://dotnet.microsoft.com/download/dotnet-core) .
1. Selezionare la versione desiderata di .NET Core.
1. Nella colonna **Run apps - Runtime** (App di esecuzione - Runtime), trovare la riga della versione di runtime di .NET Core desiderata.
1. Scaricare il programma di installazione usando il collegamento **bundle di hosting** .

> [!WARNING]
> Alcuni programmi di installazione contengono versioni che hanno terminato il loro ciclo di vita e non sono più supportate da Microsoft. Per altre informazioni, vedere i [criteri di supporto](https://dotnet.microsoft.com/platform/support/policy/dotnet-core).

### <a name="install-the-hosting-bundle"></a>Installare il bundle di hosting

1. Eseguire il programma di installazione nel server. Quando si esegue il programma di installazione da una shell dei comandi di amministratore sono disponibili i parametri seguenti:

   * `OPT_NO_ANCM=1`: Ignorare l'installazione del modulo ASP.NET Core.
   * `OPT_NO_RUNTIME=1`: Ignorare l'installazione del runtime di .NET Core. Utilizzato quando il server ospita solo le [distribuzioni autosufficienti (SCD)](/dotnet/core/deploying/#self-contained-deployments-scd).
   * `OPT_NO_SHAREDFX=1`: Ignora l'installazione del Framework condiviso ASP.NET (runtime ASP.NET). Utilizzato quando il server ospita solo le [distribuzioni autosufficienti (SCD)](/dotnet/core/deploying/#self-contained-deployments-scd).
   * `OPT_NO_X86=1`: Ignora l'installazione di Runtime x86. Usare questo parametro se si è certi che non verrà eseguito l'hosting di app a 32 bit. Se non esiste alcuna possibilità che in futuro venga eseguito l'hosting di app sia a 32 che a 64 bit, non usare questo parametro e installare entrambi i runtime.
   * `OPT_NO_SHARED_CONFIG_CHECK=1`: Disabilitare il controllo per l'utilizzo di una configurazione condivisa di IIS quando la configurazione condivisa (*applicationHost.config*) si trova nello stesso computer dell'installazione di IIS. *Disponibile solo per i programmi di installazione di bundler di hosting ASP.NET Core 2.2 o versioni successive.* Per altre informazioni, vedere <xref:host-and-deploy/aspnet-core-module#aspnet-core-module-with-an-iis-shared-configuration>.
1. Riavviare il sistema o eseguire i comandi seguenti in una shell dei comandi:

   ```console
   net stop was /y
   net start w3svc
   ```
   Il riavvio di IIS rileva una modifica alla variabile di ambiente di sistema PATH apportata dal programma di installazione.

Non è necessario arrestare manualmente singoli siti in IIS durante l'installazione del bundle di hosting. Le app ospitate (siti IIS) vengono riavviate quando IIS viene riavviato. Le app vengono riavviate quando ricevono la prima richiesta, incluso il [modulo di inizializzazione dell'applicazione](#application-initialization-module-and-idle-timeout).

ASP.NET Core adotta il comportamento di rollforward per le versioni di patch dei pacchetti di Framework condivisi. Quando le app ospitate da IIS vengono riavviate con IIS, le app vengono caricate con le ultime versioni di patch dei pacchetti a cui si fa riferimento quando ricevono la prima richiesta. Se IIS non viene riavviato, le app vengono riavviate e presentano un comportamento di rollforward quando i processi di lavoro vengono riciclati e ricevono la prima richiesta.

> [!NOTE]
> Per informazioni sulla configurazione condivisa di IIS, vedere [Modulo di ASP.NET Core con configurazione condivisa di IIS](xref:host-and-deploy/aspnet-core-module#aspnet-core-module-with-an-iis-shared-configuration).

## <a name="install-web-deploy-when-publishing-with-visual-studio"></a>Installare la Distribuzione Web quando si pubblica con Visual Studio

Quando si distribuiscono app ai server con [Distribuzione Web](/iis/install/installing-publishing-technologies/installing-and-configuring-web-deploy-on-iis-80-or-later), installare la versione più recente di Distribuzione Web nel server. Per installare Distribuzione Web usare l'[Installazione guidata piattaforma Web (WebPI)](https://www.microsoft.com/web/downloads/platform.aspx) o ottenere direttamente un programma di installazione in [Microsoft Download Center](https://www.microsoft.com/download/details.aspx?id=43717). Il metodo preferito consiste nell'usare WebPI. WebPI offre un'installazione autonoma e una configurazione per i provider di hosting.

## <a name="create-the-iis-site"></a>Creare il sito IIS

1. Nel sistema host creare una cartella per contenere le cartelle e i file pubblicati dell'app. In un passaggio successivo, il percorso della cartella viene fornito a IIS come percorso fisico dell'app. Per altre informazioni sulla cartella di distribuzione e il layout dei file di un'app, vedere <xref:host-and-deploy/directory-structure>.

1. In Gestione IIS aprire il nodo del server nel pannello **connessioni** . Fare clic con il pulsante destro del mouse sulla cartella **Siti**. Scegliere **Aggiungi sito Web** dal menu di scelta rapida.

1. Specificare un **Nome del sito** e impostare il **Percorso fisico** per la cartella di distribuzione dell'app. Specificare la configurazione dell' **associazione** e creare il sito Web selezionando **OK**:

   ![Specificare il nome del sito, il percorso fisico e il nome Host nel passaggio Aggiungi sito Web.](index/_static/add-website-ws2016.png)

   > [!WARNING]
   > Le associazioni con caratteri jolly di livello superiore (`http://*:80/` e `http://+:80`) **non** devono essere usate, poiché possono introdurre vulnerabilità a livello di sicurezza nell'app. Questo concetto vale sia per i caratteri jolly sicuri che vulnerabili. Usare nomi host espliciti al posto di caratteri jolly. L'associazione con caratteri jolly del sottodominio (ad esempio, `*.mysub.com`) non costituisce un rischio per la sicurezza se viene controllato l'intero dominio padre (a differenza di `*.com`, che è vulnerabile). Vedere la [sezione 5.4 di RFC7230](https://tools.ietf.org/html/rfc7230#section-5.4) per altre informazioni.

1. Nel nodo del server selezionare **Pool di applicazioni**.

1. Fare clic con il pulsante destro del mouse sul pool di applicazioni del sito e scegliere **Impostazioni di base** dal menu di scelta rapida.

1. Nella finestra **Modifica pool di applicazioni** impostare **Versione .NET CLR** su **Nessun codice gestito**.

   ![Impostare Nessun codice gestito per la versione .NET CLR.](index/_static/edit-apppool-ws2016.png)

    ASP.NET Core viene eseguito in un processo separato e gestisce il runtime. ASP.NET Core non si basa sul caricamento di Common Language Runtime (CLR di .NET) desktop. Core Common Language Runtime (CoreCLR) for .NET Core viene avviato per ospitare l'app nel processo di lavoro. L'impostazione di **Versione .NET CLR** su **Nessun codice gestito** è facoltativa ma consigliata.

1. *ASP.NET Core 2.2 o versioni successive*: per una [distribuzione autonoma](/dotnet/core/deploying/#self-contained-deployments-scd) a 64 bit (x64) che usa il [modello di hosting In-Process](#in-process-hosting-model), disabilitare il pool di app per i processi a 32 bit (x86).

   Nella barra laterale **Azioni** in Gestione IIS > **Pool di applicazioni** selezionare **Impostazioni predefinite pool di applicazioni** o **Impostazioni avanzate**. Individuare **Attiva applicazioni a 32 bit** e impostare il valore su `False`. Questa impostazione non viene applicata alle app distribuite per l'[hosting out-of-process](xref:host-and-deploy/aspnet-core-module#out-of-process-hosting-model).

1. Confermare che l'identità del modello del processo disponga delle autorizzazioni appropriate.

   Se l'identità predefinita del pool di app (**modello**  >  **Identity** di processo) viene modificata **da Identity ApplicationPool** a un'altra identità, verificare che la nuova identità disponga delle autorizzazioni necessarie per accedere alla cartella dell'app, al database e ad altre risorse richieste. Ad esempio, il pool di applicazioni richiede l'accesso in lettura e scrittura alle cartelle in cui l'app legge e scrive i file.

**Configurazione dell'autenticazione di Windows (facoltativa)**  
Per altre informazioni, vedere [Configurare l'autenticazione Windows](xref:security/authentication/windowsauth).

## <a name="deploy-the-app"></a>Distribuire l'app

Distribuire l'app nella cartella **Percorso fisico** di IIS specificata nella sezione [Creare il sito IIS](#create-the-iis-site). [Distribuzione Web](/iis/publish/using-web-deploy/introduction-to-web-deploy) è il meccanismo consigliato per la distribuzione, ma esistono diverse opzioni per spostare l'app dalla cartella di *pubblicazione* del progetto alla cartella di distribuzione del sistema di hosting.

### <a name="web-deploy-with-visual-studio"></a>Distribuzione web con Visual Studio

Vedere l'argomento [Visual Studio publish profiles for ASP.NET Core app deployment](xref:host-and-deploy/visual-studio-publish-profiles#publish-profiles) (Profili di pubblicazione Visual Studio per la distribuzione di app ASP.NET Core) per informazioni su come creare un profilo di pubblicazione da usare con Distribuzione Web. Se il provider di hosting fornisce un profilo di pubblicazione o il supporto per crearne uno, scaricare il profilo e importarlo usando la finestra di dialogo **Pubblica** di Visual Studio:

![Pagina della finestra di dialogo Pubblica](index/_static/pub-dialog.png)

### <a name="web-deploy-outside-of-visual-studio"></a>Distribuzione Web all'esterno di Visual Studio

È anche possibile usare [Distribuzione Web](/iis/publish/using-web-deploy/introduction-to-web-deploy) all'esterno di Visual Studio dalla riga di comando. Per altre informazioni sulla distribuzione, vedere [Strumento Distribuzione Web](/iis/publish/using-web-deploy/use-the-web-deployment-tool).

### <a name="alternatives-to-web-deploy"></a>Alternative a Distribuzione web

Usare uno dei metodi disponibili, ad esempio la copia manuale, [Xcopy](/windows-server/administration/windows-commands/xcopy), [Robocopy](/windows-server/administration/windows-commands/robocopy) o [PowerShell](/powershell/), per spostare l'app nel sistema di hosting.

Per altre informazioni sulla distribuzione di ASP.NET Core in IIS, vedere la sezione [Risorse di distribuzione per amministratori IIS](#deployment-resources-for-iis-administrators).

## <a name="browse-the-website"></a>Esplorare il sito Web

Dopo aver distribuito l'app nel sistema di hosting, effettuare una richiesta a uno degli endpoint pubblici dell'app.

Nell'esempio seguente il sito è associato al **nome host** di IIS `www.mysite.com` sulla **porta** `80`. Viene effettuata una richiesta a `http://www.mysite.com`:

![Il browser Microsoft Edge ha caricato la pagina di avvio IIS.](index/_static/browsewebsite.png)

## <a name="locked-deployment-files"></a>File di distribuzione bloccati

I file nella cartella di distribuzione sono bloccati durante l'esecuzione dell'app. I file bloccati non possono essere sovrascritti durante la distribuzione. Per rilasciare i file bloccati in una distribuzione, arrestare il pool di applicazioni usando **uno** dei metodi seguenti:

* Usare Distribuzione Web e fare riferimento a `Microsoft.NET.Sdk.Web` nel file di progetto. Nella radice della directory dell'app web viene posizionato un file *app_offline.htm*. Quando il file è presente, il modulo ASP.NET Core arresta normalmente l'app e rende disponibile il file *app_offline.htm* durante la distribuzione. Per altre informazioni, vedere la [Guida di riferimento per la configurazione del modulo ASP.NET Core](xref:host-and-deploy/aspnet-core-module#app_offlinehtm).
* Arrestare manualmente il pool di applicazioni in Gestione IIS sul server.
* Usare PowerShell per eliminare *app_offline.htm* (richiede PowerShell 5 o versione successiva):

  ```powershell
  $pathToApp = 'PATH_TO_APP'

  # Stop the AppPool
  New-Item -Path $pathToApp app_offline.htm

  # Provide script commands here to deploy the app

  # Restart the AppPool
  Remove-Item -Path $pathToApp app_offline.htm

  ```

## <a name="data-protection"></a>Protezione dei dati

Lo [stack di protezione dei dati di ASP.NET Core](xref:security/data-protection/introduction) viene usato da diversi [middleware](xref:fundamentals/middleware/index) di ASP.NET Core, inclusi quelli usati nell'autenticazione. Anche se le DPAPI (Data Protection API) non vengono chiamate dal codice dell'utente, è consigliabile configurare la protezione dati per la creazione di un [archivio di chiavi](xref:security/data-protection/implementation/key-management) crittografiche permanente con uno script di distribuzione o nel codice dell'utente. Se non si configura la protezione dei dati, le chiavi vengono mantenute in memoria ed eliminate al riavvio dell'app.

Se il gruppo di chiavi viene archiviato in memoria quando l'app viene riavviata:

* Tutti i cookie token di autenticazione basati su vengono invalidati. 
* Gli utenti devono ripetere l'accesso alla richiesta successiva. 
* Tutti i dati protetti con il gruppo di chiavi non possono più essere decrittografati. Questo può includere [i token CSRF](xref:security/anti-request-forgery#aspnet-core-antiforgery-configuration) e [ASP.NET Core TempData MVC cookie ](xref:fundamentals/app-state#tempdata).

Per configurare la protezione dati in IIS in modo da rendere permanente il gruppo di chiavi, usare **uno** degli approcci seguenti:

* **Creare chiavi del Registro di sistema di protezione dati**

  Le chiavi di protezione dati usate dalle app ASP.NET Core vengono archiviate nel Registro di sistema esterno alle app. Per mantenere le chiavi per una determinata app, creare chiavi del Registro di sistema per il pool di applicazioni.

  Per le installazioni IIS autonome non web farm è possibile usare lo [script PowerShell Data Protection Provision-AutoGenKeys.ps1 (Provisioning di protezione dati-AutoGenKeys.ps1)](https://github.com/dotnet/AspNetCore/blob/main/src/DataProtection/Provision-AutoGenKeys.ps1) per ogni pool di app usato con un'app ASP.NET Core. Questo script crea una chiave del Registro di sistema nel registro HKLM accessibile solo all'account del processo di lavoro del pool di applicazioni dell'app. Le chiavi vengono crittografate a riposo tramite DPAPI con una chiave a livello del computer.

  In scenari di web farm un'app può essere configurata in modo da usare un percorso UNC in cui archiviare il proprio gruppo di chiavi di protezione dati. Per impostazione predefinita, le chiavi di protezione dati non vengono crittografate. Assicurarsi che le autorizzazioni file per la condivisione di rete siano limitate all'account di Windows in cui viene eseguita l'app. È possibile usare un certificato X509 per la protezione delle chiavi inattive. Considerare l'implementazione di un meccanismo per consentire agli utenti di caricare i certificati: inserire i certificati nell'archivio certificati attendibili dell'utente e assicurarsi che siano disponibili in tutti i computer in cui viene eseguita l'app dell'utente. Vedere [Configurare la protezione dati di ASP.NET Core](xref:security/data-protection/configuration/overview) per altri dettagli.

* **Configurare il pool di applicazioni IIS per caricare il profilo utente**

  Questa impostazione si trova nella sezione **Modello del processo** in **Impostazioni avanzate** per il pool di app. Impostare **Caricamento del profilo utente** su `True`. Con l'impostazione `True` le chiavi vengono archiviate nella directory del profilo utente e protette tramite DPAPI con una chiave specifica per l'account utente. Le chiavi vengono salvate in modo permanente nella cartella *%LOCALAPPDATA%/ASP.NET/DataProtection-Keys*.

  Deve essere abilitato anche l'[attributo setProfileEnvironment](/iis/configuration/system.applicationhost/applicationpools/add/processmodel#configuration) del pool di app. Il valore predefinito di `setProfileEnvironment` è `true`. In alcuni scenari (ad esempio, per il sistema operativo Windows), `setProfileEnvironment` è impostato su `false`. Se le chiavi non vengono archiviate nella directory del profilo utente come previsto:

  1. Passare alla cartella *%windir%/system32/inetsrv/config*.
  1. Aprire il file *applicationHost.config*.
  1. Individuare l'elemento `<system.applicationHost><applicationPools><applicationPoolDefaults><processModel>`.
  1. Verificare che l'attributo `setProfileEnvironment` non sia presente, condizione che corrisponde all'impostazione predefinita `true`, o impostare in modo esplicito il valore dell'attributo su `true`.

* **Usare il file system come archivio del gruppo di chiavi**

  Modificare il codice dell'app per [usare il file system come archivio del gruppo di chiavi](xref:security/data-protection/configuration/overview). Usare un certificato X509 per proteggere il gruppo di chiavi e verificare che sia un certificato attendibile. Se il certificato è autofirmato, inserirlo nell'archivio radice attendibile.

  Quando si usa IIS in una web farm:

  * Usare una condivisione di file a cui possono accedere tutti i computer.
  * Distribuire un certificato X509 in ogni computer. Configurare [protezione dei dati nel codice](xref:security/data-protection/configuration/overview).

* **Impostare criteri a livello di computer per la protezione dei dati**

  Il sistema di protezione dei dati ha un supporto limitato per l'impostazione di [criteri a livello di computer](xref:security/data-protection/configuration/machine-wide-policy) predefiniti per tutte le app che usano le API di protezione dei dati. Per altre informazioni, vedere <xref:security/data-protection/introduction>.

## <a name="virtual-directories"></a>Directory virtuali

Le [directory virtuali IIS](/iis/get-started/planning-your-iis-architecture/understanding-sites-applications-and-virtual-directories-on-iis#virtual-directories) non sono supportate con le app ASP.NET Core. Un'app può essere ospitata come [applicazione secondaria](#sub-applications).

## <a name="sub-applications"></a>Applicazioni secondarie

Un'app ASP.NET Core può essere ospitata come [applicazione secondaria di IIS (app secondaria)](/iis/get-started/planning-your-iis-architecture/understanding-sites-applications-and-virtual-directories-on-iis#applications). Il percorso dell'app secondaria diventa parte dell'URL dell'app radice.

Un'app secondaria non deve includere il modulo ASP.NET Core come gestore. Se il modulo viene aggiunto come gestore in un file di *web.config* dell'applicazione secondaria, viene ricevuto un *errore 500,19 interno del server* che fa riferimento al file di configurazione difettoso durante il tentativo di esplorare la sottoapp.

L'esempio seguente illustra un file *web.config* pubblicato per un'app secondaria di ASP.NET Core:

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <system.webServer>
    <aspNetCore processPath="dotnet" 
      arguments=".\MyApp.dll" 
      stdoutLogEnabled="false" 
      stdoutLogFile=".\logs\stdout" />
  </system.webServer>
</configuration>
```

Quando si ospita un'app secondaria non ASP.NET Core in un'app ASP.NET Core, è necessario rimuovere in modo esplicito il gestore ereditato nel file *web.config* dell'app secondaria:

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <system.webServer>
    <handlers>
      <remove name="aspNetCore" />
    </handlers>
    <aspNetCore processPath="dotnet" 
      arguments=".\MyApp.dll" 
      stdoutLogEnabled="false" 
      stdoutLogFile=".\logs\stdout" />
  </system.webServer>
</configuration>
```

I collegamenti ad asset statici all'interno dell'app secondaria devono usare la notazione con tilde-barra (`~/`). La notazione con tilde-barra attiva un [helper tag](xref:mvc/views/tag-helpers/intro) per anteporre la base del percorso dell'app secondaria al collegamento relativo sottoposto a rendering. Per un'app secondaria in `/subapp_path`, il rendering di un'immagine collegata con `src="~/image.png"` viene eseguito come `src="/subapp_path/image.png"`. Il middleware dei file statici dell'app radice non elabora la richiesta di file statici. La richiesta viene elaborata dal middleware dei file statici dell'app secondaria.

Se l'attributo `src` di un asset statico viene impostato su un percorso assoluto (ad esempio, `src="/image.png"`), il rendering del collegamento viene eseguito senza la base del percorso dell'app secondaria. Il middleware dei file statici dell'app radice prova a servire l'asset dalla [radice Web](xref:fundamentals/index#web-root) dell'app radice con conseguente risposta *404 - Non trovato*, a meno che l'asset statico non sia disponibile dal'app radice.

Per ospitare un'app ASP.NET Core come app secondaria in un'altra app ASP.NET Core:

1. Definire un pool di app per l'app secondaria. Impostare **Versione .NET CLR** su **Nessun codice gestito** perché Core Common Language Runtime (CoreCLR) for .NET Core viene avviato per ospitare l'app nel processo di lavoro e non CLR desktop (.NET CLR).

1. Aggiungere il sito radice in Gestione IIS con l'applicazione secondaria in una cartella all'interno del sito principale.

1. Fare clic con il pulsante destro del mouse sulla cartella dell'applicazione secondaria in Gestione IIS e scegliere **Converti in applicazione**.

1. Nella finestra di dialogo **Aggiungi applicazione** usare il pulsante **Seleziona** per **Pool di applicazioni** per assegnare il pool di app creato per l'app secondaria. Selezionare **OK**.

L'assegnazione di un pool di app separato all'app secondaria è un requisito quando si usa il modello di hosting in-process.

Per ulteriori informazioni sul modello di hosting in-process e sulla configurazione del modulo ASP.NET Core, vedere <xref:host-and-deploy/aspnet-core-module> .

## <a name="configuration-of-iis-with-webconfig"></a>Configurazione di IIS con web.config

La configurazione di IIS è influenzata dalla sezione `<system.webServer>` di *web.config* per gli scenari IIS funzionali per le app ASP.NET Core con il modulo ASP.NET Core. Ad esempio, la configurazione di IIS è funzionale per la compressione dinamica. Se IIS è configurato a livello di server per l'uso della compressione dinamica, l'elemento `<urlCompression>` nel file *web.config* dell'app può disabilitare questa impostazione per un'app ASP.NET Core.

Per altre informazioni, vedere i seguenti argomenti:

* [Riferimento alla configurazione per \<system.webServer>](/iis/configuration/system.webServer/)
* <xref:host-and-deploy/aspnet-core-module>
* <xref:host-and-deploy/iis/modules>

Per impostare le variabili di ambiente per singole app in esecuzione in pool di applicazioni isolate (supportate per IIS 10,0 o versioni successive), vedere la sezione *comandoAppCmd.exe* dell'argomento [variabili \<environmentVariables> di ambiente](/iis/configuration/system.applicationHost/applicationPools/add/environmentVariables/#appcmdexe) nella documentazione di riferimento di IIS.

## <a name="configuration-sections-of-webconfig"></a>Sezioni di configurazione di web.config

Le sezioni di configurazione di app ASP.NET 4.x in *web.config* non vengono usate dalle app ASP.NET Core per la configurazione:

* `<system.web>`
* `<appSettings>`
* `<connectionStrings>`
* `<location>`

Le applicazioni ASP.NET Core vengono configurate tramite altri provider di configurazione. Per ulteriori informazioni, vedere [configurazione](xref:fundamentals/configuration/index).

## <a name="application-pools"></a>Pool di applicazioni

Quando si ospitano più siti Web in un server, è consigliabile isolare le app le une dalle altre eseguendo ogni app nel relativo pool di applicazioni. La finestra di dialogo **Aggiungi sito Web** di IIS ha questa configurazione come impostazione predefinita. Quando si specifica un valore in **Nome del sito**, il testo viene automaticamente trasferito alla casella di testo **Pool di applicazioni**. Quando si aggiunge il sito viene creato un nuovo pool di applicazioni con il nome del sito.

## <a name="application-pool-identity"></a>Pool di applicazioni Identity

Un account di identità del pool di app consente di eseguire un'app in un account univoco, senza dover creare e gestire domini o account locali. In IIS 8.0 o versioni successive il processo di lavoro amministrazione IIS (WAS) crea un account virtuale con il nome del nuovo pool di applicazioni ed esegue i processi di lavoro del pool di applicazioni inclusi nell'account per impostazione predefinita. Nella console di gestione IIS in **Impostazioni avanzate** per il pool di applicazioni, verificare che **Identity** sia impostato per utilizzare **ApplicationPool Identity**:

![Finestra di dialogo Impostazioni avanzate del pool di applicazione](index/_static/apppool-identity.png)

Il processo di gestione IIS crea un identificatore sicuro con il nome del pool di app nel sistema di sicurezza di Windows. Le risorse possono essere protette mediante questa identità, che tuttavia non è un account utente reale e non viene visualizzata nella console di gestione utenti di Windows.

Se il processo di lavoro IIS richiede l'accesso con privilegi elevati all'app, modificare l'elenco di controllo di accesso (ACL) della directory contenente l'app:

1. Aprire Esplora risorse, quindi spostarsi nella directory.

1. Fare clic con il pulsante destro del mouse sulla directory e scegliere **Proprietà**.

1. Nella scheda **Sicurezza** selezionare il pulsante **Modifica** e quindi il pulsante **Aggiungi**.

1. Fare clic sul pulsante **Percorsi** e verificare che il sistema sia selezionato.

1. Immettere **IIS AppPool\\<nome_pool_applicazioni>** nell'area **Immettere i nomi degli oggetti da selezionare**. Fare clic sul pulsante **Controlla nomi**. Per *DefaultAppPool* controllare i nomi usando **IIS AppPool\DefaultAppPool**. Quando il pulsante **Controlla nomi** è selezionato, nell'area dei nomi degli oggetti viene indicato un valore di **DefaultAppPool**. Non è possibile immettere il nome del pool di applicazioni direttamente nell'area dei nomi degli oggetti. Usare il formato **IIS AppPool\\<nome_pool_applicazioni>** durante il controllo dei nomi degli oggetti.

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

[HTTP/2](https://httpwg.org/specs/rfc7540.html) è supportato per le distribuzioni out-of-process che soddisfano i requisiti seguenti:

* Windows Server 2016/Windows 10 o versioni successive; IIS 10 o versioni successive
* Le connessioni server perimetrali pubbliche usano HTTP/2, mentre la connessione con proxy inverso al [server Kestrel](xref:fundamentals/servers/kestrel) usa HTTP/1.1.
* Frame di destinazione: non applicabile alle distribuzioni out-of-process, in quanto la connessione HTTP/2 è gestita interamente da IIS.
* Connessione TLS 1.2 o successiva

Se viene stabilita una connessione HTTP/2, [HttpRequest.Protocol](xref:Microsoft.AspNetCore.Http.HttpRequest.Protocol*) corrisponde a `HTTP/1.1`.

HTTP/2 è abilitato per impostazione predefinita. Se non viene stabilita una connessione HTTP/2, la connessione esegue il fallback a HTTP/1.1. Per altre informazioni sulla configurazione di HTTP/2 con le distribuzioni IIS, vedere [HTTP/2 on IIS](/iis/get-started/whats-new-in-iis-10/http2-on-iis) (HTTP/2 in IIS).

## <a name="cors-preflight-requests"></a>Richieste preliminari CORS

*Questa sezione si applica solo alle app ASP.NET Core destinate a .NET Framework.*

Per un'app ASP.NET Core destinata a .NET Framework, le richieste OPTIONS non vengono passate all'app per impostazione predefinita in IIS. Per informazioni su come configurare i gestori IIS dell'app in *web.config* per passare le richieste di opzioni, vedere [abilitare le richieste tra le origini in API Web ASP.NET 2: funzionamento di CORS](/aspnet/web-api/overview/security/enabling-cross-origin-requests-in-web-api#how-cors-works).

## <a name="deployment-resources-for-iis-administrators"></a>Risorse di distribuzione per amministratori IIS

* [Documentazione di ISS](/iis)
* [Getting Started with the IIS Manager in IIS](/iis/get-started/getting-started-with-iis/getting-started-with-the-iis-manager-in-iis-7-and-iis-8) (Introduzione a Gestione IIS in IIS)
* [Distribuzione di applicazioni .NET Core](/dotnet/core/deploying/)
* <xref:host-and-deploy/aspnet-core-module>
* <xref:host-and-deploy/directory-structure>
* <xref:host-and-deploy/iis/modules>
* <xref:test/troubleshoot-azure-iis>
* <xref:host-and-deploy/azure-iis-errors-reference>

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:test/troubleshoot>
* <xref:index>
* [Il sito IIS ufficiale di Microsoft](https://www.iis.net/)
* [Libreria di contenuti tecnici di Windows Server](/windows-server/windows-server)
* [HTTP/2 on IIS](/iis/get-started/whats-new-in-iis-10/http2-on-iis) (HTTP/2 in IIS)
* <xref:host-and-deploy/iis/transform-webconfig>

::: moniker-end
