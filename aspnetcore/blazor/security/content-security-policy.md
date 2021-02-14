---
title: Applicare un criterio di sicurezza del contenuto per ASP.NET Core Blazor
author: guardrex
description: Informazioni su come usare un criterio di sicurezza del contenuto (CSP) con ASP.NET Core Blazor app per la protezione da attacchi XSS (cross-site scripting).
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 05/19/2020
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
uid: blazor/security/content-security-policy
ms.openlocfilehash: f44f348947e31864f5d0d44c9caf1a3aa9c3b1d4
ms.sourcegitcommit: 1166b0ff3828418559510c661e8240e5c5717bb7
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/12/2021
ms.locfileid: "100280339"
---
# <a name="enforce-a-content-security-policy-for-aspnet-core-blazor"></a><span data-ttu-id="74f9a-103">Applicare un criterio di sicurezza del contenuto per ASP.NET Core Blazor</span><span class="sxs-lookup"><span data-stu-id="74f9a-103">Enforce a Content Security Policy for ASP.NET Core Blazor</span></span>

<span data-ttu-id="74f9a-104">Il [cross-site scripting (XSS)](xref:security/cross-site-scripting) è una vulnerabilità di sicurezza in cui un utente malintenzionato inserisce uno o più script lato client dannosi nel contenuto sottoposto a rendering di un'app.</span><span class="sxs-lookup"><span data-stu-id="74f9a-104">[Cross-Site Scripting (XSS)](xref:security/cross-site-scripting) is a security vulnerability where an attacker places one or more malicious client-side scripts into an app's rendered content.</span></span> <span data-ttu-id="74f9a-105">Un criterio di sicurezza del contenuto (CSP) consente di proteggersi da attacchi XSS, informando il browser di valido:</span><span class="sxs-lookup"><span data-stu-id="74f9a-105">A Content Security Policy (CSP) helps protect against XSS attacks by informing the browser of valid:</span></span>

* <span data-ttu-id="74f9a-106">Origini per il contenuto caricato, inclusi script, fogli di stile e immagini.</span><span class="sxs-lookup"><span data-stu-id="74f9a-106">Sources for loaded content, including scripts, stylesheets, and images.</span></span>
* <span data-ttu-id="74f9a-107">Azioni eseguite da una pagina, che specifica le destinazioni URL consentite dei moduli.</span><span class="sxs-lookup"><span data-stu-id="74f9a-107">Actions taken by a page, specifying permitted URL targets of forms.</span></span>
* <span data-ttu-id="74f9a-108">Plug-in che possono essere caricati.</span><span class="sxs-lookup"><span data-stu-id="74f9a-108">Plugins that can be loaded.</span></span>

<span data-ttu-id="74f9a-109">Per applicare un CSP a un'app, lo sviluppatore specifica diverse *direttive* di sicurezza del contenuto CSP in una o più `Content-Security-Policy` intestazioni o `<meta>` tag.</span><span class="sxs-lookup"><span data-stu-id="74f9a-109">To apply a CSP to an app, the developer specifies several CSP content security *directives* in one or more `Content-Security-Policy` headers or `<meta>` tags.</span></span>

<span data-ttu-id="74f9a-110">I criteri vengono valutati dal browser durante il caricamento di una pagina.</span><span class="sxs-lookup"><span data-stu-id="74f9a-110">Policies are evaluated by the browser while a page is loading.</span></span> <span data-ttu-id="74f9a-111">Il browser controlla le origini della pagina e determina se soddisfano i requisiti delle direttive di sicurezza del contenuto.</span><span class="sxs-lookup"><span data-stu-id="74f9a-111">The browser inspects the page's sources and determines if they meet the requirements of the content security directives.</span></span> <span data-ttu-id="74f9a-112">Quando le direttive dei criteri non vengono soddisfatte per una risorsa, il browser non carica la risorsa.</span><span class="sxs-lookup"><span data-stu-id="74f9a-112">When policy directives aren't met for a resource, the browser doesn't load the resource.</span></span> <span data-ttu-id="74f9a-113">Si consideri, ad esempio, un criterio che non consente script di terze parti.</span><span class="sxs-lookup"><span data-stu-id="74f9a-113">For example, consider a policy that doesn't allow third-party scripts.</span></span> <span data-ttu-id="74f9a-114">Quando una pagina contiene un `<script>` tag con un'origine di terze parti nell' `src` attributo, il browser impedisce il caricamento dello script.</span><span class="sxs-lookup"><span data-stu-id="74f9a-114">When a page contains a `<script>` tag with a third-party origin in the `src` attribute, the browser prevents the script from loading.</span></span>

<span data-ttu-id="74f9a-115">CSP è supportato nella maggior parte dei browser moderni per desktop e per dispositivi mobili, tra cui Chrome, Edge, Firefox, opera e Safari.</span><span class="sxs-lookup"><span data-stu-id="74f9a-115">CSP is supported in most modern desktop and mobile browsers, including Chrome, Edge, Firefox, Opera, and Safari.</span></span> <span data-ttu-id="74f9a-116">CSP è consigliato per le Blazor app.</span><span class="sxs-lookup"><span data-stu-id="74f9a-116">CSP is recommended for Blazor apps.</span></span>

## <a name="policy-directives"></a><span data-ttu-id="74f9a-117">Direttive dei criteri</span><span class="sxs-lookup"><span data-stu-id="74f9a-117">Policy directives</span></span>

<span data-ttu-id="74f9a-118">Specificare almeno le direttive e le origini seguenti per le Blazor app.</span><span class="sxs-lookup"><span data-stu-id="74f9a-118">Minimally, specify the following directives and sources for Blazor apps.</span></span> <span data-ttu-id="74f9a-119">Aggiungere altre direttive e origini in base alle esigenze.</span><span class="sxs-lookup"><span data-stu-id="74f9a-119">Add additional directives and sources as needed.</span></span> <span data-ttu-id="74f9a-120">Le direttive seguenti vengono usate nella sezione [applicare i criteri](#apply-the-policy) di questo articolo, in cui sono disponibili i criteri di sicurezza di esempio per Blazor WebAssembly e Blazor Server :</span><span class="sxs-lookup"><span data-stu-id="74f9a-120">The following directives are used in the [Apply the policy](#apply-the-policy) section of this article, where example security policies for Blazor WebAssembly and Blazor Server are provided:</span></span>

* <span data-ttu-id="74f9a-121">[URI di base](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy/base-uri): limita gli URL per il tag di una pagina `<base>` .</span><span class="sxs-lookup"><span data-stu-id="74f9a-121">[base-uri](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy/base-uri): Restricts the URLs for a page's `<base>` tag.</span></span> <span data-ttu-id="74f9a-122">Specificare `self` per indicare che l'origine dell'app, inclusi lo schema e il numero di porta, è un'origine valida.</span><span class="sxs-lookup"><span data-stu-id="74f9a-122">Specify `self` to indicate that the app's origin, including the scheme and port number, is a valid source.</span></span>
* <span data-ttu-id="74f9a-123">[Block-All-Mixed-Content](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy/block-all-mixed-content): impedisce il caricamento di contenuto misto http e HTTPS.</span><span class="sxs-lookup"><span data-stu-id="74f9a-123">[block-all-mixed-content](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy/block-all-mixed-content): Prevents loading mixed HTTP and HTTPS content.</span></span>
* <span data-ttu-id="74f9a-124">[default-src](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy/default-src): indica un fallback per le direttive di origine non specificate in modo esplicito dal criterio.</span><span class="sxs-lookup"><span data-stu-id="74f9a-124">[default-src](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy/default-src): Indicates a fallback for source directives that aren't explicitly specified by the policy.</span></span> <span data-ttu-id="74f9a-125">Specificare `self` per indicare che l'origine dell'app, inclusi lo schema e il numero di porta, è un'origine valida.</span><span class="sxs-lookup"><span data-stu-id="74f9a-125">Specify `self` to indicate that the app's origin, including the scheme and port number, is a valid source.</span></span>
* <span data-ttu-id="74f9a-126">[img-src](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy/img-src): indica le origini valide per le immagini.</span><span class="sxs-lookup"><span data-stu-id="74f9a-126">[img-src](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy/img-src): Indicates valid sources for images.</span></span>
  * <span data-ttu-id="74f9a-127">`data:`Consente di specificare di consentire il caricamento di immagini dagli `data:` URL.</span><span class="sxs-lookup"><span data-stu-id="74f9a-127">Specify `data:` to permit loading images from `data:` URLs.</span></span>
  * <span data-ttu-id="74f9a-128">`https:`Consente di specificare di consentire il caricamento di immagini dagli endpoint HTTPS.</span><span class="sxs-lookup"><span data-stu-id="74f9a-128">Specify `https:` to permit loading images from HTTPS endpoints.</span></span>
* <span data-ttu-id="74f9a-129">[Object-src](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy/object-src): indica le origini valide per `<object>` i `<embed>` tag, e `<applet>` .</span><span class="sxs-lookup"><span data-stu-id="74f9a-129">[object-src](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy/object-src): Indicates valid sources for the `<object>`, `<embed>`, and `<applet>` tags.</span></span> <span data-ttu-id="74f9a-130">Specificare `none` per impedire tutte le origini URL.</span><span class="sxs-lookup"><span data-stu-id="74f9a-130">Specify `none` to prevent all URL sources.</span></span>
* <span data-ttu-id="74f9a-131">[script-src](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy/script-src): indica le origini valide per gli script.</span><span class="sxs-lookup"><span data-stu-id="74f9a-131">[script-src](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy/script-src): Indicates valid sources for scripts.</span></span>
  * <span data-ttu-id="74f9a-132">Specificare l' `https://stackpath.bootstrapcdn.com/` origine host per gli script bootstrap.</span><span class="sxs-lookup"><span data-stu-id="74f9a-132">Specify the `https://stackpath.bootstrapcdn.com/` host source for Bootstrap scripts.</span></span>
  * <span data-ttu-id="74f9a-133">Specificare `self` per indicare che l'origine dell'app, inclusi lo schema e il numero di porta, è un'origine valida.</span><span class="sxs-lookup"><span data-stu-id="74f9a-133">Specify `self` to indicate that the app's origin, including the scheme and port number, is a valid source.</span></span>
  * <span data-ttu-id="74f9a-134">In un' Blazor WebAssembly app:</span><span class="sxs-lookup"><span data-stu-id="74f9a-134">In a Blazor WebAssembly app:</span></span>
    * <span data-ttu-id="74f9a-135">Specificare gli hash per consentire il caricamento degli script necessari.</span><span class="sxs-lookup"><span data-stu-id="74f9a-135">Specify hashes to permit required scripts to load.</span></span>
    * <span data-ttu-id="74f9a-136">Specificare `unsafe-eval` per usare `eval()` i metodi e per la creazione di codice da stringhe.</span><span class="sxs-lookup"><span data-stu-id="74f9a-136">Specify `unsafe-eval` to use `eval()` and methods for creating code from strings.</span></span>
  * <span data-ttu-id="74f9a-137">In un' Blazor Server app specificare gli hash per consentire il caricamento degli script necessari.</span><span class="sxs-lookup"><span data-stu-id="74f9a-137">In a Blazor Server app, specify hashes to permit required scripts to load.</span></span>
* <span data-ttu-id="74f9a-138">[Style-src](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy/style-src): indica le origini valide per i fogli di stile.</span><span class="sxs-lookup"><span data-stu-id="74f9a-138">[style-src](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy/style-src): Indicates valid sources for stylesheets.</span></span>
  * <span data-ttu-id="74f9a-139">Specificare l' `https://stackpath.bootstrapcdn.com/` origine host per i fogli di stile bootstrap.</span><span class="sxs-lookup"><span data-stu-id="74f9a-139">Specify the `https://stackpath.bootstrapcdn.com/` host source for Bootstrap stylesheets.</span></span>
  * <span data-ttu-id="74f9a-140">Specificare `self` per indicare che l'origine dell'app, inclusi lo schema e il numero di porta, è un'origine valida.</span><span class="sxs-lookup"><span data-stu-id="74f9a-140">Specify `self` to indicate that the app's origin, including the scheme and port number, is a valid source.</span></span>
  * <span data-ttu-id="74f9a-141">Specificare `unsafe-inline` per consentire l'utilizzo di stili inline.</span><span class="sxs-lookup"><span data-stu-id="74f9a-141">Specify `unsafe-inline` to allow the use of inline styles.</span></span> <span data-ttu-id="74f9a-142">La dichiarazione inline è necessaria per l'interfaccia utente nelle Blazor Server app per la riconnessione del client e del server dopo la richiesta iniziale.</span><span class="sxs-lookup"><span data-stu-id="74f9a-142">The inline declaration is required for the UI in Blazor Server apps for reconnecting the client and server after the initial request.</span></span> <span data-ttu-id="74f9a-143">In una versione futura, lo stile inline potrebbe essere rimosso in modo che `unsafe-inline` non sia più necessario.</span><span class="sxs-lookup"><span data-stu-id="74f9a-143">In a future release, inline styling might be removed so that `unsafe-inline` is no longer required.</span></span>
* <span data-ttu-id="74f9a-144">[upgrade-unsecure-requests](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy/upgrade-insecure-requests): indica che gli URL del contenuto da origini non sicure (http) devono essere acquisiti in modo sicuro su HTTPS.</span><span class="sxs-lookup"><span data-stu-id="74f9a-144">[upgrade-insecure-requests](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy/upgrade-insecure-requests): Indicates that content URLs from insecure (HTTP) sources should be acquired securely over HTTPS.</span></span>

<span data-ttu-id="74f9a-145">Le direttive precedenti sono supportate da tutti i browser ad eccezione di Microsoft Internet Explorer.</span><span class="sxs-lookup"><span data-stu-id="74f9a-145">The preceding directives are supported by all browsers except Microsoft Internet Explorer.</span></span>

<span data-ttu-id="74f9a-146">Per ottenere gli hash SHA per altri script inline:</span><span class="sxs-lookup"><span data-stu-id="74f9a-146">To obtain SHA hashes for additional inline scripts:</span></span>

* <span data-ttu-id="74f9a-147">Applicare il CSP illustrato nella sezione [applicare i criteri](#apply-the-policy) .</span><span class="sxs-lookup"><span data-stu-id="74f9a-147">Apply the CSP shown in the [Apply the policy](#apply-the-policy) section.</span></span>
* <span data-ttu-id="74f9a-148">Accedere alla console degli strumenti di sviluppo del browser durante l'esecuzione locale dell'app.</span><span class="sxs-lookup"><span data-stu-id="74f9a-148">Access the browser's developer tools console while running the app locally.</span></span> <span data-ttu-id="74f9a-149">Il browser calcola e visualizza gli hash per gli script bloccati quando è presente un'intestazione o un `meta` tag CSP.</span><span class="sxs-lookup"><span data-stu-id="74f9a-149">The browser calculates and displays hashes for blocked scripts when a CSP header or `meta` tag is present.</span></span>
* <span data-ttu-id="74f9a-150">Copiare gli hash forniti dal browser nelle `script-src` origini.</span><span class="sxs-lookup"><span data-stu-id="74f9a-150">Copy the hashes provided by the browser to the `script-src` sources.</span></span> <span data-ttu-id="74f9a-151">Usare le virgolette singole intorno a ogni hash.</span><span class="sxs-lookup"><span data-stu-id="74f9a-151">Use single quotes around each hash.</span></span>

<span data-ttu-id="74f9a-152">Per una matrice di supporto del browser livello 2 del criterio di sicurezza del contenuto, vedere è [possibile usare: livello di sicurezza del contenuto 2](https://www.caniuse.com/#feat=contentsecuritypolicy2).</span><span class="sxs-lookup"><span data-stu-id="74f9a-152">For a Content Security Policy Level 2 browser support matrix, see [Can I use: Content Security Policy Level 2](https://www.caniuse.com/#feat=contentsecuritypolicy2).</span></span>

## <a name="apply-the-policy"></a><span data-ttu-id="74f9a-153">Applicare i criteri</span><span class="sxs-lookup"><span data-stu-id="74f9a-153">Apply the policy</span></span>

<span data-ttu-id="74f9a-154">Usare un `<meta>` tag per applicare i criteri:</span><span class="sxs-lookup"><span data-stu-id="74f9a-154">Use a `<meta>` tag to apply the policy:</span></span>

* <span data-ttu-id="74f9a-155">Impostare il valore dell' `http-equiv` attributo su `Content-Security-Policy` .</span><span class="sxs-lookup"><span data-stu-id="74f9a-155">Set the value of the `http-equiv` attribute to `Content-Security-Policy`.</span></span>
* <span data-ttu-id="74f9a-156">Inserire le direttive nel `content` valore dell'attributo.</span><span class="sxs-lookup"><span data-stu-id="74f9a-156">Place the directives in the `content` attribute value.</span></span> <span data-ttu-id="74f9a-157">Separare le direttive con un punto e virgola ( `;` ).</span><span class="sxs-lookup"><span data-stu-id="74f9a-157">Separate directives with a semicolon (`;`).</span></span>
* <span data-ttu-id="74f9a-158">Posizionare sempre il `meta` tag nel `<head>` contenuto.</span><span class="sxs-lookup"><span data-stu-id="74f9a-158">Always place the `meta` tag in the `<head>` content.</span></span>

<span data-ttu-id="74f9a-159">Nelle sezioni seguenti vengono illustrati i criteri di esempio per Blazor WebAssembly e Blazor Server .</span><span class="sxs-lookup"><span data-stu-id="74f9a-159">The following sections show example policies for Blazor WebAssembly and Blazor Server.</span></span> <span data-ttu-id="74f9a-160">Questi esempi sono disponibili in questo articolo per ogni versione di Blazor .</span><span class="sxs-lookup"><span data-stu-id="74f9a-160">These examples are versioned with this article for each release of Blazor.</span></span> <span data-ttu-id="74f9a-161">Per usare una versione appropriata per il rilascio, selezionare la versione del documento con il selettore a discesa **versione** in questa pagina Web.</span><span class="sxs-lookup"><span data-stu-id="74f9a-161">To use a version appropriate for your release, select the document version with the **Version** drop down selector on this webpage.</span></span>

### Blazor WebAssembly

<span data-ttu-id="74f9a-162">Nel `<head>` contenuto della `wwwroot/index.html` pagina host applicare le direttive descritte nella sezione [direttive dei criteri](#policy-directives) :</span><span class="sxs-lookup"><span data-stu-id="74f9a-162">In the `<head>` content of the `wwwroot/index.html` host page, apply the directives described in the [Policy directives](#policy-directives) section:</span></span>

::: moniker range=">= aspnetcore-5.0"

```html
<meta http-equiv="Content-Security-Policy" 
      content="base-uri 'self';
               block-all-mixed-content;
               default-src 'self';
               img-src data: https:;
               object-src 'none';
               script-src https://stackpath.bootstrapcdn.com/ 
                          'self' 
                          'sha256-v8v3RKRPmN4odZ1CWM5gw80QKPCCWMcpNeOmimNL2AA=' 
                          'unsafe-eval';
               style-src https://stackpath.bootstrapcdn.com/
                         'self'
                         'unsafe-inline';
               upgrade-insecure-requests;">
```

::: moniker-end

::: moniker range="< aspnetcore-5.0"

```html
<meta http-equiv="Content-Security-Policy" 
      content="base-uri 'self';
               block-all-mixed-content;
               default-src 'self';
               img-src data: https:;
               object-src 'none';
               script-src https://stackpath.bootstrapcdn.com/ 
                          'self' 
                          'sha256-v8ZC9OgMhcnEQ/Me77/R9TlJfzOBqrMTW8e1KuqLaqc=' 
                          'sha256-If//FtbPc03afjLezvWHnC3Nbu4fDM04IIzkPaf3pH0=' 
                          'sha256-v8v3RKRPmN4odZ1CWM5gw80QKPCCWMcpNeOmimNL2AA=' 
                          'unsafe-eval';
               style-src https://stackpath.bootstrapcdn.com/
                         'self'
                         'unsafe-inline';
               upgrade-insecure-requests;">
```

::: moniker-end

<span data-ttu-id="74f9a-163">Aggiungere altri `script-src` `style-src` hash e come richiesto dall'app.</span><span class="sxs-lookup"><span data-stu-id="74f9a-163">Add additional `script-src` and `style-src` hashes as required by the app.</span></span> <span data-ttu-id="74f9a-164">Durante lo sviluppo, usare uno strumento online o strumenti di sviluppo del browser per calcolare automaticamente gli hash.</span><span class="sxs-lookup"><span data-stu-id="74f9a-164">During development, use an online tool or browser developer tools to have the hashes calculated for you.</span></span> <span data-ttu-id="74f9a-165">Ad esempio, il seguente errore della console degli strumenti del browser segnala l'hash per uno script obbligatorio non coperto dal criterio:</span><span class="sxs-lookup"><span data-stu-id="74f9a-165">For example, the following browser tools console error reports the hash for a required script not covered by the policy:</span></span>

> <span data-ttu-id="74f9a-166">Non è stato possibile eseguire lo script inline perché viola la seguente direttiva sui criteri di sicurezza del contenuto: "... ".</span><span class="sxs-lookup"><span data-stu-id="74f9a-166">Refused to execute inline script because it violates the following Content Security Policy directive: " ... ".</span></span> <span data-ttu-id="74f9a-167">Per abilitare l'esecuzione inline è necessario specificare la parola chiave ' unsafe-inline ', un hash (' SHA256-v8v3RKRPmN4odZ1CWM5gw80QKPCCWMcpNeOmimNL2AA =') o un parametro nonce (' nonce-...').</span><span class="sxs-lookup"><span data-stu-id="74f9a-167">Either the 'unsafe-inline' keyword, a hash ('sha256-v8v3RKRPmN4odZ1CWM5gw80QKPCCWMcpNeOmimNL2AA='), or a nonce ('nonce-...') is required to enable inline execution.</span></span>

<span data-ttu-id="74f9a-168">Lo script specifico associato all'errore viene visualizzato nella console di accanto all'errore.</span><span class="sxs-lookup"><span data-stu-id="74f9a-168">The particular script associated with the error is displayed in the console next to the error.</span></span>

### Blazor Server

<span data-ttu-id="74f9a-169">Nel `<head>` contenuto della `Pages/_Host.cshtml` pagina host applicare le direttive descritte nella sezione [direttive dei criteri](#policy-directives) :</span><span class="sxs-lookup"><span data-stu-id="74f9a-169">In the `<head>` content of the `Pages/_Host.cshtml` host page, apply the directives described in the [Policy directives](#policy-directives) section:</span></span>

::: moniker range=">= aspnetcore-5.0"

```cshtml
<meta http-equiv="Content-Security-Policy" 
      content="base-uri 'self';
               block-all-mixed-content;
               default-src 'self';
               img-src data: https:;
               object-src 'none';
               script-src https://stackpath.bootstrapcdn.com/ 
                          'self';
               style-src https://stackpath.bootstrapcdn.com/
                         'self' 
                         'unsafe-inline';
               upgrade-insecure-requests;">
```

::: moniker-end

::: moniker range="< aspnetcore-5.0"

```cshtml
<meta http-equiv="Content-Security-Policy" 
      content="base-uri 'self';
               block-all-mixed-content;
               default-src 'self';
               img-src data: https:;
               object-src 'none';
               script-src https://stackpath.bootstrapcdn.com/ 
                          'self' 
                          'sha256-34WLX60Tw3aG6hylk0plKbZZFXCuepeQ6Hu7OqRf8PI=';
               style-src https://stackpath.bootstrapcdn.com/
                         'self' 
                         'unsafe-inline';
               upgrade-insecure-requests;">
```

::: moniker-end

<span data-ttu-id="74f9a-170">Aggiungere altri `script-src` `style-src` hash e come richiesto dall'app.</span><span class="sxs-lookup"><span data-stu-id="74f9a-170">Add additional `script-src` and `style-src` hashes as required by the app.</span></span> <span data-ttu-id="74f9a-171">Durante lo sviluppo, usare uno strumento online o strumenti di sviluppo del browser per calcolare automaticamente gli hash.</span><span class="sxs-lookup"><span data-stu-id="74f9a-171">During development, use an online tool or browser developer tools to have the hashes calculated for you.</span></span> <span data-ttu-id="74f9a-172">Ad esempio, il seguente errore della console degli strumenti del browser segnala l'hash per uno script obbligatorio non coperto dal criterio:</span><span class="sxs-lookup"><span data-stu-id="74f9a-172">For example, the following browser tools console error reports the hash for a required script not covered by the policy:</span></span>

> <span data-ttu-id="74f9a-173">Non è stato possibile eseguire lo script inline perché viola la seguente direttiva sui criteri di sicurezza del contenuto: "... ".</span><span class="sxs-lookup"><span data-stu-id="74f9a-173">Refused to execute inline script because it violates the following Content Security Policy directive: " ... ".</span></span> <span data-ttu-id="74f9a-174">Per abilitare l'esecuzione inline è necessario specificare la parola chiave ' unsafe-inline ', un hash (' SHA256-v8v3RKRPmN4odZ1CWM5gw80QKPCCWMcpNeOmimNL2AA =') o un parametro nonce (' nonce-...').</span><span class="sxs-lookup"><span data-stu-id="74f9a-174">Either the 'unsafe-inline' keyword, a hash ('sha256-v8v3RKRPmN4odZ1CWM5gw80QKPCCWMcpNeOmimNL2AA='), or a nonce ('nonce-...') is required to enable inline execution.</span></span>

<span data-ttu-id="74f9a-175">Lo script specifico associato all'errore viene visualizzato nella console di accanto all'errore.</span><span class="sxs-lookup"><span data-stu-id="74f9a-175">The particular script associated with the error is displayed in the console next to the error.</span></span>

## <a name="meta-tag-limitations"></a><span data-ttu-id="74f9a-176">Limitazioni tag meta</span><span class="sxs-lookup"><span data-stu-id="74f9a-176">Meta tag limitations</span></span>

<span data-ttu-id="74f9a-177">Un `<meta>` criterio tag non supporta le direttive seguenti:</span><span class="sxs-lookup"><span data-stu-id="74f9a-177">A `<meta>` tag policy doesn't support the following directives:</span></span>

* [<span data-ttu-id="74f9a-178">frame-predecessori</span><span class="sxs-lookup"><span data-stu-id="74f9a-178">frame-ancestors</span></span>](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy/frame-ancestors)
* [<span data-ttu-id="74f9a-179">report-a</span><span class="sxs-lookup"><span data-stu-id="74f9a-179">report-to</span></span>](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy/report-to)
* [<span data-ttu-id="74f9a-180">URI del report</span><span class="sxs-lookup"><span data-stu-id="74f9a-180">report-uri</span></span>](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy/report-uri)
* [<span data-ttu-id="74f9a-181">sandbox</span><span class="sxs-lookup"><span data-stu-id="74f9a-181">sandbox</span></span>](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy/sandbox)

<span data-ttu-id="74f9a-182">Per supportare le direttive precedenti, usare un'intestazione denominata `Content-Security-Policy` .</span><span class="sxs-lookup"><span data-stu-id="74f9a-182">To support the preceding directives, use a header named `Content-Security-Policy`.</span></span> <span data-ttu-id="74f9a-183">La stringa di direttiva è il valore dell'intestazione.</span><span class="sxs-lookup"><span data-stu-id="74f9a-183">The directive string is the header's value.</span></span>

## <a name="test-a-policy-and-receive-violation-reports"></a><span data-ttu-id="74f9a-184">Testare un criterio e ricevere i report sulle violazioni</span><span class="sxs-lookup"><span data-stu-id="74f9a-184">Test a policy and receive violation reports</span></span>

<span data-ttu-id="74f9a-185">Il test aiuta a confermare che gli script di terze parti non vengono bloccati inavvertitamente durante la compilazione di un criterio iniziale.</span><span class="sxs-lookup"><span data-stu-id="74f9a-185">Testing helps confirm that third-party scripts aren't inadvertently blocked when building an initial policy.</span></span>

<span data-ttu-id="74f9a-186">Per testare un criterio in un periodo di tempo senza applicare le direttive dei criteri, impostare il `<meta>` `http-equiv` nome dell'attributo o dell'intestazione del tag di un criterio basato su intestazione su `Content-Security-Policy-Report-Only` .</span><span class="sxs-lookup"><span data-stu-id="74f9a-186">To test a policy over a period of time without enforcing the policy directives, set the `<meta>` tag's `http-equiv` attribute or header name of a header-based policy to `Content-Security-Policy-Report-Only`.</span></span> <span data-ttu-id="74f9a-187">I report degli errori vengono inviati come documenti JSON a un URL specificato.</span><span class="sxs-lookup"><span data-stu-id="74f9a-187">Failure reports are sent as JSON documents to a specified URL.</span></span> <span data-ttu-id="74f9a-188">Per ulteriori informazioni, vedere la pagina relativa ai [documenti MDN Web: Content-Security-Policy-Report-only](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy-Report-Only).</span><span class="sxs-lookup"><span data-stu-id="74f9a-188">For more information, see [MDN web docs: Content-Security-Policy-Report-Only](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy-Report-Only).</span></span>

<span data-ttu-id="74f9a-189">Per la creazione di report sulle violazioni mentre un criterio è attivo, vedere gli articoli seguenti:</span><span class="sxs-lookup"><span data-stu-id="74f9a-189">For reporting on violations while a policy is active, see the following articles:</span></span>

* [<span data-ttu-id="74f9a-190">report-a</span><span class="sxs-lookup"><span data-stu-id="74f9a-190">report-to</span></span>](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy/report-to)
* [<span data-ttu-id="74f9a-191">URI del report</span><span class="sxs-lookup"><span data-stu-id="74f9a-191">report-uri</span></span>](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy/report-uri)

<span data-ttu-id="74f9a-192">Sebbene `report-uri` non sia più consigliato per l'uso, è necessario usare entrambe le direttive fino a quando non `report-to` è supportato da tutti i browser principali.</span><span class="sxs-lookup"><span data-stu-id="74f9a-192">Although `report-uri` is no longer recommended for use, both directives should be used until `report-to` is supported by all of the major browsers.</span></span> <span data-ttu-id="74f9a-193">Non usare esclusivamente `report-uri` perché il supporto per `report-uri` è soggetto all'eliminazione *in qualsiasi momento* dai browser.</span><span class="sxs-lookup"><span data-stu-id="74f9a-193">Don't exclusively use `report-uri` because support for `report-uri` is subject to being dropped *at any time* from browsers.</span></span> <span data-ttu-id="74f9a-194">Rimuovere il supporto per `report-uri` nei criteri quando `report-to` è completamente supportato.</span><span class="sxs-lookup"><span data-stu-id="74f9a-194">Remove support for `report-uri` in your policies when `report-to` is fully supported.</span></span> <span data-ttu-id="74f9a-195">Per tenere traccia dell'adozione di `report-to` , vedere è [possibile usare: report-to](https://caniuse.com/#feat=mdn-http_headers_csp_content-security-policy_report-to).</span><span class="sxs-lookup"><span data-stu-id="74f9a-195">To track adoption of `report-to`, see [Can I use: report-to](https://caniuse.com/#feat=mdn-http_headers_csp_content-security-policy_report-to).</span></span>

<span data-ttu-id="74f9a-196">Testare e aggiornare i criteri di un'app ogni versione.</span><span class="sxs-lookup"><span data-stu-id="74f9a-196">Test and update an app's policy every release.</span></span>

## <a name="troubleshoot"></a><span data-ttu-id="74f9a-197">Risolvere problemi</span><span class="sxs-lookup"><span data-stu-id="74f9a-197">Troubleshoot</span></span>

* <span data-ttu-id="74f9a-198">Gli errori vengono visualizzati nella console degli strumenti di sviluppo del browser.</span><span class="sxs-lookup"><span data-stu-id="74f9a-198">Errors appear in the browser's developer tools console.</span></span> <span data-ttu-id="74f9a-199">I browser forniscono informazioni su:</span><span class="sxs-lookup"><span data-stu-id="74f9a-199">Browsers provide information about:</span></span>
  * <span data-ttu-id="74f9a-200">Elementi che non sono conformi ai criteri.</span><span class="sxs-lookup"><span data-stu-id="74f9a-200">Elements that don't comply with the policy.</span></span>
  * <span data-ttu-id="74f9a-201">Come modificare i criteri per consentire un elemento bloccato.</span><span class="sxs-lookup"><span data-stu-id="74f9a-201">How to modify the policy to allow for a blocked item.</span></span>
* <span data-ttu-id="74f9a-202">Un criterio è completamente efficace quando il browser del client supporta tutte le direttive incluse.</span><span class="sxs-lookup"><span data-stu-id="74f9a-202">A policy is only completely effective when the client's browser supports all of the included directives.</span></span> <span data-ttu-id="74f9a-203">Per una matrice di supporto del browser corrente, vedere è [possibile usare: Content-Security-Policy](https://caniuse.com/#search=Content-Security-Policy).</span><span class="sxs-lookup"><span data-stu-id="74f9a-203">For a current browser support matrix, see [Can I use: Content-Security-Policy](https://caniuse.com/#search=Content-Security-Policy).</span></span>

## <a name="additional-resources"></a><span data-ttu-id="74f9a-204">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="74f9a-204">Additional resources</span></span>

* [<span data-ttu-id="74f9a-205">Documenti MDN Web: Content-Security-Policy</span><span class="sxs-lookup"><span data-stu-id="74f9a-205">MDN web docs: Content-Security-Policy</span></span>](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy)
* [<span data-ttu-id="74f9a-206">Livello dei criteri di sicurezza del contenuto 2</span><span class="sxs-lookup"><span data-stu-id="74f9a-206">Content Security Policy Level 2</span></span>](https://www.w3.org/TR/CSP2/)
* [<span data-ttu-id="74f9a-207">Analizzatore di Google CSP</span><span class="sxs-lookup"><span data-stu-id="74f9a-207">Google CSP Evaluator</span></span>](https://csp-evaluator.withgoogle.com/)
