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
Per impostazione predefinita, il Framework esegue il popup della modalità di accesso e viene eseguito il fallback per reindirizzare la modalità di accesso se non è possibile aprire un popup. Configurare MSAL per l'uso della modalità di accesso Reindirizzamento impostando la `LoginMode` proprietà di <xref:Microsoft.Authentication.WebAssembly.Msal.Models.MsalProviderOptions> su `redirect` :

```csharp
builder.Services.AddMsalAuthentication(options =>
{
    ...
    options.ProviderOptions.LoginMode = "redirect";
});
```

L'impostazione predefinita è `popup` e il valore stringa non fa distinzione tra maiuscole e minuscole.
