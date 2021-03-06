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
ms.openlocfilehash: c2b2a6f049846dc2e4894b4cab8b0a1aa34762f9
ms.sourcegitcommit: 1436bd4d70937d6ec3140da56d96caab33c4320b
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/06/2021
ms.locfileid: "102394850"
---
# <a name="aspnet-core-blazor-webassembly-additional-security-scenarios"></a>ASP.NET Core Blazor WebAssembly scenari di sicurezza aggiuntivi

## <a name="attach-tokens-to-outgoing-requests"></a>Connetti token alle richieste in uscita

<xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AuthorizationMessageHandler> è <xref:System.Net.Http.DelegatingHandler> usato per alleghire i token di accesso alle <xref:System.Net.Http.HttpResponseMessage> istanze in uscita. I token vengono acquisiti usando il <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.IAccessTokenProvider> servizio, registrato dal Framework. Se non è possibile acquisire un token, <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AccessTokenNotAvailableException> viene generata un'eccezione. <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AccessTokenNotAvailableException> dispone di un <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AccessTokenNotAvailableException.Redirect%2A> metodo che può essere utilizzato per passare all'utente al provider di identità per acquisire un nuovo token.

Per praticità, il Framework fornisce il <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.BaseAddressAuthorizationMessageHandler> preconfigurato con l'indirizzo di base dell'app come un URL autorizzato. **I token di accesso vengono aggiunti solo quando l'URI della richiesta si trova all'interno dell'URI di base dell'app.** Quando gli URI di richiesta in uscita non sono inclusi nell'URI di base dell'app, usare una [ `AuthorizationMessageHandler` classe personalizzata (*scelta consigliata*)](#custom-authorizationmessagehandler-class) o [configurare `AuthorizationMessageHandler`](#configure-authorizationmessagehandler).

> [!NOTE]
> Oltre alla configurazione dell'app client per l'accesso all'API server, l'API del server deve anche consentire le richieste tra le origini (CORS) quando il client e il server non si trovano allo stesso indirizzo di base. Per altre informazioni sulla configurazione CORS sul lato server, vedere la sezione [condivisione di risorse tra le origini (CORS)](#cross-origin-resource-sharing-cors) più avanti in questo articolo.

Nell'esempio seguente:

* <xref:Microsoft.Extensions.DependencyInjection.HttpClientFactoryServiceCollectionExtensions.AddHttpClient%2A> aggiunge <xref:System.Net.Http.IHttpClientFactory> e i servizi correlati alla raccolta di servizi e configura un oggetto denominato <xref:System.Net.Http.HttpClient> ( `ServerAPI` ). <xref:System.Net.Http.HttpClient.BaseAddress?displayProperty=nameWithType> è l'indirizzo di base dell'URI della risorsa quando si inviano le richieste. <xref:System.Net.Http.IHttpClientFactory> viene fornito dal [`Microsoft.Extensions.Http`](https://www.nuget.org/packages/Microsoft.Extensions.Http) pacchetto NuGet.
* <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.BaseAddressAuthorizationMessageHandler> è <xref:System.Net.Http.DelegatingHandler> usato per alleghire i token di accesso alle <xref:System.Net.Http.HttpResponseMessage> istanze in uscita. I token di accesso vengono aggiunti solo quando l'URI della richiesta si trova all'interno dell'URI di base dell'app.
* <xref:System.Net.Http.IHttpClientFactory.CreateClient%2A?displayProperty=nameWithType> Crea e configura un' <xref:System.Net.Http.HttpClient> istanza per le richieste in uscita usando la configurazione che corrisponde al denominato <xref:System.Net.Http.HttpClient> ( `ServerAPI` ).

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

Per una soluzione ospitata Blazor basata sul [ Blazor WebAssembly modello di progetto](xref:blazor/project-structure), gli URI di richiesta si trovano nell'URI di base dell'applicazione per impostazione predefinita. Viene pertanto <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.IWebAssemblyHostEnvironment.BaseAddress?displayProperty=nameWithType> `new Uri(builder.HostEnvironment.BaseAddress)` assegnato a <xref:System.Net.Http.HttpClient.BaseAddress?displayProperty=nameWithType> in un'app generata dal modello di progetto.

L'oggetto configurato <xref:System.Net.Http.HttpClient> viene usato per effettuare richieste autorizzate usando il [`try-catch`](/dotnet/csharp/language-reference/keywords/try-catch) modello:

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

### <a name="custom-authorizationmessagehandler-class"></a>`AuthorizationMessageHandler`Classe personalizzata

*Queste linee guida sono consigliate per le app client che effettuano richieste in uscita a URI che non rientrano nell'URI di base dell'app.*

Nell'esempio seguente, una classe personalizzata si estende <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AuthorizationMessageHandler> per l'utilizzo come <xref:System.Net.Http.DelegatingHandler> per un oggetto <xref:System.Net.Http.HttpClient> . <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AuthorizationMessageHandler.ConfigureHandler%2A> Configura questo gestore per autorizzare le richieste HTTP in uscita usando un token di accesso. Il token di accesso viene collegato solo se almeno uno degli URL autorizzati è una base dell'URI della richiesta ( <xref:System.Net.Http.HttpRequestMessage.RequestUri?displayProperty=nameWithType> ).

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

In `Program.Main` ( `Program.cs` ), `CustomAuthorizationMessageHandler` viene registrato come un servizio con ambito e viene configurato come <xref:System.Net.Http.DelegatingHandler> per le istanze in uscita <xref:System.Net.Http.HttpResponseMessage> effettuate da un oggetto denominato <xref:System.Net.Http.HttpClient> :

```csharp
builder.Services.AddScoped<CustomAuthorizationMessageHandler>();

builder.Services.AddHttpClient("ServerAPI",
        client => client.BaseAddress = new Uri("https://www.example.com/base"))
    .AddHttpMessageHandler<CustomAuthorizationMessageHandler>();
```

Per una soluzione ospitata Blazor basata sul [ Blazor WebAssembly modello di progetto](xref:blazor/project-structure), <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.IWebAssemblyHostEnvironment.BaseAddress?displayProperty=nameWithType> ( `new Uri(builder.HostEnvironment.BaseAddress)` ) viene assegnato a per <xref:System.Net.Http.HttpClient.BaseAddress?displayProperty=nameWithType> impostazione predefinita.

L'oggetto configurato <xref:System.Net.Http.HttpClient> viene usato per effettuare richieste autorizzate usando il [`try-catch`](/dotnet/csharp/language-reference/keywords/try-catch) modello. Se il client viene creato con <xref:System.Net.Http.IHttpClientFactory.CreateClient%2A> ( [`Microsoft.Extensions.Http`](https://www.nuget.org/packages/Microsoft.Extensions.Http) pacchetto), <xref:System.Net.Http.HttpClient> vengono fornite le istanze di che includono i token di accesso quando si effettuano richieste all'API del server. Se l'URI della richiesta è un URI relativo, come nell'esempio seguente ( `ExampleAPIMethod` ), viene combinato con l'oggetto <xref:System.Net.Http.HttpClient.BaseAddress> quando l'app client esegue la richiesta:

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

### <a name="configure-authorizationmessagehandler"></a>Configurare`AuthorizationMessageHandler`

<xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AuthorizationMessageHandler> può essere configurato con URL autorizzati, ambiti e un URL restituito usando il <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AuthorizationMessageHandler.ConfigureHandler%2A> metodo. <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AuthorizationMessageHandler.ConfigureHandler%2A> Configura il gestore per autorizzare le richieste HTTP in uscita usando un token di accesso. Il token di accesso viene collegato solo se almeno uno degli URL autorizzati è una base dell'URI della richiesta ( <xref:System.Net.Http.HttpRequestMessage.RequestUri?displayProperty=nameWithType> ). Se l'URI della richiesta è un URI relativo, viene combinato con <xref:System.Net.Http.HttpClient.BaseAddress> .

Nell'esempio seguente <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AuthorizationMessageHandler> configura un <xref:System.Net.Http.HttpClient> in `Program.Main` ( `Program.cs` ):

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

Per una soluzione ospitata Blazor basata sul [ Blazor WebAssembly modello di progetto](xref:blazor/project-structure), per <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.IWebAssemblyHostEnvironment.BaseAddress?displayProperty=nameWithType> impostazione predefinita viene assegnato a quanto segue:

* Oggetto <xref:System.Net.Http.HttpClient.BaseAddress?displayProperty=nameWithType> ( `new Uri(builder.HostEnvironment.BaseAddress)` ).
* URL della `authorizedUrls` matrice.

## <a name="typed-httpclient"></a>Tipizzato `HttpClient`

È possibile definire un client tipizzato che gestisce tutti i problemi di acquisizione di token e HTTP all'interno di una singola classe.

`WeatherForecastClient.cs`:

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

Il segnaposto `{APP ASSEMBLY}` è il nome dell'assembly dell'app (ad esempio, `using static BlazorSample.Data;` ).

`Program.Main` (`Program.cs`):

```csharp
using System.Net.Http;
using Microsoft.AspNetCore.Components.WebAssembly.Authentication;

...

builder.Services.AddHttpClient<WeatherForecastClient>(
        client => client.BaseAddress = new Uri("https://www.example.com/base"))
    .AddHttpMessageHandler<BaseAddressAuthorizationMessageHandler>();
```

Per una soluzione ospitata Blazor basata sul [ Blazor WebAssembly modello di progetto](xref:blazor/project-structure), <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.IWebAssemblyHostEnvironment.BaseAddress?displayProperty=nameWithType> ( `new Uri(builder.HostEnvironment.BaseAddress)` ) viene assegnato a per <xref:System.Net.Http.HttpClient.BaseAddress?displayProperty=nameWithType> impostazione predefinita.

`FetchData` componente ( `Pages/FetchData.razor` ):

```razor
@inject WeatherForecastClient Client

...

protected override async Task OnInitializedAsync()
{
    forecasts = await Client.GetForecastAsync();
}
```

## <a name="configure-the-httpclient-handler"></a>Configurare il `HttpClient` gestore

Il gestore può essere configurato ulteriormente con <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AuthorizationMessageHandler.ConfigureHandler%2A> per le richieste HTTP in uscita.

`Program.Main` (`Program.cs`):

```csharp
builder.Services.AddHttpClient<WeatherForecastClient>(
        client => client.BaseAddress = new Uri("https://www.example.com/base"))
    .AddHttpMessageHandler(sp => sp.GetRequiredService<AuthorizationMessageHandler>()
    .ConfigureHandler(
        authorizedUrls: new [] { "https://www.example.com/base" },
        scopes: new[] { "example.read", "example.write" }));
```

Per una soluzione ospitata Blazor basata sul [ Blazor WebAssembly modello di progetto](xref:blazor/project-structure), per <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.IWebAssemblyHostEnvironment.BaseAddress?displayProperty=nameWithType> impostazione predefinita viene assegnato a quanto segue:

* Oggetto <xref:System.Net.Http.HttpClient.BaseAddress?displayProperty=nameWithType> ( `new Uri(builder.HostEnvironment.BaseAddress)` ).
* URL della `authorizedUrls` matrice.

## <a name="unauthenticated-or-unauthorized-web-api-requests-in-an-app-with-a-secure-default-client"></a>Richieste API Web non autenticate o non autorizzate in un'app con un client predefinito sicuro

Se l' Blazor WebAssembly app usa normalmente un valore predefinito sicuro <xref:System.Net.Http.HttpClient> , l'app può anche eseguire richieste API Web non autenticate o non autorizzate configurando un oggetto denominato <xref:System.Net.Http.HttpClient> :

`Program.Main` (`Program.cs`):

```csharp
builder.Services.AddHttpClient("ServerAPI.NoAuthenticationClient", 
    client => client.BaseAddress = new Uri("https://www.example.com/base"));
```

Per una soluzione ospitata Blazor basata sul [ Blazor WebAssembly modello di progetto](xref:blazor/project-structure), <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.IWebAssemblyHostEnvironment.BaseAddress?displayProperty=nameWithType> ( `new Uri(builder.HostEnvironment.BaseAddress)` ) viene assegnato a per <xref:System.Net.Http.HttpClient.BaseAddress?displayProperty=nameWithType> impostazione predefinita.

La registrazione precedente è aggiunta alla registrazione predefinita protetta esistente <xref:System.Net.Http.HttpClient> .

Un componente crea <xref:System.Net.Http.HttpClient> dalla <xref:System.Net.Http.IHttpClientFactory> ( [`Microsoft.Extensions.Http`](https://www.nuget.org/packages/Microsoft.Extensions.Http) pacchetto) per eseguire richieste non autenticate o non autorizzate:

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
> Il controller nell'API del server, `WeatherForecastNoAuthenticationController` per l'esempio precedente, non è contrassegnato con l' [ `[Authorize]` attributo](xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute).

La decisione di usare un client protetto o un client non sicuro come <xref:System.Net.Http.HttpClient> istanza predefinita è spetta allo sviluppatore. Un modo per prendere questa decisione consiste nel considerare il numero di endpoint autenticati e non autenticati contattati dall'app. Se la maggior parte delle richieste dell'app è quella di proteggere gli endpoint dell'API, usare l' <xref:System.Net.Http.HttpClient> istanza autenticata come predefinita. In caso contrario, registrare l'istanza non autenticata <xref:System.Net.Http.HttpClient> come predefinita.

Un approccio alternativo all'utilizzo di <xref:System.Net.Http.IHttpClientFactory> consiste nel creare un [client tipizzato](#typed-httpclient) per l'accesso non autenticato agli endpoint anonimi.

## <a name="request-additional-access-tokens"></a>Richiedere token di accesso aggiuntivi

I token di accesso possono essere ottenuti manualmente chiamando `IAccessTokenProvider.RequestAccessToken` . Nell'esempio seguente, un ambito aggiuntivo è richiesto da un'app per il valore predefinito <xref:System.Net.Http.HttpClient> . Nell'esempio Microsoft Authentication Library (MSAL) viene configurato l'ambito con `MsalProviderOptions` :

`Program.Main` (`Program.cs`):

```csharp
builder.Services.AddMsalAuthentication(options =>
{
    ...

    options.ProviderOptions.AdditionalScopesToConsent.Add("{CUSTOM SCOPE 1}");
    options.ProviderOptions.AdditionalScopesToConsent.Add("{CUSTOM SCOPE 2}");
}
```

I `{CUSTOM SCOPE 1}` `{CUSTOM SCOPE 2}` segnaposto e nell'esempio precedente sono ambiti personalizzati.

Il `IAccessTokenProvider.RequestToken` metodo fornisce un overload che consente a un'app di effettuare il provisioning di un token di accesso con un determinato set di ambiti.

In un Razor componente:

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

I `{CUSTOM SCOPE 1}` `{CUSTOM SCOPE 2}` segnaposto e nell'esempio precedente sono ambiti personalizzati.

<xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AccessTokenResult.TryGetToken%2A?displayProperty=nameWithType> Restituisce

* `true` con l'oggetto `token` da usare.
* `false` Se il token non viene recuperato.

## <a name="cross-origin-resource-sharing-cors"></a>Condivisione di risorse tra le origini (CORS)

Quando si inviano le credenziali (autorizzazioni cookie s/header) sulle richieste CORS, l' `Authorization` intestazione deve essere consentita dal criterio CORS.

Il criterio seguente include la configurazione per:

* Origin della richiesta ( `http://localhost:5000` , `https://localhost:5001` ).
* Qualsiasi metodo (verbo).
* `Content-Type` e `Authorization` intestazioni. Per consentire un'intestazione personalizzata (ad esempio, `x-custom-header` ), elencare l'intestazione quando si chiama <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.WithHeaders*> .
* Credenziali impostate dal codice JavaScript lato client ( `credentials` proprietà impostata su `include` ).

```csharp
app.UseCors(policy => 
    policy.WithOrigins("http://localhost:5000", "https://localhost:5001")
    .AllowAnyMethod()
    .WithHeaders(HeaderNames.ContentType, HeaderNames.Authorization, "x-custom-header")
    .AllowCredentials());
```

Una soluzione ospitata Blazor basata sul [ Blazor WebAssembly modello di progetto](xref:blazor/project-structure) usa lo stesso indirizzo di base per le app client e server. <xref:System.Net.Http.HttpClient.BaseAddress?displayProperty=nameWithType>Per impostazione predefinita, l'app client è impostata su un URI `builder.HostEnvironment.BaseAddress` . La configurazione di CORS **non** è necessaria nella configurazione predefinita di una Blazor soluzione ospitata. Altre app client che non sono ospitate dal progetto server e non condividono **l'indirizzo di** base dell'app Server richiedono la configurazione CORS nel progetto server.

Per ulteriori informazioni, vedere <xref:security/cors> e il componente tester richieste HTTP dell'app di esempio ( `Components/HTTPRequestTester.razor` ).

## <a name="handle-token-request-errors"></a>Gestione degli errori di richiesta di token

Quando un'applicazione a pagina singola (SPA) autentica un utente usando OpenID Connect (OIDC), lo stato di autenticazione viene gestito localmente all'interno della SPA e del Identity provider (IP) sotto forma di sessione cookie che viene impostata come risultato dell'utente che fornisce le credenziali.

I token che l'IP emette per l'utente in genere sono validi per brevi periodi di tempo, circa un'ora in modo normale, quindi l'app client deve recuperare periodicamente nuovi token. In caso contrario, l'utente verrà disconnesso dopo la scadenza dei token concessi. Nella maggior parte dei casi, i client OIDC sono in grado di effettuare il provisioning di nuovi token senza richiedere all'utente di eseguire di nuovo l'autenticazione grazie allo stato di autenticazione o alla "sessione" mantenuta all'interno dell'IP.

In alcuni casi il client non è in grado di ottenere un token senza interazione dell'utente, ad esempio quando per qualche motivo l'utente si disconnette esplicitamente dall'IP. Questo scenario si verifica se un utente visita `https://login.microsoftonline.com` e si disconnette. In questi scenari, l'app non sa immediatamente che l'utente si è disconnesso. Tutti i token che il client include potrebbero non essere più validi. Inoltre, il client non è in grado di effettuare il provisioning di un nuovo token senza interazione dell'utente dopo la scadenza del token corrente.

Questi scenari non sono specifici dell'autenticazione basata su token. Sono parte della natura delle Spa. Una SPA cookie che usa s non riesce anche a chiamare un'API server se l'autenticazione cookie viene rimossa.

Quando un'app esegue chiamate API a risorse protette, è necessario tenere presente quanto segue:

* Per eseguire il provisioning di un nuovo token di accesso per chiamare l'API, l'utente potrebbe essere necessario eseguire di nuovo l'autenticazione.
* Anche se il client dispone di un token che sembra essere valido, la chiamata al server potrebbe non riuscire perché il token è stato revocato dall'utente.

Quando l'app richiede un token, esistono due possibili risultati:

* La richiesta ha esito positivo e l'app ha un token valido.
* La richiesta ha esito negativo e l'app deve autenticare di nuovo l'utente per ottenere un nuovo token.

Quando una richiesta di token ha esito negativo, è necessario decidere se si desidera salvare lo stato corrente prima di eseguire un reindirizzamento. Esistono diversi approcci con livelli di complessità crescenti:

* Archiviare lo stato della pagina corrente nell'archiviazione della sessione. Durante l' [ `OnInitializedAsync` evento del ciclo](xref:blazor/components/lifecycle#component-initialization-methods) di vita ( <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> ), verificare se è possibile ripristinare lo stato prima di continuare.
* Aggiungere un parametro della stringa di query e usarlo come metodo per segnalare all'app che è necessario riattivare lo stato salvato in precedenza.
* Aggiungere un parametro della stringa di query con un identificatore univoco per archiviare i dati nell'archiviazione della sessione senza rischiare conflitti con altri elementi.

L'esempio seguente illustra come:

* Mantenere lo stato prima di reindirizzare alla pagina di accesso.
* Ripristinare lo stato precedente in seguito all'autenticazione utilizzando il parametro della stringa di query.

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

## <a name="save-app-state-before-an-authentication-operation"></a>Salva lo stato dell'app prima di un'operazione di autenticazione

Durante un'operazione di autenticazione, esistono casi in cui si vuole salvare lo stato dell'app prima che il browser venga reindirizzato all'indirizzo IP. Questo può avvenire quando si usa un contenitore di stato e si vuole ripristinare lo stato dopo che l'autenticazione ha avuto esito positivo. È possibile usare un oggetto stato di autenticazione personalizzato per mantenere lo stato specifico dell'app o un riferimento a esso e ripristinare lo stato dopo che l'operazione di autenticazione è stata completata correttamente. Nell'esempio seguente viene illustrato l'approccio.

Viene creata una classe contenitore di stato nell'app con proprietà che contengono i valori di stato dell'app. Nell'esempio seguente il contenitore viene usato per gestire il valore del contatore del componente del [ Blazor modello di progetto](xref:blazor/project-structure) predefinito `Counter` ( `Pages/Counter.razor` ). I metodi per la serializzazione e la deserializzazione del contenitore sono basati su <xref:System.Text.Json> .

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

Il `Counter` componente usa il contenitore di stato per mantenere il `currentCount` valore all'esterno del componente:

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

Creare un `ApplicationAuthenticationState` da <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteAuthenticationState> . Fornire una `Id` proprietà, che funge da identificatore per lo stato archiviato localmente.

`ApplicationAuthenticationState.cs`:

```csharp
using Microsoft.AspNetCore.Components.WebAssembly.Authentication;

public class ApplicationAuthenticationState : RemoteAuthenticationState
{
    public string Id { get; set; }
}
```

Il `Authentication` componente ( `Pages/Authentication.razor` ) Salva e ripristina lo stato dell'app usando l'archiviazione di sessione locale con i `StateContainer` metodi di serializzazione e deserializzazione `GetStateForLocalStorage` e `SetStateFromLocalStorage` :

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

Questo esempio USA Azure Active Directory (AAD) per l'autenticazione. In `Program.Main` ( `Program.cs` ):

* `ApplicationAuthenticationState`Viene configurato come tipo di libreria Microsoft autenticazione (MSAL) `RemoteAuthenticationState` .
* Il contenitore di stato è registrato nel contenitore dei servizi.

```csharp
builder.Services.AddMsalAuthentication<ApplicationAuthenticationState>(options =>
{
    builder.Configuration.Bind("AzureAd", options.ProviderOptions.Authentication);
});

builder.Services.AddSingleton<StateContainer>();
```

## <a name="customize-app-routes"></a>Personalizzare le route delle app

Per impostazione predefinita, la [`Microsoft.AspNetCore.Components.WebAssembly.Authentication`](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.WebAssembly.Authentication) libreria usa le route mostrate nella tabella seguente per la rappresentazione di Stati di autenticazione diversi.

| Route                            | Scopo |
| -------------------------------- | ------- |
| `authentication/login`           | Attiva un'operazione di accesso. |
| `authentication/login-callback`  | Gestisce il risultato di qualsiasi operazione di accesso. |
| `authentication/login-failed`    | Visualizza i messaggi di errore quando l'operazione di accesso non riesce per qualche motivo. |
| `authentication/logout`          | Attiva un'operazione di disconnessione. |
| `authentication/logout-callback` | Gestisce il risultato di un'operazione di disconnessione. |
| `authentication/logout-failed`   | Visualizza i messaggi di errore quando l'operazione di disconnessione non riesce per qualche motivo. |
| `authentication/logged-out`      | Indica che l'utente ha eseguito correttamente la disconnessione. |
| `authentication/profile`         | Attiva un'operazione per modificare il profilo utente. |
| `authentication/register`        | Attiva un'operazione per registrare un nuovo utente. |

Le route visualizzate nella tabella precedente sono configurabili tramite <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteAuthenticationOptions%601.AuthenticationPaths%2A?displayProperty=nameWithType> . Quando si impostano le opzioni per fornire route personalizzate, verificare che l'app disponga di una route che gestisce ogni percorso.

Nell'esempio seguente, tutti i percorsi hanno il prefisso `/security` .

`Authentication` componente ( `Pages/Authentication.razor` ):

```razor
@page "/security/{action}"
@using Microsoft.AspNetCore.Components.WebAssembly.Authentication

<RemoteAuthenticatorView Action="@Action" />

@code{
    [Parameter]
    public string Action { get; set; }
}
```

`Program.Main` (`Program.cs`):

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

Se il requisito richiede percorsi completamente diversi, impostare le route come descritto in precedenza ed eseguire il rendering <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteAuthenticatorView> di con un parametro di azione esplicito:

```razor
@page "/register"

<RemoteAuthenticatorView Action="@RemoteAuthenticationActions.Register" />
```

Se si sceglie di eseguire questa operazione, è possibile suddividere l'interfaccia utente in pagine diverse.

## <a name="customize-the-authentication-user-interface"></a>Personalizzare l'interfaccia utente di autenticazione

<xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteAuthenticatorView> include un set predefinito di elementi dell'interfaccia utente per ogni stato di autenticazione. Ogni stato può essere personalizzato passando un oggetto personalizzato <xref:Microsoft.AspNetCore.Components.RenderFragment> . Per personalizzare il testo visualizzato durante il processo di accesso iniziale, può modificare il <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteAuthenticatorView> come indicato di seguito.

`Authentication` componente ( `Pages/Authentication.razor` ):

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

<xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteAuthenticatorView>Dispone di un frammento che può essere utilizzato per ogni route di autenticazione illustrato nella tabella seguente.

| Route                            | Frammento                |
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

## <a name="customize-the-user"></a>Personalizzare l'utente

Gli utenti associati all'app possono essere personalizzati.

### <a name="customize-the-user-with-a-payload-claim"></a>Personalizzare l'utente con una richiesta di payload

Nell'esempio seguente, gli utenti autenticati dell'app ricevono un' `amr` attestazione per ognuno dei metodi di autenticazione dell'utente. L' `amr` attestazione identifica la modalità di autenticazione dell'oggetto del token nelle Identity [attestazioni del payload](/azure/active-directory/develop/access-tokens#the-amr-claim)Microsoft Platform v 1.0. Nell'esempio viene utilizzata una classe di account utente personalizzata basata su <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteUserAccount> .

Creare una classe che estende la <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteUserAccount> classe. Nell'esempio seguente la proprietà viene impostata sulla `AuthenticationMethod` matrice di valori di `amr` Proprietà JSON dell'utente. `AuthenticationMethod` viene popolato automaticamente dal framework quando l'utente viene autenticato.

```csharp
using System.Text.Json.Serialization;
using Microsoft.AspNetCore.Components.WebAssembly.Authentication;

public class CustomUserAccount : RemoteUserAccount
{
    [JsonPropertyName("amr")]
    public string[] AuthenticationMethod { get; set; }
}
```

Creare una factory che <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AccountClaimsPrincipalFactory%601> si estende per creare attestazioni dai metodi di autenticazione dell'utente archiviati in `CustomUserAccount.AuthenticationMethod` :

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

Registrare `CustomAccountFactory` per il provider di autenticazione in uso. Una delle seguenti registrazioni è valida:

* <xref:Microsoft.Extensions.DependencyInjection.WebAssemblyAuthenticationServiceCollectionExtensions.AddOidcAuthentication%2A>:

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

* <xref:Microsoft.Extensions.DependencyInjection.MsalWebAssemblyServiceCollectionExtensions.AddMsalAuthentication%2A>:

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
  
* <xref:Microsoft.Extensions.DependencyInjection.WebAssemblyAuthenticationServiceCollectionExtensions.AddApiAuthorization%2A>:

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

### <a name="aad-security-groups-and-roles-with-a-custom-user-account-class"></a>Gruppi di sicurezza e ruoli di AAD con una classe di account utente personalizzata

Per un esempio aggiuntivo che funziona con i gruppi di sicurezza AAD e i ruoli di amministratore di AAD e una classe di account utente personalizzata, vedere <xref:blazor/security/webassembly/aad-groups-roles> .

## <a name="support-prerendering-with-authentication"></a>Supporto del prerendering con autenticazione

Dopo aver seguito le linee guida in uno degli Blazor WebAssembly argomenti relativi alle app per la sicurezza, usare le istruzioni seguenti per creare un'app che:

* Esegue il prerendering dei percorsi per i quali non è necessaria l'autorizzazione.
* Non esegue il prerendering dei percorsi per cui è richiesta l'autorizzazione.

Nella **`Client`** `Program` classe () dell'app `Program.cs` , scomporre le registrazioni del servizio comuni in un metodo separato, ad esempio `ConfigureCommonServices` :

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

Nell'app Server `Startup.ConfigureServices` registrare i servizi aggiuntivi seguenti:

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

Nel metodo dell'app Server `Startup.Configure` sostituire [`endpoints.MapFallbackToFile("index.html")`](xref:Microsoft.AspNetCore.Builder.StaticFilesEndpointRouteBuilderExtensions.MapFallbackToFile%2A) con [`endpoints.MapFallbackToPage("/_Host")`](xref:Microsoft.AspNetCore.Builder.RazorPagesEndpointRouteBuilderExtensions.MapFallbackToPage%2A) :

```csharp
app.UseEndpoints(endpoints =>
{
    endpoints.MapControllers();
    endpoints.MapFallbackToPage("/_Host");
});
```

Nell'app Server creare una `Pages` cartella se non esiste. Creare una `_Host.cshtml` pagina all'interno della cartella dell'app Server `Pages` . Incollare il contenuto del **`Client`** file dell'app `wwwroot/index.html` nel `Pages/_Host.cshtml` file. Aggiornare il contenuto del file:

::: moniker range=">= aspnetcore-5.0"

* Aggiungere `@page "_Host"` all'inizio del file.
* Sostituire il `<div id="app">Loading...</div>` tag con il codice seguente:

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
  
  Nell'esempio precedente, il segnaposto `{CLIENT APP ASSEMBLY NAME}` è il nome dell'assembly dell'app client (ad esempio `BlazorSample.Client` ).

::: moniker-end

::: moniker range="< aspnetcore-5.0"

* Aggiungere `@page "_Host"` all'inizio del file.
* Sostituire il `<app>Loading...</app>` tag con il codice seguente:

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
  
  Nell'esempio precedente, il segnaposto `{CLIENT APP ASSEMBLY NAME}` è il nome dell'assembly dell'app client (ad esempio `BlazorSample.Client` ).

::: moniker-end
  
## <a name="options-for-hosted-apps-and-third-party-login-providers"></a>Opzioni per le app ospitate e i provider di accesso di terze parti

Quando si esegue l'autenticazione e l'autorizzazione di un'app ospitata Blazor WebAssembly con un provider di terze parti, sono disponibili diverse opzioni per l'autenticazione dell'utente. Quello scelto dipende dallo scenario.

Per altre informazioni, vedere <xref:security/authentication/social/additional-claims>.

### <a name="authenticate-users-to-only-call-protected-third-party-apis"></a>Autenticare gli utenti per chiamare solo le API di terze parti protette

Autenticare l'utente con un flusso OAuth sul lato client rispetto al provider di API di terze parti:

 ```csharp
 builder.services.AddOidcAuthentication(options => { ... });
 ```
 
 In questo scenario:

* Il server che ospita l'app non svolge un ruolo.
* Le API nel server non possono essere protette.
* L'app può chiamare solo API di terze parti protette.

### <a name="authenticate-users-with-a-third-party-provider-and-call-protected-apis-on-the-host-server-and-the-third-party"></a>Autenticare gli utenti con un provider di terze parti e chiamare le API protette sul server host e la terza parte

Configurare Identity con un provider di accesso di terze parti. Ottenere i token necessari per l'accesso all'API di terze parti e archiviarli.

Quando un utente esegue Identity l'accesso, raccoglie i token di accesso e di aggiornamento come parte del processo di autenticazione. A questo punto, sono disponibili due approcci per eseguire chiamate API a API di terze parti.

#### <a name="use-a-server-access-token-to-retrieve-the-third-party-access-token"></a>Usare un token di accesso al server per recuperare il token di accesso di terze parti

Usare il token di accesso generato sul server per recuperare il token di accesso di terze parti da un endpoint API server. Da qui, usare il token di accesso di terze parti per chiamare le risorse dell'API di terze parti direttamente da Identity nel client.

Questo approccio non è consigliato. Questo approccio richiede di trattare il token di accesso di terze parti come se fosse stato generato per un client pubblico. In termini di OAuth, l'app pubblica non ha un segreto client perché non può essere considerata attendibile per archiviare i segreti in modo sicuro e il token di accesso viene prodotto per un client riservato. Un client riservato è un client che dispone di un segreto client e si presuppone che sia in grado di archiviare in modo sicuro i segreti.

* Al token di accesso di terze parti potrebbero essere stati concessi ulteriori ambiti per eseguire operazioni sensibili in base al fatto che la terza parte ha emesso il token per un client più attendibile.
* Analogamente, i token di aggiornamento non devono essere rilasciati a un client che non è considerato attendibile, in quanto in questo modo si ottiene l'accesso illimitato ai client a meno che non vengano applicate altre restrizioni.

#### <a name="make-api-calls-from-the-client-to-the-server-api-in-order-to-call-third-party-apis"></a>Eseguire chiamate API dal client all'API server per chiamare le API di terze parti

Eseguire una chiamata API dal client all'API del server. Dal server recuperare il token di accesso per la risorsa API di terze parti ed emettere qualsiasi chiamata necessaria.

Sebbene questo approccio richieda un hop di rete aggiuntivo tramite il server per la chiamata di un'API di terze parti, in definitiva si ottiene un'esperienza più sicura:

* Il server può archiviare i token di aggiornamento e assicurarsi che l'app non perda l'accesso alle risorse di terze parti.
* L'app non può perdere i token di accesso dal server che potrebbe contenere autorizzazioni più sensibili.

## <a name="use-openid-connect-oidc-v20-endpoints"></a>Usare gli endpoint OpenID Connect (OIDC) v 2.0

La libreria di autenticazione e i [ Blazor modelli di progetto](xref:blazor/project-structure) usano gli endpoint OPENID Connect (OIDC) v 1.0. Per usare un endpoint v 2.0, configurare l'opzione JWT Bearer <xref:Microsoft.AspNetCore.Builder.JwtBearerOptions.Authority?displayProperty=nameWithType> . Nell'esempio seguente AAD è configurato per la versione 2.0 aggiungendo un `v2.0` segmento alla <xref:Microsoft.AspNetCore.Builder.JwtBearerOptions.Authority> proprietà:

```csharp
builder.Services.Configure<JwtBearerOptions>(
    AzureADDefaults.JwtBearerAuthenticationScheme, 
    options =>
    {
        options.Authority += "/v2.0";
    });
```

In alternativa, l'impostazione può essere eseguita nel file delle impostazioni dell'app ( `appsettings.json` ):

```json
{
  "Local": {
    "Authority": "https://login.microsoftonline.com/common/oauth2/v2.0/",
    ...
  }
}
```

Se l'uso di un segmento nell'autorità non è appropriato per il provider OIDC dell'app, ad esempio con i provider non AAD, impostare <xref:Microsoft.AspNetCore.Builder.OpenIdConnectOptions.Authority> direttamente la proprietà. Impostare la proprietà in <xref:Microsoft.AspNetCore.Builder.JwtBearerOptions> o nel file di impostazioni dell'app ( `appsettings.json` ) con la `Authority` chiave.

L'elenco di attestazioni nel token ID cambia per gli endpoint della versione 2.0. Per ulteriori informazioni, vedere la pagina relativa [all'aggiornamento alla piattaforma di identità Microsoft (v 2.0)](/azure/active-directory/azuread-dev/azure-ad-endpoint-comparison).

## <a name="configure-and-use-grpc-in-components"></a>Configurare e usare gRPC nei componenti

Per configurare un' Blazor WebAssembly app per l'uso del [framework ASP.NET Core gRPC](xref:grpc/index):

* Abilitare gRPC-Web sul server. Per altre informazioni, vedere <xref:grpc/browser>.
* Registrare i servizi di gRPC per il gestore di messaggi dell'app. Nell'esempio seguente viene configurato il gestore dei messaggi di autorizzazione dell'app per usare il [ `GreeterClient` servizio dell'esercitazione su gRPC](xref:tutorials/grpc/grpc-start#create-a-grpc-service) ( `Program.Main` ):

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

Il segnaposto `{APP ASSEMBLY}` è il nome dell'assembly dell'app (ad esempio, `BlazorSample` ). Inserire il `.proto` file nel `Shared` progetto della soluzione ospitata Blazor .

Un componente nell'app client può effettuare chiamate gRPC usando il client gRPC ( `Pages/Grpc.razor` ):

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

Il segnaposto `{APP ASSEMBLY}` è il nome dell'assembly dell'app (ad esempio, `BlazorSample` ). Per usare la `Status.DebugException` proprietà, usare [Grpc .NET. client](https://www.nuget.org/packages/Grpc.Net.Client) versione 2.30.0 o successiva.

Per altre informazioni, vedere <xref:grpc/browser>.

## <a name="build-a-custom-version-of-the-authenticationmsal-javascript-library"></a>Compilare una versione personalizzata della libreria JavaScript Authentication. MSAL

Se un'app richiede una versione personalizzata di [Microsoft Authentication Library per JavaScript (MSAL.js)](https://www.npmjs.com/package/@azure/msal-browser), seguire questa procedura:

1. Verificare che il sistema disponga della versione più recente di Developer .NET SDK oppure ottenere e installare la versione più recente di Developer SDK da [.NET Core SDK: programmi di installazione e file binari](https://github.com/dotnet/installer#installers-and-binaries). La configurazione dei feed NuGet interni non è necessaria per questo scenario.
1. Configurare il `dotnet/aspnetcore` repository GitHub per lo sviluppo in base ai documenti in [Build ASP.NET Core dall'origine](https://github.com/dotnet/aspnetcore/blob/main/docs/BuildFromSource.md). Creare un fork e clonare o scaricare un archivio ZIP del [repository GitHub DotNet/aspnetcore](https://github.com/dotnet/aspnetcore).
1. Aprire il `src/Components/WebAssembly/Authentication.Msal/src/Interop/package.json` file e impostare la versione desiderata di `@azure/msal-browser` . Per un elenco delle versioni rilasciate, visitare il [ `@azure/msal-browser` sito Web NPM](https://www.npmjs.com/package/@azure/msal-browser) e selezionare la scheda **versioni** .
1. Compilare il `Authentication.Msal` progetto nella `src/Components/WebAssembly/Authentication.Msal/src` cartella con il `yarn build` comando in una shell dei comandi.
1. Se l'app usa [Asset compressi (Brotli/gzip)](xref:blazor/host-and-deploy/webassembly#compression), comprimere il `Interop/dist/Release/AuthenticationService.js` file.
1. Copiare il `AuthenticationService.js` file e le versioni compresse ( `.br` / `.gz` ) del file, se generato, dalla `Interop/dist/Release` cartella alla cartella dell'app `publish/wwwroot/_content/Microsoft.Authentication.WebAssembly.Msal` negli asset pubblicati dell'app.

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:blazor/security/webassembly/graph-api>
* [`HttpClient` e `HttpRequestMessage` con le opzioni di richiesta dell'API fetch](xref:blazor/call-web-api#httpclient-and-httprequestmessage-with-fetch-api-request-options)
