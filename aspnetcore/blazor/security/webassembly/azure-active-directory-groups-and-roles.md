---
title: ASP.NET Core Blazor WebAssembly con gruppi e ruoli di Azure Active Directory
author: guardrex
description: Informazioni su come configurare Blazor WebAssembly per l'uso di gruppi e ruoli di Azure Active Directory.
monikerRange: '>= aspnetcore-5.0'
ms.author: riande
ms.custom: devx-track-csharp, mvc
ms.date: 01/24/2021
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
uid: blazor/security/webassembly/aad-groups-roles
ms.openlocfilehash: b725a60a310be23f7ceb626d4c543d0df6fadf62
ms.sourcegitcommit: 1436bd4d70937d6ec3140da56d96caab33c4320b
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/06/2021
ms.locfileid: "102394759"
---
# <a name="azure-active-directory-aad-groups-administrator-roles-and-app-roles"></a>Gruppi di Azure Active Directory (AAD), ruoli di amministratore e ruoli dell'app

Azure Active Directory (AAD) fornisce diversi approcci di autorizzazione che possono essere combinati con ASP.NET Core Identity :

* Gruppi
  * Sicurezza
  * Microsoft 365
  * Distribuzione
* Ruoli
  * Ruoli di amministratore di AAD
  * Ruoli dell'app

Le indicazioni fornite in questo articolo sono valide per gli Blazor WebAssembly scenari di distribuzione di AAD descritti negli argomenti seguenti:

* [App autonoma con account Microsoft](xref:blazor/security/webassembly/standalone-with-microsoft-accounts)
* [App autonoma con AAD](xref:blazor/security/webassembly/standalone-with-azure-active-directory)
* [App ospitata con AAD](xref:blazor/security/webassembly/hosted-with-azure-active-directory)

Le linee guida dell'articolo forniscono istruzioni per le app client e server:

* **Client**: app autonome Blazor WebAssembly o **`Client`** app di una soluzione ospitata Blazor .
* **Server**: autonomo ASP.NET Core API del server/app per le API Web o l' **`Server`** app di una soluzione ospitata Blazor .

## <a name="scopes"></a>Ambiti

Per consentire [Microsoft Graph chiamate API](/graph/use-the-api) per i dati del profilo utente, dell'assegnazione di ruolo e delle appartenenze ai gruppi, il **client** viene configurato con ( `https://graph.microsoft.com/User.Read` ) [API Graph autorizzazione (ambito)](/graph/permissions-reference) nel portale di Azure.

Un'app **Server** che chiama API Graph per i dati di appartenenza a gruppi e ruoli viene fornita `GroupMember.Read.All` ( `https://graph.microsoft.com/GroupMember.Read.All` ) [API Graph autorizzazione (ambito)](/graph/permissions-reference) nella portale di Azure.

Questi ambiti sono necessari in aggiunta agli ambiti richiesti negli scenari di distribuzione di AAD descritti negli argomenti elencati nella prima sezione di questo articolo.

> [!NOTE]
> Le parole "autorizzazione" e "ambito" vengono utilizzate in modo interscambiabile nella portale di Azure e in diversi set di documentazione esterni e Microsoft. Questo articolo usa la parola "scope" in tutto per le autorizzazioni assegnate a un'app nel portale di Azure.

## <a name="group-membership-claims-attribute"></a>Attributo delle attestazioni di appartenenza a gruppo

Nel manifesto dell'app nella portale di Azure per le app **client** e **Server** , impostare l' [ `groupMembershipClaims` attributo](/azure/active-directory/develop/reference-app-manifest#groupmembershipclaims-attribute) su `All` . Il valore `All` risultante è il recupero di tutti i gruppi di sicurezza, i gruppi di distribuzione e i ruoli di cui l'utente connesso è membro.

1. Aprire la registrazione portale di Azure dell'app.
1. Selezionare **Gestisci**  >  **manifesto** nella barra laterale.
1. Trovare l' `groupMembershipClaims` attributo.
1. Impostare il valore su `All`.
1. Fare clic sul pulsante **Salva**.

```json
"groupMembershipClaims": "All",
```

## <a name="custom-user-account"></a>Account utente personalizzato

Assegnare gli utenti ai gruppi di sicurezza AAD e ai ruoli di amministratore di AAD nel portale di Azure.

Gli esempi in questo articolo:

* Si supponga che un utente sia assegnato al ruolo di *amministratore fatturazione* di AAD nel tenant portale di Azure AAD per l'autorizzazione ad accedere ai dati dell'API del server.
* Usare i [criteri di autorizzazione](xref:security/authorization/policies) per controllare l'accesso all'interno delle app **client** e **server** .

Nell'app **client** Estendi <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteUserAccount> per includere le proprietà per:

* `Roles`: Matrice dei ruoli dell'app AAD (analizzata nella sezione [ruoli app](#app-roles) )
* `Wids`: Ruoli di amministratore di AAD in [attestazioni ID note ( `wids` )](/azure/active-directory/develop/access-tokens#payload-claims)
* `Oid`: [Attestazione dell'identificatore di oggetto `oid` ](/azure/active-directory/develop/id-tokens#payload-claims) non modificabile () (identifica in modo univoco un utente all'interno e tra i tenant)

Assegnare una matrice vuota a ogni proprietà della matrice in modo che il controllo `null` non sia necessario quando queste proprietà vengono usate nei `foreach` cicli.

`CustomUserAccount.cs`:

```csharp
using System;
using System.Text.Json.Serialization;
using Microsoft.AspNetCore.Components.WebAssembly.Authentication;

public class CustomUserAccount : RemoteUserAccount
{
    [JsonPropertyName("roles")]
    public string[] Roles { get; set; } = Array.Empty<string>();

    [JsonPropertyName("wids")]
    public string[] Wids { get; set; } = Array.Empty<string>();

    [JsonPropertyName("oid")]
    public string Oid { get; set; }
}
```

Aggiungere un riferimento al pacchetto per il file di progetto dell'app **client** [`Microsoft.Graph`](https://www.nuget.org/packages/Microsoft.Graph) .

Aggiungere le classi e la configurazione dell'utilità Graph SDK nella sezione *Graph SDK* dell' <xref:blazor/security/webassembly/graph-api#graph-sdk> articolo. Nella `GraphClientExtensions` classe specificare l' `User.Read` ambito per il token di accesso nel `AuthenticateRequestAsync` Metodo:

```csharp
var result = await TokenProvider.RequestAccessToken(
    new AccessTokenRequestOptions()
    {
        Scopes = new[] { "https://graph.microsoft.com/User.Read" }
    });
```

Aggiungere la factory di account utente personalizzata seguente all'app **client** . La Factory utente personalizzata viene usata per stabilire:

* Attestazioni del ruolo app ( `appRole` ) (descritte nella sezione [ruoli app](#app-roles) )
* Attestazioni ruolo amministratore AAD ( `directoryRole` )
* Un esempio di attestazione dati del profilo utente per il numero di cellulare dell'utente ( `mobilePhone` )
* Attestazioni del gruppo AAD ( `directoryGroup` )

`CustomAccountFactory.cs`:

```csharp
using System;
using System.Security.Claims;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Components.WebAssembly.Authentication;
using Microsoft.AspNetCore.Components.WebAssembly.Authentication.Internal;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Logging;
using Microsoft.Graph;

public class CustomAccountFactory
    : AccountClaimsPrincipalFactory<CustomUserAccount>
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
        CustomUserAccount account,
        RemoteAuthenticationUserOptions options)
    {
        var initialUser = await base.CreateUserAsync(account, options);

        if (initialUser.Identity.IsAuthenticated)
        {
            var userIdentity = (ClaimsIdentity)initialUser.Identity;

            foreach (var role in account.Roles)
            {
                userIdentity.AddClaim(new Claim("appRole", role));
            }

            foreach (var wid in account.Wids)
            {
                userIdentity.AddClaim(new Claim("directoryRole", wid));
            }

            try
            {
                var graphClient = ActivatorUtilities
                    .CreateInstance<GraphServiceClient>(serviceProvider);

                var requestMe = graphClient.Me.Request();
                var user = await requestMe.GetAsync();

                if (user != null)
                {
                    userIdentity.AddClaim(new Claim("mobilePhone",
                        user.MobilePhone));
                }

                var requestMemberOf = graphClient.Users[account.Oid].MemberOf;
                var memberships = await requestMemberOf.Request().GetAsync();

                if (memberships != null)
                {
                    foreach (var entry in memberships)
                    {
                        if (entry.ODataType == "#microsoft.graph.group")
                        {
                            userIdentity.AddClaim(
                                new Claim("directoryGroup", entry.Id));
                        }
                    }
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

Il codice precedente non include le appartenenze transitive. Se l'app richiede attestazioni di appartenenza a gruppi dirette e transitive, sostituire la `MemberOf` Proprietà ( `IUserMemberOfCollectionWithReferencesRequestBuilder` ) con `TransitiveMemberOf` ( `IUserTransitiveMemberOfCollectionWithReferencesRequestBuilder` ).

Il codice precedente ignora le attestazioni di appartenenza a gruppi ( `groups` ) che sono ruoli `#microsoft.graph.directoryRole` di amministratore di AAD (tipo) perché i valori GUID restituiti dalla Identity piattaforma Microsoft 2,0 sono **ID entità** del ruolo amministratore di AAD e non [**ID modello di ruolo**](/azure/active-directory/roles/permissions-reference#role-template-ids). Gli ID entità non sono stabili tra i tenant nella Identity piattaforma Microsoft 2,0 e non devono essere usati per creare criteri di autorizzazione per gli utenti nelle app. Usare sempre gli **ID dei modelli di ruolo** per i ruoli di amministratore di AAD **forniti dalle `wids` attestazioni**.

Nell' `Program.Main` app **client** configurare l'autenticazione MSAL per l'uso della factory dell'account utente personalizzata.

`Program.cs`:

```csharp
using Microsoft.AspNetCore.Components.WebAssembly.Authentication;
using Microsoft.Extensions.Configuration;

...

builder.Services.AddMsalAuthentication<RemoteAuthenticationState,
    CustomUserAccount>(options =>
{
    ...
})
.AddAccountClaimsPrincipalFactory<RemoteAuthenticationState, CustomUserAccount,
    CustomAccountFactory>();

...

builder.Services.AddGraphClient();
```

## <a name="authorization-configuration"></a>Configurazione dell'autorizzazione

Nell'app **client** creare un [criterio](xref:security/authorization/policies) per ogni [ruolo app](#app-roles), ruolo amministratore AAD o gruppo di sicurezza in `Program.Main` . L'esempio seguente crea un criterio per il ruolo di *amministratore fatturazione* di AAD:

```csharp
builder.Services.AddAuthorizationCore(options =>
{
    options.AddPolicy("BillingAdministrator", policy => 
        policy.RequireClaim("directoryRole", 
            "b0f54661-2d74-4c50-afa3-1ec803f12efe"));
});
```

Per l'elenco completo degli ID per i ruoli di amministratore di AAD, vedere [ID dei modelli di ruolo](/azure/active-directory/roles/permissions-reference#role-template-ids) nella documentazione di Azure. Per ulteriori informazioni sui criteri di autorizzazione, vedere <xref:security/authorization/policies> .

Negli esempi seguenti l'app **client** usa il criterio precedente per autorizzare l'utente.

Il [ `AuthorizeView` componente](xref:blazor/security/index#authorizeview-component) funziona con i criteri:

```razor
<AuthorizeView Policy="BillingAdministrator">
    <Authorized>
        <p>
            The user is in the 'Billing Administrator' AAD Administrator Role
            and can see this content.
        </p>
    </Authorized>
    <NotAuthorized>
        <p>
            The user is NOT in the 'Billing Administrator' role and sees this
            content.
        </p>
    </NotAuthorized>
</AuthorizeView>
```

L'accesso a un intero componente può essere basato sui criteri mediante una [ `[Authorize]` direttiva attribute](xref:blazor/security/index#authorize-attribute) ( <xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute> ):

```razor
@page "/"
@using Microsoft.AspNetCore.Authorization
@attribute [Authorize(Policy = "BillingAdministrator")]
```

Se l'utente non è connesso, viene reindirizzato alla pagina di accesso di AAD e quindi di nuovo al componente dopo l'accesso.

Un controllo dei criteri può essere [eseguito anche nel codice con la logica procedurale](xref:blazor/security/index#procedural-logic).

`Pages/CheckPolicy.razor`:

```razor
@page "/checkpolicy"
@using Microsoft.AspNetCore.Authorization
@inject IAuthorizationService AuthorizationService

<h1>Check Policy</h1>

<p>This component checks a policy in code.</p>

<button @onclick="CheckPolicy">Check 'BillingAdministrator' policy</button>

<p>Policy Message: @policyMessage</p>

@code {
    private string policyMessage = "Check hasn't been made yet.";

    [CascadingParameter]
    private Task<AuthenticationState> authenticationStateTask { get; set; }

    private async void CheckPolicy()
    {
        var user = (await authenticationStateTask).User;

        if ((await AuthorizationService.AuthorizeAsync(user, 
            "BillingAdministrator")).Succeeded)
        {
            policyMessage = "Yes! The 'BillingAdministrator' policy is met.";
        }
        else
        {
            policyMessage = "No! 'BillingAdministrator' policy is NOT met.";
        }
    }
}
```

## <a name="authorize-server-apiweb-api-access"></a>Autorizzare l'accesso API Server/API Web

Un'app per le API **Server** può autorizzare gli utenti ad accedere a endpoint API protetti con [criteri di autorizzazione](xref:security/authorization/policies) per i gruppi di sicurezza, i ruoli di amministratore di AAD e i ruoli dell'app quando un token di accesso contiene `groups` `wids` `http://schemas.microsoft.com/ws/2008/06/identity/claims/role` attestazioni, e. L'esempio seguente crea un criterio per il ruolo di *amministratore fatturazione* di AAD in `Startup.ConfigureServices` usando le `wids` attestazioni (ID noto/ID modello di ruolo):

```csharp
services.AddAuthorization(options =>
{
    options.AddPolicy("BillingAdministrator", policy => 
        policy.RequireClaim("wids", "b0f54661-2d74-4c50-afa3-1ec803f12efe"));
});
```

Per l'elenco completo degli ID per i ruoli di amministratore di AAD, vedere [ID dei modelli di ruolo](/azure/active-directory/roles/permissions-reference#role-template-ids) nella documentazione di Azure. Per ulteriori informazioni sui criteri di autorizzazione, vedere <xref:security/authorization/policies> .

L'accesso a un controller nell'app **Server** può essere basato sull'uso di un [ `[Authorize]` attributo](xref:security/authorization/simple) con il nome del criterio (documentazione API: <xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute> ).

Nell'esempio seguente viene limitato l'accesso ai dati di fatturazione da `BillingDataController` ad amministratori della fatturazione di Azure con un nome di criterio `BillingAdministrator` :

```csharp
...
using Microsoft.AspNetCore.Authorization;

[Authorize(Policy = "BillingAdministrator")]
[ApiController]
[Route("[controller]")]
public class BillingDataController : ControllerBase
{
    ...
}
```

Per altre informazioni, vedere <xref:security/authorization/policies>.

## <a name="app-roles"></a>Ruoli dell'app

Per configurare l'app nel portale di Azure per fornire attestazioni di appartenenza ai ruoli dell'app, vedere [procedura: aggiungere i ruoli dell'app nell'applicazione e riceverli nel token](/azure/active-directory/develop/howto-add-app-roles-in-azure-ad-apps) nella documentazione di Azure.

Nell'esempio seguente si presuppone che le app **client** e **Server** siano configurate con due ruoli e che i ruoli siano assegnati a un utente di test:

* `admin`
* `developer`

> [!NOTE]
> Quando si sviluppa un' Blazor WebAssembly app ospitata o una coppia client-server di app autonome (un'app autonoma Blazor WebAssembly e un'api di ASP.NET Core Server autonoma o un'app per le API Web), la `appRoles` Proprietà Manifest sia del client che del server portale di Azure le registrazioni dell'app devono includere gli stessi ruoli configurati. Dopo aver stabilito i ruoli nel manifesto dell'app client, copiarli interamente nel manifesto dell'app Server. Se non si esegue il mirroring del manifesto `appRoles` tra le registrazioni dell'app client e server, le attestazioni dei ruoli non vengono stabilite per gli utenti autenticati dell'API Server/API Web, anche se il relativo token di accesso ha le attestazioni dei ruoli corrette.

> [!NOTE]
> Sebbene non sia possibile assegnare ruoli a gruppi senza un account di Azure AD Premium, è possibile assegnare ruoli agli utenti e ricevere un'attestazione dei ruoli per gli utenti con un account Azure standard. Le indicazioni fornite in questa sezione non richiedono un account AAD Premium.
>
> Nel portale di Azure vengono assegnati più ruoli aggiungendo **_nuovamente un utente_** per ogni assegnazione di ruolo aggiuntiva.

Il `CustomAccountFactory` illustrato nella sezione [account utente personalizzato](#custom-user-account) è configurato per agire su un' `roles` attestazione con un valore di matrice JSON. Aggiungere e registrare nell' `CustomAccountFactory` app **client** come illustrato nella sezione [account utente personalizzato](#custom-user-account) . Non è necessario fornire codice per rimuovere l' `roles` attestazione originale perché viene automaticamente rimossa dal Framework.

In `Program.Main` di un'app **client** specificare l'attestazione denominata " `appRole` " come attestazione del ruolo per i <xref:System.Security.Claims.ClaimsPrincipal.IsInRole%2A?displayProperty=nameWithType> Controlli:

```csharp
builder.Services.AddMsalAuthentication(options =>
{
    ...

    options.UserOptions.RoleClaim = "appRole";
});
```

> [!NOTE]
> Se si preferisce usare l' `directoryRoles` attestazione (Aggiungi ruoli di amministratore), assegnare " `directoryRoles` " a <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteAuthenticationUserOptions.RoleClaim?displayProperty=nameWithType> .

In `Startup.ConfigureServices` di un'app **Server** specificare l'attestazione denominata " `http://schemas.microsoft.com/ws/2008/06/identity/claims/role` " come attestazione del ruolo per i <xref:System.Security.Claims.ClaimsPrincipal.IsInRole%2A?displayProperty=nameWithType> Controlli:

```csharp
services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddMicrosoftIdentityWebApi(options =>
    {
        Configuration.Bind("AzureAd", options);
        options.TokenValidationParameters.RoleClaimType = 
            "http://schemas.microsoft.com/ws/2008/06/identity/claims/role";
    },
    options => { Configuration.Bind("AzureAd", options); });
```

> [!NOTE]
> Se si preferisce usare l' `wids` attestazione (Aggiungi ruoli di amministratore), assegnare " `wids` " a <xref:Microsoft.IdentityModel.Tokens.TokenValidationParameters.RoleClaimType?displayProperty=nameWithType> .

A questo punto sono funzionali gli approcci di autorizzazione per i componenti. Qualsiasi meccanismo di autorizzazione nei componenti dell'app **client** può usare il `admin` ruolo per autorizzare l'utente:

* [`AuthorizeView` componente](xref:blazor/security/index#authorizeview-component)

  ```razor
  <AuthorizeView Roles="admin">
  ```

* [ `[Authorize]` attribute](xref:blazor/security/index#authorize-attribute) (direttiva <xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute> ) ()

  ```razor
  @attribute [Authorize(Roles = "admin")]
  ```

* [Logica procedurale](xref:blazor/security/index#procedural-logic)

  ```csharp
  var authState = await AuthenticationStateProvider.GetAuthenticationStateAsync();
  var user = authState.User;

  if (user.IsInRole("admin")) { ... }
  ```

Sono supportati più test di ruolo:

* Richiedere **che l'utente sia nel** `admin` ruolo **o** `developer` con il `AuthorizeView` componente:

  ```razor
  <AuthorizeView Roles="admin, developer">
      ...
  </AuthorizeView>
  ```

* Richiedere **che l'utente sia nei** `admin` ruoli **e** `developer` con il `AuthorizeView` componente:

  ```razor
  <AuthorizeView Roles="admin">
      <AuthorizeView Roles="developer">
          ...
      </AuthorizeView>
  </AuthorizeView>
  ```

* Richiedere **che l'utente sia nel** `admin` ruolo **o** `developer` con l' `[Authorize]` attributo:

  ```razor
  @attribute [Authorize(Roles = "admin, developer")]
  ```

* Richiedere **che l'utente sia nei** `admin` ruoli **e** `developer` con l' `[Authorize]` attributo:

  ```razor
  @attribute [Authorize(Roles = "admin")]
  @attribute [Authorize(Roles = "developer")]
  ```

* Richiedere **che l'utente sia nel** `admin` ruolo **o** `developer` con codice procedurale:

  ```razor
  @code {
      private async Task DoSomething()
      {
          var authState = await AuthenticationStateProvider
              .GetAuthenticationStateAsync();
          var user = authState.User;

          if (user.IsInRole("admin") || user.IsInRole("developer"))
          {
              ...
          }
          else
          {
              ...
          }
      }
  }
  ```

* È necessario che l'utente sia in **entrambi** i `admin` ruoli **e** `developer` con il codice procedurale modificando l' [OR condizionale ( `||` )](/dotnet/csharp/language-reference/operators/boolean-logical-operators) in un [and condizionale ( `&&` )](/dotnet/csharp/language-reference/operators/boolean-logical-operators) nell'esempio precedente:

  ```csharp
  if (user.IsInRole("admin") && user.IsInRole("developer"))
  ```

Qualsiasi meccanismo di autorizzazione nei controller dell'app **Server** può usare il `admin` ruolo per autorizzare l'utente:

* [ `[Authorize]` attribute](xref:blazor/security/index#authorize-attribute) (direttiva <xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute> ) ()

  ```csharp
  [Authorize(Roles = "admin")]
  ```

* [Logica procedurale](xref:blazor/security/index#procedural-logic)

  ```csharp
  if (User.IsInRole("admin")) { ... }
  ```

Sono supportati più test di ruolo:

* Richiedere **che l'utente sia nel** `admin` ruolo **o** `developer` con l' `[Authorize]` attributo:

  ```csharp
  [Authorize(Roles = "admin, developer")]
  ```

* Richiedere **che l'utente sia nei** `admin` ruoli **e** `developer` con l' `[Authorize]` attributo:

  ```csharp
  [Authorize(Roles = "admin")]
  [Authorize(Roles = "developer")]
  ```

* Richiedere **che l'utente sia nel** `admin` ruolo **o** `developer` con codice procedurale:

  ```csharp
  static readonly string[] scopeRequiredByApi = new string[] { "API.Access" };

  ...

  [HttpGet]
  public IEnumerable<ReturnType> Get()
  {
      HttpContext.VerifyUserHasAnyAcceptedScope(scopeRequiredByApi);

      if (User.IsInRole("admin") || User.IsInRole("developer"))
      {
          ...
      }
      else
      {
          ...
      }

      return ...
  }
  ```

* È necessario che l'utente sia in **entrambi** i `admin` ruoli **e** `developer` con il codice procedurale modificando l' [OR condizionale ( `||` )](/dotnet/csharp/language-reference/operators/boolean-logical-operators) in un [and condizionale ( `&&` )](/dotnet/csharp/language-reference/operators/boolean-logical-operators) nell'esempio precedente:

  ```csharp
  if (User.IsInRole("admin") && User.IsInRole("developer"))
  ```

## <a name="additional-resources"></a>Risorse aggiuntive

* [ID del modello di ruolo (documentazione di Azure)](/azure/active-directory/roles/permissions-reference#role-template-ids)
* [`groupMembershipClaims` attribute (documentazione di Azure)](/azure/active-directory/develop/reference-app-manifest#groupmembershipclaims-attribute)
* [Procedura: aggiungere i ruoli dell'app nell'applicazione e riceverli nel token (documentazione di Azure)](/azure/active-directory/develop/howto-add-app-roles-in-azure-ad-apps)
* [Ruoli applicazione (documentazione di Azure)](/azure/architecture/multitenant-identity/app-roles)
* <xref:security/authorization/claims>
* <xref:security/authorization/roles>
* <xref:blazor/security/index>
