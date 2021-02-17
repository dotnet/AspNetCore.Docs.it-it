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
<span data-ttu-id="968cd-101">I trailer HTTP sono simili alle intestazioni HTTP, con la differenza che vengono inviati dopo l'invio del corpo della risposta.</span><span class="sxs-lookup"><span data-stu-id="968cd-101">HTTP Trailers are similar to HTTP Headers, except they are sent after the response body is sent.</span></span> <span data-ttu-id="968cd-102">Per IIS e HTTP.sys sono supportati solo i trailer di risposta HTTP/2.</span><span class="sxs-lookup"><span data-stu-id="968cd-102">For IIS and HTTP.sys, only HTTP/2 response trailers are supported.</span></span>

```csharp
if (httpContext.Response.SupportsTrailers())
{
    httpContext.Response.DeclareTrailer("trailername"); 

    // Write body
    httpContext.Response.WriteAsync("Hello world");

    httpContext.Response.AppendTrailer("trailername", "TrailerValue");
}
```

<span data-ttu-id="968cd-103">Nell'esempio di codice precedente:</span><span class="sxs-lookup"><span data-stu-id="968cd-103">In the preceding example code:</span></span>

* <span data-ttu-id="968cd-104">`SupportsTrailers` garantisce che i trailer siano supportati per la risposta.</span><span class="sxs-lookup"><span data-stu-id="968cd-104">`SupportsTrailers` ensures that trailers are supported for the response.</span></span>
* <span data-ttu-id="968cd-105">`DeclareTrailer` aggiunge il nome del trailer specificato all' `Trailer` intestazione della risposta.</span><span class="sxs-lookup"><span data-stu-id="968cd-105">`DeclareTrailer` adds the given trailer name to the `Trailer` response header.</span></span> <span data-ttu-id="968cd-106">La dichiarazione dei trailer di una risposta è facoltativa, ma consigliata.</span><span class="sxs-lookup"><span data-stu-id="968cd-106">Declaring a response's trailers is optional, but recommended.</span></span> <span data-ttu-id="968cd-107">Se `DeclareTrailer` viene chiamato, deve essere prima dell'invio delle intestazioni di risposta.</span><span class="sxs-lookup"><span data-stu-id="968cd-107">If `DeclareTrailer` is called, it must be before the response headers are sent.</span></span>
* <span data-ttu-id="968cd-108">`AppendTrailer` Accoda il trailer.</span><span class="sxs-lookup"><span data-stu-id="968cd-108">`AppendTrailer` appends the trailer.</span></span>
