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
ms.openlocfilehash: 7f147e29040b16966af1de8130ac36ff83c2a796
ms.sourcegitcommit: a49c47d5a573379effee5c6b6e36f5c302aa756b
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/16/2021
ms.locfileid: "100552866"
---
<span data-ttu-id="17b3f-101">La pagina prodotta dal `Authentication` componente ( `Pages/Authentication.razor` ) definisce le route necessarie per la gestione di diverse fasi di autenticazione.</span><span class="sxs-lookup"><span data-stu-id="17b3f-101">The page produced by the `Authentication` component (`Pages/Authentication.razor`) defines the routes required for handling different authentication stages.</span></span>

<span data-ttu-id="17b3f-102">Il <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteAuthenticatorView> componente:</span><span class="sxs-lookup"><span data-stu-id="17b3f-102">The <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteAuthenticatorView> component:</span></span>

* <span data-ttu-id="17b3f-103">Viene fornito dal [`Microsoft.AspNetCore.Components.WebAssembly.Authentication`](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.WebAssembly.Authentication/) pacchetto.</span><span class="sxs-lookup"><span data-stu-id="17b3f-103">Is provided by the [`Microsoft.AspNetCore.Components.WebAssembly.Authentication`](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.WebAssembly.Authentication/) package.</span></span>
* <span data-ttu-id="17b3f-104">Gestisce l'esecuzione delle azioni appropriate in ogni fase dell'autenticazione.</span><span class="sxs-lookup"><span data-stu-id="17b3f-104">Manages performing the appropriate actions at each stage of authentication.</span></span>

```razor
@page "/authentication/{action}"
@using Microsoft.AspNetCore.Components.WebAssembly.Authentication

<RemoteAuthenticatorView Action="@Action" />

@code {
    [Parameter]
    public string Action { get; set; }
}
```
