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
* Considerare attendibile il certificato di sviluppo HTTPS eseguendo il comando riportato di seguito:

  ```dotnetcli
  dotnet dev-certs https --trust
  ```
  
  Il comando precedente non funziona in Linux. Vedere la documentazione della distribuzione di Linux per l'attendibilità di un certificato.

  Il comando precedente consente di visualizzare la finestra di dialogo seguente:

  ![Finestra di dialogo Avviso di sicurezza](~/getting-started/_static/cert.png)

* Selezionare **Sì** se si accetta di considerare attendibile il certificato di sviluppo.

  Per altre informazioni, vedere [Considerare attendibile il certificato di sviluppo di ASP.NET Core HTTPS](xref:security/enforcing-ssl#trust-the-aspnet-core-https-development-certificate-on-windows-and-macos).
  
[!INCLUDE[trust FF](~/includes/trust-ff.md)]