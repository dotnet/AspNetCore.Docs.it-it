---
title: Implementazione del server Web HTTP.sys in ASP.NET Core
author: rick-anderson
description: Informazioni su HTTP.sys, un server Web per ASP.NET Core in Windows. Basato sul driver in modalità kernel HTTP.sys, HTTP.sys è un'alternativa a Kestrel che consente la connessione diretta a Internet senza IIS.
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
uid: fundamentals/servers/httpsys
ms.openlocfilehash: e44cdcb7e427c1ae2531c452a7c8b49e104b3d11
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102586072"
---
# <a name="httpsys-web-server-implementation-in-aspnet-core"></a>Implementazione del server Web HTTP.sys in ASP.NET Core

[Tom Dykstra](https://github.com/tdykstra) e [Chris Ross](https://github.com/Tratcher)

::: moniker range=">= aspnetcore-3.1"

[HTTP.sys](/iis/get-started/introduction-to-iis/introduction-to-iis-architecture#hypertext-transfer-protocol-stack-httpsys) è un [server Web per ASP.NET Core](xref:fundamentals/servers/index) che può essere eseguito solo in Windows. HTTP.sys è un'alternativa al server [Kestrel](xref:fundamentals/servers/kestrel) e offre alcune funzionalità non presenti in Kestrel.

> [!IMPORTANT]
> HTTP.sys non è compatibile con il [modulo ASP.NET Core](xref:host-and-deploy/aspnet-core-module) e non può essere usato con IIS o IIS Express.

HTTP.sys supporta le funzionalità seguenti:

* [Autenticazione di Windows](xref:security/authentication/windowsauth)
* Condivisione delle porte
* HTTPS con SNI
* HTTP/2 su TLS (Windows 10 o versioni successive)
* Trasmissione diretta dei file
* Memorizzazione nella cache delle risposte
* WebSockets (Windows 8 o versioni successive)

Versioni supportate di Windows:

* Windows 7 o versione successiva
* Windows Server 2008 R2 o versione successiva

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/servers/httpsys/samples) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="when-to-use-httpsys"></a>Quando usare HTTP.sys

HTTP.sys è utile per le distribuzioni nei casi seguenti:

* È necessario esporre il server direttamente a Internet senza usare IIS.

  ![HTTP.sys comunica direttamente con Internet](httpsys/_static/httpsys-to-internet.png)

* Una distribuzione interna richiede una funzionalità non disponibile in gheppio. Per altre informazioni, vedere [gheppio rispetto a HTTP.sys](xref:fundamentals/servers/index#kestrel-vs-httpsys)

  ![HTTP.sys comunica direttamente con la rete interna](httpsys/_static/httpsys-to-internal.png)

HTTP.sys è una tecnologia consolidata che protegge da molti tipi di attacchi e offre l'affidabilità, la sicurezza e la scalabilità di un server Web con funzionalità complete. IIS viene eseguito come listener HTTP in HTTP.sys.

## <a name="http2-support"></a>Supporto HTTP/2

[HTTP/2](https://httpwg.org/specs/rfc7540.html) è abilitato per le app ASP.NET Core se vengono soddisfatti i requisiti di base seguenti:

* Windows Server 2016/Windows 10 o versioni successive
* Connessione [ALPN (Application-Layer Protocol Negotiation)](https://tools.ietf.org/html/rfc7301#section-3)
* Connessione TLS 1.2 o successiva

Se viene stabilita una connessione HTTP/2, [HttpRequest.Protocol](xref:Microsoft.AspNetCore.Http.HttpRequest.Protocol*) corrisponde a `HTTP/2`.

HTTP/2 è abilitato per impostazione predefinita. Se non viene stabilita una connessione HTTP/2, la connessione esegue il fallback a HTTP/1.1. In una versione futura di Windows sarà disponibile il flag di configurazione di HTTP/2 e sarà possibile disabilitare il protocollo HTTP/2 con HTTP.sys.

## <a name="kernel-mode-authentication-with-kerberos"></a>Autenticazione in modalità kernel con Kerberos

Per la delega all'autenticazione in modalità kernel, HTTP.sys usa il protocollo di autenticazione Kerberos. L'autenticazione in modalità utente non è supportata con Kerberos e HTTP.sys. È necessario usare l'account del computer per decrittografare il token/ticket Kerberos ottenuto da Active Directory e inoltrato dal client al server per autenticare l'utente. Registrare il nome dell'entità servizio per l'host, non l'utente dell'app.

## <a name="how-to-use-httpsys"></a>Come usare HTTP.sys

### <a name="configure-the-aspnet-core-app-to-use-httpsys"></a>Configurare l'app ASP.NET Core per l'uso di HTTP.sys

Chiamare il metodo di estensione <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderHttpSysExtensions.UseHttpSys*> quando si compila l'host, specificando le eventuali <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions> necessarie. L'esempio seguente imposta le opzioni sui rispettivi valori predefiniti:

[!code-csharp[](httpsys/samples/3.x/SampleApp/Program.cs?name=snippet1&highlight=5-13)]

Le configurazioni aggiuntive di HTTP.sys vengono gestite tramite [impostazioni del Registro di sistema](https://support.microsoft.com/help/820129/http-sys-registry-settings-for-windows).

**Opzioni di HTTP.sys**

| Proprietà | Descrizione | Predefinito |
| -------- | ----------- | :-----: |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.AllowSynchronousIO> | Controllare se l'input e/o l'output sincroni sono consentiti per `HttpContext.Request.Body` e `HttpContext.Response.Body`. | `false` |
| [Authentication.AllowAnonymous](xref:Microsoft.AspNetCore.Server.HttpSys.AuthenticationManager.AllowAnonymous) | Consentire richieste anonime. | `true` |
| [Authentication.Schemes](xref:Microsoft.AspNetCore.Server.HttpSys.AuthenticationManager.Schemes) | Specificare gli schemi di autenticazione consentiti. Può essere modificata in qualsiasi momento prima dell'eliminazione del listener. I valori vengono forniti dall' [enumerazione AuthenticationSchemes](xref:Microsoft.AspNetCore.Server.HttpSys.AuthenticationSchemes): `Basic` , `Kerberos` , `Negotiate` , `None` e `NTLM` . | `None` |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.EnableResponseCaching> | Tentare la memorizzazione nella cache in [modalità kernel](/windows-hardware/drivers/gettingstarted/user-mode-and-kernel-mode) per le risposte con intestazioni idonee. La risposta potrebbe non includere intestazioni `Set-Cookie`, `Vary` o `Pragma`. Deve includere un'intestazione `Cache-Control``public` con valore `shared-max-age` o `max-age` o un'intestazione `Expires`. | `true` |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.Http503Verbosity> | Comportamento HTTP.sys quando si rifiutano le richieste a causa di condizioni di limitazione delle richieste. | [Http503VerbosityLevel. <br> Basic](xref:Microsoft.AspNetCore.Server.HttpSys.Http503VerbosityLevel) |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.MaxAccepts> | Numero massimo di accettazioni simultanee. | 5 &times; [Environment.<br>ProcessorCount](xref:System.Environment.ProcessorCount) |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.MaxConnections> | Numero massimo di connessioni simultanee da accettare. Usare `-1` per un numero infinito. Usare `null` per usare l'impostazione a livello di computer del Registro di sistema. | `null`<br>(a livello di computer<br>impostazione |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.MaxRequestBodySize> | Vedere la sezione <a href="#maxrequestbodysize">MaxRequestBodySize</a>. | 30000000 byte<br>(~28,6 MB) |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.RequestQueueLimit> | Numero massimo di richieste che è possibile accodare. | 1000 |
| `RequestQueueMode` | Indica se il server è responsabile della creazione e della configurazione della coda di richieste o se deve connettersi a una coda esistente.<br>La maggior parte delle opzioni di configurazione esistenti non si applica quando ci si connette a una coda esistente. | `RequestQueueMode.Create` |
| `RequestQueueName` | Nome della coda di richieste HTTP.sys. | `null` (Coda anonima) |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.ThrowWriteExceptions> | Indica se le scritture del corpo della risposta che hanno esito negativo a causa di disconnessioni del client devono generare eccezioni o vengono completate normalmente. | `false`<br>(completamento normale) |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.Timeouts> | Espone la configurazione di <xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager> HTTP.sys, che può essere configurata anche nel Registro di sistema. Seguire i collegamenti API per altre informazioni su ogni impostazione, inclusi i valori predefiniti:<ul><li><xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.DrainEntityBody?displayProperty=nameWithType>: Tempo consentito per l'API del server HTTP per svuotare il corpo dell'entità in una connessione Keep-Alive.</li><li><xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.EntityBody?displayProperty=nameWithType>: Tempo consentito per l'arrivo del corpo dell'entità di richiesta.</li><li><xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.HeaderWait?displayProperty=nameWithType>: Tempo consentito all'API del server HTTP per analizzare l'intestazione della richiesta.</li><li><xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.IdleConnection?displayProperty=nameWithType>: Tempo consentito per una connessione inattiva.</li><li><xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.MinSendBytesPerSecond?displayProperty=nameWithType>: Frequenza di invio minima per la risposta.</li><li><xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.RequestQueue?displayProperty=nameWithType>: Tempo consentito affinché la richiesta rimanga nella coda di richieste prima che l'app lo prelevi.</li></ul> |  |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.UrlPrefixes> | Specificare l'elemento <xref:Microsoft.AspNetCore.Server.HttpSys.UrlPrefixCollection> da registrare con HTTP.sys. Il più utile è <xref:Microsoft.AspNetCore.Server.HttpSys.UrlPrefixCollection.Add%2A?displayProperty=nameWithType> , che viene usato per aggiungere un prefisso alla raccolta. Queste impostazioni possono essere modificate in qualsiasi momento prima dell'eliminazione del listener. |  |

<a name="maxrequestbodysize"></a>

**MaxRequestBodySize**

Dimensioni massime consentite per qualsiasi corpo della richiesta in byte. Con l'impostazione `null`, le dimensioni massime del corpo della richiesta sono illimitate. Questo limite non ha effetto sulle connessioni aggiornate, che sono sempre illimitate.

Il metodo consigliato per ignorare il limite in un'applicazione ASP.NET Core MVC per un singolo `IActionResult` prevede l'uso dell'attributo <xref:Microsoft.AspNetCore.Mvc.RequestSizeLimitAttribute> in un metodo di azione:

```csharp
[RequestSizeLimit(100000000)]
public IActionResult MyActionMethod()
```

Se l'app tenta di configurare il limite per una richiesta dopo che l'app ha avviato la lettura della richiesta stessa, viene generata un'eccezione. È possibile usare una proprietà `IsReadOnly` per indicare se la proprietà `MaxRequestBodySize` è in stato di sola lettura e pertanto è troppo tardi per configurare il limite.

Se l'app deve eseguire l'override di <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.MaxRequestBodySize> per ogni richiesta, usare <xref:Microsoft.AspNetCore.Http.Features.IHttpMaxRequestBodySizeFeature>:

[!code-csharp[](httpsys/samples/3.x/SampleApp/Startup.cs?name=snippet1&highlight=6-7)]

Se si usa Visual Studio, assicurarsi che l'app non sia configurata per l'esecuzione di IIS o IIS Express.

In Visual Studio il profilo di avvio predefinito è per IIS Express. Per eseguire il progetto come app console, modificare manualmente il profilo selezionato, come illustrato nello screenshot seguente:

![Selezionare il profilo dell'applicazione console](httpsys/_static/vs-choose-profile.png)

### <a name="configure-windows-server"></a>Configurare Windows Server

1. Determinare le porte da aprire per l'app e usare [Windows Firewall](/windows/security/threat-protection/windows-firewall/create-an-inbound-port-rule) o il cmdlet di PowerShell [New-NetFirewallRule](/powershell/module/netsecurity/new-netfirewallrule) per aprire le porte del firewall per consentire al traffico di raggiungere HTTP.sys. Nei comandi seguenti e nella configurazione dell'app, viene usata la porta 443.

1. Quando si distribuisce una macchina virtuale di Azure, aprire le porte nel [gruppo di sicurezza di rete](/azure/virtual-machines/windows/nsg-quickstart-portal). Nei comandi seguenti e nella configurazione dell'app, viene usata la porta 443.

1. Ottenere e installare i certificati X.509, se necessario.

   In Windows, creare certificati autofirmati con il [cmdlet di PowerShell New-SelfSignedCertificate](/powershell/module/pkiclient/new-selfsignedcertificate). Per un esempio non supportato, vedere [UpdateIISExpressSSLForChrome.ps1](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/includes/make-x509-cert/UpdateIISExpressSSLForChrome.ps1).

   Installare i certificati autofirmati o con firma nell'archivio **Computer locale** > **Personale** del server.

1. Se l'app è una [distribuzione dipendente dal framework](/dotnet/core/deploying/#framework-dependent-deployments-fdd), installare .NET Core, .NET Framework o entrambi (se l'app è un'app .NET Core destinata a .NET Framework).

   * **.NET Core**: se l'app richiede .NET Core, ottenere ed eseguire il programma di installazione del **runtime di .NET Core** da [download di .NET Core](https://dotnet.microsoft.com/download). Non installare l'SDK completo nel server.
   * **.NET Framework**: se l'app richiede .NET Framework, vedere la [Guida all'installazione di .NET Framework](/dotnet/framework/install/). Installare la versione di .NET Framework richiesta. Il programma di installazione per la versione più recente di .NET Framework è disponibile dalla pagina di [download per .NET Core](https://dotnet.microsoft.com/download).

   Se l'app è una [distribuzione autonoma](/dotnet/core/deploying/#self-contained-deployments-scd) include il runtime nella distribuzione. Non è richiesta l'installazione di un framework nel server.

1. Configurare gli URL e le porte nell'app.

   Per impostazione predefinita, ASP.NET Core è associato a `http://localhost:5000`. Per configurare le porte e i prefissi URL, è possibile usare:

   * <xref:Microsoft.AspNetCore.Hosting.HostingAbstractionsWebHostBuilderExtensions.UseUrls*>
   * L'argomento della riga di comando `urls`
   * La variabile di ambiente `ASPNETCORE_URLS`
   * <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.UrlPrefixes>

   L'esempio di codice seguente mostra come usare <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.UrlPrefixes> con l'indirizzo IP locale del server `10.0.0.4` sulla porta 443:

   [!code-csharp[](httpsys/samples_snapshot/3.x/Program.cs?highlight=7)]

   Un vantaggio offerto da `UrlPrefixes` è che viene generato immediatamente un messaggio di errore per i prefissi non formattati correttamente.

   Le impostazioni di `UrlPrefixes` sostituiscono le impostazioni `UseUrls`/`urls`/`ASPNETCORE_URLS`. Pertanto, un vantaggio offerto da `UseUrls`, `urls` e dalla variabile di ambiente `ASPNETCORE_URLS` è che risulta più semplice alternare Kestrel e HTTP.sys.

   HTTP.sys usa i [formati di stringa UrlPrefix dell'API del server HTTP](/windows/win32/http/urlprefix-strings).

   > [!WARNING]
   > Le associazioni con caratteri jolly di livello superiore (`http://*:80/` e `http://+:80`) **non** devono essere usate, poiché possono creare vulnerabilità a livello di sicurezza nell'app. Questo concetto vale sia per i caratteri jolly sicuri che vulnerabili. Usare nomi host o indirizzi IP espliciti al posto di caratteri jolly. L'associazione con caratteri jolly del sottodominio (ad esempio, `*.mysub.com`) non costituisce un rischio per la sicurezza se viene controllato l'intero dominio padre (a differenza di `*.com`, che è vulnerabile). Per ulteriori informazioni, vedere [RFC 7230: Section 5,4: host](https://tools.ietf.org/html/rfc7230#section-5.4).

1. Pre-registrare i prefissi URL nel server.

   Lo strumento predefinito per la configurazione di HTTP.sys è *netsh.exe*. *Netsh.exe* viene usato per riservare i prefissi URL e assegnare i certificati X.509. Per questo strumento sono necessari privilegi di amministratore.

   Usare lo strumento *netsh.exe* per registrare gli URL per l'app:

   ```console
   netsh http add urlacl url=<URL> user=<USER>
   ```

   * `<URL>`: Uniform Resource Locator completo (URL). Non usare un'associazione con caratteri jolly. Usare un nome host valido o un indirizzo IP locale. *L'URL deve includere una barra finale.*
   * `<USER>`: Specifica il nome dell'utente o del gruppo di utenti.

   Nell'esempio seguente, l'indirizzo IP locale del server è `10.0.0.4`:

   ```console
   netsh http add urlacl url=https://10.0.0.4:443/ user=Users
   ```

   Quando viene registrato un URL, lo strumento risponde con `URL reservation successfully added`.

   Per eliminare un URL registrato, usare il comando `delete urlacl`:

   ```console
   netsh http delete urlacl url=<URL>
   ```

1. Registrare i certificati X.509 nel server.

   Usare lo strumento *netsh.exe* per registrare i certificati per l'app:

   ```console
   netsh http add sslcert ipport=<IP>:<PORT> certhash=<THUMBPRINT> appid="{<GUID>}"
   ```

   * `<IP>`: Specifica l'indirizzo IP locale per l'associazione. Non usare un'associazione con caratteri jolly. Usare un indirizzo IP valido.
   * `<PORT>`: Specifica la porta per l'associazione.
   * `<THUMBPRINT>`: Identificazione personale del certificato X. 509.
   * `<GUID>`: GUID generato dallo sviluppatore per rappresentare l'app a scopo informativo.

   A scopo di riferimento, archiviare il GUID nell'app come tag di pacchetto:

   * In Visual Studio:
     * Aprire le proprietà del progetto dell'app facendo clic sull'app con il pulsante destro del mouse in **Esplora soluzioni** e scegliendo **Proprietà**.
     * Selezionare la scheda **Pacchetto**.
     * Immettere il GUID creato nel campo **Tag**.
   * Se non si usa Visual Studio:
     * Aprire il file di progetto dell'app.
     * Aggiungere una proprietà `<PackageTags>` a un `<PropertyGroup>` nuovo o esistente con il GUID creato:

       ```xml
       <PropertyGroup>
         <PackageTags>9412ee86-c21b-4eb8-bd89-f650fbf44931</PackageTags>
       </PropertyGroup>
       ```

   Nell'esempio seguente:

   * L'indirizzo IP locale del server è `10.0.0.4`.
   * Un generatore di GUID casuali online fornisce il valore `appid`.

   ```console
   netsh http add sslcert 
       ipport=10.0.0.4:443 
       certhash=b66ee04419d4ee37464ab8785ff02449980eae10 
       appid="{9412ee86-c21b-4eb8-bd89-f650fbf44931}"
   ```

   Quando viene registrato un certificato, lo strumento risponde con `SSL Certificate successfully added`.

   Per eliminare la registrazione di un certificato, usare il comando `delete sslcert`:

   ```console
   netsh http delete sslcert ipport=<IP>:<PORT>
   ```

   Documentazione di riferimento per *netsh.exe*:

   * [Comandi netsh per HTTP (Hypertext Transfer Protocol)](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc725882(v=ws.10))
   * [UrlPrefix Strings](/windows/win32/http/urlprefix-strings) (Stringhe UrlPrefix)

1. Eseguire l'app.

   Non sono necessari i privilegi di amministratore per eseguire l'app con binding a localhost tramite HTTP (non HTTPS) con un numero di porta maggiore di 1024. Per altre configurazioni (ad esempio, l'uso di un indirizzo IP locale o il binding sulla porta 443), eseguire l'app con i privilegi di amministratore.

   L'app risponde all'indirizzo IP pubblico del server. In questo esempio, il server è raggiungibile da Internet al relativo indirizzo IP pubblico `104.214.79.47`.

   In questo esempio viene usato un certificato di sviluppo. La pagina viene caricata in modo sicuro dopo aver ignorato l'avviso di certificato non attendibile del browser.

   ![Finestra del browser che mostra la pagina di indice dell'app caricata](httpsys/_static/browser.png)

## <a name="proxy-server-and-load-balancer-scenarios"></a>Scenari con server proxy e servizi di bilanciamento del carico

Per le app ospitate da HTTP.sys che interagiscono con richieste da Internet o da una rete aziendale, potrebbero essere necessari interventi di configurazione aggiuntivi in caso di hosting dietro server proxy e servizi di bilanciamento del carico. Per altre informazioni, vedere [Configurare ASP.NET Core per l'utilizzo di server proxy e servizi di bilanciamento del carico](xref:host-and-deploy/proxy-load-balancer).

## <a name="advanced-http2-features-to-support-grpc"></a>Funzionalità HTTP/2 avanzate per supportare gRPC

Funzionalità HTTP/2 aggiuntive in HTTP.sys supportano gRPC, incluso il supporto per i trailer di risposta e l'invio di frame di reimpostazione.

Requisiti per l'esecuzione di gRPC con HTTP.sys:

* Windows 10, build del sistema operativo 19041,508 o versione successiva
* Connessione TLS 1.2 o successiva

### <a name="trailers"></a>Trailer

[!INCLUDE[](~/includes/trailers.md)]

### <a name="reset"></a>Reset

[!INCLUDE[](~/includes/reset.md)]

## <a name="additional-resources"></a>Risorse aggiuntive

* [Abilitare l'autenticazione di Windows con HTTP. sys](xref:security/authentication/windowsauth#httpsys)
* [API di HTTP Server](/windows/win32/http/http-api-start-page)
* [Repository di GitHub aspnet/HttpSysServer (codice sorgente)](https://github.com/aspnet/HttpSysServer/)
* [Host](xref:fundamentals/index#host)
* <xref:test/troubleshoot>

::: moniker-end

::: moniker range="= aspnetcore-3.0"

[HTTP.sys](/iis/get-started/introduction-to-iis/introduction-to-iis-architecture#hypertext-transfer-protocol-stack-httpsys) è un [server Web per ASP.NET Core](xref:fundamentals/servers/index) che può essere eseguito solo in Windows. HTTP.sys è un'alternativa al server [Kestrel](xref:fundamentals/servers/kestrel) e offre alcune funzionalità non presenti in Kestrel.

> [!IMPORTANT]
> HTTP.sys non è compatibile con il [modulo ASP.NET Core](xref:host-and-deploy/aspnet-core-module) e non può essere usato con IIS o IIS Express.

HTTP.sys supporta le funzionalità seguenti:

* [Autenticazione di Windows](xref:security/authentication/windowsauth)
* Condivisione delle porte
* HTTPS con SNI
* HTTP/2 su TLS (Windows 10 o versioni successive)
* Trasmissione diretta dei file
* Memorizzazione nella cache delle risposte
* WebSockets (Windows 8 o versioni successive)

Versioni supportate di Windows:

* Windows 7 o versione successiva
* Windows Server 2008 R2 o versione successiva

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/servers/httpsys/samples) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="when-to-use-httpsys"></a>Quando usare HTTP.sys

HTTP.sys è utile per le distribuzioni nei casi seguenti:

* È necessario esporre il server direttamente a Internet senza usare IIS.

  ![HTTP.sys comunica direttamente con Internet](httpsys/_static/httpsys-to-internet.png)

* Una distribuzione interna richiede una funzionalità non disponibile in Kestrel, ad esempio l'[autenticazione di Windows](xref:security/authentication/windowsauth).

  ![HTTP.sys comunica direttamente con la rete interna](httpsys/_static/httpsys-to-internal.png)

HTTP.sys è una tecnologia consolidata che protegge da molti tipi di attacchi e offre l'affidabilità, la sicurezza e la scalabilità di un server Web con funzionalità complete. IIS viene eseguito come listener HTTP in HTTP.sys.

## <a name="http2-support"></a>Supporto HTTP/2

[HTTP/2](https://httpwg.org/specs/rfc7540.html) è abilitato per le app ASP.NET Core se vengono soddisfatti i requisiti di base seguenti:

* Windows Server 2016/Windows 10 o versioni successive
* Connessione [ALPN (Application-Layer Protocol Negotiation)](https://tools.ietf.org/html/rfc7301#section-3)
* Connessione TLS 1.2 o successiva

Se viene stabilita una connessione HTTP/2, [HttpRequest.Protocol](xref:Microsoft.AspNetCore.Http.HttpRequest.Protocol*) corrisponde a `HTTP/2`.

HTTP/2 è abilitato per impostazione predefinita. Se non viene stabilita una connessione HTTP/2, la connessione esegue il fallback a HTTP/1.1. In una versione futura di Windows sarà disponibile il flag di configurazione di HTTP/2 e sarà possibile disabilitare il protocollo HTTP/2 con HTTP.sys.

## <a name="kernel-mode-authentication-with-kerberos"></a>Autenticazione in modalità kernel con Kerberos

Per la delega all'autenticazione in modalità kernel, HTTP.sys usa il protocollo di autenticazione Kerberos. L'autenticazione in modalità utente non è supportata con Kerberos e HTTP.sys. È necessario usare l'account del computer per decrittografare il token/ticket Kerberos ottenuto da Active Directory e inoltrato dal client al server per autenticare l'utente. Registrare il nome dell'entità servizio per l'host, non l'utente dell'app.

## <a name="how-to-use-httpsys"></a>Come usare HTTP.sys

### <a name="configure-the-aspnet-core-app-to-use-httpsys"></a>Configurare l'app ASP.NET Core per l'uso di HTTP.sys

Chiamare il metodo di estensione <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderHttpSysExtensions.UseHttpSys*> quando si compila l'host, specificando le eventuali <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions> necessarie. L'esempio seguente imposta le opzioni sui rispettivi valori predefiniti:

[!code-csharp[](httpsys/samples/3.x/SampleApp/Program.cs?name=snippet1&highlight=5-13)]

Le configurazioni aggiuntive di HTTP.sys vengono gestite tramite [impostazioni del Registro di sistema](https://support.microsoft.com/help/820129/http-sys-registry-settings-for-windows).

**Opzioni di HTTP.sys**

| Proprietà | Descrizione | Predefinito |
| -------- | ----------- | :-----: |
| [AllowSynchronousIO](xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.AllowSynchronousIO) | Controllare se l'input e/o l'output sincroni sono consentiti per `HttpContext.Request.Body` e `HttpContext.Response.Body`. | `false` |
| [Authentication.AllowAnonymous](xref:Microsoft.AspNetCore.Server.HttpSys.AuthenticationManager.AllowAnonymous) | Consentire richieste anonime. | `true` |
| [Authentication.Schemes](xref:Microsoft.AspNetCore.Server.HttpSys.AuthenticationManager.Schemes) | Specificare gli schemi di autenticazione consentiti. Può essere modificata in qualsiasi momento prima dell'eliminazione del listener. I valori vengono forniti dall' [enumerazione AuthenticationSchemes](xref:Microsoft.AspNetCore.Server.HttpSys.AuthenticationSchemes): `Basic` , `Kerberos` , `Negotiate` , `None` e `NTLM` . | `None` |
| [EnableResponseCaching](xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.EnableResponseCaching) | Tentare la memorizzazione nella cache in [modalità kernel](/windows-hardware/drivers/gettingstarted/user-mode-and-kernel-mode) per le risposte con intestazioni idonee. La risposta potrebbe non includere intestazioni `Set-Cookie`, `Vary` o `Pragma`. Deve includere un'intestazione `Cache-Control``public` con valore `shared-max-age` o `max-age` o un'intestazione `Expires`. | `true` |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.MaxAccepts> | Numero massimo di accettazioni simultanee. | 5 &times; [Environment.<br>ProcessorCount](xref:System.Environment.ProcessorCount) |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.MaxConnections> | Numero massimo di connessioni simultanee da accettare. Usare `-1` per un numero infinito. Usare `null` per usare l'impostazione a livello di computer del Registro di sistema. | `null`<br>(a livello di computer<br>impostazione |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.MaxRequestBodySize> | Vedere la sezione <a href="#maxrequestbodysize">MaxRequestBodySize</a>. | 30000000 byte<br>(~28,6 MB) |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.RequestQueueLimit> | Numero massimo di richieste che è possibile accodare. | 1000 |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.ThrowWriteExceptions> | Indica se le scritture del corpo della risposta che hanno esito negativo a causa di disconnessioni del client devono generare eccezioni o vengono completate normalmente. | `false`<br>(completamento normale) |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.Timeouts> | Espone la configurazione di <xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager> HTTP.sys, che può essere configurata anche nel Registro di sistema. Seguire i collegamenti API per altre informazioni su ogni impostazione, inclusi i valori predefiniti:<ul><li>[TimeoutManager. DrainEntityBody](xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.DrainEntityBody): tempo consentito per l'API del server http per svuotare il corpo dell'entità in una connessione Keep-Alive.</li><li>[TimeoutManager. EntityBody](xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.EntityBody): tempo consentito per l'arrivo del corpo dell'entità di richiesta.</li><li>[TimeoutManager. HeaderWait](xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.HeaderWait): tempo consentito all'API del server http per analizzare l'intestazione della richiesta.</li><li>[TimeoutManager. IdleConnection](xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.IdleConnection): tempo consentito per una connessione inattiva.</li><li>[TimeoutManager. MinSendBytesPerSecond](xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.MinSendBytesPerSecond): frequenza di invio minima per la risposta.</li><li>[TimeoutManager. RequestQueue](xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.RequestQueue): tempo consentito affinché la richiesta rimanga nella coda di richieste prima che l'app lo prelevi.</li></ul> |  |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.UrlPrefixes> | Specificare l'elemento <xref:Microsoft.AspNetCore.Server.HttpSys.UrlPrefixCollection> da registrare con HTTP.sys. Il più utile è il metodo [UrlPrefixCollection.Add](xref:Microsoft.AspNetCore.Server.HttpSys.UrlPrefixCollection.Add*) usato per aggiungere un prefisso alla raccolta. Queste impostazioni possono essere modificate in qualsiasi momento prima dell'eliminazione del listener. |  |

<a name="maxrequestbodysize"></a>

**MaxRequestBodySize**

Dimensioni massime consentite per qualsiasi corpo della richiesta in byte. Con l'impostazione `null`, le dimensioni massime del corpo della richiesta sono illimitate. Questo limite non ha effetto sulle connessioni aggiornate, che sono sempre illimitate.

Il metodo consigliato per ignorare il limite in un'applicazione ASP.NET Core MVC per un singolo `IActionResult` prevede l'uso dell'attributo <xref:Microsoft.AspNetCore.Mvc.RequestSizeLimitAttribute> in un metodo di azione:

```csharp
[RequestSizeLimit(100000000)]
public IActionResult MyActionMethod()
```

Se l'app tenta di configurare il limite per una richiesta dopo che l'app ha avviato la lettura della richiesta stessa, viene generata un'eccezione. È possibile usare una proprietà `IsReadOnly` per indicare se la proprietà `MaxRequestBodySize` è in stato di sola lettura e pertanto è troppo tardi per configurare il limite.

Se l'app deve eseguire l'override di <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.MaxRequestBodySize> per ogni richiesta, usare <xref:Microsoft.AspNetCore.Http.Features.IHttpMaxRequestBodySizeFeature>:

[!code-csharp[](httpsys/samples/3.x/SampleApp/Startup.cs?name=snippet1&highlight=6-7)]

Se si usa Visual Studio, assicurarsi che l'app non sia configurata per l'esecuzione di IIS o IIS Express.

In Visual Studio il profilo di avvio predefinito è per IIS Express. Per eseguire il progetto come app console, modificare manualmente il profilo selezionato, come illustrato nello screenshot seguente:

![Selezionare il profilo dell'applicazione console](httpsys/_static/vs-choose-profile.png)

### <a name="configure-windows-server"></a>Configurare Windows Server

1. Determinare le porte da aprire per l'app e usare [Windows Firewall](/windows/security/threat-protection/windows-firewall/create-an-inbound-port-rule) o il cmdlet di PowerShell [New-NetFirewallRule](/powershell/module/netsecurity/new-netfirewallrule) per aprire le porte del firewall per consentire al traffico di raggiungere HTTP.sys. Nei comandi seguenti e nella configurazione dell'app, viene usata la porta 443.

1. Quando si distribuisce una macchina virtuale di Azure, aprire le porte nel [gruppo di sicurezza di rete](/azure/virtual-machines/windows/nsg-quickstart-portal). Nei comandi seguenti e nella configurazione dell'app, viene usata la porta 443.

1. Ottenere e installare i certificati X.509, se necessario.

   In Windows, creare certificati autofirmati con il [cmdlet di PowerShell New-SelfSignedCertificate](/powershell/module/pkiclient/new-selfsignedcertificate). Per un esempio non supportato, vedere [UpdateIISExpressSSLForChrome.ps1](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/includes/make-x509-cert/UpdateIISExpressSSLForChrome.ps1).

   Installare i certificati autofirmati o con firma nell'archivio **Computer locale** > **Personale** del server.

1. Se l'app è una [distribuzione dipendente dal framework](/dotnet/core/deploying/#framework-dependent-deployments-fdd), installare .NET Core, .NET Framework o entrambi (se l'app è un'app .NET Core destinata a .NET Framework).

   * **.NET Core**: se l'app richiede .NET Core, ottenere ed eseguire il programma di installazione del **runtime di .NET Core** da [download di .NET Core](https://dotnet.microsoft.com/download). Non installare l'SDK completo nel server.
   * **.NET Framework**: se l'app richiede .NET Framework, vedere la [Guida all'installazione di .NET Framework](/dotnet/framework/install/). Installare la versione di .NET Framework richiesta. Il programma di installazione per la versione più recente di .NET Framework è disponibile dalla pagina di [download per .NET Core](https://dotnet.microsoft.com/download).

   Se l'app è una [distribuzione autonoma](/dotnet/core/deploying/#self-contained-deployments-scd) include il runtime nella distribuzione. Non è richiesta l'installazione di un framework nel server.

1. Configurare gli URL e le porte nell'app.

   Per impostazione predefinita, ASP.NET Core è associato a `http://localhost:5000`. Per configurare le porte e i prefissi URL, è possibile usare:

   * <xref:Microsoft.AspNetCore.Hosting.HostingAbstractionsWebHostBuilderExtensions.UseUrls*>
   * L'argomento della riga di comando `urls`
   * La variabile di ambiente `ASPNETCORE_URLS`
   * <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.UrlPrefixes>

   L'esempio di codice seguente mostra come usare <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.UrlPrefixes> con l'indirizzo IP locale del server `10.0.0.4` sulla porta 443:

   [!code-csharp[](httpsys/samples_snapshot/3.x/Program.cs?highlight=7)]

   Un vantaggio offerto da `UrlPrefixes` è che viene generato immediatamente un messaggio di errore per i prefissi non formattati correttamente.

   Le impostazioni di `UrlPrefixes` sostituiscono le impostazioni `UseUrls`/`urls`/`ASPNETCORE_URLS`. Pertanto, un vantaggio offerto da `UseUrls`, `urls` e dalla variabile di ambiente `ASPNETCORE_URLS` è che risulta più semplice alternare Kestrel e HTTP.sys.

   HTTP.sys usa i [formati di stringa UrlPrefix dell'API del server HTTP](/windows/win32/http/urlprefix-strings).

   > [!WARNING]
   > Le associazioni con caratteri jolly di livello superiore (`http://*:80/` e `http://+:80`) **non** devono essere usate, poiché possono creare vulnerabilità a livello di sicurezza nell'app. Questo concetto vale sia per i caratteri jolly sicuri che vulnerabili. Usare nomi host o indirizzi IP espliciti al posto di caratteri jolly. L'associazione con caratteri jolly del sottodominio (ad esempio, `*.mysub.com`) non costituisce un rischio per la sicurezza se viene controllato l'intero dominio padre (a differenza di `*.com`, che è vulnerabile). Per ulteriori informazioni, vedere [RFC 7230: Section 5,4: host](https://tools.ietf.org/html/rfc7230#section-5.4).

1. Pre-registrare i prefissi URL nel server.

   Lo strumento predefinito per la configurazione di HTTP.sys è *netsh.exe*. *Netsh.exe* viene usato per riservare i prefissi URL e assegnare i certificati X.509. Per questo strumento sono necessari privilegi di amministratore.

   Usare lo strumento *netsh.exe* per registrare gli URL per l'app:

   ```console
   netsh http add urlacl url=<URL> user=<USER>
   ```

   * `<URL>`: Uniform Resource Locator completo (URL). Non usare un'associazione con caratteri jolly. Usare un nome host valido o un indirizzo IP locale. *L'URL deve includere una barra finale.*
   * `<USER>`: Specifica il nome dell'utente o del gruppo di utenti.

   Nell'esempio seguente, l'indirizzo IP locale del server è `10.0.0.4`:

   ```console
   netsh http add urlacl url=https://10.0.0.4:443/ user=Users
   ```

   Quando viene registrato un URL, lo strumento risponde con `URL reservation successfully added`.

   Per eliminare un URL registrato, usare il comando `delete urlacl`:

   ```console
   netsh http delete urlacl url=<URL>
   ```

1. Registrare i certificati X.509 nel server.

   Usare lo strumento *netsh.exe* per registrare i certificati per l'app:

   ```console
   netsh http add sslcert ipport=<IP>:<PORT> certhash=<THUMBPRINT> appid="{<GUID>}"
   ```

   * `<IP>`: Specifica l'indirizzo IP locale per l'associazione. Non usare un'associazione con caratteri jolly. Usare un indirizzo IP valido.
   * `<PORT>`: Specifica la porta per l'associazione.
   * `<THUMBPRINT>`: Identificazione personale del certificato X. 509.
   * `<GUID>`: GUID generato dallo sviluppatore per rappresentare l'app a scopo informativo.

   A scopo di riferimento, archiviare il GUID nell'app come tag di pacchetto:

   * In Visual Studio:
     * Aprire le proprietà del progetto dell'app facendo clic sull'app con il pulsante destro del mouse in **Esplora soluzioni** e scegliendo **Proprietà**.
     * Selezionare la scheda **Pacchetto**.
     * Immettere il GUID creato nel campo **Tag**.
   * Se non si usa Visual Studio:
     * Aprire il file di progetto dell'app.
     * Aggiungere una proprietà `<PackageTags>` a un `<PropertyGroup>` nuovo o esistente con il GUID creato:

       ```xml
       <PropertyGroup>
         <PackageTags>9412ee86-c21b-4eb8-bd89-f650fbf44931</PackageTags>
       </PropertyGroup>
       ```

   Nell'esempio seguente:

   * L'indirizzo IP locale del server è `10.0.0.4`.
   * Un generatore di GUID casuali online fornisce il valore `appid`.

   ```console
   netsh http add sslcert 
       ipport=10.0.0.4:443 
       certhash=b66ee04419d4ee37464ab8785ff02449980eae10 
       appid="{9412ee86-c21b-4eb8-bd89-f650fbf44931}"
   ```

   Quando viene registrato un certificato, lo strumento risponde con `SSL Certificate successfully added`.

   Per eliminare la registrazione di un certificato, usare il comando `delete sslcert`:

   ```console
   netsh http delete sslcert ipport=<IP>:<PORT>
   ```

   Documentazione di riferimento per *netsh.exe*:

   * [Comandi netsh per HTTP (Hypertext Transfer Protocol)](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc725882(v=ws.10))
   * [UrlPrefix Strings](/windows/win32/http/urlprefix-strings) (Stringhe UrlPrefix)

1. Eseguire l'app.

   Non sono necessari i privilegi di amministratore per eseguire l'app con binding a localhost tramite HTTP (non HTTPS) con un numero di porta maggiore di 1024. Per altre configurazioni (ad esempio, l'uso di un indirizzo IP locale o il binding sulla porta 443), eseguire l'app con i privilegi di amministratore.

   L'app risponde all'indirizzo IP pubblico del server. In questo esempio, il server è raggiungibile da Internet al relativo indirizzo IP pubblico `104.214.79.47`.

   In questo esempio viene usato un certificato di sviluppo. La pagina viene caricata in modo sicuro dopo aver ignorato l'avviso di certificato non attendibile del browser.

   ![Finestra del browser che mostra la pagina di indice dell'app caricata](httpsys/_static/browser.png)

## <a name="proxy-server-and-load-balancer-scenarios"></a>Scenari con server proxy e servizi di bilanciamento del carico

Per le app ospitate da HTTP.sys che interagiscono con richieste da Internet o da una rete aziendale, potrebbero essere necessari interventi di configurazione aggiuntivi in caso di hosting dietro server proxy e servizi di bilanciamento del carico. Per altre informazioni, vedere [Configurare ASP.NET Core per l'utilizzo di server proxy e servizi di bilanciamento del carico](xref:host-and-deploy/proxy-load-balancer).

## <a name="additional-resources"></a>Risorse aggiuntive

* [Abilitare l'autenticazione di Windows con HTTP. sys](xref:security/authentication/windowsauth#httpsys)
* [API di HTTP Server](/windows/win32/http/http-api-start-page)
* [Repository di GitHub aspnet/HttpSysServer (codice sorgente)](https://github.com/aspnet/HttpSysServer/)
* [Host](xref:fundamentals/index#host)
* <xref:test/troubleshoot>

::: moniker-end

::: moniker range="= aspnetcore-2.2"

[HTTP.sys](/iis/get-started/introduction-to-iis/introduction-to-iis-architecture#hypertext-transfer-protocol-stack-httpsys) è un [server Web per ASP.NET Core](xref:fundamentals/servers/index) che può essere eseguito solo in Windows. HTTP.sys è un'alternativa al server [Kestrel](xref:fundamentals/servers/kestrel) e offre alcune funzionalità non presenti in Kestrel.

> [!IMPORTANT]
> HTTP.sys non è compatibile con il [modulo ASP.NET Core](xref:host-and-deploy/aspnet-core-module) e non può essere usato con IIS o IIS Express.

HTTP.sys supporta le funzionalità seguenti:

* [Autenticazione di Windows](xref:security/authentication/windowsauth)
* Condivisione delle porte
* HTTPS con SNI
* HTTP/2 su TLS (Windows 10 o versioni successive)
* Trasmissione diretta dei file
* Memorizzazione nella cache delle risposte
* WebSockets (Windows 8 o versioni successive)

Versioni supportate di Windows:

* Windows 7 o versione successiva
* Windows Server 2008 R2 o versione successiva

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/servers/httpsys/samples) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="when-to-use-httpsys"></a>Quando usare HTTP.sys

HTTP.sys è utile per le distribuzioni nei casi seguenti:

* È necessario esporre il server direttamente a Internet senza usare IIS.

  ![HTTP.sys comunica direttamente con Internet](httpsys/_static/httpsys-to-internet.png)

* Una distribuzione interna richiede una funzionalità non disponibile in Kestrel, ad esempio l'[autenticazione di Windows](xref:security/authentication/windowsauth).

  ![HTTP.sys comunica direttamente con la rete interna](httpsys/_static/httpsys-to-internal.png)

HTTP.sys è una tecnologia consolidata che protegge da molti tipi di attacchi e offre l'affidabilità, la sicurezza e la scalabilità di un server Web con funzionalità complete. IIS viene eseguito come listener HTTP in HTTP.sys.

## <a name="http2-support"></a>Supporto HTTP/2

[HTTP/2](https://httpwg.org/specs/rfc7540.html) è abilitato per le app ASP.NET Core se vengono soddisfatti i requisiti di base seguenti:

* Windows Server 2016/Windows 10 o versioni successive
* Connessione [ALPN (Application-Layer Protocol Negotiation)](https://tools.ietf.org/html/rfc7301#section-3)
* Connessione TLS 1.2 o successiva

Se viene stabilita una connessione HTTP/2, [HttpRequest.Protocol](xref:Microsoft.AspNetCore.Http.HttpRequest.Protocol*) corrisponde a `HTTP/2`.

HTTP/2 è abilitato per impostazione predefinita. Se non viene stabilita una connessione HTTP/2, la connessione esegue il fallback a HTTP/1.1. In una versione futura di Windows sarà disponibile il flag di configurazione di HTTP/2 e sarà possibile disabilitare il protocollo HTTP/2 con HTTP.sys.

## <a name="kernel-mode-authentication-with-kerberos"></a>Autenticazione in modalità kernel con Kerberos

Per la delega all'autenticazione in modalità kernel, HTTP.sys usa il protocollo di autenticazione Kerberos. L'autenticazione in modalità utente non è supportata con Kerberos e HTTP.sys. È necessario usare l'account del computer per decrittografare il token/ticket Kerberos ottenuto da Active Directory e inoltrato dal client al server per autenticare l'utente. Registrare il nome dell'entità servizio per l'host, non l'utente dell'app.

## <a name="how-to-use-httpsys"></a>Come usare HTTP.sys

### <a name="configure-the-aspnet-core-app-to-use-httpsys"></a>Configurare l'app ASP.NET Core per l'uso di HTTP.sys

Non è necessario un riferimento al pacchetto nel file di progetto quando si usa il [metapacchetto Microsoft. AspNetCore. app](xref:fundamentals/metapackage-app) ([NuGet.org](https://www.nuget.org/packages/Microsoft.AspNetCore.App/)). Se non si usa il metapacchetto `Microsoft.AspNetCore.App` aggiungere un riferimento al pacchetto a [Microsoft.AspNetCore.Server.HttpSys](https://www.nuget.org/packages/Microsoft.AspNetCore.Server.HttpSys/).

Chiamare il metodo di estensione <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderHttpSysExtensions.UseHttpSys*> quando si compila l'host, specificando le eventuali <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions> necessarie. L'esempio seguente imposta le opzioni sui rispettivi valori predefiniti:

[!code-csharp[](httpsys/samples/2.x/SampleApp/Program.cs?name=snippet1&highlight=4-12)]

Le configurazioni aggiuntive di HTTP.sys vengono gestite tramite [impostazioni del Registro di sistema](https://support.microsoft.com/help/820129/http-sys-registry-settings-for-windows).

**Opzioni di HTTP.sys**

| Proprietà | Descrizione | Predefinito |
| -------- | ----------- | :-----: |
| [AllowSynchronousIO](xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.AllowSynchronousIO) | Controllare se l'input e/o l'output sincroni sono consentiti per `HttpContext.Request.Body` e `HttpContext.Response.Body`. | `true` |
| [Authentication.AllowAnonymous](xref:Microsoft.AspNetCore.Server.HttpSys.AuthenticationManager.AllowAnonymous) | Consentire richieste anonime. | `true` |
| [Authentication.Schemes](xref:Microsoft.AspNetCore.Server.HttpSys.AuthenticationManager.Schemes) | Specificare gli schemi di autenticazione consentiti. Può essere modificata in qualsiasi momento prima dell'eliminazione del listener. I valori vengono forniti dall' [enumerazione AuthenticationSchemes](xref:Microsoft.AspNetCore.Server.HttpSys.AuthenticationSchemes): `Basic` , `Kerberos` , `Negotiate` , `None` e `NTLM` . | `None` |
| [EnableResponseCaching](xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.EnableResponseCaching) | Tentare la memorizzazione nella cache in [modalità kernel](/windows-hardware/drivers/gettingstarted/user-mode-and-kernel-mode) per le risposte con intestazioni idonee. La risposta potrebbe non includere intestazioni `Set-Cookie`, `Vary` o `Pragma`. Deve includere un'intestazione `Cache-Control``public` con valore `shared-max-age` o `max-age` o un'intestazione `Expires`. | `true` |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.MaxAccepts> | Numero massimo di accettazioni simultanee. | 5 &times; [Environment.<br>ProcessorCount](xref:System.Environment.ProcessorCount) |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.MaxConnections> | Numero massimo di connessioni simultanee da accettare. Usare `-1` per un numero infinito. Usare `null` per usare l'impostazione a livello di computer del Registro di sistema. | `null`<br>(a livello di computer<br>impostazione |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.MaxRequestBodySize> | Vedere la sezione <a href="#maxrequestbodysize">MaxRequestBodySize</a>. | 30000000 byte<br>(~28,6 MB) |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.RequestQueueLimit> | Numero massimo di richieste che è possibile accodare. | 1000 |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.ThrowWriteExceptions> | Indica se le scritture del corpo della risposta che hanno esito negativo a causa di disconnessioni del client devono generare eccezioni o vengono completate normalmente. | `false`<br>(completamento normale) |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.Timeouts> | Espone la configurazione di <xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager> HTTP.sys, che può essere configurata anche nel Registro di sistema. Seguire i collegamenti API per altre informazioni su ogni impostazione, inclusi i valori predefiniti:<ul><li>[TimeoutManager. DrainEntityBody](xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.DrainEntityBody): tempo consentito per l'API del server http per svuotare il corpo dell'entità in una connessione Keep-Alive.</li><li>[TimeoutManager. EntityBody](xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.EntityBody): tempo consentito per l'arrivo del corpo dell'entità di richiesta.</li><li>[TimeoutManager. HeaderWait](xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.HeaderWait): tempo consentito all'API del server http per analizzare l'intestazione della richiesta.</li><li>[TimeoutManager. IdleConnection](xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.IdleConnection): tempo consentito per una connessione inattiva.</li><li>[TimeoutManager. MinSendBytesPerSecond](xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.MinSendBytesPerSecond): frequenza di invio minima per la risposta.</li><li>[TimeoutManager. RequestQueue](xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.RequestQueue): tempo consentito affinché la richiesta rimanga nella coda di richieste prima che l'app lo prelevi.</li></ul> |  |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.UrlPrefixes> | Specificare l'elemento <xref:Microsoft.AspNetCore.Server.HttpSys.UrlPrefixCollection> da registrare con HTTP.sys. Il più utile è il metodo [UrlPrefixCollection.Add](xref:Microsoft.AspNetCore.Server.HttpSys.UrlPrefixCollection.Add*) usato per aggiungere un prefisso alla raccolta. Queste impostazioni possono essere modificate in qualsiasi momento prima dell'eliminazione del listener. |  |

<a name="maxrequestbodysize"></a>

**MaxRequestBodySize**

Dimensioni massime consentite per qualsiasi corpo della richiesta in byte. Con l'impostazione `null`, le dimensioni massime del corpo della richiesta sono illimitate. Questo limite non ha effetto sulle connessioni aggiornate, che sono sempre illimitate.

Il metodo consigliato per ignorare il limite in un'applicazione ASP.NET Core MVC per un singolo `IActionResult` prevede l'uso dell'attributo <xref:Microsoft.AspNetCore.Mvc.RequestSizeLimitAttribute> in un metodo di azione:

```csharp
[RequestSizeLimit(100000000)]
public IActionResult MyActionMethod()
```

Se l'app tenta di configurare il limite per una richiesta dopo che l'app ha avviato la lettura della richiesta stessa, viene generata un'eccezione. È possibile usare una proprietà `IsReadOnly` per indicare se la proprietà `MaxRequestBodySize` è in stato di sola lettura e pertanto è troppo tardi per configurare il limite.

Se l'app deve eseguire l'override di <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.MaxRequestBodySize> per ogni richiesta, usare <xref:Microsoft.AspNetCore.Http.Features.IHttpMaxRequestBodySizeFeature>:

[!code-csharp[](httpsys/samples/2.x/SampleApp/Startup.cs?name=snippet1&highlight=6-7)]

Se si usa Visual Studio, assicurarsi che l'app non sia configurata per l'esecuzione di IIS o IIS Express.

In Visual Studio il profilo di avvio predefinito è per IIS Express. Per eseguire il progetto come app console, modificare manualmente il profilo selezionato, come illustrato nello screenshot seguente:

![Selezionare il profilo dell'applicazione console](httpsys/_static/vs-choose-profile.png)

### <a name="configure-windows-server"></a>Configurare Windows Server

1. Determinare le porte da aprire per l'app e usare [Windows Firewall](/windows/security/threat-protection/windows-firewall/create-an-inbound-port-rule) o il cmdlet di PowerShell [New-NetFirewallRule](/powershell/module/netsecurity/new-netfirewallrule) per aprire le porte del firewall per consentire al traffico di raggiungere HTTP.sys. Nei comandi seguenti e nella configurazione dell'app, viene usata la porta 443.

1. Quando si distribuisce una macchina virtuale di Azure, aprire le porte nel [gruppo di sicurezza di rete](/azure/virtual-machines/windows/nsg-quickstart-portal). Nei comandi seguenti e nella configurazione dell'app, viene usata la porta 443.

1. Ottenere e installare i certificati X.509, se necessario.

   In Windows, creare certificati autofirmati con il [cmdlet di PowerShell New-SelfSignedCertificate](/powershell/module/pkiclient/new-selfsignedcertificate). Per un esempio non supportato, vedere [UpdateIISExpressSSLForChrome.ps1](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/includes/make-x509-cert/UpdateIISExpressSSLForChrome.ps1).

   Installare i certificati autofirmati o con firma nell'archivio **Computer locale** > **Personale** del server.

1. Se l'app è una [distribuzione dipendente dal framework](/dotnet/core/deploying/#framework-dependent-deployments-fdd), installare .NET Core, .NET Framework o entrambi (se l'app è un'app .NET Core destinata a .NET Framework).

   * **.NET Core**: se l'app richiede .NET Core, ottenere ed eseguire il programma di installazione del **runtime di .NET Core** da [download di .NET Core](https://dotnet.microsoft.com/download). Non installare l'SDK completo nel server.
   * **.NET Framework**: se l'app richiede .NET Framework, vedere la [Guida all'installazione di .NET Framework](/dotnet/framework/install/). Installare la versione di .NET Framework richiesta. Il programma di installazione per la versione più recente di .NET Framework è disponibile dalla pagina di [download per .NET Core](https://dotnet.microsoft.com/download).

   Se l'app è una [distribuzione autonoma](/dotnet/core/deploying/#self-contained-deployments-scd) include il runtime nella distribuzione. Non è richiesta l'installazione di un framework nel server.

1. Configurare gli URL e le porte nell'app.

   Per impostazione predefinita, ASP.NET Core è associato a `http://localhost:5000`. Per configurare le porte e i prefissi URL, è possibile usare:

   * <xref:Microsoft.AspNetCore.Hosting.HostingAbstractionsWebHostBuilderExtensions.UseUrls*>
   * L'argomento della riga di comando `urls`
   * La variabile di ambiente `ASPNETCORE_URLS`
   * <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.UrlPrefixes>

   L'esempio di codice seguente mostra come usare <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.UrlPrefixes> con l'indirizzo IP locale del server `10.0.0.4` sulla porta 443:

   [!code-csharp[](httpsys/samples_snapshot/2.x/Program.cs?highlight=6)]

   Un vantaggio offerto da `UrlPrefixes` è che viene generato immediatamente un messaggio di errore per i prefissi non formattati correttamente.

   Le impostazioni di `UrlPrefixes` sostituiscono le impostazioni `UseUrls`/`urls`/`ASPNETCORE_URLS`. Pertanto, un vantaggio offerto da `UseUrls`, `urls` e dalla variabile di ambiente `ASPNETCORE_URLS` è che risulta più semplice alternare Kestrel e HTTP.sys.

   HTTP.sys usa i [formati di stringa UrlPrefix dell'API del server HTTP](/windows/win32/http/urlprefix-strings).

   > [!WARNING]
   > Le associazioni con caratteri jolly di livello superiore (`http://*:80/` e `http://+:80`) **non** devono essere usate, poiché possono creare vulnerabilità a livello di sicurezza nell'app. Questo concetto vale sia per i caratteri jolly sicuri che vulnerabili. Usare nomi host o indirizzi IP espliciti al posto di caratteri jolly. L'associazione con caratteri jolly del sottodominio (ad esempio, `*.mysub.com`) non costituisce un rischio per la sicurezza se viene controllato l'intero dominio padre (a differenza di `*.com`, che è vulnerabile). Per ulteriori informazioni, vedere [RFC 7230: Section 5,4: host](https://tools.ietf.org/html/rfc7230#section-5.4).

1. Pre-registrare i prefissi URL nel server.

   Lo strumento predefinito per la configurazione di HTTP.sys è *netsh.exe*. *Netsh.exe* viene usato per riservare i prefissi URL e assegnare i certificati X.509. Per questo strumento sono necessari privilegi di amministratore.

   Usare lo strumento *netsh.exe* per registrare gli URL per l'app:

   ```console
   netsh http add urlacl url=<URL> user=<USER>
   ```

   * `<URL>`: Uniform Resource Locator completo (URL). Non usare un'associazione con caratteri jolly. Usare un nome host valido o un indirizzo IP locale. *L'URL deve includere una barra finale.*
   * `<USER>`: Specifica il nome dell'utente o del gruppo di utenti.

   Nell'esempio seguente, l'indirizzo IP locale del server è `10.0.0.4`:

   ```console
   netsh http add urlacl url=https://10.0.0.4:443/ user=Users
   ```

   Quando viene registrato un URL, lo strumento risponde con `URL reservation successfully added`.

   Per eliminare un URL registrato, usare il comando `delete urlacl`:

   ```console
   netsh http delete urlacl url=<URL>
   ```

1. Registrare i certificati X.509 nel server.

   Usare lo strumento *netsh.exe* per registrare i certificati per l'app:

   ```console
   netsh http add sslcert ipport=<IP>:<PORT> certhash=<THUMBPRINT> appid="{<GUID>}"
   ```

   * `<IP>`: Specifica l'indirizzo IP locale per l'associazione. Non usare un'associazione con caratteri jolly. Usare un indirizzo IP valido.
   * `<PORT>`: Specifica la porta per l'associazione.
   * `<THUMBPRINT>`: Identificazione personale del certificato X. 509.
   * `<GUID>`: GUID generato dallo sviluppatore per rappresentare l'app a scopo informativo.

   A scopo di riferimento, archiviare il GUID nell'app come tag di pacchetto:

   * In Visual Studio:
     * Aprire le proprietà del progetto dell'app facendo clic sull'app con il pulsante destro del mouse in **Esplora soluzioni** e scegliendo **Proprietà**.
     * Selezionare la scheda **Pacchetto**.
     * Immettere il GUID creato nel campo **Tag**.
   * Se non si usa Visual Studio:
     * Aprire il file di progetto dell'app.
     * Aggiungere una proprietà `<PackageTags>` a un `<PropertyGroup>` nuovo o esistente con il GUID creato:

       ```xml
       <PropertyGroup>
         <PackageTags>9412ee86-c21b-4eb8-bd89-f650fbf44931</PackageTags>
       </PropertyGroup>
       ```

   Nell'esempio seguente:

   * L'indirizzo IP locale del server è `10.0.0.4`.
   * Un generatore di GUID casuali online fornisce il valore `appid`.

   ```console
   netsh http add sslcert 
       ipport=10.0.0.4:443 
       certhash=b66ee04419d4ee37464ab8785ff02449980eae10 
       appid="{9412ee86-c21b-4eb8-bd89-f650fbf44931}"
   ```

   Quando viene registrato un certificato, lo strumento risponde con `SSL Certificate successfully added`.

   Per eliminare la registrazione di un certificato, usare il comando `delete sslcert`:

   ```console
   netsh http delete sslcert ipport=<IP>:<PORT>
   ```

   Documentazione di riferimento per *netsh.exe*:

   * [Comandi netsh per HTTP (Hypertext Transfer Protocol)](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc725882(v=ws.10))
   * [UrlPrefix Strings](/windows/win32/http/urlprefix-strings) (Stringhe UrlPrefix)

1. Eseguire l'app.

   Non sono necessari i privilegi di amministratore per eseguire l'app con binding a localhost tramite HTTP (non HTTPS) con un numero di porta maggiore di 1024. Per altre configurazioni (ad esempio, l'uso di un indirizzo IP locale o il binding sulla porta 443), eseguire l'app con i privilegi di amministratore.

   L'app risponde all'indirizzo IP pubblico del server. In questo esempio, il server è raggiungibile da Internet al relativo indirizzo IP pubblico `104.214.79.47`.

   In questo esempio viene usato un certificato di sviluppo. La pagina viene caricata in modo sicuro dopo aver ignorato l'avviso di certificato non attendibile del browser.

   ![Finestra del browser che mostra la pagina di indice dell'app caricata](httpsys/_static/browser.png)

## <a name="proxy-server-and-load-balancer-scenarios"></a>Scenari con server proxy e servizi di bilanciamento del carico

Per le app ospitate da HTTP.sys che interagiscono con richieste da Internet o da una rete aziendale, potrebbero essere necessari interventi di configurazione aggiuntivi in caso di hosting dietro server proxy e servizi di bilanciamento del carico. Per altre informazioni, vedere [Configurare ASP.NET Core per l'utilizzo di server proxy e servizi di bilanciamento del carico](xref:host-and-deploy/proxy-load-balancer).

## <a name="additional-resources"></a>Risorse aggiuntive

* [Abilitare l'autenticazione di Windows con HTTP. sys](xref:security/authentication/windowsauth#httpsys)
* [API di HTTP Server](/windows/win32/http/http-api-start-page)
* [Repository di GitHub aspnet/HttpSysServer (codice sorgente)](https://github.com/aspnet/HttpSysServer/)
* [Host](xref:fundamentals/index#host)
* <xref:test/troubleshoot>

::: moniker-end

::: moniker range="< aspnetcore-2.2"

[HTTP.sys](/iis/get-started/introduction-to-iis/introduction-to-iis-architecture#hypertext-transfer-protocol-stack-httpsys) è un [server Web per ASP.NET Core](xref:fundamentals/servers/index) che può essere eseguito solo in Windows. HTTP.sys è un'alternativa al server [Kestrel](xref:fundamentals/servers/kestrel) e offre alcune funzionalità non presenti in Kestrel.

> [!IMPORTANT]
> HTTP.sys non è compatibile con il [modulo ASP.NET Core](xref:host-and-deploy/aspnet-core-module) e non può essere usato con IIS o IIS Express.

HTTP.sys supporta le funzionalità seguenti:

* [Autenticazione di Windows](xref:security/authentication/windowsauth)
* Condivisione delle porte
* HTTPS con SNI
* HTTP/2 su TLS (Windows 10 o versioni successive)
* Trasmissione diretta dei file
* Memorizzazione nella cache delle risposte
* WebSockets (Windows 8 o versioni successive)

Versioni supportate di Windows:

* Windows 7 o versione successiva
* Windows Server 2008 R2 o versione successiva

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/servers/httpsys/samples) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="when-to-use-httpsys"></a>Quando usare HTTP.sys

HTTP.sys è utile per le distribuzioni nei casi seguenti:

* È necessario esporre il server direttamente a Internet senza usare IIS.

  ![HTTP.sys comunica direttamente con Internet](httpsys/_static/httpsys-to-internet.png)

* Una distribuzione interna richiede una funzionalità non disponibile in Kestrel, ad esempio l'[autenticazione di Windows](xref:security/authentication/windowsauth).

  ![HTTP.sys comunica direttamente con la rete interna](httpsys/_static/httpsys-to-internal.png)

HTTP.sys è una tecnologia consolidata che protegge da molti tipi di attacchi e offre l'affidabilità, la sicurezza e la scalabilità di un server Web con funzionalità complete. IIS viene eseguito come listener HTTP in HTTP.sys.

## <a name="http2-support"></a>Supporto HTTP/2

[HTTP/2](https://httpwg.org/specs/rfc7540.html) è abilitato per le app ASP.NET Core se vengono soddisfatti i requisiti di base seguenti:

* Windows Server 2016/Windows 10 o versioni successive
* Connessione [ALPN (Application-Layer Protocol Negotiation)](https://tools.ietf.org/html/rfc7301#section-3)
* Connessione TLS 1.2 o successiva

Se viene stabilita una connessione HTTP/2, [HttpRequest.Protocol](xref:Microsoft.AspNetCore.Http.HttpRequest.Protocol*) corrisponde a `HTTP/1.1`.

HTTP/2 è abilitato per impostazione predefinita. Se non viene stabilita una connessione HTTP/2, la connessione esegue il fallback a HTTP/1.1. In una versione futura di Windows sarà disponibile il flag di configurazione di HTTP/2 e sarà possibile disabilitare il protocollo HTTP/2 con HTTP.sys.

## <a name="kernel-mode-authentication-with-kerberos"></a>Autenticazione in modalità kernel con Kerberos

Per la delega all'autenticazione in modalità kernel, HTTP.sys usa il protocollo di autenticazione Kerberos. L'autenticazione in modalità utente non è supportata con Kerberos e HTTP.sys. È necessario usare l'account del computer per decrittografare il token/ticket Kerberos ottenuto da Active Directory e inoltrato dal client al server per autenticare l'utente. Registrare il nome dell'entità servizio per l'host, non l'utente dell'app.

## <a name="how-to-use-httpsys"></a>Come usare HTTP.sys

### <a name="configure-the-aspnet-core-app-to-use-httpsys"></a>Configurare l'app ASP.NET Core per l'uso di HTTP.sys

Non è necessario un riferimento al pacchetto nel file di progetto quando si usa il [metapacchetto Microsoft. AspNetCore. app](xref:fundamentals/metapackage-app) ([NuGet.org](https://www.nuget.org/packages/Microsoft.AspNetCore.App/)). Se non si usa il metapacchetto `Microsoft.AspNetCore.App` aggiungere un riferimento al pacchetto a [Microsoft.AspNetCore.Server.HttpSys](https://www.nuget.org/packages/Microsoft.AspNetCore.Server.HttpSys/).

Chiamare il metodo di estensione <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderHttpSysExtensions.UseHttpSys*> quando si compila l'host, specificando le eventuali <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions> necessarie. L'esempio seguente imposta le opzioni sui rispettivi valori predefiniti:

[!code-csharp[](httpsys/samples/2.x/SampleApp/Program.cs?name=snippet1&highlight=4-12)]

Le configurazioni aggiuntive di HTTP.sys vengono gestite tramite [impostazioni del Registro di sistema](https://support.microsoft.com/help/820129/http-sys-registry-settings-for-windows).

**Opzioni di HTTP.sys**

| Proprietà | Descrizione | Predefinito |
| -------- | ----------- | :-----: |
| [AllowSynchronousIO](xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.AllowSynchronousIO) | Controllare se l'input e/o l'output sincroni sono consentiti per `HttpContext.Request.Body` e `HttpContext.Response.Body`. | `true` |
| [Authentication.AllowAnonymous](xref:Microsoft.AspNetCore.Server.HttpSys.AuthenticationManager.AllowAnonymous) | Consentire richieste anonime. | `true` |
| [Authentication.Schemes](xref:Microsoft.AspNetCore.Server.HttpSys.AuthenticationManager.Schemes) | Specificare gli schemi di autenticazione consentiti. Può essere modificata in qualsiasi momento prima dell'eliminazione del listener. I valori vengono forniti dall' [enumerazione AuthenticationSchemes](xref:Microsoft.AspNetCore.Server.HttpSys.AuthenticationSchemes): `Basic` , `Kerberos` , `Negotiate` , `None` e `NTLM` . | `None` |
| [EnableResponseCaching](xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.EnableResponseCaching) | Tentare la memorizzazione nella cache in [modalità kernel](/windows-hardware/drivers/gettingstarted/user-mode-and-kernel-mode) per le risposte con intestazioni idonee. La risposta potrebbe non includere intestazioni `Set-Cookie`, `Vary` o `Pragma`. Deve includere un'intestazione `Cache-Control``public` con valore `shared-max-age` o `max-age` o un'intestazione `Expires`. | `true` |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.MaxAccepts> | Numero massimo di accettazioni simultanee. | 5 &times; [Environment.<br>ProcessorCount](xref:System.Environment.ProcessorCount) |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.MaxConnections> | Numero massimo di connessioni simultanee da accettare. Usare `-1` per un numero infinito. Usare `null` per usare l'impostazione a livello di computer del Registro di sistema. | `null`<br>(a livello di computer<br>impostazione |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.MaxRequestBodySize> | Vedere la sezione <a href="#maxrequestbodysize">MaxRequestBodySize</a>. | 30000000 byte<br>(~28,6 MB) |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.RequestQueueLimit> | Numero massimo di richieste che è possibile accodare. | 1000 |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.ThrowWriteExceptions> | Indica se le scritture del corpo della risposta che hanno esito negativo a causa di disconnessioni del client devono generare eccezioni o vengono completate normalmente. | `false`<br>(completamento normale) |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.Timeouts> | Espone la configurazione di <xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager> HTTP.sys, che può essere configurata anche nel Registro di sistema. Seguire i collegamenti API per altre informazioni su ogni impostazione, inclusi i valori predefiniti:<ul><li>[TimeoutManager. DrainEntityBody](xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.DrainEntityBody): tempo consentito per l'API del server http per svuotare il corpo dell'entità in una connessione Keep-Alive.</li><li>[TimeoutManager. EntityBody](xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.EntityBody): tempo consentito per l'arrivo del corpo dell'entità di richiesta.</li><li>[TimeoutManager. HeaderWait](xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.HeaderWait): tempo consentito all'API del server http per analizzare l'intestazione della richiesta.</li><li>[TimeoutManager. IdleConnection](xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.IdleConnection): tempo consentito per una connessione inattiva.</li><li>[TimeoutManager. MinSendBytesPerSecond](xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.MinSendBytesPerSecond): frequenza di invio minima per la risposta.</li><li>[TimeoutManager. RequestQueue](xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.RequestQueue): tempo consentito affinché la richiesta rimanga nella coda di richieste prima che l'app lo prelevi.</li></ul> |  |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.UrlPrefixes> | Specificare l'elemento <xref:Microsoft.AspNetCore.Server.HttpSys.UrlPrefixCollection> da registrare con HTTP.sys. Il più utile è il metodo [UrlPrefixCollection.Add](xref:Microsoft.AspNetCore.Server.HttpSys.UrlPrefixCollection.Add*) usato per aggiungere un prefisso alla raccolta. Queste impostazioni possono essere modificate in qualsiasi momento prima dell'eliminazione del listener. |  |

<a name="maxrequestbodysize"></a>

**MaxRequestBodySize**

Dimensioni massime consentite per qualsiasi corpo della richiesta in byte. Con l'impostazione `null`, le dimensioni massime del corpo della richiesta sono illimitate. Questo limite non ha effetto sulle connessioni aggiornate, che sono sempre illimitate.

Il metodo consigliato per ignorare il limite in un'applicazione ASP.NET Core MVC per un singolo `IActionResult` prevede l'uso dell'attributo <xref:Microsoft.AspNetCore.Mvc.RequestSizeLimitAttribute> in un metodo di azione:

```csharp
[RequestSizeLimit(100000000)]
public IActionResult MyActionMethod()
```

Se l'app tenta di configurare il limite per una richiesta dopo che l'app ha avviato la lettura della richiesta stessa, viene generata un'eccezione. È possibile usare una proprietà `IsReadOnly` per indicare se la proprietà `MaxRequestBodySize` è in stato di sola lettura e pertanto è troppo tardi per configurare il limite.

Se l'app deve eseguire l'override di <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.MaxRequestBodySize> per ogni richiesta, usare <xref:Microsoft.AspNetCore.Http.Features.IHttpMaxRequestBodySizeFeature>:

[!code-csharp[](httpsys/samples/2.x/SampleApp/Startup.cs?name=snippet1&highlight=6-7)]

Se si usa Visual Studio, assicurarsi che l'app non sia configurata per l'esecuzione di IIS o IIS Express.

In Visual Studio il profilo di avvio predefinito è per IIS Express. Per eseguire il progetto come app console, modificare manualmente il profilo selezionato, come illustrato nello screenshot seguente:

![Selezionare il profilo dell'applicazione console](httpsys/_static/vs-choose-profile.png)

### <a name="configure-windows-server"></a>Configurare Windows Server

1. Determinare le porte da aprire per l'app e usare [Windows Firewall](/windows/security/threat-protection/windows-firewall/create-an-inbound-port-rule) o il cmdlet di PowerShell [New-NetFirewallRule](/powershell/module/netsecurity/new-netfirewallrule) per aprire le porte del firewall per consentire al traffico di raggiungere HTTP.sys. Nei comandi seguenti e nella configurazione dell'app, viene usata la porta 443.

1. Quando si distribuisce una macchina virtuale di Azure, aprire le porte nel [gruppo di sicurezza di rete](/azure/virtual-machines/windows/nsg-quickstart-portal). Nei comandi seguenti e nella configurazione dell'app, viene usata la porta 443.

1. Ottenere e installare i certificati X.509, se necessario.

   In Windows, creare certificati autofirmati con il [cmdlet di PowerShell New-SelfSignedCertificate](/powershell/module/pkiclient/new-selfsignedcertificate). Per un esempio non supportato, vedere [UpdateIISExpressSSLForChrome.ps1](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/includes/make-x509-cert/UpdateIISExpressSSLForChrome.ps1).

   Installare i certificati autofirmati o con firma nell'archivio **Computer locale** > **Personale** del server.

1. Se l'app è una [distribuzione dipendente dal framework](/dotnet/core/deploying/#framework-dependent-deployments-fdd), installare .NET Core, .NET Framework o entrambi (se l'app è un'app .NET Core destinata a .NET Framework).

   * **.NET Core**: se l'app richiede .NET Core, ottenere ed eseguire il programma di installazione del **runtime di .NET Core** da [download di .NET Core](https://dotnet.microsoft.com/download). Non installare l'SDK completo nel server.
   * **.NET Framework**: se l'app richiede .NET Framework, vedere la [Guida all'installazione di .NET Framework](/dotnet/framework/install/). Installare la versione di .NET Framework richiesta. Il programma di installazione per la versione più recente di .NET Framework è disponibile dalla pagina di [download per .NET Core](https://dotnet.microsoft.com/download).

   Se l'app è una [distribuzione autonoma](/dotnet/core/deploying/#self-contained-deployments-scd) include il runtime nella distribuzione. Non è richiesta l'installazione di un framework nel server.

1. Configurare gli URL e le porte nell'app.

   Per impostazione predefinita, ASP.NET Core è associato a `http://localhost:5000`. Per configurare le porte e i prefissi URL, è possibile usare:

   * <xref:Microsoft.AspNetCore.Hosting.HostingAbstractionsWebHostBuilderExtensions.UseUrls*>
   * L'argomento della riga di comando `urls`
   * La variabile di ambiente `ASPNETCORE_URLS`
   * <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.UrlPrefixes>

   L'esempio di codice seguente mostra come usare <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.UrlPrefixes> con l'indirizzo IP locale del server `10.0.0.4` sulla porta 443:

   [!code-csharp[](httpsys/samples_snapshot/2.x/Program.cs?highlight=6)]

   Un vantaggio offerto da `UrlPrefixes` è che viene generato immediatamente un messaggio di errore per i prefissi non formattati correttamente.

   Le impostazioni di `UrlPrefixes` sostituiscono le impostazioni `UseUrls`/`urls`/`ASPNETCORE_URLS`. Pertanto, un vantaggio offerto da `UseUrls`, `urls` e dalla variabile di ambiente `ASPNETCORE_URLS` è che risulta più semplice alternare Kestrel e HTTP.sys.

   HTTP.sys usa i [formati di stringa UrlPrefix dell'API del server HTTP](/windows/win32/http/urlprefix-strings).

   > [!WARNING]
   > Le associazioni con caratteri jolly di livello superiore (`http://*:80/` e `http://+:80`) **non** devono essere usate, poiché possono creare vulnerabilità a livello di sicurezza nell'app. Questo concetto vale sia per i caratteri jolly sicuri che vulnerabili. Usare nomi host o indirizzi IP espliciti al posto di caratteri jolly. L'associazione con caratteri jolly del sottodominio (ad esempio, `*.mysub.com`) non costituisce un rischio per la sicurezza se viene controllato l'intero dominio padre (a differenza di `*.com`, che è vulnerabile). Per ulteriori informazioni, vedere [RFC 7230: Section 5,4: host](https://tools.ietf.org/html/rfc7230#section-5.4).

1. Pre-registrare i prefissi URL nel server.

   Lo strumento predefinito per la configurazione di HTTP.sys è *netsh.exe*. *Netsh.exe* viene usato per riservare i prefissi URL e assegnare i certificati X.509. Per questo strumento sono necessari privilegi di amministratore.

   Usare lo strumento *netsh.exe* per registrare gli URL per l'app:

   ```console
   netsh http add urlacl url=<URL> user=<USER>
   ```

   * `<URL>`: Uniform Resource Locator completo (URL). Non usare un'associazione con caratteri jolly. Usare un nome host valido o un indirizzo IP locale. *L'URL deve includere una barra finale.*
   * `<USER>`: Specifica il nome dell'utente o del gruppo di utenti.

   Nell'esempio seguente, l'indirizzo IP locale del server è `10.0.0.4`:

   ```console
   netsh http add urlacl url=https://10.0.0.4:443/ user=Users
   ```

   Quando viene registrato un URL, lo strumento risponde con `URL reservation successfully added`.

   Per eliminare un URL registrato, usare il comando `delete urlacl`:

   ```console
   netsh http delete urlacl url=<URL>
   ```

1. Registrare i certificati X.509 nel server.

   Usare lo strumento *netsh.exe* per registrare i certificati per l'app:

   ```console
   netsh http add sslcert ipport=<IP>:<PORT> certhash=<THUMBPRINT> appid="{<GUID>}"
   ```

   * `<IP>`: Specifica l'indirizzo IP locale per l'associazione. Non usare un'associazione con caratteri jolly. Usare un indirizzo IP valido.
   * `<PORT>`: Specifica la porta per l'associazione.
   * `<THUMBPRINT>`: Identificazione personale del certificato X. 509.
   * `<GUID>`: GUID generato dallo sviluppatore per rappresentare l'app a scopo informativo.

   A scopo di riferimento, archiviare il GUID nell'app come tag di pacchetto:

   * In Visual Studio:
     * Aprire le proprietà del progetto dell'app facendo clic sull'app con il pulsante destro del mouse in **Esplora soluzioni** e scegliendo **Proprietà**.
     * Selezionare la scheda **Pacchetto**.
     * Immettere il GUID creato nel campo **Tag**.
   * Se non si usa Visual Studio:
     * Aprire il file di progetto dell'app.
     * Aggiungere una proprietà `<PackageTags>` a un `<PropertyGroup>` nuovo o esistente con il GUID creato:

       ```xml
       <PropertyGroup>
         <PackageTags>9412ee86-c21b-4eb8-bd89-f650fbf44931</PackageTags>
       </PropertyGroup>
       ```

   Nell'esempio seguente:

   * L'indirizzo IP locale del server è `10.0.0.4`.
   * Un generatore di GUID casuali online fornisce il valore `appid`.

   ```console
   netsh http add sslcert 
       ipport=10.0.0.4:443 
       certhash=b66ee04419d4ee37464ab8785ff02449980eae10 
       appid="{9412ee86-c21b-4eb8-bd89-f650fbf44931}"
   ```

   Quando viene registrato un certificato, lo strumento risponde con `SSL Certificate successfully added`.

   Per eliminare la registrazione di un certificato, usare il comando `delete sslcert`:

   ```console
   netsh http delete sslcert ipport=<IP>:<PORT>
   ```

   Documentazione di riferimento per *netsh.exe*:

   * [Comandi netsh per HTTP (Hypertext Transfer Protocol)](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc725882(v=ws.10))
   * [UrlPrefix Strings](/windows/win32/http/urlprefix-strings) (Stringhe UrlPrefix)

1. Eseguire l'app.

   Non sono necessari i privilegi di amministratore per eseguire l'app con binding a localhost tramite HTTP (non HTTPS) con un numero di porta maggiore di 1024. Per altre configurazioni (ad esempio, l'uso di un indirizzo IP locale o il binding sulla porta 443), eseguire l'app con i privilegi di amministratore.

   L'app risponde all'indirizzo IP pubblico del server. In questo esempio, il server è raggiungibile da Internet al relativo indirizzo IP pubblico `104.214.79.47`.

   In questo esempio viene usato un certificato di sviluppo. La pagina viene caricata in modo sicuro dopo aver ignorato l'avviso di certificato non attendibile del browser.

   ![Finestra del browser che mostra la pagina di indice dell'app caricata](httpsys/_static/browser.png)

## <a name="proxy-server-and-load-balancer-scenarios"></a>Scenari con server proxy e servizi di bilanciamento del carico

Per le app ospitate da HTTP.sys che interagiscono con richieste da Internet o da una rete aziendale, potrebbero essere necessari interventi di configurazione aggiuntivi in caso di hosting dietro server proxy e servizi di bilanciamento del carico. Per altre informazioni, vedere [Configurare ASP.NET Core per l'utilizzo di server proxy e servizi di bilanciamento del carico](xref:host-and-deploy/proxy-load-balancer).

## <a name="additional-resources"></a>Risorse aggiuntive

* [Abilitare l'autenticazione di Windows con HTTP. sys](xref:security/authentication/windowsauth#httpsys)
* [API di HTTP Server](/windows/win32/http/http-api-start-page)
* [Repository di GitHub aspnet/HttpSysServer (codice sorgente)](https://github.com/aspnet/HttpSysServer/)
* [Host](xref:fundamentals/index#host)
* <xref:test/troubleshoot>

::: moniker-end
