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
ms.openlocfilehash: 9c04ca466566e956b5e6dfec8131096c3995bc14
ms.sourcegitcommit: a1db01b4d3bd8c57d7a9c94ce122a6db68002d66
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/04/2021
ms.locfileid: "102110144"
---
# <a name="persist-additional-claims-and-tokens-from-external-providers-in-aspnet-core"></a><span data-ttu-id="4fb3b-103">Mantieni attestazioni e token aggiuntivi da provider esterni in ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="4fb3b-103">Persist additional claims and tokens from external providers in ASP.NET Core</span></span>

::: moniker range=">= aspnetcore-3.0"

<span data-ttu-id="4fb3b-104">Un'app ASP.NET Core può creare attestazioni e token aggiuntivi da provider di autenticazione esterni, ad esempio Facebook, Google, Microsoft e Twitter.</span><span class="sxs-lookup"><span data-stu-id="4fb3b-104">An ASP.NET Core app can establish additional claims and tokens from external authentication providers, such as Facebook, Google, Microsoft, and Twitter.</span></span> <span data-ttu-id="4fb3b-105">Ogni provider rivela informazioni diverse sugli utenti sulla propria piattaforma, ma il modello per la ricezione e la trasformazione dei dati utente in attestazioni aggiuntive è lo stesso.</span><span class="sxs-lookup"><span data-stu-id="4fb3b-105">Each provider reveals different information about users on its platform, but the pattern for receiving and transforming user data into additional claims is the same.</span></span>

<span data-ttu-id="4fb3b-106">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/security/authentication/social/additional-claims/samples) ([procedura per il download](xref:index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="4fb3b-106">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/security/authentication/social/additional-claims/samples) ([how to download](xref:index#how-to-download-a-sample))</span></span>

## <a name="prerequisites"></a><span data-ttu-id="4fb3b-107">Prerequisiti</span><span class="sxs-lookup"><span data-stu-id="4fb3b-107">Prerequisites</span></span>

<span data-ttu-id="4fb3b-108">Decidere quali provider di autenticazione esterni supportare nell'app.</span><span class="sxs-lookup"><span data-stu-id="4fb3b-108">Decide which external authentication providers to support in the app.</span></span> <span data-ttu-id="4fb3b-109">Per ogni provider, registrare l'app e ottenere un ID client e un segreto client.</span><span class="sxs-lookup"><span data-stu-id="4fb3b-109">For each provider, register the app and obtain a client ID and client secret.</span></span> <span data-ttu-id="4fb3b-110">Per altre informazioni, vedere <xref:security/authentication/social/index>.</span><span class="sxs-lookup"><span data-stu-id="4fb3b-110">For more information, see <xref:security/authentication/social/index>.</span></span> <span data-ttu-id="4fb3b-111">L'app di esempio usa il [provider di autenticazione Google](xref:security/authentication/google-logins).</span><span class="sxs-lookup"><span data-stu-id="4fb3b-111">The sample app uses the [Google authentication provider](xref:security/authentication/google-logins).</span></span>

## <a name="set-the-client-id-and-client-secret"></a><span data-ttu-id="4fb3b-112">Impostare l'ID client e il segreto client</span><span class="sxs-lookup"><span data-stu-id="4fb3b-112">Set the client ID and client secret</span></span>

<span data-ttu-id="4fb3b-113">Il provider di autenticazione OAuth stabilisce una relazione di trust con un'app che usa un ID client e un segreto client.</span><span class="sxs-lookup"><span data-stu-id="4fb3b-113">The OAuth authentication provider establishes a trust relationship with an app using a client ID and client secret.</span></span> <span data-ttu-id="4fb3b-114">I valori relativi all'ID client e al segreto client vengono creati per l'app dal provider di autenticazione esterno quando l'app viene registrata con il provider.</span><span class="sxs-lookup"><span data-stu-id="4fb3b-114">Client ID and client secret values are created for the app by the external authentication provider when the app is registered with the provider.</span></span> <span data-ttu-id="4fb3b-115">Ogni provider esterno usato dall'app deve essere configurato in modo indipendente con l'ID client e il segreto client del provider.</span><span class="sxs-lookup"><span data-stu-id="4fb3b-115">Each external provider that the app uses must be configured independently with the provider's client ID and client secret.</span></span> <span data-ttu-id="4fb3b-116">Per ulteriori informazioni, vedere gli argomenti del provider di autenticazione esterno applicabili allo scenario:</span><span class="sxs-lookup"><span data-stu-id="4fb3b-116">For more information, see the external authentication provider topics that apply to your scenario:</span></span>

* [<span data-ttu-id="4fb3b-117">Autenticazione Facebook</span><span class="sxs-lookup"><span data-stu-id="4fb3b-117">Facebook authentication</span></span>](xref:security/authentication/facebook-logins)
* [<span data-ttu-id="4fb3b-118">Autenticazione Google</span><span class="sxs-lookup"><span data-stu-id="4fb3b-118">Google authentication</span></span>](xref:security/authentication/google-logins)
* [<span data-ttu-id="4fb3b-119">Autenticazione Microsoft</span><span class="sxs-lookup"><span data-stu-id="4fb3b-119">Microsoft authentication</span></span>](xref:security/authentication/microsoft-logins)
* [<span data-ttu-id="4fb3b-120">Autenticazione Twitter</span><span class="sxs-lookup"><span data-stu-id="4fb3b-120">Twitter authentication</span></span>](xref:security/authentication/twitter-logins)
* [<span data-ttu-id="4fb3b-121">Altri provider di autenticazione</span><span class="sxs-lookup"><span data-stu-id="4fb3b-121">Other authentication providers</span></span>](xref:security/authentication/otherlogins)
* [<span data-ttu-id="4fb3b-122">OpenIdConnect</span><span class="sxs-lookup"><span data-stu-id="4fb3b-122">OpenIdConnect</span></span>](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2)

<span data-ttu-id="4fb3b-123">L'app di esempio configura il provider di autenticazione Google con un ID client e un segreto client forniti da Google:</span><span class="sxs-lookup"><span data-stu-id="4fb3b-123">The sample app configures the Google authentication provider with a client ID and client secret provided by Google:</span></span>

[!code-csharp[](additional-claims/samples/3.x/ClaimsSample/Startup.cs?name=snippet_AddGoogle&highlight=4,9)]

## <a name="establish-the-authentication-scope"></a><span data-ttu-id="4fb3b-124">Definire l'ambito di autenticazione</span><span class="sxs-lookup"><span data-stu-id="4fb3b-124">Establish the authentication scope</span></span>

<span data-ttu-id="4fb3b-125">Specificare l'elenco di autorizzazioni da recuperare dal provider specificando <xref:Microsoft.AspNetCore.Authentication.OAuth.OAuthOptions.Scope*> .</span><span class="sxs-lookup"><span data-stu-id="4fb3b-125">Specify the list of permissions to retrieve from the provider by specifying the <xref:Microsoft.AspNetCore.Authentication.OAuth.OAuthOptions.Scope*>.</span></span> <span data-ttu-id="4fb3b-126">Gli ambiti di autenticazione per i provider esterni comuni sono riportati nella tabella seguente.</span><span class="sxs-lookup"><span data-stu-id="4fb3b-126">Authentication scopes for common external providers appear in the following table.</span></span>

| <span data-ttu-id="4fb3b-127">Provider</span><span class="sxs-lookup"><span data-stu-id="4fb3b-127">Provider</span></span>  | <span data-ttu-id="4fb3b-128">Ambito</span><span class="sxs-lookup"><span data-stu-id="4fb3b-128">Scope</span></span>                                                            |
| --------- | ---------------------------------------------------------------- |
| <span data-ttu-id="4fb3b-129">Facebook</span><span class="sxs-lookup"><span data-stu-id="4fb3b-129">Facebook</span></span>  | `https://www.facebook.com/dialog/oauth`                          |
| <span data-ttu-id="4fb3b-130">Google</span><span class="sxs-lookup"><span data-stu-id="4fb3b-130">Google</span></span>    | <span data-ttu-id="4fb3b-131">`profile`, `email`, `openid`</span><span class="sxs-lookup"><span data-stu-id="4fb3b-131">`profile`, `email`, `openid`</span></span>                                     |
| <span data-ttu-id="4fb3b-132">Microsoft</span><span class="sxs-lookup"><span data-stu-id="4fb3b-132">Microsoft</span></span> | `https://login.microsoftonline.com/common/oauth2/v2.0/authorize` |
| <span data-ttu-id="4fb3b-133">Twitter</span><span class="sxs-lookup"><span data-stu-id="4fb3b-133">Twitter</span></span>   | `https://api.twitter.com/oauth/authenticate`                     |

<span data-ttu-id="4fb3b-134">Nell'app di esempio gli `profile` ambiti di Google, `email` e `openid` vengono aggiunti automaticamente dal framework quando <xref:Microsoft.Extensions.DependencyInjection.GoogleExtensions.AddGoogle%2A> viene chiamato su <xref:Microsoft.AspNetCore.Authentication.AuthenticationBuilder> .</span><span class="sxs-lookup"><span data-stu-id="4fb3b-134">In the sample app, Google's `profile`, `email`, and `openid` scopes are automatically added by the framework when <xref:Microsoft.Extensions.DependencyInjection.GoogleExtensions.AddGoogle%2A> is called on the <xref:Microsoft.AspNetCore.Authentication.AuthenticationBuilder>.</span></span> <span data-ttu-id="4fb3b-135">Se l'app richiede ambiti aggiuntivi, aggiungerli alle opzioni.</span><span class="sxs-lookup"><span data-stu-id="4fb3b-135">If the app requires additional scopes, add them to the options.</span></span> <span data-ttu-id="4fb3b-136">Nell'esempio seguente `https://www.googleapis.com/auth/user.birthday.read` viene aggiunto l'ambito Google per recuperare il compleanno di un utente:</span><span class="sxs-lookup"><span data-stu-id="4fb3b-136">In the following example, the Google `https://www.googleapis.com/auth/user.birthday.read` scope is added to retrieve a user's birthday:</span></span>

```csharp
options.Scope.Add("https://www.googleapis.com/auth/user.birthday.read");
```

## <a name="map-user-data-keys-and-create-claims"></a><span data-ttu-id="4fb3b-137">Eseguire il mapping delle chiavi di dati utente e creare attestazioni</span><span class="sxs-lookup"><span data-stu-id="4fb3b-137">Map user data keys and create claims</span></span>

<span data-ttu-id="4fb3b-138">Nelle opzioni del provider specificare un <xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionMapExtensions.MapJsonKey*> o <xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionMapExtensions.MapJsonSubKey*> per ogni chiave/sottochiave nei dati utente JSON del provider esterno per l'identità dell'app da leggere durante l'accesso.</span><span class="sxs-lookup"><span data-stu-id="4fb3b-138">In the provider's options, specify a <xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionMapExtensions.MapJsonKey*> or <xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionMapExtensions.MapJsonSubKey*> for each key/subkey in the external provider's JSON user data for the app identity to read on sign in.</span></span> <span data-ttu-id="4fb3b-139">Per ulteriori informazioni sui tipi di attestazione, vedere <xref:System.Security.Claims.ClaimTypes> .</span><span class="sxs-lookup"><span data-stu-id="4fb3b-139">For more information on claim types, see <xref:System.Security.Claims.ClaimTypes>.</span></span>

<span data-ttu-id="4fb3b-140">L'app di esempio crea attestazioni delle impostazioni locali ( `urn:google:locale` ) e immagini ( `urn:google:picture` ) dalle `locale` `picture` chiavi e nei dati utente di Google:</span><span class="sxs-lookup"><span data-stu-id="4fb3b-140">The sample app creates locale (`urn:google:locale`) and picture (`urn:google:picture`) claims from the `locale` and `picture` keys in Google user data:</span></span>

[!code-csharp[](additional-claims/samples/3.x/ClaimsSample/Startup.cs?name=snippet_AddGoogle&highlight=13-14)]

<span data-ttu-id="4fb3b-141">In `Microsoft.AspNetCore.Identity.UI.Pages.Account.Internal.ExternalLoginModel.OnPostConfirmationAsync` , un <xref:Microsoft.AspNetCore.Identity.IdentityUser> ( `ApplicationUser` ) è connesso all'app con <xref:Microsoft.AspNetCore.Identity.SignInManager%601.SignInAsync*> .</span><span class="sxs-lookup"><span data-stu-id="4fb3b-141">In `Microsoft.AspNetCore.Identity.UI.Pages.Account.Internal.ExternalLoginModel.OnPostConfirmationAsync`, an <xref:Microsoft.AspNetCore.Identity.IdentityUser> (`ApplicationUser`) is signed into the app with <xref:Microsoft.AspNetCore.Identity.SignInManager%601.SignInAsync*>.</span></span> <span data-ttu-id="4fb3b-142">Durante il processo di accesso, il <xref:Microsoft.AspNetCore.Identity.UserManager%601> può archiviare `ApplicationUser` attestazioni per i dati utente disponibili da <xref:Microsoft.AspNetCore.Identity.ExternalLoginInfo.Principal*> .</span><span class="sxs-lookup"><span data-stu-id="4fb3b-142">During the sign in process, the <xref:Microsoft.AspNetCore.Identity.UserManager%601> can store an `ApplicationUser` claims for user data available from the <xref:Microsoft.AspNetCore.Identity.ExternalLoginInfo.Principal*>.</span></span>

<span data-ttu-id="4fb3b-143">Nell'app di esempio `OnPostConfirmationAsync` (*account/ExternalLogin. cshtml. cs*) vengono stabilite le attestazioni delle impostazioni locali ( `urn:google:locale` ) e immagine ( `urn:google:picture` ) per l'accesso `ApplicationUser` , inclusa un'attestazione per <xref:System.Security.Claims.ClaimTypes.GivenName> :</span><span class="sxs-lookup"><span data-stu-id="4fb3b-143">In the sample app, `OnPostConfirmationAsync` (*Account/ExternalLogin.cshtml.cs*) establishes the locale (`urn:google:locale`) and picture (`urn:google:picture`) claims for the signed in `ApplicationUser`, including a claim for <xref:System.Security.Claims.ClaimTypes.GivenName>:</span></span>

[!code-csharp[](additional-claims/samples/3.x/ClaimsSample/Areas/Identity/Pages/Account/ExternalLogin.cshtml.cs?name=snippet_OnPostConfirmationAsync&highlight=35-51)]

<span data-ttu-id="4fb3b-144">Per impostazione predefinita, le attestazioni di un utente vengono archiviate nell'autenticazione di cookie .</span><span class="sxs-lookup"><span data-stu-id="4fb3b-144">By default, a user's claims are stored in the authentication cookie.</span></span> <span data-ttu-id="4fb3b-145">Se l'autenticazione cookie è troppo grande, può causare un errore dell'app perché:</span><span class="sxs-lookup"><span data-stu-id="4fb3b-145">If the authentication cookie is too large, it can cause the app to fail because:</span></span>

* <span data-ttu-id="4fb3b-146">Il browser rileva che l' cookie intestazione è troppo lungo.</span><span class="sxs-lookup"><span data-stu-id="4fb3b-146">The browser detects that the cookie header is too long.</span></span>
* <span data-ttu-id="4fb3b-147">La dimensione complessiva della richiesta è troppo grande.</span><span class="sxs-lookup"><span data-stu-id="4fb3b-147">The overall size of the request is too large.</span></span>

<span data-ttu-id="4fb3b-148">Se per l'elaborazione delle richieste utente è richiesta una grande quantità di dati utente:</span><span class="sxs-lookup"><span data-stu-id="4fb3b-148">If a large amount of user data is required for processing user requests:</span></span>

* <span data-ttu-id="4fb3b-149">Limitare il numero e le dimensioni delle attestazioni utente per l'elaborazione delle richieste solo ai componenti richiesti dall'app.</span><span class="sxs-lookup"><span data-stu-id="4fb3b-149">Limit the number and size of user claims for request processing to only what the app requires.</span></span>
* <span data-ttu-id="4fb3b-150">Usare un personalizzato <xref:Microsoft.AspNetCore.Authentication.Cookies.ITicketStore> per il Cookie middleware di autenticazione <xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationOptions.SessionStore> per archiviare l'identità tra le richieste.</span><span class="sxs-lookup"><span data-stu-id="4fb3b-150">Use a custom <xref:Microsoft.AspNetCore.Authentication.Cookies.ITicketStore> for the Cookie Authentication Middleware's <xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationOptions.SessionStore> to store identity across requests.</span></span> <span data-ttu-id="4fb3b-151">Mantenere grandi quantità di informazioni sull'identità sul server inviando al client solo una chiave di identificatore di sessione di piccole dimensioni.</span><span class="sxs-lookup"><span data-stu-id="4fb3b-151">Preserve large quantities of identity information on the server while only sending a small session identifier key to the client.</span></span>

## <a name="save-the-access-token"></a><span data-ttu-id="4fb3b-152">Salvare il token di accesso</span><span class="sxs-lookup"><span data-stu-id="4fb3b-152">Save the access token</span></span>

<span data-ttu-id="4fb3b-153"><xref:Microsoft.AspNetCore.Authentication.RemoteAuthenticationOptions.SaveTokens*> definisce se i token di accesso e di aggiornamento devono essere archiviati in <xref:Microsoft.AspNetCore.Http.Authentication.AuthenticationProperties> dopo un'autorizzazione riuscita.</span><span class="sxs-lookup"><span data-stu-id="4fb3b-153"><xref:Microsoft.AspNetCore.Authentication.RemoteAuthenticationOptions.SaveTokens*> defines whether access and refresh tokens should be stored in the <xref:Microsoft.AspNetCore.Http.Authentication.AuthenticationProperties> after a successful authorization.</span></span> <span data-ttu-id="4fb3b-154">`SaveTokens` è impostato su per `false` impostazione predefinita per ridurre le dimensioni dell'autenticazione finale cookie .</span><span class="sxs-lookup"><span data-stu-id="4fb3b-154">`SaveTokens` is set to `false` by default to reduce the size of the final authentication cookie.</span></span>

<span data-ttu-id="4fb3b-155">L'app di esempio imposta il valore di `SaveTokens` su `true` in <xref:Microsoft.AspNetCore.Authentication.Google.GoogleOptions> :</span><span class="sxs-lookup"><span data-stu-id="4fb3b-155">The sample app sets the value of `SaveTokens` to `true` in <xref:Microsoft.AspNetCore.Authentication.Google.GoogleOptions>:</span></span>

[!code-csharp[](additional-claims/samples/3.x/ClaimsSample/Startup.cs?name=snippet_AddGoogle&highlight=15)]

<span data-ttu-id="4fb3b-156">Quando `OnPostConfirmationAsync` viene eseguito, archivia il token di accesso ([ExternalLoginInfo. AuthenticationTokens](xref:Microsoft.AspNetCore.Identity.ExternalLoginInfo.AuthenticationTokens*)) dal provider esterno in `ApplicationUser` `AuthenticationProperties` .</span><span class="sxs-lookup"><span data-stu-id="4fb3b-156">When `OnPostConfirmationAsync` executes, store the access token ([ExternalLoginInfo.AuthenticationTokens](xref:Microsoft.AspNetCore.Identity.ExternalLoginInfo.AuthenticationTokens*)) from the external provider in the `ApplicationUser`'s `AuthenticationProperties`.</span></span>

<span data-ttu-id="4fb3b-157">L'app di esempio salva il token di accesso in `OnPostConfirmationAsync` (nuova registrazione utente) e `OnGetCallbackAsync` (utente registrato in precedenza) in *account/ExternalLogin. cshtml. cs*:</span><span class="sxs-lookup"><span data-stu-id="4fb3b-157">The sample app saves the access token in `OnPostConfirmationAsync` (new user registration) and `OnGetCallbackAsync` (previously registered user) in *Account/ExternalLogin.cshtml.cs*:</span></span>

[!code-csharp[](additional-claims/samples/3.x/ClaimsSample/Areas/Identity/Pages/Account/ExternalLogin.cshtml.cs?name=snippet_OnPostConfirmationAsync&highlight=54-56)]

> [!NOTE]
> <span data-ttu-id="4fb3b-158">Per informazioni sul passaggio dei token ai Razor componenti di un' Blazor Server app, vedere <xref:blazor/security/server/additional-scenarios#pass-tokens-to-a-blazor-server-app> .</span><span class="sxs-lookup"><span data-stu-id="4fb3b-158">For information on passing tokens to the Razor components of a Blazor Server app, see <xref:blazor/security/server/additional-scenarios#pass-tokens-to-a-blazor-server-app>.</span></span>

## <a name="how-to-add-additional-custom-tokens"></a><span data-ttu-id="4fb3b-159">Come aggiungere token personalizzati aggiuntivi</span><span class="sxs-lookup"><span data-stu-id="4fb3b-159">How to add additional custom tokens</span></span>

<span data-ttu-id="4fb3b-160">Per illustrare come aggiungere un token personalizzato, archiviato come parte di `SaveTokens` , l'app di esempio aggiunge un oggetto <xref:Microsoft.AspNetCore.Authentication.AuthenticationToken> con l'oggetto corrente <xref:System.DateTime> per un [AuthenticationToken.Name](xref:Microsoft.AspNetCore.Authentication.AuthenticationToken.Name*) di `TicketCreated` :</span><span class="sxs-lookup"><span data-stu-id="4fb3b-160">To demonstrate how to add a custom token, which is stored as part of `SaveTokens`, the sample app adds an <xref:Microsoft.AspNetCore.Authentication.AuthenticationToken> with the current <xref:System.DateTime> for an [AuthenticationToken.Name](xref:Microsoft.AspNetCore.Authentication.AuthenticationToken.Name*) of `TicketCreated`:</span></span>

[!code-csharp[](additional-claims/samples/3.x/ClaimsSample/Startup.cs?name=snippet_AddGoogle&highlight=17-30)]

## <a name="creating-and-adding-claims"></a><span data-ttu-id="4fb3b-161">Creazione e aggiunta di attestazioni</span><span class="sxs-lookup"><span data-stu-id="4fb3b-161">Creating and adding claims</span></span>

<span data-ttu-id="4fb3b-162">Il Framework fornisce azioni comuni e metodi di estensione per la creazione e l'aggiunta di attestazioni alla raccolta.</span><span class="sxs-lookup"><span data-stu-id="4fb3b-162">The framework provides common actions and extension methods for creating and adding claims to the collection.</span></span> <span data-ttu-id="4fb3b-163">Per altre informazioni, vedere <xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionMapExtensions> e <xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionUniqueExtensions>.</span><span class="sxs-lookup"><span data-stu-id="4fb3b-163">For more information, see the <xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionMapExtensions> and <xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionUniqueExtensions>.</span></span>

<span data-ttu-id="4fb3b-164">Gli utenti possono definire azioni personalizzate derivando da <xref:Microsoft.AspNetCore.Authentication.OAuth.Claims.ClaimAction> e implementando il <xref:Microsoft.AspNetCore.Authentication.OAuth.Claims.ClaimAction.Run*> metodo astratto.</span><span class="sxs-lookup"><span data-stu-id="4fb3b-164">Users can define custom actions by deriving from <xref:Microsoft.AspNetCore.Authentication.OAuth.Claims.ClaimAction> and implementing the abstract <xref:Microsoft.AspNetCore.Authentication.OAuth.Claims.ClaimAction.Run*> method.</span></span>

<span data-ttu-id="4fb3b-165">Per altre informazioni, vedere <xref:Microsoft.AspNetCore.Authentication.OAuth.Claims>.</span><span class="sxs-lookup"><span data-stu-id="4fb3b-165">For more information, see <xref:Microsoft.AspNetCore.Authentication.OAuth.Claims>.</span></span>

## <a name="add-and-update-user-claims"></a><span data-ttu-id="4fb3b-166">Aggiungi e aggiorna attestazioni utente</span><span class="sxs-lookup"><span data-stu-id="4fb3b-166">Add and update user claims</span></span>

<span data-ttu-id="4fb3b-167">Le attestazioni vengono copiate dai provider esterni al database utente alla prima registrazione, non all'accesso.</span><span class="sxs-lookup"><span data-stu-id="4fb3b-167">Claims are copied from external providers to the user database on first registration, not on sign in.</span></span> <span data-ttu-id="4fb3b-168">Se in un'app sono abilitate altre attestazioni dopo che un utente si è registrato per l'uso dell'app, chiamare [SignInManager. RefreshSignInAsync](xref:Microsoft.AspNetCore.Identity.SignInManager%601) su un utente per forzare la generazione di una nuova autenticazione cookie .</span><span class="sxs-lookup"><span data-stu-id="4fb3b-168">If additional claims are enabled in an app after a user registers to use the app, call [SignInManager.RefreshSignInAsync](xref:Microsoft.AspNetCore.Identity.SignInManager%601) on a user to force the generation of a new authentication cookie.</span></span>

<span data-ttu-id="4fb3b-169">Nell'ambiente di sviluppo che utilizza gli account utente di test, è possibile eliminare e ricreare semplicemente l'account utente.</span><span class="sxs-lookup"><span data-stu-id="4fb3b-169">In the Development environment working with test user accounts, you can simply delete and recreate the user account.</span></span> <span data-ttu-id="4fb3b-170">Per i sistemi di produzione, le nuove attestazioni aggiunte all'app possono essere inserite in account utente.</span><span class="sxs-lookup"><span data-stu-id="4fb3b-170">For production systems, new claims added to the app can be backfilled into user accounts.</span></span> <span data-ttu-id="4fb3b-171">Dopo aver eseguito [il ponteggi della `ExternalLogin` pagina](xref:security/authentication/scaffold-identity) nell'app in `Areas/Pages/Identity/Account/Manage` , aggiungere il codice seguente a `ExternalLoginModel` nel `ExternalLogin.cshtml.cs` file.</span><span class="sxs-lookup"><span data-stu-id="4fb3b-171">After [scaffolding the `ExternalLogin` page](xref:security/authentication/scaffold-identity) into the app at `Areas/Pages/Identity/Account/Manage`, add the following code to the `ExternalLoginModel` in the `ExternalLogin.cshtml.cs` file.</span></span>

<span data-ttu-id="4fb3b-172">Aggiungere un dizionario di attestazioni aggiuntive.</span><span class="sxs-lookup"><span data-stu-id="4fb3b-172">Add a dictionary of added claims.</span></span> <span data-ttu-id="4fb3b-173">Usare le chiavi del dizionario per conservare i tipi di attestazione e usare i valori per mantenere un valore predefinito.</span><span class="sxs-lookup"><span data-stu-id="4fb3b-173">Use the dictionary keys to hold the claim types, and use the values to hold a default value.</span></span> <span data-ttu-id="4fb3b-174">Aggiungere la riga seguente all'inizio della classe.</span><span class="sxs-lookup"><span data-stu-id="4fb3b-174">Add the following line to the top of the class.</span></span> <span data-ttu-id="4fb3b-175">Nell'esempio seguente si presuppone che venga aggiunta un'attestazione per l'immagine Google dell'utente con un'immagine Headshot generica come valore predefinito:</span><span class="sxs-lookup"><span data-stu-id="4fb3b-175">The following example assumes that one claim is added for the user's Google picture with a generic headshot image as the default value:</span></span>

```csharp
private readonly IReadOnlyDictionary<string, string> _claimsToSync = 
    new Dictionary<string, string>()
    {
        { "urn:google:picture", "https://localhost:5001/headshot.png" },
    };
```

<span data-ttu-id="4fb3b-176">Sostituire il codice predefinito del `OnGetCallbackAsync` metodo con il codice seguente.</span><span class="sxs-lookup"><span data-stu-id="4fb3b-176">Replace the default code of the `OnGetCallbackAsync` method with the following code.</span></span> <span data-ttu-id="4fb3b-177">Il codice esegue il ciclo nel dizionario delle attestazioni.</span><span class="sxs-lookup"><span data-stu-id="4fb3b-177">The code loops through the claims dictionary.</span></span> <span data-ttu-id="4fb3b-178">Le attestazioni vengono aggiunte (riempite) o aggiornate per ogni utente.</span><span class="sxs-lookup"><span data-stu-id="4fb3b-178">Claims are added (backfilled) or updated for each user.</span></span> <span data-ttu-id="4fb3b-179">Quando si aggiungono o si aggiornano le attestazioni, l'accesso utente viene aggiornato utilizzando il <xref:Microsoft.AspNetCore.Identity.SignInManager%601> , mantenendo le proprietà di autenticazione esistenti ( `AuthenticationProperties` ).</span><span class="sxs-lookup"><span data-stu-id="4fb3b-179">When claims are added or updated, the user sign-in is refreshed using the <xref:Microsoft.AspNetCore.Identity.SignInManager%601>, preserving the existing authentication properties (`AuthenticationProperties`).</span></span>

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

<span data-ttu-id="4fb3b-180">Un approccio simile viene eseguito quando le attestazioni cambiano mentre un utente ha eseguito l'accesso, ma non è necessario un passaggio di recupero.</span><span class="sxs-lookup"><span data-stu-id="4fb3b-180">A similar approach is taken when claims change while a user is signed in but a backfill step isn't required.</span></span> <span data-ttu-id="4fb3b-181">Per aggiornare le attestazioni di un utente, chiamare quanto segue sull'utente:</span><span class="sxs-lookup"><span data-stu-id="4fb3b-181">To update a user's claims, call the following on the user:</span></span>

* <span data-ttu-id="4fb3b-182">[Usermanager. ReplaceClaimAsync](xref:Microsoft.AspNetCore.Identity.UserManager%601) per l'utente per le attestazioni archiviate nel database di identità.</span><span class="sxs-lookup"><span data-stu-id="4fb3b-182">[UserManager.ReplaceClaimAsync](xref:Microsoft.AspNetCore.Identity.UserManager%601) on the user for claims stored in the identity database.</span></span>
* <span data-ttu-id="4fb3b-183">[SignInManager. RefreshSignInAsync](xref:Microsoft.AspNetCore.Identity.SignInManager%601) sull'utente per forzare la generazione di una nuova autenticazione cookie .</span><span class="sxs-lookup"><span data-stu-id="4fb3b-183">[SignInManager.RefreshSignInAsync](xref:Microsoft.AspNetCore.Identity.SignInManager%601) on the user to force the generation of a new authentication cookie.</span></span>

## <a name="removal-of-claim-actions-and-claims"></a><span data-ttu-id="4fb3b-184">Rimozione di azioni attestazioni e attestazioni</span><span class="sxs-lookup"><span data-stu-id="4fb3b-184">Removal of claim actions and claims</span></span>

<span data-ttu-id="4fb3b-185">[ClaimActionCollection. Remove (String)](xref:Microsoft.AspNetCore.Authentication.OAuth.Claims.ClaimActionCollection.Remove*) rimuove tutte le azioni di attestazione per l'oggetto specificato <xref:Microsoft.AspNetCore.Authentication.OAuth.Claims.ClaimAction.ClaimType> dalla raccolta.</span><span class="sxs-lookup"><span data-stu-id="4fb3b-185">[ClaimActionCollection.Remove(String)](xref:Microsoft.AspNetCore.Authentication.OAuth.Claims.ClaimActionCollection.Remove*) removes all claim actions for the given <xref:Microsoft.AspNetCore.Authentication.OAuth.Claims.ClaimAction.ClaimType> from the collection.</span></span> <span data-ttu-id="4fb3b-186">[ClaimActionCollectionMapExtensions. DeleteClaim (ClaimActionCollection, String)](xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionMapExtensions.DeleteClaim*) Elimina un'attestazione dell'oggetto specificato <xref:Microsoft.AspNetCore.Authentication.OAuth.Claims.ClaimAction.ClaimType> dall'identità.</span><span class="sxs-lookup"><span data-stu-id="4fb3b-186">[ClaimActionCollectionMapExtensions.DeleteClaim(ClaimActionCollection, String)](xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionMapExtensions.DeleteClaim*) deletes a claim of the given <xref:Microsoft.AspNetCore.Authentication.OAuth.Claims.ClaimAction.ClaimType> from the identity.</span></span> <span data-ttu-id="4fb3b-187"><xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionMapExtensions.DeleteClaim*> viene utilizzato principalmente con [OpenID Connect (OIDC)](/azure/active-directory/develop/v2-protocols-oidc) per rimuovere le attestazioni generate dal protocollo.</span><span class="sxs-lookup"><span data-stu-id="4fb3b-187"><xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionMapExtensions.DeleteClaim*> is primarily used with [OpenID Connect (OIDC)](/azure/active-directory/develop/v2-protocols-oidc) to remove protocol-generated claims.</span></span>

## <a name="sample-app-output"></a><span data-ttu-id="4fb3b-188">Esempio di output dell'app</span><span class="sxs-lookup"><span data-stu-id="4fb3b-188">Sample app output</span></span>

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

<span data-ttu-id="4fb3b-189">Un'app ASP.NET Core può creare attestazioni e token aggiuntivi da provider di autenticazione esterni, ad esempio Facebook, Google, Microsoft e Twitter.</span><span class="sxs-lookup"><span data-stu-id="4fb3b-189">An ASP.NET Core app can establish additional claims and tokens from external authentication providers, such as Facebook, Google, Microsoft, and Twitter.</span></span> <span data-ttu-id="4fb3b-190">Ogni provider rivela informazioni diverse sugli utenti sulla propria piattaforma, ma il modello per la ricezione e la trasformazione dei dati utente in attestazioni aggiuntive è lo stesso.</span><span class="sxs-lookup"><span data-stu-id="4fb3b-190">Each provider reveals different information about users on its platform, but the pattern for receiving and transforming user data into additional claims is the same.</span></span>

<span data-ttu-id="4fb3b-191">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/security/authentication/social/additional-claims/samples) ([procedura per il download](xref:index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="4fb3b-191">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/security/authentication/social/additional-claims/samples) ([how to download](xref:index#how-to-download-a-sample))</span></span>

## <a name="prerequisites"></a><span data-ttu-id="4fb3b-192">Prerequisiti</span><span class="sxs-lookup"><span data-stu-id="4fb3b-192">Prerequisites</span></span>

<span data-ttu-id="4fb3b-193">Decidere quali provider di autenticazione esterni supportare nell'app.</span><span class="sxs-lookup"><span data-stu-id="4fb3b-193">Decide which external authentication providers to support in the app.</span></span> <span data-ttu-id="4fb3b-194">Per ogni provider, registrare l'app e ottenere un ID client e un segreto client.</span><span class="sxs-lookup"><span data-stu-id="4fb3b-194">For each provider, register the app and obtain a client ID and client secret.</span></span> <span data-ttu-id="4fb3b-195">Per altre informazioni, vedere <xref:security/authentication/social/index>.</span><span class="sxs-lookup"><span data-stu-id="4fb3b-195">For more information, see <xref:security/authentication/social/index>.</span></span> <span data-ttu-id="4fb3b-196">L'app di esempio usa il [provider di autenticazione Google](xref:security/authentication/google-logins).</span><span class="sxs-lookup"><span data-stu-id="4fb3b-196">The sample app uses the [Google authentication provider](xref:security/authentication/google-logins).</span></span>

## <a name="set-the-client-id-and-client-secret"></a><span data-ttu-id="4fb3b-197">Impostare l'ID client e il segreto client</span><span class="sxs-lookup"><span data-stu-id="4fb3b-197">Set the client ID and client secret</span></span>

<span data-ttu-id="4fb3b-198">Il provider di autenticazione OAuth stabilisce una relazione di trust con un'app che usa un ID client e un segreto client.</span><span class="sxs-lookup"><span data-stu-id="4fb3b-198">The OAuth authentication provider establishes a trust relationship with an app using a client ID and client secret.</span></span> <span data-ttu-id="4fb3b-199">I valori relativi all'ID client e al segreto client vengono creati per l'app dal provider di autenticazione esterno quando l'app viene registrata con il provider.</span><span class="sxs-lookup"><span data-stu-id="4fb3b-199">Client ID and client secret values are created for the app by the external authentication provider when the app is registered with the provider.</span></span> <span data-ttu-id="4fb3b-200">Ogni provider esterno usato dall'app deve essere configurato in modo indipendente con l'ID client e il segreto client del provider.</span><span class="sxs-lookup"><span data-stu-id="4fb3b-200">Each external provider that the app uses must be configured independently with the provider's client ID and client secret.</span></span> <span data-ttu-id="4fb3b-201">Per ulteriori informazioni, vedere gli argomenti del provider di autenticazione esterno applicabili allo scenario:</span><span class="sxs-lookup"><span data-stu-id="4fb3b-201">For more information, see the external authentication provider topics that apply to your scenario:</span></span>

* [<span data-ttu-id="4fb3b-202">Autenticazione Facebook</span><span class="sxs-lookup"><span data-stu-id="4fb3b-202">Facebook authentication</span></span>](xref:security/authentication/facebook-logins)
* [<span data-ttu-id="4fb3b-203">Autenticazione Google</span><span class="sxs-lookup"><span data-stu-id="4fb3b-203">Google authentication</span></span>](xref:security/authentication/google-logins)
* [<span data-ttu-id="4fb3b-204">Autenticazione Microsoft</span><span class="sxs-lookup"><span data-stu-id="4fb3b-204">Microsoft authentication</span></span>](xref:security/authentication/microsoft-logins)
* [<span data-ttu-id="4fb3b-205">Autenticazione Twitter</span><span class="sxs-lookup"><span data-stu-id="4fb3b-205">Twitter authentication</span></span>](xref:security/authentication/twitter-logins)
* [<span data-ttu-id="4fb3b-206">Altri provider di autenticazione</span><span class="sxs-lookup"><span data-stu-id="4fb3b-206">Other authentication providers</span></span>](xref:security/authentication/otherlogins)
* [<span data-ttu-id="4fb3b-207">OpenIdConnect</span><span class="sxs-lookup"><span data-stu-id="4fb3b-207">OpenIdConnect</span></span>](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2)

<span data-ttu-id="4fb3b-208">L'app di esempio configura il provider di autenticazione Google con un ID client e un segreto client forniti da Google:</span><span class="sxs-lookup"><span data-stu-id="4fb3b-208">The sample app configures the Google authentication provider with a client ID and client secret provided by Google:</span></span>

[!code-csharp[](additional-claims/samples/2.x/ClaimsSample/Startup.cs?name=snippet_AddGoogle&highlight=4,9)]

## <a name="establish-the-authentication-scope"></a><span data-ttu-id="4fb3b-209">Definire l'ambito di autenticazione</span><span class="sxs-lookup"><span data-stu-id="4fb3b-209">Establish the authentication scope</span></span>

<span data-ttu-id="4fb3b-210">Specificare l'elenco di autorizzazioni da recuperare dal provider specificando <xref:Microsoft.AspNetCore.Authentication.OAuth.OAuthOptions.Scope*> .</span><span class="sxs-lookup"><span data-stu-id="4fb3b-210">Specify the list of permissions to retrieve from the provider by specifying the <xref:Microsoft.AspNetCore.Authentication.OAuth.OAuthOptions.Scope*>.</span></span> <span data-ttu-id="4fb3b-211">Gli ambiti di autenticazione per i provider esterni comuni sono riportati nella tabella seguente.</span><span class="sxs-lookup"><span data-stu-id="4fb3b-211">Authentication scopes for common external providers appear in the following table.</span></span>

| <span data-ttu-id="4fb3b-212">Provider</span><span class="sxs-lookup"><span data-stu-id="4fb3b-212">Provider</span></span>  | <span data-ttu-id="4fb3b-213">Ambito</span><span class="sxs-lookup"><span data-stu-id="4fb3b-213">Scope</span></span>                                                            |
| --------- | ---------------------------------------------------------------- |
| <span data-ttu-id="4fb3b-214">Facebook</span><span class="sxs-lookup"><span data-stu-id="4fb3b-214">Facebook</span></span>  | `https://www.facebook.com/dialog/oauth`                          |
| <span data-ttu-id="4fb3b-215">Google</span><span class="sxs-lookup"><span data-stu-id="4fb3b-215">Google</span></span>    | `https://www.googleapis.com/auth/userinfo.profile`               |
| <span data-ttu-id="4fb3b-216">Microsoft</span><span class="sxs-lookup"><span data-stu-id="4fb3b-216">Microsoft</span></span> | `https://login.microsoftonline.com/common/oauth2/v2.0/authorize` |
| <span data-ttu-id="4fb3b-217">Twitter</span><span class="sxs-lookup"><span data-stu-id="4fb3b-217">Twitter</span></span>   | `https://api.twitter.com/oauth/authenticate`                     |

<span data-ttu-id="4fb3b-218">Nell'app di esempio, `userinfo.profile` l'ambito di Google viene aggiunto automaticamente dal framework quando <xref:Microsoft.Extensions.DependencyInjection.GoogleExtensions.AddGoogle*> viene chiamato su <xref:Microsoft.AspNetCore.Authentication.AuthenticationBuilder> .</span><span class="sxs-lookup"><span data-stu-id="4fb3b-218">In the sample app, Google's `userinfo.profile` scope is automatically added by the framework when <xref:Microsoft.Extensions.DependencyInjection.GoogleExtensions.AddGoogle*> is called on the <xref:Microsoft.AspNetCore.Authentication.AuthenticationBuilder>.</span></span> <span data-ttu-id="4fb3b-219">Se l'app richiede ambiti aggiuntivi, aggiungerli alle opzioni.</span><span class="sxs-lookup"><span data-stu-id="4fb3b-219">If the app requires additional scopes, add them to the options.</span></span> <span data-ttu-id="4fb3b-220">Nell'esempio seguente `https://www.googleapis.com/auth/user.birthday.read` viene aggiunto l'ambito Google per recuperare il compleanno di un utente:</span><span class="sxs-lookup"><span data-stu-id="4fb3b-220">In the following example, the Google `https://www.googleapis.com/auth/user.birthday.read` scope is added in order to retrieve a user's birthday:</span></span>

```csharp
options.Scope.Add("https://www.googleapis.com/auth/user.birthday.read");
```

## <a name="map-user-data-keys-and-create-claims"></a><span data-ttu-id="4fb3b-221">Eseguire il mapping delle chiavi di dati utente e creare attestazioni</span><span class="sxs-lookup"><span data-stu-id="4fb3b-221">Map user data keys and create claims</span></span>

<span data-ttu-id="4fb3b-222">Nelle opzioni del provider specificare un <xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionMapExtensions.MapJsonKey*> o <xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionMapExtensions.MapJsonSubKey*> per ogni chiave/sottochiave nei dati utente JSON del provider esterno per l'identità dell'app da leggere durante l'accesso.</span><span class="sxs-lookup"><span data-stu-id="4fb3b-222">In the provider's options, specify a <xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionMapExtensions.MapJsonKey*> or <xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionMapExtensions.MapJsonSubKey*> for each key/subkey in the external provider's JSON user data for the app identity to read on sign in.</span></span> <span data-ttu-id="4fb3b-223">Per ulteriori informazioni sui tipi di attestazione, vedere <xref:System.Security.Claims.ClaimTypes> .</span><span class="sxs-lookup"><span data-stu-id="4fb3b-223">For more information on claim types, see <xref:System.Security.Claims.ClaimTypes>.</span></span>

<span data-ttu-id="4fb3b-224">L'app di esempio crea attestazioni delle impostazioni locali ( `urn:google:locale` ) e immagini ( `urn:google:picture` ) dalle `locale` `picture` chiavi e nei dati utente di Google:</span><span class="sxs-lookup"><span data-stu-id="4fb3b-224">The sample app creates locale (`urn:google:locale`) and picture (`urn:google:picture`) claims from the `locale` and `picture` keys in Google user data:</span></span>

[!code-csharp[](additional-claims/samples/2.x/ClaimsSample/Startup.cs?name=snippet_AddGoogle&highlight=13-14)]

<span data-ttu-id="4fb3b-225">In `Microsoft.AspNetCore.Identity.UI.Pages.Account.Internal.ExternalLoginModel.OnPostConfirmationAsync` , un <xref:Microsoft.AspNetCore.Identity.IdentityUser> ( `ApplicationUser` ) è connesso all'app con <xref:Microsoft.AspNetCore.Identity.SignInManager%601.SignInAsync*> .</span><span class="sxs-lookup"><span data-stu-id="4fb3b-225">In `Microsoft.AspNetCore.Identity.UI.Pages.Account.Internal.ExternalLoginModel.OnPostConfirmationAsync`, an <xref:Microsoft.AspNetCore.Identity.IdentityUser> (`ApplicationUser`) is signed into the app with <xref:Microsoft.AspNetCore.Identity.SignInManager%601.SignInAsync*>.</span></span> <span data-ttu-id="4fb3b-226">Durante il processo di accesso, il <xref:Microsoft.AspNetCore.Identity.UserManager%601> può archiviare `ApplicationUser` attestazioni per i dati utente disponibili da <xref:Microsoft.AspNetCore.Identity.ExternalLoginInfo.Principal*> .</span><span class="sxs-lookup"><span data-stu-id="4fb3b-226">During the sign in process, the <xref:Microsoft.AspNetCore.Identity.UserManager%601> can store an `ApplicationUser` claims for user data available from the <xref:Microsoft.AspNetCore.Identity.ExternalLoginInfo.Principal*>.</span></span>

<span data-ttu-id="4fb3b-227">Nell'app di esempio `OnPostConfirmationAsync` (*account/ExternalLogin. cshtml. cs*) vengono stabilite le attestazioni delle impostazioni locali ( `urn:google:locale` ) e immagine ( `urn:google:picture` ) per l'accesso `ApplicationUser` , inclusa un'attestazione per <xref:System.Security.Claims.ClaimTypes.GivenName> :</span><span class="sxs-lookup"><span data-stu-id="4fb3b-227">In the sample app, `OnPostConfirmationAsync` (*Account/ExternalLogin.cshtml.cs*) establishes the locale (`urn:google:locale`) and picture (`urn:google:picture`) claims for the signed in `ApplicationUser`, including a claim for <xref:System.Security.Claims.ClaimTypes.GivenName>:</span></span>

[!code-csharp[](additional-claims/samples/2.x/ClaimsSample/Areas/Identity/Pages/Account/ExternalLogin.cshtml.cs?name=snippet_OnPostConfirmationAsync&highlight=35-51)]

<span data-ttu-id="4fb3b-228">Per impostazione predefinita, le attestazioni di un utente vengono archiviate nell'autenticazione di cookie .</span><span class="sxs-lookup"><span data-stu-id="4fb3b-228">By default, a user's claims are stored in the authentication cookie.</span></span> <span data-ttu-id="4fb3b-229">Se l'autenticazione cookie è troppo grande, può causare un errore dell'app perché:</span><span class="sxs-lookup"><span data-stu-id="4fb3b-229">If the authentication cookie is too large, it can cause the app to fail because:</span></span>

* <span data-ttu-id="4fb3b-230">Il browser rileva che l' cookie intestazione è troppo lungo.</span><span class="sxs-lookup"><span data-stu-id="4fb3b-230">The browser detects that the cookie header is too long.</span></span>
* <span data-ttu-id="4fb3b-231">La dimensione complessiva della richiesta è troppo grande.</span><span class="sxs-lookup"><span data-stu-id="4fb3b-231">The overall size of the request is too large.</span></span>

<span data-ttu-id="4fb3b-232">Se per l'elaborazione delle richieste utente è richiesta una grande quantità di dati utente:</span><span class="sxs-lookup"><span data-stu-id="4fb3b-232">If a large amount of user data is required for processing user requests:</span></span>

* <span data-ttu-id="4fb3b-233">Limitare il numero e le dimensioni delle attestazioni utente per l'elaborazione delle richieste solo ai componenti richiesti dall'app.</span><span class="sxs-lookup"><span data-stu-id="4fb3b-233">Limit the number and size of user claims for request processing to only what the app requires.</span></span>
* <span data-ttu-id="4fb3b-234">Usare un personalizzato <xref:Microsoft.AspNetCore.Authentication.Cookies.ITicketStore> per il Cookie middleware di autenticazione <xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationOptions.SessionStore> per archiviare l'identità tra le richieste.</span><span class="sxs-lookup"><span data-stu-id="4fb3b-234">Use a custom <xref:Microsoft.AspNetCore.Authentication.Cookies.ITicketStore> for the Cookie Authentication Middleware's <xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationOptions.SessionStore> to store identity across requests.</span></span> <span data-ttu-id="4fb3b-235">Mantenere grandi quantità di informazioni sull'identità sul server inviando al client solo una chiave di identificatore di sessione di piccole dimensioni.</span><span class="sxs-lookup"><span data-stu-id="4fb3b-235">Preserve large quantities of identity information on the server while only sending a small session identifier key to the client.</span></span>

## <a name="save-the-access-token"></a><span data-ttu-id="4fb3b-236">Salvare il token di accesso</span><span class="sxs-lookup"><span data-stu-id="4fb3b-236">Save the access token</span></span>

<span data-ttu-id="4fb3b-237"><xref:Microsoft.AspNetCore.Authentication.RemoteAuthenticationOptions.SaveTokens*> definisce se i token di accesso e di aggiornamento devono essere archiviati in <xref:Microsoft.AspNetCore.Http.Authentication.AuthenticationProperties> dopo un'autorizzazione riuscita.</span><span class="sxs-lookup"><span data-stu-id="4fb3b-237"><xref:Microsoft.AspNetCore.Authentication.RemoteAuthenticationOptions.SaveTokens*> defines whether access and refresh tokens should be stored in the <xref:Microsoft.AspNetCore.Http.Authentication.AuthenticationProperties> after a successful authorization.</span></span> <span data-ttu-id="4fb3b-238">`SaveTokens` è impostato su per `false` impostazione predefinita per ridurre le dimensioni dell'autenticazione finale cookie .</span><span class="sxs-lookup"><span data-stu-id="4fb3b-238">`SaveTokens` is set to `false` by default to reduce the size of the final authentication cookie.</span></span>

<span data-ttu-id="4fb3b-239">L'app di esempio imposta il valore di `SaveTokens` su `true` in <xref:Microsoft.AspNetCore.Authentication.Google.GoogleOptions> :</span><span class="sxs-lookup"><span data-stu-id="4fb3b-239">The sample app sets the value of `SaveTokens` to `true` in <xref:Microsoft.AspNetCore.Authentication.Google.GoogleOptions>:</span></span>

[!code-csharp[](additional-claims/samples/2.x/ClaimsSample/Startup.cs?name=snippet_AddGoogle&highlight=15)]

<span data-ttu-id="4fb3b-240">Quando `OnPostConfirmationAsync` viene eseguito, archivia il token di accesso ([ExternalLoginInfo. AuthenticationTokens](xref:Microsoft.AspNetCore.Identity.ExternalLoginInfo.AuthenticationTokens*)) dal provider esterno in `ApplicationUser` `AuthenticationProperties` .</span><span class="sxs-lookup"><span data-stu-id="4fb3b-240">When `OnPostConfirmationAsync` executes, store the access token ([ExternalLoginInfo.AuthenticationTokens](xref:Microsoft.AspNetCore.Identity.ExternalLoginInfo.AuthenticationTokens*)) from the external provider in the `ApplicationUser`'s `AuthenticationProperties`.</span></span>

<span data-ttu-id="4fb3b-241">L'app di esempio salva il token di accesso in `OnPostConfirmationAsync` (nuova registrazione utente) e `OnGetCallbackAsync` (utente registrato in precedenza) in *account/ExternalLogin. cshtml. cs*:</span><span class="sxs-lookup"><span data-stu-id="4fb3b-241">The sample app saves the access token in `OnPostConfirmationAsync` (new user registration) and `OnGetCallbackAsync` (previously registered user) in *Account/ExternalLogin.cshtml.cs*:</span></span>

[!code-csharp[](additional-claims/samples/2.x/ClaimsSample/Areas/Identity/Pages/Account/ExternalLogin.cshtml.cs?name=snippet_OnPostConfirmationAsync&highlight=54-56)]

## <a name="how-to-add-additional-custom-tokens"></a><span data-ttu-id="4fb3b-242">Come aggiungere token personalizzati aggiuntivi</span><span class="sxs-lookup"><span data-stu-id="4fb3b-242">How to add additional custom tokens</span></span>

<span data-ttu-id="4fb3b-243">Per illustrare come aggiungere un token personalizzato, archiviato come parte di `SaveTokens` , l'app di esempio aggiunge un oggetto <xref:Microsoft.AspNetCore.Authentication.AuthenticationToken> con l'oggetto corrente <xref:System.DateTime> per un [AuthenticationToken.Name](xref:Microsoft.AspNetCore.Authentication.AuthenticationToken.Name*) di `TicketCreated` :</span><span class="sxs-lookup"><span data-stu-id="4fb3b-243">To demonstrate how to add a custom token, which is stored as part of `SaveTokens`, the sample app adds an <xref:Microsoft.AspNetCore.Authentication.AuthenticationToken> with the current <xref:System.DateTime> for an [AuthenticationToken.Name](xref:Microsoft.AspNetCore.Authentication.AuthenticationToken.Name*) of `TicketCreated`:</span></span>

[!code-csharp[](additional-claims/samples/2.x/ClaimsSample/Startup.cs?name=snippet_AddGoogle&highlight=17-30)]

## <a name="creating-and-adding-claims"></a><span data-ttu-id="4fb3b-244">Creazione e aggiunta di attestazioni</span><span class="sxs-lookup"><span data-stu-id="4fb3b-244">Creating and adding claims</span></span>

<span data-ttu-id="4fb3b-245">Il Framework fornisce azioni comuni e metodi di estensione per la creazione e l'aggiunta di attestazioni alla raccolta.</span><span class="sxs-lookup"><span data-stu-id="4fb3b-245">The framework provides common actions and extension methods for creating and adding claims to the collection.</span></span> <span data-ttu-id="4fb3b-246">Per altre informazioni, vedere <xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionMapExtensions> e <xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionUniqueExtensions>.</span><span class="sxs-lookup"><span data-stu-id="4fb3b-246">For more information, see the <xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionMapExtensions> and <xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionUniqueExtensions>.</span></span>

<span data-ttu-id="4fb3b-247">Gli utenti possono definire azioni personalizzate derivando da <xref:Microsoft.AspNetCore.Authentication.OAuth.Claims.ClaimAction> e implementando il <xref:Microsoft.AspNetCore.Authentication.OAuth.Claims.ClaimAction.Run*> metodo astratto.</span><span class="sxs-lookup"><span data-stu-id="4fb3b-247">Users can define custom actions by deriving from <xref:Microsoft.AspNetCore.Authentication.OAuth.Claims.ClaimAction> and implementing the abstract <xref:Microsoft.AspNetCore.Authentication.OAuth.Claims.ClaimAction.Run*> method.</span></span>

<span data-ttu-id="4fb3b-248">Per altre informazioni, vedere <xref:Microsoft.AspNetCore.Authentication.OAuth.Claims>.</span><span class="sxs-lookup"><span data-stu-id="4fb3b-248">For more information, see <xref:Microsoft.AspNetCore.Authentication.OAuth.Claims>.</span></span>

## <a name="removal-of-claim-actions-and-claims"></a><span data-ttu-id="4fb3b-249">Rimozione di azioni attestazioni e attestazioni</span><span class="sxs-lookup"><span data-stu-id="4fb3b-249">Removal of claim actions and claims</span></span>

<span data-ttu-id="4fb3b-250">[ClaimActionCollection. Remove (String)](xref:Microsoft.AspNetCore.Authentication.OAuth.Claims.ClaimActionCollection.Remove*) rimuove tutte le azioni di attestazione per l'oggetto specificato <xref:Microsoft.AspNetCore.Authentication.OAuth.Claims.ClaimAction.ClaimType> dalla raccolta.</span><span class="sxs-lookup"><span data-stu-id="4fb3b-250">[ClaimActionCollection.Remove(String)](xref:Microsoft.AspNetCore.Authentication.OAuth.Claims.ClaimActionCollection.Remove*) removes all claim actions for the given <xref:Microsoft.AspNetCore.Authentication.OAuth.Claims.ClaimAction.ClaimType> from the collection.</span></span> <span data-ttu-id="4fb3b-251">[ClaimActionCollectionMapExtensions. DeleteClaim (ClaimActionCollection, String)](xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionMapExtensions.DeleteClaim*) Elimina un'attestazione dell'oggetto specificato <xref:Microsoft.AspNetCore.Authentication.OAuth.Claims.ClaimAction.ClaimType> dall'identità.</span><span class="sxs-lookup"><span data-stu-id="4fb3b-251">[ClaimActionCollectionMapExtensions.DeleteClaim(ClaimActionCollection, String)](xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionMapExtensions.DeleteClaim*) deletes a claim of the given <xref:Microsoft.AspNetCore.Authentication.OAuth.Claims.ClaimAction.ClaimType> from the identity.</span></span> <span data-ttu-id="4fb3b-252"><xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionMapExtensions.DeleteClaim*> viene utilizzato principalmente con [OpenID Connect (OIDC)](/azure/active-directory/develop/v2-protocols-oidc) per rimuovere le attestazioni generate dal protocollo.</span><span class="sxs-lookup"><span data-stu-id="4fb3b-252"><xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionMapExtensions.DeleteClaim*> is primarily used with [OpenID Connect (OIDC)](/azure/active-directory/develop/v2-protocols-oidc) to remove protocol-generated claims.</span></span>

## <a name="sample-app-output"></a><span data-ttu-id="4fb3b-253">Esempio di output dell'app</span><span class="sxs-lookup"><span data-stu-id="4fb3b-253">Sample app output</span></span>

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

## <a name="additional-resources"></a><span data-ttu-id="4fb3b-254">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="4fb3b-254">Additional resources</span></span>

* <span data-ttu-id="4fb3b-255">[app SocialSample di progettazione DotNet/AspNetCore](https://github.com/dotnet/AspNetCore/tree/master/src/Security/Authentication/samples/SocialSample): l'app di esempio collegata si trova nel ramo di progettazione del [repository GitHub DotNet/AspNetCore](https://github.com/dotnet/AspNetCore) `master` .</span><span class="sxs-lookup"><span data-stu-id="4fb3b-255">[dotnet/AspNetCore engineering SocialSample app](https://github.com/dotnet/AspNetCore/tree/master/src/Security/Authentication/samples/SocialSample): The linked sample app is on the [dotnet/AspNetCore GitHub repo's](https://github.com/dotnet/AspNetCore) `master` engineering branch.</span></span> <span data-ttu-id="4fb3b-256">Il `master` ramo contiene il codice in fase di sviluppo attivo per la prossima versione di ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="4fb3b-256">The `master` branch contains code under active development for the next release of ASP.NET Core.</span></span> <span data-ttu-id="4fb3b-257">Per visualizzare una versione dell'app di esempio per una versione rilasciata di ASP.NET Core, usare l'elenco a discesa **Branch** per selezionare un ramo di rilascio (ad esempio `release/{X.Y}` ).</span><span class="sxs-lookup"><span data-stu-id="4fb3b-257">To see a version of the sample app for a released version of ASP.NET Core, use the **Branch** drop down list to select a release branch (for example `release/{X.Y}`).</span></span>
