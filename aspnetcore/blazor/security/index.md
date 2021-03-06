---
title: BlazorAutenticazione e autorizzazione ASP.NET Core
author: guardrex
description: Informazioni sugli Blazor scenari di autenticazione e autorizzazione.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 05/19/2020
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
uid: blazor/security/index
ms.openlocfilehash: a840e5d54d48441e10c2a58af591a530c8fd7de9
ms.sourcegitcommit: 1436bd4d70937d6ec3140da56d96caab33c4320b
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/06/2021
ms.locfileid: "102394889"
---
# <a name="aspnet-core-blazor-authentication-and-authorization"></a>BlazorAutenticazione e autorizzazione ASP.NET Core

ASP.NET Core supporta la configurazione e la gestione della sicurezza nelle Blazor app.

Gli scenari di sicurezza sono diversi tra Blazor Server le Blazor WebAssembly app e. Poiché Blazor Server le app vengono eseguite nel server, i controlli delle autorizzazioni sono in grado di determinare:

* Le opzioni dell'interfaccia utente presentate all'utente (ad esempio, le voci di menu disponibili per un utente).
* Le regole di accesso per le aree dell'app e i componenti.

Blazor WebAssembly app eseguite sul client. L'autorizzazione viene usata *solo* per determinare quali opzioni dell'interfaccia utente visualizzare. Poiché i controlli lato client possono essere modificati o ignorati da un utente, un' Blazor WebAssembly app non può applicare regole di accesso alle autorizzazioni.

Le [ Razor convenzioni di autorizzazione pagine](xref:security/authorization/razor-pages-authorization) non si applicano ai componenti instradabili Razor . Se un componente non instradabile Razor è [incorporato in una pagina](xref:blazor/components/prerendering-and-integration), le convenzioni di autorizzazione della pagina influiscono indirettamente sul Razor componente insieme al resto del contenuto della pagina.

> [!NOTE]
> <xref:Microsoft.AspNetCore.Identity.SignInManager%601> e <xref:Microsoft.AspNetCore.Identity.UserManager%601> non sono supportati nei Razor componenti di.

## <a name="authentication"></a>Authentication

Blazor USA i meccanismi di autenticazione ASP.NET Core esistenti per stabilire l'identità dell'utente. Il meccanismo esatto dipende dalla modalità di Blazor hosting dell'app o da Blazor WebAssembly Blazor Server .

### <a name="blazor-webassembly-authentication"></a>Blazor WebAssembly e

Nelle Blazor WebAssembly app, i controlli di autenticazione possono essere ignorati perché tutto il codice lato client può essere modificato dagli utenti. Lo stesso vale per tutte le tecnologie per app sul lato client, tra cui i framework JavaScript SPA o le app native per qualsiasi sistema operativo.

Aggiungere quanto segue:

* Riferimento al pacchetto per [`Microsoft.AspNetCore.Components.Authorization`](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.Authorization) il file di progetto dell'app.
* `Microsoft.AspNetCore.Components.Authorization`Spazio dei nomi del file dell' `_Imports.razor` app.

Per gestire l'autenticazione, l'uso di un servizio incorporato o personalizzato <xref:Microsoft.AspNetCore.Components.Authorization.AuthenticationStateProvider> viene trattato nelle sezioni seguenti.

Per ulteriori informazioni sulla creazione di app e configurazione, vedere <xref:blazor/security/webassembly/index> .

### <a name="blazor-server-authentication"></a>Blazor Server e

Blazor Server le app funzionano in una connessione in tempo reale creata con SignalR . L' [autenticazione in SignalR app basate su](xref:signalr/authn-and-authz) viene gestita quando viene stabilita la connessione. L'autenticazione può essere basata su un cookie o su altri Bearer token.

Il <xref:Microsoft.AspNetCore.Components.Authorization.AuthenticationStateProvider> servizio incorporato per le Blazor Server app ottiene i dati sullo stato di autenticazione da ASP.NET Core `HttpContext.User` . Questo è il modo in cui lo stato di autenticazione si integra con i meccanismi di autenticazione ASP.NET Core esistenti.

Per ulteriori informazioni sulla creazione di app e configurazione, vedere <xref:blazor/security/server/index> .

## <a name="authenticationstateprovider-service"></a>Servizio AuthenticationStateProvider

<xref:Microsoft.AspNetCore.Components.Authorization.AuthenticationStateProvider> è il servizio sottostante usato dal componente <xref:Microsoft.AspNetCore.Components.Authorization.AuthorizeView> e dal componente <xref:Microsoft.AspNetCore.Components.Authorization.CascadingAuthenticationState> per ottenere lo stato di autenticazione.

In genere non si usa <xref:Microsoft.AspNetCore.Components.Authorization.AuthenticationStateProvider> direttamente. Usare il [ `AuthorizeView` componente](#authorizeview-component) o gli [`Task<AuthenticationState>`](#expose-the-authentication-state-as-a-cascading-parameter) approcci descritti più avanti in questo articolo. Lo svantaggio principale dell'uso diretto di <xref:Microsoft.AspNetCore.Components.Authorization.AuthenticationStateProvider> è che il componente non riceve alcuna notifica automaticamente se i dati relativi allo stato di autenticazione sottostanti cambiano.

Il servizio <xref:Microsoft.AspNetCore.Components.Authorization.AuthenticationStateProvider> può fornire i dati <xref:System.Security.Claims.ClaimsPrincipal> dell'utente corrente, come illustrato nell'esempio seguente:

```razor
@page "/"
@using System.Security.Claims
@using Microsoft.AspNetCore.Components.Authorization
@inject AuthenticationStateProvider AuthenticationStateProvider

<h3>ClaimsPrincipal Data</h3>

<button @onclick="GetClaimsPrincipalData">Get ClaimsPrincipal Data</button>

<p>@_authMessage</p>

@if (_claims.Count() > 0)
{
    <ul>
        @foreach (var claim in _claims)
        {
            <li>@claim.Type: @claim.Value</li>
        }
    </ul>
}

<p>@_surnameMessage</p>

@code {
    private string _authMessage;
    private string _surnameMessage;
    private IEnumerable<Claim> _claims = Enumerable.Empty<Claim>();

    private async Task GetClaimsPrincipalData()
    {
        var authState = await AuthenticationStateProvider.GetAuthenticationStateAsync();
        var user = authState.User;

        if (user.Identity.IsAuthenticated)
        {
            _authMessage = $"{user.Identity.Name} is authenticated.";
            _claims = user.Claims;
            _surnameMessage = 
                $"Surname: {user.FindFirst(c => c.Type == ClaimTypes.Surname)?.Value}";
        }
        else
        {
            _authMessage = "The user is NOT authenticated.";
        }
    }
}
```

Se `user.Identity.IsAuthenticated` è `true` e perché l'utente è un <xref:System.Security.Claims.ClaimsPrincipal>, è possibile enumerare le attestazioni e valutare l'appartenenza ai ruoli.

Per altre informazioni sull'inserimento delle dipendenze e sui servizi, vedere <xref:blazor/fundamentals/dependency-injection> e <xref:fundamentals/dependency-injection>.

## <a name="implement-a-custom-authenticationstateprovider"></a>Implementare un AuthenticationStateProvider personalizzato

Se l'app richiede un provider personalizzato, implementare <xref:Microsoft.AspNetCore.Components.Authorization.AuthenticationStateProvider> ed eseguire l'override di `GetAuthenticationStateAsync` :

```csharp
using System.Security.Claims;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Components.Authorization;

public class CustomAuthStateProvider : AuthenticationStateProvider
{
    public override Task<AuthenticationState> GetAuthenticationStateAsync()
    {
        var identity = new ClaimsIdentity(new[]
        {
            new Claim(ClaimTypes.Name, "mrfibuli"),
        }, "Fake authentication type");

        var user = new ClaimsPrincipal(identity);

        return Task.FromResult(new AuthenticationState(user));
    }
}
```

In un' Blazor WebAssembly app, il `CustomAuthStateProvider` servizio è registrato in `Main` di `Program.cs` :

```csharp
using Microsoft.AspNetCore.Components.Authorization;

...

builder.Services.AddScoped<AuthenticationStateProvider, CustomAuthStateProvider>();
```

In un' Blazor Server app il `CustomAuthStateProvider` servizio è registrato in `Startup.ConfigureServices` :

```csharp
using Microsoft.AspNetCore.Components.Authorization;

...

services.AddScoped<AuthenticationStateProvider, CustomAuthStateProvider>();
```

Utilizzando l'oggetto `CustomAuthStateProvider` nell'esempio precedente, tutti gli utenti vengono autenticati con il nome utente `mrfibuli` .

## <a name="expose-the-authentication-state-as-a-cascading-parameter"></a>Esporre lo stato di autenticazione come un parametro a catena

Se per la logica procedurale sono necessari dati sullo stato di autenticazione, ad esempio quando si esegue un'azione attivata dall'utente, ottenere i dati dello stato di autenticazione definendo un parametro di propagazione di tipo `Task<` <xref:Microsoft.AspNetCore.Components.Authorization.AuthenticationState> `>` :

```razor
@page "/"

<button @onclick="LogUsername">Log username</button>

<p>@_authMessage</p>

@code {
    [CascadingParameter]
    private Task<AuthenticationState> authenticationStateTask { get; set; }

    private string _authMessage;

    private async Task LogUsername()
    {
        var authState = await authenticationStateTask;
        var user = authState.User;

        if (user.Identity.IsAuthenticated)
        {
            _authMessage = $"{user.Identity.Name} is authenticated.";
        }
        else
        {
            _authMessage = "The user is NOT authenticated.";
        }
    }
}
```

Se `user.Identity.IsAuthenticated` è `true`, è possibile enumerare le attestazioni e valutare l'appartenenza ai ruoli.

Configurare il parametro di propagazione `Task<` <xref:Microsoft.AspNetCore.Components.Authorization.AuthenticationState> `>` utilizzando <xref:Microsoft.AspNetCore.Components.Authorization.AuthorizeRouteView> i <xref:Microsoft.AspNetCore.Components.Authorization.CascadingAuthenticationState> componenti e nel `App` componente ( `App.razor` ):

```razor
<CascadingAuthenticationState>
    <Router AppAssembly="@typeof(Program).Assembly">
        <Found Context="routeData">
            <AuthorizeRouteView RouteData="@routeData" 
                DefaultLayout="@typeof(MainLayout)" />
        </Found>
        <NotFound>
            <LayoutView Layout="@typeof(MainLayout)">
                <p>Sorry, there's nothing at this address.</p>
            </LayoutView>
        </NotFound>
    </Router>
</CascadingAuthenticationState>
```

[!INCLUDE[](~/blazor/includes/prefer-exact-matches.md)]

In un' Blazor WebAssembly App aggiungere servizi per le opzioni e l'autorizzazione a `Program.Main` :

```csharp
builder.Services.AddOptions();
builder.Services.AddAuthorizationCore();
```

In un' Blazor Server applicazione i servizi per le opzioni e l'autorizzazione sono già presenti, pertanto non è richiesta alcuna azione aggiuntiva.

## <a name="authorization"></a>Autorizzazione

Dopo l'autenticazione di un utente, vengono applicate le regole di *autorizzazione* per controllare le operazioni consentite per un utente.

L'accesso viene in genere concesso o negato in base alle condizioni seguenti:

* L'utente è autenticato (ha eseguito l'accesso).
* L'utente è incluso in un *ruolo*.
* L'utente ha un'*attestazione*.
* I *criteri* sono soddisfatti.

Ognuno di questi concetti è identico a quello di un'app ASP.NET Core MVC o Razor pages. Per ulteriori informazioni sulla sicurezza ASP.NET Core, vedere gli articoli in [sicurezza ASP.NET Core e Identity ](xref:security/index).

## <a name="authorizeview-component"></a>Componente AuthorizeView

Il <xref:Microsoft.AspNetCore.Components.Authorization.AuthorizeView> componente visualizza selettivamente il contenuto dell'interfaccia utente a seconda che l'utente sia autorizzato o meno. Questo approccio è utile quando è necessario solo *visualizzare* dati per l'utente e non occorre usare l'identità dell'utente nella logica procedurale.

Il componente espone una variabile `context` di tipo <xref:Microsoft.AspNetCore.Components.Authorization.AuthenticationState>, che è possibile usare per accedere alle informazioni sull'utente connesso:

```razor
<AuthorizeView>
    <h1>Hello, @context.User.Identity.Name!</h1>
    <p>You can only see this content if you're authenticated.</p>
</AuthorizeView>
```

È anche possibile fornire contenuti diversi per la visualizzazione se l'utente non è autorizzato:

```razor
<AuthorizeView>
    <Authorized>
        <h1>Hello, @context.User.Identity.Name!</h1>
        <p>You can only see this content if you're authorized.</p>
        <button @onclick="SecureMethod">Authorized Only Button</button>
    </Authorized>
    <NotAuthorized>
        <h1>Authentication Failure!</h1>
        <p>You're not signed in.</p>
    </NotAuthorized>
</AuthorizeView>

@code {
    private void SecureMethod() { ... }
}
```

Il contenuto dei `<Authorized>` `<NotAuthorized>` tag e può includere elementi arbitrari, ad esempio altri componenti interattivi.

Un gestore eventi predefinito per un elemento autorizzato, ad esempio il `SecureMethod` metodo per l' `<button>` elemento nell'esempio precedente, può essere richiamato solo da un utente autorizzato.

Le condizioni di autorizzazione, ad esempio i ruoli o i criteri che consentono di controllare le opzioni dell'interfaccia utente o l'accesso, sono presentate nella sezione [Autorizzazione](#authorization).

Se non sono specificate condizioni di autorizzazione, <xref:Microsoft.AspNetCore.Components.Authorization.AuthorizeView> usa criteri predefiniti e considera:

* Gli utenti autenticati (che hanno eseguito l'accesso) come autorizzati.
* Gli utenti non autenticati (disconnessi) come non autorizzati.

Il <xref:Microsoft.AspNetCore.Components.Authorization.AuthorizeView> componente può essere utilizzato nel `NavMenu` componente () `Shared/NavMenu.razor` per visualizzare un elemento elenco ( `<li>...</li>` ) per un [ `NavLink` componente](xref:blazor/fundamentals/routing#navlink-and-navmenu-components) ( <xref:Microsoft.AspNetCore.Components.Routing.NavLink> ), ma si noti che questo approccio rimuove solo l'elemento dell'elenco dall'output di cui è stato eseguito il rendering. Non impedisce all'utente di spostarsi nel componente.

Le app create da un [ Blazor modello di progetto](xref:blazor/project-structure) che include l'autenticazione usano un `LoginDisplay` componente che dipende da un `AuthorizeView` componente. Il `AuthorizeView` componente visualizza selettivamente il contenuto agli utenti per il Identity lavoro correlato a. Nell'esempio seguente viene dal [ Blazor WebAssembly modello di progetto](xref:blazor/project-structure).

`Shared/LoginDisplay.razor`:

```razor
@using Microsoft.AspNetCore.Components.Authorization
@using Microsoft.AspNetCore.Components.WebAssembly.Authentication

@inject NavigationManager Navigation
@inject SignOutSessionStateManager SignOutManager

<AuthorizeView>
    <Authorized>
        Hello, @context.User.Identity.Name!
        <button class="nav-link btn btn-link" @onclick="BeginLogout">Log out</button>
    </Authorized>
    <NotAuthorized>
        <a href="authentication/login">Log in</a>
    </NotAuthorized>
</AuthorizeView>

@code{
    private async Task BeginLogout(MouseEventArgs args)
    {
        await SignOutManager.SetSignOutState();
        Navigation.NavigateTo("authentication/logout");
    }
}
```

L'esempio seguente si riferisce al [ Blazor Server modello di progetto](xref:blazor/project-structure) e usa gli ASP.NET Core Identity endpoint nell' `Identity` area dell'app per elaborare il Identity lavoro correlato.

`Shared/LoginDisplay.razor`:

```razor
<AuthorizeView>
    <Authorized>
        <a href="Identity/Account/Manage">Hello, @context.User.Identity.Name!</a>
        <form method="post" action="Identity/Account/LogOut">
            <button type="submit" class="nav-link btn btn-link">Log out</button>
        </form>
    </Authorized>
    <NotAuthorized>
        <a href="Identity/Account/Register">Register</a>
        <a href="Identity/Account/Login">Log in</a>
    </NotAuthorized>
</AuthorizeView>
```

### <a name="role-based-and-policy-based-authorization"></a>Autorizzazione basata sui ruoli e basata sui criteri

Il componente <xref:Microsoft.AspNetCore.Components.Authorization.AuthorizeView> supporta l'autorizzazione *basata sui ruoli* oppure *basata sui criteri*.

Per l'autorizzazione basata sui ruoli, usare il parametro <xref:Microsoft.AspNetCore.Components.Authorization.AuthorizeView.Roles>:

```razor
<AuthorizeView Roles="admin, superuser">
    <p>You can only see this if you're an admin or superuser.</p>
</AuthorizeView>
```

Per altre informazioni, vedere <xref:security/authorization/roles>.

Per l'autorizzazione basata sui criteri, usare il parametro <xref:Microsoft.AspNetCore.Components.Authorization.AuthorizeView.Policy>:

```razor
<AuthorizeView Policy="content-editor">
    <p>You can only see this if you satisfy the "content-editor" policy.</p>
</AuthorizeView>
```

L'autorizzazione basata sulle attestazioni è un caso speciale di autorizzazione basata su criteri. Ad esempio, è possibile definire un criterio che richiede che gli utenti abbiano una determinata attestazione. Per altre informazioni, vedere <xref:security/authorization/policies>.

Queste API possono essere usate in Blazor Server o nelle Blazor WebAssembly app.

Se non si specifica <xref:Microsoft.AspNetCore.Components.Authorization.AuthorizeView.Roles> o <xref:Microsoft.AspNetCore.Components.Authorization.AuthorizeView.Policy>, <xref:Microsoft.AspNetCore.Components.Authorization.AuthorizeView> usa i criteri predefiniti.

### <a name="content-displayed-during-asynchronous-authentication"></a>Contenuto visualizzato durante l'autenticazione asincrona

Blazor consente di determinare lo stato di autenticazione in *modo asincrono*. Lo scenario principale per questo approccio è nelle Blazor WebAssembly app che effettuano una richiesta a un endpoint esterno per l'autenticazione.

Mentre è in corso l'autenticazione <xref:Microsoft.AspNetCore.Components.Authorization.AuthorizeView> non visualizza alcun contenuto per impostazione predefinita. Per visualizzare il contenuto quando si verifica l'autenticazione, usare il `<Authorizing>` Tag:

```razor
<AuthorizeView>
    <Authorized>
        <h1>Hello, @context.User.Identity.Name!</h1>
        <p>You can only see this content if you're authenticated.</p>
    </Authorized>
    <Authorizing>
        <h1>Authentication in progress</h1>
        <p>You can only see this content while authentication is in progress.</p>
    </Authorizing>
</AuthorizeView>
```

Questo approccio non è in genere applicabile alle Blazor Server app. Blazor Server le app conoscono lo stato di autenticazione non appena viene stabilito lo stato. <xref:Microsoft.AspNetCore.Components.Authorization.AuthorizeViewCore.Authorizing> il contenuto può essere fornito nel Blazor Server componente di un'app <xref:Microsoft.AspNetCore.Components.Authorization.AuthorizeView> , ma il contenuto non viene mai visualizzato.

## <a name="authorize-attribute"></a>Attributo [Authorize]

L' [ `[Authorize]` attributo](xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute) può essere utilizzato nei Razor componenti di:

```razor
@page "/"
@attribute [Authorize]

You can only see this if you're signed in.
```

> [!IMPORTANT]
> Usare solo [`[Authorize]`](xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute) sui `@page` componenti raggiunti tramite il Blazor router. L'autorizzazione viene eseguita solo come un aspetto del routing e *non* per i componenti figlio di cui viene eseguito il rendering all'interno di una pagina. Per autorizzare la visualizzazione di parti specifiche all'interno di una pagina, usare invece <xref:Microsoft.AspNetCore.Components.Authorization.AuthorizeView>.

L' [ `[Authorize]` attributo](xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute) supporta inoltre l'autorizzazione basata su ruoli o basata su criteri. Per l'autorizzazione basata sui ruoli, usare il parametro <xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute.Roles>:

```razor
@page "/"
@attribute [Authorize(Roles = "admin, superuser")]

<p>You can only see this if you're in the 'admin' or 'superuser' role.</p>
```

Per l'autorizzazione basata sui criteri, usare il parametro <xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute.Policy>:

```razor
@page "/"
@attribute [Authorize(Policy = "content-editor")]

<p>You can only see this if you satisfy the 'content-editor' policy.</p>
```

Se non <xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute.Roles> <xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute.Policy> si specifica né né, [`[Authorize]`](xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute) Usa i criteri predefiniti, che per impostazione predefinita devono trattare:

* Gli utenti autenticati (che hanno eseguito l'accesso) come autorizzati.
* Gli utenti non autenticati (disconnessi) come non autorizzati.

## <a name="customize-unauthorized-content-with-the-router-component"></a>Personalizzare il contenuto non autorizzato con il componente Router

Il <xref:Microsoft.AspNetCore.Components.Routing.Router> componente, insieme al <xref:Microsoft.AspNetCore.Components.Authorization.AuthorizeRouteView> componente, consente all'app di specificare contenuto personalizzato se:

* L'utente non riesce [`[Authorize]`](xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute) a applicare una condizione al componente. Viene visualizzato il markup dell' [`<NotAuthorized>`](xref:Microsoft.AspNetCore.Components.Authorization.AuthorizeRouteView.NotAuthorized?displayProperty=nameWithType) elemento. L' [`[Authorize]`](xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute) attributo viene analizzato nella sezione [ `[Authorize]` attribute](#authorize-attribute) .
* L'autorizzazione asincrona è in corso, che in genere significa che è in corso il processo di autenticazione dell'utente. Viene visualizzato il markup dell' [`<Authorizing>`](xref:Microsoft.AspNetCore.Components.Authorization.AuthorizeRouteView.Authorizing?displayProperty=nameWithType) elemento.
* Non viene trovato contenuto. Viene visualizzato il markup dell' [`<NotFound>`](xref:Microsoft.AspNetCore.Components.Routing.Router.NotFound?displayProperty=nameWithType) elemento.

Nel modello di [ Blazor Server progetto](xref:blazor/project-structure)predefinito, il `App` componente ( `App.razor` ) illustra come impostare il contenuto personalizzato:

```razor
<CascadingAuthenticationState>
    <Router AppAssembly="@typeof(Program).Assembly">
        <Found Context="routeData">
            <AuthorizeRouteView RouteData="@routeData" 
                DefaultLayout="@typeof(MainLayout)">
                <NotAuthorized>
                    <h1>Sorry</h1>
                    <p>You're not authorized to reach this page.</p>
                    <p>You may need to log in as a different user.</p>
                </NotAuthorized>
                <Authorizing>
                    <h1>Authorization in progress</h1>
                    <p>Only visible while authorization is in progress.</p>
                </Authorizing>
            </AuthorizeRouteView>
        </Found>
        <NotFound>
            <LayoutView Layout="@typeof(MainLayout)">
                <h1>Sorry</h1>
                <p>Sorry, there's nothing at this address.</p>
            </LayoutView>
        </NotFound>
    </Router>
</CascadingAuthenticationState>
```

[!INCLUDE[](~/blazor/includes/prefer-exact-matches.md)]

Il contenuto dei `<NotFound>` `<NotAuthorized>` tag, e `<Authorizing>` può includere elementi arbitrari, ad esempio altri componenti interattivi.

Se il `<NotAuthorized>` tag non è specificato, <xref:Microsoft.AspNetCore.Components.Authorization.AuthorizeRouteView> utilizza il seguente messaggio di fallback:

```html
Not authorized.
```

## <a name="notification-about-authentication-state-changes"></a>Notifica per le modifiche dello stato di autenticazione

Se l'app determina che i dati sullo stato di autenticazione sottostanti sono stati modificati, ad esempio perché l'utente ha effettuato la disconnessione o un altro utente ha modificato i ruoli, un oggetto [personalizzato `AuthenticationStateProvider` ](#implement-a-custom-authenticationstateprovider) può richiamare facoltativamente il metodo <xref:Microsoft.AspNetCore.Components.Authorization.AuthenticationStateProvider.NotifyAuthenticationStateChanged%2A> sulla <xref:Microsoft.AspNetCore.Components.Authorization.AuthenticationStateProvider> classe di base. Viene così inviata notifica ai consumer dei dati di stato di autenticazione (ad esempio, <xref:Microsoft.AspNetCore.Components.Authorization.AuthorizeView>) di eseguire nuovamente il rendering usando i nuovi dati.

## <a name="procedural-logic"></a>Logica procedurale

Se l'app è necessaria per verificare le regole di autorizzazione come parte della logica procedurale, usare un parametro a cascata di tipo `Task<` <xref:Microsoft.AspNetCore.Components.Authorization.AuthenticationState> `>` per ottenere l'utente <xref:System.Security.Claims.ClaimsPrincipal> . `Task<`<xref:Microsoft.AspNetCore.Components.Authorization.AuthenticationState>`>` può essere combinato con altri servizi, ad esempio `IAuthorizationService` , per valutare i criteri.

```razor
@using Microsoft.AspNetCore.Authorization
@inject IAuthorizationService AuthorizationService

<button @onclick="@DoSomething">Do something important</button>

@code {
    [CascadingParameter]
    private Task<AuthenticationState> authenticationStateTask { get; set; }

    private async Task DoSomething()
    {
        var user = (await authenticationStateTask).User;

        if (user.Identity.IsAuthenticated)
        {
            // Perform an action only available to authenticated (signed-in) users.
        }

        if (user.IsInRole("admin"))
        {
            // Perform an action only available to users in the 'admin' role.
        }

        if ((await AuthorizationService.AuthorizeAsync(user, "content-editor"))
            .Succeeded)
        {
            // Perform an action only available to users satisfying the 
            // 'content-editor' policy.
        }
    }
}
```

> [!NOTE]
> In un Blazor WebAssembly componente dell'app aggiungere gli <xref:Microsoft.AspNetCore.Authorization> <xref:Microsoft.AspNetCore.Components.Authorization> spazi dei nomi e:
>
> ```razor
> @using Microsoft.AspNetCore.Authorization
> @using Microsoft.AspNetCore.Components.Authorization
> ```
>
> Questi spazi dei nomi possono essere forniti a livello globale aggiungendoli al file dell'app `_Imports.razor` .

## <a name="troubleshoot-errors"></a>Risolvere gli errori

Errori comuni:

* **L'autorizzazione richiede un parametro di propagazione di tipo `Task\<AuthenticationState>` . Prendere `CascadingAuthenticationState` in considerazione l'uso di per fornire questo.**

* **`null` valore ricevuto per `authenticationStateTask`**

È probabile che il progetto non sia stato creato usando un Blazor Server modello con autenticazione abilitata. Eseguire il wrapping di un oggetto intorno a una `<CascadingAuthenticationState>` parte dell'albero dell'interfaccia utente, ad esempio nel `App` componente ( `App.razor` ) come indicato di seguito:

```razor
<CascadingAuthenticationState>
    <Router AppAssembly="@typeof(Program).Assembly">
        ...
    </Router>
</CascadingAuthenticationState>
```

[!INCLUDE[](~/blazor/includes/prefer-exact-matches.md)]

Fornisce il parametro di propagazione <xref:Microsoft.AspNetCore.Components.Authorization.CascadingAuthenticationState> `Task<` <xref:Microsoft.AspNetCore.Components.Authorization.AuthenticationState> `>` , che a sua volta riceve dal <xref:Microsoft.AspNetCore.Components.Authorization.AuthenticationStateProvider> servizio di.

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:security/index>
* <xref:security/authentication/windowsauth>
* [Compilare una versione personalizzata della libreria JavaScript Authentication. MSAL](xref:blazor/security/webassembly/additional-scenarios#build-a-custom-version-of-the-authenticationmsal-javascript-library)
* [Awesome Blazor :](https://github.com/AdrienTorris/awesome-blazor#authentication) collegamenti di esempio della community di autenticazione
