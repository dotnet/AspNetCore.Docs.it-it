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
# <a name="inter-process-communication-with-grpc"></a>Comunicazione tra processi con gRPC

Di [James Newton-King](https://twitter.com/jamesnk)

le chiamate gRPC tra un client e un servizio vengono in genere inviate tramite socket TCP. Il protocollo TCP è stato progettato per la comunicazione attraverso una rete. La [comunicazione interprocesso (IPC)](https://wikipedia.org/wiki/Inter-process_communication) è più efficiente rispetto a TCP quando il client e il servizio si trovano nello stesso computer. Questo documento illustra come usare gRPC con i trasporti personalizzati negli scenari IPC.

## <a name="server-configuration"></a>Configurazione del server

I trasporti personalizzati sono supportati da [gheppio](xref:fundamentals/servers/kestrel). Il gheppio è configurato in *Program.cs*:

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

L'esempio precedente:

* Configura gli endpoint di Gheppio in `ConfigureKestrel` .
* Chiama <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ListenUnixSocket*> per ascoltare un [socket di dominio UNIX (UDS)](https://wikipedia.org/wiki/Unix_domain_socket) con il percorso specificato.

Gheppio dispone del supporto incorporato per gli endpoint UDS. I domini di aggiornamento sono supportati nelle versioni Linux, macOS e [moderne di Windows](https://devblogs.microsoft.com/commandline/af_unix-comes-to-windows/).

## <a name="client-configuration"></a>Configurazione del client

`GrpcChannel` supporta l'esecuzione di chiamate gRPC su trasporti personalizzati. Quando viene creato, un canale può essere configurato con un oggetto con `SocketsHttpHandler` un oggetto personalizzato `ConnectCallback` . Il callback consente al client di stabilire connessioni su trasporti personalizzati, quindi di inviare richieste HTTP tramite il trasporto.

Esempio di Factory di connessione socket di dominio UNIX:

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

Uso della factory di connessione personalizzata per creare un canale:

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

I canali creati usando il codice precedente inviano chiamate gRPC su socket di dominio UNIX. Il supporto per altre tecnologie IPC può essere implementato usando l'estendibilità in gheppio e `SocketsHttpHandler` .
