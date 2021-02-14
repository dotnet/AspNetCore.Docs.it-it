---
title: Usare API Graph con ASP.NET Core Blazor WebAssembly
author: guardrex
description: Informazioni su come usare API Graph con le Blazor app WebAssemlby.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 10/27/2020
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
uid: blazor/security/webassembly/graph-api
ms.openlocfilehash: 997d4dec05ddb6b9d0acb5ed36a6510c0836a4fb
ms.sourcegitcommit: 1166b0ff3828418559510c661e8240e5c5717bb7
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/12/2021
ms.locfileid: "100280502"
---
# <a name="use-graph-api-with-aspnet-core-blazor-webassembly"></a>Usare API Graph con ASP.NET Core Blazor WebAssembly

::: moniker range=">= aspnetcore-5.0"

[Microsoft Graph API](/graph/use-the-api) è un'API Web RESTful che consente Blazor ad altre app .NET Framework di accedere alle risorse del servizio Microsoft Cloud.

## <a name="graph-sdk"></a>SDK per Graph

[Microsoft Graph SDK](/graph/sdks/sdks-overview) sono progettati per semplificare la creazione di applicazioni di alta qualità, efficienti e resilienti che accedono a Microsoft Graph.

Gli esempi in questa sezione richiedono riferimenti ai pacchetti per i pacchetti seguenti nel file di progetto del file di progetto dell'app o autonomo *`Client`* :

* [`Microsoft.Extensions.Http`](https://www.nuget.org/packages/Microsoft.Extensions.Http)
* [`Microsoft.Graph`](https://www.nuget.org/packages/Microsoft.Graph)

In ognuna delle sottosezioni seguenti di questo articolo vengono usate le classi e la configurazione di utilità seguenti:

* [Chiamare API Graph da un componente usando Graph SDK](#call-graph-api-from-a-component-using-the-graph-sdk)
* [Personalizzare le attestazioni utente con Graph SDK](#customize-user-claims-with-the-graph-sdk)

Dopo l'aggiunta degli ambiti dell'API Microsoft Graph nell'area AAD della portale di Azure:

* Aggiungere la `GraphClientExtensions.cs` classe seguente all'app autonoma o all' *`Client`* app di una Blazor soluzione ospitata.
* Specificare gli ambiti necessari per la <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AccessTokenRequestOptions.Scopes> proprietà dell'oggetto <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AccessTokenRequestOptions> nel `AuthenticateRequestAsync` metodo. Nell'esempio seguente `User.Read` viene specificato l'ambito corrispondente agli esempi nelle sezioni successive di questo articolo.

```csharp
using System;
using System.Net.Http;
using System.Net.Http.Headers;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Components.WebAssembly.Authentication;
using Microsoft.Authentication.WebAssembly.Msal.Models;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Graph;

internal static class GraphClientExtensions
{
    public static IServiceCollection AddGraphClient(
        this IServiceCollection services, params string[] scopes)
    {
        services.Configure<RemoteAuthenticationOptions<MsalProviderOptions>>(
            options =>
            {
                foreach (var scope in scopes)
                {
                    options.ProviderOptions.AdditionalScopesToConsent.Add(scope);
                }
            });

        services.AddScoped<IAuthenticationProvider, 
            NoOpGraphAuthenticationProvider>();
        services.AddScoped<IHttpProvider, HttpClientHttpProvider>(sp => 
            new HttpClientHttpProvider(new HttpClient()));
        services.AddScoped(sp =>
        {
            return new GraphServiceClient(
                sp.GetRequiredService<IAuthenticationProvider>(),
                sp.GetRequiredService<IHttpProvider>());
        });

        return services;
    }

    private class NoOpGraphAuthenticationProvider : IAuthenticationProvider
    {
        public NoOpGraphAuthenticationProvider(IAccessTokenProvider tokenProvider)
        {
            TokenProvider = tokenProvider;
        }

        public IAccessTokenProvider TokenProvider { get; }

        public async Task AuthenticateRequestAsync(HttpRequestMessage request)
        {
            var result = await TokenProvider.RequestAccessToken(
                new AccessTokenRequestOptions()
                {
                    Scopes = new[] { "{SCOPE 1}", "{SCOPE 2}", ... "{SCOPE X}" }
                });

            if (result.TryGetToken(out var token))
            {
                request.Headers.Authorization ??= new AuthenticationHeaderValue(
                    "Bearer", token.Value);
            }
        }
    }

    private class HttpClientHttpProvider : IHttpProvider
    {
        private readonly HttpClient http;

        public HttpClientHttpProvider(HttpClient http)
        {
            this.http = http;
        }

        public ISerializer Serializer { get; } = new Serializer();

        public TimeSpan OverallTimeout { get; set; } = TimeSpan.FromSeconds(300);

        public void Dispose()
        {
        }

        public Task<HttpResponseMessage> SendAsync(HttpRequestMessage request)
        {
            return http.SendAsync(request);
        }

        public Task<HttpResponseMessage> SendAsync(HttpRequestMessage request, 
            HttpCompletionOption completionOption, 
            CancellationToken cancellationToken)
        {
            return http.SendAsync(request, completionOption, cancellationToken);
        }
    }
}
```

I segnaposto `"{SCOPE 1}", "{SCOPE 2}", ... "{SCOPE X}"` di ambito nel codice precedente rappresentano uno o più ambiti consentiti. Ad esempio, impostare `Scopes` su una matrice di stringhe di un ambito per `User.Read` per gli esempi nelle sezioni seguenti di questo articolo:

```csharp
Scopes = new[] { "https://graph.microsoft.com/User.Read" }
```

In `Program.Main` ( `Program.cs` ) aggiungere i servizi client Graph e la configurazione con il `AddGraphClient` metodo di estensione:

```csharp
builder.Services.AddGraphClient("{SCOPE 1}", "{SCOPE 2}", ... "{SCOPE X}");
```

I segnaposto `"{SCOPE 1}", "{SCOPE 2}", ... "{SCOPE X}"` di ambito nel codice precedente rappresentano uno o più ambiti consentiti. Ad esempio, passare l' `User.Read` ambito a `AddGraphClient` per gli esempi nelle sezioni seguenti di questo articolo:

```csharp
builder.Services.AddGraphClient("https://graph.microsoft.com/User.Read");
```

### <a name="call-graph-api-from-a-component-using-the-graph-sdk"></a>Chiamare API Graph da un componente usando Graph SDK

Questa sezione usa le [classi di utilità ( `GraphClientExtensions.cs` )](#graph-sdk) descritte in precedenza in questo articolo. Il `GraphExample` componente seguente usa un oggetto inserito `GraphServiceClient` per ottenere i dati del profilo AAD dell'utente e visualizzare il numero di telefono cellulare:

```razor
@page "/GraphExample"
@using Microsoft.AspNetCore.Authorization
@using Microsoft.Graph
@attribute [Authorize]
@inject GraphServiceClient GraphClient

<h3>Graph Client Example</h3>

@if (user != null)
{
    <p>Mobile Phone: @user.MobilePhone</p>
}

@code {
    private User user;

    protected async override Task OnInitializedAsync()
    {
        var request = GraphClient.Me.Request();
        user = await request.GetAsync();
    }
}
```

### <a name="customize-user-claims-with-the-graph-sdk"></a>Personalizzare le attestazioni utente con Graph SDK

Questa sezione usa le [classi di utilità ( `GraphClientExtensions.cs` )](#graph-sdk) descritte in precedenza in questo articolo.

Nell'esempio seguente l'app crea un'attestazione di numero di telefono cellulare per un utente dal numero di telefono cellulare del profilo utente di AAD. L'app deve avere l' `User.Read` ambito di API Graph configurato in AAD.

Nella Factory degli account utente personalizzata seguente il Framework <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteUserAccount> rappresenta l'account dell'utente. Se l'app richiede una classe di account utente personalizzata che estende <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteUserAccount> , scambiare la classe di account utente personalizzata per <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteUserAccount> nel codice riportato di seguito.

`CustomAccountFactory.cs`:

```csharp
using System;
using System.Net.Http;
using System.Net.Http.Json;
using System.Security.Claims;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Components.WebAssembly.Authentication;
using Microsoft.AspNetCore.Components.WebAssembly.Authentication.Internal;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Logging;
using Microsoft.Graph;

public class CustomAccountFactory
    : AccountClaimsPrincipalFactory<RemoteUserAccount>
{
    private readonly ILogger<CustomAccountFactory> logger;
    private readonly IServiceProvider serviceProvider;

    public CustomAccountFactory(IAccessTokenProviderAccessor accessor, 
        IServiceProvider serviceProvider,
        ILogger<CustomAccountFactory> logger)
        : base(accessor)
    {
        this.serviceProvider = serviceProvider;
        this.logger = logger;
    }

    public async override ValueTask<ClaimsPrincipal> CreateUserAsync(
        RemoteUserAccount account,
        RemoteAuthenticationUserOptions options)
    {
        var initialUser = await base.CreateUserAsync(account, options);

        if (initialUser.Identity.IsAuthenticated)
        {
            var userIdentity = (ClaimsIdentity)initialUser.Identity;

            try
            {
                var graphClient = ActivatorUtilities
                    .CreateInstance<GraphServiceClient>(serviceProvider);
                var request = graphClient.Me.Request();
                var user = await request.GetAsync();

                if (user != null)
                {
                    userIdentity.AddClaim(new Claim("mobilephone", 
                        user.MobilePhone));
                }
            }
            catch (ServiceException exception)
            {
                logger.LogError("Graph API service failure: {Message}",
                    exception.Message);
            }
        }

        return initialUser;
    }
}
```

In `Program.Main` ( `Program.cs` ) configurare l'autenticazione MSAL per l'uso della factory dell'account utente personalizzata: se l'app usa una classe di account utente personalizzata che estende <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteUserAccount> , scambiare la classe di account utente personalizzata per <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteUserAccount> nel codice seguente:

```csharp
using Microsoft.AspNetCore.Components.WebAssembly.Authentication;
using Microsoft.Extensions.Configuration;

builder.Services.AddMsalAuthentication<RemoteAuthenticationState, 
    RemoteUserAccount>(options =>
    {
        builder.Configuration.Bind("AzureAd", 
            options.ProviderOptions.Authentication);
        options.ProviderOptions.DefaultAccessTokenScopes.Add(
            "https://graph.microsoft.com/User.Read");
    })
    .AddAccountClaimsPrincipalFactory<RemoteAuthenticationState, RemoteUserAccount, 
        CustomAccountFactory>();
```

::: moniker-end

## <a name="named-client-with-graph-api"></a>Client denominato con API Graph

Negli esempi di questa sezione viene usato un denominato <xref:System.Net.Http.HttpClient> per API Graph per ottenere il numero di telefono cellulare di un utente per elaborare una chiamata.

Gli esempi in questa sezione richiedono un riferimento al pacchetto per [`Microsoft.Extensions.Http`](https://www.nuget.org/packages/Microsoft.Extensions.Http) nel file di progetto del file di progetto dell'app autonomo o *`Client`* .

Creare la classe e la configurazione del progetto seguenti per lavorare con API Graph. La classe e la configurazione seguenti vengono usate in ognuna delle sottosezioni seguenti di questo articolo:

* [Chiamare API Graph da un componente](#call-graph-api-from-a-component)
* [Personalizzare le attestazioni utente con API Graph e un client denominato](#customize-user-claims-with-graph-api-and-a-named-client)

Dopo l'aggiunta degli ambiti dell'API Microsoft Graph nell'area AAD della portale di Azure, fornire gli ambiti necessari al gestore configurato dell'app per API Graph. Nell'esempio seguente viene configurato il gestore per l' `User.Read` ambito. È possibile aggiungere ambiti aggiuntivi.

`GraphAuthorizationMessageHandler.cs`:

```csharp
using Microsoft.AspNetCore.Components;
using Microsoft.AspNetCore.Components.WebAssembly.Authentication;

public class GraphAPIAuthorizationMessageHandler : AuthorizationMessageHandler
{
    public GraphAPIAuthorizationMessageHandler(IAccessTokenProvider provider,
        NavigationManager navigationManager)
        : base(provider, navigationManager)
    {
        ConfigureHandler(
            authorizedUrls: new[] { "https://graph.microsoft.com" },
            scopes: new[] { "https://graph.microsoft.com/User.Read" });
    }
}
```

In `Program.Main` ( `Program.cs` ) configurare il nome <xref:System.Net.Http.HttpClient> per API Graph:

```csharp
builder.Services.AddScoped<GraphAPIAuthorizationMessageHandler>();

builder.Services.AddHttpClient("GraphAPI",
        client => client.BaseAddress = new Uri("https://graph.microsoft.com"))
    .AddHttpMessageHandler<GraphAPIAuthorizationMessageHandler>();
```

### <a name="call-graph-api-from-a-component"></a>Chiamare API Graph da un componente

Questa sezione usa il [gestore dei messaggi di autorizzazione Graph ( `GraphAuthorizationMessageHandler.cs` ) e `Program.Main` le aggiunte all'app](#named-client-with-graph-api) descritta in precedenza in questo articolo, che fornisce un oggetto denominato <xref:System.Net.Http.HttpClient> per API Graph.

In un Razor componente:

* Creare una <xref:System.Net.Http.HttpClient> per API Graph ed emettere una richiesta per i dati del profilo dell'utente.
* La `UserInfo.cs` classe designa le proprietà del profilo utente richieste con l' <xref:System.Text.Json.Serialization.JsonPropertyNameAttribute> attributo e il nome JSON usato da AAD per tali proprietà.

`Pages/CallUser.razor`:

```razor
@page "/CallUser"
@using System.ComponentModel.DataAnnotations
@using System.Text.Json.Serialization
@using Microsoft.AspNetCore.Components.WebAssembly.Authentication
@using Microsoft.Extensions.Logging
@inject IAccessTokenProvider TokenProvider
@inject IHttpClientFactory ClientFactory
@inject ILogger<CallUser> Logger

<h3>Call User</h3>

<EditForm Model="@callInfo" OnValidSubmit="@HandleValidSubmit">
    <DataAnnotationsValidator />
    <ValidationSummary />

    <p>
        <label>
            Message:
            <InputTextArea @bind-Value="callInfo.Message" />
        </label>
    </p>

    <button type="submit">Place call</button>

    <p>
        @formStatus
    </p>
</EditForm>

@code {
    private string formStatus;
    private CallInfo callInfo = new CallInfo();

    private async Task HandleValidSubmit()
    {
        var tokenResult = await TokenProvider.RequestAccessToken(
            new AccessTokenRequestOptions
            {
                Scopes = new[] { "https://graph.microsoft.com/User.Read" }
            });

        if (tokenResult.TryGetToken(out var token))
        {
            var client = ClientFactory.CreateClient("GraphAPI");

            var userInfo = await client.GetFromJsonAsync<UserInfo>("v1.0/me");

            if (userInfo != null)
            {
                // Use userInfo.MobilePhone and callInfo.Message to make a call

                formStatus = "Form successfully processed.";
                Logger.LogInformation(
                    $"Form successfully processed at {DateTime.UtcNow}. " +
                    $"Mobile Phone: {userInfo.MobilePhone}");
            }
        }
        else
        {
            formStatus = "There was a problem processing the form.";
            Logger.LogError("Token failure");
        }
    }

    private class CallInfo
    {
        [Required]
        [StringLength(1000, ErrorMessage = "Message too long (1,000 char limit)")]
        public string Message { get; set; }
    }

    private class UserInfo
    {
        [JsonPropertyName("mobilePhone")]
        public string MobilePhone { get; set; }
    }
}
```

### <a name="customize-user-claims-with-graph-api-and-a-named-client"></a>Personalizzare le attestazioni utente con API Graph e un client denominato

Questa sezione usa il [gestore dei messaggi di autorizzazione Graph ( `GraphAuthorizationMessageHandler.cs` ) e `Program.Main` le aggiunte all'app](#named-client-with-graph-api) descritta in precedenza in questo articolo, che fornisce un oggetto denominato <xref:System.Net.Http.HttpClient> per API Graph.

Nell'esempio seguente l'app crea un'attestazione di numero di telefono cellulare per l'utente dal numero di telefono cellulare del profilo utente di AAD. L'app deve avere l' `User.Read` ambito di API Graph configurato in AAD.

Aggiungere una `UserInfo.cs` classe all'app e designare le proprietà del profilo utente richieste con l' <xref:System.Text.Json.Serialization.JsonPropertyNameAttribute> attributo e il nome JSON usato da AAD per queste proprietà:

```csharp
using System.Text.Json.Serialization;

public class UserInfo
{
    [JsonPropertyName("mobilePhone")]
    public string MobilePhone { get; set; }
}
```

Nella Factory degli account utente personalizzata seguente il Framework <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteUserAccount> rappresenta l'account dell'utente. Se l'app richiede una classe di account utente personalizzata che estende <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteUserAccount> , scambiare la classe di account utente personalizzata per <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteUserAccount> nel codice riportato di seguito.

`CustomAccountFactory.cs`:

```csharp
using System.Net.Http;
using System.Net.Http.Json;
using System.Security.Claims;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Components.WebAssembly.Authentication;
using Microsoft.AspNetCore.Components.WebAssembly.Authentication.Internal;
using Microsoft.Extensions.Logging;

public class CustomAccountFactory
    : AccountClaimsPrincipalFactory<RemoteUserAccount>
{
    private readonly ILogger<CustomAccountFactory> logger;
    private readonly IHttpClientFactory clientFactory;

    public CustomAccountFactory(IAccessTokenProviderAccessor accessor, 
        IHttpClientFactory clientFactory, 
        ILogger<CustomAccountFactory> logger)
        : base(accessor)
    {
        this.clientFactory = clientFactory;
        this.logger = logger;
    }

    public async override ValueTask<ClaimsPrincipal> CreateUserAsync(
        RemoteUserAccount account,
        RemoteAuthenticationUserOptions options)
    {
        var initialUser = await base.CreateUserAsync(account, options);

        if (initialUser.Identity.IsAuthenticated)
        {
            var userIdentity = (ClaimsIdentity)initialUser.Identity;

            try
            {
                var client = clientFactory.CreateClient("GraphAPI");

                var userInfo = await client.GetFromJsonAsync<UserInfo>("v1.0/me");

                if (userInfo != null)
                {
                    userIdentity.AddClaim(new Claim("mobilephone", 
                        userInfo.MobilePhone));
                }
            }
            catch (AccessTokenNotAvailableException exception)
            {
                logger.LogError("Graph API access token failure: {Message}",
                    exception.Message);
            }
        }

        return initialUser;
    }
}
```

In `Program.Main` ( `Program.cs` ) configurare l'autenticazione MSAL per l'uso della factory dell'account utente personalizzata: se l'app usa una classe di account utente personalizzata che estende <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteUserAccount> , scambiare la classe di account utente personalizzata per <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteUserAccount> nel codice seguente:

```csharp
using Microsoft.AspNetCore.Components.WebAssembly.Authentication;

...

builder.Services.AddMsalAuthentication<RemoteAuthenticationState, 
    RemoteUserAccount>(options =>
    {
        builder.Configuration.Bind("AzureAd", 
            options.ProviderOptions.Authentication);
    })
    .AddAccountClaimsPrincipalFactory<RemoteAuthenticationState, RemoteUserAccount, 
        CustomAccountFactory>();
```

L'esempio precedente è per un'app che usa l'autenticazione di AAD con MSAL. Esistono modelli simili per OIDC e l'autenticazione API. Per ulteriori informazioni, vedere gli esempi nella sezione [personalizzare l'utente con un'attestazione di payload](xref:blazor/security/webassembly/additional-scenarios#customize-the-user-with-a-payload-claim) .
