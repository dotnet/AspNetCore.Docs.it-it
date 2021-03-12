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
# <a name="use-cookie-authentication-without-aspnet-core-identity"></a>USA cookie autenticazione senza ASP.NET Core Identity

Autore: [Rick Anderson](https://twitter.com/RickAndMSFT)

::: moniker range=">= aspnetcore-3.0"

ASP.NET Core Identity è un provider di autenticazione completo e completo per la creazione e la gestione degli account di accesso. Tuttavia, un cookie provider di autenticazione basato su senza ASP.NET Core Identity può essere utilizzato. Per altre informazioni, vedere <xref:security/authentication/identity>.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/authentication/cookie/samples) ([procedura per il download](xref:index#how-to-download-a-sample))

A scopo dimostrativo nell'app di esempio, l'account utente per l'utente ipotetico, Maria Rodriguez, è hardcoded nell'app. Usare l'indirizzo di **posta elettronica** `maria.rodriguez@contoso.com` e qualsiasi password per accedere all'utente. L'utente viene autenticato nel `AuthenticateUser` metodo nel file *pages/account/login. cshtml. cs* . In un esempio reale, l'utente viene autenticato a fronte di un database.

## <a name="configuration"></a>Configurazione

Nel `Startup.ConfigureServices` metodo creare i servizi del middleware di autenticazione con i <xref:Microsoft.Extensions.DependencyInjection.AuthenticationServiceCollectionExtensions.AddAuthentication*> <xref:Microsoft.Extensions.DependencyInjection.CookieExtensions.AddCookie*> metodi e:

[!code-csharp[](cookie/samples/3.x/CookieSample/Startup.cs?name=snippet1)]

<xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationDefaults.AuthenticationScheme> passato a `AddAuthentication` imposta lo schema di autenticazione predefinito per l'app. `AuthenticationScheme` è utile quando sono presenti più istanze di cookie autenticazione e si desidera [autorizzare con uno schema specifico](xref:security/authorization/limitingidentitybyscheme). L'impostazione `AuthenticationScheme` di su [ Cookie AuthenticationDefaults. AuthenticationScheme](xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationDefaults.AuthenticationScheme) fornisce il valore " Cookie s" per lo schema. È possibile specificare qualsiasi valore stringa che distingua lo schema.

Lo schema di autenticazione dell'app è diverso dallo schema di cookie autenticazione dell'app. Quando uno cookie schema di autenticazione non viene fornito a <xref:Microsoft.Extensions.DependencyInjection.CookieExtensions.AddCookie*> , USA `CookieAuthenticationDefaults.AuthenticationScheme` (" Cookie s").

La cookie proprietà Authentication <xref:Microsoft.AspNetCore.Http.CookieBuilder.IsEssential> è impostata su per `true` impostazione predefinita. cookieSono consentite le autenticazioni quando un visitatore del sito non ha acconsentito alla raccolta dei dati. Per altre informazioni, vedere <xref:security/gdpr#essential-cookies>.

In `Startup.Configure` , chiamare `UseAuthentication` e `UseAuthorization` per impostare la `HttpContext.User` proprietà ed eseguire il middleware di autorizzazione per le richieste. Chiamare i `UseAuthentication` `UseAuthorization` metodi e prima di chiamare `UseEndpoints` :

[!code-csharp[](cookie/samples/3.x/CookieSample/Startup.cs?name=snippet2)]

La <xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationOptions> classe viene utilizzata per configurare le opzioni del provider di autenticazione.

Impostare `CookieAuthenticationOptions` nella configurazione del servizio per l'autenticazione nel `Startup.ConfigureServices` Metodo:

```csharp
services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(options =>
    {
        ...
    });
```

## <a name="cookie-policy-middleware"></a>Cookie Middleware dei criteri

Il [ Cookie middleware dei criteri](xref:Microsoft.AspNetCore.CookiePolicy.CookiePolicyMiddleware) Abilita le cookie funzionalità dei criteri. L'aggiunta del middleware alla pipeline di elaborazione delle app è sensibile agli ordini &mdash; , ma interessa solo i componenti downstream registrati nella pipeline.

```csharp
app.UseCookiePolicy(cookiePolicyOptions);
```

Utilizzare <xref:Microsoft.AspNetCore.Builder.CookiePolicyOptions> fornito al Cookie middleware dei criteri per controllare le caratteristiche globali dell' cookie elaborazione e agganciare i cookie gestori di elaborazione quando cookie vengono aggiunti o eliminati.

Il <xref:Microsoft.AspNetCore.Builder.CookiePolicyOptions.MinimumSameSitePolicy> valore predefinito è `SameSiteMode.Lax` consentire l'autenticazione OAuth2. Per applicare rigorosamente un criterio dello stesso sito di `SameSiteMode.Strict` , impostare il `MinimumSameSitePolicy` . Sebbene questa impostazione interrompa OAuth2 e altri schemi di autenticazione tra le origini, eleva il livello di cookie sicurezza per altri tipi di app che non si basano sull'elaborazione di richieste tra origini.

```csharp
var cookiePolicyOptions = new CookiePolicyOptions
{
    MinimumSameSitePolicy = SameSiteMode.Strict,
};
```

L' Cookie impostazione del middleware per i criteri di `MinimumSameSitePolicy` può influire sull'impostazione di `Cookie.SameSite` in `CookieAuthenticationOptions` impostazioni in base alla matrice riportata di seguito.

| MinimumSameSitePolicy | Cookie. Navigava sullostesso sito | Risultato Cookie . Impostazione navigava sullostesso sito |
| --------------------- | --------------- | --------------------------------- |
| SameSiteMode. None     | SameSiteMode. None<br>SameSiteMode. LAX<br>SameSiteMode. Strict | SameSiteMode. None<br>SameSiteMode. LAX<br>SameSiteMode. Strict |
| SameSiteMode. LAX      | SameSiteMode. None<br>SameSiteMode. LAX<br>SameSiteMode. Strict | SameSiteMode. LAX<br>SameSiteMode. LAX<br>SameSiteMode. Strict |
| SameSiteMode. Strict   | SameSiteMode. None<br>SameSiteMode. LAX<br>SameSiteMode. Strict | SameSiteMode. Strict<br>SameSiteMode. Strict<br>SameSiteMode. Strict |

## <a name="create-an-authentication-cookie"></a>Creare un'autenticazione cookie

Per creare un oggetto che cookie contiene informazioni sull'utente, costruire un oggetto <xref:System.Security.Claims.ClaimsPrincipal> . Le informazioni utente vengono serializzate e archiviate in cookie . 

Creare una <xref:System.Security.Claims.ClaimsIdentity> con qualsiasi richiesta <xref:System.Security.Claims.Claim> e chiamare <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.SignInAsync*> per accedere all'utente:

[!code-csharp[](cookie/samples/3.x/CookieSample/Pages/Account/Login.cshtml.cs?name=snippet1)]

[!INCLUDE[request localized comments](~/includes/code-comments-loc.md)]

`SignInAsync` Crea un oggetto crittografato cookie e lo aggiunge alla risposta corrente. Se `AuthenticationScheme` non è specificato, viene usato lo schema predefinito.

<xref:Microsoft.AspNetCore.Authentication.AuthenticationProperties.RedirectUri> viene utilizzato solo in alcuni percorsi specifici per impostazione predefinita, ad esempio il percorso di accesso e i percorsi di disconnessione. Per altre informazioni, vedere l' [ Cookie origine AuthenticationHandler](https://github.com/dotnet/aspnetcore/blob/f2e6e6ff334176540ef0b3291122e359c2106d1a/src/Security/Authentication/Cookies/src/CookieAuthenticationHandler.cs#L334).

Per la crittografia viene usato il sistema di [protezione dei dati](xref:security/data-protection/using-data-protection) di ASP.NET Core. Per un'app ospitata in più computer, bilanciamento del carico tra app o uso di un Web farm, [configurare la protezione dei dati](xref:security/data-protection/configuration/overview) in modo da usare lo stesso anello chiave e l'identificatore dell'app.

## <a name="sign-out"></a>Disconnetti

Per disconnettersi dall'utente corrente ed eliminare il relativo cookie , chiamare <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.SignOutAsync*> :

[!code-csharp[](cookie/samples/3.x/CookieSample/Pages/Account/Login.cshtml.cs?name=snippet2)]

Se `CookieAuthenticationDefaults.AuthenticationScheme` (o " Cookie s") non viene usato come schema (ad esempio, "Contoso Cookie "), fornire lo schema usato durante la configurazione del provider di autenticazione. In caso contrario, verrà utilizzato lo schema predefinito.

Quando il browser viene chiuso, Elimina automaticamente le istanze di basate su sessione cookie (non persistenti cookie ), ma nessuna viene cookie cancellata quando viene chiusa una singola scheda. Il server non riceve notifiche sugli eventi di chiusura di schede o browser.

## <a name="react-to-back-end-changes"></a>Reagire alle modifiche del back-end

Una volta cookie creato un oggetto, cookie è l'unica origine di identità. Se un account utente è disabilitato nei sistemi back-end:

* Il sistema di cookie autenticazione dell'app continua a elaborare le richieste in base all'autenticazione cookie .
* L'utente rimane connesso all'app fino a quando l'autenticazione cookie è valida.

L' <xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationEvents.ValidatePrincipal*> evento può essere usato per intercettare ed eseguire l'override della convalida dell' cookie identità. La convalida cookie di a ogni richiesta attenua il rischio di revoca degli utenti che accedono all'app.

Un approccio alla cookie convalida si basa sul tenere traccia del momento in cui il database utente viene modificato. Se il database non è stato modificato dopo l'emissione dell'utente cookie , non è necessario autenticare di nuovo l'utente se cookie è ancora valido. Nell'app di esempio, il database viene implementato in `IUserRepository` e archivia un `LastChanged` valore. Quando un utente viene aggiornato nel database, il `LastChanged` valore viene impostato sull'ora corrente.

Per invalidare un oggetto cookie quando il database viene modificato in base al `LastChanged` valore, creare cookie con un' `LastChanged` attestazione contenente il `LastChanged` valore corrente dal database:

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

Per implementare una sostituzione per l' `ValidatePrincipal` evento, scrivere un metodo con la firma seguente in una classe che deriva da <xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationEvents> :

```csharp
ValidatePrincipal(CookieValidatePrincipalContext)
```

Di seguito è riportato un esempio di implementazione di `CookieAuthenticationEvents` :

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

Registrare l'istanza degli eventi durante cookie la registrazione del servizio nel `Startup.ConfigureServices` metodo. Fornire una [registrazione del servizio con ambito](xref:fundamentals/dependency-injection#service-lifetimes) per la `CustomCookieAuthenticationEvents` classe:

```csharp
services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(options =>
    {
        options.EventsType = typeof(CustomCookieAuthenticationEvents);
    });

services.AddScoped<CustomCookieAuthenticationEvents>();
```

Si consideri una situazione in cui il nome dell'utente viene aggiornato &mdash; una decisione che non influisce sulla sicurezza in alcun modo. Se si desidera aggiornare l'entità utente in modo non distruttivo, chiamare `context.ReplacePrincipal` e impostare la `context.ShouldRenew` proprietà su `true` .

> [!WARNING]
> L'approccio descritto qui viene attivato a ogni richiesta. La convalida cookie dell'autenticazione per tutti gli utenti in ogni richiesta può comportare un notevole calo delle prestazioni per l'app.

## <a name="persistent-cookies"></a>S persistente cookie

È possibile che il venga cookie mantenuto tra le sessioni del browser. Questa persistenza deve essere abilitata solo con il consenso esplicito dell'utente con una casella di controllo "Remember me" all'accesso o un meccanismo simile. 

Il frammento di codice seguente crea un'identità e corrispondente che rimane inalterata cookie attraverso le chiusure del browser. Vengono rispettate tutte le impostazioni di scadenza scorrevoli configurate in precedenza. Se l'oggetto cookie scade quando il browser viene chiuso, il browser cancella il oggetto cookie dopo il riavvio.

Imposta <xref:Microsoft.AspNetCore.Authentication.AuthenticationProperties.IsPersistent> su `true` in <xref:Microsoft.AspNetCore.Authentication.AuthenticationProperties> :

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

## <a name="absolute-cookie-expiration"></a>cookieScadenza assoluta

È possibile impostare un'ora di scadenza assoluta con <xref:Microsoft.AspNetCore.Authentication.AuthenticationProperties.ExpiresUtc> . Per creare un oggetto persistente cookie , `IsPersistent` è necessario impostare anche. In caso contrario, cookie viene creato con una durata basata sulla sessione e potrebbe scadere prima o dopo il ticket di autenticazione che possiede. Quando `ExpiresUtc` è impostato, esegue l'override del valore dell' <xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationOptions.ExpireTimeSpan> opzione di <xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationOptions> , se impostata.

Il frammento di codice seguente crea un'identità e corrisponde alla cookie durata di 20 minuti. Questa operazione ignora le impostazioni di scadenza scorrevoli configurate in precedenza.

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

ASP.NET Core Identity è un provider di autenticazione completo e completo per la creazione e la gestione degli account di accesso. Tuttavia, un cookie provider di autenticazione basato su senza ASP.NET Core Identity può essere utilizzato. Per altre informazioni, vedere <xref:security/authentication/identity>.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/security/authentication/cookie/samples) ([procedura per il download](xref:index#how-to-download-a-sample))

A scopo dimostrativo nell'app di esempio, l'account utente per l'utente ipotetico, Maria Rodriguez, è hardcoded nell'app. Usare l'indirizzo di **posta elettronica** `maria.rodriguez@contoso.com` e qualsiasi password per accedere all'utente. L'utente viene autenticato nel `AuthenticateUser` metodo nel file *pages/account/login. cshtml. cs* . In un esempio reale, l'utente viene autenticato a fronte di un database.

## <a name="configuration"></a>Configurazione

Se l'app non usa il [metapacchetto Microsoft. AspNetCore. app](xref:fundamentals/metapackage-app), creare un riferimento al pacchetto nel file di progetto per [Microsoft. AspNetCore. Authentication. Cookie pacchetto s](https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.Cookies/) .

Nel `Startup.ConfigureServices` metodo creare il servizio middleware di autenticazione con i <xref:Microsoft.Extensions.DependencyInjection.AuthenticationServiceCollectionExtensions.AddAuthentication*> metodi e <xref:Microsoft.Extensions.DependencyInjection.CookieExtensions.AddCookie*> :

[!code-csharp[](cookie/samples/2.x/CookieSample/Startup.cs?name=snippet1)]

<xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationDefaults.AuthenticationScheme> passato a `AddAuthentication` imposta lo schema di autenticazione predefinito per l'app. `AuthenticationScheme` è utile quando sono presenti più istanze di cookie autenticazione e si desidera [autorizzare con uno schema specifico](xref:security/authorization/limitingidentitybyscheme). L'impostazione `AuthenticationScheme` di su [ Cookie AuthenticationDefaults. AuthenticationScheme](xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationDefaults.AuthenticationScheme) fornisce il valore " Cookie s" per lo schema. È possibile specificare qualsiasi valore stringa che distingua lo schema.

Lo schema di autenticazione dell'app è diverso dallo schema di cookie autenticazione dell'app. Quando uno cookie schema di autenticazione non viene fornito a <xref:Microsoft.Extensions.DependencyInjection.CookieExtensions.AddCookie*> , USA `CookieAuthenticationDefaults.AuthenticationScheme` (" Cookie s").

La cookie proprietà Authentication <xref:Microsoft.AspNetCore.Http.CookieBuilder.IsEssential> è impostata su per `true` impostazione predefinita. cookieSono consentite le autenticazioni quando un visitatore del sito non ha acconsentito alla raccolta dei dati. Per altre informazioni, vedere <xref:security/gdpr#essential-cookies>.

Nel `Startup.Configure` metodo chiamare il `UseAuthentication` metodo per richiamare il middleware di autenticazione che imposta la `HttpContext.User` Proprietà. Chiamare il `UseAuthentication` metodo prima di chiamare `UseMvcWithDefaultRoute` o `UseMvc` :

[!code-csharp[](cookie/samples/2.x/CookieSample/Startup.cs?name=snippet2)]

La <xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationOptions> classe viene utilizzata per configurare le opzioni del provider di autenticazione.

Impostare `CookieAuthenticationOptions` nella configurazione del servizio per l'autenticazione nel `Startup.ConfigureServices` Metodo:

```csharp
services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(options =>
    {
        ...
    });
```

## <a name="cookie-policy-middleware"></a>Cookie Middleware dei criteri

Il [ Cookie middleware dei criteri](xref:Microsoft.AspNetCore.CookiePolicy.CookiePolicyMiddleware) Abilita le cookie funzionalità dei criteri. L'aggiunta del middleware alla pipeline di elaborazione delle app è sensibile agli ordini &mdash; , ma interessa solo i componenti downstream registrati nella pipeline.

```csharp
app.UseCookiePolicy(cookiePolicyOptions);
```

Utilizzare <xref:Microsoft.AspNetCore.Builder.CookiePolicyOptions> fornito al Cookie middleware dei criteri per controllare le caratteristiche globali dell' cookie elaborazione e agganciare i cookie gestori di elaborazione quando cookie vengono aggiunti o eliminati.

Il <xref:Microsoft.AspNetCore.Builder.CookiePolicyOptions.MinimumSameSitePolicy> valore predefinito è `SameSiteMode.Lax` consentire l'autenticazione OAuth2. Per applicare rigorosamente un criterio dello stesso sito di `SameSiteMode.Strict` , impostare il `MinimumSameSitePolicy` . Sebbene questa impostazione interrompa OAuth2 e altri schemi di autenticazione tra le origini, eleva il livello di cookie sicurezza per altri tipi di app che non si basano sull'elaborazione di richieste tra origini.

```csharp
var cookiePolicyOptions = new CookiePolicyOptions
{
    MinimumSameSitePolicy = SameSiteMode.Strict,
};
```

L' Cookie impostazione del middleware per i criteri di `MinimumSameSitePolicy` può influire sull'impostazione di `Cookie.SameSite` in `CookieAuthenticationOptions` impostazioni in base alla matrice riportata di seguito.

| MinimumSameSitePolicy | Cookie. Navigava sullostesso sito | Risultato Cookie . Impostazione navigava sullostesso sito |
| --------------------- | --------------- | --------------------------------- |
| SameSiteMode. None     | SameSiteMode. None<br>SameSiteMode. LAX<br>SameSiteMode. Strict | SameSiteMode. None<br>SameSiteMode. LAX<br>SameSiteMode. Strict |
| SameSiteMode. LAX      | SameSiteMode. None<br>SameSiteMode. LAX<br>SameSiteMode. Strict | SameSiteMode. LAX<br>SameSiteMode. LAX<br>SameSiteMode. Strict |
| SameSiteMode. Strict   | SameSiteMode. None<br>SameSiteMode. LAX<br>SameSiteMode. Strict | SameSiteMode. Strict<br>SameSiteMode. Strict<br>SameSiteMode. Strict |

## <a name="create-an-authentication-cookie"></a>Creare un'autenticazione cookie

Per creare un oggetto che cookie contiene informazioni sull'utente, costruire un oggetto <xref:System.Security.Claims.ClaimsPrincipal> . Le informazioni utente vengono serializzate e archiviate in cookie . 

Creare una <xref:System.Security.Claims.ClaimsIdentity> con qualsiasi richiesta <xref:System.Security.Claims.Claim> e chiamare <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.SignInAsync*> per accedere all'utente:

[!code-csharp[](cookie/samples/2.x/CookieSample/Pages/Account/Login.cshtml.cs?name=snippet1)]

`SignInAsync` Crea un oggetto crittografato cookie e lo aggiunge alla risposta corrente. Se `AuthenticationScheme` non è specificato, viene usato lo schema predefinito.

Per la crittografia viene usato il sistema di [protezione dei dati](xref:security/data-protection/using-data-protection) di ASP.NET Core. Per un'app ospitata in più computer, bilanciamento del carico tra app o uso di un Web farm, [configurare la protezione dei dati](xref:security/data-protection/configuration/overview) in modo da usare lo stesso anello chiave e l'identificatore dell'app.

## <a name="sign-out"></a>Disconnetti

Per disconnettersi dall'utente corrente ed eliminare il relativo cookie , chiamare <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.SignOutAsync*> :

[!code-csharp[](cookie/samples/2.x/CookieSample/Pages/Account/Login.cshtml.cs?name=snippet2)]

Se `CookieAuthenticationDefaults.AuthenticationScheme` (o " Cookie s") non viene usato come schema (ad esempio, "Contoso Cookie "), fornire lo schema usato durante la configurazione del provider di autenticazione. In caso contrario, verrà utilizzato lo schema predefinito.

## <a name="react-to-back-end-changes"></a>Reagire alle modifiche del back-end

Una volta cookie creato un oggetto, cookie è l'unica origine di identità. Se un account utente è disabilitato nei sistemi back-end:

* Il sistema di cookie autenticazione dell'app continua a elaborare le richieste in base all'autenticazione cookie .
* L'utente rimane connesso all'app fino a quando l'autenticazione cookie è valida.

L' <xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationEvents.ValidatePrincipal*> evento può essere usato per intercettare ed eseguire l'override della convalida dell' cookie identità. La convalida cookie di a ogni richiesta attenua il rischio di revoca degli utenti che accedono all'app.

Un approccio alla cookie convalida si basa sul tenere traccia del momento in cui il database utente viene modificato. Se il database non è stato modificato dopo l'emissione dell'utente cookie , non è necessario autenticare di nuovo l'utente se cookie è ancora valido. Nell'app di esempio, il database viene implementato in `IUserRepository` e archivia un `LastChanged` valore. Quando un utente viene aggiornato nel database, il `LastChanged` valore viene impostato sull'ora corrente.

Per invalidare un oggetto cookie quando il database viene modificato in base al `LastChanged` valore, creare cookie con un' `LastChanged` attestazione contenente il `LastChanged` valore corrente dal database:

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

Per implementare una sostituzione per l' `ValidatePrincipal` evento, scrivere un metodo con la firma seguente in una classe che deriva da <xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationEvents> :

```csharp
ValidatePrincipal(CookieValidatePrincipalContext)
```

Di seguito è riportato un esempio di implementazione di `CookieAuthenticationEvents` :

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

Registrare l'istanza degli eventi durante cookie la registrazione del servizio nel `Startup.ConfigureServices` metodo. Fornire una [registrazione del servizio con ambito](xref:fundamentals/dependency-injection#service-lifetimes) per la `CustomCookieAuthenticationEvents` classe:

```csharp
services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(options =>
    {
        options.EventsType = typeof(CustomCookieAuthenticationEvents);
    });

services.AddScoped<CustomCookieAuthenticationEvents>();
```

Si consideri una situazione in cui il nome dell'utente viene aggiornato &mdash; una decisione che non influisce sulla sicurezza in alcun modo. Se si desidera aggiornare l'entità utente in modo non distruttivo, chiamare `context.ReplacePrincipal` e impostare la `context.ShouldRenew` proprietà su `true` .

> [!WARNING]
> L'approccio descritto qui viene attivato a ogni richiesta. La convalida cookie dell'autenticazione per tutti gli utenti in ogni richiesta può comportare un notevole calo delle prestazioni per l'app.

## <a name="persistent-cookies"></a>S persistente cookie

È possibile che il venga cookie mantenuto tra le sessioni del browser. Questa persistenza deve essere abilitata solo con il consenso esplicito dell'utente con una casella di controllo "Remember me" all'accesso o un meccanismo simile. 

Il frammento di codice seguente crea un'identità e corrispondente che rimane inalterata cookie attraverso le chiusure del browser. Vengono rispettate tutte le impostazioni di scadenza scorrevoli configurate in precedenza. Se l'oggetto cookie scade quando il browser viene chiuso, il browser cancella il oggetto cookie dopo il riavvio.

Imposta <xref:Microsoft.AspNetCore.Authentication.AuthenticationProperties.IsPersistent> su `true` in <xref:Microsoft.AspNetCore.Authentication.AuthenticationProperties> :

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

## <a name="absolute-cookie-expiration"></a>cookieScadenza assoluta

È possibile impostare un'ora di scadenza assoluta con <xref:Microsoft.AspNetCore.Authentication.AuthenticationProperties.ExpiresUtc> . Per creare un oggetto persistente cookie , `IsPersistent` è necessario impostare anche. In caso contrario, cookie viene creato con una durata basata sulla sessione e potrebbe scadere prima o dopo il ticket di autenticazione che possiede. Quando `ExpiresUtc` è impostato, esegue l'override del valore dell' <xref:Microsoft.AspNetCore.Builder.CookieAuthenticationOptions.ExpireTimeSpan> opzione di <xref:Microsoft.AspNetCore.Builder.CookieAuthenticationOptions> , se impostata.

Il frammento di codice seguente crea un'identità e corrisponde alla cookie durata di 20 minuti. Questa operazione ignora le impostazioni di scadenza scorrevoli configurate in precedenza.

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

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:security/authorization/limitingidentitybyscheme>
* <xref:security/authorization/claims>
* [Controlli dei ruoli basati su criteri](xref:security/authorization/roles#policy-based-role-checks)
* <xref:host-and-deploy/web-farm>
