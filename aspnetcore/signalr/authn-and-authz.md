---
title: Autenticazione e autorizzazione in ASP.NET Core SignalR
author: bradygaster
description: Informazioni su come usare l'autenticazione e l'autorizzazione in ASP.NET Core SignalR .
monikerRange: '>= aspnetcore-2.1'
ms.author: bradyg
ms.custom: mvc
ms.date: 12/05/2019
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
uid: signalr/authn-and-authz
ms.openlocfilehash: 9a3102e4451bbc5cd9ff15e88bebd4e4f2c115f4
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102588100"
---
# <a name="authentication-and-authorization-in-aspnet-core-signalr"></a><span data-ttu-id="cf58d-103">Autenticazione e autorizzazione in ASP.NET Core SignalR</span><span class="sxs-lookup"><span data-stu-id="cf58d-103">Authentication and authorization in ASP.NET Core SignalR</span></span>

<span data-ttu-id="cf58d-104">Di [Andrew Stanton-Nurse](https://twitter.com/anurse)</span><span class="sxs-lookup"><span data-stu-id="cf58d-104">By [Andrew Stanton-Nurse](https://twitter.com/anurse)</span></span>

<span data-ttu-id="cf58d-105">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/signalr/authn-and-authz/sample/) [(procedura per il download)](xref:index#how-to-download-a-sample)</span><span class="sxs-lookup"><span data-stu-id="cf58d-105">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/signalr/authn-and-authz/sample/) [(how to download)](xref:index#how-to-download-a-sample)</span></span>

## <a name="authenticate-users-connecting-to-a-signalr-hub"></a><span data-ttu-id="cf58d-106">Autenticare gli utenti che si connettono a un SignalR Hub</span><span class="sxs-lookup"><span data-stu-id="cf58d-106">Authenticate users connecting to a SignalR hub</span></span>

<span data-ttu-id="cf58d-107">SignalR può essere usato con [l'autenticazione ASP.NET Core](xref:security/authentication/identity) per associare un utente a ogni connessione.</span><span class="sxs-lookup"><span data-stu-id="cf58d-107">SignalR can be used with [ASP.NET Core authentication](xref:security/authentication/identity) to associate a user with each connection.</span></span> <span data-ttu-id="cf58d-108">In un hub è possibile accedere ai dati di autenticazione dalla proprietà [HubConnectionContext. User](/dotnet/api/microsoft.aspnetcore.signalr.hubconnectioncontext.user) .</span><span class="sxs-lookup"><span data-stu-id="cf58d-108">In a hub, authentication data can be accessed from the [HubConnectionContext.User](/dotnet/api/microsoft.aspnetcore.signalr.hubconnectioncontext.user) property.</span></span> <span data-ttu-id="cf58d-109">L'autenticazione consente all'hub di chiamare i metodi su tutte le connessioni associate a un utente.</span><span class="sxs-lookup"><span data-stu-id="cf58d-109">Authentication allows the hub to call methods on all connections associated with a user.</span></span> <span data-ttu-id="cf58d-110">Per ulteriori informazioni, vedere [gestire utenti e gruppi in SignalR ](xref:signalr/groups).</span><span class="sxs-lookup"><span data-stu-id="cf58d-110">For more information, see [Manage users and groups in SignalR](xref:signalr/groups).</span></span> <span data-ttu-id="cf58d-111">Più connessioni possono essere associate a un singolo utente.</span><span class="sxs-lookup"><span data-stu-id="cf58d-111">Multiple connections may be associated with a single user.</span></span>

<span data-ttu-id="cf58d-112">Di seguito è riportato un esempio di `Startup.Configure` utilizzo di SignalR e ASP.NET Core l'autenticazione:</span><span class="sxs-lookup"><span data-stu-id="cf58d-112">The following is an example of `Startup.Configure` which uses SignalR and ASP.NET Core authentication:</span></span>

::: moniker range=">= aspnetcore-3.0"

```csharp
public void Configure(IApplicationBuilder app)
{
    ...

    app.UseStaticFiles();

    app.UseRouting();

    app.UseAuthentication();
    app.UseAuthorization();

    app.UseEndpoints(endpoints =>
    {
        endpoints.MapHub<ChatHub>("/chat");
        endpoints.MapControllerRoute("default", "{controller=Home}/{action=Index}/{id?}");
    });
}
```

::: moniker-end

::: moniker range="<= aspnetcore-2.2"

```csharp
public void Configure(IApplicationBuilder app)
{
    ...

    app.UseStaticFiles();

    app.UseAuthentication();

    app.UseSignalR(hubs =>
    {
        hubs.MapHub<ChatHub>("/chat");
    });

    app.UseMvc(routes =>
    {
        routes.MapRoute("default", "{controller=Home}/{action=Index}/{id?}");
    });
}
```

> [!NOTE]
> <span data-ttu-id="cf58d-113">L'ordine di registrazione di e del SignalR middleware di autenticazione ASP.NET Core è importante.</span><span class="sxs-lookup"><span data-stu-id="cf58d-113">The order in which you register the SignalR and ASP.NET Core authentication middleware matters.</span></span> <span data-ttu-id="cf58d-114">Chiamare sempre `UseAuthentication` prima di `UseSignalR` in modo che SignalR disponga di un utente in `HttpContext` .</span><span class="sxs-lookup"><span data-stu-id="cf58d-114">Always call `UseAuthentication` before `UseSignalR` so that SignalR has a user on the `HttpContext`.</span></span>

::: moniker-end

### <a name="cookie-authentication"></a><span data-ttu-id="cf58d-115">Cookie e</span><span class="sxs-lookup"><span data-stu-id="cf58d-115">Cookie authentication</span></span>

<span data-ttu-id="cf58d-116">In un'app basata su browser, cookie l'autenticazione consente alle credenziali utente esistenti di eseguire automaticamente il flusso alle SignalR connessioni.</span><span class="sxs-lookup"><span data-stu-id="cf58d-116">In a browser-based app, cookie authentication allows your existing user credentials to automatically flow to SignalR connections.</span></span> <span data-ttu-id="cf58d-117">Quando si usa il client browser, non è necessaria alcuna configurazione aggiuntiva.</span><span class="sxs-lookup"><span data-stu-id="cf58d-117">When using the browser client, no additional configuration is needed.</span></span> <span data-ttu-id="cf58d-118">Se l'utente è connesso all'app, la SignalR connessione eredita automaticamente l'autenticazione.</span><span class="sxs-lookup"><span data-stu-id="cf58d-118">If the user is logged in to your app, the SignalR connection automatically inherits this authentication.</span></span>

<span data-ttu-id="cf58d-119">Cookiesono un modo specifico del browser per inviare i token di accesso, ma i client non basati su browser possono inviarli.</span><span class="sxs-lookup"><span data-stu-id="cf58d-119">Cookies are a browser-specific way to send access tokens, but non-browser clients can send them.</span></span> <span data-ttu-id="cf58d-120">Quando si usa il [client .NET](xref:signalr/dotnet-client), la `Cookies` proprietà può essere configurata nella `.WithUrl` chiamata per fornire un cookie .</span><span class="sxs-lookup"><span data-stu-id="cf58d-120">When using the [.NET Client](xref:signalr/dotnet-client), the `Cookies` property can be configured in the `.WithUrl` call to provide a cookie.</span></span> <span data-ttu-id="cf58d-121">Tuttavia, l'uso dell' cookie autenticazione dal client .NET richiede che l'app fornisca un'API per lo scambio dei dati di autenticazione per un cookie .</span><span class="sxs-lookup"><span data-stu-id="cf58d-121">However, using cookie authentication from the .NET client requires the app to provide an API to exchange authentication data for a cookie.</span></span>

### <a name="bearer-token-authentication"></a><span data-ttu-id="cf58d-122">Autenticazione del token di porta</span><span class="sxs-lookup"><span data-stu-id="cf58d-122">Bearer token authentication</span></span>

<span data-ttu-id="cf58d-123">Il client può fornire un token di accesso anziché usare un oggetto cookie .</span><span class="sxs-lookup"><span data-stu-id="cf58d-123">The client can provide an access token instead of using a cookie.</span></span> <span data-ttu-id="cf58d-124">Il server convalida il token e lo usa per identificare l'utente.</span><span class="sxs-lookup"><span data-stu-id="cf58d-124">The server validates the token and uses it to identify the user.</span></span> <span data-ttu-id="cf58d-125">Questa convalida viene eseguita solo quando viene stabilita la connessione.</span><span class="sxs-lookup"><span data-stu-id="cf58d-125">This validation is done only when the connection is established.</span></span> <span data-ttu-id="cf58d-126">Durante il ciclo di vita della connessione, il server non viene riconvalidato automaticamente per verificare la revoca dei token.</span><span class="sxs-lookup"><span data-stu-id="cf58d-126">During the life of the connection, the server doesn't automatically revalidate to check for token revocation.</span></span>

<span data-ttu-id="cf58d-127">Nel client JavaScript il token può essere fornito con l'opzione [accessTokenFactory](xref:signalr/configuration#configure-bearer-authentication) .</span><span class="sxs-lookup"><span data-stu-id="cf58d-127">In the JavaScript client, the token can be provided using the [accessTokenFactory](xref:signalr/configuration#configure-bearer-authentication) option.</span></span>

[!code-typescript[Configure Access Token](authn-and-authz/sample/wwwroot/js/chat.ts?range=52-55)]

<span data-ttu-id="cf58d-128">Nel client .NET esiste una proprietà [AccessTokenProvider](xref:signalr/configuration#configure-bearer-authentication) simile che può essere usata per configurare il token:</span><span class="sxs-lookup"><span data-stu-id="cf58d-128">In the .NET client, there's a similar [AccessTokenProvider](xref:signalr/configuration#configure-bearer-authentication) property that can be used to configure the token:</span></span>

```csharp
var connection = new HubConnectionBuilder()
    .WithUrl("https://example.com/chathub", options =>
    { 
        options.AccessTokenProvider = () => Task.FromResult(_myAccessToken);
    })
    .Build();
```

> [!NOTE]
> <span data-ttu-id="cf58d-129">La funzione del token di accesso fornita viene chiamata prima di **ogni** richiesta HTTP effettuata da SignalR .</span><span class="sxs-lookup"><span data-stu-id="cf58d-129">The access token function you provide is called before **every** HTTP request made by SignalR.</span></span> <span data-ttu-id="cf58d-130">Se è necessario rinnovare il token per mantenerla attiva (perché potrebbe scadere durante la connessione), eseguire questa operazione dall'interno di questa funzione e restituire il token aggiornato.</span><span class="sxs-lookup"><span data-stu-id="cf58d-130">If you need to renew the token in order to keep the connection active (because it may expire during the connection), do so from within this function and return the updated token.</span></span>

<span data-ttu-id="cf58d-131">Nelle API Web standard, i token di porta sono inviati in un'intestazione HTTP.</span><span class="sxs-lookup"><span data-stu-id="cf58d-131">In standard web APIs, bearer tokens are sent in an HTTP header.</span></span> <span data-ttu-id="cf58d-132">Tuttavia, SignalR non è in grado di impostare queste intestazioni nei browser quando si utilizzano alcuni trasporti.</span><span class="sxs-lookup"><span data-stu-id="cf58d-132">However, SignalR is unable to set these headers in browsers when using some transports.</span></span> <span data-ttu-id="cf58d-133">Quando si usano WebSocket e Server-Sent eventi, il token viene trasmesso come parametro della stringa di query.</span><span class="sxs-lookup"><span data-stu-id="cf58d-133">When using WebSockets and Server-Sent Events, the token is transmitted as a query string parameter.</span></span> 

#### <a name="built-in-jwt-authentication"></a><span data-ttu-id="cf58d-134">Autenticazione JWT incorporata</span><span class="sxs-lookup"><span data-stu-id="cf58d-134">Built-in JWT authentication</span></span>

<span data-ttu-id="cf58d-135">Nel server bearer token autenticazione viene configurata con il [middleware di JWT Bearer](xref:Microsoft.Extensions.DependencyInjection.JwtBearerExtensions.AddJwtBearer%2A):</span><span class="sxs-lookup"><span data-stu-id="cf58d-135">On the server, bearer token authentication is configured using the [JWT Bearer middleware](xref:Microsoft.Extensions.DependencyInjection.JwtBearerExtensions.AddJwtBearer%2A):</span></span>

[!code-csharp[Configure Server to accept access token from Query String](authn-and-authz/sample/Startup.cs?name=snippet)]

[!INCLUDE[request localized comments](~/includes/code-comments-loc.md)]

> [!NOTE]
> <span data-ttu-id="cf58d-136">La stringa di query viene usata nei browser quando ci si connette con WebSocket e Server-Sent eventi a causa delle limitazioni dell'API del browser.</span><span class="sxs-lookup"><span data-stu-id="cf58d-136">The query string is used on browsers when connecting with WebSockets and Server-Sent Events due to browser API limitations.</span></span> <span data-ttu-id="cf58d-137">Quando si usa HTTPS, i valori della stringa di query sono protetti dalla connessione TLS.</span><span class="sxs-lookup"><span data-stu-id="cf58d-137">When using HTTPS, query string values are secured by the TLS connection.</span></span> <span data-ttu-id="cf58d-138">Tuttavia, molti server registrano i valori della stringa di query.</span><span class="sxs-lookup"><span data-stu-id="cf58d-138">However, many servers log query string values.</span></span> <span data-ttu-id="cf58d-139">Per ulteriori informazioni, vedere [considerazioni sulla sicurezza in SignalR ASP.NET Core ](xref:signalr/security).</span><span class="sxs-lookup"><span data-stu-id="cf58d-139">For more information, see [Security considerations in ASP.NET Core SignalR](xref:signalr/security).</span></span> <span data-ttu-id="cf58d-140">SignalR Usa le intestazioni per trasmettere i token negli ambienti che li supportano, ad esempio i client .NET e Java.</span><span class="sxs-lookup"><span data-stu-id="cf58d-140">SignalR uses headers to transmit tokens in environments which support them (such as the .NET and Java clients).</span></span>

#### <a name="identity-server-jwt-authentication"></a><span data-ttu-id="cf58d-141">Identity Autenticazione server JWT</span><span class="sxs-lookup"><span data-stu-id="cf58d-141">Identity Server JWT authentication</span></span>

<span data-ttu-id="cf58d-142">Quando Identity si usa il server, aggiungere un <xref:Microsoft.Extensions.Options.PostConfigureOptions%601> servizio al progetto:</span><span class="sxs-lookup"><span data-stu-id="cf58d-142">When using Identity Server, add a <xref:Microsoft.Extensions.Options.PostConfigureOptions%601> service to the project:</span></span>

```csharp
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.Extensions.Options;
public class ConfigureJwtBearerOptions : IPostConfigureOptions<JwtBearerOptions>
{
    public void PostConfigure(string name, JwtBearerOptions options)
    {
        var originalOnMessageReceived = options.Events.OnMessageReceived;
        options.Events.OnMessageReceived = async context =>
        {
            await originalOnMessageReceived(context);
                
            if (string.IsNullOrEmpty(context.Token))
            {
                var accessToken = context.Request.Query["access_token"];
                var path = context.HttpContext.Request.Path;
                
                if (!string.IsNullOrEmpty(accessToken) && 
                    path.StartsWithSegments("/hubs"))
                {
                    context.Token = accessToken;
                }
            }
        };
    }
}
```

<span data-ttu-id="cf58d-143">Registrare il servizio in `Startup.ConfigureServices` dopo l'aggiunta di servizi per l'autenticazione ( <xref:Microsoft.Extensions.DependencyInjection.AuthenticationServiceCollectionExtensions.AddAuthentication%2A> ) e il gestore di autenticazione per Identity Server ( <xref:Microsoft.AspNetCore.Authentication.AuthenticationBuilderExtensions.AddIdentityServerJwt%2A> ):</span><span class="sxs-lookup"><span data-stu-id="cf58d-143">Register the service in `Startup.ConfigureServices` after adding services for authentication (<xref:Microsoft.Extensions.DependencyInjection.AuthenticationServiceCollectionExtensions.AddAuthentication%2A>) and the authentication handler for Identity Server (<xref:Microsoft.AspNetCore.Authentication.AuthenticationBuilderExtensions.AddIdentityServerJwt%2A>):</span></span>

```csharp
services.AddAuthentication()
    .AddIdentityServerJwt();
services.TryAddEnumerable(
    ServiceDescriptor.Singleton<IPostConfigureOptions<JwtBearerOptions>, 
        ConfigureJwtBearerOptions>());
```

### <a name="cookies-vs-bearer-tokens"></a><span data-ttu-id="cf58d-144">Cookies rispetto ai token di porta</span><span class="sxs-lookup"><span data-stu-id="cf58d-144">Cookies vs. bearer tokens</span></span> 

<span data-ttu-id="cf58d-145">Cookiele istanze di sono specifiche per i browser.</span><span class="sxs-lookup"><span data-stu-id="cf58d-145">Cookies are specific to browsers.</span></span> <span data-ttu-id="cf58d-146">L'invio da altri tipi di client aggiunge complessità rispetto all'invio dei token di porta.</span><span class="sxs-lookup"><span data-stu-id="cf58d-146">Sending them from other kinds of clients adds complexity compared to sending bearer tokens.</span></span> <span data-ttu-id="cf58d-147">Di conseguenza, l' cookie autenticazione non è consigliata, a meno che l'app non debba solo autenticare gli utenti dal client browser.</span><span class="sxs-lookup"><span data-stu-id="cf58d-147">Consequently, cookie authentication isn't recommended unless the app only needs to authenticate users from the browser client.</span></span> <span data-ttu-id="cf58d-148">L'autenticazione del token di porta è l'approccio consigliato quando si usano client diversi dal client browser.</span><span class="sxs-lookup"><span data-stu-id="cf58d-148">Bearer token authentication is the recommended approach when using clients other than the browser client.</span></span>

### <a name="windows-authentication"></a><span data-ttu-id="cf58d-149">Autenticazione di Windows</span><span class="sxs-lookup"><span data-stu-id="cf58d-149">Windows authentication</span></span>

<span data-ttu-id="cf58d-150">Se [l'autenticazione di Windows](xref:security/authentication/windowsauth) è configurata nell'app, SignalR può usare tale identità per proteggere gli hub.</span><span class="sxs-lookup"><span data-stu-id="cf58d-150">If [Windows authentication](xref:security/authentication/windowsauth) is configured in your app, SignalR can use that identity to secure hubs.</span></span> <span data-ttu-id="cf58d-151">Tuttavia, per inviare messaggi a singoli utenti, è necessario aggiungere un provider di ID utente personalizzato.</span><span class="sxs-lookup"><span data-stu-id="cf58d-151">However, to send messages to individual users, you need to add a custom User ID provider.</span></span> <span data-ttu-id="cf58d-152">Il sistema di autenticazione di Windows non fornisce l'attestazione "identificatore nome".</span><span class="sxs-lookup"><span data-stu-id="cf58d-152">The Windows authentication system doesn't provide the "Name Identifier" claim.</span></span> <span data-ttu-id="cf58d-153">SignalR Usa l'attestazione per determinare il nome utente.</span><span class="sxs-lookup"><span data-stu-id="cf58d-153">SignalR uses the claim to determine the user name.</span></span>

<span data-ttu-id="cf58d-154">Aggiungere una nuova classe che implementi `IUserIdProvider` e recuperi una delle attestazioni dall'utente da utilizzare come identificatore.</span><span class="sxs-lookup"><span data-stu-id="cf58d-154">Add a new class that implements `IUserIdProvider` and retrieve one of the claims from the user to use as the identifier.</span></span> <span data-ttu-id="cf58d-155">Ad esempio, per usare l'attestazione "Name" (che è il nome utente di Windows nel formato `[Domain]\[Username]` ), creare la classe seguente:</span><span class="sxs-lookup"><span data-stu-id="cf58d-155">For example, to use the "Name" claim (which is the Windows username in the form `[Domain]\[Username]`), create the following class:</span></span>

[!code-csharp[Name based provider](authn-and-authz/sample/nameuseridprovider.cs?name=NameUserIdProvider)]

<span data-ttu-id="cf58d-156">Invece di `ClaimTypes.Name` , è possibile usare qualsiasi valore di `User` (ad esempio, l'identificatore del SID di Windows e così via).</span><span class="sxs-lookup"><span data-stu-id="cf58d-156">Rather than `ClaimTypes.Name`, you can use any value from the `User` (such as the Windows SID identifier, and so on).</span></span>

> [!NOTE]
> <span data-ttu-id="cf58d-157">Il valore scelto deve essere univoco tra tutti gli utenti del sistema.</span><span class="sxs-lookup"><span data-stu-id="cf58d-157">The value you choose must be unique among all the users in your system.</span></span> <span data-ttu-id="cf58d-158">In caso contrario, un messaggio destinato a un utente potrebbe finire con un altro utente.</span><span class="sxs-lookup"><span data-stu-id="cf58d-158">Otherwise, a message intended for one user could end up going to a different user.</span></span>

<span data-ttu-id="cf58d-159">Registrare il componente nel `Startup.ConfigureServices` metodo.</span><span class="sxs-lookup"><span data-stu-id="cf58d-159">Register this component in your `Startup.ConfigureServices` method.</span></span>

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // ... other services ...

    services.AddSignalR();
    services.AddSingleton<IUserIdProvider, NameUserIdProvider>();
}
```

<span data-ttu-id="cf58d-160">Nel client .NET è necessario abilitare l'autenticazione di Windows impostando la proprietà [UseDefaultCredentials](/dotnet/api/microsoft.aspnetcore.http.connections.client.httpconnectionoptions.usedefaultcredentials) :</span><span class="sxs-lookup"><span data-stu-id="cf58d-160">In the .NET Client, Windows Authentication must be enabled by setting the [UseDefaultCredentials](/dotnet/api/microsoft.aspnetcore.http.connections.client.httpconnectionoptions.usedefaultcredentials) property:</span></span>

```csharp
var connection = new HubConnectionBuilder()
    .WithUrl("https://example.com/chathub", options =>
    {
        options.UseDefaultCredentials = true;
    })
    .Build();
```

<span data-ttu-id="cf58d-161">L'autenticazione di Windows è supportata in Internet Explorer e Microsoft Edge, ma non in tutti i browser.</span><span class="sxs-lookup"><span data-stu-id="cf58d-161">Windows authentication is supported in Internet Explorer and Microsoft Edge, but not in all browsers.</span></span> <span data-ttu-id="cf58d-162">Ad esempio, in Chrome e Safari il tentativo di usare l'autenticazione di Windows e i WebSocket non riesce.</span><span class="sxs-lookup"><span data-stu-id="cf58d-162">For example, in Chrome and Safari, attempting to use Windows authentication and WebSockets fails.</span></span> <span data-ttu-id="cf58d-163">Quando l'autenticazione di Windows ha esito negativo, il client tenta di eseguire il fallback ad altri trasporti che potrebbero funzionare.</span><span class="sxs-lookup"><span data-stu-id="cf58d-163">When Windows authentication fails, the client attempts to fall back to other transports which might work.</span></span>

### <a name="use-claims-to-customize-identity-handling"></a><span data-ttu-id="cf58d-164">Usare le attestazioni per personalizzare la gestione delle identità</span><span class="sxs-lookup"><span data-stu-id="cf58d-164">Use claims to customize identity handling</span></span>

<span data-ttu-id="cf58d-165">Un'app che autentica gli utenti può derivare gli SignalR ID utente dalle attestazioni utente.</span><span class="sxs-lookup"><span data-stu-id="cf58d-165">An app that authenticates users can derive SignalR user IDs from user claims.</span></span> <span data-ttu-id="cf58d-166">Per specificare la modalità di SignalR creazione degli ID utente, implementare `IUserIdProvider` e registrare l'implementazione di.</span><span class="sxs-lookup"><span data-stu-id="cf58d-166">To specify how SignalR creates user IDs, implement `IUserIdProvider` and register the implementation.</span></span>

<span data-ttu-id="cf58d-167">Il codice di esempio illustra come usare le attestazioni per selezionare l'indirizzo di posta elettronica dell'utente come proprietà di identificazione.</span><span class="sxs-lookup"><span data-stu-id="cf58d-167">The sample code demonstrates how you would use claims to select the user's email address as the identifying property.</span></span> 

> [!NOTE]
> <span data-ttu-id="cf58d-168">Il valore scelto deve essere univoco tra tutti gli utenti del sistema.</span><span class="sxs-lookup"><span data-stu-id="cf58d-168">The value you choose must be unique among all the users in your system.</span></span> <span data-ttu-id="cf58d-169">In caso contrario, un messaggio destinato a un utente potrebbe finire con un altro utente.</span><span class="sxs-lookup"><span data-stu-id="cf58d-169">Otherwise, a message intended for one user could end up going to a different user.</span></span>

[!code-csharp[Email provider](authn-and-authz/sample/EmailBasedUserIdProvider.cs?name=EmailBasedUserIdProvider)]

<span data-ttu-id="cf58d-170">La registrazione dell'account aggiunge un'attestazione con tipo `ClaimsTypes.Email` al database di identità ASP.NET.</span><span class="sxs-lookup"><span data-stu-id="cf58d-170">The account registration adds a claim with type `ClaimsTypes.Email` to the ASP.NET identity database.</span></span>

[!code-csharp[Adding the email to the ASP.NET identity claims](authn-and-authz/sample/pages/account/Register.cshtml.cs?name=AddEmailClaim)]

<span data-ttu-id="cf58d-171">Registrare questo componente in `Startup.ConfigureServices` .</span><span class="sxs-lookup"><span data-stu-id="cf58d-171">Register this component in your `Startup.ConfigureServices`.</span></span>

```csharp
services.AddSingleton<IUserIdProvider, EmailBasedUserIdProvider>();
```

## <a name="authorize-users-to-access-hubs-and-hub-methods"></a><span data-ttu-id="cf58d-172">Autorizzare gli utenti ad accedere a hub e metodi Hub</span><span class="sxs-lookup"><span data-stu-id="cf58d-172">Authorize users to access hubs and hub methods</span></span>

<span data-ttu-id="cf58d-173">Per impostazione predefinita, tutti i metodi in un hub possono essere chiamati da un utente non autenticato.</span><span class="sxs-lookup"><span data-stu-id="cf58d-173">By default, all methods in a hub can be called by an unauthenticated user.</span></span> <span data-ttu-id="cf58d-174">Per richiedere l'autenticazione, applicare l'attributo [autorizza](/dotnet/api/microsoft.aspnetcore.authorization.authorizeattribute) all'hub:</span><span class="sxs-lookup"><span data-stu-id="cf58d-174">To require authentication, apply the [Authorize](/dotnet/api/microsoft.aspnetcore.authorization.authorizeattribute) attribute to the hub:</span></span>

[!code-csharp[Restrict a hub to only authorized users](authn-and-authz/sample/Hubs/ChatHub.cs?range=8-10,32)]

<span data-ttu-id="cf58d-175">È possibile utilizzare gli argomenti del costruttore e le proprietà dell' `[Authorize]` attributo per limitare l'accesso solo agli utenti che corrispondono a [criteri di autorizzazione](xref:security/authorization/policies)specifici.</span><span class="sxs-lookup"><span data-stu-id="cf58d-175">You can use the constructor arguments and properties of the `[Authorize]` attribute to restrict access to only users matching specific [authorization policies](xref:security/authorization/policies).</span></span> <span data-ttu-id="cf58d-176">Ad esempio, se si dispone di un criterio di autorizzazione personalizzato chiamato, `MyAuthorizationPolicy` è possibile assicurarsi che solo gli utenti che corrispondono a tale criterio possano accedere all'hub usando il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="cf58d-176">For example, if you have a custom authorization policy called `MyAuthorizationPolicy` you can ensure that only users matching that policy can access the hub using the following code:</span></span>

```csharp
[Authorize("MyAuthorizationPolicy")]
public class ChatHub : Hub
{
}
```

<span data-ttu-id="cf58d-177">Per i singoli metodi dell'hub è possibile `[Authorize]` applicare anche l'attributo.</span><span class="sxs-lookup"><span data-stu-id="cf58d-177">Individual hub methods can have the `[Authorize]` attribute applied as well.</span></span> <span data-ttu-id="cf58d-178">Se l'utente corrente non corrisponde ai criteri applicati al metodo, al chiamante viene restituito un errore:</span><span class="sxs-lookup"><span data-stu-id="cf58d-178">If the current user doesn't match the policy applied to the method, an error is returned to the caller:</span></span>

```csharp
[Authorize]
public class ChatHub : Hub
{
    public async Task Send(string message)
    {
        // ... send a message to all users ...
    }

    [Authorize("Administrators")]
    public void BanUser(string userName)
    {
        // ... ban a user from the chat room (something only Administrators can do) ...
    }
}
```

::: moniker range=">= aspnetcore-3.0"

### <a name="use-authorization-handlers-to-customize-hub-method-authorization"></a><span data-ttu-id="cf58d-179">Usare i gestori di autorizzazione per personalizzare l'autorizzazione del metodo dell'hub</span><span class="sxs-lookup"><span data-stu-id="cf58d-179">Use authorization handlers to customize hub method authorization</span></span>

<span data-ttu-id="cf58d-180">SignalR fornisce una risorsa personalizzata ai gestori di autorizzazione quando un metodo Hub richiede l'autorizzazione.</span><span class="sxs-lookup"><span data-stu-id="cf58d-180">SignalR provides a custom resource to authorization handlers when a hub method requires authorization.</span></span> <span data-ttu-id="cf58d-181">La risorsa è un'istanza di `HubInvocationContext` .</span><span class="sxs-lookup"><span data-stu-id="cf58d-181">The resource is an instance of `HubInvocationContext`.</span></span> <span data-ttu-id="cf58d-182">`HubInvocationContext`Include `HubCallerContext` , il nome del metodo Hub richiamato e gli argomenti del metodo Hub.</span><span class="sxs-lookup"><span data-stu-id="cf58d-182">The `HubInvocationContext` includes the `HubCallerContext`, the name of the hub method being invoked, and the arguments to the hub method.</span></span>

<span data-ttu-id="cf58d-183">Si consideri l'esempio di una chat room che consente l'accesso a più organizzazioni tramite Azure Active Directory.</span><span class="sxs-lookup"><span data-stu-id="cf58d-183">Consider the example of a chat room allowing multiple organization sign-in via Azure Active Directory.</span></span> <span data-ttu-id="cf58d-184">Chiunque disponga di un account Microsoft può accedere a chat, ma solo i membri dell'organizzazione proprietaria dovrebbero essere in grado di vietare gli utenti o visualizzare le cronologie della chat degli utenti.</span><span class="sxs-lookup"><span data-stu-id="cf58d-184">Anyone with a Microsoft account can sign in to chat, but only members of the owning organization should be able to ban users or view users' chat histories.</span></span> <span data-ttu-id="cf58d-185">Inoltre, potrebbe essere necessario limitare determinate funzionalità di determinati utenti.</span><span class="sxs-lookup"><span data-stu-id="cf58d-185">Furthermore, we might want to restrict certain functionality from certain users.</span></span> <span data-ttu-id="cf58d-186">Utilizzando le funzionalità aggiornate di ASP.NET Core 3,0, questa operazione è interamente possibile.</span><span class="sxs-lookup"><span data-stu-id="cf58d-186">Using the updated features in ASP.NET Core 3.0, this is entirely possible.</span></span> <span data-ttu-id="cf58d-187">Si noti che il `DomainRestrictedRequirement` funge da personalizzato `IAuthorizationRequirement` .</span><span class="sxs-lookup"><span data-stu-id="cf58d-187">Note how the `DomainRestrictedRequirement` serves as a custom `IAuthorizationRequirement`.</span></span> <span data-ttu-id="cf58d-188">Ora che il `HubInvocationContext` parametro Resource viene passato, la logica interna può ispezionare il contesto in cui viene chiamato l'hub e prendere decisioni per consentire all'utente di eseguire singoli metodi dell'hub.</span><span class="sxs-lookup"><span data-stu-id="cf58d-188">Now that the `HubInvocationContext` resource parameter is being passed in, the internal logic can inspect the context in which the Hub is being called and make decisions on allowing the user to execute individual Hub methods.</span></span>

```csharp
[Authorize]
public class ChatHub : Hub
{
    public void SendMessage(string message)
    {
    }

    [Authorize("DomainRestricted")]
    public void BanUser(string username)
    {
    }

    [Authorize("DomainRestricted")]
    public void ViewUserHistory(string username)
    {
    }
}

public class DomainRestrictedRequirement : 
    AuthorizationHandler<DomainRestrictedRequirement, HubInvocationContext>, 
    IAuthorizationRequirement
{
    protected override Task HandleRequirementAsync(AuthorizationHandlerContext context,
        DomainRestrictedRequirement requirement, 
        HubInvocationContext resource)
    {
        if (IsUserAllowedToDoThis(resource.HubMethodName, context.User.Identity.Name) && 
            context.User.Identity.Name.EndsWith("@microsoft.com"))
        {
            context.Succeed(requirement);
        }
        return Task.CompletedTask;
    }

    private bool IsUserAllowedToDoThis(string hubMethodName,
        string currentUsername)
    {
        return !(currentUsername.Equals("asdf42@microsoft.com") && 
            hubMethodName.Equals("banUser", StringComparison.OrdinalIgnoreCase));
    }
}
```

<span data-ttu-id="cf58d-189">In `Startup.ConfigureServices` aggiungere i nuovi criteri, specificando il `DomainRestrictedRequirement` requisito personalizzato come parametro per la creazione del `DomainRestricted` criterio.</span><span class="sxs-lookup"><span data-stu-id="cf58d-189">In `Startup.ConfigureServices`, add the new policy, providing the custom `DomainRestrictedRequirement` requirement as a parameter to create the `DomainRestricted` policy.</span></span>

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // ... other services ...

    services
        .AddAuthorization(options =>
        {
            options.AddPolicy("DomainRestricted", policy =>
            {
                policy.Requirements.Add(new DomainRestrictedRequirement());
            });
        });
}
```

<span data-ttu-id="cf58d-190">Nell'esempio precedente, la `DomainRestrictedRequirement` classe è un oggetto `IAuthorizationRequirement` e il relativo `AuthorizationHandler` per quel requisito.</span><span class="sxs-lookup"><span data-stu-id="cf58d-190">In the preceding example, the `DomainRestrictedRequirement` class is both an `IAuthorizationRequirement` and its own `AuthorizationHandler` for that requirement.</span></span> <span data-ttu-id="cf58d-191">È accettabile suddividere questi due componenti in classi separate per separare le problematiche.</span><span class="sxs-lookup"><span data-stu-id="cf58d-191">It's acceptable to split these two components into separate classes to separate concerns.</span></span> <span data-ttu-id="cf58d-192">Un vantaggio dell'approccio dell'esempio è che non è necessario inserire il durante l' `AuthorizationHandler` avvio, perché il requisito e il gestore sono gli stessi.</span><span class="sxs-lookup"><span data-stu-id="cf58d-192">A benefit of the example's approach is there's no need to inject the `AuthorizationHandler` during startup, as the requirement and the handler are the same thing.</span></span>

::: moniker-end

## <a name="additional-resources"></a><span data-ttu-id="cf58d-193">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="cf58d-193">Additional resources</span></span>

* [<span data-ttu-id="cf58d-194">Autenticazione del token di porta in ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="cf58d-194">Bearer Token Authentication in ASP.NET Core</span></span>](https://blogs.msdn.microsoft.com/webdev/2016/10/27/bearer-token-authentication-in-asp-net-core/)
* [<span data-ttu-id="cf58d-195">Autorizzazione basata sulle risorse</span><span class="sxs-lookup"><span data-stu-id="cf58d-195">Resource-based Authorization</span></span>](xref:security/authorization/resourcebased)
