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
ms.openlocfilehash: 087c5a7dd20bc653a05c337dde905716032e00b3
ms.sourcegitcommit: a49c47d5a573379effee5c6b6e36f5c302aa756b
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/16/2021
ms.locfileid: "100552520"
---
## <a name="debug-diagnostics"></a><span data-ttu-id="8c45b-101">Diagnostica di debug</span><span class="sxs-lookup"><span data-stu-id="8c45b-101">Debug diagnostics</span></span>

<span data-ttu-id="8c45b-102">Per un output di diagnostica dettagliato del routing, impostare `Logging:LogLevel:Microsoft` su `Debug` .</span><span class="sxs-lookup"><span data-stu-id="8c45b-102">For detailed routing diagnostic output, set `Logging:LogLevel:Microsoft` to `Debug`.</span></span> <span data-ttu-id="8c45b-103">Nell'ambiente di sviluppo impostare il livello di registrazione in *appsettings.Development.js* in:</span><span class="sxs-lookup"><span data-stu-id="8c45b-103">In the development environment, set the log level in *appsettings.Development.json*:</span></span>

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Debug",
      "Microsoft.Hosting.Lifetime": "Information"
    }
  }
}
```
