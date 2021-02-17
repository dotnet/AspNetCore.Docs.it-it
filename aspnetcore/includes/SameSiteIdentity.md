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
ms.openlocfilehash: b6f6bc2e094c9070e0ea57b507f558313f19bc15
ms.sourcegitcommit: a49c47d5a573379effee5c6b6e36f5c302aa756b
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/16/2021
ms.locfileid: "100551624"
---
Il ASP.NET Core [Identity](xref:security/authentication/identity) è in gran parte inalterato da [ cookie navigava sullostesso sito](xref:security/samesite) , ad eccezione di scenari avanzati come `IFrames` o l' `OpenIdConnect` integrazione.

Quando `Identity` si usa, ***non*** aggiungere alcun cookie provider o chiamare ` services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)` `Identity` .