---
title: Panoramica dell'autenticazione ASP.NET Core
author: mjrousos
description: Informazioni sull'autenticazione in ASP.NET Core.
ms.author: riande
ms.custom: mvc
ms.date: 1/24/2021
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
uid: security/authentication/index
ms.openlocfilehash: 72036e9c4c92ee5dd82ac4a67e766fb0e5c8f924
ms.sourcegitcommit: 83524f739dd25fbfa95ee34e95342afb383b49fe
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 01/29/2021
ms.locfileid: "99057291"
---
# <a name="overview-of-aspnet-core-authentication"></a>Panoramica dell'autenticazione ASP.NET Core

Di [Mike risveglia](https://github.com/mjrousos)

L'autenticazione è il processo di determinazione dell'identità di un utente. L' [autorizzazione](xref:security/authorization/introduction) è il processo che consente di determinare se un utente ha accesso a una risorsa. In ASP.NET Core, l'autenticazione viene gestita da `IAuthenticationService` , che viene utilizzata dal [middleware](xref:fundamentals/middleware/index)di autenticazione. Il servizio di autenticazione usa i gestori di autenticazione registrati per completare le azioni correlate all'autenticazione. Di seguito sono riportati alcuni esempi di azioni correlate all'autenticazione:

* Autenticazione di un utente.
* Risposta quando un utente non autenticato tenta di accedere a una risorsa con restrizioni.

I gestori di autenticazione registrati e le relative opzioni di configurazione sono denominati "schemi".

Gli schemi di autenticazione vengono specificati registrando i servizi di autenticazione in `Startup.ConfigureServices` :

* Chiamando un metodo di estensione specifico dello schema dopo una chiamata a `services.AddAuthentication` (ad `AddJwtBearer` esempio o `AddCookie` , ad esempio). Questi metodi di estensione usano [AuthenticationBuilder. AddScheme](xref:Microsoft.AspNetCore.Authentication.AuthenticationBuilder.AddScheme*) per registrare gli schemi con le impostazioni appropriate.
* Meno comunemente, chiamando direttamente [AuthenticationBuilder. AddScheme](xref:Microsoft.AspNetCore.Authentication.AuthenticationBuilder.AddScheme*) .

Il codice seguente, ad esempio, registra i gestori e i servizi di autenticazione per gli cookie schemi di autenticazione di JWT Bearer:

```csharp
services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(JwtBearerDefaults.AuthenticationScheme, options => Configuration.Bind("JwtSettings", options))
    .AddCookie(CookieAuthenticationDefaults.AuthenticationScheme, options => Configuration.Bind("CookieSettings", options));
```

Il `AddAuthentication` parametro `JwtBearerDefaults.AuthenticationScheme` è il nome dello schema da utilizzare per impostazione predefinita quando non è richiesto uno schema specifico.

Se si utilizzano più schemi, i criteri di autorizzazione o gli attributi di autorizzazione possono [specificare lo schema di autenticazione (o gli schemi)](xref:security/authorization/limitingidentitybyscheme) da cui dipendono per autenticare l'utente. Nell'esempio precedente, lo cookie schema di autenticazione può essere usato specificandone il nome ( `CookieAuthenticationDefaults.AuthenticationScheme` per impostazione predefinita, ma è possibile specificare un nome diverso quando si chiama `AddCookie` ).

In alcuni casi, la chiamata a `AddAuthentication` viene eseguita automaticamente da altri metodi di estensione. Ad esempio, quando [ASP.NET Core Identity](xref:security/authentication/identity) si usa, `AddAuthentication` viene chiamato internamente.

Il middleware di autenticazione viene aggiunto in chiamando `Startup.Configure` il <xref:Microsoft.AspNetCore.Builder.AuthAppBuilderExtensions.UseAuthentication*> metodo di estensione nell'app `IApplicationBuilder` . La chiamata a `UseAuthentication` registra il middleware che utilizza gli schemi di autenticazione registrati in precedenza. Chiamare `UseAuthentication` prima di qualsiasi middleware che dipende da utenti autenticati. Quando si usa il routing degli endpoint, la chiamata a `UseAuthentication` deve essere:

* Dopo `UseRouting` , in modo che le informazioni sulla Route siano disponibili per le decisioni di autenticazione.
* Prima di `UseEndpoints` , in modo che gli utenti siano autenticati prima di accedere agli endpoint.

## <a name="authentication-concepts"></a>Concetti di autenticazione

L'autenticazione è responsabile della concessione dell'autorizzazione per l' <xref:System.Security.Claims.ClaimsPrincipal> autorizzazione a prendere decisioni di autorizzazione. Sono disponibili più approcci dello schema di autenticazione per selezionare il gestore di autenticazione responsabile della generazione del set di attestazioni corretto:

  * [Schema di autenticazione](xref:security/authorization/limitingidentitybyscheme), anche illustrato nella sezione successiva.
  * Schema di autenticazione predefinito, illustrato nella sezione successiva.
  * Impostare direttamente [HttpContext. User](xref:Microsoft.AspNetCore.Http.HttpContext.User).

Nessun sondaggio automatico degli schemi. Se non si specifica lo schema predefinito, è necessario specificare lo schema nell'attributo autorizzo. in caso contrario, viene generato l'errore seguente:

  InvalidOperationException: non è stato specificato alcun authenticationScheme e non è stato trovato alcun DefaultAuthenticateScheme. Gli schemi predefiniti possono essere impostati utilizzando AddAuthentication (String defaultScheme) o AddAuthentication (Action &lt; AuthenticationOptions &gt; configureOptions).

### <a name="authentication-scheme"></a>Schema di autenticazione

Lo [schema di autenticazione](xref:security/authorization/limitingidentitybyscheme) può selezionare il gestore di autenticazione responsabile della generazione del set di attestazioni corretto. Per altre informazioni, vedere [autorizzare con uno schema specifico](xref:security/authorization/limitingidentitybyscheme).

Uno schema di autenticazione è un nome che corrisponde a:

* Gestore di autenticazione
* Opzioni per la configurazione di un'istanza specifica del gestore.

Gli schemi sono utili come meccanismo per fare riferimento ai comportamenti di autenticazione, verifica e proibizione del gestore associato. Un criterio di autorizzazione, ad esempio, può utilizzare i nomi degli schemi per specificare lo schema di autenticazione (o gli schemi) da utilizzare per autenticare l'utente. Quando si configura l'autenticazione, è normale specificare lo schema di autenticazione predefinito. Viene utilizzato lo schema predefinito, a meno che una risorsa non richieda uno schema specifico. È anche possibile:

* Specificare diversi schemi predefiniti da usare per l'autenticazione, la verifica e la proibizione delle azioni.
* Combinare più schemi in uno usando gli [schemi dei criteri](xref:security/authentication/policyschemes).

### <a name="authentication-handler"></a>Gestore di autenticazione

Un gestore di autenticazione:

* È un tipo che implementa il comportamento di uno schema.
* È derivato da <xref:Microsoft.AspNetCore.Authentication.IAuthenticationHandler> o <xref:Microsoft.AspNetCore.Authentication.AuthenticationHandler`1> .
* Ha la responsabilità principale di autenticare gli utenti.

In base alla configurazione dello schema di autenticazione e al contesto della richiesta in ingresso, i gestori di autenticazione:

* Crea <xref:Microsoft.AspNetCore.Authentication.AuthenticationTicket> oggetti che rappresentano l'identità dell'utente se l'autenticazione ha esito positivo.
* Se l'autenticazione ha esito negativo, restituire ' nessun risultato ' o ' errore '.
* Sono disponibili metodi per la verifica e la proibizione delle azioni per gli utenti che tentano di accedere alle risorse:
  * Non sono autorizzati ad accedere.
  * Quando non sono autenticati (Challenge).

### <a name="authenticate"></a>Authenticate

Un'azione di autenticazione dello schema di autenticazione è responsabile della costruzione dell'identità dell'utente in base al contesto della richiesta. Restituisce un valore <xref:Microsoft.AspNetCore.Authentication.AuthenticateResult> che indica se l'autenticazione ha avuto esito positivo e, in caso affermativo, l'identità dell'utente in un ticket di autenticazione. Vedere <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.AuthenticateAsync%2A>. Gli esempi di autenticazione includono:

* Uno cookie schema di autenticazione che costruisce l'identità dell'utente da cookie s.
* Uno schema di portatore JWT che deserializza e convalida un JWT bearer token per costruire l'identità dell'utente.

### <a name="challenge"></a>Sfida

Una richiesta di autenticazione viene richiamata dall'autorizzazione quando un utente non autenticato richiede un endpoint che richiede l'autenticazione. Viene eseguita una richiesta di autenticazione, ad esempio quando un utente anonimo richiede una risorsa limitata o fa clic su un collegamento di accesso. L'autorizzazione richiama una richiesta di verifica utilizzando gli schemi di autenticazione specificati oppure il valore predefinito se non ne viene specificato alcuno. Vedere <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.ChallengeAsync%2A>. Gli esempi di richiesta di autenticazione includono:

* Uno cookie schema di autenticazione che reindirizza l'utente a una pagina di accesso.
* Uno schema di portatore JWT che restituisce un risultato 401 con un' `www-authenticate: bearer` intestazione.

Un'azione di verifica deve consentire all'utente di conoscere il meccanismo di autenticazione da usare per accedere alla risorsa richiesta.

### <a name="forbid"></a>Impediscono

Un'azione di proibisce dello schema di autenticazione viene chiamata dall'autorizzazione quando un utente autenticato tenta di accedere a una risorsa a cui non è consentito l'accesso. Vedere <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.ForbidAsync%2A>. Gli esempi di autenticazione proibita includono:
* Uno cookie schema di autenticazione che reindirizza l'utente a una pagina che indica che l'accesso non è consentito.
* Uno schema di porta JWT che restituisce un risultato 403.
* Uno schema di autenticazione personalizzato che reindirizza a una pagina in cui l'utente può richiedere l'accesso alla risorsa.

Un'azione proibita può comunicare all'utente:

* Sono autenticati.
* Non sono autorizzati ad accedere alla risorsa richiesta.

Vedere i collegamenti seguenti per le differenze tra la verifica e la proibizione:

* [Verifica e Impedisci un gestore di risorse operativo](xref:security/authorization/resourcebased#challenge-and-forbid-with-an-operational-resource-handler).
* [Differenze tra la sfida e la proibizione](xref:security/authorization/secure-data#challenge).

## <a name="authentication-providers-per-tenant"></a>Provider di autenticazione per tenant

ASP.NET Core Framework non dispone di una soluzione incorporata per l'autenticazione multi-tenant.
Sebbene sia certamente possibile che i clienti ne scrivano uno, usando le funzionalità predefinite, si consiglia ai clienti di esaminare [Orchard Core](https://www.orchardcore.net/) a questo scopo.

Orchard core è:

* Un Framework di app modulare e multi-tenant open source creato con ASP.NET Core.
* Un sistema di gestione dei contenuti (CMS) basato su tale framework di app.

Vedere l'origine [Orchard Core](https://github.com/OrchardCMS/OrchardCore) per un esempio di provider di autenticazione per ogni tenant.

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:security/authorization/limitingidentitybyscheme>
* <xref:security/authentication/policyschemes>
* <xref:security/authorization/secure-data>
* [Richiedi utenti autenticati a livello globale](xref:security/authorization/secure-data#rau)
* [Problema di GitHub sull'uso di più schemi di autenticazione](https://github.com/dotnet/aspnetcore/issues/26002)
