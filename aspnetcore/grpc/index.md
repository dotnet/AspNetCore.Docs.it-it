---
title: Introduzione a gRPC in .NET
author: juntaoluo
description: Informazioni sui servizi gRPC con il server Kestrel e lo stack di ASP.NET Core.
monikerRange: '>= aspnetcore-3.0'
ms.author: johluo
ms.date: 09/20/2019
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
uid: grpc/index
ms.openlocfilehash: 5820049aba90a2fbd06a23756b12ac9656c3b2c4
ms.sourcegitcommit: 83524f739dd25fbfa95ee34e95342afb383b49fe
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 01/29/2021
ms.locfileid: "99057512"
---
# <a name="introduction-to-grpc-on-net"></a>Introduzione a gRPC in .NET

Di [John Luo](https://github.com/juntaoluo) e [James Newton-King](https://twitter.com/jamesnk)

[gRPC](https://grpc.io/docs/guides/) è un Framework RPC (Remote Procedure Call) a prestazioni elevate e indipendente dal linguaggio.

I principali vantaggi di gRPC sono:
* Framework RPC leggero e moderno ad alte prestazioni.
* Sviluppo di API con priorità al contratto usando i buffer del protocollo per impostazione predefinita e implementazioni indipendenti dal linguaggio.
* Strumenti disponibili per molte linguaggi consentono di generare client e server fortemente tipizzati.
* Supporto per chiamate client, server e di streaming bidirezionale.
* Utilizzo di rete ridotto grazie alla serializzazione binaria Protobuf.

Questi vantaggi rendono gRPC ideale per:
* Microservizi leggeri in cui l'efficienza è fondamentale.
* Sistemi poliglotti che richiedono l'uso di più linguaggi per lo sviluppo.
* Servizi in tempo reale da punto a punto che devono gestire richieste o risposte di streaming.

[!INCLUDE[](~/includes/gRPCazure.md)]

## <a name="c-tooling-support-for-proto-files"></a>Supporto degli strumenti C# per i file. proto

gRPC usa un approccio basato sul contratto per lo sviluppo di API. I servizi e i messaggi vengono definiti nei file *\* . proto* :

```protobuf
syntax = "proto3";

service Greeter {
  rpc SayHello (HelloRequest) returns (HelloReply);
}

message HelloRequest {
  string name = 1;
}

message HelloReply {
  string message = 1;
}
```

I tipi .NET per servizi, client e messaggi vengono generati automaticamente includendo file con *\* estensione proto* in un progetto:

* Aggiungere un riferimento al pacchetto [Grpc. Tools](https://www.nuget.org/packages/Grpc.Tools/) .
* Aggiungere i file *\* . proto* al `<Protobuf>` gruppo di elementi.

```xml
<ItemGroup>
  <Protobuf Include="Protos\greet.proto" />
</ItemGroup>
```

Per ulteriori informazioni sul supporto per gli strumenti gRPC, vedere <xref:grpc/basics> .

## <a name="grpc-services-on-aspnet-core"></a>Servizi gRPC in ASP.NET Core

i servizi gRPC possono essere ospitati in ASP.NET Core. I servizi hanno un'integrazione completa con le funzionalità DI ASP.NET Core più diffuse, ad esempio registrazione, inserimento DI dipendenze, autenticazione e autorizzazione.

Il modello di progetto di servizio gRPC fornisce un servizio Starter:

```csharp
public class GreeterService : Greeter.GreeterBase
{
    private readonly ILogger<GreeterService> _logger;

    public GreeterService(ILogger<GreeterService> logger)
    {
        _logger = logger;
    }

    public override Task<HelloReply> SayHello(HelloRequest request,
        ServerCallContext context)
    {
        _logger.LogInformation("Saying hello to {Name}", request.Name);
        return Task.FromResult(new HelloReply 
        {
            Message = "Hello " + request.Name
        });
    }
}
```

`GreeterService`eredita dal `GreeterBase` tipo, generato dal `Greeter` servizio nel file *\* . proto* . Il servizio è reso accessibile ai client in *Startup.cs*:

```csharp
app.UseEndpoints(endpoints =>
{
    endpoints.MapGrpcService<GreeterService>();
});
```

Per ulteriori informazioni sui servizi gRPC in ASP.NET Core, vedere <xref:grpc/aspnetcore> .

## <a name="call-grpc-services-with-a-net-client"></a>Chiamare i servizi gRPC con un client .NET

i client gRPC sono tipi di client concreti [generati da file *\* . proto*](xref:grpc/basics#generated-c-assets). Il client gRPC concreto dispone di metodi che vengono convertiti nel servizio gRPC nel file *\* . proto* .

```csharp
var channel = GrpcChannel.ForAddress("https://localhost:5001");
var client = new Greeter.GreeterClient(channel);

var response = await client.SayHelloAsync(
    new HelloRequest { Name = "World" });

Console.WriteLine(response.Message);
```

Un client gRPC viene creato usando un canale che rappresenta una connessione di lunga durata a un servizio gRPC. Un canale può essere creato usando `GrpcChannel.ForAddress` .

Per ulteriori informazioni sulla creazione di client e sulla chiamata di diversi metodi di servizio, vedere <xref:grpc/client> .

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:grpc/basics>
* <xref:grpc/aspnetcore>
* <xref:grpc/client>
* <xref:grpc/clientfactory>
* <xref:tutorials/grpc/grpc-start>
