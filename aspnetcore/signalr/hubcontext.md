---
title: SignalR HubContext
author: bradygaster
description: Informazioni su come usare il SignalR servizio ASP.NET Core HubContext per l'invio di notifiche ai client dall'esterno di un hub.
monikerRange: '>= aspnetcore-2.1'
ms.author: bradyg
ms.custom: mvc
ms.date: 11/12/2019
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
- IHubContext
uid: signalr/hubcontext
ms.openlocfilehash: 2b2939a7692a195c6dc1b8421433a723310b4bd6
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102589205"
---
# <a name="send-messages-from-outside-a-hub"></a>Inviare messaggi dall'esterno di un hub

Di [Mikael Mengistu](https://twitter.com/MikaelM_12)

L' SignalR Hub è l'astrazione principale per l'invio di messaggi ai client connessi al SignalR Server. È anche possibile inviare messaggi da altre posizioni nell'app usando il `IHubContext` servizio. Questo articolo illustra come accedere a SignalR `IHubContext` per inviare notifiche ai client dall'esterno di un hub.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/signalr/hubcontext/sample/) [(procedura per il download)](xref:index#how-to-download-a-sample)

## <a name="get-an-instance-of-ihubcontext"></a>Ottenere un'istanza di IHubContext

In ASP.NET Core SignalR , è possibile accedere a un'istanza di `IHubContext` tramite l'inserimento di dipendenze. È possibile inserire un'istanza di `IHubContext` in un controller, in un middleware o in un altro servizio di. Utilizzare l'istanza di per inviare messaggi ai client.

> [!NOTE]
> Questo comportamento è diverso da quello di ASP.NET 4. x SignalR che usava GlobalHost per fornire l'accesso a `IHubContext` . ASP.NET Core dispone di un Framework di inserimento delle dipendenze che elimina la necessità di questo singleton globale.

### <a name="inject-an-instance-of-ihubcontext-in-a-controller"></a>Inserire un'istanza di IHubContext in un controller

È possibile inserire un'istanza di `IHubContext` in un controller aggiungendola al costruttore:

[!code-csharp[IHubContext](hubcontext/sample/Controllers/HomeController.cs?range=12-19,57)]

Ora, con l'accesso a un'istanza di `IHubContext` , è possibile chiamare i metodi dell'hub come se si trovassero nell'hub.

[!code-csharp[IHubContext](hubcontext/sample/Controllers/HomeController.cs?range=21-25)]

### <a name="get-an-instance-of-ihubcontext-in-middleware"></a>Ottenere un'istanza di IHubContext nel middleware

Accedere all' `IHubContext` all'interno della pipeline del middleware come segue:

```csharp
app.Use(async (context, next) =>
{
    var hubContext = context.RequestServices
                            .GetRequiredService<IHubContext<ChatHub>>();
    //...
    
    if (next != null)
    {
        await next.Invoke();
    }
});
```

> [!NOTE]
> Quando i metodi dell'hub vengono chiamati dall'esterno della `Hub` classe, non esiste alcun chiamante associato alla chiamata. Non è pertanto possibile accedere alle `ConnectionId` `Caller` proprietà, e `Others` .

### <a name="get-an-instance-of-ihubcontext-from-ihost"></a>Ottenere un'istanza di IHubContext da IHost

L'accesso a un `IHubContext` dall'host Web è utile per l'integrazione con aree esterne a ASP.NET Core, ad esempio usando Framework di inserimento di dipendenze di terze parti:

```csharp
    public class Program
    {
        public static void Main(string[] args)
        {
            var host = CreateHostBuilder(args).Build();
            var hubContext = host.Services.GetService(typeof(IHubContext<ChatHub>));
            host.Run();
        }

        public static IHostBuilder CreateHostBuilder(string[] args) =>
            Host.CreateDefaultBuilder(args)
                .ConfigureWebHostDefaults(webBuilder => {
                    webBuilder.UseStartup<Startup>();
                });
    }
```

### <a name="inject-a-strongly-typed-hubcontext"></a>Inserire un HubContext fortemente tipizzato

Per inserire un HubContext fortemente tipizzato, assicurarsi che l'hub erediti da `Hub<T>` . Inserire l'oggetto usando l' `IHubContext<THub, T>` interfaccia anziché `IHubContext<THub>` .

```csharp
public class ChatController : Controller
{
    public IHubContext<ChatHub, IChatClient> _strongChatHubContext { get; }

    public ChatController(IHubContext<ChatHub, IChatClient> chatHubContext)
    {
        _strongChatHubContext = chatHubContext;
    }

    public async Task SendMessage(string user, string message)
    {
        await _strongChatHubContext.Clients.All.ReceiveMessage(user, message);
    }
}
```

Per ulteriori informazioni, vedere [Hub fortemente tipizzati](xref:signalr/hubs#strongly-typed-hubs) .

## <a name="related-resources"></a>Risorse correlate

* [Operazioni preliminari](xref:tutorials/signalr)
* [Hub](xref:signalr/hubs)
* [Pubblicare in Azure](xref:signalr/publish-to-azure-web-app)
