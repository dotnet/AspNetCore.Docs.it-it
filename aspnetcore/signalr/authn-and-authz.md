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
# <a name="authentication-and-authorization-in-aspnet-core-signalr"></a>Autenticazione e autorizzazione in ASP.NET Core SignalR

Di [Andrew Stanton-Nurse](https://twitter.com/anurse)

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/signalr/authn-and-authz/sample/) [(procedura per il download)](xref:index#how-to-download-a-sample)

## <a name="authenticate-users-connecting-to-a-signalr-hub"></a>Autenticare gli utenti che si connettono a un SignalR Hub

SignalR può essere usato con [l'autenticazione ASP.NET Core](xref:security/authentication/identity) per associare un utente a ogni connessione. In un hub è possibile accedere ai dati di autenticazione dalla proprietà [HubConnectionContext. User](/dotnet/api/microsoft.aspnetcore.signalr.hubconnectioncontext.user) . L'autenticazione consente all'hub di chiamare i metodi su tutte le connessioni associate a un utente. Per ulteriori informazioni, vedere [gestire utenti e gruppi in SignalR ](xref:signalr/groups). Più connessioni possono essere associate a un singolo utente.

Di seguito è riportato un esempio di `Startup.Configure` utilizzo di SignalR e ASP.NET Core l'autenticazione:

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
> L'ordine di registrazione di e del SignalR middleware di autenticazione ASP.NET Core è importante. Chiamare sempre `UseAuthentication` prima di `UseSignalR` in modo che SignalR disponga di un utente in `HttpContext` .

::: moniker-end

### <a name="cookie-authentication"></a>Cookie e

In un'app basata su browser, cookie l'autenticazione consente alle credenziali utente esistenti di eseguire automaticamente il flusso alle SignalR connessioni. Quando si usa il client browser, non è necessaria alcuna configurazione aggiuntiva. Se l'utente è connesso all'app, la SignalR connessione eredita automaticamente l'autenticazione.

Cookiesono un modo specifico del browser per inviare i token di accesso, ma i client non basati su browser possono inviarli. Quando si usa il [client .NET](xref:signalr/dotnet-client), la `Cookies` proprietà può essere configurata nella `.WithUrl` chiamata per fornire un cookie . Tuttavia, l'uso dell' cookie autenticazione dal client .NET richiede che l'app fornisca un'API per lo scambio dei dati di autenticazione per un cookie .

### <a name="bearer-token-authentication"></a>Autenticazione del token di porta

Il client può fornire un token di accesso anziché usare un oggetto cookie . Il server convalida il token e lo usa per identificare l'utente. Questa convalida viene eseguita solo quando viene stabilita la connessione. Durante il ciclo di vita della connessione, il server non viene riconvalidato automaticamente per verificare la revoca dei token.

Nel client JavaScript il token può essere fornito con l'opzione [accessTokenFactory](xref:signalr/configuration#configure-bearer-authentication) .

[!code-typescript[Configure Access Token](authn-and-authz/sample/wwwroot/js/chat.ts?range=52-55)]

Nel client .NET esiste una proprietà [AccessTokenProvider](xref:signalr/configuration#configure-bearer-authentication) simile che può essere usata per configurare il token:

```csharp
var connection = new HubConnectionBuilder()
    .WithUrl("https://example.com/chathub", options =>
    { 
        options.AccessTokenProvider = () => Task.FromResult(_myAccessToken);
    })
    .Build();
```

> [!NOTE]
> La funzione del token di accesso fornita viene chiamata prima di **ogni** richiesta HTTP effettuata da SignalR . Se è necessario rinnovare il token per mantenerla attiva (perché potrebbe scadere durante la connessione), eseguire questa operazione dall'interno di questa funzione e restituire il token aggiornato.

Nelle API Web standard, i token di porta sono inviati in un'intestazione HTTP. Tuttavia, SignalR non è in grado di impostare queste intestazioni nei browser quando si utilizzano alcuni trasporti. Quando si usano WebSocket e Server-Sent eventi, il token viene trasmesso come parametro della stringa di query. 

#### <a name="built-in-jwt-authentication"></a>Autenticazione JWT incorporata

Nel server bearer token autenticazione viene configurata con il [middleware di JWT Bearer](xref:Microsoft.Extensions.DependencyInjection.JwtBearerExtensions.AddJwtBearer%2A):

[!code-csharp[Configure Server to accept access token from Query String](authn-and-authz/sample/Startup.cs?name=snippet)]

[!INCLUDE[request localized comments](~/includes/code-comments-loc.md)]

> [!NOTE]
> La stringa di query viene usata nei browser quando ci si connette con WebSocket e Server-Sent eventi a causa delle limitazioni dell'API del browser. Quando si usa HTTPS, i valori della stringa di query sono protetti dalla connessione TLS. Tuttavia, molti server registrano i valori della stringa di query. Per ulteriori informazioni, vedere [considerazioni sulla sicurezza in SignalR ASP.NET Core ](xref:signalr/security). SignalR Usa le intestazioni per trasmettere i token negli ambienti che li supportano, ad esempio i client .NET e Java.

#### <a name="identity-server-jwt-authentication"></a>Identity Autenticazione server JWT

Quando Identity si usa il server, aggiungere un <xref:Microsoft.Extensions.Options.PostConfigureOptions%601> servizio al progetto:

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

Registrare il servizio in `Startup.ConfigureServices` dopo l'aggiunta di servizi per l'autenticazione ( <xref:Microsoft.Extensions.DependencyInjection.AuthenticationServiceCollectionExtensions.AddAuthentication%2A> ) e il gestore di autenticazione per Identity Server ( <xref:Microsoft.AspNetCore.Authentication.AuthenticationBuilderExtensions.AddIdentityServerJwt%2A> ):

```csharp
services.AddAuthentication()
    .AddIdentityServerJwt();
services.TryAddEnumerable(
    ServiceDescriptor.Singleton<IPostConfigureOptions<JwtBearerOptions>, 
        ConfigureJwtBearerOptions>());
```

### <a name="cookies-vs-bearer-tokens"></a>Cookies rispetto ai token di porta 

Cookiele istanze di sono specifiche per i browser. L'invio da altri tipi di client aggiunge complessità rispetto all'invio dei token di porta. Di conseguenza, l' cookie autenticazione non è consigliata, a meno che l'app non debba solo autenticare gli utenti dal client browser. L'autenticazione del token di porta è l'approccio consigliato quando si usano client diversi dal client browser.

### <a name="windows-authentication"></a>Autenticazione di Windows

Se [l'autenticazione di Windows](xref:security/authentication/windowsauth) è configurata nell'app, SignalR può usare tale identità per proteggere gli hub. Tuttavia, per inviare messaggi a singoli utenti, è necessario aggiungere un provider di ID utente personalizzato. Il sistema di autenticazione di Windows non fornisce l'attestazione "identificatore nome". SignalR Usa l'attestazione per determinare il nome utente.

Aggiungere una nuova classe che implementi `IUserIdProvider` e recuperi una delle attestazioni dall'utente da utilizzare come identificatore. Ad esempio, per usare l'attestazione "Name" (che è il nome utente di Windows nel formato `[Domain]\[Username]` ), creare la classe seguente:

[!code-csharp[Name based provider](authn-and-authz/sample/nameuseridprovider.cs?name=NameUserIdProvider)]

Invece di `ClaimTypes.Name` , è possibile usare qualsiasi valore di `User` (ad esempio, l'identificatore del SID di Windows e così via).

> [!NOTE]
> Il valore scelto deve essere univoco tra tutti gli utenti del sistema. In caso contrario, un messaggio destinato a un utente potrebbe finire con un altro utente.

Registrare il componente nel `Startup.ConfigureServices` metodo.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // ... other services ...

    services.AddSignalR();
    services.AddSingleton<IUserIdProvider, NameUserIdProvider>();
}
```

Nel client .NET è necessario abilitare l'autenticazione di Windows impostando la proprietà [UseDefaultCredentials](/dotnet/api/microsoft.aspnetcore.http.connections.client.httpconnectionoptions.usedefaultcredentials) :

```csharp
var connection = new HubConnectionBuilder()
    .WithUrl("https://example.com/chathub", options =>
    {
        options.UseDefaultCredentials = true;
    })
    .Build();
```

L'autenticazione di Windows è supportata in Internet Explorer e Microsoft Edge, ma non in tutti i browser. Ad esempio, in Chrome e Safari il tentativo di usare l'autenticazione di Windows e i WebSocket non riesce. Quando l'autenticazione di Windows ha esito negativo, il client tenta di eseguire il fallback ad altri trasporti che potrebbero funzionare.

### <a name="use-claims-to-customize-identity-handling"></a>Usare le attestazioni per personalizzare la gestione delle identità

Un'app che autentica gli utenti può derivare gli SignalR ID utente dalle attestazioni utente. Per specificare la modalità di SignalR creazione degli ID utente, implementare `IUserIdProvider` e registrare l'implementazione di.

Il codice di esempio illustra come usare le attestazioni per selezionare l'indirizzo di posta elettronica dell'utente come proprietà di identificazione. 

> [!NOTE]
> Il valore scelto deve essere univoco tra tutti gli utenti del sistema. In caso contrario, un messaggio destinato a un utente potrebbe finire con un altro utente.

[!code-csharp[Email provider](authn-and-authz/sample/EmailBasedUserIdProvider.cs?name=EmailBasedUserIdProvider)]

La registrazione dell'account aggiunge un'attestazione con tipo `ClaimsTypes.Email` al database di identità ASP.NET.

[!code-csharp[Adding the email to the ASP.NET identity claims](authn-and-authz/sample/pages/account/Register.cshtml.cs?name=AddEmailClaim)]

Registrare questo componente in `Startup.ConfigureServices` .

```csharp
services.AddSingleton<IUserIdProvider, EmailBasedUserIdProvider>();
```

## <a name="authorize-users-to-access-hubs-and-hub-methods"></a>Autorizzare gli utenti ad accedere a hub e metodi Hub

Per impostazione predefinita, tutti i metodi in un hub possono essere chiamati da un utente non autenticato. Per richiedere l'autenticazione, applicare l'attributo [autorizza](/dotnet/api/microsoft.aspnetcore.authorization.authorizeattribute) all'hub:

[!code-csharp[Restrict a hub to only authorized users](authn-and-authz/sample/Hubs/ChatHub.cs?range=8-10,32)]

È possibile utilizzare gli argomenti del costruttore e le proprietà dell' `[Authorize]` attributo per limitare l'accesso solo agli utenti che corrispondono a [criteri di autorizzazione](xref:security/authorization/policies)specifici. Ad esempio, se si dispone di un criterio di autorizzazione personalizzato chiamato, `MyAuthorizationPolicy` è possibile assicurarsi che solo gli utenti che corrispondono a tale criterio possano accedere all'hub usando il codice seguente:

```csharp
[Authorize("MyAuthorizationPolicy")]
public class ChatHub : Hub
{
}
```

Per i singoli metodi dell'hub è possibile `[Authorize]` applicare anche l'attributo. Se l'utente corrente non corrisponde ai criteri applicati al metodo, al chiamante viene restituito un errore:

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

### <a name="use-authorization-handlers-to-customize-hub-method-authorization"></a>Usare i gestori di autorizzazione per personalizzare l'autorizzazione del metodo dell'hub

SignalR fornisce una risorsa personalizzata ai gestori di autorizzazione quando un metodo Hub richiede l'autorizzazione. La risorsa è un'istanza di `HubInvocationContext` . `HubInvocationContext`Include `HubCallerContext` , il nome del metodo Hub richiamato e gli argomenti del metodo Hub.

Si consideri l'esempio di una chat room che consente l'accesso a più organizzazioni tramite Azure Active Directory. Chiunque disponga di un account Microsoft può accedere a chat, ma solo i membri dell'organizzazione proprietaria dovrebbero essere in grado di vietare gli utenti o visualizzare le cronologie della chat degli utenti. Inoltre, potrebbe essere necessario limitare determinate funzionalità di determinati utenti. Utilizzando le funzionalità aggiornate di ASP.NET Core 3,0, questa operazione è interamente possibile. Si noti che il `DomainRestrictedRequirement` funge da personalizzato `IAuthorizationRequirement` . Ora che il `HubInvocationContext` parametro Resource viene passato, la logica interna può ispezionare il contesto in cui viene chiamato l'hub e prendere decisioni per consentire all'utente di eseguire singoli metodi dell'hub.

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

In `Startup.ConfigureServices` aggiungere i nuovi criteri, specificando il `DomainRestrictedRequirement` requisito personalizzato come parametro per la creazione del `DomainRestricted` criterio.

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

Nell'esempio precedente, la `DomainRestrictedRequirement` classe è un oggetto `IAuthorizationRequirement` e il relativo `AuthorizationHandler` per quel requisito. È accettabile suddividere questi due componenti in classi separate per separare le problematiche. Un vantaggio dell'approccio dell'esempio è che non è necessario inserire il durante l' `AuthorizationHandler` avvio, perché il requisito e il gestore sono gli stessi.

::: moniker-end

## <a name="additional-resources"></a>Risorse aggiuntive

* [Autenticazione del token di porta in ASP.NET Core](https://blogs.msdn.microsoft.com/webdev/2016/10/27/bearer-token-authentication-in-asp-net-core/)
* [Autorizzazione basata sulle risorse](xref:security/authorization/resourcebased)
