---
title: Autenticazione e autorizzazione in gRPC per ASP.NET Core
author: jamesnk
description: Informazioni su come usare l'autenticazione e l'autorizzazione in gRPC per ASP.NET Core.
monikerRange: '>= aspnetcore-3.0'
ms.author: jamesnk
ms.date: 05/26/2020
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
uid: grpc/authn-and-authz
ms.openlocfilehash: 5120964459a81ef2d668877bb08ecde512c2389d
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102587996"
---
# <a name="authentication-and-authorization-in-grpc-for-aspnet-core"></a>Autenticazione e autorizzazione in gRPC per ASP.NET Core

Di [James Newton-King](https://twitter.com/jamesnk)

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/grpc/authn-and-authz/sample/) [(procedura per il download)](xref:index#how-to-download-a-sample)

## <a name="authenticate-users-calling-a-grpc-service"></a>Autenticare gli utenti che chiamano un servizio gRPC

gRPC può essere usato con [l'autenticazione ASP.NET Core](xref:security/authentication/identity) per associare un utente a ogni chiamata.

Di seguito è riportato un esempio `Startup.Configure` che usa gRPC e l'autenticazione ASP.NET Core:

```csharp
public void Configure(IApplicationBuilder app)
{
    app.UseRouting();
    
    app.UseAuthentication();
    app.UseAuthorization();

    app.UseEndpoints(endpoints =>
    {
        endpoints.MapGrpcService<GreeterService>();
    });
}
```

> [!NOTE]
> L'ordine in cui si registra il middleware di autenticazione ASP.NET Core è importante. Chiamare sempre `UseAuthentication` e `UseAuthorization` dopo `UseRouting` e prima di `UseEndpoints` .

Il meccanismo di autenticazione usato dall'app durante una chiamata deve essere configurato. La configurazione dell'autenticazione è `Startup.ConfigureServices` stata aggiunta in e sarà diversa a seconda del meccanismo di autenticazione usato dall'app. Per esempi di come proteggere le app ASP.NET Core, vedere [esempi di autenticazione](xref:security/authentication/samples).

Al termine dell'installazione dell'autenticazione, l'utente può accedere ai metodi del servizio gRPC tramite `ServerCallContext` .

```csharp
public override Task<BuyTicketsResponse> BuyTickets(
    BuyTicketsRequest request, ServerCallContext context)
{
    var user = context.GetHttpContext().User;

    // ... access data from ClaimsPrincipal ...
}

```

### <a name="bearer-token-authentication"></a>Autenticazione del token di porta

Il client può fornire un token di accesso per l'autenticazione. Il server convalida il token e lo usa per identificare l'utente.

Nel server bearer token autenticazione viene configurata con il [middleware di JWT Bearer](/dotnet/api/microsoft.extensions.dependencyinjection.jwtbearerextensions.addjwtbearer).

Nel client .NET gRPC, il token può essere inviato con chiamate tramite la `Metadata` raccolta. Le voci della `Metadata` raccolta vengono inviate con una chiamata gRPC come intestazioni http:

```csharp
public bool DoAuthenticatedCall(
    Ticketer.TicketerClient client, string token)
{
    var headers = new Metadata();
    headers.Add("Authorization", $"Bearer {token}");

    var request = new BuyTicketsRequest { Count = 1 };
    var response = await client.BuyTicketsAsync(request, headers);

    return response.Success;
}
```

La configurazione di `ChannelCredentials` su un canale rappresenta un metodo alternativo per inviare il token al servizio con chiamate gRPC. Un oggetto `ChannelCredentials` può includere `CallCredentials` , che fornisce un modo per impostare automaticamente `Metadata` .

`CallCredentials` viene eseguito ogni volta che viene effettuata una chiamata gRPC, evitando la necessità di scrivere codice in più posizioni per passare il token. Si noti che `CallCredentials` vengono applicati solo se il canale è protetto con TLS. `CallCredentials` non vengono applicati ai canali non TLS non protetti.

La credenziale nell'esempio seguente configura il canale per l'invio del token con ogni chiamata a gRPC:

```csharp
private static GrpcChannel CreateAuthenticatedChannel(string address)
{
    var credentials = CallCredentials.FromInterceptor((context, metadata) =>
    {
        if (!string.IsNullOrEmpty(_token))
        {
            metadata.Add("Authorization", $"Bearer {_token}");
        }
        return Task.CompletedTask;
    });

    // SslCredentials is used here because this channel is using TLS.
    // CallCredentials can't be used with ChannelCredentials.Insecure on non-TLS channels.
    var channel = GrpcChannel.ForAddress(address, new GrpcChannelOptions
    {
        Credentials = ChannelCredentials.Create(new SslCredentials(), credentials)
    });
    return channel;
}
```

### <a name="client-certificate-authentication"></a>Autenticazione del certificato client

Un client può in alternativa fornire un certificato client per l'autenticazione. L' [autenticazione del certificato](https://tools.ietf.org/html/rfc5246#section-7.4.4) viene eseguita a livello di TLS, molto prima che venga mai ASP.NET Core. Quando la richiesta entra ASP.NET Core, il [pacchetto di autenticazione del certificato client](xref:security/authentication/certauth) consente di risolvere il certificato in un `ClaimsPrincipal` .

> [!NOTE]
> Configurare il server in modo che accetti i certificati client. Per informazioni sull'accettazione dei certificati client in gheppio, IIS e Azure, vedere <xref:security/authentication/certauth#configure-your-server-to-require-certificates> .

Nel client .NET gRPC, il certificato client viene aggiunto a `HttpClientHandler` che viene quindi usato per creare il client di gRPC:

```csharp
public Ticketer.TicketerClient CreateClientWithCert(
    string baseAddress,
    X509Certificate2 certificate)
{
    // Add client cert to the handler
    var handler = new HttpClientHandler();
    handler.ClientCertificates.Add(certificate);

    // Create the gRPC channel
    var channel = GrpcChannel.ForAddress(baseAddress, new GrpcChannelOptions
    {
        HttpHandler = handler
    });

    return new Ticketer.TicketerClient(channel);
}
```

### <a name="other-authentication-mechanisms"></a>Altri meccanismi di autenticazione

Molti ASP.NET Core meccanismi di autenticazione supportati funzionano con gRPC:

* Azure Active Directory
* Certificato client
* IdentityServer
* Token JWT
* OAuth 2.0
* OpenID Connect
* WS-Federation

Per ulteriori informazioni sulla configurazione dell'autenticazione nel server, vedere [ASP.NET Core Authentication](xref:security/authentication/identity).

La configurazione del client di gRPC per l'uso dell'autenticazione dipende dal meccanismo di autenticazione usato. Gli esempi di bearer token e del certificato client precedenti illustrano due modi per configurare il client gRPC per l'invio di metadati di autenticazione con chiamate gRPC:

* I client gRPC fortemente tipizzati utilizzano `HttpClient` internamente. L'autenticazione può essere configurata in [HttpClientHandler](/dotnet/api/system.net.http.httpclienthandler)o aggiungendo istanze [HttpMessageHandler](/dotnet/api/system.net.http.httpmessagehandler) personalizzate a `HttpClient` .
* Ogni chiamata a gRPC ha un `CallOptions` argomento facoltativo. Le intestazioni personalizzate possono essere inviate tramite la raccolta di intestazioni dell'opzione.

> [!NOTE]
> Non è possibile usare l'autenticazione di Windows (NTLM/Kerberos/Negotiate) con gRPC. gRPC richiede HTTP/2 e HTTP/2 non supporta l'autenticazione di Windows.

## <a name="authorize-users-to-access-services-and-service-methods"></a>Autorizzare gli utenti ad accedere ai servizi e ai metodi del servizio

Per impostazione predefinita, tutti i metodi di un servizio possono essere chiamati da utenti non autenticati. Per richiedere l'autenticazione, applicare l' [`[Authorize]`](xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute) attributo al servizio:

```csharp
[Authorize]
public class TicketerService : Ticketer.TicketerBase
{
}
```

È possibile utilizzare gli argomenti del costruttore e le proprietà dell' `[Authorize]` attributo per limitare l'accesso solo agli utenti che corrispondono a [criteri di autorizzazione](xref:security/authorization/policies)specifici. Se, ad esempio, si dispone di un criterio di autorizzazione personalizzato denominato `MyAuthorizationPolicy` , assicurarsi che solo gli utenti che corrispondono a tale criterio possano accedere al servizio usando il codice seguente:

```csharp
[Authorize("MyAuthorizationPolicy")]
public class TicketerService : Ticketer.TicketerBase
{
}
```

È possibile applicare anche l'attributo a singoli metodi del servizio `[Authorize]` . Se l'utente corrente non corrisponde ai criteri applicati **sia** al metodo che alla classe, al chiamante viene restituito un errore:

```csharp
[Authorize]
public class TicketerService : Ticketer.TicketerBase
{
    public override Task<AvailableTicketsResponse> GetAvailableTickets(
        Empty request, ServerCallContext context)
    {
        // ... buy tickets for the current user ...
    }

    [Authorize("Administrators")]
    public override Task<BuyTicketsResponse> RefundTickets(
        BuyTicketsRequest request, ServerCallContext context)
    {
        // ... refund tickets (something only Administrators can do) ..
    }
}
```

## <a name="additional-resources"></a>Risorse aggiuntive

* [Autenticazione del token di porta in ASP.NET Core](https://blogs.msdn.microsoft.com/webdev/2016/10/27/bearer-token-authentication-in-asp-net-core/)
* [Configurare l'autenticazione del certificato client in ASP.NET Core](xref:security/authentication/certauth)
