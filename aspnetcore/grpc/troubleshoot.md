---
title: Risolvere i problemi di gRPC in .NET Core
author: jamesnk
description: Risolvere gli errori quando si usa gRPC in .NET Core.
monikerRange: '>= aspnetcore-3.0'
ms.author: jamesnk
ms.custom: mvc
ms.date: 07/09/2020
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
uid: grpc/troubleshoot
ms.openlocfilehash: 1fd89059183300993c7fa78aa8dab1bda247a530
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102586982"
---
# <a name="troubleshoot-grpc-on-net-core"></a>Risolvere i problemi di gRPC in .NET Core

Di [James Newton-King](https://twitter.com/jamesnk)

Questo documento illustra i problemi comuni riscontrati durante lo sviluppo di app gRPC in .NET.

## <a name="mismatch-between-client-and-service-ssltls-configuration"></a>Mancata corrispondenza tra la configurazione SSL/TLS del client e del servizio

Il modello e gli esempi di gRPC usano [Transport Layer Security (TLS)](https://tools.ietf.org/html/rfc5246) per proteggere i servizi gRPC per impostazione predefinita. i client gRPC devono usare una connessione sicura per chiamare correttamente i servizi gRPC protetti.

È possibile verificare che ASP.NET Core servizio gRPC stia usando TLS nei log scritti all'avvio dell'app. Il servizio sarà in ascolto su un endpoint HTTPS:

```
info: Microsoft.Hosting.Lifetime[0]
      Now listening on: https://localhost:5001
info: Microsoft.Hosting.Lifetime[0]
      Application started. Press Ctrl+C to shut down.
info: Microsoft.Hosting.Lifetime[0]
      Hosting environment: Development
```

Il client .NET Core deve usare `https` nell'indirizzo del server per effettuare chiamate con una connessione protetta:

```csharp
static async Task Main(string[] args)
{
    // The port number(5001) must match the port of the gRPC server.
    var channel = GrpcChannel.ForAddress("https://localhost:5001");
    var client = new Greet.GreeterClient(channel);
}
```

Tutte le implementazioni client di gRPC supportano TLS. i client gRPC di altri linguaggi richiedono in genere il canale configurato con `SslCredentials` . `SslCredentials` Specifica il certificato che verrà utilizzato dal client e deve essere utilizzato al posto di credenziali non sicure. Per esempi di configurazione delle diverse implementazioni client di gRPC per l'uso di TLS, vedere [autenticazione gRPC](https://www.grpc.io/docs/guides/auth/).

## <a name="call-a-grpc-service-with-an-untrustedinvalid-certificate"></a>Chiamare un servizio gRPC con un certificato non attendibile/non valido

Il client gRPC .NET richiede che il servizio disponga di un certificato attendibile. Quando si chiama un servizio gRPC senza un certificato attendibile, viene restituito il messaggio di errore seguente:

> Eccezione non gestita. System .NET. http. HttpRequestexception: non è stato possibile stabilire la connessione SSL. vedere l'eccezione interna.
> ---> System.Security.Authentication.AuthenticationException: Il certificato remoto non è stato ritenuto valido dalla procedura di convalida.

Questo errore può essere visualizzato se si sta testando l'app in locale e il certificato di sviluppo HTTPS ASP.NET Core non è attendibile. Per istruzioni su come risolvere questo problema, vedere [Considerare attendibile il certificato di sviluppo di ASP.NET Core HTTPS in Windows e macOS](xref:security/enforcing-ssl#trust-the-aspnet-core-https-development-certificate-on-windows-and-macos).

Se si chiama un servizio gRPC in un altro computer e non è possibile considerare attendibile il certificato, il client gRPC può essere configurato in modo da ignorare il certificato non valido. Il codice seguente usa [HttpClientHandler. ServerCertificateCustomValidationCallback](/dotnet/api/system.net.http.httpclienthandler.servercertificatecustomvalidationcallback) per consentire le chiamate senza un certificato attendibile:

```csharp
var httpHandler = new HttpClientHandler();
// Return `true` to allow certificates that are untrusted/invalid
httpHandler.ServerCertificateCustomValidationCallback = 
    HttpClientHandler.DangerousAcceptAnyServerCertificateValidator;

var channel = GrpcChannel.ForAddress("https://localhost:5001",
    new GrpcChannelOptions { HttpHandler = httpHandler });
var client = new Greet.GreeterClient(channel);
```

> [!WARNING]
> I certificati non attendibili devono essere usati solo durante lo sviluppo di app. Le app di produzione devono sempre usare certificati validi.

## <a name="call-insecure-grpc-services-with-net-core-client"></a>Chiamare i servizi gRPC non sicuri con il client .NET Core

Il client gRPC .NET può chiamare i servizi gRPC non protetti da specificando `http` nell'indirizzo del server. Ad esempio: `GrpcChannel.ForAddress("http://localhost:5000")`.

Sono necessari alcuni requisiti aggiuntivi per chiamare i servizi gRPC non sicuri a seconda della versione di .NET usata da un'app:

* .NET 5 o versioni successive richiede [Grpc .NET. client](https://www.nuget.org/packages/Grpc.Net.Client) versione 2.32.0 o successiva.
* .NET Core 3. x richiede una configurazione aggiuntiva. L'app deve impostare l' `System.Net.Http.SocketsHttpHandler.Http2UnencryptedSupport` opzione su `true` :
    
    ```csharp
    // This switch must be set before creating the GrpcChannel/HttpClient.
    AppContext.SetSwitch(
        "System.Net.Http.SocketsHttpHandler.Http2UnencryptedSupport", true);
    
    // The port number(5000) must match the port of the gRPC server.
    var channel = GrpcChannel.ForAddress("http://localhost:5000");
    var client = new Greet.GreeterClient(channel);
    ```

L' `System.Net.Http.SocketsHttpHandler.Http2UnencryptedSupport` opzione è necessaria solo per .NET Core 3. x. Non esegue alcuna operazione in .NET 5 e non è obbligatorio.

## <a name="unable-to-start-aspnet-core-grpc-app-on-macos"></a>Non è possibile avviare ASP.NET Core app gRPC in macOS

Gheppio non supporta HTTP/2 con TLS in macOS e versioni precedenti di Windows, ad esempio Windows 7. Per impostazione predefinita, il ASP.NET Core modello e gli esempi di gRPC usano TLS. Quando si tenta di avviare il server gRPC, viene visualizzato il messaggio di errore seguente:

> Non è possibile eseguire l'associazione a nell' https://localhost:5001 interfaccia di loopback IPv4:' http/2 su TLS non è supportato in MacOS perché manca il supporto di ALPN '.

Per risolvere questo problema, configurare gheppio e il client gRPC per l'uso di HTTP/2 *senza* TLS. Questa operazione deve essere eseguita solo durante lo sviluppo. Se non si usa TLS, i messaggi gRPC vengono inviati senza crittografia.

Gheppio deve configurare un endpoint HTTP/2 senza TLS in *Program.cs*:

```csharp
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.ConfigureKestrel(options =>
            {
                // Setup a HTTP/2 endpoint without TLS.
                options.ListenLocalhost(5000, o => o.Protocols = 
                    HttpProtocols.Http2);
            });
            webBuilder.UseStartup<Startup>();
        });
```

::: moniker range=">= aspnetcore-5.0"
Quando un endpoint HTTP/2 viene configurato senza TLS, l'endpoint [ListenOptions. Protocols](xref:fundamentals/servers/kestrel/endpoints#listenoptionsprotocols) deve essere impostato su `HttpProtocols.Http2` . `HttpProtocols.Http1AndHttp2` non può essere usato perché TLS è necessario per negoziare HTTP/2. Senza TLS, tutte le connessioni all'endpoint vengono predefinite a HTTP/1.1 e le chiamate a gRPC hanno esito negativo.
::: moniker-end

::: moniker range="< aspnetcore-5.0"
Quando un endpoint HTTP/2 viene configurato senza TLS, l'endpoint [ListenOptions. Protocols](xref:fundamentals/servers/kestrel#listenoptionsprotocols) deve essere impostato su `HttpProtocols.Http2` . `HttpProtocols.Http1AndHttp2` non può essere usato perché TLS è necessario per negoziare HTTP/2. Senza TLS, tutte le connessioni all'endpoint vengono predefinite a HTTP/1.1 e le chiamate a gRPC hanno esito negativo.
::: moniker-end

Il client di gRPC deve essere configurato anche per non usare TLS. Per altre informazioni, vedere [chiamare i servizi gRPC non sicuri con il client .NET Core](#call-insecure-grpc-services-with-net-core-client).

> [!WARNING]
> HTTP/2 senza TLS deve essere usato solo durante lo sviluppo di app. Le app di produzione devono sempre usare la sicurezza del trasporto. Per ulteriori informazioni, vedere [considerazioni sulla sicurezza in gRPC per ASP.NET Core](xref:grpc/security#transport-security).

## <a name="grpc-c-assets-are-not-code-generated-from-proto-files"></a>gli asset C# di gRPC non sono generati da file con estensione proto

per la generazione di codice gRPC di client concreti e classi di base del servizio è necessario fare riferimento a file e strumenti protobuf da un progetto. È necessario includere:

* file con *estensione proto* che si desidera utilizzare nel `<Protobuf>` gruppo di elementi. Il progetto deve fare riferimento ai [file *. proto* importati](https://developers.google.com/protocol-buffers/docs/proto3#importing-definitions) .
* Riferimento al pacchetto per il pacchetto di [strumenti GRPC gRPC. Tools](https://www.nuget.org/packages/Grpc.Tools/).

Per ulteriori informazioni sulla generazione di asset gRPC C#, vedere <xref:grpc/basics> .

Un ASP.NET Core app Web che ospita i servizi gRPC richiede solo la classe di base del servizio generata:

```xml
<ItemGroup>
  <Protobuf Include="Protos\greet.proto" GrpcServices="Server" />
</ItemGroup>
```

Un'app client gRPC che effettua chiamate gRPC richiede solo il client concreto generato:

```xml
<ItemGroup>
  <Protobuf Include="Protos\greet.proto" GrpcServices="Client" />
</ItemGroup>
```

## <a name="wpf-projects-unable-to-generate-grpc-c-assets-from-proto-files"></a>I progetti WPF non sono in grado di generare asset C# gRPC da file. proto

I progetti WPF presentano un [problema noto](https://github.com/dotnet/wpf/issues/810) che impedisce il corretto funzionamento della generazione del codice gRPC. Tutti i tipi di gRPC generati in un progetto WPF facendo riferimento ai `Grpc.Tools` file *. proto e. proto* creeranno errori di compilazione quando vengono utilizzati:

> errore CS0246: Impossibile trovare il tipo o il nome dello spazio dei nomi "MyGrpcServices". manca una direttiva using o un riferimento a un assembly.

Per aggirare questo problema, è possibile:

1. Creare un nuovo progetto libreria di classi .NET Core.
2. Nel nuovo progetto aggiungere i riferimenti per abilitare la [generazione di codice C# da file *\* . proto*](xref:grpc/basics#generated-c-assets):
    * Aggiungere un riferimento al pacchetto [Grpc. Tools](https://www.nuget.org/packages/Grpc.Tools/) .
    * Aggiungere i file *\* . proto* al `<Protobuf>` gruppo di elementi.
3. Nell'applicazione WPF aggiungere un riferimento al nuovo progetto.

L'applicazione WPF può utilizzare i tipi generati da gRPC dal nuovo progetto libreria di classi.

[!INCLUDE[](~/includes/gRPCazure.md)]
