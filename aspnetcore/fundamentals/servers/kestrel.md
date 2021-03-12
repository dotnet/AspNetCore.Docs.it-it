---
title: Implementazione del server Web Kestrel in ASP.NET Core
author: rick-anderson
description: Informazioni su Kestrel, il server Web multipiattaforma per ASP.NET Core.
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 05/04/2020
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
uid: fundamentals/servers/kestrel
ms.openlocfilehash: 54d7304430c2a77347b6e3279f638cf6d7add77e
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102589043"
---
# <a name="kestrel-web-server-implementation-in-aspnet-core"></a>Implementazione del server Web Kestrel in ASP.NET Core

Di [Tom Dykstra](https://github.com/tdykstra), [Chris Ross](https://github.com/Tratcher) e [Stephen Halter](https://twitter.com/halter73)

::: moniker range=">= aspnetcore-5.0"

Kestrel è un [server Web per ASP.NET Core](xref:fundamentals/servers/index) multipiattaforma. Gheppio è il server Web incluso e abilitato per impostazione predefinita nei modelli di progetto ASP.NET Core.

Kestrel supporta gli scenari seguenti:

* HTTPS
* [Http/2](xref:fundamentals/servers/kestrel/http2) (eccetto MacOS &dagger; )
* Aggiornamento opaco usato per abilitare [WebSocket](xref:fundamentals/websockets)
* Socket Unix ad alte prestazioni dietro Nginx

&dagger;HTTP/2 verrà supportato in macOS in una versione futura.

Kestrel è supportato in tutte le piattaforme e le versioni supportate da .NET Core.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/servers/kestrel/samples/5.x) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="get-started"></a>Introduzione

I modelli di progetto ASP.NET Core usano Kestrel per impostazione predefinita. In *Program.cs* il <xref:Microsoft.Extensions.Hosting.GenericHostBuilderExtensions.ConfigureWebHostDefaults*> metodo chiama <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderKestrelExtensions.UseKestrel*> :

[!code-csharp[](kestrel/samples/5.x/KestrelSample/Program.cs?name=snippet_DefaultBuilder&highlight=8)]

Per ulteriori informazioni sulla compilazione dell'host, vedere le sezioni configurare le impostazioni di *un host* e di un *generatore predefinito* di <xref:fundamentals/host/generic-host#set-up-a-host> .

## <a name="additional-resources"></a>Risorse aggiuntive

<a name="endpoint-configuration"></a>
* <xref:fundamentals/servers/kestrel/endpoints>
<a name="kestrel-options"></a>
* <xref:fundamentals/servers/kestrel/options>
<a name="http2-support"></a>
* <xref:fundamentals/servers/kestrel/http2>
<a name="when-to-use-kestrel-with-a-reverse-proxy"></a>
* <xref:fundamentals/servers/kestrel/when-to-use-a-reverse-proxy>
<a name="host-filtering"></a>
* <xref:fundamentals/servers/kestrel/host-filtering>
* <xref:test/troubleshoot>
* <xref:security/enforcing-ssl>
* <xref:host-and-deploy/proxy-load-balancer>
* [RFC 7230: Message Syntax and Routing (Section 5.4: Host)](https://tools.ietf.org/html/rfc7230#section-5.4) (RFC 7230: Sintassi e routing dei messaggi (sezione 5.4: Host))
* Quando si usano i socket UNIX in Linux, il socket non viene eliminato automaticamente all'arresto dell'app. Per altre informazioni, vedere [questo problema di GitHub](https://github.com/dotnet/aspnetcore/issues/14134).

> [!NOTE]
> A partire da ASP.NET Core 5,0, il trasporto libuv del gheppio è obsoleto. Il trasporto libuv non riceve gli aggiornamenti per supportare nuove piattaforme del sistema operativo, ad esempio Windows ARM64, e verrà rimosso in una versione futura. Rimuovere tutte le chiamate al metodo obsoleto <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderLibuvExtensions.UseLibuv%2A> e usare invece il trasporto socket predefinito di gheppio.

::: moniker-end

::: moniker range=">= aspnetcore-3.0 < aspnetcore-5.0"

Kestrel è un [server Web per ASP.NET Core](xref:fundamentals/servers/index) multipiattaforma. Kestrel è il server Web incluso per impostazione predefinita nei modelli di progetto di ASP.NET Core.

Kestrel supporta gli scenari seguenti:

* HTTPS
* Aggiornamento opaco usato per abilitare [WebSocket](https://github.com/aspnet/websockets)
* Socket Unix ad alte prestazioni dietro Nginx
* HTTP/2 (tranne che in macOS&dagger;)

&dagger;HTTP/2 verrà supportato in macOS in una versione futura.

Kestrel è supportato in tutte le piattaforme e le versioni supportate da .NET Core.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/servers/kestrel/samples/3.x) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="http2-support"></a>Supporto HTTP/2

[HTTP/2](https://httpwg.org/specs/rfc7540.html) è disponibile per le app ASP.NET Core se vengono soddisfatti i requisiti di base seguenti:

* Sistema operativo&dagger;
  * Windows Server 2016/Windows 10 o versioni successive&Dagger;
  * Linux con OpenSSL 1.0.2 o versioni successive (ad esempio, Ubuntu 16.04 o versioni successive)
* Framework di destinazione: .NET Core 2.2 o versioni successive
* Connessione [ALPN (Application-Layer Protocol Negotiation)](https://tools.ietf.org/html/rfc7301#section-3)
* Connessione TLS 1.2 o successiva

&dagger;HTTP/2 verrà supportato in macOS in una versione futura.
&Dagger;Kestrel ha un supporto limitato per HTTP/2 in Windows Server 2012 R2 e Windows 8.1. Il supporto è limitato perché l'elenco di suite di crittografia TLS supportate disponibili in questi sistemi operativi è limitato. Un certificato generato con un algoritmo ECDSA potrebbe essere necessario per proteggere le connessioni TLS.

Se viene stabilita una connessione HTTP/2, [HttpRequest.Protocol](xref:Microsoft.AspNetCore.Http.HttpRequest.Protocol*) corrisponde a `HTTP/2`.

A partire da .NET Core 3,0, HTTP/2 è abilitato per impostazione predefinita. Per altre informazioni sulla configurazione, vedere le sezioni [Opzioni Kestrel](#kestrel-options) e [ListenOptions.Protocols](#listenoptionsprotocols).

## <a name="when-to-use-kestrel-with-a-reverse-proxy"></a>Quando usare Kestrel con un proxy inverso

È possibile usare Kestrel da solo o in combinazione con un *server proxy inverso*, ad esempio [Internet Information Services (IIS)](https://www.iis.net/), [Nginx](https://nginx.org) o [Apache](https://httpd.apache.org/). Il server proxy inverso riceve le richieste HTTP dalla rete e le inoltra a Kestrel.

Kestrel usato come server Web perimetrale (esposto a Internet):

![Kestrel comunica direttamente con Internet senza un server proxy inverso](kestrel/_static/kestrel-to-internet2.png)

Kestrel usato in una configurazione proxy inverso:

![Kestrel comunica indirettamente con Internet attraverso un server proxy inverso, ad esempio IIS, Nginx o Apache](kestrel/_static/kestrel-to-internet.png)

La configurazione, con o senza un server proxy inverso, è una configurazione di hosting supportata.

Se viene utilizzato come server perimetrale in assenza di un server proxy inverso, Kestrel non supporta la condivisione tra più processi dell'IP e della porta. Quando è configurato per restare in ascolto su una porta, Kestrel gestisce tutto il traffico per tale porta indipendentemente dalle intestazioni `Host` delle richieste. Un proxy inverso che può condividere le porte può eseguire l'inoltro delle richieste a Kestrel su un unico IP e un'unica porta.

Anche se la presenza di un server proxy inverso non è necessaria, può risultare utile per i motivi seguenti.

Un proxy inverso:

* Può limitare l'area della superficie di attacco pubblica esposta delle app ospitate.
* Offre un livello aggiuntivo di configurazione e protezione.
* Potrebbe offrire un'integrazione migliore con l'infrastruttura esistente.
* Semplifica il bilanciamento del carico e la configurazione di comunicazioni protette (HTTPS). Solo il server proxy inverso richiede un certificato X. 509 e il server è in grado di comunicare con i server dell'app nella rete interna tramite HTTP normale.

> [!WARNING]
> Una configurazione che prevede un proxy inverso richiede il [filtro host](#host-filtering).

## <a name="kestrel-in-aspnet-core-apps"></a>Gheppio nelle app ASP.NET Core

I modelli di progetto ASP.NET Core usano Kestrel per impostazione predefinita. In *Program.cs* il <xref:Microsoft.Extensions.Hosting.GenericHostBuilderExtensions.ConfigureWebHostDefaults*> metodo chiama <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderKestrelExtensions.UseKestrel*> :

[!code-csharp[](kestrel/samples/3.x/KestrelSample/Program.cs?name=snippet_DefaultBuilder&highlight=8)]

Per ulteriori informazioni sulla compilazione dell'host, vedere le sezioni configurare le impostazioni di *un host* e di un *generatore predefinito* di <xref:fundamentals/host/generic-host#set-up-a-host> .

Per fornire una configurazione aggiuntiva dopo la chiamata di `ConfigureWebHostDefaults`, usare `ConfigureKestrel`:

```csharp
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.ConfigureKestrel(serverOptions =>
            {
                // Set properties and call methods on options
            })
            .UseStartup<Startup>();
        });
```

## <a name="kestrel-options"></a>Opzioni Kestrel

Il server Web Kestrel dispone di opzioni di configurazione dei vincoli che risultano particolarmente utili nelle distribuzioni con connessione Internet.

Impostare i vincoli per la proprietà <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Limits> della classe <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions>. La proprietà `Limits` contiene un'istanza della classe <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits>.

Negli esempi seguenti viene usato lo spazio dei nomi <xref:Microsoft.AspNetCore.Server.Kestrel.Core>:

```csharp
using Microsoft.AspNetCore.Server.Kestrel.Core;
```

Negli esempi illustrati più avanti in questo articolo, le opzioni di gheppio sono configurate nel codice C#. È inoltre possibile impostare le opzioni di Gheppio utilizzando un [provider di configurazione](xref:fundamentals/configuration/index). Il [provider di configurazione file](xref:fundamentals/configuration/index#file-configuration-provider) , ad esempio, può caricare la configurazione di Gheppio da un oggetto *appsettings.json* o *appSettings. { File Environment}. JSON* :

```json
{
  "Kestrel": {
    "Limits": {
      "MaxConcurrentConnections": 100,
      "MaxConcurrentUpgradedConnections": 100
    },
    "DisableStringReuse": true
  }
}
```

> [!NOTE]
> <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions> e la [configurazione dell'endpoint](#endpoint-configuration) sono configurabili dai provider di configurazione. La configurazione del gheppio rimanente deve essere configurata nel codice C#.

Usare **uno** degli approcci seguenti:

* Configurare gheppio in `Startup.ConfigureServices` :

  1. Inserire un'istanza di `IConfiguration` nella `Startup` classe. Nell'esempio seguente si presuppone che la configurazione inserita venga assegnata alla `Configuration` Proprietà.
  2. In `Startup.ConfigureServices` caricare la `Kestrel` sezione della configurazione nella configurazione di Gheppio:

     ```csharp
     using Microsoft.Extensions.Configuration
     
     public class Startup
     {
         public Startup(IConfiguration configuration)
         {
             Configuration = configuration;
         }

         public IConfiguration Configuration { get; }

         public void ConfigureServices(IServiceCollection services)
         {
             services.Configure<KestrelServerOptions>(
                 Configuration.GetSection("Kestrel"));
         }

         public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
         {
             ...
         }
     }
     ```

* Configurare il gheppio durante la compilazione dell'host:

  In *Program.cs* caricare la `Kestrel` sezione della configurazione nella configurazione di Gheppio:

  ```csharp
  // using Microsoft.Extensions.DependencyInjection;

  public static IHostBuilder CreateHostBuilder(string[] args) =>
      Host.CreateDefaultBuilder(args)
          .ConfigureServices((context, services) =>
          {
              services.Configure<KestrelServerOptions>(
                  context.Configuration.GetSection("Kestrel"));
          })
          .ConfigureWebHostDefaults(webBuilder =>
          {
              webBuilder.UseStartup<Startup>();
          });
  ```

Entrambi gli approcci precedenti funzionano con qualsiasi [provider di configurazione](xref:fundamentals/configuration/index).

### <a name="keep-alive-timeout"></a>Timeout keep-alive

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.KeepAliveTimeout>

Ottiene o imposta il [timeout keep-alive](https://tools.ietf.org/html/rfc7230#section-6.5). Il valore predefinito è 2 minuti.

[!code-csharp[](kestrel/samples/3.x/KestrelSample/Program.cs?name=snippet_Limits&highlight=19-20)]

### <a name="maximum-client-connections"></a>Numero massimo di connessioni client

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.MaxConcurrentConnections>
<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.MaxConcurrentUpgradedConnections>

È possibile impostare il numero massimo di connessioni TCP aperte simultaneamente per l'intera app con il codice seguente:

[!code-csharp[](kestrel/samples/3.x/KestrelSample/Program.cs?name=snippet_Limits&highlight=3)]

È previsto un limite separato per le connessioni che sono state aggiornate da HTTP o HTTPS a un altro protocollo (ad esempio su una richiesta WebSocket). Dopo l'aggiornamento di una connessione, questa non viene conteggiata per il limite `MaxConcurrentConnections`.

[!code-csharp[](kestrel/samples/3.x/KestrelSample/Program.cs?name=snippet_Limits&highlight=4)]

Per impostazione predefinita, il numero massimo di connessioni è illimitato (null).

### <a name="maximum-request-body-size"></a>Dimensione massima del corpo della richiesta

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.MaxRequestBodySize>

La dimensione massima predefinita del corpo della richiesta è pari a 30.000.000 di byte, equivalenti a circa 28,6 MB.

Il metodo consigliato per ignorare il limite in un'app ASP.NET Core MVC è l'uso dell'attributo <xref:Microsoft.AspNetCore.Mvc.RequestSizeLimitAttribute> in un metodo di azione:

```csharp
[RequestSizeLimit(100000000)]
public IActionResult MyActionMethod()
```

L'esempio seguente illustra come configurare il vincolo per l'app in ogni richiesta:

[!code-csharp[](kestrel/samples/3.x/KestrelSample/Program.cs?name=snippet_Limits&highlight=5)]

Eseguire l'override dell'impostazione per una richiesta specifica nel middleware:

[!code-csharp[](kestrel/samples/3.x/KestrelSample/Startup.cs?name=snippet_Limits&highlight=3-4)]

Viene generata un'eccezione se l'app configura il limite per una richiesta dopo che l'app ha iniziato a leggere la richiesta. Una proprietà `IsReadOnly` indica se la proprietà `MaxRequestBodySize` è in stato di sola lettura e pertanto è troppo tardi per configurare il limite.

Quando un'app viene eseguita [out-of-process](xref:host-and-deploy/iis/index#out-of-process-hosting-model) dietro il [modulo ASP.NET Core](xref:host-and-deploy/aspnet-core-module), il limite della dimensione del corpo della richiesta di Kestrel è disabilitato perché viene già impostato da IIS.

### <a name="minimum-request-body-data-rate"></a>Velocità minima dei dati del corpo della richiesta

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.MinRequestBodyDataRate>
<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.MinResponseDataRate>

Kestrel controlla ogni secondo se i dati arrivano alla velocità in byte al secondo specificata. Se la frequenza scende sotto il valore minimo, si è verificato il timeout della connessione. Il periodo di tolleranza è la quantità di tempo che il gheppio concede al client di aumentare la velocità di invio fino al minimo. la frequenza non viene controllata durante tale periodo di tempo. Il periodo di prova consente di evitare l'interruzione di connessioni che inizialmente inviano i dati a velocità ridotta a causa dell'avvio lento del protocollo TCP.

La velocità minima predefinita è di 240 byte al secondo, con un periodo di tolleranza di 5 secondi.

Anche per la risposta è prevista una velocità minima. Il codice per impostare il limite della richiesta e della risposta è identico e varia solo per `RequestBody` o `Response` nei nomi di proprietà e di interfaccia.

L'esempio seguente visualizza come configurare la velocità minima dei dati in *Program.cs*:

[!code-csharp[](kestrel/samples/3.x/KestrelSample/Program.cs?name=snippet_Limits&highlight=6-11)]

Ignorare i limiti di velocità minima per ogni richiesta nel middleware:

[!code-csharp[](kestrel/samples/3.x/KestrelSample/Startup.cs?name=snippet_Limits&highlight=6-21)]

La funzionalità <xref:Microsoft.AspNetCore.Server.Kestrel.Core.Features.IHttpMinResponseDataRateFeature> a cui si fa riferimento nell'esempio precedente non è presente in `HttpContext.Features` per le richieste HTTP/2, perché la modifica dei limiti di velocità per ogni richiesta in genere non è supportata per HTTP/2 a causa del supporto del protocollo per il multiplexing delle richieste. La funzionalità <xref:Microsoft.AspNetCore.Server.Kestrel.Core.Features.IHttpMinRequestBodyDataRateFeature> è tuttavia ancora presente in `HttpContext.Features` per le richieste HTTP/2, perché il limite di velocità di lettura può comunque essere *disabilitato interamente* per ogni singola richiesta impostando `IHttpMinRequestBodyDataRateFeature.MinDataRate` su `null` anche per una richiesta HTTP/2. Il tentativo di leggere `IHttpMinRequestBodyDataRateFeature.MinDataRate` o il tentativo di impostazione di un valore diverso da `null` causerà la generazione di una `NotSupportedException` per una richiesta HTTP/2.

I limiti di velocità a livello di server configurati tramite `KestrelServerOptions.Limits` vengono tuttavia applicati alle connessioni HTTP/1.x e HTTP/2.

### <a name="request-headers-timeout"></a>Timeout delle intestazioni delle richieste

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.RequestHeadersTimeout>

Ottiene o imposta la quantità massima di tempo che il server dedica alla ricezione delle intestazioni delle richieste. Il valore predefinito è 30 secondi.

[!code-csharp[](kestrel/samples/3.x/KestrelSample/Program.cs?name=snippet_Limits&highlight=21-22)]

### <a name="maximum-streams-per-connection"></a>Numero massimo di flussi per connessione

`Http2.MaxStreamsPerConnection` limita il numero di flussi di richieste simultanee per ogni connessione HTTP/2. I flussi in eccesso vengono rifiutati.

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.Limits.Http2.MaxStreamsPerConnection = 100;
});
```

Il valore predefinito è 100.

### <a name="header-table-size"></a>Dimensioni della tabella delle intestazioni

Il decodificatore HPACK decomprime le intestazioni HTTP per le connessioni HTTP/2. `Http2.HeaderTableSize` limita le dimensioni della tabella di compressione delle intestazioni usata dal decodificatore HPACK. Il valore viene specificato in ottetti e deve essere maggiore di zero (0).

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.Limits.Http2.HeaderTableSize = 4096;
});
```

Il valore predefinito è 4096.

### <a name="maximum-frame-size"></a>Dimensione massima del frame

`Http2.MaxFrameSize` indica la dimensione massima consentita di un payload del frame di connessione HTTP/2 ricevuto o inviato dal server. Il valore viene specificato in ottetti e deve essere compreso tra 2^14 (16.384) e 2^24-1 (16.777.215).

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.Limits.Http2.MaxFrameSize = 16384;
});
```

Il valore predefinito è 2^14 (16.384).

### <a name="maximum-request-header-size"></a>Dimensioni massime dell'intestazione della richiesta

`Http2.MaxRequestHeaderFieldSize` indica le dimensioni massime consentite in ottetti di valori di intestazione di richiesta. Questo limite si applica sia al nome che al valore nelle rappresentazioni compresse e non compresse. Il valore deve essere maggiore di zero (0).

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.Limits.Http2.MaxRequestHeaderFieldSize = 8192;
});
```

Il valore predefinito è 8.192.

### <a name="initial-connection-window-size"></a>Dimensioni della finestra di connessione iniziali

`Http2.InitialConnectionWindowSize` indica il numero massimo di byte di dati del corpo della richiesta che il server memorizza nel buffer in una volta, aggregati in tutte le richieste (flussi), per ogni connessione. Le richieste sono anche limitate da `Http2.InitialStreamWindowSize`. Il valore deve essere maggiore di o uguale a 65.535 e minore di 2^31 (2.147.483.648).

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.Limits.Http2.InitialConnectionWindowSize = 131072;
});
```

Il valore predefinito è 128 KB (131.072).

### <a name="initial-stream-window-size"></a>Dimensioni della finestra di flusso iniziali

`Http2.InitialStreamWindowSize` indica il numero massimo di byte di dati del corpo della richiesta che il server memorizza nel buffer in una volta per ogni richiesta (flusso). Le richieste sono anche limitate da `Http2.InitialConnectionWindowSize`. Il valore deve essere maggiore di o uguale a 65.535 e minore di 2^31 (2.147.483.648).

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.Limits.Http2.InitialStreamWindowSize = 98304;
});
```

Il valore predefinito è 96 KB (98.304).

### <a name="trailers"></a>Trailer

[!INCLUDE[](~/includes/trailers.md)]

### <a name="reset"></a>Reset

[!INCLUDE[](~/includes/reset.md)]

### <a name="synchronous-io"></a>I/O sincrono

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.AllowSynchronousIO> Controlla se è consentito l'I/O sincrono per la richiesta e la risposta. Il valore predefinito è `false`.

> [!WARNING]
> Un numero elevato di operazioni di I/O sincrone bloccanti può causare l'esaurimento del pool di thread, il che rende l'app non risponde. Abilitare solo `AllowSynchronousIO` quando si usa una libreria che non supporta l'I/O asincrono.

L'esempio seguente abilita l'I/O sincrono:

[!code-csharp[](kestrel/samples/3.x/KestrelSample/Program.cs?name=snippet_SyncIO)]

Per informazioni su altre opzioni e limiti di Kestrel, vedere:

* <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions>
* <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits>
* <xref:Microsoft.AspNetCore.Server.Kestrel.Core.ListenOptions>

## <a name="endpoint-configuration"></a>Configurazione dell'endpoint

Per impostazione predefinita, ASP.NET Core è associato a:

* `http://localhost:5000`
* `https://localhost:5001` (quando è presente un certificato di sviluppo locale)

Specificare gli URL usando gli elementi seguenti:

* La variabile di ambiente `ASPNETCORE_URLS`.
* L'argomento della riga di comando `--urls`.
* La chiave di configurazione dell'host `urls`.
* Il metodo di estensione `UseUrls`.

Il valore specificato usando i metodi seguenti può essere uno o più endpoint HTTP e HTTPS (HTTPS se è disponibile un certificato predefinito). Configurare il valore come un elenco delimitato da punto e virgola (ad esempio, `"Urls": "http://localhost:8000;http://localhost:8001"`).

Per altre informazioni su questi approcci, vedere [URL del server](xref:fundamentals/host/web-host#server-urls) e [Override della configurazione](xref:fundamentals/host/web-host#override-configuration).

Viene creato un certificato di sviluppo nei casi seguenti:

* Quando viene installato [.NET Core SDK](/dotnet/core/sdk).
* Quando per creare un certificato viene usato [lo strumento dev-certs](xref:aspnetcore-2.1#https).

Alcuni browser richiedono la concessione di autorizzazioni esplicite per considerare attendibile il certificato di sviluppo locale.

I modelli di progetto consentono di configurare le app per l'esecuzione su HTTPS per impostazione predefinita e includono il [reindirizzamento HTTPS e il supporto HSTS](xref:security/enforcing-ssl).

Chiamare i metodi <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Listen*> oppure <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ListenUnixSocket*> su <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions> per configurare le porte e i prefissi URL per Kestrel.

Anche `UseUrls`, l'argomento della riga di comando `--urls`, la chiave di configurazione dell'host `urls` e la variabile di ambiente `ASPNETCORE_URLS` funzionano, ma con le limitazioni indicate più avanti nella sezione (deve essere disponibile un certificato predefinito per la configurazione dell'endopoint HTTPS).

`KestrelServerOptions` configurazione

### <a name="configureendpointdefaultsactionlistenoptions"></a>ConfigureEndpointDefaults (azione \<ListenOptions> )

Specifica un elemento di configurazione `Action` da eseguire per ogni endpoint specificato. Se si chiama `ConfigureEndpointDefaults` più volte, gli elementi `Action` precedenti vengono sostituiti con l'ultimo elemento `Action` specificato.

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.ConfigureEndpointDefaults(listenOptions =>
    {
        // Configure endpoint defaults
    });
});
```

> [!NOTE]
> Per gli endpoint creati chiamando <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Listen*> **prima** della chiamata a <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ConfigureEndpointDefaults*> non verranno applicati i valori predefiniti.

### <a name="configurehttpsdefaultsactionhttpsconnectionadapteroptions"></a>ConfigureHttpsDefaults (azione \<HttpsConnectionAdapterOptions> )

Specifica un elemento di configurazione `Action` da eseguire per ogni endpoint HTTPS. Se si chiama `ConfigureHttpsDefaults` più volte, gli elementi `Action` precedenti vengono sostituiti con l'ultimo elemento `Action` specificato.

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.ConfigureHttpsDefaults(listenOptions =>
    {
        // certificate is an X509Certificate2
        listenOptions.ServerCertificate = certificate;
    });
});
```

> [!NOTE]
> Per gli endpoint creati chiamando <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Listen*> **prima** della chiamata a <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ConfigureHttpsDefaults*> non verranno applicati i valori predefiniti.

### <a name="configureiconfiguration"></a>Configure(IConfiguration)

Crea un loader di configurazione per la configurazione di Kestrel che accetta un elemento <xref:Microsoft.Extensions.Configuration.IConfiguration> come input. L'ambito della configurazione deve essere la sezione di configurazione per Kestrel.

### <a name="listenoptionsusehttps"></a>ListenOptions.UseHttps

Configura Kestrel per l'uso di HTTPS.

Estensioni `ListenOptions.UseHttps`:

* `UseHttps`: Configurare gheppio per l'uso di HTTPS con il certificato predefinito. Genera un'eccezione se non è stato configurato alcun certificato predefinito.
* `UseHttps(string fileName)`
* `UseHttps(string fileName, string password)`
* `UseHttps(string fileName, string password, Action<HttpsConnectionAdapterOptions> configureOptions)`
* `UseHttps(StoreName storeName, string subject)`
* `UseHttps(StoreName storeName, string subject, bool allowInvalid)`
* `UseHttps(StoreName storeName, string subject, bool allowInvalid, StoreLocation location)`
* `UseHttps(StoreName storeName, string subject, bool allowInvalid, StoreLocation location, Action<HttpsConnectionAdapterOptions> configureOptions)`
* `UseHttps(X509Certificate2 serverCertificate)`
* `UseHttps(X509Certificate2 serverCertificate, Action<HttpsConnectionAdapterOptions> configureOptions)`
* `UseHttps(Action<HttpsConnectionAdapterOptions> configureOptions)`

Parametri `ListenOptions.UseHttps`:

* `filename` è il percorso e il nome file di un file di certificato, relativo alla directory che contiene i file di contenuto dell'app.
* `password` è la password richiesta per accedere ai dati del certificato X.509.
* `configureOptions` è un elemento `Action` per configurare `HttpsConnectionAdapterOptions`. Restituisce `ListenOptions`.
* `storeName` è l'archivio certificati da cui caricare il certificato.
* `subject` è il nome dell'oggetto del certificato.
* `allowInvalid` indica se devono essere considerati i certificati non validi, come ad esempio i certificati autofirmati.
* `location` è il percorso dell'archivio da cui caricare il certificato.
* `serverCertificate` è il certificato X.509.

Nell'ambiente di produzione, HTTPS deve essere configurato in modo esplicito. Come minimo, è necessario specificare un certificato predefinito.

Configurazioni supportate descritte in seguito:

* Nessuna configurazione
* Sostituire il certificato predefinito della configurazione
* Modificare le impostazioni predefinite nel codice

*Nessuna configurazione*

Kestrel è in ascolto su `http://localhost:5000` e `https://localhost:5001` (se è disponibile un certificato predefinito).

<a name="configuration"></a>

*Sostituire il certificato predefinito della configurazione*

`CreateDefaultBuilder` chiama `Configure(context.Configuration.GetSection("Kestrel"))` per impostazione predefinita per caricare la configurazione di Kestrel. È disponibile per Kestrel uno schema di configurazione delle impostazioni delle app HTTPS predefinito. Configurare più endpoint, inclusi gli URL e i certificati da usare, da un file su disco o da un archivio certificati.

Nell'esempio seguente *appsettings.json* :

* Impostare **AllowInvalid** su `true` per consentire l'uso di certificati non validi, come ad esempio i certificati autofirmati.
* Tutti gli endpoint HTTPS che non specificano un certificato (**HttpsDefaultCert** nell'esempio che segue) usano il certificato definito in **Certificates** > **Default** o il certificato di sviluppo.

```json
{
  "Kestrel": {
    "Endpoints": {
      "Http": {
        "Url": "http://localhost:5000"
      },
      "HttpsInlineCertFile": {
        "Url": "https://localhost:5001",
        "Certificate": {
          "Path": "<path to .pfx file>",
          "Password": "<certificate password>"
        }
      },
      "HttpsInlineCertStore": {
        "Url": "https://localhost:5002",
        "Certificate": {
          "Subject": "<subject; required>",
          "Store": "<certificate store; required>",
          "Location": "<location; defaults to CurrentUser>",
          "AllowInvalid": "<true or false; defaults to false>"
        }
      },
      "HttpsDefaultCert": {
        "Url": "https://localhost:5003"
      },
      "Https": {
        "Url": "https://*:5004",
        "Certificate": {
          "Path": "<path to .pfx file>",
          "Password": "<certificate password>"
        }
      }
    },
    "Certificates": {
      "Default": {
        "Path": "<path to .pfx file>",
        "Password": "<certificate password>"
      }
    }
  }
}
```

Un'alternativa all'uso di **Path** e **Password** per qualsiasi nodo del certificato consiste nello specificare il certificato usando i campi dell'archivio certificati. Ad esempio, il certificato predefinito **Certificates**  >   può essere specificato come:

```json
"Default": {
  "Subject": "<subject; required>",
  "Store": "<cert store; required>",
  "Location": "<location; defaults to CurrentUser>",
  "AllowInvalid": "<true or false; defaults to false>"
}
```

Note di schema:

* I nomi degli endpoint non applicano la distinzione tra maiuscole e minuscole. Ad esempio, sono validi sia `HTTPS` che `Https`.
* Il parametro `Url` è obbligatorio per ogni endpoint. Il formato per questo parametro è uguale a quello del parametro di configurazione di primo livello `Urls`, con la differenza che si limita a un singolo valore.
* Questi endpoint sostituiscono quelli definiti nella configurazione di primo livello `Urls`, non vi si aggiungono. Gli endpoint definiti nel codice tramite `Listen` si aggiungono agli endpoint definiti nella sezione di configurazione.
* La sezione `Certificate` è facoltativa. Se la sezione `Certificate` non è specificata, vengono usati i valori predefiniti definiti negli scenari precedenti. Se non è disponibile alcun valore predefinito, il server genera un'eccezione e non viene avviato.
* La `Certificate` sezione supporta sia la password del **percorso** &ndash;  che i certificati dell'archivio dell' **oggetto** &ndash;  .
* In questo modo è possibile definire un numero qualsiasi di endpoint, purché non provochino conflitti di porte.
* `options.Configure(context.Configuration.GetSection("{SECTION}"))` restituisce un oggetto `KestrelConfigurationLoader` con un metodo `.Endpoint(string name, listenOptions => { })` che può essere usato per integrare le impostazioni dell'endpoint configurato:

```csharp
webBuilder.UseKestrel((context, serverOptions) =>
{
    serverOptions.Configure(context.Configuration.GetSection("Kestrel"))
        .Endpoint("HTTPS", listenOptions =>
        {
            listenOptions.HttpsOptions.SslProtocols = SslProtocols.Tls12;
        });
});
```

`KestrelServerOptions.ConfigurationLoader` è possibile accedere direttamente a per continuare a scorrere sul caricatore esistente, ad esempio quello fornito da <xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder*> .

* La sezione di configurazione per ogni endpoint è disponibile nelle opzioni del `Endpoint` metodo in modo che sia possibile leggere le impostazioni personalizzate.
* È possibile caricare più configurazioni chiamando ancora `options.Configure(context.Configuration.GetSection("{SECTION}"))` con un'altra sezione. Viene usata solo l'ultima configurazione, a meno che `Load` sia stato chiamato esplicitamente in istanze precedenti. Il metapacchetto non chiama `Load` in modo che la sezione di configurazione predefinita venga sostituita.
* `KestrelConfigurationLoader` riflette la famiglia di API `Listen` da `KestrelServerOptions` all'overload di `Endpoint`, in modo che gli endpoint di codice e di configurazione possano essere configurati nella stessa posizione. Questi overload non usano nomi e usano solo le impostazioni predefinite della configurazione.

*Modificare le impostazioni predefinite nel codice*

`ConfigureEndpointDefaults` e `ConfigureHttpsDefaults` possono essere usati per modificare le impostazioni predefinite per `ListenOptions` e `HttpsConnectionAdapterOptions`, inclusa l'esecuzione dell'override del certificato predefinito specificato nello scenario precedente. `ConfigureEndpointDefaults` e `ConfigureHttpsDefaults` devono essere chiamati prima che venga configurato qualsiasi endpoint.

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.ConfigureEndpointDefaults(listenOptions =>
    {
        // Configure endpoint defaults
    });

    serverOptions.ConfigureHttpsDefaults(listenOptions =>
    {
        listenOptions.SslProtocols = SslProtocols.Tls12;
    });
});
```

*Supporto kestrel per SNI*

[L'Indicazione nome server (SNI)](https://tools.ietf.org/html/rfc6066#section-3) può essere usata per ospitare più domini sulla stessa porta e indirizzo IP. Perché SNI funzioni è necessario che il client invii al server il nome host per la sessione protetta durante l'handshake TLS in modo che il server possa specificare il certificato corretto. Il client usa il certificato specificato per la comunicazione crittografata con il server durante la sessione protetta che segue l'handshake TLS.

Kestrel supporta SNI tramite il callback `ServerCertificateSelector`. Il callback viene richiamato una volta per ogni connessione per consentire all'app di controllare il nome host e selezionare il certificato appropriato.

Il supporto SNI richiede:

* In esecuzione su Framework di destinazione `netcoreapp2.1` o versione successiva. In `net461` o versioni successive, il callback viene richiamato, ma `name` è sempre `null` . L'elemento `name` è `null` anche se il client non specifica il parametro del nome host nell'handshake TLS.
* Esecuzione di tutti i siti Web nella stessa istanza di Kestrel. Kestrel supporta la condivisione di un indirizzo IP e di una porta tra più istanze solo con un proxy inverso.

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.ListenAnyIP(5005, listenOptions =>
    {
        listenOptions.UseHttps(httpsOptions =>
        {
            var localhostCert = CertificateLoader.LoadFromStoreCert(
                "localhost", "My", StoreLocation.CurrentUser,
                allowInvalid: true);
            var exampleCert = CertificateLoader.LoadFromStoreCert(
                "example.com", "My", StoreLocation.CurrentUser,
                allowInvalid: true);
            var subExampleCert = CertificateLoader.LoadFromStoreCert(
                "sub.example.com", "My", StoreLocation.CurrentUser,
                allowInvalid: true);
            var certs = new Dictionary<string, X509Certificate2>(
                StringComparer.OrdinalIgnoreCase);
            certs["localhost"] = localhostCert;
            certs["example.com"] = exampleCert;
            certs["sub.example.com"] = subExampleCert;

            httpsOptions.ServerCertificateSelector = (connectionContext, name) =>
            {
                if (name != null && certs.TryGetValue(name, out var cert))
                {
                    return cert;
                }

                return exampleCert;
            };
        });
    });
});
```

### <a name="connection-logging"></a>Registrazione connessione

Chiamare <xref:Microsoft.AspNetCore.Hosting.ListenOptionsConnectionLoggingExtensions.UseConnectionLogging*> per creare log a livello di debug per la comunicazione a livello di byte in una connessione. La registrazione delle connessioni è utile per la risoluzione dei problemi nella comunicazione di basso livello, ad esempio durante la crittografia TLS e dietro i proxy. Se `UseConnectionLogging` viene inserito prima `UseHttps` , viene registrato il traffico crittografato. Se `UseConnectionLogging` viene inserito dopo `UseHttps` , viene registrato il traffico decrittografato.

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.Listen(IPAddress.Any, 8000, listenOptions =>
    {
        listenOptions.UseConnectionLogging();
    });
});
```

### <a name="bind-to-a-tcp-socket"></a>Associazione a un socket TCP

Il metodo <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Listen*> esegue l'associazione a un socket TCP e un'espressione lambda per le opzioni consente di configurare un certificato X.509:

[!code-csharp[](kestrel/samples/3.x/KestrelSample/Program.cs?name=snippet_TCPSocket&highlight=12-18)]

L'esempio configura HTTPS per un endpoint con <xref:Microsoft.AspNetCore.Server.Kestrel.Core.ListenOptions>. Usare la stessa API per configurare altre impostazioni di Kestrel per endpoint specifici.

[!INCLUDE [How to make an X.509 cert](~/includes/make-x509-cert.md)]

### <a name="bind-to-a-unix-socket"></a>Associazione a un socket Unix

Impostare l'ascolto su un socket Unix con <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ListenUnixSocket*> per migliorare le prestazioni con Nginx, come visualizzato nell'esempio seguente:

[!code-csharp[](kestrel/samples/3.x/KestrelSample/Program.cs?name=snippet_UnixSocket)]

* Nel file di configurazione nginx impostare la `server`  >  `location`  >  `proxy_pass` voce su `http://unix:/tmp/{KESTREL SOCKET}:/;` . `{KESTREL SOCKET}` nome del socket fornito a <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ListenUnixSocket*> (ad esempio, `kestrel-test.sock` nell'esempio precedente).
* Verificare che il socket sia scrivibile da nginx (ad esempio, `chmod go+w /tmp/kestrel-test.sock` ).

### <a name="port-0"></a>Porta 0

Quando si specifica il numero di porta `0`, Kestrel esegue l'associazione dinamica a una porta disponibile. L'esempio seguente indica come determinare la porta alla quale Kestrel ha eseguito l'associazione in fase di esecuzione:

[!code-csharp[](kestrel/samples/3.x/KestrelSample/Startup.cs?name=snippet_Configure&highlight=3-4,15-21)]

Quando si esegue l'app, l'output della finestra della console indica la porta dinamica in cui l'app può essere raggiunta:

```console
Listening on the following addresses: http://127.0.0.1:48508
```

### <a name="limitations"></a>Limitazioni

Configurare gli endpoint con i metodi seguenti:

* <xref:Microsoft.AspNetCore.Hosting.HostingAbstractionsWebHostBuilderExtensions.UseUrls*>
* L'argomento della riga di comando `--urls`
* La chiave di configurazione dell'host `urls`
* La variabile di ambiente `ASPNETCORE_URLS`

Questi metodi sono utili se si vuole che il codice funzioni con server diversi da Kestrel. Tenere presenti, tuttavia, le limitazioni seguenti:

* HTTPS non può essere usato con questi approcci, a meno che non venga specificato un certificato predefinito nella configurazione dell'endpoint HTTPS (ad esempio, usando la configurazione `KestrelServerOptions` o un file di configurazione come illustrato in precedenza in questo argomento).
* Quando sia l'approccio `Listen` che l'approccio `UseUrls` vengono usati contemporaneamente, gli endpoint `Listen` eseguono l'override degli endpoint `UseUrls`.

### <a name="iis-endpoint-configuration"></a>Configurazione dell'endpoint IIS

Quando si usa IIS, le associazioni di URL per le associazioni di override di IIS vengono impostate mediante `Listen` o `UseUrls`. Per altre informazioni, vedere l'articolo [Introduzione al modulo ASP.NET Core](xref:host-and-deploy/aspnet-core-module).

### <a name="listenoptionsprotocols"></a>ListenOptions.Protocols

La proprietà `Protocols` stabilisce i protocolli HTTP (`HttpProtocols`) abilitati in un endpoint di connessione o per il server. Assegnare un valore alla proprietà `Protocols` dall'enumerazione `HttpProtocols`.

| Valore di enumerazione `HttpProtocols` | Protocollo di connessione consentito |
| -------------------------- | ----------------------------- |
| `Http1`                    | Solo HTTP/1.1. Può essere usato con o senza TLS. |
| `Http2`                    | Solo HTTP/2. Può essere usato senza TLS solo se il client supporta una [modalità di conoscenza pregressa](https://tools.ietf.org/html/rfc7540#section-3.4). |
| `Http1AndHttp2`            | HTTP/1.1 e HTTP/2. HTTP/2 richiede che il client selezioni HTTP/2 nell'handshake TLS [(Application-Layer Protocol negotiation) ALPN](https://tools.ietf.org/html/rfc7301#section-3) in caso contrario, il valore predefinito per la connessione è HTTP/1.1. |

Il `ListenOptions.Protocols` valore predefinito per qualsiasi endpoint è `HttpProtocols.Http1AndHttp2` .

Restrizioni relative a TLS per HTTP/2:

* TLS versione 1.2 o successiva
* Rinegoziazione disabilitata
* Compressione disabilitata
* Dimensioni minime per lo scambio di chiavi temporanee:
  * Curva ellittica Diffie-Hellman (ECDHE) &lbrack; [RFC4492](https://www.ietf.org/rfc/rfc4492.txt) &rbrack; : minimo 224 bit
  * Diffie-Hellman campo finito (DHE) &lbrack; `TLS12` &rbrack; : minimo 2048 bit
* Il pacchetto di crittografia non è consentito. 

`TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256`&lbrack;`TLS-ECDHE`&rbrack; con la curva ellittica P-256 &lbrack; `FIPS186` &rbrack; è supportata per impostazione predefinita.

Nell'esempio seguente sono consentite connessioni HTTP/1.1 e HTTP/2 sulla porta 8000. Le connessioni sono protette da TLS con un certificato incluso:

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.Listen(IPAddress.Any, 8000, listenOptions =>
    {
        listenOptions.UseHttps("testCert.pfx", "testPassword");
    });
});
```

Usare il middleware di connessione per filtrare gli handshake TLS in base alla connessione per le crittografie specifiche, se necessario.

L'esempio seguente genera un'eccezione <xref:System.NotSupportedException> per qualsiasi algoritmo crittografico non supportato dall'app. In alternativa, definire e confrontare [ITlsHandshakeFeature. CipherAlgorithm](xref:Microsoft.AspNetCore.Connections.Features.ITlsHandshakeFeature.CipherAlgorithm) con un elenco di pacchetti di crittografia accettabili.

Non viene utilizzata alcuna crittografia con un algoritmo di crittografia [CipherAlgorithmType. null](xref:System.Security.Authentication.CipherAlgorithmType) .

```csharp
// using System.Net;
// using Microsoft.AspNetCore.Connections;

webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.Listen(IPAddress.Any, 8000, listenOptions =>
    {
        listenOptions.UseHttps("testCert.pfx", "testPassword");
        listenOptions.UseTlsFilter();
    });
});
```

```csharp
using System;
using System.Security.Authentication;
using Microsoft.AspNetCore.Connections.Features;

namespace Microsoft.AspNetCore.Connections
{
    public static class TlsFilterConnectionMiddlewareExtensions
    {
        public static IConnectionBuilder UseTlsFilter(
            this IConnectionBuilder builder)
        {
            return builder.Use((connection, next) =>
            {
                var tlsFeature = connection.Features.Get<ITlsHandshakeFeature>();

                if (tlsFeature.CipherAlgorithm == CipherAlgorithmType.Null)
                {
                    throw new NotSupportedException("Prohibited cipher: " +
                        tlsFeature.CipherAlgorithm);
                }

                return next();
            });
        }
    }
}
```

Il filtro della connessione può essere configurato anche tramite un' <xref:Microsoft.AspNetCore.Connections.IConnectionBuilder> espressione lambda:

```csharp
// using System;
// using System.Net;
// using System.Security.Authentication;
// using Microsoft.AspNetCore.Connections;
// using Microsoft.AspNetCore.Connections.Features;

webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.Listen(IPAddress.Any, 8000, listenOptions =>
    {
        listenOptions.UseHttps("testCert.pfx", "testPassword");
        listenOptions.Use((context, next) =>
        {
            var tlsFeature = context.Features.Get<ITlsHandshakeFeature>();

            if (tlsFeature.CipherAlgorithm == CipherAlgorithmType.Null)
            {
                throw new NotSupportedException(
                    $"Prohibited cipher: {tlsFeature.CipherAlgorithm}");
            }

            return next();
        });
    });
});
```

In Linux <xref:System.Net.Security.CipherSuitesPolicy> è possibile usare per filtrare gli handshake TLS in base alla connessione:

```csharp
// using System.Net.Security;
// using Microsoft.AspNetCore.Hosting;
// using Microsoft.AspNetCore.Server.Kestrel.Core;
// using Microsoft.Extensions.DependencyInjection;
// using Microsoft.Extensions.Hosting;

webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.ConfigureHttpsDefaults(listenOptions =>
    {
        listenOptions.OnAuthenticate = (context, sslOptions) =>
        {
            sslOptions.CipherSuitesPolicy = new CipherSuitesPolicy(
                new[]
                {
                    TlsCipherSuite.TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,
                    TlsCipherSuite.TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384,
                    // ...
                });
        };
    });
});
```

*Impostare il protocollo dalla configurazione*

`CreateDefaultBuilder` chiama `serverOptions.Configure(context.Configuration.GetSection("Kestrel"))` per impostazione predefinita per caricare la configurazione di Kestrel.

Nell' *appsettings.json* esempio seguente viene stabilito http/1.1 come protocollo di connessione predefinito per tutti gli endpoint:

```json
{
  "Kestrel": {
    "EndpointDefaults": {
      "Protocols": "Http1"
    }
  }
}
```

Nell' *appsettings.json* esempio seguente viene stabilito il protocollo di connessione HTTP/1.1 per un endpoint specifico:

```json
{
  "Kestrel": {
    "Endpoints": {
      "HttpsDefaultCert": {
        "Url": "https://localhost:5001",
        "Protocols": "Http1"
      }
    }
  }
}
```

I protocolli specificati nei valori di override del codice sono impostati dalla configurazione.

### <a name="url-prefixes"></a>Prefissi URL

Se si usa `UseUrls`, l'argomento della riga di comando `--urls`, la chiave di configurazione dell'host `urls` o la variabile di ambiente `ASPNETCORE_URLS`, i prefissi URL possono avere uno dei formati seguenti.

Sono validi solo i prefissi URL HTTP. Kestrel non supporta HTTPS quando la configurazione delle associazioni URL viene eseguita con `UseUrls`.

* Indirizzo IPv4 con numero di porta

  ```
  http://65.55.39.10:80/
  ```

  `0.0.0.0` è un caso speciale che esegue l'associazione a tutti gli indirizzi IPv4.

* Indirizzo IPv6 con numero di porta

  ```
  http://[0:0:0:0:0:ffff:4137:270a]:80/
  ```

  `[::]` è l'equivalente IPv6 di `0.0.0.0` per IPv4.

* Nome host con numero di porta

  ```
  http://contoso.com:80/
  http://*:80/
  ```

  I nomi host, `*` e `+` non sono casi particolari. Tutto ciò che non è riconosciuto come un indirizzo IP o un elemento `localhost` valido esegue l'associazione a tutti gli IP IPv4 e IPv6. Per associare nomi host diversi ad app ASP.NET Core diverse sulla stessa porta, usare [HTTP.sys](xref:fundamentals/servers/httpsys) o un server proxy inverso come IIS, Nginx o Apache.

  > [!WARNING]
  > Una configurazione che prevede un proxy inverso richiede il [filtro host](#host-filtering).

* Nome host `localhost` con numero di porta o IP di loopback con numero di porta

  ```
  http://localhost:5000/
  http://127.0.0.1:5000/
  http://[::1]:5000/
  ```

  Se è specificato `localhost`, Kestrel tenta l'associazione alle interfacce di loopback sia IPv4 che IPv6. Se la porta richiesta è in uso da parte di un altro servizio in una delle due interfacce di loopback, Kestrel non viene avviato. Se una delle due interfacce di loopback non è disponibile per qualsiasi altro motivo (in genere perché IPv6 non è supportato), Kestrel registra un avviso.

## <a name="host-filtering"></a>Filtro host

Mentre supporta la configurazione in base ai prefissi, ad esempio `http://example.com:5000`, Kestrel ignora quasi sempre il nome host. L'host `localhost` è un caso speciale usato per l'associazione agli indirizzi di loopback. Qualsiasi host che non sia un indirizzo IP esplicito esegue l'associazione a tutti gli indirizzi IP pubblici. Le intestazioni `Host` non vengono convalidate.

Come soluzione alternativa, usare il middleware di filtro host. Il middleware di filtro host viene fornito dal pacchetto [Microsoft. AspNetCore. HostFiltering](https://www.nuget.org/packages/Microsoft.AspNetCore.HostFiltering) , fornito in modo implicito per le app ASP.NET Core. Il middleware viene aggiunto da <xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder*>, che chiama <xref:Microsoft.AspNetCore.Builder.HostFilteringServicesExtensions.AddHostFiltering*>:

[!code-csharp[](kestrel/samples-snapshot/2.x/KestrelSample/Program.cs?name=snippet_Program&highlight=9)]

Per impostazione predefinita, il middleware di filtro host è disabilitato per impostazione predefinita. Per abilitare il middleware, definire una `AllowedHosts` chiave in *appsettings.json* / *appSettings. \<EnvironmentName> JSON*. Il valore è un elenco con valori delimitati da punto e virgola di nomi host senza numeri di porta:

*appsettings.json*:

```json
{
  "AllowedHosts": "example.com;localhost"
}
```

> [!NOTE]
> Il [middleware delle intestazioni inoltrate](xref:host-and-deploy/proxy-load-balancer) include anche un'opzione <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.AllowedHosts>. Il middleware di intestazioni inoltrate e il middleware di filtro host hanno funzionalità simili per diversi scenari. L'impostazione di `AllowedHosts` con il middleware delle intestazioni inoltrate è appropriato se l'intestazione `Host` non viene mantenuta durante l'inoltro delle richieste con un server proxy inverso o il bilanciamento del carico. L'impostazione di `AllowedHosts` con il middleware del filtro host è appropriata se si usa Kestrel come server perimetrale rivolto al pubblico o se l'intestazione `Host` viene inoltrata direttamente.
>
> Per altre informazioni sul middleware delle intestazioni inoltrate, vedere <xref:host-and-deploy/proxy-load-balancer>.

## <a name="libuv-transport-configuration"></a>Configurazione del trasporto libuv

Per i progetti che richiedono l'utilizzo di libuv ( <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderLibuvExtensions.UseLibuv%2A> ):

* Aggiungere una dipendenza per il [`Microsoft.AspNetCore.Server.Kestrel.Transport.Libuv`](https://www.nuget.org/packages/Microsoft.AspNetCore.Server.Kestrel.Transport.Libuv) pacchetto al file di progetto dell'app:

  ```xml
  <PackageReference Include="Microsoft.AspNetCore.Server.Kestrel.Transport.Libuv"
                    Version="{VERSION}" />
  ```

* Chiamare <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderLibuvExtensions.UseLibuv%2A> su `IWebHostBuilder` :

  ```csharp
  public class Program
  {
      public static void Main(string[] args)
      {
          CreateHostBuilder(args).Build().Run();
      }

      public static IHostBuilder CreateHostBuilder(string[] args) =>
          Host.CreateDefaultBuilder(args)
              .ConfigureWebHostDefaults(webBuilder =>
              {
                  webBuilder.UseLibuv();
                  webBuilder.UseStartup<Startup>();
              });
  }
  ```

## <a name="http11-request-draining"></a>Svuotamento richieste HTTP/1.1

L'apertura di connessioni HTTP richiede molto tempo. Per HTTPS, è anche a elevato utilizzo di risorse. Pertanto, gheppio tenta di riutilizzare le connessioni per il protocollo HTTP/1.1. Il corpo della richiesta deve essere completamente utilizzato per consentire il riutilizzo della connessione. L'app non usa sempre il corpo della richiesta, ad esempio le `POST` richieste in cui il server restituisce una risposta di reindirizzamento o 404. Nel `POST` caso-Reindirizzamento:

* Il client potrebbe avere già inviato parte dei `POST` dati.
* Il server scrive la risposta 301.
* Non è possibile usare la connessione per una nuova richiesta fino a quando i `POST` dati del corpo della richiesta precedente non sono stati completamente letti.
* Gheppio tenta di svuotare il corpo della richiesta. Lo svuotamento del corpo della richiesta implica la lettura e l'eliminazione dei dati senza elaborarli.

Il processo di svuotamento crea un tradoff tra la possibilità di riutilizzare la connessione e il tempo necessario per svuotare i dati rimanenti:

* Lo svuotamento ha un timeout di cinque secondi, che non è configurabile.
* Se tutti i dati specificati dall' `Content-Length` `Transfer-Encoding` intestazione o non sono stati letti prima del timeout, la connessione viene chiusa.

In alcuni casi può essere necessario terminare immediatamente la richiesta, prima o dopo la scrittura della risposta. È ad esempio possibile che i client dispongano di protezioni dati restrittive, pertanto la limitazione dei dati caricati potrebbe avere una priorità. In questi casi, per terminare una richiesta, chiamare [HttpContext. Abort](xref:Microsoft.AspNetCore.Http.HttpContext.Abort%2A) da un controller, una Razor pagina o un middleware.

Ci sono alcune avvertenze per chiamare `Abort` :

* La creazione di nuove connessioni può essere lenta e dispendiosa.
* Non vi è alcuna garanzia che il client abbia letto la risposta prima che la connessione venga chiusa.
* `Abort`La chiamata a deve essere rara e riservata per i casi di errore gravi, non per errori comuni.
  * Chiamare solo `Abort` quando è necessario risolvere un problema specifico. Ad esempio, chiamare `Abort` se i client malintenzionati tentano di eseguire `POST` dati o quando si verifica un bug nel codice client che genera richieste di grandi dimensioni o numerose.
  * Non chiamare le `Abort` situazioni di errore comuni, ad esempio HTTP 404 (non trovato).

Quando si chiama [HttpResponse. CompleteAsync](xref:Microsoft.AspNetCore.Http.HttpResponse.CompleteAsync%2A) , prima `Abort` di chiamare si garantisce che il server abbia completato la scrittura della risposta. Tuttavia, il comportamento del client non è prevedibile e potrebbe non essere in grado di leggere la risposta prima che la connessione venga interrotta.

Questo processo è diverso per HTTP/2 poiché il protocollo supporta l'interruzione dei singoli flussi di richiesta senza chiudere la connessione. Il timeout di svuotamento di cinque secondi non è applicabile. Se dopo il completamento di una risposta sono presenti dati del corpo della richiesta non letti, il server invia un frame RST HTTP/2. I frame di dati del corpo della richiesta aggiuntivi vengono ignorati.

Se possibile, è preferibile che i client utilizzino l'intestazione della richiesta [expect: 100-continue](https://developer.mozilla.org/docs/Web/HTTP/Status/100) e attendano la risposta del server prima di iniziare a inviare il corpo della richiesta. Che fornisce al client la possibilità di esaminare la risposta e interrompere prima di inviare dati non necessari.

## <a name="additional-resources"></a>Risorse aggiuntive

* Quando si usano i socket UNIX in Linux, il socket non viene eliminato automaticamente all'arresto dell'app. Per altre informazioni, vedere [questo problema di GitHub](https://github.com/dotnet/aspnetcore/issues/14134).
* <xref:test/troubleshoot>
* <xref:security/enforcing-ssl>
* <xref:host-and-deploy/proxy-load-balancer>
* [RFC 7230: Message Syntax and Routing (Section 5.4: Host)](https://tools.ietf.org/html/rfc7230#section-5.4) (RFC 7230: Sintassi e routing dei messaggi (sezione 5.4: Host))

::: moniker-end

::: moniker range="= aspnetcore-2.2"

Kestrel è un [server Web per ASP.NET Core](xref:fundamentals/servers/index) multipiattaforma. Kestrel è il server Web incluso per impostazione predefinita nei modelli di progetto di ASP.NET Core.

Kestrel supporta gli scenari seguenti:

* HTTPS
* Aggiornamento opaco usato per abilitare [WebSocket](https://github.com/aspnet/websockets)
* Socket Unix ad alte prestazioni dietro Nginx
* HTTP/2 (tranne che in macOS&dagger;)

&dagger;HTTP/2 verrà supportato in macOS in una versione futura.

Kestrel è supportato in tutte le piattaforme e le versioni supportate da .NET Core.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/servers/kestrel/samples/2.x) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="http2-support"></a>Supporto HTTP/2

[HTTP/2](https://httpwg.org/specs/rfc7540.html) è disponibile per le app ASP.NET Core se vengono soddisfatti i requisiti di base seguenti:

* Sistema operativo&dagger;
  * Windows Server 2016/Windows 10 o versioni successive&Dagger;
  * Linux con OpenSSL 1.0.2 o versioni successive (ad esempio, Ubuntu 16.04 o versioni successive)
* Framework di destinazione: .NET Core 2.2 o versioni successive
* Connessione [ALPN (Application-Layer Protocol Negotiation)](https://tools.ietf.org/html/rfc7301#section-3)
* Connessione TLS 1.2 o successiva

&dagger;HTTP/2 verrà supportato in macOS in una versione futura.
&Dagger;Kestrel ha un supporto limitato per HTTP/2 in Windows Server 2012 R2 e Windows 8.1. Il supporto è limitato perché l'elenco di suite di crittografia TLS supportate disponibili in questi sistemi operativi è limitato. Un certificato generato con un algoritmo ECDSA potrebbe essere necessario per proteggere le connessioni TLS.

Se viene stabilita una connessione HTTP/2, [HttpRequest.Protocol](xref:Microsoft.AspNetCore.Http.HttpRequest.Protocol*) corrisponde a `HTTP/2`.

HTTP/2 è disabilitato per impostazione predefinita. Per altre informazioni sulla configurazione, vedere le sezioni [Opzioni Kestrel](#kestrel-options) e [ListenOptions.Protocols](#listenoptionsprotocols).

## <a name="when-to-use-kestrel-with-a-reverse-proxy"></a>Quando usare Kestrel con un proxy inverso

È possibile usare Kestrel da solo o in combinazione con un *server proxy inverso*, ad esempio [Internet Information Services (IIS)](https://www.iis.net/), [Nginx](https://nginx.org) o [Apache](https://httpd.apache.org/). Il server proxy inverso riceve le richieste HTTP dalla rete e le inoltra a Kestrel.

Kestrel usato come server Web perimetrale (esposto a Internet):

![Kestrel comunica direttamente con Internet senza un server proxy inverso](kestrel/_static/kestrel-to-internet2.png)

Kestrel usato in una configurazione proxy inverso:

![Kestrel comunica indirettamente con Internet attraverso un server proxy inverso, ad esempio IIS, Nginx o Apache](kestrel/_static/kestrel-to-internet.png)

La configurazione, con o senza un server proxy inverso, è una configurazione di hosting supportata.

Se viene utilizzato come server perimetrale in assenza di un server proxy inverso, Kestrel non supporta la condivisione tra più processi dell'IP e della porta. Quando è configurato per restare in ascolto su una porta, Kestrel gestisce tutto il traffico per tale porta indipendentemente dalle intestazioni `Host` delle richieste. Un proxy inverso che può condividere le porte può eseguire l'inoltro delle richieste a Kestrel su un unico IP e un'unica porta.

Anche se la presenza di un server proxy inverso non è necessaria, può risultare utile per i motivi seguenti.

Un proxy inverso:

* Può limitare l'area della superficie di attacco pubblica esposta delle app ospitate.
* Offre un livello aggiuntivo di configurazione e protezione.
* Potrebbe offrire un'integrazione migliore con l'infrastruttura esistente.
* Semplifica il bilanciamento del carico e la configurazione di comunicazioni protette (HTTPS). Solo il server proxy inverso richiede un certificato X. 509 e il server è in grado di comunicare con i server dell'app nella rete interna tramite HTTP normale.

> [!WARNING]
> Una configurazione che prevede un proxy inverso richiede il [filtro host](#host-filtering).

## <a name="how-to-use-kestrel-in-aspnet-core-apps"></a>Come usare Kestrel nelle app ASP.NET Core

Il pacchetto [Microsoft. AspNetCore. Server. gheppio](https://www.nuget.org/packages/Microsoft.AspNetCore.Server.Kestrel/) è incluso nel [metapacchetto Microsoft. AspNetCore. app](xref:fundamentals/metapackage-app).

I modelli di progetto ASP.NET Core usano Kestrel per impostazione predefinita. In *Program.cs* il codice del modello chiama <xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder*>, che chiama <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderKestrelExtensions.UseKestrel*> in background.

[!code-csharp[](kestrel/samples/2.x/KestrelSample/Program.cs?name=snippet_DefaultBuilder&highlight=7)]

Per ulteriori informazioni su `CreateDefaultBuilder` e sulla compilazione dell'host, vedere la sezione *configurare un host* di <xref:fundamentals/host/web-host#set-up-a-host> .

Per fornire una configurazione aggiuntiva dopo la chiamata di `CreateDefaultBuilder`, usare `ConfigureKestrel`:

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .ConfigureKestrel((context, serverOptions) =>
        {
            // Set properties and call methods on serverOptions
        });
```

Se l'app non chiama `CreateDefaultBuilder` per configurare l'host, chiamare <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderKestrelExtensions.UseKestrel*> **prima** di chiamare `ConfigureKestrel`:

```csharp
public static void Main(string[] args)
{
    var host = new WebHostBuilder()
        .UseContentRoot(Directory.GetCurrentDirectory())
        .UseKestrel()
        .UseIISIntegration()
        .UseStartup<Startup>()
        .ConfigureKestrel((context, serverOptions) =>
        {
            // Set properties and call methods on serverOptions
        })
        .Build();

    host.Run();
}
```

## <a name="kestrel-options"></a>Opzioni Kestrel

Il server Web Kestrel dispone di opzioni di configurazione dei vincoli che risultano particolarmente utili nelle distribuzioni con connessione Internet.

Impostare i vincoli per la proprietà <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Limits> della classe <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions>. La proprietà `Limits` contiene un'istanza della classe <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits>.

Negli esempi seguenti viene usato lo spazio dei nomi <xref:Microsoft.AspNetCore.Server.Kestrel.Core>:

```csharp
using Microsoft.AspNetCore.Server.Kestrel.Core;
```

Le opzioni gheppio, che sono configurate nel codice C# negli esempi seguenti, possono essere impostate anche usando un [provider di configurazione](xref:fundamentals/configuration/index). Il provider di configurazione file, ad esempio, può caricare la configurazione di Gheppio da un oggetto *appsettings.json* o *appSettings. { File Environment}. JSON* :

```json
{
  "Kestrel": {
    "Limits": {
      "MaxConcurrentConnections": 100,
      "MaxConcurrentUpgradedConnections": 100
    }
  }
}
```

Usare **uno** degli approcci seguenti:

* Configurare gheppio in `Startup.ConfigureServices` :

  1. Inserire un'istanza di `IConfiguration` nella `Startup` classe. Nell'esempio seguente si presuppone che la configurazione inserita venga assegnata alla `Configuration` Proprietà.
  2. In `Startup.ConfigureServices` caricare la `Kestrel` sezione della configurazione nella configurazione di Gheppio:

     ```csharp
     using Microsoft.Extensions.Configuration
     
     public class Startup
     {
         public Startup(IConfiguration configuration)
         {
             Configuration = configuration;
         }

         public IConfiguration Configuration { get; }

         public void ConfigureServices(IServiceCollection services)
         {
             services.Configure<KestrelServerOptions>(
                 Configuration.GetSection("Kestrel"));
         }

         public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
         {
             ...
         }
     }
     ```

* Configurare il gheppio durante la compilazione dell'host:

  In *Program.cs* caricare la `Kestrel` sezione della configurazione nella configurazione di Gheppio:

  ```csharp
  // using Microsoft.Extensions.DependencyInjection;

  public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
      WebHost.CreateDefaultBuilder(args)
          .ConfigureServices((context, services) =>
          {
              services.Configure<KestrelServerOptions>(
                  context.Configuration.GetSection("Kestrel"));
          })
          .UseStartup<Startup>();
  ```

Entrambi gli approcci precedenti funzionano con qualsiasi [provider di configurazione](xref:fundamentals/configuration/index).

### <a name="keep-alive-timeout"></a>Timeout keep-alive

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.KeepAliveTimeout>

Ottiene o imposta il [timeout keep-alive](https://tools.ietf.org/html/rfc7230#section-6.5). Il valore predefinito è 2 minuti.

[!code-csharp[](kestrel/samples/2.x/KestrelSample/Program.cs?name=snippet_Limits&highlight=15)]

### <a name="maximum-client-connections"></a>Numero massimo di connessioni client

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.MaxConcurrentConnections>
<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.MaxConcurrentUpgradedConnections>

È possibile impostare il numero massimo di connessioni TCP aperte simultaneamente per l'intera app con il codice seguente:

[!code-csharp[](kestrel/samples/2.x/KestrelSample/Program.cs?name=snippet_Limits&highlight=3)]

È previsto un limite separato per le connessioni che sono state aggiornate da HTTP o HTTPS a un altro protocollo (ad esempio su una richiesta WebSocket). Dopo l'aggiornamento di una connessione, questa non viene conteggiata per il limite `MaxConcurrentConnections`.

[!code-csharp[](kestrel/samples/2.x/KestrelSample/Program.cs?name=snippet_Limits&highlight=4)]

Per impostazione predefinita, il numero massimo di connessioni è illimitato (null).

### <a name="maximum-request-body-size"></a>Dimensione massima del corpo della richiesta

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.MaxRequestBodySize>

La dimensione massima predefinita del corpo della richiesta è pari a 30.000.000 di byte, equivalenti a circa 28,6 MB.

Il metodo consigliato per ignorare il limite in un'app ASP.NET Core MVC è l'uso dell'attributo <xref:Microsoft.AspNetCore.Mvc.RequestSizeLimitAttribute> in un metodo di azione:

```csharp
[RequestSizeLimit(100000000)]
public IActionResult MyActionMethod()
```

L'esempio seguente illustra come configurare il vincolo per l'app in ogni richiesta:

[!code-csharp[](kestrel/samples/2.x/KestrelSample/Program.cs?name=snippet_Limits&highlight=5)]

Eseguire l'override dell'impostazione per una richiesta specifica nel middleware:

[!code-csharp[](kestrel/samples/2.x/KestrelSample/Startup.cs?name=snippet_Limits&highlight=3-4)]

Viene generata un'eccezione se l'app configura il limite per una richiesta dopo che l'app ha iniziato a leggere la richiesta. Una proprietà `IsReadOnly` indica se la proprietà `MaxRequestBodySize` è in stato di sola lettura e pertanto è troppo tardi per configurare il limite.

Quando un'app viene eseguita [out-of-process](xref:host-and-deploy/iis/index#out-of-process-hosting-model) dietro il [modulo ASP.NET Core](xref:host-and-deploy/aspnet-core-module), il limite della dimensione del corpo della richiesta di Kestrel è disabilitato perché viene già impostato da IIS.

### <a name="minimum-request-body-data-rate"></a>Velocità minima dei dati del corpo della richiesta

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.MinRequestBodyDataRate>
<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.MinResponseDataRate>

Kestrel controlla ogni secondo se i dati arrivano alla velocità in byte al secondo specificata. Se la frequenza scende sotto il valore minimo, si è verificato il timeout della connessione. Il periodo di tolleranza è la quantità di tempo che il gheppio concede al client di aumentare la velocità di invio fino al minimo. la frequenza non viene controllata durante tale periodo di tempo. Il periodo di prova consente di evitare l'interruzione di connessioni che inizialmente inviano i dati a velocità ridotta a causa dell'avvio lento del protocollo TCP.

La velocità minima predefinita è di 240 byte al secondo, con un periodo di tolleranza di 5 secondi.

Anche per la risposta è prevista una velocità minima. Il codice per impostare il limite della richiesta e della risposta è identico e varia solo per `RequestBody` o `Response` nei nomi di proprietà e di interfaccia.

L'esempio seguente visualizza come configurare la velocità minima dei dati in *Program.cs*:

[!code-csharp[](kestrel/samples/2.x/KestrelSample/Program.cs?name=snippet_Limits&highlight=6-9)]

Ignorare i limiti di velocità minima per ogni richiesta nel middleware:

[!code-csharp[](kestrel/samples/2.x/KestrelSample/Startup.cs?name=snippet_Limits&highlight=6-21)]

Nessuna delle due funzionalità relative alla velocità, a cui si fa riferimento nell'esempio precedente, è presente in `HttpContext.Features` per le richieste HTTP/2 perché la modifica dei limiti di velocità per ogni richiesta non è supportata per HTTP/2 a causa del supporto del protocollo per il multiplexing delle richieste. I limiti di velocità a livello di server configurati tramite `KestrelServerOptions.Limits` vengono tuttavia applicati alle connessioni HTTP/1.x e HTTP/2.

### <a name="request-headers-timeout"></a>Timeout delle intestazioni delle richieste

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.RequestHeadersTimeout>

Ottiene o imposta la quantità massima di tempo che il server dedica alla ricezione delle intestazioni delle richieste. Il valore predefinito è 30 secondi.

[!code-csharp[](kestrel/samples/2.x/KestrelSample/Program.cs?name=snippet_Limits&highlight=16)]

### <a name="maximum-streams-per-connection"></a>Numero massimo di flussi per connessione

`Http2.MaxStreamsPerConnection` limita il numero di flussi di richieste simultanee per ogni connessione HTTP/2. I flussi in eccesso vengono rifiutati.

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .ConfigureKestrel((context, serverOptions) =>
        {
            serverOptions.Limits.Http2.MaxStreamsPerConnection = 100;
        });
```

Il valore predefinito è 100.

### <a name="header-table-size"></a>Dimensioni della tabella delle intestazioni

Il decodificatore HPACK decomprime le intestazioni HTTP per le connessioni HTTP/2. `Http2.HeaderTableSize` limita le dimensioni della tabella di compressione delle intestazioni usata dal decodificatore HPACK. Il valore viene specificato in ottetti e deve essere maggiore di zero (0).

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .ConfigureKestrel((context, serverOptions) =>
        {
            serverOptions.Limits.Http2.HeaderTableSize = 4096;
        });
```

Il valore predefinito è 4096.

### <a name="maximum-frame-size"></a>Dimensione massima del frame

`Http2.MaxFrameSize` indica le dimensioni massime del payload del frame di connessione HTTP/2 da ricevere. Il valore viene specificato in ottetti e deve essere compreso tra 2^14 (16.384) e 2^24-1 (16.777.215).

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .ConfigureKestrel((context, serverOptions) =>
        {
            serverOptions.Limits.Http2.MaxFrameSize = 16384;
        });
```

Il valore predefinito è 2^14 (16.384).

### <a name="maximum-request-header-size"></a>Dimensioni massime dell'intestazione della richiesta

`Http2.MaxRequestHeaderFieldSize` indica le dimensioni massime consentite in ottetti di valori di intestazione di richiesta. Questo limite si applica insieme sia al nome che al valore nelle rappresentazioni compresse e non compresse. Il valore deve essere maggiore di zero (0).

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .ConfigureKestrel((context, serverOptions) =>
        {
            serverOptions.Limits.Http2.MaxRequestHeaderFieldSize = 8192;
        });
```

Il valore predefinito è 8.192.

### <a name="initial-connection-window-size"></a>Dimensioni della finestra di connessione iniziali

`Http2.InitialConnectionWindowSize` indica il numero massimo di byte di dati del corpo della richiesta che il server memorizza nel buffer in una volta, aggregati in tutte le richieste (flussi), per ogni connessione. Le richieste sono anche limitate da `Http2.InitialStreamWindowSize`. Il valore deve essere maggiore di o uguale a 65.535 e minore di 2^31 (2.147.483.648).

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .ConfigureKestrel((context, serverOptions) =>
        {
            serverOptions.Limits.Http2.InitialConnectionWindowSize = 131072;
        });
```

Il valore predefinito è 128 KB (131.072).

### <a name="initial-stream-window-size"></a>Dimensioni della finestra di flusso iniziali

`Http2.InitialStreamWindowSize` indica il numero massimo di byte di dati del corpo della richiesta che il server memorizza nel buffer in una volta per ogni richiesta (flusso). Le richieste sono anche limitate da `Http2.InitialStreamWindowSize`. Il valore deve essere maggiore di o uguale a 65.535 e minore di 2^31 (2.147.483.648).

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .ConfigureKestrel((context, serverOptions) =>
        {
            serverOptions.Limits.Http2.InitialStreamWindowSize = 98304;
        });
```

Il valore predefinito è 96 KB (98.304).

### <a name="synchronous-io"></a>I/O sincrono

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.AllowSynchronousIO> Controlla se è consentito l'I/O sincrono per la richiesta e la risposta. Il valore predefinito è `true`.

> [!WARNING]
> Un numero elevato di operazioni di I/O sincrone bloccanti può causare l'esaurimento del pool di thread, il che rende l'app non risponde. Abilitare solo `AllowSynchronousIO` quando si usa una libreria che non supporta l'I/O asincrono.

L'esempio seguente abilita l'I/O sincrono:

[!code-csharp[](kestrel/samples/2.x/KestrelSample/Program.cs?name=snippet_SyncIO)]

Per informazioni su altre opzioni e limiti di Kestrel, vedere:

* <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions>
* <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits>
* <xref:Microsoft.AspNetCore.Server.Kestrel.Core.ListenOptions>

## <a name="endpoint-configuration"></a>Configurazione dell'endpoint

Per impostazione predefinita, ASP.NET Core è associato a:

* `http://localhost:5000`
* `https://localhost:5001` (quando è presente un certificato di sviluppo locale)

Specificare gli URL usando gli elementi seguenti:

* La variabile di ambiente `ASPNETCORE_URLS`.
* L'argomento della riga di comando `--urls`.
* La chiave di configurazione dell'host `urls`.
* Il metodo di estensione `UseUrls`.

Il valore specificato usando i metodi seguenti può essere uno o più endpoint HTTP e HTTPS (HTTPS se è disponibile un certificato predefinito). Configurare il valore come un elenco delimitato da punto e virgola (ad esempio, `"Urls": "http://localhost:8000;http://localhost:8001"`).

Per altre informazioni su questi approcci, vedere [URL del server](xref:fundamentals/host/web-host#server-urls) e [Override della configurazione](xref:fundamentals/host/web-host#override-configuration).

Viene creato un certificato di sviluppo nei casi seguenti:

* Quando viene installato [.NET Core SDK](/dotnet/core/sdk).
* Quando per creare un certificato viene usato [lo strumento dev-certs](xref:aspnetcore-2.1#https).

Alcuni browser richiedono la concessione di autorizzazioni esplicite per considerare attendibile il certificato di sviluppo locale.

I modelli di progetto consentono di configurare le app per l'esecuzione su HTTPS per impostazione predefinita e includono il [reindirizzamento HTTPS e il supporto HSTS](xref:security/enforcing-ssl).

Chiamare i metodi <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Listen*> oppure <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ListenUnixSocket*> su <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions> per configurare le porte e i prefissi URL per Kestrel.

Anche `UseUrls`, l'argomento della riga di comando `--urls`, la chiave di configurazione dell'host `urls` e la variabile di ambiente `ASPNETCORE_URLS` funzionano, ma con le limitazioni indicate più avanti nella sezione (deve essere disponibile un certificato predefinito per la configurazione dell'endopoint HTTPS).

`KestrelServerOptions` configurazione

### <a name="configureendpointdefaultsactionlistenoptions"></a>ConfigureEndpointDefaults (azione \<ListenOptions> )

Specifica un elemento di configurazione `Action` da eseguire per ogni endpoint specificato. Se si chiama `ConfigureEndpointDefaults` più volte, gli elementi `Action` precedenti vengono sostituiti con l'ultimo elemento `Action` specificato.

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .ConfigureKestrel((context, serverOptions) =>
        {
            serverOptions.ConfigureEndpointDefaults(listenOptions =>
            {
                // Configure endpoint defaults
            });
        });
```

> [!NOTE]
> Per gli endpoint creati chiamando <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Listen*> **prima** della chiamata a <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ConfigureEndpointDefaults*> non verranno applicati i valori predefiniti.

### <a name="configurehttpsdefaultsactionhttpsconnectionadapteroptions"></a>ConfigureHttpsDefaults (azione \<HttpsConnectionAdapterOptions> )

Specifica un elemento di configurazione `Action` da eseguire per ogni endpoint HTTPS. Se si chiama `ConfigureHttpsDefaults` più volte, gli elementi `Action` precedenti vengono sostituiti con l'ultimo elemento `Action` specificato.

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .ConfigureKestrel((context, serverOptions) =>
        {
            serverOptions.ConfigureHttpsDefaults(listenOptions =>
            {
                // certificate is an X509Certificate2
                listenOptions.ServerCertificate = certificate;
            });
        });
```

> [!NOTE]
> Per gli endpoint creati chiamando <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Listen*> **prima** della chiamata a <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ConfigureHttpsDefaults*> non verranno applicati i valori predefiniti.


### <a name="configureiconfiguration"></a>Configure(IConfiguration)

Crea un loader di configurazione per la configurazione di Kestrel che accetta un elemento <xref:Microsoft.Extensions.Configuration.IConfiguration> come input. L'ambito della configurazione deve essere la sezione di configurazione per Kestrel.

### <a name="listenoptionsusehttps"></a>ListenOptions.UseHttps

Configura Kestrel per l'uso di HTTPS.

Estensioni `ListenOptions.UseHttps`:

* `UseHttps`: Configurare gheppio per l'uso di HTTPS con il certificato predefinito. Genera un'eccezione se non è stato configurato alcun certificato predefinito.
* `UseHttps(string fileName)`
* `UseHttps(string fileName, string password)`
* `UseHttps(string fileName, string password, Action<HttpsConnectionAdapterOptions> configureOptions)`
* `UseHttps(StoreName storeName, string subject)`
* `UseHttps(StoreName storeName, string subject, bool allowInvalid)`
* `UseHttps(StoreName storeName, string subject, bool allowInvalid, StoreLocation location)`
* `UseHttps(StoreName storeName, string subject, bool allowInvalid, StoreLocation location, Action<HttpsConnectionAdapterOptions> configureOptions)`
* `UseHttps(X509Certificate2 serverCertificate)`
* `UseHttps(X509Certificate2 serverCertificate, Action<HttpsConnectionAdapterOptions> configureOptions)`
* `UseHttps(Action<HttpsConnectionAdapterOptions> configureOptions)`

Parametri `ListenOptions.UseHttps`:

* `filename` è il percorso e il nome file di un file di certificato, relativo alla directory che contiene i file di contenuto dell'app.
* `password` è la password richiesta per accedere ai dati del certificato X.509.
* `configureOptions` è un elemento `Action` per configurare `HttpsConnectionAdapterOptions`. Restituisce `ListenOptions`.
* `storeName` è l'archivio certificati da cui caricare il certificato.
* `subject` è il nome dell'oggetto del certificato.
* `allowInvalid` indica se devono essere considerati i certificati non validi, come ad esempio i certificati autofirmati.
* `location` è il percorso dell'archivio da cui caricare il certificato.
* `serverCertificate` è il certificato X.509.

Nell'ambiente di produzione, HTTPS deve essere configurato in modo esplicito. Come minimo, è necessario specificare un certificato predefinito.

Configurazioni supportate descritte in seguito:

* Nessuna configurazione
* Sostituire il certificato predefinito della configurazione
* Modificare le impostazioni predefinite nel codice

*Nessuna configurazione*

Kestrel è in ascolto su `http://localhost:5000` e `https://localhost:5001` (se è disponibile un certificato predefinito).

<a name="configuration"></a>

*Sostituire il certificato predefinito della configurazione*

`CreateDefaultBuilder` chiama `Configure(context.Configuration.GetSection("Kestrel"))` per impostazione predefinita per caricare la configurazione di Kestrel. È disponibile per Kestrel uno schema di configurazione delle impostazioni delle app HTTPS predefinito. Configurare più endpoint, inclusi gli URL e i certificati da usare, da un file su disco o da un archivio certificati.

Nell'esempio seguente *appsettings.json* :

* Impostare **AllowInvalid** su `true` per consentire l'uso di certificati non validi, come ad esempio i certificati autofirmati.
* Tutti gli endpoint HTTPS che non specificano un certificato (**HttpsDefaultCert** nell'esempio che segue) usano il certificato definito in **Certificates** > **Default** o il certificato di sviluppo.

```json
{
  "Kestrel": {
    "Endpoints": {
      "Http": {
        "Url": "http://localhost:5000"
      },

      "HttpsInlineCertFile": {
        "Url": "https://localhost:5001",
        "Certificate": {
          "Path": "<path to .pfx file>",
          "Password": "<certificate password>"
        }
      },

      "HttpsInlineCertStore": {
        "Url": "https://localhost:5002",
        "Certificate": {
          "Subject": "<subject; required>",
          "Store": "<certificate store; required>",
          "Location": "<location; defaults to CurrentUser>",
          "AllowInvalid": "<true or false; defaults to false>"
        }
      },

      "HttpsDefaultCert": {
        "Url": "https://localhost:5003"
      },

      "Https": {
        "Url": "https://*:5004",
        "Certificate": {
          "Path": "<path to .pfx file>",
          "Password": "<certificate password>"
        }
      }
    },
    "Certificates": {
      "Default": {
        "Path": "<path to .pfx file>",
        "Password": "<certificate password>"
      }
    }
  }
}
```

Un'alternativa all'uso di **Path** e **Password** per qualsiasi nodo del certificato consiste nello specificare il certificato usando i campi dell'archivio certificati. Ad esempio, il certificato predefinito **Certificates**  >   può essere specificato come:

```json
"Default": {
  "Subject": "<subject; required>",
  "Store": "<cert store; required>",
  "Location": "<location; defaults to CurrentUser>",
  "AllowInvalid": "<true or false; defaults to false>"
}
```

Note di schema:

* I nomi degli endpoint non applicano la distinzione tra maiuscole e minuscole. Ad esempio, sono validi sia `HTTPS` che `Https`.
* Il parametro `Url` è obbligatorio per ogni endpoint. Il formato per questo parametro è uguale a quello del parametro di configurazione di primo livello `Urls`, con la differenza che si limita a un singolo valore.
* Questi endpoint sostituiscono quelli definiti nella configurazione di primo livello `Urls`, non vi si aggiungono. Gli endpoint definiti nel codice tramite `Listen` si aggiungono agli endpoint definiti nella sezione di configurazione.
* La sezione `Certificate` è facoltativa. Se la sezione `Certificate` non è specificata, vengono usati i valori predefiniti definiti negli scenari precedenti. Se non è disponibile alcun valore predefinito, il server genera un'eccezione e non viene avviato.
* La `Certificate` sezione supporta sia la password del **percorso** &ndash;  che i certificati dell'archivio dell' **oggetto** &ndash;  .
* In questo modo è possibile definire un numero qualsiasi di endpoint, purché non provochino conflitti di porte.
* `options.Configure(context.Configuration.GetSection("{SECTION}"))` restituisce un oggetto `KestrelConfigurationLoader` con un metodo `.Endpoint(string name, listenOptions => { })` che può essere usato per integrare le impostazioni dell'endpoint configurato:

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .UseKestrel((context, serverOptions) =>
        {
            serverOptions.Configure(context.Configuration.GetSection("Kestrel"))
                .Endpoint("HTTPS", listenOptions =>
                {
                    listenOptions.HttpsOptions.SslProtocols = SslProtocols.Tls12;
                });
        });
```

`KestrelServerOptions.ConfigurationLoader` è possibile accedere direttamente a per continuare a scorrere sul caricatore esistente, ad esempio quello fornito da <xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder*> .

* La sezione di configurazione per ogni endpoint è disponibile nelle opzioni del `Endpoint` metodo in modo che sia possibile leggere le impostazioni personalizzate.
* È possibile caricare più configurazioni chiamando ancora `options.Configure(context.Configuration.GetSection("{SECTION}"))` con un'altra sezione. Viene usata solo l'ultima configurazione, a meno che `Load` sia stato chiamato esplicitamente in istanze precedenti. Il metapacchetto non chiama `Load` in modo che la sezione di configurazione predefinita venga sostituita.
* `KestrelConfigurationLoader` riflette la famiglia di API `Listen` da `KestrelServerOptions` all'overload di `Endpoint`, in modo che gli endpoint di codice e di configurazione possano essere configurati nella stessa posizione. Questi overload non usano nomi e usano solo le impostazioni predefinite della configurazione.

*Modificare le impostazioni predefinite nel codice*

`ConfigureEndpointDefaults` e `ConfigureHttpsDefaults` possono essere usati per modificare le impostazioni predefinite per `ListenOptions` e `HttpsConnectionAdapterOptions`, inclusa l'esecuzione dell'override del certificato predefinito specificato nello scenario precedente. `ConfigureEndpointDefaults` e `ConfigureHttpsDefaults` devono essere chiamati prima che venga configurato qualsiasi endpoint.

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .UseKestrel((context, serverOptions) =>
        {
            serverOptions.ConfigureEndpointDefaults(listenOptions =>
            {
                // Configure endpoint defaults
            });
            
            serverOptions.ConfigureHttpsDefaults(listenOptions =>
            {
                listenOptions.SslProtocols = SslProtocols.Tls12;
            });
        });
```

*Supporto kestrel per SNI*

[L'Indicazione nome server (SNI)](https://tools.ietf.org/html/rfc6066#section-3) può essere usata per ospitare più domini sulla stessa porta e indirizzo IP. Perché SNI funzioni è necessario che il client invii al server il nome host per la sessione protetta durante l'handshake TLS in modo che il server possa specificare il certificato corretto. Il client usa il certificato specificato per la comunicazione crittografata con il server durante la sessione protetta che segue l'handshake TLS.

Kestrel supporta SNI tramite il callback `ServerCertificateSelector`. Il callback viene richiamato una volta per ogni connessione per consentire all'app di controllare il nome host e selezionare il certificato appropriato.

Il supporto SNI richiede:

* In esecuzione su Framework di destinazione `netcoreapp2.1` o versione successiva. In `net461` o versioni successive, il callback viene richiamato, ma `name` è sempre `null` . L'elemento `name` è `null` anche se il client non specifica il parametro del nome host nell'handshake TLS.
* Esecuzione di tutti i siti Web nella stessa istanza di Kestrel. Kestrel supporta la condivisione di un indirizzo IP e di una porta tra più istanze solo con un proxy inverso.

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .ConfigureKestrel((context, serverOptions) =>
        {
            serverOptions.ListenAnyIP(5005, listenOptions =>
            {
                listenOptions.UseHttps(httpsOptions =>
                {
                    var localhostCert = CertificateLoader.LoadFromStoreCert(
                        "localhost", "My", StoreLocation.CurrentUser,
                        allowInvalid: true);
                    var exampleCert = CertificateLoader.LoadFromStoreCert(
                        "example.com", "My", StoreLocation.CurrentUser,
                        allowInvalid: true);
                    var subExampleCert = CertificateLoader.LoadFromStoreCert(
                        "sub.example.com", "My", StoreLocation.CurrentUser,
                        allowInvalid: true);
                    var certs = new Dictionary<string, X509Certificate2>(
                        StringComparer.OrdinalIgnoreCase);
                    certs["localhost"] = localhostCert;
                    certs["example.com"] = exampleCert;
                    certs["sub.example.com"] = subExampleCert;

                    httpsOptions.ServerCertificateSelector = (connectionContext, name) =>
                    {
                        if (name != null && certs.TryGetValue(name, out var cert))
                        {
                            return cert;
                        }

                        return exampleCert;
                    };
                });
            });
        });
```

### <a name="connection-logging"></a>Registrazione connessione

Chiamare <xref:Microsoft.AspNetCore.Hosting.ListenOptionsConnectionLoggingExtensions.UseConnectionLogging*> per creare log a livello di debug per la comunicazione a livello di byte in una connessione. La registrazione delle connessioni è utile per la risoluzione dei problemi nella comunicazione di basso livello, ad esempio durante la crittografia TLS e dietro i proxy. Se `UseConnectionLogging` viene inserito prima `UseHttps` , viene registrato il traffico crittografato. Se `UseConnectionLogging` viene inserito dopo `UseHttps` , viene registrato il traffico decrittografato.

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.Listen(IPAddress.Any, 8000, listenOptions =>
    {
        listenOptions.UseConnectionLogging();
    });
});
```

### <a name="bind-to-a-tcp-socket"></a>Associazione a un socket TCP

Il metodo <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Listen*> esegue l'associazione a un socket TCP e un'espressione lambda per le opzioni consente di configurare un certificato X.509:

[!code-csharp[](kestrel/samples/2.x/KestrelSample/Program.cs?name=snippet_TCPSocket&highlight=9-16)]

L'esempio configura HTTPS per un endpoint con <xref:Microsoft.AspNetCore.Server.Kestrel.Core.ListenOptions>. Usare la stessa API per configurare altre impostazioni di Kestrel per endpoint specifici.

[!INCLUDE [How to make an X.509 cert](~/includes/make-x509-cert.md)]

### <a name="bind-to-a-unix-socket"></a>Associazione a un socket Unix

Impostare l'ascolto su un socket Unix con <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ListenUnixSocket*> per migliorare le prestazioni con Nginx, come visualizzato nell'esempio seguente:

[!code-csharp[](kestrel/samples/2.x/KestrelSample/Program.cs?name=snippet_UnixSocket)]

* Nel file nginx confiuguration impostare la `server`  >  `location`  >  `proxy_pass` voce su `http://unix:/tmp/{KESTREL SOCKET}:/;` . `{KESTREL SOCKET}` nome del socket fornito a <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ListenUnixSocket*> (ad esempio, `kestrel-test.sock` nell'esempio precedente).
* Verificare che il socket sia scrivibile da nginx (ad esempio, `chmod go+w /tmp/kestrel-test.sock` ). 

### <a name="port-0"></a>Porta 0

Quando si specifica il numero di porta `0`, Kestrel esegue l'associazione dinamica a una porta disponibile. L'esempio seguente indica come determinare la porta alla quale Kestrel ha eseguito l'associazione in fase di esecuzione:

[!code-csharp[](kestrel/samples/2.x/KestrelSample/Startup.cs?name=snippet_Configure&highlight=3-4,15-21)]

Quando si esegue l'app, l'output della finestra della console indica la porta dinamica in cui l'app può essere raggiunta:

```console
Listening on the following addresses: http://127.0.0.1:48508
```

### <a name="limitations"></a>Limitazioni

Configurare gli endpoint con i metodi seguenti:

* <xref:Microsoft.AspNetCore.Hosting.HostingAbstractionsWebHostBuilderExtensions.UseUrls*>
* L'argomento della riga di comando `--urls`
* La chiave di configurazione dell'host `urls`
* La variabile di ambiente `ASPNETCORE_URLS`

Questi metodi sono utili se si vuole che il codice funzioni con server diversi da Kestrel. Tenere presenti, tuttavia, le limitazioni seguenti:

* HTTPS non può essere usato con questi approcci, a meno che non venga specificato un certificato predefinito nella configurazione dell'endpoint HTTPS (ad esempio, usando la configurazione `KestrelServerOptions` o un file di configurazione come illustrato in precedenza in questo argomento).
* Quando sia l'approccio `Listen` che l'approccio `UseUrls` vengono usati contemporaneamente, gli endpoint `Listen` eseguono l'override degli endpoint `UseUrls`.

### <a name="iis-endpoint-configuration"></a>Configurazione dell'endpoint IIS

Quando si usa IIS, le associazioni di URL per le associazioni di override di IIS vengono impostate mediante `Listen` o `UseUrls`. Per altre informazioni, vedere l'articolo [Introduzione al modulo ASP.NET Core](xref:host-and-deploy/aspnet-core-module).

### <a name="listenoptionsprotocols"></a>ListenOptions.Protocols

La proprietà `Protocols` stabilisce i protocolli HTTP (`HttpProtocols`) abilitati in un endpoint di connessione o per il server. Assegnare un valore alla proprietà `Protocols` dall'enumerazione `HttpProtocols`.

| Valore di enumerazione `HttpProtocols` | Protocollo di connessione consentito |
| -------------------------- | ----------------------------- |
| `Http1`                    | Solo HTTP/1.1. Può essere usato con o senza TLS. |
| `Http2`                    | Solo HTTP/2. Può essere usato senza TLS solo se il client supporta una [modalità di conoscenza pregressa](https://tools.ietf.org/html/rfc7540#section-3.4). |
| `Http1AndHttp2`            | HTTP/1.1 e HTTP/2. HTTP/2 richiede una connessione TLS e [ALPN (Application-Layer Protocol negotiation)](https://tools.ietf.org/html/rfc7301#section-3) ; in caso contrario, il valore predefinito per la connessione è HTTP/1.1. |

Il protocollo predefinito è HTTP/1.1.

Restrizioni relative a TLS per HTTP/2:

* TLS versione 1.2 o successiva
* Rinegoziazione disabilitata
* Compressione disabilitata
* Dimensioni minime per lo scambio di chiavi temporanee:
  * Curva ellittica Diffie-Hellman (ECDHE) &lbrack; [RFC4492](https://www.ietf.org/rfc/rfc4492.txt) &rbrack; : minimo 224 bit
  * Diffie-Hellman campo finito (DHE) &lbrack; `TLS12` &rbrack; : minimo 2048 bit
* Pacchetto di crittografia non bloccato

`TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256`&lbrack;`TLS-ECDHE`&rbrack; con la curva ellittica P-256 &lbrack; `FIPS186` &rbrack; è supportata per impostazione predefinita.

Nell'esempio seguente sono consentite connessioni HTTP/1.1 e HTTP/2 sulla porta 8000. Le connessioni sono protette da TLS con un certificato incluso:

```csharp
.ConfigureKestrel((context, serverOptions) =>
{
    serverOptions.Listen(IPAddress.Any, 8000, listenOptions =>
    {
        listenOptions.Protocols = HttpProtocols.Http1AndHttp2;
        listenOptions.UseHttps("testCert.pfx", "testPassword");
    });
});
```

Facoltativamente, creare un'implementazione `IConnectionAdapter` per filtrare gli handshake TLS in base alla connessione in caso di crittografie specifiche:

```csharp
.ConfigureKestrel((context, serverOptions) =>
{
    serverOptions.Listen(IPAddress.Any, 8000, listenOptions =>
    {
        listenOptions.Protocols = HttpProtocols.Http1AndHttp2;
        listenOptions.UseHttps("testCert.pfx", "testPassword");
        listenOptions.ConnectionAdapters.Add(new TlsFilterAdapter());
    });
});
```

```csharp
private class TlsFilterAdapter : IConnectionAdapter
{
    public bool IsHttps => false;

    public Task<IAdaptedConnection> OnConnectionAsync(ConnectionAdapterContext context)
    {
        var tlsFeature = context.Features.Get<ITlsHandshakeFeature>();

        // Throw NotSupportedException for any cipher algorithm that the app doesn't
        // wish to support. Alternatively, define and compare
        // ITlsHandshakeFeature.CipherAlgorithm to a list of acceptable cipher
        // suites.
        //
        // No encryption is used with a CipherAlgorithmType.Null cipher algorithm.
        if (tlsFeature.CipherAlgorithm == CipherAlgorithmType.Null)
        {
            throw new NotSupportedException("Prohibited cipher: " + tlsFeature.CipherAlgorithm);
        }

        return Task.FromResult<IAdaptedConnection>(new AdaptedConnection(context.ConnectionStream));
    }

    private class AdaptedConnection : IAdaptedConnection
    {
        public AdaptedConnection(Stream adaptedStream)
        {
            ConnectionStream = adaptedStream;
        }

        public Stream ConnectionStream { get; }

        public void Dispose()
        {
        }
    }
}
```

*Impostare il protocollo dalla configurazione*

<xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder*> chiama `serverOptions.Configure(context.Configuration.GetSection("Kestrel"))` per impostazione predefinita per caricare la configurazione di Kestrel.

Nell'esempio seguente *appsettings.json* viene stabilito un protocollo di connessione predefinito (http/1.1 e http/2) per tutti gli endpoint di Gheppio:

```json
{
  "Kestrel": {
    "EndpointDefaults": {
      "Protocols": "Http1AndHttp2"
    }
  }
}
```

Il file di configurazione di esempio seguente stabilisce un protocollo di connessione per un endpoint specifico:

```json
{
  "Kestrel": {
    "Endpoints": {
      "HttpsDefaultCert": {
        "Url": "https://localhost:5001",
        "Protocols": "Http1AndHttp2"
      }
    }
  }
}
```

I protocolli specificati nei valori di override del codice sono impostati dalla configurazione.

## <a name="libuv-transport-configuration"></a>Configurazione del trasporto libuv

Con la versione ASP.NET Core 2.1, il trasporto predefinito di Kestrel non si basa più su Libuv, ma su socket gestiti. Si tratta di una modifica importante per le app ASP.NET 2.0 Core che vengono aggiornate alla versione 2.1, che chiamano <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderLibuvExtensions.UseLibuv*> e dipendono da uno dei pacchetti seguenti:

* [Microsoft.AspNetCore.Server.Kestrel](https://www.nuget.org/packages/Microsoft.AspNetCore.Server.Kestrel/) (riferimento diretto al pacchetto)
* [Microsoft.AspNetCore.App](https://www.nuget.org/packages/Microsoft.AspNetCore.App/)

Per i progetti che richiedono l'uso di libuv:

* Aggiungere una dipendenza per il pacchetto [Microsoft.AspNetCore.Server.Kestrel.Transport.Libuv](https://www.nuget.org/packages/Microsoft.AspNetCore.Server.Kestrel.Transport.Libuv/) al file di progetto dell'app:

  ```xml
  <PackageReference Include="Microsoft.AspNetCore.Server.Kestrel.Transport.Libuv"
                    Version="{VERSION}" />
  ```

* Chiamare <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderLibuvExtensions.UseLibuv*>:

  ```csharp
  public class Program
  {
      public static void Main(string[] args)
      {
          CreateWebHostBuilder(args).Build().Run();
      }

      public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
          WebHost.CreateDefaultBuilder(args)
              .UseLibuv()
              .UseStartup<Startup>();
  }
  ```

### <a name="url-prefixes"></a>Prefissi URL

Se si usa `UseUrls`, l'argomento della riga di comando `--urls`, la chiave di configurazione dell'host `urls` o la variabile di ambiente `ASPNETCORE_URLS`, i prefissi URL possono avere uno dei formati seguenti.

Sono validi solo i prefissi URL HTTP. Kestrel non supporta HTTPS quando la configurazione delle associazioni URL viene eseguita con `UseUrls`.

* Indirizzo IPv4 con numero di porta

  ```
  http://65.55.39.10:80/
  ```

  `0.0.0.0` è un caso speciale che esegue l'associazione a tutti gli indirizzi IPv4.

* Indirizzo IPv6 con numero di porta

  ```
  http://[0:0:0:0:0:ffff:4137:270a]:80/
  ```

  `[::]` è l'equivalente IPv6 di `0.0.0.0` per IPv4.

* Nome host con numero di porta

  ```
  http://contoso.com:80/
  http://*:80/
  ```

  I nomi host, `*` e `+` non sono casi particolari. Tutto ciò che non è riconosciuto come un indirizzo IP o un elemento `localhost` valido esegue l'associazione a tutti gli IP IPv4 e IPv6. Per associare nomi host diversi ad app ASP.NET Core diverse sulla stessa porta, usare [HTTP.sys](xref:fundamentals/servers/httpsys) o un server proxy inverso come IIS, Nginx o Apache.

  > [!WARNING]
  > Una configurazione che prevede un proxy inverso richiede il [filtro host](#host-filtering).

* Nome host `localhost` con numero di porta o IP di loopback con numero di porta

  ```
  http://localhost:5000/
  http://127.0.0.1:5000/
  http://[::1]:5000/
  ```

  Se è specificato `localhost`, Kestrel tenta l'associazione alle interfacce di loopback sia IPv4 che IPv6. Se la porta richiesta è in uso da parte di un altro servizio in una delle due interfacce di loopback, Kestrel non viene avviato. Se una delle due interfacce di loopback non è disponibile per qualsiasi altro motivo (in genere perché IPv6 non è supportato), Kestrel registra un avviso.

## <a name="host-filtering"></a>Filtro host

Mentre supporta la configurazione in base ai prefissi, ad esempio `http://example.com:5000`, Kestrel ignora quasi sempre il nome host. L'host `localhost` è un caso speciale usato per l'associazione agli indirizzi di loopback. Qualsiasi host che non sia un indirizzo IP esplicito esegue l'associazione a tutti gli indirizzi IP pubblici. Le intestazioni `Host` non vengono convalidate.

Come soluzione alternativa, usare il middleware di filtro host. Il middleware di filtro host viene fornito dal pacchetto [Microsoft. AspNetCore. HostFiltering](https://www.nuget.org/packages/Microsoft.AspNetCore.HostFiltering) , incluso nel [metapacchetto Microsoft. AspNetCore. App](xref:fundamentals/metapackage-app) (ASP.NET Core 2,1 o 2,2). Il middleware viene aggiunto da <xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder*>, che chiama <xref:Microsoft.AspNetCore.Builder.HostFilteringServicesExtensions.AddHostFiltering*>:

[!code-csharp[](kestrel/samples-snapshot/2.x/KestrelSample/Program.cs?name=snippet_Program&highlight=9)]

Per impostazione predefinita, il middleware di filtro host è disabilitato per impostazione predefinita. Per abilitare il middleware, definire una `AllowedHosts` chiave in *appsettings.json* / *appSettings. \<EnvironmentName> JSON*. Il valore è un elenco con valori delimitati da punto e virgola di nomi host senza numeri di porta:

*appsettings.json*:

```json
{
  "AllowedHosts": "example.com;localhost"
}
```

> [!NOTE]
> Il [middleware delle intestazioni inoltrate](xref:host-and-deploy/proxy-load-balancer) include anche un'opzione <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.AllowedHosts>. Il middleware di intestazioni inoltrate e il middleware di filtro host hanno funzionalità simili per diversi scenari. L'impostazione di `AllowedHosts` con il middleware delle intestazioni inoltrate è appropriato se l'intestazione `Host` non viene mantenuta durante l'inoltro delle richieste con un server proxy inverso o il bilanciamento del carico. L'impostazione di `AllowedHosts` con il middleware del filtro host è appropriata se si usa Kestrel come server perimetrale rivolto al pubblico o se l'intestazione `Host` viene inoltrata direttamente.
>
> Per altre informazioni sul middleware delle intestazioni inoltrate, vedere <xref:host-and-deploy/proxy-load-balancer>.

## <a name="http11-request-draining"></a>Svuotamento richieste HTTP/1.1

L'apertura di connessioni HTTP richiede molto tempo. Per HTTPS, è anche a elevato utilizzo di risorse. Pertanto, gheppio tenta di riutilizzare le connessioni per il protocollo HTTP/1.1. Il corpo della richiesta deve essere completamente utilizzato per consentire il riutilizzo della connessione. L'app non usa sempre il corpo della richiesta, ad esempio le `POST` richieste in cui il server restituisce una risposta di reindirizzamento o 404. Nel `POST` caso-Reindirizzamento:

* Il client potrebbe avere già inviato parte dei `POST` dati.
* Il server scrive la risposta 301.
* Non è possibile usare la connessione per una nuova richiesta fino a quando i `POST` dati del corpo della richiesta precedente non sono stati completamente letti.
* Gheppio tenta di svuotare il corpo della richiesta. Lo svuotamento del corpo della richiesta implica la lettura e l'eliminazione dei dati senza elaborarli.

Il processo di svuotamento crea un tradoff tra la possibilità di riutilizzare la connessione e il tempo necessario per svuotare i dati rimanenti:

* Lo svuotamento ha un timeout di cinque secondi, che non è configurabile.
* Se tutti i dati specificati dall' `Content-Length` `Transfer-Encoding` intestazione o non sono stati letti prima del timeout, la connessione viene chiusa.

In alcuni casi può essere necessario terminare immediatamente la richiesta, prima o dopo la scrittura della risposta. È ad esempio possibile che i client dispongano di protezioni dati restrittive, pertanto la limitazione dei dati caricati potrebbe avere una priorità. In questi casi, per terminare una richiesta, chiamare [HttpContext. Abort](xref:Microsoft.AspNetCore.Http.HttpContext.Abort%2A) da un controller, una Razor pagina o un middleware.

Ci sono alcune avvertenze per chiamare `Abort` :

* La creazione di nuove connessioni può essere lenta e dispendiosa.
* Non vi è alcuna garanzia che il client abbia letto la risposta prima che la connessione venga chiusa.
* `Abort`La chiamata a deve essere rara e riservata per i casi di errore gravi, non per errori comuni.
  * Chiamare solo `Abort` quando è necessario risolvere un problema specifico. Ad esempio, chiamare `Abort` se i client malintenzionati tentano di eseguire `POST` dati o quando si verifica un bug nel codice client che genera richieste di grandi dimensioni o numerose.
  * Non chiamare le `Abort` situazioni di errore comuni, ad esempio HTTP 404 (non trovato).

Quando si chiama [HttpResponse. CompleteAsync](xref:Microsoft.AspNetCore.Http.HttpResponse.CompleteAsync%2A) , prima `Abort` di chiamare si garantisce che il server abbia completato la scrittura della risposta. Tuttavia, il comportamento del client non è prevedibile e potrebbe non essere in grado di leggere la risposta prima che la connessione venga interrotta.

Questo processo è diverso per HTTP/2 poiché il protocollo supporta l'interruzione dei singoli flussi di richiesta senza chiudere la connessione. Il timeout di svuotamento di cinque secondi non è applicabile. Se dopo il completamento di una risposta sono presenti dati del corpo della richiesta non letti, il server invia un frame RST HTTP/2. I frame di dati del corpo della richiesta aggiuntivi vengono ignorati.

Se possibile, è preferibile che i client utilizzino l'intestazione della richiesta [expect: 100-continue](https://developer.mozilla.org/docs/Web/HTTP/Status/100) e attendano la risposta del server prima di iniziare a inviare il corpo della richiesta. Che fornisce al client la possibilità di esaminare la risposta e interrompere prima di inviare dati non necessari.

## <a name="additional-resources"></a>Risorse aggiuntive

* Quando si usano i socket UNIX in Linux, il socket non viene eliminato automaticamente all'arresto dell'app. Per altre informazioni, vedere [questo problema di GitHub](https://github.com/dotnet/aspnetcore/issues/14134).
* <xref:test/troubleshoot>
* <xref:security/enforcing-ssl>
* <xref:host-and-deploy/proxy-load-balancer>
* [RFC 7230: Message Syntax and Routing (Section 5.4: Host)](https://tools.ietf.org/html/rfc7230#section-5.4) (RFC 7230: Sintassi e routing dei messaggi (sezione 5.4: Host))

::: moniker-end

::: moniker range="= aspnetcore-2.1"

Kestrel è un [server Web per ASP.NET Core](xref:fundamentals/servers/index) multipiattaforma. Kestrel è il server Web incluso per impostazione predefinita nei modelli di progetto di ASP.NET Core.

Kestrel supporta gli scenari seguenti:

* HTTPS
* Aggiornamento opaco usato per abilitare [WebSocket](https://github.com/aspnet/websockets)
* Socket Unix ad alte prestazioni dietro Nginx

Kestrel è supportato in tutte le piattaforme e le versioni supportate da .NET Core.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/servers/kestrel/samples/2.x) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="when-to-use-kestrel-with-a-reverse-proxy"></a>Quando usare Kestrel con un proxy inverso

È possibile usare Kestrel da solo o in combinazione con un *server proxy inverso*, ad esempio [Internet Information Services (IIS)](https://www.iis.net/), [Nginx](https://nginx.org) o [Apache](https://httpd.apache.org/). Il server proxy inverso riceve le richieste HTTP dalla rete e le inoltra a Kestrel.

Kestrel usato come server Web perimetrale (esposto a Internet):

![Kestrel comunica direttamente con Internet senza un server proxy inverso](kestrel/_static/kestrel-to-internet2.png)

Kestrel usato in una configurazione proxy inverso:

![Kestrel comunica indirettamente con Internet attraverso un server proxy inverso, ad esempio IIS, Nginx o Apache](kestrel/_static/kestrel-to-internet.png)

La configurazione, con o senza un server proxy inverso, è una configurazione di hosting supportata.

Se viene utilizzato come server perimetrale in assenza di un server proxy inverso, Kestrel non supporta la condivisione tra più processi dell'IP e della porta. Quando è configurato per restare in ascolto su una porta, Kestrel gestisce tutto il traffico per tale porta indipendentemente dalle intestazioni `Host` delle richieste. Un proxy inverso che può condividere le porte può eseguire l'inoltro delle richieste a Kestrel su un unico IP e un'unica porta.

Anche se la presenza di un server proxy inverso non è necessaria, può risultare utile per i motivi seguenti.

Un proxy inverso:

* Può limitare l'area della superficie di attacco pubblica esposta delle app ospitate.
* Offre un livello aggiuntivo di configurazione e protezione.
* Potrebbe offrire un'integrazione migliore con l'infrastruttura esistente.
* Semplifica il bilanciamento del carico e la configurazione di comunicazioni protette (HTTPS). Solo il server proxy inverso richiede un certificato X. 509 e il server è in grado di comunicare con i server dell'app nella rete interna tramite HTTP normale.

> [!WARNING]
> Una configurazione che prevede un proxy inverso richiede il [filtro host](#host-filtering).

## <a name="how-to-use-kestrel-in-aspnet-core-apps"></a>Come usare Kestrel nelle app ASP.NET Core

Il pacchetto [Microsoft. AspNetCore. Server. gheppio](https://www.nuget.org/packages/Microsoft.AspNetCore.Server.Kestrel/) è incluso nel [metapacchetto Microsoft. AspNetCore. app](xref:fundamentals/metapackage-app).

I modelli di progetto ASP.NET Core usano Kestrel per impostazione predefinita. In *Program.cs* il codice del modello chiama <xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder*>, che chiama <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderKestrelExtensions.UseKestrel*> in background.

Per fornire una configurazione aggiuntiva dopo la chiamata di `CreateDefaultBuilder`, chiamare <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderKestrelExtensions.UseKestrel*>:

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .UseKestrel(serverOptions =>
        {
            // Set properties and call methods on serverOptions
        });
```

Per ulteriori informazioni su `CreateDefaultBuilder` e sulla compilazione dell'host, vedere la sezione *configurare un host* di <xref:fundamentals/host/web-host#set-up-a-host> .

## <a name="kestrel-options"></a>Opzioni Kestrel

Il server Web Kestrel dispone di opzioni di configurazione dei vincoli che risultano particolarmente utili nelle distribuzioni con connessione Internet.

Impostare i vincoli per la proprietà <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Limits> della classe <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions>. La proprietà `Limits` contiene un'istanza della classe <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits>.

Negli esempi seguenti viene usato lo spazio dei nomi <xref:Microsoft.AspNetCore.Server.Kestrel.Core>:

```csharp
using Microsoft.AspNetCore.Server.Kestrel.Core;
```

Le opzioni gheppio, che sono configurate nel codice C# negli esempi seguenti, possono essere impostate anche usando un [provider di configurazione](xref:fundamentals/configuration/index). Il provider di configurazione file, ad esempio, può caricare la configurazione di Gheppio da un oggetto *appsettings.json* o *appSettings. { File Environment}. JSON* :

```json
{
  "Kestrel": {
    "Limits": {
      "MaxConcurrentConnections": 100,
      "MaxConcurrentUpgradedConnections": 100
    }
  }
}
```

Usare **uno** degli approcci seguenti:

* Configurare gheppio in `Startup.ConfigureServices` :

  1. Inserire un'istanza di `IConfiguration` nella `Startup` classe. Nell'esempio seguente si presuppone che la configurazione inserita venga assegnata alla `Configuration` Proprietà.
  2. In `Startup.ConfigureServices` caricare la `Kestrel` sezione della configurazione nella configurazione di Gheppio:

     ```csharp
     using Microsoft.Extensions.Configuration
     
     public class Startup
     {
         public Startup(IConfiguration configuration)
         {
             Configuration = configuration;
         }

         public IConfiguration Configuration { get; }

         public void ConfigureServices(IServiceCollection services)
         {
             services.Configure<KestrelServerOptions>(
                 Configuration.GetSection("Kestrel"));
         }

         public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
         {
             ...
         }
     }
     ```

* Configurare il gheppio durante la compilazione dell'host:

  In *Program.cs* caricare la `Kestrel` sezione della configurazione nella configurazione di Gheppio:

  ```csharp
  // using Microsoft.Extensions.DependencyInjection;

  public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
      WebHost.CreateDefaultBuilder(args)
          .ConfigureServices((context, services) =>
          {
              services.Configure<KestrelServerOptions>(
                  context.Configuration.GetSection("Kestrel"));
          })
          .UseStartup<Startup>();
  ```

Entrambi gli approcci precedenti funzionano con qualsiasi [provider di configurazione](xref:fundamentals/configuration/index).

### <a name="keep-alive-timeout"></a>Timeout keep-alive

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.KeepAliveTimeout>

Ottiene o imposta il [timeout keep-alive](https://tools.ietf.org/html/rfc7230#section-6.5). Il valore predefinito è 2 minuti.

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .UseKestrel(serverOptions =>
        {
            serverOptions.Limits.KeepAliveTimeout = TimeSpan.FromMinutes(2);
        });
```

### <a name="maximum-client-connections"></a>Numero massimo di connessioni client

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.MaxConcurrentConnections>
<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.MaxConcurrentUpgradedConnections>

È possibile impostare il numero massimo di connessioni TCP aperte simultaneamente per l'intera app con il codice seguente:

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .UseKestrel(serverOptions =>
        {
            serverOptions.Limits.MaxConcurrentConnections = 100;
        });
```

È previsto un limite separato per le connessioni che sono state aggiornate da HTTP o HTTPS a un altro protocollo (ad esempio su una richiesta WebSocket). Dopo l'aggiornamento di una connessione, questa non viene conteggiata per il limite `MaxConcurrentConnections`.

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .UseKestrel(serverOptions =>
        {
            serverOptions.Limits.MaxConcurrentUpgradedConnections = 100;
        });
```

Per impostazione predefinita, il numero massimo di connessioni è illimitato (null).

### <a name="maximum-request-body-size"></a>Dimensione massima del corpo della richiesta

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.MaxRequestBodySize>

La dimensione massima predefinita del corpo della richiesta è pari a 30.000.000 di byte, equivalenti a circa 28,6 MB.

Il metodo consigliato per ignorare il limite in un'app ASP.NET Core MVC è l'uso dell'attributo <xref:Microsoft.AspNetCore.Mvc.RequestSizeLimitAttribute> in un metodo di azione:

```csharp
[RequestSizeLimit(100000000)]
public IActionResult MyActionMethod()
```

L'esempio seguente illustra come configurare il vincolo per l'app in ogni richiesta:

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .UseKestrel(serverOptions =>
        {
            serverOptions.Limits.MaxRequestBodySize = 10 * 1024;
        });
```

Eseguire l'override dell'impostazione per una richiesta specifica nel middleware:

[!code-csharp[](kestrel/samples/2.x/KestrelSample/Startup.cs?name=snippet_Limits&highlight=3-4)]

Viene generata un'eccezione se l'app configura il limite per una richiesta dopo che l'app ha iniziato a leggere la richiesta. Una proprietà `IsReadOnly` indica se la proprietà `MaxRequestBodySize` è in stato di sola lettura e pertanto è troppo tardi per configurare il limite.

Quando un'app viene eseguita [out-of-process](xref:host-and-deploy/iis/index#out-of-process-hosting-model) dietro il [modulo ASP.NET Core](xref:host-and-deploy/aspnet-core-module), il limite della dimensione del corpo della richiesta di Kestrel è disabilitato perché viene già impostato da IIS.

### <a name="minimum-request-body-data-rate"></a>Velocità minima dei dati del corpo della richiesta

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.MinRequestBodyDataRate>
<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.MinResponseDataRate>

Kestrel controlla ogni secondo se i dati arrivano alla velocità in byte al secondo specificata. Se la frequenza scende sotto il valore minimo, si è verificato il timeout della connessione. Il periodo di tolleranza è la quantità di tempo che il gheppio concede al client di aumentare la velocità di invio fino al minimo. la frequenza non viene controllata durante tale periodo di tempo. Il periodo di prova consente di evitare l'interruzione di connessioni che inizialmente inviano i dati a velocità ridotta a causa dell'avvio lento del protocollo TCP.

La velocità minima predefinita è di 240 byte al secondo, con un periodo di tolleranza di 5 secondi.

Anche per la risposta è prevista una velocità minima. Il codice per impostare il limite della richiesta e della risposta è identico e varia solo per `RequestBody` o `Response` nei nomi di proprietà e di interfaccia.

L'esempio seguente visualizza come configurare la velocità minima dei dati in *Program.cs*:

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .UseKestrel(serverOptions =>
        {
            serverOptions.Limits.MinRequestBodyDataRate =
                new MinDataRate(bytesPerSecond: 100, gracePeriod: TimeSpan.FromSeconds(10));
            serverOptions.Limits.MinResponseDataRate =
                new MinDataRate(bytesPerSecond: 100, gracePeriod: TimeSpan.FromSeconds(10));
        });
```

### <a name="request-headers-timeout"></a>Timeout delle intestazioni delle richieste

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.RequestHeadersTimeout>

Ottiene o imposta la quantità massima di tempo che il server dedica alla ricezione delle intestazioni delle richieste. Il valore predefinito è 30 secondi.

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .UseKestrel(serverOptions =>
        {
            serverOptions.Limits.RequestHeadersTimeout = TimeSpan.FromMinutes(1);
        });
```

### <a name="synchronous-io"></a>I/O sincrono

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.AllowSynchronousIO> Controlla se è consentito l'I/O sincrono per la richiesta e la risposta. Il valore predefinito è `true`.

> [!WARNING]
> Un numero elevato di operazioni di I/O sincrone bloccanti può causare l'esaurimento del pool di thread, il che rende l'app non risponde. Abilitare solo `AllowSynchronousIO` quando si usa una libreria che non supporta l'I/O asincrono.

Nell'esempio seguente viene disabilitato l'I/O sincrono:

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .UseKestrel(serverOptions =>
        {
            serverOptions.AllowSynchronousIO = false;
        });
```

Per informazioni su altre opzioni e limiti di Kestrel, vedere:

* <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions>
* <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits>
* <xref:Microsoft.AspNetCore.Server.Kestrel.Core.ListenOptions>

## <a name="endpoint-configuration"></a>Configurazione dell'endpoint

Per impostazione predefinita, ASP.NET Core è associato a:

* `http://localhost:5000`
* `https://localhost:5001` (quando è presente un certificato di sviluppo locale)

Specificare gli URL usando gli elementi seguenti:

* La variabile di ambiente `ASPNETCORE_URLS`.
* L'argomento della riga di comando `--urls`.
* La chiave di configurazione dell'host `urls`.
* Il metodo di estensione `UseUrls`.

Il valore specificato usando i metodi seguenti può essere uno o più endpoint HTTP e HTTPS (HTTPS se è disponibile un certificato predefinito). Configurare il valore come un elenco delimitato da punto e virgola (ad esempio, `"Urls": "http://localhost:8000;http://localhost:8001"`).

Per altre informazioni su questi approcci, vedere [URL del server](xref:fundamentals/host/web-host#server-urls) e [Override della configurazione](xref:fundamentals/host/web-host#override-configuration).

Viene creato un certificato di sviluppo nei casi seguenti:

* Quando viene installato [.NET Core SDK](/dotnet/core/sdk).
* Quando per creare un certificato viene usato [lo strumento dev-certs](xref:aspnetcore-2.1#https).

Alcuni browser richiedono la concessione di autorizzazioni esplicite per considerare attendibile il certificato di sviluppo locale.

I modelli di progetto consentono di configurare le app per l'esecuzione su HTTPS per impostazione predefinita e includono il [reindirizzamento HTTPS e il supporto HSTS](xref:security/enforcing-ssl).

Chiamare i metodi <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Listen*> oppure <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ListenUnixSocket*> su <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions> per configurare le porte e i prefissi URL per Kestrel.

Anche `UseUrls`, l'argomento della riga di comando `--urls`, la chiave di configurazione dell'host `urls` e la variabile di ambiente `ASPNETCORE_URLS` funzionano, ma con le limitazioni indicate più avanti nella sezione (deve essere disponibile un certificato predefinito per la configurazione dell'endopoint HTTPS).

`KestrelServerOptions` configurazione

### <a name="configureendpointdefaultsactionlistenoptions"></a>ConfigureEndpointDefaults (azione \<ListenOptions> )

Specifica un elemento di configurazione `Action` da eseguire per ogni endpoint specificato. Se si chiama `ConfigureEndpointDefaults` più volte, gli elementi `Action` precedenti vengono sostituiti con l'ultimo elemento `Action` specificato.

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .ConfigureKestrel((context, serverOptions) =>
        {
            serverOptions.ConfigureEndpointDefaults(listenOptions =>
            {
                // Configure endpoint defaults
            });
        });
```

> [!NOTE]
> Per gli endpoint creati chiamando <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Listen*> **prima** della chiamata a <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ConfigureEndpointDefaults*> non verranno applicati i valori predefiniti.

### <a name="configurehttpsdefaultsactionhttpsconnectionadapteroptions"></a>ConfigureHttpsDefaults (azione \<HttpsConnectionAdapterOptions> )

Specifica un elemento di configurazione `Action` da eseguire per ogni endpoint HTTPS. Se si chiama `ConfigureHttpsDefaults` più volte, gli elementi `Action` precedenti vengono sostituiti con l'ultimo elemento `Action` specificato.

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .ConfigureKestrel((context, serverOptions) =>
        {
            serverOptions.ConfigureHttpsDefaults(listenOptions =>
            {
                // certificate is an X509Certificate2
                listenOptions.ServerCertificate = certificate;
            });
        });
```

> [!NOTE]
> Per gli endpoint creati chiamando <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Listen*> **prima** della chiamata a <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ConfigureHttpsDefaults*> non verranno applicati i valori predefiniti.

### <a name="configureiconfiguration"></a>Configure(IConfiguration)

Crea un loader di configurazione per la configurazione di Kestrel che accetta un elemento <xref:Microsoft.Extensions.Configuration.IConfiguration> come input. L'ambito della configurazione deve essere la sezione di configurazione per Kestrel.

### <a name="listenoptionsusehttps"></a>ListenOptions.UseHttps

Configura Kestrel per l'uso di HTTPS.

Estensioni `ListenOptions.UseHttps`:

* `UseHttps`: Configurare gheppio per l'uso di HTTPS con il certificato predefinito. Genera un'eccezione se non è stato configurato alcun certificato predefinito.
* `UseHttps(string fileName)`
* `UseHttps(string fileName, string password)`
* `UseHttps(string fileName, string password, Action<HttpsConnectionAdapterOptions> configureOptions)`
* `UseHttps(StoreName storeName, string subject)`
* `UseHttps(StoreName storeName, string subject, bool allowInvalid)`
* `UseHttps(StoreName storeName, string subject, bool allowInvalid, StoreLocation location)`
* `UseHttps(StoreName storeName, string subject, bool allowInvalid, StoreLocation location, Action<HttpsConnectionAdapterOptions> configureOptions)`
* `UseHttps(X509Certificate2 serverCertificate)`
* `UseHttps(X509Certificate2 serverCertificate, Action<HttpsConnectionAdapterOptions> configureOptions)`
* `UseHttps(Action<HttpsConnectionAdapterOptions> configureOptions)`

Parametri `ListenOptions.UseHttps`:

* `filename` è il percorso e il nome file di un file di certificato, relativo alla directory che contiene i file di contenuto dell'app.
* `password` è la password richiesta per accedere ai dati del certificato X.509.
* `configureOptions` è un elemento `Action` per configurare `HttpsConnectionAdapterOptions`. Restituisce `ListenOptions`.
* `storeName` è l'archivio certificati da cui caricare il certificato.
* `subject` è il nome dell'oggetto del certificato.
* `allowInvalid` indica se devono essere considerati i certificati non validi, come ad esempio i certificati autofirmati.
* `location` è il percorso dell'archivio da cui caricare il certificato.
* `serverCertificate` è il certificato X.509.

Nell'ambiente di produzione, HTTPS deve essere configurato in modo esplicito. Come minimo, è necessario specificare un certificato predefinito.

Configurazioni supportate descritte in seguito:

* Nessuna configurazione
* Sostituire il certificato predefinito della configurazione
* Modificare le impostazioni predefinite nel codice

*Nessuna configurazione*

Kestrel è in ascolto su `http://localhost:5000` e `https://localhost:5001` (se è disponibile un certificato predefinito).

<a name="configuration"></a>

*Sostituire il certificato predefinito della configurazione*

`CreateDefaultBuilder` chiama `Configure(context.Configuration.GetSection("Kestrel"))` per impostazione predefinita per caricare la configurazione di Kestrel. È disponibile per Kestrel uno schema di configurazione delle impostazioni delle app HTTPS predefinito. Configurare più endpoint, inclusi gli URL e i certificati da usare, da un file su disco o da un archivio certificati.

Nell'esempio seguente *appsettings.json* :

* Impostare **AllowInvalid** su `true` per consentire l'uso di certificati non validi, come ad esempio i certificati autofirmati.
* Tutti gli endpoint HTTPS che non specificano un certificato (**HttpsDefaultCert** nell'esempio che segue) usano il certificato definito in **Certificates** > **Default** o il certificato di sviluppo.

```json
{
  "Kestrel": {
    "Endpoints": {
      "Http": {
        "Url": "http://localhost:5000"
      },

      "HttpsInlineCertFile": {
        "Url": "https://localhost:5001",
        "Certificate": {
          "Path": "<path to .pfx file>",
          "Password": "<certificate password>"
        }
      },

      "HttpsInlineCertStore": {
        "Url": "https://localhost:5002",
        "Certificate": {
          "Subject": "<subject; required>",
          "Store": "<certificate store; required>",
          "Location": "<location; defaults to CurrentUser>",
          "AllowInvalid": "<true or false; defaults to false>"
        }
      },

      "HttpsDefaultCert": {
        "Url": "https://localhost:5003"
      },

      "Https": {
        "Url": "https://*:5004",
        "Certificate": {
          "Path": "<path to .pfx file>",
          "Password": "<certificate password>"
        }
      }
    },
    "Certificates": {
      "Default": {
        "Path": "<path to .pfx file>",
        "Password": "<certificate password>"
      }
    }
  }
}
```

Un'alternativa all'uso di **Path** e **Password** per qualsiasi nodo del certificato consiste nello specificare il certificato usando i campi dell'archivio certificati. Ad esempio, il certificato predefinito **Certificates**  >   può essere specificato come:

```json
"Default": {
  "Subject": "<subject; required>",
  "Store": "<cert store; required>",
  "Location": "<location; defaults to CurrentUser>",
  "AllowInvalid": "<true or false; defaults to false>"
}
```

Note di schema:

* I nomi degli endpoint non applicano la distinzione tra maiuscole e minuscole. Ad esempio, sono validi sia `HTTPS` che `Https`.
* Il parametro `Url` è obbligatorio per ogni endpoint. Il formato per questo parametro è uguale a quello del parametro di configurazione di primo livello `Urls`, con la differenza che si limita a un singolo valore.
* Questi endpoint sostituiscono quelli definiti nella configurazione di primo livello `Urls`, non vi si aggiungono. Gli endpoint definiti nel codice tramite `Listen` si aggiungono agli endpoint definiti nella sezione di configurazione.
* La sezione `Certificate` è facoltativa. Se la sezione `Certificate` non è specificata, vengono usati i valori predefiniti definiti negli scenari precedenti. Se non è disponibile alcun valore predefinito, il server genera un'eccezione e non viene avviato.
* La `Certificate` sezione supporta sia la password del **percorso** &ndash;  che i certificati dell'archivio dell' **oggetto** &ndash;  .
* In questo modo è possibile definire un numero qualsiasi di endpoint, purché non provochino conflitti di porte.
* `options.Configure(context.Configuration.GetSection("{SECTION}"))` restituisce un oggetto `KestrelConfigurationLoader` con un metodo `.Endpoint(string name, listenOptions => { })` che può essere usato per integrare le impostazioni dell'endpoint configurato:

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .UseKestrel((context, serverOptions) =>
        {
            serverOptions.Configure(context.Configuration.GetSection("Kestrel"))
                .Endpoint("HTTPS", listenOptions =>
                {
                    listenOptions.HttpsOptions.SslProtocols = SslProtocols.Tls12;
                });
        });
```

`KestrelServerOptions.ConfigurationLoader` è possibile accedere direttamente a per continuare a scorrere sul caricatore esistente, ad esempio quello fornito da <xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder*> .

* La sezione di configurazione per ogni endpoint è disponibile nelle opzioni del `Endpoint` metodo in modo che sia possibile leggere le impostazioni personalizzate.
* È possibile caricare più configurazioni chiamando ancora `options.Configure(context.Configuration.GetSection("{SECTION}"))` con un'altra sezione. Viene usata solo l'ultima configurazione, a meno che `Load` sia stato chiamato esplicitamente in istanze precedenti. Il metapacchetto non chiama `Load` in modo che la sezione di configurazione predefinita venga sostituita.
* `KestrelConfigurationLoader` riflette la famiglia di API `Listen` da `KestrelServerOptions` all'overload di `Endpoint`, in modo che gli endpoint di codice e di configurazione possano essere configurati nella stessa posizione. Questi overload non usano nomi e usano solo le impostazioni predefinite della configurazione.

*Modificare le impostazioni predefinite nel codice*

`ConfigureEndpointDefaults` e `ConfigureHttpsDefaults` possono essere usati per modificare le impostazioni predefinite per `ListenOptions` e `HttpsConnectionAdapterOptions`, inclusa l'esecuzione dell'override del certificato predefinito specificato nello scenario precedente. `ConfigureEndpointDefaults` e `ConfigureHttpsDefaults` devono essere chiamati prima che venga configurato qualsiasi endpoint.

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .UseKestrel((context, serverOptions) =>
        {
            serverOptions.ConfigureEndpointDefaults(listenOptions =>
            {
                // Configure endpoint defaults
            });
            
            serverOptions.ConfigureHttpsDefaults(listenOptions =>
            {
                listenOptions.SslProtocols = SslProtocols.Tls12;
            });
        });
```

*Supporto kestrel per SNI*

[L'Indicazione nome server (SNI)](https://tools.ietf.org/html/rfc6066#section-3) può essere usata per ospitare più domini sulla stessa porta e indirizzo IP. Perché SNI funzioni è necessario che il client invii al server il nome host per la sessione protetta durante l'handshake TLS in modo che il server possa specificare il certificato corretto. Il client usa il certificato specificato per la comunicazione crittografata con il server durante la sessione protetta che segue l'handshake TLS.

Kestrel supporta SNI tramite il callback `ServerCertificateSelector`. Il callback viene richiamato una volta per ogni connessione per consentire all'app di controllare il nome host e selezionare il certificato appropriato.

Il supporto SNI richiede:

* In esecuzione su Framework di destinazione `netcoreapp2.1` o versione successiva. In `net461` o versioni successive, il callback viene richiamato, ma `name` è sempre `null` . L'elemento `name` è `null` anche se il client non specifica il parametro del nome host nell'handshake TLS.
* Esecuzione di tutti i siti Web nella stessa istanza di Kestrel. Kestrel supporta la condivisione di un indirizzo IP e di una porta tra più istanze solo con un proxy inverso.

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .UseKestrel((context, serverOptions) =>
        {
            serverOptions.ListenAnyIP(5005, listenOptions =>
            {
                listenOptions.UseHttps(httpsOptions =>
                {
                    var localhostCert = CertificateLoader.LoadFromStoreCert(
                        "localhost", "My", StoreLocation.CurrentUser,
                        allowInvalid: true);
                    var exampleCert = CertificateLoader.LoadFromStoreCert(
                        "example.com", "My", StoreLocation.CurrentUser,
                        allowInvalid: true);
                    var subExampleCert = CertificateLoader.LoadFromStoreCert(
                        "sub.example.com", "My", StoreLocation.CurrentUser,
                        allowInvalid: true);
                    var certs = new Dictionary<string, X509Certificate2>(
                        StringComparer.OrdinalIgnoreCase);
                    certs["localhost"] = localhostCert;
                    certs["example.com"] = exampleCert;
                    certs["sub.example.com"] = subExampleCert;

                    httpsOptions.ServerCertificateSelector = (connectionContext, name) =>
                    {
                        if (name != null && certs.TryGetValue(name, out var cert))
                        {
                            return cert;
                        }

                        return exampleCert;
                    };
                });
            });
        })
        .Build();
```

### <a name="connection-logging"></a>Registrazione connessione

Chiamare <xref:Microsoft.AspNetCore.Hosting.ListenOptionsConnectionLoggingExtensions.UseConnectionLogging*> per creare log a livello di debug per la comunicazione a livello di byte in una connessione. La registrazione delle connessioni è utile per la risoluzione dei problemi nella comunicazione di basso livello, ad esempio durante la crittografia TLS e dietro i proxy. Se `UseConnectionLogging` viene inserito prima `UseHttps` , viene registrato il traffico crittografato. Se `UseConnectionLogging` viene inserito dopo `UseHttps` , viene registrato il traffico decrittografato.

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.Listen(IPAddress.Any, 8000, listenOptions =>
    {
        listenOptions.UseConnectionLogging();
    });
});
```

### <a name="bind-to-a-tcp-socket"></a>Associazione a un socket TCP

Il metodo <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Listen*> esegue l'associazione a un socket TCP e un'espressione lambda per le opzioni consente di configurare un certificato X.509:

```csharp
public static void Main(string[] args)
{
    CreateWebHostBuilder(args).Build().Run();
}

public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .UseKestrel(serverOptions =>
        {
            serverOptions.Listen(IPAddress.Loopback, 5000);
            serverOptions.Listen(IPAddress.Loopback, 5001, listenOptions =>
            {
                listenOptions.UseHttps("testCert.pfx", "testPassword");
            });
        });
```

```csharp
public static void Main(string[] args)
{
    CreateWebHostBuilder(args).Build().Run();
}

public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .UseKestrel(serverOptions =>
        {
            serverOptions.Listen(IPAddress.Loopback, 5000);
            serverOptions.Listen(IPAddress.Loopback, 5001, listenOptions =>
            {
                listenOptions.UseHttps("testCert.pfx", "testPassword");
            });
        });
```

L'esempio configura HTTPS per un endpoint con <xref:Microsoft.AspNetCore.Server.Kestrel.Core.ListenOptions>. Usare la stessa API per configurare altre impostazioni di Kestrel per endpoint specifici.

[!INCLUDE [How to make an X.509 cert](~/includes/make-x509-cert.md)]

### <a name="bind-to-a-unix-socket"></a>Associazione a un socket Unix

Impostare l'ascolto su un socket Unix con <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ListenUnixSocket*> per migliorare le prestazioni con Nginx, come visualizzato nell'esempio seguente:

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .UseKestrel(serverOptions =>
        {
            serverOptions.ListenUnixSocket("/tmp/kestrel-test.sock");
            serverOptions.ListenUnixSocket("/tmp/kestrel-test.sock", listenOptions =>
            {
                listenOptions.UseHttps("testCert.pfx", "testpassword");
            });
        });
```

* Nel file nginx confiuguration impostare la `server`  >  `location`  >  `proxy_pass` voce su `http://unix:/tmp/{KESTREL SOCKET}:/;` . `{KESTREL SOCKET}` nome del socket fornito a <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ListenUnixSocket*> (ad esempio, `kestrel-test.sock` nell'esempio precedente).
* Verificare che il socket sia scrivibile da nginx (ad esempio, `chmod go+w /tmp/kestrel-test.sock` ). 

### <a name="port-0"></a>Porta 0

Quando si specifica il numero di porta `0`, Kestrel esegue l'associazione dinamica a una porta disponibile. L'esempio seguente indica come determinare la porta alla quale Kestrel ha eseguito l'associazione in fase di esecuzione:

[!code-csharp[](kestrel/samples/2.x/KestrelSample/Startup.cs?name=snippet_Configure&highlight=3-4,15-21)]

Quando si esegue l'app, l'output della finestra della console indica la porta dinamica in cui l'app può essere raggiunta:

```console
Listening on the following addresses: http://127.0.0.1:48508
```

### <a name="limitations"></a>Limitazioni

Configurare gli endpoint con i metodi seguenti:

* <xref:Microsoft.AspNetCore.Hosting.HostingAbstractionsWebHostBuilderExtensions.UseUrls*>
* L'argomento della riga di comando `--urls`
* La chiave di configurazione dell'host `urls`
* La variabile di ambiente `ASPNETCORE_URLS`

Questi metodi sono utili se si vuole che il codice funzioni con server diversi da Kestrel. Tenere presenti, tuttavia, le limitazioni seguenti:

* HTTPS non può essere usato con questi approcci, a meno che non venga specificato un certificato predefinito nella configurazione dell'endpoint HTTPS (ad esempio, usando la configurazione `KestrelServerOptions` o un file di configurazione come illustrato in precedenza in questo argomento).
* Quando sia l'approccio `Listen` che l'approccio `UseUrls` vengono usati contemporaneamente, gli endpoint `Listen` eseguono l'override degli endpoint `UseUrls`.

### <a name="iis-endpoint-configuration"></a>Configurazione dell'endpoint IIS

Quando si usa IIS, le associazioni di URL per le associazioni di override di IIS vengono impostate mediante `Listen` o `UseUrls`. Per altre informazioni, vedere l'articolo [Introduzione al modulo ASP.NET Core](xref:host-and-deploy/aspnet-core-module).

## <a name="libuv-transport-configuration"></a>Configurazione del trasporto libuv

Con la versione ASP.NET Core 2.1, il trasporto predefinito di Kestrel non si basa più su Libuv, ma su socket gestiti. Si tratta di una modifica importante per le app ASP.NET 2.0 Core che vengono aggiornate alla versione 2.1, che chiamano <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderLibuvExtensions.UseLibuv*> e dipendono da uno dei pacchetti seguenti:

* [Microsoft.AspNetCore.Server.Kestrel](https://www.nuget.org/packages/Microsoft.AspNetCore.Server.Kestrel/) (riferimento diretto al pacchetto)
* [Microsoft.AspNetCore.App](https://www.nuget.org/packages/Microsoft.AspNetCore.App/)

Per i progetti che richiedono l'uso di libuv:

* Aggiungere una dipendenza per il pacchetto [Microsoft.AspNetCore.Server.Kestrel.Transport.Libuv](https://www.nuget.org/packages/Microsoft.AspNetCore.Server.Kestrel.Transport.Libuv/) al file di progetto dell'app:

  ```xml
  <PackageReference Include="Microsoft.AspNetCore.Server.Kestrel.Transport.Libuv"
                    Version="{VERSION}" />
  ```

* Chiamare <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderLibuvExtensions.UseLibuv*>:

  ```csharp
  public class Program
  {
      public static void Main(string[] args)
      {
          CreateWebHostBuilder(args).Build().Run();
      }

      public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
          WebHost.CreateDefaultBuilder(args)
              .UseLibuv()
              .UseStartup<Startup>();
  }
  ```

### <a name="url-prefixes"></a>Prefissi URL

Se si usa `UseUrls`, l'argomento della riga di comando `--urls`, la chiave di configurazione dell'host `urls` o la variabile di ambiente `ASPNETCORE_URLS`, i prefissi URL possono avere uno dei formati seguenti.

Sono validi solo i prefissi URL HTTP. Kestrel non supporta HTTPS quando la configurazione delle associazioni URL viene eseguita con `UseUrls`.

* Indirizzo IPv4 con numero di porta

  ```
  http://65.55.39.10:80/
  ```

  `0.0.0.0` è un caso speciale che esegue l'associazione a tutti gli indirizzi IPv4.

* Indirizzo IPv6 con numero di porta

  ```
  http://[0:0:0:0:0:ffff:4137:270a]:80/
  ```

  `[::]` è l'equivalente IPv6 di `0.0.0.0` per IPv4.

* Nome host con numero di porta

  ```
  http://contoso.com:80/
  http://*:80/
  ```

  I nomi host, `*` e `+` non sono casi particolari. Tutto ciò che non è riconosciuto come un indirizzo IP o un elemento `localhost` valido esegue l'associazione a tutti gli IP IPv4 e IPv6. Per associare nomi host diversi ad app ASP.NET Core diverse sulla stessa porta, usare [HTTP.sys](xref:fundamentals/servers/httpsys) o un server proxy inverso come IIS, Nginx o Apache.

  > [!WARNING]
  > Una configurazione che prevede un proxy inverso richiede il [filtro host](#host-filtering).

* Nome host `localhost` con numero di porta o IP di loopback con numero di porta

  ```
  http://localhost:5000/
  http://127.0.0.1:5000/
  http://[::1]:5000/
  ```

  Se è specificato `localhost`, Kestrel tenta l'associazione alle interfacce di loopback sia IPv4 che IPv6. Se la porta richiesta è in uso da parte di un altro servizio in una delle due interfacce di loopback, Kestrel non viene avviato. Se una delle due interfacce di loopback non è disponibile per qualsiasi altro motivo (in genere perché IPv6 non è supportato), Kestrel registra un avviso.

## <a name="host-filtering"></a>Filtro host

Mentre supporta la configurazione in base ai prefissi, ad esempio `http://example.com:5000`, Kestrel ignora quasi sempre il nome host. L'host `localhost` è un caso speciale usato per l'associazione agli indirizzi di loopback. Qualsiasi host che non sia un indirizzo IP esplicito esegue l'associazione a tutti gli indirizzi IP pubblici. Le intestazioni `Host` non vengono convalidate.

Come soluzione alternativa, usare il middleware di filtro host. Il middleware di filtro host viene fornito dal pacchetto [Microsoft. AspNetCore. HostFiltering](https://www.nuget.org/packages/Microsoft.AspNetCore.HostFiltering) , incluso nel [metapacchetto Microsoft. AspNetCore. App](xref:fundamentals/metapackage-app) (ASP.NET Core 2,1 o 2,2). Il middleware viene aggiunto da <xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder*>, che chiama <xref:Microsoft.AspNetCore.Builder.HostFilteringServicesExtensions.AddHostFiltering*>:

[!code-csharp[](kestrel/samples-snapshot/2.x/KestrelSample/Program.cs?name=snippet_Program&highlight=9)]

Per impostazione predefinita, il middleware di filtro host è disabilitato per impostazione predefinita. Per abilitare il middleware, definire una `AllowedHosts` chiave in *appsettings.json* / *appSettings. \<EnvironmentName> JSON*. Il valore è un elenco con valori delimitati da punto e virgola di nomi host senza numeri di porta:

*appsettings.json*:

```json
{
  "AllowedHosts": "example.com;localhost"
}
```

> [!NOTE]
> Il [middleware delle intestazioni inoltrate](xref:host-and-deploy/proxy-load-balancer) include anche un'opzione <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.AllowedHosts>. Il middleware di intestazioni inoltrate e il middleware di filtro host hanno funzionalità simili per diversi scenari. L'impostazione di `AllowedHosts` con il middleware delle intestazioni inoltrate è appropriato se l'intestazione `Host` non viene mantenuta durante l'inoltro delle richieste con un server proxy inverso o il bilanciamento del carico. L'impostazione di `AllowedHosts` con il middleware del filtro host è appropriata se si usa Kestrel come server perimetrale rivolto al pubblico o se l'intestazione `Host` viene inoltrata direttamente.
>
> Per altre informazioni sul middleware delle intestazioni inoltrate, vedere <xref:host-and-deploy/proxy-load-balancer>.

## <a name="http11-request-draining"></a>Svuotamento richieste HTTP/1.1

L'apertura di connessioni HTTP richiede molto tempo. Per HTTPS, è anche a elevato utilizzo di risorse. Pertanto, gheppio tenta di riutilizzare le connessioni per il protocollo HTTP/1.1. Il corpo della richiesta deve essere completamente utilizzato per consentire il riutilizzo della connessione. L'app non usa sempre il corpo della richiesta, ad esempio le `POST` richieste in cui il server restituisce una risposta di reindirizzamento o 404. Nel `POST` caso-Reindirizzamento:

* Il client potrebbe avere già inviato parte dei `POST` dati.
* Il server scrive la risposta 301.
* Non è possibile usare la connessione per una nuova richiesta fino a quando i `POST` dati del corpo della richiesta precedente non sono stati completamente letti.
* Gheppio tenta di svuotare il corpo della richiesta. Lo svuotamento del corpo della richiesta implica la lettura e l'eliminazione dei dati senza elaborarli.

Il processo di svuotamento crea un tradoff tra la possibilità di riutilizzare la connessione e il tempo necessario per svuotare i dati rimanenti:

* Lo svuotamento ha un timeout di cinque secondi, che non è configurabile.
* Se tutti i dati specificati dall' `Content-Length` `Transfer-Encoding` intestazione o non sono stati letti prima del timeout, la connessione viene chiusa.

In alcuni casi può essere necessario terminare immediatamente la richiesta, prima o dopo la scrittura della risposta. È ad esempio possibile che i client dispongano di protezioni dati restrittive, pertanto la limitazione dei dati caricati potrebbe avere una priorità. In questi casi, per terminare una richiesta, chiamare [HttpContext. Abort](xref:Microsoft.AspNetCore.Http.HttpContext.Abort%2A) da un controller, una Razor pagina o un middleware.

Ci sono alcune avvertenze per chiamare `Abort` :

* La creazione di nuove connessioni può essere lenta e dispendiosa.
* Non vi è alcuna garanzia che il client abbia letto la risposta prima che la connessione venga chiusa.
* `Abort`La chiamata a deve essere rara e riservata per i casi di errore gravi, non per errori comuni.
  * Chiamare solo `Abort` quando è necessario risolvere un problema specifico. Ad esempio, chiamare `Abort` se i client malintenzionati tentano di eseguire `POST` dati o quando si verifica un bug nel codice client che genera richieste di grandi dimensioni o numerose.
  * Non chiamare le `Abort` situazioni di errore comuni, ad esempio HTTP 404 (non trovato).

Quando si chiama [HttpResponse. CompleteAsync](xref:Microsoft.AspNetCore.Http.HttpResponse.CompleteAsync%2A) , prima `Abort` di chiamare si garantisce che il server abbia completato la scrittura della risposta. Tuttavia, il comportamento del client non è prevedibile e potrebbe non essere in grado di leggere la risposta prima che la connessione venga interrotta.

Questo processo è diverso per HTTP/2 poiché il protocollo supporta l'interruzione dei singoli flussi di richiesta senza chiudere la connessione. Il timeout di svuotamento di cinque secondi non è applicabile. Se dopo il completamento di una risposta sono presenti dati del corpo della richiesta non letti, il server invia un frame RST HTTP/2. I frame di dati del corpo della richiesta aggiuntivi vengono ignorati.

Se possibile, è preferibile che i client utilizzino l'intestazione della richiesta [expect: 100-continue](https://developer.mozilla.org/docs/Web/HTTP/Status/100) e attendano la risposta del server prima di iniziare a inviare il corpo della richiesta. Che fornisce al client la possibilità di esaminare la risposta e interrompere prima di inviare dati non necessari.

## <a name="additional-resources"></a>Risorse aggiuntive

* Quando si usano i socket UNIX in Linux, il socket non viene eliminato automaticamente all'arresto dell'app. Per altre informazioni, vedere [questo problema di GitHub](https://github.com/dotnet/aspnetcore/issues/14134).
* <xref:test/troubleshoot>
* <xref:security/enforcing-ssl>
* <xref:host-and-deploy/proxy-load-balancer>
* [RFC 7230: Message Syntax and Routing (Section 5.4: Host)](https://tools.ietf.org/html/rfc7230#section-5.4) (RFC 7230: Sintassi e routing dei messaggi (sezione 5.4: Host))

::: moniker-end
