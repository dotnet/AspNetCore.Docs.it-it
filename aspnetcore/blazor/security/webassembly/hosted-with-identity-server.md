---
title: Proteggere un'app ASP.NET Core ospitata Blazor WebAssembly con il Identity Server
author: guardrex
description: Informazioni su come proteggere un'app ASP.NET Core ospitata Blazor WebAssembly con il Identity Server.
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
uid: blazor/security/webassembly/hosted-with-identity-server
ms.openlocfilehash: c74711c10fe399718600f879c3d9151bfb1abd42
ms.sourcegitcommit: 1166b0ff3828418559510c661e8240e5c5717bb7
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/12/2021
ms.locfileid: "100281010"
---
# <a name="secure-an-aspnet-core-blazor-webassembly-hosted-app-with-identity-server"></a>Proteggere un' Blazor WebAssembly app ospitata ASP.NET Core Identity con server

Questo articolo illustra come creare un' [ Blazor WebAssembly app ospitata](xref:blazor/hosting-models#blazor-webassembly) che usa il [ Identity Server](https://identityserver.io/) per autenticare gli utenti e le chiamate API.

> [!NOTE]
> Per configurare un'app autonoma o ospitata Blazor WebAssembly per l'uso di un' Identity istanza del server esterno esistente, seguire le istruzioni riportate in <xref:blazor/security/webassembly/standalone-with-authentication-library> .

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

Per creare un nuovo Blazor WebAssembly progetto con un meccanismo di autenticazione:

1. Dopo aver scelto il modello **Blazor WebAssembly app** nella finestra di dialogo **Crea un nuovo ASP.NET Core applicazione Web** , selezionare **Cambia** in **autenticazione**.

1. Selezionare **singoli account utente** con l'opzione **Archivia account utente in-app** per archiviare gli utenti all'interno dell'app usando il sistema di ASP.NET Core [Identity](xref:security/authentication/identity) .

1. Nella sezione **Avanzate** Selezionare la casella di controllo **ASP.NET Core Hosted** .

# <a name="visual-studio-code--net-core-cli"></a>[Visual Studio Code/Interfaccia della riga di comando di .NET Core](#tab/visual-studio-code+netcore-cli)

Per creare un nuovo Blazor WebAssembly progetto con un meccanismo di autenticazione in una cartella vuota, specificare il `Individual` meccanismo di autenticazione con l' `-au|--auth` opzione per archiviare gli utenti all'interno dell'app usando il sistema di ASP.NET Core [Identity](xref:security/authentication/identity) :

```dotnetcli
dotnet new blazorwasm -au Individual -ho -o {APP NAME}
```

| Segnaposto  | Esempio        |
| ------------ | -------------- |
| `{APP NAME}` | `BlazorSample` |

Il percorso di output specificato con l'opzione `-o|--output` consente di creare una cartella di progetto se non esiste e viene incluso nel nome dell'app.

Per ulteriori informazioni, vedere il [`dotnet new`](/dotnet/core/tools/dotnet-new) comando nella Guida di .NET Core.

# <a name="visual-studio-for-mac"></a>[Visual Studio per Mac](#tab/visual-studio-mac)

Per creare un nuovo Blazor WebAssembly progetto con un meccanismo di autenticazione:

1. Nel passaggio **configurare la nuova Blazor WebAssembly app** selezionare **autenticazione singola (in-app) nell'elenco a** discesa **autenticazione** .

1. L'app viene creata per i singoli utenti archiviati nell'app con ASP.NET Core [Identity](xref:security/authentication/identity) .

1. Selezionare la casella di controllo **ASP.NET Core Hosted** .

---

## <a name="server-app-configuration"></a>*`Server`* configurazione dell'app

Le sezioni seguenti descrivono le aggiunte al progetto quando è incluso il supporto per l'autenticazione.

### <a name="startup-class"></a>Classe di avvio

La `Startup` classe presenta le aggiunte seguenti.

* In `Startup.ConfigureServices`:

  * ASP.NET Core Identity:

    ```csharp
    services.AddDbContext<ApplicationDbContext>(options =>
        options.UseSqlite(
            Configuration.GetConnectionString("DefaultConnection")));

    services.AddDefaultIdentity<ApplicationUser>(options => 
            options.SignIn.RequireConfirmedAccount = true)
        .AddEntityFrameworkStores<ApplicationDbContext>();
    ```

  * IdentityServer con un <xref:Microsoft.Extensions.DependencyInjection.IdentityServerBuilderConfigurationExtensions.AddApiAuthorization%2A> metodo helper aggiuntivo che imposta le convenzioni ASP.NET Core predefinite sul Identity Server:

    ```csharp
    services.AddIdentityServer()
        .AddApiAuthorization<ApplicationUser, ApplicationDbContext>();
    ```

  * Autenticazione con un <xref:Microsoft.AspNetCore.Authentication.AuthenticationBuilderExtensions.AddIdentityServerJwt%2A> metodo helper aggiuntivo che configura l'app per convalidare i token JWT prodotti dal Identity Server:

    ```csharp
    services.AddAuthentication()
        .AddIdentityServerJwt();
    ```

* In `Startup.Configure`:

  * Il Identity middleware server espone gli endpoint OpenID Connect (OIDC):

    ```csharp
    app.UseIdentityServer();
    ```

  * Il middleware di autenticazione è responsabile della convalida delle credenziali della richiesta e dell'impostazione dell'utente nel contesto della richiesta:

    ```csharp
    app.UseAuthentication();
    ```

  * Il middleware di autorizzazione consente le funzionalità di autorizzazione:

    ```csharp
    app.UseAuthorization();
    ```

### <a name="azure-app-service-on-linux"></a>Servizio app di Azure in Linux

Specificare l'autorità emittente in modo esplicito durante la distribuzione nel servizio app Azure in Linux. Per altre informazioni, vedere <xref:security/authentication/identity/spa#azure-app-service-on-linux>.

### <a name="addapiauthorization"></a>AddApiAuthorization

Il <xref:Microsoft.Extensions.DependencyInjection.IdentityServerBuilderConfigurationExtensions.AddApiAuthorization%2A> metodo helper configura il [ Identity server](https://identityserver.io/) per gli scenari di ASP.NET Core. IdentityServer è un Framework potente ed estendibile per la gestione dei problemi di sicurezza delle app. IdentityIl server espone complessità non necessaria per gli scenari più comuni. Di conseguenza, viene fornito un set di convenzioni e opzioni di configurazione che consideriamo un valido punto di partenza. Una volta modificate le esigenze di autenticazione, è disponibile tutta la potenza del Identity Server per personalizzare l'autenticazione per soddisfare i requisiti di un'app.

### <a name="addidentityserverjwt"></a>Aggiungi Identity ServerJwt

Il <xref:Microsoft.AspNetCore.Authentication.AuthenticationBuilderExtensions.AddIdentityServerJwt%2A> metodo helper configura uno schema di criteri per l'app come gestore di autenticazione predefinito. Il criterio è configurato in modo da consentire Identity a di gestire tutte le richieste indirizzate a qualsiasi sottopercorso nello Identity spazio URL `/Identity` . <xref:Microsoft.AspNetCore.Authentication.JwtBearer.JwtBearerHandler>Gestisce tutte le altre richieste. Inoltre, questo metodo:

* Registra una `{APPLICATION NAME}API` risorsa API con Identity un server con un ambito predefinito `{APPLICATION NAME}API` .
* Configura il middleware del token di porta JWT per convalidare i token emessi dal Identity Server per l'app.

### <a name="weatherforecastcontroller"></a>WeatherForecastController

In `WeatherForecastController` ( `Controllers/WeatherForecastController.cs` ), l' [ `[Authorize]` attributo](xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute) viene applicato alla classe. L'attributo indica che l'utente deve essere autorizzato in base ai criteri predefiniti per accedere alla risorsa. I criteri di autorizzazione predefiniti sono configurati per l'utilizzo dello schema di autenticazione predefinito, impostato da <xref:Microsoft.AspNetCore.Authentication.AuthenticationBuilderExtensions.AddIdentityServerJwt%2A> . Il metodo helper viene configurato <xref:Microsoft.AspNetCore.Authentication.JwtBearer.JwtBearerHandler> come gestore predefinito per le richieste all'app.

### <a name="applicationdbcontext"></a>ApplicationDbContext

In `ApplicationDbContext` ( `Data/ApplicationDbContext.cs` ), <xref:Microsoft.EntityFrameworkCore.DbContext> estende <xref:Microsoft.AspNetCore.ApiAuthorization.IdentityServer.ApiAuthorizationDbContext%601> per includere lo schema per il Identity Server. L'oggetto <xref:Microsoft.AspNetCore.ApiAuthorization.IdentityServer.ApiAuthorizationDbContext%601> è derivato da <xref:Microsoft.AspNetCore.Identity.EntityFrameworkCore.IdentityDbContext>.

Per ottenere il controllo completo dello schema del database, ereditare da una delle Identity <xref:Microsoft.EntityFrameworkCore.DbContext> classi disponibili e configurare il contesto per includere lo Identity schema chiamando `builder.ConfigurePersistedGrantContext(_operationalStoreOptions.Value)` nel <xref:Microsoft.EntityFrameworkCore.DbContext.OnModelCreating%2A> metodo.

### <a name="oidcconfigurationcontroller"></a>OidcConfigurationController

In `OidcConfigurationController` ( `Controllers/OidcConfigurationController.cs` ), viene eseguito il provisioning dell'endpoint client per gestire i parametri OIDC.

### <a name="app-settings"></a>Impostazioni app

Nel file di impostazioni dell'app ( `appsettings.json` ) nella radice del progetto, la `IdentityServer` sezione descrive l'elenco dei client configurati. Nell'esempio seguente è presente un singolo client. Il nome del client corrisponde al nome dell'app e viene mappato per convenzione al `ClientId` parametro OAuth. Il profilo indica il tipo di app da configurare. Il profilo viene utilizzato internamente per guidare le convenzioni che semplificano il processo di configurazione per il server. <!-- There are several profiles available, as explained in the [Application profiles](#application-profiles) section. -->

```json
"IdentityServer": {
  "Clients": {
    "{APP ASSEMBLY}.Client": {
      "Profile": "IdentityServerSPA"
    }
  }
}
```

Il segnaposto `{APP ASSEMBLY}` è il nome dell'assembly dell'app (ad esempio, `BlazorSample.Client` ).

## <a name="client-app-configuration"></a>*`Client`* configurazione dell'app

### <a name="authentication-package"></a>Pacchetto di autenticazione

Quando viene creata un'app per l'uso di singoli account utente ( `Individual` ), l'app riceve automaticamente un riferimento al pacchetto [`Microsoft.AspNetCore.Components.WebAssembly.Authentication`](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.WebAssembly.Authentication) nel file di progetto dell'app. Il pacchetto fornisce un set di primitive che consentono all'app di autenticare gli utenti e ottenere i token per chiamare le API protette.

Se si aggiunge l'autenticazione a un'app, aggiungere manualmente il pacchetto al file di progetto dell'app:

```xml
<PackageReference 
  Include="Microsoft.AspNetCore.Components.WebAssembly.Authentication" 
  Version="{VERSION}" />
```

Per il segnaposto `{VERSION}` , la versione stabile più recente del pacchetto che corrisponde alla versione del Framework condiviso dell'app è disponibile nella cronologia delle **versioni** del pacchetto in [NuGet.org](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.WebAssembly.Authentication).

### <a name="httpclient-configuration"></a>`HttpClient` configurazione

In `Program.Main` ( `Program.cs` ), un denominato <xref:System.Net.Http.HttpClient> ( `HostIS.ServerAPI` ) viene configurato per fornire <xref:System.Net.Http.HttpClient> istanze che includono token di accesso quando si effettuano richieste all'API del server:

```csharp
builder.Services.AddHttpClient("HostIS.ServerAPI", 
        client => client.BaseAddress = new Uri(builder.HostEnvironment.BaseAddress))
    .AddHttpMessageHandler<BaseAddressAuthorizationMessageHandler>();

builder.Services.AddScoped(sp => sp.GetRequiredService<IHttpClientFactory>()
    .CreateClient("HostIS.ServerAPI"));
```

> [!NOTE]
> Se si sta configurando un' Blazor WebAssembly app per l'uso di un' Identity istanza del server esistente che non fa parte di una soluzione ospitata Blazor , modificare la <xref:System.Net.Http.HttpClient> registrazione dell'indirizzo di base da <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.IWebAssemblyHostEnvironment.BaseAddress?displayProperty=nameWithType> () nell'URL dell' `builder.HostEnvironment.BaseAddress` endpoint di autorizzazione dell'API dell'app Server.

### <a name="api-authorization-support"></a>Supporto dell'autorizzazione API

Il supporto per l'autenticazione degli utenti è collegato al contenitore del servizio mediante il metodo di estensione fornito nel [`Microsoft.AspNetCore.Components.WebAssembly.Authentication`](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.WebAssembly.Authentication) pacchetto. Questo metodo configura i servizi richiesti dall'app per interagire con il sistema di autorizzazione esistente.

```csharp
builder.Services.AddApiAuthorization();
```

Per impostazione predefinita, la configurazione per l'app viene caricata per convenzione da `_configuration/{client-id}` . Per convenzione, l'ID client è impostato sul nome dell'assembly dell'app. Questo URL può essere modificato in modo che punti a un endpoint separato chiamando l'overload con options.

### <a name="imports-file"></a>Importa file

[!INCLUDE[](~/blazor/includes/security/imports-file-hosted.md)]

### <a name="index-page"></a>Pagina di indice

[!INCLUDE[](~/blazor/includes/security/index-page-authentication.md)]

### <a name="app-component"></a>Componente app

[!INCLUDE[](~/blazor/includes/security/app-component.md)]

### <a name="redirecttologin-component"></a>Componente RedirectToLogin

[!INCLUDE[](~/blazor/includes/security/redirecttologin-component.md)]

### <a name="logindisplay-component"></a>Componente LoginDisplay

Il `LoginDisplay` componente ( `Shared/LoginDisplay.razor` ) viene sottoposto a rendering nel `MainLayout` componente ( `Shared/MainLayout.razor` ) e gestisce i comportamenti seguenti:

* Per utenti autenticati:
  * Visualizza il nome utente corrente.
  * Offre un collegamento alla pagina del profilo utente in ASP.NET Core Identity .
  * Offre un pulsante per disconnettersi dall'app.
* Per utenti anonimi:
  * Offre la possibilità di effettuare la registrazione.
  * Offre la possibilità di effettuare l'accesso.

```razor
@using Microsoft.AspNetCore.Components.Authorization
@using Microsoft.AspNetCore.Components.WebAssembly.Authentication
@inject NavigationManager Navigation
@inject SignOutSessionStateManager SignOutManager

<AuthorizeView>
    <Authorized>
        <a href="authentication/profile">Hello, @context.User.Identity.Name!</a>
        <button class="nav-link btn btn-link" @onclick="BeginSignOut">
            Log out
        </button>
    </Authorized>
    <NotAuthorized>
        <a href="authentication/register">Register</a>
        <a href="authentication/login">Log in</a>
    </NotAuthorized>
</AuthorizeView>

@code {
    private async Task BeginSignOut(MouseEventArgs args)
    {
        await SignOutManager.SetSignOutState();
        Navigation.NavigateTo("authentication/logout");
    }
}
```

### <a name="authentication-component"></a>Componente di autenticazione

[!INCLUDE[](~/blazor/includes/security/authentication-component.md)]

### <a name="fetchdata-component"></a>Componente FetchData

[!INCLUDE[](~/blazor/includes/security/fetchdata-component.md)]

## <a name="run-the-app"></a>Eseguire l'app

Eseguire l'app dal progetto server. Quando si usa Visual Studio, eseguire una delle operazioni seguenti:

* Impostare l'elenco a discesa **progetti di avvio** nella barra degli strumenti sull'app per le *API server* e selezionare il pulsante **Esegui** .
* Selezionare il progetto server in **Esplora soluzioni** , quindi fare clic sul pulsante **Esegui** nella barra degli strumenti o avviare l'app dal menu **debug** .

## <a name="name-and-role-claim-with-api-authorization"></a>Nome e attestazione di ruolo con autorizzazione API

### <a name="custom-user-factory"></a>Factory utente personalizzata

Nell' *`Client`* app creare una factory utente personalizzata. Identity Il server invia più ruoli come matrice JSON in un'unica `role` attestazione. Un singolo ruolo viene inviato come valore stringa nell'attestazione. La factory crea una singola `role` attestazione per ogni ruolo dell'utente.

`CustomUserFactory.cs`:

```csharp
using System.Linq;
using System.Security.Claims;
using System.Text.Json;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Components.WebAssembly.Authentication;
using Microsoft.AspNetCore.Components.WebAssembly.Authentication.Internal;

public class CustomUserFactory
    : AccountClaimsPrincipalFactory<RemoteUserAccount>
{
    public CustomUserFactory(IAccessTokenProviderAccessor accessor)
        : base(accessor)
    {
    }

    public async override ValueTask<ClaimsPrincipal> CreateUserAsync(
        RemoteUserAccount account,
        RemoteAuthenticationUserOptions options)
    {
        var user = await base.CreateUserAsync(account, options);

        if (user.Identity.IsAuthenticated)
        {
            var identity = (ClaimsIdentity)user.Identity;
            var roleClaims = identity.FindAll(identity.RoleClaimType).ToArray();

            if (roleClaims != null && roleClaims.Any())
            {
                foreach (var existingClaim in roleClaims)
                {
                    identity.RemoveClaim(existingClaim);
                }

                var rolesElem = account.AdditionalProperties[identity.RoleClaimType];

                if (rolesElem is JsonElement roles)
                {
                    if (roles.ValueKind == JsonValueKind.Array)
                    {
                        foreach (var role in roles.EnumerateArray())
                        {
                            identity.AddClaim(new Claim(options.RoleClaim, role.GetString()));
                        }
                    }
                    else
                    {
                        identity.AddClaim(new Claim(options.RoleClaim, roles.GetString()));
                    }
                }
            }
        }

        return user;
    }
}
```

Nell' *`Client`* app registrare la factory in `Program.Main` ( `Program.cs` ):

```csharp
builder.Services.AddApiAuthorization()
    .AddAccountClaimsPrincipalFactory<CustomUserFactory>();
```

Nell' *`Server`* app, chiamare <xref:Microsoft.AspNetCore.Identity.IdentityBuilder.AddRoles*> sul Identity Generatore, che aggiunge i servizi correlati ai ruoli:

```csharp
using Microsoft.AspNetCore.Identity;

...

services.AddDefaultIdentity<ApplicationUser>(options => 
    options.SignIn.RequireConfirmedAccount = true)
    .AddRoles<IdentityRole>()
    .AddEntityFrameworkStores<ApplicationDbContext>();
```

### <a name="configure-identity-server"></a>Configura Identity Server

Usare **uno** degli approcci seguenti:

* [Opzioni di autorizzazione API](#api-authorization-options)
* [Servizio profili](#profile-service)

#### <a name="api-authorization-options"></a>Opzioni di autorizzazione API

Nell' *`Server`* app:

* Configurare Identity il server in modo che inserisca le `name` `role` attestazioni e nel token ID e nel token di accesso.
* Impedire il mapping predefinito per i ruoli nel gestore del token JWT.

```csharp
using System.IdentityModel.Tokens.Jwt;
using System.Linq;

...

services.AddIdentityServer()
    .AddApiAuthorization<ApplicationUser, ApplicationDbContext>(options => {
        options.IdentityResources["openid"].UserClaims.Add("name");
        options.ApiResources.Single().UserClaims.Add("name");
        options.IdentityResources["openid"].UserClaims.Add("role");
        options.ApiResources.Single().UserClaims.Add("role");
    });

JwtSecurityTokenHandler.DefaultInboundClaimTypeMap.Remove("role");
```

#### <a name="profile-service"></a>Servizio profili

Nell' *`Server`* app creare un' `ProfileService` implementazione.

`ProfileService.cs`:

```csharp
using IdentityModel;
using IdentityServer4.Models;
using IdentityServer4.Services;
using System.Threading.Tasks;

public class ProfileService : IProfileService
{
    public ProfileService()
    {
    }

    public async Task GetProfileDataAsync(ProfileDataRequestContext context)
    {
        var nameClaim = context.Subject.FindAll(JwtClaimTypes.Name);
        context.IssuedClaims.AddRange(nameClaim);

        var roleClaims = context.Subject.FindAll(JwtClaimTypes.Role);
        context.IssuedClaims.AddRange(roleClaims);

        await Task.CompletedTask;
    }

    public async Task IsActiveAsync(IsActiveContext context)
    {
        await Task.CompletedTask;
    }
}
```

Nell' *`Server`* app registrare il servizio profili in `Startup.ConfigureServices` :

```csharp
using IdentityServer4.Services;

...

services.AddTransient<IProfileService, ProfileService>();
```

### <a name="use-authorization-mechanisms"></a>Usare i meccanismi di autorizzazione

Nell' *`Client`* app gli approcci di autorizzazione dei componenti sono funzionali a questo punto. Qualsiasi meccanismo di autorizzazione nei componenti può utilizzare un ruolo per autorizzare l'utente:

* [ `AuthorizeView` Component](xref:blazor/security/index#authorizeview-component) (esempio: `<AuthorizeView Roles="admin">` )
* [ `[Authorize]` direttiva attribute](xref:blazor/security/index#authorize-attribute) ( <xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute> ) (esempio: `@attribute [Authorize(Roles = "admin")]` )
* [Logica procedurale](xref:blazor/security/index#procedural-logic) (esempio: `if (user.IsInRole("admin")) { ... }` )

  Sono supportati più test di ruolo:

  ```csharp
  if (user.IsInRole("admin") && user.IsInRole("developer"))
  {
      ...
  }
  ```

`User.Identity.Name` viene popolato nell' *`Client`* app con il nome utente dell'utente, che in genere è l'indirizzo di posta elettronica di accesso.

[!INCLUDE[](~/blazor/includes/security/usermanager-signinmanager.md)]

## <a name="host-in-azure-app-service-with-a-custom-domain-and-certificate"></a>Host in app Azure servizio con un dominio personalizzato e un certificato

Le linee guida seguenti illustrano:

* Come distribuire un'app ospitata Blazor WebAssembly con Identity Server per [app Azure servizio](https://azure.microsoft.com/services/app-service/) con un dominio personalizzato.
* Come creare e usare un certificato TLS per la comunicazione del protocollo HTTPS con i browser. Sebbene le linee guida siano incentrate sull'uso del certificato con un dominio personalizzato, le linee guida sono ugualmente applicabili all'uso di un dominio di app di Azure predefinito, ad esempio `contoso.azurewebsites.net` .

Per questo scenario di hosting, **non** usare lo stesso certificato per la [ Identity chiave di firma del token del server](https://docs.identityserver.io/en/latest/topics/crypto.html#token-signing-and-validation) e la comunicazione protetta HTTPS del sito con i browser:

* L'uso di certificati diversi per questi due requisiti è una procedura di sicurezza efficace, in quanto isola le chiavi private per ogni scopo.
* I certificati TLS per la comunicazione con i browser vengono gestiti in modo indipendente senza influire sulla Identity firma dei token del server.
* Quando [Azure Key Vault](https://azure.microsoft.com/services/key-vault/) fornisce un certificato a un'app del servizio app per l'associazione di dominio personalizzata, Identity il server non può ottenere lo stesso certificato da Azure Key Vault per la firma di token. Sebbene Identity sia possibile configurare il server per l'utilizzo dello stesso certificato TLS da un percorso fisico, l'inserimento dei certificati di sicurezza nel controllo del codice sorgente non è una **procedura consigliata e deve essere evitato nella maggior parte degli scenari**.

Nelle linee guida seguenti viene creato un certificato autofirmato in Azure Key Vault esclusivamente per la Identity firma di token del server. La Identity configurazione del server usa il certificato dell'insieme di credenziali delle chiavi tramite l' `My`  >  `CurrentUser` archivio certificati dell'app. Altri certificati utilizzati per il traffico HTTPS con domini personalizzati vengono creati e configurati separatamente dal Identity certificato di firma del server.

Per configurare un'app, app Azure servizio e Azure Key Vault per ospitare con un dominio personalizzato e HTTPS:

1. Creare un [piano di servizio app](/azure/app-service/overview-hosting-plans) con un livello di piano `Basic B1` o superiore. Il servizio app richiede un `Basic B1` livello di servizio o superiore per l'uso di domini personalizzati.
1. Creare un certificato PFX per il protocollo HTTPS (Secure Browser Communication) del sito con un nome comune del nome di dominio completo (FQDN) del sito controllato dall'organizzazione (ad esempio, `www.contoso.com` ). Creare il certificato con:
   * Usi chiave
     * Convalida della firma digitale ( `digitalSignature` )
     * Crittografia chiave ( `keyEncipherment` )
   * Usi chiave avanzata/estesa
     * Autenticazione client (1.3.6.1.5.5.7.3.2)
     * Autenticazione server (1.3.6.1.5.5.7.3.1)

   Per creare il certificato, usare uno degli approcci seguenti o qualsiasi altro strumento o servizio online appropriato:

   * [Azure Key Vault](/azure/key-vault/certificates/quick-create-portal#add-a-certificate-to-key-vault)
   * [MakeCert per Windows](/windows/desktop/seccrypto/makecert)
   * [OpenSSL](https://www.openssl.org)

   Prendere nota della password, che verrà usata in seguito per importare il certificato in Azure Key Vault.

   Per ulteriori informazioni sui certificati Azure Key Vault, vedere [Azure Key Vault: Certificates](/azure/key-vault/certificates/).
1. Creare una nuova Azure Key Vault o usare un insieme di credenziali delle chiavi esistente nella sottoscrizione di Azure.
1. Nell'area **certificati** dell'insieme di credenziali delle chiavi importare il certificato del sito PFX. Registrare l'identificazione personale del certificato, che viene usato nella configurazione dell'app in un secondo momento.
1. In Azure Key Vault generare un nuovo certificato autofirmato per la Identity firma del token del server. Assegnare al certificato un **nome** e un **oggetto** del certificato. Il **soggetto** viene specificato come `CN={COMMON NAME}` , dove il `{COMMON NAME}` segnaposto è il nome comune del certificato. Il nome comune può essere qualsiasi stringa alfanumerica. Ad esempio, `CN=IdentityServerSigning` è un **oggetto** certificato valido. Usare le impostazioni di **configurazione avanzata dei criteri** predefiniti. Registrare l'identificazione personale del certificato, che viene usato nella configurazione dell'app in un secondo momento.
1. Passare a app Azure servizio nel portale di Azure e creare un nuovo servizio app con la configurazione seguente:
   * **Pubblica** impostare su `Code` .
   * **Stack di runtime** impostato sul runtime dell'app.
   * Per **SKU e dimensioni**, verificare che il livello di servizio app sia `Basic B1` o superiore.  Il servizio app richiede un `Basic B1` livello di servizio o superiore per l'uso di domini personalizzati.
1. Dopo che Azure ha creato il servizio app, aprire la **configurazione** dell'app e aggiungere una nuova impostazione dell'applicazione specificando le identificazioni personali del certificato registrate in precedenza. La chiave dell'impostazione dell'app è `WEBSITE_LOAD_CERTIFICATES` . Separare le identificazioni personali del certificato nel valore dell'impostazione dell'app con una virgola, come illustrato nell'esempio seguente:
   * Chiave: `WEBSITE_LOAD_CERTIFICATES`
   * Valore: `57443A552A46DB...D55E28D412B943565,29F43A772CB6AF...1D04F0C67F85FB0B1`

   Nel portale di Azure il salvataggio delle impostazioni dell'app è un processo in due passaggi: salvare l' `WEBSITE_LOAD_CERTIFICATES` impostazione chiave-valore, quindi selezionare il pulsante **Salva** nella parte superiore del pannello.
1. Selezionare le **Impostazioni TLS/SSL** dell'app. Selezionare **certificati di chiave privata (con estensione pfx)**. Usare il processo **Import Key Vault certificate** due volte per importare il certificato del sito per la comunicazione HTTPS e il certificato di firma del token del server autofirmato del sito Identity .
1. Passare al pannello **domini personalizzati** . Nel sito Web del registrar di dominio usare l' **indirizzo IP** e l' **ID di verifica del dominio personalizzato** per configurare il dominio. Una configurazione di dominio tipica include:
   * Un **record a** con un **host** di `@` e un valore dell'indirizzo IP del portale di Azure.
   * Un **record TXT** con un **host** di `asuid` e il valore dell'ID di verifica generato da Azure e fornito dal portale di Azure.

   Assicurarsi di salvare correttamente le modifiche nel sito Web del registrar di dominio. Per alcuni siti Web del registrar è necessario un processo in due passaggi per salvare i record di dominio: uno o più record vengono salvati singolarmente seguito dall'aggiornamento della registrazione del dominio con un pulsante separato.
1. Tornare al pannello **domini personalizzati** nel portale di Azure. Selezionare **Aggiungi dominio personalizzato**. Selezionare l'opzione **A record** . Specificare il dominio e selezionare **convalida**. Se i record di dominio sono corretti e propagati in Internet, il portale consente di selezionare il pulsante **Aggiungi dominio personalizzato** .

   Potrebbero essere necessari alcuni giorni per la propagazione delle modifiche di registrazione del dominio tra i server dei nomi di dominio Internet (DNS) dopo che sono state elaborate dal registrar. Se i record di dominio non vengono aggiornati entro tre giorni lavorativi, verificare che i record siano impostati correttamente con il registrar e contattare il supporto tecnico.
1. Nel pannello **domini personalizzati** lo **stato SSL** per il dominio è contrassegnato `Not Secure` . Selezionare il collegamento **Aggiungi binding** . Selezionare il certificato HTTPS del sito dall'insieme di credenziali delle chiavi per l'associazione del dominio personalizzato.
1. In Visual Studio aprire il file di impostazioni dell'app del progetto *Server* ( `appsettings.json` o `appsettings.Production.json` ). Nella Identity configurazione del server aggiungere la sezione seguente `Key` . Specificare il certificato autofirmato **soggetto** alla `Name` chiave. Nell'esempio seguente il nome comune del certificato assegnato nell'insieme di credenziali delle chiavi è `IdentityServerSigning` , che restituisce un **oggetto** `CN=IdentityServerSigning` :

   ```json
   "IdentityServer": {

     ...

     "Key": {
       "Type": "Store",
       "StoreName": "My",
       "StoreLocation": "CurrentUser",
       "Name": "CN=IdentityServerSigning"
     }
   },
   ```

1. In Visual Studio creare un [profilo di pubblicazione](xref:host-and-deploy/visual-studio-publish-profiles#publish-profiles) del servizio app Azure per il progetto *Server* . Dalla barra dei menu selezionare: **Compila**  >  **pubblica**  >  **nuovo**  >    >  **servizio app Azure** di Azure (Windows o Linux). Quando Visual Studio è connesso a una sottoscrizione di Azure, è possibile impostare la **visualizzazione** delle risorse di Azure in base al **tipo di risorsa**. Spostarsi all'interno dell'elenco di **app Web** per trovare il servizio app per l'app e selezionarlo. Selezionare **Fine**.
1. Quando Visual Studio torna alla finestra di **pubblicazione** , le dipendenze del servizio Key vault e SQL Server database vengono rilevate automaticamente.

   Non sono necessarie modifiche di configurazione alle impostazioni predefinite per il servizio Key Vault.

   A scopo di test, il database [SQLite](https://www.sqlite.org/index.html) locale di un'app, configurato per impostazione predefinita dal Blazor modello, può essere distribuito con l'app senza alcuna configurazione aggiuntiva. La configurazione di un database diverso per Identity il server in produzione esula dall'ambito di questo articolo. Per ulteriori informazioni, vedere le risorse di database nei set di documentazione seguenti:
   * [Servizio app](/azure/app-service/)
   * [Identity Server](https://identityserver4.readthedocs.io/en/latest/)

1. Selezionare il collegamento **modifica** sotto il nome del profilo di distribuzione nella parte superiore della finestra. Modificare l'URL di destinazione sull'URL del dominio personalizzato del sito, ad esempio `https://www.contoso.com` . Salvare le impostazioni.
1. Pubblicare l'app. Visual Studio apre una finestra del browser e richiede il sito nel dominio personalizzato.

La documentazione di Azure contiene ulteriori dettagli sull'uso di servizi di Azure e domini personalizzati con binding TLS nel servizio app, incluse informazioni sull'uso di record CNAME invece di record. Per altre informazioni, vedere le risorse seguenti:

* [Documentazione del servizio app](/azure/app-service/)
* [Esercitazione: Eseguire il mapping di un nome DNS personalizzato esistente al Servizio app di Azure](/azure/app-service/app-service-web-tutorial-custom-domain)
* [Proteggere un nome DNS personalizzato con un'associazione TLS/SSL nel Servizio app di Azure](/azure/app-service/configure-ssl-bindings)
* [Azure Key Vault](/azure/key-vault/)

È consigliabile usare una nuova finestra del browser privata o in incognito per ogni esecuzione dei test dell'app dopo una modifica all'app, alla configurazione dell'app o ai servizi di Azure nella portale di Azure. I residui cookie di un'esecuzione dei test precedente possono causare l'autenticazione o l'autorizzazione non riuscita durante il test del sito anche quando la configurazione del sito è corretta. Per ulteriori informazioni su come configurare Visual Studio per aprire una nuova finestra del browser in modalità privata o in incognito per ogni esecuzione dei test, vedere la sezione relativa ai [ Cookie dati del sito e](#cookies-and-site-data) di.

Quando la configurazione del servizio app viene modificata nella portale di Azure, gli aggiornamenti sono in genere effettivi, ma non sono istantanei. In alcuni casi, è necessario attendere un breve periodo di tempo per il riavvio di un servizio app, in modo da rendere effettiva una modifica della configurazione.

Per risolvere un problema di caricamento del certificato, eseguire il comando seguente in una shell dei comandi di PowerShell portale di Azure [Kudu](https://github.com/projectkudu/kudu/wiki/Accessing-the-kudu-service) . Il comando fornisce un elenco di certificati a cui l'app può accedere dall' `My`  >  `CurrentUser` archivio certificati. L'output include gli oggetti certificato e le identificazioni personali utili durante il debug di un'app:

```powershell
Get-ChildItem -path Cert:\CurrentUser\My -Recurse | Format-List DnsNameList, Subject, Thumbprint, EnhancedKeyUsageList
```

[!INCLUDE[](~/blazor/includes/security/troubleshoot.md)]

## <a name="additional-resources"></a>Risorse aggiuntive

* [Distribuzione nel servizio app Azure](xref:security/authentication/identity/spa#deploy-to-production)
* [Importare un certificato da Key Vault (documentazione di Azure)](/azure/app-service/configure-ssl-certificate#import-a-certificate-from-key-vault)
* <xref:blazor/security/webassembly/additional-scenarios>
* [Richieste API Web non autenticate o non autorizzate in un'app con un client predefinito sicuro](xref:blazor/security/webassembly/additional-scenarios#unauthenticated-or-unauthorized-web-api-requests-in-an-app-with-a-secure-default-client)
* <xref:host-and-deploy/proxy-load-balancer>: Include informazioni aggiuntive su:
  * Uso del middleware intestazioni con inoltro per mantenere le informazioni sullo schema HTTPS tra i server proxy e le reti interne.
  * Scenari e casi d'uso aggiuntivi, tra cui la configurazione manuale dello schema, la modifica del percorso della richiesta per il routing delle richieste corretto e l'inoltro dello schema di richiesta per i proxy inversi Linux e non IIS.
