---
title: USA cookie autenticazione senza ASP.NET Core Identity
author: rick-anderson
description: Informazioni su come usare cookie l'autenticazione senza ASP.NET Core Identity .
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.date: 02/11/2020
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
uid: security/authentication/cookie
ms.openlocfilehash: 5b1f1bb3de7126c401a81b89b99a45c7e45f8f8d
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/10/2021
ms.locfileid: "102586293"
---
# <a name="use-cookie-authentication-without-aspnet-core-identity"></a><span data-ttu-id="aff71-103">USA cookie autenticazione senza ASP.NET Core Identity</span><span class="sxs-lookup"><span data-stu-id="aff71-103">Use cookie authentication without ASP.NET Core Identity</span></span>

<span data-ttu-id="aff71-104">Autore: [Rick Anderson](https://twitter.com/RickAndMSFT)</span><span class="sxs-lookup"><span data-stu-id="aff71-104">By [Rick Anderson](https://twitter.com/RickAndMSFT)</span></span>

::: moniker range=">= aspnetcore-3.0"

<span data-ttu-id="aff71-105">ASP.NET Core Identity è un provider di autenticazione completo e completo per la creazione e la gestione degli account di accesso.</span><span class="sxs-lookup"><span data-stu-id="aff71-105">ASP.NET Core Identity is a complete, full-featured authentication provider for creating and maintaining logins.</span></span> <span data-ttu-id="aff71-106">Tuttavia, un cookie provider di autenticazione basato su senza ASP.NET Core Identity può essere utilizzato.</span><span class="sxs-lookup"><span data-stu-id="aff71-106">However, a cookie-based authentication provider without ASP.NET Core Identity can be used.</span></span> <span data-ttu-id="aff71-107">Per altre informazioni, vedere <xref:security/authentication/identity>.</span><span class="sxs-lookup"><span data-stu-id="aff71-107">For more information, see <xref:security/authentication/identity>.</span></span>

<span data-ttu-id="aff71-108">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/authentication/cookie/samples) ([procedura per il download](xref:index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="aff71-108">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/authentication/cookie/samples) ([how to download](xref:index#how-to-download-a-sample))</span></span>

<span data-ttu-id="aff71-109">A scopo dimostrativo nell'app di esempio, l'account utente per l'utente ipotetico, Maria Rodriguez, è hardcoded nell'app.</span><span class="sxs-lookup"><span data-stu-id="aff71-109">For demonstration purposes in the sample app, the user account for the hypothetical user, Maria Rodriguez, is hardcoded into the app.</span></span> <span data-ttu-id="aff71-110">Usare l'indirizzo di **posta elettronica** `maria.rodriguez@contoso.com` e qualsiasi password per accedere all'utente.</span><span class="sxs-lookup"><span data-stu-id="aff71-110">Use the **Email** address `maria.rodriguez@contoso.com` and any password to sign in the user.</span></span> <span data-ttu-id="aff71-111">L'utente viene autenticato nel `AuthenticateUser` metodo nel file *pages/account/login. cshtml. cs* .</span><span class="sxs-lookup"><span data-stu-id="aff71-111">The user is authenticated in the `AuthenticateUser` method in the *Pages/Account/Login.cshtml.cs* file.</span></span> <span data-ttu-id="aff71-112">In un esempio reale, l'utente viene autenticato a fronte di un database.</span><span class="sxs-lookup"><span data-stu-id="aff71-112">In a real-world example, the user would be authenticated against a database.</span></span>

## <a name="configuration"></a><span data-ttu-id="aff71-113">Configurazione</span><span class="sxs-lookup"><span data-stu-id="aff71-113">Configuration</span></span>

<span data-ttu-id="aff71-114">Nel `Startup.ConfigureServices` metodo creare i servizi del middleware di autenticazione con i <xref:Microsoft.Extensions.DependencyInjection.AuthenticationServiceCollectionExtensions.AddAuthentication*> <xref:Microsoft.Extensions.DependencyInjection.CookieExtensions.AddCookie*> metodi e:</span><span class="sxs-lookup"><span data-stu-id="aff71-114">In the `Startup.ConfigureServices` method, create the Authentication Middleware services with the <xref:Microsoft.Extensions.DependencyInjection.AuthenticationServiceCollectionExtensions.AddAuthentication*> and <xref:Microsoft.Extensions.DependencyInjection.CookieExtensions.AddCookie*> methods:</span></span>

[!code-csharp[](cookie/samples/3.x/CookieSample/Startup.cs?name=snippet1)]

<span data-ttu-id="aff71-115"><xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationDefaults.AuthenticationScheme> passato a `AddAuthentication` imposta lo schema di autenticazione predefinito per l'app.</span><span class="sxs-lookup"><span data-stu-id="aff71-115"><xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationDefaults.AuthenticationScheme> passed to `AddAuthentication` sets the default authentication scheme for the app.</span></span> <span data-ttu-id="aff71-116">`AuthenticationScheme` è utile quando sono presenti più istanze di cookie autenticazione e si desidera [autorizzare con uno schema specifico](xref:security/authorization/limitingidentitybyscheme).</span><span class="sxs-lookup"><span data-stu-id="aff71-116">`AuthenticationScheme` is useful when there are multiple instances of cookie authentication and you want to [authorize with a specific scheme](xref:security/authorization/limitingidentitybyscheme).</span></span> <span data-ttu-id="aff71-117">L'impostazione `AuthenticationScheme` di su [ Cookie AuthenticationDefaults. AuthenticationScheme](xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationDefaults.AuthenticationScheme) fornisce il valore " Cookie s" per lo schema.</span><span class="sxs-lookup"><span data-stu-id="aff71-117">Setting the `AuthenticationScheme` to [CookieAuthenticationDefaults.AuthenticationScheme](xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationDefaults.AuthenticationScheme) provides a value of "Cookies" for the scheme.</span></span> <span data-ttu-id="aff71-118">È possibile specificare qualsiasi valore stringa che distingua lo schema.</span><span class="sxs-lookup"><span data-stu-id="aff71-118">You can supply any string value that distinguishes the scheme.</span></span>

<span data-ttu-id="aff71-119">Lo schema di autenticazione dell'app è diverso dallo schema di cookie autenticazione dell'app.</span><span class="sxs-lookup"><span data-stu-id="aff71-119">The app's authentication scheme is different from the app's cookie authentication scheme.</span></span> <span data-ttu-id="aff71-120">Quando uno cookie schema di autenticazione non viene fornito a <xref:Microsoft.Extensions.DependencyInjection.CookieExtensions.AddCookie*> , USA `CookieAuthenticationDefaults.AuthenticationScheme` (" Cookie s").</span><span class="sxs-lookup"><span data-stu-id="aff71-120">When a cookie authentication scheme isn't provided to <xref:Microsoft.Extensions.DependencyInjection.CookieExtensions.AddCookie*>, it uses `CookieAuthenticationDefaults.AuthenticationScheme` ("Cookies").</span></span>

<span data-ttu-id="aff71-121">La cookie proprietà Authentication <xref:Microsoft.AspNetCore.Http.CookieBuilder.IsEssential> è impostata su per `true` impostazione predefinita.</span><span class="sxs-lookup"><span data-stu-id="aff71-121">The authentication cookie's <xref:Microsoft.AspNetCore.Http.CookieBuilder.IsEssential> property is set to `true` by default.</span></span> <span data-ttu-id="aff71-122">cookieSono consentite le autenticazioni quando un visitatore del sito non ha acconsentito alla raccolta dei dati.</span><span class="sxs-lookup"><span data-stu-id="aff71-122">Authentication cookies are allowed when a site visitor hasn't consented to data collection.</span></span> <span data-ttu-id="aff71-123">Per altre informazioni, vedere <xref:security/gdpr#essential-cookies>.</span><span class="sxs-lookup"><span data-stu-id="aff71-123">For more information, see <xref:security/gdpr#essential-cookies>.</span></span>

<span data-ttu-id="aff71-124">In `Startup.Configure` , chiamare `UseAuthentication` e `UseAuthorization` per impostare la `HttpContext.User` proprietà ed eseguire il middleware di autorizzazione per le richieste.</span><span class="sxs-lookup"><span data-stu-id="aff71-124">In `Startup.Configure`, call `UseAuthentication` and `UseAuthorization` to set the `HttpContext.User` property and run Authorization Middleware for requests.</span></span> <span data-ttu-id="aff71-125">Chiamare i `UseAuthentication` `UseAuthorization` metodi e prima di chiamare `UseEndpoints` :</span><span class="sxs-lookup"><span data-stu-id="aff71-125">Call the `UseAuthentication` and `UseAuthorization` methods before calling `UseEndpoints`:</span></span>

[!code-csharp[](cookie/samples/3.x/CookieSample/Startup.cs?name=snippet2)]

<span data-ttu-id="aff71-126">La <xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationOptions> classe viene utilizzata per configurare le opzioni del provider di autenticazione.</span><span class="sxs-lookup"><span data-stu-id="aff71-126">The <xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationOptions> class is used to configure the authentication provider options.</span></span>

<span data-ttu-id="aff71-127">Impostare `CookieAuthenticationOptions` nella configurazione del servizio per l'autenticazione nel `Startup.ConfigureServices` Metodo:</span><span class="sxs-lookup"><span data-stu-id="aff71-127">Set `CookieAuthenticationOptions` in the service configuration for authentication in the `Startup.ConfigureServices` method:</span></span>

```csharp
services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(options =>
    {
        ...
    });
```

## <a name="cookie-policy-middleware"></a><span data-ttu-id="aff71-128">Cookie Middleware dei criteri</span><span class="sxs-lookup"><span data-stu-id="aff71-128">Cookie Policy Middleware</span></span>

<span data-ttu-id="aff71-129">Il [ Cookie middleware dei criteri](xref:Microsoft.AspNetCore.CookiePolicy.CookiePolicyMiddleware) Abilita le cookie funzionalità dei criteri.</span><span class="sxs-lookup"><span data-stu-id="aff71-129">[Cookie Policy Middleware](xref:Microsoft.AspNetCore.CookiePolicy.CookiePolicyMiddleware) enables cookie policy capabilities.</span></span> <span data-ttu-id="aff71-130">L'aggiunta del middleware alla pipeline di elaborazione delle app è sensibile agli ordini &mdash; , ma interessa solo i componenti downstream registrati nella pipeline.</span><span class="sxs-lookup"><span data-stu-id="aff71-130">Adding the middleware to the app processing pipeline is order sensitive&mdash;it only affects downstream components registered in the pipeline.</span></span>

```csharp
app.UseCookiePolicy(cookiePolicyOptions);
```

<span data-ttu-id="aff71-131">Utilizzare <xref:Microsoft.AspNetCore.Builder.CookiePolicyOptions> fornito al Cookie middleware dei criteri per controllare le caratteristiche globali dell' cookie elaborazione e agganciare i cookie gestori di elaborazione quando cookie vengono aggiunti o eliminati.</span><span class="sxs-lookup"><span data-stu-id="aff71-131">Use <xref:Microsoft.AspNetCore.Builder.CookiePolicyOptions> provided to the Cookie Policy Middleware to control global characteristics of cookie processing and hook into cookie processing handlers when cookies are appended or deleted.</span></span>

<span data-ttu-id="aff71-132">Il <xref:Microsoft.AspNetCore.Builder.CookiePolicyOptions.MinimumSameSitePolicy> valore predefinito è `SameSiteMode.Lax` consentire l'autenticazione OAuth2.</span><span class="sxs-lookup"><span data-stu-id="aff71-132">The default <xref:Microsoft.AspNetCore.Builder.CookiePolicyOptions.MinimumSameSitePolicy> value is `SameSiteMode.Lax` to permit OAuth2 authentication.</span></span> <span data-ttu-id="aff71-133">Per applicare rigorosamente un criterio dello stesso sito di `SameSiteMode.Strict` , impostare il `MinimumSameSitePolicy` .</span><span class="sxs-lookup"><span data-stu-id="aff71-133">To strictly enforce a same-site policy of `SameSiteMode.Strict`, set the `MinimumSameSitePolicy`.</span></span> <span data-ttu-id="aff71-134">Sebbene questa impostazione interrompa OAuth2 e altri schemi di autenticazione tra le origini, eleva il livello di cookie sicurezza per altri tipi di app che non si basano sull'elaborazione di richieste tra origini.</span><span class="sxs-lookup"><span data-stu-id="aff71-134">Although this setting breaks OAuth2 and other cross-origin authentication schemes, it elevates the level of cookie security for other types of apps that don't rely on cross-origin request processing.</span></span>

```csharp
var cookiePolicyOptions = new CookiePolicyOptions
{
    MinimumSameSitePolicy = SameSiteMode.Strict,
};
```

<span data-ttu-id="aff71-135">L' Cookie impostazione del middleware per i criteri di `MinimumSameSitePolicy` può influire sull'impostazione di `Cookie.SameSite` in `CookieAuthenticationOptions` impostazioni in base alla matrice riportata di seguito.</span><span class="sxs-lookup"><span data-stu-id="aff71-135">The Cookie Policy Middleware setting for `MinimumSameSitePolicy` can affect the setting of `Cookie.SameSite` in `CookieAuthenticationOptions` settings according to the matrix below.</span></span>

| <span data-ttu-id="aff71-136">MinimumSameSitePolicy</span><span class="sxs-lookup"><span data-stu-id="aff71-136">MinimumSameSitePolicy</span></span> | <span data-ttu-id="aff71-137">Cookie. Navigava sullostesso sito</span><span class="sxs-lookup"><span data-stu-id="aff71-137">Cookie.SameSite</span></span> | <span data-ttu-id="aff71-138">Risultato Cookie . Impostazione navigava sullostesso sito</span><span class="sxs-lookup"><span data-stu-id="aff71-138">Resultant Cookie.SameSite setting</span></span> |
| --------------------- | --------------- | --------------------------------- |
| <span data-ttu-id="aff71-139">SameSiteMode. None</span><span class="sxs-lookup"><span data-stu-id="aff71-139">SameSiteMode.None</span></span>     | <span data-ttu-id="aff71-140">SameSiteMode. None</span><span class="sxs-lookup"><span data-stu-id="aff71-140">SameSiteMode.None</span></span><br><span data-ttu-id="aff71-141">SameSiteMode. LAX</span><span class="sxs-lookup"><span data-stu-id="aff71-141">SameSiteMode.Lax</span></span><br><span data-ttu-id="aff71-142">SameSiteMode. Strict</span><span class="sxs-lookup"><span data-stu-id="aff71-142">SameSiteMode.Strict</span></span> | <span data-ttu-id="aff71-143">SameSiteMode. None</span><span class="sxs-lookup"><span data-stu-id="aff71-143">SameSiteMode.None</span></span><br><span data-ttu-id="aff71-144">SameSiteMode. LAX</span><span class="sxs-lookup"><span data-stu-id="aff71-144">SameSiteMode.Lax</span></span><br><span data-ttu-id="aff71-145">SameSiteMode. Strict</span><span class="sxs-lookup"><span data-stu-id="aff71-145">SameSiteMode.Strict</span></span> |
| <span data-ttu-id="aff71-146">SameSiteMode. LAX</span><span class="sxs-lookup"><span data-stu-id="aff71-146">SameSiteMode.Lax</span></span>      | <span data-ttu-id="aff71-147">SameSiteMode. None</span><span class="sxs-lookup"><span data-stu-id="aff71-147">SameSiteMode.None</span></span><br><span data-ttu-id="aff71-148">SameSiteMode. LAX</span><span class="sxs-lookup"><span data-stu-id="aff71-148">SameSiteMode.Lax</span></span><br><span data-ttu-id="aff71-149">SameSiteMode. Strict</span><span class="sxs-lookup"><span data-stu-id="aff71-149">SameSiteMode.Strict</span></span> | <span data-ttu-id="aff71-150">SameSiteMode. LAX</span><span class="sxs-lookup"><span data-stu-id="aff71-150">SameSiteMode.Lax</span></span><br><span data-ttu-id="aff71-151">SameSiteMode. LAX</span><span class="sxs-lookup"><span data-stu-id="aff71-151">SameSiteMode.Lax</span></span><br><span data-ttu-id="aff71-152">SameSiteMode. Strict</span><span class="sxs-lookup"><span data-stu-id="aff71-152">SameSiteMode.Strict</span></span> |
| <span data-ttu-id="aff71-153">SameSiteMode. Strict</span><span class="sxs-lookup"><span data-stu-id="aff71-153">SameSiteMode.Strict</span></span>   | <span data-ttu-id="aff71-154">SameSiteMode. None</span><span class="sxs-lookup"><span data-stu-id="aff71-154">SameSiteMode.None</span></span><br><span data-ttu-id="aff71-155">SameSiteMode. LAX</span><span class="sxs-lookup"><span data-stu-id="aff71-155">SameSiteMode.Lax</span></span><br><span data-ttu-id="aff71-156">SameSiteMode. Strict</span><span class="sxs-lookup"><span data-stu-id="aff71-156">SameSiteMode.Strict</span></span> | <span data-ttu-id="aff71-157">SameSiteMode. Strict</span><span class="sxs-lookup"><span data-stu-id="aff71-157">SameSiteMode.Strict</span></span><br><span data-ttu-id="aff71-158">SameSiteMode. Strict</span><span class="sxs-lookup"><span data-stu-id="aff71-158">SameSiteMode.Strict</span></span><br><span data-ttu-id="aff71-159">SameSiteMode. Strict</span><span class="sxs-lookup"><span data-stu-id="aff71-159">SameSiteMode.Strict</span></span> |

## <a name="create-an-authentication-cookie"></a><span data-ttu-id="aff71-160">Creare un'autenticazione cookie</span><span class="sxs-lookup"><span data-stu-id="aff71-160">Create an authentication cookie</span></span>

<span data-ttu-id="aff71-161">Per creare un oggetto che cookie contiene informazioni sull'utente, costruire un oggetto <xref:System.Security.Claims.ClaimsPrincipal> .</span><span class="sxs-lookup"><span data-stu-id="aff71-161">To create a cookie holding user information, construct a <xref:System.Security.Claims.ClaimsPrincipal>.</span></span> <span data-ttu-id="aff71-162">Le informazioni utente vengono serializzate e archiviate in cookie .</span><span class="sxs-lookup"><span data-stu-id="aff71-162">The user information is serialized and stored in the cookie.</span></span> 

<span data-ttu-id="aff71-163">Creare una <xref:System.Security.Claims.ClaimsIdentity> con qualsiasi richiesta <xref:System.Security.Claims.Claim> e chiamare <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.SignInAsync*> per accedere all'utente:</span><span class="sxs-lookup"><span data-stu-id="aff71-163">Create a <xref:System.Security.Claims.ClaimsIdentity> with any required <xref:System.Security.Claims.Claim>s and call <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.SignInAsync*> to sign in the user:</span></span>

[!code-csharp[](cookie/samples/3.x/CookieSample/Pages/Account/Login.cshtml.cs?name=snippet1)]

[!INCLUDE[request localized comments](~/includes/code-comments-loc.md)]

<span data-ttu-id="aff71-164">`SignInAsync` Crea un oggetto crittografato cookie e lo aggiunge alla risposta corrente.</span><span class="sxs-lookup"><span data-stu-id="aff71-164">`SignInAsync` creates an encrypted cookie and adds it to the current response.</span></span> <span data-ttu-id="aff71-165">Se `AuthenticationScheme` non è specificato, viene usato lo schema predefinito.</span><span class="sxs-lookup"><span data-stu-id="aff71-165">If `AuthenticationScheme` isn't specified, the default scheme is used.</span></span>

<span data-ttu-id="aff71-166"><xref:Microsoft.AspNetCore.Authentication.AuthenticationProperties.RedirectUri> viene utilizzato solo in alcuni percorsi specifici per impostazione predefinita, ad esempio il percorso di accesso e i percorsi di disconnessione.</span><span class="sxs-lookup"><span data-stu-id="aff71-166"><xref:Microsoft.AspNetCore.Authentication.AuthenticationProperties.RedirectUri> is only used on a few specific paths by default, for example, the login path and logout paths.</span></span> <span data-ttu-id="aff71-167">Per altre informazioni, vedere l' [ Cookie origine AuthenticationHandler](https://github.com/dotnet/aspnetcore/blob/f2e6e6ff334176540ef0b3291122e359c2106d1a/src/Security/Authentication/Cookies/src/CookieAuthenticationHandler.cs#L334).</span><span class="sxs-lookup"><span data-stu-id="aff71-167">For more information see the [CookieAuthenticationHandler source](https://github.com/dotnet/aspnetcore/blob/f2e6e6ff334176540ef0b3291122e359c2106d1a/src/Security/Authentication/Cookies/src/CookieAuthenticationHandler.cs#L334).</span></span>

<span data-ttu-id="aff71-168">Per la crittografia viene usato il sistema di [protezione dei dati](xref:security/data-protection/using-data-protection) di ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="aff71-168">ASP.NET Core's [Data Protection](xref:security/data-protection/using-data-protection) system is used for encryption.</span></span> <span data-ttu-id="aff71-169">Per un'app ospitata in più computer, bilanciamento del carico tra app o uso di un Web farm, [configurare la protezione dei dati](xref:security/data-protection/configuration/overview) in modo da usare lo stesso anello chiave e l'identificatore dell'app.</span><span class="sxs-lookup"><span data-stu-id="aff71-169">For an app hosted on multiple machines, load balancing across apps, or using a web farm, [configure data protection](xref:security/data-protection/configuration/overview) to use the same key ring and app identifier.</span></span>

## <a name="sign-out"></a><span data-ttu-id="aff71-170">Disconnetti</span><span class="sxs-lookup"><span data-stu-id="aff71-170">Sign out</span></span>

<span data-ttu-id="aff71-171">Per disconnettersi dall'utente corrente ed eliminare il relativo cookie , chiamare <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.SignOutAsync*> :</span><span class="sxs-lookup"><span data-stu-id="aff71-171">To sign out the current user and delete their cookie, call <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.SignOutAsync*>:</span></span>

[!code-csharp[](cookie/samples/3.x/CookieSample/Pages/Account/Login.cshtml.cs?name=snippet2)]

<span data-ttu-id="aff71-172">Se `CookieAuthenticationDefaults.AuthenticationScheme` (o " Cookie s") non viene usato come schema (ad esempio, "Contoso Cookie "), fornire lo schema usato durante la configurazione del provider di autenticazione.</span><span class="sxs-lookup"><span data-stu-id="aff71-172">If `CookieAuthenticationDefaults.AuthenticationScheme` (or "Cookies") isn't used as the scheme (for example, "ContosoCookie"), supply the scheme used when configuring the authentication provider.</span></span> <span data-ttu-id="aff71-173">In caso contrario, verrà utilizzato lo schema predefinito.</span><span class="sxs-lookup"><span data-stu-id="aff71-173">Otherwise, the default scheme is used.</span></span>

<span data-ttu-id="aff71-174">Quando il browser viene chiuso, Elimina automaticamente le istanze di basate su sessione cookie (non persistenti cookie ), ma nessuna viene cookie cancellata quando viene chiusa una singola scheda.</span><span class="sxs-lookup"><span data-stu-id="aff71-174">When the browser closes it automatically deletes session based cookies (non-persistent cookies), but no cookies are cleared when an individual tab is closed.</span></span> <span data-ttu-id="aff71-175">Il server non riceve notifiche sugli eventi di chiusura di schede o browser.</span><span class="sxs-lookup"><span data-stu-id="aff71-175">The server is not notified of tab or browser close events.</span></span>

## <a name="react-to-back-end-changes"></a><span data-ttu-id="aff71-176">Reagire alle modifiche del back-end</span><span class="sxs-lookup"><span data-stu-id="aff71-176">React to back-end changes</span></span>

<span data-ttu-id="aff71-177">Una volta cookie creato un oggetto, cookie è l'unica origine di identità.</span><span class="sxs-lookup"><span data-stu-id="aff71-177">Once a cookie is created, the cookie is the single source of identity.</span></span> <span data-ttu-id="aff71-178">Se un account utente è disabilitato nei sistemi back-end:</span><span class="sxs-lookup"><span data-stu-id="aff71-178">If a user account is disabled in back-end systems:</span></span>

* <span data-ttu-id="aff71-179">Il sistema di cookie autenticazione dell'app continua a elaborare le richieste in base all'autenticazione cookie .</span><span class="sxs-lookup"><span data-stu-id="aff71-179">The app's cookie authentication system continues to process requests based on the authentication cookie.</span></span>
* <span data-ttu-id="aff71-180">L'utente rimane connesso all'app fino a quando l'autenticazione cookie è valida.</span><span class="sxs-lookup"><span data-stu-id="aff71-180">The user remains signed into the app as long as the authentication cookie is valid.</span></span>

<span data-ttu-id="aff71-181">L' <xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationEvents.ValidatePrincipal*> evento può essere usato per intercettare ed eseguire l'override della convalida dell' cookie identità.</span><span class="sxs-lookup"><span data-stu-id="aff71-181">The <xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationEvents.ValidatePrincipal*> event can be used to intercept and override validation of the cookie identity.</span></span> <span data-ttu-id="aff71-182">La convalida cookie di a ogni richiesta attenua il rischio di revoca degli utenti che accedono all'app.</span><span class="sxs-lookup"><span data-stu-id="aff71-182">Validating the cookie on every request mitigates the risk of revoked users accessing the app.</span></span>

<span data-ttu-id="aff71-183">Un approccio alla cookie convalida si basa sul tenere traccia del momento in cui il database utente viene modificato.</span><span class="sxs-lookup"><span data-stu-id="aff71-183">One approach to cookie validation is based on keeping track of when the user database changes.</span></span> <span data-ttu-id="aff71-184">Se il database non è stato modificato dopo l'emissione dell'utente cookie , non è necessario autenticare di nuovo l'utente se cookie è ancora valido.</span><span class="sxs-lookup"><span data-stu-id="aff71-184">If the database hasn't been changed since the user's cookie was issued, there's no need to re-authenticate the user if their cookie is still valid.</span></span> <span data-ttu-id="aff71-185">Nell'app di esempio, il database viene implementato in `IUserRepository` e archivia un `LastChanged` valore.</span><span class="sxs-lookup"><span data-stu-id="aff71-185">In the sample app, the database is implemented in `IUserRepository` and stores a `LastChanged` value.</span></span> <span data-ttu-id="aff71-186">Quando un utente viene aggiornato nel database, il `LastChanged` valore viene impostato sull'ora corrente.</span><span class="sxs-lookup"><span data-stu-id="aff71-186">When a user is updated in the database, the `LastChanged` value is set to the current time.</span></span>

<span data-ttu-id="aff71-187">Per invalidare un oggetto cookie quando il database viene modificato in base al `LastChanged` valore, creare cookie con un' `LastChanged` attestazione contenente il `LastChanged` valore corrente dal database:</span><span class="sxs-lookup"><span data-stu-id="aff71-187">In order to invalidate a cookie when the database changes based on the `LastChanged` value, create the cookie with a `LastChanged` claim containing the current `LastChanged` value from the database:</span></span>

```csharp
var claims = new List<Claim>
{
    new Claim(ClaimTypes.Name, user.Email),
    new Claim("LastChanged", {Database Value})
};

var claimsIdentity = new ClaimsIdentity(
    claims, 
    CookieAuthenticationDefaults.AuthenticationScheme);

await HttpContext.SignInAsync(
    CookieAuthenticationDefaults.AuthenticationScheme, 
    new ClaimsPrincipal(claimsIdentity));
```

<span data-ttu-id="aff71-188">Per implementare una sostituzione per l' `ValidatePrincipal` evento, scrivere un metodo con la firma seguente in una classe che deriva da <xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationEvents> :</span><span class="sxs-lookup"><span data-stu-id="aff71-188">To implement an override for the `ValidatePrincipal` event, write a method with the following signature in a class that derives from <xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationEvents>:</span></span>

```csharp
ValidatePrincipal(CookieValidatePrincipalContext)
```

<span data-ttu-id="aff71-189">Di seguito è riportato un esempio di implementazione di `CookieAuthenticationEvents` :</span><span class="sxs-lookup"><span data-stu-id="aff71-189">The following is an example implementation of `CookieAuthenticationEvents`:</span></span>

```csharp
using System.Linq;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Authentication;
using Microsoft.AspNetCore.Authentication.Cookies;

public class CustomCookieAuthenticationEvents : CookieAuthenticationEvents
{
    private readonly IUserRepository _userRepository;

    public CustomCookieAuthenticationEvents(IUserRepository userRepository)
    {
        // Get the database from registered DI services.
        _userRepository = userRepository;
    }

    public override async Task ValidatePrincipal(CookieValidatePrincipalContext context)
    {
        var userPrincipal = context.Principal;

        // Look for the LastChanged claim.
        var lastChanged = (from c in userPrincipal.Claims
                           where c.Type == "LastChanged"
                           select c.Value).FirstOrDefault();

        if (string.IsNullOrEmpty(lastChanged) ||
            !_userRepository.ValidateLastChanged(lastChanged))
        {
            context.RejectPrincipal();

            await context.HttpContext.SignOutAsync(
                CookieAuthenticationDefaults.AuthenticationScheme);
        }
    }
}
```

<span data-ttu-id="aff71-190">Registrare l'istanza degli eventi durante cookie la registrazione del servizio nel `Startup.ConfigureServices` metodo.</span><span class="sxs-lookup"><span data-stu-id="aff71-190">Register the events instance during cookie service registration in the `Startup.ConfigureServices` method.</span></span> <span data-ttu-id="aff71-191">Fornire una [registrazione del servizio con ambito](xref:fundamentals/dependency-injection#service-lifetimes) per la `CustomCookieAuthenticationEvents` classe:</span><span class="sxs-lookup"><span data-stu-id="aff71-191">Provide a [scoped service registration](xref:fundamentals/dependency-injection#service-lifetimes) for your `CustomCookieAuthenticationEvents` class:</span></span>

```csharp
services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(options =>
    {
        options.EventsType = typeof(CustomCookieAuthenticationEvents);
    });

services.AddScoped<CustomCookieAuthenticationEvents>();
```

<span data-ttu-id="aff71-192">Si consideri una situazione in cui il nome dell'utente viene aggiornato &mdash; una decisione che non influisce sulla sicurezza in alcun modo.</span><span class="sxs-lookup"><span data-stu-id="aff71-192">Consider a situation in which the user's name is updated&mdash;a decision that doesn't affect security in any way.</span></span> <span data-ttu-id="aff71-193">Se si desidera aggiornare l'entità utente in modo non distruttivo, chiamare `context.ReplacePrincipal` e impostare la `context.ShouldRenew` proprietà su `true` .</span><span class="sxs-lookup"><span data-stu-id="aff71-193">If you want to non-destructively update the user principal, call `context.ReplacePrincipal` and set the `context.ShouldRenew` property to `true`.</span></span>

> [!WARNING]
> <span data-ttu-id="aff71-194">L'approccio descritto qui viene attivato a ogni richiesta.</span><span class="sxs-lookup"><span data-stu-id="aff71-194">The approach described here is triggered on every request.</span></span> <span data-ttu-id="aff71-195">La convalida cookie dell'autenticazione per tutti gli utenti in ogni richiesta può comportare un notevole calo delle prestazioni per l'app.</span><span class="sxs-lookup"><span data-stu-id="aff71-195">Validating authentication cookies for all users on every request can result in a large performance penalty for the app.</span></span>

## <a name="persistent-cookies"></a><span data-ttu-id="aff71-196">S persistente cookie</span><span class="sxs-lookup"><span data-stu-id="aff71-196">Persistent cookies</span></span>

<span data-ttu-id="aff71-197">È possibile che il venga cookie mantenuto tra le sessioni del browser.</span><span class="sxs-lookup"><span data-stu-id="aff71-197">You may want the cookie to persist across browser sessions.</span></span> <span data-ttu-id="aff71-198">Questa persistenza deve essere abilitata solo con il consenso esplicito dell'utente con una casella di controllo "Remember me" all'accesso o un meccanismo simile.</span><span class="sxs-lookup"><span data-stu-id="aff71-198">This persistence should only be enabled with explicit user consent with a "Remember Me" check box on sign in or a similar mechanism.</span></span> 

<span data-ttu-id="aff71-199">Il frammento di codice seguente crea un'identità e corrispondente che rimane inalterata cookie attraverso le chiusure del browser.</span><span class="sxs-lookup"><span data-stu-id="aff71-199">The following code snippet creates an identity and corresponding cookie that survives through browser closures.</span></span> <span data-ttu-id="aff71-200">Vengono rispettate tutte le impostazioni di scadenza scorrevoli configurate in precedenza.</span><span class="sxs-lookup"><span data-stu-id="aff71-200">Any sliding expiration settings previously configured are honored.</span></span> <span data-ttu-id="aff71-201">Se l'oggetto cookie scade quando il browser viene chiuso, il browser cancella il oggetto cookie dopo il riavvio.</span><span class="sxs-lookup"><span data-stu-id="aff71-201">If the cookie expires while the browser is closed, the browser clears the cookie once it's restarted.</span></span>

<span data-ttu-id="aff71-202">Imposta <xref:Microsoft.AspNetCore.Authentication.AuthenticationProperties.IsPersistent> su `true` in <xref:Microsoft.AspNetCore.Authentication.AuthenticationProperties> :</span><span class="sxs-lookup"><span data-stu-id="aff71-202">Set <xref:Microsoft.AspNetCore.Authentication.AuthenticationProperties.IsPersistent> to `true` in <xref:Microsoft.AspNetCore.Authentication.AuthenticationProperties>:</span></span>

```csharp
// using Microsoft.AspNetCore.Authentication;

await HttpContext.SignInAsync(
    CookieAuthenticationDefaults.AuthenticationScheme,
    new ClaimsPrincipal(claimsIdentity),
    new AuthenticationProperties
    {
        IsPersistent = true
    });
```

## <a name="absolute-cookie-expiration"></a><span data-ttu-id="aff71-203">cookieScadenza assoluta</span><span class="sxs-lookup"><span data-stu-id="aff71-203">Absolute cookie expiration</span></span>

<span data-ttu-id="aff71-204">È possibile impostare un'ora di scadenza assoluta con <xref:Microsoft.AspNetCore.Authentication.AuthenticationProperties.ExpiresUtc> .</span><span class="sxs-lookup"><span data-stu-id="aff71-204">An absolute expiration time can be set with <xref:Microsoft.AspNetCore.Authentication.AuthenticationProperties.ExpiresUtc>.</span></span> <span data-ttu-id="aff71-205">Per creare un oggetto persistente cookie , `IsPersistent` è necessario impostare anche.</span><span class="sxs-lookup"><span data-stu-id="aff71-205">To create a persistent cookie, `IsPersistent` must also be set.</span></span> <span data-ttu-id="aff71-206">In caso contrario, cookie viene creato con una durata basata sulla sessione e potrebbe scadere prima o dopo il ticket di autenticazione che possiede.</span><span class="sxs-lookup"><span data-stu-id="aff71-206">Otherwise, the cookie is created with a session-based lifetime and could expire either before or after the authentication ticket that it holds.</span></span> <span data-ttu-id="aff71-207">Quando `ExpiresUtc` è impostato, esegue l'override del valore dell' <xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationOptions.ExpireTimeSpan> opzione di <xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationOptions> , se impostata.</span><span class="sxs-lookup"><span data-stu-id="aff71-207">When `ExpiresUtc` is set, it overrides the value of the <xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationOptions.ExpireTimeSpan> option of <xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationOptions>, if set.</span></span>

<span data-ttu-id="aff71-208">Il frammento di codice seguente crea un'identità e corrisponde alla cookie durata di 20 minuti.</span><span class="sxs-lookup"><span data-stu-id="aff71-208">The following code snippet creates an identity and corresponding cookie that lasts for 20 minutes.</span></span> <span data-ttu-id="aff71-209">Questa operazione ignora le impostazioni di scadenza scorrevoli configurate in precedenza.</span><span class="sxs-lookup"><span data-stu-id="aff71-209">This ignores any sliding expiration settings previously configured.</span></span>

```csharp
// using Microsoft.AspNetCore.Authentication;

await HttpContext.SignInAsync(
    CookieAuthenticationDefaults.AuthenticationScheme,
    new ClaimsPrincipal(claimsIdentity),
    new AuthenticationProperties
    {
        IsPersistent = true,
        ExpiresUtc = DateTime.UtcNow.AddMinutes(20)
    });
```

::: moniker-end

::: moniker range="< aspnetcore-3.0"

<span data-ttu-id="aff71-210">ASP.NET Core Identity è un provider di autenticazione completo e completo per la creazione e la gestione degli account di accesso.</span><span class="sxs-lookup"><span data-stu-id="aff71-210">ASP.NET Core Identity is a complete, full-featured authentication provider for creating and maintaining logins.</span></span> <span data-ttu-id="aff71-211">Tuttavia, un cookie provider di autenticazione basato su senza ASP.NET Core Identity può essere utilizzato.</span><span class="sxs-lookup"><span data-stu-id="aff71-211">However, a cookie-based authentication provider without ASP.NET Core Identity can be used.</span></span> <span data-ttu-id="aff71-212">Per altre informazioni, vedere <xref:security/authentication/identity>.</span><span class="sxs-lookup"><span data-stu-id="aff71-212">For more information, see <xref:security/authentication/identity>.</span></span>

<span data-ttu-id="aff71-213">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/authentication/cookie/samples) ([procedura per il download](xref:index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="aff71-213">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/authentication/cookie/samples) ([how to download](xref:index#how-to-download-a-sample))</span></span>

<span data-ttu-id="aff71-214">A scopo dimostrativo nell'app di esempio, l'account utente per l'utente ipotetico, Maria Rodriguez, è hardcoded nell'app.</span><span class="sxs-lookup"><span data-stu-id="aff71-214">For demonstration purposes in the sample app, the user account for the hypothetical user, Maria Rodriguez, is hardcoded into the app.</span></span> <span data-ttu-id="aff71-215">Usare l'indirizzo di **posta elettronica** `maria.rodriguez@contoso.com` e qualsiasi password per accedere all'utente.</span><span class="sxs-lookup"><span data-stu-id="aff71-215">Use the **Email** address `maria.rodriguez@contoso.com` and any password to sign in the user.</span></span> <span data-ttu-id="aff71-216">L'utente viene autenticato nel `AuthenticateUser` metodo nel file *pages/account/login. cshtml. cs* .</span><span class="sxs-lookup"><span data-stu-id="aff71-216">The user is authenticated in the `AuthenticateUser` method in the *Pages/Account/Login.cshtml.cs* file.</span></span> <span data-ttu-id="aff71-217">In un esempio reale, l'utente viene autenticato a fronte di un database.</span><span class="sxs-lookup"><span data-stu-id="aff71-217">In a real-world example, the user would be authenticated against a database.</span></span>

## <a name="configuration"></a><span data-ttu-id="aff71-218">Configurazione</span><span class="sxs-lookup"><span data-stu-id="aff71-218">Configuration</span></span>

<span data-ttu-id="aff71-219">Se l'app non usa il [metapacchetto Microsoft. AspNetCore. app](xref:fundamentals/metapackage-app), creare un riferimento al pacchetto nel file di progetto per [Microsoft. AspNetCore. Authentication. Cookie pacchetto s](https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.Cookies/) .</span><span class="sxs-lookup"><span data-stu-id="aff71-219">If the app doesn't use the [Microsoft.AspNetCore.App metapackage](xref:fundamentals/metapackage-app), create a package reference in the project file for the [Microsoft.AspNetCore.Authentication.Cookies](https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.Cookies/) package.</span></span>

<span data-ttu-id="aff71-220">Nel `Startup.ConfigureServices` metodo creare il servizio middleware di autenticazione con i <xref:Microsoft.Extensions.DependencyInjection.AuthenticationServiceCollectionExtensions.AddAuthentication*> metodi e <xref:Microsoft.Extensions.DependencyInjection.CookieExtensions.AddCookie*> :</span><span class="sxs-lookup"><span data-stu-id="aff71-220">In the `Startup.ConfigureServices` method, create the Authentication Middleware service with the <xref:Microsoft.Extensions.DependencyInjection.AuthenticationServiceCollectionExtensions.AddAuthentication*> and <xref:Microsoft.Extensions.DependencyInjection.CookieExtensions.AddCookie*> methods:</span></span>

[!code-csharp[](cookie/samples/2.x/CookieSample/Startup.cs?name=snippet1)]

<span data-ttu-id="aff71-221"><xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationDefaults.AuthenticationScheme> passato a `AddAuthentication` imposta lo schema di autenticazione predefinito per l'app.</span><span class="sxs-lookup"><span data-stu-id="aff71-221"><xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationDefaults.AuthenticationScheme> passed to `AddAuthentication` sets the default authentication scheme for the app.</span></span> <span data-ttu-id="aff71-222">`AuthenticationScheme` è utile quando sono presenti più istanze di cookie autenticazione e si desidera [autorizzare con uno schema specifico](xref:security/authorization/limitingidentitybyscheme).</span><span class="sxs-lookup"><span data-stu-id="aff71-222">`AuthenticationScheme` is useful when there are multiple instances of cookie authentication and you want to [authorize with a specific scheme](xref:security/authorization/limitingidentitybyscheme).</span></span> <span data-ttu-id="aff71-223">L'impostazione `AuthenticationScheme` di su [ Cookie AuthenticationDefaults. AuthenticationScheme](xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationDefaults.AuthenticationScheme) fornisce il valore " Cookie s" per lo schema.</span><span class="sxs-lookup"><span data-stu-id="aff71-223">Setting the `AuthenticationScheme` to [CookieAuthenticationDefaults.AuthenticationScheme](xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationDefaults.AuthenticationScheme) provides a value of "Cookies" for the scheme.</span></span> <span data-ttu-id="aff71-224">È possibile specificare qualsiasi valore stringa che distingua lo schema.</span><span class="sxs-lookup"><span data-stu-id="aff71-224">You can supply any string value that distinguishes the scheme.</span></span>

<span data-ttu-id="aff71-225">Lo schema di autenticazione dell'app è diverso dallo schema di cookie autenticazione dell'app.</span><span class="sxs-lookup"><span data-stu-id="aff71-225">The app's authentication scheme is different from the app's cookie authentication scheme.</span></span> <span data-ttu-id="aff71-226">Quando uno cookie schema di autenticazione non viene fornito a <xref:Microsoft.Extensions.DependencyInjection.CookieExtensions.AddCookie*> , USA `CookieAuthenticationDefaults.AuthenticationScheme` (" Cookie s").</span><span class="sxs-lookup"><span data-stu-id="aff71-226">When a cookie authentication scheme isn't provided to <xref:Microsoft.Extensions.DependencyInjection.CookieExtensions.AddCookie*>, it uses `CookieAuthenticationDefaults.AuthenticationScheme` ("Cookies").</span></span>

<span data-ttu-id="aff71-227">La cookie proprietà Authentication <xref:Microsoft.AspNetCore.Http.CookieBuilder.IsEssential> è impostata su per `true` impostazione predefinita.</span><span class="sxs-lookup"><span data-stu-id="aff71-227">The authentication cookie's <xref:Microsoft.AspNetCore.Http.CookieBuilder.IsEssential> property is set to `true` by default.</span></span> <span data-ttu-id="aff71-228">cookieSono consentite le autenticazioni quando un visitatore del sito non ha acconsentito alla raccolta dei dati.</span><span class="sxs-lookup"><span data-stu-id="aff71-228">Authentication cookies are allowed when a site visitor hasn't consented to data collection.</span></span> <span data-ttu-id="aff71-229">Per altre informazioni, vedere <xref:security/gdpr#essential-cookies>.</span><span class="sxs-lookup"><span data-stu-id="aff71-229">For more information, see <xref:security/gdpr#essential-cookies>.</span></span>

<span data-ttu-id="aff71-230">Nel `Startup.Configure` metodo chiamare il `UseAuthentication` metodo per richiamare il middleware di autenticazione che imposta la `HttpContext.User` Proprietà.</span><span class="sxs-lookup"><span data-stu-id="aff71-230">In the `Startup.Configure` method, call the `UseAuthentication` method to invoke the Authentication Middleware that sets the `HttpContext.User` property.</span></span> <span data-ttu-id="aff71-231">Chiamare il `UseAuthentication` metodo prima di chiamare `UseMvcWithDefaultRoute` o `UseMvc` :</span><span class="sxs-lookup"><span data-stu-id="aff71-231">Call the `UseAuthentication` method before calling `UseMvcWithDefaultRoute` or `UseMvc`:</span></span>

[!code-csharp[](cookie/samples/2.x/CookieSample/Startup.cs?name=snippet2)]

<span data-ttu-id="aff71-232">La <xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationOptions> classe viene utilizzata per configurare le opzioni del provider di autenticazione.</span><span class="sxs-lookup"><span data-stu-id="aff71-232">The <xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationOptions> class is used to configure the authentication provider options.</span></span>

<span data-ttu-id="aff71-233">Impostare `CookieAuthenticationOptions` nella configurazione del servizio per l'autenticazione nel `Startup.ConfigureServices` Metodo:</span><span class="sxs-lookup"><span data-stu-id="aff71-233">Set `CookieAuthenticationOptions` in the service configuration for authentication in the `Startup.ConfigureServices` method:</span></span>

```csharp
services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(options =>
    {
        ...
    });
```

## <a name="cookie-policy-middleware"></a><span data-ttu-id="aff71-234">Cookie Middleware dei criteri</span><span class="sxs-lookup"><span data-stu-id="aff71-234">Cookie Policy Middleware</span></span>

<span data-ttu-id="aff71-235">Il [ Cookie middleware dei criteri](xref:Microsoft.AspNetCore.CookiePolicy.CookiePolicyMiddleware) Abilita le cookie funzionalità dei criteri.</span><span class="sxs-lookup"><span data-stu-id="aff71-235">[Cookie Policy Middleware](xref:Microsoft.AspNetCore.CookiePolicy.CookiePolicyMiddleware) enables cookie policy capabilities.</span></span> <span data-ttu-id="aff71-236">L'aggiunta del middleware alla pipeline di elaborazione delle app è sensibile agli ordini &mdash; , ma interessa solo i componenti downstream registrati nella pipeline.</span><span class="sxs-lookup"><span data-stu-id="aff71-236">Adding the middleware to the app processing pipeline is order sensitive&mdash;it only affects downstream components registered in the pipeline.</span></span>

```csharp
app.UseCookiePolicy(cookiePolicyOptions);
```

<span data-ttu-id="aff71-237">Utilizzare <xref:Microsoft.AspNetCore.Builder.CookiePolicyOptions> fornito al Cookie middleware dei criteri per controllare le caratteristiche globali dell' cookie elaborazione e agganciare i cookie gestori di elaborazione quando cookie vengono aggiunti o eliminati.</span><span class="sxs-lookup"><span data-stu-id="aff71-237">Use <xref:Microsoft.AspNetCore.Builder.CookiePolicyOptions> provided to the Cookie Policy Middleware to control global characteristics of cookie processing and hook into cookie processing handlers when cookies are appended or deleted.</span></span>

<span data-ttu-id="aff71-238">Il <xref:Microsoft.AspNetCore.Builder.CookiePolicyOptions.MinimumSameSitePolicy> valore predefinito è `SameSiteMode.Lax` consentire l'autenticazione OAuth2.</span><span class="sxs-lookup"><span data-stu-id="aff71-238">The default <xref:Microsoft.AspNetCore.Builder.CookiePolicyOptions.MinimumSameSitePolicy> value is `SameSiteMode.Lax` to permit OAuth2 authentication.</span></span> <span data-ttu-id="aff71-239">Per applicare rigorosamente un criterio dello stesso sito di `SameSiteMode.Strict` , impostare il `MinimumSameSitePolicy` .</span><span class="sxs-lookup"><span data-stu-id="aff71-239">To strictly enforce a same-site policy of `SameSiteMode.Strict`, set the `MinimumSameSitePolicy`.</span></span> <span data-ttu-id="aff71-240">Sebbene questa impostazione interrompa OAuth2 e altri schemi di autenticazione tra le origini, eleva il livello di cookie sicurezza per altri tipi di app che non si basano sull'elaborazione di richieste tra origini.</span><span class="sxs-lookup"><span data-stu-id="aff71-240">Although this setting breaks OAuth2 and other cross-origin authentication schemes, it elevates the level of cookie security for other types of apps that don't rely on cross-origin request processing.</span></span>

```csharp
var cookiePolicyOptions = new CookiePolicyOptions
{
    MinimumSameSitePolicy = SameSiteMode.Strict,
};
```

<span data-ttu-id="aff71-241">L' Cookie impostazione del middleware per i criteri di `MinimumSameSitePolicy` può influire sull'impostazione di `Cookie.SameSite` in `CookieAuthenticationOptions` impostazioni in base alla matrice riportata di seguito.</span><span class="sxs-lookup"><span data-stu-id="aff71-241">The Cookie Policy Middleware setting for `MinimumSameSitePolicy` can affect the setting of `Cookie.SameSite` in `CookieAuthenticationOptions` settings according to the matrix below.</span></span>

| <span data-ttu-id="aff71-242">MinimumSameSitePolicy</span><span class="sxs-lookup"><span data-stu-id="aff71-242">MinimumSameSitePolicy</span></span> | <span data-ttu-id="aff71-243">Cookie. Navigava sullostesso sito</span><span class="sxs-lookup"><span data-stu-id="aff71-243">Cookie.SameSite</span></span> | <span data-ttu-id="aff71-244">Risultato Cookie . Impostazione navigava sullostesso sito</span><span class="sxs-lookup"><span data-stu-id="aff71-244">Resultant Cookie.SameSite setting</span></span> |
| --------------------- | --------------- | --------------------------------- |
| <span data-ttu-id="aff71-245">SameSiteMode. None</span><span class="sxs-lookup"><span data-stu-id="aff71-245">SameSiteMode.None</span></span>     | <span data-ttu-id="aff71-246">SameSiteMode. None</span><span class="sxs-lookup"><span data-stu-id="aff71-246">SameSiteMode.None</span></span><br><span data-ttu-id="aff71-247">SameSiteMode. LAX</span><span class="sxs-lookup"><span data-stu-id="aff71-247">SameSiteMode.Lax</span></span><br><span data-ttu-id="aff71-248">SameSiteMode. Strict</span><span class="sxs-lookup"><span data-stu-id="aff71-248">SameSiteMode.Strict</span></span> | <span data-ttu-id="aff71-249">SameSiteMode. None</span><span class="sxs-lookup"><span data-stu-id="aff71-249">SameSiteMode.None</span></span><br><span data-ttu-id="aff71-250">SameSiteMode. LAX</span><span class="sxs-lookup"><span data-stu-id="aff71-250">SameSiteMode.Lax</span></span><br><span data-ttu-id="aff71-251">SameSiteMode. Strict</span><span class="sxs-lookup"><span data-stu-id="aff71-251">SameSiteMode.Strict</span></span> |
| <span data-ttu-id="aff71-252">SameSiteMode. LAX</span><span class="sxs-lookup"><span data-stu-id="aff71-252">SameSiteMode.Lax</span></span>      | <span data-ttu-id="aff71-253">SameSiteMode. None</span><span class="sxs-lookup"><span data-stu-id="aff71-253">SameSiteMode.None</span></span><br><span data-ttu-id="aff71-254">SameSiteMode. LAX</span><span class="sxs-lookup"><span data-stu-id="aff71-254">SameSiteMode.Lax</span></span><br><span data-ttu-id="aff71-255">SameSiteMode. Strict</span><span class="sxs-lookup"><span data-stu-id="aff71-255">SameSiteMode.Strict</span></span> | <span data-ttu-id="aff71-256">SameSiteMode. LAX</span><span class="sxs-lookup"><span data-stu-id="aff71-256">SameSiteMode.Lax</span></span><br><span data-ttu-id="aff71-257">SameSiteMode. LAX</span><span class="sxs-lookup"><span data-stu-id="aff71-257">SameSiteMode.Lax</span></span><br><span data-ttu-id="aff71-258">SameSiteMode. Strict</span><span class="sxs-lookup"><span data-stu-id="aff71-258">SameSiteMode.Strict</span></span> |
| <span data-ttu-id="aff71-259">SameSiteMode. Strict</span><span class="sxs-lookup"><span data-stu-id="aff71-259">SameSiteMode.Strict</span></span>   | <span data-ttu-id="aff71-260">SameSiteMode. None</span><span class="sxs-lookup"><span data-stu-id="aff71-260">SameSiteMode.None</span></span><br><span data-ttu-id="aff71-261">SameSiteMode. LAX</span><span class="sxs-lookup"><span data-stu-id="aff71-261">SameSiteMode.Lax</span></span><br><span data-ttu-id="aff71-262">SameSiteMode. Strict</span><span class="sxs-lookup"><span data-stu-id="aff71-262">SameSiteMode.Strict</span></span> | <span data-ttu-id="aff71-263">SameSiteMode. Strict</span><span class="sxs-lookup"><span data-stu-id="aff71-263">SameSiteMode.Strict</span></span><br><span data-ttu-id="aff71-264">SameSiteMode. Strict</span><span class="sxs-lookup"><span data-stu-id="aff71-264">SameSiteMode.Strict</span></span><br><span data-ttu-id="aff71-265">SameSiteMode. Strict</span><span class="sxs-lookup"><span data-stu-id="aff71-265">SameSiteMode.Strict</span></span> |

## <a name="create-an-authentication-cookie"></a><span data-ttu-id="aff71-266">Creare un'autenticazione cookie</span><span class="sxs-lookup"><span data-stu-id="aff71-266">Create an authentication cookie</span></span>

<span data-ttu-id="aff71-267">Per creare un oggetto che cookie contiene informazioni sull'utente, costruire un oggetto <xref:System.Security.Claims.ClaimsPrincipal> .</span><span class="sxs-lookup"><span data-stu-id="aff71-267">To create a cookie holding user information, construct a <xref:System.Security.Claims.ClaimsPrincipal>.</span></span> <span data-ttu-id="aff71-268">Le informazioni utente vengono serializzate e archiviate in cookie .</span><span class="sxs-lookup"><span data-stu-id="aff71-268">The user information is serialized and stored in the cookie.</span></span> 

<span data-ttu-id="aff71-269">Creare una <xref:System.Security.Claims.ClaimsIdentity> con qualsiasi richiesta <xref:System.Security.Claims.Claim> e chiamare <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.SignInAsync*> per accedere all'utente:</span><span class="sxs-lookup"><span data-stu-id="aff71-269">Create a <xref:System.Security.Claims.ClaimsIdentity> with any required <xref:System.Security.Claims.Claim>s and call <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.SignInAsync*> to sign in the user:</span></span>

[!code-csharp[](cookie/samples/2.x/CookieSample/Pages/Account/Login.cshtml.cs?name=snippet1)]

<span data-ttu-id="aff71-270">`SignInAsync` Crea un oggetto crittografato cookie e lo aggiunge alla risposta corrente.</span><span class="sxs-lookup"><span data-stu-id="aff71-270">`SignInAsync` creates an encrypted cookie and adds it to the current response.</span></span> <span data-ttu-id="aff71-271">Se `AuthenticationScheme` non è specificato, viene usato lo schema predefinito.</span><span class="sxs-lookup"><span data-stu-id="aff71-271">If `AuthenticationScheme` isn't specified, the default scheme is used.</span></span>

<span data-ttu-id="aff71-272">Per la crittografia viene usato il sistema di [protezione dei dati](xref:security/data-protection/using-data-protection) di ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="aff71-272">ASP.NET Core's [Data Protection](xref:security/data-protection/using-data-protection) system is used for encryption.</span></span> <span data-ttu-id="aff71-273">Per un'app ospitata in più computer, bilanciamento del carico tra app o uso di un Web farm, [configurare la protezione dei dati](xref:security/data-protection/configuration/overview) in modo da usare lo stesso anello chiave e l'identificatore dell'app.</span><span class="sxs-lookup"><span data-stu-id="aff71-273">For an app hosted on multiple machines, load balancing across apps, or using a web farm, [configure data protection](xref:security/data-protection/configuration/overview) to use the same key ring and app identifier.</span></span>

## <a name="sign-out"></a><span data-ttu-id="aff71-274">Disconnetti</span><span class="sxs-lookup"><span data-stu-id="aff71-274">Sign out</span></span>

<span data-ttu-id="aff71-275">Per disconnettersi dall'utente corrente ed eliminare il relativo cookie , chiamare <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.SignOutAsync*> :</span><span class="sxs-lookup"><span data-stu-id="aff71-275">To sign out the current user and delete their cookie, call <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.SignOutAsync*>:</span></span>

[!code-csharp[](cookie/samples/2.x/CookieSample/Pages/Account/Login.cshtml.cs?name=snippet2)]

<span data-ttu-id="aff71-276">Se `CookieAuthenticationDefaults.AuthenticationScheme` (o " Cookie s") non viene usato come schema (ad esempio, "Contoso Cookie "), fornire lo schema usato durante la configurazione del provider di autenticazione.</span><span class="sxs-lookup"><span data-stu-id="aff71-276">If `CookieAuthenticationDefaults.AuthenticationScheme` (or "Cookies") isn't used as the scheme (for example, "ContosoCookie"), supply the scheme used when configuring the authentication provider.</span></span> <span data-ttu-id="aff71-277">In caso contrario, verrà utilizzato lo schema predefinito.</span><span class="sxs-lookup"><span data-stu-id="aff71-277">Otherwise, the default scheme is used.</span></span>

## <a name="react-to-back-end-changes"></a><span data-ttu-id="aff71-278">Reagire alle modifiche del back-end</span><span class="sxs-lookup"><span data-stu-id="aff71-278">React to back-end changes</span></span>

<span data-ttu-id="aff71-279">Una volta cookie creato un oggetto, cookie è l'unica origine di identità.</span><span class="sxs-lookup"><span data-stu-id="aff71-279">Once a cookie is created, the cookie is the single source of identity.</span></span> <span data-ttu-id="aff71-280">Se un account utente è disabilitato nei sistemi back-end:</span><span class="sxs-lookup"><span data-stu-id="aff71-280">If a user account is disabled in back-end systems:</span></span>

* <span data-ttu-id="aff71-281">Il sistema di cookie autenticazione dell'app continua a elaborare le richieste in base all'autenticazione cookie .</span><span class="sxs-lookup"><span data-stu-id="aff71-281">The app's cookie authentication system continues to process requests based on the authentication cookie.</span></span>
* <span data-ttu-id="aff71-282">L'utente rimane connesso all'app fino a quando l'autenticazione cookie è valida.</span><span class="sxs-lookup"><span data-stu-id="aff71-282">The user remains signed into the app as long as the authentication cookie is valid.</span></span>

<span data-ttu-id="aff71-283">L' <xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationEvents.ValidatePrincipal*> evento può essere usato per intercettare ed eseguire l'override della convalida dell' cookie identità.</span><span class="sxs-lookup"><span data-stu-id="aff71-283">The <xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationEvents.ValidatePrincipal*> event can be used to intercept and override validation of the cookie identity.</span></span> <span data-ttu-id="aff71-284">La convalida cookie di a ogni richiesta attenua il rischio di revoca degli utenti che accedono all'app.</span><span class="sxs-lookup"><span data-stu-id="aff71-284">Validating the cookie on every request mitigates the risk of revoked users accessing the app.</span></span>

<span data-ttu-id="aff71-285">Un approccio alla cookie convalida si basa sul tenere traccia del momento in cui il database utente viene modificato.</span><span class="sxs-lookup"><span data-stu-id="aff71-285">One approach to cookie validation is based on keeping track of when the user database changes.</span></span> <span data-ttu-id="aff71-286">Se il database non è stato modificato dopo l'emissione dell'utente cookie , non è necessario autenticare di nuovo l'utente se cookie è ancora valido.</span><span class="sxs-lookup"><span data-stu-id="aff71-286">If the database hasn't been changed since the user's cookie was issued, there's no need to re-authenticate the user if their cookie is still valid.</span></span> <span data-ttu-id="aff71-287">Nell'app di esempio, il database viene implementato in `IUserRepository` e archivia un `LastChanged` valore.</span><span class="sxs-lookup"><span data-stu-id="aff71-287">In the sample app, the database is implemented in `IUserRepository` and stores a `LastChanged` value.</span></span> <span data-ttu-id="aff71-288">Quando un utente viene aggiornato nel database, il `LastChanged` valore viene impostato sull'ora corrente.</span><span class="sxs-lookup"><span data-stu-id="aff71-288">When a user is updated in the database, the `LastChanged` value is set to the current time.</span></span>

<span data-ttu-id="aff71-289">Per invalidare un oggetto cookie quando il database viene modificato in base al `LastChanged` valore, creare cookie con un' `LastChanged` attestazione contenente il `LastChanged` valore corrente dal database:</span><span class="sxs-lookup"><span data-stu-id="aff71-289">In order to invalidate a cookie when the database changes based on the `LastChanged` value, create the cookie with a `LastChanged` claim containing the current `LastChanged` value from the database:</span></span>

```csharp
var claims = new List<Claim>
{
    new Claim(ClaimTypes.Name, user.Email),
    new Claim("LastChanged", {Database Value})
};

var claimsIdentity = new ClaimsIdentity(
    claims, 
    CookieAuthenticationDefaults.AuthenticationScheme);

await HttpContext.SignInAsync(
    CookieAuthenticationDefaults.AuthenticationScheme, 
    new ClaimsPrincipal(claimsIdentity));
```

<span data-ttu-id="aff71-290">Per implementare una sostituzione per l' `ValidatePrincipal` evento, scrivere un metodo con la firma seguente in una classe che deriva da <xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationEvents> :</span><span class="sxs-lookup"><span data-stu-id="aff71-290">To implement an override for the `ValidatePrincipal` event, write a method with the following signature in a class that derives from <xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationEvents>:</span></span>

```csharp
ValidatePrincipal(CookieValidatePrincipalContext)
```

<span data-ttu-id="aff71-291">Di seguito è riportato un esempio di implementazione di `CookieAuthenticationEvents` :</span><span class="sxs-lookup"><span data-stu-id="aff71-291">The following is an example implementation of `CookieAuthenticationEvents`:</span></span>

```csharp
using System.Linq;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Authentication;
using Microsoft.AspNetCore.Authentication.Cookies;

public class CustomCookieAuthenticationEvents : CookieAuthenticationEvents
{
    private readonly IUserRepository _userRepository;

    public CustomCookieAuthenticationEvents(IUserRepository userRepository)
    {
        // Get the database from registered DI services.
        _userRepository = userRepository;
    }

    public override async Task ValidatePrincipal(CookieValidatePrincipalContext context)
    {
        var userPrincipal = context.Principal;

        // Look for the LastChanged claim.
        var lastChanged = (from c in userPrincipal.Claims
                           where c.Type == "LastChanged"
                           select c.Value).FirstOrDefault();

        if (string.IsNullOrEmpty(lastChanged) ||
            !_userRepository.ValidateLastChanged(lastChanged))
        {
            context.RejectPrincipal();

            await context.HttpContext.SignOutAsync(
                CookieAuthenticationDefaults.AuthenticationScheme);
        }
    }
}
```

<span data-ttu-id="aff71-292">Registrare l'istanza degli eventi durante cookie la registrazione del servizio nel `Startup.ConfigureServices` metodo.</span><span class="sxs-lookup"><span data-stu-id="aff71-292">Register the events instance during cookie service registration in the `Startup.ConfigureServices` method.</span></span> <span data-ttu-id="aff71-293">Fornire una [registrazione del servizio con ambito](xref:fundamentals/dependency-injection#service-lifetimes) per la `CustomCookieAuthenticationEvents` classe:</span><span class="sxs-lookup"><span data-stu-id="aff71-293">Provide a [scoped service registration](xref:fundamentals/dependency-injection#service-lifetimes) for your `CustomCookieAuthenticationEvents` class:</span></span>

```csharp
services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(options =>
    {
        options.EventsType = typeof(CustomCookieAuthenticationEvents);
    });

services.AddScoped<CustomCookieAuthenticationEvents>();
```

<span data-ttu-id="aff71-294">Si consideri una situazione in cui il nome dell'utente viene aggiornato &mdash; una decisione che non influisce sulla sicurezza in alcun modo.</span><span class="sxs-lookup"><span data-stu-id="aff71-294">Consider a situation in which the user's name is updated&mdash;a decision that doesn't affect security in any way.</span></span> <span data-ttu-id="aff71-295">Se si desidera aggiornare l'entità utente in modo non distruttivo, chiamare `context.ReplacePrincipal` e impostare la `context.ShouldRenew` proprietà su `true` .</span><span class="sxs-lookup"><span data-stu-id="aff71-295">If you want to non-destructively update the user principal, call `context.ReplacePrincipal` and set the `context.ShouldRenew` property to `true`.</span></span>

> [!WARNING]
> <span data-ttu-id="aff71-296">L'approccio descritto qui viene attivato a ogni richiesta.</span><span class="sxs-lookup"><span data-stu-id="aff71-296">The approach described here is triggered on every request.</span></span> <span data-ttu-id="aff71-297">La convalida cookie dell'autenticazione per tutti gli utenti in ogni richiesta può comportare un notevole calo delle prestazioni per l'app.</span><span class="sxs-lookup"><span data-stu-id="aff71-297">Validating authentication cookies for all users on every request can result in a large performance penalty for the app.</span></span>

## <a name="persistent-cookies"></a><span data-ttu-id="aff71-298">S persistente cookie</span><span class="sxs-lookup"><span data-stu-id="aff71-298">Persistent cookies</span></span>

<span data-ttu-id="aff71-299">È possibile che il venga cookie mantenuto tra le sessioni del browser.</span><span class="sxs-lookup"><span data-stu-id="aff71-299">You may want the cookie to persist across browser sessions.</span></span> <span data-ttu-id="aff71-300">Questa persistenza deve essere abilitata solo con il consenso esplicito dell'utente con una casella di controllo "Remember me" all'accesso o un meccanismo simile.</span><span class="sxs-lookup"><span data-stu-id="aff71-300">This persistence should only be enabled with explicit user consent with a "Remember Me" check box on sign in or a similar mechanism.</span></span> 

<span data-ttu-id="aff71-301">Il frammento di codice seguente crea un'identità e corrispondente che rimane inalterata cookie attraverso le chiusure del browser.</span><span class="sxs-lookup"><span data-stu-id="aff71-301">The following code snippet creates an identity and corresponding cookie that survives through browser closures.</span></span> <span data-ttu-id="aff71-302">Vengono rispettate tutte le impostazioni di scadenza scorrevoli configurate in precedenza.</span><span class="sxs-lookup"><span data-stu-id="aff71-302">Any sliding expiration settings previously configured are honored.</span></span> <span data-ttu-id="aff71-303">Se l'oggetto cookie scade quando il browser viene chiuso, il browser cancella il oggetto cookie dopo il riavvio.</span><span class="sxs-lookup"><span data-stu-id="aff71-303">If the cookie expires while the browser is closed, the browser clears the cookie once it's restarted.</span></span>

<span data-ttu-id="aff71-304">Imposta <xref:Microsoft.AspNetCore.Authentication.AuthenticationProperties.IsPersistent> su `true` in <xref:Microsoft.AspNetCore.Authentication.AuthenticationProperties> :</span><span class="sxs-lookup"><span data-stu-id="aff71-304">Set <xref:Microsoft.AspNetCore.Authentication.AuthenticationProperties.IsPersistent> to `true` in <xref:Microsoft.AspNetCore.Authentication.AuthenticationProperties>:</span></span>

```csharp
// using Microsoft.AspNetCore.Authentication;

await HttpContext.SignInAsync(
    CookieAuthenticationDefaults.AuthenticationScheme,
    new ClaimsPrincipal(claimsIdentity),
    new AuthenticationProperties
    {
        IsPersistent = true
    });
```

## <a name="absolute-cookie-expiration"></a><span data-ttu-id="aff71-305">cookieScadenza assoluta</span><span class="sxs-lookup"><span data-stu-id="aff71-305">Absolute cookie expiration</span></span>

<span data-ttu-id="aff71-306">È possibile impostare un'ora di scadenza assoluta con <xref:Microsoft.AspNetCore.Authentication.AuthenticationProperties.ExpiresUtc> .</span><span class="sxs-lookup"><span data-stu-id="aff71-306">An absolute expiration time can be set with <xref:Microsoft.AspNetCore.Authentication.AuthenticationProperties.ExpiresUtc>.</span></span> <span data-ttu-id="aff71-307">Per creare un oggetto persistente cookie , `IsPersistent` è necessario impostare anche.</span><span class="sxs-lookup"><span data-stu-id="aff71-307">To create a persistent cookie, `IsPersistent` must also be set.</span></span> <span data-ttu-id="aff71-308">In caso contrario, cookie viene creato con una durata basata sulla sessione e potrebbe scadere prima o dopo il ticket di autenticazione che possiede.</span><span class="sxs-lookup"><span data-stu-id="aff71-308">Otherwise, the cookie is created with a session-based lifetime and could expire either before or after the authentication ticket that it holds.</span></span> <span data-ttu-id="aff71-309">Quando `ExpiresUtc` è impostato, esegue l'override del valore dell' <xref:Microsoft.AspNetCore.Builder.CookieAuthenticationOptions.ExpireTimeSpan> opzione di <xref:Microsoft.AspNetCore.Builder.CookieAuthenticationOptions> , se impostata.</span><span class="sxs-lookup"><span data-stu-id="aff71-309">When `ExpiresUtc` is set, it overrides the value of the <xref:Microsoft.AspNetCore.Builder.CookieAuthenticationOptions.ExpireTimeSpan> option of <xref:Microsoft.AspNetCore.Builder.CookieAuthenticationOptions>, if set.</span></span>

<span data-ttu-id="aff71-310">Il frammento di codice seguente crea un'identità e corrisponde alla cookie durata di 20 minuti.</span><span class="sxs-lookup"><span data-stu-id="aff71-310">The following code snippet creates an identity and corresponding cookie that lasts for 20 minutes.</span></span> <span data-ttu-id="aff71-311">Questa operazione ignora le impostazioni di scadenza scorrevoli configurate in precedenza.</span><span class="sxs-lookup"><span data-stu-id="aff71-311">This ignores any sliding expiration settings previously configured.</span></span>

```csharp
// using Microsoft.AspNetCore.Authentication;

await HttpContext.SignInAsync(
    CookieAuthenticationDefaults.AuthenticationScheme,
    new ClaimsPrincipal(claimsIdentity),
    new AuthenticationProperties
    {
        IsPersistent = true,
        ExpiresUtc = DateTime.UtcNow.AddMinutes(20)
    });
```

::: moniker-end

## <a name="additional-resources"></a><span data-ttu-id="aff71-312">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="aff71-312">Additional resources</span></span>

* <xref:security/authorization/limitingidentitybyscheme>
* <xref:security/authorization/claims>
* [<span data-ttu-id="aff71-313">Controlli dei ruoli basati su criteri</span><span class="sxs-lookup"><span data-stu-id="aff71-313">Policy-based role checks</span></span>](xref:security/authorization/roles#policy-based-role-checks)
* <xref:host-and-deploy/web-farm>
