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
# <a name="authentication-and-authorization-in-grpc-for-aspnet-core"></a><span data-ttu-id="67073-103">Autenticazione e autorizzazione in gRPC per ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="67073-103">Authentication and authorization in gRPC for ASP.NET Core</span></span>

<span data-ttu-id="67073-104">Di [James Newton-King](https://twitter.com/jamesnk)</span><span class="sxs-lookup"><span data-stu-id="67073-104">By [James Newton-King](https://twitter.com/jamesnk)</span></span>

<span data-ttu-id="67073-105">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/grpc/authn-and-authz/sample/) [(procedura per il download)](xref:index#how-to-download-a-sample)</span><span class="sxs-lookup"><span data-stu-id="67073-105">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/grpc/authn-and-authz/sample/) [(how to download)](xref:index#how-to-download-a-sample)</span></span>

## <a name="authenticate-users-calling-a-grpc-service"></a><span data-ttu-id="67073-106">Autenticare gli utenti che chiamano un servizio gRPC</span><span class="sxs-lookup"><span data-stu-id="67073-106">Authenticate users calling a gRPC service</span></span>

<span data-ttu-id="67073-107">gRPC può essere usato con [l'autenticazione ASP.NET Core](xref:security/authentication/identity) per associare un utente a ogni chiamata.</span><span class="sxs-lookup"><span data-stu-id="67073-107">gRPC can be used with [ASP.NET Core authentication](xref:security/authentication/identity) to associate a user with each call.</span></span>

<span data-ttu-id="67073-108">Di seguito è riportato un esempio `Startup.Configure` che usa gRPC e l'autenticazione ASP.NET Core:</span><span class="sxs-lookup"><span data-stu-id="67073-108">The following is an example of `Startup.Configure` which uses gRPC and ASP.NET Core authentication:</span></span>

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
> <span data-ttu-id="67073-109">L'ordine in cui si registra il middleware di autenticazione ASP.NET Core è importante.</span><span class="sxs-lookup"><span data-stu-id="67073-109">The order in which you register the ASP.NET Core authentication middleware matters.</span></span> <span data-ttu-id="67073-110">Chiamare sempre `UseAuthentication` e `UseAuthorization` dopo `UseRouting` e prima di `UseEndpoints` .</span><span class="sxs-lookup"><span data-stu-id="67073-110">Always call `UseAuthentication` and `UseAuthorization` after `UseRouting` and before `UseEndpoints`.</span></span>

<span data-ttu-id="67073-111">Il meccanismo di autenticazione usato dall'app durante una chiamata deve essere configurato.</span><span class="sxs-lookup"><span data-stu-id="67073-111">The authentication mechanism your app uses during a call needs to be configured.</span></span> <span data-ttu-id="67073-112">La configurazione dell'autenticazione è `Startup.ConfigureServices` stata aggiunta in e sarà diversa a seconda del meccanismo di autenticazione usato dall'app.</span><span class="sxs-lookup"><span data-stu-id="67073-112">Authentication configuration is added in `Startup.ConfigureServices` and will be different depending upon the authentication mechanism your app uses.</span></span> <span data-ttu-id="67073-113">Per esempi di come proteggere le app ASP.NET Core, vedere [esempi di autenticazione](xref:security/authentication/samples).</span><span class="sxs-lookup"><span data-stu-id="67073-113">For examples of how to secure ASP.NET Core apps, see [Authentication samples](xref:security/authentication/samples).</span></span>

<span data-ttu-id="67073-114">Al termine dell'installazione dell'autenticazione, l'utente può accedere ai metodi del servizio gRPC tramite `ServerCallContext` .</span><span class="sxs-lookup"><span data-stu-id="67073-114">Once authentication has been setup, the user can be accessed in a gRPC service methods via the `ServerCallContext`.</span></span>

```csharp
public override Task<BuyTicketsResponse> BuyTickets(
    BuyTicketsRequest request, ServerCallContext context)
{
    var user = context.GetHttpContext().User;

    // ... access data from ClaimsPrincipal ...
}

```

### <a name="bearer-token-authentication"></a><span data-ttu-id="67073-115">Autenticazione del token di porta</span><span class="sxs-lookup"><span data-stu-id="67073-115">Bearer token authentication</span></span>

<span data-ttu-id="67073-116">Il client può fornire un token di accesso per l'autenticazione.</span><span class="sxs-lookup"><span data-stu-id="67073-116">The client can provide an access token for authentication.</span></span> <span data-ttu-id="67073-117">Il server convalida il token e lo usa per identificare l'utente.</span><span class="sxs-lookup"><span data-stu-id="67073-117">The server validates the token and uses it to identify the user.</span></span>

<span data-ttu-id="67073-118">Nel server bearer token autenticazione viene configurata con il [middleware di JWT Bearer](/dotnet/api/microsoft.extensions.dependencyinjection.jwtbearerextensions.addjwtbearer).</span><span class="sxs-lookup"><span data-stu-id="67073-118">On the server, bearer token authentication is configured using the [JWT Bearer middleware](/dotnet/api/microsoft.extensions.dependencyinjection.jwtbearerextensions.addjwtbearer).</span></span>

<span data-ttu-id="67073-119">Nel client .NET gRPC, il token può essere inviato con chiamate tramite la `Metadata` raccolta.</span><span class="sxs-lookup"><span data-stu-id="67073-119">In the .NET gRPC client, the token can be sent with calls by using the `Metadata` collection.</span></span> <span data-ttu-id="67073-120">Le voci della `Metadata` raccolta vengono inviate con una chiamata gRPC come intestazioni http:</span><span class="sxs-lookup"><span data-stu-id="67073-120">Entries in the `Metadata` collection are sent with a gRPC call as HTTP headers:</span></span>

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

<span data-ttu-id="67073-121">La configurazione di `ChannelCredentials` su un canale rappresenta un metodo alternativo per inviare il token al servizio con chiamate gRPC.</span><span class="sxs-lookup"><span data-stu-id="67073-121">Configuring `ChannelCredentials` on a channel is an alternative way to send the token to the service with gRPC calls.</span></span> <span data-ttu-id="67073-122">Un oggetto `ChannelCredentials` può includere `CallCredentials` , che fornisce un modo per impostare automaticamente `Metadata` .</span><span class="sxs-lookup"><span data-stu-id="67073-122">A `ChannelCredentials` can include `CallCredentials`, which provide a way to automatically set `Metadata`.</span></span>

<span data-ttu-id="67073-123">`CallCredentials` viene eseguito ogni volta che viene effettuata una chiamata gRPC, evitando la necessità di scrivere codice in più posizioni per passare il token.</span><span class="sxs-lookup"><span data-stu-id="67073-123">`CallCredentials` is run each time a gRPC call is made, which avoids the need to write code in multiple places to pass the token yourself.</span></span> <span data-ttu-id="67073-124">Si noti che `CallCredentials` vengono applicati solo se il canale è protetto con TLS.</span><span class="sxs-lookup"><span data-stu-id="67073-124">Note that `CallCredentials` are only applied if the channel is secured with TLS.</span></span> <span data-ttu-id="67073-125">`CallCredentials` non vengono applicati ai canali non TLS non protetti.</span><span class="sxs-lookup"><span data-stu-id="67073-125">`CallCredentials` aren't applied on unsecured non-TLS channels.</span></span>

<span data-ttu-id="67073-126">La credenziale nell'esempio seguente configura il canale per l'invio del token con ogni chiamata a gRPC:</span><span class="sxs-lookup"><span data-stu-id="67073-126">The credential in the following example configures the channel to send the token with every gRPC call:</span></span>

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

### <a name="client-certificate-authentication"></a><span data-ttu-id="67073-127">Autenticazione del certificato client</span><span class="sxs-lookup"><span data-stu-id="67073-127">Client certificate authentication</span></span>

<span data-ttu-id="67073-128">Un client può in alternativa fornire un certificato client per l'autenticazione.</span><span class="sxs-lookup"><span data-stu-id="67073-128">A client could alternatively provide a client certificate for authentication.</span></span> <span data-ttu-id="67073-129">L' [autenticazione del certificato](https://tools.ietf.org/html/rfc5246#section-7.4.4) viene eseguita a livello di TLS, molto prima che venga mai ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="67073-129">[Certificate authentication](https://tools.ietf.org/html/rfc5246#section-7.4.4) happens at the TLS level, long before it ever gets to ASP.NET Core.</span></span> <span data-ttu-id="67073-130">Quando la richiesta entra ASP.NET Core, il [pacchetto di autenticazione del certificato client](xref:security/authentication/certauth) consente di risolvere il certificato in un `ClaimsPrincipal` .</span><span class="sxs-lookup"><span data-stu-id="67073-130">When the request enters ASP.NET Core, the [client certificate authentication package](xref:security/authentication/certauth) allows you to resolve the certificate to a `ClaimsPrincipal`.</span></span>

> [!NOTE]
> <span data-ttu-id="67073-131">Configurare il server in modo che accetti i certificati client.</span><span class="sxs-lookup"><span data-stu-id="67073-131">Configure the server to accept client certificates.</span></span> <span data-ttu-id="67073-132">Per informazioni sull'accettazione dei certificati client in gheppio, IIS e Azure, vedere <xref:security/authentication/certauth#configure-your-server-to-require-certificates> .</span><span class="sxs-lookup"><span data-stu-id="67073-132">For information on accepting client certificates in Kestrel, IIS, and Azure, see <xref:security/authentication/certauth#configure-your-server-to-require-certificates>.</span></span>

<span data-ttu-id="67073-133">Nel client .NET gRPC, il certificato client viene aggiunto a `HttpClientHandler` che viene quindi usato per creare il client di gRPC:</span><span class="sxs-lookup"><span data-stu-id="67073-133">In the .NET gRPC client, the client certificate is added to `HttpClientHandler` that is then used to create the gRPC client:</span></span>

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

### <a name="other-authentication-mechanisms"></a><span data-ttu-id="67073-134">Altri meccanismi di autenticazione</span><span class="sxs-lookup"><span data-stu-id="67073-134">Other authentication mechanisms</span></span>

<span data-ttu-id="67073-135">Molti ASP.NET Core meccanismi di autenticazione supportati funzionano con gRPC:</span><span class="sxs-lookup"><span data-stu-id="67073-135">Many ASP.NET Core supported authentication mechanisms work with gRPC:</span></span>

* <span data-ttu-id="67073-136">Azure Active Directory</span><span class="sxs-lookup"><span data-stu-id="67073-136">Azure Active Directory</span></span>
* <span data-ttu-id="67073-137">Certificato client</span><span class="sxs-lookup"><span data-stu-id="67073-137">Client Certificate</span></span>
* <span data-ttu-id="67073-138">IdentityServer</span><span class="sxs-lookup"><span data-stu-id="67073-138">IdentityServer</span></span>
* <span data-ttu-id="67073-139">Token JWT</span><span class="sxs-lookup"><span data-stu-id="67073-139">JWT Token</span></span>
* <span data-ttu-id="67073-140">OAuth 2.0</span><span class="sxs-lookup"><span data-stu-id="67073-140">OAuth 2.0</span></span>
* <span data-ttu-id="67073-141">OpenID Connect</span><span class="sxs-lookup"><span data-stu-id="67073-141">OpenID Connect</span></span>
* <span data-ttu-id="67073-142">WS-Federation</span><span class="sxs-lookup"><span data-stu-id="67073-142">WS-Federation</span></span>

<span data-ttu-id="67073-143">Per ulteriori informazioni sulla configurazione dell'autenticazione nel server, vedere [ASP.NET Core Authentication](xref:security/authentication/identity).</span><span class="sxs-lookup"><span data-stu-id="67073-143">For more information on configuring authentication on the server, see [ASP.NET Core authentication](xref:security/authentication/identity).</span></span>

<span data-ttu-id="67073-144">La configurazione del client di gRPC per l'uso dell'autenticazione dipende dal meccanismo di autenticazione usato.</span><span class="sxs-lookup"><span data-stu-id="67073-144">Configuring the gRPC client to use authentication will depend on the authentication mechanism you are using.</span></span> <span data-ttu-id="67073-145">Gli esempi di bearer token e del certificato client precedenti illustrano due modi per configurare il client gRPC per l'invio di metadati di autenticazione con chiamate gRPC:</span><span class="sxs-lookup"><span data-stu-id="67073-145">The previous bearer token and client certificate examples show a couple of ways the gRPC client can be configured to send authentication metadata with gRPC calls:</span></span>

* <span data-ttu-id="67073-146">I client gRPC fortemente tipizzati utilizzano `HttpClient` internamente.</span><span class="sxs-lookup"><span data-stu-id="67073-146">Strongly typed gRPC clients use `HttpClient` internally.</span></span> <span data-ttu-id="67073-147">L'autenticazione può essere configurata in [HttpClientHandler](/dotnet/api/system.net.http.httpclienthandler)o aggiungendo istanze [HttpMessageHandler](/dotnet/api/system.net.http.httpmessagehandler) personalizzate a `HttpClient` .</span><span class="sxs-lookup"><span data-stu-id="67073-147">Authentication can be configured on [HttpClientHandler](/dotnet/api/system.net.http.httpclienthandler), or by adding custom [HttpMessageHandler](/dotnet/api/system.net.http.httpmessagehandler) instances to the `HttpClient`.</span></span>
* <span data-ttu-id="67073-148">Ogni chiamata a gRPC ha un `CallOptions` argomento facoltativo.</span><span class="sxs-lookup"><span data-stu-id="67073-148">Each gRPC call has an optional `CallOptions` argument.</span></span> <span data-ttu-id="67073-149">Le intestazioni personalizzate possono essere inviate tramite la raccolta di intestazioni dell'opzione.</span><span class="sxs-lookup"><span data-stu-id="67073-149">Custom headers can be sent using the option's headers collection.</span></span>

> [!NOTE]
> <span data-ttu-id="67073-150">Non è possibile usare l'autenticazione di Windows (NTLM/Kerberos/Negotiate) con gRPC.</span><span class="sxs-lookup"><span data-stu-id="67073-150">Windows Authentication (NTLM/Kerberos/Negotiate) can't be used with gRPC.</span></span> <span data-ttu-id="67073-151">gRPC richiede HTTP/2 e HTTP/2 non supporta l'autenticazione di Windows.</span><span class="sxs-lookup"><span data-stu-id="67073-151">gRPC requires HTTP/2, and HTTP/2 doesn't support Windows Authentication.</span></span>

## <a name="authorize-users-to-access-services-and-service-methods"></a><span data-ttu-id="67073-152">Autorizzare gli utenti ad accedere ai servizi e ai metodi del servizio</span><span class="sxs-lookup"><span data-stu-id="67073-152">Authorize users to access services and service methods</span></span>

<span data-ttu-id="67073-153">Per impostazione predefinita, tutti i metodi di un servizio possono essere chiamati da utenti non autenticati.</span><span class="sxs-lookup"><span data-stu-id="67073-153">By default, all methods in a service can be called by unauthenticated users.</span></span> <span data-ttu-id="67073-154">Per richiedere l'autenticazione, applicare l' [`[Authorize]`](xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute) attributo al servizio:</span><span class="sxs-lookup"><span data-stu-id="67073-154">To require authentication, apply the [`[Authorize]`](xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute) attribute to the service:</span></span>

```csharp
[Authorize]
public class TicketerService : Ticketer.TicketerBase
{
}
```

<span data-ttu-id="67073-155">È possibile utilizzare gli argomenti del costruttore e le proprietà dell' `[Authorize]` attributo per limitare l'accesso solo agli utenti che corrispondono a [criteri di autorizzazione](xref:security/authorization/policies)specifici.</span><span class="sxs-lookup"><span data-stu-id="67073-155">You can use the constructor arguments and properties of the `[Authorize]` attribute to restrict access to only users matching specific [authorization policies](xref:security/authorization/policies).</span></span> <span data-ttu-id="67073-156">Se, ad esempio, si dispone di un criterio di autorizzazione personalizzato denominato `MyAuthorizationPolicy` , assicurarsi che solo gli utenti che corrispondono a tale criterio possano accedere al servizio usando il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="67073-156">For example, if you have a custom authorization policy called `MyAuthorizationPolicy`, ensure that only users matching that policy can access the service using the following code:</span></span>

```csharp
[Authorize("MyAuthorizationPolicy")]
public class TicketerService : Ticketer.TicketerBase
{
}
```

<span data-ttu-id="67073-157">È possibile applicare anche l'attributo a singoli metodi del servizio `[Authorize]` .</span><span class="sxs-lookup"><span data-stu-id="67073-157">Individual service methods can have the `[Authorize]` attribute applied as well.</span></span> <span data-ttu-id="67073-158">Se l'utente corrente non corrisponde ai criteri applicati **sia** al metodo che alla classe, al chiamante viene restituito un errore:</span><span class="sxs-lookup"><span data-stu-id="67073-158">If the current user doesn't match the policies applied to **both** the method and the class, an error is returned to the caller:</span></span>

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

## <a name="additional-resources"></a><span data-ttu-id="67073-159">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="67073-159">Additional resources</span></span>

* [<span data-ttu-id="67073-160">Autenticazione del token di porta in ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="67073-160">Bearer Token authentication in ASP.NET Core</span></span>](https://blogs.msdn.microsoft.com/webdev/2016/10/27/bearer-token-authentication-in-asp-net-core/)
* [<span data-ttu-id="67073-161">Configurare l'autenticazione del certificato client in ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="67073-161">Configure Client Certificate authentication in ASP.NET Core</span></span>](xref:security/authentication/certauth)
