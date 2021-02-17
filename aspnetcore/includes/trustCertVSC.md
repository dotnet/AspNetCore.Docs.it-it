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
ms.openlocfilehash: df4a9a7db8097ea765b2d0f404305b771f994ddf
ms.sourcegitcommit: a49c47d5a573379effee5c6b6e36f5c302aa756b
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 02/16/2021
ms.locfileid: "100552807"
---
* <span data-ttu-id="cc16e-101">Considerare attendibile il certificato di sviluppo HTTPS eseguendo il comando riportato di seguito:</span><span class="sxs-lookup"><span data-stu-id="cc16e-101">Trust the HTTPS development certificate by running the following command:</span></span>

  ```dotnetcli
  dotnet dev-certs https --trust
  ```
  
  <span data-ttu-id="cc16e-102">Il comando precedente non funziona in Linux.</span><span class="sxs-lookup"><span data-stu-id="cc16e-102">The preceding command doesn't work on Linux.</span></span> <span data-ttu-id="cc16e-103">Vedere la documentazione della distribuzione di Linux per l'attendibilità di un certificato.</span><span class="sxs-lookup"><span data-stu-id="cc16e-103">See your Linux distribution's documentation for trusting a certificate.</span></span>

  <span data-ttu-id="cc16e-104">Il comando precedente consente di visualizzare la finestra di dialogo seguente:</span><span class="sxs-lookup"><span data-stu-id="cc16e-104">The preceding command displays the following dialog:</span></span>

  ![Finestra di dialogo Avviso di sicurezza](~/getting-started/_static/cert.png)

* <span data-ttu-id="cc16e-106">Selezionare **Sì** se si accetta di considerare attendibile il certificato di sviluppo.</span><span class="sxs-lookup"><span data-stu-id="cc16e-106">Select **Yes** if you agree to trust the development certificate.</span></span>

  <span data-ttu-id="cc16e-107">Per altre informazioni, vedere [Considerare attendibile il certificato di sviluppo di ASP.NET Core HTTPS](xref:security/enforcing-ssl#trust-the-aspnet-core-https-development-certificate-on-windows-and-macos).</span><span class="sxs-lookup"><span data-stu-id="cc16e-107">See [Trust the ASP.NET Core HTTPS development certificate](xref:security/enforcing-ssl#trust-the-aspnet-core-https-development-certificate-on-windows-and-macos) for more information.</span></span>
  
[!INCLUDE[trust FF](~/includes/trust-ff.md)]