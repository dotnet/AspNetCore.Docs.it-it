---
title: Esempio navigava sullostesso sito 2,1 MVC ASP.NET Core cookie
author: rick-anderson
description: Esempio navigava sullostesso sito 2,1 MVC ASP.NET Core cookie
monikerRange: '>= aspnetcore-2.1 < aspnetcore-3.0'
ms.author: riande
ms.custom: mvc
ms.date: 12/03/2019
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
uid: security/samesite/mvc21
ms.openlocfilehash: 8f819d283e136a63ad9f82d6432a93866210b36b
ms.sourcegitcommit: a1db01b4d3bd8c57d7a9c94ce122a6db68002d66
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/04/2021
ms.locfileid: "102110105"
---
# <a name="aspnet-core-21-mvc-samesite-cookie-sample"></a>Esempio navigava sullostesso sito 2,1 MVC ASP.NET Core cookie

ASP.NET Core 2,1 dispone del supporto incorporato per l'attributo [navigava sullostesso sito](https://www.owasp.org/index.php/SameSite) , ma è stato scritto nello standard originale. Il [comportamento con patch](https://github.com/dotnet/aspnetcore/issues/8212) ha modificato il significato di `SameSite.None` per emettere l'attributo navigava sullostesso sito con un valore di `None` , anziché non creare il valore. Se non si vuole creare il valore, è possibile impostare la `SameSite` proprietà su cookie -1.

[!INCLUDE[](~/includes/SameSiteIdentity.md)]

## <a name="writing-the-samesite-attribute"></a><a name="sampleCode"></a>Scrittura dell'attributo navigava sullostesso sito

Di seguito è riportato un esempio di come scrivere un attributo navigava sullostesso sito in un oggetto cookie :

```csharp
var cookieOptions = new CookieOptions
{
    // Set the secure flag, which Chrome's changes will require for SameSite none.
    // Note this will also require you to be running on HTTPS
    Secure = true,

    // Set the cookie to HTTP only which is good practice unless you really do need
    // to access it client side in scripts.
    HttpOnly = true,

    // Add the SameSite attribute, this will emit the attribute with a value of none.
    // To not emit the attribute at all set the SameSite property to (SameSiteMode)(-1).
    SameSite = SameSiteMode.None
};

// Add the cookie to the response cookie collection
Response.Cookies.Append(CookieName, "cookieValue", cookieOptions);
```

## <a name="setting-cookie-authentication-and-session-state-cookies"></a>Impostazione dell' Cookie autenticazione e dello stato della sessione cookie

Cookie l'autenticazione, lo stato della sessione e [vari altri componenti](../samesite.md?view=aspnetcore-2.1) impostano le opzioni di navigava sullostesso sito tramite Cookie Opzioni, ad esempio

```csharp
services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(options =>
    {
        options.Cookie.SameSite = SameSiteMode.None;
        options.Cookie.SecurePolicy = CookieSecurePolicy.Always;
        options.Cookie.IsEssential = true;
    });

services.AddSession(options =>
{
    options.Cookie.SameSite = SameSiteMode.None;
    options.Cookie.SecurePolicy = CookieSecurePolicy.Always;
    options.Cookie.IsEssential = true;
});
```

Nel codice precedente, sia l' cookie autenticazione che lo stato della sessione impostano l'attributo navigava sullostesso sito su, generando `None` l'attributo con un `None` valore e impostando anche l'attributo Secure su true.

### <a name="run-the-sample"></a>Eseguire l'esempio

Se si esegue il [progetto di esempio](https://github.com/blowdart/AspNetSameSiteSamples/tree/master/AspNetCore21MVC), caricare il debugger del browser nella pagina iniziale e utilizzarlo per visualizzare la cookie raccolta per il sito. A tale scopo, in Edge e Chrome premere `F12` quindi selezionare la `Application` scheda e fare clic sull'URL del sito sotto l' `Cookies` opzione nella `Storage` sezione.

![Browser debugger::: NO-LOC (cookie)::: list](BrowserDebugger.png)

Nell'immagine precedente è possibile vedere che l'oggetto cookie creato dall'esempio quando si fa clic sul pulsante "create navigava sullostesso sito Cookie " ha un valore di attributo navigava sullostesso sito `Lax` , che corrisponde al valore impostato nel [codice di esempio](#sampleCode).

## <a name="intercepting-cookies"></a><a name="interception"></a>Intercettazione di cookie s

Per intercettare i cookie , per modificare il valore None in base al relativo supporto nell'agente browser dell'utente, è necessario usare il `CookiePolicy` middleware. Questa deve essere inserita nella pipeline di richieste HTTP **prima** di tutti i componenti che scrivono cookie e configurati in `ConfigureServices()` .

Per inserirlo nell'uso della pipeline `app.UseCookiePolicy()` nel `Configure(IApplicationBuilder, IHostingEnvironment)` metodo in [Startup.cs](https://github.com/blowdart/AspNetSameSiteSamples/blob/master/AspNetCore21MVC/Startup.cs). Ad esempio:

```csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    if (env.IsDevelopment())
    {
       app.UseDeveloperExceptionPage();
    }
    else
    {
        app.UseExceptionHandler("/Home/Error");
        app.UseHsts();
    }

    app.UseHttpsRedirection();
    app.UseStaticFiles();
    app.UseCookiePolicy();
    app.UseAuthentication();
    app.UseSession();

    app.UseMvc(routes =>
    {
        routes.MapRoute(
            name: "default",
            template: "{controller=Home}/{action=Index}/{id?}");
    });
}
```

Nel `ConfigureServices(IServiceCollection services)` configurare il cookie criterio per la chiamata a una classe helper quando cookie vengono aggiunti o eliminati. Ad esempio:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.Configure<CookiePolicyOptions>(options =>
    {
        options.CheckConsentNeeded = context => true;
        options.MinimumSameSitePolicy = SameSiteMode.None;
        options.OnAppendCookie = cookieContext =>
            CheckSameSite(cookieContext.Context, cookieContext.CookieOptions);
        options.OnDeleteCookie = cookieContext =>
            CheckSameSite(cookieContext.Context, cookieContext.CookieOptions);
    });
}

private void CheckSameSite(HttpContext httpContext, CookieOptions options)
{
    if (options.SameSite == SameSiteMode.None)
    {
        var userAgent = httpContext.Request.Headers["User-Agent"].ToString();
        if (SameSite.BrowserDetection.DisallowsSameSiteNone(userAgent))
        {
            options.SameSite = (SameSiteMode)(-1);
        }
    }
}
```

Funzione helper `CheckSameSite(HttpContext, CookieOptions)` :

* Viene chiamato quando cookie vengono aggiunti alla richiesta o eliminati dalla richiesta.
* Verifica se la `SameSite` proprietà è impostata su `None` .
* Se `SameSite` è impostato su `None` e l'agente utente corrente non supporta il valore dell'attributo None. Il controllo viene eseguito usando la classe [SameSiteSupport](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/security/samesite/sample/snippets/SameSiteSupport.cs) :
  * Imposta `SameSite` per non creare il valore impostando la proprietà su `(SameSiteMode)(-1)`

## <a name="targeting-net-framework"></a>.NET Framework di destinazione

ASP.NET Core e System. Web (ASP.NET 4. x) hanno implementazioni indipendenti di navigava sullostesso sito. Le patch navigava sullostesso sito KB per .NET Framework non sono necessarie se si utilizza ASP.NET Core o meno il requisito di versione minima del Framework System. Web navigava sullostesso sito (.NET Framework 4.7.2) per ASP.NET Core.

ASP.NET Core su .NET richiede l'aggiornamento delle dipendenze dei pacchetti NuGet per ottenere le correzioni appropriate.

Per ottenere le modifiche ASP.NET Core per .NET Framework assicurarsi di disporre di un riferimento diretto alle versioni e ai pacchetti con patch (2.1.14 o versioni successive 2,1).

```xml
<PackageReference Include="Microsoft.Net.Http.Headers" Version="2.1.14" />
<PackageReference Include="Microsoft.AspNetCore.CookiePolicy" Version="2.1.14" />
```

### <a name="more-information"></a>Altre informazioni
 
[Aggiornamenti Chrome](https://www.chromium.org/updates/same-site) 
 Documentazione di ASP.NET Core [navigava sullostesso sito](../samesite.md?view=aspnetcore-2.1) 
 [Annuncio di modifica di ASP.NET Core 2,1 navigava sullostesso sito](https://github.com/dotnet/aspnetcore/issues/8212)