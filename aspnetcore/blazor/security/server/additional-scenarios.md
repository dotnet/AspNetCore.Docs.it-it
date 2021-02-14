---
title: ASP.NET Core Blazor Server scenari di sicurezza aggiuntivi
author: guardrex
description: Informazioni su come configurare Blazor Server per altri scenari di sicurezza.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 10/06/2020
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
uid: blazor/security/server/additional-scenarios
ms.openlocfilehash: d47cfba75b640f57cc713049594d4e8acd1fcd0e
ms.sourcegitcommit: 1166b0ff3828418559510c661e8240e5c5717bb7
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/12/2021
ms.locfileid: "100280323"
---
# <a name="aspnet-core-blazor-server-additional-security-scenarios"></a><span data-ttu-id="d2340-103">ASP.NET Core Blazor Server scenari di sicurezza aggiuntivi</span><span class="sxs-lookup"><span data-stu-id="d2340-103">ASP.NET Core Blazor Server additional security scenarios</span></span>

::: moniker range=">= aspnetcore-5.0"

<h2 id="pass-tokens-to-a-blazor-server-app"><span data-ttu-id="d2340-104">Passare i token a un' Blazor Server app</span><span class="sxs-lookup"><span data-stu-id="d2340-104">Pass tokens to a Blazor Server app</span></span></h2>

<span data-ttu-id="d2340-105">I token disponibili all'esterno dei Razor componenti di un' Blazor Server app possono essere passati ai componenti con l'approccio descritto in questa sezione.</span><span class="sxs-lookup"><span data-stu-id="d2340-105">Tokens available outside of the Razor components in a Blazor Server app can be passed to components with the approach described in this section.</span></span>

<span data-ttu-id="d2340-106">Autenticare l' Blazor Server app come se si trattasse di un normale Razor sito o di un'app MVC.</span><span class="sxs-lookup"><span data-stu-id="d2340-106">Authenticate the Blazor Server app as you would with a regular Razor Pages or MVC app.</span></span> <span data-ttu-id="d2340-107">Eseguire il provisioning e salvare i token nell'autenticazione cookie .</span><span class="sxs-lookup"><span data-stu-id="d2340-107">Provision and save the tokens to the authentication cookie.</span></span> <span data-ttu-id="d2340-108">Ad esempio:</span><span class="sxs-lookup"><span data-stu-id="d2340-108">For example:</span></span>

```csharp
using Microsoft.AspNetCore.Authentication.OpenIdConnect;
using Microsoft.IdentityModel.Protocols.OpenIdConnect;

...

services.Configure<OpenIdConnectOptions>(AzureADDefaults.OpenIdScheme, options =>
{
    options.ResponseType = OpenIdConnectResponseType.Code;
    options.SaveTokens = true;

    options.Scope.Add("offline_access");
});
```

<span data-ttu-id="d2340-109">Facoltativamente, viene aggiunto un ambito aggiuntivo con `options.Scope.Add("{SCOPE}");` , in cui il segnaposto `{SCOPE}` è l'ambito aggiuntivo da aggiungere.</span><span class="sxs-lookup"><span data-stu-id="d2340-109">Optionally, additional scopes are added with `options.Scope.Add("{SCOPE}");`, where the placeholder `{SCOPE}` is the additional scope to add.</span></span>

<span data-ttu-id="d2340-110">Definire un servizio del provider di token con **ambito** che può essere usato nell' Blazor app per risolvere i token da [inserimento delle dipendenze](xref:blazor/fundamentals/dependency-injection):</span><span class="sxs-lookup"><span data-stu-id="d2340-110">Define a **scoped** token provider service that can be used within the Blazor app to resolve the tokens from [dependency injection (DI)](xref:blazor/fundamentals/dependency-injection):</span></span>

```csharp
public class TokenProvider
{
    public string AccessToken { get; set; }
    public string RefreshToken { get; set; }
}
```

<span data-ttu-id="d2340-111">In `Startup.ConfigureServices` aggiungere servizi per:</span><span class="sxs-lookup"><span data-stu-id="d2340-111">In `Startup.ConfigureServices`, add services for:</span></span>

* `IHttpClientFactory`
* `TokenProvider`

```csharp
services.AddHttpClient();
services.AddScoped<TokenProvider>();
```

<span data-ttu-id="d2340-112">Definire una classe per passare lo stato iniziale dell'app con i token di accesso e di aggiornamento:</span><span class="sxs-lookup"><span data-stu-id="d2340-112">Define a class to pass in the initial app state with the access and refresh tokens:</span></span>

```csharp
public class InitialApplicationState
{
    public string AccessToken { get; set; }
    public string RefreshToken { get; set; }
}
```

<span data-ttu-id="d2340-113">Nel `_Host.cshtml` file creare un'istanza di e `InitialApplicationState` passarla come parametro all'app:</span><span class="sxs-lookup"><span data-stu-id="d2340-113">In the `_Host.cshtml` file, create and instance of `InitialApplicationState` and pass it as a parameter to the app:</span></span>

```cshtml
@using Microsoft.AspNetCore.Authentication

...

@{
    var tokens = new InitialApplicationState
    {
        AccessToken = await HttpContext.GetTokenAsync("access_token"),
        RefreshToken = await HttpContext.GetTokenAsync("refresh_token")
    };
}

<component type="typeof(App)" param-InitialState="tokens" 
    render-mode="ServerPrerendered" />
```

<span data-ttu-id="d2340-114">Nel `App` componente ( `App.razor` ) risolvere il servizio e inizializzarlo con i dati del parametro:</span><span class="sxs-lookup"><span data-stu-id="d2340-114">In the `App` component (`App.razor`), resolve the service and initialize it with the data from the parameter:</span></span>

```razor
@inject TokenProvider TokenProvider

...

@code {
    [Parameter]
    public InitialApplicationState InitialState { get; set; }

    protected override Task OnInitializedAsync()
    {
        TokenProvider.AccessToken = InitialState.AccessToken;
        TokenProvider.RefreshToken = InitialState.RefreshToken;

        return base.OnInitializedAsync();
    }
}
```

<span data-ttu-id="d2340-115">Aggiungere un riferimento al pacchetto all'app per il [`Microsoft.AspNet.WebApi.Client`](https://www.nuget.org/packages/Microsoft.AspNet.WebApi.Client) pacchetto NuGet.</span><span class="sxs-lookup"><span data-stu-id="d2340-115">Add a package reference to the app for the [`Microsoft.AspNet.WebApi.Client`](https://www.nuget.org/packages/Microsoft.AspNet.WebApi.Client) NuGet package.</span></span>

<span data-ttu-id="d2340-116">Nel servizio che esegue una richiesta API protetta, inserire il provider di token e recuperare il token per la richiesta dell'API:</span><span class="sxs-lookup"><span data-stu-id="d2340-116">In the service that makes a secure API request, inject the token provider and retrieve the token for the API request:</span></span>

```csharp
using System;
using System.Net.Http;
using System.Threading.Tasks;

public class WeatherForecastService
{
    private readonly HttpClient http;
    private readonly TokenProvider tokenProvider;

    public WeatherForecastService(IHttpClientFactory clientFactory, 
        TokenProvider tokenProvider)
    {
        http = clientFactory.CreateClient();
        this.tokenProvider = tokenProvider;
    }

    public async Task<WeatherForecast[]> GetForecastAsync()
    {
        var token = tokenProvider.AccessToken;
        var request = new HttpRequestMessage(HttpMethod.Get, 
            "https://localhost:5003/WeatherForecast");
        request.Headers.Add("Authorization", $"Bearer {token}");
        var response = await http.SendAsync(request);
        response.EnsureSuccessStatusCode();

        return await response.Content.ReadAsAsync<WeatherForecast[]>();
    }
}
```

<h2 id="set-the-authentication-scheme"><span data-ttu-id="d2340-117">Impostare lo schema di autenticazione</span><span class="sxs-lookup"><span data-stu-id="d2340-117">Set the authentication scheme</span></span></h2>

<span data-ttu-id="d2340-118">Per un'app che usa più di un middleware di autenticazione e pertanto dispone di più di uno schema di autenticazione, lo schema Blazor usato da può essere impostato in modo esplicito nella configurazione dell'endpoint di `Startup.Configure` .</span><span class="sxs-lookup"><span data-stu-id="d2340-118">For an app that uses more than one Authentication Middleware and thus has more than one authentication scheme, the scheme that Blazor uses can be explicitly set in the endpoint configuration of `Startup.Configure`.</span></span> <span data-ttu-id="d2340-119">Nell'esempio seguente viene impostato lo schema di Azure Active Directory:</span><span class="sxs-lookup"><span data-stu-id="d2340-119">The following example sets the Azure Active Directory scheme:</span></span>

```csharp
endpoints.MapBlazorHub().RequireAuthorization(
    new AuthorizeAttribute 
    {
        AuthenticationSchemes = AzureADDefaults.AuthenticationScheme
    });
```

::: moniker-end

::: moniker range="< aspnetcore-5.0"

<h2 id="pass-tokens-to-a-blazor-server-app"><span data-ttu-id="d2340-120">Passare i token a un' Blazor Server app</span><span class="sxs-lookup"><span data-stu-id="d2340-120">Pass tokens to a Blazor Server app</span></span></h2>

<span data-ttu-id="d2340-121">I token disponibili all'esterno dei Razor componenti di un' Blazor Server app possono essere passati ai componenti con l'approccio descritto in questa sezione.</span><span class="sxs-lookup"><span data-stu-id="d2340-121">Tokens available outside of the Razor components in a Blazor Server app can be passed to components with the approach described in this section.</span></span>

<span data-ttu-id="d2340-122">Autenticare l' Blazor Server app come se si trattasse di un normale Razor sito o di un'app MVC.</span><span class="sxs-lookup"><span data-stu-id="d2340-122">Authenticate the Blazor Server app as you would with a regular Razor Pages or MVC app.</span></span> <span data-ttu-id="d2340-123">Eseguire il provisioning e salvare i token nell'autenticazione cookie .</span><span class="sxs-lookup"><span data-stu-id="d2340-123">Provision and save the tokens to the authentication cookie.</span></span> <span data-ttu-id="d2340-124">Ad esempio:</span><span class="sxs-lookup"><span data-stu-id="d2340-124">For example:</span></span>

```csharp
using Microsoft.AspNetCore.Authentication.OpenIdConnect;
using Microsoft.IdentityModel.Protocols.OpenIdConnect;

...

services.Configure<OpenIdConnectOptions>(AzureADDefaults.OpenIdScheme, options =>
{
    options.ResponseType = OpenIdConnectResponseType.Code;
    options.SaveTokens = true;

    options.Scope.Add("offline_access");
});
```

<span data-ttu-id="d2340-125">Facoltativamente, viene aggiunto un ambito aggiuntivo con `options.Scope.Add("{SCOPE}");` , in cui il segnaposto `{SCOPE}` è l'ambito aggiuntivo da aggiungere.</span><span class="sxs-lookup"><span data-stu-id="d2340-125">Optionally, additional scopes are added with `options.Scope.Add("{SCOPE}");`, where the placeholder `{SCOPE}` is the additional scope to add.</span></span>

<span data-ttu-id="d2340-126">Facoltativamente, la risorsa viene specificata con `options.Resource = "{RESOURCE}";` , dove il segnaposto `{RESOURCE}` è la risorsa.</span><span class="sxs-lookup"><span data-stu-id="d2340-126">Optionally, the resource is specified with `options.Resource = "{RESOURCE}";`, where the placeholder `{RESOURCE}` is the resource.</span></span> <span data-ttu-id="d2340-127">Ad esempio:</span><span class="sxs-lookup"><span data-stu-id="d2340-127">For example:</span></span>

```csharp
options.Resource = "https://graph.microsoft.com";
```

<span data-ttu-id="d2340-128">Definire una classe per passare lo stato iniziale dell'app con i token di accesso e di aggiornamento:</span><span class="sxs-lookup"><span data-stu-id="d2340-128">Define a class to pass in the initial app state with the access and refresh tokens:</span></span>

```csharp
public class InitialApplicationState
{
    public string AccessToken { get; set; }
    public string RefreshToken { get; set; }
}
```

<span data-ttu-id="d2340-129">Definire un servizio del provider di token con **ambito** che può essere usato nell' Blazor app per risolvere i token da [inserimento delle dipendenze](xref:blazor/fundamentals/dependency-injection):</span><span class="sxs-lookup"><span data-stu-id="d2340-129">Define a **scoped** token provider service that can be used within the Blazor app to resolve the tokens from [dependency injection (DI)](xref:blazor/fundamentals/dependency-injection):</span></span>

```csharp
public class TokenProvider
{
    public string AccessToken { get; set; }
    public string RefreshToken { get; set; }
}
```

<span data-ttu-id="d2340-130">In `Startup.ConfigureServices` aggiungere servizi per:</span><span class="sxs-lookup"><span data-stu-id="d2340-130">In `Startup.ConfigureServices`, add services for:</span></span>

* `IHttpClientFactory`
* `TokenProvider`

```csharp
services.AddHttpClient();
services.AddScoped<TokenProvider>();
```

<span data-ttu-id="d2340-131">Nel `_Host.cshtml` file creare un'istanza di e `InitialApplicationState` passarla come parametro all'app:</span><span class="sxs-lookup"><span data-stu-id="d2340-131">In the `_Host.cshtml` file, create and instance of `InitialApplicationState` and pass it as a parameter to the app:</span></span>

```cshtml
@using Microsoft.AspNetCore.Authentication

...

@{
    var tokens = new InitialApplicationState
    {
        AccessToken = await HttpContext.GetTokenAsync("access_token"),
        RefreshToken = await HttpContext.GetTokenAsync("refresh_token")
    };
}

<app>
    <component type="typeof(App)" param-InitialState="tokens" 
        render-mode="ServerPrerendered" />
</app>
```

<span data-ttu-id="d2340-132">Nel `App` componente ( `App.razor` ) risolvere il servizio e inizializzarlo con i dati del parametro:</span><span class="sxs-lookup"><span data-stu-id="d2340-132">In the `App` component (`App.razor`), resolve the service and initialize it with the data from the parameter:</span></span>

```razor
@inject TokenProvider TokenProvider

...

@code {
    [Parameter]
    public InitialApplicationState InitialState { get; set; }

    protected override Task OnInitializedAsync()
    {
        TokenProvider.AccessToken = InitialState.AccessToken;
        TokenProvider.RefreshToken = InitialState.RefreshToken;

        return base.OnInitializedAsync();
    }
}
```

<span data-ttu-id="d2340-133">Aggiungere un riferimento al pacchetto all'app per il [`Microsoft.AspNet.WebApi.Client`](https://www.nuget.org/packages/Microsoft.AspNet.WebApi.Client) pacchetto NuGet.</span><span class="sxs-lookup"><span data-stu-id="d2340-133">Add a package reference to the app for the [`Microsoft.AspNet.WebApi.Client`](https://www.nuget.org/packages/Microsoft.AspNet.WebApi.Client) NuGet package.</span></span>

<span data-ttu-id="d2340-134">Nel servizio che esegue una richiesta API protetta, inserire il provider di token e recuperare il token per la richiesta dell'API:</span><span class="sxs-lookup"><span data-stu-id="d2340-134">In the service that makes a secure API request, inject the token provider and retrieve the token for the API request:</span></span>

```csharp
using System;
using System.Net.Http;
using System.Threading.Tasks;

public class WeatherForecastService
{
    private readonly HttpClient http;
    private readonly TokenProvider tokenProvider;

    public WeatherForecastService(IHttpClientFactory clientFactory, 
        TokenProvider tokenProvider)
    {
        http = clientFactory.CreateClient();
        this.tokenProvider = tokenProvider;
    }

    public async Task<WeatherForecast[]> GetForecastAsync()
    {
        var token = tokenProvider.AccessToken;
        var request = new HttpRequestMessage(HttpMethod.Get, 
            "https://localhost:5003/WeatherForecast");
        request.Headers.Add("Authorization", $"Bearer {token}");
        var response = await http.SendAsync(request);
        response.EnsureSuccessStatusCode();

        return await response.Content.ReadAsAsync<WeatherForecast[]>();
    }
}
```

<h2 id="set-the-authentication-scheme"><span data-ttu-id="d2340-135">Impostare lo schema di autenticazione</span><span class="sxs-lookup"><span data-stu-id="d2340-135">Set the authentication scheme</span></span></h2>

<span data-ttu-id="d2340-136">Per un'app che usa più di un middleware di autenticazione e pertanto dispone di più di uno schema di autenticazione, lo schema Blazor usato da può essere impostato in modo esplicito nella configurazione dell'endpoint di `Startup.Configure` .</span><span class="sxs-lookup"><span data-stu-id="d2340-136">For an app that uses more than one Authentication Middleware and thus has more than one authentication scheme, the scheme that Blazor uses can be explicitly set in the endpoint configuration of `Startup.Configure`.</span></span> <span data-ttu-id="d2340-137">Nell'esempio seguente viene impostato lo schema di Azure Active Directory:</span><span class="sxs-lookup"><span data-stu-id="d2340-137">The following example sets the Azure Active Directory scheme:</span></span>

```csharp
endpoints.MapBlazorHub().RequireAuthorization(
    new AuthorizeAttribute 
    {
        AuthenticationSchemes = AzureADDefaults.AuthenticationScheme
    });
```

## <a name="use-openid-connect-oidc-v20-endpoints"></a><span data-ttu-id="d2340-138">Usare gli endpoint OpenID Connect (OIDC) v 2.0</span><span class="sxs-lookup"><span data-stu-id="d2340-138">Use OpenID Connect (OIDC) v2.0 endpoints</span></span>

<span data-ttu-id="d2340-139">Nelle versioni di ASP.NET Core precedenti alla 5,0, i modelli e la libreria di autenticazione Blazor usano gli endpoint OpenID Connect (OIDC) v 1.0.</span><span class="sxs-lookup"><span data-stu-id="d2340-139">In versions of ASP.NET Core prior to 5.0, the authentication library and Blazor templates use OpenID Connect (OIDC) v1.0 endpoints.</span></span> <span data-ttu-id="d2340-140">Per usare un endpoint v 2.0 con versioni di ASP.NET Core precedenti alla 5,0, configurare l' <xref:Microsoft.AspNetCore.Builder.OpenIdConnectOptions.Authority?displayProperty=nameWithType> opzione nel <xref:Microsoft.AspNetCore.Builder.OpenIdConnectOptions> :</span><span class="sxs-lookup"><span data-stu-id="d2340-140">To use a v2.0 endpoint with versions of ASP.NET Core prior to 5.0, configure the <xref:Microsoft.AspNetCore.Builder.OpenIdConnectOptions.Authority?displayProperty=nameWithType> option in the <xref:Microsoft.AspNetCore.Builder.OpenIdConnectOptions>:</span></span>

```csharp
services.Configure<OpenIdConnectOptions>(AzureADDefaults.OpenIdScheme, 
    options =>
    {
        options.Authority += "/v2.0";
    }
```

<span data-ttu-id="d2340-141">In alternativa, l'impostazione può essere eseguita nel file delle impostazioni dell'app ( `appsettings.json` ):</span><span class="sxs-lookup"><span data-stu-id="d2340-141">Alternatively, the setting can be made in the app settings (`appsettings.json`) file:</span></span>

```json
{
  "AzureAd": {
    "Authority": "https://login.microsoftonline.com/common/oauth2/v2.0/",
    ...
  }
}
```

<span data-ttu-id="d2340-142">Se l'uso di un segmento nell'autorità non è appropriato per il provider OIDC dell'app, ad esempio con i provider non AAD, impostare <xref:Microsoft.AspNetCore.Builder.OpenIdConnectOptions.Authority> direttamente la proprietà.</span><span class="sxs-lookup"><span data-stu-id="d2340-142">If tacking on a segment to the authority isn't appropriate for the app's OIDC provider, such as with non-AAD providers, set the <xref:Microsoft.AspNetCore.Builder.OpenIdConnectOptions.Authority> property directly.</span></span> <span data-ttu-id="d2340-143">Impostare la proprietà in <xref:Microsoft.AspNetCore.Builder.OpenIdConnectOptions> o nel file di impostazioni dell'app con la <xref:Microsoft.AspNetCore.Builder.OpenIdConnectOptions.Authority> chiave.</span><span class="sxs-lookup"><span data-stu-id="d2340-143">Either set the property in <xref:Microsoft.AspNetCore.Builder.OpenIdConnectOptions> or in the app settings file with the <xref:Microsoft.AspNetCore.Builder.OpenIdConnectOptions.Authority> key.</span></span>

### <a name="code-changes"></a><span data-ttu-id="d2340-144">Modifiche al codice</span><span class="sxs-lookup"><span data-stu-id="d2340-144">Code changes</span></span>

* <span data-ttu-id="d2340-145">L'elenco di attestazioni nel token ID cambia per gli endpoint della versione 2.0.</span><span class="sxs-lookup"><span data-stu-id="d2340-145">The list of claims in the ID token changes for v2.0 endpoints.</span></span> <span data-ttu-id="d2340-146">Per ulteriori informazioni, vedere [perché eseguire l'aggiornamento alla piattaforma di identità Microsoft (v 2.0)?](/azure/active-directory/azuread-dev/azure-ad-endpoint-comparison)</span><span class="sxs-lookup"><span data-stu-id="d2340-146">For more information, see [Why update to Microsoft identity platform (v2.0)?](/azure/active-directory/azuread-dev/azure-ad-endpoint-comparison)</span></span> <span data-ttu-id="d2340-147">nella documentazione di Azure.</span><span class="sxs-lookup"><span data-stu-id="d2340-147">in the Azure documentation.</span></span>
* <span data-ttu-id="d2340-148">Poiché le risorse vengono specificate negli URI di ambito per gli endpoint v 2.0, rimuovere l' <xref:Microsoft.AspNetCore.Builder.OpenIdConnectOptions.Resource?displayProperty=nameWithType> impostazione della proprietà in <xref:Microsoft.AspNetCore.Builder.OpenIdConnectOptions> :</span><span class="sxs-lookup"><span data-stu-id="d2340-148">Since resources are specified in scope URIs for v2.0 endpoints, remove the <xref:Microsoft.AspNetCore.Builder.OpenIdConnectOptions.Resource?displayProperty=nameWithType> property setting in <xref:Microsoft.AspNetCore.Builder.OpenIdConnectOptions>:</span></span>

  ```csharp
  services.Configure<OpenIdConnectOptions>(AzureADDefaults.OpenIdScheme, options => 
      {
          ...
          options.Resource = "...";    // REMOVE THIS LINE
          ...
      }
  ```

  <span data-ttu-id="d2340-149">Per altre informazioni, vedere [ambiti, non risorse](/azure/active-directory/azuread-dev/azure-ad-endpoint-comparison#scopes-not-resources) nella documentazione di Azure.</span><span class="sxs-lookup"><span data-stu-id="d2340-149">For more information, see [Scopes, not resources](/azure/active-directory/azuread-dev/azure-ad-endpoint-comparison#scopes-not-resources) in the Azure documentation.</span></span>

### <a name="app-id-uri"></a><span data-ttu-id="d2340-150">URI ID app</span><span class="sxs-lookup"><span data-stu-id="d2340-150">App ID URI</span></span>

* <span data-ttu-id="d2340-151">Quando si usano gli endpoint v 2.0, le API definiscono un oggetto *`App ID URI`* , che è destinato a rappresentare un identificatore univoco per l'API.</span><span class="sxs-lookup"><span data-stu-id="d2340-151">When using v2.0 endpoints, APIs define an *`App ID URI`*, which is meant to represent a unique identifier for the API.</span></span>
* <span data-ttu-id="d2340-152">Tutti gli ambiti includono l'URI ID app come prefisso, mentre gli endpoint v 2.0 emettono token di accesso con l'URI ID app come destinatari.</span><span class="sxs-lookup"><span data-stu-id="d2340-152">All scopes include the App ID URI as a prefix, and v2.0 endpoints emit access tokens with the App ID URI as the audience.</span></span>
* <span data-ttu-id="d2340-153">Quando si usano gli endpoint V 2.0, l'ID client configurato nell'API del server cambia dall'ID applicazione (ID client) dell'API all'URI ID app.</span><span class="sxs-lookup"><span data-stu-id="d2340-153">When using V2.0 endpoints, the client ID configured in the Server API changes from the API Application ID (Client ID) to the App ID URI.</span></span>

<span data-ttu-id="d2340-154">`appsettings.json`:</span><span class="sxs-lookup"><span data-stu-id="d2340-154">`appsettings.json`:</span></span>

```json
{
  "AzureAd": {
    ...
    "ClientId": "https://{TENANT}.onmicrosoft.com/{APP NAME}"
    ...
  }
}
```

<span data-ttu-id="d2340-155">È possibile trovare l'URI ID app da usare nella descrizione della registrazione dell'app provider OIDC.</span><span class="sxs-lookup"><span data-stu-id="d2340-155">You can find the App ID URI to use in the OIDC provider app registration description.</span></span>

::: moniker-end
