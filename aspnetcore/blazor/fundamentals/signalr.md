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
# <a name="aspnet-core-blazor-signalr-guidance"></a>Blazor SignalR Guida ASP.NET Core

::: zone pivot="webassembly"

Questo articolo illustra come configurare e gestire le SignalR connessioni nelle Blazor app.

Per indicazioni generali sulla SignalR configurazione di ASP.NET Core, vedere gli argomenti nella <xref:signalr/introduction> sezione della documentazione relativa a. Per configurare SignalR [aggiunto a una Blazor WebAssembly soluzione ospitata](xref:tutorials/signalr-blazor), vedere <xref:signalr/configuration#configure-server-options> .

## <a name="signalr-cross-origin-negotiation-for-authentication"></a>SignalR negoziazione tra le origini per l'autenticazione

Per configurare SignalR il client sottostante per l'invio di credenziali, ad esempio le cookie intestazioni s o http Authentication:

* Usare <xref:Microsoft.AspNetCore.Components.WebAssembly.Http.WebAssemblyHttpRequestMessageExtensions.SetBrowserRequestCredentials%2A> per impostare <xref:Microsoft.AspNetCore.Components.WebAssembly.Http.BrowserRequestCredentials.Include> [`fetch`](https://developer.mozilla.org/docs/Web/API/Fetch_API/Using_Fetch) le richieste tra le origini.

  `IncludeRequestCredentialsMessageHandler.cs`:

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

* Quando viene compilata una connessione all'hub, assegnare <xref:System.Net.Http.HttpMessageHandler> a l' <xref:Microsoft.AspNetCore.Http.Connections.Client.HttpConnectionOptions.HttpMessageHandlerFactory> opzione:

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

  Nell'esempio precedente viene configurato l'URL di connessione dell'hub all'indirizzo URI assoluto in `/chathub` , ovvero l'URL usato nell' [ SignalR Blazor esercitazione with](xref:tutorials/signalr-blazor) nel `Index` componente ( `Pages/Index.razor` ). L'URI può essere impostato anche tramite una stringa, ad esempio `https://signalr.example.com` o tramite la [configurazione](xref:blazor/fundamentals/configuration).

Per altre informazioni, vedere <xref:signalr/configuration#configure-additional-options>.

::: moniker range=">= aspnetcore-5.0"

## <a name="render-mode"></a>Modalità di rendering

Se un' Blazor WebAssembly app che usa SignalR viene configurata per eseguire il prerendering sul server, il prerendering si verifica prima che venga stabilita la connessione client al server. Per altre informazioni, vedere gli articoli seguenti:

* <xref:mvc/views/tag-helpers/builtin-th/component-tag-helper>
* <xref:blazor/components/prerendering-and-integration>

::: moniker-end

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:signalr/introduction>
* <xref:signalr/configuration>

::: zone-end

::: zone pivot="server"

Questo articolo illustra come configurare e gestire le SignalR connessioni nelle Blazor app.

Per indicazioni generali sulla SignalR configurazione di ASP.NET Core, vedere gli argomenti nella <xref:signalr/introduction> sezione della documentazione relativa a. Per configurare SignalR [aggiunto a una Blazor WebAssembly soluzione ospitata](xref:tutorials/signalr-blazor), vedere <xref:signalr/configuration#configure-server-options> .

## <a name="circuit-handler-options"></a>Opzioni del gestore del circuito

Configurare il Blazor Server circuito con il <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions> illustrato nella tabella seguente.

| Opzione | Predefinito | Descrizione |
| --- | --- | --- |
| <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions.DetailedErrors> | `false` | Invia messaggi di eccezione dettagliati a JavaScript quando si verifica un'eccezione non gestita nel circuito o quando una chiamata a un metodo .NET tramite l'interoperabilità JS genera un'eccezione. |
| <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions.DisconnectedCircuitMaxRetained> | 100 | Numero massimo di circuiti disconnessi che il server utilizza in memoria per volta. |
| <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions.DisconnectedCircuitRetentionPeriod> | 3 minuti | Quantità massima di tempo durante il quale un circuito disconnesso viene mantenuto in memoria prima di essere eliminato. |
| <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions.JSInteropDefaultCallTimeout> | 1 minuto | Quantità massima di tempo di attesa del server prima del timeout di una chiamata asincrona alla funzione JavaScript. |
| <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions.MaxBufferedUnacknowledgedRenderBatches> | 10 | Numero massimo di batch di rendering non riconosciuti che il server mantiene in memoria per ogni circuito in un determinato momento per supportare una riconnessione affidabile. Dopo aver raggiunto il limite, il server smette di produrre nuovi batch di rendering finché uno o più batch non vengono riconosciuti dal client. |

Configurare le opzioni in `Startup.ConfigureServices` con un delegato di opzioni a <xref:Microsoft.Extensions.DependencyInjection.ComponentServiceCollectionExtensions.AddServerSideBlazor%2A> . Nell'esempio seguente vengono assegnati i valori predefiniti delle opzioni indicati nella tabella precedente. Verificare che `Startup.cs` utilizzi lo <xref:System> spazio dei nomi ( `using System;` ).

`Startup.ConfigureServices`:

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

Per configurare <xref:Microsoft.AspNetCore.SignalR.HubConnectionContext> , usare <xref:Microsoft.AspNetCore.SignalR.HubConnectionContextOptions> con <xref:Microsoft.Extensions.DependencyInjection.ServerSideBlazorBuilderExtensions.AddHubOptions%2A> . Per le descrizioni delle opzioni, vedere <xref:signalr/configuration#configure-server-options> . Nell'esempio seguente vengono assegnati i valori delle opzioni predefinite. Verificare che `Startup.cs` utilizzi lo <xref:System> spazio dei nomi ( `using System;` ).

`Startup.ConfigureServices`:

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

## <a name="reflect-the-connection-state-in-the-ui"></a>Riflette lo stato di connessione nell'interfaccia utente

Quando il client rileva che la connessione è stata persa, viene visualizzata un'interfaccia utente predefinita quando il client tenta di riconnettersi. Se la riconnessione non riesce, all'utente viene offerta l'opzione per riprovare.

Per personalizzare l'interfaccia utente, definire un elemento con un valore `id` di `components-reconnect-modal` nell'oggetto `<body>` della `_Host.cshtml` Razor pagina.

`Pages/_Host.cshtml`:

```cshtml
<div id="components-reconnect-modal">
    ...
</div>
```

Aggiungere gli stili CSS seguenti al foglio di stile del sito.

`wwwroot/css/site.css`:

```css
#components-reconnect-modal {
    display: none;
}

#components-reconnect-modal.components-reconnect-show {
    display: block;
}
```

Nella tabella seguente vengono descritte le classi CSS applicate all' `components-reconnect-modal` elemento dal Blazor Framework.

| Classe CSS                       | Indica&hellip; |
| ------------------------------- | ----------------- |
| `components-reconnect-show`     | Connessione persa. È in corso un tentativo di riconnessione del client. Mostra il modale. |
| `components-reconnect-hide`     | Viene ristabilita una connessione attiva al server. Nascondere il modale. |
| `components-reconnect-failed`   | Riconnessione non riuscita. probabilmente a causa di un errore di rete. Per tentare la riconnessione, chiamare `window.Blazor.reconnect()` in JavaScript. |
| `components-reconnect-rejected` | Riconnessione rifiutata. Il server è stato raggiunto ma ha rifiutato la connessione e lo stato dell'utente nel server è andato perso. Per ricaricare l'app, chiamare `location.reload()` in JavaScript. Questo stato di connessione può verificarsi nei casi seguenti:<ul><li>Si verifica un arresto anomalo del circuito sul lato server.</li><li>Il client viene disconnesso abbastanza a lungo da consentire al server di eliminare lo stato dell'utente. Le istanze dei componenti dell'utente sono state eliminate.</li><li>Il server viene riavviato oppure il processo di lavoro dell'app viene riciclato.</li></ul> |

## <a name="render-mode"></a>Modalità di rendering

Per impostazione predefinita, Blazor Server le app preeseguono il rendering dell'interfaccia utente nel server prima che venga stabilita la connessione client al server. Per altre informazioni, vedere <xref:mvc/views/tag-helpers/builtin-th/component-tag-helper>.

## <a name="initialize-the-blazor-circuit"></a>Inizializzare il Blazor circuito

Configurare l'avvio manuale del Blazor Server [ SignalR circuito](xref:blazor/hosting-models#circuits) di un'app nel `Pages/_Host.cshtml` file:

* Aggiungere un `autostart="false"` attributo al `<script>` tag per lo `blazor.server.js` script.
* Inserire uno script che chiama `Blazor.start` dopo il `blazor.server.js` tag dello script e all'interno del `</body>` tag di chiusura.

Quando `autostart` è disabilitato, qualsiasi aspetto dell'app che non dipende dal circuito funziona normalmente. Il routing lato client, ad esempio, è operativo. Tuttavia, qualsiasi aspetto che dipende dal circuito non è operativo fino a quando non `Blazor.start` viene chiamato il metodo. Il comportamento dell'app è imprevedibile senza un circuito stabilito. Ad esempio, i metodi dei componenti non vengono eseguiti mentre il circuito è disconnesso.

### <a name="initialize-blazor-when-the-document-is-ready"></a>Inizializza Blazor quando il documento è pronto

`Pages/_Host.cshtml`:

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

### <a name="chain-to-the-promise-that-results-from-a-manual-start"></a>Concatenare a `Promise` che risulta da un avvio manuale

Per eseguire attività aggiuntive, ad esempio l'inizializzazione dell'interoperabilità JS, usare [`then`](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Promise/then) per concatenare al [`Promise`](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Promise) risultante dall'avvio di un' Blazor app manuale.

`Pages/_Host.cshtml`:

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

### <a name="configure-signalr-client-logging"></a>Configurare la SignalR registrazione client

Nel generatore client passare l' `configureSignalR` oggetto di configurazione che chiama `configureLogging` con il livello di registrazione.

`Pages/_Host.cshtml`:

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

Nell'esempio precedente, `information` è equivalente a un livello di log di <xref:Microsoft.Extensions.Logging.LogLevel.Information?displayProperty=nameWithType> .

### <a name="modify-the-reconnection-handler"></a>Modificare il gestore di riconnessione

Gli eventi di connessione del circuito del gestore di riconnessione possono essere modificati per comportamenti personalizzati, ad esempio:

* Per inviare una notifica all'utente se la connessione viene eliminata.
* Per eseguire la registrazione (dal client) quando si connette un circuito.

Per modificare gli eventi di connessione, registrare i callback per le seguenti modifiche della connessione:

* Connessioni eliminate utilizzare `onConnectionDown` .
* Le connessioni stabilite/ristabilite usano `onConnectionUp` .

**`onConnectionDown` `onConnectionUp` È necessario specificare sia che.**

`Pages/_Host.cshtml`:

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

### <a name="adjust-the-reconnection-retry-count-and-interval"></a>Modificare il numero di tentativi di riconnessione e l'intervallo

Per modificare il numero di tentativi di riconnessione e l'intervallo, impostare il numero di tentativi ( `maxRetries` ) e il periodo di tempo in millisecondi consentiti per ogni nuovo tentativo ( `retryIntervalMilliseconds` ).

`Pages/_Host.cshtml`:

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

## <a name="hide-or-replace-the-reconnection-display"></a>Nascondi o Sostituisci la visualizzazione di riconnessione

Per nascondere la visualizzazione della riconnessione, impostare il gestore di riconnessione `_reconnectionDisplay` su un oggetto vuoto ( `{}` o `new Object()` ).

`Pages/_Host.cshtml`:

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

Per sostituire la visualizzazione della riconnessione, impostare `_reconnectionDisplay` nell'esempio precedente sull'elemento per la visualizzazione:

```javascript
Blazor.defaultReconnectionHandler._reconnectionDisplay = 
  document.getElementById("{ELEMENT ID}");
```

Il segnaposto `{ELEMENT ID}` è l'ID dell'elemento HTML da visualizzare.

::: moniker range=">= aspnetcore-5.0"

Personalizzare il ritardo prima che la visualizzazione della riconnessione venga visualizzata impostando la `transition-delay` proprietà nel CSS del sito per l'elemento modale. Nell'esempio seguente viene impostato il ritardo di transizione da 500 ms (impostazione predefinita) a 1.000 ms (1 secondo).

`wwwroot/css/site.css`:

```css
#components-reconnect-modal {
    transition: visibility 0s linear 1000ms;
}
```

## <a name="disconnect-the-blazor-circuit-from-the-client"></a>Disconnettere il Blazor circuito dal client

Per impostazione predefinita, un Blazor circuito viene disconnesso quando viene attivato l' [ `unload` evento della pagina](https://developer.mozilla.org/docs/Web/API/Window/unload_event) . Per disconnettere il circuito per altri scenari nel client, richiamare `Blazor.disconnect` nel gestore eventi appropriato. Nell'esempio seguente il circuito viene disconnesso quando la pagina è nascosta ([ `pagehide` evento](https://developer.mozilla.org/docs/Web/API/Window/pagehide_event)):

```javascript
window.addEventListener('pagehide', () => {
  Blazor.disconnect();
});
```

::: moniker-end

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:signalr/introduction>
* <xref:signalr/configuration>
* <xref:blazor/security/server/threat-mitigation>
* [Blazor Server eventi di riconnessione e eventi del ciclo di vita del componente](xref:blazor/components/lifecycle#blazor-server-reconnection-events)

::: zone-end
