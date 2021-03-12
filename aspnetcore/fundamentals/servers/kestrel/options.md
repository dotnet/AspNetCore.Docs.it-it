---
title: Configurare le opzioni per il server Web ASP.NET Core gheppio
author: rick-anderson
description: Informazioni sulla configurazione delle opzioni per gheppio, il server Web multipiattaforma per ASP.NET Core.
monikerRange: '>= aspnetcore-5.0'
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
uid: fundamentals/servers/kestrel/options
ms.openlocfilehash: 48b4af2dfc925c4444c2bd0e43d04f2f0f3ddd17
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102587008"
---
# <a name="configure-options-for-the-aspnet-core-kestrel-web-server"></a>Configurare le opzioni per il server Web ASP.NET Core gheppio

Il server Web Kestrel dispone di opzioni di configurazione dei vincoli che risultano particolarmente utili nelle distribuzioni con connessione Internet.

Per fornire una configurazione aggiuntiva dopo la chiamata di <xref:Microsoft.Extensions.Hosting.GenericHostBuilderExtensions.ConfigureWebHostDefaults%2A>, usare <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderKestrelExtensions.ConfigureKestrel%2A>:

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

Impostare i vincoli per la proprietà <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Limits> della classe <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions>. La proprietà `Limits` contiene un'istanza della classe <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits>.

Negli esempi seguenti viene usato lo spazio dei nomi <xref:Microsoft.AspNetCore.Server.Kestrel.Core>:

```csharp
using Microsoft.AspNetCore.Server.Kestrel.Core;
```

Negli esempi illustrati più avanti in questo articolo, le opzioni di gheppio sono configurate nel codice C#. È inoltre possibile impostare le opzioni di Gheppio utilizzando un [provider di configurazione](xref:fundamentals/configuration/index). Il [provider di configurazione file](xref:fundamentals/configuration/index#file-configuration-provider) , ad esempio, può caricare la configurazione di Gheppio da un `appsettings.json` `appsettings.{Environment}.json` file o:

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
> <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions> e la [configurazione dell'endpoint](xref:fundamentals/servers/kestrel/endpoints) sono configurabili dai provider di configurazione. La configurazione del gheppio rimanente deve essere configurata nel codice C#.

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

  In `Program.cs` caricare la `Kestrel` sezione della configurazione nella configurazione di Gheppio:

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

## <a name="general-limits"></a>Limiti generali

### <a name="keep-alive-timeout"></a>Timeout keep-alive

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.KeepAliveTimeout>

Ottiene o imposta il [timeout keep-alive](https://tools.ietf.org/html/rfc7230#section-6.5). Il valore predefinito è 2 minuti.

[!code-csharp[](samples/3.x/KestrelSample/Program.cs?name=snippet_Limits&highlight=19-20)]

### <a name="maximum-client-connections"></a>Numero massimo di connessioni client

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.MaxConcurrentConnections><br>
<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.MaxConcurrentUpgradedConnections>

È possibile impostare il numero massimo di connessioni TCP aperte simultaneamente per l'intera app con il codice seguente:

[!code-csharp[](samples/3.x/KestrelSample/Program.cs?name=snippet_Limits&highlight=3)]

È previsto un limite separato per le connessioni che sono state aggiornate da HTTP o HTTPS a un altro protocollo (ad esempio su una richiesta WebSocket). Dopo l'aggiornamento di una connessione, questa non viene conteggiata per il limite `MaxConcurrentConnections`.

[!code-csharp[](samples/3.x/KestrelSample/Program.cs?name=snippet_Limits&highlight=4)]

Per impostazione predefinita, il numero massimo di connessioni è illimitato (null).

### <a name="maximum-request-body-size"></a>Dimensione massima del corpo della richiesta

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.MaxRequestBodySize>

La dimensione massima predefinita del corpo della richiesta è pari a 30.000.000 di byte, equivalenti a circa 28,6 MB.

Il metodo consigliato per ignorare il limite in un'app ASP.NET Core MVC è l'uso dell'attributo <xref:Microsoft.AspNetCore.Mvc.RequestSizeLimitAttribute> in un metodo di azione:

```csharp
[RequestSizeLimit(100000000)]
public IActionResult MyActionMethod()
```

Nell'esempio seguente viene illustrato come configurare il vincolo per l'app in ogni richiesta:

[!code-csharp[](samples/3.x/KestrelSample/Program.cs?name=snippet_Limits&highlight=5)]

Eseguire l'override dell'impostazione per una richiesta specifica nel middleware:

[!code-csharp[](samples/3.x/KestrelSample/Startup.cs?name=snippet_Limits&highlight=3-4)]

Viene generata un'eccezione se l'app configura il limite per una richiesta dopo che l'app ha iniziato a leggere la richiesta. Una proprietà `IsReadOnly` indica se la proprietà `MaxRequestBodySize` è in stato di sola lettura e pertanto è troppo tardi per configurare il limite.

Quando un'app è in esecuzione [out-of-process](xref:host-and-deploy/iis/index#out-of-process-hosting-model) dietro il [modulo ASP.NET Core](xref:host-and-deploy/aspnet-core-module), il limite delle dimensioni del corpo della richiesta del gheppio è disabilitato. IIS imposta già il limite.

### <a name="minimum-request-body-data-rate"></a>Velocità minima dei dati del corpo della richiesta

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.MinRequestBodyDataRate><br>
<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.MinResponseDataRate>

Kestrel controlla ogni secondo se i dati arrivano alla velocità in byte al secondo specificata. Se la frequenza scende sotto il valore minimo, si è verificato il timeout della connessione. Il periodo di tolleranza è la quantità di tempo in cui il gheppio consente al client di aumentare la velocità di invio fino al minimo. La frequenza non viene controllata durante tale periodo di tempo. Il periodo di tolleranza consente di evitare l'eliminazione delle connessioni che inviano inizialmente dati a una velocità ridotta a causa dell'avvio lento di TCP.

La frequenza minima predefinita è 240 byte al secondo con un periodo di tolleranza di 5 secondi.

Anche per la risposta è prevista una velocità minima. Il codice per impostare il limite della richiesta e della risposta è identico e varia solo per `RequestBody` o `Response` nei nomi di proprietà e di interfaccia.

Di seguito è riportato un esempio in cui viene illustrato come configurare le tariffe minime per i dati in `Program.cs` :

[!code-csharp[](samples/3.x/KestrelSample/Program.cs?name=snippet_Limits&highlight=6-11)]

Ignorare i limiti di velocità minima per ogni richiesta nel middleware:

[!code-csharp[](samples/3.x/KestrelSample/Startup.cs?name=snippet_Limits&highlight=6-21)]

L'oggetto a <xref:Microsoft.AspNetCore.Server.Kestrel.Core.Features.IHttpMinResponseDataRateFeature> cui si fa riferimento nell'esempio precedente non è presente nelle <xref:Microsoft.AspNetCore.Http.HttpContext.Features?displayProperty=nameWithType> richieste http/2. La modifica dei limiti di velocità in base alle singole richieste non è in genere supportata per HTTP/2 a causa del supporto del protocollo per il multiplexing delle richieste. La funzionalità <xref:Microsoft.AspNetCore.Server.Kestrel.Core.Features.IHttpMinRequestBodyDataRateFeature> è tuttavia ancora presente in `HttpContext.Features` per le richieste HTTP/2, perché il limite di velocità di lettura può comunque essere *disabilitato interamente* per ogni singola richiesta impostando <xref:Microsoft.AspNetCore.Server.Kestrel.Core.Features.IHttpMinResponseDataRateFeature.MinDataRate?displayProperty=nameWithType> su `null` anche per una richiesta HTTP/2. Il tentativo di leggere `IHttpMinRequestBodyDataRateFeature.MinDataRate` o il tentativo di impostazione di un valore diverso da `null` causerà la generazione di una <xref:System.NotSupportedException> per una richiesta HTTP/2.

I limiti di velocità a livello di server configurati tramite <xref:Microsoft.AspNetCore.Server.Kestrel.KestrelServerOptions.Limits?displayProperty=nameWithType> vengono tuttavia applicati alle connessioni HTTP/1.x e HTTP/2.

### <a name="request-headers-timeout"></a>Timeout delle intestazioni delle richieste

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.RequestHeadersTimeout>

Ottiene o imposta la quantità massima di tempo che il server dedica alla ricezione delle intestazioni delle richieste. Il valore predefinito è 30 secondi.

[!code-csharp[](samples/3.x/KestrelSample/Program.cs?name=snippet_Limits&highlight=21-22)]

## <a name="http2-limits"></a>Limiti HTTP/2

I limiti di questa sezione sono impostati su on <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.Http2?displayProperty=nameWithType> .

### <a name="maximum-streams-per-connection"></a>Numero massimo di flussi per connessione

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.Http2Limits.MaxStreamsPerConnection>

Limita il numero di flussi di richieste simultanee per connessione HTTP/2. I flussi in eccesso vengono rifiutati.

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.Limits.Http2.MaxStreamsPerConnection = 100;
});
```

Il valore predefinito è 100.

### <a name="header-table-size"></a>Dimensioni della tabella delle intestazioni

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.Http2Limits.HeaderTableSize>

Il decodificatore HPACK decomprime le intestazioni HTTP per le connessioni HTTP/2. `HeaderTableSize` limita le dimensioni della tabella di compressione delle intestazioni usata dal decodificatore HPACK. Il valore viene specificato in ottetti e deve essere maggiore di zero (0).

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.Limits.Http2.HeaderTableSize = 4096;
});
```

Il valore predefinito è 4096.

### <a name="maximum-frame-size"></a>Dimensione massima del frame

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.Http2Limits.MaxFrameSize>

Indica la dimensione massima consentita di un payload del frame di connessione HTTP/2 ricevuto o inviato dal server. Il valore viene specificato in ottetti e deve essere compreso tra 2^14 (16.384) e 2^24-1 (16.777.215).

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.Limits.Http2.MaxFrameSize = 16384;
});
```

Il valore predefinito è 2^14 (16.384).

### <a name="maximum-request-header-size"></a>Dimensioni massime dell'intestazione della richiesta

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.Http2Limits.MaxRequestHeaderFieldSize>

Indica la dimensione massima consentita in ottetti dei valori dell'intestazione della richiesta. Questo limite si applica sia al nome che al valore nelle rappresentazioni compresse e non compresse. Il valore deve essere maggiore di zero (0).

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.Limits.Http2.MaxRequestHeaderFieldSize = 8192;
});
```

Il valore predefinito è 8.192.

### <a name="initial-connection-window-size"></a>Dimensioni della finestra di connessione iniziali

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.Http2Limits.InitialConnectionWindowSize>

Indica il numero massimo di dati del corpo della richiesta, in byte, del buffer del server contemporaneamente aggregati in tutte le richieste (flussi) per connessione. Le richieste sono anche limitate da `Http2.InitialStreamWindowSize`. Il valore deve essere maggiore di o uguale a 65.535 e minore di 2^31 (2.147.483.648).

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.Limits.Http2.InitialConnectionWindowSize = 131072;
});
```

Il valore predefinito è 128 KB (131.072).

### <a name="initial-stream-window-size"></a>Dimensioni della finestra di flusso iniziali

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.Http2Limits.InitialStreamWindowSize>

Indica il numero massimo di dati del corpo della richiesta in byte memorizzati nel buffer del server in una sola volta per richiesta (flusso). Anche le richieste sono limitate da [`InitialConnectionWindowSize`](#initial-connection-window-size) . Il valore deve essere maggiore di o uguale a 65.535 e minore di 2^31 (2.147.483.648).

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.Limits.Http2.InitialStreamWindowSize = 98304;
});
```

Il valore predefinito è 96 KB (98.304).

### <a name="http2-keep-alive-ping-configuration"></a>Configurazione del ping keep alive HTTP/2

Il gheppio può essere configurato per l'invio di ping HTTP/2 ai client connessi. I ping HTTP/2 servono più scopi:

* Mantieni attive le connessioni inattive. Alcuni client e server proxy chiudono le connessioni inattive. I ping HTTP/2 sono considerati attività su una connessione e impediscono la chiusura della connessione come inattiva.
* Chiudere le connessioni non integre. Le connessioni in cui il client non risponde al ping keep-alive nell'ora configurata vengono chiuse dal server.

Sono disponibili due opzioni di configurazione correlate a HTTP/2 ping Keep-Alive:

* <xref:Microsoft.AspNetCore.Server.Kestrel.Core.Http2Limits.KeepAlivePingDelay> è un oggetto <xref:System.TimeSpan> che configura l'intervallo del ping. Il server invia al client un ping Keep-Alive se non riceve frame per questo periodo di tempo. Quando questa opzione è impostata su, i ping Keep-Alive sono disabilitati <xref:System.TimeSpan.MaxValue?displayProperty=nameWithType> . Il valore predefinito è <xref:System.TimeSpan.MaxValue?displayProperty=nameWithType>.
* <xref:Microsoft.AspNetCore.Server.Kestrel.Core.Http2Limits.KeepAlivePingTimeout> è un oggetto <xref:System.TimeSpan> che configura il timeout del ping. Se il server non riceve frame, ad esempio un ping di risposta, durante questo timeout la connessione viene chiusa. Quando questa opzione è impostata su, il timeout keep alive è disabilitato <xref:System.TimeSpan.MaxValue?displayProperty=nameWithType> . Il valore predefinito è 20 secondi.

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.Limits.Http2.KeepAlivePingDelay = TimeSpan.FromSeconds(30);
    serverOptions.Limits.Http2.KeepAlivePingTimeout = TimeSpan.FromSeconds(60);
});
```

## <a name="other-options"></a>Altre opzioni

### <a name="synchronous-io"></a>I/O sincrono

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.AllowSynchronousIO> Controlla se è consentito l'I/O sincrono per la richiesta e la risposta. Il valore predefinito è `false`.

> [!WARNING]
> Un numero elevato di operazioni di I/O sincrone bloccanti può causare l'esaurimento del pool di thread, il che rende l'app non risponde. Abilitare solo `AllowSynchronousIO` quando si usa una libreria che non supporta l'I/O asincrono.

L'esempio seguente abilita l'I/O sincrono:

[!code-csharp[](samples/5.x/KestrelSample/Program.cs?name=snippet_SyncIO)]

Per informazioni su altre opzioni e limiti di Kestrel, vedere:

* <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions>
* <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits>
* <xref:Microsoft.AspNetCore.Server.Kestrel.Core.ListenOptions>
