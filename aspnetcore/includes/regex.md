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
ms.openlocfilehash: 73395ab632f38fef1348b4657770eb52db5778d9
ms.sourcegitcommit: a49c47d5a573379effee5c6b6e36f5c302aa756b
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/16/2021
ms.locfileid: "100551898"
---
> [!WARNING]
> <span data-ttu-id="6f9aa-101">Quando <xref:System.Text.RegularExpressions> si usa per elaborare un input non attendibile, passare un timeout.</span><span class="sxs-lookup"><span data-stu-id="6f9aa-101">When using <xref:System.Text.RegularExpressions> to process untrusted input, pass a timeout.</span></span> <span data-ttu-id="6f9aa-102">Un utente malintenzionato può fornire l'input per `RegularExpressions` provocare un [attacco Denial of Service](https://www.us-cert.gov/ncas/tips/ST04-015).</span><span class="sxs-lookup"><span data-stu-id="6f9aa-102">A malicious user can provide input to `RegularExpressions` causing a [Denial-of-Service attack](https://www.us-cert.gov/ncas/tips/ST04-015).</span></span> <span data-ttu-id="6f9aa-103">Le API di ASP.NET Core Framework che usano `RegularExpressions` passano un timeout.</span><span class="sxs-lookup"><span data-stu-id="6f9aa-103">ASP.NET Core framework APIs that use `RegularExpressions` pass a timeout.</span></span>