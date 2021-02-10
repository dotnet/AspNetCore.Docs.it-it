---
title: Blazor SignalR Guida ASP.NET Core
author: guardrex
description: Informazioni su come configurare e gestire le Blazor SignalR connessioni.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 01/27/2021
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
ms.openlocfilehash: 31cd68baa0fbc773f09fe3a1b37091b2dfc0b0a0
ms.sourcegitcommit: 04ad9cd26fcaa8bd11e261d3661f375f5f343cdc
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/10/2021
ms.locfileid: "100110036"
---
# <a name="aspnet-core-blazor-signalr-guidance"></a><span data-ttu-id="77439-103">Blazor SignalR Guida ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="77439-103">ASP.NET Core Blazor SignalR guidance</span></span>

<span data-ttu-id="77439-104">Di [Daniel Roth](https://github.com/danroth27), [Marin Buck](https://github.com/MackinnonBuck)e [Luke Latham](https://github.com/guardrex)</span><span class="sxs-lookup"><span data-stu-id="77439-104">By [Daniel Roth](https://github.com/danroth27), [Mackinnon Buck](https://github.com/MackinnonBuck), and [Luke Latham](https://github.com/guardrex)</span></span>

<span data-ttu-id="77439-105">Per indicazioni generali sulla SignalR configurazione di ASP.NET Core, vedere gli argomenti nella <xref:signalr/introduction> sezione della documentazione relativa a.</span><span class="sxs-lookup"><span data-stu-id="77439-105">For general guidance on ASP.NET Core SignalR configuration, see the topics in the <xref:signalr/introduction> area of the documentation.</span></span> <span data-ttu-id="77439-106">Per configurare SignalR [aggiunto a una Blazor WebAssembly soluzione ospitata](xref:tutorials/signalr-blazor), vedere <xref:signalr/configuration#configure-server-options> .</span><span class="sxs-lookup"><span data-stu-id="77439-106">To configure SignalR [added to a hosted Blazor WebAssembly solution](xref:tutorials/signalr-blazor), see <xref:signalr/configuration#configure-server-options>.</span></span>

## <a name="circuit-handler-options"></a><span data-ttu-id="77439-107">Opzioni del gestore del circuito</span><span class="sxs-lookup"><span data-stu-id="77439-107">Circuit handler options</span></span>

<span data-ttu-id="77439-108">*Questa sezione si applica a Blazor Server .*</span><span class="sxs-lookup"><span data-stu-id="77439-108">*This section applies to Blazor Server.*</span></span>

<span data-ttu-id="77439-109">Configurare il Blazor Server circuito con il <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions> illustrato nella tabella seguente.</span><span class="sxs-lookup"><span data-stu-id="77439-109">Configure the Blazor Server circuit with the <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions> shown in the following table.</span></span>

| <span data-ttu-id="77439-110">Opzione</span><span class="sxs-lookup"><span data-stu-id="77439-110">Option</span></span> | <span data-ttu-id="77439-111">Predefinito</span><span class="sxs-lookup"><span data-stu-id="77439-111">Default</span></span> | <span data-ttu-id="77439-112">Descrizione</span><span class="sxs-lookup"><span data-stu-id="77439-112">Description</span></span> |
| --- | --- | --- |
| <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions.DetailedErrors> | `false` | <span data-ttu-id="77439-113">Invia messaggi di eccezione dettagliati a JavaScript quando si verifica un'eccezione non gestita nel circuito o quando una chiamata a un metodo .NET tramite l'interoperabilità JS genera un'eccezione.</span><span class="sxs-lookup"><span data-stu-id="77439-113">Send detailed exception messages to JavaScript when an unhandled exception happens on the circuit or when a .NET method invocation through JS interop results in an exception.</span></span> |
| <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions.DisconnectedCircuitMaxRetained> | <span data-ttu-id="77439-114">100</span><span class="sxs-lookup"><span data-stu-id="77439-114">100</span></span> | <span data-ttu-id="77439-115">Numero massimo di circuiti disconnessi che un determinato server utilizza in memoria per volta.</span><span class="sxs-lookup"><span data-stu-id="77439-115">Maximum number of disconnected circuits that a given server holds in memory at a time.</span></span> |
| <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions.DisconnectedCircuitRetentionPeriod> | <span data-ttu-id="77439-116">3 minuti</span><span class="sxs-lookup"><span data-stu-id="77439-116">3 minutes</span></span> | <span data-ttu-id="77439-117">Quantità massima di tempo durante il quale un circuito disconnesso viene mantenuto in memoria prima di essere eliminato.</span><span class="sxs-lookup"><span data-stu-id="77439-117">Maximum amount of time a disconnected circuit is held in memory before being torn down.</span></span> |
| <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions.JSInteropDefaultCallTimeout> | <span data-ttu-id="77439-118">1 minuto</span><span class="sxs-lookup"><span data-stu-id="77439-118">1 minute</span></span> | <span data-ttu-id="77439-119">Quantità massima di tempo di attesa del server prima del timeout di una chiamata asincrona alla funzione JavaScript.</span><span class="sxs-lookup"><span data-stu-id="77439-119">Maximum amount of time the server waits before timing out an asynchronous JavaScript function invocation.</span></span> |
| <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions.MaxBufferedUnacknowledgedRenderBatches> | <span data-ttu-id="77439-120">10</span><span class="sxs-lookup"><span data-stu-id="77439-120">10</span></span> | <span data-ttu-id="77439-121">Numero massimo di batch di rendering non riconosciuti che il server mantiene in memoria per ogni circuito in un determinato momento per supportare una riconnessione affidabile.</span><span class="sxs-lookup"><span data-stu-id="77439-121">Maximum number of unacknowledged render batches the server keeps in memory per circuit at a given time to support robust reconnection.</span></span> <span data-ttu-id="77439-122">Dopo aver raggiunto il limite, il server smette di produrre nuovi batch di rendering finché uno o più batch non sono stati riconosciuti dal client.</span><span class="sxs-lookup"><span data-stu-id="77439-122">After reaching the limit, the server stops producing new render batches until one or more batches have been acknowledged by the client.</span></span> |

<span data-ttu-id="77439-123">Configurare le opzioni in `Startup.ConfigureServices` con un delegato di opzioni a <xref:Microsoft.Extensions.DependencyInjection.ComponentServiceCollectionExtensions.AddServerSideBlazor%2A> .</span><span class="sxs-lookup"><span data-stu-id="77439-123">Configure the options in `Startup.ConfigureServices` with an options delegate to <xref:Microsoft.Extensions.DependencyInjection.ComponentServiceCollectionExtensions.AddServerSideBlazor%2A>.</span></span> <span data-ttu-id="77439-124">Nell'esempio seguente vengono assegnati i valori di opzione predefiniti indicati nella tabella precedente:</span><span class="sxs-lookup"><span data-stu-id="77439-124">The following example assigns the default option values shown in the preceding table:</span></span>

```csharp
using System;

...

services.AddServerSideBlazor(options =>
{
    options.DetailedErrors = false;
    options.DisconnectedCircuitMaxRetained = 100;
    options.DisconnectedCircuitRetentionPeriod = TimeSpan.FromMinutes(3);
    options.JSInteropDefaultCallTimeout = TimeSpan.FromMinutes(1);
    options.MaxBufferedUnacknowledgedRenderBatches = 10;
});
```

<span data-ttu-id="77439-125">Per configurare <xref:Microsoft.AspNetCore.SignalR.HubConnectionContext> , usare <xref:Microsoft.AspNetCore.SignalR.HubConnectionContextOptions> con <xref:Microsoft.Extensions.DependencyInjection.ServerSideBlazorBuilderExtensions.AddHubOptions%2A> .</span><span class="sxs-lookup"><span data-stu-id="77439-125">To configure the <xref:Microsoft.AspNetCore.SignalR.HubConnectionContext>, use <xref:Microsoft.AspNetCore.SignalR.HubConnectionContextOptions> with <xref:Microsoft.Extensions.DependencyInjection.ServerSideBlazorBuilderExtensions.AddHubOptions%2A>.</span></span> <span data-ttu-id="77439-126">Per le descrizioni delle opzioni, vedere <xref:signalr/configuration#configure-server-options> .</span><span class="sxs-lookup"><span data-stu-id="77439-126">For option descriptions, see <xref:signalr/configuration#configure-server-options>.</span></span> <span data-ttu-id="77439-127">Nell'esempio seguente vengono assegnati i valori predefiniti delle opzioni:</span><span class="sxs-lookup"><span data-stu-id="77439-127">The following example assigns the default option values:</span></span>

```csharp
using System;

...

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

## <a name="signalr-cross-origin-negotiation-for-authentication"></a><span data-ttu-id="77439-128">SignalR negoziazione tra le origini per l'autenticazione</span><span class="sxs-lookup"><span data-stu-id="77439-128">SignalR cross-origin negotiation for authentication</span></span>

<span data-ttu-id="77439-129">*Questa sezione si applica a Blazor WebAssembly .*</span><span class="sxs-lookup"><span data-stu-id="77439-129">*This section applies to Blazor WebAssembly.*</span></span>

<span data-ttu-id="77439-130">Per configurare SignalR il client sottostante per l'invio di credenziali, ad esempio le cookie intestazioni s o http Authentication:</span><span class="sxs-lookup"><span data-stu-id="77439-130">To configure SignalR's underlying client to send credentials, such as cookies or HTTP authentication headers:</span></span>

* <span data-ttu-id="77439-131">Usare <xref:Microsoft.AspNetCore.Components.WebAssembly.Http.WebAssemblyHttpRequestMessageExtensions.SetBrowserRequestCredentials%2A> per impostare <xref:Microsoft.AspNetCore.Components.WebAssembly.Http.BrowserRequestCredentials.Include> le richieste tra origini [`fetch`](https://developer.mozilla.org/docs/Web/API/Fetch_API/Using_Fetch) :</span><span class="sxs-lookup"><span data-stu-id="77439-131">Use <xref:Microsoft.AspNetCore.Components.WebAssembly.Http.WebAssemblyHttpRequestMessageExtensions.SetBrowserRequestCredentials%2A> to set <xref:Microsoft.AspNetCore.Components.WebAssembly.Http.BrowserRequestCredentials.Include> on cross-origin [`fetch`](https://developer.mozilla.org/docs/Web/API/Fetch_API/Using_Fetch) requests:</span></span>

  ```csharp
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

* <span data-ttu-id="77439-132">Assegnare <xref:System.Net.Http.HttpMessageHandler> a l' <xref:Microsoft.AspNetCore.Http.Connections.Client.HttpConnectionOptions.HttpMessageHandlerFactory> opzione:</span><span class="sxs-lookup"><span data-stu-id="77439-132">Assign the <xref:System.Net.Http.HttpMessageHandler> to the <xref:Microsoft.AspNetCore.Http.Connections.Client.HttpConnectionOptions.HttpMessageHandlerFactory> option:</span></span>

  ```csharp
  var connection = new HubConnectionBuilder()
      .WithUrl(new Uri("http://signalr.example.com"), options =>
      {
          options.HttpMessageHandlerFactory = innerHandler => 
              new IncludeRequestCredentialsMessageHandler { InnerHandler = innerHandler };
      }).Build();
  ```

<span data-ttu-id="77439-133">Per altre informazioni, vedere <xref:signalr/configuration#configure-additional-options>.</span><span class="sxs-lookup"><span data-stu-id="77439-133">For more information, see <xref:signalr/configuration#configure-additional-options>.</span></span>

## <a name="reflect-the-connection-state-in-the-ui"></a><span data-ttu-id="77439-134">Riflette lo stato di connessione nell'interfaccia utente</span><span class="sxs-lookup"><span data-stu-id="77439-134">Reflect the connection state in the UI</span></span>

<span data-ttu-id="77439-135">*Questa sezione si applica a Blazor Server .*</span><span class="sxs-lookup"><span data-stu-id="77439-135">*This section applies to Blazor Server.*</span></span>

<span data-ttu-id="77439-136">Quando il client rileva che la connessione è stata persa, viene visualizzata un'interfaccia utente predefinita quando il client tenta di riconnettersi.</span><span class="sxs-lookup"><span data-stu-id="77439-136">When the client detects that the connection has been lost, a default UI is displayed to the user while the client attempts to reconnect.</span></span> <span data-ttu-id="77439-137">Se la riconnessione non riesce, all'utente viene offerta l'opzione per riprovare.</span><span class="sxs-lookup"><span data-stu-id="77439-137">If reconnection fails, the user is provided the option to retry.</span></span>

<span data-ttu-id="77439-138">Per personalizzare l'interfaccia utente, definire un elemento con un valore `id` di `components-reconnect-modal` nell'oggetto `<body>` della `_Host.cshtml` Razor pagina:</span><span class="sxs-lookup"><span data-stu-id="77439-138">To customize the UI, define an element with an `id` of `components-reconnect-modal` in the `<body>` of the `_Host.cshtml` Razor page:</span></span>

```cshtml
<div id="components-reconnect-modal">
    ...
</div>
```

<span data-ttu-id="77439-139">Aggiungere quanto segue al foglio di stile dell'app ( `wwwroot/css/app.css` o `wwwroot/css/site.css` ):</span><span class="sxs-lookup"><span data-stu-id="77439-139">Add the following to the app's stylesheet (`wwwroot/css/app.css` or `wwwroot/css/site.css`):</span></span>

```css
#components-reconnect-modal {
    display: none;
}

#components-reconnect-modal.components-reconnect-show {
    display: block;
}
```

<span data-ttu-id="77439-140">Nella tabella seguente vengono descritte le classi CSS applicate all' `components-reconnect-modal` elemento.</span><span class="sxs-lookup"><span data-stu-id="77439-140">The following table describes the CSS classes applied to the `components-reconnect-modal` element.</span></span>

| <span data-ttu-id="77439-141">Classe CSS</span><span class="sxs-lookup"><span data-stu-id="77439-141">CSS class</span></span>                       | <span data-ttu-id="77439-142">Indica&hellip;</span><span class="sxs-lookup"><span data-stu-id="77439-142">Indicates&hellip;</span></span> |
| ------------------------------- | ----------------- |
| `components-reconnect-show`     | <span data-ttu-id="77439-143">Connessione persa.</span><span class="sxs-lookup"><span data-stu-id="77439-143">A lost connection.</span></span> <span data-ttu-id="77439-144">È in corso un tentativo di riconnessione del client.</span><span class="sxs-lookup"><span data-stu-id="77439-144">The client is attempting to reconnect.</span></span> <span data-ttu-id="77439-145">Mostra il modale.</span><span class="sxs-lookup"><span data-stu-id="77439-145">Show the modal.</span></span> |
| `components-reconnect-hide`     | <span data-ttu-id="77439-146">Viene ristabilita una connessione attiva al server.</span><span class="sxs-lookup"><span data-stu-id="77439-146">An active connection is re-established to the server.</span></span> <span data-ttu-id="77439-147">Nascondere il modale.</span><span class="sxs-lookup"><span data-stu-id="77439-147">Hide the modal.</span></span> |
| `components-reconnect-failed`   | <span data-ttu-id="77439-148">Riconnessione non riuscita. probabilmente a causa di un errore di rete.</span><span class="sxs-lookup"><span data-stu-id="77439-148">Reconnection failed, probably due to a network failure.</span></span> <span data-ttu-id="77439-149">Per tentare la riconnessione, chiamare `window.Blazor.reconnect()` .</span><span class="sxs-lookup"><span data-stu-id="77439-149">To attempt reconnection, call `window.Blazor.reconnect()`.</span></span> |
| `components-reconnect-rejected` | <span data-ttu-id="77439-150">Riconnessione rifiutata.</span><span class="sxs-lookup"><span data-stu-id="77439-150">Reconnection rejected.</span></span> <span data-ttu-id="77439-151">Il server è stato raggiunto ma ha rifiutato la connessione e lo stato dell'utente nel server è andato perso.</span><span class="sxs-lookup"><span data-stu-id="77439-151">The server was reached but refused the connection, and the user's state on the server is lost.</span></span> <span data-ttu-id="77439-152">Per ricaricare l'app, chiamare `location.reload()` .</span><span class="sxs-lookup"><span data-stu-id="77439-152">To reload the app, call `location.reload()`.</span></span> <span data-ttu-id="77439-153">Questo stato di connessione può verificarsi nei casi seguenti:</span><span class="sxs-lookup"><span data-stu-id="77439-153">This connection state may result when:</span></span><ul><li><span data-ttu-id="77439-154">Si verifica un arresto anomalo del circuito sul lato server.</span><span class="sxs-lookup"><span data-stu-id="77439-154">A crash in the server-side circuit occurs.</span></span></li><li><span data-ttu-id="77439-155">Il client viene disconnesso abbastanza a lungo da consentire al server di eliminare lo stato dell'utente.</span><span class="sxs-lookup"><span data-stu-id="77439-155">The client is disconnected long enough for the server to drop the user's state.</span></span> <span data-ttu-id="77439-156">Le istanze dei componenti con cui l'utente interagisce sono state eliminate.</span><span class="sxs-lookup"><span data-stu-id="77439-156">Instances of the components that the user is interacting with are disposed.</span></span></li><li><span data-ttu-id="77439-157">Il server viene riavviato oppure il processo di lavoro dell'app viene riciclato.</span><span class="sxs-lookup"><span data-stu-id="77439-157">The server is restarted, or the app's worker process is recycled.</span></span></li></ul> |

## <a name="render-mode"></a><span data-ttu-id="77439-158">Modalità di rendering</span><span class="sxs-lookup"><span data-stu-id="77439-158">Render mode</span></span>

::: moniker range=">= aspnetcore-5.0"

<span data-ttu-id="77439-159">*Questa sezione si applica a Hosted Blazor WebAssembly e Blazor Server .*</span><span class="sxs-lookup"><span data-stu-id="77439-159">*This section applies to hosted Blazor WebAssembly and Blazor Server.*</span></span>

<span data-ttu-id="77439-160">Blazor per impostazione predefinita, le app sono configurate per eseguire il prerendering dell'interfaccia utente nel server.</span><span class="sxs-lookup"><span data-stu-id="77439-160">Blazor apps are set up by default to prerender the UI on the server.</span></span> <span data-ttu-id="77439-161">Per altre informazioni, vedere <xref:mvc/views/tag-helpers/builtin-th/component-tag-helper>.</span><span class="sxs-lookup"><span data-stu-id="77439-161">For more information, see <xref:mvc/views/tag-helpers/builtin-th/component-tag-helper>.</span></span>

::: moniker-end

::: moniker range="< aspnetcore-5.0"

<span data-ttu-id="77439-162">*Questa sezione si applica a Blazor Server .*</span><span class="sxs-lookup"><span data-stu-id="77439-162">*This section applies to Blazor Server.*</span></span>

<span data-ttu-id="77439-163">Blazor Server per impostazione predefinita, le app sono configurate per eseguire il prerendering dell'interfaccia utente nel server prima che venga stabilita la connessione client al server.</span><span class="sxs-lookup"><span data-stu-id="77439-163">Blazor Server apps are set up by default to prerender the UI on the server before the client connection to the server is established.</span></span> <span data-ttu-id="77439-164">Per altre informazioni, vedere <xref:mvc/views/tag-helpers/builtin-th/component-tag-helper>.</span><span class="sxs-lookup"><span data-stu-id="77439-164">For more information, see <xref:mvc/views/tag-helpers/builtin-th/component-tag-helper>.</span></span>

::: moniker-end

## <a name="initialize-the-blazor-circuit"></a><span data-ttu-id="77439-165">Inizializzare il Blazor circuito</span><span class="sxs-lookup"><span data-stu-id="77439-165">Initialize the Blazor circuit</span></span>

<span data-ttu-id="77439-166">*Questa sezione si applica a Blazor Server .*</span><span class="sxs-lookup"><span data-stu-id="77439-166">*This section applies to Blazor Server.*</span></span>

<span data-ttu-id="77439-167">Configurare l'avvio manuale del Blazor Server [ SignalR circuito](xref:blazor/hosting-models#circuits) di un'app nel `Pages/_Host.cshtml` file:</span><span class="sxs-lookup"><span data-stu-id="77439-167">Configure the manual start of a Blazor Server app's [SignalR circuit](xref:blazor/hosting-models#circuits) in the `Pages/_Host.cshtml` file:</span></span>

* <span data-ttu-id="77439-168">Aggiungere un `autostart="false"` attributo al `<script>` tag per lo `blazor.server.js` script.</span><span class="sxs-lookup"><span data-stu-id="77439-168">Add an `autostart="false"` attribute to the `<script>` tag for the `blazor.server.js` script.</span></span>
* <span data-ttu-id="77439-169">Inserire uno script che chiama `Blazor.start` dopo il `blazor.server.js` tag dello script e all'interno del `</body>` tag di chiusura.</span><span class="sxs-lookup"><span data-stu-id="77439-169">Place a script that calls `Blazor.start` after the `blazor.server.js` script's tag and inside the closing `</body>` tag.</span></span>

<span data-ttu-id="77439-170">Quando `autostart` è disabilitato, qualsiasi aspetto dell'app che non dipende dal circuito funziona normalmente.</span><span class="sxs-lookup"><span data-stu-id="77439-170">When `autostart` is disabled, any aspect of the app that doesn't depend on the circuit works normally.</span></span> <span data-ttu-id="77439-171">Il routing lato client, ad esempio, è operativo.</span><span class="sxs-lookup"><span data-stu-id="77439-171">For example, client-side routing is operational.</span></span> <span data-ttu-id="77439-172">Tuttavia, qualsiasi aspetto che dipende dal circuito non è operativo fino a quando non `Blazor.start` viene chiamato il metodo.</span><span class="sxs-lookup"><span data-stu-id="77439-172">However, any aspect that depends on the circuit isn't operational until `Blazor.start` is called.</span></span> <span data-ttu-id="77439-173">Il comportamento dell'app è imprevedibile senza un circuito stabilito.</span><span class="sxs-lookup"><span data-stu-id="77439-173">App behavior is unpredictable without an established circuit.</span></span> <span data-ttu-id="77439-174">Ad esempio, i metodi dei componenti non vengono eseguiti mentre il circuito è disconnesso.</span><span class="sxs-lookup"><span data-stu-id="77439-174">For example, component methods fail to execute while the circuit is disconnected.</span></span>

### <a name="initialize-blazor-when-the-document-is-ready"></a><span data-ttu-id="77439-175">Inizializza Blazor quando il documento è pronto</span><span class="sxs-lookup"><span data-stu-id="77439-175">Initialize Blazor when the document is ready</span></span>

<span data-ttu-id="77439-176">Per inizializzare l' Blazor app quando il documento è pronto:</span><span class="sxs-lookup"><span data-stu-id="77439-176">To initialize the Blazor app when the document is ready:</span></span>

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

### <a name="chain-to-the-promise-that-results-from-a-manual-start"></a><span data-ttu-id="77439-177">Concatenare a `Promise` che risulta da un avvio manuale</span><span class="sxs-lookup"><span data-stu-id="77439-177">Chain to the `Promise` that results from a manual start</span></span>

<span data-ttu-id="77439-178">Per eseguire attività aggiuntive, ad esempio l'inizializzazione dell'interoperabilità JS, usare `then` per concatenare a `Promise` che risulta da un'app manuale di Blazor avvio:</span><span class="sxs-lookup"><span data-stu-id="77439-178">To perform additional tasks, such as JS interop initialization, use `then` to chain to the `Promise` that results from a manual Blazor app start:</span></span>

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

### <a name="configure-the-signalr-client"></a><span data-ttu-id="77439-179">Configurare il SignalR client</span><span class="sxs-lookup"><span data-stu-id="77439-179">Configure the SignalR client</span></span>

#### <a name="logging"></a><span data-ttu-id="77439-180">Registrazione</span><span class="sxs-lookup"><span data-stu-id="77439-180">Logging</span></span>

<span data-ttu-id="77439-181">Per configurare la SignalR registrazione client, passare un oggetto di configurazione ( `configureSignalR` ) che chiama `configureLogging` con il livello di registrazione nel generatore client:</span><span class="sxs-lookup"><span data-stu-id="77439-181">To configure SignalR client logging, pass in a configuration object (`configureSignalR`) that calls `configureLogging` with the log level on the client builder:</span></span>

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

<span data-ttu-id="77439-182">Nell'esempio precedente, `information` è equivalente a un livello di log di <xref:Microsoft.Extensions.Logging.LogLevel.Information?displayProperty=nameWithType> .</span><span class="sxs-lookup"><span data-stu-id="77439-182">In the preceding example, `information` is equivalent to a log level of <xref:Microsoft.Extensions.Logging.LogLevel.Information?displayProperty=nameWithType>.</span></span>

### <a name="modify-the-reconnection-handler"></a><span data-ttu-id="77439-183">Modificare il gestore di riconnessione</span><span class="sxs-lookup"><span data-stu-id="77439-183">Modify the reconnection handler</span></span>

<span data-ttu-id="77439-184">Gli eventi di connessione del circuito del gestore di riconnessione possono essere modificati per comportamenti personalizzati, ad esempio:</span><span class="sxs-lookup"><span data-stu-id="77439-184">The reconnection handler's circuit connection events can be modified for custom behaviors, such as:</span></span>

* <span data-ttu-id="77439-185">Per inviare una notifica all'utente se la connessione viene eliminata.</span><span class="sxs-lookup"><span data-stu-id="77439-185">To notify the user if the connection is dropped.</span></span>
* <span data-ttu-id="77439-186">Per eseguire la registrazione (dal client) quando si connette un circuito.</span><span class="sxs-lookup"><span data-stu-id="77439-186">To perform logging (from the client) when a circuit is connected.</span></span>

<span data-ttu-id="77439-187">Per modificare gli eventi di connessione, registrare i callback per le seguenti modifiche della connessione:</span><span class="sxs-lookup"><span data-stu-id="77439-187">To modify the connection events, register callbacks for the following connection changes:</span></span>

* <span data-ttu-id="77439-188">Connessioni eliminate utilizzare `onConnectionDown` .</span><span class="sxs-lookup"><span data-stu-id="77439-188">Dropped connections use `onConnectionDown`.</span></span>
* <span data-ttu-id="77439-189">Le connessioni stabilite/ristabilite usano `onConnectionUp` .</span><span class="sxs-lookup"><span data-stu-id="77439-189">Established/re-established connections use `onConnectionUp`.</span></span>

<span data-ttu-id="77439-190">**Entrambi** `onConnectionDown` `onConnectionUp` è necessario specificare e:</span><span class="sxs-lookup"><span data-stu-id="77439-190">**Both** `onConnectionDown` and `onConnectionUp` must be specified:</span></span>

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

### <a name="adjust-the-reconnection-retry-count-and-interval"></a><span data-ttu-id="77439-191">Modificare il numero di tentativi di riconnessione e l'intervallo</span><span class="sxs-lookup"><span data-stu-id="77439-191">Adjust the reconnection retry count and interval</span></span>

<span data-ttu-id="77439-192">Per modificare il numero di tentativi di riconnessione e l'intervallo, impostare il numero di tentativi ( `maxRetries` ) e il periodo di tempo in millisecondi consentiti per ogni nuovo tentativo ( `retryIntervalMilliseconds` ):</span><span class="sxs-lookup"><span data-stu-id="77439-192">To adjust the reconnection retry count and interval, set the number of retries (`maxRetries`) and period in milliseconds permitted for each retry attempt (`retryIntervalMilliseconds`):</span></span>

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

## <a name="hide-or-replace-the-reconnection-display"></a><span data-ttu-id="77439-193">Nascondi o Sostituisci la visualizzazione di riconnessione</span><span class="sxs-lookup"><span data-stu-id="77439-193">Hide or replace the reconnection display</span></span>

<span data-ttu-id="77439-194">Per nascondere la visualizzazione della riconnessione, impostare il gestore di riconnessione `_reconnectionDisplay` su un oggetto vuoto ( `{}` o `new Object()` ):</span><span class="sxs-lookup"><span data-stu-id="77439-194">To hide the reconnection display, set the reconnection handler's `_reconnectionDisplay` to an empty object (`{}` or `new Object()`):</span></span>

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

<span data-ttu-id="77439-195">Per sostituire la visualizzazione della riconnessione, impostare `_reconnectionDisplay` nell'esempio precedente sull'elemento per la visualizzazione:</span><span class="sxs-lookup"><span data-stu-id="77439-195">To replace the reconnection display, set `_reconnectionDisplay` in the preceding example to the element for display:</span></span>

```javascript
Blazor.defaultReconnectionHandler._reconnectionDisplay = 
  document.getElementById("{ELEMENT ID}");
```

<span data-ttu-id="77439-196">Il segnaposto `{ELEMENT ID}` è l'ID dell'elemento HTML da visualizzare.</span><span class="sxs-lookup"><span data-stu-id="77439-196">The placeholder `{ELEMENT ID}` is the ID of the HTML element to display.</span></span>

::: moniker range=">= aspnetcore-5.0"

<span data-ttu-id="77439-197">Personalizzare il ritardo prima che la visualizzazione della riconnessione venga visualizzata impostando la `transition-delay` proprietà nel CSS ( `wwwroot/css/site.css` ) dell'app per l'elemento modale.</span><span class="sxs-lookup"><span data-stu-id="77439-197">Customize the delay before the reconnection display appears by setting the `transition-delay` property in the app's CSS (`wwwroot/css/site.css`) for the modal element.</span></span> <span data-ttu-id="77439-198">Nell'esempio seguente viene impostato il ritardo di transizione da 500 ms (impostazione predefinita) a 1.000 ms (1 secondo):</span><span class="sxs-lookup"><span data-stu-id="77439-198">The following example sets the transition delay from 500 ms (default) to 1,000 ms (1 second):</span></span>

```css
#components-reconnect-modal {
    transition: visibility 0s linear 1000ms;
}
```

## <a name="disconnect-the-blazor-circuit-from-the-client"></a><span data-ttu-id="77439-199">Disconnettere il Blazor circuito dal client</span><span class="sxs-lookup"><span data-stu-id="77439-199">Disconnect the Blazor circuit from the client</span></span>

<span data-ttu-id="77439-200">Per impostazione predefinita, un Blazor circuito viene disconnesso quando viene attivato l' [ `unload` evento della pagina](https://developer.mozilla.org/docs/Web/API/Window/unload_event) .</span><span class="sxs-lookup"><span data-stu-id="77439-200">By default, a Blazor circuit is disconnected when the [`unload` page event](https://developer.mozilla.org/docs/Web/API/Window/unload_event) is triggered.</span></span> <span data-ttu-id="77439-201">Per disconnettere il circuito per altri scenari nel client, richiamare `Blazor.disconnect` nel gestore eventi appropriato.</span><span class="sxs-lookup"><span data-stu-id="77439-201">To disconnect the circuit for other scenarios on the client, invoke `Blazor.disconnect` in the appropriate event handler.</span></span> <span data-ttu-id="77439-202">Nell'esempio seguente il circuito viene disconnesso quando la pagina è nascosta ([ `pagehide` evento](https://developer.mozilla.org/docs/Web/API/Window/pagehide_event)):</span><span class="sxs-lookup"><span data-stu-id="77439-202">In the following example, the circuit is disconnected when the page is hidden ([`pagehide` event](https://developer.mozilla.org/docs/Web/API/Window/pagehide_event)):</span></span>

```javascript
window.addEventListener('pagehide', () => {
  Blazor.disconnect();
});
```

<!-- HOLD for reactivation at 5x

THIS WILL BE MOVED TO ANOTHER TOPIC WHEN RE-ACTIVATED.

## Influence HTML `<head>` tag elements

*This section applies to the upcoming ASP.NET Core 5.0 release of Blazor WebAssembly and Blazor Server.*

When rendered, the `Title`, `Link`, and `Meta` components add or update data in the HTML `<head>` tag elements:

```razor
@using Microsoft.AspNetCore.Components.Web.Extensions.Head

<Title Value="{TITLE}" />
<Link href="{URL}" rel="stylesheet" />
<Meta content="{DESCRIPTION}" name="description" />
```

In the preceding example, placeholders for `{TITLE}`, `{URL}`, and `{DESCRIPTION}` are string values, Razor variables, or Razor expressions.

The following characteristics apply:

* Server-side prerendering is supported.
* The `Value` parameter is the only valid parameter for the `Title` component.
* HTML attributes provided to the `Meta` and `Link` components are captured in [additional attributes](xref:blazor/components/index#attribute-splatting-and-arbitrary-parameters) and passed through to the rendered HTML tag.
* For multiple `Title` components, the title of the page reflects the `Value` of the last `Title` component rendered.
* If multiple `Meta` or `Link` components are included with identical attributes, there's exactly one HTML tag rendered per `Meta` or `Link` component. Two `Meta` or `Link` components can't refer to the same rendered HTML tag.
* Changes to the parameters of existing `Meta` or `Link` components are reflected in their rendered HTML tags.
* When the `Link` or `Meta` components are no longer rendered and thus disposed by the framework, their rendered HTML tags are removed.

When one of the framework components is used in a child component, the rendered HTML tag influences any other child component of the parent component as long as the child component containing the framework component is rendered. The distinction between using the one of these framework components in a child component and placing a an HTML tag in `wwwroot/index.html` or `Pages/_Host.cshtml` is that a framework component's rendered HTML tag:

* Can be modified by application state. A hard-coded HTML tag can't be modified by application state.
* Is removed from the HTML `<head>` when the parent component is no longer rendered.

-->

::: moniker-end

## <a name="additional-resources"></a><span data-ttu-id="77439-203">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="77439-203">Additional resources</span></span>

* <xref:signalr/introduction>
* <xref:signalr/configuration>
* <xref:blazor/security/server/threat-mitigation>
* [<span data-ttu-id="77439-204">Blazor Server eventi di riconnessione e eventi del ciclo di vita del componente</span><span class="sxs-lookup"><span data-stu-id="77439-204">Blazor Server reconnection events and component lifecycle events</span></span>](xref:blazor/components/lifecycle#blazor-server-reconnection-events)
