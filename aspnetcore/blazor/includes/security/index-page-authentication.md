---
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
ms.openlocfilehash: b2519eaab7a83d66e60be05b1c2deeb8094b016f
ms.sourcegitcommit: a49c47d5a573379effee5c6b6e36f5c302aa756b
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/16/2021
ms.locfileid: "100551996"
---
<span data-ttu-id="9c26d-101">La `wwwroot/index.html` pagina di indice () include uno script che definisce `AuthenticationService` in JavaScript.</span><span class="sxs-lookup"><span data-stu-id="9c26d-101">The Index page (`wwwroot/index.html`) page includes a script that defines the `AuthenticationService` in JavaScript.</span></span> <span data-ttu-id="9c26d-102">`AuthenticationService` gestisce i dettagli di basso livello del protocollo OIDC.</span><span class="sxs-lookup"><span data-stu-id="9c26d-102">`AuthenticationService` handles the low-level details of the OIDC protocol.</span></span> <span data-ttu-id="9c26d-103">L'app chiama internamente i metodi definiti nello script per eseguire le operazioni di autenticazione.</span><span class="sxs-lookup"><span data-stu-id="9c26d-103">The app internally calls methods defined in the script to perform the authentication operations.</span></span>

```html
<script src="_content/Microsoft.AspNetCore.Components.WebAssembly.Authentication/
    AuthenticationService.js"></script>
```
