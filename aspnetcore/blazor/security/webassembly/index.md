---
title: ASP.NET Core protetto Blazor WebAssembly
author: guardrex
description: Informazioni su come proteggere Blazor le app WebAssemlby come applicazioni a pagina singola (Spa).
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
uid: blazor/security/webassembly/index
ms.openlocfilehash: fc2ebae6e88e312aafec790229f978c3130e64de
ms.sourcegitcommit: 04ad9cd26fcaa8bd11e261d3661f375f5f343cdc
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/10/2021
ms.locfileid: "100106687"
---
# <a name="secure-aspnet-core-blazor-webassembly"></a>ASP.NET Core protetto Blazor WebAssembly

Di [Javier Calvarro Nelson](https://github.com/javiercn)

Blazor WebAssembly le app sono protette in modo analogo alle applicazioni a pagina singola (Spa). Esistono diversi approcci per l'autenticazione degli utenti in Spa, ma l'approccio più comune e completo consiste nell'usare un'implementazione basata sul [protocollo OAuth 2,0](https://oauth.net/), ad esempio [OPENID Connect (OIDC)](https://openid.net/connect/).

## <a name="authentication-library"></a>Libreria di autenticazione

Blazor WebAssembly supporta l'autenticazione e l'autorizzazione delle app usando OIDC tramite la [`Microsoft.AspNetCore.Components.WebAssembly.Authentication`](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.WebAssembly.Authentication) libreria. La libreria fornisce un set di primitive per l'autenticazione uniforme rispetto a ASP.NET Core backend. La libreria si integra ASP.NET Core Identity con il supporto dell'autorizzazione API basato sul [ Identity server](https://identityserver.io/). La libreria può eseguire l'autenticazione per qualsiasi provider di terze parti Identity (IP) che supporta OIDC, chiamati provider OpenID (op).

Il supporto per l'autenticazione in Blazor WebAssembly è basato sulla `oidc-client.js` libreria, che consente di gestire i dettagli del protocollo di autenticazione sottostante.

Sono disponibili altre opzioni per l'autenticazione di Spa, ad esempio l'uso di navigava sullostesso sito cookie s. Tuttavia, la progettazione di Blazor WebAssembly viene stabilita in OAuth e OIDC come opzione migliore per l'autenticazione nelle Blazor WebAssembly app. È stata scelta l'autenticazione basata su [token](xref:security/anti-request-forgery#token-based-authentication) basata su [token Web JSON (token JWT)](https://self-issued.info/docs/draft-ietf-oauth-json-web-token.html) per motivi funzionali e di sicurezza: [ cookie ](xref:security/anti-request-forgery#cookie-based-authentication)

* L'uso di un protocollo basato su token offre una superficie di attacco più piccola, poiché i token non vengono inviati in tutte le richieste.
* Gli endpoint server non richiedono la protezione da [richieste intersito falsificazione (CSRF)](xref:security/anti-request-forgery) perché i token vengono inviati in modo esplicito. In questo modo è possibile ospitare Blazor WebAssembly app insieme a Razor app MVC o pagine.
* I token hanno autorizzazioni più strette rispetto a cookie . Ad esempio, non è possibile usare i token per gestire l'account utente o modificare la password di un utente, a meno che tale funzionalità non venga implementata in modo esplicito.
* I token hanno una durata breve, un'ora per impostazione predefinita, che limita la finestra di attacco. I token possono anche essere revocati in qualsiasi momento.
* Il token JWT autonomo offre garanzie al client e al server per il processo di autenticazione. Un client, ad esempio, è in grado di rilevare e verificare che i token ricevuti siano legittimi e che siano stati emessi come parte di un determinato processo di autenticazione. Se una terza parte tenta di cambiare un token durante il processo di autenticazione, il client può rilevare il token cambiato ed evitare di usarlo.
* I token con OAuth e OIDC non si basano sulla corretta presenza dell'agente utente per assicurarsi che l'applicazione sia protetta.
* I protocolli basati su token, ad esempio OAuth e OIDC, consentono di autenticare e autorizzare app ospitate e autonome con lo stesso set di caratteristiche di sicurezza.

## <a name="authentication-process-with-oidc"></a>Processo di autenticazione con OIDC

La [`Microsoft.AspNetCore.Components.WebAssembly.Authentication`](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.WebAssembly.Authentication) libreria offre diverse primitive per implementare l'autenticazione e l'autorizzazione usando OIDC. In termini generali, l'autenticazione funziona nel modo seguente:

* Quando un utente anonimo seleziona il pulsante di accesso o richiede una pagina con l' [ `[Authorize]` attributo](xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute) applicato, l'utente viene reindirizzato alla pagina di accesso dell'app ( `/authentication/login` ).
* Nella pagina di accesso, la libreria di autenticazione prepara un reindirizzamento all'endpoint di autorizzazione. L'endpoint di autorizzazione è esterno all' Blazor WebAssembly app ed è possibile ospitarlo a un'origine separata. L'endpoint è responsabile per determinare se l'utente è autenticato e per emettere uno o più token in risposta. La libreria di autenticazione fornisce un callback di accesso per ricevere la risposta di autenticazione.
  * Se l'utente non è autenticato, l'utente viene reindirizzato al sistema di autenticazione sottostante, che è in genere ASP.NET Core Identity .
  * Se l'utente è già stato autenticato, l'endpoint di autorizzazione genera i token appropriati e reindirizza di nuovo il browser all'endpoint di callback dell'account di accesso ( `/authentication/login-callback` ).
* Quando l' Blazor WebAssembly app carica l'endpoint di callback dell'account di accesso ( `/authentication/login-callback` ), viene elaborata la risposta di autenticazione.
  * Se il processo di autenticazione viene completato correttamente, l'utente viene autenticato e, facoltativamente, restituito all'URL protetto originale richiesto dall'utente.
  * Se il processo di autenticazione ha esito negativo per qualsiasi motivo, l'utente viene inviato alla pagina di accesso non riuscita ( `/authentication/login-failed` ) e viene visualizzato un errore.

## <a name="authentication-component"></a>Componente `Authentication`

Il `Authentication` componente ( `Pages/Authentication.razor` ) gestisce le operazioni di autenticazione remota e consente all'app di:

* Configurare le route delle app per gli Stati di autenticazione.
* Imposta il contenuto dell'interfaccia utente per gli Stati di autenticazione.
* Gestire lo stato di autenticazione.

Le azioni di autenticazione, ad esempio la registrazione o la firma in un utente, vengono passate al Blazor componente del Framework <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteAuthenticatorViewCore%601> , che rende permanente e controlla lo stato tra le operazioni di autenticazione.

Per altre informazioni ed esempi, vedere <xref:blazor/security/webassembly/additional-scenarios>.

## <a name="authorization"></a>Autorizzazione

Nelle Blazor WebAssembly app è possibile ignorare i controlli di autorizzazione perché tutto il codice lato client può essere modificato dagli utenti. Lo stesso vale per tutte le tecnologie per app sul lato client, tra cui i framework JavaScript SPA o le app native per qualsiasi sistema operativo.

**Eseguire sempre i controlli di autorizzazione nel server all'interno degli eventuali endpoint dell'API a cui accede l'app sul lato client.**

## <a name="require-authorization-for-the-entire-app"></a>Richiedi l'autorizzazione per l'intera app

Applicare l' [ `[Authorize]` attributo](xref:blazor/security/index#authorize-attribute) ([documentazione API](xref:System.Web.Mvc.AuthorizeAttribute)) a ogni Razor componente dell'app usando uno degli approcci seguenti:

* Usare la [`@attribute`](xref:mvc/views/razor#attribute) direttiva nel `_Imports.razor` file:

  ```razor
  @using Microsoft.AspNetCore.Authorization
  @attribute [Authorize]
  ```

* Aggiungere l'attributo a ogni Razor componente nella `Pages` cartella.

> [!NOTE]
> L'impostazione <xref:Microsoft.AspNetCore.Authorization.AuthorizationOptions.FallbackPolicy?displayProperty=nameWithType> di un oggetto su un criterio con <xref:Microsoft.AspNetCore.Authorization.AuthorizationPolicyBuilder.RequireAuthenticatedUser%2A> **non** è supportata.

## <a name="refresh-tokens"></a>Token di aggiornamento

I token di aggiornamento non possono essere protetti sul lato client nelle Blazor WebAssembly app. Pertanto, i token di aggiornamento non devono essere inviati all'app per l'uso diretto.

I token di aggiornamento possono essere gestiti e usati dall'app sul lato server in una soluzione ospitata Blazor WebAssembly per accedere alle API di terze parti. Per altre informazioni, vedere <xref:blazor/security/webassembly/additional-scenarios#authenticate-users-with-a-third-party-provider-and-call-protected-apis-on-the-host-server-and-the-third-party>.

## <a name="establish-claims-for-users"></a>Stabilire attestazioni per gli utenti

Le app richiedono spesso attestazioni per gli utenti in base a una chiamata API Web a un server. Ad esempio, le attestazioni vengono spesso usate per [definire l'autorizzazione](xref:blazor/security/index#authorization) in un'app. In questi scenari, l'app richiede un token di accesso per accedere al servizio e usa il token per ottenere i dati utente per le attestazioni. Per esempi, vedere le risorse seguenti:

* [Scenari aggiuntivi: personalizzare l'utente](xref:blazor/security/webassembly/additional-scenarios#customize-the-user)
* <xref:blazor/security/webassembly/aad-groups-roles>

## <a name="azure-app-service-on-linux-with-identity-server"></a>Servizio app Azure in Linux con Identity Server

Specificare l'autorità emittente in modo esplicito quando si distribuisce in app Azure servizio in Linux con Identity Server. Per altre informazioni, vedere <xref:security/authentication/identity/spa#azure-app-service-on-linux>.

## <a name="windows-authentication"></a>Autenticazione di Windows

Non è consigliabile usare l'autenticazione di Windows con Blazor webassembly o con altri Framework di applicazione a singola pagina. È consigliabile usare protocolli basati su token anziché l'autenticazione di Windows, ad esempio OIDC con Active Directory Federation Services (ADFS).

Se l'autenticazione di Windows viene usata con Blazor webassembly o con qualsiasi altro framework di applicazione a singola pagina, sono necessarie misure aggiuntive per proteggere l'app dai token CSRF (cross-site request false). Le stesse considerazioni che si applicano a cookie s si applicano all'autenticazione di Windows con l'aggiunta che l'autenticazione di Windows non offre alcun meccanismo per impedire la condivisione del contesto di autenticazione tra le origini. Le app che usano l'autenticazione di Windows senza protezione aggiuntiva da CSRF devono essere limitate almeno alla Intranet di un'organizzazione e non possono essere usate in Internet.

Per informazioni, vedere <xref:security/anti-request-forgery> .

## <a name="implementation-guidance"></a>Indicazioni relative all'implementazione

Gli articoli in questa *Panoramica* forniscono informazioni sull'autenticazione degli utenti nelle Blazor WebAssembly app per provider specifici.

App autonome Blazor WebAssembly :

* [Linee guida generali per i provider OIDC e la libreria di autenticazione webassembly](xref:blazor/security/webassembly/standalone-with-authentication-library)
* [Account Microsoft](xref:blazor/security/webassembly/standalone-with-microsoft-accounts)
* [Azure Active Directory (AAD)](xref:blazor/security/webassembly/standalone-with-azure-active-directory)
* [Azure Active Directory (AAD) B2C](xref:blazor/security/webassembly/standalone-with-azure-active-directory-b2c)

App ospitate Blazor WebAssembly :

* [Azure Active Directory (AAD)](xref:blazor/security/webassembly/hosted-with-azure-active-directory)
* [Azure Active Directory (AAD) B2C](xref:blazor/security/webassembly/hosted-with-azure-active-directory-b2c)
* [Identity Server](xref:blazor/security/webassembly/hosted-with-identity-server)

::: moniker range=">= aspnetcore-5.0"

Altre informazioni aggiuntive sulla configurazione sono disponibili negli articoli seguenti:

* <xref:blazor/security/webassembly/additional-scenarios>
* <xref:blazor/security/webassembly/graph-api>

::: moniker-end

::: moniker range="< aspnetcore-5.0"

Per ulteriori informazioni sulla configurazione, vedere <xref:blazor/security/webassembly/additional-scenarios> .

::: moniker-end

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:host-and-deploy/proxy-load-balancer>: Include informazioni aggiuntive su:
  * Uso del middleware intestazioni con inoltro per mantenere le informazioni sullo schema HTTPS tra i server proxy e le reti interne.
  * Scenari e casi d'uso aggiuntivi, tra cui la configurazione manuale dello schema, la modifica del percorso della richiesta per il routing delle richieste corretto e l'inoltro dello schema di richiesta per i proxy inversi Linux e non IIS.
