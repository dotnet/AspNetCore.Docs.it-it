---
title: 'Usare API Graph con ASP.NET Core :::no-loc(Blazor WebAssembly):::'
author: guardrex
description: 'Informazioni su come usare API Graph con le :::no-loc(Blazor)::: app WebAssemlby.'
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 10/27/2020
no-loc:
- ':::no-loc(appsettings.json):::'
- ':::no-loc(ASP.NET Core Identity):::'
- ':::no-loc(cookie):::'
- ':::no-loc(Cookie):::'
- ':::no-loc(Blazor):::'
- ':::no-loc(Blazor Server):::'
- ':::no-loc(Blazor WebAssembly):::'
- ':::no-loc(Identity):::'
- ":::no-loc(Let's Encrypt):::"
- ':::no-loc(Razor):::'
- ':::no-loc(SignalR):::'
uid: blazor/security/webassembly/graph-api
ms.openlocfilehash: 569a88630f7b75e866d8ecda99605ebe3bc58db8
ms.sourcegitcommit: d64bf0cbe763beda22a7728c7f10d07fc5e19262
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 11/03/2020
ms.locfileid: "93234429"
---
# <a name="use-graph-api-with-aspnet-core-no-locblazor-webassembly"></a><span data-ttu-id="d29d0-103">Usare API Graph con ASP.NET Core :::no-loc(Blazor WebAssembly):::</span><span class="sxs-lookup"><span data-stu-id="d29d0-103">Use Graph API with ASP.NET Core :::no-loc(Blazor WebAssembly):::</span></span>

<span data-ttu-id="d29d0-104">Di [Javier Calvarro Nelson](https://github.com/javiercn) e [Luke Latham](https://github.com/guardrex)</span><span class="sxs-lookup"><span data-stu-id="d29d0-104">By [Javier Calvarro Nelson](https://github.com/javiercn) and [Luke Latham](https://github.com/guardrex)</span></span>

::: moniker range=">= aspnetcore-5.0"

<span data-ttu-id="d29d0-105">[Microsoft Graph API](/graph/use-the-api) è un'API Web RESTful che consente :::no-loc(Blazor)::: ad altre app .NET Framework di accedere alle risorse del servizio Microsoft Cloud.</span><span class="sxs-lookup"><span data-stu-id="d29d0-105">[Microsoft Graph API](/graph/use-the-api) is a RESTful web API that enables :::no-loc(Blazor)::: and other .NET Framework apps to access Microsoft Cloud service resources.</span></span>

## <a name="graph-sdk"></a><span data-ttu-id="d29d0-106">SDK per Graph</span><span class="sxs-lookup"><span data-stu-id="d29d0-106">Graph SDK</span></span>

<span data-ttu-id="d29d0-107">[Microsoft Graph SDK](/graph/sdks/sdks-overview) sono progettati per semplificare la creazione di applicazioni di alta qualità, efficienti e resilienti che accedono a Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="d29d0-107">[Microsoft Graph SDKs](/graph/sdks/sdks-overview) are designed to simplify building high-quality, efficient, and resilient applications that access Microsoft Graph.</span></span>

<span data-ttu-id="d29d0-108">Gli esempi in questa sezione richiedono riferimenti ai pacchetti per i pacchetti seguenti nel file di progetto del file di progetto dell'app o autonomo *`Client`* :</span><span class="sxs-lookup"><span data-stu-id="d29d0-108">The examples in this section require package references for the following packages in the project file of the standalone or *`Client`* app's project file:</span></span>

* [`Microsoft.Extensions.Http`](https://www.nuget.org/packages/Microsoft.Extensions.Http)
* [`Microsoft.Graph`](https://www.nuget.org/packages/Microsoft.Graph)

<span data-ttu-id="d29d0-109">In ognuna delle sottosezioni seguenti di questo articolo vengono usate le classi e la configurazione di utilità seguenti:</span><span class="sxs-lookup"><span data-stu-id="d29d0-109">The following utility classes and configuration are used in each of the following subsections of this article:</span></span>

* [<span data-ttu-id="d29d0-110">Chiamare API Graph da un componente usando Graph SDK</span><span class="sxs-lookup"><span data-stu-id="d29d0-110">Call Graph API from a component using the Graph SDK</span></span>](#call-graph-api-from-a-component-using-the-graph-sdk)
* [<span data-ttu-id="d29d0-111">Personalizzare le attestazioni utente con Graph SDK</span><span class="sxs-lookup"><span data-stu-id="d29d0-111">Customize user claims with the Graph SDK</span></span>](#customize-user-claims-with-the-graph-sdk)

<span data-ttu-id="d29d0-112">Dopo l'aggiunta degli ambiti dell'API Microsoft Graph nell'area AAD della portale di Azure:</span><span class="sxs-lookup"><span data-stu-id="d29d0-112">After adding the Microsoft Graph API scopes in the AAD area of the Azure portal:</span></span>

* <span data-ttu-id="d29d0-113">Aggiungere la `GraphClientExtensions.cs` classe seguente all'app autonoma o all' *`Client`* app di una :::no-loc(Blazor)::: soluzione ospitata.</span><span class="sxs-lookup"><span data-stu-id="d29d0-113">Add the following `GraphClientExtensions.cs` class to the standalone app or *`Client`* app of a hosted :::no-loc(Blazor)::: solution.</span></span>
* <span data-ttu-id="d29d0-114">Specificare gli ambiti necessari per la <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AccessTokenRequestOptions.Scopes> proprietà dell'oggetto <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AccessTokenRequestOptions> nel `AuthenticateRequestAsync` metodo.</span><span class="sxs-lookup"><span data-stu-id="d29d0-114">Provide the required scopes to the <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AccessTokenRequestOptions.Scopes> property of the <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AccessTokenRequestOptions> in the `AuthenticateRequestAsync` method.</span></span> <span data-ttu-id="d29d0-115">Nell'esempio seguente `User.Read` viene specificato l'ambito corrispondente agli esempi nelle sezioni successive di questo articolo.</span><span class="sxs-lookup"><span data-stu-id="d29d0-115">In the following example, the `User.Read` scope is specified to match the examples in later sections of this article.</span></span>

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
        public NoOpGraphAuthenticationProvider(IAccessTokenProvider provider)
        {
            Provider = provider;
        }

        public IAccessTokenProvider TokenProvider { get; }

        public async Task AuthenticateRequestAsync(HttpRequestMessage request)
        {
            var result = await TokenProvider.RequestAccessToken(
                new AccessTokenRequestOptions()
                {
                    Scopes = {STRING ARRAY OF SCOPES}
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

<span data-ttu-id="d29d0-116">Il segnaposto `{STRING ARRAY OF SCOPES}` nel codice precedente è una matrice di stringhe degli ambiti consentiti.</span><span class="sxs-lookup"><span data-stu-id="d29d0-116">The placeholder `{STRING ARRAY OF SCOPES}` in the preceding code is a string array of the permitted scopes.</span></span> <span data-ttu-id="d29d0-117">Ad esempio, impostare sull' `Scopes` `User.Read` ambito per gli esempi nelle sezioni seguenti di questo articolo:</span><span class="sxs-lookup"><span data-stu-id="d29d0-117">For example, set `Scopes` to the `User.Read` scope for the examples in the following sections of this article:</span></span>

```csharp
Scopes = new[] { "https://graph.microsoft.com/User.Read" }
```

<span data-ttu-id="d29d0-118">In `Program.Main` ( `Program.cs` ) aggiungere i servizi client Graph e la configurazione con il `AddGraphClient` metodo di estensione:</span><span class="sxs-lookup"><span data-stu-id="d29d0-118">In `Program.Main` (`Program.cs`), add the Graph client services and configuration with the `AddGraphClient` extension method:</span></span>

```csharp
builder.Services.AddGraphClient({STRING ARRAY OF SCOPES});
```

<span data-ttu-id="d29d0-119">Il segnaposto `{STRING ARRAY OF SCOPES}` nel codice precedente è una matrice di stringhe degli ambiti consentiti.</span><span class="sxs-lookup"><span data-stu-id="d29d0-119">The placeholder `{STRING ARRAY OF SCOPES}` in the preceding code is a string array of the permitted scopes.</span></span> <span data-ttu-id="d29d0-120">Ad esempio, passare l' `User.Read` ambito a `AddGraphClient` per gli esempi nelle sezioni seguenti di questo articolo:</span><span class="sxs-lookup"><span data-stu-id="d29d0-120">For example, pass the `User.Read` scope to `AddGraphClient` for the examples in the following sections of this article:</span></span>

```csharp
builder.Services.AddGraphClient("https://graph.microsoft.com/User.Read");
```

### <a name="call-graph-api-from-a-component-using-the-graph-sdk"></a><span data-ttu-id="d29d0-121">Chiamare API Graph da un componente usando Graph SDK</span><span class="sxs-lookup"><span data-stu-id="d29d0-121">Call Graph API from a component using the Graph SDK</span></span>

<span data-ttu-id="d29d0-122">Questa sezione usa le [classi di utilità ( `GraphClientExtensions.cs` )](#graph-sdk) descritte in precedenza in questo articolo.</span><span class="sxs-lookup"><span data-stu-id="d29d0-122">This section uses the [utility classes (`GraphClientExtensions.cs`)](#graph-sdk) described earlier in this article.</span></span> <span data-ttu-id="d29d0-123">Il `GraphExample` componente seguente usa un oggetto inserito `GraphServiceClient` per ottenere i dati del profilo AAD dell'utente e visualizzare il numero di telefono cellulare:</span><span class="sxs-lookup"><span data-stu-id="d29d0-123">The following `GraphExample` component uses an injected `GraphServiceClient` to obtain the user's AAD profile data and display their mobile phone number:</span></span>

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

### <a name="customize-user-claims-with-the-graph-sdk"></a><span data-ttu-id="d29d0-124">Personalizzare le attestazioni utente con Graph SDK</span><span class="sxs-lookup"><span data-stu-id="d29d0-124">Customize user claims with the Graph SDK</span></span>

<span data-ttu-id="d29d0-125">Questa sezione usa le [classi di utilità ( `GraphClientExtensions.cs` )](#graph-sdk) descritte in precedenza in questo articolo.</span><span class="sxs-lookup"><span data-stu-id="d29d0-125">This section uses the [utility classes (`GraphClientExtensions.cs`)](#graph-sdk) described earlier in this article.</span></span>

<span data-ttu-id="d29d0-126">Nell'esempio seguente l'app crea un'attestazione di numero di telefono cellulare per un utente dal numero di telefono cellulare del profilo utente di AAD.</span><span class="sxs-lookup"><span data-stu-id="d29d0-126">In the following example, the app creates a mobile phone number claim for a user from their AAD user profile's mobile phone number.</span></span> <span data-ttu-id="d29d0-127">L'app deve avere l' `User.Read` ambito di API Graph configurato in AAD.</span><span class="sxs-lookup"><span data-stu-id="d29d0-127">The app must have the `User.Read` Graph API scope configured in AAD.</span></span>

<span data-ttu-id="d29d0-128">Nella Factory degli account utente personalizzata seguente il Framework <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteUserAccount> rappresenta l'account dell'utente.</span><span class="sxs-lookup"><span data-stu-id="d29d0-128">In the following custom user account factory, the framework's <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteUserAccount> represents the user's account.</span></span> <span data-ttu-id="d29d0-129">Se l'app richiede una classe di account utente personalizzata che estende <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteUserAccount> , scambiare la classe di account utente personalizzata per <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteUserAccount> nel codice riportato di seguito.</span><span class="sxs-lookup"><span data-stu-id="d29d0-129">If the app requires a custom user account class that extends <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteUserAccount>, swap the custom user account class for <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteUserAccount> in the following code.</span></span>

<span data-ttu-id="d29d0-130">`CustomAccountFactory.cs`:</span><span class="sxs-lookup"><span data-stu-id="d29d0-130">`CustomAccountFactory.cs`:</span></span>

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

        if (initialUser.:::no-loc(Identity):::.IsAuthenticated)
        {
            var user:::no-loc(Identity)::: = (Claims:::no-loc(Identity):::)initialUser.:::no-loc(Identity):::;

            try
            {
                var graphClient = ActivatorUtilities
                    .CreateInstance<GraphServiceClient>(serviceProvider);
                var request = graphClient.Me.Request();
                var user = await request.GetAsync();

                if (user != null)
                {
                    user:::no-loc(Identity):::.AddClaim(new Claim("mobilephone", 
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

<span data-ttu-id="d29d0-131">In `Program.Main` ( `Program.cs` ) configurare l'autenticazione MSAL per l'uso della factory dell'account utente personalizzata: se l'app usa una classe di account utente personalizzata che estende <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteUserAccount> , scambiare la classe di account utente personalizzata per <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteUserAccount> nel codice seguente:</span><span class="sxs-lookup"><span data-stu-id="d29d0-131">In `Program.Main` (`Program.cs`), configure the MSAL authentication to use the custom user account factory: If the app uses a custom user account class that extends <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteUserAccount>, swap the custom user account class for <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteUserAccount> in the following code:</span></span>

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

## <a name="named-client-with-graph-api"></a><span data-ttu-id="d29d0-132">Client denominato con API Graph</span><span class="sxs-lookup"><span data-stu-id="d29d0-132">Named client with Graph API</span></span>

<span data-ttu-id="d29d0-133">Negli esempi di questa sezione viene usato un denominato <xref:System.Net.Http.HttpClient> per API Graph per ottenere il numero di telefono cellulare di un utente per elaborare una chiamata.</span><span class="sxs-lookup"><span data-stu-id="d29d0-133">The examples in this section use a named <xref:System.Net.Http.HttpClient> for Graph API to obtain a user's mobile phone number to process a call.</span></span>

<span data-ttu-id="d29d0-134">Gli esempi in questa sezione richiedono un riferimento al pacchetto per [`Microsoft.Extensions.Http`](https://www.nuget.org/packages/Microsoft.Extensions.Http) nel file di progetto del file di progetto dell'app autonomo o *`Client`* .</span><span class="sxs-lookup"><span data-stu-id="d29d0-134">The examples in this section require a package reference for [`Microsoft.Extensions.Http`](https://www.nuget.org/packages/Microsoft.Extensions.Http) in the project file of the standalone or *`Client`* app's project file.</span></span>

<span data-ttu-id="d29d0-135">Creare la classe e la configurazione del progetto seguenti per lavorare con API Graph.</span><span class="sxs-lookup"><span data-stu-id="d29d0-135">Create the following class and project configuration for working with Graph API.</span></span> <span data-ttu-id="d29d0-136">La classe e la configurazione seguenti vengono usate in ognuna delle sottosezioni seguenti di questo articolo:</span><span class="sxs-lookup"><span data-stu-id="d29d0-136">The following class and configuration are used in each of the following subsections of this article:</span></span>

* [<span data-ttu-id="d29d0-137">Chiamare API Graph da un componente</span><span class="sxs-lookup"><span data-stu-id="d29d0-137">Call Graph API from a component</span></span>](#call-graph-api-from-a-component)
* [<span data-ttu-id="d29d0-138">Personalizzare le attestazioni utente con API Graph e un client denominato</span><span class="sxs-lookup"><span data-stu-id="d29d0-138">Customize user claims with Graph API and a named client</span></span>](#customize-user-claims-with-graph-api-and-a-named-client)

<span data-ttu-id="d29d0-139">Dopo l'aggiunta degli ambiti dell'API Microsoft Graph nell'area AAD della portale di Azure, fornire gli ambiti necessari al gestore configurato dell'app per API Graph.</span><span class="sxs-lookup"><span data-stu-id="d29d0-139">After adding the Microsoft Graph API scopes in the AAD area of the Azure portal, provide the required scopes to the app's configured handler for Graph API.</span></span> <span data-ttu-id="d29d0-140">Nell'esempio seguente viene configurato il gestore per l' `User.Read` ambito.</span><span class="sxs-lookup"><span data-stu-id="d29d0-140">The following example configures the handler for the `User.Read` scope.</span></span> <span data-ttu-id="d29d0-141">È possibile aggiungere ambiti aggiuntivi.</span><span class="sxs-lookup"><span data-stu-id="d29d0-141">Additional scopes can be added.</span></span>

<span data-ttu-id="d29d0-142">`GraphAuthorizationMessageHandler.cs`:</span><span class="sxs-lookup"><span data-stu-id="d29d0-142">`GraphAuthorizationMessageHandler.cs`:</span></span>

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

<span data-ttu-id="d29d0-143">In `Program.Main` ( `Program.cs` ) configurare il nome <xref:System.Net.Http.HttpClient> per API Graph:</span><span class="sxs-lookup"><span data-stu-id="d29d0-143">In `Program.Main` (`Program.cs`), configure the named <xref:System.Net.Http.HttpClient> for Graph API:</span></span>

```csharp
builder.Services.AddScoped<GraphAPIAuthorizationMessageHandler>();

builder.Services.AddHttpClient("GraphAPI",
        client => client.BaseAddress = new Uri("https://graph.microsoft.com"))
    .AddHttpMessageHandler<GraphAPIAuthorizationMessageHandler>();
```

### <a name="call-graph-api-from-a-component"></a><span data-ttu-id="d29d0-144">Chiamare API Graph da un componente</span><span class="sxs-lookup"><span data-stu-id="d29d0-144">Call Graph API from a component</span></span>

<span data-ttu-id="d29d0-145">Questa sezione usa il [gestore dei messaggi di autorizzazione Graph ( `GraphAuthorizationMessageHandler.cs` ) e `Program.Main` le aggiunte all'app](#named-client-with-graph-api) descritta in precedenza in questo articolo, che fornisce un oggetto denominato <xref:System.Net.Http.HttpClient> per API Graph.</span><span class="sxs-lookup"><span data-stu-id="d29d0-145">This section uses the [Graph Authorization Message Handler (`GraphAuthorizationMessageHandler.cs`) and `Program.Main` additions to the app](#named-client-with-graph-api) described earlier in this article, which provides a named <xref:System.Net.Http.HttpClient> for Graph API.</span></span>

<span data-ttu-id="d29d0-146">In un :::no-loc(Razor)::: componente:</span><span class="sxs-lookup"><span data-stu-id="d29d0-146">In a :::no-loc(Razor)::: component:</span></span>

* <span data-ttu-id="d29d0-147">Creare una <xref:System.Net.Http.HttpClient> per API Graph ed emettere una richiesta per i dati del profilo dell'utente.</span><span class="sxs-lookup"><span data-stu-id="d29d0-147">Create an <xref:System.Net.Http.HttpClient> for Graph API and issue a request for the user's profile data.</span></span>
* <span data-ttu-id="d29d0-148">La `UserInfo.cs` classe designa le proprietà del profilo utente richieste con l' <xref:System.Text.Json.Serialization.JsonPropertyNameAttribute> attributo e il nome JSON usato da AAD per tali proprietà.</span><span class="sxs-lookup"><span data-stu-id="d29d0-148">The `UserInfo.cs` class designates the required user profile properties with the <xref:System.Text.Json.Serialization.JsonPropertyNameAttribute> attribute and the JSON name used by AAD for those properties.</span></span>

<span data-ttu-id="d29d0-149">`Pages/CallUser.razor`:</span><span class="sxs-lookup"><span data-stu-id="d29d0-149">`Pages/CallUser.razor`:</span></span>

```razor
@page "/CallUser"
@using System.ComponentModel.DataAnnotations
@using System.Text.Json.Serialization
@using Microsoft.AspNetCore.Components.WebAssembly.Authentication
@using Microsoft.Extensions.Logging
@inject IAccessTokenProvider TokenProvider
@inject IHttpClientFactory ClientFactory
@inject ILogger<CallUser> Logger
@inject ICallProcessor CallProcessor

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
                CallProcessor.Send(userInfo.MobilePhone, callInfo.Message);

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

> [!NOTE]
> <span data-ttu-id="d29d0-150">Nell'esempio precedente, lo sviluppatore implementa la coda personalizzata `ICallProcessor` ( `CallProcessor` ) e quindi inserisce le chiamate automatiche.</span><span class="sxs-lookup"><span data-stu-id="d29d0-150">In the preceding example, the developer implements the custom `ICallProcessor` (`CallProcessor`) to queue and then place automated calls.</span></span>

### <a name="customize-user-claims-with-graph-api-and-a-named-client"></a><span data-ttu-id="d29d0-151">Personalizzare le attestazioni utente con API Graph e un client denominato</span><span class="sxs-lookup"><span data-stu-id="d29d0-151">Customize user claims with Graph API and a named client</span></span>

<span data-ttu-id="d29d0-152">Questa sezione usa il [gestore dei messaggi di autorizzazione Graph ( `GraphAuthorizationMessageHandler.cs` ) e `Program.Main` le aggiunte all'app](#named-client-with-graph-api) descritta in precedenza in questo articolo, che fornisce un oggetto denominato <xref:System.Net.Http.HttpClient> per API Graph.</span><span class="sxs-lookup"><span data-stu-id="d29d0-152">This section uses the [Graph Authorization Message Handler (`GraphAuthorizationMessageHandler.cs`) and `Program.Main` additions to the app](#named-client-with-graph-api) described earlier in this article, which provides a named <xref:System.Net.Http.HttpClient> for Graph API.</span></span>

<span data-ttu-id="d29d0-153">Nell'esempio seguente l'app crea un'attestazione di numero di telefono cellulare per l'utente dal numero di telefono cellulare del profilo utente di AAD.</span><span class="sxs-lookup"><span data-stu-id="d29d0-153">In the following example, the app creates a mobile phone number claim for the user from their AAD user profile's mobile phone number.</span></span> <span data-ttu-id="d29d0-154">L'app deve avere l' `User.Read` ambito di API Graph configurato in AAD.</span><span class="sxs-lookup"><span data-stu-id="d29d0-154">The app must have the `User.Read` Graph API scope configured in AAD.</span></span>

<span data-ttu-id="d29d0-155">Aggiungere una `UserInfo.cs` classe all'app e designare le proprietà del profilo utente richieste con l' <xref:System.Text.Json.Serialization.JsonPropertyNameAttribute> attributo e il nome JSON usato da AAD per queste proprietà:</span><span class="sxs-lookup"><span data-stu-id="d29d0-155">Add a `UserInfo.cs` class to the app and designate the required user profile properties with the <xref:System.Text.Json.Serialization.JsonPropertyNameAttribute> attribute and the JSON name used by AAD for those properties:</span></span>

```csharp
using System.Text.Json.Serialization;

public class UserInfo
{
    [JsonPropertyName("mobilePhone")]
    public string MobilePhone { get; set; }
}
```

<span data-ttu-id="d29d0-156">Nella Factory degli account utente personalizzata seguente il Framework <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteUserAccount> rappresenta l'account dell'utente.</span><span class="sxs-lookup"><span data-stu-id="d29d0-156">In the following custom user account factory, the framework's <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteUserAccount> represents the user's account.</span></span> <span data-ttu-id="d29d0-157">Se l'app richiede una classe di account utente personalizzata che estende <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteUserAccount> , scambiare la classe di account utente personalizzata per <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteUserAccount> nel codice riportato di seguito.</span><span class="sxs-lookup"><span data-stu-id="d29d0-157">If the app requires a custom user account class that extends <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteUserAccount>, swap the custom user account class for <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteUserAccount> in the following code.</span></span>

<span data-ttu-id="d29d0-158">`CustomAccountFactory.cs`:</span><span class="sxs-lookup"><span data-stu-id="d29d0-158">`CustomAccountFactory.cs`:</span></span>

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

        if (initialUser.:::no-loc(Identity):::.IsAuthenticated)
        {
            var user:::no-loc(Identity)::: = (Claims:::no-loc(Identity):::)initialUser.:::no-loc(Identity):::;

            try
            {
                var client = clientFactory.CreateClient("GraphAPI");

                var userInfo = await client.GetFromJsonAsync<UserInfo>("v1.0/me");

                if (userInfo != null)
                {
                    user:::no-loc(Identity):::.AddClaim(new Claim("mobilephone", 
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

<span data-ttu-id="d29d0-159">In `Program.Main` ( `Program.cs` ) configurare l'autenticazione MSAL per l'uso della factory dell'account utente personalizzata: se l'app usa una classe di account utente personalizzata che estende <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteUserAccount> , scambiare la classe di account utente personalizzata per <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteUserAccount> nel codice seguente:</span><span class="sxs-lookup"><span data-stu-id="d29d0-159">In `Program.Main` (`Program.cs`), configure the MSAL authentication to use the custom user account factory: If the app uses a custom user account class that extends <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteUserAccount>, swap the custom user account class for <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteUserAccount> in the following code:</span></span>

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

<span data-ttu-id="d29d0-160">L'esempio precedente è per un'app che usa l'autenticazione di AAD con MSAL.</span><span class="sxs-lookup"><span data-stu-id="d29d0-160">The preceding example is for an app that uses AAD authentication with MSAL.</span></span> <span data-ttu-id="d29d0-161">Esistono modelli simili per OIDC e l'autenticazione API.</span><span class="sxs-lookup"><span data-stu-id="d29d0-161">Similar patterns exist for OIDC and API authentication.</span></span> <span data-ttu-id="d29d0-162">Per ulteriori informazioni, vedere gli esempi nella sezione [personalizzare l'utente con un'attestazione di payload](xref:blazor/security/webassembly/additional-scenarios#customize-the-user-with-a-payload-claim) .</span><span class="sxs-lookup"><span data-stu-id="d29d0-162">For more information, see the examples in [Customize the user with a payload claim](xref:blazor/security/webassembly/additional-scenarios#customize-the-user-with-a-payload-claim) section.</span></span>