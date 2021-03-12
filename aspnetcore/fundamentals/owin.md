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
ms.openlocfilehash: e476f3f62a250d960c809e5062b3bd8ca3a1940a
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102587234"
---
# <a name="open-web-interface-for-net-owin-with-aspnet-core"></a><span data-ttu-id="10dd1-103">Open Web Interface for .NET (OWIN) con ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="10dd1-103">Open Web Interface for .NET (OWIN) with ASP.NET Core</span></span>

<span data-ttu-id="10dd1-104">Di [Steve Smith](https://ardalis.com/) e [Rick Anderson](https://twitter.com/RickAndMSFT)</span><span class="sxs-lookup"><span data-stu-id="10dd1-104">By [Steve Smith](https://ardalis.com/) and [Rick Anderson](https://twitter.com/RickAndMSFT)</span></span>

<span data-ttu-id="10dd1-105">ASP.NET Core:</span><span class="sxs-lookup"><span data-stu-id="10dd1-105">ASP.NET Core:</span></span>

* <span data-ttu-id="10dd1-106">Supporta Open Web Interface for .NET (OWIN).</span><span class="sxs-lookup"><span data-stu-id="10dd1-106">Supports the Open Web Interface for .NET (OWIN).</span></span>
* <span data-ttu-id="10dd1-107">Prevede sostituzioni compatibili con .NET Core per le `Microsoft.Owin.*` librerie ([Katana](/aspnet/aspnet/overview/owin-and-katana/)).</span><span class="sxs-lookup"><span data-stu-id="10dd1-107">Has .NET Core compatible replacements for the `Microsoft.Owin.*` ([Katana](/aspnet/aspnet/overview/owin-and-katana/)) libraries.</span></span>

<span data-ttu-id="10dd1-108">OWIN consente alle app Web di essere disaccoppiate dai server Web.</span><span class="sxs-lookup"><span data-stu-id="10dd1-108">OWIN allows web apps to be decoupled from web servers.</span></span> <span data-ttu-id="10dd1-109">Definisce un modo standard per usare il middleware in una pipeline per gestire le richieste e le risposte associate.</span><span class="sxs-lookup"><span data-stu-id="10dd1-109">It defines a standard way for middleware to be used in a pipeline to handle requests and associated responses.</span></span> <span data-ttu-id="10dd1-110">Le applicazioni ASP.NET Core e il middleware possono interagire con middleware, server e applicazioni basati su OWIN.</span><span class="sxs-lookup"><span data-stu-id="10dd1-110">ASP.NET Core applications and middleware can interoperate with OWIN-based applications, servers, and middleware.</span></span>

<span data-ttu-id="10dd1-111">OWIN specifica un livello di disaccoppiamento che consente di usare contemporaneamente due framework con modelli a oggetti diversi.</span><span class="sxs-lookup"><span data-stu-id="10dd1-111">OWIN provides a decoupling layer that allows two frameworks with disparate object models to be used together.</span></span> <span data-ttu-id="10dd1-112">Il pacchetto `Microsoft.AspNetCore.Owin` offre due implementazioni dell'adattatore:</span><span class="sxs-lookup"><span data-stu-id="10dd1-112">The `Microsoft.AspNetCore.Owin` package provides two adapter implementations:</span></span>

* <span data-ttu-id="10dd1-113">Da ASP.NET Core a OWIN</span><span class="sxs-lookup"><span data-stu-id="10dd1-113">ASP.NET Core to OWIN</span></span> 
* <span data-ttu-id="10dd1-114">Da OWIN a ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="10dd1-114">OWIN to ASP.NET Core</span></span>

<span data-ttu-id="10dd1-115">In questo modo ASP.NET Core può essere ospitato in un server/host compatibile con OWIN o altri componenti compatibili con OWIN possono essere eseguiti su ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="10dd1-115">This allows ASP.NET Core to be hosted on top of an OWIN compatible server/host or for other OWIN compatible components to be run on top of ASP.NET Core.</span></span>

> [!NOTE]
> <span data-ttu-id="10dd1-116">L'uso di questi adattatori comporta una riduzione delle prestazioni.</span><span class="sxs-lookup"><span data-stu-id="10dd1-116">Using these adapters comes with a performance cost.</span></span> <span data-ttu-id="10dd1-117">Le app che usano solo componenti di ASP.NET Core non devono usare il pacchetto o gli adattatori `Microsoft.AspNetCore.Owin`.</span><span class="sxs-lookup"><span data-stu-id="10dd1-117">Apps using only ASP.NET Core components shouldn't use the `Microsoft.AspNetCore.Owin` package or adapters.</span></span>

<span data-ttu-id="10dd1-118">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/owin/sample) ([procedura per il download](xref:index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="10dd1-118">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/fundamentals/owin/sample) ([how to download](xref:index#how-to-download-a-sample))</span></span>

## <a name="running-owin-middleware-in-the-aspnet-core-pipeline"></a><span data-ttu-id="10dd1-119">Esecuzione del middleware OWIN nella pipeline ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="10dd1-119">Running OWIN middleware in the ASP.NET Core pipeline</span></span>

<span data-ttu-id="10dd1-120">Il supporto di OWIN per ASP.NET Core viene distribuito come parte del pacchetto `Microsoft.AspNetCore.Owin`.</span><span class="sxs-lookup"><span data-stu-id="10dd1-120">ASP.NET Core's OWIN support is deployed as part of the `Microsoft.AspNetCore.Owin` package.</span></span> <span data-ttu-id="10dd1-121">Per importare il supporto OWIN nel progetto è necessario installare il pacchetto.</span><span class="sxs-lookup"><span data-stu-id="10dd1-121">You can import OWIN support into your project by installing this package.</span></span>

<span data-ttu-id="10dd1-122">Il middleware OWIN è conforme alle [specifiche OWIN](https://owin.org/spec/spec/owin-1.0.0.html), che richiedono l'interfaccia `Func<IDictionary<string, object>, Task>` e che siano impostate chiavi specifiche, come ad esempio `owin.ResponseBody`.</span><span class="sxs-lookup"><span data-stu-id="10dd1-122">OWIN middleware conforms to the [OWIN specification](https://owin.org/spec/spec/owin-1.0.0.html), which requires a `Func<IDictionary<string, object>, Task>` interface, and specific keys be set (such as `owin.ResponseBody`).</span></span> <span data-ttu-id="10dd1-123">Il semplice middleware OWIN illustrato di seguito visualizza "Hello World":</span><span class="sxs-lookup"><span data-stu-id="10dd1-123">The following simple OWIN middleware displays "Hello World":</span></span>

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

<span data-ttu-id="10dd1-124">La firma di esempio restituisce un oggetto `Task` e accetta un oggetto `IDictionary<string, object>` come richiesto da OWIN.</span><span class="sxs-lookup"><span data-stu-id="10dd1-124">The sample signature returns a `Task` and accepts an `IDictionary<string, object>` as required by OWIN.</span></span>

<span data-ttu-id="10dd1-125">Il codice seguente illustra come aggiungere il middleware `OwinHello`, illustrato in precedenza, alla pipeline ASP.NET Core con il metodo di estensione `UseOwin`.</span><span class="sxs-lookup"><span data-stu-id="10dd1-125">The following code shows how to add the `OwinHello` middleware (shown above) to the ASP.NET Core pipeline with the `UseOwin` extension method.</span></span>

```csharp
public void Configure(IApplicationBuilder app)
{
    app.UseOwin(pipeline =>
    {
        pipeline(next => OwinHello);
    });
}
```

<span data-ttu-id="10dd1-126">È possibile configurare altre azioni da eseguire all'interno della pipeline OWIN.</span><span class="sxs-lookup"><span data-stu-id="10dd1-126">You can configure other actions to take place within the OWIN pipeline.</span></span>

> [!NOTE]
> <span data-ttu-id="10dd1-127">Le intestazioni di risposta devono essere modificate solo prima della prima scrittura nel flusso di risposta.</span><span class="sxs-lookup"><span data-stu-id="10dd1-127">Response headers should only be modified prior to the first write to the response stream.</span></span>

> [!NOTE]
> <span data-ttu-id="10dd1-128">Le chiamate multiple al metodo `UseOwin` sono sconsigliate per non compromettere le prestazioni.</span><span class="sxs-lookup"><span data-stu-id="10dd1-128">Multiple calls to `UseOwin` is discouraged for performance reasons.</span></span> <span data-ttu-id="10dd1-129">I componenti OWIN funzionano meglio se raggruppati insieme.</span><span class="sxs-lookup"><span data-stu-id="10dd1-129">OWIN components will operate best if grouped together.</span></span>

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

## <a name="run-aspnet-core-on-an-owin-based-server-and-use-its-websockets-support"></a><span data-ttu-id="10dd1-130">Eseguire ASP.NET Core in un server basato su OWIN e usare il supporto di WebSocket</span><span class="sxs-lookup"><span data-stu-id="10dd1-130">Run ASP.NET Core on an OWIN-based server and use its WebSockets support</span></span>

<span data-ttu-id="10dd1-131">Un altro esempio di funzionalità dei server basati su OWIN che può essere sfruttata da ASP.NET Core è l'accesso a funzionalità quali i WebSocket.</span><span class="sxs-lookup"><span data-stu-id="10dd1-131">Another example of how OWIN-based servers' features can be leveraged by ASP.NET Core is access to features like WebSockets.</span></span> <span data-ttu-id="10dd1-132">Il server Web OWIN per .NET usato nell'esempio precedente ha il supporto per i WebSocket incorporato che può essere usato da un'applicazione ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="10dd1-132">The .NET OWIN web server used in the previous example has support for Web Sockets built in, which can be leveraged by an ASP.NET Core application.</span></span> <span data-ttu-id="10dd1-133">L'esempio seguente illustra una semplice app Web che supporta i WebSocket e restituisce tutto quanto inviato al server tramite WebSocket.</span><span class="sxs-lookup"><span data-stu-id="10dd1-133">The example below shows a simple web app that supports Web Sockets and echoes back everything sent to the server through WebSockets.</span></span>

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

## <a name="owin-environment"></a><span data-ttu-id="10dd1-134">Ambiente OWIN</span><span class="sxs-lookup"><span data-stu-id="10dd1-134">OWIN environment</span></span>

<span data-ttu-id="10dd1-135">È possibile creare un ambiente OWIN tramite `HttpContext`.</span><span class="sxs-lookup"><span data-stu-id="10dd1-135">You can construct an OWIN environment using the `HttpContext`.</span></span>

```csharp

   var environment = new OwinEnvironment(HttpContext);
   var features = new OwinFeatureCollection(environment);
   ```

## <a name="owin-keys"></a><span data-ttu-id="10dd1-136">Chiavi OWIN</span><span class="sxs-lookup"><span data-stu-id="10dd1-136">OWIN keys</span></span>

<span data-ttu-id="10dd1-137">OWIN dipende da un oggetto `IDictionary<string,object>` per comunicare informazioni attraverso uno scambio di richiesta/risposta HTTP.</span><span class="sxs-lookup"><span data-stu-id="10dd1-137">OWIN depends on an `IDictionary<string,object>` object to communicate information throughout an HTTP Request/Response exchange.</span></span> <span data-ttu-id="10dd1-138">ASP.NET Core implementa le chiavi elencate di seguito.</span><span class="sxs-lookup"><span data-stu-id="10dd1-138">ASP.NET Core implements the keys listed below.</span></span> <span data-ttu-id="10dd1-139">Vedere le [specifiche principali e le estensioni](https://owin.org/#spec), nonché la pagina [OWIN Key Guidelines and Common Keys](https://owin.org/spec/spec/CommonKeys.html) (Linee guida sulle chiavi OWIN e chiavi comuni).</span><span class="sxs-lookup"><span data-stu-id="10dd1-139">See the [primary specification, extensions](https://owin.org/#spec), and [OWIN Key Guidelines and Common Keys](https://owin.org/spec/spec/CommonKeys.html).</span></span>

### <a name="request-data-owin-v100"></a><span data-ttu-id="10dd1-140">Dati della richiesta (OWIN versione 1.0.0)</span><span class="sxs-lookup"><span data-stu-id="10dd1-140">Request data (OWIN v1.0.0)</span></span>

| <span data-ttu-id="10dd1-141">Chiave</span><span class="sxs-lookup"><span data-stu-id="10dd1-141">Key</span></span>               | <span data-ttu-id="10dd1-142">Valore (tipo)</span><span class="sxs-lookup"><span data-stu-id="10dd1-142">Value (type)</span></span> | <span data-ttu-id="10dd1-143">Descrizione</span><span class="sxs-lookup"><span data-stu-id="10dd1-143">Description</span></span> |
| ----------------- | ------------ | ----------- |
| <span data-ttu-id="10dd1-144">owin.RequestScheme</span><span class="sxs-lookup"><span data-stu-id="10dd1-144">owin.RequestScheme</span></span> | `String` |  |
| <span data-ttu-id="10dd1-145">owin.RequestMethod</span><span class="sxs-lookup"><span data-stu-id="10dd1-145">owin.RequestMethod</span></span>  | `String` | |    
| <span data-ttu-id="10dd1-146">owin.RequestPathBase</span><span class="sxs-lookup"><span data-stu-id="10dd1-146">owin.RequestPathBase</span></span>  | `String` | |    
| <span data-ttu-id="10dd1-147">owin.RequestPath</span><span class="sxs-lookup"><span data-stu-id="10dd1-147">owin.RequestPath</span></span> | `String` | |     
| <span data-ttu-id="10dd1-148">owin.RequestQueryString</span><span class="sxs-lookup"><span data-stu-id="10dd1-148">owin.RequestQueryString</span></span>  | `String` | |    
| <span data-ttu-id="10dd1-149">owin.RequestProtocol</span><span class="sxs-lookup"><span data-stu-id="10dd1-149">owin.RequestProtocol</span></span>  | `String` | |    
| <span data-ttu-id="10dd1-150">owin.RequestHeaders</span><span class="sxs-lookup"><span data-stu-id="10dd1-150">owin.RequestHeaders</span></span> | `IDictionary<string,string[]>`  | |
| <span data-ttu-id="10dd1-151">owin.RequestBody</span><span class="sxs-lookup"><span data-stu-id="10dd1-151">owin.RequestBody</span></span> | `Stream`  | |

### <a name="request-data-owin-v110"></a><span data-ttu-id="10dd1-152">Dati della richiesta (OWIN versione 1.1.0)</span><span class="sxs-lookup"><span data-stu-id="10dd1-152">Request data (OWIN v1.1.0)</span></span>

| <span data-ttu-id="10dd1-153">Chiave</span><span class="sxs-lookup"><span data-stu-id="10dd1-153">Key</span></span>               | <span data-ttu-id="10dd1-154">Valore (tipo)</span><span class="sxs-lookup"><span data-stu-id="10dd1-154">Value (type)</span></span> | <span data-ttu-id="10dd1-155">Descrizione</span><span class="sxs-lookup"><span data-stu-id="10dd1-155">Description</span></span> |
| ----------------- | ------------ | ----------- |
| <span data-ttu-id="10dd1-156">owin.RequestId</span><span class="sxs-lookup"><span data-stu-id="10dd1-156">owin.RequestId</span></span> | `String` | <span data-ttu-id="10dd1-157">Facoltativo</span><span class="sxs-lookup"><span data-stu-id="10dd1-157">Optional</span></span> |

### <a name="response-data-owin-v100"></a><span data-ttu-id="10dd1-158">Dati della risposta (OWIN versione 1.0.0)</span><span class="sxs-lookup"><span data-stu-id="10dd1-158">Response data (OWIN v1.0.0)</span></span>

| <span data-ttu-id="10dd1-159">Chiave</span><span class="sxs-lookup"><span data-stu-id="10dd1-159">Key</span></span>               | <span data-ttu-id="10dd1-160">Valore (tipo)</span><span class="sxs-lookup"><span data-stu-id="10dd1-160">Value (type)</span></span> | <span data-ttu-id="10dd1-161">Descrizione</span><span class="sxs-lookup"><span data-stu-id="10dd1-161">Description</span></span> |
| ----------------- | ------------ | ----------- |
| <span data-ttu-id="10dd1-162">owin.ResponseStatusCode</span><span class="sxs-lookup"><span data-stu-id="10dd1-162">owin.ResponseStatusCode</span></span> | `int` | <span data-ttu-id="10dd1-163">Facoltativo</span><span class="sxs-lookup"><span data-stu-id="10dd1-163">Optional</span></span> |
| <span data-ttu-id="10dd1-164">owin.ResponseReasonPhrase</span><span class="sxs-lookup"><span data-stu-id="10dd1-164">owin.ResponseReasonPhrase</span></span> | `String` | <span data-ttu-id="10dd1-165">Facoltativo</span><span class="sxs-lookup"><span data-stu-id="10dd1-165">Optional</span></span> |
| <span data-ttu-id="10dd1-166">owin.ResponseHeaders</span><span class="sxs-lookup"><span data-stu-id="10dd1-166">owin.ResponseHeaders</span></span> | `IDictionary<string,string[]>`  | |
| <span data-ttu-id="10dd1-167">owin.ResponseBody</span><span class="sxs-lookup"><span data-stu-id="10dd1-167">owin.ResponseBody</span></span> | `Stream`  | |

### <a name="other-data-owin-v100"></a><span data-ttu-id="10dd1-168">Altri dati (OWIN versione 1.0.0)</span><span class="sxs-lookup"><span data-stu-id="10dd1-168">Other data (OWIN v1.0.0)</span></span>

| <span data-ttu-id="10dd1-169">Chiave</span><span class="sxs-lookup"><span data-stu-id="10dd1-169">Key</span></span>               | <span data-ttu-id="10dd1-170">Valore (tipo)</span><span class="sxs-lookup"><span data-stu-id="10dd1-170">Value (type)</span></span> | <span data-ttu-id="10dd1-171">Descrizione</span><span class="sxs-lookup"><span data-stu-id="10dd1-171">Description</span></span> |
| ----------------- | ------------ | ----------- |
| <span data-ttu-id="10dd1-172">owin.CallCancelled</span><span class="sxs-lookup"><span data-stu-id="10dd1-172">owin.CallCancelled</span></span> | `CancellationToken` |  |
| <span data-ttu-id="10dd1-173">owin.Version</span><span class="sxs-lookup"><span data-stu-id="10dd1-173">owin.Version</span></span>  | `String` | |   

### <a name="common-keys"></a><span data-ttu-id="10dd1-174">Chiavi comuni</span><span class="sxs-lookup"><span data-stu-id="10dd1-174">Common keys</span></span>

| <span data-ttu-id="10dd1-175">Chiave</span><span class="sxs-lookup"><span data-stu-id="10dd1-175">Key</span></span>               | <span data-ttu-id="10dd1-176">Valore (tipo)</span><span class="sxs-lookup"><span data-stu-id="10dd1-176">Value (type)</span></span> | <span data-ttu-id="10dd1-177">Descrizione</span><span class="sxs-lookup"><span data-stu-id="10dd1-177">Description</span></span> |
| ----------------- | ------------ | ----------- |
| <span data-ttu-id="10dd1-178">ssl.ClientCertificate</span><span class="sxs-lookup"><span data-stu-id="10dd1-178">ssl.ClientCertificate</span></span> | `X509Certificate` |  |
| <span data-ttu-id="10dd1-179">ssl.LoadClientCertAsync</span><span class="sxs-lookup"><span data-stu-id="10dd1-179">ssl.LoadClientCertAsync</span></span>  | `Func<Task>` | |    
| <span data-ttu-id="10dd1-180">server.RemoteIpAddress</span><span class="sxs-lookup"><span data-stu-id="10dd1-180">server.RemoteIpAddress</span></span>  | `String` | |    
| <span data-ttu-id="10dd1-181">server.RemotePort</span><span class="sxs-lookup"><span data-stu-id="10dd1-181">server.RemotePort</span></span> | `String` | |     
| <span data-ttu-id="10dd1-182">server.LocalIpAddress</span><span class="sxs-lookup"><span data-stu-id="10dd1-182">server.LocalIpAddress</span></span>  | `String` | |    
| <span data-ttu-id="10dd1-183">server.LocalPort</span><span class="sxs-lookup"><span data-stu-id="10dd1-183">server.LocalPort</span></span>  | `String` | |    
| <span data-ttu-id="10dd1-184">server.IsLocal</span><span class="sxs-lookup"><span data-stu-id="10dd1-184">server.IsLocal</span></span>  | `bool` | |    
| <span data-ttu-id="10dd1-185">server.OnSendingHeaders</span><span class="sxs-lookup"><span data-stu-id="10dd1-185">server.OnSendingHeaders</span></span>  | `Action<Action<object>,object>` | |

### <a name="sendfiles-v030"></a><span data-ttu-id="10dd1-186">SendFiles v0.3.0</span><span class="sxs-lookup"><span data-stu-id="10dd1-186">SendFiles v0.3.0</span></span>

| <span data-ttu-id="10dd1-187">Chiave</span><span class="sxs-lookup"><span data-stu-id="10dd1-187">Key</span></span>               | <span data-ttu-id="10dd1-188">Valore (tipo)</span><span class="sxs-lookup"><span data-stu-id="10dd1-188">Value (type)</span></span> | <span data-ttu-id="10dd1-189">Descrizione</span><span class="sxs-lookup"><span data-stu-id="10dd1-189">Description</span></span> |
| ----------------- | ------------ | ----------- |
| <span data-ttu-id="10dd1-190">sendfile.SendAsync</span><span class="sxs-lookup"><span data-stu-id="10dd1-190">sendfile.SendAsync</span></span> | <span data-ttu-id="10dd1-191">Vedere [Delegate signature](https://owin.org/spec/extensions/owin-SendFile-Extension-v0.3.0.htm) (Firma delegato)</span><span class="sxs-lookup"><span data-stu-id="10dd1-191">See [delegate signature](https://owin.org/spec/extensions/owin-SendFile-Extension-v0.3.0.htm)</span></span> | <span data-ttu-id="10dd1-192">Per richiesta</span><span class="sxs-lookup"><span data-stu-id="10dd1-192">Per Request</span></span> |

### <a name="opaque-v030"></a><span data-ttu-id="10dd1-193">Opaque v0.3.0</span><span class="sxs-lookup"><span data-stu-id="10dd1-193">Opaque v0.3.0</span></span>

| <span data-ttu-id="10dd1-194">Chiave</span><span class="sxs-lookup"><span data-stu-id="10dd1-194">Key</span></span>               | <span data-ttu-id="10dd1-195">Valore (tipo)</span><span class="sxs-lookup"><span data-stu-id="10dd1-195">Value (type)</span></span> | <span data-ttu-id="10dd1-196">Descrizione</span><span class="sxs-lookup"><span data-stu-id="10dd1-196">Description</span></span> |
| ----------------- | ------------ | ----------- |
| <span data-ttu-id="10dd1-197">opaque.Version</span><span class="sxs-lookup"><span data-stu-id="10dd1-197">opaque.Version</span></span> | `String` |  |
| <span data-ttu-id="10dd1-198">opaque.Upgrade</span><span class="sxs-lookup"><span data-stu-id="10dd1-198">opaque.Upgrade</span></span> | `OpaqueUpgrade` | <span data-ttu-id="10dd1-199">Vedere [Delegate signature](https://owin.org/spec/extensions/owin-SendFile-Extension-v0.3.0.htm) (Firma delegato)</span><span class="sxs-lookup"><span data-stu-id="10dd1-199">See [delegate signature](https://owin.org/spec/extensions/owin-SendFile-Extension-v0.3.0.htm)</span></span> |
| <span data-ttu-id="10dd1-200">opaque.Stream</span><span class="sxs-lookup"><span data-stu-id="10dd1-200">opaque.Stream</span></span> | `Stream` |  |
| <span data-ttu-id="10dd1-201">opaque.CallCancelled</span><span class="sxs-lookup"><span data-stu-id="10dd1-201">opaque.CallCancelled</span></span> | `CancellationToken` |  |

### <a name="websocket-v030"></a><span data-ttu-id="10dd1-202">WebSocket v0.3.0</span><span class="sxs-lookup"><span data-stu-id="10dd1-202">WebSocket v0.3.0</span></span>

| <span data-ttu-id="10dd1-203">Chiave</span><span class="sxs-lookup"><span data-stu-id="10dd1-203">Key</span></span>               | <span data-ttu-id="10dd1-204">Valore (tipo)</span><span class="sxs-lookup"><span data-stu-id="10dd1-204">Value (type)</span></span> | <span data-ttu-id="10dd1-205">Descrizione</span><span class="sxs-lookup"><span data-stu-id="10dd1-205">Description</span></span> |
| ----------------- | ------------ | ----------- |
| <span data-ttu-id="10dd1-206">websocket.Version</span><span class="sxs-lookup"><span data-stu-id="10dd1-206">websocket.Version</span></span> | `String` |  |
| <span data-ttu-id="10dd1-207">websocket.Accept</span><span class="sxs-lookup"><span data-stu-id="10dd1-207">websocket.Accept</span></span> | `WebSocketAccept` | <span data-ttu-id="10dd1-208">Vedere [Delegate signature](https://owin.org/spec/extensions/owin-SendFile-Extension-v0.3.0.htm) (Firma delegato)</span><span class="sxs-lookup"><span data-stu-id="10dd1-208">See [delegate signature](https://owin.org/spec/extensions/owin-SendFile-Extension-v0.3.0.htm)</span></span> |
| <span data-ttu-id="10dd1-209">websocket.AcceptAlt</span><span class="sxs-lookup"><span data-stu-id="10dd1-209">websocket.AcceptAlt</span></span> |  | <span data-ttu-id="10dd1-210">Non specificata</span><span class="sxs-lookup"><span data-stu-id="10dd1-210">Non-spec</span></span> |
| <span data-ttu-id="10dd1-211">websocket.SubProtocol</span><span class="sxs-lookup"><span data-stu-id="10dd1-211">websocket.SubProtocol</span></span> | `String` | <span data-ttu-id="10dd1-212">Vedere [RFC6455 sezione 4.2.2](https://tools.ietf.org/html/rfc6455#section-4.2.2) passaggio 5.5</span><span class="sxs-lookup"><span data-stu-id="10dd1-212">See [RFC6455 Section 4.2.2](https://tools.ietf.org/html/rfc6455#section-4.2.2) Step 5.5</span></span> |
| <span data-ttu-id="10dd1-213">websocket.SendAsync</span><span class="sxs-lookup"><span data-stu-id="10dd1-213">websocket.SendAsync</span></span> | `WebSocketSendAsync` | <span data-ttu-id="10dd1-214">Vedere [Delegate signature](https://owin.org/spec/extensions/owin-SendFile-Extension-v0.3.0.htm) (Firma delegato)</span><span class="sxs-lookup"><span data-stu-id="10dd1-214">See [delegate signature](https://owin.org/spec/extensions/owin-SendFile-Extension-v0.3.0.htm)</span></span>  |
| <span data-ttu-id="10dd1-215">websocket.ReceiveAsync</span><span class="sxs-lookup"><span data-stu-id="10dd1-215">websocket.ReceiveAsync</span></span> | `WebSocketReceiveAsync` | <span data-ttu-id="10dd1-216">Vedere [Delegate signature](https://owin.org/spec/extensions/owin-SendFile-Extension-v0.3.0.htm) (Firma delegato)</span><span class="sxs-lookup"><span data-stu-id="10dd1-216">See [delegate signature](https://owin.org/spec/extensions/owin-SendFile-Extension-v0.3.0.htm)</span></span>  |
| <span data-ttu-id="10dd1-217">websocket.CloseAsync</span><span class="sxs-lookup"><span data-stu-id="10dd1-217">websocket.CloseAsync</span></span> | `WebSocketCloseAsync` | <span data-ttu-id="10dd1-218">Vedere [Delegate signature](https://owin.org/spec/extensions/owin-SendFile-Extension-v0.3.0.htm) (Firma delegato)</span><span class="sxs-lookup"><span data-stu-id="10dd1-218">See [delegate signature](https://owin.org/spec/extensions/owin-SendFile-Extension-v0.3.0.htm)</span></span>  |
| <span data-ttu-id="10dd1-219">websocket.CallCancelled</span><span class="sxs-lookup"><span data-stu-id="10dd1-219">websocket.CallCancelled</span></span> | `CancellationToken` |  |
| <span data-ttu-id="10dd1-220">websocket.ClientCloseStatus</span><span class="sxs-lookup"><span data-stu-id="10dd1-220">websocket.ClientCloseStatus</span></span> | `int` | <span data-ttu-id="10dd1-221">Facoltativo</span><span class="sxs-lookup"><span data-stu-id="10dd1-221">Optional</span></span> |
| <span data-ttu-id="10dd1-222">websocket.ClientCloseDescription</span><span class="sxs-lookup"><span data-stu-id="10dd1-222">websocket.ClientCloseDescription</span></span> | `String` | <span data-ttu-id="10dd1-223">Facoltativo</span><span class="sxs-lookup"><span data-stu-id="10dd1-223">Optional</span></span> |

## <a name="additional-resources"></a><span data-ttu-id="10dd1-224">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="10dd1-224">Additional resources</span></span>

* [<span data-ttu-id="10dd1-225">Middleware</span><span class="sxs-lookup"><span data-stu-id="10dd1-225">Middleware</span></span>](xref:fundamentals/middleware/index)
* [<span data-ttu-id="10dd1-226">Server</span><span class="sxs-lookup"><span data-stu-id="10dd1-226">Servers</span></span>](xref:fundamentals/servers/index)
