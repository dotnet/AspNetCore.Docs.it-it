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
ms.openlocfilehash: 1b045d437d1a16eabc0ab41573c8b66d9c4bb77e
ms.sourcegitcommit: a49c47d5a573379effee5c6b6e36f5c302aa756b
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/16/2021
ms.locfileid: "100551509"
---
<span data-ttu-id="d8e8b-101">Per impostazione predefinita, il Framework esegue il popup della modalità di accesso e viene eseguito il fallback per reindirizzare la modalità di accesso se non è possibile aprire un popup.</span><span class="sxs-lookup"><span data-stu-id="d8e8b-101">The framework defaults to pop-up login mode and falls back to redirect login mode if a pop-up can't be opened.</span></span> <span data-ttu-id="d8e8b-102">Configurare MSAL per l'uso della modalità di accesso Reindirizzamento impostando la `LoginMode` proprietà di <xref:Microsoft.Authentication.WebAssembly.Msal.Models.MsalProviderOptions> su `redirect` :</span><span class="sxs-lookup"><span data-stu-id="d8e8b-102">Configure MSAL to use redirect login mode by setting the `LoginMode` property of <xref:Microsoft.Authentication.WebAssembly.Msal.Models.MsalProviderOptions> to `redirect`:</span></span>

```csharp
builder.Services.AddMsalAuthentication(options =>
{
    ...
    options.ProviderOptions.LoginMode = "redirect";
});
```

<span data-ttu-id="d8e8b-103">L'impostazione predefinita è `popup` e il valore stringa non fa distinzione tra maiuscole e minuscole.</span><span class="sxs-lookup"><span data-stu-id="d8e8b-103">The default setting is `popup`, and the string value isn't case sensitive.</span></span>
