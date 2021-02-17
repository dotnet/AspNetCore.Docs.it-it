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
ms.openlocfilehash: 7caea4089d3624d4c02db4b8adbe9edb73f3d31a
ms.sourcegitcommit: a49c47d5a573379effee5c6b6e36f5c302aa756b
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/16/2021
ms.locfileid: "100551685"
---
> [!NOTE]
> Prima di ASP.NET Core 3,0, le app Web scrivono un log di tipo `LogLevel.Warning` per ogni richiesta se le impostazioni cultura richieste non sono supportate. La registrazione `LogLevel.Warning` di uno per ogni richiesta può creare file di log di grandi dimensioni con informazioni ridondanti. Questo comportamento è stato modificato in ASP.NET 3,0. `RequestLocalizationMiddleware`Scrive un log di tipo `LogLevel.Debug` , che riduce le dimensioni dei log di produzione.
