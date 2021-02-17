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
ms.openlocfilehash: 164c9e8f73502fc627c4514bd683dc183d318b1d
ms.sourcegitcommit: a49c47d5a573379effee5c6b6e36f5c302aa756b
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/16/2021
ms.locfileid: "100551823"
---
Reset consente al server di reimpostare una richiesta HTTP/2 con un codice di errore specificato. Una richiesta di reimpostazione viene considerata interrotta.

```csharp
var resetFeature = httpContext.Features.Get<IHttpResetFeature>();
resetFeature.Reset(errorCode: 2);
```

`Reset` Nell'esempio di codice precedente viene specificato il `INTERNAL_ERROR` codice di errore. Per ulteriori informazioni sui codici di errore HTTP/2, vedere la [sezione codice di errore della specifica http/2](https://tools.ietf.org/html/rfc7540#page-50).