---
title: Impedisci gli attacchi di richiesta intersito falsa (XSRF/CSRF) in ASP.NET Core
author: steve-smith
description: Scopri come impedire gli attacchi contro le app Web in cui un sito Web dannoso può influenzare l'interazione tra un browser client e l'app.
ms.author: riande
ms.custom: mvc, devx-track-js
ms.date: 12/05/2019
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
uid: security/anti-request-forgery
ms.openlocfilehash: 3bb3c059eafa8e948fe2e719207927c009902e59
ms.sourcegitcommit: 83524f739dd25fbfa95ee34e95342afb383b49fe
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 01/29/2021
ms.locfileid: "99057447"
---
# <a name="prevent-cross-site-request-forgery-xsrfcsrf-attacks-in-aspnet-core"></a><span data-ttu-id="d8c62-103">Impedisci gli attacchi di richiesta intersito falsa (XSRF/CSRF) in ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="d8c62-103">Prevent Cross-Site Request Forgery (XSRF/CSRF) attacks in ASP.NET Core</span></span>

<span data-ttu-id="d8c62-104">Di [Rick Anderson](https://twitter.com/RickAndMSFT), [Fiyaz Hasan](https://twitter.com/FiyazBinHasan)e [Steve Smith](https://ardalis.com/)</span><span class="sxs-lookup"><span data-stu-id="d8c62-104">By [Rick Anderson](https://twitter.com/RickAndMSFT), [Fiyaz Hasan](https://twitter.com/FiyazBinHasan), and [Steve Smith](https://ardalis.com/)</span></span>

<span data-ttu-id="d8c62-105">La richiesta tra siti falsificata (nota anche come XSRF o CSRF) è un attacco contro app ospitate sul Web, in cui un'app Web dannosa può influenzare l'interazione tra un browser client e un'app Web che considera attendibile il browser.</span><span class="sxs-lookup"><span data-stu-id="d8c62-105">Cross-site request forgery (also known as XSRF or CSRF) is an attack against web-hosted apps whereby a malicious web app can influence the interaction between a client browser and a web app that trusts that browser.</span></span> <span data-ttu-id="d8c62-106">Questi attacchi sono possibili perché i Web browser inviano automaticamente alcuni tipi di token di autenticazione con ogni richiesta a un sito Web.</span><span class="sxs-lookup"><span data-stu-id="d8c62-106">These attacks are possible because web browsers send some types of authentication tokens automatically with every request to a website.</span></span> <span data-ttu-id="d8c62-107">Questa forma di exploit è nota anche come un *attacco con un solo clic* o una *sessione di guida* perché l'attacco sfrutta la sessione precedentemente autenticata dell'utente.</span><span class="sxs-lookup"><span data-stu-id="d8c62-107">This form of exploit is also known as a *one-click attack* or *session riding* because the attack takes advantage of the user's previously authenticated session.</span></span>

<span data-ttu-id="d8c62-108">Esempio di attacco CSRF:</span><span class="sxs-lookup"><span data-stu-id="d8c62-108">An example of a CSRF attack:</span></span>

1. <span data-ttu-id="d8c62-109">Un utente accede `www.good-banking-site.com` utilizzando l'autenticazione basata su form.</span><span class="sxs-lookup"><span data-stu-id="d8c62-109">A user signs into `www.good-banking-site.com` using forms authentication.</span></span> <span data-ttu-id="d8c62-110">Il server autentica l'utente e genera una risposta che include un'autenticazione cookie .</span><span class="sxs-lookup"><span data-stu-id="d8c62-110">The server authenticates the user and issues a response that includes an authentication cookie.</span></span> <span data-ttu-id="d8c62-111">Il sito è vulnerabile agli attacchi poiché considera attendibile qualsiasi richiesta ricevuta con un'autenticazione valida cookie .</span><span class="sxs-lookup"><span data-stu-id="d8c62-111">The site is vulnerable to attack because it trusts any request that it receives with a valid authentication cookie.</span></span>
1. <span data-ttu-id="d8c62-112">L'utente visita un sito dannoso, `www.bad-crook-site.com` .</span><span class="sxs-lookup"><span data-stu-id="d8c62-112">The user visits a malicious site, `www.bad-crook-site.com`.</span></span>

   <span data-ttu-id="d8c62-113">Il sito dannoso, `www.bad-crook-site.com` , contiene un form HTML simile al seguente:</span><span class="sxs-lookup"><span data-stu-id="d8c62-113">The malicious site, `www.bad-crook-site.com`, contains an HTML form similar to the following:</span></span>

   ```html
   <h1>Congratulations! You're a Winner!</h1>
   <form action="http://good-banking-site.com/api/account" method="post">
       <input type="hidden" name="Transaction" value="withdraw">
       <input type="hidden" name="Amount" value="1000000">
       <input type="submit" value="Click to collect your prize!">
   </form>
   ```

   <span data-ttu-id="d8c62-114">Si noti che il post del modulo sul `action` sito vulnerabile, non sul sito dannoso.</span><span class="sxs-lookup"><span data-stu-id="d8c62-114">Notice that the form's `action` posts to the vulnerable site, not to the malicious site.</span></span> <span data-ttu-id="d8c62-115">Si tratta della parte "cross-site" di CSRF.</span><span class="sxs-lookup"><span data-stu-id="d8c62-115">This is the "cross-site" part of CSRF.</span></span>

1. <span data-ttu-id="d8c62-116">L'utente seleziona il pulsante Submit (Invia).</span><span class="sxs-lookup"><span data-stu-id="d8c62-116">The user selects the submit button.</span></span> <span data-ttu-id="d8c62-117">Il browser esegue la richiesta e include automaticamente l'autenticazione cookie per il dominio richiesto, `www.good-banking-site.com` .</span><span class="sxs-lookup"><span data-stu-id="d8c62-117">The browser makes the request and automatically includes the authentication cookie for the requested domain, `www.good-banking-site.com`.</span></span>
1. <span data-ttu-id="d8c62-118">La richiesta viene eseguita sul `www.good-banking-site.com` server con il contesto di autenticazione dell'utente e può eseguire qualsiasi azione che un utente autenticato è autorizzato a eseguire.</span><span class="sxs-lookup"><span data-stu-id="d8c62-118">The request runs on the `www.good-banking-site.com` server with the user's authentication context and can perform any action that an authenticated user is allowed to perform.</span></span>

<span data-ttu-id="d8c62-119">Oltre allo scenario in cui l'utente seleziona il pulsante per l'invio del modulo, il sito dannoso potrebbe:</span><span class="sxs-lookup"><span data-stu-id="d8c62-119">In addition to the scenario where the user selects the button to submit the form, the malicious site could:</span></span>

* <span data-ttu-id="d8c62-120">Eseguire uno script che invia automaticamente il modulo.</span><span class="sxs-lookup"><span data-stu-id="d8c62-120">Run a script that automatically submits the form.</span></span>
* <span data-ttu-id="d8c62-121">Inviare il modulo inviato come richiesta AJAX.</span><span class="sxs-lookup"><span data-stu-id="d8c62-121">Send the form submission as an AJAX request.</span></span>
* <span data-ttu-id="d8c62-122">Nascondere il form usando CSS.</span><span class="sxs-lookup"><span data-stu-id="d8c62-122">Hide the form using CSS.</span></span>

<span data-ttu-id="d8c62-123">Questi scenari alternativi non richiedono alcuna azione o input da parte dell'utente, tranne che inizialmente visitare il sito dannoso.</span><span class="sxs-lookup"><span data-stu-id="d8c62-123">These alternative scenarios don't require any action or input from the user other than initially visiting the malicious site.</span></span>

<span data-ttu-id="d8c62-124">L'uso di HTTPS non impedisce un attacco CSRF.</span><span class="sxs-lookup"><span data-stu-id="d8c62-124">Using HTTPS doesn't prevent a CSRF attack.</span></span> <span data-ttu-id="d8c62-125">Il sito dannoso può inviare una richiesta con la stessa `https://www.good-banking-site.com/` facilità con cui è possibile inviare una richiesta non protetta.</span><span class="sxs-lookup"><span data-stu-id="d8c62-125">The malicious site can send an `https://www.good-banking-site.com/` request just as easily as it can send an insecure request.</span></span>

<span data-ttu-id="d8c62-126">Alcuni attacchi hanno come destinazione endpoint che rispondono alle richieste GET, nel qual caso è possibile usare un tag di immagine per eseguire l'azione.</span><span class="sxs-lookup"><span data-stu-id="d8c62-126">Some attacks target endpoints that respond to GET requests, in which case an image tag can be used to perform the action.</span></span> <span data-ttu-id="d8c62-127">Questo tipo di attacco è comune nei siti dei forum che consentono le immagini ma bloccano JavaScript.</span><span class="sxs-lookup"><span data-stu-id="d8c62-127">This form of attack is common on forum sites that permit images but block JavaScript.</span></span> <span data-ttu-id="d8c62-128">Le app che modificano lo stato di richieste GET, in cui variabili o risorse vengono modificate, sono vulnerabili ad attacchi dannosi.</span><span class="sxs-lookup"><span data-stu-id="d8c62-128">Apps that change state on GET requests, where variables or resources are altered, are vulnerable to malicious attacks.</span></span> <span data-ttu-id="d8c62-129">**Le richieste GET che cambiano lo stato non sono sicure. Una procedura consigliata consiste nel non modificare mai lo stato di una richiesta GET.**</span><span class="sxs-lookup"><span data-stu-id="d8c62-129">**GET requests that change state are insecure. A best practice is to never change state on a GET request.**</span></span>

<span data-ttu-id="d8c62-130">Gli attacchi CSRF sono possibili per le app Web che usano cookie i per l'autenticazione perché:</span><span class="sxs-lookup"><span data-stu-id="d8c62-130">CSRF attacks are possible against web apps that use cookies for authentication because:</span></span>

* <span data-ttu-id="d8c62-131">I browser archiviano cookie i rilasciati da un'app Web.</span><span class="sxs-lookup"><span data-stu-id="d8c62-131">Browsers store cookies issued by a web app.</span></span>
* <span data-ttu-id="d8c62-132">cookieGli archivi contengono cookie la sessione s per gli utenti autenticati.</span><span class="sxs-lookup"><span data-stu-id="d8c62-132">Stored cookies include session cookies for authenticated users.</span></span>
* <span data-ttu-id="d8c62-133">I browser inviano tutti gli oggetti cookie associati a un dominio all'app Web ogni richiesta, indipendentemente dalla modalità di generazione della richiesta all'app all'interno del browser.</span><span class="sxs-lookup"><span data-stu-id="d8c62-133">Browsers send all of the cookies associated with a domain to the web app every request regardless of how the request to app was generated within the browser.</span></span>

<span data-ttu-id="d8c62-134">Tuttavia, gli attacchi CSRF non sono limitati allo sfruttamento di cookie .</span><span class="sxs-lookup"><span data-stu-id="d8c62-134">However, CSRF attacks aren't limited to exploiting cookies.</span></span> <span data-ttu-id="d8c62-135">Ad esempio, anche l'autenticazione di base e del digest è vulnerabile.</span><span class="sxs-lookup"><span data-stu-id="d8c62-135">For example, Basic and Digest authentication are also vulnerable.</span></span> <span data-ttu-id="d8c62-136">Quando un utente accede con l'autenticazione di base o del digest, il browser invia automaticamente le credenziali fino alla scadenza della sessione &dagger; .</span><span class="sxs-lookup"><span data-stu-id="d8c62-136">After a user signs in with Basic or Digest authentication, the browser automatically sends the credentials until the session&dagger; ends.</span></span>

<span data-ttu-id="d8c62-137">&dagger;In questo contesto, *Session* fa riferimento alla sessione lato client durante la quale l'utente viene autenticato.</span><span class="sxs-lookup"><span data-stu-id="d8c62-137">&dagger;In this context, *session* refers to the client-side session during which the user is authenticated.</span></span> <span data-ttu-id="d8c62-138">Non è correlato alle sessioni sul lato server o al [middleware della sessione ASP.NET Core](xref:fundamentals/app-state).</span><span class="sxs-lookup"><span data-stu-id="d8c62-138">It's unrelated to server-side sessions or [ASP.NET Core Session Middleware](xref:fundamentals/app-state).</span></span>

<span data-ttu-id="d8c62-139">Gli utenti possono proteggersi da vulnerabilità CSRF adottando le precauzioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="d8c62-139">Users can guard against CSRF vulnerabilities by taking precautions:</span></span>

* <span data-ttu-id="d8c62-140">Disconnettersi dalle app Web al termine dell'uso.</span><span class="sxs-lookup"><span data-stu-id="d8c62-140">Sign off of web apps when finished using them.</span></span>
* <span data-ttu-id="d8c62-141">Cancellare cookie periodicamente i browser.</span><span class="sxs-lookup"><span data-stu-id="d8c62-141">Clear browser cookies periodically.</span></span>

<span data-ttu-id="d8c62-142">Tuttavia, le vulnerabilità di CSRF sono fondamentalmente un problema con l'app Web e non con l'utente finale.</span><span class="sxs-lookup"><span data-stu-id="d8c62-142">However, CSRF vulnerabilities are fundamentally a problem with the web app, not the end user.</span></span>

## <a name="authentication-fundamentals"></a><span data-ttu-id="d8c62-143">Nozioni fondamentali di autenticazione</span><span class="sxs-lookup"><span data-stu-id="d8c62-143">Authentication fundamentals</span></span>

<span data-ttu-id="d8c62-144">Cookiel'autenticazione basata su è una forma comune di autenticazione.</span><span class="sxs-lookup"><span data-stu-id="d8c62-144">Cookie-based authentication is a popular form of authentication.</span></span> <span data-ttu-id="d8c62-145">I sistemi di autenticazione basata su token hanno una maggiore popolarità, soprattutto per le applicazioni a pagina singola (Spa).</span><span class="sxs-lookup"><span data-stu-id="d8c62-145">Token-based authentication systems are growing in popularity, especially for Single Page Applications (SPAs).</span></span>

### <a name="no-loccookie-based-authentication"></a><span data-ttu-id="d8c62-146">Cookieautenticazione basata su</span><span class="sxs-lookup"><span data-stu-id="d8c62-146">Cookie-based authentication</span></span>

<span data-ttu-id="d8c62-147">Quando un utente esegue l'autenticazione usando il nome utente e la password, viene emesso un token contenente un ticket di autenticazione che può essere usato per l'autenticazione e l'autorizzazione.</span><span class="sxs-lookup"><span data-stu-id="d8c62-147">When a user authenticates using their username and password, they're issued a token, containing an authentication ticket that can be used for authentication and authorization.</span></span> <span data-ttu-id="d8c62-148">Il token viene archiviato come oggetto cookie che accompagna ogni richiesta eseguita dal client.</span><span class="sxs-lookup"><span data-stu-id="d8c62-148">The token is stored as a cookie that accompanies every request the client makes.</span></span> <span data-ttu-id="d8c62-149">La generazione e cookie la convalida di questo oggetto viene eseguita dal Cookie middleware di autenticazione.</span><span class="sxs-lookup"><span data-stu-id="d8c62-149">Generating and validating this cookie is performed by the Cookie Authentication Middleware.</span></span> <span data-ttu-id="d8c62-150">Il [middleware](xref:fundamentals/middleware/index) serializza un'entità utente in un oggetto crittografato cookie .</span><span class="sxs-lookup"><span data-stu-id="d8c62-150">The [middleware](xref:fundamentals/middleware/index) serializes a user principal into an encrypted cookie.</span></span> <span data-ttu-id="d8c62-151">Nelle richieste successive il middleware convalida l'oggetto cookie , ricrea l'entità e assegna l'entità alla proprietà [User](/dotnet/api/microsoft.aspnetcore.http.httpcontext.user) di [HttpContext](/dotnet/api/microsoft.aspnetcore.http.httpcontext).</span><span class="sxs-lookup"><span data-stu-id="d8c62-151">On subsequent requests, the middleware validates the cookie, recreates the principal, and assigns the principal to the [User](/dotnet/api/microsoft.aspnetcore.http.httpcontext.user) property of [HttpContext](/dotnet/api/microsoft.aspnetcore.http.httpcontext).</span></span>

### <a name="token-based-authentication"></a><span data-ttu-id="d8c62-152">Autenticazione basata su token</span><span class="sxs-lookup"><span data-stu-id="d8c62-152">Token-based authentication</span></span>

<span data-ttu-id="d8c62-153">Quando un utente viene autenticato, viene emesso un token (non un token antifalsificazione).</span><span class="sxs-lookup"><span data-stu-id="d8c62-153">When a user is authenticated, they're issued a token (not an antiforgery token).</span></span> <span data-ttu-id="d8c62-154">Il token contiene informazioni sull'utente sotto forma di [attestazioni](/dotnet/framework/security/claims-based-identity-model) o un token di riferimento che punta l'app allo stato utente mantenuto nell'app.</span><span class="sxs-lookup"><span data-stu-id="d8c62-154">The token contains user information in the form of [claims](/dotnet/framework/security/claims-based-identity-model) or a reference token that points the app to user state maintained in the app.</span></span> <span data-ttu-id="d8c62-155">Quando un utente tenta di accedere a una risorsa che richiede l'autenticazione, il token viene inviato all'app con un'intestazione di autorizzazione aggiuntiva nel formato token di connessione.</span><span class="sxs-lookup"><span data-stu-id="d8c62-155">When a user attempts to access a resource requiring authentication, the token is sent to the app with an additional authorization header in form of Bearer token.</span></span> <span data-ttu-id="d8c62-156">Questa operazione rende l'app senza stato.</span><span class="sxs-lookup"><span data-stu-id="d8c62-156">This makes the app stateless.</span></span> <span data-ttu-id="d8c62-157">In ogni richiesta successiva il token viene passato nella richiesta di convalida lato server.</span><span class="sxs-lookup"><span data-stu-id="d8c62-157">In each subsequent request, the token is passed in the request for server-side validation.</span></span> <span data-ttu-id="d8c62-158">Questo token non è *crittografato*; è *codificato*.</span><span class="sxs-lookup"><span data-stu-id="d8c62-158">This token isn't *encrypted*; it's *encoded*.</span></span> <span data-ttu-id="d8c62-159">Sul server, il token viene decodificato per accedere alle informazioni.</span><span class="sxs-lookup"><span data-stu-id="d8c62-159">On the server, the token is decoded to access its information.</span></span> <span data-ttu-id="d8c62-160">Per inviare il token alle richieste successive, archiviare il token nella risorsa di archiviazione locale del browser.</span><span class="sxs-lookup"><span data-stu-id="d8c62-160">To send the token on subsequent requests, store the token in the browser's local storage.</span></span> <span data-ttu-id="d8c62-161">Non preoccuparsi della vulnerabilità CSRF se il token è archiviato nella risorsa di archiviazione locale del browser.</span><span class="sxs-lookup"><span data-stu-id="d8c62-161">Don't be concerned about CSRF vulnerability if the token is stored in the browser's local storage.</span></span> <span data-ttu-id="d8c62-162">CSRF rappresenta un problema quando il token viene archiviato in un oggetto cookie .</span><span class="sxs-lookup"><span data-stu-id="d8c62-162">CSRF is a concern when the token is stored in a cookie.</span></span> <span data-ttu-id="d8c62-163">Per ulteriori informazioni, vedere l'esempio di codice di GitHub Issue [Spa cookie aggiunge due](https://github.com/dotnet/AspNetCore.Docs/issues/13369).</span><span class="sxs-lookup"><span data-stu-id="d8c62-163">For more information, see the GitHub issue [SPA code sample adds two cookies](https://github.com/dotnet/AspNetCore.Docs/issues/13369).</span></span>

### <a name="multiple-apps-hosted-at-one-domain"></a><span data-ttu-id="d8c62-164">Più app ospitate in un dominio</span><span class="sxs-lookup"><span data-stu-id="d8c62-164">Multiple apps hosted at one domain</span></span>

<span data-ttu-id="d8c62-165">Gli ambienti di hosting condiviso sono vulnerabili al Hijack della sessione, all'account di accesso CSRF e ad altri attacchi.</span><span class="sxs-lookup"><span data-stu-id="d8c62-165">Shared hosting environments are vulnerable to session hijacking, login CSRF, and other attacks.</span></span>

<span data-ttu-id="d8c62-166">Sebbene `example1.contoso.net` e `example2.contoso.net` siano host diversi, esiste una relazione di trust implicita tra gli host nel `*.contoso.net` dominio.</span><span class="sxs-lookup"><span data-stu-id="d8c62-166">Although `example1.contoso.net` and `example2.contoso.net` are different hosts, there's an implicit trust relationship between hosts under the `*.contoso.net` domain.</span></span> <span data-ttu-id="d8c62-167">Questa relazione di trust implicita consente a host potenzialmente non attendibili di influenzare gli altri cookie (i criteri di stessa origine che regolano le richieste AJAX non si applicano necessariamente a http cookie s).</span><span class="sxs-lookup"><span data-stu-id="d8c62-167">This implicit trust relationship allows potentially untrusted hosts to affect each other's cookies (the same-origin policies that govern AJAX requests don't necessarily apply to HTTP cookies).</span></span>

<span data-ttu-id="d8c62-168">Gli attacchi che sfruttano cookie i trust tra le app ospitate nello stesso dominio possono essere impediti dalla condivisione dei domini.</span><span class="sxs-lookup"><span data-stu-id="d8c62-168">Attacks that exploit trusted cookies between apps hosted on the same domain can be prevented by not sharing domains.</span></span> <span data-ttu-id="d8c62-169">Quando ogni app è ospitata nel proprio dominio, non esiste alcuna cookie relazione di trust implicita da sfruttare.</span><span class="sxs-lookup"><span data-stu-id="d8c62-169">When each app is hosted on its own domain, there is no implicit cookie trust relationship to exploit.</span></span>

## <a name="aspnet-core-antiforgery-configuration"></a><span data-ttu-id="d8c62-170">ASP.NET Core configurazione antifalsificazione</span><span class="sxs-lookup"><span data-stu-id="d8c62-170">ASP.NET Core antiforgery configuration</span></span>

> [!WARNING]
> <span data-ttu-id="d8c62-171">ASP.NET Core implementa la antifalsificazione usando la [protezione dei dati ASP.NET Core](xref:security/data-protection/introduction).</span><span class="sxs-lookup"><span data-stu-id="d8c62-171">ASP.NET Core implements antiforgery using [ASP.NET Core Data Protection](xref:security/data-protection/introduction).</span></span> <span data-ttu-id="d8c62-172">Lo stack di protezione dei dati deve essere configurato in modo da funzionare in un server farm.</span><span class="sxs-lookup"><span data-stu-id="d8c62-172">The data protection stack must be configured to work in a server farm.</span></span> <span data-ttu-id="d8c62-173">Per ulteriori informazioni, vedere [configurazione della protezione dei dati](xref:security/data-protection/configuration/overview) .</span><span class="sxs-lookup"><span data-stu-id="d8c62-173">See [Configuring data protection](xref:security/data-protection/configuration/overview) for more information.</span></span>

::: moniker range=">= aspnetcore-3.0"

<span data-ttu-id="d8c62-174">Il middleware antifalsificazione viene aggiunto al contenitore di [inserimento delle dipendenze](xref:fundamentals/dependency-injection) quando viene chiamata una delle API seguenti in `Startup.ConfigureServices` :</span><span class="sxs-lookup"><span data-stu-id="d8c62-174">Antiforgery middleware is added to the [Dependency injection](xref:fundamentals/dependency-injection) container when one of the following APIs is called in `Startup.ConfigureServices`:</span></span>

* <xref:Microsoft.Extensions.DependencyInjection.MvcServiceCollectionExtensions.AddMvc*>
* <xref:Microsoft.AspNetCore.Builder.RazorPagesEndpointRouteBuilderExtensions.MapRazorPages*>
* <xref:Microsoft.AspNetCore.Builder.ControllerEndpointRouteBuilderExtensions.MapControllerRoute*>
* <xref:Microsoft.AspNetCore.Builder.ComponentEndpointRouteBuilderExtensions.MapBlazorHub*>

::: moniker-end

::: moniker range="< aspnetcore-3.0"

<span data-ttu-id="d8c62-175">Il middleware antifalsificazione viene aggiunto al contenitore di [inserimento delle dipendenze](xref:fundamentals/dependency-injection) quando <xref:Microsoft.Extensions.DependencyInjection.MvcServiceCollectionExtensions.AddMvc*> viene chiamato in `Startup.ConfigureServices`</span><span class="sxs-lookup"><span data-stu-id="d8c62-175">Antiforgery middleware is added to the [Dependency injection](xref:fundamentals/dependency-injection) container when <xref:Microsoft.Extensions.DependencyInjection.MvcServiceCollectionExtensions.AddMvc*> is called in `Startup.ConfigureServices`</span></span>

::: moniker-end

<span data-ttu-id="d8c62-176">In ASP.NET Core 2,0 o versioni successive, [FormTagHelper](xref:mvc/views/working-with-forms#the-form-tag-helper) inserisce i token antifalsificazione negli elementi del form HTML.</span><span class="sxs-lookup"><span data-stu-id="d8c62-176">In ASP.NET Core 2.0 or later, the [FormTagHelper](xref:mvc/views/working-with-forms#the-form-tag-helper) injects antiforgery tokens into HTML form elements.</span></span> <span data-ttu-id="d8c62-177">Il markup seguente in un Razor file genera automaticamente i token antifalsificazione:</span><span class="sxs-lookup"><span data-stu-id="d8c62-177">The following markup in a Razor file automatically generates antiforgery tokens:</span></span>

```cshtml
<form method="post">
    ...
</form>
```

<span data-ttu-id="d8c62-178">Analogamente, [IHtmlHelper. BeginForm](/dotnet/api/microsoft.aspnetcore.mvc.rendering.ihtmlhelper.beginform) genera token antifalsificazione per impostazione predefinita se il metodo del modulo non è Get.</span><span class="sxs-lookup"><span data-stu-id="d8c62-178">Similarly, [IHtmlHelper.BeginForm](/dotnet/api/microsoft.aspnetcore.mvc.rendering.ihtmlhelper.beginform) generates antiforgery tokens by default if the form's method isn't GET.</span></span>

<span data-ttu-id="d8c62-179">La generazione automatica dei token antifalsificazione per gli elementi del form HTML si verifica quando il `<form>` tag contiene l' `method="post"` attributo e una delle condizioni seguenti è vera:</span><span class="sxs-lookup"><span data-stu-id="d8c62-179">The automatic generation of antiforgery tokens for HTML form elements happens when the `<form>` tag contains the `method="post"` attribute and either of the following are true:</span></span>

* <span data-ttu-id="d8c62-180">L'attributo Action è vuoto ( `action=""` ).</span><span class="sxs-lookup"><span data-stu-id="d8c62-180">The action attribute is empty (`action=""`).</span></span>
* <span data-ttu-id="d8c62-181">L'attributo Action non è specificato ( `<form method="post">` ).</span><span class="sxs-lookup"><span data-stu-id="d8c62-181">The action attribute isn't supplied (`<form method="post">`).</span></span>

<span data-ttu-id="d8c62-182">La generazione automatica dei token antifalsificazione per gli elementi del modulo HTML può essere disabilitata:</span><span class="sxs-lookup"><span data-stu-id="d8c62-182">Automatic generation of antiforgery tokens for HTML form elements can be disabled:</span></span>

* <span data-ttu-id="d8c62-183">Disabilitare in modo esplicito i token antifalsificazione con l' `asp-antiforgery` attributo:</span><span class="sxs-lookup"><span data-stu-id="d8c62-183">Explicitly disable antiforgery tokens with the `asp-antiforgery` attribute:</span></span>

  ```cshtml
  <form method="post" asp-antiforgery="false">
      ...
  </form>
  ```

* <span data-ttu-id="d8c62-184">L'elemento form è escluso dagli Helper tag usando il simbolo Helper tag [! opt-out](xref:mvc/views/tag-helpers/intro#opt-out):</span><span class="sxs-lookup"><span data-stu-id="d8c62-184">The form element is opted-out of Tag Helpers by using the Tag Helper [! opt-out symbol](xref:mvc/views/tag-helpers/intro#opt-out):</span></span>

  ```cshtml
  <!form method="post">
      ...
  </!form>
  ```

* <span data-ttu-id="d8c62-185">Rimuovere `FormTagHelper` dalla visualizzazione.</span><span class="sxs-lookup"><span data-stu-id="d8c62-185">Remove the `FormTagHelper` from the view.</span></span> <span data-ttu-id="d8c62-186">`FormTagHelper`È possibile rimuovere l'oggetto da una visualizzazione aggiungendo la direttiva seguente alla Razor vista:</span><span class="sxs-lookup"><span data-stu-id="d8c62-186">The `FormTagHelper` can be removed from a view by adding the following directive to the Razor view:</span></span>

  ```cshtml
  @removeTagHelper Microsoft.AspNetCore.Mvc.TagHelpers.FormTagHelper, Microsoft.AspNetCore.Mvc.TagHelpers
  ```

> [!NOTE]
> <span data-ttu-id="d8c62-187">Le [ Razor pagine](xref:razor-pages/index) vengono protette automaticamente da XSRF/CSRF.</span><span class="sxs-lookup"><span data-stu-id="d8c62-187">[Razor Pages](xref:razor-pages/index) are automatically protected from XSRF/CSRF.</span></span> <span data-ttu-id="d8c62-188">Per ulteriori informazioni, vedere [XSRF/CSRF e Razor pages](xref:razor-pages/index#xsrf).</span><span class="sxs-lookup"><span data-stu-id="d8c62-188">For more information, see [XSRF/CSRF and Razor Pages](xref:razor-pages/index#xsrf).</span></span>

<span data-ttu-id="d8c62-189">L'approccio più comune alla difesa dagli attacchi CSRF consiste nell'usare il *modello di token di sincronizzazione* (STP).</span><span class="sxs-lookup"><span data-stu-id="d8c62-189">The most common approach to defending against CSRF attacks is to use the *Synchronizer Token Pattern* (STP).</span></span> <span data-ttu-id="d8c62-190">STP viene usato quando l'utente richiede una pagina con dati del modulo:</span><span class="sxs-lookup"><span data-stu-id="d8c62-190">STP is used when the user requests a page with form data:</span></span>

1. <span data-ttu-id="d8c62-191">Il server invia al client un token associato all'identità dell'utente corrente.</span><span class="sxs-lookup"><span data-stu-id="d8c62-191">The server sends a token associated with the current user's identity to the client.</span></span>
1. <span data-ttu-id="d8c62-192">Il client invia il token al server per la verifica.</span><span class="sxs-lookup"><span data-stu-id="d8c62-192">The client sends back the token to the server for verification.</span></span>
1. <span data-ttu-id="d8c62-193">Se il server riceve un token che non corrisponde all'identità dell'utente autenticato, la richiesta viene rifiutata.</span><span class="sxs-lookup"><span data-stu-id="d8c62-193">If the server receives a token that doesn't match the authenticated user's identity, the request is rejected.</span></span>

<span data-ttu-id="d8c62-194">Il token è univoco e imprevedibile.</span><span class="sxs-lookup"><span data-stu-id="d8c62-194">The token is unique and unpredictable.</span></span> <span data-ttu-id="d8c62-195">Il token può essere usato anche per garantire la sequenziazione corretta di una serie di richieste, ad esempio per garantire la sequenza di richiesta: pagina 1 > pagina 2 > pagina 3).</span><span class="sxs-lookup"><span data-stu-id="d8c62-195">The token can also be used to ensure proper sequencing of a series of requests (for example, ensuring the request sequence of: page 1 > page 2 > page 3).</span></span> <span data-ttu-id="d8c62-196">Tutti i moduli in ASP.NET Core modelli MVC e di Razor pagine generano token antifalsificazione.</span><span class="sxs-lookup"><span data-stu-id="d8c62-196">All of the forms in ASP.NET Core MVC and Razor Pages templates generate antiforgery tokens.</span></span> <span data-ttu-id="d8c62-197">La coppia seguente di esempi di viste genera i token antifalsificazione:</span><span class="sxs-lookup"><span data-stu-id="d8c62-197">The following pair of view examples generate antiforgery tokens:</span></span>

```cshtml
<form asp-controller="Manage" asp-action="ChangePassword" method="post">
    ...
</form>

@using (Html.BeginForm("ChangePassword", "Manage"))
{
    ...
}
```

<span data-ttu-id="d8c62-198">Aggiungere in modo esplicito un token antifalsificazione a un `<form>` elemento senza usare gli helper tag con l'helper HTML [`@Html.AntiForgeryToken`](/dotnet/api/microsoft.aspnetcore.mvc.viewfeatures.htmlhelper.antiforgerytoken) :</span><span class="sxs-lookup"><span data-stu-id="d8c62-198">Explicitly add an antiforgery token to a `<form>` element without using Tag Helpers with the HTML helper [`@Html.AntiForgeryToken`](/dotnet/api/microsoft.aspnetcore.mvc.viewfeatures.htmlhelper.antiforgerytoken):</span></span>

```cshtml
<form action="/" method="post">
    @Html.AntiForgeryToken()
</form>
```

<span data-ttu-id="d8c62-199">In ognuno dei casi precedenti, ASP.NET Core aggiunge un campo del form nascosto simile al seguente:</span><span class="sxs-lookup"><span data-stu-id="d8c62-199">In each of the preceding cases, ASP.NET Core adds a hidden form field similar to the following:</span></span>

```cshtml
<input name="__RequestVerificationToken" type="hidden" value="CfDJ8NrAkS ... s2-m9Yw">
```

<span data-ttu-id="d8c62-200">ASP.NET Core include tre [filtri](xref:mvc/controllers/filters) per l'utilizzo di token antifalsificazione:</span><span class="sxs-lookup"><span data-stu-id="d8c62-200">ASP.NET Core includes three [filters](xref:mvc/controllers/filters) for working with antiforgery tokens:</span></span>

* [<span data-ttu-id="d8c62-201">ValidateAntiForgeryToken</span><span class="sxs-lookup"><span data-stu-id="d8c62-201">ValidateAntiForgeryToken</span></span>](/dotnet/api/microsoft.aspnetcore.mvc.validateantiforgerytokenattribute)
* [<span data-ttu-id="d8c62-202">AutoValidateAntiforgeryToken</span><span class="sxs-lookup"><span data-stu-id="d8c62-202">AutoValidateAntiforgeryToken</span></span>](/dotnet/api/microsoft.aspnetcore.mvc.autovalidateantiforgerytokenattribute)
* [<span data-ttu-id="d8c62-203">IgnoreAntiforgeryToken</span><span class="sxs-lookup"><span data-stu-id="d8c62-203">IgnoreAntiforgeryToken</span></span>](/dotnet/api/microsoft.aspnetcore.mvc.ignoreantiforgerytokenattribute)

## <a name="antiforgery-options"></a><span data-ttu-id="d8c62-204">Opzioni antifalsificazione</span><span class="sxs-lookup"><span data-stu-id="d8c62-204">Antiforgery options</span></span>

<span data-ttu-id="d8c62-205">Personalizzare le [Opzioni di antifalsificazione](/dotnet/api/Microsoft.AspNetCore.Antiforgery.AntiforgeryOptions) in `Startup.ConfigureServices` :</span><span class="sxs-lookup"><span data-stu-id="d8c62-205">Customize [antiforgery options](/dotnet/api/Microsoft.AspNetCore.Antiforgery.AntiforgeryOptions) in `Startup.ConfigureServices`:</span></span>

::: moniker range=">= aspnetcore-2.0"

```csharp
services.AddAntiforgery(options => 
{
    // Set Cookie properties using CookieBuilder properties†.
    options.FormFieldName = "AntiforgeryFieldname";
    options.HeaderName = "X-CSRF-TOKEN-HEADERNAME";
    options.SuppressXFrameOptionsHeader = false;
});
```

<span data-ttu-id="d8c62-206">&dagger;Impostare le proprietà antifalsificazione `Cookie` utilizzando le proprietà della classe [ Cookie Builder](/dotnet/api/microsoft.aspnetcore.http.cookiebuilder) .</span><span class="sxs-lookup"><span data-stu-id="d8c62-206">&dagger;Set the antiforgery `Cookie` properties using the properties of the [CookieBuilder](/dotnet/api/microsoft.aspnetcore.http.cookiebuilder) class.</span></span>

| <span data-ttu-id="d8c62-207">Opzione</span><span class="sxs-lookup"><span data-stu-id="d8c62-207">Option</span></span> | <span data-ttu-id="d8c62-208">Descrizione</span><span class="sxs-lookup"><span data-stu-id="d8c62-208">Description</span></span> |
| ------ | ----------- |
| [Cookie](/dotnet/api/microsoft.aspnetcore.antiforgery.antiforgeryoptions.cookie) | <span data-ttu-id="d8c62-209">Determina le impostazioni utilizzate per creare l'antifalsificazione cookie .</span><span class="sxs-lookup"><span data-stu-id="d8c62-209">Determines the settings used to create the antiforgery cookies.</span></span> |
| [<span data-ttu-id="d8c62-210">FormFieldName</span><span class="sxs-lookup"><span data-stu-id="d8c62-210">FormFieldName</span></span>](/dotnet/api/microsoft.aspnetcore.antiforgery.antiforgeryoptions.formfieldname) | <span data-ttu-id="d8c62-211">Nome del campo del form nascosto utilizzato dal sistema antifalsificazione per il rendering dei token antifalsificazione nelle viste.</span><span class="sxs-lookup"><span data-stu-id="d8c62-211">The name of the hidden form field used by the antiforgery system to render antiforgery tokens in views.</span></span> |
| [<span data-ttu-id="d8c62-212">HeaderName</span><span class="sxs-lookup"><span data-stu-id="d8c62-212">HeaderName</span></span>](/dotnet/api/microsoft.aspnetcore.antiforgery.antiforgeryoptions.headername) | <span data-ttu-id="d8c62-213">Nome dell'intestazione utilizzata dal sistema antifalsificazione.</span><span class="sxs-lookup"><span data-stu-id="d8c62-213">The name of the header used by the antiforgery system.</span></span> <span data-ttu-id="d8c62-214">Se `null` , il sistema considera solo i dati del modulo.</span><span class="sxs-lookup"><span data-stu-id="d8c62-214">If `null`, the system considers only form data.</span></span> |
| [<span data-ttu-id="d8c62-215">SuppressXFrameOptionsHeader</span><span class="sxs-lookup"><span data-stu-id="d8c62-215">SuppressXFrameOptionsHeader</span></span>](/dotnet/api/microsoft.aspnetcore.antiforgery.antiforgeryoptions.suppressxframeoptionsheader) | <span data-ttu-id="d8c62-216">Specifica se escludere la generazione dell' `X-Frame-Options` intestazione.</span><span class="sxs-lookup"><span data-stu-id="d8c62-216">Specifies whether to suppress generation of the `X-Frame-Options` header.</span></span> <span data-ttu-id="d8c62-217">Per impostazione predefinita, l'intestazione viene generata con un valore "SAMEORIGIN".</span><span class="sxs-lookup"><span data-stu-id="d8c62-217">By default, the header is generated with a value of "SAMEORIGIN".</span></span> <span data-ttu-id="d8c62-218">Il valore predefinito è `false`.</span><span class="sxs-lookup"><span data-stu-id="d8c62-218">Defaults to `false`.</span></span> |

::: moniker-end

::: moniker range="< aspnetcore-2.0"

```csharp
services.AddAntiforgery(options => 
{
    options.CookieDomain = "contoso.com";
    options.CookieName = "X-CSRF-TOKEN-COOKIENAME";
    options.CookiePath = "Path";
    options.FormFieldName = "AntiforgeryFieldname";
    options.HeaderName = "X-CSRF-TOKEN-HEADERNAME";
    options.RequireSsl = false;
    options.SuppressXFrameOptionsHeader = false;
});
```

| <span data-ttu-id="d8c62-219">Opzione</span><span class="sxs-lookup"><span data-stu-id="d8c62-219">Option</span></span> | <span data-ttu-id="d8c62-220">Descrizione</span><span class="sxs-lookup"><span data-stu-id="d8c62-220">Description</span></span> |
| ------ | ----------- |
| [Cookie](/dotnet/api/microsoft.aspnetcore.antiforgery.antiforgeryoptions.cookie) | <span data-ttu-id="d8c62-221">Determina le impostazioni utilizzate per creare l'antifalsificazione cookie .</span><span class="sxs-lookup"><span data-stu-id="d8c62-221">Determines the settings used to create the antiforgery cookies.</span></span> |
| <span data-ttu-id="d8c62-222">[CookieDominio](/dotnet/api/microsoft.aspnetcore.antiforgery.antiforgeryoptions.cookiedomain)</span><span class="sxs-lookup"><span data-stu-id="d8c62-222">[CookieDomain](/dotnet/api/microsoft.aspnetcore.antiforgery.antiforgeryoptions.cookiedomain)</span></span> | <span data-ttu-id="d8c62-223">Dominio di cookie .</span><span class="sxs-lookup"><span data-stu-id="d8c62-223">The domain of the cookie.</span></span> <span data-ttu-id="d8c62-224">Il valore predefinito è `null`.</span><span class="sxs-lookup"><span data-stu-id="d8c62-224">Defaults to `null`.</span></span> <span data-ttu-id="d8c62-225">Questa proprietà è obsoleta e verrà rimossa in una versione futura.</span><span class="sxs-lookup"><span data-stu-id="d8c62-225">This property is obsolete and will be removed in a future version.</span></span> <span data-ttu-id="d8c62-226">L'alternativa consigliata è Cookie . Dominio.</span><span class="sxs-lookup"><span data-stu-id="d8c62-226">The recommended alternative is Cookie.Domain.</span></span> |
| <span data-ttu-id="d8c62-227">[CookieNome](/dotnet/api/microsoft.aspnetcore.antiforgery.antiforgeryoptions.cookiename)</span><span class="sxs-lookup"><span data-stu-id="d8c62-227">[CookieName](/dotnet/api/microsoft.aspnetcore.antiforgery.antiforgeryoptions.cookiename)</span></span> | <span data-ttu-id="d8c62-228">Nome di cookie.</span><span class="sxs-lookup"><span data-stu-id="d8c62-228">The name of the cookie.</span></span> <span data-ttu-id="d8c62-229">Se non è impostato, il sistema genera un nome univoco che inizia con il [ Cookie prefisso predefinito](/dotnet/api/microsoft.aspnetcore.antiforgery.antiforgeryoptions.defaultcookieprefix) (". AspNetCore. antifalsificazione. ").</span><span class="sxs-lookup"><span data-stu-id="d8c62-229">If not set, the system generates a unique name beginning with the [DefaultCookiePrefix](/dotnet/api/microsoft.aspnetcore.antiforgery.antiforgeryoptions.defaultcookieprefix) (".AspNetCore.Antiforgery.").</span></span> <span data-ttu-id="d8c62-230">Questa proprietà è obsoleta e verrà rimossa in una versione futura.</span><span class="sxs-lookup"><span data-stu-id="d8c62-230">This property is obsolete and will be removed in a future version.</span></span> <span data-ttu-id="d8c62-231">L'alternativa consigliata è Cookie . Nome.</span><span class="sxs-lookup"><span data-stu-id="d8c62-231">The recommended alternative is Cookie.Name.</span></span> |
| <span data-ttu-id="d8c62-232">[CookiePercorso](/dotnet/api/microsoft.aspnetcore.antiforgery.antiforgeryoptions.cookiepath)</span><span class="sxs-lookup"><span data-stu-id="d8c62-232">[CookiePath](/dotnet/api/microsoft.aspnetcore.antiforgery.antiforgeryoptions.cookiepath)</span></span> | <span data-ttu-id="d8c62-233">Percorso impostato nell'oggetto cookie .</span><span class="sxs-lookup"><span data-stu-id="d8c62-233">The path set on the cookie.</span></span> <span data-ttu-id="d8c62-234">Questa proprietà è obsoleta e verrà rimossa in una versione futura.</span><span class="sxs-lookup"><span data-stu-id="d8c62-234">This property is obsolete and will be removed in a future version.</span></span> <span data-ttu-id="d8c62-235">L'alternativa consigliata è Cookie . Percorso.</span><span class="sxs-lookup"><span data-stu-id="d8c62-235">The recommended alternative is Cookie.Path.</span></span> |
| [<span data-ttu-id="d8c62-236">FormFieldName</span><span class="sxs-lookup"><span data-stu-id="d8c62-236">FormFieldName</span></span>](/dotnet/api/microsoft.aspnetcore.antiforgery.antiforgeryoptions.formfieldname) | <span data-ttu-id="d8c62-237">Nome del campo del form nascosto utilizzato dal sistema antifalsificazione per il rendering dei token antifalsificazione nelle viste.</span><span class="sxs-lookup"><span data-stu-id="d8c62-237">The name of the hidden form field used by the antiforgery system to render antiforgery tokens in views.</span></span> |
| [<span data-ttu-id="d8c62-238">HeaderName</span><span class="sxs-lookup"><span data-stu-id="d8c62-238">HeaderName</span></span>](/dotnet/api/microsoft.aspnetcore.antiforgery.antiforgeryoptions.headername) | <span data-ttu-id="d8c62-239">Nome dell'intestazione utilizzata dal sistema antifalsificazione.</span><span class="sxs-lookup"><span data-stu-id="d8c62-239">The name of the header used by the antiforgery system.</span></span> <span data-ttu-id="d8c62-240">Se `null` , il sistema considera solo i dati del modulo.</span><span class="sxs-lookup"><span data-stu-id="d8c62-240">If `null`, the system considers only form data.</span></span> |
| [<span data-ttu-id="d8c62-241">RequireSsl</span><span class="sxs-lookup"><span data-stu-id="d8c62-241">RequireSsl</span></span>](/dotnet/api/microsoft.aspnetcore.antiforgery.antiforgeryoptions.requiressl) | <span data-ttu-id="d8c62-242">Specifica se HTTPS è richiesto dal sistema antifalsificazione.</span><span class="sxs-lookup"><span data-stu-id="d8c62-242">Specifies whether HTTPS is required by the antiforgery system.</span></span> <span data-ttu-id="d8c62-243">Se `true` , le richieste non HTTPS hanno esito negativo.</span><span class="sxs-lookup"><span data-stu-id="d8c62-243">If `true`, non-HTTPS requests fail.</span></span> <span data-ttu-id="d8c62-244">Il valore predefinito è `false`.</span><span class="sxs-lookup"><span data-stu-id="d8c62-244">Defaults to `false`.</span></span> <span data-ttu-id="d8c62-245">Questa proprietà è obsoleta e verrà rimossa in una versione futura.</span><span class="sxs-lookup"><span data-stu-id="d8c62-245">This property is obsolete and will be removed in a future version.</span></span> <span data-ttu-id="d8c62-246">L'alternativa consigliata consiste nell'impostare Cookie . SecurePolicy.</span><span class="sxs-lookup"><span data-stu-id="d8c62-246">The recommended alternative is to set Cookie.SecurePolicy.</span></span> |
| [<span data-ttu-id="d8c62-247">SuppressXFrameOptionsHeader</span><span class="sxs-lookup"><span data-stu-id="d8c62-247">SuppressXFrameOptionsHeader</span></span>](/dotnet/api/microsoft.aspnetcore.antiforgery.antiforgeryoptions.suppressxframeoptionsheader) | <span data-ttu-id="d8c62-248">Specifica se escludere la generazione dell' `X-Frame-Options` intestazione.</span><span class="sxs-lookup"><span data-stu-id="d8c62-248">Specifies whether to suppress generation of the `X-Frame-Options` header.</span></span> <span data-ttu-id="d8c62-249">Per impostazione predefinita, l'intestazione viene generata con un valore "SAMEORIGIN".</span><span class="sxs-lookup"><span data-stu-id="d8c62-249">By default, the header is generated with a value of "SAMEORIGIN".</span></span> <span data-ttu-id="d8c62-250">Il valore predefinito è `false`.</span><span class="sxs-lookup"><span data-stu-id="d8c62-250">Defaults to `false`.</span></span> |

::: moniker-end

<span data-ttu-id="d8c62-251">Per ulteriori informazioni, vedere [ Cookie AuthenticationOptions](/dotnet/api/Microsoft.AspNetCore.Builder.CookieAuthenticationOptions).</span><span class="sxs-lookup"><span data-stu-id="d8c62-251">For more information, see [CookieAuthenticationOptions](/dotnet/api/Microsoft.AspNetCore.Builder.CookieAuthenticationOptions).</span></span>

## <a name="configure-antiforgery-features-with-iantiforgery"></a><span data-ttu-id="d8c62-252">Configurare le funzionalità antifalsificazione con IAntiforgery</span><span class="sxs-lookup"><span data-stu-id="d8c62-252">Configure antiforgery features with IAntiforgery</span></span>

<span data-ttu-id="d8c62-253">[IAntiforgery](/dotnet/api/microsoft.aspnetcore.antiforgery.iantiforgery) fornisce l'API per configurare le funzionalità antifalsificazione.</span><span class="sxs-lookup"><span data-stu-id="d8c62-253">[IAntiforgery](/dotnet/api/microsoft.aspnetcore.antiforgery.iantiforgery) provides the API to configure antiforgery features.</span></span> <span data-ttu-id="d8c62-254">`IAntiforgery` può essere richiesto nel `Configure` metodo della `Startup` classe.</span><span class="sxs-lookup"><span data-stu-id="d8c62-254">`IAntiforgery` can be requested in the `Configure` method of the `Startup` class.</span></span> <span data-ttu-id="d8c62-255">Nell'esempio seguente viene usato il middleware della home page dell'app per generare un token antifalsificazione e inviarlo nella risposta come cookie (usando la convenzione di denominazione angolare predefinita descritta più avanti in questo argomento):</span><span class="sxs-lookup"><span data-stu-id="d8c62-255">The following example uses middleware from the app's home page to generate an antiforgery token and send it in the response as a cookie (using the default Angular naming convention described later in this topic):</span></span>

```csharp
public void Configure(IApplicationBuilder app, IAntiforgery antiforgery)
{
    app.Use(next => context =>
    {
        string path = context.Request.Path.Value;

        if (
            string.Equals(path, "/", StringComparison.OrdinalIgnoreCase) ||
            string.Equals(path, "/index.html", StringComparison.OrdinalIgnoreCase))
        {
            // The request token can be sent as a JavaScript-readable cookie, 
            // and Angular uses it by default.
            var tokens = antiforgery.GetAndStoreTokens(context);
            context.Response.Cookies.Append("XSRF-TOKEN", tokens.RequestToken, 
                new CookieOptions() { HttpOnly = false });
        }

        return next(context);
    });
}
```

### <a name="require-antiforgery-validation"></a><span data-ttu-id="d8c62-256">Richiedi convalida antifalsificazione</span><span class="sxs-lookup"><span data-stu-id="d8c62-256">Require antiforgery validation</span></span>

<span data-ttu-id="d8c62-257">[ValidateAntiForgeryToken](/dotnet/api/microsoft.aspnetcore.mvc.validateantiforgerytokenattribute) è un filtro azioni che può essere applicato a una singola azione, a un controller o a livello globale.</span><span class="sxs-lookup"><span data-stu-id="d8c62-257">[ValidateAntiForgeryToken](/dotnet/api/microsoft.aspnetcore.mvc.validateantiforgerytokenattribute) is an action filter that can be applied to an individual action, a controller, or globally.</span></span> <span data-ttu-id="d8c62-258">Le richieste effettuate alle azioni con questo filtro vengono bloccate a meno che la richiesta non includa un token antifalsificazione valido.</span><span class="sxs-lookup"><span data-stu-id="d8c62-258">Requests made to actions that have this filter applied are blocked unless the request includes a valid antiforgery token.</span></span>

```csharp
[HttpPost]
[ValidateAntiForgeryToken]
public async Task<IActionResult> RemoveLogin(RemoveLoginViewModel account)
{
    ManageMessageId? message = ManageMessageId.Error;
    var user = await GetCurrentUserAsync();

    if (user != null)
    {
        var result = 
            await _userManager.RemoveLoginAsync(
                user, account.LoginProvider, account.ProviderKey);

        if (result.Succeeded)
        {
            await _signInManager.SignInAsync(user, isPersistent: false);
            message = ManageMessageId.RemoveLoginSuccess;
        }
    }

    return RedirectToAction(nameof(ManageLogins), new { Message = message });
}
```

<span data-ttu-id="d8c62-259">L' `ValidateAntiForgeryToken` attributo richiede un token per le richieste ai metodi di azione contrassegnati, incluse le richieste HTTP Get.</span><span class="sxs-lookup"><span data-stu-id="d8c62-259">The `ValidateAntiForgeryToken` attribute requires a token for requests to the action methods it marks, including HTTP GET requests.</span></span> <span data-ttu-id="d8c62-260">Se l' `ValidateAntiForgeryToken` attributo viene applicato tra i controller dell'app, è possibile eseguirne l'override con `IgnoreAntiforgeryToken` l'attributo.</span><span class="sxs-lookup"><span data-stu-id="d8c62-260">If the `ValidateAntiForgeryToken` attribute is applied across the app's controllers, it can be overridden with the `IgnoreAntiforgeryToken` attribute.</span></span>

> [!NOTE]
> <span data-ttu-id="d8c62-261">ASP.NET Core non supporta l'aggiunta di token antifalsificazione per ottenere automaticamente le richieste.</span><span class="sxs-lookup"><span data-stu-id="d8c62-261">ASP.NET Core doesn't support adding antiforgery tokens to GET requests automatically.</span></span>

### <a name="automatically-validate-antiforgery-tokens-for-unsafe-http-methods-only"></a><span data-ttu-id="d8c62-262">Convalidare automaticamente i token antifalsificazione per metodi HTTP unsafe</span><span class="sxs-lookup"><span data-stu-id="d8c62-262">Automatically validate antiforgery tokens for unsafe HTTP methods only</span></span>

<span data-ttu-id="d8c62-263">ASP.NET Core app non generano token antifalsificazione per metodi HTTP sicuri (GET, HEAD, OPTIONS e TRACE).</span><span class="sxs-lookup"><span data-stu-id="d8c62-263">ASP.NET Core apps don't generate antiforgery tokens for safe HTTP methods (GET, HEAD, OPTIONS, and TRACE).</span></span> <span data-ttu-id="d8c62-264">Anziché applicare l' `ValidateAntiForgeryToken` attributo in generale e quindi eseguire l'override con `IgnoreAntiforgeryToken` gli attributi, è possibile usare l'attributo [AutoValidateAntiforgeryToken](/dotnet/api/microsoft.aspnetcore.mvc.autovalidateantiforgerytokenattribute) .</span><span class="sxs-lookup"><span data-stu-id="d8c62-264">Instead of broadly applying the `ValidateAntiForgeryToken` attribute and then overriding it with `IgnoreAntiforgeryToken` attributes, the [AutoValidateAntiforgeryToken](/dotnet/api/microsoft.aspnetcore.mvc.autovalidateantiforgerytokenattribute) attribute can be used.</span></span> <span data-ttu-id="d8c62-265">Questo attributo funziona in modo identico all' `ValidateAntiForgeryToken` attributo, con la differenza che non richiede token per le richieste effettuate usando i metodi HTTP seguenti:</span><span class="sxs-lookup"><span data-stu-id="d8c62-265">This attribute works identically to the `ValidateAntiForgeryToken` attribute, except that it doesn't require tokens for requests made using the following HTTP methods:</span></span>

* <span data-ttu-id="d8c62-266">GET</span><span class="sxs-lookup"><span data-stu-id="d8c62-266">GET</span></span>
* <span data-ttu-id="d8c62-267">HEAD</span><span class="sxs-lookup"><span data-stu-id="d8c62-267">HEAD</span></span>
* <span data-ttu-id="d8c62-268">OPTIONS</span><span class="sxs-lookup"><span data-stu-id="d8c62-268">OPTIONS</span></span>
* <span data-ttu-id="d8c62-269">TRACE</span><span class="sxs-lookup"><span data-stu-id="d8c62-269">TRACE</span></span>

<span data-ttu-id="d8c62-270">Si consiglia di usare `AutoValidateAntiforgeryToken` ampiamente per gli scenari non API.</span><span class="sxs-lookup"><span data-stu-id="d8c62-270">We recommend use of `AutoValidateAntiforgeryToken` broadly for non-API scenarios.</span></span> <span data-ttu-id="d8c62-271">Ciò garantisce che le azioni POST siano protette per impostazione predefinita.</span><span class="sxs-lookup"><span data-stu-id="d8c62-271">This ensures POST actions are protected by default.</span></span> <span data-ttu-id="d8c62-272">In alternativa, è possibile ignorare i token antifalsificazione per impostazione predefinita, a meno che non `ValidateAntiForgeryToken` venga applicato a singoli metodi di azione.</span><span class="sxs-lookup"><span data-stu-id="d8c62-272">The alternative is to ignore antiforgery tokens by default, unless `ValidateAntiForgeryToken` is applied to individual action methods.</span></span> <span data-ttu-id="d8c62-273">È più probabile che in questo scenario un metodo di azione POST venga lasciato non protetto per errore, lasciando l'app vulnerabile agli attacchi CSRF.</span><span class="sxs-lookup"><span data-stu-id="d8c62-273">It's more likely in this scenario for a POST action method to be left unprotected by mistake, leaving the app vulnerable to CSRF attacks.</span></span> <span data-ttu-id="d8c62-274">Tutti i post devono inviare il token antifalsificazione.</span><span class="sxs-lookup"><span data-stu-id="d8c62-274">All POSTs should send the antiforgery token.</span></span>

<span data-ttu-id="d8c62-275">Le API non dispongono di un meccanismo automatico per l'invio cookie di non parte del token.</span><span class="sxs-lookup"><span data-stu-id="d8c62-275">APIs don't have an automatic mechanism for sending the non-cookie part of the token.</span></span> <span data-ttu-id="d8c62-276">È probabile che l'implementazione dipenda dall'implementazione del codice client.</span><span class="sxs-lookup"><span data-stu-id="d8c62-276">The implementation probably depends on the client code implementation.</span></span> <span data-ttu-id="d8c62-277">Di seguito sono riportati alcuni esempi:</span><span class="sxs-lookup"><span data-stu-id="d8c62-277">Some examples are shown below:</span></span>

<span data-ttu-id="d8c62-278">Esempio a livello di classe:</span><span class="sxs-lookup"><span data-stu-id="d8c62-278">Class-level example:</span></span>

```csharp
[Authorize]
[AutoValidateAntiforgeryToken]
public class ManageController : Controller
{
```

<span data-ttu-id="d8c62-279">Esempio globale:</span><span class="sxs-lookup"><span data-stu-id="d8c62-279">Global example:</span></span>

::: moniker range="< aspnetcore-3.0"

<span data-ttu-id="d8c62-280">Servizi. AddMvc (opzioni = opzioni>. Filters. Add (New AutoValidateAntiforgeryTokenAttribute ()));</span><span class="sxs-lookup"><span data-stu-id="d8c62-280">services.AddMvc(options =>     options.Filters.Add(new AutoValidateAntiforgeryTokenAttribute()));</span></span>

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

```csharp
services.AddControllersWithViews(options =>
    options.Filters.Add(new AutoValidateAntiforgeryTokenAttribute()));
```

::: moniker-end

### <a name="override-global-or-controller-antiforgery-attributes"></a><span data-ttu-id="d8c62-281">Sostituisci attributi antifalsificazione globali o controller</span><span class="sxs-lookup"><span data-stu-id="d8c62-281">Override global or controller antiforgery attributes</span></span>

<span data-ttu-id="d8c62-282">Il filtro [IgnoreAntiforgeryToken](/dotnet/api/microsoft.aspnetcore.mvc.ignoreantiforgerytokenattribute) viene usato per eliminare la necessità di un token antifalsificazione per un'azione o un controller specifico.</span><span class="sxs-lookup"><span data-stu-id="d8c62-282">The [IgnoreAntiforgeryToken](/dotnet/api/microsoft.aspnetcore.mvc.ignoreantiforgerytokenattribute) filter is used to eliminate the need for an antiforgery token for a given action (or controller).</span></span> <span data-ttu-id="d8c62-283">Quando applicato, questo filtro sostituisce `ValidateAntiForgeryToken` i `AutoValidateAntiforgeryToken` filtri e specificati a un livello superiore (globalmente o in un controller).</span><span class="sxs-lookup"><span data-stu-id="d8c62-283">When applied, this filter overrides `ValidateAntiForgeryToken` and `AutoValidateAntiforgeryToken` filters specified at a higher level (globally or on a controller).</span></span>

```csharp
[Authorize]
[AutoValidateAntiforgeryToken]
public class ManageController : Controller
{
    [HttpPost]
    [IgnoreAntiforgeryToken]
    public async Task<IActionResult> DoSomethingSafe(SomeViewModel model)
    {
        // no antiforgery token required
    }
}
```

## <a name="refresh-tokens-after-authentication"></a><span data-ttu-id="d8c62-284">Aggiornare i token dopo l'autenticazione</span><span class="sxs-lookup"><span data-stu-id="d8c62-284">Refresh tokens after authentication</span></span>

<span data-ttu-id="d8c62-285">I token devono essere aggiornati dopo che l'utente è stato autenticato reindirizzando l'utente a una pagina della visualizzazione o delle Razor pagine.</span><span class="sxs-lookup"><span data-stu-id="d8c62-285">Tokens should be refreshed after the user is authenticated by redirecting the user to a view or Razor Pages page.</span></span>

## <a name="javascript-ajax-and-spas"></a><span data-ttu-id="d8c62-286">JavaScript, AJAX e Spa</span><span class="sxs-lookup"><span data-stu-id="d8c62-286">JavaScript, AJAX, and SPAs</span></span>

<span data-ttu-id="d8c62-287">Nelle app tradizionali basate su HTML, i token antifalsificazione vengono passati al server usando i campi dei moduli nascosti.</span><span class="sxs-lookup"><span data-stu-id="d8c62-287">In traditional HTML-based apps, antiforgery tokens are passed to the server using hidden form fields.</span></span> <span data-ttu-id="d8c62-288">Nelle app moderne basate su JavaScript e in Spa, molte richieste vengono eseguite a livello di programmazione.</span><span class="sxs-lookup"><span data-stu-id="d8c62-288">In modern JavaScript-based apps and SPAs, many requests are made programmatically.</span></span> <span data-ttu-id="d8c62-289">Queste richieste AJAX possono usare altre tecniche (ad esempio, intestazioni di richiesta o cookie s) per inviare il token.</span><span class="sxs-lookup"><span data-stu-id="d8c62-289">These AJAX requests may use other techniques (such as request headers or cookies) to send the token.</span></span>

<span data-ttu-id="d8c62-290">Se cookie vengono usati per archiviare i token di autenticazione e per autenticare le richieste API sul server, CSRF è un potenziale problema.</span><span class="sxs-lookup"><span data-stu-id="d8c62-290">If cookies are used to store authentication tokens and to authenticate API requests on the server, CSRF is a potential problem.</span></span> <span data-ttu-id="d8c62-291">Se viene usata l'archiviazione locale per archiviare il token, la vulnerabilità CSRF potrebbe essere attenuata perché i valori della risorsa di archiviazione locale non vengono inviati automaticamente al server con ogni richiesta.</span><span class="sxs-lookup"><span data-stu-id="d8c62-291">If local storage is used to store the token, CSRF vulnerability might be mitigated because values from local storage aren't sent automatically to the server with every request.</span></span> <span data-ttu-id="d8c62-292">Quindi, l'uso dell'archiviazione locale per archiviare il token antifalsificazione nel client e l'invio del token come intestazione della richiesta è un approccio consigliato.</span><span class="sxs-lookup"><span data-stu-id="d8c62-292">Thus, using local storage to store the antiforgery token on the client and sending the token as a request header is a recommended approach.</span></span>

### <a name="javascript"></a><span data-ttu-id="d8c62-293">JavaScript</span><span class="sxs-lookup"><span data-stu-id="d8c62-293">JavaScript</span></span>

<span data-ttu-id="d8c62-294">Utilizzando JavaScript con le visualizzazioni, è possibile creare il token utilizzando un servizio all'interno della vista.</span><span class="sxs-lookup"><span data-stu-id="d8c62-294">Using JavaScript with views, the token can be created using a service from within the view.</span></span> <span data-ttu-id="d8c62-295">Inserire il servizio [Microsoft. AspNetCore. antifalsification. IAntiforgery](/dotnet/api/microsoft.aspnetcore.antiforgery.iantiforgery) nella vista e chiamare [GetAndStoreTokens](/dotnet/api/microsoft.aspnetcore.antiforgery.iantiforgery.getandstoretokens):</span><span class="sxs-lookup"><span data-stu-id="d8c62-295">Inject the [Microsoft.AspNetCore.Antiforgery.IAntiforgery](/dotnet/api/microsoft.aspnetcore.antiforgery.iantiforgery) service into the view and call [GetAndStoreTokens](/dotnet/api/microsoft.aspnetcore.antiforgery.iantiforgery.getandstoretokens):</span></span>

[!code-cshtml[](anti-request-forgery/sample/MvcSample/Views/Home/Ajax.cshtml?highlight=4-10,12-13,35-36)]

<span data-ttu-id="d8c62-296">Questo approccio elimina la necessità di gestire direttamente l'impostazione del cookie Server o di leggerli dal client.</span><span class="sxs-lookup"><span data-stu-id="d8c62-296">This approach eliminates the need to deal directly with setting cookies from the server or reading them from the client.</span></span>

<span data-ttu-id="d8c62-297">Nell'esempio precedente viene usato JavaScript per leggere il valore del campo nascosto per l'intestazione AJAX POST.</span><span class="sxs-lookup"><span data-stu-id="d8c62-297">The preceding example uses JavaScript to read the hidden field value for the AJAX POST header.</span></span>

<span data-ttu-id="d8c62-298">JavaScript può anche accedere ai token in cookie s e usare il cookie contenuto di per creare un'intestazione con il valore del token.</span><span class="sxs-lookup"><span data-stu-id="d8c62-298">JavaScript can also access tokens in cookies and use the cookie's contents to create a header with the token's value.</span></span>

```csharp
context.Response.Cookies.Append("CSRF-TOKEN", tokens.RequestToken, 
    new Microsoft.AspNetCore.Http.CookieOptions { HttpOnly = false });
```

<span data-ttu-id="d8c62-299">Supponendo che lo script richieda di inviare il token in un'intestazione denominata `X-CSRF-TOKEN` , configurare il servizio antifalsificazione per cercare l' `X-CSRF-TOKEN` intestazione:</span><span class="sxs-lookup"><span data-stu-id="d8c62-299">Assuming the script requests to send the token in a header called `X-CSRF-TOKEN`, configure the antiforgery service to look for the `X-CSRF-TOKEN` header:</span></span>

```csharp
services.AddAntiforgery(options => options.HeaderName = "X-CSRF-TOKEN");
```

<span data-ttu-id="d8c62-300">Nell'esempio seguente viene usato JavaScript per creare una richiesta AJAX con l'intestazione appropriata:</span><span class="sxs-lookup"><span data-stu-id="d8c62-300">The following example uses JavaScript to make an AJAX request with the appropriate header:</span></span>

```javascript
function getCookie(cname) {
    var name = cname + "=";
    var decodedCookie = decodeURIComponent(document.cookie);
    var ca = decodedCookie.split(';');
    for(var i = 0; i <ca.length; i++) {
        var c = ca[i];
        while (c.charAt(0) == ' ') {
            c = c.substring(1);
        }
        if (c.indexOf(name) == 0) {
            return c.substring(name.length, c.length);
        }
    }
    return "";
}

var csrfToken = getCookie("CSRF-TOKEN");

var xhttp = new XMLHttpRequest();
xhttp.onreadystatechange = function() {
    if (xhttp.readyState == XMLHttpRequest.DONE) {
        if (xhttp.status == 200) {
            alert(xhttp.responseText);
        } else {
            alert('There was an error processing the AJAX request.');
        }
    }
};
xhttp.open('POST', '/api/password/changepassword', true);
xhttp.setRequestHeader("Content-type", "application/json");
xhttp.setRequestHeader("X-CSRF-TOKEN", csrfToken);
xhttp.send(JSON.stringify({ "newPassword": "ReallySecurePassword999$$$" }));
```

### <a name="angularjs"></a><span data-ttu-id="d8c62-301">AngularJS</span><span class="sxs-lookup"><span data-stu-id="d8c62-301">AngularJS</span></span>

<span data-ttu-id="d8c62-302">AngularJS usa una convenzione per indirizzare CSRF.</span><span class="sxs-lookup"><span data-stu-id="d8c62-302">AngularJS uses a convention to address CSRF.</span></span> <span data-ttu-id="d8c62-303">Se il server invia un oggetto cookie con il nome `XSRF-TOKEN` , il `$http` servizio AngularJS aggiunge cookie il valore a un'intestazione quando invia una richiesta al server.</span><span class="sxs-lookup"><span data-stu-id="d8c62-303">If the server sends a cookie with the name `XSRF-TOKEN`, the AngularJS `$http` service adds the cookie value to a header when it sends a request to the server.</span></span> <span data-ttu-id="d8c62-304">Questo processo è automatico.</span><span class="sxs-lookup"><span data-stu-id="d8c62-304">This process is automatic.</span></span> <span data-ttu-id="d8c62-305">L'intestazione non deve essere impostata in modo esplicito nel client.</span><span class="sxs-lookup"><span data-stu-id="d8c62-305">The header doesn't need to be set in the client explicitly.</span></span> <span data-ttu-id="d8c62-306">Il nome dell'intestazione è `X-XSRF-TOKEN` .</span><span class="sxs-lookup"><span data-stu-id="d8c62-306">The header name is `X-XSRF-TOKEN`.</span></span> <span data-ttu-id="d8c62-307">Il server deve rilevare questa intestazione e convalidarne il contenuto.</span><span class="sxs-lookup"><span data-stu-id="d8c62-307">The server should detect this header and validate its contents.</span></span>

<span data-ttu-id="d8c62-308">Per ASP.NET Core API da usare con questa convenzione nell'avvio dell'applicazione:</span><span class="sxs-lookup"><span data-stu-id="d8c62-308">For ASP.NET Core API to work with this convention in your application startup:</span></span>

* <span data-ttu-id="d8c62-309">Configurare l'app in modo che fornisca un token in una cookie chiamata `XSRF-TOKEN` .</span><span class="sxs-lookup"><span data-stu-id="d8c62-309">Configure your app to provide a token in a cookie called `XSRF-TOKEN`.</span></span>
* <span data-ttu-id="d8c62-310">Configurare il servizio antifalsificazione per cercare un'intestazione denominata `X-XSRF-TOKEN` .</span><span class="sxs-lookup"><span data-stu-id="d8c62-310">Configure the antiforgery service to look for a header named `X-XSRF-TOKEN`.</span></span>

```csharp
public void Configure(IApplicationBuilder app, IAntiforgery antiforgery)
{
    app.Use(next => context =>
    {
        string path = context.Request.Path.Value;

        if (
            string.Equals(path, "/", StringComparison.OrdinalIgnoreCase) ||
            string.Equals(path, "/index.html", StringComparison.OrdinalIgnoreCase))
        {
            // The request token can be sent as a JavaScript-readable cookie, 
            // and Angular uses it by default.
            var tokens = antiforgery.GetAndStoreTokens(context);
            context.Response.Cookies.Append("XSRF-TOKEN", tokens.RequestToken, 
                new CookieOptions() { HttpOnly = false });
        }

        return next(context);
    });
}

public void ConfigureServices(IServiceCollection services)
{
    // Angular's default header name for sending the XSRF token.
    services.AddAntiforgery(options => options.HeaderName = "X-XSRF-TOKEN");
}
```

<span data-ttu-id="d8c62-311">[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/security/anti-request-forgery/sample/AngularSample) ([procedura per il download](xref:index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="d8c62-311">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/security/anti-request-forgery/sample/AngularSample) ([how to download](xref:index#how-to-download-a-sample))</span></span>

## <a name="windows-authentication-and-antiforgery-no-loccookies"></a><span data-ttu-id="d8c62-312">Autenticazione di Windows e antifalsificazione cookie</span><span class="sxs-lookup"><span data-stu-id="d8c62-312">Windows authentication and antiforgery cookies</span></span>

<span data-ttu-id="d8c62-313">Quando si usa l'autenticazione di Windows, gli endpoint dell'applicazione devono essere protetti dagli attacchi CSRF nello stesso modo in cui vengono eseguiti per cookie i.</span><span class="sxs-lookup"><span data-stu-id="d8c62-313">When using Windows Authentication, application endpoints must be protected against CSRF attacks in the same way as done for cookies.</span></span>  <span data-ttu-id="d8c62-314">Il browser invia in modo implicito il contesto di autenticazione al server, pertanto gli endpoint devono essere protetti dagli attacchi CSRF.</span><span class="sxs-lookup"><span data-stu-id="d8c62-314">The browser implicitly sends the authentication context to the server, therefore endpoints need to be protected against CSRF attacks.</span></span>

## <a name="extend-antiforgery"></a><span data-ttu-id="d8c62-315">Estendi antifalsificazione</span><span class="sxs-lookup"><span data-stu-id="d8c62-315">Extend antiforgery</span></span>

<span data-ttu-id="d8c62-316">Il tipo [IAntiForgeryAdditionalDataProvider](/dotnet/api/microsoft.aspnetcore.antiforgery.iantiforgeryadditionaldataprovider) consente agli sviluppatori di estendere il comportamento del sistema anti-CSRF eseguendo il round trip di dati aggiuntivi in ogni token.</span><span class="sxs-lookup"><span data-stu-id="d8c62-316">The [IAntiForgeryAdditionalDataProvider](/dotnet/api/microsoft.aspnetcore.antiforgery.iantiforgeryadditionaldataprovider) type allows developers to extend the behavior of the anti-CSRF system by round-tripping additional data in each token.</span></span> <span data-ttu-id="d8c62-317">Il metodo [GetAdditionalData](/dotnet/api/microsoft.aspnetcore.antiforgery.iantiforgeryadditionaldataprovider.getadditionaldata) viene chiamato ogni volta che viene generato un token di campo e il valore restituito viene incorporato all'interno del token generato.</span><span class="sxs-lookup"><span data-stu-id="d8c62-317">The [GetAdditionalData](/dotnet/api/microsoft.aspnetcore.antiforgery.iantiforgeryadditionaldataprovider.getadditionaldata) method is called each time a field token is generated, and the return value is embedded within the generated token.</span></span> <span data-ttu-id="d8c62-318">Un responsabile dell'implementazione può restituire un timestamp, un parametro nonce o qualsiasi altro valore, quindi chiamare [ValidateAdditionalData](/dotnet/api/microsoft.aspnetcore.antiforgery.iantiforgeryadditionaldataprovider.validateadditionaldata) per convalidare questi dati quando il token viene convalidato.</span><span class="sxs-lookup"><span data-stu-id="d8c62-318">An implementer could return a timestamp, a nonce, or any other value and then call [ValidateAdditionalData](/dotnet/api/microsoft.aspnetcore.antiforgery.iantiforgeryadditionaldataprovider.validateadditionaldata) to validate this data when the token is validated.</span></span> <span data-ttu-id="d8c62-319">Il nome utente del client è già incorporato nei token generati, pertanto non è necessario includere tali informazioni.</span><span class="sxs-lookup"><span data-stu-id="d8c62-319">The client's username is already embedded in the generated tokens, so there's no need to include this information.</span></span> <span data-ttu-id="d8c62-320">Se un token include dati supplementari ma non `IAntiForgeryAdditionalDataProvider` è configurato alcun oggetto, i dati supplementari non vengono convalidati.</span><span class="sxs-lookup"><span data-stu-id="d8c62-320">If a token includes supplemental data but no `IAntiForgeryAdditionalDataProvider` is configured, the supplemental data isn't validated.</span></span>

## <a name="additional-resources"></a><span data-ttu-id="d8c62-321">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="d8c62-321">Additional resources</span></span>

* <span data-ttu-id="d8c62-322">[CSRF](https://www.owasp.org/index.php/Cross-Site_Request_Forgery_(CSRF)) in [Open Web Application Security Project](https://www.owasp.org/index.php/Main_Page) (OWASP).</span><span class="sxs-lookup"><span data-stu-id="d8c62-322">[CSRF](https://www.owasp.org/index.php/Cross-Site_Request_Forgery_(CSRF)) on [Open Web Application Security Project](https://www.owasp.org/index.php/Main_Page) (OWASP).</span></span>
* <xref:host-and-deploy/web-farm>
