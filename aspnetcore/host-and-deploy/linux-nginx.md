---
title: Hosting di ASP.NET Core in Linux con Nginx
author: rick-anderson
description: Informazioni su come configurare nginx come proxy inverso in Ubuntu 16,04 per l'inoltro del traffico HTTP a una ASP.NET Core app Web in esecuzione in gheppio.
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 10/30/2020
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
uid: host-and-deploy/linux-nginx
ms.openlocfilehash: 8a593654fa31e643e7c239f361f035589c75ce98
ms.sourcegitcommit: 1436bd4d70937d6ec3140da56d96caab33c4320b
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/06/2021
ms.locfileid: "102395253"
---
# <a name="host-aspnet-core-on-linux-with-nginx"></a>Hosting di ASP.NET Core in Linux con Nginx

Di [Sourabh Shirhatti](https://twitter.com/sshirhatti)

Questa guida descrive come configurare un ambiente ASP.NET Core pronto per la produzione in un server Ubuntu 16.04. Queste istruzioni si applicano probabilmente anche alle versioni più recenti di Ubuntu, sebbene non siano state testate con le versioni più recenti.

Per informazioni su altre distribuzioni Linux supportate da ASP.NET Core, vedere [Prerequisiti per .NET Core in Linux](/dotnet/core/linux-prerequisites).

> [!NOTE]
> Per Ubuntu 14,04, `supervisord` è consigliabile come soluzione per il monitoraggio del processo gheppio. `systemd` non è disponibile in Ubuntu 14,04. Per le istruzioni per Ubuntu 14.04, vedere la [versione precedente di questo argomento](https://github.com/dotnet/AspNetCore.Docs/blob/e9c1419175c4dd7e152df3746ba1df5935aaafd5/aspnetcore/publishing/linuxproduction.md).

In questa guida:

* Posizionare un'app ASP.NET Core esistente dietro un server proxy inverso.
* Configurare il server proxy inverso per l'inoltro delle richieste al server Web Kestrel.
* Verificare che l'app Web venga eseguita all'avvio come daemon.
* Configurare uno strumento di gestione del processo per consentire il riavvio dell'app Web.

## <a name="prerequisites"></a>Prerequisiti

* Accedere a un server Ubuntu 16.04 con un account utente standard con privilegio sudo.
* Il [Runtime .NET](/dotnet/core/install/linux) non di anteprima più recente installato nel server.
* Un'app ASP.NET Core esistente.

In qualsiasi momento in futuro dopo l'aggiornamento del Framework condiviso, riavviare le app ASP.NET Core ospitate dal server.

## <a name="publish-and-copy-over-the-app"></a>Pubblicare e copiare l'app

Configurare l'app per una [distribuzione dipendente dal framework](/dotnet/core/deploying/#framework-dependent-deployments-fdd).

Se l'app viene eseguita in locale e non è configurata per connessioni sicure (HTTPS), adottare uno degli approcci seguenti:

* Configurare l'app per la gestione di connessioni locali sicure. Per altre informazioni, vedere la sezione [Configurazione HTTPS](#https-configuration).
* Rimuovere `https://localhost:5001` , se presente, dalla `applicationUrl` proprietà nel `Properties/launchSettings.json` file.

Eseguire [DotNet Publish](/dotnet/core/tools/dotnet-publish) dall'ambiente di sviluppo per creare un pacchetto di un'app in una directory (ad esempio, `bin/Release/{TARGET FRAMEWORK MONIKER}/publish` , dove il segnaposto `{TARGET FRAMEWORK MONIKER}` è il moniker del Framework di destinazione/TFM) che può essere eseguito nel server:

```dotnetcli
dotnet publish --configuration Release
```

È anche possibile pubblicare l'app come [distribuzione completa](/dotnet/core/deploying/#self-contained-deployments-scd) se si preferisce non mantenere il runtime .NET Core nel server.

Copiare l'app ASP.NET Core nel server usando uno strumento che si integra con il flusso di lavoro dell'organizzazione, ad esempio `SCP` , `SFTP` . È comune individuare le app Web nella `var` Directory (ad esempio, `var/www/helloapp` ).

> [!NOTE]
> In uno scenario di distribuzione di produzione un flusso di lavoro di integrazione continua esegue le operazioni di pubblicazione dell'app e di copia degli asset nel server.

Eseguire il test dell'app:

1. Dalla riga di comando, eseguire l'app: `dotnet <app_assembly>.dll`.
1. In un browser passare a `http://<serveraddress>:<port>` per verificare se l'app funziona in Linux in locale.

## <a name="configure-a-reverse-proxy-server"></a>Configurare un server proxy inverso

Un proxy inverso è una configurazione comune per la gestione delle app Web dinamiche. Un proxy inverso termina la richiesta HTTP e la inoltra all'app ASP.NET Core.

### <a name="use-a-reverse-proxy-server"></a>Usare un server proxy inverso

Kestrel funziona perfettamente per la gestione del contenuto dinamico da ASP.NET Core. Tuttavia, le funzionalità di gestione Web sono meno avanzate rispetto a quelle offerte da server come IIS, Apache o Nginx. Un server proxy inverso è in grado di eseguire l'offload di attività come la gestione del contenuto statico, la memorizzazione nella cache delle richieste, la compressione delle richieste e la terminazione HTTPS dal server HTTP. Un server proxy inverso può risiedere in un computer dedicato o essere distribuito insieme a un server HTTP.

Ai fini di questa guida viene usata una singola istanza di Nginx. Viene eseguito sullo stesso server, insieme al server HTTP. In base ai requisiti, è possibile scegliere una configurazione diversa.

Poiché le richieste vengono inviate dal proxy inverso, utilizzare il [middleware delle intestazioni inoltro](xref:host-and-deploy/proxy-load-balancer) dal [`Microsoft.AspNetCore.HttpOverrides`](https://www.nuget.org/packages/Microsoft.AspNetCore.HttpOverrides) pacchetto. Il middleware aggiorna `Request.Scheme` usando l'intestazione `X-Forwarded-Proto`, in modo che gli URI di reindirizzamento e altri criteri di sicurezza funzionino correttamente.

[!INCLUDE[](~/includes/ForwardedHeaders.md)]

Richiamare il <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersExtensions.UseForwardedHeaders%2A> metodo all'inizio di `Startup.Configure` prima di chiamare altro middleware. Configurare il middleware per l'inoltro delle intestazioni `X-Forwarded-For` e `X-Forwarded-Proto`:

```csharp
using Microsoft.AspNetCore.HttpOverrides;

...

app.UseForwardedHeaders(new ForwardedHeadersOptions
{
    ForwardedHeaders = ForwardedHeaders.XForwardedFor | ForwardedHeaders.XForwardedProto
});

app.UseAuthentication();
```

Se non vengono specificate opzioni <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions> per il middleware, le intestazioni predefinite per l'inoltro sono `None`.

I proxy eseguiti sugli indirizzi di loopback ( `127.0.0.0/8` , `[::1]` ), incluso l'indirizzo localhost standard ( `127.0.0.1` ), sono considerati attendibili per impostazione predefinita. Se le richieste tra Internet e il server Web vengono gestite anche da altri proxy o reti attendibili all'interno dell'organizzazione, aggiungerli all'elenco di <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.KnownProxies%2A> o <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.KnownNetworks%2A> con <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions>. L'esempio seguente aggiunge un server proxy attendibile all'indirizzo IP 10.0.0.100 nel middleware delle intestazioni inoltrate `KnownProxies` in `Startup.ConfigureServices`:

```csharp
using System.Net;

...

services.Configure<ForwardedHeadersOptions>(options =>
{
    options.KnownProxies.Add(IPAddress.Parse("10.0.0.100"));
});
```

Per altre informazioni, vedere <xref:host-and-deploy/proxy-load-balancer>.

### <a name="install-nginx"></a>Installare Nginx

Usare `apt-get` per installare Nginx. Il programma di installazione crea uno `systemd` script di inizializzazione che esegue nginx come daemon all'avvio del sistema. Seguire le istruzioni di installazione per Ubuntu riportate nell'articolo [Nginx: Official Debian/Ubuntu packages](https://www.nginx.com/resources/wiki/start/topics/tutorials/install/#official-debian-ubuntu-packages) (Nginx: pacchetti Debian/Ubuntu ufficiali).

> [!NOTE]
> Se sono richiesti moduli Nginx facoltativi, potrebbe essere necessario compilare Nginx dall'origine.

Poiché Nginx è stato installato per la prima volta, avviarlo in modo esplicito eseguendo:

```bash
sudo service nginx start
```

Verificare che un browser visualizzi la pagina di destinazione predefinita per Nginx. La pagina di destinazione è raggiungibile all'indirizzo `http://<server_IP_address>/index.nginx-debian.html`.

### <a name="configure-nginx"></a>Configurare Nginx

Per configurare nginx come proxy inverso per l'inoltro delle richieste HTTP all'app ASP.NET Core, modificare `/etc/nginx/sites-available/default` . Aprirlo in un editor di testo e sostituire il contenuto con il frammento di codice seguente:

```nginx
server {
    listen        80;
    server_name   example.com *.example.com;
    location / {
        proxy_pass         http://localhost:5000;
        proxy_http_version 1.1;
        proxy_set_header   Upgrade $http_upgrade;
        proxy_set_header   Connection keep-alive;
        proxy_set_header   Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header   X-Forwarded-Proto $scheme;
    }
}
```

Se l'app è un' SignalR Blazor Server app o, vedere <xref:signalr/scale#linux-with-nginx> <xref:blazor/host-and-deploy/server#linux-with-nginx> rispettivamente e per altre informazioni.

Se nessun `server_name` corrisponde, Nginx usa il server predefinito. Se non è definito alcun server predefinito, il primo server nel file di configurazione è il server predefinito. Come procedura consigliata, aggiungere un server predefinito specifico che restituisca un codice di stato 444 nel file di configurazione. Un esempio di configurazione del server predefinito è il seguente:

```nginx
server {
    listen   80 default_server;
    # listen [::]:80 default_server deferred;
    return   444;
}
```

::: moniker range=">= aspnetcore-5.0"

Con il file di configurazione precedente e il server predefinito, Nginx accetta il traffico pubblico sulla porta 80 con l'intestazione host `example.com` o `*.example.com`. Le richieste che non corrispondono a questi host non verranno inoltrate a Kestrel. Nginx inoltra a Kestrel le richieste corrispondenti a `http://localhost:5000`. Per altre informazioni, vedere [come Nginx elabora una richiesta](https://nginx.org/docs/http/request_processing.html). Per cambiare porta/IP di Kestrel, vedere [Kestrel: Configurazione dell'endpoint](xref:fundamentals/servers/kestrel/endpoints).

::: moniker-end

::: moniker range="< aspnetcore-5.0"

Con il file di configurazione precedente e il server predefinito, Nginx accetta il traffico pubblico sulla porta 80 con l'intestazione host `example.com` o `*.example.com`. Le richieste che non corrispondono a questi host non verranno inoltrate a Kestrel. Nginx inoltra a Kestrel le richieste corrispondenti a `http://localhost:5000`. Per altre informazioni, vedere [come Nginx elabora una richiesta](https://nginx.org/docs/http/request_processing.html). Per cambiare porta/IP di Kestrel, vedere [Kestrel: Configurazione dell'endpoint](xref:fundamentals/servers/kestrel#endpoint-configuration).

::: moniker-end

> [!WARNING]
> Se non si specifica una [direttiva server_name](https://nginx.org/docs/http/server_names.html) corretta, l'app può risultare esposta a vulnerabilità di sicurezza. L'associazione con caratteri jolly del sottodominio (ad esempio, `*.example.com`) non costituisce un rischio per la sicurezza se viene controllato l'intero dominio padre (a differenza di `*.com`, che è vulnerabile). Per ulteriori informazioni, vedere la [sezione rfc7230-5,4](https://tools.ietf.org/html/rfc7230#section-5.4).

Una volta stabilita la configurazione di Nginx, eseguire `sudo nginx -t` per verificare la sintassi dei file di configurazione. Se il test dei file di configurazione ha esito positivo, è possibile forzare Nginx ad applicare le modifiche eseguendo `sudo nginx -s reload`.

Per eseguire direttamente l'app nel server:

1. Passare alla directory dell'app.
1. Eseguire l'app: `dotnet <app_assembly.dll>`, dove `app_assembly.dll` è il nome di file di assembly dell'app.

Se l'app viene eseguita nel server ma non riesce a rispondere tramite Internet, controllare il firewall del server e verificare che la porta 80 sia aperta. Se si usa una macchina virtuale Ubuntu di Azure, aggiungere una regola del gruppo di sicurezza di rete (NSG) che abiliti il traffico in ingresso della porta 80. Non è necessario abilitare una regola per il traffico in uscita della porta 80, poiché il traffico in uscita viene autorizzato automaticamente quando viene abilitata la regola in ingresso.

Al termine del test dell'app, arrestare l'app con <kbd>CTRL</kbd>  +  <kbd>C</kbd> al prompt dei comandi.

## <a name="monitor-the-app"></a>Monitorare l'app

Il server è configurato per l'invio delle richieste effettuate a al `http://<serveraddress>:80` ASP.NET Core app in esecuzione in gheppio all'indirizzo `http://127.0.0.1:5000` . Tuttavia, Nginx non è configurato per la gestione del processo Kestrel. `systemd` può essere usato per creare un file di servizio per avviare e monitorare l'app Web sottostante. `systemd` è un sistema di inizializzazione che offre molte funzionalità potenti per l'avvio, l'arresto e la gestione dei processi. 

### <a name="create-the-service-file"></a>Creare il file del servizio

Creare il file di definizione del servizio:

```bash
sudo nano /etc/systemd/system/kestrel-helloapp.service
```

L'esempio seguente è un file del servizio per l'app:

```ini
[Unit]
Description=Example .NET Web API App running on Ubuntu

[Service]
WorkingDirectory=/var/www/helloapp
ExecStart=/usr/bin/dotnet /var/www/helloapp/helloapp.dll
Restart=always
# Restart service after 10 seconds if the dotnet service crashes:
RestartSec=10
KillSignal=SIGINT
SyslogIdentifier=dotnet-example
User=www-data
Environment=ASPNETCORE_ENVIRONMENT=Production
Environment=DOTNET_PRINT_TELEMETRY_MESSAGE=false

[Install]
WantedBy=multi-user.target
```

Nell'esempio precedente, l'utente che gestisce il servizio viene specificato dall' `User` opzione. L'utente ( `www-data` ) deve esistere e avere la proprietà appropriata dei file dell'app.

Usare `TimeoutStopSec` per configurare il tempo di attesa prima che l'app si arresti dopo aver ricevuto il segnale di interrupt iniziale. Se l'app non si arresta in questo periodo, viene emesso il comando SIGKILL per terminare l'app. Specificare il valore in secondi senza unità di misura (ad esempio, `150`), un valore per l'intervallo di tempo (ad esempio, `2min 30s`) o `infinity` per disabilitare il timeout. `TimeoutStopSec` il valore predefinito è il valore di `DefaultTimeoutStopSec` nel file di configurazione Manager ( `systemd-system.conf` ,, `system.conf.d` `systemd-user.conf` , `user.conf.d` ). Il timeout predefinito per la maggior parte delle distribuzioni è di 90 secondi.

```
# The default value is 90 seconds for most distributions.
TimeoutStopSec=90
```

Linux dispone di un file system con distinzione tra maiuscole e minuscole. Se `ASPNETCORE_ENVIRONMENT` si imposta su `Production` , la ricerca del file di configurazione non viene eseguita `appsettings.Production.json` `appsettings.production.json` .

Per alcuni valori (ad esempio, le stringhe di connessione SQL) è necessario usare caratteri di escape, in modo da consentire ai provider di configurazione di leggere le variabili di ambiente. Usare il comando seguente per generare un valore con caratteri di escape corretti per l'uso nel file di configurazione:

```console
systemd-escape "<value-to-escape>"
```

::: moniker range=">= aspnetcore-3.0"

I due punti (`:`) separatori non sono supportati nei nomi delle variabili di ambiente. Usare un doppio carattere di sottolineatura (`__`) al posto dei due punti. Il [provider di configurazione delle variabili di ambiente](xref:fundamentals/configuration/index#environment-variables) converte le doppie sottolineature in due punti quando le variabili di ambiente vengono lette nella configurazione. Nell'esempio seguente la chiave della stringa di connessione `ConnectionStrings:DefaultConnection` è impostata nel file di definizione del servizio come `ConnectionStrings__DefaultConnection`:

::: moniker-end
::: moniker range="< aspnetcore-3.0"

I due punti (`:`) separatori non sono supportati nei nomi delle variabili di ambiente. Usare un doppio carattere di sottolineatura (`__`) al posto dei due punti. Il [provider di configurazione delle variabili di ambiente](xref:fundamentals/configuration/index#environment-variables-configuration-provider) converte le doppie sottolineature in due punti quando le variabili di ambiente vengono lette nella configurazione. Nell'esempio seguente la chiave della stringa di connessione `ConnectionStrings:DefaultConnection` è impostata nel file di definizione del servizio come `ConnectionStrings__DefaultConnection`:

::: moniker-end

```
Environment=ConnectionStrings__DefaultConnection={Connection String}
```

Salvare il file e abilitare il servizio.

```bash
sudo systemctl enable kestrel-helloapp.service
```

Avviare il servizio e verificare che sia in esecuzione.

```
sudo systemctl start kestrel-helloapp.service
sudo systemctl status kestrel-helloapp.service

◝ kestrel-helloapp.service - Example .NET Web API App running on Ubuntu
    Loaded: loaded (/etc/systemd/system/kestrel-helloapp.service; enabled)
    Active: active (running) since Thu 2016-10-18 04:09:35 NZDT; 35s ago
Main PID: 9021 (dotnet)
    CGroup: /system.slice/kestrel-helloapp.service
            └─9021 /usr/local/bin/dotnet /var/www/helloapp/helloapp.dll
```

Con il proxy inverso configurato e il gheppio gestito tramite `systemd` , l'app Web è completamente configurata ed è accessibile da un browser nel computer locale all'indirizzo `http://localhost` . È anche possibile accedervi da un computer remoto, escludendo eventuali firewall che potrebbero impedirlo. Se si esaminano le intestazioni di risposta, l'intestazione `Server` indica che l'app ASP.NET Core è gestita da Kestrel.

```text
HTTP/1.1 200 OK
Date: Tue, 11 Oct 2016 16:22:23 GMT
Server: Kestrel
Keep-Alive: timeout=5, max=98
Connection: Keep-Alive
Transfer-Encoding: chunked
```

### <a name="view-logs"></a>Visualizzare i log

Poiché l'app Web che usa Kestrel viene gestita con `systemd`, tutti gli eventi e i processi vengono registrati in un giornale di registrazione centralizzato. Tuttavia, questo giornale include tutte le voci per tutti i servizi e i processi gestiti da `systemd`. Per visualizzare le voci specifiche di `kestrel-helloapp.service`, usare il comando seguente:

```bash
sudo journalctl -fu kestrel-helloapp.service
```

Per un ulteriore filtro, le opzioni relative all'ora, ad esempio `--since today` , `--until 1 hour ago` o una combinazione di questi elementi possono ridurre il numero di voci restituite.

```bash
sudo journalctl -fu kestrel-helloapp.service --since "2016-10-18" --until "2016-10-18 04:00"
```

## <a name="data-protection"></a>Protezione dei dati

Lo [stack di protezione dei dati ASP.NET Core](xref:security/data-protection/introduction) viene usato da diversi [middleware](xref:fundamentals/middleware/index)di ASP.NET Core, tra cui il middleware di autenticazione (ad esempio, cookie middleware) e le protezioni di richiesta tra siti (CSRF). Anche se le DPAPI (Data Protection API) non vengono chiamate dal codice dell'utente, è consigliabile configurare la protezione dati per la creazione di un [archivio di chiavi](xref:security/data-protection/implementation/key-management) crittografiche permanente. Se non si configura la protezione dei dati, le chiavi vengono mantenute in memoria ed eliminate al riavvio dell'app.

Se il gruppo di chiavi viene archiviato in memoria quando l'app viene riavviata:

* Tutti i cookie token di autenticazione basati su vengono invalidati.
* Gli utenti devono ripetere l'accesso alla richiesta successiva.
* Tutti i dati protetti con il gruppo di chiavi non possono più essere decrittografati. Questo può includere [i token CSRF](xref:security/anti-request-forgery#aspnet-core-antiforgery-configuration) e [ASP.NET Core TempData MVC cookie ](xref:fundamentals/app-state#tempdata).

Per configurare la protezione dei dati in modo da rendere persistente il gruppo di chiavi e crittografarlo, vedere:

* <xref:security/data-protection/implementation/key-storage-providers>
* <xref:security/data-protection/implementation/key-encryption-at-rest>

## <a name="long-request-header-fields"></a>Campi di intestazione della richiesta di grandi dimensioni

Le impostazioni predefinite del server proxy limitano in genere i campi di intestazione della richiesta a 4 K o 8 K a seconda della piattaforma. Un'app può richiedere campi più lunghi del valore predefinito, ad esempio le app che usano [Azure Active Directory](https://azure.microsoft.com/services/active-directory/). Se sono necessari campi più lunghi, le impostazioni predefinite del server proxy richiedono la regolazione. I valori da applicare dipendono dallo scenario. Per altre informazioni, vedere la documentazione del server.

* [proxy_buffer_size](https://nginx.org/docs/http/ngx_http_proxy_module.html#proxy_buffer_size)
* [proxy_buffers](https://nginx.org/docs/http/ngx_http_proxy_module.html#proxy_buffers)
* [proxy_busy_buffers_size](https://nginx.org/docs/http/ngx_http_proxy_module.html#proxy_busy_buffers_size)
* [large_client_header_buffers](https://nginx.org/docs/http/ngx_http_core_module.html#large_client_header_buffers)

> [!WARNING]
> Aumentare i valori predefiniti dei buffer proxy solo se necessario. Se si aumentano questi valori, si aumenta il rischio di sovraccarico del buffer (overflow) e di attacchi Denial of Service (DoS) da parte di utenti malintenzionati.

## <a name="secure-the-app"></a>Proteggere l'app

### <a name="enable-apparmor"></a>Abilitare AppArmor

Linux Security Modules (LSM) è un framework che fa parte del kernel Linux a partire da Linux 2.6. LSM supporta diverse implementazioni di moduli di protezione. [AppArmor](https://wiki.ubuntu.com/AppArmor) è un LSM che implementa un sistema di controllo degli accessi obbligatorio, che consente di limitare il programma a un set limitato di risorse. Verificare che AppArmor sia abilitato e configurato correttamente.

### <a name="configure-the-firewall"></a>Configurare il firewall

Chiudere tutte le porte esterne che non sono in uso. Un firewall semplice (ufw) fornisce un front-end per `iptables` fornendo un'interfaccia della riga di comando per la configurazione del firewall.

> [!WARNING]
> Se non è configurato correttamente, un firewall impedisce l'accesso all'intero sistema. Se non si specifica la porta SSH corretta, non sarà possibile accedere al sistema se si usa SSH per la connessione. Il numero di porta predefinito è 22. Per altre informazioni, vedere l'[introduzione a ufw](https://help.ubuntu.com/community/UFW) e il [manuale](https://manpages.ubuntu.com/manpages/bionic/man8/ufw.8.html).

Installare `ufw` e configurarlo per consentire il traffico su tutte le porte necessarie.

```bash
sudo apt-get install ufw

sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

sudo ufw enable
```

### <a name="secure-nginx"></a>Proteggere Nginx

#### <a name="change-the-nginx-response-name"></a>Modificare il nome di risposta Nginx

Modificare `src/http/ngx_http_header_filter_module.c`:

```
static char ngx_http_server_string[] = "Server: Web Server" CRLF;
static char ngx_http_server_full_string[] = "Server: Web Server" CRLF;
```

#### <a name="configure-options"></a>Configurare le opzioni

Configurare il server con moduli aggiuntivi obbligatori. Può essere utile usare un firewall per app Web, ad esempio [ModSecurity](https://www.modsecurity.org/), per la protezione avanzata dell'app.

#### <a name="https-configuration"></a>Configurazione HTTPS

**Configurare l'app per connessioni locali sicure (HTTPS)**

Il comando [DotNet Run](/dotnet/core/tools/dotnet-run) usa le *Proprietà/launchSettings.js* dell'app nel file, che configura l'app in modo che sia in ascolto sugli URL forniti dalla `applicationUrl` Proprietà. Ad esempio: `https://localhost:5001;http://localhost:5000`.

Configurare l'app per l'uso di un certificato in fase di sviluppo per il `dotnet run` comando o l'ambiente di sviluppo (<kbd>F5</kbd> o <kbd>CTRL</kbd> + <kbd>F5</kbd> in Visual Studio Code) usando uno degli approcci seguenti:

::: moniker range=">= aspnetcore-5.0"

* [Sostituire il certificato predefinito della configurazione](xref:fundamentals/servers/kestrel/endpoints#configuration) (*consigliato*)
* [KestrelServerOptions.ConfigureHttpsDefaults](xref:fundamentals/servers/kestrel/endpoints#configurehttpsdefaultsactionhttpsconnectionadapteroptions)

::: moniker-end

::: moniker range="< aspnetcore-5.0"

* [Sostituire il certificato predefinito della configurazione](xref:fundamentals/servers/kestrel#configuration) (*consigliato*)
* [KestrelServerOptions.ConfigureHttpsDefaults](xref:fundamentals/servers/kestrel#configurehttpsdefaultsactionhttpsconnectionadapteroptions)

::: moniker-end

**Configurare il proxy inverso per connessioni client sicure (HTTPS)**

* Configurare il server per l'ascolto del traffico HTTPS sulla porta 443 specificando un certificato valido emesso da un'autorità di certificazione (CA) attendibile.

* Rafforzare la protezione adottando alcune delle procedure descritte nel file */etc/nginx/nginx.conf* che segue. Gli esempi includono la scelta di una crittografia più complessa e il reindirizzamento di tutto il traffico su HTTP a HTTPS.

  > [!NOTE]
  > Per gli ambienti di sviluppo, è consigliabile usare reindirizzamenti temporanei (302) invece dei reindirizzamenti permanenti (301). La memorizzazione nella cache dei collegamenti può causare un comportamento instabile negli ambienti di sviluppo.

* L'aggiunta di un'intestazione `HTTP Strict-Transport-Security` (HSTS) garantisce che tutte le richieste successive vengano effettuate dal client via HTTPS.

  Per informazioni importanti su HSTS, vedere <xref:security/enforcing-ssl#http-strict-transport-security-protocol-hsts> .

* Se HTTPS sarà disabilitato in futuro, usare uno degli approcci seguenti:

  * Non aggiungere l'intestazione HSTS.
  * Scegliere un `max-age` valore breve.

Aggiungere il file di configurazione */etc/nginx/proxy.conf*:

[!code-nginx[](linux-nginx/proxy.conf)]

**Sostituire** il contenuto del file di configurazione */etc/nginx/nginx.conf* con il file seguente. L'esempio contiene entrambe le sezioni `http` e `server` in un unico file di configurazione.

[!code-nginx[](linux-nginx/nginx.conf?highlight=2)]

> [!NOTE]
> Blazor WebAssembly le app richiedono un `burst` valore di parametro maggiore per contenere il numero maggiore di richieste effettuate da un'app. Per altre informazioni, vedere <xref:blazor/host-and-deploy/webassembly#nginx>.

#### <a name="secure-nginx-from-clickjacking"></a>Proteggere Nginx dal clickjacking

Il [clickjacking](https://blog.qualys.com/securitylabs/2015/10/20/clickjacking-a-common-implementation-mistake-that-can-put-your-websites-in-danger), anche noto come *attacco UI Redress*, è un attacco dannoso in cui un visitatore di un sito Web viene indotto a fare clic su un collegamento o un pulsante in una pagina diversa da quella che sta attualmente visualizzando. Usare `X-FRAME-OPTIONS` per proteggere il sito.

Per contrastare gli attacchi di clickjacking:

1. Modificare il file *nginx.conf*:

   ```bash
   sudo nano /etc/nginx/nginx.conf
   ```

   Aggiungere la riga: `add_header X-Frame-Options "SAMEORIGIN";`

1. Salvare il file.
1. Riavviare Nginx.

#### <a name="mime-type-sniffing"></a>Analisi del tipo MIME

Questa intestazione impedisce alla maggior parte dei browser di dedurre dall'analisi del tipo MIME un tipo di contenuto diverso da quello dichiarato, poiché l'intestazione indica al browser di non eseguire l'override del tipo di contenuto della risposta. Con l' `nosniff` opzione, se il server indica che il contenuto è `text/html` , il browser ne esegue il rendering come `text/html` .

1. Modificare il file *nginx.conf*:

   ```bash
   sudo nano /etc/nginx/nginx.conf
   ```

   Aggiungere la riga: `add_header X-Content-Type-Options "nosniff";`

1. Salvare il file.
1. Riavviare Nginx.

## <a name="additional-nginx-suggestions"></a>Suggerimenti nginx aggiuntivi

Dopo aver aggiornato il Framework condiviso sul server, riavviare le app ASP.NET Core ospitate dal server.

## <a name="additional-resources"></a>Risorse aggiuntive

* [Prerequisiti per .NET Core in Linux](/dotnet/core/linux-prerequisites)
* [Nginx: Binary Releases: Official Debian/Ubuntu packages](https://www.nginx.com/resources/wiki/start/topics/tutorials/install/#official-debian-ubuntu-packages) (Nginx: Versioni binarie: Pacchetti ufficiali Debian/Ubuntu)
* <xref:test/troubleshoot>
* <xref:host-and-deploy/proxy-load-balancer>
* [NGINX: Using the Forwarded header](https://www.nginx.com/resources/wiki/start/topics/examples/forwarded/) (NGINX: Uso dell'intestazione Forwarded)
