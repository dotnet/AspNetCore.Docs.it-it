---
title: Comunicazione tra processi con gRPC
author: jamesnk
description: Informazioni su come usare gRPC per la comunicazione tra processi.
monikerRange: '>= aspnetcore-5.0'
ms.author: jamesnk
ms.date: 09/16/2020
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
uid: grpc/interprocess
ms.openlocfilehash: 8c0f8fb1468e61d5aa2e7f42cb5da33c01819124
ms.sourcegitcommit: 7e394a8527c9818caebb940f692ae4fcf2f1b277
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 01/31/2021
ms.locfileid: "99217466"
---
# <a name="inter-process-communication-with-grpc"></a><span data-ttu-id="b2a08-103">Comunicazione tra processi con gRPC</span><span class="sxs-lookup"><span data-stu-id="b2a08-103">Inter-process communication with gRPC</span></span>

<span data-ttu-id="b2a08-104">Di [James Newton-King](https://twitter.com/jamesnk)</span><span class="sxs-lookup"><span data-stu-id="b2a08-104">By [James Newton-King](https://twitter.com/jamesnk)</span></span>

<span data-ttu-id="b2a08-105">le chiamate gRPC tra un client e un servizio vengono in genere inviate tramite socket TCP.</span><span class="sxs-lookup"><span data-stu-id="b2a08-105">gRPC calls between a client and service are usually sent over TCP sockets.</span></span> <span data-ttu-id="b2a08-106">Il protocollo TCP è stato progettato per la comunicazione attraverso una rete.</span><span class="sxs-lookup"><span data-stu-id="b2a08-106">TCP was designed for communicating across a network.</span></span> <span data-ttu-id="b2a08-107">La [comunicazione interprocesso (IPC)](https://wikipedia.org/wiki/Inter-process_communication) è più efficiente rispetto a TCP quando il client e il servizio si trovano nello stesso computer.</span><span class="sxs-lookup"><span data-stu-id="b2a08-107">[Inter-process communication (IPC)](https://wikipedia.org/wiki/Inter-process_communication) is more efficient than TCP when the client and service are on the same machine.</span></span> <span data-ttu-id="b2a08-108">Questo documento illustra come usare gRPC con i trasporti personalizzati negli scenari IPC.</span><span class="sxs-lookup"><span data-stu-id="b2a08-108">This document explains how to use gRPC with custom transports in IPC scenarios.</span></span>

## <a name="server-configuration"></a><span data-ttu-id="b2a08-109">Configurazione del server</span><span class="sxs-lookup"><span data-stu-id="b2a08-109">Server configuration</span></span>

<span data-ttu-id="b2a08-110">I trasporti personalizzati sono supportati da [gheppio](xref:fundamentals/servers/kestrel).</span><span class="sxs-lookup"><span data-stu-id="b2a08-110">Custom transports are supported by [Kestrel](xref:fundamentals/servers/kestrel).</span></span> <span data-ttu-id="b2a08-111">Il gheppio è configurato in *Program.cs*:</span><span class="sxs-lookup"><span data-stu-id="b2a08-111">Kestrel is configured in *Program.cs*:</span></span>

```csharp
public static readonly string SocketPath = Path.Combine(Path.GetTempPath(), "socket.tmp");

public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.UseStartup<Startup>();
            webBuilder.ConfigureKestrel(options =>
            {
                if (File.Exists(SocketPath))
                {
                    File.Delete(SocketPath);
                }
                options.ListenUnixSocket(SocketPath);
            });
        });
```

<span data-ttu-id="b2a08-112">L'esempio precedente:</span><span class="sxs-lookup"><span data-stu-id="b2a08-112">The preceding example:</span></span>

* <span data-ttu-id="b2a08-113">Configura gli endpoint di Gheppio in `ConfigureKestrel` .</span><span class="sxs-lookup"><span data-stu-id="b2a08-113">Configures Kestrel's endpoints in `ConfigureKestrel`.</span></span>
* <span data-ttu-id="b2a08-114">Chiama <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ListenUnixSocket*> per ascoltare un [socket di dominio UNIX (UDS)](https://wikipedia.org/wiki/Unix_domain_socket) con il percorso specificato.</span><span class="sxs-lookup"><span data-stu-id="b2a08-114">Calls <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ListenUnixSocket*> to listen to a [Unix domain socket (UDS)](https://wikipedia.org/wiki/Unix_domain_socket) with the specified path.</span></span>

<span data-ttu-id="b2a08-115">Gheppio dispone del supporto incorporato per gli endpoint UDS.</span><span class="sxs-lookup"><span data-stu-id="b2a08-115">Kestrel has built-in support for UDS endpoints.</span></span> <span data-ttu-id="b2a08-116">I domini di aggiornamento sono supportati nelle versioni Linux, macOS e [moderne di Windows](https://devblogs.microsoft.com/commandline/af_unix-comes-to-windows/).</span><span class="sxs-lookup"><span data-stu-id="b2a08-116">UDS are supported on Linux, macOS and [modern versions of Windows](https://devblogs.microsoft.com/commandline/af_unix-comes-to-windows/).</span></span>

## <a name="client-configuration"></a><span data-ttu-id="b2a08-117">Configurazione del client</span><span class="sxs-lookup"><span data-stu-id="b2a08-117">Client configuration</span></span>

<span data-ttu-id="b2a08-118">`GrpcChannel` supporta l'esecuzione di chiamate gRPC su trasporti personalizzati.</span><span class="sxs-lookup"><span data-stu-id="b2a08-118">`GrpcChannel` supports making gRPC calls over custom transports.</span></span> <span data-ttu-id="b2a08-119">Quando viene creato, un canale può essere configurato con un oggetto con `SocketsHttpHandler` un oggetto personalizzato `ConnectCallback` .</span><span class="sxs-lookup"><span data-stu-id="b2a08-119">When a channel is created, it can be configured with a `SocketsHttpHandler` that has a custom `ConnectCallback`.</span></span> <span data-ttu-id="b2a08-120">Il callback consente al client di stabilire connessioni su trasporti personalizzati, quindi di inviare richieste HTTP tramite il trasporto.</span><span class="sxs-lookup"><span data-stu-id="b2a08-120">The callback allows the client to make connections over custom transports and then send HTTP requests over that transport.</span></span>

<span data-ttu-id="b2a08-121">Esempio di Factory di connessione socket di dominio UNIX:</span><span class="sxs-lookup"><span data-stu-id="b2a08-121">Unix domain sockets connection factory example:</span></span>

```csharp
public class UnixDomainSocketConnectionFactory
{
    private readonly EndPoint _endPoint;

    public UnixDomainSocketConnectionFactory(EndPoint endPoint)
    {
        _endPoint = endPoint;
    }

    public async ValueTask<Stream> ConnectAsync(SocketsHttpConnectionContext _,
        CancellationToken cancellationToken = default)
    {
        var socket = new Socket(AddressFamily.Unix, SocketType.Stream, ProtocolType.Unspecified);

        try
        {
            await socket.ConnectAsync(_endPoint, cancellationToken).ConfigureAwait(false);
            return new NetworkStream(socket, true);
        }
        catch
        {
            socket.Dispose();
            throw;
        }
    }
}
```

<span data-ttu-id="b2a08-122">Uso della factory di connessione personalizzata per creare un canale:</span><span class="sxs-lookup"><span data-stu-id="b2a08-122">Using the custom connection factory to create a channel:</span></span>

```csharp
public static readonly string SocketPath = Path.Combine(Path.GetTempPath(), "socket.tmp");

public static GrpcChannel CreateChannel()
{
    var udsEndPoint = new UnixDomainSocketEndPoint(SocketPath);
    var connectionFactory = new UnixDomainSocketConnectionFactory(udsEndPoint);
    var socketsHttpHandler = new SocketsHttpHandler
    {
        ConnectCallback = connectionFactory.ConnectAsync
    };

    return GrpcChannel.ForAddress("http://localhost", new GrpcChannelOptions
    {
        HttpHandler = socketsHttpHandler
    });
}
```

<span data-ttu-id="b2a08-123">I canali creati usando il codice precedente inviano chiamate gRPC su socket di dominio UNIX.</span><span class="sxs-lookup"><span data-stu-id="b2a08-123">Channels created using the preceding code send gRPC calls over Unix domain sockets.</span></span> <span data-ttu-id="b2a08-124">Il supporto per altre tecnologie IPC può essere implementato usando l'estendibilità in gheppio e `SocketsHttpHandler` .</span><span class="sxs-lookup"><span data-stu-id="b2a08-124">Support for other IPC technologies can be implemented using the extensibility in Kestrel and `SocketsHttpHandler`.</span></span>
