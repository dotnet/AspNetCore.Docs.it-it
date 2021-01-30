---
title: Autorizzare con uno schema specifico in ASP.NET Core
author: rick-anderson
description: In questo articolo viene illustrato come limitare l'identità a uno schema specifico quando si utilizzano più metodi di autenticazione.
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.date: 11/08/2019
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
uid: security/authorization/limitingidentitybyscheme
ms.openlocfilehash: c4cbec1b829fb8fd47f7b6924b6870bd5dd7097d
ms.sourcegitcommit: 83524f739dd25fbfa95ee34e95342afb383b49fe
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 01/29/2021
ms.locfileid: "99057304"
---
# <a name="authorize-with-a-specific-scheme-in-aspnet-core"></a>Autorizzare con uno schema specifico in ASP.NET Core

Per un'introduzione agli schemi di autenticazione in ASP.NET Core, vedere [schema di autenticazione](xref:security/authentication/index#authentication-scheme).

In alcuni scenari, ad esempio le applicazioni a pagina singola (Spa), è comune usare più metodi di autenticazione. Ad esempio, l'app può usare l' cookie autenticazione basata su per accedere e l'autenticazione di connessione JWT per le richieste JavaScript. In alcuni casi, l'app può avere più istanze di un gestore di autenticazione. Ad esempio, due cookie gestori dove uno contiene un'identità di base e uno viene creato quando viene attivata un'autenticazione a più fattori (multi-factor authentication). L'autenticazione a più fattori può essere attivata perché l'utente ha richiesto un'operazione che richiede una maggiore sicurezza. Per altre informazioni sull'applicazione dell'autenticazione a più fattori quando un utente richiede una risorsa che richiede l'autenticazione a più fattori, vedere la [sezione relativa alla protezione](https://github.com/dotnet/AspNetCore.Docs/issues/15791#issuecomment-580464195)dei problemi di GitHub con l'autenticazione a

Uno schema di autenticazione viene denominato quando il servizio di autenticazione viene configurato durante l'autenticazione. Ad esempio:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // Code omitted for brevity

    services.AddAuthentication()
        .AddCookie(options => {
            options.LoginPath = "/Account/Unauthorized/";
            options.AccessDeniedPath = "/Account/Forbidden/";
        })
        .AddJwtBearer(options => {
            options.Audience = "http://localhost:5001/";
            options.Authority = "http://localhost:5000/";
        });
```

Nel codice precedente sono stati aggiunti due gestori di autenticazione: uno per cookie s e uno per Bearer.

>[!NOTE]
>Se si specifica lo schema predefinito `HttpContext.User` , la proprietà viene impostata su tale identità. Se questo comportamento non è necessario, disabilitarlo richiamando il formato senza parametri di `AddAuthentication` .

## <a name="selecting-the-scheme-with-the-authorize-attribute"></a>Selezione dello schema con l'attributo autorizzi

Al momento dell'autorizzazione, l'app indica il gestore da usare. Selezionare il gestore con cui l'app autorizzerà il passaggio di un elenco delimitato da virgole di schemi di autenticazione a `[Authorize]` . L' `[Authorize]` attributo specifica lo schema o gli schemi di autenticazione da utilizzare, indipendentemente dal fatto che sia configurato un valore predefinito. Ad esempio:

```csharp
[Authorize(AuthenticationSchemes = AuthSchemes)]
public class MixedController : Controller
    // Requires the following imports:
    // using Microsoft.AspNetCore.Authentication.Cookies;
    // using Microsoft.AspNetCore.Authentication.JwtBearer;
    private const string AuthSchemes =
        CookieAuthenticationDefaults.AuthenticationScheme + "," +
        JwtBearerDefaults.AuthenticationScheme;
```

Nell'esempio precedente, entrambi i cookie gestori di porta e sono eseguiti e hanno la possibilità di creare e aggiungere un'identità per l'utente corrente. Specificando solo un singolo schema, viene eseguito il gestore corrispondente.

```csharp
[Authorize(AuthenticationSchemes = 
    JwtBearerDefaults.AuthenticationScheme)]
public class MixedController : Controller
```

Nel codice precedente viene eseguito solo il gestore con lo schema "Bearer". Qualsiasi cookie identità basata su viene ignorata.

## <a name="selecting-the-scheme-with-policies"></a>Selezione dello schema con i criteri

Se si preferisce specificare gli schemi desiderati nel [criterio](xref:security/authorization/policies), è possibile impostare la `AuthenticationSchemes` raccolta quando si aggiungono i criteri:

```csharp
services.AddAuthorization(options =>
{
    options.AddPolicy("Over18", policy =>
    {
        policy.AuthenticationSchemes.Add(JwtBearerDefaults.AuthenticationScheme);
        policy.RequireAuthenticatedUser();
        policy.Requirements.Add(new MinimumAgeRequirement());
    });
});
```

Nell'esempio precedente, il criterio "Over18" viene eseguito solo con l'identità creata dal gestore "Bearer". Usare il criterio impostando la `[Authorize]` proprietà dell'attributo `Policy` :

```csharp
[Authorize(Policy = "Over18")]
public class RegistrationController : Controller
```

::: moniker range=">= aspnetcore-2.0"

## <a name="use-multiple-authentication-schemes"></a>Usare più schemi di autenticazione

Alcune app potrebbero dover supportare più tipi di autenticazione. Ad esempio, l'app potrebbe autenticare gli utenti da Azure Active Directory e da un database degli utenti. Un altro esempio è un'app che autentica gli utenti da Active Directory Federation Services e Azure Active Directory B2C. In questo caso, l'app deve accettare una bearer token JWT da diverse autorità emittenti.

Aggiungere tutti gli schemi di autenticazione che si desidera accettare. Ad esempio, il codice seguente in `Startup.ConfigureServices` aggiunge due schemi di autenticazione JWT Bearer con emittenti differenti:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // Code omitted for brevity

    services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
        .AddJwtBearer(options =>
        {
            options.Audience = "https://localhost:5000/";
            options.Authority = "https://localhost:5000/identity/";
        })
        .AddJwtBearer("AzureAD", options =>
        {
            options.Audience = "https://localhost:5000/";
            options.Authority = "https://login.microsoftonline.com/eb971100-6f99-4bdc-8611-1bc8edd7f436/";
        });
}
```

> [!NOTE]
> Con lo schema di autenticazione predefinito viene registrata una sola autenticazione JWT Bearer `JwtBearerDefaults.AuthenticationScheme` . È necessario registrare un'autenticazione aggiuntiva con uno schema di autenticazione univoco.

Il passaggio successivo consiste nell'aggiornare i criteri di autorizzazione predefiniti affinché accettino entrambi gli schemi di autenticazione. Ad esempio:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // Code omitted for brevity

    services.AddAuthorization(options =>
    {
        var defaultAuthorizationPolicyBuilder = new AuthorizationPolicyBuilder(
            JwtBearerDefaults.AuthenticationScheme,
            "AzureAD");
        defaultAuthorizationPolicyBuilder = 
            defaultAuthorizationPolicyBuilder.RequireAuthenticatedUser();
        options.DefaultPolicy = defaultAuthorizationPolicyBuilder.Build();
    });
}
```

Quando viene eseguito l'override dei criteri di autorizzazione predefiniti, è possibile usare l' `[Authorize]` attributo nei controller. Il controller accetta quindi le richieste con JWT emesso dal primo o dal secondo emittente.

::: moniker-end

Vedere [questo problema di GitHub](https://github.com/dotnet/aspnetcore/issues/26002) sull'uso di più schemi di autenticazione.
