---
title: Servizi gRPC con ASP.NET Core
author: juntaoluo
description: Informazioni sui concetti di base per la scrittura di servizi gRPC con ASP.NET Core.
monikerRange: '>= aspnetcore-3.0'
ms.author: johluo
ms.date: 01/29/2021
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
uid: grpc/aspnetcore
ms.openlocfilehash: 57edfa31079cb3fca6e9e8d0fa55bcbb8bbfefca
ms.sourcegitcommit: 83524f739dd25fbfa95ee34e95342afb383b49fe
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 01/29/2021
ms.locfileid: "99057480"
---
# <a name="grpc-services-with-aspnet-core"></a>Servizi gRPC con ASP.NET Core

Questo documento illustra come iniziare a usare i servizi di gRPC con ASP.NET Core.

[!INCLUDE[](~/includes/gRPCazure.md)]

## <a name="prerequisites"></a>Prerequisiti

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

[!INCLUDE[](~/includes/net-core-prereqs-vs-3.0.md)]

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

[!INCLUDE[](~/includes/net-core-prereqs-vsc-3.0.md)]

# <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

[!INCLUDE[](~/includes/net-core-prereqs-mac-3.0.md)]

---

## <a name="get-started-with-grpc-service-in-aspnet-core"></a>Introduzione all'uso del servizio gRPC in ASP.NET Core

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/tutorials/grpc/grpc-start/sample) ([procedura per il download](xref:index#how-to-download-a-sample)).

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

Per istruzioni dettagliate su come creare un progetto gRPC, vedere [Introduzione ai servizi gRPC](xref:tutorials/grpc/grpc-start) .

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio per Mac](#tab/visual-studio-code+visual-studio-mac)

Eseguire `dotnet new grpc -o GrpcGreeter` dalla riga di comando.

---

## <a name="add-grpc-services-to-an-aspnet-core-app"></a>Aggiungere servizi gRPC a un'app ASP.NET Core

gRPC richiede il pacchetto [gRPC. AspNetCore](https://www.nuget.org/packages/Grpc.AspNetCore) .

### <a name="configure-grpc"></a>Configurare gRPC

In *Startup.cs*:

* gRPC è abilitato con il `AddGrpc` metodo.
* Ogni servizio gRPC viene aggiunto alla pipeline di routing tramite il `MapGrpcService` metodo.

[!code-csharp[](~/tutorials/grpc/grpc-start/sample/GrpcGreeter/Startup.cs?name=snippet&highlight=7,24)]
[!INCLUDE[about the series](~/includes/code-comments-loc.md)]

ASP.NET Core middleware e le funzionalità condividono la pipeline di routing, di conseguenza è possibile configurare un'app per gestire gestori di richieste aggiuntivi. I gestori di richieste aggiuntivi, ad esempio i controller MVC, funzionano in parallelo con i servizi gRPC configurati.

## <a name="server-options"></a>Opzioni server

i servizi gRPC possono essere ospitati da tutti i server ASP.NET Core predefiniti.

> [!div class="checklist"]
>
> * Kestrel
> * TestServer
> * IIS&dagger;
> * HTTP.sys&dagger;

&dagger;IIS e HTTP.sys richiedono .NET 5 e Windows 10 Build 20241 o versione successiva.

Per ulteriori informazioni sulla scelta del server appropriato per un'app ASP.NET Core, vedere <xref:fundamentals/servers/index> .

::: moniker range=">= aspnetcore-5.0"

## <a name="kestrel"></a>Kestrel

[Gheppio](xref:fundamentals/servers/kestrel) è un server Web multipiattaforma per ASP.NET Core. Gheppio fornisce le prestazioni migliori e l'utilizzo della memoria, ma non alcune delle funzionalità avanzate di HTTP.sys come la condivisione delle porte.

Endpoint gRPC di Gheppio:

* Richiedi HTTP/2.
* Deve essere protetto con [Transport Layer Security (TLS)](https://tools.ietf.org/html/rfc5246).

### <a name="http2"></a>HTTP/2

gRPC richiede HTTP/2. gRPC per ASP.NET Core convalida [HttpRequest. Protocol](xref:Microsoft.AspNetCore.Http.HttpRequest.Protocol%2A) è `HTTP/2` .

Gheppio [supporta http/2](xref:fundamentals/servers/kestrel/http2) nei sistemi operativi più recenti. Per impostazione predefinita, gli endpoint gheppio sono configurati per supportare connessioni HTTP/1.1 e HTTP/2.

### <a name="tls"></a>TLS

Gli endpoint gheppio usati per gRPC devono essere protetti con TLS. In fase di sviluppo, un endpoint protetto con TLS viene creato automaticamente in corrispondenza del momento in `https://localhost:5001` cui è presente il certificato di sviluppo ASP.NET Core. Non è richiesta alcuna configurazione. Un `https` prefisso verifica che l'endpoint gheppio stia usando TLS.

In produzione, è necessario configurare in modo esplicito TLS. Nell'esempio seguente *appsettings.json* viene fornito un endpoint HTTP/2 protetto con TLS:

[!code-json[](~/grpc/aspnetcore/sample/appsettings.json?highlight=4)]

In alternativa, è possibile configurare gli endpoint gheppio in *Program.cs*:

[!code-csharp[](~/grpc/aspnetcore/sample/Program.cs?highlight=7&name=snippet)]

### <a name="protocol-negotiation"></a>Negoziazione del protocollo

TLS viene usato per una maggiore sicurezza della comunicazione. L'handshake TLS [(ALPN)](https://tools.ietf.org/html/rfc7301#section-3) viene utilizzato per negoziare il protocollo di connessione tra il client e il server quando un endpoint supporta più protocolli. Questa negoziazione determina se la connessione utilizza HTTP/1.1 o HTTP/2.

Se un endpoint HTTP/2 viene configurato senza TLS, l'endpoint [ListenOptions. Protocols](xref:fundamentals/servers/kestrel/endpoints#listenoptionsprotocols) deve essere impostato su `HttpProtocols.Http2` . Un endpoint con più protocolli (ad esempio, `HttpProtocols.Http1AndHttp2` ) non può essere usato senza TLS perché non esiste alcuna negoziazione. Per impostazione predefinita, tutte le connessioni all'endpoint non protetto sono HTTP/1.1 e le chiamate a gRPC hanno esito negativo.

Per ulteriori informazioni sull'abilitazione di HTTP/2 e TLS con gheppio, vedere la pagina relativa alla [configurazione dell'endpoint gheppio](xref:fundamentals/servers/kestrel/endpoints).

> [!NOTE]
> macOS non supporta ASP.NET Core gRPC con TLS. Per eseguire correttamente i servizi gRPC in macOS, è necessaria una configurazione aggiuntiva. Per altre informazioni, vedere [Non è possibile avviare un'app ASP.NET Core gRPC in macOS](xref:grpc/troubleshoot#unable-to-start-aspnet-core-grpc-app-on-macos).

## <a name="iis"></a>IIS

[Internet Information Services (IIS)](xref:host-and-deploy/iis/index) è un server Web flessibile, sicuro e gestibile per l'hosting di app Web, incluso ASP.NET Core. .NET 5 e Windows 10 Build 20241 o versione successiva sono necessari per ospitare i servizi gRPC con IIS.

IIS deve essere configurato per l'uso di TLS e HTTP/2. Per altre informazioni, vedere <xref:host-and-deploy/iis/protocols>.

## <a name="httpsys"></a>HTTP.sys

[HTTP.sys](xref:fundamentals/servers/httpsys) è un server Web per ASP.NET Core che può essere eseguito solo in Windows. .NET 5 e Windows 10 Build 20241 o versione successiva sono necessari per ospitare i servizi gRPC con HTTP.sys.

HTTP.sys deve essere configurato per l'uso di TLS e HTTP/2. Per ulteriori informazioni, vedere  [HTTP.sys supporto http/2 del server Web](xref:fundamentals/servers/httpsys#http2-support).

::: moniker-end

::: moniker range="< aspnetcore-5.0"

## <a name="kestrel"></a>Kestrel

[Gheppio](xref:fundamentals/servers/kestrel) è un server Web multipiattaforma per ASP.NET Core. Gheppio fornisce le prestazioni migliori e l'utilizzo della memoria, ma non alcune delle funzionalità avanzate di HTTP.sys come la condivisione delle porte.

Endpoint gRPC di Gheppio:

* Richiedi HTTP/2.
* Deve essere protetto con [Transport Layer Security (TLS)](https://tools.ietf.org/html/rfc5246).

### <a name="http2"></a>HTTP/2

gRPC richiede HTTP/2. gRPC per ASP.NET Core convalida [HttpRequest. Protocol](xref:Microsoft.AspNetCore.Http.HttpRequest.Protocol%2A) è `HTTP/2` .

Gheppio [supporta http/2](xref:fundamentals/servers/kestrel#http2-support) nei sistemi operativi più recenti. Per impostazione predefinita, gli endpoint gheppio sono configurati per supportare connessioni HTTP/1.1 e HTTP/2.

### <a name="tls"></a>TLS

Gli endpoint gheppio usati per gRPC devono essere protetti con TLS. In fase di sviluppo, un endpoint protetto con TLS viene creato automaticamente in corrispondenza del momento in `https://localhost:5001` cui è presente il certificato di sviluppo ASP.NET Core. Non è richiesta alcuna configurazione. Un `https` prefisso verifica che l'endpoint gheppio stia usando TLS.

In produzione, è necessario configurare in modo esplicito TLS. Nell'esempio seguente *appsettings.json* viene fornito un endpoint HTTP/2 protetto con TLS:

[!code-json[](~/grpc/aspnetcore/sample/appsettings.json?highlight=4)]

In alternativa, è possibile configurare gli endpoint gheppio in *Program.cs*:

[!code-csharp[](~/grpc/aspnetcore/sample/Program.cs?highlight=7&name=snippet)]

### <a name="protocol-negotiation"></a>Negoziazione del protocollo

TLS viene usato per una maggiore sicurezza della comunicazione. L'handshake TLS [(ALPN)](https://tools.ietf.org/html/rfc7301#section-3) viene utilizzato per negoziare il protocollo di connessione tra il client e il server quando un endpoint supporta più protocolli. Questa negoziazione determina se la connessione utilizza HTTP/1.1 o HTTP/2.

Se un endpoint HTTP/2 viene configurato senza TLS, l'endpoint [ListenOptions. Protocols](xref:fundamentals/servers/kestrel#listenoptionsprotocols) deve essere impostato su `HttpProtocols.Http2` . Un endpoint con più protocolli (ad esempio, `HttpProtocols.Http1AndHttp2` ) non può essere usato senza TLS perché non esiste alcuna negoziazione. Per impostazione predefinita, tutte le connessioni all'endpoint non protetto sono HTTP/1.1 e le chiamate a gRPC hanno esito negativo.

Per ulteriori informazioni sull'abilitazione di HTTP/2 e TLS con gheppio, vedere la pagina relativa alla [configurazione dell'endpoint gheppio](xref:fundamentals/servers/kestrel#endpoint-configuration).

> [!NOTE]
> macOS non supporta ASP.NET Core gRPC con TLS. Per eseguire correttamente i servizi gRPC in macOS, è necessaria una configurazione aggiuntiva. Per altre informazioni, vedere [Non è possibile avviare un'app ASP.NET Core gRPC in macOS](xref:grpc/troubleshoot#unable-to-start-aspnet-core-grpc-app-on-macos).

::: moniker-end

## <a name="integration-with-aspnet-core-apis"></a>Integrazione con API ASP.NET Core

i servizi gRPC hanno accesso completo alle funzionalità di ASP.NET Core come l' [inserimento delle dipendenze](xref:fundamentals/dependency-injection) e la [registrazione](xref:fundamentals/logging/index). Ad esempio, l'implementazione del servizio può risolvere un servizio logger dal contenitore DI inserimento delle dipendenze tramite il costruttore:

```csharp
public class GreeterService : Greeter.GreeterBase
{
    public GreeterService(ILogger<GreeterService> logger)
    {
    }
}
```

Per impostazione predefinita, l'implementazione del servizio gRPC è in grado di risolvere altri servizi DI con qualsiasi durata (singleton, ambito o temporaneo).

### <a name="resolve-httpcontext-in-grpc-methods"></a>Risolvere HttpContext nei metodi gRPC

L'API gRPC consente di accedere ad alcuni dati del messaggio HTTP/2, ad esempio il metodo, l'host, l'intestazione e i trailer. L'accesso avviene tramite l' `ServerCallContext` argomento passato a ogni metodo gRPC:

[!code-csharp[](~/grpc/aspnetcore/sample/GrcpService/GreeterService.cs?highlight=3-4&name=snippet)]

`ServerCallContext` non fornisce l'accesso completo a `HttpContext` in tutte le API di ASP.NET. Il `GetHttpContext` metodo di estensione fornisce accesso completo a `HttpContext` che rappresenta il messaggio http/2 sottostante nelle API ASP.NET:

[!code-csharp[](~/grpc/aspnetcore/sample/GrcpService/GreeterService2.cs?highlight=6-7&name=snippet)]


## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:tutorials/grpc/grpc-start>
* <xref:grpc/index>
* <xref:grpc/basics>
* <xref:fundamentals/servers/kestrel>
