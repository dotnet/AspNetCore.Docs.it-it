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
ms.openlocfilehash: 9c977e5407f9a3dc562ef0fb1127fefaa0dc5fc2
ms.sourcegitcommit: a49c47d5a573379effee5c6b6e36f5c302aa756b
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/16/2021
ms.locfileid: "100552616"
---
<a name="ddav"></a>
### <a name="disable-default-account-verification"></a>Disabilitare la verifica dell'account predefinita

Con i modelli predefiniti, l'utente viene reindirizzato al `Account.RegisterConfirmation` dove è possibile selezionare un collegamento per confermare l'account. Il valore predefinito `Account.RegisterConfirmation` viene usato ***solo*** per i test. la verifica automatica dell'account deve essere disabilitata in un'app di produzione.

Per richiedere un account confermato e impedire l'accesso immediato alla registrazione, impostare `DisplayConfirmAccountLink = false` in */areas/ Identity /pages/account/RegisterConfirmation.cshtml.cs*:

[!code-csharp[](~/security/authentication/identity/sample/WebApp3/Areas/Identity/Pages/Account/RegisterConfirmation.cshtml.cs?name=snippet&highlight=34)]