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
# <a name="configure-options-for-the-aspnet-core-kestrel-web-server"></a><span data-ttu-id="d8912-103">Configurare le opzioni per il server Web ASP.NET Core gheppio</span><span class="sxs-lookup"><span data-stu-id="d8912-103">Configure options for the ASP.NET Core Kestrel web server</span></span>

<span data-ttu-id="d8912-104">Il server Web Kestrel dispone di opzioni di configurazione dei vincoli che risultano particolarmente utili nelle distribuzioni con connessione Internet.</span><span class="sxs-lookup"><span data-stu-id="d8912-104">The Kestrel web server has constraint configuration options that are especially useful in Internet-facing deployments.</span></span>

<span data-ttu-id="d8912-105">Per fornire una configurazione aggiuntiva dopo la chiamata di <xref:Microsoft.Extensions.Hosting.GenericHostBuilderExtensions.ConfigureWebHostDefaults%2A>, usare <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderKestrelExtensions.ConfigureKestrel%2A>:</span><span class="sxs-lookup"><span data-stu-id="d8912-105">To provide additional configuration after calling <xref:Microsoft.Extensions.Hosting.GenericHostBuilderExtensions.ConfigureWebHostDefaults%2A>, use <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderKestrelExtensions.ConfigureKestrel%2A>:</span></span>

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

<span data-ttu-id="d8912-106">Impostare i vincoli per la proprietà <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Limits> della classe <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions>.</span><span class="sxs-lookup"><span data-stu-id="d8912-106">Set constraints on the <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Limits> property of the <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions> class.</span></span> <span data-ttu-id="d8912-107">La proprietà `Limits` contiene un'istanza della classe <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits>.</span><span class="sxs-lookup"><span data-stu-id="d8912-107">The `Limits` property holds an instance of the <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits> class.</span></span>

<span data-ttu-id="d8912-108">Negli esempi seguenti viene usato lo spazio dei nomi <xref:Microsoft.AspNetCore.Server.Kestrel.Core>:</span><span class="sxs-lookup"><span data-stu-id="d8912-108">The following examples use the <xref:Microsoft.AspNetCore.Server.Kestrel.Core> namespace:</span></span>

```csharp
using Microsoft.AspNetCore.Server.Kestrel.Core;
```

<span data-ttu-id="d8912-109">Negli esempi illustrati più avanti in questo articolo, le opzioni di gheppio sono configurate nel codice C#.</span><span class="sxs-lookup"><span data-stu-id="d8912-109">In examples shown later in this article, Kestrel options are configured in C# code.</span></span> <span data-ttu-id="d8912-110">È inoltre possibile impostare le opzioni di Gheppio utilizzando un [provider di configurazione](xref:fundamentals/configuration/index).</span><span class="sxs-lookup"><span data-stu-id="d8912-110">Kestrel options can also be set using a [configuration provider](xref:fundamentals/configuration/index).</span></span> <span data-ttu-id="d8912-111">Il [provider di configurazione file](xref:fundamentals/configuration/index#file-configuration-provider) , ad esempio, può caricare la configurazione di Gheppio da un `appsettings.json` `appsettings.{Environment}.json` file o:</span><span class="sxs-lookup"><span data-stu-id="d8912-111">For example, the [File Configuration Provider](xref:fundamentals/configuration/index#file-configuration-provider) can load Kestrel configuration from an `appsettings.json` or `appsettings.{Environment}.json` file:</span></span>

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
> <span data-ttu-id="d8912-112"><xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions> e la [configurazione dell'endpoint](xref:fundamentals/servers/kestrel/endpoints) sono configurabili dai provider di configurazione.</span><span class="sxs-lookup"><span data-stu-id="d8912-112"><xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions> and [endpoint configuration](xref:fundamentals/servers/kestrel/endpoints) are configurable from configuration providers.</span></span> <span data-ttu-id="d8912-113">La configurazione del gheppio rimanente deve essere configurata nel codice C#.</span><span class="sxs-lookup"><span data-stu-id="d8912-113">Remaining Kestrel configuration must be configured in C# code.</span></span>

<span data-ttu-id="d8912-114">Usare **uno** degli approcci seguenti:</span><span class="sxs-lookup"><span data-stu-id="d8912-114">Use **one** of the following approaches:</span></span>

* <span data-ttu-id="d8912-115">Configurare gheppio in `Startup.ConfigureServices` :</span><span class="sxs-lookup"><span data-stu-id="d8912-115">Configure Kestrel in `Startup.ConfigureServices`:</span></span>

  1. <span data-ttu-id="d8912-116">Inserire un'istanza di `IConfiguration` nella `Startup` classe.</span><span class="sxs-lookup"><span data-stu-id="d8912-116">Inject an instance of `IConfiguration` into the `Startup` class.</span></span> <span data-ttu-id="d8912-117">Nell'esempio seguente si presuppone che la configurazione inserita venga assegnata alla `Configuration` Proprietà.</span><span class="sxs-lookup"><span data-stu-id="d8912-117">The following example assumes that the injected configuration is assigned to the `Configuration` property.</span></span>
  2. <span data-ttu-id="d8912-118">In `Startup.ConfigureServices` caricare la `Kestrel` sezione della configurazione nella configurazione di Gheppio:</span><span class="sxs-lookup"><span data-stu-id="d8912-118">In `Startup.ConfigureServices`, load the `Kestrel` section of configuration into Kestrel's configuration:</span></span>

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

* <span data-ttu-id="d8912-119">Configurare il gheppio durante la compilazione dell'host:</span><span class="sxs-lookup"><span data-stu-id="d8912-119">Configure Kestrel when building the host:</span></span>

  <span data-ttu-id="d8912-120">In `Program.cs` caricare la `Kestrel` sezione della configurazione nella configurazione di Gheppio:</span><span class="sxs-lookup"><span data-stu-id="d8912-120">In `Program.cs`, load the `Kestrel` section of configuration into Kestrel's configuration:</span></span>

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

<span data-ttu-id="d8912-121">Entrambi gli approcci precedenti funzionano con qualsiasi [provider di configurazione](xref:fundamentals/configuration/index).</span><span class="sxs-lookup"><span data-stu-id="d8912-121">Both of the preceding approaches work with any [configuration provider](xref:fundamentals/configuration/index).</span></span>

## <a name="general-limits"></a><span data-ttu-id="d8912-122">Limiti generali</span><span class="sxs-lookup"><span data-stu-id="d8912-122">General limits</span></span>

### <a name="keep-alive-timeout"></a><span data-ttu-id="d8912-123">Timeout keep-alive</span><span class="sxs-lookup"><span data-stu-id="d8912-123">Keep-alive timeout</span></span>

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.KeepAliveTimeout>

<span data-ttu-id="d8912-124">Ottiene o imposta il [timeout keep-alive](https://tools.ietf.org/html/rfc7230#section-6.5).</span><span class="sxs-lookup"><span data-stu-id="d8912-124">Gets or sets the [keep-alive timeout](https://tools.ietf.org/html/rfc7230#section-6.5).</span></span> <span data-ttu-id="d8912-125">Il valore predefinito è 2 minuti.</span><span class="sxs-lookup"><span data-stu-id="d8912-125">Defaults to 2 minutes.</span></span>

[!code-csharp[](samples/3.x/KestrelSample/Program.cs?name=snippet_Limits&highlight=19-20)]

### <a name="maximum-client-connections"></a><span data-ttu-id="d8912-126">Numero massimo di connessioni client</span><span class="sxs-lookup"><span data-stu-id="d8912-126">Maximum client connections</span></span>

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.MaxConcurrentConnections><br>
<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.MaxConcurrentUpgradedConnections>

<span data-ttu-id="d8912-127">È possibile impostare il numero massimo di connessioni TCP aperte simultaneamente per l'intera app con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="d8912-127">The maximum number of concurrent open TCP connections can be set for the entire app with the following code:</span></span>

[!code-csharp[](samples/3.x/KestrelSample/Program.cs?name=snippet_Limits&highlight=3)]

<span data-ttu-id="d8912-128">È previsto un limite separato per le connessioni che sono state aggiornate da HTTP o HTTPS a un altro protocollo (ad esempio su una richiesta WebSocket).</span><span class="sxs-lookup"><span data-stu-id="d8912-128">There's a separate limit for connections that have been upgraded from HTTP or HTTPS to another protocol (for example, on a WebSockets request).</span></span> <span data-ttu-id="d8912-129">Dopo l'aggiornamento di una connessione, questa non viene conteggiata per il limite `MaxConcurrentConnections`.</span><span class="sxs-lookup"><span data-stu-id="d8912-129">After a connection is upgraded, it isn't counted against the `MaxConcurrentConnections` limit.</span></span>

[!code-csharp[](samples/3.x/KestrelSample/Program.cs?name=snippet_Limits&highlight=4)]

<span data-ttu-id="d8912-130">Per impostazione predefinita, il numero massimo di connessioni è illimitato (null).</span><span class="sxs-lookup"><span data-stu-id="d8912-130">The maximum number of connections is unlimited (null) by default.</span></span>

### <a name="maximum-request-body-size"></a><span data-ttu-id="d8912-131">Dimensione massima del corpo della richiesta</span><span class="sxs-lookup"><span data-stu-id="d8912-131">Maximum request body size</span></span>

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.MaxRequestBodySize>

<span data-ttu-id="d8912-132">La dimensione massima predefinita del corpo della richiesta è pari a 30.000.000 di byte, equivalenti a circa 28,6 MB.</span><span class="sxs-lookup"><span data-stu-id="d8912-132">The default maximum request body size is 30,000,000 bytes, which is approximately 28.6 MB.</span></span>

<span data-ttu-id="d8912-133">Il metodo consigliato per ignorare il limite in un'app ASP.NET Core MVC è l'uso dell'attributo <xref:Microsoft.AspNetCore.Mvc.RequestSizeLimitAttribute> in un metodo di azione:</span><span class="sxs-lookup"><span data-stu-id="d8912-133">The recommended approach to override the limit in an ASP.NET Core MVC app is to use the <xref:Microsoft.AspNetCore.Mvc.RequestSizeLimitAttribute> attribute on an action method:</span></span>

```csharp
[RequestSizeLimit(100000000)]
public IActionResult MyActionMethod()
```

<span data-ttu-id="d8912-134">Nell'esempio seguente viene illustrato come configurare il vincolo per l'app in ogni richiesta:</span><span class="sxs-lookup"><span data-stu-id="d8912-134">The following example shows how to configure the constraint for the app on every request:</span></span>

[!code-csharp[](samples/3.x/KestrelSample/Program.cs?name=snippet_Limits&highlight=5)]

<span data-ttu-id="d8912-135">Eseguire l'override dell'impostazione per una richiesta specifica nel middleware:</span><span class="sxs-lookup"><span data-stu-id="d8912-135">Override the setting on a specific request in middleware:</span></span>

[!code-csharp[](samples/3.x/KestrelSample/Startup.cs?name=snippet_Limits&highlight=3-4)]

<span data-ttu-id="d8912-136">Viene generata un'eccezione se l'app configura il limite per una richiesta dopo che l'app ha iniziato a leggere la richiesta.</span><span class="sxs-lookup"><span data-stu-id="d8912-136">An exception is thrown if the app configures the limit on a request after the app has started to read the request.</span></span> <span data-ttu-id="d8912-137">Una proprietà `IsReadOnly` indica se la proprietà `MaxRequestBodySize` è in stato di sola lettura e pertanto è troppo tardi per configurare il limite.</span><span class="sxs-lookup"><span data-stu-id="d8912-137">There's an `IsReadOnly` property that indicates if the `MaxRequestBodySize` property is in read-only state, meaning it's too late to configure the limit.</span></span>

<span data-ttu-id="d8912-138">Quando un'app è in esecuzione [out-of-process](xref:host-and-deploy/iis/index#out-of-process-hosting-model) dietro il [modulo ASP.NET Core](xref:host-and-deploy/aspnet-core-module), il limite delle dimensioni del corpo della richiesta del gheppio è disabilitato.</span><span class="sxs-lookup"><span data-stu-id="d8912-138">When an app runs [out-of-process](xref:host-and-deploy/iis/index#out-of-process-hosting-model) behind the [ASP.NET Core Module](xref:host-and-deploy/aspnet-core-module), Kestrel's request body size limit is disabled.</span></span> <span data-ttu-id="d8912-139">IIS imposta già il limite.</span><span class="sxs-lookup"><span data-stu-id="d8912-139">IIS already sets the limit.</span></span>

### <a name="minimum-request-body-data-rate"></a><span data-ttu-id="d8912-140">Velocità minima dei dati del corpo della richiesta</span><span class="sxs-lookup"><span data-stu-id="d8912-140">Minimum request body data rate</span></span>

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.MinRequestBodyDataRate><br>
<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.MinResponseDataRate>

<span data-ttu-id="d8912-141">Kestrel controlla ogni secondo se i dati arrivano alla velocità in byte al secondo specificata.</span><span class="sxs-lookup"><span data-stu-id="d8912-141">Kestrel checks every second if data is arriving at the specified rate in bytes/second.</span></span> <span data-ttu-id="d8912-142">Se la frequenza scende sotto il valore minimo, si è verificato il timeout della connessione. Il periodo di tolleranza è la quantità di tempo in cui il gheppio consente al client di aumentare la velocità di invio fino al minimo.</span><span class="sxs-lookup"><span data-stu-id="d8912-142">If the rate drops below the minimum, the connection is timed out. The grace period is the amount of time Kestrel allows the client to increase its send rate up to the minimum.</span></span> <span data-ttu-id="d8912-143">La frequenza non viene controllata durante tale periodo di tempo.</span><span class="sxs-lookup"><span data-stu-id="d8912-143">The rate isn't checked during that time.</span></span> <span data-ttu-id="d8912-144">Il periodo di tolleranza consente di evitare l'eliminazione delle connessioni che inviano inizialmente dati a una velocità ridotta a causa dell'avvio lento di TCP.</span><span class="sxs-lookup"><span data-stu-id="d8912-144">The grace period helps avoid dropping connections that are initially sending data at a slow rate because of TCP slow-start.</span></span>

<span data-ttu-id="d8912-145">La frequenza minima predefinita è 240 byte al secondo con un periodo di tolleranza di 5 secondi.</span><span class="sxs-lookup"><span data-stu-id="d8912-145">The default minimum rate is 240 bytes/second with a 5-second grace period.</span></span>

<span data-ttu-id="d8912-146">Anche per la risposta è prevista una velocità minima.</span><span class="sxs-lookup"><span data-stu-id="d8912-146">A minimum rate also applies to the response.</span></span> <span data-ttu-id="d8912-147">Il codice per impostare il limite della richiesta e della risposta è identico e varia solo per `RequestBody` o `Response` nei nomi di proprietà e di interfaccia.</span><span class="sxs-lookup"><span data-stu-id="d8912-147">The code to set the request limit and the response limit is the same except for having `RequestBody` or `Response` in the property and interface names.</span></span>

<span data-ttu-id="d8912-148">Di seguito è riportato un esempio in cui viene illustrato come configurare le tariffe minime per i dati in `Program.cs` :</span><span class="sxs-lookup"><span data-stu-id="d8912-148">Here's an example that shows how to configure the minimum data rates in `Program.cs`:</span></span>

[!code-csharp[](samples/3.x/KestrelSample/Program.cs?name=snippet_Limits&highlight=6-11)]

<span data-ttu-id="d8912-149">Ignorare i limiti di velocità minima per ogni richiesta nel middleware:</span><span class="sxs-lookup"><span data-stu-id="d8912-149">Override the minimum rate limits per request in middleware:</span></span>

[!code-csharp[](samples/3.x/KestrelSample/Startup.cs?name=snippet_Limits&highlight=6-21)]

<span data-ttu-id="d8912-150">L'oggetto a <xref:Microsoft.AspNetCore.Server.Kestrel.Core.Features.IHttpMinResponseDataRateFeature> cui si fa riferimento nell'esempio precedente non è presente nelle <xref:Microsoft.AspNetCore.Http.HttpContext.Features?displayProperty=nameWithType> richieste http/2.</span><span class="sxs-lookup"><span data-stu-id="d8912-150">The <xref:Microsoft.AspNetCore.Server.Kestrel.Core.Features.IHttpMinResponseDataRateFeature> referenced in the prior sample isn't present in <xref:Microsoft.AspNetCore.Http.HttpContext.Features?displayProperty=nameWithType> for HTTP/2 requests.</span></span> <span data-ttu-id="d8912-151">La modifica dei limiti di velocità in base alle singole richieste non è in genere supportata per HTTP/2 a causa del supporto del protocollo per il multiplexing delle richieste.</span><span class="sxs-lookup"><span data-stu-id="d8912-151">Modifying rate limits on a per-request basis isn't generally supported for HTTP/2 because of the protocol's support for request multiplexing.</span></span> <span data-ttu-id="d8912-152">La funzionalità <xref:Microsoft.AspNetCore.Server.Kestrel.Core.Features.IHttpMinRequestBodyDataRateFeature> è tuttavia ancora presente in `HttpContext.Features` per le richieste HTTP/2, perché il limite di velocità di lettura può comunque essere *disabilitato interamente* per ogni singola richiesta impostando <xref:Microsoft.AspNetCore.Server.Kestrel.Core.Features.IHttpMinResponseDataRateFeature.MinDataRate?displayProperty=nameWithType> su `null` anche per una richiesta HTTP/2.</span><span class="sxs-lookup"><span data-stu-id="d8912-152">However, the <xref:Microsoft.AspNetCore.Server.Kestrel.Core.Features.IHttpMinRequestBodyDataRateFeature> is still present `HttpContext.Features` for HTTP/2 requests, because the read rate limit can still be *disabled entirely* on a per-request basis by setting <xref:Microsoft.AspNetCore.Server.Kestrel.Core.Features.IHttpMinResponseDataRateFeature.MinDataRate?displayProperty=nameWithType> to `null` even for an HTTP/2 request.</span></span> <span data-ttu-id="d8912-153">Il tentativo di leggere `IHttpMinRequestBodyDataRateFeature.MinDataRate` o il tentativo di impostazione di un valore diverso da `null` causerà la generazione di una <xref:System.NotSupportedException> per una richiesta HTTP/2.</span><span class="sxs-lookup"><span data-stu-id="d8912-153">Attempting to read `IHttpMinRequestBodyDataRateFeature.MinDataRate` or attempting to set it to a value other than `null` will result in a <xref:System.NotSupportedException> being thrown given an HTTP/2 request.</span></span>

<span data-ttu-id="d8912-154">I limiti di velocità a livello di server configurati tramite <xref:Microsoft.AspNetCore.Server.Kestrel.KestrelServerOptions.Limits?displayProperty=nameWithType> vengono tuttavia applicati alle connessioni HTTP/1.x e HTTP/2.</span><span class="sxs-lookup"><span data-stu-id="d8912-154">Server-wide rate limits configured via <xref:Microsoft.AspNetCore.Server.Kestrel.KestrelServerOptions.Limits?displayProperty=nameWithType> still apply to both HTTP/1.x and HTTP/2 connections.</span></span>

### <a name="request-headers-timeout"></a><span data-ttu-id="d8912-155">Timeout delle intestazioni delle richieste</span><span class="sxs-lookup"><span data-stu-id="d8912-155">Request headers timeout</span></span>

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.RequestHeadersTimeout>

<span data-ttu-id="d8912-156">Ottiene o imposta la quantità massima di tempo che il server dedica alla ricezione delle intestazioni delle richieste.</span><span class="sxs-lookup"><span data-stu-id="d8912-156">Gets or sets the maximum amount of time the server spends receiving request headers.</span></span> <span data-ttu-id="d8912-157">Il valore predefinito è 30 secondi.</span><span class="sxs-lookup"><span data-stu-id="d8912-157">Defaults to 30 seconds.</span></span>

[!code-csharp[](samples/3.x/KestrelSample/Program.cs?name=snippet_Limits&highlight=21-22)]

## <a name="http2-limits"></a><span data-ttu-id="d8912-158">Limiti HTTP/2</span><span class="sxs-lookup"><span data-stu-id="d8912-158">HTTP/2 limits</span></span>

<span data-ttu-id="d8912-159">I limiti di questa sezione sono impostati su on <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.Http2?displayProperty=nameWithType> .</span><span class="sxs-lookup"><span data-stu-id="d8912-159">The limits in this section are set on <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.Http2?displayProperty=nameWithType>.</span></span>

### <a name="maximum-streams-per-connection"></a><span data-ttu-id="d8912-160">Numero massimo di flussi per connessione</span><span class="sxs-lookup"><span data-stu-id="d8912-160">Maximum streams per connection</span></span>

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.Http2Limits.MaxStreamsPerConnection>

<span data-ttu-id="d8912-161">Limita il numero di flussi di richieste simultanee per connessione HTTP/2.</span><span class="sxs-lookup"><span data-stu-id="d8912-161">Limits the number of concurrent request streams per HTTP/2 connection.</span></span> <span data-ttu-id="d8912-162">I flussi in eccesso vengono rifiutati.</span><span class="sxs-lookup"><span data-stu-id="d8912-162">Excess streams are refused.</span></span>

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.Limits.Http2.MaxStreamsPerConnection = 100;
});
```

<span data-ttu-id="d8912-163">Il valore predefinito è 100.</span><span class="sxs-lookup"><span data-stu-id="d8912-163">The default value is 100.</span></span>

### <a name="header-table-size"></a><span data-ttu-id="d8912-164">Dimensioni della tabella delle intestazioni</span><span class="sxs-lookup"><span data-stu-id="d8912-164">Header table size</span></span>

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.Http2Limits.HeaderTableSize>

<span data-ttu-id="d8912-165">Il decodificatore HPACK decomprime le intestazioni HTTP per le connessioni HTTP/2.</span><span class="sxs-lookup"><span data-stu-id="d8912-165">The HPACK decoder decompresses HTTP headers for HTTP/2 connections.</span></span> <span data-ttu-id="d8912-166">`HeaderTableSize` limita le dimensioni della tabella di compressione delle intestazioni usata dal decodificatore HPACK.</span><span class="sxs-lookup"><span data-stu-id="d8912-166">`HeaderTableSize` limits the size of the header compression table that the HPACK decoder uses.</span></span> <span data-ttu-id="d8912-167">Il valore viene specificato in ottetti e deve essere maggiore di zero (0).</span><span class="sxs-lookup"><span data-stu-id="d8912-167">The value is provided in octets and must be greater than zero (0).</span></span>

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.Limits.Http2.HeaderTableSize = 4096;
});
```

<span data-ttu-id="d8912-168">Il valore predefinito è 4096.</span><span class="sxs-lookup"><span data-stu-id="d8912-168">The default value is 4096.</span></span>

### <a name="maximum-frame-size"></a><span data-ttu-id="d8912-169">Dimensione massima del frame</span><span class="sxs-lookup"><span data-stu-id="d8912-169">Maximum frame size</span></span>

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.Http2Limits.MaxFrameSize>

<span data-ttu-id="d8912-170">Indica la dimensione massima consentita di un payload del frame di connessione HTTP/2 ricevuto o inviato dal server.</span><span class="sxs-lookup"><span data-stu-id="d8912-170">Indicates the maximum allowed size of an HTTP/2 connection frame payload received or sent by the server.</span></span> <span data-ttu-id="d8912-171">Il valore viene specificato in ottetti e deve essere compreso tra 2^14 (16.384) e 2^24-1 (16.777.215).</span><span class="sxs-lookup"><span data-stu-id="d8912-171">The value is provided in octets and must be between 2^14 (16,384) and 2^24-1 (16,777,215).</span></span>

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.Limits.Http2.MaxFrameSize = 16384;
});
```

<span data-ttu-id="d8912-172">Il valore predefinito è 2^14 (16.384).</span><span class="sxs-lookup"><span data-stu-id="d8912-172">The default value is 2^14 (16,384).</span></span>

### <a name="maximum-request-header-size"></a><span data-ttu-id="d8912-173">Dimensioni massime dell'intestazione della richiesta</span><span class="sxs-lookup"><span data-stu-id="d8912-173">Maximum request header size</span></span>

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.Http2Limits.MaxRequestHeaderFieldSize>

<span data-ttu-id="d8912-174">Indica la dimensione massima consentita in ottetti dei valori dell'intestazione della richiesta.</span><span class="sxs-lookup"><span data-stu-id="d8912-174">Indicates the maximum allowed size in octets of request header values.</span></span> <span data-ttu-id="d8912-175">Questo limite si applica sia al nome che al valore nelle rappresentazioni compresse e non compresse.</span><span class="sxs-lookup"><span data-stu-id="d8912-175">This limit applies to both name and value in their compressed and uncompressed representations.</span></span> <span data-ttu-id="d8912-176">Il valore deve essere maggiore di zero (0).</span><span class="sxs-lookup"><span data-stu-id="d8912-176">The value must be greater than zero (0).</span></span>

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.Limits.Http2.MaxRequestHeaderFieldSize = 8192;
});
```

<span data-ttu-id="d8912-177">Il valore predefinito è 8.192.</span><span class="sxs-lookup"><span data-stu-id="d8912-177">The default value is 8,192.</span></span>

### <a name="initial-connection-window-size"></a><span data-ttu-id="d8912-178">Dimensioni della finestra di connessione iniziali</span><span class="sxs-lookup"><span data-stu-id="d8912-178">Initial connection window size</span></span>

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.Http2Limits.InitialConnectionWindowSize>

<span data-ttu-id="d8912-179">Indica il numero massimo di dati del corpo della richiesta, in byte, del buffer del server contemporaneamente aggregati in tutte le richieste (flussi) per connessione.</span><span class="sxs-lookup"><span data-stu-id="d8912-179">Indicates the maximum request body data in bytes the server buffers at one time aggregated across all requests (streams) per connection.</span></span> <span data-ttu-id="d8912-180">Le richieste sono anche limitate da `Http2.InitialStreamWindowSize`.</span><span class="sxs-lookup"><span data-stu-id="d8912-180">Requests are also limited by `Http2.InitialStreamWindowSize`.</span></span> <span data-ttu-id="d8912-181">Il valore deve essere maggiore di o uguale a 65.535 e minore di 2^31 (2.147.483.648).</span><span class="sxs-lookup"><span data-stu-id="d8912-181">The value must be greater than or equal to 65,535 and less than 2^31 (2,147,483,648).</span></span>

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.Limits.Http2.InitialConnectionWindowSize = 131072;
});
```

<span data-ttu-id="d8912-182">Il valore predefinito è 128 KB (131.072).</span><span class="sxs-lookup"><span data-stu-id="d8912-182">The default value is 128 KB (131,072).</span></span>

### <a name="initial-stream-window-size"></a><span data-ttu-id="d8912-183">Dimensioni della finestra di flusso iniziali</span><span class="sxs-lookup"><span data-stu-id="d8912-183">Initial stream window size</span></span>

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.Http2Limits.InitialStreamWindowSize>

<span data-ttu-id="d8912-184">Indica il numero massimo di dati del corpo della richiesta in byte memorizzati nel buffer del server in una sola volta per richiesta (flusso).</span><span class="sxs-lookup"><span data-stu-id="d8912-184">Indicates the maximum request body data in bytes the server buffers at one time per request (stream).</span></span> <span data-ttu-id="d8912-185">Anche le richieste sono limitate da [`InitialConnectionWindowSize`](#initial-connection-window-size) .</span><span class="sxs-lookup"><span data-stu-id="d8912-185">Requests are also limited by [`InitialConnectionWindowSize`](#initial-connection-window-size).</span></span> <span data-ttu-id="d8912-186">Il valore deve essere maggiore di o uguale a 65.535 e minore di 2^31 (2.147.483.648).</span><span class="sxs-lookup"><span data-stu-id="d8912-186">The value must be greater than or equal to 65,535 and less than 2^31 (2,147,483,648).</span></span>

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.Limits.Http2.InitialStreamWindowSize = 98304;
});
```

<span data-ttu-id="d8912-187">Il valore predefinito è 96 KB (98.304).</span><span class="sxs-lookup"><span data-stu-id="d8912-187">The default value is 96 KB (98,304).</span></span>

### <a name="http2-keep-alive-ping-configuration"></a><span data-ttu-id="d8912-188">Configurazione del ping keep alive HTTP/2</span><span class="sxs-lookup"><span data-stu-id="d8912-188">HTTP/2 keep alive ping configuration</span></span>

<span data-ttu-id="d8912-189">Il gheppio può essere configurato per l'invio di ping HTTP/2 ai client connessi.</span><span class="sxs-lookup"><span data-stu-id="d8912-189">Kestrel can be configured to send HTTP/2 pings to connected clients.</span></span> <span data-ttu-id="d8912-190">I ping HTTP/2 servono più scopi:</span><span class="sxs-lookup"><span data-stu-id="d8912-190">HTTP/2 pings serve multiple purposes:</span></span>

* <span data-ttu-id="d8912-191">Mantieni attive le connessioni inattive.</span><span class="sxs-lookup"><span data-stu-id="d8912-191">Keep idle connections alive.</span></span> <span data-ttu-id="d8912-192">Alcuni client e server proxy chiudono le connessioni inattive.</span><span class="sxs-lookup"><span data-stu-id="d8912-192">Some clients and proxy servers close connections that are idle.</span></span> <span data-ttu-id="d8912-193">I ping HTTP/2 sono considerati attività su una connessione e impediscono la chiusura della connessione come inattiva.</span><span class="sxs-lookup"><span data-stu-id="d8912-193">HTTP/2 pings are considered as activity on a connection and prevent the connection from being closed as idle.</span></span>
* <span data-ttu-id="d8912-194">Chiudere le connessioni non integre.</span><span class="sxs-lookup"><span data-stu-id="d8912-194">Close unhealthy connections.</span></span> <span data-ttu-id="d8912-195">Le connessioni in cui il client non risponde al ping keep-alive nell'ora configurata vengono chiuse dal server.</span><span class="sxs-lookup"><span data-stu-id="d8912-195">Connections where the client doesn't respond to the keep alive ping in the configured time are closed by the server.</span></span>

<span data-ttu-id="d8912-196">Sono disponibili due opzioni di configurazione correlate a HTTP/2 ping Keep-Alive:</span><span class="sxs-lookup"><span data-stu-id="d8912-196">There are two configuration options related to HTTP/2 keep alive pings:</span></span>

* <span data-ttu-id="d8912-197"><xref:Microsoft.AspNetCore.Server.Kestrel.Core.Http2Limits.KeepAlivePingDelay> è un oggetto <xref:System.TimeSpan> che configura l'intervallo del ping.</span><span class="sxs-lookup"><span data-stu-id="d8912-197"><xref:Microsoft.AspNetCore.Server.Kestrel.Core.Http2Limits.KeepAlivePingDelay> is a <xref:System.TimeSpan> that configures the ping interval.</span></span> <span data-ttu-id="d8912-198">Il server invia al client un ping Keep-Alive se non riceve frame per questo periodo di tempo.</span><span class="sxs-lookup"><span data-stu-id="d8912-198">The server sends a keep alive ping to the client if it doesn't receive any frames for this period of time.</span></span> <span data-ttu-id="d8912-199">Quando questa opzione è impostata su, i ping Keep-Alive sono disabilitati <xref:System.TimeSpan.MaxValue?displayProperty=nameWithType> .</span><span class="sxs-lookup"><span data-stu-id="d8912-199">Keep alive pings are disabled when this option is set to <xref:System.TimeSpan.MaxValue?displayProperty=nameWithType>.</span></span> <span data-ttu-id="d8912-200">Il valore predefinito è <xref:System.TimeSpan.MaxValue?displayProperty=nameWithType>.</span><span class="sxs-lookup"><span data-stu-id="d8912-200">The default value is <xref:System.TimeSpan.MaxValue?displayProperty=nameWithType>.</span></span>
* <span data-ttu-id="d8912-201"><xref:Microsoft.AspNetCore.Server.Kestrel.Core.Http2Limits.KeepAlivePingTimeout> è un oggetto <xref:System.TimeSpan> che configura il timeout del ping.</span><span class="sxs-lookup"><span data-stu-id="d8912-201"><xref:Microsoft.AspNetCore.Server.Kestrel.Core.Http2Limits.KeepAlivePingTimeout> is a <xref:System.TimeSpan> that configures the ping timeout.</span></span> <span data-ttu-id="d8912-202">Se il server non riceve frame, ad esempio un ping di risposta, durante questo timeout la connessione viene chiusa.</span><span class="sxs-lookup"><span data-stu-id="d8912-202">If the server doesn't receive any frames, such as a response ping, during this timeout then the connection is closed.</span></span> <span data-ttu-id="d8912-203">Quando questa opzione è impostata su, il timeout keep alive è disabilitato <xref:System.TimeSpan.MaxValue?displayProperty=nameWithType> .</span><span class="sxs-lookup"><span data-stu-id="d8912-203">Keep alive timeout is disabled when this option is set to <xref:System.TimeSpan.MaxValue?displayProperty=nameWithType>.</span></span> <span data-ttu-id="d8912-204">Il valore predefinito è 20 secondi.</span><span class="sxs-lookup"><span data-stu-id="d8912-204">The default value is 20 seconds.</span></span>

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.Limits.Http2.KeepAlivePingDelay = TimeSpan.FromSeconds(30);
    serverOptions.Limits.Http2.KeepAlivePingTimeout = TimeSpan.FromSeconds(60);
});
```

## <a name="other-options"></a><span data-ttu-id="d8912-205">Altre opzioni</span><span class="sxs-lookup"><span data-stu-id="d8912-205">Other options</span></span>

### <a name="synchronous-io"></a><span data-ttu-id="d8912-206">I/O sincrono</span><span class="sxs-lookup"><span data-stu-id="d8912-206">Synchronous I/O</span></span>

<span data-ttu-id="d8912-207"><xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.AllowSynchronousIO> Controlla se è consentito l'I/O sincrono per la richiesta e la risposta.</span><span class="sxs-lookup"><span data-stu-id="d8912-207"><xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.AllowSynchronousIO> controls whether synchronous I/O is allowed for the request and response.</span></span> <span data-ttu-id="d8912-208">Il valore predefinito è `false`.</span><span class="sxs-lookup"><span data-stu-id="d8912-208">The default value is `false`.</span></span>

> [!WARNING]
> <span data-ttu-id="d8912-209">Un numero elevato di operazioni di I/O sincrone bloccanti può causare l'esaurimento del pool di thread, il che rende l'app non risponde.</span><span class="sxs-lookup"><span data-stu-id="d8912-209">A large number of blocking synchronous I/O operations can lead to thread pool starvation, which makes the app unresponsive.</span></span> <span data-ttu-id="d8912-210">Abilitare solo `AllowSynchronousIO` quando si usa una libreria che non supporta l'I/O asincrono.</span><span class="sxs-lookup"><span data-stu-id="d8912-210">Only enable `AllowSynchronousIO` when using a library that doesn't support asynchronous I/O.</span></span>

<span data-ttu-id="d8912-211">L'esempio seguente abilita l'I/O sincrono:</span><span class="sxs-lookup"><span data-stu-id="d8912-211">The following example enables synchronous I/O:</span></span>

[!code-csharp[](samples/5.x/KestrelSample/Program.cs?name=snippet_SyncIO)]

<span data-ttu-id="d8912-212">Per informazioni su altre opzioni e limiti di Kestrel, vedere:</span><span class="sxs-lookup"><span data-stu-id="d8912-212">For information about other Kestrel options and limits, see:</span></span>

* <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions>
* <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits>
* <xref:Microsoft.AspNetCore.Server.Kestrel.Core.ListenOptions>
