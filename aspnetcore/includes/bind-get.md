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
ms.openlocfilehash: c6880852f63b1e91789667506f01680608933226
ms.sourcegitcommit: a49c47d5a573379effee5c6b6e36f5c302aa756b
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/16/2021
ms.locfileid: "100552638"
---
> [!WARNING]
> Per motivi di sicurezza, è necessario acconsentire esplicitamente all'associazione dei dati della richiesta `GET` alle proprietà del modello di pagina. Verificare l'input dell'utente prima di eseguirne il mapping alle proprietà. La scelta dell' `GET` associazione è utile per gli scenari che si basano sulla stringa di query o sui valori della route.
>
> Per associare una proprietà alle `GET` richieste, impostare la [`[BindProperty]`](xref:Microsoft.AspNetCore.Mvc.BindPropertyAttribute) proprietà dell'attributo su `SupportsGet` `true` :
>
> ```csharp
> [BindProperty(SupportsGet = true)]
> ```
>
> Per altre informazioni, vedere [ASP.NET Core community standup: bind on Get Discussion (YouTube)](https://www.youtube.com/watch?v=p7iHB9V-KVU&feature=youtu.be&t=54m27s).
