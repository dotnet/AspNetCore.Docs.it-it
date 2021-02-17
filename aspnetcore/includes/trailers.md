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
ms.openlocfilehash: d5e1630d6a9e412c40831aa3a66aaed7524ff501
ms.sourcegitcommit: a49c47d5a573379effee5c6b6e36f5c302aa756b
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/16/2021
ms.locfileid: "100552231"
---
I trailer HTTP sono simili alle intestazioni HTTP, con la differenza che vengono inviati dopo l'invio del corpo della risposta. Per IIS e HTTP.sys sono supportati solo i trailer di risposta HTTP/2.

```csharp
if (httpContext.Response.SupportsTrailers())
{
    httpContext.Response.DeclareTrailer("trailername"); 

    // Write body
    httpContext.Response.WriteAsync("Hello world");

    httpContext.Response.AppendTrailer("trailername", "TrailerValue");
}
```

Nell'esempio di codice precedente:

* `SupportsTrailers` garantisce che i trailer siano supportati per la risposta.
* `DeclareTrailer` aggiunge il nome del trailer specificato all' `Trailer` intestazione della risposta. La dichiarazione dei trailer di una risposta è facoltativa, ma consigliata. Se `DeclareTrailer` viene chiamato, deve essere prima dell'invio delle intestazioni di risposta.
* `AppendTrailer` Accoda il trailer.
