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
# <a name="enforce-a-content-security-policy-for-aspnet-core-blazor"></a>Applicare un criterio di sicurezza del contenuto per ASP.NET Core Blazor

Il [cross-site scripting (XSS)](xref:security/cross-site-scripting) è una vulnerabilità di sicurezza in cui un utente malintenzionato inserisce uno o più script lato client dannosi nel contenuto sottoposto a rendering di un'app. Un criterio di sicurezza del contenuto (CSP) consente di proteggersi da attacchi XSS, informando il browser di valido:

* Origini per il contenuto caricato, inclusi script, fogli di stile e immagini.
* Azioni eseguite da una pagina, che specifica le destinazioni URL consentite dei moduli.
* Plug-in che possono essere caricati.

Per applicare un CSP a un'app, lo sviluppatore specifica diverse *direttive* di sicurezza del contenuto CSP in una o più `Content-Security-Policy` intestazioni o `<meta>` tag.

I criteri vengono valutati dal browser durante il caricamento di una pagina. Il browser controlla le origini della pagina e determina se soddisfano i requisiti delle direttive di sicurezza del contenuto. Quando le direttive dei criteri non vengono soddisfatte per una risorsa, il browser non carica la risorsa. Si consideri, ad esempio, un criterio che non consente script di terze parti. Quando una pagina contiene un `<script>` tag con un'origine di terze parti nell' `src` attributo, il browser impedisce il caricamento dello script.

CSP è supportato nella maggior parte dei browser moderni per desktop e per dispositivi mobili, tra cui Chrome, Edge, Firefox, opera e Safari. CSP è consigliato per le Blazor app.

## <a name="policy-directives"></a>Direttive dei criteri

Specificare almeno le direttive e le origini seguenti per le Blazor app. Aggiungere altre direttive e origini in base alle esigenze. Le direttive seguenti vengono usate nella sezione [applicare i criteri](#apply-the-policy) di questo articolo, in cui sono disponibili i criteri di sicurezza di esempio per Blazor WebAssembly e Blazor Server :

* [URI di base](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy/base-uri): limita gli URL per il tag di una pagina `<base>` . Specificare `self` per indicare che l'origine dell'app, inclusi lo schema e il numero di porta, è un'origine valida.
* [Block-All-Mixed-Content](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy/block-all-mixed-content): impedisce il caricamento di contenuto misto http e HTTPS.
* [default-src](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy/default-src): indica un fallback per le direttive di origine non specificate in modo esplicito dal criterio. Specificare `self` per indicare che l'origine dell'app, inclusi lo schema e il numero di porta, è un'origine valida.
* [img-src](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy/img-src): indica le origini valide per le immagini.
  * `data:`Consente di specificare di consentire il caricamento di immagini dagli `data:` URL.
  * `https:`Consente di specificare di consentire il caricamento di immagini dagli endpoint HTTPS.
* [Object-src](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy/object-src): indica le origini valide per `<object>` i `<embed>` tag, e `<applet>` . Specificare `none` per impedire tutte le origini URL.
* [script-src](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy/script-src): indica le origini valide per gli script.
  * Specificare l' `https://stackpath.bootstrapcdn.com/` origine host per gli script bootstrap.
  * Specificare `self` per indicare che l'origine dell'app, inclusi lo schema e il numero di porta, è un'origine valida.
  * In un' Blazor WebAssembly app:
    * Specificare gli hash per consentire il caricamento degli script necessari.
    * Specificare `unsafe-eval` per usare `eval()` i metodi e per la creazione di codice da stringhe.
  * In un' Blazor Server app specificare gli hash per consentire il caricamento degli script necessari.
* [Style-src](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy/style-src): indica le origini valide per i fogli di stile.
  * Specificare l' `https://stackpath.bootstrapcdn.com/` origine host per i fogli di stile bootstrap.
  * Specificare `self` per indicare che l'origine dell'app, inclusi lo schema e il numero di porta, è un'origine valida.
  * Specificare `unsafe-inline` per consentire l'utilizzo di stili inline. La dichiarazione inline è necessaria per l'interfaccia utente nelle Blazor Server app per la riconnessione del client e del server dopo la richiesta iniziale. In una versione futura, lo stile inline potrebbe essere rimosso in modo che `unsafe-inline` non sia più necessario.
* [upgrade-unsecure-requests](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy/upgrade-insecure-requests): indica che gli URL del contenuto da origini non sicure (http) devono essere acquisiti in modo sicuro su HTTPS.

Le direttive precedenti sono supportate da tutti i browser ad eccezione di Microsoft Internet Explorer.

Per ottenere gli hash SHA per altri script inline:

* Applicare il CSP illustrato nella sezione [applicare i criteri](#apply-the-policy) .
* Accedere alla console degli strumenti di sviluppo del browser durante l'esecuzione locale dell'app. Il browser calcola e visualizza gli hash per gli script bloccati quando è presente un'intestazione o un `meta` tag CSP.
* Copiare gli hash forniti dal browser nelle `script-src` origini. Usare le virgolette singole intorno a ogni hash.

Per una matrice di supporto del browser livello 2 del criterio di sicurezza del contenuto, vedere è [possibile usare: livello di sicurezza del contenuto 2](https://www.caniuse.com/#feat=contentsecuritypolicy2).

## <a name="apply-the-policy"></a>Applicare i criteri

Usare un `<meta>` tag per applicare i criteri:

* Impostare il valore dell' `http-equiv` attributo su `Content-Security-Policy` .
* Inserire le direttive nel `content` valore dell'attributo. Separare le direttive con un punto e virgola ( `;` ).
* Posizionare sempre il `meta` tag nel `<head>` contenuto.

Nelle sezioni seguenti vengono illustrati i criteri di esempio per Blazor WebAssembly e Blazor Server . Questi esempi sono disponibili in questo articolo per ogni versione di Blazor . Per usare una versione appropriata per il rilascio, selezionare la versione del documento con il selettore a discesa **versione** in questa pagina Web.

### Blazor WebAssembly

Nel `<head>` contenuto della `wwwroot/index.html` pagina host applicare le direttive descritte nella sezione [direttive dei criteri](#policy-directives) :

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

Aggiungere altri `script-src` `style-src` hash e come richiesto dall'app. Durante lo sviluppo, usare uno strumento online o strumenti di sviluppo del browser per calcolare automaticamente gli hash. Ad esempio, il seguente errore della console degli strumenti del browser segnala l'hash per uno script obbligatorio non coperto dal criterio:

> Non è stato possibile eseguire lo script inline perché viola la seguente direttiva sui criteri di sicurezza del contenuto: "... ". Per abilitare l'esecuzione inline è necessario specificare la parola chiave ' unsafe-inline ', un hash (' SHA256-v8v3RKRPmN4odZ1CWM5gw80QKPCCWMcpNeOmimNL2AA =') o un parametro nonce (' nonce-...').

Lo script specifico associato all'errore viene visualizzato nella console di accanto all'errore.

### Blazor Server

Nel `<head>` contenuto della `Pages/_Host.cshtml` pagina host applicare le direttive descritte nella sezione [direttive dei criteri](#policy-directives) :

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

Aggiungere altri `script-src` `style-src` hash e come richiesto dall'app. Durante lo sviluppo, usare uno strumento online o strumenti di sviluppo del browser per calcolare automaticamente gli hash. Ad esempio, il seguente errore della console degli strumenti del browser segnala l'hash per uno script obbligatorio non coperto dal criterio:

> Non è stato possibile eseguire lo script inline perché viola la seguente direttiva sui criteri di sicurezza del contenuto: "... ". Per abilitare l'esecuzione inline è necessario specificare la parola chiave ' unsafe-inline ', un hash (' SHA256-v8v3RKRPmN4odZ1CWM5gw80QKPCCWMcpNeOmimNL2AA =') o un parametro nonce (' nonce-...').

Lo script specifico associato all'errore viene visualizzato nella console di accanto all'errore.

## <a name="meta-tag-limitations"></a>Limitazioni tag meta

Un `<meta>` criterio tag non supporta le direttive seguenti:

* [frame-predecessori](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy/frame-ancestors)
* [report-a](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy/report-to)
* [URI del report](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy/report-uri)
* [sandbox](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy/sandbox)

Per supportare le direttive precedenti, usare un'intestazione denominata `Content-Security-Policy` . La stringa di direttiva è il valore dell'intestazione.

## <a name="test-a-policy-and-receive-violation-reports"></a>Testare un criterio e ricevere i report sulle violazioni

Il test aiuta a confermare che gli script di terze parti non vengono bloccati inavvertitamente durante la compilazione di un criterio iniziale.

Per testare un criterio in un periodo di tempo senza applicare le direttive dei criteri, impostare il `<meta>` `http-equiv` nome dell'attributo o dell'intestazione del tag di un criterio basato su intestazione su `Content-Security-Policy-Report-Only` . I report degli errori vengono inviati come documenti JSON a un URL specificato. Per ulteriori informazioni, vedere la pagina relativa ai [documenti MDN Web: Content-Security-Policy-Report-only](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy-Report-Only).

Per la creazione di report sulle violazioni mentre un criterio è attivo, vedere gli articoli seguenti:

* [report-a](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy/report-to)
* [URI del report](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy/report-uri)

Sebbene `report-uri` non sia più consigliato per l'uso, è necessario usare entrambe le direttive fino a quando non `report-to` è supportato da tutti i browser principali. Non usare esclusivamente `report-uri` perché il supporto per `report-uri` è soggetto all'eliminazione *in qualsiasi momento* dai browser. Rimuovere il supporto per `report-uri` nei criteri quando `report-to` è completamente supportato. Per tenere traccia dell'adozione di `report-to` , vedere è [possibile usare: report-to](https://caniuse.com/#feat=mdn-http_headers_csp_content-security-policy_report-to).

Testare e aggiornare i criteri di un'app ogni versione.

## <a name="troubleshoot"></a>Risolvere problemi

* Gli errori vengono visualizzati nella console degli strumenti di sviluppo del browser. I browser forniscono informazioni su:
  * Elementi che non sono conformi ai criteri.
  * Come modificare i criteri per consentire un elemento bloccato.
* Un criterio è completamente efficace quando il browser del client supporta tutte le direttive incluse. Per una matrice di supporto del browser corrente, vedere è [possibile usare: Content-Security-Policy](https://caniuse.com/#search=Content-Security-Policy).

## <a name="additional-resources"></a>Risorse aggiuntive

* [Documenti MDN Web: Content-Security-Policy](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy)
* [Livello dei criteri di sicurezza del contenuto 2](https://www.w3.org/TR/CSP2/)
* [Analizzatore di Google CSP](https://csp-evaluator.withgoogle.com/)
