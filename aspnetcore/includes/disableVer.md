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
### <a name="disable-default-account-verification"></a><span data-ttu-id="676cf-101">Disabilitare la verifica dell'account predefinita</span><span class="sxs-lookup"><span data-stu-id="676cf-101">Disable default account verification</span></span>

<span data-ttu-id="676cf-102">Con i modelli predefiniti, l'utente viene reindirizzato al `Account.RegisterConfirmation` dove è possibile selezionare un collegamento per confermare l'account.</span><span class="sxs-lookup"><span data-stu-id="676cf-102">With the default templates, the user is redirected to the `Account.RegisterConfirmation` where they can select a link to have the account confirmed.</span></span> <span data-ttu-id="676cf-103">Il valore predefinito `Account.RegisterConfirmation` viene usato ***solo*** per i test. la verifica automatica dell'account deve essere disabilitata in un'app di produzione.</span><span class="sxs-lookup"><span data-stu-id="676cf-103">The default `Account.RegisterConfirmation` is used ***only*** for testing, automatic account verification should be disabled in a production app.</span></span>

<span data-ttu-id="676cf-104">Per richiedere un account confermato e impedire l'accesso immediato alla registrazione, impostare `DisplayConfirmAccountLink = false` in */areas/ Identity /pages/account/RegisterConfirmation.cshtml.cs*:</span><span class="sxs-lookup"><span data-stu-id="676cf-104">To require a confirmed account and prevent immediate login at registration, set `DisplayConfirmAccountLink = false` in */Areas/Identity/Pages/Account/RegisterConfirmation.cshtml.cs*:</span></span>

[!code-csharp[](~/security/authentication/identity/sample/WebApp3/Areas/Identity/Pages/Account/RegisterConfirmation.cshtml.cs?name=snippet&highlight=34)]