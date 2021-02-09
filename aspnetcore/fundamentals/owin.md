---
title: Open Web Interface for .NET (OWIN) con ASP.NET Core
author: ardalis
description: Informazioni su come ASP.NET Core supporta Open Web Interface for .NET (OWIN) che consente ai server Web di disaccoppiare le app Web.
ms.author: riande
ms.custom: H1Hack27Feb2017
ms.date: 2/8/2021
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
uid: fundamentals/owin
ms.openlocfilehash: c44803c6f67d8e03759a01f7aa71d82088be9a11
ms.sourcegitcommit: ef8d8c79993a6608bf597ad036edcf30b231843f
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/09/2021
ms.locfileid: "99975274"
---
# <a name="open-web-interface-for-net-owin-with-aspnet-core"></a>Open Web Interface for .NET (OWIN) con ASP.NET Core

Di [Steve Smith](https://ardalis.com/) e [Rick Anderson](https://twitter.com/RickAndMSFT)

ASP.NET Core:

* Supporta Open Web Interface for .NET (OWIN).
* Prevede sostituzioni compatibili con .NET Core per le `Microsoft.Owin.*` librerie ([Katana](/aspnet/aspnet/overview/owin-and-katana/)).

OWIN consente alle app Web di essere disaccoppiate dai server Web. Definisce un modo standard per usare il middleware in una pipeline per gestire le richieste e le risposte associate. Le applicazioni ASP.NET Core e il middleware possono interagire con middleware, server e applicazioni basati su OWIN.

OWIN specifica un livello di disaccoppiamento che consente di usare contemporaneamente due framework con modelli a oggetti diversi. Il pacchetto `Microsoft.AspNetCore.Owin` offre due implementazioni dell'adattatore:

* Da ASP.NET Core a OWIN 
* Da OWIN a ASP.NET Core

In questo modo ASP.NET Core può essere ospitato in un server/host compatibile con OWIN o altri componenti compatibili con OWIN possono essere eseguiti su ASP.NET Core.

> [!NOTE]
> L'uso di questi adattatori comporta una riduzione delle prestazioni. Le app che usano solo componenti di ASP.NET Core non devono usare il pacchetto o gli adattatori `Microsoft.AspNetCore.Owin`.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/owin/sample) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="running-owin-middleware-in-the-aspnet-core-pipeline"></a>Esecuzione del middleware OWIN nella pipeline ASP.NET Core

Il supporto di OWIN per ASP.NET Core viene distribuito come parte del pacchetto `Microsoft.AspNetCore.Owin`. Per importare il supporto OWIN nel progetto è necessario installare il pacchetto.

Il middleware OWIN è conforme alle [specifiche OWIN](https://owin.org/spec/spec/owin-1.0.0.html), che richiedono l'interfaccia `Func<IDictionary<string, object>, Task>` e che siano impostate chiavi specifiche, come ad esempio `owin.ResponseBody`. Il semplice middleware OWIN illustrato di seguito visualizza "Hello World":

```csharp
public Task OwinHello(IDictionary<string, object> environment)
{
    string responseText = "Hello World via OWIN";
    byte[] responseBytes = Encoding.UTF8.GetBytes(responseText);

    // OWIN Environment Keys: https://owin.org/spec/spec/owin-1.0.0.html
    var responseStream = (Stream)environment["owin.ResponseBody"];
    var responseHeaders = (IDictionary<string, string[]>)environment["owin.ResponseHeaders"];

    responseHeaders["Content-Length"] = new string[] { responseBytes.Length.ToString(CultureInfo.InvariantCulture) };
    responseHeaders["Content-Type"] = new string[] { "text/plain" };

    return responseStream.WriteAsync(responseBytes, 0, responseBytes.Length);
}
```

La firma di esempio restituisce un oggetto `Task` e accetta un oggetto `IDictionary<string, object>` come richiesto da OWIN.

Il codice seguente illustra come aggiungere il middleware `OwinHello`, illustrato in precedenza, alla pipeline ASP.NET Core con il metodo di estensione `UseOwin`.

```csharp
public void Configure(IApplicationBuilder app)
{
    app.UseOwin(pipeline =>
    {
        pipeline(next => OwinHello);
    });
}
```

È possibile configurare altre azioni da eseguire all'interno della pipeline OWIN.

> [!NOTE]
> Le intestazioni di risposta devono essere modificate solo prima della prima scrittura nel flusso di risposta.

> [!NOTE]
> Le chiamate multiple al metodo `UseOwin` sono sconsigliate per non compromettere le prestazioni. I componenti OWIN funzionano meglio se raggruppati insieme.

```csharp
app.UseOwin(pipeline =>
{
    pipeline(next =>
    {
        return async environment =>
        {
            // Do something before.
            await next(environment);
            // Do something after.
        };
    });
});
```

<a name="hosting-on-owin"></a>

## <a name="run-aspnet-core-on-an-owin-based-server-and-use-its-websockets-support"></a>Eseguire ASP.NET Core in un server basato su OWIN e usare il supporto di WebSocket

Un altro esempio di funzionalità dei server basati su OWIN che può essere sfruttata da ASP.NET Core è l'accesso a funzionalità quali i WebSocket. Il server Web OWIN per .NET usato nell'esempio precedente ha il supporto per i WebSocket incorporato che può essere usato da un'applicazione ASP.NET Core. L'esempio seguente illustra una semplice app Web che supporta i WebSocket e restituisce tutto quanto inviato al server tramite WebSocket.

```csharp
public class Startup
{
    public void Configure(IApplicationBuilder app)
    {
        app.Use(async (context, next) =>
        {
            if (context.WebSockets.IsWebSocketRequest)
            {
                WebSocket webSocket = await context.WebSockets.AcceptWebSocketAsync();
                await EchoWebSocket(webSocket);
            }
            else
            {
                await next();
            }
        });

        app.Run(context =>
        {
            return context.Response.WriteAsync("Hello World");
        });
    }

    private async Task EchoWebSocket(WebSocket webSocket)
    {
        byte[] buffer = new byte[1024];
        WebSocketReceiveResult received = await webSocket.ReceiveAsync(
            new ArraySegment<byte>(buffer), CancellationToken.None);

        while (!webSocket.CloseStatus.HasValue)
        {
            // Echo anything we receive
            await webSocket.SendAsync(new ArraySegment<byte>(buffer, 0, received.Count), 
                received.MessageType, received.EndOfMessage, CancellationToken.None);

            received = await webSocket.ReceiveAsync(new ArraySegment<byte>(buffer), 
                CancellationToken.None);
        }

        await webSocket.CloseAsync(webSocket.CloseStatus.Value, 
            webSocket.CloseStatusDescription, CancellationToken.None);
    }
}
```

## <a name="owin-environment"></a>Ambiente OWIN

È possibile creare un ambiente OWIN tramite `HttpContext`.

```csharp

   var environment = new OwinEnvironment(HttpContext);
   var features = new OwinFeatureCollection(environment);
   ```

## <a name="owin-keys"></a>Chiavi OWIN

OWIN dipende da un oggetto `IDictionary<string,object>` per comunicare informazioni attraverso uno scambio di richiesta/risposta HTTP. ASP.NET Core implementa le chiavi elencate di seguito. Vedere le [specifiche principali e le estensioni](https://owin.org/#spec), nonché la pagina [OWIN Key Guidelines and Common Keys](https://owin.org/spec/spec/CommonKeys.html) (Linee guida sulle chiavi OWIN e chiavi comuni).

### <a name="request-data-owin-v100"></a>Dati della richiesta (OWIN versione 1.0.0)

| Chiave               | Valore (tipo) | Descrizione |
| ----------------- | ------------ | ----------- |
| owin.RequestScheme | `String` |  |
| owin.RequestMethod  | `String` | |    
| owin.RequestPathBase  | `String` | |    
| owin.RequestPath | `String` | |     
| owin.RequestQueryString  | `String` | |    
| owin.RequestProtocol  | `String` | |    
| owin.RequestHeaders | `IDictionary<string,string[]>`  | |
| owin.RequestBody | `Stream`  | |

### <a name="request-data-owin-v110"></a>Dati della richiesta (OWIN versione 1.1.0)

| Chiave               | Valore (tipo) | Descrizione |
| ----------------- | ------------ | ----------- |
| owin.RequestId | `String` | Facoltativo |

### <a name="response-data-owin-v100"></a>Dati della risposta (OWIN versione 1.0.0)

| Chiave               | Valore (tipo) | Descrizione |
| ----------------- | ------------ | ----------- |
| owin.ResponseStatusCode | `int` | Facoltativo |
| owin.ResponseReasonPhrase | `String` | Facoltativo |
| owin.ResponseHeaders | `IDictionary<string,string[]>`  | |
| owin.ResponseBody | `Stream`  | |

### <a name="other-data-owin-v100"></a>Altri dati (OWIN versione 1.0.0)

| Chiave               | Valore (tipo) | Descrizione |
| ----------------- | ------------ | ----------- |
| owin.CallCancelled | `CancellationToken` |  |
| owin.Version  | `String` | |   

### <a name="common-keys"></a>Chiavi comuni

| Chiave               | Valore (tipo) | Descrizione |
| ----------------- | ------------ | ----------- |
| ssl.ClientCertificate | `X509Certificate` |  |
| ssl.LoadClientCertAsync  | `Func<Task>` | |    
| server.RemoteIpAddress  | `String` | |    
| server.RemotePort | `String` | |     
| server.LocalIpAddress  | `String` | |    
| server.LocalPort  | `String` | |    
| server.IsLocal  | `bool` | |    
| server.OnSendingHeaders  | `Action<Action<object>,object>` | |

### <a name="sendfiles-v030"></a>SendFiles v0.3.0

| Chiave               | Valore (tipo) | Descrizione |
| ----------------- | ------------ | ----------- |
| sendfile.SendAsync | Vedere [Delegate signature](https://owin.org/spec/extensions/owin-SendFile-Extension-v0.3.0.htm) (Firma delegato) | Per richiesta |

### <a name="opaque-v030"></a>Opaque v0.3.0

| Chiave               | Valore (tipo) | Descrizione |
| ----------------- | ------------ | ----------- |
| opaque.Version | `String` |  |
| opaque.Upgrade | `OpaqueUpgrade` | Vedere [Delegate signature](https://owin.org/spec/extensions/owin-SendFile-Extension-v0.3.0.htm) (Firma delegato) |
| opaque.Stream | `Stream` |  |
| opaque.CallCancelled | `CancellationToken` |  |

### <a name="websocket-v030"></a>WebSocket v0.3.0

| Chiave               | Valore (tipo) | Descrizione |
| ----------------- | ------------ | ----------- |
| websocket.Version | `String` |  |
| websocket.Accept | `WebSocketAccept` | Vedere [Delegate signature](https://owin.org/spec/extensions/owin-SendFile-Extension-v0.3.0.htm) (Firma delegato) |
| websocket.AcceptAlt |  | Non specificata |
| websocket.SubProtocol | `String` | Vedere [RFC6455 sezione 4.2.2](https://tools.ietf.org/html/rfc6455#section-4.2.2) passaggio 5.5 |
| websocket.SendAsync | `WebSocketSendAsync` | Vedere [Delegate signature](https://owin.org/spec/extensions/owin-SendFile-Extension-v0.3.0.htm) (Firma delegato)  |
| websocket.ReceiveAsync | `WebSocketReceiveAsync` | Vedere [Delegate signature](https://owin.org/spec/extensions/owin-SendFile-Extension-v0.3.0.htm) (Firma delegato)  |
| websocket.CloseAsync | `WebSocketCloseAsync` | Vedere [Delegate signature](https://owin.org/spec/extensions/owin-SendFile-Extension-v0.3.0.htm) (Firma delegato)  |
| websocket.CallCancelled | `CancellationToken` |  |
| websocket.ClientCloseStatus | `int` | Facoltativo |
| websocket.ClientCloseDescription | `String` | Facoltativo |

## <a name="additional-resources"></a>Risorse aggiuntive

* [Middleware](xref:fundamentals/middleware/index)
* [Server](xref:fundamentals/servers/index)
