---
title: ASP.NET Core Blazor WebAssembly scenari di sicurezza aggiuntivi
author: guardrex
description: Informazioni su come configurare Blazor WebAssembly per altri scenari di sicurezza.
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
uid: blazor/security/webassembly/additional-scenarios
ms.openlocfilehash: 6c3d2f93e43cf3660a12f1c8cef1d381b587f5ea
ms.sourcegitcommit: 1166b0ff3828418559510c661e8240e5c5717bb7
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/12/2021
ms.locfileid: "100280539"
---
# <a name="aspnet-core-blazor-webassembly-additional-security-scenarios"></a><span data-ttu-id="c1d05-103">ASP.NET Core Blazor WebAssembly scenari di sicurezza aggiuntivi</span><span class="sxs-lookup"><span data-stu-id="c1d05-103">ASP.NET Core Blazor WebAssembly additional security scenarios</span></span>

## <a name="attach-tokens-to-outgoing-requests"></a><span data-ttu-id="c1d05-104">Connetti token alle richieste in uscita</span><span class="sxs-lookup"><span data-stu-id="c1d05-104">Attach tokens to outgoing requests</span></span>

<span data-ttu-id="c1d05-105"><xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AuthorizationMessageHandler> è <xref:System.Net.Http.DelegatingHandler> usato per alleghire i token di accesso alle <xref:System.Net.Http.HttpResponseMessage> istanze in uscita.</span><span class="sxs-lookup"><span data-stu-id="c1d05-105"><xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AuthorizationMessageHandler> is a <xref:System.Net.Http.DelegatingHandler> used to attach access tokens to outgoing <xref:System.Net.Http.HttpResponseMessage> instances.</span></span> <span data-ttu-id="c1d05-106">I token vengono acquisiti usando il <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.IAccessTokenProvider> servizio, registrato dal Framework.</span><span class="sxs-lookup"><span data-stu-id="c1d05-106">Tokens are acquired using the <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.IAccessTokenProvider> service, which is registered by the framework.</span></span> <span data-ttu-id="c1d05-107">Se non è possibile acquisire un token, <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AccessTokenNotAvailableException> viene generata un'eccezione.</span><span class="sxs-lookup"><span data-stu-id="c1d05-107">If a token can't be acquired, an <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AccessTokenNotAvailableException> is thrown.</span></span> <span data-ttu-id="c1d05-108"><xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AccessTokenNotAvailableException> dispone di un <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AccessTokenNotAvailableException.Redirect%2A> metodo che può essere utilizzato per passare all'utente al provider di identità per acquisire un nuovo token.</span><span class="sxs-lookup"><span data-stu-id="c1d05-108"><xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AccessTokenNotAvailableException> has a <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AccessTokenNotAvailableException.Redirect%2A> method that can be used to navigate the user to the identity provider to acquire a new token.</span></span>

<span data-ttu-id="c1d05-109">Per praticità, il Framework fornisce il <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.BaseAddressAuthorizationMessageHandler> preconfigurato con l'indirizzo di base dell'app come un URL autorizzato.</span><span class="sxs-lookup"><span data-stu-id="c1d05-109">For convenience, the framework provides the <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.BaseAddressAuthorizationMessageHandler> preconfigured with the app's base address as an authorized URL.</span></span> <span data-ttu-id="c1d05-110">**I token di accesso vengono aggiunti solo quando l'URI della richiesta si trova all'interno dell'URI di base dell'app.**</span><span class="sxs-lookup"><span data-stu-id="c1d05-110">**Access tokens are only added when the request URI is within the app's base URI.**</span></span> <span data-ttu-id="c1d05-111">Quando gli URI di richiesta in uscita non sono inclusi nell'URI di base dell'app, usare una [ `AuthorizationMessageHandler` classe personalizzata (*scelta consigliata*)](#custom-authorizationmessagehandler-class) o [configurare `AuthorizationMessageHandler`](#configure-authorizationmessagehandler).</span><span class="sxs-lookup"><span data-stu-id="c1d05-111">When outgoing request URIs aren't within the app's base URI, use a [custom `AuthorizationMessageHandler` class (*recommended*)](#custom-authorizationmessagehandler-class) or [configure the `AuthorizationMessageHandler`](#configure-authorizationmessagehandler).</span></span>

> [!NOTE]
> <span data-ttu-id="c1d05-112">Oltre alla configurazione dell'app client per l'accesso all'API server, l'API del server deve anche consentire le richieste tra le origini (CORS) quando il client e il server non si trovano allo stesso indirizzo di base.</span><span class="sxs-lookup"><span data-stu-id="c1d05-112">In addition to the client app configuration for server API access, the server API must also allow cross-origin requests (CORS) when the client and the server don't reside at the same base address.</span></span> <span data-ttu-id="c1d05-113">Per altre informazioni sulla configurazione CORS sul lato server, vedere la sezione [condivisione di risorse tra le origini (CORS)](#cross-origin-resource-sharing-cors) più avanti in questo articolo.</span><span class="sxs-lookup"><span data-stu-id="c1d05-113">For more information on server-side CORS configuration, see the [Cross-origin resource sharing (CORS)](#cross-origin-resource-sharing-cors) section later in this article.</span></span>

<span data-ttu-id="c1d05-114">Nell'esempio seguente:</span><span class="sxs-lookup"><span data-stu-id="c1d05-114">In the following example:</span></span>

* <span data-ttu-id="c1d05-115"><xref:Microsoft.Extensions.DependencyInjection.HttpClientFactoryServiceCollectionExtensions.AddHttpClient%2A> aggiunge <xref:System.Net.Http.IHttpClientFactory> e i servizi correlati alla raccolta di servizi e configura un oggetto denominato <xref:System.Net.Http.HttpClient> ( `ServerAPI` ).</span><span class="sxs-lookup"><span data-stu-id="c1d05-115"><xref:Microsoft.Extensions.DependencyInjection.HttpClientFactoryServiceCollectionExtensions.AddHttpClient%2A> adds <xref:System.Net.Http.IHttpClientFactory> and related services to the service collection and configures a named <xref:System.Net.Http.HttpClient> (`ServerAPI`).</span></span> <span data-ttu-id="c1d05-116"><xref:System.Net.Http.HttpClient.BaseAddress?displayProperty=nameWithType> è l'indirizzo di base dell'URI della risorsa quando si inviano le richieste.</span><span class="sxs-lookup"><span data-stu-id="c1d05-116"><xref:System.Net.Http.HttpClient.BaseAddress?displayProperty=nameWithType> is the base address of the resource URI when sending requests.</span></span> <span data-ttu-id="c1d05-117"><xref:System.Net.Http.IHttpClientFactory> viene fornito dal [`Microsoft.Extensions.Http`](https://www.nuget.org/packages/Microsoft.Extensions.Http) pacchetto NuGet.</span><span class="sxs-lookup"><span data-stu-id="c1d05-117"><xref:System.Net.Http.IHttpClientFactory> is provided by the [`Microsoft.Extensions.Http`](https://www.nuget.org/packages/Microsoft.Extensions.Http) NuGet package.</span></span>
* <span data-ttu-id="c1d05-118"><xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.BaseAddressAuthorizationMessageHandler> è <xref:System.Net.Http.DelegatingHandler> usato per alleghire i token di accesso alle <xref:System.Net.Http.HttpResponseMessage> istanze in uscita.</span><span class="sxs-lookup"><span data-stu-id="c1d05-118"><xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.BaseAddressAuthorizationMessageHandler> is the <xref:System.Net.Http.DelegatingHandler> used to attach access tokens to outgoing <xref:System.Net.Http.HttpResponseMessage> instances.</span></span> <span data-ttu-id="c1d05-119">I token di accesso vengono aggiunti solo quando l'URI della richiesta si trova all'interno dell'URI di base dell'app.</span><span class="sxs-lookup"><span data-stu-id="c1d05-119">Access tokens are only added when the request URI is within the app's base URI.</span></span>
* <span data-ttu-id="c1d05-120"><xref:System.Net.Http.IHttpClientFactory.CreateClient%2A?displayProperty=nameWithType> Crea e configura un' <xref:System.Net.Http.HttpClient> istanza per le richieste in uscita usando la configurazione che corrisponde al denominato <xref:System.Net.Http.HttpClient> ( `ServerAPI` ).</span><span class="sxs-lookup"><span data-stu-id="c1d05-120"><xref:System.Net.Http.IHttpClientFactory.CreateClient%2A?displayProperty=nameWithType> creates and configures an <xref:System.Net.Http.HttpClient> instance for outgoing requests using the configuration that corresponds to the named <xref:System.Net.Http.HttpClient> (`ServerAPI`).</span></span>

```csharp
using System.Net.Http;
using Microsoft.AspNetCore.Components.WebAssembly.Authentication;

...

builder.Services.AddHttpClient("ServerAPI", 
        client => client.BaseAddress = new Uri("https://www.example.com/base"))
    .AddHttpMessageHandler<BaseAddressAuthorizationMessageHandler>();

builder.Services.AddScoped(sp => sp.GetRequiredService<IHttpClientFactory>()
    .CreateClient("ServerAPI"));
```

<span data-ttu-id="c1d05-121">Per un' Blazor app basata sul Blazor WebAssembly modello di progetto ospitato, gli URI di richiesta si trovano nell'URI di base dell'applicazione per impostazione predefinita.</span><span class="sxs-lookup"><span data-stu-id="c1d05-121">For a Blazor app based on the Blazor WebAssembly Hosted project template, request URIs are within the app's base URI by default.</span></span> <span data-ttu-id="c1d05-122">Viene pertanto <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.IWebAssemblyHostEnvironment.BaseAddress?displayProperty=nameWithType> `new Uri(builder.HostEnvironment.BaseAddress)` assegnato a <xref:System.Net.Http.HttpClient.BaseAddress?displayProperty=nameWithType> in un'app generata dal modello di progetto.</span><span class="sxs-lookup"><span data-stu-id="c1d05-122">Therefore, <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.IWebAssemblyHostEnvironment.BaseAddress?displayProperty=nameWithType> (`new Uri(builder.HostEnvironment.BaseAddress)`) is assigned to the <xref:System.Net.Http.HttpClient.BaseAddress?displayProperty=nameWithType> in an app generated from the project template.</span></span>

<span data-ttu-id="c1d05-123">L'oggetto configurato <xref:System.Net.Http.HttpClient> viene usato per effettuare richieste autorizzate usando il [`try-catch`](/dotnet/csharp/language-reference/keywords/try-catch) modello:</span><span class="sxs-lookup"><span data-stu-id="c1d05-123">The configured <xref:System.Net.Http.HttpClient> is used to make authorized requests using the [`try-catch`](/dotnet/csharp/language-reference/keywords/try-catch) pattern:</span></span>

```razor
@using Microsoft.AspNetCore.Components.WebAssembly.Authentication
@inject HttpClient Http

...

protected override async Task OnInitializedAsync()
{
    private ExampleType[] examples;

    try
    {
        examples = 
            await Http.GetFromJsonAsync<ExampleType[]>("ExampleAPIMethod");

        ...
    }
    catch (AccessTokenNotAvailableException exception)
    {
        exception.Redirect();
    }
}
```

### <a name="custom-authorizationmessagehandler-class"></a><span data-ttu-id="c1d05-124">`AuthorizationMessageHandler`Classe personalizzata</span><span class="sxs-lookup"><span data-stu-id="c1d05-124">Custom `AuthorizationMessageHandler` class</span></span>

<span data-ttu-id="c1d05-125">*Queste linee guida sono consigliate per le app client che effettuano richieste in uscita a URI che non rientrano nell'URI di base dell'app.*</span><span class="sxs-lookup"><span data-stu-id="c1d05-125">*This guidance in this section is recommended for client apps that make outgoing requests to URIs that aren't within the app's base URI.*</span></span>

<span data-ttu-id="c1d05-126">Nell'esempio seguente, una classe personalizzata si estende <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AuthorizationMessageHandler> per l'utilizzo come <xref:System.Net.Http.DelegatingHandler> per un oggetto <xref:System.Net.Http.HttpClient> .</span><span class="sxs-lookup"><span data-stu-id="c1d05-126">In the following example, a custom class extends <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AuthorizationMessageHandler> for use as the <xref:System.Net.Http.DelegatingHandler> for an <xref:System.Net.Http.HttpClient>.</span></span> <span data-ttu-id="c1d05-127"><xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AuthorizationMessageHandler.ConfigureHandler%2A> Configura questo gestore per autorizzare le richieste HTTP in uscita usando un token di accesso.</span><span class="sxs-lookup"><span data-stu-id="c1d05-127"><xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AuthorizationMessageHandler.ConfigureHandler%2A> configures this handler to authorize outbound HTTP requests using an access token.</span></span> <span data-ttu-id="c1d05-128">Il token di accesso viene collegato solo se almeno uno degli URL autorizzati è una base dell'URI della richiesta ( <xref:System.Net.Http.HttpRequestMessage.RequestUri?displayProperty=nameWithType> ).</span><span class="sxs-lookup"><span data-stu-id="c1d05-128">The access token is only attached if at least one of the authorized URLs is a base of the request URI (<xref:System.Net.Http.HttpRequestMessage.RequestUri?displayProperty=nameWithType>).</span></span>

```csharp
using Microsoft.AspNetCore.Components;
using Microsoft.AspNetCore.Components.WebAssembly.Authentication;

public class CustomAuthorizationMessageHandler : AuthorizationMessageHandler
{
    public CustomAuthorizationMessageHandler(IAccessTokenProvider provider, 
        NavigationManager navigationManager)
        : base(provider, navigationManager)
    {
        ConfigureHandler(
            authorizedUrls: new[] { "https://www.example.com/base" },
            scopes: new[] { "example.read", "example.write" });
    }
}
```

<span data-ttu-id="c1d05-129">In `Program.Main` ( `Program.cs` ), `CustomAuthorizationMessageHandler` viene registrato come un servizio con ambito e viene configurato come <xref:System.Net.Http.DelegatingHandler> per le istanze in uscita <xref:System.Net.Http.HttpResponseMessage> effettuate da un oggetto denominato <xref:System.Net.Http.HttpClient> :</span><span class="sxs-lookup"><span data-stu-id="c1d05-129">In `Program.Main` (`Program.cs`), `CustomAuthorizationMessageHandler` is registered as a scoped service and is configured as the <xref:System.Net.Http.DelegatingHandler> for outgoing <xref:System.Net.Http.HttpResponseMessage> instances made by a named <xref:System.Net.Http.HttpClient>:</span></span>

```csharp
builder.Services.AddScoped<CustomAuthorizationMessageHandler>();

builder.Services.AddHttpClient("ServerAPI",
        client => client.BaseAddress = new Uri("https://www.example.com/base"))
    .AddHttpMessageHandler<CustomAuthorizationMessageHandler>();
```

<span data-ttu-id="c1d05-130">Per un' Blazor applicazione basata sul Blazor WebAssembly modello di progetto ospitato, <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.IWebAssemblyHostEnvironment.BaseAddress?displayProperty=nameWithType> ( `new Uri(builder.HostEnvironment.BaseAddress)` ) viene assegnato a per <xref:System.Net.Http.HttpClient.BaseAddress?displayProperty=nameWithType> impostazione predefinita.</span><span class="sxs-lookup"><span data-stu-id="c1d05-130">For a Blazor app based on the Blazor WebAssembly Hosted project template, <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.IWebAssemblyHostEnvironment.BaseAddress?displayProperty=nameWithType> (`new Uri(builder.HostEnvironment.BaseAddress)`) is assigned to the <xref:System.Net.Http.HttpClient.BaseAddress?displayProperty=nameWithType> by default.</span></span>

<span data-ttu-id="c1d05-131">L'oggetto configurato <xref:System.Net.Http.HttpClient> viene usato per effettuare richieste autorizzate usando il [`try-catch`](/dotnet/csharp/language-reference/keywords/try-catch) modello.</span><span class="sxs-lookup"><span data-stu-id="c1d05-131">The configured <xref:System.Net.Http.HttpClient> is used to make authorized requests using the [`try-catch`](/dotnet/csharp/language-reference/keywords/try-catch) pattern.</span></span> <span data-ttu-id="c1d05-132">Se il client viene creato con <xref:System.Net.Http.IHttpClientFactory.CreateClient%2A> ( [`Microsoft.Extensions.Http`](https://www.nuget.org/packages/Microsoft.Extensions.Http) pacchetto), <xref:System.Net.Http.HttpClient> vengono fornite le istanze di che includono i token di accesso quando si effettuano richieste all'API del server.</span><span class="sxs-lookup"><span data-stu-id="c1d05-132">Where the client is created with <xref:System.Net.Http.IHttpClientFactory.CreateClient%2A> ([`Microsoft.Extensions.Http`](https://www.nuget.org/packages/Microsoft.Extensions.Http) package), the <xref:System.Net.Http.HttpClient> is supplied instances that include access tokens when making requests to the server API.</span></span> <span data-ttu-id="c1d05-133">Se l'URI della richiesta è un URI relativo, come nell'esempio seguente ( `ExampleAPIMethod` ), viene combinato con l'oggetto <xref:System.Net.Http.HttpClient.BaseAddress> quando l'app client esegue la richiesta:</span><span class="sxs-lookup"><span data-stu-id="c1d05-133">If the request URI is a relative URI, as it is in the following example (`ExampleAPIMethod`), it's combined with the <xref:System.Net.Http.HttpClient.BaseAddress> when the client app makes the request:</span></span>

```razor
@inject IHttpClientFactory ClientFactory

...

@code {
    private ExampleType[] examples;

    protected override async Task OnInitializedAsync()
    {
        try
        {
            var client = ClientFactory.CreateClient("ServerAPI");

            examples = 
                await client.GetFromJsonAsync<ExampleType[]>("ExampleAPIMethod");
        }
        catch (AccessTokenNotAvailableException exception)
        {
            exception.Redirect();
        }
    }
}
```

### <a name="configure-authorizationmessagehandler"></a><span data-ttu-id="c1d05-134">Configurare`AuthorizationMessageHandler`</span><span class="sxs-lookup"><span data-stu-id="c1d05-134">Configure `AuthorizationMessageHandler`</span></span>

<span data-ttu-id="c1d05-135"><xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AuthorizationMessageHandler> può essere configurato con URL autorizzati, ambiti e un URL restituito usando il <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AuthorizationMessageHandler.ConfigureHandler%2A> metodo.</span><span class="sxs-lookup"><span data-stu-id="c1d05-135"><xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AuthorizationMessageHandler> can be configured with authorized URLs, scopes, and a return URL using the <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AuthorizationMessageHandler.ConfigureHandler%2A> method.</span></span> <span data-ttu-id="c1d05-136"><xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AuthorizationMessageHandler.ConfigureHandler%2A> Configura il gestore per autorizzare le richieste HTTP in uscita usando un token di accesso.</span><span class="sxs-lookup"><span data-stu-id="c1d05-136"><xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AuthorizationMessageHandler.ConfigureHandler%2A> configures the handler to authorize outbound HTTP requests using an access token.</span></span> <span data-ttu-id="c1d05-137">Il token di accesso viene collegato solo se almeno uno degli URL autorizzati è una base dell'URI della richiesta ( <xref:System.Net.Http.HttpRequestMessage.RequestUri?displayProperty=nameWithType> ).</span><span class="sxs-lookup"><span data-stu-id="c1d05-137">The access token is only attached if at least one of the authorized URLs is a base of the request URI (<xref:System.Net.Http.HttpRequestMessage.RequestUri?displayProperty=nameWithType>).</span></span> <span data-ttu-id="c1d05-138">Se l'URI della richiesta è un URI relativo, viene combinato con <xref:System.Net.Http.HttpClient.BaseAddress> .</span><span class="sxs-lookup"><span data-stu-id="c1d05-138">If the request URI is a relative URI, it's combined with the <xref:System.Net.Http.HttpClient.BaseAddress>.</span></span>

<span data-ttu-id="c1d05-139">Nell'esempio seguente <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AuthorizationMessageHandler> configura un <xref:System.Net.Http.HttpClient> in `Program.Main` ( `Program.cs` ):</span><span class="sxs-lookup"><span data-stu-id="c1d05-139">In the following example, <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AuthorizationMessageHandler> configures an <xref:System.Net.Http.HttpClient> in `Program.Main` (`Program.cs`):</span></span>

```csharp
using System.Net.Http;
using Microsoft.AspNetCore.Components.WebAssembly.Authentication;

...

builder.Services.AddScoped(sp => new HttpClient(
    sp.GetRequiredService<AuthorizationMessageHandler>()
    .ConfigureHandler(
        authorizedUrls: new[] { "https://www.example.com/base" },
        scopes: new[] { "example.read", "example.write" }))
    {
        BaseAddress = new Uri("https://www.example.com/base")
    });
```

<span data-ttu-id="c1d05-140">Per un' Blazor applicazione basata sul Blazor WebAssembly modello di progetto ospitato, per <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.IWebAssemblyHostEnvironment.BaseAddress?displayProperty=nameWithType> impostazione predefinita viene assegnato a quanto segue:</span><span class="sxs-lookup"><span data-stu-id="c1d05-140">For a Blazor app based on the Blazor WebAssembly Hosted project template, <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.IWebAssemblyHostEnvironment.BaseAddress?displayProperty=nameWithType> is assigned to the following by default:</span></span>

* <span data-ttu-id="c1d05-141">Oggetto <xref:System.Net.Http.HttpClient.BaseAddress?displayProperty=nameWithType> ( `new Uri(builder.HostEnvironment.BaseAddress)` ).</span><span class="sxs-lookup"><span data-stu-id="c1d05-141">The <xref:System.Net.Http.HttpClient.BaseAddress?displayProperty=nameWithType> (`new Uri(builder.HostEnvironment.BaseAddress)`).</span></span>
* <span data-ttu-id="c1d05-142">URL della `authorizedUrls` matrice.</span><span class="sxs-lookup"><span data-stu-id="c1d05-142">A URL of the `authorizedUrls` array.</span></span>

## <a name="typed-httpclient"></a><span data-ttu-id="c1d05-143">Tipizzato `HttpClient`</span><span class="sxs-lookup"><span data-stu-id="c1d05-143">Typed `HttpClient`</span></span>

<span data-ttu-id="c1d05-144">È possibile definire un client tipizzato che gestisce tutti i problemi di acquisizione di token e HTTP all'interno di una singola classe.</span><span class="sxs-lookup"><span data-stu-id="c1d05-144">A typed client can be defined that handles all of the HTTP and token acquisition concerns within a single class.</span></span>

<span data-ttu-id="c1d05-145">`WeatherForecastClient.cs`:</span><span class="sxs-lookup"><span data-stu-id="c1d05-145">`WeatherForecastClient.cs`:</span></span>

```csharp
using System.Net.Http;
using System.Net.Http.Json;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Components.WebAssembly.Authentication;
using static {APP ASSEMBLY}.Data;

public class WeatherForecastClient
{
    private readonly HttpClient http;
 
    public WeatherForecastClient(HttpClient http)
    {
        this.http = http;
    }
 
    public async Task<WeatherForecast[]> GetForecastAsync()
    {
        var forecasts = new WeatherForecast[0];

        try
        {
            forecasts = await http.GetFromJsonAsync<WeatherForecast[]>(
                "WeatherForecast");
        }
        catch (AccessTokenNotAvailableException exception)
        {
            exception.Redirect();
        }

        return forecasts;
    }
}
```

<span data-ttu-id="c1d05-146">Il segnaposto `{APP ASSEMBLY}` è il nome dell'assembly dell'app (ad esempio, `using static BlazorSample.Data;` ).</span><span class="sxs-lookup"><span data-stu-id="c1d05-146">The placeholder `{APP ASSEMBLY}` is the app's assembly name (for example, `using static BlazorSample.Data;`).</span></span>

<span data-ttu-id="c1d05-147">`Program.Main` (`Program.cs`):</span><span class="sxs-lookup"><span data-stu-id="c1d05-147">`Program.Main` (`Program.cs`):</span></span>

```csharp
using System.Net.Http;
using Microsoft.AspNetCore.Components.WebAssembly.Authentication;

...

builder.Services.AddHttpClient<WeatherForecastClient>(
        client => client.BaseAddress = new Uri("https://www.example.com/base"))
    .AddHttpMessageHandler<BaseAddressAuthorizationMessageHandler>();
```

<span data-ttu-id="c1d05-148">Per un' Blazor applicazione basata sul Blazor WebAssembly modello di progetto ospitato, <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.IWebAssemblyHostEnvironment.BaseAddress?displayProperty=nameWithType> ( `new Uri(builder.HostEnvironment.BaseAddress)` ) viene assegnato a per <xref:System.Net.Http.HttpClient.BaseAddress?displayProperty=nameWithType> impostazione predefinita.</span><span class="sxs-lookup"><span data-stu-id="c1d05-148">For a Blazor app based on the Blazor WebAssembly Hosted project template, <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.IWebAssemblyHostEnvironment.BaseAddress?displayProperty=nameWithType> (`new Uri(builder.HostEnvironment.BaseAddress)`) is assigned to the <xref:System.Net.Http.HttpClient.BaseAddress?displayProperty=nameWithType> by default.</span></span>

<span data-ttu-id="c1d05-149">`FetchData` componente ( `Pages/FetchData.razor` ):</span><span class="sxs-lookup"><span data-stu-id="c1d05-149">`FetchData` component (`Pages/FetchData.razor`):</span></span>

```razor
@inject WeatherForecastClient Client

...

protected override async Task OnInitializedAsync()
{
    forecasts = await Client.GetForecastAsync();
}
```

## <a name="configure-the-httpclient-handler"></a><span data-ttu-id="c1d05-150">Configurare il `HttpClient` gestore</span><span class="sxs-lookup"><span data-stu-id="c1d05-150">Configure the `HttpClient` handler</span></span>

<span data-ttu-id="c1d05-151">Il gestore può essere configurato ulteriormente con <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AuthorizationMessageHandler.ConfigureHandler%2A> per le richieste HTTP in uscita.</span><span class="sxs-lookup"><span data-stu-id="c1d05-151">The handler can be further configured with <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AuthorizationMessageHandler.ConfigureHandler%2A> for outbound HTTP requests.</span></span>

<span data-ttu-id="c1d05-152">`Program.Main` (`Program.cs`):</span><span class="sxs-lookup"><span data-stu-id="c1d05-152">`Program.Main` (`Program.cs`):</span></span>

```csharp
builder.Services.AddHttpClient<WeatherForecastClient>(
        client => client.BaseAddress = new Uri("https://www.example.com/base"))
    .AddHttpMessageHandler(sp => sp.GetRequiredService<AuthorizationMessageHandler>()
    .ConfigureHandler(
        authorizedUrls: new [] { "https://www.example.com/base" },
        scopes: new[] { "example.read", "example.write" }));
```

<span data-ttu-id="c1d05-153">Per un' Blazor applicazione basata sul Blazor WebAssembly modello di progetto ospitato, per <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.IWebAssemblyHostEnvironment.BaseAddress?displayProperty=nameWithType> impostazione predefinita viene assegnato a quanto segue:</span><span class="sxs-lookup"><span data-stu-id="c1d05-153">For a Blazor app based on the Blazor WebAssembly Hosted project template, <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.IWebAssemblyHostEnvironment.BaseAddress?displayProperty=nameWithType> is assigned to the following by default:</span></span>

* <span data-ttu-id="c1d05-154">Oggetto <xref:System.Net.Http.HttpClient.BaseAddress?displayProperty=nameWithType> ( `new Uri(builder.HostEnvironment.BaseAddress)` ).</span><span class="sxs-lookup"><span data-stu-id="c1d05-154">The <xref:System.Net.Http.HttpClient.BaseAddress?displayProperty=nameWithType> (`new Uri(builder.HostEnvironment.BaseAddress)`).</span></span>
* <span data-ttu-id="c1d05-155">URL della `authorizedUrls` matrice.</span><span class="sxs-lookup"><span data-stu-id="c1d05-155">A URL of the `authorizedUrls` array.</span></span>

## <a name="unauthenticated-or-unauthorized-web-api-requests-in-an-app-with-a-secure-default-client"></a><span data-ttu-id="c1d05-156">Richieste API Web non autenticate o non autorizzate in un'app con un client predefinito sicuro</span><span class="sxs-lookup"><span data-stu-id="c1d05-156">Unauthenticated or unauthorized web API requests in an app with a secure default client</span></span>

<span data-ttu-id="c1d05-157">Se l' Blazor WebAssembly app usa normalmente un valore predefinito sicuro <xref:System.Net.Http.HttpClient> , l'app può anche eseguire richieste API Web non autenticate o non autorizzate configurando un oggetto denominato <xref:System.Net.Http.HttpClient> :</span><span class="sxs-lookup"><span data-stu-id="c1d05-157">If the Blazor WebAssembly app ordinarily uses a secure default <xref:System.Net.Http.HttpClient>, the app can also make unauthenticated or unauthorized web API requests by configuring a named <xref:System.Net.Http.HttpClient>:</span></span>

<span data-ttu-id="c1d05-158">`Program.Main` (`Program.cs`):</span><span class="sxs-lookup"><span data-stu-id="c1d05-158">`Program.Main` (`Program.cs`):</span></span>

```csharp
builder.Services.AddHttpClient("ServerAPI.NoAuthenticationClient", 
    client => client.BaseAddress = new Uri("https://www.example.com/base"));
```

<span data-ttu-id="c1d05-159">Per un' Blazor applicazione basata sul Blazor WebAssembly modello di progetto ospitato, <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.IWebAssemblyHostEnvironment.BaseAddress?displayProperty=nameWithType> ( `new Uri(builder.HostEnvironment.BaseAddress)` ) viene assegnato a per <xref:System.Net.Http.HttpClient.BaseAddress?displayProperty=nameWithType> impostazione predefinita.</span><span class="sxs-lookup"><span data-stu-id="c1d05-159">For a Blazor app based on the Blazor WebAssembly Hosted project template, <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.IWebAssemblyHostEnvironment.BaseAddress?displayProperty=nameWithType> (`new Uri(builder.HostEnvironment.BaseAddress)`) is assigned to the <xref:System.Net.Http.HttpClient.BaseAddress?displayProperty=nameWithType> by default.</span></span>

<span data-ttu-id="c1d05-160">La registrazione precedente è aggiunta alla registrazione predefinita protetta esistente <xref:System.Net.Http.HttpClient> .</span><span class="sxs-lookup"><span data-stu-id="c1d05-160">The preceding registration is in addition to the existing secure default <xref:System.Net.Http.HttpClient> registration.</span></span>

<span data-ttu-id="c1d05-161">Un componente crea <xref:System.Net.Http.HttpClient> dalla <xref:System.Net.Http.IHttpClientFactory> ( [`Microsoft.Extensions.Http`](https://www.nuget.org/packages/Microsoft.Extensions.Http) pacchetto) per eseguire richieste non autenticate o non autorizzate:</span><span class="sxs-lookup"><span data-stu-id="c1d05-161">A component creates the <xref:System.Net.Http.HttpClient> from the <xref:System.Net.Http.IHttpClientFactory> ([`Microsoft.Extensions.Http`](https://www.nuget.org/packages/Microsoft.Extensions.Http) package) to make unauthenticated or unauthorized requests:</span></span>

```razor
@inject IHttpClientFactory ClientFactory

...

@code {
    private WeatherForecast[] forecasts;

    protected override async Task OnInitializedAsync()
    {
        var client = ClientFactory.CreateClient("ServerAPI.NoAuthenticationClient");

        forecasts = await client.GetFromJsonAsync<WeatherForecast[]>(
            "WeatherForecastNoAuthentication");
    }
}
```

> [!NOTE]
> <span data-ttu-id="c1d05-162">Il controller nell'API del server, `WeatherForecastNoAuthenticationController` per l'esempio precedente, non è contrassegnato con l' [ `[Authorize]` attributo](xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute).</span><span class="sxs-lookup"><span data-stu-id="c1d05-162">The controller in the server API, `WeatherForecastNoAuthenticationController` for the preceding example, isn't marked with the [`[Authorize]` attribute](xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute).</span></span>

<span data-ttu-id="c1d05-163">La decisione di usare un client protetto o un client non sicuro come <xref:System.Net.Http.HttpClient> istanza predefinita è spetta allo sviluppatore.</span><span class="sxs-lookup"><span data-stu-id="c1d05-163">The decision whether to use a secure client or an insecure client as the default <xref:System.Net.Http.HttpClient> instance is up to the developer.</span></span> <span data-ttu-id="c1d05-164">Un modo per prendere questa decisione consiste nel considerare il numero di endpoint autenticati e non autenticati contattati dall'app.</span><span class="sxs-lookup"><span data-stu-id="c1d05-164">One way to make this decision is to consider the number of authenticated versus unauthenticated endpoints that the app contacts.</span></span> <span data-ttu-id="c1d05-165">Se la maggior parte delle richieste dell'app è quella di proteggere gli endpoint dell'API, usare l' <xref:System.Net.Http.HttpClient> istanza autenticata come predefinita.</span><span class="sxs-lookup"><span data-stu-id="c1d05-165">If the majority of the app's requests are to secure API endpoints, use the authenticated <xref:System.Net.Http.HttpClient> instance as the default.</span></span> <span data-ttu-id="c1d05-166">In caso contrario, registrare l'istanza non autenticata <xref:System.Net.Http.HttpClient> come predefinita.</span><span class="sxs-lookup"><span data-stu-id="c1d05-166">Otherwise, register the unauthenticated <xref:System.Net.Http.HttpClient> instance as the default.</span></span>

<span data-ttu-id="c1d05-167">Un approccio alternativo all'utilizzo di <xref:System.Net.Http.IHttpClientFactory> consiste nel creare un [client tipizzato](#typed-httpclient) per l'accesso non autenticato agli endpoint anonimi.</span><span class="sxs-lookup"><span data-stu-id="c1d05-167">An alternative approach to using the <xref:System.Net.Http.IHttpClientFactory> is to create a [typed client](#typed-httpclient) for unauthenticated access to anonymous endpoints.</span></span>

## <a name="request-additional-access-tokens"></a><span data-ttu-id="c1d05-168">Richiedere token di accesso aggiuntivi</span><span class="sxs-lookup"><span data-stu-id="c1d05-168">Request additional access tokens</span></span>

<span data-ttu-id="c1d05-169">I token di accesso possono essere ottenuti manualmente chiamando `IAccessTokenProvider.RequestAccessToken` .</span><span class="sxs-lookup"><span data-stu-id="c1d05-169">Access tokens can be manually obtained by calling `IAccessTokenProvider.RequestAccessToken`.</span></span> <span data-ttu-id="c1d05-170">Nell'esempio seguente, un ambito aggiuntivo è richiesto da un'app per il valore predefinito <xref:System.Net.Http.HttpClient> .</span><span class="sxs-lookup"><span data-stu-id="c1d05-170">In the following example, an additional scope is required by an app for the default <xref:System.Net.Http.HttpClient>.</span></span> <span data-ttu-id="c1d05-171">Nell'esempio Microsoft Authentication Library (MSAL) viene configurato l'ambito con `MsalProviderOptions` :</span><span class="sxs-lookup"><span data-stu-id="c1d05-171">The Microsoft Authentication Library (MSAL) example configures the scope with `MsalProviderOptions`:</span></span>

<span data-ttu-id="c1d05-172">`Program.Main` (`Program.cs`):</span><span class="sxs-lookup"><span data-stu-id="c1d05-172">`Program.Main` (`Program.cs`):</span></span>

```csharp
builder.Services.AddMsalAuthentication(options =>
{
    ...

    options.ProviderOptions.AdditionalScopesToConsent.Add("{CUSTOM SCOPE 1}");
    options.ProviderOptions.AdditionalScopesToConsent.Add("{CUSTOM SCOPE 2}");
}
```

<span data-ttu-id="c1d05-173">I `{CUSTOM SCOPE 1}` `{CUSTOM SCOPE 2}` segnaposto e nell'esempio precedente sono ambiti personalizzati.</span><span class="sxs-lookup"><span data-stu-id="c1d05-173">The `{CUSTOM SCOPE 1}` and `{CUSTOM SCOPE 2}` placeholders in the preceding example are custom scopes.</span></span>

<span data-ttu-id="c1d05-174">Il `IAccessTokenProvider.RequestToken` metodo fornisce un overload che consente a un'app di effettuare il provisioning di un token di accesso con un determinato set di ambiti.</span><span class="sxs-lookup"><span data-stu-id="c1d05-174">The `IAccessTokenProvider.RequestToken` method provides an overload that allows an app to provision an access token with a given set of scopes.</span></span>

<span data-ttu-id="c1d05-175">In un Razor componente:</span><span class="sxs-lookup"><span data-stu-id="c1d05-175">In a Razor component:</span></span>

```razor
@using Microsoft.AspNetCore.Components.WebAssembly.Authentication
@inject IAccessTokenProvider TokenProvider

...

var tokenResult = await TokenProvider.RequestAccessToken(
    new AccessTokenRequestOptions
    {
        Scopes = new[] { "{CUSTOM SCOPE 1}", "{CUSTOM SCOPE 2}" }
    });

if (tokenResult.TryGetToken(out var token))
{
    ...
}
```

<span data-ttu-id="c1d05-176">I `{CUSTOM SCOPE 1}` `{CUSTOM SCOPE 2}` segnaposto e nell'esempio precedente sono ambiti personalizzati.</span><span class="sxs-lookup"><span data-stu-id="c1d05-176">The `{CUSTOM SCOPE 1}` and `{CUSTOM SCOPE 2}` placeholders in the preceding example are custom scopes.</span></span>

<span data-ttu-id="c1d05-177"><xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AccessTokenResult.TryGetToken%2A?displayProperty=nameWithType> Restituisce</span><span class="sxs-lookup"><span data-stu-id="c1d05-177"><xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AccessTokenResult.TryGetToken%2A?displayProperty=nameWithType> returns:</span></span>

* <span data-ttu-id="c1d05-178">`true` con l'oggetto `token` da usare.</span><span class="sxs-lookup"><span data-stu-id="c1d05-178">`true` with the `token` for use.</span></span>
* <span data-ttu-id="c1d05-179">`false` Se il token non viene recuperato.</span><span class="sxs-lookup"><span data-stu-id="c1d05-179">`false` if the token isn't retrieved.</span></span>

## <a name="cross-origin-resource-sharing-cors"></a><span data-ttu-id="c1d05-180">Condivisione di risorse tra le origini (CORS)</span><span class="sxs-lookup"><span data-stu-id="c1d05-180">Cross-origin resource sharing (CORS)</span></span>

<span data-ttu-id="c1d05-181">Quando si inviano le credenziali (autorizzazioni cookie s/header) sulle richieste CORS, l' `Authorization` intestazione deve essere consentita dal criterio CORS.</span><span class="sxs-lookup"><span data-stu-id="c1d05-181">When sending credentials (authorization cookies/headers) on CORS requests, the `Authorization` header must be allowed by the CORS policy.</span></span>

<span data-ttu-id="c1d05-182">Il criterio seguente include la configurazione per:</span><span class="sxs-lookup"><span data-stu-id="c1d05-182">The following policy includes configuration for:</span></span>

* <span data-ttu-id="c1d05-183">Origin della richiesta ( `http://localhost:5000` , `https://localhost:5001` ).</span><span class="sxs-lookup"><span data-stu-id="c1d05-183">Request origins (`http://localhost:5000`, `https://localhost:5001`).</span></span>
* <span data-ttu-id="c1d05-184">Qualsiasi metodo (verbo).</span><span class="sxs-lookup"><span data-stu-id="c1d05-184">Any method (verb).</span></span>
* <span data-ttu-id="c1d05-185">`Content-Type` e `Authorization` intestazioni.</span><span class="sxs-lookup"><span data-stu-id="c1d05-185">`Content-Type` and `Authorization` headers.</span></span> <span data-ttu-id="c1d05-186">Per consentire un'intestazione personalizzata (ad esempio, `x-custom-header` ), elencare l'intestazione quando si chiama <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.WithHeaders*> .</span><span class="sxs-lookup"><span data-stu-id="c1d05-186">To allow a custom header (for example, `x-custom-header`), list the header when calling <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.WithHeaders*>.</span></span>
* <span data-ttu-id="c1d05-187">Credenziali impostate dal codice JavaScript lato client ( `credentials` proprietà impostata su `include` ).</span><span class="sxs-lookup"><span data-stu-id="c1d05-187">Credentials set by client-side JavaScript code (`credentials` property set to `include`).</span></span>

```csharp
app.UseCors(policy => 
    policy.WithOrigins("http://localhost:5000", "https://localhost:5001")
    .AllowAnyMethod()
    .WithHeaders(HeaderNames.ContentType, HeaderNames.Authorization, "x-custom-header")
    .AllowCredentials());
```

<span data-ttu-id="c1d05-188">Una soluzione ospitata Blazor basata sul Blazor modello di progetto ospitato usa lo stesso indirizzo di base per le app client e server.</span><span class="sxs-lookup"><span data-stu-id="c1d05-188">A hosted Blazor solution based on the Blazor Hosted project template uses the same base address for the client and server apps.</span></span> <span data-ttu-id="c1d05-189"><xref:System.Net.Http.HttpClient.BaseAddress?displayProperty=nameWithType>Per impostazione predefinita, l'app client è impostata su un URI `builder.HostEnvironment.BaseAddress` .</span><span class="sxs-lookup"><span data-stu-id="c1d05-189">The client app's <xref:System.Net.Http.HttpClient.BaseAddress?displayProperty=nameWithType> is set to a URI of `builder.HostEnvironment.BaseAddress` by default.</span></span> <span data-ttu-id="c1d05-190">La configurazione di CORS **non** è necessaria nella configurazione predefinita di un'app ospitata creata dal Blazor modello di progetto ospitato.</span><span class="sxs-lookup"><span data-stu-id="c1d05-190">CORS configuration is **not** required in the default configuration of a hosted app created from the Blazor Hosted project template.</span></span> <span data-ttu-id="c1d05-191">Altre app client che non sono ospitate dal progetto server e non condividono **l'indirizzo di** base dell'app Server richiedono la configurazione CORS nel progetto server.</span><span class="sxs-lookup"><span data-stu-id="c1d05-191">Additional client apps that aren't hosted by the server project and don't share the server app's base address **do** require CORS configuration in the server project.</span></span>

<span data-ttu-id="c1d05-192">Per ulteriori informazioni, vedere <xref:security/cors> e il componente tester richieste HTTP dell'app di esempio ( `Components/HTTPRequestTester.razor` ).</span><span class="sxs-lookup"><span data-stu-id="c1d05-192">For more information, see <xref:security/cors> and the sample app's HTTP Request Tester component (`Components/HTTPRequestTester.razor`).</span></span>

## <a name="handle-token-request-errors"></a><span data-ttu-id="c1d05-193">Gestione degli errori di richiesta di token</span><span class="sxs-lookup"><span data-stu-id="c1d05-193">Handle token request errors</span></span>

<span data-ttu-id="c1d05-194">Quando un'applicazione a pagina singola (SPA) autentica un utente usando OpenID Connect (OIDC), lo stato di autenticazione viene gestito localmente all'interno della SPA e del Identity provider (IP) sotto forma di sessione cookie che viene impostata come risultato dell'utente che fornisce le credenziali.</span><span class="sxs-lookup"><span data-stu-id="c1d05-194">When a Single Page Application (SPA) authenticates a user using OpenID Connect (OIDC), the authentication state is maintained locally within the SPA and in the Identity Provider (IP) in the form of a session cookie that's set as a result of the user providing their credentials.</span></span>

<span data-ttu-id="c1d05-195">I token che l'IP emette per l'utente in genere sono validi per brevi periodi di tempo, circa un'ora in modo normale, quindi l'app client deve recuperare periodicamente nuovi token.</span><span class="sxs-lookup"><span data-stu-id="c1d05-195">The tokens that the IP emits for the user typically are valid for short periods of time, about one hour normally, so the client app must regularly fetch new tokens.</span></span> <span data-ttu-id="c1d05-196">In caso contrario, l'utente verrà disconnesso dopo la scadenza dei token concessi.</span><span class="sxs-lookup"><span data-stu-id="c1d05-196">Otherwise, the user would be logged-out after the granted tokens expire.</span></span> <span data-ttu-id="c1d05-197">Nella maggior parte dei casi, i client OIDC sono in grado di effettuare il provisioning di nuovi token senza richiedere all'utente di eseguire di nuovo l'autenticazione grazie allo stato di autenticazione o alla "sessione" mantenuta all'interno dell'IP.</span><span class="sxs-lookup"><span data-stu-id="c1d05-197">In most cases, OIDC clients are able to provision new tokens without requiring the user to authenticate again thanks to the authentication state or "session" that is kept within the IP.</span></span>

<span data-ttu-id="c1d05-198">In alcuni casi il client non è in grado di ottenere un token senza interazione dell'utente, ad esempio quando per qualche motivo l'utente si disconnette esplicitamente dall'IP.</span><span class="sxs-lookup"><span data-stu-id="c1d05-198">There are some cases in which the client can't get a token without user interaction, for example, when for some reason the user explicitly logs out from the IP.</span></span> <span data-ttu-id="c1d05-199">Questo scenario si verifica se un utente visita `https://login.microsoftonline.com` e si disconnette. In questi scenari, l'app non sa immediatamente che l'utente si è disconnesso. Tutti i token che il client include potrebbero non essere più validi.</span><span class="sxs-lookup"><span data-stu-id="c1d05-199">This scenario occurs if a user visits `https://login.microsoftonline.com` and logs out. In these scenarios, the app doesn't know immediately that the user has logged out. Any token that the client holds might no longer be valid.</span></span> <span data-ttu-id="c1d05-200">Inoltre, il client non è in grado di effettuare il provisioning di un nuovo token senza interazione dell'utente dopo la scadenza del token corrente.</span><span class="sxs-lookup"><span data-stu-id="c1d05-200">Also, the client isn't able to provision a new token without user interaction after the current token expires.</span></span>

<span data-ttu-id="c1d05-201">Questi scenari non sono specifici dell'autenticazione basata su token.</span><span class="sxs-lookup"><span data-stu-id="c1d05-201">These scenarios aren't specific to token-based authentication.</span></span> <span data-ttu-id="c1d05-202">Sono parte della natura delle Spa.</span><span class="sxs-lookup"><span data-stu-id="c1d05-202">They are part of the nature of SPAs.</span></span> <span data-ttu-id="c1d05-203">Una SPA cookie che usa s non riesce anche a chiamare un'API server se l'autenticazione cookie viene rimossa.</span><span class="sxs-lookup"><span data-stu-id="c1d05-203">An SPA using cookies also fails to call a server API if the authentication cookie is removed.</span></span>

<span data-ttu-id="c1d05-204">Quando un'app esegue chiamate API a risorse protette, è necessario tenere presente quanto segue:</span><span class="sxs-lookup"><span data-stu-id="c1d05-204">When an app performs API calls to protected resources, you must be aware of the following:</span></span>

* <span data-ttu-id="c1d05-205">Per eseguire il provisioning di un nuovo token di accesso per chiamare l'API, l'utente potrebbe essere necessario eseguire di nuovo l'autenticazione.</span><span class="sxs-lookup"><span data-stu-id="c1d05-205">To provision a new access token to call the API, the user might be required to authenticate again.</span></span>
* <span data-ttu-id="c1d05-206">Anche se il client dispone di un token che sembra essere valido, la chiamata al server potrebbe non riuscire perché il token è stato revocato dall'utente.</span><span class="sxs-lookup"><span data-stu-id="c1d05-206">Even if the client has a token that seems to be valid, the call to the server might fail because the token was revoked by the user.</span></span>

<span data-ttu-id="c1d05-207">Quando l'app richiede un token, esistono due possibili risultati:</span><span class="sxs-lookup"><span data-stu-id="c1d05-207">When the app requests a token, there are two possible outcomes:</span></span>

* <span data-ttu-id="c1d05-208">La richiesta ha esito positivo e l'app ha un token valido.</span><span class="sxs-lookup"><span data-stu-id="c1d05-208">The request succeeds, and the app has a valid token.</span></span>
* <span data-ttu-id="c1d05-209">La richiesta ha esito negativo e l'app deve autenticare di nuovo l'utente per ottenere un nuovo token.</span><span class="sxs-lookup"><span data-stu-id="c1d05-209">The request fails, and the app must authenticate the user again to obtain a new token.</span></span>

<span data-ttu-id="c1d05-210">Quando una richiesta di token ha esito negativo, è necessario decidere se si desidera salvare lo stato corrente prima di eseguire un reindirizzamento.</span><span class="sxs-lookup"><span data-stu-id="c1d05-210">When a token request fails, you need to decide whether you want to save any current state before you perform a redirection.</span></span> <span data-ttu-id="c1d05-211">Esistono diversi approcci con livelli di complessità crescenti:</span><span class="sxs-lookup"><span data-stu-id="c1d05-211">Several approaches exist with increasing levels of complexity:</span></span>

* <span data-ttu-id="c1d05-212">Archiviare lo stato della pagina corrente nell'archiviazione della sessione.</span><span class="sxs-lookup"><span data-stu-id="c1d05-212">Store the current page state in session storage.</span></span> <span data-ttu-id="c1d05-213">Durante l' [ `OnInitializedAsync` evento del ciclo](xref:blazor/components/lifecycle#component-initialization-methods) di vita ( <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> ), verificare se è possibile ripristinare lo stato prima di continuare.</span><span class="sxs-lookup"><span data-stu-id="c1d05-213">During the [`OnInitializedAsync` lifecycle event](xref:blazor/components/lifecycle#component-initialization-methods) (<xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A>), check if state can be restored before continuing.</span></span>
* <span data-ttu-id="c1d05-214">Aggiungere un parametro della stringa di query e usarlo come metodo per segnalare all'app che è necessario riattivare lo stato salvato in precedenza.</span><span class="sxs-lookup"><span data-stu-id="c1d05-214">Add a query string parameter and use that as a way to signal the app that it needs to re-hydrate the previously saved state.</span></span>
* <span data-ttu-id="c1d05-215">Aggiungere un parametro della stringa di query con un identificatore univoco per archiviare i dati nell'archiviazione della sessione senza rischiare conflitti con altri elementi.</span><span class="sxs-lookup"><span data-stu-id="c1d05-215">Add a query string parameter with a unique identifier to store data in session storage without risking collisions with other items.</span></span>

<span data-ttu-id="c1d05-216">L'esempio seguente illustra come:</span><span class="sxs-lookup"><span data-stu-id="c1d05-216">The following example shows how to:</span></span>

* <span data-ttu-id="c1d05-217">Mantenere lo stato prima di reindirizzare alla pagina di accesso.</span><span class="sxs-lookup"><span data-stu-id="c1d05-217">Preserve state before redirecting to the login page.</span></span>
* <span data-ttu-id="c1d05-218">Ripristinare lo stato precedente in seguito all'autenticazione utilizzando il parametro della stringa di query.</span><span class="sxs-lookup"><span data-stu-id="c1d05-218">Recover the previous state afterward authentication using the query string parameter.</span></span>

```razor
...
@using Microsoft.AspNetCore.Components.WebAssembly.Authentication
@inject IAccessTokenProvider TokenProvider
...

<EditForm Model="User" @onsubmit="OnSaveAsync">
    <label>User
        <InputText @bind-Value="User.Name" />
    </label>
    <label>Last name
        <InputText @bind-Value="User.LastName" />
    </label>
</EditForm>

@code {
    public class Profile
    {
        public string Name { get; set; }
        public string LastName { get; set; }
    }

    public Profile User { get; set; } = new Profile();

    protected async override Task OnInitializedAsync()
    {
        var currentQuery = new Uri(Navigation.Uri).Query;

        if (currentQuery.Contains("state=resumeSavingProfile"))
        {
            User = await JS.InvokeAsync<Profile>("sessionStorage.getState", 
                "resumeSavingProfile");
        }
    }

    public async Task OnSaveAsync()
    {
        var http = new HttpClient();
        http.BaseAddress = new Uri(Navigation.BaseUri);

        var resumeUri = Navigation.Uri + $"?state=resumeSavingProfile";

        var tokenResult = await TokenProvider.RequestAccessToken(
            new AccessTokenRequestOptions
            {
                ReturnUrl = resumeUri
            });

        if (tokenResult.TryGetToken(out var token))
        {
            http.DefaultRequestHeaders.Add("Authorization", 
                $"Bearer {token.Value}");
            await http.PostAsJsonAsync("Save", User);
        }
        else
        {
            await JS.InvokeVoidAsync("sessionStorage.setState", 
                "resumeSavingProfile", User);
            Navigation.NavigateTo(tokenResult.RedirectUrl);
        }
    }
}
```

## <a name="save-app-state-before-an-authentication-operation"></a><span data-ttu-id="c1d05-219">Salva lo stato dell'app prima di un'operazione di autenticazione</span><span class="sxs-lookup"><span data-stu-id="c1d05-219">Save app state before an authentication operation</span></span>

<span data-ttu-id="c1d05-220">Durante un'operazione di autenticazione, esistono casi in cui si vuole salvare lo stato dell'app prima che il browser venga reindirizzato all'indirizzo IP.</span><span class="sxs-lookup"><span data-stu-id="c1d05-220">During an authentication operation, there are cases where you want to save the app state before the browser is redirected to the IP.</span></span> <span data-ttu-id="c1d05-221">Questo può avvenire quando si usa un contenitore di stato e si vuole ripristinare lo stato dopo che l'autenticazione ha avuto esito positivo.</span><span class="sxs-lookup"><span data-stu-id="c1d05-221">This can be the case when you're using a state container and want to restore the state after the authentication succeeds.</span></span> <span data-ttu-id="c1d05-222">È possibile usare un oggetto stato di autenticazione personalizzato per mantenere lo stato specifico dell'app o un riferimento a esso e ripristinare lo stato dopo che l'operazione di autenticazione è stata completata correttamente.</span><span class="sxs-lookup"><span data-stu-id="c1d05-222">You can use a custom authentication state object to preserve app-specific state or a reference to it and restore that state after the authentication operation successfully completes.</span></span> <span data-ttu-id="c1d05-223">Nell'esempio seguente viene illustrato l'approccio.</span><span class="sxs-lookup"><span data-stu-id="c1d05-223">The following example demonstrates the approach.</span></span>

<span data-ttu-id="c1d05-224">Viene creata una classe contenitore di stato nell'app con proprietà che contengono i valori di stato dell'app.</span><span class="sxs-lookup"><span data-stu-id="c1d05-224">A state container class is created in the app with properties to hold the app's state values.</span></span> <span data-ttu-id="c1d05-225">Nell'esempio seguente il contenitore viene usato per gestire il valore del contatore del componente del modello di progetto predefinito `Counter` ( `Pages/Counter.razor` ).</span><span class="sxs-lookup"><span data-stu-id="c1d05-225">In the following example, the container is used to maintain the counter value of the default project template's `Counter` component (`Pages/Counter.razor`).</span></span> <span data-ttu-id="c1d05-226">I metodi per la serializzazione e la deserializzazione del contenitore sono basati su <xref:System.Text.Json> .</span><span class="sxs-lookup"><span data-stu-id="c1d05-226">Methods for serializing and deserializing the container are based on <xref:System.Text.Json>.</span></span>

```csharp
using System.Text.Json;

public class StateContainer
{
    public int CounterValue { get; set; }

    public string GetStateForLocalStorage()
    {
        return JsonSerializer.Serialize(this);
    }

    public void SetStateFromLocalStorage(string locallyStoredState)
    {
        var deserializedState = 
            JsonSerializer.Deserialize<StateContainer>(locallyStoredState);

        CounterValue = deserializedState.CounterValue;
    }
}
```

<span data-ttu-id="c1d05-227">Il `Counter` componente usa il contenitore di stato per mantenere il `currentCount` valore all'esterno del componente:</span><span class="sxs-lookup"><span data-stu-id="c1d05-227">The `Counter` component uses the state container to maintain the `currentCount` value outside of the component:</span></span>

```razor
@page "/counter"
@inject StateContainer State

<h1>Counter</h1>

<p>Current count: @currentCount</p>

<button class="btn btn-primary" @onclick="IncrementCount">Click me</button>

@code {
    private int currentCount = 0;

    protected override void OnInitialized()
    {
        if (State.CounterValue > 0)
        {
            currentCount = State.CounterValue;
        }
    }

    private void IncrementCount()
    {
        currentCount++;
        State.CounterValue = currentCount;
    }
}
```

<span data-ttu-id="c1d05-228">Creare un `ApplicationAuthenticationState` da <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteAuthenticationState> .</span><span class="sxs-lookup"><span data-stu-id="c1d05-228">Create an `ApplicationAuthenticationState` from <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteAuthenticationState>.</span></span> <span data-ttu-id="c1d05-229">Fornire una `Id` proprietà, che funge da identificatore per lo stato archiviato localmente.</span><span class="sxs-lookup"><span data-stu-id="c1d05-229">Provide an `Id` property, which serves as an identifier for the locally-stored state.</span></span>

<span data-ttu-id="c1d05-230">`ApplicationAuthenticationState.cs`:</span><span class="sxs-lookup"><span data-stu-id="c1d05-230">`ApplicationAuthenticationState.cs`:</span></span>

```csharp
using Microsoft.AspNetCore.Components.WebAssembly.Authentication;

public class ApplicationAuthenticationState : RemoteAuthenticationState
{
    public string Id { get; set; }
}
```

<span data-ttu-id="c1d05-231">Il `Authentication` componente ( `Pages/Authentication.razor` ) Salva e ripristina lo stato dell'app usando l'archiviazione di sessione locale con i `StateContainer` metodi di serializzazione e deserializzazione `GetStateForLocalStorage` e `SetStateFromLocalStorage` :</span><span class="sxs-lookup"><span data-stu-id="c1d05-231">The `Authentication` component (`Pages/Authentication.razor`) saves and restores the app's state using local session storage with the `StateContainer` serialization and deserialization methods, `GetStateForLocalStorage` and `SetStateFromLocalStorage`:</span></span>

```razor
@page "/authentication/{action}"
@inject IJSRuntime JS
@inject StateContainer State
@using Microsoft.AspNetCore.Components.WebAssembly.Authentication

<RemoteAuthenticatorViewCore Action="@Action"
                             TAuthenticationState="ApplicationAuthenticationState"
                             AuthenticationState="AuthenticationState"
                             OnLogInSucceeded="RestoreState"
                             OnLogOutSucceeded="RestoreState" />

@code {
    [Parameter]
    public string Action { get; set; }

    public ApplicationAuthenticationState AuthenticationState { get; set; } =
        new ApplicationAuthenticationState();

    protected async override Task OnInitializedAsync()
    {
        if (RemoteAuthenticationActions.IsAction(RemoteAuthenticationActions.LogIn,
            Action) ||
            RemoteAuthenticationActions.IsAction(RemoteAuthenticationActions.LogOut,
            Action))
        {
            AuthenticationState.Id = Guid.NewGuid().ToString();

            await JS.InvokeVoidAsync("sessionStorage.setItem",
                AuthenticationState.Id, State.GetStateForLocalStorage());
        }
    }

    private async Task RestoreState(ApplicationAuthenticationState state)
    {
        if (state.Id != null)
        {
            var locallyStoredState = await JS.InvokeAsync<string>(
                "sessionStorage.getItem", state.Id);

            if (locallyStoredState != null)
            {
                State.SetStateFromLocalStorage(locallyStoredState);
                await JS.InvokeVoidAsync("sessionStorage.removeItem", state.Id);
            }
        }
    }
}
```

<span data-ttu-id="c1d05-232">Questo esempio USA Azure Active Directory (AAD) per l'autenticazione.</span><span class="sxs-lookup"><span data-stu-id="c1d05-232">This example uses Azure Active Directory (AAD) for authentication.</span></span> <span data-ttu-id="c1d05-233">In `Program.Main` ( `Program.cs` ):</span><span class="sxs-lookup"><span data-stu-id="c1d05-233">In `Program.Main` (`Program.cs`):</span></span>

* <span data-ttu-id="c1d05-234">`ApplicationAuthenticationState`Viene configurato come tipo di libreria Microsoft autenticazione (MSAL) `RemoteAuthenticationState` .</span><span class="sxs-lookup"><span data-stu-id="c1d05-234">The `ApplicationAuthenticationState` is configured as the Microsoft Autentication Library (MSAL) `RemoteAuthenticationState` type.</span></span>
* <span data-ttu-id="c1d05-235">Il contenitore di stato è registrato nel contenitore dei servizi.</span><span class="sxs-lookup"><span data-stu-id="c1d05-235">The state container is registered in the service container.</span></span>

```csharp
builder.Services.AddMsalAuthentication<ApplicationAuthenticationState>(options =>
{
    builder.Configuration.Bind("AzureAd", options.ProviderOptions.Authentication);
});

builder.Services.AddSingleton<StateContainer>();
```

## <a name="customize-app-routes"></a><span data-ttu-id="c1d05-236">Personalizzare le route delle app</span><span class="sxs-lookup"><span data-stu-id="c1d05-236">Customize app routes</span></span>

<span data-ttu-id="c1d05-237">Per impostazione predefinita, la [`Microsoft.AspNetCore.Components.WebAssembly.Authentication`](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.WebAssembly.Authentication) libreria usa le route mostrate nella tabella seguente per la rappresentazione di Stati di autenticazione diversi.</span><span class="sxs-lookup"><span data-stu-id="c1d05-237">By default, the [`Microsoft.AspNetCore.Components.WebAssembly.Authentication`](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.WebAssembly.Authentication) library uses the routes shown in the following table for representing different authentication states.</span></span>

| <span data-ttu-id="c1d05-238">Route</span><span class="sxs-lookup"><span data-stu-id="c1d05-238">Route</span></span>                            | <span data-ttu-id="c1d05-239">Scopo</span><span class="sxs-lookup"><span data-stu-id="c1d05-239">Purpose</span></span> |
| -------------------------------- | ------- |
| `authentication/login`           | <span data-ttu-id="c1d05-240">Attiva un'operazione di accesso.</span><span class="sxs-lookup"><span data-stu-id="c1d05-240">Triggers a sign-in operation.</span></span> |
| `authentication/login-callback`  | <span data-ttu-id="c1d05-241">Gestisce il risultato di qualsiasi operazione di accesso.</span><span class="sxs-lookup"><span data-stu-id="c1d05-241">Handles the result of any sign-in operation.</span></span> |
| `authentication/login-failed`    | <span data-ttu-id="c1d05-242">Visualizza i messaggi di errore quando l'operazione di accesso non riesce per qualche motivo.</span><span class="sxs-lookup"><span data-stu-id="c1d05-242">Displays error messages when the sign-in operation fails for some reason.</span></span> |
| `authentication/logout`          | <span data-ttu-id="c1d05-243">Attiva un'operazione di disconnessione.</span><span class="sxs-lookup"><span data-stu-id="c1d05-243">Triggers a sign-out operation.</span></span> |
| `authentication/logout-callback` | <span data-ttu-id="c1d05-244">Gestisce il risultato di un'operazione di disconnessione.</span><span class="sxs-lookup"><span data-stu-id="c1d05-244">Handles the result of a sign-out operation.</span></span> |
| `authentication/logout-failed`   | <span data-ttu-id="c1d05-245">Visualizza i messaggi di errore quando l'operazione di disconnessione non riesce per qualche motivo.</span><span class="sxs-lookup"><span data-stu-id="c1d05-245">Displays error messages when the sign-out operation fails for some reason.</span></span> |
| `authentication/logged-out`      | <span data-ttu-id="c1d05-246">Indica che l'utente ha eseguito correttamente la disconnessione.</span><span class="sxs-lookup"><span data-stu-id="c1d05-246">Indicates that the user has successfully logout.</span></span> |
| `authentication/profile`         | <span data-ttu-id="c1d05-247">Attiva un'operazione per modificare il profilo utente.</span><span class="sxs-lookup"><span data-stu-id="c1d05-247">Triggers an operation to edit the user profile.</span></span> |
| `authentication/register`        | <span data-ttu-id="c1d05-248">Attiva un'operazione per registrare un nuovo utente.</span><span class="sxs-lookup"><span data-stu-id="c1d05-248">Triggers an operation to register a new user.</span></span> |

<span data-ttu-id="c1d05-249">Le route visualizzate nella tabella precedente sono configurabili tramite <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteAuthenticationOptions%601.AuthenticationPaths%2A?displayProperty=nameWithType> .</span><span class="sxs-lookup"><span data-stu-id="c1d05-249">The routes shown in the preceding table are configurable via <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteAuthenticationOptions%601.AuthenticationPaths%2A?displayProperty=nameWithType>.</span></span> <span data-ttu-id="c1d05-250">Quando si impostano le opzioni per fornire route personalizzate, verificare che l'app disponga di una route che gestisce ogni percorso.</span><span class="sxs-lookup"><span data-stu-id="c1d05-250">When setting options to provide custom routes, confirm that the app has a route that handles each path.</span></span>

<span data-ttu-id="c1d05-251">Nell'esempio seguente, tutti i percorsi hanno il prefisso `/security` .</span><span class="sxs-lookup"><span data-stu-id="c1d05-251">In the following example, all the paths are prefixed with `/security`.</span></span>

<span data-ttu-id="c1d05-252">`Authentication` componente ( `Pages/Authentication.razor` ):</span><span class="sxs-lookup"><span data-stu-id="c1d05-252">`Authentication` component (`Pages/Authentication.razor`):</span></span>

```razor
@page "/security/{action}"
@using Microsoft.AspNetCore.Components.WebAssembly.Authentication

<RemoteAuthenticatorView Action="@Action" />

@code{
    [Parameter]
    public string Action { get; set; }
}
```

<span data-ttu-id="c1d05-253">`Program.Main` (`Program.cs`):</span><span class="sxs-lookup"><span data-stu-id="c1d05-253">`Program.Main` (`Program.cs`):</span></span>

```csharp
builder.Services.AddApiAuthorization(options => { 
    options.AuthenticationPaths.LogInPath = "security/login";
    options.AuthenticationPaths.LogInCallbackPath = "security/login-callback";
    options.AuthenticationPaths.LogInFailedPath = "security/login-failed";
    options.AuthenticationPaths.LogOutPath = "security/logout";
    options.AuthenticationPaths.LogOutCallbackPath = "security/logout-callback";
    options.AuthenticationPaths.LogOutFailedPath = "security/logout-failed";
    options.AuthenticationPaths.LogOutSucceededPath = "security/logged-out";
    options.AuthenticationPaths.ProfilePath = "security/profile";
    options.AuthenticationPaths.RegisterPath = "security/register";
});
```

<span data-ttu-id="c1d05-254">Se il requisito richiede percorsi completamente diversi, impostare le route come descritto in precedenza ed eseguire il rendering <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteAuthenticatorView> di con un parametro di azione esplicito:</span><span class="sxs-lookup"><span data-stu-id="c1d05-254">If the requirement calls for completely different paths, set the routes as described previously and render the <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteAuthenticatorView> with an explicit action parameter:</span></span>

```razor
@page "/register"

<RemoteAuthenticatorView Action="@RemoteAuthenticationActions.Register" />
```

<span data-ttu-id="c1d05-255">Se si sceglie di eseguire questa operazione, è possibile suddividere l'interfaccia utente in pagine diverse.</span><span class="sxs-lookup"><span data-stu-id="c1d05-255">You're allowed to break the UI into different pages if you choose to do so.</span></span>

## <a name="customize-the-authentication-user-interface"></a><span data-ttu-id="c1d05-256">Personalizzare l'interfaccia utente di autenticazione</span><span class="sxs-lookup"><span data-stu-id="c1d05-256">Customize the authentication user interface</span></span>

<span data-ttu-id="c1d05-257"><xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteAuthenticatorView> include un set predefinito di elementi dell'interfaccia utente per ogni stato di autenticazione.</span><span class="sxs-lookup"><span data-stu-id="c1d05-257"><xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteAuthenticatorView> includes a default set of UI pieces for each authentication state.</span></span> <span data-ttu-id="c1d05-258">Ogni stato può essere personalizzato passando un oggetto personalizzato <xref:Microsoft.AspNetCore.Components.RenderFragment> .</span><span class="sxs-lookup"><span data-stu-id="c1d05-258">Each state can be customized by passing in a custom <xref:Microsoft.AspNetCore.Components.RenderFragment>.</span></span> <span data-ttu-id="c1d05-259">Per personalizzare il testo visualizzato durante il processo di accesso iniziale, può modificare il <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteAuthenticatorView> come indicato di seguito.</span><span class="sxs-lookup"><span data-stu-id="c1d05-259">To customize the displayed text during the initial login process, can change the <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteAuthenticatorView> as follows.</span></span>

<span data-ttu-id="c1d05-260">`Authentication` componente ( `Pages/Authentication.razor` ):</span><span class="sxs-lookup"><span data-stu-id="c1d05-260">`Authentication` component (`Pages/Authentication.razor`):</span></span>

```razor
@page "/security/{action}"
@using Microsoft.AspNetCore.Components.WebAssembly.Authentication

<RemoteAuthenticatorView Action="@Action">
    <LoggingIn>
        You are about to be redirected to https://login.microsoftonline.com.
    </LoggingIn>
</RemoteAuthenticatorView>

@code{
    [Parameter]
    public string Action { get; set; }
}
```

<span data-ttu-id="c1d05-261"><xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteAuthenticatorView>Dispone di un frammento che può essere utilizzato per ogni route di autenticazione illustrato nella tabella seguente.</span><span class="sxs-lookup"><span data-stu-id="c1d05-261">The <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteAuthenticatorView> has one fragment that can be used per authentication route shown in the following table.</span></span>

| <span data-ttu-id="c1d05-262">Route</span><span class="sxs-lookup"><span data-stu-id="c1d05-262">Route</span></span>                            | <span data-ttu-id="c1d05-263">Frammento</span><span class="sxs-lookup"><span data-stu-id="c1d05-263">Fragment</span></span>                |
| -------------------------------- | ----------------------- |
| `authentication/login`           | `<LoggingIn>`           |
| `authentication/login-callback`  | `<CompletingLoggingIn>` |
| `authentication/login-failed`    | `<LogInFailed>`         |
| `authentication/logout`          | `<LogOut>`              |
| `authentication/logout-callback` | `<CompletingLogOut>`    |
| `authentication/logout-failed`   | `<LogOutFailed>`        |
| `authentication/logged-out`      | `<LogOutSucceeded>`     |
| `authentication/profile`         | `<UserProfile>`         |
| `authentication/register`        | `<Registering>`         |

## <a name="customize-the-user"></a><span data-ttu-id="c1d05-264">Personalizzare l'utente</span><span class="sxs-lookup"><span data-stu-id="c1d05-264">Customize the user</span></span>

<span data-ttu-id="c1d05-265">Gli utenti associati all'app possono essere personalizzati.</span><span class="sxs-lookup"><span data-stu-id="c1d05-265">Users bound to the app can be customized.</span></span>

### <a name="customize-the-user-with-a-payload-claim"></a><span data-ttu-id="c1d05-266">Personalizzare l'utente con una richiesta di payload</span><span class="sxs-lookup"><span data-stu-id="c1d05-266">Customize the user with a payload claim</span></span>

<span data-ttu-id="c1d05-267">Nell'esempio seguente, gli utenti autenticati dell'app ricevono un' `amr` attestazione per ognuno dei metodi di autenticazione dell'utente.</span><span class="sxs-lookup"><span data-stu-id="c1d05-267">In the following example, the app's authenticated users receive an `amr` claim for each of the user's authentication methods.</span></span> <span data-ttu-id="c1d05-268">L' `amr` attestazione identifica la modalità di autenticazione dell'oggetto del token nelle Identity [attestazioni del payload](/azure/active-directory/develop/access-tokens#the-amr-claim)Microsoft Platform v 1.0.</span><span class="sxs-lookup"><span data-stu-id="c1d05-268">The `amr` claim identifies how the subject of the token was authenticated in Microsoft Identity Platform v1.0 [payload claims](/azure/active-directory/develop/access-tokens#the-amr-claim).</span></span> <span data-ttu-id="c1d05-269">Nell'esempio viene utilizzata una classe di account utente personalizzata basata su <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteUserAccount> .</span><span class="sxs-lookup"><span data-stu-id="c1d05-269">The example uses a custom user account class based on <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteUserAccount>.</span></span>

<span data-ttu-id="c1d05-270">Creare una classe che estende la <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteUserAccount> classe.</span><span class="sxs-lookup"><span data-stu-id="c1d05-270">Create a class that extends the <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteUserAccount> class.</span></span> <span data-ttu-id="c1d05-271">Nell'esempio seguente la proprietà viene impostata sulla `AuthenticationMethod` matrice di valori di `amr` Proprietà JSON dell'utente.</span><span class="sxs-lookup"><span data-stu-id="c1d05-271">The following example sets the `AuthenticationMethod` property to the user's array of `amr` JSON property values.</span></span> <span data-ttu-id="c1d05-272">`AuthenticationMethod` viene popolato automaticamente dal framework quando l'utente viene autenticato.</span><span class="sxs-lookup"><span data-stu-id="c1d05-272">`AuthenticationMethod` is populated automatically by the framework when the user is authenticated.</span></span>

```csharp
using System.Text.Json.Serialization;
using Microsoft.AspNetCore.Components.WebAssembly.Authentication;

public class CustomUserAccount : RemoteUserAccount
{
    [JsonPropertyName("amr")]
    public string[] AuthenticationMethod { get; set; }
}
```

<span data-ttu-id="c1d05-273">Creare una factory che <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AccountClaimsPrincipalFactory%601> si estende per creare attestazioni dai metodi di autenticazione dell'utente archiviati in `CustomUserAccount.AuthenticationMethod` :</span><span class="sxs-lookup"><span data-stu-id="c1d05-273">Create a factory that extends <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AccountClaimsPrincipalFactory%601> to create claims from the user's authentication methods stored in `CustomUserAccount.AuthenticationMethod`:</span></span>

```csharp
using System.Security.Claims;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Components;
using Microsoft.AspNetCore.Components.WebAssembly.Authentication;
using Microsoft.AspNetCore.Components.WebAssembly.Authentication.Internal;

public class CustomAccountFactory 
    : AccountClaimsPrincipalFactory<CustomUserAccount>
{
    public CustomAccountFactory(NavigationManager navigationManager, 
        IAccessTokenProviderAccessor accessor) : base(accessor)
    {
    }
  
    public async override ValueTask<ClaimsPrincipal> CreateUserAsync(
        CustomUserAccount account, RemoteAuthenticationUserOptions options)
    {
        var initialUser = await base.CreateUserAsync(account, options);

        if (initialUser.Identity.IsAuthenticated)
        {
            foreach (var value in account.AuthenticationMethod)
            {
                ((ClaimsIdentity)initialUser.Identity)
                    .AddClaim(new Claim("amr", value));
            }
        }

        return initialUser;
    }
}
```

<span data-ttu-id="c1d05-274">Registrare `CustomAccountFactory` per il provider di autenticazione in uso.</span><span class="sxs-lookup"><span data-stu-id="c1d05-274">Register the `CustomAccountFactory` for the authentication provider in use.</span></span> <span data-ttu-id="c1d05-275">Una delle seguenti registrazioni è valida:</span><span class="sxs-lookup"><span data-stu-id="c1d05-275">Any of the following registrations are valid:</span></span>

* <span data-ttu-id="c1d05-276"><xref:Microsoft.Extensions.DependencyInjection.WebAssemblyAuthenticationServiceCollectionExtensions.AddOidcAuthentication%2A>:</span><span class="sxs-lookup"><span data-stu-id="c1d05-276"><xref:Microsoft.Extensions.DependencyInjection.WebAssemblyAuthenticationServiceCollectionExtensions.AddOidcAuthentication%2A>:</span></span>

  ```csharp
  using Microsoft.AspNetCore.Components.WebAssembly.Authentication;

  ...

  builder.Services.AddOidcAuthentication<RemoteAuthenticationState, 
      CustomUserAccount>(options =>
      {
          ...
      })
      .AddAccountClaimsPrincipalFactory<RemoteAuthenticationState, 
          CustomUserAccount, CustomAccountFactory>();
  ```

* <span data-ttu-id="c1d05-277"><xref:Microsoft.Extensions.DependencyInjection.MsalWebAssemblyServiceCollectionExtensions.AddMsalAuthentication%2A>:</span><span class="sxs-lookup"><span data-stu-id="c1d05-277"><xref:Microsoft.Extensions.DependencyInjection.MsalWebAssemblyServiceCollectionExtensions.AddMsalAuthentication%2A>:</span></span>

  ```csharp
  using Microsoft.AspNetCore.Components.WebAssembly.Authentication;

  ...

  builder.Services.AddMsalAuthentication<RemoteAuthenticationState, 
      CustomUserAccount>(options =>
      {
          ...
      })
      .AddAccountClaimsPrincipalFactory<RemoteAuthenticationState, 
          CustomUserAccount, CustomAccountFactory>();
  ```
  
* <span data-ttu-id="c1d05-278"><xref:Microsoft.Extensions.DependencyInjection.WebAssemblyAuthenticationServiceCollectionExtensions.AddApiAuthorization%2A>:</span><span class="sxs-lookup"><span data-stu-id="c1d05-278"><xref:Microsoft.Extensions.DependencyInjection.WebAssemblyAuthenticationServiceCollectionExtensions.AddApiAuthorization%2A>:</span></span>

  ```csharp
  using Microsoft.AspNetCore.Components.WebAssembly.Authentication;

  ...

  builder.Services.AddApiAuthorization<RemoteAuthenticationState, 
      CustomUserAccount>(options =>
      {
          ...
      })
      .AddAccountClaimsPrincipalFactory<RemoteAuthenticationState, 
          CustomUserAccount, CustomAccountFactory>();
  ```

### <a name="aad-security-groups-and-roles-with-a-custom-user-account-class"></a><span data-ttu-id="c1d05-279">Gruppi di sicurezza e ruoli di AAD con una classe di account utente personalizzata</span><span class="sxs-lookup"><span data-stu-id="c1d05-279">AAD security groups and roles with a custom user account class</span></span>

<span data-ttu-id="c1d05-280">Per un esempio aggiuntivo che funziona con i gruppi di sicurezza AAD e i ruoli di amministratore di AAD e una classe di account utente personalizzata, vedere <xref:blazor/security/webassembly/aad-groups-roles> .</span><span class="sxs-lookup"><span data-stu-id="c1d05-280">For an additional example that works with AAD security groups and AAD Administrator Roles and a custom user account class, see <xref:blazor/security/webassembly/aad-groups-roles>.</span></span>

## <a name="support-prerendering-with-authentication"></a><span data-ttu-id="c1d05-281">Supporto del prerendering con autenticazione</span><span class="sxs-lookup"><span data-stu-id="c1d05-281">Support prerendering with authentication</span></span>

<span data-ttu-id="c1d05-282">Dopo aver seguito le indicazioni in uno degli argomenti dell'app ospitata Blazor WebAssembly , usare le istruzioni seguenti per creare un'app che:</span><span class="sxs-lookup"><span data-stu-id="c1d05-282">After following the guidance in one of the hosted Blazor WebAssembly app topics, use the following instructions to create an app that:</span></span>

* <span data-ttu-id="c1d05-283">Esegue il prerendering dei percorsi per i quali non è necessaria l'autorizzazione.</span><span class="sxs-lookup"><span data-stu-id="c1d05-283">Prerenders paths for which authorization isn't required.</span></span>
* <span data-ttu-id="c1d05-284">Non esegue il prerendering dei percorsi per cui è richiesta l'autorizzazione.</span><span class="sxs-lookup"><span data-stu-id="c1d05-284">Doesn't prerender paths for which authorization is required.</span></span>

<span data-ttu-id="c1d05-285">Nella *`Client`* `Program` classe () dell'app `Program.cs` , scomporre le registrazioni del servizio comuni in un metodo separato, ad esempio `ConfigureCommonServices` :</span><span class="sxs-lookup"><span data-stu-id="c1d05-285">In the *`Client`* app's `Program` class (`Program.cs`), factor common service registrations into a separate method (for example, `ConfigureCommonServices`):</span></span>

```csharp
public class Program
{
    public static async Task Main(string[] args)
    {
        var builder = WebAssemblyHostBuilder.CreateDefault(args);
        builder.RootComponents.Add...;

        builder.Services.AddScoped( ... );

        services.Add...;

        ConfigureCommonServices(builder.Services);

        await builder.Build().RunAsync();
    }

    public static void ConfigureCommonServices(IServiceCollection services)
    {
        // Common service registrations
    }
}
```

<span data-ttu-id="c1d05-286">Nell'app Server `Startup.ConfigureServices` registrare i servizi aggiuntivi seguenti:</span><span class="sxs-lookup"><span data-stu-id="c1d05-286">In the server app's `Startup.ConfigureServices`, register the following additional services:</span></span>

```csharp
using Microsoft.AspNetCore.Components.Authorization;
using Microsoft.AspNetCore.Components.Server;
using Microsoft.AspNetCore.Components.WebAssembly.Authentication;

public void ConfigureServices(IServiceCollection services)
{
    ...

    services.AddRazorPages();
    services.AddScoped<AuthenticationStateProvider, 
        ServerAuthenticationStateProvider>();
    services.AddScoped<SignOutSessionStateManager>();

    Client.Program.ConfigureCommonServices(services);
}
```

<span data-ttu-id="c1d05-287">Nel metodo dell'app Server `Startup.Configure` sostituire [`endpoints.MapFallbackToFile("index.html")`](xref:Microsoft.AspNetCore.Builder.StaticFilesEndpointRouteBuilderExtensions.MapFallbackToFile%2A) con [`endpoints.MapFallbackToPage("/_Host")`](xref:Microsoft.AspNetCore.Builder.RazorPagesEndpointRouteBuilderExtensions.MapFallbackToPage%2A) :</span><span class="sxs-lookup"><span data-stu-id="c1d05-287">In the server app's `Startup.Configure` method, replace [`endpoints.MapFallbackToFile("index.html")`](xref:Microsoft.AspNetCore.Builder.StaticFilesEndpointRouteBuilderExtensions.MapFallbackToFile%2A) with [`endpoints.MapFallbackToPage("/_Host")`](xref:Microsoft.AspNetCore.Builder.RazorPagesEndpointRouteBuilderExtensions.MapFallbackToPage%2A):</span></span>

```csharp
app.UseEndpoints(endpoints =>
{
    endpoints.MapControllers();
    endpoints.MapFallbackToPage("/_Host");
});
```

<span data-ttu-id="c1d05-288">Nell'app Server creare una `Pages` cartella se non esiste.</span><span class="sxs-lookup"><span data-stu-id="c1d05-288">In the server app, create a `Pages` folder if it doesn't exist.</span></span> <span data-ttu-id="c1d05-289">Creare una `_Host.cshtml` pagina all'interno della cartella dell'app Server `Pages` .</span><span class="sxs-lookup"><span data-stu-id="c1d05-289">Create a `_Host.cshtml` page inside the server app's `Pages` folder.</span></span> <span data-ttu-id="c1d05-290">Incollare il contenuto del *`Client`* file dell'app `wwwroot/index.html` nel `Pages/_Host.cshtml` file.</span><span class="sxs-lookup"><span data-stu-id="c1d05-290">Paste the contents from the *`Client`* app's `wwwroot/index.html` file into the `Pages/_Host.cshtml` file.</span></span> <span data-ttu-id="c1d05-291">Aggiornare il contenuto del file:</span><span class="sxs-lookup"><span data-stu-id="c1d05-291">Update the file's contents:</span></span>

::: moniker range=">= aspnetcore-5.0"

* <span data-ttu-id="c1d05-292">Aggiungere `@page "_Host"` all'inizio del file.</span><span class="sxs-lookup"><span data-stu-id="c1d05-292">Add `@page "_Host"` to the top of the file.</span></span>
* <span data-ttu-id="c1d05-293">Sostituire il `<div id="app">Loading...</div>` tag con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="c1d05-293">Replace the `<div id="app">Loading...</div>` tag with the following:</span></span>

  ```cshtml
  <div id="app">
      @if (!HttpContext.Request.Path.StartsWithSegments("/authentication"))
      {
          <component type="typeof({CLIENT APP ASSEMBLY NAME}.App)" 
              render-mode="Static" />
      }
      else
      {
          <text>Loading...</text>
      }
  </div>
  ```
  
  <span data-ttu-id="c1d05-294">Nell'esempio precedente, il segnaposto `{CLIENT APP ASSEMBLY NAME}` è il nome dell'assembly dell'app client (ad esempio `BlazorSample.Client` ).</span><span class="sxs-lookup"><span data-stu-id="c1d05-294">In the preceding example, the placeholder `{CLIENT APP ASSEMBLY NAME}` is the client app's assembly name (for example `BlazorSample.Client`).</span></span>

::: moniker-end

::: moniker range="< aspnetcore-5.0"

* <span data-ttu-id="c1d05-295">Aggiungere `@page "_Host"` all'inizio del file.</span><span class="sxs-lookup"><span data-stu-id="c1d05-295">Add `@page "_Host"` to the top of the file.</span></span>
* <span data-ttu-id="c1d05-296">Sostituire il `<app>Loading...</app>` tag con il codice seguente:</span><span class="sxs-lookup"><span data-stu-id="c1d05-296">Replace the `<app>Loading...</app>` tag with the following:</span></span>

  ```cshtml
  <app>
      @if (!HttpContext.Request.Path.StartsWithSegments("/authentication"))
      {
          <component type="typeof({CLIENT APP ASSEMBLY NAME}.App)" 
              render-mode="Static" />
      }
      else
      {
          <text>Loading...</text>
      }
  </app>
  ```
  
  <span data-ttu-id="c1d05-297">Nell'esempio precedente, il segnaposto `{CLIENT APP ASSEMBLY NAME}` è il nome dell'assembly dell'app client (ad esempio `BlazorSample.Client` ).</span><span class="sxs-lookup"><span data-stu-id="c1d05-297">In the preceding example, the placeholder `{CLIENT APP ASSEMBLY NAME}` is the client app's assembly name (for example `BlazorSample.Client`).</span></span>

::: moniker-end
  
## <a name="options-for-hosted-apps-and-third-party-login-providers"></a><span data-ttu-id="c1d05-298">Opzioni per le app ospitate e i provider di accesso di terze parti</span><span class="sxs-lookup"><span data-stu-id="c1d05-298">Options for hosted apps and third-party login providers</span></span>

<span data-ttu-id="c1d05-299">Quando si esegue l'autenticazione e l'autorizzazione di un'app ospitata Blazor WebAssembly con un provider di terze parti, sono disponibili diverse opzioni per l'autenticazione dell'utente.</span><span class="sxs-lookup"><span data-stu-id="c1d05-299">When authenticating and authorizing a hosted Blazor WebAssembly app with a third-party provider, there are several options available for authenticating the user.</span></span> <span data-ttu-id="c1d05-300">Quello scelto dipende dallo scenario.</span><span class="sxs-lookup"><span data-stu-id="c1d05-300">Which one you choose depends on your scenario.</span></span>

<span data-ttu-id="c1d05-301">Per altre informazioni, vedere <xref:security/authentication/social/additional-claims>.</span><span class="sxs-lookup"><span data-stu-id="c1d05-301">For more information, see <xref:security/authentication/social/additional-claims>.</span></span>

### <a name="authenticate-users-to-only-call-protected-third-party-apis"></a><span data-ttu-id="c1d05-302">Autenticare gli utenti per chiamare solo le API di terze parti protette</span><span class="sxs-lookup"><span data-stu-id="c1d05-302">Authenticate users to only call protected third party APIs</span></span>

<span data-ttu-id="c1d05-303">Autenticare l'utente con un flusso OAuth sul lato client rispetto al provider di API di terze parti:</span><span class="sxs-lookup"><span data-stu-id="c1d05-303">Authenticate the user with a client-side OAuth flow against the third-party API provider:</span></span>

 ```csharp
 builder.services.AddOidcAuthentication(options => { ... });
 ```
 
 <span data-ttu-id="c1d05-304">In questo scenario:</span><span class="sxs-lookup"><span data-stu-id="c1d05-304">In this scenario:</span></span>

* <span data-ttu-id="c1d05-305">Il server che ospita l'app non svolge un ruolo.</span><span class="sxs-lookup"><span data-stu-id="c1d05-305">The server hosting the app doesn't play a role.</span></span>
* <span data-ttu-id="c1d05-306">Le API nel server non possono essere protette.</span><span class="sxs-lookup"><span data-stu-id="c1d05-306">APIs on the server can't be protected.</span></span>
* <span data-ttu-id="c1d05-307">L'app può chiamare solo API di terze parti protette.</span><span class="sxs-lookup"><span data-stu-id="c1d05-307">The app can only call protected third-party APIs.</span></span>

### <a name="authenticate-users-with-a-third-party-provider-and-call-protected-apis-on-the-host-server-and-the-third-party"></a><span data-ttu-id="c1d05-308">Autenticare gli utenti con un provider di terze parti e chiamare le API protette sul server host e la terza parte</span><span class="sxs-lookup"><span data-stu-id="c1d05-308">Authenticate users with a third-party provider and call protected APIs on the host server and the third party</span></span>

<span data-ttu-id="c1d05-309">Configurare Identity con un provider di accesso di terze parti.</span><span class="sxs-lookup"><span data-stu-id="c1d05-309">Configure Identity with a third-party login provider.</span></span> <span data-ttu-id="c1d05-310">Ottenere i token necessari per l'accesso all'API di terze parti e archiviarli.</span><span class="sxs-lookup"><span data-stu-id="c1d05-310">Obtain the tokens required for third-party API access and store them.</span></span>

<span data-ttu-id="c1d05-311">Quando un utente esegue Identity l'accesso, raccoglie i token di accesso e di aggiornamento come parte del processo di autenticazione.</span><span class="sxs-lookup"><span data-stu-id="c1d05-311">When a user logs in, Identity collects access and refresh tokens as part of the authentication process.</span></span> <span data-ttu-id="c1d05-312">A questo punto, sono disponibili due approcci per eseguire chiamate API a API di terze parti.</span><span class="sxs-lookup"><span data-stu-id="c1d05-312">At that point, there are a couple of approaches available for making API calls to third-party APIs.</span></span>

#### <a name="use-a-server-access-token-to-retrieve-the-third-party-access-token"></a><span data-ttu-id="c1d05-313">Usare un token di accesso al server per recuperare il token di accesso di terze parti</span><span class="sxs-lookup"><span data-stu-id="c1d05-313">Use a server access token to retrieve the third-party access token</span></span>

<span data-ttu-id="c1d05-314">Usare il token di accesso generato sul server per recuperare il token di accesso di terze parti da un endpoint API server.</span><span class="sxs-lookup"><span data-stu-id="c1d05-314">Use the access token generated on the server to retrieve the third-party access token from a server API endpoint.</span></span> <span data-ttu-id="c1d05-315">Da qui, usare il token di accesso di terze parti per chiamare le risorse dell'API di terze parti direttamente da Identity nel client.</span><span class="sxs-lookup"><span data-stu-id="c1d05-315">From there, use the third-party access token to call third-party API resources directly from Identity on the client.</span></span>

<span data-ttu-id="c1d05-316">Questo approccio non è consigliato.</span><span class="sxs-lookup"><span data-stu-id="c1d05-316">We don't recommend this approach.</span></span> <span data-ttu-id="c1d05-317">Questo approccio richiede di trattare il token di accesso di terze parti come se fosse stato generato per un client pubblico.</span><span class="sxs-lookup"><span data-stu-id="c1d05-317">This approach requires treating the third-party access token as if it were generated for a public client.</span></span> <span data-ttu-id="c1d05-318">In termini di OAuth, l'app pubblica non ha un segreto client perché non può essere considerata attendibile per archiviare i segreti in modo sicuro e il token di accesso viene prodotto per un client riservato.</span><span class="sxs-lookup"><span data-stu-id="c1d05-318">In OAuth terms, the public app doesn't have a client secret because it can't be trusted to store secrets safely, and the access token is produced for a confidential client.</span></span> <span data-ttu-id="c1d05-319">Un client riservato è un client che dispone di un segreto client e si presuppone che sia in grado di archiviare in modo sicuro i segreti.</span><span class="sxs-lookup"><span data-stu-id="c1d05-319">A confidential client is a client that has a client secret and is assumed to be able to safely store secrets.</span></span>

* <span data-ttu-id="c1d05-320">Al token di accesso di terze parti potrebbero essere stati concessi ulteriori ambiti per eseguire operazioni sensibili in base al fatto che la terza parte ha emesso il token per un client più attendibile.</span><span class="sxs-lookup"><span data-stu-id="c1d05-320">The third-party access token might be granted additional scopes to perform sensitive operations based on the fact that the third-party emitted the token for a more trusted client.</span></span>
* <span data-ttu-id="c1d05-321">Analogamente, i token di aggiornamento non devono essere rilasciati a un client che non è considerato attendibile, in quanto in questo modo si ottiene l'accesso illimitato ai client a meno che non vengano applicate altre restrizioni.</span><span class="sxs-lookup"><span data-stu-id="c1d05-321">Similarly, refresh tokens shouldn't be issued to a client that isn't trusted, as doing so gives the client unlimited access unless other restrictions are put into place.</span></span>

#### <a name="make-api-calls-from-the-client-to-the-server-api-in-order-to-call-third-party-apis"></a><span data-ttu-id="c1d05-322">Eseguire chiamate API dal client all'API server per chiamare le API di terze parti</span><span class="sxs-lookup"><span data-stu-id="c1d05-322">Make API calls from the client to the server API in order to call third-party APIs</span></span>

<span data-ttu-id="c1d05-323">Eseguire una chiamata API dal client all'API del server.</span><span class="sxs-lookup"><span data-stu-id="c1d05-323">Make an API call from the client to the server API.</span></span> <span data-ttu-id="c1d05-324">Dal server recuperare il token di accesso per la risorsa API di terze parti ed emettere qualsiasi chiamata necessaria.</span><span class="sxs-lookup"><span data-stu-id="c1d05-324">From the server, retrieve the access token for the third-party API resource and issue whatever call is necessary.</span></span>

<span data-ttu-id="c1d05-325">Sebbene questo approccio richieda un hop di rete aggiuntivo tramite il server per la chiamata di un'API di terze parti, in definitiva si ottiene un'esperienza più sicura:</span><span class="sxs-lookup"><span data-stu-id="c1d05-325">While this approach requires an extra network hop through the server to call a third-party API, it ultimately results in a safer experience:</span></span>

* <span data-ttu-id="c1d05-326">Il server può archiviare i token di aggiornamento e assicurarsi che l'app non perda l'accesso alle risorse di terze parti.</span><span class="sxs-lookup"><span data-stu-id="c1d05-326">The server can store refresh tokens and ensure that the app doesn't lose access to third-party resources.</span></span>
* <span data-ttu-id="c1d05-327">L'app non può perdere i token di accesso dal server che potrebbe contenere autorizzazioni più sensibili.</span><span class="sxs-lookup"><span data-stu-id="c1d05-327">The app can't leak access tokens from the server that might contain more sensitive permissions.</span></span>

## <a name="use-openid-connect-oidc-v20-endpoints"></a><span data-ttu-id="c1d05-328">Usare gli endpoint OpenID Connect (OIDC) v 2.0</span><span class="sxs-lookup"><span data-stu-id="c1d05-328">Use OpenID Connect (OIDC) v2.0 endpoints</span></span>

<span data-ttu-id="c1d05-329">La libreria di autenticazione e i Blazor modelli di progetto usano gli endpoint OpenID Connect (OIDC) v 1.0.</span><span class="sxs-lookup"><span data-stu-id="c1d05-329">The authentication library and Blazor project templates use OpenID Connect (OIDC) v1.0 endpoints.</span></span> <span data-ttu-id="c1d05-330">Per usare un endpoint v 2.0, configurare l'opzione JWT Bearer <xref:Microsoft.AspNetCore.Builder.JwtBearerOptions.Authority?displayProperty=nameWithType> .</span><span class="sxs-lookup"><span data-stu-id="c1d05-330">To use a v2.0 endpoint, configure the JWT Bearer <xref:Microsoft.AspNetCore.Builder.JwtBearerOptions.Authority?displayProperty=nameWithType> option.</span></span> <span data-ttu-id="c1d05-331">Nell'esempio seguente AAD è configurato per la versione 2.0 aggiungendo un `v2.0` segmento alla <xref:Microsoft.AspNetCore.Builder.JwtBearerOptions.Authority> proprietà:</span><span class="sxs-lookup"><span data-stu-id="c1d05-331">In the following example, AAD is configured for v2.0 by appending a `v2.0` segment to the <xref:Microsoft.AspNetCore.Builder.JwtBearerOptions.Authority> property:</span></span>

```csharp
builder.Services.Configure<JwtBearerOptions>(
    AzureADDefaults.JwtBearerAuthenticationScheme, 
    options =>
    {
        options.Authority += "/v2.0";
    });
```

<span data-ttu-id="c1d05-332">In alternativa, l'impostazione può essere eseguita nel file delle impostazioni dell'app ( `appsettings.json` ):</span><span class="sxs-lookup"><span data-stu-id="c1d05-332">Alternatively, the setting can be made in the app settings (`appsettings.json`) file:</span></span>

```json
{
  "Local": {
    "Authority": "https://login.microsoftonline.com/common/oauth2/v2.0/",
    ...
  }
}
```

<span data-ttu-id="c1d05-333">Se l'uso di un segmento nell'autorità non è appropriato per il provider OIDC dell'app, ad esempio con i provider non AAD, impostare <xref:Microsoft.AspNetCore.Builder.OpenIdConnectOptions.Authority> direttamente la proprietà.</span><span class="sxs-lookup"><span data-stu-id="c1d05-333">If tacking on a segment to the authority isn't appropriate for the app's OIDC provider, such as with non-AAD providers, set the <xref:Microsoft.AspNetCore.Builder.OpenIdConnectOptions.Authority> property directly.</span></span> <span data-ttu-id="c1d05-334">Impostare la proprietà in <xref:Microsoft.AspNetCore.Builder.JwtBearerOptions> o nel file di impostazioni dell'app ( `appsettings.json` ) con la `Authority` chiave.</span><span class="sxs-lookup"><span data-stu-id="c1d05-334">Either set the property in <xref:Microsoft.AspNetCore.Builder.JwtBearerOptions> or in the app settings file (`appsettings.json`) with the `Authority` key.</span></span>

<span data-ttu-id="c1d05-335">L'elenco di attestazioni nel token ID cambia per gli endpoint della versione 2.0.</span><span class="sxs-lookup"><span data-stu-id="c1d05-335">The list of claims in the ID token changes for v2.0 endpoints.</span></span> <span data-ttu-id="c1d05-336">Per ulteriori informazioni, vedere la pagina relativa [all'aggiornamento alla piattaforma di identità Microsoft (v 2.0)](/azure/active-directory/azuread-dev/azure-ad-endpoint-comparison).</span><span class="sxs-lookup"><span data-stu-id="c1d05-336">For more information, see [Why update to Microsoft identity platform (v2.0)?](/azure/active-directory/azuread-dev/azure-ad-endpoint-comparison).</span></span>

## <a name="configure-and-use-grpc-in-components"></a><span data-ttu-id="c1d05-337">Configurare e usare gRPC nei componenti</span><span class="sxs-lookup"><span data-stu-id="c1d05-337">Configure and use gRPC in components</span></span>

<span data-ttu-id="c1d05-338">Per configurare un' Blazor WebAssembly app per l'uso del [framework ASP.NET Core gRPC](xref:grpc/index):</span><span class="sxs-lookup"><span data-stu-id="c1d05-338">To configure a Blazor WebAssembly app to use the [ASP.NET Core gRPC framework](xref:grpc/index):</span></span>

* <span data-ttu-id="c1d05-339">Abilitare gRPC-Web sul server.</span><span class="sxs-lookup"><span data-stu-id="c1d05-339">Enable gRPC-Web on the server.</span></span> <span data-ttu-id="c1d05-340">Per altre informazioni, vedere <xref:grpc/browser>.</span><span class="sxs-lookup"><span data-stu-id="c1d05-340">For more information, see <xref:grpc/browser>.</span></span>
* <span data-ttu-id="c1d05-341">Registrare i servizi di gRPC per il gestore di messaggi dell'app.</span><span class="sxs-lookup"><span data-stu-id="c1d05-341">Register gRPC services for the app's message handler.</span></span> <span data-ttu-id="c1d05-342">Nell'esempio seguente viene configurato il gestore dei messaggi di autorizzazione dell'app per usare il [ `GreeterClient` servizio dell'esercitazione su gRPC](xref:tutorials/grpc/grpc-start#create-a-grpc-service) ( `Program.Main` ):</span><span class="sxs-lookup"><span data-stu-id="c1d05-342">The following example configures the app's authorization message handler to use the [`GreeterClient` service from the gRPC tutorial](xref:tutorials/grpc/grpc-start#create-a-grpc-service) (`Program.Main`):</span></span>

```csharp
using System.Net.Http;
using Microsoft.AspNetCore.Components.WebAssembly.Authentication;
using Grpc.Net.Client;
using Grpc.Net.Client.Web;
using {APP ASSEMBLY}.Shared;

...

builder.Services.AddScoped(sp =>
{
    var baseAddressMessageHandler = 
        sp.GetRequiredService<BaseAddressAuthorizationMessageHandler>();
    baseAddressMessageHandler.InnerHandler = new HttpClientHandler();
    var grpcWebHandler = 
        new GrpcWebHandler(GrpcWebMode.GrpcWeb, baseAddressMessageHandler);
    var channel = GrpcChannel.ForAddress(builder.HostEnvironment.BaseAddress, 
        new GrpcChannelOptions { HttpHandler = grpcWebHandler });

    return new Greeter.GreeterClient(channel);
});
```

<span data-ttu-id="c1d05-343">Il segnaposto `{APP ASSEMBLY}` è il nome dell'assembly dell'app (ad esempio, `BlazorSample` ).</span><span class="sxs-lookup"><span data-stu-id="c1d05-343">The placeholder `{APP ASSEMBLY}` is the app's assembly name (for example, `BlazorSample`).</span></span> <span data-ttu-id="c1d05-344">Inserire il `.proto` file nel `Shared` progetto della soluzione ospitata Blazor .</span><span class="sxs-lookup"><span data-stu-id="c1d05-344">Place the `.proto` file in the `Shared` project of the hosted Blazor solution.</span></span>

<span data-ttu-id="c1d05-345">Un componente nell'app client può effettuare chiamate gRPC usando il client gRPC ( `Pages/Grpc.razor` ):</span><span class="sxs-lookup"><span data-stu-id="c1d05-345">A component in the client app can make gRPC calls using the gRPC client (`Pages/Grpc.razor`):</span></span>

```razor
@page "/grpc"
@attribute [Authorize]
@using Microsoft.AspNetCore.Authorization
@using {APP ASSEMBLY}.Shared
@inject Greeter.GreeterClient GreeterClient

<h1>Invoke gRPC service</h1>

<p>
    <input @bind="name" placeholder="Type your name" />
    <button @onclick="GetGreeting" class="btn btn-primary">Call gRPC service</button>
</p>

Server response: <strong>@serverResponse</strong>

@code {
    private string name = "Bert";
    private string serverResponse;

    private async Task GetGreeting()
    {
        try
        {
            var request = new HelloRequest { Name = name };
            var reply = await GreeterClient.SayHelloAsync(request);
            serverResponse = reply.Message;
        }
        catch (Grpc.Core.RpcException ex)
            when (ex.Status.DebugException is 
                AccessTokenNotAvailableException tokenEx)
        {
            tokenEx.Redirect();
        }
    }
}
```

<span data-ttu-id="c1d05-346">Il segnaposto `{APP ASSEMBLY}` è il nome dell'assembly dell'app (ad esempio, `BlazorSample` ).</span><span class="sxs-lookup"><span data-stu-id="c1d05-346">The placeholder `{APP ASSEMBLY}` is the app's assembly name (for example, `BlazorSample`).</span></span> <span data-ttu-id="c1d05-347">Per usare la `Status.DebugException` proprietà, usare [Grpc .NET. client](https://www.nuget.org/packages/Grpc.Net.Client) versione 2.30.0 o successiva.</span><span class="sxs-lookup"><span data-stu-id="c1d05-347">To use the `Status.DebugException` property, use [Grpc.Net.Client](https://www.nuget.org/packages/Grpc.Net.Client) version 2.30.0 or later.</span></span>

<span data-ttu-id="c1d05-348">Per altre informazioni, vedere <xref:grpc/browser>.</span><span class="sxs-lookup"><span data-stu-id="c1d05-348">For more information, see <xref:grpc/browser>.</span></span>

## <a name="build-a-custom-version-of-the-authenticationmsal-javascript-library"></a><span data-ttu-id="c1d05-349">Compilare una versione personalizzata della libreria JavaScript Authentication. MSAL</span><span class="sxs-lookup"><span data-stu-id="c1d05-349">Build a custom version of the Authentication.MSAL JavaScript library</span></span>

<span data-ttu-id="c1d05-350">Se un'app richiede una versione personalizzata di [Microsoft Authentication Library per JavaScript (MSAL.js)](https://www.npmjs.com/package/@azure/msal-browser), seguire questa procedura:</span><span class="sxs-lookup"><span data-stu-id="c1d05-350">If an app requires a custom version of the [Microsoft Authentication Library for JavaScript (MSAL.js)](https://www.npmjs.com/package/@azure/msal-browser), perform the following steps:</span></span>

1. <span data-ttu-id="c1d05-351">Verificare che il sistema disponga della versione più recente di Developer .NET SDK oppure ottenere e installare la versione più recente di Developer SDK da [.NET Core SDK: programmi di installazione e file binari](https://github.com/dotnet/installer#installers-and-binaries).</span><span class="sxs-lookup"><span data-stu-id="c1d05-351">Confirm the the system has the latest developer .NET SDK or obtain and install the latest developer SDK from [.NET Core SDK: Installers and Binaries](https://github.com/dotnet/installer#installers-and-binaries).</span></span> <span data-ttu-id="c1d05-352">La configurazione dei feed NuGet interni non è necessaria per questo scenario.</span><span class="sxs-lookup"><span data-stu-id="c1d05-352">Configuration of internal NuGet feeds isn't required for this scenario.</span></span>
1. <span data-ttu-id="c1d05-353">Configurare il `dotnet/aspnetcore` repository GitHub per lo sviluppo in base ai documenti in [Build ASP.NET Core dall'origine](https://github.com/dotnet/aspnetcore/blob/main/docs/BuildFromSource.md).</span><span class="sxs-lookup"><span data-stu-id="c1d05-353">Set up the `dotnet/aspnetcore` GitHub repository for development per the docs at [Build ASP.NET Core from Source](https://github.com/dotnet/aspnetcore/blob/main/docs/BuildFromSource.md).</span></span> <span data-ttu-id="c1d05-354">Creare un fork e clonare o scaricare un archivio ZIP del [repository GitHub DotNet/aspnetcore](https://github.com/dotnet/aspnetcore).</span><span class="sxs-lookup"><span data-stu-id="c1d05-354">Fork and clone or download a ZIP archive of the [dotnet/aspnetcore GitHub repository](https://github.com/dotnet/aspnetcore).</span></span>
1. <span data-ttu-id="c1d05-355">Aprire il `src/Components/WebAssembly/Authentication.Msal/src/Interop/package.json` file e impostare la versione desiderata di `@azure/msal-browser` .</span><span class="sxs-lookup"><span data-stu-id="c1d05-355">Open the the `src/Components/WebAssembly/Authentication.Msal/src/Interop/package.json` file and set the desired version of `@azure/msal-browser`.</span></span> <span data-ttu-id="c1d05-356">Per un elenco delle versioni rilasciate, visitare il [ `@azure/msal-browser` sito Web NPM](https://www.npmjs.com/package/@azure/msal-browser) e selezionare la scheda **versioni** .</span><span class="sxs-lookup"><span data-stu-id="c1d05-356">For a list of released versions, visit the [`@azure/msal-browser` npm website](https://www.npmjs.com/package/@azure/msal-browser) and select the **Versions** tab.</span></span>
1. <span data-ttu-id="c1d05-357">Compilare il `Authentication.Msal` progetto nella `src/Components/WebAssembly/Authentication.Msal/src` cartella con il `yarn build` comando in una shell dei comandi.</span><span class="sxs-lookup"><span data-stu-id="c1d05-357">Build the `Authentication.Msal` project in the `src/Components/WebAssembly/Authentication.Msal/src` folder with the `yarn build` command in a command shell.</span></span>
1. <span data-ttu-id="c1d05-358">Se l'app usa [Asset compressi (Brotli/gzip)](xref:blazor/host-and-deploy/webassembly#compression), comprimere il `Interop/dist/Release/AuthenticationService.js` file.</span><span class="sxs-lookup"><span data-stu-id="c1d05-358">If the app uses [compressed assets (Brotli/Gzip)](xref:blazor/host-and-deploy/webassembly#compression), compress the `Interop/dist/Release/AuthenticationService.js` file.</span></span>
1. <span data-ttu-id="c1d05-359">Copiare il `AuthenticationService.js` file e le versioni compresse ( `.br` / `.gz` ) del file, se generato, dalla `Interop/dist/Release` cartella alla cartella dell'app `publish/wwwroot/_content/Microsoft.Authentication.WebAssembly.Msal` negli asset pubblicati dell'app.</span><span class="sxs-lookup"><span data-stu-id="c1d05-359">Copy the `AuthenticationService.js` file and compressed versions (`.br`/`.gz`) of the file, if produced, from the `Interop/dist/Release` folder into the app's `publish/wwwroot/_content/Microsoft.Authentication.WebAssembly.Msal` folder in the app's published assets.</span></span>

## <a name="additional-resources"></a><span data-ttu-id="c1d05-360">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="c1d05-360">Additional resources</span></span>

* <xref:blazor/security/webassembly/graph-api>
* [<span data-ttu-id="c1d05-361">`HttpClient` e `HttpRequestMessage` con le opzioni di richiesta dell'API fetch</span><span class="sxs-lookup"><span data-stu-id="c1d05-361">`HttpClient` and `HttpRequestMessage` with Fetch API request options</span></span>](xref:blazor/call-web-api#httpclient-and-httprequestmessage-with-fetch-api-request-options)
