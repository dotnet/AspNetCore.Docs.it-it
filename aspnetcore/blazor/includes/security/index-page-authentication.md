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
La `wwwroot/index.html` pagina di indice () include uno script che definisce `AuthenticationService` in JavaScript. `AuthenticationService` gestisce i dettagli di basso livello del protocollo OIDC. L'app chiama internamente i metodi definiti nello script per eseguire le operazioni di autenticazione.

```html
<script src="_content/Microsoft.AspNetCore.Components.WebAssembly.Authentication/
    AuthenticationService.js"></script>
```
