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
ms.openlocfilehash: 3198f45819020ca551617aa12a146f2b8a9a9f8e
ms.sourcegitcommit: 1166b0ff3828418559510c661e8240e5c5717bb7
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/12/2021
ms.locfileid: "100279863"
---
# <a name="aspnet-core-blazor-signalr-guidance"></a><span data-ttu-id="2803e-103">Blazor SignalR Guida ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="2803e-103">ASP.NET Core Blazor SignalR guidance</span></span>

<span data-ttu-id="2803e-104">Per indicazioni generali sulla SignalR configurazione di ASP.NET Core, vedere gli argomenti nella <xref:signalr/introduction> sezione della documentazione relativa a.</span><span class="sxs-lookup"><span data-stu-id="2803e-104">For general guidance on ASP.NET Core SignalR configuration, see the topics in the <xref:signalr/introduction> area of the documentation.</span></span> <span data-ttu-id="2803e-105">Per configurare SignalR [aggiunto a una Blazor WebAssembly soluzione ospitata](xref:tutorials/signalr-blazor), vedere <xref:signalr/configuration#configure-server-options> .</span><span class="sxs-lookup"><span data-stu-id="2803e-105">To configure SignalR [added to a hosted Blazor WebAssembly solution](xref:tutorials/signalr-blazor), see <xref:signalr/configuration#configure-server-options>.</span></span>

## <a name="circuit-handler-options"></a><span data-ttu-id="2803e-106">Opzioni del gestore del circuito</span><span class="sxs-lookup"><span data-stu-id="2803e-106">Circuit handler options</span></span>

<span data-ttu-id="2803e-107">*Questa sezione si applica a Blazor Server .*</span><span class="sxs-lookup"><span data-stu-id="2803e-107">*This section applies to Blazor Server.*</span></span>

<span data-ttu-id="2803e-108">Configurare il Blazor Server circuito con il <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions> illustrato nella tabella seguente.</span><span class="sxs-lookup"><span data-stu-id="2803e-108">Configure the Blazor Server circuit with the <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions> shown in the following table.</span></span>

| <span data-ttu-id="2803e-109">Opzione</span><span class="sxs-lookup"><span data-stu-id="2803e-109">Option</span></span> | <span data-ttu-id="2803e-110">Predefinito</span><span class="sxs-lookup"><span data-stu-id="2803e-110">Default</span></span> | <span data-ttu-id="2803e-111">Descrizione</span><span class="sxs-lookup"><span data-stu-id="2803e-111">Description</span></span> |
| --- | --- | --- |
| <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions.DetailedErrors> | `false` | <span data-ttu-id="2803e-112">Invia messaggi di eccezione dettagliati a JavaScript quando si verifica un'eccezione non gestita nel circuito o quando una chiamata a un metodo .NET tramite l'interoperabilità JS genera un'eccezione.</span><span class="sxs-lookup"><span data-stu-id="2803e-112">Send detailed exception messages to JavaScript when an unhandled exception happens on the circuit or when a .NET method invocation through JS interop results in an exception.</span></span> |
| <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions.DisconnectedCircuitMaxRetained> | <span data-ttu-id="2803e-113">100</span><span class="sxs-lookup"><span data-stu-id="2803e-113">100</span></span> | <span data-ttu-id="2803e-114">Numero massimo di circuiti disconnessi che un determinato server utilizza in memoria per volta.</span><span class="sxs-lookup"><span data-stu-id="2803e-114">Maximum number of disconnected circuits that a given server holds in memory at a time.</span></span> |
| <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions.DisconnectedCircuitRetentionPeriod> | <span data-ttu-id="2803e-115">3 minuti</span><span class="sxs-lookup"><span data-stu-id="2803e-115">3 minutes</span></span> | <span data-ttu-id="2803e-116">Quantità massima di tempo durante il quale un circuito disconnesso viene mantenuto in memoria prima di essere eliminato.</span><span class="sxs-lookup"><span data-stu-id="2803e-116">Maximum amount of time a disconnected circuit is held in memory before being torn down.</span></span> |
| <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions.JSInteropDefaultCallTimeout> | <span data-ttu-id="2803e-117">1 minuto</span><span class="sxs-lookup"><span data-stu-id="2803e-117">1 minute</span></span> | <span data-ttu-id="2803e-118">Quantità massima di tempo di attesa del server prima del timeout di una chiamata asincrona alla funzione JavaScript.</span><span class="sxs-lookup"><span data-stu-id="2803e-118">Maximum amount of time the server waits before timing out an asynchronous JavaScript function invocation.</span></span> |
| <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions.MaxBufferedUnacknowledgedRenderBatches> | <span data-ttu-id="2803e-119">10</span><span class="sxs-lookup"><span data-stu-id="2803e-119">10</span></span> | <span data-ttu-id="2803e-120">Numero massimo di batch di rendering non riconosciuti che il server mantiene in memoria per ogni circuito in un determinato momento per supportare una riconnessione affidabile.</span><span class="sxs-lookup"><span data-stu-id="2803e-120">Maximum number of unacknowledged render batches the server keeps in memory per circuit at a given time to support robust reconnection.</span></span> <span data-ttu-id="2803e-121">Dopo aver raggiunto il limite, il server smette di produrre nuovi batch di rendering finché uno o più batch non sono stati riconosciuti dal client.</span><span class="sxs-lookup"><span data-stu-id="2803e-121">After reaching the limit, the server stops producing new render batches until one or more batches have been acknowledged by the client.</span></span> |

<span data-ttu-id="2803e-122">Configurare le opzioni in `Startup.ConfigureServices` con un delegato di opzioni a <xref:Microsoft.Extensions.DependencyInjection.ComponentServiceCollectionExtensions.AddServerSideBlazor%2A> .</span><span class="sxs-lookup"><span data-stu-id="2803e-122">Configure the options in `Startup.ConfigureServices` with an options delegate to <xref:Microsoft.Extensions.DependencyInjection.ComponentServiceCollectionExtensions.AddServerSideBlazor%2A>.</span></span> <span data-ttu-id="2803e-123">Nell'esempio seguente vengono assegnati i valori di opzione predefiniti indicati nella tabella precedente:</span><span class="sxs-lookup"><span data-stu-id="2803e-123">The following example assigns the default option values shown in the preceding table:</span></span>

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

<span data-ttu-id="2803e-124">Per configurare <xref:Microsoft.AspNetCore.SignalR.HubConnectionContext> , usare <xref:Microsoft.AspNetCore.SignalR.HubConnectionContextOptions> con <xref:Microsoft.Extensions.DependencyInjection.ServerSideBlazorBuilderExtensions.AddHubOptions%2A> .</span><span class="sxs-lookup"><span data-stu-id="2803e-124">To configure the <xref:Microsoft.AspNetCore.SignalR.HubConnectionContext>, use <xref:Microsoft.AspNetCore.SignalR.HubConnectionContextOptions> with <xref:Microsoft.Extensions.DependencyInjection.ServerSideBlazorBuilderExtensions.AddHubOptions%2A>.</span></span> <span data-ttu-id="2803e-125">Per le descrizioni delle opzioni, vedere <xref:signalr/configuration#configure-server-options> .</span><span class="sxs-lookup"><span data-stu-id="2803e-125">For option descriptions, see <xref:signalr/configuration#configure-server-options>.</span></span> <span data-ttu-id="2803e-126">Nell'esempio seguente vengono assegnati i valori predefiniti delle opzioni:</span><span class="sxs-lookup"><span data-stu-id="2803e-126">The following example assigns the default option values:</span></span>

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

## <a name="signalr-cross-origin-negotiation-for-authentication"></a><span data-ttu-id="2803e-127">SignalR negoziazione tra le origini per l'autenticazione</span><span class="sxs-lookup"><span data-stu-id="2803e-127">SignalR cross-origin negotiation for authentication</span></span>

<span data-ttu-id="2803e-128">*Questa sezione si applica a Blazor WebAssembly .*</span><span class="sxs-lookup"><span data-stu-id="2803e-128">*This section applies to Blazor WebAssembly.*</span></span>

<span data-ttu-id="2803e-129">Per configurare SignalR il client sottostante per l'invio di credenziali, ad esempio le cookie intestazioni s o http Authentication:</span><span class="sxs-lookup"><span data-stu-id="2803e-129">To configure SignalR's underlying client to send credentials, such as cookies or HTTP authentication headers:</span></span>

* <span data-ttu-id="2803e-130">Usare <xref:Microsoft.AspNetCore.Components.WebAssembly.Http.WebAssemblyHttpRequestMessageExtensions.SetBrowserRequestCredentials%2A> per impostare <xref:Microsoft.AspNetCore.Components.WebAssembly.Http.BrowserRequestCredentials.Include> le richieste tra origini [`fetch`](https://developer.mozilla.org/docs/Web/API/Fetch_API/Using_Fetch) :</span><span class="sxs-lookup"><span data-stu-id="2803e-130">Use <xref:Microsoft.AspNetCore.Components.WebAssembly.Http.WebAssemblyHttpRequestMessageExtensions.SetBrowserRequestCredentials%2A> to set <xref:Microsoft.AspNetCore.Components.WebAssembly.Http.BrowserRequestCredentials.Include> on cross-origin [`fetch`](https://developer.mozilla.org/docs/Web/API/Fetch_API/Using_Fetch) requests:</span></span>

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

* <span data-ttu-id="2803e-131">Assegnare <xref:System.Net.Http.HttpMessageHandler> a l' <xref:Microsoft.AspNetCore.Http.Connections.Client.HttpConnectionOptions.HttpMessageHandlerFactory> opzione:</span><span class="sxs-lookup"><span data-stu-id="2803e-131">Assign the <xref:System.Net.Http.HttpMessageHandler> to the <xref:Microsoft.AspNetCore.Http.Connections.Client.HttpConnectionOptions.HttpMessageHandlerFactory> option:</span></span>

  ```csharp
  var connection = new HubConnectionBuilder()
      .WithUrl(new Uri("http://signalr.example.com"), options =>
      {
          options.HttpMessageHandlerFactory = innerHandler => 
              new IncludeRequestCredentialsMessageHandler { InnerHandler = innerHandler };
      }).Build();
  ```

<span data-ttu-id="2803e-132">Per altre informazioni, vedere <xref:signalr/configuration#configure-additional-options>.</span><span class="sxs-lookup"><span data-stu-id="2803e-132">For more information, see <xref:signalr/configuration#configure-additional-options>.</span></span>

## <a name="reflect-the-connection-state-in-the-ui"></a><span data-ttu-id="2803e-133">Riflette lo stato di connessione nell'interfaccia utente</span><span class="sxs-lookup"><span data-stu-id="2803e-133">Reflect the connection state in the UI</span></span>

<span data-ttu-id="2803e-134">*Questa sezione si applica a Blazor Server .*</span><span class="sxs-lookup"><span data-stu-id="2803e-134">*This section applies to Blazor Server.*</span></span>

<span data-ttu-id="2803e-135">Quando il client rileva che la connessione è stata persa, viene visualizzata un'interfaccia utente predefinita quando il client tenta di riconnettersi.</span><span class="sxs-lookup"><span data-stu-id="2803e-135">When the client detects that the connection has been lost, a default UI is displayed to the user while the client attempts to reconnect.</span></span> <span data-ttu-id="2803e-136">Se la riconnessione non riesce, all'utente viene offerta l'opzione per riprovare.</span><span class="sxs-lookup"><span data-stu-id="2803e-136">If reconnection fails, the user is provided the option to retry.</span></span>

<span data-ttu-id="2803e-137">Per personalizzare l'interfaccia utente, definire un elemento con un valore `id` di `components-reconnect-modal` nell'oggetto `<body>` della `_Host.cshtml` Razor pagina:</span><span class="sxs-lookup"><span data-stu-id="2803e-137">To customize the UI, define an element with an `id` of `components-reconnect-modal` in the `<body>` of the `_Host.cshtml` Razor page:</span></span>

```cshtml
<div id="components-reconnect-modal">
    ...
</div>
```

<span data-ttu-id="2803e-138">Aggiungere quanto segue al foglio di stile dell'app ( `wwwroot/css/app.css` o `wwwroot/css/site.css` ):</span><span class="sxs-lookup"><span data-stu-id="2803e-138">Add the following to the app's stylesheet (`wwwroot/css/app.css` or `wwwroot/css/site.css`):</span></span>

```css
#components-reconnect-modal {
    display: none;
}

#components-reconnect-modal.components-reconnect-show {
    display: block;
}
```

<span data-ttu-id="2803e-139">Nella tabella seguente vengono descritte le classi CSS applicate all' `components-reconnect-modal` elemento.</span><span class="sxs-lookup"><span data-stu-id="2803e-139">The following table describes the CSS classes applied to the `components-reconnect-modal` element.</span></span>

| <span data-ttu-id="2803e-140">Classe CSS</span><span class="sxs-lookup"><span data-stu-id="2803e-140">CSS class</span></span>                       | <span data-ttu-id="2803e-141">Indica&hellip;</span><span class="sxs-lookup"><span data-stu-id="2803e-141">Indicates&hellip;</span></span> |
| ------------------------------- | ----------------- |
| `components-reconnect-show`     | <span data-ttu-id="2803e-142">Connessione persa.</span><span class="sxs-lookup"><span data-stu-id="2803e-142">A lost connection.</span></span> <span data-ttu-id="2803e-143">È in corso un tentativo di riconnessione del client.</span><span class="sxs-lookup"><span data-stu-id="2803e-143">The client is attempting to reconnect.</span></span> <span data-ttu-id="2803e-144">Mostra il modale.</span><span class="sxs-lookup"><span data-stu-id="2803e-144">Show the modal.</span></span> |
| `components-reconnect-hide`     | <span data-ttu-id="2803e-145">Viene ristabilita una connessione attiva al server.</span><span class="sxs-lookup"><span data-stu-id="2803e-145">An active connection is re-established to the server.</span></span> <span data-ttu-id="2803e-146">Nascondere il modale.</span><span class="sxs-lookup"><span data-stu-id="2803e-146">Hide the modal.</span></span> |
| `components-reconnect-failed`   | <span data-ttu-id="2803e-147">Riconnessione non riuscita. probabilmente a causa di un errore di rete.</span><span class="sxs-lookup"><span data-stu-id="2803e-147">Reconnection failed, probably due to a network failure.</span></span> <span data-ttu-id="2803e-148">Per tentare la riconnessione, chiamare `window.Blazor.reconnect()` .</span><span class="sxs-lookup"><span data-stu-id="2803e-148">To attempt reconnection, call `window.Blazor.reconnect()`.</span></span> |
| `components-reconnect-rejected` | <span data-ttu-id="2803e-149">Riconnessione rifiutata.</span><span class="sxs-lookup"><span data-stu-id="2803e-149">Reconnection rejected.</span></span> <span data-ttu-id="2803e-150">Il server è stato raggiunto ma ha rifiutato la connessione e lo stato dell'utente nel server è andato perso.</span><span class="sxs-lookup"><span data-stu-id="2803e-150">The server was reached but refused the connection, and the user's state on the server is lost.</span></span> <span data-ttu-id="2803e-151">Per ricaricare l'app, chiamare `location.reload()` .</span><span class="sxs-lookup"><span data-stu-id="2803e-151">To reload the app, call `location.reload()`.</span></span> <span data-ttu-id="2803e-152">Questo stato di connessione può verificarsi nei casi seguenti:</span><span class="sxs-lookup"><span data-stu-id="2803e-152">This connection state may result when:</span></span><ul><li><span data-ttu-id="2803e-153">Si verifica un arresto anomalo del circuito sul lato server.</span><span class="sxs-lookup"><span data-stu-id="2803e-153">A crash in the server-side circuit occurs.</span></span></li><li><span data-ttu-id="2803e-154">Il client viene disconnesso abbastanza a lungo da consentire al server di eliminare lo stato dell'utente.</span><span class="sxs-lookup"><span data-stu-id="2803e-154">The client is disconnected long enough for the server to drop the user's state.</span></span> <span data-ttu-id="2803e-155">Le istanze dei componenti con cui l'utente interagisce sono state eliminate.</span><span class="sxs-lookup"><span data-stu-id="2803e-155">Instances of the components that the user is interacting with are disposed.</span></span></li><li><span data-ttu-id="2803e-156">Il server viene riavviato oppure il processo di lavoro dell'app viene riciclato.</span><span class="sxs-lookup"><span data-stu-id="2803e-156">The server is restarted, or the app's worker process is recycled.</span></span></li></ul> |

## <a name="render-mode"></a><span data-ttu-id="2803e-157">Modalità di rendering</span><span class="sxs-lookup"><span data-stu-id="2803e-157">Render mode</span></span>

::: moniker range=">= aspnetcore-5.0"

<span data-ttu-id="2803e-158">*Questa sezione si applica a Hosted Blazor WebAssembly e Blazor Server .*</span><span class="sxs-lookup"><span data-stu-id="2803e-158">*This section applies to hosted Blazor WebAssembly and Blazor Server.*</span></span>

<span data-ttu-id="2803e-159">Blazor per impostazione predefinita, le app sono configurate per eseguire il prerendering dell'interfaccia utente nel server.</span><span class="sxs-lookup"><span data-stu-id="2803e-159">Blazor apps are set up by default to prerender the UI on the server.</span></span> <span data-ttu-id="2803e-160">Per altre informazioni, vedere <xref:mvc/views/tag-helpers/builtin-th/component-tag-helper>.</span><span class="sxs-lookup"><span data-stu-id="2803e-160">For more information, see <xref:mvc/views/tag-helpers/builtin-th/component-tag-helper>.</span></span>

::: moniker-end

::: moniker range="< aspnetcore-5.0"

<span data-ttu-id="2803e-161">*Questa sezione si applica a Blazor Server .*</span><span class="sxs-lookup"><span data-stu-id="2803e-161">*This section applies to Blazor Server.*</span></span>

<span data-ttu-id="2803e-162">Blazor Server per impostazione predefinita, le app sono configurate per eseguire il prerendering dell'interfaccia utente nel server prima che venga stabilita la connessione client al server.</span><span class="sxs-lookup"><span data-stu-id="2803e-162">Blazor Server apps are set up by default to prerender the UI on the server before the client connection to the server is established.</span></span> <span data-ttu-id="2803e-163">Per altre informazioni, vedere <xref:mvc/views/tag-helpers/builtin-th/component-tag-helper>.</span><span class="sxs-lookup"><span data-stu-id="2803e-163">For more information, see <xref:mvc/views/tag-helpers/builtin-th/component-tag-helper>.</span></span>

::: moniker-end

## <a name="initialize-the-blazor-circuit"></a><span data-ttu-id="2803e-164">Inizializzare il Blazor circuito</span><span class="sxs-lookup"><span data-stu-id="2803e-164">Initialize the Blazor circuit</span></span>

<span data-ttu-id="2803e-165">*Questa sezione si applica a Blazor Server .*</span><span class="sxs-lookup"><span data-stu-id="2803e-165">*This section applies to Blazor Server.*</span></span>

<span data-ttu-id="2803e-166">Configurare l'avvio manuale del Blazor Server [ SignalR circuito](xref:blazor/hosting-models#circuits) di un'app nel `Pages/_Host.cshtml` file:</span><span class="sxs-lookup"><span data-stu-id="2803e-166">Configure the manual start of a Blazor Server app's [SignalR circuit](xref:blazor/hosting-models#circuits) in the `Pages/_Host.cshtml` file:</span></span>

* <span data-ttu-id="2803e-167">Aggiungere un `autostart="false"` attributo al `<script>` tag per lo `blazor.server.js` script.</span><span class="sxs-lookup"><span data-stu-id="2803e-167">Add an `autostart="false"` attribute to the `<script>` tag for the `blazor.server.js` script.</span></span>
* <span data-ttu-id="2803e-168">Inserire uno script che chiama `Blazor.start` dopo il `blazor.server.js` tag dello script e all'interno del `</body>` tag di chiusura.</span><span class="sxs-lookup"><span data-stu-id="2803e-168">Place a script that calls `Blazor.start` after the `blazor.server.js` script's tag and inside the closing `</body>` tag.</span></span>

<span data-ttu-id="2803e-169">Quando `autostart` è disabilitato, qualsiasi aspetto dell'app che non dipende dal circuito funziona normalmente.</span><span class="sxs-lookup"><span data-stu-id="2803e-169">When `autostart` is disabled, any aspect of the app that doesn't depend on the circuit works normally.</span></span> <span data-ttu-id="2803e-170">Il routing lato client, ad esempio, è operativo.</span><span class="sxs-lookup"><span data-stu-id="2803e-170">For example, client-side routing is operational.</span></span> <span data-ttu-id="2803e-171">Tuttavia, qualsiasi aspetto che dipende dal circuito non è operativo fino a quando non `Blazor.start` viene chiamato il metodo.</span><span class="sxs-lookup"><span data-stu-id="2803e-171">However, any aspect that depends on the circuit isn't operational until `Blazor.start` is called.</span></span> <span data-ttu-id="2803e-172">Il comportamento dell'app è imprevedibile senza un circuito stabilito.</span><span class="sxs-lookup"><span data-stu-id="2803e-172">App behavior is unpredictable without an established circuit.</span></span> <span data-ttu-id="2803e-173">Ad esempio, i metodi dei componenti non vengono eseguiti mentre il circuito è disconnesso.</span><span class="sxs-lookup"><span data-stu-id="2803e-173">For example, component methods fail to execute while the circuit is disconnected.</span></span>

### <a name="initialize-blazor-when-the-document-is-ready"></a><span data-ttu-id="2803e-174">Inizializza Blazor quando il documento è pronto</span><span class="sxs-lookup"><span data-stu-id="2803e-174">Initialize Blazor when the document is ready</span></span>

<span data-ttu-id="2803e-175">Per inizializzare l' Blazor app quando il documento è pronto:</span><span class="sxs-lookup"><span data-stu-id="2803e-175">To initialize the Blazor app when the document is ready:</span></span>

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

### <a name="chain-to-the-promise-that-results-from-a-manual-start"></a><span data-ttu-id="2803e-176">Concatenare a `Promise` che risulta da un avvio manuale</span><span class="sxs-lookup"><span data-stu-id="2803e-176">Chain to the `Promise` that results from a manual start</span></span>

<span data-ttu-id="2803e-177">Per eseguire attività aggiuntive, ad esempio l'inizializzazione dell'interoperabilità JS, usare `then` per concatenare a `Promise` che risulta da un'app manuale di Blazor avvio:</span><span class="sxs-lookup"><span data-stu-id="2803e-177">To perform additional tasks, such as JS interop initialization, use `then` to chain to the `Promise` that results from a manual Blazor app start:</span></span>

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

### <a name="configure-the-signalr-client"></a><span data-ttu-id="2803e-178">Configurare il SignalR client</span><span class="sxs-lookup"><span data-stu-id="2803e-178">Configure the SignalR client</span></span>

#### <a name="logging"></a><span data-ttu-id="2803e-179">Registrazione</span><span class="sxs-lookup"><span data-stu-id="2803e-179">Logging</span></span>

<span data-ttu-id="2803e-180">Per configurare la SignalR registrazione client, passare un oggetto di configurazione ( `configureSignalR` ) che chiama `configureLogging` con il livello di registrazione nel generatore client:</span><span class="sxs-lookup"><span data-stu-id="2803e-180">To configure SignalR client logging, pass in a configuration object (`configureSignalR`) that calls `configureLogging` with the log level on the client builder:</span></span>

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

<span data-ttu-id="2803e-181">Nell'esempio precedente, `information` è equivalente a un livello di log di <xref:Microsoft.Extensions.Logging.LogLevel.Information?displayProperty=nameWithType> .</span><span class="sxs-lookup"><span data-stu-id="2803e-181">In the preceding example, `information` is equivalent to a log level of <xref:Microsoft.Extensions.Logging.LogLevel.Information?displayProperty=nameWithType>.</span></span>

### <a name="modify-the-reconnection-handler"></a><span data-ttu-id="2803e-182">Modificare il gestore di riconnessione</span><span class="sxs-lookup"><span data-stu-id="2803e-182">Modify the reconnection handler</span></span>

<span data-ttu-id="2803e-183">Gli eventi di connessione del circuito del gestore di riconnessione possono essere modificati per comportamenti personalizzati, ad esempio:</span><span class="sxs-lookup"><span data-stu-id="2803e-183">The reconnection handler's circuit connection events can be modified for custom behaviors, such as:</span></span>

* <span data-ttu-id="2803e-184">Per inviare una notifica all'utente se la connessione viene eliminata.</span><span class="sxs-lookup"><span data-stu-id="2803e-184">To notify the user if the connection is dropped.</span></span>
* <span data-ttu-id="2803e-185">Per eseguire la registrazione (dal client) quando si connette un circuito.</span><span class="sxs-lookup"><span data-stu-id="2803e-185">To perform logging (from the client) when a circuit is connected.</span></span>

<span data-ttu-id="2803e-186">Per modificare gli eventi di connessione, registrare i callback per le seguenti modifiche della connessione:</span><span class="sxs-lookup"><span data-stu-id="2803e-186">To modify the connection events, register callbacks for the following connection changes:</span></span>

* <span data-ttu-id="2803e-187">Connessioni eliminate utilizzare `onConnectionDown` .</span><span class="sxs-lookup"><span data-stu-id="2803e-187">Dropped connections use `onConnectionDown`.</span></span>
* <span data-ttu-id="2803e-188">Le connessioni stabilite/ristabilite usano `onConnectionUp` .</span><span class="sxs-lookup"><span data-stu-id="2803e-188">Established/re-established connections use `onConnectionUp`.</span></span>

<span data-ttu-id="2803e-189">**Entrambi** `onConnectionDown` `onConnectionUp` è necessario specificare e:</span><span class="sxs-lookup"><span data-stu-id="2803e-189">**Both** `onConnectionDown` and `onConnectionUp` must be specified:</span></span>

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

### <a name="adjust-the-reconnection-retry-count-and-interval"></a><span data-ttu-id="2803e-190">Modificare il numero di tentativi di riconnessione e l'intervallo</span><span class="sxs-lookup"><span data-stu-id="2803e-190">Adjust the reconnection retry count and interval</span></span>

<span data-ttu-id="2803e-191">Per modificare il numero di tentativi di riconnessione e l'intervallo, impostare il numero di tentativi ( `maxRetries` ) e il periodo di tempo in millisecondi consentiti per ogni nuovo tentativo ( `retryIntervalMilliseconds` ):</span><span class="sxs-lookup"><span data-stu-id="2803e-191">To adjust the reconnection retry count and interval, set the number of retries (`maxRetries`) and period in milliseconds permitted for each retry attempt (`retryIntervalMilliseconds`):</span></span>

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

## <a name="hide-or-replace-the-reconnection-display"></a><span data-ttu-id="2803e-192">Nascondi o Sostituisci la visualizzazione di riconnessione</span><span class="sxs-lookup"><span data-stu-id="2803e-192">Hide or replace the reconnection display</span></span>

<span data-ttu-id="2803e-193">Per nascondere la visualizzazione della riconnessione, impostare il gestore di riconnessione `_reconnectionDisplay` su un oggetto vuoto ( `{}` o `new Object()` ):</span><span class="sxs-lookup"><span data-stu-id="2803e-193">To hide the reconnection display, set the reconnection handler's `_reconnectionDisplay` to an empty object (`{}` or `new Object()`):</span></span>

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

<span data-ttu-id="2803e-194">Per sostituire la visualizzazione della riconnessione, impostare `_reconnectionDisplay` nell'esempio precedente sull'elemento per la visualizzazione:</span><span class="sxs-lookup"><span data-stu-id="2803e-194">To replace the reconnection display, set `_reconnectionDisplay` in the preceding example to the element for display:</span></span>

```javascript
Blazor.defaultReconnectionHandler._reconnectionDisplay = 
  document.getElementById("{ELEMENT ID}");
```

<span data-ttu-id="2803e-195">Il segnaposto `{ELEMENT ID}` è l'ID dell'elemento HTML da visualizzare.</span><span class="sxs-lookup"><span data-stu-id="2803e-195">The placeholder `{ELEMENT ID}` is the ID of the HTML element to display.</span></span>

::: moniker range=">= aspnetcore-5.0"

<span data-ttu-id="2803e-196">Personalizzare il ritardo prima che la visualizzazione della riconnessione venga visualizzata impostando la `transition-delay` proprietà nel CSS ( `wwwroot/css/site.css` ) dell'app per l'elemento modale.</span><span class="sxs-lookup"><span data-stu-id="2803e-196">Customize the delay before the reconnection display appears by setting the `transition-delay` property in the app's CSS (`wwwroot/css/site.css`) for the modal element.</span></span> <span data-ttu-id="2803e-197">Nell'esempio seguente viene impostato il ritardo di transizione da 500 ms (impostazione predefinita) a 1.000 ms (1 secondo):</span><span class="sxs-lookup"><span data-stu-id="2803e-197">The following example sets the transition delay from 500 ms (default) to 1,000 ms (1 second):</span></span>

```css
#components-reconnect-modal {
    transition: visibility 0s linear 1000ms;
}
```

## <a name="disconnect-the-blazor-circuit-from-the-client"></a><span data-ttu-id="2803e-198">Disconnettere il Blazor circuito dal client</span><span class="sxs-lookup"><span data-stu-id="2803e-198">Disconnect the Blazor circuit from the client</span></span>

<span data-ttu-id="2803e-199">Per impostazione predefinita, un Blazor circuito viene disconnesso quando viene attivato l' [ `unload` evento della pagina](https://developer.mozilla.org/docs/Web/API/Window/unload_event) .</span><span class="sxs-lookup"><span data-stu-id="2803e-199">By default, a Blazor circuit is disconnected when the [`unload` page event](https://developer.mozilla.org/docs/Web/API/Window/unload_event) is triggered.</span></span> <span data-ttu-id="2803e-200">Per disconnettere il circuito per altri scenari nel client, richiamare `Blazor.disconnect` nel gestore eventi appropriato.</span><span class="sxs-lookup"><span data-stu-id="2803e-200">To disconnect the circuit for other scenarios on the client, invoke `Blazor.disconnect` in the appropriate event handler.</span></span> <span data-ttu-id="2803e-201">Nell'esempio seguente il circuito viene disconnesso quando la pagina è nascosta ([ `pagehide` evento](https://developer.mozilla.org/docs/Web/API/Window/pagehide_event)):</span><span class="sxs-lookup"><span data-stu-id="2803e-201">In the following example, the circuit is disconnected when the page is hidden ([`pagehide` event](https://developer.mozilla.org/docs/Web/API/Window/pagehide_event)):</span></span>

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

## <a name="additional-resources"></a><span data-ttu-id="2803e-202">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="2803e-202">Additional resources</span></span>

* <xref:signalr/introduction>
* <xref:signalr/configuration>
* <xref:blazor/security/server/threat-mitigation>
* [<span data-ttu-id="2803e-203">Blazor Server eventi di riconnessione e eventi del ciclo di vita del componente</span><span class="sxs-lookup"><span data-stu-id="2803e-203">Blazor Server reconnection events and component lifecycle events</span></span>](xref:blazor/components/lifecycle#blazor-server-reconnection-events)
