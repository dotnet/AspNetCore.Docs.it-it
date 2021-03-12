---
title: Mantieni attestazioni e token aggiuntivi da provider esterni in ASP.NET Core
author: rick-anderson
description: Informazioni su come creare attestazioni e token aggiuntivi da provider esterni.
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 02/18/2021
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
uid: security/authentication/social/additional-claims
ms.openlocfilehash: 513de6133455e26552b79de0f07cf135e8b36825
ms.sourcegitcommit: acfe51c35497a204f75c2a61125c9408c04493e6
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102605568"
---
# <a name="persist-additional-claims-and-tokens-from-external-providers-in-aspnet-core"></a>Mantieni attestazioni e token aggiuntivi da provider esterni in ASP.NET Core

::: moniker range=">= aspnetcore-3.0"

Un'app ASP.NET Core può creare attestazioni e token aggiuntivi da provider di autenticazione esterni, ad esempio Facebook, Google, Microsoft e Twitter. Ogni provider rivela informazioni diverse sugli utenti sulla propria piattaforma, ma il modello per la ricezione e la trasformazione dei dati utente in attestazioni aggiuntive è lo stesso.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/authentication/social/additional-claims/samples) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="prerequisites"></a>Prerequisiti

Decidere quali provider di autenticazione esterni supportare nell'app. Per ogni provider, registrare l'app e ottenere un ID client e un segreto client. Per altre informazioni, vedere <xref:security/authentication/social/index>. L'app di esempio usa il [provider di autenticazione Google](xref:security/authentication/google-logins).

## <a name="set-the-client-id-and-client-secret"></a>Impostare l'ID client e il segreto client

Il provider di autenticazione OAuth stabilisce una relazione di trust con un'app che usa un ID client e un segreto client. I valori relativi all'ID client e al segreto client vengono creati per l'app dal provider di autenticazione esterno quando l'app viene registrata con il provider. Ogni provider esterno usato dall'app deve essere configurato in modo indipendente con l'ID client e il segreto client del provider. Per ulteriori informazioni, vedere gli argomenti del provider di autenticazione esterno applicabili allo scenario:

* [Autenticazione Facebook](xref:security/authentication/facebook-logins)
* [Autenticazione Google](xref:security/authentication/google-logins)
* [Autenticazione Microsoft](xref:security/authentication/microsoft-logins)
* [Autenticazione Twitter](xref:security/authentication/twitter-logins)
* [Altri provider di autenticazione](xref:security/authentication/otherlogins)
* [OpenIdConnect](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2)

L'app di esempio configura il provider di autenticazione Google con un ID client e un segreto client forniti da Google:

[!code-csharp[](additional-claims/samples/3.x/ClaimsSample/Startup.cs?name=snippet_AddGoogle&highlight=4,9)]

## <a name="establish-the-authentication-scope"></a>Definire l'ambito di autenticazione

Specificare l'elenco di autorizzazioni da recuperare dal provider specificando <xref:Microsoft.AspNetCore.Authentication.OAuth.OAuthOptions.Scope*> . Gli ambiti di autenticazione per i provider esterni comuni sono riportati nella tabella seguente.

| Provider  | Ambito                                                            |
| --------- | ---------------------------------------------------------------- |
| Facebook  | `https://www.facebook.com/dialog/oauth`                          |
| Google    | `profile`, `email`, `openid`                                     |
| Microsoft | `https://login.microsoftonline.com/common/oauth2/v2.0/authorize` |
| Twitter   | `https://api.twitter.com/oauth/authenticate`                     |

Nell'app di esempio gli `profile` ambiti di Google, `email` e `openid` vengono aggiunti automaticamente dal framework quando <xref:Microsoft.Extensions.DependencyInjection.GoogleExtensions.AddGoogle%2A> viene chiamato su <xref:Microsoft.AspNetCore.Authentication.AuthenticationBuilder> . Se l'app richiede ambiti aggiuntivi, aggiungerli alle opzioni. Nell'esempio seguente `https://www.googleapis.com/auth/user.birthday.read` viene aggiunto l'ambito Google per recuperare il compleanno di un utente:

```csharp
options.Scope.Add("https://www.googleapis.com/auth/user.birthday.read");
```

## <a name="map-user-data-keys-and-create-claims"></a>Eseguire il mapping delle chiavi di dati utente e creare attestazioni

Nelle opzioni del provider specificare un <xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionMapExtensions.MapJsonKey*> o <xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionMapExtensions.MapJsonSubKey*> per ogni chiave/sottochiave nei dati utente JSON del provider esterno per l'identità dell'app da leggere durante l'accesso. Per ulteriori informazioni sui tipi di attestazione, vedere <xref:System.Security.Claims.ClaimTypes> .

L'app di esempio crea attestazioni delle impostazioni locali ( `urn:google:locale` ) e immagini ( `urn:google:picture` ) dalle `locale` `picture` chiavi e nei dati utente di Google:

[!code-csharp[](additional-claims/samples/3.x/ClaimsSample/Startup.cs?name=snippet_AddGoogle&highlight=13-14)]

In `Microsoft.AspNetCore.Identity.UI.Pages.Account.Internal.ExternalLoginModel.OnPostConfirmationAsync` , un <xref:Microsoft.AspNetCore.Identity.IdentityUser> ( `ApplicationUser` ) è connesso all'app con <xref:Microsoft.AspNetCore.Identity.SignInManager%601.SignInAsync*> . Durante il processo di accesso, il <xref:Microsoft.AspNetCore.Identity.UserManager%601> può archiviare `ApplicationUser` attestazioni per i dati utente disponibili da <xref:Microsoft.AspNetCore.Identity.ExternalLoginInfo.Principal*> .

Nell'app di esempio `OnPostConfirmationAsync` (*account/ExternalLogin. cshtml. cs*) vengono stabilite le attestazioni delle impostazioni locali ( `urn:google:locale` ) e immagine ( `urn:google:picture` ) per l'accesso `ApplicationUser` , inclusa un'attestazione per <xref:System.Security.Claims.ClaimTypes.GivenName> :

[!code-csharp[](additional-claims/samples/3.x/ClaimsSample/Areas/Identity/Pages/Account/ExternalLogin.cshtml.cs?name=snippet_OnPostConfirmationAsync&highlight=35-51)]

Per impostazione predefinita, le attestazioni di un utente vengono archiviate nell'autenticazione di cookie . Se l'autenticazione cookie è troppo grande, può causare un errore dell'app perché:

* Il browser rileva che l' cookie intestazione è troppo lungo.
* La dimensione complessiva della richiesta è troppo grande.

Se per l'elaborazione delle richieste utente è richiesta una grande quantità di dati utente:

* Limitare il numero e le dimensioni delle attestazioni utente per l'elaborazione delle richieste solo ai componenti richiesti dall'app.
* Usare un personalizzato <xref:Microsoft.AspNetCore.Authentication.Cookies.ITicketStore> per il Cookie middleware di autenticazione <xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationOptions.SessionStore> per archiviare l'identità tra le richieste. Mantenere grandi quantità di informazioni sull'identità sul server inviando al client solo una chiave di identificatore di sessione di piccole dimensioni.

## <a name="save-the-access-token"></a>Salvare il token di accesso

<xref:Microsoft.AspNetCore.Authentication.RemoteAuthenticationOptions.SaveTokens*> definisce se i token di accesso e di aggiornamento devono essere archiviati in <xref:Microsoft.AspNetCore.Http.Authentication.AuthenticationProperties> dopo un'autorizzazione riuscita. `SaveTokens` è impostato su per `false` impostazione predefinita per ridurre le dimensioni dell'autenticazione finale cookie .

L'app di esempio imposta il valore di `SaveTokens` su `true` in <xref:Microsoft.AspNetCore.Authentication.Google.GoogleOptions> :

[!code-csharp[](additional-claims/samples/3.x/ClaimsSample/Startup.cs?name=snippet_AddGoogle&highlight=15)]

Quando `OnPostConfirmationAsync` viene eseguito, archivia il token di accesso ([ExternalLoginInfo. AuthenticationTokens](xref:Microsoft.AspNetCore.Identity.ExternalLoginInfo.AuthenticationTokens*)) dal provider esterno in `ApplicationUser` `AuthenticationProperties` .

L'app di esempio salva il token di accesso in `OnPostConfirmationAsync` (nuova registrazione utente) e `OnGetCallbackAsync` (utente registrato in precedenza) in *account/ExternalLogin. cshtml. cs*:

[!code-csharp[](additional-claims/samples/3.x/ClaimsSample/Areas/Identity/Pages/Account/ExternalLogin.cshtml.cs?name=snippet_OnPostConfirmationAsync&highlight=54-56)]

> [!NOTE]
> Per informazioni sul passaggio dei token ai Razor componenti di un' Blazor Server app, vedere <xref:blazor/security/server/additional-scenarios#pass-tokens-to-a-blazor-server-app> .

## <a name="how-to-add-additional-custom-tokens"></a>Come aggiungere token personalizzati aggiuntivi

Per illustrare come aggiungere un token personalizzato, archiviato come parte di `SaveTokens` , l'app di esempio aggiunge un oggetto <xref:Microsoft.AspNetCore.Authentication.AuthenticationToken> con l'oggetto corrente <xref:System.DateTime> per un [AuthenticationToken.Name](xref:Microsoft.AspNetCore.Authentication.AuthenticationToken.Name*) di `TicketCreated` :

[!code-csharp[](additional-claims/samples/3.x/ClaimsSample/Startup.cs?name=snippet_AddGoogle&highlight=17-30)]

## <a name="creating-and-adding-claims"></a>Creazione e aggiunta di attestazioni

Il Framework fornisce azioni comuni e metodi di estensione per la creazione e l'aggiunta di attestazioni alla raccolta. Per altre informazioni, vedere <xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionMapExtensions> e <xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionUniqueExtensions>.

Gli utenti possono definire azioni personalizzate derivando da <xref:Microsoft.AspNetCore.Authentication.OAuth.Claims.ClaimAction> e implementando il <xref:Microsoft.AspNetCore.Authentication.OAuth.Claims.ClaimAction.Run*> metodo astratto.

Per altre informazioni, vedere <xref:Microsoft.AspNetCore.Authentication.OAuth.Claims>.

## <a name="add-and-update-user-claims"></a>Aggiungi e aggiorna attestazioni utente

Le attestazioni vengono copiate dai provider esterni al database utente alla prima registrazione, non all'accesso. Se in un'app sono abilitate altre attestazioni dopo che un utente si è registrato per l'uso dell'app, chiamare [SignInManager. RefreshSignInAsync](xref:Microsoft.AspNetCore.Identity.SignInManager%601) su un utente per forzare la generazione di una nuova autenticazione cookie .

Nell'ambiente di sviluppo che utilizza gli account utente di test, è possibile eliminare e ricreare semplicemente l'account utente. Per i sistemi di produzione, le nuove attestazioni aggiunte all'app possono essere inserite in account utente. Dopo aver eseguito [il ponteggi della `ExternalLogin` pagina](xref:security/authentication/scaffold-identity) nell'app in `Areas/Pages/Identity/Account/Manage` , aggiungere il codice seguente a `ExternalLoginModel` nel `ExternalLogin.cshtml.cs` file.

Aggiungere un dizionario di attestazioni aggiuntive. Usare le chiavi del dizionario per conservare i tipi di attestazione e usare i valori per mantenere un valore predefinito. Aggiungere la riga seguente all'inizio della classe. Nell'esempio seguente si presuppone che venga aggiunta un'attestazione per l'immagine Google dell'utente con un'immagine Headshot generica come valore predefinito:

```csharp
private readonly IReadOnlyDictionary<string, string> _claimsToSync = 
    new Dictionary<string, string>()
    {
        { "urn:google:picture", "https://localhost:5001/headshot.png" },
    };
```

Sostituire il codice predefinito del `OnGetCallbackAsync` metodo con il codice seguente. Il codice esegue il ciclo nel dizionario delle attestazioni. Le attestazioni vengono aggiunte (riempite) o aggiornate per ogni utente. Quando si aggiungono o si aggiornano le attestazioni, l'accesso utente viene aggiornato utilizzando il <xref:Microsoft.AspNetCore.Identity.SignInManager%601> , mantenendo le proprietà di autenticazione esistenti ( `AuthenticationProperties` ).

```csharp
public async Task<IActionResult> OnGetCallbackAsync(
    string returnUrl = null, string remoteError = null)
{
    returnUrl = returnUrl ?? Url.Content("~/");

    if (remoteError != null)
    {
        ErrorMessage = $"Error from external provider: {remoteError}";

        return RedirectToPage("./Login", new {ReturnUrl = returnUrl });
    }

    var info = await _signInManager.GetExternalLoginInfoAsync();

    if (info == null)
    {
        ErrorMessage = "Error loading external login information.";
        return RedirectToPage("./Login", new { ReturnUrl = returnUrl });
    }

    // Sign in the user with this external login provider if the user already has a 
    // login.
    var result = await _signInManager.ExternalLoginSignInAsync(info.LoginProvider, 
        info.ProviderKey, isPersistent: false, bypassTwoFactor : true);

    if (result.Succeeded)
    {
        _logger.LogInformation("{Name} logged in with {LoginProvider} provider.", 
            info.Principal.Identity.Name, info.LoginProvider);

        if (_claimsToSync.Count > 0)
        {
            var user = await _userManager.FindByLoginAsync(info.LoginProvider, 
                info.ProviderKey);
            var userClaims = await _userManager.GetClaimsAsync(user);
            bool refreshSignIn = false;

            foreach (var addedClaim in _claimsToSync)
            {
                var userClaim = userClaims
                    .FirstOrDefault(c => c.Type == addedClaim.Key);

                if (info.Principal.HasClaim(c => c.Type == addedClaim.Key))
                {
                    var externalClaim = info.Principal.FindFirst(addedClaim.Key);

                    if (userClaim == null)
                    {
                        await _userManager.AddClaimAsync(user, 
                            new Claim(addedClaim.Key, externalClaim.Value));
                        refreshSignIn = true;
                    }
                    else if (userClaim.Value != externalClaim.Value)
                    {
                        await _userManager
                            .ReplaceClaimAsync(user, userClaim, externalClaim);
                        refreshSignIn = true;
                    }
                }
                else if (userClaim == null)
                {
                    // Fill with a default value
                    await _userManager.AddClaimAsync(user, new Claim(addedClaim.Key, 
                        addedClaim.Value));
                    refreshSignIn = true;
                }
            }

            if (refreshSignIn)
            {
                await _signInManager.RefreshSignInAsync(user);
            }
        }

        return LocalRedirect(returnUrl);
    }

    if (result.IsLockedOut)
    {
        return RedirectToPage("./Lockout");
    }
    else
    {
        // If the user does not have an account, then ask the user to create an 
        // account.
        ReturnUrl = returnUrl;
        ProviderDisplayName = info.ProviderDisplayName;

        if (info.Principal.HasClaim(c => c.Type == ClaimTypes.Email))
        {
            Input = new InputModel
            {
                Email = info.Principal.FindFirstValue(ClaimTypes.Email)
            };
        }

        return Page();
    }
}
```

Un approccio simile viene eseguito quando le attestazioni cambiano mentre un utente ha eseguito l'accesso, ma non è necessario un passaggio di recupero. Per aggiornare le attestazioni di un utente, chiamare quanto segue sull'utente:

* [Usermanager. ReplaceClaimAsync](xref:Microsoft.AspNetCore.Identity.UserManager%601) per l'utente per le attestazioni archiviate nel database di identità.
* [SignInManager. RefreshSignInAsync](xref:Microsoft.AspNetCore.Identity.SignInManager%601) sull'utente per forzare la generazione di una nuova autenticazione cookie .

## <a name="removal-of-claim-actions-and-claims"></a>Rimozione di azioni attestazioni e attestazioni

[ClaimActionCollection. Remove (String)](xref:Microsoft.AspNetCore.Authentication.OAuth.Claims.ClaimActionCollection.Remove*) rimuove tutte le azioni di attestazione per l'oggetto specificato <xref:Microsoft.AspNetCore.Authentication.OAuth.Claims.ClaimAction.ClaimType> dalla raccolta. [ClaimActionCollectionMapExtensions. DeleteClaim (ClaimActionCollection, String)](xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionMapExtensions.DeleteClaim*) Elimina un'attestazione dell'oggetto specificato <xref:Microsoft.AspNetCore.Authentication.OAuth.Claims.ClaimAction.ClaimType> dall'identità. <xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionMapExtensions.DeleteClaim*> viene utilizzato principalmente con [OpenID Connect (OIDC)](/azure/active-directory/develop/v2-protocols-oidc) per rimuovere le attestazioni generate dal protocollo.

## <a name="sample-app-output"></a>Esempio di output dell'app

```
User Claims

http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier
    9b342344f-7aab-43c2-1ac1-ba75912ca999
http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name
    someone@gmail.com
AspNet.Identity.SecurityStamp
    7D4312MOWRYYBFI1KXRPHGOSTBVWSFDE
http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname
    Judy
urn:google:locale
    en
urn:google:picture
    https://lh4.googleusercontent.com/-XXXXXX/XXXXXX/XXXXXX/XXXXXX/photo.jpg

Authentication Properties

.Token.access_token
    yc23.AlvoZqz56...1lxltXV7D-ZWP9
.Token.token_type
    Bearer
.Token.expires_at
    2019-04-11T22:14:51.0000000+00:00
.Token.TicketCreated
    4/11/2019 9:14:52 PM
.TokenNames
    access_token;token_type;expires_at;TicketCreated
.persistent
.issued
    Thu, 11 Apr 2019 20:51:06 GMT
.expires
    Thu, 25 Apr 2019 20:51:06 GMT

```

[!INCLUDE[Forward request information when behind a proxy or load balancer section](includes/forwarded-headers-middleware.md)]

::: moniker-end

::: moniker range="< aspnetcore-3.0"

Un'app ASP.NET Core può creare attestazioni e token aggiuntivi da provider di autenticazione esterni, ad esempio Facebook, Google, Microsoft e Twitter. Ogni provider rivela informazioni diverse sugli utenti sulla propria piattaforma, ma il modello per la ricezione e la trasformazione dei dati utente in attestazioni aggiuntive è lo stesso.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/authentication/social/additional-claims/samples) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="prerequisites"></a>Prerequisiti

Decidere quali provider di autenticazione esterni supportare nell'app. Per ogni provider, registrare l'app e ottenere un ID client e un segreto client. Per altre informazioni, vedere <xref:security/authentication/social/index>. L'app di esempio usa il [provider di autenticazione Google](xref:security/authentication/google-logins).

## <a name="set-the-client-id-and-client-secret"></a>Impostare l'ID client e il segreto client

Il provider di autenticazione OAuth stabilisce una relazione di trust con un'app che usa un ID client e un segreto client. I valori relativi all'ID client e al segreto client vengono creati per l'app dal provider di autenticazione esterno quando l'app viene registrata con il provider. Ogni provider esterno usato dall'app deve essere configurato in modo indipendente con l'ID client e il segreto client del provider. Per ulteriori informazioni, vedere gli argomenti del provider di autenticazione esterno applicabili allo scenario:

* [Autenticazione Facebook](xref:security/authentication/facebook-logins)
* [Autenticazione Google](xref:security/authentication/google-logins)
* [Autenticazione Microsoft](xref:security/authentication/microsoft-logins)
* [Autenticazione Twitter](xref:security/authentication/twitter-logins)
* [Altri provider di autenticazione](xref:security/authentication/otherlogins)
* [OpenIdConnect](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2)

L'app di esempio configura il provider di autenticazione Google con un ID client e un segreto client forniti da Google:

[!code-csharp[](additional-claims/samples/2.x/ClaimsSample/Startup.cs?name=snippet_AddGoogle&highlight=4,9)]

## <a name="establish-the-authentication-scope"></a>Definire l'ambito di autenticazione

Specificare l'elenco di autorizzazioni da recuperare dal provider specificando <xref:Microsoft.AspNetCore.Authentication.OAuth.OAuthOptions.Scope*> . Gli ambiti di autenticazione per i provider esterni comuni sono riportati nella tabella seguente.

| Provider  | Ambito                                                            |
| --------- | ---------------------------------------------------------------- |
| Facebook  | `https://www.facebook.com/dialog/oauth`                          |
| Google    | `https://www.googleapis.com/auth/userinfo.profile`               |
| Microsoft | `https://login.microsoftonline.com/common/oauth2/v2.0/authorize` |
| Twitter   | `https://api.twitter.com/oauth/authenticate`                     |

Nell'app di esempio, `userinfo.profile` l'ambito di Google viene aggiunto automaticamente dal framework quando <xref:Microsoft.Extensions.DependencyInjection.GoogleExtensions.AddGoogle*> viene chiamato su <xref:Microsoft.AspNetCore.Authentication.AuthenticationBuilder> . Se l'app richiede ambiti aggiuntivi, aggiungerli alle opzioni. Nell'esempio seguente `https://www.googleapis.com/auth/user.birthday.read` viene aggiunto l'ambito Google per recuperare il compleanno di un utente:

```csharp
options.Scope.Add("https://www.googleapis.com/auth/user.birthday.read");
```

## <a name="map-user-data-keys-and-create-claims"></a>Eseguire il mapping delle chiavi di dati utente e creare attestazioni

Nelle opzioni del provider specificare un <xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionMapExtensions.MapJsonKey*> o <xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionMapExtensions.MapJsonSubKey*> per ogni chiave/sottochiave nei dati utente JSON del provider esterno per l'identità dell'app da leggere durante l'accesso. Per ulteriori informazioni sui tipi di attestazione, vedere <xref:System.Security.Claims.ClaimTypes> .

L'app di esempio crea attestazioni delle impostazioni locali ( `urn:google:locale` ) e immagini ( `urn:google:picture` ) dalle `locale` `picture` chiavi e nei dati utente di Google:

[!code-csharp[](additional-claims/samples/2.x/ClaimsSample/Startup.cs?name=snippet_AddGoogle&highlight=13-14)]

In `Microsoft.AspNetCore.Identity.UI.Pages.Account.Internal.ExternalLoginModel.OnPostConfirmationAsync` , un <xref:Microsoft.AspNetCore.Identity.IdentityUser> ( `ApplicationUser` ) è connesso all'app con <xref:Microsoft.AspNetCore.Identity.SignInManager%601.SignInAsync*> . Durante il processo di accesso, il <xref:Microsoft.AspNetCore.Identity.UserManager%601> può archiviare `ApplicationUser` attestazioni per i dati utente disponibili da <xref:Microsoft.AspNetCore.Identity.ExternalLoginInfo.Principal*> .

Nell'app di esempio `OnPostConfirmationAsync` (*account/ExternalLogin. cshtml. cs*) vengono stabilite le attestazioni delle impostazioni locali ( `urn:google:locale` ) e immagine ( `urn:google:picture` ) per l'accesso `ApplicationUser` , inclusa un'attestazione per <xref:System.Security.Claims.ClaimTypes.GivenName> :

[!code-csharp[](additional-claims/samples/2.x/ClaimsSample/Areas/Identity/Pages/Account/ExternalLogin.cshtml.cs?name=snippet_OnPostConfirmationAsync&highlight=35-51)]

Per impostazione predefinita, le attestazioni di un utente vengono archiviate nell'autenticazione di cookie . Se l'autenticazione cookie è troppo grande, può causare un errore dell'app perché:

* Il browser rileva che l' cookie intestazione è troppo lungo.
* La dimensione complessiva della richiesta è troppo grande.

Se per l'elaborazione delle richieste utente è richiesta una grande quantità di dati utente:

* Limitare il numero e le dimensioni delle attestazioni utente per l'elaborazione delle richieste solo ai componenti richiesti dall'app.
* Usare un personalizzato <xref:Microsoft.AspNetCore.Authentication.Cookies.ITicketStore> per il Cookie middleware di autenticazione <xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationOptions.SessionStore> per archiviare l'identità tra le richieste. Mantenere grandi quantità di informazioni sull'identità sul server inviando al client solo una chiave di identificatore di sessione di piccole dimensioni.

## <a name="save-the-access-token"></a>Salvare il token di accesso

<xref:Microsoft.AspNetCore.Authentication.RemoteAuthenticationOptions.SaveTokens*> definisce se i token di accesso e di aggiornamento devono essere archiviati in <xref:Microsoft.AspNetCore.Http.Authentication.AuthenticationProperties> dopo un'autorizzazione riuscita. `SaveTokens` è impostato su per `false` impostazione predefinita per ridurre le dimensioni dell'autenticazione finale cookie .

L'app di esempio imposta il valore di `SaveTokens` su `true` in <xref:Microsoft.AspNetCore.Authentication.Google.GoogleOptions> :

[!code-csharp[](additional-claims/samples/2.x/ClaimsSample/Startup.cs?name=snippet_AddGoogle&highlight=15)]

Quando `OnPostConfirmationAsync` viene eseguito, archivia il token di accesso ([ExternalLoginInfo. AuthenticationTokens](xref:Microsoft.AspNetCore.Identity.ExternalLoginInfo.AuthenticationTokens*)) dal provider esterno in `ApplicationUser` `AuthenticationProperties` .

L'app di esempio salva il token di accesso in `OnPostConfirmationAsync` (nuova registrazione utente) e `OnGetCallbackAsync` (utente registrato in precedenza) in *account/ExternalLogin. cshtml. cs*:

[!code-csharp[](additional-claims/samples/2.x/ClaimsSample/Areas/Identity/Pages/Account/ExternalLogin.cshtml.cs?name=snippet_OnPostConfirmationAsync&highlight=54-56)]

## <a name="how-to-add-additional-custom-tokens"></a>Come aggiungere token personalizzati aggiuntivi

Per illustrare come aggiungere un token personalizzato, archiviato come parte di `SaveTokens` , l'app di esempio aggiunge un oggetto <xref:Microsoft.AspNetCore.Authentication.AuthenticationToken> con l'oggetto corrente <xref:System.DateTime> per un [AuthenticationToken.Name](xref:Microsoft.AspNetCore.Authentication.AuthenticationToken.Name*) di `TicketCreated` :

[!code-csharp[](additional-claims/samples/2.x/ClaimsSample/Startup.cs?name=snippet_AddGoogle&highlight=17-30)]

## <a name="creating-and-adding-claims"></a>Creazione e aggiunta di attestazioni

Il Framework fornisce azioni comuni e metodi di estensione per la creazione e l'aggiunta di attestazioni alla raccolta. Per altre informazioni, vedere <xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionMapExtensions> e <xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionUniqueExtensions>.

Gli utenti possono definire azioni personalizzate derivando da <xref:Microsoft.AspNetCore.Authentication.OAuth.Claims.ClaimAction> e implementando il <xref:Microsoft.AspNetCore.Authentication.OAuth.Claims.ClaimAction.Run*> metodo astratto.

Per altre informazioni, vedere <xref:Microsoft.AspNetCore.Authentication.OAuth.Claims>.

## <a name="removal-of-claim-actions-and-claims"></a>Rimozione di azioni attestazioni e attestazioni

[ClaimActionCollection. Remove (String)](xref:Microsoft.AspNetCore.Authentication.OAuth.Claims.ClaimActionCollection.Remove*) rimuove tutte le azioni di attestazione per l'oggetto specificato <xref:Microsoft.AspNetCore.Authentication.OAuth.Claims.ClaimAction.ClaimType> dalla raccolta. [ClaimActionCollectionMapExtensions. DeleteClaim (ClaimActionCollection, String)](xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionMapExtensions.DeleteClaim*) Elimina un'attestazione dell'oggetto specificato <xref:Microsoft.AspNetCore.Authentication.OAuth.Claims.ClaimAction.ClaimType> dall'identità. <xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionMapExtensions.DeleteClaim*> viene utilizzato principalmente con [OpenID Connect (OIDC)](/azure/active-directory/develop/v2-protocols-oidc) per rimuovere le attestazioni generate dal protocollo.

## <a name="sample-app-output"></a>Esempio di output dell'app

```
User Claims

http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier
    9b342344f-7aab-43c2-1ac1-ba75912ca999
http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name
    someone@gmail.com
AspNet.Identity.SecurityStamp
    7D4312MOWRYYBFI1KXRPHGOSTBVWSFDE
http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname
    Judy
urn:google:locale
    en
urn:google:picture
    https://lh4.googleusercontent.com/-XXXXXX/XXXXXX/XXXXXX/XXXXXX/photo.jpg

Authentication Properties

.Token.access_token
    yc23.AlvoZqz56...1lxltXV7D-ZWP9
.Token.token_type
    Bearer
.Token.expires_at
    2019-04-11T22:14:51.0000000+00:00
.Token.TicketCreated
    4/11/2019 9:14:52 PM
.TokenNames
    access_token;token_type;expires_at;TicketCreated
.persistent
.issued
    Thu, 11 Apr 2019 20:51:06 GMT
.expires
    Thu, 25 Apr 2019 20:51:06 GMT

```

[!INCLUDE[Forward request information when behind a proxy or load balancer section](includes/forwarded-headers-middleware.md)]

::: moniker-end

## <a name="additional-resources"></a>Risorse aggiuntive

* [app SocialSample di progettazione DotNet/AspNetCore](https://github.com/dotnet/AspNetCore/tree/main/src/Security/Authentication/samples/SocialSample): l'app di esempio collegata si trova nel ramo di progettazione del [repository GitHub DotNet/AspNetCore](https://github.com/dotnet/AspNetCore) `main` . Il `main` ramo contiene il codice in fase di sviluppo attivo per la prossima versione di ASP.NET Core. Per visualizzare una versione dell'app di esempio per una versione rilasciata di ASP.NET Core, usare l'elenco a discesa **Branch** per selezionare un ramo di rilascio (ad esempio `release/{X.Y}` ).
