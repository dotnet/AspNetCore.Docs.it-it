---
title: Blazor SignalR Guida ASP.NET Core
author: guardrex
description: Informazioni su come configurare e gestire le Blazor SignalR connessioni.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 02/25/2021
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
uid: blazor/fundamentals/signalr
zone_pivot_groups: blazor-hosting-models
ms.openlocfilehash: 63dfd93fbc42a869211bc5cd481a8dbee6eb6c91
ms.sourcegitcommit: 3982ff9dabb5b12aeb0a61cde2686b5253364f5d
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/04/2021
ms.locfileid: "102118915"
---
# <a name="aspnet-core-blazor-signalr-guidance"></a><span data-ttu-id="09c85-103">Blazor SignalR Guida ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="09c85-103">ASP.NET Core Blazor SignalR guidance</span></span>

::: zone pivot="webassembly"

<span data-ttu-id="09c85-104">Questo articolo illustra come configurare e gestire le SignalR connessioni nelle Blazor app.</span><span class="sxs-lookup"><span data-stu-id="09c85-104">This article explains how to configure and manage SignalR connections in Blazor apps.</span></span>

<span data-ttu-id="09c85-105">Per indicazioni generali sulla SignalR configurazione di ASP.NET Core, vedere gli argomenti nella <xref:signalr/introduction> sezione della documentazione relativa a.</span><span class="sxs-lookup"><span data-stu-id="09c85-105">For general guidance on ASP.NET Core SignalR configuration, see the topics in the <xref:signalr/introduction> area of the documentation.</span></span> <span data-ttu-id="09c85-106">Per configurare SignalR [aggiunto a una Blazor WebAssembly soluzione ospitata](xref:tutorials/signalr-blazor), vedere <xref:signalr/configuration#configure-server-options> .</span><span class="sxs-lookup"><span data-stu-id="09c85-106">To configure SignalR [added to a hosted Blazor WebAssembly solution](xref:tutorials/signalr-blazor), see <xref:signalr/configuration#configure-server-options>.</span></span>

## <a name="signalr-cross-origin-negotiation-for-authentication"></a><span data-ttu-id="09c85-107">SignalR negoziazione tra le origini per l'autenticazione</span><span class="sxs-lookup"><span data-stu-id="09c85-107">SignalR cross-origin negotiation for authentication</span></span>

<span data-ttu-id="09c85-108">Per configurare SignalR il client sottostante per l'invio di credenziali, ad esempio le cookie intestazioni s o http Authentication:</span><span class="sxs-lookup"><span data-stu-id="09c85-108">To configure SignalR's underlying client to send credentials, such as cookies or HTTP authentication headers:</span></span>

* <span data-ttu-id="09c85-109">Usare <xref:Microsoft.AspNetCore.Components.WebAssembly.Http.WebAssemblyHttpRequestMessageExtensions.SetBrowserRequestCredentials%2A> per impostare <xref:Microsoft.AspNetCore.Components.WebAssembly.Http.BrowserRequestCredentials.Include> [`fetch`](https://developer.mozilla.org/docs/Web/API/Fetch_API/Using_Fetch) le richieste tra le origini.</span><span class="sxs-lookup"><span data-stu-id="09c85-109">Use <xref:Microsoft.AspNetCore.Components.WebAssembly.Http.WebAssemblyHttpRequestMessageExtensions.SetBrowserRequestCredentials%2A> to set <xref:Microsoft.AspNetCore.Components.WebAssembly.Http.BrowserRequestCredentials.Include> on cross-origin [`fetch`](https://developer.mozilla.org/docs/Web/API/Fetch_API/Using_Fetch) requests.</span></span>

  <span data-ttu-id="09c85-110">`IncludeRequestCredentialsMessageHandler.cs`:</span><span class="sxs-lookup"><span data-stu-id="09c85-110">`IncludeRequestCredentialsMessageHandler.cs`:</span></span>

  ```csharp
  using System.Net.Http;
  using System.Threading;
  using System.Threading.Tasks;
  using Microsoft.AspNetCore.Components.WebAssembly.Http;

  public class IncludeRequestCredentialsMessageHandler : DelegatingHandler
  {
      protected override Task<HttpResponseMessage> SendAsync(
          HttpRequestMessage request, CancellationToken cancellationToken)
      {
          request.SetBrowserRequestCredentials(BrowserRequestCredentials.Include);
          return base.SendAsync(request, cancellationToken);
      }
  }
  ```

* <span data-ttu-id="09c85-111">Quando viene compilata una connessione all'hub, assegnare <xref:System.Net.Http.HttpMessageHandler> a l' <xref:Microsoft.AspNetCore.Http.Connections.Client.HttpConnectionOptions.HttpMessageHandlerFactory> opzione:</span><span class="sxs-lookup"><span data-stu-id="09c85-111">Where a hub connection is built, assign the <xref:System.Net.Http.HttpMessageHandler> to the <xref:Microsoft.AspNetCore.Http.Connections.Client.HttpConnectionOptions.HttpMessageHandlerFactory> option:</span></span>

  ```csharp
  HubConnectionBuilder hubConnecton;

  ...

  hubConnecton = new HubConnectionBuilder()
      .WithUrl(new Uri(NavigationManager.ToAbsoluteUri("/chathub")), options =>
      {
          options.HttpMessageHandlerFactory = innerHandler => 
              new IncludeRequestCredentialsMessageHandler { InnerHandler = innerHandler };
      }).Build();
  ```

  <span data-ttu-id="09c85-112">Nell'esempio precedente viene configurato l'URL di connessione dell'hub all'indirizzo URI assoluto in `/chathub` , ovvero l'URL usato nell' [ SignalR Blazor esercitazione with](xref:tutorials/signalr-blazor) nel `Index` componente ( `Pages/Index.razor` ).</span><span class="sxs-lookup"><span data-stu-id="09c85-112">The preceding example configures the hub connection URL to the absolute URI address at `/chathub`, which is the URL used in the [SignalR with Blazor tutorial](xref:tutorials/signalr-blazor) in the `Index` component (`Pages/Index.razor`).</span></span> <span data-ttu-id="09c85-113">L'URI può essere impostato anche tramite una stringa, ad esempio `https://signalr.example.com` o tramite la [configurazione](xref:blazor/fundamentals/configuration).</span><span class="sxs-lookup"><span data-stu-id="09c85-113">The URI can also be set via a string, for example `https://signalr.example.com`, or via [configuration](xref:blazor/fundamentals/configuration).</span></span>

<span data-ttu-id="09c85-114">Per altre informazioni, vedere <xref:signalr/configuration#configure-additional-options>.</span><span class="sxs-lookup"><span data-stu-id="09c85-114">For more information, see <xref:signalr/configuration#configure-additional-options>.</span></span>

::: moniker range=">= aspnetcore-5.0"

## <a name="render-mode"></a><span data-ttu-id="09c85-115">Modalità di rendering</span><span class="sxs-lookup"><span data-stu-id="09c85-115">Render mode</span></span>

<span data-ttu-id="09c85-116">Se un' Blazor WebAssembly app che usa SignalR viene configurata per eseguire il prerendering sul server, il prerendering si verifica prima che venga stabilita la connessione client al server.</span><span class="sxs-lookup"><span data-stu-id="09c85-116">If a Blazor WebAssembly app that uses SignalR is configured to prerender on the server, prerendering occurs before the client connection to the server is established.</span></span> <span data-ttu-id="09c85-117">Per altre informazioni, vedere gli articoli seguenti:</span><span class="sxs-lookup"><span data-stu-id="09c85-117">For more information, see the following articles:</span></span>

* <xref:mvc/views/tag-helpers/builtin-th/component-tag-helper>
* <xref:blazor/components/prerendering-and-integration>

::: moniker-end

## <a name="additional-resources"></a><span data-ttu-id="09c85-118">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="09c85-118">Additional resources</span></span>

* <xref:signalr/introduction>
* <xref:signalr/configuration>

::: zone-end

::: zone pivot="server"

<span data-ttu-id="09c85-119">Questo articolo illustra come configurare e gestire le SignalR connessioni nelle Blazor app.</span><span class="sxs-lookup"><span data-stu-id="09c85-119">This article explains how to configure and manage SignalR connections in Blazor apps.</span></span>

<span data-ttu-id="09c85-120">Per indicazioni generali sulla SignalR configurazione di ASP.NET Core, vedere gli argomenti nella <xref:signalr/introduction> sezione della documentazione relativa a.</span><span class="sxs-lookup"><span data-stu-id="09c85-120">For general guidance on ASP.NET Core SignalR configuration, see the topics in the <xref:signalr/introduction> area of the documentation.</span></span> <span data-ttu-id="09c85-121">Per configurare SignalR [aggiunto a una Blazor WebAssembly soluzione ospitata](xref:tutorials/signalr-blazor), vedere <xref:signalr/configuration#configure-server-options> .</span><span class="sxs-lookup"><span data-stu-id="09c85-121">To configure SignalR [added to a hosted Blazor WebAssembly solution](xref:tutorials/signalr-blazor), see <xref:signalr/configuration#configure-server-options>.</span></span>

## <a name="circuit-handler-options"></a><span data-ttu-id="09c85-122">Opzioni del gestore del circuito</span><span class="sxs-lookup"><span data-stu-id="09c85-122">Circuit handler options</span></span>

<span data-ttu-id="09c85-123">Configurare il Blazor Server circuito con il <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions> illustrato nella tabella seguente.</span><span class="sxs-lookup"><span data-stu-id="09c85-123">Configure the Blazor Server circuit with the <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions> shown in the following table.</span></span>

| <span data-ttu-id="09c85-124">Opzione</span><span class="sxs-lookup"><span data-stu-id="09c85-124">Option</span></span> | <span data-ttu-id="09c85-125">Predefinito</span><span class="sxs-lookup"><span data-stu-id="09c85-125">Default</span></span> | <span data-ttu-id="09c85-126">Descrizione</span><span class="sxs-lookup"><span data-stu-id="09c85-126">Description</span></span> |
| --- | --- | --- |
| <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions.DetailedErrors> | `false` | <span data-ttu-id="09c85-127">Invia messaggi di eccezione dettagliati a JavaScript quando si verifica un'eccezione non gestita nel circuito o quando una chiamata a un metodo .NET tramite l'interoperabilità JS genera un'eccezione.</span><span class="sxs-lookup"><span data-stu-id="09c85-127">Send detailed exception messages to JavaScript when an unhandled exception occurs on the circuit or when a .NET method invocation through JS interop results in an exception.</span></span> |
| <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions.DisconnectedCircuitMaxRetained> | <span data-ttu-id="09c85-128">100</span><span class="sxs-lookup"><span data-stu-id="09c85-128">100</span></span> | <span data-ttu-id="09c85-129">Numero massimo di circuiti disconnessi che il server utilizza in memoria per volta.</span><span class="sxs-lookup"><span data-stu-id="09c85-129">Maximum number of disconnected circuits that the server holds in memory at a time.</span></span> |
| <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions.DisconnectedCircuitRetentionPeriod> | <span data-ttu-id="09c85-130">3 minuti</span><span class="sxs-lookup"><span data-stu-id="09c85-130">3 minutes</span></span> | <span data-ttu-id="09c85-131">Quantità massima di tempo durante il quale un circuito disconnesso viene mantenuto in memoria prima di essere eliminato.</span><span class="sxs-lookup"><span data-stu-id="09c85-131">Maximum amount of time a disconnected circuit is held in memory before being torn down.</span></span> |
| <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions.JSInteropDefaultCallTimeout> | <span data-ttu-id="09c85-132">1 minuto</span><span class="sxs-lookup"><span data-stu-id="09c85-132">1 minute</span></span> | <span data-ttu-id="09c85-133">Quantità massima di tempo di attesa del server prima del timeout di una chiamata asincrona alla funzione JavaScript.</span><span class="sxs-lookup"><span data-stu-id="09c85-133">Maximum amount of time the server waits before timing out an asynchronous JavaScript function invocation.</span></span> |
| <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions.MaxBufferedUnacknowledgedRenderBatches> | <span data-ttu-id="09c85-134">10</span><span class="sxs-lookup"><span data-stu-id="09c85-134">10</span></span> | <span data-ttu-id="09c85-135">Numero massimo di batch di rendering non riconosciuti che il server mantiene in memoria per ogni circuito in un determinato momento per supportare una riconnessione affidabile.</span><span class="sxs-lookup"><span data-stu-id="09c85-135">Maximum number of unacknowledged render batches the server keeps in memory per circuit at a given time to support robust reconnection.</span></span> <span data-ttu-id="09c85-136">Dopo aver raggiunto il limite, il server smette di produrre nuovi batch di rendering finché uno o più batch non vengono riconosciuti dal client.</span><span class="sxs-lookup"><span data-stu-id="09c85-136">After reaching the limit, the server stops producing new render batches until one or more batches are acknowledged by the client.</span></span> |

<span data-ttu-id="09c85-137">Configurare le opzioni in `Startup.ConfigureServices` con un delegato di opzioni a <xref:Microsoft.Extensions.DependencyInjection.ComponentServiceCollectionExtensions.AddServerSideBlazor%2A> .</span><span class="sxs-lookup"><span data-stu-id="09c85-137">Configure the options in `Startup.ConfigureServices` with an options delegate to <xref:Microsoft.Extensions.DependencyInjection.ComponentServiceCollectionExtensions.AddServerSideBlazor%2A>.</span></span> <span data-ttu-id="09c85-138">Nell'esempio seguente vengono assegnati i valori predefiniti delle opzioni indicati nella tabella precedente.</span><span class="sxs-lookup"><span data-stu-id="09c85-138">The following example assigns the default option values shown in the preceding table.</span></span> <span data-ttu-id="09c85-139">Verificare che `Startup.cs` utilizzi lo <xref:System> spazio dei nomi ( `using System;` ).</span><span class="sxs-lookup"><span data-stu-id="09c85-139">Confirm that `Startup.cs` uses the <xref:System> namespace (`using System;`).</span></span>

<span data-ttu-id="09c85-140">`Startup.ConfigureServices`:</span><span class="sxs-lookup"><span data-stu-id="09c85-140">`Startup.ConfigureServices`:</span></span>

```csharp
services.AddServerSideBlazor(options =>
{
    options.DetailedErrors = false;
    options.DisconnectedCircuitMaxRetained = 100;
    options.DisconnectedCircuitRetentionPeriod = TimeSpan.FromMinutes(3);
    options.JSInteropDefaultCallTimeout = TimeSpan.FromMinutes(1);
    options.MaxBufferedUnacknowledgedRenderBatches = 10;
});
```

<span data-ttu-id="09c85-141">Per configurare <xref:Microsoft.AspNetCore.SignalR.HubConnectionContext> , usare <xref:Microsoft.AspNetCore.SignalR.HubConnectionContextOptions> con <xref:Microsoft.Extensions.DependencyInjection.ServerSideBlazorBuilderExtensions.AddHubOptions%2A> .</span><span class="sxs-lookup"><span data-stu-id="09c85-141">To configure the <xref:Microsoft.AspNetCore.SignalR.HubConnectionContext>, use <xref:Microsoft.AspNetCore.SignalR.HubConnectionContextOptions> with <xref:Microsoft.Extensions.DependencyInjection.ServerSideBlazorBuilderExtensions.AddHubOptions%2A>.</span></span> <span data-ttu-id="09c85-142">Per le descrizioni delle opzioni, vedere <xref:signalr/configuration#configure-server-options> .</span><span class="sxs-lookup"><span data-stu-id="09c85-142">For option descriptions, see <xref:signalr/configuration#configure-server-options>.</span></span> <span data-ttu-id="09c85-143">Nell'esempio seguente vengono assegnati i valori delle opzioni predefinite.</span><span class="sxs-lookup"><span data-stu-id="09c85-143">The following example assigns the default option values.</span></span> <span data-ttu-id="09c85-144">Verificare che `Startup.cs` utilizzi lo <xref:System> spazio dei nomi ( `using System;` ).</span><span class="sxs-lookup"><span data-stu-id="09c85-144">Confirm that `Startup.cs` uses the <xref:System> namespace (`using System;`).</span></span>

<span data-ttu-id="09c85-145">`Startup.ConfigureServices`:</span><span class="sxs-lookup"><span data-stu-id="09c85-145">`Startup.ConfigureServices`:</span></span>

```csharp
services.AddServerSideBlazor()
    .AddHubOptions(options =>
    {
        options.ClientTimeoutInterval = TimeSpan.FromSeconds(30);
        options.EnableDetailedErrors = false;
        options.HandshakeTimeout = TimeSpan.FromSeconds(15);
        options.KeepAliveInterval = TimeSpan.FromSeconds(15);
        options.MaximumParallelInvocationsPerClient = 1;
        options.MaximumReceiveMessageSize = 32 * 1024;
        options.StreamBufferCapacity = 10;
    });
```

## <a name="reflect-the-connection-state-in-the-ui"></a><span data-ttu-id="09c85-146">Riflette lo stato di connessione nell'interfaccia utente</span><span class="sxs-lookup"><span data-stu-id="09c85-146">Reflect the connection state in the UI</span></span>

<span data-ttu-id="09c85-147">Quando il client rileva che la connessione è stata persa, viene visualizzata un'interfaccia utente predefinita quando il client tenta di riconnettersi.</span><span class="sxs-lookup"><span data-stu-id="09c85-147">When the client detects that the connection has been lost, a default UI is displayed to the user while the client attempts to reconnect.</span></span> <span data-ttu-id="09c85-148">Se la riconnessione non riesce, all'utente viene offerta l'opzione per riprovare.</span><span class="sxs-lookup"><span data-stu-id="09c85-148">If reconnection fails, the user is provided the option to retry.</span></span>

<span data-ttu-id="09c85-149">Per personalizzare l'interfaccia utente, definire un elemento con un valore `id` di `components-reconnect-modal` nell'oggetto `<body>` della `_Host.cshtml` Razor pagina.</span><span class="sxs-lookup"><span data-stu-id="09c85-149">To customize the UI, define an element with an `id` of `components-reconnect-modal` in the `<body>` of the `_Host.cshtml` Razor page.</span></span>

<span data-ttu-id="09c85-150">`Pages/_Host.cshtml`:</span><span class="sxs-lookup"><span data-stu-id="09c85-150">`Pages/_Host.cshtml`:</span></span>

```cshtml
<div id="components-reconnect-modal">
    ...
</div>
```

<span data-ttu-id="09c85-151">Aggiungere gli stili CSS seguenti al foglio di stile del sito.</span><span class="sxs-lookup"><span data-stu-id="09c85-151">Add the following CSS styles to the site's stylesheet.</span></span>

<span data-ttu-id="09c85-152">`wwwroot/css/site.css`:</span><span class="sxs-lookup"><span data-stu-id="09c85-152">`wwwroot/css/site.css`:</span></span>

```css
#components-reconnect-modal {
    display: none;
}

#components-reconnect-modal.components-reconnect-show {
    display: block;
}
```

<span data-ttu-id="09c85-153">Nella tabella seguente vengono descritte le classi CSS applicate all' `components-reconnect-modal` elemento dal Blazor Framework.</span><span class="sxs-lookup"><span data-stu-id="09c85-153">The following table describes the CSS classes applied to the `components-reconnect-modal` element by the Blazor framework.</span></span>

| <span data-ttu-id="09c85-154">Classe CSS</span><span class="sxs-lookup"><span data-stu-id="09c85-154">CSS class</span></span>                       | <span data-ttu-id="09c85-155">Indica&hellip;</span><span class="sxs-lookup"><span data-stu-id="09c85-155">Indicates&hellip;</span></span> |
| ------------------------------- | ----------------- |
| `components-reconnect-show`     | <span data-ttu-id="09c85-156">Connessione persa.</span><span class="sxs-lookup"><span data-stu-id="09c85-156">A lost connection.</span></span> <span data-ttu-id="09c85-157">È in corso un tentativo di riconnessione del client.</span><span class="sxs-lookup"><span data-stu-id="09c85-157">The client is attempting to reconnect.</span></span> <span data-ttu-id="09c85-158">Mostra il modale.</span><span class="sxs-lookup"><span data-stu-id="09c85-158">Show the modal.</span></span> |
| `components-reconnect-hide`     | <span data-ttu-id="09c85-159">Viene ristabilita una connessione attiva al server.</span><span class="sxs-lookup"><span data-stu-id="09c85-159">An active connection is re-established to the server.</span></span> <span data-ttu-id="09c85-160">Nascondere il modale.</span><span class="sxs-lookup"><span data-stu-id="09c85-160">Hide the modal.</span></span> |
| `components-reconnect-failed`   | <span data-ttu-id="09c85-161">Riconnessione non riuscita. probabilmente a causa di un errore di rete.</span><span class="sxs-lookup"><span data-stu-id="09c85-161">Reconnection failed, probably due to a network failure.</span></span> <span data-ttu-id="09c85-162">Per tentare la riconnessione, chiamare `window.Blazor.reconnect()` in JavaScript.</span><span class="sxs-lookup"><span data-stu-id="09c85-162">To attempt reconnection, call `window.Blazor.reconnect()` in JavaScript.</span></span> |
| `components-reconnect-rejected` | <span data-ttu-id="09c85-163">Riconnessione rifiutata.</span><span class="sxs-lookup"><span data-stu-id="09c85-163">Reconnection rejected.</span></span> <span data-ttu-id="09c85-164">Il server è stato raggiunto ma ha rifiutato la connessione e lo stato dell'utente nel server è andato perso.</span><span class="sxs-lookup"><span data-stu-id="09c85-164">The server was reached but refused the connection, and the user's state on the server is lost.</span></span> <span data-ttu-id="09c85-165">Per ricaricare l'app, chiamare `location.reload()` in JavaScript.</span><span class="sxs-lookup"><span data-stu-id="09c85-165">To reload the app, call `location.reload()` in JavaScript.</span></span> <span data-ttu-id="09c85-166">Questo stato di connessione può verificarsi nei casi seguenti:</span><span class="sxs-lookup"><span data-stu-id="09c85-166">This connection state may result when:</span></span><ul><li><span data-ttu-id="09c85-167">Si verifica un arresto anomalo del circuito sul lato server.</span><span class="sxs-lookup"><span data-stu-id="09c85-167">A crash in the server-side circuit occurs.</span></span></li><li><span data-ttu-id="09c85-168">Il client viene disconnesso abbastanza a lungo da consentire al server di eliminare lo stato dell'utente.</span><span class="sxs-lookup"><span data-stu-id="09c85-168">The client is disconnected long enough for the server to drop the user's state.</span></span> <span data-ttu-id="09c85-169">Le istanze dei componenti dell'utente sono state eliminate.</span><span class="sxs-lookup"><span data-stu-id="09c85-169">Instances of the user's components are disposed.</span></span></li><li><span data-ttu-id="09c85-170">Il server viene riavviato oppure il processo di lavoro dell'app viene riciclato.</span><span class="sxs-lookup"><span data-stu-id="09c85-170">The server is restarted, or the app's worker process is recycled.</span></span></li></ul> |

## <a name="render-mode"></a><span data-ttu-id="09c85-171">Modalità di rendering</span><span class="sxs-lookup"><span data-stu-id="09c85-171">Render mode</span></span>

<span data-ttu-id="09c85-172">Per impostazione predefinita, Blazor Server le app preeseguono il rendering dell'interfaccia utente nel server prima che venga stabilita la connessione client al server.</span><span class="sxs-lookup"><span data-stu-id="09c85-172">By default, Blazor Server apps prerender the UI on the server before the client connection to the server is established.</span></span> <span data-ttu-id="09c85-173">Per altre informazioni, vedere <xref:mvc/views/tag-helpers/builtin-th/component-tag-helper>.</span><span class="sxs-lookup"><span data-stu-id="09c85-173">For more information, see <xref:mvc/views/tag-helpers/builtin-th/component-tag-helper>.</span></span>

## <a name="initialize-the-blazor-circuit"></a><span data-ttu-id="09c85-174">Inizializzare il Blazor circuito</span><span class="sxs-lookup"><span data-stu-id="09c85-174">Initialize the Blazor circuit</span></span>

<span data-ttu-id="09c85-175">Configurare l'avvio manuale del Blazor Server [ SignalR circuito](xref:blazor/hosting-models#circuits) di un'app nel `Pages/_Host.cshtml` file:</span><span class="sxs-lookup"><span data-stu-id="09c85-175">Configure the manual start of a Blazor Server app's [SignalR circuit](xref:blazor/hosting-models#circuits) in the `Pages/_Host.cshtml` file:</span></span>

* <span data-ttu-id="09c85-176">Aggiungere un `autostart="false"` attributo al `<script>` tag per lo `blazor.server.js` script.</span><span class="sxs-lookup"><span data-stu-id="09c85-176">Add an `autostart="false"` attribute to the `<script>` tag for the `blazor.server.js` script.</span></span>
* <span data-ttu-id="09c85-177">Inserire uno script che chiama `Blazor.start` dopo il `blazor.server.js` tag dello script e all'interno del `</body>` tag di chiusura.</span><span class="sxs-lookup"><span data-stu-id="09c85-177">Place a script that calls `Blazor.start` after the `blazor.server.js` script's tag and inside the closing `</body>` tag.</span></span>

<span data-ttu-id="09c85-178">Quando `autostart` è disabilitato, qualsiasi aspetto dell'app che non dipende dal circuito funziona normalmente.</span><span class="sxs-lookup"><span data-stu-id="09c85-178">When `autostart` is disabled, any aspect of the app that doesn't depend on the circuit works normally.</span></span> <span data-ttu-id="09c85-179">Il routing lato client, ad esempio, è operativo.</span><span class="sxs-lookup"><span data-stu-id="09c85-179">For example, client-side routing is operational.</span></span> <span data-ttu-id="09c85-180">Tuttavia, qualsiasi aspetto che dipende dal circuito non è operativo fino a quando non `Blazor.start` viene chiamato il metodo.</span><span class="sxs-lookup"><span data-stu-id="09c85-180">However, any aspect that depends on the circuit isn't operational until `Blazor.start` is called.</span></span> <span data-ttu-id="09c85-181">Il comportamento dell'app è imprevedibile senza un circuito stabilito.</span><span class="sxs-lookup"><span data-stu-id="09c85-181">App behavior is unpredictable without an established circuit.</span></span> <span data-ttu-id="09c85-182">Ad esempio, i metodi dei componenti non vengono eseguiti mentre il circuito è disconnesso.</span><span class="sxs-lookup"><span data-stu-id="09c85-182">For example, component methods fail to execute while the circuit is disconnected.</span></span>

### <a name="initialize-blazor-when-the-document-is-ready"></a><span data-ttu-id="09c85-183">Inizializza Blazor quando il documento è pronto</span><span class="sxs-lookup"><span data-stu-id="09c85-183">Initialize Blazor when the document is ready</span></span>

<span data-ttu-id="09c85-184">`Pages/_Host.cshtml`:</span><span class="sxs-lookup"><span data-stu-id="09c85-184">`Pages/_Host.cshtml`:</span></span>

```cshtml
<body>
    ...

    <script autostart="false" src="_framework/blazor.server.js"></script>
    <script>
      document.addEventListener("DOMContentLoaded", function() {
        Blazor.start();
      });
    </script>
</body>
```

### <a name="chain-to-the-promise-that-results-from-a-manual-start"></a><span data-ttu-id="09c85-185">Concatenare a `Promise` che risulta da un avvio manuale</span><span class="sxs-lookup"><span data-stu-id="09c85-185">Chain to the `Promise` that results from a manual start</span></span>

<span data-ttu-id="09c85-186">Per eseguire attività aggiuntive, ad esempio l'inizializzazione dell'interoperabilità JS, usare [`then`](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Promise/then) per concatenare al [`Promise`](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Promise) risultante dall'avvio di un' Blazor app manuale.</span><span class="sxs-lookup"><span data-stu-id="09c85-186">To perform additional tasks, such as JS interop initialization, use [`then`](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Promise/then) to chain to the [`Promise`](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Promise) that results from a manual Blazor app start.</span></span>

<span data-ttu-id="09c85-187">`Pages/_Host.cshtml`:</span><span class="sxs-lookup"><span data-stu-id="09c85-187">`Pages/_Host.cshtml`:</span></span>

```cshtml
<body>
    ...

    <script autostart="false" src="_framework/blazor.server.js"></script>
    <script>
      Blazor.start().then(function () {
        ...
      });
    </script>
</body>
```

### <a name="configure-signalr-client-logging"></a><span data-ttu-id="09c85-188">Configurare la SignalR registrazione client</span><span class="sxs-lookup"><span data-stu-id="09c85-188">Configure SignalR client logging</span></span>

<span data-ttu-id="09c85-189">Nel generatore client passare l' `configureSignalR` oggetto di configurazione che chiama `configureLogging` con il livello di registrazione.</span><span class="sxs-lookup"><span data-stu-id="09c85-189">On the client builder, pass in the `configureSignalR` configuration object that calls `configureLogging` with the log level.</span></span>

<span data-ttu-id="09c85-190">`Pages/_Host.cshtml`:</span><span class="sxs-lookup"><span data-stu-id="09c85-190">`Pages/_Host.cshtml`:</span></span>

```cshtml
<body>
    ...

    <script autostart="false" src="_framework/blazor.server.js"></script>
    <script>
      Blazor.start({
        configureSignalR: function (builder) {
          builder.configureLogging("information");
        }
      });
    </script>
</body>
```

<span data-ttu-id="09c85-191">Nell'esempio precedente, `information` è equivalente a un livello di log di <xref:Microsoft.Extensions.Logging.LogLevel.Information?displayProperty=nameWithType> .</span><span class="sxs-lookup"><span data-stu-id="09c85-191">In the preceding example, `information` is equivalent to a log level of <xref:Microsoft.Extensions.Logging.LogLevel.Information?displayProperty=nameWithType>.</span></span>

### <a name="modify-the-reconnection-handler"></a><span data-ttu-id="09c85-192">Modificare il gestore di riconnessione</span><span class="sxs-lookup"><span data-stu-id="09c85-192">Modify the reconnection handler</span></span>

<span data-ttu-id="09c85-193">Gli eventi di connessione del circuito del gestore di riconnessione possono essere modificati per comportamenti personalizzati, ad esempio:</span><span class="sxs-lookup"><span data-stu-id="09c85-193">The reconnection handler's circuit connection events can be modified for custom behaviors, such as:</span></span>

* <span data-ttu-id="09c85-194">Per inviare una notifica all'utente se la connessione viene eliminata.</span><span class="sxs-lookup"><span data-stu-id="09c85-194">To notify the user if the connection is dropped.</span></span>
* <span data-ttu-id="09c85-195">Per eseguire la registrazione (dal client) quando si connette un circuito.</span><span class="sxs-lookup"><span data-stu-id="09c85-195">To perform logging (from the client) when a circuit is connected.</span></span>

<span data-ttu-id="09c85-196">Per modificare gli eventi di connessione, registrare i callback per le seguenti modifiche della connessione:</span><span class="sxs-lookup"><span data-stu-id="09c85-196">To modify the connection events, register callbacks for the following connection changes:</span></span>

* <span data-ttu-id="09c85-197">Connessioni eliminate utilizzare `onConnectionDown` .</span><span class="sxs-lookup"><span data-stu-id="09c85-197">Dropped connections use `onConnectionDown`.</span></span>
* <span data-ttu-id="09c85-198">Le connessioni stabilite/ristabilite usano `onConnectionUp` .</span><span class="sxs-lookup"><span data-stu-id="09c85-198">Established/re-established connections use `onConnectionUp`.</span></span>

<span data-ttu-id="09c85-199">**`onConnectionDown` `onConnectionUp` È necessario specificare sia che.**</span><span class="sxs-lookup"><span data-stu-id="09c85-199">**Both `onConnectionDown` and `onConnectionUp` must be specified.**</span></span>

<span data-ttu-id="09c85-200">`Pages/_Host.cshtml`:</span><span class="sxs-lookup"><span data-stu-id="09c85-200">`Pages/_Host.cshtml`:</span></span>

```cshtml
<body>
    ...

    <script autostart="false" src="_framework/blazor.server.js"></script>
    <script>
      Blazor.start({
        reconnectionHandler: {
          onConnectionDown: (options, error) => console.error(error),
          onConnectionUp: () => console.log("Up, up, and away!")
        }
      });
    </script>
</body>
```

### <a name="adjust-the-reconnection-retry-count-and-interval"></a><span data-ttu-id="09c85-201">Modificare il numero di tentativi di riconnessione e l'intervallo</span><span class="sxs-lookup"><span data-stu-id="09c85-201">Adjust the reconnection retry count and interval</span></span>

<span data-ttu-id="09c85-202">Per modificare il numero di tentativi di riconnessione e l'intervallo, impostare il numero di tentativi ( `maxRetries` ) e il periodo di tempo in millisecondi consentiti per ogni nuovo tentativo ( `retryIntervalMilliseconds` ).</span><span class="sxs-lookup"><span data-stu-id="09c85-202">To adjust the reconnection retry count and interval, set the number of retries (`maxRetries`) and period in milliseconds permitted for each retry attempt (`retryIntervalMilliseconds`).</span></span>

<span data-ttu-id="09c85-203">`Pages/_Host.cshtml`:</span><span class="sxs-lookup"><span data-stu-id="09c85-203">`Pages/_Host.cshtml`:</span></span>

```cshtml
<body>
    ...

    <script autostart="false" src="_framework/blazor.server.js"></script>
    <script>
      Blazor.start({
        reconnectionOptions: {
          maxRetries: 3,
          retryIntervalMilliseconds: 2000
        }
      });
    </script>
</body>
```

## <a name="hide-or-replace-the-reconnection-display"></a><span data-ttu-id="09c85-204">Nascondi o Sostituisci la visualizzazione di riconnessione</span><span class="sxs-lookup"><span data-stu-id="09c85-204">Hide or replace the reconnection display</span></span>

<span data-ttu-id="09c85-205">Per nascondere la visualizzazione della riconnessione, impostare il gestore di riconnessione `_reconnectionDisplay` su un oggetto vuoto ( `{}` o `new Object()` ).</span><span class="sxs-lookup"><span data-stu-id="09c85-205">To hide the reconnection display, set the reconnection handler's `_reconnectionDisplay` to an empty object (`{}` or `new Object()`).</span></span>

<span data-ttu-id="09c85-206">`Pages/_Host.cshtml`:</span><span class="sxs-lookup"><span data-stu-id="09c85-206">`Pages/_Host.cshtml`:</span></span>

```cshtml
<body>
    ...

    <script autostart="false" src="_framework/blazor.server.js"></script>
    <script>
      window.addEventListener('beforeunload', function () {
        Blazor.defaultReconnectionHandler._reconnectionDisplay = {};
      });

      Blazor.start();
    </script>
</body>
```

<span data-ttu-id="09c85-207">Per sostituire la visualizzazione della riconnessione, impostare `_reconnectionDisplay` nell'esempio precedente sull'elemento per la visualizzazione:</span><span class="sxs-lookup"><span data-stu-id="09c85-207">To replace the reconnection display, set `_reconnectionDisplay` in the preceding example to the element for display:</span></span>

```javascript
Blazor.defaultReconnectionHandler._reconnectionDisplay = 
  document.getElementById("{ELEMENT ID}");
```

<span data-ttu-id="09c85-208">Il segnaposto `{ELEMENT ID}` è l'ID dell'elemento HTML da visualizzare.</span><span class="sxs-lookup"><span data-stu-id="09c85-208">The placeholder `{ELEMENT ID}` is the ID of the HTML element to display.</span></span>

::: moniker range=">= aspnetcore-5.0"

<span data-ttu-id="09c85-209">Personalizzare il ritardo prima che la visualizzazione della riconnessione venga visualizzata impostando la `transition-delay` proprietà nel CSS del sito per l'elemento modale.</span><span class="sxs-lookup"><span data-stu-id="09c85-209">Customize the delay before the reconnection display appears by setting the `transition-delay` property in the site's CSS for the modal element.</span></span> <span data-ttu-id="09c85-210">Nell'esempio seguente viene impostato il ritardo di transizione da 500 ms (impostazione predefinita) a 1.000 ms (1 secondo).</span><span class="sxs-lookup"><span data-stu-id="09c85-210">The following example sets the transition delay from 500 ms (default) to 1,000 ms (1 second).</span></span>

<span data-ttu-id="09c85-211">`wwwroot/css/site.css`:</span><span class="sxs-lookup"><span data-stu-id="09c85-211">`wwwroot/css/site.css`:</span></span>

```css
#components-reconnect-modal {
    transition: visibility 0s linear 1000ms;
}
```

## <a name="disconnect-the-blazor-circuit-from-the-client"></a><span data-ttu-id="09c85-212">Disconnettere il Blazor circuito dal client</span><span class="sxs-lookup"><span data-stu-id="09c85-212">Disconnect the Blazor circuit from the client</span></span>

<span data-ttu-id="09c85-213">Per impostazione predefinita, un Blazor circuito viene disconnesso quando viene attivato l' [ `unload` evento della pagina](https://developer.mozilla.org/docs/Web/API/Window/unload_event) .</span><span class="sxs-lookup"><span data-stu-id="09c85-213">By default, a Blazor circuit is disconnected when the [`unload` page event](https://developer.mozilla.org/docs/Web/API/Window/unload_event) is triggered.</span></span> <span data-ttu-id="09c85-214">Per disconnettere il circuito per altri scenari nel client, richiamare `Blazor.disconnect` nel gestore eventi appropriato.</span><span class="sxs-lookup"><span data-stu-id="09c85-214">To disconnect the circuit for other scenarios on the client, invoke `Blazor.disconnect` in the appropriate event handler.</span></span> <span data-ttu-id="09c85-215">Nell'esempio seguente il circuito viene disconnesso quando la pagina è nascosta ([ `pagehide` evento](https://developer.mozilla.org/docs/Web/API/Window/pagehide_event)):</span><span class="sxs-lookup"><span data-stu-id="09c85-215">In the following example, the circuit is disconnected when the page is hidden ([`pagehide` event](https://developer.mozilla.org/docs/Web/API/Window/pagehide_event)):</span></span>

```javascript
window.addEventListener('pagehide', () => {
  Blazor.disconnect();
});
```

::: moniker-end

## <a name="additional-resources"></a><span data-ttu-id="09c85-216">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="09c85-216">Additional resources</span></span>

* <xref:signalr/introduction>
* <xref:signalr/configuration>
* <xref:blazor/security/server/threat-mitigation>
* [<span data-ttu-id="09c85-217">Blazor Server eventi di riconnessione e eventi del ciclo di vita del componente</span><span class="sxs-lookup"><span data-stu-id="09c85-217">Blazor Server reconnection events and component lifecycle events</span></span>](xref:blazor/components/lifecycle#blazor-server-reconnection-events)

::: zone-end
